# CGOpenMPRuntimeGPU.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGOpenMPRuntimeGPU.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the CGOpenMPRuntimeGPU portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 CGOpenMPRuntimeGPU 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```cpp
 1: //===---- CGOpenMPRuntimeGPU.cpp - Interface to OpenMP GPU Runtimes ----===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This provides a generalized class for OpenMP runtime code generation
10: // specialized by GPU targets NVPTX, AMDGCN and SPIR-V.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #include "CGOpenMPRuntimeGPU.h"
15: #include "CGDebugInfo.h"
16: #include "CodeGenFunction.h"
17: #include "clang/AST/Attr.h"
18: #include "clang/AST/DeclOpenMP.h"
19: #include "clang/AST/OpenMPClause.h"
20: #include "clang/AST/StmtOpenMP.h"
21: #include "clang/AST/StmtVisitor.h"
22: #include "clang/Basic/Cuda.h"
23: #include "llvm/ADT/SmallPtrSet.h"
24: #include "llvm/Frontend/OpenMP/OMPDeviceConstants.h"
25: #include "llvm/Frontend/OpenMP/OMPGridValues.h"
```
- **EN**: This block imports local CodeGen headers `CGOpenMPRuntimeGPU.h`, `CGDebugInfo.h`, `CodeGenFunction.h`; Clang headers `clang/AST/Attr.h`, `clang/AST/DeclOpenMP.h`, `clang/AST/OpenMPClause.h`, and 3 more; LLVM headers `llvm/ADT/SmallPtrSet.h`, `llvm/Frontend/OpenMP/OMPDeviceConstants.h`, `llvm/Frontend/OpenMP/OMPGridValues.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGOpenMPRuntimeGPU.h`, `CGDebugInfo.h`, `CodeGenFunction.h`；Clang 头文件 `clang/AST/Attr.h`, `clang/AST/DeclOpenMP.h`, `clang/AST/OpenMPClause.h`, and 3 more；LLVM 头文件 `llvm/ADT/SmallPtrSet.h`, `llvm/Frontend/OpenMP/OMPDeviceConstants.h`, `llvm/Frontend/OpenMP/OMPGridValues.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 26-50
```cpp
26: 
27: using namespace clang;
28: using namespace CodeGen;
29: using namespace llvm::omp;
30: 
31: namespace {
32: /// Pre(post)-action for different OpenMP constructs specialized for NVPTX.
33: class NVPTXActionTy final : public PrePostActionTy {
34:   llvm::FunctionCallee EnterCallee = nullptr;
35:   ArrayRef<llvm::Value *> EnterArgs;
36:   llvm::FunctionCallee ExitCallee = nullptr;
37:   ArrayRef<llvm::Value *> ExitArgs;
38:   bool Conditional = false;
39:   llvm::BasicBlock *ContBlock = nullptr;
40: 
41: public:
42:   NVPTXActionTy(llvm::FunctionCallee EnterCallee,
43:                 ArrayRef<llvm::Value *> EnterArgs,
44:                 llvm::FunctionCallee ExitCallee,
45:                 ArrayRef<llvm::Value *> ExitArgs, bool Conditional = false)
46:       : EnterCallee(EnterCallee), EnterArgs(EnterArgs), ExitCallee(ExitCallee),
47:         ExitArgs(ExitArgs), Conditional(Conditional) {}
48:   void Enter(CodeGenFunction &CGF) override {
49:     llvm::Value *EnterRes = CGF.EmitRuntimeCall(EnterCallee, EnterArgs);
50:     if (Conditional) {
```
- **EN**: This block opens or references namespaces `clang`, `CodeGen`, `llvm`; introduces declarations such as `NVPTXActionTy`; defines callable entry points like `NVPTXActionTy`, `Enter`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块打开或引用命名空间 `clang`, `CodeGen`, `llvm`；给出诸如 `NVPTXActionTy` 的声明；定义可调用入口，例如 `NVPTXActionTy`, `Enter`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 51-75
```cpp
51:       llvm::Value *CallBool = CGF.Builder.CreateIsNotNull(EnterRes);
52:       auto *ThenBlock = CGF.createBasicBlock("omp_if.then");
53:       ContBlock = CGF.createBasicBlock("omp_if.end");
54:       // Generate the branch (If-stmt)
55:       CGF.Builder.CreateCondBr(CallBool, ThenBlock, ContBlock);
56:       CGF.EmitBlock(ThenBlock);
57:     }
58:   }
59:   void Done(CodeGenFunction &CGF) {
60:     // Emit the rest of blocks/branches
61:     CGF.EmitBranch(ContBlock);
62:     CGF.EmitBlock(ContBlock, true);
63:   }
64:   void Exit(CodeGenFunction &CGF) override {
65:     CGF.EmitRuntimeCall(ExitCallee, ExitArgs);
66:   }
67: };
68: 
69: /// A class to track the execution mode when codegening directives within
70: /// a target region. The appropriate mode (SPMD|NON-SPMD) is set on entry
71: /// to the target region and used by containing directives such as 'parallel'
72: /// to emit optimized code.
73: class ExecutionRuntimeModesRAII {
74: private:
75:   CGOpenMPRuntimeGPU::ExecutionMode SavedExecMode =
```
- **EN**: This block introduces declarations such as `ExecutionRuntimeModesRAII`; defines callable entry points like `Done`, `Exit`.
- **CN**: 该代码块给出诸如 `ExecutionRuntimeModesRAII` 的声明；定义可调用入口，例如 `Done`, `Exit`。

### Lines 76-100
```cpp
 76:       CGOpenMPRuntimeGPU::EM_Unknown;
 77:   CGOpenMPRuntimeGPU::ExecutionMode &ExecMode;
 78: 
 79: public:
 80:   ExecutionRuntimeModesRAII(CGOpenMPRuntimeGPU::ExecutionMode &ExecMode,
 81:                             CGOpenMPRuntimeGPU::ExecutionMode EntryMode)
 82:       : ExecMode(ExecMode) {
 83:     SavedExecMode = ExecMode;
 84:     ExecMode = EntryMode;
 85:   }
 86:   ~ExecutionRuntimeModesRAII() { ExecMode = SavedExecMode; }
 87: };
 88: 
 89: static const ValueDecl *getPrivateItem(const Expr *RefExpr) {
 90:   RefExpr = RefExpr->IgnoreParens();
 91:   if (const auto *ASE = dyn_cast<ArraySubscriptExpr>(RefExpr)) {
 92:     const Expr *Base = ASE->getBase()->IgnoreParenImpCasts();
 93:     while (const auto *TempASE = dyn_cast<ArraySubscriptExpr>(Base))
 94:       Base = TempASE->getBase()->IgnoreParenImpCasts();
 95:     RefExpr = Base;
 96:   } else if (auto *OASE = dyn_cast<ArraySectionExpr>(RefExpr)) {
 97:     const Expr *Base = OASE->getBase()->IgnoreParenImpCasts();
 98:     while (const auto *TempOASE = dyn_cast<ArraySectionExpr>(Base))
 99:       Base = TempOASE->getBase()->IgnoreParenImpCasts();
100:     while (const auto *TempASE = dyn_cast<ArraySubscriptExpr>(Base))
```
- **EN**: This block defines callable entry points like `ExecutionRuntimeModesRAII`, `~ExecutionRuntimeModesRAII`; uses control flow (if, while) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `ExecutionRuntimeModesRAII`, `~ExecutionRuntimeModesRAII`；通过控制流（if, while）细化 LLVM IR 生成 行为。

### Lines 101-125
```cpp
101:       Base = TempASE->getBase()->IgnoreParenImpCasts();
102:     RefExpr = Base;
103:   }
104:   RefExpr = RefExpr->IgnoreParenImpCasts();
105:   if (const auto *DE = dyn_cast<DeclRefExpr>(RefExpr))
106:     return cast<ValueDecl>(DE->getDecl()->getCanonicalDecl());
107:   const auto *ME = cast<MemberExpr>(RefExpr);
108:   return cast<ValueDecl>(ME->getMemberDecl()->getCanonicalDecl());
109: }
110: 
111: static RecordDecl *buildRecordForGlobalizedVars(
112:     ASTContext &C, ArrayRef<const ValueDecl *> EscapedDecls,
113:     ArrayRef<const ValueDecl *> EscapedDeclsForTeams,
114:     llvm::SmallDenseMap<const ValueDecl *, const FieldDecl *>
115:         &MappedDeclsFields,
116:     int BufSize) {
117:   using VarsDataTy = std::pair<CharUnits /*Align*/, const ValueDecl *>;
118:   if (EscapedDecls.empty() && EscapedDeclsForTeams.empty())
119:     return nullptr;
120:   SmallVector<VarsDataTy, 4> GlobalizedVars;
121:   for (const ValueDecl *D : EscapedDecls)
122:     GlobalizedVars.emplace_back(C.getDeclAlign(D), D);
123:   for (const ValueDecl *D : EscapedDeclsForTeams)
124:     GlobalizedVars.emplace_back(C.getDeclAlign(D), D);
125: 
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 126-150
```cpp
126:   // Build struct _globalized_locals_ty {
127:   //         /*  globalized vars  */[WarSize] align (decl_align)
128:   //         /*  globalized vars  */ for EscapedDeclsForTeams
129:   //       };
130:   RecordDecl *GlobalizedRD = C.buildImplicitRecord("_globalized_locals_ty");
131:   GlobalizedRD->startDefinition();
132:   llvm::SmallPtrSet<const ValueDecl *, 16> SingleEscaped(llvm::from_range,
133:                                                          EscapedDeclsForTeams);
134:   for (const auto &Pair : GlobalizedVars) {
135:     const ValueDecl *VD = Pair.second;
136:     QualType Type = VD->getType();
137:     if (Type->isLValueReferenceType())
138:       Type = C.getPointerType(Type.getNonReferenceType());
139:     else
140:       Type = Type.getNonReferenceType();
141:     SourceLocation Loc = VD->getLocation();
142:     FieldDecl *Field;
143:     if (SingleEscaped.count(VD)) {
144:       Field = FieldDecl::Create(
145:           C, GlobalizedRD, Loc, Loc, VD->getIdentifier(), Type,
146:           C.getTrivialTypeSourceInfo(Type, SourceLocation()),
147:           /*BW=*/nullptr, /*Mutable=*/false,
148:           /*InitStyle=*/ICIS_NoInit);
149:       Field->setAccess(AS_public);
150:       if (VD->hasAttrs()) {
```
- **EN**: This block defines callable entry points like `SingleEscaped`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `SingleEscaped`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 151-175
```cpp
151:         for (specific_attr_iterator<AlignedAttr> I(VD->getAttrs().begin()),
152:              E(VD->getAttrs().end());
153:              I != E; ++I)
154:           Field->addAttr(*I);
155:       }
156:     } else {
157:       if (BufSize > 1) {
158:         llvm::APInt ArraySize(32, BufSize);
159:         Type = C.getConstantArrayType(Type, ArraySize, nullptr,
160:                                       ArraySizeModifier::Normal, 0);
161:       }
162:       Field = FieldDecl::Create(
163:           C, GlobalizedRD, Loc, Loc, VD->getIdentifier(), Type,
164:           C.getTrivialTypeSourceInfo(Type, SourceLocation()),
165:           /*BW=*/nullptr, /*Mutable=*/false,
166:           /*InitStyle=*/ICIS_NoInit);
167:       Field->setAccess(AS_public);
168:       llvm::APInt Align(32, Pair.first.getQuantity());
169:       Field->addAttr(AlignedAttr::CreateImplicit(
170:           C, /*IsAlignmentExpr=*/true,
171:           IntegerLiteral::Create(C, Align,
172:                                  C.getIntTypeForBitwidth(32, /*Signed=*/0),
173:                                  SourceLocation()),
174:           {}, AlignedAttr::GNU_aligned));
175:     }
```
- **EN**: This block defines callable entry points like `ArraySize`, `Align`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `ArraySize`, `Align`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 176-200
```cpp
176:     GlobalizedRD->addDecl(Field);
177:     MappedDeclsFields.try_emplace(VD, Field);
178:   }
179:   GlobalizedRD->completeDefinition();
180:   return GlobalizedRD;
181: }
182: 
183: /// Get the list of variables that can escape their declaration context.
184: class CheckVarsEscapingDeclContext final
185:     : public ConstStmtVisitor<CheckVarsEscapingDeclContext> {
186:   CodeGenFunction &CGF;
187:   llvm::SetVector<const ValueDecl *> EscapedDecls;
188:   llvm::SetVector<const ValueDecl *> EscapedVariableLengthDecls;
189:   llvm::SetVector<const ValueDecl *> DelayedVariableLengthDecls;
190:   llvm::SmallPtrSet<const Decl *, 4> EscapedParameters;
191:   RecordDecl *GlobalizedRD = nullptr;
192:   llvm::SmallDenseMap<const ValueDecl *, const FieldDecl *> MappedDeclsFields;
193:   bool AllEscaped = false;
194:   bool IsForCombinedParallelRegion = false;
195: 
196:   void markAsEscaped(const ValueDecl *VD) {
197:     // Do not globalize declare target variables.
198:     if (!isa<VarDecl>(VD) ||
199:         OMPDeclareTargetDeclAttr::isDeclareTargetDeclaration(VD))
200:       return;
```
- **EN**: This block introduces declarations such as `CheckVarsEscapingDeclContext`; defines callable entry points like `markAsEscaped`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块给出诸如 `CheckVarsEscapingDeclContext` 的声明；定义可调用入口，例如 `markAsEscaped`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 201-225
```cpp
201:     VD = cast<ValueDecl>(VD->getCanonicalDecl());
202:     // Use user-specified allocation.
203:     if (VD->hasAttrs() && VD->hasAttr<OMPAllocateDeclAttr>())
204:       return;
205:     // Variables captured by value must be globalized.
206:     bool IsCaptured = false;
207:     if (auto *CSI = CGF.CapturedStmtInfo) {
208:       if (const FieldDecl *FD = CSI->lookup(cast<VarDecl>(VD))) {
209:         // Check if need to capture the variable that was already captured by
210:         // value in the outer region.
211:         IsCaptured = true;
212:         if (!IsForCombinedParallelRegion) {
213:           if (!FD->hasAttrs())
214:             return;
215:           const auto *Attr = FD->getAttr<OMPCaptureKindAttr>();
216:           if (!Attr)
217:             return;
218:           if (((Attr->getCaptureKind() != OMPC_map) &&
219:                !isOpenMPPrivate(Attr->getCaptureKind())) ||
220:               ((Attr->getCaptureKind() == OMPC_map) &&
221:                !FD->getType()->isAnyPointerType()))
222:             return;
223:         }
224:         if (!FD->getType()->isReferenceType()) {
225:           assert(!VD->getType()->isVariablyModifiedType() &&
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 226-250
```cpp
226:                  "Parameter captured by value with variably modified type");
227:           EscapedParameters.insert(VD);
228:         } else if (!IsForCombinedParallelRegion) {
229:           return;
230:         }
231:       }
232:     }
233:     if ((!CGF.CapturedStmtInfo ||
234:          (IsForCombinedParallelRegion && CGF.CapturedStmtInfo)) &&
235:         VD->getType()->isReferenceType())
236:       // Do not globalize variables with reference type.
237:       return;
238:     if (VD->getType()->isVariablyModifiedType()) {
239:       // If not captured at the target region level then mark the escaped
240:       // variable as delayed.
241:       if (IsCaptured)
242:         EscapedVariableLengthDecls.insert(VD);
243:       else
244:         DelayedVariableLengthDecls.insert(VD);
245:     } else
246:       EscapedDecls.insert(VD);
247:   }
248: 
249:   void VisitValueDecl(const ValueDecl *VD) {
250:     if (VD->getType()->isLValueReferenceType())
```
- **EN**: This block defines callable entry points like `VisitValueDecl`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `VisitValueDecl`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 251-275
```cpp
251:       markAsEscaped(VD);
252:     if (const auto *VarD = dyn_cast<VarDecl>(VD)) {
253:       if (!isa<ParmVarDecl>(VarD) && VarD->hasInit()) {
254:         const bool SavedAllEscaped = AllEscaped;
255:         AllEscaped = VD->getType()->isLValueReferenceType();
256:         Visit(VarD->getInit());
257:         AllEscaped = SavedAllEscaped;
258:       }
259:     }
260:   }
261:   void VisitOpenMPCapturedStmt(const CapturedStmt *S,
262:                                ArrayRef<OMPClause *> Clauses,
263:                                bool IsCombinedParallelRegion) {
264:     if (!S)
265:       return;
266:     for (const CapturedStmt::Capture &C : S->captures()) {
267:       if (C.capturesVariable() && !C.capturesVariableByCopy()) {
268:         const ValueDecl *VD = C.getCapturedVar();
269:         bool SavedIsForCombinedParallelRegion = IsForCombinedParallelRegion;
270:         if (IsCombinedParallelRegion) {
271:           // Check if the variable is privatized in the combined construct and
272:           // those private copies must be shared in the inner parallel
273:           // directive.
274:           IsForCombinedParallelRegion = false;
275:           for (const OMPClause *C : Clauses) {
```
- **EN**: This block defines callable entry points like `markAsEscaped`, `Visit`, `VisitOpenMPCapturedStmt`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `markAsEscaped`, `Visit`, `VisitOpenMPCapturedStmt`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 276-300
```cpp
276:             if (!isOpenMPPrivate(C->getClauseKind()) ||
277:                 C->getClauseKind() == OMPC_reduction ||
278:                 C->getClauseKind() == OMPC_linear ||
279:                 C->getClauseKind() == OMPC_private)
280:               continue;
281:             ArrayRef<const Expr *> Vars;
282:             if (const auto *PC = dyn_cast<OMPFirstprivateClause>(C))
283:               Vars = PC->getVarRefs();
284:             else if (const auto *PC = dyn_cast<OMPLastprivateClause>(C))
285:               Vars = PC->getVarRefs();
286:             else
287:               llvm_unreachable("Unexpected clause.");
288:             for (const auto *E : Vars) {
289:               const Decl *D =
290:                   cast<DeclRefExpr>(E)->getDecl()->getCanonicalDecl();
291:               if (D == VD->getCanonicalDecl()) {
292:                 IsForCombinedParallelRegion = true;
293:                 break;
294:               }
295:             }
296:             if (IsForCombinedParallelRegion)
297:               break;
298:           }
299:         }
300:         markAsEscaped(VD);
```
- **EN**: This block defines callable entry points like `markAsEscaped`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `markAsEscaped`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 301-325
```cpp
301:         if (isa<OMPCapturedExprDecl>(VD))
302:           VisitValueDecl(VD);
303:         IsForCombinedParallelRegion = SavedIsForCombinedParallelRegion;
304:       }
305:     }
306:   }
307: 
308:   void buildRecordForGlobalizedVars(bool IsInTTDRegion) {
309:     assert(!GlobalizedRD &&
310:            "Record for globalized variables is built already.");
311:     ArrayRef<const ValueDecl *> EscapedDeclsForParallel, EscapedDeclsForTeams;
312:     unsigned WarpSize = CGF.getTarget().getGridValue().GV_Warp_Size;
313:     if (IsInTTDRegion)
314:       EscapedDeclsForTeams = EscapedDecls.getArrayRef();
315:     else
316:       EscapedDeclsForParallel = EscapedDecls.getArrayRef();
317:     GlobalizedRD = ::buildRecordForGlobalizedVars(
318:         CGF.getContext(), EscapedDeclsForParallel, EscapedDeclsForTeams,
319:         MappedDeclsFields, WarpSize);
320:   }
321: 
322: public:
323:   CheckVarsEscapingDeclContext(CodeGenFunction &CGF,
324:                                ArrayRef<const ValueDecl *> TeamsReductions)
325:       : CGF(CGF), EscapedDecls(llvm::from_range, TeamsReductions) {}
```
- **EN**: This block defines callable entry points like `buildRecordForGlobalizedVars`, `CheckVarsEscapingDeclContext`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `buildRecordForGlobalizedVars`, `CheckVarsEscapingDeclContext`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 326-350
```cpp
326:   ~CheckVarsEscapingDeclContext() = default;
327:   void VisitDeclStmt(const DeclStmt *S) {
328:     if (!S)
329:       return;
330:     for (const Decl *D : S->decls())
331:       if (const auto *VD = dyn_cast_or_null<ValueDecl>(D))
332:         VisitValueDecl(VD);
333:   }
334:   void VisitOMPExecutableDirective(const OMPExecutableDirective *D) {
335:     if (!D)
336:       return;
337:     if (!D->hasAssociatedStmt())
338:       return;
339:     if (const auto *S =
340:             dyn_cast_or_null<CapturedStmt>(D->getAssociatedStmt())) {
341:       // Do not analyze directives that do not actually require capturing,
342:       // like `omp for` or `omp simd` directives.
343:       llvm::SmallVector<OpenMPDirectiveKind, 4> CaptureRegions;
344:       getOpenMPCaptureRegions(CaptureRegions, D->getDirectiveKind());
345:       if (CaptureRegions.size() == 1 && CaptureRegions.back() == OMPD_unknown) {
346:         VisitStmt(S->getCapturedStmt());
347:         return;
348:       }
349:       VisitOpenMPCapturedStmt(
350:           S, D->clauses(),
```
- **EN**: This block defines callable entry points like `VisitDeclStmt`, `VisitOMPExecutableDirective`, `getOpenMPCaptureRegions`, `VisitStmt`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `VisitDeclStmt`, `VisitOMPExecutableDirective`, `getOpenMPCaptureRegions`, `VisitStmt`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 351-375
```cpp
351:           CaptureRegions.back() == OMPD_parallel &&
352:               isOpenMPDistributeDirective(D->getDirectiveKind()));
353:     }
354:   }
355:   void VisitCapturedStmt(const CapturedStmt *S) {
356:     if (!S)
357:       return;
358:     for (const CapturedStmt::Capture &C : S->captures()) {
359:       if (C.capturesVariable() && !C.capturesVariableByCopy()) {
360:         const ValueDecl *VD = C.getCapturedVar();
361:         markAsEscaped(VD);
362:         if (isa<OMPCapturedExprDecl>(VD))
363:           VisitValueDecl(VD);
364:       }
365:     }
366:   }
367:   void VisitLambdaExpr(const LambdaExpr *E) {
368:     if (!E)
369:       return;
370:     for (const LambdaCapture &C : E->captures()) {
371:       if (C.capturesVariable()) {
372:         if (C.getCaptureKind() == LCK_ByRef) {
373:           const ValueDecl *VD = C.getCapturedVar();
374:           markAsEscaped(VD);
375:           if (E->isInitCapture(&C) || isa<OMPCapturedExprDecl>(VD))
```
- **EN**: This block defines callable entry points like `isOpenMPDistributeDirective`, `VisitCapturedStmt`, `markAsEscaped`, `VisitLambdaExpr`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `isOpenMPDistributeDirective`, `VisitCapturedStmt`, `markAsEscaped`, `VisitLambdaExpr`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 376-400
```cpp
376:             VisitValueDecl(VD);
377:         }
378:       }
379:     }
380:   }
381:   void VisitBlockExpr(const BlockExpr *E) {
382:     if (!E)
383:       return;
384:     for (const BlockDecl::Capture &C : E->getBlockDecl()->captures()) {
385:       if (C.isByRef()) {
386:         const VarDecl *VD = C.getVariable();
387:         markAsEscaped(VD);
388:         if (isa<OMPCapturedExprDecl>(VD) || VD->isInitCapture())
389:           VisitValueDecl(VD);
390:       }
391:     }
392:   }
393:   void VisitCallExpr(const CallExpr *E) {
394:     if (!E)
395:       return;
396:     for (const Expr *Arg : E->arguments()) {
397:       if (!Arg)
398:         continue;
399:       if (Arg->isLValue()) {
400:         const bool SavedAllEscaped = AllEscaped;
```
- **EN**: This block defines callable entry points like `VisitValueDecl`, `VisitBlockExpr`, `markAsEscaped`, `VisitCallExpr`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `VisitValueDecl`, `VisitBlockExpr`, `markAsEscaped`, `VisitCallExpr`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 401-425
```cpp
401:         AllEscaped = true;
402:         Visit(Arg);
403:         AllEscaped = SavedAllEscaped;
404:       } else {
405:         Visit(Arg);
406:       }
407:     }
408:     Visit(E->getCallee());
409:   }
410:   void VisitDeclRefExpr(const DeclRefExpr *E) {
411:     if (!E)
412:       return;
413:     const ValueDecl *VD = E->getDecl();
414:     if (AllEscaped)
415:       markAsEscaped(VD);
416:     if (isa<OMPCapturedExprDecl>(VD))
417:       VisitValueDecl(VD);
418:     else if (VD->isInitCapture())
419:       VisitValueDecl(VD);
420:   }
421:   void VisitUnaryOperator(const UnaryOperator *E) {
422:     if (!E)
423:       return;
424:     if (E->getOpcode() == UO_AddrOf) {
425:       const bool SavedAllEscaped = AllEscaped;
```
- **EN**: This block defines callable entry points like `Visit`, `VisitDeclRefExpr`, `VisitUnaryOperator`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Visit`, `VisitDeclRefExpr`, `VisitUnaryOperator`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 426-450
```cpp
426:       AllEscaped = true;
427:       Visit(E->getSubExpr());
428:       AllEscaped = SavedAllEscaped;
429:     } else {
430:       Visit(E->getSubExpr());
431:     }
432:   }
433:   void VisitImplicitCastExpr(const ImplicitCastExpr *E) {
434:     if (!E)
435:       return;
436:     if (E->getCastKind() == CK_ArrayToPointerDecay) {
437:       const bool SavedAllEscaped = AllEscaped;
438:       AllEscaped = true;
439:       Visit(E->getSubExpr());
440:       AllEscaped = SavedAllEscaped;
441:     } else {
442:       Visit(E->getSubExpr());
443:     }
444:   }
445:   void VisitExpr(const Expr *E) {
446:     if (!E)
447:       return;
448:     bool SavedAllEscaped = AllEscaped;
449:     if (!E->isLValue())
450:       AllEscaped = false;
```
- **EN**: This block defines callable entry points like `Visit`, `VisitImplicitCastExpr`, `VisitExpr`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Visit`, `VisitImplicitCastExpr`, `VisitExpr`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 451-475
```cpp
451:     for (const Stmt *Child : E->children())
452:       if (Child)
453:         Visit(Child);
454:     AllEscaped = SavedAllEscaped;
455:   }
456:   void VisitStmt(const Stmt *S) {
457:     if (!S)
458:       return;
459:     for (const Stmt *Child : S->children())
460:       if (Child)
461:         Visit(Child);
462:   }
463: 
464:   /// Returns the record that handles all the escaped local variables and used
465:   /// instead of their original storage.
466:   const RecordDecl *getGlobalizedRecord(bool IsInTTDRegion) {
467:     if (!GlobalizedRD)
468:       buildRecordForGlobalizedVars(IsInTTDRegion);
469:     return GlobalizedRD;
470:   }
471: 
472:   /// Returns the field in the globalized record for the escaped variable.
473:   const FieldDecl *getFieldForGlobalizedVar(const ValueDecl *VD) const {
474:     assert(GlobalizedRD &&
475:            "Record for globalized variables must be generated already.");
```
- **EN**: This block defines callable entry points like `VisitStmt`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `VisitStmt`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 476-500
```cpp
476:     return MappedDeclsFields.lookup(VD);
477:   }
478: 
479:   /// Returns the list of the escaped local variables/parameters.
480:   ArrayRef<const ValueDecl *> getEscapedDecls() const {
481:     return EscapedDecls.getArrayRef();
482:   }
483: 
484:   /// Checks if the escaped local variable is actually a parameter passed by
485:   /// value.
486:   const llvm::SmallPtrSetImpl<const Decl *> &getEscapedParameters() const {
487:     return EscapedParameters;
488:   }
489: 
490:   /// Returns the list of the escaped variables with the variably modified
491:   /// types.
492:   ArrayRef<const ValueDecl *> getEscapedVariableLengthDecls() const {
493:     return EscapedVariableLengthDecls.getArrayRef();
494:   }
495: 
496:   /// Returns the list of the delayed variables with the variably modified
497:   /// types.
498:   ArrayRef<const ValueDecl *> getDelayedVariableLengthDecls() const {
499:     return DelayedVariableLengthDecls.getArrayRef();
500:   }
```
- **EN**: This block defines callable entry points like `getEscapedDecls`, `getEscapedVariableLengthDecls`, `getDelayedVariableLengthDecls`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `getEscapedDecls`, `getEscapedVariableLengthDecls`, `getDelayedVariableLengthDecls`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 501-525
```cpp
501: };
502: } // anonymous namespace
503: 
504: CGOpenMPRuntimeGPU::ExecutionMode
505: CGOpenMPRuntimeGPU::getExecutionMode() const {
506:   return CurrentExecutionMode;
507: }
508: 
509: CGOpenMPRuntimeGPU::DataSharingMode
510: CGOpenMPRuntimeGPU::getDataSharingMode() const {
511:   return CurrentDataSharingMode;
512: }
513: 
514: /// Check for inner (nested) SPMD construct, if any
515: static bool hasNestedSPMDDirective(ASTContext &Ctx,
516:                                    const OMPExecutableDirective &D) {
517:   const auto *CS = D.getInnermostCapturedStmt();
518:   const auto *Body =
519:       CS->getCapturedStmt()->IgnoreContainers(/*IgnoreCaptured=*/true);
520:   const Stmt *ChildStmt = CGOpenMPRuntime::getSingleCompoundChild(Ctx, Body);
521: 
522:   if (const auto *NestedDir =
523:           dyn_cast_or_null<OMPExecutableDirective>(ChildStmt)) {
524:     OpenMPDirectiveKind DKind = NestedDir->getDirectiveKind();
525:     switch (D.getDirectiveKind()) {
```
- **EN**: This block opens or references namespaces `CGOpenMPRuntimeGPU`; defines callable entry points like `getExecutionMode`, `getDataSharingMode`, `hasNestedSPMDDirective`; uses control flow (if, switch) to specialize LLVM IR emission.
- **CN**: 该代码块打开或引用命名空间 `CGOpenMPRuntimeGPU`；定义可调用入口，例如 `getExecutionMode`, `getDataSharingMode`, `hasNestedSPMDDirective`；通过控制流（if, switch）细化 LLVM IR 生成 行为。

### Lines 526-550
```cpp
526:     case OMPD_target:
527:       if (isOpenMPParallelDirective(DKind))
528:         return true;
529:       if (DKind == OMPD_teams) {
530:         Body = NestedDir->getInnermostCapturedStmt()->IgnoreContainers(
531:             /*IgnoreCaptured=*/true);
532:         if (!Body)
533:           return false;
534:         ChildStmt = CGOpenMPRuntime::getSingleCompoundChild(Ctx, Body);
535:         if (const auto *NND =
536:                 dyn_cast_or_null<OMPExecutableDirective>(ChildStmt)) {
537:           DKind = NND->getDirectiveKind();
538:           if (isOpenMPParallelDirective(DKind))
539:             return true;
540:         }
541:       }
542:       return false;
543:     case OMPD_target_teams:
544:       return isOpenMPParallelDirective(DKind);
545:     case OMPD_target_simd:
546:     case OMPD_target_parallel:
547:     case OMPD_target_parallel_for:
548:     case OMPD_target_parallel_for_simd:
549:     case OMPD_target_teams_distribute:
550:     case OMPD_target_teams_distribute_simd:
```
- **EN**: This block defines callable entry points like `isOpenMPParallelDirective`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `isOpenMPParallelDirective`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 551-575
```cpp
551:     case OMPD_target_teams_distribute_parallel_for:
552:     case OMPD_target_teams_distribute_parallel_for_simd:
553:     case OMPD_parallel:
554:     case OMPD_for:
555:     case OMPD_parallel_for:
556:     case OMPD_parallel_master:
557:     case OMPD_parallel_sections:
558:     case OMPD_for_simd:
559:     case OMPD_parallel_for_simd:
560:     case OMPD_cancel:
561:     case OMPD_cancellation_point:
562:     case OMPD_ordered:
563:     case OMPD_threadprivate:
564:     case OMPD_allocate:
565:     case OMPD_task:
566:     case OMPD_simd:
567:     case OMPD_sections:
568:     case OMPD_section:
569:     case OMPD_single:
570:     case OMPD_master:
571:     case OMPD_critical:
572:     case OMPD_taskyield:
573:     case OMPD_barrier:
574:     case OMPD_taskwait:
575:     case OMPD_taskgroup:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 576-600
```cpp
576:     case OMPD_atomic:
577:     case OMPD_flush:
578:     case OMPD_depobj:
579:     case OMPD_scan:
580:     case OMPD_teams:
581:     case OMPD_target_data:
582:     case OMPD_target_exit_data:
583:     case OMPD_target_enter_data:
584:     case OMPD_distribute:
585:     case OMPD_distribute_simd:
586:     case OMPD_distribute_parallel_for:
587:     case OMPD_distribute_parallel_for_simd:
588:     case OMPD_teams_distribute:
589:     case OMPD_teams_distribute_simd:
590:     case OMPD_teams_distribute_parallel_for:
591:     case OMPD_teams_distribute_parallel_for_simd:
592:     case OMPD_target_update:
593:     case OMPD_declare_simd:
594:     case OMPD_declare_variant:
595:     case OMPD_begin_declare_variant:
596:     case OMPD_end_declare_variant:
597:     case OMPD_declare_target:
598:     case OMPD_end_declare_target:
599:     case OMPD_declare_reduction:
600:     case OMPD_declare_mapper:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 601-625
```cpp
601:     case OMPD_taskloop:
602:     case OMPD_taskloop_simd:
603:     case OMPD_master_taskloop:
604:     case OMPD_master_taskloop_simd:
605:     case OMPD_parallel_master_taskloop:
606:     case OMPD_parallel_master_taskloop_simd:
607:     case OMPD_requires:
608:     case OMPD_unknown:
609:     default:
610:       llvm_unreachable("Unexpected directive.");
611:     }
612:   }
613: 
614:   return false;
615: }
616: 
617: static bool supportsSPMDExecutionMode(ASTContext &Ctx,
618:                                       const OMPExecutableDirective &D) {
619:   OpenMPDirectiveKind DirectiveKind = D.getDirectiveKind();
620:   switch (DirectiveKind) {
621:   case OMPD_target:
622:   case OMPD_target_teams:
623:     return hasNestedSPMDDirective(Ctx, D);
624:   case OMPD_target_parallel_loop:
625:   case OMPD_target_parallel:
```
- **EN**: This block defines callable entry points like `supportsSPMDExecutionMode`, `hasNestedSPMDDirective`; uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `supportsSPMDExecutionMode`, `hasNestedSPMDDirective`；通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 626-650
```cpp
626:   case OMPD_target_parallel_for:
627:   case OMPD_target_parallel_for_simd:
628:   case OMPD_target_teams_distribute_parallel_for:
629:   case OMPD_target_teams_distribute_parallel_for_simd:
630:   case OMPD_target_simd:
631:   case OMPD_target_teams_distribute_simd:
632:     return true;
633:   case OMPD_target_teams_distribute:
634:     return false;
635:   case OMPD_target_teams_loop:
636:     // Whether this is true or not depends on how the directive will
637:     // eventually be emitted.
638:     if (auto *TTLD = dyn_cast<OMPTargetTeamsGenericLoopDirective>(&D))
639:       return TTLD->canBeParallelFor();
640:     return false;
641:   case OMPD_parallel:
642:   case OMPD_for:
643:   case OMPD_parallel_for:
644:   case OMPD_parallel_master:
645:   case OMPD_parallel_sections:
646:   case OMPD_for_simd:
647:   case OMPD_parallel_for_simd:
648:   case OMPD_cancel:
649:   case OMPD_cancellation_point:
650:   case OMPD_ordered:
```
- **EN**: This block uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 651-675
```cpp
651:   case OMPD_threadprivate:
652:   case OMPD_allocate:
653:   case OMPD_task:
654:   case OMPD_simd:
655:   case OMPD_sections:
656:   case OMPD_section:
657:   case OMPD_single:
658:   case OMPD_master:
659:   case OMPD_critical:
660:   case OMPD_taskyield:
661:   case OMPD_barrier:
662:   case OMPD_taskwait:
663:   case OMPD_taskgroup:
664:   case OMPD_atomic:
665:   case OMPD_flush:
666:   case OMPD_depobj:
667:   case OMPD_scan:
668:   case OMPD_teams:
669:   case OMPD_target_data:
670:   case OMPD_target_exit_data:
671:   case OMPD_target_enter_data:
672:   case OMPD_distribute:
673:   case OMPD_distribute_simd:
674:   case OMPD_distribute_parallel_for:
675:   case OMPD_distribute_parallel_for_simd:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 676-700
```cpp
676:   case OMPD_teams_distribute:
677:   case OMPD_teams_distribute_simd:
678:   case OMPD_teams_distribute_parallel_for:
679:   case OMPD_teams_distribute_parallel_for_simd:
680:   case OMPD_target_update:
681:   case OMPD_declare_simd:
682:   case OMPD_declare_variant:
683:   case OMPD_begin_declare_variant:
684:   case OMPD_end_declare_variant:
685:   case OMPD_declare_target:
686:   case OMPD_end_declare_target:
687:   case OMPD_declare_reduction:
688:   case OMPD_declare_mapper:
689:   case OMPD_taskloop:
690:   case OMPD_taskloop_simd:
691:   case OMPD_master_taskloop:
692:   case OMPD_master_taskloop_simd:
693:   case OMPD_parallel_master_taskloop:
694:   case OMPD_parallel_master_taskloop_simd:
695:   case OMPD_requires:
696:   case OMPD_unknown:
697:   default:
698:     break;
699:   }
700:   llvm_unreachable(
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 701-725
```cpp
701:       "Unknown programming model for OpenMP directive on NVPTX target.");
702: }
703: 
704: void CGOpenMPRuntimeGPU::emitNonSPMDKernel(const OMPExecutableDirective &D,
705:                                              StringRef ParentName,
706:                                              llvm::Function *&OutlinedFn,
707:                                              llvm::Constant *&OutlinedFnID,
708:                                              bool IsOffloadEntry,
709:                                              const RegionCodeGenTy &CodeGen) {
710:   ExecutionRuntimeModesRAII ModeRAII(CurrentExecutionMode, EM_NonSPMD);
711:   EntryFunctionState EST;
712:   WrapperFunctionsMap.clear();
713: 
714:   [[maybe_unused]] bool IsBareKernel = D.getSingleClause<OMPXBareClause>();
715:   assert(!IsBareKernel && "bare kernel should not be at generic mode");
716: 
717:   // Emit target region as a standalone region.
718:   class NVPTXPrePostActionTy : public PrePostActionTy {
719:     CGOpenMPRuntimeGPU::EntryFunctionState &EST;
720:     const OMPExecutableDirective &D;
721: 
722:   public:
723:     NVPTXPrePostActionTy(CGOpenMPRuntimeGPU::EntryFunctionState &EST,
724:                          const OMPExecutableDirective &D)
725:         : EST(EST), D(D) {}
```
- **EN**: This block introduces declarations such as `NVPTXPrePostActionTy`; defines callable entry points like `emitNonSPMDKernel`, `ModeRAII`, `NVPTXPrePostActionTy`; uses control flow (for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `NVPTXPrePostActionTy` 的声明；定义可调用入口，例如 `emitNonSPMDKernel`, `ModeRAII`, `NVPTXPrePostActionTy`；通过控制流（for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 726-750
```cpp
726:     void Enter(CodeGenFunction &CGF) override {
727:       auto &RT = static_cast<CGOpenMPRuntimeGPU &>(CGF.CGM.getOpenMPRuntime());
728:       RT.emitKernelInit(D, CGF, EST, /* IsSPMD */ false);
729:       // Skip target region initialization.
730:       RT.setLocThreadIdInsertPt(CGF, /*AtCurrentPoint=*/true);
731:     }
732:     void Exit(CodeGenFunction &CGF) override {
733:       auto &RT = static_cast<CGOpenMPRuntimeGPU &>(CGF.CGM.getOpenMPRuntime());
734:       RT.clearLocThreadIdInsertPt(CGF);
735:       RT.emitKernelDeinit(CGF, EST, /* IsSPMD */ false);
736:     }
737:   } Action(EST, D);
738:   CodeGen.setAction(Action);
739:   IsInTTDRegion = true;
740:   emitTargetOutlinedFunctionHelper(D, ParentName, OutlinedFn, OutlinedFnID,
741:                                    IsOffloadEntry, CodeGen);
742:   IsInTTDRegion = false;
743: }
744: 
745: void CGOpenMPRuntimeGPU::emitKernelInit(const OMPExecutableDirective &D,
746:                                         CodeGenFunction &CGF,
747:                                         EntryFunctionState &EST, bool IsSPMD) {
748:   llvm::OpenMPIRBuilder::TargetKernelDefaultAttrs Attrs;
749:   Attrs.ExecFlags =
750:       IsSPMD ? llvm::omp::OMPTgtExecModeFlags::OMP_TGT_EXEC_MODE_SPMD
```
- **EN**: This block defines callable entry points like `Enter`, `Exit`, `emitTargetOutlinedFunctionHelper`, `emitKernelInit`.
- **CN**: 该代码块定义可调用入口，例如 `Enter`, `Exit`, `emitTargetOutlinedFunctionHelper`, `emitKernelInit`。

### Lines 751-775
```cpp
751:              : llvm::omp::OMPTgtExecModeFlags::OMP_TGT_EXEC_MODE_GENERIC;
752:   computeMinAndMaxThreadsAndTeams(D, CGF, Attrs);
753: 
754:   CGBuilderTy &Bld = CGF.Builder;
755:   Bld.restoreIP(OMPBuilder.createTargetInit(Bld, Attrs));
756:   if (!IsSPMD)
757:     emitGenericVarsProlog(CGF, EST.Loc);
758: }
759: 
760: void CGOpenMPRuntimeGPU::emitKernelDeinit(CodeGenFunction &CGF,
761:                                           EntryFunctionState &EST,
762:                                           bool IsSPMD) {
763:   if (!IsSPMD)
764:     emitGenericVarsEpilog(CGF);
765: 
766:   // This is temporary until we remove the fixed sized buffer.
767:   ASTContext &C = CGM.getContext();
768:   RecordDecl *StaticRD = C.buildImplicitRecord(
769:       "_openmp_teams_reduction_type_$_", RecordDecl::TagKind::Union);
770:   StaticRD->startDefinition();
771:   for (const RecordDecl *TeamReductionRec : TeamsReductions) {
772:     CanQualType RecTy = C.getCanonicalTagType(TeamReductionRec);
773:     auto *Field = FieldDecl::Create(
774:         C, StaticRD, SourceLocation(), SourceLocation(), nullptr, RecTy,
775:         C.getTrivialTypeSourceInfo(RecTy, SourceLocation()),
```
- **EN**: This block defines callable entry points like `computeMinAndMaxThreadsAndTeams`, `emitKernelDeinit`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `computeMinAndMaxThreadsAndTeams`, `emitKernelDeinit`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 776-800
```cpp
776:         /*BW=*/nullptr, /*Mutable=*/false,
777:         /*InitStyle=*/ICIS_NoInit);
778:     Field->setAccess(AS_public);
779:     StaticRD->addDecl(Field);
780:   }
781:   StaticRD->completeDefinition();
782:   CanQualType StaticTy = C.getCanonicalTagType(StaticRD);
783:   llvm::Type *LLVMReductionsBufferTy =
784:       CGM.getTypes().ConvertTypeForMem(StaticTy);
785:   const auto &DL = CGM.getModule().getDataLayout();
786:   uint64_t ReductionDataSize =
787:       TeamsReductions.empty()
788:           ? 0
789:           : DL.getTypeAllocSize(LLVMReductionsBufferTy).getFixedValue();
790:   CGBuilderTy &Bld = CGF.Builder;
791:   OMPBuilder.createTargetDeinit(Bld, ReductionDataSize,
792:                                 C.getLangOpts().OpenMPCUDAReductionBufNum);
793:   TeamsReductions.clear();
794: }
795: 
796: void CGOpenMPRuntimeGPU::emitSPMDKernel(const OMPExecutableDirective &D,
797:                                           StringRef ParentName,
798:                                           llvm::Function *&OutlinedFn,
799:                                           llvm::Constant *&OutlinedFnID,
800:                                           bool IsOffloadEntry,
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 801-825
```cpp
801:                                           const RegionCodeGenTy &CodeGen) {
802:   ExecutionRuntimeModesRAII ModeRAII(CurrentExecutionMode, EM_SPMD);
803:   EntryFunctionState EST;
804: 
805:   bool IsBareKernel = D.getSingleClause<OMPXBareClause>();
806: 
807:   // Emit target region as a standalone region.
808:   class NVPTXPrePostActionTy : public PrePostActionTy {
809:     CGOpenMPRuntimeGPU &RT;
810:     CGOpenMPRuntimeGPU::EntryFunctionState &EST;
811:     bool IsBareKernel;
812:     DataSharingMode Mode;
813:     const OMPExecutableDirective &D;
814: 
815:   public:
816:     NVPTXPrePostActionTy(CGOpenMPRuntimeGPU &RT,
817:                          CGOpenMPRuntimeGPU::EntryFunctionState &EST,
818:                          bool IsBareKernel, const OMPExecutableDirective &D)
819:         : RT(RT), EST(EST), IsBareKernel(IsBareKernel),
820:           Mode(RT.CurrentDataSharingMode), D(D) {}
821:     void Enter(CodeGenFunction &CGF) override {
822:       if (IsBareKernel) {
823:         RT.CurrentDataSharingMode = DataSharingMode::DS_CUDA;
824:         return;
825:       }
```
- **EN**: This block introduces declarations such as `NVPTXPrePostActionTy`; defines callable entry points like `ModeRAII`, `NVPTXPrePostActionTy`, `Enter`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块给出诸如 `NVPTXPrePostActionTy` 的声明；定义可调用入口，例如 `ModeRAII`, `NVPTXPrePostActionTy`, `Enter`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 826-850
```cpp
826:       RT.emitKernelInit(D, CGF, EST, /* IsSPMD */ true);
827:       // Skip target region initialization.
828:       RT.setLocThreadIdInsertPt(CGF, /*AtCurrentPoint=*/true);
829:     }
830:     void Exit(CodeGenFunction &CGF) override {
831:       if (IsBareKernel) {
832:         RT.CurrentDataSharingMode = Mode;
833:         return;
834:       }
835:       RT.clearLocThreadIdInsertPt(CGF);
836:       RT.emitKernelDeinit(CGF, EST, /* IsSPMD */ true);
837:     }
838:   } Action(*this, EST, IsBareKernel, D);
839:   CodeGen.setAction(Action);
840:   IsInTTDRegion = true;
841:   emitTargetOutlinedFunctionHelper(D, ParentName, OutlinedFn, OutlinedFnID,
842:                                    IsOffloadEntry, CodeGen);
843:   IsInTTDRegion = false;
844: }
845: 
846: void CGOpenMPRuntimeGPU::emitTargetOutlinedFunction(
847:     const OMPExecutableDirective &D, StringRef ParentName,
848:     llvm::Function *&OutlinedFn, llvm::Constant *&OutlinedFnID,
849:     bool IsOffloadEntry, const RegionCodeGenTy &CodeGen) {
850:   if (!IsOffloadEntry) // Nothing to do.
```
- **EN**: This block defines callable entry points like `Exit`, `emitTargetOutlinedFunctionHelper`, `emitTargetOutlinedFunction`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Exit`, `emitTargetOutlinedFunctionHelper`, `emitTargetOutlinedFunction`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 851-875
```cpp
851:     return;
852: 
853:   assert(!ParentName.empty() && "Invalid target region parent name!");
854: 
855:   bool Mode = supportsSPMDExecutionMode(CGM.getContext(), D);
856:   bool IsBareKernel = D.getSingleClause<OMPXBareClause>();
857:   if (Mode || IsBareKernel)
858:     emitSPMDKernel(D, ParentName, OutlinedFn, OutlinedFnID, IsOffloadEntry,
859:                    CodeGen);
860:   else
861:     emitNonSPMDKernel(D, ParentName, OutlinedFn, OutlinedFnID, IsOffloadEntry,
862:                       CodeGen);
863: }
864: 
865: CGOpenMPRuntimeGPU::CGOpenMPRuntimeGPU(CodeGenModule &CGM)
866:     : CGOpenMPRuntime(CGM) {
867:   llvm::OpenMPIRBuilderConfig Config(
868:       CGM.getLangOpts().OpenMPIsTargetDevice, isGPU(),
869:       CGM.getLangOpts().OpenMPOffloadMandatory,
870:       /*HasRequiresReverseOffload*/ false, /*HasRequiresUnifiedAddress*/ false,
871:       hasRequiresUnifiedSharedMemory(), /*HasRequiresDynamicAllocators*/ false);
872:   Config.setDefaultTargetAS(
873:       CGM.getContext().getTargetInfo().getTargetAddressSpace(LangAS::Default));
874:   Config.setRuntimeCC(CGM.getRuntimeCC());
875: 
```
- **EN**: This block defines callable entry points like `emitNonSPMDKernel`, `CGOpenMPRuntimeGPU`, `Config`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitNonSPMDKernel`, `CGOpenMPRuntimeGPU`, `Config`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 876-900
```cpp
876:   OMPBuilder.setConfig(Config);
877: 
878:   if (!CGM.getLangOpts().OpenMPIsTargetDevice)
879:     llvm_unreachable("OpenMP can only handle device code.");
880: 
881:   if (CGM.getLangOpts().OpenMPCUDAMode)
882:     CurrentDataSharingMode = CGOpenMPRuntimeGPU::DS_CUDA;
883: 
884:   llvm::OpenMPIRBuilder &OMPBuilder = getOMPBuilder();
885:   if (CGM.getLangOpts().NoGPULib || CGM.getLangOpts().OMPHostIRFile.empty())
886:     return;
887: 
888:   OMPBuilder.createGlobalFlag(CGM.getLangOpts().OpenMPTargetDebug,
889:                               "__omp_rtl_debug_kind");
890:   OMPBuilder.createGlobalFlag(CGM.getLangOpts().OpenMPTeamSubscription,
891:                               "__omp_rtl_assume_teams_oversubscription");
892:   OMPBuilder.createGlobalFlag(CGM.getLangOpts().OpenMPThreadSubscription,
893:                               "__omp_rtl_assume_threads_oversubscription");
894:   OMPBuilder.createGlobalFlag(CGM.getLangOpts().OpenMPNoThreadState,
895:                               "__omp_rtl_assume_no_thread_state");
896:   OMPBuilder.createGlobalFlag(CGM.getLangOpts().OpenMPNoNestedParallelism,
897:                               "__omp_rtl_assume_no_nested_parallelism");
898: }
899: 
900: void CGOpenMPRuntimeGPU::emitProcBindClause(CodeGenFunction &CGF,
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 901-925
```cpp
901:                                               ProcBindKind ProcBind,
902:                                               SourceLocation Loc) {
903:   // Nothing to do.
904: }
905: 
906: llvm::Value *CGOpenMPRuntimeGPU::emitMessageClause(CodeGenFunction &CGF,
907:                                                    const Expr *Message,
908:                                                    SourceLocation Loc) {
909:   CGM.getDiags().Report(Loc, diag::warn_omp_gpu_unsupported_clause)
910:       << getOpenMPClauseName(OMPC_message);
911:   return nullptr;
912: }
913: 
914: llvm::Value *
915: CGOpenMPRuntimeGPU::emitSeverityClause(OpenMPSeverityClauseKind Severity,
916:                                        SourceLocation Loc) {
917:   CGM.getDiags().Report(Loc, diag::warn_omp_gpu_unsupported_clause)
918:       << getOpenMPClauseName(OMPC_severity);
919:   return nullptr;
920: }
921: 
922: void CGOpenMPRuntimeGPU::emitNumThreadsClause(
923:     CodeGenFunction &CGF, llvm::Value *NumThreads, SourceLocation Loc,
924:     OpenMPNumThreadsClauseModifier Modifier, OpenMPSeverityClauseKind Severity,
925:     SourceLocation SeverityLoc, const Expr *Message,
```
- **EN**: This block defines callable entry points like `getOpenMPClauseName`, `emitSeverityClause`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `getOpenMPClauseName`, `emitSeverityClause`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 926-950
```cpp
926:     SourceLocation MessageLoc) {
927:   if (Modifier == OMPC_NUMTHREADS_strict) {
928:     CGM.getDiags().Report(Loc,
929:                           diag::warn_omp_gpu_unsupported_modifier_for_clause)
930:         << "strict" << getOpenMPClauseName(OMPC_num_threads);
931:     return;
932:   }
933: 
934:   // Nothing to do.
935: }
936: 
937: void CGOpenMPRuntimeGPU::emitNumTeamsClause(CodeGenFunction &CGF,
938:                                               const Expr *NumTeams,
939:                                               const Expr *ThreadLimit,
940:                                               SourceLocation Loc) {}
941: 
942: llvm::Function *CGOpenMPRuntimeGPU::emitParallelOutlinedFunction(
943:     CodeGenFunction &CGF, const OMPExecutableDirective &D,
944:     const VarDecl *ThreadIDVar, OpenMPDirectiveKind InnermostKind,
945:     const RegionCodeGenTy &CodeGen) {
946:   // Emit target region as a standalone region.
947:   bool PrevIsInTTDRegion = IsInTTDRegion;
948:   IsInTTDRegion = false;
949:   auto *OutlinedFun =
950:       cast<llvm::Function>(CGOpenMPRuntime::emitParallelOutlinedFunction(
```
- **EN**: This block defines callable entry points like `emitNumTeamsClause`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitNumTeamsClause`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 951-975
```cpp
951:           CGF, D, ThreadIDVar, InnermostKind, CodeGen));
952:   IsInTTDRegion = PrevIsInTTDRegion;
953:   if (getExecutionMode() != CGOpenMPRuntimeGPU::EM_SPMD) {
954:     llvm::Function *WrapperFun =
955:         createParallelDataSharingWrapper(OutlinedFun, D);
956:     WrapperFunctionsMap[OutlinedFun] = WrapperFun;
957:   }
958: 
959:   return OutlinedFun;
960: }
961: 
962: /// Get list of lastprivate variables from the teams distribute ... or
963: /// teams {distribute ...} directives.
964: static void
965: getDistributeLastprivateVars(ASTContext &Ctx, const OMPExecutableDirective &D,
966:                              llvm::SmallVectorImpl<const ValueDecl *> &Vars) {
967:   assert(isOpenMPTeamsDirective(D.getDirectiveKind()) &&
968:          "expected teams directive.");
969:   const OMPExecutableDirective *Dir = &D;
970:   if (!isOpenMPDistributeDirective(D.getDirectiveKind())) {
971:     if (const Stmt *S = CGOpenMPRuntime::getSingleCompoundChild(
972:             Ctx,
973:             D.getInnermostCapturedStmt()->getCapturedStmt()->IgnoreContainers(
974:                 /*IgnoreCaptured=*/true))) {
975:       Dir = dyn_cast_or_null<OMPExecutableDirective>(S);
```
- **EN**: This block defines callable entry points like `createParallelDataSharingWrapper`, `getDistributeLastprivateVars`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `createParallelDataSharingWrapper`, `getDistributeLastprivateVars`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 976-1000
```cpp
 976:       if (Dir && !isOpenMPDistributeDirective(Dir->getDirectiveKind()))
 977:         Dir = nullptr;
 978:     }
 979:   }
 980:   if (!Dir)
 981:     return;
 982:   for (const auto *C : Dir->getClausesOfKind<OMPLastprivateClause>()) {
 983:     for (const Expr *E : C->getVarRefs())
 984:       Vars.push_back(getPrivateItem(E));
 985:   }
 986: }
 987: 
 988: /// Get list of reduction variables from the teams ... directives.
 989: static void
 990: getTeamsReductionVars(ASTContext &Ctx, const OMPExecutableDirective &D,
 991:                       llvm::SmallVectorImpl<const ValueDecl *> &Vars) {
 992:   assert(isOpenMPTeamsDirective(D.getDirectiveKind()) &&
 993:          "expected teams directive.");
 994:   for (const auto *C : D.getClausesOfKind<OMPReductionClause>()) {
 995:     for (const Expr *E : C->privates())
 996:       Vars.push_back(getPrivateItem(E));
 997:   }
 998: }
 999: 
1000: llvm::Function *CGOpenMPRuntimeGPU::emitTeamsOutlinedFunction(
```
- **EN**: This block defines callable entry points like `getTeamsReductionVars`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getTeamsReductionVars`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1001-1025
```cpp
1001:     CodeGenFunction &CGF, const OMPExecutableDirective &D,
1002:     const VarDecl *ThreadIDVar, OpenMPDirectiveKind InnermostKind,
1003:     const RegionCodeGenTy &CodeGen) {
1004:   SourceLocation Loc = D.getBeginLoc();
1005: 
1006:   const RecordDecl *GlobalizedRD = nullptr;
1007:   llvm::SmallVector<const ValueDecl *, 4> LastPrivatesReductions;
1008:   llvm::SmallDenseMap<const ValueDecl *, const FieldDecl *> MappedDeclsFields;
1009:   unsigned WarpSize = CGM.getTarget().getGridValue().GV_Warp_Size;
1010:   // Globalize team reductions variable unconditionally in all modes.
1011:   if (getExecutionMode() != CGOpenMPRuntimeGPU::EM_SPMD)
1012:     getTeamsReductionVars(CGM.getContext(), D, LastPrivatesReductions);
1013:   if (getExecutionMode() == CGOpenMPRuntimeGPU::EM_SPMD) {
1014:     getDistributeLastprivateVars(CGM.getContext(), D, LastPrivatesReductions);
1015:     if (!LastPrivatesReductions.empty()) {
1016:       GlobalizedRD = ::buildRecordForGlobalizedVars(
1017:           CGM.getContext(), {}, LastPrivatesReductions, MappedDeclsFields,
1018:           WarpSize);
1019:     }
1020:   } else if (!LastPrivatesReductions.empty()) {
1021:     assert(!TeamAndReductions.first &&
1022:            "Previous team declaration is not expected.");
1023:     TeamAndReductions.first = D.getCapturedStmt(OMPD_teams)->getCapturedDecl();
1024:     std::swap(TeamAndReductions.second, LastPrivatesReductions);
1025:   }
```
- **EN**: This block defines callable entry points like `getDistributeLastprivateVars`, `swap`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getDistributeLastprivateVars`, `swap`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1026-1050
```cpp
1026: 
1027:   // Emit target region as a standalone region.
1028:   class NVPTXPrePostActionTy : public PrePostActionTy {
1029:     SourceLocation &Loc;
1030:     const RecordDecl *GlobalizedRD;
1031:     llvm::SmallDenseMap<const ValueDecl *, const FieldDecl *>
1032:         &MappedDeclsFields;
1033: 
1034:   public:
1035:     NVPTXPrePostActionTy(
1036:         SourceLocation &Loc, const RecordDecl *GlobalizedRD,
1037:         llvm::SmallDenseMap<const ValueDecl *, const FieldDecl *>
1038:             &MappedDeclsFields)
1039:         : Loc(Loc), GlobalizedRD(GlobalizedRD),
1040:           MappedDeclsFields(MappedDeclsFields) {}
1041:     void Enter(CodeGenFunction &CGF) override {
1042:       auto &Rt =
1043:           static_cast<CGOpenMPRuntimeGPU &>(CGF.CGM.getOpenMPRuntime());
1044:       if (GlobalizedRD) {
1045:         auto I = Rt.FunctionGlobalizedDecls.try_emplace(CGF.CurFn).first;
1046:         I->getSecond().MappedParams =
1047:             std::make_unique<CodeGenFunction::OMPMapVars>();
1048:         DeclToAddrMapTy &Data = I->getSecond().LocalVarData;
1049:         for (const auto &Pair : MappedDeclsFields) {
1050:           assert(Pair.getFirst()->isCanonicalDecl() &&
```
- **EN**: This block introduces declarations such as `NVPTXPrePostActionTy`; defines callable entry points like `NVPTXPrePostActionTy`, `Enter`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `NVPTXPrePostActionTy` 的声明；定义可调用入口，例如 `NVPTXPrePostActionTy`, `Enter`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1051-1075
```cpp
1051:                  "Expected canonical declaration");
1052:           Data.try_emplace(Pair.getFirst());
1053:         }
1054:       }
1055:       Rt.emitGenericVarsProlog(CGF, Loc);
1056:     }
1057:     void Exit(CodeGenFunction &CGF) override {
1058:       static_cast<CGOpenMPRuntimeGPU &>(CGF.CGM.getOpenMPRuntime())
1059:           .emitGenericVarsEpilog(CGF);
1060:     }
1061:   } Action(Loc, GlobalizedRD, MappedDeclsFields);
1062:   CodeGen.setAction(Action);
1063:   llvm::Function *OutlinedFun = CGOpenMPRuntime::emitTeamsOutlinedFunction(
1064:       CGF, D, ThreadIDVar, InnermostKind, CodeGen);
1065: 
1066:   return OutlinedFun;
1067: }
1068: 
1069: void CGOpenMPRuntimeGPU::emitGenericVarsProlog(CodeGenFunction &CGF,
1070:                                                SourceLocation Loc) {
1071:   if (getDataSharingMode() != CGOpenMPRuntimeGPU::DS_Generic)
1072:     return;
1073: 
1074:   CGBuilderTy &Bld = CGF.Builder;
1075: 
```
- **EN**: This block defines callable entry points like `Exit`, `emitGenericVarsProlog`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Exit`, `emitGenericVarsProlog`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1076-1100
```cpp
1076:   const auto I = FunctionGlobalizedDecls.find(CGF.CurFn);
1077:   if (I == FunctionGlobalizedDecls.end())
1078:     return;
1079: 
1080:   for (auto &Rec : I->getSecond().LocalVarData) {
1081:     const auto *VD = cast<VarDecl>(Rec.first);
1082:     bool EscapedParam = I->getSecond().EscapedParameters.count(Rec.first);
1083:     QualType VarTy = VD->getType();
1084: 
1085:     // Get the local allocation of a firstprivate variable before sharing
1086:     llvm::Value *ParValue;
1087:     if (EscapedParam) {
1088:       LValue ParLVal =
1089:           CGF.MakeAddrLValue(CGF.GetAddrOfLocalVar(VD), VD->getType());
1090:       ParValue = CGF.EmitLoadOfScalar(ParLVal, Loc);
1091:     }
1092: 
1093:     // Allocate space for the variable to be globalized
1094:     llvm::Value *AllocArgs[] = {CGF.getTypeSize(VD->getType())};
1095:     llvm::CallBase *VoidPtr =
1096:         CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1097:                                 CGM.getModule(), OMPRTL___kmpc_alloc_shared),
1098:                             AllocArgs, VD->getName());
1099:     // FIXME: We should use the variables actual alignment as an argument.
1100:     VoidPtr->addRetAttr(llvm::Attribute::get(
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1101-1125
```cpp
1101:         CGM.getLLVMContext(), llvm::Attribute::Alignment,
1102:         CGM.getContext().getTargetInfo().getNewAlign() / 8));
1103: 
1104:     // Cast the void pointer and get the address of the globalized variable.
1105:     llvm::Value *CastedVoidPtr = Bld.CreatePointerBitCastOrAddrSpaceCast(
1106:         VoidPtr, Bld.getPtrTy(0), VD->getName() + "_on_stack");
1107:     LValue VarAddr =
1108:         CGF.MakeNaturalAlignPointeeRawAddrLValue(CastedVoidPtr, VarTy);
1109:     Rec.second.PrivateAddr = VarAddr.getAddress();
1110:     Rec.second.GlobalizedVal = VoidPtr;
1111: 
1112:     // Assign the local allocation to the newly globalized location.
1113:     if (EscapedParam) {
1114:       CGF.EmitStoreOfScalar(ParValue, VarAddr);
1115:       I->getSecond().MappedParams->setVarAddr(CGF, VD, VarAddr.getAddress());
1116:     }
1117:     if (auto *DI = CGF.getDebugInfo())
1118:       VoidPtr->setDebugLoc(DI->SourceLocToDebugLoc(VD->getLocation()));
1119:   }
1120: 
1121:   for (const auto *ValueD : I->getSecond().EscapedVariableLengthDecls) {
1122:     const auto *VD = cast<VarDecl>(ValueD);
1123:     std::pair<llvm::Value *, llvm::Value *> AddrSizePair =
1124:         getKmpcAllocShared(CGF, VD);
1125:     I->getSecond().EscapedVariableLengthDeclsAddrs.emplace_back(AddrSizePair);
```
- **EN**: This block defines callable entry points like `getKmpcAllocShared`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getKmpcAllocShared`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1126-1150
```cpp
1126:     LValue Base = CGF.MakeAddrLValue(AddrSizePair.first, VD->getType(),
1127:                                      CGM.getContext().getDeclAlign(VD),
1128:                                      AlignmentSource::Decl);
1129:     I->getSecond().MappedParams->setVarAddr(CGF, VD, Base.getAddress());
1130:   }
1131:   I->getSecond().MappedParams->apply(CGF);
1132: }
1133: 
1134: bool CGOpenMPRuntimeGPU::isDelayedVariableLengthDecl(CodeGenFunction &CGF,
1135:                                                      const VarDecl *VD) const {
1136:   const auto I = FunctionGlobalizedDecls.find(CGF.CurFn);
1137:   if (I == FunctionGlobalizedDecls.end())
1138:     return false;
1139: 
1140:   // Check variable declaration is delayed:
1141:   return llvm::is_contained(I->getSecond().DelayedVariableLengthDecls, VD);
1142: }
1143: 
1144: std::pair<llvm::Value *, llvm::Value *>
1145: CGOpenMPRuntimeGPU::getKmpcAllocShared(CodeGenFunction &CGF,
1146:                                        const VarDecl *VD) {
1147:   CGBuilderTy &Bld = CGF.Builder;
1148: 
1149:   // Compute size and alignment.
1150:   llvm::Value *Size = CGF.getTypeSize(VD->getType());
```
- **EN**: This block defines callable entry points like `isDelayedVariableLengthDecl`, `is_contained`, `getKmpcAllocShared`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `isDelayedVariableLengthDecl`, `is_contained`, `getKmpcAllocShared`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1151-1175
```cpp
1151:   CharUnits Align = CGM.getContext().getDeclAlign(VD);
1152:   Size = Bld.CreateNUWAdd(
1153:       Size, llvm::ConstantInt::get(CGF.SizeTy, Align.getQuantity() - 1));
1154:   llvm::Value *AlignVal =
1155:       llvm::ConstantInt::get(CGF.SizeTy, Align.getQuantity());
1156:   Size = Bld.CreateUDiv(Size, AlignVal);
1157:   Size = Bld.CreateNUWMul(Size, AlignVal);
1158: 
1159:   // Allocate space for this VLA object to be globalized.
1160:   llvm::Value *AllocArgs[] = {Size};
1161:   llvm::CallBase *VoidPtr =
1162:       CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1163:                               CGM.getModule(), OMPRTL___kmpc_alloc_shared),
1164:                           AllocArgs, VD->getName());
1165:   VoidPtr->addRetAttr(llvm::Attribute::get(
1166:       CGM.getLLVMContext(), llvm::Attribute::Alignment, Align.getQuantity()));
1167: 
1168:   return std::make_pair(VoidPtr, Size);
1169: }
1170: 
1171: void CGOpenMPRuntimeGPU::getKmpcFreeShared(
1172:     CodeGenFunction &CGF,
1173:     const std::pair<llvm::Value *, llvm::Value *> &AddrSizePair) {
1174:   // Deallocate the memory for each globalized VLA object
1175:   CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
```
- **EN**: This block defines callable entry points like `get`, `make_pair`, `getKmpcFreeShared`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `get`, `make_pair`, `getKmpcFreeShared`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 1176-1200
```cpp
1176:                           CGM.getModule(), OMPRTL___kmpc_free_shared),
1177:                       {AddrSizePair.first, AddrSizePair.second});
1178: }
1179: 
1180: void CGOpenMPRuntimeGPU::emitGenericVarsEpilog(CodeGenFunction &CGF) {
1181:   if (getDataSharingMode() != CGOpenMPRuntimeGPU::DS_Generic)
1182:     return;
1183: 
1184:   const auto I = FunctionGlobalizedDecls.find(CGF.CurFn);
1185:   if (I != FunctionGlobalizedDecls.end()) {
1186:     // Deallocate the memory for each globalized VLA object that was
1187:     // globalized in the prolog (i.e. emitGenericVarsProlog).
1188:     for (const auto &AddrSizePair :
1189:          llvm::reverse(I->getSecond().EscapedVariableLengthDeclsAddrs)) {
1190:       CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1191:                               CGM.getModule(), OMPRTL___kmpc_free_shared),
1192:                           {AddrSizePair.first, AddrSizePair.second});
1193:     }
1194:     // Deallocate the memory for each globalized value
1195:     for (auto &Rec : llvm::reverse(I->getSecond().LocalVarData)) {
1196:       const auto *VD = cast<VarDecl>(Rec.first);
1197:       I->getSecond().MappedParams->restore(CGF);
1198: 
1199:       llvm::Value *FreeArgs[] = {Rec.second.GlobalizedVal,
1200:                                  CGF.getTypeSize(VD->getType())};
```
- **EN**: This block defines callable entry points like `emitGenericVarsEpilog`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitGenericVarsEpilog`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1201-1225
```cpp
1201:       CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1202:                               CGM.getModule(), OMPRTL___kmpc_free_shared),
1203:                           FreeArgs);
1204:     }
1205:   }
1206: }
1207: 
1208: void CGOpenMPRuntimeGPU::emitTeamsCall(CodeGenFunction &CGF,
1209:                                          const OMPExecutableDirective &D,
1210:                                          SourceLocation Loc,
1211:                                          llvm::Function *OutlinedFn,
1212:                                          ArrayRef<llvm::Value *> CapturedVars) {
1213:   if (!CGF.HaveInsertPoint())
1214:     return;
1215: 
1216:   bool IsBareKernel = D.getSingleClause<OMPXBareClause>();
1217: 
1218:   RawAddress ZeroAddr = CGF.CreateDefaultAlignTempAlloca(CGF.Int32Ty,
1219:                                                          /*Name=*/".zero.addr");
1220:   CGF.Builder.CreateStore(CGF.Builder.getInt32(/*C*/ 0), ZeroAddr);
1221:   llvm::SmallVector<llvm::Value *, 16> OutlinedFnArgs;
1222:   // We don't emit any thread id function call in bare kernel, but because the
1223:   // outlined function has a pointer argument, we emit a nullptr here.
1224:   if (IsBareKernel)
1225:     OutlinedFnArgs.push_back(llvm::ConstantPointerNull::get(CGM.VoidPtrTy));
```
- **EN**: This block defines callable entry points like `emitTeamsCall`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitTeamsCall`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1226-1250
```cpp
1226:   else
1227:     OutlinedFnArgs.push_back(emitThreadIDAddress(CGF, Loc).emitRawPointer(CGF));
1228:   OutlinedFnArgs.push_back(ZeroAddr.getPointer());
1229:   OutlinedFnArgs.append(CapturedVars.begin(), CapturedVars.end());
1230:   emitOutlinedFunctionCall(CGF, Loc, OutlinedFn, OutlinedFnArgs);
1231: }
1232: 
1233: void CGOpenMPRuntimeGPU::emitParallelCall(
1234:     CodeGenFunction &CGF, SourceLocation Loc, llvm::Function *OutlinedFn,
1235:     ArrayRef<llvm::Value *> CapturedVars, const Expr *IfCond,
1236:     llvm::Value *NumThreads, OpenMPNumThreadsClauseModifier NumThreadsModifier,
1237:     OpenMPSeverityClauseKind Severity, const Expr *Message) {
1238:   if (!CGF.HaveInsertPoint())
1239:     return;
1240: 
1241:   auto &&ParallelGen = [this, Loc, OutlinedFn, CapturedVars, IfCond,
1242:                         NumThreads](CodeGenFunction &CGF,
1243:                                     PrePostActionTy &Action) {
1244:     CGBuilderTy &Bld = CGF.Builder;
1245:     llvm::Value *NumThreadsVal = NumThreads;
1246:     llvm::Function *WFn = WrapperFunctionsMap[OutlinedFn];
1247:     llvm::PointerType *FnPtrTy = llvm::PointerType::get(
1248:         CGF.getLLVMContext(), CGM.getDataLayout().getProgramAddressSpace());
1249: 
1250:     llvm::Value *ID = llvm::ConstantPointerNull::get(FnPtrTy);
```
- **EN**: This block defines callable entry points like `emitOutlinedFunctionCall`, `emitParallelCall`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitOutlinedFunctionCall`, `emitParallelCall`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1251-1275
```cpp
1251:     if (WFn)
1252:       ID = Bld.CreateBitOrPointerCast(WFn, FnPtrTy);
1253: 
1254:     llvm::Value *FnPtr = Bld.CreateBitOrPointerCast(OutlinedFn, FnPtrTy);
1255: 
1256:     // Create a private scope that will globalize the arguments
1257:     // passed from the outside of the target region.
1258:     // TODO: Is that needed?
1259:     CodeGenFunction::OMPPrivateScope PrivateArgScope(CGF);
1260: 
1261:     Address CapturedVarsAddrs = CGF.CreateDefaultAlignTempAlloca(
1262:         llvm::ArrayType::get(CGM.VoidPtrTy, CapturedVars.size()),
1263:         "captured_vars_addrs");
1264:     // There's something to share.
1265:     if (!CapturedVars.empty()) {
1266:       // Prepare for parallel region. Indicate the outlined function.
1267:       ASTContext &Ctx = CGF.getContext();
1268:       unsigned Idx = 0;
1269:       for (llvm::Value *V : CapturedVars) {
1270:         Address Dst = Bld.CreateConstArrayGEP(CapturedVarsAddrs, Idx);
1271:         llvm::Value *PtrV;
1272:         if (V->getType()->isIntegerTy())
1273:           PtrV = Bld.CreateIntToPtr(V, CGF.VoidPtrTy);
1274:         else
1275:           PtrV = Bld.CreatePointerBitCastOrAddrSpaceCast(V, CGF.VoidPtrTy);
```
- **EN**: This block defines callable entry points like `PrivateArgScope`, `get`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `PrivateArgScope`, `get`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1276-1300
```cpp
1276:         CGF.EmitStoreOfScalar(PtrV, Dst, /*Volatile=*/false,
1277:                               Ctx.getPointerType(Ctx.VoidPtrTy));
1278:         ++Idx;
1279:       }
1280:     }
1281: 
1282:     llvm::Value *IfCondVal = nullptr;
1283:     if (IfCond)
1284:       IfCondVal = Bld.CreateIntCast(CGF.EvaluateExprAsBool(IfCond), CGF.Int32Ty,
1285:                                     /* isSigned */ false);
1286:     else
1287:       IfCondVal = llvm::ConstantInt::get(CGF.Int32Ty, 1);
1288: 
1289:     if (!NumThreadsVal)
1290:       NumThreadsVal = llvm::ConstantInt::getAllOnesValue(CGF.Int32Ty);
1291:     else
1292:       NumThreadsVal = Bld.CreateZExtOrTrunc(NumThreadsVal, CGF.Int32Ty);
1293: 
1294:     // No strict prescriptiveness for the number of threads.
1295:     llvm::Value *StrictNumThreadsVal = llvm::ConstantInt::get(CGF.Int32Ty, 0);
1296: 
1297:     assert(IfCondVal && "Expected a value");
1298:     llvm::Value *RTLoc = emitUpdateLocation(CGF, Loc);
1299:     llvm::Value *Args[] = {
1300:         RTLoc,
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1301-1325
```cpp
1301:         getThreadID(CGF, Loc),
1302:         IfCondVal,
1303:         NumThreadsVal,
1304:         llvm::ConstantInt::getAllOnesValue(CGF.Int32Ty),
1305:         FnPtr,
1306:         ID,
1307:         Bld.CreateBitOrPointerCast(CapturedVarsAddrs.emitRawPointer(CGF),
1308:                                    CGF.VoidPtrPtrTy),
1309:         llvm::ConstantInt::get(CGM.SizeTy, CapturedVars.size()),
1310:         StrictNumThreadsVal};
1311: 
1312:     CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1313:                             CGM.getModule(), OMPRTL___kmpc_parallel_60),
1314:                         Args);
1315:   };
1316: 
1317:   RegionCodeGenTy RCG(ParallelGen);
1318:   RCG(CGF);
1319: }
1320: 
1321: void CGOpenMPRuntimeGPU::syncCTAThreads(CodeGenFunction &CGF) {
1322:   // Always emit simple barriers!
1323:   if (!CGF.HaveInsertPoint())
1324:     return;
1325:   // Build call __kmpc_barrier_simple_spmd(nullptr, 0);
```
- **EN**: This block defines callable entry points like `RCG`, `syncCTAThreads`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `RCG`, `syncCTAThreads`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1326-1350
```cpp
1326:   // This function does not use parameters, so we can emit just default values.
1327:   llvm::Value *Args[] = {
1328:       llvm::ConstantPointerNull::get(
1329:           cast<llvm::PointerType>(getIdentTyPointerTy())),
1330:       llvm::ConstantInt::get(CGF.Int32Ty, /*V=*/0, /*isSigned=*/true)};
1331:   CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1332:                           CGM.getModule(), OMPRTL___kmpc_barrier_simple_spmd),
1333:                       Args);
1334: }
1335: 
1336: void CGOpenMPRuntimeGPU::emitBarrierCall(CodeGenFunction &CGF,
1337:                                            SourceLocation Loc,
1338:                                            OpenMPDirectiveKind Kind, bool,
1339:                                            bool) {
1340:   // Always emit simple barriers!
1341:   if (!CGF.HaveInsertPoint())
1342:     return;
1343:   // Build call __kmpc_cancel_barrier(loc, thread_id);
1344:   unsigned Flags = getDefaultFlagsForBarriers(Kind);
1345:   llvm::Value *Args[] = {emitUpdateLocation(CGF, Loc, Flags),
1346:                          getThreadID(CGF, Loc)};
1347: 
1348:   CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1349:                           CGM.getModule(), OMPRTL___kmpc_barrier),
1350:                       Args);
```
- **EN**: This block defines callable entry points like `emitBarrierCall`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitBarrierCall`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1351-1375
```cpp
1351: }
1352: 
1353: void CGOpenMPRuntimeGPU::emitCriticalRegion(
1354:     CodeGenFunction &CGF, StringRef CriticalName,
1355:     const RegionCodeGenTy &CriticalOpGen, SourceLocation Loc,
1356:     const Expr *Hint) {
1357:   llvm::BasicBlock *LoopBB = CGF.createBasicBlock("omp.critical.loop");
1358:   llvm::BasicBlock *TestBB = CGF.createBasicBlock("omp.critical.test");
1359:   llvm::BasicBlock *SyncBB = CGF.createBasicBlock("omp.critical.sync");
1360:   llvm::BasicBlock *BodyBB = CGF.createBasicBlock("omp.critical.body");
1361:   llvm::BasicBlock *ExitBB = CGF.createBasicBlock("omp.critical.exit");
1362: 
1363:   auto &RT = static_cast<CGOpenMPRuntimeGPU &>(CGF.CGM.getOpenMPRuntime());
1364: 
1365:   // Get the mask of active threads in the warp.
1366:   llvm::Value *Mask = CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1367:       CGM.getModule(), OMPRTL___kmpc_warp_active_thread_mask));
1368:   // Fetch team-local id of the thread.
1369:   llvm::Value *ThreadID = RT.getGPUThreadID(CGF);
1370: 
1371:   // Get the width of the team.
1372:   llvm::Value *TeamWidth = RT.getGPUNumThreads(CGF);
1373: 
1374:   // Initialize the counter variable for the loop.
1375:   QualType Int32Ty =
```
- **EN**: This block defines callable entry points like `emitCriticalRegion`.
- **CN**: 该代码块定义可调用入口，例如 `emitCriticalRegion`。

### Lines 1376-1400
```cpp
1376:       CGF.getContext().getIntTypeForBitwidth(/*DestWidth=*/32, /*Signed=*/0);
1377:   Address Counter = CGF.CreateMemTempWithoutCast(Int32Ty, "critical_counter");
1378:   LValue CounterLVal = CGF.MakeAddrLValue(Counter, Int32Ty);
1379:   CGF.EmitStoreOfScalar(llvm::Constant::getNullValue(CGM.Int32Ty), CounterLVal,
1380:                         /*isInit=*/true);
1381: 
1382:   // Block checks if loop counter exceeds upper bound.
1383:   CGF.EmitBlock(LoopBB);
1384:   llvm::Value *CounterVal = CGF.EmitLoadOfScalar(CounterLVal, Loc);
1385:   llvm::Value *CmpLoopBound = CGF.Builder.CreateICmpSLT(CounterVal, TeamWidth);
1386:   CGF.Builder.CreateCondBr(CmpLoopBound, TestBB, ExitBB);
1387: 
1388:   // Block tests which single thread should execute region, and which threads
1389:   // should go straight to synchronisation point.
1390:   CGF.EmitBlock(TestBB);
1391:   CounterVal = CGF.EmitLoadOfScalar(CounterLVal, Loc);
1392:   llvm::Value *CmpThreadToCounter =
1393:       CGF.Builder.CreateICmpEQ(ThreadID, CounterVal);
1394:   CGF.Builder.CreateCondBr(CmpThreadToCounter, BodyBB, SyncBB);
1395: 
1396:   // Block emits the body of the critical region.
1397:   CGF.EmitBlock(BodyBB);
1398: 
1399:   // Output the critical statement.
1400:   CGOpenMPRuntime::emitCriticalRegion(CGF, CriticalName, CriticalOpGen, Loc,
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1401-1425
```cpp
1401:                                       Hint);
1402: 
1403:   // After the body surrounded by the critical region, the single executing
1404:   // thread will jump to the synchronisation point.
1405:   // Block waits for all threads in current team to finish then increments the
1406:   // counter variable and returns to the loop.
1407:   CGF.EmitBlock(SyncBB);
1408:   // Reconverge active threads in the warp.
1409:   (void)CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
1410:                                 CGM.getModule(), OMPRTL___kmpc_syncwarp),
1411:                             Mask);
1412: 
1413:   llvm::Value *IncCounterVal =
1414:       CGF.Builder.CreateNSWAdd(CounterVal, CGF.Builder.getInt32(1));
1415:   CGF.EmitStoreOfScalar(IncCounterVal, CounterLVal);
1416:   CGF.EmitBranch(LoopBB);
1417: 
1418:   // Block that is reached when  all threads in the team complete the region.
1419:   CGF.EmitBlock(ExitBB, /*IsFinished=*/true);
1420: }
1421: 
1422: /// Cast value to the specified type.
1423: static llvm::Value *castValueToType(CodeGenFunction &CGF, llvm::Value *Val,
1424:                                     QualType ValTy, QualType CastTy,
1425:                                     SourceLocation Loc) {
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1426-1450
```cpp
1426:   assert(!CGF.getContext().getTypeSizeInChars(CastTy).isZero() &&
1427:          "Cast type must sized.");
1428:   assert(!CGF.getContext().getTypeSizeInChars(ValTy).isZero() &&
1429:          "Val type must sized.");
1430:   llvm::Type *LLVMCastTy = CGF.ConvertTypeForMem(CastTy);
1431:   if (ValTy == CastTy)
1432:     return Val;
1433:   if (CGF.getContext().getTypeSizeInChars(ValTy) ==
1434:       CGF.getContext().getTypeSizeInChars(CastTy))
1435:     return CGF.Builder.CreateBitCast(Val, LLVMCastTy);
1436:   if (CastTy->isIntegerType() && ValTy->isIntegerType())
1437:     return CGF.Builder.CreateIntCast(Val, LLVMCastTy,
1438:                                      CastTy->hasSignedIntegerRepresentation());
1439:   Address CastItem = CGF.CreateMemTempWithoutCast(CastTy);
1440:   Address ValCastItem = CastItem.withElementType(Val->getType());
1441:   CGF.EmitStoreOfScalar(Val, ValCastItem, /*Volatile=*/false, ValTy,
1442:                         LValueBaseInfo(AlignmentSource::Type),
1443:                         TBAAAccessInfo());
1444:   return CGF.EmitLoadOfScalar(CastItem, /*Volatile=*/false, CastTy, Loc,
1445:                               LValueBaseInfo(AlignmentSource::Type),
1446:                               TBAAAccessInfo());
1447: }
1448: 
1449: ///
1450: /// Design of OpenMP reductions on the GPU
```
- **EN**: This block spells out callable entry points like `LValueBaseInfo`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出可调用入口的声明，例如 `LValueBaseInfo`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1451-1475
```cpp
1451: ///
1452: /// Consider a typical OpenMP program with one or more reduction
1453: /// clauses:
1454: ///
1455: /// float foo;
1456: /// double bar;
1457: /// #pragma omp target teams distribute parallel for \
1458: ///             reduction(+:foo) reduction(*:bar)
1459: /// for (int i = 0; i < N; i++) {
1460: ///   foo += A[i]; bar *= B[i];
1461: /// }
1462: ///
1463: /// where 'foo' and 'bar' are reduced across all OpenMP threads in
1464: /// all teams.  In our OpenMP implementation on the NVPTX device an
1465: /// OpenMP team is mapped to a CUDA threadblock and OpenMP threads
1466: /// within a team are mapped to CUDA threads within a threadblock.
1467: /// Our goal is to efficiently aggregate values across all OpenMP
1468: /// threads such that:
1469: ///
1470: ///   - the compiler and runtime are logically concise, and
1471: ///   - the reduction is performed efficiently in a hierarchical
1472: ///     manner as follows: within OpenMP threads in the same warp,
1473: ///     across warps in a threadblock, and finally across teams on
1474: ///     the NVPTX device.
1475: ///
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1476-1500
```cpp
1476: /// Introduction to Decoupling
1477: ///
1478: /// We would like to decouple the compiler and the runtime so that the
1479: /// latter is ignorant of the reduction variables (number, data types)
1480: /// and the reduction operators.  This allows a simpler interface
1481: /// and implementation while still attaining good performance.
1482: ///
1483: /// Pseudocode for the aforementioned OpenMP program generated by the
1484: /// compiler is as follows:
1485: ///
1486: /// 1. Create private copies of reduction variables on each OpenMP
1487: ///    thread: 'foo_private', 'bar_private'
1488: /// 2. Each OpenMP thread reduces the chunk of 'A' and 'B' assigned
1489: ///    to it and writes the result in 'foo_private' and 'bar_private'
1490: ///    respectively.
1491: /// 3. Call the OpenMP runtime on the GPU to reduce within a team
1492: ///    and store the result on the team master:
1493: ///
1494: ///     __kmpc_nvptx_parallel_reduce_nowait_v2(...,
1495: ///        reduceData, shuffleReduceFn, interWarpCpyFn)
1496: ///
1497: ///     where:
1498: ///       struct ReduceData {
1499: ///         double *foo;
1500: ///         double *bar;
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1501-1525
```cpp
1501: ///       } reduceData
1502: ///       reduceData.foo = &foo_private
1503: ///       reduceData.bar = &bar_private
1504: ///
1505: ///     'shuffleReduceFn' and 'interWarpCpyFn' are pointers to two
1506: ///     auxiliary functions generated by the compiler that operate on
1507: ///     variables of type 'ReduceData'.  They aid the runtime perform
1508: ///     algorithmic steps in a data agnostic manner.
1509: ///
1510: ///     'shuffleReduceFn' is a pointer to a function that reduces data
1511: ///     of type 'ReduceData' across two OpenMP threads (lanes) in the
1512: ///     same warp.  It takes the following arguments as input:
1513: ///
1514: ///     a. variable of type 'ReduceData' on the calling lane,
1515: ///     b. its lane_id,
1516: ///     c. an offset relative to the current lane_id to generate a
1517: ///        remote_lane_id.  The remote lane contains the second
1518: ///        variable of type 'ReduceData' that is to be reduced.
1519: ///     d. an algorithm version parameter determining which reduction
1520: ///        algorithm to use.
1521: ///
1522: ///     'shuffleReduceFn' retrieves data from the remote lane using
1523: ///     efficient GPU shuffle intrinsics and reduces, using the
1524: ///     algorithm specified by the 4th parameter, the two operands
1525: ///     element-wise.  The result is written to the first operand.
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1526-1550
```cpp
1526: ///
1527: ///     Different reduction algorithms are implemented in different
1528: ///     runtime functions, all calling 'shuffleReduceFn' to perform
1529: ///     the essential reduction step.  Therefore, based on the 4th
1530: ///     parameter, this function behaves slightly differently to
1531: ///     cooperate with the runtime to ensure correctness under
1532: ///     different circumstances.
1533: ///
1534: ///     'InterWarpCpyFn' is a pointer to a function that transfers
1535: ///     reduced variables across warps.  It tunnels, through CUDA
1536: ///     shared memory, the thread-private data of type 'ReduceData'
1537: ///     from lane 0 of each warp to a lane in the first warp.
1538: /// 4. Call the OpenMP runtime on the GPU to reduce across teams.
1539: ///    The last team writes the global reduced value to memory.
1540: ///
1541: ///     ret = __kmpc_nvptx_teams_reduce_nowait(...,
1542: ///             reduceData, shuffleReduceFn, interWarpCpyFn,
1543: ///             scratchpadCopyFn, loadAndReduceFn)
1544: ///
1545: ///     'scratchpadCopyFn' is a helper that stores reduced
1546: ///     data from the team master to a scratchpad array in
1547: ///     global memory.
1548: ///
1549: ///     'loadAndReduceFn' is a helper that loads data from
1550: ///     the scratchpad array and reduces it with the input
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1551-1575
```cpp
1551: ///     operand.
1552: ///
1553: ///     These compiler generated functions hide address
1554: ///     calculation and alignment information from the runtime.
1555: /// 5. if ret == 1:
1556: ///     The team master of the last team stores the reduced
1557: ///     result to the globals in memory.
1558: ///     foo += reduceData.foo; bar *= reduceData.bar
1559: ///
1560: ///
1561: /// Warp Reduction Algorithms
1562: ///
1563: /// On the warp level, we have three algorithms implemented in the
1564: /// OpenMP runtime depending on the number of active lanes:
1565: ///
1566: /// Full Warp Reduction
1567: ///
1568: /// The reduce algorithm within a warp where all lanes are active
1569: /// is implemented in the runtime as follows:
1570: ///
1571: /// full_warp_reduce(void *reduce_data,
1572: ///                  kmp_ShuffleReductFctPtr ShuffleReduceFn) {
1573: ///   for (int offset = WARPSIZE/2; offset > 0; offset /= 2)
1574: ///     ShuffleReduceFn(reduce_data, 0, offset, 0);
1575: /// }
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1576-1600
```cpp
1576: ///
1577: /// The algorithm completes in log(2, WARPSIZE) steps.
1578: ///
1579: /// 'ShuffleReduceFn' is used here with lane_id set to 0 because it is
1580: /// not used therefore we save instructions by not retrieving lane_id
1581: /// from the corresponding special registers.  The 4th parameter, which
1582: /// represents the version of the algorithm being used, is set to 0 to
1583: /// signify full warp reduction.
1584: ///
1585: /// In this version, 'ShuffleReduceFn' behaves, per element, as follows:
1586: ///
1587: /// #reduce_elem refers to an element in the local lane's data structure
1588: /// #remote_elem is retrieved from a remote lane
1589: /// remote_elem = shuffle_down(reduce_elem, offset, WARPSIZE);
1590: /// reduce_elem = reduce_elem REDUCE_OP remote_elem;
1591: ///
1592: /// Contiguous Partial Warp Reduction
1593: ///
1594: /// This reduce algorithm is used within a warp where only the first
1595: /// 'n' (n <= WARPSIZE) lanes are active.  It is typically used when the
1596: /// number of OpenMP threads in a parallel region is not a multiple of
1597: /// WARPSIZE.  The algorithm is implemented in the runtime as follows:
1598: ///
1599: /// void
1600: /// contiguous_partial_reduce(void *reduce_data,
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1601-1625
```cpp
1601: ///                           kmp_ShuffleReductFctPtr ShuffleReduceFn,
1602: ///                           int size, int lane_id) {
1603: ///   int curr_size;
1604: ///   int offset;
1605: ///   curr_size = size;
1606: ///   mask = curr_size/2;
1607: ///   while (offset>0) {
1608: ///     ShuffleReduceFn(reduce_data, lane_id, offset, 1);
1609: ///     curr_size = (curr_size+1)/2;
1610: ///     offset = curr_size/2;
1611: ///   }
1612: /// }
1613: ///
1614: /// In this version, 'ShuffleReduceFn' behaves, per element, as follows:
1615: ///
1616: /// remote_elem = shuffle_down(reduce_elem, offset, WARPSIZE);
1617: /// if (lane_id < offset)
1618: ///     reduce_elem = reduce_elem REDUCE_OP remote_elem
1619: /// else
1620: ///     reduce_elem = remote_elem
1621: ///
1622: /// This algorithm assumes that the data to be reduced are located in a
1623: /// contiguous subset of lanes starting from the first.  When there is
1624: /// an odd number of active lanes, the data in the last lane is not
1625: /// aggregated with any other lane's dat but is instead copied over.
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1626-1650
```cpp
1626: ///
1627: /// Dispersed Partial Warp Reduction
1628: ///
1629: /// This algorithm is used within a warp when any discontiguous subset of
1630: /// lanes are active.  It is used to implement the reduction operation
1631: /// across lanes in an OpenMP simd region or in a nested parallel region.
1632: ///
1633: /// void
1634: /// dispersed_partial_reduce(void *reduce_data,
1635: ///                          kmp_ShuffleReductFctPtr ShuffleReduceFn) {
1636: ///   int size, remote_id;
1637: ///   int logical_lane_id = number_of_active_lanes_before_me() * 2;
1638: ///   do {
1639: ///       remote_id = next_active_lane_id_right_after_me();
1640: ///       # the above function returns 0 of no active lane
1641: ///       # is present right after the current lane.
1642: ///       size = number_of_active_lanes_in_this_warp();
1643: ///       logical_lane_id /= 2;
1644: ///       ShuffleReduceFn(reduce_data, logical_lane_id,
1645: ///                       remote_id-1-threadIdx.x, 2);
1646: ///   } while (logical_lane_id % 2 == 0 && size > 1);
1647: /// }
1648: ///
1649: /// There is no assumption made about the initial state of the reduction.
1650: /// Any number of lanes (>=1) could be active at any position.  The reduction
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1651-1675
```cpp
1651: /// result is returned in the first active lane.
1652: ///
1653: /// In this version, 'ShuffleReduceFn' behaves, per element, as follows:
1654: ///
1655: /// remote_elem = shuffle_down(reduce_elem, offset, WARPSIZE);
1656: /// if (lane_id % 2 == 0 && offset > 0)
1657: ///     reduce_elem = reduce_elem REDUCE_OP remote_elem
1658: /// else
1659: ///     reduce_elem = remote_elem
1660: ///
1661: ///
1662: /// Intra-Team Reduction
1663: ///
1664: /// This function, as implemented in the runtime call
1665: /// '__kmpc_nvptx_parallel_reduce_nowait_v2', aggregates data across OpenMP
1666: /// threads in a team.  It first reduces within a warp using the
1667: /// aforementioned algorithms.  We then proceed to gather all such
1668: /// reduced values at the first warp.
1669: ///
1670: /// The runtime makes use of the function 'InterWarpCpyFn', which copies
1671: /// data from each of the "warp master" (zeroth lane of each warp, where
1672: /// warp-reduced data is held) to the zeroth warp.  This step reduces (in
1673: /// a mathematical sense) the problem of reduction across warp masters in
1674: /// a block to the problem of warp reduction.
1675: ///
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1676-1700
```cpp
1676: ///
1677: /// Inter-Team Reduction
1678: ///
1679: /// Once a team has reduced its data to a single value, it is stored in
1680: /// a global scratchpad array.  Since each team has a distinct slot, this
1681: /// can be done without locking.
1682: ///
1683: /// The last team to write to the scratchpad array proceeds to reduce the
1684: /// scratchpad array.  One or more workers in the last team use the helper
1685: /// 'loadAndReduceDataFn' to load and reduce values from the array, i.e.,
1686: /// the k'th worker reduces every k'th element.
1687: ///
1688: /// Finally, a call is made to '__kmpc_nvptx_parallel_reduce_nowait_v2' to
1689: /// reduce across workers and compute a globally reduced value.
1690: ///
1691: void CGOpenMPRuntimeGPU::emitReduction(
1692:     CodeGenFunction &CGF, SourceLocation Loc, ArrayRef<const Expr *> Privates,
1693:     ArrayRef<const Expr *> LHSExprs, ArrayRef<const Expr *> RHSExprs,
1694:     ArrayRef<const Expr *> ReductionOps, ReductionOptionsTy Options) {
1695:   if (!CGF.HaveInsertPoint())
1696:     return;
1697: 
1698:   bool ParallelReduction = isOpenMPParallelDirective(Options.ReductionKind);
1699:   bool TeamsReduction = isOpenMPTeamsDirective(Options.ReductionKind);
1700: 
```
- **EN**: This block defines callable entry points like `emitReduction`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitReduction`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1701-1725
```cpp
1701:   ASTContext &C = CGM.getContext();
1702: 
1703:   if (Options.SimpleReduction) {
1704:     assert(!TeamsReduction && !ParallelReduction &&
1705:            "Invalid reduction selection in emitReduction.");
1706:     (void)ParallelReduction;
1707:     CGOpenMPRuntime::emitReduction(CGF, Loc, Privates, LHSExprs, RHSExprs,
1708:                                    ReductionOps, Options);
1709:     return;
1710:   }
1711: 
1712:   llvm::SmallDenseMap<const ValueDecl *, const FieldDecl *> VarFieldMap;
1713:   llvm::SmallVector<const ValueDecl *, 4> PrivatesReductions(Privates.size());
1714:   int Cnt = 0;
1715:   for (const Expr *DRE : Privates) {
1716:     PrivatesReductions[Cnt] = cast<DeclRefExpr>(DRE)->getDecl();
1717:     ++Cnt;
1718:   }
1719:   const RecordDecl *ReductionRec = ::buildRecordForGlobalizedVars(
1720:       CGM.getContext(), PrivatesReductions, {}, VarFieldMap, 1);
1721: 
1722:   if (TeamsReduction)
1723:     TeamsReductions.push_back(ReductionRec);
1724: 
1725:   // Source location for the ident struct
```
- **EN**: This block defines callable entry points like `emitReduction`, `PrivatesReductions`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitReduction`, `PrivatesReductions`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1726-1750
```cpp
1726:   llvm::Value *RTLoc = emitUpdateLocation(CGF, Loc);
1727: 
1728:   using InsertPointTy = llvm::OpenMPIRBuilder::InsertPointTy;
1729:   InsertPointTy AllocaIP(CGF.AllocaInsertPt->getParent(),
1730:                          CGF.AllocaInsertPt->getIterator());
1731:   InsertPointTy CodeGenIP(CGF.Builder.GetInsertBlock(),
1732:                           CGF.Builder.GetInsertPoint());
1733:   llvm::OpenMPIRBuilder::LocationDescription OmpLoc(
1734:       CodeGenIP, CGF.SourceLocToDebugLoc(Loc));
1735:   llvm::SmallVector<llvm::OpenMPIRBuilder::ReductionInfo, 2> ReductionInfos;
1736: 
1737:   CodeGenFunction::OMPPrivateScope Scope(CGF);
1738:   unsigned Idx = 0;
1739:   for (const Expr *Private : Privates) {
1740:     llvm::Type *ElementType;
1741:     llvm::Value *Variable;
1742:     llvm::Value *PrivateVariable;
1743:     llvm::OpenMPIRBuilder::ReductionGenAtomicCBTy AtomicReductionGen = nullptr;
1744:     ElementType = CGF.ConvertTypeForMem(Private->getType());
1745:     const auto *RHSVar =
1746:         cast<VarDecl>(cast<DeclRefExpr>(RHSExprs[Idx])->getDecl());
1747:     PrivateVariable = CGF.GetAddrOfLocalVar(RHSVar).emitRawPointer(CGF);
1748:     const auto *LHSVar =
1749:         cast<VarDecl>(cast<DeclRefExpr>(LHSExprs[Idx])->getDecl());
1750:     Variable = CGF.GetAddrOfLocalVar(LHSVar).emitRawPointer(CGF);
```
- **EN**: This block defines callable entry points like `AllocaIP`, `CodeGenIP`, `OmpLoc`, `Scope`; uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `AllocaIP`, `CodeGenIP`, `OmpLoc`, `Scope`；通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 1751-1775
```cpp
1751:     llvm::OpenMPIRBuilder::EvalKind EvalKind;
1752:     switch (CGF.getEvaluationKind(Private->getType())) {
1753:     case TEK_Scalar:
1754:       EvalKind = llvm::OpenMPIRBuilder::EvalKind::Scalar;
1755:       break;
1756:     case TEK_Complex:
1757:       EvalKind = llvm::OpenMPIRBuilder::EvalKind::Complex;
1758:       break;
1759:     case TEK_Aggregate:
1760:       EvalKind = llvm::OpenMPIRBuilder::EvalKind::Aggregate;
1761:       break;
1762:     }
1763:     auto ReductionGen = [&](InsertPointTy CodeGenIP, unsigned I,
1764:                             llvm::Value **LHSPtr, llvm::Value **RHSPtr,
1765:                             llvm::Function *NewFunc) {
1766:       CGF.Builder.restoreIP(CodeGenIP);
1767:       auto *CurFn = CGF.CurFn;
1768:       CGF.CurFn = NewFunc;
1769: 
1770:       *LHSPtr = CGF.GetAddrOfLocalVar(
1771:                        cast<VarDecl>(cast<DeclRefExpr>(LHSExprs[I])->getDecl()))
1772:                     .emitRawPointer(CGF);
1773:       *RHSPtr = CGF.GetAddrOfLocalVar(
1774:                        cast<VarDecl>(cast<DeclRefExpr>(RHSExprs[I])->getDecl()))
1775:                     .emitRawPointer(CGF);
```
- **EN**: This block uses control flow (switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（switch, case）细化 LLVM IR 生成 行为。

### Lines 1776-1800
```cpp
1776: 
1777:       emitSingleReductionCombiner(CGF, ReductionOps[I], Privates[I],
1778:                                   cast<DeclRefExpr>(LHSExprs[I]),
1779:                                   cast<DeclRefExpr>(RHSExprs[I]));
1780: 
1781:       CGF.CurFn = CurFn;
1782: 
1783:       return InsertPointTy(CGF.Builder.GetInsertBlock(),
1784:                            CGF.Builder.GetInsertPoint());
1785:     };
1786:     ReductionInfos.emplace_back(llvm::OpenMPIRBuilder::ReductionInfo(
1787:         ElementType, Variable, PrivateVariable, EvalKind,
1788:         /*ReductionGen=*/nullptr, ReductionGen, AtomicReductionGen,
1789:         /*DataPtrPtrGen=*/nullptr));
1790:     Idx++;
1791:   }
1792: 
1793:   llvm::OpenMPIRBuilder::InsertPointTy AfterIP =
1794:       cantFail(OMPBuilder.createReductionsGPU(
1795:           OmpLoc, AllocaIP, CodeGenIP, ReductionInfos, /*IsByRef=*/{}, false,
1796:           TeamsReduction, llvm::OpenMPIRBuilder::ReductionGenCBKind::Clang,
1797:           CGF.getTarget().getGridValue(),
1798:           C.getLangOpts().OpenMPCUDAReductionBufNum, RTLoc));
1799:   CGF.Builder.restoreIP(AfterIP);
1800: }
```
- **EN**: This block defines callable entry points like `emitSingleReductionCombiner`, `InsertPointTy`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `emitSingleReductionCombiner`, `InsertPointTy`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 1801-1825
```cpp
1801: 
1802: const VarDecl *
1803: CGOpenMPRuntimeGPU::translateParameter(const FieldDecl *FD,
1804:                                        const VarDecl *NativeParam) const {
1805:   if (!NativeParam->getType()->isReferenceType())
1806:     return NativeParam;
1807:   QualType ArgType = NativeParam->getType();
1808:   QualifierCollector QC;
1809:   const Type *NonQualTy = QC.strip(ArgType);
1810:   QualType PointeeTy = cast<ReferenceType>(NonQualTy)->getPointeeType();
1811:   if (const auto *Attr = FD->getAttr<OMPCaptureKindAttr>()) {
1812:     if (Attr->getCaptureKind() == OMPC_map) {
1813:       PointeeTy = CGM.getContext().getAddrSpaceQualType(PointeeTy,
1814:                                                         LangAS::opencl_global);
1815:     }
1816:   }
1817:   ArgType = CGM.getContext().getPointerType(PointeeTy);
1818:   QC.addRestrict();
1819:   ArgType = QC.apply(CGM.getContext(), ArgType);
1820:   if (isa<ImplicitParamDecl>(NativeParam))
1821:     return ImplicitParamDecl::Create(
1822:         CGM.getContext(), /*DC=*/nullptr, NativeParam->getLocation(),
1823:         NativeParam->getIdentifier(), ArgType, ImplicitParamKind::Other);
1824:   return ParmVarDecl::Create(
1825:       CGM.getContext(),
```
- **EN**: This block defines callable entry points like `translateParameter`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `translateParameter`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1826-1850
```cpp
1826:       const_cast<DeclContext *>(NativeParam->getDeclContext()),
1827:       NativeParam->getBeginLoc(), NativeParam->getLocation(),
1828:       NativeParam->getIdentifier(), ArgType,
1829:       /*TInfo=*/nullptr, SC_None, /*DefArg=*/nullptr);
1830: }
1831: 
1832: Address
1833: CGOpenMPRuntimeGPU::getParameterAddress(CodeGenFunction &CGF,
1834:                                           const VarDecl *NativeParam,
1835:                                           const VarDecl *TargetParam) const {
1836:   assert(NativeParam != TargetParam &&
1837:          NativeParam->getType()->isReferenceType() &&
1838:          "Native arg must not be the same as target arg.");
1839:   Address LocalAddr = CGF.GetAddrOfLocalVar(TargetParam);
1840:   QualType NativeParamType = NativeParam->getType();
1841:   QualifierCollector QC;
1842:   const Type *NonQualTy = QC.strip(NativeParamType);
1843:   QualType NativePointeeTy = cast<ReferenceType>(NonQualTy)->getPointeeType();
1844:   unsigned NativePointeeAddrSpace =
1845:       CGF.getTypes().getTargetAddressSpace(NativePointeeTy);
1846:   QualType TargetTy = TargetParam->getType();
1847:   llvm::Value *TargetAddr = CGF.EmitLoadOfScalar(LocalAddr, /*Volatile=*/false,
1848:                                                  TargetTy, SourceLocation());
1849:   // Cast to native address space.
1850:   TargetAddr = CGF.Builder.CreatePointerBitCastOrAddrSpaceCast(
```
- **EN**: This block defines callable entry points like `getParameterAddress`, `SourceLocation`; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getParameterAddress`, `SourceLocation`；使用断言或不可达标记保护关键不变量。

### Lines 1851-1875
```cpp
1851:       TargetAddr,
1852:       llvm::PointerType::get(CGF.getLLVMContext(), NativePointeeAddrSpace));
1853:   Address NativeParamAddr = CGF.CreateMemTemp(NativeParamType);
1854:   CGF.EmitStoreOfScalar(TargetAddr, NativeParamAddr, /*Volatile=*/false,
1855:                         NativeParamType);
1856:   return NativeParamAddr;
1857: }
1858: 
1859: void CGOpenMPRuntimeGPU::emitOutlinedFunctionCall(
1860:     CodeGenFunction &CGF, SourceLocation Loc, llvm::FunctionCallee OutlinedFn,
1861:     ArrayRef<llvm::Value *> Args) const {
1862:   SmallVector<llvm::Value *, 4> TargetArgs;
1863:   TargetArgs.reserve(Args.size());
1864:   auto *FnType = OutlinedFn.getFunctionType();
1865:   for (unsigned I = 0, E = Args.size(); I < E; ++I) {
1866:     if (FnType->isVarArg() && FnType->getNumParams() <= I) {
1867:       TargetArgs.append(std::next(Args.begin(), I), Args.end());
1868:       break;
1869:     }
1870:     llvm::Type *TargetType = FnType->getParamType(I);
1871:     llvm::Value *NativeArg = Args[I];
1872:     if (!TargetType->isPointerTy()) {
1873:       TargetArgs.emplace_back(NativeArg);
1874:       continue;
1875:     }
```
- **EN**: This block defines callable entry points like `get`, `emitOutlinedFunctionCall`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `emitOutlinedFunctionCall`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1876-1900
```cpp
1876:     TargetArgs.emplace_back(
1877:         CGF.Builder.CreatePointerBitCastOrAddrSpaceCast(NativeArg, TargetType));
1878:   }
1879:   CGOpenMPRuntime::emitOutlinedFunctionCall(CGF, Loc, OutlinedFn, TargetArgs);
1880: }
1881: 
1882: /// Emit function which wraps the outline parallel region
1883: /// and controls the arguments which are passed to this function.
1884: /// The wrapper ensures that the outlined function is called
1885: /// with the correct arguments when data is shared.
1886: llvm::Function *CGOpenMPRuntimeGPU::createParallelDataSharingWrapper(
1887:     llvm::Function *OutlinedParallelFn, const OMPExecutableDirective &D) {
1888:   ASTContext &Ctx = CGM.getContext();
1889:   const auto &CS = *D.getCapturedStmt(OMPD_parallel);
1890: 
1891:   // Create a function that takes as argument the source thread.
1892:   FunctionArgList WrapperArgs;
1893:   QualType Int16QTy =
1894:       Ctx.getIntTypeForBitwidth(/*DestWidth=*/16, /*Signed=*/false);
1895:   QualType Int32QTy =
1896:       Ctx.getIntTypeForBitwidth(/*DestWidth=*/32, /*Signed=*/false);
1897:   auto *ParallelLevelArg = ImplicitParamDecl::Create(
1898:       Ctx, /*DC=*/nullptr, D.getBeginLoc(),
1899:       /*Id=*/nullptr, Int16QTy, ImplicitParamKind::Other);
1900:   auto *WrapperArg = ImplicitParamDecl::Create(
```
- **EN**: This block defines callable entry points like `emitOutlinedFunctionCall`.
- **CN**: 该代码块定义可调用入口，例如 `emitOutlinedFunctionCall`。

### Lines 1901-1925
```cpp
1901:       Ctx, /*DC=*/nullptr, D.getBeginLoc(),
1902:       /*Id=*/nullptr, Int32QTy, ImplicitParamKind::Other);
1903:   WrapperArgs.emplace_back(ParallelLevelArg);
1904:   WrapperArgs.emplace_back(WrapperArg);
1905: 
1906:   const CGFunctionInfo &CGFI =
1907:       CGM.getTypes().arrangeBuiltinFunctionDeclaration(Ctx.VoidTy, WrapperArgs);
1908: 
1909:   auto *Fn = llvm::Function::Create(
1910:       CGM.getTypes().GetFunctionType(CGFI), llvm::GlobalValue::InternalLinkage,
1911:       Twine(OutlinedParallelFn->getName(), "_wrapper"), &CGM.getModule());
1912: 
1913:   // Ensure we do not inline the function. This is trivially true for the ones
1914:   // passed to __kmpc_fork_call but the ones calles in serialized regions
1915:   // could be inlined. This is not a perfect but it is closer to the invariant
1916:   // we want, namely, every data environment starts with a new function.
1917:   // TODO: We should pass the if condition to the runtime function and do the
1918:   //       handling there. Much cleaner code.
1919:   Fn->addFnAttr(llvm::Attribute::NoInline);
1920: 
1921:   CGM.SetInternalFunctionAttributes(GlobalDecl(), Fn, CGFI);
1922:   Fn->setLinkage(llvm::GlobalValue::InternalLinkage);
1923:   Fn->setDoesNotRecurse();
1924: 
1925:   CodeGenFunction CGF(CGM, /*suppressNewContext=*/true);
```
- **EN**: This block spells out callable entry points like `Twine`, `CGF`.
- **CN**: 该代码块给出可调用入口的声明，例如 `Twine`, `CGF`。

### Lines 1926-1950
```cpp
1926:   CGF.StartFunction(GlobalDecl(), Ctx.VoidTy, Fn, CGFI, WrapperArgs,
1927:                     D.getBeginLoc(), D.getBeginLoc());
1928: 
1929:   const auto *RD = CS.getCapturedRecordDecl();
1930:   auto CurField = RD->field_begin();
1931: 
1932:   Address ZeroAddr = CGF.CreateDefaultAlignTempAlloca(CGF.Int32Ty,
1933:                                                       /*Name=*/".zero.addr");
1934:   CGF.Builder.CreateStore(CGF.Builder.getInt32(/*C*/ 0), ZeroAddr);
1935:   // Get the array of arguments.
1936:   SmallVector<llvm::Value *, 8> Args;
1937: 
1938:   Args.emplace_back(CGF.GetAddrOfLocalVar(WrapperArg).emitRawPointer(CGF));
1939:   Args.emplace_back(ZeroAddr.emitRawPointer(CGF));
1940: 
1941:   CGBuilderTy &Bld = CGF.Builder;
1942:   auto CI = CS.capture_begin();
1943: 
1944:   // Use global memory for data sharing.
1945:   // Handle passing of global args to workers.
1946:   RawAddress GlobalArgs =
1947:       CGF.CreateDefaultAlignTempAlloca(CGF.VoidPtrPtrTy, "global_args");
1948:   llvm::Value *GlobalArgsPtr = GlobalArgs.getPointer();
1949:   llvm::Value *DataSharingArgs[] = {GlobalArgsPtr};
1950:   CGF.EmitRuntimeCall(OMPBuilder.getOrCreateRuntimeFunction(
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 1951-1975
```cpp
1951:                           CGM.getModule(), OMPRTL___kmpc_get_shared_variables),
1952:                       DataSharingArgs);
1953: 
1954:   // Retrieve the shared variables from the list of references returned
1955:   // by the runtime. Pass the variables to the outlined function.
1956:   Address SharedArgListAddress = Address::invalid();
1957:   if (CS.capture_size() > 0 ||
1958:       isOpenMPLoopBoundSharingDirective(D.getDirectiveKind())) {
1959:     SharedArgListAddress = CGF.EmitLoadOfPointer(
1960:         GlobalArgs, CGF.getContext()
1961:                         .getPointerType(CGF.getContext().VoidPtrTy)
1962:                         .castAs<PointerType>());
1963:   }
1964:   unsigned Idx = 0;
1965:   if (isOpenMPLoopBoundSharingDirective(D.getDirectiveKind())) {
1966:     Address Src = Bld.CreateConstInBoundsGEP(SharedArgListAddress, Idx);
1967:     Address TypedAddress = Bld.CreatePointerBitCastOrAddrSpaceCast(
1968:         Src, Bld.getPtrTy(0), CGF.SizeTy);
1969:     llvm::Value *LB = CGF.EmitLoadOfScalar(
1970:         TypedAddress,
1971:         /*Volatile=*/false,
1972:         CGF.getContext().getPointerType(CGF.getContext().getSizeType()),
1973:         cast<OMPLoopDirective>(D).getLowerBoundVariable()->getExprLoc());
1974:     Args.emplace_back(LB);
1975:     ++Idx;
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1976-2000
```cpp
1976:     Src = Bld.CreateConstInBoundsGEP(SharedArgListAddress, Idx);
1977:     TypedAddress = Bld.CreatePointerBitCastOrAddrSpaceCast(Src, Bld.getPtrTy(0),
1978:                                                            CGF.SizeTy);
1979:     llvm::Value *UB = CGF.EmitLoadOfScalar(
1980:         TypedAddress,
1981:         /*Volatile=*/false,
1982:         CGF.getContext().getPointerType(CGF.getContext().getSizeType()),
1983:         cast<OMPLoopDirective>(D).getUpperBoundVariable()->getExprLoc());
1984:     Args.emplace_back(UB);
1985:     ++Idx;
1986:   }
1987:   if (CS.capture_size() > 0) {
1988:     ASTContext &CGFContext = CGF.getContext();
1989:     for (unsigned I = 0, E = CS.capture_size(); I < E; ++I, ++CI, ++CurField) {
1990:       QualType ElemTy = CurField->getType();
1991:       Address Src = Bld.CreateConstInBoundsGEP(SharedArgListAddress, I + Idx);
1992:       Address TypedAddress = Bld.CreatePointerBitCastOrAddrSpaceCast(
1993:           Src, CGF.ConvertTypeForMem(CGFContext.getPointerType(ElemTy)),
1994:           CGF.ConvertTypeForMem(ElemTy));
1995:       llvm::Value *Arg = CGF.EmitLoadOfScalar(TypedAddress,
1996:                                               /*Volatile=*/false,
1997:                                               CGFContext.getPointerType(ElemTy),
1998:                                               CI->getLocation());
1999:       if (CI->capturesVariableByCopy() &&
2000:           !CI->getCapturedVar()->getType()->isAnyPointerType()) {
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 2001-2025
```cpp
2001:         Arg = castValueToType(CGF, Arg, ElemTy, CGFContext.getUIntPtrType(),
2002:                               CI->getLocation());
2003:       }
2004:       Args.emplace_back(Arg);
2005:     }
2006:   }
2007: 
2008:   emitOutlinedFunctionCall(CGF, D.getBeginLoc(), OutlinedParallelFn, Args);
2009:   CGF.FinishFunction();
2010:   return Fn;
2011: }
2012: 
2013: void CGOpenMPRuntimeGPU::emitFunctionProlog(CodeGenFunction &CGF,
2014:                                               const Decl *D) {
2015:   if (getDataSharingMode() != CGOpenMPRuntimeGPU::DS_Generic)
2016:     return;
2017: 
2018:   assert(D && "Expected function or captured|block decl.");
2019:   assert(FunctionGlobalizedDecls.count(CGF.CurFn) == 0 &&
2020:          "Function is registered already.");
2021:   assert((!TeamAndReductions.first || TeamAndReductions.first == D) &&
2022:          "Team is set but not processed.");
2023:   const Stmt *Body = nullptr;
2024:   bool NeedToDelayGlobalization = false;
2025:   if (const auto *FD = dyn_cast<FunctionDecl>(D)) {
```
- **EN**: This block defines callable entry points like `emitOutlinedFunctionCall`, `emitFunctionProlog`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitOutlinedFunctionCall`, `emitFunctionProlog`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2026-2050
```cpp
2026:     Body = FD->getBody();
2027:   } else if (const auto *BD = dyn_cast<BlockDecl>(D)) {
2028:     Body = BD->getBody();
2029:   } else if (const auto *CD = dyn_cast<CapturedDecl>(D)) {
2030:     Body = CD->getBody();
2031:     NeedToDelayGlobalization = CGF.CapturedStmtInfo->getKind() == CR_OpenMP;
2032:     if (NeedToDelayGlobalization &&
2033:         getExecutionMode() == CGOpenMPRuntimeGPU::EM_SPMD)
2034:       return;
2035:   }
2036:   if (!Body)
2037:     return;
2038:   CheckVarsEscapingDeclContext VarChecker(CGF, TeamAndReductions.second);
2039:   VarChecker.Visit(Body);
2040:   const RecordDecl *GlobalizedVarsRecord =
2041:       VarChecker.getGlobalizedRecord(IsInTTDRegion);
2042:   TeamAndReductions.first = nullptr;
2043:   TeamAndReductions.second.clear();
2044:   ArrayRef<const ValueDecl *> EscapedVariableLengthDecls =
2045:       VarChecker.getEscapedVariableLengthDecls();
2046:   ArrayRef<const ValueDecl *> DelayedVariableLengthDecls =
2047:       VarChecker.getDelayedVariableLengthDecls();
2048:   if (!GlobalizedVarsRecord && EscapedVariableLengthDecls.empty() &&
2049:       DelayedVariableLengthDecls.empty())
2050:     return;
```
- **EN**: This block defines callable entry points like `VarChecker`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `VarChecker`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2051-2075
```cpp
2051:   auto I = FunctionGlobalizedDecls.try_emplace(CGF.CurFn).first;
2052:   I->getSecond().MappedParams =
2053:       std::make_unique<CodeGenFunction::OMPMapVars>();
2054:   I->getSecond().EscapedParameters.insert(
2055:       VarChecker.getEscapedParameters().begin(),
2056:       VarChecker.getEscapedParameters().end());
2057:   I->getSecond().EscapedVariableLengthDecls.append(
2058:       EscapedVariableLengthDecls.begin(), EscapedVariableLengthDecls.end());
2059:   I->getSecond().DelayedVariableLengthDecls.append(
2060:       DelayedVariableLengthDecls.begin(), DelayedVariableLengthDecls.end());
2061:   DeclToAddrMapTy &Data = I->getSecond().LocalVarData;
2062:   for (const ValueDecl *VD : VarChecker.getEscapedDecls()) {
2063:     assert(VD->isCanonicalDecl() && "Expected canonical declaration");
2064:     Data.try_emplace(VD);
2065:   }
2066:   if (!NeedToDelayGlobalization) {
2067:     emitGenericVarsProlog(CGF, D->getBeginLoc());
2068:     struct GlobalizationScope final : EHScopeStack::Cleanup {
2069:       GlobalizationScope() = default;
2070: 
2071:       void Emit(CodeGenFunction &CGF, Flags flags) override {
2072:         static_cast<CGOpenMPRuntimeGPU &>(CGF.CGM.getOpenMPRuntime())
2073:             .emitGenericVarsEpilog(CGF);
2074:       }
2075:     };
```
- **EN**: This block introduces declarations such as `GlobalizationScope`; defines callable entry points like `emitGenericVarsProlog`, `Emit`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `GlobalizationScope` 的声明；定义可调用入口，例如 `emitGenericVarsProlog`, `Emit`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2076-2100
```cpp
2076:     CGF.EHStack.pushCleanup<GlobalizationScope>(NormalAndEHCleanup);
2077:   }
2078: }
2079: 
2080: Address CGOpenMPRuntimeGPU::getAddressOfLocalVariable(CodeGenFunction &CGF,
2081:                                                         const VarDecl *VD) {
2082:   if (VD && VD->hasAttr<OMPAllocateDeclAttr>()) {
2083:     const auto *A = VD->getAttr<OMPAllocateDeclAttr>();
2084:     auto AS = LangAS::Default;
2085:     switch (A->getAllocatorType()) {
2086:     case OMPAllocateDeclAttr::OMPNullMemAlloc:
2087:     case OMPAllocateDeclAttr::OMPDefaultMemAlloc:
2088:     case OMPAllocateDeclAttr::OMPHighBWMemAlloc:
2089:     case OMPAllocateDeclAttr::OMPLowLatMemAlloc:
2090:       break;
2091:     case OMPAllocateDeclAttr::OMPThreadMemAlloc:
2092:       return Address::invalid();
2093:     case OMPAllocateDeclAttr::OMPUserDefinedMemAlloc:
2094:       // TODO: implement aupport for user-defined allocators.
2095:       return Address::invalid();
2096:     case OMPAllocateDeclAttr::OMPConstMemAlloc:
2097:       AS = LangAS::cuda_constant;
2098:       break;
2099:     case OMPAllocateDeclAttr::OMPPTeamMemAlloc:
2100:       AS = LangAS::cuda_shared;
```
- **EN**: This block defines callable entry points like `getAddressOfLocalVariable`, `invalid`; uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getAddressOfLocalVariable`, `invalid`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 2101-2125
```cpp
2101:       break;
2102:     case OMPAllocateDeclAttr::OMPLargeCapMemAlloc:
2103:     case OMPAllocateDeclAttr::OMPCGroupMemAlloc:
2104:       break;
2105:     }
2106:     llvm::Type *VarTy = CGF.ConvertTypeForMem(VD->getType());
2107:     auto *GV = new llvm::GlobalVariable(
2108:         CGM.getModule(), VarTy, /*isConstant=*/false,
2109:         llvm::GlobalValue::InternalLinkage, llvm::PoisonValue::get(VarTy),
2110:         VD->getName(),
2111:         /*InsertBefore=*/nullptr, llvm::GlobalValue::NotThreadLocal,
2112:         CGM.getContext().getTargetAddressSpace(AS));
2113:     CharUnits Align = CGM.getContext().getDeclAlign(VD);
2114:     GV->setAlignment(Align.getAsAlign());
2115:     return Address(
2116:         CGF.Builder.CreatePointerBitCastOrAddrSpaceCast(
2117:             GV, CGF.Builder.getPtrTy(CGM.getContext().getTargetAddressSpace(
2118:                     VD->getType().getAddressSpace()))),
2119:         VarTy, Align);
2120:   }
2121: 
2122:   if (getDataSharingMode() != CGOpenMPRuntimeGPU::DS_Generic)
2123:     return Address::invalid();
2124: 
2125:   VD = VD->getCanonicalDecl();
```
- **EN**: This block spells out callable entry points like `get`, `Address`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`, `Address`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 2126-2150
```cpp
2126:   auto I = FunctionGlobalizedDecls.find(CGF.CurFn);
2127:   if (I == FunctionGlobalizedDecls.end())
2128:     return Address::invalid();
2129:   auto VDI = I->getSecond().LocalVarData.find(VD);
2130:   if (VDI != I->getSecond().LocalVarData.end())
2131:     return VDI->second.PrivateAddr;
2132:   if (VD->hasAttrs()) {
2133:     for (specific_attr_iterator<OMPReferencedVarAttr> IT(VD->attr_begin()),
2134:          E(VD->attr_end());
2135:          IT != E; ++IT) {
2136:       auto VDI = I->getSecond().LocalVarData.find(
2137:           cast<VarDecl>(cast<DeclRefExpr>(IT->getRef())->getDecl())
2138:               ->getCanonicalDecl());
2139:       if (VDI != I->getSecond().LocalVarData.end())
2140:         return VDI->second.PrivateAddr;
2141:     }
2142:   }
2143: 
2144:   return Address::invalid();
2145: }
2146: 
2147: void CGOpenMPRuntimeGPU::functionFinished(CodeGenFunction &CGF) {
2148:   FunctionGlobalizedDecls.erase(CGF.CurFn);
2149:   CGOpenMPRuntime::functionFinished(CGF);
2150: }
```
- **EN**: This block defines callable entry points like `invalid`, `functionFinished`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `invalid`, `functionFinished`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 2151-2175
```cpp
2151: 
2152: void CGOpenMPRuntimeGPU::getDefaultDistScheduleAndChunk(
2153:     CodeGenFunction &CGF, const OMPLoopDirective &S,
2154:     OpenMPDistScheduleClauseKind &ScheduleKind,
2155:     llvm::Value *&Chunk) const {
2156:   auto &RT = static_cast<CGOpenMPRuntimeGPU &>(CGF.CGM.getOpenMPRuntime());
2157:   if (getExecutionMode() == CGOpenMPRuntimeGPU::EM_SPMD) {
2158:     ScheduleKind = OMPC_DIST_SCHEDULE_static;
2159:     Chunk = CGF.EmitScalarConversion(
2160:         RT.getGPUNumThreads(CGF),
2161:         CGF.getContext().getIntTypeForBitwidth(32, /*Signed=*/0),
2162:         S.getIterationVariable()->getType(), S.getBeginLoc());
2163:     return;
2164:   }
2165:   CGOpenMPRuntime::getDefaultDistScheduleAndChunk(
2166:       CGF, S, ScheduleKind, Chunk);
2167: }
2168: 
2169: void CGOpenMPRuntimeGPU::getDefaultScheduleAndChunk(
2170:     CodeGenFunction &CGF, const OMPLoopDirective &S,
2171:     OpenMPScheduleClauseKind &ScheduleKind,
2172:     const Expr *&ChunkExpr) const {
2173:   ScheduleKind = OMPC_SCHEDULE_static;
2174:   // Chunk size is 1 in this case.
2175:   llvm::APInt ChunkSize(32, 1);
```
- **EN**: This block defines callable entry points like `getDefaultDistScheduleAndChunk`, `getDefaultScheduleAndChunk`, `ChunkSize`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `getDefaultDistScheduleAndChunk`, `getDefaultScheduleAndChunk`, `ChunkSize`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 2176-2200
```cpp
2176:   ChunkExpr = IntegerLiteral::Create(CGF.getContext(), ChunkSize,
2177:       CGF.getContext().getIntTypeForBitwidth(32, /*Signed=*/0),
2178:       SourceLocation());
2179: }
2180: 
2181: void CGOpenMPRuntimeGPU::adjustTargetSpecificDataForLambdas(
2182:     CodeGenFunction &CGF, const OMPExecutableDirective &D) const {
2183:   assert(isOpenMPTargetExecutionDirective(D.getDirectiveKind()) &&
2184:          " Expected target-based directive.");
2185:   const CapturedStmt *CS = D.getCapturedStmt(OMPD_target);
2186:   for (const CapturedStmt::Capture &C : CS->captures()) {
2187:     // Capture variables captured by reference in lambdas for target-based
2188:     // directives.
2189:     if (!C.capturesVariable())
2190:       continue;
2191:     const VarDecl *VD = C.getCapturedVar();
2192:     const auto *RD = VD->getType()
2193:                          .getCanonicalType()
2194:                          .getNonReferenceType()
2195:                          ->getAsCXXRecordDecl();
2196:     if (!RD || !RD->isLambda())
2197:       continue;
2198:     Address VDAddr = CGF.GetAddrOfLocalVar(VD);
2199:     LValue VDLVal;
2200:     if (VD->getType().getCanonicalType()->isReferenceType())
```
- **EN**: This block defines callable entry points like `SourceLocation`, `adjustTargetSpecificDataForLambdas`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `SourceLocation`, `adjustTargetSpecificDataForLambdas`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2201-2225
```cpp
2201:       VDLVal = CGF.EmitLoadOfReferenceLValue(VDAddr, VD->getType());
2202:     else
2203:       VDLVal = CGF.MakeAddrLValue(
2204:           VDAddr, VD->getType().getCanonicalType().getNonReferenceType());
2205:     llvm::DenseMap<const ValueDecl *, FieldDecl *> Captures;
2206:     FieldDecl *ThisCapture = nullptr;
2207:     RD->getCaptureFields(Captures, ThisCapture);
2208:     if (ThisCapture && CGF.CapturedStmtInfo->isCXXThisExprCaptured()) {
2209:       LValue ThisLVal =
2210:           CGF.EmitLValueForFieldInitialization(VDLVal, ThisCapture);
2211:       llvm::Value *CXXThis = CGF.LoadCXXThis();
2212:       CGF.EmitStoreOfScalar(CXXThis, ThisLVal);
2213:     }
2214:     for (const LambdaCapture &LC : RD->captures()) {
2215:       if (LC.getCaptureKind() != LCK_ByRef)
2216:         continue;
2217:       const ValueDecl *VD = LC.getCapturedVar();
2218:       // FIXME: For now VD is always a VarDecl because OpenMP does not support
2219:       //  capturing structured bindings in lambdas yet.
2220:       if (!CS->capturesVariable(cast<VarDecl>(VD)))
2221:         continue;
2222:       auto It = Captures.find(VD);
2223:       assert(It != Captures.end() && "Found lambda capture without field.");
2224:       LValue VarLVal = CGF.EmitLValueForFieldInitialization(VDLVal, It->second);
2225:       Address VDAddr = CGF.GetAddrOfLocalVar(cast<VarDecl>(VD));
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2226-2250
```cpp
2226:       if (VD->getType().getCanonicalType()->isReferenceType())
2227:         VDAddr = CGF.EmitLoadOfReferenceLValue(VDAddr,
2228:                                                VD->getType().getCanonicalType())
2229:                      .getAddress();
2230:       CGF.EmitStoreOfScalar(VDAddr.emitRawPointer(CGF), VarLVal);
2231:     }
2232:   }
2233: }
2234: 
2235: bool CGOpenMPRuntimeGPU::hasAllocateAttributeForGlobalVar(const VarDecl *VD,
2236:                                                             LangAS &AS) {
2237:   if (!VD || !VD->hasAttr<OMPAllocateDeclAttr>())
2238:     return false;
2239:   const auto *A = VD->getAttr<OMPAllocateDeclAttr>();
2240:   switch(A->getAllocatorType()) {
2241:   case OMPAllocateDeclAttr::OMPNullMemAlloc:
2242:   case OMPAllocateDeclAttr::OMPDefaultMemAlloc:
2243:   // Not supported, fallback to the default mem space.
2244:   case OMPAllocateDeclAttr::OMPLargeCapMemAlloc:
2245:   case OMPAllocateDeclAttr::OMPCGroupMemAlloc:
2246:   case OMPAllocateDeclAttr::OMPHighBWMemAlloc:
2247:   case OMPAllocateDeclAttr::OMPLowLatMemAlloc:
2248:   case OMPAllocateDeclAttr::OMPThreadMemAlloc:
2249:     AS = LangAS::Default;
2250:     return true;
```
- **EN**: This block defines callable entry points like `hasAllocateAttributeForGlobalVar`; uses control flow (if, switch, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `hasAllocateAttributeForGlobalVar`；通过控制流（if, switch, case）细化 LLVM IR 生成 行为。

### Lines 2251-2275
```cpp
2251:   case OMPAllocateDeclAttr::OMPConstMemAlloc:
2252:     AS = LangAS::cuda_constant;
2253:     return true;
2254:   case OMPAllocateDeclAttr::OMPPTeamMemAlloc:
2255:     AS = LangAS::cuda_shared;
2256:     return true;
2257:   case OMPAllocateDeclAttr::OMPUserDefinedMemAlloc:
2258:     llvm_unreachable("Expected predefined allocator for the variables with the "
2259:                      "static storage.");
2260:   }
2261:   return false;
2262: }
2263: 
2264: static OffloadArch getOffloadArch(const CodeGenModule &CGM) {
2265:   // FIXME: This should not require parsing
2266:   return StringToOffloadArch(CGM.getTarget().getTargetOpts().CPU);
2267: }
2268: 
2269: /// Check to see if target architecture supports unified addressing which is
2270: /// a restriction for OpenMP requires clause "unified_shared_memory".
2271: void CGOpenMPRuntimeGPU::processRequiresDirective(const OMPRequiresDecl *D) {
2272:   for (const OMPClause *Clause : D->clauselists()) {
2273:     if (Clause->getClauseKind() == OMPC_unified_shared_memory) {
2274:       OffloadArch Arch = getOffloadArch(CGM);
2275:       switch (Arch) {
```
- **EN**: This block defines callable entry points like `getOffloadArch`, `StringToOffloadArch`, `processRequiresDirective`; uses control flow (if, switch, for, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getOffloadArch`, `StringToOffloadArch`, `processRequiresDirective`；通过控制流（if, switch, for, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 2276-2300
```cpp
2276:       case OffloadArch::SM_20:
2277:       case OffloadArch::SM_21:
2278:       case OffloadArch::SM_30:
2279:       case OffloadArch::SM_32_:
2280:       case OffloadArch::SM_35:
2281:       case OffloadArch::SM_37:
2282:       case OffloadArch::SM_50:
2283:       case OffloadArch::SM_52:
2284:       case OffloadArch::SM_53: {
2285:         SmallString<256> Buffer;
2286:         llvm::raw_svector_ostream Out(Buffer);
2287:         Out << "Target architecture " << OffloadArchToString(Arch)
2288:             << " does not support unified addressing";
2289:         CGM.Error(Clause->getBeginLoc(), Out.str());
2290:         return;
2291:       }
2292:       case OffloadArch::SM_60:
2293:       case OffloadArch::SM_61:
2294:       case OffloadArch::SM_62:
2295:       case OffloadArch::SM_70:
2296:       case OffloadArch::SM_72:
2297:       case OffloadArch::SM_75:
2298:       case OffloadArch::SM_80:
2299:       case OffloadArch::SM_86:
2300:       case OffloadArch::SM_87:
```
- **EN**: This block defines callable entry points like `Out`; uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `Out`；通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2301-2325
```cpp
2301:       case OffloadArch::SM_88:
2302:       case OffloadArch::SM_89:
2303:       case OffloadArch::SM_90:
2304:       case OffloadArch::SM_90a:
2305:       case OffloadArch::SM_100:
2306:       case OffloadArch::SM_100a:
2307:       case OffloadArch::SM_100f:
2308:       case OffloadArch::SM_101:
2309:       case OffloadArch::SM_101a:
2310:       case OffloadArch::SM_101f:
2311:       case OffloadArch::SM_103:
2312:       case OffloadArch::SM_103a:
2313:       case OffloadArch::SM_103f:
2314:       case OffloadArch::SM_110:
2315:       case OffloadArch::SM_110a:
2316:       case OffloadArch::SM_110f:
2317:       case OffloadArch::SM_120:
2318:       case OffloadArch::SM_120a:
2319:       case OffloadArch::SM_120f:
2320:       case OffloadArch::SM_121:
2321:       case OffloadArch::SM_121a:
2322:       case OffloadArch::SM_121f:
2323:       case OffloadArch::GFX600:
2324:       case OffloadArch::GFX601:
2325:       case OffloadArch::GFX602:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2326-2350
```cpp
2326:       case OffloadArch::GFX700:
2327:       case OffloadArch::GFX701:
2328:       case OffloadArch::GFX702:
2329:       case OffloadArch::GFX703:
2330:       case OffloadArch::GFX704:
2331:       case OffloadArch::GFX705:
2332:       case OffloadArch::GFX801:
2333:       case OffloadArch::GFX802:
2334:       case OffloadArch::GFX803:
2335:       case OffloadArch::GFX805:
2336:       case OffloadArch::GFX810:
2337:       case OffloadArch::GFX9_GENERIC:
2338:       case OffloadArch::GFX900:
2339:       case OffloadArch::GFX902:
2340:       case OffloadArch::GFX904:
2341:       case OffloadArch::GFX906:
2342:       case OffloadArch::GFX908:
2343:       case OffloadArch::GFX909:
2344:       case OffloadArch::GFX90a:
2345:       case OffloadArch::GFX90c:
2346:       case OffloadArch::GFX9_4_GENERIC:
2347:       case OffloadArch::GFX942:
2348:       case OffloadArch::GFX950:
2349:       case OffloadArch::GFX10_1_GENERIC:
2350:       case OffloadArch::GFX1010:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2351-2375
```cpp
2351:       case OffloadArch::GFX1011:
2352:       case OffloadArch::GFX1012:
2353:       case OffloadArch::GFX1013:
2354:       case OffloadArch::GFX10_3_GENERIC:
2355:       case OffloadArch::GFX1030:
2356:       case OffloadArch::GFX1031:
2357:       case OffloadArch::GFX1032:
2358:       case OffloadArch::GFX1033:
2359:       case OffloadArch::GFX1034:
2360:       case OffloadArch::GFX1035:
2361:       case OffloadArch::GFX1036:
2362:       case OffloadArch::GFX11_GENERIC:
2363:       case OffloadArch::GFX1100:
2364:       case OffloadArch::GFX1101:
2365:       case OffloadArch::GFX1102:
2366:       case OffloadArch::GFX1103:
2367:       case OffloadArch::GFX1150:
2368:       case OffloadArch::GFX1151:
2369:       case OffloadArch::GFX1152:
2370:       case OffloadArch::GFX1153:
2371:       case OffloadArch::GFX1170:
2372:       case OffloadArch::GFX1171:
2373:       case OffloadArch::GFX1172:
2374:       case OffloadArch::GFX12_GENERIC:
2375:       case OffloadArch::GFX1200:
```
- **EN**: This block uses control flow (case) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（case）细化 LLVM IR 生成 行为。

### Lines 2376-2400
```cpp
2376:       case OffloadArch::GFX1201:
2377:       case OffloadArch::GFX1250:
2378:       case OffloadArch::GFX1251:
2379:       case OffloadArch::GFX12_5_GENERIC:
2380:       case OffloadArch::GFX1310:
2381:       case OffloadArch::AMDGCNSPIRV:
2382:       case OffloadArch::Generic:
2383:       case OffloadArch::GRANITERAPIDS:
2384:       case OffloadArch::BMG_G21:
2385:       case OffloadArch::Unused:
2386:       case OffloadArch::Unknown:
2387:         break;
2388:       }
2389:     }
2390:   }
2391:   CGOpenMPRuntime::processRequiresDirective(D);
2392: }
2393: 
2394: llvm::Value *CGOpenMPRuntimeGPU::getGPUNumThreads(CodeGenFunction &CGF) {
2395:   CGBuilderTy &Bld = CGF.Builder;
2396:   llvm::Module *M = &CGF.CGM.getModule();
2397:   const char *LocSize = "__kmpc_get_hardware_num_threads_in_block";
2398:   llvm::Function *F = M->getFunction(LocSize);
2399:   if (!F) {
2400:     F = llvm::Function::Create(llvm::FunctionType::get(CGF.Int32Ty, {}, false),
```
- **EN**: This block defines callable entry points like `processRequiresDirective`; uses control flow (if, case) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `processRequiresDirective`；通过控制流（if, case）细化 LLVM IR 生成 行为。

### Lines 2401-2413
```cpp
2401:                                llvm::GlobalVariable::ExternalLinkage, LocSize,
2402:                                &CGF.CGM.getModule());
2403:   }
2404:   return Bld.CreateCall(F, {}, "nvptx_num_threads");
2405: }
2406: 
2407: llvm::Value *CGOpenMPRuntimeGPU::getGPUThreadID(CodeGenFunction &CGF) {
2408:   ArrayRef<llvm::Value *> Args{};
2409:   return CGF.EmitRuntimeCall(
2410:       OMPBuilder.getOrCreateRuntimeFunction(
2411:           CGM.getModule(), OMPRTL___kmpc_get_hardware_thread_id_in_block),
2412:       Args);
2413: }
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

## Key Concepts / 关键概念

- **CGF**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **OffloadArch**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CodeGenFunction**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Loc**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ValueDecl**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGOpenMPRuntimeGPU.h`, `CGDebugInfo.h`, `CodeGenFunction.h`
- **Clang libraries / Clang 库**: `clang/AST/Attr.h`, `clang/AST/DeclOpenMP.h`, `clang/AST/OpenMPClause.h`, `clang/AST/StmtOpenMP.h`, `clang/AST/StmtVisitor.h`, `clang/Basic/Cuda.h`
- **LLVM libraries / LLVM 库**: `llvm/ADT/SmallPtrSet.h`, `llvm/Frontend/OpenMP/OMPDeviceConstants.h`, `llvm/Frontend/OpenMP/OMPGridValues.h`
