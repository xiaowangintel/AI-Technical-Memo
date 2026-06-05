# RefactoringCallbacks.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Tooling/RefactoringCallbacks.h`
- Repository: `llvm-project`
- Purpose (EN): Structural query framework.
- 用途（中文）: 该文件为 Tooling 子系统中的 Refactoring Callbacks 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
 1: //===--- RefactoringCallbacks.h - Structural query framework ----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  Provides callbacks to make common kinds of refactorings easy.
10: //
11: //  The general idea is to construct a matcher expression that describes a
12: //  subtree match on the AST and then replace the corresponding source code
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 13-24

```cpp
13: //  either by some specific text or some other AST node.
14: //
15: //  Example:
16: //  int main(int argc, char **argv) {
17: //    ClangTool Tool(argc, argv);
18: //    MatchFinder Finder;
19: //    ReplaceStmtWithText Callback("integer", "42");
20: //    Finder.AddMatcher(id("integer", expression(integerLiteral())), Callback);
21: //    return Tool.run(newFrontendActionFactory(&Finder));
22: //  }
23: //
24: //  This will replace all integer literals with "42".
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 25-36

```cpp
25: //
26: //===----------------------------------------------------------------------===//
27: 
28: #ifndef LLVM_CLANG_TOOLING_REFACTORINGCALLBACKS_H
29: #define LLVM_CLANG_TOOLING_REFACTORINGCALLBACKS_H
30: 
31: #include "clang/ASTMatchers/ASTMatchFinder.h"
32: #include "clang/Tooling/Refactoring.h"
33: 
34: namespace clang {
35: namespace tooling {
36: 
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Tooling/Refactoring.h`. It opens, closes, or documents namespace scope for `clang`, `tooling`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Tooling/Refactoring.h` 等依赖。 它打开、关闭或说明了 `clang`, `tooling` 的命名空间作用域。

### Lines 37-48

```cpp
37: /// Base class for RefactoringCallbacks.
38: ///
39: /// Collects \c tooling::Replacements while running.
40: class RefactoringCallback : public ast_matchers::MatchFinder::MatchCallback {
41: public:
42:   RefactoringCallback();
43:   Replacements &getReplacements();
44: 
45: protected:
46:   Replacements Replace;
47: };
48: 
```
- EN: Key type declarations here include `RefactoringCallback`. It exposes API surface such as `RefactoringCallback`, `getReplacements`.
- 中文: 这里的重要类型声明包括 `RefactoringCallback`。 它暴露了 `RefactoringCallback`, `getReplacements` 等接口。

### Lines 49-60

```cpp
49: /// Adaptor between \c ast_matchers::MatchFinder and \c
50: /// tooling::RefactoringTool.
51: ///
52: /// Runs AST matchers and stores the \c tooling::Replacements in a map.
53: class ASTMatchRefactorer {
54: public:
55:   explicit ASTMatchRefactorer(
56:     std::map<std::string, Replacements> &FileToReplaces);
57: 
58:   template <typename T>
59:   void addMatcher(const T &Matcher, RefactoringCallback *Callback) {
60:     MatchFinder.addMatcher(Matcher, Callback);
```
- EN: Key type declarations here include `ASTMatchRefactorer`. It exposes API surface such as `addMatcher`.
- 中文: 这里的重要类型声明包括 `ASTMatchRefactorer`。 它暴露了 `addMatcher` 等接口。

### Lines 61-72

```cpp
61:     Callbacks.push_back(Callback);
62:   }
63: 
64:   void addDynamicMatcher(const ast_matchers::internal::DynTypedMatcher &Matcher,
65:                          RefactoringCallback *Callback);
66: 
67:   std::unique_ptr<ASTConsumer> newASTConsumer();
68: 
69: private:
70:   friend class RefactoringASTConsumer;
71:   std::vector<RefactoringCallback *> Callbacks;
72:   ast_matchers::MatchFinder MatchFinder;
```
- EN: Key type declarations here include `RefactoringASTConsumer`. It exposes API surface such as `push_back`, `newASTConsumer`.
- 中文: 这里的重要类型声明包括 `RefactoringASTConsumer`。 它暴露了 `push_back`, `newASTConsumer` 等接口。

### Lines 73-84

```cpp
73:   std::map<std::string, Replacements> &FileToReplaces;
74: };
75: 
76: /// Replace the text of the statement bound to \c FromId with the text in
77: /// \c ToText.
78: class ReplaceStmtWithText : public RefactoringCallback {
79: public:
80:   ReplaceStmtWithText(StringRef FromId, StringRef ToText);
81:   void run(const ast_matchers::MatchFinder::MatchResult &Result) override;
82: 
83: private:
84:   std::string FromId;
```
- EN: Key type declarations here include `ReplaceStmtWithText`. It exposes API surface such as `ReplaceStmtWithText`.
- 中文: 这里的重要类型声明包括 `ReplaceStmtWithText`。 它暴露了 `ReplaceStmtWithText` 等接口。

### Lines 85-96

```cpp
85:   std::string ToText;
86: };
87: 
88: /// Replace the text of an AST node bound to \c FromId with the result of
89: /// evaluating the template in \c ToTemplate.
90: ///
91: /// Expressions of the form ${NodeName} in \c ToTemplate will be
92: /// replaced by the text of the node bound to ${NodeName}. The string
93: /// "$$" will be replaced by "$".
94: class ReplaceNodeWithTemplate : public RefactoringCallback {
95: public:
96:   static llvm::Expected<std::unique_ptr<ReplaceNodeWithTemplate>>
```
- EN: Key type declarations here include `ReplaceNodeWithTemplate`.
- 中文: 这里的重要类型声明包括 `ReplaceNodeWithTemplate`。

### Lines 97-108

```cpp
 97:   create(StringRef FromId, StringRef ToTemplate);
 98:   void run(const ast_matchers::MatchFinder::MatchResult &Result) override;
 99: 
100: private:
101:   struct TemplateElement {
102:     enum { Literal, Identifier } Type;
103:     std::string Value;
104:   };
105:   ReplaceNodeWithTemplate(llvm::StringRef FromId,
106:                           std::vector<TemplateElement> Template);
107:   std::string FromId;
108:   std::vector<TemplateElement> Template;
```
- EN: Key type declarations here include `TemplateElement`. It exposes API surface such as `create`.
- 中文: 这里的重要类型声明包括 `TemplateElement`。 它暴露了 `create` 等接口。

### Lines 109-120

```cpp
109: };
110: 
111: /// Replace the text of the statement bound to \c FromId with the text of
112: /// the statement bound to \c ToId.
113: class ReplaceStmtWithStmt : public RefactoringCallback {
114: public:
115:   ReplaceStmtWithStmt(StringRef FromId, StringRef ToId);
116:   void run(const ast_matchers::MatchFinder::MatchResult &Result) override;
117: 
118: private:
119:   std::string FromId;
120:   std::string ToId;
```
- EN: Key type declarations here include `ReplaceStmtWithStmt`. It exposes API surface such as `ReplaceStmtWithStmt`.
- 中文: 这里的重要类型声明包括 `ReplaceStmtWithStmt`。 它暴露了 `ReplaceStmtWithStmt` 等接口。

### Lines 121-132

```cpp
121: };
122: 
123: /// Replace an if-statement bound to \c Id with the outdented text of its
124: /// body, choosing the consequent or the alternative based on whether
125: /// \c PickTrueBranch is true.
126: class ReplaceIfStmtWithItsBody : public RefactoringCallback {
127: public:
128:   ReplaceIfStmtWithItsBody(StringRef Id, bool PickTrueBranch);
129:   void run(const ast_matchers::MatchFinder::MatchResult &Result) override;
130: 
131: private:
132:   std::string Id;
```
- EN: Key type declarations here include `ReplaceIfStmtWithItsBody`. It exposes API surface such as `ReplaceIfStmtWithItsBody`.
- 中文: 这里的重要类型声明包括 `ReplaceIfStmtWithItsBody`。 它暴露了 `ReplaceIfStmtWithItsBody` 等接口。

### Lines 133-139

```cpp
133:   const bool PickTrueBranch;
134: };
135: 
136: } // end namespace tooling
137: } // end namespace clang
138: 
139: #endif
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `tooling`, `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `tooling`, `clang` 的命名空间作用域。

## Key Concepts / 关键概念

- `RefactoringCallback`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ASTMatchRefactorer`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `RefactoringASTConsumer`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ReplaceStmtWithText`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ReplaceNodeWithTemplate`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `TemplateElement`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ReplaceStmtWithStmt`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ReplaceIfStmtWithItsBody`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Tooling/Refactoring.h`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`, `tooling`
- Macro-style dependencies / 宏式依赖: None / 无
