# DirectIvarAssignment.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/StaticAnalyzer/Checkers/DirectIvarAssignment.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: Check that Objective C properties are set with the setter, not though a direct assignment Two versions of a checker exist: one that checks all methods and the other.
- **Purpose (CN)**: 实现或支撑 `DirectIvarAssignment` 静态分析检查器的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
   1: //===- DirectIvarAssignment.cpp - Check rules on ObjC properties -*- C++ -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: //  Check that Objective C properties are set with the setter, not though a
  10: //      direct assignment.
  11: //
  12: //  Two versions of a checker exist: one that checks all methods and the other
  13: //      that only checks the methods annotated with
  14: //      __attribute__((annotate("objc_no_direct_instance_variable_assignment")))
  15: //
  16: //  The checker does not warn about assignments to Ivars, annotated with
  17: //       __attribute__((objc_allow_direct_instance_variable_assignment"))). This
  18: //      annotation serves as a false positive suppression mechanism for the
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 19-22
```cpp
  19: //      checker. The annotation is allowed on properties and Ivars.
  20: //
  21: //===----------------------------------------------------------------------===//
  22: 
```
- **EN**: This block contains banner or explanatory comments that frame the file and document its intent.
- **CN**: 该代码块主要包含文件横幅或说明性注释，用于交代文件用途和背景。

### Lines 23-36
```cpp
  23: #include "clang/StaticAnalyzer/Checkers/BuiltinCheckerRegistration.h"
  24: #include "clang/AST/Attr.h"
  25: #include "clang/AST/DeclObjC.h"
  26: #include "clang/AST/StmtVisitor.h"
  27: #include "clang/StaticAnalyzer/Core/BugReporter/BugReporter.h"
  28: #include "clang/StaticAnalyzer/Core/Checker.h"
  29: #include "clang/StaticAnalyzer/Core/PathSensitive/AnalysisManager.h"
  30: #include "llvm/ADT/DenseMap.h"
  31: 
  32: using namespace clang;
  33: using namespace ento;
  34: 
  35: namespace {
  36: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `BuiltinCheckerRegistration.h`, `Attr.h`, `DeclObjC.h`, `StmtVisitor.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `BuiltinCheckerRegistration.h`, `Attr.h`, `DeclObjC.h`, `StmtVisitor.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 37-56
```cpp
  37: /// The default method filter, which is used to filter out the methods on which
  38: /// the check should not be performed.
  39: ///
  40: /// Checks for the init, dealloc, and any other functions that might be allowed
  41: /// to perform direct instance variable assignment based on their name.
  42: static bool DefaultMethodFilter(const ObjCMethodDecl *M) {
  43:   return M->getMethodFamily() == OMF_init ||
  44:          M->getMethodFamily() == OMF_dealloc ||
  45:          M->getMethodFamily() == OMF_copy ||
  46:          M->getMethodFamily() == OMF_mutableCopy ||
  47:          M->getSelector().getNameForSlot(0).contains("init") ||
  48:          M->getSelector().getNameForSlot(0).contains("Init");
  49: }
  50: 
  51: class DirectIvarAssignment :
  52:   public Checker<check::ASTDecl<ObjCImplementationDecl> > {
  53: 
  54:   typedef llvm::DenseMap<const ObjCIvarDecl*,
  55:                          const ObjCPropertyDecl*> IvarToPropertyMapTy;
  56: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `DefaultMethodFilter`. It introduces or references types such as `DirectIvarAssignment`. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `DefaultMethodFilter`。 它引入或引用了诸如 `DirectIvarAssignment` 等类型。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 57-66
```cpp
  57:   /// A helper class, which walks the AST and locates all assignments to ivars
  58:   /// in the given function.
  59:   class MethodCrawler : public ConstStmtVisitor<MethodCrawler> {
  60:     const IvarToPropertyMapTy &IvarToPropMap;
  61:     const ObjCMethodDecl *MD;
  62:     const ObjCInterfaceDecl *InterfD;
  63:     BugReporter &BR;
  64:     const CheckerBase *Checker;
  65:     LocationOrAnalysisDeclContext DCtx;
  66: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. It introduces or references types such as `MethodCrawler`.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 它引入或引用了诸如 `MethodCrawler` 等类型。

### Lines 67-77
```cpp
  67:   public:
  68:     MethodCrawler(const IvarToPropertyMapTy &InMap, const ObjCMethodDecl *InMD,
  69:                   const ObjCInterfaceDecl *InID, BugReporter &InBR,
  70:                   const CheckerBase *Checker, AnalysisDeclContext *InDCtx)
  71:         : IvarToPropMap(InMap), MD(InMD), InterfD(InID), BR(InBR),
  72:           Checker(Checker), DCtx(InDCtx) {}
  73: 
  74:     void VisitStmt(const Stmt *S) { VisitChildren(S); }
  75: 
  76:     void VisitBinaryOperator(const BinaryOperator *BO);
  77: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `MethodCrawler`, `VisitStmt`, `VisitBinaryOperator`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `MethodCrawler`、`VisitStmt`、`VisitBinaryOperator`。

### Lines 78-89
```cpp
  78:     void VisitChildren(const Stmt *S) {
  79:       for (const Stmt *Child : S->children())
  80:         if (Child)
  81:           this->Visit(Child);
  82:     }
  83:   };
  84: 
  85: public:
  86:   bool (*ShouldSkipMethod)(const ObjCMethodDecl *);
  87: 
  88:   DirectIvarAssignment() : ShouldSkipMethod(&DefaultMethodFilter) {}
  89: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `VisitChildren`, `bool`, `DirectIvarAssignment`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `VisitChildren`、`bool`、`DirectIvarAssignment`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 90-93
```cpp
  90:   void checkASTDecl(const ObjCImplementationDecl *D, AnalysisManager& Mgr,
  91:                     BugReporter &BR) const;
  92: };
  93: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `checkASTDecl`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `checkASTDecl`。

### Lines 94-103
```cpp
  94: static const ObjCIvarDecl *findPropertyBackingIvar(const ObjCPropertyDecl *PD,
  95:                                                const ObjCInterfaceDecl *InterD,
  96:                                                ASTContext &Ctx) {
  97:   // Check for synthesized ivars.
  98:   ObjCIvarDecl *ID = PD->getPropertyIvarDecl();
  99:   if (ID)
 100:     return ID;
 101: 
 102:   ObjCInterfaceDecl *NonConstInterD = const_cast<ObjCInterfaceDecl*>(InterD);
 103: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 104-108
```cpp
 104:   // Check for existing "_PropName".
 105:   ID = NonConstInterD->lookupInstanceVariable(PD->getDefaultSynthIvarName(Ctx));
 106:   if (ID)
 107:     return ID;
 108: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 109-115
```cpp
 109:   // Check for existing "PropName".
 110:   IdentifierInfo *PropIdent = PD->getIdentifier();
 111:   ID = NonConstInterD->lookupInstanceVariable(PropIdent);
 112: 
 113:   return ID;
 114: }
 115: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 116-123
```cpp
 116: void DirectIvarAssignment::checkASTDecl(const ObjCImplementationDecl *D,
 117:                                        AnalysisManager& Mgr,
 118:                                        BugReporter &BR) const {
 119:   const ObjCInterfaceDecl *InterD = D->getClassInterface();
 120: 
 121: 
 122:   IvarToPropertyMapTy IvarToPropMap;
 123: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `DirectIvarAssignment::checkASTDecl`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `DirectIvarAssignment::checkASTDecl`。

### Lines 124-132
```cpp
 124:   // Find all properties for this class.
 125:   for (const auto *PD : InterD->instance_properties()) {
 126:     // Find the corresponding IVar.
 127:     const ObjCIvarDecl *ID = findPropertyBackingIvar(PD, InterD,
 128:                                                      Mgr.getASTContext());
 129: 
 130:     if (!ID)
 131:       continue;
 132: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 133-145
```cpp
 133:     // Store the IVar to property mapping.
 134:     IvarToPropMap[ID] = PD;
 135:   }
 136: 
 137:   if (IvarToPropMap.empty())
 138:     return;
 139: 
 140:   for (const auto *M : D->instance_methods()) {
 141:     AnalysisDeclContext *DCtx = Mgr.getAnalysisDeclContext(M);
 142: 
 143:     if ((*ShouldSkipMethod)(M))
 144:       continue;
 145: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 146-150
```cpp
 146:     const Stmt *Body = M->getBody();
 147:     if (M->isSynthesizedAccessorStub())
 148:       continue;
 149:     assert(Body);
 150: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。

### Lines 151-156
```cpp
 151:     MethodCrawler MC(IvarToPropMap, M->getCanonicalDecl(), InterD, BR, this,
 152:                      DCtx);
 153:     MC.VisitStmt(Body);
 154:   }
 155: }
 156: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `MC`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `MC`。

### Lines 157-164
```cpp
 157: static bool isAnnotatedToAllowDirectAssignment(const Decl *D) {
 158:   for (const auto *Ann : D->specific_attrs<AnnotateAttr>())
 159:     if (Ann->getAnnotation() ==
 160:         "objc_allow_direct_instance_variable_assignment")
 161:       return true;
 162:   return false;
 163: }
 164: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `isAnnotatedToAllowDirectAssignment`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `isAnnotatedToAllowDirectAssignment`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 165-178
```cpp
 165: void DirectIvarAssignment::MethodCrawler::VisitBinaryOperator(
 166:                                                     const BinaryOperator *BO) {
 167:   if (!BO->isAssignmentOp())
 168:     return;
 169: 
 170:   const ObjCIvarRefExpr *IvarRef =
 171:           dyn_cast<ObjCIvarRefExpr>(BO->getLHS()->IgnoreParenCasts());
 172: 
 173:   if (!IvarRef)
 174:     return;
 175: 
 176:   if (const ObjCIvarDecl *D = IvarRef->getDecl()) {
 177:     IvarToPropertyMapTy::const_iterator I = IvarToPropMap.find(D);
 178: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `DirectIvarAssignment::MethodCrawler::VisitBinaryOperator`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `DirectIvarAssignment::MethodCrawler::VisitBinaryOperator`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 179-188
```cpp
 179:     if (I != IvarToPropMap.end()) {
 180:       const ObjCPropertyDecl *PD = I->second;
 181:       // Skip warnings on Ivars, annotated with
 182:       // objc_allow_direct_instance_variable_assignment. This annotation serves
 183:       // as a false positive suppression mechanism for the checker. The
 184:       // annotation is allowed on properties and ivars.
 185:       if (isAnnotatedToAllowDirectAssignment(PD) ||
 186:           isAnnotatedToAllowDirectAssignment(D))
 187:         return;
 188: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 189-199
```cpp
 189:       ObjCMethodDecl *GetterMethod =
 190:           InterfD->getInstanceMethod(PD->getGetterName());
 191:       ObjCMethodDecl *SetterMethod =
 192:           InterfD->getInstanceMethod(PD->getSetterName());
 193: 
 194:       if (SetterMethod && SetterMethod->getCanonicalDecl() == MD)
 195:         return;
 196: 
 197:       if (GetterMethod && GetterMethod->getCanonicalDecl() == MD)
 198:         return;
 199: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 200-209
```cpp
 200:       BR.EmitBasicReport(
 201:           MD, Checker, "Property access", categories::CoreFoundationObjectiveC,
 202:           "Direct assignment to an instance variable backing a property; "
 203:           "use the setter instead",
 204:           PathDiagnosticLocation(IvarRef, BR.getSourceManager(), DCtx));
 205:     }
 206:   }
 207: }
 208: }
 209: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `PathDiagnosticLocation`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `PathDiagnosticLocation`。

### Lines 210-218
```cpp
 210: // Register the checker that checks for direct accesses in functions annotated
 211: // with __attribute__((annotate("objc_no_direct_instance_variable_assignment"))).
 212: static bool AttrFilter(const ObjCMethodDecl *M) {
 213:   for (const auto *Ann : M->specific_attrs<AnnotateAttr>())
 214:     if (Ann->getAnnotation() == "objc_no_direct_instance_variable_assignment")
 215:       return false;
 216:   return true;
 217: }
 218: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `AttrFilter`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `AttrFilter`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 219-227
```cpp
 219: // Register the checker that checks for direct accesses in all functions,
 220: // except for the initialization and copy routines.
 221: void ento::registerDirectIvarAssignment(CheckerManager &mgr) {
 222:   auto Chk = mgr.registerChecker<DirectIvarAssignment>();
 223:   if (mgr.getAnalyzerOptions().getCheckerBooleanOption(Chk,
 224:                                                        "AnnotatedFunctions"))
 225:     Chk->ShouldSkipMethod = &AttrFilter;
 226: }
 227: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ento::registerDirectIvarAssignment`. Conditional branches guard special cases, feature checks, or error paths. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ento::registerDirectIvarAssignment`。 条件分支用于保护特殊情况、特性检查或错误路径。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

### Lines 228-230
```cpp
 228: bool ento::shouldRegisterDirectIvarAssignment(const CheckerManager &mgr) {
 229:   return true;
 230: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `ento::shouldRegisterDirectIvarAssignment`. Return statements hand back computed state, helper objects, or status values. Checker framework types connect the block to callback-driven static analysis execution.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `ento::shouldRegisterDirectIvarAssignment`。 返回语句将计算结果、辅助对象或状态值交回调用方。 检查器框架类型表明该代码块与回调驱动的静态分析执行过程相连。

## Key Concepts / 关键概念

- **Checker registration / 检查器注册**: Connects logic to Static Analyzer checker registration and dispatch. 将逻辑接入静态分析器的检查器注册与分派机制。
- **Clang AST integration / Clang AST 集成**: Reads semantic information from Clang AST data structures. 从 Clang AST 数据结构读取语义信息。
- **Declaration handling / 声明处理**: Works with declaration nodes that describe source-level entities. 处理描述源级实体的声明节点。
- **Statement handling / 语句处理**: Examines statement-level control flow or expression trees. 分析语句级控制流或表达式树。
- **`DirectIvarAssignment` / `DirectIvarAssignment`**: `DirectIvarAssignment` is a prominent symbol in this file and helps define its structure or behavior. `DirectIvarAssignment` 是该文件中的关键符号之一，用来界定其结构或行为。
- **`MethodCrawler` / `MethodCrawler`**: `MethodCrawler` is a prominent symbol in this file and helps define its structure or behavior. `MethodCrawler` 是该文件中的关键符号之一，用来界定其结构或行为。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/StaticAnalyzer/Checkers/BuiltinCheckerRegistration.h`, `clang/AST/Attr.h`, `clang/AST/DeclObjC.h`, `clang/AST/StmtVisitor.h`, `clang/StaticAnalyzer/Core/BugReporter/BugReporter.h`, `clang/StaticAnalyzer/Core/Checker.h`, `clang/StaticAnalyzer/Core/PathSensitive/AnalysisManager.h`
- **LLVM / LLVM**: `llvm/ADT/DenseMap.h`
