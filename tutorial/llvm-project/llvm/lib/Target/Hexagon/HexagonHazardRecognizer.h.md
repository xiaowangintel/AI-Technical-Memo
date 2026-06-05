# HexagonHazardRecognizer.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonHazardRecognizer.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file defines the hazard recognizer for scheduling on Hexagon.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===--- HexagonHazardRecognizer.h - Hexagon Post RA Hazard Recognizer ----===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // This file defines the hazard recognizer for scheduling on Hexagon.
     9: //===----------------------------------------------------------------------===//
    10: 
    11: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONPROFITRECOGNIZER_H
    12: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONPROFITRECOGNIZER_H
    13: 
    14: #include "HexagonInstrInfo.h"
    15: #include "HexagonSubtarget.h"
    16: #include "llvm/ADT/SmallSet.h"
    17: #include "llvm/CodeGen/DFAPacketizer.h"
    18: #include "llvm/CodeGen/ScheduleHazardRecognizer.h"
    19: 
    20: namespace llvm {
    21: 
    22: class HexagonHazardRecognizer : public ScheduleHazardRecognizer {
    23:   DFAPacketizer *Resources;
    24:   const HexagonInstrInfo *TII;
    25:   unsigned PacketNum = 0;
```
- EN: It imports headers such as HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SmallSet.h, llvm/CodeGen/DFAPacketizer.h, ... (5 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonHazardRecognizer, which carry the state or API of this component.
- CN: 这里引入了 HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SmallSet.h, llvm/CodeGen/DFAPacketizer.h, ... (5 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonHazardRecognizer 等类型，用来承载该组件的状态或接口。

### Lines 26-50 / 第 26-50 行

```cpp
    26:   // If the packet contains a potential dot cur instruction. This is
    27:   // used for the scheduling priority function.
    28:   SUnit *UsesDotCur = nullptr;
    29:   // The packet number when a dor cur is emitted. If its use is not generated
    30:   // in the same packet, then try to wait another cycle before emitting.
    31:   int DotCurPNum = -1;
    32:   // Does the packet contain a load. Used to restrict another load, if possible.
    33:   bool UsesLoad = false;
    34:   // Check if we should prefer a vector store that will become a .new version.
    35:   // The .new store uses different resources than a normal store, and the
    36:   // packetizer will not generate the .new if the regular store does not have
    37:   // resources available (even if the .new version does). To help, the schedule
    38:   // attempts to schedule the .new as soon as possible in the packet.
    39:   SUnit *PrefVectorStoreNew = nullptr;
    40:   // The set of registers defined by instructions in the current packet.
    41:   SmallSet<unsigned, 8> RegDefs;
    42: 
    43:   // Return true if the instruction is a store that is converted to a new value
    44:   // store because its value is defined in the same packet.
    45:   bool isNewStore(MachineInstr &MI);
    46: 
    47: public:
    48:   HexagonHazardRecognizer(const InstrItineraryData *II,
    49:                           const HexagonInstrInfo *HII,
    50:                           const HexagonSubtarget &ST)
```
- EN: It declares or implements routines such as isNewStore, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonHazardRecognizer, HexagonInstrInfo, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isNewStore 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonHazardRecognizer, HexagonInstrInfo, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:     : Resources(ST.createDFAPacketizer(II)), TII(HII) { }
    52: 
    53:   ~HexagonHazardRecognizer() override { delete Resources; }
    54: 
    55:   /// This callback is invoked when a new block of instructions is about to be
    56:   /// scheduled. The hazard state is set to an initialized state.
    57:   void Reset() override;
    58: 
    59:   /// Return the hazard type of emitting this node.  There are three
    60:   /// possible results.  Either:
    61:   ///  * NoHazard: it is legal to issue this instruction on this cycle.
    62:   ///  * Hazard: issuing this instruction would stall the machine.  If some
    63:   ///     other instruction is available, issue it first.
    64:   HazardType getHazardType(SUnit *SU, int stalls) override;
    65: 
    66:   /// This callback is invoked when an instruction is emitted to be scheduled,
    67:   /// to advance the hazard state.
    68:   void EmitInstruction(SUnit *) override;
    69: 
    70:   /// This callback may be invoked if getHazardType returns NoHazard. If, even
    71:   /// though there is no hazard, it would be better to schedule another
    72:   /// available instruction, this callback should return true.
    73:   bool ShouldPreferAnother(SUnit *) const override;
    74: 
    75:   /// This callback is invoked whenever the next top-down instruction to be
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as Resources, HexagonHazardRecognizer, Reset, getHazardType, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonHazardRecognizer, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 Resources, HexagonHazardRecognizer, Reset, getHazardType, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonHazardRecognizer，说明了它与同级后端组件的连接关系。

### Lines 76-85 / 第 76-85 行

```cpp
    76:   /// scheduled cannot issue in the current cycle, either because of latency
    77:   /// or resource conflicts.  This should increment the internal state of the
    78:   /// hazard recognizer so that previously "Hazard" instructions will now not
    79:   /// be hazards.
    80:   void AdvanceCycle() override;
    81: };
    82: 
    83: } // end namespace llvm
    84: 
    85: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONPROFITRECOGNIZER_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as AdvanceCycle, translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 AdvanceCycle 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- instruction scheduling models / 指令调度模型
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SmallSet.h, llvm/CodeGen/DFAPacketizer.h, llvm/CodeGen/ScheduleHazardRecognizer.h`
- Hexagon symbols / Hexagon 符号: `HexagonHazardRecognizer, HexagonInstrInfo, HexagonSubtarget`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
