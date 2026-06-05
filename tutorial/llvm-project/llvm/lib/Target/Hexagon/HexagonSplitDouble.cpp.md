# HexagonSplitDouble.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonSplitDouble.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon splitting of double-register operations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及寄存器模型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonSplitDouble.cpp ---------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "Hexagon.h"
    10: #include "HexagonInstrInfo.h"
    11: #include "HexagonRegisterInfo.h"
    12: #include "HexagonSubtarget.h"
    13: #include "llvm/ADT/BitVector.h"
    14: #include "llvm/ADT/STLExtras.h"
    15: #include "llvm/ADT/SmallVector.h"
    16: #include "llvm/ADT/StringRef.h"
    17: #include "llvm/CodeGen/MachineBasicBlock.h"
    18: #include "llvm/CodeGen/MachineFunction.h"
    19: #include "llvm/CodeGen/MachineFunctionPass.h"
    20: #include "llvm/CodeGen/MachineInstr.h"
    21: #include "llvm/CodeGen/MachineInstrBuilder.h"
    22: #include "llvm/CodeGen/MachineLoopInfo.h"
    23: #include "llvm/CodeGen/MachineMemOperand.h"
    24: #include "llvm/CodeGen/MachineOperand.h"
    25: #include "llvm/CodeGen/MachineRegisterInfo.h"
    26: #include "llvm/CodeGen/TargetRegisterInfo.h"
    27: #include "llvm/Config/llvm-config.h"
    28: #include "llvm/IR/DebugLoc.h"
    29: #include "llvm/Pass.h"
    30: #include "llvm/Support/CommandLine.h"
    31: #include "llvm/Support/Compiler.h"
    32: #include "llvm/Support/Debug.h"
    33: #include "llvm/Support/ErrorHandling.h"
    34: #include "llvm/Support/raw_ostream.h"
    35: #include <algorithm>
    36: #include <cassert>
    37: #include <cstdint>
    38: #include <limits>
    39: #include <map>
    40: #include <set>
    41: #include <utility>
    42: #include <vector>
    43: 
    44: #define DEBUG_TYPE "hsdr"
    45: 
    46: using namespace llvm;
    47: 
    48: static cl::opt<int> MaxHSDR("max-hsdr", cl::Hidden, cl::init(-1),
    49:     cl::desc("Maximum number of split partitions"));
    50: static cl::opt<bool> MemRefsFixed("hsdr-no-mem", cl::Hidden, cl::init(true),
    51:     cl::desc("Do not split loads or stores"));
    52:   static cl::opt<bool> SplitAll("hsdr-split-all", cl::Hidden, cl::init(false),
    53:       cl::desc("Split all partitions"));
    54: 
    55: namespace {
    56: 
    57:   class HexagonSplitDoubleRegs : public MachineFunctionPass {
    58:   public:
    59:     static char ID;
    60: 
    61:     HexagonSplitDoubleRegs() : MachineFunctionPass(ID) {}
    62: 
    63:     StringRef getPassName() const override {
    64:       return "Hexagon Split Double Registers";
    65:     }
    66: 
    67:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    68:       AU.addRequired<MachineLoopInfoWrapperPass>();
    69:       AU.addPreserved<MachineLoopInfoWrapperPass>();
    70:       MachineFunctionPass::getAnalysisUsage(AU);
    71:     }
    72: 
    73:     bool runOnMachineFunction(MachineFunction &MF) override;
    74: 
    75:   private:
    76:     static const TargetRegisterClass *const DoubleRC;
    77: 
    78:     const HexagonRegisterInfo *TRI = nullptr;
    79:     const HexagonInstrInfo *TII = nullptr;
    80:     const MachineLoopInfo *MLI;
    81:     MachineRegisterInfo *MRI;
    82: 
    83:     using USet = std::set<unsigned>;
    84:     using UUSetMap = std::map<unsigned, USet>;
    85:     using UUPair = std::pair<unsigned, unsigned>;
    86:     using UUPairMap = std::map<unsigned, UUPair>;
    87:     using LoopRegMap = std::map<const MachineLoop *, USet>;
    88: 
    89:     bool isInduction(unsigned Reg, LoopRegMap &IRM) const;
    90:     bool isVolatileInstr(const MachineInstr *MI) const;
    91:     bool isFixedInstr(const MachineInstr *MI) const;
    92:     void partitionRegisters(UUSetMap &P2Rs);
    93:     int32_t profit(const MachineInstr *MI) const;
    94:     int32_t profit(Register Reg) const;
    95:     bool isProfitable(const USet &Part, LoopRegMap &IRM) const;
    96: 
    97:     void collectIndRegsForLoop(const MachineLoop *L, USet &Rs);
    98:     void collectIndRegs(LoopRegMap &IRM);
    99: 
   100:     void createHalfInstr(unsigned Opc, MachineInstr *MI,
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (34 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonSplitDoubleRegs, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (34 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonSplitDoubleRegs 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:         const UUPairMap &PairMap, unsigned SubR);
   102:     void splitMemRef(MachineInstr *MI, const UUPairMap &PairMap);
   103:     void splitImmediate(MachineInstr *MI, const UUPairMap &PairMap);
   104:     void splitCombine(MachineInstr *MI, const UUPairMap &PairMap);
   105:     void splitExt(MachineInstr *MI, const UUPairMap &PairMap);
   106:     void splitShift(MachineInstr *MI, const UUPairMap &PairMap);
   107:     void splitAslOr(MachineInstr *MI, const UUPairMap &PairMap);
   108:     bool splitInstr(MachineInstr *MI, const UUPairMap &PairMap);
   109:     void replaceSubregUses(MachineInstr *MI, const UUPairMap &PairMap);
   110:     void collapseRegPairs(MachineInstr *MI, const UUPairMap &PairMap);
   111:     bool splitPartition(const USet &Part);
   112: 
   113:     static int Counter;
   114: 
   115:     static void dump_partition(raw_ostream&, const USet&,
   116:        const TargetRegisterInfo&);
   117:   };
   118: 
   119: } // end anonymous namespace
   120: 
   121: char HexagonSplitDoubleRegs::ID;
   122: int HexagonSplitDoubleRegs::Counter = 0;
   123: const TargetRegisterClass *const HexagonSplitDoubleRegs::DoubleRC =
   124:     &Hexagon::DoubleRegsRegClass;
   125: 
   126: INITIALIZE_PASS(HexagonSplitDoubleRegs, "hexagon-split-double",
   127:   "Hexagon Split Double Registers", false, false)
   128: 
   129: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
   130: LLVM_DUMP_METHOD void HexagonSplitDoubleRegs::dump_partition(raw_ostream &os,
   131:       const USet &Part, const TargetRegisterInfo &TRI) {
   132:   dbgs() << '{';
   133:   for (auto I : Part)
   134:     dbgs() << ' ' << printReg(I, &TRI);
   135:   dbgs() << " }";
   136: }
   137: #endif
   138: 
   139: bool HexagonSplitDoubleRegs::isInduction(unsigned Reg, LoopRegMap &IRM) const {
   140:   for (auto I : IRM) {
   141:     const USet &Rs = I.second;
   142:     if (Rs.find(Reg) != Rs.end())
   143:       return true;
   144:   }
   145:   return false;
   146: }
   147: 
   148: bool HexagonSplitDoubleRegs::isVolatileInstr(const MachineInstr *MI) const {
   149:   for (auto &MO : MI->memoperands())
   150:     if (MO->isVolatile() || MO->isAtomic())
   151:       return true;
   152:   return false;
   153: }
   154: 
   155: bool HexagonSplitDoubleRegs::isFixedInstr(const MachineInstr *MI) const {
   156:   if (MI->mayLoadOrStore())
   157:     if (MemRefsFixed || isVolatileInstr(MI))
   158:       return true;
   159:   if (MI->isDebugInstr())
   160:     return false;
   161: 
   162:   unsigned Opc = MI->getOpcode();
   163:   switch (Opc) {
   164:     default:
   165:       return true;
   166: 
   167:     case TargetOpcode::PHI:
   168:     case TargetOpcode::COPY:
   169:       break;
   170: 
   171:     case Hexagon::L2_loadrd_io:
   172:       // Not handling stack stores (only reg-based addresses).
   173:       if (MI->getOperand(1).isReg())
   174:         break;
   175:       return true;
   176:     case Hexagon::S2_storerd_io:
   177:       // Not handling stack stores (only reg-based addresses).
   178:       if (MI->getOperand(0).isReg())
   179:         break;
   180:       return true;
   181:     case Hexagon::L2_loadrd_pi:
   182:     case Hexagon::S2_storerd_pi:
   183: 
   184:     case Hexagon::A2_tfrpi:
   185:     case Hexagon::A2_combineii:
   186:     case Hexagon::A4_combineir:
   187:     case Hexagon::A4_combineii:
   188:     case Hexagon::A4_combineri:
   189:     case Hexagon::A2_combinew:
   190:     case Hexagon::CONST64:
   191: 
   192:     case Hexagon::A2_sxtw:
   193: 
   194:     case Hexagon::A2_andp:
   195:     case Hexagon::A2_orp:
   196:     case Hexagon::A2_xorp:
   197:     case Hexagon::S2_asl_i_p_or:
   198:     case Hexagon::S2_asl_i_p:
   199:     case Hexagon::S2_asr_i_p:
   200:     case Hexagon::S2_lsr_i_p:
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as splitMemRef, splitImmediate, splitCombine, splitExt, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 splitMemRef, splitImmediate, splitCombine, splitExt, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201:       break;
   202:   }
   203: 
   204:   for (auto &Op : MI->operands()) {
   205:     if (!Op.isReg())
   206:       continue;
   207:     Register R = Op.getReg();
   208:     if (!R.isVirtual())
   209:       return true;
   210:   }
   211:   return false;
   212: }
   213: 
   214: void HexagonSplitDoubleRegs::partitionRegisters(UUSetMap &P2Rs) {
   215:   using UUMap = std::map<unsigned, unsigned>;
   216:   using UVect = std::vector<unsigned>;
   217: 
   218:   unsigned NumRegs = MRI->getNumVirtRegs();
   219:   BitVector DoubleRegs(NumRegs);
   220:   for (unsigned i = 0; i < NumRegs; ++i) {
   221:     Register R = Register::index2VirtReg(i);
   222:     if (MRI->getRegClass(R) == DoubleRC)
   223:       DoubleRegs.set(i);
   224:   }
   225: 
   226:   BitVector FixedRegs(NumRegs);
   227:   for (int x = DoubleRegs.find_first(); x >= 0; x = DoubleRegs.find_next(x)) {
   228:     Register R = Register::index2VirtReg(x);
   229:     MachineInstr *DefI = MRI->getVRegDef(R);
   230:     // In some cases a register may exist, but never be defined or used.
   231:     // It should never appear anywhere, but mark it as "fixed", just to be
   232:     // safe.
   233:     if (!DefI || isFixedInstr(DefI))
   234:       FixedRegs.set(x);
   235:   }
   236: 
   237:   UUSetMap AssocMap;
   238:   for (int x = DoubleRegs.find_first(); x >= 0; x = DoubleRegs.find_next(x)) {
   239:     if (FixedRegs[x])
   240:       continue;
   241:     Register R = Register::index2VirtReg(x);
   242:     LLVM_DEBUG(dbgs() << printReg(R, TRI) << " ~~");
   243:     USet &Asc = AssocMap[R];
   244:     for (auto U = MRI->use_nodbg_begin(R), Z = MRI->use_nodbg_end();
   245:          U != Z; ++U) {
   246:       MachineOperand &Op = *U;
   247:       MachineInstr *UseI = Op.getParent();
   248:       if (isFixedInstr(UseI))
   249:         continue;
   250:       for (MachineOperand &MO : UseI->operands()) {
   251:         // Skip non-registers or registers with subregisters.
   252:         if (&MO == &Op || !MO.isReg() || MO.getSubReg())
   253:           continue;
   254:         Register T = MO.getReg();
   255:         if (!T.isVirtual()) {
   256:           FixedRegs.set(x);
   257:           continue;
   258:         }
   259:         if (MRI->getRegClass(T) != DoubleRC)
   260:           continue;
   261:         unsigned u = T.virtRegIndex();
   262:         if (FixedRegs[u])
   263:           continue;
   264:         LLVM_DEBUG(dbgs() << ' ' << printReg(T, TRI));
   265:         Asc.insert(T);
   266:         // Make it symmetric.
   267:         AssocMap[T].insert(R);
   268:       }
   269:     }
   270:     LLVM_DEBUG(dbgs() << '\n');
   271:   }
   272: 
   273:   UUMap R2P;
   274:   unsigned NextP = 1;
   275:   USet Visited;
   276:   for (int x = DoubleRegs.find_first(); x >= 0; x = DoubleRegs.find_next(x)) {
   277:     Register R = Register::index2VirtReg(x);
   278:     if (Visited.count(R))
   279:       continue;
   280:     // Create a new partition for R.
   281:     unsigned ThisP = FixedRegs[x] ? 0 : NextP++;
   282:     UVect WorkQ;
   283:     WorkQ.push_back(R);
   284:     for (unsigned i = 0; i < WorkQ.size(); ++i) {
   285:       unsigned T = WorkQ[i];
   286:       if (Visited.count(T))
   287:         continue;
   288:       R2P[T] = ThisP;
   289:       Visited.insert(T);
   290:       // Add all registers associated with T.
   291:       USet &Asc = AssocMap[T];
   292:       append_range(WorkQ, Asc);
   293:     }
   294:   }
   295: 
   296:   for (auto I : R2P)
   297:     P2Rs[I.second].insert(I.first);
   298: }
   299: 
   300: static inline int32_t profitImm(unsigned Imm) {
```
- EN: It declares or implements routines such as getReg, HexagonSplitDoubleRegs::partitionRegisters, getNumVirtRegs, DoubleRegs, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getReg, HexagonSplitDoubleRegs::partitionRegisters, getNumVirtRegs, DoubleRegs, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:   int32_t P = 0;
   302:   if (Imm == 0 || Imm == 0xFFFFFFFF)
   303:     P += 10;
   304:   return P;
   305: }
   306: 
   307: int32_t HexagonSplitDoubleRegs::profit(const MachineInstr *MI) const {
   308:   unsigned ImmX = 0;
   309:   unsigned Opc = MI->getOpcode();
   310:   switch (Opc) {
   311:     case TargetOpcode::PHI:
   312:       for (const auto &Op : MI->operands())
   313:         if (!Op.getSubReg())
   314:           return 0;
   315:       return 10;
   316:     case TargetOpcode::COPY:
   317:       if (MI->getOperand(1).getSubReg() != 0)
   318:         return 10;
   319:       return 0;
   320: 
   321:     case Hexagon::L2_loadrd_io:
   322:     case Hexagon::S2_storerd_io:
   323:       return -1;
   324:     case Hexagon::L2_loadrd_pi:
   325:     case Hexagon::S2_storerd_pi:
   326:       return 2;
   327: 
   328:     case Hexagon::A2_tfrpi:
   329:     case Hexagon::CONST64: {
   330:       uint64_t D = MI->getOperand(1).getImm();
   331:       unsigned Lo = D & 0xFFFFFFFFULL;
   332:       unsigned Hi = D >> 32;
   333:       return profitImm(Lo) + profitImm(Hi);
   334:     }
   335:     case Hexagon::A2_combineii:
   336:     case Hexagon::A4_combineii: {
   337:       const MachineOperand &Op1 = MI->getOperand(1);
   338:       const MachineOperand &Op2 = MI->getOperand(2);
   339:       int32_t Prof1 = Op1.isImm() ? profitImm(Op1.getImm()) : 0;
   340:       int32_t Prof2 = Op2.isImm() ? profitImm(Op2.getImm()) : 0;
   341:       return Prof1 + Prof2;
   342:     }
   343:     case Hexagon::A4_combineri:
   344:       ImmX++;
   345:       // Fall through into A4_combineir.
   346:       [[fallthrough]];
   347:     case Hexagon::A4_combineir: {
   348:       ImmX++;
   349:       const MachineOperand &OpX = MI->getOperand(ImmX);
   350:       if (OpX.isImm()) {
   351:         int64_t V = OpX.getImm();
   352:         if (V == 0 || V == -1)
   353:           return 10;
   354:       }
   355:       // Fall through into A2_combinew.
   356:       [[fallthrough]];
   357:     }
   358:     case Hexagon::A2_combinew:
   359:       return 2;
   360: 
   361:     case Hexagon::A2_sxtw:
   362:       return 3;
   363: 
   364:     case Hexagon::A2_andp:
   365:     case Hexagon::A2_orp:
   366:     case Hexagon::A2_xorp: {
   367:       Register Rs = MI->getOperand(1).getReg();
   368:       Register Rt = MI->getOperand(2).getReg();
   369:       return profit(Rs) + profit(Rt);
   370:     }
   371: 
   372:     case Hexagon::S2_asl_i_p_or: {
   373:       unsigned S = MI->getOperand(3).getImm();
   374:       if (S == 0 || S == 32)
   375:         return 10;
   376:       return -1;
   377:     }
   378:     case Hexagon::S2_asl_i_p:
   379:     case Hexagon::S2_asr_i_p:
   380:     case Hexagon::S2_lsr_i_p:
   381:       unsigned S = MI->getOperand(2).getImm();
   382:       if (S == 0 || S == 32)
   383:         return 10;
   384:       if (S == 16)
   385:         return 5;
   386:       if (S == 48)
   387:         return 7;
   388:       return -10;
   389:   }
   390: 
   391:   return 0;
   392: }
   393: 
   394: int32_t HexagonSplitDoubleRegs::profit(Register Reg) const {
   395:   assert(Reg.isVirtual());
   396: 
   397:   const MachineInstr *DefI = MRI->getVRegDef(Reg);
   398:   switch (DefI->getOpcode()) {
   399:     case Hexagon::A2_tfrpi:
   400:     case Hexagon::CONST64:
```
- EN: It declares or implements routines such as HexagonSplitDoubleRegs::profit, getOpcode, getOperand, profitImm, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonSplitDoubleRegs::profit, getOpcode, getOperand, profitImm, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:     case Hexagon::A2_combineii:
   402:     case Hexagon::A4_combineii:
   403:     case Hexagon::A4_combineri:
   404:     case Hexagon::A4_combineir:
   405:     case Hexagon::A2_combinew:
   406:       return profit(DefI);
   407:     default:
   408:       break;
   409:   }
   410:   return 0;
   411: }
   412: 
   413: bool HexagonSplitDoubleRegs::isProfitable(const USet &Part, LoopRegMap &IRM)
   414:       const {
   415:   unsigned FixedNum = 0, LoopPhiNum = 0;
   416:   int32_t TotalP = 0;
   417: 
   418:   for (unsigned DR : Part) {
   419:     MachineInstr *DefI = MRI->getVRegDef(DR);
   420:     int32_t P = profit(DefI);
   421:     if (P == std::numeric_limits<int>::min())
   422:       return false;
   423:     TotalP += P;
   424:     // Reduce the profitability of splitting induction registers.
   425:     if (isInduction(DR, IRM))
   426:       TotalP -= 30;
   427: 
   428:     for (auto U = MRI->use_nodbg_begin(DR), W = MRI->use_nodbg_end();
   429:          U != W; ++U) {
   430:       MachineInstr *UseI = U->getParent();
   431:       if (isFixedInstr(UseI)) {
   432:         FixedNum++;
   433:         // Calculate the cost of generating REG_SEQUENCE instructions.
   434:         for (auto &Op : UseI->operands()) {
   435:           if (Op.isReg() && Part.count(Op.getReg()))
   436:             if (Op.getSubReg())
   437:               TotalP -= 2;
   438:         }
   439:         continue;
   440:       }
   441:       // If a register from this partition is used in a fixed instruction,
   442:       // and there is also a register in this partition that is used in
   443:       // a loop phi node, then decrease the splitting profit as this can
   444:       // confuse the modulo scheduler.
   445:       if (UseI->isPHI()) {
   446:         const MachineBasicBlock *PB = UseI->getParent();
   447:         const MachineLoop *L = MLI->getLoopFor(PB);
   448:         if (L && L->getHeader() == PB)
   449:           LoopPhiNum++;
   450:       }
   451:       // Splittable instruction.
   452:       int32_t P = profit(UseI);
   453:       if (P == std::numeric_limits<int>::min())
   454:         return false;
   455:       TotalP += P;
   456:     }
   457:   }
   458: 
   459:   if (FixedNum > 0 && LoopPhiNum > 0)
   460:     TotalP -= 20*LoopPhiNum;
   461: 
   462:   LLVM_DEBUG(dbgs() << "Partition profit: " << TotalP << '\n');
   463:   if (SplitAll)
   464:     return true;
   465:   return TotalP > 0;
   466: }
   467: 
   468: void HexagonSplitDoubleRegs::collectIndRegsForLoop(const MachineLoop *L,
   469:       USet &Rs) {
   470:   const MachineBasicBlock *HB = L->getHeader();
   471:   const MachineBasicBlock *LB = L->getLoopLatch();
   472:   if (!HB || !LB)
   473:     return;
   474: 
   475:   // Examine the latch branch. Expect it to be a conditional branch to
   476:   // the header (either "br-cond header" or "br-cond exit; br header").
   477:   MachineBasicBlock *TB = nullptr, *FB = nullptr;
   478:   MachineBasicBlock *TmpLB = const_cast<MachineBasicBlock*>(LB);
   479:   SmallVector<MachineOperand,2> Cond;
   480:   bool BadLB = TII->analyzeBranch(*TmpLB, TB, FB, Cond, false);
   481:   // Only analyzable conditional branches. HII::analyzeBranch will put
   482:   // the branch opcode as the first element of Cond, and the predicate
   483:   // operand as the second.
   484:   if (BadLB || Cond.size() != 2)
   485:     return;
   486:   // Only simple jump-conditional (with or without negation).
   487:   if (!TII->PredOpcodeHasJMP_c(Cond[0].getImm()))
   488:     return;
   489:   // Must go to the header.
   490:   if (TB != HB && FB != HB)
   491:     return;
   492:   assert(Cond[1].isReg() && "Unexpected Cond vector from analyzeBranch");
   493:   // Expect a predicate register.
   494:   Register PR = Cond[1].getReg();
   495:   assert(MRI->getRegClass(PR) == &Hexagon::PredRegsRegClass);
   496: 
   497:   // Get the registers on which the loop controlling compare instruction
   498:   // depends.
   499:   Register CmpR1, CmpR2;
   500:   const MachineInstr *CmpI = MRI->getVRegDef(PR);
```
- EN: It declares or implements routines such as profit, HexagonSplitDoubleRegs::isProfitable, getVRegDef, getParent, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 profit, HexagonSplitDoubleRegs::isProfitable, getVRegDef, getParent, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:   while (CmpI->getOpcode() == Hexagon::C2_not)
   502:     CmpI = MRI->getVRegDef(CmpI->getOperand(1).getReg());
   503: 
   504:   int64_t Mask = 0, Val = 0;
   505:   bool OkCI = TII->analyzeCompare(*CmpI, CmpR1, CmpR2, Mask, Val);
   506:   if (!OkCI)
   507:     return;
   508:   // Eliminate non-double input registers.
   509:   if (CmpR1 && MRI->getRegClass(CmpR1) != DoubleRC)
   510:     CmpR1 = 0;
   511:   if (CmpR2 && MRI->getRegClass(CmpR2) != DoubleRC)
   512:     CmpR2 = 0;
   513:   if (!CmpR1 && !CmpR2)
   514:     return;
   515: 
   516:   // Now examine the top of the loop: the phi nodes that could poten-
   517:   // tially define loop induction registers. The registers defined by
   518:   // such a phi node would be used in a 64-bit add, which then would
   519:   // be used in the loop compare instruction.
   520: 
   521:   // Get the set of all double registers defined by phi nodes in the
   522:   // loop header.
   523:   using UVect = std::vector<unsigned>;
   524: 
   525:   UVect DP;
   526:   for (auto &MI : *HB) {
   527:     if (!MI.isPHI())
   528:       break;
   529:     const MachineOperand &MD = MI.getOperand(0);
   530:     Register R = MD.getReg();
   531:     if (MRI->getRegClass(R) == DoubleRC)
   532:       DP.push_back(R);
   533:   }
   534:   if (DP.empty())
   535:     return;
   536: 
   537:   auto NoIndOp = [this, CmpR1, CmpR2] (unsigned R) -> bool {
   538:     for (auto I = MRI->use_nodbg_begin(R), E = MRI->use_nodbg_end();
   539:          I != E; ++I) {
   540:       const MachineInstr *UseI = I->getParent();
   541:       if (UseI->getOpcode() != Hexagon::A2_addp)
   542:         continue;
   543:       // Get the output from the add. If it is one of the inputs to the
   544:       // loop-controlling compare instruction, then R is likely an induc-
   545:       // tion register.
   546:       Register T = UseI->getOperand(0).getReg();
   547:       if (T == CmpR1 || T == CmpR2)
   548:         return false;
   549:     }
   550:     return true;
   551:   };
   552:   UVect::iterator End = llvm::remove_if(DP, NoIndOp);
   553:   Rs.insert(DP.begin(), End);
   554:   Rs.insert(CmpR1);
   555:   Rs.insert(CmpR2);
   556: 
   557:   LLVM_DEBUG({
   558:     dbgs() << "For loop at " << printMBBReference(*HB) << " ind regs: ";
   559:     dump_partition(dbgs(), Rs, *TRI);
   560:     dbgs() << '\n';
   561:   });
   562: }
   563: 
   564: void HexagonSplitDoubleRegs::collectIndRegs(LoopRegMap &IRM) {
   565:   using LoopVector = std::vector<MachineLoop *>;
   566: 
   567:   LoopVector WorkQ;
   568: 
   569:   append_range(WorkQ, *MLI);
   570:   for (unsigned i = 0; i < WorkQ.size(); ++i)
   571:     append_range(WorkQ, *WorkQ[i]);
   572: 
   573:   USet Rs;
   574:   for (MachineLoop *L : WorkQ) {
   575:     Rs.clear();
   576:     collectIndRegsForLoop(L, Rs);
   577:     if (!Rs.empty())
   578:       IRM.insert(std::make_pair(L, Rs));
   579:   }
   580: }
   581: 
   582: void HexagonSplitDoubleRegs::createHalfInstr(unsigned Opc, MachineInstr *MI,
   583:       const UUPairMap &PairMap, unsigned SubR) {
   584:   MachineBasicBlock &B = *MI->getParent();
   585:   DebugLoc DL = MI->getDebugLoc();
   586:   MachineInstr *NewI = BuildMI(B, MI, DL, TII->get(Opc));
   587: 
   588:   for (auto &Op : MI->operands()) {
   589:     if (!Op.isReg()) {
   590:       NewI->addOperand(Op);
   591:       continue;
   592:     }
   593:     // For register operands, set the subregister.
   594:     Register R = Op.getReg();
   595:     unsigned SR = Op.getSubReg();
   596:     bool isVirtReg = R.isVirtual();
   597:     bool isKill = Op.isKill();
   598:     if (isVirtReg && MRI->getRegClass(R) == DoubleRC) {
   599:       isKill = false;
   600:       UUPairMap::const_iterator F = PairMap.find(R);
```
- EN: It declares or implements routines such as analyzeCompare, getOperand, getReg, getParent, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 analyzeCompare, getOperand, getReg, getParent, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:       if (F == PairMap.end()) {
   602:         SR = SubR;
   603:       } else {
   604:         const UUPair &P = F->second;
   605:         R = (SubR == Hexagon::isub_lo) ? P.first : P.second;
   606:         SR = 0;
   607:       }
   608:     }
   609:     auto CO = MachineOperand::CreateReg(R, Op.isDef(), Op.isImplicit(), isKill,
   610:           Op.isDead(), Op.isUndef(), Op.isEarlyClobber(), SR, Op.isDebug(),
   611:           Op.isInternalRead());
   612:     NewI->addOperand(CO);
   613:   }
   614: }
   615: 
   616: void HexagonSplitDoubleRegs::splitMemRef(MachineInstr *MI,
   617:       const UUPairMap &PairMap) {
   618:   bool Load = MI->mayLoad();
   619:   unsigned OrigOpc = MI->getOpcode();
   620:   bool PostInc = (OrigOpc == Hexagon::L2_loadrd_pi ||
   621:                   OrigOpc == Hexagon::S2_storerd_pi);
   622:   MachineInstr *LowI, *HighI;
   623:   MachineBasicBlock &B = *MI->getParent();
   624:   DebugLoc DL = MI->getDebugLoc();
   625: 
   626:   // Index of the base-address-register operand.
   627:   unsigned AdrX = PostInc ? (Load ? 2 : 1)
   628:                           : (Load ? 1 : 0);
   629:   MachineOperand &AdrOp = MI->getOperand(AdrX);
   630:   RegState RSA = getRegState(AdrOp);
   631:   MachineOperand &ValOp = Load ? MI->getOperand(0)
   632:                                : (PostInc ? MI->getOperand(3)
   633:                                           : MI->getOperand(2));
   634:   UUPairMap::const_iterator F = PairMap.find(ValOp.getReg());
   635:   assert(F != PairMap.end());
   636: 
   637:   if (Load) {
   638:     const UUPair &P = F->second;
   639:     int64_t Off = PostInc ? 0 : MI->getOperand(2).getImm();
   640:     LowI = BuildMI(B, MI, DL, TII->get(Hexagon::L2_loadri_io), P.first)
   641:              .addReg(AdrOp.getReg(), RSA & ~RegState::Kill, AdrOp.getSubReg())
   642:              .addImm(Off);
   643:     HighI = BuildMI(B, MI, DL, TII->get(Hexagon::L2_loadri_io), P.second)
   644:               .addReg(AdrOp.getReg(), RSA & ~RegState::Kill, AdrOp.getSubReg())
   645:               .addImm(Off+4);
   646:   } else {
   647:     const UUPair &P = F->second;
   648:     int64_t Off = PostInc ? 0 : MI->getOperand(1).getImm();
   649:     LowI = BuildMI(B, MI, DL, TII->get(Hexagon::S2_storeri_io))
   650:              .addReg(AdrOp.getReg(), RSA & ~RegState::Kill, AdrOp.getSubReg())
   651:              .addImm(Off)
   652:              .addReg(P.first);
   653:     HighI = BuildMI(B, MI, DL, TII->get(Hexagon::S2_storeri_io))
   654:               .addReg(AdrOp.getReg(), RSA & ~RegState::Kill, AdrOp.getSubReg())
   655:               .addImm(Off+4)
   656:               .addReg(P.second);
   657:   }
   658: 
   659:   if (PostInc) {
   660:     // Create the increment of the address register.
   661:     int64_t Inc = Load ? MI->getOperand(3).getImm()
   662:                        : MI->getOperand(2).getImm();
   663:     MachineOperand &UpdOp = Load ? MI->getOperand(1) : MI->getOperand(0);
   664:     const TargetRegisterClass *RC = MRI->getRegClass(UpdOp.getReg());
   665:     Register NewR = MRI->createVirtualRegister(RC);
   666:     assert(!UpdOp.getSubReg() && "Def operand with subreg");
   667:     BuildMI(B, MI, DL, TII->get(Hexagon::A2_addi), NewR)
   668:       .addReg(AdrOp.getReg(), RSA)
   669:       .addImm(Inc);
   670:     MRI->replaceRegWith(UpdOp.getReg(), NewR);
   671:     // The original instruction will be deleted later.
   672:   }
   673: 
   674:   // Generate a new pair of memory-operands.
   675:   MachineFunction &MF = *B.getParent();
   676:   for (auto &MO : MI->memoperands()) {
   677:     const MachinePointerInfo &Ptr = MO->getPointerInfo();
   678:     MachineMemOperand::Flags F = MO->getFlags();
   679:     Align A = MO->getAlign();
   680: 
   681:     auto *Tmp1 = MF.getMachineMemOperand(Ptr, F, 4 /*size*/, A);
   682:     LowI->addMemOperand(MF, Tmp1);
   683:     auto *Tmp2 =
   684:         MF.getMachineMemOperand(Ptr, F, 4 /*size*/, std::min(A, Align(4)));
   685:     HighI->addMemOperand(MF, Tmp2);
   686:   }
   687: }
   688: 
   689: void HexagonSplitDoubleRegs::splitImmediate(MachineInstr *MI,
   690:       const UUPairMap &PairMap) {
   691:   MachineOperand &Op0 = MI->getOperand(0);
   692:   MachineOperand &Op1 = MI->getOperand(1);
   693:   assert(Op0.isReg() && Op1.isImm());
   694:   uint64_t V = Op1.getImm();
   695: 
   696:   MachineBasicBlock &B = *MI->getParent();
   697:   DebugLoc DL = MI->getDebugLoc();
   698:   UUPairMap::const_iterator F = PairMap.find(Op0.getReg());
   699:   assert(F != PairMap.end());
   700:   const UUPair &P = F->second;
```
- EN: It declares or implements routines such as MachineOperand::CreateReg, addOperand, HexagonSplitDoubleRegs::splitMemRef, mayLoad, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 MachineOperand::CreateReg, addOperand, HexagonSplitDoubleRegs::splitMemRef, mayLoad, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701: 
   702:   // The operand to A2_tfrsi can only have 32 significant bits. Immediate
   703:   // values in MachineOperand are stored as 64-bit integers, and so the
   704:   // value -1 may be represented either as 64-bit -1, or 4294967295. Both
   705:   // will have the 32 higher bits truncated in the end, but -1 will remain
   706:   // as -1, while the latter may appear to be a large unsigned value
   707:   // requiring a constant extender. The casting to int32_t will select the
   708:   // former representation. (The same reasoning applies to all 32-bit
   709:   // values.)
   710:   BuildMI(B, MI, DL, TII->get(Hexagon::A2_tfrsi), P.first)
   711:     .addImm(int32_t(V & 0xFFFFFFFFULL));
   712:   BuildMI(B, MI, DL, TII->get(Hexagon::A2_tfrsi), P.second)
   713:     .addImm(int32_t(V >> 32));
   714: }
   715: 
   716: void HexagonSplitDoubleRegs::splitCombine(MachineInstr *MI,
   717:       const UUPairMap &PairMap) {
   718:   MachineOperand &Op0 = MI->getOperand(0);
   719:   MachineOperand &Op1 = MI->getOperand(1);
   720:   MachineOperand &Op2 = MI->getOperand(2);
   721:   assert(Op0.isReg());
   722: 
   723:   MachineBasicBlock &B = *MI->getParent();
   724:   DebugLoc DL = MI->getDebugLoc();
   725:   UUPairMap::const_iterator F = PairMap.find(Op0.getReg());
   726:   assert(F != PairMap.end());
   727:   const UUPair &P = F->second;
   728: 
   729:   if (!Op1.isReg()) {
   730:     BuildMI(B, MI, DL, TII->get(Hexagon::A2_tfrsi), P.second)
   731:       .add(Op1);
   732:   } else {
   733:     BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), P.second)
   734:       .addReg(Op1.getReg(), getRegState(Op1), Op1.getSubReg());
   735:   }
   736: 
   737:   if (!Op2.isReg()) {
   738:     BuildMI(B, MI, DL, TII->get(Hexagon::A2_tfrsi), P.first)
   739:       .add(Op2);
   740:   } else {
   741:     BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), P.first)
   742:       .addReg(Op2.getReg(), getRegState(Op2), Op2.getSubReg());
   743:   }
   744: }
   745: 
   746: void HexagonSplitDoubleRegs::splitExt(MachineInstr *MI,
   747:       const UUPairMap &PairMap) {
   748:   MachineOperand &Op0 = MI->getOperand(0);
   749:   MachineOperand &Op1 = MI->getOperand(1);
   750:   assert(Op0.isReg() && Op1.isReg());
   751: 
   752:   MachineBasicBlock &B = *MI->getParent();
   753:   DebugLoc DL = MI->getDebugLoc();
   754:   UUPairMap::const_iterator F = PairMap.find(Op0.getReg());
   755:   assert(F != PairMap.end());
   756:   const UUPair &P = F->second;
   757:   RegState RS = getRegState(Op1);
   758: 
   759:   BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), P.first)
   760:     .addReg(Op1.getReg(), RS & ~RegState::Kill, Op1.getSubReg());
   761:   BuildMI(B, MI, DL, TII->get(Hexagon::S2_asr_i_r), P.second)
   762:     .addReg(Op1.getReg(), RS, Op1.getSubReg())
   763:     .addImm(31);
   764: }
   765: 
   766: void HexagonSplitDoubleRegs::splitShift(MachineInstr *MI,
   767:       const UUPairMap &PairMap) {
   768:   using namespace Hexagon;
   769: 
   770:   MachineOperand &Op0 = MI->getOperand(0);
   771:   MachineOperand &Op1 = MI->getOperand(1);
   772:   MachineOperand &Op2 = MI->getOperand(2);
   773:   assert(Op0.isReg() && Op1.isReg() && Op2.isImm());
   774:   int64_t Sh64 = Op2.getImm();
   775:   assert(Sh64 >= 0 && Sh64 < 64);
   776:   unsigned S = Sh64;
   777: 
   778:   UUPairMap::const_iterator F = PairMap.find(Op0.getReg());
   779:   assert(F != PairMap.end());
   780:   const UUPair &P = F->second;
   781:   Register LoR = P.first;
   782:   Register HiR = P.second;
   783: 
   784:   unsigned Opc = MI->getOpcode();
   785:   bool Right = (Opc == S2_lsr_i_p || Opc == S2_asr_i_p);
   786:   bool Left = !Right;
   787:   bool Signed = (Opc == S2_asr_i_p);
   788: 
   789:   MachineBasicBlock &B = *MI->getParent();
   790:   DebugLoc DL = MI->getDebugLoc();
   791:   RegState RS = getRegState(Op1);
   792:   unsigned ShiftOpc = Left ? S2_asl_i_r
   793:                            : (Signed ? S2_asr_i_r : S2_lsr_i_r);
   794:   unsigned LoSR = isub_lo;
   795:   unsigned HiSR = isub_hi;
   796: 
   797:   if (S == 0) {
   798:     // No shift, subregister copy.
   799:     BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), LoR)
   800:       .addReg(Op1.getReg(), RS & ~RegState::Kill, LoSR);
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as BuildMI, HexagonSplitDoubleRegs::splitCombine, getOperand, assert, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 BuildMI, HexagonSplitDoubleRegs::splitCombine, getOperand, assert, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-900 / 第 801-900 行

```cpp
   801:     BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), HiR)
   802:       .addReg(Op1.getReg(), RS, HiSR);
   803:   } else if (S < 32) {
   804:     const TargetRegisterClass *IntRC = &IntRegsRegClass;
   805:     Register TmpR = MRI->createVirtualRegister(IntRC);
   806:     // Expansion:
   807:     // Shift left:    DR = shl R, #s
   808:     //   LoR  = shl R.lo, #s
   809:     //   TmpR = extractu R.lo, #s, #32-s
   810:     //   HiR  = or (TmpR, asl(R.hi, #s))
   811:     // Shift right:   DR = shr R, #s
   812:     //   HiR  = shr R.hi, #s
   813:     //   TmpR = shr R.lo, #s
   814:     //   LoR  = insert TmpR, R.hi, #s, #32-s
   815: 
   816:     // Shift left:
   817:     //   LoR  = shl R.lo, #s
   818:     // Shift right:
   819:     //   TmpR = shr R.lo, #s
   820: 
   821:     // Make a special case for A2_aslh and A2_asrh (they are predicable as
   822:     // opposed to S2_asl_i_r/S2_asr_i_r).
   823:     if (S == 16 && Left)
   824:       BuildMI(B, MI, DL, TII->get(A2_aslh), LoR)
   825:         .addReg(Op1.getReg(), RS & ~RegState::Kill, LoSR);
   826:     else if (S == 16 && Signed)
   827:       BuildMI(B, MI, DL, TII->get(A2_asrh), TmpR)
   828:         .addReg(Op1.getReg(), RS & ~RegState::Kill, LoSR);
   829:     else
   830:       BuildMI(B, MI, DL, TII->get(ShiftOpc), (Left ? LoR : TmpR))
   831:         .addReg(Op1.getReg(), RS & ~RegState::Kill, LoSR)
   832:         .addImm(S);
   833: 
   834:     if (Left) {
   835:       // TmpR = extractu R.lo, #s, #32-s
   836:       BuildMI(B, MI, DL, TII->get(S2_extractu), TmpR)
   837:         .addReg(Op1.getReg(), RS & ~RegState::Kill, LoSR)
   838:         .addImm(S)
   839:         .addImm(32-S);
   840:       // HiR  = or (TmpR, asl(R.hi, #s))
   841:       BuildMI(B, MI, DL, TII->get(S2_asl_i_r_or), HiR)
   842:         .addReg(TmpR)
   843:         .addReg(Op1.getReg(), RS, HiSR)
   844:         .addImm(S);
   845:     } else {
   846:       // HiR  = shr R.hi, #s
   847:       BuildMI(B, MI, DL, TII->get(ShiftOpc), HiR)
   848:         .addReg(Op1.getReg(), RS & ~RegState::Kill, HiSR)
   849:         .addImm(S);
   850:       // LoR  = insert TmpR, R.hi, #s, #32-s
   851:       BuildMI(B, MI, DL, TII->get(S2_insert), LoR)
   852:         .addReg(TmpR)
   853:         .addReg(Op1.getReg(), RS, HiSR)
   854:         .addImm(S)
   855:         .addImm(32-S);
   856:     }
   857:   } else if (S == 32) {
   858:     BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), (Left ? HiR : LoR))
   859:       .addReg(Op1.getReg(), RS & ~RegState::Kill, (Left ? LoSR : HiSR));
   860:     if (!Signed)
   861:       BuildMI(B, MI, DL, TII->get(A2_tfrsi), (Left ? LoR : HiR))
   862:         .addImm(0);
   863:     else  // Must be right shift.
   864:       BuildMI(B, MI, DL, TII->get(S2_asr_i_r), HiR)
   865:         .addReg(Op1.getReg(), RS, HiSR)
   866:         .addImm(31);
   867:   } else if (S < 64) {
   868:     S -= 32;
   869:     if (S == 16 && Left)
   870:       BuildMI(B, MI, DL, TII->get(A2_aslh), HiR)
   871:         .addReg(Op1.getReg(), RS & ~RegState::Kill, LoSR);
   872:     else if (S == 16 && Signed)
   873:       BuildMI(B, MI, DL, TII->get(A2_asrh), LoR)
   874:         .addReg(Op1.getReg(), RS & ~RegState::Kill, HiSR);
   875:     else
   876:       BuildMI(B, MI, DL, TII->get(ShiftOpc), (Left ? HiR : LoR))
   877:         .addReg(Op1.getReg(), RS & ~RegState::Kill, (Left ? LoSR : HiSR))
   878:         .addImm(S);
   879: 
   880:     if (Signed)
   881:       BuildMI(B, MI, DL, TII->get(S2_asr_i_r), HiR)
   882:         .addReg(Op1.getReg(), RS, HiSR)
   883:         .addImm(31);
   884:     else
   885:       BuildMI(B, MI, DL, TII->get(A2_tfrsi), (Left ? LoR : HiR))
   886:         .addImm(0);
   887:   }
   888: }
   889: 
   890: void HexagonSplitDoubleRegs::splitAslOr(MachineInstr *MI,
   891:       const UUPairMap &PairMap) {
   892:   using namespace Hexagon;
   893: 
   894:   MachineOperand &Op0 = MI->getOperand(0);
   895:   MachineOperand &Op1 = MI->getOperand(1);
   896:   MachineOperand &Op2 = MI->getOperand(2);
   897:   MachineOperand &Op3 = MI->getOperand(3);
   898:   assert(Op0.isReg() && Op1.isReg() && Op2.isReg() && Op3.isImm());
   899:   int64_t Sh64 = Op3.getImm();
   900:   assert(Sh64 >= 0 && Sh64 < 64);
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as BuildMI, createVirtualRegister, or, HexagonSplitDoubleRegs::splitAslOr, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 BuildMI, createVirtualRegister, or, HexagonSplitDoubleRegs::splitAslOr, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:   unsigned S = Sh64;
   902: 
   903:   UUPairMap::const_iterator F = PairMap.find(Op0.getReg());
   904:   assert(F != PairMap.end());
   905:   const UUPair &P = F->second;
   906:   unsigned LoR = P.first;
   907:   unsigned HiR = P.second;
   908: 
   909:   MachineBasicBlock &B = *MI->getParent();
   910:   DebugLoc DL = MI->getDebugLoc();
   911:   RegState RS1 = getRegState(Op1);
   912:   RegState RS2 = getRegState(Op2);
   913:   const TargetRegisterClass *IntRC = &IntRegsRegClass;
   914: 
   915:   unsigned LoSR = isub_lo;
   916:   unsigned HiSR = isub_hi;
   917: 
   918:   // Op0 = S2_asl_i_p_or Op1, Op2, Op3
   919:   // means:  Op0 = or (Op1, asl(Op2, Op3))
   920: 
   921:   // Expansion of
   922:   //   DR = or (R1, asl(R2, #s))
   923:   //
   924:   //   LoR  = or (R1.lo, asl(R2.lo, #s))
   925:   //   Tmp1 = extractu R2.lo, #s, #32-s
   926:   //   Tmp2 = or R1.hi, Tmp1
   927:   //   HiR  = or (Tmp2, asl(R2.hi, #s))
   928: 
   929:   if (S == 0) {
   930:     // DR  = or (R1, asl(R2, #0))
   931:     //    -> or (R1, R2)
   932:     // i.e. LoR = or R1.lo, R2.lo
   933:     //      HiR = or R1.hi, R2.hi
   934:     BuildMI(B, MI, DL, TII->get(A2_or), LoR)
   935:       .addReg(Op1.getReg(), RS1 & ~RegState::Kill, LoSR)
   936:       .addReg(Op2.getReg(), RS2 & ~RegState::Kill, LoSR);
   937:     BuildMI(B, MI, DL, TII->get(A2_or), HiR)
   938:       .addReg(Op1.getReg(), RS1, HiSR)
   939:       .addReg(Op2.getReg(), RS2, HiSR);
   940:   } else if (S < 32) {
   941:     BuildMI(B, MI, DL, TII->get(S2_asl_i_r_or), LoR)
   942:       .addReg(Op1.getReg(), RS1 & ~RegState::Kill, LoSR)
   943:       .addReg(Op2.getReg(), RS2 & ~RegState::Kill, LoSR)
   944:       .addImm(S);
   945:     Register TmpR1 = MRI->createVirtualRegister(IntRC);
   946:     BuildMI(B, MI, DL, TII->get(S2_extractu), TmpR1)
   947:       .addReg(Op2.getReg(), RS2 & ~RegState::Kill, LoSR)
   948:       .addImm(S)
   949:       .addImm(32-S);
   950:     Register TmpR2 = MRI->createVirtualRegister(IntRC);
   951:     BuildMI(B, MI, DL, TII->get(A2_or), TmpR2)
   952:       .addReg(Op1.getReg(), RS1, HiSR)
   953:       .addReg(TmpR1);
   954:     BuildMI(B, MI, DL, TII->get(S2_asl_i_r_or), HiR)
   955:       .addReg(TmpR2)
   956:       .addReg(Op2.getReg(), RS2, HiSR)
   957:       .addImm(S);
   958:   } else if (S == 32) {
   959:     // DR  = or (R1, asl(R2, #32))
   960:     //    -> or R1, R2.lo
   961:     // LoR = R1.lo
   962:     // HiR = or R1.hi, R2.lo
   963:     BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), LoR)
   964:       .addReg(Op1.getReg(), RS1 & ~RegState::Kill, LoSR);
   965:     BuildMI(B, MI, DL, TII->get(A2_or), HiR)
   966:       .addReg(Op1.getReg(), RS1, HiSR)
   967:       .addReg(Op2.getReg(), RS2, LoSR);
   968:   } else if (S < 64) {
   969:     // DR  = or (R1, asl(R2, #s))
   970:     //
   971:     // LoR = R1:lo
   972:     // HiR = or (R1:hi, asl(R2:lo, #s-32))
   973:     S -= 32;
   974:     BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), LoR)
   975:       .addReg(Op1.getReg(), RS1 & ~RegState::Kill, LoSR);
   976:     BuildMI(B, MI, DL, TII->get(S2_asl_i_r_or), HiR)
   977:       .addReg(Op1.getReg(), RS1, HiSR)
   978:       .addReg(Op2.getReg(), RS2, LoSR)
   979:       .addImm(S);
   980:   }
   981: }
   982: 
   983: bool HexagonSplitDoubleRegs::splitInstr(MachineInstr *MI,
   984:       const UUPairMap &PairMap) {
   985:   using namespace Hexagon;
   986: 
   987:   LLVM_DEBUG(dbgs() << "Splitting: " << *MI);
   988:   bool Split = false;
   989:   unsigned Opc = MI->getOpcode();
   990: 
   991:   switch (Opc) {
   992:     case TargetOpcode::PHI:
   993:     case TargetOpcode::COPY: {
   994:       Register DstR = MI->getOperand(0).getReg();
   995:       if (MRI->getRegClass(DstR) == DoubleRC) {
   996:         createHalfInstr(Opc, MI, PairMap, isub_lo);
   997:         createHalfInstr(Opc, MI, PairMap, isub_hi);
   998:         Split = true;
   999:       }
  1000:       break;
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as find, assert, getParent, getDebugLoc, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 find, assert, getParent, getDebugLoc, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:     }
  1002:     case A2_andp:
  1003:       createHalfInstr(A2_and, MI, PairMap, isub_lo);
  1004:       createHalfInstr(A2_and, MI, PairMap, isub_hi);
  1005:       Split = true;
  1006:       break;
  1007:     case A2_orp:
  1008:       createHalfInstr(A2_or, MI, PairMap, isub_lo);
  1009:       createHalfInstr(A2_or, MI, PairMap, isub_hi);
  1010:       Split = true;
  1011:       break;
  1012:     case A2_xorp:
  1013:       createHalfInstr(A2_xor, MI, PairMap, isub_lo);
  1014:       createHalfInstr(A2_xor, MI, PairMap, isub_hi);
  1015:       Split = true;
  1016:       break;
  1017: 
  1018:     case L2_loadrd_io:
  1019:     case L2_loadrd_pi:
  1020:     case S2_storerd_io:
  1021:     case S2_storerd_pi:
  1022:       splitMemRef(MI, PairMap);
  1023:       Split = true;
  1024:       break;
  1025: 
  1026:     case A2_tfrpi:
  1027:     case CONST64:
  1028:       splitImmediate(MI, PairMap);
  1029:       Split = true;
  1030:       break;
  1031: 
  1032:     case A2_combineii:
  1033:     case A4_combineir:
  1034:     case A4_combineii:
  1035:     case A4_combineri:
  1036:     case A2_combinew:
  1037:       splitCombine(MI, PairMap);
  1038:       Split = true;
  1039:       break;
  1040: 
  1041:     case A2_sxtw:
  1042:       splitExt(MI, PairMap);
  1043:       Split = true;
  1044:       break;
  1045: 
  1046:     case S2_asl_i_p:
  1047:     case S2_asr_i_p:
  1048:     case S2_lsr_i_p:
  1049:       splitShift(MI, PairMap);
  1050:       Split = true;
  1051:       break;
  1052: 
  1053:     case S2_asl_i_p_or:
  1054:       splitAslOr(MI, PairMap);
  1055:       Split = true;
  1056:       break;
  1057: 
  1058:     default:
  1059:       llvm_unreachable("Instruction not splitable");
  1060:       return false;
  1061:   }
  1062: 
  1063:   return Split;
  1064: }
  1065: 
  1066: void HexagonSplitDoubleRegs::replaceSubregUses(MachineInstr *MI,
  1067:       const UUPairMap &PairMap) {
  1068:   for (auto &Op : MI->operands()) {
  1069:     if (!Op.isReg() || !Op.isUse() || !Op.getSubReg())
  1070:       continue;
  1071:     Register R = Op.getReg();
  1072:     UUPairMap::const_iterator F = PairMap.find(R);
  1073:     if (F == PairMap.end())
  1074:       continue;
  1075:     const UUPair &P = F->second;
  1076:     switch (Op.getSubReg()) {
  1077:       case Hexagon::isub_lo:
  1078:         Op.setReg(P.first);
  1079:         break;
  1080:       case Hexagon::isub_hi:
  1081:         Op.setReg(P.second);
  1082:         break;
  1083:     }
  1084:     Op.setSubReg(0);
  1085:   }
  1086: }
  1087: 
  1088: void HexagonSplitDoubleRegs::collapseRegPairs(MachineInstr *MI,
  1089:       const UUPairMap &PairMap) {
  1090:   MachineBasicBlock &B = *MI->getParent();
  1091:   DebugLoc DL = MI->getDebugLoc();
  1092: 
  1093:   for (auto &Op : MI->operands()) {
  1094:     if (!Op.isReg() || !Op.isUse())
  1095:       continue;
  1096:     Register R = Op.getReg();
  1097:     if (!R.isVirtual())
  1098:       continue;
  1099:     if (MRI->getRegClass(R) != DoubleRC || Op.getSubReg())
  1100:       continue;
```
- EN: It declares or implements routines such as createHalfInstr, splitMemRef, splitImmediate, splitCombine, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 createHalfInstr, splitMemRef, splitImmediate, splitCombine, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

### Lines 1101-1200 / 第 1101-1200 行

```cpp
  1101:     UUPairMap::const_iterator F = PairMap.find(R);
  1102:     if (F == PairMap.end())
  1103:       continue;
  1104:     const UUPair &Pr = F->second;
  1105:     Register NewDR = MRI->createVirtualRegister(DoubleRC);
  1106:     BuildMI(B, MI, DL, TII->get(TargetOpcode::REG_SEQUENCE), NewDR)
  1107:       .addReg(Pr.first)
  1108:       .addImm(Hexagon::isub_lo)
  1109:       .addReg(Pr.second)
  1110:       .addImm(Hexagon::isub_hi);
  1111:     Op.setReg(NewDR);
  1112:   }
  1113: }
  1114: 
  1115: bool HexagonSplitDoubleRegs::splitPartition(const USet &Part) {
  1116:   using MISet = std::set<MachineInstr *>;
  1117: 
  1118:   const TargetRegisterClass *IntRC = &Hexagon::IntRegsRegClass;
  1119:   bool Changed = false;
  1120: 
  1121:   LLVM_DEBUG(dbgs() << "Splitting partition: ";
  1122:              dump_partition(dbgs(), Part, *TRI); dbgs() << '\n');
  1123: 
  1124:   UUPairMap PairMap;
  1125: 
  1126:   MISet SplitIns;
  1127:   for (unsigned DR : Part) {
  1128:     MachineInstr *DefI = MRI->getVRegDef(DR);
  1129:     SplitIns.insert(DefI);
  1130: 
  1131:     // Collect all instructions, including fixed ones.  We won't split them,
  1132:     // but we need to visit them again to insert the REG_SEQUENCE instructions.
  1133:     for (auto U = MRI->use_nodbg_begin(DR), W = MRI->use_nodbg_end();
  1134:          U != W; ++U)
  1135:       SplitIns.insert(U->getParent());
  1136: 
  1137:     Register LoR = MRI->createVirtualRegister(IntRC);
  1138:     Register HiR = MRI->createVirtualRegister(IntRC);
  1139:     LLVM_DEBUG(dbgs() << "Created mapping: " << printReg(DR, TRI) << " -> "
  1140:                       << printReg(HiR, TRI) << ':' << printReg(LoR, TRI)
  1141:                       << '\n');
  1142:     PairMap.insert(std::make_pair(DR, UUPair(LoR, HiR)));
  1143:   }
  1144: 
  1145:   MISet Erase;
  1146:   for (auto *MI : SplitIns) {
  1147:     if (isFixedInstr(MI)) {
  1148:       collapseRegPairs(MI, PairMap);
  1149:     } else {
  1150:       bool Done = splitInstr(MI, PairMap);
  1151:       if (Done)
  1152:         Erase.insert(MI);
  1153:       Changed |= Done;
  1154:     }
  1155:   }
  1156: 
  1157:   for (unsigned DR : Part) {
  1158:     // Before erasing "double" instructions, revisit all uses of the double
  1159:     // registers in this partition, and replace all uses of them with subre-
  1160:     // gisters, with the corresponding single registers.
  1161:     MISet Uses;
  1162:     for (auto U = MRI->use_nodbg_begin(DR), W = MRI->use_nodbg_end();
  1163:          U != W; ++U)
  1164:       Uses.insert(U->getParent());
  1165:     for (auto *M : Uses)
  1166:       replaceSubregUses(M, PairMap);
  1167:   }
  1168: 
  1169:   for (auto *MI : Erase) {
  1170:     MachineBasicBlock *B = MI->getParent();
  1171:     B->erase(MI);
  1172:   }
  1173: 
  1174:   return Changed;
  1175: }
  1176: 
  1177: bool HexagonSplitDoubleRegs::runOnMachineFunction(MachineFunction &MF) {
  1178:   if (skipFunction(MF.getFunction()))
  1179:     return false;
  1180: 
  1181:   LLVM_DEBUG(dbgs() << "Splitting double registers in function: "
  1182:                     << MF.getName() << '\n');
  1183: 
  1184:   auto &ST = MF.getSubtarget<HexagonSubtarget>();
  1185:   TRI = ST.getRegisterInfo();
  1186:   TII = ST.getInstrInfo();
  1187:   MRI = &MF.getRegInfo();
  1188:   MLI = &getAnalysis<MachineLoopInfoWrapperPass>().getLI();
  1189: 
  1190:   UUSetMap P2Rs;
  1191:   LoopRegMap IRM;
  1192: 
  1193:   collectIndRegs(IRM);
  1194:   partitionRegisters(P2Rs);
  1195: 
  1196:   LLVM_DEBUG({
  1197:     dbgs() << "Register partitioning: (partition #0 is fixed)\n";
  1198:     for (UUSetMap::iterator I = P2Rs.begin(), E = P2Rs.end(); I != E; ++I) {
  1199:       dbgs() << '#' << I->first << " -> ";
  1200:       dump_partition(dbgs(), I->second, *TRI);
```
- EN: It declares or implements routines such as find, createVirtualRegister, BuildMI, setReg, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 find, createVirtualRegister, BuildMI, setReg, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 1201-1227 / 第 1201-1227 行

```cpp
  1201:       dbgs() << '\n';
  1202:     }
  1203:   });
  1204: 
  1205:   bool Changed = false;
  1206:   int Limit = MaxHSDR;
  1207: 
  1208:   for (UUSetMap::iterator I = P2Rs.begin(), E = P2Rs.end(); I != E; ++I) {
  1209:     if (I->first == 0)
  1210:       continue;
  1211:     if (Limit >= 0 && Counter >= Limit)
  1212:       break;
  1213:     USet &Part = I->second;
  1214:     LLVM_DEBUG(dbgs() << "Calculating profit for partition #" << I->first
  1215:                       << '\n');
  1216:     if (!isProfitable(Part, IRM))
  1217:       continue;
  1218:     Counter++;
  1219:     Changed |= splitPartition(Part);
  1220:   }
  1221: 
  1222:   return Changed;
  1223: }
  1224: 
  1225: FunctionPass *llvm::createHexagonSplitDoubleRegs() {
  1226:   return new HexagonSplitDoubleRegs();
  1227: }
```
- EN: It declares or implements routines such as LLVM_DEBUG, splitPartition, llvm::createHexagonSplitDoubleRegs, HexagonSplitDoubleRegs, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitDoubleRegs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, splitPartition, llvm::createHexagonSplitDoubleRegs, HexagonSplitDoubleRegs 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitDoubleRegs，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, ... (34 total)`
- Hexagon symbols / Hexagon 符号: `HexagonSplitDouble, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonSplitDoubleRegs`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
