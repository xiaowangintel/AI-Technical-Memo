# CSKYISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYISelLowering.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that CSKY uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYISelLowering.cpp - CSKY DAG Lowering Implementation  ----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that CSKY uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_CSKY_CSKYISELLOWERING_H
  15: #define LLVM_LIB_TARGET_CSKY_CSKYISELLOWERING_H
  16: 
  17: #include "CSKYSelectionDAGInfo.h"
  18: #include "MCTargetDesc/CSKYBaseInfo.h"
  19: #include "llvm/CodeGen/CallingConvLower.h"
  20: #include "llvm/CodeGen/TargetLowering.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: 
  22: namespace llvm {
  23: class CSKYSubtarget;
  24: 
  25: class CSKYTargetLowering : public TargetLowering {
  26:   const CSKYSubtarget &Subtarget;
  27: 
  28: public:
  29:   explicit CSKYTargetLowering(const TargetMachine &TM,
  30:                               const CSKYSubtarget &STI);
  31: 
  32:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  33: 
  34:   EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
  35:                          EVT VT) const override;
  36: 
  37: private:
  38:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
  39:                                bool IsVarArg,
  40:                                const SmallVectorImpl<ISD::InputArg> &Ins,
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as CSKYSubtarget, CSKYTargetLowering, which organize the target-specific behavior exposed by the file. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 CSKYSubtarget, CSKYTargetLowering 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41:                                const SDLoc &DL, SelectionDAG &DAG,
  42:                                SmallVectorImpl<SDValue> &InVals) const override;
  43: 
  44:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
  45:                       bool IsVarArg,
  46:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
  47:                       LLVMContext &Context, const Type *RetTy) const override;
  48: 
  49:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
  50:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
  51:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
  52:                       SelectionDAG &DAG) const override;
  53: 
  54:   SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
  55:                     SmallVectorImpl<SDValue> &InVals) const override;
  56: 
  57:   /// If a physical register, this returns the register that receives the
  58:   /// exception address on entry to an EH pad.
  59:   Register
  60:   getExceptionPointerRegister(const Constant *PersonalityFn) const override;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61: 
  62:   /// If a physical register, this returns the register that receives the
  63:   /// exception typeid on entry to a landing pad.
  64:   Register
  65:   getExceptionSelectorRegister(const Constant *PersonalityFn) const override;
  66: 
  67:   bool isSelectSupported(SelectSupportKind Kind) const override {
  68:     // CSKY does not support scalar condition selects on vectors.
  69:     return (Kind != ScalarCondVectorVal);
  70:   }
  71: 
  72:   ConstraintType getConstraintType(StringRef Constraint) const override;
  73: 
  74:   std::pair<unsigned, const TargetRegisterClass *>
  75:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
  76:                                StringRef Constraint, MVT VT) const override;
  77: 
  78:   MachineBasicBlock *
  79:   EmitInstrWithCustomInserter(MachineInstr &MI,
  80:                               MachineBasicBlock *BB) const override;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81: 
  82:   SDValue getTargetNode(GlobalAddressSDNode *N, SDLoc DL, EVT Ty,
  83:                         SelectionDAG &DAG, unsigned Flags) const;
  84: 
  85:   SDValue getTargetNode(ExternalSymbolSDNode *N, SDLoc DL, EVT Ty,
  86:                         SelectionDAG &DAG, unsigned Flags) const;
  87: 
  88:   SDValue getTargetNode(JumpTableSDNode *N, SDLoc DL, EVT Ty, SelectionDAG &DAG,
  89:                         unsigned Flags) const;
  90: 
  91:   SDValue getTargetNode(BlockAddressSDNode *N, SDLoc DL, EVT Ty,
  92:                         SelectionDAG &DAG, unsigned Flags) const;
  93: 
  94:   SDValue getTargetNode(ConstantPoolSDNode *N, SDLoc DL, EVT Ty,
  95:                         SelectionDAG &DAG, unsigned Flags) const;
  96: 
  97:   SDValue getTargetConstantPoolValue(GlobalAddressSDNode *N, EVT Ty,
  98:                                      SelectionDAG &DAG, unsigned Flags) const;
  99: 
 100:   SDValue getTargetConstantPoolValue(ExternalSymbolSDNode *N, EVT Ty,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101:                                      SelectionDAG &DAG, unsigned Flags) const;
 102: 
 103:   SDValue getTargetConstantPoolValue(JumpTableSDNode *N, EVT Ty,
 104:                                      SelectionDAG &DAG, unsigned Flags) const;
 105: 
 106:   SDValue getTargetConstantPoolValue(BlockAddressSDNode *N, EVT Ty,
 107:                                      SelectionDAG &DAG, unsigned Flags) const;
 108: 
 109:   SDValue getTargetConstantPoolValue(ConstantPoolSDNode *N, EVT Ty,
 110:                                      SelectionDAG &DAG, unsigned Flags) const;
 111: 
 112:   template <class NodeTy, bool IsCall = false>
 113:   SDValue getAddr(NodeTy *N, SelectionDAG &DAG, bool IsLocal = true) const {
 114:     SDLoc DL(N);
 115:     EVT Ty = getPointerTy(DAG.getDataLayout());
 116: 
 117:     unsigned Flag = CSKYII::MO_None;
 118:     bool IsPIC = isPositionIndependent();
 119: 
 120:     if (IsPIC)
```

- EN: This chunk introduces interfaces or data structures such as NodeTy, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as getAddr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一段引入了 NodeTy 等接口或数据结构，用于组织该文件暴露的目标专用行为。 getAddr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:       Flag = IsLocal  ? CSKYII::MO_GOTOFF
 122:              : IsCall ? CSKYII::MO_PLT32
 123:                       : CSKYII::MO_GOT32;
 124: 
 125:     SDValue TCPV = getTargetConstantPoolValue(N, Ty, DAG, Flag);
 126:     SDValue TV = getTargetNode(N, DL, Ty, DAG, Flag);
 127:     SDValue Addr = DAG.getNode(CSKYISD::LOAD_ADDR, DL, Ty, {TV, TCPV});
 128: 
 129:     if (!IsPIC)
 130:       return Addr;
 131: 
 132:     SDValue Result =
 133:         DAG.getNode(ISD::ADD, DL, Ty, {DAG.getGLOBAL_OFFSET_TABLE(Ty), Addr});
 134:     if (IsLocal)
 135:       return Result;
 136: 
 137:     return DAG.getLoad(Ty, DL, DAG.getEntryNode(), Result,
 138:                        MachinePointerInfo::getGOT(DAG.getMachineFunction()));
 139:   }
 140: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-160

```cpp
 141:   SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
 142:   SDValue LowerExternalSymbol(SDValue Op, SelectionDAG &DAG) const;
 143:   SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
 144:   SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
 145:   SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
 146:   SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
 147:   SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
 148:   SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
 149:   SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
 150: 
 151:   SDValue getStaticTLSAddr(GlobalAddressSDNode *N, SelectionDAG &DAG,
 152:                            bool UseGOT) const;
 153:   SDValue getDynamicTLSAddr(GlobalAddressSDNode *N, SelectionDAG &DAG) const;
 154: 
 155:   CCAssignFn *CCAssignFnForCall(CallingConv::ID CC, bool IsVarArg) const;
 156:   CCAssignFn *CCAssignFnForReturn(CallingConv::ID CC, bool IsVarArg) const;
 157: 
 158:   bool decomposeMulByConstant(LLVMContext &Context, EVT VT,
 159:                               SDValue C) const override;
 160:   bool isCheapToSpeculateCttz(Type *Ty) const override;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 161-166

```cpp
 161:   bool isCheapToSpeculateCtlz(Type *Ty) const override;
 162: };
 163: 
 164: } // namespace llvm
 165: 
 166: #endif // LLVM_LIB_TARGET_CSKY_CSKYISELLOWERING_H
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

- Direct includes / 直接包含: `CSKYSelectionDAGInfo.h`, `MCTargetDesc/CSKYBaseInfo.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/TargetLowering.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `CSKYISelLowering.cpp`
