# HexagonCopyToCombine.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonCopyToCombine.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon copy patterns that can be merged or combined.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===------- HexagonCopyToCombine.cpp - Hexagon Copy-To-Combine Pass ------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // This pass replaces transfer instructions by combine instructions.
     9: // We walk along a basic block and look for two combinable instructions and try
    10: // to move them together. If we can move them next to each other we do so and
    11: // replace them with a combine instruction.
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "HexagonInstrInfo.h"
    15: #include "HexagonSubtarget.h"
    16: #include "llvm/ADT/DenseMap.h"
    17: #include "llvm/ADT/DenseSet.h"
    18: #include "llvm/CodeGen/MachineBasicBlock.h"
    19: #include "llvm/CodeGen/MachineFunction.h"
    20: #include "llvm/CodeGen/MachineFunctionPass.h"
    21: #include "llvm/CodeGen/MachineInstr.h"
    22: #include "llvm/CodeGen/MachineInstrBuilder.h"
    23: #include "llvm/CodeGen/Passes.h"
    24: #include "llvm/CodeGen/TargetRegisterInfo.h"
    25: #include "llvm/Pass.h"
    26: #include "llvm/Support/CodeGen.h"
    27: #include "llvm/Support/CommandLine.h"
    28: #include "llvm/Support/Debug.h"
    29: #include "llvm/Support/raw_ostream.h"
    30: #include "llvm/Target/TargetMachine.h"
    31: 
    32: using namespace llvm;
    33: 
    34: #define DEBUG_TYPE "hexagon-copy-combine"
    35: 
    36: static cl::opt<bool>
    37:     IsCombinesDisabled("disable-merge-into-combines", cl::Hidden,
    38: 
    39:                        cl::desc("Disable merging into combines"));
    40: static cl::opt<bool>
    41:     IsConst64Disabled("disable-const64", cl::Hidden,
    42: 
    43:                       cl::desc("Disable generation of const64"));
    44: static
    45: cl::opt<unsigned>
    46: MaxNumOfInstsBetweenNewValueStoreAndTFR("max-num-inst-between-tfr-and-nv-store",
    47:                    cl::Hidden, cl::init(4),
    48:                    cl::desc("Maximum distance between a tfr feeding a store we "
    49:                             "consider the store still to be newifiable"));
    50: 
    51: namespace {
    52: 
    53: class HexagonCopyToCombine : public MachineFunctionPass  {
    54:   const HexagonInstrInfo *TII;
    55:   const TargetRegisterInfo *TRI;
    56:   const HexagonSubtarget *ST;
    57:   bool ShouldCombineAggressively;
    58: 
    59:   DenseSet<MachineInstr *> PotentiallyNewifiableTFR;
    60:   SmallVector<MachineInstr *, 8> DbgMItoMove;
    61: 
    62: public:
    63:   static char ID;
    64: 
    65:   HexagonCopyToCombine() : MachineFunctionPass(ID) {}
    66: 
    67:   void getAnalysisUsage(AnalysisUsage &AU) const override {
    68:     MachineFunctionPass::getAnalysisUsage(AU);
    69:   }
    70: 
    71:   StringRef getPassName() const override {
    72:     return "Hexagon Copy-To-Combine Pass";
    73:   }
    74: 
    75:   bool runOnMachineFunction(MachineFunction &Fn) override;
    76: 
    77:   MachineFunctionProperties getRequiredProperties() const override {
    78:     return MachineFunctionProperties().setNoVRegs();
    79:   }
    80: 
    81: private:
    82:   MachineInstr *findPairable(MachineInstr &I1, bool &DoInsertAtI1,
    83:                              bool AllowC64);
    84: 
    85:   void findPotentialNewifiableTFRs(MachineBasicBlock &);
    86: 
    87:   void combine(MachineInstr &I1, MachineInstr &I2,
    88:                MachineBasicBlock::iterator &MI, bool DoInsertAtI1,
    89:                bool OptForSize);
    90: 
    91:   bool isSafeToMoveTogether(MachineInstr &I1, MachineInstr &I2,
    92:                             unsigned I1DestReg, unsigned I2DestReg,
    93:                             bool &DoInsertAtI1);
    94: 
    95:   void emitCombineRR(MachineBasicBlock::iterator &Before, unsigned DestReg,
    96:                      MachineOperand &HiOperand, MachineOperand &LoOperand);
    97: 
    98:   void emitCombineRI(MachineBasicBlock::iterator &Before, unsigned DestReg,
    99:                      MachineOperand &HiOperand, MachineOperand &LoOperand);
   100: 
```
- EN: It imports headers such as HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseMap.h, llvm/ADT/DenseSet.h, ... (17 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonCopyToCombine, which carry the state or API of this component.
- CN: 这里引入了 HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseMap.h, llvm/ADT/DenseSet.h, ... (17 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonCopyToCombine 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:   void emitCombineIR(MachineBasicBlock::iterator &Before, unsigned DestReg,
   102:                      MachineOperand &HiOperand, MachineOperand &LoOperand);
   103: 
   104:   void emitCombineII(MachineBasicBlock::iterator &Before, unsigned DestReg,
   105:                      MachineOperand &HiOperand, MachineOperand &LoOperand);
   106: 
   107:   void emitConst64(MachineBasicBlock::iterator &Before, unsigned DestReg,
   108:                    MachineOperand &HiOperand, MachineOperand &LoOperand);
   109: };
   110: 
   111: } // End anonymous namespace.
   112: 
   113: char HexagonCopyToCombine::ID = 0;
   114: 
   115: INITIALIZE_PASS(HexagonCopyToCombine, "hexagon-copy-combine",
   116:                 "Hexagon Copy-To-Combine Pass", false, false)
   117: 
   118: static bool isCombinableInstType(MachineInstr &MI, const HexagonInstrInfo *TII,
   119:                                  bool ShouldCombineAggressively) {
   120:   switch (MI.getOpcode()) {
   121:   case Hexagon::A2_tfr: {
   122:     // A COPY instruction can be combined if its arguments are IntRegs (32bit).
   123:     const MachineOperand &Op0 = MI.getOperand(0);
   124:     const MachineOperand &Op1 = MI.getOperand(1);
   125:     assert(Op0.isReg() && Op1.isReg());
   126: 
   127:     Register DestReg = Op0.getReg();
   128:     Register SrcReg = Op1.getReg();
   129:     return Hexagon::IntRegsRegClass.contains(DestReg) &&
   130:            Hexagon::IntRegsRegClass.contains(SrcReg);
   131:   }
   132: 
   133:   case Hexagon::A2_tfrsi: {
   134:     // A transfer-immediate can be combined if its argument is a signed 8bit
   135:     // value.
   136:     const MachineOperand &Op0 = MI.getOperand(0);
   137:     const MachineOperand &Op1 = MI.getOperand(1);
   138:     assert(Op0.isReg());
   139: 
   140:     Register DestReg = Op0.getReg();
   141:     // Ensure that TargetFlags are MO_NO_FLAG for a global. This is a
   142:     // workaround for an ABI bug that prevents GOT relocations on combine
   143:     // instructions
   144:     if (!Op1.isImm() && Op1.getTargetFlags() != HexagonII::MO_NO_FLAG)
   145:       return false;
   146: 
   147:     // Only combine constant extended A2_tfrsi if we are in aggressive mode.
   148:     bool NotExt = Op1.isImm() && isInt<8>(Op1.getImm());
   149:     return Hexagon::IntRegsRegClass.contains(DestReg) &&
   150:            (ShouldCombineAggressively || NotExt);
   151:   }
   152: 
   153:   case Hexagon::V6_vassign:
   154:     return true;
   155: 
   156:   default:
   157:     break;
   158:   }
   159: 
   160:   return false;
   161: }
   162: 
   163: template <unsigned N> static bool isGreaterThanNBitTFRI(const MachineInstr &I) {
   164:   if (I.getOpcode() == Hexagon::TFRI64_V4 ||
   165:       I.getOpcode() == Hexagon::A2_tfrsi) {
   166:     const MachineOperand &Op = I.getOperand(1);
   167:     return !Op.isImm() || !isInt<N>(Op.getImm());
   168:   }
   169:   return false;
   170: }
   171: 
   172: /// areCombinableOperations - Returns true if the two instruction can be merge
   173: /// into a combine (ignoring register constraints).
   174: static bool areCombinableOperations(const TargetRegisterInfo *TRI,
   175:                                     MachineInstr &HighRegInst,
   176:                                     MachineInstr &LowRegInst, bool AllowC64) {
   177:   unsigned HiOpc = HighRegInst.getOpcode();
   178:   unsigned LoOpc = LowRegInst.getOpcode();
   179: 
   180:   auto verifyOpc = [](unsigned Opc) -> void {
   181:     switch (Opc) {
   182:       case Hexagon::A2_tfr:
   183:       case Hexagon::A2_tfrsi:
   184:       case Hexagon::V6_vassign:
   185:         break;
   186:       default:
   187:         llvm_unreachable("Unexpected opcode");
   188:     }
   189:   };
   190:   verifyOpc(HiOpc);
   191:   verifyOpc(LoOpc);
   192: 
   193:   if (HiOpc == Hexagon::V6_vassign || LoOpc == Hexagon::V6_vassign)
   194:     return HiOpc == LoOpc;
   195: 
   196:   if (!AllowC64) {
   197:     // There is no combine of two constant extended values.
   198:     if (isGreaterThanNBitTFRI<8>(HighRegInst) &&
   199:         isGreaterThanNBitTFRI<6>(LowRegInst))
   200:       return false;
```
- EN: It declares or implements routines such as emitCombineIR, emitCombineII, emitConst64, INITIALIZE_PASS, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCopyToCombine, HexagonInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 emitCombineIR, emitCombineII, emitConst64, INITIALIZE_PASS, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine, HexagonInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201:   }
   202: 
   203:   // There is a combine of two constant extended values into CONST64,
   204:   // provided both constants are true immediates.
   205:   if (isGreaterThanNBitTFRI<16>(HighRegInst) &&
   206:       isGreaterThanNBitTFRI<16>(LowRegInst) && !IsConst64Disabled)
   207:     return (HighRegInst.getOperand(1).isImm() &&
   208:             LowRegInst.getOperand(1).isImm());
   209: 
   210:   // There is no combine of two constant extended values, unless handled above
   211:   // Make both 8-bit size checks to allow both combine (#,##) and combine(##,#)
   212:   if (isGreaterThanNBitTFRI<8>(HighRegInst) &&
   213:       isGreaterThanNBitTFRI<8>(LowRegInst))
   214:     return false;
   215: 
   216:   return true;
   217: }
   218: 
   219: static bool isEvenReg(unsigned Reg) {
   220:   assert(Register::isPhysicalRegister(Reg));
   221:   if (Hexagon::IntRegsRegClass.contains(Reg))
   222:     return (Reg - Hexagon::R0) % 2 == 0;
   223:   if (Hexagon::HvxVRRegClass.contains(Reg))
   224:     return (Reg - Hexagon::V0) % 2 == 0;
   225:   llvm_unreachable("Invalid register");
   226: }
   227: 
   228: static void removeKillInfo(MachineInstr &MI, unsigned RegNotKilled) {
   229:   for (MachineOperand &Op : MI.operands())
   230:     if (Op.isReg() && Op.getReg() == RegNotKilled && Op.isKill())
   231:       Op.setIsKill(false);
   232: }
   233: 
   234: /// Returns true if it is unsafe to move a copy instruction from \p UseReg to
   235: /// \p DestReg over the instruction \p MI.
   236: static bool isUnsafeToMoveAcross(MachineInstr &MI, unsigned UseReg,
   237:                                  unsigned DestReg,
   238:                                  const TargetRegisterInfo *TRI) {
   239:   return (UseReg && (MI.modifiesRegister(UseReg, TRI))) ||
   240:          MI.modifiesRegister(DestReg, TRI) || MI.readsRegister(DestReg, TRI) ||
   241:          MI.hasUnmodeledSideEffects() || MI.isInlineAsm() ||
   242:          MI.isMetaInstruction();
   243: }
   244: 
   245: static Register UseReg(const MachineOperand& MO) {
   246:   return MO.isReg() ? MO.getReg() : Register();
   247: }
   248: 
   249: /// isSafeToMoveTogether - Returns true if it is safe to move I1 next to I2 such
   250: /// that the two instructions can be paired in a combine.
   251: bool HexagonCopyToCombine::isSafeToMoveTogether(MachineInstr &I1,
   252:                                                 MachineInstr &I2,
   253:                                                 unsigned I1DestReg,
   254:                                                 unsigned I2DestReg,
   255:                                                 bool &DoInsertAtI1) {
   256:   Register I2UseReg = UseReg(I2.getOperand(1));
   257: 
   258:   // It is not safe to move I1 and I2 into one combine if I2 has a true
   259:   // dependence on I1.
   260:   if (I2UseReg && I1.modifiesRegister(I2UseReg, TRI))
   261:     return false;
   262: 
   263:   bool isSafe = true;
   264: 
   265:   // First try to move I2 towards I1.
   266:   {
   267:     // A reverse_iterator instantiated like below starts before I2, and I1
   268:     // respectively.
   269:     // Look at instructions I in between I2 and (excluding) I1.
   270:     MachineBasicBlock::reverse_iterator I = ++I2.getIterator().getReverse();
   271:     MachineBasicBlock::reverse_iterator End = I1.getIterator().getReverse();
   272:     // At 03 we got better results (dhrystone!) by being more conservative.
   273:     if (!ShouldCombineAggressively)
   274:       End = ++I1.getIterator().getReverse();
   275:     // If I2 kills its operand and we move I2 over an instruction that also
   276:     // uses I2's use reg we need to modify that (first) instruction to now kill
   277:     // this reg.
   278:     unsigned KilledOperand = 0;
   279:     if (I2.killsRegister(I2UseReg, /*TRI=*/nullptr))
   280:       KilledOperand = I2UseReg;
   281:     MachineInstr *KillingInstr = nullptr;
   282: 
   283:     for (; I != End; ++I) {
   284:       // If the intervening instruction I:
   285:       //   * modifies I2's use reg
   286:       //   * modifies I2's def reg
   287:       //   * reads I2's def reg
   288:       //   * or has unmodelled side effects
   289:       // we can't move I2 across it.
   290:       if (I->isDebugInstr())
   291:         continue;
   292: 
   293:       if (isUnsafeToMoveAcross(*I, I2UseReg, I2DestReg, TRI)) {
   294:         isSafe = false;
   295:         break;
   296:       }
   297: 
   298:       // Update first use of the killed operand.
   299:       if (!KillingInstr && KilledOperand &&
   300:           I->readsRegister(KilledOperand, TRI))
```
- EN: It declares or implements routines such as isEvenReg, assert, llvm_unreachable, removeKillInfo, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCopyToCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isEvenReg, assert, llvm_unreachable, removeKillInfo, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:         KillingInstr = &*I;
   302:     }
   303:     if (isSafe) {
   304:       // Update the intermediate instruction to with the kill flag.
   305:       if (KillingInstr) {
   306:         bool Added = KillingInstr->addRegisterKilled(KilledOperand, TRI, true);
   307:         (void)Added; // suppress compiler warning
   308:         assert(Added && "Must successfully update kill flag");
   309:         removeKillInfo(I2, KilledOperand);
   310:       }
   311:       DoInsertAtI1 = true;
   312:       return true;
   313:     }
   314:   }
   315: 
   316:   // Try to move I1 towards I2.
   317:   {
   318:     // Look at instructions I in between I1 and (excluding) I2.
   319:     MachineBasicBlock::iterator I(I1), End(I2);
   320:     // At O3 we got better results (dhrystone) by being more conservative here.
   321:     if (!ShouldCombineAggressively)
   322:       End = std::next(MachineBasicBlock::iterator(I2));
   323:     Register I1UseReg = UseReg(I1.getOperand(1));
   324:     // Track killed operands. If we move across an instruction that kills our
   325:     // operand, we need to update the kill information on the moved I1. It kills
   326:     // the operand now.
   327:     MachineInstr *KillingInstr = nullptr;
   328:     unsigned KilledOperand = 0;
   329: 
   330:     while(++I != End) {
   331:       MachineInstr &MI = *I;
   332:       // If the intervening instruction MI:
   333:       //   * modifies I1's use reg
   334:       //   * modifies I1's def reg
   335:       //   * reads I1's def reg
   336:       //   * or has unmodelled side effects
   337:       //   We introduce this special case because llvm has no api to remove a
   338:       //   kill flag for a register (a removeRegisterKilled() analogous to
   339:       //   addRegisterKilled) that handles aliased register correctly.
   340:       //   * or has a killed aliased register use of I1's use reg
   341:       //           %d4 = A2_tfrpi 16
   342:       //           %r6 = A2_tfr %r9
   343:       //           %r8 = KILL %r8, implicit killed %d4
   344:       //      If we want to move R6 = across the KILL instruction we would have
   345:       //      to remove the implicit killed %d4 operand. For now, we are
   346:       //      conservative and disallow the move.
   347:       // we can't move I1 across it.
   348:       if (MI.isDebugInstr()) {
   349:         if (MI.readsRegister(I1DestReg, TRI)) // Move this instruction after I2.
   350:           DbgMItoMove.push_back(&MI);
   351:         continue;
   352:       }
   353: 
   354:       if (isUnsafeToMoveAcross(MI, I1UseReg, I1DestReg, TRI) ||
   355:           // Check for an aliased register kill. Bail out if we see one.
   356:           (!MI.killsRegister(I1UseReg, /*TRI=*/nullptr) &&
   357:            MI.killsRegister(I1UseReg, TRI)))
   358:         return false;
   359: 
   360:       // Check for an exact kill (registers match).
   361:       if (I1UseReg && MI.killsRegister(I1UseReg, /*TRI=*/nullptr)) {
   362:         assert(!KillingInstr && "Should only see one killing instruction");
   363:         KilledOperand = I1UseReg;
   364:         KillingInstr = &MI;
   365:       }
   366:     }
   367:     if (KillingInstr) {
   368:       removeKillInfo(*KillingInstr, KilledOperand);
   369:       // Update I1 to set the kill flag. This flag will later be picked up by
   370:       // the new COMBINE instruction.
   371:       bool Added = I1.addRegisterKilled(KilledOperand, TRI);
   372:       (void)Added; // suppress compiler warning
   373:       assert(Added && "Must successfully update kill flag");
   374:     }
   375:     DoInsertAtI1 = false;
   376:   }
   377: 
   378:   return true;
   379: }
   380: 
   381: /// findPotentialNewifiableTFRs - Finds transfers that feed stores that could be
   382: /// newified. (A use of a 64 bit register define can not be newified)
   383: void
   384: HexagonCopyToCombine::findPotentialNewifiableTFRs(MachineBasicBlock &BB) {
   385:   DenseMap<unsigned, MachineInstr *> LastDef;
   386:   for (MachineInstr &MI : BB) {
   387:     if (MI.isDebugInstr())
   388:       continue;
   389: 
   390:     // Mark TFRs that feed a potential new value store as such.
   391:     if (TII->mayBeNewStore(MI)) {
   392:       // Look for uses of TFR instructions.
   393:       for (const MachineOperand &Op : MI.operands()) {
   394:         // Skip over anything except register uses.
   395:         if (!Op.isReg() || !Op.isUse() || !Op.getReg())
   396:           continue;
   397: 
   398:         // Look for the defining instruction.
   399:         Register Reg = Op.getReg();
   400:         MachineInstr *DefInst = LastDef[Reg];
```
- EN: It declares or implements routines such as addRegisterKilled, assert, removeKillInfo, and, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCopyToCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addRegisterKilled, assert, removeKillInfo, and, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:         if (!DefInst)
   402:           continue;
   403:         if (!isCombinableInstType(*DefInst, TII, ShouldCombineAggressively))
   404:           continue;
   405: 
   406:         // Only close newifiable stores should influence the decision.
   407:         // Ignore the debug instructions in between.
   408:         MachineBasicBlock::iterator It(DefInst);
   409:         unsigned NumInstsToDef = 0;
   410:         while (&*It != &MI) {
   411:           if (!It->isDebugInstr())
   412:             ++NumInstsToDef;
   413:           ++It;
   414:         }
   415: 
   416:         if (NumInstsToDef > MaxNumOfInstsBetweenNewValueStoreAndTFR)
   417:           continue;
   418: 
   419:         PotentiallyNewifiableTFR.insert(DefInst);
   420:       }
   421:       // Skip to next instruction.
   422:       continue;
   423:     }
   424: 
   425:     // Put instructions that last defined integer or double registers into the
   426:     // map.
   427:     for (MachineOperand &Op : MI.operands()) {
   428:       if (Op.isReg()) {
   429:         if (!Op.isDef() || !Op.getReg())
   430:           continue;
   431:         Register Reg = Op.getReg();
   432:         if (Hexagon::DoubleRegsRegClass.contains(Reg)) {
   433:           for (MCPhysReg SubReg : TRI->subregs(Reg))
   434:             LastDef[SubReg] = &MI;
   435:         } else if (Hexagon::IntRegsRegClass.contains(Reg))
   436:           LastDef[Reg] = &MI;
   437:       } else if (Op.isRegMask()) {
   438:         for (unsigned Reg : Hexagon::IntRegsRegClass)
   439:           if (Op.clobbersPhysReg(Reg))
   440:             LastDef[Reg] = &MI;
   441:       }
   442:     }
   443:   }
   444: }
   445: 
   446: bool HexagonCopyToCombine::runOnMachineFunction(MachineFunction &MF) {
   447:   if (skipFunction(MF.getFunction()))
   448:     return false;
   449: 
   450:   if (IsCombinesDisabled) return false;
   451: 
   452:   bool HasChanged = false;
   453: 
   454:   // Get target info.
   455:   ST = &MF.getSubtarget<HexagonSubtarget>();
   456:   TRI = ST->getRegisterInfo();
   457:   TII = ST->getInstrInfo();
   458: 
   459:   const Function &F = MF.getFunction();
   460:   bool OptForSize = F.hasOptSize();
   461: 
   462:   // Combine aggressively (for code size)
   463:   ShouldCombineAggressively =
   464:       MF.getTarget().getOptLevel() <= CodeGenOptLevel::Default;
   465: 
   466:   // Disable CONST64 for tiny core since it takes a LD resource.
   467:   if (!OptForSize && ST->isTinyCore())
   468:     IsConst64Disabled = true;
   469: 
   470:   // Traverse basic blocks.
   471:   for (MachineBasicBlock &MBB : MF) {
   472:     PotentiallyNewifiableTFR.clear();
   473:     findPotentialNewifiableTFRs(MBB);
   474: 
   475:     // Traverse instructions in basic block.
   476:     for (MachineBasicBlock::iterator MI = MBB.begin(), End = MBB.end();
   477:          MI != End;) {
   478:       MachineInstr &I1 = *MI++;
   479: 
   480:       if (I1.isDebugInstr())
   481:         continue;
   482: 
   483:       // Don't combine a TFR whose user could be newified (instructions that
   484:       // define double registers can not be newified - Programmer's Ref Manual
   485:       // 5.4.2 New-value stores).
   486:       if (ShouldCombineAggressively && PotentiallyNewifiableTFR.count(&I1))
   487:         continue;
   488: 
   489:       // Ignore instructions that are not combinable.
   490:       if (!isCombinableInstType(I1, TII, ShouldCombineAggressively))
   491:         continue;
   492: 
   493:       // Find a second instruction that can be merged into a combine
   494:       // instruction. In addition, also find all the debug instructions that
   495:       // need to be moved along with it.
   496:       bool DoInsertAtI1 = false;
   497:       DbgMItoMove.clear();
   498:       MachineInstr *I2 = findPairable(I1, DoInsertAtI1, OptForSize);
   499:       if (I2) {
   500:         HasChanged = true;
```
- EN: It declares or implements routines such as It, insert, getReg, HexagonCopyToCombine::runOnMachineFunction, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCopyToCombine, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 It, insert, getReg, HexagonCopyToCombine::runOnMachineFunction, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:         combine(I1, *I2, MI, DoInsertAtI1, OptForSize);
   502:       }
   503:     }
   504:   }
   505: 
   506:   return HasChanged;
   507: }
   508: 
   509: /// findPairable - Returns an instruction that can be merged with \p I1 into a
   510: /// COMBINE instruction or 0 if no such instruction can be found. Returns true
   511: /// in \p DoInsertAtI1 if the combine must be inserted at instruction \p I1
   512: /// false if the combine must be inserted at the returned instruction.
   513: MachineInstr *HexagonCopyToCombine::findPairable(MachineInstr &I1,
   514:                                                  bool &DoInsertAtI1,
   515:                                                  bool AllowC64) {
   516:   MachineBasicBlock::iterator I2 = std::next(MachineBasicBlock::iterator(I1));
   517:   while (I2 != I1.getParent()->end() && I2->isDebugInstr())
   518:     ++I2;
   519: 
   520:   Register I1DestReg = I1.getOperand(0).getReg();
   521: 
   522:   for (MachineBasicBlock::iterator End = I1.getParent()->end(); I2 != End;
   523:        ++I2) {
   524:     // Bail out early if we see a second definition of I1DestReg.
   525:     if (I2->modifiesRegister(I1DestReg, TRI))
   526:       break;
   527: 
   528:     // Ignore non-combinable instructions.
   529:     if (!isCombinableInstType(*I2, TII, ShouldCombineAggressively))
   530:       continue;
   531: 
   532:     // Don't combine a TFR whose user could be newified.
   533:     if (ShouldCombineAggressively && PotentiallyNewifiableTFR.count(&*I2))
   534:       continue;
   535: 
   536:     Register I2DestReg = I2->getOperand(0).getReg();
   537: 
   538:     // Check that registers are adjacent and that the first destination register
   539:     // is even.
   540:     bool IsI1LowReg = (I2DestReg - I1DestReg) == 1;
   541:     bool IsI2LowReg = (I1DestReg - I2DestReg) == 1;
   542:     unsigned FirstRegIndex = IsI1LowReg ? I1DestReg : I2DestReg;
   543:     if ((!IsI1LowReg && !IsI2LowReg) || !isEvenReg(FirstRegIndex))
   544:       continue;
   545: 
   546:     // Check that the two instructions are combinable.
   547:     // The order matters because in a A2_tfrsi we might can encode a int8 as
   548:     // the hi reg operand but only a uint6 as the low reg operand.
   549:     if ((IsI2LowReg && !areCombinableOperations(TRI, I1, *I2, AllowC64)) ||
   550:         (IsI1LowReg && !areCombinableOperations(TRI, *I2, I1, AllowC64)))
   551:       break;
   552: 
   553:     if (isSafeToMoveTogether(I1, *I2, I1DestReg, I2DestReg, DoInsertAtI1))
   554:       return &*I2;
   555: 
   556:     // Not safe. Stop searching.
   557:     break;
   558:   }
   559:   return nullptr;
   560: }
   561: 
   562: void HexagonCopyToCombine::combine(MachineInstr &I1, MachineInstr &I2,
   563:                                    MachineBasicBlock::iterator &MI,
   564:                                    bool DoInsertAtI1, bool OptForSize) {
   565:   // We are going to delete I2. If MI points to I2 advance it to the next
   566:   // instruction.
   567:   if (MI == I2.getIterator())
   568:     ++MI;
   569: 
   570:   // Figure out whether I1 or I2 goes into the lowreg part.
   571:   Register I1DestReg = I1.getOperand(0).getReg();
   572:   Register I2DestReg = I2.getOperand(0).getReg();
   573:   bool IsI1Loreg = (I2DestReg - I1DestReg) == 1;
   574:   unsigned LoRegDef = IsI1Loreg ? I1DestReg : I2DestReg;
   575:   unsigned SubLo;
   576: 
   577:   const TargetRegisterClass *SuperRC = nullptr;
   578:   if (Hexagon::IntRegsRegClass.contains(LoRegDef)) {
   579:     SuperRC = &Hexagon::DoubleRegsRegClass;
   580:     SubLo = Hexagon::isub_lo;
   581:   } else if (Hexagon::HvxVRRegClass.contains(LoRegDef)) {
   582:     assert(ST->useHVXOps());
   583:     SuperRC = &Hexagon::HvxWRRegClass;
   584:     SubLo = Hexagon::vsub_lo;
   585:   } else
   586:     llvm_unreachable("Unexpected register class");
   587: 
   588:   // Get the double word register.
   589:   MCRegister DoubleRegDest = TRI->getMatchingSuperReg(LoRegDef, SubLo, SuperRC);
   590:   assert(DoubleRegDest.isValid() && "Expect a valid register");
   591: 
   592:   // Setup source operands.
   593:   MachineOperand &LoOperand = IsI1Loreg ? I1.getOperand(1) : I2.getOperand(1);
   594:   MachineOperand &HiOperand = IsI1Loreg ? I2.getOperand(1) : I1.getOperand(1);
   595: 
   596:   // Figure out which source is a register and which a constant.
   597:   bool IsHiReg = HiOperand.isReg();
   598:   bool IsLoReg = LoOperand.isReg();
   599: 
   600:   // There is a combine of two constant extended values into CONST64.
```
- EN: It declares or implements routines such as combine, HexagonCopyToCombine::findPairable, std::next, getOperand, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCopyToCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 combine, HexagonCopyToCombine::findPairable, std::next, getOperand, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:   bool IsC64 = OptForSize && LoOperand.isImm() && HiOperand.isImm() &&
   602:                isGreaterThanNBitTFRI<16>(I1) && isGreaterThanNBitTFRI<16>(I2);
   603: 
   604:   MachineBasicBlock::iterator InsertPt(DoInsertAtI1 ? I1 : I2);
   605:   // Emit combine.
   606:   if (IsHiReg && IsLoReg)
   607:     emitCombineRR(InsertPt, DoubleRegDest, HiOperand, LoOperand);
   608:   else if (IsHiReg)
   609:     emitCombineRI(InsertPt, DoubleRegDest, HiOperand, LoOperand);
   610:   else if (IsLoReg)
   611:     emitCombineIR(InsertPt, DoubleRegDest, HiOperand, LoOperand);
   612:   else if (IsC64 && !IsConst64Disabled)
   613:     emitConst64(InsertPt, DoubleRegDest, HiOperand, LoOperand);
   614:   else
   615:     emitCombineII(InsertPt, DoubleRegDest, HiOperand, LoOperand);
   616: 
   617:   // Move debug instructions along with I1 if it's being
   618:   // moved towards I2.
   619:   if (!DoInsertAtI1 && DbgMItoMove.size() != 0) {
   620:     // Insert debug instructions at the new location before I2.
   621:     MachineBasicBlock *BB = InsertPt->getParent();
   622:     for (auto *NewMI : DbgMItoMove) {
   623:       // If iterator MI is pointing to DEBUG_VAL, make sure
   624:       // MI now points to next relevant instruction.
   625:       if (NewMI == MI)
   626:         ++MI;
   627:       BB->splice(InsertPt, BB, NewMI);
   628:     }
   629:   }
   630: 
   631:   I1.eraseFromParent();
   632:   I2.eraseFromParent();
   633: }
   634: 
   635: void HexagonCopyToCombine::emitConst64(MachineBasicBlock::iterator &InsertPt,
   636:                                        unsigned DoubleDestReg,
   637:                                        MachineOperand &HiOperand,
   638:                                        MachineOperand &LoOperand) {
   639:   LLVM_DEBUG(dbgs() << "Found a CONST64\n");
   640: 
   641:   DebugLoc DL = InsertPt->getDebugLoc();
   642:   MachineBasicBlock *BB = InsertPt->getParent();
   643:   assert(LoOperand.isImm() && HiOperand.isImm() &&
   644:          "Both operands must be immediate");
   645: 
   646:   int64_t V = HiOperand.getImm();
   647:   V = (V << 32) | (0x0ffffffffLL & LoOperand.getImm());
   648:   BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::CONST64), DoubleDestReg)
   649:     .addImm(V);
   650: }
   651: 
   652: void HexagonCopyToCombine::emitCombineII(MachineBasicBlock::iterator &InsertPt,
   653:                                          unsigned DoubleDestReg,
   654:                                          MachineOperand &HiOperand,
   655:                                          MachineOperand &LoOperand) {
   656:   DebugLoc DL = InsertPt->getDebugLoc();
   657:   MachineBasicBlock *BB = InsertPt->getParent();
   658: 
   659:   // Handle globals.
   660:   if (HiOperand.isGlobal()) {
   661:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A2_combineii), DoubleDestReg)
   662:       .addGlobalAddress(HiOperand.getGlobal(), HiOperand.getOffset(),
   663:                         HiOperand.getTargetFlags())
   664:       .addImm(LoOperand.getImm());
   665:     return;
   666:   }
   667:   if (LoOperand.isGlobal()) {
   668:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineii), DoubleDestReg)
   669:       .addImm(HiOperand.getImm())
   670:       .addGlobalAddress(LoOperand.getGlobal(), LoOperand.getOffset(),
   671:                         LoOperand.getTargetFlags());
   672:     return;
   673:   }
   674: 
   675:   // Handle block addresses.
   676:   if (HiOperand.isBlockAddress()) {
   677:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A2_combineii), DoubleDestReg)
   678:       .addBlockAddress(HiOperand.getBlockAddress(), HiOperand.getOffset(),
   679:                        HiOperand.getTargetFlags())
   680:       .addImm(LoOperand.getImm());
   681:     return;
   682:   }
   683:   if (LoOperand.isBlockAddress()) {
   684:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineii), DoubleDestReg)
   685:       .addImm(HiOperand.getImm())
   686:       .addBlockAddress(LoOperand.getBlockAddress(), LoOperand.getOffset(),
   687:                        LoOperand.getTargetFlags());
   688:     return;
   689:   }
   690: 
   691:   // Handle jump tables.
   692:   if (HiOperand.isJTI()) {
   693:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A2_combineii), DoubleDestReg)
   694:       .addJumpTableIndex(HiOperand.getIndex(), HiOperand.getTargetFlags())
   695:       .addImm(LoOperand.getImm());
   696:     return;
   697:   }
   698:   if (LoOperand.isJTI()) {
   699:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineii), DoubleDestReg)
   700:       .addImm(HiOperand.getImm())
```
- EN: It declares or implements routines such as isImm, InsertPt, emitCombineII, getParent, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCopyToCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isImm, InsertPt, emitCombineII, getParent, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:       .addJumpTableIndex(LoOperand.getIndex(), LoOperand.getTargetFlags());
   702:     return;
   703:   }
   704: 
   705:   // Handle constant pools.
   706:   if (HiOperand.isCPI()) {
   707:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A2_combineii), DoubleDestReg)
   708:       .addConstantPoolIndex(HiOperand.getIndex(), HiOperand.getOffset(),
   709:                             HiOperand.getTargetFlags())
   710:       .addImm(LoOperand.getImm());
   711:     return;
   712:   }
   713:   if (LoOperand.isCPI()) {
   714:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineii), DoubleDestReg)
   715:       .addImm(HiOperand.getImm())
   716:       .addConstantPoolIndex(LoOperand.getIndex(), LoOperand.getOffset(),
   717:                             LoOperand.getTargetFlags());
   718:     return;
   719:   }
   720: 
   721:   // First preference should be given to Hexagon::A2_combineii instruction
   722:   // as it can include U6 (in Hexagon::A4_combineii) as well.
   723:   // In this instruction, HiOperand is const extended, if required.
   724:   if (isInt<8>(LoOperand.getImm())) {
   725:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A2_combineii), DoubleDestReg)
   726:       .addImm(HiOperand.getImm())
   727:       .addImm(LoOperand.getImm());
   728:       return;
   729:   }
   730: 
   731:   // In this instruction, LoOperand is const extended, if required.
   732:   if (isInt<8>(HiOperand.getImm())) {
   733:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineii), DoubleDestReg)
   734:       .addImm(HiOperand.getImm())
   735:       .addImm(LoOperand.getImm());
   736:     return;
   737:   }
   738: 
   739:   // Insert new combine instruction.
   740:   //  DoubleRegDest = combine #HiImm, #LoImm
   741:   BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A2_combineii), DoubleDestReg)
   742:     .addImm(HiOperand.getImm())
   743:     .addImm(LoOperand.getImm());
   744: }
   745: 
   746: void HexagonCopyToCombine::emitCombineIR(MachineBasicBlock::iterator &InsertPt,
   747:                                          unsigned DoubleDestReg,
   748:                                          MachineOperand &HiOperand,
   749:                                          MachineOperand &LoOperand) {
   750:   Register LoReg = LoOperand.getReg();
   751:   RegState LoRegKillFlag = getKillRegState(LoOperand.isKill());
   752: 
   753:   DebugLoc DL = InsertPt->getDebugLoc();
   754:   MachineBasicBlock *BB = InsertPt->getParent();
   755: 
   756:   // Handle globals.
   757:   if (HiOperand.isGlobal()) {
   758:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineir), DoubleDestReg)
   759:       .addGlobalAddress(HiOperand.getGlobal(), HiOperand.getOffset(),
   760:                         HiOperand.getTargetFlags())
   761:       .addReg(LoReg, LoRegKillFlag);
   762:     return;
   763:   }
   764:   // Handle block addresses.
   765:   if (HiOperand.isBlockAddress()) {
   766:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineir), DoubleDestReg)
   767:       .addBlockAddress(HiOperand.getBlockAddress(), HiOperand.getOffset(),
   768:                        HiOperand.getTargetFlags())
   769:       .addReg(LoReg, LoRegKillFlag);
   770:     return;
   771:   }
   772:   // Handle jump tables.
   773:   if (HiOperand.isJTI()) {
   774:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineir), DoubleDestReg)
   775:       .addJumpTableIndex(HiOperand.getIndex(), HiOperand.getTargetFlags())
   776:       .addReg(LoReg, LoRegKillFlag);
   777:     return;
   778:   }
   779:   // Handle constant pools.
   780:   if (HiOperand.isCPI()) {
   781:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineir), DoubleDestReg)
   782:       .addConstantPoolIndex(HiOperand.getIndex(), HiOperand.getOffset(),
   783:                             HiOperand.getTargetFlags())
   784:       .addReg(LoReg, LoRegKillFlag);
   785:     return;
   786:   }
   787:   // Insert new combine instruction.
   788:   //  DoubleRegDest = combine #HiImm, LoReg
   789:   BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineir), DoubleDestReg)
   790:     .addImm(HiOperand.getImm())
   791:     .addReg(LoReg, LoRegKillFlag);
   792: }
   793: 
   794: void HexagonCopyToCombine::emitCombineRI(MachineBasicBlock::iterator &InsertPt,
   795:                                          unsigned DoubleDestReg,
   796:                                          MachineOperand &HiOperand,
   797:                                          MachineOperand &LoOperand) {
   798:   RegState HiRegKillFlag = getKillRegState(HiOperand.isKill());
   799:   Register HiReg = HiOperand.getReg();
   800: 
```
- EN: It declares or implements routines such as addJumpTableIndex, BuildMI, U6, HexagonCopyToCombine::emitCombineIR, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCopyToCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addJumpTableIndex, BuildMI, U6, HexagonCopyToCombine::emitCombineIR, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine，说明了它与同级后端组件的连接关系。

### Lines 801-873 / 第 801-873 行

```cpp
   801:   DebugLoc DL = InsertPt->getDebugLoc();
   802:   MachineBasicBlock *BB = InsertPt->getParent();
   803: 
   804:   // Handle global.
   805:   if (LoOperand.isGlobal()) {
   806:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineri), DoubleDestReg)
   807:       .addReg(HiReg, HiRegKillFlag)
   808:       .addGlobalAddress(LoOperand.getGlobal(), LoOperand.getOffset(),
   809:                         LoOperand.getTargetFlags());
   810:     return;
   811:   }
   812:   // Handle block addresses.
   813:   if (LoOperand.isBlockAddress()) {
   814:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineri), DoubleDestReg)
   815:       .addReg(HiReg, HiRegKillFlag)
   816:       .addBlockAddress(LoOperand.getBlockAddress(), LoOperand.getOffset(),
   817:                        LoOperand.getTargetFlags());
   818:     return;
   819:   }
   820:   // Handle jump tables.
   821:   if (LoOperand.isJTI()) {
   822:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineri), DoubleDestReg)
   823:       .addReg(HiOperand.getReg(), HiRegKillFlag)
   824:       .addJumpTableIndex(LoOperand.getIndex(), LoOperand.getTargetFlags());
   825:     return;
   826:   }
   827:   // Handle constant pools.
   828:   if (LoOperand.isCPI()) {
   829:     BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineri), DoubleDestReg)
   830:       .addReg(HiOperand.getReg(), HiRegKillFlag)
   831:       .addConstantPoolIndex(LoOperand.getIndex(), LoOperand.getOffset(),
   832:                             LoOperand.getTargetFlags());
   833:     return;
   834:   }
   835: 
   836:   // Insert new combine instruction.
   837:   //  DoubleRegDest = combine HiReg, #LoImm
   838:   BuildMI(*BB, InsertPt, DL, TII->get(Hexagon::A4_combineri), DoubleDestReg)
   839:     .addReg(HiReg, HiRegKillFlag)
   840:     .addImm(LoOperand.getImm());
   841: }
   842: 
   843: void HexagonCopyToCombine::emitCombineRR(MachineBasicBlock::iterator &InsertPt,
   844:                                          unsigned DoubleDestReg,
   845:                                          MachineOperand &HiOperand,
   846:                                          MachineOperand &LoOperand) {
   847:   RegState LoRegKillFlag = getKillRegState(LoOperand.isKill());
   848:   RegState HiRegKillFlag = getKillRegState(HiOperand.isKill());
   849:   Register LoReg = LoOperand.getReg();
   850:   Register HiReg = HiOperand.getReg();
   851: 
   852:   DebugLoc DL = InsertPt->getDebugLoc();
   853:   MachineBasicBlock *BB = InsertPt->getParent();
   854: 
   855:   // Insert new combine instruction.
   856:   //  DoubleRegDest = combine HiReg, LoReg
   857:   unsigned NewOpc;
   858:   if (Hexagon::DoubleRegsRegClass.contains(DoubleDestReg)) {
   859:     NewOpc = Hexagon::A2_combinew;
   860:   } else if (Hexagon::HvxWRRegClass.contains(DoubleDestReg)) {
   861:     assert(ST->useHVXOps());
   862:     NewOpc = Hexagon::V6_vcombine;
   863:   } else
   864:     llvm_unreachable("Unexpected register");
   865: 
   866:   BuildMI(*BB, InsertPt, DL, TII->get(NewOpc), DoubleDestReg)
   867:     .addReg(HiReg, HiRegKillFlag)
   868:     .addReg(LoReg, LoRegKillFlag);
   869: }
   870: 
   871: FunctionPass *llvm::createHexagonCopyToCombine() {
   872:   return new HexagonCopyToCombine();
   873: }
```
- EN: It declares or implements routines such as getDebugLoc, getParent, BuildMI, HexagonCopyToCombine::emitCombineRR, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonCopyToCombine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getDebugLoc, getParent, BuildMI, HexagonCopyToCombine::emitCombineRR, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonCopyToCombine，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseMap.h, llvm/ADT/DenseSet.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/Passes.h, ... (17 total)`
- Hexagon symbols / Hexagon 符号: `HexagonCopyToCombine, HexagonInstrInfo, HexagonSubtarget, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
