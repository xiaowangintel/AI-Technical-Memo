# UseStartsEndsWithCheck.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `clang-tools-extra/clang-tidy/modernize/UseStartsEndsWithCheck.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: Implements the `UseStartsEndsWithCheck` clang-tidy check in the `modernize` module around use starts ends with diagnostics and fixes.
- **Purpose (CN)**: 实现 `modernize` 模块中的 `UseStartsEndsWithCheck` clang-tidy 检查，围绕 Use Starts Ends With 相关诊断与修复展开。

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
   9: #include "UseStartsEndsWithCheck.h"
  10: 
  11: #include "../utils/ASTUtils.h"
  12: #include "../utils/Matchers.h"
  13: #include "clang/ASTMatchers/ASTMatchers.h"
  14: #include "clang/Lex/Lexer.h"
  15: 
  16: #include <string>
```
- **Line 1 / 第 1 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 2 / 第 2 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 3 / 第 3 行**: EN: Comment describing intent, behavior, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. CN: 用于说明意图、行为或元数据的注释：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **Line 4 / 第 4 行**: EN: Comment describing intent, behavior, or metadata: `See https://llvm.org/LICENSE.txt for license information.`. CN: 用于说明意图、行为或元数据的注释：`See https://llvm.org/LICENSE.txt for license information.`。
- **Line 5 / 第 5 行**: EN: Comment describing intent, behavior, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. CN: 用于说明意图、行为或元数据的注释：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **Line 6 / 第 6 行**: EN: Separator comment used for visual grouping. CN: 用于视觉分组的分隔注释。
- **Line 7 / 第 7 行**: EN: Banner comment marking a file or section boundary. CN: 横幅注释，用于标记文件或章节边界。
- **Line 8 / 第 8 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 9 / 第 9 行**: EN: Includes "UseStartsEndsWithCheck.h" so this file can use local declarations that pair with this file. CN: 包含 "UseStartsEndsWithCheck.h"，以便当前文件使用与该文件配套的本地声明。
- **Line 10 / 第 10 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 11 / 第 11 行**: EN: Includes "../utils/ASTUtils.h" so this file can use local declarations that pair with this file. CN: 包含 "../utils/ASTUtils.h"，以便当前文件使用与该文件配套的本地声明。
- **Line 12 / 第 12 行**: EN: Includes "../utils/Matchers.h" so this file can use local declarations that pair with this file. CN: 包含 "../utils/Matchers.h"，以便当前文件使用与该文件配套的本地声明。
- **Line 13 / 第 13 行**: EN: Includes "clang/ASTMatchers/ASTMatchers.h" so this file can use Clang AST matcher infrastructure. CN: 包含 "clang/ASTMatchers/ASTMatchers.h"，以便当前文件使用Clang AST Matcher 基础设施。
- **Line 14 / 第 14 行**: EN: Includes "clang/Lex/Lexer.h" so this file can use Clang lexer and preprocessor facilities. CN: 包含 "clang/Lex/Lexer.h"，以便当前文件使用Clang 词法分析与预处理设施。
- **Line 15 / 第 15 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 16 / 第 16 行**: EN: Includes <string> so this file can use supporting declarations or standard-library facilities. CN: 包含 <string>，以便当前文件使用辅助声明或标准库设施。

### Lines 17-32 / 第 17-32 行

```cpp
  17: 
  18: using namespace clang::ast_matchers;
  19: 
  20: namespace clang::tidy::modernize {
  21: 
  22: static bool isNegativeComparison(const Expr *ComparisonExpr) {
  23:   if (const auto *Op = dyn_cast<BinaryOperator>(ComparisonExpr))
  24:     return Op->getOpcode() == BO_NE;
  25: 
  26:   if (const auto *Op = dyn_cast<CXXOperatorCallExpr>(ComparisonExpr))
  27:     return Op->getOperator() == OO_ExclaimEqual;
  28: 
  29:   if (const auto *Op = dyn_cast<CXXRewrittenBinaryOperator>(ComparisonExpr))
  30:     return Op->getOperator() == BO_NE;
  31: 
  32:   return false;
```
- **Line 17 / 第 17 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 18 / 第 18 行**: EN: Brings namespace `clang::ast_matchers` into the local scope. CN: 将命名空间 `clang::ast_matchers` 引入当前作用域。
- **Line 19 / 第 19 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 20 / 第 20 行**: EN: Opens namespace `clang::tidy::modernize` to scope related declarations. CN: 打开命名空间 `clang::tidy::modernize`，为相关声明建立作用域。
- **Line 21 / 第 21 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 22 / 第 22 行**: EN: Defines function or method `isNegativeComparison`. CN: 定义函数或方法 `isNegativeComparison`。
- **Line 23 / 第 23 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 24 / 第 24 行**: EN: Returns a value or transfers control to the caller with `Op->getOpcode() == BO_NE`. CN: 返回一个值，或以 `Op->getOpcode() == BO_NE` 将控制权交还给调用者。
- **Line 25 / 第 25 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 26 / 第 26 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 27 / 第 27 行**: EN: Returns a value or transfers control to the caller with `Op->getOperator() == OO_ExclaimEqual`. CN: 返回一个值，或以 `Op->getOperator() == OO_ExclaimEqual` 将控制权交还给调用者。
- **Line 28 / 第 28 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 29 / 第 29 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 30 / 第 30 行**: EN: Returns a value or transfers control to the caller with `Op->getOperator() == BO_NE`. CN: 返回一个值，或以 `Op->getOperator() == BO_NE` 将控制权交还给调用者。
- **Line 31 / 第 31 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 32 / 第 32 行**: EN: Returns a value or transfers control to the caller with `false`. CN: 返回一个值，或以 `false` 将控制权交还给调用者。

### Lines 33-48 / 第 33-48 行

```cpp
  33: }
  34: 
  35: namespace {
  36: 
  37: struct NotLengthExprForStringNode {
  38:   NotLengthExprForStringNode(std::string ID, DynTypedNode Node,
  39:                              ASTContext *Context)
  40:       : ID(std::move(ID)), Node(std::move(Node)), Context(Context) {}
  41:   bool operator()(const internal::BoundNodesMap &Nodes) const {
  42:     // Match a string literal and an integer size or strlen() call.
  43:     if (const auto *StringLiteralNode = Nodes.getNodeAs<StringLiteral>(ID)) {
  44:       if (const auto *IntegerLiteralSizeNode = Node.get<IntegerLiteral>()) {
  45:         return StringLiteralNode->getLength() !=
  46:                IntegerLiteralSizeNode->getValue().getZExtValue();
  47:       }
  48: 
```
- **Line 33 / 第 33 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 34 / 第 34 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 35 / 第 35 行**: EN: Introduces an anonymous namespace for file-local helpers. CN: 引入匿名命名空间以承载文件局部辅助逻辑。
- **Line 36 / 第 36 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 37 / 第 37 行**: EN: Begins the declaration of struct `NotLengthExprForStringNode`. CN: 开始声明 struct `NotLengthExprForStringNode`。
- **Line 38 / 第 38 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 39 / 第 39 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 40 / 第 40 行**: EN: Continues logic associated with callable symbol `ID`. CN: 继续与可调用符号 `ID` 相关的逻辑。
- **Line 41 / 第 41 行**: EN: Defines function or method `operator`. CN: 定义函数或方法 `operator`。
- **Line 42 / 第 42 行**: EN: Comment describing intent, behavior, or metadata: `Match a string literal and an integer size or strlen() call.`. CN: 用于说明意图、行为或元数据的注释：`Match a string literal and an integer size or strlen() call.`。
- **Line 43 / 第 43 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 44 / 第 44 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 45 / 第 45 行**: EN: Returns a value or transfers control to the caller with `StringLiteralNode->getLength() !=`. CN: 返回一个值，或以 `StringLiteralNode->getLength() !=` 将控制权交还给调用者。
- **Line 46 / 第 46 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 47 / 第 47 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 48 / 第 48 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 49-64 / 第 49-64 行

```cpp
  49:       if (const auto *StrlenNode = Node.get<CallExpr>()) {
  50:         if (StrlenNode->getDirectCallee()->getName() != "strlen" ||
  51:             StrlenNode->getNumArgs() != 1) {
  52:           return true;
  53:         }
  54: 
  55:         if (const auto *StrlenArgNode = dyn_cast<StringLiteral>(
  56:                 StrlenNode->getArg(0)->IgnoreParenImpCasts())) {
  57:           return StrlenArgNode->getLength() != StringLiteralNode->getLength();
  58:         }
  59:       }
  60:     }
  61: 
  62:     // Match a string variable and a call to length() or size().
  63:     if (const auto *ExprNode = Nodes.getNodeAs<Expr>(ID)) {
  64:       if (const auto *MemberCallNode = Node.get<CXXMemberCallExpr>()) {
```
- **Line 49 / 第 49 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 50 / 第 50 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 51 / 第 51 行**: EN: Defines function or method `getNumArgs`. CN: 定义函数或方法 `getNumArgs`。
- **Line 52 / 第 52 行**: EN: Returns a value or transfers control to the caller with `true`. CN: 返回一个值，或以 `true` 将控制权交还给调用者。
- **Line 53 / 第 53 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 54 / 第 54 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 55 / 第 55 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 56 / 第 56 行**: EN: Defines function or method `getArg`. CN: 定义函数或方法 `getArg`。
- **Line 57 / 第 57 行**: EN: Returns a value or transfers control to the caller with `StrlenArgNode->getLength() != StringLiteralNode->getLength()`. CN: 返回一个值，或以 `StrlenArgNode->getLength() != StringLiteralNode->getLength()` 将控制权交还给调用者。
- **Line 58 / 第 58 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 59 / 第 59 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 60 / 第 60 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 61 / 第 61 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 62 / 第 62 行**: EN: Comment describing intent, behavior, or metadata: `Match a string variable and a call to length() or size().`. CN: 用于说明意图、行为或元数据的注释：`Match a string variable and a call to length() or size().`。
- **Line 63 / 第 63 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 64 / 第 64 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。

### Lines 65-80 / 第 65-80 行

```cpp
  65:         const CXXMethodDecl *MethodDeclNode = MemberCallNode->getMethodDecl();
  66:         const StringRef Name = MethodDeclNode->getName();
  67:         if (!MethodDeclNode->isConst() || MethodDeclNode->getNumParams() != 0 ||
  68:             (Name != "size" && Name != "length")) {
  69:           return true;
  70:         }
  71: 
  72:         if (const auto *OnNode =
  73:                 dyn_cast<Expr>(MemberCallNode->getImplicitObjectArgument())) {
  74:           return !utils::areStatementsIdentical(OnNode->IgnoreParenImpCasts(),
  75:                                                 ExprNode->IgnoreParenImpCasts(),
  76:                                                 *Context);
  77:         }
  78:       }
  79:     }
  80: 
```
- **Line 65 / 第 65 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 66 / 第 66 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 67 / 第 67 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 68 / 第 68 行**: EN: Defines function or method `callable`. CN: 定义函数或方法 `callable`。
- **Line 69 / 第 69 行**: EN: Returns a value or transfers control to the caller with `true`. CN: 返回一个值，或以 `true` 将控制权交还给调用者。
- **Line 70 / 第 70 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 71 / 第 71 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 72 / 第 72 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 73 / 第 73 行**: EN: Defines function or method `dyn_cast<Expr>`. CN: 定义函数或方法 `dyn_cast<Expr>`。
- **Line 74 / 第 74 行**: EN: Returns a value or transfers control to the caller with `!utils::areStatementsIdentical(OnNode->IgnoreParenImpCasts(),`. CN: 返回一个值，或以 `!utils::areStatementsIdentical(OnNode->IgnoreParenImpCasts(),` 将控制权交还给调用者。
- **Line 75 / 第 75 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 76 / 第 76 行**: EN: Comment describing intent, behavior, or metadata: `Context);`. CN: 用于说明意图、行为或元数据的注释：`Context);`。
- **Line 77 / 第 77 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 78 / 第 78 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 79 / 第 79 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 80 / 第 80 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 81-96 / 第 81-96 行

```cpp
  81:     return true;
  82:   }
  83: 
  84: private:
  85:   std::string ID;
  86:   DynTypedNode Node;
  87:   ASTContext *Context;
  88: };
  89: 
  90: AST_MATCHER_P(Expr, lengthExprForStringNode, std::string, ID) {
  91:   return Builder->removeBindings(NotLengthExprForStringNode(
  92:       ID, DynTypedNode::create(Node), &(Finder->getASTContext())));
  93: }
  94: 
  95: } // namespace
  96: 
```
- **Line 81 / 第 81 行**: EN: Returns a value or transfers control to the caller with `true`. CN: 返回一个值，或以 `true` 将控制权交还给调用者。
- **Line 82 / 第 82 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 83 / 第 83 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 84 / 第 84 行**: EN: Sets access control for the following members. CN: 设置后续成员的访问控制级别。
- **Line 85 / 第 85 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 86 / 第 86 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 87 / 第 87 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 88 / 第 88 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 89 / 第 89 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 90 / 第 90 行**: EN: Defines function or method `AST_MATCHER_P`. CN: 定义函数或方法 `AST_MATCHER_P`。
- **Line 91 / 第 91 行**: EN: Returns a value or transfers control to the caller with `Builder->removeBindings(NotLengthExprForStringNode(`. CN: 返回一个值，或以 `Builder->removeBindings(NotLengthExprForStringNode(` 将控制权交还给调用者。
- **Line 92 / 第 92 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 93 / 第 93 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 94 / 第 94 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 95 / 第 95 行**: EN: Closes a namespace scope and documents which namespace ended. CN: 结束一个命名空间作用域，并说明被关闭的命名空间。
- **Line 96 / 第 96 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。

### Lines 97-112 / 第 97-112 行

```cpp
  97: UseStartsEndsWithCheck::UseStartsEndsWithCheck(StringRef Name,
  98:                                                ClangTidyContext *Context)
  99:     : ClangTidyCheck(Name, Context) {}
 100: 
 101: void UseStartsEndsWithCheck::registerMatchers(MatchFinder *Finder) {
 102:   const auto ZeroLiteral = integerLiteral(equals(0));
 103: 
 104:   const auto ClassTypeWithMethod = [](const StringRef MethodBoundName,
 105:                                       const auto... Methods) {
 106:     return cxxRecordDecl(anyOf(
 107:         hasMethod(cxxMethodDecl(isConst(), parameterCountIs(1),
 108:                                 returns(booleanType()), hasAnyName(Methods))
 109:                       .bind(MethodBoundName))...));
 110:   };
 111: 
 112:   const auto OnClassWithStartsWithFunction =
```
- **Line 97 / 第 97 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 98 / 第 98 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 99 / 第 99 行**: EN: Continues logic associated with callable symbol `ClangTidyCheck`. CN: 继续与可调用符号 `ClangTidyCheck` 相关的逻辑。
- **Line 100 / 第 100 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 101 / 第 101 行**: EN: Starts or references matcher-registration logic for this check. CN: 开始或引用该检查的 Matcher 注册逻辑。
- **Line 102 / 第 102 行**: EN: Assigns or initializes state used by later logic. CN: 赋值或初始化后续逻辑要使用的状态。
- **Line 103 / 第 103 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 104 / 第 104 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 105 / 第 105 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 106 / 第 106 行**: EN: Returns a value or transfers control to the caller with `cxxRecordDecl(anyOf(`. CN: 返回一个值，或以 `cxxRecordDecl(anyOf(` 将控制权交还给调用者。
- **Line 107 / 第 107 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 108 / 第 108 行**: EN: Returns a value or transfers control to the caller with `s(booleanType()), hasAnyName(Methods))`. CN: 返回一个值，或以 `s(booleanType()), hasAnyName(Methods))` 将控制权交还给调用者。
- **Line 109 / 第 109 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 110 / 第 110 行**: EN: Terminates a type declaration or scoped block. CN: 结束一个类型声明或作用域块。
- **Line 111 / 第 111 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 112 / 第 112 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。

### Lines 113-128 / 第 113-128 行

```cpp
 113:       ClassTypeWithMethod("starts_with_fun", "starts_with", "startsWith",
 114:                           "startswith", "StartsWith");
 115: 
 116:   const auto OnClassWithEndsWithFunction = ClassTypeWithMethod(
 117:       "ends_with_fun", "ends_with", "endsWith", "endswith", "EndsWith");
 118: 
 119:   // Case 1: X.find(Y, [0], [LEN(Y)]) [!=]= 0 -> starts_with.
 120:   const auto FindExpr = cxxMemberCallExpr(
 121:       callee(
 122:           cxxMethodDecl(hasName("find"), ofClass(OnClassWithStartsWithFunction))
 123:               .bind("find_fun")),
 124:       hasArgument(0, expr().bind("needle")),
 125:       anyOf(
 126:           // Detect the expression: X.find(Y);
 127:           argumentCountIs(1),
 128:           // Detect the expression: X.find(Y, 0);
```
- **Line 113 / 第 113 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 114 / 第 114 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 115 / 第 115 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 116 / 第 116 行**: EN: Continues logic associated with callable symbol `ClassTypeWithMethod`. CN: 继续与可调用符号 `ClassTypeWithMethod` 相关的逻辑。
- **Line 117 / 第 117 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 118 / 第 118 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 119 / 第 119 行**: EN: Comment describing intent, behavior, or metadata: `Case 1: X.find(Y, [0], [LEN(Y)]) [!=]= 0 -> starts_with.`. CN: 用于说明意图、行为或元数据的注释：`Case 1: X.find(Y, [0], [LEN(Y)]) [!=]= 0 -> starts_with.`。
- **Line 120 / 第 120 行**: EN: Continues logic associated with callable symbol `cxxMemberCallExpr`. CN: 继续与可调用符号 `cxxMemberCallExpr` 相关的逻辑。
- **Line 121 / 第 121 行**: EN: Continues logic associated with callable symbol `callee`. CN: 继续与可调用符号 `callee` 相关的逻辑。
- **Line 122 / 第 122 行**: EN: Continues logic associated with callable symbol `cxxMethodDecl`. CN: 继续与可调用符号 `cxxMethodDecl` 相关的逻辑。
- **Line 123 / 第 123 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 124 / 第 124 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 125 / 第 125 行**: EN: Continues logic associated with callable symbol `anyOf`. CN: 继续与可调用符号 `anyOf` 相关的逻辑。
- **Line 126 / 第 126 行**: EN: Comment describing intent, behavior, or metadata: `Detect the expression: X.find(Y);`. CN: 用于说明意图、行为或元数据的注释：`Detect the expression: X.find(Y);`。
- **Line 127 / 第 127 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 128 / 第 128 行**: EN: Comment describing intent, behavior, or metadata: `Detect the expression: X.find(Y, 0);`. CN: 用于说明意图、行为或元数据的注释：`Detect the expression: X.find(Y, 0);`。

### Lines 129-144 / 第 129-144 行

```cpp
 129:           allOf(argumentCountIs(2), hasArgument(1, ZeroLiteral)),
 130:           // Detect the expression: X.find(Y, 0, LEN(Y));
 131:           allOf(argumentCountIs(3), hasArgument(1, ZeroLiteral),
 132:                 hasArgument(2, lengthExprForStringNode("needle")))));
 133: 
 134:   // Case 2: X.rfind(Y, 0, [LEN(Y)]) [!=]= 0 -> starts_with.
 135:   const auto RFindExpr = cxxMemberCallExpr(
 136:       callee(cxxMethodDecl(hasName("rfind"),
 137:                            ofClass(OnClassWithStartsWithFunction))
 138:                  .bind("find_fun")),
 139:       hasArgument(0, expr().bind("needle")),
 140:       anyOf(
 141:           // Detect the expression: X.rfind(Y, 0);
 142:           allOf(argumentCountIs(2), hasArgument(1, ZeroLiteral)),
 143:           // Detect the expression: X.rfind(Y, 0, LEN(Y));
 144:           allOf(argumentCountIs(3), hasArgument(1, ZeroLiteral),
```
- **Line 129 / 第 129 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 130 / 第 130 行**: EN: Comment describing intent, behavior, or metadata: `Detect the expression: X.find(Y, 0, LEN(Y));`. CN: 用于说明意图、行为或元数据的注释：`Detect the expression: X.find(Y, 0, LEN(Y));`。
- **Line 131 / 第 131 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 132 / 第 132 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 133 / 第 133 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 134 / 第 134 行**: EN: Comment describing intent, behavior, or metadata: `Case 2: X.rfind(Y, 0, [LEN(Y)]) [!=]= 0 -> starts_with.`. CN: 用于说明意图、行为或元数据的注释：`Case 2: X.rfind(Y, 0, [LEN(Y)]) [!=]= 0 -> starts_with.`。
- **Line 135 / 第 135 行**: EN: Continues logic associated with callable symbol `cxxMemberCallExpr`. CN: 继续与可调用符号 `cxxMemberCallExpr` 相关的逻辑。
- **Line 136 / 第 136 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 137 / 第 137 行**: EN: Continues logic associated with callable symbol `ofClass`. CN: 继续与可调用符号 `ofClass` 相关的逻辑。
- **Line 138 / 第 138 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 139 / 第 139 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 140 / 第 140 行**: EN: Continues logic associated with callable symbol `anyOf`. CN: 继续与可调用符号 `anyOf` 相关的逻辑。
- **Line 141 / 第 141 行**: EN: Comment describing intent, behavior, or metadata: `Detect the expression: X.rfind(Y, 0);`. CN: 用于说明意图、行为或元数据的注释：`Detect the expression: X.rfind(Y, 0);`。
- **Line 142 / 第 142 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 143 / 第 143 行**: EN: Comment describing intent, behavior, or metadata: `Detect the expression: X.rfind(Y, 0, LEN(Y));`. CN: 用于说明意图、行为或元数据的注释：`Detect the expression: X.rfind(Y, 0, LEN(Y));`。
- **Line 144 / 第 144 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 145-160 / 第 145-160 行

```cpp
 145:                 hasArgument(2, lengthExprForStringNode("needle")))));
 146: 
 147:   // Case 3: X.compare(0, LEN(Y), Y) [!=]= 0 -> starts_with.
 148:   const auto CompareExpr = cxxMemberCallExpr(
 149:       argumentCountIs(3), hasArgument(0, ZeroLiteral),
 150:       callee(cxxMethodDecl(hasName("compare"),
 151:                            ofClass(OnClassWithStartsWithFunction))
 152:                  .bind("find_fun")),
 153:       hasArgument(2, expr().bind("needle")),
 154:       hasArgument(1, lengthExprForStringNode("needle")));
 155: 
 156:   // Case 4: X.compare(LEN(X) - LEN(Y), LEN(Y), Y) [!=]= 0 -> ends_with.
 157:   const auto CompareEndsWithExpr = cxxMemberCallExpr(
 158:       argumentCountIs(3),
 159:       callee(cxxMethodDecl(hasName("compare"),
 160:                            ofClass(OnClassWithEndsWithFunction))
```
- **Line 145 / 第 145 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 146 / 第 146 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 147 / 第 147 行**: EN: Comment describing intent, behavior, or metadata: `Case 3: X.compare(0, LEN(Y), Y) [!=]= 0 -> starts_with.`. CN: 用于说明意图、行为或元数据的注释：`Case 3: X.compare(0, LEN(Y), Y) [!=]= 0 -> starts_with.`。
- **Line 148 / 第 148 行**: EN: Continues logic associated with callable symbol `cxxMemberCallExpr`. CN: 继续与可调用符号 `cxxMemberCallExpr` 相关的逻辑。
- **Line 149 / 第 149 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 150 / 第 150 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 151 / 第 151 行**: EN: Continues logic associated with callable symbol `ofClass`. CN: 继续与可调用符号 `ofClass` 相关的逻辑。
- **Line 152 / 第 152 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 153 / 第 153 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 154 / 第 154 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 155 / 第 155 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 156 / 第 156 行**: EN: Comment describing intent, behavior, or metadata: `Case 4: X.compare(LEN(X) - LEN(Y), LEN(Y), Y) [!=]= 0 -> ends_with.`. CN: 用于说明意图、行为或元数据的注释：`Case 4: X.compare(LEN(X) - LEN(Y), LEN(Y), Y) [!=]= 0 -> ends_with.`。
- **Line 157 / 第 157 行**: EN: Continues logic associated with callable symbol `cxxMemberCallExpr`. CN: 继续与可调用符号 `cxxMemberCallExpr` 相关的逻辑。
- **Line 158 / 第 158 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 159 / 第 159 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 160 / 第 160 行**: EN: Continues logic associated with callable symbol `ofClass`. CN: 继续与可调用符号 `ofClass` 相关的逻辑。

### Lines 161-176 / 第 161-176 行

```cpp
 161:                  .bind("find_fun")),
 162:       on(expr().bind("haystack")), hasArgument(2, expr().bind("needle")),
 163:       hasArgument(1, lengthExprForStringNode("needle")),
 164:       hasArgument(0,
 165:                   binaryOperator(hasOperatorName("-"),
 166:                                  hasLHS(lengthExprForStringNode("haystack")),
 167:                                  hasRHS(lengthExprForStringNode("needle")))));
 168: 
 169:   // All cases comparing to 0.
 170:   Finder->addMatcher(
 171:       binaryOperator(
 172:           matchers::isEqualityOperator(),
 173:           hasOperands(cxxMemberCallExpr(anyOf(FindExpr, RFindExpr, CompareExpr,
 174:                                               CompareEndsWithExpr))
 175:                           .bind("find_expr"),
 176:                       ZeroLiteral))
```
- **Line 161 / 第 161 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 162 / 第 162 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 163 / 第 163 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 164 / 第 164 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 165 / 第 165 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 166 / 第 166 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 167 / 第 167 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 168 / 第 168 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 169 / 第 169 行**: EN: Comment describing intent, behavior, or metadata: `All cases comparing to 0.`. CN: 用于说明意图、行为或元数据的注释：`All cases comparing to 0.`。
- **Line 170 / 第 170 行**: EN: Registers an AST matcher that will trigger this check on matching nodes. CN: 注册一个 AST Matcher，使该检查在节点匹配时被触发。
- **Line 171 / 第 171 行**: EN: Continues logic associated with callable symbol `binaryOperator`. CN: 继续与可调用符号 `binaryOperator` 相关的逻辑。
- **Line 172 / 第 172 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 173 / 第 173 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 174 / 第 174 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 175 / 第 175 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 176 / 第 176 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。

### Lines 177-192 / 第 177-192 行

```cpp
 177:           .bind("expr"),
 178:       this);
 179: 
 180:   // Case 5: X.rfind(Y) [!=]= LEN(X) - LEN(Y) -> ends_with.
 181:   Finder->addMatcher(
 182:       binaryOperator(
 183:           matchers::isEqualityOperator(),
 184:           hasOperands(
 185:               cxxMemberCallExpr(
 186:                   anyOf(
 187:                       argumentCountIs(1),
 188:                       allOf(argumentCountIs(2),
 189:                             hasArgument(
 190:                                 1,
 191:                                 anyOf(declRefExpr(to(varDecl(hasName("npos")))),
 192:                                       memberExpr(member(hasName("npos"))))))),
```
- **Line 177 / 第 177 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 178 / 第 178 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 179 / 第 179 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 180 / 第 180 行**: EN: Comment describing intent, behavior, or metadata: `Case 5: X.rfind(Y) [!=]= LEN(X) - LEN(Y) -> ends_with.`. CN: 用于说明意图、行为或元数据的注释：`Case 5: X.rfind(Y) [!=]= LEN(X) - LEN(Y) -> ends_with.`。
- **Line 181 / 第 181 行**: EN: Registers an AST matcher that will trigger this check on matching nodes. CN: 注册一个 AST Matcher，使该检查在节点匹配时被触发。
- **Line 182 / 第 182 行**: EN: Continues logic associated with callable symbol `binaryOperator`. CN: 继续与可调用符号 `binaryOperator` 相关的逻辑。
- **Line 183 / 第 183 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 184 / 第 184 行**: EN: Continues logic associated with callable symbol `hasOperands`. CN: 继续与可调用符号 `hasOperands` 相关的逻辑。
- **Line 185 / 第 185 行**: EN: Continues logic associated with callable symbol `cxxMemberCallExpr`. CN: 继续与可调用符号 `cxxMemberCallExpr` 相关的逻辑。
- **Line 186 / 第 186 行**: EN: Continues logic associated with callable symbol `anyOf`. CN: 继续与可调用符号 `anyOf` 相关的逻辑。
- **Line 187 / 第 187 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 188 / 第 188 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 189 / 第 189 行**: EN: Continues logic associated with callable symbol `hasArgument`. CN: 继续与可调用符号 `hasArgument` 相关的逻辑。
- **Line 190 / 第 190 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 191 / 第 191 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 192 / 第 192 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 193-208 / 第 193-208 行

```cpp
 193:                   callee(cxxMethodDecl(hasName("rfind"),
 194:                                        ofClass(OnClassWithEndsWithFunction))
 195:                              .bind("find_fun")),
 196:                   on(expr().bind("haystack")),
 197:                   hasArgument(0, expr().bind("needle")))
 198:                   .bind("find_expr"),
 199:               binaryOperator(hasOperatorName("-"),
 200:                              hasLHS(lengthExprForStringNode("haystack")),
 201:                              hasRHS(lengthExprForStringNode("needle")))))
 202:           .bind("expr"),
 203:       this);
 204: 
 205:   // Case 6: X.substr(0, LEN(Y)) [!=]= Y -> starts_with.
 206:   Finder->addMatcher(
 207:       binaryOperation(
 208:           hasAnyOperatorName("==", "!="),
```
- **Line 193 / 第 193 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 194 / 第 194 行**: EN: Continues logic associated with callable symbol `ofClass`. CN: 继续与可调用符号 `ofClass` 相关的逻辑。
- **Line 195 / 第 195 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 196 / 第 196 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 197 / 第 197 行**: EN: Continues logic associated with callable symbol `hasArgument`. CN: 继续与可调用符号 `hasArgument` 相关的逻辑。
- **Line 198 / 第 198 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 199 / 第 199 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 200 / 第 200 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 201 / 第 201 行**: EN: Continues logic associated with callable symbol `hasRHS`. CN: 继续与可调用符号 `hasRHS` 相关的逻辑。
- **Line 202 / 第 202 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 203 / 第 203 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 204 / 第 204 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 205 / 第 205 行**: EN: Comment describing intent, behavior, or metadata: `Case 6: X.substr(0, LEN(Y)) [!=]= Y -> starts_with.`. CN: 用于说明意图、行为或元数据的注释：`Case 6: X.substr(0, LEN(Y)) [!=]= Y -> starts_with.`。
- **Line 206 / 第 206 行**: EN: Registers an AST matcher that will trigger this check on matching nodes. CN: 注册一个 AST Matcher，使该检查在节点匹配时被触发。
- **Line 207 / 第 207 行**: EN: Continues logic associated with callable symbol `binaryOperation`. CN: 继续与可调用符号 `binaryOperation` 相关的逻辑。
- **Line 208 / 第 208 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。

### Lines 209-224 / 第 209-224 行

```cpp
 209:           hasOperands(
 210:               expr().bind("needle"),
 211:               cxxMemberCallExpr(
 212:                   argumentCountIs(2), hasArgument(0, ZeroLiteral),
 213:                   hasArgument(1, lengthExprForStringNode("needle")),
 214:                   callee(cxxMethodDecl(hasName("substr"),
 215:                                        ofClass(OnClassWithStartsWithFunction))
 216:                              .bind("find_fun")))
 217:                   .bind("find_expr")))
 218:           .bind("expr"),
 219:       this);
 220: }
 221: 
 222: void UseStartsEndsWithCheck::check(const MatchFinder::MatchResult &Result) {
 223:   const auto *ComparisonExpr = Result.Nodes.getNodeAs<Expr>("expr");
 224:   const auto *FindExpr = Result.Nodes.getNodeAs<CXXMemberCallExpr>("find_expr");
```
- **Line 209 / 第 209 行**: EN: Continues logic associated with callable symbol `hasOperands`. CN: 继续与可调用符号 `hasOperands` 相关的逻辑。
- **Line 210 / 第 210 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 211 / 第 211 行**: EN: Continues logic associated with callable symbol `cxxMemberCallExpr`. CN: 继续与可调用符号 `cxxMemberCallExpr` 相关的逻辑。
- **Line 212 / 第 212 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 213 / 第 213 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 214 / 第 214 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 215 / 第 215 行**: EN: Continues logic associated with callable symbol `ofClass`. CN: 继续与可调用符号 `ofClass` 相关的逻辑。
- **Line 216 / 第 216 行**: EN: Continues logic associated with callable symbol `bind`. CN: 继续与可调用符号 `bind` 相关的逻辑。
- **Line 217 / 第 217 行**: EN: Continues logic associated with callable symbol `bind`. CN: 继续与可调用符号 `bind` 相关的逻辑。
- **Line 218 / 第 218 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 219 / 第 219 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 220 / 第 220 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 221 / 第 221 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 222 / 第 222 行**: EN: Defines function or method `check`. CN: 定义函数或方法 `check`。
- **Line 223 / 第 223 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 224 / 第 224 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。

### Lines 225-240 / 第 225-240 行

```cpp
 225:   const auto *FindFun = Result.Nodes.getNodeAs<CXXMethodDecl>("find_fun");
 226:   const auto *SearchExpr = Result.Nodes.getNodeAs<Expr>("needle");
 227:   const auto *StartsWithFunction =
 228:       Result.Nodes.getNodeAs<CXXMethodDecl>("starts_with_fun");
 229:   const auto *EndsWithFunction =
 230:       Result.Nodes.getNodeAs<CXXMethodDecl>("ends_with_fun");
 231:   assert(bool(StartsWithFunction) != bool(EndsWithFunction));
 232: 
 233:   const CXXMethodDecl *ReplacementFunction =
 234:       StartsWithFunction ? StartsWithFunction : EndsWithFunction;
 235: 
 236:   if (ComparisonExpr->getBeginLoc().isMacroID() ||
 237:       FindExpr->getBeginLoc().isMacroID())
 238:     return;
 239: 
 240:   // Make sure FindExpr->getArg(0) can be used to make a range in the FitItHint.
```
- **Line 225 / 第 225 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 226 / 第 226 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 227 / 第 227 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 228 / 第 228 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 229 / 第 229 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 230 / 第 230 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 231 / 第 231 行**: EN: Checks an internal invariant in debug builds. CN: 在调试构建中检查内部不变式。
- **Line 232 / 第 232 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 233 / 第 233 行**: EN: Continues the surrounding expression or declaration. CN: 继续构造周围的表达式或声明。
- **Line 234 / 第 234 行**: EN: Performs a small step in the surrounding implementation. CN: 在周围实现中执行一个小步骤。
- **Line 235 / 第 235 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 236 / 第 236 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 237 / 第 237 行**: EN: Continues logic associated with callable symbol `getBeginLoc`. CN: 继续与可调用符号 `getBeginLoc` 相关的逻辑。
- **Line 238 / 第 238 行**: EN: Returns a value or transfers control to the caller with `void`. CN: 返回一个值，或以 `void` 将控制权交还给调用者。
- **Line 239 / 第 239 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 240 / 第 240 行**: EN: Comment describing intent, behavior, or metadata: `Make sure FindExpr->getArg(0) can be used to make a range in the FitItHint.`. CN: 用于说明意图、行为或元数据的注释：`Make sure FindExpr->getArg(0) can be used to make a range in the FitItHint.`。

### Lines 241-256 / 第 241-256 行

```cpp
 241:   if (FindExpr->getNumArgs() == 0)
 242:     return;
 243: 
 244:   // Retrieve the source text of the search expression.
 245:   const auto SearchExprText = Lexer::getSourceText(
 246:       CharSourceRange::getTokenRange(SearchExpr->getSourceRange()),
 247:       *Result.SourceManager, Result.Context->getLangOpts());
 248: 
 249:   auto Diagnostic = diag(FindExpr->getExprLoc(), "use %0 instead of %1")
 250:                     << ReplacementFunction->getName() << FindFun->getName();
 251: 
 252:   // Remove everything before the function call.
 253:   Diagnostic << FixItHint::CreateRemoval(CharSourceRange::getCharRange(
 254:       ComparisonExpr->getBeginLoc(), FindExpr->getBeginLoc()));
 255: 
 256:   // Rename the function to `starts_with` or `ends_with`.
```
- **Line 241 / 第 241 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 242 / 第 242 行**: EN: Returns a value or transfers control to the caller with `void`. CN: 返回一个值，或以 `void` 将控制权交还给调用者。
- **Line 243 / 第 243 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 244 / 第 244 行**: EN: Comment describing intent, behavior, or metadata: `Retrieve the source text of the search expression.`. CN: 用于说明意图、行为或元数据的注释：`Retrieve the source text of the search expression.`。
- **Line 245 / 第 245 行**: EN: Continues logic associated with callable symbol `getSourceText`. CN: 继续与可调用符号 `getSourceText` 相关的逻辑。
- **Line 246 / 第 246 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 247 / 第 247 行**: EN: Comment describing intent, behavior, or metadata: `Result.SourceManager, Result.Context->getLangOpts());`. CN: 用于说明意图、行为或元数据的注释：`Result.SourceManager, Result.Context->getLangOpts());`。
- **Line 248 / 第 248 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 249 / 第 249 行**: EN: Continues logic associated with callable symbol `diag`. CN: 继续与可调用符号 `diag` 相关的逻辑。
- **Line 250 / 第 250 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 251 / 第 251 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 252 / 第 252 行**: EN: Comment describing intent, behavior, or metadata: `Remove everything before the function call.`. CN: 用于说明意图、行为或元数据的注释：`Remove everything before the function call.`。
- **Line 253 / 第 253 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 254 / 第 254 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 255 / 第 255 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 256 / 第 256 行**: EN: Comment describing intent, behavior, or metadata: `Rename the function to `starts_with` or `ends_with`.`. CN: 用于说明意图、行为或元数据的注释：`Rename the function to `starts_with` or `ends_with`.`。

### Lines 257-271 / 第 257-271 行

```cpp
 257:   Diagnostic << FixItHint::CreateReplacement(FindExpr->getExprLoc(),
 258:                                              ReplacementFunction->getName());
 259: 
 260:   // Replace arguments and everything after the function call.
 261:   Diagnostic << FixItHint::CreateReplacement(
 262:       CharSourceRange::getTokenRange(FindExpr->getArg(0)->getBeginLoc(),
 263:                                      ComparisonExpr->getEndLoc()),
 264:       (SearchExprText + ")").str());
 265: 
 266:   // Add negation if necessary.
 267:   if (isNegativeComparison(ComparisonExpr))
 268:     Diagnostic << FixItHint::CreateInsertion(FindExpr->getBeginLoc(), "!");
 269: }
 270: 
 271: } // namespace clang::tidy::modernize
```
- **Line 257 / 第 257 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 258 / 第 258 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 259 / 第 259 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 260 / 第 260 行**: EN: Comment describing intent, behavior, or metadata: `Replace arguments and everything after the function call.`. CN: 用于说明意图、行为或元数据的注释：`Replace arguments and everything after the function call.`。
- **Line 261 / 第 261 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 262 / 第 262 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 263 / 第 263 行**: EN: Continues a multi-line declaration, call, or initializer. CN: 继续一个跨多行的声明、调用或初始化。
- **Line 264 / 第 264 行**: EN: Calls a helper routine or method to advance the implementation. CN: 调用辅助例程或方法以推进实现逻辑。
- **Line 265 / 第 265 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 266 / 第 266 行**: EN: Comment describing intent, behavior, or metadata: `Add negation if necessary.`. CN: 用于说明意图、行为或元数据的注释：`Add negation if necessary.`。
- **Line 267 / 第 267 行**: EN: Checks a condition before executing the guarded branch. CN: 先检查条件，再执行受保护的分支。
- **Line 268 / 第 268 行**: EN: Constructs a fix-it hint describing an automatic source edit. CN: 构造一个 fix-it 提示，用于描述自动源码编辑。
- **Line 269 / 第 269 行**: EN: Closes the current scope or block. CN: 关闭当前作用域或代码块。
- **Line 270 / 第 270 行**: EN: Blank line separating logical blocks. CN: 空行，用于分隔逻辑块。
- **Line 271 / 第 271 行**: EN: Closes a namespace scope and documents which namespace ended. CN: 结束一个命名空间作用域，并说明被关闭的命名空间。

## Key Concepts / 关键概念
- **Clang-Tidy framework / Clang-Tidy 框架**: Participates in the clang-tidy architecture that wires checks, options, and diagnostics together. / 参与 clang-tidy 架构，把检查、选项和诊断连接在一起。
- **modernize module focus / modernize 模块关注点**: This file belongs to the `modernize` module, which concentrates on modern C++ migration checks. / 该文件属于 `modernize` 模块，重点关注现代 C++ 迁移检查。
- **Clang-Tidy check lifecycle / Clang-Tidy 检查生命周期**: Defines or uses the standard hook points of a clang-tidy check. / 定义或使用 clang-tidy 检查的标准钩子。
- **Shared analysis context / 共享分析上下文**: Carries options, diagnostics, language mode, and per-run shared state. / 承载选项、诊断、语言模式以及每次运行的共享状态。
- **AST matcher registration / AST Matcher 注册**: Connects declarative AST matchers to callback-based diagnostics. / 把声明式 AST Matcher 连接到基于回调的诊断逻辑。
- **Diagnostic emission / 诊断发射**: Produces clang-tidy warnings, notes, and fix-it hints. / 产生 clang-tidy 警告、注释和修复提示。

## Dependencies / 依赖关系
- **Clang/LLVM and local headers / Clang/LLVM 与本地头文件**: `UseStartsEndsWithCheck.h`, `../utils/ASTUtils.h`, `../utils/Matchers.h`, `clang/ASTMatchers/ASTMatchers.h`, `clang/Lex/Lexer.h`
- **Standard library headers / 标准库头文件**: `<string>`
