# NVPTXISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXISelLowering.h`
- Repository: `llvm-project`
- Purpose (EN): This file defines the interfaces that NVPTX uses to lower LLVM code into a selection DAG.
- 目的（中文）: 该文件实现目标相关的指令选择逻辑，把 LLVM 或机器 IR 映射到后端可接受的表示。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXISelLowering.h - NVPTX DAG Lowering Interface ------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file defines the interfaces that NVPTX uses to lower LLVM code into a
10: // selection DAG.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_LIB_TARGET_NVPTX_NVPTXISELLOWERING_H
15: #define LLVM_LIB_TARGET_NVPTX_NVPTXISELLOWERING_H
16:
17: #include "NVPTX.h"
18: #include "llvm/CodeGen/SelectionDAG.h"
19: #include "llvm/CodeGen/TargetLowering.h"
20: #include "llvm/Support/AtomicOrdering.h"
21:
22: namespace llvm {
23:
24: class NVPTXSubtarget;
25:
26: //===--------------------------------------------------------------------===//
27: // TargetLowering Implementation
28: //===--------------------------------------------------------------------===//
29: class NVPTXTargetLowering : public TargetLowering {
30: public:
31:   explicit NVPTXTargetLowering(const NVPTXTargetMachine &TM,
32:                                const NVPTXSubtarget &STI);
33:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
34:
35:   void getTgtMemIntrinsic(SmallVectorImpl<IntrinsicInfo> &Infos,
36:                           const CallBase &I, MachineFunction &MF,
37:                           unsigned Intrinsic) const override;
38:
39:   // Helper for getting a function parameter name. Name is composed from
40:   // its index and the function name. Negative index corresponds to special
```
- EN: This range defines or declares important types such as NVPTXSubtarget, NVPTXTargetLowering, LowerOperation, shaping the data model used by NVPTXISelLowering.h.
- CN: 这一段定义或声明了 NVPTXSubtarget、NVPTXTargetLowering、LowerOperation 等关键类型，构成 NVPTXISelLowering.h 使用的数据模型。

### Lines 41-80
```cpp
41:   // parameter (unsized array) used for passing variable arguments.
42:   std::string getParamName(const Function *F, int Idx) const;
43:
44:   /// isLegalAddressingMode - Return true if the addressing mode represented
45:   /// by AM is legal for this target, for a load/store of the specified type
46:   /// Used to guide target specific optimizations, like loop strength
47:   /// reduction (LoopStrengthReduce.cpp) and memory optimization for
48:   /// address mode (CodeGenPrepare.cpp)
49:   bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM, Type *Ty,
50:                              unsigned AS,
51:                              Instruction *I = nullptr) const override;
52:
53:   bool isTruncateFree(Type *SrcTy, Type *DstTy) const override {
54:     // Truncating 64-bit to 32-bit is free in SASS.
55:     if (!SrcTy->isIntegerTy() || !DstTy->isIntegerTy())
56:       return false;
57:     return SrcTy->getPrimitiveSizeInBits() == 64 &&
58:            DstTy->getPrimitiveSizeInBits() == 32;
59:   }
60:
61:   EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Ctx,
62:                          EVT VT) const override {
63:     if (VT.isVector())
64:       return EVT::getVectorVT(Ctx, MVT::i1, VT.getVectorNumElements());
65:     return MVT::i1;
66:   }
67:
68:   ConstraintType getConstraintType(StringRef Constraint) const override;
69:   std::pair<unsigned, const TargetRegisterClass *>
70:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
71:                                StringRef Constraint, MVT VT) const override;
72:
73:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
74:                                bool isVarArg,
75:                                const SmallVectorImpl<ISD::InputArg> &Ins,
76:                                const SDLoc &dl, SelectionDAG &DAG,
77:                                SmallVectorImpl<SDValue> &InVals) const override;
78:
79:   SDValue LowerCall(CallLoweringInfo &CLI,
80:                     SmallVectorImpl<SDValue> &InVals) const override;
```
- EN: This range declares interfaces or inline helpers such as getParamName, isTruncateFree, EVT::getVectorVT, getConstraintType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getParamName、isTruncateFree、EVT::getVectorVT、getConstraintType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 81-120
```cpp
 81:
 82:   SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
 83:   SDValue LowerSTACKSAVE(SDValue Op, SelectionDAG &DAG) const;
 84:   SDValue LowerSTACKRESTORE(SDValue Op, SelectionDAG &DAG) const;
 85:
 86:   std::string getPrototype(const DataLayout &DL, Type *, const ArgListTy &,
 87:                            const SmallVectorImpl<ISD::OutputArg> &,
 88:                            std::optional<unsigned> FirstVAArg,
 89:                            const CallBase &CB, unsigned UniqueCallSite) const;
 90:
 91:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
 92:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 93:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &dl,
 94:                       SelectionDAG &DAG) const override;
 95:
 96:   void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
 97:                                     std::vector<SDValue> &Ops,
 98:                                     SelectionDAG &DAG) const override;
 99:
100:   const NVPTXTargetMachine *nvTM;
101:
102:   // PTX always uses 32-bit shift amounts
103:   MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override {
104:     return MVT::i32;
105:   }
106:
107:   TargetLoweringBase::LegalizeTypeAction
108:   getPreferredVectorAction(MVT VT) const override;
109:
110:   // Get the degree of precision we want from 32-bit floating point division
111:   // operations.
112:   NVPTX::DivPrecisionLevel getDivF32Level(const MachineFunction &MF,
113:                                           const SDNode &N) const;
114:
115:   // Get whether we should use a precise or approximate 32-bit floating point
116:   // sqrt instruction.
117:   bool usePrecSqrtF32(const SDNode *N = nullptr) const;
118:
119:   // Get whether we should use instructions that flush floating-point denormals
120:   // to sign-preserving zero.
```
- EN: This range declares interfaces or inline helpers such as LowerDYNAMIC_STACKALLOC, LowerSTACKSAVE, LowerSTACKRESTORE, getScalarShiftAmountTy, defining how other backend pieces interact with this header.
- CN: 这一段声明了 LowerDYNAMIC_STACKALLOC、LowerSTACKSAVE、LowerSTACKRESTORE、getScalarShiftAmountTy 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-160
```cpp
121:   bool useF32FTZ(const MachineFunction &MF) const;
122:
123:   SDValue getSqrtEstimate(SDValue Operand, SelectionDAG &DAG, int Enabled,
124:                           int &ExtraSteps, bool &UseOneConst,
125:                           bool Reciprocal) const override;
126:
127:   unsigned combineRepeatedFPDivisors() const override { return 2; }
128:
129:   bool allowFMA(MachineFunction &MF, CodeGenOptLevel OptLevel) const;
130:
131:   bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
132:                                   EVT) const override {
133:     return true;
134:   }
135:
136:   // The default is the same as pointer type, but brx.idx only accepts i32
137:   MVT getJumpTableRegTy(const DataLayout &) const override { return MVT::i32; }
138:
139:   unsigned getJumpTableEncoding() const override;
140:
141:   bool enableAggressiveFMAFusion(EVT VT) const override { return true; }
142:
143:   // The default is to transform llvm.ctlz(x, false) (where false indicates that
144:   // x == 0 is not undefined behavior) into a branch that checks whether x is 0
145:   // and avoids calling ctlz in that case.  We have a dedicated ctlz
146:   // instruction, so we say that ctlz is cheap to speculate.
147:   bool isCheapToSpeculateCtlz(Type *Ty) const override { return true; }
148:
149:   AtomicExpansionKind shouldCastAtomicLoadInIR(LoadInst *LI) const override {
150:     return AtomicExpansionKind::None;
151:   }
152:
153:   AtomicExpansionKind shouldCastAtomicStoreInIR(StoreInst *SI) const override {
154:     return AtomicExpansionKind::None;
155:   }
156:
157:   AtomicExpansionKind
158:   shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const override;
159:
160:   bool aggressivelyPreferBuildVectorSources(EVT VecVT) const override {
```
- EN: This range declares interfaces or inline helpers such as useF32FTZ, combineRepeatedFPDivisors, allowFMA, getJumpTableRegTy, defining how other backend pieces interact with this header.
- CN: 这一段声明了 useF32FTZ、combineRepeatedFPDivisors、allowFMA、getJumpTableRegTy 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 161-200
```cpp
161:     // There's rarely any point of packing something into a vector type if we
162:     // already have the source data.
163:     return true;
164:   }
165:
166:   bool shouldInsertFencesForAtomic(const Instruction *) const override;
167:
168:   AtomicOrdering
169:   atomicOperationOrderAfterFenceSplit(const Instruction *I) const override;
170:
171:   Instruction *emitLeadingFence(IRBuilderBase &Builder, Instruction *Inst,
172:                                 AtomicOrdering Ord) const override;
173:   Instruction *emitTrailingFence(IRBuilderBase &Builder, Instruction *Inst,
174:                                  AtomicOrdering Ord) const override;
175:
176:   unsigned getPreferredFPToIntOpcode(unsigned Op, EVT FromVT,
177:                                      EVT ToVT) const override;
178:
179:   void computeKnownBitsForTargetNode(const SDValue Op, KnownBits &Known,
180:                                      const APInt &DemandedElts,
181:                                      const SelectionDAG &DAG,
182:                                      unsigned Depth = 0) const override;
183:   bool SimplifyDemandedBitsForTargetNode(SDValue Op, const APInt &DemandedBits,
184:                                          const APInt &DemandedElts,
185:                                          KnownBits &Known,
186:                                          TargetLoweringOpt &TLO,
187:                                          unsigned Depth = 0) const override;
188:
189: private:
190:   const NVPTXSubtarget &STI; // cache the subtarget here
191:   mutable unsigned GlobalUniqueCallSite;
192:
193:   SDValue getParamSymbol(SelectionDAG &DAG, int I, EVT T) const;
194:   SDValue getCallParamSymbol(SelectionDAG &DAG, int I, EVT T) const;
195:   SDValue LowerADDRSPACECAST(SDValue Op, SelectionDAG &DAG) const;
196:   SDValue LowerBITCAST(SDValue Op, SelectionDAG &DAG) const;
197:
198:   SDValue LowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;
199:   SDValue LowerCONCAT_VECTORS(SDValue Op, SelectionDAG &DAG) const;
200:   SDValue LowerVECREDUCE(SDValue Op, SelectionDAG &DAG) const;
```
- EN: This range declares interfaces or inline helpers such as shouldInsertFencesForAtomic, atomicOperationOrderAfterFenceSplit, getParamSymbol, getCallParamSymbol, defining how other backend pieces interact with this header.
- CN: 这一段声明了 shouldInsertFencesForAtomic、atomicOperationOrderAfterFenceSplit、getParamSymbol、getCallParamSymbol 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 201-240
```cpp
201:   SDValue LowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
202:   SDValue LowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
203:   SDValue LowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG) const;
204:
205:   SDValue LowerFCOPYSIGN(SDValue Op, SelectionDAG &DAG) const;
206:
207:   SDValue LowerFROUND(SDValue Op, SelectionDAG &DAG) const;
208:   SDValue LowerFROUND32(SDValue Op, SelectionDAG &DAG) const;
209:   SDValue LowerFROUND64(SDValue Op, SelectionDAG &DAG) const;
210:
211:   SDValue PromoteBinOpIfF32FTZ(SDValue Op, SelectionDAG &DAG) const;
212:
213:   SDValue LowerINT_TO_FP(SDValue Op, SelectionDAG &DAG) const;
214:   SDValue LowerFP_TO_INT(SDValue Op, SelectionDAG &DAG) const;
215:
216:   SDValue LowerFP_ROUND(SDValue Op, SelectionDAG &DAG) const;
217:   SDValue LowerFP_EXTEND(SDValue Op, SelectionDAG &DAG) const;
218:
219:   SDValue LowerLOAD(SDValue Op, SelectionDAG &DAG) const;
220:   SDValue LowerMLOAD(SDValue Op, SelectionDAG &DAG) const;
221:   SDValue LowerSTORE(SDValue Op, SelectionDAG &DAG) const;
222:   SDValue LowerSTOREi1(SDValue Op, SelectionDAG &DAG) const;
223:
224:   SDValue LowerShiftRightParts(SDValue Op, SelectionDAG &DAG) const;
225:   SDValue LowerShiftLeftParts(SDValue Op, SelectionDAG &DAG) const;
226:
227:   SDValue LowerVAARG(SDValue Op, SelectionDAG &DAG) const;
228:   SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
229:
230:   SDValue LowerCopyToReg_128(SDValue Op, SelectionDAG &DAG) const;
231:   unsigned getNumRegisters(LLVMContext &Context, EVT VT,
232:                            std::optional<MVT> RegisterVT) const override;
233:   bool
234:   splitValueIntoRegisterParts(SelectionDAG &DAG, const SDLoc &DL, SDValue Val,
235:                               SDValue *Parts, unsigned NumParts, MVT PartVT,
236:                               std::optional<CallingConv::ID> CC) const override;
237:
238:   void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
239:                           SelectionDAG &DAG) const override;
240:   SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
```
- EN: This range declares interfaces or inline helpers such as LowerEXTRACT_VECTOR_ELT, LowerINSERT_VECTOR_ELT, LowerVECTOR_SHUFFLE, LowerFCOPYSIGN, defining how other backend pieces interact with this header.
- CN: 这一段声明了 LowerEXTRACT_VECTOR_ELT、LowerINSERT_VECTOR_ELT、LowerVECTOR_SHUFFLE、LowerFCOPYSIGN 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 241-255
```cpp
241:
242:   bool mayFoldFMULIntoFMA(SDNode *N, MachineFunction &MF,
243:                           CodeGenOptLevel OptLevel) const;
244:   SDValue performScalarizeV2F32Op(SDNode *N, DAGCombinerInfo &DCI,
245:                                   CodeGenOptLevel OptLevel) const;
246:   SDValue performFADDCombineWithOperands(SDNode *N, SDValue N0, SDValue N1,
247:                                          DAGCombinerInfo &DCI,
248:                                          CodeGenOptLevel OptLevel) const;
249:   SDValue performFADDCombine(SDNode *N, DAGCombinerInfo &DCI,
250:                              CodeGenOptLevel OptLevel) const;
251: };
252:
253: } // namespace llvm
254:
255: #endif
```
- EN: This range declares interfaces or inline helpers such as backend logic, defining how other backend pieces interact with this header.
- CN: 这一段声明了 后端逻辑 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: Instruction selection maps generic LLVM operations onto target-specific machine instructions.
  - CN: 指令选择负责把通用 LLVM 操作映射为目标相关的机器指令。
- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXSubtarget, NVPTXTargetLowering, LowerOperation, getParamName, isTruncateFree, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXSubtarget, NVPTXTargetLowering, LowerOperation, getParamName, isTruncateFree，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/SelectionDAG.h`
  - `llvm/CodeGen/TargetLowering.h`
  - `llvm/Support/AtomicOrdering.h`
