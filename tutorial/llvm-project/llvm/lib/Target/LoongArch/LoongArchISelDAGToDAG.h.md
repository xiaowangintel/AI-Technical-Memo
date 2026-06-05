# LoongArchISelDAGToDAG.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchISelDAGToDAG.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file selects target instructions from SelectionDAG nodes for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责从 SelectionDAG 节点选择目标指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //=- LoongArchISelDAGToDAG.h - A dag to dag inst selector for LoongArch ---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines an instruction selector for the LoongArch target.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #ifndef LLVM_LIB_TARGET_LOONGARCH_LOONGARCHISELDAGTODAG_H
  14: #define LLVM_LIB_TARGET_LOONGARCH_LOONGARCHISELDAGTODAG_H
  15: 
  16: #include "LoongArch.h"
  17: #include "LoongArchSelectionDAGInfo.h"
  18: #include "LoongArchTargetMachine.h"
  19: #include "llvm/CodeGen/SelectionDAGISel.h"
  20: 
  21: // LoongArch-specific code to select LoongArch machine instructions for
  22: // SelectionDAG operations.
  23: namespace llvm {
  24: class LoongArchDAGToDAGISel : public SelectionDAGISel {
```
- **EN**: It imports dependencies such as `LoongArch.h`, `LoongArchSelectionDAGInfo.h`, `LoongArchTargetMachine.h`, `SelectionDAGISel.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `LoongArchDAGToDAGISel`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArch.h`, `LoongArchSelectionDAGInfo.h`, `LoongArchTargetMachine.h`, `SelectionDAGISel.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `LoongArchDAGToDAGISel` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 25-36 / 第 25-36 行
```cpp
  25:   const LoongArchSubtarget *Subtarget = nullptr;
  26: 
  27: public:
  28:   LoongArchDAGToDAGISel() = delete;
  29: 
  30:   explicit LoongArchDAGToDAGISel(LoongArchTargetMachine &TM,
  31:                                  CodeGenOptLevel OptLevel)
  32:       : SelectionDAGISel(TM, OptLevel) {}
  33: 
  34:   bool runOnMachineFunction(MachineFunction &MF) override {
  35:     Subtarget = &MF.getSubtarget<LoongArchSubtarget>();
  36:     return SelectionDAGISel::runOnMachineFunction(MF);
```
- **EN**: The range implements or declares functions including `LoongArchDAGToDAGISel`.
- **CN**: 这一段实现或声明了 `LoongArchDAGToDAGISel` 等函数。

### Lines 37-48 / 第 37-48 行
```cpp
  37:   }
  38: 
  39:   void Select(SDNode *Node) override;
  40: 
  41:   bool SelectInlineAsmMemoryOperand(const SDValue &Op,
  42:                                     InlineAsm::ConstraintCode ConstraintID,
  43:                                     std::vector<SDValue> &OutOps) override;
  44: 
  45:   bool SelectBaseAddr(SDValue Addr, SDValue &Base);
  46:   bool SelectAddrConstant(SDValue Addr, SDValue &Base, SDValue &Offset);
  47:   bool selectNonFIBaseAddr(SDValue Addr, SDValue &Base);
  48:   bool SelectAddrRegImm12(SDValue Addr, SDValue &Base, SDValue &Offset);
```
- **EN**: This span continues the file's main responsibility: this file selects target instructions from SelectionDAG nodes for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 49-60 / 第 49-60 行
```cpp
  49: 
  50:   bool selectShiftMask(SDValue N, unsigned ShiftWidth, SDValue &ShAmt);
  51:   bool selectShiftMaskGRLen(SDValue N, SDValue &ShAmt) {
  52:     return selectShiftMask(N, Subtarget->getGRLen(), ShAmt);
  53:   }
  54:   bool selectShiftMask32(SDValue N, SDValue &ShAmt) {
  55:     return selectShiftMask(N, 32, ShAmt);
  56:   }
  57: 
  58:   bool selectSExti32(SDValue N, SDValue &Val);
  59:   bool selectZExti32(SDValue N, SDValue &Val);
  60: 
```
- **EN**: The range implements or declares functions including `selectShiftMaskGRLen`, `selectShiftMask32`.
- **CN**: 这一段实现或声明了 `selectShiftMaskGRLen`, `selectShiftMask32` 等函数。

### Lines 61-72 / 第 61-72 行
```cpp
  61:   bool selectVSplat(SDNode *N, APInt &Imm, unsigned MinSizeInBits) const;
  62: 
  63:   template <unsigned ImmSize, unsigned EltSize = 0, bool IsSigned = false>
  64:   bool selectVSplatImm(SDValue N, SDValue &SplatVal);
  65:   template <unsigned ImmSize>
  66:   bool selectVSplatImmNeg(SDValue N, SDValue &SplatVal) const;
  67:   template <unsigned EltSize = 0>
  68:   bool selectVSplatUimmInvPow2(SDValue N, SDValue &SplatImm) const;
  69:   template <unsigned EltSize = 0>
  70:   bool selectVSplatUimmPow2(SDValue N, SDValue &SplatImm) const;
  71: 
  72:   // Return the LoongArch branch opcode that matches the given DAG integer
```
- **EN**: This span continues the file's main responsibility: this file selects target instructions from SelectionDAG nodes for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-84 / 第 73-84 行
```cpp
  73:   // condition code. The CondCode must be one of those supported by the
  74:   // LoongArch ISA (see translateSetCCForBranch).
  75:   static unsigned getBranchOpcForIntCC(ISD::CondCode CC) {
  76:     switch (CC) {
  77:     default:
  78:       llvm_unreachable("Unsupported CondCode");
  79:     case ISD::SETEQ:
  80:       return LoongArch::BEQ;
  81:     case ISD::SETNE:
  82:       return LoongArch::BNE;
  83:     case ISD::SETLT:
  84:       return LoongArch::BLT;
```
- **EN**: The range implements or declares functions including `getBranchOpcForIntCC`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getBranchOpcForIntCC` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 85-96 / 第 85-96 行
```cpp
  85:     case ISD::SETGE:
  86:       return LoongArch::BGE;
  87:     case ISD::SETULT:
  88:       return LoongArch::BLTU;
  89:     case ISD::SETUGE:
  90:       return LoongArch::BGEU;
  91:     }
  92:   }
  93: 
  94: // Include the pieces autogenerated from the target description.
  95: #include "LoongArchGenDAGISel.inc"
  96: };
```
- **EN**: It imports dependencies such as `LoongArchGenDAGISel.inc` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `LoongArchGenDAGISel.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 97-107 / 第 97-107 行
```cpp
  97: 
  98: class LoongArchDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
  99: public:
 100:   static char ID;
 101:   explicit LoongArchDAGToDAGISelLegacy(LoongArchTargetMachine &TM,
 102:                                        CodeGenOptLevel OptLevel);
 103: };
 104: 
 105: } // end namespace llvm
 106: 
 107: #endif // LLVM_LIB_TARGET_LOONGARCH_LOONGARCHISELDAGTODAG_H
```
- **EN**: This block declares or refines TableGen records such as `LoongArchDAGToDAGISelLegacy`.
- **CN**: 该代码块声明或细化了 `LoongArchDAGToDAGISelLegacy` 等 TableGen 记录。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `LoongArch.h`
- `LoongArchSelectionDAGInfo.h`
- `LoongArchTargetMachine.h`
- `llvm/CodeGen/SelectionDAGISel.h`
- `LoongArchGenDAGISel.inc`
