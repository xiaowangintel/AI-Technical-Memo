# IdDependentBackwardBranchCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/altera/IdDependentBackwardBranchCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `IdDependentBackwardBranchCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `IdDependentBackwardBranchCheck`。

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

### Lines 9-12
```cpp
   9 | #include "IdDependentBackwardBranchCheck.h"
  10 | #include "clang/AST/ASTContext.h"
  11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  12 | 
```
- EN: The section imports dependencies such as `IdDependentBackwardBranchCheck.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `IdDependentBackwardBranchCheck.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 13-16
```cpp
  13 | using namespace clang::ast_matchers;
  14 | 
  15 | namespace clang::tidy::altera {
  16 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::altera` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::altera` 这样的命名空间将符号放入预期的子系统中。

### Lines 17-22
```cpp
  17 | void IdDependentBackwardBranchCheck::registerMatchers(MatchFinder *Finder) {
  18 |   // Prototype to identify all variables which hold a thread-variant ID.
  19 |   // First Matcher just finds all the direct assignments of either ID call.
  20 |   const auto ThreadID = expr(hasDescendant(callExpr(callee(functionDecl(
  21 |       anyOf(hasName("get_global_id"), hasName("get_local_id")))))));
  22 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `IdDependentBackwardBranchCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `IdDependentBackwardBranchCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 23-26
```cpp
  23 |   const auto RefVarOrField = forEachDescendant(
  24 |       stmt(anyOf(declRefExpr(to(varDecl())).bind("assign_ref_var"),
  25 |                  memberExpr(member(fieldDecl())).bind("assign_ref_field"))));
  26 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto RefVarOrField = forEachDescendant(`.
- CN: 这一段继续实现，围绕 `const auto RefVarOrField = forEachDescendant(` 展开声明或语句。

### Lines 27-36
```cpp
  27 |   Finder->addMatcher(
  28 |       compoundStmt(
  29 |           // Bind on actual get_local/global_id calls.
  30 |           forEachDescendant(
  31 |               stmt(
  32 |                   anyOf(declStmt(hasDescendant(varDecl(hasInitializer(ThreadID))
  33 |                                                    .bind("tid_dep_var"))),
  34 |                         binaryOperator(
  35 |                             isAssignmentOperator(), hasRHS(ThreadID),
  36 |                             hasLHS(anyOf(
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(` 展开声明或语句。

### Lines 37-42
```cpp
  37 |                                 declRefExpr(to(varDecl().bind("tid_dep_var"))),
  38 |                                 memberExpr(member(
  39 |                                     fieldDecl().bind("tid_dep_field"))))))))
  40 |                   .bind("straight_assignment"))),
  41 |       this);
  42 | 
```
- EN: This block continues the implementation with declarations or statements centered on `declRefExpr(to(varDecl().bind("tid_dep_var"))),`.
- CN: 这一段继续实现，围绕 `declRefExpr(to(varDecl().bind("tid_dep_var"))),` 展开声明或语句。

### Lines 43-49
```cpp
  43 |   // Bind all VarDecls that include an initializer with a variable DeclRefExpr
  44 |   // (in case it is ID-dependent).
  45 |   Finder->addMatcher(
  46 |       stmt(forEachDescendant(
  47 |           varDecl(hasInitializer(RefVarOrField)).bind("pot_tid_var"))),
  48 |       this);
  49 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Bind all VarDecls that include an initializer with a vari`.
- CN: 这一段继续实现，围绕 `// Bind all VarDecls that include an initializer with a vari` 展开声明或语句。

### Lines 50-59
```cpp
  50 |   // Bind all VarDecls that are assigned a value with a variable DeclRefExpr (in
  51 |   // case it is ID-dependent).
  52 |   Finder->addMatcher(
  53 |       stmt(forEachDescendant(binaryOperator(
  54 |           allOf(isAssignmentOperator(), hasRHS(RefVarOrField),
  55 |                 hasLHS(anyOf(
  56 |                     declRefExpr(to(varDecl().bind("pot_tid_var"))),
  57 |                     memberExpr(member(fieldDecl().bind("pot_tid_field"))))))))),
  58 |       this);
  59 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Bind all VarDecls that are assigned a value with a variab`.
- CN: 这一段继续实现，围绕 `// Bind all VarDecls that are assigned a value with a variab` 展开声明或语句。

### Lines 60-69
```cpp
  60 |   // Second Matcher looks for branch statements inside of loops and bind on the
  61 |   // condition expression IF it either calls an ID function or has a variable
  62 |   // DeclRefExpr. DeclRefExprs are checked later to confirm whether the variable
  63 |   // is ID-dependent.
  64 |   const auto CondExpr =
  65 |       expr(anyOf(hasDescendant(callExpr(callee(functionDecl(
  66 |                                             anyOf(hasName("get_global_id"),
  67 |                                                   hasName("get_local_id")))))
  68 |                                    .bind("id_call")),
  69 |                  hasDescendant(stmt(anyOf(declRefExpr(to(varDecl())),
```
- EN: This block continues the implementation with declarations or statements centered on `// Second Matcher looks for branch statements inside of loop`.
- CN: 这一段继续实现，围绕 `// Second Matcher looks for branch statements inside of loop` 展开声明或语句。

### Lines 70-78
```cpp
  70 |                                           memberExpr(member(fieldDecl())))))))
  71 |           .bind("cond_expr");
  72 |   Finder->addMatcher(stmt(anyOf(forStmt(hasCondition(CondExpr)),
  73 |                                 doStmt(hasCondition(CondExpr)),
  74 |                                 whileStmt(hasCondition(CondExpr))))
  75 |                          .bind("backward_branch"),
  76 |                      this);
  77 | }
  78 | 
```
- EN: This block continues the implementation with declarations or statements centered on `memberExpr(member(fieldDecl())))))))`.
- CN: 这一段继续实现，围绕 `memberExpr(member(fieldDecl())))))))` 展开声明或语句。

### Lines 79-83
```cpp
  79 | const IdDependentBackwardBranchCheck::IdDependencyRecord *
  80 | IdDependentBackwardBranchCheck::hasIdDepVar(const Expr *Expression) {
  81 |   if (!Expression)
  82 |     return nullptr;
  83 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 84-93
```cpp
  84 |   if (const auto *Declaration = dyn_cast<DeclRefExpr>(Expression)) {
  85 |     // It is a DeclRefExpr, so check if it's an ID-dependent variable.
  86 |     const auto *CheckVariable =
  87 |         dyn_cast_if_present<VarDecl>(Declaration->getDecl());
  88 |     if (!CheckVariable)
  89 |       return nullptr;
  90 |     auto FoundVariable = IdDepVarsMap.find(CheckVariable);
  91 |     if (FoundVariable == IdDepVarsMap.end())
  92 |       return nullptr;
  93 |     return &(FoundVariable->second);
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 94-101
```cpp
  94 |   }
  95 |   for (const auto *Child : Expression->children())
  96 |     if (const auto *ChildExpression = dyn_cast_if_present<Expr>(Child))
  97 |       if (const IdDependencyRecord *Result = hasIdDepVar(ChildExpression))
  98 |         return Result;
  99 |   return nullptr;
 100 | }
 101 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 102-106
```cpp
 102 | const IdDependentBackwardBranchCheck::IdDependencyRecord *
 103 | IdDependentBackwardBranchCheck::hasIdDepField(const Expr *Expression) {
 104 |   if (!Expression)
 105 |     return nullptr;
 106 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 107-116
```cpp
 107 |   if (const auto *MemberExpression = dyn_cast<MemberExpr>(Expression)) {
 108 |     const auto *CheckField =
 109 |         dyn_cast_if_present<FieldDecl>(MemberExpression->getMemberDecl());
 110 |     if (!CheckField)
 111 |       return nullptr;
 112 |     auto FoundField = IdDepFieldsMap.find(CheckField);
 113 |     if (FoundField == IdDepFieldsMap.end())
 114 |       return nullptr;
 115 |     return &(FoundField->second);
 116 |   }
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 117-123
```cpp
 117 |   for (const auto *Child : Expression->children())
 118 |     if (const auto *ChildExpression = dyn_cast_if_present<Expr>(Child))
 119 |       if (const IdDependencyRecord *Result = hasIdDepField(ChildExpression))
 120 |         return Result;
 121 |   return nullptr;
 122 | }
 123 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 124-132
```cpp
 124 | void IdDependentBackwardBranchCheck::saveIdDepVar(const Stmt *Statement,
 125 |                                                   const VarDecl *Variable) {
 126 |   // Record that this variable is thread-dependent.
 127 |   IdDepVarsMap[Variable] =
 128 |       IdDependencyRecord(Variable, Variable->getBeginLoc(),
 129 |                          Twine("assignment of ID-dependent variable ") +
 130 |                              Variable->getNameAsString());
 131 | }
 132 | 
```
- EN: Method definitions such as `IdDependentBackwardBranchCheck::saveIdDepVar` provide the concrete behavior declared elsewhere.
- CN: 诸如 `IdDependentBackwardBranchCheck::saveIdDepVar` 的方法定义给出了前面声明的具体行为。

### Lines 133-140
```cpp
 133 | void IdDependentBackwardBranchCheck::saveIdDepField(const Stmt *Statement,
 134 |                                                     const FieldDecl *Field) {
 135 |   // Record that this field is thread-dependent.
 136 |   IdDepFieldsMap[Field] = IdDependencyRecord(
 137 |       Field, Statement->getBeginLoc(),
 138 |       Twine("assignment of ID-dependent field ") + Field->getNameAsString());
 139 | }
 140 | 
```
- EN: Method definitions such as `IdDependentBackwardBranchCheck::saveIdDepField` provide the concrete behavior declared elsewhere.
- CN: 诸如 `IdDependentBackwardBranchCheck::saveIdDepField` 的方法定义给出了前面声明的具体行为。

### Lines 141-150
```cpp
 141 | void IdDependentBackwardBranchCheck::saveIdDepVarFromReference(
 142 |     const DeclRefExpr *RefExpr, const MemberExpr *MemExpr,
 143 |     const VarDecl *PotentialVar) {
 144 |   // If the variable is already in IdDepVarsMap, ignore it.
 145 |   if (IdDepVarsMap.contains(PotentialVar))
 146 |     return;
 147 |   std::string Message;
 148 |   llvm::raw_string_ostream StringStream(Message);
 149 |   StringStream << "inferred assignment of ID-dependent value from "
 150 |                   "ID-dependent ";
```
- EN: Method definitions such as `IdDependentBackwardBranchCheck::saveIdDepVarFromReference` provide the concrete behavior declared elsewhere.
- CN: 诸如 `IdDependentBackwardBranchCheck::saveIdDepVarFromReference` 的方法定义给出了前面声明的具体行为。

### Lines 151-160
```cpp
 151 |   if (RefExpr) {
 152 |     const auto *RefVar = dyn_cast<VarDecl>(RefExpr->getDecl());
 153 |     // If variable isn't ID-dependent, but RefVar is.
 154 |     if (IdDepVarsMap.contains(RefVar))
 155 |       StringStream << "variable " << RefVar->getNameAsString();
 156 |   }
 157 |   if (MemExpr) {
 158 |     const auto *RefField = dyn_cast<FieldDecl>(MemExpr->getMemberDecl());
 159 |     // If variable isn't ID-dependent, but RefField is.
 160 |     if (IdDepFieldsMap.contains(RefField))
```
- EN: This block continues the implementation with declarations or statements centered on `if (RefExpr) {`.
- CN: 这一段继续实现，围绕 `if (RefExpr) {` 展开声明或语句。

### Lines 161-166
```cpp
 161 |       StringStream << "member " << RefField->getNameAsString();
 162 |   }
 163 |   IdDepVarsMap[PotentialVar] =
 164 |       IdDependencyRecord(PotentialVar, PotentialVar->getBeginLoc(), Message);
 165 | }
 166 | 
```
- EN: This block continues the implementation with declarations or statements centered on `StringStream << "member " << RefField->getNameAsString();`.
- CN: 这一段继续实现，围绕 `StringStream << "member " << RefField->getNameAsString();` 展开声明或语句。

### Lines 167-176
```cpp
 167 | void IdDependentBackwardBranchCheck::saveIdDepFieldFromReference(
 168 |     const DeclRefExpr *RefExpr, const MemberExpr *MemExpr,
 169 |     const FieldDecl *PotentialField) {
 170 |   // If the field is already in IdDepFieldsMap, ignore it.
 171 |   if (IdDepFieldsMap.contains(PotentialField))
 172 |     return;
 173 |   std::string Message;
 174 |   llvm::raw_string_ostream StringStream(Message);
 175 |   StringStream << "inferred assignment of ID-dependent member from "
 176 |                   "ID-dependent ";
```
- EN: Method definitions such as `IdDependentBackwardBranchCheck::saveIdDepFieldFromReference` provide the concrete behavior declared elsewhere.
- CN: 诸如 `IdDependentBackwardBranchCheck::saveIdDepFieldFromReference` 的方法定义给出了前面声明的具体行为。

### Lines 177-186
```cpp
 177 |   if (RefExpr) {
 178 |     const auto *RefVar = dyn_cast<VarDecl>(RefExpr->getDecl());
 179 |     // If field isn't ID-dependent, but RefVar is.
 180 |     if (IdDepVarsMap.contains(RefVar))
 181 |       StringStream << "variable " << RefVar->getNameAsString();
 182 |   }
 183 |   if (MemExpr) {
 184 |     const auto *RefField = dyn_cast<FieldDecl>(MemExpr->getMemberDecl());
 185 |     if (IdDepFieldsMap.contains(RefField))
 186 |       StringStream << "member " << RefField->getNameAsString();
```
- EN: This block continues the implementation with declarations or statements centered on `if (RefExpr) {`.
- CN: 这一段继续实现，围绕 `if (RefExpr) {` 展开声明或语句。

### Lines 187-191
```cpp
 187 |   }
 188 |   IdDepFieldsMap[PotentialField] = IdDependencyRecord(
 189 |       PotentialField, PotentialField->getBeginLoc(), Message);
 190 | }
 191 | 
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 192-201
```cpp
 192 | IdDependentBackwardBranchCheck::LoopType
 193 | IdDependentBackwardBranchCheck::getLoopType(const Stmt *Loop) {
 194 |   switch (Loop->getStmtClass()) {
 195 |   case Stmt::DoStmtClass:
 196 |     return DoLoop;
 197 |   case Stmt::WhileStmtClass:
 198 |     return WhileLoop;
 199 |   case Stmt::ForStmtClass:
 200 |     return ForLoop;
 201 |   default:
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 202-205
```cpp
 202 |     return UnknownLoop;
 203 |   }
 204 | }
 205 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 206-215
```cpp
 206 | void IdDependentBackwardBranchCheck::check(
 207 |     const MatchFinder::MatchResult &Result) {
 208 |   // The first half of the callback only deals with identifying and storing
 209 |   // ID-dependency information into the IdDepVars and IdDepFields maps.
 210 |   const auto *Variable = Result.Nodes.getNodeAs<VarDecl>("tid_dep_var");
 211 |   const auto *Field = Result.Nodes.getNodeAs<FieldDecl>("tid_dep_field");
 212 |   const auto *Statement = Result.Nodes.getNodeAs<Stmt>("straight_assignment");
 213 |   const auto *RefExpr = Result.Nodes.getNodeAs<DeclRefExpr>("assign_ref_var");
 214 |   const auto *MemExpr = Result.Nodes.getNodeAs<MemberExpr>("assign_ref_field");
 215 |   const auto *PotentialVar = Result.Nodes.getNodeAs<VarDecl>("pot_tid_var");
```
- EN: Method definitions such as `IdDependentBackwardBranchCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `IdDependentBackwardBranchCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 216-225
```cpp
 216 |   const auto *PotentialField =
 217 |       Result.Nodes.getNodeAs<FieldDecl>("pot_tid_field");
 218 | 
 219 |   // Save variables and fields assigned directly through ID function calls.
 220 |   if (Statement && (Variable || Field)) {
 221 |     if (Variable)
 222 |       saveIdDepVar(Statement, Variable);
 223 |     else if (Field)
 224 |       saveIdDepField(Statement, Field);
 225 |   }
```
- EN: This block continues the implementation with declarations or statements centered on `const auto *PotentialField =`.
- CN: 这一段继续实现，围绕 `const auto *PotentialField =` 展开声明或语句。

### Lines 226-230
```cpp
 226 | 
 227 |   // Save variables assigned to values of Id-dependent variables and fields.
 228 |   if ((RefExpr || MemExpr) && PotentialVar)
 229 |     saveIdDepVarFromReference(RefExpr, MemExpr, PotentialVar);
 230 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Save variables assigned to values of Id-dependent variabl`.
- CN: 这一段继续实现，围绕 `// Save variables assigned to values of Id-dependent variabl` 展开声明或语句。

### Lines 231-234
```cpp
 231 |   // Save fields assigned to values of ID-dependent variables and fields.
 232 |   if ((RefExpr || MemExpr) && PotentialField)
 233 |     saveIdDepFieldFromReference(RefExpr, MemExpr, PotentialField);
 234 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Save fields assigned to values of ID-dependent variables `.
- CN: 这一段继续实现，围绕 `// Save fields assigned to values of ID-dependent variables ` 展开声明或语句。

### Lines 235-244
```cpp
 235 |   // The second part of the callback deals with checking if a branch inside a
 236 |   // loop is thread dependent.
 237 |   const auto *CondExpr = Result.Nodes.getNodeAs<Expr>("cond_expr");
 238 |   const auto *IDCall = Result.Nodes.getNodeAs<CallExpr>("id_call");
 239 |   const auto *Loop = Result.Nodes.getNodeAs<Stmt>("backward_branch");
 240 |   if (!Loop)
 241 |     return;
 242 |   const LoopType Type = getLoopType(Loop);
 243 |   if (CondExpr) {
 244 |     if (IDCall) { // Conditional expression calls an ID function directly.
```
- EN: This block continues the implementation with declarations or statements centered on `// The second part of the callback deals with checking if a `.
- CN: 这一段继续实现，围绕 `// The second part of the callback deals with checking if a ` 展开声明或语句。

### Lines 245-254
```cpp
 245 |       diag(CondExpr->getBeginLoc(),
 246 |            "backward branch (%select{do|while|for}0 loop) is ID-dependent due "
 247 |            "to ID function call and may cause performance degradation")
 248 |           << Type;
 249 |       return;
 250 |     }
 251 |     // Conditional expression has DeclRefExpr(s), check ID-dependency.
 252 |     const IdDependencyRecord *IdDepVar = hasIdDepVar(CondExpr);
 253 |     const IdDependencyRecord *IdDepField = hasIdDepField(CondExpr);
 254 |     if (IdDepVar) {
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 255-264
```cpp
 255 |       diag(CondExpr->getBeginLoc(),
 256 |            "backward branch (%select{do|while|for}0 loop) is ID-dependent due "
 257 |            "to variable reference to %1 and may cause performance degradation")
 258 |           << Type << IdDepVar->VariableDeclaration;
 259 |       diag(IdDepVar->Location, IdDepVar->Message, DiagnosticIDs::Note);
 260 |     } else if (IdDepField) {
 261 |       diag(CondExpr->getBeginLoc(),
 262 |            "backward branch (%select{do|while|for}0 loop) is ID-dependent due "
 263 |            "to member reference to %1 and may cause performance degradation")
 264 |           << Type << IdDepField->FieldDeclaration;
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 265-270
```cpp
 265 |       diag(IdDepField->Location, IdDepField->Message, DiagnosticIDs::Note);
 266 |     }
 267 |   }
 268 | }
 269 | 
 270 | } // namespace clang::tidy::altera
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `IdDependentBackwardBranchCheck.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`.
- CN: 直接包含依赖: `IdDependentBackwardBranchCheck.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`。
- EN: Namespace context: `clang::tidy::altera`.
- CN: 命名空间上下文: `clang::tidy::altera`。
