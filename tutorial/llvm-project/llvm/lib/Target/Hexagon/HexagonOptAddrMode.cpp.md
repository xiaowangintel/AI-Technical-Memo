# HexagonOptAddrMode.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonOptAddrMode.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon addressing-mode optimization.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonOptAddrMode.cpp ---------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // This implements a Hexagon-specific pass to optimize addressing mode for
     9: // load/store instructions.
    10: //===----------------------------------------------------------------------===//
    11: 
    12: #include "Hexagon.h"
    13: #include "HexagonInstrInfo.h"
    14: #include "HexagonSubtarget.h"
    15: #include "MCTargetDesc/HexagonBaseInfo.h"
    16: #include "llvm/ADT/DenseMap.h"
    17: #include "llvm/ADT/DenseSet.h"
    18: #include "llvm/ADT/StringRef.h"
    19: #include "llvm/CodeGen/MachineBasicBlock.h"
    20: #include "llvm/CodeGen/MachineDominanceFrontier.h"
    21: #include "llvm/CodeGen/MachineDominators.h"
    22: #include "llvm/CodeGen/MachineFunction.h"
    23: #include "llvm/CodeGen/MachineFunctionPass.h"
    24: #include "llvm/CodeGen/MachineInstr.h"
    25: #include "llvm/CodeGen/MachineInstrBuilder.h"
    26: #include "llvm/CodeGen/MachineOperand.h"
    27: #include "llvm/CodeGen/MachineRegisterInfo.h"
    28: #include "llvm/CodeGen/RDFGraph.h"
    29: #include "llvm/CodeGen/RDFLiveness.h"
    30: #include "llvm/CodeGen/RDFRegisters.h"
    31: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    32: #include "llvm/InitializePasses.h"
    33: #include "llvm/MC/MCInstrDesc.h"
    34: #include "llvm/Pass.h"
    35: #include "llvm/Support/CommandLine.h"
    36: #include "llvm/Support/Debug.h"
    37: #include "llvm/Support/ErrorHandling.h"
    38: #include "llvm/Support/raw_ostream.h"
    39: #include <cassert>
    40: #include <cstdint>
    41: 
    42: #define DEBUG_TYPE "opt-addr-mode"
    43: 
    44: using namespace llvm;
    45: using namespace rdf;
    46: 
    47: static cl::opt<int> CodeGrowthLimit("hexagon-amode-growth-limit",
    48:   cl::Hidden, cl::init(0), cl::desc("Code growth limit for address mode "
    49:   "optimization"));
    50: 
    51: extern cl::opt<unsigned> RDFFuncBlockLimit;
    52: 
    53: namespace {
    54: 
    55: class HexagonOptAddrMode : public MachineFunctionPass {
    56: public:
    57:   static char ID;
    58: 
    59:   HexagonOptAddrMode() : MachineFunctionPass(ID) {}
    60: 
    61:   StringRef getPassName() const override {
    62:     return "Optimize addressing mode of load/store";
    63:   }
    64: 
    65:   void getAnalysisUsage(AnalysisUsage &AU) const override {
    66:     MachineFunctionPass::getAnalysisUsage(AU);
    67:     AU.addRequired<MachineDominatorTreeWrapperPass>();
    68:     AU.addRequired<MachineDominanceFrontierWrapperPass>();
    69:     AU.setPreservesAll();
    70:   }
    71: 
    72:   bool runOnMachineFunction(MachineFunction &MF) override;
    73: 
    74: private:
    75:   using MISetType = DenseSet<MachineInstr *>;
    76:   using InstrEvalMap = DenseMap<MachineInstr *, bool>;
    77:   DenseSet<MachineInstr *> ProcessedAddiInsts;
    78: 
    79:   MachineRegisterInfo *MRI = nullptr;
    80:   const TargetRegisterInfo *TRI = nullptr;
    81:   const HexagonInstrInfo *HII = nullptr;
    82:   const HexagonRegisterInfo *HRI = nullptr;
    83:   MachineDominatorTree *MDT = nullptr;
    84:   DataFlowGraph *DFG = nullptr;
    85:   DataFlowGraph::DefStackMap DefM;
    86:   Liveness *LV = nullptr;
    87:   MISetType Deleted;
    88: 
    89:   bool processBlock(NodeAddr<BlockNode *> BA);
    90:   bool xformUseMI(MachineInstr *TfrMI, MachineInstr *UseMI,
    91:                   NodeAddr<UseNode *> UseN, unsigned UseMOnum);
    92:   bool processAddBases(NodeAddr<StmtNode *> AddSN, MachineInstr *AddMI);
    93:   bool usedInLoadStore(NodeAddr<StmtNode *> CurrentInstSN, int64_t NewOffset);
    94:   bool findFirstReachedInst(
    95:       MachineInstr *AddMI,
    96:       std::vector<std::pair<NodeAddr<StmtNode *>, NodeAddr<UseNode *>>>
    97:           &AddiList,
    98:       NodeAddr<StmtNode *> &UseSN);
    99:   bool updateAddBases(MachineInstr *CurrentMI, MachineInstr *FirstReachedMI,
   100:                       int64_t NewOffset);
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, MCTargetDesc/HexagonBaseInfo.h, ... (29 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, rdf) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonOptAddrMode, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, MCTargetDesc/HexagonBaseInfo.h, ... (29 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, rdf），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonOptAddrMode 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:   bool processAddUses(NodeAddr<StmtNode *> AddSN, MachineInstr *AddMI,
   102:                       const NodeList &UNodeList);
   103:   bool updateAddUses(MachineInstr *AddMI, MachineInstr *UseMI);
   104:   bool analyzeUses(unsigned DefR, const NodeList &UNodeList,
   105:                    InstrEvalMap &InstrEvalResult, short &SizeInc);
   106:   bool hasRepForm(MachineInstr &MI, unsigned TfrDefR);
   107:   bool canRemoveAddasl(NodeAddr<StmtNode *> AddAslSN, MachineInstr &MI,
   108:                        const NodeList &UNodeList);
   109:   bool isSafeToExtLR(NodeAddr<StmtNode *> SN, MachineInstr *MI,
   110:                      Register LRExtReg, const NodeList &UNodeList);
   111:   void getAllRealUses(NodeAddr<StmtNode *> SN, NodeList &UNodeList);
   112:   bool allValidCandidates(NodeAddr<StmtNode *> SA, NodeList &UNodeList);
   113:   short getBaseWithLongOffset(const MachineInstr &MI) const;
   114:   bool changeStore(MachineInstr *OldMI, MachineOperand ImmOp,
   115:                    unsigned ImmOpNum);
   116:   bool changeLoad(MachineInstr *OldMI, MachineOperand ImmOp, unsigned ImmOpNum);
   117:   bool changeAddAsl(NodeAddr<UseNode *> AddAslUN, MachineInstr *AddAslMI,
   118:                     const MachineOperand &ImmOp, unsigned ImmOpNum);
   119:   bool isValidOffset(MachineInstr *MI, int Offset);
   120:   unsigned getBaseOpPosition(MachineInstr *MI);
   121:   unsigned getOffsetOpPosition(MachineInstr *MI);
   122: };
   123: 
   124: } // end anonymous namespace
   125: 
   126: char HexagonOptAddrMode::ID = 0;
   127: 
   128: INITIALIZE_PASS_BEGIN(HexagonOptAddrMode, "amode-opt",
   129:                       "Optimize addressing mode", false, false)
   130: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   131: INITIALIZE_PASS_DEPENDENCY(MachineDominanceFrontierWrapperPass)
   132: INITIALIZE_PASS_END(HexagonOptAddrMode, "amode-opt", "Optimize addressing mode",
   133:                     false, false)
   134: 
   135: bool HexagonOptAddrMode::hasRepForm(MachineInstr &MI, unsigned TfrDefR) {
   136:   const MCInstrDesc &MID = MI.getDesc();
   137: 
   138:   if ((!MID.mayStore() && !MID.mayLoad()) || HII->isPredicated(MI))
   139:     return false;
   140: 
   141:   if (MID.mayStore()) {
   142:     MachineOperand StOp = MI.getOperand(MI.getNumOperands() - 1);
   143:     if (StOp.isReg() && StOp.getReg() == TfrDefR)
   144:       return false;
   145:   }
   146: 
   147:   if (HII->getAddrMode(MI) == HexagonII::BaseRegOffset)
   148:     // Transform to Absolute plus register offset.
   149:     return (HII->changeAddrMode_rr_ur(MI) >= 0);
   150:   else if (HII->getAddrMode(MI) == HexagonII::BaseImmOffset)
   151:     // Transform to absolute addressing mode.
   152:     return (HII->changeAddrMode_io_abs(MI) >= 0);
   153: 
   154:   return false;
   155: }
   156: 
   157: // Check if addasl instruction can be removed. This is possible only
   158: // if it's feeding to only load/store instructions with base + register
   159: // offset as these instruction can be transformed to use 'absolute plus
   160: // shifted register offset'.
   161: // ex:
   162: // Rs = ##foo
   163: // Rx = addasl(Rs, Rt, #2)
   164: // Rd = memw(Rx + #28)
   165: // Above three instructions can be replaced with Rd = memw(Rt<<#2 + ##foo+28)
   166: 
   167: bool HexagonOptAddrMode::canRemoveAddasl(NodeAddr<StmtNode *> AddAslSN,
   168:                                          MachineInstr &MI,
   169:                                          const NodeList &UNodeList) {
   170:   // check offset size in addasl. if 'offset > 3' return false
   171:   const MachineOperand &OffsetOp = MI.getOperand(3);
   172:   if (!OffsetOp.isImm() || OffsetOp.getImm() > 3)
   173:     return false;
   174: 
   175:   Register OffsetReg = MI.getOperand(2).getReg();
   176:   RegisterRef OffsetRR;
   177:   NodeId OffsetRegRD = 0;
   178:   for (NodeAddr<UseNode *> UA : AddAslSN.Addr->members_if(DFG->IsUse, *DFG)) {
   179:     RegisterRef RR = UA.Addr->getRegRef(*DFG);
   180:     if (OffsetReg == RR.asMCReg()) {
   181:       OffsetRR = RR;
   182:       OffsetRegRD = UA.Addr->getReachingDef();
   183:     }
   184:   }
   185: 
   186:   for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
   187:     NodeAddr<UseNode *> UA = *I;
   188:     NodeAddr<InstrNode *> IA = UA.Addr->getOwner(*DFG);
   189:     if (UA.Addr->getFlags() & NodeAttrs::PhiRef)
   190:       return false;
   191:     NodeAddr<RefNode*> AA = LV->getNearestAliasedRef(OffsetRR, IA);
   192:     if ((DFG->IsDef(AA) && AA.Id != OffsetRegRD) ||
   193:          AA.Addr->getReachingDef() != OffsetRegRD)
   194:       return false;
   195: 
   196:     MachineInstr &UseMI = *NodeAddr<StmtNode *>(IA).Addr->getCode();
   197:     NodeAddr<DefNode *> OffsetRegDN = DFG->addr<DefNode *>(OffsetRegRD);
   198:     // Reaching Def to an offset register can't be a phi.
   199:     if ((OffsetRegDN.Addr->getFlags() & NodeAttrs::PhiRef) &&
   200:         MI.getParent() != UseMI.getParent())
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as processAddUses, updateAddUses, analyzeUses, hasRepForm, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptAddrMode, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 processAddUses, updateAddUses, analyzeUses, hasRepForm, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201:       return false;
   202: 
   203:     const MCInstrDesc &UseMID = UseMI.getDesc();
   204:     if ((!UseMID.mayLoad() && !UseMID.mayStore()) ||
   205:         HII->getAddrMode(UseMI) != HexagonII::BaseImmOffset ||
   206:         getBaseWithLongOffset(UseMI) < 0)
   207:       return false;
   208: 
   209:     // Addasl output can't be a store value.
   210:     if (UseMID.mayStore() && UseMI.getOperand(2).isReg() &&
   211:         UseMI.getOperand(2).getReg() == MI.getOperand(0).getReg())
   212:       return false;
   213: 
   214:     for (auto &Mo : UseMI.operands())
   215:       // Is it a frame index?
   216:       if (Mo.isFI())
   217:         return false;
   218:     // Is the OffsetReg definition actually reaches UseMI?
   219:     if (!UseMI.getParent()->isLiveIn(OffsetReg) &&
   220:         MI.getParent() != UseMI.getParent()) {
   221:       LLVM_DEBUG(dbgs() << "  The offset reg " << printReg(OffsetReg, TRI)
   222:                         << " is NOT live in to MBB "
   223:                         << UseMI.getParent()->getName() << "\n");
   224:       return false;
   225:     }
   226:   }
   227:   return true;
   228: }
   229: 
   230: bool HexagonOptAddrMode::allValidCandidates(NodeAddr<StmtNode *> SA,
   231:                                             NodeList &UNodeList) {
   232:   for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
   233:     NodeAddr<UseNode *> UN = *I;
   234:     RegisterRef UR = UN.Addr->getRegRef(*DFG);
   235:     NodeSet Visited, Defs;
   236:     const auto &P = LV->getAllReachingDefsRec(UR, UN, Visited, Defs);
   237:     if (!P.second) {
   238:       LLVM_DEBUG({
   239:         dbgs() << "*** Unable to collect all reaching defs for use ***\n"
   240:                << PrintNode<UseNode*>(UN, *DFG) << '\n'
   241:                << "The program's complexity may exceed the limits.\n";
   242:       });
   243:       return false;
   244:     }
   245:     const auto &ReachingDefs = P.first;
   246:     if (ReachingDefs.size() > 1) {
   247:       LLVM_DEBUG({
   248:         dbgs() << "*** Multiple Reaching Defs found!!! ***\n";
   249:         for (auto DI : ReachingDefs) {
   250:           NodeAddr<UseNode *> DA = DFG->addr<UseNode *>(DI);
   251:           NodeAddr<StmtNode *> TempIA = DA.Addr->getOwner(*DFG);
   252:           dbgs() << "\t\t[Reaching Def]: "
   253:                  << Print<NodeAddr<InstrNode *>>(TempIA, *DFG) << "\n";
   254:         }
   255:       });
   256:       return false;
   257:     }
   258:   }
   259:   return true;
   260: }
   261: 
   262: void HexagonOptAddrMode::getAllRealUses(NodeAddr<StmtNode *> SA,
   263:                                         NodeList &UNodeList) {
   264:   for (NodeAddr<DefNode *> DA : SA.Addr->members_if(DFG->IsDef, *DFG)) {
   265:     LLVM_DEBUG(dbgs() << "\t\t[DefNode]: "
   266:                       << Print<NodeAddr<DefNode *>>(DA, *DFG) << "\n");
   267:     RegisterRef DR = DA.Addr->getRegRef(*DFG);
   268: 
   269:     auto UseSet = LV->getAllReachedUses(DR, DA);
   270: 
   271:     for (auto UI : UseSet) {
   272:       NodeAddr<UseNode *> UA = DFG->addr<UseNode *>(UI);
   273:       LLVM_DEBUG({
   274:         NodeAddr<StmtNode *> TempIA = UA.Addr->getOwner(*DFG);
   275:         dbgs() << "\t\t\t[Reached Use]: "
   276:                << Print<NodeAddr<InstrNode *>>(TempIA, *DFG) << "\n";
   277:       });
   278: 
   279:       if (UA.Addr->getFlags() & NodeAttrs::PhiRef) {
   280:         NodeAddr<PhiNode *> PA = UA.Addr->getOwner(*DFG);
   281:         NodeId id = PA.Id;
   282:         const Liveness::RefMap &phiUse = LV->getRealUses(id);
   283:         LLVM_DEBUG(dbgs() << "\t\t\t\tphi real Uses"
   284:                           << Print<Liveness::RefMap>(phiUse, *DFG) << "\n");
   285:         if (!phiUse.empty()) {
   286:           for (auto I : phiUse) {
   287:             if (!DFG->getPRI().alias(RegisterRef(I.first), DR))
   288:               continue;
   289:             auto phiUseSet = I.second;
   290:             for (auto phiUI : phiUseSet) {
   291:               NodeAddr<UseNode *> phiUA = DFG->addr<UseNode *>(phiUI.first);
   292:               UNodeList.push_back(phiUA);
   293:             }
   294:           }
   295:         }
   296:       } else
   297:         UNodeList.push_back(UA);
   298:     }
   299:   }
   300: }
```
- EN: It declares or implements routines such as getDesc, LLVM_DEBUG, HexagonOptAddrMode::allValidCandidates, getRegRef, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonOptAddrMode, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getDesc, LLVM_DEBUG, HexagonOptAddrMode::allValidCandidates, getRegRef, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonOptAddrMode，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301: 
   302: bool HexagonOptAddrMode::isSafeToExtLR(NodeAddr<StmtNode *> SN,
   303:                                        MachineInstr *MI, Register LRExtReg,
   304:                                        const NodeList &UNodeList) {
   305:   RegisterRef LRExtRR;
   306:   NodeId LRExtRegRD = 0;
   307:   // Iterate through all the UseNodes in SN and find the reaching def
   308:   // for the LRExtReg.
   309:   for (NodeAddr<UseNode *> UA : SN.Addr->members_if(DFG->IsUse, *DFG)) {
   310:     RegisterRef RR = UA.Addr->getRegRef(*DFG);
   311:     if (LRExtReg == RR.asMCReg()) {
   312:       LRExtRR = RR;
   313:       LRExtRegRD = UA.Addr->getReachingDef();
   314:     }
   315:   }
   316: 
   317:   for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
   318:     NodeAddr<UseNode *> UA = *I;
   319:     NodeAddr<InstrNode *> IA = UA.Addr->getOwner(*DFG);
   320:     // The reaching def of LRExtRR at load/store node should be same as the
   321:     // one reaching at the SN.
   322:     if (UA.Addr->getFlags() & NodeAttrs::PhiRef)
   323:       return false;
   324:     NodeAddr<RefNode*> AA = LV->getNearestAliasedRef(LRExtRR, IA);
   325:     if ((DFG->IsDef(AA) && AA.Id != LRExtRegRD) ||
   326:         AA.Addr->getReachingDef() != LRExtRegRD) {
   327:       LLVM_DEBUG(
   328:           dbgs() << "isSafeToExtLR: Returning false; another reaching def\n");
   329:       return false;
   330:     }
   331: 
   332:     // If the register is undefined (for example if it's a reserved register),
   333:     // it may still be possible to extend the range, but it's safer to be
   334:     // conservative and just punt.
   335:     if (LRExtRegRD == 0)
   336:       return false;
   337: 
   338:     MachineInstr *UseMI = NodeAddr<StmtNode *>(IA).Addr->getCode();
   339:     NodeAddr<DefNode *> LRExtRegDN = DFG->addr<DefNode *>(LRExtRegRD);
   340:     // Reaching Def to LRExtReg can't be a phi.
   341:     if ((LRExtRegDN.Addr->getFlags() & NodeAttrs::PhiRef) &&
   342:         MI->getParent() != UseMI->getParent())
   343:       return false;
   344:     // Is the OffsetReg definition actually reaches UseMI?
   345:     if (!UseMI->getParent()->isLiveIn(LRExtReg) &&
   346:         MI->getParent() != UseMI->getParent()) {
   347:       LLVM_DEBUG(dbgs() << "  The LRExtReg reg " << printReg(LRExtReg, TRI)
   348:                         << " is NOT live in to MBB "
   349:                         << UseMI->getParent()->getName() << "\n");
   350:       return false;
   351:     }
   352:   }
   353:   return true;
   354: }
   355: 
   356: bool HexagonOptAddrMode::isValidOffset(MachineInstr *MI, int Offset) {
   357:   if (HII->isHVXVec(*MI)) {
   358:     // only HVX vgather instructions handled
   359:     // TODO: extend the pass to other vector load/store operations
   360:     switch (MI->getOpcode()) {
   361:     case Hexagon::V6_vgathermh_pseudo:
   362:     case Hexagon::V6_vgathermw_pseudo:
   363:     case Hexagon::V6_vgathermhw_pseudo:
   364:     case Hexagon::V6_vgathermhq_pseudo:
   365:     case Hexagon::V6_vgathermwq_pseudo:
   366:     case Hexagon::V6_vgathermhwq_pseudo:
   367:       return HII->isValidOffset(MI->getOpcode(), Offset, HRI, false);
   368:     default:
   369:       if (HII->getAddrMode(*MI) == HexagonII::BaseImmOffset) {
   370:         // The immediates are mentioned in multiples of vector counts
   371:         unsigned AlignMask = HII->getMemAccessSize(*MI) - 1;
   372:         if ((AlignMask & Offset) == 0)
   373:           return HII->isValidOffset(MI->getOpcode(), Offset, HRI, false);
   374:       }
   375:       return false;
   376:     }
   377:   }
   378: 
   379:   if (HII->getAddrMode(*MI) != HexagonII::BaseImmOffset)
   380:     return false;
   381: 
   382:   unsigned AlignMask = 0;
   383:   switch (HII->getMemAccessSize(*MI)) {
   384:   case HexagonII::MemAccessSize::DoubleWordAccess:
   385:     AlignMask = 0x7;
   386:     break;
   387:   case HexagonII::MemAccessSize::WordAccess:
   388:     AlignMask = 0x3;
   389:     break;
   390:   case HexagonII::MemAccessSize::HalfWordAccess:
   391:     AlignMask = 0x1;
   392:     break;
   393:   case HexagonII::MemAccessSize::ByteAccess:
   394:     AlignMask = 0x0;
   395:     break;
   396:   default:
   397:     return false;
   398:   }
   399: 
   400:   if ((AlignMask & Offset) != 0)
```
- EN: It declares or implements routines such as HexagonOptAddrMode::isSafeToExtLR, getRegRef, getReachingDef, getOwner, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptAddrMode, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonOptAddrMode::isSafeToExtLR, getRegRef, getReachingDef, getOwner, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:     return false;
   402:   return HII->isValidOffset(MI->getOpcode(), Offset, HRI, false);
   403: }
   404: 
   405: unsigned HexagonOptAddrMode::getBaseOpPosition(MachineInstr *MI) {
   406:   const MCInstrDesc &MID = MI->getDesc();
   407:   switch (MI->getOpcode()) {
   408:   // vgather pseudos are mayLoad and mayStore
   409:   // hence need to explicitly specify Base and
   410:   // Offset operand positions
   411:   case Hexagon::V6_vgathermh_pseudo:
   412:   case Hexagon::V6_vgathermw_pseudo:
   413:   case Hexagon::V6_vgathermhw_pseudo:
   414:   case Hexagon::V6_vgathermhq_pseudo:
   415:   case Hexagon::V6_vgathermwq_pseudo:
   416:   case Hexagon::V6_vgathermhwq_pseudo:
   417:     return 0;
   418:   default:
   419:     return MID.mayLoad() ? 1 : 0;
   420:   }
   421: }
   422: 
   423: unsigned HexagonOptAddrMode::getOffsetOpPosition(MachineInstr *MI) {
   424:   assert(
   425:       (HII->getAddrMode(*MI) == HexagonII::BaseImmOffset) &&
   426:       "Looking for an offset in non-BaseImmOffset addressing mode instruction");
   427: 
   428:   const MCInstrDesc &MID = MI->getDesc();
   429:   switch (MI->getOpcode()) {
   430:   // vgather pseudos are mayLoad and mayStore
   431:   // hence need to explicitly specify Base and
   432:   // Offset operand positions
   433:   case Hexagon::V6_vgathermh_pseudo:
   434:   case Hexagon::V6_vgathermw_pseudo:
   435:   case Hexagon::V6_vgathermhw_pseudo:
   436:   case Hexagon::V6_vgathermhq_pseudo:
   437:   case Hexagon::V6_vgathermwq_pseudo:
   438:   case Hexagon::V6_vgathermhwq_pseudo:
   439:     return 1;
   440:   default:
   441:     return MID.mayLoad() ? 2 : 1;
   442:   }
   443: }
   444: 
   445: bool HexagonOptAddrMode::usedInLoadStore(NodeAddr<StmtNode *> CurrentInstSN,
   446:                                          int64_t NewOffset) {
   447:   NodeList LoadStoreUseList;
   448: 
   449:   getAllRealUses(CurrentInstSN, LoadStoreUseList);
   450:   bool FoundLoadStoreUse = false;
   451:   for (NodeAddr<UseNode *> UN : LoadStoreUseList) {
   452:     NodeAddr<StmtNode *> SN = UN.Addr->getOwner(*DFG);
   453:     MachineInstr *LoadStoreMI = SN.Addr->getCode();
   454:     const MCInstrDesc &MID = LoadStoreMI->getDesc();
   455:     if ((MID.mayLoad() || MID.mayStore()) &&
   456:         isValidOffset(LoadStoreMI, NewOffset)) {
   457:       FoundLoadStoreUse = true;
   458:       break;
   459:     }
   460:   }
   461:   return FoundLoadStoreUse;
   462: }
   463: 
   464: bool HexagonOptAddrMode::findFirstReachedInst(
   465:     MachineInstr *AddMI,
   466:     std::vector<std::pair<NodeAddr<StmtNode *>, NodeAddr<UseNode *>>> &AddiList,
   467:     NodeAddr<StmtNode *> &UseSN) {
   468:   // Find the very first Addi instruction in the current basic block among the
   469:   // AddiList This is the Addi that should be preserved so that we do not need
   470:   // to handle the complexity of moving instructions
   471:   //
   472:   // TODO: find Addi instructions across basic blocks
   473:   //
   474:   // TODO: Try to remove this and add a solution that optimizes the number of
   475:   // Addi instructions that can be modified.
   476:   // This change requires choosing the Addi with the median offset value, but
   477:   // would also require moving that instruction above the others. Since this
   478:   // pass runs after register allocation, there might be multiple cases that
   479:   // need to be handled if we move instructions around
   480:   MachineBasicBlock *CurrentMBB = AddMI->getParent();
   481:   for (auto &InstIter : *CurrentMBB) {
   482:     // If the instruction is an Addi and is in the AddiList
   483:     if (InstIter.getOpcode() == Hexagon::A2_addi) {
   484:       auto Iter = llvm::find_if(AddiList, [&InstIter](const auto &SUPair) {
   485:         return SUPair.first.Addr->getCode() == &InstIter;
   486:       });
   487:       if (Iter != AddiList.end()) {
   488:         UseSN = Iter->first;
   489:         return true;
   490:       }
   491:     }
   492:   }
   493:   return false;
   494: }
   495: 
   496: // This function tries to modify the immediate value in Hexagon::Addi
   497: // instructions, so that the immediates could then be moved into a load/store
   498: // instruction with offset and the add removed completely when we call
   499: // processAddUses
   500: //
```
- EN: It declares or implements routines such as isValidOffset, HexagonOptAddrMode::getBaseOpPosition, getDesc, HexagonOptAddrMode::getOffsetOpPosition, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonOptAddrMode, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isValidOffset, HexagonOptAddrMode::getBaseOpPosition, getDesc, HexagonOptAddrMode::getOffsetOpPosition, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501: // For Example, If we have the below sequence of instructions:
   502: //
   503: //         r1 = add(r2,#1024)
   504: //               ...
   505: //         r3 = add(r2,#1152)
   506: //               ...
   507: //         r4 = add(r2,#1280)
   508: //
   509: // Where the register r2 has the same reaching definition, They get modified to
   510: // the below sequence:
   511: //
   512: //         r1 = add(r2,#1024)
   513: //              ...
   514: //         r3 = add(r1,#128)
   515: //              ...
   516: //         r4 = add(r1,#256)
   517: //
   518: //  The below change helps the processAddUses method to later move the
   519: //  immediates #128 and #256 into a load/store instruction that can take an
   520: //  offset, like the Vd = mem(Rt+#s4)
   521: bool HexagonOptAddrMode::processAddBases(NodeAddr<StmtNode *> AddSN,
   522:                                          MachineInstr *AddMI) {
   523: 
   524:   bool Changed = false;
   525: 
   526:   LLVM_DEBUG(dbgs() << "\n\t\t[Processing Addi]: " << *AddMI << "\n");
   527: 
   528:   auto Processed =
   529:       [](const MachineInstr *MI,
   530:          const DenseSet<MachineInstr *> &ProcessedAddiInsts) -> bool {
   531:     // If we've already processed this Addi, just return
   532:     if (ProcessedAddiInsts.contains(MI)) {
   533:       LLVM_DEBUG(dbgs() << "\t\t\tAddi already found in ProcessedAddiInsts: "
   534:                         << *MI << "\n\t\t\tSkipping...");
   535:       return true;
   536:     }
   537:     return false;
   538:   };
   539: 
   540:   if (Processed(AddMI, ProcessedAddiInsts))
   541:     return Changed;
   542:   ProcessedAddiInsts.insert(AddMI);
   543: 
   544:   // Get the base register that would be shared by other Addi Instructions
   545:   Register BaseReg = AddMI->getOperand(1).getReg();
   546: 
   547:   // Store a list of all Addi instructions that share the above common base
   548:   // register
   549:   std::vector<std::pair<NodeAddr<StmtNode *>, NodeAddr<UseNode *>>> AddiList;
   550: 
   551:   NodeId UAReachingDefID;
   552:   // Find the UseNode that contains the base register and it's reachingDef
   553:   for (NodeAddr<UseNode *> UA : AddSN.Addr->members_if(DFG->IsUse, *DFG)) {
   554:     RegisterRef URR = UA.Addr->getRegRef(*DFG);
   555:     if (BaseReg != URR.asMCReg())
   556:       continue;
   557: 
   558:     UAReachingDefID = UA.Addr->getReachingDef();
   559:     NodeAddr<DefNode *> UADef = DFG->addr<DefNode *>(UAReachingDefID);
   560:     if (!UAReachingDefID || UADef.Addr->getFlags() & NodeAttrs::PhiRef) {
   561:       LLVM_DEBUG(dbgs() << "\t\t\t Could not find reachingDef. Skipping...\n");
   562:       return false;
   563:     }
   564:   }
   565: 
   566:   NodeAddr<DefNode *> UAReachingDef = DFG->addr<DefNode *>(UAReachingDefID);
   567:   NodeAddr<StmtNode *> ReachingDefStmt = UAReachingDef.Addr->getOwner(*DFG);
   568: 
   569:   // If the reaching definition is a predicated instruction, this might not be
   570:   // the only definition of our base register, so return immediately.
   571:   MachineInstr *ReachingDefInstr = ReachingDefStmt.Addr->getCode();
   572:   if (HII->isPredicated(*ReachingDefInstr))
   573:     return false;
   574: 
   575:   NodeList AddiUseList;
   576: 
   577:   // Find all Addi instructions that share the same base register and add them
   578:   // to the AddiList
   579:   getAllRealUses(ReachingDefStmt, AddiUseList);
   580:   for (NodeAddr<UseNode *> UN : AddiUseList) {
   581:     NodeAddr<StmtNode *> SN = UN.Addr->getOwner(*DFG);
   582:     MachineInstr *MI = SN.Addr->getCode();
   583: 
   584:     // Only add instructions if it's an Addi and it's not already processed.
   585:     if (MI->getOpcode() == Hexagon::A2_addi &&
   586:         !(MI != AddMI && Processed(MI, ProcessedAddiInsts))) {
   587:       AddiList.push_back({SN, UN});
   588: 
   589:       // This ensures that we process each instruction only once
   590:       ProcessedAddiInsts.insert(MI);
   591:     }
   592:   }
   593: 
   594:   // If there's only one Addi instruction, nothing to do here
   595:   if (AddiList.size() <= 1)
   596:     return Changed;
   597: 
   598:   NodeAddr<StmtNode *> FirstReachedUseSN;
   599:   // Find the first reached use of Addi instruction from the list
   600:   if (!findFirstReachedInst(AddMI, AddiList, FirstReachedUseSN))
```
- EN: It declares or implements routines such as add, LLVM_DEBUG, insert, getOperand, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptAddrMode, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 add, LLVM_DEBUG, insert, getOperand, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:     return Changed;
   602: 
   603:   // If we reach this point we know that the StmtNode FirstReachedUseSN is for
   604:   // an Addi instruction. So, we're guaranteed to have just one DefNode, and
   605:   // hence we can access the front() directly without checks
   606:   NodeAddr<DefNode *> FirstReachedUseDN =
   607:       FirstReachedUseSN.Addr->members_if(DFG->IsDef, *DFG).front();
   608: 
   609:   MachineInstr *FirstReachedMI = FirstReachedUseSN.Addr->getCode();
   610:   const MachineOperand FirstReachedMIImmOp = FirstReachedMI->getOperand(2);
   611:   if (!FirstReachedMIImmOp.isImm())
   612:     return false;
   613: 
   614:   for (auto &I : AddiList) {
   615:     NodeAddr<StmtNode *> CurrentInstSN = I.first;
   616:     NodeAddr<UseNode *> CurrentInstUN = I.second;
   617: 
   618:     MachineInstr *CurrentMI = CurrentInstSN.Addr->getCode();
   619:     MachineOperand &CurrentMIImmOp = CurrentMI->getOperand(2);
   620: 
   621:     int64_t NewOffset;
   622: 
   623:     // Even though we know it's an Addi instruction, the second operand could be
   624:     // a global value and not an immediate
   625:     if (!CurrentMIImmOp.isImm())
   626:       continue;
   627: 
   628:     NewOffset = CurrentMIImmOp.getImm() - FirstReachedMIImmOp.getImm();
   629: 
   630:     // This is the first occurring Addi, so skip modifying this
   631:     if (CurrentMI == FirstReachedMI) {
   632:       continue;
   633:     }
   634: 
   635:     if (CurrentMI->getParent() != FirstReachedMI->getParent())
   636:       continue;
   637: 
   638:     // Modify the Addi instruction only if it could be used to modify a
   639:     // future load/store instruction and get removed
   640:     //
   641:     // This check is needed because, if we modify the current Addi instruction
   642:     // we create RAW dependence between the FirstReached Addi and the current
   643:     // one, which could result in extra packets. So we only do this change if
   644:     // we know the current Addi would get removed later
   645:     if (!usedInLoadStore(CurrentInstSN, NewOffset)) {
   646:       return false;
   647:     }
   648: 
   649:     // Verify whether the First Addi's definition register is still live when
   650:     // we reach the current Addi
   651:     RegisterRef FirstReachedDefRR = FirstReachedUseDN.Addr->getRegRef(*DFG);
   652:     NodeAddr<InstrNode *> CurrentAddiIN = CurrentInstUN.Addr->getOwner(*DFG);
   653:     NodeAddr<RefNode *> NearestAA =
   654:         LV->getNearestAliasedRef(FirstReachedDefRR, CurrentAddiIN);
   655:     if ((DFG->IsDef(NearestAA) && NearestAA.Id != FirstReachedUseDN.Id) ||
   656:         (!DFG->IsDef(NearestAA) &&
   657:          NearestAA.Addr->getReachingDef() != FirstReachedUseDN.Id)) {
   658:       // Found another definition of FirstReachedDef
   659:       LLVM_DEBUG(dbgs() << "\t\t\tCould not modify below Addi since the first "
   660:                            "defined Addi register was redefined\n");
   661:       continue;
   662:     }
   663: 
   664:     MachineOperand CurrentMIBaseOp = CurrentMI->getOperand(1);
   665:     if (CurrentMIBaseOp.getReg() != FirstReachedMI->getOperand(1).getReg()) {
   666:       continue;
   667:     }
   668: 
   669:     // If we reached this point, then we can modify MI to use the result of
   670:     // FirstReachedMI
   671:     Changed |= updateAddBases(CurrentMI, FirstReachedMI, NewOffset);
   672: 
   673:     // Update the reachingDef of the Current AddI use after change
   674:     CurrentInstUN.Addr->linkToDef(CurrentInstUN.Id, FirstReachedUseDN);
   675:   }
   676: 
   677:   return Changed;
   678: }
   679: 
   680: bool HexagonOptAddrMode::updateAddBases(MachineInstr *CurrentMI,
   681:                                         MachineInstr *FirstReachedMI,
   682:                                         int64_t NewOffset) {
   683:   LLVM_DEBUG(dbgs() << "[About to modify the Addi]: " << *CurrentMI << "\n");
   684:   const MachineOperand FirstReachedDef = FirstReachedMI->getOperand(0);
   685:   Register FirstDefRegister = FirstReachedDef.getReg();
   686: 
   687:   MachineOperand &CurrentMIBaseOp = CurrentMI->getOperand(1);
   688:   MachineOperand &CurrentMIImmOp = CurrentMI->getOperand(2);
   689: 
   690:   CurrentMIBaseOp.setReg(FirstDefRegister);
   691:   CurrentMIBaseOp.setIsUndef(FirstReachedDef.isUndef());
   692:   CurrentMIBaseOp.setImplicit(FirstReachedDef.isImplicit());
   693:   CurrentMIImmOp.setImm(NewOffset);
   694:   ProcessedAddiInsts.insert(CurrentMI);
   695:   MRI->clearKillFlags(FirstDefRegister);
   696:   return true;
   697: }
   698: 
   699: bool HexagonOptAddrMode::processAddUses(NodeAddr<StmtNode *> AddSN,
   700:                                         MachineInstr *AddMI,
```
- EN: It declares or implements routines such as front, getCode, getOperand, getImm, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptAddrMode, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 front, getCode, getOperand, getImm, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:                                         const NodeList &UNodeList) {
   702: 
   703:   Register AddDefR = AddMI->getOperand(0).getReg();
   704:   Register BaseReg = AddMI->getOperand(1).getReg();
   705:   for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
   706:     NodeAddr<UseNode *> UN = *I;
   707:     NodeAddr<StmtNode *> SN = UN.Addr->getOwner(*DFG);
   708:     MachineInstr *MI = SN.Addr->getCode();
   709:     const MCInstrDesc &MID = MI->getDesc();
   710:     if ((!MID.mayLoad() && !MID.mayStore()) ||
   711:         HII->getAddrMode(*MI) != HexagonII::BaseImmOffset)
   712:       return false;
   713: 
   714:     MachineOperand BaseOp = MI->getOperand(getBaseOpPosition(MI));
   715: 
   716:     if (!BaseOp.isReg() || BaseOp.getReg() != AddDefR)
   717:       return false;
   718: 
   719:     MachineOperand OffsetOp = MI->getOperand(getOffsetOpPosition(MI));
   720:     if (!OffsetOp.isImm())
   721:       return false;
   722: 
   723:     int64_t newOffset = OffsetOp.getImm() + AddMI->getOperand(2).getImm();
   724:     if (!isValidOffset(MI, newOffset))
   725:       return false;
   726:   }
   727: 
   728:   // Since we'll be extending the live range of Rt in the following example,
   729:   // make sure that is safe. another definition of Rt doesn't exist between
   730:   // 'add' and load/store instruction.
   731:   //
   732:   // Ex: Rx= add(Rt,#10)
   733:   //     memw(Rx+#0) = Rs
   734:   // will be replaced with =>  memw(Rt+#10) = Rs
   735:   // Note: isSafeToExtLR arguments are loop-invariant; call it once after
   736:   // validating all uses to avoid O(N^2) behavior when UNodeList is large.
   737:   if (!isSafeToExtLR(AddSN, AddMI, BaseReg, UNodeList))
   738:     return false;
   739: 
   740:   NodeId LRExtRegRD = 0;
   741:   // Iterate through all the UseNodes in SN and find the reaching def
   742:   // for the LRExtReg.
   743:   for (NodeAddr<UseNode *> UA : AddSN.Addr->members_if(DFG->IsUse, *DFG)) {
   744:     RegisterRef RR = UA.Addr->getRegRef(*DFG);
   745:     if (BaseReg == RR.asMCReg())
   746:       LRExtRegRD = UA.Addr->getReachingDef();
   747:   }
   748: 
   749:   // Update all the uses of 'add' with the appropriate base and offset
   750:   // values.
   751:   bool Changed = false;
   752:   for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
   753:     NodeAddr<UseNode *> UseN = *I;
   754:     assert(!(UseN.Addr->getFlags() & NodeAttrs::PhiRef) &&
   755:            "Found a PhiRef node as a real reached use!!");
   756: 
   757:     NodeAddr<StmtNode *> OwnerN = UseN.Addr->getOwner(*DFG);
   758:     MachineInstr *UseMI = OwnerN.Addr->getCode();
   759:     LLVM_DEBUG(dbgs() << "\t\t[MI <BB#" << UseMI->getParent()->getNumber()
   760:                       << ">]: " << *UseMI << "\n");
   761:     Changed |= updateAddUses(AddMI, UseMI);
   762: 
   763:     // Set the reachingDef for UseNode under consideration
   764:     // after updating the Add use. This local change is
   765:     // to avoid rebuilding of the RDF graph after update.
   766:     NodeAddr<DefNode *> LRExtRegDN = DFG->addr<DefNode *>(LRExtRegRD);
   767:     UseN.Addr->linkToDef(UseN.Id, LRExtRegDN);
   768:   }
   769: 
   770:   if (Changed)
   771:     Deleted.insert(AddMI);
   772: 
   773:   return Changed;
   774: }
   775: 
   776: bool HexagonOptAddrMode::updateAddUses(MachineInstr *AddMI,
   777:                                        MachineInstr *UseMI) {
   778:   const MachineOperand ImmOp = AddMI->getOperand(2);
   779:   const MachineOperand AddRegOp = AddMI->getOperand(1);
   780:   Register NewReg = AddRegOp.getReg();
   781: 
   782:   MachineOperand &BaseOp = UseMI->getOperand(getBaseOpPosition(UseMI));
   783:   MachineOperand &OffsetOp = UseMI->getOperand(getOffsetOpPosition(UseMI));
   784:   BaseOp.setReg(NewReg);
   785:   BaseOp.setIsUndef(AddRegOp.isUndef());
   786:   BaseOp.setImplicit(AddRegOp.isImplicit());
   787:   OffsetOp.setImm(ImmOp.getImm() + OffsetOp.getImm());
   788:   MRI->clearKillFlags(NewReg);
   789: 
   790:   return true;
   791: }
   792: 
   793: bool HexagonOptAddrMode::analyzeUses(unsigned tfrDefR,
   794:                                      const NodeList &UNodeList,
   795:                                      InstrEvalMap &InstrEvalResult,
   796:                                      short &SizeInc) {
   797:   bool KeepTfr = false;
   798:   bool HasRepInstr = false;
   799:   InstrEvalResult.clear();
   800: 
```
- EN: It declares or implements routines such as getOperand, getOwner, getCode, getDesc, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonII, HexagonOptAddrMode, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, getOwner, getCode, getDesc, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonOptAddrMode，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801:   for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
   802:     bool CanBeReplaced = false;
   803:     NodeAddr<UseNode *> UN = *I;
   804:     NodeAddr<StmtNode *> SN = UN.Addr->getOwner(*DFG);
   805:     MachineInstr &MI = *SN.Addr->getCode();
   806:     const MCInstrDesc &MID = MI.getDesc();
   807:     if ((MID.mayLoad() || MID.mayStore())) {
   808:       if (!hasRepForm(MI, tfrDefR)) {
   809:         KeepTfr = true;
   810:         continue;
   811:       }
   812:       SizeInc++;
   813:       CanBeReplaced = true;
   814:     } else if (MI.getOpcode() == Hexagon::S2_addasl_rrri) {
   815:       NodeList AddaslUseList;
   816: 
   817:       LLVM_DEBUG(dbgs() << "\nGetting ReachedUses for === " << MI << "\n");
   818:       getAllRealUses(SN, AddaslUseList);
   819:       // Process phi nodes.
   820:       if (allValidCandidates(SN, AddaslUseList) &&
   821:           canRemoveAddasl(SN, MI, AddaslUseList)) {
   822:         SizeInc += AddaslUseList.size();
   823:         SizeInc -= 1; // Reduce size by 1 as addasl itself can be removed.
   824:         CanBeReplaced = true;
   825:       } else
   826:         SizeInc++;
   827:     } else
   828:       // Currently, only load/store and addasl are handled.
   829:       // Some other instructions to consider -
   830:       // A2_add -> A2_addi
   831:       // M4_mpyrr_addr -> M4_mpyrr_addi
   832:       KeepTfr = true;
   833: 
   834:     InstrEvalResult[&MI] = CanBeReplaced;
   835:     HasRepInstr |= CanBeReplaced;
   836:   }
   837: 
   838:   // Reduce total size by 2 if original tfr can be deleted.
   839:   if (!KeepTfr)
   840:     SizeInc -= 2;
   841: 
   842:   return HasRepInstr;
   843: }
   844: 
   845: bool HexagonOptAddrMode::changeLoad(MachineInstr *OldMI, MachineOperand ImmOp,
   846:                                     unsigned ImmOpNum) {
   847:   bool Changed = false;
   848:   MachineBasicBlock *BB = OldMI->getParent();
   849:   auto UsePos = MachineBasicBlock::iterator(OldMI);
   850:   MachineBasicBlock::instr_iterator InsertPt = UsePos.getInstrIterator();
   851:   ++InsertPt;
   852:   unsigned OpStart;
   853:   unsigned OpEnd = OldMI->getNumOperands();
   854:   MachineInstrBuilder MIB;
   855: 
   856:   if (ImmOpNum == 1) {
   857:     if (HII->getAddrMode(*OldMI) == HexagonII::BaseRegOffset) {
   858:       short NewOpCode = HII->changeAddrMode_rr_ur(*OldMI);
   859:       assert(NewOpCode >= 0 && "Invalid New opcode\n");
   860:       MIB = BuildMI(*BB, InsertPt, OldMI->getDebugLoc(), HII->get(NewOpCode));
   861:       MIB.add(OldMI->getOperand(0));
   862:       MIB.add(OldMI->getOperand(2));
   863:       MIB.add(OldMI->getOperand(3));
   864:       MIB.add(ImmOp);
   865:       OpStart = 4;
   866:       Changed = true;
   867:     } else if (HII->getAddrMode(*OldMI) == HexagonII::BaseImmOffset &&
   868:                OldMI->getOperand(2).isImm()) {
   869:       short NewOpCode = HII->changeAddrMode_io_abs(*OldMI);
   870:       assert(NewOpCode >= 0 && "Invalid New opcode\n");
   871:       MIB = BuildMI(*BB, InsertPt, OldMI->getDebugLoc(), HII->get(NewOpCode))
   872:                 .add(OldMI->getOperand(0));
   873:       const GlobalValue *GV = ImmOp.getGlobal();
   874:       int64_t Offset = ImmOp.getOffset() + OldMI->getOperand(2).getImm();
   875: 
   876:       MIB.addGlobalAddress(GV, Offset, ImmOp.getTargetFlags());
   877:       OpStart = 3;
   878:       Changed = true;
   879:     } else
   880:       Changed = false;
   881: 
   882:     LLVM_DEBUG(dbgs() << "[Changing]: " << *OldMI << "\n");
   883:     LLVM_DEBUG(dbgs() << "[TO]: " << *MIB << "\n");
   884:   } else if (ImmOpNum == 2) {
   885:     if (OldMI->getOperand(3).isImm() && OldMI->getOperand(3).getImm() == 0) {
   886:       short NewOpCode = HII->changeAddrMode_rr_io(*OldMI);
   887:       assert(NewOpCode >= 0 && "Invalid New opcode\n");
   888:       MIB = BuildMI(*BB, InsertPt, OldMI->getDebugLoc(), HII->get(NewOpCode));
   889:       MIB.add(OldMI->getOperand(0));
   890:       MIB.add(OldMI->getOperand(1));
   891:       MIB.add(ImmOp);
   892:       OpStart = 4;
   893:       Changed = true;
   894:       LLVM_DEBUG(dbgs() << "[Changing]: " << *OldMI << "\n");
   895:       LLVM_DEBUG(dbgs() << "[TO]: " << *MIB << "\n");
   896:     }
   897:   }
   898: 
   899:   if (Changed)
   900:     for (unsigned i = OpStart; i < OpEnd; ++i)
```
- EN: It declares or implements routines such as getOwner, getCode, getDesc, LLVM_DEBUG, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonOptAddrMode, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOwner, getCode, getDesc, LLVM_DEBUG, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:       MIB.add(OldMI->getOperand(i));
   902: 
   903:   return Changed;
   904: }
   905: 
   906: bool HexagonOptAddrMode::changeStore(MachineInstr *OldMI, MachineOperand ImmOp,
   907:                                      unsigned ImmOpNum) {
   908:   bool Changed = false;
   909:   unsigned OpStart = 0;
   910:   unsigned OpEnd = OldMI->getNumOperands();
   911:   MachineBasicBlock *BB = OldMI->getParent();
   912:   auto UsePos = MachineBasicBlock::iterator(OldMI);
   913:   MachineBasicBlock::instr_iterator InsertPt = UsePos.getInstrIterator();
   914:   ++InsertPt;
   915:   MachineInstrBuilder MIB;
   916:   if (ImmOpNum == 0) {
   917:     if (HII->getAddrMode(*OldMI) == HexagonII::BaseRegOffset) {
   918:       short NewOpCode = HII->changeAddrMode_rr_ur(*OldMI);
   919:       assert(NewOpCode >= 0 && "Invalid New opcode\n");
   920:       MIB = BuildMI(*BB, InsertPt, OldMI->getDebugLoc(), HII->get(NewOpCode));
   921:       MIB.add(OldMI->getOperand(1));
   922:       MIB.add(OldMI->getOperand(2));
   923:       MIB.add(ImmOp);
   924:       MIB.add(OldMI->getOperand(3));
   925:       OpStart = 4;
   926:       Changed = true;
   927:     } else if (HII->getAddrMode(*OldMI) == HexagonII::BaseImmOffset) {
   928:       short NewOpCode = HII->changeAddrMode_io_abs(*OldMI);
   929:       assert(NewOpCode >= 0 && "Invalid New opcode\n");
   930:       MIB = BuildMI(*BB, InsertPt, OldMI->getDebugLoc(), HII->get(NewOpCode));
   931:       const GlobalValue *GV = ImmOp.getGlobal();
   932:       int64_t Offset = ImmOp.getOffset() + OldMI->getOperand(1).getImm();
   933:       MIB.addGlobalAddress(GV, Offset, ImmOp.getTargetFlags());
   934:       MIB.add(OldMI->getOperand(2));
   935:       OpStart = 3;
   936:       Changed = true;
   937:     }
   938:   } else if (ImmOpNum == 1 && OldMI->getOperand(2).getImm() == 0) {
   939:     short NewOpCode = HII->changeAddrMode_rr_io(*OldMI);
   940:     assert(NewOpCode >= 0 && "Invalid New opcode\n");
   941:     MIB = BuildMI(*BB, InsertPt, OldMI->getDebugLoc(), HII->get(NewOpCode));
   942:     MIB.add(OldMI->getOperand(0));
   943:     MIB.add(ImmOp);
   944:     OpStart = 3;
   945:     Changed = true;
   946:   }
   947:   if (Changed) {
   948:     LLVM_DEBUG(dbgs() << "[Changing]: " << *OldMI << "\n");
   949:     LLVM_DEBUG(dbgs() << "[TO]: " << *MIB << "\n");
   950: 
   951:     for (unsigned i = OpStart; i < OpEnd; ++i)
   952:       MIB.add(OldMI->getOperand(i));
   953:   }
   954: 
   955:   return Changed;
   956: }
   957: 
   958: short HexagonOptAddrMode::getBaseWithLongOffset(const MachineInstr &MI) const {
   959:   if (HII->getAddrMode(MI) == HexagonII::BaseImmOffset) {
   960:     short TempOpCode = HII->changeAddrMode_io_rr(MI);
   961:     return HII->changeAddrMode_rr_ur(TempOpCode);
   962:   }
   963:   return HII->changeAddrMode_rr_ur(MI);
   964: }
   965: 
   966: bool HexagonOptAddrMode::changeAddAsl(NodeAddr<UseNode *> AddAslUN,
   967:                                       MachineInstr *AddAslMI,
   968:                                       const MachineOperand &ImmOp,
   969:                                       unsigned ImmOpNum) {
   970:   NodeAddr<StmtNode *> SA = AddAslUN.Addr->getOwner(*DFG);
   971: 
   972:   LLVM_DEBUG(dbgs() << "Processing addasl :" << *AddAslMI << "\n");
   973: 
   974:   NodeList UNodeList;
   975:   getAllRealUses(SA, UNodeList);
   976: 
   977:   for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
   978:     NodeAddr<UseNode *> UseUN = *I;
   979:     assert(!(UseUN.Addr->getFlags() & NodeAttrs::PhiRef) &&
   980:            "Can't transform this 'AddAsl' instruction!");
   981: 
   982:     NodeAddr<StmtNode *> UseIA = UseUN.Addr->getOwner(*DFG);
   983:     LLVM_DEBUG(dbgs() << "[InstrNode]: "
   984:                       << Print<NodeAddr<InstrNode *>>(UseIA, *DFG) << "\n");
   985:     MachineInstr *UseMI = UseIA.Addr->getCode();
   986:     LLVM_DEBUG(dbgs() << "[MI <" << printMBBReference(*UseMI->getParent())
   987:                       << ">]: " << *UseMI << "\n");
   988:     const MCInstrDesc &UseMID = UseMI->getDesc();
   989:     assert(HII->getAddrMode(*UseMI) == HexagonII::BaseImmOffset);
   990: 
   991:     auto UsePos = MachineBasicBlock::iterator(UseMI);
   992:     MachineBasicBlock::instr_iterator InsertPt = UsePos.getInstrIterator();
   993:     short NewOpCode = getBaseWithLongOffset(*UseMI);
   994:     assert(NewOpCode >= 0 && "Invalid New opcode\n");
   995: 
   996:     unsigned OpStart;
   997:     unsigned OpEnd = UseMI->getNumOperands();
   998: 
   999:     MachineBasicBlock *BB = UseMI->getParent();
  1000:     MachineInstrBuilder MIB =
```
- EN: It declares or implements routines such as add, HexagonOptAddrMode::changeStore, getNumOperands, getParent, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonOptAddrMode, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 add, HexagonOptAddrMode::changeStore, getNumOperands, getParent, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:         BuildMI(*BB, InsertPt, UseMI->getDebugLoc(), HII->get(NewOpCode));
  1002:     // change mem(Rs + # ) -> mem(Rt << # + ##)
  1003:     if (UseMID.mayLoad()) {
  1004:       MIB.add(UseMI->getOperand(0));
  1005:       MIB.add(AddAslMI->getOperand(2));
  1006:       MIB.add(AddAslMI->getOperand(3));
  1007:       const GlobalValue *GV = ImmOp.getGlobal();
  1008:       MIB.addGlobalAddress(GV, UseMI->getOperand(2).getImm()+ImmOp.getOffset(),
  1009:                            ImmOp.getTargetFlags());
  1010:       OpStart = 3;
  1011:     } else if (UseMID.mayStore()) {
  1012:       MIB.add(AddAslMI->getOperand(2));
  1013:       MIB.add(AddAslMI->getOperand(3));
  1014:       const GlobalValue *GV = ImmOp.getGlobal();
  1015:       MIB.addGlobalAddress(GV, UseMI->getOperand(1).getImm()+ImmOp.getOffset(),
  1016:                            ImmOp.getTargetFlags());
  1017:       MIB.add(UseMI->getOperand(2));
  1018:       OpStart = 3;
  1019:     } else
  1020:       llvm_unreachable("Unhandled instruction");
  1021: 
  1022:     for (unsigned i = OpStart; i < OpEnd; ++i)
  1023:       MIB.add(UseMI->getOperand(i));
  1024:     Deleted.insert(UseMI);
  1025:   }
  1026: 
  1027:   return true;
  1028: }
  1029: 
  1030: bool HexagonOptAddrMode::xformUseMI(MachineInstr *TfrMI, MachineInstr *UseMI,
  1031:                                     NodeAddr<UseNode *> UseN,
  1032:                                     unsigned UseMOnum) {
  1033:   const MachineOperand ImmOp = TfrMI->getOperand(1);
  1034:   const MCInstrDesc &MID = UseMI->getDesc();
  1035:   unsigned Changed = false;
  1036:   if (MID.mayLoad())
  1037:     Changed = changeLoad(UseMI, ImmOp, UseMOnum);
  1038:   else if (MID.mayStore())
  1039:     Changed = changeStore(UseMI, ImmOp, UseMOnum);
  1040:   else if (UseMI->getOpcode() == Hexagon::S2_addasl_rrri)
  1041:     Changed = changeAddAsl(UseN, UseMI, ImmOp, UseMOnum);
  1042: 
  1043:   if (Changed)
  1044:     Deleted.insert(UseMI);
  1045: 
  1046:   return Changed;
  1047: }
  1048: 
  1049: bool HexagonOptAddrMode::processBlock(NodeAddr<BlockNode *> BA) {
  1050:   bool Changed = false;
  1051: 
  1052:   for (auto IA : BA.Addr->members(*DFG)) {
  1053:     if (!DFG->IsCode<NodeAttrs::Stmt>(IA))
  1054:       continue;
  1055: 
  1056:     NodeAddr<StmtNode *> SA = IA;
  1057:     MachineInstr *MI = SA.Addr->getCode();
  1058:     if ((MI->getOpcode() != Hexagon::A2_tfrsi ||
  1059:          !MI->getOperand(1).isGlobal()) &&
  1060:         (MI->getOpcode() != Hexagon::A2_addi ||
  1061:          !MI->getOperand(2).isImm() || HII->isConstExtended(*MI)))
  1062:     continue;
  1063: 
  1064:     LLVM_DEBUG(dbgs() << "[Analyzing " << HII->getName(MI->getOpcode())
  1065:                       << "]: " << *MI << "\n\t[InstrNode]: "
  1066:                       << Print<NodeAddr<InstrNode *>>(IA, *DFG) << '\n');
  1067: 
  1068:     if (MI->getOpcode() == Hexagon::A2_addi)
  1069:       Changed |= processAddBases(SA, MI);
  1070:     NodeList UNodeList;
  1071:     getAllRealUses(SA, UNodeList);
  1072: 
  1073:     if (!allValidCandidates(SA, UNodeList))
  1074:       continue;
  1075: 
  1076:     // Analyze all uses of 'add'. If the output of 'add' is used as an address
  1077:     // in the base+immediate addressing mode load/store instructions, see if
  1078:     // they can be updated to use the immediate value as an offset. Thus,
  1079:     // providing us the opportunity to eliminate 'add'.
  1080:     // Ex: Rx= add(Rt,#12)
  1081:     //     memw(Rx+#0) = Rs
  1082:     // This can be replaced with memw(Rt+#12) = Rs
  1083:     //
  1084:     // This transformation is only performed if all uses can be updated and
  1085:     // the offset isn't required to be constant extended.
  1086:     if (MI->getOpcode() == Hexagon::A2_addi) {
  1087:       Changed |= processAddUses(SA, MI, UNodeList);
  1088:       continue;
  1089:     }
  1090: 
  1091:     short SizeInc = 0;
  1092:     Register DefR = MI->getOperand(0).getReg();
  1093:     InstrEvalMap InstrEvalResult;
  1094: 
  1095:     // Analyze all uses and calculate increase in size. Perform the optimization
  1096:     // only if there is no increase in size.
  1097:     if (!analyzeUses(DefR, UNodeList, InstrEvalResult, SizeInc))
  1098:       continue;
  1099:     if (SizeInc > CodeGrowthLimit)
  1100:       continue;
```
- EN: It declares or implements routines such as BuildMI, mem, add, getGlobal, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonOptAddrMode, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 BuildMI, mem, add, getGlobal, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode，说明了它与同级后端组件的连接关系。

### Lines 1101-1200 / 第 1101-1200 行

```cpp
  1101: 
  1102:     bool KeepTfr = false;
  1103: 
  1104:     LLVM_DEBUG(dbgs() << "\t[Total reached uses] : " << UNodeList.size()
  1105:                       << "\n");
  1106:     LLVM_DEBUG(dbgs() << "\t[Processing Reached Uses] ===\n");
  1107:     for (auto I = UNodeList.rbegin(), E = UNodeList.rend(); I != E; ++I) {
  1108:       NodeAddr<UseNode *> UseN = *I;
  1109:       assert(!(UseN.Addr->getFlags() & NodeAttrs::PhiRef) &&
  1110:              "Found a PhiRef node as a real reached use!!");
  1111: 
  1112:       NodeAddr<StmtNode *> OwnerN = UseN.Addr->getOwner(*DFG);
  1113:       MachineInstr *UseMI = OwnerN.Addr->getCode();
  1114:       LLVM_DEBUG(dbgs() << "\t\t[MI <" << printMBBReference(*UseMI->getParent())
  1115:                         << ">]: " << *UseMI << "\n");
  1116: 
  1117:       int UseMOnum = -1;
  1118:       unsigned NumOperands = UseMI->getNumOperands();
  1119:       for (unsigned j = 0; j < NumOperands - 1; ++j) {
  1120:         const MachineOperand &op = UseMI->getOperand(j);
  1121:         if (op.isReg() && op.isUse() && DefR == op.getReg())
  1122:           UseMOnum = j;
  1123:       }
  1124:       // It is possible that the register will not be found in any operand.
  1125:       // This could happen, for example, when DefR = R4, but the used
  1126:       // register is D2.
  1127: 
  1128:       // Change UseMI if replacement is possible. If any replacement failed,
  1129:       // or wasn't attempted, make sure to keep the TFR.
  1130:       bool Xformed = false;
  1131:       if (UseMOnum >= 0 && InstrEvalResult[UseMI])
  1132:         Xformed = xformUseMI(MI, UseMI, UseN, UseMOnum);
  1133:       Changed |=  Xformed;
  1134:       KeepTfr |= !Xformed;
  1135:     }
  1136:     if (!KeepTfr)
  1137:       Deleted.insert(MI);
  1138:   }
  1139:   return Changed;
  1140: }
  1141: 
  1142: bool HexagonOptAddrMode::runOnMachineFunction(MachineFunction &MF) {
  1143:   if (skipFunction(MF.getFunction()))
  1144:     return false;
  1145: 
  1146:   // Perform RDF optimizations only if number of basic blocks in the
  1147:   // function is less than the limit
  1148:   if (MF.size() > RDFFuncBlockLimit) {
  1149:     LLVM_DEBUG(dbgs() << "Skipping " << getPassName()
  1150:                       << ": too many basic blocks\n");
  1151:     return false;
  1152:   }
  1153: 
  1154:   bool Changed = false;
  1155:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  1156:   MRI = &MF.getRegInfo();
  1157:   TRI = MF.getSubtarget().getRegisterInfo();
  1158:   HII = HST.getInstrInfo();
  1159:   HRI = HST.getRegisterInfo();
  1160:   const auto &MDF = getAnalysis<MachineDominanceFrontierWrapperPass>().getMDF();
  1161:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  1162: 
  1163:   DataFlowGraph G(MF, *HII, *HRI, *MDT, MDF);
  1164:   // Need to keep dead phis because we can propagate uses of registers into
  1165:   // nodes dominated by those would-be phis.
  1166:   G.build(BuildOptions::KeepDeadPhis);
  1167:   DFG = &G;
  1168: 
  1169:   Liveness L(*MRI, *DFG);
  1170:   L.computePhiInfo();
  1171:   LV = &L;
  1172: 
  1173:   Deleted.clear();
  1174:   ProcessedAddiInsts.clear();
  1175:   NodeAddr<FuncNode *> FA = DFG->getFunc();
  1176:   LLVM_DEBUG(dbgs() << "==== [RefMap#]=====:\n "
  1177:                     << Print<NodeAddr<FuncNode *>>(FA, *DFG) << "\n");
  1178: 
  1179:   for (NodeAddr<BlockNode *> BA : FA.Addr->members(*DFG))
  1180:     Changed |= processBlock(BA);
  1181: 
  1182:   for (auto *MI : Deleted)
  1183:     MI->eraseFromParent();
  1184: 
  1185:   if (Changed) {
  1186:     G.build();
  1187:     L.computeLiveIns();
  1188:     L.resetLiveIns();
  1189:     L.resetKills();
  1190:   }
  1191: 
  1192:   return Changed;
  1193: }
  1194: 
  1195: //===----------------------------------------------------------------------===//
  1196: //                         Public Constructor Functions
  1197: //===----------------------------------------------------------------------===//
  1198: 
  1199: FunctionPass *llvm::createHexagonOptAddrMode() {
  1200:   return new HexagonOptAddrMode();
```
- EN: It declares or implements routines such as LLVM_DEBUG, assert, getOwner, getCode, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonOptAddrMode, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, assert, getOwner, getCode, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonOptAddrMode, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 1201-1201 / 第 1201-1201 行

```cpp
  1201: }
```
- EN: This range contains executable implementation details for a Hexagon backend subsystem.
- CN: 这一段包含了某个 Hexagon 后端子系统的可执行实现细节。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, MCTargetDesc/HexagonBaseInfo.h, llvm/ADT/DenseMap.h, llvm/ADT/DenseSet.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineDominanceFrontier.h, llvm/CodeGen/MachineDominators.h, ... (29 total)`
- Hexagon symbols / Hexagon 符号: `HexagonOptAddrMode, HexagonInstrInfo, HexagonSubtarget, HexagonBaseInfo, HexagonRegisterInfo, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
