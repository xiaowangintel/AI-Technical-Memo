# HexagonLiveVariables.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonLiveVariables.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file implements the Hexagon specific LiveVariables analysis pass.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: 
     2: //===----------------- HexagonLiveVariables.cpp ---------------------------===//
     3: //
     4: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     5: // See https://llvm.org/LICENSE.txt for license information.
     6: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     7: //
     8: //===----------------------------------------------------------------------===//
     9: // Hexagon Live Variable Analysis
    10: // This file implements the Hexagon specific LiveVariables analysis pass.
    11: // This pass recomputes physical register liveness and updates live-ins for
    12: // non-entry blocks based on use/def information.
    13: //===----------------------------------------------------------------------===//
    14: #define DEBUG_TYPE "hexagon_live_vars"
    15: 
    16: #include "HexagonLiveVariables.h"
    17: #include "HexagonTargetMachine.h"
    18: #include "llvm/CodeGen/MachineDominators.h"
    19: #include "llvm/CodeGen/MachinePostDominators.h"
    20: #include "llvm/CodeGen/MachineRegisterInfo.h"
    21: #include "llvm/CodeGen/Passes.h"
    22: #include "llvm/InitializePasses.h"
    23: #include "llvm/Support/Debug.h"
    24: #include "llvm/Support/ErrorHandling.h"
    25: 
    26: using namespace llvm;
    27: 
    28: char HexagonLiveVariables::ID = 0;
    29: char &llvm::HexagonLiveVariablesID = HexagonLiveVariables::ID;
    30: 
    31: INITIALIZE_PASS(HexagonLiveVariables, "hexagon-live-vars",
    32:                 "Hexagon Live Variable Analysis", false, false)
    33: 
    34: // TODO: Establish a protocol to handle liveness of predicated instructions.
    35: // Liveness for predicated instruction is a little convoluted.
    36: // TODO: In PhysRegDef and PhysRegUse, use a bit vector instead of 126 elems.
    37: class HexagonLiveVariablesImpl {
    38:   // Intermediate data structures
    39:   friend class llvm::HexagonLiveVariables;
    40:   typedef MachineBasicBlock::const_instr_iterator MICInstIterType;
    41: 
    42:   MachineFunction *MF;
    43: 
    44:   MachineRegisterInfo *MRI;
    45: 
    46:   const TargetRegisterInfo *TRI;
    47: 
    48:   const HexagonInstrInfo *QII;
    49: 
    50:   unsigned NumRegs;
    51: 
    52:   /// PhysRegInfo - Keep track of which instruction was the last def of a
    53:   /// physical register (possibly after a use). This is purely local to a BB.
    54:   SmallVector<MachineInstr *, 0> PhysRegDef;
    55: 
    56:   /// PhysRegInfo - Keep track of which instruction was the last use of a
    57:   /// physical register (before any def). This is purely local property to a BB.
    58:   SmallVector<MachineInstr *, 0> PhysRegUse;
    59: 
    60:   /// MBB -> (Uses, Defs)
    61:   /// Uses - use before any def in that MBB.
    62:   /// Defs - def before any uses  in that MBB.
    63:   MBBUseDef_t MBBUseDefs;
    64: 
    65:   /// MI -> (Uses, Defs)
    66:   MIUseDef_t MIUseDefs;
    67: 
    68:   /// Live-out data for each MBB => U LiveIns (For all Successors of a MBB).
    69:   DenseMap<const MachineBasicBlock *, BitVector> MBBLiveOuts;
    70: 
    71:   /// Each MachineBasicBlock is assigned a Distance which is
    72:   /// an approximation of MBB->size()*INSTR_SIZE+Some offsets.
    73:   /// This is helpful in quickly finding distance between
    74:   /// a branch and its target.
    75:   /// @note A pass which moves instructions should update this.
    76:   /// @note The data in distance map should be used carefully because
    77:   /// difference in the distances of two MI might not give relative distances
    78:   /// between them. The DistanceMap is mainly useful during pullup.
    79:   DenseMap<const MachineBasicBlock *, unsigned> DistanceMap;
    80: 
    81:   // Blocks in depth first order
    82:   SmallVector<MachineBasicBlock *, 16> BlocksDepthFirst;
    83: 
    84:   /// @brief Constructs use-defs of \p MBB by analyzing each MachineOperand.
    85:   /// Collects relevant information so that global liveness can be updated.
    86:   void constructUseDef(MachineBasicBlock *MBB);
    87: 
    88:   /// Collects used-before-define set of registers.
    89:   /// A register is considered to be completely defined if
    90:   /// 1. The register
    91:   /// 2. Any of its super-reg
    92:   /// 3. All of its subregs
    93:   /// are defined. In these cases the register is not considered as
    94:   /// used-before-defined. In case of partial definition of a register
    95:   /// before its use, only the remaining subregs are included in the use-set.
    96:   /// @note: Assumes that a register can be completely defined, by defining
    97:   /// all of its sub-regs (if any).
    98:   void handlePhysRegUse(MachineOperand *MO, MachineInstr *MI, BitVector &Uses);
    99: 
   100:   /// Collects defined-before-use set of registers. If there is any
```
- EN: It imports headers such as HexagonLiveVariables.h, HexagonTargetMachine.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachinePostDominators.h, ... (9 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonLiveVariablesImpl, which carry the state or API of this component.
- CN: 这里引入了 HexagonLiveVariables.h, HexagonTargetMachine.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachinePostDominators.h, ... (9 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonLiveVariablesImpl 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:   /// use of register or its aliases then the register is not counted
   102:   /// as defined-before-use
   103:   /// @note: Assumes that a register can be completely defined, by defining
   104:   /// all of its sub-regs (if any).
   105:   void handlePhysRegDef(MachineOperand *MO, MachineInstr *MI, BitVector &Defs);
   106: 
   107:   /// updateGlobalLiveness - wrapper around another overload
   108:   inline bool updateGlobalLiveness(MachineFunction &Fn);
   109:   bool updateGlobalLiveness(MachineBasicBlock *X, MachineBasicBlock *Y);
   110: 
   111:   /// updateGlobalLiveness - updates liveness based on
   112:   /// livein and liveout entries.
   113:   bool updateGlobalLiveness(MachineBasicBlock *MBB, BitVector &Defs,
   114:                             BitVector &LiveIns);
   115: 
   116:   /// update live-ins when live-out has been calculated
   117:   bool updateLiveIns(MachineBasicBlock *MBB, BitVector &LiveIns,
   118:                      const BitVector &LiveOuts);
   119: 
   120:   bool updateLiveOuts(MachineBasicBlock *MBB, BitVector &LiveOuts);
   121: 
   122:   /// updateLocalLiveness - update only kill flags of operands.
   123:   inline bool updateLocalLiveness(MachineFunction &Fn);
   124: 
   125:   /// updateLocalLiveness - update only kill flags of operands.
   126:   bool updateLocalLiveness(MachineBasicBlock *MBB, bool UpdateBundle);
   127: 
   128:   /// incrementalUpdate - update the liveness when \p MIDelta is moved from
   129:   /// \p From to \p To.
   130:   /// @note: This is extremely fragile now. It 'assumes' that the other
   131:   /// successor(s) of \p To do not use Defs of MIDelta.
   132:   /// It deletes the live-in of the \p From MBB.
   133:   bool incrementalUpdate(MICInstIterType MIDelta, MachineBasicBlock *From,
   134:                          MachineBasicBlock *To);
   135: 
   136:   /// addNewMBB - inform the LiveVariable Analysis that new MBB has been added.
   137:   /// update the liveness of this new MBB.
   138:   /// @note MBB should be empty. If we want to add an MI, add it after calling
   139:   /// this function.
   140:   void addNewMBB(MachineBasicBlock *MBB);
   141: 
   142:   void addNewMI(MachineInstr *MI, MachineBasicBlock *MBB);
   143:   unsigned getNumRegs() const { return NumRegs; }
   144: 
   145:   // Useful for clearing out after passes which move instructions around.
   146:   // e.g. GlobalScheduler.
   147:   void clearDistanceMap() { DistanceMap.clear(); }
   148: 
   149:   /// Computes \p DistanceMap.
   150:   void generateDistanceMap(const MachineFunction &Fn);
   151: 
   152: public:
   153:   bool runOnMachineFunction(MachineFunction &Fn, MachineDominatorTree &MDT,
   154:                             MachinePostDominatorTree &MPDT);
   155: };
   156: 
   157: //===----------------------------------------------------------------------===//
   158: //                    HexagonLiveVariables Functions
   159: //===----------------------------------------------------------------------===//
   160: HexagonLiveVariables::HexagonLiveVariables()
   161:     : MachineFunctionPass(ID), HLVComplete(false),
   162:       HLV(std::make_unique<HexagonLiveVariablesImpl>()) {
   163:   initializeHexagonLiveVariablesPass(*PassRegistry::getPassRegistry());
   164: }
   165: 
   166: void HexagonLiveVariables::getAnalysisUsage(AnalysisUsage &AU) const {
   167:   AU.setPreservesCFG();
   168:   AU.addRequired<MachineDominatorTreeWrapperPass>();
   169:   AU.addRequired<MachinePostDominatorTreeWrapperPass>();
   170:   AU.addPreserved<MachineDominatorTreeWrapperPass>();
   171:   AU.addPreserved<MachinePostDominatorTreeWrapperPass>();
   172:   AU.addPreserved("packets");
   173:   MachineFunctionPass::getAnalysisUsage(AU);
   174: }
   175: 
   176: void HexagonLiveVariables::recalculate(MachineFunction &MF) {
   177:   if (HLVComplete)
   178:     return;
   179:   auto &MDT = getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
   180:   auto &MPDT =
   181:       getAnalysis<MachinePostDominatorTreeWrapperPass>().getPostDomTree();
   182:   HLV->runOnMachineFunction(MF, MDT, MPDT);
   183: }
   184: 
   185: bool HexagonLiveVariables::updateLocalLiveness(MachineFunction &Fn) {
   186:   return HLV->updateLocalLiveness(Fn);
   187: }
   188: 
   189: bool HexagonLiveVariables::updateLocalLiveness(MachineBasicBlock *MBB,
   190:                                                bool updateBundle) {
   191:   HLV->constructUseDef(MBB); // XXX: This destroys MBBLiveOuts!
   192:   return HLV->updateLocalLiveness(MBB, updateBundle);
   193: }
   194: 
   195: bool HexagonLiveVariables::incrementalUpdate(MICInstIterType MIDelta,
   196:                                              MachineBasicBlock *From,
   197:                                              MachineBasicBlock *To) {
   198:   assert(MIDelta->getParent() == To);
   199:   assert(From != To);
   200:   return HLV->incrementalUpdate(MIDelta, From, To);
```
- EN: It declares or implements routines such as regs, updateGlobalLiveness, updateLiveIns, updateLiveOuts, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLiveVariables, HexagonLiveVariablesImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 regs, updateGlobalLiveness, updateLiveIns, updateLiveOuts, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariables, HexagonLiveVariablesImpl，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201: }
   202: 
   203: void HexagonLiveVariables::addNewMBB(MachineBasicBlock *MBB) {
   204:   assert(MBB->empty());
   205:   HLV->addNewMBB(MBB);
   206: }
   207: 
   208: void HexagonLiveVariables::addNewMI(MachineInstr *MI, MachineBasicBlock *MBB) {
   209:   HLV->addNewMI(MI, MBB);
   210: }
   211: 
   212: void HexagonLiveVariables::constructUseDef(MachineBasicBlock *MBB) {
   213:   HLV->constructUseDef(MBB);
   214: }
   215: 
   216: bool HexagonLiveVariables::runOnMachineFunction(MachineFunction &Fn) {
   217:   auto &MDT = getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
   218:   auto &MPDT =
   219:       getAnalysis<MachinePostDominatorTreeWrapperPass>().getPostDomTree();
   220:   HLVComplete = !HLV->runOnMachineFunction(Fn, MDT, MPDT);
   221:   return HLVComplete;
   222: }
   223: 
   224: bool HexagonLiveVariables::isLiveOut(const MachineBasicBlock *MBB,
   225:                                      unsigned Reg) const {
   226:   assert(HLVComplete && "Liveness Analysis not available");
   227:   auto It = HLV->MBBLiveOuts.find(MBB);
   228:   if (It == HLV->MBBLiveOuts.end())
   229:     llvm_unreachable("MBB not found in liveness map");
   230:   if (Reg >= It->second.size())
   231:     llvm_unreachable("Register index out of bounds");
   232:   return It->second[Reg];
   233: }
   234: 
   235: const BitVector &
   236: HexagonLiveVariables::getLiveOuts(const MachineBasicBlock *MBB) const {
   237:   assert(HLVComplete && "Liveness Analysis not available");
   238:   auto It = HLV->MBBLiveOuts.find(MBB);
   239:   if (It == HLV->MBBLiveOuts.end())
   240:     llvm_unreachable("MBB not found in liveness map");
   241:   return It->second;
   242: }
   243: 
   244: // Returns true when \p Reg is used within [MIBegin, MIEnd)
   245: // @note: MIBegin and MIEnd should be from same MBB
   246: // @note: It returns just the first use found in the range.
   247: // The Use is closest to MIEnd.
   248: // Takes care of aliases and predicated defs as well.
   249: bool HexagonLiveVariables::isUsedWithin(
   250:     MICInstIterType MIBegin, MICInstIterType MIEnd, unsigned Reg,
   251:     MICInstIterType &Use,
   252:     SmallPtrSet<MachineInstr *, 2> *ExceptionsList) const {
   253:   assert(HLVComplete && "Liveness Analysis not available");
   254:   Use = MIEnd;
   255:   if (MIBegin == MIEnd) // NULL Range.
   256:     return false;
   257:   MICInstIterType MII = MIEnd;
   258:   do {
   259:     --MII;
   260:     if (MII->isBundle() || MII->isDebugInstr())
   261:       continue;
   262:     if (ExceptionsList && ExceptionsList->contains(&*MII))
   263:       continue;
   264:     auto It = HLV->MIUseDefs.find(&*MII);
   265:     assert(It != HLV->MIUseDefs.end());
   266:     for (MCRegAliasIterator AI(Reg, HLV->TRI, true); AI.isValid(); ++AI)
   267:       if (It->second.first[*AI]) {
   268:         Use = MII;
   269:         return true;
   270:       }
   271:   } while (MII != MIBegin);
   272:   return false;
   273: }
   274: 
   275: // Returns true when \p Reg id defined within [MIBegin, MIEnd)
   276: // @note: MIBegin and MIEnd should be from same MBB
   277: // The Def is closest to MIEnd.
   278: // Takes care of aliases and predicated defs as well.
   279: bool HexagonLiveVariables::isDefinedWithin(MICInstIterType MIBegin,
   280:                                            MICInstIterType MIEnd, unsigned Reg,
   281:                                            MICInstIterType &Def) const {
   282:   assert(HLVComplete && "Liveness Analysis not available");
   283:   Def = MIEnd;
   284:   if (MIBegin == MIEnd) // NULL Range.
   285:     return false;
   286:   MICInstIterType MII = MIEnd;
   287:   do {
   288:     --MII;
   289:     if (MII->isBundle() || MII->isDebugInstr())
   290:       continue;
   291:     auto It = HLV->MIUseDefs.find(&*MII);
   292:     assert(It != HLV->MIUseDefs.end());
   293:     for (MCRegAliasIterator AI(Reg, HLV->TRI, true); AI.isValid(); ++AI)
   294:       if (It->second.second[*AI]) {
   295:         Def = MII;
   296:         return true;
   297:       }
   298:   } while (MII != MIBegin);
   299:   return false;
   300: }
```
- EN: It declares or implements routines such as HexagonLiveVariables::addNewMBB, assert, addNewMBB, HexagonLiveVariables::addNewMI, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLiveVariables, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonLiveVariables::addNewMBB, assert, addNewMBB, HexagonLiveVariables::addNewMI, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariables，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301: 
   302: // Returns true if any of the defs of MII is live-in in the MBB.
   303: bool HexagonLiveVariables::isDefLiveIn(const MachineInstr *MI,
   304:                                        const MachineBasicBlock *MBB) const {
   305:   assert(HLVComplete && "Liveness Analysis not available");
   306:   assert(MI && "Invalid machine instruction");
   307:   assert(MBB && "Invalid machine basic block");
   308:   auto It = HLV->MIUseDefs.find(MI);
   309:   assert(It != HLV->MIUseDefs.end() && "Missing MI use/def information");
   310:   BitVector MBBLiveIns(HLV->NumRegs);
   311:   for (MachineBasicBlock::livein_iterator lit = MBB->livein_begin();
   312:        lit != MBB->livein_end(); ++lit) {
   313:     // Include all the aliases of reg *lit.
   314:     for (MCRegAliasIterator AI((*lit).PhysReg, HLV->TRI, true); AI.isValid();
   315:          ++AI)
   316:       MBBLiveIns.set(*AI);
   317:   }
   318:   // Intersect.
   319:   return MBBLiveIns.anyCommon(It->second.second);
   320: }
   321: 
   322: MBBUseDef_t &HexagonLiveVariables::getMBBUseDefs() { return HLV->MBBUseDefs; }
   323: 
   324: MIUseDef_t &HexagonLiveVariables::getMIUseDefs() { return HLV->MIUseDefs; }
   325: 
   326: unsigned HexagonLiveVariables::getDistanceBetween(const MachineBasicBlock *From,
   327:                                                   const MachineBasicBlock *To,
   328:                                                   unsigned BufferPerMBB) const {
   329:   assert(HLV->DistanceMap.find(From) != HLV->DistanceMap.end());
   330:   assert(HLV->DistanceMap.find(To) != HLV->DistanceMap.end());
   331:   unsigned FromSize = HLV->DistanceMap[From];
   332:   if (From == To)
   333:     return FromSize;
   334:   const MachineFunction *MF = From->getParent();
   335:   MachineFunction::const_iterator MBBI = MF->begin();
   336:   unsigned S = BufferPerMBB;
   337:   bool ToFirst = false;
   338:   while (MBBI != MF->end()) {
   339:     const MachineBasicBlock *MBB = &*MBBI;
   340:     if (MBB == From)
   341:       break;
   342:     else if (MBB == To) {
   343:       ToFirst = true;
   344:       break;
   345:     }
   346:     ++MBBI;
   347:   }
   348:   const MachineBasicBlock *ToFind = To;
   349:   if (ToFirst)
   350:     ToFind = From;
   351:   while (MBBI != MF->end()) {
   352:     const MachineBasicBlock *MBB = &*MBBI;
   353:     if (MBB == ToFind)
   354:       break;
   355:     S += HLV->DistanceMap[MBB] + BufferPerMBB;
   356:     ++MBBI;
   357:   }
   358:   if (ToFirst) // Jump in the opposite direction.
   359:     S += FromSize + HLV->DistanceMap[To] + 2 * BufferPerMBB;
   360:   return S;
   361: }
   362: 
   363: void HexagonLiveVariables::regenerateDistanceMap(const MachineFunction &Fn) {
   364:   HLV->clearDistanceMap();
   365:   HLV->generateDistanceMap(Fn);
   366: }
   367: 
   368: //===----------------------------------------------------------------------===//
   369: //                    HexagonLiveVariablesImpl Functions
   370: //===----------------------------------------------------------------------===//
   371: bool HexagonLiveVariablesImpl::runOnMachineFunction(
   372:     MachineFunction &Fn, MachineDominatorTree &MDT,
   373:     MachinePostDominatorTree &MPDT) {
   374:   LLVM_DEBUG(dbgs() << "\nHexagon Live Variables";);
   375:   Fn.RenumberBlocks();
   376: 
   377:   MF = &Fn;
   378:   MRI = &Fn.getRegInfo();
   379:   auto &ST = Fn.getSubtarget<HexagonSubtarget>();
   380:   TRI = ST.getRegisterInfo();
   381:   QII = ST.getInstrInfo();
   382: 
   383:   NumRegs = TRI->getNumRegs();
   384: 
   385:   MBBUseDefs.clear();
   386:   MIUseDefs.clear();
   387:   MBBLiveOuts.clear();
   388: 
   389:   LLVM_DEBUG(dbgs() << "\nNumber of registers in Hexagon is:" << NumRegs);
   390: 
   391:   PhysRegDef.resize(NumRegs);
   392:   PhysRegUse.resize(NumRegs);
   393: 
   394:   for (MachineFunction::iterator MBBI = Fn.begin(), E = Fn.end(); MBBI != E;
   395:        ++MBBI) {
   396:     constructUseDef(&*MBBI);
   397:   }
   398:   updateGlobalLiveness(Fn);
   399:   return false;
   400: }
```
- EN: It declares or implements routines such as HexagonLiveVariables::isDefLiveIn, assert, find, MBBLiveIns, ... (28 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLiveVariables, HexagonLiveVariablesImpl, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonLiveVariables::isDefLiveIn, assert, find, MBBLiveIns, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariables, HexagonLiveVariablesImpl, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401: 
   402: void HexagonLiveVariablesImpl::constructUseDef(MachineBasicBlock *MBB) {
   403:   std::fill(PhysRegDef.begin(), PhysRegDef.end(), (MachineInstr *)0);
   404:   std::fill(PhysRegUse.begin(), PhysRegUse.end(), (MachineInstr *)0);
   405: 
   406:   // Loop over all of the instructions, processing them.
   407:   std::pair<BitVector, BitVector> &UseDef = MBBUseDefs[MBB];
   408:   // Use before any def in a BB.
   409:   BitVector &Uses = UseDef.first;
   410:   // Defs before any use in a BB.
   411:   BitVector &Defs = UseDef.second;
   412:   // Initializing the LiveOut bit vector.
   413:   BitVector &LiveOuts = MBBLiveOuts[MBB];
   414:   Uses.resize(NumRegs, false);
   415:   Defs.resize(NumRegs, false);
   416:   LiveOuts.resize(NumRegs, false);
   417:   // BitVector might contain set bits out of previous liveness updates.
   418:   Uses.reset();
   419:   Defs.reset();
   420:   LiveOuts.reset();
   421:   LLVM_DEBUG(dbgs() << "\nBB#" << MBB->getNumber(););
   422:   // MBB Number in the MSB 32 bits.
   423:   unsigned MBBInsSize = 0;
   424:   for (MachineBasicBlock::instr_iterator MII = MBB->instr_begin(),
   425:                                          E = MBB->instr_end();
   426:        MII != E; ++MII) {
   427:     MachineInstr *MI = &*MII;
   428:     MBBInsSize += QII->getSize(*MI);
   429:     // TODO: Handle isDebugInstr
   430:     if (MI->isBundle() || MI->isDebugInstr())
   431:       continue;
   432:     LLVM_DEBUG(dbgs() << "\n\n" << *MI;);
   433:     // Clear kill and dead markers. LV will recompute them.
   434:     UseDef_t &MIUseDef = MIUseDefs[MI];
   435:     MIUseDef.first.resize(NumRegs);  // Uses
   436:     MIUseDef.second.resize(NumRegs); // Defs
   437:     MIUseDef.first.reset();          // Uses
   438:     MIUseDef.second.reset();         // Defs
   439: 
   440:     SmallVector<MachineOperand *, 4> UseRegs;
   441:     SmallVector<MachineOperand *, 4> DefRegs;
   442:     SmallVector<unsigned, 1> RegMasks;
   443:     // Process all of the operands of the instruction...
   444:     unsigned NumOperandsToProcess = MI->getNumOperands();
   445:     for (unsigned i = 0; i != NumOperandsToProcess; ++i) {
   446:       MachineOperand &MO = MI->getOperand(i);
   447:       if (MO.isRegMask()) {
   448:         // Assuming that predicated defs are not defs, for now.
   449:         if (!QII->isPredicated(*MI))
   450:           DefRegs.push_back(&MO);
   451:         continue;
   452:       }
   453:       if (!MO.isReg() || MO.getReg() == 0)
   454:         continue;
   455:       unsigned Reg = MO.getReg();
   456:       if (MO.isUse()) {
   457:         // Assuming that the kill-flags on call-instructions are correct.
   458:         MO.setIsKill(false);
   459:         UseRegs.push_back(&MO);
   460:         MIUseDef.first.set(Reg);
   461:       } else /*MO.isDef()*/ {
   462:         assert(MO.isDef());
   463:         if (!QII->isPredicated(*MI) && !MI->isKill()) {
   464:           // Assuming that predicated defs are not defs, for now.
   465:           // KILL instructions are no-ops
   466:           MO.setIsDead(false);
   467:           DefRegs.push_back(&MO);
   468:         }
   469:         MIUseDef.second.set(Reg); // Set all defs (including predicated).
   470:       }
   471:     }
   472:     // Process all uses.
   473:     for (unsigned i = 0, e = UseRegs.size(); i != e; ++i)
   474:       handlePhysRegUse(UseRegs[i], MI, Uses);
   475:     // Process all defs.
   476:     for (unsigned i = 0, e = DefRegs.size(); i != e; ++i)
   477:       handlePhysRegDef(DefRegs[i], MI, Defs);
   478:   }
   479:   DistanceMap[MBB] = MBBInsSize;
   480: }
   481: 
   482: void HexagonLiveVariablesImpl::handlePhysRegUse(MachineOperand *MO,
   483:                                                 MachineInstr *MI,
   484:                                                 BitVector &Uses) {
   485:   unsigned Reg = MO->getReg();
   486:   LLVM_DEBUG(dbgs() << "\nLooking at:";);
   487:   // If the reg/super-reg is already defined in this MBB => return.
   488:   for (MCSuperRegIterator SupI(Reg, TRI, true); SupI.isValid(); ++SupI) {
   489:     LLVM_DEBUG(dbgs() << printReg(*SupI, TRI););
   490:     if (PhysRegDef[*SupI])
   491:       return;
   492:   }
   493:   // Handle if sub-regs are defined.
   494:   SmallVector<unsigned, 2> undefSubRegs;
   495:   bool subRegDefined = false;
   496:   for (MCSubRegIterator SubI(Reg, TRI); SubI.isValid(); ++SubI) {
   497:     LLVM_DEBUG(dbgs() << printReg(*SubI, TRI););
   498:     if (PhysRegDef[*SubI])
   499:       subRegDefined = true;
   500:     else
```
- EN: It declares or implements routines such as HexagonLiveVariablesImpl::constructUseDef, std::fill, resize, reset, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLiveVariablesImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonLiveVariablesImpl::constructUseDef, std::fill, resize, reset, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariablesImpl，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:       undefSubRegs.push_back(*SubI);
   502:   }
   503: 
   504:   LLVM_DEBUG(dbgs() << "\nUses:");
   505:   if (undefSubRegs.empty()) {
   506:     if (!subRegDefined) { // None of the subregs are defined.
   507:       // Include all subregs (including self) to the uses.
   508:       for (MCSubRegIterator SubI(Reg, TRI, true); SubI.isValid(); ++SubI) {
   509:         LLVM_DEBUG(dbgs() << printReg(*SubI, TRI));
   510:         PhysRegUse[*SubI] = MI;
   511:         Uses.set(*SubI);
   512:       }
   513:     } // All subregs defined.
   514:     return;
   515:   }
   516:   // Some subregs are defined.
   517:   for (unsigned i = 0; i < undefSubRegs.size(); ++i) {
   518:     LLVM_DEBUG(dbgs() << printReg(undefSubRegs[i], TRI));
   519:     PhysRegUse[undefSubRegs[i]] = MI;
   520:     Uses.set(undefSubRegs[i]);
   521:   }
   522: }
   523: 
   524: // Assumes that an MI cannot have a reg and its super/sub reg as uses.
   525: void HexagonLiveVariablesImpl::handlePhysRegDef(MachineOperand *MO,
   526:                                                 MachineInstr *MI,
   527:                                                 BitVector &Defs) {
   528:   auto SetRegDef = [&](unsigned Reg) -> void {
   529:     PhysRegDef[Reg] = MI;
   530:     for (MCRegAliasIterator AI(Reg, TRI, true); AI.isValid(); ++AI) {
   531:       if (PhysRegUse[*AI]) {
   532:         LLVM_DEBUG(dbgs() << "\nUsed in current BB:" << printReg(*AI, TRI));
   533:         return;
   534:       }
   535:     }
   536:     LLVM_DEBUG(dbgs() << "\nDefs:" << printReg(Reg, TRI));
   537:     Defs.set(Reg);
   538:   };
   539: 
   540:   if (MO->isReg()) {
   541:     SetRegDef(MO->getReg());
   542:   } else if (MO->isRegMask()) {
   543:     for (unsigned R = 1, NR = TRI->getNumRegs(); R != NR; ++R)
   544:       if (MO->clobbersPhysReg(R))
   545:         SetRegDef(R);
   546:   }
   547: }
   548: 
   549: namespace {
   550: struct BlockState {
   551:   bool SuccQueued : 1;
   552:   bool Done : 1;
   553:   BlockState() : SuccQueued(false), Done(false) {}
   554: };
   555: } // namespace
   556: 
   557: // Populates 'Blocks' with basic blocks of 'Fn' in depth-first order
   558: static void gatherBlocksDF(MachineFunction &Fn,
   559:                            SmallVectorImpl<MachineBasicBlock *> *Blocks) {
   560:   Blocks->clear();
   561:   Blocks->reserve(Fn.size());
   562: 
   563:   SmallVector<BlockState, 16> State(Fn.size());
   564:   SmallVector<MachineBasicBlock *, 16> WorkStack;
   565:   WorkStack.push_back(&Fn.front());
   566:   while (!WorkStack.empty()) {
   567:     MachineBasicBlock *W = WorkStack.back();
   568:     BlockState &WState = State[W->getNumber()];
   569:     if (WState.Done) {
   570:       WorkStack.pop_back();
   571:       continue;
   572:     }
   573:     if (W->succ_empty() || WState.SuccQueued) {
   574:       WorkStack.pop_back();
   575:       Blocks->push_back(W);
   576:       WState.SuccQueued = true;
   577:       WState.Done = true;
   578:       continue;
   579:     }
   580:     WState.SuccQueued = true;
   581:     for (MachineBasicBlock::succ_iterator I = W->succ_begin(),
   582:                                           E = W->succ_end();
   583:          I != E; ++I) {
   584:       MachineBasicBlock *S = *I;
   585:       if (State[S->getNumber()].SuccQueued)
   586:         continue;
   587:       WorkStack.push_back(S);
   588:     }
   589:   }
   590: 
   591:   LLVM_DEBUG(
   592:       dbgs() << "gatherBlocksDF: {";
   593:       for (SmallVectorImpl<MachineBasicBlock *>::iterator B = Blocks->begin(),
   594:            BE = Blocks->end();
   595:            B != BE; ++B) { dbgs() << " BB#" << (*B)->getNumber(); } dbgs()
   596:       << " }\n";);
   597: }
   598: 
   599: bool HexagonLiveVariablesImpl::updateGlobalLiveness(MachineFunction &Fn) {
   600:   bool Changed = false;
```
- EN: It declares types such as BlockState, which carry the state or API of this component. It declares or implements routines such as push_back, LLVM_DEBUG, subregs, isValid, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLiveVariablesImpl, showing how the code connects to sibling backend components.
- CN: 这里声明了 BlockState 等类型，用来承载该组件的状态或接口。 这里声明或实现了 push_back, LLVM_DEBUG, subregs, isValid, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariablesImpl，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:   // Removing live-ins and recomputing.
   602:   MachineFunction::iterator I = Fn.begin(), E = Fn.end();
   603:   // Not touching the live-ins of entry basic block.
   604:   for (++I; I != E; ++I) {
   605:     std::vector<MachineBasicBlock::RegisterMaskPair> OldLiveIn(
   606:         I->livein_begin(), I->livein_end());
   607:     for (unsigned i = 0; i < OldLiveIn.size(); ++i)
   608:       I->removeLiveIn(OldLiveIn[i].PhysReg);
   609:   }
   610: 
   611:   gatherBlocksDF(Fn, &BlocksDepthFirst);
   612: 
   613:   BitVector Defs;
   614:   BitVector LiveIns;
   615:   bool Repeat;
   616:   do {
   617:     Repeat = false;
   618:     for (SmallVectorImpl<MachineBasicBlock *>::iterator
   619:              B = BlocksDepthFirst.begin(),
   620:              BE = BlocksDepthFirst.end();
   621:          B != BE; ++B) {
   622:       Repeat |= updateGlobalLiveness(*B, Defs, LiveIns);
   623:     }
   624:     Changed |= Repeat;
   625:   } while (Repeat);
   626: 
   627:   Changed |= updateLocalLiveness(Fn);
   628:   return Changed;
   629: }
   630: 
   631: bool HexagonLiveVariablesImpl::updateGlobalLiveness(MachineBasicBlock *X,
   632:                                                     MachineBasicBlock *Y) {
   633:   assert(X && "Invalid start block");
   634:   assert(Y && "Invalid end block");
   635: 
   636:   bool Changed = false;
   637:   BitVector Defs;
   638:   BitVector LiveIns;
   639: 
   640:   const SmallVectorImpl<MachineBasicBlock *>::iterator BE =
   641:       BlocksDepthFirst.end();
   642:   SmallVectorImpl<MachineBasicBlock *>::iterator B;
   643:   for (B = BlocksDepthFirst.begin(); (B != BE); ++B) {
   644:     if (*B == X)
   645:       break;
   646:     if (*B == Y)
   647:       break;
   648:   }
   649: 
   650:   bool Repeat;
   651:   do {
   652:     Repeat = false;
   653:     for (; B != BE; ++B)
   654:       Repeat |= updateGlobalLiveness(*B, Defs, LiveIns);
   655:     Changed |= Repeat;
   656:     B = BlocksDepthFirst.begin();
   657:   } while (Repeat);
   658: 
   659:   return Changed;
   660: }
   661: 
   662: // Defs and LiveIns could be local variables within updateGlobalLiveness, but
   663: // have been pulled out to (hopefully) improve performance.
   664: bool HexagonLiveVariablesImpl::updateGlobalLiveness(MachineBasicBlock *MBB,
   665:                                                     BitVector &Defs,
   666:                                                     BitVector &LiveIns) {
   667:   LLVM_DEBUG(dbgs() << "\nTrying to Update Liveness MBB#" << MBB->getNumber());
   668:   bool Changed = false;
   669:   LLVM_DEBUG(dbgs() << "\nUpdating Liveness MBB#" << MBB->getNumber());
   670:   // Update live-outs
   671:   auto LiveOutIt = MBBLiveOuts.find(MBB);
   672:   if (LiveOutIt == MBBLiveOuts.end())
   673:     LiveOutIt = MBBLiveOuts.insert({MBB, BitVector(NumRegs)}).first;
   674:   BitVector &LiveOuts = LiveOutIt->second;
   675:   for (MachineBasicBlock::succ_iterator MBBSucc = MBB->succ_begin();
   676:        MBBSucc != MBB->succ_end(); ++MBBSucc) {
   677:     MachineBasicBlock *Succ = *MBBSucc;
   678:     LLVM_DEBUG(dbgs() << "\n\t\tAdding LiveOut:";);
   679:     for (MachineBasicBlock::livein_iterator LI = Succ->livein_begin(),
   680:                                             LE = Succ->livein_end();
   681:          LI != LE; ++LI) {
   682:       if (!LiveOuts[(*LI).PhysReg]) {
   683:         LLVM_DEBUG(dbgs() << " " << printReg((*LI).PhysReg, TRI););
   684:         LiveOuts.set((*LI).PhysReg);
   685:         Changed = true;
   686:       }
   687:     }
   688:   }
   689:   LLVM_DEBUG(dbgs() << "\nUpdated Successors of MBB#" << MBB->getNumber());
   690:   // Update live-ins
   691:   Changed |= updateLiveIns(MBB, LiveIns, LiveOuts);
   692: 
   693:   return Changed;
   694: }
   695: 
   696: // update live-ins when live-out has been calculated
   697: bool HexagonLiveVariablesImpl::updateLiveIns(MachineBasicBlock *MBB,
   698:                                              BitVector &LiveIns,
   699:                                              const BitVector &LiveOuts) {
   700:   LLVM_DEBUG(dbgs() << "\n[updateLiveIns] MBB#" << MBB->getNumber());
```
- EN: It declares or implements routines such as begin, OldLiveIn, size, removeLiveIn, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLiveVariablesImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 begin, OldLiveIn, size, removeLiveIn, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariablesImpl，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:   bool Changed = false;
   702:   const std::pair<BitVector, BitVector> &UseDefs = MBBUseDefs[MBB];
   703:   LiveIns = LiveOuts;
   704:   // LiveIns = (LiveOuts - Defs) | Uses
   705:   // Equivalent to: LiveIns = (LiveOuts & ~Defs) | Uses
   706:   LiveIns.reset(UseDefs.second);
   707:   LiveIns |= UseDefs.first;
   708:   LLVM_DEBUG(dbgs() << "\n\t\tAdded LiveIn:";);
   709:   for (int i = LiveIns.find_first(); i >= 0; i = LiveIns.find_next(i)) {
   710:     // TODO: remove costly check of MBB->isLiveIn when fully functional.
   711:     if (!MBB->isLiveIn(i) && MRI->isAllocatable(i)) {
   712:       LLVM_DEBUG(dbgs() << " " << printReg(i, TRI));
   713:       MBB->addLiveIn(i);
   714:       Changed = true;
   715:     }
   716:   }
   717:   return Changed;
   718: }
   719: 
   720: bool HexagonLiveVariablesImpl::updateLiveOuts(MachineBasicBlock *MBB,
   721:                                               BitVector &LiveOuts) {
   722:   bool Changed = false;
   723:   for (auto SI = MBB->succ_begin(), SE = MBB->succ_end(); SI != SE; ++SI) {
   724:     MachineBasicBlock *SB = *SI;
   725:     for (auto I = SB->livein_begin(), E = SB->livein_end(); I != E; ++I) {
   726:       unsigned R = (*I).PhysReg;
   727:       if (LiveOuts[R])
   728:         continue;
   729:       LiveOuts.set(R);
   730:       Changed = true;
   731:     }
   732:   }
   733:   return Changed;
   734: }
   735: 
   736: bool HexagonLiveVariablesImpl::updateLocalLiveness(MachineFunction &Fn) {
   737:   LLVM_DEBUG(dbgs() << "\n[updateLocalLiveness]");
   738:   for (MachineFunction::iterator B = Fn.begin(), E = Fn.end(); B != E; ++B)
   739:     updateLocalLiveness(&*B, false);
   740:   return true;
   741: }
   742: 
   743: bool HexagonLiveVariablesImpl::updateLocalLiveness(MachineBasicBlock *MBB,
   744:                                                    bool UpdateBundle) {
   745:   assert(MBB && "Invalid basic block");
   746:   LLVM_DEBUG(dbgs() << "\n[updateLocalLiveness] MBB#" << MBB->getNumber());
   747: 
   748:   BitVector &LiveOut = MBBLiveOuts[MBB];
   749:   updateLiveOuts(MBB, LiveOut);
   750: 
   751:   BitVector Used = LiveOut;
   752:   SmallVector<MachineInstr *, 2> BundleHeads;
   753:   // Bottom up traversal of MBB.
   754:   for (MachineBasicBlock::reverse_instr_iterator MII = MBB->instr_rbegin(),
   755:                                                  MIREnd = MBB->instr_rend();
   756:        MII != MIREnd; ++MII) {
   757:     MachineInstr *MI = &*MII;
   758:     // The bundle liveness is updated differently.
   759:     if (MI->isBundle()) {
   760:       if (UpdateBundle)
   761:         BundleHeads.push_back(MI);
   762:       continue;
   763:     }
   764:     if (MI->isDebugInstr()) // DBG_VALUE may have invalid reg.
   765:       continue;
   766:     SmallVector<MachineOperand *, 4> UseRegs;
   767:     SmallVector<MachineOperand *, 2> DefRegs;
   768:     for (unsigned i = 0; i < MI->getNumOperands(); ++i) {
   769:       MachineOperand &MO = MI->getOperand(i);
   770:       if (MO.isReg()) { // DBG_VALUE may have invalid reg.
   771:         if (MO.isUse())
   772:           UseRegs.push_back(&MO);
   773:         else { // Def
   774:           if (!QII->isPredicated(*MI) && !MI->isKill()) {
   775:             // Assuming that predicated defs are not defs, for now.
   776:             // KILL instructions are no-ops
   777:             DefRegs.push_back(&MO);
   778:           }
   779:         }
   780:       } else if (MO.isRegMask()) {
   781:         if (!QII->isPredicated(*MI))
   782:           DefRegs.push_back(&MO);
   783:       }
   784:     }
   785:     // In case of a def. remove Reg and its sub-regs from Used list
   786:     // such that uses in the same MI can be marked as kill.
   787:     auto RemoveDef = [&](unsigned Reg, bool Implicit) -> void {
   788:       for (MCSubRegIterator SI(Reg, TRI, true); SI.isValid(); ++SI) {
   789:         Used.reset(*SI);
   790:         if (Implicit) {
   791:           // For implicit defs, check if there is an implicit use of an
   792:           // aliased register. If so, mark the aliased reg as used.
   793:           for (auto *UseOp : UseRegs)
   794:             if (UseOp->isImplicit() && TRI->regsOverlap(*SI, UseOp->getReg()))
   795:               Used.set(UseOp->getReg());
   796:         }
   797:       }
   798:     };
   799:     for (unsigned i = 0; i < DefRegs.size(); ++i) {
   800:       MachineOperand &MO = *DefRegs[i];
```
- EN: It declares or implements routines such as reset, find_next, LLVM_DEBUG, addLiveIn, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLiveVariablesImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 reset, find_next, LLVM_DEBUG, addLiveIn, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariablesImpl，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801:       if (MO.isReg()) {
   802:         RemoveDef(MO.getReg(), MO.isImplicit());
   803:       } else if (MO.isRegMask()) {
   804:         for (unsigned R = 1, NR = TRI->getNumRegs(); R != NR; ++R)
   805:           if (MO.clobbersPhysReg(R))
   806:             RemoveDef(R, true);
   807:       }
   808:     }
   809:     // The order is important as we are looking from right to left.
   810:     for (unsigned i = UseRegs.size(); i > 0;) {
   811:       --i;
   812:       unsigned UseReg = UseRegs[i]->getReg();
   813:       bool Killed = true;
   814:       for (MCRegAliasIterator AI(UseReg, TRI, true); AI.isValid(); ++AI) {
   815:         if (Used[*AI])
   816:           Killed = false;
   817:       }
   818:       Used.set(UseReg);
   819:       if (Killed && !UseRegs[i]->isDebug())
   820:         UseRegs[i]->setIsKill(true);
   821:     }
   822:   }
   823:   // Recreates bundle for updating liveness.
   824:   for (SmallVectorImpl<MachineInstr *>::iterator MII = BundleHeads.begin();
   825:        MII != BundleHeads.end(); ++MII) {
   826:     MachineInstr *MI = *MII;
   827:     assert(MI && "Invalid bundle head");
   828:     assert(MI->isBundle() && "Expected a bundle head instruction");
   829:     assert(MI->getParent() == MBB && "Bundle head not in expected block");
   830:     MachineBasicBlock::instr_iterator BS = MI->getIterator();
   831:     MachineBasicBlock::instr_iterator BE = getBundleEnd(BS);
   832:     for (++BS; BS != BE; ++BS)
   833:       // Remove from bundle so that BUNDLE head can be erased.
   834:       BS->unbundleFromPred();
   835: 
   836:     BS = MI->getIterator();
   837:     ++BS;
   838:     bool memShufDisabled = QII->getBundleNoShuf(*MI);
   839:     MI->eraseFromParent();
   840:     finalizeBundle(*MBB, BS, BE);
   841:     MachineBasicBlock::instr_iterator BundleMII = std::prev(BS);
   842:     if (memShufDisabled)
   843:       QII->setBundleNoShuf(BundleMII);
   844:   }
   845:   return true;
   846: }
   847: 
   848: // It deletes the live-in of the \p From MBB.
   849: bool HexagonLiveVariablesImpl::incrementalUpdate(MICInstIterType MIDelta,
   850:                                                  MachineBasicBlock *From,
   851:                                                  MachineBasicBlock *To) {
   852:   while (!From->livein_empty())
   853:     From->removeLiveIn((*From->livein_begin()).PhysReg);
   854:   // Handle MI use-def of From.
   855:   constructUseDef(From);
   856:   // Handle MI use-def of To.
   857:   constructUseDef(To);
   858:   // Calculate live-in of From and To
   859:   // Reuse this by setting all MBBs except From and To as visited.
   860:   updateGlobalLiveness(From, To);
   861:   // Update local liveness of To.
   862:   updateLocalLiveness(From, true);
   863:   updateLocalLiveness(To, true);
   864: 
   865:   // Do this after the liveness update because MIDelta might not be in the
   866:   // MIUseDefs before liveness update (since MIDelta might be newly inserted).
   867:   MIUseDef_t::const_iterator MIUseDef = MIUseDefs.find(&*MIDelta);
   868:   if (MIUseDef == MIUseDefs.end())
   869:     llvm_unreachable("MIDelta not found in MIUseDefs after liveness update");
   870:   const BitVector &Defs = MIUseDef->second.second;
   871:   int Reg = Defs.find_first();
   872:   // Adding all the defs as live-ins. This is conservative approach but we
   873:   // need to add them so as to avoid dealing with callee saved registers and
   874:   // any unwanted errors in liveness that might arise.
   875:   while (Reg >= 0) {
   876:     From->addLiveIn(Reg);
   877:     Reg = Defs.find_next(Reg);
   878:   }
   879:   return true;
   880: }
   881: 
   882: void HexagonLiveVariablesImpl::addNewMBB(MachineBasicBlock *MBB) {
   883:   // Resize and init.
   884:   constructUseDef(MBB); // This is to set up some containers for MBB.
   885:   gatherBlocksDF(*MBB->getParent(), &BlocksDepthFirst);
   886:   updateGlobalLiveness(MBB, MBB);
   887: }
   888: 
   889: // TODO: This is a slow implementation because constructUseDef destroys
   890: // the MBBLiveOuts which is generated again by updateGlobalLiveness.
   891: void HexagonLiveVariablesImpl::addNewMI(MachineInstr *MI,
   892:                                         MachineBasicBlock *MBB) {
   893:   constructUseDef(MBB); // This is to set up some containers for MBB.
   894:   updateGlobalLiveness(MBB, MBB);
   895: }
   896: 
   897: void HexagonLiveVariablesImpl::generateDistanceMap(const MachineFunction &Fn) {
   898:   assert(DistanceMap.empty() && "DistanceMap not empty, first clear!");
   899:   for (MachineFunction::const_iterator MBBI = Fn.begin(), E = Fn.end();
   900:        MBBI != E; ++MBBI) {
```
- EN: It declares or implements routines such as RemoveDef, getReg, isValid, set, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLiveVariablesImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 RemoveDef, getReg, isValid, set, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLiveVariablesImpl，说明了它与同级后端组件的连接关系。

### Lines 901-911 / 第 901-911 行

```cpp
   901:     const MachineBasicBlock *MBB = &*MBBI;
   902:     unsigned MBBInsSize = 0;
   903:     for (MachineBasicBlock::const_instr_iterator MII = MBB->instr_begin(),
   904:                                                  E = MBB->instr_end();
   905:          MII != E; ++MII) {
   906:       const MachineInstr *MI = &*MII;
   907:       MBBInsSize += QII->getSize(*MI);
   908:     }
   909:     DistanceMap[MBB] = MBBInsSize;
   910:   }
   911: }
```
- EN: It declares or implements routines such as getSize, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getSize 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- generated metadata tables / 生成的元数据表

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonLiveVariables.h, HexagonTargetMachine.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachinePostDominators.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/InitializePasses.h, llvm/Support/Debug.h, llvm/Support/ErrorHandling.h`
- Hexagon symbols / Hexagon 符号: `HexagonLiveVariables, HexagonTargetMachine, HexagonLiveVariablesID, HexagonLiveVariablesImpl, HexagonInstrInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
