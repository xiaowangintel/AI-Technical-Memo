# SystemZFrameLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZFrameLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责实现栈帧降低以及序言/尾声逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZFrameLowering.h - Frame lowering for SystemZ -----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZFRAMELOWERING_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZFRAMELOWERING_H
  11: 
  12: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  13: #include "SystemZInstrBuilder.h"
  14: #include "SystemZMachineFunctionInfo.h"
  15: #include "llvm/ADT/IndexedMap.h"
  16: #include "llvm/CodeGen/TargetFrameLowering.h"
  17: #include "llvm/Support/TypeSize.h"
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCTargetDesc.h`, `SystemZInstrBuilder.h`, `SystemZMachineFunctionInfo.h`, `IndexedMap.h`, `TargetFrameLowering.h`, `TypeSize.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCTargetDesc.h`, `SystemZInstrBuilder.h`, `SystemZMachineFunctionInfo.h`, `IndexedMap.h`, `TargetFrameLowering.h`, `TypeSize.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: namespace llvm {
  20: class SystemZSubtarget;
  21: 
  22: class SystemZFrameLowering : public TargetFrameLowering {
  23: public:
  24:   SystemZFrameLowering(StackDirection D, Align StackAl, int LAO, Align TransAl,
  25:                        bool StackReal, unsigned PointerSize);
  26: 
  27:   static std::unique_ptr<SystemZFrameLowering>
  28:   create(const SystemZSubtarget &STI);
  29: 
  30:   // Override TargetFrameLowering.
  31:   bool allocateScavengingFrameIndexesNearIncomingSP(
  32:     const MachineFunction &MF) const override {
  33:     // SystemZ wants normal register scavenging slots, as close to the stack or
  34:     // frame pointer as possible.
  35:     // The default implementation assumes an x86-like layout, where the frame
  36:     // pointer is at the opposite end of the frame from the stack pointer.
```
- **EN**: This block declares or refines TableGen records such as `SystemZSubtarget`, `SystemZFrameLowering`.
- **CN**: 该代码块声明或细化了 `SystemZSubtarget`, `SystemZFrameLowering` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```cpp
  37:     // This meant that when frame pointer elimination was disabled,
  38:     // the slots ended up being as close as possible to the incoming
  39:     // stack pointer, which is the opposite of what we want on SystemZ.
  40:     return false;
  41:   }
  42: 
  43:   bool hasReservedCallFrame(const MachineFunction &MF) const override;
  44: 
  45:   // Return the offset of the backchain.
  46:   virtual unsigned getBackchainOffset(MachineFunction &MF) const = 0;
  47: 
  48:   // Return the offset of the return address.
  49:   virtual int getReturnAddressOffset(MachineFunction &MF) const = 0;
  50: 
  51:   // Get or create the frame index of where the old frame pointer is stored.
  52:   virtual int getOrCreateFramePointerSaveIndex(MachineFunction &MF) const = 0;
  53: 
  54:   // Return the size of a pointer (in bytes).
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   unsigned getPointerSize() const { return PointerSize; }
  56: 
  57:   // Emit instructions before MBBI (in MBB) to add NumBytes to Reg.
  58:   void emitIncrement(MachineBasicBlock &MBB, MachineBasicBlock::iterator &MBBI,
  59:                      const DebugLoc &DL, Register Reg, int64_t NumBytes,
  60:                      const TargetInstrInfo *TII) const;
  61: 
  62: private:
  63:   unsigned PointerSize;
  64: };
  65: 
  66: class SystemZELFFrameLowering : public SystemZFrameLowering {
  67:   IndexedMap<unsigned> RegSpillOffsets;
  68: 
  69: public:
  70:   SystemZELFFrameLowering(unsigned PointerSize);
  71: 
  72:   // Override TargetFrameLowering.
```
- **EN**: This block declares or refines TableGen records such as `SystemZELFFrameLowering`. The range implements or declares functions including `getPointerSize`.
- **CN**: 该代码块声明或细化了 `SystemZELFFrameLowering` 等 TableGen 记录。 这一段实现或声明了 `getPointerSize` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   bool
  74:   assignCalleeSavedSpillSlots(MachineFunction &MF,
  75:                               const TargetRegisterInfo *TRI,
  76:                               std::vector<CalleeSavedInfo> &CSI) const override;
  77:   void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
  78:                             RegScavenger *RS) const override;
  79:   bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
  80:                                  MachineBasicBlock::iterator MBBI,
  81:                                  ArrayRef<CalleeSavedInfo> CSI,
  82:                                  const TargetRegisterInfo *TRI) const override;
  83:   bool
  84:   restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
  85:                               MachineBasicBlock::iterator MBBII,
  86:                               MutableArrayRef<CalleeSavedInfo> CSI,
  87:                               const TargetRegisterInfo *TRI) const override;
  88:   void processFunctionBeforeFrameFinalized(MachineFunction &MF,
  89:                                            RegScavenger *RS) const override;
  90:   void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  92:   void inlineStackProbe(MachineFunction &MF,
  93:                         MachineBasicBlock &PrologMBB) const override;
  94:   StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
  95:                                      Register &FrameReg) const override;
  96:   void
  97:   orderFrameObjects(const MachineFunction &MF,
  98:                     SmallVectorImpl<int> &ObjectsToAllocate) const override;
  99: 
 100:   // Return the byte offset from the incoming stack pointer of Reg's
 101:   // ABI-defined save slot.  Return 0 if no slot is defined for Reg.  Adjust
 102:   // the offset in case MF has packed-stack.
 103:   unsigned getRegSpillOffset(MachineFunction &MF, Register Reg) const;
 104: 
 105:   bool usePackedStack(MachineFunction &MF) const;
 106: 
 107:   // Return the offset of the backchain.
 108:   unsigned getBackchainOffset(MachineFunction &MF) const override {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     // The back chain is stored topmost with packed-stack.
 110:     return usePackedStack(MF) ? SystemZMC::ELFCallFrameSize - 8 : 0;
 111:   }
 112: 
 113:   // Return the offset of the return address.
 114:   int getReturnAddressOffset(MachineFunction &MF) const override {
 115:     return (usePackedStack(MF) ? -2 : 14) * getPointerSize();
 116:   }
 117: 
 118:   // Get or create the frame index of where the old frame pointer is stored.
 119:   int getOrCreateFramePointerSaveIndex(MachineFunction &MF) const override;
 120: 
 121: protected:
 122:   bool hasFPImpl(const MachineFunction &MF) const override;
 123: };
 124: 
 125: class SystemZXPLINKFrameLowering : public SystemZFrameLowering {
 126:   IndexedMap<unsigned> RegSpillOffsets;
```
- **EN**: This block declares or refines TableGen records such as `SystemZXPLINKFrameLowering`.
- **CN**: 该代码块声明或细化了 `SystemZXPLINKFrameLowering` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```cpp
 127: 
 128: public:
 129:   SystemZXPLINKFrameLowering(unsigned PointerSize);
 130: 
 131:   bool
 132:   assignCalleeSavedSpillSlots(MachineFunction &MF,
 133:                               const TargetRegisterInfo *TRI,
 134:                               std::vector<CalleeSavedInfo> &CSI) const override;
 135: 
 136:   void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
 137:                             RegScavenger *RS) const override;
 138: 
 139:   bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
 140:                                  MachineBasicBlock::iterator MBBI,
 141:                                  ArrayRef<CalleeSavedInfo> CSI,
 142:                                  const TargetRegisterInfo *TRI) const override;
 143: 
 144:   bool
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
 146:                               MachineBasicBlock::iterator MBBII,
 147:                               MutableArrayRef<CalleeSavedInfo> CSI,
 148:                               const TargetRegisterInfo *TRI) const override;
 149: 
 150:   void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
 151: 
 152:   void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
 153: 
 154:   void inlineStackProbe(MachineFunction &MF,
 155:                         MachineBasicBlock &PrologMBB) const override;
 156: 
 157:   void processFunctionBeforeFrameFinalized(MachineFunction &MF,
 158:                                            RegScavenger *RS) const override;
 159: 
 160:   void determineFrameLayout(MachineFunction &MF) const;
 161: 
 162:   // Return the offset of the backchain.
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   unsigned getBackchainOffset(MachineFunction &MF) const override {
 164:     // The back chain is always the first element of the frame.
 165:     return 0;
 166:   }
 167: 
 168:   // Return the offset of the return address.
 169:   int getReturnAddressOffset(MachineFunction &MF) const override {
 170:     return 3 * getPointerSize();
 171:   }
 172: 
 173:   // Get or create the frame index of where the old frame pointer is stored.
 174:   int getOrCreateFramePointerSaveIndex(MachineFunction &MF) const override;
 175: 
 176: protected:
 177:   bool hasFPImpl(const MachineFunction &MF) const override;
 178: };
 179: } // end namespace llvm
 180: 
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-181 / 第 181-181 行
```cpp
 181: #endif
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。

## Dependencies / 依赖关系
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `SystemZInstrBuilder.h`
- `SystemZMachineFunctionInfo.h`
- `llvm/ADT/IndexedMap.h`
- `llvm/CodeGen/TargetFrameLowering.h`
- `llvm/Support/TypeSize.h`
