# HexagonMCCompound.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCCompound.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file is looks at a packet and tries to form compound insns
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及调度或成包。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //=== HexagonMCCompound.cpp - Hexagon Compound checker  -------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file is looks at a packet and tries to form compound insns
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "MCTargetDesc/HexagonBaseInfo.h"
    14: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    15: #include "MCTargetDesc/HexagonMCShuffler.h"
    16: #include "llvm/MC/MCContext.h"
    17: #include "llvm/MC/MCExpr.h"
    18: #include "llvm/MC/MCInst.h"
    19: #include "llvm/Support/Debug.h"
    20: #include "llvm/Support/ErrorHandling.h"
    21: #include "llvm/Support/raw_ostream.h"
    22: #include <cassert>
    23: #include <cstdint>
    24: 
    25: using namespace llvm;
    26: using namespace Hexagon;
    27: 
    28: #define DEBUG_TYPE "hexagon-mccompound"
    29: 
    30: enum OpcodeIndex {
    31:   fp0_jump_nt = 0,
    32:   fp0_jump_t,
    33:   fp1_jump_nt,
    34:   fp1_jump_t,
    35:   tp0_jump_nt,
    36:   tp0_jump_t,
    37:   tp1_jump_nt,
    38:   tp1_jump_t
    39: };
    40: 
    41: static const unsigned tstBitOpcode[8] = {
    42:     J4_tstbit0_fp0_jump_nt, J4_tstbit0_fp0_jump_t,  J4_tstbit0_fp1_jump_nt,
    43:     J4_tstbit0_fp1_jump_t,  J4_tstbit0_tp0_jump_nt, J4_tstbit0_tp0_jump_t,
    44:     J4_tstbit0_tp1_jump_nt, J4_tstbit0_tp1_jump_t};
    45: static const unsigned cmpeqBitOpcode[8] = {
    46:     J4_cmpeq_fp0_jump_nt, J4_cmpeq_fp0_jump_t,  J4_cmpeq_fp1_jump_nt,
    47:     J4_cmpeq_fp1_jump_t,  J4_cmpeq_tp0_jump_nt, J4_cmpeq_tp0_jump_t,
    48:     J4_cmpeq_tp1_jump_nt, J4_cmpeq_tp1_jump_t};
    49: static const unsigned cmpgtBitOpcode[8] = {
    50:     J4_cmpgt_fp0_jump_nt, J4_cmpgt_fp0_jump_t,  J4_cmpgt_fp1_jump_nt,
```
- EN: It imports headers such as MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCShuffler.h, llvm/MC/MCContext.h, ... (11 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as OpcodeIndex, which carry the state or API of this component.
- CN: 这里引入了 MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCShuffler.h, llvm/MC/MCContext.h, ... (11 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 OpcodeIndex 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:     J4_cmpgt_fp1_jump_t,  J4_cmpgt_tp0_jump_nt, J4_cmpgt_tp0_jump_t,
    52:     J4_cmpgt_tp1_jump_nt, J4_cmpgt_tp1_jump_t};
    53: static const unsigned cmpgtuBitOpcode[8] = {
    54:     J4_cmpgtu_fp0_jump_nt, J4_cmpgtu_fp0_jump_t,  J4_cmpgtu_fp1_jump_nt,
    55:     J4_cmpgtu_fp1_jump_t,  J4_cmpgtu_tp0_jump_nt, J4_cmpgtu_tp0_jump_t,
    56:     J4_cmpgtu_tp1_jump_nt, J4_cmpgtu_tp1_jump_t};
    57: static const unsigned cmpeqiBitOpcode[8] = {
    58:     J4_cmpeqi_fp0_jump_nt, J4_cmpeqi_fp0_jump_t,  J4_cmpeqi_fp1_jump_nt,
    59:     J4_cmpeqi_fp1_jump_t,  J4_cmpeqi_tp0_jump_nt, J4_cmpeqi_tp0_jump_t,
    60:     J4_cmpeqi_tp1_jump_nt, J4_cmpeqi_tp1_jump_t};
    61: static const unsigned cmpgtiBitOpcode[8] = {
    62:     J4_cmpgti_fp0_jump_nt, J4_cmpgti_fp0_jump_t,  J4_cmpgti_fp1_jump_nt,
    63:     J4_cmpgti_fp1_jump_t,  J4_cmpgti_tp0_jump_nt, J4_cmpgti_tp0_jump_t,
    64:     J4_cmpgti_tp1_jump_nt, J4_cmpgti_tp1_jump_t};
    65: static const unsigned cmpgtuiBitOpcode[8] = {
    66:     J4_cmpgtui_fp0_jump_nt, J4_cmpgtui_fp0_jump_t,  J4_cmpgtui_fp1_jump_nt,
    67:     J4_cmpgtui_fp1_jump_t,  J4_cmpgtui_tp0_jump_nt, J4_cmpgtui_tp0_jump_t,
    68:     J4_cmpgtui_tp1_jump_nt, J4_cmpgtui_tp1_jump_t};
    69: static const unsigned cmpeqn1BitOpcode[8] = {
    70:     J4_cmpeqn1_fp0_jump_nt, J4_cmpeqn1_fp0_jump_t,  J4_cmpeqn1_fp1_jump_nt,
    71:     J4_cmpeqn1_fp1_jump_t,  J4_cmpeqn1_tp0_jump_nt, J4_cmpeqn1_tp0_jump_t,
    72:     J4_cmpeqn1_tp1_jump_nt, J4_cmpeqn1_tp1_jump_t};
    73: static const unsigned cmpgtn1BitOpcode[8] = {
    74:     J4_cmpgtn1_fp0_jump_nt, J4_cmpgtn1_fp0_jump_t,  J4_cmpgtn1_fp1_jump_nt,
    75:     J4_cmpgtn1_fp1_jump_t,  J4_cmpgtn1_tp0_jump_nt, J4_cmpgtn1_tp0_jump_t,
    76:     J4_cmpgtn1_tp1_jump_nt, J4_cmpgtn1_tp1_jump_t,
    77: };
    78: 
    79: // enum HexagonII::CompoundGroup
    80: static unsigned getCompoundCandidateGroup(MCInst const &MI, bool IsExtended) {
    81:   MCRegister DstReg, SrcReg, Src1Reg, Src2Reg;
    82: 
    83:   switch (MI.getOpcode()) {
    84:   default:
    85:     return HexagonII::HCG_None;
    86:   //
    87:   // Compound pairs.
    88:   // "p0=cmp.eq(Rs16,Rt16); if (p0.new) jump:nt #r9:2"
    89:   // "Rd16=#U6 ; jump #r9:2"
    90:   // "Rd16=Rs16 ; jump #r9:2"
    91:   //
    92:   case Hexagon::C2_cmpeq:
    93:   case Hexagon::C2_cmpgt:
    94:   case Hexagon::C2_cmpgtu:
    95:     if (IsExtended)
    96:       return HexagonII::HCG_None;
    97:     DstReg = MI.getOperand(0).getReg();
    98:     Src1Reg = MI.getOperand(1).getReg();
    99:     Src2Reg = MI.getOperand(2).getReg();
   100:     if ((Hexagon::P0 == DstReg || Hexagon::P1 == DstReg) &&
```
- EN: It declares or implements routines such as getCompoundCandidateGroup, eq, getOperand, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getCompoundCandidateGroup, eq, getOperand 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:         HexagonMCInstrInfo::isIntRegForSubInst(Src1Reg) &&
   102:         HexagonMCInstrInfo::isIntRegForSubInst(Src2Reg))
   103:       return HexagonII::HCG_A;
   104:     break;
   105:   case Hexagon::C2_cmpeqi:
   106:   case Hexagon::C2_cmpgti:
   107:   case Hexagon::C2_cmpgtui:
   108:     if (IsExtended)
   109:       return HexagonII::HCG_None;
   110:     // P0 = cmp.eq(Rs,#u2)
   111:     DstReg = MI.getOperand(0).getReg();
   112:     SrcReg = MI.getOperand(1).getReg();
   113:     if ((Hexagon::P0 == DstReg || Hexagon::P1 == DstReg) &&
   114:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   115:         (HexagonMCInstrInfo::inRange<5>(MI, 2) ||
   116:          HexagonMCInstrInfo::minConstant(MI, 2) == -1))
   117:       return HexagonII::HCG_A;
   118:     break;
   119:   case Hexagon::A2_tfr:
   120:     if (IsExtended)
   121:       return HexagonII::HCG_None;
   122:     // Rd = Rs
   123:     DstReg = MI.getOperand(0).getReg();
   124:     SrcReg = MI.getOperand(1).getReg();
   125:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   126:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg))
   127:       return HexagonII::HCG_A;
   128:     break;
   129:   case Hexagon::A2_tfrsi:
   130:     if (IsExtended)
   131:       return HexagonII::HCG_None;
   132:     // Rd = #u6
   133:     DstReg = MI.getOperand(0).getReg();
   134:     if (HexagonMCInstrInfo::minConstant(MI, 1) <= 63 &&
   135:         HexagonMCInstrInfo::minConstant(MI, 1) >= 0 &&
   136:         HexagonMCInstrInfo::isIntRegForSubInst(DstReg))
   137:       return HexagonII::HCG_A;
   138:     break;
   139:   case Hexagon::S2_tstbit_i:
   140:     if (IsExtended)
   141:       return HexagonII::HCG_None;
   142:     DstReg = MI.getOperand(0).getReg();
   143:     Src1Reg = MI.getOperand(1).getReg();
   144:     if ((Hexagon::P0 == DstReg || Hexagon::P1 == DstReg) &&
   145:         HexagonMCInstrInfo::isIntRegForSubInst(Src1Reg) &&
   146:         HexagonMCInstrInfo::minConstant(MI, 2) == 0)
   147:       return HexagonII::HCG_A;
   148:     break;
   149:   // The fact that .new form is used pretty much guarantees
   150:   // that predicate register will match. Nevertheless,
```
- EN: It declares or implements routines such as eq, getOperand, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 eq, getOperand 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   // there could be some false positives without additional
   152:   // checking.
   153:   case Hexagon::J2_jumptnew:
   154:   case Hexagon::J2_jumpfnew:
   155:   case Hexagon::J2_jumptnewpt:
   156:   case Hexagon::J2_jumpfnewpt:
   157:     Src1Reg = MI.getOperand(0).getReg();
   158:     if (Hexagon::P0 == Src1Reg || Hexagon::P1 == Src1Reg)
   159:       return HexagonII::HCG_B;
   160:     break;
   161:   // Transfer and jump:
   162:   // Rd=#U6 ; jump #r9:2
   163:   // Rd=Rs ; jump #r9:2
   164:   // Do not test for jump range here.
   165:   case Hexagon::J2_jump:
   166:   case Hexagon::RESTORE_DEALLOC_RET_JMP_V4:
   167:     return HexagonII::HCG_C;
   168:     break;
   169:   }
   170: 
   171:   return HexagonII::HCG_None;
   172: }
   173: 
   174: /// getCompoundOp - Return the index from 0-7 into the above opcode lists.
   175: static unsigned getCompoundOp(MCInst const &HMCI) {
   176:   const MCOperand &Predicate = HMCI.getOperand(0);
   177:   MCRegister PredReg = Predicate.getReg();
   178: 
   179:   assert((PredReg == Hexagon::P0) || (PredReg == Hexagon::P1) ||
   180:          (PredReg == Hexagon::P2) || (PredReg == Hexagon::P3));
   181: 
   182:   switch (HMCI.getOpcode()) {
   183:   default:
   184:     llvm_unreachable("Expected match not found.\n");
   185:     break;
   186:   case Hexagon::J2_jumpfnew:
   187:     return (PredReg == Hexagon::P0) ? fp0_jump_nt : fp1_jump_nt;
   188:   case Hexagon::J2_jumpfnewpt:
   189:     return (PredReg == Hexagon::P0) ? fp0_jump_t : fp1_jump_t;
   190:   case Hexagon::J2_jumptnew:
   191:     return (PredReg == Hexagon::P0) ? tp0_jump_nt : tp1_jump_nt;
   192:   case Hexagon::J2_jumptnewpt:
   193:     return (PredReg == Hexagon::P0) ? tp0_jump_t : tp1_jump_t;
   194:   }
   195: }
   196: 
   197: static MCInst *getCompoundInsn(MCContext &Context, MCInst const &L,
   198:                                MCInst const &R) {
   199:   MCInst *CompoundInsn = nullptr;
   200:   unsigned compoundOpcode;
```
- EN: It declares or implements routines such as getOperand, getCompoundOp, getReg, assert, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, getCompoundOp, getReg, assert, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   MCOperand Rs, Rt;
   202:   int64_t Value;
   203:   bool Success;
   204: 
   205:   switch (L.getOpcode()) {
   206:   default:
   207:     LLVM_DEBUG(dbgs() << "Possible compound ignored\n");
   208:     return CompoundInsn;
   209: 
   210:   case Hexagon::A2_tfrsi:
   211:     Rt = L.getOperand(0);
   212:     compoundOpcode = J4_jumpseti;
   213:     CompoundInsn = Context.createMCInst();
   214:     CompoundInsn->setOpcode(compoundOpcode);
   215: 
   216:     CompoundInsn->addOperand(Rt);
   217:     CompoundInsn->addOperand(L.getOperand(1)); // Immediate
   218:     CompoundInsn->addOperand(R.getOperand(0)); // Jump target
   219:     break;
   220: 
   221:   case Hexagon::A2_tfr:
   222:     Rt = L.getOperand(0);
   223:     Rs = L.getOperand(1);
   224: 
   225:     compoundOpcode = J4_jumpsetr;
   226:     CompoundInsn = Context.createMCInst();
   227:     CompoundInsn->setOpcode(compoundOpcode);
   228:     CompoundInsn->addOperand(Rt);
   229:     CompoundInsn->addOperand(Rs);
   230:     CompoundInsn->addOperand(R.getOperand(0)); // Jump target.
   231: 
   232:     break;
   233: 
   234:   case Hexagon::C2_cmpeq:
   235:     LLVM_DEBUG(dbgs() << "CX: C2_cmpeq\n");
   236:     Rs = L.getOperand(1);
   237:     Rt = L.getOperand(2);
   238: 
   239:     compoundOpcode = cmpeqBitOpcode[getCompoundOp(R)];
   240:     CompoundInsn = Context.createMCInst();
   241:     CompoundInsn->setOpcode(compoundOpcode);
   242:     CompoundInsn->addOperand(Rs);
   243:     CompoundInsn->addOperand(Rt);
   244:     CompoundInsn->addOperand(R.getOperand(1));
   245:     break;
   246: 
   247:   case Hexagon::C2_cmpgt:
   248:     LLVM_DEBUG(dbgs() << "CX: C2_cmpgt\n");
   249:     Rs = L.getOperand(1);
   250:     Rt = L.getOperand(2);
```
- EN: It declares or implements routines such as LLVM_DEBUG, getOperand, createMCInst, setOpcode, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 LLVM_DEBUG, getOperand, createMCInst, setOpcode, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-300 / 第 251-300 行

```cpp
   251: 
   252:     compoundOpcode = cmpgtBitOpcode[getCompoundOp(R)];
   253:     CompoundInsn = Context.createMCInst();
   254:     CompoundInsn->setOpcode(compoundOpcode);
   255:     CompoundInsn->addOperand(Rs);
   256:     CompoundInsn->addOperand(Rt);
   257:     CompoundInsn->addOperand(R.getOperand(1));
   258:     break;
   259: 
   260:   case Hexagon::C2_cmpgtu:
   261:     LLVM_DEBUG(dbgs() << "CX: C2_cmpgtu\n");
   262:     Rs = L.getOperand(1);
   263:     Rt = L.getOperand(2);
   264: 
   265:     compoundOpcode = cmpgtuBitOpcode[getCompoundOp(R)];
   266:     CompoundInsn = Context.createMCInst();
   267:     CompoundInsn->setOpcode(compoundOpcode);
   268:     CompoundInsn->addOperand(Rs);
   269:     CompoundInsn->addOperand(Rt);
   270:     CompoundInsn->addOperand(R.getOperand(1));
   271:     break;
   272: 
   273:   case Hexagon::C2_cmpeqi:
   274:     LLVM_DEBUG(dbgs() << "CX: C2_cmpeqi\n");
   275:     Success = L.getOperand(2).getExpr()->evaluateAsAbsolute(Value);
   276:     (void)Success;
   277:     assert(Success);
   278:     if (Value == -1)
   279:       compoundOpcode = cmpeqn1BitOpcode[getCompoundOp(R)];
   280:     else
   281:       compoundOpcode = cmpeqiBitOpcode[getCompoundOp(R)];
   282: 
   283:     Rs = L.getOperand(1);
   284:     CompoundInsn = Context.createMCInst();
   285:     CompoundInsn->setOpcode(compoundOpcode);
   286:     CompoundInsn->addOperand(Rs);
   287:     CompoundInsn->addOperand(L.getOperand(2));
   288:     CompoundInsn->addOperand(R.getOperand(1));
   289:     break;
   290: 
   291:   case Hexagon::C2_cmpgti:
   292:     LLVM_DEBUG(dbgs() << "CX: C2_cmpgti\n");
   293:     Success = L.getOperand(2).getExpr()->evaluateAsAbsolute(Value);
   294:     (void)Success;
   295:     assert(Success);
   296:     if (Value == -1)
   297:       compoundOpcode = cmpgtn1BitOpcode[getCompoundOp(R)];
   298:     else
   299:       compoundOpcode = cmpgtiBitOpcode[getCompoundOp(R)];
   300: 
```
- EN: It declares or implements routines such as createMCInst, setOpcode, addOperand, LLVM_DEBUG, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 createMCInst, setOpcode, addOperand, LLVM_DEBUG, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 301-350 / 第 301-350 行

```cpp
   301:     Rs = L.getOperand(1);
   302:     CompoundInsn = Context.createMCInst();
   303:     CompoundInsn->setOpcode(compoundOpcode);
   304:     CompoundInsn->addOperand(Rs);
   305:     CompoundInsn->addOperand(L.getOperand(2));
   306:     CompoundInsn->addOperand(R.getOperand(1));
   307:     break;
   308: 
   309:   case Hexagon::C2_cmpgtui:
   310:     LLVM_DEBUG(dbgs() << "CX: C2_cmpgtui\n");
   311:     Rs = L.getOperand(1);
   312:     compoundOpcode = cmpgtuiBitOpcode[getCompoundOp(R)];
   313:     CompoundInsn = Context.createMCInst();
   314:     CompoundInsn->setOpcode(compoundOpcode);
   315:     CompoundInsn->addOperand(Rs);
   316:     CompoundInsn->addOperand(L.getOperand(2));
   317:     CompoundInsn->addOperand(R.getOperand(1));
   318:     break;
   319: 
   320:   case Hexagon::S2_tstbit_i:
   321:     LLVM_DEBUG(dbgs() << "CX: S2_tstbit_i\n");
   322:     Rs = L.getOperand(1);
   323:     compoundOpcode = tstBitOpcode[getCompoundOp(R)];
   324:     CompoundInsn = Context.createMCInst();
   325:     CompoundInsn->setOpcode(compoundOpcode);
   326:     CompoundInsn->addOperand(Rs);
   327:     CompoundInsn->addOperand(R.getOperand(1));
   328:     break;
   329:   }
   330: 
   331:   return CompoundInsn;
   332: }
   333: 
   334: /// Non-Symmetrical. See if these two instructions are fit for compound pair.
   335: static bool isOrderedCompoundPair(MCInst const &MIa, bool IsExtendedA,
   336:                                   MCInst const &MIb, bool IsExtendedB) {
   337:   unsigned MIaG = getCompoundCandidateGroup(MIa, IsExtendedA);
   338:   unsigned MIbG = getCompoundCandidateGroup(MIb, IsExtendedB);
   339:   // We have two candidates - check that this is the same register
   340:   // we are talking about.
   341:   unsigned Opca = MIa.getOpcode();
   342:   if (MIaG == HexagonII::HCG_A && MIbG == HexagonII::HCG_C &&
   343:       (Opca == Hexagon::A2_tfr || Opca == Hexagon::A2_tfrsi))
   344:     return true;
   345:   return ((MIaG == HexagonII::HCG_A && MIbG == HexagonII::HCG_B) &&
   346:           (MIa.getOperand(0).getReg() == MIb.getOperand(0).getReg()));
   347: }
   348: 
   349: static bool lookForCompound(MCInstrInfo const &MCII, MCContext &Context,
   350:                             MCInst &MCI) {
```
- EN: It declares or implements routines such as getOperand, createMCInst, setOpcode, addOperand, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, createMCInst, setOpcode, addOperand, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```cpp
   351:   assert(HexagonMCInstrInfo::isBundle(MCI));
   352:   bool JExtended = false;
   353:   for (MCInst::iterator J =
   354:            MCI.begin() + HexagonMCInstrInfo::bundleInstructionsOffset;
   355:        J != MCI.end(); ++J) {
   356:     MCInst const *JumpInst = J->getInst();
   357:     if (HexagonMCInstrInfo::isImmext(*JumpInst)) {
   358:       JExtended = true;
   359:       continue;
   360:     }
   361:     if (HexagonMCInstrInfo::getType(MCII, *JumpInst) == HexagonII::TypeJ) {
   362:       // Try to pair with another insn (B)undled with jump.
   363:       bool BExtended = false;
   364:       for (MCInst::iterator B =
   365:                MCI.begin() + HexagonMCInstrInfo::bundleInstructionsOffset;
   366:            B != MCI.end(); ++B) {
   367:         MCInst const *Inst = B->getInst();
   368:         if (JumpInst == Inst) {
   369:           BExtended = false;
   370:           continue;
   371:         }
   372:         if (HexagonMCInstrInfo::isImmext(*Inst)) {
   373:           BExtended = true;
   374:           continue;
   375:         }
   376:         LLVM_DEBUG(dbgs() << "J,B: " << JumpInst->getOpcode() << ","
   377:                           << Inst->getOpcode() << "\n");
   378:         if (isOrderedCompoundPair(*Inst, BExtended, *JumpInst, JExtended)) {
   379:           MCInst *CompoundInsn = getCompoundInsn(Context, *Inst, *JumpInst);
   380:           if (CompoundInsn) {
   381:             LLVM_DEBUG(dbgs() << "B: " << Inst->getOpcode() << ","
   382:                               << JumpInst->getOpcode() << " Compounds to "
   383:                               << CompoundInsn->getOpcode() << "\n");
   384:             J->setInst(CompoundInsn);
   385:             MCI.erase(B);
   386:             return true;
   387:           }
   388:         }
   389:         BExtended = false;
   390:       }
   391:     }
   392:     JExtended = false;
   393:   }
   394:   return false;
   395: }
   396: 
   397: /// tryCompound - Given a bundle check for compound insns when one
   398: /// is found update the contents of the bundle with the compound insn.
   399: /// If a compound instruction is found then the bundle will have one
   400: /// additional slot.
```
- EN: It declares or implements routines such as assert, end, getInst, LLVM_DEBUG, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, end, getInst, LLVM_DEBUG, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 401-433 / 第 401-433 行

```cpp
   401: void HexagonMCInstrInfo::tryCompound(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   402:                                      MCContext &Context, MCInst &MCI) {
   403:   assert(HexagonMCInstrInfo::isBundle(MCI) &&
   404:          "Non-Bundle where Bundle expected");
   405: 
   406:   // By definition a compound must have 2 insn.
   407:   if (MCI.size() < 2)
   408:     return;
   409: 
   410:   // Create a vector, needed to keep the order of jump instructions.
   411:   MCInst CheckList(MCI);
   412: 
   413:   // Keep the last known good bundle around in case the shuffle fails.
   414:   MCInst LastValidBundle(MCI);
   415: 
   416:   bool PreviouslyValid = llvm::HexagonMCShuffle(Context, false, MCII, STI, MCI);
   417: 
   418:   // Look for compounds until none are found, only update the bundle when
   419:   // a compound is found.
   420:   while (lookForCompound(MCII, Context, CheckList)) {
   421:     // Need to update the bundle.
   422:     MCI = CheckList;
   423: 
   424:     const bool IsValid = llvm::HexagonMCShuffle(Context, false, MCII, STI, MCI);
   425:     if (PreviouslyValid && !IsValid) {
   426:       LLVM_DEBUG(dbgs() << "Found ERROR\n");
   427:       MCI = LastValidBundle;
   428:     } else if (IsValid) {
   429:       LastValidBundle = MCI;
   430:       PreviouslyValid = true;
   431:     }
   432:   }
   433: }
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::tryCompound, assert, CheckList, LastValidBundle, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCShuffle, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::tryCompound, assert, CheckList, LastValidBundle, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCShuffle，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCShuffler.h, llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCInst.h, llvm/Support/Debug.h, llvm/Support/ErrorHandling.h, llvm/Support/raw_ostream.h, cassert, ... (11 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCCompound, HexagonBaseInfo, HexagonMCInstrInfo, HexagonMCShuffler, HexagonII, HexagonMCShuffle`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
