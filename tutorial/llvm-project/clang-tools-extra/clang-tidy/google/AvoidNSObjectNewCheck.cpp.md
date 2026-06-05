# AvoidNSObjectNewCheck.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `clang-tools-extra/clang-tidy/google/AvoidNSObjectNewCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: Implements the `AvoidNSObjectNewCheck` clang-tidy check in the `google` module around avoid n s object new diagnostics and fixes.
- **Purpose (CN)**: 实现 `google` 模块中的 `AvoidNSObjectNewCheck` clang-tidy 检查，围绕 Avoid N S Object New 相关诊断与修复展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14 / 第 1-14 行

```cpp
   1: //===----------------------------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "AvoidNSObjectNewCheck.h"
  10: #include "clang/AST/ASTContext.h"
  11: #include "clang/ASTMatchers/ASTMatchFinder.h"
  12: #include "clang/Basic/LangOptions.h"
  13: #include "clang/Basic/SourceLocation.h"
  14: #include "clang/Basic/SourceManager.h"
```
- **Line 1 / 第 1 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 2 / 第 2 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 3 / 第 3 行**: EN: Comment describing intent, behavior, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. CN: 用于说明意图、行为或元数据的注释：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **Line 4 / 第 4 行**: EN: Comment describing intent, behavior, or metadata: `See https://llvm.org/LICENSE.txt for license information.`. CN: 用于说明意图、行为或元数据的注释：`See https://llvm.org/LICENSE.txt for license information.`。
- **Line 5 / 第 5 行**: EN: Comment describing intent, behavior, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. CN: 用于说明意图、行为或元数据的注释：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **Line 6 / 第 6 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 7 / 第 7 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 8 / 第 8 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 9 / 第 9 行**: EN: Includes "AvoidNSObjectNewCheck.h" so this file can use local declarations that pair with this file. CN: 包含 "AvoidNSObjectNewCheck.h"，以便当前文件使用与该文件配套的本地声明。
- **Line 10 / 第 10 行**: EN: Includes "clang/AST/ASTContext.h" so this file can use Clang AST data structures and traversal APIs. CN: 包含 "clang/AST/ASTContext.h"，以便当前文件使用Clang AST 数据结构与遍历 API。
- **Line 11 / 第 11 行**: EN: Includes "clang/ASTMatchers/ASTMatchFinder.h" so this file can use Clang AST matcher infrastructure. CN: 包含 "clang/ASTMatchers/ASTMatchFinder.h"，以便当前文件使用Clang AST Matcher 基础设施。
- **Line 12 / 第 12 行**: EN: Includes "clang/Basic/LangOptions.h" so this file can use Clang basic support types and diagnostics. CN: 包含 "clang/Basic/LangOptions.h"，以便当前文件使用Clang 基础支持类型与诊断设施。
- **Line 13 / 第 13 行**: EN: Includes "clang/Basic/SourceLocation.h" so this file can use Clang basic support types and diagnostics. CN: 包含 "clang/Basic/SourceLocation.h"，以便当前文件使用Clang 基础支持类型与诊断设施。
- **Line 14 / 第 14 行**: EN: Includes "clang/Basic/SourceManager.h" so this file can use Clang basic support types and diagnostics. CN: 包含 "clang/Basic/SourceManager.h"，以便当前文件使用Clang 基础支持类型与诊断设施。

### Lines 15-28 / 第 15-28 行

```cpp
  15: #include "clang/Lex/Lexer.h"
  16: #include "llvm/Support/FormatVariadic.h"
  17: #include <string>
  18: #include <utility>
  19: 
  20: using namespace clang::ast_matchers;
  21: 
  22: namespace clang::tidy::google::objc {
  23: 
  24: static bool isMessageExpressionInsideMacro(const ObjCMessageExpr *Expr) {
  25:   const SourceLocation ReceiverLocation = Expr->getReceiverRange().getBegin();
  26:   if (ReceiverLocation.isMacroID())
  27:     return true;
  28: 
```
- **Line 15 / 第 15 行**: EN: Includes "clang/Lex/Lexer.h" so this file can use Clang lexer and preprocessor facilities. CN: 包含 "clang/Lex/Lexer.h"，以便当前文件使用Clang 词法分析与预处理设施。
- **Line 16 / 第 16 行**: EN: Includes "llvm/Support/FormatVariadic.h" so this file can use LLVM support utilities such as diagnostics, filesystem, and strings. CN: 包含 "llvm/Support/FormatVariadic.h"，以便当前文件使用LLVM 支持工具，例如诊断、文件系统和字符串设施。
- **Line 17 / 第 17 行**: EN: Includes <string> so this file can use supporting declarations or standard-library facilities. CN: 包含 <string>，以便当前文件使用辅助声明或标准库设施。
- **Line 18 / 第 18 行**: EN: Includes <utility> so this file can use supporting declarations or standard-library facilities. CN: 包含 <utility>，以便当前文件使用辅助声明或标准库设施。
- **Line 19 / 第 19 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 20 / 第 20 行**: EN: Brings namespace `clang::ast_matchers` into the local scope. CN: 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **Line 21 / 第 21 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 22 / 第 22 行**: EN: Opens namespace `clang::tidy::google::objc` to scope related declarations. CN: 打开命名空间 `clang::tidy::google::objc`，为相关声明建立作用域。
- **Line 23 / 第 23 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 24 / 第 24 行**: EN: Defines function or method `isMessageExpressionInsideMacro`. CN: 定义函数或方法 `isMessageExpressionInsideMacro`。
- **Line 25 / 第 25 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 26 / 第 26 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 27 / 第 27 行**: EN: Returns a value or transfers control to the caller with `true`. CN: 返回一个值，或以 `true` 将控制权交还给调用者。
- **Line 28 / 第 28 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 29-42 / 第 29-42 行

```cpp
  29:   const SourceLocation SelectorLocation = Expr->getSelectorStartLoc();
  30:   if (SelectorLocation.isMacroID())
  31:     return true;
  32: 
  33:   return false;
  34: }
  35: 
  36: // Walk up the class hierarchy looking for an -init method, returning true
  37: // if one is found and has not been marked unavailable.
  38: static bool isInitMethodAvailable(const ObjCInterfaceDecl *ClassDecl) {
  39:   while (ClassDecl != nullptr) {
  40:     for (const auto *MethodDecl : ClassDecl->instance_methods())
  41:       if (MethodDecl->getSelector().getAsString() == "init")
  42:         return !MethodDecl->isUnavailable();
```
- **Line 29 / 第 29 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 30 / 第 30 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 31 / 第 31 行**: EN: Returns a value or transfers control to the caller with `true`. CN: 返回一个值，或以 `true` 将控制权交还给调用者。
- **Line 32 / 第 32 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 33 / 第 33 行**: EN: Returns a value or transfers control to the caller with `false`. CN: 返回一个值，或以 `false` 将控制权交还给调用者。
- **Line 34 / 第 34 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 35 / 第 35 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 36 / 第 36 行**: EN: Comment describing intent, behavior, or metadata: `Walk up the class hierarchy looking for an -init method, returning true`. CN: 用于说明意图、行为或元数据的注释：`Walk up the class hierarchy looking for an -init method, returning true`。
- **Line 37 / 第 37 行**: EN: Comment describing intent, behavior, or metadata: `if one is found and has not been marked unavailable.`. CN: 用于说明意图、行为或元数据的注释：`if one is found and has not been marked unavailable.`。
- **Line 38 / 第 38 行**: EN: Defines function or method `isInitMethodAvailable`. CN: 定义函数或方法 `isInitMethodAvailable`。
- **Line 39 / 第 39 行**: EN: Starts a loop that continues while the condition holds. CN: 开始一个在条件满足时持续执行的循环。
- **Line 40 / 第 40 行**: EN: Starts a loop that iterates over a range, container, or index sequence. CN: 开始一个循环，用于遍历范围、容器或索引序列。
- **Line 41 / 第 41 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 42 / 第 42 行**: EN: Returns a value or transfers control to the caller with `!MethodDecl->isUnavailable()`. CN: 返回一个值，或以 `!MethodDecl->isUnavailable()` 将控制权交还给调用者。

### Lines 43-56 / 第 43-56 行

```cpp
  43:     ClassDecl = ClassDecl->getSuperClass();
  44:   }
  45: 
  46:   // No -init method found in the class hierarchy. This should occur only rarely
  47:   // in Objective-C code, and only really applies to classes not derived from
  48:   // NSObject.
  49:   return false;
  50: }
  51: 
  52: // Returns the string for the Objective-C message receiver. Keeps any generics
  53: // included in the receiver class type, which are stripped if the class type is
  54: // used. While the generics arguments will not make any difference to the
  55: // returned code at this time, the style guide allows them and they should be
  56: // left in any fix-it hint.
```
- **Line 43 / 第 43 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 44 / 第 44 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 45 / 第 45 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 46 / 第 46 行**: EN: Comment describing intent, behavior, or metadata: `No -init method found in the class hierarchy. This should occur only rarely`. CN: 用于说明意图、行为或元数据的注释：`No -init method found in the class hierarchy. This should occur only rarely`。
- **Line 47 / 第 47 行**: EN: Comment describing intent, behavior, or metadata: `in Objective-C code, and only really applies to classes not derived from`. CN: 用于说明意图、行为或元数据的注释：`in Objective-C code, and only really applies to classes not derived from`。
- **Line 48 / 第 48 行**: EN: Comment describing intent, behavior, or metadata: `NSObject.`. CN: 用于说明意图、行为或元数据的注释：`NSObject.`。
- **Line 49 / 第 49 行**: EN: Returns a value or transfers control to the caller with `false`. CN: 返回一个值，或以 `false` 将控制权交还给调用者。
- **Line 50 / 第 50 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 51 / 第 51 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 52 / 第 52 行**: EN: Comment describing intent, behavior, or metadata: `Returns the string for the Objective-C message receiver. Keeps any generics`. CN: 用于说明意图、行为或元数据的注释：`Returns the string for the Objective-C message receiver. Keeps any generics`。
- **Line 53 / 第 53 行**: EN: Comment describing intent, behavior, or metadata: `included in the receiver class type, which are stripped if the class type is`. CN: 用于说明意图、行为或元数据的注释：`included in the receiver class type, which are stripped if the class type is`。
- **Line 54 / 第 54 行**: EN: Comment describing intent, behavior, or metadata: `used. While the generics arguments will not make any difference to the`. CN: 用于说明意图、行为或元数据的注释：`used. While the generics arguments will not make any difference to the`。
- **Line 55 / 第 55 行**: EN: Comment describing intent, behavior, or metadata: `returned code at this time, the style guide allows them and they should be`. CN: 用于说明意图、行为或元数据的注释：`returned code at this time, the style guide allows them and they should be`。
- **Line 56 / 第 56 行**: EN: Comment describing intent, behavior, or metadata: `left in any fix-it hint.`. CN: 用于说明意图、行为或元数据的注释：`left in any fix-it hint.`。

### Lines 57-70 / 第 57-70 行

```cpp
  57: static StringRef getReceiverString(SourceRange ReceiverRange,
  58:                                    const SourceManager &SM,
  59:                                    const LangOptions &LangOpts) {
  60:   const CharSourceRange CharRange = Lexer::makeFileCharRange(
  61:       CharSourceRange::getTokenRange(ReceiverRange), SM, LangOpts);
  62:   return Lexer::getSourceText(CharRange, SM, LangOpts);
  63: }
  64: 
  65: static FixItHint getCallFixItHint(const ObjCMessageExpr *Expr,
  66:                                   const SourceManager &SM,
  67:                                   const LangOptions &LangOpts) {
  68:   // Check whether the messaged class has a known factory method to use instead
  69:   // of -init.
  70:   StringRef Receiver =
```
- **Line 57 / 第 57 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 58 / 第 58 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 59 / 第 59 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 60 / 第 60 行**: EN: Continues logic associated with callable symbol `makeFileCharRange`. CN: 继续与可调用符号 `makeFileCharRange` 相关的逻辑。
- **Line 61 / 第 61 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 62 / 第 62 行**: EN: Returns a value or transfers control to the caller with `Lexer::getSourceText(CharRange, SM, LangOpts)`. CN: 返回一个值，或以 `Lexer::getSourceText(CharRange, SM, LangOpts)` 将控制权交还给调用者。
- **Line 63 / 第 63 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 64 / 第 64 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 65 / 第 65 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 66 / 第 66 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 67 / 第 67 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 68 / 第 68 行**: EN: Comment describing intent, behavior, or metadata: `Check whether the messaged class has a known factory method to use instead`. CN: 用于说明意图、行为或元数据的注释：`Check whether the messaged class has a known factory method to use instead`。
- **Line 69 / 第 69 行**: EN: Comment describing intent, behavior, or metadata: `of -init.`. CN: 用于说明意图、行为或元数据的注释：`of -init.`。
- **Line 70 / 第 70 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。

### Lines 71-84 / 第 71-84 行

```cpp
  71:       getReceiverString(Expr->getReceiverRange(), SM, LangOpts);
  72:   // Some classes should use standard factory methods instead of alloc/init.
  73:   static constexpr std::pair<StringRef, StringRef> ClassToFactoryMethodMap[] = {
  74:       {"NSDate", "date"}, {"NSNull", "null"}};
  75:   const auto *FoundClassFactory =
  76:       llvm::find_if(ClassToFactoryMethodMap,
  77:                     [&](const auto &Entry) { return Entry.first == Receiver; });
  78:   if (FoundClassFactory != std::end(ClassToFactoryMethodMap)) {
  79:     const auto &[ClassName, FactorySelector] = *FoundClassFactory;
  80:     const std::string NewCall =
  81:         std::string(llvm::formatv("[{0} {1}]", ClassName, FactorySelector));
  82:     return FixItHint::CreateReplacement(Expr->getSourceRange(), NewCall);
  83:   }
  84: 
```
- **Line 71 / 第 71 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 72 / 第 72 行**: EN: Comment describing intent, behavior, or metadata: `Some classes should use standard factory methods instead of alloc/init.`. CN: 用于说明意图、行为或元数据的注释：`Some classes should use standard factory methods instead of alloc/init.`。
- **Line 73 / 第 73 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 74 / 第 74 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 75 / 第 75 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 76 / 第 76 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 77 / 第 77 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 78 / 第 78 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 79 / 第 79 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 80 / 第 80 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 81 / 第 81 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 82 / 第 82 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 83 / 第 83 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 84 / 第 84 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 85-98 / 第 85-98 行

```cpp
  85:   if (isInitMethodAvailable(Expr->getReceiverInterface())) {
  86:     const std::string NewCall =
  87:         std::string(llvm::formatv("[[{0} alloc] init]", Receiver));
  88:     return FixItHint::CreateReplacement(Expr->getSourceRange(), NewCall);
  89:   }
  90: 
  91:   return {}; // No known replacement available.
  92: }
  93: 
  94: void AvoidNSObjectNewCheck::registerMatchers(MatchFinder *Finder) {
  95:   // Add two matchers, to catch calls to +new and implementations of +new.
  96:   Finder->addMatcher(
  97:       objcMessageExpr(isClassMessage(), hasSelector("new")).bind("new_call"),
  98:       this);
```
- **Line 85 / 第 85 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 86 / 第 86 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 87 / 第 87 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 88 / 第 88 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 89 / 第 89 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 90 / 第 90 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 91 / 第 91 行**: EN: Returns a value or transfers control to the caller with `{}; // No known replacement available.`. CN: 返回一个值，或以 `{}; // No known replacement available.` 将控制权交还给调用者。
- **Line 92 / 第 92 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 93 / 第 93 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 94 / 第 94 行**: EN: Starts or references matcher-registration logic for this check. CN: 开始或引用该检查的 Matcher 注册逻辑。
- **Line 95 / 第 95 行**: EN: Comment describing intent, behavior, or metadata: `Add two matchers, to catch calls to +new and implementations of +new.`. CN: 用于说明意图、行为或元数据的注释：`Add two matchers, to catch calls to +new and implementations of +new.`。
- **Line 96 / 第 96 行**: EN: Registers an AST matcher that will trigger this check on matching nodes. CN: 注册一个 AST Matcher，使该检查在节点匹配时被触发。
- **Line 97 / 第 97 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 98 / 第 98 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。

### Lines 99-112 / 第 99-112 行

```cpp
  99:   Finder->addMatcher(
 100:       objcMethodDecl(isClassMethod(), isDefinition(), hasName("new"))
 101:           .bind("new_override"),
 102:       this);
 103: }
 104: 
 105: void AvoidNSObjectNewCheck::check(const MatchFinder::MatchResult &Result) {
 106:   if (const auto *CallExpr =
 107:           Result.Nodes.getNodeAs<ObjCMessageExpr>("new_call")) {
 108:     // Don't warn if the call expression originates from a macro expansion.
 109:     if (isMessageExpressionInsideMacro(CallExpr))
 110:       return;
 111: 
 112:     diag(CallExpr->getExprLoc(), "do not create objects with +new")
```
- **Line 99 / 第 99 行**: EN: Registers an AST matcher that will trigger this check on matching nodes. CN: 注册一个 AST Matcher，使该检查在节点匹配时被触发。
- **Line 100 / 第 100 行**: EN: Continues logic associated with callable symbol `objcMethodDecl`. CN: 继续与可调用符号 `objcMethodDecl` 相关的逻辑。
- **Line 101 / 第 101 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 102 / 第 102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 103 / 第 103 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 104 / 第 104 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 105 / 第 105 行**: EN: Defines function or method `check`. CN: 定义函数或方法 `check`。
- **Line 106 / 第 106 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 107 / 第 107 行**: EN: Defines function or method `getNodeAs<ObjCMessageExpr>`. CN: 定义函数或方法 `getNodeAs<ObjCMessageExpr>`。
- **Line 108 / 第 108 行**: EN: Comment describing intent, behavior, or metadata: `Don't warn if the call expression originates from a macro expansion.`. CN: 用于说明意图、行为或元数据的注释：`Don't warn if the call expression originates from a macro expansion.`。
- **Line 109 / 第 109 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 110 / 第 110 行**: EN: Returns a value or transfers control to the caller with `void`. CN: 返回一个值，或以 `void` 将控制权交还给调用者。
- **Line 111 / 第 111 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 112 / 第 112 行**: EN: Emits a clang-tidy diagnostic, often paired with notes or fix-its. CN: 发出一条 clang-tidy 诊断，通常会配合注释或修复建议。

### Lines 113-123 / 第 113-123 行

```cpp
 113:         << getCallFixItHint(CallExpr, *Result.SourceManager,
 114:                             Result.Context->getLangOpts());
 115:   }
 116: 
 117:   if (const auto *DeclExpr =
 118:           Result.Nodes.getNodeAs<ObjCMethodDecl>("new_override")) {
 119:     diag(DeclExpr->getBeginLoc(), "classes should not override +new");
 120:   }
 121: }
 122: 
 123: } // namespace clang::tidy::google::objc
```
- **Line 113 / 第 113 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 114 / 第 114 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 115 / 第 115 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 116 / 第 116 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 117 / 第 117 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 118 / 第 118 行**: EN: Defines function or method `getNodeAs<ObjCMethodDecl>`. CN: 定义函数或方法 `getNodeAs<ObjCMethodDecl>`。
- **Line 119 / 第 119 行**: EN: Emits a clang-tidy diagnostic, often paired with notes or fix-its. CN: 发出一条 clang-tidy 诊断，通常会配合注释或修复建议。
- **Line 120 / 第 120 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 121 / 第 121 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 122 / 第 122 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 123 / 第 123 行**: EN: Closes a namespace scope and documents which namespace ended. CN: 结束一个命名空间作用域，并说明被关闭的命名空间。

## Key Concepts / 关键概念
- **Clang-Tidy framework / Clang-Tidy 框架**: Participates in the clang-tidy architecture that wires checks, options, and diagnostics together. / 参与 clang-tidy 架构，把检查、选项和诊断连接在一起。
- **google module focus / google 模块关注点**: This file belongs to the `google` module, which concentrates on Google style and API checks. / 该文件属于 `google` 模块，重点关注Google 风格与 API 检查。
- **AST matcher registration / AST Matcher 注册**: Connects declarative AST matchers to callback-based diagnostics. / 把声明式 AST Matcher 连接到基于回调的诊断逻辑。
- **Diagnostic emission / 诊断发射**: Produces clang-tidy warnings, notes, and fix-it hints. / 产生 clang-tidy 警告、注释和修复提示。
- **Automated fix-its / 自动修复建议**: Builds source edits that can be applied automatically. / 构建可自动应用的源码编辑。
- **Token-level source handling / 词法级源码处理**: Inspects tokens and spelling while preparing diagnostics or fixes. / 在准备诊断或修复时检查 token 及其拼写。

## Dependencies / 依赖关系
- **Clang/LLVM and local headers / Clang/LLVM 与本地头文件**: `AvoidNSObjectNewCheck.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Basic/LangOptions.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/SourceManager.h`, `clang/Lex/Lexer.h`, `llvm/Support/FormatVariadic.h`
- **Standard library headers / 标准库头文件**: `<string>`, `<utility>`
