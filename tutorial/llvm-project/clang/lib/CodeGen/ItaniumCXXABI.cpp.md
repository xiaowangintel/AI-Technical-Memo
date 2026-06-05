# ItaniumCXXABI.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/ItaniumCXXABI.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the ItaniumCXXABI portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 ItaniumCXXABI 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
 1: //===------- ItaniumCXXABI.cpp - Emit LLVM Code from ASTs for a Module ----===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This provides C++ code generation targeting the Itanium C++ ABI.  The class
10: // in this file generates structures that follow the Itanium C++ ABI, which is
11: // documented at:
12: //  https://itanium-cxx-abi.github.io/cxx-abi/abi.html
13: //  https://itanium-cxx-abi.github.io/cxx-abi/abi-eh.html
14: //
15: // It also supports the closely-related ARM ABI, documented at:
16: // https://developer.arm.com/documentation/ihi0041/g/
17: //
18: //===----------------------------------------------------------------------===//
19: 
20: #include "CGCXXABI.h"
21: #include "CGCleanup.h"
22: #include "CGDebugInfo.h"
23: #include "CGRecordLayout.h"
24: #include "CGVTables.h"
25: #include "CodeGenFunction.h"
26: #include "CodeGenModule.h"
27: #include "TargetInfo.h"
28: #include "clang/AST/Attr.h"
29: #include "clang/AST/Mangle.h"
30: #include "clang/AST/StmtCXX.h"
```
- **EN**: This block imports local CodeGen headers `CGCXXABI.h`, `CGCleanup.h`, `CGDebugInfo.h`, and 5 more; Clang headers `clang/AST/Attr.h`, `clang/AST/Mangle.h`, `clang/AST/StmtCXX.h`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGCXXABI.h`, `CGCleanup.h`, `CGDebugInfo.h`, and 5 more；Clang 头文件 `clang/AST/Attr.h`, `clang/AST/Mangle.h`, `clang/AST/StmtCXX.h`；包含影响本编译单元构建方式的预处理结构。

### Lines 31-60
```cpp
31: #include "clang/AST/Type.h"
32: #include "clang/CodeGen/ConstantInitBuilder.h"
33: #include "llvm/IR/DataLayout.h"
34: #include "llvm/IR/GlobalValue.h"
35: #include "llvm/IR/Instructions.h"
36: #include "llvm/IR/Intrinsics.h"
37: #include "llvm/IR/Value.h"
38: #include "llvm/Support/ConvertEBCDIC.h"
39: #include "llvm/Support/ScopedPrinter.h"
40: 
41: #include <optional>
42: 
43: using namespace clang;
44: using namespace CodeGen;
45: 
46: namespace {
47: class ItaniumCXXABI : public CodeGen::CGCXXABI {
48:   /// VTables - All the vtables which have been defined.
49:   llvm::DenseMap<const CXXRecordDecl *, llvm::GlobalVariable *> VTables;
50: 
51:   /// All the thread wrapper functions that have been used.
52:   llvm::SmallVector<std::pair<const VarDecl *, llvm::Function *>, 8>
53:       ThreadWrappers;
54: 
55: protected:
56:   bool UseARMMethodPtrABI;
57:   bool UseARMGuardVarABI;
58:   bool Use32BitVTableOffsetABI;
59: 
60:   ItaniumMangleContext &getMangleContext() {
```
- **EN**: This block imports Clang headers `clang/AST/Type.h`, `clang/CodeGen/ConstantInitBuilder.h`; LLVM headers `llvm/IR/DataLayout.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/Instructions.h`, and 4 more; other headers `optional`; opens or references namespaces `clang`, `CodeGen`; introduces declarations such as `ItaniumCXXABI`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/AST/Type.h`, `clang/CodeGen/ConstantInitBuilder.h`；LLVM 头文件 `llvm/IR/DataLayout.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/Instructions.h`, and 4 more；其他头文件 `optional`；打开或引用命名空间 `clang`, `CodeGen`；给出诸如 `ItaniumCXXABI` 的声明；包含影响本编译单元构建方式的预处理结构。

### Lines 61-90
```cpp
61:     return cast<ItaniumMangleContext>(CodeGen::CGCXXABI::getMangleContext());
62:   }
63: 
64: public:
65:   ItaniumCXXABI(CodeGen::CodeGenModule &CGM,
66:                 bool UseARMMethodPtrABI = false,
67:                 bool UseARMGuardVarABI = false) :
68:     CGCXXABI(CGM), UseARMMethodPtrABI(UseARMMethodPtrABI),
69:     UseARMGuardVarABI(UseARMGuardVarABI),
70:     Use32BitVTableOffsetABI(false) { }
71: 
72:   bool classifyReturnType(CGFunctionInfo &FI) const override;
73: 
74:   RecordArgABI getRecordArgABI(const CXXRecordDecl *RD) const override {
75:     // If C++ prohibits us from making a copy, pass by address.
76:     if (!RD->canPassInRegisters())
77:       return RAA_Indirect;
78:     return RAA_Default;
79:   }
80: 
81:   bool isThisCompleteObject(GlobalDecl GD) const override {
82:     // The Itanium ABI has separate complete-object vs.  base-object
83:     // variants of both constructors and destructors.
84:     if (isa<CXXDestructorDecl>(GD.getDecl())) {
85:       switch (GD.getDtorType()) {
86:       case Dtor_Complete:
87:       case Dtor_Deleting:
88:         return true;
89: 
90:       case Dtor_Base:
```
- **EN**: This block defines callable entry points like `ItaniumCXXABI`, `classifyReturnType`, `getRecordArgABI`, `isThisCompleteObject`; uses control flow (if, switch, case) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `ItaniumCXXABI`, `classifyReturnType`, `getRecordArgABI`, `isThisCompleteObject`；通过控制流（if, switch, case）细化 ABI 降级 行为。

### Lines 91-120
```cpp
 91:         return false;
 92: 
 93:       case Dtor_Comdat:
 94:         llvm_unreachable("emitting dtor comdat as function?");
 95:       case Dtor_Unified:
 96:         llvm_unreachable("emitting unified dtor as function?");
 97:       case Dtor_VectorDeleting:
 98:         llvm_unreachable("unexpected dtor kind for this ABI");
 99:       }
100:       llvm_unreachable("bad dtor kind");
101:     }
102:     if (isa<CXXConstructorDecl>(GD.getDecl())) {
103:       switch (GD.getCtorType()) {
104:       case Ctor_Complete:
105:         return true;
106: 
107:       case Ctor_Base:
108:         return false;
109: 
110:       case Ctor_CopyingClosure:
111:       case Ctor_DefaultClosure:
112:         llvm_unreachable("closure ctors in Itanium ABI?");
113: 
114:       case Ctor_Comdat:
115:         llvm_unreachable("emitting ctor comdat as function?");
116: 
117:       case Ctor_Unified:
118:         llvm_unreachable("emitting unified ctor as function?");
119:       }
120:       llvm_unreachable("bad dtor kind");
```
- **EN**: This block uses control flow (if, switch, for, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, for, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 121-150
```cpp
121:     }
122: 
123:     // No other kinds.
124:     return false;
125:   }
126: 
127:   bool isZeroInitializable(const MemberPointerType *MPT) override;
128: 
129:   llvm::Type *ConvertMemberPointerType(const MemberPointerType *MPT) override;
130: 
131:   CGCallee
132:     EmitLoadOfMemberFunctionPointer(CodeGenFunction &CGF,
133:                                     const Expr *E,
134:                                     Address This,
135:                                     llvm::Value *&ThisPtrForCall,
136:                                     llvm::Value *MemFnPtr,
137:                                     const MemberPointerType *MPT) override;
138: 
139:   llvm::Value *EmitMemberDataPointerAddress(CodeGenFunction &CGF, const Expr *E,
140:                                             Address Base, llvm::Value *MemPtr,
141:                                             const MemberPointerType *MPT,
142:                                             bool IsInBounds) override;
143: 
144:   llvm::Value *EmitMemberPointerConversion(CodeGenFunction &CGF,
145:                                            const CastExpr *E,
146:                                            llvm::Value *Src) override;
147:   llvm::Constant *EmitMemberPointerConversion(const CastExpr *E,
148:                                               llvm::Constant *Src) override;
149: 
150:   llvm::Constant *EmitNullMemberPointer(const MemberPointerType *MPT) override;
```
- **EN**: This block spells out callable entry points like `isZeroInitializable`, `EmitLoadOfMemberFunctionPointer`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `isZeroInitializable`, `EmitLoadOfMemberFunctionPointer`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 151-180
```cpp
151: 
152:   llvm::Constant *EmitMemberFunctionPointer(const CXXMethodDecl *MD) override;
153:   llvm::Constant *EmitMemberDataPointer(const MemberPointerType *MPT,
154:                                         CharUnits offset) override;
155:   llvm::Constant *EmitMemberPointer(const APValue &MP, QualType MPT) override;
156:   llvm::Constant *BuildMemberPointer(const CXXMethodDecl *MD,
157:                                      CharUnits ThisAdjustment);
158: 
159:   llvm::Value *EmitMemberPointerComparison(CodeGenFunction &CGF,
160:                                            llvm::Value *L, llvm::Value *R,
161:                                            const MemberPointerType *MPT,
162:                                            bool Inequality) override;
163: 
164:   llvm::Value *EmitMemberPointerIsNotNull(CodeGenFunction &CGF,
165:                                          llvm::Value *Addr,
166:                                          const MemberPointerType *MPT) override;
167: 
168:   void emitVirtualObjectDelete(CodeGenFunction &CGF, const CXXDeleteExpr *DE,
169:                                Address Ptr, QualType ElementType,
170:                                const CXXDestructorDecl *Dtor) override;
171: 
172:   void emitRethrow(CodeGenFunction &CGF, bool isNoReturn) override;
173:   void emitThrow(CodeGenFunction &CGF, const CXXThrowExpr *E) override;
174: 
175:   void emitBeginCatch(CodeGenFunction &CGF, const CXXCatchStmt *C) override;
176: 
177:   llvm::CallInst *
178:   emitTerminateForUnexpectedException(CodeGenFunction &CGF,
179:                                       llvm::Value *Exn) override;
180: 
```
- **EN**: This block spells out callable entry points like `emitVirtualObjectDelete`, `emitRethrow`, `emitThrow`, `emitBeginCatch`, `emitTerminateForUnexpectedException`.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitVirtualObjectDelete`, `emitRethrow`, `emitThrow`, `emitBeginCatch`, `emitTerminateForUnexpectedException`。

### Lines 181-210
```cpp
181:   void EmitFundamentalRTTIDescriptors(const CXXRecordDecl *RD);
182:   llvm::Constant *getAddrOfRTTIDescriptor(QualType Ty) override;
183:   CatchTypeInfo
184:   getAddrOfCXXCatchHandlerType(QualType Ty,
185:                                QualType CatchHandlerType) override {
186:     return CatchTypeInfo{getAddrOfRTTIDescriptor(Ty), 0};
187:   }
188: 
189:   bool shouldTypeidBeNullChecked(QualType SrcRecordTy) override;
190:   void EmitBadTypeidCall(CodeGenFunction &CGF) override;
191:   llvm::Value *EmitTypeid(CodeGenFunction &CGF, QualType SrcRecordTy,
192:                           Address ThisPtr,
193:                           llvm::Type *StdTypeInfoPtrTy) override;
194: 
195:   bool shouldDynamicCastCallBeNullChecked(bool SrcIsPtr,
196:                                           QualType SrcRecordTy) override;
197: 
198:   /// Determine whether we know that all instances of type RecordTy will have
199:   /// the same vtable pointer values, that is distinct from all other vtable
200:   /// pointers. While this is required by the Itanium ABI, it doesn't happen in
201:   /// practice in some cases due to language extensions.
202:   bool hasUniqueVTablePointer(QualType RecordTy) {
203:     const CXXRecordDecl *RD = RecordTy->getAsCXXRecordDecl();
204: 
205:     // Under -fapple-kext, multiple definitions of the same vtable may be
206:     // emitted.
207:     if (!CGM.getCodeGenOpts().AssumeUniqueVTables ||
208:         getContext().getLangOpts().AppleKext)
209:       return false;
210: 
```
- **EN**: This block defines callable entry points like `EmitFundamentalRTTIDescriptors`, `getAddrOfCXXCatchHandlerType`, `shouldTypeidBeNullChecked`, `EmitBadTypeidCall`, `shouldDynamicCastCallBeNullChecked`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitFundamentalRTTIDescriptors`, `getAddrOfCXXCatchHandlerType`, `shouldTypeidBeNullChecked`, `EmitBadTypeidCall`, `shouldDynamicCastCallBeNullChecked`；通过控制流（if）细化 ABI 降级 行为。

### Lines 211-240
```cpp
211:     // If the type_info* would be null, the vtable might be merged with that of
212:     // another type.
213:     if (!CGM.shouldEmitRTTI())
214:       return false;
215: 
216:     // If there's only one definition of the vtable in the program, it has a
217:     // unique address.
218:     if (!llvm::GlobalValue::isWeakForLinker(CGM.getVTableLinkage(RD)))
219:       return true;
220: 
221:     // Even if there are multiple definitions of the vtable, they are required
222:     // by the ABI to use the same symbol name, so should be merged at load
223:     // time. However, if the class has hidden visibility, there can be
224:     // different versions of the class in different modules, and the ABI
225:     // library might treat them as being the same.
226:     if (CGM.GetLLVMVisibility(RD->getVisibility()) !=
227:         llvm::GlobalValue::DefaultVisibility)
228:       return false;
229: 
230:     return true;
231:   }
232: 
233:   bool shouldEmitExactDynamicCast(QualType DestRecordTy) override {
234:     return hasUniqueVTablePointer(DestRecordTy);
235:   }
236: 
237:   std::optional<ExactDynamicCastInfo>
238:   getExactDynamicCastInfo(QualType SrcRecordTy, QualType DestTy,
239:                           QualType DestRecordTy) override;
240: 
```
- **EN**: This block defines callable entry points like `shouldEmitExactDynamicCast`, `hasUniqueVTablePointer`, `getExactDynamicCastInfo`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `shouldEmitExactDynamicCast`, `hasUniqueVTablePointer`, `getExactDynamicCastInfo`；通过控制流（if）细化 ABI 降级 行为。

### Lines 241-270
```cpp
241:   llvm::Value *emitDynamicCastCall(CodeGenFunction &CGF, Address Value,
242:                                    QualType SrcRecordTy, QualType DestTy,
243:                                    QualType DestRecordTy,
244:                                    llvm::BasicBlock *CastEnd) override;
245: 
246:   llvm::Value *emitExactDynamicCast(CodeGenFunction &CGF, Address ThisAddr,
247:                                     QualType SrcRecordTy, QualType DestTy,
248:                                     QualType DestRecordTy,
249:                                     const ExactDynamicCastInfo &CastInfo,
250:                                     llvm::BasicBlock *CastSuccess,
251:                                     llvm::BasicBlock *CastFail) override;
252: 
253:   llvm::Value *emitDynamicCastToVoid(CodeGenFunction &CGF, Address Value,
254:                                      QualType SrcRecordTy) override;
255: 
256:   bool EmitBadCastCall(CodeGenFunction &CGF) override;
257: 
258:   llvm::Value *
259:     GetVirtualBaseClassOffset(CodeGenFunction &CGF, Address This,
260:                               const CXXRecordDecl *ClassDecl,
261:                               const CXXRecordDecl *BaseClassDecl) override;
262: 
263:   void EmitCXXConstructors(const CXXConstructorDecl *D) override;
264: 
265:   AddedStructorArgCounts
266:   buildStructorSignature(GlobalDecl GD,
267:                          SmallVectorImpl<CanQualType> &ArgTys) override;
268: 
269:   bool useThunkForDtorVariant(const CXXDestructorDecl *Dtor,
270:                               CXXDtorType DT) const override {
```
- **EN**: This block defines callable entry points like `EmitBadCastCall`, `GetVirtualBaseClassOffset`, `EmitCXXConstructors`, `buildStructorSignature`, `useThunkForDtorVariant`.
- **CN**: 该代码块定义可调用入口，例如 `EmitBadCastCall`, `GetVirtualBaseClassOffset`, `EmitCXXConstructors`, `buildStructorSignature`, `useThunkForDtorVariant`。

### Lines 271-300
```cpp
271:     // Itanium does not emit any destructor variant as an inline thunk.
272:     // Delegating may occur as an optimization, but all variants are either
273:     // emitted with external linkage or as linkonce if they are inline and used.
274:     return false;
275:   }
276: 
277:   void EmitCXXDestructors(const CXXDestructorDecl *D) override;
278: 
279:   void addImplicitStructorParams(CodeGenFunction &CGF, QualType &ResTy,
280:                                  FunctionArgList &Params) override;
281: 
282:   void EmitInstanceFunctionProlog(CodeGenFunction &CGF) override;
283: 
284:   AddedStructorArgs getImplicitConstructorArgs(CodeGenFunction &CGF,
285:                                                const CXXConstructorDecl *D,
286:                                                CXXCtorType Type,
287:                                                bool ForVirtualBase,
288:                                                bool Delegating) override;
289: 
290:   llvm::Value *getCXXDestructorImplicitParam(CodeGenFunction &CGF,
291:                                              const CXXDestructorDecl *DD,
292:                                              CXXDtorType Type,
293:                                              bool ForVirtualBase,
294:                                              bool Delegating) override;
295: 
296:   void EmitDestructorCall(CodeGenFunction &CGF, const CXXDestructorDecl *DD,
297:                           CXXDtorType Type, bool ForVirtualBase,
298:                           bool Delegating, Address This,
299:                           QualType ThisTy) override;
300: 
```
- **EN**: This block spells out callable entry points like `EmitCXXDestructors`, `addImplicitStructorParams`, `EmitInstanceFunctionProlog`, `getImplicitConstructorArgs`, `EmitDestructorCall`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitCXXDestructors`, `addImplicitStructorParams`, `EmitInstanceFunctionProlog`, `getImplicitConstructorArgs`, `EmitDestructorCall`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 301-330
```cpp
301:   void emitVTableDefinitions(CodeGenVTables &CGVT,
302:                              const CXXRecordDecl *RD) override;
303: 
304:   bool isVirtualOffsetNeededForVTableField(CodeGenFunction &CGF,
305:                                            CodeGenFunction::VPtr Vptr) override;
306: 
307:   bool doStructorsInitializeVPtrs(const CXXRecordDecl *VTableClass) override {
308:     return true;
309:   }
310: 
311:   llvm::Constant *
312:   getVTableAddressPoint(BaseSubobject Base,
313:                         const CXXRecordDecl *VTableClass) override;
314: 
315:   llvm::Value *getVTableAddressPointInStructor(
316:       CodeGenFunction &CGF, const CXXRecordDecl *VTableClass,
317:       BaseSubobject Base, const CXXRecordDecl *NearestVBase) override;
318: 
319:   llvm::Value *getVTableAddressPointInStructorWithVTT(
320:       CodeGenFunction &CGF, const CXXRecordDecl *VTableClass,
321:       BaseSubobject Base, const CXXRecordDecl *NearestVBase);
322: 
323:   llvm::GlobalVariable *getAddrOfVTable(const CXXRecordDecl *RD,
324:                                         CharUnits VPtrOffset) override;
325: 
326:   CGCallee getVirtualFunctionPointer(CodeGenFunction &CGF, GlobalDecl GD,
327:                                      Address This, llvm::Type *Ty,
328:                                      SourceLocation Loc) override;
329: 
330:   llvm::Value *
```
- **EN**: This block defines callable entry points like `emitVTableDefinitions`, `isVirtualOffsetNeededForVTableField`, `doStructorsInitializeVPtrs`, `getVTableAddressPoint`, `getVirtualFunctionPointer`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `emitVTableDefinitions`, `isVirtualOffsetNeededForVTableField`, `doStructorsInitializeVPtrs`, `getVTableAddressPoint`, `getVirtualFunctionPointer`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 331-360
```cpp
331:   EmitVirtualDestructorCall(CodeGenFunction &CGF, const CXXDestructorDecl *Dtor,
332:                             CXXDtorType DtorType, Address This,
333:                             DeleteOrMemberCallExpr E,
334:                             llvm::CallBase **CallOrInvoke) override;
335: 
336:   void emitVirtualInheritanceTables(const CXXRecordDecl *RD) override;
337: 
338:   bool canSpeculativelyEmitVTable(const CXXRecordDecl *RD) const override;
339:   bool canSpeculativelyEmitVTableAsBaseClass(const CXXRecordDecl *RD) const;
340: 
341:   void setThunkLinkage(llvm::Function *Thunk, bool ForVTable, GlobalDecl GD,
342:                        bool ReturnAdjustment) override {
343:     // Allow inlining of thunks by emitting them with available_externally
344:     // linkage together with vtables when needed.
345:     if (ForVTable && !Thunk->hasLocalLinkage())
346:       Thunk->setLinkage(llvm::GlobalValue::AvailableExternallyLinkage);
347:     CGM.setGVProperties(Thunk, GD);
348:   }
349: 
350:   bool exportThunk() override { return true; }
351: 
352:   llvm::Value *performThisAdjustment(CodeGenFunction &CGF, Address This,
353:                                      const CXXRecordDecl *UnadjustedThisClass,
354:                                      const ThunkInfo &TI) override;
355: 
356:   llvm::Value *performReturnAdjustment(CodeGenFunction &CGF, Address Ret,
357:                                        const CXXRecordDecl *UnadjustedRetClass,
358:                                        const ReturnAdjustment &RA) override;
359: 
360:   size_t getSrcArgforCopyCtor(const CXXConstructorDecl *,
```
- **EN**: This block defines callable entry points like `EmitVirtualDestructorCall`, `emitVirtualInheritanceTables`, `canSpeculativelyEmitVTable`, `canSpeculativelyEmitVTableAsBaseClass`, `setThunkLinkage`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitVirtualDestructorCall`, `emitVirtualInheritanceTables`, `canSpeculativelyEmitVTable`, `canSpeculativelyEmitVTableAsBaseClass`, `setThunkLinkage`；通过控制流（if）细化 ABI 降级 行为。

### Lines 361-390
```cpp
361:                               FunctionArgList &Args) const override {
362:     assert(!Args.empty() && "expected the arglist to not be empty!");
363:     return Args.size() - 1;
364:   }
365: 
366:   StringRef GetPureVirtualCallName() override { return "__cxa_pure_virtual"; }
367:   StringRef GetDeletedVirtualCallName() override
368:     { return "__cxa_deleted_virtual"; }
369: 
370:   CharUnits getArrayCookieSizeImpl(QualType elementType) override;
371:   Address InitializeArrayCookie(CodeGenFunction &CGF,
372:                                 Address NewPtr,
373:                                 llvm::Value *NumElements,
374:                                 const CXXNewExpr *expr,
375:                                 QualType ElementType) override;
376:   llvm::Value *readArrayCookieImpl(CodeGenFunction &CGF,
377:                                    Address allocPtr,
378:                                    CharUnits cookieSize) override;
379: 
380:   void EmitGuardedInit(CodeGenFunction &CGF, const VarDecl &D,
381:                        llvm::GlobalVariable *DeclPtr,
382:                        bool PerformInit) override;
383:   void registerGlobalDtor(CodeGenFunction &CGF, const VarDecl &D,
384:                           llvm::FunctionCallee dtor,
385:                           llvm::Constant *addr) override;
386: 
387:   llvm::Function *getOrCreateThreadLocalWrapper(const VarDecl *VD,
388:                                                 llvm::Value *Val);
389:   void EmitThreadLocalInitFuncs(
390:       CodeGenModule &CGM,
```
- **EN**: This block defines callable entry points like `GetPureVirtualCallName`, `GetDeletedVirtualCallName`, `getArrayCookieSizeImpl`, `InitializeArrayCookie`, `EmitGuardedInit`; returns or forwards computed values for the surrounding ABI lowering logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `GetPureVirtualCallName`, `GetDeletedVirtualCallName`, `getArrayCookieSizeImpl`, `InitializeArrayCookie`, `EmitGuardedInit`；为周围的 ABI 降级 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 391-420
```cpp
391:       ArrayRef<const VarDecl *> CXXThreadLocals,
392:       ArrayRef<llvm::Function *> CXXThreadLocalInits,
393:       ArrayRef<const VarDecl *> CXXThreadLocalInitVars) override;
394: 
395:   bool usesThreadWrapperFunction(const VarDecl *VD) const override {
396:     return !isEmittedWithConstantInitializer(VD) ||
397:            mayNeedDestruction(VD);
398:   }
399:   LValue EmitThreadLocalVarDeclLValue(CodeGenFunction &CGF, const VarDecl *VD,
400:                                       QualType LValType) override;
401: 
402:   bool NeedsVTTParameter(GlobalDecl GD) override;
403: 
404:   llvm::Constant *
405:   getOrCreateVirtualFunctionPointerThunk(const CXXMethodDecl *MD);
406: 
407:   /**************************** RTTI Uniqueness ******************************/
408: 
409: protected:
410:   /// Returns true if the ABI requires RTTI type_info objects to be unique
411:   /// across a program.
412:   virtual bool shouldRTTIBeUnique() const { return true; }
413: 
414: public:
415:   /// What sort of unique-RTTI behavior should we use?
416:   enum RTTIUniquenessKind {
417:     /// We are guaranteeing, or need to guarantee, that the RTTI string
418:     /// is unique.
419:     RUK_Unique,
420: 
```
- **EN**: This block introduces declarations such as `RTTIUniquenessKind`; defines callable entry points like `usesThreadWrapperFunction`, `mayNeedDestruction`, `EmitThreadLocalVarDeclLValue`, `NeedsVTTParameter`, `getOrCreateVirtualFunctionPointerThunk`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块给出诸如 `RTTIUniquenessKind` 的声明；定义可调用入口，例如 `usesThreadWrapperFunction`, `mayNeedDestruction`, `EmitThreadLocalVarDeclLValue`, `NeedsVTTParameter`, `getOrCreateVirtualFunctionPointerThunk`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 421-450
```cpp
421:     /// We are not guaranteeing uniqueness for the RTTI string, so we
422:     /// can demote to hidden visibility but must use string comparisons.
423:     RUK_NonUniqueHidden,
424: 
425:     /// We are not guaranteeing uniqueness for the RTTI string, so we
426:     /// have to use string comparisons, but we also have to emit it with
427:     /// non-hidden visibility.
428:     RUK_NonUniqueVisible
429:   };
430: 
431:   /// Return the required visibility status for the given type and linkage in
432:   /// the current ABI.
433:   RTTIUniquenessKind
434:   classifyRTTIUniqueness(QualType CanTy,
435:                          llvm::GlobalValue::LinkageTypes Linkage) const;
436:   friend class ItaniumRTTIBuilder;
437: 
438:   void emitCXXStructor(GlobalDecl GD) override;
439: 
440:   std::pair<llvm::Value *, const CXXRecordDecl *>
441:   LoadVTablePtr(CodeGenFunction &CGF, Address This,
442:                 const CXXRecordDecl *RD) override;
443: 
444:  private:
445:    llvm::Constant *
446:    getSignedVirtualMemberFunctionPointer(const CXXMethodDecl *MD);
447: 
448:    bool hasAnyUnusedVirtualInlineFunction(const CXXRecordDecl *RD) const {
449:      const auto &VtableLayout =
450:          CGM.getItaniumVTableContext().getVTableLayout(RD);
```
- **EN**: This block introduces declarations such as `ItaniumRTTIBuilder`; defines callable entry points like `classifyRTTIUniqueness`, `emitCXXStructor`, `LoadVTablePtr`, `getSignedVirtualMemberFunctionPointer`, `hasAnyUnusedVirtualInlineFunction`.
- **CN**: 该代码块给出诸如 `ItaniumRTTIBuilder` 的声明；定义可调用入口，例如 `classifyRTTIUniqueness`, `emitCXXStructor`, `LoadVTablePtr`, `getSignedVirtualMemberFunctionPointer`, `hasAnyUnusedVirtualInlineFunction`。

### Lines 451-480
```cpp
451: 
452:      for (const auto &VtableComponent : VtableLayout.vtable_components()) {
453:        // Skip empty slot.
454:        if (!VtableComponent.isUsedFunctionPointerKind())
455:          continue;
456: 
457:        const CXXMethodDecl *Method = VtableComponent.getFunctionDecl();
458:        const FunctionDecl *FD = Method->getDefinition();
459:        const bool IsInlined =
460:            Method->getCanonicalDecl()->isInlined() || (FD && FD->isInlined());
461:        if (!IsInlined)
462:          continue;
463: 
464:        StringRef Name = CGM.getMangledName(
465:            VtableComponent.getGlobalDecl(/*HasVectorDeletingDtors=*/false));
466:        auto *Entry = CGM.GetGlobalValue(Name);
467:        // This checks if virtual inline function has already been emitted.
468:        // Note that it is possible that this inline function would be emitted
469:        // after trying to emit vtable speculatively. Because of this we do
470:        // an extra pass after emitting all deferred vtables to find and emit
471:        // these vtables opportunistically.
472:        if (!Entry || Entry->isDeclaration())
473:          return true;
474:      }
475:      return false;
476:   }
477: 
478:   bool isVTableHidden(const CXXRecordDecl *RD) const {
479:     const auto &VtableLayout =
480:             CGM.getItaniumVTableContext().getVTableLayout(RD);
```
- **EN**: This block defines callable entry points like `isVTableHidden`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `isVTableHidden`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 481-510
```cpp
481: 
482:     for (const auto &VtableComponent : VtableLayout.vtable_components()) {
483:       if (VtableComponent.isRTTIKind()) {
484:         const CXXRecordDecl *RTTIDecl = VtableComponent.getRTTIDecl();
485:         if (RTTIDecl->getVisibility() == Visibility::HiddenVisibility)
486:           return true;
487:       } else if (VtableComponent.isUsedFunctionPointerKind()) {
488:         const CXXMethodDecl *Method = VtableComponent.getFunctionDecl();
489:         if (Method->getVisibility() == Visibility::HiddenVisibility &&
490:             !Method->isDefined())
491:           return true;
492:       }
493:     }
494:     return false;
495:   }
496: };
497: 
498: class ARMCXXABI : public ItaniumCXXABI {
499: public:
500:   ARMCXXABI(CodeGen::CodeGenModule &CGM) :
501:     ItaniumCXXABI(CGM, /*UseARMMethodPtrABI=*/true,
502:                   /*UseARMGuardVarABI=*/true) {}
503: 
504:   bool constructorsAndDestructorsReturnThis() const override { return true; }
505: 
506:   void EmitReturnFromThunk(CodeGenFunction &CGF, RValue RV,
507:                            QualType ResTy) override;
508: 
509:   CharUnits getArrayCookieSizeImpl(QualType elementType) override;
510:   Address InitializeArrayCookie(CodeGenFunction &CGF,
```
- **EN**: This block introduces declarations such as `ARMCXXABI`; defines callable entry points like `ARMCXXABI`, `EmitReturnFromThunk`, `getArrayCookieSizeImpl`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块给出诸如 `ARMCXXABI` 的声明；定义可调用入口，例如 `ARMCXXABI`, `EmitReturnFromThunk`, `getArrayCookieSizeImpl`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 511-540
```cpp
511:                                 Address NewPtr,
512:                                 llvm::Value *NumElements,
513:                                 const CXXNewExpr *expr,
514:                                 QualType ElementType) override;
515:   llvm::Value *readArrayCookieImpl(CodeGenFunction &CGF, Address allocPtr,
516:                                    CharUnits cookieSize) override;
517: };
518: 
519: class AppleARM64CXXABI : public ARMCXXABI {
520: public:
521:   AppleARM64CXXABI(CodeGen::CodeGenModule &CGM) : ARMCXXABI(CGM) {
522:     Use32BitVTableOffsetABI = true;
523:   }
524: 
525:   // ARM64 libraries are prepared for non-unique RTTI.
526:   bool shouldRTTIBeUnique() const override { return false; }
527: };
528: 
529: class FuchsiaCXXABI final : public ItaniumCXXABI {
530: public:
531:   explicit FuchsiaCXXABI(CodeGen::CodeGenModule &CGM)
532:       : ItaniumCXXABI(CGM) {}
533: 
534: private:
535:   bool constructorsAndDestructorsReturnThis() const override { return true; }
536: };
537: 
538: class WebAssemblyCXXABI final : public ItaniumCXXABI {
539: public:
540:   explicit WebAssemblyCXXABI(CodeGen::CodeGenModule &CGM)
```
- **EN**: This block introduces declarations such as `AppleARM64CXXABI`, `FuchsiaCXXABI`, `WebAssemblyCXXABI`; defines callable entry points like `AppleARM64CXXABI`, `shouldRTTIBeUnique`, `FuchsiaCXXABI`, `constructorsAndDestructorsReturnThis`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块给出诸如 `AppleARM64CXXABI`, `FuchsiaCXXABI`, `WebAssemblyCXXABI` 的声明；定义可调用入口，例如 `AppleARM64CXXABI`, `shouldRTTIBeUnique`, `FuchsiaCXXABI`, `constructorsAndDestructorsReturnThis`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 541-570
```cpp
541:       : ItaniumCXXABI(CGM, /*UseARMMethodPtrABI=*/true,
542:                       /*UseARMGuardVarABI=*/true) {}
543:   void emitBeginCatch(CodeGenFunction &CGF, const CXXCatchStmt *C) override;
544:   llvm::CallInst *
545:   emitTerminateForUnexpectedException(CodeGenFunction &CGF,
546:                                       llvm::Value *Exn) override;
547: 
548: private:
549:   bool constructorsAndDestructorsReturnThis() const override { return true; }
550:   bool canCallMismatchedFunctionType() const override { return false; }
551: };
552: 
553: class XLCXXABI final : public ItaniumCXXABI {
554: public:
555:   explicit XLCXXABI(CodeGen::CodeGenModule &CGM)
556:       : ItaniumCXXABI(CGM) {}
557: 
558:   void registerGlobalDtor(CodeGenFunction &CGF, const VarDecl &D,
559:                           llvm::FunctionCallee dtor,
560:                           llvm::Constant *addr) override;
561: 
562:   bool useSinitAndSterm() const override { return true; }
563: 
564: private:
565:   void emitCXXStermFinalizer(const VarDecl &D, llvm::Function *dtorStub,
566:                              llvm::Constant *addr);
567: };
568: }
569: 
570: CodeGen::CGCXXABI *CodeGen::CreateItaniumCXXABI(CodeGenModule &CGM) {
```
- **EN**: This block introduces declarations such as `XLCXXABI`; defines callable entry points like `ItaniumCXXABI`, `emitTerminateForUnexpectedException`, `constructorsAndDestructorsReturnThis`, `canCallMismatchedFunctionType`, `XLCXXABI`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块给出诸如 `XLCXXABI` 的声明；定义可调用入口，例如 `ItaniumCXXABI`, `emitTerminateForUnexpectedException`, `constructorsAndDestructorsReturnThis`, `canCallMismatchedFunctionType`, `XLCXXABI`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 571-600
```cpp
571:   switch (CGM.getContext().getCXXABIKind()) {
572:   // For IR-generation purposes, there's no significant difference
573:   // between the ARM and iOS ABIs.
574:   case TargetCXXABI::GenericARM:
575:   case TargetCXXABI::iOS:
576:   case TargetCXXABI::WatchOS:
577:     return new ARMCXXABI(CGM);
578: 
579:   case TargetCXXABI::AppleARM64:
580:     return new AppleARM64CXXABI(CGM);
581: 
582:   case TargetCXXABI::Fuchsia:
583:     return new FuchsiaCXXABI(CGM);
584: 
585:   // Note that AArch64 uses the generic ItaniumCXXABI class since it doesn't
586:   // include the other 32-bit ARM oddities: constructor/destructor return values
587:   // and array cookies.
588:   case TargetCXXABI::GenericAArch64:
589:     return new ItaniumCXXABI(CGM, /*UseARMMethodPtrABI=*/true,
590:                              /*UseARMGuardVarABI=*/true);
591: 
592:   case TargetCXXABI::GenericMIPS:
593:     return new ItaniumCXXABI(CGM, /*UseARMMethodPtrABI=*/true);
594: 
595:   case TargetCXXABI::WebAssembly:
596:     return new WebAssemblyCXXABI(CGM);
597: 
598:   case TargetCXXABI::XL:
599:     return new XLCXXABI(CGM);
600: 
```
- **EN**: This block defines callable entry points like `ARMCXXABI`, `AppleARM64CXXABI`, `FuchsiaCXXABI`, `ItaniumCXXABI`, `WebAssemblyCXXABI`; uses control flow (switch, case) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `ARMCXXABI`, `AppleARM64CXXABI`, `FuchsiaCXXABI`, `ItaniumCXXABI`, `WebAssemblyCXXABI`；通过控制流（switch, case）细化 ABI 降级 行为。

### Lines 601-630
```cpp
601:   case TargetCXXABI::GenericItanium:
602:     return new ItaniumCXXABI(CGM);
603: 
604:   case TargetCXXABI::Microsoft:
605:     llvm_unreachable("Microsoft ABI is not Itanium-based");
606:   }
607:   llvm_unreachable("bad ABI kind");
608: }
609: 
610: llvm::Type *
611: ItaniumCXXABI::ConvertMemberPointerType(const MemberPointerType *MPT) {
612:   if (MPT->isMemberDataPointer())
613:     return CGM.PtrDiffTy;
614:   return llvm::StructType::get(CGM.PtrDiffTy, CGM.PtrDiffTy);
615: }
616: 
617: /// In the Itanium and ARM ABIs, method pointers have the form:
618: ///   struct { ptrdiff_t ptr; ptrdiff_t adj; } memptr;
619: ///
620: /// In the Itanium ABI:
621: ///  - method pointers are virtual if (memptr.ptr & 1) is nonzero
622: ///  - the this-adjustment is (memptr.adj)
623: ///  - the virtual offset is (memptr.ptr - 1)
624: ///
625: /// In the ARM ABI:
626: ///  - method pointers are virtual if (memptr.adj & 1) is nonzero
627: ///  - the this-adjustment is (memptr.adj >> 1)
628: ///  - the virtual offset is (memptr.ptr)
629: /// ARM uses 'adj' for the virtual flag because Thumb functions
630: /// may be only single-byte aligned.
```
- **EN**: This block defines callable entry points like `ItaniumCXXABI`, `ConvertMemberPointerType`, `get`; uses control flow (if, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `ItaniumCXXABI`, `ConvertMemberPointerType`, `get`；通过控制流（if, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 631-660
```cpp
631: ///
632: /// If the member is virtual, the adjusted 'this' pointer points
633: /// to a vtable pointer from which the virtual offset is applied.
634: ///
635: /// If the member is non-virtual, memptr.ptr is the address of
636: /// the function to call.
637: CGCallee ItaniumCXXABI::EmitLoadOfMemberFunctionPointer(
638:     CodeGenFunction &CGF, const Expr *E, Address ThisAddr,
639:     llvm::Value *&ThisPtrForCall,
640:     llvm::Value *MemFnPtr, const MemberPointerType *MPT) {
641:   CGBuilderTy &Builder = CGF.Builder;
642: 
643:   const FunctionProtoType *FPT =
644:       MPT->getPointeeType()->castAs<FunctionProtoType>();
645:   auto *RD = MPT->getMostRecentCXXRecordDecl();
646: 
647:   llvm::Constant *ptrdiff_1 = llvm::ConstantInt::get(CGM.PtrDiffTy, 1);
648: 
649:   llvm::BasicBlock *FnVirtual = CGF.createBasicBlock("memptr.virtual");
650:   llvm::BasicBlock *FnNonVirtual = CGF.createBasicBlock("memptr.nonvirtual");
651:   llvm::BasicBlock *FnEnd = CGF.createBasicBlock("memptr.end");
652: 
653:   // Extract memptr.adj, which is in the second field.
654:   llvm::Value *RawAdj = Builder.CreateExtractValue(MemFnPtr, 1, "memptr.adj");
655: 
656:   // Compute the true adjustment.
657:   llvm::Value *Adj = RawAdj;
658:   if (UseARMMethodPtrABI)
659:     Adj = Builder.CreateAShr(Adj, ptrdiff_1, "memptr.adj.shifted");
660: 
```
- **EN**: This block defines callable entry points like `EmitLoadOfMemberFunctionPointer`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitLoadOfMemberFunctionPointer`；通过控制流（if）细化 ABI 降级 行为。

### Lines 661-690
```cpp
661:   // Apply the adjustment and cast back to the original struct type
662:   // for consistency.
663:   llvm::Value *This = ThisAddr.emitRawPointer(CGF);
664:   This = Builder.CreateInBoundsGEP(Builder.getInt8Ty(), This, Adj);
665:   ThisPtrForCall = This;
666: 
667:   // Load the function pointer.
668:   llvm::Value *FnAsInt = Builder.CreateExtractValue(MemFnPtr, 0, "memptr.ptr");
669: 
670:   // If the LSB in the function pointer is 1, the function pointer points to
671:   // a virtual function.
672:   llvm::Value *IsVirtual;
673:   if (UseARMMethodPtrABI)
674:     IsVirtual = Builder.CreateAnd(RawAdj, ptrdiff_1);
675:   else
676:     IsVirtual = Builder.CreateAnd(FnAsInt, ptrdiff_1);
677:   IsVirtual = Builder.CreateIsNotNull(IsVirtual, "memptr.isvirtual");
678:   Builder.CreateCondBr(IsVirtual, FnVirtual, FnNonVirtual);
679: 
680:   // In the virtual path, the adjustment left 'This' pointing to the
681:   // vtable of the correct base subobject.  The "function pointer" is an
682:   // offset within the vtable (+1 for the virtual flag on non-ARM).
683:   CGF.EmitBlock(FnVirtual);
684: 
685:   // Cast the adjusted this to a pointer to vtable pointer and load.
686:   llvm::Type *VTableTy = CGF.CGM.GlobalsInt8PtrTy;
687:   CharUnits VTablePtrAlign =
688:     CGF.CGM.getDynamicOffsetAlignment(ThisAddr.getAlignment(), RD,
689:                                       CGF.getPointerAlign());
690:   llvm::Value *VTable = CGF.GetVTablePtr(
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 691-720
```cpp
691:       Address(This, ThisAddr.getElementType(), VTablePtrAlign), VTableTy, RD);
692: 
693:   // Apply the offset.
694:   // On ARM64, to reserve extra space in virtual member function pointers,
695:   // we only pay attention to the low 32 bits of the offset.
696:   llvm::Value *VTableOffset = FnAsInt;
697:   if (!UseARMMethodPtrABI)
698:     VTableOffset = Builder.CreateSub(VTableOffset, ptrdiff_1);
699:   if (Use32BitVTableOffsetABI) {
700:     VTableOffset = Builder.CreateTrunc(VTableOffset, CGF.Int32Ty);
701:     VTableOffset = Builder.CreateZExt(VTableOffset, CGM.PtrDiffTy);
702:   }
703: 
704:   // Check the address of the function pointer if CFI on member function
705:   // pointers is enabled.
706:   llvm::Constant *CheckSourceLocation;
707:   llvm::Constant *CheckTypeDesc;
708:   bool ShouldEmitCFICheck = CGF.SanOpts.has(SanitizerKind::CFIMFCall) &&
709:                             CGM.HasHiddenLTOVisibility(RD);
710: 
711:   if (ShouldEmitCFICheck) {
712:     if (const auto *BinOp = dyn_cast<BinaryOperator>(E)) {
713:       if (BinOp->isPtrMemOp() &&
714:           BinOp->getRHS()
715:               ->getType()
716:               ->hasPointeeToCFIUncheckedCalleeFunctionType())
717:         ShouldEmitCFICheck = false;
718:     }
719:   }
720: 
```
- **EN**: This block defines callable entry points like `Address`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `Address`；通过控制流（if）细化 ABI 降级 行为。

### Lines 721-750
```cpp
721:   bool ShouldEmitVFEInfo = CGM.getCodeGenOpts().VirtualFunctionElimination &&
722:                            CGM.HasHiddenLTOVisibility(RD);
723:   // TODO: Update this name not to be restricted to WPD only
724:   // as we now emit the vtable info info for speculative devirtualization as
725:   // well.
726:   bool ShouldEmitWPDInfo =
727:       (CGM.getCodeGenOpts().WholeProgramVTables &&
728:        // Don't insert type tests if we are forcing public visibility.
729:        !CGM.AlwaysHasLTOVisibilityPublic(RD)) ||
730:       CGM.getCodeGenOpts().DevirtualizeSpeculatively;
731:   llvm::Value *VirtualFn = nullptr;
732: 
733:   {
734:     auto CheckOrdinal = SanitizerKind::SO_CFIMFCall;
735:     auto CheckHandler = SanitizerHandler::CFICheckFail;
736:     SanitizerDebugLocation SanScope(&CGF, {CheckOrdinal}, CheckHandler);
737: 
738:     llvm::Value *TypeId = nullptr;
739:     llvm::Value *CheckResult = nullptr;
740: 
741:     if (ShouldEmitCFICheck || ShouldEmitVFEInfo || ShouldEmitWPDInfo) {
742:       // If doing CFI, VFE or WPD, we will need the metadata node to check
743:       // against.
744:       llvm::Metadata *MD =
745:           CGM.CreateMetadataIdentifierForVirtualMemPtrType(QualType(MPT, 0));
746:       TypeId = llvm::MetadataAsValue::get(CGF.getLLVMContext(), MD);
747:     }
748: 
749:     if (ShouldEmitVFEInfo) {
750:       llvm::Value *VFPAddr =
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 751-780
```cpp
751:           Builder.CreateGEP(CGF.Int8Ty, VTable, VTableOffset);
752: 
753:       // If doing VFE, load from the vtable with a type.checked.load intrinsic
754:       // call. Note that we use the GEP to calculate the address to load from
755:       // and pass 0 as the offset to the intrinsic. This is because every
756:       // vtable slot of the correct type is marked with matching metadata, and
757:       // we know that the load must be from one of these slots.
758:       llvm::Value *CheckedLoad = Builder.CreateCall(
759:           CGM.getIntrinsic(llvm::Intrinsic::type_checked_load),
760:           {VFPAddr, llvm::ConstantInt::get(CGM.Int32Ty, 0), TypeId});
761:       CheckResult = Builder.CreateExtractValue(CheckedLoad, 1);
762:       VirtualFn = Builder.CreateExtractValue(CheckedLoad, 0);
763:     } else {
764:       // When not doing VFE, emit a normal load, as it allows more
765:       // optimisations than type.checked.load.
766:       if (ShouldEmitCFICheck || ShouldEmitWPDInfo) {
767:         llvm::Value *VFPAddr =
768:             Builder.CreateGEP(CGF.Int8Ty, VTable, VTableOffset);
769:         llvm::Intrinsic::ID IID = CGM.HasHiddenLTOVisibility(RD)
770:                                       ? llvm::Intrinsic::type_test
771:                                       : llvm::Intrinsic::public_type_test;
772: 
773:         CheckResult =
774:             Builder.CreateCall(CGM.getIntrinsic(IID), {VFPAddr, TypeId});
775:       }
776: 
777:       if (CGM.getLangOpts().RelativeCXXABIVTables) {
778:         VirtualFn = CGF.Builder.CreateCall(
779:             CGM.getIntrinsic(llvm::Intrinsic::load_relative,
780:                              {VTableOffset->getType()}),
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 781-810
```cpp
781:             {VTable, VTableOffset});
782:       } else {
783:         llvm::Value *VFPAddr =
784:             CGF.Builder.CreateGEP(CGF.Int8Ty, VTable, VTableOffset);
785:         VirtualFn = CGF.Builder.CreateAlignedLoad(CGF.DefaultPtrTy, VFPAddr,
786:                                                   CGF.getPointerAlign(),
787:                                                   "memptr.virtualfn");
788:       }
789:     }
790:     assert(VirtualFn && "Virtual fuction pointer not created!");
791:     assert((!ShouldEmitCFICheck || !ShouldEmitVFEInfo || !ShouldEmitWPDInfo ||
792:             CheckResult) &&
793:            "Check result required but not created!");
794: 
795:     if (ShouldEmitCFICheck) {
796:       // If doing CFI, emit the check.
797:       CheckSourceLocation = CGF.EmitCheckSourceLocation(E->getBeginLoc());
798:       CheckTypeDesc = CGF.EmitCheckTypeDescriptor(QualType(MPT, 0));
799:       llvm::Constant *StaticData[] = {
800:           llvm::ConstantInt::get(CGF.Int8Ty, CodeGenFunction::CFITCK_VMFCall),
801:           CheckSourceLocation,
802:           CheckTypeDesc,
803:       };
804: 
805:       if (CGM.getCodeGenOpts().SanitizeTrap.has(SanitizerKind::CFIMFCall)) {
806:         CGF.EmitTrapCheck(CheckResult, CheckHandler);
807:       } else {
808:         llvm::Value *AllVtables = llvm::MetadataAsValue::get(
809:             CGM.getLLVMContext(),
810:             llvm::MDString::get(CGM.getLLVMContext(), "all-vtables"));
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 811-840
```cpp
811:         llvm::Value *ValidVtable = Builder.CreateCall(
812:             CGM.getIntrinsic(llvm::Intrinsic::type_test), {VTable, AllVtables});
813:         CGF.EmitCheck(std::make_pair(CheckResult, CheckOrdinal), CheckHandler,
814:                       StaticData, {VTable, ValidVtable});
815:       }
816: 
817:       FnVirtual = Builder.GetInsertBlock();
818:     }
819:   } // End of sanitizer scope
820: 
821:   CGF.EmitBranch(FnEnd);
822: 
823:   // In the non-virtual path, the function pointer is actually a
824:   // function pointer.
825:   CGF.EmitBlock(FnNonVirtual);
826:   llvm::Value *NonVirtualFn =
827:       Builder.CreateIntToPtr(FnAsInt, CGF.DefaultPtrTy, "memptr.nonvirtualfn");
828: 
829:   // Check the function pointer if CFI on member function pointers is enabled.
830:   if (ShouldEmitCFICheck) {
831:     CXXRecordDecl *RD = MPT->getMostRecentCXXRecordDecl();
832:     if (RD->hasDefinition()) {
833:       auto CheckOrdinal = SanitizerKind::SO_CFIMFCall;
834:       auto CheckHandler = SanitizerHandler::CFICheckFail;
835:       SanitizerDebugLocation SanScope(&CGF, {CheckOrdinal}, CheckHandler);
836: 
837:       llvm::Constant *StaticData[] = {
838:           llvm::ConstantInt::get(CGF.Int8Ty, CodeGenFunction::CFITCK_NVMFCall),
839:           CheckSourceLocation,
840:           CheckTypeDesc,
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 841-870
```cpp
841:       };
842: 
843:       llvm::Value *Bit = Builder.getFalse();
844:       for (const CXXRecordDecl *Base : CGM.getMostBaseClasses(RD)) {
845:         llvm::Metadata *MD = CGM.CreateMetadataIdentifierForType(
846:             getContext().getMemberPointerType(MPT->getPointeeType(),
847:                                               /*Qualifier=*/std::nullopt,
848:                                               Base->getCanonicalDecl()));
849:         llvm::Value *TypeId =
850:             llvm::MetadataAsValue::get(CGF.getLLVMContext(), MD);
851: 
852:         llvm::Value *TypeTest =
853:             Builder.CreateCall(CGM.getIntrinsic(llvm::Intrinsic::type_test),
854:                                {NonVirtualFn, TypeId});
855:         Bit = Builder.CreateOr(Bit, TypeTest);
856:       }
857: 
858:       CGF.EmitCheck(std::make_pair(Bit, CheckOrdinal), CheckHandler, StaticData,
859:                     {NonVirtualFn, llvm::UndefValue::get(CGF.IntPtrTy)});
860: 
861:       FnNonVirtual = Builder.GetInsertBlock();
862:     }
863:   }
864: 
865:   // We're done.
866:   CGF.EmitBlock(FnEnd);
867:   llvm::PHINode *CalleePtr = Builder.CreatePHI(CGF.DefaultPtrTy, 2);
868:   CalleePtr->addIncoming(VirtualFn, FnVirtual);
869:   CalleePtr->addIncoming(NonVirtualFn, FnNonVirtual);
870: 
```
- **EN**: This block defines callable entry points like `getContext`, `get`; uses control flow (for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `get`；通过控制流（for）细化 ABI 降级 行为。

### Lines 871-900
```cpp
871:   CGPointerAuthInfo PointerAuth;
872: 
873:   if (const auto &Schema =
874:           CGM.getCodeGenOpts().PointerAuth.CXXMemberFunctionPointers) {
875:     llvm::PHINode *DiscriminatorPHI = Builder.CreatePHI(CGF.IntPtrTy, 2);
876:     DiscriminatorPHI->addIncoming(llvm::ConstantInt::get(CGF.IntPtrTy, 0),
877:                                   FnVirtual);
878:     const auto &AuthInfo =
879:         CGM.getMemberFunctionPointerAuthInfo(QualType(MPT, 0));
880:     assert(Schema.getKey() == AuthInfo.getKey() &&
881:            "Keys for virtual and non-virtual member functions must match");
882:     auto *NonVirtualDiscriminator = AuthInfo.getDiscriminator();
883:     DiscriminatorPHI->addIncoming(NonVirtualDiscriminator, FnNonVirtual);
884:     PointerAuth = CGPointerAuthInfo(
885:         Schema.getKey(), Schema.getAuthenticationMode(), Schema.isIsaPointer(),
886:         Schema.authenticatesNullValues(), DiscriminatorPHI);
887:   }
888: 
889:   CGCallee Callee(FPT, CalleePtr, PointerAuth);
890:   return Callee;
891: }
892: 
893: /// Compute an l-value by applying the given pointer-to-member to a
894: /// base object.
895: llvm::Value *ItaniumCXXABI::EmitMemberDataPointerAddress(
896:     CodeGenFunction &CGF, const Expr *E, Address Base, llvm::Value *MemPtr,
897:     const MemberPointerType *MPT, bool IsInBounds) {
898:   assert(MemPtr->getType() == CGM.PtrDiffTy);
899: 
900:   CGBuilderTy &Builder = CGF.Builder;
```
- **EN**: This block defines callable entry points like `Callee`; uses control flow (if, for) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Callee`；通过控制流（if, for）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 901-930
```cpp
901: 
902:   // Apply the offset.
903:   llvm::Value *BaseAddr = Base.emitRawPointer(CGF);
904:   return Builder.CreateGEP(CGF.Int8Ty, BaseAddr, MemPtr, "memptr.offset",
905:                            IsInBounds ? llvm::GEPNoWrapFlags::inBounds()
906:                                       : llvm::GEPNoWrapFlags::none());
907: }
908: 
909: // See if it's possible to return a constant signed pointer.
910: static llvm::Constant *pointerAuthResignConstant(
911:     llvm::Value *Ptr, const CGPointerAuthInfo &CurAuthInfo,
912:     const CGPointerAuthInfo &NewAuthInfo, CodeGenModule &CGM) {
913:   const auto *CPA = dyn_cast<llvm::ConstantPtrAuth>(Ptr);
914: 
915:   if (!CPA)
916:     return nullptr;
917: 
918:   assert(CPA->getKey()->getZExtValue() == CurAuthInfo.getKey() &&
919:          CPA->getAddrDiscriminator()->isNullValue() &&
920:          CPA->getDiscriminator() == CurAuthInfo.getDiscriminator() &&
921:          "unexpected key or discriminators");
922: 
923:   return CGM.getConstantSignedPointer(
924:       CPA->getPointer(), NewAuthInfo.getKey(), nullptr,
925:       cast<llvm::ConstantInt>(NewAuthInfo.getDiscriminator()));
926: }
927: 
928: /// Perform a bitcast, derived-to-base, or base-to-derived member pointer
929: /// conversion.
930: ///
```
- **EN**: This block defines callable entry points like `none`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `none`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 931-960
```cpp
931: /// Bitcast conversions are always a no-op under Itanium.
932: ///
933: /// Obligatory offset/adjustment diagram:
934: ///         <-- offset -->          <-- adjustment -->
935: ///   |--------------------------|----------------------|--------------------|
936: ///   ^Derived address point     ^Base address point    ^Member address point
937: ///
938: /// So when converting a base member pointer to a derived member pointer,
939: /// we add the offset to the adjustment because the address point has
940: /// decreased;  and conversely, when converting a derived MP to a base MP
941: /// we subtract the offset from the adjustment because the address point
942: /// has increased.
943: ///
944: /// The standard forbids (at compile time) conversion to and from
945: /// virtual bases, which is why we don't have to consider them here.
946: ///
947: /// The standard forbids (at run time) casting a derived MP to a base
948: /// MP when the derived MP does not point to a member of the base.
949: /// This is why -1 is a reasonable choice for null data member
950: /// pointers.
951: llvm::Value *
952: ItaniumCXXABI::EmitMemberPointerConversion(CodeGenFunction &CGF,
953:                                            const CastExpr *E,
954:                                            llvm::Value *src) {
955:   // Use constant emission if we can.
956:   if (isa<llvm::Constant>(src))
957:     return EmitMemberPointerConversion(E, cast<llvm::Constant>(src));
958: 
959:   assert(E->getCastKind() == CK_DerivedToBaseMemberPointer ||
960:          E->getCastKind() == CK_BaseToDerivedMemberPointer ||
```
- **EN**: This block defines callable entry points like `EmitMemberPointerConversion`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitMemberPointerConversion`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 961-990
```cpp
961:          E->getCastKind() == CK_ReinterpretMemberPointer);
962: 
963:   CGBuilderTy &Builder = CGF.Builder;
964:   QualType DstType = E->getType();
965: 
966:   if (DstType->isMemberFunctionPointerType()) {
967:     if (const auto &NewAuthInfo =
968:             CGM.getMemberFunctionPointerAuthInfo(DstType)) {
969:       QualType SrcType = E->getSubExpr()->getType();
970:       assert(SrcType->isMemberFunctionPointerType());
971:       const auto &CurAuthInfo = CGM.getMemberFunctionPointerAuthInfo(SrcType);
972:       llvm::Value *MemFnPtr = Builder.CreateExtractValue(src, 0, "memptr.ptr");
973:       llvm::Type *OrigTy = MemFnPtr->getType();
974: 
975:       llvm::BasicBlock *StartBB = Builder.GetInsertBlock();
976:       llvm::BasicBlock *ResignBB = CGF.createBasicBlock("resign");
977:       llvm::BasicBlock *MergeBB = CGF.createBasicBlock("merge");
978: 
979:       // Check whether we have a virtual offset or a pointer to a function.
980:       assert(UseARMMethodPtrABI && "ARM ABI expected");
981:       llvm::Value *Adj = Builder.CreateExtractValue(src, 1, "memptr.adj");
982:       llvm::Constant *Ptrdiff_1 = llvm::ConstantInt::get(CGM.PtrDiffTy, 1);
983:       llvm::Value *AndVal = Builder.CreateAnd(Adj, Ptrdiff_1);
984:       llvm::Value *IsVirtualOffset =
985:           Builder.CreateIsNotNull(AndVal, "is.virtual.offset");
986:       Builder.CreateCondBr(IsVirtualOffset, MergeBB, ResignBB);
987: 
988:       CGF.EmitBlock(ResignBB);
989:       llvm::Type *PtrTy = llvm::PointerType::getUnqual(CGM.getLLVMContext());
990:       MemFnPtr = Builder.CreateIntToPtr(MemFnPtr, PtrTy);
```
- **EN**: This block uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 991-1020
```cpp
 991:       MemFnPtr =
 992:           CGF.emitPointerAuthResign(MemFnPtr, SrcType, CurAuthInfo, NewAuthInfo,
 993:                                     isa<llvm::Constant>(src));
 994:       MemFnPtr = Builder.CreatePtrToInt(MemFnPtr, OrigTy);
 995:       llvm::Value *ResignedVal = Builder.CreateInsertValue(src, MemFnPtr, 0);
 996:       ResignBB = Builder.GetInsertBlock();
 997: 
 998:       CGF.EmitBlock(MergeBB);
 999:       llvm::PHINode *NewSrc = Builder.CreatePHI(src->getType(), 2);
1000:       NewSrc->addIncoming(src, StartBB);
1001:       NewSrc->addIncoming(ResignedVal, ResignBB);
1002:       src = NewSrc;
1003:     }
1004:   }
1005: 
1006:   // Under Itanium, reinterprets don't require any additional processing.
1007:   if (E->getCastKind() == CK_ReinterpretMemberPointer) return src;
1008: 
1009:   llvm::Constant *adj = getMemberPointerAdjustment(E);
1010:   if (!adj) return src;
1011: 
1012:   bool isDerivedToBase = (E->getCastKind() == CK_DerivedToBaseMemberPointer);
1013: 
1014:   const MemberPointerType *destTy =
1015:     E->getType()->castAs<MemberPointerType>();
1016: 
1017:   // For member data pointers, this is just a matter of adding the
1018:   // offset if the source is non-null.
1019:   if (destTy->isMemberDataPointer()) {
1020:     llvm::Value *dst;
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1021-1050
```cpp
1021:     if (isDerivedToBase)
1022:       dst = Builder.CreateNSWSub(src, adj, "adj");
1023:     else
1024:       dst = Builder.CreateNSWAdd(src, adj, "adj");
1025: 
1026:     // Null check.
1027:     llvm::Value *null = llvm::Constant::getAllOnesValue(src->getType());
1028:     llvm::Value *isNull = Builder.CreateICmpEQ(src, null, "memptr.isnull");
1029:     return Builder.CreateSelect(isNull, src, dst);
1030:   }
1031: 
1032:   // The this-adjustment is left-shifted by 1 on ARM.
1033:   if (UseARMMethodPtrABI) {
1034:     uint64_t offset = cast<llvm::ConstantInt>(adj)->getZExtValue();
1035:     offset <<= 1;
1036:     adj = llvm::ConstantInt::get(adj->getType(), offset);
1037:   }
1038: 
1039:   llvm::Value *srcAdj = Builder.CreateExtractValue(src, 1, "src.adj");
1040:   llvm::Value *dstAdj;
1041:   if (isDerivedToBase)
1042:     dstAdj = Builder.CreateNSWSub(srcAdj, adj, "adj");
1043:   else
1044:     dstAdj = Builder.CreateNSWAdd(srcAdj, adj, "adj");
1045: 
1046:   return Builder.CreateInsertValue(src, dstAdj, 1);
1047: }
1048: 
1049: static llvm::Constant *
1050: pointerAuthResignMemberFunctionPointer(llvm::Constant *Src, QualType DestType,
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1051-1080
```cpp
1051:                                        QualType SrcType, CodeGenModule &CGM) {
1052:   assert(DestType->isMemberFunctionPointerType() &&
1053:          SrcType->isMemberFunctionPointerType() &&
1054:          "member function pointers expected");
1055:   if (DestType == SrcType)
1056:     return Src;
1057: 
1058:   const auto &NewAuthInfo = CGM.getMemberFunctionPointerAuthInfo(DestType);
1059:   const auto &CurAuthInfo = CGM.getMemberFunctionPointerAuthInfo(SrcType);
1060: 
1061:   if (!NewAuthInfo && !CurAuthInfo)
1062:     return Src;
1063: 
1064:   llvm::Constant *MemFnPtr = Src->getAggregateElement(0u);
1065:   if (MemFnPtr->getNumOperands() == 0) {
1066:     // src must be a pair of null pointers.
1067:     assert(isa<llvm::ConstantInt>(MemFnPtr) && "constant int expected");
1068:     return Src;
1069:   }
1070: 
1071:   llvm::Constant *ConstPtr = pointerAuthResignConstant(
1072:       cast<llvm::User>(MemFnPtr)->getOperand(0), CurAuthInfo, NewAuthInfo, CGM);
1073:   ConstPtr = llvm::ConstantExpr::getPtrToInt(ConstPtr, MemFnPtr->getType());
1074:   return ConstantFoldInsertValueInstruction(Src, ConstPtr, 0);
1075: }
1076: 
1077: llvm::Constant *
1078: ItaniumCXXABI::EmitMemberPointerConversion(const CastExpr *E,
1079:                                            llvm::Constant *src) {
1080:   assert(E->getCastKind() == CK_DerivedToBaseMemberPointer ||
```
- **EN**: This block defines callable entry points like `ConstantFoldInsertValueInstruction`, `EmitMemberPointerConversion`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `ConstantFoldInsertValueInstruction`, `EmitMemberPointerConversion`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1081-1110
```cpp
1081:          E->getCastKind() == CK_BaseToDerivedMemberPointer ||
1082:          E->getCastKind() == CK_ReinterpretMemberPointer);
1083: 
1084:   QualType DstType = E->getType();
1085: 
1086:   if (DstType->isMemberFunctionPointerType())
1087:     src = pointerAuthResignMemberFunctionPointer(
1088:         src, DstType, E->getSubExpr()->getType(), CGM);
1089: 
1090:   // Under Itanium, reinterprets don't require any additional processing.
1091:   if (E->getCastKind() == CK_ReinterpretMemberPointer) return src;
1092: 
1093:   // If the adjustment is trivial, we don't need to do anything.
1094:   llvm::Constant *adj = getMemberPointerAdjustment(E);
1095:   if (!adj) return src;
1096: 
1097:   bool isDerivedToBase = (E->getCastKind() == CK_DerivedToBaseMemberPointer);
1098: 
1099:   const MemberPointerType *destTy =
1100:     E->getType()->castAs<MemberPointerType>();
1101: 
1102:   // For member data pointers, this is just a matter of adding the
1103:   // offset if the source is non-null.
1104:   if (destTy->isMemberDataPointer()) {
1105:     // null maps to null.
1106:     if (src->isAllOnesValue()) return src;
1107: 
1108:     if (isDerivedToBase)
1109:       return llvm::ConstantExpr::getNSWSub(src, adj);
1110:     else
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1111-1140
```cpp
1111:       return llvm::ConstantExpr::getNSWAdd(src, adj);
1112:   }
1113: 
1114:   // The this-adjustment is left-shifted by 1 on ARM.
1115:   if (UseARMMethodPtrABI) {
1116:     uint64_t offset = cast<llvm::ConstantInt>(adj)->getZExtValue();
1117:     offset <<= 1;
1118:     adj = llvm::ConstantInt::get(adj->getType(), offset);
1119:   }
1120: 
1121:   llvm::Constant *srcAdj = src->getAggregateElement(1);
1122:   llvm::Constant *dstAdj;
1123:   if (isDerivedToBase)
1124:     dstAdj = llvm::ConstantExpr::getNSWSub(srcAdj, adj);
1125:   else
1126:     dstAdj = llvm::ConstantExpr::getNSWAdd(srcAdj, adj);
1127: 
1128:   llvm::Constant *res = ConstantFoldInsertValueInstruction(src, dstAdj, 1);
1129:   assert(res != nullptr && "Folding must succeed");
1130:   return res;
1131: }
1132: 
1133: llvm::Constant *
1134: ItaniumCXXABI::EmitNullMemberPointer(const MemberPointerType *MPT) {
1135:   // Itanium C++ ABI 2.3:
1136:   //   A NULL pointer is represented as -1.
1137:   if (MPT->isMemberDataPointer())
1138:     return llvm::ConstantInt::get(CGM.PtrDiffTy, -1ULL, /*isSigned=*/true);
1139: 
1140:   llvm::Constant *Zero = llvm::ConstantInt::get(CGM.PtrDiffTy, 0);
```
- **EN**: This block defines callable entry points like `getNSWAdd`, `EmitNullMemberPointer`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getNSWAdd`, `EmitNullMemberPointer`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1141-1170
```cpp
1141:   llvm::Constant *Values[2] = { Zero, Zero };
1142:   return llvm::ConstantStruct::getAnon(Values);
1143: }
1144: 
1145: llvm::Constant *
1146: ItaniumCXXABI::EmitMemberDataPointer(const MemberPointerType *MPT,
1147:                                      CharUnits offset) {
1148:   // Itanium C++ ABI 2.3:
1149:   //   A pointer to data member is an offset from the base address of
1150:   //   the class object containing it, represented as a ptrdiff_t
1151:   return llvm::ConstantInt::get(CGM.PtrDiffTy, offset.getQuantity());
1152: }
1153: 
1154: llvm::Constant *
1155: ItaniumCXXABI::EmitMemberFunctionPointer(const CXXMethodDecl *MD) {
1156:   return BuildMemberPointer(MD, CharUnits::Zero());
1157: }
1158: 
1159: llvm::Constant *ItaniumCXXABI::BuildMemberPointer(const CXXMethodDecl *MD,
1160:                                                   CharUnits ThisAdjustment) {
1161:   assert(MD->isInstance() && "Member function must not be static!");
1162: 
1163:   CodeGenTypes &Types = CGM.getTypes();
1164: 
1165:   // Get the function pointer (or index if this is a virtual function).
1166:   llvm::Constant *MemPtr[2];
1167:   if (MD->isVirtual()) {
1168:     uint64_t Index = CGM.getItaniumVTableContext().getMethodVTableIndex(MD);
1169:     uint64_t VTableOffset;
1170:     if (CGM.getLangOpts().RelativeCXXABIVTables) {
```
- **EN**: This block defines callable entry points like `getAnon`, `EmitMemberDataPointer`, `get`, `EmitMemberFunctionPointer`, `BuildMemberPointer`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getAnon`, `EmitMemberDataPointer`, `get`, `EmitMemberFunctionPointer`, `BuildMemberPointer`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1171-1200
```cpp
1171:       // Multiply by 4-byte relative offsets.
1172:       VTableOffset = Index * 4;
1173:     } else {
1174:       const ASTContext &Context = getContext();
1175:       CharUnits PointerWidth = Context.toCharUnitsFromBits(
1176:           Context.getTargetInfo().getPointerWidth(LangAS::Default));
1177:       VTableOffset = Index * PointerWidth.getQuantity();
1178:     }
1179: 
1180:     if (UseARMMethodPtrABI) {
1181:       // ARM C++ ABI 3.2.1:
1182:       //   This ABI specifies that adj contains twice the this
1183:       //   adjustment, plus 1 if the member function is virtual. The
1184:       //   least significant bit of adj then makes exactly the same
1185:       //   discrimination as the least significant bit of ptr does for
1186:       //   Itanium.
1187: 
1188:       // We cannot use the Itanium ABI's representation for virtual member
1189:       // function pointers under pointer authentication because it would
1190:       // require us to store both the virtual offset and the constant
1191:       // discriminator in the pointer, which would be immediately vulnerable
1192:       // to attack.  Instead we introduce a thunk that does the virtual dispatch
1193:       // and store it as if it were a non-virtual member function.  This means
1194:       // that virtual function pointers may not compare equal anymore, but
1195:       // fortunately they aren't required to by the standard, and we do make
1196:       // a best-effort attempt to re-use the thunk.
1197:       //
1198:       // To support interoperation with code in which pointer authentication
1199:       // is disabled, derefencing a member function pointer must still handle
1200:       // the virtual case, but it can use a discriminator which should never
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1201-1230
```cpp
1201:       // be valid.
1202:       const auto &Schema =
1203:           CGM.getCodeGenOpts().PointerAuth.CXXMemberFunctionPointers;
1204:       if (Schema)
1205:         MemPtr[0] = llvm::ConstantExpr::getPtrToInt(
1206:             getSignedVirtualMemberFunctionPointer(MD), CGM.PtrDiffTy);
1207:       else
1208:         MemPtr[0] = llvm::ConstantInt::get(CGM.PtrDiffTy, VTableOffset);
1209:       // Don't set the LSB of adj to 1 if pointer authentication for member
1210:       // function pointers is enabled.
1211:       MemPtr[1] = llvm::ConstantInt::get(
1212:           CGM.PtrDiffTy, 2 * ThisAdjustment.getQuantity() + !Schema);
1213:     } else {
1214:       // Itanium C++ ABI 2.3:
1215:       //   For a virtual function, [the pointer field] is 1 plus the
1216:       //   virtual table offset (in bytes) of the function,
1217:       //   represented as a ptrdiff_t.
1218:       MemPtr[0] = llvm::ConstantInt::get(CGM.PtrDiffTy, VTableOffset + 1);
1219:       MemPtr[1] = llvm::ConstantInt::get(CGM.PtrDiffTy,
1220:                                          ThisAdjustment.getQuantity());
1221:     }
1222:   } else {
1223:     const FunctionProtoType *FPT = MD->getType()->castAs<FunctionProtoType>();
1224:     llvm::Type *Ty;
1225:     // Check whether the function has a computable LLVM signature.
1226:     if (Types.isFuncTypeConvertible(FPT)) {
1227:       // The function has a computable LLVM signature; use the correct type.
1228:       Ty = Types.GetFunctionType(Types.arrangeCXXMethodDeclaration(MD));
1229:     } else {
1230:       // Use an arbitrary non-function type to tell GetAddrOfFunction that the
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1231-1260
```cpp
1231:       // function type is incomplete.
1232:       Ty = CGM.PtrDiffTy;
1233:     }
1234:     llvm::Constant *addr = CGM.getMemberFunctionPointer(MD, Ty);
1235: 
1236:     MemPtr[0] = llvm::ConstantExpr::getPtrToInt(addr, CGM.PtrDiffTy);
1237:     MemPtr[1] = llvm::ConstantInt::get(CGM.PtrDiffTy,
1238:                                        (UseARMMethodPtrABI ? 2 : 1) *
1239:                                        ThisAdjustment.getQuantity());
1240:   }
1241: 
1242:   return llvm::ConstantStruct::getAnon(MemPtr);
1243: }
1244: 
1245: llvm::Constant *ItaniumCXXABI::EmitMemberPointer(const APValue &MP,
1246:                                                  QualType MPType) {
1247:   const MemberPointerType *MPT = MPType->castAs<MemberPointerType>();
1248:   const ValueDecl *MPD = MP.getMemberPointerDecl();
1249:   if (!MPD)
1250:     return EmitNullMemberPointer(MPT);
1251: 
1252:   CharUnits ThisAdjustment = getContext().getMemberPointerPathAdjustment(MP);
1253: 
1254:   if (const CXXMethodDecl *MD = dyn_cast<CXXMethodDecl>(MPD)) {
1255:     llvm::Constant *Src = BuildMemberPointer(MD, ThisAdjustment);
1256:     QualType SrcType = getContext().getMemberPointerType(
1257:         MD->getType(), /*Qualifier=*/std::nullopt, MD->getParent());
1258:     return pointerAuthResignMemberFunctionPointer(Src, MPType, SrcType, CGM);
1259:   }
1260: 
```
- **EN**: This block defines callable entry points like `getAnon`, `pointerAuthResignMemberFunctionPointer`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getAnon`, `pointerAuthResignMemberFunctionPointer`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1261-1290
```cpp
1261:   getContext().recordMemberDataPointerEvaluation(MPD);
1262:   CharUnits FieldOffset =
1263:     getContext().toCharUnitsFromBits(getContext().getFieldOffset(MPD));
1264:   return EmitMemberDataPointer(MPT, ThisAdjustment + FieldOffset);
1265: }
1266: 
1267: /// The comparison algorithm is pretty easy: the member pointers are
1268: /// the same if they're either bitwise identical *or* both null.
1269: ///
1270: /// ARM is different here only because null-ness is more complicated.
1271: llvm::Value *
1272: ItaniumCXXABI::EmitMemberPointerComparison(CodeGenFunction &CGF,
1273:                                            llvm::Value *L,
1274:                                            llvm::Value *R,
1275:                                            const MemberPointerType *MPT,
1276:                                            bool Inequality) {
1277:   CGBuilderTy &Builder = CGF.Builder;
1278: 
1279:   llvm::ICmpInst::Predicate Eq;
1280:   llvm::Instruction::BinaryOps And, Or;
1281:   if (Inequality) {
1282:     Eq = llvm::ICmpInst::ICMP_NE;
1283:     And = llvm::Instruction::Or;
1284:     Or = llvm::Instruction::And;
1285:   } else {
1286:     Eq = llvm::ICmpInst::ICMP_EQ;
1287:     And = llvm::Instruction::And;
1288:     Or = llvm::Instruction::Or;
1289:   }
1290: 
```
- **EN**: This block defines callable entry points like `getContext`, `EmitMemberDataPointer`, `EmitMemberPointerComparison`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `EmitMemberDataPointer`, `EmitMemberPointerComparison`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1291-1320
```cpp
1291:   // Member data pointers are easy because there's a unique null
1292:   // value, so it just comes down to bitwise equality.
1293:   if (MPT->isMemberDataPointer())
1294:     return Builder.CreateICmp(Eq, L, R);
1295: 
1296:   // For member function pointers, the tautologies are more complex.
1297:   // The Itanium tautology is:
1298:   //   (L == R) <==> (L.ptr == R.ptr && (L.ptr == 0 || L.adj == R.adj))
1299:   // The ARM tautology is:
1300:   //   (L == R) <==> (L.ptr == R.ptr &&
1301:   //                  (L.adj == R.adj ||
1302:   //                   (L.ptr == 0 && ((L.adj|R.adj) & 1) == 0)))
1303:   // The inequality tautologies have exactly the same structure, except
1304:   // applying De Morgan's laws.
1305: 
1306:   llvm::Value *LPtr = Builder.CreateExtractValue(L, 0, "lhs.memptr.ptr");
1307:   llvm::Value *RPtr = Builder.CreateExtractValue(R, 0, "rhs.memptr.ptr");
1308: 
1309:   // This condition tests whether L.ptr == R.ptr.  This must always be
1310:   // true for equality to hold.
1311:   llvm::Value *PtrEq = Builder.CreateICmp(Eq, LPtr, RPtr, "cmp.ptr");
1312: 
1313:   // This condition, together with the assumption that L.ptr == R.ptr,
1314:   // tests whether the pointers are both null.  ARM imposes an extra
1315:   // condition.
1316:   llvm::Value *Zero = llvm::Constant::getNullValue(LPtr->getType());
1317:   llvm::Value *EqZero = Builder.CreateICmp(Eq, LPtr, Zero, "cmp.ptr.null");
1318: 
1319:   // This condition tests whether L.adj == R.adj.  If this isn't
1320:   // true, the pointers are unequal unless they're both null.
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1321-1350
```cpp
1321:   llvm::Value *LAdj = Builder.CreateExtractValue(L, 1, "lhs.memptr.adj");
1322:   llvm::Value *RAdj = Builder.CreateExtractValue(R, 1, "rhs.memptr.adj");
1323:   llvm::Value *AdjEq = Builder.CreateICmp(Eq, LAdj, RAdj, "cmp.adj");
1324: 
1325:   // Null member function pointers on ARM clear the low bit of Adj,
1326:   // so the zero condition has to check that neither low bit is set.
1327:   if (UseARMMethodPtrABI) {
1328:     llvm::Value *One = llvm::ConstantInt::get(LPtr->getType(), 1);
1329: 
1330:     // Compute (l.adj | r.adj) & 1 and test it against zero.
1331:     llvm::Value *OrAdj = Builder.CreateOr(LAdj, RAdj, "or.adj");
1332:     llvm::Value *OrAdjAnd1 = Builder.CreateAnd(OrAdj, One);
1333:     llvm::Value *OrAdjAnd1EqZero = Builder.CreateICmp(Eq, OrAdjAnd1, Zero,
1334:                                                       "cmp.or.adj");
1335:     EqZero = Builder.CreateBinOp(And, EqZero, OrAdjAnd1EqZero);
1336:   }
1337: 
1338:   // Tie together all our conditions.
1339:   llvm::Value *Result = Builder.CreateBinOp(Or, EqZero, AdjEq);
1340:   Result = Builder.CreateBinOp(And, PtrEq, Result,
1341:                                Inequality ? "memptr.ne" : "memptr.eq");
1342:   return Result;
1343: }
1344: 
1345: llvm::Value *
1346: ItaniumCXXABI::EmitMemberPointerIsNotNull(CodeGenFunction &CGF,
1347:                                           llvm::Value *MemPtr,
1348:                                           const MemberPointerType *MPT) {
1349:   CGBuilderTy &Builder = CGF.Builder;
1350: 
```
- **EN**: This block defines callable entry points like `EmitMemberPointerIsNotNull`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitMemberPointerIsNotNull`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1351-1380
```cpp
1351:   /// For member data pointers, this is just a check against -1.
1352:   if (MPT->isMemberDataPointer()) {
1353:     assert(MemPtr->getType() == CGM.PtrDiffTy);
1354:     llvm::Value *NegativeOne =
1355:       llvm::Constant::getAllOnesValue(MemPtr->getType());
1356:     return Builder.CreateICmpNE(MemPtr, NegativeOne, "memptr.tobool");
1357:   }
1358: 
1359:   // In Itanium, a member function pointer is not null if 'ptr' is not null.
1360:   llvm::Value *Ptr = Builder.CreateExtractValue(MemPtr, 0, "memptr.ptr");
1361: 
1362:   llvm::Constant *Zero = llvm::ConstantInt::get(Ptr->getType(), 0);
1363:   llvm::Value *Result = Builder.CreateICmpNE(Ptr, Zero, "memptr.tobool");
1364: 
1365:   // On ARM, a member function pointer is also non-null if the low bit of 'adj'
1366:   // (the virtual bit) is set.
1367:   if (UseARMMethodPtrABI) {
1368:     llvm::Constant *One = llvm::ConstantInt::get(Ptr->getType(), 1);
1369:     llvm::Value *Adj = Builder.CreateExtractValue(MemPtr, 1, "memptr.adj");
1370:     llvm::Value *VirtualBit = Builder.CreateAnd(Adj, One, "memptr.virtualbit");
1371:     llvm::Value *IsVirtual = Builder.CreateICmpNE(VirtualBit, Zero,
1372:                                                   "memptr.isvirtual");
1373:     Result = Builder.CreateOr(Result, IsVirtual);
1374:   }
1375: 
1376:   return Result;
1377: }
1378: 
1379: bool ItaniumCXXABI::classifyReturnType(CGFunctionInfo &FI) const {
1380:   const CXXRecordDecl *RD = FI.getReturnType()->getAsCXXRecordDecl();
```
- **EN**: This block defines callable entry points like `getAllOnesValue`, `classifyReturnType`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getAllOnesValue`, `classifyReturnType`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1381-1410
```cpp
1381:   if (!RD)
1382:     return false;
1383: 
1384:   // If C++ prohibits us from making a copy, return by address using the target
1385:   // hook getSRetAddrSpace to decide the AS.
1386:   if (!RD->canPassInRegisters()) {
1387:     auto Align = CGM.getContext().getTypeAlignInChars(FI.getReturnType());
1388:     LangAS SRetAS = CGM.getTargetCodeGenInfo().getSRetAddrSpace(RD);
1389:     unsigned AS = CGM.getContext().getTargetAddressSpace(SRetAS);
1390:     FI.getReturnInfo() =
1391:         ABIArgInfo::getIndirect(Align, /*AddrSpace=*/AS, /*ByVal=*/false);
1392:     return true;
1393:   }
1394:   return false;
1395: }
1396: 
1397: /// The Itanium ABI requires non-zero initialization only for data
1398: /// member pointers, for which '0' is a valid offset.
1399: bool ItaniumCXXABI::isZeroInitializable(const MemberPointerType *MPT) {
1400:   return MPT->isMemberFunctionPointer();
1401: }
1402: 
1403: /// The Itanium ABI always places an offset to the complete object
1404: /// at entry -2 in the vtable.
1405: void ItaniumCXXABI::emitVirtualObjectDelete(CodeGenFunction &CGF,
1406:                                             const CXXDeleteExpr *DE,
1407:                                             Address Ptr,
1408:                                             QualType ElementType,
1409:                                             const CXXDestructorDecl *Dtor) {
1410:   bool UseGlobalDelete = DE->isGlobalDelete();
```
- **EN**: This block defines callable entry points like `getIndirect`, `isZeroInitializable`, `emitVirtualObjectDelete`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getIndirect`, `isZeroInitializable`, `emitVirtualObjectDelete`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1411-1440
```cpp
1411:   if (UseGlobalDelete) {
1412:     // Derive the complete-object pointer, which is what we need
1413:     // to pass to the deallocation function.
1414: 
1415:     // Grab the vtable pointer as an intptr_t*.
1416:     auto *ClassDecl = ElementType->castAsCXXRecordDecl();
1417:     llvm::Value *VTable = CGF.GetVTablePtr(Ptr, CGF.DefaultPtrTy, ClassDecl);
1418: 
1419:     // Track back to entry -2 and pull out the offset there.
1420:     llvm::Value *OffsetPtr = CGF.Builder.CreateConstInBoundsGEP1_64(
1421:         CGF.IntPtrTy, VTable, -2, "complete-offset.ptr");
1422:     llvm::Value *Offset = CGF.Builder.CreateAlignedLoad(CGF.IntPtrTy, OffsetPtr,
1423:                                                         CGF.getPointerAlign());
1424: 
1425:     // Apply the offset.
1426:     llvm::Value *CompletePtr = Ptr.emitRawPointer(CGF);
1427:     CompletePtr =
1428:         CGF.Builder.CreateInBoundsGEP(CGF.Int8Ty, CompletePtr, Offset);
1429: 
1430:     // If we're supposed to call the global delete, make sure we do so
1431:     // even if the destructor throws.
1432:     CGF.pushCallObjectDeleteCleanup(DE->getOperatorDelete(), CompletePtr,
1433:                                     ElementType);
1434:   }
1435: 
1436:   // FIXME: Provide a source location here even though there's no
1437:   // CXXMemberCallExpr for dtor call.
1438:   CXXDtorType DtorType = UseGlobalDelete ? Dtor_Complete : Dtor_Deleting;
1439:   EmitVirtualDestructorCall(CGF, Dtor, DtorType, Ptr, DE,
1440:                             /*CallOrInvoke=*/nullptr);
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1441-1470
```cpp
1441: 
1442:   if (UseGlobalDelete)
1443:     CGF.PopCleanupBlock();
1444: }
1445: 
1446: void ItaniumCXXABI::emitRethrow(CodeGenFunction &CGF, bool isNoReturn) {
1447:   // void __cxa_rethrow();
1448: 
1449:   llvm::FunctionType *FTy =
1450:     llvm::FunctionType::get(CGM.VoidTy, /*isVarArg=*/false);
1451: 
1452:   llvm::FunctionCallee Fn = CGM.CreateRuntimeFunction(FTy, "__cxa_rethrow");
1453: 
1454:   if (isNoReturn)
1455:     CGF.EmitNoreturnRuntimeCallOrInvoke(Fn, {});
1456:   else
1457:     CGF.EmitRuntimeCallOrInvoke(Fn);
1458: }
1459: 
1460: static llvm::FunctionCallee getAllocateExceptionFn(CodeGenModule &CGM) {
1461:   // void *__cxa_allocate_exception(size_t thrown_size);
1462: 
1463:   llvm::FunctionType *FTy =
1464:     llvm::FunctionType::get(CGM.Int8PtrTy, CGM.SizeTy, /*isVarArg=*/false);
1465: 
1466:   return CGM.CreateRuntimeFunction(FTy, "__cxa_allocate_exception");
1467: }
1468: 
1469: static llvm::FunctionCallee getThrowFn(CodeGenModule &CGM) {
1470:   // void __cxa_throw(void *thrown_exception, std::type_info *tinfo,
```
- **EN**: This block defines callable entry points like `emitRethrow`, `get`, `getAllocateExceptionFn`, `getThrowFn`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitRethrow`, `get`, `getAllocateExceptionFn`, `getThrowFn`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1471-1500
```cpp
1471:   //                  void (*dest) (void *));
1472: 
1473:   llvm::Type *Args[3] = { CGM.Int8PtrTy, CGM.GlobalsInt8PtrTy, CGM.Int8PtrTy };
1474:   llvm::FunctionType *FTy =
1475:     llvm::FunctionType::get(CGM.VoidTy, Args, /*isVarArg=*/false);
1476: 
1477:   return CGM.CreateRuntimeFunction(FTy, "__cxa_throw");
1478: }
1479: 
1480: void ItaniumCXXABI::emitThrow(CodeGenFunction &CGF, const CXXThrowExpr *E) {
1481:   QualType ThrowType = E->getSubExpr()->getType();
1482:   // Now allocate the exception object.
1483:   llvm::Type *SizeTy = CGF.ConvertType(getContext().getSizeType());
1484:   uint64_t TypeSize = getContext().getTypeSizeInChars(ThrowType).getQuantity();
1485: 
1486:   llvm::FunctionCallee AllocExceptionFn = getAllocateExceptionFn(CGM);
1487:   llvm::CallInst *ExceptionPtr = CGF.EmitNounwindRuntimeCall(
1488:       AllocExceptionFn, llvm::ConstantInt::get(SizeTy, TypeSize), "exception");
1489: 
1490:   CharUnits ExnAlign = CGF.getContext().getExnObjectAlignment();
1491:   CGF.EmitAnyExprToExn(
1492:       E->getSubExpr(), Address(ExceptionPtr, CGM.Int8Ty, ExnAlign));
1493: 
1494:   // Now throw the exception.
1495:   llvm::Constant *TypeInfo = CGM.GetAddrOfRTTIDescriptor(ThrowType,
1496:                                                          /*ForEH=*/true);
1497: 
1498:   // The address of the destructor.  If the exception type has a
1499:   // trivial destructor (or isn't a record), we just pass null.
1500:   llvm::Constant *Dtor = nullptr;
```
- **EN**: This block defines callable entry points like `get`, `emitThrow`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `get`, `emitThrow`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 1501-1530
```cpp
1501:   if (const auto *Record = ThrowType->getAsCXXRecordDecl();
1502:       Record && !Record->hasTrivialDestructor()) {
1503:     // __cxa_throw is declared to take its destructor as void (*)(void *). We
1504:     // must match that if function pointers can be authenticated with a
1505:     // discriminator based on their type.
1506:     const ASTContext &Ctx = getContext();
1507:     QualType DtorTy = Ctx.getFunctionType(Ctx.VoidTy, {Ctx.VoidPtrTy},
1508:                                           FunctionProtoType::ExtProtoInfo());
1509: 
1510:     CXXDestructorDecl *DtorD = Record->getDestructor();
1511:     Dtor = CGM.getAddrOfCXXStructor(GlobalDecl(DtorD, Dtor_Complete));
1512:     Dtor = CGM.getFunctionPointer(Dtor, DtorTy);
1513:   }
1514:   if (!Dtor) Dtor = llvm::Constant::getNullValue(CGM.Int8PtrTy);
1515: 
1516:   llvm::Value *args[] = { ExceptionPtr, TypeInfo, Dtor };
1517:   CGF.EmitNoreturnRuntimeCallOrInvoke(getThrowFn(CGM), args);
1518: }
1519: 
1520: static llvm::FunctionCallee getItaniumDynamicCastFn(CodeGenFunction &CGF) {
1521:   // void *__dynamic_cast(const void *sub,
1522:   //                      global_as const abi::__class_type_info *src,
1523:   //                      global_as const abi::__class_type_info *dst,
1524:   //                      std::ptrdiff_t src2dst_offset);
1525: 
1526:   llvm::Type *Int8PtrTy = CGF.Int8PtrTy;
1527:   llvm::Type *GlobInt8PtrTy = CGF.GlobalsInt8PtrTy;
1528:   llvm::Type *PtrDiffTy =
1529:     CGF.ConvertType(CGF.getContext().getPointerDiffType());
1530: 
```
- **EN**: This block defines callable entry points like `ExtProtoInfo`, `getItaniumDynamicCastFn`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `ExtProtoInfo`, `getItaniumDynamicCastFn`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1531-1560
```cpp
1531:   llvm::Type *Args[4] = { Int8PtrTy, GlobInt8PtrTy, GlobInt8PtrTy, PtrDiffTy };
1532: 
1533:   llvm::FunctionType *FTy = llvm::FunctionType::get(Int8PtrTy, Args, false);
1534: 
1535:   // Mark the function as nounwind willreturn readonly.
1536:   llvm::AttrBuilder FuncAttrs(CGF.getLLVMContext());
1537:   FuncAttrs.addAttribute(llvm::Attribute::NoUnwind);
1538:   FuncAttrs.addAttribute(llvm::Attribute::WillReturn);
1539:   FuncAttrs.addMemoryAttr(llvm::MemoryEffects::readOnly());
1540:   llvm::AttributeList Attrs = llvm::AttributeList::get(
1541:       CGF.getLLVMContext(), llvm::AttributeList::FunctionIndex, FuncAttrs);
1542: 
1543:   return CGF.CGM.CreateRuntimeFunction(FTy, "__dynamic_cast", Attrs);
1544: }
1545: 
1546: static llvm::FunctionCallee getBadCastFn(CodeGenFunction &CGF) {
1547:   // void __cxa_bad_cast();
1548:   llvm::FunctionType *FTy = llvm::FunctionType::get(CGF.VoidTy, false);
1549:   return CGF.CGM.CreateRuntimeFunction(FTy, "__cxa_bad_cast");
1550: }
1551: 
1552: /// Compute the src2dst_offset hint as described in the
1553: /// Itanium C++ ABI [2.9.7]
1554: static CharUnits computeOffsetHint(ASTContext &Context,
1555:                                    const CXXRecordDecl *Src,
1556:                                    const CXXRecordDecl *Dst) {
1557:   CXXBasePaths Paths(/*FindAmbiguities=*/true, /*RecordPaths=*/true,
1558:                      /*DetectVirtual=*/false);
1559: 
1560:   // If Dst is not derived from Src we can skip the whole computation below and
```
- **EN**: This block defines callable entry points like `FuncAttrs`, `getBadCastFn`, `computeOffsetHint`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `FuncAttrs`, `getBadCastFn`, `computeOffsetHint`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 1561-1590
```cpp
1561:   // return that Src is not a public base of Dst.  Record all inheritance paths.
1562:   if (!Dst->isDerivedFrom(Src, Paths))
1563:     return CharUnits::fromQuantity(-2ULL);
1564: 
1565:   unsigned NumPublicPaths = 0;
1566:   CharUnits Offset;
1567: 
1568:   // Now walk all possible inheritance paths.
1569:   for (const CXXBasePath &Path : Paths) {
1570:     if (Path.Access != AS_public)  // Ignore non-public inheritance.
1571:       continue;
1572: 
1573:     ++NumPublicPaths;
1574: 
1575:     for (const CXXBasePathElement &PathElement : Path) {
1576:       // If the path contains a virtual base class we can't give any hint.
1577:       // -1: no hint.
1578:       if (PathElement.Base->isVirtual())
1579:         return CharUnits::fromQuantity(-1ULL);
1580: 
1581:       if (NumPublicPaths > 1) // Won't use offsets, skip computation.
1582:         continue;
1583: 
1584:       // Accumulate the base class offsets.
1585:       const ASTRecordLayout &L = Context.getASTRecordLayout(PathElement.Class);
1586:       Offset += L.getBaseClassOffset(
1587:           PathElement.Base->getType()->getAsCXXRecordDecl());
1588:     }
1589:   }
1590: 
```
- **EN**: This block uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if, for）细化 ABI 降级 行为。

### Lines 1591-1620
```cpp
1591:   // -2: Src is not a public base of Dst.
1592:   if (NumPublicPaths == 0)
1593:     return CharUnits::fromQuantity(-2ULL);
1594: 
1595:   // -3: Src is a multiple public base type but never a virtual base type.
1596:   if (NumPublicPaths > 1)
1597:     return CharUnits::fromQuantity(-3ULL);
1598: 
1599:   // Otherwise, the Src type is a unique public nonvirtual base type of Dst.
1600:   // Return the offset of Src from the origin of Dst.
1601:   return Offset;
1602: }
1603: 
1604: static llvm::FunctionCallee getBadTypeidFn(CodeGenFunction &CGF) {
1605:   // void __cxa_bad_typeid();
1606:   llvm::FunctionType *FTy = llvm::FunctionType::get(CGF.VoidTy, false);
1607: 
1608:   return CGF.CGM.CreateRuntimeFunction(FTy, "__cxa_bad_typeid");
1609: }
1610: 
1611: bool ItaniumCXXABI::shouldTypeidBeNullChecked(QualType SrcRecordTy) {
1612:   return true;
1613: }
1614: 
1615: void ItaniumCXXABI::EmitBadTypeidCall(CodeGenFunction &CGF) {
1616:   llvm::FunctionCallee Fn = getBadTypeidFn(CGF);
1617:   llvm::CallBase *Call = CGF.EmitRuntimeCallOrInvoke(Fn);
1618:   Call->setDoesNotReturn();
1619:   CGF.Builder.CreateUnreachable();
1620: }
```
- **EN**: This block defines callable entry points like `getBadTypeidFn`, `shouldTypeidBeNullChecked`, `EmitBadTypeidCall`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getBadTypeidFn`, `shouldTypeidBeNullChecked`, `EmitBadTypeidCall`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1621-1650
```cpp
1621: 
1622: llvm::Value *ItaniumCXXABI::EmitTypeid(CodeGenFunction &CGF,
1623:                                        QualType SrcRecordTy,
1624:                                        Address ThisPtr,
1625:                                        llvm::Type *StdTypeInfoPtrTy) {
1626:   auto *ClassDecl = SrcRecordTy->castAsCXXRecordDecl();
1627:   llvm::Value *Value = CGF.GetVTablePtr(ThisPtr, CGM.GlobalsInt8PtrTy,
1628:                                         ClassDecl);
1629: 
1630:   if (CGM.getLangOpts().RelativeCXXABIVTables) {
1631:     // Load the type info.
1632:     Value = CGF.Builder.CreateCall(
1633:         CGM.getIntrinsic(llvm::Intrinsic::load_relative, {CGM.Int32Ty}),
1634:         {Value, llvm::ConstantInt::getSigned(CGM.Int32Ty, -4)});
1635:   } else {
1636:     // Load the type info.
1637:     Value =
1638:         CGF.Builder.CreateConstInBoundsGEP1_64(StdTypeInfoPtrTy, Value, -1ULL);
1639:   }
1640:   return CGF.Builder.CreateAlignedLoad(StdTypeInfoPtrTy, Value,
1641:                                        CGF.getPointerAlign());
1642: }
1643: 
1644: bool ItaniumCXXABI::shouldDynamicCastCallBeNullChecked(bool SrcIsPtr,
1645:                                                        QualType SrcRecordTy) {
1646:   return SrcIsPtr;
1647: }
1648: 
1649: llvm::Value *ItaniumCXXABI::emitDynamicCastCall(
1650:     CodeGenFunction &CGF, Address ThisAddr, QualType SrcRecordTy,
```
- **EN**: This block defines callable entry points like `shouldDynamicCastCallBeNullChecked`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `shouldDynamicCastCallBeNullChecked`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1651-1680
```cpp
1651:     QualType DestTy, QualType DestRecordTy, llvm::BasicBlock *CastEnd) {
1652:   llvm::Type *PtrDiffLTy =
1653:       CGF.ConvertType(CGF.getContext().getPointerDiffType());
1654: 
1655:   llvm::Value *SrcRTTI =
1656:       CGF.CGM.GetAddrOfRTTIDescriptor(SrcRecordTy.getUnqualifiedType());
1657:   llvm::Value *DestRTTI =
1658:       CGF.CGM.GetAddrOfRTTIDescriptor(DestRecordTy.getUnqualifiedType());
1659: 
1660:   // Compute the offset hint.
1661:   const CXXRecordDecl *SrcDecl = SrcRecordTy->getAsCXXRecordDecl();
1662:   const CXXRecordDecl *DestDecl = DestRecordTy->getAsCXXRecordDecl();
1663:   llvm::Value *OffsetHint = llvm::ConstantInt::getSigned(
1664:       PtrDiffLTy,
1665:       computeOffsetHint(CGF.getContext(), SrcDecl, DestDecl).getQuantity());
1666: 
1667:   // Emit the call to __dynamic_cast.
1668:   llvm::Value *Value = ThisAddr.emitRawPointer(CGF);
1669:   if (CGM.getCodeGenOpts().PointerAuth.CXXVTablePointers) {
1670:     // We perform a no-op load of the vtable pointer here to force an
1671:     // authentication. In environments that do not support pointer
1672:     // authentication this is a an actual no-op that will be elided. When
1673:     // pointer authentication is supported and enforced on vtable pointers this
1674:     // load can trap.
1675:     llvm::Value *Vtable =
1676:         CGF.GetVTablePtr(ThisAddr, CGM.Int8PtrTy, SrcDecl,
1677:                          CodeGenFunction::VTableAuthMode::MustTrap);
1678:     assert(Vtable);
1679:     (void)Vtable;
1680:   }
```
- **EN**: This block defines callable entry points like `computeOffsetHint`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `computeOffsetHint`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1681-1710
```cpp
1681: 
1682:   llvm::Value *args[] = {Value, SrcRTTI, DestRTTI, OffsetHint};
1683:   Value = CGF.EmitNounwindRuntimeCall(getItaniumDynamicCastFn(CGF), args);
1684: 
1685:   /// C++ [expr.dynamic.cast]p9:
1686:   ///   A failed cast to reference type throws std::bad_cast
1687:   if (DestTy->isReferenceType()) {
1688:     llvm::BasicBlock *BadCastBlock =
1689:         CGF.createBasicBlock("dynamic_cast.bad_cast");
1690: 
1691:     llvm::Value *IsNull = CGF.Builder.CreateIsNull(Value);
1692:     CGF.Builder.CreateCondBr(IsNull, BadCastBlock, CastEnd);
1693: 
1694:     CGF.EmitBlock(BadCastBlock);
1695:     EmitBadCastCall(CGF);
1696:   }
1697: 
1698:   return Value;
1699: }
1700: 
1701: std::optional<CGCXXABI::ExactDynamicCastInfo>
1702: ItaniumCXXABI::getExactDynamicCastInfo(QualType SrcRecordTy, QualType DestTy,
1703:                                        QualType DestRecordTy) {
1704:   assert(shouldEmitExactDynamicCast(DestRecordTy));
1705: 
1706:   ASTContext &Context = getContext();
1707: 
1708:   // Find all the inheritance paths.
1709:   const CXXRecordDecl *SrcDecl = SrcRecordTy->getAsCXXRecordDecl();
1710:   const CXXRecordDecl *DestDecl = DestRecordTy->getAsCXXRecordDecl();
```
- **EN**: This block defines callable entry points like `EmitBadCastCall`, `getExactDynamicCastInfo`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitBadCastCall`, `getExactDynamicCastInfo`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1711-1740
```cpp
1711:   CXXBasePaths Paths(/*FindAmbiguities=*/true, /*RecordPaths=*/true,
1712:                      /*DetectVirtual=*/false);
1713:   (void)DestDecl->isDerivedFrom(SrcDecl, Paths);
1714: 
1715:   // Find an offset within `DestDecl` where a `SrcDecl` instance and its vptr
1716:   // might appear.
1717:   std::optional<CharUnits> Offset;
1718:   for (const CXXBasePath &Path : Paths) {
1719:     // dynamic_cast only finds public inheritance paths.
1720:     if (Path.Access != AS_public)
1721:       continue;
1722: 
1723:     CharUnits PathOffset;
1724:     for (const CXXBasePathElement &PathElement : Path) {
1725:       // Find the offset along this inheritance step.
1726:       const CXXRecordDecl *Base =
1727:           PathElement.Base->getType()->getAsCXXRecordDecl();
1728:       if (PathElement.Base->isVirtual()) {
1729:         // For a virtual base class, we know that the derived class is exactly
1730:         // DestDecl, so we can use the vbase offset from its layout.
1731:         const ASTRecordLayout &L = Context.getASTRecordLayout(DestDecl);
1732:         PathOffset = L.getVBaseClassOffset(Base);
1733:       } else {
1734:         const ASTRecordLayout &L =
1735:             Context.getASTRecordLayout(PathElement.Class);
1736:         PathOffset += L.getBaseClassOffset(Base);
1737:       }
1738:     }
1739: 
1740:     if (!Offset)
```
- **EN**: This block defines callable entry points like `Paths`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `Paths`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 1741-1770
```cpp
1741:       Offset = PathOffset;
1742:     else if (Offset != PathOffset) {
1743:       // Base appears in at least two different places.
1744:       return ExactDynamicCastInfo{/*RequiresCastToPrimaryBase=*/true,
1745:                                   CharUnits::Zero()};
1746:     }
1747:   }
1748:   if (!Offset)
1749:     return std::nullopt;
1750:   return ExactDynamicCastInfo{/*RequiresCastToPrimaryBase=*/false, *Offset};
1751: }
1752: 
1753: llvm::Value *ItaniumCXXABI::emitExactDynamicCast(
1754:     CodeGenFunction &CGF, Address ThisAddr, QualType SrcRecordTy,
1755:     QualType DestTy, QualType DestRecordTy,
1756:     const ExactDynamicCastInfo &ExactCastInfo, llvm::BasicBlock *CastSuccess,
1757:     llvm::BasicBlock *CastFail) {
1758:   const CXXRecordDecl *SrcDecl = SrcRecordTy->getAsCXXRecordDecl();
1759:   const CXXRecordDecl *DestDecl = DestRecordTy->getAsCXXRecordDecl();
1760:   auto AuthenticateVTable = [&](Address ThisAddr, const CXXRecordDecl *Decl) {
1761:     if (!CGF.getLangOpts().PointerAuthCalls)
1762:       return;
1763:     (void)CGF.GetVTablePtr(ThisAddr, CGF.DefaultPtrTy, Decl,
1764:                            CodeGenFunction::VTableAuthMode::MustTrap);
1765:   };
1766: 
1767:   bool PerformPostCastAuthentication = false;
1768:   llvm::Value *VTable = nullptr;
1769:   if (ExactCastInfo.RequiresCastToPrimaryBase) {
1770:     // Base appears in at least two different places. Find the most-derived
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1771-1800
```cpp
1771:     // object and see if it's a DestDecl. Note that the most-derived object
1772:     // must be at least as aligned as this base class subobject, and must
1773:     // have a vptr at offset 0.
1774:     llvm::Value *PrimaryBase =
1775:         emitDynamicCastToVoid(CGF, ThisAddr, SrcRecordTy);
1776:     ThisAddr = Address(PrimaryBase, CGF.VoidPtrTy, ThisAddr.getAlignment());
1777:     SrcDecl = DestDecl;
1778:     // This unauthenticated load is unavoidable, so we're relying on the
1779:     // authenticated load in the dynamic cast to void, and we'll manually
1780:     // authenticate the resulting v-table at the end of the cast check.
1781:     PerformPostCastAuthentication = CGF.getLangOpts().PointerAuthCalls;
1782:     CGPointerAuthInfo StrippingAuthInfo(0, PointerAuthenticationMode::Strip,
1783:                                         false, false, nullptr);
1784:     Address VTablePtrPtr = ThisAddr.withElementType(CGF.VoidPtrPtrTy);
1785:     VTable = CGF.Builder.CreateLoad(VTablePtrPtr, "vtable");
1786:     if (PerformPostCastAuthentication)
1787:       VTable = CGF.EmitPointerAuthAuth(StrippingAuthInfo, VTable);
1788:   } else
1789:     VTable = CGF.GetVTablePtr(ThisAddr, CGF.DefaultPtrTy, SrcDecl);
1790: 
1791:   // Compare the vptr against the expected vptr for the destination type at
1792:   // this offset.
1793:   llvm::Constant *ExpectedVTable = getVTableAddressPoint(
1794:       BaseSubobject(SrcDecl, ExactCastInfo.Offset), DestDecl);
1795:   llvm::Value *Success = CGF.Builder.CreateICmpEQ(VTable, ExpectedVTable);
1796:   llvm::Value *AdjustedThisPtr = ThisAddr.emitRawPointer(CGF);
1797: 
1798:   if (!ExactCastInfo.Offset.isZero()) {
1799:     CharUnits::QuantityType Offset = ExactCastInfo.Offset.getQuantity();
1800:     llvm::Constant *OffsetConstant =
```
- **EN**: This block defines callable entry points like `emitDynamicCastToVoid`, `StrippingAuthInfo`, `BaseSubobject`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitDynamicCastToVoid`, `StrippingAuthInfo`, `BaseSubobject`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1801-1830
```cpp
1801:         llvm::ConstantInt::get(CGF.PtrDiffTy, -Offset);
1802:     AdjustedThisPtr = CGF.Builder.CreateInBoundsGEP(CGF.CharTy, AdjustedThisPtr,
1803:                                                     OffsetConstant);
1804:     PerformPostCastAuthentication = CGF.getLangOpts().PointerAuthCalls;
1805:   }
1806: 
1807:   if (PerformPostCastAuthentication) {
1808:     // If we've changed the object pointer we authenticate the vtable pointer
1809:     // of the resulting object.
1810:     llvm::BasicBlock *NonNullBlock = CGF.Builder.GetInsertBlock();
1811:     llvm::BasicBlock *PostCastAuthSuccess =
1812:         CGF.createBasicBlock("dynamic_cast.postauth.success");
1813:     llvm::BasicBlock *PostCastAuthComplete =
1814:         CGF.createBasicBlock("dynamic_cast.postauth.complete");
1815:     CGF.Builder.CreateCondBr(Success, PostCastAuthSuccess,
1816:                              PostCastAuthComplete);
1817:     CGF.EmitBlock(PostCastAuthSuccess);
1818:     Address AdjustedThisAddr =
1819:         Address(AdjustedThisPtr, CGF.IntPtrTy, CGF.getPointerAlign());
1820:     AuthenticateVTable(AdjustedThisAddr, DestDecl);
1821:     CGF.EmitBranch(PostCastAuthComplete);
1822:     CGF.EmitBlock(PostCastAuthComplete);
1823:     llvm::PHINode *PHI = CGF.Builder.CreatePHI(AdjustedThisPtr->getType(), 2);
1824:     PHI->addIncoming(AdjustedThisPtr, PostCastAuthSuccess);
1825:     llvm::Value *NullValue =
1826:         llvm::Constant::getNullValue(AdjustedThisPtr->getType());
1827:     PHI->addIncoming(NullValue, NonNullBlock);
1828:     AdjustedThisPtr = PHI;
1829:   }
1830:   CGF.Builder.CreateCondBr(Success, CastSuccess, CastFail);
```
- **EN**: This block defines callable entry points like `get`, `Address`, `AuthenticateVTable`, `getNullValue`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `Address`, `AuthenticateVTable`, `getNullValue`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1831-1860
```cpp
1831:   return AdjustedThisPtr;
1832: }
1833: 
1834: llvm::Value *ItaniumCXXABI::emitDynamicCastToVoid(CodeGenFunction &CGF,
1835:                                                   Address ThisAddr,
1836:                                                   QualType SrcRecordTy) {
1837:   auto *ClassDecl = SrcRecordTy->castAsCXXRecordDecl();
1838:   llvm::Value *OffsetToTop;
1839:   if (CGM.getLangOpts().RelativeCXXABIVTables) {
1840:     // Get the vtable pointer.
1841:     llvm::Value *VTable =
1842:         CGF.GetVTablePtr(ThisAddr, CGF.DefaultPtrTy, ClassDecl);
1843: 
1844:     // Get the offset-to-top from the vtable.
1845:     OffsetToTop =
1846:         CGF.Builder.CreateConstInBoundsGEP1_32(CGM.Int32Ty, VTable, -2U);
1847:     OffsetToTop = CGF.Builder.CreateAlignedLoad(
1848:         CGM.Int32Ty, OffsetToTop, CharUnits::fromQuantity(4), "offset.to.top");
1849:   } else {
1850:     llvm::Type *PtrDiffLTy =
1851:         CGF.ConvertType(CGF.getContext().getPointerDiffType());
1852: 
1853:     // Get the vtable pointer.
1854:     llvm::Value *VTable =
1855:         CGF.GetVTablePtr(ThisAddr, CGF.DefaultPtrTy, ClassDecl);
1856: 
1857:     // Get the offset-to-top from the vtable.
1858:     OffsetToTop =
1859:         CGF.Builder.CreateConstInBoundsGEP1_64(PtrDiffLTy, VTable, -2ULL);
1860:     OffsetToTop = CGF.Builder.CreateAlignedLoad(
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 1861-1890
```cpp
1861:         PtrDiffLTy, OffsetToTop, CGF.getPointerAlign(), "offset.to.top");
1862:   }
1863:   // Finally, add the offset to the pointer.
1864:   return CGF.Builder.CreateInBoundsGEP(CGF.Int8Ty, ThisAddr.emitRawPointer(CGF),
1865:                                        OffsetToTop);
1866: }
1867: 
1868: bool ItaniumCXXABI::EmitBadCastCall(CodeGenFunction &CGF) {
1869:   llvm::FunctionCallee Fn = getBadCastFn(CGF);
1870:   llvm::CallBase *Call = CGF.EmitRuntimeCallOrInvoke(Fn);
1871:   Call->setDoesNotReturn();
1872:   CGF.Builder.CreateUnreachable();
1873:   return true;
1874: }
1875: 
1876: llvm::Value *
1877: ItaniumCXXABI::GetVirtualBaseClassOffset(CodeGenFunction &CGF,
1878:                                          Address This,
1879:                                          const CXXRecordDecl *ClassDecl,
1880:                                          const CXXRecordDecl *BaseClassDecl) {
1881:   llvm::Value *VTablePtr = CGF.GetVTablePtr(This, CGM.Int8PtrTy, ClassDecl);
1882:   CharUnits VBaseOffsetOffset =
1883:       CGM.getItaniumVTableContext().getVirtualBaseOffsetOffset(ClassDecl,
1884:                                                                BaseClassDecl);
1885:   llvm::Value *VBaseOffsetPtr =
1886:     CGF.Builder.CreateConstGEP1_64(
1887:         CGF.Int8Ty, VTablePtr, VBaseOffsetOffset.getQuantity(),
1888:         "vbase.offset.ptr");
1889: 
1890:   llvm::Value *VBaseOffset;
```
- **EN**: This block defines callable entry points like `EmitBadCastCall`, `GetVirtualBaseClassOffset`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `EmitBadCastCall`, `GetVirtualBaseClassOffset`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 1891-1920
```cpp
1891:   if (CGM.getLangOpts().RelativeCXXABIVTables) {
1892:     VBaseOffset = CGF.Builder.CreateAlignedLoad(
1893:         CGF.Int32Ty, VBaseOffsetPtr, CharUnits::fromQuantity(4),
1894:         "vbase.offset");
1895:   } else {
1896:     VBaseOffset = CGF.Builder.CreateAlignedLoad(
1897:         CGM.PtrDiffTy, VBaseOffsetPtr, CGF.getPointerAlign(), "vbase.offset");
1898:   }
1899:   return VBaseOffset;
1900: }
1901: 
1902: void ItaniumCXXABI::EmitCXXConstructors(const CXXConstructorDecl *D) {
1903:   // Just make sure we're in sync with TargetCXXABI.
1904:   assert(CGM.getTarget().getCXXABI().hasConstructorVariants());
1905: 
1906:   // The constructor used for constructing this as a base class;
1907:   // ignores virtual bases.
1908:   CGM.EmitGlobal(GlobalDecl(D, Ctor_Base));
1909: 
1910:   // The constructor used for constructing this as a complete class;
1911:   // constructs the virtual bases, then calls the base constructor.
1912:   if (!D->getParent()->isAbstract()) {
1913:     // We don't need to emit the complete ctor if the class is abstract.
1914:     CGM.EmitGlobal(GlobalDecl(D, Ctor_Complete));
1915:   }
1916: }
1917: 
1918: CGCXXABI::AddedStructorArgCounts
1919: ItaniumCXXABI::buildStructorSignature(GlobalDecl GD,
1920:                                       SmallVectorImpl<CanQualType> &ArgTys) {
```
- **EN**: This block defines callable entry points like `EmitCXXConstructors`, `buildStructorSignature`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `EmitCXXConstructors`, `buildStructorSignature`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1921-1950
```cpp
1921:   ASTContext &Context = getContext();
1922: 
1923:   // All parameters are already in place except VTT, which goes after 'this'.
1924:   // These are Clang types, so we don't need to worry about sret yet.
1925: 
1926:   // Check if we need to add a VTT parameter (which has type global void **).
1927:   if ((isa<CXXConstructorDecl>(GD.getDecl()) ? GD.getCtorType() == Ctor_Base
1928:                                              : GD.getDtorType() == Dtor_Base) &&
1929:       cast<CXXMethodDecl>(GD.getDecl())->getParent()->getNumVBases() != 0) {
1930:     LangAS AS = CGM.GetGlobalVarAddressSpace(nullptr);
1931:     QualType Q = Context.getAddrSpaceQualType(Context.VoidPtrTy, AS);
1932:     ArgTys.insert(ArgTys.begin() + 1,
1933:                   Context.getPointerType(CanQualType::CreateUnsafe(Q)));
1934:     return AddedStructorArgCounts::prefix(1);
1935:   }
1936:   return AddedStructorArgCounts{};
1937: }
1938: 
1939: void ItaniumCXXABI::EmitCXXDestructors(const CXXDestructorDecl *D) {
1940:   // The destructor used for destructing this as a base class; ignores
1941:   // virtual bases.
1942:   CGM.EmitGlobal(GlobalDecl(D, Dtor_Base));
1943: 
1944:   // The destructor used for destructing this as a most-derived class;
1945:   // call the base destructor and then destructs any virtual bases.
1946:   CGM.EmitGlobal(GlobalDecl(D, Dtor_Complete));
1947: 
1948:   // The destructor in a virtual table is always a 'deleting'
1949:   // destructor, which calls the complete destructor and then uses the
1950:   // appropriate operator delete.
```
- **EN**: This block defines callable entry points like `prefix`, `EmitCXXDestructors`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `prefix`, `EmitCXXDestructors`；通过控制流（if）细化 ABI 降级 行为。

### Lines 1951-1980
```cpp
1951:   if (D->isVirtual())
1952:     CGM.EmitGlobal(GlobalDecl(D, Dtor_Deleting));
1953: }
1954: 
1955: void ItaniumCXXABI::addImplicitStructorParams(CodeGenFunction &CGF,
1956:                                               QualType &ResTy,
1957:                                               FunctionArgList &Params) {
1958:   const CXXMethodDecl *MD = cast<CXXMethodDecl>(CGF.CurGD.getDecl());
1959:   assert(isa<CXXConstructorDecl>(MD) || isa<CXXDestructorDecl>(MD));
1960: 
1961:   // Check if we need a VTT parameter as well.
1962:   if (NeedsVTTParameter(CGF.CurGD)) {
1963:     ASTContext &Context = getContext();
1964: 
1965:     // FIXME: avoid the fake decl
1966:     LangAS AS = CGM.GetGlobalVarAddressSpace(nullptr);
1967:     QualType Q = Context.getAddrSpaceQualType(Context.VoidPtrTy, AS);
1968:     QualType T = Context.getPointerType(Q);
1969:     auto *VTTDecl = ImplicitParamDecl::Create(
1970:         Context, /*DC=*/nullptr, MD->getLocation(), &Context.Idents.get("vtt"),
1971:         T, ImplicitParamKind::CXXVTT);
1972:     Params.insert(Params.begin() + 1, VTTDecl);
1973:     getStructorImplicitParamDecl(CGF) = VTTDecl;
1974:   }
1975: }
1976: 
1977: void ItaniumCXXABI::EmitInstanceFunctionProlog(CodeGenFunction &CGF) {
1978:   // Naked functions have no prolog.
1979:   if (CGF.CurFuncDecl && CGF.CurFuncDecl->hasAttr<NakedAttr>())
1980:     return;
```
- **EN**: This block defines callable entry points like `addImplicitStructorParams`, `EmitInstanceFunctionProlog`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `addImplicitStructorParams`, `EmitInstanceFunctionProlog`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1981-2010
```cpp
1981: 
1982:   /// Initialize the 'this' slot. In the Itanium C++ ABI, no prologue
1983:   /// adjustments are required, because they are all handled by thunks.
1984:   setCXXABIThisValue(CGF, loadIncomingCXXThis(CGF));
1985: 
1986:   /// Initialize the 'vtt' slot if needed.
1987:   if (getStructorImplicitParamDecl(CGF)) {
1988:     getStructorImplicitParamValue(CGF) = CGF.Builder.CreateLoad(
1989:         CGF.GetAddrOfLocalVar(getStructorImplicitParamDecl(CGF)), "vtt");
1990:   }
1991: 
1992:   /// If this is a function that the ABI specifies returns 'this', initialize
1993:   /// the return slot to 'this' at the start of the function.
1994:   ///
1995:   /// Unlike the setting of return types, this is done within the ABI
1996:   /// implementation instead of by clients of CGCXXABI because:
1997:   /// 1) getThisValue is currently protected
1998:   /// 2) in theory, an ABI could implement 'this' returns some other way;
1999:   ///    HasThisReturn only specifies a contract, not the implementation
2000:   if (HasThisReturn(CGF.CurGD))
2001:     CGF.Builder.CreateStore(getThisValue(CGF), CGF.ReturnValue);
2002: }
2003: 
2004: CGCXXABI::AddedStructorArgs ItaniumCXXABI::getImplicitConstructorArgs(
2005:     CodeGenFunction &CGF, const CXXConstructorDecl *D, CXXCtorType Type,
2006:     bool ForVirtualBase, bool Delegating) {
2007:   if (!NeedsVTTParameter(GlobalDecl(D, Type)))
2008:     return AddedStructorArgs{};
2009: 
2010:   // Insert the implicit 'vtt' argument as the second argument. Make sure to
```
- **EN**: This block defines callable entry points like `setCXXABIThisValue`, `getStructorImplicitParamValue`, `getImplicitConstructorArgs`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `setCXXABIThisValue`, `getStructorImplicitParamValue`, `getImplicitConstructorArgs`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2011-2040
```cpp
2011:   // correctly reflect its address space, which can differ from generic on
2012:   // some targets.
2013:   llvm::Value *VTT =
2014:       CGF.GetVTTParameter(GlobalDecl(D, Type), ForVirtualBase, Delegating);
2015:   LangAS AS = CGM.GetGlobalVarAddressSpace(nullptr);
2016:   QualType Q = getContext().getAddrSpaceQualType(getContext().VoidPtrTy, AS);
2017:   QualType VTTTy = getContext().getPointerType(Q);
2018:   return AddedStructorArgs::prefix({{VTT, VTTTy}});
2019: }
2020: 
2021: llvm::Value *ItaniumCXXABI::getCXXDestructorImplicitParam(
2022:     CodeGenFunction &CGF, const CXXDestructorDecl *DD, CXXDtorType Type,
2023:     bool ForVirtualBase, bool Delegating) {
2024:   GlobalDecl GD(DD, Type);
2025:   return CGF.GetVTTParameter(GD, ForVirtualBase, Delegating);
2026: }
2027: 
2028: void ItaniumCXXABI::EmitDestructorCall(CodeGenFunction &CGF,
2029:                                        const CXXDestructorDecl *DD,
2030:                                        CXXDtorType Type, bool ForVirtualBase,
2031:                                        bool Delegating, Address This,
2032:                                        QualType ThisTy) {
2033:   GlobalDecl GD(DD, Type);
2034:   llvm::Value *VTT =
2035:       getCXXDestructorImplicitParam(CGF, DD, Type, ForVirtualBase, Delegating);
2036:   QualType VTTTy = getContext().getPointerType(getContext().VoidPtrTy);
2037: 
2038:   CGCallee Callee;
2039:   if (getContext().getLangOpts().AppleKext &&
2040:       Type != Dtor_Base && DD->isVirtual())
```
- **EN**: This block defines callable entry points like `GD`, `EmitDestructorCall`, `getCXXDestructorImplicitParam`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `GD`, `EmitDestructorCall`, `getCXXDestructorImplicitParam`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2041-2070
```cpp
2041:     Callee = CGF.BuildAppleKextVirtualDestructorCall(DD, Type, DD->getParent());
2042:   else
2043:     Callee = CGCallee::forDirect(CGM.getAddrOfCXXStructor(GD), GD);
2044: 
2045:   CGF.EmitCXXDestructorCall(GD, Callee, CGF.getAsNaturalPointerTo(This, ThisTy),
2046:                             ThisTy, VTT, VTTTy, nullptr);
2047: }
2048: 
2049: // Check if any non-inline method has the specified attribute.
2050: template <typename T>
2051: static bool CXXRecordNonInlineHasAttr(const CXXRecordDecl *RD) {
2052:   for (const auto *D : RD->noload_decls()) {
2053:     if (const auto *FD = dyn_cast<FunctionDecl>(D)) {
2054:       if (FD->isInlined() || FD->doesThisDeclarationHaveABody() ||
2055:           FD->isPureVirtual())
2056:         continue;
2057:       if (D->hasAttr<T>())
2058:         return true;
2059:     }
2060:   }
2061: 
2062:   return false;
2063: }
2064: 
2065: static void setVTableSelectiveDLLImportExport(CodeGenModule &CGM,
2066:                                               llvm::GlobalVariable *VTable,
2067:                                               const CXXRecordDecl *RD) {
2068:   if (VTable->getDLLStorageClass() !=
2069:           llvm::GlobalVariable::DefaultStorageClass ||
2070:       RD->hasAttr<DLLImportAttr>() || RD->hasAttr<DLLExportAttr>())
```
- **EN**: This block defines callable entry points like `CXXRecordNonInlineHasAttr`, `setVTableSelectiveDLLImportExport`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `CXXRecordNonInlineHasAttr`, `setVTableSelectiveDLLImportExport`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 2071-2100
```cpp
2071:     return;
2072: 
2073:   if (CGM.getVTables().isVTableExternal(RD)) {
2074:     if (CXXRecordNonInlineHasAttr<DLLImportAttr>(RD))
2075:       VTable->setDLLStorageClass(llvm::GlobalValue::DLLImportStorageClass);
2076:   } else if (CXXRecordNonInlineHasAttr<DLLExportAttr>(RD))
2077:     VTable->setDLLStorageClass(llvm::GlobalValue::DLLExportStorageClass);
2078: }
2079: 
2080: void ItaniumCXXABI::emitVTableDefinitions(CodeGenVTables &CGVT,
2081:                                           const CXXRecordDecl *RD) {
2082:   llvm::GlobalVariable *VTable = getAddrOfVTable(RD, CharUnits());
2083:   if (VTable->hasInitializer())
2084:     return;
2085: 
2086:   ItaniumVTableContext &VTContext = CGM.getItaniumVTableContext();
2087:   const VTableLayout &VTLayout = VTContext.getVTableLayout(RD);
2088:   llvm::GlobalVariable::LinkageTypes Linkage = CGM.getVTableLinkage(RD);
2089:   llvm::Constant *RTTI =
2090:       CGM.GetAddrOfRTTIDescriptor(CGM.getContext().getCanonicalTagType(RD));
2091: 
2092:   // Create and set the initializer.
2093:   ConstantInitBuilder builder(CGM);
2094:   auto components = builder.beginStruct();
2095:   CGVT.createVTableInitializer(components, VTLayout, RTTI,
2096:                                llvm::GlobalValue::isLocalLinkage(Linkage));
2097:   components.finishAndSetAsInitializer(VTable);
2098: 
2099:   // Set the correct linkage.
2100:   VTable->setLinkage(Linkage);
```
- **EN**: This block defines callable entry points like `emitVTableDefinitions`, `builder`, `isLocalLinkage`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitVTableDefinitions`, `builder`, `isLocalLinkage`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2101-2130
```cpp
2101: 
2102:   if (CGM.supportsCOMDAT() && VTable->isWeakForLinker())
2103:     VTable->setComdat(CGM.getModule().getOrInsertComdat(VTable->getName()));
2104: 
2105:   if (CGM.getTarget().hasPS4DLLImportExport())
2106:     setVTableSelectiveDLLImportExport(CGM, VTable, RD);
2107: 
2108:   // Set the right visibility.
2109:   CGM.setGVProperties(VTable, RD);
2110: 
2111:   // If this is the magic class __cxxabiv1::__fundamental_type_info,
2112:   // we will emit the typeinfo for the fundamental types. This is the
2113:   // same behaviour as GCC.
2114:   const DeclContext *DC = RD->getDeclContext();
2115:   if (RD->getIdentifier() &&
2116:       RD->getIdentifier()->isStr("__fundamental_type_info") &&
2117:       isa<NamespaceDecl>(DC) && cast<NamespaceDecl>(DC)->getIdentifier() &&
2118:       cast<NamespaceDecl>(DC)->getIdentifier()->isStr("__cxxabiv1") &&
2119:       DC->getParent()->isTranslationUnit())
2120:     EmitFundamentalRTTIDescriptors(RD);
2121: 
2122:   // Always emit type metadata on non-available_externally definitions, and on
2123:   // available_externally definitions if we are performing whole program
2124:   // devirtualization or speculative devirtualization. We need the type metadata
2125:   // on all vtable definitions to ensure we associate derived classes with base
2126:   // classes defined in headers but with a strong definition only in a shared
2127:   // library.
2128:   if (!VTable->isDeclarationForLinker() ||
2129:       CGM.getCodeGenOpts().WholeProgramVTables ||
2130:       CGM.getCodeGenOpts().DevirtualizeSpeculatively) {
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 2131-2160
```cpp
2131:     CGM.EmitVTableTypeMetadata(RD, VTable, VTLayout);
2132:     // For available_externally definitions, add the vtable to
2133:     // @llvm.compiler.used so that it isn't deleted before whole program
2134:     // analysis.
2135:     if (VTable->isDeclarationForLinker()) {
2136:       assert(CGM.getCodeGenOpts().WholeProgramVTables ||
2137:              CGM.getCodeGenOpts().DevirtualizeSpeculatively);
2138:       CGM.addCompilerUsedGlobal(VTable);
2139:     }
2140:   }
2141: 
2142:   if (CGM.getLangOpts().RelativeCXXABIVTables) {
2143:     CGVT.RemoveHwasanMetadata(VTable);
2144:     if (!VTable->isDSOLocal())
2145:       CGVT.GenerateRelativeVTableAlias(VTable, VTable->getName());
2146:   }
2147: 
2148:   // Emit symbol for debugger only if requested debug info.
2149:   if (CGDebugInfo *DI = CGM.getModuleDebugInfo())
2150:     DI->emitVTableSymbol(VTable, RD);
2151: }
2152: 
2153: bool ItaniumCXXABI::isVirtualOffsetNeededForVTableField(
2154:     CodeGenFunction &CGF, CodeGenFunction::VPtr Vptr) {
2155:   if (Vptr.NearestVBase == nullptr)
2156:     return false;
2157:   return NeedsVTTParameter(CGF.CurGD);
2158: }
2159: 
2160: llvm::Value *ItaniumCXXABI::getVTableAddressPointInStructor(
```
- **EN**: This block defines callable entry points like `isVirtualOffsetNeededForVTableField`, `NeedsVTTParameter`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `isVirtualOffsetNeededForVTableField`, `NeedsVTTParameter`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2161-2190
```cpp
2161:     CodeGenFunction &CGF, const CXXRecordDecl *VTableClass, BaseSubobject Base,
2162:     const CXXRecordDecl *NearestVBase) {
2163: 
2164:   if ((Base.getBase()->getNumVBases() || NearestVBase != nullptr) &&
2165:       NeedsVTTParameter(CGF.CurGD)) {
2166:     return getVTableAddressPointInStructorWithVTT(CGF, VTableClass, Base,
2167:                                                   NearestVBase);
2168:   }
2169:   return getVTableAddressPoint(Base, VTableClass);
2170: }
2171: 
2172: llvm::Constant *
2173: ItaniumCXXABI::getVTableAddressPoint(BaseSubobject Base,
2174:                                      const CXXRecordDecl *VTableClass) {
2175:   llvm::GlobalValue *VTable = getAddrOfVTable(VTableClass, CharUnits());
2176: 
2177:   // Find the appropriate vtable within the vtable group, and the address point
2178:   // within that vtable.
2179:   const VTableLayout &Layout =
2180:       CGM.getItaniumVTableContext().getVTableLayout(VTableClass);
2181:   VTableLayout::AddressPointLocation AddressPoint =
2182:       Layout.getAddressPoint(Base);
2183:   llvm::Value *Indices[] = {
2184:     llvm::ConstantInt::get(CGM.Int32Ty, 0),
2185:     llvm::ConstantInt::get(CGM.Int32Ty, AddressPoint.VTableIndex),
2186:     llvm::ConstantInt::get(CGM.Int32Ty, AddressPoint.AddressPointIndex),
2187:   };
2188: 
2189:   // Add inrange attribute to indicate that only the VTableIndex can be
2190:   // accessed.
```
- **EN**: This block defines callable entry points like `getVTableAddressPointInStructorWithVTT`, `getVTableAddressPoint`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getVTableAddressPointInStructorWithVTT`, `getVTableAddressPoint`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2191-2220
```cpp
2191:   unsigned ComponentSize =
2192:       CGM.getDataLayout().getTypeAllocSize(CGM.getVTableComponentType());
2193:   unsigned VTableSize =
2194:       ComponentSize * Layout.getVTableSize(AddressPoint.VTableIndex);
2195:   unsigned Offset = ComponentSize * AddressPoint.AddressPointIndex;
2196:   llvm::ConstantRange InRange(
2197:       llvm::APInt(32, (int)-Offset, true),
2198:       llvm::APInt(32, (int)(VTableSize - Offset), true));
2199:   return llvm::ConstantExpr::getGetElementPtr(
2200:       VTable->getValueType(), VTable, Indices, /*InBounds=*/true, InRange);
2201: }
2202: 
2203: llvm::Value *ItaniumCXXABI::getVTableAddressPointInStructorWithVTT(
2204:     CodeGenFunction &CGF, const CXXRecordDecl *VTableClass, BaseSubobject Base,
2205:     const CXXRecordDecl *NearestVBase) {
2206:   assert((Base.getBase()->getNumVBases() || NearestVBase != nullptr) &&
2207:          NeedsVTTParameter(CGF.CurGD) && "This class doesn't have VTT");
2208: 
2209:   // Get the secondary vpointer index.
2210:   uint64_t VirtualPointerIndex =
2211:       CGM.getVTables().getSecondaryVirtualPointerIndex(VTableClass, Base);
2212: 
2213:   /// Load the VTT.
2214:   llvm::Value *VTT = CGF.LoadCXXVTT();
2215:   if (VirtualPointerIndex)
2216:     VTT = CGF.Builder.CreateConstInBoundsGEP1_64(CGF.GlobalsVoidPtrTy, VTT,
2217:                                                  VirtualPointerIndex);
2218: 
2219:   // And load the address point from the VTT.
2220:   llvm::Value *AP =
```
- **EN**: This block introduces declarations such as `doesn`; defines callable entry points like `InRange`, `getGetElementPtr`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `doesn` 的声明；定义可调用入口，例如 `InRange`, `getGetElementPtr`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2221-2250
```cpp
2221:       CGF.Builder.CreateAlignedLoad(CGF.GlobalsVoidPtrTy, VTT,
2222:                                     CGF.getPointerAlign());
2223: 
2224:   if (auto &Schema = CGF.CGM.getCodeGenOpts().PointerAuth.CXXVTTVTablePointers) {
2225:     CGPointerAuthInfo PointerAuth = CGF.EmitPointerAuthInfo(Schema, VTT,
2226:                                                             GlobalDecl(),
2227:                                                             QualType());
2228:     AP = CGF.EmitPointerAuthAuth(PointerAuth, AP);
2229:   }
2230: 
2231:   return AP;
2232: }
2233: 
2234: llvm::GlobalVariable *ItaniumCXXABI::getAddrOfVTable(const CXXRecordDecl *RD,
2235:                                                      CharUnits VPtrOffset) {
2236:   assert(VPtrOffset.isZero() && "Itanium ABI only supports zero vptr offsets");
2237: 
2238:   llvm::GlobalVariable *&VTable = VTables[RD];
2239:   if (VTable)
2240:     return VTable;
2241: 
2242:   // Queue up this vtable for possible deferred emission.
2243:   CGM.addDeferredVTable(RD);
2244: 
2245:   SmallString<256> Name;
2246:   llvm::raw_svector_ostream Out(Name);
2247:   getMangleContext().mangleCXXVTable(RD, Out);
2248: 
2249:   const VTableLayout &VTLayout =
2250:       CGM.getItaniumVTableContext().getVTableLayout(RD);
```
- **EN**: This block defines callable entry points like `GlobalDecl`, `Out`, `getMangleContext`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `GlobalDecl`, `Out`, `getMangleContext`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2251-2280
```cpp
2251:   llvm::Type *VTableType = CGM.getVTables().getVTableType(VTLayout);
2252: 
2253:   // Use pointer to global alignment for the vtable. Otherwise we would align
2254:   // them based on the size of the initializer which doesn't make sense as only
2255:   // single values are read.
2256:   unsigned PAlign = CGM.getVtableGlobalVarAlignment();
2257: 
2258:   VTable = CGM.CreateOrReplaceCXXRuntimeVariable(
2259:       Name, VTableType, llvm::GlobalValue::ExternalLinkage,
2260:       getContext().toCharUnitsFromBits(PAlign).getAsAlign());
2261:   VTable->setUnnamedAddr(llvm::GlobalValue::UnnamedAddr::Global);
2262: 
2263:   if (CGM.getTarget().hasPS4DLLImportExport())
2264:     setVTableSelectiveDLLImportExport(CGM, VTable, RD);
2265: 
2266:   CGM.setGVProperties(VTable, RD);
2267:   return VTable;
2268: }
2269: 
2270: CGCallee ItaniumCXXABI::getVirtualFunctionPointer(CodeGenFunction &CGF,
2271:                                                   GlobalDecl GD,
2272:                                                   Address This,
2273:                                                   llvm::Type *Ty,
2274:                                                   SourceLocation Loc) {
2275:   llvm::Type *PtrTy = CGM.GlobalsInt8PtrTy;
2276:   auto *MethodDecl = cast<CXXMethodDecl>(GD.getDecl());
2277:   llvm::Value *VTable = CGF.GetVTablePtr(This, PtrTy, MethodDecl->getParent());
2278: 
2279:   // For the translation of virtual functions, we need to map the (potential)
2280:   // host vtable to the device vtable. This is done by calling the runtime
```
- **EN**: This block defines callable entry points like `getContext`, `getVirtualFunctionPointer`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `getVirtualFunctionPointer`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2281-2310
```cpp
2281:   // function
2282:   // __llvm_omp_indirect_call_lookup.
2283:   if (CGM.getLangOpts().OpenMPIsTargetDevice) {
2284:     auto *NewPtrTy = CGM.VoidPtrTy;
2285:     llvm::Type *RtlFnArgs[] = {NewPtrTy};
2286:     llvm::FunctionCallee DeviceRtlFn = CGM.CreateRuntimeFunction(
2287:         llvm::FunctionType::get(NewPtrTy, RtlFnArgs, false),
2288:         "__llvm_omp_indirect_call_lookup");
2289:     auto *BackupTy = VTable->getType();
2290:     // Need to convert to generic address space
2291:     VTable = CGF.Builder.CreatePointerBitCastOrAddrSpaceCast(VTable, NewPtrTy);
2292:     VTable = CGF.EmitRuntimeCall(DeviceRtlFn, {VTable});
2293:     // convert to original address space
2294:     VTable = CGF.Builder.CreatePointerBitCastOrAddrSpaceCast(VTable, BackupTy);
2295:   }
2296: 
2297:   uint64_t VTableIndex = CGM.getItaniumVTableContext().getMethodVTableIndex(GD);
2298:   llvm::Value *VFunc, *VTableSlotPtr = nullptr;
2299:   auto &Schema = CGM.getCodeGenOpts().PointerAuth.CXXVirtualFunctionPointers;
2300: 
2301:   llvm::Type *ComponentTy = CGM.getVTables().getVTableComponentType();
2302:   uint64_t ByteOffset =
2303:       VTableIndex * CGM.getDataLayout().getTypeSizeInBits(ComponentTy) / 8;
2304: 
2305:   if (!Schema && CGF.ShouldEmitVTableTypeCheckedLoad(MethodDecl->getParent())) {
2306:     VFunc = CGF.EmitVTableTypeCheckedLoad(MethodDecl->getParent(), VTable,
2307:                                           PtrTy, ByteOffset);
2308:   } else {
2309:     CGF.EmitTypeMetadataCodeForVCall(MethodDecl->getParent(), VTable, Loc);
2310: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2311-2340
```cpp
2311:     llvm::Value *VFuncLoad;
2312:     if (CGM.getLangOpts().RelativeCXXABIVTables) {
2313:       VFuncLoad = CGF.Builder.CreateCall(
2314:           CGM.getIntrinsic(llvm::Intrinsic::load_relative, {CGM.Int32Ty}),
2315:           {VTable, llvm::ConstantInt::get(CGM.Int32Ty, ByteOffset)});
2316:     } else {
2317:       VTableSlotPtr = CGF.Builder.CreateConstInBoundsGEP1_64(
2318:           PtrTy, VTable, VTableIndex, "vfn");
2319:       VFuncLoad = CGF.Builder.CreateAlignedLoad(PtrTy, VTableSlotPtr,
2320:                                                 CGF.getPointerAlign());
2321:     }
2322: 
2323:     // Add !invariant.load md to virtual function load to indicate that
2324:     // function didn't change inside vtable.
2325:     // It's safe to add it without -fstrict-vtable-pointers, but it would not
2326:     // help in devirtualization because it will only matter if we will have 2
2327:     // the same virtual function loads from the same vtable load, which won't
2328:     // happen without enabled devirtualization with -fstrict-vtable-pointers.
2329:     if (CGM.getCodeGenOpts().OptimizationLevel > 0 &&
2330:         CGM.getCodeGenOpts().StrictVTablePointers) {
2331:       if (auto *VFuncLoadInstr = dyn_cast<llvm::Instruction>(VFuncLoad)) {
2332:         VFuncLoadInstr->setMetadata(
2333:             llvm::LLVMContext::MD_invariant_load,
2334:             llvm::MDNode::get(CGM.getLLVMContext(),
2335:                               llvm::ArrayRef<llvm::Metadata *>()));
2336:       }
2337:     }
2338:     VFunc = VFuncLoad;
2339:   }
2340: 
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2341-2370
```cpp
2341:   CGPointerAuthInfo PointerAuth;
2342:   if (Schema) {
2343:     assert(VTableSlotPtr && "virtual function pointer not set");
2344:     GD = CGM.getItaniumVTableContext().findOriginalMethod(GD.getCanonicalDecl());
2345:     PointerAuth = CGF.EmitPointerAuthInfo(Schema, VTableSlotPtr, GD, QualType());
2346:   }
2347:   CGCallee Callee(GD, VFunc, PointerAuth);
2348:   return Callee;
2349: }
2350: 
2351: llvm::Value *ItaniumCXXABI::EmitVirtualDestructorCall(
2352:     CodeGenFunction &CGF, const CXXDestructorDecl *Dtor, CXXDtorType DtorType,
2353:     Address This, DeleteOrMemberCallExpr E, llvm::CallBase **CallOrInvoke) {
2354:   auto *CE = dyn_cast<const CXXMemberCallExpr *>(E);
2355:   auto *D = dyn_cast<const CXXDeleteExpr *>(E);
2356:   assert((CE != nullptr) ^ (D != nullptr));
2357:   assert(CE == nullptr || CE->arguments().empty());
2358:   assert(DtorType == Dtor_Deleting || DtorType == Dtor_Complete);
2359: 
2360:   GlobalDecl GD(Dtor, DtorType);
2361:   const CGFunctionInfo *FInfo =
2362:       &CGM.getTypes().arrangeCXXStructorDeclaration(GD);
2363:   llvm::FunctionType *Ty = CGF.CGM.getTypes().GetFunctionType(*FInfo);
2364:   CGCallee Callee = CGCallee::forVirtual(CE, GD, This, Ty);
2365: 
2366:   QualType ThisTy;
2367:   if (CE) {
2368:     ThisTy = CE->getObjectType();
2369:   } else {
2370:     ThisTy = D->getDestroyedType();
```
- **EN**: This block defines callable entry points like `Callee`, `GD`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Callee`, `GD`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2371-2400
```cpp
2371:   }
2372: 
2373:   CGF.EmitCXXDestructorCall(GD, Callee, This.emitRawPointer(CGF), ThisTy,
2374:                             nullptr, QualType(), nullptr, CallOrInvoke);
2375:   return nullptr;
2376: }
2377: 
2378: void ItaniumCXXABI::emitVirtualInheritanceTables(const CXXRecordDecl *RD) {
2379:   CodeGenVTables &VTables = CGM.getVTables();
2380:   llvm::GlobalVariable *VTT = VTables.GetAddrOfVTT(RD);
2381:   VTables.EmitVTTDefinition(VTT, CGM.getVTableLinkage(RD), RD);
2382: }
2383: 
2384: bool ItaniumCXXABI::canSpeculativelyEmitVTableAsBaseClass(
2385:     const CXXRecordDecl *RD) const {
2386:   // We don't emit available_externally vtables if we are in -fapple-kext mode
2387:   // because kext mode does not permit devirtualization.
2388:   if (CGM.getLangOpts().AppleKext)
2389:     return false;
2390: 
2391:   // If the vtable is hidden then it is not safe to emit an available_externally
2392:   // copy of vtable.
2393:   if (isVTableHidden(RD))
2394:     return false;
2395: 
2396:   if (CGM.getCodeGenOpts().ForceEmitVTables)
2397:     return true;
2398: 
2399:   // A speculative vtable can only be generated if all virtual inline functions
2400:   // defined by this class are emitted. The vtable in the final program contains
```
- **EN**: This block defines callable entry points like `QualType`, `emitVirtualInheritanceTables`, `canSpeculativelyEmitVTableAsBaseClass`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `QualType`, `emitVirtualInheritanceTables`, `canSpeculativelyEmitVTableAsBaseClass`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2401-2430
```cpp
2401:   // for each virtual inline function not used in the current TU a function that
2402:   // is equivalent to the unused function. The function in the actual vtable
2403:   // does not have to be declared under the same symbol (e.g., a virtual
2404:   // destructor that can be substituted with its base class's destructor). Since
2405:   // inline functions are emitted lazily and this emissions does not account for
2406:   // speculative emission of a vtable, we might generate a speculative vtable
2407:   // with references to inline functions that are not emitted under that name.
2408:   // This can lead to problems when devirtualizing a call to such a function,
2409:   // that result in linking errors. Hence, if there are any unused virtual
2410:   // inline function, we cannot emit the speculative vtable.
2411:   // FIXME we can still emit a copy of the vtable if we
2412:   // can emit definition of the inline functions.
2413:   if (hasAnyUnusedVirtualInlineFunction(RD))
2414:     return false;
2415: 
2416:   // For a class with virtual bases, we must also be able to speculatively
2417:   // emit the VTT, because CodeGen doesn't have separate notions of "can emit
2418:   // the vtable" and "can emit the VTT". For a base subobject, this means we
2419:   // need to be able to emit non-virtual base vtables.
2420:   if (RD->getNumVBases()) {
2421:     for (const auto &B : RD->bases()) {
2422:       auto *BRD = B.getType()->getAsCXXRecordDecl();
2423:       assert(BRD && "no class for base specifier");
2424:       if (B.isVirtual() || !BRD->isDynamicClass())
2425:         continue;
2426:       if (!canSpeculativelyEmitVTableAsBaseClass(BRD))
2427:         return false;
2428:     }
2429:   }
2430: 
```
- **EN**: This block introduces declarations such as `for`; uses control flow (if, for) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `for` 的声明；通过控制流（if, for）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2431-2460
```cpp
2431:   return true;
2432: }
2433: 
2434: bool ItaniumCXXABI::canSpeculativelyEmitVTable(const CXXRecordDecl *RD) const {
2435:   if (!canSpeculativelyEmitVTableAsBaseClass(RD))
2436:     return false;
2437: 
2438:   if (RD->shouldEmitInExternalSource())
2439:     return false;
2440: 
2441:   // For a complete-object vtable (or more specifically, for the VTT), we need
2442:   // to be able to speculatively emit the vtables of all dynamic virtual bases.
2443:   for (const auto &B : RD->vbases()) {
2444:     auto *BRD = B.getType()->getAsCXXRecordDecl();
2445:     assert(BRD && "no class for base specifier");
2446:     if (!BRD->isDynamicClass())
2447:       continue;
2448:     if (!canSpeculativelyEmitVTableAsBaseClass(BRD))
2449:       return false;
2450:   }
2451: 
2452:   return true;
2453: }
2454: static llvm::Value *performTypeAdjustment(CodeGenFunction &CGF,
2455:                                           Address InitialPtr,
2456:                                           const CXXRecordDecl *UnadjustedClass,
2457:                                           int64_t NonVirtualAdjustment,
2458:                                           int64_t VirtualAdjustment,
2459:                                           bool IsReturnAdjustment) {
2460:   if (!NonVirtualAdjustment && !VirtualAdjustment)
```
- **EN**: This block introduces declarations such as `for`; defines callable entry points like `canSpeculativelyEmitVTable`; uses control flow (if, for) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `for` 的声明；定义可调用入口，例如 `canSpeculativelyEmitVTable`；通过控制流（if, for）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2461-2490
```cpp
2461:     return InitialPtr.emitRawPointer(CGF);
2462: 
2463:   Address V = InitialPtr.withElementType(CGF.Int8Ty);
2464: 
2465:   // In a base-to-derived cast, the non-virtual adjustment is applied first.
2466:   if (NonVirtualAdjustment && !IsReturnAdjustment) {
2467:     V = CGF.Builder.CreateConstInBoundsByteGEP(V,
2468:                               CharUnits::fromQuantity(NonVirtualAdjustment));
2469:   }
2470: 
2471:   // Perform the virtual adjustment if we have one.
2472:   llvm::Value *ResultPtr;
2473:   if (VirtualAdjustment) {
2474:     llvm::Value *VTablePtr =
2475:         CGF.GetVTablePtr(V, CGF.Int8PtrTy, UnadjustedClass);
2476: 
2477:     llvm::Value *Offset;
2478:     llvm::Value *OffsetPtr = CGF.Builder.CreateConstInBoundsGEP1_64(
2479:         CGF.Int8Ty, VTablePtr, VirtualAdjustment);
2480:     if (CGF.CGM.getLangOpts().RelativeCXXABIVTables) {
2481:       // Load the adjustment offset from the vtable as a 32-bit int.
2482:       Offset =
2483:           CGF.Builder.CreateAlignedLoad(CGF.Int32Ty, OffsetPtr,
2484:                                         CharUnits::fromQuantity(4));
2485:     } else {
2486:       llvm::Type *PtrDiffTy =
2487:           CGF.ConvertType(CGF.getContext().getPointerDiffType());
2488: 
2489:       // Load the adjustment offset from the vtable.
2490:       Offset = CGF.Builder.CreateAlignedLoad(PtrDiffTy, OffsetPtr,
```
- **EN**: This block defines callable entry points like `fromQuantity`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `fromQuantity`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2491-2520
```cpp
2491:                                              CGF.getPointerAlign());
2492:     }
2493:     // Adjust our pointer.
2494:     ResultPtr = CGF.Builder.CreateInBoundsGEP(V.getElementType(),
2495:                                               V.emitRawPointer(CGF), Offset);
2496:   } else {
2497:     ResultPtr = V.emitRawPointer(CGF);
2498:   }
2499: 
2500:   // In a derived-to-base conversion, the non-virtual adjustment is
2501:   // applied second.
2502:   if (NonVirtualAdjustment && IsReturnAdjustment) {
2503:     ResultPtr = CGF.Builder.CreateConstInBoundsGEP1_64(CGF.Int8Ty, ResultPtr,
2504:                                                        NonVirtualAdjustment);
2505:   }
2506: 
2507:   return ResultPtr;
2508: }
2509: 
2510: llvm::Value *
2511: ItaniumCXXABI::performThisAdjustment(CodeGenFunction &CGF, Address This,
2512:                                      const CXXRecordDecl *UnadjustedClass,
2513:                                      const ThunkInfo &TI) {
2514:   return performTypeAdjustment(CGF, This, UnadjustedClass, TI.This.NonVirtual,
2515:                                TI.This.Virtual.Itanium.VCallOffsetOffset,
2516:                                /*IsReturnAdjustment=*/false);
2517: }
2518: 
2519: llvm::Value *
2520: ItaniumCXXABI::performReturnAdjustment(CodeGenFunction &CGF, Address Ret,
```
- **EN**: This block defines callable entry points like `performThisAdjustment`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `performThisAdjustment`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2521-2550
```cpp
2521:                                        const CXXRecordDecl *UnadjustedClass,
2522:                                        const ReturnAdjustment &RA) {
2523:   return performTypeAdjustment(CGF, Ret, UnadjustedClass, RA.NonVirtual,
2524:                                RA.Virtual.Itanium.VBaseOffsetOffset,
2525:                                /*IsReturnAdjustment=*/true);
2526: }
2527: 
2528: void ARMCXXABI::EmitReturnFromThunk(CodeGenFunction &CGF,
2529:                                     RValue RV, QualType ResultType) {
2530:   if (!isa<CXXDestructorDecl>(CGF.CurGD.getDecl()))
2531:     return ItaniumCXXABI::EmitReturnFromThunk(CGF, RV, ResultType);
2532: 
2533:   // Destructor thunks in the ARM ABI have indeterminate results.
2534:   llvm::Type *T = CGF.ReturnValue.getElementType();
2535:   RValue Undef = RValue::get(llvm::UndefValue::get(T));
2536:   return ItaniumCXXABI::EmitReturnFromThunk(CGF, Undef, ResultType);
2537: }
2538: 
2539: /************************** Array allocation cookies **************************/
2540: 
2541: CharUnits ItaniumCXXABI::getArrayCookieSizeImpl(QualType elementType) {
2542:   // The array cookie is a size_t; pad that up to the element alignment.
2543:   // The cookie is actually right-justified in that space.
2544:   return std::max(CharUnits::fromQuantity(CGM.SizeSizeInBytes),
2545:                   CGM.getContext().getPreferredTypeAlignInChars(elementType));
2546: }
2547: 
2548: Address ItaniumCXXABI::InitializeArrayCookie(CodeGenFunction &CGF,
2549:                                              Address NewPtr,
2550:                                              llvm::Value *NumElements,
```
- **EN**: This block defines callable entry points like `EmitReturnFromThunk`, `getArrayCookieSizeImpl`, `max`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitReturnFromThunk`, `getArrayCookieSizeImpl`, `max`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2551-2580
```cpp
2551:                                              const CXXNewExpr *expr,
2552:                                              QualType ElementType) {
2553:   assert(requiresArrayCookie(expr));
2554: 
2555:   unsigned AS = NewPtr.getAddressSpace();
2556: 
2557:   ASTContext &Ctx = getContext();
2558:   CharUnits SizeSize = CGF.getSizeSize();
2559: 
2560:   // The size of the cookie.
2561:   CharUnits CookieSize =
2562:       std::max(SizeSize, Ctx.getPreferredTypeAlignInChars(ElementType));
2563:   assert(CookieSize == getArrayCookieSizeImpl(ElementType));
2564: 
2565:   // Compute an offset to the cookie.
2566:   Address CookiePtr = NewPtr;
2567:   CharUnits CookieOffset = CookieSize - SizeSize;
2568:   if (!CookieOffset.isZero())
2569:     CookiePtr = CGF.Builder.CreateConstInBoundsByteGEP(CookiePtr, CookieOffset);
2570: 
2571:   // Write the number of elements into the appropriate slot.
2572:   Address NumElementsPtr = CookiePtr.withElementType(CGF.SizeTy);
2573:   llvm::Instruction *SI = CGF.Builder.CreateStore(NumElements, NumElementsPtr);
2574: 
2575:   // Handle the array cookie specially in ASan.
2576:   if (CGM.getLangOpts().Sanitize.has(SanitizerKind::Address) && AS == 0 &&
2577:       (expr->getOperatorNew()->isReplaceableGlobalAllocationFunction() ||
2578:        CGM.getCodeGenOpts().SanitizeAddressPoisonCustomArrayCookie)) {
2579:     // The store to the CookiePtr does not need to be instrumented.
2580:     SI->setNoSanitizeMetadata();
```
- **EN**: This block defines callable entry points like `max`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `max`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2581-2610
```cpp
2581:     llvm::FunctionType *FTy =
2582:         llvm::FunctionType::get(CGM.VoidTy, NumElementsPtr.getType(), false);
2583:     llvm::FunctionCallee F =
2584:         CGM.CreateRuntimeFunction(FTy, "__asan_poison_cxx_array_cookie");
2585:     CGF.Builder.CreateCall(F, NumElementsPtr.emitRawPointer(CGF));
2586:   }
2587: 
2588:   // Finally, compute a pointer to the actual data buffer by skipping
2589:   // over the cookie completely.
2590:   return CGF.Builder.CreateConstInBoundsByteGEP(NewPtr, CookieSize);
2591: }
2592: 
2593: llvm::Value *ItaniumCXXABI::readArrayCookieImpl(CodeGenFunction &CGF,
2594:                                                 Address allocPtr,
2595:                                                 CharUnits cookieSize) {
2596:   // The element size is right-justified in the cookie.
2597:   Address numElementsPtr = allocPtr;
2598:   CharUnits numElementsOffset = cookieSize - CGF.getSizeSize();
2599:   if (!numElementsOffset.isZero())
2600:     numElementsPtr =
2601:       CGF.Builder.CreateConstInBoundsByteGEP(numElementsPtr, numElementsOffset);
2602: 
2603:   unsigned AS = allocPtr.getAddressSpace();
2604:   numElementsPtr = numElementsPtr.withElementType(CGF.SizeTy);
2605:   if (!CGM.getLangOpts().Sanitize.has(SanitizerKind::Address) || AS != 0)
2606:     return CGF.Builder.CreateLoad(numElementsPtr);
2607:   // In asan mode emit a function call instead of a regular load and let the
2608:   // run-time deal with it: if the shadow is properly poisoned return the
2609:   // cookie, otherwise return 0 to avoid an infinite loop calling DTORs.
2610:   // We can't simply ignore this load using nosanitize metadata because
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2611-2640
```cpp
2611:   // the metadata may be lost.
2612:   llvm::FunctionType *FTy =
2613:       llvm::FunctionType::get(CGF.SizeTy, CGF.DefaultPtrTy, false);
2614:   llvm::FunctionCallee F =
2615:       CGM.CreateRuntimeFunction(FTy, "__asan_load_cxx_array_cookie");
2616:   return CGF.Builder.CreateCall(F, numElementsPtr.emitRawPointer(CGF));
2617: }
2618: 
2619: CharUnits ARMCXXABI::getArrayCookieSizeImpl(QualType elementType) {
2620:   // ARM says that the cookie is always:
2621:   //   struct array_cookie {
2622:   //     std::size_t element_size; // element_size != 0
2623:   //     std::size_t element_count;
2624:   //   };
2625:   // But the base ABI doesn't give anything an alignment greater than
2626:   // 8, so we can dismiss this as typical ABI-author blindness to
2627:   // actual language complexity and round up to the element alignment.
2628:   return std::max(CharUnits::fromQuantity(2 * CGM.SizeSizeInBytes),
2629:                   CGM.getContext().getTypeAlignInChars(elementType));
2630: }
2631: 
2632: Address ARMCXXABI::InitializeArrayCookie(CodeGenFunction &CGF,
2633:                                          Address newPtr,
2634:                                          llvm::Value *numElements,
2635:                                          const CXXNewExpr *expr,
2636:                                          QualType elementType) {
2637:   assert(requiresArrayCookie(expr));
2638: 
2639:   // The cookie is always at the start of the buffer.
2640:   Address cookie = newPtr;
```
- **EN**: This block defines callable entry points like `get`, `getArrayCookieSizeImpl`, `max`, `InitializeArrayCookie`; returns or forwards computed values for the surrounding ABI lowering logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `get`, `getArrayCookieSizeImpl`, `max`, `InitializeArrayCookie`；为周围的 ABI 降级 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 2641-2670
```cpp
2641: 
2642:   // The first element is the element size.
2643:   cookie = cookie.withElementType(CGF.SizeTy);
2644:   llvm::Value *elementSize = llvm::ConstantInt::get(CGF.SizeTy,
2645:                  getContext().getTypeSizeInChars(elementType).getQuantity());
2646:   CGF.Builder.CreateStore(elementSize, cookie);
2647: 
2648:   // The second element is the element count.
2649:   cookie = CGF.Builder.CreateConstInBoundsGEP(cookie, 1);
2650:   CGF.Builder.CreateStore(numElements, cookie);
2651: 
2652:   // Finally, compute a pointer to the actual data buffer by skipping
2653:   // over the cookie completely.
2654:   CharUnits cookieSize = ARMCXXABI::getArrayCookieSizeImpl(elementType);
2655:   return CGF.Builder.CreateConstInBoundsByteGEP(newPtr, cookieSize);
2656: }
2657: 
2658: llvm::Value *ARMCXXABI::readArrayCookieImpl(CodeGenFunction &CGF,
2659:                                             Address allocPtr,
2660:                                             CharUnits cookieSize) {
2661:   // The number of elements is at offset sizeof(size_t) relative to
2662:   // the allocated pointer.
2663:   Address numElementsPtr
2664:     = CGF.Builder.CreateConstInBoundsByteGEP(allocPtr, CGF.getSizeSize());
2665: 
2666:   numElementsPtr = numElementsPtr.withElementType(CGF.SizeTy);
2667:   return CGF.Builder.CreateLoad(numElementsPtr);
2668: }
2669: 
2670: /*********************** Static local initialization **************************/
```
- **EN**: This block defines callable entry points like `getContext`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 2671-2700
```cpp
2671: 
2672: static llvm::FunctionCallee getGuardAcquireFn(CodeGenModule &CGM,
2673:                                               llvm::PointerType *GuardPtrTy) {
2674:   // int __cxa_guard_acquire(__guard *guard_object);
2675:   llvm::FunctionType *FTy =
2676:     llvm::FunctionType::get(CGM.getTypes().ConvertType(CGM.getContext().IntTy),
2677:                             GuardPtrTy, /*isVarArg=*/false);
2678:   return CGM.CreateRuntimeFunction(
2679:       FTy, "__cxa_guard_acquire",
2680:       llvm::AttributeList::get(CGM.getLLVMContext(),
2681:                                llvm::AttributeList::FunctionIndex,
2682:                                llvm::Attribute::NoUnwind));
2683: }
2684: 
2685: static llvm::FunctionCallee getGuardReleaseFn(CodeGenModule &CGM,
2686:                                               llvm::PointerType *GuardPtrTy) {
2687:   // void __cxa_guard_release(__guard *guard_object);
2688:   llvm::FunctionType *FTy =
2689:     llvm::FunctionType::get(CGM.VoidTy, GuardPtrTy, /*isVarArg=*/false);
2690:   return CGM.CreateRuntimeFunction(
2691:       FTy, "__cxa_guard_release",
2692:       llvm::AttributeList::get(CGM.getLLVMContext(),
2693:                                llvm::AttributeList::FunctionIndex,
2694:                                llvm::Attribute::NoUnwind));
2695: }
2696: 
2697: static llvm::FunctionCallee getGuardAbortFn(CodeGenModule &CGM,
2698:                                             llvm::PointerType *GuardPtrTy) {
2699:   // void __cxa_guard_abort(__guard *guard_object);
2700:   llvm::FunctionType *FTy =
```
- **EN**: This block defines callable entry points like `getGuardAcquireFn`, `get`, `getGuardReleaseFn`, `getGuardAbortFn`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `getGuardAcquireFn`, `get`, `getGuardReleaseFn`, `getGuardAbortFn`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 2701-2730
```cpp
2701:     llvm::FunctionType::get(CGM.VoidTy, GuardPtrTy, /*isVarArg=*/false);
2702:   return CGM.CreateRuntimeFunction(
2703:       FTy, "__cxa_guard_abort",
2704:       llvm::AttributeList::get(CGM.getLLVMContext(),
2705:                                llvm::AttributeList::FunctionIndex,
2706:                                llvm::Attribute::NoUnwind));
2707: }
2708: 
2709: namespace {
2710:   struct CallGuardAbort final : EHScopeStack::Cleanup {
2711:     llvm::GlobalVariable *Guard;
2712:     CallGuardAbort(llvm::GlobalVariable *Guard) : Guard(Guard) {}
2713: 
2714:     void Emit(CodeGenFunction &CGF, Flags flags) override {
2715:       CGF.EmitNounwindRuntimeCall(getGuardAbortFn(CGF.CGM, Guard->getType()),
2716:                                   Guard);
2717:     }
2718:   };
2719: }
2720: 
2721: /// The ARM code here follows the Itanium code closely enough that we
2722: /// just special-case it at particular places.
2723: void ItaniumCXXABI::EmitGuardedInit(CodeGenFunction &CGF,
2724:                                     const VarDecl &D,
2725:                                     llvm::GlobalVariable *var,
2726:                                     bool shouldPerformInit) {
2727:   CGBuilderTy &Builder = CGF.Builder;
2728: 
2729:   // Inline variables that weren't instantiated from variable templates have
2730:   // partially-ordered initialization within their translation unit.
```
- **EN**: This block introduces declarations such as `CallGuardAbort`; defines callable entry points like `get`, `CallGuardAbort`, `Emit`, `EmitGuardedInit`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块给出诸如 `CallGuardAbort` 的声明；定义可调用入口，例如 `get`, `CallGuardAbort`, `Emit`, `EmitGuardedInit`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 2731-2760
```cpp
2731:   bool NonTemplateInline =
2732:       D.isInline() &&
2733:       !isTemplateInstantiation(D.getTemplateSpecializationKind());
2734: 
2735:   // We only need to use thread-safe statics for local non-TLS variables and
2736:   // inline variables; other global initialization is always single-threaded
2737:   // or (through lazy dynamic loading in multiple threads) unsequenced.
2738:   bool threadsafe = getContext().getLangOpts().ThreadsafeStatics &&
2739:                     (D.isLocalVarDecl() || NonTemplateInline) &&
2740:                     !D.getTLSKind();
2741: 
2742:   // If we have a global variable with internal linkage and thread-safe statics
2743:   // are disabled, we can just let the guard variable be of type i8.
2744:   bool useInt8GuardVariable = !threadsafe && var->hasInternalLinkage();
2745: 
2746:   llvm::IntegerType *guardTy;
2747:   CharUnits guardAlignment;
2748:   if (useInt8GuardVariable) {
2749:     guardTy = CGF.Int8Ty;
2750:     guardAlignment = CharUnits::One();
2751:   } else {
2752:     // Guard variables are 64 bits in the generic ABI and size width on ARM
2753:     // (i.e. 32-bit on AArch32, 64-bit on AArch64).
2754:     if (UseARMGuardVarABI) {
2755:       guardTy = CGF.SizeTy;
2756:       guardAlignment = CGF.getSizeAlign();
2757:     } else {
2758:       guardTy = CGF.Int64Ty;
2759:       guardAlignment =
2760:           CharUnits::fromQuantity(CGM.getDataLayout().getABITypeAlign(guardTy));
```
- **EN**: This block defines callable entry points like `fromQuantity`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `fromQuantity`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2761-2790
```cpp
2761:     }
2762:   }
2763:   llvm::PointerType *guardPtrTy = llvm::PointerType::get(
2764:       CGF.CGM.getLLVMContext(),
2765:       CGF.CGM.getDataLayout().getDefaultGlobalsAddressSpace());
2766: 
2767:   // Create the guard variable if we don't already have it (as we
2768:   // might if we're double-emitting this function body).
2769:   llvm::GlobalVariable *guard = CGM.getStaticLocalDeclGuardAddress(&D);
2770:   if (!guard) {
2771:     // Mangle the name for the guard.
2772:     SmallString<256> guardName;
2773:     {
2774:       llvm::raw_svector_ostream out(guardName);
2775:       getMangleContext().mangleStaticGuardVariable(&D, out);
2776:     }
2777: 
2778:     // Create the guard variable with a zero-initializer.
2779:     // Just absorb linkage, visibility and dll storage class  from the guarded
2780:     // variable.
2781:     guard = new llvm::GlobalVariable(CGM.getModule(), guardTy,
2782:                                      false, var->getLinkage(),
2783:                                      llvm::ConstantInt::get(guardTy, 0),
2784:                                      guardName.str());
2785:     guard->setDSOLocal(var->isDSOLocal());
2786:     guard->setVisibility(var->getVisibility());
2787:     guard->setDLLStorageClass(var->getDLLStorageClass());
2788:     // If the variable is thread-local, so is its guard variable.
2789:     guard->setThreadLocalMode(var->getThreadLocalMode());
2790:     guard->setAlignment(guardAlignment.getAsAlign());
```
- **EN**: This block defines callable entry points like `out`, `getMangleContext`, `get`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `out`, `getMangleContext`, `get`；通过控制流（if）细化 ABI 降级 行为。

### Lines 2791-2820
```cpp
2791: 
2792:     // The ABI says: "It is suggested that it be emitted in the same COMDAT
2793:     // group as the associated data object." In practice, this doesn't work for
2794:     // non-ELF and non-Wasm object formats, so only do it for ELF and Wasm.
2795:     llvm::Comdat *C = var->getComdat();
2796:     if (!D.isLocalVarDecl() && C &&
2797:         (CGM.getTarget().getTriple().isOSBinFormatELF() ||
2798:          CGM.getTarget().getTriple().isOSBinFormatWasm())) {
2799:       guard->setComdat(C);
2800:     } else if (CGM.supportsCOMDAT() && guard->isWeakForLinker()) {
2801:       guard->setComdat(CGM.getModule().getOrInsertComdat(guard->getName()));
2802:     }
2803: 
2804:     CGM.setStaticLocalDeclGuardAddress(&D, guard);
2805:   }
2806: 
2807:   Address guardAddr = Address(guard, guard->getValueType(), guardAlignment);
2808: 
2809:   // Test whether the variable has completed initialization.
2810:   //
2811:   // Itanium C++ ABI 3.3.2:
2812:   //   The following is pseudo-code showing how these functions can be used:
2813:   //     if (obj_guard.first_byte == 0) {
2814:   //       if ( __cxa_guard_acquire (&obj_guard) ) {
2815:   //         try {
2816:   //           ... initialize the object ...;
2817:   //         } catch (...) {
2818:   //            __cxa_guard_abort (&obj_guard);
2819:   //            throw;
2820:   //         }
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 2821-2850
```cpp
2821:   //         ... queue object destructor with __cxa_atexit() ...;
2822:   //         __cxa_guard_release (&obj_guard);
2823:   //       }
2824:   //     }
2825:   //
2826:   // If threadsafe statics are enabled, but we don't have inline atomics, just
2827:   // call __cxa_guard_acquire unconditionally.  The "inline" check isn't
2828:   // actually inline, and the user might not expect calls to __atomic libcalls.
2829: 
2830:   unsigned MaxInlineWidthInBits = CGF.getTarget().getMaxAtomicInlineWidth();
2831:   llvm::BasicBlock *EndBlock = CGF.createBasicBlock("init.end");
2832:   if (!threadsafe || MaxInlineWidthInBits) {
2833:     // Load the first byte of the guard variable.
2834:     llvm::LoadInst *LI =
2835:         Builder.CreateLoad(guardAddr.withElementType(CGM.Int8Ty));
2836: 
2837:     // Itanium ABI:
2838:     //   An implementation supporting thread-safety on multiprocessor
2839:     //   systems must also guarantee that references to the initialized
2840:     //   object do not occur before the load of the initialization flag.
2841:     //
2842:     // In LLVM, we do this by marking the load Acquire.
2843:     if (threadsafe)
2844:       LI->setAtomic(llvm::AtomicOrdering::Acquire);
2845: 
2846:     // For ARM, we should only check the first bit, rather than the entire byte:
2847:     //
2848:     // ARM C++ ABI 3.2.3.1:
2849:     //   To support the potential use of initialization guard variables
2850:     //   as semaphores that are the target of ARM SWP and LDREX/STREX
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 2851-2880
```cpp
2851:     //   synchronizing instructions we define a static initialization
2852:     //   guard variable to be a 4-byte aligned, 4-byte word with the
2853:     //   following inline access protocol.
2854:     //     #define INITIALIZED 1
2855:     //     if ((obj_guard & INITIALIZED) != INITIALIZED) {
2856:     //       if (__cxa_guard_acquire(&obj_guard))
2857:     //         ...
2858:     //     }
2859:     //
2860:     // and similarly for ARM64:
2861:     //
2862:     // ARM64 C++ ABI 3.2.2:
2863:     //   This ABI instead only specifies the value bit 0 of the static guard
2864:     //   variable; all other bits are platform defined. Bit 0 shall be 0 when the
2865:     //   variable is not initialized and 1 when it is.
2866:     llvm::Value *V =
2867:         (UseARMGuardVarABI && !useInt8GuardVariable)
2868:             ? Builder.CreateAnd(LI, llvm::ConstantInt::get(CGM.Int8Ty, 1))
2869:             : LI;
2870:     llvm::Value *NeedsInit = Builder.CreateIsNull(V, "guard.uninitialized");
2871: 
2872:     llvm::BasicBlock *InitCheckBlock = CGF.createBasicBlock("init.check");
2873: 
2874:     // Check if the first byte of the guard variable is zero.
2875:     CGF.EmitCXXGuardedInitBranch(NeedsInit, InitCheckBlock, EndBlock,
2876:                                  CodeGenFunction::GuardKind::VariableGuard, &D);
2877: 
2878:     CGF.EmitBlock(InitCheckBlock);
2879:   }
2880: 
```
- **EN**: This block documents intent or context for the surrounding ABI lowering code.
- **CN**: 该代码块说明周围 ABI 降级 代码的意图或上下文。

### Lines 2881-2910
```cpp
2881:   // The semantics of dynamic initialization of variables with static or thread
2882:   // storage duration depends on whether they are declared at block-scope. The
2883:   // initialization of such variables at block-scope can be aborted with an
2884:   // exception and later retried (per C++20 [stmt.dcl]p4), and recursive entry
2885:   // to their initialization has undefined behavior (also per C++20
2886:   // [stmt.dcl]p4). For such variables declared at non-block scope, exceptions
2887:   // lead to termination (per C++20 [except.terminate]p1), and recursive
2888:   // references to the variables are governed only by the lifetime rules (per
2889:   // C++20 [class.cdtor]p2), which means such references are perfectly fine as
2890:   // long as they avoid touching memory. As a result, block-scope variables must
2891:   // not be marked as initialized until after initialization completes (unless
2892:   // the mark is reverted following an exception), but non-block-scope variables
2893:   // must be marked prior to initialization so that recursive accesses during
2894:   // initialization do not restart initialization.
2895: 
2896:   // Variables used when coping with thread-safe statics and exceptions.
2897:   if (threadsafe) {
2898:     // Call __cxa_guard_acquire.
2899:     llvm::Value *V
2900:       = CGF.EmitNounwindRuntimeCall(getGuardAcquireFn(CGM, guardPtrTy), guard);
2901: 
2902:     llvm::BasicBlock *InitBlock = CGF.createBasicBlock("init");
2903: 
2904:     Builder.CreateCondBr(Builder.CreateIsNotNull(V, "tobool"),
2905:                          InitBlock, EndBlock);
2906: 
2907:     // Call __cxa_guard_abort along the exceptional edge.
2908:     CGF.EHStack.pushCleanup<CallGuardAbort>(EHCleanup, guard);
2909: 
2910:     CGF.EmitBlock(InitBlock);
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 2911-2940
```cpp
2911:   } else if (!D.isLocalVarDecl()) {
2912:     // For non-local variables, store 1 into the first byte of the guard
2913:     // variable before the object initialization begins so that references
2914:     // to the variable during initialization don't restart initialization.
2915:     Builder.CreateStore(llvm::ConstantInt::get(CGM.Int8Ty, 1),
2916:                         guardAddr.withElementType(CGM.Int8Ty));
2917:   }
2918: 
2919:   // Emit the initializer and add a global destructor if appropriate.
2920:   CGF.EmitCXXGlobalVarDeclInit(D, var, shouldPerformInit);
2921: 
2922:   if (threadsafe) {
2923:     // Pop the guard-abort cleanup if we pushed one.
2924:     CGF.PopCleanupBlock();
2925: 
2926:     // Call __cxa_guard_release.  This cannot throw.
2927:     CGF.EmitNounwindRuntimeCall(getGuardReleaseFn(CGM, guardPtrTy),
2928:                                 guardAddr.emitRawPointer(CGF));
2929:   } else if (D.isLocalVarDecl()) {
2930:     // For local variables, store 1 into the first byte of the guard variable
2931:     // after the object initialization completes so that initialization is
2932:     // retried if initialization is interrupted by an exception.
2933:     Builder.CreateStore(llvm::ConstantInt::get(CGM.Int8Ty, 1),
2934:                         guardAddr.withElementType(CGM.Int8Ty));
2935:   }
2936: 
2937:   CGF.EmitBlock(EndBlock);
2938: }
2939: 
2940: /// Register a global destructor using __cxa_atexit.
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 2941-2970
```cpp
2941: static void emitGlobalDtorWithCXAAtExit(CodeGenFunction &CGF,
2942:                                         llvm::FunctionCallee dtor,
2943:                                         llvm::Constant *addr, bool TLS) {
2944:   assert(!CGF.getTarget().getTriple().isOSAIX() &&
2945:          "unexpected call to emitGlobalDtorWithCXAAtExit");
2946:   assert((TLS || CGF.getTypes().getCodeGenOpts().CXAAtExit) &&
2947:          "__cxa_atexit is disabled");
2948:   const char *Name = "__cxa_atexit";
2949:   if (TLS) {
2950:     const llvm::Triple &T = CGF.getTarget().getTriple();
2951:     Name = T.isOSDarwin() ?  "_tlv_atexit" : "__cxa_thread_atexit";
2952:   }
2953: 
2954:   // We're assuming that the destructor function is something we can
2955:   // reasonably call with the default CC.
2956:   llvm::Type *dtorTy = CGF.DefaultPtrTy;
2957: 
2958:   // Preserve address space of addr.
2959:   auto AddrAS = addr ? addr->getType()->getPointerAddressSpace() : 0;
2960:   auto AddrPtrTy = AddrAS ? llvm::PointerType::get(CGF.getLLVMContext(), AddrAS)
2961:                           : CGF.Int8PtrTy;
2962: 
2963:   // Create a variable that binds the atexit to this shared object.
2964:   llvm::Constant *handle =
2965:       CGF.CGM.CreateRuntimeVariable(CGF.Int8Ty, "__dso_handle");
2966:   auto *GV = cast<llvm::GlobalValue>(handle->stripPointerCasts());
2967:   GV->setVisibility(llvm::GlobalValue::HiddenVisibility);
2968: 
2969:   // extern "C" int __cxa_atexit(void (*f)(void *), void *p, void *d);
2970:   llvm::Type *paramTys[] = {dtorTy, AddrPtrTy, handle->getType()};
```
- **EN**: This block defines callable entry points like `emitGlobalDtorWithCXAAtExit`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitGlobalDtorWithCXAAtExit`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 2971-3000
```cpp
2971:   llvm::FunctionType *atexitTy =
2972:     llvm::FunctionType::get(CGF.IntTy, paramTys, false);
2973: 
2974:   // Fetch the actual function.
2975:   llvm::FunctionCallee atexit = CGF.CGM.CreateRuntimeFunction(atexitTy, Name);
2976:   if (llvm::Function *fn = dyn_cast<llvm::Function>(atexit.getCallee()))
2977:     fn->setDoesNotThrow();
2978: 
2979:   const auto &Context = CGF.CGM.getContext();
2980:   FunctionProtoType::ExtProtoInfo EPI(Context.getDefaultCallingConvention(
2981:       /*IsVariadic=*/false, /*IsCXXMethod=*/false));
2982:   QualType fnType =
2983:       Context.getFunctionType(Context.VoidTy, {Context.VoidPtrTy}, EPI);
2984:   llvm::Value *dtorCallee = dtor.getCallee();
2985:   dtorCallee =
2986:       CGF.CGM.getFunctionPointer(cast<llvm::Constant>(dtorCallee), fnType);
2987: 
2988:   if (dtorCallee->getType()->getPointerAddressSpace() != AddrAS)
2989:     dtorCallee = CGF.performAddrSpaceCast(dtorCallee, AddrPtrTy);
2990: 
2991:   if (!addr)
2992:     // addr is null when we are trying to register a dtor annotated with
2993:     // __attribute__((destructor)) in a constructor function. Using null here is
2994:     // okay because this argument is just passed back to the destructor
2995:     // function.
2996:     addr = llvm::Constant::getNullValue(CGF.Int8PtrTy);
2997: 
2998:   llvm::Value *args[] = {dtorCallee, addr, handle};
2999:   CGF.EmitNounwindRuntimeCall(atexit, args);
3000: }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3001-3030
```cpp
3001: 
3002: static llvm::Function *createGlobalInitOrCleanupFn(CodeGen::CodeGenModule &CGM,
3003:                                                    StringRef FnName) {
3004:   // Create a function that registers/unregisters destructors that have the same
3005:   // priority.
3006:   llvm::FunctionType *FTy = llvm::FunctionType::get(CGM.VoidTy, false);
3007:   llvm::Function *GlobalInitOrCleanupFn = CGM.CreateGlobalInitOrCleanUpFunction(
3008:       FTy, FnName, CGM.getTypes().arrangeNullaryFunction(), SourceLocation());
3009: 
3010:   return GlobalInitOrCleanupFn;
3011: }
3012: 
3013: void CodeGenModule::unregisterGlobalDtorsWithUnAtExit() {
3014:   for (const auto &I : DtorsUsingAtExit) {
3015:     int Priority = I.first;
3016:     std::string GlobalCleanupFnName =
3017:         std::string("__GLOBAL_cleanup_") + llvm::to_string(Priority);
3018: 
3019:     llvm::Function *GlobalCleanupFn =
3020:         createGlobalInitOrCleanupFn(*this, GlobalCleanupFnName);
3021: 
3022:     CodeGenFunction CGF(*this);
3023:     CGF.StartFunction(GlobalDecl(), getContext().VoidTy, GlobalCleanupFn,
3024:                       getTypes().arrangeNullaryFunction(), FunctionArgList(),
3025:                       SourceLocation(), SourceLocation());
3026:     auto AL = ApplyDebugLocation::CreateArtificial(CGF);
3027: 
3028:     // Get the destructor function type, void(*)(void).
3029:     llvm::FunctionType *dtorFuncTy = llvm::FunctionType::get(CGF.VoidTy, false);
3030: 
```
- **EN**: This block defines callable entry points like `unregisterGlobalDtorsWithUnAtExit`, `string`, `createGlobalInitOrCleanupFn`, `CGF`, `getTypes`; uses control flow (for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `unregisterGlobalDtorsWithUnAtExit`, `string`, `createGlobalInitOrCleanupFn`, `CGF`, `getTypes`；通过控制流（for）细化 ABI 降级 行为。

### Lines 3031-3060
```cpp
3031:     // Destructor functions are run/unregistered in non-ascending
3032:     // order of their priorities.
3033:     const llvm::TinyPtrVector<llvm::Function *> &Dtors = I.second;
3034:     auto itv = Dtors.rbegin();
3035:     while (itv != Dtors.rend()) {
3036:       llvm::Function *Dtor = *itv;
3037: 
3038:       // We're assuming that the destructor function is something we can
3039:       // reasonably call with the correct CC.
3040:       llvm::Value *V = CGF.unregisterGlobalDtorWithUnAtExit(Dtor);
3041:       llvm::Value *NeedsDestruct =
3042:           CGF.Builder.CreateIsNull(V, "needs_destruct");
3043: 
3044:       llvm::BasicBlock *DestructCallBlock =
3045:           CGF.createBasicBlock("destruct.call");
3046:       llvm::BasicBlock *EndBlock = CGF.createBasicBlock(
3047:           (itv + 1) != Dtors.rend() ? "unatexit.call" : "destruct.end");
3048:       // Check if unatexit returns a value of 0. If it does, jump to
3049:       // DestructCallBlock, otherwise jump to EndBlock directly.
3050:       CGF.Builder.CreateCondBr(NeedsDestruct, DestructCallBlock, EndBlock);
3051: 
3052:       CGF.EmitBlock(DestructCallBlock);
3053: 
3054:       // Emit the call to casted Dtor.
3055:       llvm::CallInst *CI = CGF.Builder.CreateCall(dtorFuncTy, Dtor);
3056:       // Make sure the call and the callee agree on calling convention.
3057:       CI->setCallingConv(Dtor->getCallingConv());
3058: 
3059:       CGF.EmitBlock(EndBlock);
3060: 
```
- **EN**: This block uses control flow (while) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（while）细化 ABI 降级 行为。

### Lines 3061-3090
```cpp
3061:       itv++;
3062:     }
3063: 
3064:     CGF.FinishFunction();
3065:     AddGlobalDtor(GlobalCleanupFn, Priority);
3066:   }
3067: }
3068: 
3069: void CodeGenModule::registerGlobalDtorsWithAtExit() {
3070:   for (const auto &I : DtorsUsingAtExit) {
3071:     int Priority = I.first;
3072:     std::string GlobalInitFnName =
3073:         std::string("__GLOBAL_init_") + llvm::to_string(Priority);
3074:     llvm::Function *GlobalInitFn =
3075:         createGlobalInitOrCleanupFn(*this, GlobalInitFnName);
3076: 
3077:     CodeGenFunction CGF(*this);
3078:     CGF.StartFunction(GlobalDecl(), getContext().VoidTy, GlobalInitFn,
3079:                       getTypes().arrangeNullaryFunction(), FunctionArgList(),
3080:                       SourceLocation(), SourceLocation());
3081:     auto AL = ApplyDebugLocation::CreateArtificial(CGF);
3082: 
3083:     // Since constructor functions are run in non-descending order of their
3084:     // priorities, destructors are registered in non-descending order of their
3085:     // priorities, and since destructor functions are run in the reverse order
3086:     // of their registration, destructor functions are run in non-ascending
3087:     // order of their priorities.
3088:     const llvm::TinyPtrVector<llvm::Function *> &Dtors = I.second;
3089:     for (auto *Dtor : Dtors) {
3090:       // Register the destructor function calling __cxa_atexit if it is
```
- **EN**: This block defines callable entry points like `AddGlobalDtor`, `registerGlobalDtorsWithAtExit`, `string`, `createGlobalInitOrCleanupFn`, `CGF`; uses control flow (for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `AddGlobalDtor`, `registerGlobalDtorsWithAtExit`, `string`, `createGlobalInitOrCleanupFn`, `CGF`；通过控制流（for）细化 ABI 降级 行为。

### Lines 3091-3120
```cpp
3091:       // available. Otherwise fall back on calling atexit.
3092:       if (getCodeGenOpts().CXAAtExit) {
3093:         emitGlobalDtorWithCXAAtExit(CGF, Dtor, nullptr, false);
3094:       } else {
3095:         // We're assuming that the destructor function is something we can
3096:         // reasonably call with the correct CC.
3097:         CGF.registerGlobalDtorWithAtExit(Dtor);
3098:       }
3099:     }
3100: 
3101:     CGF.FinishFunction();
3102:     AddGlobalCtor(GlobalInitFn, Priority);
3103:   }
3104: 
3105:   if (getCXXABI().useSinitAndSterm())
3106:     unregisterGlobalDtorsWithUnAtExit();
3107: }
3108: 
3109: /// Register a global destructor as best as we know how.
3110: void ItaniumCXXABI::registerGlobalDtor(CodeGenFunction &CGF, const VarDecl &D,
3111:                                        llvm::FunctionCallee dtor,
3112:                                        llvm::Constant *addr) {
3113:   if (D.isNoDestroy(CGM.getContext()))
3114:     return;
3115: 
3116:   // HLSL doesn't support atexit.
3117:   if (CGM.getLangOpts().HLSL)
3118:     return CGM.AddCXXDtorEntry(dtor, addr);
3119: 
3120:   // OpenMP offloading supports C++ constructors and destructors but we do not
```
- **EN**: This block defines callable entry points like `emitGlobalDtorWithCXAAtExit`, `AddGlobalCtor`, `registerGlobalDtor`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitGlobalDtorWithCXAAtExit`, `AddGlobalCtor`, `registerGlobalDtor`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3121-3150
```cpp
3121:   // always have 'atexit' available. Instead lower these to use the LLVM global
3122:   // destructors which we can handle directly in the runtime. Note that this is
3123:   // not strictly 1-to-1 with using `atexit` because we no longer tear down
3124:   // globals in reverse order of when they were constructed.
3125:   if (!CGM.getLangOpts().hasAtExit() && !D.isStaticLocal())
3126:     return CGF.registerGlobalDtorWithLLVM(D, dtor, addr);
3127: 
3128:   // emitGlobalDtorWithCXAAtExit will emit a call to either __cxa_thread_atexit
3129:   // or __cxa_atexit depending on whether this VarDecl is a thread-local storage
3130:   // or not. CXAAtExit controls only __cxa_atexit, so use it if it is enabled.
3131:   // We can always use __cxa_thread_atexit.
3132:   if (CGM.getCodeGenOpts().CXAAtExit || D.getTLSKind())
3133:     return emitGlobalDtorWithCXAAtExit(CGF, dtor, addr, D.getTLSKind());
3134: 
3135:   // In Apple kexts, we want to add a global destructor entry.
3136:   // FIXME: shouldn't this be guarded by some variable?
3137:   if (CGM.getLangOpts().AppleKext) {
3138:     // Generate a global destructor entry.
3139:     return CGM.AddCXXDtorEntry(dtor, addr);
3140:   }
3141: 
3142:   CGF.registerGlobalDtorWithAtExit(D, dtor, addr);
3143: }
3144: 
3145: static bool isThreadWrapperReplaceable(const VarDecl *VD,
3146:                                        CodeGen::CodeGenModule &CGM) {
3147:   assert(!VD->isStaticLocal() && "static local VarDecls don't need wrappers!");
3148:   // Darwin prefers to have references to thread local variables to go through
3149:   // the thread wrapper instead of directly referencing the backing variable.
3150:   return VD->getTLSKind() == VarDecl::TLS_Dynamic &&
```
- **EN**: This block defines callable entry points like `isThreadWrapperReplaceable`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `isThreadWrapperReplaceable`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3151-3180
```cpp
3151:          CGM.getTarget().getTriple().isOSDarwin();
3152: }
3153: 
3154: /// Get the appropriate linkage for the wrapper function. This is essentially
3155: /// the weak form of the variable's linkage; every translation unit which needs
3156: /// the wrapper emits a copy, and we want the linker to merge them.
3157: static llvm::GlobalValue::LinkageTypes
3158: getThreadLocalWrapperLinkage(const VarDecl *VD, CodeGen::CodeGenModule &CGM) {
3159:   llvm::GlobalValue::LinkageTypes VarLinkage =
3160:       CGM.getLLVMLinkageVarDefinition(VD);
3161: 
3162:   // For internal linkage variables, we don't need an external or weak wrapper.
3163:   if (llvm::GlobalValue::isLocalLinkage(VarLinkage))
3164:     return VarLinkage;
3165: 
3166:   // If the thread wrapper is replaceable, give it appropriate linkage.
3167:   if (isThreadWrapperReplaceable(VD, CGM))
3168:     if (!llvm::GlobalVariable::isLinkOnceLinkage(VarLinkage) &&
3169:         !llvm::GlobalVariable::isWeakODRLinkage(VarLinkage))
3170:       return VarLinkage;
3171:   return llvm::GlobalValue::WeakODRLinkage;
3172: }
3173: 
3174: llvm::Function *
3175: ItaniumCXXABI::getOrCreateThreadLocalWrapper(const VarDecl *VD,
3176:                                              llvm::Value *Val) {
3177:   // Mangle the name for the thread_local wrapper function.
3178:   SmallString<256> WrapperName;
3179:   {
3180:     llvm::raw_svector_ostream Out(WrapperName);
```
- **EN**: This block defines callable entry points like `getThreadLocalWrapperLinkage`, `getOrCreateThreadLocalWrapper`, `Out`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getThreadLocalWrapperLinkage`, `getOrCreateThreadLocalWrapper`, `Out`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3181-3210
```cpp
3181:     getMangleContext().mangleItaniumThreadLocalWrapper(VD, Out);
3182:   }
3183: 
3184:   // FIXME: If VD is a definition, we should regenerate the function attributes
3185:   // before returning.
3186:   if (llvm::Value *V = CGM.getModule().getNamedValue(WrapperName))
3187:     return cast<llvm::Function>(V);
3188: 
3189:   QualType RetQT = VD->getType();
3190:   if (RetQT->isReferenceType())
3191:     RetQT = RetQT.getNonReferenceType();
3192: 
3193:   const CGFunctionInfo &FI = CGM.getTypes().arrangeBuiltinFunctionDeclaration(
3194:       getContext().getPointerType(RetQT), FunctionArgList());
3195: 
3196:   llvm::FunctionType *FnTy = CGM.getTypes().GetFunctionType(FI);
3197:   llvm::Function *Wrapper =
3198:       llvm::Function::Create(FnTy, getThreadLocalWrapperLinkage(VD, CGM),
3199:                              WrapperName.str(), &CGM.getModule());
3200: 
3201:   if (CGM.supportsCOMDAT() && Wrapper->isWeakForLinker())
3202:     Wrapper->setComdat(CGM.getModule().getOrInsertComdat(Wrapper->getName()));
3203: 
3204:   CGM.SetLLVMFunctionAttributes(GlobalDecl(), FI, Wrapper, /*IsThunk=*/false);
3205: 
3206:   // Always resolve references to the wrapper at link time.
3207:   if (!Wrapper->hasLocalLinkage())
3208:     if (!isThreadWrapperReplaceable(VD, CGM) ||
3209:         llvm::GlobalVariable::isLinkOnceLinkage(Wrapper->getLinkage()) ||
3210:         llvm::GlobalVariable::isWeakODRLinkage(Wrapper->getLinkage()) ||
```
- **EN**: This block spells out callable entry points like `getMangleContext`, `getContext`, `Create`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `getMangleContext`, `getContext`, `Create`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3211-3240
```cpp
3211:         VD->getVisibility() == HiddenVisibility)
3212:       Wrapper->setVisibility(llvm::GlobalValue::HiddenVisibility);
3213: 
3214:   if (isThreadWrapperReplaceable(VD, CGM)) {
3215:     Wrapper->setCallingConv(llvm::CallingConv::CXX_FAST_TLS);
3216:     Wrapper->addFnAttr(llvm::Attribute::NoUnwind);
3217:   }
3218: 
3219:   ThreadWrappers.push_back({VD, Wrapper});
3220:   return Wrapper;
3221: }
3222: 
3223: void ItaniumCXXABI::EmitThreadLocalInitFuncs(
3224:     CodeGenModule &CGM, ArrayRef<const VarDecl *> CXXThreadLocals,
3225:     ArrayRef<llvm::Function *> CXXThreadLocalInits,
3226:     ArrayRef<const VarDecl *> CXXThreadLocalInitVars) {
3227:   llvm::Function *InitFunc = nullptr;
3228: 
3229:   // Separate initializers into those with ordered (or partially-ordered)
3230:   // initialization and those with unordered initialization.
3231:   llvm::SmallVector<llvm::Function *, 8> OrderedInits;
3232:   llvm::SmallDenseMap<const VarDecl *, llvm::Function *> UnorderedInits;
3233:   for (unsigned I = 0; I != CXXThreadLocalInits.size(); ++I) {
3234:     if (isTemplateInstantiation(
3235:             CXXThreadLocalInitVars[I]->getTemplateSpecializationKind()))
3236:       UnorderedInits[CXXThreadLocalInitVars[I]->getCanonicalDecl()] =
3237:           CXXThreadLocalInits[I];
3238:     else
3239:       OrderedInits.push_back(CXXThreadLocalInits[I]);
3240:   }
```
- **EN**: This block defines callable entry points like `EmitThreadLocalInitFuncs`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitThreadLocalInitFuncs`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 3241-3270
```cpp
3241: 
3242:   if (!OrderedInits.empty()) {
3243:     // Generate a guarded initialization function.
3244:     llvm::FunctionType *FTy =
3245:         llvm::FunctionType::get(CGM.VoidTy, /*isVarArg=*/false);
3246:     const CGFunctionInfo &FI = CGM.getTypes().arrangeNullaryFunction();
3247:     InitFunc = CGM.CreateGlobalInitOrCleanUpFunction(FTy, "__tls_init", FI,
3248:                                                      SourceLocation(),
3249:                                                      /*TLS=*/true);
3250:     llvm::GlobalVariable *Guard = new llvm::GlobalVariable(
3251:         CGM.getModule(), CGM.Int8Ty, /*isConstant=*/false,
3252:         llvm::GlobalVariable::InternalLinkage,
3253:         llvm::ConstantInt::get(CGM.Int8Ty, 0), "__tls_guard");
3254:     Guard->setThreadLocal(true);
3255:     Guard->setThreadLocalMode(CGM.GetDefaultLLVMTLSModel());
3256: 
3257:     CharUnits GuardAlign = CharUnits::One();
3258:     Guard->setAlignment(GuardAlign.getAsAlign());
3259: 
3260:     CodeGenFunction(CGM).GenerateCXXGlobalInitFunc(
3261:         InitFunc, OrderedInits, ConstantAddress(Guard, CGM.Int8Ty, GuardAlign));
3262:     // On Darwin platforms, use CXX_FAST_TLS calling convention.
3263:     if (CGM.getTarget().getTriple().isOSDarwin()) {
3264:       InitFunc->setCallingConv(llvm::CallingConv::CXX_FAST_TLS);
3265:       InitFunc->addFnAttr(llvm::Attribute::NoUnwind);
3266:     }
3267:   }
3268: 
3269:   // Create declarations for thread wrappers for all thread-local variables
3270:   // with non-discardable definitions in this translation unit.
```
- **EN**: This block defines callable entry points like `get`, `SourceLocation`, `CodeGenFunction`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`, `SourceLocation`, `CodeGenFunction`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3271-3300
```cpp
3271:   for (const VarDecl *VD : CXXThreadLocals) {
3272:     if (VD->hasDefinition() &&
3273:         !isDiscardableGVALinkage(getContext().GetGVALinkageForVariable(VD))) {
3274:       llvm::GlobalValue *GV = CGM.GetGlobalValue(CGM.getMangledName(VD));
3275:       getOrCreateThreadLocalWrapper(VD, GV);
3276:     }
3277:   }
3278: 
3279:   // Emit all referenced thread wrappers.
3280:   for (auto VDAndWrapper : ThreadWrappers) {
3281:     const VarDecl *VD = VDAndWrapper.first;
3282:     llvm::GlobalVariable *Var =
3283:         cast<llvm::GlobalVariable>(CGM.GetGlobalValue(CGM.getMangledName(VD)));
3284:     llvm::Function *Wrapper = VDAndWrapper.second;
3285: 
3286:     // Some targets require that all access to thread local variables go through
3287:     // the thread wrapper.  This means that we cannot attempt to create a thread
3288:     // wrapper or a thread helper.
3289:     if (!VD->hasDefinition()) {
3290:       if (isThreadWrapperReplaceable(VD, CGM)) {
3291:         Wrapper->setLinkage(llvm::Function::ExternalLinkage);
3292:         continue;
3293:       }
3294: 
3295:       // If this isn't a TU in which this variable is defined, the thread
3296:       // wrapper is discardable.
3297:       if (Wrapper->getLinkage() == llvm::Function::WeakODRLinkage)
3298:         Wrapper->setLinkage(llvm::Function::LinkOnceODRLinkage);
3299:     }
3300: 
```
- **EN**: This block defines callable entry points like `getOrCreateThreadLocalWrapper`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getOrCreateThreadLocalWrapper`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 3301-3330
```cpp
3301:     CGM.SetLLVMFunctionAttributesForDefinition(nullptr, Wrapper);
3302: 
3303:     // Mangle the name for the thread_local initialization function.
3304:     SmallString<256> InitFnName;
3305:     {
3306:       llvm::raw_svector_ostream Out(InitFnName);
3307:       getMangleContext().mangleItaniumThreadLocalInit(VD, Out);
3308:     }
3309: 
3310:     llvm::FunctionType *InitFnTy = llvm::FunctionType::get(CGM.VoidTy, false);
3311: 
3312:     // If we have a definition for the variable, emit the initialization
3313:     // function as an alias to the global Init function (if any). Otherwise,
3314:     // produce a declaration of the initialization function.
3315:     llvm::GlobalValue *Init = nullptr;
3316:     bool InitIsInitFunc = false;
3317:     bool HasConstantInitialization = false;
3318:     if (!usesThreadWrapperFunction(VD)) {
3319:       HasConstantInitialization = true;
3320:     } else if (VD->hasDefinition()) {
3321:       InitIsInitFunc = true;
3322:       llvm::Function *InitFuncToUse = InitFunc;
3323:       if (isTemplateInstantiation(VD->getTemplateSpecializationKind()))
3324:         InitFuncToUse = UnorderedInits.lookup(VD->getCanonicalDecl());
3325:       if (InitFuncToUse)
3326:         Init = llvm::GlobalAlias::create(Var->getLinkage(), InitFnName.str(),
3327:                                          InitFuncToUse);
3328:     } else {
3329:       // Emit a weak global function referring to the initialization function.
3330:       // This function will not exist if the TU defining the thread_local
```
- **EN**: This block defines callable entry points like `Out`, `getMangleContext`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `Out`, `getMangleContext`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3331-3360
```cpp
3331:       // variable in question does not need any dynamic initialization for
3332:       // its thread_local variables.
3333:       Init = llvm::Function::Create(InitFnTy,
3334:                                     llvm::GlobalVariable::ExternalWeakLinkage,
3335:                                     InitFnName.str(), &CGM.getModule());
3336:       const CGFunctionInfo &FI = CGM.getTypes().arrangeNullaryFunction();
3337:       CGM.SetLLVMFunctionAttributes(
3338:           GlobalDecl(), FI, cast<llvm::Function>(Init), /*IsThunk=*/false);
3339:     }
3340: 
3341:     if (Init) {
3342:       Init->setVisibility(Var->getVisibility());
3343:       // Don't mark an extern_weak function DSO local on windows.
3344:       if (!CGM.getTriple().isOSWindows() || !Init->hasExternalWeakLinkage())
3345:         Init->setDSOLocal(Var->isDSOLocal());
3346:     }
3347: 
3348:     llvm::LLVMContext &Context = CGM.getModule().getContext();
3349: 
3350:     // The linker on AIX is not happy with missing weak symbols.  However,
3351:     // other TUs will not know whether the initialization routine exists
3352:     // so create an empty, init function to satisfy the linker.
3353:     // This is needed whenever a thread wrapper function is not used, and
3354:     // also when the symbol is weak.
3355:     if (CGM.getTriple().isOSAIX() && VD->hasDefinition() &&
3356:         isEmittedWithConstantInitializer(VD, true) &&
3357:         !mayNeedDestruction(VD)) {
3358:       // Init should be null.  If it were non-null, then the logic above would
3359:       // either be defining the function to be an alias or declaring the
3360:       // function with the expectation that the definition of the variable
```
- **EN**: This block defines callable entry points like `GlobalDecl`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `GlobalDecl`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3361-3390
```cpp
3361:       // is elsewhere.
3362:       assert(Init == nullptr && "Expected Init to be null.");
3363: 
3364:       llvm::Function *Func = llvm::Function::Create(
3365:           InitFnTy, Var->getLinkage(), InitFnName.str(), &CGM.getModule());
3366:       const CGFunctionInfo &FI = CGM.getTypes().arrangeNullaryFunction();
3367:       CGM.SetLLVMFunctionAttributes(GlobalDecl(), FI,
3368:                                     cast<llvm::Function>(Func),
3369:                                     /*IsThunk=*/false);
3370:       // Create a function body that just returns
3371:       llvm::BasicBlock *Entry = llvm::BasicBlock::Create(Context, "", Func);
3372:       CGBuilderTy Builder(CGM, Entry);
3373:       Builder.CreateRetVoid();
3374:     }
3375: 
3376:     llvm::BasicBlock *Entry = llvm::BasicBlock::Create(Context, "", Wrapper);
3377:     CGBuilderTy Builder(CGM, Entry);
3378:     if (HasConstantInitialization) {
3379:       // No dynamic initialization to invoke.
3380:     } else if (InitIsInitFunc) {
3381:       if (Init) {
3382:         llvm::CallInst *CallVal = Builder.CreateCall(InitFnTy, Init);
3383:         if (isThreadWrapperReplaceable(VD, CGM)) {
3384:           CallVal->setCallingConv(llvm::CallingConv::CXX_FAST_TLS);
3385:           llvm::Function *Fn =
3386:               cast<llvm::Function>(cast<llvm::GlobalAlias>(Init)->getAliasee());
3387:           Fn->setCallingConv(llvm::CallingConv::CXX_FAST_TLS);
3388:         }
3389:       }
3390:     } else if (CGM.getTriple().isOSAIX()) {
```
- **EN**: This block defines callable entry points like `Builder`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Builder`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3391-3420
```cpp
3391:       // On AIX, except if constinit and also neither of class type or of
3392:       // (possibly multi-dimensional) array of class type, thread_local vars
3393:       // will have init routines regardless of whether they are
3394:       // const-initialized.  Since the routine is guaranteed to exist, we can
3395:       // unconditionally call it without testing for its existance.  This
3396:       // avoids potentially unresolved weak symbols which the AIX linker
3397:       // isn't happy with.
3398:       Builder.CreateCall(InitFnTy, Init);
3399:     } else {
3400:       // Don't know whether we have an init function. Call it if it exists.
3401:       llvm::Value *Have = Builder.CreateIsNotNull(Init);
3402:       llvm::BasicBlock *InitBB = llvm::BasicBlock::Create(Context, "", Wrapper);
3403:       llvm::BasicBlock *ExitBB = llvm::BasicBlock::Create(Context, "", Wrapper);
3404:       Builder.CreateCondBr(Have, InitBB, ExitBB);
3405: 
3406:       Builder.SetInsertPoint(InitBB);
3407:       Builder.CreateCall(InitFnTy, Init);
3408:       Builder.CreateBr(ExitBB);
3409: 
3410:       Builder.SetInsertPoint(ExitBB);
3411:     }
3412: 
3413:     // For a reference, the result of the wrapper function is a pointer to
3414:     // the referenced object.
3415:     llvm::Value *Val = Builder.CreateThreadLocalAddress(Var);
3416: 
3417:     if (VD->getType()->isReferenceType()) {
3418:       CharUnits Align = CGM.getContext().getDeclAlign(VD);
3419:       Val = Builder.CreateAlignedLoad(Var->getValueType(), Val, Align);
3420:     }
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 3421-3450
```cpp
3421:     Val = Builder.CreateAddrSpaceCast(Val, Wrapper->getReturnType());
3422: 
3423:     Builder.CreateRet(Val);
3424:   }
3425: }
3426: 
3427: LValue ItaniumCXXABI::EmitThreadLocalVarDeclLValue(CodeGenFunction &CGF,
3428:                                                    const VarDecl *VD,
3429:                                                    QualType LValType) {
3430:   llvm::Value *Val = CGF.CGM.GetAddrOfGlobalVar(VD);
3431:   llvm::Function *Wrapper = getOrCreateThreadLocalWrapper(VD, Val);
3432: 
3433:   llvm::CallInst *CallVal = CGF.Builder.CreateCall(Wrapper);
3434:   CallVal->setCallingConv(Wrapper->getCallingConv());
3435: 
3436:   LValue LV;
3437:   if (VD->getType()->isReferenceType())
3438:     LV = CGF.MakeNaturalAlignRawAddrLValue(CallVal, LValType);
3439:   else
3440:     LV = CGF.MakeRawAddrLValue(CallVal, LValType,
3441:                                CGF.getContext().getDeclAlign(VD));
3442:   // FIXME: need setObjCGCLValueClass?
3443:   return LV;
3444: }
3445: 
3446: /// Return whether the given global decl needs a VTT parameter, which it does
3447: /// if it's a base constructor or destructor with virtual bases.
3448: bool ItaniumCXXABI::NeedsVTTParameter(GlobalDecl GD) {
3449:   const CXXMethodDecl *MD = cast<CXXMethodDecl>(GD.getDecl());
3450: 
```
- **EN**: This block defines callable entry points like `EmitThreadLocalVarDeclLValue`, `NeedsVTTParameter`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitThreadLocalVarDeclLValue`, `NeedsVTTParameter`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3451-3480
```cpp
3451:   // We don't have any virtual bases, just return early.
3452:   if (!MD->getParent()->getNumVBases())
3453:     return false;
3454: 
3455:   // Check if we have a base constructor.
3456:   if (isa<CXXConstructorDecl>(MD) && GD.getCtorType() == Ctor_Base)
3457:     return true;
3458: 
3459:   // Check if we have a base destructor.
3460:   if (isa<CXXDestructorDecl>(MD) && GD.getDtorType() == Dtor_Base)
3461:     return true;
3462: 
3463:   return false;
3464: }
3465: 
3466: llvm::Constant *
3467: ItaniumCXXABI::getOrCreateVirtualFunctionPointerThunk(const CXXMethodDecl *MD) {
3468:   SmallString<256> MethodName;
3469:   llvm::raw_svector_ostream Out(MethodName);
3470:   getMangleContext().mangleCXXName(MD, Out);
3471:   MethodName += "_vfpthunk_";
3472:   StringRef ThunkName = MethodName.str();
3473:   llvm::Function *ThunkFn;
3474:   if ((ThunkFn = cast_or_null<llvm::Function>(
3475:            CGM.getModule().getNamedValue(ThunkName))))
3476:     return ThunkFn;
3477: 
3478:   const CGFunctionInfo &FnInfo = CGM.getTypes().arrangeCXXMethodDeclaration(MD);
3479:   llvm::FunctionType *ThunkTy = CGM.getTypes().GetFunctionType(FnInfo);
3480:   llvm::GlobalValue::LinkageTypes Linkage =
```
- **EN**: This block defines callable entry points like `getOrCreateVirtualFunctionPointerThunk`, `Out`, `getMangleContext`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getOrCreateVirtualFunctionPointerThunk`, `Out`, `getMangleContext`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3481-3510
```cpp
3481:       MD->isExternallyVisible() ? llvm::GlobalValue::LinkOnceODRLinkage
3482:                                 : llvm::GlobalValue::InternalLinkage;
3483:   ThunkFn =
3484:       llvm::Function::Create(ThunkTy, Linkage, ThunkName, &CGM.getModule());
3485:   if (Linkage == llvm::GlobalValue::LinkOnceODRLinkage)
3486:     ThunkFn->setVisibility(llvm::GlobalValue::HiddenVisibility);
3487:   assert(ThunkFn->getName() == ThunkName && "name was uniqued!");
3488: 
3489:   CGM.SetLLVMFunctionAttributes(MD, FnInfo, ThunkFn, /*IsThunk=*/true);
3490:   CGM.SetLLVMFunctionAttributesForDefinition(MD, ThunkFn);
3491: 
3492:   // Stack protection sometimes gets inserted after the musttail call.
3493:   ThunkFn->removeFnAttr(llvm::Attribute::StackProtect);
3494:   ThunkFn->removeFnAttr(llvm::Attribute::StackProtectStrong);
3495:   ThunkFn->removeFnAttr(llvm::Attribute::StackProtectReq);
3496: 
3497:   // Start codegen.
3498:   CodeGenFunction CGF(CGM);
3499:   CGF.CurGD = GlobalDecl(MD);
3500:   CGF.CurFuncIsThunk = true;
3501: 
3502:   // Build FunctionArgs.
3503:   FunctionArgList FunctionArgs;
3504:   CGF.BuildFunctionArgList(CGF.CurGD, FunctionArgs);
3505: 
3506:   CGF.StartFunction(GlobalDecl(), FnInfo.getReturnType(), ThunkFn, FnInfo,
3507:                     FunctionArgs, MD->getLocation(), SourceLocation());
3508: 
3509:   // Emit an artificial location for this function.
3510:   auto AL = ApplyDebugLocation::CreateArtificial(CGF);
```
- **EN**: This block spells out callable entry points like `Create`, `CGF`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出可调用入口的声明，例如 `Create`, `CGF`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 3511-3540
```cpp
3511: 
3512:   llvm::Value *ThisVal = loadIncomingCXXThis(CGF);
3513:   setCXXABIThisValue(CGF, ThisVal);
3514: 
3515:   CallArgList CallArgs;
3516:   for (const VarDecl *VD : FunctionArgs)
3517:     CGF.EmitDelegateCallArg(CallArgs, VD, SourceLocation());
3518: 
3519:   const FunctionProtoType *FPT = MD->getType()->getAs<FunctionProtoType>();
3520:   RequiredArgs Required = RequiredArgs::forPrototypePlus(FPT, /*this*/ 1);
3521:   const CGFunctionInfo &CallInfo =
3522:       CGM.getTypes().arrangeCXXMethodCall(CallArgs, FPT, Required, 0);
3523:   CGCallee Callee = CGCallee::forVirtual(nullptr, GlobalDecl(MD),
3524:                                          getThisAddress(CGF), ThunkTy);
3525:   llvm::CallBase *CallOrInvoke;
3526:   CGF.EmitCall(CallInfo, Callee, ReturnValueSlot(), CallArgs, &CallOrInvoke,
3527:                /*IsMustTail=*/true, SourceLocation(), true);
3528:   auto *Call = cast<llvm::CallInst>(CallOrInvoke);
3529:   Call->setTailCallKind(llvm::CallInst::TCK_MustTail);
3530:   if (Call->getType()->isVoidTy())
3531:     CGF.Builder.CreateRetVoid();
3532:   else
3533:     CGF.Builder.CreateRet(Call);
3534: 
3535:   // Finish the function to maintain CodeGenFunction invariants.
3536:   // FIXME: Don't emit unreachable code.
3537:   CGF.EmitBlock(CGF.createBasicBlock());
3538:   CGF.FinishFunction();
3539:   return ThunkFn;
3540: }
```
- **EN**: This block spells out callable entry points like `setCXXABIThisValue`, `getThisAddress`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `setCXXABIThisValue`, `getThisAddress`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 3541-3570
```cpp
3541: 
3542: namespace {
3543: class ItaniumRTTIBuilder {
3544:   CodeGenModule &CGM;  // Per-module state.
3545:   llvm::LLVMContext &VMContext;
3546:   const ItaniumCXXABI &CXXABI;  // Per-module state.
3547: 
3548:   /// Fields - The fields of the RTTI descriptor currently being built.
3549:   SmallVector<llvm::Constant *, 16> Fields;
3550: 
3551:   /// GetAddrOfTypeName - Returns the mangled type name of the given type.
3552:   llvm::GlobalVariable *
3553:   GetAddrOfTypeName(QualType Ty, llvm::GlobalVariable::LinkageTypes Linkage);
3554: 
3555:   /// GetAddrOfExternalRTTIDescriptor - Returns the constant for the RTTI
3556:   /// descriptor of the given type.
3557:   llvm::Constant *GetAddrOfExternalRTTIDescriptor(QualType Ty);
3558: 
3559:   /// BuildVTablePointer - Build the vtable pointer for the given type.
3560:   void BuildVTablePointer(const Type *Ty, llvm::Constant *StorageAddress);
3561: 
3562:   /// BuildSIClassTypeInfo - Build an abi::__si_class_type_info, used for single
3563:   /// inheritance, according to the Itanium C++ ABI, 2.9.5p6b.
3564:   void BuildSIClassTypeInfo(const CXXRecordDecl *RD);
3565: 
3566:   /// BuildVMIClassTypeInfo - Build an abi::__vmi_class_type_info, used for
3567:   /// classes with bases that do not satisfy the abi::__si_class_type_info
3568:   /// constraints, according ti the Itanium C++ ABI, 2.9.5p5c.
3569:   void BuildVMIClassTypeInfo(const CXXRecordDecl *RD);
3570: 
```
- **EN**: This block introduces declarations such as `ItaniumRTTIBuilder`; defines callable entry points like `GetAddrOfTypeName`, `BuildVTablePointer`, `BuildSIClassTypeInfo`, `BuildVMIClassTypeInfo`.
- **CN**: 该代码块给出诸如 `ItaniumRTTIBuilder` 的声明；定义可调用入口，例如 `GetAddrOfTypeName`, `BuildVTablePointer`, `BuildSIClassTypeInfo`, `BuildVMIClassTypeInfo`。

### Lines 3571-3600
```cpp
3571:   /// BuildPointerTypeInfo - Build an abi::__pointer_type_info struct, used
3572:   /// for pointer types.
3573:   void BuildPointerTypeInfo(QualType PointeeTy);
3574: 
3575:   /// BuildObjCObjectTypeInfo - Build the appropriate kind of
3576:   /// type_info for an object type.
3577:   void BuildObjCObjectTypeInfo(const ObjCObjectType *Ty);
3578: 
3579:   /// BuildPointerToMemberTypeInfo - Build an abi::__pointer_to_member_type_info
3580:   /// struct, used for member pointer types.
3581:   void BuildPointerToMemberTypeInfo(const MemberPointerType *Ty);
3582: 
3583: public:
3584:   ItaniumRTTIBuilder(const ItaniumCXXABI &ABI)
3585:       : CGM(ABI.CGM), VMContext(CGM.getModule().getContext()), CXXABI(ABI) {}
3586: 
3587:   // Pointer type info flags.
3588:   enum {
3589:     /// PTI_Const - Type has const qualifier.
3590:     PTI_Const = 0x1,
3591: 
3592:     /// PTI_Volatile - Type has volatile qualifier.
3593:     PTI_Volatile = 0x2,
3594: 
3595:     /// PTI_Restrict - Type has restrict qualifier.
3596:     PTI_Restrict = 0x4,
3597: 
3598:     /// PTI_Incomplete - Type is incomplete.
3599:     PTI_Incomplete = 0x8,
3600: 
```
- **EN**: This block defines callable entry points like `BuildPointerTypeInfo`, `BuildObjCObjectTypeInfo`, `BuildPointerToMemberTypeInfo`, `ItaniumRTTIBuilder`.
- **CN**: 该代码块定义可调用入口，例如 `BuildPointerTypeInfo`, `BuildObjCObjectTypeInfo`, `BuildPointerToMemberTypeInfo`, `ItaniumRTTIBuilder`。

### Lines 3601-3630
```cpp
3601:     /// PTI_ContainingClassIncomplete - Containing class is incomplete.
3602:     /// (in pointer to member).
3603:     PTI_ContainingClassIncomplete = 0x10,
3604: 
3605:     /// PTI_TransactionSafe - Pointee is transaction_safe function (C++ TM TS).
3606:     //PTI_TransactionSafe = 0x20,
3607: 
3608:     /// PTI_Noexcept - Pointee is noexcept function (C++1z).
3609:     PTI_Noexcept = 0x40,
3610:   };
3611: 
3612:   // VMI type info flags.
3613:   enum {
3614:     /// VMI_NonDiamondRepeat - Class has non-diamond repeated inheritance.
3615:     VMI_NonDiamondRepeat = 0x1,
3616: 
3617:     /// VMI_DiamondShaped - Class is diamond shaped.
3618:     VMI_DiamondShaped = 0x2
3619:   };
3620: 
3621:   // Base class type info flags.
3622:   enum {
3623:     /// BCTI_Virtual - Base class is virtual.
3624:     BCTI_Virtual = 0x1,
3625: 
3626:     /// BCTI_Public - Base class is public.
3627:     BCTI_Public = 0x2
3628:   };
3629: 
3630:   /// BuildTypeInfo - Build the RTTI type info struct for the given type, or
```
- **EN**: This block documents intent or context for the surrounding ABI lowering code.
- **CN**: 该代码块说明周围 ABI 降级 代码的意图或上下文。

### Lines 3631-3660
```cpp
3631:   /// link to an existing RTTI descriptor if one already exists.
3632:   llvm::Constant *BuildTypeInfo(QualType Ty);
3633: 
3634:   /// BuildTypeInfo - Build the RTTI type info struct for the given type.
3635:   llvm::Constant *BuildTypeInfo(
3636:       QualType Ty,
3637:       llvm::GlobalVariable::LinkageTypes Linkage,
3638:       llvm::GlobalValue::VisibilityTypes Visibility,
3639:       llvm::GlobalValue::DLLStorageClassTypes DLLStorageClass);
3640: };
3641: }
3642: 
3643: llvm::GlobalVariable *ItaniumRTTIBuilder::GetAddrOfTypeName(
3644:     QualType Ty, llvm::GlobalVariable::LinkageTypes Linkage) {
3645:   SmallString<256> Name;
3646:   llvm::raw_svector_ostream Out(Name);
3647:   CGM.getCXXABI().getMangleContext().mangleCXXRTTIName(Ty, Out);
3648: 
3649:   // We know that the mangled name of the type starts at index 4 of the
3650:   // mangled name of the typename, so we can just index into it in order to
3651:   // get the mangled name of the type.
3652:   llvm::Constant *Init;
3653:   if (CGM.getTriple().isOSzOS()) {
3654:     // On z/OS, typename is stored as 2 encodings: EBCDIC followed by ASCII.
3655:     SmallString<256> DualEncodedName;
3656:     llvm::ConverterEBCDIC::convertToEBCDIC(Name.substr(4), DualEncodedName);
3657:     DualEncodedName += '\0';
3658:     DualEncodedName += Name.substr(4);
3659:     Init = llvm::ConstantDataArray::getString(VMContext, DualEncodedName);
3660:   } else
```
- **EN**: This block defines callable entry points like `Out`, `convertToEBCDIC`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `Out`, `convertToEBCDIC`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3661-3690
```cpp
3661:     Init = llvm::ConstantDataArray::getString(VMContext, Name.substr(4));
3662: 
3663:   auto Align = CGM.getContext().getTypeAlignInChars(CGM.getContext().CharTy);
3664: 
3665:   llvm::GlobalVariable *GV = CGM.CreateOrReplaceCXXRuntimeVariable(
3666:       Name, Init->getType(), Linkage, Align.getAsAlign());
3667: 
3668:   GV->setInitializer(Init);
3669: 
3670:   return GV;
3671: }
3672: 
3673: llvm::Constant *
3674: ItaniumRTTIBuilder::GetAddrOfExternalRTTIDescriptor(QualType Ty) {
3675:   // Mangle the RTTI name.
3676:   SmallString<256> Name;
3677:   llvm::raw_svector_ostream Out(Name);
3678:   CGM.getCXXABI().getMangleContext().mangleCXXRTTI(Ty, Out);
3679: 
3680:   // Look for an existing global.
3681:   llvm::GlobalVariable *GV = CGM.getModule().getNamedGlobal(Name);
3682: 
3683:   if (!GV) {
3684:     // Create a new global variable.
3685:     // Note for the future: If we would ever like to do deferred emission of
3686:     // RTTI, check if emitting vtables opportunistically need any adjustment.
3687: 
3688:     GV = new llvm::GlobalVariable(
3689:         CGM.getModule(), CGM.GlobalsInt8PtrTy,
3690:         /*isConstant=*/true, llvm::GlobalValue::ExternalLinkage, nullptr, Name);
```
- **EN**: This block defines callable entry points like `GetAddrOfExternalRTTIDescriptor`, `Out`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `GetAddrOfExternalRTTIDescriptor`, `Out`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3691-3720
```cpp
3691:     const CXXRecordDecl *RD = Ty->getAsCXXRecordDecl();
3692:     CGM.setGVProperties(GV, RD);
3693:     // Import the typeinfo symbol when all non-inline virtual methods are
3694:     // imported.
3695:     if (CGM.getTarget().hasPS4DLLImportExport()) {
3696:       if (RD && CXXRecordNonInlineHasAttr<DLLImportAttr>(RD)) {
3697:         GV->setDLLStorageClass(llvm::GlobalVariable::DLLImportStorageClass);
3698:         CGM.setDSOLocal(GV);
3699:       }
3700:     }
3701:   }
3702: 
3703:   return GV;
3704: }
3705: 
3706: /// TypeInfoIsInStandardLibrary - Given a builtin type, returns whether the type
3707: /// info for that type is defined in the standard library.
3708: static bool TypeInfoIsInStandardLibrary(const BuiltinType *Ty) {
3709:   // Itanium C++ ABI 2.9.2:
3710:   //   Basic type information (e.g. for "int", "bool", etc.) will be kept in
3711:   //   the run-time support library. Specifically, the run-time support
3712:   //   library should contain type_info objects for the types X, X* and
3713:   //   X const*, for every X in: void, std::nullptr_t, bool, wchar_t, char,
3714:   //   unsigned char, signed char, short, unsigned short, int, unsigned int,
3715:   //   long, unsigned long, long long, unsigned long long, float, double,
3716:   //   long double, char16_t, char32_t, and the IEEE 754r decimal and
3717:   //   half-precision floating point types.
3718:   //
3719:   // GCC also emits RTTI for __int128.
3720:   // FIXME: We do not emit RTTI information for decimal types here.
```
- **EN**: This block defines callable entry points like `TypeInfoIsInStandardLibrary`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `TypeInfoIsInStandardLibrary`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3721-3750
```cpp
3721: 
3722:   // Types added here must also be added to EmitFundamentalRTTIDescriptors.
3723:   switch (Ty->getKind()) {
3724:     case BuiltinType::Void:
3725:     case BuiltinType::NullPtr:
3726:     case BuiltinType::Bool:
3727:     case BuiltinType::WChar_S:
3728:     case BuiltinType::WChar_U:
3729:     case BuiltinType::Char_U:
3730:     case BuiltinType::Char_S:
3731:     case BuiltinType::UChar:
3732:     case BuiltinType::SChar:
3733:     case BuiltinType::Short:
3734:     case BuiltinType::UShort:
3735:     case BuiltinType::Int:
3736:     case BuiltinType::UInt:
3737:     case BuiltinType::Long:
3738:     case BuiltinType::ULong:
3739:     case BuiltinType::LongLong:
3740:     case BuiltinType::ULongLong:
3741:     case BuiltinType::Half:
3742:     case BuiltinType::Float:
3743:     case BuiltinType::Double:
3744:     case BuiltinType::LongDouble:
3745:     case BuiltinType::Float16:
3746:     case BuiltinType::Float128:
3747:     case BuiltinType::Ibm128:
3748:     case BuiltinType::Char8:
3749:     case BuiltinType::Char16:
3750:     case BuiltinType::Char32:
```
- **EN**: This block uses control flow (switch, case) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 ABI 降级 行为。

### Lines 3751-3780
```cpp
3751:     case BuiltinType::Int128:
3752:     case BuiltinType::UInt128:
3753:       return true;
3754: 
3755: #define IMAGE_TYPE(ImgType, Id, SingletonId, Access, Suffix) \
3756:     case BuiltinType::Id:
3757: #include "clang/Basic/OpenCLImageTypes.def"
3758: #define EXT_OPAQUE_TYPE(ExtType, Id, Ext) \
3759:     case BuiltinType::Id:
3760: #include "clang/Basic/OpenCLExtensionTypes.def"
3761:     case BuiltinType::OCLSampler:
3762:     case BuiltinType::OCLEvent:
3763:     case BuiltinType::OCLClkEvent:
3764:     case BuiltinType::OCLQueue:
3765:     case BuiltinType::OCLReserveID:
3766: #define SVE_TYPE(Name, Id, SingletonId) \
3767:     case BuiltinType::Id:
3768: #include "clang/Basic/AArch64ACLETypes.def"
3769: #define PPC_VECTOR_TYPE(Name, Id, Size) \
3770:     case BuiltinType::Id:
3771: #include "clang/Basic/PPCTypes.def"
3772: #define RVV_TYPE(Name, Id, SingletonId) case BuiltinType::Id:
3773: #include "clang/Basic/RISCVVTypes.def"
3774: #define WASM_TYPE(Name, Id, SingletonId) case BuiltinType::Id:
3775: #include "clang/Basic/WebAssemblyReferenceTypes.def"
3776: #define AMDGPU_TYPE(Name, Id, SingletonId, Width, Align) case BuiltinType::Id:
3777: #include "clang/Basic/AMDGPUTypes.def"
3778: #define HLSL_INTANGIBLE_TYPE(Name, Id, SingletonId) case BuiltinType::Id:
3779: #include "clang/Basic/HLSLIntangibleTypes.def"
3780:     case BuiltinType::ShortAccum:
```
- **EN**: This block imports Clang headers `clang/Basic/OpenCLImageTypes.def`, `clang/Basic/OpenCLExtensionTypes.def`, `clang/Basic/AArch64ACLETypes.def`, and 5 more; uses control flow (case) to specialize ABI lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/Basic/OpenCLImageTypes.def`, `clang/Basic/OpenCLExtensionTypes.def`, `clang/Basic/AArch64ACLETypes.def`, and 5 more；通过控制流（case）细化 ABI 降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 3781-3810
```cpp
3781:     case BuiltinType::Accum:
3782:     case BuiltinType::LongAccum:
3783:     case BuiltinType::UShortAccum:
3784:     case BuiltinType::UAccum:
3785:     case BuiltinType::ULongAccum:
3786:     case BuiltinType::ShortFract:
3787:     case BuiltinType::Fract:
3788:     case BuiltinType::LongFract:
3789:     case BuiltinType::UShortFract:
3790:     case BuiltinType::UFract:
3791:     case BuiltinType::ULongFract:
3792:     case BuiltinType::SatShortAccum:
3793:     case BuiltinType::SatAccum:
3794:     case BuiltinType::SatLongAccum:
3795:     case BuiltinType::SatUShortAccum:
3796:     case BuiltinType::SatUAccum:
3797:     case BuiltinType::SatULongAccum:
3798:     case BuiltinType::SatShortFract:
3799:     case BuiltinType::SatFract:
3800:     case BuiltinType::SatLongFract:
3801:     case BuiltinType::SatUShortFract:
3802:     case BuiltinType::SatUFract:
3803:     case BuiltinType::SatULongFract:
3804:     case BuiltinType::BFloat16:
3805:       return false;
3806: 
3807:     case BuiltinType::Dependent:
3808: #define BUILTIN_TYPE(Id, SingletonId)
3809: #define PLACEHOLDER_TYPE(Id, SingletonId) \
3810:     case BuiltinType::Id:
```
- **EN**: This block uses control flow (case) to specialize ABI lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块通过控制流（case）细化 ABI 降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 3811-3840
```cpp
3811: #include "clang/AST/BuiltinTypes.def"
3812:       llvm_unreachable("asking for RRTI for a placeholder type!");
3813: 
3814:     case BuiltinType::ObjCId:
3815:     case BuiltinType::ObjCClass:
3816:     case BuiltinType::ObjCSel:
3817:       llvm_unreachable("FIXME: Objective-C types are unsupported!");
3818:   }
3819: 
3820:   llvm_unreachable("Invalid BuiltinType Kind!");
3821: }
3822: 
3823: static bool TypeInfoIsInStandardLibrary(const PointerType *PointerTy) {
3824:   QualType PointeeTy = PointerTy->getPointeeType();
3825:   const BuiltinType *BuiltinTy = dyn_cast<BuiltinType>(PointeeTy);
3826:   if (!BuiltinTy)
3827:     return false;
3828: 
3829:   // Check the qualifiers.
3830:   Qualifiers Quals = PointeeTy.getQualifiers();
3831:   Quals.removeConst();
3832: 
3833:   if (!Quals.empty())
3834:     return false;
3835: 
3836:   return TypeInfoIsInStandardLibrary(BuiltinTy);
3837: }
3838: 
3839: /// IsStandardLibraryRTTIDescriptor - Returns whether the type
3840: /// information for the given type exists in the standard library.
```
- **EN**: This block imports Clang headers `clang/AST/BuiltinTypes.def`; defines callable entry points like `TypeInfoIsInStandardLibrary`; uses control flow (if, for, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/AST/BuiltinTypes.def`；定义可调用入口，例如 `TypeInfoIsInStandardLibrary`；通过控制流（if, for, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 3841-3870
```cpp
3841: static bool IsStandardLibraryRTTIDescriptor(QualType Ty) {
3842:   // Type info for builtin types is defined in the standard library.
3843:   if (const BuiltinType *BuiltinTy = dyn_cast<BuiltinType>(Ty))
3844:     return TypeInfoIsInStandardLibrary(BuiltinTy);
3845: 
3846:   // Type info for some pointer types to builtin types is defined in the
3847:   // standard library.
3848:   if (const PointerType *PointerTy = dyn_cast<PointerType>(Ty))
3849:     return TypeInfoIsInStandardLibrary(PointerTy);
3850: 
3851:   return false;
3852: }
3853: 
3854: /// ShouldUseExternalRTTIDescriptor - Returns whether the type information for
3855: /// the given type exists somewhere else, and that we should not emit the type
3856: /// information in this translation unit.  Assumes that it is not a
3857: /// standard-library type.
3858: static bool ShouldUseExternalRTTIDescriptor(CodeGenModule &CGM,
3859:                                             QualType Ty) {
3860:   ASTContext &Context = CGM.getContext();
3861: 
3862:   // If RTTI is disabled, assume it might be disabled in the
3863:   // translation unit that defines any potential key function, too.
3864:   if (!Context.getLangOpts().RTTI) return false;
3865: 
3866:   if (const RecordType *RecordTy = dyn_cast<RecordType>(Ty)) {
3867:     const CXXRecordDecl *RD =
3868:         cast<CXXRecordDecl>(RecordTy->getDecl())->getDefinitionOrSelf();
3869:     if (!RD->hasDefinition())
3870:       return false;
```
- **EN**: This block defines callable entry points like `IsStandardLibraryRTTIDescriptor`, `ShouldUseExternalRTTIDescriptor`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `IsStandardLibraryRTTIDescriptor`, `ShouldUseExternalRTTIDescriptor`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3871-3900
```cpp
3871: 
3872:     if (!RD->isDynamicClass())
3873:       return false;
3874: 
3875:     // FIXME: this may need to be reconsidered if the key function
3876:     // changes.
3877:     // N.B. We must always emit the RTTI data ourselves if there exists a key
3878:     // function.
3879:     bool IsDLLImport = RD->hasAttr<DLLImportAttr>();
3880: 
3881:     // Don't import the RTTI but emit it locally.
3882:     if (CGM.getTriple().isOSCygMing())
3883:       return false;
3884: 
3885:     if (CGM.getVTables().isVTableExternal(RD)) {
3886:       if (CGM.getTarget().hasPS4DLLImportExport())
3887:         return true;
3888: 
3889:       return IsDLLImport && !CGM.getTriple().isWindowsItaniumEnvironment()
3890:                  ? false
3891:                  : true;
3892:     }
3893:     if (IsDLLImport)
3894:       return true;
3895:   }
3896: 
3897:   return false;
3898: }
3899: 
3900: /// IsIncompleteClassType - Returns whether the given record type is incomplete.
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 3901-3930
```cpp
3901: static bool IsIncompleteClassType(const RecordType *RecordTy) {
3902:   return !RecordTy->getDecl()->getDefinitionOrSelf()->isCompleteDefinition();
3903: }
3904: 
3905: /// ContainsIncompleteClassType - Returns whether the given type contains an
3906: /// incomplete class type. This is true if
3907: ///
3908: ///   * The given type is an incomplete class type.
3909: ///   * The given type is a pointer type whose pointee type contains an
3910: ///     incomplete class type.
3911: ///   * The given type is a member pointer type whose class is an incomplete
3912: ///     class type.
3913: ///   * The given type is a member pointer type whoise pointee type contains an
3914: ///     incomplete class type.
3915: /// is an indirect or direct pointer to an incomplete class type.
3916: static bool ContainsIncompleteClassType(QualType Ty) {
3917:   if (const RecordType *RecordTy = dyn_cast<RecordType>(Ty)) {
3918:     if (IsIncompleteClassType(RecordTy))
3919:       return true;
3920:   }
3921: 
3922:   if (const PointerType *PointerTy = dyn_cast<PointerType>(Ty))
3923:     return ContainsIncompleteClassType(PointerTy->getPointeeType());
3924: 
3925:   if (const MemberPointerType *MemberPointerTy =
3926:       dyn_cast<MemberPointerType>(Ty)) {
3927:     // Check if the class type is incomplete.
3928:     if (!MemberPointerTy->getMostRecentCXXRecordDecl()->hasDefinition())
3929:       return true;
3930: 
```
- **EN**: This block defines callable entry points like `IsIncompleteClassType`, `ContainsIncompleteClassType`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `IsIncompleteClassType`, `ContainsIncompleteClassType`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3931-3960
```cpp
3931:     return ContainsIncompleteClassType(MemberPointerTy->getPointeeType());
3932:   }
3933: 
3934:   return false;
3935: }
3936: 
3937: // CanUseSingleInheritance - Return whether the given record decl has a "single,
3938: // public, non-virtual base at offset zero (i.e. the derived class is dynamic
3939: // iff the base is)", according to Itanium C++ ABI, 2.95p6b.
3940: static bool CanUseSingleInheritance(const CXXRecordDecl *RD) {
3941:   // Check the number of bases.
3942:   if (RD->getNumBases() != 1)
3943:     return false;
3944: 
3945:   // Get the base.
3946:   CXXRecordDecl::base_class_const_iterator Base = RD->bases_begin();
3947: 
3948:   // Check that the base is not virtual.
3949:   if (Base->isVirtual())
3950:     return false;
3951: 
3952:   // Check that the base is public.
3953:   if (Base->getAccessSpecifier() != AS_public)
3954:     return false;
3955: 
3956:   // Check that the class is dynamic iff the base is.
3957:   auto *BaseDecl = Base->getType()->castAsCXXRecordDecl();
3958:   if (!BaseDecl->isEmpty() &&
3959:       BaseDecl->isDynamicClass() != RD->isDynamicClass())
3960:     return false;
```
- **EN**: This block defines callable entry points like `ContainsIncompleteClassType`, `CanUseSingleInheritance`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `ContainsIncompleteClassType`, `CanUseSingleInheritance`；通过控制流（if）细化 ABI 降级 行为。

### Lines 3961-3990
```cpp
3961: 
3962:   return true;
3963: }
3964: 
3965: void ItaniumRTTIBuilder::BuildVTablePointer(const Type *Ty,
3966:                                             llvm::Constant *StorageAddress) {
3967:   // abi::__class_type_info.
3968:   static const char * const ClassTypeInfo =
3969:     "_ZTVN10__cxxabiv117__class_type_infoE";
3970:   // abi::__si_class_type_info.
3971:   static const char * const SIClassTypeInfo =
3972:     "_ZTVN10__cxxabiv120__si_class_type_infoE";
3973:   // abi::__vmi_class_type_info.
3974:   static const char * const VMIClassTypeInfo =
3975:     "_ZTVN10__cxxabiv121__vmi_class_type_infoE";
3976: 
3977:   const char *VTableName = nullptr;
3978: 
3979:   switch (Ty->getTypeClass()) {
3980: #define TYPE(Class, Base)
3981: #define ABSTRACT_TYPE(Class, Base)
3982: #define NON_CANONICAL_UNLESS_DEPENDENT_TYPE(Class, Base) case Type::Class:
3983: #define NON_CANONICAL_TYPE(Class, Base) case Type::Class:
3984: #define DEPENDENT_TYPE(Class, Base) case Type::Class:
3985: #include "clang/AST/TypeNodes.inc"
3986:     llvm_unreachable("Non-canonical and dependent types shouldn't get here");
3987: 
3988:   case Type::LValueReference:
3989:   case Type::RValueReference:
3990:     llvm_unreachable("References shouldn't get here");
```
- **EN**: This block imports Clang headers `clang/AST/TypeNodes.inc`; defines callable entry points like `BuildVTablePointer`; uses control flow (switch, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/AST/TypeNodes.inc`；定义可调用入口，例如 `BuildVTablePointer`；通过控制流（switch, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 3991-4020
```cpp
3991: 
3992:   case Type::Auto:
3993:   case Type::DeducedTemplateSpecialization:
3994:     llvm_unreachable("Undeduced type shouldn't get here");
3995: 
3996:   case Type::Pipe:
3997:     llvm_unreachable("Pipe types shouldn't get here");
3998: 
3999:   case Type::ArrayParameter:
4000:     llvm_unreachable("Array Parameter types should not get here.");
4001: 
4002:   case Type::Builtin:
4003:   case Type::BitInt:
4004:   case Type::OverflowBehavior:
4005:   // GCC treats vector and complex types as fundamental types.
4006:   case Type::Vector:
4007:   case Type::ExtVector:
4008:   case Type::ConstantMatrix:
4009:   case Type::Complex:
4010:   case Type::Atomic:
4011:   // FIXME: GCC treats block pointers as fundamental types?!
4012:   case Type::BlockPointer:
4013:     // abi::__fundamental_type_info.
4014:     VTableName = "_ZTVN10__cxxabiv123__fundamental_type_infoE";
4015:     break;
4016: 
4017:   case Type::ConstantArray:
4018:   case Type::IncompleteArray:
4019:   case Type::VariableArray:
4020:     // abi::__array_type_info.
```
- **EN**: This block uses control flow (case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4021-4050
```cpp
4021:     VTableName = "_ZTVN10__cxxabiv117__array_type_infoE";
4022:     break;
4023: 
4024:   case Type::FunctionNoProto:
4025:   case Type::FunctionProto:
4026:     // abi::__function_type_info.
4027:     VTableName = "_ZTVN10__cxxabiv120__function_type_infoE";
4028:     break;
4029: 
4030:   case Type::Enum:
4031:     // abi::__enum_type_info.
4032:     VTableName = "_ZTVN10__cxxabiv116__enum_type_infoE";
4033:     break;
4034: 
4035:   case Type::Record: {
4036:     const auto *RD = cast<CXXRecordDecl>(cast<RecordType>(Ty)->getDecl())
4037:                          ->getDefinitionOrSelf();
4038: 
4039:     if (!RD->hasDefinition() || !RD->getNumBases()) {
4040:       VTableName = ClassTypeInfo;
4041:     } else if (CanUseSingleInheritance(RD)) {
4042:       VTableName = SIClassTypeInfo;
4043:     } else {
4044:       VTableName = VMIClassTypeInfo;
4045:     }
4046: 
4047:     break;
4048:   }
4049: 
4050:   case Type::ObjCObject:
```
- **EN**: This block uses control flow (if, case) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if, case）细化 ABI 降级 行为。

### Lines 4051-4080
```cpp
4051:     // Ignore protocol qualifiers.
4052:     Ty = cast<ObjCObjectType>(Ty)->getBaseType().getTypePtr();
4053: 
4054:     // Handle id and Class.
4055:     if (isa<BuiltinType>(Ty)) {
4056:       VTableName = ClassTypeInfo;
4057:       break;
4058:     }
4059: 
4060:     assert(isa<ObjCInterfaceType>(Ty));
4061:     [[fallthrough]];
4062: 
4063:   case Type::ObjCInterface:
4064:     if (cast<ObjCInterfaceType>(Ty)->getDecl()->getSuperClass()) {
4065:       VTableName = SIClassTypeInfo;
4066:     } else {
4067:       VTableName = ClassTypeInfo;
4068:     }
4069:     break;
4070: 
4071:   case Type::ObjCObjectPointer:
4072:   case Type::Pointer:
4073:     // abi::__pointer_type_info.
4074:     VTableName = "_ZTVN10__cxxabiv119__pointer_type_infoE";
4075:     break;
4076: 
4077:   case Type::MemberPointer:
4078:     // abi::__pointer_to_member_type_info.
4079:     VTableName = "_ZTVN10__cxxabiv129__pointer_to_member_type_infoE";
4080:     break;
```
- **EN**: This block uses control flow (if, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4081-4110
```cpp
4081: 
4082:   case Type::HLSLAttributedResource:
4083:   case Type::HLSLInlineSpirv:
4084:     llvm_unreachable("HLSL doesn't support virtual functions");
4085:   }
4086: 
4087:   llvm::Constant *VTable = nullptr;
4088: 
4089:   // Check if the alias exists. If it doesn't, then get or create the global.
4090:   if (CGM.getLangOpts().RelativeCXXABIVTables)
4091:     VTable = CGM.getModule().getNamedAlias(VTableName);
4092:   if (!VTable) {
4093:     llvm::Type *Ty = llvm::ArrayType::get(CGM.GlobalsInt8PtrTy, 0);
4094:     VTable = CGM.getModule().getOrInsertGlobal(VTableName, Ty);
4095:   }
4096: 
4097:   CGM.setDSOLocal(cast<llvm::GlobalValue>(VTable->stripPointerCasts()));
4098: 
4099:   llvm::Type *PtrDiffTy =
4100:       CGM.getTypes().ConvertType(CGM.getContext().getPointerDiffType());
4101: 
4102:   // The vtable address point is 2.
4103:   if (CGM.getLangOpts().RelativeCXXABIVTables) {
4104:     // The vtable address point is 8 bytes after its start:
4105:     // 4 for the offset to top + 4 for the relative offset to rtti.
4106:     llvm::Constant *Eight = llvm::ConstantInt::get(CGM.Int32Ty, 8);
4107:     VTable = llvm::ConstantExpr::getInBoundsPtrAdd(VTable, Eight);
4108:   } else {
4109:     llvm::Constant *Two = llvm::ConstantInt::get(PtrDiffTy, 2);
4110:     VTable = llvm::ConstantExpr::getInBoundsGetElementPtr(CGM.GlobalsInt8PtrTy,
```
- **EN**: This block uses control flow (if, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4111-4140
```cpp
4111:                                                           VTable, Two);
4112:   }
4113: 
4114:   if (const auto &Schema =
4115:           CGM.getCodeGenOpts().PointerAuth.CXXTypeInfoVTablePointer)
4116:     VTable = CGM.getConstantSignedPointer(
4117:         VTable, Schema,
4118:         Schema.isAddressDiscriminated() ? StorageAddress : nullptr,
4119:         GlobalDecl(), QualType(Ty, 0));
4120: 
4121:   Fields.push_back(VTable);
4122: }
4123: 
4124: /// Return the linkage that the type info and type info name constants
4125: /// should have for the given type.
4126: static llvm::GlobalVariable::LinkageTypes getTypeInfoLinkage(CodeGenModule &CGM,
4127:                                                              QualType Ty) {
4128:   // Itanium C++ ABI 2.9.5p7:
4129:   //   In addition, it and all of the intermediate abi::__pointer_type_info
4130:   //   structs in the chain down to the abi::__class_type_info for the
4131:   //   incomplete class type must be prevented from resolving to the
4132:   //   corresponding type_info structs for the complete class type, possibly
4133:   //   by making them local static objects. Finally, a dummy class RTTI is
4134:   //   generated for the incomplete type that will not resolve to the final
4135:   //   complete class RTTI (because the latter need not exist), possibly by
4136:   //   making it a local static object.
4137:   if (ContainsIncompleteClassType(Ty))
4138:     return llvm::GlobalValue::InternalLinkage;
4139: 
4140:   switch (Ty->getLinkage()) {
```
- **EN**: This block defines callable entry points like `getTypeInfoLinkage`; uses control flow (if, switch) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getTypeInfoLinkage`；通过控制流（if, switch）细化 ABI 降级 行为。

### Lines 4141-4170
```cpp
4141:   case Linkage::Invalid:
4142:     llvm_unreachable("Linkage hasn't been computed!");
4143: 
4144:   case Linkage::None:
4145:   case Linkage::Internal:
4146:   case Linkage::UniqueExternal:
4147:     return llvm::GlobalValue::InternalLinkage;
4148: 
4149:   case Linkage::VisibleNone:
4150:   case Linkage::Module:
4151:   case Linkage::External:
4152:     // RTTI is not enabled, which means that this type info struct is going
4153:     // to be used for exception handling. Give it linkonce_odr linkage.
4154:     if (!CGM.getLangOpts().RTTI)
4155:       return llvm::GlobalValue::LinkOnceODRLinkage;
4156: 
4157:     if (const RecordType *Record = dyn_cast<RecordType>(Ty)) {
4158:       const auto *RD =
4159:           cast<CXXRecordDecl>(Record->getDecl())->getDefinitionOrSelf();
4160:       if (RD->hasAttr<WeakAttr>())
4161:         return llvm::GlobalValue::WeakODRLinkage;
4162:       if (CGM.getTriple().isWindowsItaniumEnvironment())
4163:         if (RD->hasAttr<DLLImportAttr>() &&
4164:             ShouldUseExternalRTTIDescriptor(CGM, Ty))
4165:           return llvm::GlobalValue::ExternalLinkage;
4166:       // MinGW always uses LinkOnceODRLinkage for type info.
4167:       if (RD->isDynamicClass() &&
4168:           !CGM.getContext().getTargetInfo().getTriple().isOSCygMing())
4169:         return CGM.getVTableLinkage(RD);
4170:     }
```
- **EN**: This block uses control flow (if, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4171-4200
```cpp
4171: 
4172:     return llvm::GlobalValue::LinkOnceODRLinkage;
4173:   }
4174: 
4175:   llvm_unreachable("Invalid linkage!");
4176: }
4177: 
4178: llvm::Constant *ItaniumRTTIBuilder::BuildTypeInfo(QualType Ty) {
4179:   // We want to operate on the canonical type.
4180:   Ty = Ty.getCanonicalType();
4181: 
4182:   // Check if we've already emitted an RTTI descriptor for this type.
4183:   SmallString<256> Name;
4184:   llvm::raw_svector_ostream Out(Name);
4185:   CGM.getCXXABI().getMangleContext().mangleCXXRTTI(Ty, Out);
4186: 
4187:   llvm::GlobalVariable *OldGV = CGM.getModule().getNamedGlobal(Name);
4188:   if (OldGV && !OldGV->isDeclaration()) {
4189:     assert(!OldGV->hasAvailableExternallyLinkage() &&
4190:            "available_externally typeinfos not yet implemented");
4191: 
4192:     return OldGV;
4193:   }
4194: 
4195:   // Check if there is already an external RTTI descriptor for this type.
4196:   if (IsStandardLibraryRTTIDescriptor(Ty) ||
4197:       ShouldUseExternalRTTIDescriptor(CGM, Ty))
4198:     return GetAddrOfExternalRTTIDescriptor(Ty);
4199: 
4200:   // Emit the standard library with external linkage.
```
- **EN**: This block defines callable entry points like `Out`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Out`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4201-4230
```cpp
4201:   llvm::GlobalVariable::LinkageTypes Linkage = getTypeInfoLinkage(CGM, Ty);
4202: 
4203:   // Give the type_info object and name the formal visibility of the
4204:   // type itself.
4205:   llvm::GlobalValue::VisibilityTypes llvmVisibility;
4206:   if (llvm::GlobalValue::isLocalLinkage(Linkage))
4207:     // If the linkage is local, only default visibility makes sense.
4208:     llvmVisibility = llvm::GlobalValue::DefaultVisibility;
4209:   else if (CXXABI.classifyRTTIUniqueness(Ty, Linkage) ==
4210:            ItaniumCXXABI::RUK_NonUniqueHidden)
4211:     llvmVisibility = llvm::GlobalValue::HiddenVisibility;
4212:   else
4213:     llvmVisibility = CodeGenModule::GetLLVMVisibility(Ty->getVisibility());
4214: 
4215:   llvm::GlobalValue::DLLStorageClassTypes DLLStorageClass =
4216:       llvm::GlobalValue::DefaultStorageClass;
4217:   if (auto RD = Ty->getAsCXXRecordDecl()) {
4218:     if ((CGM.getTriple().isWindowsItaniumEnvironment() &&
4219:          RD->hasAttr<DLLExportAttr>()) ||
4220:         (CGM.shouldMapVisibilityToDLLExport(RD) &&
4221:          !llvm::GlobalValue::isLocalLinkage(Linkage) &&
4222:          llvmVisibility == llvm::GlobalValue::DefaultVisibility))
4223:       DLLStorageClass = llvm::GlobalValue::DLLExportStorageClass;
4224:   }
4225:   return BuildTypeInfo(Ty, Linkage, llvmVisibility, DLLStorageClass);
4226: }
4227: 
4228: llvm::Constant *ItaniumRTTIBuilder::BuildTypeInfo(
4229:       QualType Ty,
4230:       llvm::GlobalVariable::LinkageTypes Linkage,
```
- **EN**: This block defines callable entry points like `BuildTypeInfo`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `BuildTypeInfo`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4231-4260
```cpp
4231:       llvm::GlobalValue::VisibilityTypes Visibility,
4232:       llvm::GlobalValue::DLLStorageClassTypes DLLStorageClass) {
4233:   SmallString<256> Name;
4234:   llvm::raw_svector_ostream Out(Name);
4235:   CGM.getCXXABI().getMangleContext().mangleCXXRTTI(Ty, Out);
4236:   llvm::Module &M = CGM.getModule();
4237:   llvm::GlobalVariable *OldGV = M.getNamedGlobal(Name);
4238:   // int8 is an arbitrary type to be replaced later with replaceInitializer.
4239:   llvm::GlobalVariable *GV =
4240:       new llvm::GlobalVariable(M, CGM.Int8Ty, /*isConstant=*/true, Linkage,
4241:                                /*Initializer=*/nullptr, Name);
4242: 
4243:   // Add the vtable pointer.
4244:   BuildVTablePointer(cast<Type>(Ty), GV);
4245: 
4246:   // And the name.
4247:   llvm::GlobalVariable *TypeName = GetAddrOfTypeName(Ty, Linkage);
4248:   llvm::Constant *TypeNameField;
4249: 
4250:   // If we're supposed to demote the visibility, be sure to set a flag
4251:   // to use a string comparison for type_info comparisons.
4252:   ItaniumCXXABI::RTTIUniquenessKind RTTIUniqueness =
4253:       CXXABI.classifyRTTIUniqueness(Ty, Linkage);
4254:   if (RTTIUniqueness != ItaniumCXXABI::RUK_Unique) {
4255:     // The flag is the sign bit, which on ARM64 is defined to be clear
4256:     // for global pointers.  This is very ARM64-specific.
4257:     TypeNameField = llvm::ConstantExpr::getPtrToInt(TypeName, CGM.Int64Ty);
4258:     llvm::Constant *flag =
4259:         llvm::ConstantInt::get(CGM.Int64Ty, ((uint64_t)1) << 63);
4260:     TypeNameField = llvm::ConstantExpr::getAdd(TypeNameField, flag);
```
- **EN**: This block defines callable entry points like `Out`, `GlobalVariable`, `get`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `Out`, `GlobalVariable`, `get`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4261-4290
```cpp
4261:     TypeNameField =
4262:         llvm::ConstantExpr::getIntToPtr(TypeNameField, CGM.GlobalsInt8PtrTy);
4263:   } else {
4264:     TypeNameField = TypeName;
4265:   }
4266:   Fields.push_back(TypeNameField);
4267: 
4268:   switch (Ty->getTypeClass()) {
4269: #define TYPE(Class, Base)
4270: #define ABSTRACT_TYPE(Class, Base)
4271: #define NON_CANONICAL_UNLESS_DEPENDENT_TYPE(Class, Base) case Type::Class:
4272: #define NON_CANONICAL_TYPE(Class, Base) case Type::Class:
4273: #define DEPENDENT_TYPE(Class, Base) case Type::Class:
4274: #include "clang/AST/TypeNodes.inc"
4275:     llvm_unreachable("Non-canonical and dependent types shouldn't get here");
4276: 
4277:   // GCC treats vector types as fundamental types.
4278:   case Type::Builtin:
4279:   case Type::Vector:
4280:   case Type::ExtVector:
4281:   case Type::ConstantMatrix:
4282:   case Type::Complex:
4283:   case Type::BlockPointer:
4284:     // Itanium C++ ABI 2.9.5p4:
4285:     // abi::__fundamental_type_info adds no data members to std::type_info.
4286:     break;
4287: 
4288:   case Type::LValueReference:
4289:   case Type::RValueReference:
4290:     llvm_unreachable("References shouldn't get here");
```
- **EN**: This block imports Clang headers `clang/AST/TypeNodes.inc`; defines callable entry points like `getIntToPtr`; uses control flow (switch, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 Clang 头文件 `clang/AST/TypeNodes.inc`；定义可调用入口，例如 `getIntToPtr`；通过控制流（switch, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 4291-4320
```cpp
4291: 
4292:   case Type::Auto:
4293:   case Type::DeducedTemplateSpecialization:
4294:     llvm_unreachable("Undeduced type shouldn't get here");
4295: 
4296:   case Type::Pipe:
4297:     break;
4298: 
4299:   case Type::BitInt:
4300:     break;
4301: 
4302:   case Type::ConstantArray:
4303:   case Type::IncompleteArray:
4304:   case Type::VariableArray:
4305:   case Type::ArrayParameter:
4306:     // Itanium C++ ABI 2.9.5p5:
4307:     // abi::__array_type_info adds no data members to std::type_info.
4308:     break;
4309: 
4310:   case Type::FunctionNoProto:
4311:   case Type::FunctionProto:
4312:     // Itanium C++ ABI 2.9.5p5:
4313:     // abi::__function_type_info adds no data members to std::type_info.
4314:     break;
4315: 
4316:   case Type::Enum:
4317:     // Itanium C++ ABI 2.9.5p5:
4318:     // abi::__enum_type_info adds no data members to std::type_info.
4319:     break;
4320: 
```
- **EN**: This block uses control flow (case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4321-4350
```cpp
4321:   case Type::Record: {
4322:     const auto *RD = cast<CXXRecordDecl>(cast<RecordType>(Ty)->getDecl())
4323:                          ->getDefinitionOrSelf();
4324:     if (!RD->hasDefinition() || !RD->getNumBases()) {
4325:       // We don't need to emit any fields.
4326:       break;
4327:     }
4328: 
4329:     if (CanUseSingleInheritance(RD))
4330:       BuildSIClassTypeInfo(RD);
4331:     else
4332:       BuildVMIClassTypeInfo(RD);
4333: 
4334:     break;
4335:   }
4336: 
4337:   case Type::ObjCObject:
4338:   case Type::ObjCInterface:
4339:     BuildObjCObjectTypeInfo(cast<ObjCObjectType>(Ty));
4340:     break;
4341: 
4342:   case Type::ObjCObjectPointer:
4343:     BuildPointerTypeInfo(cast<ObjCObjectPointerType>(Ty)->getPointeeType());
4344:     break;
4345: 
4346:   case Type::Pointer:
4347:     BuildPointerTypeInfo(cast<PointerType>(Ty)->getPointeeType());
4348:     break;
4349: 
4350:   case Type::MemberPointer:
```
- **EN**: This block defines callable entry points like `BuildVMIClassTypeInfo`, `BuildObjCObjectTypeInfo`, `BuildPointerTypeInfo`; uses control flow (if, case) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `BuildVMIClassTypeInfo`, `BuildObjCObjectTypeInfo`, `BuildPointerTypeInfo`；通过控制流（if, case）细化 ABI 降级 行为。

### Lines 4351-4380
```cpp
4351:     BuildPointerToMemberTypeInfo(cast<MemberPointerType>(Ty));
4352:     break;
4353: 
4354:   case Type::Atomic:
4355:     // No fields, at least for the moment.
4356:     break;
4357: 
4358:   case Type::OverflowBehavior:
4359:     break;
4360: 
4361:   case Type::HLSLAttributedResource:
4362:   case Type::HLSLInlineSpirv:
4363:     llvm_unreachable("HLSL doesn't support RTTI");
4364:   }
4365: 
4366:   GV->replaceInitializer(llvm::ConstantStruct::getAnon(Fields));
4367: 
4368:   // Export the typeinfo in the same circumstances as the vtable is exported.
4369:   auto GVDLLStorageClass = DLLStorageClass;
4370:   if (CGM.getTarget().hasPS4DLLImportExport() &&
4371:       GVDLLStorageClass != llvm::GlobalVariable::DLLExportStorageClass) {
4372:     if (const RecordType *RecordTy = dyn_cast<RecordType>(Ty)) {
4373:       const auto *RD =
4374:           cast<CXXRecordDecl>(RecordTy->getDecl())->getDefinitionOrSelf();
4375:       if (RD->hasAttr<DLLExportAttr>() ||
4376:           CXXRecordNonInlineHasAttr<DLLExportAttr>(RD))
4377:         GVDLLStorageClass = llvm::GlobalVariable::DLLExportStorageClass;
4378:     }
4379:   }
4380: 
```
- **EN**: This block defines callable entry points like `BuildPointerToMemberTypeInfo`; uses control flow (if, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `BuildPointerToMemberTypeInfo`；通过控制流（if, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4381-4410
```cpp
4381:   // If there's already an old global variable, replace it with the new one.
4382:   if (OldGV) {
4383:     GV->takeName(OldGV);
4384:     OldGV->replaceAllUsesWith(GV);
4385:     OldGV->eraseFromParent();
4386:   }
4387: 
4388:   if (CGM.supportsCOMDAT() && GV->isWeakForLinker())
4389:     GV->setComdat(M.getOrInsertComdat(GV->getName()));
4390: 
4391:   CharUnits Align = CGM.getContext().toCharUnitsFromBits(
4392:       CGM.getTarget().getPointerAlign(CGM.GetGlobalVarAddressSpace(nullptr)));
4393:   GV->setAlignment(Align.getAsAlign());
4394: 
4395:   // The Itanium ABI specifies that type_info objects must be globally
4396:   // unique, with one exception: if the type is an incomplete class
4397:   // type or a (possibly indirect) pointer to one.  That exception
4398:   // affects the general case of comparing type_info objects produced
4399:   // by the typeid operator, which is why the comparison operators on
4400:   // std::type_info generally use the type_info name pointers instead
4401:   // of the object addresses.  However, the language's built-in uses
4402:   // of RTTI generally require class types to be complete, even when
4403:   // manipulating pointers to those class types.  This allows the
4404:   // implementation of dynamic_cast to rely on address equality tests,
4405:   // which is much faster.
4406: 
4407:   // All of this is to say that it's important that both the type_info
4408:   // object and the type_info name be uniqued when weakly emitted.
4409: 
4410:   TypeName->setVisibility(Visibility);
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 4411-4440
```cpp
4411:   CGM.setDSOLocal(TypeName);
4412: 
4413:   GV->setVisibility(Visibility);
4414:   CGM.setDSOLocal(GV);
4415: 
4416:   TypeName->setDLLStorageClass(DLLStorageClass);
4417:   GV->setDLLStorageClass(GVDLLStorageClass);
4418: 
4419:   TypeName->setPartition(CGM.getCodeGenOpts().SymbolPartition);
4420:   GV->setPartition(CGM.getCodeGenOpts().SymbolPartition);
4421: 
4422:   return GV;
4423: }
4424: 
4425: /// BuildObjCObjectTypeInfo - Build the appropriate kind of type_info
4426: /// for the given Objective-C object type.
4427: void ItaniumRTTIBuilder::BuildObjCObjectTypeInfo(const ObjCObjectType *OT) {
4428:   // Drop qualifiers.
4429:   const Type *T = OT->getBaseType().getTypePtr();
4430:   assert(isa<BuiltinType>(T) || isa<ObjCInterfaceType>(T));
4431: 
4432:   // The builtin types are abi::__class_type_infos and don't require
4433:   // extra fields.
4434:   if (isa<BuiltinType>(T)) return;
4435: 
4436:   ObjCInterfaceDecl *Class = cast<ObjCInterfaceType>(T)->getDecl();
4437:   ObjCInterfaceDecl *Super = Class->getSuperClass();
4438: 
4439:   // Root classes are also __class_type_info.
4440:   if (!Super) return;
```
- **EN**: This block defines callable entry points like `BuildObjCObjectTypeInfo`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `BuildObjCObjectTypeInfo`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4441-4470
```cpp
4441: 
4442:   QualType SuperTy = CGM.getContext().getObjCInterfaceType(Super);
4443: 
4444:   // Everything else is single inheritance.
4445:   llvm::Constant *BaseTypeInfo =
4446:       ItaniumRTTIBuilder(CXXABI).BuildTypeInfo(SuperTy);
4447:   Fields.push_back(BaseTypeInfo);
4448: }
4449: 
4450: /// BuildSIClassTypeInfo - Build an abi::__si_class_type_info, used for single
4451: /// inheritance, according to the Itanium C++ ABI, 2.95p6b.
4452: void ItaniumRTTIBuilder::BuildSIClassTypeInfo(const CXXRecordDecl *RD) {
4453:   // Itanium C++ ABI 2.9.5p6b:
4454:   // It adds to abi::__class_type_info a single member pointing to the
4455:   // type_info structure for the base type,
4456:   llvm::Constant *BaseTypeInfo =
4457:     ItaniumRTTIBuilder(CXXABI).BuildTypeInfo(RD->bases_begin()->getType());
4458:   Fields.push_back(BaseTypeInfo);
4459: }
4460: 
4461: namespace {
4462:   /// SeenBases - Contains virtual and non-virtual bases seen when traversing
4463:   /// a class hierarchy.
4464:   struct SeenBases {
4465:     llvm::SmallPtrSet<const CXXRecordDecl *, 16> NonVirtualBases;
4466:     llvm::SmallPtrSet<const CXXRecordDecl *, 16> VirtualBases;
4467:   };
4468: }
4469: 
4470: /// ComputeVMIClassTypeInfoFlags - Compute the value of the flags member in
```
- **EN**: This block introduces declarations such as `SeenBases`; defines callable entry points like `ItaniumRTTIBuilder`, `BuildSIClassTypeInfo`.
- **CN**: 该代码块给出诸如 `SeenBases` 的声明；定义可调用入口，例如 `ItaniumRTTIBuilder`, `BuildSIClassTypeInfo`。

### Lines 4471-4500
```cpp
4471: /// abi::__vmi_class_type_info.
4472: ///
4473: static unsigned ComputeVMIClassTypeInfoFlags(const CXXBaseSpecifier *Base,
4474:                                              SeenBases &Bases) {
4475: 
4476:   unsigned Flags = 0;
4477: 
4478:   auto *BaseDecl = Base->getType()->castAsCXXRecordDecl();
4479:   if (Base->isVirtual()) {
4480:     // Mark the virtual base as seen.
4481:     if (!Bases.VirtualBases.insert(BaseDecl).second) {
4482:       // If this virtual base has been seen before, then the class is diamond
4483:       // shaped.
4484:       Flags |= ItaniumRTTIBuilder::VMI_DiamondShaped;
4485:     } else {
4486:       if (Bases.NonVirtualBases.count(BaseDecl))
4487:         Flags |= ItaniumRTTIBuilder::VMI_NonDiamondRepeat;
4488:     }
4489:   } else {
4490:     // Mark the non-virtual base as seen.
4491:     if (!Bases.NonVirtualBases.insert(BaseDecl).second) {
4492:       // If this non-virtual base has been seen before, then the class has non-
4493:       // diamond shaped repeated inheritance.
4494:       Flags |= ItaniumRTTIBuilder::VMI_NonDiamondRepeat;
4495:     } else {
4496:       if (Bases.VirtualBases.count(BaseDecl))
4497:         Flags |= ItaniumRTTIBuilder::VMI_NonDiamondRepeat;
4498:     }
4499:   }
4500: 
```
- **EN**: This block defines callable entry points like `ComputeVMIClassTypeInfoFlags`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `ComputeVMIClassTypeInfoFlags`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4501-4530
```cpp
4501:   // Walk all bases.
4502:   for (const auto &I : BaseDecl->bases())
4503:     Flags |= ComputeVMIClassTypeInfoFlags(&I, Bases);
4504: 
4505:   return Flags;
4506: }
4507: 
4508: static unsigned ComputeVMIClassTypeInfoFlags(const CXXRecordDecl *RD) {
4509:   unsigned Flags = 0;
4510:   SeenBases Bases;
4511: 
4512:   // Walk all bases.
4513:   for (const auto &I : RD->bases())
4514:     Flags |= ComputeVMIClassTypeInfoFlags(&I, Bases);
4515: 
4516:   return Flags;
4517: }
4518: 
4519: /// BuildVMIClassTypeInfo - Build an abi::__vmi_class_type_info, used for
4520: /// classes with bases that do not satisfy the abi::__si_class_type_info
4521: /// constraints, according ti the Itanium C++ ABI, 2.9.5p5c.
4522: void ItaniumRTTIBuilder::BuildVMIClassTypeInfo(const CXXRecordDecl *RD) {
4523:   llvm::Type *UnsignedIntLTy =
4524:     CGM.getTypes().ConvertType(CGM.getContext().UnsignedIntTy);
4525: 
4526:   // Itanium C++ ABI 2.9.5p6c:
4527:   //   __flags is a word with flags describing details about the class
4528:   //   structure, which may be referenced by using the __flags_masks
4529:   //   enumeration. These flags refer to both direct and indirect bases.
4530:   unsigned Flags = ComputeVMIClassTypeInfoFlags(RD);
```
- **EN**: This block defines callable entry points like `ComputeVMIClassTypeInfoFlags`, `BuildVMIClassTypeInfo`; uses control flow (for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `ComputeVMIClassTypeInfoFlags`, `BuildVMIClassTypeInfo`；通过控制流（for）细化 ABI 降级 行为。

### Lines 4531-4560
```cpp
4531:   Fields.push_back(llvm::ConstantInt::get(UnsignedIntLTy, Flags));
4532: 
4533:   // Itanium C++ ABI 2.9.5p6c:
4534:   //   __base_count is a word with the number of direct proper base class
4535:   //   descriptions that follow.
4536:   Fields.push_back(llvm::ConstantInt::get(UnsignedIntLTy, RD->getNumBases()));
4537: 
4538:   if (!RD->getNumBases())
4539:     return;
4540: 
4541:   // Now add the base class descriptions.
4542: 
4543:   // Itanium C++ ABI 2.9.5p6c:
4544:   //   __base_info[] is an array of base class descriptions -- one for every
4545:   //   direct proper base. Each description is of the type:
4546:   //
4547:   //   struct abi::__base_class_type_info {
4548:   //   public:
4549:   //     const __class_type_info *__base_type;
4550:   //     long __offset_flags;
4551:   //
4552:   //     enum __offset_flags_masks {
4553:   //       __virtual_mask = 0x1,
4554:   //       __public_mask = 0x2,
4555:   //       __offset_shift = 8
4556:   //     };
4557:   //   };
4558: 
4559:   // If we're in mingw and 'long' isn't wide enough for a pointer, use 'long
4560:   // long' instead of 'long' for __offset_flags. libstdc++abi uses long long on
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 4561-4590
```cpp
4561:   // LLP64 platforms.
4562:   // FIXME: Consider updating libc++abi to match, and extend this logic to all
4563:   // LLP64 platforms.
4564:   QualType OffsetFlagsTy = CGM.getContext().LongTy;
4565:   const TargetInfo &TI = CGM.getContext().getTargetInfo();
4566:   if (TI.getTriple().isOSCygMing() &&
4567:       TI.getPointerWidth(LangAS::Default) > TI.getLongWidth())
4568:     OffsetFlagsTy = CGM.getContext().LongLongTy;
4569:   llvm::Type *OffsetFlagsLTy =
4570:       CGM.getTypes().ConvertType(OffsetFlagsTy);
4571: 
4572:   for (const auto &Base : RD->bases()) {
4573:     // The __base_type member points to the RTTI for the base type.
4574:     Fields.push_back(ItaniumRTTIBuilder(CXXABI).BuildTypeInfo(Base.getType()));
4575: 
4576:     auto *BaseDecl = Base.getType()->castAsCXXRecordDecl();
4577:     int64_t OffsetFlags = 0;
4578: 
4579:     // All but the lower 8 bits of __offset_flags are a signed offset.
4580:     // For a non-virtual base, this is the offset in the object of the base
4581:     // subobject. For a virtual base, this is the offset in the virtual table of
4582:     // the virtual base offset for the virtual base referenced (negative).
4583:     CharUnits Offset;
4584:     if (Base.isVirtual())
4585:       Offset =
4586:         CGM.getItaniumVTableContext().getVirtualBaseOffsetOffset(RD, BaseDecl);
4587:     else {
4588:       const ASTRecordLayout &Layout = CGM.getContext().getASTRecordLayout(RD);
4589:       Offset = Layout.getBaseClassOffset(BaseDecl);
4590:     };
```
- **EN**: This block uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if, for）细化 ABI 降级 行为。

### Lines 4591-4620
```cpp
4591: 
4592:     OffsetFlags = uint64_t(Offset.getQuantity()) << 8;
4593: 
4594:     // The low-order byte of __offset_flags contains flags, as given by the
4595:     // masks from the enumeration __offset_flags_masks.
4596:     if (Base.isVirtual())
4597:       OffsetFlags |= BCTI_Virtual;
4598:     if (Base.getAccessSpecifier() == AS_public)
4599:       OffsetFlags |= BCTI_Public;
4600: 
4601:     Fields.push_back(llvm::ConstantInt::getSigned(OffsetFlagsLTy, OffsetFlags));
4602:   }
4603: }
4604: 
4605: /// Compute the flags for a __pbase_type_info, and remove the corresponding
4606: /// pieces from \p Type.
4607: static unsigned extractPBaseFlags(ASTContext &Ctx, QualType &Type) {
4608:   unsigned Flags = 0;
4609: 
4610:   if (Type.isConstQualified())
4611:     Flags |= ItaniumRTTIBuilder::PTI_Const;
4612:   if (Type.isVolatileQualified())
4613:     Flags |= ItaniumRTTIBuilder::PTI_Volatile;
4614:   if (Type.isRestrictQualified())
4615:     Flags |= ItaniumRTTIBuilder::PTI_Restrict;
4616:   Type = Type.getUnqualifiedType();
4617: 
4618:   // Itanium C++ ABI 2.9.5p7:
4619:   //   When the abi::__pbase_type_info is for a direct or indirect pointer to an
4620:   //   incomplete class type, the incomplete target type flag is set.
```
- **EN**: This block defines callable entry points like `extractPBaseFlags`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `extractPBaseFlags`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4621-4650
```cpp
4621:   if (ContainsIncompleteClassType(Type))
4622:     Flags |= ItaniumRTTIBuilder::PTI_Incomplete;
4623: 
4624:   if (auto *Proto = Type->getAs<FunctionProtoType>()) {
4625:     if (Proto->isNothrow()) {
4626:       Flags |= ItaniumRTTIBuilder::PTI_Noexcept;
4627:       Type = Ctx.getFunctionTypeWithExceptionSpec(Type, EST_None);
4628:     }
4629:   }
4630: 
4631:   return Flags;
4632: }
4633: 
4634: /// BuildPointerTypeInfo - Build an abi::__pointer_type_info struct,
4635: /// used for pointer types.
4636: void ItaniumRTTIBuilder::BuildPointerTypeInfo(QualType PointeeTy) {
4637:   // Itanium C++ ABI 2.9.5p7:
4638:   //   __flags is a flag word describing the cv-qualification and other
4639:   //   attributes of the type pointed to
4640:   unsigned Flags = extractPBaseFlags(CGM.getContext(), PointeeTy);
4641: 
4642:   llvm::Type *UnsignedIntLTy =
4643:     CGM.getTypes().ConvertType(CGM.getContext().UnsignedIntTy);
4644:   Fields.push_back(llvm::ConstantInt::get(UnsignedIntLTy, Flags));
4645: 
4646:   // Itanium C++ ABI 2.9.5p7:
4647:   //  __pointee is a pointer to the std::type_info derivation for the
4648:   //  unqualified type being pointed to.
4649:   llvm::Constant *PointeeTypeInfo =
4650:       ItaniumRTTIBuilder(CXXABI).BuildTypeInfo(PointeeTy);
```
- **EN**: This block defines callable entry points like `BuildPointerTypeInfo`, `ItaniumRTTIBuilder`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `BuildPointerTypeInfo`, `ItaniumRTTIBuilder`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4651-4680
```cpp
4651:   Fields.push_back(PointeeTypeInfo);
4652: }
4653: 
4654: /// BuildPointerToMemberTypeInfo - Build an abi::__pointer_to_member_type_info
4655: /// struct, used for member pointer types.
4656: void
4657: ItaniumRTTIBuilder::BuildPointerToMemberTypeInfo(const MemberPointerType *Ty) {
4658:   QualType PointeeTy = Ty->getPointeeType();
4659: 
4660:   // Itanium C++ ABI 2.9.5p7:
4661:   //   __flags is a flag word describing the cv-qualification and other
4662:   //   attributes of the type pointed to.
4663:   unsigned Flags = extractPBaseFlags(CGM.getContext(), PointeeTy);
4664: 
4665:   const auto *RD = Ty->getMostRecentCXXRecordDecl();
4666:   if (!RD->hasDefinition())
4667:     Flags |= PTI_ContainingClassIncomplete;
4668: 
4669:   llvm::Type *UnsignedIntLTy =
4670:     CGM.getTypes().ConvertType(CGM.getContext().UnsignedIntTy);
4671:   Fields.push_back(llvm::ConstantInt::get(UnsignedIntLTy, Flags));
4672: 
4673:   // Itanium C++ ABI 2.9.5p7:
4674:   //   __pointee is a pointer to the std::type_info derivation for the
4675:   //   unqualified type being pointed to.
4676:   llvm::Constant *PointeeTypeInfo =
4677:       ItaniumRTTIBuilder(CXXABI).BuildTypeInfo(PointeeTy);
4678:   Fields.push_back(PointeeTypeInfo);
4679: 
4680:   // Itanium C++ ABI 2.9.5p9:
```
- **EN**: This block defines callable entry points like `BuildPointerToMemberTypeInfo`, `ItaniumRTTIBuilder`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `BuildPointerToMemberTypeInfo`, `ItaniumRTTIBuilder`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4681-4710
```cpp
4681:   //   __context is a pointer to an abi::__class_type_info corresponding to the
4682:   //   class type containing the member pointed to
4683:   //   (e.g., the "A" in "int A::*").
4684:   CanQualType T = CGM.getContext().getCanonicalTagType(RD);
4685:   Fields.push_back(ItaniumRTTIBuilder(CXXABI).BuildTypeInfo(T));
4686: }
4687: 
4688: llvm::Constant *ItaniumCXXABI::getAddrOfRTTIDescriptor(QualType Ty) {
4689:   return ItaniumRTTIBuilder(*this).BuildTypeInfo(Ty);
4690: }
4691: 
4692: void ItaniumCXXABI::EmitFundamentalRTTIDescriptors(const CXXRecordDecl *RD) {
4693:   // Types added here must also be added to TypeInfoIsInStandardLibrary.
4694:   QualType FundamentalTypes[] = {
4695:       getContext().VoidTy,             getContext().NullPtrTy,
4696:       getContext().BoolTy,             getContext().WCharTy,
4697:       getContext().CharTy,             getContext().UnsignedCharTy,
4698:       getContext().SignedCharTy,       getContext().ShortTy,
4699:       getContext().UnsignedShortTy,    getContext().IntTy,
4700:       getContext().UnsignedIntTy,      getContext().LongTy,
4701:       getContext().UnsignedLongTy,     getContext().LongLongTy,
4702:       getContext().UnsignedLongLongTy, getContext().Int128Ty,
4703:       getContext().UnsignedInt128Ty,   getContext().HalfTy,
4704:       getContext().FloatTy,            getContext().DoubleTy,
4705:       getContext().LongDoubleTy,       getContext().Float128Ty,
4706:       getContext().Char8Ty,            getContext().Char16Ty,
4707:       getContext().Char32Ty
4708:   };
4709:   llvm::GlobalValue::DLLStorageClassTypes DLLStorageClass =
4710:       RD->hasAttr<DLLExportAttr>() || CGM.shouldMapVisibilityToDLLExport(RD)
```
- **EN**: This block defines callable entry points like `ItaniumRTTIBuilder`, `EmitFundamentalRTTIDescriptors`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `ItaniumRTTIBuilder`, `EmitFundamentalRTTIDescriptors`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 4711-4740
```cpp
4711:           ? llvm::GlobalValue::DLLExportStorageClass
4712:           : llvm::GlobalValue::DefaultStorageClass;
4713:   llvm::GlobalValue::VisibilityTypes Visibility =
4714:       CodeGenModule::GetLLVMVisibility(RD->getVisibility());
4715:   for (const QualType &FundamentalType : FundamentalTypes) {
4716:     QualType PointerType = getContext().getPointerType(FundamentalType);
4717:     QualType PointerTypeConst = getContext().getPointerType(
4718:         FundamentalType.withConst());
4719:     for (QualType Type : {FundamentalType, PointerType, PointerTypeConst})
4720:       ItaniumRTTIBuilder(*this).BuildTypeInfo(
4721:           Type, llvm::GlobalValue::ExternalLinkage,
4722:           Visibility, DLLStorageClass);
4723:   }
4724: }
4725: 
4726: /// What sort of uniqueness rules should we use for the RTTI for the
4727: /// given type?
4728: ItaniumCXXABI::RTTIUniquenessKind ItaniumCXXABI::classifyRTTIUniqueness(
4729:     QualType CanTy, llvm::GlobalValue::LinkageTypes Linkage) const {
4730:   if (shouldRTTIBeUnique())
4731:     return RUK_Unique;
4732: 
4733:   // It's only necessary for linkonce_odr or weak_odr linkage.
4734:   if (Linkage != llvm::GlobalValue::LinkOnceODRLinkage &&
4735:       Linkage != llvm::GlobalValue::WeakODRLinkage)
4736:     return RUK_Unique;
4737: 
4738:   // It's only necessary with default visibility.
4739:   if (CanTy->getVisibility() != DefaultVisibility)
4740:     return RUK_Unique;
```
- **EN**: This block defines callable entry points like `GetLLVMVisibility`, `ItaniumRTTIBuilder`, `classifyRTTIUniqueness`; uses control flow (if, for) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `GetLLVMVisibility`, `ItaniumRTTIBuilder`, `classifyRTTIUniqueness`；通过控制流（if, for）细化 ABI 降级 行为。

### Lines 4741-4770
```cpp
4741: 
4742:   // If we're not required to publish this symbol, hide it.
4743:   if (Linkage == llvm::GlobalValue::LinkOnceODRLinkage)
4744:     return RUK_NonUniqueHidden;
4745: 
4746:   // If we're required to publish this symbol, as we might be under an
4747:   // explicit instantiation, leave it with default visibility but
4748:   // enable string-comparisons.
4749:   assert(Linkage == llvm::GlobalValue::WeakODRLinkage);
4750:   return RUK_NonUniqueVisible;
4751: }
4752: 
4753: // Find out how to codegen the complete destructor and constructor
4754: namespace {
4755: enum class StructorCodegen { Emit, RAUW, Alias, COMDAT };
4756: }
4757: static StructorCodegen getCodegenToUse(CodeGenModule &CGM,
4758:                                        const CXXMethodDecl *MD) {
4759:   if (!CGM.getCodeGenOpts().CXXCtorDtorAliases)
4760:     return StructorCodegen::Emit;
4761: 
4762:   // The complete and base structors are not equivalent if there are any virtual
4763:   // bases, so emit separate functions.
4764:   if (MD->getParent()->getNumVBases())
4765:     return StructorCodegen::Emit;
4766: 
4767:   GlobalDecl AliasDecl;
4768:   if (const auto *DD = dyn_cast<CXXDestructorDecl>(MD)) {
4769:     AliasDecl = GlobalDecl(DD, Dtor_Complete);
4770:   } else {
```
- **EN**: This block introduces declarations such as `StructorCodegen`; defines callable entry points like `getCodegenToUse`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `StructorCodegen` 的声明；定义可调用入口，例如 `getCodegenToUse`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4771-4800
```cpp
4771:     const auto *CD = cast<CXXConstructorDecl>(MD);
4772:     AliasDecl = GlobalDecl(CD, Ctor_Complete);
4773:   }
4774:   llvm::GlobalValue::LinkageTypes Linkage = CGM.getFunctionLinkage(AliasDecl);
4775: 
4776:   if (llvm::GlobalValue::isDiscardableIfUnused(Linkage))
4777:     return StructorCodegen::RAUW;
4778: 
4779:   // FIXME: Should we allow available_externally aliases?
4780:   if (!llvm::GlobalAlias::isValidLinkage(Linkage))
4781:     return StructorCodegen::RAUW;
4782: 
4783:   if (llvm::GlobalValue::isWeakForLinker(Linkage)) {
4784:     // Only ELF and wasm support COMDATs with arbitrary names (C5/D5).
4785:     if (CGM.getTarget().getTriple().isOSBinFormatELF() ||
4786:         CGM.getTarget().getTriple().isOSBinFormatWasm())
4787:       return StructorCodegen::COMDAT;
4788:     return StructorCodegen::Emit;
4789:   }
4790: 
4791:   return StructorCodegen::Alias;
4792: }
4793: 
4794: static void emitConstructorDestructorAlias(CodeGenModule &CGM,
4795:                                            GlobalDecl AliasDecl,
4796:                                            GlobalDecl TargetDecl) {
4797:   llvm::GlobalValue::LinkageTypes Linkage = CGM.getFunctionLinkage(AliasDecl);
4798: 
4799:   StringRef MangledName = CGM.getMangledName(AliasDecl);
4800:   llvm::GlobalValue *Entry = CGM.GetGlobalValue(MangledName);
```
- **EN**: This block defines callable entry points like `emitConstructorDestructorAlias`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitConstructorDestructorAlias`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4801-4830
```cpp
4801:   if (Entry && !Entry->isDeclaration())
4802:     return;
4803: 
4804:   auto *Aliasee = cast<llvm::GlobalValue>(CGM.GetAddrOfGlobal(TargetDecl));
4805: 
4806:   // Create the alias with no name.
4807:   auto *Alias = llvm::GlobalAlias::create(Linkage, "", Aliasee);
4808: 
4809:   // Constructors and destructors are always unnamed_addr.
4810:   Alias->setUnnamedAddr(llvm::GlobalValue::UnnamedAddr::Global);
4811: 
4812:   // Switch any previous uses to the alias.
4813:   if (Entry) {
4814:     assert(Entry->getType() == Aliasee->getType() &&
4815:            "declaration exists with different type");
4816:     Alias->takeName(Entry);
4817:     Entry->replaceAllUsesWith(Alias);
4818:     Entry->eraseFromParent();
4819:   } else {
4820:     Alias->setName(MangledName);
4821:   }
4822: 
4823:   // Finally, set up the alias with its proper name and attributes.
4824:   CGM.SetCommonAttributes(AliasDecl, Alias);
4825: }
4826: 
4827: void ItaniumCXXABI::emitCXXStructor(GlobalDecl GD) {
4828:   auto *MD = cast<CXXMethodDecl>(GD.getDecl());
4829:   auto *CD = dyn_cast<CXXConstructorDecl>(MD);
4830:   const CXXDestructorDecl *DD = CD ? nullptr : cast<CXXDestructorDecl>(MD);
```
- **EN**: This block defines callable entry points like `emitCXXStructor`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitCXXStructor`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 4831-4860
```cpp
4831: 
4832:   StructorCodegen CGType = getCodegenToUse(CGM, MD);
4833: 
4834:   if (CD ? GD.getCtorType() == Ctor_Complete
4835:          : GD.getDtorType() == Dtor_Complete) {
4836:     GlobalDecl BaseDecl;
4837:     if (CD)
4838:       BaseDecl = GD.getWithCtorType(Ctor_Base);
4839:     else
4840:       BaseDecl = GD.getWithDtorType(Dtor_Base);
4841: 
4842:     if (CGType == StructorCodegen::Alias || CGType == StructorCodegen::COMDAT) {
4843:       emitConstructorDestructorAlias(CGM, GD, BaseDecl);
4844:       return;
4845:     }
4846: 
4847:     if (CGType == StructorCodegen::RAUW) {
4848:       StringRef MangledName = CGM.getMangledName(GD);
4849:       auto *Aliasee = CGM.GetAddrOfGlobal(BaseDecl);
4850:       CGM.addReplacement(MangledName, Aliasee);
4851:       return;
4852:     }
4853:   }
4854: 
4855:   // The base destructor is equivalent to the base destructor of its
4856:   // base class if there is exactly one non-virtual base class with a
4857:   // non-trivial destructor, there are no fields with a non-trivial
4858:   // destructor, and the body of the destructor is trivial.
4859:   if (DD && GD.getDtorType() == Dtor_Base &&
4860:       CGType != StructorCodegen::COMDAT &&
```
- **EN**: This block defines callable entry points like `emitConstructorDestructorAlias`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitConstructorDestructorAlias`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4861-4890
```cpp
4861:       !CGM.TryEmitBaseDestructorAsAlias(DD))
4862:     return;
4863: 
4864:   // FIXME: The deleting destructor is equivalent to the selected operator
4865:   // delete if:
4866:   //  * either the delete is a destroying operator delete or the destructor
4867:   //    would be trivial if it weren't virtual,
4868:   //  * the conversion from the 'this' parameter to the first parameter of the
4869:   //    destructor is equivalent to a bitcast,
4870:   //  * the destructor does not have an implicit "this" return, and
4871:   //  * the operator delete has the same calling convention and IR function type
4872:   //    as the destructor.
4873:   // In such cases we should try to emit the deleting dtor as an alias to the
4874:   // selected 'operator delete'.
4875: 
4876:   llvm::Function *Fn = CGM.codegenCXXStructor(GD);
4877: 
4878:   if (CGType == StructorCodegen::COMDAT) {
4879:     SmallString<256> Buffer;
4880:     llvm::raw_svector_ostream Out(Buffer);
4881:     if (DD)
4882:       getMangleContext().mangleCXXDtorComdat(DD, Out);
4883:     else
4884:       getMangleContext().mangleCXXCtorComdat(CD, Out);
4885:     llvm::Comdat *C = CGM.getModule().getOrInsertComdat(Out.str());
4886:     Fn->setComdat(C);
4887:   } else {
4888:     CGM.maybeSetTrivialComdat(*MD, *Fn);
4889:   }
4890: }
```
- **EN**: This block defines callable entry points like `Out`, `getMangleContext`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `Out`, `getMangleContext`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4891-4920
```cpp
4891: 
4892: static llvm::FunctionCallee getBeginCatchFn(CodeGenModule &CGM) {
4893:   // void *__cxa_begin_catch(void*);
4894:   llvm::FunctionType *FTy = llvm::FunctionType::get(
4895:       CGM.Int8PtrTy, CGM.Int8PtrTy, /*isVarArg=*/false);
4896: 
4897:   return CGM.CreateRuntimeFunction(FTy, "__cxa_begin_catch");
4898: }
4899: 
4900: static llvm::FunctionCallee getEndCatchFn(CodeGenModule &CGM) {
4901:   // void __cxa_end_catch();
4902:   llvm::FunctionType *FTy =
4903:       llvm::FunctionType::get(CGM.VoidTy, /*isVarArg=*/false);
4904: 
4905:   return CGM.CreateRuntimeFunction(FTy, "__cxa_end_catch");
4906: }
4907: 
4908: static llvm::FunctionCallee getGetExceptionPtrFn(CodeGenModule &CGM) {
4909:   // void *__cxa_get_exception_ptr(void*);
4910:   llvm::FunctionType *FTy = llvm::FunctionType::get(
4911:       CGM.Int8PtrTy, CGM.Int8PtrTy, /*isVarArg=*/false);
4912: 
4913:   return CGM.CreateRuntimeFunction(FTy, "__cxa_get_exception_ptr");
4914: }
4915: 
4916: namespace {
4917:   /// A cleanup to call __cxa_end_catch.  In many cases, the caught
4918:   /// exception type lets us state definitively that the thrown exception
4919:   /// type does not have a destructor.  In particular:
4920:   ///   - Catch-alls tell us nothing, so we have to conservatively
```
- **EN**: This block defines callable entry points like `getBeginCatchFn`, `getEndCatchFn`, `get`, `getGetExceptionPtrFn`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `getBeginCatchFn`, `getEndCatchFn`, `get`, `getGetExceptionPtrFn`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 4921-4950
```cpp
4921:   ///     assume that the thrown exception might have a destructor.
4922:   ///   - Catches by reference behave according to their base types.
4923:   ///   - Catches of non-record types will only trigger for exceptions
4924:   ///     of non-record types, which never have destructors.
4925:   ///   - Catches of record types can trigger for arbitrary subclasses
4926:   ///     of the caught type, so we have to assume the actual thrown
4927:   ///     exception type might have a throwing destructor, even if the
4928:   ///     caught type's destructor is trivial or nothrow.
4929:   struct CallEndCatch final : EHScopeStack::Cleanup {
4930:     CallEndCatch(bool MightThrow) : MightThrow(MightThrow) {}
4931:     bool MightThrow;
4932: 
4933:     void Emit(CodeGenFunction &CGF, Flags flags) override {
4934:       if (!MightThrow) {
4935:         CGF.EmitNounwindRuntimeCall(getEndCatchFn(CGF.CGM));
4936:         return;
4937:       }
4938: 
4939:       CGF.EmitRuntimeCallOrInvoke(getEndCatchFn(CGF.CGM));
4940:     }
4941:   };
4942: }
4943: 
4944: /// Emits a call to __cxa_begin_catch and enters a cleanup to call
4945: /// __cxa_end_catch. If -fassume-nothrow-exception-dtor is specified, we assume
4946: /// that the exception object's dtor is nothrow, therefore the __cxa_end_catch
4947: /// call can be marked as nounwind even if EndMightThrow is true.
4948: ///
4949: /// \param EndMightThrow - true if __cxa_end_catch might throw
4950: static llvm::Value *CallBeginCatch(CodeGenFunction &CGF,
```
- **EN**: This block introduces declarations such as `CallEndCatch`; defines callable entry points like `CallEndCatch`, `Emit`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块给出诸如 `CallEndCatch` 的声明；定义可调用入口，例如 `CallEndCatch`, `Emit`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4951-4980
```cpp
4951:                                    llvm::Value *Exn,
4952:                                    bool EndMightThrow) {
4953:   llvm::CallInst *call =
4954:     CGF.EmitNounwindRuntimeCall(getBeginCatchFn(CGF.CGM), Exn);
4955: 
4956:   CGF.EHStack.pushCleanup<CallEndCatch>(
4957:       NormalAndEHCleanup,
4958:       EndMightThrow && !CGF.CGM.getLangOpts().AssumeNothrowExceptionDtor);
4959: 
4960:   return call;
4961: }
4962: 
4963: /// A "special initializer" callback for initializing a catch
4964: /// parameter during catch initialization.
4965: static void InitCatchParam(CodeGenFunction &CGF,
4966:                            const VarDecl &CatchParam,
4967:                            Address ParamAddr,
4968:                            SourceLocation Loc) {
4969:   // Load the exception from where the landing pad saved it.
4970:   llvm::Value *Exn = CGF.getExceptionFromSlot();
4971: 
4972:   CanQualType CatchType =
4973:     CGF.CGM.getContext().getCanonicalType(CatchParam.getType());
4974:   llvm::Type *LLVMCatchTy = CGF.ConvertTypeForMem(CatchType);
4975: 
4976:   // If we're catching by reference, we can just cast the object
4977:   // pointer to the appropriate pointer.
4978:   if (isa<ReferenceType>(CatchType)) {
4979:     QualType CaughtType = cast<ReferenceType>(CatchType)->getPointeeType();
4980:     bool EndCatchMightThrow = CaughtType->isRecordType();
```
- **EN**: This block defines callable entry points like `InitCatchParam`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `InitCatchParam`；通过控制流（if）细化 ABI 降级 行为。

### Lines 4981-5010
```cpp
4981: 
4982:     // __cxa_begin_catch returns the adjusted object pointer.
4983:     llvm::Value *AdjustedExn = CallBeginCatch(CGF, Exn, EndCatchMightThrow);
4984: 
4985:     // We have no way to tell the personality function that we're
4986:     // catching by reference, so if we're catching a pointer,
4987:     // __cxa_begin_catch will actually return that pointer by value.
4988:     if (const PointerType *PT = dyn_cast<PointerType>(CaughtType)) {
4989:       QualType PointeeType = PT->getPointeeType();
4990: 
4991:       // When catching by reference, generally we should just ignore
4992:       // this by-value pointer and use the exception object instead.
4993:       if (!PointeeType->isRecordType()) {
4994: 
4995:         // Exn points to the struct _Unwind_Exception header, which
4996:         // we have to skip past in order to reach the exception data.
4997:         unsigned HeaderSize =
4998:           CGF.CGM.getTargetCodeGenInfo().getSizeOfUnwindException();
4999:         AdjustedExn =
5000:             CGF.Builder.CreateConstGEP1_32(CGF.Int8Ty, Exn, HeaderSize);
5001: 
5002:       // However, if we're catching a pointer-to-record type that won't
5003:       // work, because the personality function might have adjusted
5004:       // the pointer.  There's actually no way for us to fully satisfy
5005:       // the language/ABI contract here:  we can't use Exn because it
5006:       // might have the wrong adjustment, but we can't use the by-value
5007:       // pointer because it's off by a level of abstraction.
5008:       //
5009:       // The current solution is to dump the adjusted pointer into an
5010:       // alloca, which breaks language semantics (because changing the
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 5011-5040
```cpp
5011:       // pointer doesn't change the exception) but at least works.
5012:       // The better solution would be to filter out non-exact matches
5013:       // and rethrow them, but this is tricky because the rethrow
5014:       // really needs to be catchable by other sites at this landing
5015:       // pad.  The best solution is to fix the personality function.
5016:       } else {
5017:         // Pull the pointer for the reference type off.
5018:         llvm::Type *PtrTy = CGF.ConvertTypeForMem(CaughtType);
5019: 
5020:         // Create the temporary and write the adjusted pointer into it.
5021:         Address ExnPtrTmp =
5022:           CGF.CreateTempAlloca(PtrTy, CGF.getPointerAlign(), "exn.byref.tmp");
5023:         llvm::Value *Casted = CGF.Builder.CreateBitCast(AdjustedExn, PtrTy);
5024:         CGF.Builder.CreateStore(Casted, ExnPtrTmp);
5025: 
5026:         // Bind the reference to the temporary.
5027:         AdjustedExn = ExnPtrTmp.emitRawPointer(CGF);
5028:       }
5029:     }
5030: 
5031:     llvm::Value *ExnCast =
5032:       CGF.Builder.CreateBitCast(AdjustedExn, LLVMCatchTy, "exn.byref");
5033:     CGF.Builder.CreateStore(ExnCast, ParamAddr);
5034:     return;
5035:   }
5036: 
5037:   // Scalars and complexes.
5038:   TypeEvaluationKind TEK = CGF.getEvaluationKind(CatchType);
5039:   if (TEK != TEK_Aggregate) {
5040:     llvm::Value *AdjustedExn = CallBeginCatch(CGF, Exn, false);
```
- **EN**: This block uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块通过控制流（if）细化 ABI 降级 行为。

### Lines 5041-5070
```cpp
5041: 
5042:     // If the catch type is a pointer type, __cxa_begin_catch returns
5043:     // the pointer by value.
5044:     if (CatchType->hasPointerRepresentation()) {
5045:       llvm::Value *CastExn =
5046:         CGF.Builder.CreateBitCast(AdjustedExn, LLVMCatchTy, "exn.casted");
5047: 
5048:       switch (CatchType.getQualifiers().getObjCLifetime()) {
5049:       case Qualifiers::OCL_Strong:
5050:         CastExn = CGF.EmitARCRetainNonBlock(CastExn);
5051:         [[fallthrough]];
5052: 
5053:       case Qualifiers::OCL_None:
5054:       case Qualifiers::OCL_ExplicitNone:
5055:       case Qualifiers::OCL_Autoreleasing:
5056:         CGF.Builder.CreateStore(CastExn, ParamAddr);
5057:         return;
5058: 
5059:       case Qualifiers::OCL_Weak:
5060:         CGF.EmitARCInitWeak(ParamAddr, CastExn);
5061:         return;
5062:       }
5063:       llvm_unreachable("bad ownership qualifier!");
5064:     }
5065: 
5066:     // Otherwise, it returns a pointer into the exception object.
5067: 
5068:     LValue srcLV = CGF.MakeNaturalAlignAddrLValue(AdjustedExn, CatchType);
5069:     LValue destLV = CGF.MakeAddrLValue(ParamAddr, CatchType);
5070:     switch (TEK) {
```
- **EN**: This block uses control flow (if, switch, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, switch, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5071-5100
```cpp
5071:     case TEK_Complex:
5072:       CGF.EmitStoreOfComplex(CGF.EmitLoadOfComplex(srcLV, Loc), destLV,
5073:                              /*init*/ true);
5074:       return;
5075:     case TEK_Scalar: {
5076:       llvm::Value *ExnLoad = CGF.EmitLoadOfScalar(srcLV, Loc);
5077:       CGF.EmitStoreOfScalar(ExnLoad, destLV, /*init*/ true);
5078:       return;
5079:     }
5080:     case TEK_Aggregate:
5081:       llvm_unreachable("evaluation kind filtered out!");
5082:     }
5083:     llvm_unreachable("bad evaluation kind");
5084:   }
5085: 
5086:   assert(isa<RecordType>(CatchType) && "unexpected catch type!");
5087:   auto catchRD = CatchType->getAsCXXRecordDecl();
5088:   CharUnits caughtExnAlignment = CGF.CGM.getClassPointerAlignment(catchRD);
5089: 
5090:   llvm::Type *PtrTy = CGF.DefaultPtrTy;
5091: 
5092:   // Check for a copy expression.  If we don't have a copy expression,
5093:   // that means a trivial copy is okay.
5094:   const Expr *copyExpr = CatchParam.getInit();
5095:   if (!copyExpr) {
5096:     llvm::Value *rawAdjustedExn = CallBeginCatch(CGF, Exn, true);
5097:     Address adjustedExn(CGF.Builder.CreateBitCast(rawAdjustedExn, PtrTy),
5098:                         LLVMCatchTy, caughtExnAlignment);
5099:     LValue Dest = CGF.MakeAddrLValue(ParamAddr, CatchType);
5100:     LValue Src = CGF.MakeAddrLValue(adjustedExn, CatchType);
```
- **EN**: This block defines callable entry points like `adjustedExn`; uses control flow (if, case) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `adjustedExn`；通过控制流（if, case）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5101-5130
```cpp
5101:     CGF.EmitAggregateCopy(Dest, Src, CatchType, AggValueSlot::DoesNotOverlap);
5102:     return;
5103:   }
5104: 
5105:   // We have to call __cxa_get_exception_ptr to get the adjusted
5106:   // pointer before copying.
5107:   llvm::CallInst *rawAdjustedExn =
5108:     CGF.EmitNounwindRuntimeCall(getGetExceptionPtrFn(CGF.CGM), Exn);
5109: 
5110:   // Cast that to the appropriate type.
5111:   Address adjustedExn(CGF.Builder.CreateBitCast(rawAdjustedExn, PtrTy),
5112:                       LLVMCatchTy, caughtExnAlignment);
5113: 
5114:   // The copy expression is defined in terms of an OpaqueValueExpr.
5115:   // Find it and map it to the adjusted expression.
5116:   CodeGenFunction::OpaqueValueMapping
5117:     opaque(CGF, OpaqueValueExpr::findInCopyConstruct(copyExpr),
5118:            CGF.MakeAddrLValue(adjustedExn, CatchParam.getType()));
5119: 
5120:   // Call the copy ctor in a terminate scope.
5121:   CGF.EHStack.pushTerminate();
5122: 
5123:   // Perform the copy construction.
5124:   CGF.EmitAggExpr(copyExpr,
5125:                   AggValueSlot::forAddr(ParamAddr, Qualifiers(),
5126:                                         AggValueSlot::IsNotDestructed,
5127:                                         AggValueSlot::DoesNotNeedGCBarriers,
5128:                                         AggValueSlot::IsNotAliased,
5129:                                         AggValueSlot::DoesNotOverlap));
5130: 
```
- **EN**: This block spells out callable entry points like `adjustedExn`, `opaque`, `forAddr`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块给出可调用入口的声明，例如 `adjustedExn`, `opaque`, `forAddr`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 5131-5160
```cpp
5131:   // Leave the terminate scope.
5132:   CGF.EHStack.popTerminate();
5133: 
5134:   // Undo the opaque value mapping.
5135:   opaque.pop();
5136: 
5137:   // Finally we can call __cxa_begin_catch.
5138:   CallBeginCatch(CGF, Exn, true);
5139: }
5140: 
5141: /// Begins a catch statement by initializing the catch variable and
5142: /// calling __cxa_begin_catch.
5143: void ItaniumCXXABI::emitBeginCatch(CodeGenFunction &CGF,
5144:                                    const CXXCatchStmt *S) {
5145:   // We have to be very careful with the ordering of cleanups here:
5146:   //   C++ [except.throw]p4:
5147:   //     The destruction [of the exception temporary] occurs
5148:   //     immediately after the destruction of the object declared in
5149:   //     the exception-declaration in the handler.
5150:   //
5151:   // So the precise ordering is:
5152:   //   1.  Construct catch variable.
5153:   //   2.  __cxa_begin_catch
5154:   //   3.  Enter __cxa_end_catch cleanup
5155:   //   4.  Enter dtor cleanup
5156:   //
5157:   // We do this by using a slightly abnormal initialization process.
5158:   // Delegation sequence:
5159:   //   - ExitCXXTryStmt opens a RunCleanupsScope
5160:   //     - EmitAutoVarAlloca creates the variable and debug info
```
- **EN**: This block defines callable entry points like `CallBeginCatch`, `emitBeginCatch`.
- **CN**: 该代码块定义可调用入口，例如 `CallBeginCatch`, `emitBeginCatch`。

### Lines 5161-5190
```cpp
5161:   //       - InitCatchParam initializes the variable from the exception
5162:   //       - CallBeginCatch calls __cxa_begin_catch
5163:   //       - CallBeginCatch enters the __cxa_end_catch cleanup
5164:   //     - EmitAutoVarCleanups enters the variable destructor cleanup
5165:   //   - EmitCXXTryStmt emits the code for the catch body
5166:   //   - EmitCXXTryStmt close the RunCleanupsScope
5167: 
5168:   VarDecl *CatchParam = S->getExceptionDecl();
5169:   if (!CatchParam) {
5170:     llvm::Value *Exn = CGF.getExceptionFromSlot();
5171:     CallBeginCatch(CGF, Exn, true);
5172:     return;
5173:   }
5174: 
5175:   // Emit the local.
5176:   CodeGenFunction::AutoVarEmission var = CGF.EmitAutoVarAlloca(*CatchParam);
5177:   {
5178:     ApplyAtomGroup Grp(CGF.getDebugInfo());
5179:     InitCatchParam(CGF, *CatchParam, var.getObjectAddress(CGF),
5180:                    S->getBeginLoc());
5181:   }
5182:   CGF.EmitAutoVarCleanups(var);
5183: }
5184: 
5185: /// Get or define the following function:
5186: ///   void @__clang_call_terminate(i8* %exn) nounwind noreturn
5187: /// This code is used only in C++.
5188: static llvm::FunctionCallee getClangCallTerminateFn(CodeGenModule &CGM) {
5189:   ASTContext &C = CGM.getContext();
5190:   const CGFunctionInfo &FI = CGM.getTypes().arrangeBuiltinFunctionDeclaration(
```
- **EN**: This block defines callable entry points like `CallBeginCatch`, `Grp`, `InitCatchParam`, `getClangCallTerminateFn`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `CallBeginCatch`, `Grp`, `InitCatchParam`, `getClangCallTerminateFn`；通过控制流（if）细化 ABI 降级 行为。

### Lines 5191-5220
```cpp
5191:       C.VoidTy, {C.getPointerType(C.CharTy)});
5192:   llvm::FunctionType *fnTy = CGM.getTypes().GetFunctionType(FI);
5193:   llvm::FunctionCallee fnRef = CGM.CreateRuntimeFunction(
5194:       fnTy, "__clang_call_terminate", llvm::AttributeList(), /*Local=*/true);
5195:   llvm::Function *fn =
5196:       cast<llvm::Function>(fnRef.getCallee()->stripPointerCasts());
5197:   if (fn->empty()) {
5198:     CGM.SetLLVMFunctionAttributes(GlobalDecl(), FI, fn, /*IsThunk=*/false);
5199:     CGM.SetLLVMFunctionAttributesForDefinition(nullptr, fn);
5200:     fn->setDoesNotThrow();
5201:     fn->setDoesNotReturn();
5202: 
5203:     // What we really want is to massively penalize inlining without
5204:     // forbidding it completely.  The difference between that and
5205:     // 'noinline' is negligible.
5206:     fn->addFnAttr(llvm::Attribute::NoInline);
5207: 
5208:     // Allow this function to be shared across translation units, but
5209:     // we don't want it to turn into an exported symbol.
5210:     fn->setLinkage(llvm::Function::LinkOnceODRLinkage);
5211:     fn->setVisibility(llvm::Function::HiddenVisibility);
5212:     if (CGM.supportsCOMDAT())
5213:       fn->setComdat(CGM.getModule().getOrInsertComdat(fn->getName()));
5214: 
5215:     // Set up the function.
5216:     llvm::BasicBlock *entry =
5217:         llvm::BasicBlock::Create(CGM.getLLVMContext(), "", fn);
5218:     CGBuilderTy builder(CGM, entry);
5219: 
5220:     // Pull the exception pointer out of the parameter list.
```
- **EN**: This block defines callable entry points like `Create`, `builder`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `Create`, `builder`；通过控制流（if）细化 ABI 降级 行为。

### Lines 5221-5250
```cpp
5221:     llvm::Value *exn = &*fn->arg_begin();
5222: 
5223:     // Call __cxa_begin_catch(exn).
5224:     llvm::CallInst *catchCall = builder.CreateCall(getBeginCatchFn(CGM), exn);
5225:     catchCall->setDoesNotThrow();
5226:     catchCall->setCallingConv(CGM.getRuntimeCC());
5227: 
5228:     // Call std::terminate().
5229:     llvm::CallInst *termCall = builder.CreateCall(CGM.getTerminateFn());
5230:     termCall->setDoesNotThrow();
5231:     termCall->setDoesNotReturn();
5232:     termCall->setCallingConv(CGM.getRuntimeCC());
5233: 
5234:     // std::terminate cannot return.
5235:     builder.CreateUnreachable();
5236:   }
5237:   return fnRef;
5238: }
5239: 
5240: llvm::CallInst *
5241: ItaniumCXXABI::emitTerminateForUnexpectedException(CodeGenFunction &CGF,
5242:                                                    llvm::Value *Exn) {
5243:   // In C++, we want to call __cxa_begin_catch() before terminating.
5244:   if (Exn) {
5245:     assert(CGF.CGM.getLangOpts().CPlusPlus);
5246:     return CGF.EmitNounwindRuntimeCall(getClangCallTerminateFn(CGF.CGM), Exn);
5247:   }
5248:   return CGF.EmitNounwindRuntimeCall(CGF.CGM.getTerminateFn());
5249: }
5250: 
```
- **EN**: This block defines callable entry points like `emitTerminateForUnexpectedException`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitTerminateForUnexpectedException`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5251-5280
```cpp
5251: std::pair<llvm::Value *, const CXXRecordDecl *>
5252: ItaniumCXXABI::LoadVTablePtr(CodeGenFunction &CGF, Address This,
5253:                              const CXXRecordDecl *RD) {
5254:   return {CGF.GetVTablePtr(This, CGM.Int8PtrTy, RD), RD};
5255: }
5256: 
5257: llvm::Constant *
5258: ItaniumCXXABI::getSignedVirtualMemberFunctionPointer(const CXXMethodDecl *MD) {
5259:   const CXXMethodDecl *origMD =
5260:       cast<CXXMethodDecl>(CGM.getItaniumVTableContext()
5261:                               .findOriginalMethod(MD->getCanonicalDecl())
5262:                               .getDecl());
5263:   llvm::Constant *thunk = getOrCreateVirtualFunctionPointerThunk(origMD);
5264:   QualType funcType = CGM.getContext().getMemberPointerType(
5265:       MD->getType(), /*Qualifier=*/std::nullopt, MD->getParent());
5266:   return CGM.getMemberFunctionPointer(thunk, funcType);
5267: }
5268: 
5269: void WebAssemblyCXXABI::emitBeginCatch(CodeGenFunction &CGF,
5270:                                        const CXXCatchStmt *C) {
5271:   if (CGF.getTarget().hasFeature("exception-handling"))
5272:     CGF.EHStack.pushCleanup<CatchRetScope>(
5273:         NormalCleanup, cast<llvm::CatchPadInst>(CGF.CurrentFuncletPad));
5274:   ItaniumCXXABI::emitBeginCatch(CGF, C);
5275: }
5276: 
5277: llvm::CallInst *
5278: WebAssemblyCXXABI::emitTerminateForUnexpectedException(CodeGenFunction &CGF,
5279:                                                        llvm::Value *Exn) {
5280:   // Itanium ABI calls __clang_call_terminate(), which __cxa_begin_catch() on
```
- **EN**: This block defines callable entry points like `LoadVTablePtr`, `getSignedVirtualMemberFunctionPointer`, `emitBeginCatch`, `emitTerminateForUnexpectedException`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `LoadVTablePtr`, `getSignedVirtualMemberFunctionPointer`, `emitBeginCatch`, `emitTerminateForUnexpectedException`；通过控制流（if）细化 ABI 降级 行为。

### Lines 5281-5310
```cpp
5281:   // the violating exception to mark it handled, but it is currently hard to do
5282:   // with wasm EH instruction structure with catch/catch_all, we just call
5283:   // std::terminate and ignore the violating exception as in CGCXXABI in Wasm EH
5284:   // and call __clang_call_terminate only in Emscripten EH.
5285:   // TODO Consider code transformation that makes calling __clang_call_terminate
5286:   // in Wasm EH possible.
5287:   if (Exn && !EHPersonality::get(CGF).isWasmPersonality()) {
5288:     assert(CGF.CGM.getLangOpts().CPlusPlus);
5289:     return CGF.EmitNounwindRuntimeCall(getClangCallTerminateFn(CGF.CGM), Exn);
5290:   }
5291:   return CGCXXABI::emitTerminateForUnexpectedException(CGF, Exn);
5292: }
5293: 
5294: /// Register a global destructor as best as we know how.
5295: void XLCXXABI::registerGlobalDtor(CodeGenFunction &CGF, const VarDecl &D,
5296:                                   llvm::FunctionCallee Dtor,
5297:                                   llvm::Constant *Addr) {
5298:   if (D.getTLSKind() != VarDecl::TLS_None) {
5299:     llvm::PointerType *PtrTy = CGF.DefaultPtrTy;
5300: 
5301:     // extern "C" int __pt_atexit_np(int flags, int(*)(int,...), ...);
5302:     llvm::FunctionType *AtExitTy =
5303:         llvm::FunctionType::get(CGM.IntTy, {CGM.IntTy, PtrTy}, true);
5304: 
5305:     // Fetch the actual function.
5306:     llvm::FunctionCallee AtExit =
5307:         CGM.CreateRuntimeFunction(AtExitTy, "__pt_atexit_np");
5308: 
5309:     // Create __dtor function for the var decl.
5310:     llvm::Function *DtorStub = CGF.createTLSAtExitStub(D, Dtor, Addr, AtExit);
```
- **EN**: This block defines callable entry points like `emitTerminateForUnexpectedException`, `registerGlobalDtor`; uses control flow (if) to specialize ABI lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitTerminateForUnexpectedException`, `registerGlobalDtor`；通过控制流（if）细化 ABI 降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 5311-5340
```cpp
5311: 
5312:     // Register above __dtor with atexit().
5313:     // First param is flags and must be 0, second param is function ptr
5314:     llvm::Value *NV = llvm::Constant::getNullValue(CGM.IntTy);
5315:     CGF.EmitNounwindRuntimeCall(AtExit, {NV, DtorStub});
5316: 
5317:     // Cannot unregister TLS __dtor so done
5318:     return;
5319:   }
5320: 
5321:   // Create __dtor function for the var decl.
5322:   llvm::Function *DtorStub =
5323:       cast<llvm::Function>(CGF.createAtExitStub(D, Dtor, Addr));
5324: 
5325:   // Register above __dtor with atexit().
5326:   CGF.registerGlobalDtorWithAtExit(DtorStub);
5327: 
5328:   // Emit __finalize function to unregister __dtor and (as appropriate) call
5329:   // __dtor.
5330:   emitCXXStermFinalizer(D, DtorStub, Addr);
5331: }
5332: 
5333: void XLCXXABI::emitCXXStermFinalizer(const VarDecl &D, llvm::Function *dtorStub,
5334:                                      llvm::Constant *addr) {
5335:   llvm::FunctionType *FTy = llvm::FunctionType::get(CGM.VoidTy, false);
5336:   SmallString<256> FnName;
5337:   {
5338:     llvm::raw_svector_ostream Out(FnName);
5339:     getMangleContext().mangleDynamicStermFinalizer(&D, Out);
5340:   }
```
- **EN**: This block defines callable entry points like `emitCXXStermFinalizer`, `Out`, `getMangleContext`; returns or forwards computed values for the surrounding ABI lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `emitCXXStermFinalizer`, `Out`, `getMangleContext`；为周围的 ABI 降级 逻辑返回或转发计算结果。

### Lines 5341-5370
```cpp
5341: 
5342:   // Create the finalization action associated with a variable.
5343:   const CGFunctionInfo &FI = CGM.getTypes().arrangeNullaryFunction();
5344:   llvm::Function *StermFinalizer = CGM.CreateGlobalInitOrCleanUpFunction(
5345:       FTy, FnName.str(), FI, D.getLocation());
5346: 
5347:   CodeGenFunction CGF(CGM);
5348: 
5349:   CGF.StartFunction(GlobalDecl(), CGM.getContext().VoidTy, StermFinalizer, FI,
5350:                     FunctionArgList(), D.getLocation(),
5351:                     D.getInit()->getExprLoc());
5352: 
5353:   // The unatexit subroutine unregisters __dtor functions that were previously
5354:   // registered by the atexit subroutine. If the referenced function is found,
5355:   // the unatexit returns a value of 0, meaning that the cleanup is still
5356:   // pending (and we should call the __dtor function).
5357:   llvm::Value *V = CGF.unregisterGlobalDtorWithUnAtExit(dtorStub);
5358: 
5359:   llvm::Value *NeedsDestruct = CGF.Builder.CreateIsNull(V, "needs_destruct");
5360: 
5361:   llvm::BasicBlock *DestructCallBlock = CGF.createBasicBlock("destruct.call");
5362:   llvm::BasicBlock *EndBlock = CGF.createBasicBlock("destruct.end");
5363: 
5364:   // Check if unatexit returns a value of 0. If it does, jump to
5365:   // DestructCallBlock, otherwise jump to EndBlock directly.
5366:   CGF.Builder.CreateCondBr(NeedsDestruct, DestructCallBlock, EndBlock);
5367: 
5368:   CGF.EmitBlock(DestructCallBlock);
5369: 
5370:   // Emit the call to dtorStub.
```
- **EN**: This block spells out callable entry points like `CGF`, `FunctionArgList`.
- **CN**: 该代码块给出可调用入口的声明，例如 `CGF`, `FunctionArgList`。

### Lines 5371-5393
```cpp
5371:   llvm::CallInst *CI = CGF.Builder.CreateCall(dtorStub);
5372: 
5373:   // Make sure the call and the callee agree on calling convention.
5374:   CI->setCallingConv(dtorStub->getCallingConv());
5375: 
5376:   CGF.EmitBlock(EndBlock);
5377: 
5378:   CGF.FinishFunction();
5379: 
5380:   if (auto *IPA = D.getAttr<InitPriorityAttr>()) {
5381:     CGM.AddCXXPrioritizedStermFinalizerEntry(StermFinalizer,
5382:                                              IPA->getPriority());
5383:   } else if (isTemplateInstantiation(D.getTemplateSpecializationKind()) ||
5384:              getContext().GetGVALinkageForVariable(&D) == GVA_DiscardableODR) {
5385:     // According to C++ [basic.start.init]p2, class template static data
5386:     // members (i.e., implicitly or explicitly instantiated specializations)
5387:     // have unordered initialization. As a consequence, we can put them into
5388:     // their own llvm.global_dtors entry.
5389:     CGM.AddCXXStermFinalizerToGlobalDtor(StermFinalizer, 65535);
5390:   } else {
5391:     CGM.AddCXXStermFinalizerEntry(StermFinalizer);
5392:   }
5393: }
```
- **EN**: This block defines callable entry points like `getContext`; uses control flow (if) to specialize ABI lowering.
- **CN**: 该代码块定义可调用入口，例如 `getContext`；通过控制流（if）细化 ABI 降级 行为。

## Key Concepts / 关键概念

- **CGF**: Central symbol in this file's implementation of ABI lowering. / 是该文件实现 ABI 降级 时的核心符号。
- **CGM**: Central symbol in this file's implementation of ABI lowering. / 是该文件实现 ABI 降级 时的核心符号。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles ABI lowering state. / 充当构建辅助器，逐步组装 ABI 降级 状态。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CodeGenFunction**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **QualType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getContext**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGCXXABI.h`, `CGCleanup.h`, `CGDebugInfo.h`, `CGRecordLayout.h`, `CGVTables.h`, `CodeGenFunction.h`, `CodeGenModule.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/AST/Attr.h`, `clang/AST/Mangle.h`, `clang/AST/StmtCXX.h`, `clang/AST/Type.h`, `clang/CodeGen/ConstantInitBuilder.h`, `clang/Basic/OpenCLImageTypes.def`, `clang/Basic/OpenCLExtensionTypes.def`, `clang/Basic/AArch64ACLETypes.def`, and 8 more
- **LLVM libraries / LLVM 库**: `llvm/IR/DataLayout.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Value.h`, `llvm/Support/ConvertEBCDIC.h`, `llvm/Support/ScopedPrinter.h`
- **Other headers / 其他头文件**: `optional`
