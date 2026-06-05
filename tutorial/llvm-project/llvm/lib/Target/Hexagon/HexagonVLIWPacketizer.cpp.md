# HexagonVLIWPacketizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonVLIWPacketizer.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon VLIW packet formation and packetization constraints.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及调度或成包。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonPacketizer.cpp - VLIW packetizer ----------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements a simple VLIW packetizer using DFA. The packetizer works on
    10: // machine basic blocks. For each instruction I in BB, the packetizer consults
    11: // the DFA to see if machine resources are available to execute I. If so, the
    12: // packetizer checks if I depends on any instruction J in the current packet.
    13: // If no dependency is found, I is added to current packet and machine resource
    14: // is marked as taken. If any dependency is found, a target API call is made to
    15: // prune the dependence.
    16: //
    17: //===----------------------------------------------------------------------===//
    18: 
    19: #include "HexagonVLIWPacketizer.h"
    20: #include "Hexagon.h"
    21: #include "HexagonInstrInfo.h"
    22: #include "HexagonRegisterInfo.h"
    23: #include "HexagonSubtarget.h"
    24: #include "llvm/ADT/BitVector.h"
    25: #include "llvm/ADT/DenseSet.h"
    26: #include "llvm/ADT/STLExtras.h"
    27: #include "llvm/ADT/StringExtras.h"
    28: #include "llvm/Analysis/AliasAnalysis.h"
    29: #include "llvm/CodeGen/MachineBasicBlock.h"
    30: #include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
    31: #include "llvm/CodeGen/MachineDominators.h"
    32: #include "llvm/CodeGen/MachineFrameInfo.h"
    33: #include "llvm/CodeGen/MachineFunction.h"
    34: #include "llvm/CodeGen/MachineFunctionPass.h"
    35: #include "llvm/CodeGen/MachineInstr.h"
    36: #include "llvm/CodeGen/MachineInstrBundle.h"
    37: #include "llvm/CodeGen/MachineLoopInfo.h"
    38: #include "llvm/CodeGen/MachineOperand.h"
    39: #include "llvm/CodeGen/ScheduleDAG.h"
    40: #include "llvm/CodeGen/TargetRegisterInfo.h"
    41: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    42: #include "llvm/IR/DebugLoc.h"
    43: #include "llvm/InitializePasses.h"
    44: #include "llvm/MC/MCInstrDesc.h"
    45: #include "llvm/Pass.h"
    46: #include "llvm/Support/CommandLine.h"
    47: #include "llvm/Support/Debug.h"
    48: #include "llvm/Support/ErrorHandling.h"
    49: #include "llvm/Support/raw_ostream.h"
    50: #include <cassert>
    51: #include <cstdint>
    52: #include <iterator>
    53: 
    54: using namespace llvm;
    55: 
    56: #define DEBUG_TYPE "packets"
    57: 
    58: cl::opt<bool> DisablePacketizer("disable-packetizer", cl::Hidden,
    59:                                 cl::desc("Disable Hexagon packetizer pass"));
    60: 
    61: static cl::opt<bool> Slot1Store("slot1-store-slot0-load", cl::Hidden,
    62:                                 cl::init(true),
    63:                                 cl::desc("Allow slot1 store and slot0 load"));
    64: 
    65: static cl::opt<bool> PacketizeVolatiles(
    66:     "hexagon-packetize-volatiles", cl::Hidden, cl::init(true),
    67:     cl::desc("Allow non-solo packetization of volatile memory references"));
    68: 
    69: static cl::opt<bool>
    70:     EnableGenAllInsnClass("enable-gen-insn", cl::Hidden,
    71:                           cl::desc("Generate all instruction with TC"));
    72: 
    73: static cl::opt<bool>
    74:     DisableVecDblNVStores("disable-vecdbl-nv-stores", cl::Hidden,
    75:                           cl::desc("Disable vector double new-value-stores"));
    76: 
    77: extern cl::opt<bool> ScheduleInlineAsm;
    78: 
    79: namespace {
    80: 
    81:   class HexagonPacketizer : public MachineFunctionPass {
    82:   public:
    83:     static char ID;
    84: 
    85:     HexagonPacketizer(bool Min = false)
    86:       : MachineFunctionPass(ID), Minimal(Min) {}
    87: 
    88:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    89:       AU.setPreservesCFG();
    90:       AU.addRequired<AAResultsWrapperPass>();
    91:       AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();
    92:       AU.addRequired<MachineDominatorTreeWrapperPass>();
    93:       AU.addRequired<MachineLoopInfoWrapperPass>();
    94:       AU.addPreserved<MachineDominatorTreeWrapperPass>();
    95:       AU.addPreserved<MachineLoopInfoWrapperPass>();
    96:       MachineFunctionPass::getAnalysisUsage(AU);
    97:     }
    98: 
    99:     StringRef getPassName() const override { return "Hexagon Packetizer"; }
   100:     bool runOnMachineFunction(MachineFunction &Fn) override;
```
- EN: It imports headers such as HexagonVLIWPacketizer.h, Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, ... (34 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonPacketizer, which carry the state or API of this component.
- CN: 这里引入了 HexagonVLIWPacketizer.h, Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, ... (34 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonPacketizer 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101: 
   102:     MachineFunctionProperties getRequiredProperties() const override {
   103:       return MachineFunctionProperties().setNoVRegs();
   104:     }
   105: 
   106:   private:
   107:     const HexagonInstrInfo *HII = nullptr;
   108:     const HexagonRegisterInfo *HRI = nullptr;
   109:     const bool Minimal = false;
   110:   };
   111: 
   112: } // end anonymous namespace
   113: 
   114: char HexagonPacketizer::ID = 0;
   115: 
   116: INITIALIZE_PASS_BEGIN(HexagonPacketizer, "hexagon-packetizer",
   117:                       "Hexagon Packetizer", false, false)
   118: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   119: INITIALIZE_PASS_DEPENDENCY(MachineBranchProbabilityInfoWrapperPass)
   120: INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
   121: INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
   122: INITIALIZE_PASS_END(HexagonPacketizer, "hexagon-packetizer",
   123:                     "Hexagon Packetizer", false, false)
   124: 
   125: HexagonPacketizerList::HexagonPacketizerList(MachineFunction &MF,
   126:       MachineLoopInfo &MLI, AAResults *AA,
   127:       const MachineBranchProbabilityInfo *MBPI, bool Minimal)
   128:     : VLIWPacketizerList(MF, MLI, AA), MBPI(MBPI), MLI(&MLI),
   129:       Minimal(Minimal) {
   130:   HII = MF.getSubtarget<HexagonSubtarget>().getInstrInfo();
   131:   HRI = MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   132: 
   133:   addMutation(std::make_unique<HexagonSubtarget::UsrOverflowMutation>());
   134:   addMutation(std::make_unique<HexagonSubtarget::HVXMemLatencyMutation>());
   135:   addMutation(std::make_unique<HexagonSubtarget::BankConflictMutation>());
   136: }
   137: 
   138: // Check if FirstI modifies a register that SecondI reads.
   139: static bool hasWriteToReadDep(const MachineInstr &FirstI,
   140:                               const MachineInstr &SecondI,
   141:                               const TargetRegisterInfo *TRI) {
   142:   for (auto &MO : FirstI.operands()) {
   143:     if (!MO.isReg() || !MO.isDef())
   144:       continue;
   145:     Register R = MO.getReg();
   146:     if (SecondI.readsRegister(R, TRI))
   147:       return true;
   148:   }
   149:   return false;
   150: }
   151: 
   152: 
   153: static MachineBasicBlock::iterator moveInstrOut(MachineInstr &MI,
   154:       MachineBasicBlock::iterator BundleIt, bool Before) {
   155:   MachineBasicBlock::instr_iterator InsertPt;
   156:   if (Before)
   157:     InsertPt = BundleIt.getInstrIterator();
   158:   else
   159:     InsertPt = std::next(BundleIt).getInstrIterator();
   160: 
   161:   MachineBasicBlock &B = *MI.getParent();
   162:   // The instruction should at least be bundled with the preceding instruction
   163:   // (there will always be one, i.e. BUNDLE, if nothing else).
   164:   assert(MI.isBundledWithPred());
   165:   if (MI.isBundledWithSucc()) {
   166:     MI.clearFlag(MachineInstr::BundledSucc);
   167:     MI.clearFlag(MachineInstr::BundledPred);
   168:   } else {
   169:     // If it's not bundled with the successor (i.e. it is the last one
   170:     // in the bundle), then we can simply unbundle it from the predecessor,
   171:     // which will take care of updating the predecessor's flag.
   172:     MI.unbundleFromPred();
   173:   }
   174:   B.splice(InsertPt, &B, MI.getIterator());
   175: 
   176:   // Get the size of the bundle without asserting.
   177:   MachineBasicBlock::const_instr_iterator I = BundleIt.getInstrIterator();
   178:   MachineBasicBlock::const_instr_iterator E = B.instr_end();
   179:   unsigned Size = 0;
   180:   for (++I; I != E && I->isBundledWithPred(); ++I)
   181:     ++Size;
   182: 
   183:   // If there are still two or more instructions, then there is nothing
   184:   // else to be done.
   185:   if (Size > 1)
   186:     return BundleIt;
   187: 
   188:   // Otherwise, extract the single instruction out and delete the bundle.
   189:   MachineBasicBlock::iterator NextIt = std::next(BundleIt);
   190:   MachineInstr &SingleI = *BundleIt->getNextNode();
   191:   SingleI.unbundleFromPred();
   192:   assert(!SingleI.isBundledWithSucc());
   193:   BundleIt->eraseFromParent();
   194:   return NextIt;
   195: }
   196: 
   197: bool HexagonPacketizer::runOnMachineFunction(MachineFunction &MF) {
   198:   // FIXME: This pass causes verification failures.
   199:   MF.getProperties().setFailsVerification();
   200: 
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as getRequiredProperties, MachineFunctionProperties, INITIALIZE_PASS_BEGIN, getSubtarget<HexagonSubtarget>, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 getRequiredProperties, MachineFunctionProperties, INITIALIZE_PASS_BEGIN, getSubtarget<HexagonSubtarget>, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-300 / 第 201-300 行

```cpp
   201:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
   202:   HII = HST.getInstrInfo();
   203:   HRI = HST.getRegisterInfo();
   204:   auto &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
   205:   auto *AA = &getAnalysis<AAResultsWrapperPass>().getAAResults();
   206:   auto *MBPI =
   207:       &getAnalysis<MachineBranchProbabilityInfoWrapperPass>().getMBPI();
   208: 
   209:   if (EnableGenAllInsnClass)
   210:     HII->genAllInsnTimingClasses(MF);
   211: 
   212:   // Instantiate the packetizer.
   213:   bool MinOnly = Minimal || DisablePacketizer || !HST.usePackets() ||
   214:                  skipFunction(MF.getFunction());
   215:   HexagonPacketizerList Packetizer(MF, MLI, AA, MBPI, MinOnly);
   216: 
   217:   // DFA state table should not be empty.
   218:   assert(Packetizer.getResourceTracker() && "Empty DFA table!");
   219: 
   220:   // Loop over all basic blocks and remove KILL pseudo-instructions
   221:   // These instructions confuse the dependence analysis. Consider:
   222:   // D0 = ...   (Insn 0)
   223:   // R0 = KILL R0, D0 (Insn 1)
   224:   // R0 = ... (Insn 2)
   225:   // Here, Insn 1 will result in the dependence graph not emitting an output
   226:   // dependence between Insn 0 and Insn 2. This can lead to incorrect
   227:   // packetization
   228:   for (MachineBasicBlock &MB : MF) {
   229:     for (MachineInstr &MI : llvm::make_early_inc_range(MB))
   230:       if (MI.isKill())
   231:         MB.erase(&MI);
   232:   }
   233: 
   234:   // TinyCore with Duplexes: Translate to big-instructions.
   235:   if (HST.isTinyCoreWithDuplex())
   236:     HII->translateInstrsForDup(MF, true);
   237: 
   238:   // Loop over all of the basic blocks.
   239:   for (auto &MB : MF) {
   240:     auto Begin = MB.begin(), End = MB.end();
   241:     while (Begin != End) {
   242:       // Find the first non-boundary starting from the end of the last
   243:       // scheduling region.
   244:       MachineBasicBlock::iterator RB = Begin;
   245:       while (RB != End && HII->isSchedulingBoundary(*RB, &MB, MF))
   246:         ++RB;
   247:       // Find the first boundary starting from the beginning of the new
   248:       // region.
   249:       MachineBasicBlock::iterator RE = RB;
   250:       while (RE != End && !HII->isSchedulingBoundary(*RE, &MB, MF))
   251:         ++RE;
   252:       // Add the scheduling boundary if it's not block end.
   253:       if (RE != End)
   254:         ++RE;
   255:       // If RB == End, then RE == End.
   256:       if (RB != End)
   257:         Packetizer.PacketizeMIs(&MB, RB, RE);
   258: 
   259:       Begin = RE;
   260:     }
   261:   }
   262: 
   263:   // TinyCore with Duplexes: Translate to tiny-instructions.
   264:   if (HST.isTinyCoreWithDuplex())
   265:     HII->translateInstrsForDup(MF, false);
   266: 
   267:   Packetizer.unpacketizeSoloInstrs(MF);
   268:   return true;
   269: }
   270: 
   271: // Reserve resources for a constant extender. Trigger an assertion if the
   272: // reservation fails.
   273: void HexagonPacketizerList::reserveResourcesForConstExt() {
   274:   if (!tryAllocateResourcesForConstExt(true))
   275:     llvm_unreachable("Resources not available");
   276: }
   277: 
   278: bool HexagonPacketizerList::canReserveResourcesForConstExt() {
   279:   return tryAllocateResourcesForConstExt(false);
   280: }
   281: 
   282: // Allocate resources (i.e. 4 bytes) for constant extender. If succeeded,
   283: // return true, otherwise, return false.
   284: bool HexagonPacketizerList::tryAllocateResourcesForConstExt(bool Reserve) {
   285:   auto *ExtMI = MF.CreateMachineInstr(HII->get(Hexagon::A4_ext), DebugLoc());
   286:   bool Avail = ResourceTracker->canReserveResources(*ExtMI);
   287:   if (Reserve && Avail)
   288:     ResourceTracker->reserveResources(*ExtMI);
   289:   MF.deleteMachineInstr(ExtMI);
   290:   return Avail;
   291: }
   292: 
   293: bool HexagonPacketizerList::isCallDependent(const MachineInstr &MI,
   294:       SDep::Kind DepType, unsigned DepReg) {
   295:   // Check for LR dependence.
   296:   if (DepReg == HRI->getRARegister())
   297:     return true;
   298: 
   299:   if (HII->isDeallocRet(MI))
   300:     if (DepReg == HRI->getFrameRegister() || DepReg == HRI->getStackRegister())
```
- EN: It declares or implements routines such as getSubtarget<HexagonSubtarget>, getInstrInfo, getRegisterInfo, getAnalysis<MachineLoopInfoWrapperPass>, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget<HexagonSubtarget>, getInstrInfo, getRegisterInfo, getAnalysis<MachineLoopInfoWrapperPass>, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:       return true;
   302: 
   303:   // Call-like instructions can be packetized with preceding instructions
   304:   // that define registers implicitly used or modified by the call. Explicit
   305:   // uses are still prohibited, as in the case of indirect calls:
   306:   //   r0 = ...
   307:   //   J2_jumpr r0
   308:   if (DepType == SDep::Data) {
   309:     for (const MachineOperand &MO : MI.operands())
   310:       if (MO.isReg() && MO.getReg() == DepReg && !MO.isImplicit())
   311:         return true;
   312:   }
   313: 
   314:   return false;
   315: }
   316: 
   317: static bool isRegDependence(const SDep::Kind DepType) {
   318:   return DepType == SDep::Data || DepType == SDep::Anti ||
   319:          DepType == SDep::Output;
   320: }
   321: 
   322: static bool isDirectJump(const MachineInstr &MI) {
   323:   return MI.getOpcode() == Hexagon::J2_jump;
   324: }
   325: 
   326: static bool isSchedBarrier(const MachineInstr &MI) {
   327:   switch (MI.getOpcode()) {
   328:   case Hexagon::Y2_barrier:
   329:     return true;
   330:   }
   331:   return false;
   332: }
   333: 
   334: static bool isControlFlow(const MachineInstr &MI) {
   335:   return MI.getDesc().isTerminator() || MI.getDesc().isCall();
   336: }
   337: 
   338: /// Returns true if the instruction modifies a callee-saved register.
   339: static bool doesModifyCalleeSavedReg(const MachineInstr &MI,
   340:                                      const TargetRegisterInfo *TRI) {
   341:   const MachineFunction &MF = *MI.getParent()->getParent();
   342:   for (auto *CSR = TRI->getCalleeSavedRegs(&MF); CSR && *CSR; ++CSR)
   343:     if (MI.modifiesRegister(*CSR, TRI))
   344:       return true;
   345:   return false;
   346: }
   347: 
   348: // Returns true if an instruction can be promoted to .new predicate or
   349: // new-value store.
   350: bool HexagonPacketizerList::isNewifiable(const MachineInstr &MI,
   351:       const TargetRegisterClass *NewRC) {
   352:   // Vector stores can be predicated, and can be new-value stores, but
   353:   // they cannot be predicated on a .new predicate value.
   354:   if (NewRC == &Hexagon::PredRegsRegClass) {
   355:     if (HII->isHVXVec(MI) && MI.mayStore())
   356:       return false;
   357:     return HII->isPredicated(MI) && HII->getDotNewPredOp(MI, nullptr) > 0;
   358:   }
   359:   // If the class is not PredRegs, it could only apply to new-value stores.
   360:   return HII->mayBeNewStore(MI);
   361: }
   362: 
   363: // Promote an instructiont to its .cur form.
   364: // At this time, we have already made a call to canPromoteToDotCur and made
   365: // sure that it can *indeed* be promoted.
   366: bool HexagonPacketizerList::promoteToDotCur(MachineInstr &MI,
   367:       SDep::Kind DepType, MachineBasicBlock::iterator &MII,
   368:       const TargetRegisterClass* RC) {
   369:   assert(DepType == SDep::Data);
   370:   int CurOpcode = HII->getDotCurOp(MI);
   371:   MI.setDesc(HII->get(CurOpcode));
   372:   return true;
   373: }
   374: 
   375: void HexagonPacketizerList::cleanUpDotCur() {
   376:   MachineInstr *MI = nullptr;
   377:   for (auto *BI : CurrentPacketMIs) {
   378:     LLVM_DEBUG(dbgs() << "Cleanup packet has "; BI->dump(););
   379:     if (HII->isDotCurInst(*BI)) {
   380:       MI = BI;
   381:       continue;
   382:     }
   383:     if (MI) {
   384:       for (auto &MO : BI->operands())
   385:         if (MO.isReg() && MO.getReg() == MI->getOperand(0).getReg())
   386:           return;
   387:     }
   388:   }
   389:   if (!MI)
   390:     return;
   391:   // We did not find a use of the CUR, so de-cur it.
   392:   MI->setDesc(HII->get(HII->getNonDotCurOp(*MI)));
   393:   LLVM_DEBUG(dbgs() << "Demoted CUR "; MI->dump(););
   394: }
   395: 
   396: // Check to see if an instruction can be dot cur.
   397: bool HexagonPacketizerList::canPromoteToDotCur(const MachineInstr &MI,
   398:       const SUnit *PacketSU, unsigned DepReg, MachineBasicBlock::iterator &MII,
   399:       const TargetRegisterClass *RC) {
   400:   if (!HII->isHVXVec(MI))
```
- EN: It declares or implements routines such as isRegDependence, isDirectJump, isSchedBarrier, isControlFlow, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isRegDependence, isDirectJump, isSchedBarrier, isControlFlow, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:     return false;
   402:   if (!HII->isHVXVec(*MII))
   403:     return false;
   404: 
   405:   // Already a dot new instruction.
   406:   if (HII->isDotCurInst(MI) && !HII->mayBeCurLoad(MI))
   407:     return false;
   408: 
   409:   if (!HII->mayBeCurLoad(MI))
   410:     return false;
   411: 
   412:   // The "cur value" cannot come from inline asm.
   413:   if (PacketSU->getInstr()->isInlineAsm())
   414:     return false;
   415: 
   416:   // Make sure candidate instruction uses cur.
   417:   LLVM_DEBUG(dbgs() << "Can we DOT Cur Vector MI\n"; MI.dump();
   418:              dbgs() << "in packet\n";);
   419:   MachineInstr &MJ = *MII;
   420:   LLVM_DEBUG({
   421:     dbgs() << "Checking CUR against ";
   422:     MJ.dump();
   423:   });
   424:   Register DestReg = MI.getOperand(0).getReg();
   425:   bool FoundMatch = false;
   426:   for (auto &MO : MJ.operands())
   427:     if (MO.isReg() && MO.getReg() == DestReg)
   428:       FoundMatch = true;
   429:   if (!FoundMatch)
   430:     return false;
   431: 
   432:   // Check for existing uses of a vector register within the packet which
   433:   // would be affected by converting a vector load into .cur format.
   434:   for (auto *BI : CurrentPacketMIs) {
   435:     LLVM_DEBUG(dbgs() << "packet has "; BI->dump(););
   436:     if (BI->readsRegister(DepReg, MF.getSubtarget().getRegisterInfo()))
   437:       return false;
   438:   }
   439: 
   440:   LLVM_DEBUG(dbgs() << "Can Dot CUR MI\n"; MI.dump(););
   441:   // We can convert the opcode into a .cur.
   442:   return true;
   443: }
   444: 
   445: // Promote an instruction to its .new form. At this time, we have already
   446: // made a call to canPromoteToDotNew and made sure that it can *indeed* be
   447: // promoted.
   448: bool HexagonPacketizerList::promoteToDotNew(MachineInstr &MI,
   449:       SDep::Kind DepType, MachineBasicBlock::iterator &MII,
   450:       const TargetRegisterClass* RC) {
   451:   assert(DepType == SDep::Data);
   452:   int NewOpcode;
   453:   if (RC == &Hexagon::PredRegsRegClass)
   454:     NewOpcode = HII->getDotNewPredOp(MI, MBPI);
   455:   else
   456:     NewOpcode = HII->getDotNewOp(MI);
   457:   MI.setDesc(HII->get(NewOpcode));
   458:   return true;
   459: }
   460: 
   461: bool HexagonPacketizerList::demoteToDotOld(MachineInstr &MI) {
   462:   int NewOpcode = HII->getDotOldOp(MI);
   463:   MI.setDesc(HII->get(NewOpcode));
   464:   return true;
   465: }
   466: 
   467: bool HexagonPacketizerList::useCallersSP(MachineInstr &MI) {
   468:   unsigned Opc = MI.getOpcode();
   469:   switch (Opc) {
   470:     case Hexagon::S2_storerd_io:
   471:     case Hexagon::S2_storeri_io:
   472:     case Hexagon::S2_storerh_io:
   473:     case Hexagon::S2_storerb_io:
   474:       break;
   475:     default:
   476:       llvm_unreachable("Unexpected instruction");
   477:   }
   478:   unsigned FrameSize = MF.getFrameInfo().getStackSize();
   479:   MachineOperand &Off = MI.getOperand(1);
   480:   int64_t NewOff = Off.getImm() - (FrameSize + HEXAGON_LRFP_SIZE);
   481:   if (HII->isValidOffset(Opc, NewOff, HRI)) {
   482:     Off.setImm(NewOff);
   483:     return true;
   484:   }
   485:   return false;
   486: }
   487: 
   488: void HexagonPacketizerList::useCalleesSP(MachineInstr &MI) {
   489:   unsigned Opc = MI.getOpcode();
   490:   switch (Opc) {
   491:     case Hexagon::S2_storerd_io:
   492:     case Hexagon::S2_storeri_io:
   493:     case Hexagon::S2_storerh_io:
   494:     case Hexagon::S2_storerb_io:
   495:       break;
   496:     default:
   497:       llvm_unreachable("Unexpected instruction");
   498:   }
   499:   unsigned FrameSize = MF.getFrameInfo().getStackSize();
   500:   MachineOperand &Off = MI.getOperand(1);
```
- EN: It declares or implements routines such as dump, getOperand, HexagonPacketizerList::promoteToDotNew, assert, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dump, getOperand, HexagonPacketizerList::promoteToDotNew, assert, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:   Off.setImm(Off.getImm() + FrameSize + HEXAGON_LRFP_SIZE);
   502: }
   503: 
   504: /// Return true if we can update the offset in MI so that MI and MJ
   505: /// can be packetized together.
   506: bool HexagonPacketizerList::updateOffset(SUnit *SUI, SUnit *SUJ) {
   507:   assert(SUI->getInstr() && SUJ->getInstr());
   508:   MachineInstr &MI = *SUI->getInstr();
   509:   MachineInstr &MJ = *SUJ->getInstr();
   510: 
   511:   unsigned BPI, OPI;
   512:   if (!HII->getBaseAndOffsetPosition(MI, BPI, OPI))
   513:     return false;
   514:   unsigned BPJ, OPJ;
   515:   if (!HII->getBaseAndOffsetPosition(MJ, BPJ, OPJ))
   516:     return false;
   517:   Register Reg = MI.getOperand(BPI).getReg();
   518:   if (Reg != MJ.getOperand(BPJ).getReg())
   519:     return false;
   520:   // Make sure that the dependences do not restrict adding MI to the packet.
   521:   // That is, ignore anti dependences, and make sure the only data dependence
   522:   // involves the specific register.
   523:   for (const auto &PI : SUI->Preds)
   524:     if (PI.getKind() != SDep::Anti &&
   525:         (PI.getKind() != SDep::Data || PI.getReg() != Reg))
   526:       return false;
   527:   int Incr;
   528:   if (!HII->getIncrementValue(MJ, Incr))
   529:     return false;
   530: 
   531:   int64_t Offset = MI.getOperand(OPI).getImm();
   532:   if (!HII->isValidOffset(MI.getOpcode(), Offset+Incr, HRI))
   533:     return false;
   534: 
   535:   MI.getOperand(OPI).setImm(Offset + Incr);
   536:   ChangedOffset = Offset;
   537:   return true;
   538: }
   539: 
   540: /// Undo the changed offset. This is needed if the instruction cannot be
   541: /// added to the current packet due to a different instruction.
   542: void HexagonPacketizerList::undoChangedOffset(MachineInstr &MI) {
   543:   unsigned BP, OP;
   544:   if (!HII->getBaseAndOffsetPosition(MI, BP, OP))
   545:     llvm_unreachable("Unable to find base and offset operands.");
   546:   MI.getOperand(OP).setImm(ChangedOffset);
   547: }
   548: 
   549: enum PredicateKind {
   550:   PK_False,
   551:   PK_True,
   552:   PK_Unknown
   553: };
   554: 
   555: /// Returns true if an instruction is predicated on p0 and false if it's
   556: /// predicated on !p0.
   557: static PredicateKind getPredicateSense(const MachineInstr &MI,
   558:                                        const HexagonInstrInfo *HII) {
   559:   if (!HII->isPredicated(MI))
   560:     return PK_Unknown;
   561:   if (HII->isPredicatedTrue(MI))
   562:     return PK_True;
   563:   return PK_False;
   564: }
   565: 
   566: static const MachineOperand &getPostIncrementOperand(const MachineInstr &MI,
   567:       const HexagonInstrInfo *HII) {
   568:   assert(HII->isPostIncrement(MI) && "Not a post increment operation.");
   569: #ifndef NDEBUG
   570:   // Post Increment means duplicates. Use dense map to find duplicates in the
   571:   // list. Caution: Densemap initializes with the minimum of 64 buckets,
   572:   // whereas there are at most 5 operands in the post increment.
   573:   DenseSet<unsigned> DefRegsSet;
   574:   for (auto &MO : MI.operands())
   575:     if (MO.isReg() && MO.isDef())
   576:       DefRegsSet.insert(MO.getReg());
   577: 
   578:   for (auto &MO : MI.operands())
   579:     if (MO.isReg() && MO.isUse() && DefRegsSet.count(MO.getReg()))
   580:       return MO;
   581: #else
   582:   if (MI.mayLoad()) {
   583:     const MachineOperand &Op1 = MI.getOperand(1);
   584:     // The 2nd operand is always the post increment operand in load.
   585:     assert(Op1.isReg() && "Post increment operand has be to a register.");
   586:     return Op1;
   587:   }
   588:   if (MI.getDesc().mayStore()) {
   589:     const MachineOperand &Op0 = MI.getOperand(0);
   590:     // The 1st operand is always the post increment operand in store.
   591:     assert(Op0.isReg() && "Post increment operand has be to a register.");
   592:     return Op0;
   593:   }
   594: #endif
   595:   // we should never come here.
   596:   llvm_unreachable("mayLoad or mayStore not set for Post Increment operation");
   597: }
   598: 
   599: // Get the value being stored.
   600: static const MachineOperand& getStoreValueOperand(const MachineInstr &MI) {
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares types such as PredicateKind, which carry the state or API of this component. It declares or implements routines such as setImm, HexagonPacketizerList::updateOffset, assert, getInstr, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明了 PredicateKind 等类型，用来承载该组件的状态或接口。 这里声明或实现了 setImm, HexagonPacketizerList::updateOffset, assert, getInstr, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 601-700 / 第 601-700 行

```cpp
   601:   // value being stored is always the last operand.
   602:   return MI.getOperand(MI.getNumOperands()-1);
   603: }
   604: 
   605: static bool isLoadAbsSet(const MachineInstr &MI) {
   606:   unsigned Opc = MI.getOpcode();
   607:   switch (Opc) {
   608:     case Hexagon::L4_loadrd_ap:
   609:     case Hexagon::L4_loadrb_ap:
   610:     case Hexagon::L4_loadrh_ap:
   611:     case Hexagon::L4_loadrub_ap:
   612:     case Hexagon::L4_loadruh_ap:
   613:     case Hexagon::L4_loadri_ap:
   614:       return true;
   615:   }
   616:   return false;
   617: }
   618: 
   619: static const MachineOperand &getAbsSetOperand(const MachineInstr &MI) {
   620:   assert(isLoadAbsSet(MI));
   621:   return MI.getOperand(1);
   622: }
   623: 
   624: // Can be new value store?
   625: // Following restrictions are to be respected in convert a store into
   626: // a new value store.
   627: // 1. If an instruction uses auto-increment, its address register cannot
   628: //    be a new-value register. Arch Spec 5.4.2.1
   629: // 2. If an instruction uses absolute-set addressing mode, its address
   630: //    register cannot be a new-value register. Arch Spec 5.4.2.1.
   631: // 3. If an instruction produces a 64-bit result, its registers cannot be used
   632: //    as new-value registers. Arch Spec 5.4.2.2.
   633: // 4. If the instruction that sets the new-value register is conditional, then
   634: //    the instruction that uses the new-value register must also be conditional,
   635: //    and both must always have their predicates evaluate identically.
   636: //    Arch Spec 5.4.2.3.
   637: // 5. There is an implied restriction that a packet cannot have another store,
   638: //    if there is a new value store in the packet. Corollary: if there is
   639: //    already a store in a packet, there can not be a new value store.
   640: //    Arch Spec: 3.4.4.2
   641: bool HexagonPacketizerList::canPromoteToNewValueStore(const MachineInstr &MI,
   642:       const MachineInstr &PacketMI, unsigned DepReg) {
   643:   // Make sure we are looking at the store, that can be promoted.
   644:   if (!HII->mayBeNewStore(MI))
   645:     return false;
   646: 
   647:   // Make sure there is dependency and can be new value'd.
   648:   const MachineOperand &Val = getStoreValueOperand(MI);
   649:   if (Val.isReg() && Val.getReg() != DepReg)
   650:     return false;
   651: 
   652:   const MCInstrDesc& MCID = PacketMI.getDesc();
   653: 
   654:   // First operand is always the result.
   655:   const TargetRegisterClass *PacketRC = HII->getRegClass(MCID, 0);
   656:   // Double regs can not feed into new value store: PRM section: 5.4.2.2.
   657:   if (PacketRC == &Hexagon::DoubleRegsRegClass)
   658:     return false;
   659: 
   660:   // New-value stores are of class NV (slot 0), dual stores require class ST
   661:   // in slot 0 (PRM 5.5).
   662:   for (auto *I : CurrentPacketMIs) {
   663:     SUnit *PacketSU = MIToSUnit.find(I)->second;
   664:     if (PacketSU->getInstr()->mayStore())
   665:       return false;
   666:   }
   667: 
   668:   // Make sure it's NOT the post increment register that we are going to
   669:   // new value.
   670:   if (HII->isPostIncrement(MI) &&
   671:       getPostIncrementOperand(MI, HII).getReg() == DepReg) {
   672:     return false;
   673:   }
   674: 
   675:   if (HII->isPostIncrement(PacketMI) && PacketMI.mayLoad() &&
   676:       getPostIncrementOperand(PacketMI, HII).getReg() == DepReg) {
   677:     // If source is post_inc, or absolute-set addressing, it can not feed
   678:     // into new value store
   679:     //   r3 = memw(r2++#4)
   680:     //   memw(r30 + #-1404) = r2.new -> can not be new value store
   681:     // arch spec section: 5.4.2.1.
   682:     return false;
   683:   }
   684: 
   685:   if (isLoadAbsSet(PacketMI) && getAbsSetOperand(PacketMI).getReg() == DepReg)
   686:     return false;
   687: 
   688:   // If the source that feeds the store is predicated, new value store must
   689:   // also be predicated.
   690:   if (HII->isPredicated(PacketMI)) {
   691:     if (!HII->isPredicated(MI))
   692:       return false;
   693: 
   694:     // Check to make sure that they both will have their predicates
   695:     // evaluate identically.
   696:     unsigned predRegNumSrc = 0;
   697:     unsigned predRegNumDst = 0;
   698:     const TargetRegisterClass* predRegClass = nullptr;
   699: 
   700:     // Get predicate register used in the source instruction.
```
- EN: It declares or implements routines such as getOperand, isLoadAbsSet, getOpcode, getAbsSetOperand, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, isLoadAbsSet, getOpcode, getAbsSetOperand, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:     for (auto &MO : PacketMI.operands()) {
   702:       if (!MO.isReg())
   703:         continue;
   704:       predRegNumSrc = MO.getReg();
   705:       predRegClass = HRI->getMinimalPhysRegClass(predRegNumSrc);
   706:       if (predRegClass == &Hexagon::PredRegsRegClass)
   707:         break;
   708:     }
   709:     assert((predRegClass == &Hexagon::PredRegsRegClass) &&
   710:         "predicate register not found in a predicated PacketMI instruction");
   711: 
   712:     // Get predicate register used in new-value store instruction.
   713:     for (auto &MO : MI.operands()) {
   714:       if (!MO.isReg())
   715:         continue;
   716:       predRegNumDst = MO.getReg();
   717:       predRegClass = HRI->getMinimalPhysRegClass(predRegNumDst);
   718:       if (predRegClass == &Hexagon::PredRegsRegClass)
   719:         break;
   720:     }
   721:     assert((predRegClass == &Hexagon::PredRegsRegClass) &&
   722:            "predicate register not found in a predicated MI instruction");
   723: 
   724:     // New-value register producer and user (store) need to satisfy these
   725:     // constraints:
   726:     // 1) Both instructions should be predicated on the same register.
   727:     // 2) If producer of the new-value register is .new predicated then store
   728:     // should also be .new predicated and if producer is not .new predicated
   729:     // then store should not be .new predicated.
   730:     // 3) Both new-value register producer and user should have same predicate
   731:     // sense, i.e, either both should be negated or both should be non-negated.
   732:     if (predRegNumDst != predRegNumSrc ||
   733:         HII->isDotNewInst(PacketMI) != HII->isDotNewInst(MI) ||
   734:         getPredicateSense(MI, HII) != getPredicateSense(PacketMI, HII))
   735:       return false;
   736:   }
   737: 
   738:   // Make sure that other than the new-value register no other store instruction
   739:   // register has been modified in the same packet. Predicate registers can be
   740:   // modified by they should not be modified between the producer and the store
   741:   // instruction as it will make them both conditional on different values.
   742:   // We already know this to be true for all the instructions before and
   743:   // including PacketMI. However, we need to perform the check for the
   744:   // remaining instructions in the packet.
   745: 
   746:   unsigned StartCheck = 0;
   747: 
   748:   for (auto *I : CurrentPacketMIs) {
   749:     SUnit *TempSU = MIToSUnit.find(I)->second;
   750:     MachineInstr &TempMI = *TempSU->getInstr();
   751: 
   752:     // Following condition is true for all the instructions until PacketMI is
   753:     // reached (StartCheck is set to 0 before the for loop).
   754:     // StartCheck flag is 1 for all the instructions after PacketMI.
   755:     if (&TempMI != &PacketMI && !StartCheck) // Start processing only after
   756:       continue;                              // encountering PacketMI.
   757: 
   758:     StartCheck = 1;
   759:     if (&TempMI == &PacketMI) // We don't want to check PacketMI for dependence.
   760:       continue;
   761: 
   762:     for (auto &MO : MI.operands())
   763:       if (MO.isReg() && TempSU->getInstr()->modifiesRegister(MO.getReg(), HRI))
   764:         return false;
   765:   }
   766: 
   767:   // Make sure that for non-POST_INC stores:
   768:   // 1. The only use of reg is DepReg and no other registers.
   769:   //    This handles base+index registers.
   770:   //    The following store can not be dot new.
   771:   //    Eg.   r0 = add(r0, #3)
   772:   //          memw(r1+r0<<#2) = r0
   773:   if (!HII->isPostIncrement(MI)) {
   774:     for (unsigned opNum = 0; opNum < MI.getNumOperands()-1; opNum++) {
   775:       const MachineOperand &MO = MI.getOperand(opNum);
   776:       if (MO.isReg() && MO.getReg() == DepReg)
   777:         return false;
   778:     }
   779:   }
   780: 
   781:   // If data definition is because of implicit definition of the register,
   782:   // do not newify the store. Eg.
   783:   // %r9 = ZXTH %r12, implicit %d6, implicit-def %r12
   784:   // S2_storerh_io %r8, 2, killed %r12; mem:ST2[%scevgep343]
   785:   for (auto &MO : PacketMI.operands()) {
   786:     if (MO.isRegMask() && MO.clobbersPhysReg(DepReg))
   787:       return false;
   788:     if (!MO.isReg() || !MO.isDef() || !MO.isImplicit())
   789:       continue;
   790:     Register R = MO.getReg();
   791:     if (R == DepReg || HRI->isSuperRegister(DepReg, R))
   792:       return false;
   793:   }
   794: 
   795:   // Handle imp-use of super reg case. There is a target independent side
   796:   // change that should prevent this situation but I am handling it for
   797:   // just-in-case. For example, we cannot newify R2 in the following case:
   798:   // %r3 = A2_tfrsi 0;
   799:   // S2_storeri_io killed %r0, 0, killed %r2, implicit killed %d1;
   800:   for (auto &MO : MI.operands()) {
```
- EN: It declares or implements routines such as getReg, getMinimalPhysRegClass, assert, getInstr, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getReg, getMinimalPhysRegClass, assert, getInstr, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-900 / 第 801-900 行

```cpp
   801:     if (MO.isReg() && MO.isUse() && MO.isImplicit() && MO.getReg() == DepReg)
   802:       return false;
   803:   }
   804: 
   805:   // Can be dot new store.
   806:   return true;
   807: }
   808: 
   809: // Can this MI to promoted to either new value store or new value jump.
   810: bool HexagonPacketizerList::canPromoteToNewValue(const MachineInstr &MI,
   811:       const SUnit *PacketSU, unsigned DepReg,
   812:       MachineBasicBlock::iterator &MII) {
   813:   if (!HII->mayBeNewStore(MI))
   814:     return false;
   815: 
   816:   // Check to see the store can be new value'ed.
   817:   MachineInstr &PacketMI = *PacketSU->getInstr();
   818:   if (canPromoteToNewValueStore(MI, PacketMI, DepReg))
   819:     return true;
   820: 
   821:   // Check to see the compare/jump can be new value'ed.
   822:   // This is done as a pass on its own. Don't need to check it here.
   823:   return false;
   824: }
   825: 
   826: static bool isImplicitDependency(const MachineInstr &I, bool CheckDef,
   827:       unsigned DepReg) {
   828:   for (auto &MO : I.operands()) {
   829:     if (CheckDef && MO.isRegMask() && MO.clobbersPhysReg(DepReg))
   830:       return true;
   831:     if (!MO.isReg() || MO.getReg() != DepReg || !MO.isImplicit())
   832:       continue;
   833:     if (CheckDef == MO.isDef())
   834:       return true;
   835:   }
   836:   return false;
   837: }
   838: 
   839: // Check to see if an instruction can be dot new.
   840: bool HexagonPacketizerList::canPromoteToDotNew(const MachineInstr &MI,
   841:       const SUnit *PacketSU, unsigned DepReg, MachineBasicBlock::iterator &MII,
   842:       const TargetRegisterClass* RC) {
   843:   // Already a dot new instruction.
   844:   if (HII->isDotNewInst(MI) && !HII->mayBeNewStore(MI))
   845:     return false;
   846: 
   847:   if (!isNewifiable(MI, RC))
   848:     return false;
   849: 
   850:   const MachineInstr &PI = *PacketSU->getInstr();
   851: 
   852:   // The "new value" cannot come from inline asm.
   853:   if (PI.isInlineAsm())
   854:     return false;
   855: 
   856:   // IMPLICIT_DEFs won't materialize as real instructions, so .new makes no
   857:   // sense.
   858:   if (PI.isImplicitDef())
   859:     return false;
   860: 
   861:   // If dependency is through an implicitly defined register, we should not
   862:   // newify the use.
   863:   if (isImplicitDependency(PI, true, DepReg) ||
   864:       isImplicitDependency(MI, false, DepReg))
   865:     return false;
   866: 
   867:   const MCInstrDesc& MCID = PI.getDesc();
   868:   const TargetRegisterClass *VecRC = HII->getRegClass(MCID, 0);
   869:   if (DisableVecDblNVStores && VecRC == &Hexagon::HvxWRRegClass)
   870:     return false;
   871: 
   872:   // predicate .new
   873:   if (RC == &Hexagon::PredRegsRegClass)
   874:     return HII->predCanBeUsedAsDotNew(PI, DepReg);
   875: 
   876:   if (RC != &Hexagon::PredRegsRegClass && !HII->mayBeNewStore(MI))
   877:     return false;
   878: 
   879:   // Create a dot new machine instruction to see if resources can be
   880:   // allocated. If not, bail out now.
   881:   int NewOpcode = (RC != &Hexagon::PredRegsRegClass) ? HII->getDotNewOp(MI) :
   882:     HII->getDotNewPredOp(MI, MBPI);
   883:   const MCInstrDesc &D = HII->get(NewOpcode);
   884:   MachineInstr *NewMI = MF.CreateMachineInstr(D, DebugLoc());
   885:   bool ResourcesAvailable = ResourceTracker->canReserveResources(*NewMI);
   886:   MF.deleteMachineInstr(NewMI);
   887:   if (!ResourcesAvailable)
   888:     return false;
   889: 
   890:   // New Value Store only. New Value Jump generated as a separate pass.
   891:   if (!canPromoteToNewValue(MI, PacketSU, DepReg, MII))
   892:     return false;
   893: 
   894:   return true;
   895: }
   896: 
   897: // Go through the packet instructions and search for an anti dependency between
   898: // them and DepReg from MI. Consider this case:
   899: // Trying to add
   900: // a) %r1 = TFRI_cdNotPt %p3, 2
```
- EN: It declares or implements routines such as HexagonPacketizerList::canPromoteToNewValue, getInstr, isImplicitDependency, HexagonPacketizerList::canPromoteToDotNew, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPacketizerList::canPromoteToNewValue, getInstr, isImplicitDependency, HexagonPacketizerList::canPromoteToDotNew, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901: // to this packet:
   902: // {
   903: //   b) %p0 = C2_or killed %p3, killed %p0
   904: //   c) %p3 = C2_tfrrp %r23
   905: //   d) %r1 = C2_cmovenewit %p3, 4
   906: //  }
   907: // The P3 from a) and d) will be complements after
   908: // a)'s P3 is converted to .new form
   909: // Anti-dep between c) and b) is irrelevant for this case
   910: bool HexagonPacketizerList::restrictingDepExistInPacket(MachineInstr &MI,
   911:                                                         unsigned DepReg) {
   912:   SUnit *PacketSUDep = MIToSUnit.find(&MI)->second;
   913: 
   914:   for (auto *I : CurrentPacketMIs) {
   915:     // We only care for dependencies to predicated instructions
   916:     if (!HII->isPredicated(*I))
   917:       continue;
   918: 
   919:     // Scheduling Unit for current insn in the packet
   920:     SUnit *PacketSU = MIToSUnit.find(I)->second;
   921: 
   922:     // Look at dependencies between current members of the packet and
   923:     // predicate defining instruction MI. Make sure that dependency is
   924:     // on the exact register we care about.
   925:     if (PacketSU->isSucc(PacketSUDep)) {
   926:       for (unsigned i = 0; i < PacketSU->Succs.size(); ++i) {
   927:         auto &Dep = PacketSU->Succs[i];
   928:         if (Dep.getSUnit() == PacketSUDep && Dep.getKind() == SDep::Anti &&
   929:             Dep.getReg() == DepReg)
   930:           return true;
   931:       }
   932:     }
   933:   }
   934: 
   935:   return false;
   936: }
   937: 
   938: /// Gets the predicate register of a predicated instruction.
   939: static unsigned getPredicatedRegister(MachineInstr &MI,
   940:                                       const HexagonInstrInfo *QII) {
   941:   /// We use the following rule: The first predicate register that is a use is
   942:   /// the predicate register of a predicated instruction.
   943:   assert(QII->isPredicated(MI) && "Must be predicated instruction");
   944: 
   945:   for (auto &Op : MI.operands()) {
   946:     if (Op.isReg() && Op.getReg() && Op.isUse() &&
   947:         Hexagon::PredRegsRegClass.contains(Op.getReg()))
   948:       return Op.getReg();
   949:   }
   950: 
   951:   llvm_unreachable("Unknown instruction operand layout");
   952:   return 0;
   953: }
   954: 
   955: // Given two predicated instructions, this function detects whether
   956: // the predicates are complements.
   957: bool HexagonPacketizerList::arePredicatesComplements(MachineInstr &MI1,
   958:                                                      MachineInstr &MI2) {
   959:   // If we don't know the predicate sense of the instructions bail out early, we
   960:   // need it later.
   961:   if (getPredicateSense(MI1, HII) == PK_Unknown ||
   962:       getPredicateSense(MI2, HII) == PK_Unknown)
   963:     return false;
   964: 
   965:   // Scheduling unit for candidate.
   966:   SUnit *SU = MIToSUnit[&MI1];
   967: 
   968:   // One corner case deals with the following scenario:
   969:   // Trying to add
   970:   // a) %r24 = A2_tfrt %p0, %r25
   971:   // to this packet:
   972:   // {
   973:   //   b) %r25 = A2_tfrf %p0, %r24
   974:   //   c) %p0 = C2_cmpeqi %r26, 1
   975:   // }
   976:   //
   977:   // On general check a) and b) are complements, but presence of c) will
   978:   // convert a) to .new form, and then it is not a complement.
   979:   // We attempt to detect it by analyzing existing dependencies in the packet.
   980: 
   981:   // Analyze relationships between all existing members of the packet.
   982:   // Look for Anti dependency on the same predicate reg as used in the
   983:   // candidate.
   984:   for (auto *I : CurrentPacketMIs) {
   985:     // Scheduling Unit for current insn in the packet.
   986:     SUnit *PacketSU = MIToSUnit.find(I)->second;
   987: 
   988:     // If this instruction in the packet is succeeded by the candidate...
   989:     if (PacketSU->isSucc(SU)) {
   990:       for (unsigned i = 0; i < PacketSU->Succs.size(); ++i) {
   991:         auto Dep = PacketSU->Succs[i];
   992:         // The corner case exist when there is true data dependency between
   993:         // candidate and one of current packet members, this dep is on
   994:         // predicate reg, and there already exist anti dep on the same pred in
   995:         // the packet.
   996:         if (Dep.getSUnit() == SU && Dep.getKind() == SDep::Data &&
   997:             Hexagon::PredRegsRegClass.contains(Dep.getReg())) {
   998:           // Here I know that I is predicate setting instruction with true
   999:           // data dep to candidate on the register we care about - c) in the
  1000:           // above example. Now I need to see if there is an anti dependency
```
- EN: It declares or implements routines such as HexagonPacketizerList::restrictingDepExistInPacket, size, getPredicatedRegister, assert, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPacketizerList::restrictingDepExistInPacket, size, getPredicatedRegister, assert, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:           // from c) to any other instruction in the same packet on the pred
  1002:           // reg of interest.
  1003:           if (restrictingDepExistInPacket(*I, Dep.getReg()))
  1004:             return false;
  1005:         }
  1006:       }
  1007:     }
  1008:   }
  1009: 
  1010:   // If the above case does not apply, check regular complement condition.
  1011:   // Check that the predicate register is the same and that the predicate
  1012:   // sense is different We also need to differentiate .old vs. .new: !p0
  1013:   // is not complementary to p0.new.
  1014:   unsigned PReg1 = getPredicatedRegister(MI1, HII);
  1015:   unsigned PReg2 = getPredicatedRegister(MI2, HII);
  1016:   return PReg1 == PReg2 &&
  1017:          Hexagon::PredRegsRegClass.contains(PReg1) &&
  1018:          Hexagon::PredRegsRegClass.contains(PReg2) &&
  1019:          getPredicateSense(MI1, HII) != getPredicateSense(MI2, HII) &&
  1020:          HII->isDotNewInst(MI1) == HII->isDotNewInst(MI2);
  1021: }
  1022: 
  1023: // Initialize packetizer flags.
  1024: void HexagonPacketizerList::initPacketizerState() {
  1025:   Dependence = false;
  1026:   PromotedToDotNew = false;
  1027:   GlueToNewValueJump = false;
  1028:   GlueAllocframeStore = false;
  1029:   FoundSequentialDependence = false;
  1030:   ChangedOffset = INT64_MAX;
  1031: }
  1032: 
  1033: // Ignore bundling of pseudo instructions.
  1034: bool HexagonPacketizerList::ignorePseudoInstruction(const MachineInstr &MI,
  1035:                                                     const MachineBasicBlock *) {
  1036:   if (MI.isDebugInstr())
  1037:     return true;
  1038: 
  1039:   if (MI.isCFIInstruction())
  1040:     return false;
  1041: 
  1042:   // We must print out inline assembly.
  1043:   if (MI.isInlineAsm())
  1044:     return false;
  1045: 
  1046:   if (MI.isImplicitDef())
  1047:     return false;
  1048: 
  1049:   // We check if MI has any functional units mapped to it. If it doesn't,
  1050:   // we ignore the instruction.
  1051:   const MCInstrDesc& TID = MI.getDesc();
  1052:   auto *IS = ResourceTracker->getInstrItins()->beginStage(TID.getSchedClass());
  1053:   return !IS->getUnits();
  1054: }
  1055: 
  1056: bool HexagonPacketizerList::isSoloInstruction(const MachineInstr &MI) {
  1057:   // Ensure any bundles created by gather packetize remain separate.
  1058:   if (MI.isBundle())
  1059:     return true;
  1060: 
  1061:   if (MI.isEHLabel() || MI.isCFIInstruction())
  1062:     return true;
  1063: 
  1064:   // Consider inline asm to not be a solo instruction by default.
  1065:   // Inline asm will be put in a packet temporarily, but then it will be
  1066:   // removed, and placed outside of the packet (before or after, depending
  1067:   // on dependencies).  This is to reduce the impact of inline asm as a
  1068:   // "packet splitting" instruction.
  1069:   if (MI.isInlineAsm() && !ScheduleInlineAsm)
  1070:     return true;
  1071: 
  1072:   if (isSchedBarrier(MI))
  1073:     return true;
  1074: 
  1075:   if (HII->isSolo(MI))
  1076:     return true;
  1077: 
  1078:   if (MI.getOpcode() == Hexagon::PATCHABLE_FUNCTION_ENTER ||
  1079:       MI.getOpcode() == Hexagon::PATCHABLE_FUNCTION_EXIT ||
  1080:       MI.getOpcode() == Hexagon::PATCHABLE_TAIL_CALL ||
  1081:       MI.getOpcode() == Hexagon::PATCHABLE_EVENT_CALL ||
  1082:       MI.getOpcode() == Hexagon::PATCHABLE_TYPED_EVENT_CALL)
  1083:     return true;
  1084: 
  1085:   if (MI.getOpcode() == Hexagon::A2_nop)
  1086:     return true;
  1087: 
  1088:   return false;
  1089: }
  1090: 
  1091: // Quick check if instructions MI and MJ cannot coexist in the same packet.
  1092: // Limit the tests to be "one-way", e.g.  "if MI->isBranch and MJ->isInlineAsm",
  1093: // but not the symmetric case: "if MJ->isBranch and MI->isInlineAsm".
  1094: // For full test call this function twice:
  1095: //   cannotCoexistAsymm(MI, MJ) || cannotCoexistAsymm(MJ, MI)
  1096: // Doing the test only one way saves the amount of code in this function,
  1097: // since every test would need to be repeated with the MI and MJ reversed.
  1098: static bool cannotCoexistAsymm(const MachineInstr &MI, const MachineInstr &MJ,
  1099:       const HexagonInstrInfo &HII) {
  1100:   const MachineFunction *MF = MI.getParent()->getParent();
```
- EN: It declares or implements routines such as getPredicatedRegister, contains, HexagonPacketizerList::initPacketizerState, HexagonPacketizerList::ignorePseudoInstruction, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPacketizerList, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getPredicatedRegister, contains, HexagonPacketizerList::initPacketizerState, HexagonPacketizerList::ignorePseudoInstruction, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 1101-1200 / 第 1101-1200 行

```cpp
  1101:   if (MF->getSubtarget<HexagonSubtarget>().hasV60OpsOnly() &&
  1102:       HII.isHVXMemWithAIndirect(MI, MJ))
  1103:     return true;
  1104: 
  1105:   // Don't allow a store and an instruction that must be in slot0 and
  1106:   // doesn't allow a slot1 instruction.
  1107:   if (MI.mayStore() && HII.isRestrictNoSlot1Store(MJ) && HII.isPureSlot0(MJ))
  1108:     return true;
  1109: 
  1110:   // An inline asm cannot be together with a branch, because we may not be
  1111:   // able to remove the asm out after packetizing (i.e. if the asm must be
  1112:   // moved past the bundle).  Similarly, two asms cannot be together to avoid
  1113:   // complications when determining their relative order outside of a bundle.
  1114:   if (MI.isInlineAsm())
  1115:     return MJ.isInlineAsm() || MJ.isBranch() || MJ.isBarrier() ||
  1116:            MJ.isCall() || MJ.isTerminator();
  1117: 
  1118:   // New-value stores cannot coexist with any other stores.
  1119:   if (HII.isNewValueStore(MI) && MJ.mayStore())
  1120:     return true;
  1121: 
  1122:   switch (MI.getOpcode()) {
  1123:   case Hexagon::S2_storew_locked:
  1124:   case Hexagon::S4_stored_locked:
  1125:   case Hexagon::L2_loadw_locked:
  1126:   case Hexagon::L4_loadd_locked:
  1127:   case Hexagon::Y2_dccleana:
  1128:   case Hexagon::Y2_dccleaninva:
  1129:   case Hexagon::Y2_dcinva:
  1130:   case Hexagon::Y2_dczeroa:
  1131:   case Hexagon::Y4_l2fetch:
  1132:   case Hexagon::Y5_l2fetch: {
  1133:     // These instructions can only be grouped with ALU32 or non-floating-point
  1134:     // XTYPE instructions.  Since there is no convenient way of identifying fp
  1135:     // XTYPE instructions, only allow grouping with ALU32 for now.
  1136:     unsigned TJ = HII.getType(MJ);
  1137:     if (TJ != HexagonII::TypeALU32_2op &&
  1138:         TJ != HexagonII::TypeALU32_3op &&
  1139:         TJ != HexagonII::TypeALU32_ADDI)
  1140:       return true;
  1141:     break;
  1142:   }
  1143:   default:
  1144:     break;
  1145:   }
  1146: 
  1147:   // "False" really means that the quick check failed to determine if
  1148:   // I and J cannot coexist.
  1149:   return false;
  1150: }
  1151: 
  1152: // Full, symmetric check.
  1153: bool HexagonPacketizerList::cannotCoexist(const MachineInstr &MI,
  1154:       const MachineInstr &MJ) {
  1155:   return cannotCoexistAsymm(MI, MJ, *HII) || cannotCoexistAsymm(MJ, MI, *HII);
  1156: }
  1157: 
  1158: void HexagonPacketizerList::unpacketizeSoloInstrs(MachineFunction &MF) {
  1159:   for (auto &B : MF) {
  1160:     MachineBasicBlock::iterator BundleIt;
  1161:     for (MachineInstr &MI : llvm::make_early_inc_range(B.instrs())) {
  1162:       if (MI.isBundle())
  1163:         BundleIt = MI.getIterator();
  1164:       if (!MI.isInsideBundle())
  1165:         continue;
  1166: 
  1167:       // Decide on where to insert the instruction that we are pulling out.
  1168:       // Debug instructions always go before the bundle, but the placement of
  1169:       // INLINE_ASM depends on potential dependencies.  By default, try to
  1170:       // put it before the bundle, but if the asm writes to a register that
  1171:       // other instructions in the bundle read, then we need to place it
  1172:       // after the bundle (to preserve the bundle semantics).
  1173:       bool InsertBeforeBundle;
  1174:       if (MI.isInlineAsm())
  1175:         InsertBeforeBundle = !hasWriteToReadDep(MI, *BundleIt, HRI);
  1176:       else if (MI.isDebugInstr())
  1177:         InsertBeforeBundle = true;
  1178:       else
  1179:         continue;
  1180: 
  1181:       BundleIt = moveInstrOut(MI, BundleIt, InsertBeforeBundle);
  1182:     }
  1183:   }
  1184: }
  1185: 
  1186: // Check if a given instruction is of class "system".
  1187: static bool isSystemInstr(const MachineInstr &MI) {
  1188:   unsigned Opc = MI.getOpcode();
  1189:   switch (Opc) {
  1190:     case Hexagon::Y2_barrier:
  1191:     case Hexagon::Y2_dcfetchbo:
  1192:     case Hexagon::Y4_l2fetch:
  1193:     case Hexagon::Y5_l2fetch:
  1194:       return true;
  1195:   }
  1196:   return false;
  1197: }
  1198: 
  1199: bool HexagonPacketizerList::hasDeadDependence(const MachineInstr &I,
  1200:                                               const MachineInstr &J) {
```
- EN: It declares or implements routines such as packetizing, getType, HexagonPacketizerList::cannotCoexist, cannotCoexistAsymm, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonII, HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 packetizing, getType, HexagonPacketizerList::cannotCoexist, cannotCoexistAsymm, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonII, HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 1201-1300 / 第 1201-1300 行

```cpp
  1201:   // The dependence graph may not include edges between dead definitions,
  1202:   // so without extra checks, we could end up packetizing two instruction
  1203:   // defining the same (dead) register.
  1204:   if (I.isCall() || J.isCall())
  1205:     return false;
  1206:   if (HII->isPredicated(I) || HII->isPredicated(J))
  1207:     return false;
  1208: 
  1209:   BitVector DeadDefs(Hexagon::NUM_TARGET_REGS);
  1210:   for (auto &MO : I.operands()) {
  1211:     if (!MO.isReg() || !MO.isDef() || !MO.isDead())
  1212:       continue;
  1213:     DeadDefs[MO.getReg()] = true;
  1214:   }
  1215: 
  1216:   for (auto &MO : J.operands()) {
  1217:     if (!MO.isReg() || !MO.isDef() || !MO.isDead())
  1218:       continue;
  1219:     Register R = MO.getReg();
  1220:     if (R != Hexagon::USR_OVF && DeadDefs[R])
  1221:       return true;
  1222:   }
  1223:   return false;
  1224: }
  1225: 
  1226: bool HexagonPacketizerList::hasControlDependence(const MachineInstr &I,
  1227:                                                  const MachineInstr &J) {
  1228:   // A save callee-save register function call can only be in a packet
  1229:   // with instructions that don't write to the callee-save registers.
  1230:   if ((HII->isSaveCalleeSavedRegsCall(I) &&
  1231:        doesModifyCalleeSavedReg(J, HRI)) ||
  1232:       (HII->isSaveCalleeSavedRegsCall(J) &&
  1233:        doesModifyCalleeSavedReg(I, HRI)))
  1234:     return true;
  1235: 
  1236:   // Two control flow instructions cannot go in the same packet.
  1237:   if (isControlFlow(I) && isControlFlow(J))
  1238:     return true;
  1239: 
  1240:   // \ref-manual (7.3.4) A loop setup packet in loopN or spNloop0 cannot
  1241:   // contain a speculative indirect jump,
  1242:   // a new-value compare jump or a dealloc_return.
  1243:   auto isBadForLoopN = [this] (const MachineInstr &MI) -> bool {
  1244:     if (MI.isCall() || HII->isDeallocRet(MI) || HII->isNewValueJump(MI))
  1245:       return true;
  1246:     if (HII->isPredicated(MI) && HII->isPredicatedNew(MI) && HII->isJumpR(MI))
  1247:       return true;
  1248:     return false;
  1249:   };
  1250: 
  1251:   if (HII->isLoopN(I) && isBadForLoopN(J))
  1252:     return true;
  1253:   if (HII->isLoopN(J) && isBadForLoopN(I))
  1254:     return true;
  1255: 
  1256:   // dealloc_return cannot appear in the same packet as a conditional or
  1257:   // unconditional jump.
  1258:   return HII->isDeallocRet(I) &&
  1259:          (J.isBranch() || J.isCall() || J.isBarrier());
  1260: }
  1261: 
  1262: bool HexagonPacketizerList::hasRegMaskDependence(const MachineInstr &I,
  1263:                                                  const MachineInstr &J) {
  1264:   // Adding I to a packet that has J.
  1265: 
  1266:   // Regmasks are not reflected in the scheduling dependency graph, so
  1267:   // we need to check them manually. This code assumes that regmasks only
  1268:   // occur on calls, and the problematic case is when we add an instruction
  1269:   // defining a register R to a packet that has a call that clobbers R via
  1270:   // a regmask. Those cannot be packetized together, because the call will
  1271:   // be executed last. That's also a reason why it is ok to add a call
  1272:   // clobbering R to a packet that defines R.
  1273: 
  1274:   // Look for regmasks in J.
  1275:   for (const MachineOperand &OpJ : J.operands()) {
  1276:     if (!OpJ.isRegMask())
  1277:       continue;
  1278:     assert((J.isCall() || HII->isTailCall(J)) && "Regmask on a non-call");
  1279:     for (const MachineOperand &OpI : I.operands()) {
  1280:       if (OpI.isReg()) {
  1281:         if (OpJ.clobbersPhysReg(OpI.getReg()))
  1282:           return true;
  1283:       } else if (OpI.isRegMask()) {
  1284:         // Both are regmasks. Assume that they intersect.
  1285:         return true;
  1286:       }
  1287:     }
  1288:   }
  1289:   return false;
  1290: }
  1291: 
  1292: bool HexagonPacketizerList::hasDualStoreDependence(const MachineInstr &I,
  1293:                                                    const MachineInstr &J) {
  1294:   bool SysI = isSystemInstr(I), SysJ = isSystemInstr(J);
  1295:   bool StoreI = I.mayStore(), StoreJ = J.mayStore();
  1296:   if ((SysI && StoreJ) || (SysJ && StoreI))
  1297:     return true;
  1298: 
  1299:   if (StoreI && StoreJ) {
  1300:     if (HII->isNewValueInst(J) || HII->isMemOp(J) || HII->isMemOp(I))
```
- EN: It declares or implements routines such as DeadDefs, getReg, HexagonPacketizerList::hasControlDependence, isDeallocRet, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 DeadDefs, getReg, HexagonPacketizerList::hasControlDependence, isDeallocRet, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 1301-1400 / 第 1301-1400 行

```cpp
  1301:       return true;
  1302:   } else {
  1303:     // A memop cannot be in the same packet with another memop or a store.
  1304:     // Two stores can be together, but here I and J cannot both be stores.
  1305:     bool MopStI = HII->isMemOp(I) || StoreI;
  1306:     bool MopStJ = HII->isMemOp(J) || StoreJ;
  1307:     if (MopStI && MopStJ)
  1308:       return true;
  1309:   }
  1310: 
  1311:   return (StoreJ && HII->isDeallocRet(I)) || (StoreI && HII->isDeallocRet(J));
  1312: }
  1313: 
  1314: // SUI is the current instruction that is outside of the current packet.
  1315: // SUJ is the current instruction inside the current packet against which that
  1316: // SUI will be packetized.
  1317: bool HexagonPacketizerList::isLegalToPacketizeTogether(SUnit *SUI, SUnit *SUJ) {
  1318:   assert(SUI->getInstr() && SUJ->getInstr());
  1319:   MachineInstr &I = *SUI->getInstr();
  1320:   MachineInstr &J = *SUJ->getInstr();
  1321: 
  1322:   // Clear IgnoreDepMIs when Packet starts.
  1323:   if (CurrentPacketMIs.size() == 1)
  1324:     IgnoreDepMIs.clear();
  1325: 
  1326:   MachineBasicBlock::iterator II = I.getIterator();
  1327: 
  1328:   // Solo instructions cannot go in the packet.
  1329:   assert(!isSoloInstruction(I) && "Unexpected solo instr!");
  1330: 
  1331:   if (cannotCoexist(I, J))
  1332:     return false;
  1333: 
  1334:   Dependence = hasDeadDependence(I, J) || hasControlDependence(I, J);
  1335:   if (Dependence)
  1336:     return false;
  1337: 
  1338:   // Regmasks are not accounted for in the scheduling graph, so we need
  1339:   // to explicitly check for dependencies caused by them. They should only
  1340:   // appear on calls, so it's not too pessimistic to reject all regmask
  1341:   // dependencies.
  1342:   Dependence = hasRegMaskDependence(I, J);
  1343:   if (Dependence)
  1344:     return false;
  1345: 
  1346:   // Dual-store does not allow second store, if the first store is not
  1347:   // in SLOT0. New value store, new value jump, dealloc_return and memop
  1348:   // always take SLOT0. Arch spec 3.4.4.2.
  1349:   Dependence = hasDualStoreDependence(I, J);
  1350:   if (Dependence)
  1351:     return false;
  1352: 
  1353:   // If an instruction feeds new value jump, glue it.
  1354:   MachineBasicBlock::iterator NextMII = I.getIterator();
  1355:   ++NextMII;
  1356:   if (NextMII != I.getParent()->end() && HII->isNewValueJump(*NextMII)) {
  1357:     MachineInstr &NextMI = *NextMII;
  1358: 
  1359:     bool secondRegMatch = false;
  1360:     const MachineOperand &NOp0 = NextMI.getOperand(0);
  1361:     const MachineOperand &NOp1 = NextMI.getOperand(1);
  1362: 
  1363:     if (NOp1.isReg() && I.getOperand(0).getReg() == NOp1.getReg())
  1364:       secondRegMatch = true;
  1365: 
  1366:     for (MachineInstr *PI : CurrentPacketMIs) {
  1367:       // NVJ can not be part of the dual jump - Arch Spec: section 7.8.
  1368:       if (PI->isCall()) {
  1369:         Dependence = true;
  1370:         break;
  1371:       }
  1372:       // Validate:
  1373:       // 1. Packet does not have a store in it.
  1374:       // 2. If the first operand of the nvj is newified, and the second
  1375:       //    operand is also a reg, it (second reg) is not defined in
  1376:       //    the same packet.
  1377:       // 3. If the second operand of the nvj is newified, (which means
  1378:       //    first operand is also a reg), first reg is not defined in
  1379:       //    the same packet.
  1380:       if (PI->getOpcode() == Hexagon::S2_allocframe || PI->mayStore() ||
  1381:           HII->isLoopN(*PI)) {
  1382:         Dependence = true;
  1383:         break;
  1384:       }
  1385:       // Check #2/#3.
  1386:       const MachineOperand &OpR = secondRegMatch ? NOp0 : NOp1;
  1387:       if (OpR.isReg() && PI->modifiesRegister(OpR.getReg(), HRI)) {
  1388:         Dependence = true;
  1389:         break;
  1390:       }
  1391:     }
  1392: 
  1393:     GlueToNewValueJump = true;
  1394:     if (Dependence)
  1395:       return false;
  1396:   }
  1397: 
  1398:   // There no dependency between a prolog instruction and its successor.
  1399:   if (!SUJ->isSucc(SUI))
  1400:     return true;
```
- EN: It declares or implements routines such as HexagonPacketizerList::isLegalToPacketizeTogether, assert, getInstr, getIterator, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPacketizerList::isLegalToPacketizeTogether, assert, getInstr, getIterator, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 1401-1500 / 第 1401-1500 行

```cpp
  1401: 
  1402:   for (unsigned i = 0; i < SUJ->Succs.size(); ++i) {
  1403:     if (FoundSequentialDependence)
  1404:       break;
  1405: 
  1406:     if (SUJ->Succs[i].getSUnit() != SUI)
  1407:       continue;
  1408: 
  1409:     SDep::Kind DepType = SUJ->Succs[i].getKind();
  1410:     // For direct calls:
  1411:     // Ignore register dependences for call instructions for packetization
  1412:     // purposes except for those due to r31 and predicate registers.
  1413:     //
  1414:     // For indirect calls:
  1415:     // Same as direct calls + check for true dependences to the register
  1416:     // used in the indirect call.
  1417:     //
  1418:     // We completely ignore Order dependences for call instructions.
  1419:     //
  1420:     // For returns:
  1421:     // Ignore register dependences for return instructions like jumpr,
  1422:     // dealloc return unless we have dependencies on the explicit uses
  1423:     // of the registers used by jumpr (like r31) or dealloc return
  1424:     // (like r29 or r30).
  1425:     unsigned DepReg = 0;
  1426:     const TargetRegisterClass *RC = nullptr;
  1427:     if (DepType == SDep::Data) {
  1428:       DepReg = SUJ->Succs[i].getReg();
  1429:       RC = HRI->getMinimalPhysRegClass(DepReg);
  1430:     }
  1431: 
  1432:     if (I.isCall() || HII->isJumpR(I) || I.isReturn() || HII->isTailCall(I)) {
  1433:       if (!isRegDependence(DepType))
  1434:         continue;
  1435:       if (!isCallDependent(I, DepType, SUJ->Succs[i].getReg()))
  1436:         continue;
  1437:     }
  1438: 
  1439:     if (DepType == SDep::Data) {
  1440:       if (canPromoteToDotCur(J, SUJ, DepReg, II, RC))
  1441:         if (promoteToDotCur(J, DepType, II, RC))
  1442:           continue;
  1443:     }
  1444: 
  1445:     // Data dependence ok if we have load.cur.
  1446:     if (DepType == SDep::Data && HII->isDotCurInst(J)) {
  1447:       if (HII->isHVXVec(I))
  1448:         continue;
  1449:     }
  1450: 
  1451:     // For instructions that can be promoted to dot-new, try to promote.
  1452:     if (DepType == SDep::Data) {
  1453:       if (canPromoteToDotNew(I, SUJ, DepReg, II, RC)) {
  1454:         if (promoteToDotNew(I, DepType, II, RC)) {
  1455:           PromotedToDotNew = true;
  1456:           if (cannotCoexist(I, J))
  1457:             FoundSequentialDependence = true;
  1458:           continue;
  1459:         }
  1460:       }
  1461:       if (HII->isNewValueJump(I))
  1462:         continue;
  1463:     }
  1464: 
  1465:     // For predicated instructions, if the predicates are complements then
  1466:     // there can be no dependence.
  1467:     if (HII->isPredicated(I) && HII->isPredicated(J) &&
  1468:         arePredicatesComplements(I, J)) {
  1469:       // Not always safe to do this translation.
  1470:       // DAG Builder attempts to reduce dependence edges using transitive
  1471:       // nature of dependencies. Here is an example:
  1472:       //
  1473:       // r0 = tfr_pt ... (1)
  1474:       // r0 = tfr_pf ... (2)
  1475:       // r0 = tfr_pt ... (3)
  1476:       //
  1477:       // There will be an output dependence between (1)->(2) and (2)->(3).
  1478:       // However, there is no dependence edge between (1)->(3). This results
  1479:       // in all 3 instructions going in the same packet. We ignore dependce
  1480:       // only once to avoid this situation.
  1481:       auto Itr = find(IgnoreDepMIs, &J);
  1482:       if (Itr != IgnoreDepMIs.end()) {
  1483:         Dependence = true;
  1484:         return false;
  1485:       }
  1486:       IgnoreDepMIs.push_back(&I);
  1487:       continue;
  1488:     }
  1489: 
  1490:     // Ignore Order dependences between unconditional direct branches
  1491:     // and non-control-flow instructions.
  1492:     if (isDirectJump(I) && !J.isBranch() && !J.isCall() &&
  1493:         DepType == SDep::Order)
  1494:       continue;
  1495: 
  1496:     // Ignore all dependences for jumps except for true and output
  1497:     // dependences.
  1498:     if (I.isConditionalBranch() && DepType != SDep::Data &&
  1499:         DepType != SDep::Output)
  1500:       continue;
```
- EN: It declares or implements routines such as size, getKind, getReg, getMinimalPhysRegClass, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 size, getKind, getReg, getMinimalPhysRegClass, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1501-1600 / 第 1501-1600 行

```cpp
  1501: 
  1502:     if (DepType == SDep::Output) {
  1503:       FoundSequentialDependence = true;
  1504:       break;
  1505:     }
  1506: 
  1507:     // For Order dependences:
  1508:     // 1. Volatile loads/stores can be packetized together, unless other
  1509:     //    rules prevent is.
  1510:     // 2. Store followed by a load is not allowed.
  1511:     // 3. Store followed by a store is valid.
  1512:     // 4. Load followed by any memory operation is allowed.
  1513:     if (DepType == SDep::Order) {
  1514:       if (!PacketizeVolatiles) {
  1515:         bool OrdRefs = I.hasOrderedMemoryRef() || J.hasOrderedMemoryRef();
  1516:         if (OrdRefs) {
  1517:           FoundSequentialDependence = true;
  1518:           break;
  1519:         }
  1520:       }
  1521:       // J is first, I is second.
  1522:       bool LoadJ = J.mayLoad(), StoreJ = J.mayStore();
  1523:       bool LoadI = I.mayLoad(), StoreI = I.mayStore();
  1524:       bool NVStoreJ = HII->isNewValueStore(J);
  1525:       bool NVStoreI = HII->isNewValueStore(I);
  1526:       bool IsVecJ = HII->isHVXVec(J);
  1527:       bool IsVecI = HII->isHVXVec(I);
  1528: 
  1529:       // Don't reorder the loads if there is an order dependence. This would
  1530:       // occur if the first instruction must go in slot0.
  1531:       if (LoadJ && LoadI && HII->isPureSlot0(J)) {
  1532:         FoundSequentialDependence = true;
  1533:         break;
  1534:       }
  1535: 
  1536:       if (Slot1Store && MF.getSubtarget<HexagonSubtarget>().hasV65Ops() &&
  1537:           ((LoadJ && StoreI && !NVStoreI) ||
  1538:            (StoreJ && LoadI && !NVStoreJ)) &&
  1539:           (J.getOpcode() != Hexagon::S2_allocframe &&
  1540:            I.getOpcode() != Hexagon::S2_allocframe) &&
  1541:           (J.getOpcode() != Hexagon::L2_deallocframe &&
  1542:            I.getOpcode() != Hexagon::L2_deallocframe) &&
  1543:           (!HII->isMemOp(J) && !HII->isMemOp(I)) && (!IsVecJ && !IsVecI))
  1544:         setmemShufDisabled(true);
  1545:       else
  1546:         if (StoreJ && LoadI && alias(J, I)) {
  1547:           FoundSequentialDependence = true;
  1548:           break;
  1549:         }
  1550: 
  1551:       if (!StoreJ)
  1552:         if (!LoadJ || (!LoadI && !StoreI)) {
  1553:           // If J is neither load nor store, assume a dependency.
  1554:           // If J is a load, but I is neither, also assume a dependency.
  1555:           FoundSequentialDependence = true;
  1556:           break;
  1557:         }
  1558:       // Store followed by store: not OK on V2.
  1559:       // Store followed by load: not OK on all.
  1560:       // Load followed by store: OK on all.
  1561:       // Load followed by load: OK on all.
  1562:       continue;
  1563:     }
  1564: 
  1565:     // Special case for ALLOCFRAME: even though there is dependency
  1566:     // between ALLOCFRAME and subsequent store, allow it to be packetized
  1567:     // in a same packet. This implies that the store is using the caller's
  1568:     // SP. Hence, offset needs to be updated accordingly.
  1569:     if (DepType == SDep::Data && J.getOpcode() == Hexagon::S2_allocframe) {
  1570:       unsigned Opc = I.getOpcode();
  1571:       switch (Opc) {
  1572:         case Hexagon::S2_storerd_io:
  1573:         case Hexagon::S2_storeri_io:
  1574:         case Hexagon::S2_storerh_io:
  1575:         case Hexagon::S2_storerb_io:
  1576:           if (I.getOperand(0).getReg() == HRI->getStackRegister()) {
  1577:             // Since this store is to be glued with allocframe in the same
  1578:             // packet, it will use SP of the previous stack frame, i.e.
  1579:             // caller's SP. Therefore, we need to recalculate offset
  1580:             // according to this change.
  1581:             GlueAllocframeStore = useCallersSP(I);
  1582:             if (GlueAllocframeStore)
  1583:               continue;
  1584:           }
  1585:           break;
  1586:         default:
  1587:           break;
  1588:       }
  1589:     }
  1590: 
  1591:     // There are certain anti-dependencies that cannot be ignored.
  1592:     // Specifically:
  1593:     //   J2_call ... implicit-def %r0   ; SUJ
  1594:     //   R0 = ...                   ; SUI
  1595:     // Those cannot be packetized together, since the call will observe
  1596:     // the effect of the assignment to R0.
  1597:     if ((DepType == SDep::Anti || DepType == SDep::Output) && J.isCall()) {
  1598:       // Check if I defines any volatile register. We should also check
  1599:       // registers that the call may read, but these happen to be a
  1600:       // subset of the volatile register set.
```
- EN: It declares or implements routines such as hasOrderedMemoryRef, mayLoad, isNewValueStore, isHVXVec, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 hasOrderedMemoryRef, mayLoad, isNewValueStore, isHVXVec, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 1601-1700 / 第 1601-1700 行

```cpp
  1601:       for (const MachineOperand &Op : I.operands()) {
  1602:         if (Op.isReg() && Op.isDef()) {
  1603:           Register R = Op.getReg();
  1604:           if (!J.readsRegister(R, HRI) && !J.modifiesRegister(R, HRI))
  1605:             continue;
  1606:         } else if (!Op.isRegMask()) {
  1607:           // If I has a regmask assume dependency.
  1608:           continue;
  1609:         }
  1610:         FoundSequentialDependence = true;
  1611:         break;
  1612:       }
  1613:     }
  1614: 
  1615:     // Skip over remaining anti-dependences. Two instructions that are
  1616:     // anti-dependent can share a packet, since in most such cases all
  1617:     // operands are read before any modifications take place.
  1618:     // The exceptions are branch and call instructions, since they are
  1619:     // executed after all other instructions have completed (at least
  1620:     // conceptually).
  1621:     if (DepType != SDep::Anti) {
  1622:       FoundSequentialDependence = true;
  1623:       break;
  1624:     }
  1625:   }
  1626: 
  1627:   if (FoundSequentialDependence) {
  1628:     Dependence = true;
  1629:     return false;
  1630:   }
  1631: 
  1632:   return true;
  1633: }
  1634: 
  1635: bool HexagonPacketizerList::isLegalToPruneDependencies(SUnit *SUI, SUnit *SUJ) {
  1636:   assert(SUI->getInstr() && SUJ->getInstr());
  1637:   MachineInstr &I = *SUI->getInstr();
  1638:   MachineInstr &J = *SUJ->getInstr();
  1639: 
  1640:   bool Coexist = !cannotCoexist(I, J);
  1641: 
  1642:   if (Coexist && !Dependence)
  1643:     return true;
  1644: 
  1645:   // Check if the instruction was promoted to a dot-new. If so, demote it
  1646:   // back into a dot-old.
  1647:   if (PromotedToDotNew)
  1648:     demoteToDotOld(I);
  1649: 
  1650:   cleanUpDotCur();
  1651:   // Check if the instruction (must be a store) was glued with an allocframe
  1652:   // instruction. If so, restore its offset to its original value, i.e. use
  1653:   // current SP instead of caller's SP.
  1654:   if (GlueAllocframeStore) {
  1655:     useCalleesSP(I);
  1656:     GlueAllocframeStore = false;
  1657:   }
  1658: 
  1659:   if (ChangedOffset != INT64_MAX)
  1660:     undoChangedOffset(I);
  1661: 
  1662:   if (GlueToNewValueJump) {
  1663:     // Putting I and J together would prevent the new-value jump from being
  1664:     // packetized with the producer. In that case I and J must be separated.
  1665:     GlueToNewValueJump = false;
  1666:     return false;
  1667:   }
  1668: 
  1669:   if (!Coexist)
  1670:     return false;
  1671: 
  1672:   if (ChangedOffset == INT64_MAX && updateOffset(SUI, SUJ)) {
  1673:     FoundSequentialDependence = false;
  1674:     Dependence = false;
  1675:     return true;
  1676:   }
  1677: 
  1678:   return false;
  1679: }
  1680: 
  1681: 
  1682: bool HexagonPacketizerList::foundLSInPacket() {
  1683:   bool FoundLoad = false;
  1684:   bool FoundStore = false;
  1685: 
  1686:   for (auto *MJ : CurrentPacketMIs) {
  1687:     unsigned Opc = MJ->getOpcode();
  1688:     if (Opc == Hexagon::S2_allocframe || Opc == Hexagon::L2_deallocframe)
  1689:       continue;
  1690:     if (HII->isMemOp(*MJ))
  1691:       continue;
  1692:     if (MJ->mayLoad())
  1693:       FoundLoad = true;
  1694:     if (MJ->mayStore() && !HII->isNewValueStore(*MJ))
  1695:       FoundStore = true;
  1696:   }
  1697:   return FoundLoad && FoundStore;
  1698: }
  1699: 
  1700: 
```
- EN: It declares or implements routines such as getReg, completed, HexagonPacketizerList::isLegalToPruneDependencies, assert, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getReg, completed, HexagonPacketizerList::isLegalToPruneDependencies, assert, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 1701-1800 / 第 1701-1800 行

```cpp
  1701: MachineBasicBlock::iterator
  1702: HexagonPacketizerList::addToPacket(MachineInstr &MI) {
  1703:   MachineBasicBlock::iterator MII = MI.getIterator();
  1704:   MachineBasicBlock *MBB = MI.getParent();
  1705: 
  1706:   if (CurrentPacketMIs.empty()) {
  1707:     PacketStalls = false;
  1708:     PacketStallCycles = 0;
  1709:   }
  1710:   PacketStalls |= producesStall(MI);
  1711:   PacketStallCycles = std::max(PacketStallCycles, calcStall(MI));
  1712: 
  1713:   if (MI.isImplicitDef()) {
  1714:     // Add to the packet to allow subsequent instructions to be checked
  1715:     // properly.
  1716:     CurrentPacketMIs.push_back(&MI);
  1717:     return MII;
  1718:   }
  1719:   assert(ResourceTracker->canReserveResources(MI));
  1720: 
  1721:   bool ExtMI = HII->isExtended(MI) || HII->isConstExtended(MI);
  1722:   bool Good = true;
  1723: 
  1724:   if (GlueToNewValueJump) {
  1725:     MachineInstr &NvjMI = *++MII;
  1726:     // We need to put both instructions in the same packet: MI and NvjMI.
  1727:     // Either of them can require a constant extender. Try to add both to
  1728:     // the current packet, and if that fails, end the packet and start a
  1729:     // new one.
  1730:     ResourceTracker->reserveResources(MI);
  1731:     if (ExtMI)
  1732:       Good = tryAllocateResourcesForConstExt(true);
  1733: 
  1734:     bool ExtNvjMI = HII->isExtended(NvjMI) || HII->isConstExtended(NvjMI);
  1735:     if (Good) {
  1736:       if (ResourceTracker->canReserveResources(NvjMI))
  1737:         ResourceTracker->reserveResources(NvjMI);
  1738:       else
  1739:         Good = false;
  1740:     }
  1741:     if (Good && ExtNvjMI)
  1742:       Good = tryAllocateResourcesForConstExt(true);
  1743: 
  1744:     if (!Good) {
  1745:       endPacket(MBB, MI);
  1746:       assert(ResourceTracker->canReserveResources(MI));
  1747:       ResourceTracker->reserveResources(MI);
  1748:       if (ExtMI) {
  1749:         assert(canReserveResourcesForConstExt());
  1750:         tryAllocateResourcesForConstExt(true);
  1751:       }
  1752:       assert(ResourceTracker->canReserveResources(NvjMI));
  1753:       ResourceTracker->reserveResources(NvjMI);
  1754:       if (ExtNvjMI) {
  1755:         assert(canReserveResourcesForConstExt());
  1756:         reserveResourcesForConstExt();
  1757:       }
  1758:     }
  1759:     CurrentPacketMIs.push_back(&MI);
  1760:     CurrentPacketMIs.push_back(&NvjMI);
  1761:     return MII;
  1762:   }
  1763: 
  1764:   ResourceTracker->reserveResources(MI);
  1765:   if (ExtMI && !tryAllocateResourcesForConstExt(true)) {
  1766:     endPacket(MBB, MI);
  1767:     if (PromotedToDotNew)
  1768:       demoteToDotOld(MI);
  1769:     if (GlueAllocframeStore) {
  1770:       useCalleesSP(MI);
  1771:       GlueAllocframeStore = false;
  1772:     }
  1773:     ResourceTracker->reserveResources(MI);
  1774:     reserveResourcesForConstExt();
  1775:   }
  1776: 
  1777:   CurrentPacketMIs.push_back(&MI);
  1778:   return MII;
  1779: }
  1780: 
  1781: void HexagonPacketizerList::endPacket(MachineBasicBlock *MBB,
  1782:                                       MachineBasicBlock::iterator EndMI) {
  1783:   // Replace VLIWPacketizerList::endPacket(MBB, EndMI).
  1784:   LLVM_DEBUG({
  1785:     if (!CurrentPacketMIs.empty()) {
  1786:       dbgs() << "Finalizing packet:\n";
  1787:       unsigned Idx = 0;
  1788:       for (MachineInstr *MI : CurrentPacketMIs) {
  1789:         unsigned R = ResourceTracker->getUsedResources(Idx++);
  1790:         dbgs() << " * [res:0x" << utohexstr(R) << "] " << *MI;
  1791:       }
  1792:     }
  1793:   });
  1794: 
  1795:   bool memShufDisabled = getmemShufDisabled();
  1796:   if (memShufDisabled && !foundLSInPacket()) {
  1797:     setmemShufDisabled(false);
  1798:     LLVM_DEBUG(dbgs() << "  Not added to NoShufPacket\n");
  1799:   }
  1800:   memShufDisabled = getmemShufDisabled();
```
- EN: It declares or implements routines such as HexagonPacketizerList::addToPacket, getIterator, getParent, producesStall, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonPacketizerList, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPacketizerList::addToPacket, getIterator, getParent, producesStall, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList，说明了它与同级后端组件的连接关系。

### Lines 1801-1900 / 第 1801-1900 行

```cpp
  1801: 
  1802:   OldPacketMIs.clear();
  1803:   for (MachineInstr *MI : CurrentPacketMIs) {
  1804:     MachineBasicBlock::instr_iterator NextMI = std::next(MI->getIterator());
  1805:     for (auto &I : make_range(HII->expandVGatherPseudo(*MI), NextMI))
  1806:       OldPacketMIs.push_back(&I);
  1807:   }
  1808:   CurrentPacketMIs.clear();
  1809: 
  1810:   if (OldPacketMIs.size() > 1) {
  1811:     MachineBasicBlock::instr_iterator FirstMI(OldPacketMIs.front());
  1812:     MachineBasicBlock::instr_iterator LastMI(EndMI.getInstrIterator());
  1813:     finalizeBundle(*MBB, FirstMI, LastMI);
  1814:     auto BundleMII = std::prev(FirstMI);
  1815:     if (memShufDisabled)
  1816:       HII->setBundleNoShuf(BundleMII);
  1817: 
  1818:     setmemShufDisabled(false);
  1819:   }
  1820: 
  1821:   PacketHasDuplex = false;
  1822:   PacketHasSLOT0OnlyInsn = false;
  1823:   ResourceTracker->clearResources();
  1824:   LLVM_DEBUG(dbgs() << "End packet\n");
  1825: }
  1826: 
  1827: bool HexagonPacketizerList::shouldAddToPacket(const MachineInstr &MI) {
  1828:   if (Minimal)
  1829:     return false;
  1830: 
  1831:   if (producesStall(MI))
  1832:     return false;
  1833: 
  1834:   // If TinyCore with Duplexes is enabled, check if this MI can form a Duplex
  1835:   // with any other instruction in the existing packet.
  1836:   auto &HST = MI.getParent()->getParent()->getSubtarget<HexagonSubtarget>();
  1837:   // Constraint 1: Only one duplex allowed per packet.
  1838:   // Constraint 2: Consider duplex checks only if there is at least one
  1839:   // instruction in a packet.
  1840:   // Constraint 3: If one of the existing instructions in the packet has a
  1841:   // SLOT0 only instruction that can not be duplexed, do not attempt to form
  1842:   // duplexes. (TODO: This will invalidate the L4_return* instructions to form a
  1843:   // duplex)
  1844:   if (HST.isTinyCoreWithDuplex() && CurrentPacketMIs.size() > 0 &&
  1845:       !PacketHasDuplex) {
  1846:     // Check for SLOT0 only non-duplexable instruction in packet.
  1847:     for (auto &MJ : CurrentPacketMIs)
  1848:       PacketHasSLOT0OnlyInsn |= HII->isPureSlot0(*MJ);
  1849:     // Get the Big Core Opcode (dup_*).
  1850:     int Opcode = HII->getDuplexOpcode(MI, false);
  1851:     if (Opcode >= 0) {
  1852:       // We now have an instruction that can be duplexed.
  1853:       for (auto &MJ : CurrentPacketMIs) {
  1854:         if (HII->isDuplexPair(MI, *MJ) && !PacketHasSLOT0OnlyInsn) {
  1855:           PacketHasDuplex = true;
  1856:           return true;
  1857:         }
  1858:       }
  1859:       // If it can not be duplexed, check if there is a valid transition in DFA
  1860:       // with the original opcode.
  1861:       MachineInstr &MIRef = const_cast<MachineInstr &>(MI);
  1862:       MIRef.setDesc(HII->get(Opcode));
  1863:       return ResourceTracker->canReserveResources(MIRef);
  1864:     }
  1865:   }
  1866: 
  1867:   return true;
  1868: }
  1869: 
  1870: // V60 forward scheduling.
  1871: unsigned int HexagonPacketizerList::calcStall(const MachineInstr &I) {
  1872:   // Check whether the previous packet is in a different loop. If this is the
  1873:   // case, there is little point in trying to avoid a stall because that would
  1874:   // favor the rare case (loop entry) over the common case (loop iteration).
  1875:   //
  1876:   // TODO: We should really be able to check all the incoming edges if this is
  1877:   // the first packet in a basic block, so we can avoid stalls from the loop
  1878:   // backedge.
  1879:   if (!OldPacketMIs.empty()) {
  1880:     auto *OldBB = OldPacketMIs.front()->getParent();
  1881:     auto *ThisBB = I.getParent();
  1882:     if (MLI->getLoopFor(OldBB) != MLI->getLoopFor(ThisBB))
  1883:       return 0;
  1884:   }
  1885: 
  1886:   SUnit *SUI = MIToSUnit[const_cast<MachineInstr *>(&I)];
  1887:   if (!SUI)
  1888:     return 0;
  1889: 
  1890:   // If the latency is 0 and there is a data dependence between this
  1891:   // instruction and any instruction in the current packet, we disregard any
  1892:   // potential stalls due to the instructions in the previous packet. Most of
  1893:   // the instruction pairs that can go together in the same packet have 0
  1894:   // latency between them. The exceptions are
  1895:   // 1. NewValueJumps as they're generated much later and the latencies can't
  1896:   // be changed at that point.
  1897:   // 2. .cur instructions, if its consumer has a 0 latency successor (such as
  1898:   // .new). In this case, the latency between .cur and the consumer stays
  1899:   // non-zero even though we can have  both .cur and .new in the same packet.
  1900:   // Changing the latency to 0 is not an option as it causes software pipeliner
```
- EN: It declares or implements routines such as clear, std::next, FirstMI, LastMI, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPacketizerList, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 clear, std::next, FirstMI, LastMI, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 1901-1948 / 第 1901-1948 行

```cpp
  1901:   // to not pipeline in some cases.
  1902: 
  1903:   // For Example:
  1904:   // {
  1905:   //   I1:  v6.cur = vmem(r0++#1)
  1906:   //   I2:  v7 = valign(v6,v4,r2)
  1907:   //   I3:  vmem(r5++#1) = v7.new
  1908:   // }
  1909:   // Here I2 and I3 has 0 cycle latency, but I1 and I2 has 2.
  1910: 
  1911:   for (auto *J : CurrentPacketMIs) {
  1912:     SUnit *SUJ = MIToSUnit[J];
  1913:     for (auto &Pred : SUI->Preds)
  1914:       if (Pred.getSUnit() == SUJ)
  1915:         if ((Pred.getLatency() == 0 && Pred.isAssignedRegDep()) ||
  1916:             HII->isNewValueJump(I) || HII->isToBeScheduledASAP(*J, I))
  1917:           return 0;
  1918:   }
  1919: 
  1920:   // Check if the latency is greater than one between this instruction and any
  1921:   // instruction in the previous packet.
  1922:   for (auto *J : OldPacketMIs) {
  1923:     SUnit *SUJ = MIToSUnit[J];
  1924:     for (auto &Pred : SUI->Preds)
  1925:       if (Pred.getSUnit() == SUJ && Pred.getLatency() > 1)
  1926:         return Pred.getLatency();
  1927:   }
  1928: 
  1929:   return 0;
  1930: }
  1931: 
  1932: bool HexagonPacketizerList::producesStall(const MachineInstr &I) {
  1933:   unsigned int Latency = calcStall(I);
  1934:   if (Latency == 0)
  1935:     return false;
  1936:   // Ignore stall unless it stalls more than previous instruction in packet
  1937:   if (PacketStalls)
  1938:     return Latency > PacketStallCycles;
  1939:   return true;
  1940: }
  1941: 
  1942: //===----------------------------------------------------------------------===//
  1943: //                         Public Constructor Functions
  1944: //===----------------------------------------------------------------------===//
  1945: 
  1946: FunctionPass *llvm::createHexagonPacketizer(bool Minimal) {
  1947:   return new HexagonPacketizer(Minimal);
  1948: }
```
- EN: It declares or implements routines such as HexagonPacketizerList::producesStall, calcStall, llvm::createHexagonPacketizer, HexagonPacketizer, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPacketizerList, HexagonPacketizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonPacketizerList::producesStall, calcStall, llvm::createHexagonPacketizer, HexagonPacketizer 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPacketizerList, HexagonPacketizer，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- instruction scheduling models / 指令调度模型
- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonVLIWPacketizer.h, Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, llvm/ADT/DenseSet.h, llvm/ADT/STLExtras.h, llvm/ADT/StringExtras.h, llvm/Analysis/AliasAnalysis.h, ... (34 total)`
- Hexagon symbols / Hexagon 符号: `HexagonPacketizer, HexagonVLIWPacketizer, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonPacketizerList, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
