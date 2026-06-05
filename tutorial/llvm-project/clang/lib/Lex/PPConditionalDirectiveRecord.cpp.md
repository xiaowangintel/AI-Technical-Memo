# PPConditionalDirectiveRecord.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Lex/PPConditionalDirectiveRecord.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: This file implements the PPConditionalDirectiveRecord class, which maintains a record of conditional directive regions.
- **Purpose (CN)**: 该文件在 Clang 的词法分析与预处理子系统中实现与 PPConditionalDirectiveRecord 相关的逻辑。对应英文说明：This file implements the PPConditionalDirectiveRecord class, which maintains a record of conditional directive regions。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
//===--- PPConditionalDirectiveRecord.h - Preprocessing Directives-*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file implements the PPConditionalDirectiveRecord class, which maintains
//  a record of conditional directive regions.
//
//===----------------------------------------------------------------------===//
#include "clang/Lex/PPConditionalDirectiveRecord.h"
#include "llvm/Support/Capacity.h"

using namespace clang;

PPConditionalDirectiveRecord::PPConditionalDirectiveRecord(SourceManager &SM)
  : SourceMgr(SM) {
  CondDirectiveStack.push_back(SourceLocation());
}

bool PPConditionalDirectiveRecord::rangeIntersectsConditionalDirective(
                                                      SourceRange Range) const {
  if (Range.isInvalid())
```

- **L1**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L2**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L3**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L4**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L5**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L6**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L7**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L8**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L9**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L10**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L11**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L12**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L13**: Includes `clang/Lex/PPConditionalDirectiveRecord.h` so this translation unit can use declarations from that header. / 引入 `clang/Lex/PPConditionalDirectiveRecord.h`，使当前编译单元能够使用该头文件中的声明。
- **L14**: Includes `llvm/Support/Capacity.h` so this translation unit can use declarations from that header. / 引入 `llvm/Support/Capacity.h`，使当前编译单元能够使用该头文件中的声明。
- **L15**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L16**: Imports namespace `clang` into the current scope for shorter symbol references. / 将命名空间 `clang` 导入当前作用域，以便更简洁地引用符号。
- **L17**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L18**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L19**: This line completes a function, method, or callable signature and opens its body. / 这一行完成函数、方法或可调用对象的签名，并打开其函数体。
- **L20**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L21**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L22**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L23**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L24**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L25**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。

### Lines 26-50 / 第 26-50 行

```cpp
    return false;

  CondDirectiveLocsTy::const_iterator low = llvm::lower_bound(
      CondDirectiveLocs, Range.getBegin(), CondDirectiveLoc::Comp(SourceMgr));
  if (low == CondDirectiveLocs.end())
    return false;

  if (SourceMgr.isBeforeInTranslationUnit(Range.getEnd(), low->getLoc()))
    return false;

  CondDirectiveLocsTy::const_iterator
    upp = std::upper_bound(low, CondDirectiveLocs.end(),
                           Range.getEnd(), CondDirectiveLoc::Comp(SourceMgr));
  SourceLocation uppRegion;
  if (upp != CondDirectiveLocs.end())
    uppRegion = upp->getRegionLoc();

  return low->getRegionLoc() != uppRegion;
}

SourceLocation PPConditionalDirectiveRecord::findConditionalDirectiveRegionLoc(
                                                     SourceLocation Loc) const {
  if (Loc.isInvalid())
    return SourceLocation();
  if (CondDirectiveLocs.empty())
```

- **L26**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L27**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L28**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L29**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L30**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L31**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L32**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L33**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L34**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L35**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L36**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L37**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L38**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L39**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L40**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L41**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L42**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L43**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L44**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L45**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L46**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L47**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L48**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L49**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L50**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。

### Lines 51-75 / 第 51-75 行

```cpp
    return SourceLocation();

  if (SourceMgr.isBeforeInTranslationUnit(CondDirectiveLocs.back().getLoc(),
                                          Loc))
    return CondDirectiveStack.back();

  CondDirectiveLocsTy::const_iterator low = llvm::lower_bound(
      CondDirectiveLocs, Loc, CondDirectiveLoc::Comp(SourceMgr));
  assert(low != CondDirectiveLocs.end());
  return low->getRegionLoc();
}

void PPConditionalDirectiveRecord::addCondDirectiveLoc(
                                                      CondDirectiveLoc DirLoc) {
  // Ignore directives in system headers.
  if (SourceMgr.isInSystemHeader(DirLoc.getLoc()))
    return;

  assert(CondDirectiveLocs.empty() ||
         SourceMgr.isBeforeInTranslationUnit(CondDirectiveLocs.back().getLoc(),
                                             DirLoc.getLoc()));
  CondDirectiveLocs.push_back(DirLoc);
}

void PPConditionalDirectiveRecord::If(SourceLocation Loc,
```

- **L51**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L52**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L53**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L54**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L55**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L56**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L57**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L58**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L59**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L60**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L61**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L62**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L63**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L64**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L65**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L66**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L67**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L68**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L69**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L70**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L71**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L72**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L73**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L74**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L75**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 76-100 / 第 76-100 行

```cpp
                                      SourceRange ConditionRange,
                                      ConditionValueKind ConditionValue) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.push_back(Loc);
}

void PPConditionalDirectiveRecord::Ifdef(SourceLocation Loc,
                                         const Token &MacroNameTok,
                                         const MacroDefinition &MD) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.push_back(Loc);
}

void PPConditionalDirectiveRecord::Ifndef(SourceLocation Loc,
                                          const Token &MacroNameTok,
                                          const MacroDefinition &MD) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.push_back(Loc);
}

void PPConditionalDirectiveRecord::Elif(SourceLocation Loc,
                                        SourceRange ConditionRange,
                                        ConditionValueKind ConditionValue,
                                        SourceLocation IfLoc) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
```

- **L76**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L77**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L78**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L79**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L80**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L81**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L82**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L83**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L84**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L85**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L86**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L87**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L88**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L89**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L90**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L91**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L92**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L93**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L94**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L95**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L96**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L97**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L98**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L99**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L100**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。

### Lines 101-125 / 第 101-125 行

```cpp
  CondDirectiveStack.back() = Loc;
}

void PPConditionalDirectiveRecord::Elifdef(SourceLocation Loc, const Token &,
                                           const MacroDefinition &) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.back() = Loc;
}
void PPConditionalDirectiveRecord::Elifdef(SourceLocation Loc, SourceRange,
                                           SourceLocation) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.back() = Loc;
}

void PPConditionalDirectiveRecord::Elifndef(SourceLocation Loc, const Token &,
                                            const MacroDefinition &) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.back() = Loc;
}
void PPConditionalDirectiveRecord::Elifndef(SourceLocation Loc, SourceRange,
                                            SourceLocation) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.back() = Loc;
}

```

- **L101**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L102**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L103**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L104**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L105**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L106**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L107**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L108**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L109**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L110**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L111**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L112**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L113**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L114**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L115**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L116**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L117**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L118**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L119**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L120**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L121**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L122**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L123**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L124**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L125**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 126-141 / 第 126-141 行

```cpp
void PPConditionalDirectiveRecord::Else(SourceLocation Loc,
                                        SourceLocation IfLoc) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  CondDirectiveStack.back() = Loc;
}

void PPConditionalDirectiveRecord::Endif(SourceLocation Loc,
                                         SourceLocation IfLoc) {
  addCondDirectiveLoc(CondDirectiveLoc(Loc, CondDirectiveStack.back()));
  assert(!CondDirectiveStack.empty());
  CondDirectiveStack.pop_back();
}

size_t PPConditionalDirectiveRecord::getTotalMemory() const {
  return llvm::capacity_in_bytes(CondDirectiveLocs);
}
```

- **L126**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L127**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L128**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L129**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L130**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L131**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L132**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L133**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L134**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L135**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L136**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L137**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L138**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L139**: This line completes a function, method, or callable signature and opens its body. / 这一行完成函数、方法或可调用对象的签名，并打开其函数体。
- **L140**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L141**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file is a implementation unit in Clang's **Lex** subsystem. / 该文件是 Clang **Lex** 子系统中的实现单元。
- **Scale / 规模**: 141 lines and 2 direct includes. / 共 141 行，并直接包含 2 个头文件。
- **Subsystem focus / 子系统关注点**: tokenization, source buffer handling, preprocessor integration. / 词法切分、源码缓冲处理、预处理器集成。
- **Visible entry points / 关键入口**: `SourceMgr`, `push_back`, `getBegin`, `getEnd`, `getRegionLoc`, `SourceLocation`, `back`, `CondDirectiveLoc::Comp`, `assert`, `getLoc`. / 可见的关键入口包括 `SourceMgr`、`push_back`、`getBegin`、`getEnd`、`getRegionLoc`、`SourceLocation`、`back`、`CondDirectiveLoc::Comp`、`assert`、`getLoc`。

## Dependencies / 依赖关系

- **Clang headers / Clang 头文件**: `clang/Lex/PPConditionalDirectiveRecord.h`.
- **LLVM headers / LLVM 头文件**: `llvm/Support/Capacity.h`.
- **Referenced routines / 关键例程**: `SourceMgr`, `push_back`, `getBegin`, `getEnd`, `getRegionLoc`, `SourceLocation`, `back`, `CondDirectiveLoc::Comp`, `assert`, `getLoc`.
