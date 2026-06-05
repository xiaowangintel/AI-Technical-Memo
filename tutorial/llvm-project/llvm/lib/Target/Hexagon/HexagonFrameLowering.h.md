# HexagonFrameLowering.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonFrameLowering.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): (the default implementation calls hasFP).
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //==- HexagonFrameLowering.h - Define frame lowering for Hexagon -*- C++ -*-==//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONFRAMELOWERING_H
    10: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONFRAMELOWERING_H
    11: 
    12: #include "Hexagon.h"
    13: #include "HexagonBlockRanges.h"
    14: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    15: #include "llvm/ADT/STLExtras.h"
    16: #include "llvm/CodeGen/MachineBasicBlock.h"
    17: #include "llvm/CodeGen/MachineFrameInfo.h"
    18: #include "llvm/CodeGen/TargetFrameLowering.h"
    19: #include <vector>
    20: 
    21: namespace llvm {
    22: 
    23: class BitVector;
    24: class HexagonInstrInfo;
    25: class HexagonRegisterInfo;
```
- EN: It imports headers such as Hexagon.h, HexagonBlockRanges.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/STLExtras.h, ... (8 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as BitVector, HexagonInstrInfo, HexagonRegisterInfo, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonBlockRanges.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/STLExtras.h, ... (8 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 BitVector, HexagonInstrInfo, HexagonRegisterInfo 等类型，用来承载该组件的状态或接口。

### Lines 26-50 / 第 26-50 行

```cpp
    26: class MachineFunction;
    27: class MachineInstr;
    28: class MachineRegisterInfo;
    29: class TargetRegisterClass;
    30: 
    31: class HexagonFrameLowering : public TargetFrameLowering {
    32: public:
    33:   // First register which could possibly hold a variable argument.
    34:   int FirstVarArgSavedReg;
    35:   explicit HexagonFrameLowering()
    36:       : TargetFrameLowering(StackGrowsDown, Align(8), 0, Align(1), true) {}
    37: 
    38:   void
    39:   orderFrameObjects(const MachineFunction &MF,
    40:                     SmallVectorImpl<int> &ObjectsToAllocate) const override;
    41: 
    42:   // All of the prolog/epilog functionality, including saving and restoring
    43:   // callee-saved registers is handled in emitPrologue. This is to have the
    44:   // logic for shrink-wrapping in one place.
    45:   void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const
    46:       override;
    47:   void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const
    48:       override {}
    49: 
    50:   bool enableCalleeSaveSkip(const MachineFunction &MF) const override;
```
- EN: It declares types such as MachineFunction, MachineInstr, MachineRegisterInfo, TargetRegisterClass, ... (5 total), which carry the state or API of this component. It defines declarative TableGen records like MachineFunction, MachineInstr, MachineRegisterInfo, TargetRegisterClass, HexagonFrameLowering; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonFrameLowering, orderFrameObjects, emitPrologue, emitEpilogue, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 MachineFunction, MachineInstr, MachineRegisterInfo, TargetRegisterClass, ... (5 total) 等类型，用来承载该组件的状态或接口。 这里定义了 MachineFunction, MachineInstr, MachineRegisterInfo, TargetRegisterClass, HexagonFrameLowering 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonFrameLowering, orderFrameObjects, emitPrologue, emitEpilogue, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-75 / 第 51-75 行

```cpp
    51: 
    52:   bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
    53:                                  MachineBasicBlock::iterator MI,
    54:                                  ArrayRef<CalleeSavedInfo> CSI,
    55:                                  const TargetRegisterInfo *TRI) const override {
    56:     return true;
    57:   }
    58: 
    59:   bool
    60:   restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
    61:                               MachineBasicBlock::iterator MI,
    62:                               MutableArrayRef<CalleeSavedInfo> CSI,
    63:                               const TargetRegisterInfo *TRI) const override {
    64:     return true;
    65:   }
    66: 
    67:   bool hasReservedCallFrame(const MachineFunction &MF) const override {
    68:     // We always reserve call frame as a part of the initial stack allocation.
    69:     return true;
    70:   }
    71: 
    72:   bool canSimplifyCallFramePseudos(const MachineFunction &MF) const override {
    73:     // Override this function to avoid calling hasFP before CSI is set
    74:     // (the default implementation calls hasFP).
    75:     return true;
```
- EN: It declares or implements routines such as spillCalleeSavedRegisters, restoreCalleeSavedRegisters, hasReservedCallFrame, canSimplifyCallFramePseudos, translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 spillCalleeSavedRegisters, restoreCalleeSavedRegisters, hasReservedCallFrame, canSimplifyCallFramePseudos 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   }
    77: 
    78:   MachineBasicBlock::iterator
    79:   eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
    80:                                 MachineBasicBlock::iterator I) const override;
    81:   void processFunctionBeforeFrameFinalized(MachineFunction &MF,
    82:       RegScavenger *RS = nullptr) const override;
    83:   void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
    84:       RegScavenger *RS) const override;
    85: 
    86:   bool targetHandlesStackFrameRounding() const override {
    87:     return true;
    88:   }
    89: 
    90:   StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
    91:                                      Register &FrameReg) const override;
    92: 
    93:   const SpillSlot *getCalleeSavedSpillSlots(unsigned &NumEntries)
    94:       const override {
    95:     static const SpillSlot Offsets[] = {
    96:       { Hexagon::R17, -4 }, { Hexagon::R16, -8 }, { Hexagon::D8, -8 },
    97:       { Hexagon::R19, -12 }, { Hexagon::R18, -16 }, { Hexagon::D9, -16 },
    98:       { Hexagon::R21, -20 }, { Hexagon::R20, -24 }, { Hexagon::D10, -24 },
    99:       { Hexagon::R23, -28 }, { Hexagon::R22, -32 }, { Hexagon::D11, -32 },
   100:       { Hexagon::R25, -36 }, { Hexagon::R24, -40 }, { Hexagon::D12, -40 },
```
- EN: It declares or implements routines such as eliminateCallFramePseudoInstr, processFunctionBeforeFrameFinalized, determineCalleeSaves, targetHandlesStackFrameRounding, ... (6 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 eliminateCallFramePseudoInstr, processFunctionBeforeFrameFinalized, determineCalleeSaves, targetHandlesStackFrameRounding, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-125 / 第 101-125 行

```cpp
   101:       { Hexagon::R27, -44 }, { Hexagon::R26, -48 }, { Hexagon::D13, -48 }
   102:     };
   103:     NumEntries = std::size(Offsets);
   104:     return Offsets;
   105:   }
   106: 
   107:   bool assignCalleeSavedSpillSlots(MachineFunction &MF,
   108:       const TargetRegisterInfo *TRI, std::vector<CalleeSavedInfo> &CSI)
   109:       const override;
   110: 
   111:   bool needsAligna(const MachineFunction &MF) const;
   112:   const MachineInstr *getAlignaInstr(const MachineFunction &MF) const;
   113: 
   114:   void insertCFIInstructions(MachineFunction &MF) const;
   115: 
   116: protected:
   117:   bool hasFPImpl(const MachineFunction &MF) const override;
   118: 
   119: private:
   120:   using CSIVect = std::vector<CalleeSavedInfo>;
   121: 
   122:   void expandAlloca(MachineInstr *AI, const HexagonInstrInfo &TII,
   123:       Register SP, unsigned CF) const;
   124:   void insertPrologueInBlock(MachineBasicBlock &MBB, bool PrologueStubs) const;
   125:   void insertEpilogueInBlock(MachineBasicBlock &MBB) const;
```
- EN: It declares or implements routines such as std::size, assignCalleeSavedSpillSlots, needsAligna, getAlignaInstr, ... (9 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 std::size, assignCalleeSavedSpillSlots, needsAligna, getAlignaInstr, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 126-150 / 第 126-150 行

```cpp
   126:   void insertAllocframe(MachineBasicBlock &MBB,
   127:       MachineBasicBlock::iterator InsertPt, unsigned NumBytes) const;
   128:   bool insertCSRSpillsInBlock(MachineBasicBlock &MBB, const CSIVect &CSI,
   129:       const HexagonRegisterInfo &HRI, bool &PrologueStubs) const;
   130:   bool insertCSRRestoresInBlock(MachineBasicBlock &MBB, const CSIVect &CSI,
   131:       const HexagonRegisterInfo &HRI) const;
   132:   void updateEntryPaths(MachineFunction &MF, MachineBasicBlock &SaveB) const;
   133:   bool updateExitPaths(MachineBasicBlock &MBB, MachineBasicBlock &RestoreB,
   134:       BitVector &DoneT, BitVector &DoneF, BitVector &Path) const;
   135:   void insertCFIInstructionsAt(MachineBasicBlock &MBB,
   136:       MachineBasicBlock::iterator At) const;
   137: 
   138:   bool expandCopy(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   139:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   140:       SmallVectorImpl<Register> &NewRegs) const;
   141:   bool expandStoreInt(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   142:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   143:       SmallVectorImpl<Register> &NewRegs) const;
   144:   bool expandLoadInt(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   145:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   146:       SmallVectorImpl<Register> &NewRegs) const;
   147:   bool expandStoreVecPred(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   148:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   149:       SmallVectorImpl<Register> &NewRegs) const;
   150:   bool expandLoadVecPred(MachineBasicBlock &B, MachineBasicBlock::iterator It,
```
- EN: It declares or implements routines such as insertAllocframe, insertCSRSpillsInBlock, insertCSRRestoresInBlock, updateEntryPaths, ... (10 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 insertAllocframe, insertCSRSpillsInBlock, insertCSRRestoresInBlock, updateEntryPaths, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 151-175 / 第 151-175 行

```cpp
   151:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   152:       SmallVectorImpl<Register> &NewRegs) const;
   153:   bool expandStoreVec2(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   154:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   155:       SmallVectorImpl<Register> &NewRegs) const;
   156:   bool expandLoadVec2(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   157:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   158:       SmallVectorImpl<Register> &NewRegs) const;
   159:   bool expandStoreVec(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   160:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   161:       SmallVectorImpl<Register> &NewRegs) const;
   162:   bool expandLoadVec(MachineBasicBlock &B, MachineBasicBlock::iterator It,
   163:       MachineRegisterInfo &MRI, const HexagonInstrInfo &HII,
   164:       SmallVectorImpl<Register> &NewRegs) const;
   165:   bool expandSpillMacros(MachineFunction &MF,
   166:       SmallVectorImpl<Register> &NewRegs) const;
   167: 
   168:   Register findPhysReg(MachineFunction &MF, HexagonBlockRanges::IndexRange &FIR,
   169:       HexagonBlockRanges::InstrIndexMap &IndexMap,
   170:       HexagonBlockRanges::RegToRangeMap &DeadMap,
   171:       const TargetRegisterClass *RC) const;
   172:   void optimizeSpillSlots(MachineFunction &MF,
   173:       SmallVectorImpl<Register> &VRegs) const;
   174: 
   175:   void findShrunkPrologEpilog(MachineFunction &MF, MachineBasicBlock *&PrologB,
```
- EN: It declares or implements routines such as expandStoreVec2, expandLoadVec2, expandStoreVec, expandLoadVec, ... (7 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 expandStoreVec2, expandLoadVec2, expandStoreVec, expandLoadVec, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonBlockRanges，说明了它与同级后端组件的连接关系。

### Lines 176-188 / 第 176-188 行

```cpp
   176:       MachineBasicBlock *&EpilogB) const;
   177: 
   178:   void addCalleeSaveRegistersAsImpOperand(MachineInstr *MI, const CSIVect &CSI,
   179:       bool IsDef, bool IsKill) const;
   180:   bool shouldInlineCSR(const MachineFunction &MF, const CSIVect &CSI) const;
   181:   bool useSpillFunction(const MachineFunction &MF, const CSIVect &CSI) const;
   182:   bool useRestoreFunction(const MachineFunction &MF, const CSIVect &CSI) const;
   183:   bool mayOverflowFrameOffset(MachineFunction &MF) const;
   184: };
   185: 
   186: } // end namespace llvm
   187: 
   188: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONFRAMELOWERING_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as addCalleeSaveRegistersAsImpOperand, shouldInlineCSR, useSpillFunction, useRestoreFunction, ... (5 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 addCalleeSaveRegistersAsImpOperand, shouldInlineCSR, useSpillFunction, useRestoreFunction, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonBlockRanges.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/STLExtras.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/TargetFrameLowering.h, vector`
- Hexagon symbols / Hexagon 符号: `HexagonFrameLowering, HexagonBlockRanges, HexagonMCTargetDesc, HexagonInstrInfo, HexagonRegisterInfo`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
