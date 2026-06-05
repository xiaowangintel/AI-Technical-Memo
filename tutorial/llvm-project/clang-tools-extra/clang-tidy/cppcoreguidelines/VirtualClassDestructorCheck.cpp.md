# VirtualClassDestructorCheck.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `clang-tools-extra/clang-tidy/cppcoreguidelines/VirtualClassDestructorCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: Implements the `VirtualClassDestructorCheck` clang-tidy check in the `cppcoreguidelines` module around virtual class destructor diagnostics and fixes.
- **Purpose (CN)**: 实现 `cppcoreguidelines` 模块中的 `VirtualClassDestructorCheck` clang-tidy 检查，围绕 Virtual Class Destructor 相关诊断与修复展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
   1: //===----------------------------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "VirtualClassDestructorCheck.h"
  10: #include "../utils/LexerUtils.h"
  11: #include "clang/AST/ASTContext.h"
  12: #include "clang/ASTMatchers/ASTMatchFinder.h"
  13: #include "clang/Lex/Lexer.h"
  14: #include <optional>
  15: #include <string>
  16: 
```
- **Line 1 / 第 1 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 2 / 第 2 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 3 / 第 3 行**: EN: Comment describing intent, behavior, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. CN: 用于说明意图、行为或元数据的注释：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **Line 4 / 第 4 行**: EN: Comment describing intent, behavior, or metadata: `See https://llvm.org/LICENSE.txt for license information.`. CN: 用于说明意图、行为或元数据的注释：`See https://llvm.org/LICENSE.txt for license information.`。
- **Line 5 / 第 5 行**: EN: Comment describing intent, behavior, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. CN: 用于说明意图、行为或元数据的注释：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **Line 6 / 第 6 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 7 / 第 7 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 8 / 第 8 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 9 / 第 9 行**: EN: Includes "VirtualClassDestructorCheck.h" so this file can use local declarations that pair with this file. CN: 包含 "VirtualClassDestructorCheck.h"，以便当前文件使用与该文件配套的本地声明。
- **Line 10 / 第 10 行**: EN: Includes "../utils/LexerUtils.h" so this file can use local declarations that pair with this file. CN: 包含 "../utils/LexerUtils.h"，以便当前文件使用与该文件配套的本地声明。
- **Line 11 / 第 11 行**: EN: Includes "clang/AST/ASTContext.h" so this file can use Clang AST data structures and traversal APIs. CN: 包含 "clang/AST/ASTContext.h"，以便当前文件使用Clang AST 数据结构与遍历 API。
- **Line 12 / 第 12 行**: EN: Includes "clang/ASTMatchers/ASTMatchFinder.h" so this file can use Clang AST matcher infrastructure. CN: 包含 "clang/ASTMatchers/ASTMatchFinder.h"，以便当前文件使用Clang AST Matcher 基础设施。
- **Line 13 / 第 13 行**: EN: Includes "clang/Lex/Lexer.h" so this file can use Clang lexer and preprocessor facilities. CN: 包含 "clang/Lex/Lexer.h"，以便当前文件使用Clang 词法分析与预处理设施。
- **Line 14 / 第 14 行**: EN: Includes <optional> so this file can use supporting declarations or standard-library facilities. CN: 包含 <optional>，以便当前文件使用辅助声明或标准库设施。
- **Line 15 / 第 15 行**: EN: Includes <string> so this file can use supporting declarations or standard-library facilities. CN: 包含 <string>，以便当前文件使用辅助声明或标准库设施。
- **Line 16 / 第 16 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 17-32 / 第 17-32 行

```cpp
  17: using namespace clang::ast_matchers;
  18: 
  19: namespace clang::tidy::cppcoreguidelines {
  20: 
  21: namespace {
  22: 
  23: AST_MATCHER(CXXRecordDecl, hasPublicVirtualOrProtectedNonVirtualDestructor) {
  24:   // We need to call Node.getDestructor() instead of matching a
  25:   // CXXDestructorDecl. Otherwise, tests will fail for class templates, since
  26:   // the primary template (not the specialization) always gets a non-virtual
  27:   // CXXDestructorDecl in the AST. https://bugs.llvm.org/show_bug.cgi?id=51912
  28:   const CXXDestructorDecl *Destructor = Node.getDestructor();
  29:   if (!Destructor)
  30:     return false;
  31: 
  32:   return (((Destructor->getAccess() == AccessSpecifier::AS_public) &&
```
- **Line 17 / 第 17 行**: EN: Brings namespace `clang::ast_matchers` into the local scope. CN: 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **Line 18 / 第 18 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 19 / 第 19 行**: EN: Opens namespace `clang::tidy::cppcoreguidelines` to scope related declarations. CN: 打开命名空间 `clang::tidy::cppcoreguidelines`，为相关声明建立作用域。
- **Line 20 / 第 20 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 21 / 第 21 行**: EN: Introduces an anonymous namespace for file-local helpers. CN: 引入匿名命名空间以承载文件局部辅助逻辑。
- **Line 22 / 第 22 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 23 / 第 23 行**: EN: Defines function or method `AST_MATCHER`. CN: 定义函数或方法 `AST_MATCHER`。
- **Line 24 / 第 24 行**: EN: Comment describing intent, behavior, or metadata: `We need to call Node.getDestructor() instead of matching a`. CN: 用于说明意图、行为或元数据的注释：`We need to call Node.getDestructor() instead of matching a`。
- **Line 25 / 第 25 行**: EN: Comment describing intent, behavior, or metadata: `CXXDestructorDecl. Otherwise, tests will fail for class templates, since`. CN: 用于说明意图、行为或元数据的注释：`CXXDestructorDecl. Otherwise, tests will fail for class templates, since`。
- **Line 26 / 第 26 行**: EN: Comment describing intent, behavior, or metadata: `the primary template (not the specialization) always gets a non-virtual`. CN: 用于说明意图、行为或元数据的注释：`the primary template (not the specialization) always gets a non-virtual`。
- **Line 27 / 第 27 行**: EN: Comment describing intent, behavior, or metadata: `CXXDestructorDecl in the AST. https://bugs.llvm.org/show_bug.cgi?id=51912`. CN: 用于说明意图、行为或元数据的注释：`CXXDestructorDecl in the AST. https://bugs.llvm.org/show_bug.cgi?id=51912`。
- **Line 28 / 第 28 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 29 / 第 29 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 30 / 第 30 行**: EN: Returns a value or transfers control to the caller with `false`. CN: 返回一个值，或以 `false` 将控制权交还给调用者。
- **Line 31 / 第 31 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 32 / 第 32 行**: EN: Returns a value or transfers control to the caller with `(((Destructor->getAccess() == AccessSpecifier::AS_public) &&`. CN: 返回一个值，或以 `(((Destructor->getAccess() == AccessSpecifier::AS_public) &&` 将控制权交还给调用者。

### Lines 33-48 / 第 33-48 行

```cpp
  33:            Destructor->isVirtual()) ||
  34:           ((Destructor->getAccess() == AccessSpecifier::AS_protected) &&
  35:            !Destructor->isVirtual()));
  36: }
  37: 
  38: } // namespace
  39: 
  40: void VirtualClassDestructorCheck::registerMatchers(MatchFinder *Finder) {
  41:   ast_matchers::internal::Matcher<CXXRecordDecl> InheritsVirtualMethod =
  42:       hasAnyBase(hasType(cxxRecordDecl(has(cxxMethodDecl(isVirtual())))));
  43: 
  44:   Finder->addMatcher(
  45:       cxxRecordDecl(
  46:           anyOf(has(cxxMethodDecl(isVirtual())), InheritsVirtualMethod),
  47:           unless(isFinal()),
  48:           unless(hasPublicVirtualOrProtectedNonVirtualDestructor()))
```
- **Line 33 / 第 33 行**: EN: Continues logic associated with callable symbol `isVirtual`. CN: 继续与可调用符号 `isVirtual` 相关的逻辑。
- **Line 34 / 第 34 行**: EN: Continues logic associated with callable symbol `getAccess`. CN: 继续与可调用符号 `getAccess` 相关的逻辑。
- **Line 35 / 第 35 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 36 / 第 36 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 37 / 第 37 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 38 / 第 38 行**: EN: Closes a namespace scope and documents which namespace ended. CN: 结束一个命名空间作用域，并说明被关闭的命名空间。
- **Line 39 / 第 39 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 40 / 第 40 行**: EN: Starts or references matcher-registration logic for this check. CN: 开始或引用该检查的 Matcher 注册逻辑。
- **Line 41 / 第 41 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 42 / 第 42 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 43 / 第 43 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 44 / 第 44 行**: EN: Registers an AST matcher that will trigger this check on matching nodes. CN: 注册一个 AST Matcher，使该检查在节点匹配时被触发。
- **Line 45 / 第 45 行**: EN: Continues logic associated with callable symbol `cxxRecordDecl`. CN: 继续与可调用符号 `cxxRecordDecl` 相关的逻辑。
- **Line 46 / 第 46 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 47 / 第 47 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 48 / 第 48 行**: EN: Continues logic associated with callable symbol `unless`. CN: 继续与可调用符号 `unless` 相关的逻辑。

### Lines 49-64 / 第 49-64 行

```cpp
  49:           .bind("ProblematicClassOrStruct"),
  50:       this);
  51: }
  52: 
  53: static std::optional<CharSourceRange>
  54: getVirtualKeywordRange(const CXXDestructorDecl &Destructor,
  55:                        const SourceManager &SM, const LangOptions &LangOpts) {
  56:   if (Destructor.getLocation().isMacroID())
  57:     return std::nullopt;
  58: 
  59:   const SourceLocation VirtualBeginLoc = Destructor.getBeginLoc();
  60:   const SourceLocation VirtualBeginSpellingLoc =
  61:       SM.getSpellingLoc(Destructor.getBeginLoc());
  62:   const SourceLocation VirtualEndLoc = VirtualBeginSpellingLoc.getLocWithOffset(
  63:       Lexer::MeasureTokenLength(VirtualBeginSpellingLoc, SM, LangOpts));
  64: 
```
- **Line 49 / 第 49 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 50 / 第 50 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 51 / 第 51 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 52 / 第 52 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 53 / 第 53 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 54 / 第 54 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 55 / 第 55 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 56 / 第 56 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 57 / 第 57 行**: EN: Returns a value or transfers control to the caller with `std::nullopt`. CN: 返回一个值，或以 `std::nullopt` 将控制权交还给调用者。
- **Line 58 / 第 58 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 59 / 第 59 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 60 / 第 60 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 61 / 第 61 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 62 / 第 62 行**: EN: Continues logic associated with callable symbol `getLocWithOffset`. CN: 继续与可调用符号 `getLocWithOffset` 相关的逻辑。
- **Line 63 / 第 63 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 64 / 第 64 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 65-80 / 第 65-80 行

```cpp
  65:   /// Range ends with \c StartOfNextToken so that any whitespace after \c
  66:   /// virtual is included.
  67:   std::optional<Token> NextToken =
  68:       utils::lexer::findNextTokenSkippingComments(VirtualEndLoc, SM, LangOpts);
  69:   if (!NextToken)
  70:     return std::nullopt;
  71:   const SourceLocation StartOfNextToken = NextToken->getLocation();
  72: 
  73:   return CharSourceRange::getCharRange(VirtualBeginLoc, StartOfNextToken);
  74: }
  75: 
  76: static const AccessSpecDecl *
  77: getPublicASDecl(const CXXRecordDecl &StructOrClass) {
  78:   for (DeclContext::specific_decl_iterator<AccessSpecDecl>
  79:            AS{StructOrClass.decls_begin()},
  80:        ASEnd{StructOrClass.decls_end()};
```
- **Line 65 / 第 65 行**: EN: Comment describing intent, behavior, or metadata: `Range ends with \c StartOfNextToken so that any whitespace after \c`. CN: 用于说明意图、行为或元数据的注释：`Range ends with \c StartOfNextToken so that any whitespace after \c`。
- **Line 66 / 第 66 行**: EN: Comment describing intent, behavior, or metadata: `virtual is included.`. CN: 用于说明意图、行为或元数据的注释：`virtual is included.`。
- **Line 67 / 第 67 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 68 / 第 68 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 69 / 第 69 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 70 / 第 70 行**: EN: Returns a value or transfers control to the caller with `std::nullopt`. CN: 返回一个值，或以 `std::nullopt` 将控制权交还给调用者。
- **Line 71 / 第 71 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 72 / 第 72 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 73 / 第 73 行**: EN: Returns a value or transfers control to the caller with `CharSourceRange::getCharRange(VirtualBeginLoc, StartOfNextToken)`. CN: 返回一个值，或以 `CharSourceRange::getCharRange(VirtualBeginLoc, StartOfNextToken)` 将控制权交还给调用者。
- **Line 74 / 第 74 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 75 / 第 75 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 76 / 第 76 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 77 / 第 77 行**: EN: Defines function or method `getPublicASDecl`. CN: 定义函数或方法 `getPublicASDecl`。
- **Line 78 / 第 78 行**: EN: Starts a loop that iterates over a range, container, or index sequence. CN: 开始一个循环，用于遍历范围、容器或索引序列。
- **Line 79 / 第 79 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 80 / 第 80 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。

### Lines 81-96 / 第 81-96 行

```cpp
  81:        AS != ASEnd; ++AS) {
  82:     const AccessSpecDecl *ASDecl = *AS;
  83:     if (ASDecl->getAccess() == AccessSpecifier::AS_public)
  84:       return ASDecl;
  85:   }
  86: 
  87:   return nullptr;
  88: }
  89: 
  90: static FixItHint
  91: generateUserDeclaredDestructor(const CXXRecordDecl &StructOrClass,
  92:                                const SourceManager &SourceManager) {
  93:   std::string DestructorString;
  94:   SourceLocation Loc;
  95:   bool AppendLineBreak = false;
  96: 
```
- **Line 81 / 第 81 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 82 / 第 82 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 83 / 第 83 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 84 / 第 84 行**: EN: Returns a value or transfers control to the caller with `ASDecl`. CN: 返回一个值，或以 `ASDecl` 将控制权交还给调用者。
- **Line 85 / 第 85 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 86 / 第 86 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 87 / 第 87 行**: EN: Returns a value or transfers control to the caller with `nullptr`. CN: 返回一个值，或以 `nullptr` 将控制权交还给调用者。
- **Line 88 / 第 88 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 89 / 第 89 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 90 / 第 90 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 91 / 第 91 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 92 / 第 92 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 93 / 第 93 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 94 / 第 94 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 95 / 第 95 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 96 / 第 96 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 97-112 / 第 97-112 行

```cpp
  97:   const AccessSpecDecl *AccessSpecDecl = getPublicASDecl(StructOrClass);
  98: 
  99:   if (!AccessSpecDecl) {
 100:     if (StructOrClass.isClass()) {
 101:       Loc = StructOrClass.getEndLoc();
 102:       DestructorString = "public:";
 103:       AppendLineBreak = true;
 104:     } else {
 105:       Loc = StructOrClass.getBraceRange().getBegin().getLocWithOffset(1);
 106:     }
 107:   } else {
 108:     Loc = AccessSpecDecl->getEndLoc().getLocWithOffset(1);
 109:   }
 110: 
 111:   DestructorString = (llvm::Twine(DestructorString) + "\nvirtual ~" +
 112:                       StructOrClass.getName().str() + "() = default;" +
```
- **Line 97 / 第 97 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 98 / 第 98 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 99 / 第 99 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 100 / 第 100 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 101 / 第 101 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 102 / 第 102 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 103 / 第 103 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 104 / 第 104 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 105 / 第 105 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 106 / 第 106 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 107 / 第 107 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 108 / 第 108 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 109 / 第 109 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 110 / 第 110 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 111 / 第 111 行**: EN: Continues logic associated with callable symbol `Twine`. CN: 继续与可调用符号 `Twine` 相关的逻辑。
- **Line 112 / 第 112 行**: EN: Continues logic associated with callable symbol `getName`. CN: 继续与可调用符号 `getName` 相关的逻辑。

### Lines 113-128 / 第 113-128 行

```cpp
 113:                       (AppendLineBreak ? "\n" : ""))
 114:                          .str();
 115: 
 116:   return FixItHint::CreateInsertion(Loc, DestructorString);
 117: }
 118: 
 119: static std::string getSourceText(const CXXDestructorDecl &Destructor) {
 120:   std::string SourceText;
 121:   llvm::raw_string_ostream DestructorStream(SourceText);
 122:   Destructor.print(DestructorStream);
 123:   return SourceText;
 124: }
 125: 
 126: static std::string eraseKeyword(std::string &DestructorString,
 127:                                 const std::string &Keyword) {
 128:   const size_t KeywordIndex = DestructorString.find(Keyword);
```
- **Line 113 / 第 113 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 114 / 第 114 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 115 / 第 115 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 116 / 第 116 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 117 / 第 117 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 118 / 第 118 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 119 / 第 119 行**: EN: Defines function or method `getSourceText`. CN: 定义函数或方法 `getSourceText`。
- **Line 120 / 第 120 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 121 / 第 121 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 122 / 第 122 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 123 / 第 123 行**: EN: Returns a value or transfers control to the caller with `SourceText`. CN: 返回一个值，或以 `SourceText` 将控制权交还给调用者。
- **Line 124 / 第 124 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 125 / 第 125 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 126 / 第 126 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 127 / 第 127 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 128 / 第 128 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。

### Lines 129-144 / 第 129-144 行

```cpp
 129:   if (KeywordIndex != std::string::npos)
 130:     DestructorString.erase(KeywordIndex, Keyword.length());
 131:   return DestructorString;
 132: }
 133: 
 134: static FixItHint changePrivateDestructorVisibilityTo(
 135:     const std::string &Visibility, const CXXDestructorDecl &Destructor,
 136:     const SourceManager &SM, const LangOptions &LangOpts) {
 137:   std::string DestructorString =
 138:       (llvm::Twine() + Visibility + ":\n" +
 139:        (Visibility == "public" && !Destructor.isVirtual() ? "virtual " : ""))
 140:           .str();
 141: 
 142:   std::string OriginalDestructor = getSourceText(Destructor);
 143:   if (Visibility == "protected" && Destructor.isVirtualAsWritten())
 144:     OriginalDestructor = eraseKeyword(OriginalDestructor, "virtual ");
```
- **Line 129 / 第 129 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 130 / 第 130 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 131 / 第 131 行**: EN: Returns a value or transfers control to the caller with `DestructorString`. CN: 返回一个值，或以 `DestructorString` 将控制权交还给调用者。
- **Line 132 / 第 132 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 133 / 第 133 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 134 / 第 134 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 135 / 第 135 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 136 / 第 136 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 137 / 第 137 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 138 / 第 138 行**: EN: Continues logic associated with callable symbol `Twine`. CN: 继续与可调用符号 `Twine` 相关的逻辑。
- **Line 139 / 第 139 行**: EN: Continues logic associated with callable symbol `isVirtual`. CN: 继续与可调用符号 `isVirtual` 相关的逻辑。
- **Line 140 / 第 140 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 141 / 第 141 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 142 / 第 142 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 143 / 第 143 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 144 / 第 144 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。

### Lines 145-160 / 第 145-160 行

```cpp
 145: 
 146:   DestructorString =
 147:       (llvm::Twine(DestructorString) + OriginalDestructor +
 148:        (Destructor.isExplicitlyDefaulted() ? ";\n" : "") + "private:")
 149:           .str();
 150: 
 151:   /// Semicolons ending an explicitly defaulted destructor have to be deleted.
 152:   /// Otherwise, the left-over semicolon trails the \c private: access
 153:   /// specifier.
 154:   SourceLocation EndLocation;
 155:   if (Destructor.isExplicitlyDefaulted())
 156:     EndLocation =
 157:         utils::lexer::findNextTerminator(Destructor.getEndLoc(), SM, LangOpts)
 158:             .getLocWithOffset(1);
 159:   else
 160:     EndLocation = Destructor.getEndLoc().getLocWithOffset(1);
```
- **Line 145 / 第 145 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 146 / 第 146 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 147 / 第 147 行**: EN: Continues logic associated with callable symbol `Twine`. CN: 继续与可调用符号 `Twine` 相关的逻辑。
- **Line 148 / 第 148 行**: EN: Continues logic associated with callable symbol `isExplicitlyDefaulted`. CN: 继续与可调用符号 `isExplicitlyDefaulted` 相关的逻辑。
- **Line 149 / 第 149 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 150 / 第 150 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 151 / 第 151 行**: EN: Comment describing intent, behavior, or metadata: `Semicolons ending an explicitly defaulted destructor have to be deleted.`. CN: 用于说明意图、行为或元数据的注释：`Semicolons ending an explicitly defaulted destructor have to be deleted.`。
- **Line 152 / 第 152 行**: EN: Comment describing intent, behavior, or metadata: `Otherwise, the left-over semicolon trails the \c private: access`. CN: 用于说明意图、行为或元数据的注释：`Otherwise, the left-over semicolon trails the \c private: access`。
- **Line 153 / 第 153 行**: EN: Comment describing intent, behavior, or metadata: `specifier.`. CN: 用于说明意图、行为或元数据的注释：`specifier.`。
- **Line 154 / 第 154 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 155 / 第 155 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 156 / 第 156 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 157 / 第 157 行**: EN: Continues logic associated with callable symbol `findNextTerminator`. CN: 继续与可调用符号 `findNextTerminator` 相关的逻辑。
- **Line 158 / 第 158 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 159 / 第 159 行**: EN: Begins the fallback branch of a preceding conditional. CN: 开始前置条件语句的后备分支。
- **Line 160 / 第 160 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。

### Lines 161-176 / 第 161-176 行

```cpp
 161: 
 162:   auto OriginalDestructorRange =
 163:       CharSourceRange::getCharRange(Destructor.getBeginLoc(), EndLocation);
 164:   return FixItHint::CreateReplacement(OriginalDestructorRange,
 165:                                       DestructorString);
 166: }
 167: 
 168: void VirtualClassDestructorCheck::check(
 169:     const MatchFinder::MatchResult &Result) {
 170:   const auto *MatchedClassOrStruct =
 171:       Result.Nodes.getNodeAs<CXXRecordDecl>("ProblematicClassOrStruct");
 172: 
 173:   const CXXDestructorDecl *Destructor = MatchedClassOrStruct->getDestructor();
 174:   if (!Destructor)
 175:     return;
 176: 
```
- **Line 161 / 第 161 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 162 / 第 162 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 163 / 第 163 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 164 / 第 164 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 165 / 第 165 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 166 / 第 166 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 167 / 第 167 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 168 / 第 168 行**: EN: Continues logic associated with callable symbol `check`. CN: 继续与可调用符号 `check` 相关的逻辑。
- **Line 169 / 第 169 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 170 / 第 170 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 171 / 第 171 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 172 / 第 172 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 173 / 第 173 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 174 / 第 174 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 175 / 第 175 行**: EN: Returns a value or transfers control to the caller with `void`. CN: 返回一个值，或以 `void` 将控制权交还给调用者。
- **Line 176 / 第 176 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 177-192 / 第 177-192 行

```cpp
 177:   if (Destructor->getAccess() == AccessSpecifier::AS_private) {
 178:     diag(MatchedClassOrStruct->getLocation(),
 179:          "destructor of %0 is private and prevents using the type")
 180:         << MatchedClassOrStruct;
 181:     diag(MatchedClassOrStruct->getLocation(),
 182:          /*Description=*/"make it public and virtual", DiagnosticIDs::Note)
 183:         << changePrivateDestructorVisibilityTo(
 184:                "public", *Destructor, *Result.SourceManager, getLangOpts());
 185:     diag(MatchedClassOrStruct->getLocation(),
 186:          /*Description=*/"make it protected", DiagnosticIDs::Note)
 187:         << changePrivateDestructorVisibilityTo(
 188:                "protected", *Destructor, *Result.SourceManager, getLangOpts());
 189: 
 190:     return;
 191:   }
 192: 
```
- **Line 177 / 第 177 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 178 / 第 178 行**: EN: Emits a clang-tidy diagnostic, often paired with notes or fix-its. CN: 发出一条 clang-tidy 诊断，通常会配合注释或修复建议。
- **Line 179 / 第 179 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 180 / 第 180 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 181 / 第 181 行**: EN: Emits a clang-tidy diagnostic, often paired with notes or fix-its. CN: 发出一条 clang-tidy 诊断，通常会配合注释或修复建议。
- **Line 182 / 第 182 行**: EN: Comment highlights an implementation note: `Description=*/"make it public and virtual", DiagnosticIDs::Note)`. CN: 注释强调了一条实现说明：`Description=*/"make it public and virtual", DiagnosticIDs::Note)`。
- **Line 183 / 第 183 行**: EN: Continues logic associated with callable symbol `changePrivateDestructorVisibilityTo`. CN: 继续与可调用符号 `changePrivateDestructorVisibilityTo` 相关的逻辑。
- **Line 184 / 第 184 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 185 / 第 185 行**: EN: Emits a clang-tidy diagnostic, often paired with notes or fix-its. CN: 发出一条 clang-tidy 诊断，通常会配合注释或修复建议。
- **Line 186 / 第 186 行**: EN: Comment highlights an implementation note: `Description=*/"make it protected", DiagnosticIDs::Note)`. CN: 注释强调了一条实现说明：`Description=*/"make it protected", DiagnosticIDs::Note)`。
- **Line 187 / 第 187 行**: EN: Continues logic associated with callable symbol `changePrivateDestructorVisibilityTo`. CN: 继续与可调用符号 `changePrivateDestructorVisibilityTo` 相关的逻辑。
- **Line 188 / 第 188 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 189 / 第 189 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 190 / 第 190 行**: EN: Returns a value or transfers control to the caller with `void`. CN: 返回一个值，或以 `void` 将控制权交还给调用者。
- **Line 191 / 第 191 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 192 / 第 192 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 193-208 / 第 193-208 行

```cpp
 193:   // Implicit destructors are public and non-virtual for classes and structs.
 194:   bool ProtectedAndVirtual = false;
 195:   FixItHint Fix;
 196: 
 197:   if (MatchedClassOrStruct->hasUserDeclaredDestructor()) {
 198:     if (Destructor->getAccess() == AccessSpecifier::AS_public) {
 199:       Fix = FixItHint::CreateInsertion(Destructor->getLocation(), "virtual ");
 200:     } else if (Destructor->getAccess() == AccessSpecifier::AS_protected) {
 201:       ProtectedAndVirtual = true;
 202:       if (const auto MaybeRange =
 203:               getVirtualKeywordRange(*Destructor, *Result.SourceManager,
 204:                                      Result.Context->getLangOpts()))
 205:         Fix = FixItHint::CreateRemoval(*MaybeRange);
 206:     }
 207:   } else {
 208:     Fix = generateUserDeclaredDestructor(*MatchedClassOrStruct,
```
- **Line 193 / 第 193 行**: EN: Comment describing intent, behavior, or metadata: `Implicit destructors are public and non-virtual for classes and structs.`. CN: 用于说明意图、行为或元数据的注释：`Implicit destructors are public and non-virtual for classes and structs.`。
- **Line 194 / 第 194 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 195 / 第 195 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 196 / 第 196 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 197 / 第 197 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 198 / 第 198 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 199 / 第 199 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 200 / 第 200 行**: EN: Defines function or method `if`. CN: 定义函数或方法 `if`。
- **Line 201 / 第 201 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 202 / 第 202 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 203 / 第 203 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 204 / 第 204 行**: EN: Continues logic associated with callable symbol `getLangOpts`. CN: 继续与可调用符号 `getLangOpts` 相关的逻辑。
- **Line 205 / 第 205 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 206 / 第 206 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 207 / 第 207 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 208 / 第 208 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 209-222 / 第 209-222 行

```cpp
 209:                                          *Result.SourceManager);
 210:   }
 211: 
 212:   diag(MatchedClassOrStruct->getLocation(),
 213:        "destructor of %0 is %select{public and non-virtual|protected and "
 214:        "virtual}1")
 215:       << MatchedClassOrStruct << ProtectedAndVirtual;
 216:   diag(MatchedClassOrStruct->getLocation(),
 217:        "make it %select{public and virtual|protected and non-virtual}0",
 218:        DiagnosticIDs::Note)
 219:       << ProtectedAndVirtual << Fix;
 220: }
 221: 
 222: } // namespace clang::tidy::cppcoreguidelines
```
- **Line 209 / 第 209 行**: EN: Comment describing intent, behavior, or metadata: `Result.SourceManager);`. CN: 用于说明意图、行为或元数据的注释：`Result.SourceManager);`。
- **Line 210 / 第 210 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 211 / 第 211 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 212 / 第 212 行**: EN: Emits a clang-tidy diagnostic, often paired with notes or fix-its. CN: 发出一条 clang-tidy 诊断，通常会配合注释或修复建议。
- **Line 213 / 第 213 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 214 / 第 214 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 215 / 第 215 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 216 / 第 216 行**: EN: Emits a clang-tidy diagnostic, often paired with notes or fix-its. CN: 发出一条 clang-tidy 诊断，通常会配合注释或修复建议。
- **Line 217 / 第 217 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 218 / 第 218 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 219 / 第 219 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 220 / 第 220 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 221 / 第 221 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 222 / 第 222 行**: EN: Closes a namespace scope and documents which namespace ended. CN: 结束一个命名空间作用域，并说明被关闭的命名空间。

## Key Concepts / 关键概念
- **Clang-Tidy framework / Clang-Tidy 框架**: Participates in the clang-tidy architecture that wires checks, options, and diagnostics together. / 参与 clang-tidy 架构，把检查、选项和诊断连接在一起。
- **cppcoreguidelines module focus / cppcoreguidelines 模块关注点**: This file belongs to the `cppcoreguidelines` module, which concentrates on C++ Core Guidelines checks. / 该文件属于 `cppcoreguidelines` 模块，重点关注C++ Core Guidelines 检查。
- **AST matcher registration / AST Matcher 注册**: Connects declarative AST matchers to callback-based diagnostics. / 把声明式 AST Matcher 连接到基于回调的诊断逻辑。
- **Diagnostic emission / 诊断发射**: Produces clang-tidy warnings, notes, and fix-it hints. / 产生 clang-tidy 警告、注释和修复提示。
- **Automated fix-its / 自动修复建议**: Builds source edits that can be applied automatically. / 构建可自动应用的源码编辑。
- **Token-level source handling / 词法级源码处理**: Inspects tokens and spelling while preparing diagnostics or fixes. / 在准备诊断或修复时检查 token 及其拼写。

## Dependencies / 依赖关系
- **Clang/LLVM and local headers / Clang/LLVM 与本地头文件**: `VirtualClassDestructorCheck.h`, `../utils/LexerUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Lex/Lexer.h`
- **Standard library headers / 标准库头文件**: `<optional>`, `<string>`
