# AMDGPU.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetBuiltins/AMDGPU.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Implements builtin handling and lowering logic for the AMDGPU backend.
- **Purpose (CN) / 目的（中文）**: 实现 AMDGPU 后端的内建函数处理与降级逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```cpp
 1: //===------- AMDCPU.cpp - Emit LLVM Code for builtins ---------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This contains code to emit Builtin calls as LLVM code.
10: //
11: //===----------------------------------------------------------------------===//
12: 
13: #include "CGBuiltin.h"
14: #include "CodeGenFunction.h"
15: #include "TargetInfo.h"
16: #include "clang/Basic/DiagnosticFrontend.h"
17: #include "clang/Basic/SyncScope.h"
18: #include "clang/Basic/TargetBuiltins.h"
19: #include "llvm/Analysis/ValueTracking.h"
20: #include "llvm/CodeGen/MachineFunction.h"
21: #include "llvm/IR/IntrinsicsAMDGPU.h"
22: #include "llvm/IR/IntrinsicsR600.h"
23: #include "llvm/IR/IntrinsicsSPIRV.h"
24: #include "llvm/IR/MemoryModelRelaxationAnnotations.h"
25: #include "llvm/Support/AMDGPUAddrSpace.h"
```
- **EN**: This block imports local CodeGen headers `CGBuiltin.h`, `CodeGenFunction.h`, `TargetInfo.h`; Clang headers `clang/Basic/DiagnosticFrontend.h`, `clang/Basic/SyncScope.h`, `clang/Basic/TargetBuiltins.h`; LLVM headers `llvm/Analysis/ValueTracking.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/IntrinsicsAMDGPU.h`, and 4 more; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGBuiltin.h`, `CodeGenFunction.h`, `TargetInfo.h`；Clang 头文件 `clang/Basic/DiagnosticFrontend.h`, `clang/Basic/SyncScope.h`, `clang/Basic/TargetBuiltins.h`；LLVM 头文件 `llvm/Analysis/ValueTracking.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/IntrinsicsAMDGPU.h`, and 4 more；包含影响本编译单元构建方式的预处理结构。

### Lines 26-50
```cpp
26: #include "llvm/Support/AtomicOrdering.h"
27: 
28: using namespace clang;
29: using namespace CodeGen;
30: using namespace llvm;
31: 
32: namespace {
33: 
34: // Has second type mangled argument.
35: static Value *
36: emitBinaryExpMaybeConstrainedFPBuiltin(CodeGenFunction &CGF, const CallExpr *E,
37:                                        Intrinsic::ID IntrinsicID,
38:                                        Intrinsic::ID ConstrainedIntrinsicID) {
39:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
40:   llvm::Value *Src1 = CGF.EmitScalarExpr(E->getArg(1));
41: 
42:   CodeGenFunction::CGFPOptionsRAII FPOptsRAII(CGF, E);
43:   if (CGF.Builder.getIsFPConstrained()) {
44:     Function *F = CGF.CGM.getIntrinsic(ConstrainedIntrinsicID,
45:                                        {Src0->getType(), Src1->getType()});
46:     return CGF.Builder.CreateConstrainedFPCall(F, {Src0, Src1});
47:   }
48: 
49:   Function *F =
50:       CGF.CGM.getIntrinsic(IntrinsicID, {Src0->getType(), Src1->getType()});
```
- **EN**: This block imports LLVM headers `llvm/Support/AtomicOrdering.h`; opens or references namespaces `clang`, `CodeGen`, `llvm`; defines callable entry points like `emitBinaryExpMaybeConstrainedFPBuiltin`, `FPOptsRAII`; uses control flow (if) to specialize target builtin lowering; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/Support/AtomicOrdering.h`；打开或引用命名空间 `clang`, `CodeGen`, `llvm`；定义可调用入口，例如 `emitBinaryExpMaybeConstrainedFPBuiltin`, `FPOptsRAII`；通过控制流（if）细化 目标内建函数降级 行为；包含影响本编译单元构建方式的预处理结构。

### Lines 51-75
```cpp
51:   return CGF.Builder.CreateCall(F, {Src0, Src1});
52: }
53: 
54: // If \p E is not null pointer, insert address space cast to match return
55: // type of \p E if necessary.
56: Value *EmitAMDGPUDispatchPtr(CodeGenFunction &CGF,
57:                              const CallExpr *E = nullptr) {
58:   auto *F = CGF.CGM.getIntrinsic(Intrinsic::amdgcn_dispatch_ptr);
59:   auto *Call = CGF.Builder.CreateCall(F);
60:   if (!E)
61:     return Call;
62:   QualType BuiltinRetType = E->getType();
63:   auto *RetTy = cast<llvm::PointerType>(CGF.ConvertType(BuiltinRetType));
64:   if (RetTy == Call->getType())
65:     return Call;
66:   return CGF.Builder.CreateAddrSpaceCast(Call, RetTy);
67: }
68: 
69: Value *EmitAMDGPUImplicitArgPtr(CodeGenFunction &CGF) {
70:   auto *F = CGF.CGM.getIntrinsic(Intrinsic::amdgcn_implicitarg_ptr);
71:   auto *Call = CGF.Builder.CreateCall(F);
72:   Call->addRetAttr(
73:       Attribute::getWithDereferenceableBytes(Call->getContext(), 256));
74:   Call->addRetAttr(Attribute::getWithAlignment(Call->getContext(), Align(8)));
75:   return Call;
```
- **EN**: This block defines callable entry points like `getWithDereferenceableBytes`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getWithDereferenceableBytes`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 76-100
```cpp
 76: }
 77: 
 78: static llvm::Intrinsic::ID getAMDGPUWorkGroupID(CodeGenFunction &CGF,
 79:                                                 unsigned Index) {
 80:   switch (Index) {
 81:   case 0:
 82:     return llvm::Intrinsic::amdgcn_workgroup_id_x;
 83:   case 1:
 84:     return llvm::Intrinsic::amdgcn_workgroup_id_y;
 85:   case 2:
 86:     return llvm::Intrinsic::amdgcn_workgroup_id_z;
 87:   default:
 88:     llvm_unreachable("unhandled index");
 89:   }
 90: }
 91: 
 92: static void setNoundefInvariantLoad(llvm::LoadInst *Ld) {
 93:   Ld->setMetadata(llvm::LLVMContext::MD_noundef,
 94:                   llvm::MDNode::get(Ld->getContext(), {}));
 95:   Ld->setMetadata(llvm::LLVMContext::MD_invariant_load,
 96:                   llvm::MDNode::get(Ld->getContext(), {}));
 97: }
 98: 
 99: static void addMaxWorkGroupSizeRangeMetadata(CodeGenFunction &CGF,
100:                                              llvm::LoadInst *GroupSize) {
```
- **EN**: This block defines callable entry points like `getAMDGPUWorkGroupID`, `setNoundefInvariantLoad`, `addMaxWorkGroupSizeRangeMetadata`; uses control flow (switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getAMDGPUWorkGroupID`, `setNoundefInvariantLoad`, `addMaxWorkGroupSizeRangeMetadata`；通过控制流（switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 101-125
```cpp
101:   llvm::MDBuilder MDHelper(CGF.getLLVMContext());
102:   llvm::MDNode *RNode = MDHelper.createRange(
103:       APInt(16, 1), APInt(16, CGF.getTarget().getMaxOpenCLWorkGroupSize() + 1));
104:   GroupSize->setMetadata(llvm::LLVMContext::MD_range, RNode);
105:   setNoundefInvariantLoad(GroupSize);
106: }
107: 
108: static Value *emitAMDGPUWorkGroupSizeV5(CodeGenFunction &CGF, unsigned Index) {
109:   llvm::Value *ImplicitArgPtr = EmitAMDGPUImplicitArgPtr(CGF);
110: 
111:   // offsetof(amdhsa_implicit_kernarg_v5, block_count[Index])
112:   unsigned BlockCountOffset = 0 + Index * 4;
113:   // offsetof(amdhsa_implicit_kernarg_v5, group_size[Index])
114:   unsigned GroupSizeOffset = 12 + Index * 2;
115:   // offsetof(amdhsa_implicit_kernarg_v5, remainder[Index])
116:   unsigned RemainderOffset = 18 + Index * 2;
117: 
118:   if (CGF.CGM.getLangOpts().OffloadUniformBlock) {
119:     // Indexing the implicit kernarg segment.
120:     llvm::Value *GroupSizeGEP = CGF.Builder.CreateConstInBoundsGEP1_64(
121:         CGF.Int8Ty, ImplicitArgPtr, GroupSizeOffset);
122:     llvm::LoadInst *GroupSize = CGF.Builder.CreateLoad(
123:         Address(GroupSizeGEP, CGF.Int16Ty, CharUnits::fromQuantity(2)));
124: 
125:     addMaxWorkGroupSizeRangeMetadata(CGF, GroupSize);
```
- **EN**: This block defines callable entry points like `MDHelper`, `APInt`, `setNoundefInvariantLoad`, `Address`, `addMaxWorkGroupSizeRangeMetadata`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `MDHelper`, `APInt`, `setNoundefInvariantLoad`, `Address`, `addMaxWorkGroupSizeRangeMetadata`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 126-150
```cpp
126: 
127:     return CGF.Builder.CreateZExt(GroupSize, CGF.Int32Ty);
128:   }
129: 
130:   llvm::Value *BlockCountGEP = CGF.Builder.CreateConstGEP1_64(
131:       CGF.Int8Ty, ImplicitArgPtr, BlockCountOffset);
132:   llvm::LoadInst *BlockCount = CGF.Builder.CreateLoad(
133:       Address(BlockCountGEP, CGF.Int32Ty, CharUnits::fromQuantity(4)));
134:   setNoundefInvariantLoad(BlockCount);
135: 
136:   llvm::Value *WorkgroupID =
137:       CGF.Builder.CreateIntrinsic(getAMDGPUWorkGroupID(CGF, Index), {});
138:   llvm::Value *IsFull = CGF.Builder.CreateICmpULT(WorkgroupID, BlockCount);
139: 
140:   llvm::Value *StructOffset = CGF.Builder.CreateSelect(
141:       IsFull, ConstantInt::get(CGF.Int32Ty, GroupSizeOffset),
142:       ConstantInt::get(CGF.Int32Ty, RemainderOffset));
143: 
144:   llvm::Value *SizeGEP =
145:       CGF.Builder.CreateInBoundsGEP(CGF.Int8Ty, ImplicitArgPtr, StructOffset);
146:   llvm::LoadInst *Size = CGF.Builder.CreateLoad(
147:       Address(SizeGEP, CGF.Int16Ty, CharUnits::fromQuantity(2)));
148:   addMaxWorkGroupSizeRangeMetadata(CGF, Size);
149:   setNoundefInvariantLoad(Size);
150: 
```
- **EN**: This block defines callable entry points like `Address`, `setNoundefInvariantLoad`, `get`, `addMaxWorkGroupSizeRangeMetadata`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `Address`, `setNoundefInvariantLoad`, `get`, `addMaxWorkGroupSizeRangeMetadata`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 151-175
```cpp
151:   return CGF.Builder.CreateZExt(Size, CGF.Int32Ty);
152: }
153: 
154: static Value *emitAMDGPUWorkGroupSizeV4(CodeGenFunction &CGF, unsigned Index) {
155:   llvm::Value *DispatchPtr = EmitAMDGPUDispatchPtr(CGF);
156: 
157:   // Indexing the HSA kernel_dispatch_packet struct.
158:   llvm::Value *GroupSizeGEP = CGF.Builder.CreateConstInBoundsGEP1_64(
159:       CGF.Int8Ty, DispatchPtr, 4 + Index * 2);
160:   llvm::LoadInst *GroupSizeLD = CGF.Builder.CreateLoad(
161:       Address(GroupSizeGEP, CGF.Int16Ty, CharUnits::fromQuantity(2)));
162: 
163:   addMaxWorkGroupSizeRangeMetadata(CGF, GroupSizeLD);
164: 
165:   llvm::Value *GroupSize = CGF.Builder.CreateZExt(GroupSizeLD, CGF.Int32Ty);
166: 
167:   if (CGF.CGM.getLangOpts().OffloadUniformBlock)
168:     return GroupSize;
169: 
170:   llvm::Value *WorkgroupID =
171:       CGF.Builder.CreateIntrinsic(getAMDGPUWorkGroupID(CGF, Index), {});
172: 
173:   llvm::Value *GridSizeGEP = CGF.Builder.CreateConstInBoundsGEP1_64(
174:       CGF.Int8Ty, DispatchPtr, 12 + Index * 4);
175:   llvm::LoadInst *GridSize = CGF.Builder.CreateLoad(
```
- **EN**: This block defines callable entry points like `Address`, `addMaxWorkGroupSizeRangeMetadata`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `Address`, `addMaxWorkGroupSizeRangeMetadata`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 176-200
```cpp
176:       Address(GridSizeGEP, CGF.Int32Ty, CharUnits::fromQuantity(4)));
177: 
178:   llvm::MDBuilder MDB(CGF.getLLVMContext());
179: 
180:   // Known non-zero.
181:   GridSize->setMetadata(llvm::LLVMContext::MD_range,
182:                         MDB.createRange(APInt(32, 1), APInt::getZero(32)));
183:   GridSize->setMetadata(llvm::LLVMContext::MD_invariant_load,
184:                         llvm::MDNode::get(CGF.getLLVMContext(), {}));
185: 
186:   llvm::Value *Mul = CGF.Builder.CreateMul(WorkgroupID, GroupSize);
187:   llvm::Value *Remainder = CGF.Builder.CreateSub(GridSize, Mul);
188: 
189:   llvm::Value *IsPartial = CGF.Builder.CreateICmpULT(Remainder, GroupSize);
190: 
191:   return CGF.Builder.CreateSelect(IsPartial, Remainder, GroupSize);
192: }
193: 
194: // \p Index is 0, 1, and 2 for x, y, and z dimension, respectively.
195: /// Emit code based on Code Object ABI version.
196: /// COV_4    : Emit code to use dispatch ptr
197: /// COV_5+   : Emit code to use implicitarg ptr
198: /// COV_NONE : Emit code to load a global variable "__oclc_ABI_version"
199: ///            and use its value for COV_4 or COV_5+ approach. It is used for
200: ///            compiling device libraries in an ABI-agnostic way.
```
- **EN**: This block defines callable entry points like `Address`, `MDB`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `Address`, `MDB`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 201-225
```cpp
201: Value *EmitAMDGPUWorkGroupSize(CodeGenFunction &CGF, unsigned Index) {
202:   auto Cov = CGF.getTarget().getTargetOpts().CodeObjectVersion;
203: 
204:   // Do not emit __oclc_ABI_version references with non-empt environment.
205:   if (Cov == CodeObjectVersionKind::COV_None &&
206:       CGF.getTarget().getTriple().hasEnvironment())
207:     Cov = CodeObjectVersionKind::COV_6;
208: 
209:   if (Cov == CodeObjectVersionKind::COV_None) {
210:     StringRef Name = "__oclc_ABI_version";
211:     auto *ABIVersionC = CGF.CGM.getModule().getNamedGlobal(Name);
212:     if (!ABIVersionC)
213:       ABIVersionC = new llvm::GlobalVariable(
214:           CGF.CGM.getModule(), CGF.Int32Ty, false,
215:           llvm::GlobalValue::ExternalLinkage, nullptr, Name, nullptr,
216:           llvm::GlobalVariable::NotThreadLocal,
217:           CGF.CGM.getContext().getTargetAddressSpace(LangAS::opencl_constant));
218: 
219:     // This load will be eliminated by the IPSCCP because it is constant
220:     // weak_odr without externally_initialized. Either changing it to weak or
221:     // adding externally_initialized will keep the load.
222:     Value *ABIVersion = CGF.Builder.CreateAlignedLoad(CGF.Int32Ty, ABIVersionC,
223:                                                       CGF.CGM.getIntAlign());
224: 
225:     Value *IsCOV5 = CGF.Builder.CreateICmpSGE(
```
- **EN**: This block uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if）细化 目标内建函数降级 行为。

### Lines 226-250
```cpp
226:         ABIVersion,
227:         llvm::ConstantInt::get(CGF.Int32Ty, CodeObjectVersionKind::COV_5));
228: 
229:     llvm::Value *V5Impl = emitAMDGPUWorkGroupSizeV5(CGF, Index);
230:     llvm::Value *V4Impl = emitAMDGPUWorkGroupSizeV4(CGF, Index);
231:     return CGF.Builder.CreateSelect(IsCOV5, V5Impl, V4Impl);
232:   }
233: 
234:   return Cov >= CodeObjectVersionKind::COV_5
235:              ? emitAMDGPUWorkGroupSizeV5(CGF, Index)
236:              : emitAMDGPUWorkGroupSizeV4(CGF, Index);
237: }
238: 
239: // \p Index is 0, 1, and 2 for x, y, and z dimension, respectively.
240: Value *EmitAMDGPUGridSize(CodeGenFunction &CGF, unsigned Index) {
241:   const unsigned XOffset = 12;
242:   auto *DP = EmitAMDGPUDispatchPtr(CGF);
243:   // Indexing the HSA kernel_dispatch_packet struct.
244:   auto *Offset = llvm::ConstantInt::get(CGF.Int32Ty, XOffset + Index * 4);
245:   auto *GEP = CGF.Builder.CreateGEP(CGF.Int8Ty, DP, Offset);
246:   auto *LD = CGF.Builder.CreateLoad(
247:       Address(GEP, CGF.Int32Ty, CharUnits::fromQuantity(4)));
248: 
249:   llvm::MDBuilder MDB(CGF.getLLVMContext());
250: 
```
- **EN**: This block defines callable entry points like `get`, `emitAMDGPUWorkGroupSizeV4`, `Address`, `MDB`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `get`, `emitAMDGPUWorkGroupSizeV4`, `Address`, `MDB`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 251-275
```cpp
251:   // Known non-zero.
252:   LD->setMetadata(llvm::LLVMContext::MD_range,
253:                   MDB.createRange(APInt(32, 1), APInt::getZero(32)));
254:   LD->setMetadata(llvm::LLVMContext::MD_invariant_load,
255:                   llvm::MDNode::get(CGF.getLLVMContext(), {}));
256:   return LD;
257: }
258: } // namespace
259: 
260: // Generates the IR for __builtin_read_exec_*.
261: // Lowers the builtin to amdgcn_ballot intrinsic.
262: static Value *EmitAMDGCNBallotForExec(CodeGenFunction &CGF, const CallExpr *E,
263:                                       llvm::Type *RegisterType,
264:                                       llvm::Type *ValueType, bool isExecHi) {
265:   CodeGen::CGBuilderTy &Builder = CGF.Builder;
266:   CodeGen::CodeGenModule &CGM = CGF.CGM;
267: 
268:   Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_ballot, {RegisterType});
269:   llvm::Value *Call = Builder.CreateCall(F, {Builder.getInt1(true)});
270: 
271:   if (isExecHi) {
272:     Value *Rt2 = Builder.CreateLShr(Call, 32);
273:     Rt2 = Builder.CreateTrunc(Rt2, CGF.Int32Ty);
274:     return Rt2;
275:   }
```
- **EN**: This block opens or references namespaces `static`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块打开或引用命名空间 `static`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 276-300
```cpp
276: 
277:   return Call;
278: }
279: 
280: static llvm::Value *loadTextureDescPtorAsVec8I32(CodeGenFunction &CGF,
281:                                                  llvm::Value *RsrcPtr) {
282:   auto &B = CGF.Builder;
283:   auto *VecTy = llvm::FixedVectorType::get(B.getInt32Ty(), 8);
284: 
285:   if (RsrcPtr->getType() == VecTy)
286:     return RsrcPtr;
287: 
288:   if (RsrcPtr->getType()->isIntegerTy(32)) {
289:     llvm::PointerType *VecPtrTy =
290:         llvm::PointerType::get(CGF.getLLVMContext(), 8);
291:     llvm::Value *Ptr = B.CreateIntToPtr(RsrcPtr, VecPtrTy, "tex.rsrc.from.int");
292:     return B.CreateAlignedLoad(VecTy, Ptr, llvm::Align(32), "tex.rsrc.val");
293:   }
294: 
295:   if (RsrcPtr->getType()->isPointerTy()) {
296:     auto *VecPtrTy = llvm::PointerType::get(
297:         CGF.getLLVMContext(), RsrcPtr->getType()->getPointerAddressSpace());
298:     llvm::Value *Typed = B.CreateBitCast(RsrcPtr, VecPtrTy, "tex.rsrc.typed");
299:     return B.CreateAlignedLoad(VecTy, Typed, llvm::Align(32), "tex.rsrc.val");
300:   }
```
- **EN**: This block defines callable entry points like `get`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 301-325
```cpp
301: 
302:   const auto &DL = CGF.CGM.getDataLayout();
303:   if (DL.getTypeSizeInBits(RsrcPtr->getType()) == 256)
304:     return B.CreateBitCast(RsrcPtr, VecTy, "tex.rsrc.val");
305: 
306:   llvm::report_fatal_error("Unexpected texture resource argument form");
307: }
308: 
309: llvm::CallInst *
310: emitAMDGCNImageOverloadedReturnType(clang::CodeGen::CodeGenFunction &CGF,
311:                                     const clang::CallExpr *E,
312:                                     unsigned IntrinsicID, bool IsImageStore) {
313:   auto findTextureDescIndex = [&CGF](const CallExpr *E) -> unsigned {
314:     QualType TexQT = CGF.getContext().AMDGPUTextureTy;
315:     for (unsigned I = 0, N = E->getNumArgs(); I < N; ++I) {
316:       QualType ArgTy = E->getArg(I)->getType();
317:       if (ArgTy == TexQT) {
318:         return I;
319:       }
320: 
321:       if (ArgTy.getCanonicalType() == TexQT.getCanonicalType()) {
322:         return I;
323:       }
324:     }
325: 
```
- **EN**: This block defines callable entry points like `report_fatal_error`, `emitAMDGCNImageOverloadedReturnType`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `report_fatal_error`, `emitAMDGCNImageOverloadedReturnType`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 326-350
```cpp
326:     return ~0U;
327:   };
328: 
329:   clang::SmallVector<llvm::Value *, 10> Args;
330:   unsigned RsrcIndex = findTextureDescIndex(E);
331: 
332:   if (RsrcIndex == ~0U) {
333:     llvm::report_fatal_error("Invalid argument count for image builtin");
334:   }
335: 
336:   for (unsigned I = 0; I < E->getNumArgs(); ++I) {
337:     llvm::Value *V = CGF.EmitScalarExpr(E->getArg(I));
338:     if (I == RsrcIndex)
339:       V = loadTextureDescPtorAsVec8I32(CGF, V);
340:     Args.push_back(V);
341:   }
342: 
343:   llvm::Type *RetTy = IsImageStore ? CGF.VoidTy : CGF.ConvertType(E->getType());
344:   llvm::CallInst *Call = CGF.Builder.CreateIntrinsic(RetTy, IntrinsicID, Args);
345:   return Call;
346: }
347: 
348: // Emit an intrinsic that has 1 float or double operand, and 1 integer.
349: static Value *emitFPIntBuiltin(CodeGenFunction &CGF,
350:                                const CallExpr *E,
```
- **EN**: This block defines callable entry points like `report_fatal_error`; uses control flow (if, for) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `report_fatal_error`；通过控制流（if, for）细化 目标内建函数降级 行为。

### Lines 351-375
```cpp
351:                                unsigned IntrinsicID) {
352:   llvm::Value *Src0 = CGF.EmitScalarExpr(E->getArg(0));
353:   llvm::Value *Src1 = CGF.EmitScalarExpr(E->getArg(1));
354: 
355:   Function *F = CGF.CGM.getIntrinsic(IntrinsicID, Src0->getType());
356:   return CGF.Builder.CreateCall(F, {Src0, Src1});
357: }
358: 
359: static inline StringRef mapScopeToSPIRV(StringRef AMDGCNScope) {
360:   if (AMDGCNScope == "agent")
361:     return "device";
362:   if (AMDGCNScope == "wavefront")
363:     return "subgroup";
364:   return AMDGCNScope;
365: }
366: 
367: static llvm::AtomicOrdering mapCABIAtomicOrdering(unsigned AO) {
368:   // Map C11/C++11 memory ordering to LLVM memory ordering
369:   assert(llvm::isValidAtomicOrderingCABI(AO));
370:   switch (static_cast<llvm::AtomicOrderingCABI>(AO)) {
371:   case llvm::AtomicOrderingCABI::acquire:
372:   case llvm::AtomicOrderingCABI::consume:
373:     return llvm::AtomicOrdering::Acquire;
374:   case llvm::AtomicOrderingCABI::release:
375:     return llvm::AtomicOrdering::Release;
```
- **EN**: This block defines callable entry points like `mapScopeToSPIRV`, `mapCABIAtomicOrdering`; uses control flow (if, switch, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `mapScopeToSPIRV`, `mapCABIAtomicOrdering`；通过控制流（if, switch, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 376-400
```cpp
376:   case llvm::AtomicOrderingCABI::acq_rel:
377:     return llvm::AtomicOrdering::AcquireRelease;
378:   case llvm::AtomicOrderingCABI::seq_cst:
379:     return llvm::AtomicOrdering::SequentiallyConsistent;
380:   case llvm::AtomicOrderingCABI::relaxed:
381:     return llvm::AtomicOrdering::Monotonic;
382:   }
383:   llvm_unreachable("Unknown AtomicOrderingCABI enum");
384: }
385: 
386: // For processing memory ordering and memory scope arguments of various
387: // amdgcn builtins.
388: // \p Order takes a C++11 compatible memory-ordering specifier and converts
389: // it into LLVM's memory ordering specifier using atomic C ABI, and writes
390: // to \p AO. \p Scope takes a const char * and converts it into AMDGCN
391: // specific SyncScopeID and writes it to \p SSID.
392: void CodeGenFunction::ProcessOrderScopeAMDGCN(Value *Order, Value *Scope,
393:                                               llvm::AtomicOrdering &AO,
394:                                               llvm::SyncScope::ID &SSID) {
395:   int ord = cast<llvm::ConstantInt>(Order)->getZExtValue();
396: 
397:   // Map C11/C++11 memory ordering to LLVM memory ordering
398:   AO = mapCABIAtomicOrdering(ord);
399: 
400:   // Some of the atomic builtins take the scope as a string name.
```
- **EN**: This block defines callable entry points like `ProcessOrderScopeAMDGCN`; uses control flow (case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `ProcessOrderScopeAMDGCN`；通过控制流（case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 401-425
```cpp
401:   StringRef scp;
402:   if (llvm::getConstantStringInfo(Scope, scp)) {
403:     if (getTarget().getTriple().isSPIRV())
404:       scp = mapScopeToSPIRV(scp);
405:     SSID = getLLVMContext().getOrInsertSyncScopeID(scp);
406:     return;
407:   }
408: 
409:   // Older builtins had an enum argument for the memory scope.
410:   const char *SSN = nullptr;
411:   int scope = cast<llvm::ConstantInt>(Scope)->getZExtValue();
412:   switch (scope) {
413:   case AtomicScopeGenericModel::System: // __MEMORY_SCOPE_SYSTEM
414:     SSID = llvm::SyncScope::System;
415:     break;
416:   case AtomicScopeGenericModel::Device: // __MEMORY_SCOPE_DEVICE
417:     SSN = getTarget().getTriple().isSPIRV() ? "device" : "agent";
418:     break;
419:   case AtomicScopeGenericModel::Workgroup: // __MEMORY_SCOPE_WRKGRP
420:     SSN = "workgroup";
421:     break;
422:   case AtomicScopeGenericModel::Cluster: // __MEMORY_SCOPE_CLUSTR
423:     SSN = getTarget().getTriple().isSPIRV() ? "workgroup" : "cluster";
424:     break;
425:   case AtomicScopeGenericModel::Wavefront: // __MEMORY_SCOPE_WVFRNT
```
- **EN**: This block uses control flow (if, switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, switch, case）细化 目标内建函数降级 行为。

### Lines 426-450
```cpp
426:     SSN = getTarget().getTriple().isSPIRV() ? "subgroup" : "wavefront";
427:     break;
428:   case AtomicScopeGenericModel::Single: // __MEMORY_SCOPE_SINGLE
429:     SSID = llvm::SyncScope::SingleThread;
430:     break;
431:   default:
432:     SSID = llvm::SyncScope::System;
433:     break;
434:   }
435:   if (SSN)
436:     SSID = getLLVMContext().getOrInsertSyncScopeID(SSN);
437: }
438: 
439: void CodeGenFunction::AddAMDGPUFenceAddressSpaceMMRA(llvm::Instruction *Inst,
440:                                                      const CallExpr *E) {
441:   constexpr const char *Tag = "amdgpu-synchronize-as";
442: 
443:   LLVMContext &Ctx = Inst->getContext();
444:   SmallVector<MMRAMetadata::TagT, 3> MMRAs;
445:   for (unsigned K = 2; K < E->getNumArgs(); ++K) {
446:     llvm::Value *V = EmitScalarExpr(E->getArg(K));
447:     StringRef AS;
448:     if (llvm::getConstantStringInfo(V, AS)) {
449:       MMRAs.push_back({Tag, AS});
450:       // TODO: Delete the resulting unused constant?
```
- **EN**: This block defines callable entry points like `AddAMDGPUFenceAddressSpaceMMRA`; uses control flow (if, for, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `AddAMDGPUFenceAddressSpaceMMRA`；通过控制流（if, for, case）细化 目标内建函数降级 行为。

### Lines 451-475
```cpp
451:       continue;
452:     }
453:     CGM.Error(E->getExprLoc(),
454:               "expected an address space name as a string literal");
455:   }
456: 
457:   llvm::sort(MMRAs);
458:   MMRAs.erase(llvm::unique(MMRAs), MMRAs.end());
459:   Inst->setMetadata(LLVMContext::MD_mmra, MMRAMetadata::getMD(Ctx, MMRAs));
460: }
461: 
462: static Value *GetAMDGPUPredicate(CodeGenFunction &CGF, Twine Name) {
463:   Constant *SpecId = ConstantInt::getAllOnesValue(CGF.Int32Ty);
464: 
465:   LLVMContext &Ctx = CGF.getLLVMContext();
466:   MDNode *Predicate = MDNode::get(Ctx, MDString::get(Ctx, Name.str()));
467:   std::vector<Value *> Args = {SpecId, ConstantInt::getFalse(Ctx),
468:                                MetadataAsValue::get(Ctx, Predicate)};
469:   CallInst *Call = CGF.Builder.CreateIntrinsic(
470:       Intrinsic::spv_named_boolean_spec_constant, Args);
471: 
472:   return Call;
473: }
474: 
475: static Intrinsic::ID getIntrinsicIDforWaveReduction(unsigned BuiltinID) {
```
- **EN**: This block defines callable entry points like `sort`, `getIntrinsicIDforWaveReduction`; returns or forwards computed values for the surrounding target builtin lowering logic.
- **CN**: 该代码块定义可调用入口，例如 `sort`, `getIntrinsicIDforWaveReduction`；为周围的 目标内建函数降级 逻辑返回或转发计算结果。

### Lines 476-500
```cpp
476:   switch (BuiltinID) {
477:   default:
478:     llvm_unreachable("Unknown BuiltinID for wave reduction");
479:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_add_u32:
480:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_add_u64:
481:     return Intrinsic::amdgcn_wave_reduce_add;
482:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fadd_f32:
483:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fadd_f64:
484:     return Intrinsic::amdgcn_wave_reduce_fadd;
485:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_sub_u32:
486:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_sub_u64:
487:     return Intrinsic::amdgcn_wave_reduce_sub;
488:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fsub_f32:
489:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fsub_f64:
490:     return Intrinsic::amdgcn_wave_reduce_fsub;
491:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_min_i32:
492:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_min_i64:
493:     return Intrinsic::amdgcn_wave_reduce_min;
494:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fmin_f32:
495:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fmin_f64:
496:     return Intrinsic::amdgcn_wave_reduce_fmin;
497:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_min_u32:
498:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_min_u64:
499:     return Intrinsic::amdgcn_wave_reduce_umin;
500:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_max_i32:
```
- **EN**: This block uses control flow (switch, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（switch, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 501-525
```cpp
501:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_max_i64:
502:     return Intrinsic::amdgcn_wave_reduce_max;
503:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fmax_f32:
504:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_fmax_f64:
505:     return Intrinsic::amdgcn_wave_reduce_fmax;
506:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_max_u32:
507:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_max_u64:
508:     return Intrinsic::amdgcn_wave_reduce_umax;
509:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_and_b32:
510:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_and_b64:
511:     return Intrinsic::amdgcn_wave_reduce_and;
512:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_or_b32:
513:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_or_b64:
514:     return Intrinsic::amdgcn_wave_reduce_or;
515:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_xor_b32:
516:   case clang::AMDGPU::BI__builtin_amdgcn_wave_reduce_xor_b64:
517:     return Intrinsic::amdgcn_wave_reduce_xor;
518:   }
519: }
520: 
521: Value *CodeGenFunction::EmitAMDGPUBuiltinExpr(unsigned BuiltinID,
522:                                               const CallExpr *E) {
523:   llvm::AtomicOrdering AO = llvm::AtomicOrdering::SequentiallyConsistent;
524:   llvm::SyncScope::ID SSID;
525:   switch (BuiltinID) {
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 526-550
```cpp
526:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_add_u32:
527:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fadd_f32:
528:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fadd_f64:
529:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_sub_u32:
530:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fsub_f32:
531:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fsub_f64:
532:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_i32:
533:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_u32:
534:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fmin_f32:
535:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fmin_f64:
536:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_i32:
537:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_u32:
538:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fmax_f32:
539:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_fmax_f64:
540:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_and_b32:
541:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_or_b32:
542:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_xor_b32:
543:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_add_u64:
544:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_sub_u64:
545:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_i64:
546:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_min_u64:
547:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_i64:
548:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_max_u64:
549:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_and_b64:
550:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_or_b64:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 551-575
```cpp
551:   case AMDGPU::BI__builtin_amdgcn_wave_reduce_xor_b64: {
552:     Intrinsic::ID IID = getIntrinsicIDforWaveReduction(BuiltinID);
553:     llvm::Value *Value = EmitScalarExpr(E->getArg(0));
554:     llvm::Value *Strategy = EmitScalarExpr(E->getArg(1));
555:     llvm::Function *F = CGM.getIntrinsic(IID, {Value->getType()});
556:     return Builder.CreateCall(F, {Value, Strategy});
557:   }
558:   case AMDGPU::BI__builtin_amdgcn_div_scale:
559:   case AMDGPU::BI__builtin_amdgcn_div_scalef: {
560:     // Translate from the intrinsics's struct return to the builtin's out
561:     // argument.
562: 
563:     Address FlagOutPtr = EmitPointerWithAlignment(E->getArg(3));
564: 
565:     llvm::Value *X = EmitScalarExpr(E->getArg(0));
566:     llvm::Value *Y = EmitScalarExpr(E->getArg(1));
567:     llvm::Value *Z = EmitScalarExpr(E->getArg(2));
568: 
569:     llvm::Function *Callee = CGM.getIntrinsic(Intrinsic::amdgcn_div_scale,
570:                                            X->getType());
571: 
572:     llvm::Value *Tmp = Builder.CreateCall(Callee, {X, Y, Z});
573: 
574:     llvm::Value *Result = Builder.CreateExtractValue(Tmp, 0);
575:     llvm::Value *Flag = Builder.CreateExtractValue(Tmp, 1);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 576-600
```cpp
576: 
577:     llvm::Type *RealFlagType = FlagOutPtr.getElementType();
578: 
579:     llvm::Value *FlagExt = Builder.CreateZExt(Flag, RealFlagType);
580:     Builder.CreateStore(FlagExt, FlagOutPtr);
581:     return Result;
582:   }
583:   case AMDGPU::BI__builtin_amdgcn_div_fmas:
584:   case AMDGPU::BI__builtin_amdgcn_div_fmasf: {
585:     llvm::Value *Src0 = EmitScalarExpr(E->getArg(0));
586:     llvm::Value *Src1 = EmitScalarExpr(E->getArg(1));
587:     llvm::Value *Src2 = EmitScalarExpr(E->getArg(2));
588:     llvm::Value *Src3 = EmitScalarExpr(E->getArg(3));
589: 
590:     llvm::Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_div_fmas,
591:                                       Src0->getType());
592:     llvm::Value *Src3ToBool = Builder.CreateIsNotNull(Src3);
593:     return Builder.CreateCall(F, {Src0, Src1, Src2, Src3ToBool});
594:   }
595: 
596:   case AMDGPU::BI__builtin_amdgcn_ds_swizzle:
597:     return emitBuiltinWithOneOverloadedType<2>(*this, E,
598:                                                Intrinsic::amdgcn_ds_swizzle);
599:   case AMDGPU::BI__builtin_amdgcn_mov_dpp8:
600:   case AMDGPU::BI__builtin_amdgcn_mov_dpp:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 601-625
```cpp
601:   case AMDGPU::BI__builtin_amdgcn_update_dpp: {
602:     llvm::SmallVector<llvm::Value *, 6> Args;
603:     // Find out if any arguments are required to be integer constant
604:     // expressions.
605:     unsigned ICEArguments = 0;
606:     ASTContext::GetBuiltinTypeError Error;
607:     getContext().GetBuiltinType(BuiltinID, Error, &ICEArguments);
608:     assert(Error == ASTContext::GE_None && "Should not codegen an error");
609:     llvm::Type *DataTy = ConvertType(E->getArg(0)->getType());
610:     unsigned Size = DataTy->getPrimitiveSizeInBits();
611:     llvm::Type *IntTy =
612:         llvm::IntegerType::get(Builder.getContext(), std::max(Size, 32u));
613:     Function *F =
614:         CGM.getIntrinsic(BuiltinID == AMDGPU::BI__builtin_amdgcn_mov_dpp8
615:                              ? Intrinsic::amdgcn_mov_dpp8
616:                              : Intrinsic::amdgcn_update_dpp,
617:                          IntTy);
618:     assert(E->getNumArgs() == 5 || E->getNumArgs() == 6 ||
619:            E->getNumArgs() == 2);
620:     bool InsertOld = BuiltinID == AMDGPU::BI__builtin_amdgcn_mov_dpp;
621:     if (InsertOld)
622:       Args.push_back(llvm::PoisonValue::get(IntTy));
623:     for (unsigned I = 0; I != E->getNumArgs(); ++I) {
624:       llvm::Value *V = EmitScalarOrConstFoldImmArg(ICEArguments, I, E);
625:       if (I < (BuiltinID == AMDGPU::BI__builtin_amdgcn_update_dpp ? 2u : 1u) &&
```
- **EN**: This block defines callable entry points like `getContext`, `get`; uses control flow (if, for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `getContext`, `get`；通过控制流（if, for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 626-650
```cpp
626:           Size < 32) {
627:         if (!DataTy->isIntegerTy())
628:           V = Builder.CreateBitCast(
629:               V, llvm::IntegerType::get(Builder.getContext(), Size));
630:         V = Builder.CreateZExtOrBitCast(V, IntTy);
631:       }
632:       llvm::Type *ExpTy =
633:           F->getFunctionType()->getFunctionParamType(I + InsertOld);
634:       Args.push_back(Builder.CreateTruncOrBitCast(V, ExpTy));
635:     }
636:     Value *V = Builder.CreateCall(F, Args);
637:     if (Size < 32 && !DataTy->isIntegerTy())
638:       V = Builder.CreateTrunc(
639:           V, llvm::IntegerType::get(Builder.getContext(), Size));
640:     return Builder.CreateTruncOrBitCast(V, DataTy);
641:   }
642:   case AMDGPU::BI__builtin_amdgcn_permlane16:
643:   case AMDGPU::BI__builtin_amdgcn_permlanex16:
644:     return emitBuiltinWithOneOverloadedType<6>(
645:         *this, E,
646:         BuiltinID == AMDGPU::BI__builtin_amdgcn_permlane16
647:             ? Intrinsic::amdgcn_permlane16
648:             : Intrinsic::amdgcn_permlanex16);
649:   case AMDGPU::BI__builtin_amdgcn_permlane64:
650:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 651-675
```cpp
651:                                                Intrinsic::amdgcn_permlane64);
652:   case AMDGPU::BI__builtin_amdgcn_readlane:
653:     return emitBuiltinWithOneOverloadedType<2>(*this, E,
654:                                                Intrinsic::amdgcn_readlane);
655:   case AMDGPU::BI__builtin_amdgcn_wave_shuffle:
656:     return emitBuiltinWithOneOverloadedType<2>(*this, E,
657:                                                Intrinsic::amdgcn_wave_shuffle);
658:   case AMDGPU::BI__builtin_amdgcn_readfirstlane:
659:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
660:                                                Intrinsic::amdgcn_readfirstlane);
661:   case AMDGPU::BI__builtin_amdgcn_div_fixup:
662:   case AMDGPU::BI__builtin_amdgcn_div_fixupf:
663:   case AMDGPU::BI__builtin_amdgcn_div_fixuph:
664:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
665:                                                Intrinsic::amdgcn_div_fixup);
666:   case AMDGPU::BI__builtin_amdgcn_trig_preop:
667:   case AMDGPU::BI__builtin_amdgcn_trig_preopf:
668:     return emitFPIntBuiltin(*this, E, Intrinsic::amdgcn_trig_preop);
669:   case AMDGPU::BI__builtin_amdgcn_rcp:
670:   case AMDGPU::BI__builtin_amdgcn_rcpf:
671:   case AMDGPU::BI__builtin_amdgcn_rcph:
672:   case AMDGPU::BI__builtin_amdgcn_rcp_bf16:
673:     return emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::amdgcn_rcp);
674:   case AMDGPU::BI__builtin_amdgcn_sqrt:
675:   case AMDGPU::BI__builtin_amdgcn_sqrtf:
```
- **EN**: This block spells out callable entry points like `emitFPIntBuiltin`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitFPIntBuiltin`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 676-700
```cpp
676:   case AMDGPU::BI__builtin_amdgcn_sqrth:
677:   case AMDGPU::BI__builtin_amdgcn_sqrt_bf16:
678:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
679:                                                Intrinsic::amdgcn_sqrt);
680:   case AMDGPU::BI__builtin_amdgcn_rsq:
681:   case AMDGPU::BI__builtin_amdgcn_rsqf:
682:   case AMDGPU::BI__builtin_amdgcn_rsqh:
683:   case AMDGPU::BI__builtin_amdgcn_rsq_bf16:
684:     return emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::amdgcn_rsq);
685:   case AMDGPU::BI__builtin_amdgcn_rsq_clamp:
686:   case AMDGPU::BI__builtin_amdgcn_rsq_clampf:
687:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
688:                                                Intrinsic::amdgcn_rsq_clamp);
689:   case AMDGPU::BI__builtin_amdgcn_sinf:
690:   case AMDGPU::BI__builtin_amdgcn_sinh:
691:   case AMDGPU::BI__builtin_amdgcn_sin_bf16:
692:     return emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::amdgcn_sin);
693:   case AMDGPU::BI__builtin_amdgcn_cosf:
694:   case AMDGPU::BI__builtin_amdgcn_cosh:
695:   case AMDGPU::BI__builtin_amdgcn_cos_bf16:
696:     return emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::amdgcn_cos);
697:   case AMDGPU::BI__builtin_amdgcn_dispatch_ptr:
698:     return EmitAMDGPUDispatchPtr(*this, E);
699:   case AMDGPU::BI__builtin_amdgcn_logf:
700:   case AMDGPU::BI__builtin_amdgcn_log_bf16:
```
- **EN**: This block spells out callable entry points like `EmitAMDGPUDispatchPtr`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitAMDGPUDispatchPtr`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 701-725
```cpp
701:     return emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::amdgcn_log);
702:   case AMDGPU::BI__builtin_amdgcn_exp2f:
703:   case AMDGPU::BI__builtin_amdgcn_exp2_bf16:
704:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
705:                                                Intrinsic::amdgcn_exp2);
706:   case AMDGPU::BI__builtin_amdgcn_log_clampf:
707:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
708:                                                Intrinsic::amdgcn_log_clamp);
709:   case AMDGPU::BI__builtin_amdgcn_ldexp:
710:   case AMDGPU::BI__builtin_amdgcn_ldexpf: {
711:     llvm::Value *Src0 = EmitScalarExpr(E->getArg(0));
712:     llvm::Value *Src1 = EmitScalarExpr(E->getArg(1));
713:     llvm::Function *F =
714:         CGM.getIntrinsic(Intrinsic::ldexp, {Src0->getType(), Src1->getType()});
715:     return Builder.CreateCall(F, {Src0, Src1});
716:   }
717:   case AMDGPU::BI__builtin_amdgcn_ldexph: {
718:     // The raw instruction has a different behavior for out of bounds exponent
719:     // values (implicit truncation instead of saturate to short_min/short_max).
720:     llvm::Value *Src0 = EmitScalarExpr(E->getArg(0));
721:     llvm::Value *Src1 = EmitScalarExpr(E->getArg(1));
722:     llvm::Function *F =
723:         CGM.getIntrinsic(Intrinsic::ldexp, {Src0->getType(), Int16Ty});
724:     return Builder.CreateCall(F, {Src0, Builder.CreateTrunc(Src1, Int16Ty)});
725:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 726-750
```cpp
726:   case AMDGPU::BI__builtin_amdgcn_frexp_mant:
727:   case AMDGPU::BI__builtin_amdgcn_frexp_mantf:
728:   case AMDGPU::BI__builtin_amdgcn_frexp_manth:
729:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
730:                                                Intrinsic::amdgcn_frexp_mant);
731:   case AMDGPU::BI__builtin_amdgcn_frexp_exp:
732:   case AMDGPU::BI__builtin_amdgcn_frexp_expf: {
733:     Value *Src0 = EmitScalarExpr(E->getArg(0));
734:     Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_frexp_exp,
735:                                 { Builder.getInt32Ty(), Src0->getType() });
736:     return Builder.CreateCall(F, Src0);
737:   }
738:   case AMDGPU::BI__builtin_amdgcn_frexp_exph: {
739:     Value *Src0 = EmitScalarExpr(E->getArg(0));
740:     Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_frexp_exp,
741:                                 { Builder.getInt16Ty(), Src0->getType() });
742:     return Builder.CreateCall(F, Src0);
743:   }
744:   case AMDGPU::BI__builtin_amdgcn_fract:
745:   case AMDGPU::BI__builtin_amdgcn_fractf:
746:   case AMDGPU::BI__builtin_amdgcn_fracth:
747:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
748:                                                Intrinsic::amdgcn_fract);
749:   case AMDGPU::BI__builtin_amdgcn_lerp:
750:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 751-775
```cpp
751:                                                Intrinsic::amdgcn_lerp);
752:   case AMDGPU::BI__builtin_amdgcn_ubfe:
753:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
754:                                                Intrinsic::amdgcn_ubfe);
755:   case AMDGPU::BI__builtin_amdgcn_sbfe:
756:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
757:                                                Intrinsic::amdgcn_sbfe);
758:   case AMDGPU::BI__builtin_amdgcn_ballot_w32:
759:   case AMDGPU::BI__builtin_amdgcn_ballot_w64: {
760:     llvm::Type *ResultType = ConvertType(E->getType());
761:     llvm::Value *Src = EmitScalarExpr(E->getArg(0));
762:     Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_ballot, {ResultType});
763:     return Builder.CreateCall(F, {Src});
764:   }
765:   case AMDGPU::BI__builtin_amdgcn_inverse_ballot_w32:
766:   case AMDGPU::BI__builtin_amdgcn_inverse_ballot_w64: {
767:     llvm::Value *Src = EmitScalarExpr(E->getArg(0));
768:     Function *F =
769:         CGM.getIntrinsic(Intrinsic::amdgcn_inverse_ballot, {Src->getType()});
770:     return Builder.CreateCall(F, {Src});
771:   }
772:   case AMDGPU::BI__builtin_amdgcn_tanhf:
773:   case AMDGPU::BI__builtin_amdgcn_tanhh:
774:   case AMDGPU::BI__builtin_amdgcn_tanh_bf16:
775:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 776-800
```cpp
776:                                                Intrinsic::amdgcn_tanh);
777:   case AMDGPU::BI__builtin_amdgcn_uicmp:
778:   case AMDGPU::BI__builtin_amdgcn_uicmpl:
779:   case AMDGPU::BI__builtin_amdgcn_sicmp:
780:   case AMDGPU::BI__builtin_amdgcn_sicmpl: {
781:     llvm::Value *Src0 = EmitScalarExpr(E->getArg(0));
782:     llvm::Value *Src1 = EmitScalarExpr(E->getArg(1));
783:     llvm::Value *Src2 = EmitScalarExpr(E->getArg(2));
784: 
785:     // FIXME-GFX10: How should 32 bit mask be handled?
786:     Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_icmp,
787:       { Builder.getInt64Ty(), Src0->getType() });
788:     return Builder.CreateCall(F, { Src0, Src1, Src2 });
789:   }
790:   case AMDGPU::BI__builtin_amdgcn_fcmp:
791:   case AMDGPU::BI__builtin_amdgcn_fcmpf: {
792:     llvm::Value *Src0 = EmitScalarExpr(E->getArg(0));
793:     llvm::Value *Src1 = EmitScalarExpr(E->getArg(1));
794:     llvm::Value *Src2 = EmitScalarExpr(E->getArg(2));
795: 
796:     // FIXME-GFX10: How should 32 bit mask be handled?
797:     Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_fcmp,
798:       { Builder.getInt64Ty(), Src0->getType() });
799:     return Builder.CreateCall(F, { Src0, Src1, Src2 });
800:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 801-825
```cpp
801:   case AMDGPU::BI__builtin_amdgcn_class:
802:   case AMDGPU::BI__builtin_amdgcn_classf:
803:   case AMDGPU::BI__builtin_amdgcn_classh:
804:     return emitFPIntBuiltin(*this, E, Intrinsic::amdgcn_class);
805:   case AMDGPU::BI__builtin_amdgcn_fmed3f:
806:   case AMDGPU::BI__builtin_amdgcn_fmed3h:
807:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
808:                                                Intrinsic::amdgcn_fmed3);
809:   case AMDGPU::BI__builtin_amdgcn_ds_append:
810:   case AMDGPU::BI__builtin_amdgcn_ds_consume: {
811:     Intrinsic::ID Intrin = BuiltinID == AMDGPU::BI__builtin_amdgcn_ds_append ?
812:       Intrinsic::amdgcn_ds_append : Intrinsic::amdgcn_ds_consume;
813:     Value *Src0 = EmitScalarExpr(E->getArg(0));
814:     Function *F = CGM.getIntrinsic(Intrin, { Src0->getType() });
815:     return Builder.CreateCall(F, { Src0, Builder.getFalse() });
816:   }
817:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b64_i32:
818:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b64_v2i32:
819:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4i16:
820:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4f16:
821:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4bf16:
822:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8i16:
823:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8f16:
824:   case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8bf16:
825:   case AMDGPU::BI__builtin_amdgcn_global_load_tr4_b64_v2i32:
```
- **EN**: This block defines callable entry points like `emitFPIntBuiltin`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `emitFPIntBuiltin`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 826-850
```cpp
826:   case AMDGPU::BI__builtin_amdgcn_global_load_tr8_b64_v2i32:
827:   case AMDGPU::BI__builtin_amdgcn_global_load_tr6_b96_v3i32:
828:   case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8i16:
829:   case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8f16:
830:   case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8bf16:
831:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr4_b64_v2i32:
832:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr8_b64_v2i32:
833:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr6_b96_v3i32:
834:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8i16:
835:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8f16:
836:   case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8bf16:
837:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr4_b64_v2i32:
838:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr8_b64_v2i32:
839:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr6_b96_v3i32:
840:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4f16:
841:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4bf16:
842:   case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4i16: {
843:     Intrinsic::ID IID;
844:     switch (BuiltinID) {
845:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b64_i32:
846:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b64_v2i32:
847:     case AMDGPU::BI__builtin_amdgcn_global_load_tr8_b64_v2i32:
848:       IID = Intrinsic::amdgcn_global_load_tr_b64;
849:       break;
850:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4i16:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 851-875
```cpp
851:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4f16:
852:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v4bf16:
853:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8i16:
854:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8f16:
855:     case AMDGPU::BI__builtin_amdgcn_global_load_tr_b128_v8bf16:
856:     case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8i16:
857:     case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8f16:
858:     case AMDGPU::BI__builtin_amdgcn_global_load_tr16_b128_v8bf16:
859:       IID = Intrinsic::amdgcn_global_load_tr_b128;
860:       break;
861:     case AMDGPU::BI__builtin_amdgcn_global_load_tr4_b64_v2i32:
862:       IID = Intrinsic::amdgcn_global_load_tr4_b64;
863:       break;
864:     case AMDGPU::BI__builtin_amdgcn_global_load_tr6_b96_v3i32:
865:       IID = Intrinsic::amdgcn_global_load_tr6_b96;
866:       break;
867:     case AMDGPU::BI__builtin_amdgcn_ds_load_tr4_b64_v2i32:
868:       IID = Intrinsic::amdgcn_ds_load_tr4_b64;
869:       break;
870:     case AMDGPU::BI__builtin_amdgcn_ds_load_tr6_b96_v3i32:
871:       IID = Intrinsic::amdgcn_ds_load_tr6_b96;
872:       break;
873:     case AMDGPU::BI__builtin_amdgcn_ds_load_tr8_b64_v2i32:
874:       IID = Intrinsic::amdgcn_ds_load_tr8_b64;
875:       break;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 876-900
```cpp
876:     case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8i16:
877:     case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8f16:
878:     case AMDGPU::BI__builtin_amdgcn_ds_load_tr16_b128_v8bf16:
879:       IID = Intrinsic::amdgcn_ds_load_tr16_b128;
880:       break;
881:     case AMDGPU::BI__builtin_amdgcn_ds_read_tr4_b64_v2i32:
882:       IID = Intrinsic::amdgcn_ds_read_tr4_b64;
883:       break;
884:     case AMDGPU::BI__builtin_amdgcn_ds_read_tr8_b64_v2i32:
885:       IID = Intrinsic::amdgcn_ds_read_tr8_b64;
886:       break;
887:     case AMDGPU::BI__builtin_amdgcn_ds_read_tr6_b96_v3i32:
888:       IID = Intrinsic::amdgcn_ds_read_tr6_b96;
889:       break;
890:     case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4i16:
891:     case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4f16:
892:     case AMDGPU::BI__builtin_amdgcn_ds_read_tr16_b64_v4bf16:
893:       IID = Intrinsic::amdgcn_ds_read_tr16_b64;
894:       break;
895:     }
896:     llvm::Type *LoadTy = ConvertType(E->getType());
897:     llvm::Value *Addr = EmitScalarExpr(E->getArg(0));
898:     llvm::Function *F = CGM.getIntrinsic(IID, {LoadTy});
899:     return Builder.CreateCall(F, {Addr});
900:   }
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 901-925
```cpp
901:   case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b32:
902:   case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b64:
903:   case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b128:
904:   case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b32:
905:   case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b64:
906:   case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b128: {
907: 
908:     Intrinsic::ID IID;
909:     switch (BuiltinID) {
910:     case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b32:
911:       IID = Intrinsic::amdgcn_global_load_monitor_b32;
912:       break;
913:     case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b64:
914:       IID = Intrinsic::amdgcn_global_load_monitor_b64;
915:       break;
916:     case AMDGPU::BI__builtin_amdgcn_global_load_monitor_b128:
917:       IID = Intrinsic::amdgcn_global_load_monitor_b128;
918:       break;
919:     case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b32:
920:       IID = Intrinsic::amdgcn_flat_load_monitor_b32;
921:       break;
922:     case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b64:
923:       IID = Intrinsic::amdgcn_flat_load_monitor_b64;
924:       break;
925:     case AMDGPU::BI__builtin_amdgcn_flat_load_monitor_b128:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 926-950
```cpp
926:       IID = Intrinsic::amdgcn_flat_load_monitor_b128;
927:       break;
928:     }
929: 
930:     LLVMContext &Ctx = CGM.getLLVMContext();
931:     llvm::Type *LoadTy = ConvertType(E->getType());
932:     llvm::Value *Addr = EmitScalarExpr(E->getArg(0));
933: 
934:     auto *AOExpr = cast<llvm::ConstantInt>(EmitScalarExpr(E->getArg(1)));
935:     auto *ScopeExpr = cast<llvm::ConstantInt>(EmitScalarExpr(E->getArg(2)));
936: 
937:     auto Scope = static_cast<SyncScope>(ScopeExpr->getZExtValue());
938:     llvm::AtomicOrdering AO = mapCABIAtomicOrdering(AOExpr->getZExtValue());
939: 
940:     StringRef ScopeStr = CGM.getTargetCodeGenInfo().getLLVMSyncScopeStr(
941:         CGM.getLangOpts(), Scope, AO);
942: 
943:     llvm::MDNode *MD =
944:         llvm::MDNode::get(Ctx, {llvm::MDString::get(Ctx, ScopeStr)});
945:     llvm::Value *ScopeMD = llvm::MetadataAsValue::get(Ctx, MD);
946:     llvm::Function *F = CGM.getIntrinsic(IID, {LoadTy});
947:     return Builder.CreateCall(F, {Addr, AOExpr, ScopeMD});
948:   }
949:   case AMDGPU::BI__builtin_amdgcn_cluster_load_b32:
950:   case AMDGPU::BI__builtin_amdgcn_cluster_load_b64:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 951-975
```cpp
951:   case AMDGPU::BI__builtin_amdgcn_cluster_load_b128: {
952:     Intrinsic::ID IID;
953:     switch (BuiltinID) {
954:     case AMDGPU::BI__builtin_amdgcn_cluster_load_b32:
955:       IID = Intrinsic::amdgcn_cluster_load_b32;
956:       break;
957:     case AMDGPU::BI__builtin_amdgcn_cluster_load_b64:
958:       IID = Intrinsic::amdgcn_cluster_load_b64;
959:       break;
960:     case AMDGPU::BI__builtin_amdgcn_cluster_load_b128:
961:       IID = Intrinsic::amdgcn_cluster_load_b128;
962:       break;
963:     }
964:     SmallVector<Value *, 3> Args;
965:     for (int i = 0, e = E->getNumArgs(); i != e; ++i)
966:       Args.push_back(EmitScalarExpr(E->getArg(i)));
967:     llvm::Function *F = CGM.getIntrinsic(IID, {ConvertType(E->getType())});
968:     return Builder.CreateCall(F, {Args});
969:   }
970:   case AMDGPU::BI__builtin_amdgcn_load_to_lds: {
971:     // Should this have asan instrumentation?
972:     return emitBuiltinWithOneOverloadedType<5>(*this, E,
973:                                                Intrinsic::amdgcn_load_to_lds);
974:   }
975:   case AMDGPU::BI__builtin_amdgcn_load_async_to_lds: {
```
- **EN**: This block uses control flow (switch, for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, for, case）细化 目标内建函数降级 行为。

### Lines 976-1000
```cpp
 976:     // Should this have asan instrumentation?
 977:     return emitBuiltinWithOneOverloadedType<5>(
 978:         *this, E, Intrinsic::amdgcn_load_async_to_lds);
 979:   }
 980:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_32x4B:
 981:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_32x4B:
 982:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_16x8B:
 983:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_16x8B:
 984:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_8x16B:
 985:   case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_8x16B: {
 986:     Intrinsic::ID IID;
 987:     switch (BuiltinID) {
 988:     case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_32x4B:
 989:       IID = Intrinsic::amdgcn_cooperative_atomic_load_32x4B;
 990:       break;
 991:     case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_32x4B:
 992:       IID = Intrinsic::amdgcn_cooperative_atomic_store_32x4B;
 993:       break;
 994:     case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_16x8B:
 995:       IID = Intrinsic::amdgcn_cooperative_atomic_load_16x8B;
 996:       break;
 997:     case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_16x8B:
 998:       IID = Intrinsic::amdgcn_cooperative_atomic_store_16x8B;
 999:       break;
1000:     case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_load_8x16B:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 1001-1025
```cpp
1001:       IID = Intrinsic::amdgcn_cooperative_atomic_load_8x16B;
1002:       break;
1003:     case AMDGPU::BI__builtin_amdgcn_cooperative_atomic_store_8x16B:
1004:       IID = Intrinsic::amdgcn_cooperative_atomic_store_8x16B;
1005:       break;
1006:     }
1007: 
1008:     LLVMContext &Ctx = CGM.getLLVMContext();
1009:     SmallVector<Value *, 5> Args;
1010:     // last argument is a MD string
1011:     const unsigned ScopeArg = E->getNumArgs() - 1;
1012:     for (unsigned i = 0; i != ScopeArg; ++i)
1013:       Args.push_back(EmitScalarExpr(E->getArg(i)));
1014:     StringRef Arg = cast<StringLiteral>(E->getArg(ScopeArg)->IgnoreParenCasts())
1015:                         ->getString();
1016:     llvm::MDNode *MD = llvm::MDNode::get(Ctx, {llvm::MDString::get(Ctx, Arg)});
1017:     Args.push_back(llvm::MetadataAsValue::get(Ctx, MD));
1018:     // Intrinsic is typed based on the pointer AS. Pointer is always the first
1019:     // argument.
1020:     llvm::Function *F = CGM.getIntrinsic(IID, {Args[0]->getType()});
1021:     return Builder.CreateCall(F, {Args});
1022:   }
1023:   case AMDGPU::BI__builtin_amdgcn_get_fpenv: {
1024:     Function *F = CGM.getIntrinsic(Intrinsic::get_fpenv,
1025:                                    {llvm::Type::getInt64Ty(getLLVMContext())});
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1026-1050
```cpp
1026:     return Builder.CreateCall(F);
1027:   }
1028:   case AMDGPU::BI__builtin_amdgcn_set_fpenv: {
1029:     Function *F = CGM.getIntrinsic(Intrinsic::set_fpenv,
1030:                                    {llvm::Type::getInt64Ty(getLLVMContext())});
1031:     llvm::Value *Env = EmitScalarExpr(E->getArg(0));
1032:     return Builder.CreateCall(F, {Env});
1033:   }
1034:   case AMDGPU::BI__builtin_amdgcn_processor_is: {
1035:     assert(CGM.getTriple().isSPIRV() &&
1036:            "__builtin_amdgcn_processor_is should never reach CodeGen for "
1037:            "concrete targets!");
1038:     StringRef Proc = cast<clang::StringLiteral>(E->getArg(0))->getString();
1039:     return GetAMDGPUPredicate(*this, "is." + Proc);
1040:   }
1041:   case AMDGPU::BI__builtin_amdgcn_is_invocable: {
1042:     assert(CGM.getTriple().isSPIRV() &&
1043:            "__builtin_amdgcn_is_invocable should never reach CodeGen for "
1044:            "concrete targets!");
1045:     auto *FD = cast<FunctionDecl>(
1046:         cast<DeclRefExpr>(E->getArg(0))->getReferencedDeclOfCallee());
1047:     StringRef RF =
1048:         getContext().BuiltinInfo.getRequiredFeatures(FD->getBuiltinID());
1049:     return GetAMDGPUPredicate(*this, "has." + RF);
1050:   }
```
- **EN**: This block defines callable entry points like `GetAMDGPUPredicate`, `getContext`; uses control flow (for, case) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `GetAMDGPUPredicate`, `getContext`；通过控制流（for, case）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1051-1075
```cpp
1051:   case AMDGPU::BI__builtin_amdgcn_read_exec:
1052:     return EmitAMDGCNBallotForExec(*this, E, Int64Ty, Int64Ty, false);
1053:   case AMDGPU::BI__builtin_amdgcn_read_exec_lo:
1054:     return EmitAMDGCNBallotForExec(*this, E, Int32Ty, Int32Ty, false);
1055:   case AMDGPU::BI__builtin_amdgcn_read_exec_hi:
1056:     return EmitAMDGCNBallotForExec(*this, E, Int64Ty, Int64Ty, true);
1057:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray:
1058:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray_h:
1059:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray_l:
1060:   case AMDGPU::BI__builtin_amdgcn_image_bvh_intersect_ray_lh: {
1061:     llvm::Value *NodePtr = EmitScalarExpr(E->getArg(0));
1062:     llvm::Value *RayExtent = EmitScalarExpr(E->getArg(1));
1063:     llvm::Value *RayOrigin = EmitScalarExpr(E->getArg(2));
1064:     llvm::Value *RayDir = EmitScalarExpr(E->getArg(3));
1065:     llvm::Value *RayInverseDir = EmitScalarExpr(E->getArg(4));
1066:     llvm::Value *TextureDescr = EmitScalarExpr(E->getArg(5));
1067: 
1068:     // The builtins take these arguments as vec4 where the last element is
1069:     // ignored. The intrinsic takes them as vec3.
1070:     RayOrigin = Builder.CreateShuffleVector(RayOrigin, RayOrigin,
1071:                                             {0, 1, 2});
1072:     RayDir =
1073:         Builder.CreateShuffleVector(RayDir, RayDir, {0, 1, 2});
1074:     RayInverseDir = Builder.CreateShuffleVector(RayInverseDir, RayInverseDir,
1075:                                                 {0, 1, 2});
```
- **EN**: This block defines callable entry points like `EmitAMDGCNBallotForExec`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `EmitAMDGCNBallotForExec`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1076-1100
```cpp
1076: 
1077:     Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_image_bvh_intersect_ray,
1078:                                    {NodePtr->getType(), RayDir->getType()});
1079:     return Builder.CreateCall(F, {NodePtr, RayExtent, RayOrigin, RayDir,
1080:                                   RayInverseDir, TextureDescr});
1081:   }
1082:   case AMDGPU::BI__builtin_amdgcn_image_bvh8_intersect_ray:
1083:   case AMDGPU::BI__builtin_amdgcn_image_bvh_dual_intersect_ray: {
1084:     Intrinsic::ID IID;
1085:     switch (BuiltinID) {
1086:     case AMDGPU::BI__builtin_amdgcn_image_bvh8_intersect_ray:
1087:       IID = Intrinsic::amdgcn_image_bvh8_intersect_ray;
1088:       break;
1089:     case AMDGPU::BI__builtin_amdgcn_image_bvh_dual_intersect_ray:
1090:       IID = Intrinsic::amdgcn_image_bvh_dual_intersect_ray;
1091:       break;
1092:     }
1093:     llvm::Value *NodePtr = EmitScalarExpr(E->getArg(0));
1094:     llvm::Value *RayExtent = EmitScalarExpr(E->getArg(1));
1095:     llvm::Value *InstanceMask = EmitScalarExpr(E->getArg(2));
1096:     llvm::Value *RayOrigin = EmitScalarExpr(E->getArg(3));
1097:     llvm::Value *RayDir = EmitScalarExpr(E->getArg(4));
1098:     llvm::Value *Offset = EmitScalarExpr(E->getArg(5));
1099:     llvm::Value *TextureDescr = EmitScalarExpr(E->getArg(6));
1100: 
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 1101-1125
```cpp
1101:     Address RetRayOriginPtr = EmitPointerWithAlignment(E->getArg(7));
1102:     Address RetRayDirPtr = EmitPointerWithAlignment(E->getArg(8));
1103: 
1104:     llvm::Function *IntrinsicFunc = CGM.getIntrinsic(IID);
1105: 
1106:     llvm::CallInst *CI = Builder.CreateCall(
1107:         IntrinsicFunc, {NodePtr, RayExtent, InstanceMask, RayOrigin, RayDir,
1108:                         Offset, TextureDescr});
1109: 
1110:     llvm::Value *RetVData = Builder.CreateExtractValue(CI, 0);
1111:     llvm::Value *RetRayOrigin = Builder.CreateExtractValue(CI, 1);
1112:     llvm::Value *RetRayDir = Builder.CreateExtractValue(CI, 2);
1113: 
1114:     Builder.CreateStore(RetRayOrigin, RetRayOriginPtr);
1115:     Builder.CreateStore(RetRayDir, RetRayDirPtr);
1116: 
1117:     return RetVData;
1118:   }
1119: 
1120:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_rtn:
1121:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push4_pop1_rtn:
1122:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push8_pop1_rtn:
1123:   case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push8_pop2_rtn: {
1124:     Intrinsic::ID IID;
1125:     switch (BuiltinID) {
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 1126-1150
```cpp
1126:     case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_rtn:
1127:       IID = Intrinsic::amdgcn_ds_bvh_stack_rtn;
1128:       break;
1129:     case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push4_pop1_rtn:
1130:       IID = Intrinsic::amdgcn_ds_bvh_stack_push4_pop1_rtn;
1131:       break;
1132:     case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push8_pop1_rtn:
1133:       IID = Intrinsic::amdgcn_ds_bvh_stack_push8_pop1_rtn;
1134:       break;
1135:     case AMDGPU::BI__builtin_amdgcn_ds_bvh_stack_push8_pop2_rtn:
1136:       IID = Intrinsic::amdgcn_ds_bvh_stack_push8_pop2_rtn;
1137:       break;
1138:     }
1139: 
1140:     SmallVector<Value *, 4> Args;
1141:     for (int i = 0, e = E->getNumArgs(); i != e; ++i)
1142:       Args.push_back(EmitScalarExpr(E->getArg(i)));
1143: 
1144:     Function *F = CGM.getIntrinsic(IID);
1145:     Value *Call = Builder.CreateCall(F, Args);
1146:     Value *Rtn = Builder.CreateExtractValue(Call, 0);
1147:     Value *A = Builder.CreateExtractValue(Call, 1);
1148:     llvm::Type *RetTy = ConvertType(E->getType());
1149:     Value *I0 = Builder.CreateInsertElement(PoisonValue::get(RetTy), Rtn,
1150:                                             (uint64_t)0);
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1151-1175
```cpp
1151:     // ds_bvh_stack_push8_pop2_rtn returns {i64, i32} but the builtin returns
1152:     // <2 x i64>, zext the second value.
1153:     if (A->getType()->getPrimitiveSizeInBits() <
1154:         RetTy->getScalarType()->getPrimitiveSizeInBits())
1155:       A = Builder.CreateZExt(A, RetTy->getScalarType());
1156: 
1157:     return Builder.CreateInsertElement(I0, A, 1);
1158:   }
1159:   case AMDGPU::BI__builtin_amdgcn_image_load_1d_v4f32_i32:
1160:   case AMDGPU::BI__builtin_amdgcn_image_load_1d_v4f16_i32:
1161:     return emitAMDGCNImageOverloadedReturnType(
1162:         *this, E, Intrinsic::amdgcn_image_load_1d, false);
1163:   case AMDGPU::BI__builtin_amdgcn_image_load_1darray_v4f32_i32:
1164:   case AMDGPU::BI__builtin_amdgcn_image_load_1darray_v4f16_i32:
1165:     return emitAMDGCNImageOverloadedReturnType(
1166:         *this, E, Intrinsic::amdgcn_image_load_1darray, false);
1167:   case AMDGPU::BI__builtin_amdgcn_image_load_2d_f32_i32:
1168:   case AMDGPU::BI__builtin_amdgcn_image_load_2d_v4f32_i32:
1169:   case AMDGPU::BI__builtin_amdgcn_image_load_2d_v4f16_i32:
1170:     return emitAMDGCNImageOverloadedReturnType(
1171:         *this, E, Intrinsic::amdgcn_image_load_2d, false);
1172:   case AMDGPU::BI__builtin_amdgcn_image_load_2darray_f32_i32:
1173:   case AMDGPU::BI__builtin_amdgcn_image_load_2darray_v4f32_i32:
1174:   case AMDGPU::BI__builtin_amdgcn_image_load_2darray_v4f16_i32:
1175:     return emitAMDGCNImageOverloadedReturnType(
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1176-1200
```cpp
1176:         *this, E, Intrinsic::amdgcn_image_load_2darray, false);
1177:   case AMDGPU::BI__builtin_amdgcn_image_load_3d_v4f32_i32:
1178:   case AMDGPU::BI__builtin_amdgcn_image_load_3d_v4f16_i32:
1179:     return emitAMDGCNImageOverloadedReturnType(
1180:         *this, E, Intrinsic::amdgcn_image_load_3d, false);
1181:   case AMDGPU::BI__builtin_amdgcn_image_load_cube_v4f32_i32:
1182:   case AMDGPU::BI__builtin_amdgcn_image_load_cube_v4f16_i32:
1183:     return emitAMDGCNImageOverloadedReturnType(
1184:         *this, E, Intrinsic::amdgcn_image_load_cube, false);
1185:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_1d_v4f32_i32:
1186:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_1d_v4f16_i32:
1187:     return emitAMDGCNImageOverloadedReturnType(
1188:         *this, E, Intrinsic::amdgcn_image_load_mip_1d, false);
1189:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_1darray_v4f32_i32:
1190:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_1darray_v4f16_i32:
1191:     return emitAMDGCNImageOverloadedReturnType(
1192:         *this, E, Intrinsic::amdgcn_image_load_mip_1darray, false);
1193:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2d_f32_i32:
1194:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2d_v4f32_i32:
1195:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2d_v4f16_i32:
1196:     return emitAMDGCNImageOverloadedReturnType(
1197:         *this, E, Intrinsic::amdgcn_image_load_mip_2d, false);
1198:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2darray_f32_i32:
1199:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2darray_v4f32_i32:
1200:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_2darray_v4f16_i32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1201-1225
```cpp
1201:     return emitAMDGCNImageOverloadedReturnType(
1202:         *this, E, Intrinsic::amdgcn_image_load_mip_2darray, false);
1203:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_3d_v4f32_i32:
1204:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_3d_v4f16_i32:
1205:     return emitAMDGCNImageOverloadedReturnType(
1206:         *this, E, Intrinsic::amdgcn_image_load_mip_3d, false);
1207:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_cube_v4f32_i32:
1208:   case AMDGPU::BI__builtin_amdgcn_image_load_mip_cube_v4f16_i32:
1209:     return emitAMDGCNImageOverloadedReturnType(
1210:         *this, E, Intrinsic::amdgcn_image_load_mip_cube, false);
1211:   case AMDGPU::BI__builtin_amdgcn_image_store_1d_v4f32_i32:
1212:   case AMDGPU::BI__builtin_amdgcn_image_store_1d_v4f16_i32:
1213:     return emitAMDGCNImageOverloadedReturnType(
1214:         *this, E, Intrinsic::amdgcn_image_store_1d, true);
1215:   case AMDGPU::BI__builtin_amdgcn_image_store_1darray_v4f32_i32:
1216:   case AMDGPU::BI__builtin_amdgcn_image_store_1darray_v4f16_i32:
1217:     return emitAMDGCNImageOverloadedReturnType(
1218:         *this, E, Intrinsic::amdgcn_image_store_1darray, true);
1219:   case AMDGPU::BI__builtin_amdgcn_image_store_2d_f32_i32:
1220:   case AMDGPU::BI__builtin_amdgcn_image_store_2d_v4f32_i32:
1221:   case AMDGPU::BI__builtin_amdgcn_image_store_2d_v4f16_i32:
1222:     return emitAMDGCNImageOverloadedReturnType(
1223:         *this, E, Intrinsic::amdgcn_image_store_2d, true);
1224:   case AMDGPU::BI__builtin_amdgcn_image_store_2darray_f32_i32:
1225:   case AMDGPU::BI__builtin_amdgcn_image_store_2darray_v4f32_i32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1226-1250
```cpp
1226:   case AMDGPU::BI__builtin_amdgcn_image_store_2darray_v4f16_i32:
1227:     return emitAMDGCNImageOverloadedReturnType(
1228:         *this, E, Intrinsic::amdgcn_image_store_2darray, true);
1229:   case AMDGPU::BI__builtin_amdgcn_image_store_3d_v4f32_i32:
1230:   case AMDGPU::BI__builtin_amdgcn_image_store_3d_v4f16_i32:
1231:     return emitAMDGCNImageOverloadedReturnType(
1232:         *this, E, Intrinsic::amdgcn_image_store_3d, true);
1233:   case AMDGPU::BI__builtin_amdgcn_image_store_cube_v4f32_i32:
1234:   case AMDGPU::BI__builtin_amdgcn_image_store_cube_v4f16_i32:
1235:     return emitAMDGCNImageOverloadedReturnType(
1236:         *this, E, Intrinsic::amdgcn_image_store_cube, true);
1237:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1d_v4f32_i32:
1238:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1d_v4f16_i32:
1239:     return emitAMDGCNImageOverloadedReturnType(
1240:         *this, E, Intrinsic::amdgcn_image_store_mip_1d, true);
1241:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1darray_v4f32_i32:
1242:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_1darray_v4f16_i32:
1243:     return emitAMDGCNImageOverloadedReturnType(
1244:         *this, E, Intrinsic::amdgcn_image_store_mip_1darray, true);
1245:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2d_f32_i32:
1246:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2d_v4f32_i32:
1247:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2d_v4f16_i32:
1248:     return emitAMDGCNImageOverloadedReturnType(
1249:         *this, E, Intrinsic::amdgcn_image_store_mip_2d, true);
1250:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2darray_f32_i32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1251-1275
```cpp
1251:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2darray_v4f32_i32:
1252:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_2darray_v4f16_i32:
1253:     return emitAMDGCNImageOverloadedReturnType(
1254:         *this, E, Intrinsic::amdgcn_image_store_mip_2darray, true);
1255:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_3d_v4f32_i32:
1256:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_3d_v4f16_i32:
1257:     return emitAMDGCNImageOverloadedReturnType(
1258:         *this, E, Intrinsic::amdgcn_image_store_mip_3d, true);
1259:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_cube_v4f32_i32:
1260:   case AMDGPU::BI__builtin_amdgcn_image_store_mip_cube_v4f16_i32:
1261:     return emitAMDGCNImageOverloadedReturnType(
1262:         *this, E, Intrinsic::amdgcn_image_store_mip_cube, true);
1263:   case AMDGPU::BI__builtin_amdgcn_image_sample_1d_v4f32_f32:
1264:   case AMDGPU::BI__builtin_amdgcn_image_sample_1d_v4f16_f32:
1265:     return emitAMDGCNImageOverloadedReturnType(
1266:         *this, E, Intrinsic::amdgcn_image_sample_1d, false);
1267:   case AMDGPU::BI__builtin_amdgcn_image_sample_1darray_v4f32_f32:
1268:   case AMDGPU::BI__builtin_amdgcn_image_sample_1darray_v4f16_f32:
1269:     return emitAMDGCNImageOverloadedReturnType(
1270:         *this, E, Intrinsic::amdgcn_image_sample_1darray, false);
1271:   case AMDGPU::BI__builtin_amdgcn_image_sample_2d_f32_f32:
1272:   case AMDGPU::BI__builtin_amdgcn_image_sample_2d_v4f32_f32:
1273:   case AMDGPU::BI__builtin_amdgcn_image_sample_2d_v4f16_f32:
1274:     return emitAMDGCNImageOverloadedReturnType(
1275:         *this, E, Intrinsic::amdgcn_image_sample_2d, false);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1276-1300
```cpp
1276:   case AMDGPU::BI__builtin_amdgcn_image_sample_2darray_f32_f32:
1277:   case AMDGPU::BI__builtin_amdgcn_image_sample_2darray_v4f32_f32:
1278:   case AMDGPU::BI__builtin_amdgcn_image_sample_2darray_v4f16_f32:
1279:     return emitAMDGCNImageOverloadedReturnType(
1280:         *this, E, Intrinsic::amdgcn_image_sample_2darray, false);
1281:   case AMDGPU::BI__builtin_amdgcn_image_sample_3d_v4f32_f32:
1282:   case AMDGPU::BI__builtin_amdgcn_image_sample_3d_v4f16_f32:
1283:     return emitAMDGCNImageOverloadedReturnType(
1284:         *this, E, Intrinsic::amdgcn_image_sample_3d, false);
1285:   case AMDGPU::BI__builtin_amdgcn_image_sample_cube_v4f32_f32:
1286:   case AMDGPU::BI__builtin_amdgcn_image_sample_cube_v4f16_f32:
1287:     return emitAMDGCNImageOverloadedReturnType(
1288:         *this, E, Intrinsic::amdgcn_image_sample_cube, false);
1289:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_1d_v4f32_f32:
1290:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_1d_v4f16_f32:
1291:     return emitAMDGCNImageOverloadedReturnType(
1292:         *this, E, Intrinsic::amdgcn_image_sample_lz_1d, false);
1293:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_1d_v4f32_f32:
1294:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_1d_v4f16_f32:
1295:     return emitAMDGCNImageOverloadedReturnType(
1296:         *this, E, Intrinsic::amdgcn_image_sample_l_1d, false);
1297:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_1d_v4f32_f32:
1298:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_1d_v4f16_f32:
1299:     return emitAMDGCNImageOverloadedReturnType(
1300:         *this, E, Intrinsic::amdgcn_image_sample_d_1d, false);
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1301-1325
```cpp
1301:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_2d_v4f32_f32:
1302:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_2d_v4f16_f32:
1303:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_2d_f32_f32:
1304:     return emitAMDGCNImageOverloadedReturnType(
1305:         *this, E, Intrinsic::amdgcn_image_sample_lz_2d, false);
1306:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_2d_v4f32_f32:
1307:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_2d_v4f16_f32:
1308:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_2d_f32_f32:
1309:     return emitAMDGCNImageOverloadedReturnType(
1310:         *this, E, Intrinsic::amdgcn_image_sample_l_2d, false);
1311:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_2d_v4f32_f32:
1312:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_2d_v4f16_f32:
1313:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_2d_f32_f32:
1314:     return emitAMDGCNImageOverloadedReturnType(
1315:         *this, E, Intrinsic::amdgcn_image_sample_d_2d, false);
1316:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_3d_v4f32_f32:
1317:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_3d_v4f16_f32:
1318:     return emitAMDGCNImageOverloadedReturnType(
1319:         *this, E, Intrinsic::amdgcn_image_sample_lz_3d, false);
1320:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_3d_v4f32_f32:
1321:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_3d_v4f16_f32:
1322:     return emitAMDGCNImageOverloadedReturnType(
1323:         *this, E, Intrinsic::amdgcn_image_sample_l_3d, false);
1324:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_3d_v4f32_f32:
1325:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_3d_v4f16_f32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1326-1350
```cpp
1326:     return emitAMDGCNImageOverloadedReturnType(
1327:         *this, E, Intrinsic::amdgcn_image_sample_d_3d, false);
1328:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_cube_v4f32_f32:
1329:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_cube_v4f16_f32:
1330:     return emitAMDGCNImageOverloadedReturnType(
1331:         *this, E, Intrinsic::amdgcn_image_sample_lz_cube, false);
1332:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_cube_v4f32_f32:
1333:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_cube_v4f16_f32:
1334:     return emitAMDGCNImageOverloadedReturnType(
1335:         *this, E, Intrinsic::amdgcn_image_sample_l_cube, false);
1336:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_1darray_v4f32_f32:
1337:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_1darray_v4f16_f32:
1338:     return emitAMDGCNImageOverloadedReturnType(
1339:         *this, E, Intrinsic::amdgcn_image_sample_lz_1darray, false);
1340:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_1darray_v4f32_f32:
1341:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_1darray_v4f16_f32:
1342:     return emitAMDGCNImageOverloadedReturnType(
1343:         *this, E, Intrinsic::amdgcn_image_sample_l_1darray, false);
1344:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_1darray_v4f32_f32:
1345:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_1darray_v4f16_f32:
1346:     return emitAMDGCNImageOverloadedReturnType(
1347:         *this, E, Intrinsic::amdgcn_image_sample_d_1darray, false);
1348:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_2darray_v4f32_f32:
1349:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_2darray_v4f16_f32:
1350:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_lz_2darray_f32_f32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1351-1375
```cpp
1351:     return emitAMDGCNImageOverloadedReturnType(
1352:         *this, E, Intrinsic::amdgcn_image_sample_lz_2darray, false);
1353:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_2darray_v4f32_f32:
1354:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_2darray_v4f16_f32:
1355:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_l_2darray_f32_f32:
1356:     return emitAMDGCNImageOverloadedReturnType(
1357:         *this, E, Intrinsic::amdgcn_image_sample_l_2darray, false);
1358:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_2darray_v4f32_f32:
1359:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_2darray_v4f16_f32:
1360:   case clang::AMDGPU::BI__builtin_amdgcn_image_sample_d_2darray_f32_f32:
1361:     return emitAMDGCNImageOverloadedReturnType(
1362:         *this, E, Intrinsic::amdgcn_image_sample_d_2darray, false);
1363:   case clang::AMDGPU::BI__builtin_amdgcn_image_gather4_lz_2d_v4f32_f32:
1364:     return emitAMDGCNImageOverloadedReturnType(
1365:         *this, E, Intrinsic::amdgcn_image_gather4_lz_2d, false);
1366:   case AMDGPU::BI__builtin_amdgcn_mfma_scale_f32_16x16x128_f8f6f4:
1367:   case AMDGPU::BI__builtin_amdgcn_mfma_scale_f32_32x32x64_f8f6f4: {
1368:     llvm::FixedVectorType *VT = FixedVectorType::get(Builder.getInt32Ty(), 8);
1369:     Function *F = CGM.getIntrinsic(
1370:         BuiltinID == AMDGPU::BI__builtin_amdgcn_mfma_scale_f32_32x32x64_f8f6f4
1371:             ? Intrinsic::amdgcn_mfma_scale_f32_32x32x64_f8f6f4
1372:             : Intrinsic::amdgcn_mfma_scale_f32_16x16x128_f8f6f4,
1373:         {VT, VT});
1374: 
1375:     SmallVector<Value *, 9> Args;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1376-1400
```cpp
1376:     for (unsigned I = 0, N = E->getNumArgs(); I != N; ++I)
1377:       Args.push_back(EmitScalarExpr(E->getArg(I)));
1378:     return Builder.CreateCall(F, Args);
1379:   }
1380:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32:
1381:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w32:
1382:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64:
1383:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w64:
1384:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w32:
1385:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w32:
1386:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w64:
1387:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w64:
1388:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32:
1389:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64:
1390:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w32:
1391:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w64:
1392:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32:
1393:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64:
1394:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32:
1395:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64:
1396:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32_gfx12:
1397:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64_gfx12:
1398:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w32_gfx12:
1399:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w64_gfx12:
1400:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32_gfx12:
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 1401-1425
```cpp
1401:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64_gfx12:
1402:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w32_gfx12:
1403:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w64_gfx12:
1404:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32_gfx12:
1405:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64_gfx12:
1406:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32_gfx12:
1407:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64_gfx12:
1408:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w32_gfx12:
1409:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w64_gfx12:
1410:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w32_gfx12:
1411:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w64_gfx12:
1412:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w32_gfx12:
1413:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w64_gfx12:
1414:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w32_gfx12:
1415:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w64_gfx12:
1416:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x32_iu4_w32_gfx12:
1417:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x32_iu4_w64_gfx12:
1418:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_f16_w32:
1419:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_f16_w64:
1420:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w32:
1421:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w64:
1422:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x32_f16_w32:
1423:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x32_f16_w64:
1424:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w32:
1425:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w64:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1426-1450
```cpp
1426:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w32:
1427:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w64:
1428:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w32:
1429:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w64:
1430:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w32:
1431:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w64:
1432:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w32:
1433:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w64:
1434:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w32:
1435:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w64:
1436:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w32:
1437:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w64:
1438:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w32:
1439:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w64:
1440:   // GFX1250 WMMA builtins
1441:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x4_f32:
1442:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x32_bf16:
1443:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x32_f16:
1444:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x32_f16:
1445:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x32_bf16:
1446:   case AMDGPU::BI__builtin_amdgcn_wmma_bf16f32_16x16x32_bf16:
1447:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_fp8_fp8:
1448:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_fp8_bf8:
1449:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_bf8_fp8:
1450:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_bf8_bf8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1451-1475
```cpp
1451:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_fp8_fp8:
1452:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_fp8_bf8:
1453:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_bf8_fp8:
1454:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_bf8_bf8:
1455:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_fp8_fp8:
1456:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_fp8_bf8:
1457:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_bf8_fp8:
1458:   case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_bf8_bf8:
1459:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_fp8_fp8:
1460:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_fp8_bf8:
1461:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_bf8_fp8:
1462:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_bf8_bf8:
1463:   case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x64_iu8:
1464:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_f8f6f4:
1465:   case AMDGPU::BI__builtin_amdgcn_wmma_f32_32x16x128_f4:
1466:   case AMDGPU::BI__builtin_amdgcn_wmma_scale_f32_16x16x128_f8f6f4:
1467:   case AMDGPU::BI__builtin_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4:
1468:   case AMDGPU::BI__builtin_amdgcn_wmma_scale_f32_32x16x128_f4:
1469:   case AMDGPU::BI__builtin_amdgcn_wmma_scale16_f32_32x16x128_f4:
1470:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x64_f16:
1471:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x64_bf16:
1472:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x64_f16:
1473:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x64_bf16:
1474:   case AMDGPU::BI__builtin_amdgcn_swmmac_bf16f32_16x16x64_bf16:
1475:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_fp8_fp8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1476-1500
```cpp
1476:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_fp8_bf8:
1477:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_bf8_fp8:
1478:   case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_bf8_bf8:
1479:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_fp8_fp8:
1480:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_fp8_bf8:
1481:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_bf8_fp8:
1482:   case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_bf8_bf8:
1483:   case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x128_iu8: {
1484: 
1485:     // These operations perform a matrix multiplication and accumulation of
1486:     // the form:
1487:     //             D = A * B + C
1488:     // We need to specify one type for matrices AB and one for matrices CD.
1489:     // Sparse matrix operations can have different types for A and B as well as
1490:     // an additional type for sparsity index.
1491:     // Destination type should be put before types used for source operands.
1492:     SmallVector<unsigned, 2> ArgsForMatchingMatrixTypes;
1493:     // On GFX12, the intrinsics with 16-bit accumulator use a packed layout.
1494:     // There is no need for the variable opsel argument, so always set it to
1495:     // "false".
1496:     bool AppendFalseForOpselArg = false;
1497:     unsigned BuiltinWMMAOp;
1498:     // Need return type when D and C are of different types.
1499:     bool NeedReturnType = false;
1500:     // Need to remove unused neg modifiers.
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1501-1525
```cpp
1501:     bool RemoveABNeg = false;
1502: 
1503:     switch (BuiltinID) {
1504:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w32:
1505:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w64:
1506:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w32_gfx12:
1507:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_f16_w64_gfx12:
1508:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1509:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x16_f16;
1510:       break;
1511:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32:
1512:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64:
1513:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32_gfx12:
1514:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64_gfx12:
1515:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1516:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x16_bf16;
1517:       break;
1518:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w32_gfx12:
1519:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w64_gfx12:
1520:       AppendFalseForOpselArg = true;
1521:       [[fallthrough]];
1522:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w32:
1523:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_w64:
1524:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1525:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x16_f16;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 1526-1550
```cpp
1526:       break;
1527:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32_gfx12:
1528:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64_gfx12:
1529:       AppendFalseForOpselArg = true;
1530:       [[fallthrough]];
1531:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32:
1532:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64:
1533:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1534:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_bf16_16x16x16_bf16;
1535:       break;
1536:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w32:
1537:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w64:
1538:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1539:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x16_f16_tied;
1540:       break;
1541:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w32:
1542:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w64:
1543:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1544:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_bf16_16x16x16_bf16_tied;
1545:       break;
1546:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32:
1547:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64:
1548:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32_gfx12:
1549:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64_gfx12:
1550:       ArgsForMatchingMatrixTypes = {4, 1}; // CD, AB
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1551-1575
```cpp
1551:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_i32_16x16x16_iu8;
1552:       break;
1553:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32:
1554:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64:
1555:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32_gfx12:
1556:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64_gfx12:
1557:       ArgsForMatchingMatrixTypes = {4, 1}; // CD, AB
1558:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_i32_16x16x16_iu4;
1559:       break;
1560:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w32_gfx12:
1561:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w64_gfx12:
1562:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1563:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x16_fp8_fp8;
1564:       break;
1565:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w32_gfx12:
1566:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w64_gfx12:
1567:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1568:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x16_fp8_bf8;
1569:       break;
1570:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w32_gfx12:
1571:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w64_gfx12:
1572:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1573:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x16_bf8_fp8;
1574:       break;
1575:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w32_gfx12:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1576-1600
```cpp
1576:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w64_gfx12:
1577:       ArgsForMatchingMatrixTypes = {2, 0}; // CD, AB
1578:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x16_bf8_bf8;
1579:       break;
1580:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x32_iu4_w32_gfx12:
1581:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x32_iu4_w64_gfx12:
1582:       ArgsForMatchingMatrixTypes = {4, 1}; // CD, AB
1583:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_i32_16x16x32_iu4;
1584:       break;
1585:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_f16_w32:
1586:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_f16_w64:
1587:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1588:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x32_f16;
1589:       break;
1590:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w32:
1591:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w64:
1592:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1593:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x32_bf16;
1594:       break;
1595:     case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x32_f16_w32:
1596:     case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x32_f16_w64:
1597:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1598:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f16_16x16x32_f16;
1599:       break;
1600:     case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1601-1625
```cpp
1601:     case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w64:
1602:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1603:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_bf16_16x16x32_bf16;
1604:       break;
1605:     case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w32:
1606:     case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w64:
1607:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5}; // CD, A, B, Index
1608:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_i32_16x16x32_iu8;
1609:       break;
1610:     case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w32:
1611:     case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w64:
1612:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5}; // CD, A, B, Index
1613:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_i32_16x16x32_iu4;
1614:       break;
1615:     case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w32:
1616:     case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w64:
1617:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5}; // CD, A, B, Index
1618:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_i32_16x16x64_iu4;
1619:       break;
1620:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w32:
1621:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w64:
1622:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1623:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x32_fp8_fp8;
1624:       break;
1625:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1626-1650
```cpp
1626:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w64:
1627:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1628:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x32_fp8_bf8;
1629:       break;
1630:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w32:
1631:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w64:
1632:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1633:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x32_bf8_fp8;
1634:       break;
1635:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w32:
1636:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w64:
1637:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3}; // CD, A, B, Index
1638:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x32_bf8_bf8;
1639:       break;
1640:     // GFX1250 WMMA builtins
1641:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x4_f32:
1642:       ArgsForMatchingMatrixTypes = {3, 0};
1643:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x4_f32;
1644:       RemoveABNeg = true;
1645:       break;
1646:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x32_bf16:
1647:       ArgsForMatchingMatrixTypes = {3, 0};
1648:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x32_bf16;
1649:       RemoveABNeg = true;
1650:       break;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1651-1675
```cpp
1651:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x32_f16:
1652:       ArgsForMatchingMatrixTypes = {3, 0};
1653:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x32_f16;
1654:       RemoveABNeg = true;
1655:       break;
1656:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x32_f16:
1657:       ArgsForMatchingMatrixTypes = {3, 0};
1658:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x32_f16;
1659:       RemoveABNeg = true;
1660:       break;
1661:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16_16x16x32_bf16:
1662:       ArgsForMatchingMatrixTypes = {3, 0};
1663:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_bf16_16x16x32_bf16;
1664:       RemoveABNeg = true;
1665:       break;
1666:     case AMDGPU::BI__builtin_amdgcn_wmma_bf16f32_16x16x32_bf16:
1667:       NeedReturnType = true;
1668:       ArgsForMatchingMatrixTypes = {0, 3};
1669:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_bf16f32_16x16x32_bf16;
1670:       RemoveABNeg = true;
1671:       break;
1672:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_fp8_fp8:
1673:       ArgsForMatchingMatrixTypes = {3, 0};
1674:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x64_fp8_fp8;
1675:       break;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1676-1700
```cpp
1676:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_fp8_bf8:
1677:       ArgsForMatchingMatrixTypes = {3, 0};
1678:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x64_fp8_bf8;
1679:       break;
1680:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_bf8_fp8:
1681:       ArgsForMatchingMatrixTypes = {3, 0};
1682:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x64_bf8_fp8;
1683:       break;
1684:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x64_bf8_bf8:
1685:       ArgsForMatchingMatrixTypes = {3, 0};
1686:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x64_bf8_bf8;
1687:       break;
1688:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_fp8_fp8:
1689:       ArgsForMatchingMatrixTypes = {3, 0};
1690:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x64_fp8_fp8;
1691:       break;
1692:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_fp8_bf8:
1693:       ArgsForMatchingMatrixTypes = {3, 0};
1694:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x64_fp8_bf8;
1695:       break;
1696:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_bf8_fp8:
1697:       ArgsForMatchingMatrixTypes = {3, 0};
1698:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x64_bf8_fp8;
1699:       break;
1700:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x64_bf8_bf8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1701-1725
```cpp
1701:       ArgsForMatchingMatrixTypes = {3, 0};
1702:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x64_bf8_bf8;
1703:       break;
1704:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_fp8_fp8:
1705:       ArgsForMatchingMatrixTypes = {3, 0};
1706:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x128_fp8_fp8;
1707:       break;
1708:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_fp8_bf8:
1709:       ArgsForMatchingMatrixTypes = {3, 0};
1710:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x128_fp8_bf8;
1711:       break;
1712:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_bf8_fp8:
1713:       ArgsForMatchingMatrixTypes = {3, 0};
1714:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x128_bf8_fp8;
1715:       break;
1716:     case AMDGPU::BI__builtin_amdgcn_wmma_f16_16x16x128_bf8_bf8:
1717:       ArgsForMatchingMatrixTypes = {3, 0};
1718:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f16_16x16x128_bf8_bf8;
1719:       break;
1720:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_fp8_fp8:
1721:       ArgsForMatchingMatrixTypes = {3, 0};
1722:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x128_fp8_fp8;
1723:       break;
1724:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_fp8_bf8:
1725:       ArgsForMatchingMatrixTypes = {3, 0};
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1726-1750
```cpp
1726:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x128_fp8_bf8;
1727:       break;
1728:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_bf8_fp8:
1729:       ArgsForMatchingMatrixTypes = {3, 0};
1730:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x128_bf8_fp8;
1731:       break;
1732:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_bf8_bf8:
1733:       ArgsForMatchingMatrixTypes = {3, 0};
1734:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x128_bf8_bf8;
1735:       break;
1736:     case AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x64_iu8:
1737:       ArgsForMatchingMatrixTypes = {4, 1};
1738:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_i32_16x16x64_iu8;
1739:       break;
1740:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_16x16x128_f8f6f4:
1741:       ArgsForMatchingMatrixTypes = {5, 1, 3};
1742:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_16x16x128_f8f6f4;
1743:       break;
1744:     case AMDGPU::BI__builtin_amdgcn_wmma_scale_f32_16x16x128_f8f6f4:
1745:       ArgsForMatchingMatrixTypes = {5, 1, 3};
1746:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_scale_f32_16x16x128_f8f6f4;
1747:       break;
1748:     case AMDGPU::BI__builtin_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4:
1749:       ArgsForMatchingMatrixTypes = {5, 1, 3};
1750:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_scale16_f32_16x16x128_f8f6f4;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1751-1775
```cpp
1751:       break;
1752:     case AMDGPU::BI__builtin_amdgcn_wmma_f32_32x16x128_f4:
1753:       ArgsForMatchingMatrixTypes = {3, 0, 1};
1754:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_f32_32x16x128_f4;
1755:       break;
1756:     case AMDGPU::BI__builtin_amdgcn_wmma_scale_f32_32x16x128_f4:
1757:       ArgsForMatchingMatrixTypes = {3, 0, 1};
1758:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_scale_f32_32x16x128_f4;
1759:       break;
1760:     case AMDGPU::BI__builtin_amdgcn_wmma_scale16_f32_32x16x128_f4:
1761:       ArgsForMatchingMatrixTypes = {3, 0, 1};
1762:       BuiltinWMMAOp = Intrinsic::amdgcn_wmma_scale16_f32_32x16x128_f4;
1763:       break;
1764:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x64_f16:
1765:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5};
1766:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x64_f16;
1767:       break;
1768:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x64_bf16:
1769:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5};
1770:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x64_bf16;
1771:       break;
1772:     case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x64_f16:
1773:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5};
1774:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f16_16x16x64_f16;
1775:       break;
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1776-1800
```cpp
1776:     case AMDGPU::BI__builtin_amdgcn_swmmac_bf16_16x16x64_bf16:
1777:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5};
1778:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_bf16_16x16x64_bf16;
1779:       break;
1780:     case AMDGPU::BI__builtin_amdgcn_swmmac_bf16f32_16x16x64_bf16:
1781:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5};
1782:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_bf16f32_16x16x64_bf16;
1783:       break;
1784:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_fp8_fp8:
1785:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1786:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x128_fp8_fp8;
1787:       break;
1788:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_fp8_bf8:
1789:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1790:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x128_fp8_bf8;
1791:       break;
1792:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_bf8_fp8:
1793:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1794:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x128_bf8_fp8;
1795:       break;
1796:     case AMDGPU::BI__builtin_amdgcn_swmmac_f32_16x16x128_bf8_bf8:
1797:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1798:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f32_16x16x128_bf8_bf8;
1799:       break;
1800:     case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_fp8_fp8:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1801-1825
```cpp
1801:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1802:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f16_16x16x128_fp8_fp8;
1803:       break;
1804:     case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_fp8_bf8:
1805:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1806:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f16_16x16x128_fp8_bf8;
1807:       break;
1808:     case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_bf8_fp8:
1809:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1810:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f16_16x16x128_bf8_fp8;
1811:       break;
1812:     case AMDGPU::BI__builtin_amdgcn_swmmac_f16_16x16x128_bf8_bf8:
1813:       ArgsForMatchingMatrixTypes = {2, 0, 1, 3};
1814:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_f16_16x16x128_bf8_bf8;
1815:       break;
1816:     case AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x128_iu8:
1817:       ArgsForMatchingMatrixTypes = {4, 1, 3, 5};
1818:       BuiltinWMMAOp = Intrinsic::amdgcn_swmmac_i32_16x16x128_iu8;
1819:       break;
1820:     }
1821: 
1822:     SmallVector<Value *, 6> Args;
1823:     for (int i = 0, e = E->getNumArgs(); i != e; ++i) {
1824:       // Remove unused neg modifiers.
1825:       if (RemoveABNeg && (i == 0 || i == 2))
```
- **EN**: This block uses control flow (if, for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, for, case）细化 目标内建函数降级 行为。

### Lines 1826-1850
```cpp
1826:         continue;
1827:       Args.push_back(EmitScalarExpr(E->getArg(i)));
1828:     }
1829:     if (AppendFalseForOpselArg)
1830:       Args.push_back(Builder.getFalse());
1831: 
1832:     // Handle the optional clamp argument of the following two builtins.
1833:     if (BuiltinID == AMDGPU::BI__builtin_amdgcn_wmma_i32_16x16x64_iu8) {
1834:       if (Args.size() == 7)
1835:         Args.push_back(Builder.getFalse());
1836:       assert(Args.size() == 8 && "Expected 8 arguments");
1837:       Args[7] = Builder.CreateZExtOrTrunc(Args[7], Builder.getInt1Ty());
1838:     } else if (BuiltinID ==
1839:                AMDGPU::BI__builtin_amdgcn_swmmac_i32_16x16x128_iu8) {
1840:       if (Args.size() == 8)
1841:         Args.push_back(Builder.getFalse());
1842:       assert(Args.size() == 9 && "Expected 9 arguments");
1843:       Args[8] = Builder.CreateZExtOrTrunc(Args[8], Builder.getInt1Ty());
1844:     }
1845: 
1846:     SmallVector<llvm::Type *, 6> ArgTypes;
1847:     if (NeedReturnType)
1848:       ArgTypes.push_back(ConvertType(E->getType()));
1849:     for (auto ArgIdx : ArgsForMatchingMatrixTypes)
1850:       ArgTypes.push_back(Args[ArgIdx]->getType());
```
- **EN**: This block uses control flow (if, for) to specialize target builtin lowering; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块通过控制流（if, for）细化 目标内建函数降级 行为；使用断言或不可达标记保护关键不变量。

### Lines 1851-1875
```cpp
1851: 
1852:     Function *F = CGM.getIntrinsic(BuiltinWMMAOp, ArgTypes);
1853:     return Builder.CreateCall(F, Args);
1854:   }
1855:   // amdgcn workgroup size
1856:   case AMDGPU::BI__builtin_amdgcn_workgroup_size_x:
1857:     return EmitAMDGPUWorkGroupSize(*this, 0);
1858:   case AMDGPU::BI__builtin_amdgcn_workgroup_size_y:
1859:     return EmitAMDGPUWorkGroupSize(*this, 1);
1860:   case AMDGPU::BI__builtin_amdgcn_workgroup_size_z:
1861:     return EmitAMDGPUWorkGroupSize(*this, 2);
1862: 
1863:   // amdgcn grid size
1864:   case AMDGPU::BI__builtin_amdgcn_grid_size_x:
1865:     return EmitAMDGPUGridSize(*this, 0);
1866:   case AMDGPU::BI__builtin_amdgcn_grid_size_y:
1867:     return EmitAMDGPUGridSize(*this, 1);
1868:   case AMDGPU::BI__builtin_amdgcn_grid_size_z:
1869:     return EmitAMDGPUGridSize(*this, 2);
1870: 
1871:   // r600 intrinsics
1872:   case AMDGPU::BI__builtin_r600_recipsqrt_ieee:
1873:   case AMDGPU::BI__builtin_r600_recipsqrt_ieeef:
1874:     return emitBuiltinWithOneOverloadedType<1>(*this, E,
1875:                                                Intrinsic::r600_recipsqrt_ieee);
```
- **EN**: This block spells out callable entry points like `EmitAMDGPUWorkGroupSize`, `EmitAMDGPUGridSize`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `EmitAMDGPUWorkGroupSize`, `EmitAMDGPUGridSize`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1876-1900
```cpp
1876:   case AMDGPU::BI__builtin_amdgcn_alignbit: {
1877:     llvm::Value *Src0 = EmitScalarExpr(E->getArg(0));
1878:     llvm::Value *Src1 = EmitScalarExpr(E->getArg(1));
1879:     llvm::Value *Src2 = EmitScalarExpr(E->getArg(2));
1880:     Function *F = CGM.getIntrinsic(Intrinsic::fshr, Src0->getType());
1881:     return Builder.CreateCall(F, { Src0, Src1, Src2 });
1882:   }
1883:   case AMDGPU::BI__builtin_amdgcn_fence: {
1884:     ProcessOrderScopeAMDGCN(EmitScalarExpr(E->getArg(0)),
1885:                             EmitScalarExpr(E->getArg(1)), AO, SSID);
1886:     FenceInst *Fence = Builder.CreateFence(AO, SSID);
1887:     if (E->getNumArgs() > 2)
1888:       AddAMDGPUFenceAddressSpaceMMRA(Fence, E);
1889:     return Fence;
1890:   }
1891:   case AMDGPU::BI__builtin_amdgcn_atomic_inc32:
1892:   case AMDGPU::BI__builtin_amdgcn_atomic_inc64:
1893:   case AMDGPU::BI__builtin_amdgcn_atomic_dec32:
1894:   case AMDGPU::BI__builtin_amdgcn_atomic_dec64:
1895:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_f64:
1896:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_f32:
1897:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_v2f16:
1898:   case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_v2bf16:
1899:   case AMDGPU::BI__builtin_amdgcn_ds_faddf:
1900:   case AMDGPU::BI__builtin_amdgcn_ds_fminf:
```
- **EN**: This block defines callable entry points like `ProcessOrderScopeAMDGCN`; uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `ProcessOrderScopeAMDGCN`；通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 1901-1925
```cpp
1901:   case AMDGPU::BI__builtin_amdgcn_ds_fmaxf:
1902:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_f32:
1903:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_f64:
1904:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_v2f16:
1905:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_v2f16:
1906:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_f32:
1907:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_f64:
1908:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_v2bf16:
1909:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_v2bf16:
1910:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fmin_f64:
1911:   case AMDGPU::BI__builtin_amdgcn_global_atomic_fmax_f64:
1912:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fmin_f64:
1913:   case AMDGPU::BI__builtin_amdgcn_flat_atomic_fmax_f64: {
1914:     llvm::AtomicRMWInst::BinOp BinOp;
1915:     switch (BuiltinID) {
1916:     case AMDGPU::BI__builtin_amdgcn_atomic_inc32:
1917:     case AMDGPU::BI__builtin_amdgcn_atomic_inc64:
1918:       BinOp = llvm::AtomicRMWInst::UIncWrap;
1919:       break;
1920:     case AMDGPU::BI__builtin_amdgcn_atomic_dec32:
1921:     case AMDGPU::BI__builtin_amdgcn_atomic_dec64:
1922:       BinOp = llvm::AtomicRMWInst::UDecWrap;
1923:       break;
1924:     case AMDGPU::BI__builtin_amdgcn_ds_faddf:
1925:     case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_f64:
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 1926-1950
```cpp
1926:     case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_f32:
1927:     case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_v2f16:
1928:     case AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_v2bf16:
1929:     case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_f32:
1930:     case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_f64:
1931:     case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_v2f16:
1932:     case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_v2f16:
1933:     case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_f32:
1934:     case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_f64:
1935:     case AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_v2bf16:
1936:     case AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_v2bf16:
1937:       BinOp = llvm::AtomicRMWInst::FAdd;
1938:       break;
1939:     case AMDGPU::BI__builtin_amdgcn_ds_fminf:
1940:     case AMDGPU::BI__builtin_amdgcn_global_atomic_fmin_f64:
1941:     case AMDGPU::BI__builtin_amdgcn_flat_atomic_fmin_f64:
1942:       BinOp = llvm::AtomicRMWInst::FMin;
1943:       break;
1944:     case AMDGPU::BI__builtin_amdgcn_global_atomic_fmax_f64:
1945:     case AMDGPU::BI__builtin_amdgcn_flat_atomic_fmax_f64:
1946:     case AMDGPU::BI__builtin_amdgcn_ds_fmaxf:
1947:       BinOp = llvm::AtomicRMWInst::FMax;
1948:       break;
1949:     }
1950: 
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 1951-1975
```cpp
1951:     Address Ptr = CheckAtomicAlignment(*this, E);
1952:     Value *Val = EmitScalarExpr(E->getArg(1));
1953:     llvm::Type *OrigTy = Val->getType();
1954:     QualType PtrTy = E->getArg(0)->IgnoreImpCasts()->getType();
1955: 
1956:     bool Volatile;
1957: 
1958:     if (BuiltinID == AMDGPU::BI__builtin_amdgcn_ds_faddf ||
1959:         BuiltinID == AMDGPU::BI__builtin_amdgcn_ds_fminf ||
1960:         BuiltinID == AMDGPU::BI__builtin_amdgcn_ds_fmaxf) {
1961:       // __builtin_amdgcn_ds_faddf/fminf/fmaxf has an explicit volatile argument
1962:       Volatile =
1963:           cast<ConstantInt>(EmitScalarExpr(E->getArg(4)))->getZExtValue();
1964:     } else {
1965:       // Infer volatile from the passed type.
1966:       Volatile =
1967:           PtrTy->castAs<PointerType>()->getPointeeType().isVolatileQualified();
1968:     }
1969: 
1970:     if (E->getNumArgs() >= 4) {
1971:       // Some of the builtins have explicit ordering and scope arguments.
1972:       ProcessOrderScopeAMDGCN(EmitScalarExpr(E->getArg(2)),
1973:                               EmitScalarExpr(E->getArg(3)), AO, SSID);
1974:     } else {
1975:       // Most of the builtins do not have syncscope/order arguments. For DS
```
- **EN**: This block defines callable entry points like `ProcessOrderScopeAMDGCN`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `ProcessOrderScopeAMDGCN`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 1976-2000
```cpp
1976:       // atomics the scope doesn't really matter, as they implicitly operate at
1977:       // workgroup scope.
1978:       //
1979:       // The global/flat cases need to use agent scope to consistently produce
1980:       // the native instruction instead of a cmpxchg expansion.
1981:       if (getTarget().getTriple().isSPIRV())
1982:         SSID = getLLVMContext().getOrInsertSyncScopeID("device");
1983:       else
1984:         SSID = getLLVMContext().getOrInsertSyncScopeID("agent");
1985:       AO = AtomicOrdering::Monotonic;
1986: 
1987:       // The v2bf16 builtin uses i16 instead of a natural bfloat type.
1988:       if (BuiltinID == AMDGPU::BI__builtin_amdgcn_ds_atomic_fadd_v2bf16 ||
1989:           BuiltinID == AMDGPU::BI__builtin_amdgcn_global_atomic_fadd_v2bf16 ||
1990:           BuiltinID == AMDGPU::BI__builtin_amdgcn_flat_atomic_fadd_v2bf16) {
1991:         llvm::Type *V2BF16Ty = FixedVectorType::get(
1992:             llvm::Type::getBFloatTy(Builder.getContext()), 2);
1993:         Val = Builder.CreateBitCast(Val, V2BF16Ty);
1994:       }
1995:     }
1996: 
1997:     llvm::AtomicRMWInst *RMW =
1998:         Builder.CreateAtomicRMW(BinOp, Ptr, Val, AO, SSID);
1999:     if (Volatile)
2000:       RMW->setVolatile(true);
```
- **EN**: This block defines callable entry points like `getBFloatTy`; uses control flow (if) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getBFloatTy`；通过控制流（if）细化 目标内建函数降级 行为。

### Lines 2001-2025
```cpp
2001: 
2002:     unsigned AddrSpace = Ptr.getType()->getAddressSpace();
2003:     if (AddrSpace != llvm::AMDGPUAS::LOCAL_ADDRESS) {
2004:       // Most targets require "amdgpu.no.fine.grained.memory" to emit the native
2005:       // instruction for flat and global operations.
2006:       llvm::MDTuple *EmptyMD = MDNode::get(getLLVMContext(), {});
2007:       RMW->setMetadata("amdgpu.no.fine.grained.memory", EmptyMD);
2008: 
2009:       // Most targets require "amdgpu.ignore.denormal.mode" to emit the native
2010:       // instruction, but this only matters for float fadd.
2011:       if (BinOp == llvm::AtomicRMWInst::FAdd && Val->getType()->isFloatTy())
2012:         RMW->setMetadata("amdgpu.ignore.denormal.mode", EmptyMD);
2013:     }
2014: 
2015:     return Builder.CreateBitCast(RMW, OrigTy);
2016:   }
2017:   case AMDGPU::BI__builtin_amdgcn_s_sendmsg_rtn:
2018:   case AMDGPU::BI__builtin_amdgcn_s_sendmsg_rtnl: {
2019:     llvm::Value *Arg = EmitScalarExpr(E->getArg(0));
2020:     llvm::Type *ResultType = ConvertType(E->getType());
2021:     // s_sendmsg_rtn is mangled using return type only.
2022:     Function *F =
2023:         CGM.getIntrinsic(Intrinsic::amdgcn_s_sendmsg_rtn, {ResultType});
2024:     return Builder.CreateCall(F, {Arg});
2025:   }
```
- **EN**: This block uses control flow (if, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（if, case）细化 目标内建函数降级 行为。

### Lines 2026-2050
```cpp
2026:   case AMDGPU::BI__builtin_amdgcn_permlane16_swap:
2027:   case AMDGPU::BI__builtin_amdgcn_permlane32_swap: {
2028:     // Because builtin types are limited, and the intrinsic uses a struct/pair
2029:     // output, marshal the pair-of-i32 to <2 x i32>.
2030:     Value *VDstOld = EmitScalarExpr(E->getArg(0));
2031:     Value *VSrcOld = EmitScalarExpr(E->getArg(1));
2032:     Value *FI = EmitScalarExpr(E->getArg(2));
2033:     Value *BoundCtrl = EmitScalarExpr(E->getArg(3));
2034:     Function *F =
2035:         CGM.getIntrinsic(BuiltinID == AMDGPU::BI__builtin_amdgcn_permlane16_swap
2036:                              ? Intrinsic::amdgcn_permlane16_swap
2037:                              : Intrinsic::amdgcn_permlane32_swap);
2038:     llvm::CallInst *Call =
2039:         Builder.CreateCall(F, {VDstOld, VSrcOld, FI, BoundCtrl});
2040: 
2041:     llvm::Value *Elt0 = Builder.CreateExtractValue(Call, 0);
2042:     llvm::Value *Elt1 = Builder.CreateExtractValue(Call, 1);
2043: 
2044:     llvm::Type *ResultType = ConvertType(E->getType());
2045: 
2046:     llvm::Value *Insert0 = Builder.CreateInsertElement(
2047:         llvm::PoisonValue::get(ResultType), Elt0, UINT64_C(0));
2048:     llvm::Value *AsVector =
2049:         Builder.CreateInsertElement(Insert0, Elt1, UINT64_C(1));
2050:     return AsVector;
```
- **EN**: This block defines callable entry points like `get`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `get`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2051-2075
```cpp
2051:   }
2052:   case AMDGPU::BI__builtin_amdgcn_bitop3_b32:
2053:   case AMDGPU::BI__builtin_amdgcn_bitop3_b16:
2054:     return emitBuiltinWithOneOverloadedType<4>(*this, E,
2055:                                                Intrinsic::amdgcn_bitop3);
2056:   case AMDGPU::BI__builtin_amdgcn_make_buffer_rsrc: {
2057:     // TODO: LLVM has this overloaded to allow for fat pointers, but since
2058:     // those haven't been plumbed through to Clang yet, default to creating the
2059:     // resource type.
2060:     SmallVector<Value *, 4> Args;
2061:     for (unsigned I = 0; I < 4; ++I)
2062:       Args.push_back(EmitScalarExpr(E->getArg(I)));
2063:     llvm::PointerType *RetTy = llvm::PointerType::get(
2064:         Builder.getContext(), llvm::AMDGPUAS::BUFFER_RESOURCE);
2065:     Function *F = CGM.getIntrinsic(Intrinsic::amdgcn_make_buffer_rsrc,
2066:                                    {RetTy, Args[0]->getType()});
2067:     return Builder.CreateCall(F, Args);
2068:   }
2069:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b8:
2070:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b16:
2071:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b32:
2072:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b64:
2073:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b96:
2074:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_b128:
2075:     return emitBuiltinWithOneOverloadedType<5>(
```
- **EN**: This block uses control flow (for, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（for, case）细化 目标内建函数降级 行为。

### Lines 2076-2100
```cpp
2076:         *this, E, Intrinsic::amdgcn_raw_ptr_buffer_store);
2077:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_format_v4f32:
2078:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_store_format_v4f16:
2079:     return emitBuiltinWithOneOverloadedType<5>(
2080:         *this, E, Intrinsic::amdgcn_raw_ptr_buffer_store_format);
2081:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b8:
2082:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b16:
2083:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b32:
2084:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b64:
2085:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b96:
2086:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b128: {
2087:     llvm::Type *RetTy = nullptr;
2088:     switch (BuiltinID) {
2089:     case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b8:
2090:       RetTy = Int8Ty;
2091:       break;
2092:     case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b16:
2093:       RetTy = Int16Ty;
2094:       break;
2095:     case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b32:
2096:       RetTy = Int32Ty;
2097:       break;
2098:     case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b64:
2099:       RetTy = llvm::FixedVectorType::get(Int32Ty, /*NumElements=*/2);
2100:       break;
```
- **EN**: This block uses control flow (switch, case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（switch, case）细化 目标内建函数降级 行为。

### Lines 2101-2125
```cpp
2101:     case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b96:
2102:       RetTy = llvm::FixedVectorType::get(Int32Ty, /*NumElements=*/3);
2103:       break;
2104:     case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_b128:
2105:       RetTy = llvm::FixedVectorType::get(Int32Ty, /*NumElements=*/4);
2106:       break;
2107:     }
2108:     Function *F =
2109:         CGM.getIntrinsic(Intrinsic::amdgcn_raw_ptr_buffer_load, RetTy);
2110:     return Builder.CreateCall(
2111:         F, {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1)),
2112:             EmitScalarExpr(E->getArg(2)), EmitScalarExpr(E->getArg(3))});
2113:   }
2114:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_format_v4f32:
2115:   case AMDGPU::BI__builtin_amdgcn_raw_buffer_load_format_v4f16: {
2116:     llvm::Type *RetTy = ConvertType(E->getType());
2117:     Function *F =
2118:         CGM.getIntrinsic(Intrinsic::amdgcn_raw_ptr_buffer_load_format, {RetTy});
2119: 
2120:     return Builder.CreateCall(
2121:         F, {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1)),
2122:             EmitScalarExpr(E->getArg(2)), EmitScalarExpr(E->getArg(3))});
2123:   }
2124:   case AMDGPU::BI__builtin_amdgcn_struct_buffer_store_format_v4f32:
2125:   case AMDGPU::BI__builtin_amdgcn_struct_buffer_store_format_v4f16:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2126-2150
```cpp
2126:     return emitBuiltinWithOneOverloadedType<6>(
2127:         *this, E, Intrinsic::amdgcn_struct_ptr_buffer_store_format);
2128:   case AMDGPU::BI__builtin_amdgcn_struct_buffer_load_format_v4f32:
2129:   case AMDGPU::BI__builtin_amdgcn_struct_buffer_load_format_v4f16: {
2130:     llvm::Type *RetTy = ConvertType(E->getType());
2131:     Function *F = CGM.getIntrinsic(
2132:         Intrinsic::amdgcn_struct_ptr_buffer_load_format, {RetTy});
2133: 
2134:     return Builder.CreateCall(
2135:         F, {EmitScalarExpr(E->getArg(0)), EmitScalarExpr(E->getArg(1)),
2136:             EmitScalarExpr(E->getArg(2)), EmitScalarExpr(E->getArg(3)),
2137:             EmitScalarExpr(E->getArg(4))});
2138:   }
2139:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_add_i32:
2140:     return emitBuiltinWithOneOverloadedType<5>(
2141:         *this, E, Intrinsic::amdgcn_raw_ptr_buffer_atomic_add);
2142:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_f32:
2143:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_v2f16:
2144:     return emitBuiltinWithOneOverloadedType<5>(
2145:         *this, E, Intrinsic::amdgcn_raw_ptr_buffer_atomic_fadd);
2146:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f32:
2147:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f64:
2148:     return emitBuiltinWithOneOverloadedType<5>(
2149:         *this, E, Intrinsic::amdgcn_raw_ptr_buffer_atomic_fmin);
2150:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f32:
```
- **EN**: This block uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2151-2175
```cpp
2151:   case AMDGPU::BI__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f64:
2152:     return emitBuiltinWithOneOverloadedType<5>(
2153:         *this, E, Intrinsic::amdgcn_raw_ptr_buffer_atomic_fmax);
2154:   case AMDGPU::BI__builtin_amdgcn_s_prefetch_data:
2155:     return emitBuiltinWithOneOverloadedType<2>(
2156:         *this, E, Intrinsic::amdgcn_s_prefetch_data);
2157:   case Builtin::BIlogbf:
2158:   case Builtin::BI__builtin_logbf: {
2159:     Value *Src0 = EmitScalarExpr(E->getArg(0));
2160:     Function *FrExpFunc = CGM.getIntrinsic(
2161:         Intrinsic::frexp, {Src0->getType(), Builder.getInt32Ty()});
2162:     CallInst *FrExp = Builder.CreateCall(FrExpFunc, Src0);
2163:     Value *Exp = Builder.CreateExtractValue(FrExp, 1);
2164:     Value *Add = Builder.CreateAdd(
2165:         Exp, ConstantInt::getSigned(Exp->getType(), -1), "", false, true);
2166:     Value *SIToFP = Builder.CreateSIToFP(Add, Builder.getFloatTy());
2167:     Value *Fabs =
2168:         emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::fabs);
2169:     Value *FCmpONE = Builder.CreateFCmpONE(
2170:         Fabs, ConstantFP::getInfinity(Builder.getFloatTy()));
2171:     Value *Sel1 = Builder.CreateSelect(FCmpONE, SIToFP, Fabs);
2172:     Value *FCmpOEQ =
2173:         Builder.CreateFCmpOEQ(Src0, ConstantFP::getZero(Builder.getFloatTy()));
2174:     Value *Sel2 = Builder.CreateSelect(
2175:         FCmpOEQ,
```
- **EN**: This block defines callable entry points like `getSigned`, `getInfinity`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getSigned`, `getInfinity`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2176-2200
```cpp
2176:         ConstantFP::getInfinity(Builder.getFloatTy(), /*Negative=*/true), Sel1);
2177:     return Sel2;
2178:   }
2179:   case Builtin::BIlogb:
2180:   case Builtin::BI__builtin_logb: {
2181:     Value *Src0 = EmitScalarExpr(E->getArg(0));
2182:     Function *FrExpFunc = CGM.getIntrinsic(
2183:         Intrinsic::frexp, {Src0->getType(), Builder.getInt32Ty()});
2184:     CallInst *FrExp = Builder.CreateCall(FrExpFunc, Src0);
2185:     Value *Exp = Builder.CreateExtractValue(FrExp, 1);
2186:     Value *Add = Builder.CreateAdd(
2187:         Exp, ConstantInt::getSigned(Exp->getType(), -1), "", false, true);
2188:     Value *SIToFP = Builder.CreateSIToFP(Add, Builder.getDoubleTy());
2189:     Value *Fabs =
2190:         emitBuiltinWithOneOverloadedType<1>(*this, E, Intrinsic::fabs);
2191:     Value *FCmpONE = Builder.CreateFCmpONE(
2192:         Fabs, ConstantFP::getInfinity(Builder.getDoubleTy()));
2193:     Value *Sel1 = Builder.CreateSelect(FCmpONE, SIToFP, Fabs);
2194:     Value *FCmpOEQ =
2195:         Builder.CreateFCmpOEQ(Src0, ConstantFP::getZero(Builder.getDoubleTy()));
2196:     Value *Sel2 = Builder.CreateSelect(
2197:         FCmpOEQ,
2198:         ConstantFP::getInfinity(Builder.getDoubleTy(), /*Negative=*/true),
2199:         Sel1);
2200:     return Sel2;
```
- **EN**: This block defines callable entry points like `getInfinity`, `getSigned`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块定义可调用入口，例如 `getInfinity`, `getSigned`；通过控制流（case）细化 目标内建函数降级 行为。

### Lines 2201-2223
```cpp
2201:   }
2202:   case Builtin::BIscalbnf:
2203:   case Builtin::BI__builtin_scalbnf:
2204:   case Builtin::BIscalbn:
2205:   case Builtin::BI__builtin_scalbn:
2206:     return emitBinaryExpMaybeConstrainedFPBuiltin(
2207:         *this, E, Intrinsic::ldexp, Intrinsic::experimental_constrained_ldexp);
2208:   case AMDGPU::BI__builtin_amdgcn_permlane_bcast:
2209:     return emitBuiltinWithOneOverloadedType<3>(
2210:         *this, E, Intrinsic::amdgcn_permlane_bcast);
2211:   case AMDGPU::BI__builtin_amdgcn_permlane_up:
2212:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
2213:                                                Intrinsic::amdgcn_permlane_up);
2214:   case AMDGPU::BI__builtin_amdgcn_permlane_down:
2215:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
2216:                                                Intrinsic::amdgcn_permlane_down);
2217:   case AMDGPU::BI__builtin_amdgcn_permlane_xor:
2218:     return emitBuiltinWithOneOverloadedType<3>(*this, E,
2219:                                                Intrinsic::amdgcn_permlane_xor);
2220:   default:
2221:     return nullptr;
2222:   }
2223: }
```
- **EN**: This block spells out callable entry points like `emitBinaryExpMaybeConstrainedFPBuiltin`; uses control flow (case) to specialize target builtin lowering.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitBinaryExpMaybeConstrainedFPBuiltin`；通过控制流（case）细化 目标内建函数降级 行为。

## Key Concepts / 关键概念

- **Intrinsic**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Builder**: Acts as a construction helper that incrementally assembles target builtin lowering state. / 充当构建辅助器，逐步组装 目标内建函数降级 状态。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGF**: Central symbol in this file's implementation of target builtin lowering. / 是该文件实现 目标内建函数降级 时的核心符号。
- **getArg**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **EmitScalarExpr**: Suggests an entry point that materializes IR or helper objects for target builtin lowering. / 暗示其是为 目标内建函数降级 生成 IR 或辅助对象的入口。
- **ArgsForMatchingMatrixTypes**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **BuiltinWMMAOp**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGBuiltin.h`, `CodeGenFunction.h`, `TargetInfo.h`
- **Clang libraries / Clang 库**: `clang/Basic/DiagnosticFrontend.h`, `clang/Basic/SyncScope.h`, `clang/Basic/TargetBuiltins.h`
- **LLVM libraries / LLVM 库**: `llvm/Analysis/ValueTracking.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/IntrinsicsAMDGPU.h`, `llvm/IR/IntrinsicsR600.h`, `llvm/IR/IntrinsicsSPIRV.h`, `llvm/IR/MemoryModelRelaxationAnnotations.h`, `llvm/Support/AMDGPUAddrSpace.h`, `llvm/Support/AtomicOrdering.h`
