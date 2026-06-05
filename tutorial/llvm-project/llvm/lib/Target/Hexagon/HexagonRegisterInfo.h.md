# HexagonRegisterInfo.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonRegisterInfo.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file contains the Hexagon implementation of the TargetRegisterInfo
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。 重点涉及寄存器模型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //==- HexagonRegisterInfo.h - Hexagon Register Information Impl --*- C++ -*-==//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains the Hexagon implementation of the TargetRegisterInfo
    10: // class.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONREGISTERINFO_H
    15: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONREGISTERINFO_H
    16: 
    17: #include "llvm/CodeGen/TargetRegisterInfo.h"
    18: 
    19: #define GET_REGINFO_HEADER
    20: #include "HexagonGenRegisterInfo.inc"
    21: 
    22: namespace llvm {
    23: 
    24: namespace Hexagon {
    25:   // Generic (pseudo) subreg indices for use with getHexagonSubRegIndex.
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as llvm/CodeGen/TargetRegisterInfo.h, HexagonGenRegisterInfo.inc, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, Hexagon) to keep the implementation scoped to LLVM/Hexagon components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 llvm/CodeGen/TargetRegisterInfo.h, HexagonGenRegisterInfo.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。

### Lines 26-50 / 第 26-50 行

```cpp
    26:   enum { ps_sub_lo = 0, ps_sub_hi = 1 };
    27: }
    28: 
    29: class HexagonRegisterInfo : public HexagonGenRegisterInfo {
    30: public:
    31:   HexagonRegisterInfo(unsigned HwMode);
    32: 
    33:   /// Code Generation virtual methods...
    34:   const MCPhysReg *getCalleeSavedRegs(const MachineFunction *MF)
    35:         const override;
    36:   const uint32_t *getCallPreservedMask(const MachineFunction &MF,
    37:         CallingConv::ID) const override;
    38: 
    39:   BitVector getReservedRegs(const MachineFunction &MF) const override;
    40: 
    41:   bool eliminateFrameIndex(MachineBasicBlock::iterator II, int SPAdj,
    42:         unsigned FIOperandNum, RegScavenger *RS = nullptr) const override;
    43: 
    44:   /// Returns true since we may need scavenging for a temporary register
    45:   /// when generating hardware loop instructions.
    46:   bool requiresRegisterScavenging(const MachineFunction &MF) const override {
    47:     return true;
    48:   }
    49: 
    50:   /// Returns true. Spill code for predicate registers might need an extra
```
- EN: It declares types such as HexagonRegisterInfo, which carry the state or API of this component. It defines declarative TableGen records like HexagonRegisterInfo; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonRegisterInfo, getCalleeSavedRegs, getCallPreservedMask, getReservedRegs, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 HexagonRegisterInfo 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonRegisterInfo 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonRegisterInfo, getCalleeSavedRegs, getCallPreservedMask, getReservedRegs, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-75 / 第 51-75 行

```cpp
    51:   /// register.
    52:   bool requiresFrameIndexScavenging(const MachineFunction &MF) const override {
    53:     return true;
    54:   }
    55: 
    56:   /// Returns true if the frame pointer is valid.
    57:   bool useFPForScavengingIndex(const MachineFunction &MF) const override;
    58: 
    59:   bool shouldCoalesce(MachineInstr *MI, const TargetRegisterClass *SrcRC,
    60:         unsigned SubReg, const TargetRegisterClass *DstRC, unsigned DstSubReg,
    61:         const TargetRegisterClass *NewRC, LiveIntervals &LIS) const override;
    62: 
    63:   // Debug information queries.
    64:   Register getFrameRegister(const MachineFunction &MF) const override;
    65:   Register getFrameRegister() const;
    66:   Register getStackRegister() const;
    67: 
    68:   unsigned getHexagonSubRegIndex(const TargetRegisterClass &RC,
    69:         unsigned GenIdx) const;
    70: 
    71:   const MCPhysReg *getCallerSavedRegs(const MachineFunction *MF,
    72:         const TargetRegisterClass *RC) const;
    73: 
    74:   const TargetRegisterClass *
    75:   getPointerRegClass(unsigned Kind = 0) const override;
```
- EN: It declares or implements routines such as requiresFrameIndexScavenging, useFPForScavengingIndex, shouldCoalesce, getFrameRegister, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 requiresFrameIndexScavenging, useFPForScavengingIndex, shouldCoalesce, getFrameRegister, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 76-92 / 第 76-92 行

```cpp
    76: 
    77:   /// Returns true if the given reserved physical register is live across
    78:   /// function calls/returns.
    79:   bool isGlobalReg(MCPhysReg Reg) const;
    80: 
    81:   /// Returns true if the given reserved physical register Reg is live
    82:   /// across function calls/returns. This function should not be used for
    83:   /// non-reserved registers, instead register liveness information should be
    84:   /// checked
    85:   bool isFakeReg(MCPhysReg Reg) const;
    86: 
    87:   bool isEHReturnCalleeSaveReg(Register Reg) const;
    88: };
    89: 
    90: } // end namespace llvm
    91: 
    92: #endif
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as isGlobalReg, isFakeReg, isEHReturnCalleeSaveReg, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 isGlobalReg, isFakeReg, isEHReturnCalleeSaveReg 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- ABI and stack-frame lowering / ABI 与栈帧下沉
- calling convention encoding / 调用约定编码

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/CodeGen/TargetRegisterInfo.h, HexagonGenRegisterInfo.inc`
- Hexagon symbols / Hexagon 符号: `HexagonRegisterInfo, HexagonGenRegisterInfo`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
