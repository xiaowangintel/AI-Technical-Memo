# ScriptLexer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lld/ELF/ScriptLexer.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: This file defines a lexer for the linker script.
- **Purpose (CN) / 用途（中文）**: 实现 ELF 链接器逻辑，例如符号解析、节布局、重定位以及驱动行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行

```cpp
   1: //===- ScriptLexer.cpp ----------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines a lexer for the linker script.
  10: //
  11: // The linker script's grammar is not complex but ambiguous due to the
  12: // lack of the formal specification of the language. What we are trying to
  13: // do in this and other files in LLD is to make a "reasonable" linker
  14: // script processor.
  15: //
  16: // Among simplicity, compatibility and efficiency, we put the most
  17: // emphasis on simplicity when we wrote this lexer. Compatibility with the
  18: // GNU linkers is important, but we did not try to clone every tiny corner
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
- **L13**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L14**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L15**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L16**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L17**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L18**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

### Lines 19-28 / 第 19-28 行

```cpp
  19: // case of their lexers, as even ld.bfd and ld.gold are subtly different
  20: // in various corner cases. We do not care much about efficiency because
  21: // the time spent in parsing linker scripts is usually negligible.
  22: //
  23: // Overall, this lexer works fine for most linker scripts. There might
  24: // be room for improving compatibility, but that's probably not at the
  25: // top of our todo list.
  26: //
  27: //===----------------------------------------------------------------------===//
  28: 
```

- **L19**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L20**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L21**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L22**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L23**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L24**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L25**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L26**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L27**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L28**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 29-39 / 第 29-39 行

```cpp
  29: #include "ScriptLexer.h"
  30: #include "Config.h"
  31: #include "llvm/ADT/Twine.h"
  32: #include "llvm/Support/ErrorHandling.h"
  33: #include "llvm/Support/FileSystem.h"
  34: #include "llvm/Support/Path.h"
  35: 
  36: using namespace llvm;
  37: using namespace lld;
  38: using namespace lld::elf;
  39: 
```

- **L29**: Includes \`ScriptLexer.h\` so this file can use declarations from that header. / 引入 \`ScriptLexer.h\`，使当前文件能够使用该头文件中的声明。
- **L30**: Includes \`Config.h\` so this file can use declarations from that header. / 引入 \`Config.h\`，使当前文件能够使用该头文件中的声明。
- **L31**: Includes \`llvm/ADT/Twine.h\` so this file can use declarations from that header. / 引入 \`llvm/ADT/Twine.h\`，使当前文件能够使用该头文件中的声明。
- **L32**: Includes \`llvm/Support/ErrorHandling.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/ErrorHandling.h\`，使当前文件能够使用该头文件中的声明。
- **L33**: Includes \`llvm/Support/FileSystem.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/FileSystem.h\`，使当前文件能够使用该头文件中的声明。
- **L34**: Includes \`llvm/Support/Path.h\` so this file can use declarations from that header. / 引入 \`llvm/Support/Path.h\`，使当前文件能够使用该头文件中的声明。
- **L35**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L36**: Imports namespace \`llvm\` into the current scope for shorter symbol references. / 将命名空间 \`llvm\` 导入当前作用域，以便更简洁地引用符号。
- **L37**: Imports namespace \`lld\` into the current scope for shorter symbol references. / 将命名空间 \`lld\` 导入当前作用域，以便更简洁地引用符号。
- **L38**: Imports namespace \`lld::elf\` into the current scope for shorter symbol references. / 将命名空间 \`lld::elf\` 导入当前作用域，以便更简洁地引用符号。
- **L39**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 40-53 / 第 40-53 行

```cpp
  40: ScriptLexer::Buffer::Buffer(Ctx &ctx, MemoryBufferRef mb)
  41:     : s(mb.getBuffer()), filename(mb.getBufferIdentifier()),
  42:       begin(mb.getBufferStart()) {
  43:   if (ctx.arg.sysroot == "")
  44:     return;
  45:   StringRef path = filename;
  46:   for (; !path.empty(); path = sys::path::parent_path(path)) {
  47:     if (!sys::fs::equivalent(ctx.arg.sysroot, path))
  48:       continue;
  49:     isUnderSysroot = true;
  50:     return;
  51:   }
  52: }
  53: 
```

- **L40**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L41**: Continues a multi-line declaration, call, or initializer. / 继续一个跨多行的声明、调用或初始化。
- **L42**: Defines function or method \`begin\`. / 定义函数或方法 \`begin\`。
- **L43**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L44**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L45**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L46**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L47**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L48**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L49**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L50**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L51**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L52**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L53**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 54-67 / 第 54-67 行

```cpp
  54: ScriptLexer::ScriptLexer(Ctx &ctx, MemoryBufferRef mb)
  55:     : ctx(ctx), curBuf(ctx, mb) {
  56:   activeFilenames.insert(mb.getBufferIdentifier());
  57: }
  58: 
  59: // Returns a whole line containing the current token.
  60: StringRef ScriptLexer::getLine() {
  61:   StringRef s(curBuf.begin, curBuf.s.end() - curBuf.begin);
  62:   size_t pos = s.rfind('\n', prevTok.data() - s.data());
  63:   if (pos != StringRef::npos)
  64:     s = s.substr(pos + 1);
  65:   return s.substr(0, s.find_first_of("\r\n"));
  66: }
  67: 
```

- **L54**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L55**: Defines function or method \`ctx\`. / 定义函数或方法 \`ctx\`。
- **L56**: Declares function or method \`insert\`. / 声明函数或方法 \`insert\`。
- **L57**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L58**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L59**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L60**: Defines function or method \`getLine\`. / 定义函数或方法 \`getLine\`。
- **L61**: Declares function or method \`s\`. / 声明函数或方法 \`s\`。
- **L62**: Declares function or method \`rfind\`. / 声明函数或方法 \`rfind\`。
- **L63**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L64**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L65**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L66**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L67**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 68-76 / 第 68-76 行

```cpp
  68: // Returns 0-based column number of the current token.
  69: size_t ScriptLexer::getColumnNumber() {
  70:   return prevTok.data() - getLine().data();
  71: }
  72: 
  73: std::string ScriptLexer::getCurrentLocation() {
  74:   return (curBuf.filename + ":" + Twine(prevTokLine)).str();
  75: }
  76: 
```

- **L68**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L69**: Defines function or method \`getColumnNumber\`. / 定义函数或方法 \`getColumnNumber\`。
- **L70**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L71**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L72**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L73**: Defines function or method \`getCurrentLocation\`. / 定义函数或方法 \`getCurrentLocation\`。
- **L74**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L75**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L76**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 77-88 / 第 77-88 行

```cpp
  77: // We don't want to record cascading errors. Keep only the first one.
  78: void ScriptLexer::setError(const Twine &msg) {
  79:   if (errCount(ctx))
  80:     return;
  81: 
  82:   std::string s = (getCurrentLocation() + ": " + msg).str();
  83:   if (prevTok.size())
  84:     s += "\n>>> " + getLine().str() + "\n>>> " +
  85:          std::string(getColumnNumber(), ' ') + "^";
  86:   ErrAlways(ctx) << s;
  87: }
  88: 
```

- **L77**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L78**: Defines function or method \`setError\`. / 定义函数或方法 \`setError\`。
- **L79**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L80**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L81**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L82**: Declares function or method \`getCurrentLocation\`. / 声明函数或方法 \`getCurrentLocation\`。
- **L83**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L84**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L85**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L86**: Calls a helper routine or completes a callable declaration. / 调用辅助例程或完成一个可调用对象的声明。
- **L87**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L88**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 89-100 / 第 89-100 行

```cpp
  89: void ScriptLexer::lex() {
  90:   for (;;) {
  91:     StringRef &s = curBuf.s;
  92:     s = skipSpace(s);
  93:     if (s.empty()) {
  94:       // If this buffer is from an INCLUDE, the caller is responsible for
  95:       // popping to the parent buffer.
  96:       eof = true;
  97:       return;
  98:     }
  99:     curTokState = lexState;
 100: 
```

- **L89**: Defines function or method \`lex\`. / 定义函数或方法 \`lex\`。
- **L90**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L91**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L92**: Declares function or method \`skipSpace\`. / 声明函数或方法 \`skipSpace\`。
- **L93**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L94**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L95**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L96**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L97**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L98**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L99**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L100**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 101-114 / 第 101-114 行

```cpp
 101:     // Quoted token. Note that double-quote characters are parts of a token
 102:     // because, in a glob match context, only unquoted tokens are interpreted
 103:     // as glob patterns. Double-quoted tokens are literal patterns in that
 104:     // context.
 105:     if (s.starts_with("\"")) {
 106:       size_t e = s.find("\"", 1);
 107:       if (e == StringRef::npos) {
 108:         size_t lineno =
 109:             StringRef(curBuf.begin, s.data() - curBuf.begin).count('\n');
 110:         ErrAlways(ctx) << curBuf.filename << ":" << (lineno + 1)
 111:                        << ": unclosed quote";
 112:         return;
 113:       }
 114: 
```

- **L101**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L102**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L103**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L104**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L105**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L106**: Declares function or method \`find\`. / 声明函数或方法 \`find\`。
- **L107**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L108**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L109**: Declares function or method \`StringRef\`. / 声明函数或方法 \`StringRef\`。
- **L110**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L111**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L112**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L113**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L114**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 115-132 / 第 115-132 行

```cpp
 115:       curTok = s.take_front(e + 1);
 116:       s = s.substr(e + 1);
 117:       return;
 118:     }
 119: 
 120:     // In Script and Expr states, recognize compound assignment operators.
 121:     auto recognizeAssign = [&]() -> bool {
 122:       if (s.starts_with("<<=") || s.starts_with(">>=")) {
 123:         curTok = s.substr(0, 3);
 124:         s = s.substr(3);
 125:         return true;
 126:       }
 127:       if (s.size() > 1 && (s[1] == '=' && strchr("+-*/!&^|", s[0]))) {
 128:         curTok = s.substr(0, 2);
 129:         s = s.substr(2);
 130:         return true;
 131:       }
 132:       return false;
```

- **L115**: Declares function or method \`take_front\`. / 声明函数或方法 \`take_front\`。
- **L116**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L117**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L118**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L119**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L120**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L121**: Opens the body of the current declaration, branch, or helper construct. / 打开当前声明、分支或辅助结构的主体。
- **L122**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L123**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L124**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L125**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L126**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L127**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L128**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L129**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L130**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L131**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L132**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。

### Lines 133-150 / 第 133-150 行

```cpp
 133:     };
 134: 
 135:     // Unquoted token. The non-expression token is more relaxed than tokens in
 136:     // C-like languages, so that you can write "file-name.cpp" as one bare
 137:     // token.
 138:     size_t pos;
 139:     constexpr StringRef scriptAndVersionChars =
 140:         "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
 141:         "0123456789_.$/\\~=+[]*?-!^:";
 142:     constexpr StringRef exprChars =
 143:         "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
 144:         "0123456789_.$";
 145:     switch (lexState) {
 146:     case State::Script:
 147:       if (recognizeAssign())
 148:         return;
 149:       pos = s.find_first_not_of(scriptAndVersionChars);
 150:       break;
```

- **L133**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L134**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L135**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L136**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L137**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L138**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L139**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L140**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L141**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L142**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L143**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L144**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L145**: Starts a multi-way branch based on a selector expression. / 开始一个基于选择表达式的多分支结构。
- **L146**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L147**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L148**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L149**: Declares function or method \`find_first_not_of\`. / 声明函数或方法 \`find_first_not_of\`。
- **L150**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。

### Lines 151-168 / 第 151-168 行

```cpp
 151:     case State::Expr:
 152:       if (recognizeAssign())
 153:         return;
 154:       pos = s.find_first_not_of(exprChars);
 155:       if (pos == 0 && s.size() >= 2 &&
 156:           ((s[0] == s[1] && strchr("<>&|", s[0])) ||
 157:            is_contained({"==", "!=", "<=", ">=", "<<", ">>"}, s.substr(0, 2))))
 158:         pos = 2;
 159:       break;
 160:     case State::VersionNode:
 161:       // Treat `:` as a token boundary unless it's part of a scope operator `::`
 162:       // (for extern "C++"). This behavior resembles GNU ld and allows proper
 163:       // tokenization of patterns like `local:*`.
 164:       pos = 0;
 165:       for (; pos != s.size(); ++pos) {
 166:         if (s[pos] == ':') {
 167:           if (pos + 1 != s.size() && s[pos + 1] == ':') {
 168:             ++pos;
```

- **L151**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L152**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L153**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L154**: Declares function or method \`find_first_not_of\`. / 声明函数或方法 \`find_first_not_of\`。
- **L155**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L156**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L157**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L158**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L159**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L160**: Marks one \`switch\` case alternative. / 标记 \`switch\` 语句中的一个分支选项。
- **L161**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L162**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L163**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L164**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L165**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L166**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L167**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L168**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。

### Lines 169-177 / 第 169-177 行

```cpp
 169:             continue;
 170:           }
 171:         } else if (scriptAndVersionChars.contains(s[pos]))
 172:           continue;
 173:         break;
 174:       }
 175:       break;
 176:     }
 177: 
```

- **L169**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L170**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L171**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L172**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L173**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L174**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L175**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L176**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L177**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 178-195 / 第 178-195 行

```cpp
 178:     if (pos == 0)
 179:       pos = 1;
 180:     curTok = s.substr(0, pos);
 181:     s = s.substr(pos);
 182:     break;
 183:   }
 184: }
 185: 
 186: // Skip leading whitespace characters or comments.
 187: StringRef ScriptLexer::skipSpace(StringRef s) {
 188:   for (;;) {
 189:     if (s.starts_with("/*")) {
 190:       size_t e = s.find("*/", 2);
 191:       if (e == StringRef::npos) {
 192:         setError("unclosed comment in a linker script");
 193:         return "";
 194:       }
 195:       curBuf.lineNumber += s.substr(0, e).count('\n');
```

- **L178**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L179**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L180**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L181**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L182**: Terminates the nearest loop or switch branch. / 终止最近一层的循环或 \`switch\` 分支。
- **L183**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L184**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L185**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L186**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L187**: Defines function or method \`skipSpace\`. / 定义函数或方法 \`skipSpace\`。
- **L188**: Starts a counted or iterator-based loop. / 开始一个计数式或迭代器式循环。
- **L189**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L190**: Declares function or method \`find\`. / 声明函数或方法 \`find\`。
- **L191**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L192**: Declares function or method \`setError\`. / 声明函数或方法 \`setError\`。
- **L193**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L194**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L195**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。

### Lines 196-213 / 第 196-213 行

```cpp
 196:       s = s.substr(e + 2);
 197:       continue;
 198:     }
 199:     if (s.starts_with("#")) {
 200:       size_t e = s.find('\n', 1);
 201:       if (e == StringRef::npos)
 202:         e = s.size() - 1;
 203:       else
 204:         ++curBuf.lineNumber;
 205:       s = s.substr(e + 1);
 206:       continue;
 207:     }
 208:     StringRef saved = s;
 209:     s = s.ltrim();
 210:     auto len = saved.size() - s.size();
 211:     if (len == 0)
 212:       return s;
 213:     curBuf.lineNumber += saved.substr(0, len).count('\n');
```

- **L196**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L197**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L198**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L199**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L200**: Declares function or method \`find\`. / 声明函数或方法 \`find\`。
- **L201**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L202**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L203**: Begins the fallback branch for the preceding conditional logic. / 开始前一个条件逻辑的后备分支。
- **L204**: Completes a declaration or executable statement in the current scope. / 在当前作用域中完成一条声明或可执行语句。
- **L205**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。
- **L206**: Skips directly to the next loop iteration. / 直接跳到下一次循环迭代。
- **L207**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L208**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L209**: Declares function or method \`ltrim\`. / 声明函数或方法 \`ltrim\`。
- **L210**: Declares function or method \`size\`. / 声明函数或方法 \`size\`。
- **L211**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L212**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L213**: Declares function or method \`substr\`. / 声明函数或方法 \`substr\`。

### Lines 214-228 / 第 214-228 行

```cpp
 214:   }
 215: }
 216: 
 217: // Used to determine whether to stop parsing. Treat errors like EOF.
 218: bool ScriptLexer::atEOF() { return eof || errCount(ctx); }
 219: 
 220: StringRef ScriptLexer::next() {
 221:   prevTok = peek();
 222:   // `prevTokLine` is not updated for EOF so that the line number in `setError`
 223:   // will be more useful.
 224:   if (prevTok.size())
 225:     prevTokLine = curBuf.lineNumber;
 226:   return std::exchange(curTok, StringRef(curBuf.s.data(), 0));
 227: }
 228: 
```

- **L214**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L215**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L216**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L217**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L218**: Defines function or method \`atEOF\`. / 定义函数或方法 \`atEOF\`。
- **L219**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L220**: Defines function or method \`next\`. / 定义函数或方法 \`next\`。
- **L221**: Declares function or method \`peek\`. / 声明函数或方法 \`peek\`。
- **L222**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L223**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L224**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L225**: Assigns or initializes state used by the surrounding logic. / 赋值或初始化周边逻辑要使用的状态。
- **L226**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L227**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L228**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 229-239 / 第 229-239 行

```cpp
 229: StringRef ScriptLexer::peek() {
 230:   // curTok is invalid if curTokState and lexState mismatch.
 231:   if (curTok.size() && curTokState != lexState) {
 232:     curBuf.s = StringRef(curTok.data(), curBuf.s.end() - curTok.data());
 233:     curTok = {};
 234:   }
 235:   if (curTok.empty())
 236:     lex();
 237:   return curTok;
 238: }
 239: 
```

- **L229**: Defines function or method \`peek\`. / 定义函数或方法 \`peek\`。
- **L230**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L231**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L232**: Declares function or method \`StringRef\`. / 声明函数或方法 \`StringRef\`。
- **L233**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L234**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L235**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L236**: Declares function or method \`lex\`. / 声明函数或方法 \`lex\`。
- **L237**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L238**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L239**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 240-248 / 第 240-248 行

```cpp
 240: bool ScriptLexer::consume(StringRef tok) {
 241:   if (peek() != tok)
 242:     return false;
 243:   next();
 244:   return true;
 245: }
 246: 
 247: void ScriptLexer::skip() { (void)next(); }
 248: 
```

- **L240**: Defines function or method \`consume\`. / 定义函数或方法 \`consume\`。
- **L241**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L242**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L243**: Declares function or method \`next\`. / 声明函数或方法 \`next\`。
- **L244**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L245**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L246**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L247**: Defines function or method \`skip\`. / 定义函数或方法 \`skip\`。
- **L248**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 249-260 / 第 249-260 行

```cpp
 249: void ScriptLexer::expect(StringRef expect) {
 250:   if (errCount(ctx))
 251:     return;
 252:   StringRef tok = next();
 253:   if (tok != expect) {
 254:     if (atEOF())
 255:       setError("unexpected EOF");
 256:     else
 257:       setError(expect + " expected, but got " + tok);
 258:   }
 259: }
 260: 
```

- **L249**: Defines function or method \`expect\`. / 定义函数或方法 \`expect\`。
- **L250**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L251**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L252**: Declares function or method \`next\`. / 声明函数或方法 \`next\`。
- **L253**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L254**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L255**: Declares function or method \`setError\`. / 声明函数或方法 \`setError\`。
- **L256**: Begins the fallback branch for the preceding conditional logic. / 开始前一个条件逻辑的后备分支。
- **L257**: Declares function or method \`setError\`. / 声明函数或方法 \`setError\`。
- **L258**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L259**: Closes the current scope or body. / 关闭当前作用域或代码体。
- **L260**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 261-270 / 第 261-270 行

```cpp
 261: ScriptLexer::Token ScriptLexer::till(StringRef tok) {
 262:   StringRef str = next();
 263:   if (str == tok)
 264:     return {};
 265:   if (!atEOF())
 266:     return {str};
 267:   prevTok = {};
 268:   setError("unexpected EOF");
 269:   return {};
 270: }
```

- **L261**: Defines function or method \`till\`. / 定义函数或方法 \`till\`。
- **L262**: Declares function or method \`next\`. / 声明函数或方法 \`next\`。
- **L263**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L264**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L265**: Starts a conditional branch that runs only when the predicate is true. / 开始一个条件分支，仅在判定条件为真时执行。
- **L266**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L267**: Closes the current declaration or scoped construct. / 结束当前声明或带作用域的结构。
- **L268**: Declares function or method \`setError\`. / 声明函数或方法 \`setError\`。
- **L269**: Returns a value or exits the current function at this point. / 在此返回一个值，或直接结束当前函数。
- **L270**: Closes the current scope or body. / 关闭当前作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file defines a lexer for the linker script. / 实现 ELF 链接器逻辑，例如符号解析、节布局、重定位以及驱动行为。
- **Scale / 规模**: 270 lines, 6 direct includes, 0 named types, and 28 detected routines. / 共 270 行，含 6 个直接包含、0 个具名类型、28 个检测到的例程。
- **ELF linking / ELF 链接**: The code works on ELF symbols, sections, relocations, or output layout. / 该代码处理 ELF 符号、节、重定位或输出布局。

## Dependencies / 依赖关系

- **LLVM / LLVM**: `llvm/ADT/Twine.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Path.h`.
- **System or local / 系统或本地**: `ScriptLexer.h`, `Config.h`.
- **Header roles / 头文件角色**: support-library helpers / Support 库辅助功能 (3), standard-library or local support header / 标准库或本地支持头文件 (2), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1).
- **Visible routines / 可见例程**: `begin`, `empty`, `ctx`, `insert`, `getLine`, `s`, `rfind`, `substr`, `getColumnNumber`, `data`, `getCurrentLocation`, `setError`.
