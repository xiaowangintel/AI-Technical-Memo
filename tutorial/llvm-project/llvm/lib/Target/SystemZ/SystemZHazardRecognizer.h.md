# SystemZHazardRecognizer.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZHazardRecognizer.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=-- SystemZHazardRecognizer.h - SystemZ Hazard Recognizer -----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares a hazard recognizer for the SystemZ scheduler.
  10: //
  11: // This class is used by the SystemZ scheduling strategy to maintain
  12: // the state during scheduling, and provide cost functions for
  13: // scheduling candidates. This includes:
  14: //
  15: // * Decoder grouping. A decoder group can maximally hold 3 uops, and
  16: // instructions that always begin a new group should be scheduled when
  17: // the current decoder group is empty.
  18: // * Processor resources usage. It is beneficial to balance the use of
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```cpp
  19: // resources.
  20: //
  21: // A goal is to consider all instructions, also those outside of any
  22: // scheduling region. Such instructions are "advanced" past and include
  23: // single instructions before a scheduling region, branches etc.
  24: //
  25: // A block that has only one predecessor continues scheduling with the state
  26: // of it (which may be updated by emitting branches).
  27: //
  28: // ===---------------------------------------------------------------------===//
  29: 
  30: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZHAZARDRECOGNIZER_H
  31: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZHAZARDRECOGNIZER_H
  32: 
  33: #include "SystemZSubtarget.h"
  34: #include "llvm/ADT/SmallVector.h"
  35: #include "llvm/CodeGen/MachineFunction.h"
  36: #include "llvm/CodeGen/MachineScheduler.h"
```
- **EN**: It imports dependencies such as `SystemZSubtarget.h`, `SmallVector.h`, `MachineFunction.h`, `MachineScheduler.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZSubtarget.h`, `SmallVector.h`, `MachineFunction.h`, `MachineScheduler.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: #include "llvm/CodeGen/ScheduleHazardRecognizer.h"
  38: #include "llvm/MC/MCInstrDesc.h"
  39: #include "llvm/Support/raw_ostream.h"
  40: #include <string>
  41: 
  42: namespace llvm {
  43: 
  44: /// SystemZHazardRecognizer maintains the state for one MBB during scheduling.
  45: class SystemZHazardRecognizer : public ScheduleHazardRecognizer {
  46: 
  47:   const SystemZInstrInfo *TII;
  48:   const TargetSchedModel *SchedModel;
  49: 
  50:   /// Keep track of the number of decoder slots used in the current
  51:   /// decoder group.
  52:   unsigned CurrGroupSize;
  53: 
  54:   /// True if an instruction with four reg operands have been scheduled into
```
- **EN**: It imports dependencies such as `ScheduleHazardRecognizer.h`, `MCInstrDesc.h`, `raw_ostream.h`, `string` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZHazardRecognizer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `ScheduleHazardRecognizer.h`, `MCInstrDesc.h`, `raw_ostream.h`, `string` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZHazardRecognizer` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   /// the current decoder group.
  56:   bool CurrGroupHas4RegOps;
  57: 
  58:   /// The tracking of resources here are quite similar to the common
  59:   /// code use of a critical resource. However, z13 differs in the way
  60:   /// that it has two processor sides which may be interesting to
  61:   /// model in the future (a work in progress).
  62: 
  63:   /// Counters for the number of uops scheduled per processor
  64:   /// resource.
  65:   SmallVector<int, 0> ProcResourceCounters;
  66: 
  67:   /// This is the resource with the greatest queue, which the
  68:   /// scheduler tries to avoid.
  69:   unsigned CriticalResourceIdx;
  70: 
  71:   /// Return the number of decoder slots MI requires.
  72:   inline unsigned getNumDecoderSlots(SUnit *SU) const;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74:   /// Return true if MI fits into current decoder group.
  75:   bool fitsIntoCurrentGroup(SUnit *SU) const;
  76: 
  77:   /// Return true if this instruction has four register operands.
  78:   bool has4RegOps(const MachineInstr *MI) const;
  79: 
  80:   /// Two decoder groups per cycle are formed (for z13), meaning 2x3
  81:   /// instructions. This function returns a number between 0 and 5,
  82:   /// representing the current decoder slot of the current cycle.  If an SU
  83:   /// is passed which will begin a new decoder group, the returned value is
  84:   /// the cycle index of the next group.
  85:   unsigned getCurrCycleIdx(SUnit *SU = nullptr) const;
  86: 
  87:   /// LastFPdOpCycleIdx stores the numbeer returned by getCurrCycleIdx()
  88:   /// when a stalling operation is scheduled (which uses the FPd resource).
  89:   unsigned LastFPdOpCycleIdx;
  90: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   /// A counter of decoder groups scheduled.
  92:   unsigned GrpCount;
  93: 
  94:   unsigned getCurrGroupSize() {return CurrGroupSize;};
  95: 
  96:   /// Start next decoder group.
  97:   void nextGroup();
  98: 
  99:   /// Clear all counters for processor resources.
 100:   void clearProcResCounters();
 101: 
 102:   /// With the goal of alternating processor sides for stalling (FPd)
 103:   /// ops, return true if it seems good to schedule an FPd op next.
 104:   bool isFPdOpPreferred_distance(SUnit *SU) const;
 105: 
 106:   /// Last emitted instruction or nullptr.
 107:   MachineInstr *LastEmittedMI;
 108: 
```
- **EN**: The range implements or declares functions including `getCurrGroupSize`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getCurrGroupSize` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109: public:
 110:   SystemZHazardRecognizer(const SystemZInstrInfo *tii,
 111:                           const TargetSchedModel *SM)
 112:       : TII(tii), SchedModel(SM) {
 113:     Reset();
 114:   }
 115: 
 116:   HazardType getHazardType(SUnit *SU, int Stalls = 0) override;
 117:   void Reset() override;
 118:   void EmitInstruction(SUnit *SU) override;
 119: 
 120:   /// Resolves and cache a resolved scheduling class for an SUnit.
 121:   const MCSchedClassDesc *getSchedClass(SUnit *SU) const {
 122:     if (!SU->SchedClass && SchedModel->hasInstrSchedModel())
 123:       SU->SchedClass = SchedModel->resolveSchedClass(SU->getInstr());
 124:     return SU->SchedClass;
 125:   }
 126: 
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   /// Wrap a non-scheduled instruction in an SU and emit it.
 128:   void emitInstruction(MachineInstr *MI, bool TakenBranch = false);
 129: 
 130:   // Cost functions used by SystemZPostRASchedStrategy while
 131:   // evaluating candidates.
 132: 
 133:   /// Return the cost of decoder grouping for SU. If SU must start a
 134:   /// new decoder group, this is negative if this fits the schedule or
 135:   /// positive if it would mean ending a group prematurely. For normal
 136:   /// instructions this returns 0.
 137:   int groupingCost(SUnit *SU) const;
 138: 
 139:   /// Return the cost of SU in regards to processor resources usage.
 140:   /// A positive value means it would be better to wait with SU, while
 141:   /// a negative value means it would be good to schedule SU next.
 142:   int resourcesCost(SUnit *SU);
 143: 
 144: #ifndef NDEBUG
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-161 / 第 145-161 行
```cpp
 145:   // Debug dumping.
 146:   std::string CurGroupDbg; // current group as text
 147:   void dumpSU(SUnit *SU, raw_ostream &OS) const;
 148:   void dumpCurrGroup(std::string Msg = "") const;
 149:   void dumpProcResourceCounters() const;
 150:   void dumpState() const;
 151: #endif
 152: 
 153:   MachineBasicBlock::iterator getLastEmittedMI() { return LastEmittedMI; }
 154: 
 155:   /// Copy counters from end of single predecessor.
 156:   void copyState(SystemZHazardRecognizer *Incoming);
 157: };
 158: 
 159: } // namespace llvm
 160: 
 161: #endif /* LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZHAZARDRECOGNIZER_H */
```
- **EN**: The range implements or declares functions including `getLastEmittedMI`.
- **CN**: 这一段实现或声明了 `getLastEmittedMI` 等函数。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `SystemZSubtarget.h`
- `llvm/ADT/SmallVector.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineScheduler.h`
- `llvm/CodeGen/ScheduleHazardRecognizer.h`
- `llvm/MC/MCInstrDesc.h`
- `llvm/Support/raw_ostream.h`
- `string`
