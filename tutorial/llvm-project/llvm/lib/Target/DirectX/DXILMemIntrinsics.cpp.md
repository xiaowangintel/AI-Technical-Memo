# DXILMemIntrinsics.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILMemIntrinsics.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILMemIntrinsics support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILMemIntrinsics.cpp - Eliminate Memory Intrinsics ----------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILMemIntrinsics.h"
10: #include "DirectX.h"
11: #include "llvm/Analysis/DXILResource.h"
12: #include "llvm/IR/IRBuilder.h"
13: #include "llvm/IR/IntrinsicInst.h"
14: #include "llvm/IR/IntrinsicsDirectX.h"
15: #include "llvm/IR/Module.h"
16:
17: #define DEBUG_TYPE "dxil-mem-intrinsics"
18:
19: using namespace llvm;
20:
21: void expandMemSet(MemSetInst *MemSet) {
22:   IRBuilder<> Builder(MemSet);
23:   Value *Dst = MemSet->getDest();
24:   Value *Val = MemSet->getValue();
25:   ConstantInt *LengthCI = dyn_cast<ConstantInt>(MemSet->getLength());
26:   assert(LengthCI && "Expected length to be a ConstantInt");
27:
28:   [[maybe_unused]] const DataLayout &DL =
29:       Builder.GetInsertBlock()->getModule()->getDataLayout();
30:   [[maybe_unused]] uint64_t OrigLength = LengthCI->getZExtValue();
31:
32:   AllocaInst *Alloca = dyn_cast<AllocaInst>(Dst);
33:
34:   assert(Alloca && "Expected memset on an Alloca");
35:   assert(OrigLength == Alloca->getAllocationSize(DL)->getFixedValue() &&
36:          "Expected for memset size to match DataLayout size");
37:
38:   Type *AllocatedTy = Alloca->getAllocatedType();
39:   ArrayType *ArrTy = dyn_cast<ArrayType>(AllocatedTy);
40:   assert(ArrTy && "Expected Alloca for an Array Type");
```
- EN: This range implements operational logic in helpers such as expandMemSet, Builder, getDest, getValue, translating backend policy into executable code.
- CN: 这一段实现了 expandMemSet、Builder、getDest、getValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:
42:   Type *ElemTy = ArrTy->getElementType();
43:   uint64_t Size = ArrTy->getArrayNumElements();
44:
45:   [[maybe_unused]] uint64_t ElemSize = DL.getTypeStoreSize(ElemTy);
46:
47:   assert(ElemSize > 0 && "Size must be set");
48:   assert(OrigLength == ElemSize * Size && "Size in bytes must match");
49:
50:   Value *TypedVal = Val;
51:
52:   if (Val->getType() != ElemTy)
53:     TypedVal = Builder.CreateIntCast(Val, ElemTy, false);
54:
55:   for (uint64_t I = 0; I < Size; ++I) {
56:     Value *Zero = Builder.getInt32(0);
57:     Value *Offset = Builder.getInt32(I);
58:     Value *Ptr = Builder.CreateGEP(ArrTy, Dst, {Zero, Offset}, "gep");
59:     Builder.CreateStore(TypedVal, Ptr);
60:   }
61:
62:   MemSet->eraseFromParent();
63: }
64:
65: static Type *getPointeeType(Value *Ptr, const DataLayout &DL) {
66:   if (auto *GV = dyn_cast<GlobalVariable>(Ptr))
67:     return GV->getValueType();
68:   if (auto *AI = dyn_cast<AllocaInst>(Ptr))
69:     return AI->getAllocatedType();
70:
71:   if (auto *II = dyn_cast<IntrinsicInst>(Ptr)) {
72:     if (II->getIntrinsicID() == Intrinsic::dx_resource_getpointer) {
73:       Type *Ty = cast<dxil::AnyResourceExtType>(II->getArgOperand(0)->getType())
74:                      ->getResourceType();
75:       assert(Ty && "getpointer used on untyped resource");
76:       return Ty;
77:     }
78:   }
79:
80:   if (auto *GEP = dyn_cast<GEPOperator>(Ptr)) {
```
- EN: This range implements operational logic in helpers such as getElementType, getArrayNumElements, getTypeStoreSize, assert, translating backend policy into executable code.
- CN: 这一段实现了 getElementType、getArrayNumElements、getTypeStoreSize、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:     Type *Ty = GEP->getResultElementType();
 82:     if (!Ty->isIntegerTy(8))
 83:       return Ty;
 84:
 85:     // We have ptradd, so we have to hope there's enough information to work out
 86:     // what we're indexing.
 87:     Type *IndexedType = getPointeeType(GEP->getPointerOperand(), DL);
 88:     if (auto *AT = dyn_cast<ArrayType>(IndexedType))
 89:       return AT->getElementType();
 90:
 91:     if (auto *ST = dyn_cast<StructType>(IndexedType)) {
 92:       // Indexing a struct should always be constant
 93:       APInt ConstantOffset(DL.getIndexTypeSizeInBits(GEP->getType()), 0);
 94:       [[maybe_unused]] bool IsConst =
 95:           GEP->accumulateConstantOffset(DL, ConstantOffset);
 96:       assert(IsConst && "Non-constant GEP into struct?");
 97:
 98:       // Now, work out what we'll find at that offset.
 99:       const StructLayout *Layout = DL.getStructLayout(ST);
100:       unsigned Idx =
101:           Layout->getElementContainingOffset(ConstantOffset.getZExtValue());
102:
103:       return ST->getTypeAtIndex(Idx);
104:     }
105:
106:     llvm_unreachable("Could not infer type from GEP");
107:   }
108:
109:   llvm_unreachable("Could not calculate pointee type");
110: }
111:
112: static size_t flattenTypes(Type *ContainerTy, const DataLayout &DL,
113:                            SmallVectorImpl<std::pair<Type *, size_t>> &FlatTys,
114:                            size_t NextOffset = 0) {
115:   if (auto *AT = dyn_cast<ArrayType>(ContainerTy)) {
116:     for (uint64_t I = 0, E = AT->getNumElements(); I != E; ++I)
117:       NextOffset = flattenTypes(AT->getElementType(), DL, FlatTys, NextOffset);
118:     return NextOffset;
119:   }
120:   if (auto *ST = dyn_cast<StructType>(ContainerTy)) {
```
- EN: This range defines or declares important types such as getResultElementType, getPointeeType, getElementType, ConstantOffset, shaping the data model used by DXILMemIntrinsics.cpp.
- CN: 这一段定义或声明了 getResultElementType、getPointeeType、getElementType、ConstantOffset 等关键类型，构成 DXILMemIntrinsics.cpp 使用的数据模型。

### Lines 121-160
```cpp
121:     for (Type *Ty : ST->elements())
122:       NextOffset = flattenTypes(Ty, DL, FlatTys, NextOffset);
123:     return NextOffset;
124:   }
125:
126:   FlatTys.emplace_back(ContainerTy, NextOffset);
127:   return NextOffset + DL.getTypeStoreSize(ContainerTy);
128: }
129:
130: void expandMemCpy(MemCpyInst *MemCpy) {
131:   IRBuilder<> Builder(MemCpy);
132:   Value *Dst = MemCpy->getDest();
133:   Value *Src = MemCpy->getSource();
134:   ConstantInt *LengthCI = dyn_cast<ConstantInt>(MemCpy->getLength());
135:   assert(LengthCI && "Expected Length to be a ConstantInt");
136:   assert(!MemCpy->isVolatile() && "Handling for volatile not implemented");
137:
138:   uint64_t ByteLength = LengthCI->getZExtValue();
139:   // If length to copy is zero, no memcpy is needed.
140:   if (ByteLength == 0)
141:     return;
142:
143:   const DataLayout &DL = Builder.GetInsertBlock()->getModule()->getDataLayout();
144:
145:   SmallVector<std::pair<Type *, size_t>> FlattenedTypes;
146:   [[maybe_unused]] size_t MaxLength =
147:       flattenTypes(getPointeeType(Dst, DL), DL, FlattenedTypes);
148:   assert(MaxLength >= ByteLength && "Dst not large enough for memcpy");
149:
150:   LLVM_DEBUG({
151:     // Check if Src is layout compatible with Dst. This should always be true
152:     // unless the frontend did something wrong.
153:     SmallVector<std::pair<Type *, size_t>> SrcTypes;
154:     size_t SrcLength = flattenTypes(getPointeeType(Src, DL), DL, SrcTypes);
155:     assert(SrcLength >= ByteLength && "Src not large enough for memcpy");
156:     for (const auto &[LHS, RHS] : zip(FlattenedTypes, SrcTypes)) {
157:       auto &[DstTy, DstOffset] = LHS;
158:       auto &[SrcTy, SrcOffset] = RHS;
159:       assert(DstTy == SrcTy && "Mismatched types for memcpy");
160:       assert(DstOffset == SrcOffset && "Incompatible layouts for memcpy");
```
- EN: This range implements operational logic in helpers such as flattenTypes, emplace_back, getTypeStoreSize, expandMemCpy, translating backend policy into executable code.
- CN: 这一段实现了 flattenTypes、emplace_back、getTypeStoreSize、expandMemCpy 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:       if (DstOffset >= ByteLength)
162:         break;
163:     }
164:   });
165:
166:   for (const auto &[Ty, Offset] : FlattenedTypes) {
167:     if (Offset >= ByteLength)
168:       break;
169:     // TODO: Should we skip padding types here?
170:     Type *Int8Ty = Builder.getInt8Ty();
171:     Value *ByteOffset = Builder.getInt32(Offset);
172:     Value *SrcPtr = Builder.CreateInBoundsGEP(Int8Ty, Src, ByteOffset);
173:     Value *SrcVal = Builder.CreateLoad(Ty, SrcPtr);
174:     Value *DstPtr = Builder.CreateInBoundsGEP(Int8Ty, Dst, ByteOffset);
175:     Builder.CreateStore(SrcVal, DstPtr);
176:   }
177:
178:   MemCpy->eraseFromParent();
179: }
180:
181: void expandMemMove(MemMoveInst *MemMove) {
182:   report_fatal_error("memmove expansion is not implemented yet.");
183: }
184:
185: static bool eliminateMemIntrinsics(Module &M) {
186:   bool HadMemIntrinsicUses = false;
187:   for (auto &F : make_early_inc_range(M.functions())) {
188:     Intrinsic::ID IID = F.getIntrinsicID();
189:     switch (IID) {
190:     case Intrinsic::memcpy:
191:     case Intrinsic::memcpy_inline:
192:     case Intrinsic::memmove:
193:     case Intrinsic::memset:
194:     case Intrinsic::memset_inline:
195:       break;
196:     default:
197:       continue;
198:     }
199:     for (User *U : make_early_inc_range(F.users())) {
200:       HadMemIntrinsicUses = true;
```
- EN: This range implements operational logic in helpers such as getInt8Ty, getInt32, CreateInBoundsGEP, CreateLoad, translating backend policy into executable code.
- CN: 这一段实现了 getInt8Ty、getInt32、CreateInBoundsGEP、CreateLoad 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-238
```cpp
201:       if (auto *MemSet = dyn_cast<MemSetInst>(U))
202:         expandMemSet(MemSet);
203:       else if (auto *MemCpy = dyn_cast<MemCpyInst>(U))
204:         expandMemCpy(MemCpy);
205:       else if (auto *MemMove = dyn_cast<MemMoveInst>(U))
206:         expandMemMove(MemMove);
207:       else
208:         llvm_unreachable("Unhandled memory intrinsic");
209:     }
210:     assert(F.user_empty() && "Mem intrinsic not eliminated?");
211:     F.eraseFromParent();
212:   }
213:   return HadMemIntrinsicUses;
214: }
215:
216: PreservedAnalyses DXILMemIntrinsics::run(Module &M, ModuleAnalysisManager &) {
217:   if (eliminateMemIntrinsics(M))
218:     return PreservedAnalyses::none();
219:   return PreservedAnalyses::all();
220: }
221:
222: class DXILMemIntrinsicsLegacy : public ModulePass {
223: public:
224:   bool runOnModule(Module &M) override { return eliminateMemIntrinsics(M); }
225:   DXILMemIntrinsicsLegacy() : ModulePass(ID) {}
226:
227:   static char ID; // Pass identification.
228: };
229: char DXILMemIntrinsicsLegacy::ID = 0;
230:
231: INITIALIZE_PASS_BEGIN(DXILMemIntrinsicsLegacy, DEBUG_TYPE,
232:                       "DXIL Memory Intrinsic Elimination", false, false)
233: INITIALIZE_PASS_END(DXILMemIntrinsicsLegacy, DEBUG_TYPE,
234:                     "DXIL Memory Intrinsic Elimination", false, false)
235:
236: ModulePass *llvm::createDXILMemIntrinsicsLegacyPass() {
237:   return new DXILMemIntrinsicsLegacy();
238: }
```
- EN: This range defines or declares important types such as expandMemSet, expandMemCpy, expandMemMove, llvm_unreachable, shaping the data model used by DXILMemIntrinsics.cpp.
- CN: 这一段定义或声明了 expandMemSet、expandMemCpy、expandMemMove、llvm_unreachable 等关键类型，构成 DXILMemIntrinsics.cpp 使用的数据模型。

## Key Concepts / 关键概念

- EN: Intrinsic handling connects target-specific builtins with LLVM IR or machine-level lowering paths.
  - CN: 内建函数处理负责把目标特定 builtin 与 LLVM IR 或机器级降级路径连接起来。
- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include expandMemSet, Builder, getDest, getValue, getLength, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 expandMemSet, Builder, getDest, getValue, getLength，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILMemIntrinsics.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/DXILResource.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/Module.h`
