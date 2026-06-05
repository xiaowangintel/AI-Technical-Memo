# HexagonEarlyIfConv.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonEarlyIfConv.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon early if-conversion over predicated control flow.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonEarlyIfConv.cpp ---------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements a Hexagon-specific if-conversion pass that runs on the
    10: // SSA form.
    11: // In SSA it is not straightforward to represent instructions that condi-
    12: // tionally define registers, since a conditionally-defined register may
    13: // only be used under the same condition on which the definition was based.
    14: // To avoid complications of this nature, this patch will only generate
    15: // predicated stores, and speculate other instructions from the "if-conver-
    16: // ted" block.
    17: // The code will recognize CFG patterns where a block with a conditional
    18: // branch "splits" into a "true block" and a "false block". Either of these
    19: // could be omitted (in case of a triangle, for example).
    20: // If after conversion of the side block(s) the CFG allows it, the resul-
    21: // ting blocks may be merged. If the "join" block contained PHI nodes, they
    22: // will be replaced with MUX (or MUX-like) instructions to maintain the
    23: // semantics of the PHI.
    24: //
    25: // Example:
    26: //
    27: //         %40 = L2_loadrub_io killed %39, 1
    28: //         %41 = S2_tstbit_i killed %40, 0
    29: //         J2_jumpt killed %41, <%bb.5>, implicit dead %pc
    30: //         J2_jump <%bb.4>, implicit dead %pc
    31: //     Successors according to CFG: %bb.4(62) %bb.5(62)
    32: //
    33: // %bb.4: derived from LLVM BB %if.then
    34: //     Predecessors according to CFG: %bb.3
    35: //         %11 = A2_addp %6, %10
    36: //         S2_storerd_io %32, 16, %11
    37: //     Successors according to CFG: %bb.5
    38: //
    39: // %bb.5: derived from LLVM BB %if.end
    40: //     Predecessors according to CFG: %bb.3 %bb.4
    41: //         %12 = PHI %6, <%bb.3>, %11, <%bb.4>
    42: //         %13 = A2_addp %7, %12
    43: //         %42 = C2_cmpeqi %9, 10
    44: //         J2_jumpf killed %42, <%bb.3>, implicit dead %pc
    45: //         J2_jump <%bb.6>, implicit dead %pc
    46: //     Successors according to CFG: %bb.6(4) %bb.3(124)
    47: //
    48: // would become:
    49: //
    50: //         %40 = L2_loadrub_io killed %39, 1
    51: //         %41 = S2_tstbit_i killed %40, 0
    52: // spec->  %11 = A2_addp %6, %10
    53: // pred->  S2_pstorerdf_io %41, %32, 16, %11
    54: //         %46 = PS_pselect %41, %6, %11
    55: //         %13 = A2_addp %7, %46
    56: //         %42 = C2_cmpeqi %9, 10
    57: //         J2_jumpf killed %42, <%bb.3>, implicit dead %pc
    58: //         J2_jump <%bb.6>, implicit dead %pc
    59: //     Successors according to CFG: %bb.6 %bb.3
    60: 
    61: #include "Hexagon.h"
    62: #include "HexagonInstrInfo.h"
    63: #include "HexagonSubtarget.h"
    64: #include "llvm/ADT/DenseSet.h"
    65: #include "llvm/ADT/SmallVector.h"
    66: #include "llvm/ADT/StringRef.h"
    67: #include "llvm/ADT/iterator_range.h"
    68: #include "llvm/CodeGen/MachineBasicBlock.h"
    69: #include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
    70: #include "llvm/CodeGen/MachineDominators.h"
    71: #include "llvm/CodeGen/MachineFunction.h"
    72: #include "llvm/CodeGen/MachineFunctionPass.h"
    73: #include "llvm/CodeGen/MachineInstr.h"
    74: #include "llvm/CodeGen/MachineInstrBuilder.h"
    75: #include "llvm/CodeGen/MachineLoopInfo.h"
    76: #include "llvm/CodeGen/MachineOperand.h"
    77: #include "llvm/CodeGen/MachineRegisterInfo.h"
    78: #include "llvm/CodeGen/TargetRegisterInfo.h"
    79: #include "llvm/IR/DebugLoc.h"
    80: #include "llvm/Pass.h"
    81: #include "llvm/Support/BranchProbability.h"
    82: #include "llvm/Support/CommandLine.h"
    83: #include "llvm/Support/Compiler.h"
    84: #include "llvm/Support/Debug.h"
    85: #include "llvm/Support/ErrorHandling.h"
    86: #include "llvm/Support/raw_ostream.h"
    87: #include <cassert>
    88: #include <iterator>
    89: 
    90: #define DEBUG_TYPE "hexagon-eif"
    91: 
    92: using namespace llvm;
    93: 
    94: static cl::opt<bool> EnableHexagonBP("enable-hexagon-br-prob", cl::Hidden,
    95:   cl::init(true), cl::desc("Enable branch probability info"));
    96: static cl::opt<unsigned> SizeLimit("eif-limit", cl::init(6), cl::Hidden,
    97:   cl::desc("Size limit in Hexagon early if-conversion"));
    98: static cl::opt<bool> SkipExitBranches("eif-no-loop-exit", cl::init(false),
    99:   cl::Hidden, cl::desc("Do not convert branches that may exit the loop"));
   100: 
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseSet.h, ... (28 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseSet.h, ... (28 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。

### Lines 101-200 / 第 101-200 行

```cpp
   101: namespace {
   102: 
   103:   struct PrintMB {
   104:     PrintMB(const MachineBasicBlock *B) : MB(B) {}
   105: 
   106:     const MachineBasicBlock *MB;
   107:   };
   108:   raw_ostream &operator<< (raw_ostream &OS, const PrintMB &P) {
   109:     if (!P.MB)
   110:       return OS << "<none>";
   111:     return OS << '#' << P.MB->getNumber();
   112:   }
   113: 
   114:   struct FlowPattern {
   115:     FlowPattern() = default;
   116:     FlowPattern(MachineBasicBlock *B, unsigned PR, MachineBasicBlock *TB,
   117:           MachineBasicBlock *FB, MachineBasicBlock *JB)
   118:       : SplitB(B), TrueB(TB), FalseB(FB), JoinB(JB), PredR(PR) {}
   119: 
   120:     MachineBasicBlock *SplitB = nullptr;
   121:     MachineBasicBlock *TrueB = nullptr;
   122:     MachineBasicBlock *FalseB = nullptr;
   123:     MachineBasicBlock *JoinB = nullptr;
   124:     unsigned PredR = 0;
   125:   };
   126: 
   127:   struct PrintFP {
   128:     PrintFP(const FlowPattern &P, const TargetRegisterInfo &T)
   129:       : FP(P), TRI(T) {}
   130: 
   131:     const FlowPattern &FP;
   132:     const TargetRegisterInfo &TRI;
   133:     friend raw_ostream &operator<< (raw_ostream &OS, const PrintFP &P);
   134:   };
   135:   [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS, const PrintFP &P);
   136:   raw_ostream &operator<<(raw_ostream &OS, const PrintFP &P) {
   137:     OS << "{ SplitB:" << PrintMB(P.FP.SplitB)
   138:        << ", PredR:" << printReg(P.FP.PredR, &P.TRI)
   139:        << ", TrueB:" << PrintMB(P.FP.TrueB)
   140:        << ", FalseB:" << PrintMB(P.FP.FalseB)
   141:        << ", JoinB:" << PrintMB(P.FP.JoinB) << " }";
   142:     return OS;
   143:   }
   144: 
   145:   class HexagonEarlyIfConversion : public MachineFunctionPass {
   146:   public:
   147:     static char ID;
   148: 
   149:     HexagonEarlyIfConversion() : MachineFunctionPass(ID) {}
   150: 
   151:     StringRef getPassName() const override {
   152:       return "Hexagon early if conversion";
   153:     }
   154: 
   155:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   156:       AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();
   157:       AU.addRequired<MachineDominatorTreeWrapperPass>();
   158:       AU.addPreserved<MachineDominatorTreeWrapperPass>();
   159:       AU.addRequired<MachineLoopInfoWrapperPass>();
   160:       MachineFunctionPass::getAnalysisUsage(AU);
   161:     }
   162: 
   163:     bool runOnMachineFunction(MachineFunction &MF) override;
   164: 
   165:   private:
   166:     using BlockSetType = DenseSet<MachineBasicBlock *>;
   167: 
   168:     bool isPreheader(const MachineBasicBlock *B) const;
   169:     bool matchFlowPattern(MachineBasicBlock *B, MachineLoop *L,
   170:           FlowPattern &FP);
   171:     bool visitBlock(MachineBasicBlock *B, MachineLoop *L);
   172:     bool visitLoop(MachineLoop *L);
   173: 
   174:     bool hasEHLabel(const MachineBasicBlock *B) const;
   175:     bool hasUncondBranch(const MachineBasicBlock *B) const;
   176:     bool isValidCandidate(const MachineBasicBlock *B) const;
   177:     bool usesUndefVReg(const MachineInstr *MI) const;
   178:     bool isValid(const FlowPattern &FP) const;
   179:     unsigned countPredicateDefs(const MachineBasicBlock *B) const;
   180:     unsigned computePhiCost(const MachineBasicBlock *B,
   181:           const FlowPattern &FP) const;
   182:     bool isProfitable(const FlowPattern &FP) const;
   183:     bool isPredicableStore(const MachineInstr *MI) const;
   184:     bool isSafeToSpeculate(const MachineInstr *MI) const;
   185:     bool isPredicate(unsigned R) const;
   186: 
   187:     unsigned getCondStoreOpcode(unsigned Opc, bool IfTrue) const;
   188:     void predicateInstr(MachineBasicBlock *ToB, MachineBasicBlock::iterator At,
   189:           MachineInstr *MI, unsigned PredR, bool IfTrue);
   190:     void predicateBlockNB(MachineBasicBlock *ToB,
   191:           MachineBasicBlock::iterator At, MachineBasicBlock *FromB,
   192:           unsigned PredR, bool IfTrue);
   193: 
   194:     unsigned buildMux(MachineBasicBlock *B, MachineBasicBlock::iterator At,
   195:           const TargetRegisterClass *DRC, unsigned PredR, unsigned TR,
   196:           unsigned TSR, unsigned FR, unsigned FSR);
   197:     void updatePhiNodes(MachineBasicBlock *WhereB, const FlowPattern &FP);
   198:     void convert(const FlowPattern &FP);
   199: 
   200:     void removeBlock(MachineBasicBlock *B);
```
- EN: It declares types such as PrintMB, FlowPattern, PrintFP, HexagonEarlyIfConversion, which carry the state or API of this component. It defines declarative TableGen records like HexagonEarlyIfConversion; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as PrintMB, getNumber, FlowPattern, PrintFP, ... (35 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 PrintMB, FlowPattern, PrintFP, HexagonEarlyIfConversion 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonEarlyIfConversion 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 PrintMB, getNumber, FlowPattern, PrintFP, ... (35 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-300 / 第 201-300 行

```cpp
   201:     void eliminatePhis(MachineBasicBlock *B);
   202:     void mergeBlocks(MachineBasicBlock *PredB, MachineBasicBlock *SuccB);
   203:     void simplifyFlowGraph(const FlowPattern &FP);
   204: 
   205:     const HexagonInstrInfo *HII = nullptr;
   206:     const TargetRegisterInfo *TRI = nullptr;
   207:     MachineFunction *MFN = nullptr;
   208:     MachineRegisterInfo *MRI = nullptr;
   209:     MachineDominatorTree *MDT = nullptr;
   210:     MachineLoopInfo *MLI = nullptr;
   211:     BlockSetType Deleted;
   212:     const MachineBranchProbabilityInfo *MBPI = nullptr;
   213:   };
   214: 
   215: } // end anonymous namespace
   216: 
   217: char HexagonEarlyIfConversion::ID = 0;
   218: 
   219: INITIALIZE_PASS(HexagonEarlyIfConversion, "hexagon-early-if",
   220:   "Hexagon early if conversion", false, false)
   221: 
   222: bool HexagonEarlyIfConversion::isPreheader(const MachineBasicBlock *B) const {
   223:   if (B->succ_size() != 1)
   224:     return false;
   225:   MachineBasicBlock *SB = *B->succ_begin();
   226:   MachineLoop *L = MLI->getLoopFor(SB);
   227:   return L && SB == L->getHeader() && MDT->dominates(B, SB);
   228: }
   229: 
   230: bool HexagonEarlyIfConversion::matchFlowPattern(MachineBasicBlock *B,
   231:     MachineLoop *L, FlowPattern &FP) {
   232:   LLVM_DEBUG(dbgs() << "Checking flow pattern at " << printMBBReference(*B)
   233:                     << "\n");
   234: 
   235:   // Interested only in conditional branches, no .new, no new-value, etc.
   236:   // Check the terminators directly, it's easier than handling all responses
   237:   // from analyzeBranch.
   238:   MachineBasicBlock *TB = nullptr, *FB = nullptr;
   239:   MachineBasicBlock::const_iterator T1I = B->getFirstTerminator();
   240:   if (T1I == B->end())
   241:     return false;
   242:   unsigned Opc = T1I->getOpcode();
   243:   if (Opc != Hexagon::J2_jumpt && Opc != Hexagon::J2_jumpf)
   244:     return false;
   245:   Register PredR = T1I->getOperand(0).getReg();
   246: 
   247:   // Get the layout successor, or 0 if B does not have one.
   248:   MachineFunction::iterator NextBI = std::next(MachineFunction::iterator(B));
   249:   MachineBasicBlock *NextB = (NextBI != MFN->end()) ? &*NextBI : nullptr;
   250: 
   251:   MachineBasicBlock *T1B = T1I->getOperand(1).getMBB();
   252:   MachineBasicBlock::const_iterator T2I = std::next(T1I);
   253:   // The second terminator should be an unconditional branch.
   254:   assert(T2I == B->end() || T2I->getOpcode() == Hexagon::J2_jump);
   255:   MachineBasicBlock *T2B = (T2I == B->end()) ? NextB
   256:                                              : T2I->getOperand(0).getMBB();
   257:   if (T1B == T2B) {
   258:     // XXX merge if T1B == NextB, or convert branch to unconditional.
   259:     // mark as diamond with both sides equal?
   260:     return false;
   261:   }
   262: 
   263:   // Record the true/false blocks in such a way that "true" means "if (PredR)",
   264:   // and "false" means "if (!PredR)".
   265:   if (Opc == Hexagon::J2_jumpt)
   266:     TB = T1B, FB = T2B;
   267:   else
   268:     TB = T2B, FB = T1B;
   269: 
   270:   if (!MDT->properlyDominates(B, TB) || !MDT->properlyDominates(B, FB))
   271:     return false;
   272: 
   273:   // Detect triangle first. In case of a triangle, one of the blocks TB/FB
   274:   // can fall through into the other, in other words, it will be executed
   275:   // in both cases. We only want to predicate the block that is executed
   276:   // conditionally.
   277:   assert(TB && FB && "Failed to find triangle control flow blocks");
   278:   unsigned TNP = TB->pred_size(), FNP = FB->pred_size();
   279:   unsigned TNS = TB->succ_size(), FNS = FB->succ_size();
   280: 
   281:   // A block is predicable if it has one predecessor (it must be B), and
   282:   // it has a single successor. In fact, the block has to end either with
   283:   // an unconditional branch (which can be predicated), or with a fall-
   284:   // through.
   285:   // Also, skip blocks that do not belong to the same loop.
   286:   bool TOk = (TNP == 1 && TNS == 1 && MLI->getLoopFor(TB) == L);
   287:   bool FOk = (FNP == 1 && FNS == 1 && MLI->getLoopFor(FB) == L);
   288: 
   289:   // If requested (via an option), do not consider branches where the
   290:   // true and false targets do not belong to the same loop.
   291:   if (SkipExitBranches && MLI->getLoopFor(TB) != MLI->getLoopFor(FB))
   292:     return false;
   293: 
   294:   // If neither is predicable, there is nothing interesting.
   295:   if (!TOk && !FOk)
   296:     return false;
   297: 
   298:   MachineBasicBlock *TSB = (TNS > 0) ? *TB->succ_begin() : nullptr;
   299:   MachineBasicBlock *FSB = (FNS > 0) ? *FB->succ_begin() : nullptr;
   300:   MachineBasicBlock *JB = nullptr;
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as eliminatePhis, mergeBlocks, simplifyFlowGraph, INITIALIZE_PASS, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 eliminatePhis, mergeBlocks, simplifyFlowGraph, INITIALIZE_PASS, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 301-400 / 第 301-400 行

```cpp
   301: 
   302:   if (TOk) {
   303:     if (FOk) {
   304:       if (TSB == FSB)
   305:         JB = TSB;
   306:       // Diamond: "if (P) then TB; else FB;".
   307:     } else {
   308:       // TOk && !FOk
   309:       if (TSB == FB)
   310:         JB = FB;
   311:       FB = nullptr;
   312:     }
   313:   } else {
   314:     // !TOk && FOk  (at least one must be true by now).
   315:     if (FSB == TB)
   316:       JB = TB;
   317:     TB = nullptr;
   318:   }
   319:   // Don't try to predicate loop preheaders.
   320:   if ((TB && isPreheader(TB)) || (FB && isPreheader(FB))) {
   321:     LLVM_DEBUG(dbgs() << "One of blocks " << PrintMB(TB) << ", " << PrintMB(FB)
   322:                       << " is a loop preheader. Skipping.\n");
   323:     return false;
   324:   }
   325: 
   326:   FP = FlowPattern(B, PredR, TB, FB, JB);
   327:   LLVM_DEBUG(dbgs() << "Detected " << PrintFP(FP, *TRI) << "\n");
   328:   return true;
   329: }
   330: 
   331: // KLUDGE: HexagonInstrInfo::analyzeBranch won't work on a block that
   332: // contains EH_LABEL.
   333: bool HexagonEarlyIfConversion::hasEHLabel(const MachineBasicBlock *B) const {
   334:   for (auto &I : *B)
   335:     if (I.isEHLabel())
   336:       return true;
   337:   return false;
   338: }
   339: 
   340: // KLUDGE: HexagonInstrInfo::analyzeBranch may be unable to recognize
   341: // that a block can never fall-through.
   342: bool HexagonEarlyIfConversion::hasUncondBranch(const MachineBasicBlock *B)
   343:       const {
   344:   MachineBasicBlock::const_iterator I = B->getFirstTerminator(), E = B->end();
   345:   while (I != E) {
   346:     if (I->isBarrier())
   347:       return true;
   348:     ++I;
   349:   }
   350:   return false;
   351: }
   352: 
   353: bool HexagonEarlyIfConversion::isValidCandidate(const MachineBasicBlock *B)
   354:       const {
   355:   if (!B)
   356:     return true;
   357:   if (B->isEHPad() || B->hasAddressTaken())
   358:     return false;
   359:   if (B->succ_empty())
   360:     return false;
   361: 
   362:   for (auto &MI : *B) {
   363:     if (MI.isDebugInstr())
   364:       continue;
   365:     if (MI.isConditionalBranch())
   366:       return false;
   367:     unsigned Opc = MI.getOpcode();
   368:     bool IsJMP = (Opc == Hexagon::J2_jump);
   369:     if (!isPredicableStore(&MI) && !IsJMP && !isSafeToSpeculate(&MI))
   370:       return false;
   371:     // Look for predicate registers defined by this instruction. It's ok
   372:     // to speculate such an instruction, but the predicate register cannot
   373:     // be used outside of this block (or else it won't be possible to
   374:     // update the use of it after predication). PHI uses will be updated
   375:     // to use a result of a MUX, and a MUX cannot be created for predicate
   376:     // registers.
   377:     for (const MachineOperand &MO : MI.operands()) {
   378:       if (!MO.isReg() || !MO.isDef())
   379:         continue;
   380:       Register R = MO.getReg();
   381:       if (!R.isVirtual())
   382:         continue;
   383:       if (!isPredicate(R))
   384:         continue;
   385:       for (const MachineOperand &U : MRI->use_operands(R))
   386:         if (U.getParent()->isPHI())
   387:           return false;
   388:     }
   389:   }
   390:   return true;
   391: }
   392: 
   393: bool HexagonEarlyIfConversion::usesUndefVReg(const MachineInstr *MI) const {
   394:   for (const MachineOperand &MO : MI->operands()) {
   395:     if (!MO.isReg() || !MO.isUse())
   396:       continue;
   397:     Register R = MO.getReg();
   398:     if (!R.isVirtual())
   399:       continue;
   400:     const MachineInstr *DefI = MRI->getVRegDef(R);
```
- EN: It declares or implements routines such as LLVM_DEBUG, FlowPattern, HexagonEarlyIfConversion::hasEHLabel, HexagonEarlyIfConversion::hasUncondBranch, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonEarlyIfConversion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, FlowPattern, HexagonEarlyIfConversion::hasEHLabel, HexagonEarlyIfConversion::hasUncondBranch, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonEarlyIfConversion，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:     // "Undefined" virtual registers are actually defined via IMPLICIT_DEF.
   402:     assert(DefI && "Expecting a reaching def in MRI");
   403:     if (DefI->isImplicitDef())
   404:       return true;
   405:   }
   406:   return false;
   407: }
   408: 
   409: bool HexagonEarlyIfConversion::isValid(const FlowPattern &FP) const {
   410:   if (hasEHLabel(FP.SplitB))  // KLUDGE: see function definition
   411:     return false;
   412:   if (FP.TrueB && !isValidCandidate(FP.TrueB))
   413:     return false;
   414:   if (FP.FalseB && !isValidCandidate(FP.FalseB))
   415:     return false;
   416:   // Check the PHIs in the join block. If any of them use a register
   417:   // that is defined as IMPLICIT_DEF, do not convert this. This can
   418:   // legitimately happen if one side of the split never executes, but
   419:   // the compiler is unable to prove it. That side may then seem to
   420:   // provide an "undef" value to the join block, however it will never
   421:   // execute at run-time. If we convert this case, the "undef" will
   422:   // be used in a MUX instruction, and that may seem like actually
   423:   // using an undefined value to other optimizations. This could lead
   424:   // to trouble further down the optimization stream, cause assertions
   425:   // to fail, etc.
   426:   if (FP.JoinB) {
   427:     const MachineBasicBlock &B = *FP.JoinB;
   428:     for (auto &MI : B) {
   429:       if (!MI.isPHI())
   430:         break;
   431:       if (usesUndefVReg(&MI))
   432:         return false;
   433:       Register DefR = MI.getOperand(0).getReg();
   434:       if (isPredicate(DefR))
   435:         return false;
   436:     }
   437:   }
   438:   return true;
   439: }
   440: 
   441: unsigned HexagonEarlyIfConversion::computePhiCost(const MachineBasicBlock *B,
   442:       const FlowPattern &FP) const {
   443:   if (B->pred_size() < 2)
   444:     return 0;
   445: 
   446:   unsigned Cost = 0;
   447:   for (const MachineInstr &MI : *B) {
   448:     if (!MI.isPHI())
   449:       break;
   450:     // If both incoming blocks are one of the TrueB/FalseB/SplitB, then
   451:     // a MUX may be needed. Otherwise the PHI will need to be updated at
   452:     // no extra cost.
   453:     // Find the interesting PHI operands for further checks.
   454:     SmallVector<unsigned,2> Inc;
   455:     for (unsigned i = 1, e = MI.getNumOperands(); i != e; i += 2) {
   456:       const MachineBasicBlock *BB = MI.getOperand(i+1).getMBB();
   457:       if (BB == FP.SplitB || BB == FP.TrueB || BB == FP.FalseB)
   458:         Inc.push_back(i);
   459:     }
   460:     assert(Inc.size() <= 2);
   461:     if (Inc.size() < 2)
   462:       continue;
   463: 
   464:     const MachineOperand &RA = MI.getOperand(1);
   465:     const MachineOperand &RB = MI.getOperand(3);
   466:     assert(RA.isReg() && RB.isReg());
   467:     // Must have a MUX if the phi uses a subregister.
   468:     if (RA.getSubReg() != 0 || RB.getSubReg() != 0) {
   469:       Cost++;
   470:       continue;
   471:     }
   472:     const MachineInstr *Def1 = MRI->getVRegDef(RA.getReg());
   473:     const MachineInstr *Def3 = MRI->getVRegDef(RB.getReg());
   474:     if (!HII->isPredicable(*Def1) || !HII->isPredicable(*Def3))
   475:       Cost++;
   476:   }
   477:   return Cost;
   478: }
   479: 
   480: unsigned HexagonEarlyIfConversion::countPredicateDefs(
   481:       const MachineBasicBlock *B) const {
   482:   unsigned PredDefs = 0;
   483:   for (auto &MI : *B) {
   484:     for (const MachineOperand &MO : MI.operands()) {
   485:       if (!MO.isReg() || !MO.isDef())
   486:         continue;
   487:       Register R = MO.getReg();
   488:       if (!R.isVirtual())
   489:         continue;
   490:       if (isPredicate(R))
   491:         PredDefs++;
   492:     }
   493:   }
   494:   return PredDefs;
   495: }
   496: 
   497: bool HexagonEarlyIfConversion::isProfitable(const FlowPattern &FP) const {
   498:   BranchProbability JumpProb(1, 10);
   499:   BranchProbability Prob(9, 10);
   500:   if (MBPI && FP.TrueB && !FP.FalseB &&
```
- EN: It declares or implements routines such as assert, HexagonEarlyIfConversion::isValid, getOperand, HexagonEarlyIfConversion::computePhiCost, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonEarlyIfConversion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, HexagonEarlyIfConversion::isValid, getOperand, HexagonEarlyIfConversion::computePhiCost, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonEarlyIfConversion，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:       (MBPI->getEdgeProbability(FP.SplitB, FP.TrueB) < JumpProb ||
   502:        MBPI->getEdgeProbability(FP.SplitB, FP.TrueB) > Prob))
   503:     return false;
   504: 
   505:   if (MBPI && !FP.TrueB && FP.FalseB &&
   506:       (MBPI->getEdgeProbability(FP.SplitB, FP.FalseB) < JumpProb ||
   507:        MBPI->getEdgeProbability(FP.SplitB, FP.FalseB) > Prob))
   508:     return false;
   509: 
   510:   if (FP.TrueB && FP.FalseB) {
   511:     // Do not IfCovert if the branch is one sided.
   512:     if (MBPI) {
   513:       if (MBPI->getEdgeProbability(FP.SplitB, FP.TrueB) > Prob)
   514:         return false;
   515:       if (MBPI->getEdgeProbability(FP.SplitB, FP.FalseB) > Prob)
   516:         return false;
   517:     }
   518: 
   519:     // If both sides are predicable, convert them if they join, and the
   520:     // join block has no other predecessors.
   521:     MachineBasicBlock *TSB = *FP.TrueB->succ_begin();
   522:     MachineBasicBlock *FSB = *FP.FalseB->succ_begin();
   523:     if (TSB != FSB)
   524:       return false;
   525:     if (TSB->pred_size() != 2)
   526:       return false;
   527:   }
   528: 
   529:   // Calculate the total size of the predicated blocks.
   530:   // Assume instruction counts without branches to be the approximation of
   531:   // the code size. If the predicated blocks are smaller than a packet size,
   532:   // approximate the spare room in the packet that could be filled with the
   533:   // predicated/speculated instructions.
   534:   auto TotalCount = [] (const MachineBasicBlock *B, unsigned &Spare) {
   535:     if (!B)
   536:       return 0u;
   537:     unsigned T = std::count_if(B->begin(), B->getFirstTerminator(),
   538:                                [](const MachineInstr &MI) {
   539:                                  return !MI.isMetaInstruction();
   540:                                });
   541:     if (T < HEXAGON_PACKET_SIZE)
   542:       Spare += HEXAGON_PACKET_SIZE-T;
   543:     return T;
   544:   };
   545:   unsigned Spare = 0;
   546:   unsigned TotalIn = TotalCount(FP.TrueB, Spare) + TotalCount(FP.FalseB, Spare);
   547:   LLVM_DEBUG(
   548:       dbgs() << "Total number of instructions to be predicated/speculated: "
   549:              << TotalIn << ", spare room: " << Spare << "\n");
   550:   if (TotalIn >= SizeLimit+Spare)
   551:     return false;
   552: 
   553:   // Count the number of PHI nodes that will need to be updated (converted
   554:   // to MUX). Those can be later converted to predicated instructions, so
   555:   // they aren't always adding extra cost.
   556:   // KLUDGE: Also, count the number of predicate register definitions in
   557:   // each block. The scheduler may increase the pressure of these and cause
   558:   // expensive spills (e.g. bitmnp01).
   559:   unsigned TotalPh = 0;
   560:   unsigned PredDefs = countPredicateDefs(FP.SplitB);
   561:   if (FP.JoinB) {
   562:     TotalPh = computePhiCost(FP.JoinB, FP);
   563:     PredDefs += countPredicateDefs(FP.JoinB);
   564:   } else {
   565:     if (FP.TrueB && !FP.TrueB->succ_empty()) {
   566:       MachineBasicBlock *SB = *FP.TrueB->succ_begin();
   567:       TotalPh += computePhiCost(SB, FP);
   568:       PredDefs += countPredicateDefs(SB);
   569:     }
   570:     if (FP.FalseB && !FP.FalseB->succ_empty()) {
   571:       MachineBasicBlock *SB = *FP.FalseB->succ_begin();
   572:       TotalPh += computePhiCost(SB, FP);
   573:       PredDefs += countPredicateDefs(SB);
   574:     }
   575:   }
   576:   LLVM_DEBUG(dbgs() << "Total number of extra muxes from converted phis: "
   577:                     << TotalPh << "\n");
   578:   if (TotalIn+TotalPh >= SizeLimit+Spare)
   579:     return false;
   580: 
   581:   LLVM_DEBUG(dbgs() << "Total number of predicate registers: " << PredDefs
   582:                     << "\n");
   583:   if (PredDefs > 4)
   584:     return false;
   585: 
   586:   return true;
   587: }
   588: 
   589: bool HexagonEarlyIfConversion::visitBlock(MachineBasicBlock *B,
   590:       MachineLoop *L) {
   591:   bool Changed = false;
   592: 
   593:   // Visit all dominated blocks from the same loop first, then process B.
   594:   MachineDomTreeNode *N = MDT->getNode(B);
   595: 
   596:   // We will change CFG/DT during this traversal, so take precautions to
   597:   // avoid problems related to invalidated iterators. In fact, processing
   598:   // a child C of B cannot cause another child to be removed, but it can
   599:   // cause a new child to be added (which was a child of C before C itself
   600:   // was removed. This new child C, however, would have been processed
```
- EN: It declares or implements routines such as succ_begin, std::count_if, isMetaInstruction, TotalCount, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonEarlyIfConversion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 succ_begin, std::count_if, isMetaInstruction, TotalCount, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonEarlyIfConversion，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:   // prior to processing B, so there is no need to process it again.
   602:   // Simply keep a list of children of B, and traverse that list.
   603:   using DTNodeVectType = SmallVector<MachineDomTreeNode *, 4>;
   604:   DTNodeVectType Cn(llvm::children<MachineDomTreeNode *>(N));
   605:   for (auto &I : Cn) {
   606:     MachineBasicBlock *SB = I->getBlock();
   607:     if (!Deleted.count(SB))
   608:       Changed |= visitBlock(SB, L);
   609:   }
   610:   // When walking down the dominator tree, we want to traverse through
   611:   // blocks from nested (other) loops, because they can dominate blocks
   612:   // that are in L. Skip the non-L blocks only after the tree traversal.
   613:   if (MLI->getLoopFor(B) != L)
   614:     return Changed;
   615: 
   616:   FlowPattern FP;
   617:   if (!matchFlowPattern(B, L, FP))
   618:     return Changed;
   619: 
   620:   if (!isValid(FP)) {
   621:     LLVM_DEBUG(dbgs() << "Conversion is not valid\n");
   622:     return Changed;
   623:   }
   624:   if (!isProfitable(FP)) {
   625:     LLVM_DEBUG(dbgs() << "Conversion is not profitable\n");
   626:     return Changed;
   627:   }
   628: 
   629:   convert(FP);
   630:   simplifyFlowGraph(FP);
   631:   return true;
   632: }
   633: 
   634: bool HexagonEarlyIfConversion::visitLoop(MachineLoop *L) {
   635:   MachineBasicBlock *HB = L ? L->getHeader() : nullptr;
   636:   LLVM_DEBUG((L ? dbgs() << "Visiting loop H:" << PrintMB(HB)
   637:                 : dbgs() << "Visiting function")
   638:              << "\n");
   639:   bool Changed = false;
   640:   if (L) {
   641:     for (MachineLoop *I : *L)
   642:       Changed |= visitLoop(I);
   643:   }
   644: 
   645:   MachineBasicBlock *EntryB = GraphTraits<MachineFunction*>::getEntryNode(MFN);
   646:   Changed |= visitBlock(L ? HB : EntryB, L);
   647:   return Changed;
   648: }
   649: 
   650: bool HexagonEarlyIfConversion::isPredicableStore(const MachineInstr *MI)
   651:       const {
   652:   // HexagonInstrInfo::isPredicable will consider these stores are non-
   653:   // -predicable if the offset would become constant-extended after
   654:   // predication.
   655:   unsigned Opc = MI->getOpcode();
   656:   switch (Opc) {
   657:     case Hexagon::S2_storerb_io:
   658:     case Hexagon::S2_storerbnew_io:
   659:     case Hexagon::S2_storerh_io:
   660:     case Hexagon::S2_storerhnew_io:
   661:     case Hexagon::S2_storeri_io:
   662:     case Hexagon::S2_storerinew_io:
   663:     case Hexagon::S2_storerd_io:
   664:     case Hexagon::S4_storeirb_io:
   665:     case Hexagon::S4_storeirh_io:
   666:     case Hexagon::S4_storeiri_io:
   667:       return true;
   668:   }
   669: 
   670:   // TargetInstrInfo::isPredicable takes a non-const pointer.
   671:   return MI->mayStore() && HII->isPredicable(const_cast<MachineInstr&>(*MI));
   672: }
   673: 
   674: bool HexagonEarlyIfConversion::isSafeToSpeculate(const MachineInstr *MI)
   675:       const {
   676:   if (MI->mayLoadOrStore())
   677:     return false;
   678:   if (MI->isCall() || MI->isBarrier() || MI->isBranch())
   679:     return false;
   680:   if (MI->hasUnmodeledSideEffects())
   681:     return false;
   682:   if (MI->getOpcode() == TargetOpcode::LIFETIME_END)
   683:     return false;
   684: 
   685:   return true;
   686: }
   687: 
   688: bool HexagonEarlyIfConversion::isPredicate(unsigned R) const {
   689:   const TargetRegisterClass *RC = MRI->getRegClass(R);
   690:   return RC == &Hexagon::PredRegsRegClass ||
   691:          RC == &Hexagon::HvxQRRegClass;
   692: }
   693: 
   694: unsigned HexagonEarlyIfConversion::getCondStoreOpcode(unsigned Opc,
   695:       bool IfTrue) const {
   696:   return HII->getCondOpcode(Opc, !IfTrue);
   697: }
   698: 
   699: void HexagonEarlyIfConversion::predicateInstr(MachineBasicBlock *ToB,
   700:       MachineBasicBlock::iterator At, MachineInstr *MI,
```
- EN: It declares or implements routines such as Cn, getBlock, LLVM_DEBUG, convert, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonEarlyIfConversion, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 Cn, getBlock, LLVM_DEBUG, convert, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonEarlyIfConversion, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:       unsigned PredR, bool IfTrue) {
   702:   DebugLoc DL;
   703:   if (At != ToB->end())
   704:     DL = At->getDebugLoc();
   705:   else if (!ToB->empty())
   706:     DL = ToB->back().getDebugLoc();
   707: 
   708:   unsigned Opc = MI->getOpcode();
   709: 
   710:   if (isPredicableStore(MI)) {
   711:     unsigned COpc = getCondStoreOpcode(Opc, IfTrue);
   712:     assert(COpc);
   713:     MachineInstrBuilder MIB = BuildMI(*ToB, At, DL, HII->get(COpc));
   714:     MachineInstr::mop_iterator MOI = MI->operands_begin();
   715:     if (HII->isPostIncrement(*MI)) {
   716:       MIB.add(*MOI);
   717:       ++MOI;
   718:     }
   719:     MIB.addReg(PredR);
   720:     for (const MachineOperand &MO : make_range(MOI, MI->operands_end()))
   721:       MIB.add(MO);
   722: 
   723:     // Set memory references.
   724:     MIB.cloneMemRefs(*MI);
   725: 
   726:     MI->eraseFromParent();
   727:     return;
   728:   }
   729: 
   730:   if (Opc == Hexagon::J2_jump) {
   731:     MachineBasicBlock *TB = MI->getOperand(0).getMBB();
   732:     const MCInstrDesc &D = HII->get(IfTrue ? Hexagon::J2_jumpt
   733:                                            : Hexagon::J2_jumpf);
   734:     BuildMI(*ToB, At, DL, D)
   735:       .addReg(PredR)
   736:       .addMBB(TB);
   737:     MI->eraseFromParent();
   738:     return;
   739:   }
   740: 
   741:   // Print the offending instruction unconditionally as we are about to
   742:   // abort.
   743:   dbgs() << *MI;
   744:   llvm_unreachable("Unexpected instruction");
   745: }
   746: 
   747: // Predicate/speculate non-branch instructions from FromB into block ToB.
   748: // Leave the branches alone, they will be handled later. Btw, at this point
   749: // FromB should have at most one branch, and it should be unconditional.
   750: void HexagonEarlyIfConversion::predicateBlockNB(MachineBasicBlock *ToB,
   751:       MachineBasicBlock::iterator At, MachineBasicBlock *FromB,
   752:       unsigned PredR, bool IfTrue) {
   753:   LLVM_DEBUG(dbgs() << "Predicating block " << PrintMB(FromB) << "\n");
   754:   MachineBasicBlock::iterator End = FromB->getFirstTerminator();
   755:   MachineBasicBlock::iterator I, NextI;
   756: 
   757:   for (I = FromB->begin(); I != End; I = NextI) {
   758:     assert(!I->isPHI());
   759:     NextI = std::next(I);
   760:     if (isSafeToSpeculate(&*I))
   761:       ToB->splice(At, FromB, I);
   762:     else
   763:       predicateInstr(ToB, At, &*I, PredR, IfTrue);
   764:   }
   765: }
   766: 
   767: unsigned HexagonEarlyIfConversion::buildMux(MachineBasicBlock *B,
   768:       MachineBasicBlock::iterator At, const TargetRegisterClass *DRC,
   769:       unsigned PredR, unsigned TR, unsigned TSR, unsigned FR, unsigned FSR) {
   770:   unsigned Opc = 0;
   771:   switch (DRC->getID()) {
   772:     case Hexagon::IntRegsRegClassID:
   773:     case Hexagon::IntRegsLow8RegClassID:
   774:       Opc = Hexagon::C2_mux;
   775:       break;
   776:     case Hexagon::DoubleRegsRegClassID:
   777:     case Hexagon::GeneralDoubleLow8RegsRegClassID:
   778:       Opc = Hexagon::PS_pselect;
   779:       break;
   780:     case Hexagon::HvxVRRegClassID:
   781:       Opc = Hexagon::PS_vselect;
   782:       break;
   783:     case Hexagon::HvxWRRegClassID:
   784:       Opc = Hexagon::PS_wselect;
   785:       break;
   786:     default:
   787:       llvm_unreachable("unexpected register type");
   788:   }
   789:   const MCInstrDesc &D = HII->get(Opc);
   790: 
   791:   DebugLoc DL = B->findBranchDebugLoc();
   792:   Register MuxR = MRI->createVirtualRegister(DRC);
   793:   BuildMI(*B, At, DL, D, MuxR)
   794:       .addReg(PredR)
   795:       .addReg(TR, {}, TSR)
   796:       .addReg(FR, {}, FSR);
   797:   return MuxR;
   798: }
   799: 
   800: void HexagonEarlyIfConversion::updatePhiNodes(MachineBasicBlock *WhereB,
```
- EN: It declares or implements routines such as getOpcode, getCondStoreOpcode, assert, BuildMI, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonEarlyIfConversion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOpcode, getCondStoreOpcode, assert, BuildMI, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonEarlyIfConversion，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801:       const FlowPattern &FP) {
   802:   // Visit all PHI nodes in the WhereB block and generate MUX instructions
   803:   // in the split block. Update the PHI nodes with the values of the MUX.
   804:   auto NonPHI = WhereB->getFirstNonPHI();
   805:   for (auto I = WhereB->begin(); I != NonPHI; ++I) {
   806:     MachineInstr *PN = &*I;
   807:     // Registers and subregisters corresponding to TrueB, FalseB and SplitB.
   808:     unsigned TR = 0, TSR = 0, FR = 0, FSR = 0, SR = 0, SSR = 0;
   809:     for (int i = PN->getNumOperands()-2; i > 0; i -= 2) {
   810:       const MachineOperand &RO = PN->getOperand(i), &BO = PN->getOperand(i+1);
   811:       if (BO.getMBB() == FP.SplitB)
   812:         SR = RO.getReg(), SSR = RO.getSubReg();
   813:       else if (BO.getMBB() == FP.TrueB)
   814:         TR = RO.getReg(), TSR = RO.getSubReg();
   815:       else if (BO.getMBB() == FP.FalseB)
   816:         FR = RO.getReg(), FSR = RO.getSubReg();
   817:       else
   818:         continue;
   819:       PN->removeOperand(i+1);
   820:       PN->removeOperand(i);
   821:     }
   822:     if (TR == 0)
   823:       TR = SR, TSR = SSR;
   824:     else if (FR == 0)
   825:       FR = SR, FSR = SSR;
   826: 
   827:     assert(TR || FR);
   828:     unsigned MuxR = 0, MuxSR = 0;
   829: 
   830:     if (TR && FR) {
   831:       Register DR = PN->getOperand(0).getReg();
   832:       const TargetRegisterClass *RC = MRI->getRegClass(DR);
   833:       MuxR = buildMux(FP.SplitB, FP.SplitB->getFirstTerminator(), RC,
   834:                       FP.PredR, TR, TSR, FR, FSR);
   835:     } else if (TR) {
   836:       MuxR = TR;
   837:       MuxSR = TSR;
   838:     } else {
   839:       MuxR = FR;
   840:       MuxSR = FSR;
   841:     }
   842: 
   843:     PN->addOperand(MachineOperand::CreateReg(MuxR, false, false, false, false,
   844:                                              false, false, MuxSR));
   845:     PN->addOperand(MachineOperand::CreateMBB(FP.SplitB));
   846:   }
   847: }
   848: 
   849: void HexagonEarlyIfConversion::convert(const FlowPattern &FP) {
   850:   MachineBasicBlock *TSB = nullptr, *FSB = nullptr;
   851:   MachineBasicBlock::iterator OldTI = FP.SplitB->getFirstTerminator();
   852:   assert(OldTI != FP.SplitB->end());
   853:   DebugLoc DL = OldTI->getDebugLoc();
   854: 
   855:   if (FP.TrueB) {
   856:     TSB = *FP.TrueB->succ_begin();
   857:     predicateBlockNB(FP.SplitB, OldTI, FP.TrueB, FP.PredR, true);
   858:   }
   859:   if (FP.FalseB) {
   860:     FSB = *FP.FalseB->succ_begin();
   861:     MachineBasicBlock::iterator At = FP.SplitB->getFirstTerminator();
   862:     predicateBlockNB(FP.SplitB, At, FP.FalseB, FP.PredR, false);
   863:   }
   864: 
   865:   // Regenerate new terminators in the split block and update the successors.
   866:   // First, remember any information that may be needed later and remove the
   867:   // existing terminators/successors from the split block.
   868:   MachineBasicBlock *SSB = nullptr;
   869:   FP.SplitB->erase(OldTI, FP.SplitB->end());
   870:   while (!FP.SplitB->succ_empty()) {
   871:     MachineBasicBlock *T = *FP.SplitB->succ_begin();
   872:     // It's possible that the split block had a successor that is not a pre-
   873:     // dicated block. This could only happen if there was only one block to
   874:     // be predicated. Example:
   875:     //   split_b:
   876:     //     if (p) jump true_b
   877:     //     jump unrelated2_b
   878:     //   unrelated1_b:
   879:     //     ...
   880:     //   unrelated2_b:  ; can have other predecessors, so it's not "false_b"
   881:     //     jump other_b
   882:     //   true_b:        ; only reachable from split_b, can be predicated
   883:     //     ...
   884:     //
   885:     // Find this successor (SSB) if it exists.
   886:     if (T != FP.TrueB && T != FP.FalseB) {
   887:       assert(!SSB);
   888:       SSB = T;
   889:     }
   890:     FP.SplitB->removeSuccessor(FP.SplitB->succ_begin());
   891:   }
   892: 
   893:   // Insert new branches and update the successors of the split block. This
   894:   // may create unconditional branches to the layout successor, etc., but
   895:   // that will be cleaned up later. For now, make sure that correct code is
   896:   // generated.
   897:   if (FP.JoinB) {
   898:     assert(!SSB || SSB == FP.JoinB);
   899:     BuildMI(*FP.SplitB, FP.SplitB->end(), DL, HII->get(Hexagon::J2_jump))
   900:       .addMBB(FP.JoinB);
```
- EN: It declares or implements routines such as getFirstNonPHI, getOperand, removeOperand, assert, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonEarlyIfConversion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getFirstNonPHI, getOperand, removeOperand, assert, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonEarlyIfConversion，说明了它与同级后端组件的连接关系。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:     FP.SplitB->addSuccessor(FP.JoinB);
   902:   } else {
   903:     bool HasBranch = false;
   904:     if (TSB) {
   905:       BuildMI(*FP.SplitB, FP.SplitB->end(), DL, HII->get(Hexagon::J2_jumpt))
   906:         .addReg(FP.PredR)
   907:         .addMBB(TSB);
   908:       FP.SplitB->addSuccessor(TSB);
   909:       HasBranch = true;
   910:     }
   911:     if (FSB) {
   912:       const MCInstrDesc &D = HasBranch ? HII->get(Hexagon::J2_jump)
   913:                                        : HII->get(Hexagon::J2_jumpf);
   914:       MachineInstrBuilder MIB = BuildMI(*FP.SplitB, FP.SplitB->end(), DL, D);
   915:       if (!HasBranch)
   916:         MIB.addReg(FP.PredR);
   917:       MIB.addMBB(FSB);
   918:       FP.SplitB->addSuccessor(FSB);
   919:     }
   920:     if (SSB) {
   921:       // This cannot happen if both TSB and FSB are set. [TF]SB are the
   922:       // successor blocks of the TrueB and FalseB (or null of the TrueB
   923:       // or FalseB block is null). SSB is the potential successor block
   924:       // of the SplitB that is neither TrueB nor FalseB.
   925:       BuildMI(*FP.SplitB, FP.SplitB->end(), DL, HII->get(Hexagon::J2_jump))
   926:         .addMBB(SSB);
   927:       FP.SplitB->addSuccessor(SSB);
   928:     }
   929:   }
   930: 
   931:   // What is left to do is to update the PHI nodes that could have entries
   932:   // referring to predicated blocks.
   933:   if (FP.JoinB) {
   934:     updatePhiNodes(FP.JoinB, FP);
   935:   } else {
   936:     if (TSB)
   937:       updatePhiNodes(TSB, FP);
   938:     if (FSB)
   939:       updatePhiNodes(FSB, FP);
   940:     // Nothing to update in SSB, since SSB's predecessors haven't changed.
   941:   }
   942: }
   943: 
   944: void HexagonEarlyIfConversion::removeBlock(MachineBasicBlock *B) {
   945:   LLVM_DEBUG(dbgs() << "Removing block " << PrintMB(B) << "\n");
   946: 
   947:   // Transfer the immediate dominator information from B to its descendants.
   948:   MachineDomTreeNode *N = MDT->getNode(B);
   949:   MachineDomTreeNode *IDN = N->getIDom();
   950:   if (IDN) {
   951:     MachineBasicBlock *IDB = IDN->getBlock();
   952: 
   953:     using GTN = GraphTraits<MachineDomTreeNode *>;
   954:     using DTNodeVectType = SmallVector<MachineDomTreeNode *, 4>;
   955: 
   956:     DTNodeVectType Cn(GTN::child_begin(N), GTN::child_end(N));
   957:     for (auto &I : Cn) {
   958:       MachineBasicBlock *SB = I->getBlock();
   959:       MDT->changeImmediateDominator(SB, IDB);
   960:     }
   961:   }
   962: 
   963:   while (!B->succ_empty())
   964:     B->removeSuccessor(B->succ_begin());
   965: 
   966:   for (MachineBasicBlock *Pred : B->predecessors())
   967:     Pred->removeSuccessor(B, true);
   968: 
   969:   Deleted.insert(B);
   970:   MDT->eraseNode(B);
   971:   MFN->erase(B->getIterator());
   972: }
   973: 
   974: void HexagonEarlyIfConversion::eliminatePhis(MachineBasicBlock *B) {
   975:   LLVM_DEBUG(dbgs() << "Removing phi nodes from block " << PrintMB(B) << "\n");
   976:   MachineBasicBlock::iterator I, NextI, NonPHI = B->getFirstNonPHI();
   977:   for (I = B->begin(); I != NonPHI; I = NextI) {
   978:     NextI = std::next(I);
   979:     MachineInstr *PN = &*I;
   980:     assert(PN->getNumOperands() == 3 && "Invalid phi node");
   981:     MachineOperand &UO = PN->getOperand(1);
   982:     Register UseR = UO.getReg(), UseSR = UO.getSubReg();
   983:     Register DefR = PN->getOperand(0).getReg();
   984:     unsigned NewR = UseR;
   985:     if (UseSR) {
   986:       // MRI.replaceVregUsesWith does not allow to update the subregister,
   987:       // so instead of doing the use-iteration here, create a copy into a
   988:       // "non-subregistered" register.
   989:       const DebugLoc &DL = PN->getDebugLoc();
   990:       const TargetRegisterClass *RC = MRI->getRegClass(DefR);
   991:       NewR = MRI->createVirtualRegister(RC);
   992:       NonPHI = BuildMI(*B, NonPHI, DL, HII->get(TargetOpcode::COPY), NewR)
   993:                    .addReg(UseR, {}, UseSR);
   994:     }
   995:     MRI->replaceRegWith(DefR, NewR);
   996:     B->erase(I);
   997:   }
   998: }
   999: 
  1000: void HexagonEarlyIfConversion::mergeBlocks(MachineBasicBlock *PredB,
```
- EN: It declares or implements routines such as addSuccessor, BuildMI, get, addMBB, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonEarlyIfConversion, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addSuccessor, BuildMI, get, addMBB, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonEarlyIfConversion，说明了它与同级后端组件的连接关系。

### Lines 1001-1070 / 第 1001-1070 行

```cpp
  1001:       MachineBasicBlock *SuccB) {
  1002:   LLVM_DEBUG(dbgs() << "Merging blocks " << PrintMB(PredB) << " and "
  1003:                     << PrintMB(SuccB) << "\n");
  1004:   bool TermOk = hasUncondBranch(SuccB);
  1005:   eliminatePhis(SuccB);
  1006:   HII->removeBranch(*PredB);
  1007:   PredB->removeSuccessor(SuccB);
  1008:   PredB->splice(PredB->end(), SuccB, SuccB->begin(), SuccB->end());
  1009:   PredB->transferSuccessorsAndUpdatePHIs(SuccB);
  1010:   MachineBasicBlock *OldLayoutSuccessor = SuccB->getNextNode();
  1011:   removeBlock(SuccB);
  1012:   if (!TermOk)
  1013:     PredB->updateTerminator(OldLayoutSuccessor);
  1014: }
  1015: 
  1016: void HexagonEarlyIfConversion::simplifyFlowGraph(const FlowPattern &FP) {
  1017:   MachineBasicBlock *OldLayoutSuccessor = FP.SplitB->getNextNode();
  1018:   if (FP.TrueB)
  1019:     removeBlock(FP.TrueB);
  1020:   if (FP.FalseB)
  1021:     removeBlock(FP.FalseB);
  1022: 
  1023:   FP.SplitB->updateTerminator(OldLayoutSuccessor);
  1024:   if (FP.SplitB->succ_size() != 1)
  1025:     return;
  1026: 
  1027:   MachineBasicBlock *SB = *FP.SplitB->succ_begin();
  1028:   if (SB->pred_size() != 1)
  1029:     return;
  1030: 
  1031:   // By now, the split block has only one successor (SB), and SB has only
  1032:   // one predecessor. We can try to merge them. We will need to update ter-
  1033:   // minators in FP.Split+SB, and that requires working analyzeBranch, which
  1034:   // fails on Hexagon for blocks that have EH_LABELs. However, if SB ends
  1035:   // with an unconditional branch, we won't need to touch the terminators.
  1036:   if (!hasEHLabel(SB) || hasUncondBranch(SB))
  1037:     mergeBlocks(FP.SplitB, SB);
  1038: }
  1039: 
  1040: bool HexagonEarlyIfConversion::runOnMachineFunction(MachineFunction &MF) {
  1041:   if (skipFunction(MF.getFunction()))
  1042:     return false;
  1043: 
  1044:   auto &ST = MF.getSubtarget<HexagonSubtarget>();
  1045:   HII = ST.getInstrInfo();
  1046:   TRI = ST.getRegisterInfo();
  1047:   MFN = &MF;
  1048:   MRI = &MF.getRegInfo();
  1049:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  1050:   MLI = &getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  1051:   MBPI = EnableHexagonBP
  1052:              ? &getAnalysis<MachineBranchProbabilityInfoWrapperPass>().getMBPI()
  1053:              : nullptr;
  1054: 
  1055:   Deleted.clear();
  1056:   bool Changed = false;
  1057: 
  1058:   for (MachineLoop *L : *MLI)
  1059:     Changed |= visitLoop(L);
  1060:   Changed |= visitLoop(nullptr);
  1061: 
  1062:   return Changed;
  1063: }
  1064: 
  1065: //===----------------------------------------------------------------------===//
  1066: //                         Public Constructor Functions
  1067: //===----------------------------------------------------------------------===//
  1068: FunctionPass *llvm::createHexagonEarlyIfConversion() {
  1069:   return new HexagonEarlyIfConversion();
  1070: }
```
- EN: It declares or implements routines such as LLVM_DEBUG, hasUncondBranch, eliminatePhis, removeBranch, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonEarlyIfConversion, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, hasUncondBranch, eliminatePhis, removeBranch, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonEarlyIfConversion, HexagonSubtarget，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseSet.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/ADT/iterator_range.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineBranchProbabilityInfo.h, llvm/CodeGen/MachineDominators.h, ... (28 total)`
- Hexagon symbols / Hexagon 符号: `HexagonEarlyIfConv, HexagonInstrInfo, HexagonSubtarget, HexagonEarlyIfConversion`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
