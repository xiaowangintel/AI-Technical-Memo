# HexagonGlobalRegion.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonGlobalRegion.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Void LivenessInfo::parseOperands(MachineInstr *MI, BitVector &Gen,
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及 HVX/向量处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===-- HexagonGlobalRegion.cpp - VLIW global scheduling infrastructure ---===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // Basic infrastructure for global scheduling. Liveness update.
     9: // This is the least complete portion. Basically it is empty infrastructure
    10: // to be extended and improved.
    11: // Currently in place only trace region formation routines and non fully
    12: // functional skeleton for incremental liveness update.
    13: //
    14: //===----------------------------------------------------------------------===//
    15: #define DEBUG_TYPE "global_sched"
    16: #include "HexagonGlobalRegion.h"
    17: #include "HexagonTargetMachine.h"
    18: #include "llvm/Support/raw_ostream.h"
    19: 
    20: using namespace llvm;
    21: 
    22: LivenessInfo::LivenessInfo(const TargetInstrInfo *TII,
    23:                            const TargetRegisterInfo *TRI,
    24:                            MachineBasicBlock *MBB)
    25:     : TII(TII), TRI(TRI) {
    26:   LiveIns.resize(TRI->getNumRegs());
    27:   LiveOuts.resize(TRI->getNumRegs());
    28: 
    29:   LiveIns.reset();
    30:   LiveOuts.reset();
    31: 
    32:   // Live-ins are simple, just copy from MBB.
    33:   for (const auto &LI : MBB->liveins())
    34:     setUsed(LiveIns, LI.PhysReg);
    35: 
    36:   // Live-outs are concatenation of all the BB successors.
    37:   // As of now, we are only dealing with a-cyclic regions
    38:   // with side exits, but no side entrances.
    39:   for (const MachineBasicBlock *Succ : MBB->successors())
    40:     for (const auto &LI : Succ->liveins())
    41:       setUsed(LiveOuts, LI.PhysReg);
    42: }
    43: 
    44: // Pessimistically check if at least one def of this register in this
    45: // instruction (bundle or not) is done under predication.
    46: static bool isPredicatedDef(MachineInstr *MI, unsigned Reg,
    47:                             const HexagonInstrInfo *QII) {
    48:   if (!MI->isBundle())
    49:     return QII->isPredicated(*MI);
    50:   MachineBasicBlock *Parent = MI->getParent();
```
- EN: It imports headers such as HexagonGlobalRegion.h, HexagonTargetMachine.h, llvm/Support/raw_ostream.h, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as LivenessInfo::LivenessInfo, resize, reset, instruction, ... (5 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonGlobalRegion.h, HexagonTargetMachine.h, llvm/Support/raw_ostream.h 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 LivenessInfo::LivenessInfo, resize, reset, instruction, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   if (!Parent)
    52:     return false;
    53:   MachineBasicBlock::instr_iterator MII = MI->getIterator();
    54:   MachineBasicBlock::instr_iterator MIIE = Parent->instr_end();
    55:   for (++MII; MII != MIIE && MII->isInsideBundle(); ++MII) {
    56:     if (!QII->isPredicated(*MII))
    57:       continue;
    58:     for (unsigned i = 0, e = MII->getNumOperands(); i != e; ++i) {
    59:       const MachineOperand &MO = MII->getOperand(i);
    60:       if (!MO.isReg())
    61:         continue;
    62:       if (MO.isDef() && !MO.isDead() && MO.getReg() == Reg) {
    63:         LLVM_DEBUG(dbgs() << "\t\tCond def: "; MII->dump());
    64:         return true;
    65:       }
    66:     }
    67:   }
    68:   return false;
    69: }
    70: 
    71: /// Determine def/use set for MI.
    72: /// Beware, if def is conditional, like here:
    73: /// BUNDLE %PC<imp-def>, %R0<imp-def>, %P0<imp-use,kill>, %R16<imp-use>
    74: ///   * %R0<def> = LDriuh_cdnNotPt %P0<kill,internal>, %R16, 0;
    75: ///   * %P0<def> = C2_cmpeqi %R16, 0;
    76: /// It is not a statefull definition of R0.
    77: ///
    78: void LivenessInfo::parseOperands(MachineInstr *MI, BitVector &Gen,
    79:                                  BitVector &Kill, BitVector &Use) {
    80:   const auto *QII = static_cast<const HexagonInstrInfo *>(TII);
    81: 
    82:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
    83:     const MachineOperand &MO = MI->getOperand(i);
    84:     if (!MO.isReg())
    85:       continue;
    86:     // If it is a predicated instruction, it may, or may not
    87:     // be setting its destination, and we do not know it
    88:     // at the compile time.
    89:     if (MO.isDef() && !MO.isDead()) {
    90:       if (isPredicatedDef(MI, MO.getReg(), QII))
    91:         LLVM_DEBUG(dbgs() << "\tConditional define of "
    92:                           << printReg(MO.getReg(), TRI) << " in ";
    93:                    MI->dump());
    94:       else
    95:         setUsed(Gen, MO.getReg());
    96:     }
    97:     if (MO.isKill())
    98:       setUsed(Kill, MO.getReg());
    99:     if (MO.isUse())
   100:       setUsed(Use, MO.getReg());
```
- EN: It declares or implements routines such as getIterator, instr_end, isInsideBundle, getOperand, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getIterator, instr_end, isInsideBundle, getOperand, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   }
   102: }
   103: 
   104: void LivenessInfo::parseOperandsWithReset(MachineInstr *MI, BitVector &Gen,
   105:                                           BitVector &Kill, BitVector &Use) {
   106:   Gen.reset();
   107:   Use.reset();
   108:   Kill.reset();
   109:   parseOperands(MI, Gen, Kill, Use);
   110: }
   111: 
   112: /// setUsed - Set the register and its sub-registers as being used.
   113: /// Taken from RegScavenger::setUsed().
   114: void LivenessInfo::setUsed(BitVector &Set, unsigned Reg) {
   115:   Set.set(Reg);
   116: 
   117:   for (MCSubRegIterator SubRegs(Reg, TRI); SubRegs.isValid(); ++SubRegs)
   118:     Set.set(*SubRegs);
   119: }
   120: 
   121: #ifndef NDEBUG
   122: static void dumpRI(const TargetRegisterInfo *TRI, BitVector &Set) {
   123:   for (unsigned i = 0; i < Set.size(); i++)
   124:     if (Set.test(i))
   125:       LLVM_DEBUG(dbgs() << " " << printReg(i, TRI));
   126: }
   127: #endif
   128: 
   129: // This function incrementally updates liveness for the given BB.
   130: // First it gathers LiveOut set, and then iterates bottom-up
   131: // over bundles/instructions while updating live set.
   132: void LivenessInfo::UpdateLiveness(MachineBasicBlock *MBB) {
   133:   BitVector NewLiveIns(TRI->getNumRegs());
   134:   BitVector NewLiveOuts(TRI->getNumRegs());
   135:   BitVector LiveIns(TRI->getNumRegs());
   136:   BitVector LocalGen(TRI->getNumRegs());
   137:   BitVector LocalUse(TRI->getNumRegs());
   138:   BitVector LocalKill(TRI->getNumRegs());
   139: 
   140:   NewLiveIns.reset();
   141:   NewLiveOuts.reset();
   142: 
   143:   LLVM_DEBUG(dbgs() << "\n\t\tUpdateLiveness for BB(" << MBB->getNumber()
   144:                     << ")\n");
   145: 
   146:   // Original Live-ins are simple, just copy from MBB.
   147:   for (const auto &LI : MBB->liveins())
   148:     setUsed(NewLiveIns, LI.PhysReg);
   149: 
   150:   // Live-outs are concatenation of all the BB successors.
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as LivenessInfo::parseOperandsWithReset, reset, parseOperands, RegScavenger::setUsed, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 LivenessInfo::parseOperandsWithReset, reset, parseOperands, RegScavenger::setUsed, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   // As of now, we are only dealing with a-cyclic regions
   152:   // with side exits, but no side entrances.
   153:   for (const MachineBasicBlock *Succ : MBB->successors())
   154:     for (const auto &LI : Succ->liveins())
   155:       setUsed(NewLiveOuts, LI.PhysReg);
   156: 
   157:   LiveIns = NewLiveIns;
   158:   // This needs to be a sequential walk, not parallel update.
   159:   LLVM_DEBUG(dbgs() << "\t\tOriginal live ins :\t"; dumpRI(TRI, NewLiveIns);
   160:              dbgs() << "\n");
   161:   LLVM_DEBUG(dbgs() << "\t\tOriginal live outs:\t"; dumpRI(TRI, NewLiveOuts);
   162:              dbgs() << "\n");
   163: 
   164:   NewLiveIns = NewLiveOuts;
   165:   // Scan BB backwards to get exposed uses.
   166:   // TODO: Handle predicates if needed.
   167:   std::vector<MachineInstr *> BundleList;
   168:   for (MachineBasicBlock::iterator MI = MBB->begin(), MIE = MBB->end();
   169:        MI != MIE; ++MI)
   170:     if (!MI->isDebugInstr())
   171:       BundleList.push_back(&*MI);
   172: 
   173:   while (!BundleList.empty()) {
   174:     MachineInstr *MI = BundleList.back();
   175:     BundleList.pop_back();
   176:     parseOperandsWithReset(MI, LocalGen, LocalKill, LocalUse);
   177:     LLVM_DEBUG(dbgs() << "\t\tIncr gen:\t"; dumpRI(TRI, LocalGen);
   178:                dbgs() << "\n");
   179:     LLVM_DEBUG(dbgs() << "\t\tIncr use:\t"; dumpRI(TRI, LocalUse);
   180:                dbgs() << "\n");
   181:     // NewLiveIns = (NewLiveIns - LocalGen) U LocalUse.
   182:     BitVector NotGen(LocalGen);
   183:     NotGen.flip();
   184:     NewLiveIns &= NotGen;
   185:     NewLiveIns |= LocalUse;
   186:   }
   187: 
   188:   LLVM_DEBUG(dbgs() << "\t\tAnswer:\t"; dumpRI(TRI, NewLiveIns);
   189:              dbgs() << "\n");
   190: 
   191:   // TODO: Consider implementing a register aliasing filter if duplicate
   192:   // live-in entries become problematic.
   193: 
   194:   // Set new live in.
   195:   LLVM_DEBUG(dbgs() << "\t\tNew LiveIn       :\t");
   196: 
   197:   for (unsigned i = 0; i < LiveIns.size(); ++i) {
   198:     if (NewLiveIns.test(i))
   199:       LLVM_DEBUG(dbgs() << " " << printReg(i, TRI));
   200:     if (LiveIns.test(i) == NewLiveIns.test(i))
```
- EN: It declares or implements routines such as dumpRI, dbgs, back, pop_back, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 dumpRI, dbgs, back, pop_back, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201:       continue;
   202:     if (LiveIns.test(i))
   203:       MBB->removeLiveIn(i);
   204:     if (NewLiveIns.test(i))
   205:       MBB->addLiveIn(i);
   206:   }
   207:   LLVM_DEBUG(dbgs() << "\n");
   208: }
   209: 
   210: void LivenessInfo::dump() {
   211:   for (unsigned i = 0; i < LiveIns.size(); i++)
   212:     if (LiveIns.test(i))
   213:       LLVM_DEBUG(dbgs() << "\t\tlive-in:  " << printReg(i, TRI) << "\n");
   214:   for (unsigned i = 0; i < LiveOuts.size(); i++)
   215:     if (LiveOuts.test(i))
   216:       LLVM_DEBUG(dbgs() << "\t\tlive-out: " << printReg(i, TRI) << "\n");
   217: }
   218: 
   219: ///
   220: /// BasicBlockRegion Methods.
   221: ///
   222: BasicBlockRegion::BasicBlockRegion(const TargetInstrInfo *TII,
   223:                                    const TargetRegisterInfo *TRI,
   224:                                    MachineBasicBlock *MBB)
   225:     : TII(TII), TRI(TRI) {
   226:   // Should be the root BB.
   227:   Elements.push_back(MBB);
   228:   ElementIndex[MBB] = 0;
   229:   LiveInfo[MBB] = std::make_unique<LivenessInfo>(TII, TRI, MBB);
   230: }
   231: 
   232: BasicBlockRegion::~BasicBlockRegion() {
   233:   LiveInfo.clear();
   234:   Elements.clear();
   235:   ElementIndex.clear();
   236: }
   237: 
   238: LivenessInfo *BasicBlockRegion::getLivenessInfoForBB(MachineBasicBlock *MBB) {
   239:   auto It = LiveInfo.find(MBB);
   240:   assert(It != LiveInfo.end() && "Missing Liveness info");
   241:   assert(It->second && "Missing Liveness info");
   242:   return It->second.get();
   243: }
   244: 
   245: void BasicBlockRegion::addBBtoRegion(MachineBasicBlock *MBB) {
   246:   // It is OK to have duplicates if we reparse for additional BBs.
   247:   if (LiveInfo.find(MBB) != LiveInfo.end())
   248:     return;
   249:   ElementIndex[MBB] = static_cast<unsigned>(Elements.size());
   250:   Elements.push_back(MBB);
```
- EN: It declares or implements routines such as LLVM_DEBUG, LivenessInfo::dump, size, BasicBlockRegion::BasicBlockRegion, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 LLVM_DEBUG, LivenessInfo::dump, size, BasicBlockRegion::BasicBlockRegion, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 251-252 / 第 251-252 行

```cpp
   251:   LiveInfo[MBB] = std::make_unique<LivenessInfo>(TII, TRI, MBB);
   252: }
```
- EN: It declares or implements routines such as std::make_unique<LivenessInfo>, translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 std::make_unique<LivenessInfo> 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonGlobalRegion.h, HexagonTargetMachine.h, llvm/Support/raw_ostream.h`
- Hexagon symbols / Hexagon 符号: `HexagonGlobalRegion, HexagonTargetMachine, HexagonInstrInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
