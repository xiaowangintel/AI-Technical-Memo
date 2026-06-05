# HexagonMCDuplexInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCDuplexInfo.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon MC-layer target description glue.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonMCDuplexInfo.cpp - Instruction bundle checking --------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements duplexing of instructions to reduce code size
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "HexagonMCExpr.h"
    14: #include "MCTargetDesc/HexagonBaseInfo.h"
    15: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    16: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    17: #include "llvm/ADT/SmallVector.h"
    18: #include "llvm/MC/MCSubtargetInfo.h"
    19: #include "llvm/Support/Debug.h"
    20: #include "llvm/Support/ErrorHandling.h"
    21: #include "llvm/Support/MathExtras.h"
    22: #include "llvm/Support/raw_ostream.h"
    23: #include <cassert>
    24: #include <cstdint>
    25: #include <iterator>
    26: #include <map>
    27: #include <utility>
    28: 
    29: using namespace llvm;
    30: using namespace Hexagon;
    31: 
    32: #define DEBUG_TYPE "hexagon-mcduplex-info"
    33: 
    34: // pair table of subInstructions with opcodes
    35: static const std::pair<unsigned, unsigned> opcodeData[] = {
    36:     std::make_pair((unsigned)SA1_addi, 0),
    37:     std::make_pair((unsigned)SA1_addrx, 6144),
    38:     std::make_pair((unsigned)SA1_addsp, 3072),
    39:     std::make_pair((unsigned)SA1_and1, 4608),
    40:     std::make_pair((unsigned)SA1_clrf, 6768),
    41:     std::make_pair((unsigned)SA1_clrfnew, 6736),
    42:     std::make_pair((unsigned)SA1_clrt, 6752),
    43:     std::make_pair((unsigned)SA1_clrtnew, 6720),
    44:     std::make_pair((unsigned)SA1_cmpeqi, 6400),
    45:     std::make_pair((unsigned)SA1_combine0i, 7168),
    46:     std::make_pair((unsigned)SA1_combine1i, 7176),
    47:     std::make_pair((unsigned)SA1_combine2i, 7184),
    48:     std::make_pair((unsigned)SA1_combine3i, 7192),
    49:     std::make_pair((unsigned)SA1_combinerz, 7432),
    50:     std::make_pair((unsigned)SA1_combinezr, 7424),
    51:     std::make_pair((unsigned)SA1_dec, 4864),
    52:     std::make_pair((unsigned)SA1_inc, 4352),
    53:     std::make_pair((unsigned)SA1_seti, 2048),
    54:     std::make_pair((unsigned)SA1_setin1, 6656),
    55:     std::make_pair((unsigned)SA1_sxtb, 5376),
    56:     std::make_pair((unsigned)SA1_sxth, 5120),
    57:     std::make_pair((unsigned)SA1_tfr, 4096),
    58:     std::make_pair((unsigned)SA1_zxtb, 5888),
    59:     std::make_pair((unsigned)SA1_zxth, 5632),
    60:     std::make_pair((unsigned)SL1_loadri_io, 0),
    61:     std::make_pair((unsigned)SL1_loadrub_io, 4096),
    62:     std::make_pair((unsigned)SL2_deallocframe, 7936),
    63:     std::make_pair((unsigned)SL2_jumpr31, 8128),
    64:     std::make_pair((unsigned)SL2_jumpr31_f, 8133),
    65:     std::make_pair((unsigned)SL2_jumpr31_fnew, 8135),
    66:     std::make_pair((unsigned)SL2_jumpr31_t, 8132),
    67:     std::make_pair((unsigned)SL2_jumpr31_tnew, 8134),
    68:     std::make_pair((unsigned)SL2_loadrb_io, 4096),
    69:     std::make_pair((unsigned)SL2_loadrd_sp, 7680),
    70:     std::make_pair((unsigned)SL2_loadrh_io, 0),
    71:     std::make_pair((unsigned)SL2_loadri_sp, 7168),
    72:     std::make_pair((unsigned)SL2_loadruh_io, 2048),
    73:     std::make_pair((unsigned)SL2_return, 8000),
    74:     std::make_pair((unsigned)SL2_return_f, 8005),
    75:     std::make_pair((unsigned)SL2_return_fnew, 8007),
    76:     std::make_pair((unsigned)SL2_return_t, 8004),
    77:     std::make_pair((unsigned)SL2_return_tnew, 8006),
    78:     std::make_pair((unsigned)SS1_storeb_io, 4096),
    79:     std::make_pair((unsigned)SS1_storew_io, 0),
    80:     std::make_pair((unsigned)SS2_allocframe, 7168),
    81:     std::make_pair((unsigned)SS2_storebi0, 4608),
    82:     std::make_pair((unsigned)SS2_storebi1, 4864),
    83:     std::make_pair((unsigned)SS2_stored_sp, 2560),
    84:     std::make_pair((unsigned)SS2_storeh_io, 0),
    85:     std::make_pair((unsigned)SS2_storew_sp, 2048),
    86:     std::make_pair((unsigned)SS2_storewi0, 4096),
    87:     std::make_pair((unsigned)SS2_storewi1, 4352)};
    88: 
    89: bool HexagonMCInstrInfo::isDuplexPairMatch(unsigned Ga, unsigned Gb) {
    90:   switch (Ga) {
    91:   case HexagonII::HSIG_None:
    92:   default:
    93:     return false;
    94:   case HexagonII::HSIG_L1:
    95:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_A);
    96:   case HexagonII::HSIG_L2:
    97:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_L2 ||
    98:             Gb == HexagonII::HSIG_A);
    99:   case HexagonII::HSIG_S1:
   100:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_L2 ||
```
- EN: It imports headers such as HexagonMCExpr.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (15 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonMCInstrInfo::isDuplexPairMatch, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonMCExpr.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (15 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonMCInstrInfo::isDuplexPairMatch 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-200 / 第 101-200 行

```cpp
   101:             Gb == HexagonII::HSIG_S1 || Gb == HexagonII::HSIG_A);
   102:   case HexagonII::HSIG_S2:
   103:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_L2 ||
   104:             Gb == HexagonII::HSIG_S1 || Gb == HexagonII::HSIG_S2 ||
   105:             Gb == HexagonII::HSIG_A);
   106:   case HexagonII::HSIG_A:
   107:     return (Gb == HexagonII::HSIG_A);
   108:   case HexagonII::HSIG_Compound:
   109:     return (Gb == HexagonII::HSIG_Compound);
   110:   }
   111:   return false;
   112: }
   113: 
   114: unsigned HexagonMCInstrInfo::iClassOfDuplexPair(unsigned Ga, unsigned Gb) {
   115:   switch (Ga) {
   116:   case HexagonII::HSIG_None:
   117:   default:
   118:     break;
   119:   case HexagonII::HSIG_L1:
   120:     switch (Gb) {
   121:     default:
   122:       break;
   123:     case HexagonII::HSIG_L1:
   124:       return 0;
   125:     case HexagonII::HSIG_A:
   126:       return 0x4;
   127:     }
   128:     break;
   129:   case HexagonII::HSIG_L2:
   130:     switch (Gb) {
   131:     default:
   132:       break;
   133:     case HexagonII::HSIG_L1:
   134:       return 0x1;
   135:     case HexagonII::HSIG_L2:
   136:       return 0x2;
   137:     case HexagonII::HSIG_A:
   138:       return 0x5;
   139:     }
   140:     break;
   141:   case HexagonII::HSIG_S1:
   142:     switch (Gb) {
   143:     default:
   144:       break;
   145:     case HexagonII::HSIG_L1:
   146:       return 0x8;
   147:     case HexagonII::HSIG_L2:
   148:       return 0x9;
   149:     case HexagonII::HSIG_S1:
   150:       return 0xA;
   151:     case HexagonII::HSIG_A:
   152:       return 0x6;
   153:     }
   154:     break;
   155:   case HexagonII::HSIG_S2:
   156:     switch (Gb) {
   157:     default:
   158:       break;
   159:     case HexagonII::HSIG_L1:
   160:       return 0xC;
   161:     case HexagonII::HSIG_L2:
   162:       return 0xD;
   163:     case HexagonII::HSIG_S1:
   164:       return 0xB;
   165:     case HexagonII::HSIG_S2:
   166:       return 0xE;
   167:     case HexagonII::HSIG_A:
   168:       return 0x7;
   169:     }
   170:     break;
   171:   case HexagonII::HSIG_A:
   172:     switch (Gb) {
   173:     default:
   174:       break;
   175:     case HexagonII::HSIG_A:
   176:       return 0x3;
   177:     }
   178:     break;
   179:   case HexagonII::HSIG_Compound:
   180:     switch (Gb) {
   181:     case HexagonII::HSIG_Compound:
   182:       return 0xFFFFFFFF;
   183:     }
   184:     break;
   185:   }
   186:   return 0xFFFFFFFF;
   187: }
   188: 
   189: unsigned HexagonMCInstrInfo::getDuplexCandidateGroup(MCInst const &MCI) {
   190:   MCRegister DstReg, PredReg, SrcReg, Src1Reg, Src2Reg;
   191: 
   192:   switch (MCI.getOpcode()) {
   193:   default:
   194:     return HexagonII::HSIG_None;
   195:   //
   196:   // Group L1:
   197:   //
   198:   // Rd = memw(Rs+#u4:2)
   199:   // Rd = memub(Rs+#u4:0)
   200:   case Hexagon::L2_loadri_io:
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::iClassOfDuplexPair, HexagonMCInstrInfo::getDuplexCandidateGroup, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::iClassOfDuplexPair, HexagonMCInstrInfo::getDuplexCandidateGroup 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 201-300 / 第 201-300 行

```cpp
   201:     DstReg = MCI.getOperand(0).getReg();
   202:     SrcReg = MCI.getOperand(1).getReg();
   203:     // Special case this one from Group L2.
   204:     // Rd = memw(r29+#u5:2)
   205:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg)) {
   206:       if (HexagonMCInstrInfo::isIntReg(SrcReg) &&
   207:           Hexagon::R29 == SrcReg && inRange<5, 2>(MCI, 2)) {
   208:         return HexagonII::HSIG_L2;
   209:       }
   210:       // Rd = memw(Rs+#u4:2)
   211:       if (HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   212:           inRange<4, 2>(MCI, 2)) {
   213:         return HexagonII::HSIG_L1;
   214:       }
   215:     }
   216:     break;
   217:   case Hexagon::L2_loadrub_io:
   218:     // Rd = memub(Rs+#u4:0)
   219:     DstReg = MCI.getOperand(0).getReg();
   220:     SrcReg = MCI.getOperand(1).getReg();
   221:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   222:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   223:         inRange<4>(MCI, 2)) {
   224:       return HexagonII::HSIG_L1;
   225:     }
   226:     break;
   227:   //
   228:   // Group L2:
   229:   //
   230:   // Rd = memh/memuh(Rs+#u3:1)
   231:   // Rd = memb(Rs+#u3:0)
   232:   // Rd = memw(r29+#u5:2) - Handled above.
   233:   // Rdd = memd(r29+#u5:3)
   234:   // deallocframe
   235:   // [if ([!]p0[.new])] dealloc_return
   236:   // [if ([!]p0[.new])] jumpr r31
   237:   case Hexagon::L2_loadrh_io:
   238:   case Hexagon::L2_loadruh_io:
   239:     // Rd = memh/memuh(Rs+#u3:1)
   240:     DstReg = MCI.getOperand(0).getReg();
   241:     SrcReg = MCI.getOperand(1).getReg();
   242:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   243:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   244:         inRange<3, 1>(MCI, 2)) {
   245:       return HexagonII::HSIG_L2;
   246:     }
   247:     break;
   248:   case Hexagon::L2_loadrb_io:
   249:     // Rd = memb(Rs+#u3:0)
   250:     DstReg = MCI.getOperand(0).getReg();
   251:     SrcReg = MCI.getOperand(1).getReg();
   252:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   253:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   254:         inRange<3>(MCI, 2)) {
   255:       return HexagonII::HSIG_L2;
   256:     }
   257:     break;
   258:   case Hexagon::L2_loadrd_io:
   259:     // Rdd = memd(r29+#u5:3)
   260:     DstReg = MCI.getOperand(0).getReg();
   261:     SrcReg = MCI.getOperand(1).getReg();
   262:     if (HexagonMCInstrInfo::isDblRegForSubInst(DstReg) &&
   263:         HexagonMCInstrInfo::isIntReg(SrcReg) && Hexagon::R29 == SrcReg &&
   264:         inRange<5, 3>(MCI, 2)) {
   265:       return HexagonII::HSIG_L2;
   266:     }
   267:     break;
   268: 
   269:   case Hexagon::L4_return:
   270:   case Hexagon::L2_deallocframe:
   271:     return HexagonII::HSIG_L2;
   272: 
   273:   case Hexagon::EH_RETURN_JMPR:
   274:   case Hexagon::J2_jumpr:
   275:   case Hexagon::PS_jmpret:
   276:     // jumpr r31
   277:     // Actual form JMPR implicit-def %pc, implicit %r31, implicit internal %r0.
   278:     DstReg = MCI.getOperand(0).getReg();
   279:     if (Hexagon::R31 == DstReg)
   280:       return HexagonII::HSIG_L2;
   281:     break;
   282: 
   283:   case Hexagon::J2_jumprt:
   284:   case Hexagon::J2_jumprf:
   285:   case Hexagon::J2_jumprtnew:
   286:   case Hexagon::J2_jumprfnew:
   287:   case Hexagon::PS_jmprett:
   288:   case Hexagon::PS_jmpretf:
   289:   case Hexagon::PS_jmprettnew:
   290:   case Hexagon::PS_jmpretfnew:
   291:   case Hexagon::PS_jmprettnewpt:
   292:   case Hexagon::PS_jmpretfnewpt:
   293:     DstReg = MCI.getOperand(1).getReg();
   294:     SrcReg = MCI.getOperand(0).getReg();
   295:     // [if ([!]p0[.new])] jumpr r31
   296:     if ((Hexagon::P0 == SrcReg) && (Hexagon::R31 == DstReg)) {
   297:       return HexagonII::HSIG_L2;
   298:     }
   299:     break;
   300:   case Hexagon::L4_return_t:
```
- EN: It declares or implements routines such as getOperand, memw, memub, memuh, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, memw, memub, memuh, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:   case Hexagon::L4_return_f:
   302:   case Hexagon::L4_return_tnew_pnt:
   303:   case Hexagon::L4_return_fnew_pnt:
   304:     // [if ([!]p0[.new])] dealloc_return
   305:     SrcReg = MCI.getOperand(1).getReg();
   306:     if (Hexagon::P0 == SrcReg) {
   307:       return HexagonII::HSIG_L2;
   308:     }
   309:     break;
   310:   //
   311:   // Group S1:
   312:   //
   313:   // memw(Rs+#u4:2) = Rt
   314:   // memb(Rs+#u4:0) = Rt
   315:   case Hexagon::S2_storeri_io:
   316:     // Special case this one from Group S2.
   317:     // memw(r29+#u5:2) = Rt
   318:     Src1Reg = MCI.getOperand(0).getReg();
   319:     Src2Reg = MCI.getOperand(2).getReg();
   320:     if (HexagonMCInstrInfo::isIntReg(Src1Reg) &&
   321:         HexagonMCInstrInfo::isIntRegForSubInst(Src2Reg) &&
   322:         Hexagon::R29 == Src1Reg && inRange<5, 2>(MCI, 1)) {
   323:       return HexagonII::HSIG_S2;
   324:     }
   325:     // memw(Rs+#u4:2) = Rt
   326:     if (HexagonMCInstrInfo::isIntRegForSubInst(Src1Reg) &&
   327:         HexagonMCInstrInfo::isIntRegForSubInst(Src2Reg) &&
   328:         inRange<4, 2>(MCI, 1)) {
   329:       return HexagonII::HSIG_S1;
   330:     }
   331:     break;
   332:   case Hexagon::S2_storerb_io:
   333:     // memb(Rs+#u4:0) = Rt
   334:     Src1Reg = MCI.getOperand(0).getReg();
   335:     Src2Reg = MCI.getOperand(2).getReg();
   336:     if (HexagonMCInstrInfo::isIntRegForSubInst(Src1Reg) &&
   337:         HexagonMCInstrInfo::isIntRegForSubInst(Src2Reg) &&
   338:         inRange<4>(MCI, 1)) {
   339:       return HexagonII::HSIG_S1;
   340:     }
   341:     break;
   342:   //
   343:   // Group S2:
   344:   //
   345:   // memh(Rs+#u3:1) = Rt
   346:   // memw(r29+#u5:2) = Rt
   347:   // memd(r29+#s6:3) = Rtt
   348:   // memw(Rs+#u4:2) = #U1
   349:   // memb(Rs+#u4) = #U1
   350:   // allocframe(#u5:3)
   351:   case Hexagon::S2_storerh_io:
   352:     // memh(Rs+#u3:1) = Rt
   353:     Src1Reg = MCI.getOperand(0).getReg();
   354:     Src2Reg = MCI.getOperand(2).getReg();
   355:     if (HexagonMCInstrInfo::isIntRegForSubInst(Src1Reg) &&
   356:         HexagonMCInstrInfo::isIntRegForSubInst(Src2Reg) &&
   357:         inRange<3, 1>(MCI, 1)) {
   358:       return HexagonII::HSIG_S2;
   359:     }
   360:     break;
   361:   case Hexagon::S2_storerd_io:
   362:     // memd(r29+#s6:3) = Rtt
   363:     Src1Reg = MCI.getOperand(0).getReg();
   364:     Src2Reg = MCI.getOperand(2).getReg();
   365:     if (HexagonMCInstrInfo::isDblRegForSubInst(Src2Reg) &&
   366:         HexagonMCInstrInfo::isIntReg(Src1Reg) && Hexagon::R29 == Src1Reg &&
   367:         inSRange<6, 3>(MCI, 1)) {
   368:       return HexagonII::HSIG_S2;
   369:     }
   370:     break;
   371:   case Hexagon::S4_storeiri_io:
   372:     // memw(Rs+#u4:2) = #U1
   373:     Src1Reg = MCI.getOperand(0).getReg();
   374:     if (HexagonMCInstrInfo::isIntRegForSubInst(Src1Reg) &&
   375:         inRange<4, 2>(MCI, 1) && inRange<1>(MCI, 2)) {
   376:       return HexagonII::HSIG_S2;
   377:     }
   378:     break;
   379:   case Hexagon::S4_storeirb_io:
   380:     // memb(Rs+#u4) = #U1
   381:     Src1Reg = MCI.getOperand(0).getReg();
   382:     if (HexagonMCInstrInfo::isIntRegForSubInst(Src1Reg) &&
   383:         inRange<4>(MCI, 1) && inRange<1>(MCI, 2)) {
   384:       return HexagonII::HSIG_S2;
   385:     }
   386:     break;
   387:   case Hexagon::S2_allocframe:
   388:     if (inRange<5, 3>(MCI, 2))
   389:       return HexagonII::HSIG_S2;
   390:     break;
   391:   //
   392:   // Group A:
   393:   //
   394:   // Rx = add(Rx,#s7)
   395:   // Rd = Rs
   396:   // Rd = #u6
   397:   // Rd = #-1
   398:   // if ([!]P0[.new]) Rd = #0
   399:   // Rd = add(r29,#u6:2)
   400:   // Rx = add(Rx,Rs)
```
- EN: It declares or implements routines such as memw, getOperand, memb, memh, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 memw, getOperand, memb, memh, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:   // P0 = cmp.eq(Rs,#u2)
   402:   // Rdd = combine(#0,Rs)
   403:   // Rdd = combine(Rs,#0)
   404:   // Rdd = combine(#u2,#U2)
   405:   // Rd = add(Rs,#1)
   406:   // Rd = add(Rs,#-1)
   407:   // Rd = sxth/sxtb/zxtb/zxth(Rs)
   408:   // Rd = and(Rs,#1)
   409:   case Hexagon::A2_addi:
   410:     DstReg = MCI.getOperand(0).getReg();
   411:     SrcReg = MCI.getOperand(1).getReg();
   412:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg)) {
   413:       // Rd = add(r29,#u6:2)
   414:       if (HexagonMCInstrInfo::isIntReg(SrcReg) && Hexagon::R29 == SrcReg &&
   415:           inRange<6, 2>(MCI, 2)) {
   416:         return HexagonII::HSIG_A;
   417:       }
   418:       // Rx = add(Rx,#s7)
   419:       if (DstReg == SrcReg) {
   420:         return HexagonII::HSIG_A;
   421:       }
   422:       // Rd = add(Rs,#1)
   423:       // Rd = add(Rs,#-1)
   424:       if (HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   425:           (minConstant(MCI, 2) == 1 || minConstant(MCI, 2) == -1)) {
   426:         return HexagonII::HSIG_A;
   427:       }
   428:     }
   429:     break;
   430:   case Hexagon::A2_add:
   431:     // Rx = add(Rx,Rs)
   432:     DstReg = MCI.getOperand(0).getReg();
   433:     Src1Reg = MCI.getOperand(1).getReg();
   434:     Src2Reg = MCI.getOperand(2).getReg();
   435:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) && (DstReg == Src1Reg) &&
   436:         HexagonMCInstrInfo::isIntRegForSubInst(Src2Reg)) {
   437:       return HexagonII::HSIG_A;
   438:     }
   439:     break;
   440:   case Hexagon::A2_andir:
   441:     DstReg = MCI.getOperand(0).getReg();
   442:     SrcReg = MCI.getOperand(1).getReg();
   443:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   444:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   445:         (minConstant(MCI, 2) == 1 || minConstant(MCI, 2) == 255)) {
   446:       return HexagonII::HSIG_A;
   447:     }
   448:     break;
   449:   case Hexagon::A2_tfr:
   450:     // Rd = Rs
   451:     DstReg = MCI.getOperand(0).getReg();
   452:     SrcReg = MCI.getOperand(1).getReg();
   453:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   454:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg)) {
   455:       return HexagonII::HSIG_A;
   456:     }
   457:     break;
   458:   case Hexagon::A2_tfrsi:
   459:     DstReg = MCI.getOperand(0).getReg();
   460: 
   461:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg)) {
   462:       return HexagonII::HSIG_A;
   463:     }
   464:     break;
   465:   case Hexagon::C2_cmoveit:
   466:   case Hexagon::C2_cmovenewit:
   467:   case Hexagon::C2_cmoveif:
   468:   case Hexagon::C2_cmovenewif:
   469:     // if ([!]P0[.new]) Rd = #0
   470:     // Actual form:
   471:     // %r16 = C2_cmovenewit internal %p0, 0, implicit undef %r16;
   472:     DstReg = MCI.getOperand(0).getReg();  // Rd
   473:     PredReg = MCI.getOperand(1).getReg(); // P0
   474:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   475:         Hexagon::P0 == PredReg && minConstant(MCI, 2) == 0) {
   476:       return HexagonII::HSIG_A;
   477:     }
   478:     break;
   479:   case Hexagon::C2_cmpeqi:
   480:     // P0 = cmp.eq(Rs,#u2)
   481:     DstReg = MCI.getOperand(0).getReg();
   482:     SrcReg = MCI.getOperand(1).getReg();
   483:     if (Hexagon::P0 == DstReg &&
   484:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   485:         inRange<2>(MCI, 2)) {
   486:       return HexagonII::HSIG_A;
   487:     }
   488:     break;
   489:   case Hexagon::A2_combineii:
   490:   case Hexagon::A4_combineii:
   491:     // Rdd = combine(#u2,#U2)
   492:     DstReg = MCI.getOperand(0).getReg();
   493:     if (HexagonMCInstrInfo::isDblRegForSubInst(DstReg) &&
   494:         inRange<2>(MCI, 1) && inRange<2>(MCI, 2)) {
   495:       return HexagonII::HSIG_A;
   496:     }
   497:     break;
   498:   case Hexagon::A4_combineri:
   499:     // Rdd = combine(Rs,#0)
   500:     DstReg = MCI.getOperand(0).getReg();
```
- EN: It declares or implements routines such as eq, getOperand, add, combine, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 eq, getOperand, add, combine 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:     SrcReg = MCI.getOperand(1).getReg();
   502:     if (HexagonMCInstrInfo::isDblRegForSubInst(DstReg) &&
   503:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   504:         minConstant(MCI, 2) == 0) {
   505:       return HexagonII::HSIG_A;
   506:     }
   507:     break;
   508:   case Hexagon::A4_combineir:
   509:     // Rdd = combine(#0,Rs)
   510:     DstReg = MCI.getOperand(0).getReg();
   511:     SrcReg = MCI.getOperand(2).getReg();
   512:     if (HexagonMCInstrInfo::isDblRegForSubInst(DstReg) &&
   513:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg) &&
   514:         minConstant(MCI, 1) == 0) {
   515:       return HexagonII::HSIG_A;
   516:     }
   517:     break;
   518:   case Hexagon::A2_sxtb:
   519:   case Hexagon::A2_sxth:
   520:   case Hexagon::A2_zxtb:
   521:   case Hexagon::A2_zxth:
   522:     // Rd = sxth/sxtb/zxtb/zxth(Rs)
   523:     DstReg = MCI.getOperand(0).getReg();
   524:     SrcReg = MCI.getOperand(1).getReg();
   525:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg) &&
   526:         HexagonMCInstrInfo::isIntRegForSubInst(SrcReg)) {
   527:       return HexagonII::HSIG_A;
   528:     }
   529:     break;
   530:   }
   531: 
   532:   return HexagonII::HSIG_None;
   533: }
   534: 
   535: bool HexagonMCInstrInfo::subInstWouldBeExtended(MCInst const &potentialDuplex) {
   536:   MCRegister DstReg, SrcReg;
   537:   switch (potentialDuplex.getOpcode()) {
   538:   case Hexagon::A2_addi:
   539:     // testing for case of: Rx = add(Rx,#s7)
   540:     DstReg = potentialDuplex.getOperand(0).getReg();
   541:     SrcReg = potentialDuplex.getOperand(1).getReg();
   542:     if (DstReg == SrcReg && HexagonMCInstrInfo::isIntRegForSubInst(DstReg)) {
   543:       int64_t Value;
   544:       if (!potentialDuplex.getOperand(2).getExpr()->evaluateAsAbsolute(Value))
   545:         return true;
   546:       if (!isShiftedInt<7, 0>(Value))
   547:         return true;
   548:     }
   549:     break;
   550:   case Hexagon::A2_tfrsi:
   551:     DstReg = potentialDuplex.getOperand(0).getReg();
   552: 
   553:     if (HexagonMCInstrInfo::isIntRegForSubInst(DstReg)) {
   554:       int64_t Value;
   555:       if (!potentialDuplex.getOperand(1).getExpr()->evaluateAsAbsolute(Value))
   556:         return true;
   557:       // Check for case of Rd = #-1.
   558:       if (Value == -1)
   559:         return false;
   560:       // Check for case of Rd = #u6.
   561:       if (!isShiftedUInt<6, 0>(Value))
   562:         return true;
   563:     }
   564:     break;
   565:   default:
   566:     break;
   567:   }
   568:   return false;
   569: }
   570: 
   571: /// non-Symmetrical. See if these two instructions are fit for duplex pair.
   572: bool HexagonMCInstrInfo::isOrderedDuplexPair(MCInstrInfo const &MCII,
   573:                                              MCInst const &MIa, bool ExtendedA,
   574:                                              MCInst const &MIb, bool ExtendedB,
   575:                                              bool bisReversable,
   576:                                              MCSubtargetInfo const &STI) {
   577:   // Slot 1 cannot be extended in duplexes PRM 10.5
   578:   if (ExtendedA)
   579:     return false;
   580:   // Only A2_addi and A2_tfrsi can be extended in duplex form PRM 10.5
   581:   if (ExtendedB) {
   582:     unsigned Opcode = MIb.getOpcode();
   583:     if ((Opcode != Hexagon::A2_addi) && (Opcode != Hexagon::A2_tfrsi))
   584:       return false;
   585:   }
   586:   unsigned MIaG = HexagonMCInstrInfo::getDuplexCandidateGroup(MIa),
   587:            MIbG = HexagonMCInstrInfo::getDuplexCandidateGroup(MIb);
   588: 
   589:   static std::map<unsigned, unsigned> subinstOpcodeMap(std::begin(opcodeData),
   590:                                                        std::end(opcodeData));
   591: 
   592:   // If a duplex contains 2 insns in the same group, the insns must be
   593:   // ordered such that the numerically smaller opcode is in slot 1.
   594:   if ((MIaG != HexagonII::HSIG_None) && (MIaG == MIbG) && bisReversable) {
   595:     MCInst SubInst0 = HexagonMCInstrInfo::deriveSubInst(MIa);
   596:     MCInst SubInst1 = HexagonMCInstrInfo::deriveSubInst(MIb);
   597: 
   598:     unsigned zeroedSubInstS0 =
   599:         subinstOpcodeMap.find(SubInst0.getOpcode())->second;
   600:     unsigned zeroedSubInstS1 =
```
- EN: It declares or implements routines such as getOperand, combine, zxth, HexagonMCInstrInfo::subInstWouldBeExtended, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, combine, zxth, HexagonMCInstrInfo::subInstWouldBeExtended, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:         subinstOpcodeMap.find(SubInst1.getOpcode())->second;
   602: 
   603:     if (zeroedSubInstS0 < zeroedSubInstS1)
   604:       // subinstS0 (maps to slot 0) must be greater than
   605:       // subinstS1 (maps to slot 1)
   606:       return false;
   607:   }
   608: 
   609:   // allocframe must always be in slot 0
   610:   if (MIb.getOpcode() == Hexagon::S2_allocframe)
   611:     return false;
   612: 
   613:   if ((MIaG != HexagonII::HSIG_None) && (MIbG != HexagonII::HSIG_None)) {
   614:     // Prevent 2 instructions with extenders from duplexing
   615:     // Note that MIb (slot1) can be extended and MIa (slot0)
   616:     //   can never be extended
   617:     if (subInstWouldBeExtended(MIa))
   618:       return false;
   619: 
   620:     // If duplexing produces an extender, but the original did not
   621:     //   have an extender, do not duplex.
   622:     if (subInstWouldBeExtended(MIb) && !ExtendedB)
   623:       return false;
   624:   }
   625: 
   626:   // If jumpr r31 appears, it must be in slot 0, and never slot 1 (MIb).
   627:   if (MIbG == HexagonII::HSIG_L2) {
   628:     if ((MIb.getNumOperands() > 1) && MIb.getOperand(1).isReg() &&
   629:         (MIb.getOperand(1).getReg() == Hexagon::R31))
   630:       return false;
   631:     if ((MIb.getNumOperands() > 0) && MIb.getOperand(0).isReg() &&
   632:         (MIb.getOperand(0).getReg() == Hexagon::R31))
   633:       return false;
   634:   }
   635: 
   636:   if (STI.getCPU().equals_insensitive("hexagonv5") ||
   637:       STI.getCPU().equals_insensitive("hexagonv55") ||
   638:       STI.getCPU().equals_insensitive("hexagonv60")) {
   639:     // If a store appears, it must be in slot 0 (MIa) 1st, and then slot 1 (MIb);
   640:     //   therefore, not duplexable if slot 1 is a store, and slot 0 is not.
   641:     if ((MIbG == HexagonII::HSIG_S1) || (MIbG == HexagonII::HSIG_S2)) {
   642:       if ((MIaG != HexagonII::HSIG_S1) && (MIaG != HexagonII::HSIG_S2))
   643:         return false;
   644:     }
   645:   }
   646: 
   647:   return (isDuplexPairMatch(MIaG, MIbG));
   648: }
   649: 
   650: /// Symmetrical. See if these two instructions are fit for duplex pair.
   651: bool HexagonMCInstrInfo::isDuplexPair(MCInst const &MIa, MCInst const &MIb) {
   652:   unsigned MIaG = getDuplexCandidateGroup(MIa),
   653:            MIbG = getDuplexCandidateGroup(MIb);
   654:   return (isDuplexPairMatch(MIaG, MIbG) || isDuplexPairMatch(MIbG, MIaG));
   655: }
   656: 
   657: inline static void addOps(MCInst &subInstPtr, MCInst const &Inst,
   658:                           unsigned opNum) {
   659:   if (Inst.getOperand(opNum).isReg()) {
   660:     switch (Inst.getOperand(opNum).getReg().id()) {
   661:     default:
   662:       llvm_unreachable("Not Duplexable Register");
   663:       break;
   664:     case Hexagon::R0:
   665:     case Hexagon::R1:
   666:     case Hexagon::R2:
   667:     case Hexagon::R3:
   668:     case Hexagon::R4:
   669:     case Hexagon::R5:
   670:     case Hexagon::R6:
   671:     case Hexagon::R7:
   672:     case Hexagon::D0:
   673:     case Hexagon::D1:
   674:     case Hexagon::D2:
   675:     case Hexagon::D3:
   676:     case Hexagon::R16:
   677:     case Hexagon::R17:
   678:     case Hexagon::R18:
   679:     case Hexagon::R19:
   680:     case Hexagon::R20:
   681:     case Hexagon::R21:
   682:     case Hexagon::R22:
   683:     case Hexagon::R23:
   684:     case Hexagon::D8:
   685:     case Hexagon::D9:
   686:     case Hexagon::D10:
   687:     case Hexagon::D11:
   688:     case Hexagon::P0:
   689:       subInstPtr.addOperand(Inst.getOperand(opNum));
   690:       break;
   691:     }
   692:   } else
   693:     subInstPtr.addOperand(Inst.getOperand(opNum));
   694: }
   695: 
   696: MCInst HexagonMCInstrInfo::deriveSubInst(MCInst const &Inst) {
   697:   MCInst Result;
   698:   Result.setLoc(Inst.getLoc());
   699:   bool Absolute;
   700:   int64_t Value;
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::isDuplexPair, getDuplexCandidateGroup, addOps, llvm_unreachable, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::isDuplexPair, getDuplexCandidateGroup, addOps, llvm_unreachable, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:   switch (Inst.getOpcode()) {
   702:   default:
   703:     // dbgs() << "opcode: "<< Inst->getOpcode() << "\n";
   704:     llvm_unreachable("Unimplemented subinstruction \n");
   705:     break;
   706:   case Hexagon::A2_addi:
   707:     Absolute = Inst.getOperand(2).getExpr()->evaluateAsAbsolute(Value);
   708:     if (Absolute) {
   709:       if (Value == 1) {
   710:         Result.setOpcode(Hexagon::SA1_inc);
   711:         addOps(Result, Inst, 0);
   712:         addOps(Result, Inst, 1);
   713:         break;
   714:       } //  1,2 SUBInst $Rd = add($Rs, #1)
   715:       if (Value == -1) {
   716:         Result.setOpcode(Hexagon::SA1_dec);
   717:         addOps(Result, Inst, 0);
   718:         addOps(Result, Inst, 1);
   719:         addOps(Result, Inst, 2);
   720:         break;
   721:       } //  1,2 SUBInst $Rd = add($Rs,#-1)
   722:       if (Inst.getOperand(1).getReg() == Hexagon::R29) {
   723:         Result.setOpcode(Hexagon::SA1_addsp);
   724:         addOps(Result, Inst, 0);
   725:         addOps(Result, Inst, 2);
   726:         break;
   727:       } //  1,3 SUBInst $Rd = add(r29, #$u6_2)
   728:     }
   729:     Result.setOpcode(Hexagon::SA1_addi);
   730:     addOps(Result, Inst, 0);
   731:     addOps(Result, Inst, 1);
   732:     addOps(Result, Inst, 2);
   733:     break; //    1,2,3 SUBInst $Rx = add($Rx, #$s7)
   734:   case Hexagon::A2_add:
   735:     Result.setOpcode(Hexagon::SA1_addrx);
   736:     addOps(Result, Inst, 0);
   737:     addOps(Result, Inst, 1);
   738:     addOps(Result, Inst, 2);
   739:     break; //    1,2,3 SUBInst $Rx = add($_src_, $Rs)
   740:   case Hexagon::S2_allocframe:
   741:     Result.setOpcode(Hexagon::SS2_allocframe);
   742:     addOps(Result, Inst, 2);
   743:     break; //    1 SUBInst allocframe(#$u5_3)
   744:   case Hexagon::A2_andir:
   745:     if (minConstant(Inst, 2) == 255) {
   746:       Result.setOpcode(Hexagon::SA1_zxtb);
   747:       addOps(Result, Inst, 0);
   748:       addOps(Result, Inst, 1);
   749:       break; //    1,2    $Rd = and($Rs, #255)
   750:     } else {
   751:       Result.setOpcode(Hexagon::SA1_and1);
   752:       addOps(Result, Inst, 0);
   753:       addOps(Result, Inst, 1);
   754:       break; //    1,2 SUBInst $Rd = and($Rs, #1)
   755:     }
   756:   case Hexagon::C2_cmpeqi:
   757:     Result.setOpcode(Hexagon::SA1_cmpeqi);
   758:     addOps(Result, Inst, 1);
   759:     addOps(Result, Inst, 2);
   760:     break; //    2,3 SUBInst p0 = cmp.eq($Rs, #$u2)
   761:   case Hexagon::A4_combineii:
   762:   case Hexagon::A2_combineii:
   763:     Absolute = Inst.getOperand(1).getExpr()->evaluateAsAbsolute(Value);
   764:     assert(Absolute);(void)Absolute;
   765:     if (Value == 1) {
   766:       Result.setOpcode(Hexagon::SA1_combine1i);
   767:       addOps(Result, Inst, 0);
   768:       addOps(Result, Inst, 2);
   769:       break; //  1,3 SUBInst $Rdd = combine(#1, #$u2)
   770:     }
   771:     if (Value == 3) {
   772:       Result.setOpcode(Hexagon::SA1_combine3i);
   773:       addOps(Result, Inst, 0);
   774:       addOps(Result, Inst, 2);
   775:       break; //  1,3 SUBInst $Rdd = combine(#3, #$u2)
   776:     }
   777:     if (Value == 0) {
   778:       Result.setOpcode(Hexagon::SA1_combine0i);
   779:       addOps(Result, Inst, 0);
   780:       addOps(Result, Inst, 2);
   781:       break; //  1,3 SUBInst $Rdd = combine(#0, #$u2)
   782:     }
   783:     if (Value == 2) {
   784:       Result.setOpcode(Hexagon::SA1_combine2i);
   785:       addOps(Result, Inst, 0);
   786:       addOps(Result, Inst, 2);
   787:       break; //  1,3 SUBInst $Rdd = combine(#2, #$u2)
   788:     }
   789:     break;
   790:   case Hexagon::A4_combineir:
   791:     Result.setOpcode(Hexagon::SA1_combinezr);
   792:     addOps(Result, Inst, 0);
   793:     addOps(Result, Inst, 2);
   794:     break; //    1,3 SUBInst $Rdd = combine(#0, $Rs)
   795:   case Hexagon::A4_combineri:
   796:     Result.setOpcode(Hexagon::SA1_combinerz);
   797:     addOps(Result, Inst, 0);
   798:     addOps(Result, Inst, 1);
   799:     break; //    1,2 SUBInst $Rdd = combine($Rs, #0)
   800:   case Hexagon::L4_return_tnew_pnt:
```
- EN: It declares or implements routines such as llvm_unreachable, getOperand, setOpcode, addOps, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 llvm_unreachable, getOperand, setOpcode, addOps, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-900 / 第 801-900 行

```cpp
   801:   case Hexagon::L4_return_tnew_pt:
   802:     Result.setOpcode(Hexagon::SL2_return_tnew);
   803:     break; //    none  SUBInst if (p0.new) dealloc_return:nt
   804:   case Hexagon::L4_return_fnew_pnt:
   805:   case Hexagon::L4_return_fnew_pt:
   806:     Result.setOpcode(Hexagon::SL2_return_fnew);
   807:     break; //    none  SUBInst if (!p0.new) dealloc_return:nt
   808:   case Hexagon::L4_return_f:
   809:     Result.setOpcode(Hexagon::SL2_return_f);
   810:     break; //    none  SUBInst if (!p0) dealloc_return
   811:   case Hexagon::L4_return_t:
   812:     Result.setOpcode(Hexagon::SL2_return_t);
   813:     break; //    none  SUBInst if (p0) dealloc_return
   814:   case Hexagon::L4_return:
   815:     Result.setOpcode(Hexagon::SL2_return);
   816:     break; //    none  SUBInst dealloc_return
   817:   case Hexagon::L2_deallocframe:
   818:     Result.setOpcode(Hexagon::SL2_deallocframe);
   819:     break; //    none  SUBInst deallocframe
   820:   case Hexagon::EH_RETURN_JMPR:
   821:   case Hexagon::J2_jumpr:
   822:   case Hexagon::PS_jmpret:
   823:     Result.setOpcode(Hexagon::SL2_jumpr31);
   824:     break; //    none  SUBInst jumpr r31
   825:   case Hexagon::J2_jumprf:
   826:   case Hexagon::PS_jmpretf:
   827:     Result.setOpcode(Hexagon::SL2_jumpr31_f);
   828:     break; //    none  SUBInst if (!p0) jumpr r31
   829:   case Hexagon::J2_jumprfnew:
   830:   case Hexagon::PS_jmpretfnewpt:
   831:   case Hexagon::PS_jmpretfnew:
   832:     Result.setOpcode(Hexagon::SL2_jumpr31_fnew);
   833:     break; //    none  SUBInst if (!p0.new) jumpr:nt r31
   834:   case Hexagon::J2_jumprt:
   835:   case Hexagon::PS_jmprett:
   836:     Result.setOpcode(Hexagon::SL2_jumpr31_t);
   837:     break; //    none  SUBInst if (p0) jumpr r31
   838:   case Hexagon::J2_jumprtnew:
   839:   case Hexagon::PS_jmprettnewpt:
   840:   case Hexagon::PS_jmprettnew:
   841:     Result.setOpcode(Hexagon::SL2_jumpr31_tnew);
   842:     break; //    none  SUBInst if (p0.new) jumpr:nt r31
   843:   case Hexagon::L2_loadrb_io:
   844:     Result.setOpcode(Hexagon::SL2_loadrb_io);
   845:     addOps(Result, Inst, 0);
   846:     addOps(Result, Inst, 1);
   847:     addOps(Result, Inst, 2);
   848:     break; //    1,2,3 SUBInst $Rd = memb($Rs + #$u3_0)
   849:   case Hexagon::L2_loadrd_io:
   850:     Result.setOpcode(Hexagon::SL2_loadrd_sp);
   851:     addOps(Result, Inst, 0);
   852:     addOps(Result, Inst, 2);
   853:     break; //    1,3 SUBInst $Rdd = memd(r29 + #$u5_3)
   854:   case Hexagon::L2_loadrh_io:
   855:     Result.setOpcode(Hexagon::SL2_loadrh_io);
   856:     addOps(Result, Inst, 0);
   857:     addOps(Result, Inst, 1);
   858:     addOps(Result, Inst, 2);
   859:     break; //    1,2,3 SUBInst $Rd = memh($Rs + #$u3_1)
   860:   case Hexagon::L2_loadrub_io:
   861:     Result.setOpcode(Hexagon::SL1_loadrub_io);
   862:     addOps(Result, Inst, 0);
   863:     addOps(Result, Inst, 1);
   864:     addOps(Result, Inst, 2);
   865:     break; //    1,2,3 SUBInst $Rd = memub($Rs + #$u4_0)
   866:   case Hexagon::L2_loadruh_io:
   867:     Result.setOpcode(Hexagon::SL2_loadruh_io);
   868:     addOps(Result, Inst, 0);
   869:     addOps(Result, Inst, 1);
   870:     addOps(Result, Inst, 2);
   871:     break; //    1,2,3 SUBInst $Rd = memuh($Rs + #$u3_1)
   872:   case Hexagon::L2_loadri_io:
   873:     if (Inst.getOperand(1).getReg() == Hexagon::R29) {
   874:       Result.setOpcode(Hexagon::SL2_loadri_sp);
   875:       addOps(Result, Inst, 0);
   876:       addOps(Result, Inst, 2);
   877:       break; //  2 1,3 SUBInst $Rd = memw(r29 + #$u5_2)
   878:     } else {
   879:       Result.setOpcode(Hexagon::SL1_loadri_io);
   880:       addOps(Result, Inst, 0);
   881:       addOps(Result, Inst, 1);
   882:       addOps(Result, Inst, 2);
   883:       break; //    1,2,3 SUBInst $Rd = memw($Rs + #$u4_2)
   884:     }
   885:   case Hexagon::S4_storeirb_io:
   886:     Absolute = Inst.getOperand(2).getExpr()->evaluateAsAbsolute(Value);
   887:     assert(Absolute);(void)Absolute;
   888:     if (Value == 0) {
   889:       Result.setOpcode(Hexagon::SS2_storebi0);
   890:       addOps(Result, Inst, 0);
   891:       addOps(Result, Inst, 1);
   892:       break; //    1,2 SUBInst memb($Rs + #$u4_0)=#0
   893:     } else if (Value == 1) {
   894:       Result.setOpcode(Hexagon::SS2_storebi1);
   895:       addOps(Result, Inst, 0);
   896:       addOps(Result, Inst, 1);
   897:       break; //  2 1,2 SUBInst memb($Rs + #$u4_0)=#1
   898:     }
   899:     break;
   900:   case Hexagon::S2_storerb_io:
```
- EN: It declares or implements routines such as setOpcode, addOps, memb, memd, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 setOpcode, addOps, memb, memd, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:     Result.setOpcode(Hexagon::SS1_storeb_io);
   902:     addOps(Result, Inst, 0);
   903:     addOps(Result, Inst, 1);
   904:     addOps(Result, Inst, 2);
   905:     break; //    1,2,3 SUBInst memb($Rs + #$u4_0) = $Rt
   906:   case Hexagon::S2_storerd_io:
   907:     Result.setOpcode(Hexagon::SS2_stored_sp);
   908:     addOps(Result, Inst, 1);
   909:     addOps(Result, Inst, 2);
   910:     break; //    2,3 SUBInst memd(r29 + #$s6_3) = $Rtt
   911:   case Hexagon::S2_storerh_io:
   912:     Result.setOpcode(Hexagon::SS2_storeh_io);
   913:     addOps(Result, Inst, 0);
   914:     addOps(Result, Inst, 1);
   915:     addOps(Result, Inst, 2);
   916:     break; //    1,2,3 SUBInst memb($Rs + #$u4_0) = $Rt
   917:   case Hexagon::S4_storeiri_io:
   918:     Absolute = Inst.getOperand(2).getExpr()->evaluateAsAbsolute(Value);
   919:     assert(Absolute);(void)Absolute;
   920:     if (Value == 0) {
   921:       Result.setOpcode(Hexagon::SS2_storewi0);
   922:       addOps(Result, Inst, 0);
   923:       addOps(Result, Inst, 1);
   924:       break; //  3 1,2 SUBInst memw($Rs + #$u4_2)=#0
   925:     } else if (Value == 1) {
   926:       Result.setOpcode(Hexagon::SS2_storewi1);
   927:       addOps(Result, Inst, 0);
   928:       addOps(Result, Inst, 1);
   929:       break; //  3 1,2 SUBInst memw($Rs + #$u4_2)=#1
   930:     } else if (Inst.getOperand(0).getReg() == Hexagon::R29) {
   931:       Result.setOpcode(Hexagon::SS2_storew_sp);
   932:       addOps(Result, Inst, 1);
   933:       addOps(Result, Inst, 2);
   934:       break; //  1 2,3 SUBInst memw(r29 + #$u5_2) = $Rt
   935:     }
   936:     break;
   937:   case Hexagon::S2_storeri_io:
   938:     if (Inst.getOperand(0).getReg() == Hexagon::R29) {
   939:       Result.setOpcode(Hexagon::SS2_storew_sp);
   940:       addOps(Result, Inst, 1);
   941:       addOps(Result, Inst, 2); //  1,2,3 SUBInst memw(sp + #$u5_2) = $Rt
   942:     } else {
   943:       Result.setOpcode(Hexagon::SS1_storew_io);
   944:       addOps(Result, Inst, 0);
   945:       addOps(Result, Inst, 1);
   946:       addOps(Result, Inst, 2); //  1,2,3 SUBInst memw($Rs + #$u4_2) = $Rt
   947:     }
   948:     break;
   949:   case Hexagon::A2_sxtb:
   950:     Result.setOpcode(Hexagon::SA1_sxtb);
   951:     addOps(Result, Inst, 0);
   952:     addOps(Result, Inst, 1);
   953:     break; //  1,2 SUBInst $Rd = sxtb($Rs)
   954:   case Hexagon::A2_sxth:
   955:     Result.setOpcode(Hexagon::SA1_sxth);
   956:     addOps(Result, Inst, 0);
   957:     addOps(Result, Inst, 1);
   958:     break; //  1,2 SUBInst $Rd = sxth($Rs)
   959:   case Hexagon::A2_tfr:
   960:     Result.setOpcode(Hexagon::SA1_tfr);
   961:     addOps(Result, Inst, 0);
   962:     addOps(Result, Inst, 1);
   963:     break; //  1,2 SUBInst $Rd = $Rs
   964:   case Hexagon::C2_cmovenewif:
   965:     Result.setOpcode(Hexagon::SA1_clrfnew);
   966:     addOps(Result, Inst, 0);
   967:     addOps(Result, Inst, 1);
   968:     break; //  2 SUBInst if (!p0.new) $Rd = #0
   969:   case Hexagon::C2_cmovenewit:
   970:     Result.setOpcode(Hexagon::SA1_clrtnew);
   971:     addOps(Result, Inst, 0);
   972:     addOps(Result, Inst, 1);
   973:     break; //  2 SUBInst if (p0.new) $Rd = #0
   974:   case Hexagon::C2_cmoveif:
   975:     Result.setOpcode(Hexagon::SA1_clrf);
   976:     addOps(Result, Inst, 0);
   977:     addOps(Result, Inst, 1);
   978:     break; //  2 SUBInst if (!p0) $Rd = #0
   979:   case Hexagon::C2_cmoveit:
   980:     Result.setOpcode(Hexagon::SA1_clrt);
   981:     addOps(Result, Inst, 0);
   982:     addOps(Result, Inst, 1);
   983:     break; //  2 SUBInst if (p0) $Rd = #0
   984:   case Hexagon::A2_tfrsi:
   985:     Absolute = Inst.getOperand(1).getExpr()->evaluateAsAbsolute(Value);
   986:     if (Absolute && Value == -1) {
   987:       Result.setOpcode(Hexagon::SA1_setin1);
   988:       addOps(Result, Inst, 0);
   989:       addOps(Result, Inst, 1);
   990:       break; //  2 1 SUBInst $Rd = #-1
   991:     } else {
   992:       Result.setOpcode(Hexagon::SA1_seti);
   993:       addOps(Result, Inst, 0);
   994:       addOps(Result, Inst, 1);
   995:       break; //    1,2 SUBInst $Rd = #$u6
   996:     }
   997:   case Hexagon::A2_zxtb:
   998:     Result.setOpcode(Hexagon::SA1_zxtb);
   999:     addOps(Result, Inst, 0);
  1000:     addOps(Result, Inst, 1);
```
- EN: It declares or implements routines such as setOpcode, addOps, memb, memd, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 setOpcode, addOps, memb, memd, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:     break; //    1,2    $Rd = and($Rs, #255)
  1002: 
  1003:   case Hexagon::A2_zxth:
  1004:     Result.setOpcode(Hexagon::SA1_zxth);
  1005:     addOps(Result, Inst, 0);
  1006:     addOps(Result, Inst, 1);
  1007:     break; //    1,2 SUBInst $Rd = zxth($Rs)
  1008:   }
  1009:   return Result;
  1010: }
  1011: 
  1012: static bool isStoreInst(unsigned opCode) {
  1013:   switch (opCode) {
  1014:   case Hexagon::S2_storeri_io:
  1015:   case Hexagon::S2_storerb_io:
  1016:   case Hexagon::S2_storerh_io:
  1017:   case Hexagon::S2_storerd_io:
  1018:   case Hexagon::S4_storeiri_io:
  1019:   case Hexagon::S4_storeirb_io:
  1020:   case Hexagon::S2_allocframe:
  1021:     return true;
  1022:   default:
  1023:     return false;
  1024:   }
  1025: }
  1026: 
  1027: SmallVector<DuplexCandidate, 8>
  1028: HexagonMCInstrInfo::getDuplexPossibilties(MCInstrInfo const &MCII,
  1029:                                           MCSubtargetInfo const &STI,
  1030:                                           MCInst const &MCB) {
  1031:   assert(isBundle(MCB));
  1032:   SmallVector<DuplexCandidate, 8> duplexToTry;
  1033:   // Use an "order matters" version of isDuplexPair.
  1034:   unsigned numInstrInPacket = MCB.getNumOperands();
  1035: 
  1036:   for (unsigned distance = 1; distance < numInstrInPacket; ++distance) {
  1037:     for (unsigned j = HexagonMCInstrInfo::bundleInstructionsOffset,
  1038:                   k = j + distance;
  1039:          (j < numInstrInPacket) && (k < numInstrInPacket); ++j, ++k) {
  1040: 
  1041:       // Check if reversible.
  1042:       bool bisReversable = true;
  1043:       if (isStoreInst(MCB.getOperand(j).getInst()->getOpcode()) &&
  1044:           isStoreInst(MCB.getOperand(k).getInst()->getOpcode())) {
  1045:         LLVM_DEBUG(dbgs() << "skip out of order write pair: " << k << "," << j
  1046:                           << "\n");
  1047:         bisReversable = false;
  1048:       }
  1049:       if (HexagonMCInstrInfo::isMemReorderDisabled(MCB)) // }:mem_noshuf
  1050:         bisReversable = false;
  1051: 
  1052:       // Try in order.
  1053:       if (isOrderedDuplexPair(
  1054:               MCII, *MCB.getOperand(k).getInst(),
  1055:               HexagonMCInstrInfo::hasExtenderForIndex(MCB, k - 1),
  1056:               *MCB.getOperand(j).getInst(),
  1057:               HexagonMCInstrInfo::hasExtenderForIndex(MCB, j - 1),
  1058:               bisReversable, STI)) {
  1059:         // Get iClass.
  1060:         unsigned iClass = iClassOfDuplexPair(
  1061:             getDuplexCandidateGroup(*MCB.getOperand(k).getInst()),
  1062:             getDuplexCandidateGroup(*MCB.getOperand(j).getInst()));
  1063: 
  1064:         // Save off pairs for duplex checking.
  1065:         duplexToTry.push_back(DuplexCandidate(j, k, iClass));
  1066:         LLVM_DEBUG(dbgs() << "adding pair: " << j << "," << k << ":"
  1067:                           << MCB.getOperand(j).getInst()->getOpcode() << ","
  1068:                           << MCB.getOperand(k).getInst()->getOpcode() << "\n");
  1069:         continue;
  1070:       } else {
  1071:         LLVM_DEBUG(dbgs() << "skipping pair: " << j << "," << k << ":"
  1072:                           << MCB.getOperand(j).getInst()->getOpcode() << ","
  1073:                           << MCB.getOperand(k).getInst()->getOpcode() << "\n");
  1074:       }
  1075: 
  1076:       // Try reverse.
  1077:       if (bisReversable) {
  1078:         if (isOrderedDuplexPair(
  1079:                 MCII, *MCB.getOperand(j).getInst(),
  1080:                 HexagonMCInstrInfo::hasExtenderForIndex(MCB, j - 1),
  1081:                 *MCB.getOperand(k).getInst(),
  1082:                 HexagonMCInstrInfo::hasExtenderForIndex(MCB, k - 1),
  1083:                 bisReversable, STI)) {
  1084:           // Get iClass.
  1085:           unsigned iClass = iClassOfDuplexPair(
  1086:               getDuplexCandidateGroup(*MCB.getOperand(j).getInst()),
  1087:               getDuplexCandidateGroup(*MCB.getOperand(k).getInst()));
  1088: 
  1089:           // Save off pairs for duplex checking.
  1090:           duplexToTry.push_back(DuplexCandidate(k, j, iClass));
  1091:           LLVM_DEBUG(dbgs()
  1092:                      << "adding pair:" << k << "," << j << ":"
  1093:                      << MCB.getOperand(j).getInst()->getOpcode() << ","
  1094:                      << MCB.getOperand(k).getInst()->getOpcode() << "\n");
  1095:         } else {
  1096:           LLVM_DEBUG(dbgs()
  1097:                      << "skipping pair: " << k << "," << j << ":"
  1098:                      << MCB.getOperand(j).getInst()->getOpcode() << ","
  1099:                      << MCB.getOperand(k).getInst()->getOpcode() << "\n");
  1100:         }
```
- EN: It declares or implements routines such as and, addOps, isStoreInst, HexagonMCInstrInfo::getDuplexPossibilties, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 and, addOps, isStoreInst, HexagonMCInstrInfo::getDuplexPossibilties, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 1101-1105 / 第 1101-1105 行

```cpp
  1101:       }
  1102:     }
  1103:   }
  1104:   return duplexToTry;
  1105: }
```
- EN: This range contains executable implementation details for a Hexagon backend subsystem.
- CN: 这一段包含了某个 Hexagon 后端子系统的可执行实现细节。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonMCExpr.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/SmallVector.h, llvm/MC/MCSubtargetInfo.h, llvm/Support/Debug.h, llvm/Support/ErrorHandling.h, llvm/Support/MathExtras.h, llvm/Support/raw_ostream.h, ... (15 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCDuplexInfo, HexagonMCExpr, HexagonBaseInfo, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
