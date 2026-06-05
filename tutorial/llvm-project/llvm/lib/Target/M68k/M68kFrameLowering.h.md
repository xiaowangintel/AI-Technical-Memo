# M68kFrameLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kFrameLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements stack frame lowering and prologue/epilogue logic for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责实现栈帧降低以及序言/尾声逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kFrameLowering.h - Define frame lowering for M68k ----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains the M68k declaration of TargetFrameLowering class.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_M68K_M68KFRAMELOWERING_H
  15: #define LLVM_LIB_TARGET_M68K_M68KFRAMELOWERING_H
  16: 
  17: #include "M68k.h"
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68k.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68k.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/CodeGen/TargetFrameLowering.h"
  20: 
  21: namespace llvm {
  22: class MachineInstrBuilder;
  23: class MCCFIInstruction;
  24: class M68kSubtarget;
  25: class M68kRegisterInfo;
  26: struct Align;
  27: 
  28: class M68kFrameLowering : public TargetFrameLowering {
  29:   // Cached subtarget predicates.
  30:   const M68kSubtarget &STI;
  31:   const TargetInstrInfo &TII;
  32:   const M68kRegisterInfo *TRI;
  33: 
  34:   /// Stack slot size in bytes.
  35:   unsigned SlotSize;
  36: 
```
- **EN**: It imports dependencies such as `TargetFrameLowering.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `MachineInstrBuilder`, `MCCFIInstruction`, `M68kSubtarget`, `M68kRegisterInfo`, `M68kFrameLowering`.
- **CN**: 它引入了 `TargetFrameLowering.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `MachineInstrBuilder`, `MCCFIInstruction`, `M68kSubtarget`, `M68kRegisterInfo`, `M68kFrameLowering` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   unsigned StackPtr;
  38: 
  39:   /// If we're forcing a stack realignment we can't rely on just the frame
  40:   /// info, we need to know the ABI stack alignment as well in case we have a
  41:   /// call out.  Otherwise just make sure we have some alignment - we'll go
  42:   /// with the minimum SlotSize.
  43:   uint64_t calculateMaxStackAlign(const MachineFunction &MF) const;
  44: 
  45:   /// Adjusts the stack pointer using LEA, SUB, or ADD.
  46:   MachineInstrBuilder BuildStackAdjustment(MachineBasicBlock &MBB,
  47:                                            MachineBasicBlock::iterator MBBI,
  48:                                            const DebugLoc &DL, int64_t Offset,
  49:                                            bool InEpilogue) const;
  50: 
  51:   /// Aligns the stack pointer by ANDing it with -MaxAlign.
  52:   void BuildStackAlignAND(MachineBasicBlock &MBB,
  53:                           MachineBasicBlock::iterator MBBI, const DebugLoc &DL,
  54:                           unsigned Reg, uint64_t MaxAlign) const;
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55: 
  56:   /// Wraps up getting a CFI index and building a MachineInstr for it.
  57:   void BuildCFI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  58:                 const DebugLoc &DL, const MCCFIInstruction &CFIInst) const;
  59: 
  60:   void emitPrologueCalleeSavedFrameMoves(MachineBasicBlock &MBB,
  61:                                          MachineBasicBlock::iterator MBBI,
  62:                                          const DebugLoc &DL) const;
  63: 
  64:   unsigned getPSPSlotOffsetFromSP(const MachineFunction &MF) const;
  65: 
  66: public:
  67:   explicit M68kFrameLowering(const M68kSubtarget &sti, Align Alignment);
  68: 
  69:   static const M68kFrameLowering *create(const M68kSubtarget &ST);
  70: 
  71:   /// This method is called during prolog/epilog code insertion to eliminate
  72:   /// call frame setup and destroy pseudo instructions (but only if the Target
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   /// is using them).  It is responsible for eliminating these instructions,
  74:   /// replacing them with concrete instructions.  This method need only be
  75:   /// implemented if using call frame setup/destroy pseudo instructions.
  76:   /// Returns an iterator pointing to the instruction after the replaced one.
  77:   MachineBasicBlock::iterator
  78:   eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
  79:                                 MachineBasicBlock::iterator MI) const override;
  80: 
  81:   /// Insert prolog code into the function.
  82:   void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  83: 
  84:   /// Insert epilog code into the function.
  85:   void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  86: 
  87:   /// This method determines which of the registers reported by
  88:   /// TargetRegisterInfo::getCalleeSavedRegs() should actually get saved.
  89:   /// The default implementation checks populates the \p SavedRegs bitset with
  90:   /// all registers which are modified in the function, targets may override
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   /// this function to save additional registers.
  92:   /// This method also sets up the register scavenger ensuring there is a free
  93:   /// register or a frameindex available.
  94:   void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
  95:                             RegScavenger *RS = nullptr) const override;
  96: 
  97:   /// Allows target to override spill slot assignment logic.  If implemented,
  98:   /// assignCalleeSavedSpillSlots() should assign frame slots to all CSI
  99:   /// entries and return true.  If this method returns false, spill slots will
 100:   /// be assigned using generic implementation.  assignCalleeSavedSpillSlots()
 101:   /// may add, delete or rearrange elements of CSI.
 102:   bool
 103:   assignCalleeSavedSpillSlots(MachineFunction &MF,
 104:                               const TargetRegisterInfo *TRI,
 105:                               std::vector<CalleeSavedInfo> &CSI) const override;
 106: 
 107:   /// Issues instruction(s) to spill all callee saved registers and returns
 108:   /// true if it isn't possible / profitable to do so by issuing a series of
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   /// store instructions via storeRegToStackSlot(). Returns false otherwise.
 110:   bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
 111:                                  MachineBasicBlock::iterator MI,
 112:                                  ArrayRef<CalleeSavedInfo> CSI,
 113:                                  const TargetRegisterInfo *TRI) const override;
 114: 
 115:   /// Issues instruction(s) to restore all callee saved registers and returns
 116:   /// true if it isn't possible / profitable to do so by issuing a series of
 117:   /// load instructions via loadRegToStackSlot().  Returns false otherwise.
 118:   bool
 119:   restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
 120:                               MachineBasicBlock::iterator MI,
 121:                               MutableArrayRef<CalleeSavedInfo> CSI,
 122:                               const TargetRegisterInfo *TRI) const override;
 123: 
 124:   /// Under normal circumstances, when a frame pointer is not required, we
 125:   /// reserve argument space for call sites in the function immediately on
 126:   /// entry to the current function. This eliminates the need for add/sub sp
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   /// brackets around call sites. Returns true if the call frame is included as
 128:   /// part of the stack frame.
 129:   bool hasReservedCallFrame(const MachineFunction &MF) const override;
 130: 
 131:   /// If there is a reserved call frame, the call frame pseudos can be
 132:   /// simplified.  Having a FP, as in the default implementation, is not
 133:   /// sufficient here since we can't always use it.  Use a more nuanced
 134:   /// condition.
 135:   bool canSimplifyCallFramePseudos(const MachineFunction &MF) const override;
 136: 
 137:   // Do we need to perform FI resolution for this function. Normally, this is
 138:   // required only when the function has any stack objects. However, FI
 139:   // resolution actually has another job, not apparent from the title - it
 140:   // resolves callframe setup/destroy that were not simplified earlier.
 141:   //
 142:   // So, this is required for M68k functions that have push sequences even
 143:   // when there are no stack objects.
 144:   bool needsFrameIndexResolution(const MachineFunction &MF) const override;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145: 
 146:   /// This method should return the base register and offset used to reference
 147:   /// a frame index location. The offset is returned directly, and the base
 148:   /// register is returned via FrameReg.
 149:   StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
 150:                                      Register &FrameReg) const override;
 151: 
 152:   /// Check the instruction before/after the passed instruction. If
 153:   /// it is an ADD/SUB/LEA instruction it is deleted argument and the
 154:   /// stack adjustment is returned as a positive value for ADD/LEA and
 155:   /// a negative for SUB.
 156:   int mergeSPUpdates(MachineBasicBlock &MBB, MachineBasicBlock::iterator &MBBI,
 157:                      bool doMergeWithPrevious) const;
 158: 
 159:   /// Emit a series of instructions to increment / decrement the stack
 160:   /// pointer by a constant value.
 161:   void emitSPUpdate(MachineBasicBlock &MBB, MachineBasicBlock::iterator &MBBI,
 162:                     int64_t NumBytes, bool InEpilogue) const;
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-173 / 第 163-173 行
```cpp
 163: 
 164: protected:
 165:   /// Return true if the specified function should have a dedicated frame
 166:   /// pointer register.  This is true if the function has variable sized
 167:   /// allocas, if it needs dynamic stack realignment, if frame pointer
 168:   /// elimination is disabled, or if the frame address is taken.
 169:   bool hasFPImpl(const MachineFunction &MF) const override;
 170: };
 171: } // namespace llvm
 172: 
 173: #endif // LLVM_LIB_TARGET_M68K_M68KFRAMELOWERING_H
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。

## Dependencies / 依赖关系
- `M68k.h`
- `llvm/CodeGen/TargetFrameLowering.h`
