# AVRISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRISelLowering.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that AVR uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRISelLowering.h - AVR DAG Lowering Interface ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that AVR uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_AVR_ISEL_LOWERING_H
  15: #define LLVM_AVR_ISEL_LOWERING_H
  16: 
  17: #include "llvm/CodeGen/CallingConvLower.h"
  18: #include "llvm/CodeGen/TargetLowering.h"
  19: 
  20: namespace llvm {
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: 
  22: class AVRSubtarget;
  23: class AVRTargetMachine;
  24: 
  25: /// Performs target lowering for the AVR.
  26: class AVRTargetLowering : public TargetLowering {
  27: public:
  28:   explicit AVRTargetLowering(const AVRTargetMachine &TM,
  29:                              const AVRSubtarget &STI);
  30: 
  31: public:
  32:   MVT getScalarShiftAmountTy(const DataLayout &, EVT LHSTy) const override {
  33:     return MVT::i8;
  34:   }
  35: 
  36:   MVT::SimpleValueType getCmpLibcallReturnType() const override {
  37:     return MVT::i8;
  38:   }
  39: 
  40:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
```

- EN: This chunk introduces interfaces or data structures such as AVRSubtarget, AVRTargetMachine, AVRTargetLowering, which organize the target-specific behavior exposed by the file. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一段引入了 AVRSubtarget, AVRTargetMachine, AVRTargetLowering 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41: 
  42:   void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
  43:                           SelectionDAG &DAG) const override;
  44: 
  45:   bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM, Type *Ty,
  46:                              unsigned AS,
  47:                              Instruction *I = nullptr) const override;
  48: 
  49:   bool getPreIndexedAddressParts(SDNode *N, SDValue &Base, SDValue &Offset,
  50:                                  ISD::MemIndexedMode &AM,
  51:                                  SelectionDAG &DAG) const override;
  52: 
  53:   bool getPostIndexedAddressParts(SDNode *N, SDNode *Op, SDValue &Base,
  54:                                   SDValue &Offset, ISD::MemIndexedMode &AM,
  55:                                   SelectionDAG &DAG) const override;
  56: 
  57:   bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override;
  58: 
  59:   EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
  60:                          EVT VT) const override;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61: 
  62:   MachineBasicBlock *
  63:   EmitInstrWithCustomInserter(MachineInstr &MI,
  64:                               MachineBasicBlock *MBB) const override;
  65: 
  66:   ConstraintType getConstraintType(StringRef Constraint) const override;
  67: 
  68:   ConstraintWeight
  69:   getSingleConstraintMatchWeight(AsmOperandInfo &info,
  70:                                  const char *constraint) const override;
  71: 
  72:   std::pair<unsigned, const TargetRegisterClass *>
  73:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
  74:                                StringRef Constraint, MVT VT) const override;
  75: 
  76:   InlineAsm::ConstraintCode
  77:   getInlineAsmMemConstraint(StringRef ConstraintCode) const override;
  78: 
  79:   void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
  80:                                     std::vector<SDValue> &Ops,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81:                                     SelectionDAG &DAG) const override;
  82: 
  83:   Register getRegisterByName(const char *RegName, LLT VT,
  84:                              const MachineFunction &MF) const override;
  85: 
  86:   bool shouldSplitFunctionArgumentsAsLittleEndian(
  87:       const DataLayout &DL) const override {
  88:     return false;
  89:   }
  90: 
  91:   ShiftLegalizationStrategy
  92:   preferredShiftLegalizationStrategy(SelectionDAG &DAG, SDNode *N,
  93:                                      unsigned ExpansionFactor) const override {
  94:     return ShiftLegalizationStrategy::LowerToLibcall;
  95:   }
  96: 
  97: private:
  98:   SDValue getAVRCmp(SDValue LHS, SDValue RHS, ISD::CondCode CC, SDValue &AVRcc,
  99:                     SelectionDAG &DAG, SDLoc dl) const;
 100:   SDValue getAVRCmp(SDValue LHS, SDValue RHS, SelectionDAG &DAG,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101:                     SDLoc dl) const;
 102:   SDValue LowerShifts(SDValue Op, SelectionDAG &DAG) const;
 103:   SDValue LowerDivRem(SDValue Op, SelectionDAG &DAG) const;
 104:   SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
 105:   SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
 106:   SDValue LowerBR_CC(SDValue Op, SelectionDAG &DAG) const;
 107:   SDValue LowerINLINEASM(SDValue Op, SelectionDAG &DAG) const;
 108:   SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;
 109:   SDValue LowerSETCC(SDValue Op, SelectionDAG &DAG) const;
 110:   SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
 111: 
 112:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
 113:                       bool isVarArg,
 114:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 115:                       LLVMContext &Context, const Type *RetTy) const override;
 116: 
 117:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
 118:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 119:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &dl,
 120:                       SelectionDAG &DAG) const override;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
 122:                                bool isVarArg,
 123:                                const SmallVectorImpl<ISD::InputArg> &Ins,
 124:                                const SDLoc &dl, SelectionDAG &DAG,
 125:                                SmallVectorImpl<SDValue> &InVals) const override;
 126:   SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
 127:                     SmallVectorImpl<SDValue> &InVals) const override;
 128:   SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
 129:                           CallingConv::ID CallConv, bool isVarArg,
 130:                           const SmallVectorImpl<ISD::InputArg> &Ins,
 131:                           const SDLoc &dl, SelectionDAG &DAG,
 132:                           SmallVectorImpl<SDValue> &InVals) const;
 133: 
 134: protected:
 135:   const AVRSubtarget &Subtarget;
 136: 
 137: private:
 138:   MachineBasicBlock *insertShift(MachineInstr &MI, MachineBasicBlock *BB,
 139:                                  bool Tiny) const;
 140:   MachineBasicBlock *insertWideShift(MachineInstr &MI,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-152

```cpp
 141:                                      MachineBasicBlock *BB) const;
 142:   MachineBasicBlock *insertMul(MachineInstr &MI, MachineBasicBlock *BB) const;
 143:   MachineBasicBlock *insertCopyZero(MachineInstr &MI,
 144:                                     MachineBasicBlock *BB) const;
 145:   MachineBasicBlock *insertAtomicArithmeticOp(MachineInstr &MI,
 146:                                               MachineBasicBlock *BB,
 147:                                               unsigned Opcode, int Width) const;
 148: };
 149: 
 150: } // end namespace llvm
 151: 
 152: #endif // LLVM_AVR_ISEL_LOWERING_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

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

- Direct includes / 直接包含: `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/TargetLowering.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `AVRISelLowering.cpp`
