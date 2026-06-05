# SystemZISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZISelLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责实现目标后端的 SelectionDAG 降低。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- SystemZISelLowering.h - SystemZ DAG lowering interface --*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that SystemZ uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZISELLOWERING_H
  15: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZISELLOWERING_H
  16: 
  17: #include "SystemZ.h"
  18: #include "SystemZInstrInfo.h"
  19: #include "llvm/CodeGen/MachineBasicBlock.h"
  20: #include "llvm/CodeGen/SelectionDAG.h"
  21: #include "llvm/CodeGen/TargetLowering.h"
  22: #include <optional>
  23: 
  24: namespace llvm {
  25: 
  26: namespace SystemZICMP {
  27: // Describes whether an integer comparison needs to be signed or unsigned,
  28: // or whether either type is OK.
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZ.h`, `SystemZInstrInfo.h`, `MachineBasicBlock.h`, `SelectionDAG.h`, `TargetLowering.h`, `optional` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZ.h`, `SystemZInstrInfo.h`, `MachineBasicBlock.h`, `SelectionDAG.h`, `TargetLowering.h`, `optional` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29: enum {
  30:   Any,
  31:   UnsignedOnly,
  32:   SignedOnly
  33: };
  34: } // end namespace SystemZICMP
  35: 
  36: class SystemZSubtarget;
  37: 
  38: class SystemZTargetLowering : public TargetLowering {
  39: public:
  40:   explicit SystemZTargetLowering(const TargetMachine &TM,
  41:                                  const SystemZSubtarget &STI);
  42: 
  43:   bool useSoftFloat() const override;
  44: 
  45:   // Override TargetLowering.
  46:   MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override {
  47:     return MVT::i32;
  48:   }
  49:   unsigned getVectorIdxWidth(const DataLayout &DL) const override {
  50:     // Only the lower 12 bits of an element index are used, so we don't
  51:     // want to clobber the upper 32 bits of a GPR unnecessarily.
  52:     return 32;
  53:   }
  54:   TargetLoweringBase::LegalizeTypeAction getPreferredVectorAction(MVT VT)
  55:     const override {
  56:     // Widen subvectors to the full width rather than promoting integer
```
- **EN**: This block declares or refines TableGen records such as `SystemZSubtarget`, `SystemZTargetLowering`. The range implements or declares functions including `getPreferredVectorAction`.
- **CN**: 该代码块声明或细化了 `SystemZSubtarget`, `SystemZTargetLowering` 等 TableGen 记录。 这一段实现或声明了 `getPreferredVectorAction` 等函数。

### Lines 57-84 / 第 57-84 行
```cpp
  57:     // elements.  This is better because:
  58:     //
  59:     // (a) it means that we can handle the ABI for passing and returning
  60:     //     sub-128 vectors without having to handle them as legal types.
  61:     //
  62:     // (b) we don't have instructions to extend on load and truncate on store,
  63:     //     so promoting the integers is less efficient.
  64:     //
  65:     // (c) there are no multiplication instructions for the widest integer
  66:     //     type (v2i64).
  67: 
  68:     // Expand (narrow) f16 vectors during type legalization to avoid
  69:     // operations for all elements as with expansion after widening.
  70:     if (VT.getScalarType() == MVT::f16)
  71:       return VT.getVectorElementCount().isScalar() ? TypeScalarizeVector
  72:                                                    : TypeSplitVector;
  73:     if (VT.getScalarSizeInBits() % 8 == 0)
  74:       return TypeWidenVector;
  75:     return TargetLoweringBase::getPreferredVectorAction(VT);
  76:   }
  77:   unsigned
  78:   getNumRegisters(LLVMContext &Context, EVT VT,
  79:                   std::optional<MVT> RegisterVT) const override {
  80:     // i128 inline assembly operand.
  81:     if (VT == MVT::i128 && RegisterVT && *RegisterVT == MVT::Untyped)
  82:       return 1;
  83:     return TargetLowering::getNumRegisters(Context, VT);
  84:   }
```
- **EN**: The range implements or declares functions including `getNumRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getNumRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   unsigned
  86:   getVectorTypeBreakdownForCallingConv(LLVMContext &Context, CallingConv::ID CC,
  87:                                        EVT VT, EVT &IntermediateVT,
  88:                                        unsigned &NumIntermediates,
  89:                                        MVT &RegisterVT) const override;
  90:   MVT getRegisterTypeForCallingConv(LLVMContext &Context, CallingConv::ID CC,
  91:                                     EVT VT) const override;
  92:   unsigned getNumRegistersForCallingConv(LLVMContext &Context,
  93:                                          CallingConv::ID CC,
  94:                                          EVT VT) const override;
  95:   bool isCheapToSpeculateCtlz(Type *) const override { return true; }
  96:   bool isCheapToSpeculateCttz(Type *) const override { return true; }
  97:   bool preferZeroCompareBranch() const override { return true; }
  98:   bool isMaskAndCmp0FoldingBeneficial(const Instruction &AndI) const override {
  99:     ConstantInt* Mask = dyn_cast<ConstantInt>(AndI.getOperand(1));
 100:     return Mask && Mask->getValue().isIntN(16);
 101:   }
 102:   bool convertSetCCLogicToBitwiseLogic(EVT VT) const override {
 103:     return VT.isScalarInteger();
 104:   }
 105:   EVT getSetCCResultType(const DataLayout &DL, LLVMContext &,
 106:                          EVT) const override;
 107:   bool isFMAFasterThanFMulAndFAdd(const MachineFunction &MF,
 108:                                   EVT VT) const override;
 109:   bool isFPImmLegal(const APFloat &Imm, EVT VT,
 110:                     bool ForCodeSize) const override;
 111:   bool ShouldShrinkFPConstant(EVT VT) const override {
 112:     // Do not shrink 64-bit FP constpool entries since LDEB is slower than
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 113-140 / 第 113-140 行
```cpp
 113:     // LD, and having the full constant in memory enables reg/mem opcodes.
 114:     return VT != MVT::f64;
 115:   }
 116:   MachineBasicBlock *emitEHSjLjSetJmp(MachineInstr &MI,
 117:                                       MachineBasicBlock *MBB) const;
 118: 
 119:   MachineBasicBlock *emitEHSjLjLongJmp(MachineInstr &MI,
 120:                                        MachineBasicBlock *MBB) const;
 121: 
 122:   bool hasInlineStackProbe(const MachineFunction &MF) const override;
 123:   AtomicExpansionKind shouldCastAtomicLoadInIR(LoadInst *LI) const override;
 124:   AtomicExpansionKind shouldCastAtomicStoreInIR(StoreInst *SI) const override;
 125:   AtomicExpansionKind
 126:   shouldExpandAtomicRMWInIR(const AtomicRMWInst *RMW) const override;
 127:   bool isLegalICmpImmediate(int64_t Imm) const override;
 128:   bool isLegalAddImmediate(int64_t Imm) const override;
 129:   bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM, Type *Ty,
 130:                              unsigned AS,
 131:                              Instruction *I = nullptr) const override;
 132:   bool allowsMisalignedMemoryAccesses(EVT VT, unsigned AS, Align Alignment,
 133:                                       MachineMemOperand::Flags Flags,
 134:                                       unsigned *Fast) const override;
 135:   bool findOptimalMemOpLowering(LLVMContext &Context, std::vector<EVT> &MemOps,
 136:                                 unsigned Limit, const MemOp &Op, unsigned DstAS,
 137:                                 unsigned SrcAS,
 138:                                 const AttributeList &FuncAttributes,
 139:                                 EVT *LargestVT = nullptr) const override;
 140:   EVT getOptimalMemOpType(LLVMContext &Context, const MemOp &Op,
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 141-168 / 第 141-168 行
```cpp
 141:                           const AttributeList &FuncAttributes) const override;
 142:   bool isTruncateFree(Type *, Type *) const override;
 143:   bool isTruncateFree(EVT, EVT) const override;
 144: 
 145:   bool shouldFormOverflowOp(unsigned Opcode, EVT VT,
 146:                             bool MathUsed) const override {
 147:     // Form add and sub with overflow intrinsics regardless of any extra
 148:     // users of the math result.
 149:     return VT == MVT::i32 || VT == MVT::i64 || VT == MVT::i128;
 150:   }
 151: 
 152:   bool shouldConsiderGEPOffsetSplit() const override { return true; }
 153: 
 154:   bool preferSelectsOverBooleanArithmetic(EVT VT) const override {
 155:     return true;
 156:   }
 157: 
 158:   // This function currently returns cost for srl/ipm/cc sequence for merging.
 159:   CondMergingParams
 160:   getJumpConditionMergingParams(Instruction::BinaryOps Opc, const Value *Lhs,
 161:                                 const Value *Rhs) const override;
 162: 
 163:   // Handle Lowering flag assembly outputs.
 164:   SDValue LowerAsmOutputForConstraint(SDValue &Chain, SDValue &Flag,
 165:                                       const SDLoc &DL,
 166:                                       const AsmOperandInfo &Constraint,
 167:                                       SelectionDAG &DAG) const override;
 168: 
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 169-196 / 第 169-196 行
```cpp
 169:   std::pair<unsigned, const TargetRegisterClass *>
 170:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
 171:                                StringRef Constraint, MVT VT) const override;
 172:   TargetLowering::ConstraintType
 173:   getConstraintType(StringRef Constraint) const override;
 174:   TargetLowering::ConstraintWeight
 175:     getSingleConstraintMatchWeight(AsmOperandInfo &info,
 176:                                    const char *constraint) const override;
 177:   void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
 178:                                     std::vector<SDValue> &Ops,
 179:                                     SelectionDAG &DAG) const override;
 180: 
 181:   InlineAsm::ConstraintCode
 182:   getInlineAsmMemConstraint(StringRef ConstraintCode) const override {
 183:     if (ConstraintCode.size() == 1) {
 184:       switch(ConstraintCode[0]) {
 185:       default:
 186:         break;
 187:       case 'o':
 188:         return InlineAsm::ConstraintCode::o;
 189:       case 'Q':
 190:         return InlineAsm::ConstraintCode::Q;
 191:       case 'R':
 192:         return InlineAsm::ConstraintCode::R;
 193:       case 'S':
 194:         return InlineAsm::ConstraintCode::S;
 195:       case 'T':
 196:         return InlineAsm::ConstraintCode::T;
```
- **EN**: The range implements or declares functions including `getInlineAsmMemConstraint`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getInlineAsmMemConstraint` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```cpp
 197:       }
 198:     } else if (ConstraintCode.size() == 2 && ConstraintCode[0] == 'Z') {
 199:       switch (ConstraintCode[1]) {
 200:       default:
 201:         break;
 202:       case 'Q':
 203:         return InlineAsm::ConstraintCode::ZQ;
 204:       case 'R':
 205:         return InlineAsm::ConstraintCode::ZR;
 206:       case 'S':
 207:         return InlineAsm::ConstraintCode::ZS;
 208:       case 'T':
 209:         return InlineAsm::ConstraintCode::ZT;
 210:       }
 211:     }
 212:     return TargetLowering::getInlineAsmMemConstraint(ConstraintCode);
 213:   }
 214: 
 215:   Register getRegisterByName(const char *RegName, LLT VT,
 216:                              const MachineFunction &MF) const override;
 217: 
 218:   /// If a physical register, this returns the register that receives the
 219:   /// exception address on entry to an EH pad.
 220:   Register
 221:   getExceptionPointerRegister(const Constant *PersonalityFn) const override;
 222: 
 223:   /// If a physical register, this returns the register that receives the
 224:   /// exception typeid on entry to a landing pad.
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:   Register
 226:   getExceptionSelectorRegister(const Constant *PersonalityFn) const override;
 227: 
 228:   /// Override to support customized stack guard loading.
 229:   bool useLoadStackGuardNode(const Module &M) const override { return true; }
 230:   void
 231:   insertSSPDeclarations(Module &M,
 232:                         const LibcallLoweringInfo &Libcalls) const override {}
 233: 
 234:   MachineBasicBlock *
 235:   EmitInstrWithCustomInserter(MachineInstr &MI,
 236:                               MachineBasicBlock *BB) const override;
 237:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
 238:   void LowerOperationWrapper(SDNode *N, SmallVectorImpl<SDValue> &Results,
 239:                              SelectionDAG &DAG) const override;
 240:   void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue>&Results,
 241:                           SelectionDAG &DAG) const override;
 242:   const MCPhysReg *getScratchRegisters(CallingConv::ID CC) const override;
 243:   bool allowTruncateForTailCall(Type *, Type *) const override;
 244:   bool mayBeEmittedAsTailCall(const CallInst *CI) const override;
 245:   bool splitValueIntoRegisterParts(
 246:       SelectionDAG & DAG, const SDLoc &DL, SDValue Val, SDValue *Parts,
 247:       unsigned NumParts, MVT PartVT, std::optional<CallingConv::ID> CC)
 248:       const override;
 249:   SDValue joinRegisterPartsIntoValue(
 250:       SelectionDAG & DAG, const SDLoc &DL, const SDValue *Parts,
 251:       unsigned NumParts, MVT PartVT, EVT ValueVT,
 252:       std::optional<CallingConv::ID> CC) const override;
```
- **EN**: The range implements or declares functions including `splitValueIntoRegisterParts`.
- **CN**: 这一段实现或声明了 `splitValueIntoRegisterParts` 等函数。

### Lines 253-280 / 第 253-280 行
```cpp
 253:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
 254:                                bool isVarArg,
 255:                                const SmallVectorImpl<ISD::InputArg> &Ins,
 256:                                const SDLoc &DL, SelectionDAG &DAG,
 257:                                SmallVectorImpl<SDValue> &InVals) const override;
 258:   SDValue LowerCall(CallLoweringInfo &CLI,
 259:                     SmallVectorImpl<SDValue> &InVals) const override;
 260: 
 261:   std::pair<SDValue, SDValue>
 262:   makeExternalCall(SDValue Chain, SelectionDAG &DAG, const char *CalleeName,
 263:                    EVT RetVT, ArrayRef<SDValue> Ops, CallingConv::ID CallConv,
 264:                    bool IsSigned, SDLoc DL, bool DoesNotReturn,
 265:                    bool IsReturnValueUsed) const;
 266: 
 267:   SDValue useLibCall(SelectionDAG &DAG, RTLIB::Libcall LC, MVT VT, SDValue Arg,
 268:                      SDLoc DL, SDValue Chain, bool IsStrict) const;
 269: 
 270:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
 271:                       bool isVarArg,
 272:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 273:                       LLVMContext &Context,
 274:                       const Type *RetTy) const override;
 275:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 276:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 277:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
 278:                       SelectionDAG &DAG) const override;
 279:   SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
 280: 
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 281-308 / 第 281-308 行
```cpp
 281:   /// Determine which of the bits specified in Mask are known to be either
 282:   /// zero or one and return them in the KnownZero/KnownOne bitsets.
 283:   void computeKnownBitsForTargetNode(const SDValue Op,
 284:                                      KnownBits &Known,
 285:                                      const APInt &DemandedElts,
 286:                                      const SelectionDAG &DAG,
 287:                                      unsigned Depth = 0) const override;
 288: 
 289:   /// Determine the number of bits in the operation that are sign bits.
 290:   unsigned ComputeNumSignBitsForTargetNode(SDValue Op,
 291:                                            const APInt &DemandedElts,
 292:                                            const SelectionDAG &DAG,
 293:                                            unsigned Depth) const override;
 294: 
 295:   bool isGuaranteedNotToBeUndefOrPoisonForTargetNode(
 296:       SDValue Op, const APInt &DemandedElts, const SelectionDAG &DAG,
 297:       UndefPoisonKind Kind, unsigned Depth) const override;
 298: 
 299:   ISD::NodeType getExtendForAtomicOps() const override {
 300:     return ISD::ANY_EXTEND;
 301:   }
 302:   ISD::NodeType getExtendForAtomicCmpSwapArg() const override {
 303:     return ISD::ZERO_EXTEND;
 304:   }
 305: 
 306:   bool supportSwiftError() const override {
 307:     return true;
 308:   }
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 309-336 / 第 309-336 行
```cpp
 309: 
 310:   unsigned getStackProbeSize(const MachineFunction &MF) const;
 311:   bool hasAndNot(SDValue Y) const override;
 312: 
 313: private:
 314:   const SystemZSubtarget &Subtarget;
 315: 
 316:   // Implement LowerOperation for individual opcodes.
 317:   SDValue getVectorCmp(SelectionDAG &DAG, unsigned Opcode,
 318:                        const SDLoc &DL, EVT VT,
 319:                        SDValue CmpOp0, SDValue CmpOp1, SDValue Chain) const;
 320:   SDValue lowerVectorSETCC(SelectionDAG &DAG, const SDLoc &DL,
 321:                            EVT VT, ISD::CondCode CC,
 322:                            SDValue CmpOp0, SDValue CmpOp1,
 323:                            SDValue Chain = SDValue(),
 324:                            bool IsSignaling = false) const;
 325:   SDValue lowerSETCC(SDValue Op, SelectionDAG &DAG) const;
 326:   SDValue lowerSTRICT_FSETCC(SDValue Op, SelectionDAG &DAG,
 327:                              bool IsSignaling) const;
 328:   SDValue lowerBR_CC(SDValue Op, SelectionDAG &DAG) const;
 329:   SDValue lowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;
 330:   SDValue lowerGlobalAddress(GlobalAddressSDNode *Node,
 331:                              SelectionDAG &DAG) const;
 332:   SDValue lowerTLSGetOffset(GlobalAddressSDNode *Node,
 333:                             SelectionDAG &DAG, unsigned Opcode,
 334:                             SDValue GOTOffset) const;
 335:   SDValue lowerThreadPointer(const SDLoc &DL, SelectionDAG &DAG) const;
 336:   SDValue lowerGlobalTLSAddress(GlobalAddressSDNode *Node,
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 337-364 / 第 337-364 行
```cpp
 337:                                 SelectionDAG &DAG) const;
 338:   SDValue lowerBlockAddress(BlockAddressSDNode *Node,
 339:                             SelectionDAG &DAG) const;
 340:   SDValue lowerJumpTable(JumpTableSDNode *JT, SelectionDAG &DAG) const;
 341:   SDValue lowerConstantPool(ConstantPoolSDNode *CP, SelectionDAG &DAG) const;
 342:   SDValue lowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
 343:   SDValue lowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
 344:   SDValue lowerVASTART(SDValue Op, SelectionDAG &DAG) const;
 345:   SDValue lowerVASTART_ELF(SDValue Op, SelectionDAG &DAG) const;
 346:   SDValue lowerVASTART_XPLINK(SDValue Op, SelectionDAG &DAG) const;
 347:   SDValue lowerVACOPY(SDValue Op, SelectionDAG &DAG) const;
 348:   SDValue lowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
 349:   SDValue lowerDYNAMIC_STACKALLOC_ELF(SDValue Op, SelectionDAG &DAG) const;
 350:   SDValue lowerDYNAMIC_STACKALLOC_XPLINK(SDValue Op, SelectionDAG &DAG) const;
 351:   SDValue lowerGET_DYNAMIC_AREA_OFFSET(SDValue Op, SelectionDAG &DAG) const;
 352:   SDValue lowerMULH(SDValue Op, SelectionDAG &DAG, unsigned Opcode) const;
 353:   SDValue lowerSMUL_LOHI(SDValue Op, SelectionDAG &DAG) const;
 354:   SDValue lowerUMUL_LOHI(SDValue Op, SelectionDAG &DAG) const;
 355:   SDValue lowerSDIVREM(SDValue Op, SelectionDAG &DAG) const;
 356:   SDValue lowerUDIVREM(SDValue Op, SelectionDAG &DAG) const;
 357:   SDValue lowerXALUO(SDValue Op, SelectionDAG &DAG) const;
 358:   SDValue lowerUADDSUBO_CARRY(SDValue Op, SelectionDAG &DAG) const;
 359:   SDValue lowerBITCAST(SDValue Op, SelectionDAG &DAG) const;
 360:   SDValue lowerOR(SDValue Op, SelectionDAG &DAG) const;
 361:   SDValue lowerCTPOP(SDValue Op, SelectionDAG &DAG) const;
 362:   SDValue lowerVECREDUCE_ADD(SDValue Op, SelectionDAG &DAG) const;
 363:   SDValue lowerATOMIC_FENCE(SDValue Op, SelectionDAG &DAG) const;
 364:   SDValue lowerATOMIC_LOAD(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   SDValue lowerATOMIC_STORE(SDValue Op, SelectionDAG &DAG) const;
 366:   SDValue lowerATOMIC_LDST_I128(SDValue Op, SelectionDAG &DAG) const;
 367:   SDValue lowerATOMIC_LOAD_OP(SDValue Op, SelectionDAG &DAG,
 368:                               unsigned Opcode) const;
 369:   SDValue lowerATOMIC_LOAD_SUB(SDValue Op, SelectionDAG &DAG) const;
 370:   SDValue lowerATOMIC_CMP_SWAP(SDValue Op, SelectionDAG &DAG) const;
 371:   SDValue lowerSTACKSAVE(SDValue Op, SelectionDAG &DAG) const;
 372:   SDValue lowerSTACKRESTORE(SDValue Op, SelectionDAG &DAG) const;
 373:   SDValue lowerPREFETCH(SDValue Op, SelectionDAG &DAG) const;
 374:   SDValue lowerINTRINSIC_W_CHAIN(SDValue Op, SelectionDAG &DAG) const;
 375:   SDValue lowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;
 376:   bool isVectorElementLoad(SDValue Op) const;
 377:   SDValue buildVector(SelectionDAG &DAG, const SDLoc &DL, EVT VT,
 378:                       SmallVectorImpl<SDValue> &Elems) const;
 379:   SDValue lowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;
 380:   SDValue lowerVECTOR_SHUFFLE(SDValue Op, SelectionDAG &DAG) const;
 381:   SDValue lowerSCALAR_TO_VECTOR(SDValue Op, SelectionDAG &DAG) const;
 382:   SDValue lowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
 383:   SDValue lowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
 384:   SDValue lowerSIGN_EXTEND_VECTOR_INREG(SDValue Op, SelectionDAG &DAG) const;
 385:   SDValue lowerZERO_EXTEND_VECTOR_INREG(SDValue Op, SelectionDAG &DAG) const;
 386:   SDValue lowerShift(SDValue Op, SelectionDAG &DAG, unsigned ByScalar) const;
 387:   SDValue lowerFSHL(SDValue Op, SelectionDAG &DAG) const;
 388:   SDValue lowerFSHR(SDValue Op, SelectionDAG &DAG) const;
 389:   SDValue lowerFP_EXTEND(SDValue Op, SelectionDAG &DAG) const;
 390:   SDValue lower_FP_TO_INT(SDValue Op, SelectionDAG &DAG) const;
 391:   SDValue lower_INT_TO_FP(SDValue Op, SelectionDAG &DAG) const;
 392:   SDValue lowerLoadF16(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   SDValue lowerStoreF16(SDValue Op, SelectionDAG &DAG) const;
 394: 
 395:   SDValue lowerIS_FPCLASS(SDValue Op, SelectionDAG &DAG) const;
 396:   SDValue lowerGET_ROUNDING(SDValue Op, SelectionDAG &DAG) const;
 397:   SDValue lowerREADCYCLECOUNTER(SDValue Op, SelectionDAG &DAG) const;
 398: 
 399:   bool canTreatAsByteVector(EVT VT) const;
 400:   SDValue combineExtract(const SDLoc &DL, EVT ElemVT, EVT VecVT, SDValue OrigOp,
 401:                          unsigned Index, DAGCombinerInfo &DCI,
 402:                          bool Force) const;
 403:   SDValue combineTruncateExtract(const SDLoc &DL, EVT TruncVT, SDValue Op,
 404:                                  DAGCombinerInfo &DCI) const;
 405:   SDValue combineZERO_EXTEND(SDNode *N, DAGCombinerInfo &DCI) const;
 406:   SDValue combineSIGN_EXTEND(SDNode *N, DAGCombinerInfo &DCI) const;
 407:   SDValue combineSIGN_EXTEND_INREG(SDNode *N, DAGCombinerInfo &DCI) const;
 408:   SDValue combineMERGE(SDNode *N, DAGCombinerInfo &DCI) const;
 409:   bool canLoadStoreByteSwapped(EVT VT) const;
 410:   SDValue combineLOAD(SDNode *N, DAGCombinerInfo &DCI) const;
 411:   SDValue combineSTORE(SDNode *N, DAGCombinerInfo &DCI) const;
 412:   SDValue combineVECTOR_SHUFFLE(SDNode *N, DAGCombinerInfo &DCI) const;
 413:   SDValue combineEXTRACT_VECTOR_ELT(SDNode *N, DAGCombinerInfo &DCI) const;
 414:   SDValue combineJOIN_DWORDS(SDNode *N, DAGCombinerInfo &DCI) const;
 415:   SDValue combineFP_ROUND(SDNode *N, DAGCombinerInfo &DCI) const;
 416:   SDValue combineFP_EXTEND(SDNode *N, DAGCombinerInfo &DCI) const;
 417:   SDValue combineINT_TO_FP(SDNode *N, DAGCombinerInfo &DCI) const;
 418:   SDValue combineFCOPYSIGN(SDNode *N, DAGCombinerInfo &DCI) const;
 419:   SDValue combineBSWAP(SDNode *N, DAGCombinerInfo &DCI) const;
 420:   SDValue combineSETCC(SDNode *N, DAGCombinerInfo &DCI) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 421-448 / 第 421-448 行
```cpp
 421:   SDValue combineBR_CCMASK(SDNode *N, DAGCombinerInfo &DCI) const;
 422:   SDValue combineSELECT_CCMASK(SDNode *N, DAGCombinerInfo &DCI) const;
 423:   SDValue combineGET_CCMASK(SDNode *N, DAGCombinerInfo &DCI) const;
 424:   SDValue combineShiftToMulAddHigh(SDNode *N, DAGCombinerInfo &DCI) const;
 425:   SDValue combineMUL(SDNode *N, DAGCombinerInfo &DCI) const;
 426:   SDValue combineIntDIVREM(SDNode *N, DAGCombinerInfo &DCI) const;
 427:   SDValue combineINTRINSIC(SDNode *N, DAGCombinerInfo &DCI) const;
 428: 
 429:   SDValue unwrapAddress(SDValue N) const override;
 430: 
 431:   // If the last instruction before MBBI in MBB was some form of COMPARE,
 432:   // try to replace it with a COMPARE AND BRANCH just before MBBI.
 433:   // CCMask and Target are the BRC-like operands for the branch.
 434:   // Return true if the change was made.
 435:   bool convertPrevCompareToBranch(MachineBasicBlock *MBB,
 436:                                   MachineBasicBlock::iterator MBBI,
 437:                                   unsigned CCMask,
 438:                                   MachineBasicBlock *Target) const;
 439: 
 440:   // Implement EmitInstrWithCustomInserter for individual operation types.
 441:   MachineBasicBlock *emitAdjCallStack(MachineInstr &MI,
 442:                                       MachineBasicBlock *BB) const;
 443:   MachineBasicBlock *emitSelect(MachineInstr &MI, MachineBasicBlock *BB) const;
 444:   MachineBasicBlock *emitCondStore(MachineInstr &MI, MachineBasicBlock *BB,
 445:                                    unsigned StoreOpcode, unsigned STOCOpcode,
 446:                                    bool Invert) const;
 447:   MachineBasicBlock *emitICmp128Hi(MachineInstr &MI, MachineBasicBlock *BB,
 448:                                    bool Unsigned) const;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 449-476 / 第 449-476 行
```cpp
 449:   MachineBasicBlock *emitPair128(MachineInstr &MI,
 450:                                  MachineBasicBlock *MBB) const;
 451:   MachineBasicBlock *emitExt128(MachineInstr &MI, MachineBasicBlock *MBB,
 452:                                 bool ClearEven) const;
 453:   MachineBasicBlock *emitAtomicLoadBinary(MachineInstr &MI,
 454:                                           MachineBasicBlock *BB,
 455:                                           unsigned BinOpcode,
 456:                                           bool Invert = false) const;
 457:   MachineBasicBlock *emitAtomicLoadMinMax(MachineInstr &MI,
 458:                                           MachineBasicBlock *MBB,
 459:                                           unsigned CompareOpcode,
 460:                                           unsigned KeepOldMask) const;
 461:   MachineBasicBlock *emitAtomicCmpSwapW(MachineInstr &MI,
 462:                                         MachineBasicBlock *BB) const;
 463:   MachineBasicBlock *emitMemMemWrapper(MachineInstr &MI, MachineBasicBlock *BB,
 464:                                        unsigned Opcode,
 465:                                        bool IsMemset = false) const;
 466:   MachineBasicBlock *emitStringWrapper(MachineInstr &MI, MachineBasicBlock *BB,
 467:                                        unsigned Opcode) const;
 468:   MachineBasicBlock *emitTransactionBegin(MachineInstr &MI,
 469:                                           MachineBasicBlock *MBB,
 470:                                           unsigned Opcode, bool NoFloat) const;
 471:   MachineBasicBlock *emitLoadAndTestCmp0(MachineInstr &MI,
 472:                                          MachineBasicBlock *MBB,
 473:                                          unsigned Opcode) const;
 474:   MachineBasicBlock *emitProbedAlloca(MachineInstr &MI,
 475:                                       MachineBasicBlock *MBB) const;
 476: 
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 477-504 / 第 477-504 行
```cpp
 477:   SDValue getBackchainAddress(SDValue SP, SelectionDAG &DAG) const;
 478: 
 479:   MachineMemOperand::Flags
 480:   getTargetMMOFlags(const Instruction &I) const override;
 481:   const TargetRegisterClass *getRepRegClassFor(MVT VT) const override;
 482: 
 483: private:
 484:   bool isInternal(const Function *Fn) const;
 485:   mutable std::map<const Function *, bool> IsInternalCache;
 486:   void verifyNarrowIntegerArgs_Call(const SmallVectorImpl<ISD::OutputArg> &Outs,
 487:                                     const Function *F, SDValue Callee) const;
 488:   void verifyNarrowIntegerArgs_Ret(const SmallVectorImpl<ISD::OutputArg> &Outs,
 489:                                    const Function *F) const;
 490:   bool
 491:   verifyNarrowIntegerArgs(const SmallVectorImpl<ISD::OutputArg> &Outs) const;
 492: 
 493: public:
 494: };
 495: 
 496: struct SystemZVectorConstantInfo {
 497: private:
 498:   APInt IntBits;             // The 128 bits as an integer.
 499:   APInt SplatBits;           // Smallest splat value.
 500:   APInt SplatUndef;          // Bits correspoding to undef operands of the BVN.
 501:   unsigned SplatBitSize = 0;
 502:   bool isFP128 = false;
 503: public:
 504:   unsigned Opcode = 0;
```
- **EN**: It introduces interface types such as `SystemZVectorConstantInfo`, shaping how other backend components interact with this file.
- **CN**: 它引入了 `SystemZVectorConstantInfo` 等接口类型，定义了其他后端组件与本文件交互的方式。

### Lines 505-518 / 第 505-518 行
```cpp
 505:   SmallVector<unsigned, 2> OpVals;
 506:   MVT VecVT;
 507:   SystemZVectorConstantInfo(APInt IntImm);
 508:   SystemZVectorConstantInfo(APFloat FPImm)
 509:       : SystemZVectorConstantInfo(FPImm.bitcastToAPInt()) {
 510:     isFP128 = (&FPImm.getSemantics() == &APFloat::IEEEquad());
 511:   }
 512:   SystemZVectorConstantInfo(BuildVectorSDNode *BVN);
 513:   bool isVectorConstantLegal(const SystemZSubtarget &Subtarget);
 514: };
 515: 
 516: } // end namespace llvm
 517: 
 518: #endif
```
- **EN**: The range implements or declares functions including `SystemZVectorConstantInfo`.
- **CN**: 这一段实现或声明了 `SystemZVectorConstantInfo` 等函数。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。

## Dependencies / 依赖关系
- `SystemZ.h`
- `SystemZInstrInfo.h`
- `llvm/CodeGen/MachineBasicBlock.h`
- `llvm/CodeGen/SelectionDAG.h`
- `llvm/CodeGen/TargetLowering.h`
- `optional`
