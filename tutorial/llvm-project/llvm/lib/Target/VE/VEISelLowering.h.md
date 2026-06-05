# VEISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VEISelLowering.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEISelLowering.h - VE DAG Lowering Interface ------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-16
```cpp
//
// This file defines the interfaces that VE uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_VE_VEISELLOWERING_H
#define LLVM_LIB_TARGET_VE_VEISELLOWERING_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 17-23
```cpp
#include "VE.h"
#include "llvm/CodeGen/TargetLowering.h"

namespace llvm {
class VESubtarget;

/// Convert a DAG integer condition code to a VE ICC condition.
```
- **EN**: Pulls in the headers needed for this implementation, including `VE.h`, `llvm/CodeGen/TargetLowering.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VE.h`, `llvm/CodeGen/TargetLowering.h`。

### Lines 24-37
```cpp
inline static VECC::CondCode intCondCode2Icc(ISD::CondCode CC) {
  switch (CC) {
  default:
    llvm_unreachable("Unknown integer condition code!");
  case ISD::SETEQ:
    return VECC::CC_IEQ;
  case ISD::SETNE:
    return VECC::CC_INE;
  case ISD::SETLT:
    return VECC::CC_IL;
  case ISD::SETGT:
    return VECC::CC_IG;
  case ISD::SETLE:
    return VECC::CC_ILE;
```
- **EN**: Implements logic around `intCondCode2Icc`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `intCondCode2Icc`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 38-50
```cpp
  case ISD::SETGE:
    return VECC::CC_IGE;
  case ISD::SETULT:
    return VECC::CC_IL;
  case ISD::SETULE:
    return VECC::CC_ILE;
  case ISD::SETUGT:
    return VECC::CC_IG;
  case ISD::SETUGE:
    return VECC::CC_IGE;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 51-64
```cpp
/// Convert a DAG floating point condition code to a VE FCC condition.
inline static VECC::CondCode fpCondCode2Fcc(ISD::CondCode CC) {
  switch (CC) {
  default:
    llvm_unreachable("Unknown fp condition code!");
  case ISD::SETFALSE:
    return VECC::CC_AF;
  case ISD::SETEQ:
  case ISD::SETOEQ:
    return VECC::CC_EQ;
  case ISD::SETNE:
  case ISD::SETONE:
    return VECC::CC_NE;
  case ISD::SETLT:
```
- **EN**: Implements logic around `fpCondCode2Fcc`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `fpCondCode2Fcc`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 65-78
```cpp
  case ISD::SETOLT:
    return VECC::CC_L;
  case ISD::SETGT:
  case ISD::SETOGT:
    return VECC::CC_G;
  case ISD::SETLE:
  case ISD::SETOLE:
    return VECC::CC_LE;
  case ISD::SETGE:
  case ISD::SETOGE:
    return VECC::CC_GE;
  case ISD::SETO:
    return VECC::CC_NUM;
  case ISD::SETUO:
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 79-92
```cpp
    return VECC::CC_NAN;
  case ISD::SETUEQ:
    return VECC::CC_EQNAN;
  case ISD::SETUNE:
    return VECC::CC_NENAN;
  case ISD::SETULT:
    return VECC::CC_LNAN;
  case ISD::SETUGT:
    return VECC::CC_GNAN;
  case ISD::SETULE:
    return VECC::CC_LENAN;
  case ISD::SETUGE:
    return VECC::CC_GENAN;
  case ISD::SETTRUE:
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 93-101
```cpp
    return VECC::CC_AT;
  }
}

/// getImmVal - get immediate representation of integer value
inline static uint64_t getImmVal(const ConstantSDNode *N) {
  return N->getSExtValue();
}

```
- **EN**: Implements logic around `getImmVal`, `getSExtValue`; this block returns target-specific results.
- **CN**: 围绕 `getImmVal`, `getSExtValue` 实现具体逻辑；这一段返回目标相关结果。

### Lines 102-112
```cpp
/// getFpImmVal - get immediate representation of floating point value
inline static uint64_t getFpImmVal(const ConstantFPSDNode *N) {
  const APInt &Imm = N->getValueAPF().bitcastToAPInt();
  uint64_t Val = Imm.getZExtValue();
  if (Imm.getBitWidth() == 32) {
    // Immediate value of float place places at higher bits on VE.
    Val <<= 32;
  }
  return Val;
}

```
- **EN**: Implements logic around `getFpImmVal`, `getValueAPF`, `getZExtValue`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getFpImmVal`, `getValueAPF`, `getZExtValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 113-121
```cpp
class VECustomDAG;

class VETargetLowering : public TargetLowering {
  const VESubtarget *Subtarget;

  void initRegisterClasses();
  void initSPUActions();
  void initVPUActions();

```
- **EN**: Introduces declarations for `VECustomDAG`, `VETargetLowering`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VECustomDAG`, `VETargetLowering` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 122-128
```cpp
public:
  VETargetLowering(const TargetMachine &TM, const VESubtarget &STI);

  MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override {
    return MVT::i32;
  }

```
- **EN**: Implements logic around `VETargetLowering`, `getScalarShiftAmountTy`; this block returns target-specific results.
- **CN**: 围绕 `VETargetLowering`, `getScalarShiftAmountTy` 实现具体逻辑；这一段返回目标相关结果。

### Lines 129-135
```cpp
  Register getRegisterByName(const char *RegName, LLT VT,
                             const MachineFunction &MF) const override;

  /// getSetCCResultType - Return the ISD::SETCC ValueType
  EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
                         EVT VT) const override;

```
- **EN**: Implements logic around `getRegisterByName`, `getSetCCResultType`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getRegisterByName`, `getSetCCResultType` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 136-144
```cpp
  SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
                               bool isVarArg,
                               const SmallVectorImpl<ISD::InputArg> &Ins,
                               const SDLoc &dl, SelectionDAG &DAG,
                               SmallVectorImpl<SDValue> &InVals) const override;

  SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
                    SmallVectorImpl<SDValue> &InVals) const override;

```
- **EN**: Implements logic around `LowerFormalArguments`, `LowerCall`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerFormalArguments`, `LowerCall` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 145-154
```cpp
  bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
                      bool isVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &ArgsFlags,
                      LLVMContext &Context,
                      const Type *RetTy) const override;
  SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
                      const SmallVectorImpl<ISD::OutputArg> &Outs,
                      const SmallVectorImpl<SDValue> &OutVals, const SDLoc &dl,
                      SelectionDAG &DAG) const override;

```
- **EN**: Implements logic around `CanLowerReturn`, `LowerReturn`; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `CanLowerReturn`, `LowerReturn` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 155-168
```cpp
  /// Helper functions for atomic operations.
  bool shouldInsertFencesForAtomic(const Instruction *I) const override {
    // VE uses release consistency, so need fence for each atomics.
    return true;
  }
  Instruction *emitLeadingFence(IRBuilderBase &Builder, Instruction *Inst,
                                AtomicOrdering Ord) const override;
  Instruction *emitTrailingFence(IRBuilderBase &Builder, Instruction *Inst,
                                 AtomicOrdering Ord) const override;
  TargetLoweringBase::AtomicExpansionKind
  shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const override;
  ISD::NodeType getExtendForAtomicOps() const override {
    return ISD::ANY_EXTEND;
  }
```
- **EN**: Implements logic around `shouldInsertFencesForAtomic`, `emitLeadingFence`, `emitTrailingFence`, `shouldExpandAtomicRMWInIR`, ...; this block returns target-specific results.
- **CN**: 围绕 `shouldInsertFencesForAtomic`, `emitLeadingFence`, `emitTrailingFence`, `shouldExpandAtomicRMWInIR`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 169-182
```cpp

  /// Custom Lower {
  TargetLoweringBase::LegalizeAction
  getCustomOperationAction(SDNode &) const override;

  SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  unsigned getJumpTableEncoding() const override;
  const MCExpr *LowerCustomJumpTableEntry(const MachineJumpTableInfo *MJTI,
                                          const MachineBasicBlock *MBB,
                                          unsigned Uid,
                                          MCContext &Ctx) const override;
  SDValue getPICJumpTableRelocBase(SDValue Table,
                                   SelectionDAG &DAG) const override;
  // VE doesn't need getPICJumpTableRelocBaseExpr since it is used for only
```
- **EN**: Implements logic around `getCustomOperationAction`, `LowerOperation`, `getJumpTableEncoding`, `LowerCustomJumpTableEntry`, ...; this block maps fixups or relocations; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `getCustomOperationAction`, `LowerOperation`, `getJumpTableEncoding`, `LowerCustomJumpTableEntry`, ... 实现具体逻辑；这一段映射 fixup 或重定位，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 183-196
```cpp
  // EK_LabelDifference32.

  SDValue lowerATOMIC_FENCE(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerATOMIC_SWAP(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerEH_SJLJ_LONGJMP(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerEH_SJLJ_SETJMP(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerEH_SJLJ_SETUP_DISPATCH(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: Implements logic around `lowerATOMIC_FENCE`, `lowerATOMIC_SWAP`, `lowerBlockAddress`, `lowerConstantPool`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `lowerATOMIC_FENCE`, `lowerATOMIC_SWAP`, `lowerBlockAddress`, `lowerConstantPool`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 197-206
```cpp
  SDValue lowerLOAD(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerSTORE(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerToTLSGeneralDynamicModel(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerVASTART(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerVAARG(SDValue Op, SelectionDAG &DAG) const;

  SDValue lowerBUILD_VECTOR(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerEXTRACT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerINSERT_VECTOR_ELT(SDValue Op, SelectionDAG &DAG) const;
  /// } Custom Lower
```
- **EN**: Implements logic around `lowerLOAD`, `lowerSTORE`, `lowerToTLSGeneralDynamicModel`, `lowerVASTART`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `lowerLOAD`, `lowerSTORE`, `lowerToTLSGeneralDynamicModel`, `lowerVASTART`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 207-213
```cpp

  /// Replace the results of node with an illegal result
  /// type with new values built out of custom code.
  ///
  void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
                          SelectionDAG &DAG) const override;

```
- **EN**: Implements logic around `ReplaceNodeResults`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `ReplaceNodeResults` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 214-224
```cpp
  /// Custom Inserter {
  MachineBasicBlock *
  EmitInstrWithCustomInserter(MachineInstr &MI,
                              MachineBasicBlock *MBB) const override;
  MachineBasicBlock *emitEHSjLjLongJmp(MachineInstr &MI,
                                       MachineBasicBlock *MBB) const;
  MachineBasicBlock *emitEHSjLjSetJmp(MachineInstr &MI,
                                      MachineBasicBlock *MBB) const;
  MachineBasicBlock *emitSjLjDispatchBlock(MachineInstr &MI,
                                           MachineBasicBlock *BB) const;

```
- **EN**: Implements logic around `EmitInstrWithCustomInserter`, `emitEHSjLjLongJmp`, `emitEHSjLjSetJmp`, `emitSjLjDispatchBlock`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `EmitInstrWithCustomInserter`, `emitEHSjLjLongJmp`, `emitEHSjLjSetJmp`, `emitSjLjDispatchBlock` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 225-235
```cpp
  void setupEntryBlockForSjLj(MachineInstr &MI, MachineBasicBlock *MBB,
                              MachineBasicBlock *DispatchBB, int FI,
                              int Offset) const;
  // Setup basic block address.
  Register prepareMBB(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
                      MachineBasicBlock *TargetBB, const DebugLoc &DL) const;
  // Prepare function/variable address.
  Register prepareSymbol(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
                         StringRef Symbol, const DebugLoc &DL, bool IsLocal,
                         bool IsCall) const;
  /// } Custom Inserter
```
- **EN**: Implements logic around `setupEntryBlockForSjLj`, `prepareMBB`, `prepareSymbol`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `setupEntryBlockForSjLj`, `prepareMBB`, `prepareSymbol` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 236-248
```cpp

  /// VVP Lowering {
  SDValue lowerToVVP(SDValue Op, SelectionDAG &DAG) const;
  SDValue lowerVVP_LOAD_STORE(SDValue Op, VECustomDAG &) const;
  SDValue lowerVVP_GATHER_SCATTER(SDValue Op, VECustomDAG &) const;

  SDValue legalizeInternalVectorOp(SDValue Op, SelectionDAG &DAG) const;
  SDValue legalizeInternalLoadStoreOp(SDValue Op, VECustomDAG &CDAG) const;
  SDValue splitVectorOp(SDValue Op, VECustomDAG &CDAG) const;
  SDValue splitPackedLoadStore(SDValue Op, VECustomDAG &CDAG) const;
  SDValue legalizePackedAVL(SDValue Op, VECustomDAG &CDAG) const;
  SDValue splitMaskArithmetic(SDValue Op, SelectionDAG &DAG) const;
  /// } VVPLowering
```
- **EN**: Implements logic around `lowerToVVP`, `lowerVVP_LOAD_STORE`, `lowerVVP_GATHER_SCATTER`, `legalizeInternalVectorOp`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `lowerToVVP`, `lowerVVP_LOAD_STORE`, `lowerVVP_GATHER_SCATTER`, `legalizeInternalVectorOp`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 249-256
```cpp

  /// Custom DAGCombine {
  SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;

  SDValue combineSelect(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue combineSelectCC(SDNode *N, DAGCombinerInfo &DCI) const;
  SDValue combineTRUNCATE(SDNode *N, DAGCombinerInfo &DCI) const;
  /// } Custom DAGCombine
```
- **EN**: Implements logic around `PerformDAGCombine`, `combineSelect`, `combineSelectCC`, `combineTRUNCATE`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `PerformDAGCombine`, `combineSelect`, `combineSelectCC`, `combineTRUNCATE` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 257-266
```cpp

  SDValue withTargetFlags(SDValue Op, unsigned TF, SelectionDAG &DAG) const;
  SDValue makeHiLoPair(SDValue Op, unsigned HiTF, unsigned LoTF,
                       SelectionDAG &DAG) const;
  SDValue makeAddress(SDValue Op, SelectionDAG &DAG) const;

  bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
  bool isFPImmLegal(const APFloat &Imm, EVT VT,
                    bool ForCodeSize) const override;
  /// Returns true if the target allows unaligned memory accesses of the
```
- **EN**: Implements logic around `withTargetFlags`, `makeHiLoPair`, `makeAddress`, `isOffsetFoldingLegal`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `withTargetFlags`, `makeHiLoPair`, `makeAddress`, `isOffsetFoldingLegal`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 267-273
```cpp
  /// specified type.
  bool allowsMisalignedMemoryAccesses(EVT VT, unsigned AS, Align A,
                                      MachineMemOperand::Flags Flags,
                                      unsigned *Fast) const override;

  /// Inline Assembly {

```
- **EN**: Implements logic around `allowsMisalignedMemoryAccesses`.
- **CN**: 围绕 `allowsMisalignedMemoryAccesses` 实现具体逻辑。

### Lines 274-280
```cpp
  ConstraintType getConstraintType(StringRef Constraint) const override;
  std::pair<unsigned, const TargetRegisterClass *>
  getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                               StringRef Constraint, MVT VT) const override;

  /// } Inline Assembly

```
- **EN**: Implements logic around `getConstraintType`, `getRegForInlineAsmConstraint`.
- **CN**: 围绕 `getConstraintType`, `getRegForInlineAsmConstraint` 实现具体逻辑。

### Lines 281-294
```cpp
  /// Target Optimization {

  // Return lower limit for number of blocks in a jump table.
  unsigned getMinimumJumpTableEntries() const override;

  // SX-Aurora VE's s/udiv is 5-9 times slower than multiply.
  bool isIntDivCheap(EVT, AttributeList) const override { return false; }
  // VE doesn't have rem.
  bool hasStandaloneRem(EVT) const override { return false; }
  // VE LDZ instruction returns 64 if the input is zero.
  bool isCheapToSpeculateCtlz(Type *) const override { return true; }
  // VE LDZ instruction is fast.
  bool isCtlzFast() const override { return true; }
  // VE has NND instruction.
```
- **EN**: Implements logic around `getMinimumJumpTableEntries`, `isIntDivCheap`, `hasStandaloneRem`, `isCheapToSpeculateCtlz`, ...; this block returns target-specific results.
- **CN**: 围绕 `getMinimumJumpTableEntries`, `isIntDivCheap`, `hasStandaloneRem`, `isCheapToSpeculateCtlz`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 295-300
```cpp
  bool hasAndNot(SDValue Y) const override;
  /// } Target Optimization
};
} // namespace llvm

#endif // LLVM_LIB_TARGET_VE_VEISELLOWERING_H
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **SelectionDAG lowering / SelectionDAG 降低**:
  - **EN**: Custom lowering/legalization rules for target operations
  - **CN**: 为目标操作定制 lowering 与合法化规则
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VE.h`, `llvm/CodeGen/TargetLowering.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_VE_VEISELLOWERING_H`
