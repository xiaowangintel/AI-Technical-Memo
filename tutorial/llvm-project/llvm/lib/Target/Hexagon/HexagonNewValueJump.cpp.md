# HexagonNewValueJump.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonNewValueJump.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon new-value jump formation and legality checks.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonNewValueJump.cpp - Hexagon Backend New Value Jump -----------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements NewValueJump pass in Hexagon.
    10: // Ideally, we should merge this as a Peephole pass prior to register
    11: // allocation, but because we have a spill in between the feeder and new value
    12: // jump instructions, we are forced to write after register allocation.
    13: // Having said that, we should re-attempt to pull this earlier at some point
    14: // in future.
    15: 
    16: // The basic approach looks for sequence of predicated jump, compare instruction
    17: // that generates the predicate and, the feeder to the predicate. Once it finds
    18: // all, it collapses compare and jump instruction into a new value jump
    19: // instructions.
    20: //
    21: //===----------------------------------------------------------------------===//
    22: 
    23: #include "llvm/InitializePasses.h"
    24: #include "Hexagon.h"
    25: #include "HexagonInstrInfo.h"
    26: #include "HexagonRegisterInfo.h"
    27: #include "HexagonSubtarget.h"
    28: #include "llvm/ADT/Statistic.h"
    29: #include "llvm/CodeGen/MachineBasicBlock.h"
    30: #include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
    31: #include "llvm/CodeGen/MachineFunction.h"
    32: #include "llvm/CodeGen/MachineFunctionPass.h"
    33: #include "llvm/CodeGen/MachineInstr.h"
    34: #include "llvm/CodeGen/MachineInstrBuilder.h"
    35: #include "llvm/CodeGen/MachineOperand.h"
    36: #include "llvm/CodeGen/MachineRegisterInfo.h"
    37: #include "llvm/CodeGen/TargetOpcodes.h"
    38: #include "llvm/CodeGen/TargetRegisterInfo.h"
    39: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    40: #include "llvm/IR/DebugLoc.h"
    41: #include "llvm/MC/MCInstrDesc.h"
    42: #include "llvm/Pass.h"
    43: #include "llvm/Support/BranchProbability.h"
    44: #include "llvm/Support/CommandLine.h"
    45: #include "llvm/Support/Debug.h"
    46: #include "llvm/Support/ErrorHandling.h"
    47: #include "llvm/Support/MathExtras.h"
    48: #include "llvm/Support/raw_ostream.h"
    49: #include <cassert>
    50: #include <cstdint>
```
- EN: It imports headers such as llvm/InitializePasses.h, Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, ... (28 total), establishing the LLVM/Hexagon APIs used below. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonNewValueJump, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里引入了 llvm/InitializePasses.h, Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, ... (28 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonNewValueJump, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 51-100 / 第 51-100 行

```cpp
    51: #include <iterator>
    52: 
    53: using namespace llvm;
    54: 
    55: #define DEBUG_TYPE "hexagon-nvj"
    56: 
    57: STATISTIC(NumNVJGenerated, "Number of New Value Jump Instructions created");
    58: 
    59: static cl::opt<int> DbgNVJCount("nvj-count", cl::init(-1), cl::Hidden,
    60:     cl::desc("Maximum number of predicated jumps to be converted to "
    61:     "New Value Jump"));
    62: 
    63: static cl::opt<bool> DisableNewValueJumps("disable-nvjump", cl::Hidden,
    64:                                           cl::desc("Disable New Value Jumps"));
    65: 
    66: extern cl::opt<bool> DisablePacketizer;
    67: 
    68: namespace {
    69: 
    70:   struct HexagonNewValueJump : public MachineFunctionPass {
    71:     static char ID;
    72: 
    73:     HexagonNewValueJump() : MachineFunctionPass(ID) {}
    74: 
    75:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    76:       AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();
    77:       MachineFunctionPass::getAnalysisUsage(AU);
    78:     }
    79: 
    80:     StringRef getPassName() const override { return "Hexagon NewValueJump"; }
    81: 
    82:     bool runOnMachineFunction(MachineFunction &Fn) override;
    83: 
    84:     MachineFunctionProperties getRequiredProperties() const override {
    85:       return MachineFunctionProperties().setNoVRegs();
    86:     }
    87: 
    88:   private:
    89:     const HexagonInstrInfo *QII;
    90:     const HexagonRegisterInfo *QRI;
    91: 
    92:     /// A handle to the branch probability pass.
    93:     const MachineBranchProbabilityInfo *MBPI;
    94: 
    95:     bool isNewValueJumpCandidate(const MachineInstr &MI) const;
    96:   };
    97: 
    98: } // end anonymous namespace
    99: 
   100: char HexagonNewValueJump::ID = 0;
```
- EN: It imports headers such as iterator, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonNewValueJump, which carry the state or API of this component.
- CN: 这里引入了 iterator 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonNewValueJump 等类型，用来承载该组件的状态或接口。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102: INITIALIZE_PASS_BEGIN(HexagonNewValueJump, "hexagon-nvj",
   103:                       "Hexagon NewValueJump", false, false)
   104: INITIALIZE_PASS_DEPENDENCY(MachineBranchProbabilityInfoWrapperPass)
   105: INITIALIZE_PASS_END(HexagonNewValueJump, "hexagon-nvj",
   106:                     "Hexagon NewValueJump", false, false)
   107: 
   108: // We have identified this II could be feeder to NVJ,
   109: // verify that it can be.
   110: static bool canBeFeederToNewValueJump(const HexagonInstrInfo *QII,
   111:                                       const TargetRegisterInfo *TRI,
   112:                                       MachineBasicBlock::iterator II,
   113:                                       MachineBasicBlock::iterator end,
   114:                                       MachineBasicBlock::iterator skip,
   115:                                       MachineFunction &MF) {
   116:   // Predicated instruction can not be feeder to NVJ.
   117:   if (QII->isPredicated(*II))
   118:     return false;
   119: 
   120:   // Bail out if feederReg is a paired register (double regs in
   121:   // our case). One would think that we can check to see if a given
   122:   // register cmpReg1 or cmpReg2 is a sub register of feederReg
   123:   // using -- if (QRI->isSubRegister(feederReg, cmpReg1) logic
   124:   // before the callsite of this function
   125:   // But we can not as it comes in the following fashion.
   126:   //    %d0 = Hexagon_S2_lsr_r_p killed %d0, killed %r2
   127:   //    %r0 = KILL %r0, implicit killed %d0
   128:   //    %p0 = CMPEQri killed %r0, 0
   129:   // Hence, we need to check if it's a KILL instruction.
   130:   if (II->getOpcode() == TargetOpcode::KILL)
   131:     return false;
   132: 
   133:   if (II->isImplicitDef())
   134:     return false;
   135: 
   136:   if (QII->isSolo(*II))
   137:     return false;
   138: 
   139:   if (QII->isFloat(*II))
   140:     return false;
   141: 
   142:   // Make sure that the (unique) def operand is a register from IntRegs.
   143:   bool HadDef = false;
   144:   for (const MachineOperand &Op : II->operands()) {
   145:     if (!Op.isReg() || !Op.isDef())
   146:       continue;
   147:     if (HadDef)
   148:       return false;
   149:     HadDef = true;
   150:     if (!Hexagon::IntRegsRegClass.contains(Op.getReg()))
```
- EN: It declares or implements routines such as INITIALIZE_PASS_BEGIN, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonNewValueJump, HexagonInstrInfo, Hexagon_S2_lsr_r_p, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 INITIALIZE_PASS_BEGIN 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonNewValueJump, HexagonInstrInfo, Hexagon_S2_lsr_r_p，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:       return false;
   152:   }
   153:   assert(HadDef);
   154: 
   155:   // Make sure there is no 'def' or 'use' of any of the uses of
   156:   // feeder insn between its definition, this MI and jump, jmpInst
   157:   // skipping compare, cmpInst.
   158:   // Here's the example.
   159:   //    r21=memub(r22+r24<<#0)
   160:   //    p0 = cmp.eq(r21, #0)
   161:   //    r4=memub(r3+r21<<#0)
   162:   //    if (p0.new) jump:t .LBB29_45
   163:   // Without this check, it will be converted into
   164:   //    r4=memub(r3+r21<<#0)
   165:   //    r21=memub(r22+r24<<#0)
   166:   //    p0 = cmp.eq(r21, #0)
   167:   //    if (p0.new) jump:t .LBB29_45
   168:   // and result WAR hazards if converted to New Value Jump.
   169:   for (unsigned i = 0; i < II->getNumOperands(); ++i) {
   170:     if (II->getOperand(i).isReg() &&
   171:         (II->getOperand(i).isUse() || II->getOperand(i).isDef())) {
   172:       MachineBasicBlock::iterator localII = II;
   173:       ++localII;
   174:       Register Reg = II->getOperand(i).getReg();
   175:       for (MachineBasicBlock::iterator localBegin = localII; localBegin != end;
   176:            ++localBegin) {
   177:         if (localBegin == skip)
   178:           continue;
   179:         // Check for Subregisters too.
   180:         if (localBegin->modifiesRegister(Reg, TRI) ||
   181:             localBegin->readsRegister(Reg, TRI))
   182:           return false;
   183:       }
   184:     }
   185:   }
   186:   return true;
   187: }
   188: 
   189: // These are the common checks that need to performed
   190: // to determine if
   191: // 1. compare instruction can be moved before jump.
   192: // 2. feeder to the compare instruction can be moved before jump.
   193: static bool commonChecksToProhibitNewValueJump(bool afterRA,
   194:                           MachineBasicBlock::iterator MII) {
   195:   // If store in path, bail out.
   196:   if (MII->mayStore())
   197:     return false;
   198: 
   199:   // if call in path, bail out.
   200:   if (MII->isCall())
```
- EN: It declares or implements routines such as assert, getNumOperands, getOperand, commonChecksToProhibitNewValueJump, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, getNumOperands, getOperand, commonChecksToProhibitNewValueJump 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-250 / 第 201-250 行

```cpp
   201:     return false;
   202: 
   203:   // if NVJ is running prior to RA, do the following checks.
   204:   if (!afterRA) {
   205:     // The following Target Opcode instructions are spurious
   206:     // to new value jump. If they are in the path, bail out.
   207:     // KILL sets kill flag on the opcode. It also sets up a
   208:     // single register, out of pair.
   209:     //    %d0 = S2_lsr_r_p killed %d0, killed %r2
   210:     //    %r0 = KILL %r0, implicit killed %d0
   211:     //    %p0 = C2_cmpeqi killed %r0, 0
   212:     // PHI can be anything after RA.
   213:     // COPY can remateriaze things in between feeder, compare and nvj.
   214:     if (MII->getOpcode() == TargetOpcode::KILL ||
   215:         MII->getOpcode() == TargetOpcode::PHI ||
   216:         MII->getOpcode() == TargetOpcode::COPY)
   217:       return false;
   218: 
   219:     // The following pseudo Hexagon instructions sets "use" and "def"
   220:     // of registers by individual passes in the backend. At this time,
   221:     // we don't know the scope of usage and definitions of these
   222:     // instructions.
   223:     if (MII->getOpcode() == Hexagon::LDriw_pred ||
   224:         MII->getOpcode() == Hexagon::STriw_pred)
   225:       return false;
   226:   }
   227: 
   228:   return true;
   229: }
   230: 
   231: static bool canCompareBeNewValueJump(const HexagonInstrInfo *QII,
   232:                                      const TargetRegisterInfo *TRI,
   233:                                      MachineBasicBlock::iterator II,
   234:                                      unsigned pReg,
   235:                                      bool secondReg,
   236:                                      bool optLocation,
   237:                                      MachineBasicBlock::iterator end,
   238:                                      MachineFunction &MF) {
   239:   MachineInstr &MI = *II;
   240: 
   241:   // If the second operand of the compare is an imm, make sure it's in the
   242:   // range specified by the arch.
   243:   if (!secondReg) {
   244:     const MachineOperand &Op2 = MI.getOperand(2);
   245:     if (!Op2.isImm())
   246:       return false;
   247: 
   248:     int64_t v = Op2.getImm();
   249:     bool Valid = false;
   250: 
```
- EN: It declares or implements routines such as canCompareBeNewValueJump, getOperand, getImm, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 canCompareBeNewValueJump, getOperand, getImm 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:     switch (MI.getOpcode()) {
   252:       case Hexagon::C2_cmpeqi:
   253:       case Hexagon::C4_cmpneqi:
   254:       case Hexagon::C2_cmpgti:
   255:       case Hexagon::C4_cmpltei:
   256:         Valid = (isUInt<5>(v) || v == -1);
   257:         break;
   258:       case Hexagon::C2_cmpgtui:
   259:       case Hexagon::C4_cmplteui:
   260:         Valid = isUInt<5>(v);
   261:         break;
   262:       case Hexagon::S2_tstbit_i:
   263:       case Hexagon::S4_ntstbit_i:
   264:         Valid = (v == 0);
   265:         break;
   266:     }
   267: 
   268:     if (!Valid)
   269:       return false;
   270:   }
   271: 
   272:   unsigned cmpReg1, cmpOp2 = 0; // cmpOp2 assignment silences compiler warning.
   273:   cmpReg1 = MI.getOperand(1).getReg();
   274: 
   275:   if (secondReg) {
   276:     cmpOp2 = MI.getOperand(2).getReg();
   277: 
   278:     // If the same register appears as both operands, we cannot generate a new
   279:     // value compare. Only one operand may use the .new suffix.
   280:     if (cmpReg1 == cmpOp2)
   281:       return false;
   282: 
   283:     // Make sure that the second register is not from COPY
   284:     // at machine code level, we don't need this, but if we decide
   285:     // to move new value jump prior to RA, we would be needing this.
   286:     MachineRegisterInfo &MRI = MF.getRegInfo();
   287:     if (!Register::isPhysicalRegister(cmpOp2)) {
   288:       MachineInstr *def = MRI.getVRegDef(cmpOp2);
   289:       if (def->getOpcode() == TargetOpcode::COPY)
   290:         return false;
   291:     }
   292:   }
   293: 
   294:   // Walk the instructions after the compare (predicate def) to the jump,
   295:   // and satisfy the following conditions.
   296:   ++II;
   297:   for (MachineBasicBlock::iterator localII = II; localII != end; ++localII) {
   298:     if (localII->isDebugInstr())
   299:       continue;
   300: 
```
- EN: It declares or implements routines such as isUInt<5>, getOperand, getRegInfo, getVRegDef, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 isUInt<5>, getOperand, getRegInfo, getVRegDef 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301:     // Check 1.
   302:     // If "common" checks fail, bail out.
   303:     if (!commonChecksToProhibitNewValueJump(optLocation, localII))
   304:       return false;
   305: 
   306:     // Check 2.
   307:     // If there is a def or use of predicate (result of compare), bail out.
   308:     if (localII->modifiesRegister(pReg, TRI) ||
   309:         localII->readsRegister(pReg, TRI))
   310:       return false;
   311: 
   312:     // Check 3.
   313:     // If there is a def of any of the use of the compare (operands of compare),
   314:     // bail out.
   315:     // Eg.
   316:     //    p0 = cmp.eq(r2, r0)
   317:     //    r2 = r4
   318:     //    if (p0.new) jump:t .LBB28_3
   319:     if (localII->modifiesRegister(cmpReg1, TRI) ||
   320:         (secondReg && localII->modifiesRegister(cmpOp2, TRI)))
   321:       return false;
   322:   }
   323:   return true;
   324: }
   325: 
   326: // Given a compare operator, return a matching New Value Jump compare operator.
   327: // Make sure that MI here is included in isNewValueJumpCandidate.
   328: static unsigned getNewValueJumpOpcode(MachineInstr *MI, int reg,
   329:                                       bool secondRegNewified,
   330:                                       MachineBasicBlock *jmpTarget,
   331:                                       const MachineBranchProbabilityInfo
   332:                                       *MBPI) {
   333:   bool taken = false;
   334:   MachineBasicBlock *Src = MI->getParent();
   335:   const BranchProbability Prediction =
   336:     MBPI->getEdgeProbability(Src, jmpTarget);
   337: 
   338:   if (Prediction >= BranchProbability(1,2))
   339:     taken = true;
   340: 
   341:   switch (MI->getOpcode()) {
   342:     case Hexagon::C2_cmpeq:
   343:       return taken ? Hexagon::J4_cmpeq_t_jumpnv_t
   344:                    : Hexagon::J4_cmpeq_t_jumpnv_nt;
   345: 
   346:     case Hexagon::C2_cmpeqi:
   347:       if (reg >= 0)
   348:         return taken ? Hexagon::J4_cmpeqi_t_jumpnv_t
   349:                      : Hexagon::J4_cmpeqi_t_jumpnv_nt;
   350:       return taken ? Hexagon::J4_cmpeqn1_t_jumpnv_t
```
- EN: It declares or implements routines such as getNewValueJumpOpcode, getParent, getEdgeProbability, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getNewValueJumpOpcode, getParent, getEdgeProbability 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-400 / 第 351-400 行

```cpp
   351:                    : Hexagon::J4_cmpeqn1_t_jumpnv_nt;
   352: 
   353:     case Hexagon::C4_cmpneqi:
   354:       if (reg >= 0)
   355:         return taken ? Hexagon::J4_cmpeqi_f_jumpnv_t
   356:                      : Hexagon::J4_cmpeqi_f_jumpnv_nt;
   357:       return taken ? Hexagon::J4_cmpeqn1_f_jumpnv_t :
   358:                      Hexagon::J4_cmpeqn1_f_jumpnv_nt;
   359: 
   360:     case Hexagon::C2_cmpgt:
   361:       if (secondRegNewified)
   362:         return taken ? Hexagon::J4_cmplt_t_jumpnv_t
   363:                      : Hexagon::J4_cmplt_t_jumpnv_nt;
   364:       return taken ? Hexagon::J4_cmpgt_t_jumpnv_t
   365:                    : Hexagon::J4_cmpgt_t_jumpnv_nt;
   366: 
   367:     case Hexagon::C2_cmpgti:
   368:       if (reg >= 0)
   369:         return taken ? Hexagon::J4_cmpgti_t_jumpnv_t
   370:                      : Hexagon::J4_cmpgti_t_jumpnv_nt;
   371:       return taken ? Hexagon::J4_cmpgtn1_t_jumpnv_t
   372:                    : Hexagon::J4_cmpgtn1_t_jumpnv_nt;
   373: 
   374:     case Hexagon::C2_cmpgtu:
   375:       if (secondRegNewified)
   376:         return taken ? Hexagon::J4_cmpltu_t_jumpnv_t
   377:                      : Hexagon::J4_cmpltu_t_jumpnv_nt;
   378:       return taken ? Hexagon::J4_cmpgtu_t_jumpnv_t
   379:                    : Hexagon::J4_cmpgtu_t_jumpnv_nt;
   380: 
   381:     case Hexagon::C2_cmpgtui:
   382:       return taken ? Hexagon::J4_cmpgtui_t_jumpnv_t
   383:                    : Hexagon::J4_cmpgtui_t_jumpnv_nt;
   384: 
   385:     case Hexagon::C4_cmpneq:
   386:       return taken ? Hexagon::J4_cmpeq_f_jumpnv_t
   387:                    : Hexagon::J4_cmpeq_f_jumpnv_nt;
   388: 
   389:     case Hexagon::C4_cmplte:
   390:       if (secondRegNewified)
   391:         return taken ? Hexagon::J4_cmplt_f_jumpnv_t
   392:                      : Hexagon::J4_cmplt_f_jumpnv_nt;
   393:       return taken ? Hexagon::J4_cmpgt_f_jumpnv_t
   394:                    : Hexagon::J4_cmpgt_f_jumpnv_nt;
   395: 
   396:     case Hexagon::C4_cmplteu:
   397:       if (secondRegNewified)
   398:         return taken ? Hexagon::J4_cmpltu_f_jumpnv_t
   399:                      : Hexagon::J4_cmpltu_f_jumpnv_nt;
   400:       return taken ? Hexagon::J4_cmpgtu_f_jumpnv_t
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 401-450 / 第 401-450 行

```cpp
   401:                    : Hexagon::J4_cmpgtu_f_jumpnv_nt;
   402: 
   403:     case Hexagon::C4_cmpltei:
   404:       if (reg >= 0)
   405:         return taken ? Hexagon::J4_cmpgti_f_jumpnv_t
   406:                      : Hexagon::J4_cmpgti_f_jumpnv_nt;
   407:       return taken ? Hexagon::J4_cmpgtn1_f_jumpnv_t
   408:                    : Hexagon::J4_cmpgtn1_f_jumpnv_nt;
   409: 
   410:     case Hexagon::C4_cmplteui:
   411:       return taken ? Hexagon::J4_cmpgtui_f_jumpnv_t
   412:                    : Hexagon::J4_cmpgtui_f_jumpnv_nt;
   413: 
   414:     default:
   415:        llvm_unreachable("Could not find matching New Value Jump instruction.");
   416:   }
   417:   // return *some value* to avoid compiler warning
   418:   return 0;
   419: }
   420: 
   421: bool HexagonNewValueJump::isNewValueJumpCandidate(
   422:     const MachineInstr &MI) const {
   423:   switch (MI.getOpcode()) {
   424:   case Hexagon::C2_cmpeq:
   425:   case Hexagon::C2_cmpeqi:
   426:   case Hexagon::C2_cmpgt:
   427:   case Hexagon::C2_cmpgti:
   428:   case Hexagon::C2_cmpgtu:
   429:   case Hexagon::C2_cmpgtui:
   430:   case Hexagon::C4_cmpneq:
   431:   case Hexagon::C4_cmpneqi:
   432:   case Hexagon::C4_cmplte:
   433:   case Hexagon::C4_cmplteu:
   434:   case Hexagon::C4_cmpltei:
   435:   case Hexagon::C4_cmplteui:
   436:     return true;
   437: 
   438:   default:
   439:     return false;
   440:   }
   441: }
   442: 
   443: bool HexagonNewValueJump::runOnMachineFunction(MachineFunction &MF) {
   444:   LLVM_DEBUG(dbgs() << "********** Hexagon New Value Jump **********\n"
   445:                     << "********** Function: " << MF.getName() << "\n");
   446: 
   447:   if (skipFunction(MF.getFunction()))
   448:     return false;
   449: 
   450:   // If we move NewValueJump before register allocation we'll need live variable
```
- EN: It declares or implements routines such as llvm_unreachable, HexagonNewValueJump::isNewValueJumpCandidate, HexagonNewValueJump::runOnMachineFunction, LLVM_DEBUG, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonNewValueJump, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm_unreachable, HexagonNewValueJump::isNewValueJumpCandidate, HexagonNewValueJump::runOnMachineFunction, LLVM_DEBUG 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonNewValueJump，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```cpp
   451:   // analysis here too.
   452: 
   453:   QII = static_cast<const HexagonInstrInfo *>(MF.getSubtarget().getInstrInfo());
   454:   QRI = static_cast<const HexagonRegisterInfo *>(
   455:       MF.getSubtarget().getRegisterInfo());
   456:   MBPI = &getAnalysis<MachineBranchProbabilityInfoWrapperPass>().getMBPI();
   457: 
   458:   // New value jumps require the feeder instruction to be in the same packet.
   459:   // If packetization is disabled, we cannot generate new value jumps.
   460:   if (DisableNewValueJumps || DisablePacketizer ||
   461:       !MF.getSubtarget<HexagonSubtarget>().useNewValueJumps())
   462:     return false;
   463: 
   464:   int nvjCount = DbgNVJCount;
   465:   int nvjGenerated = 0;
   466: 
   467:   // Loop through all the bb's of the function
   468:   for (MachineFunction::iterator MBBb = MF.begin(), MBBe = MF.end();
   469:        MBBb != MBBe; ++MBBb) {
   470:     MachineBasicBlock *MBB = &*MBBb;
   471: 
   472:     LLVM_DEBUG(dbgs() << "** dumping bb ** " << MBB->getNumber() << "\n");
   473:     LLVM_DEBUG(MBB->dump());
   474:     LLVM_DEBUG(dbgs() << "\n"
   475:                       << "********** dumping instr bottom up **********\n");
   476:     bool foundJump    = false;
   477:     bool foundCompare = false;
   478:     bool invertPredicate = false;
   479:     unsigned predReg = 0; // predicate reg of the jump.
   480:     unsigned cmpReg1 = 0;
   481:     int cmpOp2 = 0;
   482:     MachineBasicBlock::iterator jmpPos;
   483:     MachineBasicBlock::iterator cmpPos;
   484:     MachineInstr *cmpInstr = nullptr, *jmpInstr = nullptr;
   485:     MachineBasicBlock *jmpTarget = nullptr;
   486:     bool afterRA = false;
   487:     bool isSecondOpReg = false;
   488:     bool isSecondOpNewified = false;
   489:     // Traverse the basic block - bottom up
   490:     for (MachineBasicBlock::iterator MII = MBB->end(), E = MBB->begin();
   491:          MII != E;) {
   492:       MachineInstr &MI = *--MII;
   493:       if (MI.isDebugInstr()) {
   494:         continue;
   495:       }
   496: 
   497:       if ((nvjCount == 0) || (nvjCount > -1 && nvjCount <= nvjGenerated))
   498:         break;
   499: 
   500:       LLVM_DEBUG(dbgs() << "Instr: "; MI.dump(); dbgs() << "\n");
```
- EN: It declares or implements routines such as getSubtarget, getAnalysis<MachineBranchProbabilityInfoWrapperPass>, LLVM_DEBUG, dump, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget, getAnalysis<MachineBranchProbabilityInfoWrapperPass>, LLVM_DEBUG, dump, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 501-550 / 第 501-550 行

```cpp
   501: 
   502:       if (!foundJump && (MI.getOpcode() == Hexagon::J2_jumpt ||
   503:                          MI.getOpcode() == Hexagon::J2_jumptpt ||
   504:                          MI.getOpcode() == Hexagon::J2_jumpf ||
   505:                          MI.getOpcode() == Hexagon::J2_jumpfpt ||
   506:                          MI.getOpcode() == Hexagon::J2_jumptnewpt ||
   507:                          MI.getOpcode() == Hexagon::J2_jumptnew ||
   508:                          MI.getOpcode() == Hexagon::J2_jumpfnewpt ||
   509:                          MI.getOpcode() == Hexagon::J2_jumpfnew)) {
   510:         // This is where you would insert your compare and
   511:         // instr that feeds compare
   512:         jmpPos = MII;
   513:         jmpInstr = &MI;
   514:         predReg = MI.getOperand(0).getReg();
   515:         afterRA = Register::isPhysicalRegister(predReg);
   516: 
   517:         // If ifconverter had not messed up with the kill flags of the
   518:         // operands, the following check on the kill flag would suffice.
   519:         // if(!jmpInstr->getOperand(0).isKill()) break;
   520: 
   521:         // This predicate register is live out of BB
   522:         // this would only work if we can actually use Live
   523:         // variable analysis on phy regs - but LLVM does not
   524:         // provide LV analysis on phys regs.
   525:         //if(LVs.isLiveOut(predReg, *MBB)) break;
   526: 
   527:         // Get all the successors of this block - which will always
   528:         // be 2. Check if the predicate register is live-in in those
   529:         // successor. If yes, we can not delete the predicate -
   530:         // I am doing this only because LLVM does not provide LiveOut
   531:         // at the BB level.
   532:         bool predLive = false;
   533:         for (const MachineBasicBlock *SuccMBB : MBB->successors())
   534:           if (SuccMBB->isLiveIn(predReg))
   535:             predLive = true;
   536:         if (predLive)
   537:           break;
   538: 
   539:         if (!MI.getOperand(1).isMBB())
   540:           continue;
   541:         jmpTarget = MI.getOperand(1).getMBB();
   542:         foundJump = true;
   543:         if (MI.getOpcode() == Hexagon::J2_jumpf ||
   544:             MI.getOpcode() == Hexagon::J2_jumpfnewpt ||
   545:             MI.getOpcode() == Hexagon::J2_jumpfnew) {
   546:           invertPredicate = true;
   547:         }
   548:         continue;
   549:       }
   550: 
```
- EN: It declares or implements routines such as getOperand, Register::isPhysicalRegister, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getOperand, Register::isPhysicalRegister 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 551-600 / 第 551-600 行

```cpp
   551:       // No new value jump if there is a barrier. A barrier has to be in its
   552:       // own packet. A barrier has zero operands. We conservatively bail out
   553:       // here if we see any instruction with zero operands.
   554:       if (foundJump && MI.getNumOperands() == 0)
   555:         break;
   556: 
   557:       if (foundJump && !foundCompare && MI.getOperand(0).isReg() &&
   558:           MI.getOperand(0).getReg() == predReg) {
   559:         // Not all compares can be new value compare. Arch Spec: 7.6.1.1
   560:         if (isNewValueJumpCandidate(MI)) {
   561:           assert(
   562:               (MI.getDesc().isCompare()) &&
   563:               "Only compare instruction can be collapsed into New Value Jump");
   564:           isSecondOpReg = MI.getOperand(2).isReg();
   565: 
   566:           if (!canCompareBeNewValueJump(QII, QRI, MII, predReg, isSecondOpReg,
   567:                                         afterRA, jmpPos, MF))
   568:             break;
   569: 
   570:           cmpInstr = &MI;
   571:           cmpPos = MII;
   572:           foundCompare = true;
   573: 
   574:           // We need cmpReg1 and cmpOp2(imm or reg) while building
   575:           // new value jump instruction.
   576:           cmpReg1 = MI.getOperand(1).getReg();
   577: 
   578:           if (isSecondOpReg)
   579:             cmpOp2 = MI.getOperand(2).getReg();
   580:           else
   581:             cmpOp2 = MI.getOperand(2).getImm();
   582:           continue;
   583:         }
   584:       }
   585: 
   586:       if (foundCompare && foundJump) {
   587:         // If "common" checks fail, bail out on this BB.
   588:         if (!commonChecksToProhibitNewValueJump(afterRA, MII))
   589:           break;
   590: 
   591:         bool foundFeeder = false;
   592:         MachineBasicBlock::iterator feederPos = MII;
   593:         if (MI.getOperand(0).isReg() && MI.getOperand(0).isDef() &&
   594:             (MI.getOperand(0).getReg() == cmpReg1 ||
   595:              (isSecondOpReg &&
   596:               MI.getOperand(0).getReg() == (unsigned)cmpOp2))) {
   597: 
   598:           Register feederReg = MI.getOperand(0).getReg();
   599: 
   600:           // First try to see if we can get the feeder from the first operand
```
- EN: It declares or implements routines such as assert, getOperand, cmpOp2, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, getOperand, cmpOp2 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 601-650 / 第 601-650 行

```cpp
   601:           // of the compare. If we can not, and if secondOpReg is true
   602:           // (second operand of the compare is also register), try that one.
   603:           // TODO: Try to come up with some heuristic to figure out which
   604:           // feeder would benefit.
   605: 
   606:           if (feederReg == cmpReg1) {
   607:             if (!canBeFeederToNewValueJump(QII, QRI, MII, jmpPos, cmpPos, MF)) {
   608:               if (!isSecondOpReg)
   609:                 break;
   610:               else
   611:                 continue;
   612:             } else
   613:               foundFeeder = true;
   614:           }
   615: 
   616:           if (!foundFeeder && isSecondOpReg && feederReg == (unsigned)cmpOp2)
   617:             if (!canBeFeederToNewValueJump(QII, QRI, MII, jmpPos, cmpPos, MF))
   618:               break;
   619: 
   620:           if (isSecondOpReg) {
   621:             // In case of CMPLT, or CMPLTU, or EQ with the second register
   622:             // to newify, swap the operands.
   623:             unsigned COp = cmpInstr->getOpcode();
   624:             if ((COp == Hexagon::C2_cmpeq || COp == Hexagon::C4_cmpneq) &&
   625:                 (feederReg == (unsigned)cmpOp2)) {
   626:               unsigned tmp = cmpReg1;
   627:               cmpReg1 = cmpOp2;
   628:               cmpOp2 = tmp;
   629:             }
   630: 
   631:             // Now we have swapped the operands, all we need to check is,
   632:             // if the second operand (after swap) is the feeder.
   633:             // And if it is, make a note.
   634:             if (feederReg == (unsigned)cmpOp2)
   635:               isSecondOpNewified = true;
   636:           }
   637: 
   638:           // Now that we are moving feeder close the jump,
   639:           // make sure we are respecting the kill values of
   640:           // the operands of the feeder.
   641: 
   642:           auto TransferKills = [jmpPos,cmpPos] (MachineInstr &MI) {
   643:             for (MachineOperand &MO : MI.operands()) {
   644:               if (!MO.isReg() || !MO.isUse())
   645:                 continue;
   646:               Register UseR = MO.getReg();
   647:               for (auto I = std::next(MI.getIterator()); I != jmpPos; ++I) {
   648:                 if (I == cmpPos)
   649:                   continue;
   650:                 for (MachineOperand &Op : I->operands()) {
```
- EN: It declares or implements routines such as getOpcode, getReg, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getOpcode, getReg 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 651-700 / 第 651-700 行

```cpp
   651:                   if (!Op.isReg() || !Op.isUse() || !Op.isKill())
   652:                     continue;
   653:                   if (Op.getReg() != UseR)
   654:                     continue;
   655:                   // We found that there is kill of a use register
   656:                   // Set up a kill flag on the register
   657:                   Op.setIsKill(false);
   658:                   MO.setIsKill(true);
   659:                   return;
   660:                 }
   661:               }
   662:             }
   663:           };
   664: 
   665:           TransferKills(*feederPos);
   666:           TransferKills(*cmpPos);
   667:           bool MO1IsKill = cmpPos->killsRegister(cmpReg1, QRI);
   668:           bool MO2IsKill = isSecondOpReg && cmpPos->killsRegister(cmpOp2, QRI);
   669: 
   670:           MBB->splice(jmpPos, MI.getParent(), MI);
   671:           MBB->splice(jmpPos, MI.getParent(), cmpInstr);
   672:           DebugLoc dl = MI.getDebugLoc();
   673:           MachineInstr *NewMI;
   674: 
   675:           assert((isNewValueJumpCandidate(*cmpInstr)) &&
   676:                  "This compare is not a New Value Jump candidate.");
   677:           unsigned opc = getNewValueJumpOpcode(cmpInstr, cmpOp2,
   678:                                                isSecondOpNewified,
   679:                                                jmpTarget, MBPI);
   680:           if (invertPredicate)
   681:             opc = QII->getInvertedPredicatedOpcode(opc);
   682: 
   683:           if (isSecondOpReg)
   684:             NewMI = BuildMI(*MBB, jmpPos, dl, QII->get(opc))
   685:                         .addReg(cmpReg1, getKillRegState(MO1IsKill))
   686:                         .addReg(cmpOp2, getKillRegState(MO2IsKill))
   687:                         .addMBB(jmpTarget);
   688: 
   689:           else
   690:             NewMI = BuildMI(*MBB, jmpPos, dl, QII->get(opc))
   691:                         .addReg(cmpReg1, getKillRegState(MO1IsKill))
   692:                         .addImm(cmpOp2)
   693:                         .addMBB(jmpTarget);
   694: 
   695:           assert(NewMI && "New Value Jump Instruction Not created!");
   696:           (void)NewMI;
   697:           if (cmpInstr->getOperand(0).isReg() &&
   698:               cmpInstr->getOperand(0).isKill())
   699:             cmpInstr->getOperand(0).setIsKill(false);
   700:           if (cmpInstr->getOperand(1).isReg() &&
```
- EN: It declares or implements routines such as setIsKill, TransferKills, killsRegister, splice, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 setIsKill, TransferKills, killsRegister, splice, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 701-718 / 第 701-718 行

```cpp
   701:               cmpInstr->getOperand(1).isKill())
   702:             cmpInstr->getOperand(1).setIsKill(false);
   703:           cmpInstr->eraseFromParent();
   704:           jmpInstr->eraseFromParent();
   705:           ++nvjGenerated;
   706:           ++NumNVJGenerated;
   707:           break;
   708:         }
   709:       }
   710:     }
   711:   }
   712: 
   713:   return true;
   714: }
   715: 
   716: FunctionPass *llvm::createHexagonNewValueJump() {
   717:   return new HexagonNewValueJump();
   718: }
```
- EN: It declares or implements routines such as getOperand, eraseFromParent, llvm::createHexagonNewValueJump, HexagonNewValueJump, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonNewValueJump, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, eraseFromParent, llvm::createHexagonNewValueJump, HexagonNewValueJump 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonNewValueJump，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/InitializePasses.h, Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineBranchProbabilityInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, ... (29 total)`
- Hexagon symbols / Hexagon 符号: `HexagonNewValueJump, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, Hexagon_S2_lsr_r_p`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
