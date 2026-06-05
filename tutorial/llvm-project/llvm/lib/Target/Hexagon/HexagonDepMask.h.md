# HexagonDepMask.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonDepMask.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Declares Hexagon mask-oriented rewriting and analysis.
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。
- Generation status / 生成状态: The file appears to be auto-generated or largely generated from upstream target data. / 该文件看起来是自动生成的，或主要由上游目标数据生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-250 / 第 1-250 行

```cpp
     1: //===----------------------------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // Automatically generated file, do not edit!
     9: //===----------------------------------------------------------------------===//
    10: 
    11: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONDEPMASK_H
    12: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONDEPMASK_H
    13: 
    14: HexagonInstruction InstructionEncodings[] = {
    15: { /*Tag:A2_addi*/
    16:   /*Rd32=add(Rs32,#s16)*/
    17:   0xf0000000,
    18:   0xb0000000,
    19:   0x0fe03fe0,
    20:   0 },
    21: { /*Tag:A2_andir*/
    22:   /*Rd32=and(Rs32,#s10)*/
    23:   0xffc00000,
    24:   0x76000000,
    25:   0x00203fe0,
    26:   0 },
    27: { /*Tag:A2_combineii*/
    28:   /*Rdd32=combine(#s8,#S8)*/
    29:   0xff800000,
    30:   0x7c000000,
    31:   0x00001fe0,
    32:   0 },
    33: { /*Tag:A2_orir*/
    34:   /*Rd32=or(Rs32,#s10)*/
    35:   0xffc00000,
    36:   0x76800000,
    37:   0x00203fe0,
    38:   0 },
    39: { /*Tag:A2_paddif*/
    40:   /*if (!Pu4) Rd32=add(Rs32,#s8)*/
    41:   0xff802000,
    42:   0x74800000,
    43:   0x00001fe0,
    44:   0 },
    45: { /*Tag:A2_paddifnew*/
    46:   /*if (!Pu4.new) Rd32=add(Rs32,#s8)*/
    47:   0xff802000,
    48:   0x74802000,
    49:   0x00001fe0,
    50:   0 },
    51: { /*Tag:A2_paddit*/
    52:   /*if (Pu4) Rd32=add(Rs32,#s8)*/
    53:   0xff802000,
    54:   0x74000000,
    55:   0x00001fe0,
    56:   0 },
    57: { /*Tag:A2_padditnew*/
    58:   /*if (Pu4.new) Rd32=add(Rs32,#s8)*/
    59:   0xff802000,
    60:   0x74002000,
    61:   0x00001fe0,
    62:   0 },
    63: { /*Tag:A2_subri*/
    64:   /*Rd32=sub(#s10,Rs32)*/
    65:   0xffc00000,
    66:   0x76400000,
    67:   0x00203fe0,
    68:   0 },
    69: { /*Tag:A2_tfrsi*/
    70:   /*Rd32=#s16*/
    71:   0xff000000,
    72:   0x78000000,
    73:   0x00df3fe0,
    74:   0 },
    75: { /*Tag:A4_cmpbgtui*/
    76:   /*Pd4=cmpb.gtu(Rs32,#u7)*/
    77:   0xff601018,
    78:   0xdd400000,
    79:   0x00000fe0,
    80:   0 },
    81: { /*Tag:A4_cmpheqi*/
    82:   /*Pd4=cmph.eq(Rs32,#s8)*/
    83:   0xff600018,
    84:   0xdd000008,
    85:   0x00001fe0,
    86:   0 },
    87: { /*Tag:A4_cmphgti*/
    88:   /*Pd4=cmph.gt(Rs32,#s8)*/
    89:   0xff600018,
    90:   0xdd200008,
    91:   0x00001fe0,
    92:   0 },
    93: { /*Tag:A4_cmphgtui*/
    94:   /*Pd4=cmph.gtu(Rs32,#u7)*/
    95:   0xff601018,
    96:   0xdd400008,
    97:   0x00000fe0,
    98:   0 },
    99: { /*Tag:A4_combineii*/
   100:   /*Rdd32=combine(#s8,#U6)*/
   101:   0xff800000,
   102:   0x7c800000,
   103:   0x001f2000,
   104:   0 },
   105: { /*Tag:A4_combineir*/
   106:   /*Rdd32=combine(#s8,Rs32)*/
   107:   0xff602000,
   108:   0x73202000,
   109:   0x00001fe0,
   110:   0 },
   111: { /*Tag:A4_combineri*/
   112:   /*Rdd32=combine(Rs32,#s8)*/
   113:   0xff602000,
   114:   0x73002000,
   115:   0x00001fe0,
   116:   0 },
   117: { /*Tag:A4_rcmpeqi*/
   118:   /*Rd32=cmp.eq(Rs32,#s8)*/
   119:   0xff602000,
   120:   0x73402000,
   121:   0x00001fe0,
   122:   0 },
   123: { /*Tag:A4_rcmpneqi*/
   124:   /*Rd32=!cmp.eq(Rs32,#s8)*/
   125:   0xff602000,
   126:   0x73602000,
   127:   0x00001fe0,
   128:   0 },
   129: { /*Tag:C2_cmoveif*/
   130:   /*if (!Pu4) Rd32=#s12*/
   131:   0xff902000,
   132:   0x7e800000,
   133:   0x000f1fe0,
   134:   0 },
   135: { /*Tag:C2_cmoveit*/
   136:   /*if (Pu4) Rd32=#s12*/
   137:   0xff902000,
   138:   0x7e000000,
   139:   0x000f1fe0,
   140:   0 },
   141: { /*Tag:C2_cmovenewif*/
   142:   /*if (!Pu4.new) Rd32=#s12*/
   143:   0xff902000,
   144:   0x7e802000,
   145:   0x000f1fe0,
   146:   0 },
   147: { /*Tag:C2_cmovenewit*/
   148:   /*if (Pu4.new) Rd32=#s12*/
   149:   0xff902000,
   150:   0x7e002000,
   151:   0x000f1fe0,
   152:   0 },
   153: { /*Tag:C2_cmpeqi*/
   154:   /*Pd4=cmp.eq(Rs32,#s10)*/
   155:   0xffc0001c,
   156:   0x75000000,
   157:   0x00203fe0,
   158:   0 },
   159: { /*Tag:C2_cmpgti*/
   160:   /*Pd4=cmp.gt(Rs32,#s10)*/
   161:   0xffc0001c,
   162:   0x75400000,
   163:   0x00203fe0,
   164:   0 },
   165: { /*Tag:C2_cmpgtui*/
   166:   /*Pd4=cmp.gtu(Rs32,#u9)*/
   167:   0xffe0001c,
   168:   0x75800000,
   169:   0x00003fe0,
   170:   0 },
   171: { /*Tag:C2_muxii*/
   172:   /*Rd32=mux(Pu4,#s8,#S8)*/
   173:   0xfe000000,
   174:   0x7a000000,
   175:   0x00001fe0,
   176:   0 },
   177: { /*Tag:C2_muxir*/
   178:   /*Rd32=mux(Pu4,Rs32,#s8)*/
   179:   0xff802000,
   180:   0x73000000,
   181:   0x00001fe0,
   182:   0 },
   183: { /*Tag:C2_muxri*/
   184:   /*Rd32=mux(Pu4,#s8,Rs32)*/
   185:   0xff802000,
   186:   0x73800000,
   187:   0x00001fe0,
   188:   0 },
   189: { /*Tag:C4_addipc*/
   190:   /*Rd32=add(pc,#u6)*/
   191:   0xffff0000,
   192:   0x6a490000,
   193:   0x00001f80,
   194:   0 },
   195: { /*Tag:C4_cmpltei*/
   196:   /*Pd4=!cmp.gt(Rs32,#s10)*/
   197:   0xffc0001c,
   198:   0x75400010,
   199:   0x00203fe0,
   200:   0 },
   201: { /*Tag:C4_cmplteui*/
   202:   /*Pd4=!cmp.gtu(Rs32,#u9)*/
   203:   0xffe0001c,
   204:   0x75800010,
   205:   0x00003fe0,
   206:   0 },
   207: { /*Tag:C4_cmpneqi*/
   208:   /*Pd4=!cmp.eq(Rs32,#s10)*/
   209:   0xffc0001c,
   210:   0x75000010,
   211:   0x00203fe0,
   212:   0 },
   213: { /*Tag:J2_call*/
   214:   /*call #r22:2*/
   215:   0xfe000001,
   216:   0x5a000000,
   217:   0x01ff3ffe,
   218:   0 },
   219: { /*Tag:J2_callf*/
   220:   /*if (!Pu4) call #r15:2*/
   221:   0xff200800,
   222:   0x5d200000,
   223:   0x00df20fe,
   224:   0 },
   225: { /*Tag:J2_callt*/
   226:   /*if (Pu4) call #r15:2*/
   227:   0xff200800,
   228:   0x5d000000,
   229:   0x00df20fe,
   230:   0 },
   231: { /*Tag:J2_jump*/
   232:   /*jump #r22:2*/
   233:   0xfe000000,
   234:   0x58000000,
   235:   0x01ff3ffe,
   236:   0 },
   237: { /*Tag:J2_jumpf*/
   238:   /*if (!Pu4) jump:nt #r15:2*/
   239:   0xff201800,
   240:   0x5c200000,
   241:   0x00df20fe,
   242:   0 },
   243: { /*Tag:J2_jumpfnew*/
   244:   /*if (!Pu4.new) jump:nt #r15:2*/
   245:   0xff201800,
   246:   0x5c200800,
   247:   0x00df20fe,
   248:   0 },
   249: { /*Tag:J2_jumpfnewpt*/
   250:   /*if (!Pu4.new) jump:t #r15:2*/
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstruction, showing how the code connects to sibling backend components. Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstruction，说明了它与同级后端组件的连接关系。 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 251-500 / 第 251-500 行

```cpp
   251:   0xff201800,
   252:   0x5c201800,
   253:   0x00df20fe,
   254:   0 },
   255: { /*Tag:J2_jumpfpt*/
   256:   /*if (!Pu4) jump:t #r15:2*/
   257:   0xff201800,
   258:   0x5c201000,
   259:   0x00df20fe,
   260:   0 },
   261: { /*Tag:J2_jumpt*/
   262:   /*if (Pu4) jump:nt #r15:2*/
   263:   0xff201800,
   264:   0x5c000000,
   265:   0x00df20fe,
   266:   0 },
   267: { /*Tag:J2_jumptnew*/
   268:   /*if (Pu4.new) jump:nt #r15:2*/
   269:   0xff201800,
   270:   0x5c000800,
   271:   0x00df20fe,
   272:   0 },
   273: { /*Tag:J2_jumptnewpt*/
   274:   /*if (Pu4.new) jump:t #r15:2*/
   275:   0xff201800,
   276:   0x5c001800,
   277:   0x00df20fe,
   278:   0 },
   279: { /*Tag:J2_jumptpt*/
   280:   /*if (Pu4) jump:t #r15:2*/
   281:   0xff201800,
   282:   0x5c001000,
   283:   0x00df20fe,
   284:   0 },
   285: { /*Tag:J2_loop0i*/
   286:   /*loop0(#r7:2,#U10)*/
   287:   0xffe00000,
   288:   0x69000000,
   289:   0x00001f18,
   290:   0 },
   291: { /*Tag:J2_loop0r*/
   292:   /*loop0(#r7:2,Rs32)*/
   293:   0xffe00000,
   294:   0x60000000,
   295:   0x00001f18,
   296:   0 },
   297: { /*Tag:J2_loop1i*/
   298:   /*loop1(#r7:2,#U10)*/
   299:   0xffe00000,
   300:   0x69200000,
   301:   0x00001f18,
   302:   0 },
   303: { /*Tag:J2_loop1r*/
   304:   /*loop1(#r7:2,Rs32)*/
   305:   0xffe00000,
   306:   0x60200000,
   307:   0x00001f18,
   308:   0 },
   309: { /*Tag:J2_ploop1si*/
   310:   /*p3=sp1loop0(#r7:2,#U10)*/
   311:   0xffe00000,
   312:   0x69a00000,
   313:   0x00001f18,
   314:   0 },
   315: { /*Tag:J2_ploop1sr*/
   316:   /*p3=sp1loop0(#r7:2,Rs32)*/
   317:   0xffe00000,
   318:   0x60a00000,
   319:   0x00001f18,
   320:   0 },
   321: { /*Tag:J2_ploop2si*/
   322:   /*p3=sp2loop0(#r7:2,#U10)*/
   323:   0xffe00000,
   324:   0x69c00000,
   325:   0x00001f18,
   326:   0 },
   327: { /*Tag:J2_ploop2sr*/
   328:   /*p3=sp2loop0(#r7:2,Rs32)*/
   329:   0xffe00000,
   330:   0x60c00000,
   331:   0x00001f18,
   332:   0 },
   333: { /*Tag:J2_ploop3si*/
   334:   /*p3=sp3loop0(#r7:2,#U10)*/
   335:   0xffe00000,
   336:   0x69e00000,
   337:   0x00001f18,
   338:   0 },
   339: { /*Tag:J2_ploop3sr*/
   340:   /*p3=sp3loop0(#r7:2,Rs32)*/
   341:   0xffe00000,
   342:   0x60e00000,
   343:   0x00001f18,
   344:   0 },
   345: { /*Tag:J4_cmpeq_f_jumpnv_nt*/
   346:   /*if (!cmp.eq(Ns8.new,Rt32)) jump:nt #r9:2*/
   347:   0xffc02000,
   348:   0x20400000,
   349:   0x003000fe,
   350:   0 },
   351: { /*Tag:J4_cmpeq_f_jumpnv_t*/
   352:   /*if (!cmp.eq(Ns8.new,Rt32)) jump:t #r9:2*/
   353:   0xffc02000,
   354:   0x20402000,
   355:   0x003000fe,
   356:   0 },
   357: { /*Tag:J4_cmpeq_fp0_jump_nt*/
   358:   /*p0=cmp.eq(Rs16,Rt16); if (!p0.new) jump:nt #r9:2*/
   359:   0xffc03000,
   360:   0x14400000,
   361:   0x003000fe,
   362:   0 },
   363: { /*Tag:J4_cmpeq_fp0_jump_t*/
   364:   /*p0=cmp.eq(Rs16,Rt16); if (!p0.new) jump:t #r9:2*/
   365:   0xffc03000,
   366:   0x14402000,
   367:   0x003000fe,
   368:   0 },
   369: { /*Tag:J4_cmpeq_fp1_jump_nt*/
   370:   /*p1=cmp.eq(Rs16,Rt16); if (!p1.new) jump:nt #r9:2*/
   371:   0xffc03000,
   372:   0x14401000,
   373:   0x003000fe,
   374:   0 },
   375: { /*Tag:J4_cmpeq_fp1_jump_t*/
   376:   /*p1=cmp.eq(Rs16,Rt16); if (!p1.new) jump:t #r9:2*/
   377:   0xffc03000,
   378:   0x14403000,
   379:   0x003000fe,
   380:   0 },
   381: { /*Tag:J4_cmpeq_t_jumpnv_nt*/
   382:   /*if (cmp.eq(Ns8.new,Rt32)) jump:nt #r9:2*/
   383:   0xffc02000,
   384:   0x20000000,
   385:   0x003000fe,
   386:   0 },
   387: { /*Tag:J4_cmpeq_t_jumpnv_t*/
   388:   /*if (cmp.eq(Ns8.new,Rt32)) jump:t #r9:2*/
   389:   0xffc02000,
   390:   0x20002000,
   391:   0x003000fe,
   392:   0 },
   393: { /*Tag:J4_cmpeq_tp0_jump_nt*/
   394:   /*p0=cmp.eq(Rs16,Rt16); if (p0.new) jump:nt #r9:2*/
   395:   0xffc03000,
   396:   0x14000000,
   397:   0x003000fe,
   398:   0 },
   399: { /*Tag:J4_cmpeq_tp0_jump_t*/
   400:   /*p0=cmp.eq(Rs16,Rt16); if (p0.new) jump:t #r9:2*/
   401:   0xffc03000,
   402:   0x14002000,
   403:   0x003000fe,
   404:   0 },
   405: { /*Tag:J4_cmpeq_tp1_jump_nt*/
   406:   /*p1=cmp.eq(Rs16,Rt16); if (p1.new) jump:nt #r9:2*/
   407:   0xffc03000,
   408:   0x14001000,
   409:   0x003000fe,
   410:   0 },
   411: { /*Tag:J4_cmpeq_tp1_jump_t*/
   412:   /*p1=cmp.eq(Rs16,Rt16); if (p1.new) jump:t #r9:2*/
   413:   0xffc03000,
   414:   0x14003000,
   415:   0x003000fe,
   416:   0 },
   417: { /*Tag:J4_cmpeqi_f_jumpnv_nt*/
   418:   /*if (!cmp.eq(Ns8.new,#U5)) jump:nt #r9:2*/
   419:   0xffc02000,
   420:   0x24400000,
   421:   0x003000fe,
   422:   0 },
   423: { /*Tag:J4_cmpeqi_f_jumpnv_t*/
   424:   /*if (!cmp.eq(Ns8.new,#U5)) jump:t #r9:2*/
   425:   0xffc02000,
   426:   0x24402000,
   427:   0x003000fe,
   428:   0 },
   429: { /*Tag:J4_cmpeqi_fp0_jump_nt*/
   430:   /*p0=cmp.eq(Rs16,#U5); if (!p0.new) jump:nt #r9:2*/
   431:   0xffc02000,
   432:   0x10400000,
   433:   0x003000fe,
   434:   0 },
   435: { /*Tag:J4_cmpeqi_fp0_jump_t*/
   436:   /*p0=cmp.eq(Rs16,#U5); if (!p0.new) jump:t #r9:2*/
   437:   0xffc02000,
   438:   0x10402000,
   439:   0x003000fe,
   440:   0 },
   441: { /*Tag:J4_cmpeqi_fp1_jump_nt*/
   442:   /*p1=cmp.eq(Rs16,#U5); if (!p1.new) jump:nt #r9:2*/
   443:   0xffc02000,
   444:   0x12400000,
   445:   0x003000fe,
   446:   0 },
   447: { /*Tag:J4_cmpeqi_fp1_jump_t*/
   448:   /*p1=cmp.eq(Rs16,#U5); if (!p1.new) jump:t #r9:2*/
   449:   0xffc02000,
   450:   0x12402000,
   451:   0x003000fe,
   452:   0 },
   453: { /*Tag:J4_cmpeqi_t_jumpnv_nt*/
   454:   /*if (cmp.eq(Ns8.new,#U5)) jump:nt #r9:2*/
   455:   0xffc02000,
   456:   0x24000000,
   457:   0x003000fe,
   458:   0 },
   459: { /*Tag:J4_cmpeqi_t_jumpnv_t*/
   460:   /*if (cmp.eq(Ns8.new,#U5)) jump:t #r9:2*/
   461:   0xffc02000,
   462:   0x24002000,
   463:   0x003000fe,
   464:   0 },
   465: { /*Tag:J4_cmpeqi_tp0_jump_nt*/
   466:   /*p0=cmp.eq(Rs16,#U5); if (p0.new) jump:nt #r9:2*/
   467:   0xffc02000,
   468:   0x10000000,
   469:   0x003000fe,
   470:   0 },
   471: { /*Tag:J4_cmpeqi_tp0_jump_t*/
   472:   /*p0=cmp.eq(Rs16,#U5); if (p0.new) jump:t #r9:2*/
   473:   0xffc02000,
   474:   0x10002000,
   475:   0x003000fe,
   476:   0 },
   477: { /*Tag:J4_cmpeqi_tp1_jump_nt*/
   478:   /*p1=cmp.eq(Rs16,#U5); if (p1.new) jump:nt #r9:2*/
   479:   0xffc02000,
   480:   0x12000000,
   481:   0x003000fe,
   482:   0 },
   483: { /*Tag:J4_cmpeqi_tp1_jump_t*/
   484:   /*p1=cmp.eq(Rs16,#U5); if (p1.new) jump:t #r9:2*/
   485:   0xffc02000,
   486:   0x12002000,
   487:   0x003000fe,
   488:   0 },
   489: { /*Tag:J4_cmpeqn1_f_jumpnv_nt*/
   490:   /*if (!cmp.eq(Ns8.new,#-1)) jump:nt #r9:2*/
   491:   0xffc02000,
   492:   0x26400000,
   493:   0x003000fe,
   494:   0 },
   495: { /*Tag:J4_cmpeqn1_f_jumpnv_t*/
   496:   /*if (!cmp.eq(Ns8.new,#-1)) jump:t #r9:2*/
   497:   0xffc02000,
   498:   0x26402000,
   499:   0x003000fe,
   500:   0 },
```
- EN: It declares or implements routines such as eq, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 eq 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 501-750 / 第 501-750 行

```cpp
   501: { /*Tag:J4_cmpeqn1_fp0_jump_nt*/
   502:   /*p0=cmp.eq(Rs16,#-1); if (!p0.new) jump:nt #r9:2*/
   503:   0xffc02300,
   504:   0x11c00000,
   505:   0x003000fe,
   506:   0 },
   507: { /*Tag:J4_cmpeqn1_fp0_jump_t*/
   508:   /*p0=cmp.eq(Rs16,#-1); if (!p0.new) jump:t #r9:2*/
   509:   0xffc02300,
   510:   0x11c02000,
   511:   0x003000fe,
   512:   0 },
   513: { /*Tag:J4_cmpeqn1_fp1_jump_nt*/
   514:   /*p1=cmp.eq(Rs16,#-1); if (!p1.new) jump:nt #r9:2*/
   515:   0xffc02300,
   516:   0x13c00000,
   517:   0x003000fe,
   518:   0 },
   519: { /*Tag:J4_cmpeqn1_fp1_jump_t*/
   520:   /*p1=cmp.eq(Rs16,#-1); if (!p1.new) jump:t #r9:2*/
   521:   0xffc02300,
   522:   0x13c02000,
   523:   0x003000fe,
   524:   0 },
   525: { /*Tag:J4_cmpeqn1_t_jumpnv_nt*/
   526:   /*if (cmp.eq(Ns8.new,#-1)) jump:nt #r9:2*/
   527:   0xffc02000,
   528:   0x26000000,
   529:   0x003000fe,
   530:   0 },
   531: { /*Tag:J4_cmpeqn1_t_jumpnv_t*/
   532:   /*if (cmp.eq(Ns8.new,#-1)) jump:t #r9:2*/
   533:   0xffc02000,
   534:   0x26002000,
   535:   0x003000fe,
   536:   0 },
   537: { /*Tag:J4_cmpeqn1_tp0_jump_nt*/
   538:   /*p0=cmp.eq(Rs16,#-1); if (p0.new) jump:nt #r9:2*/
   539:   0xffc02300,
   540:   0x11800000,
   541:   0x003000fe,
   542:   0 },
   543: { /*Tag:J4_cmpeqn1_tp0_jump_t*/
   544:   /*p0=cmp.eq(Rs16,#-1); if (p0.new) jump:t #r9:2*/
   545:   0xffc02300,
   546:   0x11802000,
   547:   0x003000fe,
   548:   0 },
   549: { /*Tag:J4_cmpeqn1_tp1_jump_nt*/
   550:   /*p1=cmp.eq(Rs16,#-1); if (p1.new) jump:nt #r9:2*/
   551:   0xffc02300,
   552:   0x13800000,
   553:   0x003000fe,
   554:   0 },
   555: { /*Tag:J4_cmpeqn1_tp1_jump_t*/
   556:   /*p1=cmp.eq(Rs16,#-1); if (p1.new) jump:t #r9:2*/
   557:   0xffc02300,
   558:   0x13802000,
   559:   0x003000fe,
   560:   0 },
   561: { /*Tag:J4_cmpgt_f_jumpnv_nt*/
   562:   /*if (!cmp.gt(Ns8.new,Rt32)) jump:nt #r9:2*/
   563:   0xffc02000,
   564:   0x20c00000,
   565:   0x003000fe,
   566:   0 },
   567: { /*Tag:J4_cmpgt_f_jumpnv_t*/
   568:   /*if (!cmp.gt(Ns8.new,Rt32)) jump:t #r9:2*/
   569:   0xffc02000,
   570:   0x20c02000,
   571:   0x003000fe,
   572:   0 },
   573: { /*Tag:J4_cmpgt_fp0_jump_nt*/
   574:   /*p0=cmp.gt(Rs16,Rt16); if (!p0.new) jump:nt #r9:2*/
   575:   0xffc03000,
   576:   0x14c00000,
   577:   0x003000fe,
   578:   0 },
   579: { /*Tag:J4_cmpgt_fp0_jump_t*/
   580:   /*p0=cmp.gt(Rs16,Rt16); if (!p0.new) jump:t #r9:2*/
   581:   0xffc03000,
   582:   0x14c02000,
   583:   0x003000fe,
   584:   0 },
   585: { /*Tag:J4_cmpgt_fp1_jump_nt*/
   586:   /*p1=cmp.gt(Rs16,Rt16); if (!p1.new) jump:nt #r9:2*/
   587:   0xffc03000,
   588:   0x14c01000,
   589:   0x003000fe,
   590:   0 },
   591: { /*Tag:J4_cmpgt_fp1_jump_t*/
   592:   /*p1=cmp.gt(Rs16,Rt16); if (!p1.new) jump:t #r9:2*/
   593:   0xffc03000,
   594:   0x14c03000,
   595:   0x003000fe,
   596:   0 },
   597: { /*Tag:J4_cmpgt_t_jumpnv_nt*/
   598:   /*if (cmp.gt(Ns8.new,Rt32)) jump:nt #r9:2*/
   599:   0xffc02000,
   600:   0x20800000,
   601:   0x003000fe,
   602:   0 },
   603: { /*Tag:J4_cmpgt_t_jumpnv_t*/
   604:   /*if (cmp.gt(Ns8.new,Rt32)) jump:t #r9:2*/
   605:   0xffc02000,
   606:   0x20802000,
   607:   0x003000fe,
   608:   0 },
   609: { /*Tag:J4_cmpgt_tp0_jump_nt*/
   610:   /*p0=cmp.gt(Rs16,Rt16); if (p0.new) jump:nt #r9:2*/
   611:   0xffc03000,
   612:   0x14800000,
   613:   0x003000fe,
   614:   0 },
   615: { /*Tag:J4_cmpgt_tp0_jump_t*/
   616:   /*p0=cmp.gt(Rs16,Rt16); if (p0.new) jump:t #r9:2*/
   617:   0xffc03000,
   618:   0x14802000,
   619:   0x003000fe,
   620:   0 },
   621: { /*Tag:J4_cmpgt_tp1_jump_nt*/
   622:   /*p1=cmp.gt(Rs16,Rt16); if (p1.new) jump:nt #r9:2*/
   623:   0xffc03000,
   624:   0x14801000,
   625:   0x003000fe,
   626:   0 },
   627: { /*Tag:J4_cmpgt_tp1_jump_t*/
   628:   /*p1=cmp.gt(Rs16,Rt16); if (p1.new) jump:t #r9:2*/
   629:   0xffc03000,
   630:   0x14803000,
   631:   0x003000fe,
   632:   0 },
   633: { /*Tag:J4_cmpgti_f_jumpnv_nt*/
   634:   /*if (!cmp.gt(Ns8.new,#U5)) jump:nt #r9:2*/
   635:   0xffc02000,
   636:   0x24c00000,
   637:   0x003000fe,
   638:   0 },
   639: { /*Tag:J4_cmpgti_f_jumpnv_t*/
   640:   /*if (!cmp.gt(Ns8.new,#U5)) jump:t #r9:2*/
   641:   0xffc02000,
   642:   0x24c02000,
   643:   0x003000fe,
   644:   0 },
   645: { /*Tag:J4_cmpgti_fp0_jump_nt*/
   646:   /*p0=cmp.gt(Rs16,#U5); if (!p0.new) jump:nt #r9:2*/
   647:   0xffc02000,
   648:   0x10c00000,
   649:   0x003000fe,
   650:   0 },
   651: { /*Tag:J4_cmpgti_fp0_jump_t*/
   652:   /*p0=cmp.gt(Rs16,#U5); if (!p0.new) jump:t #r9:2*/
   653:   0xffc02000,
   654:   0x10c02000,
   655:   0x003000fe,
   656:   0 },
   657: { /*Tag:J4_cmpgti_fp1_jump_nt*/
   658:   /*p1=cmp.gt(Rs16,#U5); if (!p1.new) jump:nt #r9:2*/
   659:   0xffc02000,
   660:   0x12c00000,
   661:   0x003000fe,
   662:   0 },
   663: { /*Tag:J4_cmpgti_fp1_jump_t*/
   664:   /*p1=cmp.gt(Rs16,#U5); if (!p1.new) jump:t #r9:2*/
   665:   0xffc02000,
   666:   0x12c02000,
   667:   0x003000fe,
   668:   0 },
   669: { /*Tag:J4_cmpgti_t_jumpnv_nt*/
   670:   /*if (cmp.gt(Ns8.new,#U5)) jump:nt #r9:2*/
   671:   0xffc02000,
   672:   0x24800000,
   673:   0x003000fe,
   674:   0 },
   675: { /*Tag:J4_cmpgti_t_jumpnv_t*/
   676:   /*if (cmp.gt(Ns8.new,#U5)) jump:t #r9:2*/
   677:   0xffc02000,
   678:   0x24802000,
   679:   0x003000fe,
   680:   0 },
   681: { /*Tag:J4_cmpgti_tp0_jump_nt*/
   682:   /*p0=cmp.gt(Rs16,#U5); if (p0.new) jump:nt #r9:2*/
   683:   0xffc02000,
   684:   0x10800000,
   685:   0x003000fe,
   686:   0 },
   687: { /*Tag:J4_cmpgti_tp0_jump_t*/
   688:   /*p0=cmp.gt(Rs16,#U5); if (p0.new) jump:t #r9:2*/
   689:   0xffc02000,
   690:   0x10802000,
   691:   0x003000fe,
   692:   0 },
   693: { /*Tag:J4_cmpgti_tp1_jump_nt*/
   694:   /*p1=cmp.gt(Rs16,#U5); if (p1.new) jump:nt #r9:2*/
   695:   0xffc02000,
   696:   0x12800000,
   697:   0x003000fe,
   698:   0 },
   699: { /*Tag:J4_cmpgti_tp1_jump_t*/
   700:   /*p1=cmp.gt(Rs16,#U5); if (p1.new) jump:t #r9:2*/
   701:   0xffc02000,
   702:   0x12802000,
   703:   0x003000fe,
   704:   0 },
   705: { /*Tag:J4_cmpgtn1_f_jumpnv_nt*/
   706:   /*if (!cmp.gt(Ns8.new,#-1)) jump:nt #r9:2*/
   707:   0xffc02000,
   708:   0x26c00000,
   709:   0x003000fe,
   710:   0 },
   711: { /*Tag:J4_cmpgtn1_f_jumpnv_t*/
   712:   /*if (!cmp.gt(Ns8.new,#-1)) jump:t #r9:2*/
   713:   0xffc02000,
   714:   0x26c02000,
   715:   0x003000fe,
   716:   0 },
   717: { /*Tag:J4_cmpgtn1_fp0_jump_nt*/
   718:   /*p0=cmp.gt(Rs16,#-1); if (!p0.new) jump:nt #r9:2*/
   719:   0xffc02300,
   720:   0x11c00100,
   721:   0x003000fe,
   722:   0 },
   723: { /*Tag:J4_cmpgtn1_fp0_jump_t*/
   724:   /*p0=cmp.gt(Rs16,#-1); if (!p0.new) jump:t #r9:2*/
   725:   0xffc02300,
   726:   0x11c02100,
   727:   0x003000fe,
   728:   0 },
   729: { /*Tag:J4_cmpgtn1_fp1_jump_nt*/
   730:   /*p1=cmp.gt(Rs16,#-1); if (!p1.new) jump:nt #r9:2*/
   731:   0xffc02300,
   732:   0x13c00100,
   733:   0x003000fe,
   734:   0 },
   735: { /*Tag:J4_cmpgtn1_fp1_jump_t*/
   736:   /*p1=cmp.gt(Rs16,#-1); if (!p1.new) jump:t #r9:2*/
   737:   0xffc02300,
   738:   0x13c02100,
   739:   0x003000fe,
   740:   0 },
   741: { /*Tag:J4_cmpgtn1_t_jumpnv_nt*/
   742:   /*if (cmp.gt(Ns8.new,#-1)) jump:nt #r9:2*/
   743:   0xffc02000,
   744:   0x26800000,
   745:   0x003000fe,
   746:   0 },
   747: { /*Tag:J4_cmpgtn1_t_jumpnv_t*/
   748:   /*if (cmp.gt(Ns8.new,#-1)) jump:t #r9:2*/
   749:   0xffc02000,
   750:   0x26802000,
```
- EN: It declares or implements routines such as eq, gt, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 eq, gt 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 751-1000 / 第 751-1000 行

```cpp
   751:   0x003000fe,
   752:   0 },
   753: { /*Tag:J4_cmpgtn1_tp0_jump_nt*/
   754:   /*p0=cmp.gt(Rs16,#-1); if (p0.new) jump:nt #r9:2*/
   755:   0xffc02300,
   756:   0x11800100,
   757:   0x003000fe,
   758:   0 },
   759: { /*Tag:J4_cmpgtn1_tp0_jump_t*/
   760:   /*p0=cmp.gt(Rs16,#-1); if (p0.new) jump:t #r9:2*/
   761:   0xffc02300,
   762:   0x11802100,
   763:   0x003000fe,
   764:   0 },
   765: { /*Tag:J4_cmpgtn1_tp1_jump_nt*/
   766:   /*p1=cmp.gt(Rs16,#-1); if (p1.new) jump:nt #r9:2*/
   767:   0xffc02300,
   768:   0x13800100,
   769:   0x003000fe,
   770:   0 },
   771: { /*Tag:J4_cmpgtn1_tp1_jump_t*/
   772:   /*p1=cmp.gt(Rs16,#-1); if (p1.new) jump:t #r9:2*/
   773:   0xffc02300,
   774:   0x13802100,
   775:   0x003000fe,
   776:   0 },
   777: { /*Tag:J4_cmpgtu_f_jumpnv_nt*/
   778:   /*if (!cmp.gtu(Ns8.new,Rt32)) jump:nt #r9:2*/
   779:   0xffc02000,
   780:   0x21400000,
   781:   0x003000fe,
   782:   0 },
   783: { /*Tag:J4_cmpgtu_f_jumpnv_t*/
   784:   /*if (!cmp.gtu(Ns8.new,Rt32)) jump:t #r9:2*/
   785:   0xffc02000,
   786:   0x21402000,
   787:   0x003000fe,
   788:   0 },
   789: { /*Tag:J4_cmpgtu_fp0_jump_nt*/
   790:   /*p0=cmp.gtu(Rs16,Rt16); if (!p0.new) jump:nt #r9:2*/
   791:   0xffc03000,
   792:   0x15400000,
   793:   0x003000fe,
   794:   0 },
   795: { /*Tag:J4_cmpgtu_fp0_jump_t*/
   796:   /*p0=cmp.gtu(Rs16,Rt16); if (!p0.new) jump:t #r9:2*/
   797:   0xffc03000,
   798:   0x15402000,
   799:   0x003000fe,
   800:   0 },
   801: { /*Tag:J4_cmpgtu_fp1_jump_nt*/
   802:   /*p1=cmp.gtu(Rs16,Rt16); if (!p1.new) jump:nt #r9:2*/
   803:   0xffc03000,
   804:   0x15401000,
   805:   0x003000fe,
   806:   0 },
   807: { /*Tag:J4_cmpgtu_fp1_jump_t*/
   808:   /*p1=cmp.gtu(Rs16,Rt16); if (!p1.new) jump:t #r9:2*/
   809:   0xffc03000,
   810:   0x15403000,
   811:   0x003000fe,
   812:   0 },
   813: { /*Tag:J4_cmpgtu_t_jumpnv_nt*/
   814:   /*if (cmp.gtu(Ns8.new,Rt32)) jump:nt #r9:2*/
   815:   0xffc02000,
   816:   0x21000000,
   817:   0x003000fe,
   818:   0 },
   819: { /*Tag:J4_cmpgtu_t_jumpnv_t*/
   820:   /*if (cmp.gtu(Ns8.new,Rt32)) jump:t #r9:2*/
   821:   0xffc02000,
   822:   0x21002000,
   823:   0x003000fe,
   824:   0 },
   825: { /*Tag:J4_cmpgtu_tp0_jump_nt*/
   826:   /*p0=cmp.gtu(Rs16,Rt16); if (p0.new) jump:nt #r9:2*/
   827:   0xffc03000,
   828:   0x15000000,
   829:   0x003000fe,
   830:   0 },
   831: { /*Tag:J4_cmpgtu_tp0_jump_t*/
   832:   /*p0=cmp.gtu(Rs16,Rt16); if (p0.new) jump:t #r9:2*/
   833:   0xffc03000,
   834:   0x15002000,
   835:   0x003000fe,
   836:   0 },
   837: { /*Tag:J4_cmpgtu_tp1_jump_nt*/
   838:   /*p1=cmp.gtu(Rs16,Rt16); if (p1.new) jump:nt #r9:2*/
   839:   0xffc03000,
   840:   0x15001000,
   841:   0x003000fe,
   842:   0 },
   843: { /*Tag:J4_cmpgtu_tp1_jump_t*/
   844:   /*p1=cmp.gtu(Rs16,Rt16); if (p1.new) jump:t #r9:2*/
   845:   0xffc03000,
   846:   0x15003000,
   847:   0x003000fe,
   848:   0 },
   849: { /*Tag:J4_cmpgtui_f_jumpnv_nt*/
   850:   /*if (!cmp.gtu(Ns8.new,#U5)) jump:nt #r9:2*/
   851:   0xffc02000,
   852:   0x25400000,
   853:   0x003000fe,
   854:   0 },
   855: { /*Tag:J4_cmpgtui_f_jumpnv_t*/
   856:   /*if (!cmp.gtu(Ns8.new,#U5)) jump:t #r9:2*/
   857:   0xffc02000,
   858:   0x25402000,
   859:   0x003000fe,
   860:   0 },
   861: { /*Tag:J4_cmpgtui_fp0_jump_nt*/
   862:   /*p0=cmp.gtu(Rs16,#U5); if (!p0.new) jump:nt #r9:2*/
   863:   0xffc02000,
   864:   0x11400000,
   865:   0x003000fe,
   866:   0 },
   867: { /*Tag:J4_cmpgtui_fp0_jump_t*/
   868:   /*p0=cmp.gtu(Rs16,#U5); if (!p0.new) jump:t #r9:2*/
   869:   0xffc02000,
   870:   0x11402000,
   871:   0x003000fe,
   872:   0 },
   873: { /*Tag:J4_cmpgtui_fp1_jump_nt*/
   874:   /*p1=cmp.gtu(Rs16,#U5); if (!p1.new) jump:nt #r9:2*/
   875:   0xffc02000,
   876:   0x13400000,
   877:   0x003000fe,
   878:   0 },
   879: { /*Tag:J4_cmpgtui_fp1_jump_t*/
   880:   /*p1=cmp.gtu(Rs16,#U5); if (!p1.new) jump:t #r9:2*/
   881:   0xffc02000,
   882:   0x13402000,
   883:   0x003000fe,
   884:   0 },
   885: { /*Tag:J4_cmpgtui_t_jumpnv_nt*/
   886:   /*if (cmp.gtu(Ns8.new,#U5)) jump:nt #r9:2*/
   887:   0xffc02000,
   888:   0x25000000,
   889:   0x003000fe,
   890:   0 },
   891: { /*Tag:J4_cmpgtui_t_jumpnv_t*/
   892:   /*if (cmp.gtu(Ns8.new,#U5)) jump:t #r9:2*/
   893:   0xffc02000,
   894:   0x25002000,
   895:   0x003000fe,
   896:   0 },
   897: { /*Tag:J4_cmpgtui_tp0_jump_nt*/
   898:   /*p0=cmp.gtu(Rs16,#U5); if (p0.new) jump:nt #r9:2*/
   899:   0xffc02000,
   900:   0x11000000,
   901:   0x003000fe,
   902:   0 },
   903: { /*Tag:J4_cmpgtui_tp0_jump_t*/
   904:   /*p0=cmp.gtu(Rs16,#U5); if (p0.new) jump:t #r9:2*/
   905:   0xffc02000,
   906:   0x11002000,
   907:   0x003000fe,
   908:   0 },
   909: { /*Tag:J4_cmpgtui_tp1_jump_nt*/
   910:   /*p1=cmp.gtu(Rs16,#U5); if (p1.new) jump:nt #r9:2*/
   911:   0xffc02000,
   912:   0x13000000,
   913:   0x003000fe,
   914:   0 },
   915: { /*Tag:J4_cmpgtui_tp1_jump_t*/
   916:   /*p1=cmp.gtu(Rs16,#U5); if (p1.new) jump:t #r9:2*/
   917:   0xffc02000,
   918:   0x13002000,
   919:   0x003000fe,
   920:   0 },
   921: { /*Tag:J4_cmplt_f_jumpnv_nt*/
   922:   /*if (!cmp.gt(Rt32,Ns8.new)) jump:nt #r9:2*/
   923:   0xffc02000,
   924:   0x21c00000,
   925:   0x003000fe,
   926:   0 },
   927: { /*Tag:J4_cmplt_f_jumpnv_t*/
   928:   /*if (!cmp.gt(Rt32,Ns8.new)) jump:t #r9:2*/
   929:   0xffc02000,
   930:   0x21c02000,
   931:   0x003000fe,
   932:   0 },
   933: { /*Tag:J4_cmplt_t_jumpnv_nt*/
   934:   /*if (cmp.gt(Rt32,Ns8.new)) jump:nt #r9:2*/
   935:   0xffc02000,
   936:   0x21800000,
   937:   0x003000fe,
   938:   0 },
   939: { /*Tag:J4_cmplt_t_jumpnv_t*/
   940:   /*if (cmp.gt(Rt32,Ns8.new)) jump:t #r9:2*/
   941:   0xffc02000,
   942:   0x21802000,
   943:   0x003000fe,
   944:   0 },
   945: { /*Tag:J4_cmpltu_f_jumpnv_nt*/
   946:   /*if (!cmp.gtu(Rt32,Ns8.new)) jump:nt #r9:2*/
   947:   0xffc02000,
   948:   0x22400000,
   949:   0x003000fe,
   950:   0 },
   951: { /*Tag:J4_cmpltu_f_jumpnv_t*/
   952:   /*if (!cmp.gtu(Rt32,Ns8.new)) jump:t #r9:2*/
   953:   0xffc02000,
   954:   0x22402000,
   955:   0x003000fe,
   956:   0 },
   957: { /*Tag:J4_cmpltu_t_jumpnv_nt*/
   958:   /*if (cmp.gtu(Rt32,Ns8.new)) jump:nt #r9:2*/
   959:   0xffc02000,
   960:   0x22000000,
   961:   0x003000fe,
   962:   0 },
   963: { /*Tag:J4_cmpltu_t_jumpnv_t*/
   964:   /*if (cmp.gtu(Rt32,Ns8.new)) jump:t #r9:2*/
   965:   0xffc02000,
   966:   0x22002000,
   967:   0x003000fe,
   968:   0 },
   969: { /*Tag:J4_jumpseti*/
   970:   /*Rd16=#U6 ; jump #r9:2*/
   971:   0xff000000,
   972:   0x16000000,
   973:   0x003000fe,
   974:   0 },
   975: { /*Tag:J4_jumpsetr*/
   976:   /*Rd16=Rs16 ; jump #r9:2*/
   977:   0xff000000,
   978:   0x17000000,
   979:   0x003000fe,
   980:   0 },
   981: { /*Tag:J4_tstbit0_f_jumpnv_nt*/
   982:   /*if (!tstbit(Ns8.new,#0)) jump:nt #r9:2*/
   983:   0xffc02000,
   984:   0x25c00000,
   985:   0x003000fe,
   986:   0 },
   987: { /*Tag:J4_tstbit0_f_jumpnv_t*/
   988:   /*if (!tstbit(Ns8.new,#0)) jump:t #r9:2*/
   989:   0xffc02000,
   990:   0x25c02000,
   991:   0x003000fe,
   992:   0 },
   993: { /*Tag:J4_tstbit0_fp0_jump_nt*/
   994:   /*p0=tstbit(Rs16,#0); if (!p0.new) jump:nt #r9:2*/
   995:   0xffc02300,
   996:   0x11c00300,
   997:   0x003000fe,
   998:   0 },
   999: { /*Tag:J4_tstbit0_fp0_jump_t*/
  1000:   /*p0=tstbit(Rs16,#0); if (!p0.new) jump:t #r9:2*/
```
- EN: It declares or implements routines such as gt, gtu, tstbit, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 gt, gtu, tstbit 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1250 / 第 1001-1250 行

```cpp
  1001:   0xffc02300,
  1002:   0x11c02300,
  1003:   0x003000fe,
  1004:   0 },
  1005: { /*Tag:J4_tstbit0_fp1_jump_nt*/
  1006:   /*p1=tstbit(Rs16,#0); if (!p1.new) jump:nt #r9:2*/
  1007:   0xffc02300,
  1008:   0x13c00300,
  1009:   0x003000fe,
  1010:   0 },
  1011: { /*Tag:J4_tstbit0_fp1_jump_t*/
  1012:   /*p1=tstbit(Rs16,#0); if (!p1.new) jump:t #r9:2*/
  1013:   0xffc02300,
  1014:   0x13c02300,
  1015:   0x003000fe,
  1016:   0 },
  1017: { /*Tag:J4_tstbit0_t_jumpnv_nt*/
  1018:   /*if (tstbit(Ns8.new,#0)) jump:nt #r9:2*/
  1019:   0xffc02000,
  1020:   0x25800000,
  1021:   0x003000fe,
  1022:   0 },
  1023: { /*Tag:J4_tstbit0_t_jumpnv_t*/
  1024:   /*if (tstbit(Ns8.new,#0)) jump:t #r9:2*/
  1025:   0xffc02000,
  1026:   0x25802000,
  1027:   0x003000fe,
  1028:   0 },
  1029: { /*Tag:J4_tstbit0_tp0_jump_nt*/
  1030:   /*p0=tstbit(Rs16,#0); if (p0.new) jump:nt #r9:2*/
  1031:   0xffc02300,
  1032:   0x11800300,
  1033:   0x003000fe,
  1034:   0 },
  1035: { /*Tag:J4_tstbit0_tp0_jump_t*/
  1036:   /*p0=tstbit(Rs16,#0); if (p0.new) jump:t #r9:2*/
  1037:   0xffc02300,
  1038:   0x11802300,
  1039:   0x003000fe,
  1040:   0 },
  1041: { /*Tag:J4_tstbit0_tp1_jump_nt*/
  1042:   /*p1=tstbit(Rs16,#0); if (p1.new) jump:nt #r9:2*/
  1043:   0xffc02300,
  1044:   0x13800300,
  1045:   0x003000fe,
  1046:   0 },
  1047: { /*Tag:J4_tstbit0_tp1_jump_t*/
  1048:   /*p1=tstbit(Rs16,#0); if (p1.new) jump:t #r9:2*/
  1049:   0xffc02300,
  1050:   0x13802300,
  1051:   0x003000fe,
  1052:   0 },
  1053: { /*Tag:L2_loadalignb_io*/
  1054:   /*Ryy32=memb_fifo(Rs32+#s11:0)*/
  1055:   0xf9e00000,
  1056:   0x90800000,
  1057:   0x06003fe0,
  1058:   0 },
  1059: { /*Tag:L2_loadalignh_io*/
  1060:   /*Ryy32=memh_fifo(Rs32+#s11:1)*/
  1061:   0xf9e00000,
  1062:   0x90400000,
  1063:   0x06003fe0,
  1064:   0 },
  1065: { /*Tag:L2_loadbsw2_io*/
  1066:   /*Rd32=membh(Rs32+#s11:1)*/
  1067:   0xf9e00000,
  1068:   0x90200000,
  1069:   0x06003fe0,
  1070:   0 },
  1071: { /*Tag:L2_loadbsw4_io*/
  1072:   /*Rdd32=membh(Rs32+#s11:2)*/
  1073:   0xf9e00000,
  1074:   0x90e00000,
  1075:   0x06003fe0,
  1076:   0 },
  1077: { /*Tag:L2_loadbzw2_io*/
  1078:   /*Rd32=memubh(Rs32+#s11:1)*/
  1079:   0xf9e00000,
  1080:   0x90600000,
  1081:   0x06003fe0,
  1082:   0 },
  1083: { /*Tag:L2_loadbzw4_io*/
  1084:   /*Rdd32=memubh(Rs32+#s11:2)*/
  1085:   0xf9e00000,
  1086:   0x90a00000,
  1087:   0x06003fe0,
  1088:   0 },
  1089: { /*Tag:L2_loadrb_io*/
  1090:   /*Rd32=memb(Rs32+#s11:0)*/
  1091:   0xf9e00000,
  1092:   0x91000000,
  1093:   0x06003fe0,
  1094:   0 },
  1095: { /*Tag:L2_loadrbgp*/
  1096:   /*Rd32=memb(gp+#u16:0)*/
  1097:   0xf9e00000,
  1098:   0x49000000,
  1099:   0x061f3fe0,
  1100:   0 },
  1101: { /*Tag:L2_loadrd_io*/
  1102:   /*Rdd32=memd(Rs32+#s11:3)*/
  1103:   0xf9e00000,
  1104:   0x91c00000,
  1105:   0x06003fe0,
  1106:   0 },
  1107: { /*Tag:L2_loadrdgp*/
  1108:   /*Rdd32=memd(gp+#u16:3)*/
  1109:   0xf9e00000,
  1110:   0x49c00000,
  1111:   0x061f3fe0,
  1112:   0 },
  1113: { /*Tag:L2_loadrh_io*/
  1114:   /*Rd32=memh(Rs32+#s11:1)*/
  1115:   0xf9e00000,
  1116:   0x91400000,
  1117:   0x06003fe0,
  1118:   0 },
  1119: { /*Tag:L2_loadrhgp*/
  1120:   /*Rd32=memh(gp+#u16:1)*/
  1121:   0xf9e00000,
  1122:   0x49400000,
  1123:   0x061f3fe0,
  1124:   0 },
  1125: { /*Tag:L2_loadri_io*/
  1126:   /*Rd32=memw(Rs32+#s11:2)*/
  1127:   0xf9e00000,
  1128:   0x91800000,
  1129:   0x06003fe0,
  1130:   0 },
  1131: { /*Tag:L2_loadrigp*/
  1132:   /*Rd32=memw(gp+#u16:2)*/
  1133:   0xf9e00000,
  1134:   0x49800000,
  1135:   0x061f3fe0,
  1136:   0 },
  1137: { /*Tag:L2_loadrub_io*/
  1138:   /*Rd32=memub(Rs32+#s11:0)*/
  1139:   0xf9e00000,
  1140:   0x91200000,
  1141:   0x06003fe0,
  1142:   0 },
  1143: { /*Tag:L2_loadrubgp*/
  1144:   /*Rd32=memub(gp+#u16:0)*/
  1145:   0xf9e00000,
  1146:   0x49200000,
  1147:   0x061f3fe0,
  1148:   0 },
  1149: { /*Tag:L2_loadruh_io*/
  1150:   /*Rd32=memuh(Rs32+#s11:1)*/
  1151:   0xf9e00000,
  1152:   0x91600000,
  1153:   0x06003fe0,
  1154:   0 },
  1155: { /*Tag:L2_loadruhgp*/
  1156:   /*Rd32=memuh(gp+#u16:1)*/
  1157:   0xf9e00000,
  1158:   0x49600000,
  1159:   0x061f3fe0,
  1160:   0 },
  1161: { /*Tag:L2_ploadrbf_io*/
  1162:   /*if (!Pt4) Rd32=memb(Rs32+#u6:0)*/
  1163:   0xffe02000,
  1164:   0x45000000,
  1165:   0x000007e0,
  1166:   0 },
  1167: { /*Tag:L2_ploadrbfnew_io*/
  1168:   /*if (!Pt4.new) Rd32=memb(Rs32+#u6:0)*/
  1169:   0xffe02000,
  1170:   0x47000000,
  1171:   0x000007e0,
  1172:   0 },
  1173: { /*Tag:L2_ploadrbt_io*/
  1174:   /*if (Pt4) Rd32=memb(Rs32+#u6:0)*/
  1175:   0xffe02000,
  1176:   0x41000000,
  1177:   0x000007e0,
  1178:   0 },
  1179: { /*Tag:L2_ploadrbtnew_io*/
  1180:   /*if (Pt4.new) Rd32=memb(Rs32+#u6:0)*/
  1181:   0xffe02000,
  1182:   0x43000000,
  1183:   0x000007e0,
  1184:   0 },
  1185: { /*Tag:L2_ploadrdf_io*/
  1186:   /*if (!Pt4) Rdd32=memd(Rs32+#u6:3)*/
  1187:   0xffe02000,
  1188:   0x45c00000,
  1189:   0x000007e0,
  1190:   0 },
  1191: { /*Tag:L2_ploadrdfnew_io*/
  1192:   /*if (!Pt4.new) Rdd32=memd(Rs32+#u6:3)*/
  1193:   0xffe02000,
  1194:   0x47c00000,
  1195:   0x000007e0,
  1196:   0 },
  1197: { /*Tag:L2_ploadrdt_io*/
  1198:   /*if (Pt4) Rdd32=memd(Rs32+#u6:3)*/
  1199:   0xffe02000,
  1200:   0x41c00000,
  1201:   0x000007e0,
  1202:   0 },
  1203: { /*Tag:L2_ploadrdtnew_io*/
  1204:   /*if (Pt4.new) Rdd32=memd(Rs32+#u6:3)*/
  1205:   0xffe02000,
  1206:   0x43c00000,
  1207:   0x000007e0,
  1208:   0 },
  1209: { /*Tag:L2_ploadrhf_io*/
  1210:   /*if (!Pt4) Rd32=memh(Rs32+#u6:1)*/
  1211:   0xffe02000,
  1212:   0x45400000,
  1213:   0x000007e0,
  1214:   0 },
  1215: { /*Tag:L2_ploadrhfnew_io*/
  1216:   /*if (!Pt4.new) Rd32=memh(Rs32+#u6:1)*/
  1217:   0xffe02000,
  1218:   0x47400000,
  1219:   0x000007e0,
  1220:   0 },
  1221: { /*Tag:L2_ploadrht_io*/
  1222:   /*if (Pt4) Rd32=memh(Rs32+#u6:1)*/
  1223:   0xffe02000,
  1224:   0x41400000,
  1225:   0x000007e0,
  1226:   0 },
  1227: { /*Tag:L2_ploadrhtnew_io*/
  1228:   /*if (Pt4.new) Rd32=memh(Rs32+#u6:1)*/
  1229:   0xffe02000,
  1230:   0x43400000,
  1231:   0x000007e0,
  1232:   0 },
  1233: { /*Tag:L2_ploadrif_io*/
  1234:   /*if (!Pt4) Rd32=memw(Rs32+#u6:2)*/
  1235:   0xffe02000,
  1236:   0x45800000,
  1237:   0x000007e0,
  1238:   0 },
  1239: { /*Tag:L2_ploadrifnew_io*/
  1240:   /*if (!Pt4.new) Rd32=memw(Rs32+#u6:2)*/
  1241:   0xffe02000,
  1242:   0x47800000,
  1243:   0x000007e0,
  1244:   0 },
  1245: { /*Tag:L2_ploadrit_io*/
  1246:   /*if (Pt4) Rd32=memw(Rs32+#u6:2)*/
  1247:   0xffe02000,
  1248:   0x41800000,
  1249:   0x000007e0,
  1250:   0 },
```
- EN: It declares or implements routines such as tstbit, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 tstbit 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1251-1500 / 第 1251-1500 行

```cpp
  1251: { /*Tag:L2_ploadritnew_io*/
  1252:   /*if (Pt4.new) Rd32=memw(Rs32+#u6:2)*/
  1253:   0xffe02000,
  1254:   0x43800000,
  1255:   0x000007e0,
  1256:   0 },
  1257: { /*Tag:L2_ploadrubf_io*/
  1258:   /*if (!Pt4) Rd32=memub(Rs32+#u6:0)*/
  1259:   0xffe02000,
  1260:   0x45200000,
  1261:   0x000007e0,
  1262:   0 },
  1263: { /*Tag:L2_ploadrubfnew_io*/
  1264:   /*if (!Pt4.new) Rd32=memub(Rs32+#u6:0)*/
  1265:   0xffe02000,
  1266:   0x47200000,
  1267:   0x000007e0,
  1268:   0 },
  1269: { /*Tag:L2_ploadrubt_io*/
  1270:   /*if (Pt4) Rd32=memub(Rs32+#u6:0)*/
  1271:   0xffe02000,
  1272:   0x41200000,
  1273:   0x000007e0,
  1274:   0 },
  1275: { /*Tag:L2_ploadrubtnew_io*/
  1276:   /*if (Pt4.new) Rd32=memub(Rs32+#u6:0)*/
  1277:   0xffe02000,
  1278:   0x43200000,
  1279:   0x000007e0,
  1280:   0 },
  1281: { /*Tag:L2_ploadruhf_io*/
  1282:   /*if (!Pt4) Rd32=memuh(Rs32+#u6:1)*/
  1283:   0xffe02000,
  1284:   0x45600000,
  1285:   0x000007e0,
  1286:   0 },
  1287: { /*Tag:L2_ploadruhfnew_io*/
  1288:   /*if (!Pt4.new) Rd32=memuh(Rs32+#u6:1)*/
  1289:   0xffe02000,
  1290:   0x47600000,
  1291:   0x000007e0,
  1292:   0 },
  1293: { /*Tag:L2_ploadruht_io*/
  1294:   /*if (Pt4) Rd32=memuh(Rs32+#u6:1)*/
  1295:   0xffe02000,
  1296:   0x41600000,
  1297:   0x000007e0,
  1298:   0 },
  1299: { /*Tag:L2_ploadruhtnew_io*/
  1300:   /*if (Pt4.new) Rd32=memuh(Rs32+#u6:1)*/
  1301:   0xffe02000,
  1302:   0x43600000,
  1303:   0x000007e0,
  1304:   0 },
  1305: { /*Tag:L4_add_memopb_io*/
  1306:   /*memb(Rs32+#u6:0)+=Rt32*/
  1307:   0xff602060,
  1308:   0x3e000000,
  1309:   0x00001f80,
  1310:   0 },
  1311: { /*Tag:L4_add_memoph_io*/
  1312:   /*memh(Rs32+#u6:1)+=Rt32*/
  1313:   0xff602060,
  1314:   0x3e200000,
  1315:   0x00001f80,
  1316:   0 },
  1317: { /*Tag:L4_add_memopw_io*/
  1318:   /*memw(Rs32+#u6:2)+=Rt32*/
  1319:   0xff602060,
  1320:   0x3e400000,
  1321:   0x00001f80,
  1322:   0 },
  1323: { /*Tag:L4_and_memopb_io*/
  1324:   /*memb(Rs32+#u6:0)&=Rt32*/
  1325:   0xff602060,
  1326:   0x3e000040,
  1327:   0x00001f80,
  1328:   0 },
  1329: { /*Tag:L4_and_memoph_io*/
  1330:   /*memh(Rs32+#u6:1)&=Rt32*/
  1331:   0xff602060,
  1332:   0x3e200040,
  1333:   0x00001f80,
  1334:   0 },
  1335: { /*Tag:L4_and_memopw_io*/
  1336:   /*memw(Rs32+#u6:2)&=Rt32*/
  1337:   0xff602060,
  1338:   0x3e400040,
  1339:   0x00001f80,
  1340:   0 },
  1341: { /*Tag:L4_iadd_memopb_io*/
  1342:   /*memb(Rs32+#u6:0)+=#U5*/
  1343:   0xff602060,
  1344:   0x3f000000,
  1345:   0x00001f80,
  1346:   0 },
  1347: { /*Tag:L4_iadd_memoph_io*/
  1348:   /*memh(Rs32+#u6:1)+=#U5*/
  1349:   0xff602060,
  1350:   0x3f200000,
  1351:   0x00001f80,
  1352:   0 },
  1353: { /*Tag:L4_iadd_memopw_io*/
  1354:   /*memw(Rs32+#u6:2)+=#U5*/
  1355:   0xff602060,
  1356:   0x3f400000,
  1357:   0x00001f80,
  1358:   0 },
  1359: { /*Tag:L4_iand_memopb_io*/
  1360:   /*memb(Rs32+#u6:0)=clrbit(#U5)*/
  1361:   0xff602060,
  1362:   0x3f000040,
  1363:   0x00001f80,
  1364:   0 },
  1365: { /*Tag:L4_iand_memoph_io*/
  1366:   /*memh(Rs32+#u6:1)=clrbit(#U5)*/
  1367:   0xff602060,
  1368:   0x3f200040,
  1369:   0x00001f80,
  1370:   0 },
  1371: { /*Tag:L4_iand_memopw_io*/
  1372:   /*memw(Rs32+#u6:2)=clrbit(#U5)*/
  1373:   0xff602060,
  1374:   0x3f400040,
  1375:   0x00001f80,
  1376:   0 },
  1377: { /*Tag:L4_ior_memopb_io*/
  1378:   /*memb(Rs32+#u6:0)=setbit(#U5)*/
  1379:   0xff602060,
  1380:   0x3f000060,
  1381:   0x00001f80,
  1382:   0 },
  1383: { /*Tag:L4_ior_memoph_io*/
  1384:   /*memh(Rs32+#u6:1)=setbit(#U5)*/
  1385:   0xff602060,
  1386:   0x3f200060,
  1387:   0x00001f80,
  1388:   0 },
  1389: { /*Tag:L4_ior_memopw_io*/
  1390:   /*memw(Rs32+#u6:2)=setbit(#U5)*/
  1391:   0xff602060,
  1392:   0x3f400060,
  1393:   0x00001f80,
  1394:   0 },
  1395: { /*Tag:L4_isub_memopb_io*/
  1396:   /*memb(Rs32+#u6:0)-=#U5*/
  1397:   0xff602060,
  1398:   0x3f000020,
  1399:   0x00001f80,
  1400:   0 },
  1401: { /*Tag:L4_isub_memoph_io*/
  1402:   /*memh(Rs32+#u6:1)-=#U5*/
  1403:   0xff602060,
  1404:   0x3f200020,
  1405:   0x00001f80,
  1406:   0 },
  1407: { /*Tag:L4_isub_memopw_io*/
  1408:   /*memw(Rs32+#u6:2)-=#U5*/
  1409:   0xff602060,
  1410:   0x3f400020,
  1411:   0x00001f80,
  1412:   0 },
  1413: { /*Tag:L4_loadalignb_ap*/
  1414:   /*Ryy32=memb_fifo(Re32=#U6)*/
  1415:   0xffe03000,
  1416:   0x9a801000,
  1417:   0x00000f60,
  1418:   0 },
  1419: { /*Tag:L4_loadalignb_ur*/
  1420:   /*Ryy32=memb_fifo(Rt32<<#u2+#U6)*/
  1421:   0xffe01000,
  1422:   0x9c801000,
  1423:   0x00000f60,
  1424:   0 },
  1425: { /*Tag:L4_loadalignh_ap*/
  1426:   /*Ryy32=memh_fifo(Re32=#U6)*/
  1427:   0xffe03000,
  1428:   0x9a401000,
  1429:   0x00000f60,
  1430:   0 },
  1431: { /*Tag:L4_loadalignh_ur*/
  1432:   /*Ryy32=memh_fifo(Rt32<<#u2+#U6)*/
  1433:   0xffe01000,
  1434:   0x9c401000,
  1435:   0x00000f60,
  1436:   0 },
  1437: { /*Tag:L4_loadbsw2_ap*/
  1438:   /*Rd32=membh(Re32=#U6)*/
  1439:   0xffe03000,
  1440:   0x9a201000,
  1441:   0x00000f60,
  1442:   0 },
  1443: { /*Tag:L4_loadbsw2_ur*/
  1444:   /*Rd32=membh(Rt32<<#u2+#U6)*/
  1445:   0xffe01000,
  1446:   0x9c201000,
  1447:   0x00000f60,
  1448:   0 },
  1449: { /*Tag:L4_loadbsw4_ap*/
  1450:   /*Rdd32=membh(Re32=#U6)*/
  1451:   0xffe03000,
  1452:   0x9ae01000,
  1453:   0x00000f60,
  1454:   0 },
  1455: { /*Tag:L4_loadbsw4_ur*/
  1456:   /*Rdd32=membh(Rt32<<#u2+#U6)*/
  1457:   0xffe01000,
  1458:   0x9ce01000,
  1459:   0x00000f60,
  1460:   0 },
  1461: { /*Tag:L4_loadbzw2_ap*/
  1462:   /*Rd32=memubh(Re32=#U6)*/
  1463:   0xffe03000,
  1464:   0x9a601000,
  1465:   0x00000f60,
  1466:   0 },
  1467: { /*Tag:L4_loadbzw2_ur*/
  1468:   /*Rd32=memubh(Rt32<<#u2+#U6)*/
  1469:   0xffe01000,
  1470:   0x9c601000,
  1471:   0x00000f60,
  1472:   0 },
  1473: { /*Tag:L4_loadbzw4_ap*/
  1474:   /*Rdd32=memubh(Re32=#U6)*/
  1475:   0xffe03000,
  1476:   0x9aa01000,
  1477:   0x00000f60,
  1478:   0 },
  1479: { /*Tag:L4_loadbzw4_ur*/
  1480:   /*Rdd32=memubh(Rt32<<#u2+#U6)*/
  1481:   0xffe01000,
  1482:   0x9ca01000,
  1483:   0x00000f60,
  1484:   0 },
  1485: { /*Tag:L4_loadrb_ap*/
  1486:   /*Rd32=memb(Re32=#U6)*/
  1487:   0xffe03000,
  1488:   0x9b001000,
  1489:   0x00000f60,
  1490:   0 },
  1491: { /*Tag:L4_loadrb_ur*/
  1492:   /*Rd32=memb(Rt32<<#u2+#U6)*/
  1493:   0xffe01000,
  1494:   0x9d001000,
  1495:   0x00000f60,
  1496:   0 },
  1497: { /*Tag:L4_loadrd_ap*/
  1498:   /*Rdd32=memd(Re32=#U6)*/
  1499:   0xffe03000,
  1500:   0x9bc01000,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1501-1750 / 第 1501-1750 行

```cpp
  1501:   0x00000f60,
  1502:   0 },
  1503: { /*Tag:L4_loadrd_ur*/
  1504:   /*Rdd32=memd(Rt32<<#u2+#U6)*/
  1505:   0xffe01000,
  1506:   0x9dc01000,
  1507:   0x00000f60,
  1508:   0 },
  1509: { /*Tag:L4_loadrh_ap*/
  1510:   /*Rd32=memh(Re32=#U6)*/
  1511:   0xffe03000,
  1512:   0x9b401000,
  1513:   0x00000f60,
  1514:   0 },
  1515: { /*Tag:L4_loadrh_ur*/
  1516:   /*Rd32=memh(Rt32<<#u2+#U6)*/
  1517:   0xffe01000,
  1518:   0x9d401000,
  1519:   0x00000f60,
  1520:   0 },
  1521: { /*Tag:L4_loadri_ap*/
  1522:   /*Rd32=memw(Re32=#U6)*/
  1523:   0xffe03000,
  1524:   0x9b801000,
  1525:   0x00000f60,
  1526:   0 },
  1527: { /*Tag:L4_loadri_ur*/
  1528:   /*Rd32=memw(Rt32<<#u2+#U6)*/
  1529:   0xffe01000,
  1530:   0x9d801000,
  1531:   0x00000f60,
  1532:   0 },
  1533: { /*Tag:L4_loadrub_ap*/
  1534:   /*Rd32=memub(Re32=#U6)*/
  1535:   0xffe03000,
  1536:   0x9b201000,
  1537:   0x00000f60,
  1538:   0 },
  1539: { /*Tag:L4_loadrub_ur*/
  1540:   /*Rd32=memub(Rt32<<#u2+#U6)*/
  1541:   0xffe01000,
  1542:   0x9d201000,
  1543:   0x00000f60,
  1544:   0 },
  1545: { /*Tag:L4_loadruh_ap*/
  1546:   /*Rd32=memuh(Re32=#U6)*/
  1547:   0xffe03000,
  1548:   0x9b601000,
  1549:   0x00000f60,
  1550:   0 },
  1551: { /*Tag:L4_loadruh_ur*/
  1552:   /*Rd32=memuh(Rt32<<#u2+#U6)*/
  1553:   0xffe01000,
  1554:   0x9d601000,
  1555:   0x00000f60,
  1556:   0 },
  1557: { /*Tag:L4_or_memopb_io*/
  1558:   /*memb(Rs32+#u6:0)|=Rt32*/
  1559:   0xff602060,
  1560:   0x3e000060,
  1561:   0x00001f80,
  1562:   0 },
  1563: { /*Tag:L4_or_memoph_io*/
  1564:   /*memh(Rs32+#u6:1)|=Rt32*/
  1565:   0xff602060,
  1566:   0x3e200060,
  1567:   0x00001f80,
  1568:   0 },
  1569: { /*Tag:L4_or_memopw_io*/
  1570:   /*memw(Rs32+#u6:2)|=Rt32*/
  1571:   0xff602060,
  1572:   0x3e400060,
  1573:   0x00001f80,
  1574:   0 },
  1575: { /*Tag:L4_ploadrbf_abs*/
  1576:   /*if (!Pt4) Rd32=memb(#u6)*/
  1577:   0xffe03880,
  1578:   0x9f002880,
  1579:   0x001f0100,
  1580:   0 },
  1581: { /*Tag:L4_ploadrbfnew_abs*/
  1582:   /*if (!Pt4.new) Rd32=memb(#u6)*/
  1583:   0xffe03880,
  1584:   0x9f003880,
  1585:   0x001f0100,
  1586:   0 },
  1587: { /*Tag:L4_ploadrbt_abs*/
  1588:   /*if (Pt4) Rd32=memb(#u6)*/
  1589:   0xffe03880,
  1590:   0x9f002080,
  1591:   0x001f0100,
  1592:   0 },
  1593: { /*Tag:L4_ploadrbtnew_abs*/
  1594:   /*if (Pt4.new) Rd32=memb(#u6)*/
  1595:   0xffe03880,
  1596:   0x9f003080,
  1597:   0x001f0100,
  1598:   0 },
  1599: { /*Tag:L4_ploadrdf_abs*/
  1600:   /*if (!Pt4) Rdd32=memd(#u6)*/
  1601:   0xffe03880,
  1602:   0x9fc02880,
  1603:   0x001f0100,
  1604:   0 },
  1605: { /*Tag:L4_ploadrdfnew_abs*/
  1606:   /*if (!Pt4.new) Rdd32=memd(#u6)*/
  1607:   0xffe03880,
  1608:   0x9fc03880,
  1609:   0x001f0100,
  1610:   0 },
  1611: { /*Tag:L4_ploadrdt_abs*/
  1612:   /*if (Pt4) Rdd32=memd(#u6)*/
  1613:   0xffe03880,
  1614:   0x9fc02080,
  1615:   0x001f0100,
  1616:   0 },
  1617: { /*Tag:L4_ploadrdtnew_abs*/
  1618:   /*if (Pt4.new) Rdd32=memd(#u6)*/
  1619:   0xffe03880,
  1620:   0x9fc03080,
  1621:   0x001f0100,
  1622:   0 },
  1623: { /*Tag:L4_ploadrhf_abs*/
  1624:   /*if (!Pt4) Rd32=memh(#u6)*/
  1625:   0xffe03880,
  1626:   0x9f402880,
  1627:   0x001f0100,
  1628:   0 },
  1629: { /*Tag:L4_ploadrhfnew_abs*/
  1630:   /*if (!Pt4.new) Rd32=memh(#u6)*/
  1631:   0xffe03880,
  1632:   0x9f403880,
  1633:   0x001f0100,
  1634:   0 },
  1635: { /*Tag:L4_ploadrht_abs*/
  1636:   /*if (Pt4) Rd32=memh(#u6)*/
  1637:   0xffe03880,
  1638:   0x9f402080,
  1639:   0x001f0100,
  1640:   0 },
  1641: { /*Tag:L4_ploadrhtnew_abs*/
  1642:   /*if (Pt4.new) Rd32=memh(#u6)*/
  1643:   0xffe03880,
  1644:   0x9f403080,
  1645:   0x001f0100,
  1646:   0 },
  1647: { /*Tag:L4_ploadrif_abs*/
  1648:   /*if (!Pt4) Rd32=memw(#u6)*/
  1649:   0xffe03880,
  1650:   0x9f802880,
  1651:   0x001f0100,
  1652:   0 },
  1653: { /*Tag:L4_ploadrifnew_abs*/
  1654:   /*if (!Pt4.new) Rd32=memw(#u6)*/
  1655:   0xffe03880,
  1656:   0x9f803880,
  1657:   0x001f0100,
  1658:   0 },
  1659: { /*Tag:L4_ploadrit_abs*/
  1660:   /*if (Pt4) Rd32=memw(#u6)*/
  1661:   0xffe03880,
  1662:   0x9f802080,
  1663:   0x001f0100,
  1664:   0 },
  1665: { /*Tag:L4_ploadritnew_abs*/
  1666:   /*if (Pt4.new) Rd32=memw(#u6)*/
  1667:   0xffe03880,
  1668:   0x9f803080,
  1669:   0x001f0100,
  1670:   0 },
  1671: { /*Tag:L4_ploadrubf_abs*/
  1672:   /*if (!Pt4) Rd32=memub(#u6)*/
  1673:   0xffe03880,
  1674:   0x9f202880,
  1675:   0x001f0100,
  1676:   0 },
  1677: { /*Tag:L4_ploadrubfnew_abs*/
  1678:   /*if (!Pt4.new) Rd32=memub(#u6)*/
  1679:   0xffe03880,
  1680:   0x9f203880,
  1681:   0x001f0100,
  1682:   0 },
  1683: { /*Tag:L4_ploadrubt_abs*/
  1684:   /*if (Pt4) Rd32=memub(#u6)*/
  1685:   0xffe03880,
  1686:   0x9f202080,
  1687:   0x001f0100,
  1688:   0 },
  1689: { /*Tag:L4_ploadrubtnew_abs*/
  1690:   /*if (Pt4.new) Rd32=memub(#u6)*/
  1691:   0xffe03880,
  1692:   0x9f203080,
  1693:   0x001f0100,
  1694:   0 },
  1695: { /*Tag:L4_ploadruhf_abs*/
  1696:   /*if (!Pt4) Rd32=memuh(#u6)*/
  1697:   0xffe03880,
  1698:   0x9f602880,
  1699:   0x001f0100,
  1700:   0 },
  1701: { /*Tag:L4_ploadruhfnew_abs*/
  1702:   /*if (!Pt4.new) Rd32=memuh(#u6)*/
  1703:   0xffe03880,
  1704:   0x9f603880,
  1705:   0x001f0100,
  1706:   0 },
  1707: { /*Tag:L4_ploadruht_abs*/
  1708:   /*if (Pt4) Rd32=memuh(#u6)*/
  1709:   0xffe03880,
  1710:   0x9f602080,
  1711:   0x001f0100,
  1712:   0 },
  1713: { /*Tag:L4_ploadruhtnew_abs*/
  1714:   /*if (Pt4.new) Rd32=memuh(#u6)*/
  1715:   0xffe03880,
  1716:   0x9f603080,
  1717:   0x001f0100,
  1718:   0 },
  1719: { /*Tag:L4_sub_memopb_io*/
  1720:   /*memb(Rs32+#u6:0)-=Rt32*/
  1721:   0xff602060,
  1722:   0x3e000020,
  1723:   0x00001f80,
  1724:   0 },
  1725: { /*Tag:L4_sub_memoph_io*/
  1726:   /*memh(Rs32+#u6:1)-=Rt32*/
  1727:   0xff602060,
  1728:   0x3e200020,
  1729:   0x00001f80,
  1730:   0 },
  1731: { /*Tag:L4_sub_memopw_io*/
  1732:   /*memw(Rs32+#u6:2)-=Rt32*/
  1733:   0xff602060,
  1734:   0x3e400020,
  1735:   0x00001f80,
  1736:   0 },
  1737: { /*Tag:M2_accii*/
  1738:   /*Rx32+=add(Rs32,#s8)*/
  1739:   0xff802000,
  1740:   0xe2000000,
  1741:   0x00001fe0,
  1742:   0 },
  1743: { /*Tag:M2_macsin*/
  1744:   /*Rx32-=mpyi(Rs32,#u8)*/
  1745:   0xff802000,
  1746:   0xe1800000,
  1747:   0x00001fe0,
  1748:   0 },
  1749: { /*Tag:M2_macsip*/
  1750:   /*Rx32+=mpyi(Rs32,#u8)*/
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1751-2000 / 第 1751-2000 行

```cpp
  1751:   0xff802000,
  1752:   0xe1000000,
  1753:   0x00001fe0,
  1754:   0 },
  1755: { /*Tag:M2_mpysip*/
  1756:   /*Rd32=+mpyi(Rs32,#u8)*/
  1757:   0xff802000,
  1758:   0xe0000000,
  1759:   0x00001fe0,
  1760:   0 },
  1761: { /*Tag:M2_naccii*/
  1762:   /*Rx32-=add(Rs32,#s8)*/
  1763:   0xff802000,
  1764:   0xe2800000,
  1765:   0x00001fe0,
  1766:   0 },
  1767: { /*Tag:M4_mpyri_addi*/
  1768:   /*Rd32=add(#u6,mpyi(Rs32,#U6))*/
  1769:   0xff000000,
  1770:   0xd8000000,
  1771:   0x006020e0,
  1772:   0 },
  1773: { /*Tag:M4_mpyri_addr*/
  1774:   /*Rd32=add(Ru32,mpyi(Rs32,#u6))*/
  1775:   0xff800000,
  1776:   0xdf800000,
  1777:   0x006020e0,
  1778:   0 },
  1779: { /*Tag:M4_mpyrr_addi*/
  1780:   /*Rd32=add(#u6,mpyi(Rs32,Rt32))*/
  1781:   0xff800000,
  1782:   0xd7000000,
  1783:   0x006020e0,
  1784:   0 },
  1785: { /*Tag:PS_loadrbabs*/
  1786:   /*Rd32=memb(#u16:0)*/
  1787:   0xf9e00000,
  1788:   0x49000000,
  1789:   0x061f3fe0,
  1790:   0 },
  1791: { /*Tag:PS_loadrdabs*/
  1792:   /*Rdd32=memd(#u16:3)*/
  1793:   0xf9e00000,
  1794:   0x49c00000,
  1795:   0x061f3fe0,
  1796:   0 },
  1797: { /*Tag:PS_loadrhabs*/
  1798:   /*Rd32=memh(#u16:1)*/
  1799:   0xf9e00000,
  1800:   0x49400000,
  1801:   0x061f3fe0,
  1802:   0 },
  1803: { /*Tag:PS_loadriabs*/
  1804:   /*Rd32=memw(#u16:2)*/
  1805:   0xf9e00000,
  1806:   0x49800000,
  1807:   0x061f3fe0,
  1808:   0 },
  1809: { /*Tag:PS_loadrubabs*/
  1810:   /*Rd32=memub(#u16:0)*/
  1811:   0xf9e00000,
  1812:   0x49200000,
  1813:   0x061f3fe0,
  1814:   0 },
  1815: { /*Tag:PS_loadruhabs*/
  1816:   /*Rd32=memuh(#u16:1)*/
  1817:   0xf9e00000,
  1818:   0x49600000,
  1819:   0x061f3fe0,
  1820:   0 },
  1821: { /*Tag:PS_storerbabs*/
  1822:   /*memb(#u16:0)=Rt32*/
  1823:   0xf9e00000,
  1824:   0x48000000,
  1825:   0x061f20ff,
  1826:   0 },
  1827: { /*Tag:PS_storerbnewabs*/
  1828:   /*memb(#u16:0)=Nt8.new*/
  1829:   0xf9e01800,
  1830:   0x48a00000,
  1831:   0x061f20ff,
  1832:   0 },
  1833: { /*Tag:PS_storerdabs*/
  1834:   /*memd(#u16:3)=Rtt32*/
  1835:   0xf9e00000,
  1836:   0x48c00000,
  1837:   0x061f20ff,
  1838:   0 },
  1839: { /*Tag:PS_storerfabs*/
  1840:   /*memh(#u16:1)=Rt32.h*/
  1841:   0xf9e00000,
  1842:   0x48600000,
  1843:   0x061f20ff,
  1844:   0 },
  1845: { /*Tag:PS_storerhabs*/
  1846:   /*memh(#u16:1)=Rt32*/
  1847:   0xf9e00000,
  1848:   0x48400000,
  1849:   0x061f20ff,
  1850:   0 },
  1851: { /*Tag:PS_storerhnewabs*/
  1852:   /*memh(#u16:1)=Nt8.new*/
  1853:   0xf9e01800,
  1854:   0x48a00800,
  1855:   0x061f20ff,
  1856:   0 },
  1857: { /*Tag:PS_storeriabs*/
  1858:   /*memw(#u16:2)=Rt32*/
  1859:   0xf9e00000,
  1860:   0x48800000,
  1861:   0x061f20ff,
  1862:   0 },
  1863: { /*Tag:PS_storerinewabs*/
  1864:   /*memw(#u16:2)=Nt8.new*/
  1865:   0xf9e01800,
  1866:   0x48a01000,
  1867:   0x061f20ff,
  1868:   0 },
  1869: { /*Tag:S2_pstorerbf_io*/
  1870:   /*if (!Pv4) memb(Rs32+#u6:0)=Rt32*/
  1871:   0xffe00004,
  1872:   0x44000000,
  1873:   0x000020f8,
  1874:   0 },
  1875: { /*Tag:S2_pstorerbnewf_io*/
  1876:   /*if (!Pv4) memb(Rs32+#u6:0)=Nt8.new*/
  1877:   0xffe01804,
  1878:   0x44a00000,
  1879:   0x000020f8,
  1880:   0 },
  1881: { /*Tag:S2_pstorerbnewt_io*/
  1882:   /*if (Pv4) memb(Rs32+#u6:0)=Nt8.new*/
  1883:   0xffe01804,
  1884:   0x40a00000,
  1885:   0x000020f8,
  1886:   0 },
  1887: { /*Tag:S2_pstorerbt_io*/
  1888:   /*if (Pv4) memb(Rs32+#u6:0)=Rt32*/
  1889:   0xffe00004,
  1890:   0x40000000,
  1891:   0x000020f8,
  1892:   0 },
  1893: { /*Tag:S2_pstorerdf_io*/
  1894:   /*if (!Pv4) memd(Rs32+#u6:3)=Rtt32*/
  1895:   0xffe00004,
  1896:   0x44c00000,
  1897:   0x000020f8,
  1898:   0 },
  1899: { /*Tag:S2_pstorerdt_io*/
  1900:   /*if (Pv4) memd(Rs32+#u6:3)=Rtt32*/
  1901:   0xffe00004,
  1902:   0x40c00000,
  1903:   0x000020f8,
  1904:   0 },
  1905: { /*Tag:S2_pstorerff_io*/
  1906:   /*if (!Pv4) memh(Rs32+#u6:1)=Rt32.h*/
  1907:   0xffe00004,
  1908:   0x44600000,
  1909:   0x000020f8,
  1910:   0 },
  1911: { /*Tag:S2_pstorerft_io*/
  1912:   /*if (Pv4) memh(Rs32+#u6:1)=Rt32.h*/
  1913:   0xffe00004,
  1914:   0x40600000,
  1915:   0x000020f8,
  1916:   0 },
  1917: { /*Tag:S2_pstorerhf_io*/
  1918:   /*if (!Pv4) memh(Rs32+#u6:1)=Rt32*/
  1919:   0xffe00004,
  1920:   0x44400000,
  1921:   0x000020f8,
  1922:   0 },
  1923: { /*Tag:S2_pstorerhnewf_io*/
  1924:   /*if (!Pv4) memh(Rs32+#u6:1)=Nt8.new*/
  1925:   0xffe01804,
  1926:   0x44a00800,
  1927:   0x000020f8,
  1928:   0 },
  1929: { /*Tag:S2_pstorerhnewt_io*/
  1930:   /*if (Pv4) memh(Rs32+#u6:1)=Nt8.new*/
  1931:   0xffe01804,
  1932:   0x40a00800,
  1933:   0x000020f8,
  1934:   0 },
  1935: { /*Tag:S2_pstorerht_io*/
  1936:   /*if (Pv4) memh(Rs32+#u6:1)=Rt32*/
  1937:   0xffe00004,
  1938:   0x40400000,
  1939:   0x000020f8,
  1940:   0 },
  1941: { /*Tag:S2_pstorerif_io*/
  1942:   /*if (!Pv4) memw(Rs32+#u6:2)=Rt32*/
  1943:   0xffe00004,
  1944:   0x44800000,
  1945:   0x000020f8,
  1946:   0 },
  1947: { /*Tag:S2_pstorerinewf_io*/
  1948:   /*if (!Pv4) memw(Rs32+#u6:2)=Nt8.new*/
  1949:   0xffe01804,
  1950:   0x44a01000,
  1951:   0x000020f8,
  1952:   0 },
  1953: { /*Tag:S2_pstorerinewt_io*/
  1954:   /*if (Pv4) memw(Rs32+#u6:2)=Nt8.new*/
  1955:   0xffe01804,
  1956:   0x40a01000,
  1957:   0x000020f8,
  1958:   0 },
  1959: { /*Tag:S2_pstorerit_io*/
  1960:   /*if (Pv4) memw(Rs32+#u6:2)=Rt32*/
  1961:   0xffe00004,
  1962:   0x40800000,
  1963:   0x000020f8,
  1964:   0 },
  1965: { /*Tag:S2_storerb_io*/
  1966:   /*memb(Rs32+#s11:0)=Rt32*/
  1967:   0xf9e00000,
  1968:   0xa1000000,
  1969:   0x060020ff,
  1970:   0 },
  1971: { /*Tag:S2_storerbgp*/
  1972:   /*memb(gp+#u16:0)=Rt32*/
  1973:   0xf9e00000,
  1974:   0x48000000,
  1975:   0x061f20ff,
  1976:   0 },
  1977: { /*Tag:S2_storerbnew_io*/
  1978:   /*memb(Rs32+#s11:0)=Nt8.new*/
  1979:   0xf9e01800,
  1980:   0xa1a00000,
  1981:   0x060020ff,
  1982:   0 },
  1983: { /*Tag:S2_storerbnewgp*/
  1984:   /*memb(gp+#u16:0)=Nt8.new*/
  1985:   0xf9e01800,
  1986:   0x48a00000,
  1987:   0x061f20ff,
  1988:   0 },
  1989: { /*Tag:S2_storerd_io*/
  1990:   /*memd(Rs32+#s11:3)=Rtt32*/
  1991:   0xf9e00000,
  1992:   0xa1c00000,
  1993:   0x060020ff,
  1994:   0 },
  1995: { /*Tag:S2_storerdgp*/
  1996:   /*memd(gp+#u16:3)=Rtt32*/
  1997:   0xf9e00000,
  1998:   0x48c00000,
  1999:   0x061f20ff,
  2000:   0 },
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2001-2250 / 第 2001-2250 行

```cpp
  2001: { /*Tag:S2_storerf_io*/
  2002:   /*memh(Rs32+#s11:1)=Rt32.h*/
  2003:   0xf9e00000,
  2004:   0xa1600000,
  2005:   0x060020ff,
  2006:   0 },
  2007: { /*Tag:S2_storerfgp*/
  2008:   /*memh(gp+#u16:1)=Rt32.h*/
  2009:   0xf9e00000,
  2010:   0x48600000,
  2011:   0x061f20ff,
  2012:   0 },
  2013: { /*Tag:S2_storerh_io*/
  2014:   /*memh(Rs32+#s11:1)=Rt32*/
  2015:   0xf9e00000,
  2016:   0xa1400000,
  2017:   0x060020ff,
  2018:   0 },
  2019: { /*Tag:S2_storerhgp*/
  2020:   /*memh(gp+#u16:1)=Rt32*/
  2021:   0xf9e00000,
  2022:   0x48400000,
  2023:   0x061f20ff,
  2024:   0 },
  2025: { /*Tag:S2_storerhnew_io*/
  2026:   /*memh(Rs32+#s11:1)=Nt8.new*/
  2027:   0xf9e01800,
  2028:   0xa1a00800,
  2029:   0x060020ff,
  2030:   0 },
  2031: { /*Tag:S2_storerhnewgp*/
  2032:   /*memh(gp+#u16:1)=Nt8.new*/
  2033:   0xf9e01800,
  2034:   0x48a00800,
  2035:   0x061f20ff,
  2036:   0 },
  2037: { /*Tag:S2_storeri_io*/
  2038:   /*memw(Rs32+#s11:2)=Rt32*/
  2039:   0xf9e00000,
  2040:   0xa1800000,
  2041:   0x060020ff,
  2042:   0 },
  2043: { /*Tag:S2_storerigp*/
  2044:   /*memw(gp+#u16:2)=Rt32*/
  2045:   0xf9e00000,
  2046:   0x48800000,
  2047:   0x061f20ff,
  2048:   0 },
  2049: { /*Tag:S2_storerinew_io*/
  2050:   /*memw(Rs32+#s11:2)=Nt8.new*/
  2051:   0xf9e01800,
  2052:   0xa1a01000,
  2053:   0x060020ff,
  2054:   0 },
  2055: { /*Tag:S2_storerinewgp*/
  2056:   /*memw(gp+#u16:2)=Nt8.new*/
  2057:   0xf9e01800,
  2058:   0x48a01000,
  2059:   0x061f20ff,
  2060:   0 },
  2061: { /*Tag:S4_addaddi*/
  2062:   /*Rd32=add(Rs32,add(Ru32,#s6))*/
  2063:   0xff800000,
  2064:   0xdb000000,
  2065:   0x006020e0,
  2066:   0 },
  2067: { /*Tag:S4_addi_asl_ri*/
  2068:   /*Rx32=add(#u8,asl(Rx32,#U5))*/
  2069:   0xff000016,
  2070:   0xde000004,
  2071:   0x00e020e8,
  2072:   0 },
  2073: { /*Tag:S4_addi_lsr_ri*/
  2074:   /*Rx32=add(#u8,lsr(Rx32,#U5))*/
  2075:   0xff000016,
  2076:   0xde000014,
  2077:   0x00e020e8,
  2078:   0 },
  2079: { /*Tag:S4_andi_asl_ri*/
  2080:   /*Rx32=and(#u8,asl(Rx32,#U5))*/
  2081:   0xff000016,
  2082:   0xde000000,
  2083:   0x00e020e8,
  2084:   0 },
  2085: { /*Tag:S4_andi_lsr_ri*/
  2086:   /*Rx32=and(#u8,lsr(Rx32,#U5))*/
  2087:   0xff000016,
  2088:   0xde000010,
  2089:   0x00e020e8,
  2090:   0 },
  2091: { /*Tag:S4_or_andi*/
  2092:   /*Rx32|=and(Rs32,#s10)*/
  2093:   0xffc00000,
  2094:   0xda000000,
  2095:   0x00203fe0,
  2096:   0 },
  2097: { /*Tag:S4_or_andix*/
  2098:   /*Rx32=or(Ru32,and(Rx32,#s10))*/
  2099:   0xffc00000,
  2100:   0xda400000,
  2101:   0x00203fe0,
  2102:   0 },
  2103: { /*Tag:S4_or_ori*/
  2104:   /*Rx32|=or(Rs32,#s10)*/
  2105:   0xffc00000,
  2106:   0xda800000,
  2107:   0x00203fe0,
  2108:   0 },
  2109: { /*Tag:S4_ori_asl_ri*/
  2110:   /*Rx32=or(#u8,asl(Rx32,#U5))*/
  2111:   0xff000016,
  2112:   0xde000002,
  2113:   0x00e020e8,
  2114:   0 },
  2115: { /*Tag:S4_ori_lsr_ri*/
  2116:   /*Rx32=or(#u8,lsr(Rx32,#U5))*/
  2117:   0xff000016,
  2118:   0xde000012,
  2119:   0x00e020e8,
  2120:   0 },
  2121: { /*Tag:S4_pstorerbf_abs*/
  2122:   /*if (!Pv4) memb(#u6)=Rt32*/
  2123:   0xffe02084,
  2124:   0xaf000084,
  2125:   0x00030078,
  2126:   0 },
  2127: { /*Tag:S4_pstorerbfnew_abs*/
  2128:   /*if (!Pv4.new) memb(#u6)=Rt32*/
  2129:   0xffe02084,
  2130:   0xaf002084,
  2131:   0x00030078,
  2132:   0 },
  2133: { /*Tag:S4_pstorerbfnew_io*/
  2134:   /*if (!Pv4.new) memb(Rs32+#u6:0)=Rt32*/
  2135:   0xffe00004,
  2136:   0x46000000,
  2137:   0x000020f8,
  2138:   0 },
  2139: { /*Tag:S4_pstorerbnewf_abs*/
  2140:   /*if (!Pv4) memb(#u6)=Nt8.new*/
  2141:   0xffe03884,
  2142:   0xafa00084,
  2143:   0x00030078,
  2144:   0 },
  2145: { /*Tag:S4_pstorerbnewfnew_abs*/
  2146:   /*if (!Pv4.new) memb(#u6)=Nt8.new*/
  2147:   0xffe03884,
  2148:   0xafa02084,
  2149:   0x00030078,
  2150:   0 },
  2151: { /*Tag:S4_pstorerbnewfnew_io*/
  2152:   /*if (!Pv4.new) memb(Rs32+#u6:0)=Nt8.new*/
  2153:   0xffe01804,
  2154:   0x46a00000,
  2155:   0x000020f8,
  2156:   0 },
  2157: { /*Tag:S4_pstorerbnewt_abs*/
  2158:   /*if (Pv4) memb(#u6)=Nt8.new*/
  2159:   0xffe03884,
  2160:   0xafa00080,
  2161:   0x00030078,
  2162:   0 },
  2163: { /*Tag:S4_pstorerbnewtnew_abs*/
  2164:   /*if (Pv4.new) memb(#u6)=Nt8.new*/
  2165:   0xffe03884,
  2166:   0xafa02080,
  2167:   0x00030078,
  2168:   0 },
  2169: { /*Tag:S4_pstorerbnewtnew_io*/
  2170:   /*if (Pv4.new) memb(Rs32+#u6:0)=Nt8.new*/
  2171:   0xffe01804,
  2172:   0x42a00000,
  2173:   0x000020f8,
  2174:   0 },
  2175: { /*Tag:S4_pstorerbt_abs*/
  2176:   /*if (Pv4) memb(#u6)=Rt32*/
  2177:   0xffe02084,
  2178:   0xaf000080,
  2179:   0x00030078,
  2180:   0 },
  2181: { /*Tag:S4_pstorerbtnew_abs*/
  2182:   /*if (Pv4.new) memb(#u6)=Rt32*/
  2183:   0xffe02084,
  2184:   0xaf002080,
  2185:   0x00030078,
  2186:   0 },
  2187: { /*Tag:S4_pstorerbtnew_io*/
  2188:   /*if (Pv4.new) memb(Rs32+#u6:0)=Rt32*/
  2189:   0xffe00004,
  2190:   0x42000000,
  2191:   0x000020f8,
  2192:   0 },
  2193: { /*Tag:S4_pstorerdf_abs*/
  2194:   /*if (!Pv4) memd(#u6)=Rtt32*/
  2195:   0xffe02084,
  2196:   0xafc00084,
  2197:   0x00030078,
  2198:   0 },
  2199: { /*Tag:S4_pstorerdfnew_abs*/
  2200:   /*if (!Pv4.new) memd(#u6)=Rtt32*/
  2201:   0xffe02084,
  2202:   0xafc02084,
  2203:   0x00030078,
  2204:   0 },
  2205: { /*Tag:S4_pstorerdfnew_io*/
  2206:   /*if (!Pv4.new) memd(Rs32+#u6:3)=Rtt32*/
  2207:   0xffe00004,
  2208:   0x46c00000,
  2209:   0x000020f8,
  2210:   0 },
  2211: { /*Tag:S4_pstorerdt_abs*/
  2212:   /*if (Pv4) memd(#u6)=Rtt32*/
  2213:   0xffe02084,
  2214:   0xafc00080,
  2215:   0x00030078,
  2216:   0 },
  2217: { /*Tag:S4_pstorerdtnew_abs*/
  2218:   /*if (Pv4.new) memd(#u6)=Rtt32*/
  2219:   0xffe02084,
  2220:   0xafc02080,
  2221:   0x00030078,
  2222:   0 },
  2223: { /*Tag:S4_pstorerdtnew_io*/
  2224:   /*if (Pv4.new) memd(Rs32+#u6:3)=Rtt32*/
  2225:   0xffe00004,
  2226:   0x42c00000,
  2227:   0x000020f8,
  2228:   0 },
  2229: { /*Tag:S4_pstorerff_abs*/
  2230:   /*if (!Pv4) memh(#u6)=Rt32.h*/
  2231:   0xffe02084,
  2232:   0xaf600084,
  2233:   0x00030078,
  2234:   0 },
  2235: { /*Tag:S4_pstorerffnew_abs*/
  2236:   /*if (!Pv4.new) memh(#u6)=Rt32.h*/
  2237:   0xffe02084,
  2238:   0xaf602084,
  2239:   0x00030078,
  2240:   0 },
  2241: { /*Tag:S4_pstorerffnew_io*/
  2242:   /*if (!Pv4.new) memh(Rs32+#u6:1)=Rt32.h*/
  2243:   0xffe00004,
  2244:   0x46600000,
  2245:   0x000020f8,
  2246:   0 },
  2247: { /*Tag:S4_pstorerft_abs*/
  2248:   /*if (Pv4) memh(#u6)=Rt32.h*/
  2249:   0xffe02084,
  2250:   0xaf600080,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2251-2500 / 第 2251-2500 行

```cpp
  2251:   0x00030078,
  2252:   0 },
  2253: { /*Tag:S4_pstorerftnew_abs*/
  2254:   /*if (Pv4.new) memh(#u6)=Rt32.h*/
  2255:   0xffe02084,
  2256:   0xaf602080,
  2257:   0x00030078,
  2258:   0 },
  2259: { /*Tag:S4_pstorerftnew_io*/
  2260:   /*if (Pv4.new) memh(Rs32+#u6:1)=Rt32.h*/
  2261:   0xffe00004,
  2262:   0x42600000,
  2263:   0x000020f8,
  2264:   0 },
  2265: { /*Tag:S4_pstorerhf_abs*/
  2266:   /*if (!Pv4) memh(#u6)=Rt32*/
  2267:   0xffe02084,
  2268:   0xaf400084,
  2269:   0x00030078,
  2270:   0 },
  2271: { /*Tag:S4_pstorerhfnew_abs*/
  2272:   /*if (!Pv4.new) memh(#u6)=Rt32*/
  2273:   0xffe02084,
  2274:   0xaf402084,
  2275:   0x00030078,
  2276:   0 },
  2277: { /*Tag:S4_pstorerhfnew_io*/
  2278:   /*if (!Pv4.new) memh(Rs32+#u6:1)=Rt32*/
  2279:   0xffe00004,
  2280:   0x46400000,
  2281:   0x000020f8,
  2282:   0 },
  2283: { /*Tag:S4_pstorerhnewf_abs*/
  2284:   /*if (!Pv4) memh(#u6)=Nt8.new*/
  2285:   0xffe03884,
  2286:   0xafa00884,
  2287:   0x00030078,
  2288:   0 },
  2289: { /*Tag:S4_pstorerhnewfnew_abs*/
  2290:   /*if (!Pv4.new) memh(#u6)=Nt8.new*/
  2291:   0xffe03884,
  2292:   0xafa02884,
  2293:   0x00030078,
  2294:   0 },
  2295: { /*Tag:S4_pstorerhnewfnew_io*/
  2296:   /*if (!Pv4.new) memh(Rs32+#u6:1)=Nt8.new*/
  2297:   0xffe01804,
  2298:   0x46a00800,
  2299:   0x000020f8,
  2300:   0 },
  2301: { /*Tag:S4_pstorerhnewt_abs*/
  2302:   /*if (Pv4) memh(#u6)=Nt8.new*/
  2303:   0xffe03884,
  2304:   0xafa00880,
  2305:   0x00030078,
  2306:   0 },
  2307: { /*Tag:S4_pstorerhnewtnew_abs*/
  2308:   /*if (Pv4.new) memh(#u6)=Nt8.new*/
  2309:   0xffe03884,
  2310:   0xafa02880,
  2311:   0x00030078,
  2312:   0 },
  2313: { /*Tag:S4_pstorerhnewtnew_io*/
  2314:   /*if (Pv4.new) memh(Rs32+#u6:1)=Nt8.new*/
  2315:   0xffe01804,
  2316:   0x42a00800,
  2317:   0x000020f8,
  2318:   0 },
  2319: { /*Tag:S4_pstorerht_abs*/
  2320:   /*if (Pv4) memh(#u6)=Rt32*/
  2321:   0xffe02084,
  2322:   0xaf400080,
  2323:   0x00030078,
  2324:   0 },
  2325: { /*Tag:S4_pstorerhtnew_abs*/
  2326:   /*if (Pv4.new) memh(#u6)=Rt32*/
  2327:   0xffe02084,
  2328:   0xaf402080,
  2329:   0x00030078,
  2330:   0 },
  2331: { /*Tag:S4_pstorerhtnew_io*/
  2332:   /*if (Pv4.new) memh(Rs32+#u6:1)=Rt32*/
  2333:   0xffe00004,
  2334:   0x42400000,
  2335:   0x000020f8,
  2336:   0 },
  2337: { /*Tag:S4_pstorerif_abs*/
  2338:   /*if (!Pv4) memw(#u6)=Rt32*/
  2339:   0xffe02084,
  2340:   0xaf800084,
  2341:   0x00030078,
  2342:   0 },
  2343: { /*Tag:S4_pstorerifnew_abs*/
  2344:   /*if (!Pv4.new) memw(#u6)=Rt32*/
  2345:   0xffe02084,
  2346:   0xaf802084,
  2347:   0x00030078,
  2348:   0 },
  2349: { /*Tag:S4_pstorerifnew_io*/
  2350:   /*if (!Pv4.new) memw(Rs32+#u6:2)=Rt32*/
  2351:   0xffe00004,
  2352:   0x46800000,
  2353:   0x000020f8,
  2354:   0 },
  2355: { /*Tag:S4_pstorerinewf_abs*/
  2356:   /*if (!Pv4) memw(#u6)=Nt8.new*/
  2357:   0xffe03884,
  2358:   0xafa01084,
  2359:   0x00030078,
  2360:   0 },
  2361: { /*Tag:S4_pstorerinewfnew_abs*/
  2362:   /*if (!Pv4.new) memw(#u6)=Nt8.new*/
  2363:   0xffe03884,
  2364:   0xafa03084,
  2365:   0x00030078,
  2366:   0 },
  2367: { /*Tag:S4_pstorerinewfnew_io*/
  2368:   /*if (!Pv4.new) memw(Rs32+#u6:2)=Nt8.new*/
  2369:   0xffe01804,
  2370:   0x46a01000,
  2371:   0x000020f8,
  2372:   0 },
  2373: { /*Tag:S4_pstorerinewt_abs*/
  2374:   /*if (Pv4) memw(#u6)=Nt8.new*/
  2375:   0xffe03884,
  2376:   0xafa01080,
  2377:   0x00030078,
  2378:   0 },
  2379: { /*Tag:S4_pstorerinewtnew_abs*/
  2380:   /*if (Pv4.new) memw(#u6)=Nt8.new*/
  2381:   0xffe03884,
  2382:   0xafa03080,
  2383:   0x00030078,
  2384:   0 },
  2385: { /*Tag:S4_pstorerinewtnew_io*/
  2386:   /*if (Pv4.new) memw(Rs32+#u6:2)=Nt8.new*/
  2387:   0xffe01804,
  2388:   0x42a01000,
  2389:   0x000020f8,
  2390:   0 },
  2391: { /*Tag:S4_pstorerit_abs*/
  2392:   /*if (Pv4) memw(#u6)=Rt32*/
  2393:   0xffe02084,
  2394:   0xaf800080,
  2395:   0x00030078,
  2396:   0 },
  2397: { /*Tag:S4_pstoreritnew_abs*/
  2398:   /*if (Pv4.new) memw(#u6)=Rt32*/
  2399:   0xffe02084,
  2400:   0xaf802080,
  2401:   0x00030078,
  2402:   0 },
  2403: { /*Tag:S4_pstoreritnew_io*/
  2404:   /*if (Pv4.new) memw(Rs32+#u6:2)=Rt32*/
  2405:   0xffe00004,
  2406:   0x42800000,
  2407:   0x000020f8,
  2408:   0 },
  2409: { /*Tag:S4_storeirb_io*/
  2410:   /*memb(Rs32+#u6:0)=#S8*/
  2411:   0xfe600000,
  2412:   0x3c000000,
  2413:   0x0000207f,
  2414:   0 },
  2415: { /*Tag:S4_storeirbf_io*/
  2416:   /*if (!Pv4) memb(Rs32+#u6:0)=#S6*/
  2417:   0xffe00000,
  2418:   0x38800000,
  2419:   0x0000201f,
  2420:   0 },
  2421: { /*Tag:S4_storeirbfnew_io*/
  2422:   /*if (!Pv4.new) memb(Rs32+#u6:0)=#S6*/
  2423:   0xffe00000,
  2424:   0x39800000,
  2425:   0x0000201f,
  2426:   0 },
  2427: { /*Tag:S4_storeirbt_io*/
  2428:   /*if (Pv4) memb(Rs32+#u6:0)=#S6*/
  2429:   0xffe00000,
  2430:   0x38000000,
  2431:   0x0000201f,
  2432:   0 },
  2433: { /*Tag:S4_storeirbtnew_io*/
  2434:   /*if (Pv4.new) memb(Rs32+#u6:0)=#S6*/
  2435:   0xffe00000,
  2436:   0x39000000,
  2437:   0x0000201f,
  2438:   0 },
  2439: { /*Tag:S4_storeirh_io*/
  2440:   /*memh(Rs32+#u6:1)=#S8*/
  2441:   0xfe600000,
  2442:   0x3c200000,
  2443:   0x0000207f,
  2444:   0 },
  2445: { /*Tag:S4_storeirhf_io*/
  2446:   /*if (!Pv4) memh(Rs32+#u6:1)=#S6*/
  2447:   0xffe00000,
  2448:   0x38a00000,
  2449:   0x0000201f,
  2450:   0 },
  2451: { /*Tag:S4_storeirhfnew_io*/
  2452:   /*if (!Pv4.new) memh(Rs32+#u6:1)=#S6*/
  2453:   0xffe00000,
  2454:   0x39a00000,
  2455:   0x0000201f,
  2456:   0 },
  2457: { /*Tag:S4_storeirht_io*/
  2458:   /*if (Pv4) memh(Rs32+#u6:1)=#S6*/
  2459:   0xffe00000,
  2460:   0x38200000,
  2461:   0x0000201f,
  2462:   0 },
  2463: { /*Tag:S4_storeirhtnew_io*/
  2464:   /*if (Pv4.new) memh(Rs32+#u6:1)=#S6*/
  2465:   0xffe00000,
  2466:   0x39200000,
  2467:   0x0000201f,
  2468:   0 },
  2469: { /*Tag:S4_storeiri_io*/
  2470:   /*memw(Rs32+#u6:2)=#S8*/
  2471:   0xfe600000,
  2472:   0x3c400000,
  2473:   0x0000207f,
  2474:   0 },
  2475: { /*Tag:S4_storeirif_io*/
  2476:   /*if (!Pv4) memw(Rs32+#u6:2)=#S6*/
  2477:   0xffe00000,
  2478:   0x38c00000,
  2479:   0x0000201f,
  2480:   0 },
  2481: { /*Tag:S4_storeirifnew_io*/
  2482:   /*if (!Pv4.new) memw(Rs32+#u6:2)=#S6*/
  2483:   0xffe00000,
  2484:   0x39c00000,
  2485:   0x0000201f,
  2486:   0 },
  2487: { /*Tag:S4_storeirit_io*/
  2488:   /*if (Pv4) memw(Rs32+#u6:2)=#S6*/
  2489:   0xffe00000,
  2490:   0x38400000,
  2491:   0x0000201f,
  2492:   0 },
  2493: { /*Tag:S4_storeiritnew_io*/
  2494:   /*if (Pv4.new) memw(Rs32+#u6:2)=#S6*/
  2495:   0xffe00000,
  2496:   0x39400000,
  2497:   0x0000201f,
  2498:   0 },
  2499: { /*Tag:S4_storerb_ap*/
  2500:   /*memb(Re32=#U6)=Rt32*/
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2501-2750 / 第 2501-2750 行

```cpp
  2501:   0xffe02080,
  2502:   0xab000080,
  2503:   0x0000003f,
  2504:   0 },
  2505: { /*Tag:S4_storerb_ur*/
  2506:   /*memb(Ru32<<#u2+#U6)=Rt32*/
  2507:   0xffe00080,
  2508:   0xad000080,
  2509:   0x0000003f,
  2510:   0 },
  2511: { /*Tag:S4_storerbnew_ap*/
  2512:   /*memb(Re32=#U6)=Nt8.new*/
  2513:   0xffe03880,
  2514:   0xaba00080,
  2515:   0x0000003f,
  2516:   0 },
  2517: { /*Tag:S4_storerbnew_ur*/
  2518:   /*memb(Ru32<<#u2+#U6)=Nt8.new*/
  2519:   0xffe01880,
  2520:   0xada00080,
  2521:   0x0000003f,
  2522:   0 },
  2523: { /*Tag:S4_storerd_ap*/
  2524:   /*memd(Re32=#U6)=Rtt32*/
  2525:   0xffe02080,
  2526:   0xabc00080,
  2527:   0x0000003f,
  2528:   0 },
  2529: { /*Tag:S4_storerd_ur*/
  2530:   /*memd(Ru32<<#u2+#U6)=Rtt32*/
  2531:   0xffe00080,
  2532:   0xadc00080,
  2533:   0x0000003f,
  2534:   0 },
  2535: { /*Tag:S4_storerf_ap*/
  2536:   /*memh(Re32=#U6)=Rt32.h*/
  2537:   0xffe02080,
  2538:   0xab600080,
  2539:   0x0000003f,
  2540:   0 },
  2541: { /*Tag:S4_storerf_ur*/
  2542:   /*memh(Ru32<<#u2+#U6)=Rt32.h*/
  2543:   0xffe00080,
  2544:   0xad600080,
  2545:   0x0000003f,
  2546:   0 },
  2547: { /*Tag:S4_storerh_ap*/
  2548:   /*memh(Re32=#U6)=Rt32*/
  2549:   0xffe02080,
  2550:   0xab400080,
  2551:   0x0000003f,
  2552:   0 },
  2553: { /*Tag:S4_storerh_ur*/
  2554:   /*memh(Ru32<<#u2+#U6)=Rt32*/
  2555:   0xffe00080,
  2556:   0xad400080,
  2557:   0x0000003f,
  2558:   0 },
  2559: { /*Tag:S4_storerhnew_ap*/
  2560:   /*memh(Re32=#U6)=Nt8.new*/
  2561:   0xffe03880,
  2562:   0xaba00880,
  2563:   0x0000003f,
  2564:   0 },
  2565: { /*Tag:S4_storerhnew_ur*/
  2566:   /*memh(Ru32<<#u2+#U6)=Nt8.new*/
  2567:   0xffe01880,
  2568:   0xada00880,
  2569:   0x0000003f,
  2570:   0 },
  2571: { /*Tag:S4_storeri_ap*/
  2572:   /*memw(Re32=#U6)=Rt32*/
  2573:   0xffe02080,
  2574:   0xab800080,
  2575:   0x0000003f,
  2576:   0 },
  2577: { /*Tag:S4_storeri_ur*/
  2578:   /*memw(Ru32<<#u2+#U6)=Rt32*/
  2579:   0xffe00080,
  2580:   0xad800080,
  2581:   0x0000003f,
  2582:   0 },
  2583: { /*Tag:S4_storerinew_ap*/
  2584:   /*memw(Re32=#U6)=Nt8.new*/
  2585:   0xffe03880,
  2586:   0xaba01080,
  2587:   0x0000003f,
  2588:   0 },
  2589: { /*Tag:S4_storerinew_ur*/
  2590:   /*memw(Ru32<<#u2+#U6)=Nt8.new*/
  2591:   0xffe01880,
  2592:   0xada01080,
  2593:   0x0000003f,
  2594:   0 },
  2595: { /*Tag:S4_subaddi*/
  2596:   /*Rd32=add(Rs32,sub(#s6,Ru32))*/
  2597:   0xff800000,
  2598:   0xdb800000,
  2599:   0x006020e0,
  2600:   0 },
  2601: { /*Tag:S4_subi_asl_ri*/
  2602:   /*Rx32=sub(#u8,asl(Rx32,#U5))*/
  2603:   0xff000016,
  2604:   0xde000006,
  2605:   0x00e020e8,
  2606:   0 },
  2607: { /*Tag:S4_subi_lsr_ri*/
  2608:   /*Rx32=sub(#u8,lsr(Rx32,#U5))*/
  2609:   0xff000016,
  2610:   0xde000016,
  2611:   0x00e020e8,
  2612:   0 },
  2613: { /*Tag:SA1_addi*/
  2614:   /*Rx16=add(Rx16,#s7)*/
  2615:   0xf8002000,
  2616:   0x20002000,
  2617:   0x07f00000,
  2618:   1 },
  2619: { /*Tag:SA1_addi*/
  2620:   /*Rx16=add(Rx16,#s7)*/
  2621:   0xf8002000,
  2622:   0x40000000,
  2623:   0x07f00000,
  2624:   1 },
  2625: { /*Tag:SA1_addi*/
  2626:   /*Rx16=add(Rx16,#s7)*/
  2627:   0xf8002000,
  2628:   0x40002000,
  2629:   0x07f00000,
  2630:   1 },
  2631: { /*Tag:SA1_addi*/
  2632:   /*Rx16=add(Rx16,#s7)*/
  2633:   0xf8002000,
  2634:   0x60000000,
  2635:   0x07f00000,
  2636:   1 },
  2637: { /*Tag:SA1_addi*/
  2638:   /*Rx16=add(Rx16,#s7)*/
  2639:   0xf8002000,
  2640:   0x60002000,
  2641:   0x07f00000,
  2642:   1 },
  2643: { /*Tag:SA1_seti*/
  2644:   /*Rd16=#u6*/
  2645:   0xfc002000,
  2646:   0x28002000,
  2647:   0x03f00000,
  2648:   1 },
  2649: { /*Tag:SA1_seti*/
  2650:   /*Rd16=#u6*/
  2651:   0xfc002000,
  2652:   0x48000000,
  2653:   0x03f00000,
  2654:   1 },
  2655: { /*Tag:SA1_seti*/
  2656:   /*Rd16=#u6*/
  2657:   0xfc002000,
  2658:   0x48002000,
  2659:   0x03f00000,
  2660:   1 },
  2661: { /*Tag:SA1_seti*/
  2662:   /*Rd16=#u6*/
  2663:   0xfc002000,
  2664:   0x68000000,
  2665:   0x03f00000,
  2666:   1 },
  2667: { /*Tag:SA1_seti*/
  2668:   /*Rd16=#u6*/
  2669:   0xfc002000,
  2670:   0x68002000,
  2671:   0x03f00000,
  2672:   1 },
  2673: { /*Tag:dup_A2_addi*/
  2674:   /*Rd32=add(Rs32,#s16)*/
  2675:   0x00000000,
  2676:   0x00000000,
  2677:   0x00000000,
  2678:   0 },
  2679: { /*Tag:dup_A2_andir*/
  2680:   /*Rd32=and(Rs32,#s10)*/
  2681:   0x00000000,
  2682:   0x00000000,
  2683:   0x00000000,
  2684:   0 },
  2685: { /*Tag:dup_A2_combineii*/
  2686:   /*Rdd32=combine(#s8,#S8)*/
  2687:   0x00000000,
  2688:   0x00000000,
  2689:   0x00000000,
  2690:   0 },
  2691: { /*Tag:dup_A2_tfrsi*/
  2692:   /*Rd32=#s16*/
  2693:   0x00000000,
  2694:   0x00000000,
  2695:   0x00000000,
  2696:   0 },
  2697: { /*Tag:dup_A4_combineii*/
  2698:   /*Rdd32=combine(#s8,#U6)*/
  2699:   0x00000000,
  2700:   0x00000000,
  2701:   0x00002404,
  2702:   0 },
  2703: { /*Tag:dup_A4_combineir*/
  2704:   /*Rdd32=combine(#s8,Rs32)*/
  2705:   0x00000000,
  2706:   0x00000000,
  2707:   0x00000000,
  2708:   0 },
  2709: { /*Tag:dup_A4_combineri*/
  2710:   /*Rdd32=combine(Rs32,#s8)*/
  2711:   0x00000000,
  2712:   0x00000000,
  2713:   0x00000000,
  2714:   0 },
  2715: { /*Tag:dup_C2_cmoveif*/
  2716:   /*if (!Pu4) Rd32=#s12*/
  2717:   0x00000000,
  2718:   0x00000000,
  2719:   0x00000000,
  2720:   0 },
  2721: { /*Tag:dup_C2_cmoveit*/
  2722:   /*if (Pu4) Rd32=#s12*/
  2723:   0x00000000,
  2724:   0x00000000,
  2725:   0x00000000,
  2726:   0 },
  2727: { /*Tag:dup_C2_cmovenewif*/
  2728:   /*if (!Pu4.new) Rd32=#s12*/
  2729:   0x00000000,
  2730:   0x00000000,
  2731:   0x00000000,
  2732:   0 },
  2733: { /*Tag:dup_C2_cmovenewit*/
  2734:   /*if (Pu4.new) Rd32=#s12*/
  2735:   0x00000000,
  2736:   0x00000000,
  2737:   0x00000000,
  2738:   0 },
  2739: { /*Tag:dup_C2_cmpeqi*/
  2740:   /*Pd4=cmp.eq(Rs32,#s10)*/
  2741:   0x00000000,
  2742:   0x00000000,
  2743:   0x00000000,
  2744:   0 },
  2745: { /*Tag:dup_L2_loadrb_io*/
  2746:   /*Rd32=memb(Rs32+#s11:0)*/
  2747:   0x00000000,
  2748:   0x00000000,
  2749:   0x00000000,
  2750:   0 },
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2751-2819 / 第 2751-2819 行

```cpp
  2751: { /*Tag:dup_L2_loadrd_io*/
  2752:   /*Rdd32=memd(Rs32+#s11:3)*/
  2753:   0x00000000,
  2754:   0x00000000,
  2755:   0x00000000,
  2756:   0 },
  2757: { /*Tag:dup_L2_loadrh_io*/
  2758:   /*Rd32=memh(Rs32+#s11:1)*/
  2759:   0x00000000,
  2760:   0x00000000,
  2761:   0x00000000,
  2762:   0 },
  2763: { /*Tag:dup_L2_loadri_io*/
  2764:   /*Rd32=memw(Rs32+#s11:2)*/
  2765:   0x00000000,
  2766:   0x00000000,
  2767:   0x00000000,
  2768:   0 },
  2769: { /*Tag:dup_L2_loadrub_io*/
  2770:   /*Rd32=memub(Rs32+#s11:0)*/
  2771:   0x00000000,
  2772:   0x00000000,
  2773:   0x00000000,
  2774:   0 },
  2775: { /*Tag:dup_L2_loadruh_io*/
  2776:   /*Rd32=memuh(Rs32+#s11:1)*/
  2777:   0x00000000,
  2778:   0x00000000,
  2779:   0x00000000,
  2780:   0 },
  2781: { /*Tag:dup_S2_storerb_io*/
  2782:   /*memb(Rs32+#s11:0)=Rt32*/
  2783:   0x00000000,
  2784:   0x00000000,
  2785:   0x00000000,
  2786:   0 },
  2787: { /*Tag:dup_S2_storerd_io*/
  2788:   /*memd(Rs32+#s11:3)=Rtt32*/
  2789:   0x00000000,
  2790:   0x00000000,
  2791:   0x00000000,
  2792:   0 },
  2793: { /*Tag:dup_S2_storerh_io*/
  2794:   /*memh(Rs32+#s11:1)=Rt32*/
  2795:   0x00000000,
  2796:   0x00000000,
  2797:   0x00000000,
  2798:   0 },
  2799: { /*Tag:dup_S2_storeri_io*/
  2800:   /*memw(Rs32+#s11:2)=Rt32*/
  2801:   0x00000000,
  2802:   0x00000000,
  2803:   0x00000000,
  2804:   0 },
  2805: { /*Tag:dup_S4_storeirb_io*/
  2806:   /*memb(Rs32+#u6:0)=#S8*/
  2807:   0x00000000,
  2808:   0x00000000,
  2809:   0x00002404,
  2810:   0 },
  2811: { /*Tag:dup_S4_storeiri_io*/
  2812:   /*memw(Rs32+#u6:2)=#S8*/
  2813:   0x00000000,
  2814:   0x00000000,
  2815:   0x00002404,
  2816:   0 }
  2817: };
  2818: 
  2819: #endif  // LLVM_LIB_TARGET_HEXAGON_HEXAGONDEPMASK_H
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- generated target metadata / 生成的目标元数据

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonInstruction`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
