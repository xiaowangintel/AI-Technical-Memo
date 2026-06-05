# BPFISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFISelLowering.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that BPF uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFISelLowering.h - BPF DAG Lowering Interface ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that BPF uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_BPF_BPFISELLOWERING_H
  15: #define LLVM_LIB_TARGET_BPF_BPFISELLOWERING_H
  16: 
  17: #include "BPF.h"
  18: #include "llvm/CodeGen/SelectionDAG.h"
  19: #include "llvm/CodeGen/TargetLowering.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: namespace llvm {
  22: class BPFSubtarget;
  23: 
  24: class BPFTargetLowering : public TargetLowering {
  25: public:
  26:   explicit BPFTargetLowering(const TargetMachine &TM, const BPFSubtarget &STI);
  27: 
  28:   // Provide custom lowering hooks for some operations.
  29:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  30: 
  31:   // This method decides whether folding a constant offset
  32:   // with the given GlobalAddress is legal.
  33:   bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
  34: 
  35:   bool allowsMisalignedMemoryAccesses(EVT VT, unsigned, Align,
  36:                                       MachineMemOperand::Flags,
  37:                                       unsigned *) const override;
  38: 
  39:   BPFTargetLowering::ConstraintType
  40:   getConstraintType(StringRef Constraint) const override;
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as BPFSubtarget, BPFTargetLowering, which organize the target-specific behavior exposed by the file. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 BPFSubtarget, BPFTargetLowering 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41: 
  42:   std::pair<unsigned, const TargetRegisterClass *>
  43:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
  44:                                StringRef Constraint, MVT VT) const override;
  45: 
  46:   MachineBasicBlock *
  47:   EmitInstrWithCustomInserter(MachineInstr &MI,
  48:                               MachineBasicBlock *BB) const override;
  49: 
  50:   bool getHasAlu32() const { return HasAlu32; }
  51:   bool getHasJmp32() const { return HasJmp32; }
  52:   bool getHasJmpExt() const { return HasJmpExt; }
  53: 
  54:   EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
  55:                          EVT VT) const override;
  56: 
  57:   // Exception handling support.
  58:   Register getExceptionPointerRegister(const Constant *) const override {
  59:     return BPF::R0;
  60:   }
```

- EN: Function bodies or method definitions such as getHasAlu32, getHasJmp32, getHasJmpExt contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getHasAlu32, getHasJmp32, getHasJmpExt 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61:   Register getExceptionSelectorRegister(const Constant *) const override {
  62:     return BPF::R0;
  63:   }
  64: 
  65:   MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override;
  66: 
  67:   unsigned getJumpTableEncoding() const override;
  68: 
  69: private:
  70:   // Control Instruction Selection Features
  71:   bool HasAlu32;
  72:   bool HasJmp32;
  73:   bool HasJmpExt;
  74:   bool HasMovsx;
  75: 
  76:   // Allows Misalignment
  77:   bool AllowsMisalignedMemAccess;
  78: 
  79:   SDValue LowerSDIVSREM(SDValue Op, SelectionDAG &DAG) const;
  80:   SDValue LowerShiftParts(SDValue Op, SelectionDAG &DAG) const;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81:   SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
  82:   SDValue LowerBR_CC(SDValue Op, SelectionDAG &DAG) const;
  83:   SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;
  84:   SDValue LowerATOMIC_LOAD_STORE(SDValue Op, SelectionDAG &DAG) const;
  85:   SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
  86:   SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
  87:   SDValue LowerTRAP(SDValue Op, SelectionDAG &DAG) const;
  88:   SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
  89:   SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
  90: 
  91:   template <class NodeTy>
  92:   SDValue getAddr(NodeTy *N, SelectionDAG &DAG, unsigned Flags = 0) const;
  93: 
  94:   // Lower the result values of a call, copying them out of physregs into vregs
  95:   SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
  96:                           CallingConv::ID CallConv, bool IsVarArg,
  97:                           const SmallVectorImpl<ISD::InputArg> &Ins,
  98:                           const SDLoc &DL, SelectionDAG &DAG,
  99:                           SmallVectorImpl<SDValue> &InVals) const;
 100: 
```

- EN: This chunk introduces interfaces or data structures such as NodeTy, which organize the target-specific behavior exposed by the file. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一段引入了 NodeTy 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101:   // Lower a call into CALLSEQ_START - BPFISD:CALL - CALLSEQ_END chain
 102:   SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
 103:                     SmallVectorImpl<SDValue> &InVals) const override;
 104: 
 105:   // Lower incoming arguments, copy physregs into vregs
 106:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
 107:                                bool IsVarArg,
 108:                                const SmallVectorImpl<ISD::InputArg> &Ins,
 109:                                const SDLoc &DL, SelectionDAG &DAG,
 110:                                SmallVectorImpl<SDValue> &InVals) const override;
 111: 
 112:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 113:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 114:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
 115:                       SelectionDAG &DAG) const override;
 116: 
 117:   void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
 118:                           SelectionDAG &DAG) const override;
 119: 
 120:   EVT getOptimalMemOpType(LLVMContext &Context, const MemOp &Op,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:                           const AttributeList &FuncAttributes) const override {
 122:     return Op.size() >= 8 ? MVT::i64 : MVT::i32;
 123:   }
 124: 
 125:   bool isIntDivCheap(EVT VT, AttributeList Attr) const override {
 126:     return false;
 127:   }
 128: 
 129:   bool shouldConvertConstantLoadToIntImm(const APInt &Imm,
 130:                                          Type *Ty) const override {
 131:     return true;
 132:   }
 133: 
 134:   // Prevent reducing load width during SelectionDag phase.
 135:   // Otherwise, we may transform the following
 136:   //   ctx = ctx + reloc_offset
 137:   //   ... (*(u32 *)ctx) & 0x8000...
 138:   // to
 139:   //   ctx = ctx + reloc_offset
 140:   //   ... (*(u8 *)(ctx + 1)) & 0x80 ...
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-160

```cpp
 141:   // which will be rejected by the verifier.
 142:   bool
 143:   shouldReduceLoadWidth(SDNode *Load, ISD::LoadExtType ExtTy, EVT NewVT,
 144:                         std::optional<unsigned> ByteOffset) const override {
 145:     return false;
 146:   }
 147: 
 148:   bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM,
 149:                              Type *Ty, unsigned AS,
 150:                              Instruction *I = nullptr) const override;
 151: 
 152:   // isTruncateFree - Return true if it's free to truncate a value of
 153:   // type Ty1 to type Ty2. e.g. On BPF at alu32 mode, it's free to truncate
 154:   // a i64 value in register R1 to i32 by referencing its sub-register W1.
 155:   bool isTruncateFree(Type *Ty1, Type *Ty2) const override;
 156:   bool isTruncateFree(EVT VT1, EVT VT2) const override;
 157: 
 158:   // For 32bit ALU result zext to 64bit is free.
 159:   bool isZExtFree(Type *Ty1, Type *Ty2) const override;
 160:   bool isZExtFree(EVT VT1, EVT VT2) const override;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 161-180

```cpp
 161:   bool isZExtFree(SDValue Val, EVT VT2) const override;
 162: 
 163:   unsigned EmitSubregExt(MachineInstr &MI, MachineBasicBlock *BB, unsigned Reg,
 164:                          bool isSigned) const;
 165: 
 166:   MachineBasicBlock * EmitInstrWithCustomInserterMemcpy(MachineInstr &MI,
 167:                                                         MachineBasicBlock *BB)
 168:                                                         const;
 169:   MachineBasicBlock *
 170:   EmitInstrWithCustomInserterLDimm64(MachineInstr &MI,
 171:                                      MachineBasicBlock *BB) const;
 172: 
 173:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
 174:                       bool IsVarArg,
 175:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 176:                       LLVMContext &Context, const Type *RetTy) const override;
 177: };
 178: }
 179: 
 180: #endif
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

## Key Concepts / 关键概念

- Custom DAG legalization / 自定义 DAG 合法化
- Calling-lowering hooks / 调用降级钩子
- SelectionDAG lowering / SelectionDAG 降级
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- IR to target lowering / IR 到目标降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `BPFISelLowering.cpp`
