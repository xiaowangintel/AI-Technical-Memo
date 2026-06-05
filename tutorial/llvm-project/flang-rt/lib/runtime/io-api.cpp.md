# io-api.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `flang-rt/lib/runtime/io-api.cpp` | `flang-rt/lib/runtime/io-api.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements the Flang runtime library, including descriptors, I/O, memory management, numerics, and execution support. In this file, the main focus is `io api`; the header comment highlights: Implements the I/O statement API. | 实现 Flang 运行时库，包括描述符、I/O、内存管理、数值计算与执行支持。 本文件的核心主题是 `io api`；文件头注释强调：Implements the I/O statement API。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````cpp
//===-- lib/runtime/io-api.cpp ----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Implements the I/O statement API

// template function BeginExternalListIo<> is in runtime/io-api-common.h.
// APIs BeginExternalListOutput, OutputInteger{8,16,32,64,128},
// OutputReal{32,64}, OutputComplex{32,64}, OutputAscii, & EndIoStatement()
// are in runtime/io-api-minimal.cpp.

#include "flang/Runtime/io-api.h"
#include "descriptor-io.h"
#include "edit-input.h"
#include "edit-output.h"
#include "io-api-common.h"
#include "unit.h"
#include "flang-rt/runtime/descriptor.h"
#include "flang-rt/runtime/environment.h"
#include "flang-rt/runtime/format.h"
#include "flang-rt/runtime/io-stmt.h"
#include "flang-rt/runtime/memory.h"
#include "flang-rt/runtime/terminator.h"
#include "flang-rt/runtime/tools.h"
#include "flang/Common/optional.h"
#include <cstdlib>
#include <memory>

````

- **L1 EN**: Comment documents intent or context: `lib/runtime/io-api.cpp ----------------------------------*- C++ -*-===//`.
  **L1 CN**: 注释记录了意图或上下文：`lib/runtime/io-api.cpp ----------------------------------*- C++ -*-===//`。
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
- **L9 EN**: Comment documents intent or context: `Implements the I/O statement API`.
  **L9 CN**: 注释记录了意图或上下文：`Implements the I/O statement API`。
- **L10 EN**: Blank line separates nearby declarations or logic blocks.
  **L10 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L11 EN**: Comment documents intent or context: `template function BeginExternalListIo<> is in runtime/io-api-common.h.`.
  **L11 CN**: 注释记录了意图或上下文：`template function BeginExternalListIo<> is in runtime/io-api-common.h.`。
- **L12 EN**: Comment documents intent or context: `APIs BeginExternalListOutput, OutputInteger{8,16,32,64,128},`.
  **L12 CN**: 注释记录了意图或上下文：`APIs BeginExternalListOutput, OutputInteger{8,16,32,64,128},`。
- **L13 EN**: Comment documents intent or context: `OutputReal{32,64}, OutputComplex{32,64}, OutputAscii, & EndIoStatement()`.
  **L13 CN**: 注释记录了意图或上下文：`OutputReal{32,64}, OutputComplex{32,64}, OutputAscii, & EndIoStatement()`。
- **L14 EN**: Comment documents intent or context: `are in runtime/io-api-minimal.cpp.`.
  **L14 CN**: 注释记录了意图或上下文：`are in runtime/io-api-minimal.cpp.`。
- **L15 EN**: Blank line separates nearby declarations or logic blocks.
  **L15 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L16 EN**: Includes `flang/Runtime/io-api.h` to access Flang runtime declarations.
  **L16 CN**: 引入 `flang/Runtime/io-api.h` 以使用 Flang 运行时声明。
- **L17 EN**: Includes `descriptor-io.h` to access project-local declarations and helper interfaces.
  **L17 CN**: 引入 `descriptor-io.h` 以使用 项目内声明与辅助接口。
- **L18 EN**: Includes `edit-input.h` to access project-local declarations and helper interfaces.
  **L18 CN**: 引入 `edit-input.h` 以使用 项目内声明与辅助接口。
- **L19 EN**: Includes `edit-output.h` to access project-local declarations and helper interfaces.
  **L19 CN**: 引入 `edit-output.h` 以使用 项目内声明与辅助接口。
- **L20 EN**: Includes `io-api-common.h` to access project-local declarations and helper interfaces.
  **L20 CN**: 引入 `io-api-common.h` 以使用 项目内声明与辅助接口。
- **L21 EN**: Includes `unit.h` to access project-local declarations and helper interfaces.
  **L21 CN**: 引入 `unit.h` 以使用 项目内声明与辅助接口。
- **L22 EN**: Includes `flang-rt/runtime/descriptor.h` to access Flang runtime public headers.
  **L22 CN**: 引入 `flang-rt/runtime/descriptor.h` 以使用 Flang 运行时公共头文件。
- **L23 EN**: Includes `flang-rt/runtime/environment.h` to access Flang runtime public headers.
  **L23 CN**: 引入 `flang-rt/runtime/environment.h` 以使用 Flang 运行时公共头文件。
- **L24 EN**: Includes `flang-rt/runtime/format.h` to access Flang runtime public headers.
  **L24 CN**: 引入 `flang-rt/runtime/format.h` 以使用 Flang 运行时公共头文件。
- **L25 EN**: Includes `flang-rt/runtime/io-stmt.h` to access Flang runtime public headers.
  **L25 CN**: 引入 `flang-rt/runtime/io-stmt.h` 以使用 Flang 运行时公共头文件。
- **L26 EN**: Includes `flang-rt/runtime/memory.h` to access Flang runtime public headers.
  **L26 CN**: 引入 `flang-rt/runtime/memory.h` 以使用 Flang 运行时公共头文件。
- **L27 EN**: Includes `flang-rt/runtime/terminator.h` to access Flang runtime public headers.
  **L27 CN**: 引入 `flang-rt/runtime/terminator.h` 以使用 Flang 运行时公共头文件。
- **L28 EN**: Includes `flang-rt/runtime/tools.h` to access Flang runtime public headers.
  **L28 CN**: 引入 `flang-rt/runtime/tools.h` 以使用 Flang 运行时公共头文件。
- **L29 EN**: Includes `flang/Common/optional.h` to access Flang common data structures and compiler-wide helpers.
  **L29 CN**: 引入 `flang/Common/optional.h` 以使用 Flang 通用数据结构与编译器级辅助工具。
- **L30 EN**: Includes `cstdlib` to access general C runtime utilities.
  **L30 CN**: 引入 `cstdlib` 以使用 通用 C 运行时工具。
- **L31 EN**: Includes `memory` to access smart pointers and allocation helpers.
  **L31 CN**: 引入 `memory` 以使用 智能指针与分配辅助工具。
- **L32 EN**: Blank line separates nearby declarations or logic blocks.
  **L32 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 33-64

````cpp
namespace Fortran::runtime::io {
RT_EXT_API_GROUP_BEGIN

template <Direction DIR>
RT_API_ATTRS Cookie BeginInternalArrayListIO(const Descriptor &descriptor,
    void ** /*scratchArea*/, std::size_t /*scratchBytes*/,
    const char *sourceFile, int sourceLine) {
  Terminator oom{sourceFile, sourceLine};
  return &New<InternalListIoStatementState<DIR>>{oom}(
      descriptor, sourceFile, sourceLine)
              .release()
              ->ioStatementState();
}

Cookie IODEF(BeginInternalArrayListOutput)(const Descriptor &descriptor,
    void **scratchArea, std::size_t scratchBytes, const char *sourceFile,
    int sourceLine) {
  return BeginInternalArrayListIO<Direction::Output>(
      descriptor, scratchArea, scratchBytes, sourceFile, sourceLine);
}

Cookie IODEF(BeginInternalArrayListInput)(const Descriptor &descriptor,
    void **scratchArea, std::size_t scratchBytes, const char *sourceFile,
    int sourceLine) {
  return BeginInternalArrayListIO<Direction::Input>(
      descriptor, scratchArea, scratchBytes, sourceFile, sourceLine);
}

template <Direction DIR>
RT_API_ATTRS Cookie BeginInternalArrayFormattedIO(const Descriptor &descriptor,
    const char *format, std::size_t formatLength,
    const Descriptor *formatDescriptor, void ** /*scratchArea*/,
````

- **L33 EN**: Enters namespace `Fortran` to scope related declarations.
  **L33 CN**: 进入命名空间 `Fortran` 以组织相关声明。
- **L34 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L34 CN**: 延续周围的声明、表达式或控制流结构。
- **L35 EN**: Blank line separates nearby declarations or logic blocks.
  **L35 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L36 EN**: Begins a template declaration parameterizing subsequent code.
  **L36 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L37 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L37 CN**: 延续周围的声明、表达式或控制流结构。
- **L38 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L38 CN**: 延续周围的声明、表达式或控制流结构。
- **L39 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L39 CN**: 延续周围的声明、表达式或控制流结构。
- **L40 EN**: Executes statement `Terminator oom{sourceFile, sourceLine};`.
  **L40 CN**: 执行语句 `Terminator oom{sourceFile, sourceLine};`。
- **L41 EN**: Returns from the current function, often propagating a computed result.
  **L41 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L42 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L42 CN**: 延续周围的声明、表达式或控制流结构。
- **L43 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L43 CN**: 延续周围的声明、表达式或控制流结构。
- **L44 EN**: Executes statement involving `ioStatementState`.
  **L44 CN**: 执行涉及 `ioStatementState` 的语句。
- **L45 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L45 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L46 EN**: Blank line separates nearby declarations or logic blocks.
  **L46 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L47 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L47 CN**: 延续周围的声明、表达式或控制流结构。
- **L48 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L48 CN**: 延续周围的声明、表达式或控制流结构。
- **L49 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L49 CN**: 延续周围的声明、表达式或控制流结构。
- **L50 EN**: Returns from the current function, often propagating a computed result.
  **L50 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L51 EN**: Executes statement `descriptor, scratchArea, scratchBytes, sourceFile, sourceLine);`.
  **L51 CN**: 执行语句 `descriptor, scratchArea, scratchBytes, sourceFile, sourceLine);`。
- **L52 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L52 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L53 EN**: Blank line separates nearby declarations or logic blocks.
  **L53 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L54 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L54 CN**: 延续周围的声明、表达式或控制流结构。
- **L55 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L55 CN**: 延续周围的声明、表达式或控制流结构。
- **L56 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L56 CN**: 延续周围的声明、表达式或控制流结构。
- **L57 EN**: Returns from the current function, often propagating a computed result.
  **L57 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L58 EN**: Executes statement `descriptor, scratchArea, scratchBytes, sourceFile, sourceLine);`.
  **L58 CN**: 执行语句 `descriptor, scratchArea, scratchBytes, sourceFile, sourceLine);`。
- **L59 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L59 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L60 EN**: Blank line separates nearby declarations or logic blocks.
  **L60 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L61 EN**: Begins a template declaration parameterizing subsequent code.
  **L61 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L62 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L62 CN**: 延续周围的声明、表达式或控制流结构。
- **L63 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L63 CN**: 延续周围的声明、表达式或控制流结构。
- **L64 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L64 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 65-96

````cpp
    std::size_t /*scratchBytes*/, const char *sourceFile, int sourceLine) {
  Terminator oom{sourceFile, sourceLine};
  return &New<InternalFormattedIoStatementState<DIR>>{oom}(descriptor, format,
      formatLength, formatDescriptor, sourceFile, sourceLine)
              .release()
              ->ioStatementState();
}

Cookie IODEF(BeginInternalArrayFormattedOutput)(const Descriptor &descriptor,
    const char *format, std::size_t formatLength,
    const Descriptor *formatDescriptor, void **scratchArea,
    std::size_t scratchBytes, const char *sourceFile, int sourceLine) {
  return BeginInternalArrayFormattedIO<Direction::Output>(descriptor, format,
      formatLength, formatDescriptor, scratchArea, scratchBytes, sourceFile,
      sourceLine);
}

Cookie IODEF(BeginInternalArrayFormattedInput)(const Descriptor &descriptor,
    const char *format, std::size_t formatLength,
    const Descriptor *formatDescriptor, void **scratchArea,
    std::size_t scratchBytes, const char *sourceFile, int sourceLine) {
  return BeginInternalArrayFormattedIO<Direction::Input>(descriptor, format,
      formatLength, formatDescriptor, scratchArea, scratchBytes, sourceFile,
      sourceLine);
}

template <Direction DIR>
RT_API_ATTRS Cookie BeginInternalListIO(
    std::conditional_t<DIR == Direction::Input, const char, char> *internal,
    std::size_t internalLength, void ** /*scratchArea*/,
    std::size_t /*scratchBytes*/, const char *sourceFile, int sourceLine) {
  Terminator oom{sourceFile, sourceLine};
````

- **L65 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L65 CN**: 延续周围的声明、表达式或控制流结构。
- **L66 EN**: Executes statement `Terminator oom{sourceFile, sourceLine};`.
  **L66 CN**: 执行语句 `Terminator oom{sourceFile, sourceLine};`。
- **L67 EN**: Returns from the current function, often propagating a computed result.
  **L67 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L68 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L68 CN**: 延续周围的声明、表达式或控制流结构。
- **L69 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L69 CN**: 延续周围的声明、表达式或控制流结构。
- **L70 EN**: Executes statement involving `ioStatementState`.
  **L70 CN**: 执行涉及 `ioStatementState` 的语句。
- **L71 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L71 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L72 EN**: Blank line separates nearby declarations or logic blocks.
  **L72 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L73 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L73 CN**: 延续周围的声明、表达式或控制流结构。
- **L74 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L74 CN**: 延续周围的声明、表达式或控制流结构。
- **L75 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L75 CN**: 延续周围的声明、表达式或控制流结构。
- **L76 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L76 CN**: 延续周围的声明、表达式或控制流结构。
- **L77 EN**: Returns from the current function, often propagating a computed result.
  **L77 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L78 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L78 CN**: 延续周围的声明、表达式或控制流结构。
- **L79 EN**: Executes statement `sourceLine);`.
  **L79 CN**: 执行语句 `sourceLine);`。
- **L80 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L80 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L81 EN**: Blank line separates nearby declarations or logic blocks.
  **L81 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L82 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L82 CN**: 延续周围的声明、表达式或控制流结构。
- **L83 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L83 CN**: 延续周围的声明、表达式或控制流结构。
- **L84 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L84 CN**: 延续周围的声明、表达式或控制流结构。
- **L85 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L85 CN**: 延续周围的声明、表达式或控制流结构。
- **L86 EN**: Returns from the current function, often propagating a computed result.
  **L86 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L87 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L87 CN**: 延续周围的声明、表达式或控制流结构。
- **L88 EN**: Executes statement `sourceLine);`.
  **L88 CN**: 执行语句 `sourceLine);`。
- **L89 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L89 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L90 EN**: Blank line separates nearby declarations or logic blocks.
  **L90 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L91 EN**: Begins a template declaration parameterizing subsequent code.
  **L91 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L92 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L92 CN**: 延续周围的声明、表达式或控制流结构。
- **L93 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L93 CN**: 延续周围的声明、表达式或控制流结构。
- **L94 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L94 CN**: 延续周围的声明、表达式或控制流结构。
- **L95 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L95 CN**: 延续周围的声明、表达式或控制流结构。
- **L96 EN**: Executes statement `Terminator oom{sourceFile, sourceLine};`.
  **L96 CN**: 执行语句 `Terminator oom{sourceFile, sourceLine};`。

### Lines 97-128

````cpp
  return &New<InternalListIoStatementState<DIR>>{oom}(
      internal, internalLength, sourceFile, sourceLine)
              .release()
              ->ioStatementState();
}

Cookie IODEF(BeginInternalListOutput)(char *internal,
    std::size_t internalLength, void **scratchArea, std::size_t scratchBytes,
    const char *sourceFile, int sourceLine) {
  return BeginInternalListIO<Direction::Output>(internal, internalLength,
      scratchArea, scratchBytes, sourceFile, sourceLine);
}

Cookie IODEF(BeginInternalListInput)(const char *internal,
    std::size_t internalLength, void **scratchArea, std::size_t scratchBytes,
    const char *sourceFile, int sourceLine) {
  return BeginInternalListIO<Direction::Input>(internal, internalLength,
      scratchArea, scratchBytes, sourceFile, sourceLine);
}

template <Direction DIR>
RT_API_ATTRS Cookie BeginInternalFormattedIO(
    std::conditional_t<DIR == Direction::Input, const char, char> *internal,
    std::size_t internalLength, const char *format, std::size_t formatLength,
    const Descriptor *formatDescriptor, void ** /*scratchArea*/,
    std::size_t /*scratchBytes*/, const char *sourceFile, int sourceLine) {
  Terminator oom{sourceFile, sourceLine};
  return &New<InternalFormattedIoStatementState<DIR>>{oom}(internal,
      internalLength, format, formatLength, formatDescriptor, sourceFile,
      sourceLine)
              .release()
              ->ioStatementState();
````

- **L97 EN**: Returns from the current function, often propagating a computed result.
  **L97 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L98 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L98 CN**: 延续周围的声明、表达式或控制流结构。
- **L99 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L99 CN**: 延续周围的声明、表达式或控制流结构。
- **L100 EN**: Executes statement involving `ioStatementState`.
  **L100 CN**: 执行涉及 `ioStatementState` 的语句。
- **L101 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L101 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L102 EN**: Blank line separates nearby declarations or logic blocks.
  **L102 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L103 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L103 CN**: 延续周围的声明、表达式或控制流结构。
- **L104 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L104 CN**: 延续周围的声明、表达式或控制流结构。
- **L105 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L105 CN**: 延续周围的声明、表达式或控制流结构。
- **L106 EN**: Returns from the current function, often propagating a computed result.
  **L106 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L107 EN**: Executes statement `scratchArea, scratchBytes, sourceFile, sourceLine);`.
  **L107 CN**: 执行语句 `scratchArea, scratchBytes, sourceFile, sourceLine);`。
- **L108 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L108 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L109 EN**: Blank line separates nearby declarations or logic blocks.
  **L109 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L110 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L110 CN**: 延续周围的声明、表达式或控制流结构。
- **L111 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L111 CN**: 延续周围的声明、表达式或控制流结构。
- **L112 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L112 CN**: 延续周围的声明、表达式或控制流结构。
- **L113 EN**: Returns from the current function, often propagating a computed result.
  **L113 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L114 EN**: Executes statement `scratchArea, scratchBytes, sourceFile, sourceLine);`.
  **L114 CN**: 执行语句 `scratchArea, scratchBytes, sourceFile, sourceLine);`。
- **L115 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L115 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L116 EN**: Blank line separates nearby declarations or logic blocks.
  **L116 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L117 EN**: Begins a template declaration parameterizing subsequent code.
  **L117 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L118 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L118 CN**: 延续周围的声明、表达式或控制流结构。
- **L119 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L119 CN**: 延续周围的声明、表达式或控制流结构。
- **L120 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L120 CN**: 延续周围的声明、表达式或控制流结构。
- **L121 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L121 CN**: 延续周围的声明、表达式或控制流结构。
- **L122 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L122 CN**: 延续周围的声明、表达式或控制流结构。
- **L123 EN**: Executes statement `Terminator oom{sourceFile, sourceLine};`.
  **L123 CN**: 执行语句 `Terminator oom{sourceFile, sourceLine};`。
- **L124 EN**: Returns from the current function, often propagating a computed result.
  **L124 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L125 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L125 CN**: 延续周围的声明、表达式或控制流结构。
- **L126 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L126 CN**: 延续周围的声明、表达式或控制流结构。
- **L127 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L127 CN**: 延续周围的声明、表达式或控制流结构。
- **L128 EN**: Executes statement involving `ioStatementState`.
  **L128 CN**: 执行涉及 `ioStatementState` 的语句。

### Lines 129-160

````cpp
}

Cookie IODEF(BeginInternalFormattedOutput)(char *internal,
    std::size_t internalLength, const char *format, std::size_t formatLength,
    const Descriptor *formatDescriptor, void **scratchArea,
    std::size_t scratchBytes, const char *sourceFile, int sourceLine) {
  return BeginInternalFormattedIO<Direction::Output>(internal, internalLength,
      format, formatLength, formatDescriptor, scratchArea, scratchBytes,
      sourceFile, sourceLine);
}

Cookie IODEF(BeginInternalFormattedInput)(const char *internal,
    std::size_t internalLength, const char *format, std::size_t formatLength,
    const Descriptor *formatDescriptor, void **scratchArea,
    std::size_t scratchBytes, const char *sourceFile, int sourceLine) {
  return BeginInternalFormattedIO<Direction::Input>(internal, internalLength,
      format, formatLength, formatDescriptor, scratchArea, scratchBytes,
      sourceFile, sourceLine);
}

Cookie IODEF(BeginExternalListInput)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  return BeginExternalListIO<Direction::Input, ExternalListIoStatementState>(
      unitNumber, sourceFile, sourceLine);
}

template <Direction DIR>
RT_API_ATTRS Cookie BeginExternalFormattedIO(const char *format,
    std::size_t formatLength, const Descriptor *formatDescriptor,
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  Cookie errorCookie{nullptr};
````

- **L129 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L129 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L130 EN**: Blank line separates nearby declarations or logic blocks.
  **L130 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L131 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L131 CN**: 延续周围的声明、表达式或控制流结构。
- **L132 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L132 CN**: 延续周围的声明、表达式或控制流结构。
- **L133 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L133 CN**: 延续周围的声明、表达式或控制流结构。
- **L134 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L134 CN**: 延续周围的声明、表达式或控制流结构。
- **L135 EN**: Returns from the current function, often propagating a computed result.
  **L135 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L136 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L136 CN**: 延续周围的声明、表达式或控制流结构。
- **L137 EN**: Executes statement `sourceFile, sourceLine);`.
  **L137 CN**: 执行语句 `sourceFile, sourceLine);`。
- **L138 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L138 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L139 EN**: Blank line separates nearby declarations or logic blocks.
  **L139 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L140 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L140 CN**: 延续周围的声明、表达式或控制流结构。
- **L141 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L141 CN**: 延续周围的声明、表达式或控制流结构。
- **L142 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L142 CN**: 延续周围的声明、表达式或控制流结构。
- **L143 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L143 CN**: 延续周围的声明、表达式或控制流结构。
- **L144 EN**: Returns from the current function, often propagating a computed result.
  **L144 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L145 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L145 CN**: 延续周围的声明、表达式或控制流结构。
- **L146 EN**: Executes statement `sourceFile, sourceLine);`.
  **L146 CN**: 执行语句 `sourceFile, sourceLine);`。
- **L147 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L147 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L148 EN**: Blank line separates nearby declarations or logic blocks.
  **L148 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L149 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L149 CN**: 延续周围的声明、表达式或控制流结构。
- **L150 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L150 CN**: 延续周围的声明、表达式或控制流结构。
- **L151 EN**: Returns from the current function, often propagating a computed result.
  **L151 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L152 EN**: Executes statement `unitNumber, sourceFile, sourceLine);`.
  **L152 CN**: 执行语句 `unitNumber, sourceFile, sourceLine);`。
- **L153 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L153 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L154 EN**: Blank line separates nearby declarations or logic blocks.
  **L154 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L155 EN**: Begins a template declaration parameterizing subsequent code.
  **L155 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L156 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L156 CN**: 延续周围的声明、表达式或控制流结构。
- **L157 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L157 CN**: 延续周围的声明、表达式或控制流结构。
- **L158 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L158 CN**: 延续周围的声明、表达式或控制流结构。
- **L159 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L159 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L160 EN**: Executes statement `Cookie errorCookie{nullptr};`.
  **L160 CN**: 执行语句 `Cookie errorCookie{nullptr};`。

### Lines 161-192

````cpp
  ExternalFileUnit *unit{GetOrCreateUnit(
      unitNumber, DIR, false /*!unformatted*/, terminator, errorCookie)};
  if (!unit) {
    return errorCookie;
  }
  Iostat iostat{IostatOk};
  if (!unit->isUnformatted.has_value()) {
    unit->isUnformatted = false;
  }
  if (*unit->isUnformatted) {
    iostat = IostatFormattedIoOnUnformattedUnit;
  }
  if (ChildIo * child{unit->GetChildIo()}) {
    if (iostat == IostatOk) {
      iostat = child->CheckFormattingAndDirection(false, DIR);
    }
    if (iostat == IostatOk) {
      return &child->BeginIoStatement<ChildFormattedIoStatementState<DIR>>(
          *child, format, formatLength, formatDescriptor, sourceFile,
          sourceLine);
    } else {
      return &child->BeginIoStatement<ErroneousIoStatementState>(
          iostat, nullptr /* no unit */, sourceFile, sourceLine);
    }
  } else {
    if (iostat == IostatOk) {
      iostat = unit->SetDirection(DIR);
    }
    if (iostat == IostatOk) {
      return &unit->BeginIoStatement<ExternalFormattedIoStatementState<DIR>>(
          terminator, *unit, format, formatLength, formatDescriptor, sourceFile,
          sourceLine);
````

- **L161 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L161 CN**: 延续周围的声明、表达式或控制流结构。
- **L162 EN**: Executes statement `unitNumber, DIR, false /*!unformatted*/, terminator, errorCookie)};`.
  **L162 CN**: 执行语句 `unitNumber, DIR, false /*!unformatted*/, terminator, errorCookie)};`。
- **L163 EN**: Introduces conditional control flow with an `if` statement.
  **L163 CN**: 通过 `if` 语句引入条件控制流。
- **L164 EN**: Returns from the current function, often propagating a computed result.
  **L164 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L165 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L165 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L166 EN**: Executes statement `Iostat iostat{IostatOk};`.
  **L166 CN**: 执行语句 `Iostat iostat{IostatOk};`。
- **L167 EN**: Introduces conditional control flow with an `if` statement.
  **L167 CN**: 通过 `if` 语句引入条件控制流。
- **L168 EN**: Initializes or updates `unit->isUnformatted`.
  **L168 CN**: 初始化或更新 `unit->isUnformatted`。
- **L169 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L169 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L170 EN**: Introduces conditional control flow with an `if` statement.
  **L170 CN**: 通过 `if` 语句引入条件控制流。
- **L171 EN**: Initializes or updates `iostat`.
  **L171 CN**: 初始化或更新 `iostat`。
- **L172 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L172 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L173 EN**: Introduces conditional control flow with an `if` statement.
  **L173 CN**: 通过 `if` 语句引入条件控制流。
- **L174 EN**: Introduces conditional control flow with an `if` statement.
  **L174 CN**: 通过 `if` 语句引入条件控制流。
- **L175 EN**: Initializes or updates `iostat`.
  **L175 CN**: 初始化或更新 `iostat`。
- **L176 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L176 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L177 EN**: Introduces conditional control flow with an `if` statement.
  **L177 CN**: 通过 `if` 语句引入条件控制流。
- **L178 EN**: Returns from the current function, often propagating a computed result.
  **L178 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L179 EN**: Comment documents intent or context: `child, format, formatLength, formatDescriptor, sourceFile,`.
  **L179 CN**: 注释记录了意图或上下文：`child, format, formatLength, formatDescriptor, sourceFile,`。
- **L180 EN**: Executes statement `sourceLine);`.
  **L180 CN**: 执行语句 `sourceLine);`。
- **L181 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L181 CN**: 延续周围的声明、表达式或控制流结构。
- **L182 EN**: Returns from the current function, often propagating a computed result.
  **L182 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L183 EN**: Executes statement `iostat, nullptr /* no unit */, sourceFile, sourceLine);`.
  **L183 CN**: 执行语句 `iostat, nullptr /* no unit */, sourceFile, sourceLine);`。
- **L184 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L184 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L185 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L185 CN**: 延续周围的声明、表达式或控制流结构。
- **L186 EN**: Introduces conditional control flow with an `if` statement.
  **L186 CN**: 通过 `if` 语句引入条件控制流。
- **L187 EN**: Initializes or updates `iostat`.
  **L187 CN**: 初始化或更新 `iostat`。
- **L188 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L188 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L189 EN**: Introduces conditional control flow with an `if` statement.
  **L189 CN**: 通过 `if` 语句引入条件控制流。
- **L190 EN**: Returns from the current function, often propagating a computed result.
  **L190 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L191 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L191 CN**: 延续周围的声明、表达式或控制流结构。
- **L192 EN**: Executes statement `sourceLine);`.
  **L192 CN**: 执行语句 `sourceLine);`。

### Lines 193-224

````cpp
    } else {
      return &unit->BeginIoStatement<ErroneousIoStatementState>(
          terminator, iostat, unit, sourceFile, sourceLine);
    }
  }
}

Cookie IODEF(BeginExternalFormattedOutput)(const char *format,
    std::size_t formatLength, const Descriptor *formatDescriptor,
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  return BeginExternalFormattedIO<Direction::Output>(format, formatLength,
      formatDescriptor, unitNumber, sourceFile, sourceLine);
}

Cookie IODEF(BeginExternalFormattedInput)(const char *format,
    std::size_t formatLength, const Descriptor *formatDescriptor,
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  return BeginExternalFormattedIO<Direction::Input>(format, formatLength,
      formatDescriptor, unitNumber, sourceFile, sourceLine);
}

template <Direction DIR>
RT_API_ATTRS Cookie BeginUnformattedIO(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  Cookie errorCookie{nullptr};
  ExternalFileUnit *unit{GetOrCreateUnit(
      unitNumber, DIR, true /*unformatted*/, terminator, errorCookie)};
  if (!unit) {
    return errorCookie;
  }
  Iostat iostat{IostatOk};
````

- **L193 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L193 CN**: 延续周围的声明、表达式或控制流结构。
- **L194 EN**: Returns from the current function, often propagating a computed result.
  **L194 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L195 EN**: Executes statement `terminator, iostat, unit, sourceFile, sourceLine);`.
  **L195 CN**: 执行语句 `terminator, iostat, unit, sourceFile, sourceLine);`。
- **L196 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L196 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L197 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L197 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L198 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L198 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L199 EN**: Blank line separates nearby declarations or logic blocks.
  **L199 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L200 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L200 CN**: 延续周围的声明、表达式或控制流结构。
- **L201 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L201 CN**: 延续周围的声明、表达式或控制流结构。
- **L202 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L202 CN**: 延续周围的声明、表达式或控制流结构。
- **L203 EN**: Returns from the current function, often propagating a computed result.
  **L203 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L204 EN**: Executes statement `formatDescriptor, unitNumber, sourceFile, sourceLine);`.
  **L204 CN**: 执行语句 `formatDescriptor, unitNumber, sourceFile, sourceLine);`。
- **L205 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L205 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L206 EN**: Blank line separates nearby declarations or logic blocks.
  **L206 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L207 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L207 CN**: 延续周围的声明、表达式或控制流结构。
- **L208 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L208 CN**: 延续周围的声明、表达式或控制流结构。
- **L209 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L209 CN**: 延续周围的声明、表达式或控制流结构。
- **L210 EN**: Returns from the current function, often propagating a computed result.
  **L210 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L211 EN**: Executes statement `formatDescriptor, unitNumber, sourceFile, sourceLine);`.
  **L211 CN**: 执行语句 `formatDescriptor, unitNumber, sourceFile, sourceLine);`。
- **L212 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L212 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L213 EN**: Blank line separates nearby declarations or logic blocks.
  **L213 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L214 EN**: Begins a template declaration parameterizing subsequent code.
  **L214 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L215 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L215 CN**: 延续周围的声明、表达式或控制流结构。
- **L216 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L216 CN**: 延续周围的声明、表达式或控制流结构。
- **L217 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L217 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L218 EN**: Executes statement `Cookie errorCookie{nullptr};`.
  **L218 CN**: 执行语句 `Cookie errorCookie{nullptr};`。
- **L219 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L219 CN**: 延续周围的声明、表达式或控制流结构。
- **L220 EN**: Executes statement `unitNumber, DIR, true /*unformatted*/, terminator, errorCookie)};`.
  **L220 CN**: 执行语句 `unitNumber, DIR, true /*unformatted*/, terminator, errorCookie)};`。
- **L221 EN**: Introduces conditional control flow with an `if` statement.
  **L221 CN**: 通过 `if` 语句引入条件控制流。
- **L222 EN**: Returns from the current function, often propagating a computed result.
  **L222 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L223 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L223 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L224 EN**: Executes statement `Iostat iostat{IostatOk};`.
  **L224 CN**: 执行语句 `Iostat iostat{IostatOk};`。

### Lines 225-256

````cpp
  if (!unit->isUnformatted.has_value()) {
    unit->isUnformatted = true;
  }
  if (!*unit->isUnformatted) {
    iostat = IostatUnformattedIoOnFormattedUnit;
  }
  if (ChildIo * child{unit->GetChildIo()}) {
    if (iostat == IostatOk) {
      iostat = child->CheckFormattingAndDirection(true, DIR);
    }
    if (iostat == IostatOk) {
      return &child->BeginIoStatement<ChildUnformattedIoStatementState<DIR>>(
          *child, sourceFile, sourceLine);
    } else {
      return &child->BeginIoStatement<ErroneousIoStatementState>(
          iostat, nullptr /* no unit */, sourceFile, sourceLine);
    }
  } else {
    if (iostat == IostatOk) {
      iostat = unit->SetDirection(DIR);
    }
    if (iostat == IostatOk) {
      IoStatementState &io{
          unit->BeginIoStatement<ExternalUnformattedIoStatementState<DIR>>(
              terminator, *unit, sourceFile, sourceLine)};
      if constexpr (DIR == Direction::Output) {
        if (unit->access == Access::Sequential) {
          // Create space for (sub)record header to be completed by
          // ExternalFileUnit::AdvanceRecord()
          unit->recordLength.reset(); // in case of prior BACKSPACE
          io.Emit("\0\0\0\0", 4); // placeholder for record length header
        }
````

- **L225 EN**: Introduces conditional control flow with an `if` statement.
  **L225 CN**: 通过 `if` 语句引入条件控制流。
- **L226 EN**: Initializes or updates `unit->isUnformatted`.
  **L226 CN**: 初始化或更新 `unit->isUnformatted`。
- **L227 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L227 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L228 EN**: Introduces conditional control flow with an `if` statement.
  **L228 CN**: 通过 `if` 语句引入条件控制流。
- **L229 EN**: Initializes or updates `iostat`.
  **L229 CN**: 初始化或更新 `iostat`。
- **L230 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L230 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L231 EN**: Introduces conditional control flow with an `if` statement.
  **L231 CN**: 通过 `if` 语句引入条件控制流。
- **L232 EN**: Introduces conditional control flow with an `if` statement.
  **L232 CN**: 通过 `if` 语句引入条件控制流。
- **L233 EN**: Initializes or updates `iostat`.
  **L233 CN**: 初始化或更新 `iostat`。
- **L234 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L234 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L235 EN**: Introduces conditional control flow with an `if` statement.
  **L235 CN**: 通过 `if` 语句引入条件控制流。
- **L236 EN**: Returns from the current function, often propagating a computed result.
  **L236 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L237 EN**: Comment documents intent or context: `child, sourceFile, sourceLine);`.
  **L237 CN**: 注释记录了意图或上下文：`child, sourceFile, sourceLine);`。
- **L238 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L238 CN**: 延续周围的声明、表达式或控制流结构。
- **L239 EN**: Returns from the current function, often propagating a computed result.
  **L239 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L240 EN**: Executes statement `iostat, nullptr /* no unit */, sourceFile, sourceLine);`.
  **L240 CN**: 执行语句 `iostat, nullptr /* no unit */, sourceFile, sourceLine);`。
- **L241 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L241 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L242 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L242 CN**: 延续周围的声明、表达式或控制流结构。
- **L243 EN**: Introduces conditional control flow with an `if` statement.
  **L243 CN**: 通过 `if` 语句引入条件控制流。
- **L244 EN**: Initializes or updates `iostat`.
  **L244 CN**: 初始化或更新 `iostat`。
- **L245 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L245 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L246 EN**: Introduces conditional control flow with an `if` statement.
  **L246 CN**: 通过 `if` 语句引入条件控制流。
- **L247 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L247 CN**: 延续周围的声明、表达式或控制流结构。
- **L248 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L248 CN**: 延续周围的声明、表达式或控制流结构。
- **L249 EN**: Executes statement `terminator, *unit, sourceFile, sourceLine)};`.
  **L249 CN**: 执行语句 `terminator, *unit, sourceFile, sourceLine)};`。
- **L250 EN**: Introduces conditional control flow with an `if` statement.
  **L250 CN**: 通过 `if` 语句引入条件控制流。
- **L251 EN**: Introduces conditional control flow with an `if` statement.
  **L251 CN**: 通过 `if` 语句引入条件控制流。
- **L252 EN**: Comment documents intent or context: `Create space for (sub)record header to be completed by`.
  **L252 CN**: 注释记录了意图或上下文：`Create space for (sub)record header to be completed by`。
- **L253 EN**: Comment documents intent or context: `ExternalFileUnit::AdvanceRecord()`.
  **L253 CN**: 注释记录了意图或上下文：`ExternalFileUnit::AdvanceRecord()`。
- **L254 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L254 CN**: 延续周围的声明、表达式或控制流结构。
- **L255 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L255 CN**: 延续周围的声明、表达式或控制流结构。
- **L256 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L256 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 257-288

````cpp
      }
      return &io;
    } else {
      return &unit->BeginIoStatement<ErroneousIoStatementState>(
          terminator, iostat, unit, sourceFile, sourceLine);
    }
  }
}

Cookie IODEF(BeginUnformattedOutput)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  return BeginUnformattedIO<Direction::Output>(
      unitNumber, sourceFile, sourceLine);
}

Cookie IODEF(BeginUnformattedInput)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  return BeginUnformattedIO<Direction::Input>(
      unitNumber, sourceFile, sourceLine);
}

Cookie IODEF(BeginOpenUnit)( // OPEN(without NEWUNIT=)
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  bool wasExtant{false};
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUpOrCreate(
          unitNumber, terminator, wasExtant)}) {
    if (ChildIo * child{unit->GetChildIo()}) {
      return &child->BeginIoStatement<ErroneousIoStatementState>(
          IostatBadOpOnChildUnit, nullptr /* no unit */, sourceFile,
          sourceLine);
````

- **L257 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L257 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L258 EN**: Returns from the current function, often propagating a computed result.
  **L258 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L259 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L259 CN**: 延续周围的声明、表达式或控制流结构。
- **L260 EN**: Returns from the current function, often propagating a computed result.
  **L260 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L261 EN**: Executes statement `terminator, iostat, unit, sourceFile, sourceLine);`.
  **L261 CN**: 执行语句 `terminator, iostat, unit, sourceFile, sourceLine);`。
- **L262 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L262 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L263 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L263 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L264 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L264 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L265 EN**: Blank line separates nearby declarations or logic blocks.
  **L265 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L266 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L266 CN**: 延续周围的声明、表达式或控制流结构。
- **L267 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L267 CN**: 延续周围的声明、表达式或控制流结构。
- **L268 EN**: Returns from the current function, often propagating a computed result.
  **L268 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L269 EN**: Executes statement `unitNumber, sourceFile, sourceLine);`.
  **L269 CN**: 执行语句 `unitNumber, sourceFile, sourceLine);`。
- **L270 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L270 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L271 EN**: Blank line separates nearby declarations or logic blocks.
  **L271 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L272 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L272 CN**: 延续周围的声明、表达式或控制流结构。
- **L273 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L273 CN**: 延续周围的声明、表达式或控制流结构。
- **L274 EN**: Returns from the current function, often propagating a computed result.
  **L274 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L275 EN**: Executes statement `unitNumber, sourceFile, sourceLine);`.
  **L275 CN**: 执行语句 `unitNumber, sourceFile, sourceLine);`。
- **L276 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L276 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L277 EN**: Blank line separates nearby declarations or logic blocks.
  **L277 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L278 EN**: Declares or defines callable `IODEF`.
  **L278 CN**: 声明或定义可调用实体 `IODEF`。
- **L279 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L279 CN**: 延续周围的声明、表达式或控制流结构。
- **L280 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L280 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L281 EN**: Executes statement `bool wasExtant{false};`.
  **L281 CN**: 执行语句 `bool wasExtant{false};`。
- **L282 EN**: Introduces conditional control flow with an `if` statement.
  **L282 CN**: 通过 `if` 语句引入条件控制流。
- **L283 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L283 CN**: 延续周围的声明、表达式或控制流结构。
- **L284 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L284 CN**: 延续周围的声明、表达式或控制流结构。
- **L285 EN**: Introduces conditional control flow with an `if` statement.
  **L285 CN**: 通过 `if` 语句引入条件控制流。
- **L286 EN**: Returns from the current function, often propagating a computed result.
  **L286 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L287 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L287 CN**: 延续周围的声明、表达式或控制流结构。
- **L288 EN**: Executes statement `sourceLine);`.
  **L288 CN**: 执行语句 `sourceLine);`。

### Lines 289-320

````cpp
    } else {
      return &unit->BeginIoStatement<OpenStatementState>(terminator, *unit,
          wasExtant, false /*not NEWUNIT=*/, sourceFile, sourceLine);
    }
  } else {
    return NoopUnit(terminator, unitNumber, IostatBadUnitNumber);
  }
}

Cookie IODEF(BeginOpenNewUnit)( // OPEN(NEWUNIT=j)
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  ExternalFileUnit &unit{
      ExternalFileUnit::NewUnit(terminator, false /*not child I/O*/)};
  return &unit.BeginIoStatement<OpenStatementState>(terminator, unit,
      false /*was an existing file*/, true /*NEWUNIT=*/, sourceFile,
      sourceLine);
}

Cookie IODEF(BeginWait)(ExternalUnit unitNumber, AsynchronousId id,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUp(unitNumber, terminator)}) {
    if (unit->Wait(id)) {
      return &unit->BeginIoStatement<ExternalMiscIoStatementState>(terminator,
          *unit, ExternalMiscIoStatementState::Wait, sourceFile, sourceLine);
    } else {
      return &unit->BeginIoStatement<ErroneousIoStatementState>(
          terminator, IostatBadWaitId, unit, sourceFile, sourceLine);
    }
  } else {
````

- **L289 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L289 CN**: 延续周围的声明、表达式或控制流结构。
- **L290 EN**: Returns from the current function, often propagating a computed result.
  **L290 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L291 EN**: Executes statement `wasExtant, false /*not NEWUNIT=*/, sourceFile, sourceLine);`.
  **L291 CN**: 执行语句 `wasExtant, false /*not NEWUNIT=*/, sourceFile, sourceLine);`。
- **L292 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L292 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L293 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L293 CN**: 延续周围的声明、表达式或控制流结构。
- **L294 EN**: Returns from the current function, often propagating a computed result.
  **L294 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L295 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L295 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L296 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L296 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L297 EN**: Blank line separates nearby declarations or logic blocks.
  **L297 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L298 EN**: Declares or defines callable `IODEF`.
  **L298 CN**: 声明或定义可调用实体 `IODEF`。
- **L299 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L299 CN**: 延续周围的声明、表达式或控制流结构。
- **L300 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L300 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L301 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L301 CN**: 延续周围的声明、表达式或控制流结构。
- **L302 EN**: Executes statement involving `NewUnit`.
  **L302 CN**: 执行涉及 `NewUnit` 的语句。
- **L303 EN**: Returns from the current function, often propagating a computed result.
  **L303 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L304 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L304 CN**: 延续周围的声明、表达式或控制流结构。
- **L305 EN**: Executes statement `sourceLine);`.
  **L305 CN**: 执行语句 `sourceLine);`。
- **L306 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L306 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L307 EN**: Blank line separates nearby declarations or logic blocks.
  **L307 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L308 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L308 CN**: 延续周围的声明、表达式或控制流结构。
- **L309 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L309 CN**: 延续周围的声明、表达式或控制流结构。
- **L310 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L310 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L311 EN**: Introduces conditional control flow with an `if` statement.
  **L311 CN**: 通过 `if` 语句引入条件控制流。
- **L312 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L312 CN**: 延续周围的声明、表达式或控制流结构。
- **L313 EN**: Introduces conditional control flow with an `if` statement.
  **L313 CN**: 通过 `if` 语句引入条件控制流。
- **L314 EN**: Returns from the current function, often propagating a computed result.
  **L314 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L315 EN**: Comment documents intent or context: `unit, ExternalMiscIoStatementState::Wait, sourceFile, sourceLine);`.
  **L315 CN**: 注释记录了意图或上下文：`unit, ExternalMiscIoStatementState::Wait, sourceFile, sourceLine);`。
- **L316 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L316 CN**: 延续周围的声明、表达式或控制流结构。
- **L317 EN**: Returns from the current function, often propagating a computed result.
  **L317 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L318 EN**: Executes statement `terminator, IostatBadWaitId, unit, sourceFile, sourceLine);`.
  **L318 CN**: 执行语句 `terminator, IostatBadWaitId, unit, sourceFile, sourceLine);`。
- **L319 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L319 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L320 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L320 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 321-352

````cpp
    return NoopUnit(
        terminator, unitNumber, id == 0 ? IostatOk : IostatBadWaitUnit);
  }
}
Cookie IODEF(BeginWaitAll)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  return IONAME(BeginWait)(unitNumber, 0 /*no ID=*/, sourceFile, sourceLine);
}

Cookie IODEF(BeginClose)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUp(unitNumber, terminator)}) {
    if (ChildIo * child{unit->GetChildIo()}) {
      return &child->BeginIoStatement<ErroneousIoStatementState>(
          IostatBadOpOnChildUnit, nullptr /* no unit */, sourceFile,
          sourceLine);
    }
  }
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUpForClose(unitNumber, terminator)}) {
    return &unit->BeginIoStatement<CloseStatementState>(
        terminator, *unit, sourceFile, sourceLine);
  } else {
    // CLOSE(UNIT=bad unit) is just a no-op
    return NoopUnit(terminator, unitNumber);
  }
}

Cookie IODEF(BeginFlush)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
````

- **L321 EN**: Returns from the current function, often propagating a computed result.
  **L321 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L322 EN**: Executes statement `terminator, unitNumber, id == 0 ? IostatOk : IostatBadWaitUnit);`.
  **L322 CN**: 执行语句 `terminator, unitNumber, id == 0 ? IostatOk : IostatBadWaitUnit);`。
- **L323 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L323 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L324 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L324 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L325 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L325 CN**: 延续周围的声明、表达式或控制流结构。
- **L326 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L326 CN**: 延续周围的声明、表达式或控制流结构。
- **L327 EN**: Returns from the current function, often propagating a computed result.
  **L327 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L328 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L328 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L329 EN**: Blank line separates nearby declarations or logic blocks.
  **L329 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L330 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L330 CN**: 延续周围的声明、表达式或控制流结构。
- **L331 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L331 CN**: 延续周围的声明、表达式或控制流结构。
- **L332 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L332 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L333 EN**: Introduces conditional control flow with an `if` statement.
  **L333 CN**: 通过 `if` 语句引入条件控制流。
- **L334 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L334 CN**: 延续周围的声明、表达式或控制流结构。
- **L335 EN**: Introduces conditional control flow with an `if` statement.
  **L335 CN**: 通过 `if` 语句引入条件控制流。
- **L336 EN**: Returns from the current function, often propagating a computed result.
  **L336 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L337 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L337 CN**: 延续周围的声明、表达式或控制流结构。
- **L338 EN**: Executes statement `sourceLine);`.
  **L338 CN**: 执行语句 `sourceLine);`。
- **L339 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L339 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L340 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L340 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L341 EN**: Introduces conditional control flow with an `if` statement.
  **L341 CN**: 通过 `if` 语句引入条件控制流。
- **L342 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L342 CN**: 延续周围的声明、表达式或控制流结构。
- **L343 EN**: Returns from the current function, often propagating a computed result.
  **L343 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L344 EN**: Executes statement `terminator, *unit, sourceFile, sourceLine);`.
  **L344 CN**: 执行语句 `terminator, *unit, sourceFile, sourceLine);`。
- **L345 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L345 CN**: 延续周围的声明、表达式或控制流结构。
- **L346 EN**: Comment documents intent or context: `CLOSE(UNIT=bad unit) is just a no-op`.
  **L346 CN**: 注释记录了意图或上下文：`CLOSE(UNIT=bad unit) is just a no-op`。
- **L347 EN**: Returns from the current function, often propagating a computed result.
  **L347 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L348 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L348 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L349 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L349 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L350 EN**: Blank line separates nearby declarations or logic blocks.
  **L350 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L351 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L351 CN**: 延续周围的声明、表达式或控制流结构。
- **L352 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L352 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 353-384

````cpp
  Terminator terminator{sourceFile, sourceLine};
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUp(unitNumber, terminator)}) {
    if (ChildIo * child{unit->GetChildIo()}) {
      return &child->BeginIoStatement<ExternalMiscIoStatementState>(
          *unit, ExternalMiscIoStatementState::Flush, sourceFile, sourceLine);
    } else {
      return &unit->BeginIoStatement<ExternalMiscIoStatementState>(terminator,
          *unit, ExternalMiscIoStatementState::Flush, sourceFile, sourceLine);
    }
  } else {
    // FLUSH(UNIT=bad unit) is an error; an unconnected unit is a no-op
    return NoopUnit(terminator, unitNumber,
        unitNumber >= 0 ? IostatOk : IostatBadFlushUnit);
  }
}

Cookie IODEF(BeginBackspace)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUp(unitNumber, terminator)}) {
    if (ChildIo * child{unit->GetChildIo()}) {
      return &child->BeginIoStatement<ErroneousIoStatementState>(
          IostatBadOpOnChildUnit, nullptr /* no unit */, sourceFile,
          sourceLine);
    } else {
      return &unit->BeginIoStatement<ExternalMiscIoStatementState>(terminator,
          *unit, ExternalMiscIoStatementState::Backspace, sourceFile,
          sourceLine);
    }
  } else {
````

- **L353 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L353 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L354 EN**: Introduces conditional control flow with an `if` statement.
  **L354 CN**: 通过 `if` 语句引入条件控制流。
- **L355 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L355 CN**: 延续周围的声明、表达式或控制流结构。
- **L356 EN**: Introduces conditional control flow with an `if` statement.
  **L356 CN**: 通过 `if` 语句引入条件控制流。
- **L357 EN**: Returns from the current function, often propagating a computed result.
  **L357 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L358 EN**: Comment documents intent or context: `unit, ExternalMiscIoStatementState::Flush, sourceFile, sourceLine);`.
  **L358 CN**: 注释记录了意图或上下文：`unit, ExternalMiscIoStatementState::Flush, sourceFile, sourceLine);`。
- **L359 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L359 CN**: 延续周围的声明、表达式或控制流结构。
- **L360 EN**: Returns from the current function, often propagating a computed result.
  **L360 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L361 EN**: Comment documents intent or context: `unit, ExternalMiscIoStatementState::Flush, sourceFile, sourceLine);`.
  **L361 CN**: 注释记录了意图或上下文：`unit, ExternalMiscIoStatementState::Flush, sourceFile, sourceLine);`。
- **L362 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L362 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L363 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L363 CN**: 延续周围的声明、表达式或控制流结构。
- **L364 EN**: Comment documents intent or context: `FLUSH(UNIT=bad unit) is an error; an unconnected unit is a no-op`.
  **L364 CN**: 注释记录了意图或上下文：`FLUSH(UNIT=bad unit) is an error; an unconnected unit is a no-op`。
- **L365 EN**: Returns from the current function, often propagating a computed result.
  **L365 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L366 EN**: Executes statement `unitNumber >= 0 ? IostatOk : IostatBadFlushUnit);`.
  **L366 CN**: 执行语句 `unitNumber >= 0 ? IostatOk : IostatBadFlushUnit);`。
- **L367 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L367 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L368 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L368 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L369 EN**: Blank line separates nearby declarations or logic blocks.
  **L369 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L370 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L370 CN**: 延续周围的声明、表达式或控制流结构。
- **L371 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L371 CN**: 延续周围的声明、表达式或控制流结构。
- **L372 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L372 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L373 EN**: Introduces conditional control flow with an `if` statement.
  **L373 CN**: 通过 `if` 语句引入条件控制流。
- **L374 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L374 CN**: 延续周围的声明、表达式或控制流结构。
- **L375 EN**: Introduces conditional control flow with an `if` statement.
  **L375 CN**: 通过 `if` 语句引入条件控制流。
- **L376 EN**: Returns from the current function, often propagating a computed result.
  **L376 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L377 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L377 CN**: 延续周围的声明、表达式或控制流结构。
- **L378 EN**: Executes statement `sourceLine);`.
  **L378 CN**: 执行语句 `sourceLine);`。
- **L379 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L379 CN**: 延续周围的声明、表达式或控制流结构。
- **L380 EN**: Returns from the current function, often propagating a computed result.
  **L380 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L381 EN**: Comment documents intent or context: `unit, ExternalMiscIoStatementState::Backspace, sourceFile,`.
  **L381 CN**: 注释记录了意图或上下文：`unit, ExternalMiscIoStatementState::Backspace, sourceFile,`。
- **L382 EN**: Executes statement `sourceLine);`.
  **L382 CN**: 执行语句 `sourceLine);`。
- **L383 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L383 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L384 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L384 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 385-416

````cpp
    return NoopUnit(terminator, unitNumber, IostatBadBackspaceUnit);
  }
}

Cookie IODEF(BeginEndfile)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  Cookie errorCookie{nullptr};
  if (ExternalFileUnit *
      unit{GetOrCreateUnit(unitNumber, Direction::Output, common::nullopt,
          terminator, errorCookie)}) {
    if (ChildIo * child{unit->GetChildIo()}) {
      return &child->BeginIoStatement<ErroneousIoStatementState>(
          IostatBadOpOnChildUnit, nullptr /* no unit */, sourceFile,
          sourceLine);
    } else {
      return &unit->BeginIoStatement<ExternalMiscIoStatementState>(terminator,
          *unit, ExternalMiscIoStatementState::Endfile, sourceFile, sourceLine);
    }
  } else {
    return errorCookie;
  }
}

Cookie IODEF(BeginRewind)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  Cookie errorCookie{nullptr};
  if (ExternalFileUnit *
      unit{GetOrCreateUnit(unitNumber, Direction::Input, common::nullopt,
          terminator, errorCookie)}) {
    if (ChildIo * child{unit->GetChildIo()}) {
````

- **L385 EN**: Returns from the current function, often propagating a computed result.
  **L385 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L386 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L386 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L387 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L387 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L388 EN**: Blank line separates nearby declarations or logic blocks.
  **L388 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L389 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L389 CN**: 延续周围的声明、表达式或控制流结构。
- **L390 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L390 CN**: 延续周围的声明、表达式或控制流结构。
- **L391 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L391 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L392 EN**: Executes statement `Cookie errorCookie{nullptr};`.
  **L392 CN**: 执行语句 `Cookie errorCookie{nullptr};`。
- **L393 EN**: Introduces conditional control flow with an `if` statement.
  **L393 CN**: 通过 `if` 语句引入条件控制流。
- **L394 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L394 CN**: 延续周围的声明、表达式或控制流结构。
- **L395 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L395 CN**: 延续周围的声明、表达式或控制流结构。
- **L396 EN**: Introduces conditional control flow with an `if` statement.
  **L396 CN**: 通过 `if` 语句引入条件控制流。
- **L397 EN**: Returns from the current function, often propagating a computed result.
  **L397 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L398 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L398 CN**: 延续周围的声明、表达式或控制流结构。
- **L399 EN**: Executes statement `sourceLine);`.
  **L399 CN**: 执行语句 `sourceLine);`。
- **L400 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L400 CN**: 延续周围的声明、表达式或控制流结构。
- **L401 EN**: Returns from the current function, often propagating a computed result.
  **L401 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L402 EN**: Comment documents intent or context: `unit, ExternalMiscIoStatementState::Endfile, sourceFile, sourceLine);`.
  **L402 CN**: 注释记录了意图或上下文：`unit, ExternalMiscIoStatementState::Endfile, sourceFile, sourceLine);`。
- **L403 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L403 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L404 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L404 CN**: 延续周围的声明、表达式或控制流结构。
- **L405 EN**: Returns from the current function, often propagating a computed result.
  **L405 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L406 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L406 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L407 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L407 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L408 EN**: Blank line separates nearby declarations or logic blocks.
  **L408 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L409 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L409 CN**: 延续周围的声明、表达式或控制流结构。
- **L410 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L410 CN**: 延续周围的声明、表达式或控制流结构。
- **L411 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L411 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L412 EN**: Executes statement `Cookie errorCookie{nullptr};`.
  **L412 CN**: 执行语句 `Cookie errorCookie{nullptr};`。
- **L413 EN**: Introduces conditional control flow with an `if` statement.
  **L413 CN**: 通过 `if` 语句引入条件控制流。
- **L414 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L414 CN**: 延续周围的声明、表达式或控制流结构。
- **L415 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L415 CN**: 延续周围的声明、表达式或控制流结构。
- **L416 EN**: Introduces conditional control flow with an `if` statement.
  **L416 CN**: 通过 `if` 语句引入条件控制流。

### Lines 417-448

````cpp
      return &child->BeginIoStatement<ErroneousIoStatementState>(
          IostatBadOpOnChildUnit, nullptr /* no unit */, sourceFile,
          sourceLine);
    } else {
      return &unit->BeginIoStatement<ExternalMiscIoStatementState>(terminator,
          *unit, ExternalMiscIoStatementState::Rewind, sourceFile, sourceLine);
    }
  } else {
    return errorCookie;
  }
}

Cookie IODEF(BeginInquireUnit)(
    ExternalUnit unitNumber, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUp(unitNumber, terminator)}) {
    if (ChildIo * child{unit->GetChildIo()}) {
      return &child->BeginIoStatement<InquireUnitState>(
          *unit, sourceFile, sourceLine);
    } else {
      return &unit->BeginIoStatement<InquireUnitState>(
          terminator, *unit, sourceFile, sourceLine);
    }
  } else {
    // INQUIRE(UNIT=unrecognized unit)
    return &New<InquireNoUnitState>{terminator}(
        sourceFile, sourceLine, unitNumber)
                .release()
                ->ioStatementState();
  }
}
````

- **L417 EN**: Returns from the current function, often propagating a computed result.
  **L417 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L418 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L418 CN**: 延续周围的声明、表达式或控制流结构。
- **L419 EN**: Executes statement `sourceLine);`.
  **L419 CN**: 执行语句 `sourceLine);`。
- **L420 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L420 CN**: 延续周围的声明、表达式或控制流结构。
- **L421 EN**: Returns from the current function, often propagating a computed result.
  **L421 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L422 EN**: Comment documents intent or context: `unit, ExternalMiscIoStatementState::Rewind, sourceFile, sourceLine);`.
  **L422 CN**: 注释记录了意图或上下文：`unit, ExternalMiscIoStatementState::Rewind, sourceFile, sourceLine);`。
- **L423 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L423 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L424 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L424 CN**: 延续周围的声明、表达式或控制流结构。
- **L425 EN**: Returns from the current function, often propagating a computed result.
  **L425 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L426 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L426 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L427 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L427 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L428 EN**: Blank line separates nearby declarations or logic blocks.
  **L428 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L429 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L429 CN**: 延续周围的声明、表达式或控制流结构。
- **L430 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L430 CN**: 延续周围的声明、表达式或控制流结构。
- **L431 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L431 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L432 EN**: Introduces conditional control flow with an `if` statement.
  **L432 CN**: 通过 `if` 语句引入条件控制流。
- **L433 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L433 CN**: 延续周围的声明、表达式或控制流结构。
- **L434 EN**: Introduces conditional control flow with an `if` statement.
  **L434 CN**: 通过 `if` 语句引入条件控制流。
- **L435 EN**: Returns from the current function, often propagating a computed result.
  **L435 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L436 EN**: Comment documents intent or context: `unit, sourceFile, sourceLine);`.
  **L436 CN**: 注释记录了意图或上下文：`unit, sourceFile, sourceLine);`。
- **L437 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L437 CN**: 延续周围的声明、表达式或控制流结构。
- **L438 EN**: Returns from the current function, often propagating a computed result.
  **L438 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L439 EN**: Executes statement `terminator, *unit, sourceFile, sourceLine);`.
  **L439 CN**: 执行语句 `terminator, *unit, sourceFile, sourceLine);`。
- **L440 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L440 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L441 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L441 CN**: 延续周围的声明、表达式或控制流结构。
- **L442 EN**: Comment documents intent or context: `INQUIRE(UNIT=unrecognized unit)`.
  **L442 CN**: 注释记录了意图或上下文：`INQUIRE(UNIT=unrecognized unit)`。
- **L443 EN**: Returns from the current function, often propagating a computed result.
  **L443 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L444 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L444 CN**: 延续周围的声明、表达式或控制流结构。
- **L445 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L445 CN**: 延续周围的声明、表达式或控制流结构。
- **L446 EN**: Executes statement involving `ioStatementState`.
  **L446 CN**: 执行涉及 `ioStatementState` 的语句。
- **L447 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L447 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L448 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L448 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 449-480

````cpp

Cookie IODEF(BeginInquireFile)(const char *path, std::size_t pathLength,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  auto trimmed{SaveDefaultCharacter(
      path, TrimTrailingSpaces(path, pathLength), terminator)};
  if (ExternalFileUnit *
      unit{ExternalFileUnit::LookUp(trimmed.get(),
          Fortran::runtime::strlen(trimmed.get()), terminator)}) {
    // INQUIRE(FILE=) to a connected unit
    if (ChildIo * child{unit->GetChildIo()}) {
      return &child->BeginIoStatement<InquireUnitState>(
          *unit, sourceFile, sourceLine);
    } else {
      return &unit->BeginIoStatement<InquireUnitState>(
          terminator, *unit, sourceFile, sourceLine);
    }
  } else {
    return &New<InquireUnconnectedFileState>{terminator}(
        std::move(trimmed), sourceFile, sourceLine)
                .release()
                ->ioStatementState();
  }
}

Cookie IODEF(BeginInquireIoLength)(const char *sourceFile, int sourceLine) {
  Terminator oom{sourceFile, sourceLine};
  return &New<InquireIOLengthState>{oom}(sourceFile, sourceLine)
              .release()
              ->ioStatementState();
}

````

- **L449 EN**: Blank line separates nearby declarations or logic blocks.
  **L449 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L450 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L450 CN**: 延续周围的声明、表达式或控制流结构。
- **L451 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L451 CN**: 延续周围的声明、表达式或控制流结构。
- **L452 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L452 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L453 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L453 CN**: 延续周围的声明、表达式或控制流结构。
- **L454 EN**: Executes statement involving `TrimTrailingSpaces`.
  **L454 CN**: 执行涉及 `TrimTrailingSpaces` 的语句。
- **L455 EN**: Introduces conditional control flow with an `if` statement.
  **L455 CN**: 通过 `if` 语句引入条件控制流。
- **L456 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L456 CN**: 延续周围的声明、表达式或控制流结构。
- **L457 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L457 CN**: 延续周围的声明、表达式或控制流结构。
- **L458 EN**: Comment documents intent or context: `INQUIRE(FILE=) to a connected unit`.
  **L458 CN**: 注释记录了意图或上下文：`INQUIRE(FILE=) to a connected unit`。
- **L459 EN**: Introduces conditional control flow with an `if` statement.
  **L459 CN**: 通过 `if` 语句引入条件控制流。
- **L460 EN**: Returns from the current function, often propagating a computed result.
  **L460 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L461 EN**: Comment documents intent or context: `unit, sourceFile, sourceLine);`.
  **L461 CN**: 注释记录了意图或上下文：`unit, sourceFile, sourceLine);`。
- **L462 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L462 CN**: 延续周围的声明、表达式或控制流结构。
- **L463 EN**: Returns from the current function, often propagating a computed result.
  **L463 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L464 EN**: Executes statement `terminator, *unit, sourceFile, sourceLine);`.
  **L464 CN**: 执行语句 `terminator, *unit, sourceFile, sourceLine);`。
- **L465 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L465 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L466 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L466 CN**: 延续周围的声明、表达式或控制流结构。
- **L467 EN**: Returns from the current function, often propagating a computed result.
  **L467 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L468 EN**: Declares or defines callable `move`.
  **L468 CN**: 声明或定义可调用实体 `move`。
- **L469 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L469 CN**: 延续周围的声明、表达式或控制流结构。
- **L470 EN**: Executes statement involving `ioStatementState`.
  **L470 CN**: 执行涉及 `ioStatementState` 的语句。
- **L471 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L471 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L472 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L472 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L473 EN**: Blank line separates nearby declarations or logic blocks.
  **L473 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L474 EN**: Declares or defines callable `IODEF`.
  **L474 CN**: 声明或定义可调用实体 `IODEF`。
- **L475 EN**: Executes statement `Terminator oom{sourceFile, sourceLine};`.
  **L475 CN**: 执行语句 `Terminator oom{sourceFile, sourceLine};`。
- **L476 EN**: Returns from the current function, often propagating a computed result.
  **L476 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L477 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L477 CN**: 延续周围的声明、表达式或控制流结构。
- **L478 EN**: Executes statement involving `ioStatementState`.
  **L478 CN**: 执行涉及 `ioStatementState` 的语句。
- **L479 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L479 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L480 EN**: Blank line separates nearby declarations or logic blocks.
  **L480 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 481-512

````cpp
// Control list items

void IODEF(EnableHandlers)(Cookie cookie, bool hasIoStat, bool hasErr,
    bool hasEnd, bool hasEor, bool hasIoMsg) {
  IoErrorHandler &handler{cookie->GetIoErrorHandler()};
  if (hasIoStat) {
    handler.HasIoStat();
  }
  if (hasErr) {
    handler.HasErrLabel();
  }
  if (hasEnd) {
    handler.HasEndLabel();
  }
  if (hasEor) {
    handler.HasEorLabel();
  }
  if (hasIoMsg) {
    handler.HasIoMsg();
  }
}

static RT_API_ATTRS bool YesOrNo(const char *keyword, std::size_t length,
    const char *what, IoErrorHandler &handler) {
  static const char *keywords[]{"YES", "NO", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    return true;
  case 1:
    return false;
  default:
    handler.SignalError(IostatErrorInKeyword, "Invalid %s='%.*s'", what,
````

- **L481 EN**: Comment documents intent or context: `Control list items`.
  **L481 CN**: 注释记录了意图或上下文：`Control list items`。
- **L482 EN**: Blank line separates nearby declarations or logic blocks.
  **L482 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L483 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L483 CN**: 延续周围的声明、表达式或控制流结构。
- **L484 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L484 CN**: 延续周围的声明、表达式或控制流结构。
- **L485 EN**: Executes statement involving `GetIoErrorHandler`.
  **L485 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L486 EN**: Introduces conditional control flow with an `if` statement.
  **L486 CN**: 通过 `if` 语句引入条件控制流。
- **L487 EN**: Executes statement involving `HasIoStat`.
  **L487 CN**: 执行涉及 `HasIoStat` 的语句。
- **L488 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L488 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L489 EN**: Introduces conditional control flow with an `if` statement.
  **L489 CN**: 通过 `if` 语句引入条件控制流。
- **L490 EN**: Executes statement involving `HasErrLabel`.
  **L490 CN**: 执行涉及 `HasErrLabel` 的语句。
- **L491 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L491 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L492 EN**: Introduces conditional control flow with an `if` statement.
  **L492 CN**: 通过 `if` 语句引入条件控制流。
- **L493 EN**: Executes statement involving `HasEndLabel`.
  **L493 CN**: 执行涉及 `HasEndLabel` 的语句。
- **L494 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L494 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L495 EN**: Introduces conditional control flow with an `if` statement.
  **L495 CN**: 通过 `if` 语句引入条件控制流。
- **L496 EN**: Executes statement involving `HasEorLabel`.
  **L496 CN**: 执行涉及 `HasEorLabel` 的语句。
- **L497 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L497 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L498 EN**: Introduces conditional control flow with an `if` statement.
  **L498 CN**: 通过 `if` 语句引入条件控制流。
- **L499 EN**: Executes statement involving `HasIoMsg`.
  **L499 CN**: 执行涉及 `HasIoMsg` 的语句。
- **L500 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L500 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L501 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L501 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L502 EN**: Blank line separates nearby declarations or logic blocks.
  **L502 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L503 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L503 CN**: 延续周围的声明、表达式或控制流结构。
- **L504 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L504 CN**: 延续周围的声明、表达式或控制流结构。
- **L505 EN**: Executes statement `static const char *keywords[]{"YES", "NO", nullptr};`.
  **L505 CN**: 执行语句 `static const char *keywords[]{"YES", "NO", nullptr};`。
- **L506 EN**: Begins a `switch` dispatch over discrete cases.
  **L506 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L507 EN**: Marks one `switch` case label.
  **L507 CN**: 标记一个 `switch` 的 case 标签。
- **L508 EN**: Returns from the current function, often propagating a computed result.
  **L508 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L509 EN**: Marks one `switch` case label.
  **L509 CN**: 标记一个 `switch` 的 case 标签。
- **L510 EN**: Returns from the current function, often propagating a computed result.
  **L510 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L511 EN**: Provides the default branch for a `switch` statement.
  **L511 CN**: 为 `switch` 语句提供默认分支。
- **L512 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L512 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 513-544

````cpp
        static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetAdvance)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  bool nonAdvancing{!YesOrNo(keyword, length, "ADVANCE", handler)};
  if (nonAdvancing && io.GetConnectionState().access == Access::Direct) {
    handler.SignalError("Non-advancing I/O attempted on direct access file");
  } else {
    auto *unit{io.GetExternalFileUnit()};
    if (unit && unit->GetChildIo()) {
      // ADVANCE= is ignored for child I/O (12.6.4.8.3 p3)
    } else {
      io.mutableModes().nonAdvancing = nonAdvancing;
    }
  }
  return !handler.InError();
}

bool IODEF(SetBlank)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    open->set_mustBeFormatted();
  }
  static const char *keywords[]{"NULL", "ZERO", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    io.mutableModes().editingFlags &= ~blankZero;
    return true;
````

- **L513 EN**: Executes statement `static_cast<int>(length), keyword);`.
  **L513 CN**: 执行语句 `static_cast<int>(length), keyword);`。
- **L514 EN**: Returns from the current function, often propagating a computed result.
  **L514 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L515 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L515 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L516 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L516 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L517 EN**: Blank line separates nearby declarations or logic blocks.
  **L517 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L518 EN**: Declares or defines callable `IODEF`.
  **L518 CN**: 声明或定义可调用实体 `IODEF`。
- **L519 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L519 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L520 EN**: Executes statement involving `GetIoErrorHandler`.
  **L520 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L521 EN**: Executes statement involving `YesOrNo`.
  **L521 CN**: 执行涉及 `YesOrNo` 的语句。
- **L522 EN**: Introduces conditional control flow with an `if` statement.
  **L522 CN**: 通过 `if` 语句引入条件控制流。
- **L523 EN**: Executes statement involving `SignalError`.
  **L523 CN**: 执行涉及 `SignalError` 的语句。
- **L524 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L524 CN**: 延续周围的声明、表达式或控制流结构。
- **L525 EN**: Executes statement involving `GetExternalFileUnit`.
  **L525 CN**: 执行涉及 `GetExternalFileUnit` 的语句。
- **L526 EN**: Introduces conditional control flow with an `if` statement.
  **L526 CN**: 通过 `if` 语句引入条件控制流。
- **L527 EN**: Comment documents intent or context: `ADVANCE= is ignored for child I/O (12.6.4.8.3 p3)`.
  **L527 CN**: 注释记录了意图或上下文：`ADVANCE= is ignored for child I/O (12.6.4.8.3 p3)`。
- **L528 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L528 CN**: 延续周围的声明、表达式或控制流结构。
- **L529 EN**: Initializes or updates `io.mutableModes().nonAdvancing`.
  **L529 CN**: 初始化或更新 `io.mutableModes().nonAdvancing`。
- **L530 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L530 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L531 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L531 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L532 EN**: Returns from the current function, often propagating a computed result.
  **L532 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L533 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L533 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L534 EN**: Blank line separates nearby declarations or logic blocks.
  **L534 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L535 EN**: Declares or defines callable `IODEF`.
  **L535 CN**: 声明或定义可调用实体 `IODEF`。
- **L536 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L536 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L537 EN**: Introduces conditional control flow with an `if` statement.
  **L537 CN**: 通过 `if` 语句引入条件控制流。
- **L538 EN**: Executes statement involving `set_mustBeFormatted`.
  **L538 CN**: 执行涉及 `set_mustBeFormatted` 的语句。
- **L539 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L539 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L540 EN**: Executes statement `static const char *keywords[]{"NULL", "ZERO", nullptr};`.
  **L540 CN**: 执行语句 `static const char *keywords[]{"NULL", "ZERO", nullptr};`。
- **L541 EN**: Begins a `switch` dispatch over discrete cases.
  **L541 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L542 EN**: Marks one `switch` case label.
  **L542 CN**: 标记一个 `switch` 的 case 标签。
- **L543 EN**: Initializes or updates `&`.
  **L543 CN**: 初始化或更新 `&`。
- **L544 EN**: Returns from the current function, often propagating a computed result.
  **L544 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 545-576

````cpp
  case 1:
    io.mutableModes().editingFlags |= blankZero;
    return true;
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
        "Invalid BLANK='%.*s'", static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetDecimal)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    open->set_mustBeFormatted();
  }
  static const char *keywords[]{"COMMA", "POINT", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    io.mutableModes().editingFlags |= decimalComma;
    return true;
  case 1:
    io.mutableModes().editingFlags &= ~decimalComma;
    return true;
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
        "Invalid DECIMAL='%.*s'", static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetDelim)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
````

- **L545 EN**: Marks one `switch` case label.
  **L545 CN**: 标记一个 `switch` 的 case 标签。
- **L546 EN**: Initializes or updates `|`.
  **L546 CN**: 初始化或更新 `|`。
- **L547 EN**: Returns from the current function, often propagating a computed result.
  **L547 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L548 EN**: Provides the default branch for a `switch` statement.
  **L548 CN**: 为 `switch` 语句提供默认分支。
- **L549 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L549 CN**: 延续周围的声明、表达式或控制流结构。
- **L550 EN**: Executes statement `"Invalid BLANK='%.*s'", static_cast<int>(length), keyword);`.
  **L550 CN**: 执行语句 `"Invalid BLANK='%.*s'", static_cast<int>(length), keyword);`。
- **L551 EN**: Returns from the current function, often propagating a computed result.
  **L551 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L552 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L552 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L553 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L553 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L554 EN**: Blank line separates nearby declarations or logic blocks.
  **L554 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L555 EN**: Declares or defines callable `IODEF`.
  **L555 CN**: 声明或定义可调用实体 `IODEF`。
- **L556 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L556 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L557 EN**: Introduces conditional control flow with an `if` statement.
  **L557 CN**: 通过 `if` 语句引入条件控制流。
- **L558 EN**: Executes statement involving `set_mustBeFormatted`.
  **L558 CN**: 执行涉及 `set_mustBeFormatted` 的语句。
- **L559 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L559 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L560 EN**: Executes statement `static const char *keywords[]{"COMMA", "POINT", nullptr};`.
  **L560 CN**: 执行语句 `static const char *keywords[]{"COMMA", "POINT", nullptr};`。
- **L561 EN**: Begins a `switch` dispatch over discrete cases.
  **L561 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L562 EN**: Marks one `switch` case label.
  **L562 CN**: 标记一个 `switch` 的 case 标签。
- **L563 EN**: Initializes or updates `|`.
  **L563 CN**: 初始化或更新 `|`。
- **L564 EN**: Returns from the current function, often propagating a computed result.
  **L564 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L565 EN**: Marks one `switch` case label.
  **L565 CN**: 标记一个 `switch` 的 case 标签。
- **L566 EN**: Initializes or updates `&`.
  **L566 CN**: 初始化或更新 `&`。
- **L567 EN**: Returns from the current function, often propagating a computed result.
  **L567 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L568 EN**: Provides the default branch for a `switch` statement.
  **L568 CN**: 为 `switch` 语句提供默认分支。
- **L569 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L569 CN**: 延续周围的声明、表达式或控制流结构。
- **L570 EN**: Executes statement `"Invalid DECIMAL='%.*s'", static_cast<int>(length), keyword);`.
  **L570 CN**: 执行语句 `"Invalid DECIMAL='%.*s'", static_cast<int>(length), keyword);`。
- **L571 EN**: Returns from the current function, often propagating a computed result.
  **L571 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L572 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L572 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L573 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L573 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L574 EN**: Blank line separates nearby declarations or logic blocks.
  **L574 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L575 EN**: Declares or defines callable `IODEF`.
  **L575 CN**: 声明或定义可调用实体 `IODEF`。
- **L576 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L576 CN**: 执行语句 `IoStatementState &io{*cookie};`。

### Lines 577-608

````cpp
  if (auto *open{io.get_if<OpenStatementState>()}) {
    open->set_mustBeFormatted();
  }
  static const char *keywords[]{"APOSTROPHE", "QUOTE", "NONE", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    io.mutableModes().delim = '\'';
    return true;
  case 1:
    io.mutableModes().delim = '"';
    return true;
  case 2:
    io.mutableModes().delim = '\0';
    return true;
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
        "Invalid DELIM='%.*s'", static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetPad)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    open->set_mustBeFormatted();
  }
  io.mutableModes().pad = YesOrNo(keyword, length, "PAD", handler);
  return !handler.InError();
}

bool IODEF(SetPos)(Cookie cookie, std::int64_t pos) {
````

- **L577 EN**: Introduces conditional control flow with an `if` statement.
  **L577 CN**: 通过 `if` 语句引入条件控制流。
- **L578 EN**: Executes statement involving `set_mustBeFormatted`.
  **L578 CN**: 执行涉及 `set_mustBeFormatted` 的语句。
- **L579 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L579 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L580 EN**: Executes statement `static const char *keywords[]{"APOSTROPHE", "QUOTE", "NONE", nullptr};`.
  **L580 CN**: 执行语句 `static const char *keywords[]{"APOSTROPHE", "QUOTE", "NONE", nullptr};`。
- **L581 EN**: Begins a `switch` dispatch over discrete cases.
  **L581 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L582 EN**: Marks one `switch` case label.
  **L582 CN**: 标记一个 `switch` 的 case 标签。
- **L583 EN**: Initializes or updates `io.mutableModes().delim`.
  **L583 CN**: 初始化或更新 `io.mutableModes().delim`。
- **L584 EN**: Returns from the current function, often propagating a computed result.
  **L584 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L585 EN**: Marks one `switch` case label.
  **L585 CN**: 标记一个 `switch` 的 case 标签。
- **L586 EN**: Initializes or updates `io.mutableModes().delim`.
  **L586 CN**: 初始化或更新 `io.mutableModes().delim`。
- **L587 EN**: Returns from the current function, often propagating a computed result.
  **L587 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L588 EN**: Marks one `switch` case label.
  **L588 CN**: 标记一个 `switch` 的 case 标签。
- **L589 EN**: Initializes or updates `io.mutableModes().delim`.
  **L589 CN**: 初始化或更新 `io.mutableModes().delim`。
- **L590 EN**: Returns from the current function, often propagating a computed result.
  **L590 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L591 EN**: Provides the default branch for a `switch` statement.
  **L591 CN**: 为 `switch` 语句提供默认分支。
- **L592 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L592 CN**: 延续周围的声明、表达式或控制流结构。
- **L593 EN**: Executes statement `"Invalid DELIM='%.*s'", static_cast<int>(length), keyword);`.
  **L593 CN**: 执行语句 `"Invalid DELIM='%.*s'", static_cast<int>(length), keyword);`。
- **L594 EN**: Returns from the current function, often propagating a computed result.
  **L594 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L595 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L595 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L596 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L596 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L597 EN**: Blank line separates nearby declarations or logic blocks.
  **L597 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L598 EN**: Declares or defines callable `IODEF`.
  **L598 CN**: 声明或定义可调用实体 `IODEF`。
- **L599 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L599 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L600 EN**: Executes statement involving `GetIoErrorHandler`.
  **L600 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L601 EN**: Introduces conditional control flow with an `if` statement.
  **L601 CN**: 通过 `if` 语句引入条件控制流。
- **L602 EN**: Executes statement involving `set_mustBeFormatted`.
  **L602 CN**: 执行涉及 `set_mustBeFormatted` 的语句。
- **L603 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L603 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L604 EN**: Initializes or updates `io.mutableModes().pad`.
  **L604 CN**: 初始化或更新 `io.mutableModes().pad`。
- **L605 EN**: Returns from the current function, often propagating a computed result.
  **L605 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L606 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L606 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L607 EN**: Blank line separates nearby declarations or logic blocks.
  **L607 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L608 EN**: Declares or defines callable `IODEF`.
  **L608 CN**: 声明或定义可调用实体 `IODEF`。

### Lines 609-640

````cpp
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  if (auto *unit{io.GetExternalFileUnit()}) {
    return unit->SetStreamPos(pos, handler);
  } else if (!io.get_if<ErroneousIoStatementState>()) {
    handler.Crash("SetPos() called on internal unit");
  }
  return false;
}

bool IODEF(SetRec)(Cookie cookie, std::int64_t rec) {
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  if (auto *unit{io.GetExternalFileUnit()}) {
    if (unit->GetChildIo()) {
      handler.SignalError(
          IostatBadOpOnChildUnit, "REC= specifier on child I/O");
    } else {
      handler.HasRec();
      unit->SetDirectRec(rec, handler);
    }
  } else if (!io.get_if<ErroneousIoStatementState>()) {
    handler.Crash("SetRec() called on internal unit");
  }
  return true;
}

bool IODEF(SetRound)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    open->set_mustBeFormatted();
  }
````

- **L609 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L609 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L610 EN**: Executes statement involving `GetIoErrorHandler`.
  **L610 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L611 EN**: Introduces conditional control flow with an `if` statement.
  **L611 CN**: 通过 `if` 语句引入条件控制流。
- **L612 EN**: Returns from the current function, often propagating a computed result.
  **L612 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L613 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L613 CN**: 延续周围的声明、表达式或控制流结构。
- **L614 EN**: Executes statement involving `Crash`.
  **L614 CN**: 执行涉及 `Crash` 的语句。
- **L615 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L615 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L616 EN**: Returns from the current function, often propagating a computed result.
  **L616 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L617 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L617 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L618 EN**: Blank line separates nearby declarations or logic blocks.
  **L618 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L619 EN**: Declares or defines callable `IODEF`.
  **L619 CN**: 声明或定义可调用实体 `IODEF`。
- **L620 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L620 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L621 EN**: Executes statement involving `GetIoErrorHandler`.
  **L621 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L622 EN**: Introduces conditional control flow with an `if` statement.
  **L622 CN**: 通过 `if` 语句引入条件控制流。
- **L623 EN**: Introduces conditional control flow with an `if` statement.
  **L623 CN**: 通过 `if` 语句引入条件控制流。
- **L624 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L624 CN**: 延续周围的声明、表达式或控制流结构。
- **L625 EN**: Executes statement `IostatBadOpOnChildUnit, "REC= specifier on child I/O");`.
  **L625 CN**: 执行语句 `IostatBadOpOnChildUnit, "REC= specifier on child I/O");`。
- **L626 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L626 CN**: 延续周围的声明、表达式或控制流结构。
- **L627 EN**: Executes statement involving `HasRec`.
  **L627 CN**: 执行涉及 `HasRec` 的语句。
- **L628 EN**: Executes statement involving `SetDirectRec`.
  **L628 CN**: 执行涉及 `SetDirectRec` 的语句。
- **L629 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L629 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L630 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L630 CN**: 延续周围的声明、表达式或控制流结构。
- **L631 EN**: Executes statement involving `Crash`.
  **L631 CN**: 执行涉及 `Crash` 的语句。
- **L632 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L632 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L633 EN**: Returns from the current function, often propagating a computed result.
  **L633 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L634 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L634 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L635 EN**: Blank line separates nearby declarations or logic blocks.
  **L635 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L636 EN**: Declares or defines callable `IODEF`.
  **L636 CN**: 声明或定义可调用实体 `IODEF`。
- **L637 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L637 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L638 EN**: Introduces conditional control flow with an `if` statement.
  **L638 CN**: 通过 `if` 语句引入条件控制流。
- **L639 EN**: Executes statement involving `set_mustBeFormatted`.
  **L639 CN**: 执行涉及 `set_mustBeFormatted` 的语句。
- **L640 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L640 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 641-672

````cpp
  static const char *keywords[]{"UP", "DOWN", "ZERO", "NEAREST", "COMPATIBLE",
      "PROCESSOR_DEFINED", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    io.mutableModes().round = decimal::RoundUp;
    return true;
  case 1:
    io.mutableModes().round = decimal::RoundDown;
    return true;
  case 2:
    io.mutableModes().round = decimal::RoundToZero;
    return true;
  case 3:
    io.mutableModes().round = decimal::RoundNearest;
    return true;
  case 4:
    io.mutableModes().round = decimal::RoundCompatible;
    return true;
  case 5:
    io.mutableModes().round = executionEnvironment.defaultOutputRoundingMode;
    return true;
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
        "Invalid ROUND='%.*s'", static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetSign)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    open->set_mustBeFormatted();
````

- **L641 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L641 CN**: 延续周围的声明、表达式或控制流结构。
- **L642 EN**: Executes statement `"PROCESSOR_DEFINED", nullptr};`.
  **L642 CN**: 执行语句 `"PROCESSOR_DEFINED", nullptr};`。
- **L643 EN**: Begins a `switch` dispatch over discrete cases.
  **L643 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L644 EN**: Marks one `switch` case label.
  **L644 CN**: 标记一个 `switch` 的 case 标签。
- **L645 EN**: Initializes or updates `io.mutableModes().round`.
  **L645 CN**: 初始化或更新 `io.mutableModes().round`。
- **L646 EN**: Returns from the current function, often propagating a computed result.
  **L646 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L647 EN**: Marks one `switch` case label.
  **L647 CN**: 标记一个 `switch` 的 case 标签。
- **L648 EN**: Initializes or updates `io.mutableModes().round`.
  **L648 CN**: 初始化或更新 `io.mutableModes().round`。
- **L649 EN**: Returns from the current function, often propagating a computed result.
  **L649 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L650 EN**: Marks one `switch` case label.
  **L650 CN**: 标记一个 `switch` 的 case 标签。
- **L651 EN**: Initializes or updates `io.mutableModes().round`.
  **L651 CN**: 初始化或更新 `io.mutableModes().round`。
- **L652 EN**: Returns from the current function, often propagating a computed result.
  **L652 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L653 EN**: Marks one `switch` case label.
  **L653 CN**: 标记一个 `switch` 的 case 标签。
- **L654 EN**: Initializes or updates `io.mutableModes().round`.
  **L654 CN**: 初始化或更新 `io.mutableModes().round`。
- **L655 EN**: Returns from the current function, often propagating a computed result.
  **L655 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L656 EN**: Marks one `switch` case label.
  **L656 CN**: 标记一个 `switch` 的 case 标签。
- **L657 EN**: Initializes or updates `io.mutableModes().round`.
  **L657 CN**: 初始化或更新 `io.mutableModes().round`。
- **L658 EN**: Returns from the current function, often propagating a computed result.
  **L658 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L659 EN**: Marks one `switch` case label.
  **L659 CN**: 标记一个 `switch` 的 case 标签。
- **L660 EN**: Initializes or updates `io.mutableModes().round`.
  **L660 CN**: 初始化或更新 `io.mutableModes().round`。
- **L661 EN**: Returns from the current function, often propagating a computed result.
  **L661 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L662 EN**: Provides the default branch for a `switch` statement.
  **L662 CN**: 为 `switch` 语句提供默认分支。
- **L663 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L663 CN**: 延续周围的声明、表达式或控制流结构。
- **L664 EN**: Executes statement `"Invalid ROUND='%.*s'", static_cast<int>(length), keyword);`.
  **L664 CN**: 执行语句 `"Invalid ROUND='%.*s'", static_cast<int>(length), keyword);`。
- **L665 EN**: Returns from the current function, often propagating a computed result.
  **L665 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L666 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L666 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L667 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L667 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L668 EN**: Blank line separates nearby declarations or logic blocks.
  **L668 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L669 EN**: Declares or defines callable `IODEF`.
  **L669 CN**: 声明或定义可调用实体 `IODEF`。
- **L670 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L670 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L671 EN**: Introduces conditional control flow with an `if` statement.
  **L671 CN**: 通过 `if` 语句引入条件控制流。
- **L672 EN**: Executes statement involving `set_mustBeFormatted`.
  **L672 CN**: 执行涉及 `set_mustBeFormatted` 的语句。

### Lines 673-704

````cpp
  }
  static const char *keywords[]{
      "PLUS", "SUPPRESS", "PROCESSOR_DEFINED", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    io.mutableModes().editingFlags |= signPlus;
    return true;
  case 1:
  case 2: // processor default is SS
    io.mutableModes().editingFlags &= ~signPlus;
    return true;
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
        "Invalid SIGN='%.*s'", static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetLeadingZero)(
    Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    open->set_mustBeFormatted();
  }
  static const char *keywords[]{
      "PRINT", "PROCESSOR_DEFINED", "SUPPRESS", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0: // LZP, print leading zero, if the field has room for it
  case 1: // LZ, processor default, treated as LZP
    io.mutableModes().editingFlags &= ~leadingZeroSuppress;
    return true;
  case 2:
````

- **L673 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L673 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L674 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L674 CN**: 延续周围的声明、表达式或控制流结构。
- **L675 EN**: Executes statement `"PLUS", "SUPPRESS", "PROCESSOR_DEFINED", nullptr};`.
  **L675 CN**: 执行语句 `"PLUS", "SUPPRESS", "PROCESSOR_DEFINED", nullptr};`。
- **L676 EN**: Begins a `switch` dispatch over discrete cases.
  **L676 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L677 EN**: Marks one `switch` case label.
  **L677 CN**: 标记一个 `switch` 的 case 标签。
- **L678 EN**: Initializes or updates `|`.
  **L678 CN**: 初始化或更新 `|`。
- **L679 EN**: Returns from the current function, often propagating a computed result.
  **L679 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L680 EN**: Marks one `switch` case label.
  **L680 CN**: 标记一个 `switch` 的 case 标签。
- **L681 EN**: Marks one `switch` case label.
  **L681 CN**: 标记一个 `switch` 的 case 标签。
- **L682 EN**: Initializes or updates `&`.
  **L682 CN**: 初始化或更新 `&`。
- **L683 EN**: Returns from the current function, often propagating a computed result.
  **L683 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L684 EN**: Provides the default branch for a `switch` statement.
  **L684 CN**: 为 `switch` 语句提供默认分支。
- **L685 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L685 CN**: 延续周围的声明、表达式或控制流结构。
- **L686 EN**: Executes statement `"Invalid SIGN='%.*s'", static_cast<int>(length), keyword);`.
  **L686 CN**: 执行语句 `"Invalid SIGN='%.*s'", static_cast<int>(length), keyword);`。
- **L687 EN**: Returns from the current function, often propagating a computed result.
  **L687 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L688 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L688 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L689 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L689 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L690 EN**: Blank line separates nearby declarations or logic blocks.
  **L690 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L691 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L691 CN**: 延续周围的声明、表达式或控制流结构。
- **L692 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L692 CN**: 延续周围的声明、表达式或控制流结构。
- **L693 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L693 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L694 EN**: Introduces conditional control flow with an `if` statement.
  **L694 CN**: 通过 `if` 语句引入条件控制流。
- **L695 EN**: Executes statement involving `set_mustBeFormatted`.
  **L695 CN**: 执行涉及 `set_mustBeFormatted` 的语句。
- **L696 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L696 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L697 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L697 CN**: 延续周围的声明、表达式或控制流结构。
- **L698 EN**: Executes statement `"PRINT", "PROCESSOR_DEFINED", "SUPPRESS", nullptr};`.
  **L698 CN**: 执行语句 `"PRINT", "PROCESSOR_DEFINED", "SUPPRESS", nullptr};`。
- **L699 EN**: Begins a `switch` dispatch over discrete cases.
  **L699 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L700 EN**: Marks one `switch` case label.
  **L700 CN**: 标记一个 `switch` 的 case 标签。
- **L701 EN**: Marks one `switch` case label.
  **L701 CN**: 标记一个 `switch` 的 case 标签。
- **L702 EN**: Initializes or updates `&`.
  **L702 CN**: 初始化或更新 `&`。
- **L703 EN**: Returns from the current function, often propagating a computed result.
  **L703 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L704 EN**: Marks one `switch` case label.
  **L704 CN**: 标记一个 `switch` 的 case 标签。

### Lines 705-736

````cpp
    io.mutableModes().editingFlags |= leadingZeroSuppress;
    return true;
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
        "Invalid LEADING_ZERO='%.*s'", static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetAccess)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetAccess() called when not in an OPEN statement");
    }
    return false;
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetAccess() called after GetNewUnit() for an OPEN statement");
  }
  static const char *keywords[]{
      "SEQUENTIAL", "DIRECT", "STREAM", "APPEND", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    open->set_access(Access::Sequential);
    break;
  case 1:
    open->set_access(Access::Direct);
    break;
````

- **L705 EN**: Initializes or updates `|`.
  **L705 CN**: 初始化或更新 `|`。
- **L706 EN**: Returns from the current function, often propagating a computed result.
  **L706 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L707 EN**: Provides the default branch for a `switch` statement.
  **L707 CN**: 为 `switch` 语句提供默认分支。
- **L708 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L708 CN**: 延续周围的声明、表达式或控制流结构。
- **L709 EN**: Executes statement `"Invalid LEADING_ZERO='%.*s'", static_cast<int>(length), keyword);`.
  **L709 CN**: 执行语句 `"Invalid LEADING_ZERO='%.*s'", static_cast<int>(length), keyword);`。
- **L710 EN**: Returns from the current function, often propagating a computed result.
  **L710 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L711 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L711 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L712 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L712 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L713 EN**: Blank line separates nearby declarations or logic blocks.
  **L713 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L714 EN**: Declares or defines callable `IODEF`.
  **L714 CN**: 声明或定义可调用实体 `IODEF`。
- **L715 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L715 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L716 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L716 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L717 EN**: Introduces conditional control flow with an `if` statement.
  **L717 CN**: 通过 `if` 语句引入条件控制流。
- **L718 EN**: Introduces conditional control flow with an `if` statement.
  **L718 CN**: 通过 `if` 语句引入条件控制流。
- **L719 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L719 CN**: 延续周围的声明、表达式或控制流结构。
- **L720 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L720 CN**: 延续周围的声明、表达式或控制流结构。
- **L721 EN**: Executes statement involving `SetAccess`.
  **L721 CN**: 执行涉及 `SetAccess` 的语句。
- **L722 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L722 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L723 EN**: Returns from the current function, often propagating a computed result.
  **L723 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L724 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L724 CN**: 延续周围的声明、表达式或控制流结构。
- **L725 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L725 CN**: 延续周围的声明、表达式或控制流结构。
- **L726 EN**: Executes statement involving `SetAccess`.
  **L726 CN**: 执行涉及 `SetAccess` 的语句。
- **L727 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L727 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L728 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L728 CN**: 延续周围的声明、表达式或控制流结构。
- **L729 EN**: Executes statement `"SEQUENTIAL", "DIRECT", "STREAM", "APPEND", nullptr};`.
  **L729 CN**: 执行语句 `"SEQUENTIAL", "DIRECT", "STREAM", "APPEND", nullptr};`。
- **L730 EN**: Begins a `switch` dispatch over discrete cases.
  **L730 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L731 EN**: Marks one `switch` case label.
  **L731 CN**: 标记一个 `switch` 的 case 标签。
- **L732 EN**: Executes statement involving `set_access`.
  **L732 CN**: 执行涉及 `set_access` 的语句。
- **L733 EN**: Breaks out of the current loop or switch.
  **L733 CN**: 跳出当前循环或 switch。
- **L734 EN**: Marks one `switch` case label.
  **L734 CN**: 标记一个 `switch` 的 case 标签。
- **L735 EN**: Executes statement involving `set_access`.
  **L735 CN**: 执行涉及 `set_access` 的语句。
- **L736 EN**: Breaks out of the current loop or switch.
  **L736 CN**: 跳出当前循环或 switch。

### Lines 737-768

````cpp
  case 2:
    open->set_access(Access::Stream);
    break;
  case 3: // Sun Fortran extension ACCESS=APPEND: treat as if POSITION=APPEND
    open->set_position(Position::Append);
    break;
  default:
    open->SignalError(IostatErrorInKeyword, "Invalid ACCESS='%.*s'",
        static_cast<int>(length), keyword);
  }
  return true;
}

bool IODEF(SetAction)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetAction() called when not in an OPEN statement");
    }
    return false;
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetAction() called after GetNewUnit() for an OPEN statement");
  }
  common::optional<Action> action;
  static const char *keywords[]{"READ", "WRITE", "READWRITE", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    action = Action::Read;
````

- **L737 EN**: Marks one `switch` case label.
  **L737 CN**: 标记一个 `switch` 的 case 标签。
- **L738 EN**: Executes statement involving `set_access`.
  **L738 CN**: 执行涉及 `set_access` 的语句。
- **L739 EN**: Breaks out of the current loop or switch.
  **L739 CN**: 跳出当前循环或 switch。
- **L740 EN**: Marks one `switch` case label.
  **L740 CN**: 标记一个 `switch` 的 case 标签。
- **L741 EN**: Executes statement involving `set_position`.
  **L741 CN**: 执行涉及 `set_position` 的语句。
- **L742 EN**: Breaks out of the current loop or switch.
  **L742 CN**: 跳出当前循环或 switch。
- **L743 EN**: Provides the default branch for a `switch` statement.
  **L743 CN**: 为 `switch` 语句提供默认分支。
- **L744 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L744 CN**: 延续周围的声明、表达式或控制流结构。
- **L745 EN**: Executes statement `static_cast<int>(length), keyword);`.
  **L745 CN**: 执行语句 `static_cast<int>(length), keyword);`。
- **L746 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L746 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L747 EN**: Returns from the current function, often propagating a computed result.
  **L747 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L748 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L748 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L749 EN**: Blank line separates nearby declarations or logic blocks.
  **L749 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L750 EN**: Declares or defines callable `IODEF`.
  **L750 CN**: 声明或定义可调用实体 `IODEF`。
- **L751 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L751 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L752 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L752 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L753 EN**: Introduces conditional control flow with an `if` statement.
  **L753 CN**: 通过 `if` 语句引入条件控制流。
- **L754 EN**: Introduces conditional control flow with an `if` statement.
  **L754 CN**: 通过 `if` 语句引入条件控制流。
- **L755 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L755 CN**: 延续周围的声明、表达式或控制流结构。
- **L756 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L756 CN**: 延续周围的声明、表达式或控制流结构。
- **L757 EN**: Executes statement involving `SetAction`.
  **L757 CN**: 执行涉及 `SetAction` 的语句。
- **L758 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L758 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L759 EN**: Returns from the current function, often propagating a computed result.
  **L759 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L760 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L760 CN**: 延续周围的声明、表达式或控制流结构。
- **L761 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L761 CN**: 延续周围的声明、表达式或控制流结构。
- **L762 EN**: Executes statement involving `SetAction`.
  **L762 CN**: 执行涉及 `SetAction` 的语句。
- **L763 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L763 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L764 EN**: Executes statement `common::optional<Action> action;`.
  **L764 CN**: 执行语句 `common::optional<Action> action;`。
- **L765 EN**: Executes statement `static const char *keywords[]{"READ", "WRITE", "READWRITE", nullptr};`.
  **L765 CN**: 执行语句 `static const char *keywords[]{"READ", "WRITE", "READWRITE", nullptr};`。
- **L766 EN**: Begins a `switch` dispatch over discrete cases.
  **L766 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L767 EN**: Marks one `switch` case label.
  **L767 CN**: 标记一个 `switch` 的 case 标签。
- **L768 EN**: Initializes or updates `action`.
  **L768 CN**: 初始化或更新 `action`。

### Lines 769-800

````cpp
    break;
  case 1:
    action = Action::Write;
    break;
  case 2:
    action = Action::ReadWrite;
    break;
  default:
    open->SignalError(IostatErrorInKeyword, "Invalid ACTION='%.*s'",
        static_cast<int>(length), keyword);
    return false;
  }
  RUNTIME_CHECK(io.GetIoErrorHandler(), action.has_value());
  if (open->wasExtant()) {
    if ((*action != Action::Write) != open->unit().mayRead() ||
        (*action != Action::Read) != open->unit().mayWrite()) {
      open->SignalError("ACTION= may not be changed on an open unit");
    }
  }
  open->set_action(*action);
  return true;
}

bool IODEF(SetAsynchronous)(
    Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  bool isYes{YesOrNo(keyword, length, "ASYNCHRONOUS", handler)};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    if (open->completedOperation()) {
      handler.Crash(
          "SetAsynchronous() called after GetNewUnit() for an OPEN statement");
````

- **L769 EN**: Breaks out of the current loop or switch.
  **L769 CN**: 跳出当前循环或 switch。
- **L770 EN**: Marks one `switch` case label.
  **L770 CN**: 标记一个 `switch` 的 case 标签。
- **L771 EN**: Initializes or updates `action`.
  **L771 CN**: 初始化或更新 `action`。
- **L772 EN**: Breaks out of the current loop or switch.
  **L772 CN**: 跳出当前循环或 switch。
- **L773 EN**: Marks one `switch` case label.
  **L773 CN**: 标记一个 `switch` 的 case 标签。
- **L774 EN**: Initializes or updates `action`.
  **L774 CN**: 初始化或更新 `action`。
- **L775 EN**: Breaks out of the current loop or switch.
  **L775 CN**: 跳出当前循环或 switch。
- **L776 EN**: Provides the default branch for a `switch` statement.
  **L776 CN**: 为 `switch` 语句提供默认分支。
- **L777 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L777 CN**: 延续周围的声明、表达式或控制流结构。
- **L778 EN**: Executes statement `static_cast<int>(length), keyword);`.
  **L778 CN**: 执行语句 `static_cast<int>(length), keyword);`。
- **L779 EN**: Returns from the current function, often propagating a computed result.
  **L779 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L780 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L780 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L781 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L781 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L782 EN**: Introduces conditional control flow with an `if` statement.
  **L782 CN**: 通过 `if` 语句引入条件控制流。
- **L783 EN**: Introduces conditional control flow with an `if` statement.
  **L783 CN**: 通过 `if` 语句引入条件控制流。
- **L784 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L784 CN**: 延续周围的声明、表达式或控制流结构。
- **L785 EN**: Executes statement involving `SignalError`.
  **L785 CN**: 执行涉及 `SignalError` 的语句。
- **L786 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L786 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L787 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L787 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L788 EN**: Executes statement involving `set_action`.
  **L788 CN**: 执行涉及 `set_action` 的语句。
- **L789 EN**: Returns from the current function, often propagating a computed result.
  **L789 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L790 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L790 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L791 EN**: Blank line separates nearby declarations or logic blocks.
  **L791 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L792 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L792 CN**: 延续周围的声明、表达式或控制流结构。
- **L793 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L793 CN**: 延续周围的声明、表达式或控制流结构。
- **L794 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L794 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L795 EN**: Executes statement involving `GetIoErrorHandler`.
  **L795 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L796 EN**: Executes statement involving `YesOrNo`.
  **L796 CN**: 执行涉及 `YesOrNo` 的语句。
- **L797 EN**: Introduces conditional control flow with an `if` statement.
  **L797 CN**: 通过 `if` 语句引入条件控制流。
- **L798 EN**: Introduces conditional control flow with an `if` statement.
  **L798 CN**: 通过 `if` 语句引入条件控制流。
- **L799 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L799 CN**: 延续周围的声明、表达式或控制流结构。
- **L800 EN**: Executes statement involving `SetAsynchronous`.
  **L800 CN**: 执行涉及 `SetAsynchronous` 的语句。

### Lines 801-832

````cpp
    }
    open->unit().set_mayAsynchronous(isYes);
  } else if (!isYes) {
    // ASYNCHRONOUS='NO' is the default, so this is a no-op
  } else if (auto *ext{io.get_if<ExternalIoStatementBase>()}) {
    if (ext->unit().mayAsynchronous()) {
      ext->SetAsynchronous();
    } else {
      handler.SignalError(IostatBadAsynchronous);
    }
  } else if (io.get_if<NoopStatementState>() ||
      io.get_if<ErroneousIoStatementState>()) {
    // no error
  } else if (io.get_if<ChildIoStatementState<Direction::Output>>() ||
      io.get_if<ChildIoStatementState<Direction::Input>>()) {
    io.GetIoErrorHandler().SignalError(IostatChildAsynchronous);
  } else {
    handler.Crash("SetAsynchronous('YES') called when not in an OPEN or "
                  "external I/O statement");
  }
  return !handler.InError();
}

bool IODEF(SetCarriagecontrol)(
    Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetCarriageControl() called when not in an OPEN statement");
````

- **L801 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L801 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L802 EN**: Executes statement involving `unit`.
  **L802 CN**: 执行涉及 `unit` 的语句。
- **L803 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L803 CN**: 延续周围的声明、表达式或控制流结构。
- **L804 EN**: Comment documents intent or context: `ASYNCHRONOUS='NO' is the default, so this is a no-op`.
  **L804 CN**: 注释记录了意图或上下文：`ASYNCHRONOUS='NO' is the default, so this is a no-op`。
- **L805 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L805 CN**: 延续周围的声明、表达式或控制流结构。
- **L806 EN**: Introduces conditional control flow with an `if` statement.
  **L806 CN**: 通过 `if` 语句引入条件控制流。
- **L807 EN**: Executes statement involving `SetAsynchronous`.
  **L807 CN**: 执行涉及 `SetAsynchronous` 的语句。
- **L808 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L808 CN**: 延续周围的声明、表达式或控制流结构。
- **L809 EN**: Executes statement involving `SignalError`.
  **L809 CN**: 执行涉及 `SignalError` 的语句。
- **L810 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L810 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L811 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L811 CN**: 延续周围的声明、表达式或控制流结构。
- **L812 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L812 CN**: 延续周围的声明、表达式或控制流结构。
- **L813 EN**: Comment documents intent or context: `no error`.
  **L813 CN**: 注释记录了意图或上下文：`no error`。
- **L814 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L814 CN**: 延续周围的声明、表达式或控制流结构。
- **L815 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L815 CN**: 延续周围的声明、表达式或控制流结构。
- **L816 EN**: Executes statement involving `GetIoErrorHandler`.
  **L816 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L817 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L817 CN**: 延续周围的声明、表达式或控制流结构。
- **L818 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L818 CN**: 延续周围的声明、表达式或控制流结构。
- **L819 EN**: Executes statement `"external I/O statement");`.
  **L819 CN**: 执行语句 `"external I/O statement");`。
- **L820 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L820 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L821 EN**: Returns from the current function, often propagating a computed result.
  **L821 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L822 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L822 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L823 EN**: Blank line separates nearby declarations or logic blocks.
  **L823 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L824 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L824 CN**: 延续周围的声明、表达式或控制流结构。
- **L825 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L825 CN**: 延续周围的声明、表达式或控制流结构。
- **L826 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L826 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L827 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L827 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L828 EN**: Introduces conditional control flow with an `if` statement.
  **L828 CN**: 通过 `if` 语句引入条件控制流。
- **L829 EN**: Introduces conditional control flow with an `if` statement.
  **L829 CN**: 通过 `if` 语句引入条件控制流。
- **L830 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L830 CN**: 延续周围的声明、表达式或控制流结构。
- **L831 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L831 CN**: 延续周围的声明、表达式或控制流结构。
- **L832 EN**: Executes statement involving `SetCarriageControl`.
  **L832 CN**: 执行涉及 `SetCarriageControl` 的语句。

### Lines 833-864

````cpp
    }
    return false;
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetCarriageControl() called after GetNewUnit() for an OPEN statement");
  }
  open->set_mustBeFormatted();
  static const char *keywords[]{"LIST", "FORTRAN", "NONE", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    return true;
  case 1:
  case 2:
    open->SignalError(IostatErrorInKeyword,
        "Unimplemented CARRIAGECONTROL='%.*s'", static_cast<int>(length),
        keyword);
    return false;
  default:
    open->SignalError(IostatErrorInKeyword, "Invalid CARRIAGECONTROL='%.*s'",
        static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetConvert)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetConvert() called when not in an OPEN statement");
````

- **L833 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L833 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L834 EN**: Returns from the current function, often propagating a computed result.
  **L834 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L835 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L835 CN**: 延续周围的声明、表达式或控制流结构。
- **L836 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L836 CN**: 延续周围的声明、表达式或控制流结构。
- **L837 EN**: Executes statement involving `SetCarriageControl`.
  **L837 CN**: 执行涉及 `SetCarriageControl` 的语句。
- **L838 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L838 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L839 EN**: Executes statement involving `set_mustBeFormatted`.
  **L839 CN**: 执行涉及 `set_mustBeFormatted` 的语句。
- **L840 EN**: Executes statement `static const char *keywords[]{"LIST", "FORTRAN", "NONE", nullptr};`.
  **L840 CN**: 执行语句 `static const char *keywords[]{"LIST", "FORTRAN", "NONE", nullptr};`。
- **L841 EN**: Begins a `switch` dispatch over discrete cases.
  **L841 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L842 EN**: Marks one `switch` case label.
  **L842 CN**: 标记一个 `switch` 的 case 标签。
- **L843 EN**: Returns from the current function, often propagating a computed result.
  **L843 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L844 EN**: Marks one `switch` case label.
  **L844 CN**: 标记一个 `switch` 的 case 标签。
- **L845 EN**: Marks one `switch` case label.
  **L845 CN**: 标记一个 `switch` 的 case 标签。
- **L846 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L846 CN**: 延续周围的声明、表达式或控制流结构。
- **L847 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L847 CN**: 延续周围的声明、表达式或控制流结构。
- **L848 EN**: Executes statement `keyword);`.
  **L848 CN**: 执行语句 `keyword);`。
- **L849 EN**: Returns from the current function, often propagating a computed result.
  **L849 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L850 EN**: Provides the default branch for a `switch` statement.
  **L850 CN**: 为 `switch` 语句提供默认分支。
- **L851 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L851 CN**: 延续周围的声明、表达式或控制流结构。
- **L852 EN**: Executes statement `static_cast<int>(length), keyword);`.
  **L852 CN**: 执行语句 `static_cast<int>(length), keyword);`。
- **L853 EN**: Returns from the current function, often propagating a computed result.
  **L853 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L854 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L854 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L855 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L855 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L856 EN**: Blank line separates nearby declarations or logic blocks.
  **L856 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L857 EN**: Declares or defines callable `IODEF`.
  **L857 CN**: 声明或定义可调用实体 `IODEF`。
- **L858 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L858 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L859 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L859 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L860 EN**: Introduces conditional control flow with an `if` statement.
  **L860 CN**: 通过 `if` 语句引入条件控制流。
- **L861 EN**: Introduces conditional control flow with an `if` statement.
  **L861 CN**: 通过 `if` 语句引入条件控制流。
- **L862 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L862 CN**: 延续周围的声明、表达式或控制流结构。
- **L863 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L863 CN**: 延续周围的声明、表达式或控制流结构。
- **L864 EN**: Executes statement involving `SetConvert`.
  **L864 CN**: 执行涉及 `SetConvert` 的语句。

### Lines 865-896

````cpp
    }
    return false;
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetConvert() called after GetNewUnit() for an OPEN statement");
  }
  if (auto convert{GetConvertFromString(keyword, length)}) {
    open->set_convert(*convert);
    return true;
  } else {
    open->SignalError(IostatErrorInKeyword, "Invalid CONVERT='%.*s'",
        static_cast<int>(length), keyword);
    return false;
  }
}

bool IODEF(SetEncoding)(
    Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetEncoding() called when not in an OPEN statement");
    }
    return false;
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetEncoding() called after GetNewUnit() for an OPEN statement");
  }
  open->set_mustBeFormatted();
````

- **L865 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L865 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L866 EN**: Returns from the current function, often propagating a computed result.
  **L866 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L867 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L867 CN**: 延续周围的声明、表达式或控制流结构。
- **L868 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L868 CN**: 延续周围的声明、表达式或控制流结构。
- **L869 EN**: Executes statement involving `SetConvert`.
  **L869 CN**: 执行涉及 `SetConvert` 的语句。
- **L870 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L870 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L871 EN**: Introduces conditional control flow with an `if` statement.
  **L871 CN**: 通过 `if` 语句引入条件控制流。
- **L872 EN**: Executes statement involving `set_convert`.
  **L872 CN**: 执行涉及 `set_convert` 的语句。
- **L873 EN**: Returns from the current function, often propagating a computed result.
  **L873 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L874 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L874 CN**: 延续周围的声明、表达式或控制流结构。
- **L875 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L875 CN**: 延续周围的声明、表达式或控制流结构。
- **L876 EN**: Executes statement `static_cast<int>(length), keyword);`.
  **L876 CN**: 执行语句 `static_cast<int>(length), keyword);`。
- **L877 EN**: Returns from the current function, often propagating a computed result.
  **L877 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L878 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L878 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L879 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L879 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L880 EN**: Blank line separates nearby declarations or logic blocks.
  **L880 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L881 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L881 CN**: 延续周围的声明、表达式或控制流结构。
- **L882 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L882 CN**: 延续周围的声明、表达式或控制流结构。
- **L883 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L883 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L884 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L884 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L885 EN**: Introduces conditional control flow with an `if` statement.
  **L885 CN**: 通过 `if` 语句引入条件控制流。
- **L886 EN**: Introduces conditional control flow with an `if` statement.
  **L886 CN**: 通过 `if` 语句引入条件控制流。
- **L887 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L887 CN**: 延续周围的声明、表达式或控制流结构。
- **L888 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L888 CN**: 延续周围的声明、表达式或控制流结构。
- **L889 EN**: Executes statement involving `SetEncoding`.
  **L889 CN**: 执行涉及 `SetEncoding` 的语句。
- **L890 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L890 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L891 EN**: Returns from the current function, often propagating a computed result.
  **L891 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L892 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L892 CN**: 延续周围的声明、表达式或控制流结构。
- **L893 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L893 CN**: 延续周围的声明、表达式或控制流结构。
- **L894 EN**: Executes statement involving `SetEncoding`.
  **L894 CN**: 执行涉及 `SetEncoding` 的语句。
- **L895 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L895 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L896 EN**: Executes statement involving `set_mustBeFormatted`.
  **L896 CN**: 执行涉及 `set_mustBeFormatted` 的语句。

### Lines 897-928

````cpp
  // Allow the encoding to be changed on an open unit -- it's
  // useful and safe.
  static const char *keywords[]{"UTF-8", "DEFAULT", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
  case 0:
    open->unit().isUTF8 = true;
    break;
  case 1:
    open->unit().isUTF8 = false;
    break;
  default:
    open->SignalError(IostatErrorInKeyword, "Invalid ENCODING='%.*s'",
        static_cast<int>(length), keyword);
  }
  return true;
}

bool IODEF(SetForm)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetForm() called when not in an OPEN statement");
    }
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetForm() called after GetNewUnit() for an OPEN statement");
  }
  static const char *keywords[]{"FORMATTED", "UNFORMATTED", "BINARY", nullptr};
  switch (IdentifyValue(keyword, length, keywords)) {
````

- **L897 EN**: Comment documents intent or context: `Allow the encoding to be changed on an open unit -- it's`.
  **L897 CN**: 注释记录了意图或上下文：`Allow the encoding to be changed on an open unit -- it's`。
- **L898 EN**: Comment documents intent or context: `useful and safe.`.
  **L898 CN**: 注释记录了意图或上下文：`useful and safe.`。
- **L899 EN**: Executes statement `static const char *keywords[]{"UTF-8", "DEFAULT", nullptr};`.
  **L899 CN**: 执行语句 `static const char *keywords[]{"UTF-8", "DEFAULT", nullptr};`。
- **L900 EN**: Begins a `switch` dispatch over discrete cases.
  **L900 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L901 EN**: Marks one `switch` case label.
  **L901 CN**: 标记一个 `switch` 的 case 标签。
- **L902 EN**: Initializes or updates `open->unit().isUTF8`.
  **L902 CN**: 初始化或更新 `open->unit().isUTF8`。
- **L903 EN**: Breaks out of the current loop or switch.
  **L903 CN**: 跳出当前循环或 switch。
- **L904 EN**: Marks one `switch` case label.
  **L904 CN**: 标记一个 `switch` 的 case 标签。
- **L905 EN**: Initializes or updates `open->unit().isUTF8`.
  **L905 CN**: 初始化或更新 `open->unit().isUTF8`。
- **L906 EN**: Breaks out of the current loop or switch.
  **L906 CN**: 跳出当前循环或 switch。
- **L907 EN**: Provides the default branch for a `switch` statement.
  **L907 CN**: 为 `switch` 语句提供默认分支。
- **L908 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L908 CN**: 延续周围的声明、表达式或控制流结构。
- **L909 EN**: Executes statement `static_cast<int>(length), keyword);`.
  **L909 CN**: 执行语句 `static_cast<int>(length), keyword);`。
- **L910 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L910 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L911 EN**: Returns from the current function, often propagating a computed result.
  **L911 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L912 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L912 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L913 EN**: Blank line separates nearby declarations or logic blocks.
  **L913 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L914 EN**: Declares or defines callable `IODEF`.
  **L914 CN**: 声明或定义可调用实体 `IODEF`。
- **L915 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L915 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L916 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L916 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L917 EN**: Introduces conditional control flow with an `if` statement.
  **L917 CN**: 通过 `if` 语句引入条件控制流。
- **L918 EN**: Introduces conditional control flow with an `if` statement.
  **L918 CN**: 通过 `if` 语句引入条件控制流。
- **L919 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L919 CN**: 延续周围的声明、表达式或控制流结构。
- **L920 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L920 CN**: 延续周围的声明、表达式或控制流结构。
- **L921 EN**: Executes statement involving `SetForm`.
  **L921 CN**: 执行涉及 `SetForm` 的语句。
- **L922 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L922 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L923 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L923 CN**: 延续周围的声明、表达式或控制流结构。
- **L924 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L924 CN**: 延续周围的声明、表达式或控制流结构。
- **L925 EN**: Executes statement involving `SetForm`.
  **L925 CN**: 执行涉及 `SetForm` 的语句。
- **L926 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L926 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L927 EN**: Executes statement `static const char *keywords[]{"FORMATTED", "UNFORMATTED", "BINARY", nullptr};`.
  **L927 CN**: 执行语句 `static const char *keywords[]{"FORMATTED", "UNFORMATTED", "BINARY", nullptr};`。
- **L928 EN**: Begins a `switch` dispatch over discrete cases.
  **L928 CN**: 开始一个针对离散分支的 `switch` 分派。

### Lines 929-960

````cpp
  case 0: // FORM='FORMATTED'
    open->set_isUnformatted(false);
    break;
  case 1: // FORM='UNFORMATTED'
    open->set_isUnformatted(true);
    break;
  case 2: // legacy FORM='BINARY' means an unformatted stream
    open->set_isUnformatted(true);
    open->set_access(Access::Stream);
    break;
  default:
    open->SignalError(IostatErrorInKeyword, "Invalid FORM='%.*s'",
        static_cast<int>(length), keyword);
  }
  return true;
}

bool IODEF(SetPosition)(
    Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetPosition() called when not in an OPEN statement");
    }
    return false;
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetPosition() called after GetNewUnit() for an OPEN statement");
  }
````

- **L929 EN**: Marks one `switch` case label.
  **L929 CN**: 标记一个 `switch` 的 case 标签。
- **L930 EN**: Executes statement involving `set_isUnformatted`.
  **L930 CN**: 执行涉及 `set_isUnformatted` 的语句。
- **L931 EN**: Breaks out of the current loop or switch.
  **L931 CN**: 跳出当前循环或 switch。
- **L932 EN**: Marks one `switch` case label.
  **L932 CN**: 标记一个 `switch` 的 case 标签。
- **L933 EN**: Executes statement involving `set_isUnformatted`.
  **L933 CN**: 执行涉及 `set_isUnformatted` 的语句。
- **L934 EN**: Breaks out of the current loop or switch.
  **L934 CN**: 跳出当前循环或 switch。
- **L935 EN**: Marks one `switch` case label.
  **L935 CN**: 标记一个 `switch` 的 case 标签。
- **L936 EN**: Executes statement involving `set_isUnformatted`.
  **L936 CN**: 执行涉及 `set_isUnformatted` 的语句。
- **L937 EN**: Executes statement involving `set_access`.
  **L937 CN**: 执行涉及 `set_access` 的语句。
- **L938 EN**: Breaks out of the current loop or switch.
  **L938 CN**: 跳出当前循环或 switch。
- **L939 EN**: Provides the default branch for a `switch` statement.
  **L939 CN**: 为 `switch` 语句提供默认分支。
- **L940 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L940 CN**: 延续周围的声明、表达式或控制流结构。
- **L941 EN**: Executes statement `static_cast<int>(length), keyword);`.
  **L941 CN**: 执行语句 `static_cast<int>(length), keyword);`。
- **L942 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L942 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L943 EN**: Returns from the current function, often propagating a computed result.
  **L943 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L944 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L944 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L945 EN**: Blank line separates nearby declarations or logic blocks.
  **L945 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L946 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L946 CN**: 延续周围的声明、表达式或控制流结构。
- **L947 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L947 CN**: 延续周围的声明、表达式或控制流结构。
- **L948 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L948 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L949 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L949 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L950 EN**: Introduces conditional control flow with an `if` statement.
  **L950 CN**: 通过 `if` 语句引入条件控制流。
- **L951 EN**: Introduces conditional control flow with an `if` statement.
  **L951 CN**: 通过 `if` 语句引入条件控制流。
- **L952 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L952 CN**: 延续周围的声明、表达式或控制流结构。
- **L953 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L953 CN**: 延续周围的声明、表达式或控制流结构。
- **L954 EN**: Executes statement involving `SetPosition`.
  **L954 CN**: 执行涉及 `SetPosition` 的语句。
- **L955 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L955 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L956 EN**: Returns from the current function, often propagating a computed result.
  **L956 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L957 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L957 CN**: 延续周围的声明、表达式或控制流结构。
- **L958 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L958 CN**: 延续周围的声明、表达式或控制流结构。
- **L959 EN**: Executes statement involving `SetPosition`.
  **L959 CN**: 执行涉及 `SetPosition` 的语句。
- **L960 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L960 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 961-992

````cpp
  static const char *positions[]{"ASIS", "REWIND", "APPEND", nullptr};
  switch (IdentifyValue(keyword, length, positions)) {
  case 0:
    open->set_position(Position::AsIs);
    return true;
  case 1:
    open->set_position(Position::Rewind);
    return true;
  case 2:
    open->set_position(Position::Append);
    return true;
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
        "Invalid POSITION='%.*s'", static_cast<int>(length), keyword);
  }
  return true;
}

bool IODEF(SetRecl)(Cookie cookie, std::size_t n) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "SetRecl() called when not in an OPEN statement");
    }
    return false;
  } else if (open->completedOperation()) {
    io.GetIoErrorHandler().Crash(
        "SetRecl() called after GetNewUnit() for an OPEN statement");
  }
````

- **L961 EN**: Executes statement `static const char *positions[]{"ASIS", "REWIND", "APPEND", nullptr};`.
  **L961 CN**: 执行语句 `static const char *positions[]{"ASIS", "REWIND", "APPEND", nullptr};`。
- **L962 EN**: Begins a `switch` dispatch over discrete cases.
  **L962 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L963 EN**: Marks one `switch` case label.
  **L963 CN**: 标记一个 `switch` 的 case 标签。
- **L964 EN**: Executes statement involving `set_position`.
  **L964 CN**: 执行涉及 `set_position` 的语句。
- **L965 EN**: Returns from the current function, often propagating a computed result.
  **L965 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L966 EN**: Marks one `switch` case label.
  **L966 CN**: 标记一个 `switch` 的 case 标签。
- **L967 EN**: Executes statement involving `set_position`.
  **L967 CN**: 执行涉及 `set_position` 的语句。
- **L968 EN**: Returns from the current function, often propagating a computed result.
  **L968 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L969 EN**: Marks one `switch` case label.
  **L969 CN**: 标记一个 `switch` 的 case 标签。
- **L970 EN**: Executes statement involving `set_position`.
  **L970 CN**: 执行涉及 `set_position` 的语句。
- **L971 EN**: Returns from the current function, often propagating a computed result.
  **L971 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L972 EN**: Provides the default branch for a `switch` statement.
  **L972 CN**: 为 `switch` 语句提供默认分支。
- **L973 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L973 CN**: 延续周围的声明、表达式或控制流结构。
- **L974 EN**: Executes statement `"Invalid POSITION='%.*s'", static_cast<int>(length), keyword);`.
  **L974 CN**: 执行语句 `"Invalid POSITION='%.*s'", static_cast<int>(length), keyword);`。
- **L975 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L975 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L976 EN**: Returns from the current function, often propagating a computed result.
  **L976 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L977 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L977 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L978 EN**: Blank line separates nearby declarations or logic blocks.
  **L978 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L979 EN**: Declares or defines callable `IODEF`.
  **L979 CN**: 声明或定义可调用实体 `IODEF`。
- **L980 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L980 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L981 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L981 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L982 EN**: Introduces conditional control flow with an `if` statement.
  **L982 CN**: 通过 `if` 语句引入条件控制流。
- **L983 EN**: Introduces conditional control flow with an `if` statement.
  **L983 CN**: 通过 `if` 语句引入条件控制流。
- **L984 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L984 CN**: 延续周围的声明、表达式或控制流结构。
- **L985 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L985 CN**: 延续周围的声明、表达式或控制流结构。
- **L986 EN**: Executes statement involving `SetRecl`.
  **L986 CN**: 执行涉及 `SetRecl` 的语句。
- **L987 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L987 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L988 EN**: Returns from the current function, often propagating a computed result.
  **L988 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L989 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L989 CN**: 延续周围的声明、表达式或控制流结构。
- **L990 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L990 CN**: 延续周围的声明、表达式或控制流结构。
- **L991 EN**: Executes statement involving `SetRecl`.
  **L991 CN**: 执行涉及 `SetRecl` 的语句。
- **L992 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L992 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 993-1024

````cpp
  if (static_cast<std::int64_t>(n) <= 0) {
    io.GetIoErrorHandler().SignalError("RECL= must be greater than zero");
    return false;
  } else if (open->wasExtant() &&
      open->unit().openRecl.value_or(0) != static_cast<std::int64_t>(n)) {
    open->SignalError("RECL= may not be changed for an open unit");
    return false;
  } else {
    open->unit().openRecl = n;
    return true;
  }
}

bool IODEF(SetStatus)(Cookie cookie, const char *keyword, std::size_t length) {
  IoStatementState &io{*cookie};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    if (open->completedOperation()) {
      io.GetIoErrorHandler().Crash(
          "SetStatus() called after GetNewUnit() for an OPEN statement");
    }
    static const char *statuses[]{
        "OLD", "NEW", "SCRATCH", "REPLACE", "UNKNOWN", nullptr};
    switch (IdentifyValue(keyword, length, statuses)) {
    case 0:
      open->set_status(OpenStatus::Old);
      return true;
    case 1:
      open->set_status(OpenStatus::New);
      return true;
    case 2:
      open->set_status(OpenStatus::Scratch);
      return true;
````

- **L993 EN**: Introduces conditional control flow with an `if` statement.
  **L993 CN**: 通过 `if` 语句引入条件控制流。
- **L994 EN**: Executes statement involving `GetIoErrorHandler`.
  **L994 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L995 EN**: Returns from the current function, often propagating a computed result.
  **L995 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L996 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L996 CN**: 延续周围的声明、表达式或控制流结构。
- **L997 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L997 CN**: 延续周围的声明、表达式或控制流结构。
- **L998 EN**: Executes statement involving `SignalError`.
  **L998 CN**: 执行涉及 `SignalError` 的语句。
- **L999 EN**: Returns from the current function, often propagating a computed result.
  **L999 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1000 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1000 CN**: 延续周围的声明、表达式或控制流结构。
- **L1001 EN**: Initializes or updates `open->unit().openRecl`.
  **L1001 CN**: 初始化或更新 `open->unit().openRecl`。
- **L1002 EN**: Returns from the current function, often propagating a computed result.
  **L1002 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1003 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1003 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1004 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1004 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1005 EN**: Blank line separates nearby declarations or logic blocks.
  **L1005 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1006 EN**: Declares or defines callable `IODEF`.
  **L1006 CN**: 声明或定义可调用实体 `IODEF`。
- **L1007 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1007 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1008 EN**: Introduces conditional control flow with an `if` statement.
  **L1008 CN**: 通过 `if` 语句引入条件控制流。
- **L1009 EN**: Introduces conditional control flow with an `if` statement.
  **L1009 CN**: 通过 `if` 语句引入条件控制流。
- **L1010 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1010 CN**: 延续周围的声明、表达式或控制流结构。
- **L1011 EN**: Executes statement involving `SetStatus`.
  **L1011 CN**: 执行涉及 `SetStatus` 的语句。
- **L1012 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1012 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1013 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1013 CN**: 延续周围的声明、表达式或控制流结构。
- **L1014 EN**: Executes statement `"OLD", "NEW", "SCRATCH", "REPLACE", "UNKNOWN", nullptr};`.
  **L1014 CN**: 执行语句 `"OLD", "NEW", "SCRATCH", "REPLACE", "UNKNOWN", nullptr};`。
- **L1015 EN**: Begins a `switch` dispatch over discrete cases.
  **L1015 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1016 EN**: Marks one `switch` case label.
  **L1016 CN**: 标记一个 `switch` 的 case 标签。
- **L1017 EN**: Executes statement involving `set_status`.
  **L1017 CN**: 执行涉及 `set_status` 的语句。
- **L1018 EN**: Returns from the current function, often propagating a computed result.
  **L1018 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1019 EN**: Marks one `switch` case label.
  **L1019 CN**: 标记一个 `switch` 的 case 标签。
- **L1020 EN**: Executes statement involving `set_status`.
  **L1020 CN**: 执行涉及 `set_status` 的语句。
- **L1021 EN**: Returns from the current function, often propagating a computed result.
  **L1021 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1022 EN**: Marks one `switch` case label.
  **L1022 CN**: 标记一个 `switch` 的 case 标签。
- **L1023 EN**: Executes statement involving `set_status`.
  **L1023 CN**: 执行涉及 `set_status` 的语句。
- **L1024 EN**: Returns from the current function, often propagating a computed result.
  **L1024 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 1025-1056

````cpp
    case 3:
      open->set_status(OpenStatus::Replace);
      return true;
    case 4:
      open->set_status(OpenStatus::Unknown);
      return true;
    default:
      io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
          "Invalid STATUS='%.*s'", static_cast<int>(length), keyword);
    }
    return false;
  }
  if (auto *close{io.get_if<CloseStatementState>()}) {
    static const char *statuses[]{"KEEP", "DELETE", nullptr};
    switch (IdentifyValue(keyword, length, statuses)) {
    case 0:
      close->set_status(CloseStatus::Keep);
      return true;
    case 1:
      close->set_status(CloseStatus::Delete);
      return true;
    default:
      io.GetIoErrorHandler().SignalError(IostatErrorInKeyword,
          "Invalid STATUS='%.*s'", static_cast<int>(length), keyword);
    }
    return false;
  }
  if (io.get_if<NoopStatementState>() ||
      io.get_if<ErroneousIoStatementState>()) {
    return true; // don't bother validating STATUS= in a no-op CLOSE
  }
  io.GetIoErrorHandler().Crash(
````

- **L1025 EN**: Marks one `switch` case label.
  **L1025 CN**: 标记一个 `switch` 的 case 标签。
- **L1026 EN**: Executes statement involving `set_status`.
  **L1026 CN**: 执行涉及 `set_status` 的语句。
- **L1027 EN**: Returns from the current function, often propagating a computed result.
  **L1027 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1028 EN**: Marks one `switch` case label.
  **L1028 CN**: 标记一个 `switch` 的 case 标签。
- **L1029 EN**: Executes statement involving `set_status`.
  **L1029 CN**: 执行涉及 `set_status` 的语句。
- **L1030 EN**: Returns from the current function, often propagating a computed result.
  **L1030 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1031 EN**: Provides the default branch for a `switch` statement.
  **L1031 CN**: 为 `switch` 语句提供默认分支。
- **L1032 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1032 CN**: 延续周围的声明、表达式或控制流结构。
- **L1033 EN**: Executes statement `"Invalid STATUS='%.*s'", static_cast<int>(length), keyword);`.
  **L1033 CN**: 执行语句 `"Invalid STATUS='%.*s'", static_cast<int>(length), keyword);`。
- **L1034 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1034 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1035 EN**: Returns from the current function, often propagating a computed result.
  **L1035 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1036 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1036 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1037 EN**: Introduces conditional control flow with an `if` statement.
  **L1037 CN**: 通过 `if` 语句引入条件控制流。
- **L1038 EN**: Executes statement `static const char *statuses[]{"KEEP", "DELETE", nullptr};`.
  **L1038 CN**: 执行语句 `static const char *statuses[]{"KEEP", "DELETE", nullptr};`。
- **L1039 EN**: Begins a `switch` dispatch over discrete cases.
  **L1039 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1040 EN**: Marks one `switch` case label.
  **L1040 CN**: 标记一个 `switch` 的 case 标签。
- **L1041 EN**: Executes statement involving `set_status`.
  **L1041 CN**: 执行涉及 `set_status` 的语句。
- **L1042 EN**: Returns from the current function, often propagating a computed result.
  **L1042 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1043 EN**: Marks one `switch` case label.
  **L1043 CN**: 标记一个 `switch` 的 case 标签。
- **L1044 EN**: Executes statement involving `set_status`.
  **L1044 CN**: 执行涉及 `set_status` 的语句。
- **L1045 EN**: Returns from the current function, often propagating a computed result.
  **L1045 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1046 EN**: Provides the default branch for a `switch` statement.
  **L1046 CN**: 为 `switch` 语句提供默认分支。
- **L1047 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1047 CN**: 延续周围的声明、表达式或控制流结构。
- **L1048 EN**: Executes statement `"Invalid STATUS='%.*s'", static_cast<int>(length), keyword);`.
  **L1048 CN**: 执行语句 `"Invalid STATUS='%.*s'", static_cast<int>(length), keyword);`。
- **L1049 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1049 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1050 EN**: Returns from the current function, often propagating a computed result.
  **L1050 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1051 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1051 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1052 EN**: Introduces conditional control flow with an `if` statement.
  **L1052 CN**: 通过 `if` 语句引入条件控制流。
- **L1053 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1053 CN**: 延续周围的声明、表达式或控制流结构。
- **L1054 EN**: Returns from the current function, often propagating a computed result.
  **L1054 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1055 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1055 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1056 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1056 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 1057-1088

````cpp
      "SetStatus() called when not in an OPEN or CLOSE statement");
}

bool IODEF(SetFile)(Cookie cookie, const char *path, std::size_t chars) {
  IoStatementState &io{*cookie};
  if (auto *open{io.get_if<OpenStatementState>()}) {
    if (open->completedOperation()) {
      io.GetIoErrorHandler().Crash(
          "SetFile() called after GetNewUnit() for an OPEN statement");
    }
    open->set_path(path, chars);
    return true;
  } else if (!io.get_if<NoopStatementState>() &&
      !io.get_if<ErroneousIoStatementState>()) {
    io.GetIoErrorHandler().Crash(
        "SetFile() called when not in an OPEN statement");
  }
  return false;
}

bool IODEF(GetNewUnit)(Cookie cookie, int &unit, int kind) {
  IoStatementState &io{*cookie};
  auto *open{io.get_if<OpenStatementState>()};
  if (!open) {
    if (!io.get_if<NoopStatementState>() &&
        !io.get_if<ErroneousIoStatementState>()) {
      io.GetIoErrorHandler().Crash(
          "GetNewUnit() called when not in an OPEN statement");
    }
    return false;
  } else if (!open->InError()) {
    open->CompleteOperation();
````

- **L1057 EN**: Executes statement involving `SetStatus`.
  **L1057 CN**: 执行涉及 `SetStatus` 的语句。
- **L1058 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1058 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1059 EN**: Blank line separates nearby declarations or logic blocks.
  **L1059 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1060 EN**: Declares or defines callable `IODEF`.
  **L1060 CN**: 声明或定义可调用实体 `IODEF`。
- **L1061 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1061 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1062 EN**: Introduces conditional control flow with an `if` statement.
  **L1062 CN**: 通过 `if` 语句引入条件控制流。
- **L1063 EN**: Introduces conditional control flow with an `if` statement.
  **L1063 CN**: 通过 `if` 语句引入条件控制流。
- **L1064 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1064 CN**: 延续周围的声明、表达式或控制流结构。
- **L1065 EN**: Executes statement involving `SetFile`.
  **L1065 CN**: 执行涉及 `SetFile` 的语句。
- **L1066 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1066 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1067 EN**: Executes statement involving `set_path`.
  **L1067 CN**: 执行涉及 `set_path` 的语句。
- **L1068 EN**: Returns from the current function, often propagating a computed result.
  **L1068 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1069 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1069 CN**: 延续周围的声明、表达式或控制流结构。
- **L1070 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1070 CN**: 延续周围的声明、表达式或控制流结构。
- **L1071 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1071 CN**: 延续周围的声明、表达式或控制流结构。
- **L1072 EN**: Executes statement involving `SetFile`.
  **L1072 CN**: 执行涉及 `SetFile` 的语句。
- **L1073 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1073 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1074 EN**: Returns from the current function, often propagating a computed result.
  **L1074 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1075 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1075 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1076 EN**: Blank line separates nearby declarations or logic blocks.
  **L1076 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1077 EN**: Declares or defines callable `IODEF`.
  **L1077 CN**: 声明或定义可调用实体 `IODEF`。
- **L1078 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1078 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1079 EN**: Executes statement `auto *open{io.get_if<OpenStatementState>()};`.
  **L1079 CN**: 执行语句 `auto *open{io.get_if<OpenStatementState>()};`。
- **L1080 EN**: Introduces conditional control flow with an `if` statement.
  **L1080 CN**: 通过 `if` 语句引入条件控制流。
- **L1081 EN**: Introduces conditional control flow with an `if` statement.
  **L1081 CN**: 通过 `if` 语句引入条件控制流。
- **L1082 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1082 CN**: 延续周围的声明、表达式或控制流结构。
- **L1083 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1083 CN**: 延续周围的声明、表达式或控制流结构。
- **L1084 EN**: Executes statement involving `GetNewUnit`.
  **L1084 CN**: 执行涉及 `GetNewUnit` 的语句。
- **L1085 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1085 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1086 EN**: Returns from the current function, often propagating a computed result.
  **L1086 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1087 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1087 CN**: 延续周围的声明、表达式或控制流结构。
- **L1088 EN**: Executes statement involving `CompleteOperation`.
  **L1088 CN**: 执行涉及 `CompleteOperation` 的语句。

### Lines 1089-1120

````cpp
  }
  if (open->InError()) {
    // A failed OPEN(NEWUNIT=n) does not modify 'n'
    return false;
  }
  std::int64_t result{open->unit().unitNumber()};
  if (!SetInteger(unit, kind, result)) {
    open->SignalError("GetNewUnit(): bad INTEGER kind(%d) or out-of-range "
                      "value(%jd) for result",
        kind, static_cast<std::intmax_t>(result));
  }
  return true;
}

// Data transfers

bool IODEF(OutputDescriptor)(Cookie cookie, const Descriptor &descriptor) {
  return descr::DescriptorIO<Direction::Output>(*cookie, descriptor);
}

bool IODEF(InputDescriptor)(Cookie cookie, const Descriptor &descriptor) {
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor);
}

bool IODEF(InputInteger)(Cookie cookie, std::int64_t &n, int kind) {
  IoStatementState &io{*cookie};
  if (io.BeginReadingRecord()) {
    if (auto edit{io.GetNextDataEdit()}) {
      return edit->descriptor == DataEdit::ListDirectedNullValue ||
          EditIntegerInput(io, *edit, reinterpret_cast<void *>(&n), kind,
              /*isSigned=*/true);
    }
````

- **L1089 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1089 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1090 EN**: Introduces conditional control flow with an `if` statement.
  **L1090 CN**: 通过 `if` 语句引入条件控制流。
- **L1091 EN**: Comment documents intent or context: `A failed OPEN(NEWUNIT=n) does not modify 'n'`.
  **L1091 CN**: 注释记录了意图或上下文：`A failed OPEN(NEWUNIT=n) does not modify 'n'`。
- **L1092 EN**: Returns from the current function, often propagating a computed result.
  **L1092 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1093 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1093 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1094 EN**: Executes statement involving `unit`.
  **L1094 CN**: 执行涉及 `unit` 的语句。
- **L1095 EN**: Introduces conditional control flow with an `if` statement.
  **L1095 CN**: 通过 `if` 语句引入条件控制流。
- **L1096 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1096 CN**: 延续周围的声明、表达式或控制流结构。
- **L1097 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1097 CN**: 延续周围的声明、表达式或控制流结构。
- **L1098 EN**: Executes statement `kind, static_cast<std::intmax_t>(result));`.
  **L1098 CN**: 执行语句 `kind, static_cast<std::intmax_t>(result));`。
- **L1099 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1099 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1100 EN**: Returns from the current function, often propagating a computed result.
  **L1100 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1101 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1101 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1102 EN**: Blank line separates nearby declarations or logic blocks.
  **L1102 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1103 EN**: Comment documents intent or context: `Data transfers`.
  **L1103 CN**: 注释记录了意图或上下文：`Data transfers`。
- **L1104 EN**: Blank line separates nearby declarations or logic blocks.
  **L1104 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1105 EN**: Declares or defines callable `IODEF`.
  **L1105 CN**: 声明或定义可调用实体 `IODEF`。
- **L1106 EN**: Returns from the current function, often propagating a computed result.
  **L1106 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1107 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1107 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1108 EN**: Blank line separates nearby declarations or logic blocks.
  **L1108 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1109 EN**: Declares or defines callable `IODEF`.
  **L1109 CN**: 声明或定义可调用实体 `IODEF`。
- **L1110 EN**: Returns from the current function, often propagating a computed result.
  **L1110 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1111 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1111 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1112 EN**: Blank line separates nearby declarations or logic blocks.
  **L1112 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1113 EN**: Declares or defines callable `IODEF`.
  **L1113 CN**: 声明或定义可调用实体 `IODEF`。
- **L1114 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1114 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1115 EN**: Introduces conditional control flow with an `if` statement.
  **L1115 CN**: 通过 `if` 语句引入条件控制流。
- **L1116 EN**: Introduces conditional control flow with an `if` statement.
  **L1116 CN**: 通过 `if` 语句引入条件控制流。
- **L1117 EN**: Returns from the current function, often propagating a computed result.
  **L1117 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1118 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1118 CN**: 延续周围的声明、表达式或控制流结构。
- **L1119 EN**: Comment documents intent or context: `isSigned=*/true);`.
  **L1119 CN**: 注释记录了意图或上下文：`isSigned=*/true);`。
- **L1120 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1120 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 1121-1152

````cpp
  }
  return false;
}

bool IODEF(InputReal32)(Cookie cookie, float &x) {
  if (!cookie->CheckFormattedStmtType<Direction::Input>("InputReal32")) {
    return false;
  }
  StaticDescriptor<0> staticDescriptor;
  Descriptor &descriptor{staticDescriptor.descriptor()};
  descriptor.Establish(TypeCategory::Real, 4, reinterpret_cast<void *>(&x), 0);
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor);
}

bool IODEF(InputReal64)(Cookie cookie, double &x) {
  if (!cookie->CheckFormattedStmtType<Direction::Input>("InputReal64")) {
    return false;
  }
  StaticDescriptor<0> staticDescriptor;
  Descriptor &descriptor{staticDescriptor.descriptor()};
  descriptor.Establish(TypeCategory::Real, 8, reinterpret_cast<void *>(&x), 0);
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor);
}

bool IODEF(InputComplex32)(Cookie cookie, float z[2]) {
  if (!cookie->CheckFormattedStmtType<Direction::Input>("InputComplex32")) {
    return false;
  }
  StaticDescriptor<0> staticDescriptor;
  Descriptor &descriptor{staticDescriptor.descriptor()};
  descriptor.Establish(
      TypeCategory::Complex, 4, reinterpret_cast<void *>(z), 0);
````

- **L1121 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1121 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1122 EN**: Returns from the current function, often propagating a computed result.
  **L1122 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1123 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1123 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1124 EN**: Blank line separates nearby declarations or logic blocks.
  **L1124 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1125 EN**: Declares or defines callable `IODEF`.
  **L1125 CN**: 声明或定义可调用实体 `IODEF`。
- **L1126 EN**: Introduces conditional control flow with an `if` statement.
  **L1126 CN**: 通过 `if` 语句引入条件控制流。
- **L1127 EN**: Returns from the current function, often propagating a computed result.
  **L1127 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1128 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1128 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1129 EN**: Executes statement `StaticDescriptor<0> staticDescriptor;`.
  **L1129 CN**: 执行语句 `StaticDescriptor<0> staticDescriptor;`。
- **L1130 EN**: Executes statement involving `descriptor`.
  **L1130 CN**: 执行涉及 `descriptor` 的语句。
- **L1131 EN**: Executes statement involving `Establish`.
  **L1131 CN**: 执行涉及 `Establish` 的语句。
- **L1132 EN**: Returns from the current function, often propagating a computed result.
  **L1132 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1133 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1133 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1134 EN**: Blank line separates nearby declarations or logic blocks.
  **L1134 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1135 EN**: Declares or defines callable `IODEF`.
  **L1135 CN**: 声明或定义可调用实体 `IODEF`。
- **L1136 EN**: Introduces conditional control flow with an `if` statement.
  **L1136 CN**: 通过 `if` 语句引入条件控制流。
- **L1137 EN**: Returns from the current function, often propagating a computed result.
  **L1137 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1138 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1138 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1139 EN**: Executes statement `StaticDescriptor<0> staticDescriptor;`.
  **L1139 CN**: 执行语句 `StaticDescriptor<0> staticDescriptor;`。
- **L1140 EN**: Executes statement involving `descriptor`.
  **L1140 CN**: 执行涉及 `descriptor` 的语句。
- **L1141 EN**: Executes statement involving `Establish`.
  **L1141 CN**: 执行涉及 `Establish` 的语句。
- **L1142 EN**: Returns from the current function, often propagating a computed result.
  **L1142 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1143 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1143 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1144 EN**: Blank line separates nearby declarations or logic blocks.
  **L1144 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1145 EN**: Declares or defines callable `IODEF`.
  **L1145 CN**: 声明或定义可调用实体 `IODEF`。
- **L1146 EN**: Introduces conditional control flow with an `if` statement.
  **L1146 CN**: 通过 `if` 语句引入条件控制流。
- **L1147 EN**: Returns from the current function, often propagating a computed result.
  **L1147 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1148 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1148 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1149 EN**: Executes statement `StaticDescriptor<0> staticDescriptor;`.
  **L1149 CN**: 执行语句 `StaticDescriptor<0> staticDescriptor;`。
- **L1150 EN**: Executes statement involving `descriptor`.
  **L1150 CN**: 执行涉及 `descriptor` 的语句。
- **L1151 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1151 CN**: 延续周围的声明、表达式或控制流结构。
- **L1152 EN**: Executes statement `TypeCategory::Complex, 4, reinterpret_cast<void *>(z), 0);`.
  **L1152 CN**: 执行语句 `TypeCategory::Complex, 4, reinterpret_cast<void *>(z), 0);`。

### Lines 1153-1184

````cpp
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor);
}

bool IODEF(InputComplex64)(Cookie cookie, double z[2]) {
  if (!cookie->CheckFormattedStmtType<Direction::Input>("InputComplex64")) {
    return false;
  }
  StaticDescriptor<0> staticDescriptor;
  Descriptor &descriptor{staticDescriptor.descriptor()};
  descriptor.Establish(
      TypeCategory::Complex, 8, reinterpret_cast<void *>(z), 0);
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor);
}

bool IODEF(OutputCharacter)(
    Cookie cookie, const char *x, std::size_t length, int kind) {
  if (!cookie->CheckFormattedStmtType<Direction::Output>("OutputCharacter")) {
    return false;
  }
  StaticDescriptor<0> staticDescriptor;
  Descriptor &descriptor{staticDescriptor.descriptor()};
  descriptor.Establish(
      kind, length, reinterpret_cast<void *>(const_cast<char *>(x)), 0);
  return descr::DescriptorIO<Direction::Output>(*cookie, descriptor);
}

bool IODEF(InputCharacter)(
    Cookie cookie, char *x, std::size_t length, int kind) {
  if (!cookie->CheckFormattedStmtType<Direction::Input>("InputCharacter")) {
    return false;
  }
  StaticDescriptor<0> staticDescriptor;
````

- **L1153 EN**: Returns from the current function, often propagating a computed result.
  **L1153 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1154 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1154 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1155 EN**: Blank line separates nearby declarations or logic blocks.
  **L1155 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1156 EN**: Declares or defines callable `IODEF`.
  **L1156 CN**: 声明或定义可调用实体 `IODEF`。
- **L1157 EN**: Introduces conditional control flow with an `if` statement.
  **L1157 CN**: 通过 `if` 语句引入条件控制流。
- **L1158 EN**: Returns from the current function, often propagating a computed result.
  **L1158 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1159 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1159 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1160 EN**: Executes statement `StaticDescriptor<0> staticDescriptor;`.
  **L1160 CN**: 执行语句 `StaticDescriptor<0> staticDescriptor;`。
- **L1161 EN**: Executes statement involving `descriptor`.
  **L1161 CN**: 执行涉及 `descriptor` 的语句。
- **L1162 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1162 CN**: 延续周围的声明、表达式或控制流结构。
- **L1163 EN**: Executes statement `TypeCategory::Complex, 8, reinterpret_cast<void *>(z), 0);`.
  **L1163 CN**: 执行语句 `TypeCategory::Complex, 8, reinterpret_cast<void *>(z), 0);`。
- **L1164 EN**: Returns from the current function, often propagating a computed result.
  **L1164 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1165 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1165 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1166 EN**: Blank line separates nearby declarations or logic blocks.
  **L1166 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1167 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1167 CN**: 延续周围的声明、表达式或控制流结构。
- **L1168 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1168 CN**: 延续周围的声明、表达式或控制流结构。
- **L1169 EN**: Introduces conditional control flow with an `if` statement.
  **L1169 CN**: 通过 `if` 语句引入条件控制流。
- **L1170 EN**: Returns from the current function, often propagating a computed result.
  **L1170 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1171 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1171 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1172 EN**: Executes statement `StaticDescriptor<0> staticDescriptor;`.
  **L1172 CN**: 执行语句 `StaticDescriptor<0> staticDescriptor;`。
- **L1173 EN**: Executes statement involving `descriptor`.
  **L1173 CN**: 执行涉及 `descriptor` 的语句。
- **L1174 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1174 CN**: 延续周围的声明、表达式或控制流结构。
- **L1175 EN**: Executes statement `kind, length, reinterpret_cast<void *>(const_cast<char *>(x)), 0);`.
  **L1175 CN**: 执行语句 `kind, length, reinterpret_cast<void *>(const_cast<char *>(x)), 0);`。
- **L1176 EN**: Returns from the current function, often propagating a computed result.
  **L1176 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1177 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1177 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1178 EN**: Blank line separates nearby declarations or logic blocks.
  **L1178 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1179 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1179 CN**: 延续周围的声明、表达式或控制流结构。
- **L1180 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1180 CN**: 延续周围的声明、表达式或控制流结构。
- **L1181 EN**: Introduces conditional control flow with an `if` statement.
  **L1181 CN**: 通过 `if` 语句引入条件控制流。
- **L1182 EN**: Returns from the current function, often propagating a computed result.
  **L1182 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1183 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1183 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1184 EN**: Executes statement `StaticDescriptor<0> staticDescriptor;`.
  **L1184 CN**: 执行语句 `StaticDescriptor<0> staticDescriptor;`。

### Lines 1185-1216

````cpp
  Descriptor &descriptor{staticDescriptor.descriptor()};
  descriptor.Establish(kind, length, reinterpret_cast<void *>(x), 0);
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor);
}

bool IODEF(InputAscii)(Cookie cookie, char *x, std::size_t length) {
  return IONAME(InputCharacter)(cookie, x, length, 1);
}

bool IODEF(InputLogical)(Cookie cookie, bool &truth) {
  if (!cookie->CheckFormattedStmtType<Direction::Input>("InputLogical")) {
    return false;
  }
  StaticDescriptor<0> staticDescriptor;
  Descriptor &descriptor{staticDescriptor.descriptor()};
  descriptor.Establish(
      TypeCategory::Logical, sizeof truth, reinterpret_cast<void *>(&truth), 0);
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor);
}

bool IODEF(OutputDerivedType)(Cookie cookie, const Descriptor &descriptor,
    const NonTbpDefinedIoTable *table) {
  return descr::DescriptorIO<Direction::Output>(*cookie, descriptor, table);
}

bool IODEF(InputDerivedType)(Cookie cookie, const Descriptor &descriptor,
    const NonTbpDefinedIoTable *table) {
  return descr::DescriptorIO<Direction::Input>(*cookie, descriptor, table);
}

std::size_t IODEF(GetSize)(Cookie cookie) {
  IoStatementState &io{*cookie};
````

- **L1185 EN**: Executes statement involving `descriptor`.
  **L1185 CN**: 执行涉及 `descriptor` 的语句。
- **L1186 EN**: Executes statement involving `Establish`.
  **L1186 CN**: 执行涉及 `Establish` 的语句。
- **L1187 EN**: Returns from the current function, often propagating a computed result.
  **L1187 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1188 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1188 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1189 EN**: Blank line separates nearby declarations or logic blocks.
  **L1189 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1190 EN**: Declares or defines callable `IODEF`.
  **L1190 CN**: 声明或定义可调用实体 `IODEF`。
- **L1191 EN**: Returns from the current function, often propagating a computed result.
  **L1191 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1192 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1192 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1193 EN**: Blank line separates nearby declarations or logic blocks.
  **L1193 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1194 EN**: Declares or defines callable `IODEF`.
  **L1194 CN**: 声明或定义可调用实体 `IODEF`。
- **L1195 EN**: Introduces conditional control flow with an `if` statement.
  **L1195 CN**: 通过 `if` 语句引入条件控制流。
- **L1196 EN**: Returns from the current function, often propagating a computed result.
  **L1196 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1197 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1197 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1198 EN**: Executes statement `StaticDescriptor<0> staticDescriptor;`.
  **L1198 CN**: 执行语句 `StaticDescriptor<0> staticDescriptor;`。
- **L1199 EN**: Executes statement involving `descriptor`.
  **L1199 CN**: 执行涉及 `descriptor` 的语句。
- **L1200 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1200 CN**: 延续周围的声明、表达式或控制流结构。
- **L1201 EN**: Executes statement `TypeCategory::Logical, sizeof truth, reinterpret_cast<void *>(&truth), 0);`.
  **L1201 CN**: 执行语句 `TypeCategory::Logical, sizeof truth, reinterpret_cast<void *>(&truth), 0);`。
- **L1202 EN**: Returns from the current function, often propagating a computed result.
  **L1202 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1203 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1203 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1204 EN**: Blank line separates nearby declarations or logic blocks.
  **L1204 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1205 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1205 CN**: 延续周围的声明、表达式或控制流结构。
- **L1206 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1206 CN**: 延续周围的声明、表达式或控制流结构。
- **L1207 EN**: Returns from the current function, often propagating a computed result.
  **L1207 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1208 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1208 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1209 EN**: Blank line separates nearby declarations or logic blocks.
  **L1209 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1210 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1210 CN**: 延续周围的声明、表达式或控制流结构。
- **L1211 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1211 CN**: 延续周围的声明、表达式或控制流结构。
- **L1212 EN**: Returns from the current function, often propagating a computed result.
  **L1212 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1213 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1213 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1214 EN**: Blank line separates nearby declarations or logic blocks.
  **L1214 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1215 EN**: Declares or defines callable `IODEF`.
  **L1215 CN**: 声明或定义可调用实体 `IODEF`。
- **L1216 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1216 CN**: 执行语句 `IoStatementState &io{*cookie};`。

### Lines 1217-1248

````cpp
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  if (!handler.InError()) {
    io.CompleteOperation();
  }
  if (const auto *formatted{
          io.get_if<FormattedIoStatementState<Direction::Input>>()}) {
    return formatted->GetEditDescriptorChars();
  } else if (!io.get_if<NoopStatementState>() &&
      !io.get_if<ErroneousIoStatementState>()) {
    handler.Crash("GetIoSize() called for an I/O statement that is not a "
                  "formatted READ()");
  }
  return 0;
}

std::size_t IODEF(GetIoLength)(Cookie cookie) {
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  if (!handler.InError()) {
    io.CompleteOperation();
  }
  if (const auto *inq{io.get_if<InquireIOLengthState>()}) {
    return inq->bytes();
  } else if (!io.get_if<NoopStatementState>() &&
      !io.get_if<ErroneousIoStatementState>()) {
    handler.Crash("GetIoLength() called for an I/O statement that is not "
                  "INQUIRE(IOLENGTH=)");
  }
  return 0;
}

void IODEF(GetIoMsg)(Cookie cookie, char *msg, std::size_t length) {
````

- **L1217 EN**: Executes statement involving `GetIoErrorHandler`.
  **L1217 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L1218 EN**: Introduces conditional control flow with an `if` statement.
  **L1218 CN**: 通过 `if` 语句引入条件控制流。
- **L1219 EN**: Executes statement involving `CompleteOperation`.
  **L1219 CN**: 执行涉及 `CompleteOperation` 的语句。
- **L1220 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1220 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1221 EN**: Introduces conditional control flow with an `if` statement.
  **L1221 CN**: 通过 `if` 语句引入条件控制流。
- **L1222 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1222 CN**: 延续周围的声明、表达式或控制流结构。
- **L1223 EN**: Returns from the current function, often propagating a computed result.
  **L1223 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1224 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1224 CN**: 延续周围的声明、表达式或控制流结构。
- **L1225 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1225 CN**: 延续周围的声明、表达式或控制流结构。
- **L1226 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1226 CN**: 延续周围的声明、表达式或控制流结构。
- **L1227 EN**: Executes statement involving `READ`.
  **L1227 CN**: 执行涉及 `READ` 的语句。
- **L1228 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1228 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1229 EN**: Returns from the current function, often propagating a computed result.
  **L1229 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1230 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1230 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1231 EN**: Blank line separates nearby declarations or logic blocks.
  **L1231 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1232 EN**: Declares or defines callable `IODEF`.
  **L1232 CN**: 声明或定义可调用实体 `IODEF`。
- **L1233 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1233 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1234 EN**: Executes statement involving `GetIoErrorHandler`.
  **L1234 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L1235 EN**: Introduces conditional control flow with an `if` statement.
  **L1235 CN**: 通过 `if` 语句引入条件控制流。
- **L1236 EN**: Executes statement involving `CompleteOperation`.
  **L1236 CN**: 执行涉及 `CompleteOperation` 的语句。
- **L1237 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1237 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1238 EN**: Introduces conditional control flow with an `if` statement.
  **L1238 CN**: 通过 `if` 语句引入条件控制流。
- **L1239 EN**: Returns from the current function, often propagating a computed result.
  **L1239 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1240 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1240 CN**: 延续周围的声明、表达式或控制流结构。
- **L1241 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1241 CN**: 延续周围的声明、表达式或控制流结构。
- **L1242 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1242 CN**: 延续周围的声明、表达式或控制流结构。
- **L1243 EN**: Executes statement involving `INQUIRE`.
  **L1243 CN**: 执行涉及 `INQUIRE` 的语句。
- **L1244 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1244 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1245 EN**: Returns from the current function, often propagating a computed result.
  **L1245 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1246 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1246 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1247 EN**: Blank line separates nearby declarations or logic blocks.
  **L1247 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1248 EN**: Declares or defines callable `IODEF`.
  **L1248 CN**: 声明或定义可调用实体 `IODEF`。

### Lines 1249-1280

````cpp
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  if (!handler.InError()) {
    io.CompleteOperation();
  }
  if (handler.InError()) { // leave "msg" alone when no error
    handler.GetIoMsg(msg, length);
  }
}

AsynchronousId IODEF(GetAsynchronousId)(Cookie cookie) {
  IoStatementState &io{*cookie};
  IoErrorHandler &handler{io.GetIoErrorHandler()};
  if (auto *ext{io.get_if<ExternalIoStatementBase>()}) {
    return ext->asynchronousID();
  } else if (!io.get_if<NoopStatementState>() &&
      !io.get_if<ErroneousIoStatementState>()) {
    handler.Crash(
        "GetAsynchronousId() called when not in an external I/O statement");
  }
  return 0;
}

bool IODEF(InquireCharacter)(Cookie cookie, InquiryKeywordHash inquiry,
    char *result, std::size_t length) {
  IoStatementState &io{*cookie};
  return io.Inquire(inquiry, result, length);
}

bool IODEF(InquireLogical)(
    Cookie cookie, InquiryKeywordHash inquiry, bool &result) {
  IoStatementState &io{*cookie};
````

- **L1249 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1249 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1250 EN**: Executes statement involving `GetIoErrorHandler`.
  **L1250 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L1251 EN**: Introduces conditional control flow with an `if` statement.
  **L1251 CN**: 通过 `if` 语句引入条件控制流。
- **L1252 EN**: Executes statement involving `CompleteOperation`.
  **L1252 CN**: 执行涉及 `CompleteOperation` 的语句。
- **L1253 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1253 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1254 EN**: Introduces conditional control flow with an `if` statement.
  **L1254 CN**: 通过 `if` 语句引入条件控制流。
- **L1255 EN**: Executes statement involving `GetIoMsg`.
  **L1255 CN**: 执行涉及 `GetIoMsg` 的语句。
- **L1256 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1256 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1257 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1257 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1258 EN**: Blank line separates nearby declarations or logic blocks.
  **L1258 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1259 EN**: Declares or defines callable `IODEF`.
  **L1259 CN**: 声明或定义可调用实体 `IODEF`。
- **L1260 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1260 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1261 EN**: Executes statement involving `GetIoErrorHandler`.
  **L1261 CN**: 执行涉及 `GetIoErrorHandler` 的语句。
- **L1262 EN**: Introduces conditional control flow with an `if` statement.
  **L1262 CN**: 通过 `if` 语句引入条件控制流。
- **L1263 EN**: Returns from the current function, often propagating a computed result.
  **L1263 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1264 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1264 CN**: 延续周围的声明、表达式或控制流结构。
- **L1265 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1265 CN**: 延续周围的声明、表达式或控制流结构。
- **L1266 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1266 CN**: 延续周围的声明、表达式或控制流结构。
- **L1267 EN**: Executes statement involving `GetAsynchronousId`.
  **L1267 CN**: 执行涉及 `GetAsynchronousId` 的语句。
- **L1268 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1268 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1269 EN**: Returns from the current function, often propagating a computed result.
  **L1269 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1270 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1270 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1271 EN**: Blank line separates nearby declarations or logic blocks.
  **L1271 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1272 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1272 CN**: 延续周围的声明、表达式或控制流结构。
- **L1273 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1273 CN**: 延续周围的声明、表达式或控制流结构。
- **L1274 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1274 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1275 EN**: Returns from the current function, often propagating a computed result.
  **L1275 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1276 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1276 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1277 EN**: Blank line separates nearby declarations or logic blocks.
  **L1277 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1278 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1278 CN**: 延续周围的声明、表达式或控制流结构。
- **L1279 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1279 CN**: 延续周围的声明、表达式或控制流结构。
- **L1280 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1280 CN**: 执行语句 `IoStatementState &io{*cookie};`。

### Lines 1281-1312

````cpp
  return io.Inquire(inquiry, result);
}

bool IODEF(InquirePendingId)(Cookie cookie, AsynchronousId id, bool &result) {
  IoStatementState &io{*cookie};
  return io.Inquire(HashInquiryKeyword("PENDING"), id, result);
}

bool IODEF(InquireInteger64)(
    Cookie cookie, InquiryKeywordHash inquiry, std::int64_t &result, int kind) {
  IoStatementState &io{*cookie};
  std::int64_t n{0}; // safe "undefined" value
  if (io.Inquire(inquiry, n)) {
    if (SetInteger(result, kind, n)) {
      return true;
    }
    io.GetIoErrorHandler().SignalError(
        "InquireInteger64(): bad INTEGER kind(%d) or out-of-range "
        "value(%jd) for result",
        kind, static_cast<std::intmax_t>(n));
  }
  return false;
}

template <typename INT>
static RT_API_ATTRS enum Iostat CheckUnitNumberInRangeImpl(INT unit,
    bool handleError, char *ioMsg, std::size_t ioMsgLength,
    const char *sourceFile, int sourceLine) {
  static_assert(sizeof(INT) >= sizeof(ExternalUnit),
      "only intended to be used when the INT to ExternalUnit conversion is "
      "narrowing");
  if (unit != static_cast<ExternalUnit>(unit)) {
````

- **L1281 EN**: Returns from the current function, often propagating a computed result.
  **L1281 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1282 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1282 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1283 EN**: Blank line separates nearby declarations or logic blocks.
  **L1283 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1284 EN**: Declares or defines callable `IODEF`.
  **L1284 CN**: 声明或定义可调用实体 `IODEF`。
- **L1285 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1285 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1286 EN**: Returns from the current function, often propagating a computed result.
  **L1286 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1287 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1287 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1288 EN**: Blank line separates nearby declarations or logic blocks.
  **L1288 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1289 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1289 CN**: 延续周围的声明、表达式或控制流结构。
- **L1290 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1290 CN**: 延续周围的声明、表达式或控制流结构。
- **L1291 EN**: Executes statement `IoStatementState &io{*cookie};`.
  **L1291 CN**: 执行语句 `IoStatementState &io{*cookie};`。
- **L1292 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1292 CN**: 延续周围的声明、表达式或控制流结构。
- **L1293 EN**: Introduces conditional control flow with an `if` statement.
  **L1293 CN**: 通过 `if` 语句引入条件控制流。
- **L1294 EN**: Introduces conditional control flow with an `if` statement.
  **L1294 CN**: 通过 `if` 语句引入条件控制流。
- **L1295 EN**: Returns from the current function, often propagating a computed result.
  **L1295 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1296 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1296 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1297 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1297 CN**: 延续周围的声明、表达式或控制流结构。
- **L1298 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1298 CN**: 延续周围的声明、表达式或控制流结构。
- **L1299 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1299 CN**: 延续周围的声明、表达式或控制流结构。
- **L1300 EN**: Executes statement `kind, static_cast<std::intmax_t>(n));`.
  **L1300 CN**: 执行语句 `kind, static_cast<std::intmax_t>(n));`。
- **L1301 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1301 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1302 EN**: Returns from the current function, often propagating a computed result.
  **L1302 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1303 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1303 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1304 EN**: Blank line separates nearby declarations or logic blocks.
  **L1304 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1305 EN**: Begins a template declaration parameterizing subsequent code.
  **L1305 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L1306 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1306 CN**: 延续周围的声明、表达式或控制流结构。
- **L1307 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1307 CN**: 延续周围的声明、表达式或控制流结构。
- **L1308 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1308 CN**: 延续周围的声明、表达式或控制流结构。
- **L1309 EN**: Performs a compile-time assertion to enforce invariants.
  **L1309 CN**: 执行编译期断言以约束不变量。
- **L1310 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1310 CN**: 延续周围的声明、表达式或控制流结构。
- **L1311 EN**: Executes statement `"narrowing");`.
  **L1311 CN**: 执行语句 `"narrowing");`。
- **L1312 EN**: Introduces conditional control flow with an `if` statement.
  **L1312 CN**: 通过 `if` 语句引入条件控制流。

### Lines 1313-1344

````cpp
    Terminator oom{sourceFile, sourceLine};
    IoErrorHandler errorHandler{oom};
    if (handleError) {
      errorHandler.HasIoStat();
      if (ioMsg) {
        errorHandler.HasIoMsg();
      }
    }
    // Only provide the bad unit number in the message if SignalError can print
    // it accurately. Otherwise, the generic IostatUnitOverflow message will be
    // used.
    if constexpr (sizeof(INT) > sizeof(std::intmax_t)) {
      errorHandler.SignalError(IostatUnitOverflow);
    } else if (static_cast<std::intmax_t>(unit) == unit) {
      errorHandler.SignalError(IostatUnitOverflow,
          "UNIT number %jd is out of range", static_cast<std::intmax_t>(unit));
    } else {
      errorHandler.SignalError(IostatUnitOverflow);
    }
    if (ioMsg) {
      errorHandler.GetIoMsg(ioMsg, ioMsgLength);
    }
    return static_cast<enum Iostat>(errorHandler.GetIoStat());
  }
  return IostatOk;
}

enum Iostat IODEF(CheckUnitNumberInRange64)(std::int64_t unit, bool handleError,
    char *ioMsg, std::size_t ioMsgLength, const char *sourceFile,
    int sourceLine) {
  return CheckUnitNumberInRangeImpl(
      unit, handleError, ioMsg, ioMsgLength, sourceFile, sourceLine);
````

- **L1313 EN**: Executes statement `Terminator oom{sourceFile, sourceLine};`.
  **L1313 CN**: 执行语句 `Terminator oom{sourceFile, sourceLine};`。
- **L1314 EN**: Executes statement `IoErrorHandler errorHandler{oom};`.
  **L1314 CN**: 执行语句 `IoErrorHandler errorHandler{oom};`。
- **L1315 EN**: Introduces conditional control flow with an `if` statement.
  **L1315 CN**: 通过 `if` 语句引入条件控制流。
- **L1316 EN**: Executes statement involving `HasIoStat`.
  **L1316 CN**: 执行涉及 `HasIoStat` 的语句。
- **L1317 EN**: Introduces conditional control flow with an `if` statement.
  **L1317 CN**: 通过 `if` 语句引入条件控制流。
- **L1318 EN**: Executes statement involving `HasIoMsg`.
  **L1318 CN**: 执行涉及 `HasIoMsg` 的语句。
- **L1319 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1319 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1320 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1320 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1321 EN**: Comment documents intent or context: `Only provide the bad unit number in the message if SignalError can print`.
  **L1321 CN**: 注释记录了意图或上下文：`Only provide the bad unit number in the message if SignalError can print`。
- **L1322 EN**: Comment documents intent or context: `it accurately. Otherwise, the generic IostatUnitOverflow message will be`.
  **L1322 CN**: 注释记录了意图或上下文：`it accurately. Otherwise, the generic IostatUnitOverflow message will be`。
- **L1323 EN**: Comment documents intent or context: `used.`.
  **L1323 CN**: 注释记录了意图或上下文：`used.`。
- **L1324 EN**: Introduces conditional control flow with an `if` statement.
  **L1324 CN**: 通过 `if` 语句引入条件控制流。
- **L1325 EN**: Executes statement involving `SignalError`.
  **L1325 CN**: 执行涉及 `SignalError` 的语句。
- **L1326 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1326 CN**: 延续周围的声明、表达式或控制流结构。
- **L1327 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1327 CN**: 延续周围的声明、表达式或控制流结构。
- **L1328 EN**: Executes statement `"UNIT number %jd is out of range", static_cast<std::intmax_t>(unit));`.
  **L1328 CN**: 执行语句 `"UNIT number %jd is out of range", static_cast<std::intmax_t>(unit));`。
- **L1329 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1329 CN**: 延续周围的声明、表达式或控制流结构。
- **L1330 EN**: Executes statement involving `SignalError`.
  **L1330 CN**: 执行涉及 `SignalError` 的语句。
- **L1331 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1331 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1332 EN**: Introduces conditional control flow with an `if` statement.
  **L1332 CN**: 通过 `if` 语句引入条件控制流。
- **L1333 EN**: Executes statement involving `GetIoMsg`.
  **L1333 CN**: 执行涉及 `GetIoMsg` 的语句。
- **L1334 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1334 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1335 EN**: Returns from the current function, often propagating a computed result.
  **L1335 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1336 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1336 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1337 EN**: Returns from the current function, often propagating a computed result.
  **L1337 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1338 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1338 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1339 EN**: Blank line separates nearby declarations or logic blocks.
  **L1339 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1340 EN**: Declares or defines enum `Iostat`.
  **L1340 CN**: 声明或定义 enum `Iostat`。
- **L1341 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1341 CN**: 延续周围的声明、表达式或控制流结构。
- **L1342 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1342 CN**: 延续周围的声明、表达式或控制流结构。
- **L1343 EN**: Returns from the current function, often propagating a computed result.
  **L1343 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1344 EN**: Executes statement `unit, handleError, ioMsg, ioMsgLength, sourceFile, sourceLine);`.
  **L1344 CN**: 执行语句 `unit, handleError, ioMsg, ioMsgLength, sourceFile, sourceLine);`。

### Lines 1345-1357

````cpp
}

#ifdef __SIZEOF_INT128__
enum Iostat IODEF(CheckUnitNumberInRange128)(common::int128_t unit,
    bool handleError, char *ioMsg, std::size_t ioMsgLength,
    const char *sourceFile, int sourceLine) {
  return CheckUnitNumberInRangeImpl(
      unit, handleError, ioMsg, ioMsgLength, sourceFile, sourceLine);
}
#endif

RT_EXT_API_GROUP_END
} // namespace Fortran::runtime::io
````

- **L1345 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1345 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1346 EN**: Blank line separates nearby declarations or logic blocks.
  **L1346 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1347 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L1347 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L1348 EN**: Declares or defines enum `Iostat`.
  **L1348 CN**: 声明或定义 enum `Iostat`。
- **L1349 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1349 CN**: 延续周围的声明、表达式或控制流结构。
- **L1350 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1350 CN**: 延续周围的声明、表达式或控制流结构。
- **L1351 EN**: Returns from the current function, often propagating a computed result.
  **L1351 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1352 EN**: Executes statement `unit, handleError, ioMsg, ioMsgLength, sourceFile, sourceLine);`.
  **L1352 CN**: 执行语句 `unit, handleError, ioMsg, ioMsgLength, sourceFile, sourceLine);`。
- **L1353 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1353 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1354 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L1354 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L1355 EN**: Blank line separates nearby declarations or logic blocks.
  **L1355 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1356 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1356 CN**: 延续周围的声明、表达式或控制流结构。
- **L1357 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1357 CN**: 延续周围的声明、表达式或控制流结构。

## Key Concepts / 关键概念

- **Scale / 规模**: The file contains approximately 1357 source lines, which suggests a substantial implementation unit. / 该文件约有 1357 行源码，说明它是一个较大的实现单元。
- **Fortran execution semantics / Fortran 执行语义**: The implementation materializes language-level behavior such as descriptors, I/O state, allocation, reductions, and error handling. / 实现把语言层面的行为具体化，例如描述符、I/O 状态、分配、归约与错误处理。
- **Runtime layering / 运行时分层**: Source files in this area usually bridge generated code, public runtime headers, and lower-level system facilities. / 此处源码通常桥接生成代码、公共运行时头文件以及更底层的系统设施。
- **Interface surface / 接口表面**: Direct includes such as `flang/Runtime/io-api.h`, `descriptor-io.h`, `edit-input.h`, `edit-output.h` show which nearby abstractions this file relies on first. / 直接包含的头文件（如 `flang/Runtime/io-api.h`, `descriptor-io.h`, `edit-input.h`, `edit-output.h`）展示了此文件首先依赖的周边抽象。
- **Primary entry points / 主要入口点**: Notable callables include `constexpr`, `IODEF`, `move`. / 值得关注的可调用实体包括 `constexpr`, `IODEF`, `move`。
- **Core types / 核心类型**: Important declared or referenced types include `Iostat`. / 重要的已声明或被引用类型包括 `Iostat`。
- **Namespaces / 命名空间**: The code uses namespaces such as `Fortran` to organize symbols. / 代码使用 `Fortran` 等命名空间来组织符号。

## Dependencies / 依赖关系

- **Project headers / 项目头文件**: `flang/Runtime/io-api.h`, `descriptor-io.h`, `edit-input.h`, `edit-output.h`, `io-api-common.h`, `unit.h`, `flang-rt/runtime/descriptor.h`, `flang-rt/runtime/environment.h`, `flang-rt/runtime/format.h`, `flang-rt/runtime/io-stmt.h`, `flang-rt/runtime/memory.h`, `flang-rt/runtime/terminator.h`. These provide subsystem-specific declarations. / 这些头文件提供子系统专用声明。
- **Standard or platform headers / 标准库或平台头文件**: `cstdlib`, `memory`. They connect the file to language-runtime or OS services. / 它们将该文件连接到语言运行时或操作系统服务。
- **Callable surface / 可调用表面**: Functions or methods defined here include `constexpr`, `IODEF`, `move`. These are likely the main entry points exported to nearby code. / 此处定义的函数或方法包括 `constexpr`, `IODEF`, `move`，它们通常是对周边代码暴露的主要入口。
- **Type coupling / 类型耦合**: Declared types such as `Iostat` capture the data model shared with dependent code. / `Iostat` 等声明类型体现了与依赖方共享的数据模型。
