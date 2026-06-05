# HexagonFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonFrameLowering.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon stack frame lowering and prologue/epilogue generation.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===- HexagonFrameLowering.cpp - Define frame lowering -------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //
     8: //===----------------------------------------------------------------------===//
     9: 
    10: #include "HexagonFrameLowering.h"
    11: #include "HexagonBlockRanges.h"
    12: #include "HexagonInstrInfo.h"
    13: #include "HexagonMachineFunctionInfo.h"
    14: #include "HexagonRegisterInfo.h"
    15: #include "HexagonSubtarget.h"
    16: #include "HexagonTargetMachine.h"
    17: #include "MCTargetDesc/HexagonBaseInfo.h"
    18: #include "llvm/ADT/BitVector.h"
    19: #include "llvm/ADT/DenseMap.h"
    20: #include "llvm/ADT/PostOrderIterator.h"
    21: #include "llvm/ADT/SetVector.h"
    22: #include "llvm/ADT/SmallSet.h"
    23: #include "llvm/ADT/SmallVector.h"
    24: #include "llvm/CodeGen/LivePhysRegs.h"
    25: #include "llvm/CodeGen/MachineBasicBlock.h"
    26: #include "llvm/CodeGen/MachineDominators.h"
    27: #include "llvm/CodeGen/MachineFrameInfo.h"
    28: #include "llvm/CodeGen/MachineFunction.h"
    29: #include "llvm/CodeGen/MachineFunctionPass.h"
    30: #include "llvm/CodeGen/MachineInstr.h"
    31: #include "llvm/CodeGen/MachineInstrBuilder.h"
    32: #include "llvm/CodeGen/MachineMemOperand.h"
    33: #include "llvm/CodeGen/MachineModuleInfo.h"
    34: #include "llvm/CodeGen/MachineOperand.h"
    35: #include "llvm/CodeGen/MachinePostDominators.h"
    36: #include "llvm/CodeGen/MachineRegisterInfo.h"
    37: #include "llvm/CodeGen/PseudoSourceValue.h"
    38: #include "llvm/CodeGen/RegisterScavenging.h"
    39: #include "llvm/CodeGen/TargetRegisterInfo.h"
    40: #include "llvm/IR/Attributes.h"
    41: #include "llvm/IR/DebugLoc.h"
    42: #include "llvm/IR/Function.h"
    43: #include "llvm/MC/MCDwarf.h"
    44: #include "llvm/MC/MCRegisterInfo.h"
    45: #include "llvm/Pass.h"
    46: #include "llvm/Support/CodeGen.h"
    47: #include "llvm/Support/CommandLine.h"
    48: #include "llvm/Support/Compiler.h"
    49: #include "llvm/Support/Debug.h"
    50: #include "llvm/Support/ErrorHandling.h"
    51: #include "llvm/Support/MathExtras.h"
    52: #include "llvm/Support/raw_ostream.h"
    53: #include "llvm/Target/TargetMachine.h"
    54: #include "llvm/Target/TargetOptions.h"
    55: #include <algorithm>
    56: #include <cassert>
    57: #include <cstdint>
    58: #include <iterator>
    59: #include <limits>
    60: #include <map>
    61: #include <optional>
    62: #include <utility>
    63: #include <vector>
    64: 
    65: #define DEBUG_TYPE "hexagon-pei"
    66: 
    67: // Hexagon stack frame layout as defined by the ABI:
    68: //
    69: //                                                       Incoming arguments
    70: //                                                       passed via stack
    71: //                                                                      |
    72: //                                                                      |
    73: //        SP during function's                 FP during function's     |
    74: //    +-- runtime (top of stack)               runtime (bottom) --+     |
    75: //    |                                                           |     |
    76: // --++---------------------+------------------+-----------------++-+-------
    77: //   |  parameter area for  |  variable-size   |   fixed-size    |LR|  arg
    78: //   |   called functions   |  local objects   |  local objects  |FP|
    79: // --+----------------------+------------------+-----------------+--+-------
    80: //    <-    size known    -> <- size unknown -> <- size known  ->
    81: //
    82: // Low address                                                 High address
    83: //
    84: // <--- stack growth
    85: //
    86: //
    87: // - In any circumstances, the outgoing function arguments are always accessi-
    88: //   ble using the SP, and the incoming arguments are accessible using the FP.
    89: // - If the local objects are not aligned, they can always be accessed using
    90: //   the FP.
    91: // - If there are no variable-sized objects, the local objects can always be
    92: //   accessed using the SP, regardless whether they are aligned or not. (The
    93: //   alignment padding will be at the bottom of the stack (highest address),
    94: //   and so the offset with respect to the SP will be known at the compile-
    95: //   -time.)
    96: //
    97: // The only complication occurs if there are both, local aligned objects, and
    98: // dynamically allocated (variable-sized) objects. The alignment pad will be
    99: // placed between the FP and the local objects, thus preventing the use of the
   100: // FP to access the local objects. At the same time, the variable-sized objects
   101: // will be between the SP and the local objects, thus introducing an unknown
   102: // distance from the SP to the locals.
   103: //
   104: // To avoid this problem, a new register is created that holds the aligned
   105: // address of the bottom of the stack, referred in the sources as AP (aligned
   106: // pointer). The AP will be equal to "FP-p", where "p" is the smallest pad
   107: // that aligns AP to the required boundary (a maximum of the alignments of
   108: // all stack objects, fixed- and variable-sized). All local objects[1] will
   109: // then use AP as the base pointer.
   110: // [1] The exception is with "fixed" stack objects. "Fixed" stack objects get
   111: // their name from being allocated at fixed locations on the stack, relative
   112: // to the FP. In the presence of dynamic allocation and local alignment, such
   113: // objects can only be accessed through the FP.
   114: //
   115: // Illustration of the AP:
   116: //                                                                FP --+
   117: //                                                                     |
   118: // ---------------+---------------------+-----+-----------------------++-+--
   119: //   Rest of the  | Local stack objects | Pad |  Fixed stack objects  |LR|
   120: //   stack frame  | (aligned)           |     |  (CSR, spills, etc.)  |FP|
   121: // ---------------+---------------------+-----+-----------------+-----+--+--
   122: //                                      |<-- Multiple of the -->|
   123: //                                           stack alignment    +-- AP
   124: //
   125: // The AP is set up at the beginning of the function. Since it is not a dedi-
   126: // cated (reserved) register, it needs to be kept live throughout the function
   127: // to be available as the base register for local object accesses.
   128: // Normally, an address of a stack objects is obtained by a pseudo-instruction
   129: // PS_fi. To access local objects with the AP register present, a different
   130: // pseudo-instruction needs to be used: PS_fia. The PS_fia takes one extra
   131: // argument compared to PS_fi: the first input register is the AP register.
   132: // This keeps the register live between its definition and its uses.
   133: 
   134: // The AP register is originally set up using pseudo-instruction PS_aligna:
   135: //   AP = PS_aligna A
   136: // where
   137: //   A  - required stack alignment
   138: // The alignment value must be the maximum of all alignments required by
   139: // any stack object.
   140: 
   141: // The dynamic allocation uses a pseudo-instruction PS_alloca:
   142: //   Rd = PS_alloca Rs, A
   143: // where
   144: //   Rd - address of the allocated space
   145: //   Rs - minimum size (the actual allocated can be larger to accommodate
   146: //        alignment)
   147: //   A  - required alignment
   148: 
   149: using namespace llvm;
   150: 
   151: static cl::opt<bool> DisableDeallocRet("disable-hexagon-dealloc-ret",
   152:     cl::Hidden, cl::desc("Disable Dealloc Return for Hexagon target"));
   153: 
   154: static cl::opt<unsigned>
   155:     NumberScavengerSlots("number-scavenger-slots", cl::Hidden,
   156:                          cl::desc("Set the number of scavenger slots"),
   157:                          cl::init(2));
   158: 
   159: static cl::opt<int>
   160:     SpillFuncThreshold("spill-func-threshold", cl::Hidden,
   161:                        cl::desc("Specify O2(not Os) spill func threshold"),
   162:                        cl::init(6));
   163: 
   164: static cl::opt<int>
   165:     SpillFuncThresholdOs("spill-func-threshold-Os", cl::Hidden,
   166:                          cl::desc("Specify Os spill func threshold"),
   167:                          cl::init(1));
   168: 
   169: static cl::opt<bool> EnableStackOVFSanitizer(
   170:     "enable-stackovf-sanitizer", cl::Hidden,
   171:     cl::desc("Enable runtime checks for stack overflow."), cl::init(false));
   172: 
   173: static cl::opt<bool>
   174:     EnableShrinkWrapping("hexagon-shrink-frame", cl::init(true), cl::Hidden,
   175:                          cl::desc("Enable stack frame shrink wrapping"));
   176: 
   177: static cl::opt<unsigned>
   178:     ShrinkLimit("shrink-frame-limit",
   179:                 cl::init(std::numeric_limits<unsigned>::max()), cl::Hidden,
   180:                 cl::desc("Max count of stack frame shrink-wraps"));
   181: 
   182: static cl::opt<bool>
   183:     EnableSaveRestoreLong("enable-save-restore-long", cl::Hidden,
   184:                           cl::desc("Enable long calls for save-restore stubs."),
   185:                           cl::init(false));
   186: 
   187: static cl::opt<bool> EliminateFramePointer("hexagon-fp-elim", cl::init(true),
   188:     cl::Hidden, cl::desc("Refrain from using FP whenever possible"));
   189: 
   190: static cl::opt<bool> OptimizeSpillSlots("hexagon-opt-spill", cl::Hidden,
   191:     cl::init(true), cl::desc("Optimize spill slots"));
   192: 
   193: #ifndef NDEBUG
   194: static cl::opt<unsigned> SpillOptMax("spill-opt-max", cl::Hidden,
   195:     cl::init(std::numeric_limits<unsigned>::max()));
   196: static unsigned SpillOptCount = 0;
   197: #endif
   198: 
   199: namespace {
   200: 
```
- EN: It imports headers such as HexagonFrameLowering.h, HexagonBlockRanges.h, HexagonInstrInfo.h, HexagonMachineFunctionInfo.h, ... (54 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as DisableDeallocRet, NumberScavengerSlots, SpillFuncThreshold, SpillFuncThresholdOs, ... (11 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonFrameLowering.h, HexagonBlockRanges.h, HexagonInstrInfo.h, HexagonMachineFunctionInfo.h, ... (54 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 DisableDeallocRet, NumberScavengerSlots, SpillFuncThreshold, SpillFuncThresholdOs, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 201-400 / 第 201-400 行

```cpp
   201:   class HexagonCallFrameInformation : public MachineFunctionPass {
   202:   public:
   203:     static char ID;
   204: 
   205:     HexagonCallFrameInformation() : MachineFunctionPass(ID) {}
   206: 
   207:     bool runOnMachineFunction(MachineFunction &MF) override;
   208: 
   209:     MachineFunctionProperties getRequiredProperties() const override {
   210:       return MachineFunctionProperties().setNoVRegs();
   211:     }
   212:   };
   213: 
   214:   char HexagonCallFrameInformation::ID = 0;
   215: 
   216: } // end anonymous namespace
   217: 
   218: bool HexagonCallFrameInformation::runOnMachineFunction(MachineFunction &MF) {
   219:   auto &HFI = *MF.getSubtarget<HexagonSubtarget>().getFrameLowering();
   220:   bool NeedCFI = MF.needsFrameMoves();
   221: 
   222:   if (!NeedCFI)
   223:     return false;
   224:   HFI.insertCFIInstructions(MF);
   225:   return true;
   226: }
   227: 
   228: INITIALIZE_PASS(HexagonCallFrameInformation, "hexagon-cfi",
   229:                 "Hexagon call frame information", false, false)
   230: 
   231: FunctionPass *llvm::createHexagonCallFrameInformation() {
   232:   return new HexagonCallFrameInformation();
   233: }
   234: 
   235: /// Map a register pair Reg to the subregister that has the greater "number",
   236: /// i.e. D3 (aka R7:6) will be mapped to R7, etc.
   237: static Register getMax32BitSubRegister(Register Reg,
   238:                                        const TargetRegisterInfo &TRI,
   239:                                        bool hireg = true) {
   240:     if (Reg < Hexagon::D0 || Reg > Hexagon::D15)
   241:       return Reg;
   242: 
   243:     Register RegNo = 0;
   244:     for (MCPhysReg SubReg : TRI.subregs(Reg)) {
   245:       if (hireg) {
   246:         if (SubReg > RegNo)
   247:           RegNo = SubReg;
   248:       } else {
   249:         if (!RegNo || SubReg < RegNo)
   250:           RegNo = SubReg;
   251:       }
   252:     }
   253:     return RegNo;
   254: }
   255: 
   256: /// Returns the callee saved register with the largest id in the vector.
   257: static Register getMaxCalleeSavedReg(ArrayRef<CalleeSavedInfo> CSI,
   258:                                      const TargetRegisterInfo &TRI) {
   259:   static_assert(Hexagon::R1 > 0,
   260:                 "Assume physical registers are encoded as positive integers");
   261:   if (CSI.empty())
   262:     return 0;
   263: 
   264:   Register Max = getMax32BitSubRegister(CSI[0].getReg(), TRI);
   265:   for (unsigned I = 1, E = CSI.size(); I < E; ++I) {
   266:     Register Reg = getMax32BitSubRegister(CSI[I].getReg(), TRI);
   267:     if (Reg > Max)
   268:       Max = Reg;
   269:   }
   270:   return Max;
   271: }
   272: 
   273: /// Checks if the basic block contains any instruction that needs a stack
   274: /// frame to be already in place.
   275: static bool needsStackFrame(const MachineBasicBlock &MBB, const BitVector &CSR,
   276:                             const HexagonRegisterInfo &HRI) {
   277:     for (const MachineInstr &MI : MBB) {
   278:       if (MI.isCall())
   279:         return true;
   280:       unsigned Opc = MI.getOpcode();
   281:       switch (Opc) {
   282:         case Hexagon::PS_alloca:
   283:         case Hexagon::PS_aligna:
   284:           return true;
   285:         default:
   286:           break;
   287:       }
   288:       // Check individual operands.
   289:       for (const MachineOperand &MO : MI.operands()) {
   290:         // While the presence of a frame index does not prove that a stack
   291:         // frame will be required, all frame indexes should be within alloc-
   292:         // frame/deallocframe. Otherwise, the code that translates a frame
   293:         // index into an offset would have to be aware of the placement of
   294:         // the frame creation/destruction instructions.
   295:         if (MO.isFI())
   296:           return true;
   297:         if (MO.isReg()) {
   298:           Register R = MO.getReg();
   299:           // Debug instructions may refer to $noreg.
   300:           if (!R)
   301:             continue;
   302:           // Virtual registers will need scavenging, which then may require
   303:           // a stack slot.
   304:           if (R.isVirtual())
   305:             return true;
   306:           for (MCPhysReg S : HRI.subregs_inclusive(R))
   307:             if (CSR[S])
   308:               return true;
   309:           continue;
   310:         }
   311:         if (MO.isRegMask()) {
   312:           // A regmask would normally have all callee-saved registers marked
   313:           // as preserved, so this check would not be needed, but in case of
   314:           // ever having other regmasks (for other calling conventions),
   315:           // make sure they would be processed correctly.
   316:           const uint32_t *BM = MO.getRegMask();
   317:           for (int x = CSR.find_first(); x >= 0; x = CSR.find_next(x)) {
   318:             unsigned R = x;
   319:             // If this regmask does not preserve a CSR, a frame will be needed.
   320:             if (!(BM[R/32] & (1u << (R%32))))
   321:               return true;
   322:           }
   323:         }
   324:       }
   325:     }
   326:     return false;
   327: }
   328: 
   329:   /// Returns true if MBB has a machine instructions that indicates a tail call
   330:   /// in the block.
   331: static bool hasTailCall(const MachineBasicBlock &MBB) {
   332:     MachineBasicBlock::const_iterator I = MBB.getLastNonDebugInstr();
   333:     if (I == MBB.end())
   334:       return false;
   335:     unsigned RetOpc = I->getOpcode();
   336:     return RetOpc == Hexagon::PS_tailcall_i || RetOpc == Hexagon::PS_tailcall_r;
   337: }
   338: 
   339: /// Returns true if MBB contains an instruction that returns.
   340: static bool hasReturn(const MachineBasicBlock &MBB) {
   341:     for (const MachineInstr &MI : MBB.terminators())
   342:       if (MI.isReturn())
   343:         return true;
   344:     return false;
   345: }
   346: 
   347: /// Returns the "return" instruction from this block, or nullptr if there
   348: /// isn't any.
   349: static MachineInstr *getReturn(MachineBasicBlock &MBB) {
   350:     for (auto &I : MBB)
   351:       if (I.isReturn())
   352:         return &I;
   353:     return nullptr;
   354: }
   355: 
   356: static bool isRestoreCall(unsigned Opc) {
   357:     switch (Opc) {
   358:       case Hexagon::RESTORE_DEALLOC_RET_JMP_V4:
   359:       case Hexagon::RESTORE_DEALLOC_RET_JMP_V4_PIC:
   360:       case Hexagon::RESTORE_DEALLOC_RET_JMP_V4_EXT:
   361:       case Hexagon::RESTORE_DEALLOC_RET_JMP_V4_EXT_PIC:
   362:       case Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT:
   363:       case Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT_PIC:
   364:       case Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4:
   365:       case Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_PIC:
   366:         return true;
   367:     }
   368:     return false;
   369: }
   370: 
   371: static inline bool isOptNone(const MachineFunction &MF) {
   372:     return MF.getFunction().hasOptNone() ||
   373:            MF.getTarget().getOptLevel() == CodeGenOptLevel::None;
   374: }
   375: 
   376: static inline bool isOptSize(const MachineFunction &MF) {
   377:     const Function &F = MF.getFunction();
   378:     return F.hasOptSize() && !F.hasMinSize();
   379: }
   380: 
   381: static inline bool isMinSize(const MachineFunction &MF) {
   382:     return MF.getFunction().hasMinSize();
   383: }
   384: 
   385: /// Implements shrink-wrapping of the stack frame. By default, stack frame
   386: /// is created in the function entry block, and is cleaned up in every block
   387: /// that returns. This function finds alternate blocks: one for the frame
   388: /// setup (prolog) and one for the cleanup (epilog).
   389: void HexagonFrameLowering::findShrunkPrologEpilog(MachineFunction &MF,
   390:       MachineBasicBlock *&PrologB, MachineBasicBlock *&EpilogB) const {
   391:   static unsigned ShrinkCounter = 0;
   392: 
   393:   if (MF.getSubtarget<HexagonSubtarget>().isEnvironmentMusl() &&
   394:       MF.getFunction().isVarArg())
   395:     return;
   396:   if (ShrinkLimit.getPosition()) {
   397:     if (ShrinkCounter >= ShrinkLimit)
   398:       return;
   399:     ShrinkCounter++;
   400:   }
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonCallFrameInformation, which carry the state or API of this component. It defines declarative TableGen records like HexagonCallFrameInformation; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonCallFrameInformation, runOnMachineFunction, getRequiredProperties, MachineFunctionProperties, ... (29 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonCallFrameInformation 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonCallFrameInformation 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonCallFrameInformation, runOnMachineFunction, getRequiredProperties, MachineFunctionProperties, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 401-600 / 第 401-600 行

```cpp
   401: 
   402:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   403: 
   404:   MachineDominatorTree MDT;
   405:   MDT.recalculate(MF);
   406:   MachinePostDominatorTree MPT;
   407:   MPT.recalculate(MF);
   408: 
   409:   using UnsignedMap = DenseMap<unsigned, unsigned>;
   410:   using RPOTType = ReversePostOrderTraversal<const MachineFunction *>;
   411: 
   412:   UnsignedMap RPO;
   413:   RPOTType RPOT(&MF);
   414:   unsigned RPON = 0;
   415:   for (auto &I : RPOT)
   416:     RPO[I->getNumber()] = RPON++;
   417: 
   418:   // Don't process functions that have loops, at least for now. Placement
   419:   // of prolog and epilog must take loop structure into account. For simpli-
   420:   // city don't do it right now.
   421:   for (auto &I : MF) {
   422:     unsigned BN = RPO[I.getNumber()];
   423:     for (MachineBasicBlock *Succ : I.successors())
   424:       // If found a back-edge, return.
   425:       if (RPO[Succ->getNumber()] <= BN)
   426:         return;
   427:   }
   428: 
   429:   // Collect the set of blocks that need a stack frame to execute. Scan
   430:   // each block for uses/defs of callee-saved registers, calls, etc.
   431:   SmallVector<MachineBasicBlock*,16> SFBlocks;
   432:   BitVector CSR(Hexagon::NUM_TARGET_REGS);
   433:   for (const MCPhysReg *P = HRI.getCalleeSavedRegs(&MF); *P; ++P)
   434:     for (MCPhysReg S : HRI.subregs_inclusive(*P))
   435:       CSR[S] = true;
   436: 
   437:   for (auto &I : MF)
   438:     if (needsStackFrame(I, CSR, HRI))
   439:       SFBlocks.push_back(&I);
   440: 
   441:   LLVM_DEBUG({
   442:     dbgs() << "Blocks needing SF: {";
   443:     for (auto &B : SFBlocks)
   444:       dbgs() << " " << printMBBReference(*B);
   445:     dbgs() << " }\n";
   446:   });
   447:   // No frame needed?
   448:   if (SFBlocks.empty())
   449:     return;
   450: 
   451:   // Pick a common dominator and a common post-dominator.
   452:   MachineBasicBlock *DomB = SFBlocks[0];
   453:   for (unsigned i = 1, n = SFBlocks.size(); i < n; ++i) {
   454:     DomB = MDT.findNearestCommonDominator(DomB, SFBlocks[i]);
   455:     if (!DomB)
   456:       break;
   457:   }
   458:   MachineBasicBlock *PDomB = SFBlocks[0];
   459:   for (unsigned i = 1, n = SFBlocks.size(); i < n; ++i) {
   460:     PDomB = MPT.findNearestCommonDominator(PDomB, SFBlocks[i]);
   461:     if (!PDomB)
   462:       break;
   463:   }
   464:   LLVM_DEBUG({
   465:     dbgs() << "Computed dom block: ";
   466:     if (DomB)
   467:       dbgs() << printMBBReference(*DomB);
   468:     else
   469:       dbgs() << "<null>";
   470:     dbgs() << ", computed pdom block: ";
   471:     if (PDomB)
   472:       dbgs() << printMBBReference(*PDomB);
   473:     else
   474:       dbgs() << "<null>";
   475:     dbgs() << "\n";
   476:   });
   477:   if (!DomB || !PDomB)
   478:     return;
   479: 
   480:   // Make sure that DomB dominates PDomB and PDomB post-dominates DomB.
   481:   if (!MDT.dominates(DomB, PDomB)) {
   482:     LLVM_DEBUG(dbgs() << "Dom block does not dominate pdom block\n");
   483:     return;
   484:   }
   485:   if (!MPT.dominates(PDomB, DomB)) {
   486:     LLVM_DEBUG(dbgs() << "PDom block does not post-dominate dom block\n");
   487:     return;
   488:   }
   489: 
   490:   // Finally, everything seems right.
   491:   PrologB = DomB;
   492:   EpilogB = PDomB;
   493: }
   494: 
   495: /// Perform most of the PEI work here:
   496: /// - saving/restoring of the callee-saved registers,
   497: /// - stack frame creation and destruction.
   498: /// Normally, this work is distributed among various functions, but doing it
   499: /// in one place allows shrink-wrapping of the stack frame.
   500: void HexagonFrameLowering::emitPrologue(MachineFunction &MF,
   501:                                         MachineBasicBlock &MBB) const {
   502:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   503: 
   504:   MachineFrameInfo &MFI = MF.getFrameInfo();
   505:   const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
   506: 
   507:   MachineBasicBlock *PrologB = &MF.front(), *EpilogB = nullptr;
   508:   if (EnableShrinkWrapping)
   509:     findShrunkPrologEpilog(MF, PrologB, EpilogB);
   510: 
   511:   bool PrologueStubs = false;
   512:   insertCSRSpillsInBlock(*PrologB, CSI, HRI, PrologueStubs);
   513:   insertPrologueInBlock(*PrologB, PrologueStubs);
   514:   updateEntryPaths(MF, *PrologB);
   515: 
   516:   if (EpilogB) {
   517:     insertCSRRestoresInBlock(*EpilogB, CSI, HRI);
   518:     insertEpilogueInBlock(*EpilogB);
   519:   } else {
   520:     for (auto &B : MF)
   521:       if (B.isReturnBlock())
   522:         insertCSRRestoresInBlock(B, CSI, HRI);
   523: 
   524:     for (auto &B : MF)
   525:       if (B.isReturnBlock())
   526:         insertEpilogueInBlock(B);
   527: 
   528:     for (auto &B : MF) {
   529:       if (B.empty())
   530:         continue;
   531:       MachineInstr *RetI = getReturn(B);
   532:       if (!RetI || isRestoreCall(RetI->getOpcode()))
   533:         continue;
   534:       for (auto &R : CSI)
   535:         RetI->addOperand(MachineOperand::CreateReg(R.getReg(), false, true));
   536:     }
   537:   }
   538: 
   539:   if (EpilogB) {
   540:     // If there is an epilog block, it may not have a return instruction.
   541:     // In such case, we need to add the callee-saved registers as live-ins
   542:     // in all blocks on all paths from the epilog to any return block.
   543:     unsigned MaxBN = MF.getNumBlockIDs();
   544:     BitVector DoneT(MaxBN+1), DoneF(MaxBN+1), Path(MaxBN+1);
   545:     updateExitPaths(*EpilogB, *EpilogB, DoneT, DoneF, Path);
   546:   }
   547: }
   548: 
   549: /// Returns true if the target can safely skip saving callee-saved registers
   550: /// for noreturn nounwind functions.
   551: bool HexagonFrameLowering::enableCalleeSaveSkip(
   552:     const MachineFunction &MF) const {
   553:   const auto &F = MF.getFunction();
   554:   assert(F.hasFnAttribute(Attribute::NoReturn) &&
   555:          F.getFunction().hasFnAttribute(Attribute::NoUnwind) &&
   556:          !F.getFunction().hasFnAttribute(Attribute::UWTable));
   557:   (void)F;
   558: 
   559:   // No need to save callee saved registers if the function does not return.
   560:   return MF.getSubtarget<HexagonSubtarget>().noreturnStackElim();
   561: }
   562: 
   563: // Helper function used to determine when to eliminate the stack frame for
   564: // functions marked as noreturn and when the noreturn-stack-elim options are
   565: // specified. When both these conditions are true, then a FP may not be needed
   566: // if the function makes a call. It is very similar to enableCalleeSaveSkip,
   567: // but it used to check if the allocframe can be eliminated as well.
   568: static bool enableAllocFrameElim(const MachineFunction &MF) {
   569:   const auto &F = MF.getFunction();
   570:   const auto &MFI = MF.getFrameInfo();
   571:   const auto &HST = MF.getSubtarget<HexagonSubtarget>();
   572:   assert(!MFI.hasVarSizedObjects() &&
   573:          !HST.getRegisterInfo()->hasStackRealignment(MF));
   574:   return F.hasFnAttribute(Attribute::NoReturn) &&
   575:     F.hasFnAttribute(Attribute::NoUnwind) &&
   576:     !F.hasFnAttribute(Attribute::UWTable) && HST.noreturnStackElim() &&
   577:     MFI.getStackSize() == 0;
   578: }
   579: 
   580: void HexagonFrameLowering::insertPrologueInBlock(MachineBasicBlock &MBB,
   581:       bool PrologueStubs) const {
   582:   MachineFunction &MF = *MBB.getParent();
   583:   MachineFrameInfo &MFI = MF.getFrameInfo();
   584:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
   585:   auto &HII = *HST.getInstrInfo();
   586:   auto &HRI = *HST.getRegisterInfo();
   587: 
   588:   Align MaxAlign = std::max(MFI.getMaxAlign(), getStackAlign());
   589: 
   590:   // Calculate the total stack frame size.
   591:   // Get the number of bytes to allocate from the FrameInfo.
   592:   unsigned FrameSize = MFI.getStackSize();
   593:   // Round up the max call frame size to the max alignment on the stack.
   594:   unsigned MaxCFA = alignTo(MFI.getMaxCallFrameSize(), MaxAlign);
   595:   MFI.setMaxCallFrameSize(MaxCFA);
   596: 
   597:   FrameSize = MaxCFA + alignTo(FrameSize, MaxAlign);
   598:   MFI.setStackSize(FrameSize);
   599: 
   600:   bool AlignStack = (MaxAlign > getStackAlign());
```
- EN: It declares or implements routines such as getSubtarget<HexagonSubtarget>, recalculate, RPOT, CSR, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonFrameLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget<HexagonSubtarget>, recalculate, RPOT, CSR, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonFrameLowering，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```cpp
   601: 
   602:   // Get the number of bytes to allocate from the FrameInfo.
   603:   unsigned NumBytes = MFI.getStackSize();
   604:   Register SP = HRI.getStackRegister();
   605:   unsigned MaxCF = MFI.getMaxCallFrameSize();
   606:   MachineBasicBlock::iterator InsertPt = MBB.begin();
   607: 
   608:   SmallVector<MachineInstr *, 4> AdjustRegs;
   609:   for (auto &MBB : MF)
   610:     for (auto &MI : MBB)
   611:       if (MI.getOpcode() == Hexagon::PS_alloca)
   612:         AdjustRegs.push_back(&MI);
   613: 
   614:   for (auto *MI : AdjustRegs) {
   615:     assert((MI->getOpcode() == Hexagon::PS_alloca) && "Expected alloca");
   616:     expandAlloca(MI, HII, SP, MaxCF);
   617:     MI->eraseFromParent();
   618:   }
   619: 
   620:   DebugLoc dl = MBB.findDebugLoc(InsertPt);
   621: 
   622:   if (MF.getFunction().isVarArg() &&
   623:       MF.getSubtarget<HexagonSubtarget>().isEnvironmentMusl()) {
   624:     // Calculate the size of register saved area.
   625:     int NumVarArgRegs = 6 - FirstVarArgSavedReg;
   626:     int RegisterSavedAreaSizePlusPadding = (NumVarArgRegs % 2 == 0)
   627:                                               ? NumVarArgRegs * 4
   628:                                               : NumVarArgRegs * 4 + 4;
   629:     if (RegisterSavedAreaSizePlusPadding > 0) {
   630:       // Decrement the stack pointer by size of register saved area plus
   631:       // padding if any.
   632:       BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::A2_addi), SP)
   633:         .addReg(SP)
   634:         .addImm(-RegisterSavedAreaSizePlusPadding)
   635:         .setMIFlag(MachineInstr::FrameSetup);
   636: 
   637:       int NumBytes = 0;
   638:       // Copy all the named arguments below register saved area.
   639:       auto &HMFI = *MF.getInfo<HexagonMachineFunctionInfo>();
   640:       for (int i = HMFI.getFirstNamedArgFrameIndex(),
   641:                e = HMFI.getLastNamedArgFrameIndex(); i >= e; --i) {
   642:         uint64_t ObjSize = MFI.getObjectSize(i);
   643:         Align ObjAlign = MFI.getObjectAlign(i);
   644: 
   645:         // Determine the kind of load/store that should be used.
   646:         unsigned LDOpc, STOpc;
   647:         uint64_t OpcodeChecker = ObjAlign.value();
   648: 
   649:         // Handle cases where alignment of an object is > its size.
   650:         if (ObjAlign > ObjSize) {
   651:           if (ObjSize <= 1)
   652:             OpcodeChecker = 1;
   653:           else if (ObjSize <= 2)
   654:             OpcodeChecker = 2;
   655:           else if (ObjSize <= 4)
   656:             OpcodeChecker = 4;
   657:           else if (ObjSize > 4)
   658:             OpcodeChecker = 8;
   659:         }
   660: 
   661:         switch (OpcodeChecker) {
   662:           case 1:
   663:             LDOpc = Hexagon::L2_loadrb_io;
   664:             STOpc = Hexagon::S2_storerb_io;
   665:             break;
   666:           case 2:
   667:             LDOpc = Hexagon::L2_loadrh_io;
   668:             STOpc = Hexagon::S2_storerh_io;
   669:             break;
   670:           case 4:
   671:             LDOpc = Hexagon::L2_loadri_io;
   672:             STOpc = Hexagon::S2_storeri_io;
   673:             break;
   674:           case 8:
   675:           default:
   676:             LDOpc = Hexagon::L2_loadrd_io;
   677:             STOpc = Hexagon::S2_storerd_io;
   678:             break;
   679:         }
   680: 
   681:         Register RegUsed = LDOpc == Hexagon::L2_loadrd_io ? Hexagon::D3
   682:                                                           : Hexagon::R6;
   683:         int LoadStoreCount = ObjSize / OpcodeChecker;
   684: 
   685:         if (ObjSize % OpcodeChecker)
   686:           ++LoadStoreCount;
   687: 
   688:         // Get the start location of the load. NumBytes is basically the
   689:         // offset from the stack pointer of previous function, which would be
   690:         // the caller in this case, as this function has variable argument
   691:         // list.
   692:         if (NumBytes != 0)
   693:           NumBytes = alignTo(NumBytes, ObjAlign);
   694: 
   695:         int Count = 0;
   696:         while (Count < LoadStoreCount) {
   697:           // Load the value of the named argument on stack.
   698:           BuildMI(MBB, InsertPt, dl, HII.get(LDOpc), RegUsed)
   699:               .addReg(SP)
   700:               .addImm(RegisterSavedAreaSizePlusPadding +
   701:                       ObjAlign.value() * Count + NumBytes)
   702:               .setMIFlag(MachineInstr::FrameSetup);
   703: 
   704:           // Store it below the register saved area plus padding.
   705:           BuildMI(MBB, InsertPt, dl, HII.get(STOpc))
   706:               .addReg(SP)
   707:               .addImm(ObjAlign.value() * Count + NumBytes)
   708:               .addReg(RegUsed)
   709:               .setMIFlag(MachineInstr::FrameSetup);
   710: 
   711:           Count++;
   712:         }
   713:         NumBytes += MFI.getObjectSize(i);
   714:       }
   715: 
   716:       // Make NumBytes 8 byte aligned
   717:       NumBytes = alignTo(NumBytes, 8);
   718: 
   719:       // If the number of registers having variable arguments is odd,
   720:       // leave 4 bytes of padding to get to the location where first
   721:       // variable argument which was passed through register was copied.
   722:       NumBytes = (NumVarArgRegs % 2 == 0) ? NumBytes : NumBytes + 4;
   723: 
   724:       for (int j = FirstVarArgSavedReg, i = 0; j < 6; ++j, ++i) {
   725:         BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::S2_storeri_io))
   726:           .addReg(SP)
   727:           .addImm(NumBytes + 4 * i)
   728:           .addReg(Hexagon::R0 + j)
   729:           .setMIFlag(MachineInstr::FrameSetup);
   730:       }
   731:     }
   732:   }
   733: 
   734:   if (hasFP(MF)) {
   735:     insertAllocframe(MBB, InsertPt, NumBytes);
   736:     if (AlignStack) {
   737:       BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::A2_andir), SP)
   738:           .addReg(SP)
   739:           .addImm(-int64_t(MaxAlign.value()));
   740:     }
   741:     // If the stack-checking is enabled, and we spilled the callee-saved
   742:     // registers inline (i.e. did not use a spill function), then call
   743:     // the stack checker directly.
   744:     if (EnableStackOVFSanitizer && !PrologueStubs)
   745:       BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::PS_call_stk))
   746:              .addExternalSymbol("__runtime_stack_check");
   747:   } else if (NumBytes > 0) {
   748:     assert(alignTo(NumBytes, 8) == NumBytes);
   749:     BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::A2_addi), SP)
   750:       .addReg(SP)
   751:       .addImm(-int(NumBytes));
   752:   }
   753: }
   754: 
   755: void HexagonFrameLowering::insertEpilogueInBlock(MachineBasicBlock &MBB) const {
   756:   MachineFunction &MF = *MBB.getParent();
   757:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
   758:   auto &HII = *HST.getInstrInfo();
   759:   auto &HRI = *HST.getRegisterInfo();
   760:   Register SP = HRI.getStackRegister();
   761: 
   762:   MachineBasicBlock::iterator InsertPt = MBB.getFirstTerminator();
   763:   DebugLoc dl = MBB.findDebugLoc(InsertPt);
   764: 
   765:   if (!hasFP(MF)) {
   766:     MachineFrameInfo &MFI = MF.getFrameInfo();
   767:     unsigned NumBytes = MFI.getStackSize();
   768:     if (MF.getFunction().isVarArg() &&
   769:         MF.getSubtarget<HexagonSubtarget>().isEnvironmentMusl()) {
   770:       // On Hexagon Linux, deallocate the stack for the register saved area.
   771:       int NumVarArgRegs = 6 - FirstVarArgSavedReg;
   772:       int RegisterSavedAreaSizePlusPadding = (NumVarArgRegs % 2 == 0) ?
   773:         (NumVarArgRegs * 4) : (NumVarArgRegs * 4 + 4);
   774:       NumBytes += RegisterSavedAreaSizePlusPadding;
   775:     }
   776:     if (NumBytes) {
   777:       BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::A2_addi), SP)
   778:         .addReg(SP)
   779:         .addImm(NumBytes);
   780:     }
   781:     return;
   782:   }
   783: 
   784:   MachineInstr *RetI = getReturn(MBB);
   785:   unsigned RetOpc = RetI ? RetI->getOpcode() : 0;
   786: 
   787:   // Handle EH_RETURN.
   788:   if (RetOpc == Hexagon::EH_RETURN_JMPR) {
   789:     BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::L2_deallocframe))
   790:         .addDef(Hexagon::D15)
   791:         .addReg(Hexagon::R30);
   792:     BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::A2_add), SP)
   793:         .addReg(SP)
   794:         .addReg(Hexagon::R28);
   795:     return;
   796:   }
   797: 
   798:   // Check for RESTORE_DEALLOC_RET* tail call. Don't emit an extra dealloc-
   799:   // frame instruction if we encounter it.
   800:   if (RetOpc == Hexagon::RESTORE_DEALLOC_RET_JMP_V4 ||
```
- EN: It declares or implements routines such as getStackSize, getStackRegister, getMaxCallFrameSize, begin, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonMachineFunctionInfo, HexagonFrameLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getStackSize, getStackRegister, getMaxCallFrameSize, begin, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonMachineFunctionInfo, HexagonFrameLowering，说明了它与同级后端组件的连接关系。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:       RetOpc == Hexagon::RESTORE_DEALLOC_RET_JMP_V4_PIC ||
   802:       RetOpc == Hexagon::RESTORE_DEALLOC_RET_JMP_V4_EXT ||
   803:       RetOpc == Hexagon::RESTORE_DEALLOC_RET_JMP_V4_EXT_PIC) {
   804:     MachineBasicBlock::iterator It = RetI;
   805:     ++It;
   806:     // Delete all instructions after the RESTORE (except labels).
   807:     while (It != MBB.end()) {
   808:       if (!It->isLabel())
   809:         It = MBB.erase(It);
   810:       else
   811:         ++It;
   812:     }
   813:     return;
   814:   }
   815: 
   816:   // It is possible that the restoring code is a call to a library function.
   817:   // All of the restore* functions include "deallocframe", so we need to make
   818:   // sure that we don't add an extra one.
   819:   bool NeedsDeallocframe = true;
   820:   if (!MBB.empty() && InsertPt != MBB.begin()) {
   821:     MachineBasicBlock::iterator PrevIt = std::prev(InsertPt);
   822:     unsigned COpc = PrevIt->getOpcode();
   823:     if (COpc == Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4 ||
   824:         COpc == Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_PIC ||
   825:         COpc == Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT ||
   826:         COpc == Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT_PIC ||
   827:         COpc == Hexagon::PS_call_nr || COpc == Hexagon::PS_callr_nr)
   828:       NeedsDeallocframe = false;
   829:   }
   830: 
   831:   if (!MF.getSubtarget<HexagonSubtarget>().isEnvironmentMusl() ||
   832:       !MF.getFunction().isVarArg()) {
   833:     if (!NeedsDeallocframe)
   834:       return;
   835:     // If the returning instruction is PS_jmpret, replace it with
   836:     // dealloc_return, otherwise just add deallocframe. The function
   837:     // could be returning via a tail call.
   838:     if (RetOpc != Hexagon::PS_jmpret || DisableDeallocRet) {
   839:       BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::L2_deallocframe))
   840:       .addDef(Hexagon::D15)
   841:       .addReg(Hexagon::R30);
   842:       return;
   843:     }
   844:     unsigned NewOpc = Hexagon::L4_return;
   845:     MachineInstr *NewI = BuildMI(MBB, RetI, dl, HII.get(NewOpc))
   846:       .addDef(Hexagon::D15)
   847:       .addReg(Hexagon::R30);
   848:     // Transfer the function live-out registers.
   849:     NewI->copyImplicitOps(MF, *RetI);
   850:     MBB.erase(RetI);
   851:   } else {
   852:     // L2_deallocframe instruction after it.
   853:     // Calculate the size of register saved area.
   854:     int NumVarArgRegs = 6 - FirstVarArgSavedReg;
   855:     int RegisterSavedAreaSizePlusPadding = (NumVarArgRegs % 2 == 0) ?
   856:       (NumVarArgRegs * 4) : (NumVarArgRegs * 4 + 4);
   857: 
   858:     MachineBasicBlock::iterator Term = MBB.getFirstTerminator();
   859:     MachineBasicBlock::iterator I = (Term == MBB.begin()) ? MBB.end()
   860:                                                           : std::prev(Term);
   861:     if (I == MBB.end() ||
   862:        (I->getOpcode() != Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT &&
   863:         I->getOpcode() != Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT_PIC &&
   864:         I->getOpcode() != Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4 &&
   865:         I->getOpcode() != Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_PIC))
   866:       BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::L2_deallocframe))
   867:         .addDef(Hexagon::D15)
   868:         .addReg(Hexagon::R30);
   869:     if (RegisterSavedAreaSizePlusPadding != 0)
   870:       BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::A2_addi), SP)
   871:         .addReg(SP)
   872:         .addImm(RegisterSavedAreaSizePlusPadding);
   873:   }
   874: }
   875: 
   876: void HexagonFrameLowering::insertAllocframe(MachineBasicBlock &MBB,
   877:       MachineBasicBlock::iterator InsertPt, unsigned NumBytes) const {
   878:   MachineFunction &MF = *MBB.getParent();
   879:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
   880:   auto &HII = *HST.getInstrInfo();
   881:   auto &HRI = *HST.getRegisterInfo();
   882: 
   883:   // Check for overflow.
   884:   // Hexagon_TODO: Ugh! hardcoding. Is there an API that can be used?
   885:   const unsigned int ALLOCFRAME_MAX = 16384;
   886: 
   887:   // Create a dummy memory operand to avoid allocframe from being treated as
   888:   // a volatile memory reference.
   889:   auto *MMO = MF.getMachineMemOperand(MachinePointerInfo::getStack(MF, 0),
   890:                                       MachineMemOperand::MOStore, 4, Align(4));
   891: 
   892:   DebugLoc dl = MBB.findDebugLoc(InsertPt);
   893:   Register SP = HRI.getStackRegister();
   894: 
   895:   if (NumBytes >= ALLOCFRAME_MAX) {
   896:     // Emit allocframe(#0).
   897:     BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::S2_allocframe))
   898:         .addDef(SP)
   899:         .addReg(SP)
   900:         .addImm(0)
   901:         .addMemOperand(MMO)
   902:         .setMIFlag(MachineInstr::FrameSetup);
   903: 
   904:     // Subtract the size from the stack pointer.
   905:     Register SP = HRI.getStackRegister();
   906:     BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::A2_addi), SP)
   907:         .addReg(SP)
   908:         .addImm(-int(NumBytes))
   909:         .setMIFlag(MachineInstr::FrameSetup);
   910:   } else {
   911:     BuildMI(MBB, InsertPt, dl, HII.get(Hexagon::S2_allocframe))
   912:         .addDef(SP)
   913:         .addReg(SP)
   914:         .addImm(NumBytes)
   915:         .addMemOperand(MMO)
   916:         .setMIFlag(MachineInstr::FrameSetup);
   917:   }
   918: }
   919: 
   920: void HexagonFrameLowering::updateEntryPaths(MachineFunction &MF,
   921:       MachineBasicBlock &SaveB) const {
   922:   SetVector<unsigned> Worklist;
   923: 
   924:   MachineBasicBlock &EntryB = MF.front();
   925:   Worklist.insert(EntryB.getNumber());
   926: 
   927:   unsigned SaveN = SaveB.getNumber();
   928:   auto &CSI = MF.getFrameInfo().getCalleeSavedInfo();
   929: 
   930:   for (unsigned i = 0; i < Worklist.size(); ++i) {
   931:     unsigned BN = Worklist[i];
   932:     MachineBasicBlock &MBB = *MF.getBlockNumbered(BN);
   933:     for (auto &R : CSI)
   934:       if (!MBB.isLiveIn(R.getReg()))
   935:         MBB.addLiveIn(R.getReg());
   936:     if (BN != SaveN)
   937:       for (auto &SB : MBB.successors())
   938:         Worklist.insert(SB->getNumber());
   939:   }
   940: }
   941: 
   942: bool HexagonFrameLowering::updateExitPaths(MachineBasicBlock &MBB,
   943:       MachineBasicBlock &RestoreB, BitVector &DoneT, BitVector &DoneF,
   944:       BitVector &Path) const {
   945:   assert(MBB.getNumber() >= 0);
   946:   unsigned BN = MBB.getNumber();
   947:   if (Path[BN] || DoneF[BN])
   948:     return false;
   949:   if (DoneT[BN])
   950:     return true;
   951: 
   952:   auto &CSI = MBB.getParent()->getFrameInfo().getCalleeSavedInfo();
   953: 
   954:   Path[BN] = true;
   955:   bool ReachedExit = false;
   956:   for (auto &SB : MBB.successors())
   957:     ReachedExit |= updateExitPaths(*SB, RestoreB, DoneT, DoneF, Path);
   958: 
   959:   if (!MBB.empty() && MBB.back().isReturn()) {
   960:     // Add implicit uses of all callee-saved registers to the reached
   961:     // return instructions. This is to prevent the anti-dependency breaker
   962:     // from renaming these registers.
   963:     MachineInstr &RetI = MBB.back();
   964:     if (!isRestoreCall(RetI.getOpcode()))
   965:       for (auto &R : CSI)
   966:         RetI.addOperand(MachineOperand::CreateReg(R.getReg(), false, true));
   967:     ReachedExit = true;
   968:   }
   969: 
   970:   // We don't want to add unnecessary live-ins to the restore block: since
   971:   // the callee-saved registers are being defined in it, the entry of the
   972:   // restore block cannot be on the path from the definitions to any exit.
   973:   if (ReachedExit && &MBB != &RestoreB) {
   974:     for (auto &R : CSI)
   975:       if (!MBB.isLiveIn(R.getReg()))
   976:         MBB.addLiveIn(R.getReg());
   977:     DoneT[BN] = true;
   978:   }
   979:   if (!ReachedExit)
   980:     DoneF[BN] = true;
   981: 
   982:   Path[BN] = false;
   983:   return ReachedExit;
   984: }
   985: 
   986: static std::optional<MachineBasicBlock::iterator>
   987: findCFILocation(MachineBasicBlock &B) {
   988:     // The CFI instructions need to be inserted right after allocframe.
   989:     // An exception to this is a situation where allocframe is bundled
   990:     // with a call: then the CFI instructions need to be inserted before
   991:     // the packet with the allocframe+call (in case the call throws an
   992:     // exception).
   993:     auto End = B.instr_end();
   994: 
   995:     for (MachineInstr &I : B) {
   996:       MachineBasicBlock::iterator It = I.getIterator();
   997:       if (!I.isBundle()) {
   998:         if (I.getOpcode() == Hexagon::S2_allocframe)
   999:           return std::next(It);
  1000:         continue;
```
- EN: It declares or implements routines such as RESTORE, std::prev, getOpcode, BuildMI, ... (30 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonFrameLowering, Hexagon_TODO, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 RESTORE, std::prev, getOpcode, BuildMI, ... (30 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonFrameLowering, Hexagon_TODO，说明了它与同级后端组件的连接关系。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:       }
  1002:       // I is a bundle.
  1003:       bool HasCall = false, HasAllocFrame = false;
  1004:       auto T = It.getInstrIterator();
  1005:       while (++T != End && T->isBundled()) {
  1006:         if (T->getOpcode() == Hexagon::S2_allocframe)
  1007:           HasAllocFrame = true;
  1008:         else if (T->isCall())
  1009:           HasCall = true;
  1010:       }
  1011:       if (HasAllocFrame)
  1012:         return HasCall ? It : std::next(It);
  1013:     }
  1014:     return std::nullopt;
  1015: }
  1016: 
  1017: void HexagonFrameLowering::insertCFIInstructions(MachineFunction &MF) const {
  1018:     for (auto &B : MF)
  1019:       if (auto At = findCFILocation(B))
  1020:         insertCFIInstructionsAt(B, *At);
  1021: }
  1022: 
  1023: void HexagonFrameLowering::insertCFIInstructionsAt(MachineBasicBlock &MBB,
  1024:       MachineBasicBlock::iterator At) const {
  1025:   MachineFunction &MF = *MBB.getParent();
  1026:   MachineFrameInfo &MFI = MF.getFrameInfo();
  1027:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  1028:   auto &HII = *HST.getInstrInfo();
  1029:   auto &HRI = *HST.getRegisterInfo();
  1030: 
  1031:   // If CFI instructions have debug information attached, something goes
  1032:   // wrong with the final assembly generation: the prolog_end is placed
  1033:   // in a wrong location.
  1034:   DebugLoc DL;
  1035:   const MCInstrDesc &CFID = HII.get(TargetOpcode::CFI_INSTRUCTION);
  1036: 
  1037:   MCSymbol *FrameLabel = MF.getContext().createTempSymbol();
  1038:   bool HasFP = hasFP(MF);
  1039: 
  1040:   if (HasFP) {
  1041:     unsigned DwFPReg = HRI.getDwarfRegNum(HRI.getFrameRegister(), true);
  1042:     unsigned DwRAReg = HRI.getDwarfRegNum(HRI.getRARegister(), true);
  1043: 
  1044:     // Define CFA via an offset from the value of FP.
  1045:     //
  1046:     //  -8   -4    0 (SP)
  1047:     // --+----+----+---------------------
  1048:     //   | FP | LR |          increasing addresses -->
  1049:     // --+----+----+---------------------
  1050:     //   |         +-- Old SP (before allocframe)
  1051:     //   +-- New FP (after allocframe)
  1052:     //
  1053:     // MCCFIInstruction::cfiDefCfa adds the offset from the register.
  1054:     // MCCFIInstruction::createOffset takes the offset without sign change.
  1055:     auto DefCfa = MCCFIInstruction::cfiDefCfa(FrameLabel, DwFPReg, 8);
  1056:     BuildMI(MBB, At, DL, CFID)
  1057:         .addCFIIndex(MF.addFrameInst(DefCfa));
  1058:     // R31 (return addr) = CFA - 4
  1059:     auto OffR31 = MCCFIInstruction::createOffset(FrameLabel, DwRAReg, -4);
  1060:     BuildMI(MBB, At, DL, CFID)
  1061:         .addCFIIndex(MF.addFrameInst(OffR31));
  1062:     // R30 (frame ptr) = CFA - 8
  1063:     auto OffR30 = MCCFIInstruction::createOffset(FrameLabel, DwFPReg, -8);
  1064:     BuildMI(MBB, At, DL, CFID)
  1065:         .addCFIIndex(MF.addFrameInst(OffR30));
  1066:   }
  1067: 
  1068:   static const MCPhysReg RegsToMove[] = {
  1069:     Hexagon::R1,  Hexagon::R0,  Hexagon::R3,  Hexagon::R2,
  1070:     Hexagon::R17, Hexagon::R16, Hexagon::R19, Hexagon::R18,
  1071:     Hexagon::R21, Hexagon::R20, Hexagon::R23, Hexagon::R22,
  1072:     Hexagon::R25, Hexagon::R24, Hexagon::R27, Hexagon::R26,
  1073:     Hexagon::D0,  Hexagon::D1,  Hexagon::D8,  Hexagon::D9,
  1074:     Hexagon::D10, Hexagon::D11, Hexagon::D12, Hexagon::D13
  1075:   };
  1076: 
  1077:   const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
  1078: 
  1079:   for (MCPhysReg Reg : RegsToMove) {
  1080:     auto IfR = [Reg] (const CalleeSavedInfo &C) -> bool {
  1081:       return C.getReg() == Reg;
  1082:     };
  1083:     auto F = find_if(CSI, IfR);
  1084:     if (F == CSI.end())
  1085:       continue;
  1086: 
  1087:     int64_t Offset;
  1088:     if (HasFP) {
  1089:       // If the function has a frame pointer (i.e. has an allocframe),
  1090:       // then the CFA has been defined in terms of FP. Any offsets in
  1091:       // the following CFI instructions have to be defined relative
  1092:       // to FP, which points to the bottom of the stack frame.
  1093:       // The function getFrameIndexReference can still choose to use SP
  1094:       // for the offset calculation, so we cannot simply call it here.
  1095:       // Instead, get the offset (relative to the FP) directly.
  1096:       Offset = MFI.getObjectOffset(F->getFrameIdx());
  1097:     } else {
  1098:       Register FrameReg;
  1099:       Offset =
  1100:           getFrameIndexReference(MF, F->getFrameIdx(), FrameReg).getFixed();
  1101:     }
  1102:     // Subtract 8 to make room for R30 and R31, which are added above.
  1103:     Offset -= 8;
  1104: 
  1105:     if (Reg < Hexagon::D0 || Reg > Hexagon::D15) {
  1106:       unsigned DwarfReg = HRI.getDwarfRegNum(Reg, true);
  1107:       auto OffReg = MCCFIInstruction::createOffset(FrameLabel, DwarfReg,
  1108:                                                    Offset);
  1109:       BuildMI(MBB, At, DL, CFID)
  1110:           .addCFIIndex(MF.addFrameInst(OffReg));
  1111:     } else {
  1112:       // Split the double regs into subregs, and generate appropriate
  1113:       // cfi_offsets.
  1114:       // The only reason, we are split double regs is, llvm-mc does not
  1115:       // understand paired registers for cfi_offset.
  1116:       // Eg .cfi_offset r1:0, -64
  1117: 
  1118:       Register HiReg = HRI.getSubReg(Reg, Hexagon::isub_hi);
  1119:       Register LoReg = HRI.getSubReg(Reg, Hexagon::isub_lo);
  1120:       unsigned HiDwarfReg = HRI.getDwarfRegNum(HiReg, true);
  1121:       unsigned LoDwarfReg = HRI.getDwarfRegNum(LoReg, true);
  1122:       auto OffHi = MCCFIInstruction::createOffset(FrameLabel, HiDwarfReg,
  1123:                                                   Offset+4);
  1124:       BuildMI(MBB, At, DL, CFID)
  1125:           .addCFIIndex(MF.addFrameInst(OffHi));
  1126:       auto OffLo = MCCFIInstruction::createOffset(FrameLabel, LoDwarfReg,
  1127:                                                   Offset);
  1128:       BuildMI(MBB, At, DL, CFID)
  1129:           .addCFIIndex(MF.addFrameInst(OffLo));
  1130:     }
  1131:   }
  1132: }
  1133: 
  1134: bool HexagonFrameLowering::hasFPImpl(const MachineFunction &MF) const {
  1135:   auto &MFI = MF.getFrameInfo();
  1136:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  1137:   bool HasExtraAlign = HRI.hasStackRealignment(MF);
  1138:   bool HasAlloca = MFI.hasVarSizedObjects();
  1139: 
  1140:   // Insert ALLOCFRAME if we need to or at -O0 for the debugger.  Think
  1141:   // that this shouldn't be required, but doing so now because gcc does and
  1142:   // gdb can't break at the start of the function without it.  Will remove if
  1143:   // this turns out to be a gdb bug.
  1144:   //
  1145:   if (MF.getTarget().getOptLevel() == CodeGenOptLevel::None)
  1146:     return true;
  1147: 
  1148:   // By default we want to use SP (since it's always there). FP requires
  1149:   // some setup (i.e. ALLOCFRAME).
  1150:   // Both, alloca and stack alignment modify the stack pointer by an
  1151:   // undetermined value, so we need to save it at the entry to the function
  1152:   // (i.e. use allocframe).
  1153:   if (HasAlloca || HasExtraAlign)
  1154:     return true;
  1155: 
  1156:   // If FP-elimination is disabled, we have to use FP. This must not be
  1157:   // gated on stack size: the user/ABI-requested frame pointer is needed
  1158:   // regardless of whether the function currently has a stack frame.
  1159:   // Every other target checks DisableFramePointerElim unconditionally.
  1160:   const TargetMachine &TM = MF.getTarget();
  1161:   if (TM.Options.DisableFramePointerElim(MF) || !EliminateFramePointer)
  1162:     return true;
  1163: 
  1164:   if (MFI.getStackSize() > 0) {
  1165:     if (EnableStackOVFSanitizer)
  1166:       return true;
  1167:   }
  1168: 
  1169:   const auto &HMFI = *MF.getInfo<HexagonMachineFunctionInfo>();
  1170:   if ((MFI.hasCalls() && !enableAllocFrameElim(MF)) || HMFI.hasClobberLR())
  1171:     return true;
  1172: 
  1173:   return false;
  1174: }
  1175: 
  1176: enum SpillKind {
  1177:   SK_ToMem,
  1178:   SK_FromMem,
  1179:   SK_FromMemTailcall
  1180: };
  1181: 
  1182: static const char *getSpillFunctionFor(Register MaxReg, SpillKind SpillType,
  1183:       bool Stkchk = false) {
  1184:   const char * V4SpillToMemoryFunctions[] = {
  1185:     "__save_r16_through_r17",
  1186:     "__save_r16_through_r19",
  1187:     "__save_r16_through_r21",
  1188:     "__save_r16_through_r23",
  1189:     "__save_r16_through_r25",
  1190:     "__save_r16_through_r27" };
  1191: 
  1192:   const char * V4SpillToMemoryStkchkFunctions[] = {
  1193:     "__save_r16_through_r17_stkchk",
  1194:     "__save_r16_through_r19_stkchk",
  1195:     "__save_r16_through_r21_stkchk",
  1196:     "__save_r16_through_r23_stkchk",
  1197:     "__save_r16_through_r25_stkchk",
  1198:     "__save_r16_through_r27_stkchk" };
  1199: 
  1200:   const char * V4SpillFromMemoryFunctions[] = {
```
- EN: It declares types such as SpillKind, which carry the state or API of this component. It declares or implements routines such as getInstrIterator, HexagonFrameLowering::insertCFIInstructions, HexagonFrameLowering::insertCFIInstructionsAt, getParent, ... (28 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFrameLowering, HexagonSubtarget, HexagonMachineFunctionInfo, showing how the code connects to sibling backend components.
- CN: 这里声明了 SpillKind 等类型，用来承载该组件的状态或接口。 这里声明或实现了 getInstrIterator, HexagonFrameLowering::insertCFIInstructions, HexagonFrameLowering::insertCFIInstructionsAt, getParent, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFrameLowering, HexagonSubtarget, HexagonMachineFunctionInfo，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:     "__restore_r16_through_r17_and_deallocframe",
  1202:     "__restore_r16_through_r19_and_deallocframe",
  1203:     "__restore_r16_through_r21_and_deallocframe",
  1204:     "__restore_r16_through_r23_and_deallocframe",
  1205:     "__restore_r16_through_r25_and_deallocframe",
  1206:     "__restore_r16_through_r27_and_deallocframe" };
  1207: 
  1208:   const char * V4SpillFromMemoryTailcallFunctions[] = {
  1209:     "__restore_r16_through_r17_and_deallocframe_before_tailcall",
  1210:     "__restore_r16_through_r19_and_deallocframe_before_tailcall",
  1211:     "__restore_r16_through_r21_and_deallocframe_before_tailcall",
  1212:     "__restore_r16_through_r23_and_deallocframe_before_tailcall",
  1213:     "__restore_r16_through_r25_and_deallocframe_before_tailcall",
  1214:     "__restore_r16_through_r27_and_deallocframe_before_tailcall"
  1215:   };
  1216: 
  1217:   const char **SpillFunc = nullptr;
  1218: 
  1219:   switch(SpillType) {
  1220:   case SK_ToMem:
  1221:     SpillFunc = Stkchk ? V4SpillToMemoryStkchkFunctions
  1222:                        : V4SpillToMemoryFunctions;
  1223:     break;
  1224:   case SK_FromMem:
  1225:     SpillFunc = V4SpillFromMemoryFunctions;
  1226:     break;
  1227:   case SK_FromMemTailcall:
  1228:     SpillFunc = V4SpillFromMemoryTailcallFunctions;
  1229:     break;
  1230:   }
  1231:   assert(SpillFunc && "Unknown spill kind");
  1232: 
  1233:   // Spill all callee-saved registers up to the highest register used.
  1234:   switch (MaxReg) {
  1235:   case Hexagon::R17:
  1236:     return SpillFunc[0];
  1237:   case Hexagon::R19:
  1238:     return SpillFunc[1];
  1239:   case Hexagon::R21:
  1240:     return SpillFunc[2];
  1241:   case Hexagon::R23:
  1242:     return SpillFunc[3];
  1243:   case Hexagon::R25:
  1244:     return SpillFunc[4];
  1245:   case Hexagon::R27:
  1246:     return SpillFunc[5];
  1247:   default:
  1248:     llvm_unreachable("Unhandled maximum callee save register");
  1249:   }
  1250:   return nullptr;
  1251: }
  1252: 
  1253: StackOffset
  1254: HexagonFrameLowering::getFrameIndexReference(const MachineFunction &MF, int FI,
  1255:                                              Register &FrameReg) const {
  1256:   auto &MFI = MF.getFrameInfo();
  1257:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  1258: 
  1259:   int Offset = MFI.getObjectOffset(FI);
  1260:   bool HasAlloca = MFI.hasVarSizedObjects();
  1261:   bool HasExtraAlign = HRI.hasStackRealignment(MF);
  1262:   bool NoOpt = MF.getTarget().getOptLevel() == CodeGenOptLevel::None;
  1263: 
  1264:   auto &HMFI = *MF.getInfo<HexagonMachineFunctionInfo>();
  1265:   unsigned FrameSize = MFI.getStackSize();
  1266:   Register SP = HRI.getStackRegister();
  1267:   Register FP = HRI.getFrameRegister();
  1268:   Register AP = HMFI.getStackAlignBaseReg();
  1269:   // It may happen that AP will be absent even HasAlloca && HasExtraAlign
  1270:   // is true. HasExtraAlign may be set because of vector spills, without
  1271:   // aligned locals or aligned outgoing function arguments. Since vector
  1272:   // spills will ultimately be "unaligned", it is safe to use FP as the
  1273:   // base register.
  1274:   // In fact, in such a scenario the stack is actually not required to be
  1275:   // aligned, although it may end up being aligned anyway, since this
  1276:   // particular case is not easily detectable. The alignment will be
  1277:   // unnecessary, but not incorrect.
  1278:   // Unfortunately there is no quick way to verify that the above is
  1279:   // indeed the case (and that it's not a result of an error), so just
  1280:   // assume that missing AP will be replaced by FP.
  1281:   // (A better fix would be to rematerialize AP from FP and always align
  1282:   // vector spills.)
  1283:   bool UseFP = false, UseAP = false;  // Default: use SP (except at -O0).
  1284:   // Use FP at -O0, except when there are objects with extra alignment.
  1285:   // That additional alignment requirement may cause a pad to be inserted,
  1286:   // which will make it impossible to use FP to access objects located
  1287:   // past the pad.
  1288:   if (NoOpt && !HasExtraAlign)
  1289:     UseFP = true;
  1290:   if (MFI.isFixedObjectIndex(FI) || MFI.isObjectPreAllocated(FI)) {
  1291:     // Fixed and preallocated objects will be located before any padding
  1292:     // so FP must be used to access them.
  1293:     UseFP |= (HasAlloca || HasExtraAlign);
  1294:   } else {
  1295:     if (HasAlloca) {
  1296:       if (HasExtraAlign)
  1297:         UseAP = true;
  1298:       else
  1299:         UseFP = true;
  1300:     }
  1301:   }
  1302: 
  1303:   // If FP was picked, then there had better be FP.
  1304:   bool HasFP = hasFP(MF);
  1305:   assert((HasFP || !UseFP) && "This function must have frame pointer");
  1306: 
  1307:   // Having FP implies allocframe. Allocframe will store extra 8 bytes:
  1308:   // FP/LR. If the base register is used to access an object across these
  1309:   // 8 bytes, then the offset will need to be adjusted by 8.
  1310:   //
  1311:   // After allocframe:
  1312:   //                    HexagonISelLowering adds 8 to ---+
  1313:   //                    the offsets of all stack-based   |
  1314:   //                    arguments (*)                    |
  1315:   //                                                     |
  1316:   //   getObjectOffset < 0   0     8  getObjectOffset >= 8
  1317:   // ------------------------+-----+------------------------> increasing
  1318:   //     <local objects>     |FP/LR|    <input arguments>     addresses
  1319:   // -----------------+------+-----+------------------------>
  1320:   //                  |      |
  1321:   //    SP/AP point --+      +-- FP points here (**)
  1322:   //    somewhere on
  1323:   //    this side of FP/LR
  1324:   //
  1325:   // (*) See LowerFormalArguments. The FP/LR is assumed to be present.
  1326:   // (**) *FP == old-FP. FP+0..7 are the bytes of FP/LR.
  1327: 
  1328:   // The lowering assumes that FP/LR is present, and so the offsets of
  1329:   // the formal arguments start at 8. If FP/LR is not there we need to
  1330:   // reduce the offset by 8.
  1331:   if (Offset > 0 && !HasFP)
  1332:     Offset -= 8;
  1333: 
  1334:   if (UseFP)
  1335:     FrameReg = FP;
  1336:   else if (UseAP)
  1337:     FrameReg = AP;
  1338:   else
  1339:     FrameReg = SP;
  1340: 
  1341:   // Calculate the actual offset in the instruction. If there is no FP
  1342:   // (in other words, no allocframe), then SP will not be adjusted (i.e.
  1343:   // there will be no SP -= FrameSize), so the frame size should not be
  1344:   // added to the calculated offset.
  1345:   int RealOffset = Offset;
  1346:   if (!UseFP && !UseAP)
  1347:     RealOffset = FrameSize+Offset;
  1348:   return StackOffset::getFixed(RealOffset);
  1349: }
  1350: 
  1351: bool HexagonFrameLowering::insertCSRSpillsInBlock(MachineBasicBlock &MBB,
  1352:       const CSIVect &CSI, const HexagonRegisterInfo &HRI,
  1353:       bool &PrologueStubs) const {
  1354:   if (CSI.empty())
  1355:     return true;
  1356: 
  1357:   MachineBasicBlock::iterator MI = MBB.begin();
  1358:   PrologueStubs = false;
  1359:   MachineFunction &MF = *MBB.getParent();
  1360:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  1361:   auto &HII = *HST.getInstrInfo();
  1362: 
  1363:   if (useSpillFunction(MF, CSI)) {
  1364:     PrologueStubs = true;
  1365:     Register MaxReg = getMaxCalleeSavedReg(CSI, HRI);
  1366:     bool StkOvrFlowEnabled = EnableStackOVFSanitizer;
  1367:     const char *SpillFun = getSpillFunctionFor(MaxReg, SK_ToMem,
  1368:                                                StkOvrFlowEnabled);
  1369:     auto &HTM = static_cast<const HexagonTargetMachine&>(MF.getTarget());
  1370:     bool IsPIC = HTM.isPositionIndependent();
  1371:     bool LongCalls = HST.useLongCalls() || EnableSaveRestoreLong;
  1372: 
  1373:     // Call spill function.
  1374:     DebugLoc DL = MI != MBB.end() ? MI->getDebugLoc() : DebugLoc();
  1375:     unsigned SpillOpc;
  1376:     if (StkOvrFlowEnabled) {
  1377:       if (LongCalls)
  1378:         SpillOpc = IsPIC ? Hexagon::SAVE_REGISTERS_CALL_V4STK_EXT_PIC
  1379:                          : Hexagon::SAVE_REGISTERS_CALL_V4STK_EXT;
  1380:       else
  1381:         SpillOpc = IsPIC ? Hexagon::SAVE_REGISTERS_CALL_V4STK_PIC
  1382:                          : Hexagon::SAVE_REGISTERS_CALL_V4STK;
  1383:     } else {
  1384:       if (LongCalls)
  1385:         SpillOpc = IsPIC ? Hexagon::SAVE_REGISTERS_CALL_V4_EXT_PIC
  1386:                          : Hexagon::SAVE_REGISTERS_CALL_V4_EXT;
  1387:       else
  1388:         SpillOpc = IsPIC ? Hexagon::SAVE_REGISTERS_CALL_V4_PIC
  1389:                          : Hexagon::SAVE_REGISTERS_CALL_V4;
  1390:     }
  1391: 
  1392:     MachineInstr *SaveRegsCall =
  1393:         BuildMI(MBB, MI, DL, HII.get(SpillOpc))
  1394:           .addExternalSymbol(SpillFun);
  1395: 
  1396:     // Add callee-saved registers as use.
  1397:     addCalleeSaveRegistersAsImpOperand(SaveRegsCall, CSI, false, true);
  1398:     // Add live in registers.
  1399:     for (const CalleeSavedInfo &I : CSI)
  1400:       MBB.addLiveIn(I.getReg());
```
- EN: It declares or implements routines such as assert, llvm_unreachable, HexagonFrameLowering::getFrameIndexReference, getFrameInfo, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonFrameLowering, HexagonSubtarget, HexagonMachineFunctionInfo, HexagonISelLowering, ... (6 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, llvm_unreachable, HexagonFrameLowering::getFrameIndexReference, getFrameInfo, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonFrameLowering, HexagonSubtarget, HexagonMachineFunctionInfo, HexagonISelLowering, ... (6 total)，说明了它与同级后端组件的连接关系。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:   } else {
  1402:     for (const CalleeSavedInfo &I : CSI) {
  1403:       MCRegister Reg = I.getReg();
  1404:       // Add live in registers. We treat eh_return callee saved register r0 - r3
  1405:       // specially. They are not really callee saved registers as they are not
  1406:       // supposed to be killed.
  1407:       bool IsKill = !HRI.isEHReturnCalleeSaveReg(Reg);
  1408:       int FI = I.getFrameIdx();
  1409:       const TargetRegisterClass *RC = HRI.getMinimalPhysRegClass(Reg);
  1410:       HII.storeRegToStackSlot(MBB, MI, Reg, IsKill, FI, RC, Register());
  1411:       if (IsKill)
  1412:         MBB.addLiveIn(Reg);
  1413:     }
  1414:   }
  1415: 
  1416:   // Move PS_aligna to after all CSR spills (both inline and spill-function
  1417:   // paths). PS_aligna initializes the AP register (e.g. R16) with an aligned
  1418:   // value derived from FP. Since AP is a callee-saved register, its original
  1419:   // value must be saved before it is overwritten, and it must be defined
  1420:   // before any AP-relative stack accesses.
  1421:   // MI points to the first non-spill instruction; all spills are before it.
  1422:   auto &HFI = *MF.getSubtarget<HexagonSubtarget>().getFrameLowering();
  1423:   if (const MachineInstr *AlignaI = HFI.getAlignaInstr(MF)) {
  1424:     MachineInstr *AI = const_cast<MachineInstr *>(AlignaI);
  1425:     // PS_aligna is always created in EntryBB during ISEL. Since PS_aligna
  1426:     // causes needsStackFrame() to return true, EntryBB will be included in
  1427:     // the set of blocks needing a frame. Because EntryBB dominates all blocks,
  1428:     // shrink-wrapping will always place PrologB at EntryBB when PS_aligna
  1429:     // exists. Therefore, this assertion should always hold.
  1430:     assert(AI->getParent() == &MBB && "PS_aligna not in prologue block");
  1431:     MBB.splice(MI, AI->getParent(), AI->getIterator());
  1432:   }
  1433: 
  1434:   return true;
  1435: }
  1436: 
  1437: bool HexagonFrameLowering::insertCSRRestoresInBlock(MachineBasicBlock &MBB,
  1438:       const CSIVect &CSI, const HexagonRegisterInfo &HRI) const {
  1439:   if (CSI.empty())
  1440:     return false;
  1441: 
  1442:   MachineBasicBlock::iterator MI = MBB.getFirstTerminator();
  1443:   MachineFunction &MF = *MBB.getParent();
  1444:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  1445:   auto &HII = *HST.getInstrInfo();
  1446: 
  1447:   if (useRestoreFunction(MF, CSI)) {
  1448:     bool HasTC = hasTailCall(MBB) || !hasReturn(MBB);
  1449:     Register MaxR = getMaxCalleeSavedReg(CSI, HRI);
  1450:     SpillKind Kind = HasTC ? SK_FromMemTailcall : SK_FromMem;
  1451:     const char *RestoreFn = getSpillFunctionFor(MaxR, Kind);
  1452:     auto &HTM = static_cast<const HexagonTargetMachine&>(MF.getTarget());
  1453:     bool IsPIC = HTM.isPositionIndependent();
  1454:     bool LongCalls = HST.useLongCalls() || EnableSaveRestoreLong;
  1455: 
  1456:     // Call spill function.
  1457:     DebugLoc DL = MI != MBB.end() ? MI->getDebugLoc()
  1458:                                   : MBB.findDebugLoc(MBB.end());
  1459:     MachineInstr *DeallocCall = nullptr;
  1460: 
  1461:     if (HasTC) {
  1462:       unsigned RetOpc;
  1463:       if (LongCalls)
  1464:         RetOpc = IsPIC ? Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT_PIC
  1465:                        : Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_EXT;
  1466:       else
  1467:         RetOpc = IsPIC ? Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4_PIC
  1468:                        : Hexagon::RESTORE_DEALLOC_BEFORE_TAILCALL_V4;
  1469:       DeallocCall = BuildMI(MBB, MI, DL, HII.get(RetOpc))
  1470:           .addExternalSymbol(RestoreFn);
  1471:     } else {
  1472:       // The block has a return.
  1473:       MachineBasicBlock::iterator It = MBB.getFirstTerminator();
  1474:       assert(It->isReturn() && std::next(It) == MBB.end());
  1475:       unsigned RetOpc;
  1476:       if (LongCalls)
  1477:         RetOpc = IsPIC ? Hexagon::RESTORE_DEALLOC_RET_JMP_V4_EXT_PIC
  1478:                        : Hexagon::RESTORE_DEALLOC_RET_JMP_V4_EXT;
  1479:       else
  1480:         RetOpc = IsPIC ? Hexagon::RESTORE_DEALLOC_RET_JMP_V4_PIC
  1481:                        : Hexagon::RESTORE_DEALLOC_RET_JMP_V4;
  1482:       DeallocCall = BuildMI(MBB, It, DL, HII.get(RetOpc))
  1483:           .addExternalSymbol(RestoreFn);
  1484:       // Transfer the function live-out registers.
  1485:       DeallocCall->copyImplicitOps(MF, *It);
  1486:     }
  1487:     addCalleeSaveRegistersAsImpOperand(DeallocCall, CSI, true, false);
  1488:     return true;
  1489:   }
  1490: 
  1491:   for (const CalleeSavedInfo &I : CSI) {
  1492:     MCRegister Reg = I.getReg();
  1493:     const TargetRegisterClass *RC = HRI.getMinimalPhysRegClass(Reg);
  1494:     int FI = I.getFrameIdx();
  1495:     HII.loadRegFromStackSlot(MBB, MI, Reg, FI, RC, Register());
  1496:   }
  1497: 
  1498:   return true;
  1499: }
  1500: 
  1501: MachineBasicBlock::iterator HexagonFrameLowering::eliminateCallFramePseudoInstr(
  1502:     MachineFunction &MF, MachineBasicBlock &MBB,
  1503:     MachineBasicBlock::iterator I) const {
  1504:   MachineInstr &MI = *I;
  1505:   unsigned Opc = MI.getOpcode();
  1506:   (void)Opc; // Silence compiler warning.
  1507:   assert((Opc == Hexagon::ADJCALLSTACKDOWN || Opc == Hexagon::ADJCALLSTACKUP) &&
  1508:          "Cannot handle this call frame pseudo instruction");
  1509:   return MBB.erase(I);
  1510: }
  1511: 
  1512: void HexagonFrameLowering::processFunctionBeforeFrameFinalized(
  1513:     MachineFunction &MF, RegScavenger *RS) const {
  1514:   // If this function has uses aligned stack and also has variable sized stack
  1515:   // objects, then we need to map all spill slots to fixed positions, so that
  1516:   // they can be accessed through FP. Otherwise they would have to be accessed
  1517:   // via AP, which may not be available at the particular place in the program.
  1518:   MachineFrameInfo &MFI = MF.getFrameInfo();
  1519:   bool HasAlloca = MFI.hasVarSizedObjects();
  1520:   bool NeedsAlign = (MFI.getMaxAlign() > getStackAlign());
  1521: 
  1522:   if (!HasAlloca || !NeedsAlign)
  1523:     return;
  1524: 
  1525:   // Set the physical aligned-stack base address register.
  1526:   MCRegister AP;
  1527:   if (const MachineInstr *AI = getAlignaInstr(MF))
  1528:     AP = AI->getOperand(0).getReg();
  1529:   auto &HMFI = *MF.getInfo<HexagonMachineFunctionInfo>();
  1530:   assert(!AP.isValid() || AP.isPhysical());
  1531:   HMFI.setStackAlignBaseReg(AP);
  1532: }
  1533: 
  1534: /// Returns true if there are no caller-saved registers available in class RC.
  1535: static bool needToReserveScavengingSpillSlots(MachineFunction &MF,
  1536:       const HexagonRegisterInfo &HRI, const TargetRegisterClass *RC) {
  1537:   MachineRegisterInfo &MRI = MF.getRegInfo();
  1538: 
  1539:   auto IsUsed = [&HRI,&MRI] (Register Reg) -> bool {
  1540:     for (MCRegAliasIterator AI(Reg, &HRI, true); AI.isValid(); ++AI)
  1541:       if (MRI.isPhysRegUsed(*AI))
  1542:         return true;
  1543:     return false;
  1544:   };
  1545: 
  1546:   // Check for an unused caller-saved register. Callee-saved registers
  1547:   // have become pristine by now.
  1548:   for (const MCPhysReg *P = HRI.getCallerSavedRegs(&MF, RC); *P; ++P)
  1549:     if (!IsUsed(*P))
  1550:       return false;
  1551: 
  1552:   // All caller-saved registers are used.
  1553:   return true;
  1554: }
  1555: 
  1556: #ifndef NDEBUG
  1557: static void dump_registers(BitVector &Regs, const TargetRegisterInfo &TRI) {
  1558:   dbgs() << '{';
  1559:   for (int x = Regs.find_first(); x >= 0; x = Regs.find_next(x)) {
  1560:     Register R = x;
  1561:     dbgs() << ' ' << printReg(R, &TRI);
  1562:   }
  1563:   dbgs() << " }";
  1564: }
  1565: #endif
  1566: 
  1567: bool HexagonFrameLowering::assignCalleeSavedSpillSlots(MachineFunction &MF,
  1568:       const TargetRegisterInfo *TRI, std::vector<CalleeSavedInfo> &CSI) const {
  1569:   LLVM_DEBUG(dbgs() << __func__ << " on " << MF.getName() << '\n');
  1570:   MachineFrameInfo &MFI = MF.getFrameInfo();
  1571:   BitVector SRegs(Hexagon::NUM_TARGET_REGS);
  1572: 
  1573:   // Generate a set of unique, callee-saved registers (SRegs), where each
  1574:   // register in the set is maximal in terms of sub-/super-register relation,
  1575:   // i.e. for each R in SRegs, no proper super-register of R is also in SRegs.
  1576: 
  1577:   // (1) For each callee-saved register, add that register and all of its
  1578:   // sub-registers to SRegs.
  1579:   LLVM_DEBUG(dbgs() << "Initial CS registers: {");
  1580:   for (const CalleeSavedInfo &I : CSI) {
  1581:     Register R = I.getReg();
  1582:     LLVM_DEBUG(dbgs() << ' ' << printReg(R, TRI));
  1583:     for (MCPhysReg SR : TRI->subregs_inclusive(R))
  1584:       SRegs[SR] = true;
  1585:   }
  1586:   LLVM_DEBUG(dbgs() << " }\n");
  1587:   LLVM_DEBUG(dbgs() << "SRegs.1: "; dump_registers(SRegs, *TRI);
  1588:              dbgs() << "\n");
  1589: 
  1590:   // (2) For each reserved register, remove that register and all of its
  1591:   // sub- and super-registers from SRegs.
  1592:   BitVector Reserved = TRI->getReservedRegs(MF);
  1593:   // Unreserve the stack align register: it is reserved for this function
  1594:   // only, it still needs to be saved/restored.
  1595:   Register AP =
  1596:       MF.getInfo<HexagonMachineFunctionInfo>()->getStackAlignBaseReg();
  1597:   if (AP.isValid()) {
  1598:     Reserved[AP] = false;
  1599:     // Unreserve super-regs if no other subregisters are reserved.
  1600:     for (MCPhysReg SP : TRI->superregs(AP)) {
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getReg, isEHReturnCalleeSaveReg, getFrameIdx, getMinimalPhysRegClass, ... (42 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getReg, isEHReturnCalleeSaveReg, getFrameIdx, getMinimalPhysRegClass, ... (42 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:       bool HasResSub = false;
  1602:       for (MCPhysReg SB : TRI->subregs(SP)) {
  1603:         if (!Reserved[SB])
  1604:           continue;
  1605:         HasResSub = true;
  1606:         break;
  1607:       }
  1608:       if (!HasResSub)
  1609:         Reserved[SP] = false;
  1610:     }
  1611:   }
  1612: 
  1613:   for (int x = Reserved.find_first(); x >= 0; x = Reserved.find_next(x)) {
  1614:     Register R = x;
  1615:     for (MCPhysReg SR : TRI->superregs_inclusive(R))
  1616:       SRegs[SR] = false;
  1617:   }
  1618:   LLVM_DEBUG(dbgs() << "Res:     "; dump_registers(Reserved, *TRI);
  1619:              dbgs() << "\n");
  1620:   LLVM_DEBUG(dbgs() << "SRegs.2: "; dump_registers(SRegs, *TRI);
  1621:              dbgs() << "\n");
  1622: 
  1623:   // (3) Collect all registers that have at least one sub-register in SRegs,
  1624:   // and also have no sub-registers that are reserved. These will be the can-
  1625:   // didates for saving as a whole instead of their individual sub-registers.
  1626:   // (Saving R17:16 instead of R16 is fine, but only if R17 was not reserved.)
  1627:   BitVector TmpSup(Hexagon::NUM_TARGET_REGS);
  1628:   for (int x = SRegs.find_first(); x >= 0; x = SRegs.find_next(x)) {
  1629:     Register R = x;
  1630:     for (MCPhysReg SR : TRI->superregs(R))
  1631:       TmpSup[SR] = true;
  1632:   }
  1633:   for (int x = TmpSup.find_first(); x >= 0; x = TmpSup.find_next(x)) {
  1634:     Register R = x;
  1635:     for (MCPhysReg SR : TRI->subregs_inclusive(R)) {
  1636:       if (!Reserved[SR])
  1637:         continue;
  1638:       TmpSup[R] = false;
  1639:       break;
  1640:     }
  1641:   }
  1642:   LLVM_DEBUG(dbgs() << "TmpSup:  "; dump_registers(TmpSup, *TRI);
  1643:              dbgs() << "\n");
  1644: 
  1645:   // (4) Include all super-registers found in (3) into SRegs.
  1646:   SRegs |= TmpSup;
  1647:   LLVM_DEBUG(dbgs() << "SRegs.4: "; dump_registers(SRegs, *TRI);
  1648:              dbgs() << "\n");
  1649: 
  1650:   // (5) For each register R in SRegs, if any super-register of R is in SRegs,
  1651:   // remove R from SRegs.
  1652:   for (int x = SRegs.find_first(); x >= 0; x = SRegs.find_next(x)) {
  1653:     Register R = x;
  1654:     for (MCPhysReg SR : TRI->superregs(R)) {
  1655:       if (!SRegs[SR])
  1656:         continue;
  1657:       SRegs[R] = false;
  1658:       break;
  1659:     }
  1660:   }
  1661:   LLVM_DEBUG(dbgs() << "SRegs.5: "; dump_registers(SRegs, *TRI);
  1662:              dbgs() << "\n");
  1663: 
  1664:   // Now, for each register that has a fixed stack slot, create the stack
  1665:   // object for it.
  1666:   CSI.clear();
  1667: 
  1668:   using SpillSlot = TargetFrameLowering::SpillSlot;
  1669: 
  1670:   unsigned NumFixed;
  1671:   int64_t MinOffset = 0; // CS offsets are negative.
  1672:   const SpillSlot *FixedSlots = getCalleeSavedSpillSlots(NumFixed);
  1673:   for (const SpillSlot *S = FixedSlots; S != FixedSlots+NumFixed; ++S) {
  1674:     if (!SRegs[S->Reg])
  1675:       continue;
  1676:     const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(S->Reg);
  1677:     int FI = MFI.CreateFixedSpillStackObject(TRI->getSpillSize(*RC), S->Offset);
  1678:     MinOffset = std::min(MinOffset, S->Offset);
  1679:     CSI.push_back(CalleeSavedInfo(S->Reg, FI));
  1680:     SRegs[S->Reg] = false;
  1681:   }
  1682: 
  1683:   // There can be some registers that don't have fixed slots. For example,
  1684:   // we need to store R0-R3 in functions with exception handling. For each
  1685:   // such register, create a non-fixed stack object.
  1686:   for (int x = SRegs.find_first(); x >= 0; x = SRegs.find_next(x)) {
  1687:     Register R = x;
  1688:     const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(R);
  1689:     unsigned Size = TRI->getSpillSize(*RC);
  1690:     int64_t Off = MinOffset - Size;
  1691:     Align Alignment = std::min(TRI->getSpillAlign(*RC), getStackAlign());
  1692:     Off &= -Alignment.value();
  1693:     int FI = MFI.CreateFixedSpillStackObject(Size, Off);
  1694:     MinOffset = std::min(MinOffset, Off);
  1695:     CSI.push_back(CalleeSavedInfo(R, FI));
  1696:     SRegs[R] = false;
  1697:   }
  1698: 
  1699:   LLVM_DEBUG({
  1700:     dbgs() << "CS information: {";
  1701:     for (const CalleeSavedInfo &I : CSI) {
  1702:       int FI = I.getFrameIdx();
  1703:       int Off = MFI.getObjectOffset(FI);
  1704:       dbgs() << ' ' << printReg(I.getReg(), TRI) << ":fi#" << FI << ":sp";
  1705:       if (Off >= 0)
  1706:         dbgs() << '+';
  1707:       dbgs() << Off;
  1708:     }
  1709:     dbgs() << " }\n";
  1710:   });
  1711: 
  1712: #ifndef NDEBUG
  1713:   // Verify that all registers were handled.
  1714:   bool MissedReg = false;
  1715:   for (int x = SRegs.find_first(); x >= 0; x = SRegs.find_next(x)) {
  1716:     Register R = x;
  1717:     dbgs() << printReg(R, TRI) << ' ';
  1718:     MissedReg = true;
  1719:   }
  1720:   if (MissedReg)
  1721:     llvm_unreachable("...there are unhandled callee-saved registers!");
  1722: #endif
  1723: 
  1724:   return true;
  1725: }
  1726: 
  1727: bool HexagonFrameLowering::expandCopy(MachineBasicBlock &B,
  1728:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1729:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1730:   MachineInstr *MI = &*It;
  1731:   DebugLoc DL = MI->getDebugLoc();
  1732:   Register DstR = MI->getOperand(0).getReg();
  1733:   Register SrcR = MI->getOperand(1).getReg();
  1734:   if (!Hexagon::ModRegsRegClass.contains(DstR) ||
  1735:       !Hexagon::ModRegsRegClass.contains(SrcR))
  1736:     return false;
  1737: 
  1738:   Register TmpR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  1739:   BuildMI(B, It, DL, HII.get(TargetOpcode::COPY), TmpR).add(MI->getOperand(1));
  1740:   BuildMI(B, It, DL, HII.get(TargetOpcode::COPY), DstR)
  1741:     .addReg(TmpR, RegState::Kill);
  1742: 
  1743:   NewRegs.push_back(TmpR);
  1744:   B.erase(It);
  1745:   return true;
  1746: }
  1747: 
  1748: bool HexagonFrameLowering::expandStoreInt(MachineBasicBlock &B,
  1749:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1750:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1751:   MachineInstr *MI = &*It;
  1752:   if (!MI->getOperand(0).isFI())
  1753:     return false;
  1754: 
  1755:   DebugLoc DL = MI->getDebugLoc();
  1756:   unsigned Opc = MI->getOpcode();
  1757:   Register SrcR = MI->getOperand(2).getReg();
  1758:   bool IsKill = MI->getOperand(2).isKill();
  1759:   int FI = MI->getOperand(0).getIndex();
  1760: 
  1761:   // TmpR = C2_tfrpr SrcR   if SrcR is a predicate register
  1762:   // TmpR = A2_tfrcrr SrcR  if SrcR is a modifier register
  1763:   Register TmpR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  1764:   unsigned TfrOpc = (Opc == Hexagon::STriw_pred) ? Hexagon::C2_tfrpr
  1765:                                                  : Hexagon::A2_tfrcrr;
  1766:   BuildMI(B, It, DL, HII.get(TfrOpc), TmpR)
  1767:     .addReg(SrcR, getKillRegState(IsKill));
  1768: 
  1769:   // S2_storeri_io FI, 0, TmpR
  1770:   BuildMI(B, It, DL, HII.get(Hexagon::S2_storeri_io))
  1771:       .addFrameIndex(FI)
  1772:       .addImm(0)
  1773:       .addReg(TmpR, RegState::Kill)
  1774:       .cloneMemRefs(*MI);
  1775: 
  1776:   NewRegs.push_back(TmpR);
  1777:   B.erase(It);
  1778:   return true;
  1779: }
  1780: 
  1781: bool HexagonFrameLowering::expandLoadInt(MachineBasicBlock &B,
  1782:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1783:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1784:   MachineInstr *MI = &*It;
  1785:   if (!MI->getOperand(1).isFI())
  1786:     return false;
  1787: 
  1788:   DebugLoc DL = MI->getDebugLoc();
  1789:   unsigned Opc = MI->getOpcode();
  1790:   Register DstR = MI->getOperand(0).getReg();
  1791:   int FI = MI->getOperand(1).getIndex();
  1792: 
  1793:   // TmpR = L2_loadri_io FI, 0
  1794:   Register TmpR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  1795:   BuildMI(B, It, DL, HII.get(Hexagon::L2_loadri_io), TmpR)
  1796:       .addFrameIndex(FI)
  1797:       .addImm(0)
  1798:       .cloneMemRefs(*MI);
  1799: 
  1800:   // DstR = C2_tfrrp TmpR   if DstR is a predicate register
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as find_next, dump_registers, dbgs, TmpSup, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFrameLowering, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 find_next, dump_registers, dbgs, TmpSup, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFrameLowering, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:   // DstR = A2_tfrrcr TmpR  if DstR is a modifier register
  1802:   unsigned TfrOpc = (Opc == Hexagon::LDriw_pred) ? Hexagon::C2_tfrrp
  1803:                                                  : Hexagon::A2_tfrrcr;
  1804:   BuildMI(B, It, DL, HII.get(TfrOpc), DstR)
  1805:     .addReg(TmpR, RegState::Kill);
  1806: 
  1807:   NewRegs.push_back(TmpR);
  1808:   B.erase(It);
  1809:   return true;
  1810: }
  1811: 
  1812: bool HexagonFrameLowering::expandStoreVecPred(MachineBasicBlock &B,
  1813:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1814:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1815:   MachineInstr *MI = &*It;
  1816:   if (!MI->getOperand(0).isFI())
  1817:     return false;
  1818: 
  1819:   DebugLoc DL = MI->getDebugLoc();
  1820:   Register SrcR = MI->getOperand(2).getReg();
  1821:   bool IsKill = MI->getOperand(2).isKill();
  1822:   int FI = MI->getOperand(0).getIndex();
  1823:   auto *RC = &Hexagon::HvxVRRegClass;
  1824: 
  1825:   // Insert transfer to general vector register.
  1826:   //   TmpR0 = A2_tfrsi 0x01010101
  1827:   //   TmpR1 = V6_vandqrt Qx, TmpR0
  1828:   //   store FI, 0, TmpR1
  1829:   Register TmpR0 = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  1830:   Register TmpR1 = MRI.createVirtualRegister(RC);
  1831: 
  1832:   BuildMI(B, It, DL, HII.get(Hexagon::A2_tfrsi), TmpR0)
  1833:     .addImm(0x01010101);
  1834: 
  1835:   BuildMI(B, It, DL, HII.get(Hexagon::V6_vandqrt), TmpR1)
  1836:     .addReg(SrcR, getKillRegState(IsKill))
  1837:     .addReg(TmpR0, RegState::Kill);
  1838: 
  1839:   HII.storeRegToStackSlot(B, It, TmpR1, true, FI, RC, Register());
  1840:   expandStoreVec(B, std::prev(It), MRI, HII, NewRegs);
  1841: 
  1842:   NewRegs.push_back(TmpR0);
  1843:   NewRegs.push_back(TmpR1);
  1844:   B.erase(It);
  1845:   return true;
  1846: }
  1847: 
  1848: bool HexagonFrameLowering::expandLoadVecPred(MachineBasicBlock &B,
  1849:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1850:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1851:   MachineInstr *MI = &*It;
  1852:   if (!MI->getOperand(1).isFI())
  1853:     return false;
  1854: 
  1855:   DebugLoc DL = MI->getDebugLoc();
  1856:   Register DstR = MI->getOperand(0).getReg();
  1857:   int FI = MI->getOperand(1).getIndex();
  1858:   auto *RC = &Hexagon::HvxVRRegClass;
  1859: 
  1860:   // TmpR0 = A2_tfrsi 0x01010101
  1861:   // TmpR1 = load FI, 0
  1862:   // DstR = V6_vandvrt TmpR1, TmpR0
  1863:   Register TmpR0 = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
  1864:   Register TmpR1 = MRI.createVirtualRegister(RC);
  1865: 
  1866:   BuildMI(B, It, DL, HII.get(Hexagon::A2_tfrsi), TmpR0)
  1867:     .addImm(0x01010101);
  1868:   HII.loadRegFromStackSlot(B, It, TmpR1, FI, RC, Register());
  1869:   expandLoadVec(B, std::prev(It), MRI, HII, NewRegs);
  1870: 
  1871:   BuildMI(B, It, DL, HII.get(Hexagon::V6_vandvrt), DstR)
  1872:     .addReg(TmpR1, RegState::Kill)
  1873:     .addReg(TmpR0, RegState::Kill);
  1874: 
  1875:   NewRegs.push_back(TmpR0);
  1876:   NewRegs.push_back(TmpR1);
  1877:   B.erase(It);
  1878:   return true;
  1879: }
  1880: 
  1881: bool HexagonFrameLowering::expandStoreVec2(MachineBasicBlock &B,
  1882:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1883:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1884:   MachineFunction &MF = *B.getParent();
  1885:   auto &MFI = MF.getFrameInfo();
  1886:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  1887:   MachineInstr *MI = &*It;
  1888:   if (!MI->getOperand(0).isFI())
  1889:     return false;
  1890: 
  1891:   // It is possible that the double vector being stored is only partially
  1892:   // defined. From the point of view of the liveness tracking, it is ok to
  1893:   // store it as a whole, but if we break it up we may end up storing a
  1894:   // register that is entirely undefined.
  1895:   LivePhysRegs LPR(HRI);
  1896:   LPR.addLiveIns(B);
  1897:   SmallVector<std::pair<MCPhysReg, const MachineOperand*>,2> Clobbers;
  1898:   for (auto R = B.begin(); R != It; ++R) {
  1899:     Clobbers.clear();
  1900:     LPR.stepForward(*R, Clobbers);
  1901:   }
  1902: 
  1903:   DebugLoc DL = MI->getDebugLoc();
  1904:   Register SrcR = MI->getOperand(2).getReg();
  1905:   Register SrcLo = HRI.getSubReg(SrcR, Hexagon::vsub_lo);
  1906:   Register SrcHi = HRI.getSubReg(SrcR, Hexagon::vsub_hi);
  1907:   bool IsKill = MI->getOperand(2).isKill();
  1908:   int FI = MI->getOperand(0).getIndex();
  1909: 
  1910:   unsigned Size = HRI.getSpillSize(Hexagon::HvxVRRegClass);
  1911:   Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  1912:   Align HasAlign = MFI.getObjectAlign(FI);
  1913:   unsigned StoreOpc;
  1914: 
  1915:   // Store low part.
  1916:   if (LPR.contains(SrcLo)) {
  1917:     StoreOpc = NeedAlign <= HasAlign ? Hexagon::V6_vS32b_ai
  1918:                                      : Hexagon::V6_vS32Ub_ai;
  1919:     BuildMI(B, It, DL, HII.get(StoreOpc))
  1920:         .addFrameIndex(FI)
  1921:         .addImm(0)
  1922:         .addReg(SrcLo, getKillRegState(IsKill))
  1923:         .cloneMemRefs(*MI);
  1924:   }
  1925: 
  1926:   // Store high part.
  1927:   if (LPR.contains(SrcHi)) {
  1928:     StoreOpc = NeedAlign <= HasAlign ? Hexagon::V6_vS32b_ai
  1929:                                      : Hexagon::V6_vS32Ub_ai;
  1930:     BuildMI(B, It, DL, HII.get(StoreOpc))
  1931:         .addFrameIndex(FI)
  1932:         .addImm(Size)
  1933:         .addReg(SrcHi, getKillRegState(IsKill))
  1934:         .cloneMemRefs(*MI);
  1935:   }
  1936: 
  1937:   B.erase(It);
  1938:   return true;
  1939: }
  1940: 
  1941: bool HexagonFrameLowering::expandLoadVec2(MachineBasicBlock &B,
  1942:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1943:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1944:   MachineFunction &MF = *B.getParent();
  1945:   auto &MFI = MF.getFrameInfo();
  1946:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  1947:   MachineInstr *MI = &*It;
  1948:   if (!MI->getOperand(1).isFI())
  1949:     return false;
  1950: 
  1951:   DebugLoc DL = MI->getDebugLoc();
  1952:   Register DstR = MI->getOperand(0).getReg();
  1953:   Register DstHi = HRI.getSubReg(DstR, Hexagon::vsub_hi);
  1954:   Register DstLo = HRI.getSubReg(DstR, Hexagon::vsub_lo);
  1955:   int FI = MI->getOperand(1).getIndex();
  1956: 
  1957:   unsigned Size = HRI.getSpillSize(Hexagon::HvxVRRegClass);
  1958:   Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  1959:   Align HasAlign = MFI.getObjectAlign(FI);
  1960:   unsigned LoadOpc;
  1961: 
  1962:   // Load low part.
  1963:   LoadOpc = NeedAlign <= HasAlign ? Hexagon::V6_vL32b_ai
  1964:                                   : Hexagon::V6_vL32Ub_ai;
  1965:   BuildMI(B, It, DL, HII.get(LoadOpc), DstLo)
  1966:       .addFrameIndex(FI)
  1967:       .addImm(0)
  1968:       .cloneMemRefs(*MI);
  1969: 
  1970:   // Load high part.
  1971:   LoadOpc = NeedAlign <= HasAlign ? Hexagon::V6_vL32b_ai
  1972:                                   : Hexagon::V6_vL32Ub_ai;
  1973:   BuildMI(B, It, DL, HII.get(LoadOpc), DstHi)
  1974:       .addFrameIndex(FI)
  1975:       .addImm(Size)
  1976:       .cloneMemRefs(*MI);
  1977: 
  1978:   B.erase(It);
  1979:   return true;
  1980: }
  1981: 
  1982: bool HexagonFrameLowering::expandStoreVec(MachineBasicBlock &B,
  1983:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  1984:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  1985:   MachineFunction &MF = *B.getParent();
  1986:   auto &MFI = MF.getFrameInfo();
  1987:   MachineInstr *MI = &*It;
  1988:   if (!MI->getOperand(0).isFI())
  1989:     return false;
  1990: 
  1991:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  1992:   DebugLoc DL = MI->getDebugLoc();
  1993:   Register SrcR = MI->getOperand(2).getReg();
  1994:   bool IsKill = MI->getOperand(2).isKill();
  1995:   int FI = MI->getOperand(0).getIndex();
  1996: 
  1997:   Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  1998:   Align HasAlign = MFI.getObjectAlign(FI);
  1999:   unsigned StoreOpc = NeedAlign <= HasAlign ? Hexagon::V6_vS32b_ai
  2000:                                             : Hexagon::V6_vS32Ub_ai;
```
- EN: It declares or implements routines such as BuildMI, push_back, erase, HexagonFrameLowering::expandStoreVecPred, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFrameLowering, HexagonInstrInfo, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 BuildMI, push_back, erase, HexagonFrameLowering::expandStoreVecPred, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFrameLowering, HexagonInstrInfo, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:   BuildMI(B, It, DL, HII.get(StoreOpc))
  2002:       .addFrameIndex(FI)
  2003:       .addImm(0)
  2004:       .addReg(SrcR, getKillRegState(IsKill))
  2005:       .cloneMemRefs(*MI);
  2006: 
  2007:   B.erase(It);
  2008:   return true;
  2009: }
  2010: 
  2011: bool HexagonFrameLowering::expandLoadVec(MachineBasicBlock &B,
  2012:       MachineBasicBlock::iterator It, MachineRegisterInfo &MRI,
  2013:       const HexagonInstrInfo &HII, SmallVectorImpl<Register> &NewRegs) const {
  2014:   MachineFunction &MF = *B.getParent();
  2015:   auto &MFI = MF.getFrameInfo();
  2016:   MachineInstr *MI = &*It;
  2017:   if (!MI->getOperand(1).isFI())
  2018:     return false;
  2019: 
  2020:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  2021:   DebugLoc DL = MI->getDebugLoc();
  2022:   Register DstR = MI->getOperand(0).getReg();
  2023:   int FI = MI->getOperand(1).getIndex();
  2024: 
  2025:   Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  2026:   Align HasAlign = MFI.getObjectAlign(FI);
  2027:   unsigned LoadOpc = NeedAlign <= HasAlign ? Hexagon::V6_vL32b_ai
  2028:                                            : Hexagon::V6_vL32Ub_ai;
  2029:   BuildMI(B, It, DL, HII.get(LoadOpc), DstR)
  2030:       .addFrameIndex(FI)
  2031:       .addImm(0)
  2032:       .cloneMemRefs(*MI);
  2033: 
  2034:   B.erase(It);
  2035:   return true;
  2036: }
  2037: 
  2038: bool HexagonFrameLowering::expandSpillMacros(MachineFunction &MF,
  2039:       SmallVectorImpl<Register> &NewRegs) const {
  2040:   auto &HII = *MF.getSubtarget<HexagonSubtarget>().getInstrInfo();
  2041:   MachineRegisterInfo &MRI = MF.getRegInfo();
  2042:   bool Changed = false;
  2043: 
  2044:   for (auto &B : MF) {
  2045:     // Traverse the basic block.
  2046:     MachineBasicBlock::iterator NextI;
  2047:     for (auto I = B.begin(), E = B.end(); I != E; I = NextI) {
  2048:       MachineInstr *MI = &*I;
  2049:       NextI = std::next(I);
  2050:       unsigned Opc = MI->getOpcode();
  2051: 
  2052:       switch (Opc) {
  2053:         case TargetOpcode::COPY:
  2054:           Changed |= expandCopy(B, I, MRI, HII, NewRegs);
  2055:           break;
  2056:         case Hexagon::STriw_pred:
  2057:         case Hexagon::STriw_ctr:
  2058:           Changed |= expandStoreInt(B, I, MRI, HII, NewRegs);
  2059:           break;
  2060:         case Hexagon::LDriw_pred:
  2061:         case Hexagon::LDriw_ctr:
  2062:           Changed |= expandLoadInt(B, I, MRI, HII, NewRegs);
  2063:           break;
  2064:         case Hexagon::PS_vstorerq_ai:
  2065:           Changed |= expandStoreVecPred(B, I, MRI, HII, NewRegs);
  2066:           break;
  2067:         case Hexagon::PS_vloadrq_ai:
  2068:           Changed |= expandLoadVecPred(B, I, MRI, HII, NewRegs);
  2069:           break;
  2070:         case Hexagon::PS_vloadrw_ai:
  2071:           Changed |= expandLoadVec2(B, I, MRI, HII, NewRegs);
  2072:           break;
  2073:         case Hexagon::PS_vstorerw_ai:
  2074:           Changed |= expandStoreVec2(B, I, MRI, HII, NewRegs);
  2075:           break;
  2076:       }
  2077:     }
  2078:   }
  2079: 
  2080:   return Changed;
  2081: }
  2082: 
  2083: void HexagonFrameLowering::determineCalleeSaves(MachineFunction &MF,
  2084:                                                 BitVector &SavedRegs,
  2085:                                                 RegScavenger *RS) const {
  2086:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  2087: 
  2088:   SavedRegs.resize(HRI.getNumRegs());
  2089: 
  2090:   // If we have a function containing __builtin_eh_return we want to spill and
  2091:   // restore all callee saved registers. Pretend that they are used.
  2092:   if (MF.getInfo<HexagonMachineFunctionInfo>()->hasEHReturn())
  2093:     for (const MCPhysReg *R = HRI.getCalleeSavedRegs(&MF); *R; ++R)
  2094:       SavedRegs.set(*R);
  2095: 
  2096:   // Replace predicate register pseudo spill code.
  2097:   SmallVector<Register,8> NewRegs;
  2098:   expandSpillMacros(MF, NewRegs);
  2099:   if (OptimizeSpillSlots && !isOptNone(MF))
  2100:     optimizeSpillSlots(MF, NewRegs);
  2101: 
  2102:   // We need to reserve a spill slot if scavenging could potentially require
  2103:   // spilling a scavenged register.
  2104:   if (!NewRegs.empty() || mayOverflowFrameOffset(MF)) {
  2105:     MachineFrameInfo &MFI = MF.getFrameInfo();
  2106:     MachineRegisterInfo &MRI = MF.getRegInfo();
  2107:     SetVector<const TargetRegisterClass*> SpillRCs;
  2108:     // Reserve an int register in any case, because it could be used to hold
  2109:     // the stack offset in case it does not fit into a spill instruction.
  2110:     SpillRCs.insert(&Hexagon::IntRegsRegClass);
  2111: 
  2112:     for (Register VR : NewRegs)
  2113:       SpillRCs.insert(MRI.getRegClass(VR));
  2114: 
  2115:     for (const auto *RC : SpillRCs) {
  2116:       if (!needToReserveScavengingSpillSlots(MF, HRI, RC))
  2117:         continue;
  2118:       unsigned Num = 1;
  2119:       switch (RC->getID()) {
  2120:         case Hexagon::IntRegsRegClassID:
  2121:           Num = NumberScavengerSlots;
  2122:           break;
  2123:         case Hexagon::HvxQRRegClassID:
  2124:           Num = 2; // Vector predicate spills also need a vector register.
  2125:           break;
  2126:       }
  2127:       unsigned S = HRI.getSpillSize(*RC);
  2128:       Align A = HRI.getSpillAlign(*RC);
  2129:       for (unsigned i = 0; i < Num; i++) {
  2130:         int NewFI = MFI.CreateSpillStackObject(S, A);
  2131:         RS->addScavengingFrameIndex(NewFI);
  2132:       }
  2133:     }
  2134:   }
  2135: 
  2136:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
  2137: }
  2138: 
  2139: Register HexagonFrameLowering::findPhysReg(MachineFunction &MF,
  2140:       HexagonBlockRanges::IndexRange &FIR,
  2141:       HexagonBlockRanges::InstrIndexMap &IndexMap,
  2142:       HexagonBlockRanges::RegToRangeMap &DeadMap,
  2143:       const TargetRegisterClass *RC) const {
  2144:   auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
  2145:   auto &MRI = MF.getRegInfo();
  2146: 
  2147:   auto isDead = [&FIR,&DeadMap] (Register Reg) -> bool {
  2148:     auto F = DeadMap.find({Reg,0});
  2149:     if (F == DeadMap.end())
  2150:       return false;
  2151:     for (auto &DR : F->second)
  2152:       if (DR.contains(FIR))
  2153:         return true;
  2154:     return false;
  2155:   };
  2156: 
  2157:   for (Register Reg : RC->getRawAllocationOrder(MF)) {
  2158:     bool Dead = true;
  2159:     for (auto R : HexagonBlockRanges::expandToSubRegs({Reg,0}, MRI, HRI)) {
  2160:       if (isDead(R.Reg))
  2161:         continue;
  2162:       Dead = false;
  2163:       break;
  2164:     }
  2165:     if (Dead)
  2166:       return Reg;
  2167:   }
  2168:   return 0;
  2169: }
  2170: 
  2171: void HexagonFrameLowering::optimizeSpillSlots(MachineFunction &MF,
  2172:       SmallVectorImpl<Register> &VRegs) const {
  2173:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  2174:   auto &HII = *HST.getInstrInfo();
  2175:   auto &HRI = *HST.getRegisterInfo();
  2176:   auto &MRI = MF.getRegInfo();
  2177:   HexagonBlockRanges HBR(MF);
  2178: 
  2179:   using BlockIndexMap =
  2180:       std::map<MachineBasicBlock *, HexagonBlockRanges::InstrIndexMap>;
  2181:   using BlockRangeMap =
  2182:       std::map<MachineBasicBlock *, HexagonBlockRanges::RangeList>;
  2183:   using IndexType = HexagonBlockRanges::IndexType;
  2184: 
  2185:   struct SlotInfo {
  2186:     BlockRangeMap Map;
  2187:     unsigned Size = 0;
  2188:     const TargetRegisterClass *RC = nullptr;
  2189: 
  2190:     SlotInfo() = default;
  2191:   };
  2192: 
  2193:   BlockIndexMap BlockIndexes;
  2194:   SmallSet<int,4> BadFIs;
  2195:   std::map<int,SlotInfo> FIRangeMap;
  2196: 
  2197:   // Accumulate register classes: get a common class for a pre-existing
  2198:   // class HaveRC and a new class NewRC. Return nullptr if a common class
  2199:   // cannot be found, otherwise return the resulting class. If HaveRC is
  2200:   // nullptr, assume that it is still unset.
```
- EN: It declares types such as SlotInfo, which carry the state or API of this component. It declares or implements routines such as BuildMI, erase, HexagonFrameLowering::expandLoadVec, getParent, ... (36 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFrameLowering, HexagonInstrInfo, HexagonSubtarget, HexagonMachineFunctionInfo, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明了 SlotInfo 等类型，用来承载该组件的状态或接口。 这里声明或实现了 BuildMI, erase, HexagonFrameLowering::expandLoadVec, getParent, ... (36 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFrameLowering, HexagonInstrInfo, HexagonSubtarget, HexagonMachineFunctionInfo, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:   auto getCommonRC =
  2202:       [](const TargetRegisterClass *HaveRC,
  2203:          const TargetRegisterClass *NewRC) -> const TargetRegisterClass * {
  2204:     if (HaveRC == nullptr || HaveRC == NewRC)
  2205:       return NewRC;
  2206:     // Different classes, both non-null. Pick the more general one.
  2207:     if (HaveRC->hasSubClassEq(NewRC))
  2208:       return HaveRC;
  2209:     if (NewRC->hasSubClassEq(HaveRC))
  2210:       return NewRC;
  2211:     return nullptr;
  2212:   };
  2213: 
  2214:   // Scan all blocks in the function. Check all occurrences of frame indexes,
  2215:   // and collect relevant information.
  2216:   for (auto &B : MF) {
  2217:     std::map<int,IndexType> LastStore, LastLoad;
  2218:     auto P = BlockIndexes.emplace(&B, HexagonBlockRanges::InstrIndexMap(B));
  2219:     auto &IndexMap = P.first->second;
  2220:     LLVM_DEBUG(dbgs() << "Index map for " << printMBBReference(B) << "\n"
  2221:                       << IndexMap << '\n');
  2222: 
  2223:     for (auto &In : B) {
  2224:       int LFI, SFI;
  2225:       bool Load = HII.isLoadFromStackSlot(In, LFI) && !HII.isPredicated(In);
  2226:       bool Store = HII.isStoreToStackSlot(In, SFI) && !HII.isPredicated(In);
  2227:       if (Load && Store) {
  2228:         // If it's both a load and a store, then we won't handle it.
  2229:         BadFIs.insert(LFI);
  2230:         BadFIs.insert(SFI);
  2231:         continue;
  2232:       }
  2233:       // Check for register classes of the register used as the source for
  2234:       // the store, and the register used as the destination for the load.
  2235:       // Also, only accept base+imm_offset addressing modes. Other addressing
  2236:       // modes can have side-effects (post-increments, etc.). For stack
  2237:       // slots they are very unlikely, so there is not much loss due to
  2238:       // this restriction.
  2239:       if (Load || Store) {
  2240:         int TFI = Load ? LFI : SFI;
  2241:         unsigned AM = HII.getAddrMode(In);
  2242:         SlotInfo &SI = FIRangeMap[TFI];
  2243:         bool Bad = (AM != HexagonII::BaseImmOffset);
  2244:         if (!Bad) {
  2245:           // If the addressing mode is ok, check the register class.
  2246:           unsigned OpNum = Load ? 0 : 2;
  2247:           auto *RC = HII.getRegClass(In.getDesc(), OpNum);
  2248:           RC = getCommonRC(SI.RC, RC);
  2249:           if (RC == nullptr)
  2250:             Bad = true;
  2251:           else
  2252:             SI.RC = RC;
  2253:         }
  2254:         if (!Bad) {
  2255:           // Check sizes.
  2256:           unsigned S = HII.getMemAccessSize(In);
  2257:           if (SI.Size != 0 && SI.Size != S)
  2258:             Bad = true;
  2259:           else
  2260:             SI.Size = S;
  2261:         }
  2262:         if (!Bad) {
  2263:           for (auto *Mo : In.memoperands()) {
  2264:             if (!Mo->isVolatile() && !Mo->isAtomic())
  2265:               continue;
  2266:             Bad = true;
  2267:             break;
  2268:           }
  2269:         }
  2270:         if (Bad)
  2271:           BadFIs.insert(TFI);
  2272:       }
  2273: 
  2274:       // Locate uses of frame indices.
  2275:       for (unsigned i = 0, n = In.getNumOperands(); i < n; ++i) {
  2276:         const MachineOperand &Op = In.getOperand(i);
  2277:         if (!Op.isFI())
  2278:           continue;
  2279:         int FI = Op.getIndex();
  2280:         // Make sure that the following operand is an immediate and that
  2281:         // it is 0. This is the offset in the stack object.
  2282:         if (i+1 >= n || !In.getOperand(i+1).isImm() ||
  2283:             In.getOperand(i+1).getImm() != 0)
  2284:           BadFIs.insert(FI);
  2285:         if (BadFIs.count(FI))
  2286:           continue;
  2287: 
  2288:         IndexType Index = IndexMap.getIndex(&In);
  2289:         auto &LS = LastStore[FI];
  2290:         auto &LL = LastLoad[FI];
  2291:         if (Load) {
  2292:           if (LS == IndexType::None)
  2293:             LS = IndexType::Entry;
  2294:           LL = Index;
  2295:         } else if (Store) {
  2296:           HexagonBlockRanges::RangeList &RL = FIRangeMap[FI].Map[&B];
  2297:           if (LS != IndexType::None)
  2298:             RL.add(LS, LL, false, false);
  2299:           else if (LL != IndexType::None)
  2300:             RL.add(IndexType::Entry, LL, false, false);
  2301:           LL = IndexType::None;
  2302:           LS = Index;
  2303:         } else {
  2304:           BadFIs.insert(FI);
  2305:         }
  2306:       }
  2307:     }
  2308: 
  2309:     for (auto &I : LastLoad) {
  2310:       IndexType LL = I.second;
  2311:       if (LL == IndexType::None)
  2312:         continue;
  2313:       auto &RL = FIRangeMap[I.first].Map[&B];
  2314:       IndexType &LS = LastStore[I.first];
  2315:       if (LS != IndexType::None)
  2316:         RL.add(LS, LL, false, false);
  2317:       else
  2318:         RL.add(IndexType::Entry, LL, false, false);
  2319:       LS = IndexType::None;
  2320:     }
  2321:     for (auto &I : LastStore) {
  2322:       IndexType LS = I.second;
  2323:       if (LS == IndexType::None)
  2324:         continue;
  2325:       auto &RL = FIRangeMap[I.first].Map[&B];
  2326:       RL.add(LS, IndexType::None, false, false);
  2327:     }
  2328:   }
  2329: 
  2330:   LLVM_DEBUG({
  2331:     for (auto &P : FIRangeMap) {
  2332:       dbgs() << "fi#" << P.first;
  2333:       if (BadFIs.count(P.first))
  2334:         dbgs() << " (bad)";
  2335:       dbgs() << "  RC: ";
  2336:       if (P.second.RC != nullptr)
  2337:         dbgs() << HRI.getRegClassName(P.second.RC) << '\n';
  2338:       else
  2339:         dbgs() << "<null>\n";
  2340:       for (auto &R : P.second.Map)
  2341:         dbgs() << "  " << printMBBReference(*R.first) << " { " << R.second
  2342:                << "}\n";
  2343:     }
  2344:   });
  2345: 
  2346:   // When a slot is loaded from in a block without being stored to in the
  2347:   // same block, it is live-on-entry to this block. To avoid CFG analysis,
  2348:   // consider this slot to be live-on-exit from all blocks.
  2349:   SmallSet<int,4> LoxFIs;
  2350: 
  2351:   std::map<MachineBasicBlock*,std::vector<int>> BlockFIMap;
  2352: 
  2353:   for (auto &P : FIRangeMap) {
  2354:     // P = pair(FI, map: BB->RangeList)
  2355:     if (BadFIs.count(P.first))
  2356:       continue;
  2357:     for (auto &B : MF) {
  2358:       auto F = P.second.Map.find(&B);
  2359:       // F = pair(BB, RangeList)
  2360:       if (F == P.second.Map.end() || F->second.empty())
  2361:         continue;
  2362:       HexagonBlockRanges::IndexRange &IR = F->second.front();
  2363:       if (IR.start() == IndexType::Entry)
  2364:         LoxFIs.insert(P.first);
  2365:       BlockFIMap[&B].push_back(P.first);
  2366:     }
  2367:   }
  2368: 
  2369:   LLVM_DEBUG({
  2370:     dbgs() << "Block-to-FI map (* -- live-on-exit):\n";
  2371:     for (auto &P : BlockFIMap) {
  2372:       auto &FIs = P.second;
  2373:       if (FIs.empty())
  2374:         continue;
  2375:       dbgs() << "  " << printMBBReference(*P.first) << ": {";
  2376:       for (auto I : FIs) {
  2377:         dbgs() << " fi#" << I;
  2378:         if (LoxFIs.count(I))
  2379:           dbgs() << '*';
  2380:       }
  2381:       dbgs() << " }\n";
  2382:     }
  2383:   });
  2384: 
  2385: #ifndef NDEBUG
  2386:   bool HasOptLimit = SpillOptMax.getPosition();
  2387: #endif
  2388: 
  2389:   // eliminate loads, when all loads eliminated, eliminate all stores.
  2390:   for (auto &B : MF) {
  2391:     auto F = BlockIndexes.find(&B);
  2392:     assert(F != BlockIndexes.end());
  2393:     HexagonBlockRanges::InstrIndexMap &IM = F->second;
  2394:     HexagonBlockRanges::RegToRangeMap LM = HBR.computeLiveMap(IM);
  2395:     HexagonBlockRanges::RegToRangeMap DM = HBR.computeDeadMap(IM, LM);
  2396:     LLVM_DEBUG(dbgs() << printMBBReference(B) << " dead map\n"
  2397:                       << HexagonBlockRanges::PrintRangeMap(DM, HRI));
  2398: 
  2399:     for (auto FI : BlockFIMap[&B]) {
  2400:       if (BadFIs.count(FI))
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as emplace, LLVM_DEBUG, isLoadFromStackSlot, isStoreToStackSlot, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 emplace, LLVM_DEBUG, isLoadFromStackSlot, isStoreToStackSlot, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:         continue;
  2402:       LLVM_DEBUG(dbgs() << "Working on fi#" << FI << '\n');
  2403:       HexagonBlockRanges::RangeList &RL = FIRangeMap[FI].Map[&B];
  2404:       for (auto &Range : RL) {
  2405:         LLVM_DEBUG(dbgs() << "--Examining range:" << RL << '\n');
  2406:         if (!IndexType::isInstr(Range.start()) ||
  2407:             !IndexType::isInstr(Range.end()))
  2408:           continue;
  2409:         MachineInstr &SI = *IM.getInstr(Range.start());
  2410:         MachineInstr &EI = *IM.getInstr(Range.end());
  2411:         assert(SI.mayStore() && "Unexpected start instruction");
  2412:         assert(EI.mayLoad() && "Unexpected end instruction");
  2413:         MachineOperand &SrcOp = SI.getOperand(2);
  2414: 
  2415:         HexagonBlockRanges::RegisterRef SrcRR = { SrcOp.getReg(),
  2416:                                                   SrcOp.getSubReg() };
  2417:         auto *RC = HII.getRegClass(SI.getDesc(), 2);
  2418:         // The this-> is needed to unconfuse MSVC.
  2419:         Register FoundR = this->findPhysReg(MF, Range, IM, DM, RC);
  2420:         LLVM_DEBUG(dbgs() << "Replacement reg:" << printReg(FoundR, &HRI)
  2421:                           << '\n');
  2422:         if (FoundR == 0)
  2423:           continue;
  2424: #ifndef NDEBUG
  2425:         if (HasOptLimit) {
  2426:           if (SpillOptCount >= SpillOptMax)
  2427:             return;
  2428:           SpillOptCount++;
  2429:         }
  2430: #endif
  2431: 
  2432:         // Generate the copy-in: "FoundR = COPY SrcR" at the store location.
  2433:         MachineBasicBlock::iterator StartIt = SI.getIterator(), NextIt;
  2434:         MachineInstr *CopyIn = nullptr;
  2435:         if (SrcRR.Reg != FoundR || SrcRR.Sub != 0) {
  2436:           const DebugLoc &DL = SI.getDebugLoc();
  2437:           CopyIn = BuildMI(B, StartIt, DL, HII.get(TargetOpcode::COPY), FoundR)
  2438:                        .add(SrcOp);
  2439:         }
  2440: 
  2441:         ++StartIt;
  2442:         // Check if this is a last store and the FI is live-on-exit.
  2443:         if (LoxFIs.count(FI) && (&Range == &RL.back())) {
  2444:           // Update store's source register.
  2445:           if (unsigned SR = SrcOp.getSubReg())
  2446:             SrcOp.setReg(HRI.getSubReg(FoundR, SR));
  2447:           else
  2448:             SrcOp.setReg(FoundR);
  2449:           SrcOp.setSubReg(0);
  2450:           // We are keeping this register live.
  2451:           SrcOp.setIsKill(false);
  2452:         } else {
  2453:           B.erase(&SI);
  2454:           IM.replaceInstr(&SI, CopyIn);
  2455:         }
  2456: 
  2457:         auto EndIt = std::next(EI.getIterator());
  2458:         for (auto It = StartIt; It != EndIt; It = NextIt) {
  2459:           MachineInstr &MI = *It;
  2460:           NextIt = std::next(It);
  2461:           int TFI;
  2462:           if (!HII.isLoadFromStackSlot(MI, TFI) || TFI != FI)
  2463:             continue;
  2464:           Register DstR = MI.getOperand(0).getReg();
  2465:           assert(MI.getOperand(0).getSubReg() == 0);
  2466:           MachineInstr *CopyOut = nullptr;
  2467:           if (DstR != FoundR) {
  2468:             DebugLoc DL = MI.getDebugLoc();
  2469:             unsigned MemSize = HII.getMemAccessSize(MI);
  2470:             assert(HII.getAddrMode(MI) == HexagonII::BaseImmOffset);
  2471:             unsigned CopyOpc = TargetOpcode::COPY;
  2472:             if (HII.isSignExtendingLoad(MI))
  2473:               CopyOpc = (MemSize == 1) ? Hexagon::A2_sxtb : Hexagon::A2_sxth;
  2474:             else if (HII.isZeroExtendingLoad(MI))
  2475:               CopyOpc = (MemSize == 1) ? Hexagon::A2_zxtb : Hexagon::A2_zxth;
  2476:             CopyOut = BuildMI(B, It, DL, HII.get(CopyOpc), DstR)
  2477:                         .addReg(FoundR, getKillRegState(&MI == &EI));
  2478:           }
  2479:           IM.replaceInstr(&MI, CopyOut);
  2480:           B.erase(It);
  2481:         }
  2482: 
  2483:         // Update the dead map.
  2484:         HexagonBlockRanges::RegisterRef FoundRR = { FoundR, 0 };
  2485:         for (auto RR : HexagonBlockRanges::expandToSubRegs(FoundRR, MRI, HRI))
  2486:           DM[RR].subtract(Range);
  2487:       } // for Range in range list
  2488:     }
  2489:   }
  2490: }
  2491: 
  2492: void HexagonFrameLowering::expandAlloca(MachineInstr *AI,
  2493:       const HexagonInstrInfo &HII, Register SP, unsigned CF) const {
  2494:   MachineBasicBlock &MB = *AI->getParent();
  2495:   DebugLoc DL = AI->getDebugLoc();
  2496:   unsigned A = AI->getOperand(2).getImm();
  2497: 
  2498:   // Have
  2499:   //    Rd  = alloca Rs, #A
  2500:   //
  2501:   // If Rs and Rd are different registers, use this sequence:
  2502:   //    Rd  = sub(r29, Rs)
  2503:   //    r29 = sub(r29, Rs)
  2504:   //    Rd  = and(Rd, #-A)    ; if necessary
  2505:   //    r29 = and(r29, #-A)   ; if necessary
  2506:   //    Rd  = add(Rd, #CF)    ; CF size aligned to at most A
  2507:   // otherwise, do
  2508:   //    Rd  = sub(r29, Rs)
  2509:   //    Rd  = and(Rd, #-A)    ; if necessary
  2510:   //    r29 = Rd
  2511:   //    Rd  = add(Rd, #CF)    ; CF size aligned to at most A
  2512: 
  2513:   MachineOperand &RdOp = AI->getOperand(0);
  2514:   MachineOperand &RsOp = AI->getOperand(1);
  2515:   Register Rd = RdOp.getReg(), Rs = RsOp.getReg();
  2516: 
  2517:   // Rd = sub(r29, Rs)
  2518:   BuildMI(MB, AI, DL, HII.get(Hexagon::A2_sub), Rd)
  2519:       .addReg(SP)
  2520:       .addReg(Rs);
  2521:   if (Rs != Rd) {
  2522:     // r29 = sub(r29, Rs)
  2523:     BuildMI(MB, AI, DL, HII.get(Hexagon::A2_sub), SP)
  2524:         .addReg(SP)
  2525:         .addReg(Rs);
  2526:   }
  2527:   if (A > 8) {
  2528:     // Rd  = and(Rd, #-A)
  2529:     BuildMI(MB, AI, DL, HII.get(Hexagon::A2_andir), Rd)
  2530:         .addReg(Rd)
  2531:         .addImm(-int64_t(A));
  2532:     if (Rs != Rd)
  2533:       BuildMI(MB, AI, DL, HII.get(Hexagon::A2_andir), SP)
  2534:           .addReg(SP)
  2535:           .addImm(-int64_t(A));
  2536:   }
  2537:   if (Rs == Rd) {
  2538:     // r29 = Rd
  2539:     BuildMI(MB, AI, DL, HII.get(TargetOpcode::COPY), SP)
  2540:         .addReg(Rd);
  2541:   }
  2542:   if (CF > 0) {
  2543:     // Rd = add(Rd, #CF)
  2544:     BuildMI(MB, AI, DL, HII.get(Hexagon::A2_addi), Rd)
  2545:         .addReg(Rd)
  2546:         .addImm(CF);
  2547:   }
  2548: }
  2549: 
  2550: bool HexagonFrameLowering::needsAligna(const MachineFunction &MF) const {
  2551:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  2552:   if (!MFI.hasVarSizedObjects())
  2553:     return false;
  2554:   // Do not check for max stack object alignment here, because the stack
  2555:   // may not be complete yet. Assume that we will need PS_aligna if there
  2556:   // are variable-sized objects.
  2557:   return true;
  2558: }
  2559: 
  2560: const MachineInstr *HexagonFrameLowering::getAlignaInstr(
  2561:       const MachineFunction &MF) const {
  2562:   for (auto &B : MF)
  2563:     for (auto &I : B)
  2564:       if (I.getOpcode() == Hexagon::PS_aligna)
  2565:         return &I;
  2566:   return nullptr;
  2567: }
  2568: 
  2569: /// Adds all callee-saved registers as implicit uses or defs to the
  2570: /// instruction.
  2571: void HexagonFrameLowering::addCalleeSaveRegistersAsImpOperand(MachineInstr *MI,
  2572:       const CSIVect &CSI, bool IsDef, bool IsKill) const {
  2573:   // Add the callee-saved registers as implicit uses.
  2574:   for (auto &R : CSI)
  2575:     MI->addOperand(MachineOperand::CreateReg(R.getReg(), IsDef, true, IsKill));
  2576: }
  2577: 
  2578: /// Determine whether the callee-saved register saves and restores should
  2579: /// be generated via inline code. If this function returns "true", inline
  2580: /// code will be generated. If this function returns "false", additional
  2581: /// checks are performed, which may still lead to the inline code.
  2582: bool HexagonFrameLowering::shouldInlineCSR(const MachineFunction &MF,
  2583:       const CSIVect &CSI) const {
  2584:   if (MF.getSubtarget<HexagonSubtarget>().isEnvironmentMusl())
  2585:     return true;
  2586:   if (MF.getInfo<HexagonMachineFunctionInfo>()->hasEHReturn())
  2587:     return true;
  2588:   if (!hasFP(MF))
  2589:     return true;
  2590:   if (!isOptSize(MF) && !isMinSize(MF))
  2591:     if (MF.getTarget().getOptLevel() > CodeGenOptLevel::Default)
  2592:       return true;
  2593: 
  2594:   // Check if CSI only has double registers, and if the registers form
  2595:   // a contiguous block starting from D8.
  2596:   BitVector Regs(Hexagon::NUM_TARGET_REGS);
  2597:   for (const CalleeSavedInfo &I : CSI) {
  2598:     MCRegister R = I.getReg();
  2599:     if (!Hexagon::DoubleRegsRegClass.contains(R))
  2600:       return true;
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as LLVM_DEBUG, getInstr, assert, getOperand, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 LLVM_DEBUG, getInstr, assert, getOperand, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2601-2758 / 第 2601-2758 行

```cpp
  2601:     Regs[R] = true;
  2602:   }
  2603:   int F = Regs.find_first();
  2604:   if (F != Hexagon::D8)
  2605:     return true;
  2606:   while (F >= 0) {
  2607:     int N = Regs.find_next(F);
  2608:     if (N >= 0 && N != F+1)
  2609:       return true;
  2610:     F = N;
  2611:   }
  2612: 
  2613:   return false;
  2614: }
  2615: 
  2616: bool HexagonFrameLowering::useSpillFunction(const MachineFunction &MF,
  2617:       const CSIVect &CSI) const {
  2618:   if (shouldInlineCSR(MF, CSI))
  2619:     return false;
  2620:   unsigned NumCSI = CSI.size();
  2621:   if (NumCSI <= 1)
  2622:     return false;
  2623: 
  2624:   unsigned Threshold = isOptSize(MF) ? SpillFuncThresholdOs
  2625:                                      : SpillFuncThreshold;
  2626:   return Threshold < NumCSI;
  2627: }
  2628: 
  2629: bool HexagonFrameLowering::useRestoreFunction(const MachineFunction &MF,
  2630:       const CSIVect &CSI) const {
  2631:   if (shouldInlineCSR(MF, CSI))
  2632:     return false;
  2633:   // The restore functions do a bit more than just restoring registers.
  2634:   // The non-returning versions will go back directly to the caller's
  2635:   // caller, others will clean up the stack frame in preparation for
  2636:   // a tail call. Using them can still save code size even if only one
  2637:   // register is getting restores. Make the decision based on -Oz:
  2638:   // using -Os will use inline restore for a single register.
  2639:   if (isMinSize(MF))
  2640:     return true;
  2641:   unsigned NumCSI = CSI.size();
  2642:   if (NumCSI <= 1)
  2643:     return false;
  2644: 
  2645:   unsigned Threshold = isOptSize(MF) ? SpillFuncThresholdOs-1
  2646:                                      : SpillFuncThreshold;
  2647:   return Threshold < NumCSI;
  2648: }
  2649: 
  2650: bool HexagonFrameLowering::mayOverflowFrameOffset(MachineFunction &MF) const {
  2651:   unsigned StackSize = MF.getFrameInfo().estimateStackSize(MF);
  2652:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
  2653:   // A fairly simplistic guess as to whether a potential load/store to a
  2654:   // stack location could require an extra register.
  2655:   if (HST.useHVXOps() && StackSize > 256)
  2656:     return true;
  2657: 
  2658:   // Check if the function has store-immediate instructions that access
  2659:   // the stack. Since the offset field is not extendable, if the stack
  2660:   // size exceeds the offset limit (6 bits, shifted), the stores will
  2661:   // require a new base register.
  2662:   bool HasImmStack = false;
  2663:   unsigned MinLS = ~0u;   // Log_2 of the memory access size.
  2664: 
  2665:   for (const MachineBasicBlock &B : MF) {
  2666:     for (const MachineInstr &MI : B) {
  2667:       unsigned LS = 0;
  2668:       switch (MI.getOpcode()) {
  2669:         case Hexagon::S4_storeirit_io:
  2670:         case Hexagon::S4_storeirif_io:
  2671:         case Hexagon::S4_storeiri_io:
  2672:           ++LS;
  2673:           [[fallthrough]];
  2674:         case Hexagon::S4_storeirht_io:
  2675:         case Hexagon::S4_storeirhf_io:
  2676:         case Hexagon::S4_storeirh_io:
  2677:           ++LS;
  2678:           [[fallthrough]];
  2679:         case Hexagon::S4_storeirbt_io:
  2680:         case Hexagon::S4_storeirbf_io:
  2681:         case Hexagon::S4_storeirb_io:
  2682:           if (MI.getOperand(0).isFI())
  2683:             HasImmStack = true;
  2684:           MinLS = std::min(MinLS, LS);
  2685:           break;
  2686:       }
  2687:     }
  2688:   }
  2689: 
  2690:   if (HasImmStack)
  2691:     return !isUInt<6>(StackSize >> MinLS);
  2692: 
  2693:   return false;
  2694: }
  2695: 
  2696: namespace {
  2697: // Struct used by orderFrameObjects to help sort the stack objects.
  2698: struct HexagonFrameSortingObject {
  2699:   bool IsValid = false;
  2700:   unsigned Index = 0; // Index of Object into MFI list.
  2701:   unsigned Size = 0;
  2702:   Align ObjectAlignment = Align(1); // Alignment of Object in bytes.
  2703: };
  2704: 
  2705: struct HexagonFrameSortingComparator {
  2706:   inline bool operator()(const HexagonFrameSortingObject &A,
  2707:                          const HexagonFrameSortingObject &B) const {
  2708:     return std::make_tuple(!A.IsValid, A.ObjectAlignment, A.Size) <
  2709:            std::make_tuple(!B.IsValid, B.ObjectAlignment, B.Size);
  2710:   }
  2711: };
  2712: } // namespace
  2713: 
  2714: // Sort objects on the stack by alignment value and then by size to minimize
  2715: // padding.
  2716: void HexagonFrameLowering::orderFrameObjects(
  2717:     const MachineFunction &MF, SmallVectorImpl<int> &ObjectsToAllocate) const {
  2718: 
  2719:   if (ObjectsToAllocate.empty())
  2720:     return;
  2721: 
  2722:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  2723:   int NObjects = ObjectsToAllocate.size();
  2724: 
  2725:   // Create an array of all MFI objects.
  2726:   SmallVector<HexagonFrameSortingObject> SortingObjects(
  2727:       MFI.getObjectIndexEnd());
  2728: 
  2729:   for (int i = 0, j = 0, e = MFI.getObjectIndexEnd(); i < e && j != NObjects;
  2730:        ++i) {
  2731:     if (i != ObjectsToAllocate[j])
  2732:       continue;
  2733:     j++;
  2734: 
  2735:     // A variable size object has size equal to 0. Since Hexagon sets
  2736:     // getUseLocalStackAllocationBlock() to true, a local block is allocated
  2737:     // earlier. This case is not handled here for now.
  2738:     int Size = MFI.getObjectSize(i);
  2739:     if (Size == 0)
  2740:       return;
  2741: 
  2742:     SortingObjects[i].IsValid = true;
  2743:     SortingObjects[i].Index = i;
  2744:     SortingObjects[i].Size = Size;
  2745:     SortingObjects[i].ObjectAlignment = MFI.getObjectAlign(i);
  2746:   }
  2747: 
  2748:   // Sort objects by alignment and then by size.
  2749:   llvm::stable_sort(SortingObjects, HexagonFrameSortingComparator());
  2750: 
  2751:   // Modify the original list to represent the final order.
  2752:   int i = NObjects;
  2753:   for (auto &Obj : SortingObjects) {
  2754:     if (i == 0)
  2755:       break;
  2756:     ObjectsToAllocate[--i] = Obj.Index;
  2757:   }
  2758: }
```
- EN: It declares types such as HexagonFrameSortingObject, HexagonFrameSortingComparator, which carry the state or API of this component. It declares or implements routines such as find_first, find_next, HexagonFrameLowering::useSpillFunction, size, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFrameLowering, HexagonSubtarget, HexagonFrameSortingObject, HexagonFrameSortingComparator, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonFrameSortingObject, HexagonFrameSortingComparator 等类型，用来承载该组件的状态或接口。 这里声明或实现了 find_first, find_next, HexagonFrameLowering::useSpillFunction, size, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFrameLowering, HexagonSubtarget, HexagonFrameSortingObject, HexagonFrameSortingComparator，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonFrameLowering.h, HexagonBlockRanges.h, HexagonInstrInfo.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, HexagonTargetMachine.h, MCTargetDesc/HexagonBaseInfo.h, llvm/ADT/BitVector.h, llvm/ADT/DenseMap.h, ... (54 total)`
- Hexagon symbols / Hexagon 符号: `HexagonFrameLowering, HexagonBlockRanges, HexagonInstrInfo, HexagonMachineFunctionInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonTargetMachine, HexagonBaseInfo, HexagonCallFrameInformation, Hexagon_TODO, ... (14 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
