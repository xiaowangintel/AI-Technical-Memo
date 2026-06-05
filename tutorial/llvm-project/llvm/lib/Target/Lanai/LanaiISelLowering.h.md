# LanaiISelLowering.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiISelLowering.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that Lanai uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiISelLowering.h - Lanai DAG Lowering Interface -....-*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that Lanai uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_LANAI_LANAIISELLOWERING_H
  15: #define LLVM_LIB_TARGET_LANAI_LANAIISELLOWERING_H
  16: 
  17: #include "Lanai.h"
  18: #include "LanaiRegisterInfo.h"
  19: #include "llvm/CodeGen/SelectionDAG.h"
  20: #include "llvm/CodeGen/TargetLowering.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: 
  22: namespace llvm {
  23: 
  24: class LanaiSubtarget;
  25: 
  26: class LanaiTargetLowering : public TargetLowering {
  27: public:
  28:   LanaiTargetLowering(const TargetMachine &TM, const LanaiSubtarget &STI);
  29: 
  30:   // LowerOperation - Provide custom lowering hooks for some operations.
  31:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  32: 
  33:   SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
  34:   SDValue LowerBR_CC(SDValue Op, SelectionDAG &DAG) const;
  35:   SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
  36:   SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
  37:   SDValue LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const;
  38:   SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
  39:   SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
  40:   SDValue LowerMUL(SDValue Op, SelectionDAG &DAG) const;
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as LanaiSubtarget, LanaiTargetLowering, which organize the target-specific behavior exposed by the file. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 LanaiSubtarget, LanaiTargetLowering 等接口或数据结构，用于组织该文件暴露的目标专用行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41:   SDValue LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const;
  42:   SDValue LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const;
  43:   SDValue LowerSETCC(SDValue Op, SelectionDAG &DAG) const;
  44:   SDValue LowerSHL_PARTS(SDValue Op, SelectionDAG &DAG) const;
  45:   SDValue LowerSRL_PARTS(SDValue Op, SelectionDAG &DAG) const;
  46:   SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
  47: 
  48:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
  49:                       bool IsVarArg,
  50:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
  51:                       LLVMContext &Context, const Type *RetTy) const override;
  52: 
  53:   Register getRegisterByName(const char *RegName, LLT VT,
  54:                              const MachineFunction &MF) const override;
  55:   std::pair<unsigned, const TargetRegisterClass *>
  56:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
  57:                                StringRef Constraint, MVT VT) const override;
  58:   ConstraintWeight
  59:   getSingleConstraintMatchWeight(AsmOperandInfo &Info,
  60:                                  const char *Constraint) const override;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61:   void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
  62:                                     std::vector<SDValue> &Ops,
  63:                                     SelectionDAG &DAG) const override;
  64: 
  65:   SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
  66: 
  67:   void computeKnownBitsForTargetNode(const SDValue Op, KnownBits &Known,
  68:                                      const APInt &DemandedElts,
  69:                                      const SelectionDAG &DAG,
  70:                                      unsigned Depth = 0) const override;
  71: 
  72: private:
  73:   SDValue LowerCCCCallTo(SDValue Chain, SDValue Callee,
  74:                          CallingConv::ID CallConv, bool IsVarArg,
  75:                          bool IsTailCall,
  76:                          const SmallVectorImpl<ISD::OutputArg> &Outs,
  77:                          const SmallVectorImpl<SDValue> &OutVals,
  78:                          const SmallVectorImpl<ISD::InputArg> &Ins,
  79:                          const SDLoc &dl, SelectionDAG &DAG,
  80:                          SmallVectorImpl<SDValue> &InVals) const;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81: 
  82:   SDValue LowerCCCArguments(SDValue Chain, CallingConv::ID CallConv,
  83:                             bool IsVarArg,
  84:                             const SmallVectorImpl<ISD::InputArg> &Ins,
  85:                             const SDLoc &DL, SelectionDAG &DAG,
  86:                             SmallVectorImpl<SDValue> &InVals) const;
  87: 
  88:   SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
  89:                           CallingConv::ID CallConv, bool IsVarArg,
  90:                           const SmallVectorImpl<ISD::InputArg> &Ins,
  91:                           const SDLoc &DL, SelectionDAG &DAG,
  92:                           SmallVectorImpl<SDValue> &InVals) const;
  93: 
  94:   SDValue LowerCall(TargetLowering::CallLoweringInfo &CLI,
  95:                     SmallVectorImpl<SDValue> &InVals) const override;
  96: 
  97:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CallConv,
  98:                                bool IsVarArg,
  99:                                const SmallVectorImpl<ISD::InputArg> &Ins,
 100:                                const SDLoc &DL, SelectionDAG &DAG,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-112

```cpp
 101:                                SmallVectorImpl<SDValue> &InVals) const override;
 102: 
 103:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 104:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 105:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
 106:                       SelectionDAG &DAG) const override;
 107: 
 108:   const LanaiRegisterInfo *TRI;
 109: };
 110: } // namespace llvm
 111: 
 112: #endif // LLVM_LIB_TARGET_LANAI_LANAIISELLOWERING_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

## Key Concepts / 关键概念

- Custom DAG legalization / 自定义 DAG 合法化
- Calling-lowering hooks / 调用降级钩子
- SelectionDAG lowering / SelectionDAG 降级
- MachineFunction state / MachineFunction 状态
- Register classes / 寄存器类
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Lanai.h`, `LanaiRegisterInfo.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `LanaiISelLowering.cpp`
