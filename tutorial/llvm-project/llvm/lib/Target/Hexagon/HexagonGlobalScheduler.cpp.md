# HexagonGlobalScheduler.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonGlobalScheduler.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon global scheduling heuristics and transformations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: 
     2: //===----- HexagonGlobalScheduler.cpp - Global Scheduler ------------------===//
     3: //
     4: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     5: // See https://llvm.org/LICENSE.txt for license information.
     6: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     7: //
     8: //===----------------------------------------------------------------------===//
     9: //
    10: // Basic infrastructure for the global scheduling + Hexagon pull-up pass.
    11: // Currently run at the very end of code generation for Hexagon, cleans
    12: // up lost scheduling opportunities. Currently breaks liveness, so no passes
    13: // that rely on liveness info should run afterwards. Will be fixed in future
    14: // versions.
    15: //
    16: //===----------------------------------------------------------------------===//
    17: #include "Hexagon.h"
    18: #include "HexagonGlobalRegion.h"
    19: #include "HexagonMachineFunctionInfo.h"
    20: #include "HexagonRegisterInfo.h"
    21: #include "HexagonSubtarget.h"
    22: #include "HexagonTargetMachine.h"
    23: #include "HexagonVLIWPacketizer.h"
    24: #include "llvm/ADT/DenseMap.h"
    25: #include "llvm/ADT/SmallSet.h"
    26: #include "llvm/ADT/Statistic.h"
    27: #include "llvm/Analysis/AliasAnalysis.h"
    28: #include "llvm/Analysis/ValueTracking.h"
    29: #include "llvm/CodeGen/DFAPacketizer.h"
    30: #include "llvm/CodeGen/LatencyPriorityQueue.h"
    31: #include "llvm/CodeGen/LiveIntervals.h"
    32: #include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
    33: #include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
    34: #include "llvm/CodeGen/MachineDominators.h"
    35: #include "llvm/CodeGen/MachineFrameInfo.h"
    36: #include "llvm/CodeGen/MachineFunctionPass.h"
    37: #include "llvm/CodeGen/MachineInstrBuilder.h"
    38: #include "llvm/CodeGen/MachineLoopInfo.h"
    39: #include "llvm/CodeGen/MachineRegisterInfo.h"
    40: #include "llvm/CodeGen/Passes.h"
    41: #include "llvm/CodeGen/PseudoSourceValue.h"
    42: #include "llvm/CodeGen/SchedulerRegistry.h"
    43: #include "llvm/CodeGen/TargetInstrInfo.h"
    44: #include "llvm/CodeGen/TargetRegisterInfo.h"
    45: #include "llvm/CodeGen/TargetSchedule.h"
    46: #include "llvm/IR/Operator.h"
    47: #include "llvm/InitializePasses.h"
    48: #include "llvm/MC/MCInstrItineraries.h"
    49: #include "llvm/Support/CommandLine.h"
    50: #include "llvm/Support/Compiler.h"
    51: #include "llvm/Support/Debug.h"
    52: #include "llvm/Support/MathExtras.h"
    53: #include "llvm/Target/TargetMachine.h"
    54: #include "llvm/Transforms/Utils/BasicBlockUtils.h"
    55: 
    56: #include <list>
    57: #include <map>
    58: 
    59: #define DEBUG_TYPE "global_sched"
    60: 
    61: using namespace llvm;
    62: 
    63: STATISTIC(HexagonNumPullUps, "Number of instructions pull-ups");
    64: STATISTIC(HexagonNumDualJumps, "Number of dual jumps formed");
    65: 
    66: static cl::opt<bool> DisablePullUp("disable-pull-up", cl::Hidden,
    67:                                    cl::desc("Disable Hexagon pull-up pass"));
    68: 
    69: static cl::opt<bool> EnableSpeculativePullUp(
    70:     "enable-speculative-pull-up", cl::Hidden,
    71:     cl::desc("Enable speculation during Hexagon pull-up pass"));
    72: 
    73: static cl::opt<bool> EnableLocalPullUp(
    74:     "enable-local-pull-up", cl::Hidden, cl::init(true),
    75:     cl::desc("Enable same BB pull during Hexagon pull-up pass"));
    76: 
    77: static cl::opt<bool> AllowSpeculateLoads(
    78:     "speculate-loads-on-pull-up", cl::Hidden, cl::init(true),
    79:     cl::desc("Allow speculative loads during Hexagon pull-up pass"));
    80: 
    81: static cl::opt<bool> AllowCmpBranchLoads(
    82:     "cmp-branch-loads-pull-up", cl::Hidden, cl::init(true),
    83:     cl::desc("Allow compare-branch loads during Hexagon pull-up pass"));
    84: 
    85: static cl::opt<bool> AllowUnlikelyPath("unlikely-path-pull-up", cl::Hidden,
    86:                                        cl::init(true),
    87:                                        cl::desc("Allow unlikely path pull up"));
    88: 
    89: static cl::opt<bool>
    90:     PerformDualJumps("dual-jump-in-pull-up", cl::Hidden, cl::init(true),
    91:                      cl::desc("Perform dual jump formation during pull up"));
    92: 
    93: static cl::opt<bool> AllowDependentPullUp(
    94:     "enable-dependent-pull-up", cl::Hidden, cl::init(true),
    95:     cl::desc("Perform dual jump formation during pull up"));
    96: 
    97: static cl::opt<bool>
    98:     AllowBBPeelPullUp("enable-bb-peel-pull-up", cl::Hidden, cl::init(true),
    99:                       cl::desc("Peel a reg copy out of a BBloop"));
   100: 
   101: static cl::opt<bool> PreventCompoundSeparation(
   102:     "prevent-compound-separation", cl::Hidden,
   103:     cl::desc("Do not destroy existing compounds during pull up"));
   104: 
   105: static cl::opt<bool> PreventDuplexSeparation(
   106:     "prevent-duplex-separation", cl::Hidden, cl::init(true),
   107:     cl::desc("Do not destroy existing duplexes during pull up"));
   108: 
   109: static cl::opt<unsigned> MainCandidateQueueSize("pull-up-main-queue-size",
   110:                                                 cl::Hidden, cl::init(8));
   111: 
   112: static cl::opt<unsigned> SecondaryCandidateQueueSize("pull-up-sec-queue-size",
   113:                                                      cl::Hidden, cl::init(2));
   114: 
   115: static cl::opt<bool> PostPullUpOpt(
   116:     "post-pull-up-opt", cl::Hidden, cl::Optional, cl::init(true),
   117:     cl::desc("Enable opt. exposed by pull-up e.g., remove redundant jumps"));
   118: 
   119: static cl::opt<bool> SpeculateNonPredInsn(
   120:     "speculate-non-pred-insn", cl::Hidden, cl::Optional, cl::init(true),
   121:     cl::desc("Speculate non-predicable instructions in parent BB"));
   122: 
   123: static cl::opt<bool>
   124:     DisableCheckBundles("disable-hexagon-check-bundles", cl::Hidden,
   125:                         cl::init(true),
   126:                         cl::desc("Disable Hexagon check bundles pass"));
   127: 
   128: static cl::opt<bool>
   129:     WarnOnBundleSize("warn-on-bundle-size", cl::Hidden,
   130:                      cl::desc("Hexagon check bundles and warn on size"));
   131: 
   132: static cl::opt<bool>
   133:     ForceNoopHazards("force-noop-hazards", cl::Hidden, cl::init(false),
   134:                      cl::desc("Force noop hazards in scheduler"));
   135: static cl::opt<bool> OneFloatPerPacket(
   136:     "single-float-packet", cl::Hidden,
   137:     cl::desc("Allow only one single floating point instruction in a packet"));
   138: static cl::opt<bool> OneComplexPerPacket(
   139:     "single-complex-packet", cl::Hidden,
   140:     cl::desc("Allow only one complex instruction in a packet"));
   141: 
   142: namespace llvm {
   143: FunctionPass *createHexagonGlobalScheduler();
   144: void initializeHexagonGlobalSchedulerPass(PassRegistry &);
   145: } // namespace llvm
   146: 
   147: namespace {
   148: class HexagonGlobalSchedulerImpl;
   149: 
   150: class HexagonGlobalScheduler : public MachineFunctionPass {
   151: public:
   152:   static char ID;
   153:   HexagonGlobalScheduler() : MachineFunctionPass(ID) {
   154:     initializeHexagonGlobalSchedulerPass(*PassRegistry::getPassRegistry());
   155:   }
   156: 
   157:   void getAnalysisUsage(AnalysisUsage &AU) const override {
   158:     AU.addRequiredID(MachineDominatorsID);
   159:     AU.addRequired<MachineLoopInfoWrapperPass>();
   160:     AU.addRequired<AAResultsWrapperPass>();
   161:     AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();
   162:     AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();
   163:     AU.addRequired<MachineDominatorTreeWrapperPass>();
   164:     MachineFunctionPass::getAnalysisUsage(AU);
   165:   }
   166: 
   167:   StringRef getPassName() const override { return "Hexagon Global Scheduler"; }
   168: 
   169:   bool runOnMachineFunction(MachineFunction &Fn) override;
   170: };
   171: char HexagonGlobalScheduler::ID = 0;
   172: 
   173: // Describes a single pull-up candidate.
   174: class PullUpCandidate {
   175:   MachineBasicBlock::instr_iterator CandidateLocation;
   176:   MachineBasicBlock::iterator HomeBundle;
   177:   bool DependentOp;
   178:   signed BenefitCost;
   179:   std::vector<MachineInstr *> Backtrack;
   180: 
   181: public:
   182:   PullUpCandidate(MachineBasicBlock::instr_iterator MII) {
   183:     CandidateLocation = MII;
   184:     BenefitCost = 0;
   185:   }
   186: 
   187:   PullUpCandidate(MachineBasicBlock::instr_iterator MII,
   188:                   MachineBasicBlock::iterator HomeBundle,
   189:                   std::vector<MachineInstr *> &backtrack, bool DependentOp,
   190:                   signed Cost)
   191:       : CandidateLocation(MII), HomeBundle(HomeBundle),
   192:         DependentOp(DependentOp), BenefitCost(Cost) {
   193:     // Copy of the backtrack.
   194:     Backtrack = backtrack;
   195:   }
   196: 
   197:   void populate(MachineBasicBlock::instr_iterator &MII,
   198:                 MachineBasicBlock::iterator &WorkPoint,
   199:                 std::vector<MachineInstr *> &backtrack, bool &dependentOp) {
   200:     MII = CandidateLocation;
```
- EN: It imports headers such as Hexagon.h, HexagonGlobalRegion.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, ... (40 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonGlobalSchedulerImpl, HexagonGlobalScheduler, PullUpCandidate, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonGlobalRegion.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, ... (40 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonGlobalSchedulerImpl, HexagonGlobalScheduler, PullUpCandidate 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:     WorkPoint = HomeBundle;
   202:     backtrack = Backtrack;
   203:     dependentOp = DependentOp;
   204:   }
   205: 
   206:   signed getCost() { return BenefitCost; }
   207: 
   208:   MachineInstr *getCandidate() { return &*CandidateLocation; }
   209: 
   210:   void dump() {
   211:     dbgs() << "Cost(" << BenefitCost;
   212:     dbgs() << ") Dependent(" << DependentOp;
   213:     dbgs() << ") backtrack size(" << Backtrack.size() << ")\t";
   214:     CandidateLocation->dump();
   215:   }
   216: };
   217: 
   218: /// PullUpCandidateSorter - A Sort utility for pull-up candidates.
   219: struct PullUpCandidateSorter {
   220:   PullUpCandidateSorter() {}
   221:   bool operator()(PullUpCandidate *LHS, PullUpCandidate *RHS) {
   222:     return LHS->getCost() > RHS->getCost();
   223:   }
   224: };
   225: 
   226: // Describes a single pull-up opportunity: location to which
   227: // pull-up is possible with additional information about it.
   228: // Also contains a list of pull-up candidates for this location.
   229: class PullUpState {
   230:   friend class HexagonGlobalSchedulerImpl;
   231:   // Available opportunity for pull-up.
   232:   // FAIAP a bundle with an empty slot.
   233:   MachineBasicBlock::iterator HomeLocation;
   234:   // Home bundle copy. This is here for speed of iteration.
   235:   SmallVector<MachineInstr *, HEXAGON_PACKET_SIZE> HomeBundle;
   236:   // Multiple candidates for the Home location.
   237:   SmallVector<PullUpCandidate *, 8> PullUpCandidates;
   238: 
   239:   const HexagonInstrInfo *QII;
   240: 
   241: public:
   242:   PullUpState(const HexagonInstrInfo *QII) : HomeLocation(NULL), QII(QII) {}
   243: 
   244:   ~PullUpState() { reset(); }
   245: 
   246:   void addPullUpCandidate(MachineBasicBlock::instr_iterator MII,
   247:                           MachineBasicBlock::iterator HomeBundle,
   248:                           std::vector<MachineInstr *> &backtrack,
   249:                           bool DependentOp, signed Cost) {
   250:     LLVM_DEBUG(dbgs() << "\t[addPullUpCandidate]: "; (*MII).dump());
   251:     PullUpCandidate *PUI =
   252:         new PullUpCandidate(MII, HomeBundle, backtrack, DependentOp, Cost);
   253:     PullUpCandidates.push_back(PUI);
   254:   }
   255: 
   256:   void dump() {
   257:     unsigned element = 0;
   258:     for (unsigned i = 0; i < HomeBundle.size(); i++) {
   259:       dbgs() << "[" << element++;
   260:       dbgs() << "] Home Duplex("
   261:              << QII->getDuplexCandidateGroup(*HomeBundle[i]);
   262:       dbgs() << ") Compound (" << QII->getCompoundCandidateGroup(*HomeBundle[i])
   263:              << ") ";
   264:       HomeBundle[i]->dump();
   265:     }
   266:     dbgs() << "\n";
   267:     element = 0;
   268:     for (SmallVector<PullUpCandidate *, 4>::iterator
   269:              I = PullUpCandidates.begin(),
   270:              E = PullUpCandidates.end();
   271:          I != E; ++I) {
   272:       dbgs() << "[" << element++ << "] Cand: Compound(";
   273:       dbgs() << QII->getCompoundCandidateGroup(*(*I)->getCandidate()) << ") ";
   274:       (*I)->dump();
   275:     }
   276:   }
   277: 
   278:   void reset() {
   279:     HomeLocation = NULL;
   280:     for (SmallVector<PullUpCandidate *, 4>::iterator
   281:              I = PullUpCandidates.begin(),
   282:              E = PullUpCandidates.end();
   283:          I != E; ++I)
   284:       delete *I;
   285:     PullUpCandidates.clear();
   286:     HomeBundle.clear();
   287:   }
   288: 
   289:   void addHomeLocation(MachineBasicBlock::iterator WorkPoint) {
   290:     reset();
   291:     HomeLocation = WorkPoint;
   292:   }
   293: 
   294:   unsigned haveCandidates() { return PullUpCandidates.size(); }
   295: };
   296: 
   297: class HexagonGlobalSchedulerImpl : public HexagonPacketizerList {
   298:   // List of PullUp regions for this function.
   299:   std::vector<BasicBlockRegion *> PullUpRegions;
   300:   // Map of approximate distance for each BB from the
   301:   // function base.
   302:   DenseMap<MachineBasicBlock *, unsigned> BlockToInstOffset;
   303:   // Keep track of multiple pull-up candidates.
   304:   PullUpState CurrentState;
   305:   // Empty basic blocks as a result of pull-up.
   306:   std::vector<MachineBasicBlock *> EmptyBBs;
   307:   // Save all the Speculated MachineInstr that were moved
   308:   // FROM MachineBasicBlock because we don't want to have
   309:   // more than one speculated instructions pulled into one packet.
   310:   // TODO: This can be removed once we have a use-def dependency chain
   311:   // for all the instructions in a function.
   312:   std::map<MachineInstr *, MachineBasicBlock *> SpeculatedIns;
   313:   // All the regs and their aliases used by an instruction.
   314:   std::map<MachineInstr *, std::vector<unsigned>> MIUseSet;
   315:   // All the regs and their aliases defined by an instruction.
   316:   std::map<MachineInstr *, std::vector<unsigned>> MIDefSet;
   317: 
   318:   AliasAnalysis *AA;
   319:   const MachineBranchProbabilityInfo *MBPI;
   320:   const MachineBlockFrequencyInfo *MBFI;
   321:   const MachineRegisterInfo *MRI;
   322:   const MachineFrameInfo &MFI;
   323:   const HexagonRegisterInfo *QRI;
   324:   const HexagonInstrInfo *QII;
   325:   MachineLoopInfo &MLI;
   326:   MachineDominatorTree &MDT;
   327:   MachineInstrBuilder Ext;
   328:   MachineInstrBuilder Nop;
   329:   const unsigned PacketSize;
   330:   TargetSchedModel TSchedModel;
   331: 
   332: public:
   333:   // Ctor.
   334:   HexagonGlobalSchedulerImpl(MachineFunction &MF, MachineLoopInfo &MLI,
   335:                              MachineDominatorTree &MDT, AliasAnalysis *AA,
   336:                              const MachineBranchProbabilityInfo *MBPI,
   337:                              const MachineBlockFrequencyInfo *MBFI,
   338:                              const MachineRegisterInfo *MRI,
   339:                              const MachineFrameInfo &MFI,
   340:                              const HexagonRegisterInfo *QRI);
   341:   HexagonGlobalSchedulerImpl(const HexagonGlobalSchedulerImpl &) = delete;
   342:   HexagonGlobalSchedulerImpl &
   343:   operator=(const HexagonGlobalSchedulerImpl &) = delete;
   344: 
   345:   ~HexagonGlobalSchedulerImpl() {
   346:     // Free regions.
   347:     for (std::vector<BasicBlockRegion *>::iterator I = PullUpRegions.begin(),
   348:                                                    E = PullUpRegions.end();
   349:          I != E; ++I)
   350:       delete *I;
   351:     MF.deleteMachineInstr(Ext);
   352:     MF.deleteMachineInstr(Nop);
   353:   }
   354: 
   355:   // initPacketizerState - initialize some internal flags.
   356:   void initPacketizerState() override;
   357: 
   358:   // ignorePseudoInstruction - Ignore bundling of pseudo instructions.
   359:   bool ignoreInstruction(MachineInstr *MI);
   360: 
   361:   // isSoloInstruction - return true if instruction MI can not be packetized
   362:   // with any other instruction, which means that MI itself is a packet.
   363:   bool isSoloInstruction(const MachineInstr &MI) override;
   364: 
   365:   // Add MI to packetizer state. Returns false if it cannot fit in the packet.
   366:   bool incrementalAddToPacket(MachineInstr &MI);
   367: 
   368:   // formPullUpRegions - Top level call to form regions.
   369:   bool formPullUpRegions(MachineFunction &Fn);
   370: 
   371:   // performPullUp - Top level call for pull-up.
   372:   bool performPullUp();
   373: 
   374:   // performPullUpCFG - Top level call for pull-up CFG.
   375:   bool performPullUpCFG(MachineFunction &Fn);
   376: 
   377:   // performExposedOptimizations -
   378:   // Look for optimization opportunities after pullup.
   379:   bool performExposedOptimizations(MachineFunction &Fn);
   380: 
   381:   // optimizeBranching -
   382:   // 1. A conditional-jump transfers control to a BB with
   383:   // jump as the only instruction.
   384:   // if(p0) jump t1
   385:   // // ...
   386:   // t1: jump t2
   387:   // 2. When a BB with a single conditional jump, jumps to succ-of-succ and
   388:   // falls-through BB with only jump instruction.
   389:   // { if(p0) jump t1 }
   390:   // { jump t2 }
   391:   // t1: { ... }
   392:   MachineBasicBlock *optimizeBranches(MachineBasicBlock *MBB,
   393:                                       MachineBasicBlock *TBB,
   394:                                       MachineInstr *FirstTerm,
   395:                                       MachineBasicBlock *FBB);
   396: 
   397:   // removeRedundantBranches -
   398:   // 1. Remove jump to the layout successor.
   399:   // 2. Remove multiple (dual) jump to the same target.
   400:   bool removeRedundantBranches(MachineBasicBlock *MBB, MachineBasicBlock *TBB,
```
- EN: It declares types such as PullUpCandidateSorter, PullUpState, HexagonGlobalSchedulerImpl, which carry the state or API of this component. It defines declarative TableGen records like PullUpState, HexagonGlobalSchedulerImpl; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getCost, getCandidate, dump, PullUpCandidateSorter, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 PullUpCandidateSorter, PullUpState, HexagonGlobalSchedulerImpl 等类型，用来承载该组件的状态或接口。 这里定义了 PullUpState, HexagonGlobalSchedulerImpl 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getCost, getCandidate, dump, PullUpCandidateSorter, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 401-600 / 第 401-600 行

```cpp
   401:                                MachineInstr *FirstTerm, MachineBasicBlock *FBB,
   402:                                MachineInstr *SecondTerm);
   403: 
   404:   // optimizeDualJumps - optimize dual jumps in a packet
   405:   // For now: Replace dual jump by single jump in case of a fall through.
   406:   bool optimizeDualJumps(MachineBasicBlock *MBB, MachineBasicBlock *TBB,
   407:                          MachineInstr *FirstTerm, MachineBasicBlock *FBB,
   408:                          MachineInstr *SecondTerm);
   409: 
   410:   void GenUseDefChain(MachineFunction &Fn);
   411: 
   412:   // Return region pointer or null if none found.
   413:   BasicBlockRegion *getRegionForMBB(std::vector<BasicBlockRegion *> &Regions,
   414:                                     MachineBasicBlock *MBB);
   415: 
   416:   // Saves all the used-regs and their aliases in Uses.
   417:   // Saves all the defined-regs and their aliases in Defs.
   418:   void MIUseDefSet(MachineInstr *MI, std::vector<unsigned> &Defs,
   419:                    std::vector<unsigned> &Uses);
   420: 
   421:   // This is a very useful debug utility.
   422:   unsigned countCompounds(MachineFunction &Fn);
   423: 
   424:   // Check bundle counts
   425:   void checkBundleCounts(MachineFunction &Fn);
   426: 
   427: private:
   428:   // Get next BB to be included into the region.
   429:   MachineBasicBlock *getNextPURBB(MachineBasicBlock *MBB, bool SecondBest);
   430: 
   431:   void setUsedRegs(BitVector &Set, unsigned Reg);
   432:   bool AliasingRegs(unsigned RegA, unsigned RegB);
   433: 
   434:   // Test is true if the two MIs cannot be safely reordered.
   435:   bool ReorderDependencyTest(MachineInstr *MIa, MachineInstr *MIb);
   436: 
   437:   bool canAddMIToThisPacket(
   438:       MachineInstr *MI,
   439:       SmallVector<MachineInstr *, HEXAGON_PACKET_SIZE> &Bundle);
   440: 
   441:   bool CanPromoteToDotNew(MachineInstr *MI, unsigned Reg);
   442: 
   443:   bool pullUpPeelBBLoop(MachineBasicBlock *PredBB, MachineBasicBlock *LoopBB);
   444: 
   445:   MachineInstr *findBundleAndBranch(MachineBasicBlock *BB,
   446:                                     MachineBasicBlock::iterator &Bundle);
   447: 
   448:   // Does this bundle have any slots left?
   449:   bool ResourcesAvailableInBundle(BasicBlockRegion *CurrentRegion,
   450:                                   MachineBasicBlock::iterator &TargetPacket);
   451: 
   452:   // Perform the actual move.
   453:   MachineInstr *MoveAndUpdateLiveness(
   454:       BasicBlockRegion *CurrentRegion, MachineBasicBlock *HomeBB,
   455:       MachineInstr *InstrToMove, bool NeedToNewify, unsigned DepReg,
   456:       bool MovingDependentOp, MachineBasicBlock *OriginBB,
   457:       MachineInstr *OriginalInstruction, SmallVector<MachineOperand, 4> &Cond,
   458:       MachineBasicBlock::iterator &SourceLocation,
   459:       MachineBasicBlock::iterator &TargetPacket,
   460:       MachineBasicBlock::iterator &NextMI,
   461:       std::vector<MachineInstr *> &backtrack);
   462: 
   463:   // Updates incremental kill patterns along the backtrack.
   464:   void updateKillAlongThePath(MachineBasicBlock *HomeBB,
   465:                               MachineBasicBlock *OriginBB,
   466:                               MachineBasicBlock::instr_iterator &Head,
   467:                               MachineBasicBlock::instr_iterator &Tail,
   468:                               MachineBasicBlock::iterator &SourcePacket,
   469:                               MachineBasicBlock::iterator &TargetPacket,
   470:                               std::vector<MachineInstr *> &backtrack);
   471: 
   472:   // Gather list of pull-up candidates.
   473:   bool findPullUpCandidates(MachineBasicBlock::iterator &WorkPoint,
   474:                             MachineBasicBlock::iterator &FromHere,
   475:                             std::vector<MachineInstr *> &backtrack,
   476:                             unsigned MaxCandidates);
   477: 
   478:   // See if the instruction could be pulled up.
   479:   bool tryMultipleInstructions(
   480:       MachineBasicBlock::iterator &RetVal, /* output parameter */
   481:       std::vector<BasicBlockRegion *>::iterator &CurrentRegion,
   482:       MachineBasicBlock::iterator &NextMI,
   483:       MachineBasicBlock::iterator &ToThisBBEnd,
   484:       MachineBasicBlock::iterator &FromThisBBEnd, bool PathInRegion = true);
   485: 
   486:   // Try to move MI into existing bundle.
   487:   bool MoveMItoBundle(BasicBlockRegion *CurrentRegion,
   488:                       MachineBasicBlock::instr_iterator &InstrToMove,
   489:                       MachineBasicBlock::iterator &NextMI,
   490:                       MachineBasicBlock::iterator &TargetPacket,
   491:                       MachineBasicBlock::iterator &SourceLocation,
   492:                       std::vector<MachineInstr *> &backtrack,
   493:                       bool MovingDependentOp, bool PathInRegion);
   494: 
   495:   // Insert temporary MI copy into MBB.
   496:   MachineBasicBlock::instr_iterator
   497:   insertTempCopy(MachineBasicBlock *MBB,
   498:                  MachineBasicBlock::iterator &TargetPacket, MachineInstr *MI,
   499:                  bool DeleteOldCopy);
   500: 
   501:   MachineBasicBlock::instr_iterator
   502:   findInsertPositionInBundle(MachineBasicBlock::iterator &Bundle,
   503:                              MachineInstr *MI, bool &LastInBundle);
   504: 
   505:   bool NeedToNewify(MachineBasicBlock::instr_iterator NewMI, unsigned *DepReg,
   506:                     MachineInstr *TargetPacket);
   507: 
   508:   bool CanNewifiedBeUsedInBundle(MachineBasicBlock::instr_iterator NewMI,
   509:                                  unsigned DepReg, MachineInstr *TargetPacket);
   510: 
   511:   void addInstructionToExistingBundle(MachineBasicBlock *HomeBB,
   512:                                       MachineBasicBlock::instr_iterator &Head,
   513:                                       MachineBasicBlock::instr_iterator &Tail,
   514:                                       MachineBasicBlock::instr_iterator &NewMI,
   515:                                       MachineBasicBlock::iterator &TargetPacket,
   516:                                       MachineBasicBlock::iterator &NextMI,
   517:                                       std::vector<MachineInstr *> &backtrack);
   518: 
   519:   void removeInstructionFromExistingBundle(
   520:       MachineBasicBlock *HomeBB, MachineBasicBlock::instr_iterator &Head,
   521:       MachineBasicBlock::instr_iterator &Tail,
   522:       MachineBasicBlock::iterator &SourceLocation,
   523:       MachineBasicBlock::iterator &NextMI, bool MovingDependentOp,
   524:       std::vector<MachineInstr *> &backtrack);
   525: 
   526:   // Check for conditional register operaton.
   527:   bool MIsCondAssign(MachineInstr *BMI, MachineInstr *MI,
   528:                      SmallVector<unsigned, 4> &Defs);
   529: 
   530:   // Test all the conditions required for instruction to be
   531:   // speculative. These are just required conditions, cost
   532:   // or benefit should be computed elsewhere.
   533:   bool canMIBeSpeculated(MachineInstr *MI, MachineBasicBlock *ToBB,
   534:                          MachineBasicBlock *FromBB,
   535:                          std::vector<MachineInstr *> &backtrack);
   536: 
   537:   // See if this branch target belongs to the current region.
   538:   bool isBranchWithinRegion(BasicBlockRegion *CurrentRegion, MachineInstr *MI);
   539: 
   540:   // A collection of low level utilities.
   541:   bool MIsAreDependent(MachineInstr *MIa, MachineInstr *MIb);
   542:   bool MIsHaveTrueDependency(MachineInstr *MIa, MachineInstr *MIb);
   543:   bool canReorderMIs(MachineInstr *MIa, MachineInstr *MIb);
   544:   bool canCauseStall(MachineInstr *MI, MachineInstr *MJ);
   545:   bool canThisMIBeMoved(MachineInstr *MI,
   546:                         MachineBasicBlock::iterator &WorkPoint,
   547:                         bool &MovingDependentOp, int &Cost);
   548:   bool MIisDualJumpCandidate(MachineInstr *MI,
   549:                              MachineBasicBlock::iterator &WorkPoint);
   550:   bool DemoteToDotOld(MachineInstr *MI);
   551:   bool isNewifiable(MachineBasicBlock::instr_iterator MII, unsigned DepReg,
   552:                     MachineInstr *TargetPacket);
   553:   bool IsNewifyStore(MachineInstr *MI);
   554:   bool isJumpOutOfRange(MachineInstr *MI);
   555:   bool IsDualJumpFirstCandidate(MachineInstr *MI);
   556:   bool IsDualJumpFirstCandidate(MachineBasicBlock *MBB);
   557:   bool IsDualJumpFirstCandidate(MachineBasicBlock::iterator &TargetPacket);
   558:   bool IsNotDualJumpFirstCandidate(MachineInstr *MI);
   559:   bool isJumpOutOfRange(MachineInstr *UnCond, MachineInstr *Cond);
   560:   bool IsDualJumpSecondCandidate(MachineInstr *MI);
   561:   bool tryAllocateResourcesForConstExt(MachineInstr *MI, bool UpdateState);
   562:   bool isCompoundPair(MachineInstr *MIa, MachineInstr *MIb);
   563:   bool doesMIDefinesPredicate(MachineInstr *MI, SmallVector<unsigned, 4> &Defs);
   564:   bool AnalyzeBBBranches(MachineBasicBlock *MBB, MachineBasicBlock *&TBB,
   565:                          MachineInstr *&FirstTerm, MachineBasicBlock *&FBB,
   566:                          MachineInstr *&SecondTerm);
   567:   inline bool multipleBranchesFromToBB(MachineBasicBlock *BB) const;
   568: };
   569: } // namespace
   570: 
   571: INITIALIZE_PASS_BEGIN(HexagonGlobalScheduler, "global-sched",
   572:                       "Hexagon Global Scheduler", false, false)
   573: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   574: INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
   575: INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
   576: INITIALIZE_PASS_DEPENDENCY(MachineBranchProbabilityInfoWrapperPass)
   577: INITIALIZE_PASS_DEPENDENCY(MachineBlockFrequencyInfoWrapperPass)
   578: INITIALIZE_PASS_END(HexagonGlobalScheduler, "global-sched",
   579:                     "Hexagon Global Scheduler", false, false)
   580: 
   581: /// HexagonGlobalSchedulerImpl Ctor.
   582: HexagonGlobalSchedulerImpl::HexagonGlobalSchedulerImpl(
   583:     MachineFunction &MF, MachineLoopInfo &MLI, MachineDominatorTree &MDT,
   584:     AliasAnalysis *AA, const MachineBranchProbabilityInfo *MBPI,
   585:     const MachineBlockFrequencyInfo *MBFI, const MachineRegisterInfo *MRI,
   586:     const MachineFrameInfo &MFI, const HexagonRegisterInfo *QRI)
   587:     : HexagonPacketizerList(MF, MLI, AA, nullptr, false), PullUpRegions(0),
   588:       CurrentState((const HexagonInstrInfo *)TII), AA(AA), MBPI(MBPI),
   589:       MBFI(MBFI), MRI(MRI), MFI(MFI), QRI(QRI), MLI(MLI), MDT(MDT),
   590:       PacketSize(MF.getSubtarget().getSchedModel().IssueWidth) {
   591:   QII = (const HexagonInstrInfo *)TII;
   592:   Ext = BuildMI(MF, DebugLoc(), QII->get(Hexagon::A4_ext));
   593:   Nop = BuildMI(MF, DebugLoc(), QII->get(Hexagon::A2_nop));
   594:   TSchedModel.init(&MF.getSubtarget());
   595: }
   596: 
   597: // Return bundle size without debug instructions.
   598: static unsigned nonDbgBundleSize(MachineBasicBlock::iterator &TargetPacket) {
   599:   MachineBasicBlock::instr_iterator MII = TargetPacket.getInstrIterator();
   600:   MachineBasicBlock::instr_iterator End = MII->getParent()->instr_end();
```
- EN: It opens namespaces (INITIALIZE_PASS_BEGIN) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as optimizeDualJumps, GenUseDefChain, getRegionForMBB, MIUseDefSet, ... (53 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGlobalScheduler, HexagonGlobalSchedulerImpl, HexagonRegisterInfo, HexagonPacketizerList, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（INITIALIZE_PASS_BEGIN），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 optimizeDualJumps, GenUseDefChain, getRegionForMBB, MIUseDefSet, ... (53 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGlobalScheduler, HexagonGlobalSchedulerImpl, HexagonRegisterInfo, HexagonPacketizerList, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```cpp
   601:   unsigned count = 0;
   602:   for (++MII; MII != End && MII->isInsideBundle(); ++MII) {
   603:     if (MII->isDebugInstr())
   604:       continue;
   605:     count++;
   606:   }
   607:   return count;
   608: }
   609: 
   610: /// The pass main entry point.
   611: bool HexagonGlobalScheduler::runOnMachineFunction(MachineFunction &Fn) {
   612:   auto &HST = Fn.getSubtarget<HexagonSubtarget>();
   613:   if (DisablePullUp || !HST.usePackets() || skipFunction(Fn.getFunction()))
   614:     return false;
   615: 
   616:   const MachineRegisterInfo *MRI = &Fn.getRegInfo();
   617:   const MachineFrameInfo &MFI = Fn.getFrameInfo();
   618:   const HexagonRegisterInfo *QRI = HST.getRegisterInfo();
   619:   MachineLoopInfo &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
   620:   MachineDominatorTree &MDT =
   621:       getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
   622:   const MachineBranchProbabilityInfo *MBPI =
   623:       &getAnalysis<MachineBranchProbabilityInfoWrapperPass>().getMBPI();
   624:   const MachineBlockFrequencyInfo *MBFI =
   625:       &getAnalysis<MachineBlockFrequencyInfoWrapperPass>().getMBFI();
   626:   AliasAnalysis *AA = &getAnalysis<AAResultsWrapperPass>().getAAResults();
   627: 
   628:   // Preserve comounds if Opt Size.
   629:   const Function &F = Fn.getFunction();
   630:   if (F.hasOptSize() && PreventCompoundSeparation.getNumOccurrences() == 0)
   631:     PreventCompoundSeparation = true;
   632: 
   633:   // Instantiate the Scheduler.
   634:   HexagonGlobalSchedulerImpl GlobalSchedulerState(Fn, MLI, MDT, AA, MBPI, MBFI,
   635:                                                   MRI, MFI, QRI);
   636: 
   637:   // DFA state table should not be empty.
   638:   assert(GlobalSchedulerState.getResourceTracker() && "Empty DFA table!");
   639: 
   640:   // Loop over all of the basic blocks.
   641:   // PullUp regions are basically traces with no side entrances.
   642:   // Might want to traverse BB by frequency.
   643:   GlobalSchedulerState.checkBundleCounts(Fn);
   644: 
   645:   // Pullup does not handle hazards yet.
   646:   if (!DisablePullUp.getPosition() && ForceNoopHazards)
   647:     return true;
   648: 
   649:   LLVM_DEBUG(GlobalSchedulerState.countCompounds(Fn));
   650:   GlobalSchedulerState.GenUseDefChain(Fn);
   651:   GlobalSchedulerState.formPullUpRegions(Fn);
   652:   GlobalSchedulerState.performPullUp();
   653:   GlobalSchedulerState.performPullUpCFG(Fn);
   654:   if (PostPullUpOpt) {
   655:     GlobalSchedulerState.formPullUpRegions(Fn);
   656:     GlobalSchedulerState.performExposedOptimizations(Fn);
   657:   }
   658:   LLVM_DEBUG(GlobalSchedulerState.countCompounds(Fn));
   659: 
   660:   return true;
   661: }
   662: 
   663: /// Allocate resources (i.e. 4 bytes) for constant extender. If succeess, return
   664: /// true, otherwise, return false.
   665: bool HexagonGlobalSchedulerImpl::tryAllocateResourcesForConstExt(
   666:     MachineInstr *MI, bool UpdateState = true) {
   667:   if (ResourceTracker->canReserveResources(*Ext)) {
   668:     // We do not always want to change the state of ResourceTracker.
   669:     // When we do not want to change it, we need to test for additional
   670:     // corner cases.
   671:     if (UpdateState)
   672:       ResourceTracker->reserveResources(*Ext);
   673:     else if (CurrentPacketMIs.size() >= PacketSize - 1)
   674:       return false;
   675:     return true;
   676:   }
   677: 
   678:   return false;
   679: }
   680: 
   681: static bool IsSchedBarrier(const MachineInstr *MI) {
   682:   return MI->getOpcode() == Hexagon::Y2_barrier;
   683: }
   684: 
   685: static bool IsIndirectCall(const MachineInstr *MI) {
   686:   return MI->getOpcode() == Hexagon::J2_callr;
   687: }
   688: 
   689: #ifndef NDEBUG
   690: static void DumpLinked(MachineInstr *MI) {
   691:   if (MI->isBundledWithPred())
   692:     dbgs() << "^";
   693:   else
   694:     dbgs() << " ";
   695:   if (MI->isBundledWithSucc())
   696:     dbgs() << "v";
   697:   else
   698:     dbgs() << " ";
   699:   MI->dump();
   700: }
   701: 
   702: static void DumpPacket(MachineBasicBlock::instr_iterator MII) {
   703:   if (MII == MachineBasicBlock::instr_iterator()) {
   704:     dbgs() << "\tNULL\n";
   705:     return;
   706:   }
   707:   MachineInstr *MI = &*MII;
   708:   MachineBasicBlock *MBB = MI->getParent();
   709:   // Uninserted instruction.
   710:   if (!MBB) {
   711:     dbgs() << "\tUnattached: ";
   712:     DumpLinked(MI);
   713:     return;
   714:   }
   715:   dbgs() << "\t";
   716:   DumpLinked(MI);
   717:   if (MI->isBundle()) {
   718:     MachineBasicBlock::instr_iterator MIE = MI->getParent()->instr_end();
   719:     for (++MII; MII != MIE && MII->isInsideBundle() && !MII->isBundle();
   720:          ++MII) {
   721:       dbgs() << "\t\t*";
   722:       DumpLinked(&*MII);
   723:     }
   724:   }
   725: }
   726: 
   727: static void DumpPacket(MachineBasicBlock::instr_iterator MII,
   728:                        MachineBasicBlock::instr_iterator BBEnd) {
   729:   if (MII == BBEnd) {
   730:     dbgs() << "\tBBEnd\n";
   731:     return;
   732:   }
   733: 
   734:   DumpPacket(MII);
   735: }
   736: #endif
   737: 
   738: static bool isBranch(MachineInstr *MI) {
   739:   if (MI->isBundle()) {
   740:     MachineBasicBlock::instr_iterator MII = MI->getIterator();
   741:     MachineBasicBlock::instr_iterator MIE = MI->getParent()->instr_end();
   742:     for (++MII; MII != MIE && MII->isInsideBundle() && !MII->isBundle();
   743:          ++MII) {
   744:       if (MII->isBranch())
   745:         return true;
   746:     }
   747:   } else
   748:     return MI->isBranch();
   749:   return false;
   750: }
   751: 
   752: /// Any of those must not be first dual jump. Everything else is OK.
   753: bool HexagonGlobalSchedulerImpl::IsNotDualJumpFirstCandidate(MachineInstr *MI) {
   754:   if (MI->isCall() || (MI->isBranch() && !QII->isPredicated(*MI)) ||
   755:       MI->isReturn() || QII->isEndLoopN(MI->getOpcode()))
   756:     return true;
   757:   return false;
   758: }
   759: 
   760: /// These four functions clearly belong in HexagonInstrInfo.cpp.
   761: /// Is this MI could be first dual jump instruction?
   762: bool HexagonGlobalSchedulerImpl::IsDualJumpFirstCandidate(MachineInstr *MI) {
   763:   if (!PerformDualJumps)
   764:     return false;
   765:   if (MI->isBranch() && QII->isPredicated(*MI) && !QII->isNewValueJump(*MI) &&
   766:       !MI->isIndirectBranch() && !QII->isEndLoopN(MI->getOpcode()))
   767:     return true;
   768:   // Missing loopN here, but not sure if there will be any benefit from it.
   769:   return false;
   770: }
   771: 
   772: /// This version covers the whole packet.
   773: bool HexagonGlobalSchedulerImpl::IsDualJumpFirstCandidate(
   774:     MachineBasicBlock::iterator &TargetPacket) {
   775:   if (!PerformDualJumps)
   776:     return false;
   777:   MachineInstr *MI = &*TargetPacket;
   778: 
   779:   if (MI->isBundle()) {
   780:     // If this is a bundle, it must be the last bundle in BB.
   781:     if (&(*MI->getParent()->rbegin()) != MI)
   782:       return false;
   783: 
   784:     MachineBasicBlock::instr_iterator MII = MI->getIterator();
   785:     MachineBasicBlock::instr_iterator BBEnd = MI->getParent()->instr_end();
   786:     // If there is a control flow op in this packet, this is the case
   787:     // we look for, even if they are dependent on other members.
   788:     for (++MII; MII != BBEnd && MII->isInsideBundle() && !MII->isBundle();
   789:          ++MII)
   790:       if (IsNotDualJumpFirstCandidate(&*MII))
   791:         return false;
   792:   } else
   793:     return IsDualJumpFirstCandidate(MI);
   794: 
   795:   return true;
   796: }
   797: 
   798: /// This version cover whole BB. There could be a BB
   799: /// with no control flow in it. In this case we can still pull-up a jump
   800: /// into it. Negative proof.
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as isInsideBundle, HexagonGlobalScheduler::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getRegInfo, ... (33 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 isInsideBundle, HexagonGlobalScheduler::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getRegInfo, ... (33 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801: bool HexagonGlobalSchedulerImpl::IsDualJumpFirstCandidate(
   802:     MachineBasicBlock *MBB) {
   803:   if (!PerformDualJumps)
   804:     return false;
   805: 
   806:   for (MachineBasicBlock::instr_iterator MII = MBB->instr_begin(),
   807:                                          MBBEnd = MBB->instr_end();
   808:        MII != MBBEnd; ++MII) {
   809:     MachineInstr *MI = &*MII;
   810:     if (MI->isDebugInstr())
   811:       continue;
   812:     if (!MI->isBundle() && IsNotDualJumpFirstCandidate(MI))
   813:       return false;
   814:   }
   815:   return true;
   816: }
   817: 
   818: /// Is this MI could be second dual jump instruction?
   819: bool HexagonGlobalSchedulerImpl::IsDualJumpSecondCandidate(MachineInstr *MI) {
   820:   if (!PerformDualJumps)
   821:     return false;
   822:   if ((MI->isBranch() && !QII->isNewValueJump(*MI) && !MI->isIndirectBranch() &&
   823:        !QII->isEndLoopN(MI->getOpcode())) ||
   824:       (MI->isCall() && !IsIndirectCall(MI)))
   825:     return true;
   826:   return false;
   827: }
   828: 
   829: // Since we have no exact knowledge of code layout,
   830: // allow some safety buffer for jump target.
   831: // This is measured in bytes.
   832: static const unsigned SafetyBuffer = 200;
   833: 
   834: static MachineBasicBlock::instr_iterator
   835: getHexagonFirstInstrTerminator(MachineBasicBlock *MBB) {
   836:   MachineBasicBlock::instr_iterator MIB = MBB->instr_begin();
   837:   MachineBasicBlock::instr_iterator MIE = MBB->instr_end();
   838:   MachineBasicBlock::instr_iterator MII = MIB;
   839:   while (MII != MIE) {
   840:     if (!MII->isBundle() && MII->isTerminator())
   841:       return MII;
   842:     ++MII;
   843:   }
   844:   return MIE;
   845: }
   846: 
   847: /// Check if a given instruction is:
   848: /// - a jump to a distant target
   849: /// - that exceeds its immediate range
   850: /// If both conditions are true, it requires constant extension.
   851: bool HexagonGlobalSchedulerImpl::isJumpOutOfRange(MachineInstr *MI) {
   852:   if (!MI || !MI->isBranch())
   853:     return false;
   854:   MachineBasicBlock *MBB = MI->getParent();
   855:   auto FirstTerm = getHexagonFirstInstrTerminator(MBB);
   856:   if (FirstTerm == MBB->instr_end())
   857:     return false;
   858: 
   859:   unsigned InstOffset = BlockToInstOffset[MBB];
   860:   unsigned Distance = 0;
   861:   MachineBasicBlock::instr_iterator FTMII = FirstTerm;
   862: 
   863:   // To save time, estimate exact position of a branch instruction
   864:   // as one at the end of the MBB.
   865:   // Number of instructions times typical instruction size.
   866:   InstOffset += (QII->nonDbgBBSize(MBB) * HEXAGON_INSTR_SIZE);
   867: 
   868:   MachineBasicBlock *TBB = NULL, *FBB = NULL;
   869:   SmallVector<MachineOperand, 4> Cond;
   870: 
   871:   // Try to analyze this branch.
   872:   if (QII->analyzeBranch(*MBB, TBB, FBB, Cond, false)) {
   873:     // Could not analyze it. See if this is something we can recognize.
   874:     // If it is a NVJ, it should always have its target in
   875:     // a fixed location.
   876:     if (QII->isNewValueJump(*FirstTerm))
   877:       TBB = FirstTerm->getOperand(QII->getCExtOpNum(*FirstTerm)).getMBB();
   878:   }
   879:   if (TBB && (MI == &*FirstTerm)) {
   880:     Distance =
   881:         (unsigned)std::abs((long long)InstOffset - BlockToInstOffset[TBB]) +
   882:         SafetyBuffer;
   883:     LLVM_DEBUG(dbgs() << "\tFirst term offset(" << Distance << "): ";
   884:                FirstTerm->dump());
   885:     return !QII->isJumpWithinBranchRange(*FirstTerm, Distance);
   886:   }
   887:   if (FBB) {
   888:     // Look for second terminator.
   889:     FTMII++;
   890:     MachineInstr *SecondTerm = &*FTMII;
   891:     assert(FTMII != MBB->instr_end() &&
   892:            (SecondTerm->isBranch() || SecondTerm->isCall()) &&
   893:            "Bad second terminator");
   894:     if (MI != SecondTerm)
   895:       return false;
   896:     // Analyze the second branch in the BB.
   897:     Distance =
   898:         (unsigned)std::abs((long long)InstOffset - BlockToInstOffset[FBB]) +
   899:         SafetyBuffer;
   900:     LLVM_DEBUG(dbgs() << "\tSecond term offset(" << Distance << "): ";
   901:                FirstTerm->dump());
   902:     return !QII->isJumpWithinBranchRange(*SecondTerm, Distance);
   903:   }
   904:   return false;
   905: }
   906: 
   907: /// Returns true if an instruction can be promoted to .new predicate
   908: /// or new-value store.
   909: /// Performs implicit version checking.
   910: bool HexagonGlobalSchedulerImpl::isNewifiable(
   911:     MachineBasicBlock::instr_iterator MII, unsigned DepReg,
   912:     MachineInstr *TargetPacket) {
   913:   MachineInstr *MI = &*MII;
   914:   if (QII->isDotNewInst(*MI) ||
   915:       !CanNewifiedBeUsedInBundle(MII, DepReg, TargetPacket))
   916:     return false;
   917:   return (QII->isPredicated(*MI) && QII->getDotNewPredOp(*MI, nullptr) > 0) ||
   918:          QII->mayBeNewStore(*MI);
   919: }
   920: 
   921: bool HexagonGlobalSchedulerImpl::DemoteToDotOld(MachineInstr *MI) {
   922:   int NewOpcode = QII->getDotOldOp(*MI);
   923:   MI->setDesc(QII->get(NewOpcode));
   924:   return true;
   925: }
   926: 
   927: // initPacketizerState - Initialize packetizer flags
   928: void HexagonGlobalSchedulerImpl::initPacketizerState(void) {
   929:   CurrentPacketMIs.clear();
   930:   return;
   931: }
   932: 
   933: // ignorePseudoInstruction - Ignore bundling of pseudo instructions.
   934: bool HexagonGlobalSchedulerImpl::ignoreInstruction(MachineInstr *MI) {
   935:   if (MI->isDebugInstr())
   936:     return true;
   937: 
   938:   // We must print out inline assembly
   939:   if (MI->isInlineAsm())
   940:     return false;
   941: 
   942:   // We check if MI has any functional units mapped to it.
   943:   // If it doesn't, we ignore the instruction.
   944:   const MCInstrDesc &TID = MI->getDesc();
   945:   unsigned SchedClass = TID.getSchedClass();
   946:   const InstrStage *IS =
   947:       ResourceTracker->getInstrItins()->beginStage(SchedClass);
   948:   unsigned FuncUnits = IS->getUnits();
   949:   return !FuncUnits;
   950: }
   951: 
   952: // isSoloInstruction: - Returns true for instructions that must be
   953: // scheduled in their own packet.
   954: bool HexagonGlobalSchedulerImpl::isSoloInstruction(const MachineInstr &MI) {
   955:   if (MI.isInlineAsm())
   956:     return true;
   957: 
   958:   if (MI.isEHLabel())
   959:     return true;
   960: 
   961:   // From Hexagon V4 Programmer's Reference Manual 3.4.4 Grouping constraints:
   962:   // trap, pause, barrier, icinva, isync, and syncht are solo instructions.
   963:   // They must not be grouped with other instructions in a packet.
   964:   if (IsSchedBarrier(&MI))
   965:     return true;
   966: 
   967:   if (MI.getOpcode() == Hexagon::A2_nop)
   968:     return true;
   969: 
   970:   return false;
   971: }
   972: 
   973: /// Return region ptr or null if non found.
   974: BasicBlockRegion *HexagonGlobalSchedulerImpl::getRegionForMBB(
   975:     std::vector<BasicBlockRegion *> &Regions, MachineBasicBlock *MBB) {
   976:   for (std::vector<BasicBlockRegion *>::iterator I = Regions.begin(),
   977:                                                  E = Regions.end();
   978:        I != E; ++I) {
   979:     if ((*I)->findMBB(MBB))
   980:       return *I;
   981:   }
   982:   return NULL;
   983: }
   984: 
   985: /// Select best candidate to form regions.
   986: static inline bool selectBestBB(BlockFrequency &BBaFreq, unsigned BBaSize,
   987:                                 BlockFrequency &BBbFreq, unsigned BBbSize) {
   988:   if (BBaFreq.getFrequency() > BBbFreq.getFrequency())
   989:     return true;
   990:   // TODO: This needs fine tuning.
   991:   // if (BBaSize < BBbSize)
   992:   //  return true;
   993:   if (BBaFreq.getFrequency() == BBbFreq.getFrequency())
   994:     return true;
   995:   return false;
   996: }
   997: 
   998: /// Returns BB pointer if one of MBB successors should be added to the
   999: /// current PullUp Region, NULL otherwise.
  1000: /// If SecondBest is defined, get next one after Best match.
```
- EN: It declares or implements routines such as HexagonGlobalSchedulerImpl::IsDualJumpFirstCandidate, HexagonGlobalSchedulerImpl::IsDualJumpSecondCandidate, getHexagonFirstInstrTerminator, instr_begin, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGlobalSchedulerImpl::IsDualJumpFirstCandidate, HexagonGlobalSchedulerImpl::IsDualJumpSecondCandidate, getHexagonFirstInstrTerminator, instr_begin, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001: /// Most of the time, since we practically always have only two successors,
  1002: /// this is "the other" BB successor which still matches original
  1003: /// selection criterion.
  1004: MachineBasicBlock *
  1005: HexagonGlobalSchedulerImpl::getNextPURBB(MachineBasicBlock *MBB,
  1006:                                          bool SecondBest = false) {
  1007:   if (!MBB)
  1008:     return NULL;
  1009: 
  1010:   BlockFrequency BestBlockFreq = BlockFrequency(0);
  1011:   unsigned BestBlockSize = 0;
  1012:   MachineBasicBlock *BestBB = NULL;
  1013:   MachineBasicBlock *SecondBestBB = NULL;
  1014: 
  1015:   // Catch single BB loops.
  1016:   for (MachineBasicBlock *Succ : MBB->successors())
  1017:     if (Succ == MBB)
  1018:       return NULL;
  1019: 
  1020:   // Iterate through successors to MBB.
  1021:   for (MachineBasicBlock *Succ : MBB->successors()) {
  1022:     BlockFrequency BlockFreq = MBFI->getBlockFreq(Succ);
  1023: 
  1024:     LLVM_DEBUG(dbgs() << "\tsucc BB(" << Succ->getNumber() << ") freq("
  1025:                       << BlockFreq.getFrequency() << ")");
  1026: 
  1027:     if (!SecondBest && getRegionForMBB(PullUpRegions, Succ))
  1028:       continue;
  1029: 
  1030:     // If there is more then one predecessor to this block, do not include it.
  1031:     // It means there is a side entrance to it.
  1032:     if (Succ->pred_size() > 1)
  1033:       continue;
  1034: 
  1035:     // If this block is a target of an indirect branch, it should
  1036:     // also not be included.
  1037:     if (Succ->isEHPad() || Succ->hasAddressTaken())
  1038:       continue;
  1039: 
  1040:     // Get BB edge frequency.
  1041:     BlockFrequency EdgeFreq = BlockFreq * MBPI->getEdgeProbability(MBB, Succ);
  1042:     LLVM_DEBUG(dbgs() << "\tedge with freq(" << EdgeFreq.getFrequency()
  1043:                       << ")\n");
  1044: 
  1045:     if (selectBestBB(EdgeFreq, QII->nonDbgBBSize(Succ), BestBlockFreq,
  1046:                      BestBlockSize)) {
  1047:       BestBlockFreq = EdgeFreq;
  1048:       BestBlockSize = QII->nonDbgBBSize(Succ);
  1049:       SecondBestBB = BestBB;
  1050:       BestBB = Succ;
  1051:     } else if (!SecondBestBB) {
  1052:       SecondBestBB = Succ;
  1053:     }
  1054:   }
  1055:   if (SecondBest)
  1056:     return SecondBestBB;
  1057:   else
  1058:     return BestBB;
  1059: }
  1060: 
  1061: /// Form region to perform pull-up.
  1062: bool HexagonGlobalSchedulerImpl::formPullUpRegions(MachineFunction &Fn) {
  1063:   const Function &F = Fn.getFunction();
  1064:   // Check for single-block functions and skip them.
  1065:   if (std::next(F.begin()) == F.end())
  1066:     return false;
  1067: 
  1068:   // Compute map for BB distances.
  1069:   // Offset of the current instruction from the start.
  1070:   unsigned InstOffset = 0;
  1071: 
  1072:   LLVM_DEBUG(dbgs() << "****** Form PullUpRegions **************\n");
  1073:   // Loop over all basic blocks.
  1074:   // PullUp regions are basically traces with no side entrances.
  1075:   for (MachineFunction::iterator MBB = Fn.begin(), MBBe = Fn.end(); MBB != MBBe;
  1076:        ++MBB) {
  1077:     if (MBB->getAlignment() > llvm::Align(1)) {
  1078:       // Although we don't know the exact layout of the final code, we need
  1079:       // to account for alignment padding somehow. This heuristic pads each
  1080:       // aligned basic block according to the alignment value.
  1081:       int ByteAlign = MBB->getAlignment().value() - 1;
  1082:       InstOffset = (InstOffset + ByteAlign) & ~(ByteAlign);
  1083:     }
  1084:     // Remember BB layout offset.
  1085:     BlockToInstOffset[&*MBB] = InstOffset;
  1086:     for (MachineBasicBlock::instr_iterator MII = MBB->instr_begin(),
  1087:                                            MIE = MBB->instr_end();
  1088:          MII != MIE; ++MII)
  1089:       if (!MII->isBundle())
  1090:         InstOffset += QII->getSize(*MII);
  1091: 
  1092:     // If this BB is already in a region, move on.
  1093:     if (getRegionForMBB(PullUpRegions, &*MBB))
  1094:       continue;
  1095: 
  1096:     LLVM_DEBUG(dbgs() << "\nRoot BB(" << MBB->getNumber() << ") name("
  1097:                       << MBB->getName() << ") size(" << QII->nonDbgBBSize(&*MBB)
  1098:                       << ") freq(" << printBlockFreq(*MBFI, *MBB)
  1099:                       << ") pred_size(" << MBB->pred_size() << ") in_func("
  1100:                       << MBB->getParent()->getFunction().getName() << ")\n");
  1101: 
  1102:     BasicBlockRegion *PUR = new BasicBlockRegion(TII, QRI, &*MBB);
  1103:     PullUpRegions.push_back(PUR);
  1104: 
  1105:     for (MachineBasicBlock *MBBR = getNextPURBB(&*MBB); MBBR;
  1106:          MBBR = getNextPURBB(MBBR)) {
  1107:       LLVM_DEBUG(dbgs() << "Add BB(" << MBBR->getNumber() << ") name("
  1108:                         << MBBR->getName() << ") size("
  1109:                         << QII->nonDbgBBSize(MBBR) << ") freq("
  1110:                         << printBlockFreq(*MBFI, *MBBR) << ") in_func("
  1111:                         << MBBR->getParent()->getFunction().getName() << ")\n");
  1112:       PUR->addBBtoRegion(MBBR);
  1113:     }
  1114:   }
  1115:   return true;
  1116: }
  1117: 
  1118: /// Return true if MI is an instruction we are unable to reason about
  1119: /// (like something with unmodeled memory side effects).
  1120: static inline bool isGlobalMemoryObject(MachineInstr *MI) {
  1121:   if (MI->hasUnmodeledSideEffects() || MI->hasOrderedMemoryRef() ||
  1122:       MI->isCall() ||
  1123:       (MI->getOpcode() == Hexagon::J2_jump && !MI->getOperand(0).isMBB()))
  1124:     return true;
  1125:   return false;
  1126: }
  1127: 
  1128: // This MI might have either incomplete info, or known to be unsafe
  1129: // to deal with (i.e. volatile object).
  1130: static inline bool isUnsafeMemoryObject(MachineInstr *MI) {
  1131:   if (!MI || MI->memoperands_empty())
  1132:     return true;
  1133: 
  1134:   // We purposefully do no check for hasOneMemOperand() here
  1135:   // in hope to trigger an assert downstream in order to
  1136:   // finish implementation.
  1137:   if ((*MI->memoperands_begin())->isVolatile() || MI->hasUnmodeledSideEffects())
  1138:     return true;
  1139: 
  1140:   if (!(*MI->memoperands_begin())->getValue())
  1141:     return true;
  1142: 
  1143:   return false;
  1144: }
  1145: 
  1146: /// This returns true if the two MIs could be memory dependent.
  1147: static bool MIsNeedChainEdge(AliasAnalysis *AA, const TargetInstrInfo *TII,
  1148:                              MachineInstr *MIa, MachineInstr *MIb) {
  1149:   // Cover a trivial case - no edge is need to itself.
  1150:   if (MIa == MIb)
  1151:     return false;
  1152: 
  1153:   if (TII->areMemAccessesTriviallyDisjoint(*MIa, *MIb))
  1154:     return false;
  1155: 
  1156:   if (isUnsafeMemoryObject(MIa) || isUnsafeMemoryObject(MIb))
  1157:     return true;
  1158: 
  1159:   // If we are dealing with two "normal" loads, we do not need an edge
  1160:   // between them - they could be reordered.
  1161:   if (!MIa->mayStore() && !MIb->mayStore())
  1162:     return false;
  1163: 
  1164:   // To this point analysis is generic. From here on we do need AA.
  1165:   if (!AA)
  1166:     return true;
  1167: 
  1168:   MachineMemOperand *MMOa = *MIa->memoperands_begin();
  1169:   MachineMemOperand *MMOb = *MIb->memoperands_begin();
  1170: 
  1171:   // TODO: Need to handle multiple memory operands.
  1172:   // if either instruction has more than one memory operand, punt.
  1173:   if (!(MIa->hasOneMemOperand() && MIb->hasOneMemOperand()))
  1174:     return true;
  1175: 
  1176:   if (!MMOa->getSize().hasValue() || !MMOb->getSize().hasValue())
  1177:     return true;
  1178: 
  1179:   assert((MMOa->getOffset() >= 0) && "Negative MachineMemOperand offset");
  1180:   assert((MMOb->getOffset() >= 0) && "Negative MachineMemOperand offset");
  1181:   assert((MMOa->getSize().hasValue() && MMOb->getSize().hasValue()) &&
  1182:          "Size 0 memory access");
  1183: 
  1184:   // If the base address of the two memoperands is the same. For instance,
  1185:   // x and x+4, then we can easily reason about them using the offset and size
  1186:   // of access.
  1187:   if (MMOa->getValue() == MMOb->getValue()) {
  1188:     if (MMOa->getOffset() > MMOb->getOffset()) {
  1189:       uint64_t offDiff = MMOa->getOffset() - MMOb->getOffset();
  1190:       return !(MMOb->getSize().getValue() <= offDiff);
  1191:     } else if (MMOa->getOffset() < MMOb->getOffset()) {
  1192:       uint64_t offDiff = MMOb->getOffset() - MMOa->getOffset();
  1193:       return !(MMOa->getSize().getValue() <= offDiff);
  1194:     }
  1195:     // MMOa->getOffset() == MMOb->getOffset()
  1196:     return true;
  1197:   }
  1198: 
  1199:   int64_t MinOffset = std::min(MMOa->getOffset(), MMOb->getOffset());
  1200:   int64_t Overlapa = MMOa->getSize().getValue() + MMOa->getOffset() - MinOffset;
```
- EN: It declares or implements routines such as HexagonGlobalSchedulerImpl::getNextPURBB, BlockFrequency, getBlockFreq, LLVM_DEBUG, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGlobalSchedulerImpl::getNextPURBB, BlockFrequency, getBlockFreq, LLVM_DEBUG, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:   int64_t Overlapb = MMOb->getSize().getValue() + MMOb->getOffset() - MinOffset;
  1202: 
  1203:   AliasResult AAResult =
  1204:       AA->alias(MemoryLocation(MMOa->getValue(), Overlapa, MMOa->getAAInfo()),
  1205:                 MemoryLocation(MMOb->getValue(), Overlapb, MMOb->getAAInfo()));
  1206: 
  1207:   return (AAResult != AliasResult::NoAlias);
  1208: }
  1209: 
  1210: /// Gather register def/uses from MI.
  1211: /// This treats possible (predicated) defs
  1212: /// as actually happening ones (conservatively).
  1213: static inline void parseOperands(MachineInstr *MI,
  1214:                                  SmallVector<unsigned, 4> &Defs,
  1215:                                  SmallVector<unsigned, 8> &Uses) {
  1216:   Defs.clear();
  1217:   Uses.clear();
  1218: 
  1219:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
  1220:     const MachineOperand &MO = MI->getOperand(i);
  1221: 
  1222:     if (MO.isReg()) {
  1223:       unsigned Reg = MO.getReg();
  1224:       if (!Reg)
  1225:         continue;
  1226:       assert(Register::isPhysicalRegister(Reg));
  1227:       if (MO.isUse())
  1228:         Uses.push_back(MO.getReg());
  1229:       if (MO.isDef())
  1230:         Defs.push_back(MO.getReg());
  1231:     } else if (MO.isRegMask()) {
  1232:       for (unsigned R = 1, NR = Hexagon::NUM_TARGET_REGS; R != NR; ++R)
  1233:         if (MO.clobbersPhysReg(R))
  1234:           Defs.push_back(R);
  1235:     }
  1236:   }
  1237: }
  1238: 
  1239: void HexagonGlobalSchedulerImpl::MIUseDefSet(MachineInstr *MI,
  1240:                                              std::vector<unsigned> &Defs,
  1241:                                              std::vector<unsigned> &Uses) {
  1242:   Defs.clear();
  1243:   Uses.clear();
  1244:   assert(!MI->isBundle() && "Cannot parse regs of a bundle.");
  1245:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
  1246:     const MachineOperand &MO = MI->getOperand(i);
  1247: 
  1248:     if (MO.isReg()) {
  1249:       unsigned Reg = MO.getReg();
  1250:       if (!Reg)
  1251:         continue;
  1252:       assert(Register::isPhysicalRegister(Reg));
  1253:       std::vector<unsigned> &Refs = MO.isUse() ? Uses : Defs;
  1254:       for (MCRegAliasIterator AI(MO.getReg(), QRI, true); AI.isValid(); ++AI)
  1255:         Refs.push_back(*AI);
  1256:     } else if (MO.isRegMask()) {
  1257:       for (unsigned R = 1, NR = Hexagon::NUM_TARGET_REGS; R != NR; ++R)
  1258:         if (MO.clobbersPhysReg(R))
  1259:           Defs.push_back(R);
  1260:     }
  1261:   }
  1262: }
  1263: 
  1264: /// Some apparent dependencies are not actually restricting us since there
  1265: /// is a delay between assignment and actual usage, like in case of a call.
  1266: /// There could be more cases here, but this one seems the most obvious.
  1267: static bool isDelayedUseException(MachineInstr *MIa, MachineInstr *MIb) {
  1268:   if (MIa->isCall() && !MIb->isCall())
  1269:     return true;
  1270:   if (!MIa->isCall() && MIb->isCall())
  1271:     return true;
  1272:   return false;
  1273: }
  1274: 
  1275: /// This is a check for resources availability and dependency
  1276: /// for an MI being tried for an existing bundle.
  1277: /// This is needed because we can:
  1278: ///   - save time by filtering out trivial cases
  1279: ///   - we want to reuse infrastructure that does not really knows
  1280: ///     how to deal with parallel semantics of a bundle that already
  1281: ///     exists. For instance, the following case:
  1282: ///     SI %R6<def> = L2_ploadrif_io %P0<kill>, %R7, 4;
  1283: ///     SJ %R6<def> = A2_tfr %R0;
  1284: ///     will be happily allowed by isLegalToPacketizeTogether since in serial
  1285: ///     semantics it never happens, and even if it does, it is legal. Not so
  1286: ///     for when we __speculatively__ trying and MI for a bundle.
  1287: ///
  1288: /// Note:  This is not equivalent to MIsAreDependent().
  1289: /// MIsAreDependent only understands serial semantics.
  1290: /// These are OK to packetize together:
  1291: ///  %R0<def> = L2_loadri_io %R18, 76; mem:LD4[%sunkaddr226](tbaa=!"int")
  1292: ///  %R2<def> = ASL %R0<kill>, 3; flags:  Inside bundle
  1293: ///
  1294: bool HexagonGlobalSchedulerImpl::canAddMIToThisPacket(
  1295:     MachineInstr *MI,
  1296:     SmallVector<MachineInstr *, HEXAGON_PACKET_SIZE> &Bundle) {
  1297:   if (!MI)
  1298:     return false;
  1299:   LLVM_DEBUG(dbgs() << "\n\t[canAddMIToThisPacket]: "; MI->dump());
  1300: 
  1301:   // Const extenders need custom resource checking...
  1302:   // Should be OK if we can update the check everywhere.
  1303:   if ((QII->isConstExtended(*MI) || QII->isExtended(*MI) ||
  1304:        isJumpOutOfRange(MI)) &&
  1305:       !tryAllocateResourcesForConstExt(MI, false))
  1306:     return false;
  1307: 
  1308:   // Ask DFA if machine resource is available for MI.
  1309:   if (!ResourceTracker->canReserveResources(*MI) || !shouldAddToPacket(*MI)) {
  1310:     LLVM_DEBUG(dbgs() << "\tNo DFA resources.\n");
  1311:     return false;
  1312:   }
  1313: 
  1314:   SmallVector<unsigned, 4> BundleDefs;
  1315:   SmallVector<unsigned, 8> BundleUses;
  1316:   SmallVector<unsigned, 4> Defs;
  1317:   SmallVector<unsigned, 8> Uses;
  1318:   MachineInstr *FirstCompound = NULL, *SecondCompound = NULL;
  1319:   MachineInstr *FirstDuplex = NULL, *SecondDuplex = NULL;
  1320: 
  1321:   parseOperands(MI, Defs, Uses);
  1322:   for (SmallVector<MachineInstr *, HEXAGON_PACKET_SIZE>::iterator
  1323:            BI = Bundle.begin(),
  1324:            BE = Bundle.end();
  1325:        BI != BE; ++BI) {
  1326:     BundleDefs.clear();
  1327:     BundleUses.clear();
  1328:     parseOperands(*BI, BundleDefs, BundleUses);
  1329: 
  1330:     MachineInstr *Inst1 = *BI;
  1331:     MachineInstr *Inst2 = MI;
  1332: 
  1333:     if (Inst1->getParent() && OneFloatPerPacket && QII->isFloat(*Inst1) &&
  1334:         QII->isFloat(*Inst2))
  1335:       return false;
  1336: 
  1337:     if (Inst1->getParent() && OneComplexPerPacket && QII->isComplex(*Inst1) &&
  1338:         QII->isComplex(*Inst2))
  1339:       return false;
  1340: 
  1341:     if (PreventCompoundSeparation)
  1342:       if (QII->getCompoundCandidateGroup(**BI)) {
  1343:         if (!FirstCompound)
  1344:           FirstCompound = *BI;
  1345:         else {
  1346:           SecondCompound = *BI;
  1347:           if (isCompoundPair(FirstCompound, SecondCompound)) {
  1348:             if (MI->mayLoad() || MI->mayStore()) {
  1349:               LLVM_DEBUG(dbgs() << "\tPrevent compound destruction.\n");
  1350:               return false;
  1351:             }
  1352:           }
  1353:         }
  1354:       }
  1355:     if (PreventDuplexSeparation)
  1356:       if (QII->getDuplexCandidateGroup(**BI)) {
  1357:         if (!FirstDuplex)
  1358:           FirstDuplex = *BI;
  1359:         else {
  1360:           SecondDuplex = *BI;
  1361:           if (QII->isDuplexPair(*FirstDuplex, *SecondDuplex)) {
  1362:             if (MI->mayLoad() || MI->mayStore()) {
  1363:               LLVM_DEBUG(dbgs() << "\tPrevent duplex destruction.\n");
  1364:               return false;
  1365:             }
  1366:           }
  1367:         }
  1368:       }
  1369: 
  1370:     for (unsigned i = 0; i < Defs.size(); i++) {
  1371:       // Check for multiple definitions in the same packet.
  1372:       for (unsigned j = 0; j < BundleDefs.size(); j++)
  1373:         // Multiple defs in the same packet.
  1374:         // Calls are OK here.
  1375:         // Also if we have multiple defs of PC, this simply means we are
  1376:         // dealing with dual jumps.
  1377:         if (AliasingRegs(Defs[i], BundleDefs[j]) &&
  1378:             !isDelayedUseException(MI, *BI) &&
  1379:             !(IsDualJumpFirstCandidate(*BI) && IsDualJumpSecondCandidate(MI))) {
  1380:           LLVM_DEBUG(dbgs() << "\tMultiple defs.\n\t"; MI->dump();
  1381:                      dbgs() << "\t"; (*BI)->dump());
  1382:           return false;
  1383:         }
  1384: 
  1385:       // See if we are creating a swap case as we go, and disallow
  1386:       // it for now.
  1387:       // Also, this is not OK:
  1388:       //  if (!p0) r7 = r5
  1389:       //  if (!p0) r5 = #0
  1390:       // But this is fine:
  1391:       //  if (!p0) r7 = r5
  1392:       //  if (p0) r5 = #0
  1393:       // Aslo - this is not a swap, but an opportunity to newify:
  1394:       // %P1<def> = C2_cmpeqi %R0, 0; flags:
  1395:       // %R0<def> = L2_ploadrif_io %P1<kill>, %R29, 8;
  1396:       // TODO: Handle this.
  1397:       for (unsigned j = 0; j < BundleUses.size(); j++)
  1398:         if (AliasingRegs(Defs[i], BundleUses[j])) {
  1399:           for (unsigned k = 0; k < BundleDefs.size(); k++)
  1400:             for (unsigned l = 0; l < Uses.size(); l++) {
```
- EN: It declares or implements routines such as alias, possible, clear, getOperand, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 alias, possible, clear, getOperand, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:               if (AliasingRegs(BundleDefs[k], Uses[l]) &&
  1402:                   !isDelayedUseException(MI, *BI)) {
  1403:                 LLVM_DEBUG(dbgs() << "\tSwap detected:\n\t"; MI->dump();
  1404:                            dbgs() << "\t"; (*BI)->dump());
  1405:                 return false;
  1406:               }
  1407:             }
  1408:         }
  1409:     }
  1410: 
  1411:     for (unsigned i = 0; i < Uses.size(); i++) {
  1412:       // Check for true data dependency.
  1413:       for (unsigned j = 0; j < BundleDefs.size(); j++)
  1414:         if (AliasingRegs(Uses[i], BundleDefs[j]) &&
  1415:             !isDelayedUseException(MI, *BI)) {
  1416:           LLVM_DEBUG(dbgs() << "\tImmediate Use detected on reg("
  1417:                             << printReg(Uses[i], QRI) << ")\n\t";
  1418:                      MI->dump(); dbgs() << "\t"; (*BI)->dump());
  1419:           // TODO: This could be an opportunity for newifying:
  1420:           //  %P0<def> = C2_cmpeqi %R26, 0
  1421:           //  %R26<def> = A2_tfr %R0<kill>
  1422:           // if (CanPromoteToDotNew(MI, Uses[i]))
  1423:           //  LLVM_DEBUG(dbgs() << "\tCan promoto to .new form.\n");
  1424:           // else
  1425:           return false;
  1426:         }
  1427:     }
  1428: 
  1429:     // For calls we also check callee save regs.
  1430:     if ((*BI)->isCall()) {
  1431:       for (const uint16_t *I = QRI->getCalleeSavedRegs(&MF); *I; ++I) {
  1432:         for (unsigned i = 0; i < Defs.size(); i++) {
  1433:           if (AliasingRegs(Defs[i], *I)) {
  1434:             LLVM_DEBUG(dbgs() << "\tAlias with call.\n");
  1435:             return false;
  1436:           }
  1437:         }
  1438:       }
  1439:     }
  1440: 
  1441:     // If this is return, we are probably speculating (otherwise
  1442:     // we could not pull in there) and will not win from pulling
  1443:     // into this location anyhow.
  1444:     // Example: a side exit.
  1445:     // if (!p0) dealloc_return
  1446:     // TODO: Can check that we do not overwrite return value
  1447:     // and proceed.
  1448:     if ((*BI)->isBarrier()) {
  1449:       LLVM_DEBUG(dbgs() << "\tBarrier interference.\n");
  1450:       return false;
  1451:     }
  1452: 
  1453:     // \ref-manual (7.3.4) A loop setup packet in loopN or spNloop0 cannot
  1454:     // contain a speculative indirect jump,
  1455:     // a new-value compare jump or a dealloc_return.
  1456:     // Speculative indirect jumps (predicate + .new + indirect):
  1457:     // if ([!]Ps.new) jumpr:t Rs
  1458:     // if ([!]Ps.new) jumpr:nt Rs
  1459:     // @note: We don't want to pull across a call to be on the safe side.
  1460:     if (QII->isLoopN(*MI) &&
  1461:         ((QII->isPredicated(**BI) && QII->isPredicatedNew(**BI) &&
  1462:           QII->isJumpR(**BI)) ||
  1463:          QII->isNewValueJump(**BI) || QII->isDeallocRet(**BI) ||
  1464:          (*BI)->isCall())) {
  1465:       LLVM_DEBUG(dbgs() << "\tLoopN pull interference.\n");
  1466:       return false;
  1467:     }
  1468: 
  1469:     // The opposite is also true.
  1470:     if (QII->isLoopN(**BI) &&
  1471:         ((QII->isPredicated(*MI) && QII->isPredicatedNew(*MI) &&
  1472:           QII->isJumpR(*MI)) ||
  1473:          QII->isNewValueJump(*MI) || QII->isDeallocRet(*MI) || MI->isCall())) {
  1474:       LLVM_DEBUG(dbgs() << "\tResident LoopN.\n");
  1475:       return false;
  1476:     }
  1477: 
  1478:     // @todo \ref-manual 7.6.1
  1479:     // Presence of NVJ adds more restrictions.
  1480:     if (QII->isNewValueJump(**BI) &&
  1481:         (MI->mayStore() || MI->getOpcode() == Hexagon::S2_allocframe ||
  1482:          MI->isCall())) {
  1483:       LLVM_DEBUG(dbgs() << "\tNew val Jump.\n");
  1484:       return false;
  1485:     }
  1486: 
  1487:     // For memory operations, check aliasing.
  1488:     // First, be conservative on these objects. Might be overly constraining,
  1489:     // so recheck.
  1490:     if (isGlobalMemoryObject(*BI) || isGlobalMemoryObject(MI))
  1491:       // Currently it catches things like this:
  1492:       // S2_storerinew_io %R29, 32, %R16
  1493:       // S2_storeri_io %R29, 68, %R0
  1494:       // which we can reason about.
  1495:       // TODO: revisit.
  1496:       return false;
  1497: 
  1498:     // If packet has a new-value store, MI can't be a store instruction.
  1499:     if (QII->isNewValueStore(**BI) && MI->mayStore()) {
  1500:       LLVM_DEBUG(dbgs() << "\tNew Value Store to store.\n");
  1501:       return false;
  1502:     }
  1503: 
  1504:     if ((QII->isMemOp(**BI) && MI->mayStore()) ||
  1505:         (QII->isMemOp(*MI) && (*BI)->mayStore())) {
  1506:       LLVM_DEBUG(
  1507:           dbgs() << "\tSlot 0 not available for store because of memop.\n");
  1508:       return false;
  1509:     }
  1510: 
  1511:     // If any of these is true, check aliasing.
  1512:     if ((MI->mayLoad() && (*BI)->mayStore()) ||
  1513:         (MI->mayStore() && (*BI)->mayLoad()) ||
  1514:         (MI->mayStore() && (*BI)->mayStore())) {
  1515:       if (MIsNeedChainEdge(AA, TII, MI, *BI)) {
  1516:         LLVM_DEBUG(dbgs() << "\tAliasing detected:\n\t"; MI->dump();
  1517:                    dbgs() << "\t"; (*BI)->dump());
  1518:         return false;
  1519:       }
  1520:     }
  1521:     // Do not move an instruction to this packet if this packet
  1522:     // already contains a speculated instruction.
  1523:     std::map<MachineInstr *, MachineBasicBlock *>::iterator MIMoved;
  1524:     MIMoved = SpeculatedIns.find(*BI);
  1525:     if ((MIMoved != SpeculatedIns.end()) &&
  1526:         (MIMoved->second != (*BI)->getParent())) {
  1527:       LLVM_DEBUG(
  1528:           dbgs() << "This packet already contains a speculated instruction";
  1529:           (*BI)->dump(););
  1530:       return false;
  1531:     }
  1532:   }
  1533: 
  1534:   // Do not pull-up vector instructions because these instructions have
  1535:   // multi-cycle latencies, and the pull-up pass doesn't correctly account
  1536:   // for instructions that stall for more than one cycle.
  1537:   if (QII->isHVXVec(*MI))
  1538:     return false;
  1539: 
  1540:   return true;
  1541: }
  1542: 
  1543: /// Test is true if the two MIs cannot be safely reordered.
  1544: bool HexagonGlobalSchedulerImpl::ReorderDependencyTest(MachineInstr *MIa,
  1545:                                                        MachineInstr *MIb) {
  1546:   SmallVector<unsigned, 4> DefsA;
  1547:   SmallVector<unsigned, 4> DefsB;
  1548:   SmallVector<unsigned, 8> UsesA;
  1549:   SmallVector<unsigned, 8> UsesB;
  1550: 
  1551:   parseOperands(MIa, DefsA, UsesA);
  1552:   parseOperands(MIb, DefsB, UsesB);
  1553: 
  1554:   for (SmallVector<unsigned, 4>::iterator IDA = DefsA.begin(),
  1555:                                           IDAE = DefsA.end();
  1556:        IDA != IDAE; ++IDA) {
  1557:     for (SmallVector<unsigned, 8>::iterator IUB = UsesB.begin(),
  1558:                                             IUBE = UsesB.end();
  1559:          IUB != IUBE; ++IUB)
  1560:       // True data dependency.
  1561:       if (AliasingRegs(*IDA, *IUB))
  1562:         return true;
  1563: 
  1564:     for (SmallVector<unsigned, 4>::iterator IDB = DefsB.begin(),
  1565:                                             IDBE = DefsB.end();
  1566:          IDB != IDBE; ++IDB)
  1567:       // Output dependency.
  1568:       if (AliasingRegs(*IDA, *IDB))
  1569:         return true;
  1570:   }
  1571: 
  1572:   for (SmallVector<unsigned, 4>::iterator IDB = DefsB.begin(),
  1573:                                           IDBE = DefsB.end();
  1574:        IDB != IDBE; ++IDB) {
  1575:     for (SmallVector<unsigned, 8>::iterator IUA = UsesA.begin(),
  1576:                                             IUAE = UsesA.end();
  1577:          IUA != IUAE; ++IUA)
  1578:       // True data dependency.
  1579:       if (AliasingRegs(*IDB, *IUA))
  1580:         return true;
  1581:   }
  1582: 
  1583:   // Do not reorder two calls...
  1584:   if (MIa->isCall() && MIb->isCall())
  1585:     return true;
  1586: 
  1587:   // For calls we also check callee save regs.
  1588:   if (MIa->isCall())
  1589:     for (const uint16_t *I = QRI->getCalleeSavedRegs(&MF); *I; ++I) {
  1590:       for (unsigned i = 0; i < DefsB.size(); i++) {
  1591:         if (AliasingRegs(DefsB[i], *I))
  1592:           return true;
  1593:       }
  1594:     }
  1595: 
  1596:   if (MIb->isCall())
  1597:     for (const uint16_t *I = QRI->getCalleeSavedRegs(&MF); *I; ++I) {
  1598:       for (unsigned i = 0; i < DefsA.size(); i++) {
  1599:         if (AliasingRegs(DefsA[i], *I))
  1600:           return true;
```
- EN: It declares or implements routines such as dump, size, LLVM_DEBUG, speculating, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dump, size, LLVM_DEBUG, speculating, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:       }
  1602:     }
  1603: 
  1604:   // For memory operations, check aliasing.
  1605:   // First, be conservative on these objects.
  1606:   // Might be overly constraining, so recheck.
  1607:   if ((isGlobalMemoryObject(MIa)) || (isGlobalMemoryObject(MIb)))
  1608:     return true;
  1609: 
  1610:   // If any of these is true, check aliasing.
  1611:   if (((MIa->mayLoad() && MIb->mayStore()) ||
  1612:        (MIa->mayStore() && MIb->mayLoad()) ||
  1613:        (MIa->mayStore() && MIb->mayStore())) &&
  1614:       MIsNeedChainEdge(AA, TII, MIa, MIb))
  1615:     return true;
  1616: 
  1617:   return false;
  1618: }
  1619: 
  1620: /// Serial semantics.
  1621: bool HexagonGlobalSchedulerImpl::MIsAreDependent(MachineInstr *MIa,
  1622:                                                  MachineInstr *MIb) {
  1623:   if (MIa == MIb)
  1624:     return false;
  1625: 
  1626:   if (ReorderDependencyTest(MIa, MIb)) {
  1627:     LLVM_DEBUG(dbgs() << "\t\t[MIsAreDependent]:\n\t\t"; MIa->dump();
  1628:                dbgs() << "\t\t"; MIb->dump());
  1629:     return true;
  1630:   }
  1631:   return false;
  1632: }
  1633: 
  1634: /// Serial semantics.
  1635: bool HexagonGlobalSchedulerImpl::MIsHaveTrueDependency(MachineInstr *MIa,
  1636:                                                        MachineInstr *MIb) {
  1637:   if (MIa == MIb)
  1638:     return false;
  1639: 
  1640:   SmallVector<unsigned, 4> DefsA;
  1641:   SmallVector<unsigned, 4> DefsB;
  1642:   SmallVector<unsigned, 8> UsesA;
  1643:   SmallVector<unsigned, 8> UsesB;
  1644: 
  1645:   parseOperands(MIa, DefsA, UsesA);
  1646:   parseOperands(MIb, DefsB, UsesB);
  1647: 
  1648:   for (SmallVector<unsigned, 4>::iterator IDA = DefsA.begin(),
  1649:                                           IDAE = DefsA.end();
  1650:        IDA != IDAE; ++IDA) {
  1651:     for (SmallVector<unsigned, 8>::iterator IUB = UsesB.begin(),
  1652:                                             IUBE = UsesB.end();
  1653:          IUB != IUBE; ++IUB)
  1654:       // True data dependency.
  1655:       if (AliasingRegs(*IDA, *IUB))
  1656:         return true;
  1657:   }
  1658:   return false;
  1659: }
  1660: 
  1661: /// Sequential semantics. Can these two MIs be reordered?
  1662: /// Moving MIa from "behind" to "in front" of MIb.
  1663: bool HexagonGlobalSchedulerImpl::canReorderMIs(MachineInstr *MIa,
  1664:                                                MachineInstr *MIb) {
  1665:   if (!MIa || !MIb)
  1666:     return false;
  1667: 
  1668:   // Within bundle semantics are parallel.
  1669:   if (MIa->isBundle()) {
  1670:     MachineBasicBlock::instr_iterator MII = MIa->getIterator();
  1671:     MachineBasicBlock::instr_iterator MIIE = MIa->getParent()->instr_end();
  1672:     for (++MII; MII != MIIE && MII->isInsideBundle(); ++MII) {
  1673:       if (MII->isDebugInstr())
  1674:         continue;
  1675:       if (MIsAreDependent(&*MII, MIb))
  1676:         return false;
  1677:     }
  1678:     return true;
  1679:   }
  1680:   return !MIsAreDependent(MIa, MIb);
  1681: }
  1682: 
  1683: static inline bool MIMustNotBePulledUp(MachineInstr *MI) {
  1684:   if (MI->isInlineAsm() || MI->isEHLabel() || IsSchedBarrier(MI))
  1685:     return true;
  1686:   return false;
  1687: }
  1688: 
  1689: static inline bool MIShouldNotBePulledUp(MachineInstr *MI) {
  1690:   if (MI->isBranch() || MI->isReturn() || MI->isCall() || MI->isBarrier() ||
  1691:       MI->isTerminator() || MIMustNotBePulledUp(MI))
  1692:     return true;
  1693:   return false;
  1694: }
  1695: 
  1696: // Only approve dual jump candidate:
  1697: // It is a branch, and we move it to last packet of the target location.
  1698: bool HexagonGlobalSchedulerImpl::MIisDualJumpCandidate(
  1699:     MachineInstr *MI, MachineBasicBlock::iterator &WorkPoint) {
  1700:   if (!PerformDualJumps || !IsDualJumpSecondCandidate(MI) ||
  1701:       MIMustNotBePulledUp(MI) || ignoreInstruction(MI))
  1702:     return false;
  1703: 
  1704:   MachineBasicBlock *FromThisBB = MI->getParent();
  1705:   MachineBasicBlock *ToThisBB = WorkPoint->getParent();
  1706: 
  1707:   LLVM_DEBUG(dbgs() << "\t\t[MIisDualJumpCandidate] To BB("
  1708:                     << ToThisBB->getNumber() << ") From BB("
  1709:                     << FromThisBB->getNumber() << ")\n");
  1710:   // If the question is about the same BB, we do not want to get
  1711:   // dual jump involved - it is a different case.
  1712:   if (FromThisBB == ToThisBB)
  1713:     return false;
  1714: 
  1715:   // Dual jump could only be done on neigboring BBs.
  1716:   // The FromThisBB must only have one predecessor - the basic
  1717:   // block we are trying to merge.
  1718:   if ((*(FromThisBB->pred_begin()) != ToThisBB) ||
  1719:       (std::next(FromThisBB->pred_begin()) != FromThisBB->pred_end()))
  1720:     return false;
  1721: 
  1722:   // If this block is a target of an indirect branch, it should
  1723:   // also not be included.
  1724:   if (FromThisBB->isEHPad() || FromThisBB->hasAddressTaken())
  1725:     return false;
  1726: 
  1727:   // Now we must preserve original fall through paths. In fact we
  1728:   // might be dealing with 3way branching.
  1729:   MachineBasicBlock *ToTBB = NULL, *ToFBB = NULL;
  1730: 
  1731:   if (ToThisBB->succ_size() == 2) {
  1732:     // Check the branch from target block.
  1733:     // If we have two successors, we must understand the branch.
  1734:     SmallVector<MachineOperand, 4> ToCond;
  1735:     if (!QII->analyzeBranch(*ToThisBB, ToTBB, ToFBB, ToCond, false)) {
  1736:       // Have the branch. Check the topology.
  1737:       LLVM_DEBUG(dbgs() << "\t\tToThisBB has two successors: TBB("
  1738:                         << ToTBB->getNumber() << ") and FBB(";
  1739:                  if (ToFBB) dbgs() << ToFBB->getNumber() << ").\n";
  1740:                  else dbgs() << "None"
  1741:                              << ").\n";);
  1742:       if (ToTBB == FromThisBB) {
  1743:         // If the from BB is not the fall through, we can only handle case
  1744:         // when second branch is unconditional jump.
  1745:         return false;
  1746:       } else if (ToFBB == FromThisBB || !ToFBB) {
  1747:         // If the fall through path of ToBB is our FromBB, we have more freedom
  1748:         // of operation.
  1749:         LLVM_DEBUG(dbgs() << "\t\tFall through jump target.\n");
  1750:       }
  1751:     } else {
  1752:       LLVM_DEBUG(dbgs() << "\t\tUnable to analyze first branch.\n");
  1753:       return false;
  1754:     }
  1755:   } else if (ToThisBB->succ_size() == 1) {
  1756:     ToFBB = *ToThisBB->succ_begin();
  1757:     assert(ToFBB == FromThisBB && "Bad CFG layout");
  1758:   } else
  1759:     return false;
  1760: 
  1761:   // First unbundled control flow instruction in the BB.
  1762:   if (!MI->isBundled() && MI == &*FromThisBB->getFirstNonDebugInstr())
  1763:     return IsDualJumpFirstCandidate(WorkPoint);
  1764: 
  1765:   return false;
  1766: }
  1767: 
  1768: // Check whether moving MI to MJ's packet would cause a stall from a previous
  1769: // packet.
  1770: bool HexagonGlobalSchedulerImpl::canCauseStall(MachineInstr *MI,
  1771:                                                MachineInstr *MJ) {
  1772:   SmallVector<unsigned, 4> DefsMJI;
  1773:   SmallVector<unsigned, 8> UsesMJI;
  1774:   SmallVector<unsigned, 4> DefsMI;
  1775:   SmallVector<unsigned, 8> UsesMI;
  1776:   parseOperands(MI, DefsMI, UsesMI);
  1777: 
  1778:   for (auto Use : UsesMI) {
  1779:     int UseIdx = MI->findRegisterUseOperandIdx(Use, /*TRI=*/nullptr);
  1780:     if (UseIdx == -1)
  1781:       continue;
  1782:     bool ShouldBreak = false;
  1783:     int BundleCount = 0;
  1784:     for (MachineBasicBlock::instr_iterator
  1785:              Begin = MJ->getParent()->instr_begin(),
  1786:              MJI = MJ->getIterator();
  1787:          MJI != Begin; --MJI) {
  1788:       if (MJI->isBundle()) {
  1789:         ++BundleCount;
  1790:         continue;
  1791:       }
  1792:       parseOperands(&*MJI, DefsMJI, UsesMJI);
  1793:       for (auto Def : DefsMJI) {
  1794:         if (Def == Use || AliasingRegs(Def, Use)) {
  1795:           int DefIdx = MJI->findRegisterDefOperandIdx(Def, /*TRI=*/nullptr);
  1796:           if (DefIdx >= 0) {
  1797:             int Latency =
  1798:                 TSchedModel.computeOperandLatency(&*MJI, DefIdx, MI, UseIdx);
  1799:             if (Latency > BundleCount)
  1800:               // There will be a stall if MI is moved to MJ's packet.
```
- EN: It declares or implements routines such as HexagonGlobalSchedulerImpl::MIsAreDependent, dump, HexagonGlobalSchedulerImpl::MIsHaveTrueDependency, parseOperands, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGlobalSchedulerImpl::MIsAreDependent, dump, HexagonGlobalSchedulerImpl::MIsHaveTrueDependency, parseOperands, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:               return true;
  1802:             // We found the def for the use and it does not cause a stall.
  1803:             // Continue checking the next use for a potential stall.
  1804:             ShouldBreak = true;
  1805:             break;
  1806:           }
  1807:         }
  1808:       }
  1809:       if (ShouldBreak)
  1810:         break;
  1811:       if (!MJI->isBundled() && !MJI->isDebugInstr())
  1812:         ++BundleCount;
  1813:     }
  1814:   }
  1815:   return false;
  1816: }
  1817: 
  1818: /// Analyze this instruction. If this is an unbundled instruction, see
  1819: /// if it in theory could be packetized.
  1820: /// If it is already part of a packet, see if it has internal
  1821: /// dependencies to this packet.
  1822: bool HexagonGlobalSchedulerImpl::canThisMIBeMoved(
  1823:     MachineInstr *MI, MachineBasicBlock::iterator &WorkPoint,
  1824:     bool &MovingDependentOp, int &Cost) {
  1825:   if (!MI)
  1826:     return false;
  1827:   // By default, it is a normal move.
  1828:   MovingDependentOp = false;
  1829:   Cost = 0;
  1830:   // If MI is a 'formed' compound not potential compound, bail out.
  1831:   if (QII->isCompoundBranchInstr(*MI))
  1832:     return false;
  1833:   // See if we can potentially break potential compound candidates,
  1834:   // and do not do it.
  1835:   if (PreventCompoundSeparation && MI->isBundled()) {
  1836:     enum HexagonII::CompoundGroup MICG = QII->getCompoundCandidateGroup(*MI);
  1837:     if (MICG != HexagonII::HCG_None) {
  1838:       // Check internal dependencies in the bundle.
  1839:       // First, find the bundle header.
  1840:       MachineBasicBlock::instr_iterator MII = MI->getIterator();
  1841:       for (--MII; MII->isBundled(); --MII)
  1842:         if (MII->isBundle())
  1843:           break;
  1844: 
  1845:       MachineBasicBlock::instr_iterator BBEnd = MI->getParent()->instr_end();
  1846:       for (++MII; MII != BBEnd && MII->isInsideBundle() && !MII->isBundle();
  1847:            ++MII) {
  1848:         if (&(*MII) == MI)
  1849:           continue;
  1850:         if (isCompoundPair(&*MII, MI)) {
  1851:           LLVM_DEBUG(dbgs() << "\tPrevent Compound separation.\n");
  1852:           return false;
  1853:         }
  1854:       }
  1855:     }
  1856:   }
  1857:   // Same thing for duplex candidates.
  1858:   if (PreventDuplexSeparation && MI->isBundled()) {
  1859:     if (QII->getDuplexCandidateGroup(*MI) != HexagonII::HSIG_None) {
  1860:       // Check internal dependencies in the bundle.
  1861:       // First, find the bundle header.
  1862:       MachineBasicBlock::instr_iterator MII = MI->getIterator();
  1863:       for (--MII; MII->isBundled(); --MII)
  1864:         if (MII->isBundle())
  1865:           break;
  1866: 
  1867:       MachineBasicBlock::instr_iterator BBEnd = MI->getParent()->instr_end();
  1868:       for (++MII; MII != BBEnd && MII->isInsideBundle() && !MII->isBundle();
  1869:            ++MII) {
  1870:         if ((&(*MII) != MI) && QII->isDuplexPair(*MII, *MI)) {
  1871:           LLVM_DEBUG(dbgs() << "\tPrevent Duplex separation.\n");
  1872:           return false;
  1873:         }
  1874:       }
  1875:     }
  1876:   }
  1877: 
  1878:   // If we perform dual jump formation during the pull-up,
  1879:   // then we want to consider several additional situations.
  1880:   // a) Allow moving of dependent instruction from a packet
  1881:   // b) Allow moving some control flow instructions if they meet
  1882:   //    dual jump criteria.
  1883:   if (MIisDualJumpCandidate(MI, WorkPoint)) {
  1884:     LLVM_DEBUG(dbgs() << "\t\tDual jump candidate:\t"; MI->dump());
  1885:     // Here we are breaking our general assumption about not moving dependent
  1886:     // instructions. To save us two more expensive checks down the line,
  1887:     // propagate the information directly.
  1888:     MovingDependentOp = true;
  1889:     return true;
  1890:   }
  1891: 
  1892:   // Any of these should not even be tried.
  1893:   if (MIShouldNotBePulledUp(MI) || ignoreInstruction(MI))
  1894:     return false;
  1895:   // Pulling up these instructions could put them
  1896:   // out of jump range/offset size.
  1897:   if (QII->isLoopN(*MI)) {
  1898:     unsigned dist_looplabel =
  1899:         BlockToInstOffset.find(MI->getOperand(0).getMBB())->second;
  1900:     unsigned dist_newloop0 =
  1901:         BlockToInstOffset.find(WorkPoint->getParent())->second;
  1902:     // Check if the jump in the last instruction is within range.
  1903:     unsigned Distance =
  1904:         (unsigned)std::abs((long long)dist_looplabel - dist_newloop0) +
  1905:         QII->nonDbgBBSize(WorkPoint->getParent()) * 4 + SafetyBuffer;
  1906:     const HexagonInstrInfo *HII = (const HexagonInstrInfo *)TII;
  1907:     if (!HII->isJumpWithinBranchRange(*MI, Distance)) {
  1908:       LLVM_DEBUG(dbgs() << "\nloopN cannot be moved since Distance: "
  1909:                         << Distance << " outside branch range.";);
  1910:       return false;
  1911:     }
  1912:     LLVM_DEBUG(dbgs() << "\nloopN can be moved since Distance: " << Distance
  1913:                       << " within branch range.";);
  1914:   }
  1915:   // If the def-set of an MI is one of the live-ins then MI should
  1916:   // kill that reg and no instruction before MI should use it.
  1917:   // For simplicity, allow only if MI is the first instruction in the MBB.
  1918:   std::map<MachineInstr *, std::vector<unsigned>>::const_iterator DefIter =
  1919:       MIDefSet.find(MI);
  1920:   MachineBasicBlock *MBB = MI->getParent();
  1921:   for (unsigned i = 0; DefIter != MIDefSet.end() && i < DefIter->second.size();
  1922:        ++i) {
  1923:     if (MBB->isLiveIn(DefIter->second[i]) &&
  1924:         &*MBB->getFirstNonDebugInstr() != MI)
  1925:       return false;
  1926:   }
  1927:   // If it is part of a bundle, analyze it.
  1928:   if (MI->isBundled()) {
  1929:     // Cannot move bundle header itself. This function is about
  1930:     // individual MI move.
  1931:     if (MI->isBundle())
  1932:       return false;
  1933: 
  1934:     // Check internal dependencies in the bundle.
  1935:     // First, find the bundle header.
  1936:     MachineBasicBlock::instr_iterator MII = MI->getIterator();
  1937:     for (--MII; MII->isBundled(); --MII)
  1938:       if (MII->isBundle())
  1939:         break;
  1940: 
  1941:     MachineBasicBlock::instr_iterator BBEnd = MI->getParent()->instr_end();
  1942:     for (++MII; MII != BBEnd && MII->isInsideBundle() && !MII->isBundle();
  1943:          ++MII) {
  1944:       if (MII->isDebugInstr())
  1945:         continue;
  1946:       if (MIsAreDependent(&*MII, MI)) {
  1947:         if (!AllowDependentPullUp) {
  1948:           LLVM_DEBUG(dbgs() << "\t\tDependent.\n");
  1949:           return false;
  1950:         } else {
  1951:           // There are a few cases that we can safely move a dependent
  1952:           // instruction away from this packet.
  1953:           // One example is an instruction setting a call operands.
  1954:           if ((MII->isCall() && !IsIndirectCall(&*MII)) ||
  1955:               IsDualJumpSecondCandidate(&*MII) || MI->isBranch()) {
  1956:             LLVM_DEBUG(dbgs() << "\t\tDependent, but allow to move.\n");
  1957:             MovingDependentOp = true;
  1958:             Cost -= 10;
  1959:             continue;
  1960:           } else {
  1961:             LLVM_DEBUG(dbgs() << "\t\tDependent, and do not allow for now.\n");
  1962:             return false;
  1963:           }
  1964:         }
  1965:       }
  1966:     }
  1967:   }
  1968:   return true;
  1969: }
  1970: 
  1971: /// Return true if MI defines a predicate and parse all defs.
  1972: bool HexagonGlobalSchedulerImpl::doesMIDefinesPredicate(
  1973:     MachineInstr *MI, SmallVector<unsigned, 4> &Defs) {
  1974:   bool defsPredicate = false;
  1975:   Defs.clear();
  1976: 
  1977:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
  1978:     const MachineOperand &MO = MI->getOperand(i);
  1979: 
  1980:     // Regmasks are considered "implicit".
  1981:     if (!MO.isReg())
  1982:       continue;
  1983:     unsigned Reg = MO.getReg();
  1984: 
  1985:     if (!Reg || QRI->isFakeReg(Reg))
  1986:       continue;
  1987: 
  1988:     assert(Register::isPhysicalRegister(Reg));
  1989: 
  1990:     if (MO.isDef() && !MO.isImplicit()) {
  1991:       const TargetRegisterClass *RC = QRI->getMinimalPhysRegClass(Reg);
  1992:       if (RC == &Hexagon::PredRegsRegClass) {
  1993:         defsPredicate = true;
  1994:         Defs.push_back(MO.getReg());
  1995:       }
  1996:     }
  1997:   }
  1998:   return defsPredicate;
  1999: }
  2000: 
```
- EN: It declares types such as HexagonII::CompoundGroup, which carry the state or API of this component. It declares or implements routines such as HexagonGlobalSchedulerImpl::canThisMIBeMoved, getCompoundCandidateGroup, getIterator, isBundled, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 HexagonII::CompoundGroup 等类型，用来承载该组件的状态或接口。 这里声明或实现了 HexagonGlobalSchedulerImpl::canThisMIBeMoved, getCompoundCandidateGroup, getIterator, isBundled, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001: /// We have just tentatively added a predicated MI to an existing packet.
  2002: /// Now we need to determine if it needs to be changed to .new form.
  2003: /// It only handles compare/predicate right now.
  2004: /// TODO - clean this logic up.
  2005: /// TODO - generalize to handle any .new
  2006: bool HexagonGlobalSchedulerImpl::NeedToNewify(
  2007:     MachineBasicBlock::instr_iterator NewMI, unsigned *DepReg,
  2008:     MachineInstr *TargetPacket = NULL) {
  2009:   MachineBasicBlock::instr_iterator MII = NewMI;
  2010:   SmallVector<unsigned, 4> DefsA;
  2011:   SmallVector<unsigned, 4> DefsB;
  2012:   SmallVector<unsigned, 8> UsesB;
  2013: 
  2014:   // If this is not a normal bundle, we are probably
  2015:   // trying to size two lonesome instructions together,
  2016:   // and trying to say if one of them will need to be
  2017:   // newified. In this is the case we have something like this:
  2018:   //   BB#5:
  2019:   //    %P0<def> = CMPGEri %R4, 2
  2020:   //    S2_pstorerif_io %P0<kill>, %R29, 16, %R21<kill>
  2021:   //    BUNDLE %R7<imp-def>, %R4<imp-def>, %R7<imp-use>
  2022:   parseOperands(&*NewMI, DefsB, UsesB);
  2023:   if (TargetPacket && !TargetPacket->isBundled()) {
  2024:     if (doesMIDefinesPredicate(TargetPacket, DefsA)) {
  2025:       for (SmallVector<unsigned, 4>::iterator IA = DefsA.begin(),
  2026:                                               IAE = DefsA.end();
  2027:            IA != IAE; ++IA)
  2028:         for (SmallVector<unsigned, 8>::iterator IB = UsesB.begin(),
  2029:                                                 IBE = UsesB.end();
  2030:              IB != IBE; ++IB)
  2031:           if (*IA == *IB) {
  2032:             *DepReg = *IA;
  2033:             return true;
  2034:           }
  2035:     }
  2036:     return false;
  2037:   }
  2038: 
  2039:   // Find bundle header.
  2040:   for (--MII; MII->isBundled(); --MII)
  2041:     if (MII->isBundle())
  2042:       break;
  2043: 
  2044:   // Iterate down, if there is data dependent cmp found, need to .newify.
  2045:   // Also, we can have the following:
  2046:   //   {
  2047:   //  p0 = r7
  2048:   //  if (!p0.new) jump:t .LBB4_18
  2049:   //  if (p0.new) r8 = zxth(r12)
  2050:   //   }
  2051:   MachineBasicBlock::instr_iterator BBEnd = MII->getParent()->instr_end();
  2052:   for (++MII; MII != BBEnd && MII->isBundled() && !MII->isBundle(); ++MII) {
  2053:     if (MII == NewMI)
  2054:       continue;
  2055:     if (doesMIDefinesPredicate(&*MII, DefsA)) {
  2056:       for (SmallVector<unsigned, 4>::iterator IA = DefsA.begin(),
  2057:                                               IAE = DefsA.end();
  2058:            IA != IAE; ++IA)
  2059:         for (SmallVector<unsigned, 8>::iterator IB = UsesB.begin(),
  2060:                                                 IBE = UsesB.end();
  2061:              IB != IBE; ++IB)
  2062:           // We do not have multiple predicate regs defined in any instruction,
  2063:           // if we ever will, this needs to be generalized.
  2064:           if (*IA == *IB) {
  2065:             *DepReg = *IA;
  2066:             return true;
  2067:           }
  2068:       DefsA.clear();
  2069:     }
  2070:   }
  2071:   LLVM_DEBUG(dbgs() << "\nNo need to newify:"; NewMI->dump());
  2072:   return false;
  2073: }
  2074: 
  2075: /// We know this instruction needs to be newified to be added to the packet,
  2076: /// but not all combinations are legal.
  2077: /// It is a complimentary check to NeedToNewify().
  2078: /// The packet actually contains the new instruction during the check.
  2079: bool HexagonGlobalSchedulerImpl::CanNewifiedBeUsedInBundle(
  2080:     MachineBasicBlock::instr_iterator NewMI, unsigned DepReg,
  2081:     MachineInstr *TargetPacket) {
  2082:   MachineBasicBlock::instr_iterator MII = NewMI;
  2083:   if (!TargetPacket || !TargetPacket->isBundled())
  2084:     return true;
  2085: 
  2086:   // Find the bundle header.
  2087:   for (--MII; MII->isBundled(); --MII)
  2088:     if (MII->isBundle())
  2089:       break;
  2090: 
  2091:   MachineBasicBlock::instr_iterator BBEnd = MII->getParent()->instr_end();
  2092:   for (++MII; MII != BBEnd && MII->isBundled() && !MII->isBundle(); ++MII) {
  2093:     // Effectively we look for the case of late predicates.
  2094:     // No additional checks at the time.
  2095:     if (MII == NewMI || !QII->isPredicateLate(MII->getOpcode()))
  2096:       continue;
  2097:     SmallVector<unsigned, 4> DefsA;
  2098:     if (!doesMIDefinesPredicate(&*MII, DefsA))
  2099:       continue;
  2100:     for (auto &IA : DefsA)
  2101:       if (IA == DepReg)
  2102:         return false;
  2103:   }
  2104:   return true;
  2105: }
  2106: 
  2107: /// setUsed - Set the register and its sub-registers as being used.
  2108: /// Similar to RegScavenger::setUsed().
  2109: void HexagonGlobalSchedulerImpl::setUsedRegs(BitVector &Set, unsigned Reg) {
  2110:   Set.reset(Reg);
  2111:   for (MCSubRegIterator SubRegs(Reg, QRI); SubRegs.isValid(); ++SubRegs)
  2112:     Set.reset(*SubRegs);
  2113: }
  2114: 
  2115: /// Are these two registers overlaping?
  2116: bool HexagonGlobalSchedulerImpl::AliasingRegs(unsigned RegA, unsigned RegB) {
  2117:   if (RegA == RegB)
  2118:     return true;
  2119: 
  2120:   for (MCSubRegIterator SubRegs(RegA, QRI); SubRegs.isValid(); ++SubRegs)
  2121:     if (RegB == *SubRegs)
  2122:       return true;
  2123: 
  2124:   for (MCSubRegIterator SubRegs(RegB, QRI); SubRegs.isValid(); ++SubRegs)
  2125:     if (RegA == *SubRegs)
  2126:       return true;
  2127: 
  2128:   return false;
  2129: }
  2130: 
  2131: /// Find use with this reg, and unmark the kill flag.
  2132: static inline void unmarkKillReg(MachineInstr *MI, unsigned Reg) {
  2133:   if (MI->isDebugInstr())
  2134:     return;
  2135: 
  2136:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
  2137:     MachineOperand &MO = MI->getOperand(i);
  2138: 
  2139:     if (!MO.isReg())
  2140:       continue;
  2141: 
  2142:     if (MO.isKill() && (MO.getReg() == Reg))
  2143:       MO.setIsKill(false);
  2144:   }
  2145: }
  2146: 
  2147: /// Find use with this reg, and unmark the kill flag.
  2148: static inline void markKillReg(MachineInstr *MI, unsigned Reg) {
  2149:   if (MI->isDebugInstr())
  2150:     return;
  2151: 
  2152:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
  2153:     MachineOperand &MO = MI->getOperand(i);
  2154: 
  2155:     if (!MO.isReg())
  2156:       continue;
  2157: 
  2158:     if (MO.isUse() && (MO.getReg() == Reg))
  2159:       MO.setIsKill(true);
  2160:   }
  2161: }
  2162: 
  2163: /// We have just moved an instruction that could have changed kill patterns
  2164: /// along the path it was moved. We need to update it.
  2165: void HexagonGlobalSchedulerImpl::updateKillAlongThePath(
  2166:     MachineBasicBlock *HomeBB, MachineBasicBlock *OriginBB,
  2167:     MachineBasicBlock::instr_iterator &Head,
  2168:     MachineBasicBlock::instr_iterator &Tail,
  2169:     MachineBasicBlock::iterator &SourcePacket,
  2170:     MachineBasicBlock::iterator &TargetPacket,
  2171:     std::vector<MachineInstr *> &backtrack) {
  2172:   // This is the instruction being moved.
  2173:   MachineInstr *MI = &*Head;
  2174:   MachineBasicBlock *CurrentBB = OriginBB;
  2175:   SmallSet<unsigned, 8> KilledUseSet;
  2176: 
  2177:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
  2178:     const MachineOperand &MO = MI->getOperand(i);
  2179:     if (!MO.isReg())
  2180:       continue;
  2181:     unsigned Reg = MO.getReg();
  2182:     if (!Reg)
  2183:       continue;
  2184: 
  2185:     if (MO.isKill())
  2186:       KilledUseSet.insert(Reg);
  2187:   }
  2188: 
  2189:   // If there are no kills here, we are done.
  2190:   if (KilledUseSet.empty())
  2191:     return;
  2192: 
  2193:   LLVM_DEBUG(dbgs() << "\n[updateKillAlongThePath]\n");
  2194:   LLVM_DEBUG(dbgs() << "\t\tInstrToMove   :\t"; MI->dump());
  2195:   LLVM_DEBUG(dbgs() << "\t\tSourceLocation:\n";
  2196:              DumpPacket(SourcePacket.getInstrIterator()));
  2197:   LLVM_DEBUG(dbgs() << "\t\tTargetPacket  :\n";
  2198:              DumpPacket(TargetPacket.getInstrIterator()));
  2199:   LLVM_DEBUG(dbgs() << "\tUpdate Kills. Need to update (" << KilledUseSet.size()
  2200:                     << ")kills. From BB (" << OriginBB->getNumber() << ")\n");
```
- EN: It declares or implements routines such as HexagonGlobalSchedulerImpl::NeedToNewify, parseOperands, isBundled, getParent, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGlobalSchedulerImpl::NeedToNewify, parseOperands, isBundled, getParent, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:   LLVM_DEBUG(dbgs() << "\tMove path:\n");
  2202:   assert(!backtrack.empty() && "Empty back track");
  2203: 
  2204:   // We have pulled up an instruction, with one of its uses marked as kill.
  2205:   // If there is any other use of the same register along the move path,
  2206:   // and there are no side exits with killed register live-in along them,
  2207:   // we need to mark last use of that reg as kill.
  2208:   for (signed i = backtrack.size() - 1; i >= 0; --i) {
  2209:     LLVM_DEBUG(dbgs() << "\t\t[" << i << "]BB("
  2210:                       << backtrack[i]->getParent()->getNumber() << ")\t";
  2211:                backtrack[i]->dump());
  2212:     if (CurrentBB != backtrack[i]->getParent()) {
  2213:       LLVM_DEBUG(dbgs() << "\t\tChange BB from (" << CurrentBB->getNumber()
  2214:                         << ") to(" << backtrack[i]->getParent()->getNumber()
  2215:                         << ")\n");
  2216:       for (MachineBasicBlock::const_succ_iterator
  2217:                SI = backtrack[i]->getParent()->succ_begin(),
  2218:                SE = backtrack[i]->getParent()->succ_end();
  2219:            SI != SE; ++SI) {
  2220:         if (*SI == CurrentBB)
  2221:           continue;
  2222: 
  2223:         LLVM_DEBUG(dbgs() << "\t\tSide Exit:\n\t"; (*SI)->dump());
  2224:         // If any reg kill is live along this side exit, it is not
  2225:         // a kill any more.
  2226:         for (MachineBasicBlock::livein_iterator I = (*SI)->livein_begin(),
  2227:                                                 E = (*SI)->livein_end();
  2228:              I != E; ++I) {
  2229:           if (KilledUseSet.count((*I).PhysReg)) {
  2230:             LLVM_DEBUG(dbgs() << "\t\tReg (" << printReg((*I).PhysReg, QRI)
  2231:                               << ") is LiveIn along side exit.\n");
  2232:             KilledUseSet.erase((*I).PhysReg);
  2233:             unmarkKillReg(MI, (*I).PhysReg);
  2234:           }
  2235:           if (KilledUseSet.empty())
  2236:             return;
  2237:         }
  2238:       }
  2239:       CurrentBB = backtrack[i]->getParent();
  2240:     }
  2241: 
  2242:     // Done with the whole path.
  2243:     if (backtrack[i] == &*TargetPacket)
  2244:       return;
  2245: 
  2246:     // Starting the tracking. Do not update source bundle.
  2247:     // If TargetPacket == SourcePacket we have returned
  2248:     // in the previous check.
  2249:     if (backtrack[i] == &*SourcePacket)
  2250:       continue;
  2251: 
  2252:     // Ignore DBG_VALUE.
  2253:     if (backtrack[i]->isDebugInstr())
  2254:       continue;
  2255: 
  2256:     // Encountered an intermediary bundle. Process it.
  2257:     // Beware, sometimes check for backtrack[i] == TargetPacket
  2258:     // does not work, so this instruction could be one from the target bundle.
  2259:     SmallVector<unsigned, 4> Defs;
  2260:     SmallVector<unsigned, 8> Uses;
  2261:     MachineInstr *MIU = backtrack[i];
  2262:     parseOperands(MIU, Defs, Uses);
  2263: 
  2264:     for (SmallVector<unsigned, 8>::iterator IA = Uses.begin(), IAE = Uses.end();
  2265:          IA != IAE; ++IA) {
  2266:       if (KilledUseSet.count(*IA)) {
  2267:         // Now this is new kill point for this Reg.
  2268:         // Update the bundle, and any local uses.
  2269:         markKillReg(MIU, *IA);
  2270: 
  2271:         // Unmark the current MI.
  2272:         unmarkKillReg(MI, *IA);
  2273: 
  2274:         if (MIU->isBundle()) {
  2275:           // TODO: Can do this cleaner and faster.
  2276:           MachineBasicBlock::instr_iterator MII = MIU->getIterator();
  2277:           MachineBasicBlock::instr_iterator End = CurrentBB->instr_end();
  2278:           for (++MII; MII != End && MII->isInsideBundle(); ++MII)
  2279:             markKillReg(&*MII, *IA);
  2280:         }
  2281: 
  2282:         // We have updated this kill reg, if there are more, keep on going.
  2283:         KilledUseSet.erase(*IA);
  2284: 
  2285:         // If the set is exhausted, just leave.
  2286:         if (KilledUseSet.empty())
  2287:           return;
  2288:       }
  2289:     }
  2290:   }
  2291: }
  2292: 
  2293: /// This is houskeeping for bundle with instruction just added to it.
  2294: void HexagonGlobalSchedulerImpl::addInstructionToExistingBundle(
  2295:     MachineBasicBlock *HomeBB, MachineBasicBlock::instr_iterator &Head,
  2296:     MachineBasicBlock::instr_iterator &Tail,
  2297:     MachineBasicBlock::instr_iterator &NewMI,
  2298:     MachineBasicBlock::iterator &TargetPacket,
  2299:     MachineBasicBlock::iterator &NextMI,
  2300:     std::vector<MachineInstr *> &backtrack) {
  2301:   Tail = getBundleEnd(Head);
  2302:   LLVM_DEBUG(dbgs() << "\t\t\t[Add] Head home: "; DumpPacket(Head));
  2303: 
  2304:   // Old header to be deleted shortly.
  2305:   MachineBasicBlock::instr_iterator Outcast = Head;
  2306:   // Unbundle old header.
  2307:   if (Outcast->isBundle() && Outcast->isBundledWithSucc())
  2308:     Outcast->unbundleFromSucc();
  2309: 
  2310:   bool memShufDisabled = QII->getBundleNoShuf(*Outcast);
  2311: 
  2312:   // Create new bundle header and update MI flags.
  2313:   finalizeBundle(*HomeBB, ++Head, Tail);
  2314:   MachineBasicBlock::instr_iterator BundleMII = std::prev(Head);
  2315:   if (memShufDisabled)
  2316:     QII->setBundleNoShuf(BundleMII);
  2317:   --Head;
  2318: 
  2319:   LLVM_DEBUG(dbgs() << "\t\t\t[Add] New Head : "; DumpPacket(Head));
  2320: 
  2321:   // The old header could be listed in the back tracking,
  2322:   // so if it is, we need to update it.
  2323:   for (unsigned i = 0; i < backtrack.size(); ++i)
  2324:     if (backtrack[i] == &*Outcast)
  2325:       backtrack[i] = &*Head;
  2326: 
  2327:   // Same for top MI iterator.
  2328:   if (NextMI == Outcast)
  2329:     NextMI = Head;
  2330: 
  2331:   TargetPacket = Head;
  2332:   HomeBB->erase(Outcast);
  2333: }
  2334: 
  2335: /// This handles houskeeping for bundle with instruction just deleted from it.
  2336: /// We do not see the original moved instruction in here.
  2337: void HexagonGlobalSchedulerImpl::removeInstructionFromExistingBundle(
  2338:     MachineBasicBlock *HomeBB, MachineBasicBlock::instr_iterator &Head,
  2339:     MachineBasicBlock::instr_iterator &Tail,
  2340:     MachineBasicBlock::iterator &SourceLocation,
  2341:     MachineBasicBlock::iterator &NextMI, bool MovingDependentOp,
  2342:     std::vector<MachineInstr *> &backtrack) {
  2343:   // Empty BBs will be deleted shortly.
  2344:   if (HomeBB->empty()) {
  2345:     Head = MachineBasicBlock::instr_iterator();
  2346:     Tail = MachineBasicBlock::instr_iterator();
  2347:     return;
  2348:   }
  2349: 
  2350:   if (!SourceLocation->isBundle()) {
  2351:     LLVM_DEBUG(dbgs() << "\t\t\tOriginal instruction was not bundled.\n\t\t\t";
  2352:                SourceLocation->dump());
  2353:     // If original instruction was not bundled, and we have moved it
  2354:     // and it is in the back track, we probably want to remove it from there.
  2355:     LLVM_DEBUG(dbgs() << "\t\t\t[Rem] New head: "; backtrack.back()->dump());
  2356: 
  2357:     for (unsigned i = 0; i < backtrack.size(); ++i) {
  2358:       if (backtrack[i] == &*SourceLocation) {
  2359:         // By definition, this should be the last instruction in the backtrack.
  2360:         assert((backtrack[i] == backtrack.back()) && "Lost back track");
  2361:         backtrack.pop_back();
  2362:       }
  2363:       // Point the main iterator to the next instruction.
  2364:       if (NextMI == SourceLocation)
  2365:         NextMI++;
  2366:     }
  2367:     SourceLocation = MachineBasicBlock::iterator();
  2368:     Head = MachineBasicBlock::instr_iterator();
  2369:     Tail = MachineBasicBlock::instr_iterator();
  2370:     return;
  2371:   }
  2372: 
  2373:   // The old header, soon to be deleted.
  2374:   MachineBasicBlock::instr_iterator Outcast = SourceLocation.getInstrIterator();
  2375:   LLVM_DEBUG(dbgs() << "\t\t\t[Rem] SourceLocation after bundle update: ";
  2376:              DumpPacket(Outcast));
  2377: 
  2378:   // If bundle has been already destroyed. BB->splat seems to do it some times
  2379:   // but not the other.
  2380:   // We already know that SourceLocation is bundle header.
  2381:   if (!SourceLocation->isBundledWithSucc()) {
  2382:     assert(!Head->isBundledWithSucc() && !Head->isBundledWithPred() &&
  2383:            "Bad bundle");
  2384:   } else {
  2385:     Head = SourceLocation.getInstrIterator();
  2386:     Tail = getBundleEnd(Head);
  2387:     unsigned Size = 0;
  2388:     unsigned BBSizeWithDbg = 0;
  2389:     MachineBasicBlock::const_instr_iterator I(Head);
  2390:     MachineBasicBlock::const_instr_iterator E = Head->getParent()->instr_end();
  2391: 
  2392:     for (++I; I != E && I->isBundledWithPred(); ++I) {
  2393:       ++BBSizeWithDbg;
  2394:       if (!I->isDebugInstr())
  2395:         ++Size;
  2396:     }
  2397: 
  2398:     LLVM_DEBUG(dbgs() << "\t\t\t[Rem] Size(" << Size << ") Head orig: ";
  2399:                DumpPacket(Head));
  2400:     // The old header, soon to be deleted.
```
- EN: It declares or implements routines such as LLVM_DEBUG, assert, dump, erase, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, assert, dump, erase, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:     Outcast = Head;
  2402: 
  2403:     // The old Header is still counted here.
  2404:     if (Size > 1) {
  2405:       if (Outcast->isBundle() && Outcast->isBundledWithSucc())
  2406:         Outcast->unbundleFromSucc();
  2407: 
  2408:       bool memShufDisabled = QII->getBundleNoShuf(*Outcast);
  2409:       // The finalizeBundle() assumes that "original" sequence
  2410:       // it is finalizing is sequentially correct. That basically
  2411:       // means that swap case might not be handled properly.
  2412:       // I find insert point for the pull-up instruction myself,
  2413:       // and I should try to catch that swap case there, and refuse
  2414:       // to insert if I cannot guarantee correct serial semantics.
  2415:       // In the future, I need my own incremental "inserToBundle"
  2416:       // function.
  2417:       finalizeBundle(*HomeBB, ++Head, Tail);
  2418:       MachineBasicBlock::instr_iterator BundleMII = std::prev(Head);
  2419:       if (memShufDisabled)
  2420:         QII->setBundleNoShuf(BundleMII);
  2421: 
  2422:       --Head;
  2423:     } else if (Size == 1) {
  2424:       // There is only one non-debug instruction in the bundle.
  2425:       if (BBSizeWithDbg > 1) {
  2426:         // There are some debug instructions that should be unbundled too.
  2427:         MachineBasicBlock::instr_iterator I(Head);
  2428:         MachineBasicBlock::instr_iterator E = Head->getParent()->instr_end();
  2429:         for (++I; I != E && I->isBundledWithPred(); ++I) {
  2430:           I->unbundleFromPred();
  2431:           // Set Head to the non-debug instruction.
  2432:           if (!I->isDebugInstr())
  2433:             Head = I;
  2434:         }
  2435:       } else {
  2436:         // This means that only one original instruction is
  2437:         // left in the bundle. We need to "unbundle" it because the
  2438:         // rest of API will not like it.
  2439:         ++Head;
  2440:         if (Head->isBundledWithPred())
  2441:           Head->unbundleFromPred();
  2442:         if (Head->isBundledWithSucc())
  2443:           Head->unbundleFromSucc();
  2444:       }
  2445:     } else
  2446:       llvm_unreachable("Corrupt bundle");
  2447:   }
  2448: 
  2449:   LLVM_DEBUG(dbgs() << "\t\t\t[Rem] New Head : "; DumpPacket(Head));
  2450:   SourceLocation = Head;
  2451: 
  2452:   // The old header could be listed in the back tracking,
  2453:   // so if it is, we need to update it.
  2454:   for (unsigned i = 0; i < backtrack.size(); ++i)
  2455:     if (backtrack[i] == &*Outcast)
  2456:       backtrack[i] = &*Head;
  2457: 
  2458:   // Same for top MI iterator.
  2459:   if (NextMI == Outcast)
  2460:     NextMI = Head;
  2461: 
  2462:   HomeBB->erase(Outcast);
  2463: }
  2464: 
  2465: #ifndef NDEBUG
  2466: static void debugLivenessForBB(const MachineBasicBlock *MBB,
  2467:                                const TargetRegisterInfo *TRI) {
  2468:   LLVM_DEBUG(dbgs() << "\tLiveness for BB:\n"; MBB->dump());
  2469:   for (MachineBasicBlock::const_succ_iterator SI = MBB->succ_begin(),
  2470:                                               SE = MBB->succ_end();
  2471:        SI != SE; ++SI) {
  2472:     LLVM_DEBUG(dbgs() << "\tSuccessor BB (" << (*SI)->getNumber() << "):");
  2473:     for (MachineBasicBlock::livein_iterator I = (*SI)->livein_begin(),
  2474:                                             E = (*SI)->livein_end();
  2475:          I != E; ++I)
  2476:       LLVM_DEBUG(dbgs() << "\t" << printReg((*I).PhysReg, TRI));
  2477:     LLVM_DEBUG(dbgs() << "\n");
  2478:   }
  2479: }
  2480: #endif
  2481: 
  2482: // Blocks should be considered empty if they contain only debug info;
  2483: // else the debug info would affect codegen.
  2484: static bool IsEmptyBlock(MachineBasicBlock *MBB) {
  2485:   if (MBB->empty())
  2486:     return true;
  2487:   for (MachineBasicBlock::iterator MBBI = MBB->begin(), MBBE = MBB->end();
  2488:        MBBI != MBBE; ++MBBI) {
  2489:     if (!MBBI->isDebugInstr())
  2490:       return false;
  2491:   }
  2492:   return true;
  2493: }
  2494: 
  2495: /// Treat given instruction as a branch, go through its operands
  2496: /// and see if any of them is a BB address. If so, return it.
  2497: /// Return NULL otherwise.
  2498: static inline MachineBasicBlock *getBranchDestination(MachineInstr *MI) {
  2499:   if (!MI || !MI->isBranch() || MI->isBundle())
  2500:     return NULL;
  2501: 
  2502:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
  2503:     const MachineOperand &MO = MI->getOperand(i);
  2504:     if (MO.isMBB())
  2505:       return MO.getMBB();
  2506:   }
  2507:   return NULL;
  2508: }
  2509: 
  2510: /// Similar to HexagonInstrInfo::analyzeBranch but handles
  2511: /// serveral more general cases including parsing empty BBs when possible.
  2512: bool HexagonGlobalSchedulerImpl::AnalyzeBBBranches(MachineBasicBlock *MBB,
  2513:                                                    MachineBasicBlock *&TBB,
  2514:                                                    MachineInstr *&FirstTerm,
  2515:                                                    MachineBasicBlock *&FBB,
  2516:                                                    MachineInstr *&SecondTerm) {
  2517:   // Hexagon allowes up to two jumps in MBB.
  2518:   FirstTerm = NULL;
  2519:   SecondTerm = NULL;
  2520: 
  2521:   LLVM_DEBUG(dbgs() << "\n\t\tAnalyze Branches in BB(" << MBB->getNumber()
  2522:                     << ")\n");
  2523:   if (MBB->succ_size() == 0) {
  2524:     LLVM_DEBUG(dbgs() << "\n\t\tBlock has no successors.\n");
  2525:     return true;
  2526:   }
  2527:   // Find both jumps.
  2528:   // We largely rely on implied assumption that BB branching always
  2529:   // looks like this:
  2530:   //  J2_jumpf %P0, <BB#60>, %PC<imp-def>;
  2531:   //  J2_jump <BB#49>
  2532:   // Branches also could be in different packets.
  2533:   MachineBasicBlock::instr_iterator MIB = MBB->instr_begin();
  2534:   MachineBasicBlock::instr_iterator MIE = MBB->instr_end();
  2535:   MachineBasicBlock::instr_iterator MII = MIB;
  2536: 
  2537:   if (QII->nonDbgBBSize(MBB) == 1) {
  2538:     MII = MBB->getFirstNonDebugInstr().getInstrIterator();
  2539:     if (MII->isBranch())
  2540:       FirstTerm = &*MII;
  2541:   } else {
  2542:     // We have already eliminated the case when MIB == MIE.
  2543:     while (MII != MIE) {
  2544:       if (!MII->isBundle() && MII->isBranch()) {
  2545:         if (!FirstTerm)
  2546:           FirstTerm = &*MII;
  2547:         else
  2548:           SecondTerm = &*MII;
  2549:       }
  2550:       ++MII;
  2551:     }
  2552:   }
  2553:   if ((FirstTerm && FirstTerm->isIndirectBranch()) ||
  2554:       (SecondTerm && SecondTerm->isIndirectBranch())) {
  2555:     LLVM_DEBUG(dbgs() << "\n\t\tCannot analyze BB with indirect branch.");
  2556:     return true;
  2557:   }
  2558:   if ((FirstTerm && FirstTerm->getOpcode() == Hexagon::J2_jump &&
  2559:        !FirstTerm->getOperand(0).isMBB()) ||
  2560:       (SecondTerm && SecondTerm->getOpcode() == Hexagon::J2_jump &&
  2561:        !SecondTerm->getOperand(0).isMBB())) {
  2562:     LLVM_DEBUG(
  2563:         dbgs() << "\n\t\tCannot analyze BB with a branch out of function.");
  2564:     return true;
  2565:   }
  2566: 
  2567:   // Now try to analyze this branch.
  2568:   SmallVector<MachineOperand, 4> Cond;
  2569:   if (QII->analyzeBranch(*MBB, TBB, FBB, Cond, false)) {
  2570:     LLVM_DEBUG(dbgs() << "\t\tFail to analyze with analyzeBranch.\n");
  2571:     LLVM_DEBUG(dbgs() << "\t\tFirst term: "; if (FirstTerm) FirstTerm->dump();
  2572:                else dbgs() << "None\n";);
  2573:     // Could not analyze it. See if this is something we can recognize.
  2574:     TBB = getBranchDestination(FirstTerm);
  2575:   }
  2576:   // There are several cases not handled by HexagonInstrInfo::analyzeBranch.
  2577:   if (!TBB) {
  2578:     LLVM_DEBUG(dbgs() << "\t\tMissing TBB.\n");
  2579:     // There is a branch, but TBB is not found.
  2580:     // The BB could also be empty at this point. See if it is a trivial
  2581:     // layout case.
  2582:     if (MBB->succ_size() == 1) {
  2583:       TBB = *MBB->succ_begin();
  2584:       LLVM_DEBUG(dbgs() << "\t\tFall through TBB(" << TBB->getNumber()
  2585:                         << ").\n");
  2586:       return false;
  2587:     } else if (MBB->succ_size() == 2) {
  2588:       // This should cover majority of remaining cases.
  2589:       if (FirstTerm && SecondTerm &&
  2590:           (QII->isPredicated(*FirstTerm) || QII->isNewValueJump(*FirstTerm)) &&
  2591:           !QII->isPredicated(*SecondTerm)) {
  2592:         TBB = getBranchDestination(FirstTerm);
  2593:         FBB = getBranchDestination(SecondTerm);
  2594:         LLVM_DEBUG(dbgs() << "\t\tCanonical dual jump layout: TBB("
  2595:                           << TBB->getNumber() << ") FBB(" << FBB->getNumber()
  2596:                           << ").\n");
  2597:         return false;
  2598:       } else if (SecondTerm && SecondTerm->getOpcode() == Hexagon::J2_jump &&
  2599:                  SecondTerm->getOperand(0).isMBB()) {
  2600:         // Look at the second term if I know it, to find out what is the fall
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getBundleNoShuf, finalizeBundle, std::prev, I, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getBundleNoShuf, finalizeBundle, std::prev, I, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601:         // through for this BB.
  2602:         FBB = SecondTerm->getOperand(0).getMBB();
  2603:         assert(MBB->succ_size() == 2 && "Expected exactly 2 successors");
  2604:         MachineBasicBlock *Succ0 = *MBB->succ_begin();
  2605:         MachineBasicBlock *Succ1 = *std::next(MBB->succ_begin());
  2606:         if (FBB == Succ0)
  2607:           TBB = Succ1;
  2608:         else
  2609:           TBB = Succ0;
  2610:         LLVM_DEBUG(dbgs() << "\t\tSecond br is J2_jump TBB(" << TBB->getNumber()
  2611:                           << ") FBB(" << FBB->getNumber() << ").\n");
  2612:         return false;
  2613:       } else {
  2614:         // This might be an empty BB but still with two
  2615:         // successors set. Try to use CFG layout to sort it out.
  2616:         // This could happen when last jump was pulled up from a BB, and
  2617:         // CFG is being updated. At that point this method is called and
  2618:         // returns best guess possible for TBB/FBB. Fortunately order of those
  2619:         // is irrelevant, and rather used a worklist for CFG update.
  2620:         MachineFunction::iterator MBBIter = MBB->getIterator();
  2621:         MachineFunction &MF = *MBB->getParent();
  2622:         (void)MF; // supress compiler warning
  2623:         // If there are no other clues, assume next sequential BB
  2624:         // in CFG as FBB.
  2625:         ++MBBIter;
  2626:         assert(MBBIter != MF.end() && "I give up.");
  2627:         FBB = &(*MBBIter);
  2628:         assert(MBB->succ_size() == 2 && "Expected exactly 2 successors");
  2629:         MachineBasicBlock *S0 = *MBB->succ_begin();
  2630:         MachineBasicBlock *S1 = *std::next(MBB->succ_begin());
  2631:         if (FBB == S0)
  2632:           TBB = S1;
  2633:         else if (FBB == S1) {
  2634:           TBB = S0;
  2635:         } else {
  2636:           // This case can arise when the layout successor basic block (++IMBB)
  2637:           // got empty during pull-up.
  2638:           // As a result, ++IMBB is not one of MBB's successors.
  2639:           MBBIter = MF.begin();
  2640:           while (!MBB->isSuccessor(&*MBBIter) && (MBBIter != MF.end()))
  2641:             ++MBBIter;
  2642:           assert(MBBIter != MF.end() && "Malformed BB with invalid successors");
  2643:           FBB = &*MBBIter;
  2644:           if (FBB == S0)
  2645:             TBB = S1;
  2646:           else
  2647:             TBB = S0;
  2648:         }
  2649:         LLVM_DEBUG(dbgs() << "\t\tUse layout TBB(" << TBB->getNumber()
  2650:                           << ") FBB(" << FBB->getNumber() << ").\n");
  2651:         return false;
  2652:       }
  2653:     }
  2654:     assert(!FirstTerm && "Bad BB");
  2655:     return true;
  2656:   }
  2657:   // Ok, we have TBB, but maybe missing FBB.
  2658:   if (!FBB && SecondTerm) {
  2659:     LLVM_DEBUG(dbgs() << "\t\tMissing FBB.\n");
  2660:     // analyzeBranch could lie to us, ignore it in this case.
  2661:     // For the canonical case simply take known branch targets.
  2662:     if ((QII->isPredicated(*FirstTerm) || QII->isNewValueJump(*FirstTerm)) &&
  2663:         !QII->isPredicated(*SecondTerm)) {
  2664:       FBB = getBranchDestination(SecondTerm);
  2665:     } else {
  2666:       // Second term is also predicated.
  2667:       // Use CFG layout. Assign layout successor as FBB.
  2668:       for (MachineBasicBlock *Succ : MBB->successors()) {
  2669:         if (MBB->isLayoutSuccessor(Succ))
  2670:           FBB = Succ;
  2671:       }
  2672:       if (FBB == NULL) {
  2673:         LLVM_DEBUG(dbgs() << "\nNo layout successor found.");
  2674:         LLVM_DEBUG(dbgs() << "Possibly the layout successor is an empty BB");
  2675:         return true;
  2676:       }
  2677:       if (TBB == FBB)
  2678:         LLVM_DEBUG(dbgs() << "Malformed branch with useless branch condition";);
  2679:     }
  2680:     LLVM_DEBUG(dbgs() << "\t\tSecond term: "; SecondTerm->dump());
  2681:   } else if (TBB && !FBB) {
  2682:     // If BB ends in endloop, and it is a single BB hw loop,
  2683:     // we will have a single terminator, but we can figure FBB
  2684:     // easily from CFG.
  2685:     if (MBB->succ_size() == 2) {
  2686:       MachineBasicBlock *S0 = *MBB->succ_begin();
  2687:       MachineBasicBlock *S1 = *std::next(MBB->succ_begin());
  2688:       if (TBB == S0)
  2689:         FBB = S1;
  2690:       else
  2691:         FBB = S0;
  2692:     }
  2693:   }
  2694: 
  2695:   LLVM_DEBUG(dbgs() << "\t\tFinal TBB(" << TBB->getNumber() << ").\n";
  2696:              if (FBB) dbgs() << "\t\tFinal FBB(" << FBB->getNumber() << ").\n";
  2697:              else dbgs() << "\t\tFinal FBB(None)\n";);
  2698:   return false;
  2699: }
  2700: 
  2701: /// updateBranches - Updates all branches to \p From in the basic block \p
  2702: /// InBlock to branches to \p To.
  2703: static void updateBranches(MachineBasicBlock &InBlock, MachineBasicBlock *From,
  2704:                            MachineBasicBlock *To) {
  2705:   for (MachineBasicBlock::instr_iterator BI = InBlock.instr_begin(),
  2706:                                          E = InBlock.instr_end();
  2707:        BI != E; ++BI) {
  2708:     MachineInstr *Inst = &*BI;
  2709:     // Ignore anything that is not a branch.
  2710:     if (!Inst->isBranch())
  2711:       continue;
  2712:     for (MachineInstr::mop_iterator OI = Inst->operands_begin(),
  2713:                                     OE = Inst->operands_end();
  2714:          OI != OE; ++OI) {
  2715:       MachineOperand &Opd = *OI;
  2716:       // Look for basic block "From".
  2717:       if (!Opd.isMBB() || Opd.getMBB() != From)
  2718:         continue;
  2719:       // Update it.
  2720:       Opd.setMBB(To);
  2721:     }
  2722:   }
  2723: }
  2724: 
  2725: /// Rewrite all predecessors of the old block to go to the fallthrough
  2726: /// instead.
  2727: /// NB: Collect predecessors into a snapshot vector before iterating to
  2728: /// avoid iterator invalidation on MBB's predecessor list. Each call to
  2729: /// ReplaceUsesOfBlockWith modifies both the successor list of Pred and
  2730: /// the predecessor list of MBB, which invalidates debug-mode iterators
  2731: /// (detected by _GLIBCXX_DEBUG).
  2732: static void updatePredecessors(MachineBasicBlock &MBB,
  2733:                                MachineBasicBlock *MFBB) {
  2734:   MachineFunction &MF = *MBB.getParent();
  2735: 
  2736:   if (MFBB->getIterator() == MF.end())
  2737:     return;
  2738: 
  2739:   // Snapshot the predecessor list to avoid iterator invalidation.
  2740:   SmallVector<MachineBasicBlock *, 4> Preds(MBB.pred_begin(), MBB.pred_end());
  2741:   for (MachineBasicBlock *Pred : Preds) {
  2742:     if (!Pred->isSuccessor(&MBB))
  2743:       continue;
  2744:     Pred->ReplaceUsesOfBlockWith(&MBB, MFBB);
  2745:     updateBranches(*Pred, &MBB, MFBB);
  2746:   }
  2747: }
  2748: 
  2749: static void UpdateCFG(MachineBasicBlock *HomeBB, MachineBasicBlock *OriginBB,
  2750:                       MachineInstr *MII, MachineBasicBlock *HomeTBB,
  2751:                       MachineBasicBlock *HomeFBB, MachineInstr *FTA,
  2752:                       MachineInstr *STA,
  2753:                       const MachineBranchProbabilityInfo *MBPI) {
  2754:   MachineBasicBlock *S2Add = NULL, *S2Remove = NULL;
  2755:   bool RemoveLSIfPresent = false;
  2756:   if ((&*MII == FTA) && MII->isConditionalBranch()) {
  2757:     LLVM_DEBUG(dbgs() << "\nNew firstterm conditional jump added to HomeBB";);
  2758:     S2Add = HomeTBB;
  2759:     S2Remove = HomeTBB;
  2760:   } else if ((&*MII == STA) && MII->isConditionalBranch()) {
  2761:     LLVM_DEBUG(dbgs() << "\nNew secondterm conditional jump added to HomeBB";);
  2762:     // AnalyzeBBBranches might not give correct information in this case.
  2763:     // The branch destination may be a symbol, not necessarily a block.
  2764:     if (MachineBasicBlock *Dest = getBranchDestination(MII)) {
  2765:       LLVM_DEBUG(dbgs() << "\nBranch destination for pulled instruction is BB#"
  2766:                         << Dest->getNumber(););
  2767:       S2Add = Dest;
  2768:       S2Remove = Dest;
  2769:     }
  2770:   } else if ((&*MII == FTA) && MII->isUnconditionalBranch()) {
  2771:     LLVM_DEBUG(dbgs() << "\nNew firstterm unconditional jump added to HomeBB";);
  2772:     S2Add = HomeTBB;
  2773:     S2Remove = HomeTBB;
  2774:     RemoveLSIfPresent = true;
  2775:   } else if ((&*MII == STA) && MII->isUnconditionalBranch()) {
  2776:     LLVM_DEBUG(
  2777:         dbgs() << "\nNew secondterm unconditional jump added to HomeBB";);
  2778:     S2Add = HomeFBB;
  2779:     S2Remove = HomeFBB;
  2780:     RemoveLSIfPresent = true;
  2781:   }
  2782:   if (S2Add && !HomeBB->isSuccessor(S2Add)) {
  2783:     HomeBB->addSuccessor(S2Add, MBPI->getEdgeProbability(OriginBB, S2Add));
  2784:   }
  2785:   if (S2Remove)
  2786:     OriginBB->removeSuccessor(S2Remove);
  2787:   if (RemoveLSIfPresent) {
  2788:     MachineFunction::iterator HomeBBLS = HomeBB->getIterator();
  2789:     ++HomeBBLS;
  2790:     if (HomeBBLS != HomeBB->getParent()->end() &&
  2791:         HomeBB->isLayoutSuccessor(&*HomeBBLS)) {
  2792:       LLVM_DEBUG(dbgs() << "\nRemoving LayoutSucc BB#" << HomeBBLS->getNumber()
  2793:                         << "from list of successors";);
  2794:       HomeBB->removeSuccessor(&*HomeBBLS);
  2795:     }
  2796:   }
  2797: }
  2798: 
  2799: /// Move instruction from/to BB, Update liveness info,
  2800: /// return pointer to the newly inserted and modified
```
- EN: It declares or implements routines such as getOperand, assert, succ_begin, std::next, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getOperand, assert, succ_begin, std::next, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801: /// instruction.
  2802: MachineInstr *HexagonGlobalSchedulerImpl::MoveAndUpdateLiveness(
  2803:     BasicBlockRegion *CurrentRegion, MachineBasicBlock *HomeBB,
  2804:     MachineInstr *InstrToMove, bool NeedToNewify, unsigned DepReg,
  2805:     bool MovingDependentOp, MachineBasicBlock *OriginBB,
  2806:     MachineInstr *OriginalInstruction, SmallVector<MachineOperand, 4> &Cond,
  2807:     MachineBasicBlock::iterator &SourceLocation,
  2808:     MachineBasicBlock::iterator &TargetPacket,
  2809:     MachineBasicBlock::iterator &NextMI,
  2810:     std::vector<MachineInstr *> &backtrack) {
  2811:   LLVM_DEBUG(
  2812:       dbgs() << "\n...............[MoveAndUpdateLiveness]..............\n");
  2813:   LLVM_DEBUG(dbgs() << "\t\tInstrToMove        :\t"; InstrToMove->dump());
  2814:   LLVM_DEBUG(dbgs() << "\t\tOriginalInstruction:\t";
  2815:              OriginalInstruction->dump());
  2816:   LLVM_DEBUG(dbgs() << "\t\tSourceLocation     :\t";
  2817:              DumpPacket(SourceLocation.getInstrIterator()));
  2818:   LLVM_DEBUG(dbgs() << "\t\tTargetPacket       :\t";
  2819:              DumpPacket(TargetPacket.getInstrIterator()));
  2820: 
  2821:   MachineBasicBlock::instr_iterator OriginalHead =
  2822:       SourceLocation.getInstrIterator();
  2823:   MachineBasicBlock::instr_iterator OriginalTail = getBundleEnd(OriginalHead);
  2824:   MachineBasicBlock::instr_iterator OutcastFrom =
  2825:       OriginalInstruction->getIterator();
  2826: 
  2827:   // Remove our temporary instruction.
  2828:   MachineBasicBlock::instr_iterator kill_it(InstrToMove);
  2829:   HomeBB->erase(kill_it);
  2830: 
  2831:   MachineBasicBlock::instr_iterator TargetHead(TargetPacket.getInstrIterator());
  2832:   MachineBasicBlock::instr_iterator TargetTail = getBundleEnd(TargetHead);
  2833: 
  2834:   LLVM_DEBUG(dbgs() << "\n\tTo BB before:\n"; debugLivenessForBB(HomeBB, QRI));
  2835:   LLVM_DEBUG(dbgs() << "\n\tFrom BB before:\n";
  2836:              debugLivenessForBB(OriginBB, QRI));
  2837: 
  2838:   // Before we perform the move, we need to collect the worklist
  2839:   // of BBs for liveness updated.
  2840:   std::list<MachineBasicBlock *> WorkList;
  2841: 
  2842:   // Insert into the work list all BBs along the backtrace.
  2843:   for (std::vector<MachineInstr *>::iterator RI = backtrack.begin(),
  2844:                                              RIE = backtrack.end();
  2845:        RI != RIE; RI++)
  2846:     WorkList.push_back((*RI)->getParent());
  2847: 
  2848:   // Only keep unique entries.
  2849:   // TODO: Use a different container here.
  2850:   WorkList.unique();
  2851: 
  2852:   // Move the original instruction.
  2853:   // If this instruction is inside a bundle, update the bundle.
  2854:   MachineBasicBlock::instr_iterator BBEnd =
  2855:       TargetHead->getParent()->instr_end();
  2856:   bool LastInstructionInBundle = false;
  2857:   MachineBasicBlock::instr_iterator MII = findInsertPositionInBundle(
  2858:       TargetPacket, &*OutcastFrom, LastInstructionInBundle);
  2859: 
  2860:   (void)BBEnd;
  2861:   LLVM_DEBUG(dbgs() << "\n\t\t\tHead target      : "; DumpPacket(TargetHead));
  2862:   LLVM_DEBUG(dbgs() << "\t\t\tTail target        : ";
  2863:              DumpPacket(TargetTail, BBEnd));
  2864:   LLVM_DEBUG(dbgs() << "\t\t\tInsert right before: "; DumpPacket(MII, BBEnd));
  2865: 
  2866:   MIBundleBuilder Bundle(&*TargetHead);
  2867: 
  2868:   // Actual move. One day liveness might be updated here.
  2869:   if (OriginalInstruction->isBundled()) {
  2870:     Bundle.insert(MII, OriginalInstruction->removeFromBundle());
  2871:     --MII;
  2872:   } else {
  2873:     // This is one case currently unhandled by Bundle.insert
  2874:     // and needs to be fixed upstream. Meanwhile use old way to handle
  2875:     // this odd case.
  2876:     if (OriginalInstruction->getIterator() == TargetTail) {
  2877:       LLVM_DEBUG(dbgs() << "\t\t\tSpecial case move.\n");
  2878:       MachineBasicBlock::instr_iterator MIIToPred = MII;
  2879:       --MIIToPred;
  2880:       LLVM_DEBUG(dbgs() << "\t\t\tInser after        : ";
  2881:                  DumpPacket(MIIToPred, BBEnd));
  2882:       // Unbundle it in its current location.
  2883:       if (OutcastFrom->isBundledWithSucc()) {
  2884:         OutcastFrom->clearFlag(MachineInstr::BundledSucc);
  2885:         OutcastFrom->clearFlag(MachineInstr::BundledPred);
  2886:       } else if (OutcastFrom->isBundledWithPred()) {
  2887:         OutcastFrom->unbundleFromPred();
  2888:       }
  2889:       HomeBB->splice(MII, OriginBB, OutcastFrom);
  2890:       if (!MII->isBundledWithPred())
  2891:         MII->bundleWithPred();
  2892:       if (!LastInstructionInBundle && !MII->isBundledWithSucc())
  2893:         MII->bundleWithSucc();
  2894:       // This is the instruction after which we have inserted.
  2895:       if (!MIIToPred->isBundledWithSucc())
  2896:         MIIToPred->bundleWithSucc();
  2897:     } else {
  2898:       Bundle.insert(MII, OriginalInstruction->removeFromParent());
  2899:       --MII;
  2900:     }
  2901:   }
  2902:   // Source location bundle is updated later in the
  2903:   // removeInstructionFromExistingBundle().
  2904: 
  2905:   LLVM_DEBUG(dbgs() << "\t\t\tNew packet head: "; DumpPacket(TargetHead));
  2906:   LLVM_DEBUG(dbgs() << "\t\t\tInserted op    : "; MII->dump());
  2907:   LLVM_DEBUG(dbgs() << "\n\tTo BB after move:\n";
  2908:              debugLivenessForBB(HomeBB, QRI));
  2909:   LLVM_DEBUG(dbgs() << "\n\tFrom BB after:\n";
  2910:              debugLivenessForBB(OriginBB, QRI));
  2911: 
  2912:   // Update kill patterns. Do it before we have predicated the moved
  2913:   // instruction.
  2914:   updateKillAlongThePath(HomeBB, OriginBB, MII, TargetTail, SourceLocation,
  2915:                          TargetPacket, backtrack);
  2916:   // I need to know:
  2917:   // - true/false predication
  2918:   // - do I need to .new it?
  2919:   // - do I need to .old it?
  2920:   // If the original instruction used new value operands,
  2921:   // it might need to be changed to the generic form
  2922:   // before further processing.
  2923:   if (QII->isDotNewInst(*MII)) {
  2924:     DemoteToDotOld(&*MII);
  2925:     LLVM_DEBUG(dbgs() << "\t\t\tDemoted to .old\t:"; MII->dump());
  2926:   }
  2927: 
  2928:   // We have previously checked whether this instruction could
  2929:   // be placed in this packet, including all possible transformations
  2930:   // it might need, so if any request will fail now, something is wrong.
  2931:   //
  2932:   // Need for predication and the exact condition is determined by
  2933:   // the path between original and current instruction location.
  2934:   if (!Cond.empty()) { // To be predicated
  2935:     LLVM_DEBUG(dbgs() << "\t\t\tPredicating:"; MII->dump());
  2936:     assert(TII->isPredicable(*MII) && "MII is not predicable");
  2937:     TII->PredicateInstruction(*MII, Cond);
  2938:     if (NeedToNewify) {
  2939:       assert((DepReg < std::numeric_limits<unsigned>::max()) &&
  2940:              "Invalid pred reg value");
  2941:       LLVM_DEBUG(dbgs() << "\t\t\tNeeds to NEWify on Reg("
  2942:                         << printReg(DepReg, QRI) << ").\n");
  2943:       int NewOpcode = QII->getDotNewPredOp(*MII, MBPI);
  2944:       MII->setDesc(QII->get(NewOpcode));
  2945: 
  2946:       // Now we need to mark newly created predicate operand as
  2947:       // internal read.
  2948:       // TODO: Better look for predicate operand.
  2949:       for (unsigned i = 0, e = MII->getNumOperands(); i != e; ++i) {
  2950:         MachineOperand &MO = MII->getOperand(i);
  2951:         if (!MO.isReg())
  2952:           continue;
  2953:         if (MO.isDef())
  2954:           continue;
  2955:         if (DepReg == MO.getReg())
  2956:           MO.setIsInternalRead();
  2957:       }
  2958:     }
  2959:     LLVM_DEBUG(dbgs() << "\t\t\tNew predicated form:\t"; MII->dump());
  2960:     // If the predicate has changed kill pattern, now we need to propagate
  2961:     // that again. This is important for liveness computation.
  2962:     updateKillAlongThePath(HomeBB, OriginBB, MII, TargetTail, SourceLocation,
  2963:                            TargetPacket, backtrack);
  2964:   }
  2965: 
  2966:   // Create new bundle header, remove the old one.
  2967:   addInstructionToExistingBundle(HomeBB, TargetHead, TargetTail, MII,
  2968:                                  TargetPacket, NextMI, backtrack);
  2969: 
  2970:   // If moved instruction was inside a bundle, update that bundle.
  2971:   removeInstructionFromExistingBundle(OriginBB, ++OriginalHead, OriginalTail,
  2972:                                       SourceLocation, NextMI, MovingDependentOp,
  2973:                                       backtrack);
  2974: 
  2975:   // If removed instruction could have been dependent on any
  2976:   // of the remaining ops, we need to oldify possible affected ones.
  2977:   LLVM_DEBUG(dbgs() << "\t\tTargetHead:\t"; DumpPacket(TargetHead, BBEnd));
  2978:   LLVM_DEBUG(dbgs() << "\t\tOriginalHead:\t"; DumpPacket(OriginalHead, BBEnd));
  2979:   LLVM_DEBUG(dbgs() << "\t\tOriginalInstruction:\t";
  2980:              DumpPacket(OriginalInstruction->getIterator(), BBEnd));
  2981:   LLVM_DEBUG(dbgs() << "\t\tOutcastFrom:\t"; DumpPacket(OutcastFrom, BBEnd));
  2982: 
  2983:   // Clean up the original source bundle on a global scope.
  2984:   if (OriginalHead != MachineBasicBlock::instr_iterator() &&
  2985:       QII->isEndLoopN(OriginalHead->getOpcode())) {
  2986:     // Single endloop left. Since it is not a real instruction,
  2987:     // we can simply add it to a non empty previous bundle, if one exist,
  2988:     // or let assembler to produce a fake bundle for it.
  2989:     LLVM_DEBUG(dbgs() << "\t\tOnly endloop in packet.\n");
  2990:     MachineBasicBlock::instr_iterator I(OriginalHead);
  2991:     if (OriginBB->begin() != I) {
  2992:       --I;
  2993:       if (I->isBundled()) {
  2994:         if (!I->isBundledWithSucc())
  2995:           I->bundleWithSucc();
  2996:         if (!OriginalHead->isBundledWithPred())
  2997:           OriginalHead->bundleWithPred();
  2998:       }
  2999:       // else we probably need to create a new bundle here.
  3000:       // SourceLocation = NULL;
```
- EN: It declares or implements routines such as HexagonGlobalSchedulerImpl::MoveAndUpdateLiveness, LLVM_DEBUG, dump, DumpPacket, ... (30 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGlobalSchedulerImpl::MoveAndUpdateLiveness, LLVM_DEBUG, dump, DumpPacket, ... (30 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 3001-3200 / 第 3001-3200 行

```cpp
  3001:     }
  3002:   } else if (MovingDependentOp &&
  3003:              OriginalHead != MachineBasicBlock::instr_iterator()) {
  3004:     if (OriginalHead->isBundled()) {
  3005:       for (MachineBasicBlock::instr_iterator J = ++OriginalHead;
  3006:            J != OriginalTail && J->isInsideBundle() && !J->isBundle(); ++J) {
  3007:         // Need to oldify it.
  3008:         if (MIsHaveTrueDependency(OriginalInstruction, &*J) &&
  3009:             QII->isDotNewInst(*J)) {
  3010:           LLVM_DEBUG(dbgs() << "\t\tDemoting to .old:\t"; J->dump());
  3011:           DemoteToDotOld(&*J);
  3012:         }
  3013:       }
  3014:     } else {
  3015:       // Single instruction left.
  3016:       if (MIsHaveTrueDependency(OriginalInstruction, &*OriginalHead) &&
  3017:           QII->isDotNewInst(*OriginalHead)) {
  3018:         LLVM_DEBUG(dbgs() << "\t\tDemoting to .old op:\t";
  3019:                    OriginalHead->dump());
  3020:         DemoteToDotOld(&*OriginalHead);
  3021:       }
  3022:     }
  3023:   }
  3024: 
  3025:   // Now we need to update liveness to all BBs involved
  3026:   // including those we might have "passed" through on the way here.
  3027:   LLVM_DEBUG(dbgs() << "\n\tTo BB after bundle update:\n"; HomeBB->dump());
  3028:   LLVM_DEBUG(dbgs() << "\n\n\tFrom BB after bundle update:\n";
  3029:              OriginBB->dump());
  3030: 
  3031:   // Update global liveness.
  3032:   LLVM_DEBUG(dbgs() << "\n\tWorkList:\t");
  3033:   for (std::list<MachineBasicBlock *>::iterator BBI = WorkList.begin(),
  3034:                                                 BBIE = WorkList.end();
  3035:        BBI != BBIE; BBI++) {
  3036:     LLVM_DEBUG(dbgs() << "BB#" << (*BBI)->getNumber() << " ");
  3037:   }
  3038:   LLVM_DEBUG(dbgs() << "\n");
  3039: 
  3040:   do {
  3041:     MachineBasicBlock *BB = WorkList.back();
  3042:     WorkList.pop_back();
  3043:     CurrentRegion->getLivenessInfoForBB(BB)->UpdateLiveness(BB);
  3044:   } while (!WorkList.empty());
  3045: 
  3046:   // No need to analyze for empty BB or update CFG for same BB pullup.
  3047:   if (OriginBB == HomeBB)
  3048:     return &*TargetHead;
  3049:   // If the instruction moved was a branch we need to update the
  3050:   // successor/predecessor of OriginBB and HomeBB accordingly.
  3051:   MachineBasicBlock *HomeTBB, *HomeFBB;
  3052:   MachineInstr *FTA = NULL, *STA = NULL;
  3053:   bool HomeBBAnalyzed = !AnalyzeBBBranches(HomeBB, HomeTBB, FTA, HomeFBB, STA);
  3054:   if (MII->isBranch()) {
  3055:     if (HomeBBAnalyzed) {
  3056:       UpdateCFG(HomeBB, OriginBB, &*MII, HomeTBB, HomeFBB, FTA, STA, MBPI);
  3057:     } else {
  3058:       llvm_unreachable("Underimplememted AnalyzeBBBranches");
  3059:     }
  3060:   }
  3061:   // If we have exhausted the OriginBB clean it up.
  3062:   // Beware that we could have created dual conditional jumps, which
  3063:   // ultimately means we can have three way jumps.
  3064:   if (IsEmptyBlock(OriginBB) && !OriginBB->isEHPad() &&
  3065:       !OriginBB->hasAddressTaken() && !OriginBB->succ_empty()) {
  3066:     // Dead block? Unlikely, but check.
  3067:     LLVM_DEBUG(dbgs() << "Empty BB(" << OriginBB->getNumber() << ").\n");
  3068:     // Update region map.
  3069:     CurrentRegion->RemoveBBFromRegion(OriginBB);
  3070:     // Keep the list of empty basic blocks to be freed later.
  3071:     EmptyBBs.push_back(OriginBB);
  3072:     if (OriginBB->pred_empty() || OriginBB->succ_empty())
  3073:       return &*TargetHead;
  3074: 
  3075:     if (OriginBB->succ_size() == 1) {
  3076:       // Find empty block's successor.
  3077:       MachineBasicBlock *CommonFBB = *OriginBB->succ_begin();
  3078:       updatePredecessors(*OriginBB, CommonFBB);
  3079:       // Remove the only successor entry for empty BB.
  3080:       OriginBB->removeSuccessor(CommonFBB);
  3081:     } else {
  3082:       // Three way branching is not yet fully supported.
  3083:       assert((OriginBB->succ_size() == 2) && "Underimplemented 3way branch.");
  3084:       MachineBasicBlock *OriginTBB, *OriginFBB;
  3085:       MachineInstr *FTB = NULL, *STB = NULL;
  3086: 
  3087:       LLVM_DEBUG(dbgs() << "\tComplex case.\n");
  3088:       if (HomeBBAnalyzed &&
  3089:           !AnalyzeBBBranches(OriginBB, OriginTBB, FTB, OriginFBB, STB)) {
  3090:         assert(OriginFBB && "Missing Origin FBB");
  3091:         if (HomeFBB == OriginBB) {
  3092:           // OriginBB is FBB for HomeBB.
  3093:           if (HomeTBB == OriginTBB) {
  3094:             // Shared TBB target, common FBB.
  3095:             updatePredecessors(*OriginBB, OriginFBB);
  3096:           } else if (HomeTBB == OriginFBB) {
  3097:             // Shared TBB target, common FBB.
  3098:             updatePredecessors(*OriginBB, OriginTBB);
  3099:           } else {
  3100:             // Three way branch. Add new successor to HomeBB.
  3101:             updatePredecessors(*OriginBB, OriginFBB);
  3102:             // TODO: Update the weight as well.
  3103:             // Adding the successor to make updatePredecessor happy.
  3104:             HomeBB->addSuccessor(OriginBB);
  3105:             updatePredecessors(*OriginBB, OriginTBB);
  3106:           }
  3107:         } else if (HomeTBB == OriginBB) {
  3108:           // OriginBB is TBB for HomeBB.
  3109:           if (HomeFBB == OriginTBB) {
  3110:             // Shared TBB target, common FBB.
  3111:             updatePredecessors(*OriginBB, OriginFBB);
  3112:           } else if (HomeFBB == OriginFBB) {
  3113:             // Shared TBB target, common FBB.
  3114:             updatePredecessors(*OriginBB, OriginTBB);
  3115:           } else {
  3116:             // Three way branch. Add new successor to HomeBB.
  3117:             updatePredecessors(*OriginBB, OriginFBB);
  3118:             // TODO: Update the weight as well.
  3119:             // Adding the successor to make updatePredecessor happy.
  3120:             HomeBB->addSuccessor(OriginBB);
  3121:             updatePredecessors(*OriginBB, OriginTBB);
  3122:           }
  3123:         } else
  3124:           llvm_unreachable("CFG update failed");
  3125:         // The empty BB can now be relieved of its successors.
  3126:         OriginBB->removeSuccessor(OriginFBB);
  3127:         OriginBB->removeSuccessor(OriginTBB);
  3128:       } else
  3129:         llvm_unreachable("Underimplemented analyzeBranch");
  3130:     }
  3131:     LLVM_DEBUG(dbgs() << "Updated BB(" << HomeBB->getNumber() << ").\n";
  3132:                HomeBB->dump());
  3133:   }
  3134:   return &*TargetHead;
  3135: }
  3136: 
  3137: // Find where inside a given bundle current instruction should be inserted.
  3138: // Instruction will be inserted _before_ this position.
  3139: MachineBasicBlock::instr_iterator
  3140: HexagonGlobalSchedulerImpl::findInsertPositionInBundle(
  3141:     MachineBasicBlock::iterator &Bundle, MachineInstr *MI, bool &LastInBundle) {
  3142:   MachineBasicBlock::instr_iterator MII = Bundle.getInstrIterator();
  3143:   MachineBasicBlock *MBB = MII->getParent();
  3144:   MachineBasicBlock::instr_iterator BBEnd = MBB->instr_end();
  3145:   MachineBasicBlock::instr_iterator FirstBranch = BBEnd;
  3146:   MachineBasicBlock::instr_iterator LastBundledInstruction = BBEnd;
  3147:   MachineBasicBlock::instr_iterator DualJumpFirstCandidate = BBEnd;
  3148: 
  3149:   assert(MII->isBundle() && "Missing insert location");
  3150:   bool isDualJumpSecondCandidate = IsDualJumpSecondCandidate(MI);
  3151:   LastInBundle = false;
  3152: 
  3153:   for (++MII; MII != BBEnd && MII->isInsideBundle() && !MII->isBundle();
  3154:        ++MII) {
  3155:     if (MII->isBranch() && (FirstBranch == BBEnd))
  3156:       FirstBranch = MII;
  3157:     // If what we insert is a dual jump, we need to find
  3158:     // first jump, and insert new instruction after it.
  3159:     if (isDualJumpSecondCandidate && IsDualJumpFirstCandidate(&*MII))
  3160:       DualJumpFirstCandidate = MII;
  3161:     LastBundledInstruction = MII;
  3162:   }
  3163: 
  3164:   if (DualJumpFirstCandidate != BBEnd) {
  3165:     // First respect dual jumps.
  3166:     ++DualJumpFirstCandidate;
  3167:     if (DualJumpFirstCandidate == BBEnd ||
  3168:         DualJumpFirstCandidate == LastBundledInstruction)
  3169:       LastInBundle = true;
  3170:     return DualJumpFirstCandidate;
  3171:   } else if (FirstBranch != BBEnd) {
  3172:     // If we have no dual jumps, but do have a single
  3173:     // branch in the bundle, add our new instruction
  3174:     // right before it.
  3175:     return FirstBranch;
  3176:   } else if (LastBundledInstruction != BBEnd) {
  3177:     LastInBundle = true;
  3178:     return ++LastBundledInstruction;
  3179:   } else
  3180:     llvm_unreachable("Lost in bundle");
  3181:   return MBB->instr_begin();
  3182: }
  3183: 
  3184: /// This function for now needs to try to insert new instruction
  3185: /// in correct serial semantics fashion - i.e. find "correct" insert
  3186: /// point for instruction as if inserting in serial sequence.
  3187: MachineBasicBlock::instr_iterator HexagonGlobalSchedulerImpl::insertTempCopy(
  3188:     MachineBasicBlock *MBB, MachineBasicBlock::iterator &TargetPacket,
  3189:     MachineInstr *MI, bool DeleteOldCopy) {
  3190:   MachineBasicBlock::instr_iterator MII;
  3191:   MachineBasicBlock *CurrentBB = MI->getParent();
  3192: 
  3193:   assert(CurrentBB && "Corrupt instruction");
  3194:   // Create a temporary copy of the instruction we are considering.
  3195:   // LLVM refuses to deal with an instruction which was not inserted
  3196:   // to any BB. We can visit multiple BBs on the way "up", so we
  3197:   // create a temp copy of the original instruction and delete it later.
  3198:   // It is way cheaper than using splice and then
  3199:   // needing to undo it most of the time.
  3200:   MachineInstr *NewMI = MI->getParent()->getParent()->CloneMachineInstr(MI);
```
- EN: It declares or implements routines such as isInsideBundle, dump, DemoteToDotOld, LLVM_DEBUG, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isInsideBundle, dump, DemoteToDotOld, LLVM_DEBUG, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 3201-3400 / 第 3201-3400 行

```cpp
  3201:   // Make sure all bundling flags are cleared.
  3202:   if (NewMI->isBundledWithPred())
  3203:     NewMI->unbundleFromPred();
  3204:   if (NewMI->isBundledWithSucc())
  3205:     NewMI->unbundleFromSucc();
  3206: 
  3207:   if (DeleteOldCopy) {
  3208:     // Remove our temporary instruction.
  3209:     // MachineBasicBlock::erase method calls unbundleSingleMI()
  3210:     // prior to deletion, so we do not have to do it here.
  3211:     MachineBasicBlock::instr_iterator kill_it(MI);
  3212:     CurrentBB->erase(kill_it);
  3213:   }
  3214: 
  3215:   // If the original instruction used new value operands,
  3216:   // it might need to be changed to generic form
  3217:   // before further processing.
  3218:   if (QII->isDotNewInst(*NewMI))
  3219:     DemoteToDotOld(NewMI);
  3220: 
  3221:   // Insert new temporary instruction.
  3222:   // If this is the destination packet, insert the tmp after
  3223:   // its header. Otherwise, as second instr in BB.
  3224:   if (TargetPacket->getParent() == MBB) {
  3225:     MII = TargetPacket.getInstrIterator();
  3226: 
  3227:     if (MII->isBundled()) {
  3228:       bool LastInBundle = false;
  3229:       MachineBasicBlock::instr_iterator InsertBefore =
  3230:           findInsertPositionInBundle(TargetPacket, NewMI, LastInBundle);
  3231:       MIBundleBuilder Bundle(&*TargetPacket);
  3232:       Bundle.insert(InsertBefore, NewMI);
  3233:     } else
  3234:       MBB->insertAfter(MII, NewMI);
  3235:   } else {
  3236:     MII = MBB->instr_begin();
  3237: 
  3238:     // Skip debug instructions.
  3239:     while (MII->isDebugInstr())
  3240:       MII++;
  3241: 
  3242:     if (MII->isBundled()) {
  3243:       MIBundleBuilder Bundle(&*MII);
  3244:       Bundle.insert(++MII, NewMI);
  3245:     } else
  3246:       MBB->insertAfter(MII, NewMI);
  3247:   }
  3248:   return NewMI->getIterator();
  3249: }
  3250: 
  3251: // Check for a conditionally assigned register within the block.
  3252: bool HexagonGlobalSchedulerImpl::MIsCondAssign(MachineInstr *BMI,
  3253:                                                MachineInstr *MI,
  3254:                                                SmallVector<unsigned, 4> &Defs) {
  3255:   if (!QII->isPredicated(*BMI))
  3256:     return false;
  3257:   // Its a conditional instruction, now is it the same registers as MI?
  3258:   SmallVector<unsigned, 4> CondDefs;
  3259:   SmallVector<unsigned, 8> CondUses;
  3260:   parseOperands(BMI, CondDefs, CondUses);
  3261: 
  3262:   for (SmallVector<unsigned, 4>::iterator ID = Defs.begin(), IDE = Defs.end();
  3263:        ID != IDE; ++ID) {
  3264:     for (SmallVector<unsigned, 4>::iterator CID = CondDefs.begin(),
  3265:                                             CIDE = CondDefs.end();
  3266:          CID != CIDE; ++CID) {
  3267:       if (AliasingRegs(*CID, *ID)) {
  3268:         LLVM_DEBUG(dbgs() << "\tFound conditional def, can't move\n";
  3269:                    BMI->dump());
  3270:         return true;
  3271:       }
  3272:     }
  3273:   }
  3274:   return false;
  3275: }
  3276: 
  3277: // Returns the Union of all the elements in Set1 and
  3278: // Union of all the elements in Set2 separately.
  3279: // Constraints:
  3280: // Set1 and Set2 should contain an entry for each element in Range.
  3281: template <typename ElemType, typename IndexType>
  3282: void Unify(std::vector<ElemType> Range,
  3283:            std::map<ElemType, std::vector<IndexType>> &Set1,
  3284:            std::map<ElemType, std::vector<IndexType>> &Set2,
  3285:            std::pair<std::vector<IndexType>, std::vector<IndexType>> &UnionSet,
  3286:            unsigned union_size = 100) {
  3287:   typedef
  3288:       typename std::map<ElemType, std::vector<IndexType>>::iterator PosIter_t;
  3289:   typedef typename std::vector<IndexType>::iterator IndexIter_t;
  3290:   std::vector<IndexType> &Union1 = UnionSet.first;
  3291:   std::vector<IndexType> &Union2 = UnionSet.second;
  3292:   Union1.resize(union_size, 0);
  3293:   Union2.resize(union_size, 0);
  3294:   LLVM_DEBUG(dbgs() << "\n\t\tElements in the range:\n";);
  3295:   typename std::vector<ElemType>::iterator iter = Range.begin();
  3296:   while (iter != Range.end()) {
  3297:     if ((*iter)->isDebugInstr()) {
  3298:       ++iter;
  3299:       continue;
  3300:     }
  3301:     LLVM_DEBUG((*iter)->dump());
  3302:     PosIter_t set1_pos = Set1.find(*iter);
  3303:     assert(set1_pos != Set1.end() &&
  3304:            "Set1 should contain an entry for each element in Range.");
  3305:     IndexIter_t set1idx = set1_pos->second.begin();
  3306:     while (set1idx != set1_pos->second.end()) {
  3307:       Union1[*set1idx] = 1;
  3308:       ++set1idx;
  3309:     }
  3310:     PosIter_t set2_pos = Set2.find(*iter);
  3311:     assert(set2_pos != Set2.end() &&
  3312:            "Set2 should contain an entry for each element in Range.");
  3313:     IndexIter_t set2idx = set2_pos->second.begin();
  3314:     while (set2idx != set2_pos->second.end()) {
  3315:       Union2[*set2idx] = 1;
  3316:       ++set2idx;
  3317:     }
  3318:     ++iter;
  3319:   }
  3320: }
  3321: 
  3322: static void UpdateBundle(MachineInstr *BundleHead) {
  3323:   assert(BundleHead->isBundle() && "Not a bundle header");
  3324:   if (!BundleHead)
  3325:     return;
  3326:   unsigned Size = BundleHead->getBundleSize();
  3327:   if (Size >= 2)
  3328:     return;
  3329:   if (Size == 1) {
  3330:     MachineBasicBlock::instr_iterator MIter = BundleHead->getIterator();
  3331:     MachineInstr *MI = &*(++MIter);
  3332:     MI->unbundleFromPred();
  3333:   }
  3334:   BundleHead->eraseFromParent();
  3335: }
  3336: 
  3337: /// Gatekeeper for instruction speculation.
  3338: /// If all MI defs are dead (not live-in) to any other
  3339: /// BB but the one we are moving into, and it could not cause
  3340: /// exception by early execution, allow it to be pulled up.
  3341: bool HexagonGlobalSchedulerImpl::canMIBeSpeculated(
  3342:     MachineInstr *MI, MachineBasicBlock *ToBB, MachineBasicBlock *FromBB,
  3343:     std::vector<MachineInstr *> &backtrack) {
  3344:   // For now disallow memory accesses from speculation.
  3345:   // Generally we can check if they potentially may trap/cause an exception.
  3346:   if (!EnableSpeculativePullUp || !MI || MI->mayStore())
  3347:     return false;
  3348: 
  3349:   LLVM_DEBUG(dbgs() << "\t[canMIBeSpeculated] From BB(" << FromBB->getNumber()
  3350:                     << "):\t";
  3351:              MI->dump());
  3352:   LLVM_DEBUG(dbgs() << "\tTo this BB:\n"; ToBB->dump());
  3353: 
  3354:   if (!ToBB->isSuccessor(FromBB))
  3355:     return false;
  3356: 
  3357:   // This is a very tricky topic. Speculating arithmetic instructions with
  3358:   // results dead out of a loop more times then required by number of
  3359:   // iterations is safe, while speculating loads can cause an exception.
  3360:   // Simplest of checks is to not cross loop exit edge, or in our case
  3361:   // do not pull-in to a loop exit BB, but there are implications for
  3362:   // non-natural loops (not recognized by LLVM as loops) and multi-threaded
  3363:   // code.
  3364:   if (AllowSpeculateLoads && MI->mayLoad()) {
  3365:     // Invariant loads should always be safe.
  3366:     if (!MI->isDereferenceableInvariantLoad())
  3367:       return false;
  3368:     LLVM_DEBUG(dbgs() << "\tSpeculating a Load.\n");
  3369:   }
  3370: 
  3371:   SmallVector<unsigned, 4> Defs;
  3372:   SmallVector<unsigned, 8> Uses;
  3373:   parseOperands(MI, Defs, Uses);
  3374: 
  3375:   // Do not speculate instructions that modify reserved global registers.
  3376:   for (unsigned R : Defs)
  3377:     if (MRI->isReserved(R) && QRI->isGlobalReg(R))
  3378:       return false;
  3379: 
  3380:   for (MachineBasicBlock::const_succ_iterator SI = ToBB->succ_begin(),
  3381:                                               SE = ToBB->succ_end();
  3382:        SI != SE; ++SI) {
  3383:     // TODO: Allow an instruction (I) which 'defines' the live-in reg (R)
  3384:     // along the path when I is the first instruction to use the R.
  3385:     // i.e., I kills R before any other instruction in the BB uses it.
  3386:     // TODO: We have already parsed live sets - reuse them.
  3387:     if (*SI == FromBB)
  3388:       continue;
  3389:     LLVM_DEBUG(dbgs() << "\tTarget succesor BB to check:\n"; (*SI)->dump());
  3390:     LLVM_DEBUG(
  3391:         for (MachineBasicBlock::const_succ_iterator SII = (*SI)->succ_begin(),
  3392:              SIE = (*SI)->succ_end();
  3393:              SII != SIE; ++SII)(*SII)
  3394:             ->dump());
  3395:     for (MachineBasicBlock::livein_iterator I = (*SI)->livein_begin(),
  3396:                                             E = (*SI)->livein_end();
  3397:          I != E; ++I)
  3398:       for (SmallVector<unsigned, 4>::iterator ID = Defs.begin(),
  3399:                                               IDE = Defs.end();
  3400:            ID != IDE; ++ID) {
```
- EN: It declares or implements routines such as unbundleSingleMI, erase, getInstrIterator, findInsertPositionInBundle, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 unbundleSingleMI, erase, getInstrIterator, findInsertPositionInBundle, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 3401-3600 / 第 3401-3600 行

```cpp
  3401:         if (AliasingRegs((*I).PhysReg, *ID))
  3402:           return false;
  3403:       }
  3404: 
  3405:     // Check the successor blocks for conditional define.
  3406:     // TODO: We should really test the whole path here.
  3407:     for (MachineBasicBlock::instr_iterator BI = (*SI)->instr_begin(),
  3408:                                            E = (*SI)->instr_end();
  3409:          BI != E; ++BI) {
  3410:       if (BI->isBundle() || BI->isDebugInstr())
  3411:         continue;
  3412:       LLVM_DEBUG(dbgs() << "\t\tcheck against:\t"; BI->dump());
  3413:       if (MIsCondAssign(&*BI, MI, Defs))
  3414:         return false;
  3415:     }
  3416:   }
  3417:   // Taking a very conservative approach during speculation.
  3418:   // Traverse the path (FromBB, ToBB] and make sure
  3419:   // that the def-use set of the instruction to be moved
  3420:   // are not modified.
  3421:   std::vector<MachineBasicBlock *> PathBB;
  3422:   for (unsigned i = 0; i < backtrack.size(); ++i) {
  3423:     // Insert unique BB along the path but skip FromBB
  3424:     MachineBasicBlock *MBB = backtrack[i]->getParent();
  3425:     if ((MBB != FromBB) &&
  3426:         (std::find(PathBB.begin(), PathBB.end(), MBB) == PathBB.end()))
  3427:       PathBB.push_back(MBB);
  3428:   }
  3429:   bool WaitingForTargetPacket = true;
  3430:   MachineBasicBlock::instr_iterator MII;
  3431:   std::vector<MachineInstr *> TraversalRange;
  3432:   LLVM_DEBUG(dbgs() << "\n\tElements in the range:");
  3433:   // TODO: Use just the backtrack to get TraversalRange because it
  3434:   // contains the path (only when speculated from a path in region).
  3435:   // Note: We check the dependency of instruction-to-move with
  3436:   // all the instructions (starting from backtrack[0]) in the parent BBs
  3437:   // because a BB might have a branching from in between due to packetization
  3438:   // and just checking packets in the backtrack won't be comprehensive.
  3439:   for (unsigned i = 0; i < PathBB.size(); ++i) {
  3440:     for (MII = PathBB[i]->instr_begin(); MII != PathBB[i]->instr_end(); ++MII) {
  3441:       // Skip instructions until the target packet is found.
  3442:       // although target packet is already checked for correctness,
  3443:       // it is good to check here to validate intermediate pullups.
  3444:       if (backtrack[0] == &*MII)
  3445:         WaitingForTargetPacket = false;
  3446:       if (WaitingForTargetPacket)
  3447:         continue;
  3448:       if (MII->isBundle())
  3449:         continue;
  3450:       // TODO: Ideally we should check that there is a `linear' control flow
  3451:       // in the TraversalRange in all possible manner. For e.g.,
  3452:       // BB0 { packet1: if(p0) indirect_jump BB1;
  3453:       //      packet2: jump BB2 }
  3454:       // BB1 { i1 }. In this case we should not pull `i1' into packet2.
  3455:       if (MII->isCall() || MII->isReturn() ||
  3456:           (MII->getOpcode() == Hexagon::J2_jump && !MII->getOperand(0).isMBB()))
  3457:         return false;
  3458:       if (MI != &*MII) {
  3459:         TraversalRange.push_back(&*MII);
  3460:         LLVM_DEBUG(MII->dump(););
  3461:       }
  3462:     }
  3463:   }
  3464:   // Get the union of def/use set of all the instructions along TraversalRange.
  3465:   std::pair<std::vector<unsigned>, std::vector<unsigned>> RangeDefUse;
  3466:   Unify(TraversalRange, MIDefSet, MIUseSet, RangeDefUse, QRI->getNumRegs());
  3467:   // No instruction (along TraversalRange) should 'define' the use set of MI
  3468:   for (unsigned j = 0; j < Uses.size(); ++j)
  3469:     if (RangeDefUse.first[Uses[j]]) {
  3470:       LLVM_DEBUG(dbgs() << "\n\t\tUnresolved dependency along path to HOME for "
  3471:                         << printReg(Uses[j], QRI););
  3472:       return false;
  3473:     }
  3474:   // No instruction (along TraversalRange) should 'define' or 'use'
  3475:   // the def set of MI
  3476:   for (unsigned j = 0; j < Defs.size(); ++j)
  3477:     if (RangeDefUse.first[Defs[j]] || RangeDefUse.second[Defs[j]]) {
  3478:       LLVM_DEBUG(dbgs() << "\n\t\tUnresolved dependency along path to HOME for "
  3479:                         << printReg(Defs[j], QRI););
  3480:       return false;
  3481:     }
  3482:   return true;
  3483: }
  3484: 
  3485: /// Try to move InstrToMove to TargetPacket using path stored in backtrack.
  3486: /// SourceLocation is current iterator point. It must be updated to the new
  3487: /// iteration location after all updates.
  3488: /// Alogrithm:
  3489: /// To move an instruction (I) from OriginBB through HomeBB via backtrack.
  3490: /// for each packet (i) in backtrack, analyzeBranch
  3491: /// case 1 (success)
  3492: ///   case Pulling from conditional branch:
  3493: ///     if I is predicable
  3494: ///         Try to predicate on the branch condition
  3495: ///     else
  3496: ///         Try to speculate I to backtrack[i].
  3497: ///   case Pulling from unconditional branch:
  3498: ///         Just pullup. (TODO: Speculate here as well)
  3499: /// case 2 (fails)
  3500: ///     Try to speculate I backtrack[i].
  3501: bool HexagonGlobalSchedulerImpl::MoveMItoBundle(
  3502:     BasicBlockRegion *CurrentRegion,
  3503:     MachineBasicBlock::instr_iterator &InstrToMove,
  3504:     MachineBasicBlock::iterator &NextMI,
  3505:     MachineBasicBlock::iterator &TargetPacket,
  3506:     MachineBasicBlock::iterator &SourceLocation,
  3507:     std::vector<MachineInstr *> &backtrack, bool MovingDependentOp,
  3508:     bool PathInRegion) {
  3509:   MachineBasicBlock *HomeBB = TargetPacket->getParent();
  3510:   MachineBasicBlock *OriginBB = InstrToMove->getParent();
  3511:   MachineBasicBlock *CurrentBB = OriginBB;
  3512:   MachineBasicBlock *CleanupBB = OriginBB;
  3513:   MachineBasicBlock *PreviousBB = OriginBB;
  3514:   MachineInstr *OriginalInstructionToMove = &*InstrToMove;
  3515: 
  3516:   assert(HomeBB && "Missing HomeBB");
  3517:   assert(OriginBB && "Missing OriginBB");
  3518: 
  3519:   LLVM_DEBUG(dbgs() << "\n.........[MoveMItoBundle]..............\n");
  3520:   LLVM_DEBUG(dbgs() << "\t\tInstrToMove   :\t"; InstrToMove->dump());
  3521:   LLVM_DEBUG(dbgs() << "\t\tTargetPacket  :\t";
  3522:              DumpPacket(TargetPacket.getInstrIterator()));
  3523:   LLVM_DEBUG(dbgs() << "\t\tSourceLocation:\t";
  3524:              DumpPacket(SourceLocation.getInstrIterator()));
  3525: 
  3526:   // We do not allow to move instructions in the same BB.
  3527:   if (HomeBB == OriginBB) {
  3528:     LLVM_DEBUG(dbgs() << "\t\tSame BB pull-up.\n");
  3529:     if (!EnableLocalPullUp)
  3530:       return false;
  3531:   }
  3532: 
  3533:   if (OneFloatPerPacket && QII->isFloat(*TargetPacket) &&
  3534:       QII->isFloat(*InstrToMove))
  3535:     return false;
  3536: 
  3537:   if (OneComplexPerPacket && QII->isComplex(*TargetPacket) &&
  3538:       QII->isComplex(*InstrToMove))
  3539:     return false;
  3540: 
  3541:   LLVM_DEBUG(dbgs() << "\t\tWay home:\n");
  3542:   // Test integrity of the back track.
  3543:   for (unsigned i = 0; i < backtrack.size(); ++i) {
  3544:     assert(backtrack[i]->getParent() && "Messed back track.");
  3545:     LLVM_DEBUG(dbgs() << "\t\t[" << i << "] BB("
  3546:                       << backtrack[i]->getParent()->getNumber() << ")\t";
  3547:                backtrack[i]->dump());
  3548:   }
  3549:   LLVM_DEBUG(dbgs() << "\n");
  3550: 
  3551:   bool NeedCleanup = false;
  3552:   bool NeedToPredicate = false;
  3553:   bool MINeedToNewify = false;
  3554:   unsigned DepReg = std::numeric_limits<unsigned>::max();
  3555:   bool isDualJump = false;
  3556:   SmallVector<MachineOperand, 4> Cond;
  3557:   SmallVector<MachineOperand, 4> PredCond;
  3558:   std::vector<MachineInstr *> PullUpPath;
  3559:   if (PathInRegion)
  3560:     PullUpPath = backtrack;
  3561:   else {
  3562:     PullUpPath.push_back(&*TargetPacket);
  3563:     PullUpPath.push_back(&*InstrToMove);
  3564:   }
  3565: 
  3566:   // Now start iterating over all instructions
  3567:   // preceeding the one we are trying to move,
  3568:   // and see if they could be reodered/bypassed.
  3569:   for (std::vector<MachineInstr *>::reverse_iterator RI = backtrack.rbegin(),
  3570:                                                      RIE = backtrack.rend();
  3571:        RI < RIE; ++RI) {
  3572:     // Once most of debug will be gone, this will be a real assert.
  3573:     // assert((backtrack.front() == ToThisBundle) && "Lost my way home.");
  3574:     MachineInstr *MIWH = *RI;
  3575:     if (QII->isDotNewInst(*InstrToMove)) {
  3576:       LLVM_DEBUG(dbgs() << "Cannot move a dot new instruction:";
  3577:                  InstrToMove->dump());
  3578:       if (NeedCleanup)
  3579:         CleanupBB->erase(InstrToMove);
  3580:       return false;
  3581:     }
  3582:     if (canCauseStall(&*InstrToMove, MIWH)) {
  3583:       if (NeedCleanup)
  3584:         CleanupBB->erase(InstrToMove);
  3585:       return false;
  3586:     }
  3587:     LLVM_DEBUG(dbgs() << "\t> Step home BB(" << MIWH->getParent()->getNumber()
  3588:                       << "):\t";
  3589:                DumpPacket(MIWH->getIterator()));
  3590: 
  3591:     // See if we cross a jump, and possibly change the form of instruction.
  3592:     // Passing through BBs with dual jumps in different packets
  3593:     // takes extra care.
  3594:     bool isBranchMIWH = isBranch(MIWH);
  3595:     if (((&*SourceLocation != MIWH) && isBranchMIWH) ||
  3596:         (CurrentBB != MIWH->getParent())) {
  3597:       LLVM_DEBUG(dbgs() << "\tChange BB from(" << CurrentBB->getNumber()
  3598:                         << ") to (" << MIWH->getParent()->getNumber() << ")\n");
  3599:       PreviousBB = CurrentBB;
  3600:       CurrentBB = MIWH->getParent();
```
- EN: It declares or implements routines such as dump, size, getParent, LLVM_DEBUG, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dump, size, getParent, LLVM_DEBUG, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 3601-3800 / 第 3601-3800 行

```cpp
  3601: 
  3602:       // See what kind of branch we are dealing with.
  3603:       MachineBasicBlock *PredTBB = NULL;
  3604:       MachineBasicBlock *PredFBB = NULL;
  3605: 
  3606:       if (QII->analyzeBranch(*CurrentBB, PredTBB, PredFBB, Cond, false)) {
  3607:         // We currently do not handle NV jumps of this kind:
  3608:         // if (cmp.eq(r0.new, #0)) jump:t .LBB12_69
  3609:         // TODO: Need to handle them.
  3610:         LLVM_DEBUG(dbgs() << "\tCould not analyze branch.\n");
  3611: 
  3612:         // This is the main point of lost performance.
  3613:         // We could try to speculate here, but for that we need accurate
  3614:         // liveness info, and it is not ready yet.
  3615:         if (!canMIBeSpeculated(&*InstrToMove, CurrentBB, PreviousBB,
  3616:                                PullUpPath)) {
  3617:           if (NeedCleanup)
  3618:             CleanupBB->erase(InstrToMove);
  3619:           return false;
  3620:         } else {
  3621:           // Save speculated instruction moved.
  3622:           SpeculatedIns.insert(
  3623:               std::make_pair(OriginalInstructionToMove, OriginBB));
  3624:           LLVM_DEBUG(dbgs() << "\nSpeculatedInsToMove"; InstrToMove->dump());
  3625:         }
  3626: 
  3627:         LLVM_DEBUG(dbgs() << "\tSpeculating.\n");
  3628:         // If we are speculating, we can come through a predication
  3629:         // into an unconditional branch...
  3630:         // For now simply bail out.
  3631:         // TODO: See if this ever happens.
  3632:         if (NeedToPredicate) {
  3633:           LLVM_DEBUG(dbgs()
  3634:                      << "\tUnderimplemented pred for speculative move.\n");
  3635:           if (NeedCleanup)
  3636:             CleanupBB->erase(InstrToMove);
  3637:           return false;
  3638:         }
  3639:         InstrToMove =
  3640:             insertTempCopy(CurrentBB, TargetPacket, &*InstrToMove, NeedCleanup);
  3641:         NeedCleanup = true;
  3642:         NeedToPredicate = false;
  3643:         assert(!NeedToPredicate && "Need to handle predication for this case");
  3644:         CleanupBB = CurrentBB;
  3645:         // No need to recheck for resources - instruction did not change.
  3646:         LLVM_DEBUG(dbgs() << "\tUpdated BB:\n"; CurrentBB->dump());
  3647:       } else {
  3648:         bool LocalNeedPredication = true;
  3649:         // We were able to analyze the branch.
  3650:         if (!isBranchMIWH && !PredTBB) {
  3651:           LLVM_DEBUG(dbgs() << "\tDo not need predicate for this case.\n");
  3652:           LocalNeedPredication = false;
  3653:         }
  3654:         // First see if this is a potential dual jump situation.
  3655:         if (IsDualJumpSecondCandidate(&*InstrToMove) &&
  3656:             IsDualJumpFirstCandidate(TargetPacket)) {
  3657:           LLVM_DEBUG(dbgs() << "\tPerforming unrestricted dual jump.\n");
  3658:           isDualJump = true;
  3659:         } else if (LocalNeedPredication && (PredFBB != PreviousBB)) {
  3660:           // Predicate instruction based on condition feeding it.
  3661:           // This is generally a statefull pull-up path.
  3662:           // Can this insn be predicated? If so, try to do it.
  3663:           if (TII->isPredicable(*InstrToMove)) {
  3664:             if (PredTBB) {
  3665:               if (PreviousBB != PredTBB) {
  3666:                 // If we "came" not from TBB, we need to invert condition.
  3667:                 if (TII->reverseBranchCondition(Cond)) {
  3668:                   LLVM_DEBUG(dbgs() << "\tUnable to invert condition.\n");
  3669:                   if (NeedCleanup)
  3670:                     CleanupBB->erase(InstrToMove);
  3671:                   return false;
  3672:                 }
  3673:               }
  3674:               LLVM_DEBUG(dbgs() << "\tTBB(" << PredTBB->getNumber()
  3675:                                 << ")InvertCondition("
  3676:                                 << (PreviousBB != PredTBB) << ")\n");
  3677:             }
  3678:             // Create a new copy of the instruction we are trying to move.
  3679:             // It changes enough (new BB, predicated form) and untill we
  3680:             // reach home, we do not even know if it is going to work.
  3681:             InstrToMove = insertTempCopy(CurrentBB, TargetPacket, &*InstrToMove,
  3682:                                          NeedCleanup);
  3683:             NeedCleanup = true;
  3684:             NeedToPredicate = true;
  3685:             CleanupBB = CurrentBB;
  3686: 
  3687:             if (PredCond.empty() && // If not already predicated.
  3688:                 TII->PredicateInstruction(*InstrToMove, Cond)) {
  3689:               LLVM_DEBUG(dbgs() << "\tNew predicated insn:\t";
  3690:                          InstrToMove->dump());
  3691:               // After predication some instruction could become const extended:
  3692:               // L2_loadrigp == "$dst=memw(#$global)"
  3693:               // L4_ploadrit_abs == "if ($src1) $dst=memw(##$global)"
  3694:               // Resource checking for those is different.
  3695:               if ((QII->isExtended(*InstrToMove) ||
  3696:                    QII->isConstExtended(*InstrToMove) ||
  3697:                    isJumpOutOfRange(&*InstrToMove)) &&
  3698:                   !tryAllocateResourcesForConstExt(&*InstrToMove, false)) {
  3699:                 // If we cannot, do not modify the state.
  3700:                 LLVM_DEBUG(dbgs()
  3701:                            << "\tEI Could not be added to the packet.\n");
  3702:                 CleanupBB->erase(InstrToMove);
  3703:                 return false;
  3704:               }
  3705: 
  3706:               if (!ResourceTracker->canReserveResources(*InstrToMove) ||
  3707:                   !shouldAddToPacket(*InstrToMove)) {
  3708:                 // It will not fit in its new form...
  3709:                 LLVM_DEBUG(dbgs() << "\tCould not be added in its new form.\n");
  3710:                 CurrentBB->erase(InstrToMove);
  3711:                 return false;
  3712:               }
  3713: 
  3714:               // Need also verify that we can newify it if we want to.
  3715:               if (NeedToNewify(InstrToMove, &DepReg, &*TargetPacket)) {
  3716:                 if (isNewifiable(InstrToMove, DepReg, &*TargetPacket)) {
  3717:                   MINeedToNewify = true;
  3718:                   LLVM_DEBUG(dbgs() << "\t\t\tNeeds to NEWify on Reg("
  3719:                                     << printReg(DepReg, QRI) << ").\n");
  3720:                 } else {
  3721:                   LLVM_DEBUG(dbgs() << "\tNon newifiable in this bundle: ";
  3722:                              InstrToMove->dump());
  3723:                   CleanupBB->erase(InstrToMove);
  3724:                   return false;
  3725:                 }
  3726:               }
  3727: 
  3728:               LLVM_DEBUG(dbgs() << "\tUpdated BB:\n"; CurrentBB->dump());
  3729:               PredCond = Cond;
  3730:               // Now the instruction uses the pred-reg as well.
  3731:               if (!Cond.empty() && (Cond.size() == 2)) {
  3732:                 MIUseSet[OriginalInstructionToMove].push_back(Cond[1].getReg());
  3733:               }
  3734:               assert(((Cond.size() <= 2) &&
  3735:                       !(QII->isNewValueJump(Cond[0].getImm()))) &&
  3736:                      "Update MIUseSet for new-value compare jumps");
  3737:             } else {
  3738:               LLVM_DEBUG(dbgs() << "\tCould not predicate it\n");
  3739:               LLVM_DEBUG(dbgs() << "\tTrying to speculate!\t";
  3740:                          InstrToMove->dump());
  3741:               bool DistantSpeculation = false;
  3742:               std::vector<MachineInstr *> NonPredPullUpPath;
  3743:               unsigned btidx = 0;
  3744:               // Generate a backtrack path for instruction to be speculated.
  3745:               // Original backtrack may start from a different (ancestor)
  3746:               // target packet.
  3747:               while (btidx < backtrack.size()) {
  3748:                 const MachineBasicBlock *btBB = backtrack[btidx]->getParent();
  3749:                 if ((btBB == PreviousBB) || (btBB == CurrentBB))
  3750:                   NonPredPullUpPath.push_back(backtrack[btidx]);
  3751:                 ++btidx;
  3752:               }
  3753:               // Speculate only to immediate predecessor.
  3754:               if (PreviousBB != CurrentBB) {
  3755:                 if (*(PreviousBB->pred_begin()) != CurrentBB) {
  3756:                   // In a region there are no side entries.
  3757:                   DistantSpeculation = true;
  3758:                   LLVM_DEBUG(dbgs()
  3759:                                  << "\n\tMI not in immediate successor of BB#"
  3760:                                  << CurrentBB->getNumber() << ", MI is in BB#"
  3761:                                  << PreviousBB->getNumber(););
  3762:                 }
  3763:                 assert((PreviousBB->pred_size() < 2) &&
  3764:                        "Region with a side entry");
  3765:               }
  3766:               // TODO: Speculate ins. when pulled from unlikely path.
  3767:               if (DistantSpeculation || /*!PathInRegion ||*/
  3768:                   InstrToMove->mayLoad() || InstrToMove->mayStore() ||
  3769:                   InstrToMove->hasUnmodeledSideEffects() ||
  3770:                   !canMIBeSpeculated(&*InstrToMove, CurrentBB, PreviousBB,
  3771:                                      NonPredPullUpPath)) {
  3772:                 CleanupBB->erase(InstrToMove);
  3773:                 return false;
  3774:               } else {
  3775:                 // Save speculated instruction moved.
  3776:                 NeedToPredicate = false;
  3777:                 SpeculatedIns.insert(
  3778:                     std::make_pair(OriginalInstructionToMove, OriginBB));
  3779:                 LLVM_DEBUG(dbgs() << "\nPredicable+SpeculatedInsToMove";
  3780:                            InstrToMove->dump());
  3781:               }
  3782:             }
  3783:           } else {
  3784:             // This is a non-predicable instruction. We still can try to
  3785:             // speculate it here.
  3786:             LLVM_DEBUG(dbgs() << "\tNon predicable insn!\t";
  3787:                        InstrToMove->dump());
  3788:             // TODO: Speculate ins. when pulled from unlikely path.
  3789:             if (!SpeculateNonPredInsn || !PathInRegion ||
  3790:                 InstrToMove->mayLoad() || InstrToMove->mayStore() ||
  3791:                 InstrToMove->hasUnmodeledSideEffects() ||
  3792:                 !canMIBeSpeculated(&*InstrToMove, CurrentBB, PreviousBB,
  3793:                                    PullUpPath)) {
  3794:               if (NeedCleanup)
  3795:                 CleanupBB->erase(InstrToMove);
  3796:               return false;
  3797:             } else {
  3798:               // Save speculated instruction moved.
  3799:               SpeculatedIns.insert(
  3800:                   std::make_pair(OriginalInstructionToMove, OriginBB));
```
- EN: It declares or implements routines such as insert, dump, LLVM_DEBUG, insertTempCopy, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 insert, dump, LLVM_DEBUG, insertTempCopy, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 3801-4000 / 第 3801-4000 行

```cpp
  3801:               LLVM_DEBUG(dbgs() << "\nNonPredicable+SpeculatedInsToMove";
  3802:                          InstrToMove->dump());
  3803:             }
  3804: 
  3805:             InstrToMove = insertTempCopy(CurrentBB, TargetPacket, &*InstrToMove,
  3806:                                          NeedCleanup);
  3807:             NeedCleanup = true;
  3808:             CleanupBB = CurrentBB;
  3809:           }
  3810:         } else {
  3811:           // No branch. Fall through.
  3812:           LLVM_DEBUG(dbgs() << "\tFall through BB.\n"
  3813:                             << "\tCurrentBB:" << CurrentBB->getNumber()
  3814:                             << "\tPreviousBB:" << PreviousBB->getNumber();
  3815:                      if (PredFBB) dbgs()
  3816:                      << "\tPredFBB:" << PredFBB->getNumber(););
  3817:           // Even though this is a fall though case, we still can
  3818:           // have a dual jump situation here with a CALL involved.
  3819:           // For now simply avoid it.
  3820:           if (IsDualJumpSecondCandidate(&*InstrToMove)) {
  3821:             llvm_unreachable("Dual jumps with known?");
  3822:             LLVM_DEBUG(dbgs() << "\tUnderimplemented dual jump formation.\n");
  3823:             if (NeedCleanup)
  3824:               CleanupBB->erase(InstrToMove);
  3825:             return false;
  3826:           }
  3827: 
  3828:           if (!CurrentBB->isSuccessor(PreviousBB)) {
  3829:             LLVM_DEBUG(dbgs() << "\tNon-successor fall through.\n");
  3830:             if (NeedCleanup)
  3831:               CleanupBB->erase(InstrToMove);
  3832:             return false;
  3833:           }
  3834:           SpeculatedIns.insert(
  3835:               std::make_pair(OriginalInstructionToMove, OriginBB));
  3836:           LLVM_DEBUG(dbgs() << "\nSpeculatedInsToMove+FallThroughBB";
  3837:                      InstrToMove->dump());
  3838:           // Create a temp copy.
  3839:           InstrToMove = insertTempCopy(CurrentBB, TargetPacket, &*InstrToMove,
  3840:                                        NeedCleanup);
  3841:           NeedCleanup = true;
  3842:           NeedToPredicate = false;
  3843:           CleanupBB = CurrentBB;
  3844:           LLVM_DEBUG(dbgs() << "\tUpdated BB:\n"; CurrentBB->dump());
  3845:         }
  3846:       }
  3847:     }
  3848:     // If we have reached Home, great.
  3849:     // Original check should have verified that instruction could be added
  3850:     // to the target packet, so here we do nothing for deps.
  3851:     if (MIWH == backtrack.front()) {
  3852:       LLVM_DEBUG(dbgs() << "\tHOME!\n");
  3853:       break;
  3854:     }
  3855: 
  3856:     // Test if we can reorder the two MIs.
  3857:     // The exception is when we are forming dual jumps - we can pull up
  3858:     // dependent instruction to the last bundle of an immediate predecesor
  3859:     // of the current BB if control flow permits it.
  3860:     // In this special case we also need to update the bundle we are moving
  3861:     // from.
  3862:     if (!(MovingDependentOp && (MIWH == &*SourceLocation)) &&
  3863:         !canReorderMIs(MIWH, &*InstrToMove)) {
  3864:       if (NeedCleanup)
  3865:         CleanupBB->erase(InstrToMove);
  3866:       return false;
  3867:     }
  3868:   }
  3869:   // We have previously tested this instruction, but has not updated the state
  3870:   // for it. Do it now.
  3871:   if (QII->isExtended(*InstrToMove) || QII->isConstExtended(*InstrToMove) ||
  3872:       isJumpOutOfRange(&*InstrToMove)) {
  3873:     if (!tryAllocateResourcesForConstExt(&*InstrToMove))
  3874:       llvm_unreachable("Missed dependency test");
  3875:   }
  3876: 
  3877:   // Ok. We can safely move this instruction all the way up.
  3878:   // We also potentially have a slot for it.
  3879:   // During move original instruction could have changed (becoming predicated).
  3880:   // Now try to place the final instance of it into the current packet.
  3881:   LLVM_DEBUG(dbgs() << "\nWant to move ";
  3882:              if (MovingDependentOp) dbgs() << "dependent op"; dbgs() << ": ";
  3883:              InstrToMove->dump(); dbgs() << "To BB:\n"; HomeBB->dump();
  3884:              dbgs() << "From BB:\n"; OriginBB->dump());
  3885: 
  3886:   // Keep these two statistics separately.
  3887:   if (!isDualJump)
  3888:     HexagonNumPullUps++;
  3889:   else
  3890:     HexagonNumDualJumps++;
  3891: 
  3892:   // This means we have not yet inserted the temp copy of InstrToMove
  3893:   // in the target bundle. We are probably inside the same BB.
  3894:   if (!NeedCleanup) {
  3895:     InstrToMove =
  3896:         insertTempCopy(HomeBB, TargetPacket, &*InstrToMove, NeedCleanup);
  3897:     NeedCleanup = true;
  3898:   }
  3899: 
  3900:   // No problems detected. Add it.
  3901:   // If we were adding InstrToMove to a single, not yet packetized
  3902:   // instruction, we need to create bundle header for it before proceeding.
  3903:   // Be carefull since endPacket also resets the DFA state.
  3904:   if (!TargetPacket->isBundle()) {
  3905:     LLVM_DEBUG(dbgs() << "\tForm a new bundle.\n");
  3906:     finalizeBundle(*HomeBB, TargetPacket.getInstrIterator(),
  3907:                    std::next(InstrToMove));
  3908:     LLVM_DEBUG(HomeBB->dump());
  3909:     // Now we need to adjust pointer to the newly created packet header.
  3910:     MachineBasicBlock::instr_iterator MII = TargetPacket.getInstrIterator();
  3911:     MII--;
  3912: 
  3913:     // Is it also on the way home?
  3914:     for (unsigned i = 0; i < backtrack.size(); ++i)
  3915:       if (backtrack[i] == &*TargetPacket)
  3916:         backtrack[i] = &*MII;
  3917: 
  3918:     // Is it where our next MI is pointing?
  3919:     if (NextMI == TargetPacket)
  3920:       NextMI = MII;
  3921:     TargetPacket = MII;
  3922:   }
  3923: 
  3924:   // Move and Update Liveness info.
  3925:   MoveAndUpdateLiveness(CurrentRegion, HomeBB, &*InstrToMove, MINeedToNewify,
  3926:                         DepReg, MovingDependentOp, OriginBB,
  3927:                         OriginalInstructionToMove, PredCond, SourceLocation,
  3928:                         TargetPacket, NextMI, backtrack);
  3929: 
  3930:   LLVM_DEBUG(dbgs() << "\n______Updated______\n"; HomeBB->dump();
  3931:              OriginBB->dump());
  3932: 
  3933:   return true;
  3934: }
  3935: 
  3936: /// Verify that we respect CFG layout during pull-up.
  3937: bool HexagonGlobalSchedulerImpl::isBranchWithinRegion(
  3938:     BasicBlockRegion *CurrentRegion, MachineInstr *MI) {
  3939:   assert(MI && MI->isBranch() && "Missing call info");
  3940: 
  3941:   MachineBasicBlock *MBB = MI->getParent();
  3942:   LLVM_DEBUG(dbgs() << "\t[isBranchWithinRegion] BB(" << MBB->getNumber()
  3943:                     << ") Branch instr:\t";
  3944:              MI->dump());
  3945:   // If there is only one successor, it is safe to pull.
  3946:   if (MBB->succ_size() <= 1)
  3947:     return true;
  3948:   // If there are multiple successors (jump table), we should
  3949:   // not allow pull up over this instruction.
  3950:   if (MBB->succ_size() > 2)
  3951:     return false;
  3952: 
  3953:   MachineBasicBlock *NextRegionBB;
  3954:   MachineBasicBlock *TBB, *FBB;
  3955:   MachineInstr *FirstTerm = NULL;
  3956:   MachineInstr *SecondTerm = NULL;
  3957: 
  3958:   if (AnalyzeBBBranches(MBB, TBB, FirstTerm, FBB, SecondTerm)) {
  3959:     LLVM_DEBUG(dbgs() << "\t\tAnalyzeBBBranches failed!\n");
  3960:     return false;
  3961:   }
  3962: 
  3963:   // If there is no jump in this BB, it simply falls through.
  3964:   if (!FirstTerm) {
  3965:     LLVM_DEBUG(dbgs() << "\t\tNo FirstTerm\n");
  3966:     return true;
  3967:   } else if (QII->isEndLoopN(FirstTerm->getOpcode())) {
  3968:     // We can easily analyze where endloop would take us
  3969:     // but here it would be pointless either way since
  3970:     // the region will not cross it.
  3971:     LLVM_DEBUG(dbgs() << "\t\tEndloop terminator\n");
  3972:     return false;
  3973:   }
  3974:   // On some occasions we see code like this:
  3975:   // BB#142: derived from LLVM BB %init, Align 4 (16 bytes)
  3976:   //  Live Ins: %R17 %R18
  3977:   //  Predecessors according to CFG: BB#2
  3978:   //  EH_LABEL <MCSym=.Ltmp35>
  3979:   //  J2_jump <BB#3>, %PC<imp-def>
  3980:   //  Successors according to CFG: BB#3(1048575) BB#138(1)
  3981:   // It breaks most assumptions about CFG layout, so untill we know
  3982:   // the source of it, let's have a safeguard.
  3983:   if (MBB->succ_size() > 1 && !TII->isPredicated(*FirstTerm) &&
  3984:       !QII->isNewValueJump(*FirstTerm)) {
  3985:     LLVM_DEBUG(dbgs() << "\t\tBadly formed BB.\n");
  3986:     return false;
  3987:   }
  3988: 
  3989:   LLVM_DEBUG(dbgs() << "\t\tFirstTerm:  "; FirstTerm->dump());
  3990:   LLVM_DEBUG(dbgs() << "\t\tSecondTerm: "; if (SecondTerm) SecondTerm->dump();
  3991:              else dbgs() << "None\n";);
  3992: 
  3993:   // All cases where there is only one branch in BB are OK to proceed.
  3994:   if (!SecondTerm)
  3995:     return true;
  3996: 
  3997:   assert(!QII->isEndLoopN(SecondTerm->getOpcode()) && "Found endloop.");
  3998: 
  3999:   // Find next BB in this region - if there is none, we will likely
  4000:   // stop pulling in the next check outside of this function.
```
- EN: It declares or implements routines such as dump, insertTempCopy, LLVM_DEBUG, llvm_unreachable, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonNumPullUps, HexagonNumDualJumps, HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dump, insertTempCopy, LLVM_DEBUG, llvm_unreachable, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonNumPullUps, HexagonNumDualJumps, HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 4001-4200 / 第 4001-4200 行

```cpp
  4001:   // This largely is don't care.
  4002:   NextRegionBB = CurrentRegion->findNextMBB(MBB);
  4003:   if (!NextRegionBB) {
  4004:     LLVM_DEBUG(dbgs() << "\t\tNo next BB in the region...\n");
  4005:     return true;
  4006:   }
  4007:   LLVM_DEBUG(dbgs() << "\t\tNextRegionBB(" << NextRegionBB->getNumber()
  4008:                     << ")\n");
  4009:   assert(TBB && "Corrupt BB layout");
  4010:   // This means we are trying to pull into a packet _before_ the first
  4011:   // branch in the MBB.
  4012:   if (MI == FirstTerm) {
  4013:     LLVM_DEBUG(dbgs() << "\t\tTBB(" << TBB->getNumber()
  4014:                       << ") NextBB in the region(" << NextRegionBB->getNumber()
  4015:                       << ")\n");
  4016:     return (TBB == NextRegionBB);
  4017:   }
  4018:   assert(FBB && "Corrupt BB layout");
  4019:   // This means we are trying to pull into the packet _after_ first branch,
  4020:   // and it is OK if we pull from the second branch target.
  4021:   // This pull is always speculative.
  4022:   if ((MI != SecondTerm)) {
  4023:     LLVM_DEBUG(dbgs() << "\t\tDual terminator not matching SecondTerm.\n");
  4024:     return false;
  4025:   }
  4026:   // Analyze the second branch in the BB.
  4027:   LLVM_DEBUG(dbgs() << "\t\tFBB(" << FBB->getNumber()
  4028:                     << ") NextBB in the region(" << NextRegionBB->getNumber()
  4029:                     << ")\n");
  4030:   return (FBB == NextRegionBB);
  4031: }
  4032: 
  4033: /// Check if a given instruction is:
  4034: /// - a jump to a distant target
  4035: /// - that exceeds its immediate range
  4036: /// If both conditions are true, it requires constant extension.
  4037: bool HexagonGlobalSchedulerImpl::isJumpOutOfRange(MachineInstr *UnCond,
  4038:                                                   MachineInstr *Cond) {
  4039:   if (!UnCond || !UnCond->isBranch())
  4040:     return false;
  4041: 
  4042:   MachineBasicBlock *UnCondBB = UnCond->getParent();
  4043:   MachineBasicBlock *CondBB = Cond->getParent();
  4044:   MachineInstr *FirstTerm = &*(CondBB->getFirstInstrTerminator());
  4045:   // This might be worth an assert.
  4046:   if (FirstTerm == &*CondBB->instr_end())
  4047:     return false;
  4048: 
  4049:   unsigned InstOffset = BlockToInstOffset[UnCondBB];
  4050:   unsigned Distance = 0;
  4051: 
  4052:   // To save time, estimate exact position of a branch instruction
  4053:   // as one at the end of the UnCondBB.
  4054:   // Number of instructions times typical instruction size.
  4055:   InstOffset += (QII->nonDbgBBSize(UnCondBB) * HEXAGON_INSTR_SIZE);
  4056: 
  4057:   MachineBasicBlock *TBB = NULL, *FBB = NULL;
  4058:   SmallVector<MachineOperand, 4> CondList;
  4059: 
  4060:   // Find the target of the unconditional branch in UnCondBB, which is returned
  4061:   // in TBB. Then use the CondBB to extract the FirsTerm. We desire to replace
  4062:   // the branch target in FirstTerm with the branch location from the UnCondBB,
  4063:   // provided it is within the distance of the opcode in FirstTerm.
  4064:   if (QII->analyzeBranch(*UnCondBB, TBB, FBB, CondList, false))
  4065:     // Could not analyze it. give up.
  4066:     return false;
  4067: 
  4068:   if (TBB && (Cond == FirstTerm)) {
  4069:     Distance =
  4070:         (unsigned)std::abs((long long)InstOffset - BlockToInstOffset[TBB]) +
  4071:         SafetyBuffer;
  4072:     return !QII->isJumpWithinBranchRange(*FirstTerm, Distance);
  4073:   }
  4074:   return false;
  4075: }
  4076: 
  4077: // findBundleAndBranch returns the branch instruction and the
  4078: // bundle which contains it. Null is returned if not found.
  4079: MachineInstr *HexagonGlobalSchedulerImpl::findBundleAndBranch(
  4080:     MachineBasicBlock *BB, MachineBasicBlock::iterator &Bundle) {
  4081:   // Find the conditional branch out of BB.
  4082:   if (!BB)
  4083:     return NULL;
  4084:   MachineInstr *CondBranch = NULL;
  4085:   Bundle = BB->end();
  4086:   for (MachineBasicBlock::instr_iterator MII = BB->getFirstInstrTerminator(),
  4087:                                          MBBEnd = BB->instr_end();
  4088:        MII != MBBEnd; ++MII) {
  4089:     MachineInstr *MI = &*MII;
  4090:     if (MII->isConditionalBranch()) {
  4091:       CondBranch = MI;
  4092:     }
  4093:   }
  4094:   if (!CondBranch)
  4095:     return NULL;
  4096:   MachineBasicBlock::instr_iterator MII = CondBranch->getIterator();
  4097:   if (!MII->isBundled())
  4098:     return NULL;
  4099:   // Find bundle header.
  4100:   for (--MII; MII->isBundled(); --MII)
  4101:     if (MII->isBundle()) {
  4102:       Bundle = MII;
  4103:       break;
  4104:     }
  4105:   return CondBranch;
  4106: }
  4107: 
  4108: // pullUpPeelBBLoop
  4109: // A single BB loop with a register copy at the beginning in its
  4110: // own bundle, benefits from eliminating the extra bundle. We do
  4111: // this by predicating the register copy in the predecessor BB, and
  4112: // again in the last bundle of the loop.
  4113: bool HexagonGlobalSchedulerImpl::pullUpPeelBBLoop(MachineBasicBlock *PredBB,
  4114:                                                   MachineBasicBlock *LoopBB) {
  4115:   if (!AllowBBPeelPullUp)
  4116:     return false;
  4117:   if (!LoopBB || !PredBB)
  4118:     return false;
  4119: 
  4120:   // We consider single BB loops only. Check for it here.
  4121:   if (LoopBB->isEHPad() || LoopBB->hasAddressTaken())
  4122:     return false;
  4123:   if (LoopBB->succ_size() != 2)
  4124:     return false;
  4125:   if (LoopBB->pred_size() != 2)
  4126:     return false;
  4127:   // Make sure one of the successors and one of the predecssors is to self.
  4128:   if (!(LoopBB->isSuccessor(LoopBB) && LoopBB->isPredecessor(LoopBB)))
  4129:     return false;
  4130: 
  4131:   // Find the none self successor block. We know we only have 2 successors.
  4132:   MachineBasicBlock *SuccBB = NULL;
  4133:   for (MachineBasicBlock::succ_iterator SI = LoopBB->succ_begin(),
  4134:                                         SE = LoopBB->succ_end();
  4135:        SI != SE; ++SI)
  4136:     if (*SI != LoopBB) {
  4137:       SuccBB = *SI;
  4138:       break;
  4139:     }
  4140:   if (!SuccBB)
  4141:     return false;
  4142: 
  4143:   // Find the conditional branch and its bundle inside PredBB.
  4144:   MachineBasicBlock::iterator PredBundle;
  4145:   MachineInstr *PredCondBranch = NULL;
  4146:   PredCondBranch = findBundleAndBranch(PredBB, PredBundle);
  4147:   if (!PredCondBranch)
  4148:     return false;
  4149:   if (PredBundle == PredBB->end())
  4150:     return false;
  4151:   LLVM_DEBUG(dbgs() << "PredBB's Branch: ");
  4152:   LLVM_DEBUG(dbgs() << *PredCondBranch);
  4153: 
  4154:   // Look for leading reg copy as single bundle and make sure its live in.
  4155:   MachineBasicBlock::instr_iterator FMI = LoopBB->instr_begin();
  4156:   // Skip debug instructions.
  4157:   while (FMI->isDebugInstr())
  4158:     FMI++;
  4159: 
  4160:   MachineInstr *RegMI = &*FMI;
  4161:   if (RegMI->isBundle())
  4162:     return false;
  4163:   int TfrOpcode = RegMI->getOpcode();
  4164:   if (TfrOpcode != Hexagon::A2_tfr && TfrOpcode != Hexagon::A2_tfr)
  4165:     return false;
  4166:   if (!(RegMI->getOperand(0).isReg() && RegMI->getOperand(1).isReg()))
  4167:     return false;
  4168:   unsigned InLoopReg = RegMI->getOperand(1).getReg();
  4169:   if (!LoopBB->isLiveIn(InLoopReg))
  4170:     return false;
  4171: 
  4172:   // Create a region to pass to ResourcesAvailableInBundle.
  4173:   BasicBlockRegion PUR(BasicBlockRegion(TII, QRI, PredBB));
  4174:   PUR.addBBtoRegion(LoopBB);
  4175:   PUR.addBBtoRegion(SuccBB);
  4176: 
  4177:   // Make sure we have space in PredBB's last bundle.
  4178:   if (!ResourcesAvailableInBundle(&PUR, PredBundle))
  4179:     return false;
  4180:   SmallVector<MachineInstr *, HEXAGON_PACKET_SIZE> PredBundlePkt(
  4181:       CurrentState.HomeBundle);
  4182: 
  4183:   // Find condition to use for predicating the reg copy into PredBB.
  4184:   MachineBasicBlock *TBB = NULL, *FBB = NULL;
  4185:   SmallVector<MachineOperand, 4> Cond;
  4186:   if (QII->analyzeBranch(*PredBB, TBB, FBB, Cond, false))
  4187:     return false;
  4188:   if (Cond.empty())
  4189:     return false;
  4190: 
  4191:   // Find condition to use for predicating the reg copy at the end of LoopBB.
  4192:   MachineBasicBlock *LTBB = NULL, *LFBB = NULL;
  4193:   SmallVector<MachineOperand, 4> LCond;
  4194:   if (QII->analyzeBranch(*LoopBB, LTBB, LFBB, LCond, false))
  4195:     return false;
  4196:   if (LCond.empty())
  4197:     return false;
  4198: 
  4199:   // Move predicated reg copy to previous BB's last bundle.
  4200:   if (!TII->isPredicable(*RegMI))
```
- EN: It declares or implements routines such as findNextMBB, LLVM_DEBUG, assert, HexagonGlobalSchedulerImpl::isJumpOutOfRange, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 findNextMBB, LLVM_DEBUG, assert, HexagonGlobalSchedulerImpl::isJumpOutOfRange, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 4201-4400 / 第 4201-4400 行

```cpp
  4201:     return false;
  4202:   MachineInstr *InstrToMove =
  4203:       &*insertTempCopy(PredBB, PredBundle, RegMI, false);
  4204:   if (!canAddMIToThisPacket(InstrToMove, PredBundlePkt)) {
  4205:     PredBB->erase_instr(InstrToMove);
  4206:     return false;
  4207:   }
  4208: 
  4209:   if (!TII->PredicateInstruction(*InstrToMove, Cond)) {
  4210:     // Failed to predicate the copy reg.
  4211:     PredBB->erase_instr(InstrToMove);
  4212:     return false;
  4213:   }
  4214: 
  4215:   // Can we newify this instruction?
  4216:   unsigned DepReg = 0;
  4217:   if (NeedToNewify(InstrToMove->getIterator(), &DepReg, &*PredBundle) &&
  4218:       !isNewifiable(InstrToMove->getIterator(), DepReg, &*PredBundle)) {
  4219:     PredBB->erase_instr(InstrToMove);
  4220:     return false;
  4221:   }
  4222:   // Newify it, and then undo it if we determine we are using a .old.
  4223:   int NewOpcode = QII->getDotNewPredOp(*InstrToMove, MBPI);
  4224:   // Undo newify if we have a non .new predicated jump we are matching.
  4225:   if (!QII->isDotNewInst(*PredCondBranch))
  4226:     NewOpcode = QII->getDotOldOp(*InstrToMove);
  4227:   NewOpcode = QII->getInvertedPredicatedOpcode(NewOpcode);
  4228:   // Properly set the opcode on the new hoisted reg copy instruction.
  4229:   InstrToMove->setDesc(QII->get(NewOpcode));
  4230:   if (!incrementalAddToPacket(*InstrToMove)) {
  4231:     PredBB->erase_instr(InstrToMove);
  4232:     return false;
  4233:   }
  4234: 
  4235:   // Find the conditional branch and its bundle for LoopBB.
  4236:   MachineBasicBlock::iterator LoopBundle;
  4237:   MachineInstr *LoopCondBranch = NULL;
  4238:   LoopCondBranch = findBundleAndBranch(LoopBB, LoopBundle);
  4239:   if (!LoopCondBranch)
  4240:     return false;
  4241:   if (LoopBundle == LoopBB->end())
  4242:     return false;
  4243:   LLVM_DEBUG(dbgs() << "LoopBB's Branch: ");
  4244:   LLVM_DEBUG(dbgs() << *LoopCondBranch);
  4245: 
  4246:   // Make sure we have space in LoopBB's last bundle.
  4247:   if (!ResourcesAvailableInBundle(&PUR, LoopBundle))
  4248:     return false;
  4249:   SmallVector<MachineInstr *, HEXAGON_PACKET_SIZE> LoopBundlePkt(
  4250:       CurrentState.HomeBundle);
  4251: 
  4252:   // Move predicated reg copy to last bundle of LoopBB.
  4253:   MachineInstr *InstrToSink =
  4254:       &*insertTempCopy(LoopBB, LoopBundle, RegMI, false);
  4255:   if (!canAddMIToThisPacket(InstrToSink, LoopBundlePkt)) {
  4256:     // Get rid of previous instruction as well.
  4257:     PredBB->erase_instr(InstrToMove);
  4258:     LoopBB->erase_instr(InstrToSink);
  4259:     return false;
  4260:   }
  4261: 
  4262:   if (!TII->PredicateInstruction(*InstrToSink, LCond)) {
  4263:     // Get rid of previous instruction as well.
  4264:     PredBB->erase_instr(InstrToMove);
  4265:     LoopBB->erase_instr(InstrToSink);
  4266:     return false;
  4267:   }
  4268:   // Can we newify this instruction?
  4269:   if (NeedToNewify(InstrToSink->getIterator(), &DepReg, &*LoopBundle) &&
  4270:       !isNewifiable(InstrToSink->getIterator(), DepReg, &*LoopBundle)) {
  4271:     // Get rid of previous instruction as well.
  4272:     PredBB->erase_instr(InstrToMove);
  4273:     PredBB->erase_instr(InstrToSink);
  4274:     return false;
  4275:   }
  4276:   NewOpcode = QII->getDotNewPredOp(*InstrToSink, MBPI);
  4277:   // Undo newify if we have a non .new predicated jump we are matching.
  4278:   if (!QII->isDotNewInst(*LoopCondBranch))
  4279:     NewOpcode = QII->getDotOldOp(*InstrToSink);
  4280:   InstrToSink->setDesc(QII->get(NewOpcode));
  4281:   if (!incrementalAddToPacket(*InstrToSink)) {
  4282:     // Get rid of previous instruction as well.
  4283:     PredBB->erase_instr(InstrToMove);
  4284:     LoopBB->erase_instr(InstrToSink);
  4285:     return false;
  4286:   }
  4287: 
  4288:   // Remove old instruction.
  4289:   LoopBB->erase_instr(RegMI);
  4290:   // Set loop alignment to 32.
  4291:   LoopBB->setAlignment(llvm::Align(32));
  4292: 
  4293:   LLVM_DEBUG(dbgs() << "Peeled Single BBLoop copy\n");
  4294:   LLVM_DEBUG(dbgs() << *InstrToMove);
  4295:   LLVM_DEBUG(dbgs() << *InstrToSink);
  4296:   LLVM_DEBUG(dbgs() << *PredBB);
  4297:   LLVM_DEBUG(dbgs() << *LoopBB);
  4298:   LLVM_DEBUG(dbgs() << *SuccBB);
  4299:   LLVM_DEBUG(dbgs() << "--- BBLoop ---\n\n");
  4300:   return true;
  4301: }
  4302: 
  4303: bool HexagonGlobalSchedulerImpl::performPullUpCFG(MachineFunction &Fn) {
  4304:   const Function &F = Fn.getFunction();
  4305:   // Check for single-block functions and skip them.
  4306:   if (std::next(F.begin()) == F.end())
  4307:     return false;
  4308:   bool Changed = false;
  4309:   LLVM_DEBUG(dbgs() << "****** PullUpCFG **************\n");
  4310: 
  4311:   // Loop over all basic blocks, asking if 3 consecutive blocks are
  4312:   // the jump opportunity.
  4313:   MachineBasicBlock *PrevBlock = NULL;
  4314:   MachineBasicBlock *JumpBlock = NULL;
  4315:   for (MachineFunction::iterator MBB = Fn.begin(), MBBe = Fn.end(); MBB != MBBe;
  4316:        ++MBB) {
  4317:     MachineBasicBlock *FallBlock = &*MBB;
  4318:     if (PrevBlock && JumpBlock) {
  4319:       Changed |= pullUpPeelBBLoop(PrevBlock, JumpBlock);
  4320:     }
  4321:     PrevBlock = JumpBlock;
  4322:     JumpBlock = FallBlock;
  4323:   }
  4324:   return Changed;
  4325: }
  4326: 
  4327: void HexagonGlobalSchedulerImpl::GenUseDefChain(MachineFunction &Fn) {
  4328:   std::vector<unsigned> Defs;
  4329:   std::vector<unsigned> Uses;
  4330:   for (MachineFunction::iterator MBBIter = Fn.begin(); MBBIter != Fn.end();
  4331:        ++MBBIter) {
  4332:     for (MachineBasicBlock::instr_iterator MIter = MBBIter->instr_begin();
  4333:          MIter != MBBIter->instr_end(); ++MIter) {
  4334:       if (MIter->isBundle() || MIter->isDebugInstr())
  4335:         continue;
  4336:       LLVM_DEBUG(dbgs() << "\n\nInserted Ins:"; MIter->dump());
  4337:       MIUseDefSet(&*MIter, Defs, Uses);
  4338:       LLVM_DEBUG(dbgs() << "\n\tDefs:";
  4339:                  for (unsigned i = 0; i < Defs.size(); ++i) dbgs()
  4340:                  << printReg(Defs[i], QRI) << ",");
  4341:       LLVM_DEBUG(dbgs() << "\n\tUses:";
  4342:                  for (unsigned i = 0; i < Uses.size(); ++i) dbgs()
  4343:                  << printReg(Uses[i], QRI) << ",");
  4344:       MIDefSet[&*MIter] = Defs;
  4345:       MIUseSet[&*MIter] = Uses;
  4346:     }
  4347:   }
  4348: }
  4349: 
  4350: // optimizeBranching -
  4351: // 1. A conditional-jump transfers control to a BB with
  4352: // jump as the only instruction.
  4353: // if(p0) jump t1
  4354: // // ...
  4355: // t1: jump t2
  4356: // 2. When a BB with a single conditional jump, jumps to succ-of-succ and
  4357: // falls-through BB with only jump instruction.
  4358: // { if(p0) jump t1 }
  4359: // { jump t2 }
  4360: // t1: { ... }
  4361: MachineBasicBlock *HexagonGlobalSchedulerImpl::optimizeBranches(
  4362:     MachineBasicBlock *MBB, MachineBasicBlock *TBB, MachineInstr *FirstTerm,
  4363:     MachineBasicBlock *FBB) {
  4364:   LLVM_DEBUG(dbgs() << "\n\t\t[optimizeBranching]\n");
  4365:   if ((TBB == MBB) || (FBB == MBB))
  4366:     LLVM_DEBUG(dbgs() << "Cannot deal with loops in BB#" << MBB->getNumber(););
  4367: 
  4368:   // LLVM_DEBUG(dbgs() << "\n\t\tTBBMIb:"; MII->dump(););
  4369:   // { if(p) jump t1; }
  4370:   // t1: { jump t2; }
  4371:   // --> { if(p) jump t2
  4372:   // remove t1: { jump t2; }, if it's address is not taken/not a landing pad.
  4373:   if (QII->nonDbgBBSize(TBB) == 1) {
  4374:     MachineInstr *TBBMIb = &*TBB->getFirstNonDebugInstr();
  4375:     if (TBBMIb->getOpcode() == Hexagon::J2_jump &&
  4376:         TBBMIb->getOperand(0).isMBB()) {
  4377:       MachineBasicBlock *NewTarget = TBBMIb->getOperand(0).getMBB();
  4378:       if (TBB == NewTarget) // Infinite loop.
  4379:         return NULL;
  4380: 
  4381:       LLVM_DEBUG(dbgs() << "\nSuboptimal branching in TBB");
  4382:       // Check if the jump in the last instruction is within range.
  4383:       int64_t InstOffset =
  4384:           BlockToInstOffset.find(MBB)->second + QII->nonDbgBBSize(MBB) * 4;
  4385:       unsigned Distance = (unsigned)std::abs(
  4386:           InstOffset - BlockToInstOffset.find(NewTarget)->second);
  4387:       if (!QII->isJumpWithinBranchRange(*FirstTerm, Distance)) {
  4388:         LLVM_DEBUG(dbgs() << "\nUnconditional jump target:" << Distance
  4389:                           << " out of range.");
  4390:         return NULL;
  4391:       }
  4392:       // We need to make sure that the TBB is _not_ also a target for another
  4393:       // branch. This is suboptimal since theoretically we can update both
  4394:       // branches.
  4395:       if (!TBB->hasAddressTaken() && !TBB->isEHPad() && TBB->pred_size() == 1) {
  4396:         updatePredecessors(*TBB, NewTarget);
  4397:         // TBB has only one successor since only one J2_jump instr.
  4398:         TBB->removeSuccessor(TBB->succ_begin());
  4399:         TBBMIb->removeFromParent();
  4400:         if (!TBB->empty()) {
```
- EN: It declares or implements routines such as insertTempCopy, erase_instr, getDotNewPredOp, getInvertedPredicatedOpcode, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 insertTempCopy, erase_instr, getDotNewPredOp, getInvertedPredicatedOpcode, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 4401-4600 / 第 4401-4600 行

```cpp
  4401:           // There are only debug instructions in TBB now. Move them to
  4402:           // the beginning of NewTarget.
  4403:           NewTarget->splice(NewTarget->getFirstNonPHI(), TBB, TBB->begin(),
  4404:                             TBB->end());
  4405:         }
  4406:         return TBB;
  4407:       } else {
  4408:         MBB->ReplaceUsesOfBlockWith(TBB, NewTarget);
  4409:         return NULL;
  4410:       }
  4411:     }
  4412:   }
  4413:   // { if(p) jump t1; } may contain more instructions
  4414:   // { jump t2; } --only one instruction
  4415:   // t1: {...}
  4416:   // TBB is layout successor of FBB, then we can change the branch target
  4417:   // for conditional jump and invert the predicate to remove jump t2.
  4418:   // { if(!p) jump t2; }
  4419:   // t1: {...}
  4420:   if (QII->nonDbgBBSize(FBB) == 1) {
  4421:     MachineInstr *FBBMIb = &*FBB->getFirstNonDebugInstr();
  4422:     if (FBBMIb->getOpcode() == Hexagon::J2_jump &&
  4423:         FBBMIb->getOperand(0).isMBB()) {
  4424:       MachineBasicBlock *NewTarget = FBBMIb->getOperand(0).getMBB();
  4425:       if (FBB->hasAddressTaken() || FBB->isEHPad() ||
  4426:           !FBB->isLayoutSuccessor(TBB) || (FBB == NewTarget /*Infinite loop*/))
  4427:         return NULL;
  4428: 
  4429:       LLVM_DEBUG(dbgs() << "\nSuboptimal branching in FBB");
  4430:       // Check if the jump in the last instruction is within range.
  4431:       int64_t InstOffset =
  4432:           BlockToInstOffset.find(MBB)->second + QII->nonDbgBBSize(MBB) * 4;
  4433:       unsigned Distance = (unsigned)std::abs(
  4434:           InstOffset - BlockToInstOffset.find(NewTarget)->second);
  4435:       if (!QII->isJumpWithinBranchRange(*FirstTerm, Distance)) {
  4436:         LLVM_DEBUG(dbgs() << "\nUnconditional jump target:" << Distance
  4437:                           << " out of range.");
  4438:         return NULL;
  4439:       }
  4440:       if (!QII->invertAndChangeJumpTarget(*FirstTerm, NewTarget))
  4441:         return NULL;
  4442:       LLVM_DEBUG(dbgs() << "\nNew instruction:"; FirstTerm->dump(););
  4443:       updatePredecessors(*FBB, NewTarget);
  4444:       // Only one successor remains for FBB
  4445:       FBB->removeSuccessor(FBB->succ_begin());
  4446:       FBBMIb->removeFromParent();
  4447:       return FBB;
  4448:     }
  4449:   }
  4450:   return NULL;
  4451: }
  4452: 
  4453: // performExposedOptimizations -
  4454: // look for optimization opportunities after pullup.
  4455: // e.g. jump to adjacent targets
  4456: bool HexagonGlobalSchedulerImpl::performExposedOptimizations(
  4457:     MachineFunction &Fn) {
  4458:   // Check for single-block functions and skip them.
  4459:   if (std::next(Fn.getFunction().begin()) == Fn.getFunction().end())
  4460:     return true;
  4461:   LLVM_DEBUG(dbgs() << "\n\t\t[performExposedOptimizations]\n");
  4462:   // Erasing the empty basic blocks formed during pullup.
  4463:   std::vector<MachineBasicBlock *>::iterator ebb = EmptyBBs.begin();
  4464:   while (ebb != EmptyBBs.end()) {
  4465:     assert(IsEmptyBlock(*ebb) && "Pullup inserted packets into an empty BB");
  4466:     LLVM_DEBUG(dbgs() << "Removing BB(" << (*ebb)->getNumber()
  4467:                       << ") from parent.\n");
  4468:     (*ebb)->eraseFromParent();
  4469:     ++ebb;
  4470:   }
  4471:   MachineBasicBlock *TBB = NULL, *FBB = NULL;
  4472:   MachineInstr *FirstTerm = NULL, *SecondTerm = NULL;
  4473: 
  4474:   SmallVector<MachineBasicBlock *, 4> Erase;
  4475: 
  4476:   for (MachineBasicBlock &MBB : Fn) {
  4477:     if (MBB.succ_size() > 2 ||
  4478:         AnalyzeBBBranches(&MBB, TBB, FirstTerm, FBB, SecondTerm)) {
  4479:       LLVM_DEBUG(dbgs() << "\nAnalyzeBBBranches failed in BB#"
  4480:                         << MBB.getNumber() << "\n";);
  4481:       continue;
  4482:     }
  4483:     if (FirstTerm && QII->isCompoundBranchInstr(*FirstTerm))
  4484:       continue;
  4485:     if (TBB && FirstTerm &&
  4486:         removeRedundantBranches(&MBB, TBB, FirstTerm, FBB, SecondTerm)) {
  4487:       LLVM_DEBUG(dbgs() << "\nRemoved redundant branches in BB#"
  4488:                         << MBB.getNumber(););
  4489:       continue;
  4490:     }
  4491:     if (FirstTerm && SecondTerm &&
  4492:         optimizeDualJumps(&MBB, TBB, FirstTerm, FBB, SecondTerm)) {
  4493:       LLVM_DEBUG(dbgs() << "\nRemoved dual jumps in in BB#"
  4494:                         << MBB.getNumber(););
  4495:       continue;
  4496:     }
  4497:     if (TBB && FBB && FirstTerm && !SecondTerm) {
  4498:       MachineBasicBlock *MBBToErase =
  4499:           optimizeBranches(&MBB, TBB, FirstTerm, FBB);
  4500:       if (MBBToErase) {
  4501:         assert(IsEmptyBlock(MBBToErase) && "Erasing non-empty BB");
  4502:         Erase.push_back(MBBToErase);
  4503:         LLVM_DEBUG(dbgs() << "\nOptimized jump from BB#" << MBB.getNumber());
  4504:       }
  4505:     }
  4506:   }
  4507:   for (MachineBasicBlock *MBB : Erase)
  4508:     MBB->eraseFromParent();
  4509: 
  4510:   return false;
  4511: }
  4512: 
  4513: // 1. Remove jump to the layout successor.
  4514: // 2. Remove multiple (dual) jump to the same target.
  4515: bool HexagonGlobalSchedulerImpl::removeRedundantBranches(
  4516:     MachineBasicBlock *MBB, MachineBasicBlock *TBB, MachineInstr *FirstTerm,
  4517:     MachineBasicBlock *FBB, MachineInstr *SecondTerm) {
  4518:   bool Analyzed = false;
  4519:   LLVM_DEBUG(dbgs() << "\n\t\t[removeRedundantBranches]\n");
  4520:   MachineInstr *Head = NULL, *ToErase = NULL;
  4521:   if (!FBB && (FirstTerm->getOpcode() == Hexagon::J2_jump) &&
  4522:       MBB->isLayoutSuccessor(TBB)) {
  4523:     // Jmp layout_succ_basic_block <-- Remove
  4524:     LLVM_DEBUG(
  4525:         dbgs() << "\nRemoving Uncond. jump to the layout successor in BB#"
  4526:                << MBB->getNumber());
  4527:     ToErase = FirstTerm;
  4528:   } else if (SecondTerm && (TBB == FBB) &&
  4529:              (SecondTerm->getOpcode() == Hexagon::J2_jump)) {
  4530:     // If both branching instructions in same packet or are consecutive.
  4531:     // Jmp_c t1 <-- Remove
  4532:     // Jmp t1
  4533:     // @Note: If they are in different packets or if they are separated
  4534:     // by packet(s), this opt. cannot be done.
  4535:     MachineBasicBlock::instr_iterator FirstTermIter = FirstTerm->getIterator();
  4536:     MachineBasicBlock::instr_iterator SecondTermIter =
  4537:         SecondTerm->getIterator();
  4538:     if (++FirstTermIter == SecondTermIter) {
  4539:       LLVM_DEBUG(dbgs() << "\nRemoving multiple branching to same target in BB#"
  4540:                         << MBB->getNumber());
  4541:       // TODO: This might make the `p' register assignment instruction dead.
  4542:       // and can be removed.
  4543:       ToErase = FirstTerm;
  4544:     }
  4545:   } else if (SecondTerm && (SecondTerm->getOpcode() == Hexagon::J2_jump) &&
  4546:              FBB && MBB->isLayoutSuccessor(FBB)) {
  4547:     // Jmp_c t1
  4548:     // Jmp layout_succ_basic_block <-- Remove
  4549:     LLVM_DEBUG(dbgs() << "\nRemoving fall through branch in BB#"
  4550:                       << MBB->getNumber());
  4551:     ToErase = SecondTerm;
  4552:   } else if (SecondTerm && QII->PredOpcodeHasJMP_c(SecondTerm->getOpcode()) &&
  4553:              MBB->isLayoutSuccessor(getBranchDestination(SecondTerm))) {
  4554:     // Jmp_c t1
  4555:     // Jmp_c layout_succ_basic_block <-- Remove
  4556:     // In this case AnalyzeBBBranches might assign FBB to some other BB.
  4557:     // So using the jump target of SecondTerm to check.
  4558:     LLVM_DEBUG(dbgs() << "\nRemoving Cond. jump to the layout successor in BB#"
  4559:                       << MBB->getNumber());
  4560:     ToErase = SecondTerm;
  4561:   }
  4562:   // Remove the instruction from the BB
  4563:   if (ToErase) {
  4564:     if (ToErase->isBundled()) {
  4565:       Head = &*getBundleStart(ToErase->getIterator());
  4566:       ToErase->eraseFromBundle();
  4567:       UpdateBundle(Head);
  4568:     } else
  4569:       ToErase->eraseFromParent();
  4570:     Analyzed = true;
  4571:   }
  4572:   return Analyzed;
  4573: }
  4574: 
  4575: // ----- convert
  4576: // p = <expr>
  4577: // if(p) jump layout_succ_basic_block
  4578: // jump t
  4579: // ----- to
  4580: // p = <expr>
  4581: // if(!p) jump t
  4582: // for now only looking at the dual jump
  4583: bool HexagonGlobalSchedulerImpl::optimizeDualJumps(MachineBasicBlock *MBB,
  4584:                                                    MachineBasicBlock *TBB,
  4585:                                                    MachineInstr *FirstTerm,
  4586:                                                    MachineBasicBlock *FBB,
  4587:                                                    MachineInstr *SecondTerm) {
  4588:   LLVM_DEBUG(dbgs() << "\n******* optimizeDualJumps *******");
  4589: 
  4590:   bool Analyzed = false;
  4591: 
  4592:   if (QII->PredOpcodeHasJMP_c(FirstTerm->getOpcode()) &&
  4593:       (SecondTerm->getOpcode() == Hexagon::J2_jump)) {
  4594: 
  4595:     if (TBB == FBB) {
  4596:       LLVM_DEBUG(dbgs() << "\nBoth successors are the same.");
  4597:       return Analyzed;
  4598:     }
  4599: 
  4600:     // Do not optimize for dual jumps if this MBB
```
- EN: It declares or implements routines such as splice, ReplaceUsesOfBlockWith, getFirstNonDebugInstr, getOperand, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 splice, ReplaceUsesOfBlockWith, getFirstNonDebugInstr, getOperand, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 4601-4800 / 第 4601-4800 行

```cpp
  4601:     // contains a speculatively pulled-up instruction.
  4602:     // A speculated instruction is more likely to be at the end of MBB.
  4603:     MachineBasicBlock::reverse_instr_iterator SII = MBB->instr_rbegin();
  4604:     while (SII != MBB->instr_rend()) {
  4605:       MachineInstr *SI = &*SII;
  4606:       std::map<MachineInstr *, MachineBasicBlock *>::iterator MIMoved;
  4607:       MIMoved = SpeculatedIns.find(SI);
  4608:       if ((MIMoved != SpeculatedIns.end()) &&
  4609:           (MIMoved->second != SI->getParent())) {
  4610:         return Analyzed;
  4611:       }
  4612:       ++SII;
  4613:     }
  4614: 
  4615:     LLVM_DEBUG(dbgs() << "\nCandidate for jump optimization in BB("
  4616:                       << MBB->getNumber() << ").\n";);
  4617: 
  4618:     // Predicated jump to layout successor followed by an unconditional jump.
  4619:     if (MBB->isLayoutSuccessor(TBB)) {
  4620: 
  4621:       // Check if the jump in the last instruction is within range.
  4622:       int64_t InstOffset =
  4623:           BlockToInstOffset.find(&*MBB)->second + QII->nonDbgBBSize(MBB) * 4;
  4624:       unsigned Distance =
  4625:           (unsigned)std::abs(InstOffset - BlockToInstOffset.find(FBB)->second) +
  4626:           SafetyBuffer;
  4627:       if (!QII->isJumpWithinBranchRange(*FirstTerm, Distance)) {
  4628:         LLVM_DEBUG(dbgs() << "\nUnconditional jump target:" << Distance
  4629:                           << " out of range.");
  4630:         return Analyzed;
  4631:       }
  4632: 
  4633:       // modify the second last -predicated- instruction (sense and target)
  4634:       LLVM_DEBUG(dbgs() << "\nFirst Instr:" << *FirstTerm;);
  4635:       LLVM_DEBUG(dbgs() << "\nSecond Instr:" << *SecondTerm;);
  4636:       LLVM_DEBUG(dbgs() << "\nOld Succ BB(" << TBB->getNumber() << ").";);
  4637: 
  4638:       QII->invertAndChangeJumpTarget(*FirstTerm, FBB);
  4639: 
  4640:       LLVM_DEBUG(dbgs() << "\nNew First Instruction:" << *FirstTerm;);
  4641: 
  4642:       // unbundle if there is only one instruction left
  4643:       MachineInstr *SecondHead, *FirstHead;
  4644:       FirstHead = FirstTerm->isBundled()
  4645:                       ? &*getBundleStart(FirstTerm->getIterator())
  4646:                       : nullptr;
  4647:       SecondHead = SecondTerm->isBundled()
  4648:                        ? &*getBundleStart(SecondTerm->getIterator())
  4649:                        : nullptr;
  4650: 
  4651:       // 1. Both unbundled, 2. FirstTerm inside bundle, second outside.
  4652:       if (!SecondHead)
  4653:         SecondTerm->eraseFromParent();
  4654:       else if (!FirstHead) {
  4655:         // 3. FirstHead outside, SecondHead inside.
  4656:         SecondTerm->eraseFromBundle();
  4657:         UpdateBundle(SecondHead);
  4658:       } else if (FirstHead == SecondHead) {
  4659:         // 4. Both are in the same bundle
  4660:         assert((FirstHead && SecondHead) && "Unbundled Instruction");
  4661:         SecondTerm->eraseFromBundle();
  4662:         if (SecondHead->getBundleSize() < 2)
  4663:           UpdateBundle(SecondHead);
  4664:       } else {
  4665:         // 5. Both are in different bundles
  4666:         SecondTerm->eraseFromBundle();
  4667:         UpdateBundle(SecondHead);
  4668:       }
  4669:       Analyzed = true;
  4670:     }
  4671:   }
  4672:   return Analyzed;
  4673: }
  4674: 
  4675: /// Are there any resources left in this bundle?
  4676: bool HexagonGlobalSchedulerImpl::ResourcesAvailableInBundle(
  4677:     BasicBlockRegion *CurrentRegion,
  4678:     MachineBasicBlock::iterator &TargetPacket) {
  4679:   MachineBasicBlock::instr_iterator MII = TargetPacket.getInstrIterator();
  4680: 
  4681:   // If this is a single instruction, form new packet around it.
  4682:   if (!TargetPacket->isBundle()) {
  4683:     if (ignoreInstruction(&*MII) || isSoloInstruction(*MII))
  4684:       return false;
  4685: 
  4686:     // Before we begin, we need to make sure that we do not
  4687:     // look at an unconditional jump outside the current region.
  4688:     if (MII->isBranch() && !isBranchWithinRegion(CurrentRegion, &*MII))
  4689:       return false;
  4690: 
  4691:     // Build up state for this new packet.
  4692:     // Note, we cannot create a bundle header for it,
  4693:     // so this "bundle" only exist in DFA state, and not in code.
  4694:     initPacketizerState();
  4695:     ResourceTracker->clearResources();
  4696:     CurrentState.addHomeLocation(MII);
  4697:     return incrementalAddToPacket(*MII);
  4698:   }
  4699: 
  4700:   MachineBasicBlock::instr_iterator End = MII->getParent()->instr_end();
  4701: 
  4702:   // Build up state for this packet.
  4703:   initPacketizerState();
  4704:   ResourceTracker->clearResources();
  4705:   CurrentState.addHomeLocation(MII);
  4706: 
  4707:   for (++MII; MII != End && MII->isInsideBundle(); ++MII) {
  4708:     if (MII->getOpcode() == TargetOpcode::DBG_VALUE ||
  4709:         MII->getOpcode() == TargetOpcode::IMPLICIT_DEF ||
  4710:         MII->getOpcode() == TargetOpcode::CFI_INSTRUCTION || MII->isEHLabel())
  4711:       continue;
  4712: 
  4713:     // Before we begin, we need to make sure that we do not
  4714:     // look at an unconditional jump outside the current region.
  4715:     // TODO: See if we can profit from handling this kind of cases:
  4716:     // B#15: derived from LLVM BB %if.then22
  4717:     // Predecessors according to CFG: BB#13
  4718:     // BUNDLE %PC<imp-def>, %P2<imp-use,kill>
  4719:     //   * J2_jumpf %P2<kill,internal>, <BB#17>, %PC<imp-def>; flags:
  4720:     //   * J2_jump <BB#18>, %PC<imp-def>; flags:
  4721:     // Successors according to CFG: BB#18(62) BB#17(62)
  4722:     // Curently we do not allow them.
  4723:     if (MII->isBranch() && !isBranchWithinRegion(CurrentRegion, &*MII))
  4724:       return false;
  4725: 
  4726:     if (!incrementalAddToPacket(*MII))
  4727:       return false;
  4728:   }
  4729:   return ResourceTracker->canReserveResources(*Nop);
  4730: }
  4731: 
  4732: /// Symmetrical. See if these two instructions are fit for compound pair.
  4733: bool HexagonGlobalSchedulerImpl::isCompoundPair(MachineInstr *MIa,
  4734:                                                 MachineInstr *MIb) {
  4735:   enum HexagonII::CompoundGroup MIaG = QII->getCompoundCandidateGroup(*MIa),
  4736:                                 MIbG = QII->getCompoundCandidateGroup(*MIb);
  4737:   // We have two candidates - check that this is the same register
  4738:   // we are talking about.
  4739:   unsigned Opcb = MIb->getOpcode();
  4740:   if (MIaG == HexagonII::HCG_C && MIbG == HexagonII::HCG_A &&
  4741:       (Opcb == Hexagon::A2_tfr || Opcb == Hexagon::A2_tfrsi))
  4742:     return true;
  4743:   unsigned Opca = MIa->getOpcode();
  4744:   if (MIbG == HexagonII::HCG_C && MIaG == HexagonII::HCG_A &&
  4745:       (Opca == Hexagon::A2_tfr || Opca == Hexagon::A2_tfrsi))
  4746:     return true;
  4747:   return (((MIaG == HexagonII::HCG_A && MIbG == HexagonII::HCG_B) ||
  4748:            (MIbG == HexagonII::HCG_A && MIaG == HexagonII::HCG_B)) &&
  4749:           (MIa->getOperand(0).getReg() == MIb->getOperand(0).getReg()));
  4750: }
  4751: 
  4752: // This is a weird situation when BB conditionally branches + falls through
  4753: // to layout successor. \ref bug17792
  4754: inline bool HexagonGlobalSchedulerImpl::multipleBranchesFromToBB(
  4755:     MachineBasicBlock *BB) const {
  4756:   if (BB->succ_size() != 1)
  4757:     return false;
  4758:   SmallVector<MachineInstr *, 2> Jumpers = QII->getBranchingInstrs(*BB);
  4759:   return ((Jumpers.size() == 1) && !Jumpers[0]->isUnconditionalBranch());
  4760: }
  4761: 
  4762: /// Gather a worklist of MaxCandidates pull-up candidates.
  4763: /// Compute relative cost.
  4764: bool HexagonGlobalSchedulerImpl::findPullUpCandidates(
  4765:     MachineBasicBlock::iterator &WorkPoint,
  4766:     MachineBasicBlock::iterator &FromHere,
  4767:     std::vector<MachineInstr *> &backtrack, unsigned MaxCandidates = 1) {
  4768: 
  4769:   const HexagonInstrInfo *QII = (const HexagonInstrInfo *)TII;
  4770:   MachineBasicBlock *FromThisBB = FromHere->getParent();
  4771:   bool MovingDependentOp = false;
  4772:   signed CostBenefit = 0;
  4773: 
  4774:   // Do not collect more than that many candidates.
  4775:   if (CurrentState.haveCandidates() >= MaxCandidates)
  4776:     return false;
  4777: 
  4778:   LLVM_DEBUG(dbgs() << "\n\tTry from BB(" << FromThisBB->getNumber() << "):\n";
  4779:              DumpPacket(FromHere.getInstrIterator()));
  4780: 
  4781:   if (FromHere->isBundle()) {
  4782:     MachineBasicBlock::instr_iterator MII = FromHere.getInstrIterator();
  4783:     for (++MII; MII != FromThisBB->instr_end() && MII->isInsideBundle();
  4784:          ++MII) {
  4785:       if (MII->isDebugInstr())
  4786:         continue;
  4787:       LLVM_DEBUG(dbgs() << "\tCandidate from BB("
  4788:                         << MII->getParent()->getNumber() << "): ";
  4789:                  MII->dump());
  4790: 
  4791:       // See if this instruction could be moved.
  4792:       if (!canThisMIBeMoved(&*MII, WorkPoint, MovingDependentOp, CostBenefit))
  4793:         continue;
  4794: 
  4795:       MachineBasicBlock::instr_iterator InstrToMove = MII;
  4796:       if (canAddMIToThisPacket(&*InstrToMove, CurrentState.HomeBundle)) {
  4797:         CostBenefit -= (backtrack.size() * 4);
  4798:         // Prefer instructions in empty packets.
  4799:         CostBenefit += (PacketSize - nonDbgBundleSize(FromHere)) * 2;
  4800:         // Prefer Compares.
```
- EN: It declares types such as HexagonII::CompoundGroup, which carry the state or API of this component. It declares or implements routines such as instr_rbegin, find, LLVM_DEBUG, invertAndChangeJumpTarget, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 HexagonII::CompoundGroup 等类型，用来承载该组件的状态或接口。 这里声明或实现了 instr_rbegin, find, LLVM_DEBUG, invertAndChangeJumpTarget, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 4801-5000 / 第 4801-5000 行

```cpp
  4801:         if (MII->isCompare())
  4802:           CostBenefit += 10;
  4803:         // Check duplex conditions;
  4804:         for (unsigned i = 0; i < CurrentState.HomeBundle.size(); i++) {
  4805:           if (QII->isDuplexPair(*CurrentState.HomeBundle[i], *MII)) {
  4806:             LLVM_DEBUG(dbgs() << "\tGot real Duplex (bundle).\n");
  4807:             CostBenefit += 20;
  4808:           }
  4809:           if (isCompoundPair(CurrentState.HomeBundle[i], &*MII)) {
  4810:             LLVM_DEBUG(dbgs() << "\tGot compound (bundle).\n");
  4811:             CostBenefit += 40;
  4812:           }
  4813:         }
  4814:         // Create a record for this location.
  4815:         CurrentState.addPullUpCandidate(InstrToMove, WorkPoint, backtrack,
  4816:                                         MovingDependentOp, CostBenefit);
  4817:       } else
  4818:         LLVM_DEBUG(dbgs() << "\tNo resources in the target packet.\n");
  4819:     }
  4820:   }
  4821:   // This is a standalone instruction.
  4822:   // First see if this MI can even be moved. Cost model for a single instruction
  4823:   // should be rather different from moving something out of a bundle.
  4824:   else if (canThisMIBeMoved(&*FromHere, WorkPoint, MovingDependentOp,
  4825:                             CostBenefit)) {
  4826:     MachineBasicBlock::instr_iterator InstrToMove = FromHere.getInstrIterator();
  4827:     if (canAddMIToThisPacket(&*InstrToMove, CurrentState.HomeBundle)) {
  4828:       CostBenefit -= (backtrack.size() * 4);
  4829:       // Prefer Compares.
  4830:       if (InstrToMove->isCompare())
  4831:         CostBenefit += 10;
  4832:       // It is better to pull a single instruction in to a bundle - save
  4833:       // a cycle immediately.
  4834:       CostBenefit += 10;
  4835:       // Search for duplex match.
  4836:       for (unsigned i = 0; i < CurrentState.HomeBundle.size(); i++) {
  4837:         if (QII->isDuplexPair(*CurrentState.HomeBundle[i], *InstrToMove)) {
  4838:           LLVM_DEBUG(dbgs() << "\tGot real Duplex (single).\n");
  4839:           CostBenefit += 30;
  4840:         }
  4841:         if (isCompoundPair(CurrentState.HomeBundle[i], &*InstrToMove)) {
  4842:           LLVM_DEBUG(dbgs() << "\tGot compound (single).\n");
  4843:           CostBenefit += 50;
  4844:         }
  4845:       }
  4846:       // Create a record for this location.
  4847:       CurrentState.addPullUpCandidate(InstrToMove, WorkPoint, backtrack,
  4848:                                       MovingDependentOp, CostBenefit);
  4849:     } else
  4850:       LLVM_DEBUG(dbgs() << "\tNo resources for single in the target packet.\n");
  4851:   }
  4852:   return true;
  4853: }
  4854: 
  4855: /// Try to move a candidate MI.
  4856: /// The move can destroy all iterator system, so we have to drag them
  4857: /// around to keep them up to date.
  4858: bool HexagonGlobalSchedulerImpl::tryMultipleInstructions(
  4859:     MachineBasicBlock::iterator &RetVal, /* output parameter */
  4860:     std::vector<BasicBlockRegion *>::iterator &CurrentRegion,
  4861:     MachineBasicBlock::iterator &NextMI,
  4862:     MachineBasicBlock::iterator &ToThisBBEnd,
  4863:     MachineBasicBlock::iterator &FromThisBBEnd, bool PathInRegion) {
  4864: 
  4865:   MachineBasicBlock::instr_iterator MII;
  4866:   MachineBasicBlock::iterator WorkPoint;
  4867:   bool MovingDependentOp = false;
  4868:   std::vector<MachineInstr *> backtrack;
  4869: 
  4870:   LLVM_DEBUG(dbgs() << "\n\tTry Multiple candidates: \n");
  4871: 
  4872:   std::sort(CurrentState.PullUpCandidates.begin(),
  4873:             CurrentState.PullUpCandidates.end(), PullUpCandidateSorter());
  4874:   LLVM_DEBUG(CurrentState.dump());
  4875:   // Iterate through candidates in sorted order.
  4876:   for (SmallVector<PullUpCandidate *, 4>::iterator
  4877:            I = CurrentState.PullUpCandidates.begin(),
  4878:            E = CurrentState.PullUpCandidates.end();
  4879:        I != E; ++I) {
  4880:     (*I)->populate(MII, WorkPoint, backtrack, MovingDependentOp);
  4881: 
  4882:     MachineBasicBlock *FromThisBB = MII->getParent();
  4883:     MachineBasicBlock *ToThisBB = WorkPoint->getParent();
  4884: 
  4885:     LLVM_DEBUG(dbgs() << "\n\tCandidate: "; MII->dump());
  4886:     LLVM_DEBUG(dbgs() << "\tDependent(" << MovingDependentOp << ") FromBB("
  4887:                       << FromThisBB->getNumber() << ") ToBB("
  4888:                       << ToThisBB->getNumber() << ") to this packet:\n";
  4889:                DumpPacket(WorkPoint.getInstrIterator()));
  4890: 
  4891:     MachineBasicBlock::instr_iterator FromHereII = MII;
  4892:     if (MII->isInsideBundle()) {
  4893:       while (!FromHereII->isBundle())
  4894:         --FromHereII;
  4895:       LLVM_DEBUG(dbgs() << "\tFrom here:\n"; DumpPacket(FromHereII));
  4896: 
  4897:       MachineBasicBlock::iterator FromHere(FromHereII);
  4898:       // We have instruction that could be moved from its current position.
  4899:       if (MoveMItoBundle(*CurrentRegion, MII, NextMI, WorkPoint, FromHere,
  4900:                          backtrack, MovingDependentOp, PathInRegion)) {
  4901:         // If BB from which we pull is now empty, move on.
  4902:         if (IsEmptyBlock(FromThisBB)) {
  4903:           LLVM_DEBUG(dbgs() << "\n\tExhosted BB (bundle).\n");
  4904:           return false;
  4905:         }
  4906:         FromThisBBEnd = FromThisBB->end();
  4907:         ToThisBBEnd = ToThisBB->end();
  4908: 
  4909:         LLVM_DEBUG(dbgs() << "\n\tAfter updates(bundle to bundle):\n");
  4910:         LLVM_DEBUG(dbgs() << "\t\tWorkPoint: ";
  4911:                    DumpPacket(WorkPoint.getInstrIterator()));
  4912: 
  4913:         // We should not increment current position,
  4914:         // but rather try one more time to pull from the same bundle.
  4915:         RetVal = WorkPoint;
  4916:         return true;
  4917:       } else
  4918:         LLVM_DEBUG(dbgs() << "\tCould not move packetized instr.\n");
  4919:     } else {
  4920:       MachineBasicBlock::iterator FromHere(FromHereII);
  4921:       if (MoveMItoBundle(*CurrentRegion, MII, NextMI, WorkPoint, FromHere,
  4922:                          backtrack, MovingDependentOp, PathInRegion)) {
  4923: 
  4924:         // If BB from which we pull is now empty, move on.
  4925:         if (IsEmptyBlock(FromThisBB)) {
  4926:           LLVM_DEBUG(dbgs() << "\n\tExhosted BB (single).\n");
  4927:           return false;
  4928:         }
  4929:         FromThisBBEnd = FromThisBB->end();
  4930:         ToThisBBEnd = ToThisBB->end();
  4931: 
  4932:         LLVM_DEBUG(dbgs() << "\tAfter updates (single to bundle):\n");
  4933:         LLVM_DEBUG(dbgs() << "\t\tWorkPoint: ";
  4934:                    DumpPacket(WorkPoint.getInstrIterator()));
  4935:         // We should not increment current position,
  4936:         // but rather try one more time to pull from the same bundle.
  4937:         RetVal = WorkPoint;
  4938:         return true;
  4939:       } else
  4940:         LLVM_DEBUG(dbgs() << "\tCould not move single.\n");
  4941:     }
  4942:   }
  4943:   LLVM_DEBUG(dbgs() << "\tNot a single candidate fit.\n");
  4944:   return false;
  4945: }
  4946: 
  4947: /// Main function. Iterate all current regions one at a time,
  4948: /// and look for pull-up opportunities.
  4949: /// Pseudo sequence:
  4950: /// - for all bundles and single instructions in region:
  4951: /// - see if resources are available (in the same cycle) - this is HOME.
  4952: /// - Starting from next BB in region, find an instruction that could be:
  4953: ///   - removed from its current location
  4954: ///   - added to underutilized bundle (including bundles with only one op)
  4955: /// - If so, trace path back to HOME and check that candidate could be
  4956: ///   reordered with all the intermediate instructions.
  4957: bool HexagonGlobalSchedulerImpl::performPullUp() {
  4958:   std::vector<MachineInstr *> backtrack;
  4959:   MachineBasicBlock::iterator FromHere;
  4960:   MachineBasicBlock::iterator FromThisBBEnd;
  4961: 
  4962:   LLVM_DEBUG(dbgs() << "****** PullUpRegions ***********\n");
  4963:   // For all regions...
  4964:   for (std::vector<BasicBlockRegion *>::iterator
  4965:            CurrentRegion = PullUpRegions.begin(),
  4966:            E = PullUpRegions.end();
  4967:        CurrentRegion != E; ++CurrentRegion) {
  4968: 
  4969:     LLVM_DEBUG(dbgs() << "\n\nRegion with(" << (*CurrentRegion)->size()
  4970:                       << ")BBs\n");
  4971: 
  4972:     if (!EnableLocalPullUp && (*CurrentRegion)->size() < 2)
  4973:       continue;
  4974: 
  4975:     // For all MBB in the region... except the last one.
  4976:     // ...except when we want to allow local pull-up.
  4977:     for (auto ToThisBB = (*CurrentRegion)->getRootMBB(),
  4978:               LastBBInRegion = (*CurrentRegion)->getLastMBB();
  4979:          ToThisBB != LastBBInRegion; ++ToThisBB) {
  4980:       // If we do not want to allow same BB pull-up, take an early exit.
  4981:       if (!EnableLocalPullUp && (std::next(ToThisBB) == LastBBInRegion))
  4982:         break;
  4983:       if (multipleBranchesFromToBB(*ToThisBB))
  4984:         break;
  4985: 
  4986:       auto FromThisBB = ToThisBB;
  4987:       MachineBasicBlock::iterator ToThisBBEnd = (*ToThisBB)->end();
  4988:       MachineBasicBlock::iterator MI = (*ToThisBB)->begin();
  4989: 
  4990:       LLVM_DEBUG(dbgs() << "\n\tHome iterator moved to new BB("
  4991:                         << (*ToThisBB)->getNumber() << ")\n";
  4992:                  (*ToThisBB)->dump());
  4993: 
  4994:       // For all instructions in the BB.
  4995:       while (MI != ToThisBBEnd) {
  4996:         MachineBasicBlock::iterator WorkPoint = MI;
  4997:         ++MI;
  4998: 
  4999:         // Trivial check that there are unused resources
  5000:         // in the current location (cycle).
```
- EN: It declares or implements routines such as size, LLVM_DEBUG, addPullUpCandidate, getInstrIterator, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 size, LLVM_DEBUG, addPullUpCandidate, getInstrIterator, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 5001-5200 / 第 5001-5200 行

```cpp
  5001:         while (ResourcesAvailableInBundle(*CurrentRegion, WorkPoint)) {
  5002:           LLVM_DEBUG(dbgs() << "\nxxxx Next Home in BB("
  5003:                             << (*ToThisBB)->getNumber() << "):\n";
  5004:                      DumpPacket(WorkPoint.getInstrIterator()));
  5005:           // Keep the path to the candidate.
  5006:           // It is the traveled path between home and work point.
  5007:           // Reset it for the new iteration.
  5008:           backtrack.clear();
  5009: 
  5010:           // The point of pull-up source (WorkPoint) could begin from the
  5011:           // current BB, but only if we allow pull-up in the same BB.
  5012:           // At the moment we do not.
  5013:           // We also do not process last block in the region,
  5014:           // so it is safe to always begin with the next BB in the region.
  5015:           // Start from "next" BB in the region.
  5016:           if (EnableLocalPullUp) {
  5017:             FromThisBB = ToThisBB;
  5018:             FromHere = WorkPoint;
  5019:             ++FromHere;
  5020:             FromThisBBEnd = (*FromThisBB)->end();
  5021: 
  5022:             // Initialize backtrack.
  5023:             // These are instructions between Home location
  5024:             // and the WorkPoint.
  5025:             for (MachineBasicBlock::iterator I = WorkPoint, IE = FromHere;
  5026:                  I != IE; ++I)
  5027:               backtrack.push_back(&*I);
  5028:           } else {
  5029:             FromThisBB = ToThisBB;
  5030:             ++FromThisBB;
  5031:             FromHere = (*FromThisBB)->begin();
  5032:             FromThisBBEnd = (*FromThisBB)->end();
  5033: 
  5034:             // Initialize backtrack.
  5035:             // These are instructions between Home location
  5036:             // and the end of the home BB.
  5037:             for (MachineBasicBlock::iterator I = WorkPoint, IE = ToThisBBEnd;
  5038:                  I != IE; ++I)
  5039:               backtrack.push_back(&*I);
  5040:           }
  5041: 
  5042:           // Search for pull-up candidate.
  5043:           while (true) {
  5044:             // If this BB is over, move onto the next one
  5045:             // in this region.
  5046:             if (FromHere == FromThisBBEnd) {
  5047:               ++FromThisBB;
  5048:               // Refresh LastBBInRegion in case tryMultipleInstructions modified
  5049:               // the regions Elements vector, invalidating the iterator.
  5050:               LastBBInRegion = (*CurrentRegion)->getLastMBB();
  5051:               if (FromThisBB == LastBBInRegion)
  5052:                 break;
  5053:               else {
  5054:                 LLVM_DEBUG(dbgs() << "\n\tNext BB in this region\n";
  5055:                            (*FromThisBB)->dump());
  5056:                 FromThisBBEnd = (*FromThisBB)->end();
  5057:                 FromHere = (*FromThisBB)->begin();
  5058:                 if (FromThisBBEnd == FromHere)
  5059:                   break;
  5060:               }
  5061:             }
  5062:             if ((*FromHere).isDebugInstr()) {
  5063:               ++FromHere;
  5064:               continue;
  5065:             }
  5066:             // This is a step Home.
  5067:             backtrack.push_back(&*FromHere);
  5068:             if (!findPullUpCandidates(WorkPoint, FromHere, backtrack,
  5069:                                       MainCandidateQueueSize))
  5070:               break;
  5071:             ++FromHere;
  5072:           }
  5073:           // Try to pull-up one of the selected candidates.
  5074:           if (!tryMultipleInstructions(/*output*/ WorkPoint, CurrentRegion, MI,
  5075:                                        ToThisBBEnd, FromThisBBEnd))
  5076:             break;
  5077:         }
  5078:       }
  5079:       // Refresh LastBBInRegion after potential CFG modifications.
  5080:       LastBBInRegion = (*CurrentRegion)->getLastMBB();
  5081:     }
  5082:     // AllowUnlikelyPath is on by default,
  5083:     // if we wish to disable it, we can do so here.
  5084:     if (!AllowUnlikelyPath)
  5085:       continue;
  5086: 
  5087:     // We have parsed the likely path through the region.
  5088:     // Now traverse the other (unlikely) path.
  5089:     //
  5090:     // Note: BasicBlockRegion uses a vector for MBB storage, so adding BBs to
  5091:     // the region while iterating could invalidate iterators. Collect the work
  5092:     // items first, then process them.
  5093:     std::vector<std::pair<MachineBasicBlock *, MachineBasicBlock *>>
  5094:         UnlikelyWork;
  5095:     UnlikelyWork.reserve((*CurrentRegion)->size());
  5096:     for (auto ToIt = (*CurrentRegion)->getRootMBB(),
  5097:               End = (*CurrentRegion)->getLastMBB();
  5098:          ToIt != End; ++ToIt) {
  5099:       MachineBasicBlock *ToBB = *ToIt;
  5100:       MachineBasicBlock *SecondBest = getNextPURBB(ToBB, true);
  5101:       if (SecondBest)
  5102:         UnlikelyWork.emplace_back(ToBB, SecondBest);
  5103:     }
  5104: 
  5105:     for (auto [ToBB, SecondBest] : UnlikelyWork) {
  5106:       LLVM_DEBUG(dbgs() << "\tFor BB:\n"; ToBB->dump());
  5107:       LLVM_DEBUG(dbgs() << "\tHave SecondBest:\n"; SecondBest->dump());
  5108:       // Adding this BB to the region should not be done if we
  5109:       // plan to reuse it(the region) again. For now it is OK.
  5110:       (*CurrentRegion)->addBBtoRegion(SecondBest);
  5111:       LLVM_DEBUG(dbgs() << "\tHome iterator moved to new BB("
  5112:                         << ToBB->getNumber() << ")\n";
  5113:                  ToBB->dump());
  5114:       MachineBasicBlock::iterator ToThisBBEnd = ToBB->end();
  5115:       MachineBasicBlock::iterator MI = ToBB->begin();
  5116: 
  5117:       // For all instructions in the BB.
  5118:       while (MI != ToThisBBEnd) {
  5119:         MachineBasicBlock::iterator WorkPoint = MI;
  5120:         ++MI;
  5121: 
  5122:         // Trivial check that there are unused resources
  5123:         // in the current location (cycle).
  5124:         while (ResourcesAvailableInBundle(*CurrentRegion, WorkPoint)) {
  5125:           LLVM_DEBUG(dbgs() << "\nxxxx Second visit Home in BB("
  5126:                             << ToBB->getNumber() << "):\n";
  5127:                      DumpPacket(WorkPoint.getInstrIterator()));
  5128: 
  5129:           FromHere = SecondBest->begin();
  5130:           FromThisBBEnd = SecondBest->end();
  5131: 
  5132:           // Keep the path to the candidate.
  5133:           backtrack.clear();
  5134: 
  5135:           // This is Home location.
  5136:           for (MachineBasicBlock::iterator I = WorkPoint, IE = ToThisBBEnd;
  5137:                I != IE; ++I)
  5138:             backtrack.push_back(&*I);
  5139: 
  5140:           while (true) {
  5141:             // If this BB is over, move onto the next one
  5142:             // in this region.
  5143:             if (FromHere == FromThisBBEnd) {
  5144:               LLVM_DEBUG(dbgs()
  5145:                          << "\tOnly do one successor for the second try\n");
  5146:               break;
  5147:             }
  5148:             if ((*FromHere).isDebugInstr()) {
  5149:               ++FromHere;
  5150:               continue;
  5151:             }
  5152:             // This is a step Home.
  5153:             backtrack.push_back(&*FromHere);
  5154:             if (!findPullUpCandidates(WorkPoint, FromHere, backtrack,
  5155:                                       SecondaryCandidateQueueSize))
  5156:               break;
  5157:             ++FromHere;
  5158:           }
  5159:           // Try to pull-up one of selected candidate.
  5160:           if (!tryMultipleInstructions(/*output*/ WorkPoint, CurrentRegion, MI,
  5161:                                        ToThisBBEnd, FromThisBBEnd, false))
  5162:             break;
  5163:         }
  5164:       }
  5165:     }
  5166:   }
  5167:   return true;
  5168: }
  5169: 
  5170: bool HexagonGlobalSchedulerImpl::incrementalAddToPacket(MachineInstr &MI) {
  5171: 
  5172:   LLVM_DEBUG(dbgs() << "\t[AddToPacket] (" << CurrentPacketMIs.size()
  5173:                     << ") adding:\t";
  5174:              MI.dump());
  5175: 
  5176:   if (!ResourceTracker->canReserveResources(MI) || !shouldAddToPacket(MI))
  5177:     return false;
  5178: 
  5179:   ResourceTracker->reserveResources(MI);
  5180:   CurrentPacketMIs.push_back(&MI);
  5181:   CurrentState.HomeBundle.push_back(&MI);
  5182: 
  5183:   if (QII->isExtended(MI) || QII->isConstExtended(MI) ||
  5184:       isJumpOutOfRange(&MI)) {
  5185:     // If at this point of time we cannot reserve resources,
  5186:     // this might mean that the packet came into the pull-up
  5187:     // pass already in danger of overflowing.
  5188:     // Nevertheless, since this is only a possibility of overflow
  5189:     // no error should be issued here.
  5190:     if (ResourceTracker->canReserveResources(*Ext)) {
  5191:       ResourceTracker->reserveResources(*Ext);
  5192:       LLVM_DEBUG(dbgs() << "\t[AddToPacket] (" << CurrentPacketMIs.size()
  5193:                         << ") adding:\t  immext_i\n");
  5194:       CurrentPacketMIs.push_back(Ext);
  5195:       CurrentState.HomeBundle.push_back(Ext);
  5196:       return true;
  5197:     } else {
  5198:       LLVM_DEBUG(dbgs() << "\t  Previous overflow possible.\n");
  5199:       return false;
  5200:     }
```
- EN: It declares or implements routines such as DumpPacket, clear, source, end, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 DumpPacket, clear, source, end, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl，说明了它与同级后端组件的连接关系。

### Lines 5201-5330 / 第 5201-5330 行

```cpp
  5201:   }
  5202:   return true;
  5203: }
  5204: 
  5205: void HexagonGlobalSchedulerImpl::checkBundleCounts(MachineFunction &Fn) {
  5206:   if (DisableCheckBundles)
  5207:     return;
  5208: 
  5209:   unsigned BundleLimit = 4;
  5210: 
  5211:   for (MachineFunction::iterator MBBi = Fn.begin(), MBBe = Fn.end();
  5212:        MBBi != MBBe; ++MBBi) {
  5213: 
  5214:     for (MachineBasicBlock::iterator MI = MBBi->instr_begin(),
  5215:                                      ME = MBBi->instr_end();
  5216:          MI != ME; ++MI) {
  5217:       if (MI->isBundle()) {
  5218:         MachineBasicBlock::instr_iterator MII = MI.getInstrIterator();
  5219:         MachineBasicBlock::instr_iterator End = MII->getParent()->instr_end();
  5220: 
  5221:         unsigned InstrCount = 0;
  5222: 
  5223:         for (++MII; MII != End && MII->isInsideBundle(); ++MII) {
  5224:           if (MII->getOpcode() == TargetOpcode::DBG_VALUE ||
  5225:               MII->getOpcode() == TargetOpcode::IMPLICIT_DEF ||
  5226:               MII->getOpcode() == TargetOpcode::CFI_INSTRUCTION ||
  5227:               MII->isEHLabel() || QII->isEndLoopN(MII->getOpcode())) {
  5228:             continue;
  5229:           } else {
  5230:             InstrCount++;
  5231:           }
  5232:         }
  5233:         if (InstrCount > BundleLimit) {
  5234:           if (WarnOnBundleSize) {
  5235:             LLVM_DEBUG(dbgs() << "Warning bundle size exceeded " << *MI);
  5236:           } else {
  5237:             assert(0 && "Bundle size exceeded");
  5238:           }
  5239:         }
  5240:       }
  5241:     }
  5242:   }
  5243: }
  5244: 
  5245: /// Debugging only. Count compound and duplex opportunities.
  5246: unsigned HexagonGlobalSchedulerImpl::countCompounds(MachineFunction &Fn) {
  5247:   unsigned CompoundCount = 0;
  5248:   [[maybe_unused]] unsigned DuplexCount = 0;
  5249:   [[maybe_unused]] unsigned InstOffset = 0;
  5250: 
  5251:   // Loop over all basic blocks.
  5252:   for (MachineFunction::iterator MBB = Fn.begin(), MBBe = Fn.end(); MBB != MBBe;
  5253:        ++MBB) {
  5254:     LLVM_DEBUG(dbgs() << "\n BB#" << MBB->getNumber() << " " << MBB->getName()
  5255:                       << " in_func "
  5256:                       << MBB->getParent()->getFunction().getName() << " \n");
  5257:     for (MachineBasicBlock::iterator MI = MBB->instr_begin(),
  5258:                                      ME = MBB->instr_end();
  5259:          MI != ME; ++MI) {
  5260:       if (MI->isDebugInstr())
  5261:         continue;
  5262:       if (MI->isBundle()) {
  5263:         MachineBasicBlock::instr_iterator MII = MI.getInstrIterator();
  5264:         MachineBasicBlock::instr_iterator MIE = MI->getParent()->instr_end();
  5265:         MachineInstr *FirstCompound = NULL, *SecondCompound = NULL;
  5266:         MachineInstr *FirstDuplex = NULL, *SecondDuplex = NULL;
  5267:         LLVM_DEBUG(dbgs() << "{\n");
  5268: 
  5269:         for (++MII; MII != MIE && MII->isInsideBundle() && !MII->isBundle();
  5270:              ++MII) {
  5271:           if (MII->isDebugInstr())
  5272:             continue;
  5273:           LLVM_DEBUG(dbgs() << "(" << InstOffset << ")\t");
  5274:           InstOffset += QII->getSize(*MII);
  5275:           if (QII->getCompoundCandidateGroup(*MII)) {
  5276:             if (!FirstCompound) {
  5277:               FirstCompound = &*MII;
  5278:               LLVM_DEBUG(dbgs() << "XX ");
  5279:             } else {
  5280:               SecondCompound = &*MII;
  5281:               LLVM_DEBUG(dbgs() << "YY ");
  5282:             }
  5283:           }
  5284:           if (QII->getDuplexCandidateGroup(*MII)) {
  5285:             if (!FirstDuplex) {
  5286:               FirstDuplex = &*MII;
  5287:               LLVM_DEBUG(dbgs() << "AA ");
  5288:             } else {
  5289:               SecondDuplex = &*MII;
  5290:               LLVM_DEBUG(dbgs() << "VV ");
  5291:             }
  5292:           }
  5293:           LLVM_DEBUG(MII->dump());
  5294:         }
  5295:         LLVM_DEBUG(dbgs() << "}\n");
  5296:         if (SecondCompound) {
  5297:           if (isCompoundPair(FirstCompound, SecondCompound)) {
  5298:             LLVM_DEBUG(dbgs() << "Compound pair (" << CompoundCount << ")\n");
  5299:             CompoundCount++;
  5300:           }
  5301:         }
  5302:         if (SecondDuplex) {
  5303:           if (QII->isDuplexPair(*FirstDuplex, *SecondDuplex)) {
  5304:             LLVM_DEBUG(dbgs() << "Duplex pair (" << DuplexCount << ")\n");
  5305:             DuplexCount++;
  5306:           }
  5307:         }
  5308:       } else {
  5309:         LLVM_DEBUG(dbgs() << "(" << InstOffset << ")\t");
  5310:         if (QII->getCompoundCandidateGroup(*MI))
  5311:           LLVM_DEBUG(dbgs() << "XX ");
  5312:         if (QII->getDuplexCandidateGroup(*MI))
  5313:           LLVM_DEBUG(dbgs() << "AA ");
  5314:         InstOffset += QII->getSize(*MI);
  5315:         LLVM_DEBUG(MI->dump());
  5316:       }
  5317:     }
  5318:   }
  5319:   LLVM_DEBUG(dbgs() << "Total compound(" << CompoundCount << ") duplex("
  5320:                     << DuplexCount << ")\n");
  5321:   return CompoundCount;
  5322: }
  5323: 
  5324: //===----------------------------------------------------------------------===//
  5325: //                         Public Constructor Functions
  5326: //===----------------------------------------------------------------------===//
  5327: 
  5328: FunctionPass *llvm::createHexagonGlobalScheduler() {
  5329:   return new HexagonGlobalScheduler();
  5330: }
```
- EN: It declares or implements routines such as HexagonGlobalSchedulerImpl::checkBundleCounts, getInstrIterator, getParent, isInsideBundle, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGlobalSchedulerImpl, HexagonGlobalScheduler, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGlobalSchedulerImpl::checkBundleCounts, getInstrIterator, getParent, isInsideBundle, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGlobalSchedulerImpl, HexagonGlobalScheduler，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- instruction scheduling models / 指令调度模型
- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonGlobalRegion.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, HexagonTargetMachine.h, HexagonVLIWPacketizer.h, llvm/ADT/DenseMap.h, llvm/ADT/SmallSet.h, llvm/ADT/Statistic.h, ... (40 total)`
- Hexagon symbols / Hexagon 符号: `HexagonGlobalScheduler, HexagonGlobalRegion, HexagonMachineFunctionInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonTargetMachine, HexagonVLIWPacketizer, HexagonNumPullUps, HexagonNumDualJumps, HexagonGlobalSchedulerImpl, ... (13 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
