# HexagonVLIWPacketizer.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonVLIWPacketizer.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Set to true if the packet contains an instruction that stalls with an
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。 重点涉及指令语义与选择。 重点涉及调度或成包。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonPacketizer.h - VLIW packetizer --------------------*- C++ -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONVLIWPACKETIZER_H
    10: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONVLIWPACKETIZER_H
    11: 
    12: #include "llvm/CodeGen/DFAPacketizer.h"
    13: #include "llvm/CodeGen/MachineBasicBlock.h"
    14: #include "llvm/CodeGen/ScheduleDAG.h"
    15: #include <vector>
    16: 
    17: namespace llvm {
    18: 
    19: class HexagonInstrInfo;
    20: class HexagonRegisterInfo;
    21: class MachineBranchProbabilityInfo;
    22: class MachineFunction;
    23: class MachineInstr;
    24: class MachineLoopInfo;
    25: class TargetRegisterClass;
```
- EN: It imports headers such as llvm/CodeGen/DFAPacketizer.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/ScheduleDAG.h, vector, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonInstrInfo, HexagonRegisterInfo, MachineBranchProbabilityInfo, MachineFunction, ... (7 total), which carry the state or API of this component.
- CN: 这里引入了 llvm/CodeGen/DFAPacketizer.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/ScheduleDAG.h, vector 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonInstrInfo, HexagonRegisterInfo, MachineBranchProbabilityInfo, MachineFunction, ... (7 total) 等类型，用来承载该组件的状态或接口。

### Lines 26-50 / 第 26-50 行

```cpp
    26: 
    27: class HexagonPacketizerList : public VLIWPacketizerList {
    28:   // Vector of instructions assigned to the packet that has just been created.
    29:   std::vector<MachineInstr *> OldPacketMIs;
    30: 
    31:   // Has the instruction been promoted to a dot-new instruction.
    32:   bool PromotedToDotNew;
    33: 
    34:   // Has the instruction been glued to allocframe.
    35:   bool GlueAllocframeStore;
    36: 
    37:   // Has the feeder instruction been glued to new value jump.
    38:   bool GlueToNewValueJump;
    39: 
    40:   // This holds the offset value, when pruning the dependences.
    41:   int64_t ChangedOffset;
    42: 
    43:   // Check if there is a dependence between some instruction already in this
    44:   // packet and this instruction.
    45:   bool Dependence;
    46: 
    47:   // Only check for dependence if there are resources available to
    48:   // schedule this instruction.
    49:   bool FoundSequentialDependence;
    50: 
```
- EN: It declares types such as HexagonPacketizerList, which carry the state or API of this component. It defines declarative TableGen records like HexagonPacketizerList; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonPacketizerList 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonPacketizerList 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:   bool MemShufDisabled = false;
    52: 
    53:   // Track MIs with ignored dependence.
    54:   std::vector<MachineInstr*> IgnoreDepMIs;
    55: 
    56:   // Set to true if the packet contains an instruction that stalls with an
    57:   // instruction from the previous packet.
    58:   bool PacketStalls = false;
    59:   // Set to the number of cycles of stall a given instruction will incur
    60:   // because of dependence on instruction in previous packet.
    61:   unsigned int PacketStallCycles = 0;
    62: 
    63:   // Set to true if the packet has a duplex pair of sub-instructions.
    64:   bool PacketHasDuplex = false;
    65: 
    66:   // Set to true if the packet has a instruction that can only be executed
    67:   // in SLOT0.
    68:   bool PacketHasSLOT0OnlyInsn = false;
    69: 
    70: protected:
    71:   /// A handle to the branch probability pass.
    72:   const MachineBranchProbabilityInfo *MBPI;
    73:   const MachineLoopInfo *MLI;
    74: 
    75: private:
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   const HexagonInstrInfo *HII;
    77:   const HexagonRegisterInfo *HRI;
    78:   const bool Minimal;
    79: 
    80: public:
    81:   HexagonPacketizerList(MachineFunction &MF, MachineLoopInfo &MLI,
    82:                         AAResults *AA, const MachineBranchProbabilityInfo *MBPI,
    83:                         bool Minimal);
    84: 
    85:   // initPacketizerState - initialize some internal flags.
    86:   void initPacketizerState() override;
    87: 
    88:   // ignorePseudoInstruction - Ignore bundling of pseudo instructions.
    89:   bool ignorePseudoInstruction(const MachineInstr &MI,
    90:                                const MachineBasicBlock *MBB) override;
    91: 
    92:   // isSoloInstruction - return true if instruction MI can not be packetized
    93:   // with any other instruction, which means that MI itself is a packet.
    94:   bool isSoloInstruction(const MachineInstr &MI) override;
    95: 
    96:   // isLegalToPacketizeTogether - Is it legal to packetize SUI and SUJ
    97:   // together.
    98:   bool isLegalToPacketizeTogether(SUnit *SUI, SUnit *SUJ) override;
    99: 
   100:   // isLegalToPruneDependencies - Is it legal to prune dependence between SUI
```
- EN: It declares or implements routines such as HexagonPacketizerList, initPacketizerState, ignorePseudoInstruction, isSoloInstruction, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonRegisterInfo, HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPacketizerList, initPacketizerState, ignorePseudoInstruction, isSoloInstruction, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonRegisterInfo, HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   // and SUJ.
   102:   bool isLegalToPruneDependencies(SUnit *SUI, SUnit *SUJ) override;
   103: 
   104:   bool foundLSInPacket();
   105:   MachineBasicBlock::iterator addToPacket(MachineInstr &MI) override;
   106:   void endPacket(MachineBasicBlock *MBB,
   107:                  MachineBasicBlock::iterator MI) override;
   108:   bool shouldAddToPacket(const MachineInstr &MI) override;
   109: 
   110:   void unpacketizeSoloInstrs(MachineFunction &MF);
   111: 
   112: protected:
   113:   bool getmemShufDisabled() {
   114:     return MemShufDisabled;
   115:   };
   116:   void setmemShufDisabled(bool val) {
   117:     MemShufDisabled = val;
   118:   };
   119:   bool isCallDependent(const MachineInstr &MI, SDep::Kind DepType,
   120:                        unsigned DepReg);
   121:   bool promoteToDotCur(MachineInstr &MI, SDep::Kind DepType,
   122:                        MachineBasicBlock::iterator &MII,
   123:                        const TargetRegisterClass *RC);
   124:   bool canPromoteToDotCur(const MachineInstr &MI, const SUnit *PacketSU,
   125:                           unsigned DepReg, MachineBasicBlock::iterator &MII,
```
- EN: It declares or implements routines such as isLegalToPruneDependencies, foundLSInPacket, addToPacket, endPacket, ... (10 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 isLegalToPruneDependencies, foundLSInPacket, addToPacket, endPacket, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 126-150 / 第 126-150 行

```cpp
   126:                           const TargetRegisterClass *RC);
   127:   void cleanUpDotCur();
   128: 
   129:   bool promoteToDotNew(MachineInstr &MI, SDep::Kind DepType,
   130:                        MachineBasicBlock::iterator &MII,
   131:                        const TargetRegisterClass *RC);
   132:   bool canPromoteToDotNew(const MachineInstr &MI, const SUnit *PacketSU,
   133:                           unsigned DepReg, MachineBasicBlock::iterator &MII,
   134:                           const TargetRegisterClass *RC);
   135:   bool canPromoteToNewValue(const MachineInstr &MI, const SUnit *PacketSU,
   136:                             unsigned DepReg, MachineBasicBlock::iterator &MII);
   137:   bool canPromoteToNewValueStore(const MachineInstr &MI,
   138:                                  const MachineInstr &PacketMI, unsigned DepReg);
   139:   bool demoteToDotOld(MachineInstr &MI);
   140:   bool useCallersSP(MachineInstr &MI);
   141:   void useCalleesSP(MachineInstr &MI);
   142:   bool updateOffset(SUnit *SUI, SUnit *SUJ);
   143:   void undoChangedOffset(MachineInstr &MI);
   144:   bool arePredicatesComplements(MachineInstr &MI1, MachineInstr &MI2);
   145:   bool restrictingDepExistInPacket(MachineInstr&, unsigned);
   146:   bool isNewifiable(const MachineInstr &MI, const TargetRegisterClass *NewRC);
   147:   bool isCurifiable(MachineInstr &MI);
   148:   bool cannotCoexist(const MachineInstr &MI, const MachineInstr &MJ);
   149: 
   150:   bool isPromotedToDotNew() const {
```
- EN: It declares or implements routines such as cleanUpDotCur, promoteToDotNew, canPromoteToDotNew, canPromoteToNewValue, ... (16 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 cleanUpDotCur, promoteToDotNew, canPromoteToDotNew, canPromoteToNewValue, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 151-167 / 第 151-167 行

```cpp
   151:     return PromotedToDotNew;
   152:   }
   153: 
   154:   bool tryAllocateResourcesForConstExt(bool Reserve);
   155:   bool canReserveResourcesForConstExt();
   156:   void reserveResourcesForConstExt();
   157:   bool hasDeadDependence(const MachineInstr &I, const MachineInstr &J);
   158:   bool hasControlDependence(const MachineInstr &I, const MachineInstr &J);
   159:   bool hasRegMaskDependence(const MachineInstr &I, const MachineInstr &J);
   160:   bool hasDualStoreDependence(const MachineInstr &I, const MachineInstr &J);
   161:   bool producesStall(const MachineInstr &MI);
   162:   unsigned int calcStall(const MachineInstr &MI);
   163: };
   164: 
   165: } // end namespace llvm
   166: 
   167: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONVLIWPACKETIZER_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as tryAllocateResourcesForConstExt, canReserveResourcesForConstExt, reserveResourcesForConstExt, hasDeadDependence, ... (9 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 tryAllocateResourcesForConstExt, canReserveResourcesForConstExt, reserveResourcesForConstExt, hasDeadDependence, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- instruction scheduling models / 指令调度模型
- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/CodeGen/DFAPacketizer.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/ScheduleDAG.h, vector`
- Hexagon symbols / Hexagon 符号: `HexagonPacketizer, HexagonInstrInfo, HexagonRegisterInfo, HexagonPacketizerList`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
