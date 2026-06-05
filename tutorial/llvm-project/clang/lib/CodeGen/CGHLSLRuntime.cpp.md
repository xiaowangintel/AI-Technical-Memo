# CGHLSLRuntime.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGHLSLRuntime.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements the CGHLSLRuntime portion of Clang's CodeGen pipeline.
- **Purpose (CN) / 目的（中文）**: 实现 Clang CodeGen 流水线中的 CGHLSLRuntime 部分。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
 1: //===----- CGHLSLRuntime.cpp - Interface to HLSL Runtimes -----------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This provides an abstract class for HLSL code generation.  Concrete
10: // subclasses of this implement code generation for specific HLSL
11: // runtime libraries.
12: //
13: //===----------------------------------------------------------------------===//
14: 
15: #include "CGHLSLRuntime.h"
16: #include "CGDebugInfo.h"
17: #include "CGRecordLayout.h"
18: #include "CodeGenFunction.h"
19: #include "CodeGenModule.h"
20: #include "HLSLBufferLayoutBuilder.h"
```
- **EN**: This block imports local CodeGen headers `CGHLSLRuntime.h`, `CGDebugInfo.h`, `CGRecordLayout.h`, and 3 more; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGHLSLRuntime.h`, `CGDebugInfo.h`, `CGRecordLayout.h`, and 3 more；包含影响本编译单元构建方式的预处理结构。

### Lines 21-40
```cpp
21: #include "TargetInfo.h"
22: #include "clang/AST/ASTContext.h"
23: #include "clang/AST/Attr.h"
24: #include "clang/AST/Decl.h"
25: #include "clang/AST/Expr.h"
26: #include "clang/AST/HLSLResource.h"
27: #include "clang/AST/RecursiveASTVisitor.h"
28: #include "clang/AST/Type.h"
29: #include "clang/Basic/DiagnosticFrontend.h"
30: #include "clang/Basic/TargetOptions.h"
31: #include "llvm/ADT/DenseMap.h"
32: #include "llvm/ADT/ScopeExit.h"
33: #include "llvm/ADT/SmallString.h"
34: #include "llvm/ADT/SmallVector.h"
35: #include "llvm/Frontend/HLSL/RootSignatureMetadata.h"
36: #include "llvm/IR/Constants.h"
37: #include "llvm/IR/DerivedTypes.h"
38: #include "llvm/IR/GlobalVariable.h"
39: #include "llvm/IR/IntrinsicInst.h"
40: #include "llvm/IR/LLVMContext.h"
```
- **EN**: This block imports local CodeGen headers `TargetInfo.h`; Clang headers `clang/AST/ASTContext.h`, `clang/AST/Attr.h`, `clang/AST/Decl.h`, and 6 more; LLVM headers `llvm/ADT/DenseMap.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallString.h`, and 7 more; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `TargetInfo.h`；Clang 头文件 `clang/AST/ASTContext.h`, `clang/AST/Attr.h`, `clang/AST/Decl.h`, and 6 more；LLVM 头文件 `llvm/ADT/DenseMap.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallString.h`, and 7 more；包含影响本编译单元构建方式的预处理结构。

### Lines 41-60
```cpp
41: #include "llvm/IR/Metadata.h"
42: #include "llvm/IR/Module.h"
43: #include "llvm/IR/Type.h"
44: #include "llvm/IR/Value.h"
45: #include "llvm/Support/Alignment.h"
46: #include "llvm/Support/ErrorHandling.h"
47: #include "llvm/Support/FormatVariadic.h"
48: #include <cstdint>
49: #include <optional>
50: 
51: using namespace clang;
52: using namespace CodeGen;
53: using namespace clang::hlsl;
54: using namespace llvm;
55: 
56: using llvm::hlsl::CBufferRowSizeInBytes;
57: 
58: namespace {
59: 
60: void addDxilValVersion(StringRef ValVersionStr, llvm::Module &M) {
```
- **EN**: This block imports LLVM headers `llvm/IR/Metadata.h`, `llvm/IR/Module.h`, `llvm/IR/Type.h`, and 4 more; other headers `cstdint`, `optional`; opens or references namespaces `clang`, `CodeGen`, `llvm`; defines callable entry points like `addDxilValVersion`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/IR/Metadata.h`, `llvm/IR/Module.h`, `llvm/IR/Type.h`, and 4 more；其他头文件 `cstdint`, `optional`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；定义可调用入口，例如 `addDxilValVersion`；包含影响本编译单元构建方式的预处理结构。

### Lines 61-80
```cpp
61:   // The validation of ValVersionStr is done at HLSLToolChain::TranslateArgs.
62:   // Assume ValVersionStr is legal here.
63:   VersionTuple Version;
64:   if (Version.tryParse(ValVersionStr) || Version.getBuild() ||
65:       Version.getSubminor() || !Version.getMinor()) {
66:     return;
67:   }
68: 
69:   uint64_t Major = Version.getMajor();
70:   uint64_t Minor = *Version.getMinor();
71: 
72:   auto &Ctx = M.getContext();
73:   IRBuilder<> B(M.getContext());
74:   MDNode *Val = MDNode::get(Ctx, {ConstantAsMetadata::get(B.getInt32(Major)),
75:                                   ConstantAsMetadata::get(B.getInt32(Minor))});
76:   StringRef DXILValKey = "dx.valver";
77:   auto *DXILValMD = M.getOrInsertNamedMetadata(DXILValKey);
78:   DXILValMD->addOperand(Val);
79: }
80: 
```
- **EN**: This block defines callable entry points like `B`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `B`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 81-100
```cpp
 81: void addRootSignatureMD(llvm::dxbc::RootSignatureVersion RootSigVer,
 82:                         ArrayRef<llvm::hlsl::rootsig::RootElement> Elements,
 83:                         llvm::Function *Fn, llvm::Module &M) {
 84:   auto &Ctx = M.getContext();
 85: 
 86:   llvm::hlsl::rootsig::MetadataBuilder RSBuilder(Ctx, Elements);
 87:   MDNode *RootSignature = RSBuilder.BuildRootSignature();
 88: 
 89:   ConstantAsMetadata *Version = ConstantAsMetadata::get(ConstantInt::get(
 90:       llvm::Type::getInt32Ty(Ctx), llvm::to_underlying(RootSigVer)));
 91:   ValueAsMetadata *EntryFunc = Fn ? ValueAsMetadata::get(Fn) : nullptr;
 92:   MDNode *MDVals = MDNode::get(Ctx, {EntryFunc, RootSignature, Version});
 93: 
 94:   StringRef RootSignatureValKey = "dx.rootsignatures";
 95:   auto *RootSignatureValMD = M.getOrInsertNamedMetadata(RootSignatureValKey);
 96:   RootSignatureValMD->addOperand(MDVals);
 97: }
 98: 
 99: // Given a MemberExpr of a resource or resource array type, find the parent
100: // VarDecl of the struct or class instance that contains this resource and
```
- **EN**: This block defines callable entry points like `addRootSignatureMD`, `RSBuilder`, `getInt32Ty`.
- **CN**: 该代码块定义可调用入口，例如 `addRootSignatureMD`, `RSBuilder`, `getInt32Ty`。

### Lines 101-120
```cpp
101: // build the full resource name based on the member access path.
102: //
103: // For example, for a member access like "myStructArray[0].memberA",
104: // this function will find the VarDecl of "myStructArray" and use the
105: // EmbeddedResourceNameBuilder to build the resource name
106: // "myStructArray.0.memberA".
107: static const VarDecl *findStructResourceParentDeclAndBuildName(
108:     const MemberExpr *ME, EmbeddedResourceNameBuilder &NameBuilder) {
109: 
110:   SmallVector<const Expr *> WorkList;
111:   const VarDecl *VD = nullptr;
112:   const Expr *E = ME;
113: 
114:   for (;;) {
115:     if (const auto *DRE = dyn_cast<DeclRefExpr>(E)) {
116:       assert(isa<VarDecl>(DRE->getDecl()) &&
117:              "member expr base is not a var decl");
118:       VD = cast<VarDecl>(DRE->getDecl());
119:       NameBuilder.pushName(VD->getName());
120:       break;
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 121-140
```cpp
121:     }
122: 
123:     WorkList.push_back(E);
124:     if (const auto *MExp = dyn_cast<MemberExpr>(E))
125:       E = MExp->getBase();
126:     else if (const auto *ICE = dyn_cast<ImplicitCastExpr>(E))
127:       E = ICE->getSubExpr();
128:     else if (const auto *ASE = dyn_cast<ArraySubscriptExpr>(E))
129:       E = ASE->getBase();
130:     else if (isa<CXXThisExpr>(E))
131:       // Resource member access on "this" pointer not yet implemented
132:       // (llvm/llvm-project#190299)
133:       return nullptr;
134:     else
135:       llvm_unreachable("unexpected expr type in resource member access");
136: 
137:     assert(E && "expected valid expression");
138:   }
139: 
140:   while (!WorkList.empty()) {
```
- **EN**: This block uses control flow (if, while) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, while）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 141-160
```cpp
141:     E = WorkList.pop_back_val();
142:     if (const auto *ME = dyn_cast<MemberExpr>(E)) {
143:       NameBuilder.pushName(
144:           ME->getMemberNameInfo().getName().getAsIdentifierInfo()->getName());
145:     } else if (const auto *ICE = dyn_cast<ImplicitCastExpr>(E)) {
146:       if (ICE->getCastKind() == CK_UncheckedDerivedToBase) {
147:         CXXRecordDecl *DerivedRD =
148:             ICE->getSubExpr()->getType()->getAsCXXRecordDecl();
149:         CXXRecordDecl *BaseRD = ICE->getType()->getAsCXXRecordDecl();
150:         NameBuilder.pushBaseNameHierarchy(DerivedRD, BaseRD);
151:       }
152:     } else if (const auto *ASE = dyn_cast<ArraySubscriptExpr>(E)) {
153:       const Expr *IdxExpr = ASE->getIdx();
154:       std::optional<llvm::APSInt> Value =
155:           IdxExpr->getIntegerConstantExpr(VD->getASTContext());
156:       assert(Value &&
157:              "expected constant index in struct with resource array access");
158:       NameBuilder.pushArrayIndex(Value->getZExtValue());
159:     } else {
160:       llvm_unreachable("unexpected expr type in resource member access");
```
- **EN**: This block introduces declarations such as `with`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `with` 的声明；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 161-180
```cpp
161:     }
162:   }
163:   return VD;
164: }
165: 
166: // Given a MemberExpr of a resource or resource array type, find the
167: // corresponding global resource declaration associated with the owning struct
168: // or class instance via HLSLAssociatedResourceDeclAttr.
169: static const VarDecl *
170: findAssociatedResourceDeclForStruct(ASTContext &AST, const MemberExpr *ME) {
171: 
172:   EmbeddedResourceNameBuilder NameBuilder;
173:   const VarDecl *ParentVD =
174:       findStructResourceParentDeclAndBuildName(ME, NameBuilder);
175:   if (!ParentVD)
176:     return nullptr;
177: 
178:   if (!ParentVD->hasGlobalStorage())
179:     return nullptr;
180: 
```
- **EN**: This block defines callable entry points like `findAssociatedResourceDeclForStruct`, `findStructResourceParentDeclAndBuildName`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `findAssociatedResourceDeclForStruct`, `findStructResourceParentDeclAndBuildName`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 181-200
```cpp
181:   IdentifierInfo *II = NameBuilder.getNameAsIdentifier(AST);
182:   for (const Attr *A : ParentVD->getAttrs()) {
183:     if (const auto *ADA = dyn_cast<HLSLAssociatedResourceDeclAttr>(A)) {
184:       VarDecl *AssocResVD = ADA->getResDecl();
185:       if (AssocResVD->getIdentifier() == II)
186:         return AssocResVD;
187:     }
188:   }
189:   return nullptr;
190: }
191: 
192: // Find array variable declaration from DeclRef expression
193: static const ValueDecl *getArrayDecl(ASTContext &AST, const Expr *E) {
194:   E = E->IgnoreImpCasts();
195:   if (const auto *DRE = dyn_cast_or_null<DeclRefExpr>(E))
196:     return DRE->getDecl();
197:   if (isa<MemberExpr>(E))
198:     return findAssociatedResourceDeclForStruct(AST, cast<MemberExpr>(E));
199:   return nullptr;
200: }
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 201-220
```cpp
201: 
202: // Find array variable declaration from nested array subscript AST nodes
203: static const ValueDecl *getArrayDecl(ASTContext &AST,
204:                                      const ArraySubscriptExpr *ASE) {
205:   const Expr *E = nullptr;
206:   while (ASE != nullptr) {
207:     E = ASE->getBase()->IgnoreImpCasts();
208:     if (!E)
209:       return nullptr;
210:     ASE = dyn_cast<ArraySubscriptExpr>(E);
211:   }
212:   return getArrayDecl(AST, E);
213: }
214: 
215: // Get the total size of the array, or 0 if the array is unbounded.
216: static int getTotalArraySize(ASTContext &AST, const clang::Type *Ty) {
217:   Ty = Ty->getUnqualifiedDesugaredType();
218:   assert(Ty->isArrayType() && "expected array type");
219:   if (Ty->isIncompleteArrayType())
220:     return 0;
```
- **EN**: This block defines callable entry points like `getArrayDecl`, `getTotalArraySize`; uses control flow (if, while) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getArrayDecl`, `getTotalArraySize`；通过控制流（if, while）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 221-240
```cpp
221:   return AST.getConstantArrayElementCount(cast<ConstantArrayType>(Ty));
222: }
223: 
224: static Value *buildNameForResource(llvm::StringRef BaseName,
225:                                    CodeGenModule &CGM) {
226:   llvm::SmallString<64> GlobalName = {BaseName, ".str"};
227:   return CGM.GetAddrOfConstantCString(BaseName.str(), GlobalName.c_str())
228:       .getPointer();
229: }
230: 
231: static CXXMethodDecl *lookupMethod(CXXRecordDecl *Record, StringRef Name,
232:                                    StorageClass SC = SC_None) {
233:   for (auto *Method : Record->methods()) {
234:     if (Method->getStorageClass() == SC && Method->getName() == Name)
235:       return Method;
236:   }
237:   return nullptr;
238: }
239: 
240: static CXXMethodDecl *lookupResourceInitMethodAndSetupArgs(
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 241-260
```cpp
241:     CodeGenModule &CGM, CXXRecordDecl *ResourceDecl, llvm::Value *Range,
242:     llvm::Value *Index, StringRef Name, ResourceBindingAttrs &Binding,
243:     CallArgList &Args) {
244:   assert(Binding.hasBinding() && "at least one binding attribute expected");
245: 
246:   ASTContext &AST = CGM.getContext();
247:   CXXMethodDecl *CreateMethod = nullptr;
248:   Value *NameStr = buildNameForResource(Name, CGM);
249:   Value *Space = llvm::ConstantInt::get(CGM.IntTy, Binding.getSpace());
250: 
251:   bool HasCounter = hasCounterHandle(ResourceDecl);
252:   assert((!HasCounter || Binding.hasCounterImplicitOrderID()) &&
253:          "resources with counter handle must have a binding with counter "
254:          "implicit order ID");
255:   if (Binding.isExplicit()) {
256:     // explicit binding
257:     auto *RegSlot = llvm::ConstantInt::get(CGM.IntTy, Binding.getSlot());
258:     Args.add(RValue::get(RegSlot), AST.UnsignedIntTy);
259:     const char *Name = Binding.hasCounterImplicitOrderID()
260:                            ? "__createFromBindingWithImplicitCounter"
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 261-280
```cpp
261:                            : "__createFromBinding";
262:     CreateMethod = lookupMethod(ResourceDecl, Name, SC_Static);
263:   } else {
264:     // implicit binding
265:     auto *OrderID =
266:         llvm::ConstantInt::get(CGM.IntTy, Binding.getImplicitOrderID());
267:     Args.add(RValue::get(OrderID), AST.UnsignedIntTy);
268:     const char *Name = Binding.hasCounterImplicitOrderID()
269:                            ? "__createFromImplicitBindingWithImplicitCounter"
270:                            : "__createFromImplicitBinding";
271:     CreateMethod = lookupMethod(ResourceDecl, Name, SC_Static);
272:   }
273:   Args.add(RValue::get(Space), AST.UnsignedIntTy);
274:   Args.add(RValue::get(Range), AST.IntTy);
275:   Args.add(RValue::get(Index), AST.UnsignedIntTy);
276:   Args.add(RValue::get(NameStr), AST.getPointerType(AST.CharTy.withConst()));
277:   if (HasCounter) {
278:     uint32_t CounterBinding = Binding.getCounterImplicitOrderID();
279:     auto *CounterOrderID = llvm::ConstantInt::get(CGM.IntTy, CounterBinding);
280:     Args.add(RValue::get(CounterOrderID), AST.UnsignedIntTy);
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 281-300
```cpp
281:   }
282: 
283:   return CreateMethod;
284: }
285: 
286: static void callResourceInitMethod(CodeGenFunction &CGF,
287:                                    CXXMethodDecl *CreateMethod,
288:                                    CallArgList &Args, Address ReturnAddress) {
289:   llvm::Constant *CalleeFn = CGF.CGM.GetAddrOfFunction(CreateMethod);
290:   const FunctionProtoType *Proto =
291:       CreateMethod->getType()->getAs<FunctionProtoType>();
292:   const CGFunctionInfo &FnInfo =
293:       CGF.CGM.getTypes().arrangeFreeFunctionCall(Args, Proto, false);
294:   ReturnValueSlot ReturnValue(ReturnAddress, false);
295:   CGCallee Callee(CGCalleeInfo(Proto), CalleeFn);
296:   CGF.EmitCall(FnInfo, Callee, ReturnValue, Args, nullptr);
297: }
298: 
299: // Initializes local resource array variable. For multi-dimensional arrays it
300: // calls itself recursively to initialize its sub-arrays. The Index used in the
```
- **EN**: This block defines callable entry points like `callResourceInitMethod`, `ReturnValue`, `Callee`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `callResourceInitMethod`, `ReturnValue`, `Callee`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 301-320
```cpp
301: // resource constructor calls will begin at StartIndex and will be incremented
302: // for each array element. The last used resource Index is returned to the
303: // caller. If the function returns std::nullopt, it indicates an error.
304: static std::optional<llvm::Value *> initializeLocalResourceArray(
305:     CodeGenFunction &CGF, CXXRecordDecl *ResourceDecl,
306:     const ConstantArrayType *ArrayTy, AggValueSlot &ValueSlot,
307:     llvm::Value *Range, llvm::Value *StartIndex, StringRef ResourceName,
308:     ResourceBindingAttrs &Binding, ArrayRef<llvm::Value *> PrevGEPIndices,
309:     SourceLocation ArraySubsExprLoc) {
310: 
311:   ASTContext &AST = CGF.getContext();
312:   llvm::IntegerType *IntTy = CGF.CGM.IntTy;
313:   llvm::Value *Index = StartIndex;
314:   llvm::Value *One = llvm::ConstantInt::get(IntTy, 1);
315:   const uint64_t ArraySize = ArrayTy->getSExtSize();
316:   QualType ElemType = ArrayTy->getElementType();
317:   Address TmpArrayAddr = ValueSlot.getAddress();
318: 
319:   // Add additional index to the getelementptr call indices.
320:   // This index will be updated for each array element in the loops below.
```
- **EN**: This block defines callable entry points like `initializeLocalResourceArray`.
- **CN**: 该代码块定义可调用入口，例如 `initializeLocalResourceArray`。

### Lines 321-340
```cpp
321:   SmallVector<llvm::Value *> GEPIndices(PrevGEPIndices);
322:   GEPIndices.push_back(llvm::ConstantInt::get(IntTy, 0));
323: 
324:   // For array of arrays, recursively initialize the sub-arrays.
325:   if (ElemType->isArrayType()) {
326:     const ConstantArrayType *SubArrayTy = cast<ConstantArrayType>(ElemType);
327:     for (uint64_t I = 0; I < ArraySize; I++) {
328:       if (I > 0) {
329:         Index = CGF.Builder.CreateAdd(Index, One);
330:         GEPIndices.back() = llvm::ConstantInt::get(IntTy, I);
331:       }
332:       std::optional<llvm::Value *> MaybeIndex = initializeLocalResourceArray(
333:           CGF, ResourceDecl, SubArrayTy, ValueSlot, Range, Index, ResourceName,
334:           Binding, GEPIndices, ArraySubsExprLoc);
335:       if (!MaybeIndex)
336:         return std::nullopt;
337:       Index = *MaybeIndex;
338:     }
339:     return Index;
340:   }
```
- **EN**: This block defines callable entry points like `GEPIndices`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `GEPIndices`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 341-360
```cpp
341: 
342:   // For array of resources, initialize each resource in the array.
343:   llvm::Type *Ty = CGF.ConvertTypeForMem(ElemType);
344:   CharUnits ElemSize = AST.getTypeSizeInChars(ElemType);
345:   CharUnits Align =
346:       TmpArrayAddr.getAlignment().alignmentOfArrayElement(ElemSize);
347: 
348:   for (uint64_t I = 0; I < ArraySize; I++) {
349:     if (I > 0) {
350:       Index = CGF.Builder.CreateAdd(Index, One);
351:       GEPIndices.back() = llvm::ConstantInt::get(IntTy, I);
352:     }
353:     Address ReturnAddress =
354:         CGF.Builder.CreateGEP(TmpArrayAddr, GEPIndices, Ty, Align);
355: 
356:     CallArgList Args;
357:     CXXMethodDecl *CreateMethod = lookupResourceInitMethodAndSetupArgs(
358:         CGF.CGM, ResourceDecl, Range, Index, ResourceName, Binding, Args);
359: 
360:     if (!CreateMethod)
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 361-380
```cpp
361:       // This can happen if someone creates an array of structs that looks like
362:       // an HLSL resource record array but it does not have the required static
363:       // create method. No binding will be generated for it.
364:       return std::nullopt;
365: 
366:     callResourceInitMethod(CGF, CreateMethod, Args, ReturnAddress);
367:   }
368:   return Index;
369: }
370: 
371: } // namespace
372: 
373: llvm::Type *
374: CGHLSLRuntime::convertHLSLSpecificType(const Type *T,
375:                                        const CGHLSLOffsetInfo &OffsetInfo) {
376:   assert(T->isHLSLSpecificType() && "Not an HLSL specific type!");
377: 
378:   // Check if the target has a specific translation for this type first.
379:   if (llvm::Type *TargetTy =
380:           CGM.getTargetCodeGenInfo().getHLSLType(CGM, T, OffsetInfo))
```
- **EN**: This block opens or references namespaces `llvm`; defines callable entry points like `callResourceInitMethod`, `convertHLSLSpecificType`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块打开或引用命名空间 `llvm`；定义可调用入口，例如 `callResourceInitMethod`, `convertHLSLSpecificType`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 381-400
```cpp
381:     return TargetTy;
382: 
383:   llvm_unreachable("Generic handling of HLSL types is not supported.");
384: }
385: 
386: llvm::Triple::ArchType CGHLSLRuntime::getArch() {
387:   return CGM.getTarget().getTriple().getArch();
388: }
389: 
390: // Emits constant global variables for buffer constants declarations
391: // and creates metadata linking the constant globals with the buffer global.
392: void CGHLSLRuntime::emitBufferGlobalsAndMetadata(
393:     const HLSLBufferDecl *BufDecl, llvm::GlobalVariable *BufGV,
394:     const CGHLSLOffsetInfo &OffsetInfo) {
395:   LLVMContext &Ctx = CGM.getLLVMContext();
396: 
397:   // get the layout struct from constant buffer target type
398:   llvm::Type *BufType = BufGV->getValueType();
399:   llvm::StructType *LayoutStruct = cast<llvm::StructType>(
400:       cast<llvm::TargetExtType>(BufType)->getTypeParameter(0));
```
- **EN**: This block defines callable entry points like `getArch`, `emitBufferGlobalsAndMetadata`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getArch`, `emitBufferGlobalsAndMetadata`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 401-420
```cpp
401: 
402:   SmallVector<std::pair<VarDecl *, uint32_t>> DeclsWithOffset;
403:   size_t OffsetIdx = 0;
404:   for (Decl *D : BufDecl->buffer_decls()) {
405:     if (isa<CXXRecordDecl, EmptyDecl>(D))
406:       // Nothing to do for this declaration.
407:       continue;
408:     if (isa<FunctionDecl>(D)) {
409:       // A function within an cbuffer is effectively a top-level function.
410:       CGM.EmitTopLevelDecl(D);
411:       continue;
412:     }
413:     VarDecl *VD = dyn_cast<VarDecl>(D);
414:     if (!VD)
415:       continue;
416: 
417:     QualType VDTy = VD->getType();
418:     if (VDTy.getAddressSpace() != LangAS::hlsl_constant) {
419:       if (VD->getStorageClass() == SC_Static ||
420:           VDTy.getAddressSpace() == LangAS::hlsl_groupshared ||
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 421-440
```cpp
421:           VDTy->isHLSLResourceRecord() || VDTy->isHLSLResourceRecordArray()) {
422:         // Emit static and groupshared variables and resource classes inside
423:         // cbuffer as regular globals
424:         CGM.EmitGlobal(VD);
425:       }
426:       continue;
427:     }
428: 
429:     DeclsWithOffset.emplace_back(VD, OffsetInfo[OffsetIdx++]);
430:   }
431: 
432:   if (!OffsetInfo.empty())
433:     llvm::stable_sort(DeclsWithOffset, [](const auto &LHS, const auto &RHS) {
434:       return CGHLSLOffsetInfo::compareOffsets(LHS.second, RHS.second);
435:     });
436: 
437:   // Associate the buffer global variable with its constants
438:   SmallVector<llvm::Metadata *> BufGlobals;
439:   BufGlobals.reserve(DeclsWithOffset.size() + 1);
440:   BufGlobals.push_back(ValueAsMetadata::get(BufGV));
```
- **EN**: This block defines callable entry points like `compareOffsets`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `compareOffsets`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 441-460
```cpp
441: 
442:   auto ElemIt = LayoutStruct->element_begin();
443:   for (auto &[VD, _] : DeclsWithOffset) {
444:     if (CGM.getTargetCodeGenInfo().isHLSLPadding(*ElemIt))
445:       ++ElemIt;
446: 
447:     assert(ElemIt != LayoutStruct->element_end() &&
448:            "number of elements in layout struct does not match");
449:     llvm::Type *LayoutType = *ElemIt++;
450: 
451:     GlobalVariable *ElemGV =
452:         cast<GlobalVariable>(CGM.GetAddrOfGlobalVar(VD, LayoutType));
453:     BufGlobals.push_back(ValueAsMetadata::get(ElemGV));
454:   }
455:   assert(ElemIt == LayoutStruct->element_end() &&
456:          "number of elements in layout struct does not match");
457: 
458:   // add buffer metadata to the module
459:   CGM.getModule()
460:       .getOrInsertNamedMetadata("hlsl.cbs")
```
- **EN**: This block introduces declarations such as `does`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `does` 的声明；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 461-480
```cpp
461:       ->addOperand(MDNode::get(Ctx, BufGlobals));
462: }
463: 
464: // Creates resource handle type for the HLSL buffer declaration
465: static const clang::HLSLAttributedResourceType *
466: createBufferHandleType(const HLSLBufferDecl *BufDecl) {
467:   ASTContext &AST = BufDecl->getASTContext();
468:   QualType QT = AST.getHLSLAttributedResourceType(
469:       AST.HLSLResourceTy, AST.getCanonicalTagType(BufDecl->getLayoutStruct()),
470:       HLSLAttributedResourceType::Attributes(ResourceClass::CBuffer));
471:   return cast<HLSLAttributedResourceType>(QT.getTypePtr());
472: }
473: 
474: CGHLSLOffsetInfo CGHLSLOffsetInfo::fromDecl(const HLSLBufferDecl &BufDecl) {
475:   CGHLSLOffsetInfo Result;
476: 
477:   // If we don't have packoffset info, just return an empty result.
478:   if (!BufDecl.hasValidPackoffset())
479:     return Result;
480: 
```
- **EN**: This block defines callable entry points like `createBufferHandleType`, `Attributes`, `fromDecl`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `createBufferHandleType`, `Attributes`, `fromDecl`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 481-500
```cpp
481:   for (Decl *D : BufDecl.buffer_decls()) {
482:     if (isa<CXXRecordDecl, EmptyDecl>(D) || isa<FunctionDecl>(D)) {
483:       continue;
484:     }
485:     VarDecl *VD = dyn_cast<VarDecl>(D);
486:     if (!VD || VD->getType().getAddressSpace() != LangAS::hlsl_constant)
487:       continue;
488: 
489:     if (!VD->hasAttrs()) {
490:       Result.Offsets.push_back(Unspecified);
491:       continue;
492:     }
493: 
494:     uint32_t Offset = Unspecified;
495:     for (auto *Attr : VD->getAttrs()) {
496:       if (auto *POA = dyn_cast<HLSLPackOffsetAttr>(Attr)) {
497:         Offset = POA->getOffsetInBytes();
498:         break;
499:       }
500:       auto *RBA = dyn_cast<HLSLResourceBindingAttr>(Attr);
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 501-520
```cpp
501:       if (RBA &&
502:           RBA->getRegisterType() == HLSLResourceBindingAttr::RegisterType::C) {
503:         Offset = RBA->getSlotNumber() * CBufferRowSizeInBytes;
504:         break;
505:       }
506:     }
507:     Result.Offsets.push_back(Offset);
508:   }
509:   return Result;
510: }
511: 
512: // Codegen for HLSLBufferDecl
513: void CGHLSLRuntime::addBuffer(const HLSLBufferDecl *BufDecl) {
514: 
515:   assert(BufDecl->isCBuffer() && "tbuffer codegen is not supported yet");
516: 
517:   // create resource handle type for the buffer
518:   const clang::HLSLAttributedResourceType *ResHandleTy =
519:       createBufferHandleType(BufDecl);
520: 
```
- **EN**: This block defines callable entry points like `addBuffer`, `createBufferHandleType`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `addBuffer`, `createBufferHandleType`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 521-540
```cpp
521:   // empty constant buffer is ignored
522:   if (ResHandleTy->getContainedType()->getAsCXXRecordDecl()->isEmpty())
523:     return;
524: 
525:   // create global variable for the constant buffer
526:   CGHLSLOffsetInfo OffsetInfo = CGHLSLOffsetInfo::fromDecl(*BufDecl);
527:   llvm::Type *LayoutTy = convertHLSLSpecificType(ResHandleTy, OffsetInfo);
528:   llvm::GlobalVariable *BufGV = new GlobalVariable(
529:       LayoutTy, /*isConstant*/ false,
530:       GlobalValue::LinkageTypes::ExternalLinkage, PoisonValue::get(LayoutTy),
531:       llvm::formatv("{0}{1}", BufDecl->getName(),
532:                     BufDecl->isCBuffer() ? ".cb" : ".tb"),
533:       GlobalValue::NotThreadLocal);
534:   CGM.getModule().insertGlobalVariable(BufGV);
535: 
536:   // Add globals for constant buffer elements and create metadata nodes
537:   emitBufferGlobalsAndMetadata(BufDecl, BufGV, OffsetInfo);
538: 
539:   // Initialize cbuffer from binding (implicit or explicit)
540:   initializeBufferFromBinding(BufDecl, BufGV);
```
- **EN**: This block defines callable entry points like `emitBufferGlobalsAndMetadata`, `initializeBufferFromBinding`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitBufferGlobalsAndMetadata`, `initializeBufferFromBinding`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 541-560
```cpp
541: }
542: 
543: void CGHLSLRuntime::addRootSignature(
544:     const HLSLRootSignatureDecl *SignatureDecl) {
545:   llvm::Module &M = CGM.getModule();
546:   Triple T(M.getTargetTriple());
547: 
548:   // Generated later with the function decl if not targeting root signature
549:   if (T.getEnvironment() != Triple::EnvironmentType::RootSignature)
550:     return;
551: 
552:   addRootSignatureMD(SignatureDecl->getVersion(),
553:                      SignatureDecl->getRootElements(), nullptr, M);
554: }
555: 
556: llvm::StructType *
557: CGHLSLRuntime::getHLSLBufferLayoutType(const RecordType *StructType) {
558:   const auto Entry = LayoutTypes.find(StructType);
559:   if (Entry != LayoutTypes.end())
560:     return Entry->getSecond();
```
- **EN**: This block defines callable entry points like `addRootSignature`, `T`, `addRootSignatureMD`, `getHLSLBufferLayoutType`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `addRootSignature`, `T`, `addRootSignatureMD`, `getHLSLBufferLayoutType`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 561-580
```cpp
561:   return nullptr;
562: }
563: 
564: void CGHLSLRuntime::addHLSLBufferLayoutType(const RecordType *StructType,
565:                                             llvm::StructType *LayoutTy) {
566:   assert(getHLSLBufferLayoutType(StructType) == nullptr &&
567:          "layout type for this struct already exist");
568:   LayoutTypes[StructType] = LayoutTy;
569: }
570: 
571: void CGHLSLRuntime::finishCodeGen() {
572:   auto &TargetOpts = CGM.getTarget().getTargetOpts();
573:   auto &CodeGenOpts = CGM.getCodeGenOpts();
574:   auto &LangOpts = CGM.getLangOpts();
575:   llvm::Module &M = CGM.getModule();
576:   Triple T(M.getTargetTriple());
577:   if (T.getArch() == Triple::ArchType::dxil)
578:     addDxilValVersion(TargetOpts.DxilValidatorVersion, M);
579:   if (CodeGenOpts.ResMayAlias)
580:     M.setModuleFlag(llvm::Module::ModFlagBehavior::Error, "dx.resmayalias", 1);
```
- **EN**: This block introduces declarations such as `already`; defines callable entry points like `addHLSLBufferLayoutType`, `finishCodeGen`, `T`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `already` 的声明；定义可调用入口，例如 `addHLSLBufferLayoutType`, `finishCodeGen`, `T`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 581-600
```cpp
581:   if (CodeGenOpts.AllResourcesBound)
582:     M.setModuleFlag(llvm::Module::ModFlagBehavior::Error,
583:                     "dx.allresourcesbound", 1);
584:   if (CodeGenOpts.OptimizationLevel == 0)
585:     M.addModuleFlag(llvm::Module::ModFlagBehavior::Override,
586:                     "dx.disable_optimizations", 1);
587: 
588:   // NativeHalfType corresponds to the -fnative-half-type clang option which is
589:   // aliased by clang-dxc's -enable-16bit-types option. This option is used to
590:   // set the UseNativeLowPrecision DXIL module flag in the DirectX backend
591:   if (LangOpts.NativeHalfType)
592:     M.setModuleFlag(llvm::Module::ModFlagBehavior::Error, "dx.nativelowprec",
593:                     1);
594: 
595:   generateGlobalCtorDtorCalls();
596: }
597: 
598: void clang::CodeGen::CGHLSLRuntime::setHLSLEntryAttributes(
599:     const FunctionDecl *FD, llvm::Function *Fn) {
600:   const auto *ShaderAttr = FD->getAttr<HLSLShaderAttr>();
```
- **EN**: This block defines callable entry points like `generateGlobalCtorDtorCalls`, `setHLSLEntryAttributes`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `generateGlobalCtorDtorCalls`, `setHLSLEntryAttributes`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 601-620
```cpp
601:   assert(ShaderAttr && "All entry functions must have a HLSLShaderAttr");
602:   const StringRef ShaderAttrKindStr = "hlsl.shader";
603:   Fn->addFnAttr(ShaderAttrKindStr,
604:                 llvm::Triple::getEnvironmentTypeName(ShaderAttr->getType()));
605:   if (HLSLNumThreadsAttr *NumThreadsAttr = FD->getAttr<HLSLNumThreadsAttr>()) {
606:     const StringRef NumThreadsKindStr = "hlsl.numthreads";
607:     std::string NumThreadsStr =
608:         formatv("{0},{1},{2}", NumThreadsAttr->getX(), NumThreadsAttr->getY(),
609:                 NumThreadsAttr->getZ());
610:     Fn->addFnAttr(NumThreadsKindStr, NumThreadsStr);
611:   }
612:   if (HLSLWaveSizeAttr *WaveSizeAttr = FD->getAttr<HLSLWaveSizeAttr>()) {
613:     const StringRef WaveSizeKindStr = "hlsl.wavesize";
614:     std::string WaveSizeStr =
615:         formatv("{0},{1},{2}", WaveSizeAttr->getMin(), WaveSizeAttr->getMax(),
616:                 WaveSizeAttr->getPreferred());
617:     Fn->addFnAttr(WaveSizeKindStr, WaveSizeStr);
618:   }
619:   // HLSL entry functions are materialized for module functions with
620:   // HLSLShaderAttr attribute. SetLLVMFunctionAttributesForDefinition called
```
- **EN**: This block defines callable entry points like `getEnvironmentTypeName`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getEnvironmentTypeName`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 621-640
```cpp
621:   // later in the compiler-flow for such module functions is not aware of and
622:   // hence not able to set attributes of the newly materialized entry functions.
623:   // So, set attributes of entry function here, as appropriate.
624:   Fn->addFnAttr(llvm::Attribute::NoInline);
625: 
626:   if (CGM.getLangOpts().HLSLSpvEnableMaximalReconvergence) {
627:     Fn->addFnAttr("enable-maximal-reconvergence", "true");
628:   }
629: }
630: 
631: static Value *buildVectorInput(IRBuilder<> &B, Function *F, llvm::Type *Ty) {
632:   if (const auto *VT = dyn_cast<FixedVectorType>(Ty)) {
633:     Value *Result = PoisonValue::get(Ty);
634:     for (unsigned I = 0; I < VT->getNumElements(); ++I) {
635:       Value *Elt = B.CreateCall(F, {B.getInt32(I)});
636:       Result = B.CreateInsertElement(Result, Elt, I);
637:     }
638:     return Result;
639:   }
640:   return B.CreateCall(F, {B.getInt32(0)});
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 641-660
```cpp
641: }
642: 
643: static void addSPIRVBuiltinDecoration(llvm::GlobalVariable *GV,
644:                                       unsigned BuiltIn) {
645:   LLVMContext &Ctx = GV->getContext();
646:   IRBuilder<> B(GV->getContext());
647:   MDNode *Operands = MDNode::get(
648:       Ctx,
649:       {ConstantAsMetadata::get(B.getInt32(/* Spirv::Decoration::BuiltIn */ 11)),
650:        ConstantAsMetadata::get(B.getInt32(BuiltIn))});
651:   MDNode *Decoration = MDNode::get(Ctx, {Operands});
652:   GV->addMetadata("spirv.Decorations", *Decoration);
653: }
654: 
655: static void addLocationDecoration(llvm::GlobalVariable *GV, unsigned Location) {
656:   LLVMContext &Ctx = GV->getContext();
657:   IRBuilder<> B(GV->getContext());
658:   MDNode *Operands =
659:       MDNode::get(Ctx, {ConstantAsMetadata::get(B.getInt32(/* Location */ 30)),
660:                         ConstantAsMetadata::get(B.getInt32(Location))});
```
- **EN**: This block defines callable entry points like `addSPIRVBuiltinDecoration`, `B`, `addLocationDecoration`.
- **CN**: 该代码块定义可调用入口，例如 `addSPIRVBuiltinDecoration`, `B`, `addLocationDecoration`。

### Lines 661-680
```cpp
661:   MDNode *Decoration = MDNode::get(Ctx, {Operands});
662:   GV->addMetadata("spirv.Decorations", *Decoration);
663: }
664: 
665: static llvm::Value *createSPIRVBuiltinLoad(IRBuilder<> &B, llvm::Module &M,
666:                                            llvm::Type *Ty, const Twine &Name,
667:                                            unsigned BuiltInID) {
668:   auto *GV = new llvm::GlobalVariable(
669:       M, Ty, /* isConstant= */ true, llvm::GlobalValue::ExternalLinkage,
670:       /* Initializer= */ nullptr, Name, /* insertBefore= */ nullptr,
671:       llvm::GlobalVariable::GeneralDynamicTLSModel,
672:       /* AddressSpace */ 7, /* isExternallyInitialized= */ true);
673:   addSPIRVBuiltinDecoration(GV, BuiltInID);
674:   GV->setVisibility(llvm::GlobalValue::HiddenVisibility);
675:   return B.CreateLoad(Ty, GV);
676: }
677: 
678: static llvm::Value *createSPIRVLocationLoad(IRBuilder<> &B, llvm::Module &M,
679:                                             llvm::Type *Ty, unsigned Location,
680:                                             StringRef Name) {
```
- **EN**: This block defines callable entry points like `addSPIRVBuiltinDecoration`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `addSPIRVBuiltinDecoration`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 681-700
```cpp
681:   auto *GV = new llvm::GlobalVariable(
682:       M, Ty, /* isConstant= */ true, llvm::GlobalValue::ExternalLinkage,
683:       /* Initializer= */ nullptr, /* Name= */ Name, /* insertBefore= */ nullptr,
684:       llvm::GlobalVariable::GeneralDynamicTLSModel,
685:       /* AddressSpace */ 7, /* isExternallyInitialized= */ true);
686:   GV->setVisibility(llvm::GlobalValue::HiddenVisibility);
687:   addLocationDecoration(GV, Location);
688:   return B.CreateLoad(Ty, GV);
689: }
690: 
691: llvm::Value *CGHLSLRuntime::emitSPIRVUserSemanticLoad(
692:     llvm::IRBuilder<> &B, llvm::Type *Type, const clang::DeclaratorDecl *Decl,
693:     HLSLAppliedSemanticAttr *Semantic, std::optional<unsigned> Index) {
694:   Twine BaseName = Twine(Semantic->getAttrName()->getName());
695:   Twine VariableName = BaseName.concat(Twine(Index.value_or(0)));
696: 
697:   unsigned Location = SPIRVLastAssignedInputSemanticLocation;
698:   if (auto *L = Decl->getAttr<HLSLVkLocationAttr>())
699:     Location = L->getLocation();
700: 
```
- **EN**: This block defines callable entry points like `addLocationDecoration`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `addLocationDecoration`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 701-720
```cpp
701:   // DXC completely ignores the semantic/index pair. Location are assigned from
702:   // the first semantic to the last.
703:   llvm::ArrayType *AT = dyn_cast<llvm::ArrayType>(Type);
704:   unsigned ElementCount = AT ? AT->getNumElements() : 1;
705:   SPIRVLastAssignedInputSemanticLocation += ElementCount;
706: 
707:   return createSPIRVLocationLoad(B, CGM.getModule(), Type, Location,
708:                                  VariableName.str());
709: }
710: 
711: static void createSPIRVLocationStore(IRBuilder<> &B, llvm::Module &M,
712:                                      llvm::Value *Source, unsigned Location,
713:                                      StringRef Name) {
714:   auto *GV = new llvm::GlobalVariable(
715:       M, Source->getType(), /* isConstant= */ false,
716:       llvm::GlobalValue::ExternalLinkage,
717:       /* Initializer= */ nullptr, /* Name= */ Name, /* insertBefore= */ nullptr,
718:       llvm::GlobalVariable::GeneralDynamicTLSModel,
719:       /* AddressSpace */ 8, /* isExternallyInitialized= */ false);
720:   GV->setVisibility(llvm::GlobalValue::HiddenVisibility);
```
- **EN**: This block defines callable entry points like `createSPIRVLocationLoad`, `createSPIRVLocationStore`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `createSPIRVLocationLoad`, `createSPIRVLocationStore`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 721-740
```cpp
721:   addLocationDecoration(GV, Location);
722:   B.CreateStore(Source, GV);
723: }
724: 
725: void CGHLSLRuntime::emitSPIRVUserSemanticStore(
726:     llvm::IRBuilder<> &B, llvm::Value *Source,
727:     const clang::DeclaratorDecl *Decl, HLSLAppliedSemanticAttr *Semantic,
728:     std::optional<unsigned> Index) {
729:   Twine BaseName = Twine(Semantic->getAttrName()->getName());
730:   Twine VariableName = BaseName.concat(Twine(Index.value_or(0)));
731: 
732:   unsigned Location = SPIRVLastAssignedOutputSemanticLocation;
733:   if (auto *L = Decl->getAttr<HLSLVkLocationAttr>())
734:     Location = L->getLocation();
735: 
736:   // DXC completely ignores the semantic/index pair. Location are assigned from
737:   // the first semantic to the last.
738:   llvm::ArrayType *AT = dyn_cast<llvm::ArrayType>(Source->getType());
739:   unsigned ElementCount = AT ? AT->getNumElements() : 1;
740:   SPIRVLastAssignedOutputSemanticLocation += ElementCount;
```
- **EN**: This block defines callable entry points like `addLocationDecoration`, `emitSPIRVUserSemanticStore`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `addLocationDecoration`, `emitSPIRVUserSemanticStore`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 741-760
```cpp
741:   createSPIRVLocationStore(B, CGM.getModule(), Source, Location,
742:                            VariableName.str());
743: }
744: 
745: llvm::Value *
746: CGHLSLRuntime::emitDXILUserSemanticLoad(llvm::IRBuilder<> &B, llvm::Type *Type,
747:                                         HLSLAppliedSemanticAttr *Semantic,
748:                                         std::optional<unsigned> Index) {
749:   Twine BaseName = Twine(Semantic->getAttrName()->getName());
750:   Twine VariableName = BaseName.concat(Twine(Index.value_or(0)));
751: 
752:   // DXIL packing rules etc shall be handled here.
753:   // FIXME: generate proper sigpoint, index, col, row values.
754:   // FIXME: also DXIL loads vectors element by element.
755:   SmallVector<Value *> Args{B.getInt32(4), B.getInt32(0), B.getInt32(0),
756:                             B.getInt8(0),
757:                             llvm::PoisonValue::get(B.getInt32Ty())};
758: 
759:   llvm::Intrinsic::ID IntrinsicID = llvm::Intrinsic::dx_load_input;
760:   llvm::Value *Value = B.CreateIntrinsic(/*ReturnType=*/Type, IntrinsicID, Args,
```
- **EN**: This block defines callable entry points like `createSPIRVLocationStore`, `emitDXILUserSemanticLoad`.
- **CN**: 该代码块定义可调用入口，例如 `createSPIRVLocationStore`, `emitDXILUserSemanticLoad`。

### Lines 761-780
```cpp
761:                                          nullptr, VariableName);
762:   return Value;
763: }
764: 
765: void CGHLSLRuntime::emitDXILUserSemanticStore(llvm::IRBuilder<> &B,
766:                                               llvm::Value *Source,
767:                                               HLSLAppliedSemanticAttr *Semantic,
768:                                               std::optional<unsigned> Index) {
769:   // DXIL packing rules etc shall be handled here.
770:   // FIXME: generate proper sigpoint, index, col, row values.
771:   SmallVector<Value *> Args{B.getInt32(4),
772:                             B.getInt32(0),
773:                             B.getInt32(0),
774:                             B.getInt8(0),
775:                             llvm::PoisonValue::get(B.getInt32Ty()),
776:                             Source};
777: 
778:   llvm::Intrinsic::ID IntrinsicID = llvm::Intrinsic::dx_store_output;
779:   B.CreateIntrinsic(/*ReturnType=*/CGM.VoidTy, IntrinsicID, Args, nullptr);
780: }
```
- **EN**: This block defines callable entry points like `emitDXILUserSemanticStore`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `emitDXILUserSemanticStore`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 781-800
```cpp
781: 
782: llvm::Value *CGHLSLRuntime::emitUserSemanticLoad(
783:     IRBuilder<> &B, llvm::Type *Type, const clang::DeclaratorDecl *Decl,
784:     HLSLAppliedSemanticAttr *Semantic, std::optional<unsigned> Index) {
785:   if (CGM.getTarget().getTriple().isSPIRV())
786:     return emitSPIRVUserSemanticLoad(B, Type, Decl, Semantic, Index);
787: 
788:   if (CGM.getTarget().getTriple().isDXIL())
789:     return emitDXILUserSemanticLoad(B, Type, Semantic, Index);
790: 
791:   llvm_unreachable("Unsupported target for user-semantic load.");
792: }
793: 
794: void CGHLSLRuntime::emitUserSemanticStore(IRBuilder<> &B, llvm::Value *Source,
795:                                           const clang::DeclaratorDecl *Decl,
796:                                           HLSLAppliedSemanticAttr *Semantic,
797:                                           std::optional<unsigned> Index) {
798:   if (CGM.getTarget().getTriple().isSPIRV())
799:     return emitSPIRVUserSemanticStore(B, Source, Decl, Semantic, Index);
800: 
```
- **EN**: This block defines callable entry points like `emitUserSemanticStore`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitUserSemanticStore`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 801-820
```cpp
801:   if (CGM.getTarget().getTriple().isDXIL())
802:     return emitDXILUserSemanticStore(B, Source, Semantic, Index);
803: 
804:   llvm_unreachable("Unsupported target for user-semantic load.");
805: }
806: 
807: llvm::Value *CGHLSLRuntime::emitSystemSemanticLoad(
808:     IRBuilder<> &B, const FunctionDecl *FD, llvm::Type *Type,
809:     const clang::DeclaratorDecl *Decl, HLSLAppliedSemanticAttr *Semantic,
810:     std::optional<unsigned> Index) {
811: 
812:   std::string SemanticName = Semantic->getAttrName()->getName().upper();
813:   if (SemanticName == "SV_GROUPINDEX") {
814:     llvm::Function *GroupIndex =
815:         CGM.getIntrinsic(getFlattenedThreadIdInGroupIntrinsic());
816:     return B.CreateCall(FunctionCallee(GroupIndex));
817:   }
818: 
819:   if (SemanticName == "SV_DISPATCHTHREADID") {
820:     llvm::Intrinsic::ID IntrinID = getThreadIdIntrinsic();
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 821-840
```cpp
821:     llvm::Function *ThreadIDIntrinsic =
822:         llvm::Intrinsic::isOverloaded(IntrinID)
823:             ? CGM.getIntrinsic(IntrinID, {CGM.Int32Ty})
824:             : CGM.getIntrinsic(IntrinID);
825:     return buildVectorInput(B, ThreadIDIntrinsic, Type);
826:   }
827: 
828:   if (SemanticName == "SV_GROUPTHREADID") {
829:     llvm::Intrinsic::ID IntrinID = getGroupThreadIdIntrinsic();
830:     llvm::Function *GroupThreadIDIntrinsic =
831:         llvm::Intrinsic::isOverloaded(IntrinID)
832:             ? CGM.getIntrinsic(IntrinID, {CGM.Int32Ty})
833:             : CGM.getIntrinsic(IntrinID);
834:     return buildVectorInput(B, GroupThreadIDIntrinsic, Type);
835:   }
836: 
837:   if (SemanticName == "SV_GROUPID") {
838:     llvm::Intrinsic::ID IntrinID = getGroupIdIntrinsic();
839:     llvm::Function *GroupIDIntrinsic =
840:         llvm::Intrinsic::isOverloaded(IntrinID)
```
- **EN**: This block defines callable entry points like `buildVectorInput`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `buildVectorInput`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 841-860
```cpp
841:             ? CGM.getIntrinsic(IntrinID, {CGM.Int32Ty})
842:             : CGM.getIntrinsic(IntrinID);
843:     return buildVectorInput(B, GroupIDIntrinsic, Type);
844:   }
845: 
846:   const auto *ShaderAttr = FD->getAttr<HLSLShaderAttr>();
847:   assert(ShaderAttr && "Entry point has no shader attribute");
848:   llvm::Triple::EnvironmentType ST = ShaderAttr->getType();
849: 
850:   if (SemanticName == "SV_POSITION") {
851:     if (ST == Triple::EnvironmentType::Pixel) {
852:       if (CGM.getTarget().getTriple().isSPIRV())
853:         return createSPIRVBuiltinLoad(B, CGM.getModule(), Type,
854:                                       Semantic->getAttrName()->getName(),
855:                                       /* BuiltIn::FragCoord */ 15);
856:       if (CGM.getTarget().getTriple().isDXIL())
857:         return emitDXILUserSemanticLoad(B, Type, Semantic, Index);
858:     }
859: 
860:     if (ST == Triple::EnvironmentType::Vertex) {
```
- **EN**: This block defines callable entry points like `buildVectorInput`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `buildVectorInput`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 861-880
```cpp
861:       return emitUserSemanticLoad(B, Type, Decl, Semantic, Index);
862:     }
863:   }
864: 
865:   if (SemanticName == "SV_VERTEXID") {
866:     if (ST == Triple::EnvironmentType::Vertex) {
867:       if (CGM.getTarget().getTriple().isSPIRV())
868:         return createSPIRVBuiltinLoad(B, CGM.getModule(), Type,
869:                                       Semantic->getAttrName()->getName(),
870:                                       /* BuiltIn::VertexIndex */ 42);
871:       else
872:         return emitDXILUserSemanticLoad(B, Type, Semantic, Index);
873:     }
874:   }
875: 
876:   llvm_unreachable(
877:       "Load hasn't been implemented yet for this system semantic. FIXME");
878: }
879: 
880: static void createSPIRVBuiltinStore(IRBuilder<> &B, llvm::Module &M,
```
- **EN**: This block defines callable entry points like `emitUserSemanticLoad`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitUserSemanticLoad`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 881-900
```cpp
881:                                     llvm::Value *Source, const Twine &Name,
882:                                     unsigned BuiltInID) {
883:   auto *GV = new llvm::GlobalVariable(
884:       M, Source->getType(), /* isConstant= */ false,
885:       llvm::GlobalValue::ExternalLinkage,
886:       /* Initializer= */ nullptr, Name, /* insertBefore= */ nullptr,
887:       llvm::GlobalVariable::GeneralDynamicTLSModel,
888:       /* AddressSpace */ 8, /* isExternallyInitialized= */ false);
889:   addSPIRVBuiltinDecoration(GV, BuiltInID);
890:   GV->setVisibility(llvm::GlobalValue::HiddenVisibility);
891:   B.CreateStore(Source, GV);
892: }
893: 
894: void CGHLSLRuntime::emitSystemSemanticStore(IRBuilder<> &B, llvm::Value *Source,
895:                                             const clang::DeclaratorDecl *Decl,
896:                                             HLSLAppliedSemanticAttr *Semantic,
897:                                             std::optional<unsigned> Index) {
898: 
899:   std::string SemanticName = Semantic->getAttrName()->getName().upper();
900:   if (SemanticName == "SV_POSITION") {
```
- **EN**: This block defines callable entry points like `addSPIRVBuiltinDecoration`, `emitSystemSemanticStore`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `addSPIRVBuiltinDecoration`, `emitSystemSemanticStore`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 901-920
```cpp
901:     if (CGM.getTarget().getTriple().isDXIL()) {
902:       emitDXILUserSemanticStore(B, Source, Semantic, Index);
903:       return;
904:     }
905: 
906:     if (CGM.getTarget().getTriple().isSPIRV()) {
907:       createSPIRVBuiltinStore(B, CGM.getModule(), Source,
908:                               Semantic->getAttrName()->getName(),
909:                               /* BuiltIn::Position */ 0);
910:       return;
911:     }
912:   }
913: 
914:   if (SemanticName == "SV_TARGET") {
915:     emitUserSemanticStore(B, Source, Decl, Semantic, Index);
916:     return;
917:   }
918: 
919:   llvm_unreachable(
920:       "Store hasn't been implemented yet for this system semantic. FIXME");
```
- **EN**: This block defines callable entry points like `emitDXILUserSemanticStore`, `emitUserSemanticStore`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitDXILUserSemanticStore`, `emitUserSemanticStore`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 921-940
```cpp
921: }
922: 
923: llvm::Value *CGHLSLRuntime::handleScalarSemanticLoad(
924:     IRBuilder<> &B, const FunctionDecl *FD, llvm::Type *Type,
925:     const clang::DeclaratorDecl *Decl, HLSLAppliedSemanticAttr *Semantic) {
926: 
927:   std::optional<unsigned> Index = Semantic->getSemanticIndex();
928:   if (Semantic->getAttrName()->getName().starts_with_insensitive("SV_"))
929:     return emitSystemSemanticLoad(B, FD, Type, Decl, Semantic, Index);
930:   return emitUserSemanticLoad(B, Type, Decl, Semantic, Index);
931: }
932: 
933: void CGHLSLRuntime::handleScalarSemanticStore(
934:     IRBuilder<> &B, const FunctionDecl *FD, llvm::Value *Source,
935:     const clang::DeclaratorDecl *Decl, HLSLAppliedSemanticAttr *Semantic) {
936:   std::optional<unsigned> Index = Semantic->getSemanticIndex();
937:   if (Semantic->getAttrName()->getName().starts_with_insensitive("SV_"))
938:     emitSystemSemanticStore(B, Source, Decl, Semantic, Index);
939:   else
940:     emitUserSemanticStore(B, Source, Decl, Semantic, Index);
```
- **EN**: This block defines callable entry points like `emitUserSemanticLoad`, `handleScalarSemanticStore`, `emitUserSemanticStore`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitUserSemanticLoad`, `handleScalarSemanticStore`, `emitUserSemanticStore`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 941-960
```cpp
941: }
942: 
943: std::pair<llvm::Value *, specific_attr_iterator<HLSLAppliedSemanticAttr>>
944: CGHLSLRuntime::handleStructSemanticLoad(
945:     IRBuilder<> &B, const FunctionDecl *FD, llvm::Type *Type,
946:     const clang::DeclaratorDecl *Decl,
947:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrBegin,
948:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrEnd) {
949:   const llvm::StructType *ST = cast<StructType>(Type);
950:   const clang::RecordDecl *RD = Decl->getType()->getAsRecordDecl();
951: 
952:   assert(RD->getNumFields() == ST->getNumElements());
953: 
954:   llvm::Value *Aggregate = llvm::PoisonValue::get(Type);
955:   auto FieldDecl = RD->field_begin();
956:   for (unsigned I = 0; I < ST->getNumElements(); ++I) {
957:     auto [ChildValue, NextAttr] = handleSemanticLoad(
958:         B, FD, ST->getElementType(I), *FieldDecl, AttrBegin, AttrEnd);
959:     AttrBegin = NextAttr;
960:     assert(ChildValue);
```
- **EN**: This block defines callable entry points like `handleStructSemanticLoad`; uses control flow (for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `handleStructSemanticLoad`；通过控制流（for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 961-980
```cpp
961:     Aggregate = B.CreateInsertValue(Aggregate, ChildValue, I);
962:     ++FieldDecl;
963:   }
964: 
965:   return std::make_pair(Aggregate, AttrBegin);
966: }
967: 
968: specific_attr_iterator<HLSLAppliedSemanticAttr>
969: CGHLSLRuntime::handleStructSemanticStore(
970:     IRBuilder<> &B, const FunctionDecl *FD, llvm::Value *Source,
971:     const clang::DeclaratorDecl *Decl,
972:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrBegin,
973:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrEnd) {
974: 
975:   const llvm::StructType *ST = cast<StructType>(Source->getType());
976: 
977:   const clang::RecordDecl *RD = nullptr;
978:   if (const FunctionDecl *FD = dyn_cast<FunctionDecl>(Decl))
979:     RD = FD->getDeclaredReturnType()->getAsRecordDecl();
980:   else
```
- **EN**: This block defines callable entry points like `make_pair`, `handleStructSemanticStore`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `make_pair`, `handleStructSemanticStore`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 981-1000
```cpp
 981:     RD = Decl->getType()->getAsRecordDecl();
 982:   assert(RD);
 983: 
 984:   assert(RD->getNumFields() == ST->getNumElements());
 985: 
 986:   auto FieldDecl = RD->field_begin();
 987:   for (unsigned I = 0; I < ST->getNumElements(); ++I, ++FieldDecl) {
 988:     llvm::Value *Extract = B.CreateExtractValue(Source, I);
 989:     AttrBegin =
 990:         handleSemanticStore(B, FD, Extract, *FieldDecl, AttrBegin, AttrEnd);
 991:   }
 992: 
 993:   return AttrBegin;
 994: }
 995: 
 996: std::pair<llvm::Value *, specific_attr_iterator<HLSLAppliedSemanticAttr>>
 997: CGHLSLRuntime::handleSemanticLoad(
 998:     IRBuilder<> &B, const FunctionDecl *FD, llvm::Type *Type,
 999:     const clang::DeclaratorDecl *Decl,
1000:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrBegin,
```
- **EN**: This block defines callable entry points like `handleSemanticStore`; uses control flow (for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `handleSemanticStore`；通过控制流（for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1001-1020
```cpp
1001:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrEnd) {
1002:   assert(AttrBegin != AttrEnd);
1003:   if (Type->isStructTy())
1004:     return handleStructSemanticLoad(B, FD, Type, Decl, AttrBegin, AttrEnd);
1005: 
1006:   HLSLAppliedSemanticAttr *Attr = *AttrBegin;
1007:   ++AttrBegin;
1008:   return std::make_pair(handleScalarSemanticLoad(B, FD, Type, Decl, Attr),
1009:                         AttrBegin);
1010: }
1011: 
1012: specific_attr_iterator<HLSLAppliedSemanticAttr>
1013: CGHLSLRuntime::handleSemanticStore(
1014:     IRBuilder<> &B, const FunctionDecl *FD, llvm::Value *Source,
1015:     const clang::DeclaratorDecl *Decl,
1016:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrBegin,
1017:     specific_attr_iterator<HLSLAppliedSemanticAttr> AttrEnd) {
1018:   assert(AttrBegin != AttrEnd);
1019:   if (Source->getType()->isStructTy())
1020:     return handleStructSemanticStore(B, FD, Source, Decl, AttrBegin, AttrEnd);
```
- **EN**: This block defines callable entry points like `make_pair`, `handleSemanticStore`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `make_pair`, `handleSemanticStore`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1021-1040
```cpp
1021: 
1022:   HLSLAppliedSemanticAttr *Attr = *AttrBegin;
1023:   ++AttrBegin;
1024:   handleScalarSemanticStore(B, FD, Source, Decl, Attr);
1025:   return AttrBegin;
1026: }
1027: 
1028: void CGHLSLRuntime::emitEntryFunction(const FunctionDecl *FD,
1029:                                       llvm::Function *Fn) {
1030:   llvm::Module &M = CGM.getModule();
1031:   llvm::LLVMContext &Ctx = M.getContext();
1032:   auto *EntryTy = llvm::FunctionType::get(llvm::Type::getVoidTy(Ctx), false);
1033:   Function *EntryFn =
1034:       Function::Create(EntryTy, Function::ExternalLinkage, FD->getName(), &M);
1035: 
1036:   // Copy function attributes over, we have no argument or return attributes
1037:   // that can be valid on the real entry.
1038:   AttributeList NewAttrs = AttributeList::get(Ctx, AttributeList::FunctionIndex,
1039:                                               Fn->getAttributes().getFnAttrs());
1040:   EntryFn->setAttributes(NewAttrs);
```
- **EN**: This block defines callable entry points like `handleScalarSemanticStore`, `emitEntryFunction`, `Create`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `handleScalarSemanticStore`, `emitEntryFunction`, `Create`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 1041-1060
```cpp
1041:   setHLSLEntryAttributes(FD, EntryFn);
1042: 
1043:   // Set the called function as internal linkage.
1044:   Fn->setLinkage(GlobalValue::InternalLinkage);
1045: 
1046:   BasicBlock *BB = BasicBlock::Create(Ctx, "entry", EntryFn);
1047:   IRBuilder<> B(BB);
1048:   llvm::SmallVector<Value *> Args;
1049: 
1050:   SmallVector<OperandBundleDef, 1> OB;
1051:   if (CGM.shouldEmitConvergenceTokens()) {
1052:     assert(EntryFn->isConvergent());
1053:     llvm::Value *I =
1054:         B.CreateIntrinsic(llvm::Intrinsic::experimental_convergence_entry, {});
1055:     llvm::Value *bundleArgs[] = {I};
1056:     OB.emplace_back("convergencectrl", bundleArgs);
1057:   }
1058: 
1059:   SmallVector<std::pair<llvm::Value *, llvm::Type *>> OutputSemantic;
1060: 
```
- **EN**: This block defines callable entry points like `setHLSLEntryAttributes`, `B`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `setHLSLEntryAttributes`, `B`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1061-1080
```cpp
1061:   unsigned SRetOffset = 0;
1062:   for (const auto &Param : Fn->args()) {
1063:     if (Param.hasStructRetAttr()) {
1064:       SRetOffset = 1;
1065:       llvm::Type *VarType = Param.getParamStructRetType();
1066:       llvm::Value *Var =
1067:           CGM.getLangOpts().EmitLogicalPointer
1068:               ? cast<Instruction>(B.CreateStructuredAlloca(VarType))
1069:               : cast<Instruction>(B.CreateAlloca(VarType));
1070:       OutputSemantic.push_back(std::make_pair(Var, VarType));
1071:       Args.push_back(Var);
1072:       continue;
1073:     }
1074: 
1075:     const ParmVarDecl *PD = FD->getParamDecl(Param.getArgNo() - SRetOffset);
1076:     llvm::Value *SemanticValue = nullptr;
1077:     // FIXME: support inout/out parameters for semantics.
1078:     if ([[maybe_unused]] HLSLParamModifierAttr *MA =
1079:             PD->getAttr<HLSLParamModifierAttr>()) {
1080:       llvm_unreachable("Not handled yet");
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1081-1100
```cpp
1081:     } else {
1082:       llvm::Type *ParamType =
1083:           Param.hasByValAttr() ? Param.getParamByValType() : Param.getType();
1084:       auto AttrBegin = PD->specific_attr_begin<HLSLAppliedSemanticAttr>();
1085:       auto AttrEnd = PD->specific_attr_end<HLSLAppliedSemanticAttr>();
1086:       auto Result =
1087:           handleSemanticLoad(B, FD, ParamType, PD, AttrBegin, AttrEnd);
1088:       SemanticValue = Result.first;
1089:       if (!SemanticValue)
1090:         return;
1091:       if (Param.hasByValAttr()) {
1092:         llvm::Value *Var =
1093:             CGM.getLangOpts().EmitLogicalPointer
1094:                 ? cast<Instruction>(
1095:                       B.CreateStructuredAlloca(Param.getParamByValType()))
1096:                 : cast<Instruction>(B.CreateAlloca(Param.getParamByValType()));
1097:         B.CreateStore(SemanticValue, Var);
1098:         SemanticValue = Var;
1099:       }
1100:     }
```
- **EN**: This block defines callable entry points like `handleSemanticLoad`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `handleSemanticLoad`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1101-1120
```cpp
1101: 
1102:     assert(SemanticValue);
1103:     Args.push_back(SemanticValue);
1104:   }
1105: 
1106:   CallInst *CI = B.CreateCall(FunctionCallee(Fn), Args, OB);
1107:   CI->setCallingConv(Fn->getCallingConv());
1108: 
1109:   if (Fn->getReturnType() != CGM.VoidTy)
1110:     // Element type is unused, so set to dummy value (NULL).
1111:     OutputSemantic.push_back(std::make_pair(CI, nullptr));
1112: 
1113:   for (auto &SourcePair : OutputSemantic) {
1114:     llvm::Value *Source = SourcePair.first;
1115:     llvm::Type *ElementType = SourcePair.second;
1116:     AllocaInst *AI = dyn_cast<AllocaInst>(Source);
1117:     llvm::Value *SourceValue = AI ? B.CreateLoad(ElementType, Source) : Source;
1118: 
1119:     auto AttrBegin = FD->specific_attr_begin<HLSLAppliedSemanticAttr>();
1120:     auto AttrEnd = FD->specific_attr_end<HLSLAppliedSemanticAttr>();
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1121-1140
```cpp
1121:     handleSemanticStore(B, FD, SourceValue, FD, AttrBegin, AttrEnd);
1122:   }
1123: 
1124:   B.CreateRetVoid();
1125: 
1126:   // Add and identify root signature to function, if applicable
1127:   for (const Attr *Attr : FD->getAttrs()) {
1128:     if (const auto *RSAttr = dyn_cast<RootSignatureAttr>(Attr)) {
1129:       auto *RSDecl = RSAttr->getSignatureDecl();
1130:       addRootSignatureMD(RSDecl->getVersion(), RSDecl->getRootElements(),
1131:                          EntryFn, M);
1132:     }
1133:   }
1134: }
1135: 
1136: static void gatherFunctions(SmallVectorImpl<Function *> &Fns, llvm::Module &M,
1137:                             bool CtorOrDtor) {
1138:   const auto *GV =
1139:       M.getNamedGlobal(CtorOrDtor ? "llvm.global_ctors" : "llvm.global_dtors");
1140:   if (!GV)
```
- **EN**: This block defines callable entry points like `handleSemanticStore`, `addRootSignatureMD`, `gatherFunctions`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `handleSemanticStore`, `addRootSignatureMD`, `gatherFunctions`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1141-1160
```cpp
1141:     return;
1142:   const auto *CA = dyn_cast<ConstantArray>(GV->getInitializer());
1143:   if (!CA)
1144:     return;
1145:   // The global_ctor array elements are a struct [Priority, Fn *, COMDat].
1146:   // HLSL neither supports priorities or COMDat values, so we will check those
1147:   // in an assert but not handle them.
1148: 
1149:   for (const auto &Ctor : CA->operands()) {
1150:     if (isa<ConstantAggregateZero>(Ctor))
1151:       continue;
1152:     ConstantStruct *CS = cast<ConstantStruct>(Ctor);
1153: 
1154:     assert(cast<ConstantInt>(CS->getOperand(0))->getValue() == 65535 &&
1155:            "HLSL doesn't support setting priority for global ctors.");
1156:     assert(isa<ConstantPointerNull>(CS->getOperand(2)) &&
1157:            "HLSL doesn't support COMDat for global ctors.");
1158:     Fns.push_back(cast<Function>(CS->getOperand(1)));
1159:   }
1160: }
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1161-1180
```cpp
1161: 
1162: void CGHLSLRuntime::generateGlobalCtorDtorCalls() {
1163:   llvm::Module &M = CGM.getModule();
1164:   SmallVector<Function *> CtorFns;
1165:   SmallVector<Function *> DtorFns;
1166:   gatherFunctions(CtorFns, M, true);
1167:   gatherFunctions(DtorFns, M, false);
1168: 
1169:   // Insert a call to the global constructor at the beginning of the entry block
1170:   // to externally exported functions. This is a bit of a hack, but HLSL allows
1171:   // global constructors, but doesn't support driver initialization of globals.
1172:   for (auto &F : M.functions()) {
1173:     if (!F.hasFnAttribute("hlsl.shader"))
1174:       continue;
1175:     auto *Token = getConvergenceToken(F.getEntryBlock());
1176:     Instruction *IP = &*F.getEntryBlock().begin();
1177:     SmallVector<OperandBundleDef, 1> OB;
1178:     if (Token) {
1179:       llvm::Value *bundleArgs[] = {Token};
1180:       OB.emplace_back("convergencectrl", bundleArgs);
```
- **EN**: This block defines callable entry points like `generateGlobalCtorDtorCalls`, `gatherFunctions`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `generateGlobalCtorDtorCalls`, `gatherFunctions`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1181-1200
```cpp
1181:       IP = Token->getNextNode();
1182:     }
1183:     IRBuilder<> B(IP);
1184:     for (auto *Fn : CtorFns) {
1185:       auto CI = B.CreateCall(FunctionCallee(Fn), {}, OB);
1186:       CI->setCallingConv(Fn->getCallingConv());
1187:     }
1188: 
1189:     // Insert global dtors before the terminator of the last instruction
1190:     B.SetInsertPoint(F.back().getTerminator());
1191:     for (auto *Fn : DtorFns) {
1192:       auto CI = B.CreateCall(FunctionCallee(Fn), {}, OB);
1193:       CI->setCallingConv(Fn->getCallingConv());
1194:     }
1195:   }
1196: 
1197:   // No need to keep global ctors/dtors for non-lib profile after call to
1198:   // ctors/dtors added for entry.
1199:   Triple T(M.getTargetTriple());
1200:   if (T.getEnvironment() != Triple::EnvironmentType::Library) {
```
- **EN**: This block defines callable entry points like `B`, `T`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `B`, `T`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1201-1220
```cpp
1201:     if (auto *GV = M.getNamedGlobal("llvm.global_ctors"))
1202:       GV->eraseFromParent();
1203:     if (auto *GV = M.getNamedGlobal("llvm.global_dtors"))
1204:       GV->eraseFromParent();
1205:   }
1206: }
1207: 
1208: static void initializeBuffer(CodeGenModule &CGM, llvm::GlobalVariable *GV,
1209:                              Intrinsic::ID IntrID,
1210:                              ArrayRef<llvm::Value *> Args) {
1211: 
1212:   LLVMContext &Ctx = CGM.getLLVMContext();
1213:   llvm::Function *InitResFunc = llvm::Function::Create(
1214:       llvm::FunctionType::get(CGM.VoidTy, false),
1215:       llvm::GlobalValue::InternalLinkage,
1216:       ("_init_buffer_" + GV->getName()).str(), CGM.getModule());
1217:   InitResFunc->addFnAttr(llvm::Attribute::AlwaysInline);
1218: 
1219:   llvm::BasicBlock *EntryBB =
1220:       llvm::BasicBlock::Create(Ctx, "entry", InitResFunc);
```
- **EN**: This block defines callable entry points like `initializeBuffer`, `get`, `Create`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `initializeBuffer`, `get`, `Create`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1221-1240
```cpp
1221:   CGBuilderTy Builder(CGM, Ctx);
1222:   const DataLayout &DL = CGM.getModule().getDataLayout();
1223:   Builder.SetInsertPoint(EntryBB);
1224: 
1225:   // Make sure the global variable is buffer resource handle
1226:   llvm::Type *HandleTy = GV->getValueType();
1227:   assert(HandleTy->isTargetExtTy() && "unexpected type of the buffer global");
1228: 
1229:   llvm::Value *CreateHandle = Builder.CreateIntrinsic(
1230:       /*ReturnType=*/HandleTy, IntrID, Args, nullptr,
1231:       Twine(GV->getName()).concat("_h"));
1232: 
1233:   Builder.CreateAlignedStore(CreateHandle, GV, GV->getPointerAlignment(DL));
1234:   Builder.CreateRetVoid();
1235: 
1236:   CGM.AddCXXGlobalInit(InitResFunc);
1237: }
1238: 
1239: void CGHLSLRuntime::initializeBufferFromBinding(const HLSLBufferDecl *BufDecl,
1240:                                                 llvm::GlobalVariable *GV) {
```
- **EN**: This block defines callable entry points like `Builder`, `Twine`, `initializeBufferFromBinding`; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Builder`, `Twine`, `initializeBufferFromBinding`；使用断言或不可达标记保护关键不变量。

### Lines 1241-1260
```cpp
1241:   ResourceBindingAttrs Binding(BufDecl);
1242:   assert(Binding.hasBinding() &&
1243:          "cbuffer/tbuffer should always have resource binding attribute");
1244: 
1245:   auto *Index = llvm::ConstantInt::get(CGM.IntTy, 0);
1246:   auto *RangeSize = llvm::ConstantInt::get(CGM.IntTy, 1);
1247:   auto *Space = llvm::ConstantInt::get(CGM.IntTy, Binding.getSpace());
1248:   Value *Name = buildNameForResource(BufDecl->getName(), CGM);
1249: 
1250:   // buffer with explicit binding
1251:   if (Binding.isExplicit()) {
1252:     llvm::Intrinsic::ID IntrinsicID =
1253:         CGM.getHLSLRuntime().getCreateHandleFromBindingIntrinsic();
1254:     auto *RegSlot = llvm::ConstantInt::get(CGM.IntTy, Binding.getSlot());
1255:     SmallVector<Value *> Args{Space, RegSlot, RangeSize, Index, Name};
1256:     initializeBuffer(CGM, GV, IntrinsicID, Args);
1257:   } else {
1258:     // buffer with implicit binding
1259:     llvm::Intrinsic::ID IntrinsicID =
1260:         CGM.getHLSLRuntime().getCreateHandleFromImplicitBindingIntrinsic();
```
- **EN**: This block defines callable entry points like `Binding`, `initializeBuffer`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Binding`, `initializeBuffer`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1261-1280
```cpp
1261:     auto *OrderID =
1262:         llvm::ConstantInt::get(CGM.IntTy, Binding.getImplicitOrderID());
1263:     SmallVector<Value *> Args{OrderID, Space, RangeSize, Index, Name};
1264:     initializeBuffer(CGM, GV, IntrinsicID, Args);
1265:   }
1266: }
1267: 
1268: void CGHLSLRuntime::handleGlobalVarDefinition(const VarDecl *VD,
1269:                                               llvm::GlobalVariable *GV) {
1270:   if (auto Attr = VD->getAttr<HLSLVkExtBuiltinInputAttr>())
1271:     addSPIRVBuiltinDecoration(GV, Attr->getBuiltIn());
1272:   if (auto Attr = VD->getAttr<HLSLVkExtBuiltinOutputAttr>())
1273:     addSPIRVBuiltinDecoration(GV, Attr->getBuiltIn());
1274: }
1275: 
1276: llvm::Instruction *CGHLSLRuntime::getConvergenceToken(BasicBlock &BB) {
1277:   if (!CGM.shouldEmitConvergenceTokens())
1278:     return nullptr;
1279: 
1280:   auto E = BB.end();
```
- **EN**: This block defines callable entry points like `get`, `initializeBuffer`, `handleGlobalVarDefinition`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `get`, `initializeBuffer`, `handleGlobalVarDefinition`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1281-1300
```cpp
1281:   for (auto I = BB.begin(); I != E; ++I) {
1282:     auto *II = dyn_cast<llvm::IntrinsicInst>(&*I);
1283:     if (II && llvm::isConvergenceControlIntrinsic(II->getIntrinsicID())) {
1284:       return II;
1285:     }
1286:   }
1287:   llvm_unreachable("Convergence token should have been emitted.");
1288:   return nullptr;
1289: }
1290: 
1291: class OpaqueValueVisitor : public RecursiveASTVisitor<OpaqueValueVisitor> {
1292: public:
1293:   llvm::SmallVector<OpaqueValueExpr *, 8> OVEs;
1294:   llvm::SmallPtrSet<OpaqueValueExpr *, 8> Visited;
1295:   OpaqueValueVisitor() {}
1296: 
1297:   bool VisitHLSLOutArgExpr(HLSLOutArgExpr *) {
1298:     // These need to be bound in CodeGenFunction::EmitHLSLOutArgLValues
1299:     // or CodeGenFunction::EmitHLSLOutArgExpr. If they are part of this
1300:     // traversal, the temporary containing the copy out will not have
```
- **EN**: This block introduces declarations such as `OpaqueValueVisitor`; defines callable entry points like `OpaqueValueVisitor`, `VisitHLSLOutArgExpr`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `OpaqueValueVisitor` 的声明；定义可调用入口，例如 `OpaqueValueVisitor`, `VisitHLSLOutArgExpr`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1301-1320
```cpp
1301:     // been created yet.
1302:     return false;
1303:   }
1304: 
1305:   bool VisitOpaqueValueExpr(OpaqueValueExpr *E) {
1306:     // Traverse the source expression first.
1307:     if (E->getSourceExpr())
1308:       TraverseStmt(E->getSourceExpr());
1309: 
1310:     // Then add this OVE if we haven't seen it before.
1311:     if (Visited.insert(E).second)
1312:       OVEs.push_back(E);
1313: 
1314:     return true;
1315:   }
1316: };
1317: 
1318: void CGHLSLRuntime::emitInitListOpaqueValues(CodeGenFunction &CGF,
1319:                                              InitListExpr *E) {
1320: 
```
- **EN**: This block defines callable entry points like `VisitOpaqueValueExpr`, `emitInitListOpaqueValues`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `VisitOpaqueValueExpr`, `emitInitListOpaqueValues`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1321-1340
```cpp
1321:   typedef CodeGenFunction::OpaqueValueMappingData OpaqueValueMappingData;
1322:   OpaqueValueVisitor Visitor;
1323:   Visitor.TraverseStmt(E);
1324:   for (auto *OVE : Visitor.OVEs) {
1325:     if (CGF.isOpaqueValueEmitted(OVE))
1326:       continue;
1327:     if (OpaqueValueMappingData::shouldBindAsLValue(OVE)) {
1328:       LValue LV = CGF.EmitLValue(OVE->getSourceExpr());
1329:       OpaqueValueMappingData::bind(CGF, OVE, LV);
1330:     } else {
1331:       RValue RV = CGF.EmitAnyExpr(OVE->getSourceExpr());
1332:       OpaqueValueMappingData::bind(CGF, OVE, RV);
1333:     }
1334:   }
1335: }
1336: 
1337: std::optional<LValue> CGHLSLRuntime::emitResourceArraySubscriptExpr(
1338:     const ArraySubscriptExpr *ArraySubsExpr, CodeGenFunction &CGF) {
1339:   assert((ArraySubsExpr->getType()->isHLSLResourceRecord() ||
1340:           ArraySubsExpr->getType()->isHLSLResourceRecordArray()) &&
```
- **EN**: This block defines callable entry points like `bind`, `emitResourceArraySubscriptExpr`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `bind`, `emitResourceArraySubscriptExpr`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1341-1360
```cpp
1341:          "expected resource array subscript expression");
1342: 
1343:   // Let clang codegen handle local and static resource array subscripts,
1344:   // or when the subscript references on opaque expression (as part of
1345:   // ArrayInitLoopExpr AST node).
1346:   const VarDecl *ArrayDecl = dyn_cast_or_null<VarDecl>(
1347:       getArrayDecl(CGF.CGM.getContext(), ArraySubsExpr));
1348:   if (!ArrayDecl || !ArrayDecl->hasGlobalStorage() ||
1349:       ArrayDecl->getStorageClass() == SC_Static)
1350:     return std::nullopt;
1351: 
1352:   // get the resource array type
1353:   ASTContext &AST = ArrayDecl->getASTContext();
1354:   const Type *ResArrayTy = ArrayDecl->getType().getTypePtr();
1355:   assert(ResArrayTy->isHLSLResourceRecordArray() &&
1356:          "expected array of resource classes");
1357: 
1358:   // Iterate through all nested array subscript expressions to calculate
1359:   // the index in the flattened resource array (if this is a multi-
1360:   // dimensional array). The index is calculated as a sum of all indices
```
- **EN**: This block spells out callable entry points like `getArrayDecl`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出可调用入口的声明，例如 `getArrayDecl`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1361-1380
```cpp
1361:   // multiplied by the total size of the array at that level.
1362:   Value *Index = nullptr;
1363:   const ArraySubscriptExpr *ASE = ArraySubsExpr;
1364:   while (ASE != nullptr) {
1365:     Value *SubIndex = CGF.EmitScalarExpr(ASE->getIdx());
1366:     if (const auto *ArrayTy =
1367:             dyn_cast<ConstantArrayType>(ASE->getType().getTypePtr())) {
1368:       Value *Multiplier = llvm::ConstantInt::get(
1369:           CGM.IntTy, AST.getConstantArrayElementCount(ArrayTy));
1370:       SubIndex = CGF.Builder.CreateMul(SubIndex, Multiplier);
1371:     }
1372:     Index = Index ? CGF.Builder.CreateAdd(Index, SubIndex) : SubIndex;
1373:     ASE = dyn_cast<ArraySubscriptExpr>(ASE->getBase()->IgnoreParenImpCasts());
1374:   }
1375: 
1376:   // Find binding info for the resource array. For implicit binding
1377:   // an HLSLResourceBindingAttr should have been added by SemaHLSL.
1378:   ResourceBindingAttrs Binding(ArrayDecl);
1379:   assert(Binding.hasBinding() &&
1380:          "resource array must have a binding attribute");
```
- **EN**: This block defines callable entry points like `Binding`; uses control flow (if, while) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Binding`；通过控制流（if, while）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1381-1400
```cpp
1381: 
1382:   // Find the individual resource type.
1383:   QualType ResultTy = ArraySubsExpr->getType();
1384:   QualType ResourceTy =
1385:       ResultTy->isArrayType() ? AST.getBaseElementType(ResultTy) : ResultTy;
1386: 
1387:   // Create a temporary variable for the result, which is either going
1388:   // to be a single resource instance or a local array of resources (we need to
1389:   // return an LValue).
1390:   RawAddress TmpVar = CGF.CreateMemTempWithoutCast(ResultTy);
1391:   if (CGF.EmitLifetimeStart(TmpVar.getPointer()))
1392:     CGF.pushFullExprCleanup<CodeGenFunction::CallLifetimeEnd>(
1393:         NormalEHLifetimeMarker, TmpVar);
1394: 
1395:   AggValueSlot ValueSlot = AggValueSlot::forAddr(
1396:       TmpVar, Qualifiers(), AggValueSlot::IsDestructed_t(true),
1397:       AggValueSlot::DoesNotNeedGCBarriers, AggValueSlot::IsAliased_t(false),
1398:       AggValueSlot::DoesNotOverlap);
1399: 
1400:   // Calculate total array size (= range size).
```
- **EN**: This block spells out callable entry points like `Qualifiers`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块给出可调用入口的声明，例如 `Qualifiers`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1401-1420
```cpp
1401:   llvm::Value *Range = llvm::ConstantInt::getSigned(
1402:       CGM.IntTy, getTotalArraySize(AST, ResArrayTy));
1403: 
1404:   // If the result of the subscript operation is a single resource, call the
1405:   // constructor.
1406:   if (ResultTy == ResourceTy) {
1407:     CallArgList Args;
1408:     CXXMethodDecl *CreateMethod = lookupResourceInitMethodAndSetupArgs(
1409:         CGF.CGM, ResourceTy->getAsCXXRecordDecl(), Range, Index,
1410:         ArrayDecl->getName(), Binding, Args);
1411: 
1412:     if (!CreateMethod) {
1413:       // This can happen if someone creates an array of structs that looks like
1414:       // an HLSL resource record array but it does not have the required static
1415:       // create method. No binding will be generated for it.
1416:       assert(!ResourceTy->getAsCXXRecordDecl()->isImplicit() &&
1417:              "create method lookup should always succeed for built-in resource "
1418:              "records");
1419:       return std::nullopt;
1420:     }
```
- **EN**: This block uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1421-1440
```cpp
1421: 
1422:     callResourceInitMethod(CGF, CreateMethod, Args, ValueSlot.getAddress());
1423: 
1424:   } else {
1425:     // The result of the subscript operation is a local resource array which
1426:     // needs to be initialized.
1427:     const ConstantArrayType *ArrayTy =
1428:         cast<ConstantArrayType>(ResultTy.getTypePtr());
1429:     std::optional<llvm::Value *> EndIndex = initializeLocalResourceArray(
1430:         CGF, ResourceTy->getAsCXXRecordDecl(), ArrayTy, ValueSlot, Range, Index,
1431:         ArrayDecl->getName(), Binding, {llvm::ConstantInt::get(CGM.IntTy, 0)},
1432:         ArraySubsExpr->getExprLoc());
1433:     if (!EndIndex)
1434:       return std::nullopt;
1435:   }
1436:   return CGF.MakeAddrLValue(TmpVar, ResultTy, AlignmentSource::Decl);
1437: }
1438: 
1439: // If RHSExpr is a global resource array, initialize all of its resources and
1440: // set them into LHS. Returns false if no copy has been performed and the
```
- **EN**: This block defines callable entry points like `callResourceInitMethod`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `callResourceInitMethod`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1441-1460
```cpp
1441: // array copy should be handled by Clang codegen.
1442: bool CGHLSLRuntime::emitResourceArrayCopy(LValue &LHS, Expr *RHSExpr,
1443:                                           CodeGenFunction &CGF) {
1444:   QualType ResultTy = RHSExpr->getType();
1445:   assert(ResultTy->isHLSLResourceRecordArray() && "expected resource array");
1446: 
1447:   // Let Clang codegen handle local and static resource array copies.
1448:   const VarDecl *ArrayDecl =
1449:       dyn_cast_or_null<VarDecl>(getArrayDecl(CGF.CGM.getContext(), RHSExpr));
1450:   if (!ArrayDecl || !ArrayDecl->hasGlobalStorage() ||
1451:       ArrayDecl->getStorageClass() == SC_Static)
1452:     return false;
1453: 
1454:   // Find binding info for the resource array. For implicit binding
1455:   // the HLSLResourceBindingAttr should have been added by SemaHLSL.
1456:   ResourceBindingAttrs Binding(ArrayDecl);
1457:   assert(Binding.hasBinding() &&
1458:          "resource array must have a binding attribute");
1459: 
1460:   // Find the individual resource type.
```
- **EN**: This block defines callable entry points like `emitResourceArrayCopy`, `Binding`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitResourceArrayCopy`, `Binding`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1461-1480
```cpp
1461:   ASTContext &AST = ArrayDecl->getASTContext();
1462:   QualType ResTy = AST.getBaseElementType(ResultTy);
1463:   const auto *ResArrayTy = cast<ConstantArrayType>(ResultTy.getTypePtr());
1464: 
1465:   // Use the provided LHS for the result.
1466:   AggValueSlot ValueSlot = AggValueSlot::forAddr(
1467:       LHS.getAddress(), Qualifiers(), AggValueSlot::IsDestructed_t(true),
1468:       AggValueSlot::DoesNotNeedGCBarriers, AggValueSlot::IsAliased_t(false),
1469:       AggValueSlot::DoesNotOverlap);
1470: 
1471:   // Create Value for index and total array size (= range size).
1472:   int Size = getTotalArraySize(AST, ResArrayTy);
1473:   llvm::Value *Zero = llvm::ConstantInt::get(CGM.IntTy, 0);
1474:   llvm::Value *Range = llvm::ConstantInt::get(CGM.IntTy, Size);
1475: 
1476:   // Initialize individual resources in the array into LHS.
1477:   std::optional<llvm::Value *> EndIndex = initializeLocalResourceArray(
1478:       CGF, ResTy->getAsCXXRecordDecl(), ResArrayTy, ValueSlot, Range, Zero,
1479:       ArrayDecl->getName(), Binding, {Zero}, RHSExpr->getExprLoc());
1480:   return EndIndex.has_value();
```
- **EN**: This block defines callable entry points like `IsAliased_t`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `IsAliased_t`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 1481-1500
```cpp
1481: }
1482: 
1483: RawAddress CGHLSLRuntime::createBufferMatrixTempAddress(const LValue &LV,
1484:                                                         SourceLocation Loc,
1485:                                                         CodeGenFunction &CGF) {
1486: 
1487:   assert(LV.getType()->isConstantMatrixType() && "expected matrix type");
1488:   assert(LV.getType().getAddressSpace() == LangAS::hlsl_constant &&
1489:          "expected cbuffer matrix");
1490: 
1491:   QualType MatQualTy = LV.getType();
1492:   llvm::Type *LayoutTy = HLSLBufferLayoutBuilder(CGF.CGM).layOutType(MatQualTy);
1493:   Address SrcAddr = LV.getAddress();
1494: 
1495:   if (LayoutTy == CGF.ConvertTypeForMem(MatQualTy))
1496:     return SrcAddr;
1497: 
1498:   RawAddress DestAlloca =
1499:       CGF.CreateMemTempWithoutCast(MatQualTy, "matrix.buf.copy");
1500:   emitBufferCopy(CGF, DestAlloca, SrcAddr, MatQualTy);
```
- **EN**: This block defines callable entry points like `createBufferMatrixTempAddress`, `emitBufferCopy`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `createBufferMatrixTempAddress`, `emitBufferCopy`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1501-1520
```cpp
1501:   return DestAlloca;
1502: }
1503: 
1504: std::optional<LValue> CGHLSLRuntime::emitBufferArraySubscriptExpr(
1505:     const ArraySubscriptExpr *E, CodeGenFunction &CGF,
1506:     llvm::function_ref<llvm::Value *(bool Promote)> EmitIdxAfterBase) {
1507:   // Find the element type to index by first padding the element type per HLSL
1508:   // buffer rules, and then padding out to a 16-byte register boundary if
1509:   // necessary.
1510:   llvm::Type *LayoutTy =
1511:       HLSLBufferLayoutBuilder(CGF.CGM).layOutType(E->getType());
1512:   uint64_t LayoutSizeInBits =
1513:       CGM.getDataLayout().getTypeSizeInBits(LayoutTy).getFixedValue();
1514:   CharUnits ElementSize = CharUnits::fromQuantity(LayoutSizeInBits / 8);
1515:   CharUnits RowAlignedSize = ElementSize.alignTo(CharUnits::fromQuantity(16));
1516:   if (RowAlignedSize > ElementSize) {
1517:     llvm::Type *Padding = CGM.getTargetCodeGenInfo().getHLSLPadding(
1518:         CGM, RowAlignedSize - ElementSize);
1519:     assert(Padding && "No padding type for target?");
1520:     LayoutTy = llvm::StructType::get(CGF.getLLVMContext(), {LayoutTy, Padding},
```
- **EN**: This block defines callable entry points like `emitBufferArraySubscriptExpr`, `HLSLBufferLayoutBuilder`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitBufferArraySubscriptExpr`, `HLSLBufferLayoutBuilder`；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1521-1540
```cpp
1521:                                      /*isPacked=*/true);
1522:   }
1523: 
1524:   // If the layout type doesn't introduce any padding, we don't need to do
1525:   // anything special.
1526:   llvm::Type *OrigTy = CGF.CGM.getTypes().ConvertTypeForMem(E->getType());
1527:   if (LayoutTy == OrigTy)
1528:     return std::nullopt;
1529: 
1530:   LValueBaseInfo EltBaseInfo;
1531:   TBAAAccessInfo EltTBAAInfo;
1532: 
1533:   // Index into the object as-if we have an array of the padded element type,
1534:   // and then dereference the element itself to avoid reading padding that may
1535:   // be past the end of the in-memory object.
1536:   SmallVector<llvm::Value *, 2> Indices;
1537:   llvm::Value *Idx = EmitIdxAfterBase(/*Promote*/ true);
1538:   Indices.push_back(Idx);
1539:   Indices.push_back(llvm::ConstantInt::get(CGF.Int32Ty, 0));
1540: 
```
- **EN**: This block uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1541-1560
```cpp
1541:   if (CGF.getLangOpts().EmitLogicalPointer) {
1542:     // The fact that we emit an array-to-pointer decay might be an oversight,
1543:     // but for now, we simply ignore it (see #179951).
1544:     const CastExpr *CE = cast<CastExpr>(E->getBase());
1545:     assert(CE->getCastKind() == CastKind::CK_ArrayToPointerDecay);
1546: 
1547:     LValue LV = CGF.EmitLValue(CE->getSubExpr());
1548:     Address Addr = LV.getAddress();
1549:     LayoutTy = llvm::ArrayType::get(
1550:         LayoutTy,
1551:         cast<llvm::ArrayType>(Addr.getElementType())->getNumElements());
1552:     auto *GEP = cast<StructuredGEPInst>(CGF.Builder.CreateStructuredGEP(
1553:         LayoutTy, Addr.emitRawPointer(CGF), Indices, "cbufferidx"));
1554:     Addr =
1555:         Address(GEP, GEP->getResultElementType(), RowAlignedSize, KnownNonNull);
1556:     return CGF.MakeAddrLValue(Addr, E->getType(), EltBaseInfo, EltTBAAInfo);
1557:   }
1558: 
1559:   Address Addr =
1560:       CGF.EmitPointerWithAlignment(E->getBase(), &EltBaseInfo, &EltTBAAInfo);
```
- **EN**: This block defines callable entry points like `Address`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `Address`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1561-1580
```cpp
1561:   llvm::Value *GEP = CGF.Builder.CreateGEP(LayoutTy, Addr.emitRawPointer(CGF),
1562:                                            Indices, "cbufferidx");
1563:   Addr = Address(GEP, Addr.getElementType(), RowAlignedSize, KnownNonNull);
1564:   return CGF.MakeAddrLValue(Addr, E->getType(), EltBaseInfo, EltTBAAInfo);
1565: }
1566: 
1567: std::optional<LValue>
1568: CGHLSLRuntime::emitResourceMemberExpr(CodeGenFunction &CGF,
1569:                                       const MemberExpr *ME) {
1570:   assert((ME->getType()->isHLSLResourceRecord() ||
1571:           ME->getType()->isHLSLResourceRecordArray()) &&
1572:          "expected resource member expression");
1573: 
1574:   if (ME->getType()->isHLSLResourceRecordArray()) {
1575:     // FIXME: Handle member access of the whole array of resources
1576:     // (llvm/llvm-project#187087). Access to individual resource array elements
1577:     // is already handled in emitResourceArraySubscriptExpr.
1578:     return std::nullopt;
1579:   }
1580: 
```
- **EN**: This block defines callable entry points like `emitResourceMemberExpr`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitResourceMemberExpr`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1581-1600
```cpp
1581:   const VarDecl *ResourceVD =
1582:       findAssociatedResourceDeclForStruct(CGF.CGM.getContext(), ME);
1583:   if (!ResourceVD)
1584:     return std::nullopt;
1585: 
1586:   GlobalVariable *ResGV =
1587:       cast<GlobalVariable>(CGM.GetAddrOfGlobalVar(ResourceVD));
1588:   const DataLayout &DL = CGM.getDataLayout();
1589:   llvm::Type *Ty = ResGV->getValueType();
1590:   CharUnits Align = CharUnits::fromQuantity(DL.getABITypeAlign(Ty));
1591:   Address Addr = Address(ResGV, Ty, Align);
1592:   LValue LV = LValue::MakeAddr(Addr, ME->getType(), CGM.getContext(),
1593:                                LValueBaseInfo(AlignmentSource::Type),
1594:                                CGM.getTBAAAccessInfo(ME->getType()));
1595:   return LV;
1596: }
1597: 
1598: namespace {
1599: /// Utility for emitting copies following the HLSL buffer layout rules (ie,
1600: /// copying out of a cbuffer).
```
- **EN**: This block defines callable entry points like `findAssociatedResourceDeclForStruct`, `LValueBaseInfo`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `findAssociatedResourceDeclForStruct`, `LValueBaseInfo`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1601-1620
```cpp
1601: class HLSLBufferCopyEmitter {
1602:   CodeGenFunction &CGF;
1603:   Address DstPtr;
1604:   Address SrcPtr;
1605:   llvm::Type *LayoutTy = nullptr;
1606: 
1607:   SmallVector<llvm::Value *> CurStoreIndices;
1608:   SmallVector<llvm::Value *> CurLoadIndices;
1609: 
1610:   // Creates & returns either a structured.gep or a ptradd/gep depending on
1611:   // langopts.
1612:   llvm::Value *emitAccessChain(llvm::Type *BaseTy, llvm::Value *Base,
1613:                                ArrayRef<llvm::Value *> Indices) {
1614:     bool EmitLogical = CGF.getLangOpts().EmitLogicalPointer;
1615:     if (EmitLogical)
1616:       return CGF.Builder.CreateAccessChain(EmitLogical, BaseTy, Base, Indices);
1617: 
1618:     llvm::SmallVector<llvm::Value *> GEPIndices;
1619:     GEPIndices.reserve(Indices.size() + 1);
1620:     GEPIndices.push_back(llvm::ConstantInt::get(CGF.IntTy, 0));
```
- **EN**: This block introduces declarations such as `HLSLBufferCopyEmitter`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块给出诸如 `HLSLBufferCopyEmitter` 的声明；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1621-1640
```cpp
1621:     GEPIndices.append(Indices.begin(), Indices.end());
1622:     return CGF.Builder.CreateAccessChain(EmitLogical, BaseTy, Base, GEPIndices);
1623:   }
1624: 
1625:   bool isBufferLayoutArray(llvm::StructType *ST) {
1626:     // A buffer layout array is a struct with two elements: the padded array,
1627:     // and the last element. That is, is should look something like this:
1628:     //
1629:     //   { [%n x { %type, %padding }], %type }
1630:     //
1631:     if (!ST || ST->getNumElements() != 2)
1632:       return false;
1633: 
1634:     auto *PaddedEltsTy = dyn_cast<llvm::ArrayType>(ST->getElementType(0));
1635:     if (!PaddedEltsTy)
1636:       return false;
1637: 
1638:     auto *PaddedTy = dyn_cast<llvm::StructType>(PaddedEltsTy->getElementType());
1639:     if (!PaddedTy || PaddedTy->getNumElements() != 2)
1640:       return false;
```
- **EN**: This block defines callable entry points like `isBufferLayoutArray`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `isBufferLayoutArray`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1641-1660
```cpp
1641: 
1642:     if (!CGF.CGM.getTargetCodeGenInfo().isHLSLPadding(
1643:             PaddedTy->getElementType(1)))
1644:       return false;
1645: 
1646:     llvm::Type *ElementTy = ST->getElementType(1);
1647:     if (PaddedTy->getElementType(0) != ElementTy)
1648:       return false;
1649:     return true;
1650:   }
1651: 
1652:   void emitBufferLayoutCopy(Value *Src, llvm::StructType *SrcTy, Value *Dst,
1653:                             llvm::ArrayType *DstTy) {
1654:     // Those assumptions are checked by isBufferLayoutArray.
1655:     auto *SrcPaddedArrayTy = cast<llvm::ArrayType>(SrcTy->getElementType(0));
1656:     assert(SrcPaddedArrayTy->getNumElements() + 1 == DstTy->getNumElements());
1657:     assert(cast<llvm::StructType>(SrcPaddedArrayTy->getElementType())
1658:                ->getElementType(0) == SrcTy->getElementType(1));
1659: 
1660:     auto *SrcDataTy = SrcTy->getElementType(1);
```
- **EN**: This block defines callable entry points like `emitBufferLayoutCopy`; uses control flow (if) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitBufferLayoutCopy`；通过控制流（if）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1661-1680
```cpp
1661:     auto Zero = llvm::ConstantInt::get(CGF.IntTy, 0);
1662: 
1663:     for (unsigned I = 0; I < SrcPaddedArrayTy->getNumElements(); ++I) {
1664:       auto Index = llvm::ConstantInt::get(CGF.IntTy, I);
1665:       auto *SrcElt = emitAccessChain(SrcTy, Src, {Zero, Index, Zero});
1666:       auto *DstElt = emitAccessChain(DstTy, Dst, {Index});
1667:       emitElementCopy(SrcElt, SrcDataTy, DstElt, DstTy->getElementType());
1668:     }
1669: 
1670:     auto *SrcElt =
1671:         emitAccessChain(SrcTy, Src, {llvm::ConstantInt::get(CGF.IntTy, 1)});
1672:     auto *DstElt = emitAccessChain(
1673:         DstTy, Dst,
1674:         {llvm::ConstantInt::get(CGF.IntTy, DstTy->getNumElements() - 1)});
1675:     emitElementCopy(SrcElt, SrcDataTy, DstElt, DstTy->getElementType());
1676:   }
1677: 
1678:   void emitCopy(Value *Src, llvm::StructType *SrcTy, Value *Dst,
1679:                 llvm::Type *DstTy) {
1680:     if (isBufferLayoutArray(SrcTy))
```
- **EN**: This block defines callable entry points like `emitElementCopy`, `emitCopy`; uses control flow (if, for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitElementCopy`, `emitCopy`；通过控制流（if, for）细化 LLVM IR 生成 行为。

### Lines 1681-1700
```cpp
1681:       return emitBufferLayoutCopy(Src, SrcTy, Dst,
1682:                                   cast<llvm::ArrayType>(DstTy));
1683: 
1684:     unsigned SrcIndex = 0;
1685:     unsigned DstIndex = 0;
1686: 
1687:     auto *DstST = cast<llvm::StructType>(DstTy);
1688:     while (SrcIndex < SrcTy->getNumElements() &&
1689:            DstIndex < DstST->getNumElements()) {
1690:       if (CGF.CGM.getTargetCodeGenInfo().isHLSLPadding(
1691:               SrcTy->getElementType(SrcIndex))) {
1692:         SrcIndex += 1;
1693:         continue;
1694:       }
1695: 
1696:       if (CGF.CGM.getTargetCodeGenInfo().isHLSLPadding(
1697:               DstST->getElementType(DstIndex))) {
1698:         DstIndex += 1;
1699:         continue;
1700:       }
```
- **EN**: This block defines callable entry points like `emitBufferLayoutCopy`; uses control flow (if, while) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitBufferLayoutCopy`；通过控制流（if, while）细化 LLVM IR 生成 行为。

### Lines 1701-1720
```cpp
1701: 
1702:       auto *SrcElt = emitAccessChain(
1703:           SrcTy, Src, {llvm::ConstantInt::get(CGF.IntTy, SrcIndex)});
1704:       auto *DstElt = emitAccessChain(
1705:           DstTy, Dst, {llvm::ConstantInt::get(CGF.IntTy, DstIndex)});
1706:       emitElementCopy(SrcElt, SrcTy->getElementType(SrcIndex), DstElt,
1707:                       DstST->getElementType(DstIndex));
1708:       DstIndex += 1;
1709:       SrcIndex += 1;
1710:     }
1711:   }
1712: 
1713:   void emitCopy(Value *Src, llvm::ArrayType *SrcTy, Value *Dst,
1714:                 llvm::Type *DstTy) {
1715:     for (unsigned I = 0, E = SrcTy->getNumElements(); I < E; ++I) {
1716:       auto *SrcElt =
1717:           emitAccessChain(SrcTy, Src, {llvm::ConstantInt::get(CGF.IntTy, I)});
1718:       auto *DstElt =
1719:           emitAccessChain(DstTy, Dst, {llvm::ConstantInt::get(CGF.IntTy, I)});
1720:       emitElementCopy(SrcElt, SrcTy->getElementType(), DstElt,
```
- **EN**: This block defines callable entry points like `emitElementCopy`, `emitCopy`; uses control flow (for) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitElementCopy`, `emitCopy`；通过控制流（for）细化 LLVM IR 生成 行为。

### Lines 1721-1740
```cpp
1721:                       cast<llvm::ArrayType>(DstTy)->getElementType());
1722:     }
1723:   }
1724: 
1725:   void emitElementCopy(Value *Src, llvm::Type *SrcTy, Value *Dst,
1726:                        llvm::Type *DstTy) {
1727:     if (auto *AT = dyn_cast<llvm::ArrayType>(SrcTy))
1728:       return emitCopy(Src, AT, Dst, DstTy);
1729:     if (auto *ST = dyn_cast<llvm::StructType>(SrcTy))
1730:       return emitCopy(Src, ST, Dst, DstTy);
1731: 
1732:     // When we have a scalar or vector element we can emit the copy.
1733:     CharUnits SrcAlign =
1734:         CharUnits::fromQuantity(CGF.CGM.getDataLayout().getABITypeAlign(SrcTy));
1735:     CharUnits DstAlign =
1736:         CharUnits::fromQuantity(CGF.CGM.getDataLayout().getABITypeAlign(DstTy));
1737:     Address SrcAddr(Src, SrcTy, SrcAlign);
1738:     Address DstAddr(Dst, DstTy, DstAlign);
1739:     llvm::Value *Load = CGF.Builder.CreateLoad(SrcAddr, "cbuf.load");
1740:     CGF.Builder.CreateStore(Load, DstAddr);
```
- **EN**: This block defines callable entry points like `emitElementCopy`, `fromQuantity`, `SrcAddr`, `DstAddr`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `emitElementCopy`, `fromQuantity`, `SrcAddr`, `DstAddr`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 1741-1760
```cpp
1741:   }
1742: 
1743: public:
1744:   HLSLBufferCopyEmitter(CodeGenFunction &CGF, Address DstPtr, Address SrcPtr)
1745:       : CGF(CGF), DstPtr(DstPtr), SrcPtr(SrcPtr) {}
1746: 
1747:   bool emitCopy(QualType CType) {
1748:     LayoutTy = HLSLBufferLayoutBuilder(CGF.CGM).layOutType(CType);
1749: 
1750:     // TODO: We should be able to fall back to a regular memcpy if the layout
1751:     // type doesn't have any padding, but that runs into issues in the backend
1752:     // currently.
1753:     //
1754:     // See https://github.com/llvm/wg-hlsl/issues/351
1755:     emitElementCopy(SrcPtr.getBasePointer(), LayoutTy, DstPtr.getBasePointer(),
1756:                     DstPtr.getElementType());
1757:     return true;
1758:   }
1759: };
1760: } // namespace
```
- **EN**: This block defines callable entry points like `HLSLBufferCopyEmitter`, `emitCopy`, `emitElementCopy`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `HLSLBufferCopyEmitter`, `emitCopy`, `emitElementCopy`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 1761-1780
```cpp
1761: 
1762: bool CGHLSLRuntime::emitBufferCopy(CodeGenFunction &CGF, Address DstPtr,
1763:                                    Address SrcPtr, QualType CType) {
1764:   return HLSLBufferCopyEmitter(CGF, DstPtr, SrcPtr).emitCopy(CType);
1765: }
1766: 
1767: LValue CGHLSLRuntime::emitBufferMemberExpr(CodeGenFunction &CGF,
1768:                                            const MemberExpr *E) {
1769:   LValue Base =
1770:       CGF.EmitCheckedLValue(E->getBase(), CodeGenFunction::TCK_MemberAccess);
1771:   auto *Field = dyn_cast<FieldDecl>(E->getMemberDecl());
1772:   assert(Field && "Unexpected access into HLSL buffer");
1773: 
1774:   const RecordDecl *Rec = Field->getParent();
1775: 
1776:   // Work out the buffer layout type to index into.
1777:   QualType RecType = CGM.getContext().getCanonicalTagType(Rec);
1778:   assert(RecType->isStructureOrClassType() && "Invalid type in HLSL buffer");
1779:   // Since this is a member of an object in the buffer and not the buffer's
1780:   // struct/class itself, we shouldn't have any offsets on the members we need
```
- **EN**: This block defines callable entry points like `emitBufferCopy`, `HLSLBufferCopyEmitter`, `emitBufferMemberExpr`; returns or forwards computed values for the surrounding LLVM IR emission logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `emitBufferCopy`, `HLSLBufferCopyEmitter`, `emitBufferMemberExpr`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 1781-1800
```cpp
1781:   // to contend with.
1782:   CGHLSLOffsetInfo EmptyOffsets;
1783:   llvm::StructType *LayoutTy = HLSLBufferLayoutBuilder(CGM).layOutStruct(
1784:       RecType->getAsCanonical<RecordType>(), EmptyOffsets);
1785: 
1786:   // Get the field index for the layout struct, accounting for padding.
1787:   unsigned FieldIdx =
1788:       CGM.getTypes().getCGRecordLayout(Rec).getLLVMFieldNo(Field);
1789:   assert(FieldIdx < LayoutTy->getNumElements() &&
1790:          "Layout struct is smaller than member struct");
1791:   unsigned Skipped = 0;
1792:   for (unsigned I = 0; I <= FieldIdx;) {
1793:     llvm::Type *ElementTy = LayoutTy->getElementType(I + Skipped);
1794:     if (CGF.CGM.getTargetCodeGenInfo().isHLSLPadding(ElementTy))
1795:       ++Skipped;
1796:     else
1797:       ++I;
1798:   }
1799:   FieldIdx += Skipped;
1800:   assert(FieldIdx < LayoutTy->getNumElements() && "Access out of bounds");
```
- **EN**: This block introduces declarations such as `is`; uses control flow (if, for) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块给出诸如 `is` 的声明；通过控制流（if, for）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量。

### Lines 1801-1820
```cpp
1801: 
1802:   // Now index into the struct, making sure that the type we return is the
1803:   // buffer layout type rather than the original type in the AST.
1804:   QualType FieldType = Field->getType();
1805:   llvm::Type *FieldLLVMTy = CGM.getTypes().ConvertTypeForMem(FieldType);
1806:   CharUnits Align = CharUnits::fromQuantity(
1807:       CGF.CGM.getDataLayout().getABITypeAlign(FieldLLVMTy));
1808: 
1809:   Value *Ptr = CGF.getLangOpts().EmitLogicalPointer
1810:                    ? CGF.Builder.CreateStructuredGEP(
1811:                          LayoutTy, Base.getPointer(CGF),
1812:                          llvm::ConstantInt::get(CGM.IntTy, FieldIdx))
1813:                    : CGF.Builder.CreateStructGEP(LayoutTy, Base.getPointer(CGF),
1814:                                                  FieldIdx, Field->getName());
1815:   Address Addr(Ptr, FieldLLVMTy, Align, KnownNonNull);
1816: 
1817:   LValue LV = LValue::MakeAddr(Addr, FieldType, CGM.getContext(),
1818:                                LValueBaseInfo(AlignmentSource::Type),
1819:                                CGM.getTBAAAccessInfo(FieldType));
1820:   LV.getQuals().addCVRQualifiers(Base.getVRQualifiers());
```
- **EN**: This block spells out callable entry points like `get`, `Addr`, `LValueBaseInfo`.
- **CN**: 该代码块给出可调用入口的声明，例如 `get`, `Addr`, `LValueBaseInfo`。

### Lines 1821-1823
```cpp
1821: 
1822:   return LV;
1823: }
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

## Key Concepts / 关键概念

- **CGM**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **CGF**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Index**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **AST**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **Semantic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **getType**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGHLSLRuntime.h`, `CGDebugInfo.h`, `CGRecordLayout.h`, `CodeGenFunction.h`, `CodeGenModule.h`, `HLSLBufferLayoutBuilder.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/AST/ASTContext.h`, `clang/AST/Attr.h`, `clang/AST/Decl.h`, `clang/AST/Expr.h`, `clang/AST/HLSLResource.h`, `clang/AST/RecursiveASTVisitor.h`, `clang/AST/Type.h`, `clang/Basic/DiagnosticFrontend.h`, and 1 more
- **LLVM libraries / LLVM 库**: `llvm/ADT/DenseMap.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/Frontend/HLSL/RootSignatureMetadata.h`, `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/GlobalVariable.h`, and 9 more
- **Other headers / 其他头文件**: `cstdint`, `optional`
