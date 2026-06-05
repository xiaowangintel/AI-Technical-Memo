# LoongArchISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchISelLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责实现目标后端的 SelectionDAG 降低。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=- LoongArchISelLowering.h - LoongArch DAG Lowering Interface -*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that LoongArch uses to lower LLVM code into
  10: // a selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_LOONGARCH_LOONGARCHISELLOWERING_H
  15: #define LLVM_LIB_TARGET_LOONGARCH_LOONGARCHISELLOWERING_H
  16: 
  17: #include "LoongArch.h"
  18: #include "llvm/CodeGen/CallingConvLower.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArch.h`, `CallingConvLower.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArch.h`, `CallingConvLower.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/CodeGen/SelectionDAG.h"
  20: #include "llvm/CodeGen/SelectionDAGNodes.h"
  21: #include "llvm/CodeGen/TargetLowering.h"
  22: 
  23: namespace llvm {
  24: class LoongArchSubtarget;
  25: class LoongArchTargetLowering : public TargetLowering {
  26:   const LoongArchSubtarget &Subtarget;
  27: 
  28: public:
  29:   explicit LoongArchTargetLowering(const TargetMachine &TM,
  30:                                    const LoongArchSubtarget &STI);
  31: 
  32:   const LoongArchSubtarget &getSubtarget() const { return Subtarget; }
  33: 
  34:   bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
  35: 
  36:   // Provide custom lowering hooks for some operations.
```
- **EN**: It imports dependencies such as `SelectionDAG.h`, `SelectionDAGNodes.h`, `TargetLowering.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `LoongArchSubtarget`, `LoongArchTargetLowering`.
- **CN**: 它引入了 `SelectionDAG.h`, `SelectionDAGNodes.h`, `TargetLowering.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `LoongArchSubtarget`, `LoongArchTargetLowering` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  38:   void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
  39:                           SelectionDAG &DAG) const override;
  40: 
  41:   SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
  42: 
  43:   // Lower incoming arguments, copy physregs into vregs.
  44:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
  45:                                bool IsVarArg,
  46:                                const SmallVectorImpl<ISD::InputArg> &Ins,
  47:                                const SDLoc &DL, SelectionDAG &DAG,
  48:                                SmallVectorImpl<SDValue> &InVals) const override;
  49:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
  50:                       bool IsVarArg,
  51:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
  52:                       LLVMContext &Context, const Type *RetTy) const override;
  53:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
  54:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
  56:                       SelectionDAG &DAG) const override;
  57:   SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
  58:                     SmallVectorImpl<SDValue> &InVals) const override;
  59:   bool canMergeStoresTo(unsigned AddressSpace, EVT MemVT,
  60:                         const MachineFunction &MF) const override;
  61:   bool isCheapToSpeculateCttz(Type *Ty) const override;
  62:   bool isCheapToSpeculateCtlz(Type *Ty) const override;
  63:   bool hasAndNot(SDValue Y) const override;
  64:   TargetLowering::AtomicExpansionKind
  65:   shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const override;
  66:   void emitExpandAtomicRMW(AtomicRMWInst *AI) const override;
  67: 
  68:   Value *emitMaskedAtomicRMWIntrinsic(IRBuilderBase &Builder, AtomicRMWInst *AI,
  69:                                       Value *AlignedAddr, Value *Incr,
  70:                                       Value *Mask, Value *ShiftAmt,
  71:                                       AtomicOrdering Ord) const override;
  72: 
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
  74:                          EVT VT) const override;
  75:   TargetLowering::AtomicExpansionKind
  76:   shouldExpandAtomicCmpXchgInIR(const AtomicCmpXchgInst *CI) const override;
  77:   Value *emitMaskedAtomicCmpXchgIntrinsic(IRBuilderBase &Builder,
  78:                                           AtomicCmpXchgInst *CI,
  79:                                           Value *AlignedAddr, Value *CmpVal,
  80:                                           Value *NewVal, Value *Mask,
  81:                                           AtomicOrdering Ord) const override;
  82: 
  83:   void getTgtMemIntrinsic(SmallVectorImpl<IntrinsicInfo> &Infos,
  84:                           const CallBase &I, MachineFunction &MF,
  85:                           unsigned Intrinsic) const override;
  86: 
  87:   bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
  88:                                   EVT VT) const override;
  89: 
  90:   Register
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   getExceptionPointerRegister(const Constant *PersonalityFn) const override;
  92: 
  93:   Register
  94:   getExceptionSelectorRegister(const Constant *PersonalityFn) const override;
  95: 
  96:   bool isFsqrtCheap(SDValue Operand, SelectionDAG &DAG) const override {
  97:     return true;
  98:   }
  99: 
 100:   SDValue getSqrtEstimate(SDValue Operand, SelectionDAG &DAG, int Enabled,
 101:                           int &RefinementSteps, bool &UseOneConstNR,
 102:                           bool Reciprocal) const override;
 103: 
 104:   SDValue getRecipEstimate(SDValue Operand, SelectionDAG &DAG, int Enabled,
 105:                            int &RefinementSteps) const override;
 106: 
 107:   ISD::NodeType getExtendForAtomicOps() const override {
 108:     return ISD::SIGN_EXTEND;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   }
 110: 
 111:   ISD::NodeType getExtendForAtomicCmpSwapArg() const override;
 112: 
 113:   Register getRegisterByName(const char *RegName, LLT VT,
 114:                              const MachineFunction &MF) const override;
 115:   bool mayBeEmittedAsTailCall(const CallInst *CI) const override;
 116: 
 117:   bool decomposeMulByConstant(LLVMContext &Context, EVT VT,
 118:                               SDValue C) const override;
 119: 
 120:   bool isUsedByReturnOnly(SDNode *N, SDValue &Chain) const override;
 121: 
 122:   bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM, Type *Ty,
 123:                              unsigned AS,
 124:                              Instruction *I = nullptr) const override;
 125: 
 126:   bool isLegalICmpImmediate(int64_t Imm) const override;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   bool isLegalAddImmediate(int64_t Imm) const override;
 128:   bool isZExtFree(SDValue Val, EVT VT2) const override;
 129:   bool isSExtCheaperThanZExt(EVT SrcVT, EVT DstVT) const override;
 130:   bool signExtendConstant(const ConstantInt *CI) const override;
 131: 
 132:   bool hasAndNotCompare(SDValue Y) const override;
 133: 
 134:   bool convertSelectOfConstantsToMath(EVT VT) const override { return true; }
 135: 
 136:   bool allowsMisalignedMemoryAccesses(
 137:       EVT VT, unsigned AddrSpace = 0, Align Alignment = Align(1),
 138:       MachineMemOperand::Flags Flags = MachineMemOperand::MONone,
 139:       unsigned *Fast = nullptr) const override;
 140: 
 141:   bool isShuffleMaskLegal(ArrayRef<int> Mask, EVT VT) const override {
 142:     if (!VT.isSimple())
 143:       return false;
 144: 
```
- **EN**: The range implements or declares functions including `isShuffleMaskLegal`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isShuffleMaskLegal` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     // Not for i1 vectors
 146:     if (VT.getSimpleVT().getScalarType() == MVT::i1)
 147:       return false;
 148: 
 149:     return isTypeLegal(VT.getSimpleVT());
 150:   }
 151:   bool shouldConsiderGEPOffsetSplit() const override { return true; }
 152:   bool shouldSignExtendTypeInLibCall(Type *Ty, bool IsSigned) const override;
 153:   bool shouldExtendTypeInLibCall(EVT Type) const override;
 154: 
 155:   bool shouldAlignPointerArgs(CallInst *CI, unsigned &MinSize,
 156:                               Align &PrefAlign) const override;
 157: 
 158:   bool isFPImmVLDILegal(const APFloat &Imm, EVT VT) const;
 159:   LegalizeTypeAction getPreferredVectorAction(MVT VT) const override;
 160: 
 161:   void computeKnownBitsForTargetNode(const SDValue Op, KnownBits &Known,
 162:                                      const APInt &DemandedElts,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:                                      const SelectionDAG &DAG,
 164:                                      unsigned Depth) const override;
 165:   bool SimplifyDemandedBitsForTargetNode(SDValue Op, const APInt &DemandedBits,
 166:                                          const APInt &DemandedElts,
 167:                                          KnownBits &Known,
 168:                                          TargetLoweringOpt &TLO,
 169:                                          unsigned Depth) const override;
 170: 
 171:   bool shouldScalarizeBinop(SDValue VecOp) const override;
 172:   bool isExtractSubvectorCheap(EVT ResVT, EVT SrcVT,
 173:                                unsigned Index) const override;
 174:   bool isExtractVecEltCheap(EVT VT, unsigned Index) const override;
 175: 
 176:   /// Check if a constant splat can be generated using [x]vldi, where imm[12]
 177:   /// is 1.
 178:   std::pair<bool, uint64_t>
 179:   isImmVLDILegalForMode1(const APInt &SplatValue,
 180:                          const unsigned SplatBitSize) const;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181: 
 182: private:
 183:   /// Target-specific function used to lower LoongArch calling conventions.
 184:   typedef bool LoongArchCCAssignFn(const DataLayout &DL, LoongArchABI::ABI ABI,
 185:                                    unsigned ValNo, MVT ValVT,
 186:                                    CCValAssign::LocInfo LocInfo,
 187:                                    ISD::ArgFlagsTy ArgFlags, CCState &State,
 188:                                    bool IsRet, Type *OrigTy);
 189: 
 190:   void analyzeInputArgs(MachineFunction &MF, CCState &CCInfo,
 191:                         const SmallVectorImpl<ISD::InputArg> &Ins, bool IsRet,
 192:                         LoongArchCCAssignFn Fn) const;
 193:   void analyzeOutputArgs(MachineFunction &MF, CCState &CCInfo,
 194:                          const SmallVectorImpl<ISD::OutputArg> &Outs,
 195:                          bool IsRet, CallLoweringInfo *CLI,
 196:                          LoongArchCCAssignFn Fn) const;
 197: 
 198:   template <class NodeTy>
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   SDValue getAddr(NodeTy *N, SelectionDAG &DAG, CodeModel::Model M,
 200:                   bool IsLocal = true) const;
 201:   SDValue getStaticTLSAddr(GlobalAddressSDNode *N, SelectionDAG &DAG,
 202:                            unsigned Opc, bool UseGOT, bool Large = false) const;
 203:   SDValue getDynamicTLSAddr(GlobalAddressSDNode *N, SelectionDAG &DAG,
 204:                             unsigned Opc, bool Large = false) const;
 205:   SDValue getTLSDescAddr(GlobalAddressSDNode *N, SelectionDAG &DAG,
 206:                          unsigned Opc, bool Large = false) const;
 207:   SDValue lowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
 208:   SDValue lowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
 209:   SDValue lowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
 210:   SDValue lowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
 211:   SDValue lowerShiftLeftParts(SDValue Op, SelectionDAG &DAG) const;
 212:   SDValue lowerShiftRightParts(SDValue Op, SelectionDAG &DAG, bool IsSRA) const;
 213: 
 214:   MachineBasicBlock *
 215:   EmitInstrWithCustomInserter(MachineInstr &MI,
 216:                               MachineBasicBlock *BB) const override;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   SDValue lowerATOMIC_FENCE(SDValue Op, SelectionDAG &DAG) const;
 218:   SDValue lowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
 219:   SDValue lowerEH_DWARF_CFA(SDValue Op, SelectionDAG &DAG) const;
 220:   SDValue lowerFP_TO_SINT(SDValue Op, SelectionDAG &DAG) const;
 221:   SDValue lowerBITCAST(SDValue Op, SelectionDAG &DAG) const;
 222:   SDValue lowerUINT_TO_FP(SDValue Op, SelectionDAG &DAG) const;
 223:   SDValue lowerSINT_TO_FP(SDValue Op, SelectionDAG &DAG) const;
 224:   SDValue lowerVASTART(SDValue Op, SelectionDAG &DAG) const;
 225:   SDValue lowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;
 226:   SDValue lowerINTRINSIC_W_CHAIN(SDValue Op, SelectionDAG &DAG) const;
 227:   SDValue lowerINTRINSIC_VOID(SDValue Op, SelectionDAG &DAG) const;
 228:   SDValue lowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
 229:   SDValue lowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
 230:   SDValue lowerWRITE_REGISTER(SDValue Op, SelectionDAG &DAG) const;
 231:   SDValue lowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
 232:   SDValue lowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
 233:   SDValue lowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;
 234:   SDValue lowerCONCAT_VECTORS(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   SDValue lowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG) const;
 236:   SDValue lowerBITREVERSE(SDValue Op, SelectionDAG &DAG) const;
 237:   SDValue lowerSCALAR_TO_VECTOR(SDValue Op, SelectionDAG &DAG) const;
 238:   SDValue lowerPREFETCH(SDValue Op, SelectionDAG &DAG) const;
 239:   SDValue lowerSELECT(SDValue Op, SelectionDAG &DAG) const;
 240:   SDValue lowerBRCOND(SDValue Op, SelectionDAG &DAG) const;
 241:   SDValue lowerFP_TO_FP16(SDValue Op, SelectionDAG &DAG) const;
 242:   SDValue lowerFP16_TO_FP(SDValue Op, SelectionDAG &DAG) const;
 243:   SDValue lowerFP_TO_BF16(SDValue Op, SelectionDAG &DAG) const;
 244:   SDValue lowerBF16_TO_FP(SDValue Op, SelectionDAG &DAG) const;
 245:   SDValue lowerVECREDUCE_ADD(SDValue Op, SelectionDAG &DAG) const;
 246:   SDValue lowerVECREDUCE(SDValue Op, SelectionDAG &DAG) const;
 247:   SDValue lowerConstantFP(SDValue Op, SelectionDAG &DAG) const;
 248:   SDValue lowerSETCC(SDValue Op, SelectionDAG &DAG) const;
 249:   SDValue lowerRotate(SDValue Op, SelectionDAG &DAG) const;
 250:   SDValue lowerFP_ROUND(SDValue Op, SelectionDAG &DAG) const;
 251:   SDValue lowerFP_EXTEND(SDValue Op, SelectionDAG &DAG) const;
 252:   SDValue lowerSIGN_EXTEND_VECTOR_INREG(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-270 / 第 253-270 行
```cpp
 253: 
 254:   bool isFPImmLegal(const APFloat &Imm, EVT VT,
 255:                     bool ForCodeSize) const override;
 256: 
 257:   bool shouldInsertFencesForAtomic(const Instruction *I) const override;
 258: 
 259:   ConstraintType getConstraintType(StringRef Constraint) const override;
 260: 
 261:   InlineAsm::ConstraintCode
 262:   getInlineAsmMemConstraint(StringRef ConstraintCode) const override;
 263: 
 264:   std::pair<unsigned, const TargetRegisterClass *>
 265:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
 266:                                StringRef Constraint, MVT VT) const override;
 267: 
 268:   void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
 269:                                     std::vector<SDValue> &Ops,
 270:                                     SelectionDAG &DAG) const override;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 271-288 / 第 271-288 行
```cpp
 271: 
 272:   bool isEligibleForTailCallOptimization(
 273:       CCState &CCInfo, CallLoweringInfo &CLI, MachineFunction &MF,
 274:       const SmallVectorImpl<CCValAssign> &ArgLocs) const;
 275: 
 276:   bool
 277:   splitValueIntoRegisterParts(SelectionDAG &DAG, const SDLoc &DL, SDValue Val,
 278:                               SDValue *Parts, unsigned NumParts, MVT PartVT,
 279:                               std::optional<CallingConv::ID> CC) const override;
 280: 
 281:   SDValue
 282:   joinRegisterPartsIntoValue(SelectionDAG &DAG, const SDLoc &DL,
 283:                              const SDValue *Parts, unsigned NumParts,
 284:                              MVT PartVT, EVT ValueVT,
 285:                              std::optional<CallingConv::ID> CC) const override;
 286: 
 287:   /// Return the register type for a given MVT, ensuring vectors are treated
 288:   /// as a series of gpr sized integers.
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 289-301 / 第 289-301 行
```cpp
 289:   MVT getRegisterTypeForCallingConv(LLVMContext &Context, CallingConv::ID CC,
 290:                                     EVT VT) const override;
 291: 
 292:   /// Return the number of registers for a given MVT, ensuring vectors are
 293:   /// treated as a series of gpr sized integers.
 294:   unsigned getNumRegistersForCallingConv(LLVMContext &Context,
 295:                                          CallingConv::ID CC,
 296:                                          EVT VT) const override;
 297: };
 298: 
 299: } // end namespace llvm
 300: 
 301: #endif // LLVM_LIB_TARGET_LOONGARCH_LOONGARCHISELLOWERING_H
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArch.h`
- `llvm/CodeGen/CallingConvLower.h`
- `llvm/CodeGen/SelectionDAG.h`
- `llvm/CodeGen/SelectionDAGNodes.h`
- `llvm/CodeGen/TargetLowering.h`
