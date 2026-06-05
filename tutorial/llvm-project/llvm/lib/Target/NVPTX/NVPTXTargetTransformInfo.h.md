# NVPTXTargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXTargetTransformInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file a TargetTransformInfoImplBase conforming object specific to the NVPTX target machine.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXTargetTransformInfo.h - NVPTX specific TTI ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: /// \file
 9: /// This file a TargetTransformInfoImplBase conforming object specific to the
10: /// NVPTX target machine. It uses the target's detailed information to
11: /// provide more precise answers to certain TTI queries, while letting the
12: /// target independent and default TTI implementations handle the rest.
13: ///
14: //===----------------------------------------------------------------------===//
15:
16: #ifndef LLVM_LIB_TARGET_NVPTX_NVPTXTARGETTRANSFORMINFO_H
17: #define LLVM_LIB_TARGET_NVPTX_NVPTXTARGETTRANSFORMINFO_H
18:
19: #include "MCTargetDesc/NVPTXBaseInfo.h"
20: #include "NVPTXTargetMachine.h"
21: #include "NVPTXUtilities.h"
22: #include "llvm/Analysis/TargetTransformInfo.h"
23: #include "llvm/CodeGen/BasicTTIImpl.h"
24: #include "llvm/CodeGen/TargetLowering.h"
25: #include <optional>
26:
27: namespace llvm {
28:
29: class NVPTXTTIImpl final : public BasicTTIImplBase<NVPTXTTIImpl> {
30:   typedef BasicTTIImplBase<NVPTXTTIImpl> BaseT;
31:   typedef TargetTransformInfo TTI;
32:   friend BaseT;
33:
34:   const NVPTXSubtarget *ST;
35:   const NVPTXTargetLowering *TLI;
36:
37:   const NVPTXSubtarget *getST() const { return ST; };
38:   const NVPTXTargetLowering *getTLI() const { return TLI; };
39:
40:   /// \returns true if the result of the value could potentially be
```
- EN: This range defines or declares important types such as NVPTXTTIImpl, getST, getTLI, shaping the data model used by NVPTXTargetTransformInfo.h.
- CN: 这一段定义或声明了 NVPTXTTIImpl、getST、getTLI 等关键类型，构成 NVPTXTargetTransformInfo.h 使用的数据模型。

### Lines 41-80
```cpp
41:   /// different across threads in a warp.
42:   bool isSourceOfDivergence(const Value *V) const;
43:
44: public:
45:   explicit NVPTXTTIImpl(const NVPTXTargetMachine *TM, const Function &F)
46:       : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl()),
47:         TLI(ST->getTargetLowering()) {}
48:
49:   bool hasBranchDivergence(const Function *F = nullptr) const override {
50:     return true;
51:   }
52:
53:   unsigned getFlatAddressSpace() const override {
54:     return AddressSpace::ADDRESS_SPACE_GENERIC;
55:   }
56:
57:   bool
58:   canHaveNonUndefGlobalInitializerInAddressSpace(unsigned AS) const override {
59:     return AS != AddressSpace::ADDRESS_SPACE_SHARED &&
60:            AS != AddressSpace::ADDRESS_SPACE_LOCAL &&
61:            AS != AddressSpace::ADDRESS_SPACE_ENTRY_PARAM;
62:   }
63:
64:   std::optional<Instruction *>
65:   instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const override;
66:
67:   // Loads and stores can be vectorized if the alignment is at least as big as
68:   // the load/store we want to vectorize.
69:   bool isLegalToVectorizeLoadChain(unsigned ChainSizeInBytes, Align Alignment,
70:                                    unsigned AddrSpace) const override {
71:     return Alignment >= ChainSizeInBytes;
72:   }
73:   bool isLegalToVectorizeStoreChain(unsigned ChainSizeInBytes, Align Alignment,
74:                                     unsigned AddrSpace) const override {
75:     return isLegalToVectorizeLoadChain(ChainSizeInBytes, Alignment, AddrSpace);
76:   }
77:
78:   // NVPTX has infinite registers of all kinds, but the actual machine doesn't.
79:   // We conservatively return 1 here which is just enough to enable the
80:   // vectorizers but disables heuristics based on the number of registers.
```
- EN: This range declares interfaces or inline helpers such as isSourceOfDivergence, NVPTXTTIImpl, TLI, hasBranchDivergence, defining how other backend pieces interact with this header.
- CN: 这一段声明了 isSourceOfDivergence、NVPTXTTIImpl、TLI、hasBranchDivergence 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-120
```cpp
 81:   // FIXME: Return a more reasonable number, while keeping an eye on
 82:   // LoopVectorizer's unrolling heuristics.
 83:   unsigned getNumberOfRegisters(unsigned ClassID) const override { return 1; }
 84:
 85:   // Only <2 x half> should be vectorized, so always return 32 for the vector
 86:   // register size.
 87:   TypeSize
 88:   getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override {
 89:     return TypeSize::getFixed(32);
 90:   }
 91:   unsigned getMinVectorRegisterBitWidth() const override { return 32; }
 92:
 93:   bool shouldExpandReduction(const IntrinsicInst *II) const override {
 94:     // Turn off ExpandReductions pass for NVPTX, which doesn't have advanced
 95:     // swizzling operations. Our backend/Selection DAG can expand these
 96:     // reductions with less movs.
 97:     return false;
 98:   }
 99:
100:   // We don't want to prevent inlining because of target-cpu and -features
101:   // attributes that were added to newer versions of LLVM/Clang: There are
102:   // no incompatible functions in PTX, ptxas will throw errors in such cases.
103:   bool areInlineCompatible(const Function *Caller,
104:                            const Function *Callee) const override {
105:     return true;
106:   }
107:
108:   // Increase the inlining cost threshold by a factor of 11, reflecting that
109:   // calls are particularly expensive in NVPTX.
110:   unsigned getInliningThresholdMultiplier() const override { return 11; }
111:
112:   InstructionCost
113:   getInstructionCost(const User *U, ArrayRef<const Value *> Operands,
114:                      TTI::TargetCostKind CostKind) const override;
115:
116:   InstructionCost getArithmeticInstrCost(
117:       unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
118:       TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
119:       TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
120:       ArrayRef<const Value *> Args = {},
```
- EN: This range declares interfaces or inline helpers such as getNumberOfRegisters, getRegisterBitWidth, TypeSize::getFixed, getMinVectorRegisterBitWidth, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getNumberOfRegisters、getRegisterBitWidth、TypeSize::getFixed、getMinVectorRegisterBitWidth 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-160
```cpp
121:       const Instruction *CxtI = nullptr) const override;
122:
123:   InstructionCost
124:   getScalarizationOverhead(VectorType *InTy, const APInt &DemandedElts,
125:                            bool Insert, bool Extract,
126:                            TTI::TargetCostKind CostKind,
127:                            bool ForPoisonSrc = true, ArrayRef<Value *> VL = {},
128:                            TTI::VectorInstrContext VIC =
129:                                TTI::VectorInstrContext::None) const override {
130:     if (!InTy->getElementCount().isFixed())
131:       return InstructionCost::getInvalid();
132:
133:     auto VT = getTLI()->getValueType(DL, InTy);
134:     auto NumElements = InTy->getElementCount().getFixedValue();
135:     InstructionCost Cost = 0;
136:     if (Insert && !VL.empty()) {
137:       bool AllConstant = all_of(seq(NumElements), [&](int Idx) {
138:         return !DemandedElts[Idx] || isa<Constant>(VL[Idx]);
139:       });
140:       if (AllConstant) {
141:         Cost += TTI::TCC_Free;
142:         Insert = false;
143:       }
144:     }
145:     if (Insert && NVPTX::isPackedVectorTy(VT) && VT.is32BitVector()) {
146:       // Can be built in a single 32-bit mov (64-bit regs are emulated in SASS
147:       // with 2x 32-bit regs)
148:       Cost += 1;
149:       Insert = false;
150:     }
151:     if (Insert && VT == MVT::v4i8) {
152:       InstructionCost Cost = 3; // 3 x PRMT
153:       for (auto Idx : seq(NumElements))
154:         if (DemandedElts[Idx])
155:           Cost += 1; // zext operand to i32
156:       Insert = false;
157:     }
158:     return Cost + BaseT::getScalarizationOverhead(InTy, DemandedElts, Insert,
159:                                                   Extract, CostKind,
160:                                                   ForPoisonSrc, VL);
```
- EN: This range declares interfaces or inline helpers such as InstructionCost::getInvalid, getTLI, getElementCount, all_of, defining how other backend pieces interact with this header.
- CN: 这一段声明了 InstructionCost::getInvalid、getTLI、getElementCount、all_of 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 161-200
```cpp
161:   }
162:
163:   void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
164:                                TTI::UnrollingPreferences &UP,
165:                                OptimizationRemarkEmitter *ORE) const override;
166:
167:   void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
168:                              TTI::PeelingPreferences &PP) const override;
169:
170:   bool hasVolatileVariant(Instruction *I, unsigned AddrSpace) const override {
171:     // Volatile loads/stores are only supported for shared and global address
172:     // spaces, or for generic AS that maps to them.
173:     if (!(AddrSpace == llvm::ADDRESS_SPACE_GENERIC ||
174:           AddrSpace == llvm::ADDRESS_SPACE_GLOBAL ||
175:           AddrSpace == llvm::ADDRESS_SPACE_SHARED))
176:       return false;
177:
178:     switch(I->getOpcode()){
179:     default:
180:       return false;
181:     case Instruction::Load:
182:     case Instruction::Store:
183:       return true;
184:     }
185:   }
186:
187:   APInt getAddrSpaceCastPreservedPtrMask(unsigned SrcAS,
188:                                          unsigned DstAS) const override {
189:     if (SrcAS != llvm::ADDRESS_SPACE_GENERIC)
190:       return BaseT::getAddrSpaceCastPreservedPtrMask(SrcAS, DstAS);
191:     if (DstAS != llvm::ADDRESS_SPACE_GLOBAL &&
192:         DstAS != llvm::ADDRESS_SPACE_SHARED)
193:       return BaseT::getAddrSpaceCastPreservedPtrMask(SrcAS, DstAS);
194:
195:     // Address change within 4K size does not change the original address space
196:     // and is safe to perform address cast form SrcAS to DstAS.
197:     APInt PtrMask(DL.getPointerSizeInBits(llvm::ADDRESS_SPACE_GENERIC), 0xfff);
198:     return PtrMask;
199:   }
200:
```
- EN: This range declares interfaces or inline helpers such as hasVolatileVariant, BaseT::getAddrSpaceCastPreservedPtrMask, PtrMask, defining how other backend pieces interact with this header.
- CN: 这一段声明了 hasVolatileVariant、BaseT::getAddrSpaceCastPreservedPtrMask、PtrMask 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 201-239
```cpp
201:   bool collectFlatAddressOperands(SmallVectorImpl<int> &OpIndexes,
202:                                   Intrinsic::ID IID) const override;
203:
204:   bool isLegalMaskedStore(Type *DataType, Align Alignment, unsigned AddrSpace,
205:                           TTI::MaskKind MaskKind) const override;
206:
207:   bool isLegalMaskedLoad(Type *DataType, Align Alignment, unsigned AddrSpace,
208:                          TTI::MaskKind MaskKind) const override;
209:
210:   unsigned getLoadStoreVecRegBitWidth(unsigned AddrSpace) const override;
211:
212:   Value *rewriteIntrinsicWithAddressSpace(IntrinsicInst *II, Value *OldV,
213:                                           Value *NewV) const override;
214:   unsigned getAssumedAddrSpace(const Value *V) const override;
215:
216:   void collectKernelLaunchBounds(
217:       const Function &F,
218:       SmallVectorImpl<std::pair<StringRef, int64_t>> &LB) const override;
219:
220:   bool shouldBuildRelLookupTables() const override {
221:     // Self-referential globals are not supported.
222:     return false;
223:   }
224:
225:   InstructionCost getPartialReductionCost(
226:       unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
227:       ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
228:       TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
229:       TTI::TargetCostKind CostKind,
230:       std::optional<FastMathFlags> FMF) const override {
231:     return InstructionCost::getInvalid();
232:   }
233:
234:   ValueUniformity getValueUniformity(const Value *V) const override;
235: };
236:
237: } // end namespace llvm
238:
239: #endif
```
- EN: This range declares interfaces or inline helpers such as getLoadStoreVecRegBitWidth, getAssumedAddrSpace, shouldBuildRelLookupTables, InstructionCost::getInvalid, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getLoadStoreVecRegBitWidth、getAssumedAddrSpace、shouldBuildRelLookupTables、InstructionCost::getInvalid 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXTTIImpl, getST, getTLI, isSourceOfDivergence, TLI, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXTTIImpl, getST, getTLI, isSourceOfDivergence, TLI，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/NVPTXBaseInfo.h`
  - `NVPTXTargetMachine.h`
  - `NVPTXUtilities.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/TargetTransformInfo.h`
  - `llvm/CodeGen/BasicTTIImpl.h`
  - `llvm/CodeGen/TargetLowering.h`
- System/standard headers / 系统或标准头文件:
  - `optional`
