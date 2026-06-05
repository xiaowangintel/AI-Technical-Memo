# ARCISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCISelLowering.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that ARC uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCISelLowering.h - ARC DAG Lowering Interface -----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that ARC uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_ARC_ARCISELLOWERING_H
  15: #define LLVM_LIB_TARGET_ARC_ARCISELLOWERING_H
  16: 
  17: #include "ARC.h"
  18: #include "llvm/CodeGen/SelectionDAG.h"
  19: #include "llvm/CodeGen/TargetLowering.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: namespace llvm {
  22: 
  23: // Forward delcarations
  24: class ARCSubtarget;
  25: class ARCTargetMachine;
  26: 
  27: //===--------------------------------------------------------------------===//
  28: // TargetLowering Implementation
  29: //===--------------------------------------------------------------------===//
  30: class ARCTargetLowering : public TargetLowering {
  31: public:
  32:   explicit ARCTargetLowering(const TargetMachine &TM,
  33:                              const ARCSubtarget &Subtarget);
  34: 
  35:   /// Provide custom lowering hooks for some operations.
  36:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  37: 
  38:   /// Return true if the addressing mode represented by AM is legal for this
  39:   /// target, for a load/store of the specified type.
  40:   bool isLegalAddressingMode(const DataLayout &DL, const AddrMode &AM, Type *Ty,
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as ARCSubtarget, ARCTargetMachine, ARCTargetLowering, which organize the target-specific behavior exposed by the file. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 ARCSubtarget, ARCTargetMachine, ARCTargetLowering 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41:                              unsigned AS,
  42:                              Instruction *I = nullptr) const override;
  43: 
  44: private:
  45:   const ARCSubtarget &Subtarget;
  46: 
  47:   void ReplaceNodeResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
  48:                           SelectionDAG &DAG) const override;
  49: 
  50:   // Lower Operand helpers
  51:   SDValue LowerCallArguments(SDValue Chain, CallingConv::ID CallConv,
  52:                              bool isVarArg,
  53:                              const SmallVectorImpl<ISD::InputArg> &Ins,
  54:                              SDLoc dl, SelectionDAG &DAG,
  55:                              SmallVectorImpl<SDValue> &InVals) const;
  56:   // Lower Operand specifics
  57:   SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
  58:   SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
  59:   SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;
  60:   SDValue LowerBR_CC(SDValue Op, SelectionDAG &DAG) const;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61:   SDValue LowerSIGN_EXTEND_INREG(SDValue Op, SelectionDAG &DAG) const;
  62:   SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
  63:   SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
  64: 
  65:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
  66:                                bool isVarArg,
  67:                                const SmallVectorImpl<ISD::InputArg> &Ins,
  68:                                const SDLoc &dl, SelectionDAG &DAG,
  69:                                SmallVectorImpl<SDValue> &InVals) const override;
  70: 
  71:   SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
  72:                     SmallVectorImpl<SDValue> &InVals) const override;
  73: 
  74:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
  75:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
  76:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &dl,
  77:                       SelectionDAG &DAG) const override;
  78: 
  79:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
  80:                       bool isVarArg,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-89

```cpp
  81:                       const SmallVectorImpl<ISD::OutputArg> &ArgsFlags,
  82:                       LLVMContext &Context, const Type *RetTy) const override;
  83: 
  84:   bool mayBeEmittedAsTailCall(const CallInst *CI) const override;
  85: };
  86: 
  87: } // end namespace llvm
  88: 
  89: #endif // LLVM_LIB_TARGET_ARC_ARCISELLOWERING_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

## Key Concepts / 关键概念

- Custom DAG legalization / 自定义 DAG 合法化
- Calling-lowering hooks / 调用降级钩子
- SelectionDAG lowering / SelectionDAG 降级
- MachineFunction state / MachineFunction 状态
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Calling convention / 调用约定

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARC.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `ARCISelLowering.cpp`
