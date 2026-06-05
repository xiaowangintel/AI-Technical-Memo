# HexagonHazardRecognizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonHazardRecognizer.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file defines the hazard recognizer for scheduling on Hexagon.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===-- HexagonHazardRecognizer.cpp - Hexagon Post RA Hazard Recognizer ---===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file defines the hazard recognizer for scheduling on Hexagon.
    10: // Use a DFA based hazard recognizer.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "HexagonHazardRecognizer.h"
    15: #include "llvm/CodeGen/MachineFunction.h"
    16: #include "llvm/CodeGen/MachineInstr.h"
    17: #include "llvm/CodeGen/MachineOperand.h"
    18: #include "llvm/CodeGen/ScheduleDAG.h"
    19: #include "llvm/Support/Debug.h"
    20: #include "llvm/Support/raw_ostream.h"
    21: #include <cassert>
    22: 
    23: using namespace llvm;
    24: 
    25: #define DEBUG_TYPE "post-RA-sched"
```
- EN: It imports headers such as HexagonHazardRecognizer.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, ... (8 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 HexagonHazardRecognizer.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, ... (8 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 26-50 / 第 26-50 行

```cpp
    26: 
    27: void HexagonHazardRecognizer::Reset() {
    28:   LLVM_DEBUG(dbgs() << "Reset hazard recognizer\n");
    29:   Resources->clearResources();
    30:   PacketNum = 0;
    31:   UsesDotCur = nullptr;
    32:   DotCurPNum = -1;
    33:   UsesLoad = false;
    34:   PrefVectorStoreNew = nullptr;
    35:   RegDefs.clear();
    36: }
    37: 
    38: ScheduleHazardRecognizer::HazardType
    39: HexagonHazardRecognizer::getHazardType(SUnit *SU, int stalls) {
    40:   MachineInstr *MI = SU->getInstr();
    41:   if (!MI || TII->isZeroCost(MI->getOpcode()) || MI->isMetaInstruction())
    42:     return NoHazard;
    43: 
    44:   if (!Resources->canReserveResources(*MI)) {
    45:     LLVM_DEBUG(dbgs() << "*** Hazard in cycle " << PacketNum << ", " << *MI);
    46:     HazardType RetVal = Hazard;
    47:     if (isNewStore(*MI)) {
    48:       // The .new store version uses different resources so check if it
    49:       // causes a hazard.
    50:       MachineFunction *MF = MI->getParent()->getParent();
```
- EN: It declares or implements routines such as HexagonHazardRecognizer::Reset, LLVM_DEBUG, clearResources, clear, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonHazardRecognizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonHazardRecognizer::Reset, LLVM_DEBUG, clearResources, clear, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonHazardRecognizer，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:       MachineInstr *NewMI =
    52:         MF->CreateMachineInstr(TII->get(TII->getDotNewOp(*MI)),
    53:                                MI->getDebugLoc());
    54:       if (Resources->canReserveResources(*NewMI))
    55:         RetVal = NoHazard;
    56:       LLVM_DEBUG(dbgs() << "*** Try .new version? " << (RetVal == NoHazard)
    57:                         << "\n");
    58:       MF->deleteMachineInstr(NewMI);
    59:     }
    60:     return RetVal;
    61:   }
    62: 
    63:   if (SU == UsesDotCur && DotCurPNum != (int)PacketNum) {
    64:     LLVM_DEBUG(dbgs() << "*** .cur Hazard in cycle " << PacketNum << ", "
    65:                       << *MI);
    66:     return Hazard;
    67:   }
    68: 
    69:   return NoHazard;
    70: }
    71: 
    72: void HexagonHazardRecognizer::AdvanceCycle() {
    73:   LLVM_DEBUG(dbgs() << "Advance cycle, clear state\n");
    74:   Resources->clearResources();
    75:   if (DotCurPNum != -1 && DotCurPNum != (int)PacketNum) {
```
- EN: It declares or implements routines such as CreateMachineInstr, LLVM_DEBUG, deleteMachineInstr, HexagonHazardRecognizer::AdvanceCycle, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonHazardRecognizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 CreateMachineInstr, LLVM_DEBUG, deleteMachineInstr, HexagonHazardRecognizer::AdvanceCycle, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonHazardRecognizer，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:     UsesDotCur = nullptr;
    77:     DotCurPNum = -1;
    78:   }
    79:   UsesLoad = false;
    80:   PrefVectorStoreNew = nullptr;
    81:   PacketNum++;
    82:   RegDefs.clear();
    83: }
    84: 
    85: /// Handle the cases when we prefer one instruction over another. Case 1 - we
    86: /// prefer not to generate multiple loads in the packet to avoid a potential
    87: /// bank conflict. Case 2 - if a packet contains a dot cur instruction, then we
    88: /// prefer the instruction that can use the dot cur result. However, if the use
    89: /// is not scheduled in the same packet, then prefer other instructions in the
    90: /// subsequent packet. Case 3 - we prefer a vector store that can be converted
    91: /// to a .new store. The packetizer will not generate the .new store if the
    92: /// store doesn't have resources to fit in the packet (but the .new store may
    93: /// have resources). We attempt to schedule the store as soon as possible to
    94: /// help packetize the two instructions together.
    95: bool HexagonHazardRecognizer::ShouldPreferAnother(SUnit *SU) const {
    96:   if (PrefVectorStoreNew != nullptr && PrefVectorStoreNew != SU)
    97:     return true;
    98:   if (UsesLoad && SU->isInstr() && SU->getInstr()->mayLoad())
    99:     return true;
   100:   return UsesDotCur && ((SU == UsesDotCur) ^ (DotCurPNum == (int)PacketNum));
```
- EN: It declares or implements routines such as clear, packet, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonHazardRecognizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 clear, packet 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonHazardRecognizer，说明了它与同级后端组件的连接关系。

### Lines 101-125 / 第 101-125 行

```cpp
   101: }
   102: 
   103: /// Return true if the instruction would be converted to a new value store when
   104: /// packetized.
   105: bool HexagonHazardRecognizer::isNewStore(MachineInstr &MI) {
   106:   if (!TII->mayBeNewStore(MI))
   107:     return false;
   108:   MachineOperand &MO = MI.getOperand(MI.getNumOperands() - 1);
   109:   return MO.isReg() && RegDefs.contains(MO.getReg());
   110: }
   111: 
   112: void HexagonHazardRecognizer::EmitInstruction(SUnit *SU) {
   113:   MachineInstr *MI = SU->getInstr();
   114:   if (!MI)
   115:     return;
   116: 
   117:   // Keep the set of definitions for each packet, which is used to determine
   118:   // if a .new can be used.
   119:   for (const MachineOperand &MO : MI->operands())
   120:     if (MO.isReg() && MO.isDef() && !MO.isImplicit())
   121:       RegDefs.insert(MO.getReg());
   122: 
   123:   if (TII->isZeroCost(MI->getOpcode()) || MI->isMetaInstruction())
   124:     return;
   125: 
```
- EN: It declares or implements routines such as HexagonHazardRecognizer::isNewStore, getOperand, isReg, HexagonHazardRecognizer::EmitInstruction, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonHazardRecognizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonHazardRecognizer::isNewStore, getOperand, isReg, HexagonHazardRecognizer::EmitInstruction, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonHazardRecognizer，说明了它与同级后端组件的连接关系。

### Lines 126-150 / 第 126-150 行

```cpp
   126:   if (!Resources->canReserveResources(*MI) || isNewStore(*MI)) {
   127:     // It must be a .new store since other instructions must be able to be
   128:     // reserved at this point.
   129:     assert(TII->mayBeNewStore(*MI) && "Expecting .new store");
   130:     MachineFunction *MF = MI->getParent()->getParent();
   131:     MachineInstr *NewMI =
   132:         MF->CreateMachineInstr(TII->get(TII->getDotNewOp(*MI)),
   133:                                MI->getDebugLoc());
   134:     if (Resources->canReserveResources(*NewMI))
   135:       Resources->reserveResources(*NewMI);
   136:     else
   137:       Resources->reserveResources(*MI);
   138:     MF->deleteMachineInstr(NewMI);
   139:   } else
   140:     Resources->reserveResources(*MI);
   141:   LLVM_DEBUG(dbgs() << " Add instruction " << *MI);
   142: 
   143:   // When scheduling a dot cur instruction, check if there is an instruction
   144:   // that can use the dot cur in the same packet. If so, we'll attempt to
   145:   // schedule it before other instructions. We only do this if the load has a
   146:   // single zero-latency use.
   147:   if (TII->mayBeCurLoad(*MI))
   148:     for (auto &S : SU->Succs)
   149:       if (S.isAssignedRegDep() && S.getLatency() == 0 &&
   150:           S.getSUnit()->NumPredsLeft == 1) {
```
- EN: It declares or implements routines such as assert, getParent, CreateMachineInstr, reserveResources, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, getParent, CreateMachineInstr, reserveResources, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 151-170 / 第 151-170 行

```cpp
   151:         UsesDotCur = S.getSUnit();
   152:         DotCurPNum = PacketNum;
   153:         break;
   154:       }
   155:   if (SU == UsesDotCur) {
   156:     UsesDotCur = nullptr;
   157:     DotCurPNum = -1;
   158:   }
   159: 
   160:   UsesLoad = MI->mayLoad();
   161: 
   162:   if (TII->isHVXVec(*MI) && !MI->mayLoad() && !MI->mayStore())
   163:     for (auto &S : SU->Succs)
   164:       if (S.isAssignedRegDep() && S.getLatency() == 0 &&
   165:           TII->mayBeNewStore(*S.getSUnit()->getInstr()) &&
   166:           Resources->canReserveResources(*S.getSUnit()->getInstr())) {
   167:         PrefVectorStoreNew = S.getSUnit();
   168:         break;
   169:       }
   170: }
```
- EN: It declares or implements routines such as getSUnit, mayLoad, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getSUnit, mayLoad 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- instruction scheduling models / 指令调度模型
- register modeling / 寄存器建模
- instruction semantics / 指令语义

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonHazardRecognizer.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/ScheduleDAG.h, llvm/Support/Debug.h, llvm/Support/raw_ostream.h, cassert`
- Hexagon symbols / Hexagon 符号: `HexagonHazardRecognizer`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
