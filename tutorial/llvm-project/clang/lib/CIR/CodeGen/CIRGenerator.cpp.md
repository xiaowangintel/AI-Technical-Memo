# CIRGenerator.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/CIR/CodeGen/CIRGenerator.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This builds an AST and converts it to CIR.
- **Purpose (CN)**: 实现与 `CIRGenerator` 相关的 CIR 代码生成支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: //===--- CIRGenerator.cpp - Emit CIR from ASTs ----------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This builds an AST and converts it to CIR.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CIRGenModule.h"
  14: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `CIRGenModule.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `CIRGenModule.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 15-19
```cpp
  15: #include "mlir/Dialect/OpenACC/OpenACCOpsDialect.h.inc"
  16: #include "mlir/Dialect/OpenMP/OpenMPDialect.h"
  17: #include "mlir/IR/MLIRContext.h"
  18: #include "mlir/Target/LLVMIR/Import.h"
  19: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `OpenACCOpsDialect.h.inc`, `OpenMPDialect.h`, `MLIRContext.h`, `Import.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `OpenACCOpsDialect.h.inc`, `OpenMPDialect.h`, `MLIRContext.h`, `Import.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 20-31
```cpp
  20: #include "clang/AST/DeclGroup.h"
  21: #include "clang/CIR/CIRGenerator.h"
  22: #include "clang/CIR/Dialect/IR/CIRDialect.h"
  23: #include "clang/CIR/Dialect/OpenACC/RegisterOpenACCExtensions.h"
  24: #include "clang/CIR/Dialect/OpenMP/RegisterOpenMPExtensions.h"
  25: #include "llvm/IR/DataLayout.h"
  26: 
  27: using namespace cir;
  28: using namespace clang;
  29: 
  30: void CIRGenerator::anchor() {}
  31: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Notable callable symbols here include `CIRGenerator::anchor`. Included headers like `DeclGroup.h`, `CIRGenerator.h`, `CIRDialect.h`, `RegisterOpenACCExtensions.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 其中值得关注的可调用符号包括 `CIRGenerator::anchor`。 像 `DeclGroup.h`, `CIRGenerator.h`, `CIRDialect.h`, `RegisterOpenACCExtensions.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 32-41
```cpp
  32: CIRGenerator::CIRGenerator(clang::DiagnosticsEngine &diags,
  33:                            llvm::IntrusiveRefCntPtr<llvm::vfs::FileSystem> vfs,
  34:                            const CodeGenOptions &cgo)
  35:     : diags(diags), fs(std::move(vfs)), codeGenOpts{cgo},
  36:       handlingTopLevelDecls{0} {}
  37: CIRGenerator::~CIRGenerator() {
  38:   // There should normally not be any leftover inline method definitions.
  39:   assert(deferredInlineMemberFuncDefs.empty() || diags.hasErrorOccurred());
  40: }
  41: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::CIRGenerator`, `assert`. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::CIRGenerator`、`assert`。 断言用于说明实现期望始终成立的不变量。

### Lines 42-53
```cpp
  42: static void setMLIRDataLayout(mlir::ModuleOp &mod, const llvm::DataLayout &dl) {
  43:   mlir::MLIRContext *mlirContext = mod.getContext();
  44:   mlir::DataLayoutSpecInterface dlSpec =
  45:       mlir::translateDataLayout(dl, mlirContext);
  46:   mod->setAttr(mlir::DLTIDialect::kDataLayoutAttrName, dlSpec);
  47: }
  48: 
  49: void CIRGenerator::Initialize(ASTContext &astContext) {
  50:   using namespace llvm;
  51: 
  52:   this->astContext = &astContext;
  53: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `setMLIRDataLayout`, `mlir::translateDataLayout`, `CIRGenerator::Initialize`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `setMLIRDataLayout`、`mlir::translateDataLayout`、`CIRGenerator::Initialize`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 54-59
```cpp
  54:   mlirContext = std::make_unique<mlir::MLIRContext>();
  55:   mlirContext->loadDialect<mlir::DLTIDialect>();
  56:   mlirContext->loadDialect<cir::CIRDialect>();
  57:   mlirContext->getOrLoadDialect<mlir::acc::OpenACCDialect>();
  58:   mlirContext->getOrLoadDialect<mlir::omp::OpenMPDialect>();
  59: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 60-65
```cpp
  60:   // Register extensions to integrate CIR types with OpenACC and OpenMP.
  61:   mlir::DialectRegistry registry;
  62:   cir::acc::registerOpenACCExtensions(registry);
  63:   cir::omp::registerOpenMPExtensions(registry);
  64:   mlirContext->appendDialectRegistry(registry);
  65: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `cir::acc::registerOpenACCExtensions`, `cir::omp::registerOpenMPExtensions`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `cir::acc::registerOpenACCExtensions`、`cir::omp::registerOpenMPExtensions`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 66-77
```cpp
  66:   cgm = std::make_unique<clang::CIRGen::CIRGenModule>(
  67:       *mlirContext.get(), astContext, codeGenOpts, diags);
  68:   mlir::ModuleOp mod = cgm->getModule();
  69:   llvm::DataLayout layout =
  70:       llvm::DataLayout(astContext.getTargetInfo().getDataLayoutString());
  71:   setMLIRDataLayout(mod, layout);
  72: }
  73: 
  74: bool CIRGenerator::verifyModule() const { return cgm->verifyModule(); }
  75: 
  76: mlir::ModuleOp CIRGenerator::getModule() const { return cgm->getModule(); }
  77: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `llvm::DataLayout`, `setMLIRDataLayout`, `CIRGenerator::verifyModule`, `CIRGenerator::getModule`. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `llvm::DataLayout`、`setMLIRDataLayout`、`CIRGenerator::verifyModule`、`CIRGenerator::getModule`。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 78-89
```cpp
  78: bool CIRGenerator::HandleTopLevelDecl(DeclGroupRef group) {
  79:   if (diags.hasUnrecoverableErrorOccurred())
  80:     return true;
  81: 
  82:   HandlingTopLevelDeclRAII handlingDecl(*this);
  83: 
  84:   for (Decl *decl : group)
  85:     cgm->emitTopLevelDecl(decl);
  86: 
  87:   return true;
  88: }
  89: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::HandleTopLevelDecl`, `handlingDecl`. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::HandleTopLevelDecl`、`handlingDecl`。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 90-94
```cpp
  90: void CIRGenerator::HandleTranslationUnit(ASTContext &astContext) {
  91:   // Release the Builder when there is no error.
  92:   if (!diags.hasErrorOccurred() && cgm)
  93:     cgm->release();
  94: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::HandleTranslationUnit`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::HandleTranslationUnit`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 95-99
```cpp
  95:   // If there are errors before or when releasing the cgm, reset the module to
  96:   // stop here before invoking the backend.
  97:   assert(!cir::MissingFeatures::cleanupAfterErrorDiags());
  98: }
  99: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 100-105
```cpp
 100: void CIRGenerator::HandleInlineFunctionDefinition(FunctionDecl *d) {
 101:   if (diags.hasErrorOccurred())
 102:     return;
 103: 
 104:   assert(d->doesThisDeclarationHaveABody());
 105: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::HandleInlineFunctionDefinition`, `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::HandleInlineFunctionDefinition`、`assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。

### Lines 106-114
```cpp
 106:   // We may want to emit this definition. However, that decision might be
 107:   // based on computing the linkage, and we have to defer that in case we are
 108:   // inside of something that will chagne the method's final linkage, e.g.
 109:   //   typedef struct {
 110:   //     void bar();
 111:   //     void foo() { bar(); }
 112:   //   } A;
 113:   deferredInlineMemberFuncDefs.push_back(d);
 114: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。

### Lines 115-120
```cpp
 115:   // Provide some coverage mapping even for methods that aren't emitted.
 116:   // Don't do this for templated classes though, as they may not be
 117:   // instantiable.
 118:   assert(!cir::MissingFeatures::coverageMapping());
 119: }
 120: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 121-124
```cpp
 121: void CIRGenerator::emitDeferredDecls() {
 122:   if (deferredInlineMemberFuncDefs.empty())
 123:     return;
 124: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::emitDeferredDecls`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::emitDeferredDecls`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 125-135
```cpp
 125:   // Emit any deferred inline method definitions. Note that more deferred
 126:   // methods may be added during this loop, since ASTConsumer callbacks can be
 127:   // invoked if AST inspection results in declarations being added. Therefore,
 128:   // we use an index to loop over the deferredInlineMemberFuncDefs rather than
 129:   // a range.
 130:   HandlingTopLevelDeclRAII handlingDecls(*this);
 131:   for (unsigned i = 0; i != deferredInlineMemberFuncDefs.size(); ++i)
 132:     cgm->emitTopLevelDecl(deferredInlineMemberFuncDefs[i]);
 133:   deferredInlineMemberFuncDefs.clear();
 134: }
 135: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handlingDecls`. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handlingDecls`。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 136-143
```cpp
 136: /// HandleTagDeclDefinition - This callback is invoked each time a TagDecl to
 137: /// (e.g. struct, union, enum, class) is completed. This allows the client to
 138: /// hack on the type, which can occur at any point in the file (because these
 139: /// can be defined in declspecs).
 140: void CIRGenerator::HandleTagDeclDefinition(TagDecl *d) {
 141:   if (diags.hasErrorOccurred())
 142:     return;
 143: 
```
- **EN**: This block defines or extends a data type that carries state, configuration, or interface information. Notable callable symbols here include `CIRGenerator::HandleTagDeclDefinition`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或扩展了承载状态、配置或接口信息的数据类型。 其中值得关注的可调用符号包括 `CIRGenerator::HandleTagDeclDefinition`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 144-149
```cpp
 144:   // Don't allow re-entrant calls to CIRGen triggered by PCH deserialization to
 145:   // emit deferred decls.
 146:   HandlingTopLevelDeclRAII handlingDecl(*this, /*EmitDeferred=*/false);
 147: 
 148:   cgm->updateCompletedType(d);
 149: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handlingDecl`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handlingDecl`。

### Lines 150-154
```cpp
 150:   // For MSVC compatibility, treat declarations of static data members with
 151:   // inline initializers as definitions.
 152:   if (astContext->getTargetInfo().getCXXABI().isMicrosoft())
 153:     cgm->errorNYI(d->getSourceRange(), "HandleTagDeclDefinition: MSABI");
 154: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 155-171
```cpp
 155:   // For OpenMP emit declare reduction functions or declare mapper, if
 156:   // required.
 157:   if (astContext->getLangOpts().OpenMP) {
 158:     for (Decl *member : d->decls()) {
 159:       if (auto *drd = dyn_cast<OMPDeclareReductionDecl>(member)) {
 160:         if (astContext->DeclMustBeEmitted(drd))
 161:           cgm->errorNYI(d->getSourceRange(),
 162:                         "HandleTagDeclDefinition: OMPDeclareReductionDecl");
 163:       } else if (auto *dmd = dyn_cast<OMPDeclareMapperDecl>(member)) {
 164:         if (astContext->DeclMustBeEmitted(dmd))
 165:           cgm->errorNYI(d->getSourceRange(),
 166:                         "HandleTagDeclDefinition: OMPDeclareMapperDecl");
 167:       }
 168:     }
 169:   }
 170: }
 171: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Loop constructs indicate repeated processing over collections or state transitions.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 循环结构说明这里会对集合元素或状态迁移进行重复处理。

### Lines 172-178
```cpp
 172: void CIRGenerator::HandleTagDeclRequiredDefinition(const TagDecl *D) {
 173:   if (diags.hasErrorOccurred())
 174:     return;
 175: 
 176:   assert(!cir::MissingFeatures::generateDebugInfo());
 177: }
 178: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::HandleTagDeclRequiredDefinition`, `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::HandleTagDeclRequiredDefinition`、`assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 179-185
```cpp
 179: void CIRGenerator::HandleCXXStaticMemberVarInstantiation(VarDecl *D) {
 180:   if (diags.hasErrorOccurred())
 181:     return;
 182: 
 183:   cgm->handleCXXStaticMemberVarInstantiation(D);
 184: }
 185: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::HandleCXXStaticMemberVarInstantiation`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::HandleCXXStaticMemberVarInstantiation`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 186-191
```cpp
 186: void CIRGenerator::HandleOpenACCRoutineReference(const FunctionDecl *FD,
 187:                                                  const OpenACCRoutineDecl *RD) {
 188:   llvm::StringRef mangledName = cgm->getMangledName(FD);
 189:   cir::FuncOp entry =
 190:       mlir::dyn_cast_if_present<cir::FuncOp>(cgm->getGlobalValue(mangledName));
 191: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::HandleOpenACCRoutineReference`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::HandleOpenACCRoutineReference`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 192-197
```cpp
 192:   // if this wasn't generated, don't force it to be.
 193:   if (!entry)
 194:     return;
 195:   cgm->emitOpenACCRoutineDecl(FD, entry, RD->getBeginLoc(), RD->clauses());
 196: }
 197: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 198-204
```cpp
 198: void CIRGenerator::CompleteTentativeDefinition(VarDecl *d) {
 199:   if (diags.hasErrorOccurred())
 200:     return;
 201: 
 202:   cgm->emitTentativeDefinition(d);
 203: }
 204: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::CompleteTentativeDefinition`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::CompleteTentativeDefinition`。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 205-210
```cpp
 205: void CIRGenerator::HandleVTable(CXXRecordDecl *rd) {
 206:   if (diags.hasErrorOccurred())
 207:     return;
 208: 
 209:   cgm->emitVTable(rd);
 210: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `CIRGenerator::HandleVTable`. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `CIRGenerator::HandleVTable`。 条件分支用于保护特殊情况、特性检查或错误路径。

## Key Concepts / 关键概念

- **Clang AST integration / Clang AST 集成**: Reads semantic information from Clang AST data structures. 从 Clang AST 数据结构读取语义信息。
- **Declaration handling / 声明处理**: Works with declaration nodes that describe source-level entities. 处理描述源级实体的声明节点。
- **MLIR/CIR integration / MLIR/CIR 集成**: Bridges Clang logic to MLIR/CIR operations, attributes, or types. 将 Clang 逻辑连接到 MLIR/CIR 的操作、属性或类型。
- **CIR dialect usage / CIR 方言使用**: Manipulates CIR-specific types, attributes, and operations. 操作 CIR 专用的类型、属性和操作。
- **CIR generation pipeline / CIR 生成流水线**: Participates in the lowering pipeline from Clang semantics to CIR constructs. 参与从 Clang 语义到 CIR 构造的 lowering 流水线。
- **OpenMP support / OpenMP 支持**: Contains logic related to OpenMP semantics or offloading. 包含与 OpenMP 语义或卸载相关的逻辑。

## Dependencies / 依赖关系

- **Clang / Clang**: `clang/AST/DeclGroup.h`, `clang/CIR/CIRGenerator.h`, `clang/CIR/Dialect/IR/CIRDialect.h`, `clang/CIR/Dialect/OpenACC/RegisterOpenACCExtensions.h`, `clang/CIR/Dialect/OpenMP/RegisterOpenMPExtensions.h`
- **LLVM / LLVM**: `llvm/IR/DataLayout.h`
- **MLIR / MLIR**: `mlir/Dialect/OpenACC/OpenACCOpsDialect.h.inc`, `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/IR/MLIRContext.h`, `mlir/Target/LLVMIR/Import.h`
- **StdLib/Other / 标准库/其他**: `CIRGenModule.h`
