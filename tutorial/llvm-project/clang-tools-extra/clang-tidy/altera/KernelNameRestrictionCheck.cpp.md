# KernelNameRestrictionCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/altera/KernelNameRestrictionCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Defines the diagnostic and helper logic behind `KernelNameRestrictionPPCallbacks`.
- 用途 (CN): 定义 `KernelNameRestrictionPPCallbacks` 背后的诊断与辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8
```cpp
   1 | //===----------------------------------------------------------------------===//
   2 | //
   3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4 | // See https://llvm.org/LICENSE.txt for license information.
   5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6 | //
   7 | //===----------------------------------------------------------------------===//
   8 | 
```
- EN: Standard LLVM banner and licensing notice for the file.
- CN: 这是文件的标准 LLVM 版权与许可证说明。

### Lines 9-14
```cpp
   9 | #include "KernelNameRestrictionCheck.h"
  10 | #include "clang/Frontend/CompilerInstance.h"
  11 | #include "clang/Lex/PPCallbacks.h"
  12 | #include "clang/Lex/Preprocessor.h"
  13 | #include <vector>
  14 | 
```
- EN: The section imports dependencies such as `KernelNameRestrictionCheck.h`, `clang/Frontend/CompilerInstance.h`, `clang/Lex/PPCallbacks.h`, `clang/Lex/Preprocessor.h` needed by this file.
- CN: 本段引入了 `KernelNameRestrictionCheck.h`、`clang/Frontend/CompilerInstance.h`、`clang/Lex/PPCallbacks.h`、`clang/Lex/Preprocessor.h` 等依赖，供当前文件使用。

### Lines 15-18
```cpp
  15 | using namespace clang::ast_matchers;
  16 | 
  17 | namespace clang::tidy::altera {
  18 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::altera` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::altera` 这样的命名空间将符号放入预期的子系统中。

### Lines 19-26
```cpp
  19 | namespace {
  20 | 
  21 | class KernelNameRestrictionPPCallbacks : public PPCallbacks {
  22 | public:
  23 |   explicit KernelNameRestrictionPPCallbacks(ClangTidyCheck &Check,
  24 |                                             const SourceManager &SM)
  25 |       : Check(Check), SM(SM) {}
  26 | 
```
- EN: It declares class `KernelNameRestrictionPPCallbacks` and derives from `PPCallbacks`, which defines the framework contract it follows.
- CN: 这里声明类 `KernelNameRestrictionPPCallbacks`，并继承自 `PPCallbacks`，说明它遵循的框架契约。

### Lines 27-34
```cpp
  27 |   void InclusionDirective(SourceLocation HashLoc, const Token &IncludeTok,
  28 |                           StringRef FileName, bool IsAngled,
  29 |                           CharSourceRange FileNameRange,
  30 |                           OptionalFileEntryRef File, StringRef SearchPath,
  31 |                           StringRef RelativePath, const Module *SuggestedModule,
  32 |                           bool ModuleImported,
  33 |                           SrcMgr::CharacteristicKind FileType) override;
  34 | 
```
- EN: This block continues the implementation with declarations or statements centered on `void InclusionDirective(SourceLocation HashLoc, const Token `.
- CN: 这一段继续实现，围绕 `void InclusionDirective(SourceLocation HashLoc, const Token ` 展开声明或语句。

### Lines 35-41
```cpp
  35 |   void EndOfMainFile() override;
  36 | 
  37 | private:
  38 |   /// Returns true if the name of the file with path FileName is 'kernel.cl',
  39 |   /// 'verilog.cl', or 'vhdl.cl'. The file name check is case insensitive.
  40 |   bool fileNameIsRestricted(StringRef FileName);
  41 | 
```
- EN: This block continues the implementation with declarations or statements centered on `void EndOfMainFile() override;`.
- CN: 这一段继续实现，围绕 `void EndOfMainFile() override;` 展开声明或语句。

### Lines 42-46
```cpp
  42 |   struct IncludeDirective {
  43 |     SourceLocation Loc; // Location in the include directive.
  44 |     StringRef FileName; // Filename as a string.
  45 |   };
  46 | 
```
- EN: It declares class `IncludeDirective` as a key type for this file.
- CN: 这里声明类 `IncludeDirective`，它是当前文件的核心类型。

### Lines 47-51
```cpp
  47 |   std::vector<IncludeDirective> IncludeDirectives;
  48 |   ClangTidyCheck &Check;
  49 |   const SourceManager &SM;
  50 | };
  51 | 
```
- EN: This block continues the implementation with declarations or statements centered on `std::vector<IncludeDirective> IncludeDirectives;`.
- CN: 这一段继续实现，围绕 `std::vector<IncludeDirective> IncludeDirectives;` 展开声明或语句。

### Lines 52-60
```cpp
  52 | } // namespace
  53 | 
  54 | void KernelNameRestrictionCheck::registerPPCallbacks(const SourceManager &SM,
  55 |                                                      Preprocessor *PP,
  56 |                                                      Preprocessor *) {
  57 |   PP->addPPCallbacks(
  58 |       std::make_unique<KernelNameRestrictionPPCallbacks>(*this, SM));
  59 | }
  60 | 
```
- EN: Method definitions such as `KernelNameRestrictionCheck::registerPPCallbacks` provide the concrete behavior declared elsewhere.
- CN: 诸如 `KernelNameRestrictionCheck::registerPPCallbacks` 的方法定义给出了前面声明的具体行为。

### Lines 61-68
```cpp
  61 | void KernelNameRestrictionPPCallbacks::InclusionDirective(
  62 |     SourceLocation HashLoc, const Token &, StringRef FileName, bool,
  63 |     CharSourceRange, OptionalFileEntryRef, StringRef, StringRef, const Module *,
  64 |     bool, SrcMgr::CharacteristicKind) {
  65 |   IncludeDirective ID = {HashLoc, FileName};
  66 |   IncludeDirectives.push_back(std::move(ID));
  67 | }
  68 | 
```
- EN: Method definitions such as `KernelNameRestrictionPPCallbacks::InclusionDirective` provide the concrete behavior declared elsewhere.
- CN: 诸如 `KernelNameRestrictionPPCallbacks::InclusionDirective` 的方法定义给出了前面声明的具体行为。

### Lines 69-75
```cpp
  69 | bool KernelNameRestrictionPPCallbacks::fileNameIsRestricted(
  70 |     StringRef FileName) {
  71 |   return FileName.equals_insensitive("kernel.cl") ||
  72 |          FileName.equals_insensitive("verilog.cl") ||
  73 |          FileName.equals_insensitive("vhdl.cl");
  74 | }
  75 | 
```
- EN: Method definitions such as `KernelNameRestrictionPPCallbacks::fileNameIsRestricted` provide the concrete behavior declared elsewhere.
- CN: 诸如 `KernelNameRestrictionPPCallbacks::fileNameIsRestricted` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 76-85
```cpp
  76 | void KernelNameRestrictionPPCallbacks::EndOfMainFile() {
  77 |   // Check main file for restricted names.
  78 |   OptionalFileEntryRef Entry = SM.getFileEntryRefForID(SM.getMainFileID());
  79 |   const StringRef FileName = llvm::sys::path::filename(Entry->getName());
  80 |   if (fileNameIsRestricted(FileName))
  81 |     Check.diag(SM.getLocForStartOfFile(SM.getMainFileID()),
  82 |                "compiling '%0' may cause additional compilation errors due "
  83 |                "to the name of the kernel source file; consider renaming the "
  84 |                "included kernel source file")
  85 |         << FileName;
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `KernelNameRestrictionPPCallbacks::EndOfMainFile` provide the concrete behavior declared elsewhere.
- CN: 诸如 `KernelNameRestrictionPPCallbacks::EndOfMainFile` 的方法定义给出了前面声明的具体行为。

### Lines 86-89
```cpp
  86 | 
  87 |   if (IncludeDirectives.empty())
  88 |     return;
  89 | 
```
- EN: This block continues the implementation with declarations or statements centered on `if (IncludeDirectives.empty())`.
- CN: 这一段继续实现，围绕 `if (IncludeDirectives.empty())` 展开声明或语句。

### Lines 90-99
```cpp
  90 |   // Check included files for restricted names.
  91 |   for (const IncludeDirective &ID : IncludeDirectives) {
  92 |     const StringRef FileName = llvm::sys::path::filename(ID.FileName);
  93 |     if (fileNameIsRestricted(FileName))
  94 |       Check.diag(ID.Loc,
  95 |                  "including '%0' may cause additional compilation errors due "
  96 |                  "to the name of the kernel source file; consider renaming the "
  97 |                  "included kernel source file")
  98 |           << FileName;
  99 |   }
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 100-102
```cpp
 100 | }
 101 | 
 102 | } // namespace clang::tidy::altera
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `KernelNameRestrictionCheck.h`, `clang/Frontend/CompilerInstance.h`, `clang/Lex/PPCallbacks.h`, `clang/Lex/Preprocessor.h`, `vector`.
- CN: 直接包含依赖: `KernelNameRestrictionCheck.h`、`clang/Frontend/CompilerInstance.h`、`clang/Lex/PPCallbacks.h`、`clang/Lex/Preprocessor.h`、`vector`。
- EN: Framework base types: `PPCallbacks`.
- CN: 框架基类: `PPCallbacks`。
- EN: Namespace context: `clang::tidy::altera`.
- CN: 命名空间上下文: `clang::tidy::altera`。
