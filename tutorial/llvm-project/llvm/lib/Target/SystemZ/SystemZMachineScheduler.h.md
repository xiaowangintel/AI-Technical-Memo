# SystemZMachineScheduler.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZMachineScheduler.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //==- SystemZMachineScheduler.h - SystemZ Scheduler Interface ----*- C++ -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: // -------------------------- Pre RA scheduling ----------------------------- //
  10: //
  11: // SystemZPreRASchedStrategy performs latency scheduling in certain types of
  12: // regions where this is beneficial, and also helps copy coalescing and
  13: // comparison elimination.
  14: //
  15: // -------------------------- Post RA scheduling ---------------------------- //
  16: //
  17: // SystemZPostRASchedStrategy is a scheduling strategy which is plugged into
  18: // the MachineScheduler. It has a sorted Available set of SUs and a pickNode()
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```cpp
  19: // implementation that looks to optimize decoder grouping and balance the
  20: // usage of processor resources. Scheduler states are saved for the end
  21: // region of each MBB, so that a successor block can learn from it.
  22: //
  23: //----------------------------------------------------------------------------//
  24: 
  25: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZMACHINESCHEDULER_H
  26: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZMACHINESCHEDULER_H
  27: 
  28: #include "SystemZHazardRecognizer.h"
  29: #include "llvm/CodeGen/MachineScheduler.h"
  30: #include "llvm/CodeGen/ScheduleDAG.h"
  31: #include <set>
  32: 
  33: namespace llvm {
  34: 
  35: /// A MachineSchedStrategy implementation for SystemZ pre RA scheduling.
  36: class SystemZPreRASchedStrategy : public GenericScheduler {
```
- **EN**: It imports dependencies such as `SystemZHazardRecognizer.h`, `MachineScheduler.h`, `ScheduleDAG.h`, `set` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZPreRASchedStrategy`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZHazardRecognizer.h`, `MachineScheduler.h`, `ScheduleDAG.h`, `set` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZPreRASchedStrategy` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   void initializeLatencyReduction();
  38: 
  39:   Register Cmp0SrcReg;
  40:   // Return true if MI defines the Cmp0SrcReg that is used by a scheduled
  41:   // compare with 0. If CCDef is true MI must also have an implicit def of CC.
  42:   bool definesCmp0Src(const MachineInstr *MI, bool CCDef = true) const;
  43: 
  44:   // True if the region has many instructions in def-use sequences and would
  45:   // likely benefit from latency reduction.
  46:   bool HasDataSequences;
  47: 
  48: protected:
  49:   bool tryCandidate(SchedCandidate &Cand, SchedCandidate &TryCand,
  50:                     SchedBoundary *Zone) const override;
  51: 
  52: public:
  53:   SystemZPreRASchedStrategy(const MachineSchedContext *C)
  54:       : GenericScheduler(C) {}
```
- **EN**: The range implements or declares functions including `SystemZPreRASchedStrategy`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPreRASchedStrategy` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55: 
  56:   void initPolicy(MachineBasicBlock::iterator Begin,
  57:                   MachineBasicBlock::iterator End,
  58:                   unsigned NumRegionInstrs) override;
  59:   void initialize(ScheduleDAGMI *dag) override;
  60:   void schedNode(SUnit *SU, bool IsTopNode) override;
  61: };
  62: 
  63: /// A MachineSchedStrategy implementation for SystemZ post RA scheduling.
  64: class SystemZPostRASchedStrategy : public MachineSchedStrategy {
  65: 
  66:   const MachineLoopInfo *MLI;
  67:   const SystemZInstrInfo *TII;
  68: 
  69:   // A SchedModel is needed before any DAG is built while advancing past
  70:   // non-scheduled instructions, so it would not always be possible to call
  71:   // DAG->getSchedClass(SU).
  72:   TargetSchedModel SchedModel;
```
- **EN**: This block declares or refines TableGen records such as `SystemZPostRASchedStrategy`.
- **CN**: 该代码块声明或细化了 `SystemZPostRASchedStrategy` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74:   /// A candidate during instruction evaluation.
  75:   struct Candidate {
  76:     SUnit *SU = nullptr;
  77: 
  78:     /// The decoding cost.
  79:     int GroupingCost = 0;
  80: 
  81:     /// The processor resources cost.
  82:     int ResourcesCost = 0;
  83: 
  84:     Candidate() = default;
  85:     Candidate(SUnit *SU_, SystemZHazardRecognizer &HazardRec);
  86: 
  87:     // Compare two candidates.
  88:     bool operator<(const Candidate &other);
  89: 
  90:     // Check if this node is free of cost ("as good as any").
```
- **EN**: It introduces interface types such as `Candidate`, shaping how other backend components interact with this file. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `Candidate` 等接口类型，定义了其他后端组件与本文件交互的方式。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     bool noCost() const {
  92:       return (GroupingCost <= 0 && !ResourcesCost);
  93:     }
  94: 
  95: #ifndef NDEBUG
  96:     void dumpCosts() {
  97:       if (GroupingCost != 0)
  98:         dbgs() << "  Grouping cost:" << GroupingCost;
  99:       if (ResourcesCost != 0)
 100:         dbgs() << "  Resource cost:" << ResourcesCost;
 101:     }
 102: #endif
 103:   };
 104: 
 105:   // A sorter for the Available set that makes sure that SUs are considered
 106:   // in the best order.
 107:   struct SUSorter {
 108:     bool operator() (SUnit *lhs, SUnit *rhs) const {
```
- **EN**: It introduces interface types such as `SUSorter`, shaping how other backend components interact with this file. The range implements or declares functions including `noCost`, `dumpCosts`, `operator`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SUSorter` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `noCost`, `dumpCosts`, `operator` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:       if (lhs->isScheduleHigh && !rhs->isScheduleHigh)
 110:         return true;
 111:       if (!lhs->isScheduleHigh && rhs->isScheduleHigh)
 112:         return false;
 113: 
 114:       if (lhs->getHeight() > rhs->getHeight())
 115:         return true;
 116:       else if (lhs->getHeight() < rhs->getHeight())
 117:         return false;
 118: 
 119:       return (lhs->NodeNum < rhs->NodeNum);
 120:     }
 121:   };
 122:   // A set of SUs with a sorter and dump method.
 123:   struct SUSet : std::set<SUnit*, SUSorter> {
 124:     #ifndef NDEBUG
 125:     void dump(SystemZHazardRecognizer &HazardRec) const;
 126:     #endif
```
- **EN**: It introduces interface types such as `SUSet`, shaping how other backend components interact with this file. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SUSet` 等接口类型，定义了其他后端组件与本文件交互的方式。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   };
 128: 
 129:   /// The set of available SUs to schedule next.
 130:   SUSet Available;
 131: 
 132:   /// Current MBB
 133:   MachineBasicBlock *MBB;
 134: 
 135:   /// Maintain hazard recognizers for all blocks, so that the scheduler state
 136:   /// can be maintained past BB boundaries when appropariate.
 137:   typedef std::map<MachineBasicBlock*, SystemZHazardRecognizer*> MBB2HazRec;
 138:   MBB2HazRec SchedStates;
 139: 
 140:   /// Pointer to the HazardRecognizer that tracks the scheduler state for
 141:   /// the current region.
 142:   SystemZHazardRecognizer *HazardRec;
 143: 
 144:   /// Update the scheduler state by emitting (non-scheduled) instructions
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   /// up to, but not including, NextBegin.
 146:   void advanceTo(MachineBasicBlock::iterator NextBegin);
 147: 
 148: public:
 149:   SystemZPostRASchedStrategy(const MachineSchedContext *C);
 150:   ~SystemZPostRASchedStrategy() override;
 151: 
 152:   /// Called for a region before scheduling.
 153:   void initPolicy(MachineBasicBlock::iterator Begin,
 154:                   MachineBasicBlock::iterator End,
 155:                   unsigned NumRegionInstrs) override;
 156: 
 157:   /// PostRA scheduling does not track pressure.
 158:   bool shouldTrackPressure() const override { return false; }
 159: 
 160:   // Process scheduling regions top-down so that scheduler states can be
 161:   // transferrred over scheduling boundaries.
 162:   bool doMBBSchedRegionsTopDown() const override { return true; }
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163: 
 164:   void initialize(ScheduleDAGMI *dag) override;
 165: 
 166:   /// Tell the strategy that MBB is about to be processed.
 167:   void enterMBB(MachineBasicBlock *NextMBB) override;
 168: 
 169:   /// Tell the strategy that current MBB is done.
 170:   void leaveMBB() override;
 171: 
 172:   /// Pick the next node to schedule, or return NULL.
 173:   SUnit *pickNode(bool &IsTopNode) override;
 174: 
 175:   /// ScheduleDAGMI has scheduled an instruction - tell HazardRec
 176:   /// about it.
 177:   void schedNode(SUnit *SU, bool IsTopNode) override;
 178: 
 179:   /// SU has had all predecessor dependencies resolved. Put it into
 180:   /// Available.
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-189 / 第 181-189 行
```cpp
 181:   void releaseTopNode(SUnit *SU) override;
 182: 
 183:   /// Currently only scheduling top-down, so this method is empty.
 184:   void releaseBottomNode(SUnit *SU) override {};
 185: };
 186: 
 187: } // end namespace llvm
 188: 
 189: #endif // LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZMACHINESCHEDULER_H
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `SystemZHazardRecognizer.h`
- `llvm/CodeGen/MachineScheduler.h`
- `llvm/CodeGen/ScheduleDAG.h`
- `set`
