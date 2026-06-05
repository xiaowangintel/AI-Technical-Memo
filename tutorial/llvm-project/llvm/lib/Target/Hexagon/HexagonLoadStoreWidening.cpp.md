# HexagonLoadStoreWidening.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonLoadStoreWidening.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon widening of neighboring loads and stores.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===---HexagonLoadStoreWidening.cpp---------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // HexagonStoreWidening:
     9: // Replace sequences of "narrow" stores to adjacent memory locations with
    10: // a fewer "wide" stores that have the same effect.
    11: // For example, replace:
    12: //   S4_storeirb_io  %100, 0, 0   ; store-immediate-byte
    13: //   S4_storeirb_io  %100, 1, 0   ; store-immediate-byte
    14: // with
    15: //   S4_storeirh_io  %100, 0, 0   ; store-immediate-halfword
    16: // The above is the general idea.  The actual cases handled by the code
    17: // may be a bit more complex.
    18: // The purpose of this pass is to reduce the number of outstanding stores,
    19: // or as one could say, "reduce store queue pressure".  Also, wide stores
    20: // mean fewer stores, and since there are only two memory instructions allowed
    21: // per packet, it also means fewer packets, and ultimately fewer cycles.
    22: //
    23: // HexagonLoadWidening does the same thing as HexagonStoreWidening but
    24: // for Loads. Here, we try to replace 4-byte Loads with register-pair loads.
    25: // For example:
    26: // Replace
    27: //   %2:intregs = L2_loadri_io %1:intregs, 0 :: (load (s32) from %ptr1, align 8)
    28: //   %3:intregs = L2_loadri_io %1:intregs, 4 :: (load (s32) from %ptr2)
    29: // with
    30: //   %4:doubleregs = L2_loadrd_io %1:intregs, 0 :: (load (s64) from %ptr1)
    31: //   %2:intregs = COPY %4.isub_lo:doubleregs
    32: //   %3:intregs = COPY %4.isub_hi:doubleregs
    33: //
    34: // LoadWidening for 8 and 16-bit loads is not useful as we end up generating 2N
    35: // insts to replace N loads: 1 widened load, N bitwise and, N - 1 shifts
    36: 
    37: //===---------------------------------------------------------------------===//
    38: 
    39: #include "Hexagon.h"
    40: #include "HexagonInstrInfo.h"
    41: #include "HexagonRegisterInfo.h"
    42: #include "HexagonSubtarget.h"
    43: #include "llvm/ADT/SmallPtrSet.h"
    44: #include "llvm/Analysis/AliasAnalysis.h"
    45: #include "llvm/Analysis/MemoryLocation.h"
    46: #include "llvm/CodeGen/MachineBasicBlock.h"
    47: #include "llvm/CodeGen/MachineFunction.h"
    48: #include "llvm/CodeGen/MachineFunctionPass.h"
    49: #include "llvm/CodeGen/MachineInstr.h"
    50: #include "llvm/CodeGen/MachineInstrBuilder.h"
    51: #include "llvm/CodeGen/MachineMemOperand.h"
    52: #include "llvm/CodeGen/MachineOperand.h"
    53: #include "llvm/CodeGen/MachineRegisterInfo.h"
    54: #include "llvm/IR/DebugLoc.h"
    55: #include "llvm/InitializePasses.h"
    56: #include "llvm/MC/MCInstrDesc.h"
    57: #include "llvm/Pass.h"
    58: #include "llvm/Support/Debug.h"
    59: #include "llvm/Support/ErrorHandling.h"
    60: #include "llvm/Support/MathExtras.h"
    61: #include "llvm/Support/raw_ostream.h"
    62: #include <algorithm>
    63: #include <cassert>
    64: #include <cstdint>
    65: #include <iterator>
    66: 
    67: using namespace llvm;
    68: 
    69: #define DEBUG_TYPE "hexagon-load-store-widening"
    70: 
    71: static cl::opt<unsigned> MaxMBBSizeForLoadStoreWidening(
    72:     "max-bb-size-for-load-store-widening", cl::Hidden, cl::init(1000),
    73:     cl::desc("Limit block size to analyze in load/store widening pass"));
    74: 
    75: namespace {
    76: 
    77: struct HexagonLoadStoreWidening {
    78:   enum WideningMode { Store, Load };
    79:   const HexagonInstrInfo *TII;
    80:   const HexagonRegisterInfo *TRI;
    81:   MachineRegisterInfo *MRI;
    82:   AliasAnalysis *AA;
    83:   MachineFunction *MF;
    84: 
    85: public:
    86:   HexagonLoadStoreWidening(const HexagonInstrInfo *TII,
    87:                            const HexagonRegisterInfo *TRI,
    88:                            MachineRegisterInfo *MRI, AliasAnalysis *AA,
    89:                            MachineFunction *MF, bool StoreMode)
    90:       : TII(TII), TRI(TRI), MRI(MRI), AA(AA), MF(MF),
    91:         Mode(StoreMode ? WideningMode::Store : WideningMode::Load),
    92:         HII(MF->getSubtarget<HexagonSubtarget>().getInstrInfo()) {}
    93: 
    94:   bool run();
    95: 
    96: private:
    97:   const bool Mode;
    98:   const unsigned MaxWideSize = 8;
    99:   const HexagonInstrInfo *HII = nullptr;
   100: 
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (27 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonLoadStoreWidening, WideningMode, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (27 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonLoadStoreWidening, WideningMode 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:   using InstrSet = SmallPtrSet<MachineInstr *, 16>;
   102:   using InstrGroup = SmallVector<MachineInstr *, 8>;
   103:   using InstrGroupList = SmallVector<InstrGroup, 8>;
   104: 
   105:   InstrSet ProcessedInsts;
   106: 
   107:   unsigned getBaseAddressRegister(const MachineInstr *MI);
   108:   int64_t getOffset(const MachineInstr *MI);
   109:   int64_t getPostIncrementValue(const MachineInstr *MI);
   110:   bool handledInstType(const MachineInstr *MI);
   111: 
   112:   void createGroup(MachineInstr *BaseInst, InstrGroup &Group);
   113:   void createGroups(MachineBasicBlock &MBB, InstrGroupList &StoreGroups);
   114:   bool processBasicBlock(MachineBasicBlock &MBB);
   115:   bool processGroup(InstrGroup &Group);
   116:   bool selectInsts(InstrGroup::iterator Begin, InstrGroup::iterator End,
   117:                    InstrGroup &OG, unsigned &TotalSize, unsigned MaxSize);
   118:   bool createWideInsts(InstrGroup &OG, InstrGroup &NG, unsigned TotalSize);
   119:   bool createWideStores(InstrGroup &OG, InstrGroup &NG, unsigned TotalSize);
   120:   bool createWideLoads(InstrGroup &OG, InstrGroup &NG, unsigned TotalSize);
   121:   bool replaceInsts(InstrGroup &OG, InstrGroup &NG);
   122:   bool areAdjacent(const MachineInstr *S1, const MachineInstr *S2);
   123:   bool canSwapInstructions(const MachineInstr *A, const MachineInstr *B);
   124:   MachineInstr *widenLoadStoreAddAsl(Register NewPairReg,
   125:                                      const MachineInstr *FirstMem,
   126:                                      const DebugLoc &DL);
   127: };
   128: 
   129: struct HexagonStoreWidening : public MachineFunctionPass {
   130:   static char ID;
   131: 
   132:   HexagonStoreWidening() : MachineFunctionPass(ID) {}
   133: 
   134:   StringRef getPassName() const override { return "Hexagon Store Widening"; }
   135: 
   136:   void getAnalysisUsage(AnalysisUsage &AU) const override {
   137:     AU.addRequired<AAResultsWrapperPass>();
   138:     AU.addPreserved<AAResultsWrapperPass>();
   139:     MachineFunctionPass::getAnalysisUsage(AU);
   140:   }
   141: 
   142:   bool runOnMachineFunction(MachineFunction &MFn) override {
   143:     if (skipFunction(MFn.getFunction()))
   144:       return false;
   145: 
   146:     auto &ST = MFn.getSubtarget<HexagonSubtarget>();
   147:     const HexagonInstrInfo *TII = ST.getInstrInfo();
   148:     const HexagonRegisterInfo *TRI = ST.getRegisterInfo();
   149:     MachineRegisterInfo *MRI = &MFn.getRegInfo();
   150:     AliasAnalysis *AA = &getAnalysis<AAResultsWrapperPass>().getAAResults();
   151: 
   152:     return HexagonLoadStoreWidening(TII, TRI, MRI, AA, &MFn, true).run();
   153:   }
   154: };
   155: 
   156: struct HexagonLoadWidening : public MachineFunctionPass {
   157:   static char ID;
   158: 
   159:   HexagonLoadWidening() : MachineFunctionPass(ID) {}
   160: 
   161:   StringRef getPassName() const override { return "Hexagon Load Widening"; }
   162: 
   163:   void getAnalysisUsage(AnalysisUsage &AU) const override {
   164:     AU.addRequired<AAResultsWrapperPass>();
   165:     AU.addPreserved<AAResultsWrapperPass>();
   166:     MachineFunctionPass::getAnalysisUsage(AU);
   167:   }
   168: 
   169:   bool runOnMachineFunction(MachineFunction &MFn) override {
   170:     if (skipFunction(MFn.getFunction()))
   171:       return false;
   172: 
   173:     auto &ST = MFn.getSubtarget<HexagonSubtarget>();
   174:     const HexagonInstrInfo *TII = ST.getInstrInfo();
   175:     const HexagonRegisterInfo *TRI = ST.getRegisterInfo();
   176:     MachineRegisterInfo *MRI = &MFn.getRegInfo();
   177:     AliasAnalysis *AA = &getAnalysis<AAResultsWrapperPass>().getAAResults();
   178:     return HexagonLoadStoreWidening(TII, TRI, MRI, AA, &MFn, false).run();
   179:   }
   180: };
   181: 
   182: char HexagonStoreWidening::ID = 0;
   183: char HexagonLoadWidening::ID = 0;
   184: 
   185: } // end anonymous namespace
   186: 
   187: INITIALIZE_PASS_BEGIN(HexagonStoreWidening, "hexagon-widen-stores",
   188:                       "Hexagon Store Widening", false, false)
   189: INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
   190: INITIALIZE_PASS_END(HexagonStoreWidening, "hexagon-widen-stores",
   191:                     "Hexagon Store Widening", false, false)
   192: 
   193: INITIALIZE_PASS_BEGIN(HexagonLoadWidening, "hexagon-widen-loads",
   194:                       "Hexagon Load Widening", false, false)
   195: INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
   196: INITIALIZE_PASS_END(HexagonLoadWidening, "hexagon-widen-loads",
   197:                     "Hexagon Load Widening", false, false)
   198: 
   199: static const MachineMemOperand &getMemTarget(const MachineInstr *MI) {
   200:   assert(!MI->memoperands_empty() && "Expecting memory operands");
```
- EN: It opens namespaces (INITIALIZE_PASS_BEGIN) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonStoreWidening, HexagonLoadWidening, which carry the state or API of this component. It declares or implements routines such as getBaseAddressRegister, getOffset, getPostIncrementValue, handledInstType, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（INITIALIZE_PASS_BEGIN），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonStoreWidening, HexagonLoadWidening 等类型，用来承载该组件的状态或接口。 这里声明或实现了 getBaseAddressRegister, getOffset, getPostIncrementValue, handledInstType, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-300 / 第 201-300 行

```cpp
   201:   return **MI->memoperands_begin();
   202: }
   203: 
   204: unsigned
   205: HexagonLoadStoreWidening::getBaseAddressRegister(const MachineInstr *MI) {
   206:   assert(HexagonLoadStoreWidening::handledInstType(MI) && "Unhandled opcode");
   207:   unsigned Base, Offset;
   208:   HII->getBaseAndOffsetPosition(*MI, Base, Offset);
   209:   const MachineOperand &MO = MI->getOperand(Base);
   210:   assert(MO.isReg() && "Expecting register operand");
   211:   return MO.getReg();
   212: }
   213: 
   214: int64_t HexagonLoadStoreWidening::getOffset(const MachineInstr *MI) {
   215:   assert(HexagonLoadStoreWidening::handledInstType(MI) && "Unhandled opcode");
   216: 
   217:   // On Hexagon, post-incs always have an offset of 0
   218:   // There is no Offset operand to post-incs
   219:   if (HII->isPostIncrement(*MI))
   220:     return 0;
   221: 
   222:   unsigned Base, Offset;
   223: 
   224:   HII->getBaseAndOffsetPosition(*MI, Base, Offset);
   225:   const MachineOperand &MO = MI->getOperand(Offset);
   226:   switch (MO.getType()) {
   227:   case MachineOperand::MO_Immediate:
   228:     return MO.getImm();
   229:   case MachineOperand::MO_GlobalAddress:
   230:     return MO.getOffset();
   231:   default:
   232:     break;
   233:   }
   234:   llvm_unreachable("Expecting an immediate or global operand");
   235: }
   236: 
   237: inline int64_t
   238: HexagonLoadStoreWidening::getPostIncrementValue(const MachineInstr *MI) {
   239:   unsigned Base, PostIncIdx;
   240:   HII->getBaseAndOffsetPosition(*MI, Base, PostIncIdx);
   241:   const MachineOperand &MO = MI->getOperand(PostIncIdx);
   242:   return MO.getImm();
   243: }
   244: 
   245: // Filtering function: any loads/stores whose opcodes are not "approved" of by
   246: // this function will not be subjected to widening.
   247: inline bool HexagonLoadStoreWidening::handledInstType(const MachineInstr *MI) {
   248:   unsigned Opc = MI->getOpcode();
   249:   if (Mode == WideningMode::Store) {
   250:     switch (Opc) {
   251:     case Hexagon::S4_storeirb_io:
   252:     case Hexagon::S4_storeirh_io:
   253:     case Hexagon::S4_storeiri_io:
   254:     case Hexagon::S2_storeri_io:
   255:       // Base address must be a register. (Implement FI later.)
   256:       return MI->getOperand(0).isReg();
   257:     case Hexagon::S2_storeri_pi:
   258:       return MI->getOperand(1).isReg();
   259:     }
   260:   } else {
   261:     // LoadWidening for 8 and 16 bit loads needs 2x instructions to replace x
   262:     // loads. So we only widen 32 bit loads as we don't need to select the
   263:     // right bits with AND & SHIFT ops.
   264:     switch (Opc) {
   265:     case Hexagon::L2_loadri_io:
   266:       // Base address must be a register and offset must be immediate.
   267:       return !MI->memoperands_empty() && MI->getOperand(1).isReg() &&
   268:              MI->getOperand(2).isImm();
   269:     case Hexagon::L2_loadri_pi:
   270:       return !MI->memoperands_empty() && MI->getOperand(2).isReg();
   271:     }
   272:   }
   273:   return false;
   274: }
   275: 
   276: static void addDefsUsesToList(const MachineInstr *MI,
   277:                               DenseSet<Register> &RegDefs,
   278:                               DenseSet<Register> &RegUses) {
   279:   for (const auto &Op : MI->operands()) {
   280:     if (!Op.isReg())
   281:       continue;
   282:     if (Op.isDef())
   283:       RegDefs.insert(Op.getReg());
   284:     if (Op.readsReg())
   285:       RegUses.insert(Op.getReg());
   286:   }
   287: }
   288: 
   289: bool HexagonLoadStoreWidening::canSwapInstructions(const MachineInstr *A,
   290:                                                    const MachineInstr *B) {
   291:   DenseSet<Register> ARegDefs;
   292:   DenseSet<Register> ARegUses;
   293:   addDefsUsesToList(A, ARegDefs, ARegUses);
   294:   if (A->mayLoadOrStore() && B->mayLoadOrStore() &&
   295:       (A->mayStore() || B->mayStore()) && A->mayAlias(AA, *B, true))
   296:     return false;
   297:   for (const auto &BOp : B->operands()) {
   298:     if (!BOp.isReg())
   299:       continue;
   300:     if ((BOp.isDef() || BOp.readsReg()) && ARegDefs.contains(BOp.getReg()))
```
- EN: It declares or implements routines such as memoperands_begin, HexagonLoadStoreWidening::getBaseAddressRegister, assert, getBaseAndOffsetPosition, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 memoperands_begin, HexagonLoadStoreWidening::getBaseAddressRegister, assert, getBaseAndOffsetPosition, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:       return false;
   302:     if (BOp.isDef() && ARegUses.contains(BOp.getReg()))
   303:       return false;
   304:   }
   305:   return true;
   306: }
   307: 
   308: // Inspect a machine basic block, and generate groups out of loads/stores
   309: // encountered in the block.
   310: //
   311: // A load/store group is a group of loads or stores that use the same base
   312: // register, and which can be reordered within that group without altering the
   313: // semantics of the program.  A single group could be widened as
   314: // a whole, if there existed a single load/store instruction with the same
   315: // semantics as the entire group.  In many cases, a single group may need more
   316: // than one wide load or store.
   317: void HexagonLoadStoreWidening::createGroups(MachineBasicBlock &MBB,
   318:                                             InstrGroupList &StoreGroups) {
   319:   // Traverse all instructions and if we encounter
   320:   // a load/store, then try to create a group starting at that instruction
   321:   // i.e. a sequence of independent loads/stores that can be widened.
   322:   for (auto I = MBB.begin(); I != MBB.end(); ++I) {
   323:     MachineInstr *MI = &(*I);
   324:     if (!handledInstType(MI))
   325:       continue;
   326:     if (ProcessedInsts.count(MI))
   327:       continue;
   328: 
   329:     // Found a store.  Try to create a store group.
   330:     InstrGroup G;
   331:     createGroup(MI, G);
   332:     if (G.size() > 1)
   333:       StoreGroups.push_back(G);
   334:   }
   335: }
   336: 
   337: // Create a single load/store group.  The insts need to be independent between
   338: // themselves, and also there cannot be other instructions between them
   339: // that could read or modify storage being read from or stored into.
   340: void HexagonLoadStoreWidening::createGroup(MachineInstr *BaseInst,
   341:                                            InstrGroup &Group) {
   342:   assert(handledInstType(BaseInst) && "Unexpected instruction");
   343:   unsigned BaseReg = getBaseAddressRegister(BaseInst);
   344:   InstrGroup Other;
   345: 
   346:   Group.push_back(BaseInst);
   347:   LLVM_DEBUG(dbgs() << "BaseInst: "; BaseInst->dump());
   348:   auto End = BaseInst->getParent()->end();
   349:   auto I = BaseInst->getIterator();
   350: 
   351:   while (true) {
   352:     I = std::next(I);
   353:     if (I == End)
   354:       break;
   355:     MachineInstr *MI = &(*I);
   356: 
   357:     // Assume calls are aliased to everything.
   358:     if (MI->isCall() || MI->hasUnmodeledSideEffects() ||
   359:         MI->hasOrderedMemoryRef())
   360:       return;
   361: 
   362:     if (!handledInstType(MI)) {
   363:       if (MI->mayLoadOrStore())
   364:         Other.push_back(MI);
   365:       continue;
   366:     }
   367: 
   368:     // We have a handledInstType instruction
   369:     // If this load/store instruction is aliased with anything already in the
   370:     // group, terminate the group now.
   371:     for (auto GI : Group)
   372:       if (GI->mayAlias(AA, *MI, true))
   373:         return;
   374:     if (Mode == WideningMode::Load) {
   375:       // Check if current load MI can be moved to the first load instruction
   376:       // in Group. If any load instruction aliases with memory instructions in
   377:       // Other, terminate the group.
   378:       for (auto MemI : Other)
   379:         if (!canSwapInstructions(MI, MemI))
   380:           return;
   381:     } else {
   382:       // Check if store instructions in the group can be moved to current
   383:       // store MI. If any store instruction aliases with memory instructions
   384:       // in Other, terminate the group.
   385:       for (auto MemI : Other) {
   386:         if (std::distance(Group.back()->getIterator(), MemI->getIterator()) <=
   387:             0)
   388:           continue;
   389:         for (auto GI : Group)
   390:           if (!canSwapInstructions(MemI, GI))
   391:             return;
   392:       }
   393:     }
   394: 
   395:     unsigned BR = getBaseAddressRegister(MI);
   396:     if (BR == BaseReg) {
   397:       LLVM_DEBUG(dbgs() << "Added MI to group: "; MI->dump());
   398:       Group.push_back(MI);
   399:       ProcessedInsts.insert(MI);
   400:     }
```
- EN: It declares or implements routines such as HexagonLoadStoreWidening::createGroups, end, createGroup, HexagonLoadStoreWidening::createGroup, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonLoadStoreWidening::createGroups, end, createGroup, HexagonLoadStoreWidening::createGroup, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:   } // while
   402: }
   403: 
   404: // Check if load/store instructions S1 and S2 are adjacent.  More precisely,
   405: // S2 has to access memory immediately following that accessed by S1.
   406: bool HexagonLoadStoreWidening::areAdjacent(const MachineInstr *S1,
   407:                                            const MachineInstr *S2) {
   408:   if (!handledInstType(S1) || !handledInstType(S2))
   409:     return false;
   410: 
   411:   const MachineMemOperand &S1MO = getMemTarget(S1);
   412: 
   413:   // Currently only handling immediate stores.
   414:   int Off1 = getOffset(S1);
   415:   int Off2 = getOffset(S2);
   416: 
   417:   return (Off1 >= 0) ? Off1 + S1MO.getSize().getValue() == unsigned(Off2)
   418:                      : int(Off1 + S1MO.getSize().getValue()) == Off2;
   419: }
   420: 
   421: /// Given a sequence of adjacent loads/stores, and a maximum size of a single
   422: /// wide inst, pick a group of insts that can be replaced by a single load/store
   423: /// of size not exceeding MaxSize.  The selected sequence will be recorded
   424: /// in OG ("old group" of instructions).
   425: /// OG should be empty on entry, and should be left empty if the function
   426: /// fails.
   427: bool HexagonLoadStoreWidening::selectInsts(InstrGroup::iterator Begin,
   428:                                            InstrGroup::iterator End,
   429:                                            InstrGroup &OG, unsigned &TotalSize,
   430:                                            unsigned MaxSize) {
   431:   assert(Begin != End && "No instructions to analyze");
   432:   assert(OG.empty() && "Old group not empty on entry");
   433: 
   434:   if (std::distance(Begin, End) <= 1)
   435:     return false;
   436: 
   437:   MachineInstr *FirstMI = *Begin;
   438:   assert(!FirstMI->memoperands_empty() && "Expecting some memory operands");
   439:   const MachineMemOperand &FirstMMO = getMemTarget(FirstMI);
   440:   if (!FirstMMO.getType().isValid())
   441:     return false;
   442: 
   443:   unsigned Alignment = FirstMMO.getAlign().value();
   444:   unsigned SizeAccum = FirstMMO.getSize().getValue();
   445:   unsigned FirstOffset = getOffset(FirstMI);
   446: 
   447:   // The initial value of SizeAccum should always be a power of 2.
   448:   assert(isPowerOf2_32(SizeAccum) && "First store size not a power of 2");
   449: 
   450:   // If the size of the first store equals to or exceeds the limit, do nothing.
   451:   if (SizeAccum >= MaxSize)
   452:     return false;
   453: 
   454:   // If the size of the first load/store is greater than or equal to the address
   455:   // stored to, then the inst cannot be made any wider.
   456:   if (SizeAccum >= Alignment) {
   457:     LLVM_DEBUG(
   458:         dbgs() << "Size of load/store greater than equal to its alignment\n");
   459:     return false;
   460:   }
   461: 
   462:   // The offset of a load/store will put restrictions on how wide the inst can
   463:   // be.  Offsets in loads/stores of size 2^n bytes need to have the n lowest
   464:   // bits be 0.  If the first inst already exhausts the offset limits, quit.
   465:   // Test this by checking if the next wider size would exceed the limit.
   466:   // For post-increment instructions, the increment amount needs to follow the
   467:   // same rule.
   468:   unsigned OffsetOrIncVal = 0;
   469:   if (HII->isPostIncrement(*FirstMI))
   470:     OffsetOrIncVal = getPostIncrementValue(FirstMI);
   471:   else
   472:     OffsetOrIncVal = FirstOffset;
   473:   if ((2 * SizeAccum - 1) & OffsetOrIncVal) {
   474:     LLVM_DEBUG(dbgs() << "Instruction cannot be widened as the offset/postinc"
   475:                       << " value: " << getPostIncrementValue(FirstMI)
   476:                       << " is invalid in the widened version\n");
   477:     return false;
   478:   }
   479: 
   480:   OG.push_back(FirstMI);
   481:   MachineInstr *S1 = FirstMI;
   482: 
   483:   // Pow2Num will be the largest number of elements in OG such that the sum
   484:   // of sizes of loads/stores 0...Pow2Num-1 will be a power of 2.
   485:   unsigned Pow2Num = 1;
   486:   unsigned Pow2Size = SizeAccum;
   487:   bool HavePostInc = HII->isPostIncrement(*S1);
   488: 
   489:   // Be greedy: keep accumulating insts as long as they are to adjacent
   490:   // memory locations, and as long as the total number of bytes stored
   491:   // does not exceed the limit (MaxSize).
   492:   // Keep track of when the total size covered is a power of 2, since
   493:   // this is a size a single load/store can cover.
   494:   for (InstrGroup::iterator I = Begin + 1; I != End; ++I) {
   495:     MachineInstr *S2 = *I;
   496:     // Insts are sorted, so if S1 and S2 are not adjacent, there won't be
   497:     // any other store to fill the "hole".
   498:     if (!areAdjacent(S1, S2))
   499:       break;
   500: 
```
- EN: It declares or implements routines such as HexagonLoadStoreWidening::areAdjacent, getMemTarget, getOffset, OG, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonLoadStoreWidening::areAdjacent, getMemTarget, getOffset, OG, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:     // Cannot widen two post increments, need to return two registers
   502:     // with incremented values
   503:     if (HavePostInc && HII->isPostIncrement(*S2))
   504:       break;
   505: 
   506:     unsigned S2Size = getMemTarget(S2).getSize().getValue();
   507:     if (SizeAccum + S2Size > std::min(MaxSize, Alignment))
   508:       break;
   509: 
   510:     OG.push_back(S2);
   511:     SizeAccum += S2Size;
   512:     if (isPowerOf2_32(SizeAccum)) {
   513:       Pow2Num = OG.size();
   514:       Pow2Size = SizeAccum;
   515:     }
   516:     if ((2 * Pow2Size - 1) & FirstOffset)
   517:       break;
   518: 
   519:     S1 = S2;
   520:   }
   521: 
   522:   // The insts don't add up to anything that can be widened.  Clean up.
   523:   if (Pow2Num <= 1) {
   524:     OG.clear();
   525:     return false;
   526:   }
   527: 
   528:   // Only leave the loads/stores being widened.
   529:   OG.resize(Pow2Num);
   530:   TotalSize = Pow2Size;
   531:   return true;
   532: }
   533: 
   534: /// Given an "old group" OG of insts, create a "new group" NG of instructions
   535: /// to replace them.
   536: bool HexagonLoadStoreWidening::createWideInsts(InstrGroup &OG, InstrGroup &NG,
   537:                                                unsigned TotalSize) {
   538:   if (Mode == WideningMode::Store) {
   539:     return createWideStores(OG, NG, TotalSize);
   540:   }
   541:   return createWideLoads(OG, NG, TotalSize);
   542: }
   543: 
   544: // check for the pattern:
   545: // %def = S2_addasl_rrri %base, %offset, #imm
   546: // %other = L2_loadrd_io %def, 0
   547: // to
   548: // %other = L4_loadrd_rr %base, %offset, #imm
   549: // If addasl does not define the base of the load,
   550: // do a normal widening.
   551: // Same behavior followed for store.
   552: MachineInstr *HexagonLoadStoreWidening::widenLoadStoreAddAsl(
   553:     Register NewPairReg, const MachineInstr *FirstMem, const DebugLoc &DL) {
   554: 
   555:   unsigned Base, Offset;
   556:   HII->getBaseAndOffsetPosition(*FirstMem, Base, Offset);
   557:   auto BaseOp = FirstMem->getOperand(Base);
   558:   auto OffMO = FirstMem->getOperand(Offset);
   559:   Register BaseReg = BaseOp.getReg();
   560: 
   561:   MachineInstr *AddaslDef = nullptr;
   562:   if (Register::isVirtualRegister(BaseReg))
   563:     AddaslDef = MRI->getVRegDef(BaseReg);
   564: 
   565:   // Skip through trivial copies to the defining addasl.
   566:   while (AddaslDef && AddaslDef->getOpcode() == Hexagon::COPY) {
   567:     const MachineOperand &Src = AddaslDef->getOperand(1);
   568:     if (!Src.isReg() || !Register::isVirtualRegister(Src.getReg()))
   569:       break;
   570:     AddaslDef = MRI->getUniqueVRegDef(Src.getReg());
   571:   }
   572: 
   573:   if (!AddaslDef || AddaslDef->getOpcode() != Hexagon::S2_addasl_rrri ||
   574:       (OffMO.isImm() && OffMO.getImm() != 0))
   575:     return nullptr;
   576: 
   577:   int64_t ImmVal = AddaslDef->getOperand(3).getImm();
   578:   if (ImmVal > 3 || ImmVal < 0)
   579:     return nullptr;
   580: 
   581:   MachineInstr *WidenedInstr = nullptr;
   582:   auto Reg1 = AddaslDef->getOperand(1).getReg();
   583:   auto Reg2 = AddaslDef->getOperand(2).getReg();
   584:   unsigned Reg2SubReg = AddaslDef->getOperand(2).getSubReg();
   585: 
   586:   // S4_storerd_rr and L4_loadrd_rr require IntRegs for Reg2.
   587:   const TargetRegisterClass *Reg2RC = MRI->getRegClass(Reg2);
   588:   // Cannot widen if:
   589:   // 1. DoubleReg without subreg, or
   590:   // 2. Neither DoubleReg nor IntReg
   591:   if ((Hexagon::DoubleRegsRegClass.hasSubClassEq(Reg2RC) && Reg2SubReg == 0) ||
   592:       (!Hexagon::DoubleRegsRegClass.hasSubClassEq(Reg2RC) &&
   593:        !Hexagon::IntRegsRegClass.hasSubClassEq(Reg2RC)))
   594:     return nullptr;
   595: 
   596:   auto Reg3 = AddaslDef->getOperand(3).getImm();
   597:   MachineInstrBuilder MIB;
   598: 
   599:   if (Mode == WideningMode::Load) {
   600:     auto LdOp = FirstMem->getOperand(0);
```
- EN: It declares or implements routines such as getMemTarget, push_back, size, clear, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getMemTarget, push_back, size, clear, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:     MIB = BuildMI(*MF, DL, TII->get(Hexagon::L4_loadrd_rr))
   602:               .addDef(NewPairReg, getKillRegState(LdOp.isKill()),
   603:                       LdOp.getSubReg())
   604:               .addReg(Reg1)
   605:               .addReg(Reg2, {}, Reg2SubReg)
   606:               .addImm(Reg3);
   607:   } else {
   608:     MIB = BuildMI(*MF, DL, TII->get(Hexagon::S4_storerd_rr))
   609:               .addReg(Reg1)
   610:               .addReg(Reg2, {}, Reg2SubReg)
   611:               .addImm(Reg3)
   612:               .addReg(NewPairReg);
   613:   }
   614: 
   615:   // Reset kill flags of the addasl instruction
   616:   AddaslDef->getOperand(1).setIsKill(false);
   617:   AddaslDef->getOperand(2).setIsKill(false);
   618: 
   619:   WidenedInstr = *&MIB;
   620:   return WidenedInstr;
   621: }
   622: 
   623: /// Given an "old group" OG of stores, create a "new group" NG of instructions
   624: /// to replace them.  Ideally, NG would only have a single instruction in it,
   625: /// but that may only be possible for store-immediate.
   626: bool HexagonLoadStoreWidening::createWideStores(InstrGroup &OG, InstrGroup &NG,
   627:                                                 unsigned TotalSize) {
   628:   // XXX Current limitations:
   629:   // - only handle a TotalSize of up to 8
   630: 
   631:   LLVM_DEBUG(dbgs() << "Creating wide stores\n");
   632:   if (TotalSize > MaxWideSize)
   633:     return false;
   634: 
   635:   uint64_t Acc = 0; // Value accumulator.
   636:   unsigned Shift = 0;
   637:   bool HaveImm = false;
   638:   bool HaveReg = false;
   639: 
   640:   for (MachineInstr *MI : OG) {
   641:     const MachineMemOperand &MMO = getMemTarget(MI);
   642:     MachineOperand &SO = HII->isPostIncrement(*MI)
   643:                              ? MI->getOperand(3)
   644:                              : MI->getOperand(2); // Source.
   645:     unsigned NBits;
   646:     uint64_t Mask;
   647:     uint64_t Val;
   648: 
   649:     switch (SO.getType()) {
   650:     case MachineOperand::MO_Immediate:
   651:       LLVM_DEBUG(dbgs() << "Have store immediate\n");
   652:       HaveImm = true;
   653: 
   654:       NBits = MMO.getSizeInBits().toRaw();
   655:       Mask = (0xFFFFFFFFFFFFFFFFU >> (64 - NBits));
   656:       Val = (SO.getImm() & Mask) << Shift;
   657:       Acc |= Val;
   658:       Shift += NBits;
   659:       break;
   660:     case MachineOperand::MO_Register:
   661:       HaveReg = true;
   662:       break;
   663:     default:
   664:       LLVM_DEBUG(dbgs() << "Unhandled store\n");
   665:       return false;
   666:     }
   667:   }
   668: 
   669:   if (HaveImm && HaveReg) {
   670:     LLVM_DEBUG(dbgs() << "Cannot merge store register and store imm\n");
   671:     return false;
   672:   }
   673: 
   674:   MachineInstr *FirstSt = OG.front();
   675:   DebugLoc DL = OG.back()->getDebugLoc();
   676:   const MachineMemOperand &OldM = getMemTarget(FirstSt);
   677:   MachineMemOperand *NewM =
   678:       MF->getMachineMemOperand(OldM.getPointerInfo(), OldM.getFlags(),
   679:                                TotalSize, OldM.getAlign(), OldM.getAAInfo());
   680:   MachineInstr *StI;
   681:   MachineOperand &MR =
   682:       (HII->isPostIncrement(*FirstSt) ? FirstSt->getOperand(1)
   683:                                       : FirstSt->getOperand(0));
   684:   auto SecondSt = OG.back();
   685:   if (HaveReg) {
   686:     MachineOperand FReg =
   687:         (HII->isPostIncrement(*FirstSt) ? FirstSt->getOperand(3)
   688:                                         : FirstSt->getOperand(2));
   689:     // Post increments appear first in the sorted group.
   690:     // Cannot have a post increment for the second instruction
   691:     assert(!HII->isPostIncrement(*SecondSt) && "Unexpected PostInc");
   692:     MachineOperand SReg = SecondSt->getOperand(2);
   693:     assert(FReg.isReg() && SReg.isReg() &&
   694:            "Cannot merge store register and store imm");
   695:     const MCInstrDesc &CombD = TII->get(Hexagon::A2_combinew);
   696:     Register VReg =
   697:         MF->getRegInfo().createVirtualRegister(&Hexagon::DoubleRegsRegClass);
   698:     MachineInstr *CombI = BuildMI(*MF, DL, CombD, VReg).add(SReg).add(FReg);
   699:     NG.push_back(CombI);
   700: 
```
- EN: It declares or implements routines such as addImm, getOperand, HexagonLoadStoreWidening::createWideStores, LLVM_DEBUG, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addImm, getOperand, HexagonLoadStoreWidening::createWideStores, LLVM_DEBUG, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:     if (FirstSt->getOpcode() == Hexagon::S2_storeri_pi) {
   702:       const MCInstrDesc &StD = TII->get(Hexagon::S2_storerd_pi);
   703:       auto IncDestMO = FirstSt->getOperand(0);
   704:       auto IncMO = FirstSt->getOperand(2);
   705:       StI =
   706:           BuildMI(*MF, DL, StD).add(IncDestMO).add(MR).add(IncMO).addReg(VReg);
   707:     } else {
   708:       StI = widenLoadStoreAddAsl(VReg, FirstSt, DL);
   709:       if (!StI) {
   710:         const MCInstrDesc &StD = TII->get(Hexagon::S2_storerd_io);
   711:         auto OffMO = FirstSt->getOperand(1);
   712:         StI = BuildMI(*MF, DL, StD).add(MR).add(OffMO).addReg(VReg);
   713:       }
   714:     }
   715:     StI->addMemOperand(*MF, NewM);
   716:     NG.push_back(StI);
   717:     return true;
   718:   }
   719: 
   720:   // Handle store immediates
   721:   // There are no post increment store immediates on Hexagon
   722:   assert(!HII->isPostIncrement(*FirstSt) && "Unexpected PostInc");
   723:   auto Off = FirstSt->getOperand(1).getImm();
   724:   if (TotalSize == 8) {
   725:     // Create vreg = A2_tfrsi #Acc; nreg = combine(#s32, vreg); memd = nreg
   726:     uint64_t Mask = 0xFFFFFFFFU;
   727:     int LowerAcc = int(Mask & Acc);
   728:     int UpperAcc = Acc >> 32;
   729:     Register DReg =
   730:         MF->getRegInfo().createVirtualRegister(&Hexagon::DoubleRegsRegClass);
   731:     MachineInstr *CombI;
   732:     if (Acc != 0) {
   733:       const MCInstrDesc &TfrD = TII->get(Hexagon::A2_tfrsi);
   734:       const TargetRegisterClass *RC = TII->getRegClass(TfrD, 0);
   735:       Register VReg = MF->getRegInfo().createVirtualRegister(RC);
   736:       MachineInstr *TfrI = BuildMI(*MF, DL, TfrD, VReg).addImm(LowerAcc);
   737:       NG.push_back(TfrI);
   738:       const MCInstrDesc &CombD = TII->get(Hexagon::A4_combineir);
   739:       CombI = BuildMI(*MF, DL, CombD, DReg)
   740:                   .addImm(UpperAcc)
   741:                   .addReg(VReg, RegState::Kill);
   742:     }
   743:     // If immediates are 0, we do not need A2_tfrsi
   744:     else {
   745:       const MCInstrDesc &CombD = TII->get(Hexagon::A4_combineii);
   746:       CombI = BuildMI(*MF, DL, CombD, DReg).addImm(0).addImm(0);
   747:     }
   748:     NG.push_back(CombI);
   749:     const MCInstrDesc &StD = TII->get(Hexagon::S2_storerd_io);
   750:     StI =
   751:         BuildMI(*MF, DL, StD).add(MR).addImm(Off).addReg(DReg, RegState::Kill);
   752:   } else if (Acc < 0x10000) {
   753:     // Create mem[hw] = #Acc
   754:     unsigned WOpc = (TotalSize == 2)   ? Hexagon::S4_storeirh_io
   755:                     : (TotalSize == 4) ? Hexagon::S4_storeiri_io
   756:                                        : 0;
   757:     assert(WOpc && "Unexpected size");
   758: 
   759:     int Val = (TotalSize == 2) ? int16_t(Acc) : int(Acc);
   760:     const MCInstrDesc &StD = TII->get(WOpc);
   761:     StI = BuildMI(*MF, DL, StD).add(MR).addImm(Off).addImm(Val);
   762:   } else {
   763:     // Create vreg = A2_tfrsi #Acc; mem[hw] = vreg
   764:     const MCInstrDesc &TfrD = TII->get(Hexagon::A2_tfrsi);
   765:     const TargetRegisterClass *RC = TII->getRegClass(TfrD, 0);
   766:     Register VReg = MF->getRegInfo().createVirtualRegister(RC);
   767:     MachineInstr *TfrI = BuildMI(*MF, DL, TfrD, VReg).addImm(int(Acc));
   768:     NG.push_back(TfrI);
   769: 
   770:     unsigned WOpc = (TotalSize == 2)   ? Hexagon::S2_storerh_io
   771:                     : (TotalSize == 4) ? Hexagon::S2_storeri_io
   772:                                        : 0;
   773:     assert(WOpc && "Unexpected size");
   774: 
   775:     const MCInstrDesc &StD = TII->get(WOpc);
   776:     StI =
   777:         BuildMI(*MF, DL, StD).add(MR).addImm(Off).addReg(VReg, RegState::Kill);
   778:   }
   779:   StI->addMemOperand(*MF, NewM);
   780:   NG.push_back(StI);
   781: 
   782:   return true;
   783: }
   784: 
   785: /// Given an "old group" OG of loads, create a "new group" NG of instructions
   786: /// to replace them.  Ideally, NG would only have a single instruction in it,
   787: /// but that may only be possible for double register loads.
   788: bool HexagonLoadStoreWidening::createWideLoads(InstrGroup &OG, InstrGroup &NG,
   789:                                                unsigned TotalSize) {
   790:   LLVM_DEBUG(dbgs() << "Creating wide loads\n");
   791:   // XXX Current limitations:
   792:   // - only expect stores of immediate values in OG,
   793:   // - only handle a TotalSize of up to 8
   794:   if (TotalSize > MaxWideSize)
   795:     return false;
   796:   assert(OG.size() == 2 && "Expecting two elements in Instruction Group.");
   797: 
   798:   MachineInstr *FirstLd = OG.front();
   799:   const MachineMemOperand &OldM = getMemTarget(FirstLd);
   800:   MachineMemOperand *NewM =
```
- EN: It declares or implements routines such as get, getOperand, BuildMI, widenLoadStoreAddAsl, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 get, getOperand, BuildMI, widenLoadStoreAddAsl, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801:       MF->getMachineMemOperand(OldM.getPointerInfo(), OldM.getFlags(),
   802:                                TotalSize, OldM.getAlign(), OldM.getAAInfo());
   803: 
   804:   MachineOperand &MR = FirstLd->getOperand(0);
   805:   MachineOperand &MRBase =
   806:       (HII->isPostIncrement(*FirstLd) ? FirstLd->getOperand(2)
   807:                                       : FirstLd->getOperand(1));
   808:   DebugLoc DL = OG.back()->getDebugLoc();
   809: 
   810:   // Create the double register Load Instruction.
   811:   Register NewMR = MRI->createVirtualRegister(&Hexagon::DoubleRegsRegClass);
   812:   MachineInstr *LdI;
   813: 
   814:   // Post increments appear first in the sorted group
   815:   if (FirstLd->getOpcode() == Hexagon::L2_loadri_pi) {
   816:     auto IncDestMO = FirstLd->getOperand(1);
   817:     auto IncMO = FirstLd->getOperand(3);
   818:     LdI = BuildMI(*MF, DL, TII->get(Hexagon::L2_loadrd_pi))
   819:               .addDef(NewMR, getKillRegState(MR.isKill()), MR.getSubReg())
   820:               .add(IncDestMO)
   821:               .add(MRBase)
   822:               .add(IncMO);
   823:     LdI->addMemOperand(*MF, NewM);
   824:   } else {
   825:     // Try rr widening with addasl-defined base; otherwise fall back to io.
   826:     MachineInstr *RR = widenLoadStoreAddAsl(NewMR, FirstLd, DL);
   827:     if (RR) {
   828:       RR->addMemOperand(*MF, NewM);
   829:       LdI = RR;
   830:     } else {
   831:       auto OffMO = FirstLd->getOperand(2);
   832:       LdI = BuildMI(*MF, DL, TII->get(Hexagon::L2_loadrd_io))
   833:                 .addDef(NewMR, getKillRegState(MR.isKill()), MR.getSubReg())
   834:                 .add(MRBase)
   835:                 .add(OffMO);
   836:       LdI->addMemOperand(*MF, NewM);
   837:     }
   838:   }
   839:   NG.push_back(LdI);
   840: 
   841:   auto getHalfReg = [&](MachineInstr *DoubleReg, unsigned SubReg,
   842:                         MachineInstr *DstReg) {
   843:     Register DestReg = DstReg->getOperand(0).getReg();
   844:     return BuildMI(*MF, DL, TII->get(Hexagon::COPY), DestReg)
   845:         .addReg(NewMR, getKillRegState(LdI->isKill()), SubReg);
   846:   };
   847: 
   848:   MachineInstr *LdI_lo = getHalfReg(LdI, Hexagon::isub_lo, FirstLd);
   849:   MachineInstr *LdI_hi = getHalfReg(LdI, Hexagon::isub_hi, OG.back());
   850:   NG.push_back(LdI_lo);
   851:   NG.push_back(LdI_hi);
   852: 
   853:   return true;
   854: }
   855: 
   856: // Replace instructions from the old group OG with instructions from the
   857: // new group NG.  Conceptually, remove all instructions in OG, and then
   858: // insert all instructions in NG, starting at where the first instruction
   859: // from OG was (in the order in which they appeared in the basic block).
   860: // (The ordering in OG does not have to match the order in the basic block.)
   861: bool HexagonLoadStoreWidening::replaceInsts(InstrGroup &OG, InstrGroup &NG) {
   862:   LLVM_DEBUG({
   863:     dbgs() << "Replacing:\n";
   864:     for (auto I : OG)
   865:       dbgs() << "  " << *I;
   866:     dbgs() << "with\n";
   867:     for (auto I : NG)
   868:       dbgs() << "  " << *I;
   869:   });
   870: 
   871:   MachineBasicBlock *MBB = OG.back()->getParent();
   872:   MachineBasicBlock::iterator InsertAt = MBB->end();
   873: 
   874:   // Need to establish the insertion point.
   875:   // For loads the best one is right before the first load in the OG,
   876:   // but in the order in which the insts occur in the program list.
   877:   // For stores the best point is right after the last store in the OG.
   878:   // Since the ordering in OG does not correspond
   879:   // to the order in the program list, we need to do some work to find
   880:   // the insertion point.
   881: 
   882:   // Create a set of all instructions in OG (for quick lookup).
   883:   InstrSet OldMemInsts(llvm::from_range, OG);
   884: 
   885:   if (Mode == WideningMode::Load) {
   886:     // Find the first load instruction in the block that is present in OG.
   887:     for (auto &I : *MBB) {
   888:       if (OldMemInsts.count(&I)) {
   889:         InsertAt = I;
   890:         break;
   891:       }
   892:     }
   893: 
   894:     assert((InsertAt != MBB->end()) && "Cannot locate any load from the group");
   895: 
   896:     for (auto *I : NG)
   897:       MBB->insert(InsertAt, I);
   898:   } else {
   899:     // Find the last store instruction in the block that is present in OG.
   900:     auto I = MBB->rbegin();
```
- EN: It declares or implements routines such as getMachineMemOperand, getOperand, isPostIncrement, back, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getMachineMemOperand, getOperand, isPostIncrement, back, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening，说明了它与同级后端组件的连接关系。

### Lines 901-992 / 第 901-992 行

```cpp
   901:     for (; I != MBB->rend(); ++I) {
   902:       if (OldMemInsts.count(&(*I))) {
   903:         InsertAt = (*I).getIterator();
   904:         break;
   905:       }
   906:     }
   907: 
   908:     assert((I != MBB->rend()) && "Cannot locate any store from the group");
   909: 
   910:     for (auto I = NG.rbegin(); I != NG.rend(); ++I)
   911:       MBB->insertAfter(InsertAt, *I);
   912:   }
   913: 
   914:   for (auto *I : OG)
   915:     I->eraseFromParent();
   916: 
   917:   return true;
   918: }
   919: 
   920: // Break up the group into smaller groups, each of which can be replaced by
   921: // a single wide load/store.  Widen each such smaller group and replace the old
   922: // instructions with the widened ones.
   923: bool HexagonLoadStoreWidening::processGroup(InstrGroup &Group) {
   924:   bool Changed = false;
   925:   InstrGroup::iterator I = Group.begin(), E = Group.end();
   926:   InstrGroup OG, NG; // Old and new groups.
   927:   unsigned CollectedSize;
   928: 
   929:   while (I != E) {
   930:     OG.clear();
   931:     NG.clear();
   932: 
   933:     bool Succ = selectInsts(I++, E, OG, CollectedSize, MaxWideSize) &&
   934:                 createWideInsts(OG, NG, CollectedSize) && replaceInsts(OG, NG);
   935:     if (!Succ)
   936:       continue;
   937: 
   938:     assert(OG.size() > 1 && "Created invalid group");
   939:     assert(std::distance(I, E) + 1 >= int(OG.size()) && "Too many elements");
   940:     I += OG.size() - 1;
   941: 
   942:     Changed = true;
   943:   }
   944: 
   945:   return Changed;
   946: }
   947: 
   948: // Process a single basic block: create the load/store groups, and replace them
   949: // with the widened insts, if possible.  Processing of each basic block
   950: // is independent from processing of any other basic block.  This transfor-
   951: // mation could be stopped after having processed any basic block without
   952: // any ill effects (other than not having performed widening in the unpro-
   953: // cessed blocks).  Also, the basic blocks can be processed in any order.
   954: bool HexagonLoadStoreWidening::processBasicBlock(MachineBasicBlock &MBB) {
   955:   InstrGroupList SGs;
   956:   bool Changed = false;
   957: 
   958:   // To prevent long compile time check for max BB size.
   959:   if (MBB.size() > MaxMBBSizeForLoadStoreWidening)
   960:     return false;
   961: 
   962:   createGroups(MBB, SGs);
   963: 
   964:   auto Less = [this](const MachineInstr *A, const MachineInstr *B) -> bool {
   965:     return getOffset(A) < getOffset(B);
   966:   };
   967:   for (auto &G : SGs) {
   968:     assert(G.size() > 1 && "Group with fewer than 2 elements");
   969:     llvm::sort(G, Less);
   970: 
   971:     Changed |= processGroup(G);
   972:   }
   973: 
   974:   return Changed;
   975: }
   976: 
   977: bool HexagonLoadStoreWidening::run() {
   978:   bool Changed = false;
   979: 
   980:   for (auto &B : *MF)
   981:     Changed |= processBasicBlock(B);
   982: 
   983:   return Changed;
   984: }
   985: 
   986: FunctionPass *llvm::createHexagonStoreWidening() {
   987:   return new HexagonStoreWidening();
   988: }
   989: 
   990: FunctionPass *llvm::createHexagonLoadWidening() {
   991:   return new HexagonLoadWidening();
   992: }
```
- EN: It declares or implements routines such as rend, getIterator, assert, insertAfter, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoadStoreWidening, HexagonStoreWidening, HexagonLoadWidening, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 rend, getIterator, assert, insertAfter, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoadStoreWidening, HexagonStoreWidening, HexagonLoadWidening，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SmallPtrSet.h, llvm/Analysis/AliasAnalysis.h, llvm/Analysis/MemoryLocation.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, ... (27 total)`
- Hexagon symbols / Hexagon 符号: `HexagonLoadStoreWidening, HexagonStoreWidening, HexagonLoadWidening, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
