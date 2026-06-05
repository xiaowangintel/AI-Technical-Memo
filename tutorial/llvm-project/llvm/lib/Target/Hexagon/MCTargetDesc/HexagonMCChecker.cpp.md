# HexagonMCChecker.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCChecker.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon MC-layer target description glue.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===----- HexagonMCChecker.cpp - Instruction bundle checking -------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements the checking of insns inside a bundle according to the
    10: // packet constraint rules of the Hexagon ISA.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "MCTargetDesc/HexagonMCChecker.h"
    15: #include "MCTargetDesc/HexagonBaseInfo.h"
    16: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    17: #include "MCTargetDesc/HexagonMCShuffler.h"
    18: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    19: 
    20: #include "llvm/ADT/Twine.h"
    21: #include "llvm/MC/MCContext.h"
    22: #include "llvm/MC/MCInst.h"
    23: #include "llvm/MC/MCInstrDesc.h"
    24: #include "llvm/MC/MCRegisterInfo.h"
    25: #include "llvm/MC/MCSubtargetInfo.h"
    26: #include "llvm/Support/CommandLine.h"
    27: #include "llvm/Support/SourceMgr.h"
    28: #include <cassert>
    29: 
    30: using namespace llvm;
    31: 
    32: static cl::opt<bool>
    33:     RelaxNVChecks("relax-nv-checks", cl::Hidden,
    34:                   cl::desc("Relax checks of new-value validity"));
    35: 
    36: const HexagonMCChecker::PredSense
    37:     HexagonMCChecker::Unconditional(Hexagon::NoRegister, false);
    38: 
    39: void HexagonMCChecker::init() {
    40:   // Initialize read-only registers set.
    41:   ReadOnly.insert(Hexagon::PC);
    42:   ReadOnly.insert(Hexagon::C9_8);
    43: 
    44:   // Figure out the loop-registers definitions.
    45:   if (HexagonMCInstrInfo::isInnerLoop(MCB)) {
    46:     Defs[Hexagon::SA0].insert(Unconditional); // FIXME: define or change SA0?
    47:     Defs[Hexagon::LC0].insert(Unconditional);
    48:   }
    49:   if (HexagonMCInstrInfo::isOuterLoop(MCB)) {
    50:     Defs[Hexagon::SA1].insert(Unconditional); // FIXME: define or change SA0?
    51:     Defs[Hexagon::LC1].insert(Unconditional);
    52:   }
    53: 
    54:   if (HexagonMCInstrInfo::isBundle(MCB))
    55:     // Unfurl a bundle.
    56:     for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCB)) {
    57:       MCInst const &Inst = *I.getInst();
    58:       if (HexagonMCInstrInfo::isDuplex(MCII, Inst)) {
    59:         init(*Inst.getOperand(0).getInst());
    60:         init(*Inst.getOperand(1).getInst());
    61:       } else
    62:         init(Inst);
    63:     }
    64:   else
    65:     init(MCB);
    66: }
    67: 
    68: void HexagonMCChecker::initReg(MCInst const &MCI, MCRegister R,
    69:                                MCRegister &PredReg, bool &isTrue) {
    70:   if (HexagonMCInstrInfo::isPredicated(MCII, MCI) &&
    71:       HexagonMCInstrInfo::isPredReg(RI, R)) {
    72:     // Note an used predicate register.
    73:     PredReg = R;
    74:     isTrue = HexagonMCInstrInfo::isPredicatedTrue(MCII, MCI);
    75: 
    76:     // Note use of new predicate register.
    77:     if (HexagonMCInstrInfo::isPredicatedNew(MCII, MCI))
    78:       NewPreds.insert(PredReg);
    79:   } else
    80:     // Note register use.  Super-registers are not tracked directly,
    81:     // but their components.
    82:     for (MCRegAliasIterator SRI(R, &RI, RI.subregs(R).empty()); SRI.isValid();
    83:          ++SRI)
    84:       if (RI.subregs(*SRI).empty())
    85:         // Skip super-registers used indirectly.
    86:         Uses.insert(*SRI);
    87: 
    88:   if (HexagonMCInstrInfo::IsReverseVecRegPair(R))
    89:     ReversePairs.insert(R);
    90: }
    91: 
    92: void HexagonMCChecker::init(MCInst const &MCI) {
    93:   const MCInstrDesc &MCID = HexagonMCInstrInfo::getDesc(MCII, MCI);
    94:   MCRegister PredReg;
    95:   bool isTrue = false;
    96: 
    97:   // Get used registers.
    98:   for (unsigned i = MCID.getNumDefs(); i < MCID.getNumOperands(); ++i)
    99:     if (MCI.getOperand(i).isReg())
   100:       initReg(MCI, MCI.getOperand(i).getReg(), PredReg, isTrue);
```
- EN: It imports headers such as MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCShuffler.h, ... (14 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as RelaxNVChecks, HexagonMCChecker::Unconditional, HexagonMCChecker::init, insert, ... (11 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里引入了 MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCShuffler.h, ... (14 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 RelaxNVChecks, HexagonMCChecker::Unconditional, HexagonMCChecker::init, insert, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 101-200 / 第 101-200 行

```cpp
   101:   for (MCPhysReg ImpUse : MCID.implicit_uses())
   102:     initReg(MCI, ImpUse, PredReg, isTrue);
   103: 
   104:   const bool IgnoreTmpDst = (HexagonMCInstrInfo::hasTmpDst(MCII, MCI) ||
   105:                              HexagonMCInstrInfo::hasHvxTmp(MCII, MCI)) &&
   106:                             STI.hasFeature(Hexagon::ArchV69);
   107: 
   108:   // Get implicit register definitions.
   109:   for (MCPhysReg R : MCID.implicit_defs()) {
   110:     if (Hexagon::R31 != R && MCID.isCall())
   111:       // Any register other than the LR and the PC are actually volatile ones
   112:       // as defined by the ABI, not modified implicitly by the call insn.
   113:       continue;
   114:     if (Hexagon::PC == R)
   115:       // Branches are the only insns that can change the PC,
   116:       // otherwise a read-only register.
   117:       continue;
   118: 
   119:     if (Hexagon::USR_OVF == R)
   120:       // Many insns change the USR implicitly, but only one or another flag.
   121:       // The instruction table models the USR.OVF flag, which can be
   122:       // implicitly modified more than once, but cannot be modified in the
   123:       // same packet with an instruction that modifies is explicitly. Deal
   124:       // with such situations individually.
   125:       SoftDefs.insert(R);
   126:     else if (HexagonMCInstrInfo::isPredReg(RI, R) &&
   127:              HexagonMCInstrInfo::isPredicateLate(MCII, MCI))
   128:       // Include implicit late predicates.
   129:       LatePreds.insert(R);
   130:     else if (!IgnoreTmpDst)
   131:       Defs[R].insert(PredSense(PredReg, isTrue));
   132:   }
   133: 
   134:   // Figure out explicit register definitions.
   135:   for (unsigned i = 0; i < MCID.getNumDefs(); ++i) {
   136:     MCRegister R = MCI.getOperand(i).getReg(), S = MCRegister();
   137:     // USR has subregisters (while C8 does not for technical reasons), so
   138:     // reset R to USR, since we know how to handle multiple defs of USR,
   139:     // taking into account its subregisters.
   140:     if (R == Hexagon::C8)
   141:       R = Hexagon::USR;
   142: 
   143:     if (HexagonMCInstrInfo::IsReverseVecRegPair(R))
   144:       ReversePairs.insert(R);
   145: 
   146:     // Note register definitions, direct ones as well as indirect side-effects.
   147:     // Super-registers are not tracked directly, but their components.
   148:     for (MCRegAliasIterator SRI(R, &RI, RI.subregs(R).empty()); SRI.isValid();
   149:          ++SRI) {
   150:       if (!RI.subregs(*SRI).empty())
   151:         // Skip super-registers defined indirectly.
   152:         continue;
   153: 
   154:       if (R == *SRI) {
   155:         if (S == R)
   156:           // Avoid scoring the defined register multiple times.
   157:           continue;
   158:         else
   159:           // Note that the defined register has already been scored.
   160:           S = R;
   161:       }
   162: 
   163:       if (Hexagon::P3_0 != R && Hexagon::P3_0 == *SRI)
   164:         // P3:0 is a special case, since multiple predicate register definitions
   165:         // in a packet is allowed as the equivalent of their logical "and".
   166:         // Only an explicit definition of P3:0 is noted as such; if a
   167:         // side-effect, then note as a soft definition.
   168:         SoftDefs.insert(*SRI);
   169:       else if (HexagonMCInstrInfo::isPredicateLate(MCII, MCI) &&
   170:                HexagonMCInstrInfo::isPredReg(RI, *SRI))
   171:         // Some insns produce predicates too late to be used in the same packet.
   172:         LatePreds.insert(*SRI);
   173:       else if (i == 0 && HexagonMCInstrInfo::getType(MCII, MCI) ==
   174:                              HexagonII::TypeCVI_VM_TMP_LD)
   175:         // Temporary loads should be used in the same packet, but don't commit
   176:         // results, so it should be disregarded if another insn changes the same
   177:         // register.
   178:         // TODO: relies on the impossibility of a current and a temporary loads
   179:         // in the same packet.
   180:         TmpDefs.insert(*SRI);
   181:       else if (!IgnoreTmpDst)
   182:         Defs[*SRI].insert(PredSense(PredReg, isTrue));
   183:     }
   184:   }
   185: 
   186:   // Figure out definitions of new predicate registers.
   187:   if (HexagonMCInstrInfo::isPredicatedNew(MCII, MCI))
   188:     for (unsigned i = MCID.getNumDefs(); i < MCID.getNumOperands(); ++i)
   189:       if (MCI.getOperand(i).isReg()) {
   190:         MCRegister P = MCI.getOperand(i).getReg();
   191: 
   192:         if (HexagonMCInstrInfo::isPredReg(RI, P))
   193:           NewPreds.insert(P);
   194:       }
   195: }
   196: 
   197: HexagonMCChecker::HexagonMCChecker(MCContext &Context, MCInstrInfo const &MCII,
   198:                                    MCSubtargetInfo const &STI, MCInst &mcb,
   199:                                    MCRegisterInfo const &ri, bool ReportErrors)
   200:     : Context(Context), MCB(mcb), RI(ri), MCII(MCII), STI(STI),
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::hasTmpDst, getNumDefs, getOperand, isValid, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, HexagonMCChecker, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::hasTmpDst, getNumDefs, getOperand, isValid, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII, HexagonMCChecker，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201:       ReportErrors(ReportErrors) {
   202:   init();
   203: }
   204: 
   205: HexagonMCChecker::HexagonMCChecker(HexagonMCChecker const &Other,
   206:                                    MCSubtargetInfo const &STI,
   207:                                    bool CopyReportErrors)
   208:     : Context(Other.Context), MCB(Other.MCB), RI(Other.RI), MCII(Other.MCII),
   209:       STI(STI), ReportErrors(CopyReportErrors ? Other.ReportErrors : false) {
   210:   init();
   211: }
   212: 
   213: bool HexagonMCChecker::check(bool FullCheck) {
   214:   bool chkP = checkPredicates();
   215:   bool chkNV = checkNewValues();
   216:   bool chkR = checkRegisters();
   217:   bool chkRRO = checkRegistersReadOnly();
   218:   checkRegisterCurDefs();
   219:   bool chkS = checkSolo();
   220:   bool chkSh = true;
   221:   if (FullCheck)
   222:     chkSh = checkShuffle();
   223:   bool chkSl = true;
   224:   if (FullCheck)
   225:     chkSl = checkSlots();
   226:   bool chkAXOK = checkAXOK();
   227:   bool chkCofMax1 = checkCOFMax1();
   228:   bool chkHWLoop = checkHWLoop();
   229:   bool chkValidTmpDst = FullCheck ? checkValidTmpDst() : true;
   230:   bool chkLegalVecRegPair = checkLegalVecRegPair();
   231:   bool ChkHVXAccum = checkHVXAccum();
   232:   bool chk = chkP && chkNV && chkR && chkRRO && chkS && chkSh && chkSl &&
   233:              chkAXOK && chkCofMax1 && chkHWLoop && chkValidTmpDst &&
   234:              chkLegalVecRegPair && ChkHVXAccum;
   235: 
   236:   return chk;
   237: }
   238: 
   239: static bool isDuplexAGroup(unsigned Opcode) {
   240:   switch (Opcode) {
   241:   case Hexagon::SA1_addi:
   242:   case Hexagon::SA1_addrx:
   243:   case Hexagon::SA1_addsp:
   244:   case Hexagon::SA1_and1:
   245:   case Hexagon::SA1_clrf:
   246:   case Hexagon::SA1_clrfnew:
   247:   case Hexagon::SA1_clrt:
   248:   case Hexagon::SA1_clrtnew:
   249:   case Hexagon::SA1_cmpeqi:
   250:   case Hexagon::SA1_combine0i:
   251:   case Hexagon::SA1_combine1i:
   252:   case Hexagon::SA1_combine2i:
   253:   case Hexagon::SA1_combine3i:
   254:   case Hexagon::SA1_combinerz:
   255:   case Hexagon::SA1_combinezr:
   256:   case Hexagon::SA1_dec:
   257:   case Hexagon::SA1_inc:
   258:   case Hexagon::SA1_seti:
   259:   case Hexagon::SA1_setin1:
   260:   case Hexagon::SA1_sxtb:
   261:   case Hexagon::SA1_sxth:
   262:   case Hexagon::SA1_tfr:
   263:   case Hexagon::SA1_zxtb:
   264:   case Hexagon::SA1_zxth:
   265:     return true;
   266:     break;
   267:   default:
   268:     return false;
   269:   }
   270: }
   271: 
   272: static bool isNeitherAnorX(MCInstrInfo const &MCII, MCInst const &ID) {
   273:   if (HexagonMCInstrInfo::isFloat(MCII, ID))
   274:     return true;
   275:   unsigned Type = HexagonMCInstrInfo::getType(MCII, ID);
   276:   switch (Type) {
   277:   case HexagonII::TypeALU32_2op:
   278:   case HexagonII::TypeALU32_3op:
   279:   case HexagonII::TypeALU32_ADDI:
   280:   case HexagonII::TypeS_2op:
   281:   case HexagonII::TypeS_3op:
   282:   case HexagonII::TypeEXTENDER:
   283:   case HexagonII::TypeM:
   284:   case HexagonII::TypeALU64:
   285:     return false;
   286:   case HexagonII::TypeSUBINSN: {
   287:     return !isDuplexAGroup(ID.getOpcode());
   288:   }
   289:   case HexagonII::TypeDUPLEX:
   290:     llvm_unreachable("unexpected duplex instruction");
   291:   default:
   292:     return true;
   293:   }
   294: }
   295: 
   296: bool HexagonMCChecker::checkAXOK() {
   297:   MCInst const *HasSoloAXInst = nullptr;
   298:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   299:     if (HexagonMCInstrInfo::isSoloAX(MCII, I)) {
   300:       HasSoloAXInst = &I;
```
- EN: It declares or implements routines such as ReportErrors, init, HexagonMCChecker::HexagonMCChecker, HexagonMCChecker::check, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCChecker, HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 ReportErrors, init, HexagonMCChecker::HexagonMCChecker, HexagonMCChecker::check, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker, HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:     }
   302:   }
   303:   if (!HasSoloAXInst)
   304:     return true;
   305:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   306:     if (&I != HasSoloAXInst && isNeitherAnorX(MCII, I)) {
   307:       reportError(
   308:           HasSoloAXInst->getLoc(),
   309:           Twine("Instruction can only be in a packet with ALU or non-FPU XTYPE "
   310:                 "instructions"));
   311:       reportError(I.getLoc(),
   312:                   Twine("Not an ALU or non-FPU XTYPE instruction"));
   313:       return false;
   314:     }
   315:   }
   316:   return true;
   317: }
   318: 
   319: void HexagonMCChecker::reportBranchErrors() {
   320:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   321:     if (HexagonMCInstrInfo::IsABranchingInst(MCII, STI, I))
   322:       reportNote(I.getLoc(), "Branching instruction");
   323:   }
   324: }
   325: 
   326: bool HexagonMCChecker::checkHWLoop() {
   327:   if (!HexagonMCInstrInfo::isInnerLoop(MCB) &&
   328:       !HexagonMCInstrInfo::isOuterLoop(MCB))
   329:     return true;
   330:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   331:     if (HexagonMCInstrInfo::IsABranchingInst(MCII, STI, I)) {
   332:       reportError(MCB.getLoc(),
   333:                   "Branches cannot be in a packet with hardware loops");
   334:       reportBranchErrors();
   335:       return false;
   336:     }
   337:   }
   338:   return true;
   339: }
   340: 
   341: bool HexagonMCChecker::checkCOFMax1() {
   342:   SmallVector<MCInst const *, 2> BranchLocations;
   343:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   344:     if (HexagonMCInstrInfo::IsABranchingInst(MCII, STI, I))
   345:       BranchLocations.push_back(&I);
   346:   }
   347:   for (unsigned J = 0, N = BranchLocations.size(); J < N; ++J) {
   348:     MCInst const &I = *BranchLocations[J];
   349:     if (HexagonMCInstrInfo::isCofMax1(MCII, I)) {
   350:       bool Relax1 = HexagonMCInstrInfo::isCofRelax1(MCII, I);
   351:       bool Relax2 = HexagonMCInstrInfo::isCofRelax2(MCII, I);
   352:       if (N > 1 && !Relax1 && !Relax2) {
   353:         reportError(I.getLoc(),
   354:                     "Instruction may not be in a packet with other branches");
   355:         reportBranchErrors();
   356:         return false;
   357:       }
   358:       if (N > 1 && J == 0 && !Relax1) {
   359:         reportError(I.getLoc(),
   360:                     "Instruction may not be the first branch in packet");
   361:         reportBranchErrors();
   362:         return false;
   363:       }
   364:       if (N > 1 && J == 1 && !Relax2) {
   365:         reportError(I.getLoc(),
   366:                     "Instruction may not be the second branch in packet");
   367:         reportBranchErrors();
   368:         return false;
   369:       }
   370:     }
   371:   }
   372:   return true;
   373: }
   374: 
   375: bool HexagonMCChecker::checkSlots() {
   376:   if (HexagonMCInstrInfo::slotsConsumed(MCII, STI, MCB) >
   377:       HexagonMCInstrInfo::packetSizeSlots(STI)) {
   378:     reportError("invalid instruction packet: out of slots");
   379:     return false;
   380:   }
   381:   return true;
   382: }
   383: 
   384: // Check legal use of predicate registers.
   385: bool HexagonMCChecker::checkPredicates() {
   386:   // Check for proper use of new predicate registers.
   387:   for (const auto &I : NewPreds) {
   388:     MCRegister P = I;
   389: 
   390:     if (!Defs.count(P) || LatePreds.count(P) || Defs.count(Hexagon::P3_0)) {
   391:       // Error out if the new predicate register is not defined,
   392:       // or defined "late"
   393:       // (e.g., "{ if (p3.new)... ; p3 = sp1loop0(#r7:2, Rs) }").
   394:       reportErrorNewValue(P);
   395:       return false;
   396:     }
   397:   }
   398: 
   399:   // Check for proper use of auto-anded of predicate registers.
   400:   for (const auto &I : LatePreds) {
```
- EN: It declares or implements routines such as reportError, HexagonMCChecker::reportBranchErrors, HexagonMCChecker::checkHWLoop, reportBranchErrors, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCChecker, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 reportError, HexagonMCChecker::reportBranchErrors, HexagonMCChecker::checkHWLoop, reportBranchErrors, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCChecker，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:     MCRegister P = I;
   402: 
   403:     if (LatePreds.count(P) > 1 || Defs.count(P)) {
   404:       // Error out if predicate register defined "late" multiple times or
   405:       // defined late and regularly defined
   406:       // (e.g., "{ p3 = sp1loop0(...); p3 = cmp.eq(...) }".
   407:       reportErrorRegisters(P);
   408:       return false;
   409:     }
   410:   }
   411: 
   412:   return true;
   413: }
   414: 
   415: // Check legal use of new values.
   416: bool HexagonMCChecker::checkNewValues() {
   417:   for (auto const &ConsumerInst :
   418:        HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   419:     if (!HexagonMCInstrInfo::isNewValue(MCII, ConsumerInst))
   420:       continue;
   421: 
   422:     const HexagonMCInstrInfo::PredicateInfo ConsumerPredInfo =
   423:         HexagonMCInstrInfo::predicateInfo(MCII, ConsumerInst);
   424: 
   425:     bool Branch = HexagonMCInstrInfo::getDesc(MCII, ConsumerInst).isBranch();
   426:     MCOperand const &Op =
   427:         HexagonMCInstrInfo::getNewValueOperand(MCII, ConsumerInst);
   428:     assert(Op.isReg());
   429: 
   430:     auto Producer = registerProducer(Op.getReg(), ConsumerPredInfo);
   431:     const MCInst *const ProducerInst = std::get<0>(Producer);
   432:     const HexagonMCInstrInfo::PredicateInfo ProducerPredInfo =
   433:         std::get<2>(Producer);
   434: 
   435:     if (ProducerInst == nullptr) {
   436:       reportError(ConsumerInst.getLoc(),
   437:                   "New value register consumer has no producer");
   438:       return false;
   439:     }
   440:     if (!RelaxNVChecks) {
   441:       // Checks that statically prove correct new value consumption
   442:       if (ProducerPredInfo.isPredicated() &&
   443:           (!ConsumerPredInfo.isPredicated() ||
   444:            llvm::HexagonMCInstrInfo::getType(MCII, ConsumerInst) ==
   445:                HexagonII::TypeNCJ)) {
   446:         reportNote(
   447:             ProducerInst->getLoc(),
   448:             "Register producer is predicated and consumer is unconditional");
   449:         reportError(ConsumerInst.getLoc(),
   450:                     "Instruction does not have a valid new register producer");
   451:         return false;
   452:       }
   453:       if (ProducerPredInfo.Register != Hexagon::NoRegister &&
   454:           ProducerPredInfo.Register != ConsumerPredInfo.Register) {
   455:         reportNote(ProducerInst->getLoc(),
   456:                    "Register producer does not use the same predicate "
   457:                    "register as the consumer");
   458:         reportError(ConsumerInst.getLoc(),
   459:                     "Instruction does not have a valid new register producer");
   460:         return false;
   461:       }
   462:     }
   463:     if (ProducerPredInfo.Register == ConsumerPredInfo.Register &&
   464:         ConsumerPredInfo.PredicatedTrue != ProducerPredInfo.PredicatedTrue) {
   465:       reportNote(
   466:           ProducerInst->getLoc(),
   467:           "Register producer has the opposite predicate sense as consumer");
   468:       reportError(ConsumerInst.getLoc(),
   469:                   "Instruction does not have a valid new register producer");
   470:       return false;
   471:     }
   472: 
   473:     MCInstrDesc const &Desc = HexagonMCInstrInfo::getDesc(MCII, *ProducerInst);
   474:     const unsigned ProducerOpIndex = std::get<1>(Producer);
   475: 
   476:     if (Desc.operands()[ProducerOpIndex].RegClass ==
   477:         Hexagon::DoubleRegsRegClassID) {
   478:       reportNote(ProducerInst->getLoc(),
   479:                  "Double registers cannot be new-value producers");
   480:       reportError(ConsumerInst.getLoc(),
   481:                   "Instruction does not have a valid new register producer");
   482:       return false;
   483:     }
   484: 
   485:     // The ProducerOpIsMemIndex logic checks for the index of the producer
   486:     // register operand.  Z-reg load instructions have an implicit operand
   487:     // that's not encoded, so the producer won't appear as the 1-th def, it
   488:     // will be at the 0-th.
   489:     const unsigned ProducerOpSearchIndex =
   490:         (HexagonMCInstrInfo::getType(MCII, *ProducerInst) ==
   491:          HexagonII::TypeCVI_ZW)
   492:             ? 0
   493:             : 1;
   494: 
   495:     const bool ProducerOpIsMemIndex =
   496:         ((Desc.mayLoad() && ProducerOpIndex == ProducerOpSearchIndex) ||
   497:          (Desc.mayStore() && ProducerOpIndex == 0));
   498: 
   499:     if (ProducerOpIsMemIndex) {
   500:       unsigned Mode = HexagonMCInstrInfo::getAddrMode(MCII, *ProducerInst);
```
- EN: It declares or implements routines such as sp1loop0, reportErrorRegisters, HexagonMCChecker::checkNewValues, HexagonMCInstrInfo::predicateInfo, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCChecker, HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 sp1loop0, reportErrorRegisters, HexagonMCChecker::checkNewValues, HexagonMCInstrInfo::predicateInfo, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker, HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501: 
   502:       StringRef ModeError;
   503:       if (Mode == HexagonII::AbsoluteSet)
   504:         ModeError = "Absolute-set";
   505:       if (Mode == HexagonII::PostInc)
   506:         ModeError = "Auto-increment";
   507:       if (!ModeError.empty()) {
   508:         reportNote(ProducerInst->getLoc(),
   509:                    ModeError + " registers cannot be a new-value "
   510:                                "producer");
   511:         reportError(ConsumerInst.getLoc(),
   512:                     "Instruction does not have a valid new register producer");
   513:         return false;
   514:       }
   515:     }
   516:     if (Branch && HexagonMCInstrInfo::isFloat(MCII, *ProducerInst)) {
   517:       reportNote(ProducerInst->getLoc(),
   518:                  "FPU instructions cannot be new-value producers for jumps");
   519:       reportError(ConsumerInst.getLoc(),
   520:                   "Instruction does not have a valid new register producer");
   521:       return false;
   522:     }
   523:   }
   524:   return true;
   525: }
   526: 
   527: bool HexagonMCChecker::checkRegistersReadOnly() {
   528:   for (auto I : HexagonMCInstrInfo::bundleInstructions(MCB)) {
   529:     MCInst const &Inst = *I.getInst();
   530:     unsigned Defs = HexagonMCInstrInfo::getDesc(MCII, Inst).getNumDefs();
   531:     for (unsigned j = 0; j < Defs; ++j) {
   532:       MCOperand const &Operand = Inst.getOperand(j);
   533:       assert(Operand.isReg() && "Def is not a register");
   534:       MCRegister Register = Operand.getReg();
   535:       if (ReadOnly.find(Register) != ReadOnly.end()) {
   536:         reportError(Inst.getLoc(), "Cannot write to read-only register `" +
   537:                                        Twine(RI.getName(Register)) + "'");
   538:         return false;
   539:       }
   540:     }
   541:   }
   542:   return true;
   543: }
   544: 
   545: bool HexagonMCChecker::registerUsed(MCRegister Register) {
   546:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB))
   547:     for (unsigned j = HexagonMCInstrInfo::getDesc(MCII, I).getNumDefs(),
   548:                   n = I.getNumOperands();
   549:          j < n; ++j) {
   550:       MCOperand const &Operand = I.getOperand(j);
   551:       if (Operand.isReg() && Operand.getReg() == Register)
   552:         return true;
   553:     }
   554:   return false;
   555: }
   556: 
   557: std::tuple<MCInst const *, unsigned, HexagonMCInstrInfo::PredicateInfo>
   558: HexagonMCChecker::registerProducer(
   559:     MCRegister Register, HexagonMCInstrInfo::PredicateInfo ConsumerPredicate) {
   560:   std::tuple<MCInst const *, unsigned, HexagonMCInstrInfo::PredicateInfo>
   561:       WrongSense;
   562: 
   563:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   564:     MCInstrDesc const &Desc = HexagonMCInstrInfo::getDesc(MCII, I);
   565:     auto ProducerPredicate = HexagonMCInstrInfo::predicateInfo(MCII, I);
   566: 
   567:     for (unsigned J = 0, N = Desc.getNumDefs(); J < N; ++J)
   568:       for (auto K = MCRegAliasIterator(I.getOperand(J).getReg(), &RI, true);
   569:            K.isValid(); ++K)
   570:         if (*K == Register) {
   571:           if (RelaxNVChecks ||
   572:               (ProducerPredicate.Register == ConsumerPredicate.Register &&
   573:                (ProducerPredicate.Register == Hexagon::NoRegister ||
   574:                 ProducerPredicate.PredicatedTrue ==
   575:                     ConsumerPredicate.PredicatedTrue)))
   576:             return std::make_tuple(&I, J, ProducerPredicate);
   577:           std::get<0>(WrongSense) = &I;
   578:           std::get<1>(WrongSense) = J;
   579:           std::get<2>(WrongSense) = ProducerPredicate;
   580:         }
   581:     if (Register == Hexagon::VTMP && HexagonMCInstrInfo::hasTmpDst(MCII, I))
   582:       return std::make_tuple(&I, 0, HexagonMCInstrInfo::PredicateInfo());
   583:   }
   584:   return WrongSense;
   585: }
   586: 
   587: void HexagonMCChecker::checkRegisterCurDefs() {
   588:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   589:     if (HexagonMCInstrInfo::isCVINew(MCII, I) &&
   590:         HexagonMCInstrInfo::getDesc(MCII, I).mayLoad()) {
   591:       const MCRegister RegDef = I.getOperand(0).getReg();
   592: 
   593:       bool HasRegDefUse = false;
   594:       for (MCRegAliasIterator Alias(RegDef, &RI, true); Alias.isValid();
   595:            ++Alias)
   596:         HasRegDefUse = HasRegDefUse || registerUsed(*Alias);
   597: 
   598:       if (!HasRegDefUse)
   599:         reportWarning("Register `" + Twine(RI.getName(RegDef)) +
   600:                       "' used with `.cur' "
```
- EN: It declares or implements routines such as reportNote, reportError, HexagonMCChecker::checkRegistersReadOnly, getInst, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonII, HexagonMCInstrInfo, HexagonMCChecker, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 reportNote, reportError, HexagonMCChecker::checkRegistersReadOnly, getInst, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonMCInstrInfo, HexagonMCChecker，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:                       "but not used in the same packet");
   602:     }
   603:   }
   604: }
   605: 
   606: // Check for legal register uses and definitions.
   607: bool HexagonMCChecker::checkRegisters() {
   608:   // Check for proper register definitions.
   609:   for (const auto &I : Defs) {
   610:     MCRegister R = I.first;
   611: 
   612:     if (isLoopRegister(R) && Defs.count(R) > 1 &&
   613:         (HexagonMCInstrInfo::isInnerLoop(MCB) ||
   614:          HexagonMCInstrInfo::isOuterLoop(MCB))) {
   615:       // Error out for definitions of loop registers at the end of a loop.
   616:       reportError("loop-setup and some branch instructions "
   617:                   "cannot be in the same packet");
   618:       return false;
   619:     }
   620:     if (SoftDefs.count(R)) {
   621:       // Error out for explicit changes to registers also weakly defined
   622:       // (e.g., "{ usr = r0; r0 = sfadd(...) }").
   623:       MCRegister UsrR = Hexagon::USR;
   624:       MCRegister BadR = RI.isSubRegister(Hexagon::USR, R) ? UsrR : R;
   625:       reportErrorRegisters(BadR);
   626:       return false;
   627:     }
   628:     if (!HexagonMCInstrInfo::isPredReg(RI, R) && Defs[R].size() > 1) {
   629:       // Check for multiple register definitions.
   630:       PredSet &PM = Defs[R];
   631: 
   632:       // Check for multiple unconditional register definitions.
   633:       if (PM.count(Unconditional)) {
   634:         // Error out on an unconditional change when there are any other
   635:         // changes, conditional or not.
   636:         MCRegister UsrR = Hexagon::USR;
   637:         MCRegister BadR = RI.isSubRegister(Hexagon::USR, R) ? UsrR : R;
   638:         reportErrorRegisters(BadR);
   639:         return false;
   640:       }
   641:       // Check for multiple conditional register definitions.
   642:       for (const auto &J : PM) {
   643:         PredSense P = J;
   644: 
   645:         // Check for multiple uses of the same condition.
   646:         if (PM.count(P) > 1) {
   647:           // Error out on conditional changes based on the same predicate
   648:           // (e.g., "{ if (!p0) r0 =...; if (!p0) r0 =... }").
   649:           reportErrorRegisters(R);
   650:           return false;
   651:         }
   652:         // Check for the use of the complementary condition.
   653:         P.second = !P.second;
   654:         if (PM.count(P) && PM.size() > 2) {
   655:           // Error out on conditional changes based on the same predicate
   656:           // multiple times
   657:           // (e.g., "if (p0) r0 =...; if (!p0) r0 =... }; if (!p0) r0 =...").
   658:           reportErrorRegisters(R);
   659:           return false;
   660:         }
   661:       }
   662:     }
   663:   }
   664: 
   665:   // Check for use of temporary definitions.
   666:   for (const auto &I : TmpDefs) {
   667:     MCRegister R = I;
   668: 
   669:     if (!Uses.count(R)) {
   670:       // special case for vhist
   671:       bool vHistFound = false;
   672:       for (auto const &HMI : HexagonMCInstrInfo::bundleInstructions(MCB)) {
   673:         if (HexagonMCInstrInfo::getType(MCII, *HMI.getInst()) ==
   674:             HexagonII::TypeCVI_HIST) {
   675:           vHistFound = true; // vhist() implicitly uses ALL REGxx.tmp
   676:           break;
   677:         }
   678:       }
   679:       // Warn on an unused temporary definition.
   680:       if (!vHistFound) {
   681:         reportWarning("register `" + Twine(RI.getName(R)) +
   682:                       "' used with `.tmp' but not used in the same packet");
   683:         return true;
   684:       }
   685:     }
   686:   }
   687: 
   688:   return true;
   689: }
   690: 
   691: // Check for legal use of solo insns.
   692: bool HexagonMCChecker::checkSolo() {
   693:   if (HexagonMCInstrInfo::bundleSize(MCB) > 1)
   694:     for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   695:       if (HexagonMCInstrInfo::isSolo(MCII, I)) {
   696:         reportError(I.getLoc(), "Instruction is marked `isSolo' and "
   697:                                 "cannot have other instructions in "
   698:                                 "the same packet");
   699:         return false;
   700:       }
```
- EN: It declares or implements routines such as HexagonMCChecker::checkRegisters, reportError, reportErrorRegisters, reportWarning, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCChecker, HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCChecker::checkRegisters, reportError, reportErrorRegisters, reportWarning, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker, HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:     }
   702: 
   703:   return true;
   704: }
   705: 
   706: bool HexagonMCChecker::checkShuffle() {
   707:   HexagonMCShuffler MCSDX(Context, ReportErrors, MCII, STI, MCB);
   708:   return MCSDX.check();
   709: }
   710: 
   711: bool HexagonMCChecker::checkValidTmpDst() {
   712:   if (!STI.hasFeature(Hexagon::ArchV69)) {
   713:     return true;
   714:   }
   715:   auto HasTmp = [&](MCInst const &I) {
   716:     return HexagonMCInstrInfo::hasTmpDst(MCII, I) ||
   717:            HexagonMCInstrInfo::hasHvxTmp(MCII, I);
   718:   };
   719:   unsigned HasTmpCount =
   720:       llvm::count_if(HexagonMCInstrInfo::bundleInstructions(MCII, MCB), HasTmp);
   721: 
   722:   if (HasTmpCount > 1) {
   723:     reportError(
   724:         MCB.getLoc(),
   725:         "this packet has more than one HVX vtmp/.tmp destination instruction");
   726: 
   727:     for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB))
   728:       if (HasTmp(I))
   729:         reportNote(I.getLoc(),
   730:                    "this is an HVX vtmp/.tmp destination instruction");
   731: 
   732:     return false;
   733:   }
   734:   return true;
   735: }
   736: 
   737: void HexagonMCChecker::compoundRegisterMap(unsigned &Register) {
   738:   switch (Register) {
   739:   default:
   740:     break;
   741:   case Hexagon::R15:
   742:     Register = Hexagon::R23;
   743:     break;
   744:   case Hexagon::R14:
   745:     Register = Hexagon::R22;
   746:     break;
   747:   case Hexagon::R13:
   748:     Register = Hexagon::R21;
   749:     break;
   750:   case Hexagon::R12:
   751:     Register = Hexagon::R20;
   752:     break;
   753:   case Hexagon::R11:
   754:     Register = Hexagon::R19;
   755:     break;
   756:   case Hexagon::R10:
   757:     Register = Hexagon::R18;
   758:     break;
   759:   case Hexagon::R9:
   760:     Register = Hexagon::R17;
   761:     break;
   762:   case Hexagon::R8:
   763:     Register = Hexagon::R16;
   764:     break;
   765:   }
   766: }
   767: 
   768: void HexagonMCChecker::reportErrorRegisters(MCRegister Register) {
   769:   reportError("register `" + Twine(RI.getName(Register)) +
   770:               "' modified more than once");
   771: }
   772: 
   773: void HexagonMCChecker::reportErrorNewValue(MCRegister Register) {
   774:   reportError("register `" + Twine(RI.getName(Register)) +
   775:               "' used with `.new' "
   776:               "but not validly modified in the same packet");
   777: }
   778: 
   779: void HexagonMCChecker::reportError(Twine const &Msg) {
   780:   reportError(MCB.getLoc(), Msg);
   781: }
   782: 
   783: void HexagonMCChecker::reportError(SMLoc Loc, Twine const &Msg) {
   784:   if (ReportErrors)
   785:     Context.reportError(Loc, Msg);
   786: }
   787: 
   788: void HexagonMCChecker::reportNote(SMLoc Loc, llvm::Twine const &Msg) {
   789:   if (ReportErrors) {
   790:     auto SM = Context.getSourceManager();
   791:     if (SM)
   792:       SM->PrintMessage(Loc, SourceMgr::DK_Note, Msg);
   793:   }
   794: }
   795: 
   796: void HexagonMCChecker::reportWarning(Twine const &Msg) {
   797:   if (ReportErrors)
   798:     Context.reportWarning(MCB.getLoc(), Msg);
   799: }
   800: 
```
- EN: It declares or implements routines such as HexagonMCChecker::checkShuffle, MCSDX, check, HexagonMCChecker::checkValidTmpDst, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCChecker, HexagonMCShuffler, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCChecker::checkShuffle, MCSDX, check, HexagonMCChecker::checkValidTmpDst, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker, HexagonMCShuffler, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 801-831 / 第 801-831 行

```cpp
   801: bool HexagonMCChecker::checkLegalVecRegPair() {
   802:   const bool IsPermitted = STI.hasFeature(Hexagon::ArchV67);
   803:   const bool HasReversePairs = ReversePairs.size() != 0;
   804: 
   805:   if (!IsPermitted && HasReversePairs) {
   806:     for (auto R : ReversePairs)
   807:       reportError("register pair `" + Twine(RI.getName(R)) +
   808:                   "' is not permitted for this architecture");
   809:     return false;
   810:   }
   811:   return true;
   812: }
   813: 
   814: // Vd.tmp can't be accumulated
   815: bool HexagonMCChecker::checkHVXAccum()
   816: {
   817:   for (const auto &I : HexagonMCInstrInfo::bundleInstructions(MCII, MCB)) {
   818:     bool IsTarget =
   819:         HexagonMCInstrInfo::isAccumulator(MCII, I) && I.getOperand(0).isReg();
   820:     if (!IsTarget)
   821:       continue;
   822:     MCRegister R = I.getOperand(0).getReg();
   823:     TmpDefsIterator It = TmpDefs.find(R);
   824:     if (It != TmpDefs.end()) {
   825:       reportError("register `" + Twine(RI.getName(R)) + ".tmp" +
   826:                   "' is accumulated in this packet");
   827:       return false;
   828:     }
   829:   }
   830:   return true;
   831: }
```
- EN: It declares or implements routines such as HexagonMCChecker::checkLegalVecRegPair, hasFeature, HexagonMCChecker::checkHVXAccum, HexagonMCInstrInfo::isAccumulator, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCChecker, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCChecker::checkLegalVecRegPair, hasFeature, HexagonMCChecker::checkHVXAccum, HexagonMCInstrInfo::isAccumulator, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCChecker, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCShuffler.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/Twine.h, llvm/MC/MCContext.h, llvm/MC/MCInst.h, llvm/MC/MCInstrDesc.h, llvm/MC/MCRegisterInfo.h, ... (14 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCChecker, HexagonBaseInfo, HexagonMCInstrInfo, HexagonMCShuffler, HexagonMCTargetDesc, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
