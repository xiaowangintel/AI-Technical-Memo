# TokenRewriter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Rewrite/TokenRewriter.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: This file implements the TokenRewriter class, which is used for code.
- **Purpose (CN) / 用途（中文）**: 该文件在 Clang 的源码重写子系统中实现与 TokenRewriter 相关的逻辑。对应英文说明：This file implements the TokenRewriter class, which is used for code。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
//===- TokenRewriter.cpp - Token-based code rewriting interface -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file implements the TokenRewriter class, which is used for code
//  transformations.
//
//===----------------------------------------------------------------------===//

#include "clang/Rewrite/Core/TokenRewriter.h"
#include "clang/Basic/SourceManager.h"
#include "clang/Lex/Lexer.h"
#include "clang/Lex/ScratchBuffer.h"
#include "clang/Lex/Token.h"
#include <cassert>
#include <cstring>
#include <map>
#include <utility>

using namespace clang;

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
- **L13**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L14**: Includes `clang/Rewrite/Core/TokenRewriter.h` so this translation unit can use declarations from that header. / 引入 `clang/Rewrite/Core/TokenRewriter.h`，使当前编译单元能够使用该头文件中的声明。
- **L15**: Includes `clang/Basic/SourceManager.h` so this translation unit can use declarations from that header. / 引入 `clang/Basic/SourceManager.h`，使当前编译单元能够使用该头文件中的声明。
- **L16**: Includes `clang/Lex/Lexer.h` so this translation unit can use declarations from that header. / 引入 `clang/Lex/Lexer.h`，使当前编译单元能够使用该头文件中的声明。
- **L17**: Includes `clang/Lex/ScratchBuffer.h` so this translation unit can use declarations from that header. / 引入 `clang/Lex/ScratchBuffer.h`，使当前编译单元能够使用该头文件中的声明。
- **L18**: Includes `clang/Lex/Token.h` so this translation unit can use declarations from that header. / 引入 `clang/Lex/Token.h`，使当前编译单元能够使用该头文件中的声明。
- **L19**: Includes `cassert` so this translation unit can use declarations from that header. / 引入 `cassert`，使当前编译单元能够使用该头文件中的声明。
- **L20**: Includes `cstring` so this translation unit can use declarations from that header. / 引入 `cstring`，使当前编译单元能够使用该头文件中的声明。
- **L21**: Includes `map` so this translation unit can use declarations from that header. / 引入 `map`，使当前编译单元能够使用该头文件中的声明。
- **L22**: Includes `utility` so this translation unit can use declarations from that header. / 引入 `utility`，使当前编译单元能够使用该头文件中的声明。
- **L23**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L24**: Imports namespace `clang` into the current scope for shorter symbol references. / 将命名空间 `clang` 导入当前作用域，以便更简洁地引用符号。
- **L25**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 26-50 / 第 26-50 行

```cpp
TokenRewriter::TokenRewriter(FileID FID, SourceManager &SM,
                             const LangOptions &LangOpts) {
  ScratchBuf.reset(new ScratchBuffer(SM));

  // Create a lexer to lex all the tokens of the main file in raw mode.
  llvm::MemoryBufferRef FromFile = SM.getBufferOrFake(FID);
  Lexer RawLex(FID, FromFile, SM, LangOpts);

  // Return all comments and whitespace as tokens.
  RawLex.SetKeepWhitespaceMode(true);

  // Lex the file, populating our datastructures.
  Token RawTok;
  RawLex.LexFromRawLexer(RawTok);
  while (RawTok.isNot(tok::eof)) {
#if 0
    if (Tok.is(tok::raw_identifier)) {
      // Look up the identifier info for the token.  This should use
      // IdentifierTable directly instead of PP.
      PP.LookUpIdentifierInfo(Tok);
    }
#endif

    AddToken(RawTok, TokenList.end());
    RawLex.LexFromRawLexer(RawTok);
```

- **L26**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L27**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L28**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L29**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L30**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L31**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L32**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L33**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L34**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L35**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L36**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L37**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L38**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L39**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L40**: Starts a loop that repeats while the condition remains true. / 开始一个在条件为真时持续执行的循环。
- **L41**: Starts a conditional-compilation region controlled by build or feature macros. / 开始一个由构建宏或特性宏控制的条件编译区域。
- **L42**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L43**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L44**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L45**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L46**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L47**: Ends the active conditional-compilation region. / 结束当前的条件编译区域。
- **L48**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L49**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L50**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。

### Lines 51-75 / 第 51-75 行

```cpp
  }
}

TokenRewriter::~TokenRewriter() = default;

/// RemapIterator - Convert from token_iterator (a const iterator) to
/// TokenRefTy (a non-const iterator).
TokenRewriter::TokenRefTy TokenRewriter::RemapIterator(token_iterator I) {
  if (I == token_end()) return TokenList.end();

  // FIXME: This is horrible, we should use our own list or something to avoid
  // this.
  std::map<SourceLocation, TokenRefTy>::iterator MapIt =
    TokenAtLoc.find(I->getLocation());
  assert(MapIt != TokenAtLoc.end() && "iterator not in rewriter?");
  return MapIt->second;
}

/// AddToken - Add the specified token into the Rewriter before the other
/// position.
TokenRewriter::TokenRefTy
TokenRewriter::AddToken(const Token &T, TokenRefTy Where) {
  Where = TokenList.insert(Where, T);

  bool InsertSuccess = TokenAtLoc.insert(std::make_pair(T.getLocation(),
```

- **L51**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L52**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L53**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L54**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L55**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L56**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L57**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L58**: This line completes a function, method, or callable signature and opens its body. / 这一行完成函数、方法或可调用对象的签名，并打开其函数体。
- **L59**: Starts a conditional branch that executes only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L60**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L61**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L62**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L63**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L64**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L65**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L66**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L67**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L68**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L69**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L70**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L71**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L72**: This line completes a function, method, or callable signature and opens its body. / 这一行完成函数、方法或可调用对象的签名，并打开其函数体。
- **L73**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L74**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L75**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 76-99 / 第 76-99 行

```cpp
                                                        Where)).second;
  assert(InsertSuccess && "Token location already in rewriter!");
  (void)InsertSuccess;
  return Where;
}

TokenRewriter::token_iterator
TokenRewriter::AddTokenBefore(token_iterator I, const char *Val) {
  unsigned Len = strlen(Val);

  // Plop the string into the scratch buffer, then create a token for this
  // string.
  Token Tok;
  Tok.startToken();
  const char *Spelling;
  Tok.setLocation(ScratchBuf->getToken(Val, Len, Spelling));
  Tok.setLength(Len);

  // TODO: Form a whole lexer around this and relex the token!  For now, just
  // set kind to tok::unknown.
  Tok.setKind(tok::unknown);

  return AddToken(Tok, RemapIterator(I));
}
```

- **L76**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L77**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L78**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L79**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L80**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L81**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L82**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L83**: This line completes a function, method, or callable signature and opens its body. / 这一行完成函数、方法或可调用对象的签名，并打开其函数体。
- **L84**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L85**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L86**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L87**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L88**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L89**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L90**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L91**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L92**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L93**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L94**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L95**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L96**: This line declares a function, method, or callable interface without defining the body here. / 这一行声明函数、方法或可调用接口，但不在此处给出函数体。
- **L97**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L98**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L99**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file is a implementation unit in Clang's **Rewrite** subsystem. / 该文件是 Clang **Rewrite** 子系统中的实现单元。
- **Scale / 规模**: 99 lines and 9 direct includes. / 共 99 行，并直接包含 9 个头文件。
- **Subsystem focus / 子系统关注点**: source rewriting, text replacement, buffer updates. / 源码重写、文本替换、缓冲区更新。
- **Visible entry points / 关键入口**: `reset`, `getBufferOrFake`, `RawLex`, `SetKeepWhitespaceMode`, `LexFromRawLexer`, `LookUpIdentifierInfo`, `AddToken`, `TokenRewriter::RemapIterator`, `find`, `assert`. / 可见的关键入口包括 `reset`、`getBufferOrFake`、`RawLex`、`SetKeepWhitespaceMode`、`LexFromRawLexer`、`LookUpIdentifierInfo`、`AddToken`、`TokenRewriter::RemapIterator`、`find`、`assert`。

## Dependencies / 依赖关系

- **Clang headers / Clang 头文件**: `clang/Rewrite/Core/TokenRewriter.h`, `clang/Basic/SourceManager.h`, `clang/Lex/Lexer.h`, `clang/Lex/ScratchBuffer.h`, `clang/Lex/Token.h`.
- **System/other headers / 系统或其他头文件**: `cassert`, `cstring`, `map`, `utility`.
- **Referenced routines / 关键例程**: `reset`, `getBufferOrFake`, `RawLex`, `SetKeepWhitespaceMode`, `LexFromRawLexer`, `LookUpIdentifierInfo`, `AddToken`, `TokenRewriter::RemapIterator`, `find`, `assert`.
