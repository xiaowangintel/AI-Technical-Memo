# GCDAntipatternChecker.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/StaticAnalyzer/Checkers/GCDAntipatternChecker.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This file defines GCDAntipatternChecker which checks against a common antipattern when synchronous API is emulated from asynchronous callbacks using a semaphore:.
- **Purpose (CN)**: 实现或支撑 `GCDAntipatternChecker` 静态分析检查器的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
   1: //===- GCDAntipatternChecker.cpp ---------------------------------*- C++ -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines GCDAntipatternChecker which checks against a common
  10: // antipattern when synchronous API is emulated from asynchronous callbacks
  11: // using a semaphore:
  12: //
  13: //   dispatch_semaphore_t sema = dispatch_semaphore_create(0);
  14: //
  15: //   AnyCFunctionCall(^{
  16: //     // code…
  17: //     dispatch_semaphore_signal(sema);
  18: //   })
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 19-30
```cpp
  19: //   dispatch_semaphore_wait(sema, *)
  20: //
  21: // Such code is a common performance problem, due to inability of GCD to
  22: // properly handle QoS when a combination of queues and semaphores is used.
  23: // Good code would either use asynchronous API (when available), or perform
  24: // the necessary action in asynchronous callback.
  25: //
  26: // Currently, the check is performed using a simple heuristical AST pattern
  27: // matching.
  28: //
  29: //===----------------------------------------------------------------------===//
  30: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 31-38
```cpp
  31: #include "clang/StaticAnalyzer/Checkers/BuiltinCheckerRegistration.h"
  32: #include "clang/ASTMatchers/ASTMatchFinder.h"
  33: #include "clang/StaticAnalyzer/Core/BugReporter/BugReporter.h"
  34: #include "clang/StaticAnalyzer/Core/BugReporter/BugType.h"
  35: #include "clang/StaticAnalyzer/Core/Checker.h"
  36: #include "clang/StaticAnalyzer/Core/PathSensitive/AnalysisManager.h"
  37: #include "llvm/Support/Debug.h"
  38: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `BuiltinCheckerRegistration.h`, `ASTMatchFinder.h`, `BugReporter.h`, `BugType.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `BuiltinCheckerRegistration.h`, `ASTMatchFinder.h`, `BugReporter.h`, `BugType.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 39-47
```cpp
  39: using namespace clang;
  40: using namespace ento;
  41: using namespace ast_matchers;
  42: 
  43: namespace {
  44: 
  45: // ID of a node at which the diagnostic would be emitted.
  46: const char *WarnAtNode = "waitcall";
  47: 
```
- **EN**: This block organizes symbols with namespaces or using-declarations for shorter references.
- **CN**: 该代码块通过命名空间或 using 声明组织符号并简化后续引用。

### Lines 48-54
```cpp
  48: class GCDAntipatternChecker : public Checker<check::ASTCodeBody> {
  49: public:
  50:   void checkASTCodeBody(const Decl *D,
  51:                         AnalysisManager &AM,
  52:                         BugReporter &BR) const;
  53: };
  54: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `checkASTCodeBody`. It introduces or references types such as `GCDAntipatternChecker`.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `checkASTCodeBody`。 它引入或引用了诸如 `GCDAntipatternChecker` 等类型。

### Lines 55-58
```cpp
  55: decltype(auto) callsName(const char *FunctionName) {
  56:   return callee(functionDecl(hasName(FunctionName)));
  57: }
  58: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 59-63
```cpp
  59: decltype(auto) equalsBoundArgDecl(int ArgIdx, const char *DeclName) {
  60:   return hasArgument(ArgIdx, ignoringParenCasts(declRefExpr(
  61:                                  to(varDecl(equalsBoundNode(DeclName))))));
  62: }
  63: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 64-68
```cpp
  64: decltype(auto) bindAssignmentToDecl(const char *DeclName) {
  65:   return hasLHS(ignoringParenImpCasts(
  66:                          declRefExpr(to(varDecl().bind(DeclName)))));
  67: }
  68: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 69-91
```cpp
  69: /// The pattern is very common in tests, and it is OK to use it there.
  70: /// We have to heuristics for detecting tests: method name starts with "test"
  71: /// (used in XCTest), and a class name contains "mock" or "test" (used in
  72: /// helpers which are not tests themselves, but used exclusively in tests).
  73: static bool isTest(const Decl *D) {
  74:   if (const auto* ND = dyn_cast<NamedDecl>(D)) {
  75:     std::string DeclName = ND->getNameAsString();
  76:     if (StringRef(DeclName).starts_with("test"))
  77:       return true;
  78:   }
  79:   if (const auto *OD = dyn_cast<ObjCMethodDecl>(D)) {
  80:     if (const auto *CD = dyn_cast<ObjCContainerDecl>(OD->getParent())) {
  81:       std::string ContainerName = CD->getNameAsString();
  82:       StringRef CN(ContainerName);
  83:       if (CN.contains_insensitive("test") || CN.contains_insensitive("mock"))
  84:         return true;
  85:     }
  86:   }
  87:   return false;
  88: }
  89: 
  90: static auto findGCDAntiPatternWithSemaphore() -> decltype(compoundStmt()) {
  91: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `isTest`, `CN`, `findGCDAntiPatternWithSemaphore`. It introduces or references types such as `name`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `isTest`、`CN`、`findGCDAntiPatternWithSemaphore`。 它引入或引用了诸如 `name` 等类型。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 92-96
```cpp
  92:   const char *SemaphoreBinding = "semaphore_name";
  93:   auto SemaphoreCreateM = callExpr(allOf(
  94:       callsName("dispatch_semaphore_create"),
  95:       hasArgument(0, ignoringParenCasts(integerLiteral(equals(0))))));
  96: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `callsName`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `callsName`。

### Lines 97-102
```cpp
  97:   auto SemaphoreBindingM = anyOf(
  98:       forEachDescendant(
  99:           varDecl(hasDescendant(SemaphoreCreateM)).bind(SemaphoreBinding)),
 100:       forEachDescendant(binaryOperator(bindAssignmentToDecl(SemaphoreBinding),
 101:                      hasRHS(SemaphoreCreateM))));
 102: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `forEachDescendant`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `forEachDescendant`。

### Lines 103-106
```cpp
 103:   auto HasBlockArgumentM = hasAnyArgument(hasType(
 104:             hasCanonicalType(blockPointerType())
 105:             ));
 106: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `hasCanonicalType`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `hasCanonicalType`。

### Lines 107-114
```cpp
 107:   auto ArgCallsSignalM = hasAnyArgument(stmt(hasDescendant(callExpr(
 108:           allOf(
 109:               callsName("dispatch_semaphore_signal"),
 110:               equalsBoundArgDecl(0, SemaphoreBinding)
 111:               )))));
 112: 
 113:   auto HasBlockAndCallsSignalM = allOf(HasBlockArgumentM, ArgCallsSignalM);
 114: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `allOf`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `allOf`。

### Lines 115-121
```cpp
 115:   auto HasBlockCallingSignalM =
 116:     forEachDescendant(
 117:       stmt(anyOf(
 118:         callExpr(HasBlockAndCallsSignalM),
 119:         objcMessageExpr(HasBlockAndCallsSignalM)
 120:            )));
 121: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `forEachDescendant`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `forEachDescendant`。

### Lines 122-129
```cpp
 122:   auto SemaphoreWaitM = forEachDescendant(
 123:     callExpr(
 124:       allOf(
 125:         callsName("dispatch_semaphore_wait"),
 126:         equalsBoundArgDecl(0, SemaphoreBinding)
 127:       )
 128:     ).bind(WarnAtNode));
 129: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `callExpr`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `callExpr`。

### Lines 130-138
```cpp
 130:   return compoundStmt(
 131:       SemaphoreBindingM, HasBlockCallingSignalM, SemaphoreWaitM);
 132: }
 133: 
 134: static auto findGCDAntiPatternWithGroup() -> decltype(compoundStmt()) {
 135: 
 136:   const char *GroupBinding = "group_name";
 137:   auto DispatchGroupCreateM = callExpr(callsName("dispatch_group_create"));
 138: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `findGCDAntiPatternWithGroup`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `findGCDAntiPatternWithGroup`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 139-144
```cpp
 139:   auto GroupBindingM = anyOf(
 140:       forEachDescendant(
 141:           varDecl(hasDescendant(DispatchGroupCreateM)).bind(GroupBinding)),
 142:       forEachDescendant(binaryOperator(bindAssignmentToDecl(GroupBinding),
 143:                      hasRHS(DispatchGroupCreateM))));
 144: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `forEachDescendant`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `forEachDescendant`。

### Lines 145-148
```cpp
 145:   auto GroupEnterM = forEachDescendant(
 146:       stmt(callExpr(allOf(callsName("dispatch_group_enter"),
 147:                           equalsBoundArgDecl(0, GroupBinding)))));
 148: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `stmt`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `stmt`。

### Lines 149-152
```cpp
 149:   auto HasBlockArgumentM = hasAnyArgument(hasType(
 150:             hasCanonicalType(blockPointerType())
 151:             ));
 152: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `hasCanonicalType`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `hasCanonicalType`。

### Lines 153-160
```cpp
 153:   auto ArgCallsSignalM = hasAnyArgument(stmt(hasDescendant(callExpr(
 154:           allOf(
 155:               callsName("dispatch_group_leave"),
 156:               equalsBoundArgDecl(0, GroupBinding)
 157:               )))));
 158: 
 159:   auto HasBlockAndCallsLeaveM = allOf(HasBlockArgumentM, ArgCallsSignalM);
 160: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `allOf`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `allOf`。

### Lines 161-167
```cpp
 161:   auto AcceptsBlockM =
 162:     forEachDescendant(
 163:       stmt(anyOf(
 164:         callExpr(HasBlockAndCallsLeaveM),
 165:         objcMessageExpr(HasBlockAndCallsLeaveM)
 166:            )));
 167: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `forEachDescendant`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `forEachDescendant`。

### Lines 168-178
```cpp
 168:   auto GroupWaitM = forEachDescendant(
 169:     callExpr(
 170:       allOf(
 171:         callsName("dispatch_group_wait"),
 172:         equalsBoundArgDecl(0, GroupBinding)
 173:       )
 174:     ).bind(WarnAtNode));
 175: 
 176:   return compoundStmt(GroupBindingM, GroupEnterM, AcceptsBlockM, GroupWaitM);
 177: }
 178: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `callExpr`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `callExpr`。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 179-186
```cpp
 179: static void emitDiagnostics(const BoundNodes &Nodes,
 180:                             const char* Type,
 181:                             BugReporter &BR,
 182:                             AnalysisDeclContext *ADC,
 183:                             const GCDAntipatternChecker *Checker) {
 184:   const auto *SW = Nodes.getNodeAs<CallExpr>(WarnAtNode);
 185:   assert(SW);
 186: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `emitDiagnostics`, `assert`. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `emitDiagnostics`、`assert`。 断言用于说明实现期望始终成立的不变量。

### Lines 187-192
```cpp
 187:   std::string Diagnostics;
 188:   llvm::raw_string_ostream OS(Diagnostics);
 189:   OS << "Waiting on a callback using a " << Type << " creates useless threads "
 190:      << "and is subject to priority inversion; consider "
 191:      << "using a synchronous API or changing the caller to be asynchronous";
 192: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `OS`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `OS`。

### Lines 193-202
```cpp
 193:   BR.EmitBasicReport(
 194:     ADC->getDecl(),
 195:     Checker,
 196:     /*Name=*/"GCD performance anti-pattern",
 197:     /*BugCategory=*/"Performance",
 198:     OS.str(),
 199:     PathDiagnosticLocation::createBegin(SW, BR.getSourceManager(), ADC),
 200:     SW->getSourceRange());
 201: }
 202: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `PathDiagnosticLocation::createBegin`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `PathDiagnosticLocation::createBegin`。

### Lines 203-210
```cpp
 203: void GCDAntipatternChecker::checkASTCodeBody(const Decl *D,
 204:                                              AnalysisManager &AM,
 205:                                              BugReporter &BR) const {
 206:   if (isTest(D))
 207:     return;
 208: 
 209:   AnalysisDeclContext *ADC = AM.getAnalysisDeclContext(D);
 210: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `GCDAntipatternChecker::checkASTCodeBody`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `GCDAntipatternChecker::checkASTCodeBody`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 211-215
```cpp
 211:   auto SemaphoreMatcherM = findGCDAntiPatternWithSemaphore();
 212:   auto Matches = match(SemaphoreMatcherM, *D->getBody(), AM.getASTContext());
 213:   for (BoundNodes Match : Matches)
 214:     emitDiagnostics(Match, "semaphore", BR, ADC, this);
 215: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 216-223
```cpp
 216:   auto GroupMatcherM = findGCDAntiPatternWithGroup();
 217:   Matches = match(GroupMatcherM, *D->getBody(), AM.getASTContext());
 218:   for (BoundNodes Match : Matches)
 219:     emitDiagnostics(Match, "group", BR, ADC, this);
 220: }
 221: 
 222: } // end of anonymous namespace
 223: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 224-227
```cpp
 224: void ento::registerGCDAntipattern(CheckerManager &Mgr) {
 225:   Mgr.registerChecker<GCDAntipatternChecker>();
 226: }
 227: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ento::registerGCDAntipattern`. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ento::registerGCDAntipattern`。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 228-230
```cpp
 228: bool ento::shouldRegisterGCDAntipattern(const CheckerManager &mgr) {
 229:   return true;
 230: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ento::shouldRegisterGCDAntipattern`. Return statements hand back computed state, helper objects, or status values. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ento::shouldRegisterGCDAntipattern`。 返回语句将计算结果、辅助对象或状态值交回调用方。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

## Key Concepts / 关键概念

- **Checker registration / 检查器注册**: Connects logic to Static Analyzer checker registration and dispatch. 将逻辑接入静态分析器的检查器注册与分派机制。
- **Diagnostic reporting / 诊断报告**: Builds bug reports or diagnostics for discovered problems. 为发现的问题构建缺陷报告或诊断信息。
- **Clang AST integration / Clang AST 集成**: Reads semantic information from Clang AST data structures. 从 Clang AST 数据结构读取语义信息。
- **Declaration handling / 声明处理**: Works with declaration nodes that describe source-level entities. 处理描述源级实体的声明节点。
- **Statement handling / 语句处理**: Examines statement-level control flow or expression trees. 分析语句级控制流或表达式树。
- **`GCDAntipatternChecker` / `GCDAntipatternChecker`**: `GCDAntipatternChecker` is a prominent symbol in this file and helps define its structure or behavior. `GCDAntipatternChecker` 是该文件中的关键符号之一，用来界定其结构或行为。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/StaticAnalyzer/Checkers/BuiltinCheckerRegistration.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/StaticAnalyzer/Core/BugReporter/BugReporter.h`, `clang/StaticAnalyzer/Core/BugReporter/BugType.h`, `clang/StaticAnalyzer/Core/Checker.h`, `clang/StaticAnalyzer/Core/PathSensitive/AnalysisManager.h`
- **LLVM / LLVM**: `llvm/Support/Debug.h`
