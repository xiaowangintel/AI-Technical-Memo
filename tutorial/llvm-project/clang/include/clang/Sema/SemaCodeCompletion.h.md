# SemaCodeCompletion.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/SemaCodeCompletion.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares facilities that support code completion.
- 用途（中文）: 该文件为 Sema 子系统中的 Sema Code Completion 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21

```cpp
 1: //===----- SemaCodeCompletion.h ------ Code completion support ------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: /// \file
 9: /// This file declares facilities that support code completion.
10: ///
11: //===----------------------------------------------------------------------===//
12: 
13: #ifndef LLVM_CLANG_SEMA_SEMACODECOMPLETION_H
14: #define LLVM_CLANG_SEMA_SEMACODECOMPLETION_H
15: 
16: #include "clang/AST/ASTFwd.h"
17: #include "clang/AST/Type.h"
18: #include "clang/Basic/AttributeCommonInfo.h"
19: #include "clang/Basic/IdentifierTable.h"
20: #include "clang/Basic/LLVM.h"
21: #include "clang/Basic/SourceLocation.h"
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/ASTFwd.h`, `clang/AST/Type.h`, `clang/Basic/AttributeCommonInfo.h` and 3 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/ASTFwd.h`, `clang/AST/Type.h`, `clang/Basic/AttributeCommonInfo.h` 以及另外 3 项依赖。

### Lines 22-42

```cpp
22: #include "clang/Lex/ModuleLoader.h"
23: #include "clang/Sema/CodeCompleteConsumer.h"
24: #include "clang/Sema/DeclSpec.h"
25: #include "clang/Sema/Designator.h"
26: #include "clang/Sema/HeuristicResolver.h"
27: #include "clang/Sema/Ownership.h"
28: #include "clang/Sema/SemaBase.h"
29: #include "llvm/ADT/StringRef.h"
30: #include <optional>
31: 
32: namespace clang {
33: class DeclGroupRef;
34: class MacroInfo;
35: class Scope;
36: class TemplateName;
37: 
38: class SemaCodeCompletion : public SemaBase {
39: public:
40:   SemaCodeCompletion(Sema &S, CodeCompleteConsumer *CompletionConsumer);
41: 
42:   using TemplateTy = OpaquePtr<TemplateName>;
```
- EN: This block imports dependencies such as `clang/Lex/ModuleLoader.h`, `clang/Sema/CodeCompleteConsumer.h`, `clang/Sema/DeclSpec.h` and 6 more. It opens, closes, or documents namespace scope for `clang`. Key type declarations here include `DeclGroupRef`, `MacroInfo`, `Scope`, `TemplateName`.
- 中文: 这一块引入了 `clang/Lex/ModuleLoader.h`, `clang/Sema/CodeCompleteConsumer.h`, `clang/Sema/DeclSpec.h` 以及另外 6 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。 这里的重要类型声明包括 `DeclGroupRef`, `MacroInfo`, `Scope`, `TemplateName`。

### Lines 43-63

```cpp
43:   using DeclGroupPtrTy = OpaquePtr<DeclGroupRef>;
44: 
45:   /// Code-completion consumer.
46:   CodeCompleteConsumer *CodeCompleter;
47:   HeuristicResolver Resolver;
48: 
49:   /// Describes the context in which code completion occurs.
50:   enum ParserCompletionContext {
51:     /// Code completion occurs at top-level or namespace context.
52:     PCC_Namespace,
53:     /// Code completion occurs within a class, struct, or union.
54:     PCC_Class,
55:     /// Code completion occurs within an Objective-C interface, protocol,
56:     /// or category.
57:     PCC_ObjCInterface,
58:     /// Code completion occurs within an Objective-C implementation or
59:     /// category implementation
60:     PCC_ObjCImplementation,
61:     /// Code completion occurs within the list of instance variables
62:     /// in an Objective-C interface, protocol, category, or implementation.
63:     PCC_ObjCInstanceVariableList,
```
- EN: It introduces enum-based state or option sets such as `ParserCompletionContext`. It defines convenient aliases such as `DeclGroupPtrTy`.
- 中文: 它引入了 `ParserCompletionContext` 等基于枚举的状态或选项集合。 它定义了 `DeclGroupPtrTy` 等便捷别名。

### Lines 64-84

```cpp
64:     /// Code completion occurs following one or more template
65:     /// headers.
66:     PCC_Template,
67:     /// Code completion occurs following one or more template
68:     /// headers within a class.
69:     PCC_MemberTemplate,
70:     /// Code completion occurs within an expression.
71:     PCC_Expression,
72:     /// Code completion occurs within a statement, which may
73:     /// also be an expression or a declaration.
74:     PCC_Statement,
75:     /// Code completion occurs at the beginning of the
76:     /// initialization statement (or expression) in a for loop.
77:     PCC_ForInit,
78:     /// Code completion occurs within the condition of an if,
79:     /// while, switch, or for statement.
80:     PCC_Condition,
81:     /// Code completion occurs within the body of a function on a
82:     /// recovery path, where we do not have a specific handle on our position
83:     /// in the grammar.
84:     PCC_RecoveryInFunction,
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 85-105

```cpp
 85:     /// Code completion occurs where only a type is permitted.
 86:     PCC_Type,
 87:     /// Code completion occurs in a parenthesized expression, which
 88:     /// might also be a type cast.
 89:     PCC_ParenthesizedExpression,
 90:     /// Code completion occurs within a sequence of declaration
 91:     /// specifiers within a function, method, or block.
 92:     PCC_LocalDeclarationSpecifiers,
 93:     /// Code completion occurs at top-level in a REPL session
 94:     PCC_TopLevelOrExpression,
 95:   };
 96: 
 97:   void CodeCompleteModuleImport(SourceLocation ImportLoc, ModuleIdPath Path);
 98:   void CodeCompleteOrdinaryName(Scope *S,
 99:                                 ParserCompletionContext CompletionContext);
100:   void CodeCompleteDeclSpec(Scope *S, DeclSpec &DS, bool AllowNonIdentifiers,
101:                             bool AllowNestedNameSpecifiers);
102: 
103:   struct CodeCompleteExpressionData;
104:   void CodeCompleteExpression(Scope *S, const CodeCompleteExpressionData &Data,
105:                               bool IsAddressOfOperand = false);
```
- EN: Key type declarations here include `CodeCompleteExpressionData`. It exposes API surface such as `CodeCompleteModuleImport`.
- 中文: 这里的重要类型声明包括 `CodeCompleteExpressionData`。 它暴露了 `CodeCompleteModuleImport` 等接口。

### Lines 106-126

```cpp
106:   void CodeCompleteExpression(Scope *S, QualType PreferredType,
107:                               bool IsParenthesized = false,
108:                               bool IsAddressOfOperand = false);
109:   void CodeCompleteMemberReferenceExpr(Scope *S, Expr *Base, Expr *OtherOpBase,
110:                                        SourceLocation OpLoc, bool IsArrow,
111:                                        bool IsBaseExprStatement,
112:                                        QualType PreferredType);
113:   void CodeCompletePostfixExpression(Scope *S, ExprResult LHS,
114:                                      QualType PreferredType);
115:   void CodeCompleteTag(Scope *S, unsigned TagSpec);
116:   void CodeCompleteTypeQualifiers(DeclSpec &DS);
117:   void CodeCompleteFunctionQualifiers(DeclSpec &DS, Declarator &D,
118:                                       const VirtSpecifiers *VS = nullptr);
119:   void CodeCompleteBracketDeclarator(Scope *S);
120:   void CodeCompleteCase(Scope *S);
121:   enum class AttributeCompletion {
122:     Attribute,
123:     Scope,
124:     None,
125:   };
126:   void CodeCompleteAttribute(
```
- EN: Key type declarations here include `AttributeCompletion`. It introduces enum-based state or option sets such as `AttributeCompletion`. It exposes API surface such as `CodeCompleteTag`, `CodeCompleteTypeQualifiers`, `CodeCompleteBracketDeclarator`, `CodeCompleteCase`.
- 中文: 这里的重要类型声明包括 `AttributeCompletion`。 它引入了 `AttributeCompletion` 等基于枚举的状态或选项集合。 它暴露了 `CodeCompleteTag`, `CodeCompleteTypeQualifiers`, `CodeCompleteBracketDeclarator`, `CodeCompleteCase` 等接口。

### Lines 127-147

```cpp
127:       AttributeCommonInfo::Syntax Syntax,
128:       AttributeCompletion Completion = AttributeCompletion::Attribute,
129:       const IdentifierInfo *Scope = nullptr);
130:   /// Determines the preferred type of the current function argument, by
131:   /// examining the signatures of all possible overloads.
132:   /// Returns null if unknown or ambiguous, or if code completion is off.
133:   ///
134:   /// If the code completion point has been reached, also reports the function
135:   /// signatures that were considered.
136:   ///
137:   /// FIXME: rename to GuessCallArgumentType to reduce confusion.
138:   QualType ProduceCallSignatureHelp(Expr *Fn, ArrayRef<Expr *> Args,
139:                                     SourceLocation OpenParLoc);
140:   QualType ProduceConstructorSignatureHelp(QualType Type, SourceLocation Loc,
141:                                            ArrayRef<Expr *> Args,
142:                                            SourceLocation OpenParLoc,
143:                                            bool Braced);
144:   QualType ProduceCtorInitMemberSignatureHelp(
145:       Decl *ConstructorDecl, CXXScopeSpec SS, ParsedType TemplateTypeTy,
146:       ArrayRef<Expr *> ArgExprs, IdentifierInfo *II, SourceLocation OpenParLoc,
147:       bool Braced);
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 148-168

```cpp
148:   QualType ProduceTemplateArgumentSignatureHelp(
149:       TemplateTy, ArrayRef<ParsedTemplateArgument>, SourceLocation LAngleLoc);
150:   void CodeCompleteInitializer(Scope *S, Decl *D);
151:   /// Trigger code completion for a record of \p BaseType. \p InitExprs are
152:   /// expressions in the initializer list seen so far and \p D is the current
153:   /// Designation being parsed.
154:   void CodeCompleteDesignator(const QualType BaseType,
155:                               llvm::ArrayRef<Expr *> InitExprs,
156:                               const Designation &D);
157:   void CodeCompleteKeywordAfterIf(bool AfterExclaim) const;
158:   void CodeCompleteAfterIf(Scope *S, bool IsBracedThen);
159: 
160:   void CodeCompleteQualifiedId(Scope *S, CXXScopeSpec &SS, bool EnteringContext,
161:                                bool IsUsingDeclaration, bool IsAddressOfOperand,
162:                                bool IsInDeclarationContext, QualType BaseType,
163:                                QualType PreferredType);
164:   void CodeCompleteUsing(Scope *S);
165:   void CodeCompleteUsingDirective(Scope *S);
166:   void CodeCompleteNamespaceDecl(Scope *S);
167:   void CodeCompleteNamespaceAliasDecl(Scope *S);
168:   void CodeCompleteOperatorName(Scope *S);
```
- EN: It exposes API surface such as `CodeCompleteInitializer`, `CodeCompleteKeywordAfterIf`, `CodeCompleteAfterIf`, `CodeCompleteUsing`.
- 中文: 它暴露了 `CodeCompleteInitializer`, `CodeCompleteKeywordAfterIf`, `CodeCompleteAfterIf`, `CodeCompleteUsing` 等接口。

### Lines 169-189

```cpp
169:   void CodeCompleteConstructorInitializer(
170:       Decl *Constructor, ArrayRef<CXXCtorInitializer *> Initializers);
171: 
172:   void CodeCompleteLambdaIntroducer(Scope *S, LambdaIntroducer &Intro,
173:                                     bool AfterAmpersand);
174:   void CodeCompleteAfterFunctionEquals(Declarator &D);
175: 
176:   void CodeCompleteObjCAtDirective(Scope *S);
177:   void CodeCompleteObjCAtVisibility(Scope *S);
178:   void CodeCompleteObjCAtStatement(Scope *S);
179:   void CodeCompleteObjCAtExpression(Scope *S);
180:   void CodeCompleteObjCPropertyFlags(Scope *S, ObjCDeclSpec &ODS);
181:   void CodeCompleteObjCPropertyGetter(Scope *S);
182:   void CodeCompleteObjCPropertySetter(Scope *S);
183:   void CodeCompleteObjCPassingType(Scope *S, ObjCDeclSpec &DS,
184:                                    bool IsParameter);
185:   void CodeCompleteObjCMessageReceiver(Scope *S);
186:   void CodeCompleteObjCSuperMessage(Scope *S, SourceLocation SuperLoc,
187:                                     ArrayRef<const IdentifierInfo *> SelIdents,
188:                                     bool AtArgumentExpression);
189:   void CodeCompleteObjCClassMessage(Scope *S, ParsedType Receiver,
```
- EN: It exposes API surface such as `CodeCompleteAfterFunctionEquals`, `CodeCompleteObjCAtDirective`, `CodeCompleteObjCAtVisibility`, `CodeCompleteObjCAtStatement`.
- 中文: 它暴露了 `CodeCompleteAfterFunctionEquals`, `CodeCompleteObjCAtDirective`, `CodeCompleteObjCAtVisibility`, `CodeCompleteObjCAtStatement` 等接口。

### Lines 190-210

```cpp
190:                                     ArrayRef<const IdentifierInfo *> SelIdents,
191:                                     bool AtArgumentExpression,
192:                                     bool IsSuper = false);
193:   void CodeCompleteObjCInstanceMessage(
194:       Scope *S, Expr *Receiver, ArrayRef<const IdentifierInfo *> SelIdents,
195:       bool AtArgumentExpression, ObjCInterfaceDecl *Super = nullptr);
196:   void CodeCompleteObjCForCollection(Scope *S, DeclGroupPtrTy IterationVar);
197:   void CodeCompleteObjCSelector(Scope *S,
198:                                 ArrayRef<const IdentifierInfo *> SelIdents);
199:   void CodeCompleteObjCProtocolReferences(ArrayRef<IdentifierLoc> Protocols);
200:   void CodeCompleteObjCProtocolDecl(Scope *S);
201:   void CodeCompleteObjCInterfaceDecl(Scope *S);
202:   void CodeCompleteObjCClassForwardDecl(Scope *S);
203:   void CodeCompleteObjCSuperclass(Scope *S, IdentifierInfo *ClassName,
204:                                   SourceLocation ClassNameLoc);
205:   void CodeCompleteObjCImplementationDecl(Scope *S);
206:   void CodeCompleteObjCInterfaceCategory(Scope *S, IdentifierInfo *ClassName,
207:                                          SourceLocation ClassNameLoc);
208:   void CodeCompleteObjCImplementationCategory(Scope *S,
209:                                               IdentifierInfo *ClassName,
210:                                               SourceLocation ClassNameLoc);
```
- EN: It exposes API surface such as `CodeCompleteObjCForCollection`, `CodeCompleteObjCProtocolReferences`, `CodeCompleteObjCProtocolDecl`, `CodeCompleteObjCInterfaceDecl`.
- 中文: 它暴露了 `CodeCompleteObjCForCollection`, `CodeCompleteObjCProtocolReferences`, `CodeCompleteObjCProtocolDecl`, `CodeCompleteObjCInterfaceDecl` 等接口。

### Lines 211-231

```cpp
211:   void CodeCompleteObjCPropertyDefinition(Scope *S);
212:   void CodeCompleteObjCPropertySynthesizeIvar(Scope *S,
213:                                               IdentifierInfo *PropertyName);
214:   void CodeCompleteObjCMethodDecl(Scope *S,
215:                                   std::optional<bool> IsInstanceMethod,
216:                                   ParsedType ReturnType);
217:   void CodeCompleteObjCMethodDeclSelector(
218:       Scope *S, bool IsInstanceMethod, bool AtParameterName,
219:       ParsedType ReturnType, ArrayRef<const IdentifierInfo *> SelIdents);
220:   void CodeCompleteObjCClassPropertyRefExpr(Scope *S,
221:                                             const IdentifierInfo &ClassName,
222:                                             SourceLocation ClassNameLoc,
223:                                             bool IsBaseExprStatement);
224:   void CodeCompletePreprocessorDirective(bool InConditional);
225:   void CodeCompleteInPreprocessorConditionalExclusion(Scope *S);
226:   void CodeCompletePreprocessorMacroName(bool IsDefinition);
227:   void CodeCompletePreprocessorExpression();
228:   void CodeCompletePreprocessorMacroArgument(Scope *S, IdentifierInfo *Macro,
229:                                              MacroInfo *MacroInfo,
230:                                              unsigned Argument);
231:   void CodeCompleteIncludedFile(llvm::StringRef Dir, bool IsAngled);
```
- EN: It exposes API surface such as `CodeCompleteObjCPropertyDefinition`, `CodeCompletePreprocessorDirective`, `CodeCompleteInPreprocessorConditionalExclusion`, `CodeCompletePreprocessorMacroName`.
- 中文: 它暴露了 `CodeCompleteObjCPropertyDefinition`, `CodeCompletePreprocessorDirective`, `CodeCompleteInPreprocessorConditionalExclusion`, `CodeCompletePreprocessorMacroName` 等接口。

### Lines 232-242

```cpp
232:   void CodeCompleteNaturalLanguage();
233:   void CodeCompleteAvailabilityPlatformName();
234:   void
235:   GatherGlobalCodeCompletions(CodeCompletionAllocator &Allocator,
236:                               CodeCompletionTUInfo &CCTUInfo,
237:                               SmallVectorImpl<CodeCompletionResult> &Results);
238: };
239: 
240: } // namespace clang
241: 
242: #endif // LLVM_CLANG_SEMA_SEMACODECOMPLETION_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. It exposes API surface such as `CodeCompleteNaturalLanguage`, `CodeCompleteAvailabilityPlatformName`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 它暴露了 `CodeCompleteNaturalLanguage`, `CodeCompleteAvailabilityPlatformName` 等接口。

## Key Concepts / 关键概念

- `DeclGroupRef`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `MacroInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Scope`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `TemplateName`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `SemaCodeCompletion`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `TemplateTy`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `DeclGroupPtrTy`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `ParserCompletionContext`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/ASTFwd.h`, `clang/AST/Type.h`, `clang/Basic/AttributeCommonInfo.h`, `clang/Basic/IdentifierTable.h`, `clang/Basic/LLVM.h`, `clang/Basic/SourceLocation.h`, `clang/Lex/ModuleLoader.h`, `clang/Sema/CodeCompleteConsumer.h`, `clang/Sema/DeclSpec.h`, `clang/Sema/Designator.h`, `clang/Sema/HeuristicResolver.h`, `clang/Sema/Ownership.h`, `clang/Sema/SemaBase.h`, `llvm/ADT/StringRef.h`, `optional`
- Forward declarations / 前向声明: `DeclGroupRef`, `MacroInfo`, `Scope`, `TemplateName`, `CodeCompleteExpressionData`
- Namespace context / 命名空间上下文: `clang`
- Macro-style dependencies / 宏式依赖: None / 无
