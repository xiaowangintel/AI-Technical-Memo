# HexagonDepInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonDepInstrFormats.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon instruction format records using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。 重点涉及指令语义与选择。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。
- Generation status / 生成状态: The file appears to be auto-generated or largely generated from upstream target data. / 该文件看起来是自动生成的，或主要由上游目标数据生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-250 / 第 1-250 行

```tablegen
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
    11: class Enc_01d3d0 : OpcodeHexagon {
    12:   bits <5> Vu32;
    13:   let Inst{12-8} = Vu32{4-0};
    14:   bits <5> Rt32;
    15:   let Inst{20-16} = Rt32{4-0};
    16:   bits <5> Vdd32;
    17:   let Inst{4-0} = Vdd32{4-0};
    18: }
    19: class Enc_02553a : OpcodeHexagon {
    20:   bits <7> Ii;
    21:   let Inst{11-5} = Ii{6-0};
    22:   bits <5> Rs32;
    23:   let Inst{20-16} = Rs32{4-0};
    24:   bits <2> Pd4;
    25:   let Inst{1-0} = Pd4{1-0};
    26: }
    27: class Enc_03833b : OpcodeHexagon {
    28:   bits <5> Rss32;
    29:   let Inst{20-16} = Rss32{4-0};
    30:   bits <5> Rt32;
    31:   let Inst{12-8} = Rt32{4-0};
    32:   bits <2> Pd4;
    33:   let Inst{1-0} = Pd4{1-0};
    34: }
    35: class Enc_041d7b : OpcodeHexagon {
    36:   bits <11> Ii;
    37:   let Inst{21-20} = Ii{10-9};
    38:   let Inst{7-1} = Ii{8-2};
    39:   bits <4> Rs16;
    40:   let Inst{19-16} = Rs16{3-0};
    41:   bits <0> n1;
    42: }
    43: class Enc_046afa : OpcodeHexagon {
    44:   bits <1> Mu2;
    45:   let Inst{13-13} = Mu2{0-0};
    46:   bits <5> Vss32;
    47:   let Inst{4-0} = Vss32{4-0};
    48:   bits <5> Rx32;
    49:   let Inst{20-16} = Rx32{4-0};
    50: }
    51: class Enc_04c959 : OpcodeHexagon {
    52:   bits <2> Ii;
    53:   let Inst{13-13} = Ii{1-1};
    54:   let Inst{7-7} = Ii{0-0};
    55:   bits <6> II;
    56:   let Inst{11-8} = II{5-2};
    57:   let Inst{6-5} = II{1-0};
    58:   bits <5> Rt32;
    59:   let Inst{20-16} = Rt32{4-0};
    60:   bits <5> Ryy32;
    61:   let Inst{4-0} = Ryy32{4-0};
    62: }
    63: class Enc_0527db : OpcodeHexagon {
    64:   bits <4> Rs16;
    65:   let Inst{7-4} = Rs16{3-0};
    66:   bits <4> Rx16;
    67:   let Inst{3-0} = Rx16{3-0};
    68: }
    69: class Enc_052c7d : OpcodeHexagon {
    70:   bits <5> Ii;
    71:   let Inst{6-3} = Ii{4-1};
    72:   bits <5> Rt32;
    73:   let Inst{12-8} = Rt32{4-0};
    74:   bits <5> Rx32;
    75:   let Inst{20-16} = Rx32{4-0};
    76: }
    77: class Enc_08d755 : OpcodeHexagon {
    78:   bits <8> Ii;
    79:   let Inst{12-5} = Ii{7-0};
    80:   bits <5> Rs32;
    81:   let Inst{20-16} = Rs32{4-0};
    82:   bits <2> Pd4;
    83:   let Inst{1-0} = Pd4{1-0};
    84: }
    85: class Enc_0aa344 : OpcodeHexagon {
    86:   bits <5> Gss32;
    87:   let Inst{20-16} = Gss32{4-0};
    88:   bits <5> Rdd32;
    89:   let Inst{4-0} = Rdd32{4-0};
    90: }
    91: class Enc_0b2e5b : OpcodeHexagon {
    92:   bits <3> Ii;
    93:   let Inst{7-5} = Ii{2-0};
    94:   bits <5> Vu32;
    95:   let Inst{12-8} = Vu32{4-0};
    96:   bits <5> Vv32;
    97:   let Inst{20-16} = Vv32{4-0};
    98:   bits <5> Vd32;
    99:   let Inst{4-0} = Vd32{4-0};
   100: }
   101: class Enc_0b51ce : OpcodeHexagon {
   102:   bits <3> Ii;
   103:   let Inst{10-8} = Ii{2-0};
   104:   bits <2> Qv4;
   105:   let Inst{12-11} = Qv4{1-0};
   106:   bits <5> Vs32;
   107:   let Inst{4-0} = Vs32{4-0};
   108:   bits <5> Rx32;
   109:   let Inst{20-16} = Rx32{4-0};
   110: }
   111: class Enc_0cb018 : OpcodeHexagon {
   112:   bits <5> Cs32;
   113:   let Inst{20-16} = Cs32{4-0};
   114:   bits <5> Rd32;
   115:   let Inst{4-0} = Rd32{4-0};
   116: }
   117: class Enc_0d8870 : OpcodeHexagon {
   118:   bits <12> Ii;
   119:   let Inst{26-25} = Ii{11-10};
   120:   let Inst{13-13} = Ii{9-9};
   121:   let Inst{7-0} = Ii{8-1};
   122:   bits <5> Rs32;
   123:   let Inst{20-16} = Rs32{4-0};
   124:   bits <3> Nt8;
   125:   let Inst{10-8} = Nt8{2-0};
   126: }
   127: class Enc_0d8adb : OpcodeHexagon {
   128:   bits <8> Ii;
   129:   let Inst{12-5} = Ii{7-0};
   130:   bits <5> Rss32;
   131:   let Inst{20-16} = Rss32{4-0};
   132:   bits <2> Pd4;
   133:   let Inst{1-0} = Pd4{1-0};
   134: }
   135: class Enc_0e41fa : OpcodeHexagon {
   136:   bits <5> Vuu32;
   137:   let Inst{12-8} = Vuu32{4-0};
   138:   bits <5> Rt32;
   139:   let Inst{20-16} = Rt32{4-0};
   140:   bits <5> Vd32;
   141:   let Inst{4-0} = Vd32{4-0};
   142: }
   143: class Enc_0ed752 : OpcodeHexagon {
   144:   bits <5> Rss32;
   145:   let Inst{20-16} = Rss32{4-0};
   146:   bits <5> Cdd32;
   147:   let Inst{4-0} = Cdd32{4-0};
   148: }
   149: class Enc_0f8bab : OpcodeHexagon {
   150:   bits <5> Vu32;
   151:   let Inst{12-8} = Vu32{4-0};
   152:   bits <5> Rt32;
   153:   let Inst{20-16} = Rt32{4-0};
   154:   bits <2> Qd4;
   155:   let Inst{1-0} = Qd4{1-0};
   156: }
   157: class Enc_0fa531 : OpcodeHexagon {
   158:   bits <15> Ii;
   159:   let Inst{21-21} = Ii{14-14};
   160:   let Inst{13-13} = Ii{13-13};
   161:   let Inst{11-1} = Ii{12-2};
   162:   bits <5> Rs32;
   163:   let Inst{20-16} = Rs32{4-0};
   164: }
   165: class Enc_10bc21 : OpcodeHexagon {
   166:   bits <4> Ii;
   167:   let Inst{6-3} = Ii{3-0};
   168:   bits <5> Rt32;
   169:   let Inst{12-8} = Rt32{4-0};
   170:   bits <5> Rx32;
   171:   let Inst{20-16} = Rx32{4-0};
   172: }
   173: class Enc_1178da : OpcodeHexagon {
   174:   bits <3> Ii;
   175:   let Inst{7-5} = Ii{2-0};
   176:   bits <5> Vu32;
   177:   let Inst{12-8} = Vu32{4-0};
   178:   bits <5> Vv32;
   179:   let Inst{20-16} = Vv32{4-0};
   180:   bits <5> Vxx32;
   181:   let Inst{4-0} = Vxx32{4-0};
   182: }
   183: class Enc_11a146 : OpcodeHexagon {
   184:   bits <4> Ii;
   185:   let Inst{11-8} = Ii{3-0};
   186:   bits <5> Rss32;
   187:   let Inst{20-16} = Rss32{4-0};
   188:   bits <5> Rd32;
   189:   let Inst{4-0} = Rd32{4-0};
   190: }
   191: class Enc_12b6e9 : OpcodeHexagon {
   192:   bits <4> Ii;
   193:   let Inst{11-8} = Ii{3-0};
   194:   bits <5> Rss32;
   195:   let Inst{20-16} = Rss32{4-0};
   196:   bits <5> Rdd32;
   197:   let Inst{4-0} = Rdd32{4-0};
   198: }
   199: class Enc_134437 : OpcodeHexagon {
   200:   bits <2> Qs4;
   201:   let Inst{9-8} = Qs4{1-0};
   202:   bits <2> Qt4;
   203:   let Inst{23-22} = Qt4{1-0};
   204:   bits <2> Qd4;
   205:   let Inst{1-0} = Qd4{1-0};
   206: }
   207: class Enc_140c83 : OpcodeHexagon {
   208:   bits <10> Ii;
   209:   let Inst{21-21} = Ii{9-9};
   210:   let Inst{13-5} = Ii{8-0};
   211:   bits <5> Rs32;
   212:   let Inst{20-16} = Rs32{4-0};
   213:   bits <5> Rd32;
   214:   let Inst{4-0} = Rd32{4-0};
   215: }
   216: class Enc_143445 : OpcodeHexagon {
   217:   bits <13> Ii;
   218:   let Inst{26-25} = Ii{12-11};
   219:   let Inst{13-13} = Ii{10-10};
   220:   let Inst{7-0} = Ii{9-2};
   221:   bits <5> Rs32;
   222:   let Inst{20-16} = Rs32{4-0};
   223:   bits <5> Rt32;
   224:   let Inst{12-8} = Rt32{4-0};
   225: }
   226: class Enc_143a3c : OpcodeHexagon {
   227:   bits <6> Ii;
   228:   let Inst{13-8} = Ii{5-0};
   229:   bits <6> II;
   230:   let Inst{23-21} = II{5-3};
   231:   let Inst{7-5} = II{2-0};
   232:   bits <5> Rss32;
   233:   let Inst{20-16} = Rss32{4-0};
   234:   bits <5> Rxx32;
   235:   let Inst{4-0} = Rxx32{4-0};
   236: }
   237: class Enc_14640c : OpcodeHexagon {
   238:   bits <11> Ii;
   239:   let Inst{21-20} = Ii{10-9};
   240:   let Inst{7-1} = Ii{8-2};
   241:   bits <4> Rs16;
   242:   let Inst{19-16} = Rs16{3-0};
   243:   bits <0> n1;
   244: }
   245: class Enc_14d27a : OpcodeHexagon {
   246:   bits <5> II;
   247:   let Inst{12-8} = II{4-0};
   248:   bits <11> Ii;
   249:   let Inst{21-20} = Ii{10-9};
   250:   let Inst{7-1} = Ii{8-2};
```
- EN: It declares types such as Enc_01d3d0, Enc_02553a, Enc_03833b, Enc_041d7b, ... (29 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_01d3d0, Enc_02553a, Enc_03833b, Enc_041d7b, Enc_046afa, ... (29 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 Enc_01d3d0, Enc_02553a, Enc_03833b, Enc_041d7b, ... (29 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_01d3d0, Enc_02553a, Enc_03833b, Enc_041d7b, Enc_046afa, ... (29 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-500 / 第 251-500 行

```tablegen
   251:   bits <4> Rs16;
   252:   let Inst{19-16} = Rs16{3-0};
   253: }
   254: class Enc_152467 : OpcodeHexagon {
   255:   bits <5> Ii;
   256:   let Inst{8-5} = Ii{4-1};
   257:   bits <5> Rd32;
   258:   let Inst{4-0} = Rd32{4-0};
   259:   bits <5> Rx32;
   260:   let Inst{20-16} = Rx32{4-0};
   261: }
   262: class Enc_158beb : OpcodeHexagon {
   263:   bits <2> Qs4;
   264:   let Inst{6-5} = Qs4{1-0};
   265:   bits <5> Rt32;
   266:   let Inst{20-16} = Rt32{4-0};
   267:   bits <1> Mu2;
   268:   let Inst{13-13} = Mu2{0-0};
   269:   bits <5> Vv32;
   270:   let Inst{4-0} = Vv32{4-0};
   271: }
   272: class Enc_163a3c : OpcodeHexagon {
   273:   bits <7> Ii;
   274:   let Inst{12-7} = Ii{6-1};
   275:   bits <5> Rs32;
   276:   let Inst{20-16} = Rs32{4-0};
   277:   bits <5> Rt32;
   278:   let Inst{4-0} = Rt32{4-0};
   279: }
   280: class Enc_16c48b : OpcodeHexagon {
   281:   bits <5> Rt32;
   282:   let Inst{20-16} = Rt32{4-0};
   283:   bits <1> Mu2;
   284:   let Inst{13-13} = Mu2{0-0};
   285:   bits <5> Vv32;
   286:   let Inst{12-8} = Vv32{4-0};
   287:   bits <5> Vw32;
   288:   let Inst{4-0} = Vw32{4-0};
   289: }
   290: class Enc_178717 : OpcodeHexagon {
   291:   bits <11> Ii;
   292:   let Inst{21-20} = Ii{10-9};
   293:   let Inst{7-1} = Ii{8-2};
   294:   bits <4> Rs16;
   295:   let Inst{19-16} = Rs16{3-0};
   296:   bits <0> n1;
   297: }
   298: class Enc_179b35 : OpcodeHexagon {
   299:   bits <5> Rs32;
   300:   let Inst{20-16} = Rs32{4-0};
   301:   bits <5> Rtt32;
   302:   let Inst{12-8} = Rtt32{4-0};
   303:   bits <5> Rx32;
   304:   let Inst{4-0} = Rx32{4-0};
   305: }
   306: class Enc_18c338 : OpcodeHexagon {
   307:   bits <8> Ii;
   308:   let Inst{12-5} = Ii{7-0};
   309:   bits <8> II;
   310:   let Inst{22-16} = II{7-1};
   311:   let Inst{13-13} = II{0-0};
   312:   bits <5> Rdd32;
   313:   let Inst{4-0} = Rdd32{4-0};
   314: }
   315: class Enc_1a9974 : OpcodeHexagon {
   316:   bits <2> Ii;
   317:   let Inst{13-13} = Ii{1-1};
   318:   let Inst{7-7} = Ii{0-0};
   319:   bits <2> Pv4;
   320:   let Inst{6-5} = Pv4{1-0};
   321:   bits <5> Rs32;
   322:   let Inst{20-16} = Rs32{4-0};
   323:   bits <5> Ru32;
   324:   let Inst{12-8} = Ru32{4-0};
   325:   bits <5> Rtt32;
   326:   let Inst{4-0} = Rtt32{4-0};
   327: }
   328: class Enc_1aa186 : OpcodeHexagon {
   329:   bits <5> Rss32;
   330:   let Inst{20-16} = Rss32{4-0};
   331:   bits <5> Rt32;
   332:   let Inst{12-8} = Rt32{4-0};
   333:   bits <5> Rxx32;
   334:   let Inst{4-0} = Rxx32{4-0};
   335: }
   336: class Enc_1aaec1 : OpcodeHexagon {
   337:   bits <3> Ii;
   338:   let Inst{10-8} = Ii{2-0};
   339:   bits <3> Os8;
   340:   let Inst{2-0} = Os8{2-0};
   341:   bits <5> Rx32;
   342:   let Inst{20-16} = Rx32{4-0};
   343: }
   344: class Enc_1b64fb : OpcodeHexagon {
   345:   bits <16> Ii;
   346:   let Inst{26-25} = Ii{15-14};
   347:   let Inst{20-16} = Ii{13-9};
   348:   let Inst{13-13} = Ii{8-8};
   349:   let Inst{7-0} = Ii{7-0};
   350:   bits <5> Rt32;
   351:   let Inst{12-8} = Rt32{4-0};
   352: }
   353: class Enc_1bd127 : OpcodeHexagon {
   354:   bits <5> Vu32;
   355:   let Inst{12-8} = Vu32{4-0};
   356:   bits <3> Rt8;
   357:   let Inst{18-16} = Rt8{2-0};
   358:   bits <5> Vdddd32;
   359:   let Inst{4-0} = Vdddd32{4-0};
   360: }
   361: class Enc_1cf4ca : OpcodeHexagon {
   362:   bits <6> Ii;
   363:   let Inst{17-16} = Ii{5-4};
   364:   let Inst{6-3} = Ii{3-0};
   365:   bits <2> Pv4;
   366:   let Inst{1-0} = Pv4{1-0};
   367:   bits <5> Rt32;
   368:   let Inst{12-8} = Rt32{4-0};
   369: }
   370: class Enc_1de724 : OpcodeHexagon {
   371:   bits <11> Ii;
   372:   let Inst{21-20} = Ii{10-9};
   373:   let Inst{7-1} = Ii{8-2};
   374:   bits <4> Rs16;
   375:   let Inst{19-16} = Rs16{3-0};
   376:   bits <0> n1;
   377: }
   378: class Enc_1ef990 : OpcodeHexagon {
   379:   bits <2> Pv4;
   380:   let Inst{12-11} = Pv4{1-0};
   381:   bits <1> Mu2;
   382:   let Inst{13-13} = Mu2{0-0};
   383:   bits <5> Vs32;
   384:   let Inst{4-0} = Vs32{4-0};
   385:   bits <5> Rx32;
   386:   let Inst{20-16} = Rx32{4-0};
   387: }
   388: class Enc_1f19b5 : OpcodeHexagon {
   389:   bits <5> Ii;
   390:   let Inst{9-5} = Ii{4-0};
   391:   bits <5> Rss32;
   392:   let Inst{20-16} = Rss32{4-0};
   393:   bits <2> Pd4;
   394:   let Inst{1-0} = Pd4{1-0};
   395: }
   396: class Enc_1f5ba6 : OpcodeHexagon {
   397:   bits <4> Rd16;
   398:   let Inst{3-0} = Rd16{3-0};
   399: }
   400: class Enc_1f5d8f : OpcodeHexagon {
   401:   bits <1> Mu2;
   402:   let Inst{13-13} = Mu2{0-0};
   403:   bits <5> Ryy32;
   404:   let Inst{4-0} = Ryy32{4-0};
   405:   bits <5> Rx32;
   406:   let Inst{20-16} = Rx32{4-0};
   407: }
   408: class Enc_211aaa : OpcodeHexagon {
   409:   bits <11> Ii;
   410:   let Inst{26-25} = Ii{10-9};
   411:   let Inst{13-5} = Ii{8-0};
   412:   bits <5> Rs32;
   413:   let Inst{20-16} = Rs32{4-0};
   414:   bits <5> Rd32;
   415:   let Inst{4-0} = Rd32{4-0};
   416: }
   417: class Enc_217147 : OpcodeHexagon {
   418:   bits <2> Qv4;
   419:   let Inst{23-22} = Qv4{1-0};
   420: }
   421: class Enc_222336 : OpcodeHexagon {
   422:   bits <4> Ii;
   423:   let Inst{8-5} = Ii{3-0};
   424:   bits <5> Rd32;
   425:   let Inst{4-0} = Rd32{4-0};
   426:   bits <5> Rx32;
   427:   let Inst{20-16} = Rx32{4-0};
   428: }
   429: class Enc_223005 : OpcodeHexagon {
   430:   bits <6> Ii;
   431:   let Inst{6-3} = Ii{5-2};
   432:   bits <3> Nt8;
   433:   let Inst{10-8} = Nt8{2-0};
   434:   bits <5> Rx32;
   435:   let Inst{20-16} = Rx32{4-0};
   436: }
   437: class Enc_226535 : OpcodeHexagon {
   438:   bits <8> Ii;
   439:   let Inst{12-7} = Ii{7-2};
   440:   bits <5> Rs32;
   441:   let Inst{20-16} = Rs32{4-0};
   442:   bits <5> Rt32;
   443:   let Inst{4-0} = Rt32{4-0};
   444: }
   445: class Enc_22c845 : OpcodeHexagon {
   446:   bits <14> Ii;
   447:   let Inst{10-0} = Ii{13-3};
   448:   bits <5> Rx32;
   449:   let Inst{20-16} = Rx32{4-0};
   450: }
   451: class Enc_2301d6 : OpcodeHexagon {
   452:   bits <6> Ii;
   453:   let Inst{20-16} = Ii{5-1};
   454:   let Inst{8-8} = Ii{0-0};
   455:   bits <2> Pt4;
   456:   let Inst{10-9} = Pt4{1-0};
   457:   bits <5> Rd32;
   458:   let Inst{4-0} = Rd32{4-0};
   459: }
   460: class Enc_245865 : OpcodeHexagon {
   461:   bits <5> Vu32;
   462:   let Inst{12-8} = Vu32{4-0};
   463:   bits <5> Vv32;
   464:   let Inst{23-19} = Vv32{4-0};
   465:   bits <3> Rt8;
   466:   let Inst{18-16} = Rt8{2-0};
   467:   bits <5> Vx32;
   468:   let Inst{4-0} = Vx32{4-0};
   469: }
   470: class Enc_24a7dc : OpcodeHexagon {
   471:   bits <5> Vu32;
   472:   let Inst{12-8} = Vu32{4-0};
   473:   bits <5> Vv32;
   474:   let Inst{23-19} = Vv32{4-0};
   475:   bits <3> Rt8;
   476:   let Inst{18-16} = Rt8{2-0};
   477:   bits <5> Vdd32;
   478:   let Inst{4-0} = Vdd32{4-0};
   479: }
   480: class Enc_25bef0 : OpcodeHexagon {
   481:   bits <16> Ii;
   482:   let Inst{26-25} = Ii{15-14};
   483:   let Inst{20-16} = Ii{13-9};
   484:   let Inst{13-5} = Ii{8-0};
   485:   bits <5> Rd32;
   486:   let Inst{4-0} = Rd32{4-0};
   487: }
   488: class Enc_263841 : OpcodeHexagon {
   489:   bits <5> Vu32;
   490:   let Inst{12-8} = Vu32{4-0};
   491:   bits <5> Rtt32;
   492:   let Inst{20-16} = Rtt32{4-0};
   493:   bits <5> Vd32;
   494:   let Inst{4-0} = Vd32{4-0};
   495: }
   496: class Enc_277737 : OpcodeHexagon {
   497:   bits <8> Ii;
   498:   let Inst{22-21} = Ii{7-6};
   499:   let Inst{13-13} = Ii{5-5};
   500:   let Inst{7-5} = Ii{4-2};
```
- EN: It declares types such as Enc_152467, Enc_158beb, Enc_163a3c, Enc_16c48b, ... (30 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_152467, Enc_158beb, Enc_163a3c, Enc_16c48b, Enc_178717, ... (30 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_152467, Enc_158beb, Enc_163a3c, Enc_16c48b, ... (30 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_152467, Enc_158beb, Enc_163a3c, Enc_16c48b, Enc_178717, ... (30 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 501-750 / 第 501-750 行

```tablegen
   501:   bits <5> Ru32;
   502:   let Inst{4-0} = Ru32{4-0};
   503:   bits <5> Rs32;
   504:   let Inst{20-16} = Rs32{4-0};
   505:   bits <5> Rd32;
   506:   let Inst{12-8} = Rd32{4-0};
   507: }
   508: class Enc_27b757 : OpcodeHexagon {
   509:   bits <4> Ii;
   510:   let Inst{13-13} = Ii{3-3};
   511:   let Inst{10-8} = Ii{2-0};
   512:   bits <2> Pv4;
   513:   let Inst{12-11} = Pv4{1-0};
   514:   bits <5> Rt32;
   515:   let Inst{20-16} = Rt32{4-0};
   516:   bits <5> Vs32;
   517:   let Inst{4-0} = Vs32{4-0};
   518: }
   519: class Enc_27fd0e : OpcodeHexagon {
   520:   bits <6> Ii;
   521:   let Inst{8-5} = Ii{5-2};
   522:   bits <1> Mu2;
   523:   let Inst{13-13} = Mu2{0-0};
   524:   bits <5> Rd32;
   525:   let Inst{4-0} = Rd32{4-0};
   526:   bits <5> Rx32;
   527:   let Inst{20-16} = Rx32{4-0};
   528: }
   529: class Enc_284ebb : OpcodeHexagon {
   530:   bits <2> Ps4;
   531:   let Inst{17-16} = Ps4{1-0};
   532:   bits <2> Pt4;
   533:   let Inst{9-8} = Pt4{1-0};
   534:   bits <2> Pd4;
   535:   let Inst{1-0} = Pd4{1-0};
   536: }
   537: class Enc_28a2dc : OpcodeHexagon {
   538:   bits <5> Ii;
   539:   let Inst{12-8} = Ii{4-0};
   540:   bits <5> Rs32;
   541:   let Inst{20-16} = Rs32{4-0};
   542:   bits <5> Rx32;
   543:   let Inst{4-0} = Rx32{4-0};
   544: }
   545: class Enc_28dcbb : OpcodeHexagon {
   546:   bits <5> Rt32;
   547:   let Inst{20-16} = Rt32{4-0};
   548:   bits <1> Mu2;
   549:   let Inst{13-13} = Mu2{0-0};
   550:   bits <5> Vvv32;
   551:   let Inst{4-0} = Vvv32{4-0};
   552: }
   553: class Enc_2a3787 : OpcodeHexagon {
   554:   bits <13> Ii;
   555:   let Inst{26-25} = Ii{12-11};
   556:   let Inst{13-5} = Ii{10-2};
   557:   bits <5> Rs32;
   558:   let Inst{20-16} = Rs32{4-0};
   559:   bits <5> Rd32;
   560:   let Inst{4-0} = Rd32{4-0};
   561: }
   562: class Enc_2a7b91 : OpcodeHexagon {
   563:   bits <6> Ii;
   564:   let Inst{20-16} = Ii{5-1};
   565:   let Inst{8-8} = Ii{0-0};
   566:   bits <2> Pt4;
   567:   let Inst{10-9} = Pt4{1-0};
   568:   bits <5> Rdd32;
   569:   let Inst{4-0} = Rdd32{4-0};
   570: }
   571: class Enc_2ae154 : OpcodeHexagon {
   572:   bits <5> Rs32;
   573:   let Inst{20-16} = Rs32{4-0};
   574:   bits <5> Rt32;
   575:   let Inst{12-8} = Rt32{4-0};
   576:   bits <5> Rx32;
   577:   let Inst{4-0} = Rx32{4-0};
   578: }
   579: class Enc_2b3f60 : OpcodeHexagon {
   580:   bits <5> Rss32;
   581:   let Inst{20-16} = Rss32{4-0};
   582:   bits <5> Rtt32;
   583:   let Inst{12-8} = Rtt32{4-0};
   584:   bits <5> Rdd32;
   585:   let Inst{4-0} = Rdd32{4-0};
   586:   bits <2> Px4;
   587:   let Inst{6-5} = Px4{1-0};
   588: }
   589: class Enc_2b518f : OpcodeHexagon {
   590:   bits <32> Ii;
   591:   let Inst{27-16} = Ii{31-20};
   592:   let Inst{13-0} = Ii{19-6};
   593: }
   594: class Enc_2bae10 : OpcodeHexagon {
   595:   bits <4> Ii;
   596:   let Inst{10-8} = Ii{3-1};
   597:   bits <4> Rs16;
   598:   let Inst{7-4} = Rs16{3-0};
   599:   bits <4> Rd16;
   600:   let Inst{3-0} = Rd16{3-0};
   601: }
   602: class Enc_2d7491 : OpcodeHexagon {
   603:   bits <13> Ii;
   604:   let Inst{26-25} = Ii{12-11};
   605:   let Inst{13-5} = Ii{10-2};
   606:   bits <5> Rs32;
   607:   let Inst{20-16} = Rs32{4-0};
   608:   bits <5> Rdd32;
   609:   let Inst{4-0} = Rdd32{4-0};
   610: }
   611: class Enc_2d829e : OpcodeHexagon {
   612:   bits <14> Ii;
   613:   let Inst{10-0} = Ii{13-3};
   614:   bits <5> Rs32;
   615:   let Inst{20-16} = Rs32{4-0};
   616: }
   617: class Enc_2df31d : OpcodeHexagon {
   618:   bits <8> Ii;
   619:   let Inst{9-4} = Ii{7-2};
   620:   bits <4> Rd16;
   621:   let Inst{3-0} = Rd16{3-0};
   622: }
   623: class Enc_2e1979 : OpcodeHexagon {
   624:   bits <2> Ii;
   625:   let Inst{13-13} = Ii{1-1};
   626:   let Inst{7-7} = Ii{0-0};
   627:   bits <2> Pv4;
   628:   let Inst{6-5} = Pv4{1-0};
   629:   bits <5> Rs32;
   630:   let Inst{20-16} = Rs32{4-0};
   631:   bits <5> Rt32;
   632:   let Inst{12-8} = Rt32{4-0};
   633:   bits <5> Rd32;
   634:   let Inst{4-0} = Rd32{4-0};
   635: }
   636: class Enc_2ea740 : OpcodeHexagon {
   637:   bits <4> Ii;
   638:   let Inst{13-13} = Ii{3-3};
   639:   let Inst{10-8} = Ii{2-0};
   640:   bits <2> Qv4;
   641:   let Inst{12-11} = Qv4{1-0};
   642:   bits <5> Rt32;
   643:   let Inst{20-16} = Rt32{4-0};
   644:   bits <5> Vs32;
   645:   let Inst{4-0} = Vs32{4-0};
   646: }
   647: class Enc_2ebe3b : OpcodeHexagon {
   648:   bits <1> Mu2;
   649:   let Inst{13-13} = Mu2{0-0};
   650:   bits <5> Vd32;
   651:   let Inst{4-0} = Vd32{4-0};
   652:   bits <5> Rx32;
   653:   let Inst{20-16} = Rx32{4-0};
   654: }
   655: class Enc_2f2f04 : OpcodeHexagon {
   656:   bits <1> Ii;
   657:   let Inst{5-5} = Ii{0-0};
   658:   bits <5> Vuu32;
   659:   let Inst{12-8} = Vuu32{4-0};
   660:   bits <5> Rt32;
   661:   let Inst{20-16} = Rt32{4-0};
   662:   bits <5> Vdd32;
   663:   let Inst{4-0} = Vdd32{4-0};
   664: }
   665: class Enc_2fbf3c : OpcodeHexagon {
   666:   bits <3> Ii;
   667:   let Inst{10-8} = Ii{2-0};
   668:   bits <4> Rs16;
   669:   let Inst{7-4} = Rs16{3-0};
   670:   bits <4> Rd16;
   671:   let Inst{3-0} = Rd16{3-0};
   672: }
   673: class Enc_310ba1 : OpcodeHexagon {
   674:   bits <5> Vu32;
   675:   let Inst{12-8} = Vu32{4-0};
   676:   bits <5> Rtt32;
   677:   let Inst{20-16} = Rtt32{4-0};
   678:   bits <5> Vx32;
   679:   let Inst{4-0} = Vx32{4-0};
   680: }
   681: class Enc_311abd : OpcodeHexagon {
   682:   bits <5> Ii;
   683:   let Inst{12-8} = Ii{4-0};
   684:   bits <5> Rs32;
   685:   let Inst{20-16} = Rs32{4-0};
   686:   bits <5> Rdd32;
   687:   let Inst{4-0} = Rdd32{4-0};
   688: }
   689: class Enc_31aa6a : OpcodeHexagon {
   690:   bits <5> Ii;
   691:   let Inst{6-3} = Ii{4-1};
   692:   bits <2> Pv4;
   693:   let Inst{1-0} = Pv4{1-0};
   694:   bits <3> Nt8;
   695:   let Inst{10-8} = Nt8{2-0};
   696:   bits <5> Rx32;
   697:   let Inst{20-16} = Rx32{4-0};
   698: }
   699: class Enc_31db33 : OpcodeHexagon {
   700:   bits <2> Qt4;
   701:   let Inst{6-5} = Qt4{1-0};
   702:   bits <5> Vu32;
   703:   let Inst{12-8} = Vu32{4-0};
   704:   bits <5> Vv32;
   705:   let Inst{20-16} = Vv32{4-0};
   706:   bits <5> Vd32;
   707:   let Inst{4-0} = Vd32{4-0};
   708: }
   709: class Enc_322e1b : OpcodeHexagon {
   710:   bits <6> Ii;
   711:   let Inst{22-21} = Ii{5-4};
   712:   let Inst{13-13} = Ii{3-3};
   713:   let Inst{7-5} = Ii{2-0};
   714:   bits <6> II;
   715:   let Inst{23-23} = II{5-5};
   716:   let Inst{4-0} = II{4-0};
   717:   bits <5> Rs32;
   718:   let Inst{20-16} = Rs32{4-0};
   719:   bits <5> Rd32;
   720:   let Inst{12-8} = Rd32{4-0};
   721: }
   722: class Enc_323f2d : OpcodeHexagon {
   723:   bits <6> II;
   724:   let Inst{11-8} = II{5-2};
   725:   let Inst{6-5} = II{1-0};
   726:   bits <5> Rd32;
   727:   let Inst{4-0} = Rd32{4-0};
   728:   bits <5> Re32;
   729:   let Inst{20-16} = Re32{4-0};
   730: }
   731: class Enc_329361 : OpcodeHexagon {
   732:   bits <2> Pu4;
   733:   let Inst{6-5} = Pu4{1-0};
   734:   bits <5> Rss32;
   735:   let Inst{20-16} = Rss32{4-0};
   736:   bits <5> Rtt32;
   737:   let Inst{12-8} = Rtt32{4-0};
   738:   bits <5> Rdd32;
   739:   let Inst{4-0} = Rdd32{4-0};
   740: }
   741: class Enc_33f8ba : OpcodeHexagon {
   742:   bits <8> Ii;
   743:   let Inst{12-8} = Ii{7-3};
   744:   let Inst{4-2} = Ii{2-0};
   745:   bits <5> Rx32;
   746:   let Inst{20-16} = Rx32{4-0};
   747: }
   748: class Enc_3680c2 : OpcodeHexagon {
   749:   bits <7> Ii;
   750:   let Inst{11-5} = Ii{6-0};
```
- EN: It declares types such as Enc_27b757, Enc_27fd0e, Enc_284ebb, Enc_28a2dc, ... (28 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_27b757, Enc_27fd0e, Enc_284ebb, Enc_28a2dc, Enc_28dcbb, ... (28 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_27b757, Enc_27fd0e, Enc_284ebb, Enc_28a2dc, ... (28 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_27b757, Enc_27fd0e, Enc_284ebb, Enc_28a2dc, Enc_28dcbb, ... (28 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 751-1000 / 第 751-1000 行

```tablegen
   751:   bits <5> Rss32;
   752:   let Inst{20-16} = Rss32{4-0};
   753:   bits <2> Pd4;
   754:   let Inst{1-0} = Pd4{1-0};
   755: }
   756: class Enc_3694bd : OpcodeHexagon {
   757:   bits <11> Ii;
   758:   let Inst{21-20} = Ii{10-9};
   759:   let Inst{7-1} = Ii{8-2};
   760:   bits <3> Ns8;
   761:   let Inst{18-16} = Ns8{2-0};
   762:   bits <0> n1;
   763: }
   764: class Enc_372c9d : OpcodeHexagon {
   765:   bits <2> Pv4;
   766:   let Inst{12-11} = Pv4{1-0};
   767:   bits <1> Mu2;
   768:   let Inst{13-13} = Mu2{0-0};
   769:   bits <3> Os8;
   770:   let Inst{2-0} = Os8{2-0};
   771:   bits <5> Rx32;
   772:   let Inst{20-16} = Rx32{4-0};
   773: }
   774: class Enc_395cc4 : OpcodeHexagon {
   775:   bits <7> Ii;
   776:   let Inst{6-3} = Ii{6-3};
   777:   bits <1> Mu2;
   778:   let Inst{13-13} = Mu2{0-0};
   779:   bits <5> Rtt32;
   780:   let Inst{12-8} = Rtt32{4-0};
   781:   bits <5> Rx32;
   782:   let Inst{20-16} = Rx32{4-0};
   783: }
   784: class Enc_397f23 : OpcodeHexagon {
   785:   bits <8> Ii;
   786:   let Inst{13-13} = Ii{7-7};
   787:   let Inst{7-3} = Ii{6-2};
   788:   bits <2> Pv4;
   789:   let Inst{1-0} = Pv4{1-0};
   790:   bits <5> Rs32;
   791:   let Inst{20-16} = Rs32{4-0};
   792:   bits <5> Rt32;
   793:   let Inst{12-8} = Rt32{4-0};
   794: }
   795: class Enc_399e12 : OpcodeHexagon {
   796:   bits <4> Rs16;
   797:   let Inst{7-4} = Rs16{3-0};
   798:   bits <3> Rdd8;
   799:   let Inst{2-0} = Rdd8{2-0};
   800: }
   801: class Enc_3a2484 : OpcodeHexagon {
   802:   bits <11> Ii;
   803:   let Inst{21-20} = Ii{10-9};
   804:   let Inst{7-1} = Ii{8-2};
   805:   bits <4> Rs16;
   806:   let Inst{19-16} = Rs16{3-0};
   807:   bits <0> n1;
   808: }
   809: class Enc_3a3d62 : OpcodeHexagon {
   810:   bits <5> Rs32;
   811:   let Inst{20-16} = Rs32{4-0};
   812:   bits <5> Rdd32;
   813:   let Inst{4-0} = Rdd32{4-0};
   814: }
   815: class Enc_3b7631 : OpcodeHexagon {
   816:   bits <5> Vu32;
   817:   let Inst{12-8} = Vu32{4-0};
   818:   bits <5> Vdddd32;
   819:   let Inst{4-0} = Vdddd32{4-0};
   820:   bits <3> Rx8;
   821:   let Inst{18-16} = Rx8{2-0};
   822: }
   823: class Enc_3d5b28 : OpcodeHexagon {
   824:   bits <5> Rss32;
   825:   let Inst{20-16} = Rss32{4-0};
   826:   bits <5> Rt32;
   827:   let Inst{12-8} = Rt32{4-0};
   828:   bits <5> Rd32;
   829:   let Inst{4-0} = Rd32{4-0};
   830: }
   831: class Enc_3d6d37 : OpcodeHexagon {
   832:   bits <2> Qs4;
   833:   let Inst{6-5} = Qs4{1-0};
   834:   bits <5> Rt32;
   835:   let Inst{20-16} = Rt32{4-0};
   836:   bits <1> Mu2;
   837:   let Inst{13-13} = Mu2{0-0};
   838:   bits <5> Vvv32;
   839:   let Inst{12-8} = Vvv32{4-0};
   840:   bits <5> Vw32;
   841:   let Inst{4-0} = Vw32{4-0};
   842: }
   843: class Enc_3d920a : OpcodeHexagon {
   844:   bits <6> Ii;
   845:   let Inst{8-5} = Ii{5-2};
   846:   bits <5> Rd32;
   847:   let Inst{4-0} = Rd32{4-0};
   848:   bits <5> Rx32;
   849:   let Inst{20-16} = Rx32{4-0};
   850: }
   851: class Enc_3dac0b : OpcodeHexagon {
   852:   bits <2> Qt4;
   853:   let Inst{6-5} = Qt4{1-0};
   854:   bits <5> Vu32;
   855:   let Inst{12-8} = Vu32{4-0};
   856:   bits <5> Vv32;
   857:   let Inst{20-16} = Vv32{4-0};
   858:   bits <5> Vdd32;
   859:   let Inst{4-0} = Vdd32{4-0};
   860: }
   861: class Enc_3e3989 : OpcodeHexagon {
   862:   bits <11> Ii;
   863:   let Inst{21-20} = Ii{10-9};
   864:   let Inst{7-1} = Ii{8-2};
   865:   bits <4> Rs16;
   866:   let Inst{19-16} = Rs16{3-0};
   867:   bits <0> n1;
   868: }
   869: class Enc_3f97c8 : OpcodeHexagon {
   870:   bits <6> Ii;
   871:   let Inst{6-3} = Ii{5-2};
   872:   bits <1> Mu2;
   873:   let Inst{13-13} = Mu2{0-0};
   874:   bits <3> Nt8;
   875:   let Inst{10-8} = Nt8{2-0};
   876:   bits <5> Rx32;
   877:   let Inst{20-16} = Rx32{4-0};
   878: }
   879: class Enc_3fc427 : OpcodeHexagon {
   880:   bits <5> Vu32;
   881:   let Inst{12-8} = Vu32{4-0};
   882:   bits <5> Vv32;
   883:   let Inst{20-16} = Vv32{4-0};
   884:   bits <5> Vxx32;
   885:   let Inst{4-0} = Vxx32{4-0};
   886: }
   887: class Enc_403871 : OpcodeHexagon {
   888:   bits <5> Rx32;
   889:   let Inst{20-16} = Rx32{4-0};
   890: }
   891: class Enc_405228 : OpcodeHexagon {
   892:   bits <11> Ii;
   893:   let Inst{21-20} = Ii{10-9};
   894:   let Inst{7-1} = Ii{8-2};
   895:   bits <4> Rs16;
   896:   let Inst{19-16} = Rs16{3-0};
   897:   bits <0> n1;
   898: }
   899: class Enc_412ff0 : OpcodeHexagon {
   900:   bits <5> Rss32;
   901:   let Inst{20-16} = Rss32{4-0};
   902:   bits <5> Ru32;
   903:   let Inst{4-0} = Ru32{4-0};
   904:   bits <5> Rxx32;
   905:   let Inst{12-8} = Rxx32{4-0};
   906: }
   907: class Enc_420cf3 : OpcodeHexagon {
   908:   bits <6> Ii;
   909:   let Inst{22-21} = Ii{5-4};
   910:   let Inst{13-13} = Ii{3-3};
   911:   let Inst{7-5} = Ii{2-0};
   912:   bits <5> Ru32;
   913:   let Inst{4-0} = Ru32{4-0};
   914:   bits <5> Rs32;
   915:   let Inst{20-16} = Rs32{4-0};
   916:   bits <5> Rd32;
   917:   let Inst{12-8} = Rd32{4-0};
   918: }
   919: class Enc_437f33 : OpcodeHexagon {
   920:   bits <5> Rs32;
   921:   let Inst{20-16} = Rs32{4-0};
   922:   bits <5> Rt32;
   923:   let Inst{12-8} = Rt32{4-0};
   924:   bits <2> Pu4;
   925:   let Inst{6-5} = Pu4{1-0};
   926:   bits <5> Rx32;
   927:   let Inst{4-0} = Rx32{4-0};
   928: }
   929: class Enc_44215c : OpcodeHexagon {
   930:   bits <6> Ii;
   931:   let Inst{17-16} = Ii{5-4};
   932:   let Inst{6-3} = Ii{3-0};
   933:   bits <2> Pv4;
   934:   let Inst{1-0} = Pv4{1-0};
   935:   bits <3> Nt8;
   936:   let Inst{10-8} = Nt8{2-0};
   937: }
   938: class Enc_44271f : OpcodeHexagon {
   939:   bits <5> Gs32;
   940:   let Inst{20-16} = Gs32{4-0};
   941:   bits <5> Rd32;
   942:   let Inst{4-0} = Rd32{4-0};
   943: }
   944: class Enc_44661f : OpcodeHexagon {
   945:   bits <1> Mu2;
   946:   let Inst{13-13} = Mu2{0-0};
   947:   bits <5> Rx32;
   948:   let Inst{20-16} = Rx32{4-0};
   949: }
   950: class Enc_448f7f : OpcodeHexagon {
   951:   bits <11> Ii;
   952:   let Inst{26-25} = Ii{10-9};
   953:   let Inst{13-13} = Ii{8-8};
   954:   let Inst{7-0} = Ii{7-0};
   955:   bits <5> Rs32;
   956:   let Inst{20-16} = Rs32{4-0};
   957:   bits <5> Rt32;
   958:   let Inst{12-8} = Rt32{4-0};
   959: }
   960: class Enc_45364e : OpcodeHexagon {
   961:   bits <5> Vu32;
   962:   let Inst{12-8} = Vu32{4-0};
   963:   bits <5> Vv32;
   964:   let Inst{20-16} = Vv32{4-0};
   965:   bits <5> Vd32;
   966:   let Inst{4-0} = Vd32{4-0};
   967: }
   968: class Enc_454a26 : OpcodeHexagon {
   969:   bits <2> Pt4;
   970:   let Inst{9-8} = Pt4{1-0};
   971:   bits <2> Ps4;
   972:   let Inst{17-16} = Ps4{1-0};
   973:   bits <2> Pd4;
   974:   let Inst{1-0} = Pd4{1-0};
   975: }
   976: class Enc_46c951 : OpcodeHexagon {
   977:   bits <6> Ii;
   978:   let Inst{12-7} = Ii{5-0};
   979:   bits <5> II;
   980:   let Inst{4-0} = II{4-0};
   981:   bits <5> Rs32;
   982:   let Inst{20-16} = Rs32{4-0};
   983: }
   984: class Enc_46f33d : OpcodeHexagon {
   985:   bits <5> Rss32;
   986:   let Inst{20-16} = Rss32{4-0};
   987:   bits <5> Rt32;
   988:   let Inst{12-8} = Rt32{4-0};
   989: }
   990: class Enc_47ee5e : OpcodeHexagon {
   991:   bits <2> Ii;
   992:   let Inst{13-13} = Ii{1-1};
   993:   let Inst{7-7} = Ii{0-0};
   994:   bits <2> Pv4;
   995:   let Inst{6-5} = Pv4{1-0};
   996:   bits <5> Rs32;
   997:   let Inst{20-16} = Rs32{4-0};
   998:   bits <5> Ru32;
   999:   let Inst{12-8} = Ru32{4-0};
  1000:   bits <3> Nt8;
```
- EN: It declares types such as Enc_3694bd, Enc_372c9d, Enc_395cc4, Enc_397f23, ... (29 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_3694bd, Enc_372c9d, Enc_395cc4, Enc_397f23, Enc_399e12, ... (29 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_3694bd, Enc_372c9d, Enc_395cc4, Enc_397f23, ... (29 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_3694bd, Enc_372c9d, Enc_395cc4, Enc_397f23, Enc_399e12, ... (29 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 1001-1250 / 第 1001-1250 行

```tablegen
  1001:   let Inst{2-0} = Nt8{2-0};
  1002: }
  1003: class Enc_47ef61 : OpcodeHexagon {
  1004:   bits <3> Ii;
  1005:   let Inst{7-5} = Ii{2-0};
  1006:   bits <5> Rt32;
  1007:   let Inst{12-8} = Rt32{4-0};
  1008:   bits <5> Rs32;
  1009:   let Inst{20-16} = Rs32{4-0};
  1010:   bits <5> Rd32;
  1011:   let Inst{4-0} = Rd32{4-0};
  1012: }
  1013: class Enc_48b75f : OpcodeHexagon {
  1014:   bits <5> Rs32;
  1015:   let Inst{20-16} = Rs32{4-0};
  1016:   bits <2> Pd4;
  1017:   let Inst{1-0} = Pd4{1-0};
  1018: }
  1019: class Enc_4aca3a : OpcodeHexagon {
  1020:   bits <11> Ii;
  1021:   let Inst{21-20} = Ii{10-9};
  1022:   let Inst{7-1} = Ii{8-2};
  1023:   bits <3> Ns8;
  1024:   let Inst{18-16} = Ns8{2-0};
  1025:   bits <0> n1;
  1026: }
  1027: class Enc_4b39e4 : OpcodeHexagon {
  1028:   bits <3> Ii;
  1029:   let Inst{7-5} = Ii{2-0};
  1030:   bits <5> Vu32;
  1031:   let Inst{12-8} = Vu32{4-0};
  1032:   bits <5> Vv32;
  1033:   let Inst{20-16} = Vv32{4-0};
  1034:   bits <5> Vdd32;
  1035:   let Inst{4-0} = Vdd32{4-0};
  1036: }
  1037: class Enc_4dc228 : OpcodeHexagon {
  1038:   bits <9> Ii;
  1039:   let Inst{12-8} = Ii{8-4};
  1040:   let Inst{4-3} = Ii{3-2};
  1041:   bits <10> II;
  1042:   let Inst{20-16} = II{9-5};
  1043:   let Inst{7-5} = II{4-2};
  1044:   let Inst{1-0} = II{1-0};
  1045: }
  1046: class Enc_4df4e9 : OpcodeHexagon {
  1047:   bits <11> Ii;
  1048:   let Inst{26-25} = Ii{10-9};
  1049:   let Inst{13-13} = Ii{8-8};
  1050:   let Inst{7-0} = Ii{7-0};
  1051:   bits <5> Rs32;
  1052:   let Inst{20-16} = Rs32{4-0};
  1053:   bits <3> Nt8;
  1054:   let Inst{10-8} = Nt8{2-0};
  1055: }
  1056: class Enc_4dff07 : OpcodeHexagon {
  1057:   bits <2> Qv4;
  1058:   let Inst{12-11} = Qv4{1-0};
  1059:   bits <1> Mu2;
  1060:   let Inst{13-13} = Mu2{0-0};
  1061:   bits <5> Vs32;
  1062:   let Inst{4-0} = Vs32{4-0};
  1063:   bits <5> Rx32;
  1064:   let Inst{20-16} = Rx32{4-0};
  1065: }
  1066: class Enc_4e4a80 : OpcodeHexagon {
  1067:   bits <2> Qs4;
  1068:   let Inst{6-5} = Qs4{1-0};
  1069:   bits <5> Rt32;
  1070:   let Inst{20-16} = Rt32{4-0};
  1071:   bits <1> Mu2;
  1072:   let Inst{13-13} = Mu2{0-0};
  1073:   bits <5> Vvv32;
  1074:   let Inst{4-0} = Vvv32{4-0};
  1075: }
  1076: class Enc_4f4ed7 : OpcodeHexagon {
  1077:   bits <18> Ii;
  1078:   let Inst{26-25} = Ii{17-16};
  1079:   let Inst{20-16} = Ii{15-11};
  1080:   let Inst{13-5} = Ii{10-2};
  1081:   bits <5> Rd32;
  1082:   let Inst{4-0} = Rd32{4-0};
  1083: }
  1084: class Enc_4f677b : OpcodeHexagon {
  1085:   bits <2> Ii;
  1086:   let Inst{13-13} = Ii{1-1};
  1087:   let Inst{7-7} = Ii{0-0};
  1088:   bits <6> II;
  1089:   let Inst{11-8} = II{5-2};
  1090:   let Inst{6-5} = II{1-0};
  1091:   bits <5> Rt32;
  1092:   let Inst{20-16} = Rt32{4-0};
  1093:   bits <5> Rd32;
  1094:   let Inst{4-0} = Rd32{4-0};
  1095: }
  1096: class Enc_500cb0 : OpcodeHexagon {
  1097:   bits <5> Vu32;
  1098:   let Inst{12-8} = Vu32{4-0};
  1099:   bits <5> Vxx32;
  1100:   let Inst{4-0} = Vxx32{4-0};
  1101: }
  1102: class Enc_509701 : OpcodeHexagon {
  1103:   bits <19> Ii;
  1104:   let Inst{26-25} = Ii{18-17};
  1105:   let Inst{20-16} = Ii{16-12};
  1106:   let Inst{13-5} = Ii{11-3};
  1107:   bits <5> Rdd32;
  1108:   let Inst{4-0} = Rdd32{4-0};
  1109: }
  1110: class Enc_50b5ac : OpcodeHexagon {
  1111:   bits <6> Ii;
  1112:   let Inst{17-16} = Ii{5-4};
  1113:   let Inst{6-3} = Ii{3-0};
  1114:   bits <2> Pv4;
  1115:   let Inst{1-0} = Pv4{1-0};
  1116:   bits <5> Rtt32;
  1117:   let Inst{12-8} = Rtt32{4-0};
  1118: }
  1119: class Enc_50e578 : OpcodeHexagon {
  1120:   bits <5> Vu32;
  1121:   let Inst{12-8} = Vu32{4-0};
  1122:   bits <5> Rs32;
  1123:   let Inst{20-16} = Rs32{4-0};
  1124:   bits <5> Rd32;
  1125:   let Inst{4-0} = Rd32{4-0};
  1126: }
  1127: class Enc_5138b3 : OpcodeHexagon {
  1128:   bits <5> Vu32;
  1129:   let Inst{12-8} = Vu32{4-0};
  1130:   bits <5> Rt32;
  1131:   let Inst{20-16} = Rt32{4-0};
  1132:   bits <5> Vx32;
  1133:   let Inst{4-0} = Vx32{4-0};
  1134: }
  1135: class Enc_51436c : OpcodeHexagon {
  1136:   bits <16> Ii;
  1137:   let Inst{23-22} = Ii{15-14};
  1138:   let Inst{13-0} = Ii{13-0};
  1139:   bits <5> Rx32;
  1140:   let Inst{20-16} = Rx32{4-0};
  1141: }
  1142: class Enc_51635c : OpcodeHexagon {
  1143:   bits <7> Ii;
  1144:   let Inst{8-4} = Ii{6-2};
  1145:   bits <4> Rd16;
  1146:   let Inst{3-0} = Rd16{3-0};
  1147: }
  1148: class Enc_527412 : OpcodeHexagon {
  1149:   bits <2> Ps4;
  1150:   let Inst{17-16} = Ps4{1-0};
  1151:   bits <2> Pt4;
  1152:   let Inst{9-8} = Pt4{1-0};
  1153:   bits <5> Rd32;
  1154:   let Inst{4-0} = Rd32{4-0};
  1155: }
  1156: class Enc_52a5dd : OpcodeHexagon {
  1157:   bits <4> Ii;
  1158:   let Inst{6-3} = Ii{3-0};
  1159:   bits <2> Pv4;
  1160:   let Inst{1-0} = Pv4{1-0};
  1161:   bits <3> Nt8;
  1162:   let Inst{10-8} = Nt8{2-0};
  1163:   bits <5> Rx32;
  1164:   let Inst{20-16} = Rx32{4-0};
  1165: }
  1166: class Enc_53dca9 : OpcodeHexagon {
  1167:   bits <6> Ii;
  1168:   let Inst{11-8} = Ii{5-2};
  1169:   bits <4> Rs16;
  1170:   let Inst{7-4} = Rs16{3-0};
  1171:   bits <4> Rd16;
  1172:   let Inst{3-0} = Rd16{3-0};
  1173: }
  1174: class Enc_541f26 : OpcodeHexagon {
  1175:   bits <18> Ii;
  1176:   let Inst{26-25} = Ii{17-16};
  1177:   let Inst{20-16} = Ii{15-11};
  1178:   let Inst{13-13} = Ii{10-10};
  1179:   let Inst{7-0} = Ii{9-2};
  1180:   bits <5> Rt32;
  1181:   let Inst{12-8} = Rt32{4-0};
  1182: }
  1183: class Enc_55355c : OpcodeHexagon {
  1184:   bits <2> Ii;
  1185:   let Inst{13-13} = Ii{1-1};
  1186:   let Inst{7-7} = Ii{0-0};
  1187:   bits <5> Rs32;
  1188:   let Inst{20-16} = Rs32{4-0};
  1189:   bits <5> Ru32;
  1190:   let Inst{12-8} = Ru32{4-0};
  1191:   bits <5> Rtt32;
  1192:   let Inst{4-0} = Rtt32{4-0};
  1193: }
  1194: class Enc_569cfe : OpcodeHexagon {
  1195:   bits <5> Rt32;
  1196:   let Inst{20-16} = Rt32{4-0};
  1197:   bits <5> Vx32;
  1198:   let Inst{4-0} = Vx32{4-0};
  1199: }
  1200: class Enc_57a33e : OpcodeHexagon {
  1201:   bits <9> Ii;
  1202:   let Inst{13-13} = Ii{8-8};
  1203:   let Inst{7-3} = Ii{7-3};
  1204:   bits <2> Pv4;
  1205:   let Inst{1-0} = Pv4{1-0};
  1206:   bits <5> Rs32;
  1207:   let Inst{20-16} = Rs32{4-0};
  1208:   bits <5> Rtt32;
  1209:   let Inst{12-8} = Rtt32{4-0};
  1210: }
  1211: class Enc_585242 : OpcodeHexagon {
  1212:   bits <6> Ii;
  1213:   let Inst{13-13} = Ii{5-5};
  1214:   let Inst{7-3} = Ii{4-0};
  1215:   bits <2> Pv4;
  1216:   let Inst{1-0} = Pv4{1-0};
  1217:   bits <5> Rs32;
  1218:   let Inst{20-16} = Rs32{4-0};
  1219:   bits <3> Nt8;
  1220:   let Inst{10-8} = Nt8{2-0};
  1221: }
  1222: class Enc_58a8bf : OpcodeHexagon {
  1223:   bits <3> Ii;
  1224:   let Inst{10-8} = Ii{2-0};
  1225:   bits <2> Pv4;
  1226:   let Inst{12-11} = Pv4{1-0};
  1227:   bits <5> Vd32;
  1228:   let Inst{4-0} = Vd32{4-0};
  1229:   bits <5> Rx32;
  1230:   let Inst{20-16} = Rx32{4-0};
  1231: }
  1232: class Enc_598f6c : OpcodeHexagon {
  1233:   bits <5> Rtt32;
  1234:   let Inst{12-8} = Rtt32{4-0};
  1235: }
  1236: class Enc_5a18b3 : OpcodeHexagon {
  1237:   bits <11> Ii;
  1238:   let Inst{21-20} = Ii{10-9};
  1239:   let Inst{7-1} = Ii{8-2};
  1240:   bits <3> Ns8;
  1241:   let Inst{18-16} = Ns8{2-0};
  1242:   bits <0> n1;
  1243: }
  1244: class Enc_5ab2be : OpcodeHexagon {
  1245:   bits <5> Rs32;
  1246:   let Inst{20-16} = Rs32{4-0};
  1247:   bits <5> Rt32;
  1248:   let Inst{12-8} = Rt32{4-0};
  1249:   bits <5> Rd32;
  1250:   let Inst{4-0} = Rd32{4-0};
```
- EN: It declares types such as Enc_47ef61, Enc_48b75f, Enc_4aca3a, Enc_4b39e4, ... (29 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_47ef61, Enc_48b75f, Enc_4aca3a, Enc_4b39e4, Enc_4dc228, ... (29 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_47ef61, Enc_48b75f, Enc_4aca3a, Enc_4b39e4, ... (29 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_47ef61, Enc_48b75f, Enc_4aca3a, Enc_4b39e4, Enc_4dc228, ... (29 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 1251-1500 / 第 1251-1500 行

```tablegen
  1251: }
  1252: class Enc_5bdd42 : OpcodeHexagon {
  1253:   bits <7> Ii;
  1254:   let Inst{8-5} = Ii{6-3};
  1255:   bits <5> Rdd32;
  1256:   let Inst{4-0} = Rdd32{4-0};
  1257:   bits <5> Rx32;
  1258:   let Inst{20-16} = Rx32{4-0};
  1259: }
  1260: class Enc_5c124a : OpcodeHexagon {
  1261:   bits <19> Ii;
  1262:   let Inst{26-25} = Ii{18-17};
  1263:   let Inst{20-16} = Ii{16-12};
  1264:   let Inst{13-13} = Ii{11-11};
  1265:   let Inst{7-0} = Ii{10-3};
  1266:   bits <5> Rtt32;
  1267:   let Inst{12-8} = Rtt32{4-0};
  1268: }
  1269: class Enc_5ccba9 : OpcodeHexagon {
  1270:   bits <8> Ii;
  1271:   let Inst{12-7} = Ii{7-2};
  1272:   bits <6> II;
  1273:   let Inst{13-13} = II{5-5};
  1274:   let Inst{4-0} = II{4-0};
  1275:   bits <2> Pv4;
  1276:   let Inst{6-5} = Pv4{1-0};
  1277:   bits <5> Rs32;
  1278:   let Inst{20-16} = Rs32{4-0};
  1279: }
  1280: class Enc_5cd7e9 : OpcodeHexagon {
  1281:   bits <12> Ii;
  1282:   let Inst{26-25} = Ii{11-10};
  1283:   let Inst{13-5} = Ii{9-1};
  1284:   bits <5> Rs32;
  1285:   let Inst{20-16} = Rs32{4-0};
  1286:   bits <5> Ryy32;
  1287:   let Inst{4-0} = Ryy32{4-0};
  1288: }
  1289: class Enc_5d6c34 : OpcodeHexagon {
  1290:   bits <6> Ii;
  1291:   let Inst{13-8} = Ii{5-0};
  1292:   bits <5> Rs32;
  1293:   let Inst{20-16} = Rs32{4-0};
  1294:   bits <2> Pd4;
  1295:   let Inst{1-0} = Pd4{1-0};
  1296: }
  1297: class Enc_5de85f : OpcodeHexagon {
  1298:   bits <11> Ii;
  1299:   let Inst{21-20} = Ii{10-9};
  1300:   let Inst{7-1} = Ii{8-2};
  1301:   bits <5> Rt32;
  1302:   let Inst{12-8} = Rt32{4-0};
  1303:   bits <3> Ns8;
  1304:   let Inst{18-16} = Ns8{2-0};
  1305: }
  1306: class Enc_5e2823 : OpcodeHexagon {
  1307:   bits <5> Rs32;
  1308:   let Inst{20-16} = Rs32{4-0};
  1309:   bits <5> Rd32;
  1310:   let Inst{4-0} = Rd32{4-0};
  1311: }
  1312: class Enc_5e8512 : OpcodeHexagon {
  1313:   bits <5> Vu32;
  1314:   let Inst{12-8} = Vu32{4-0};
  1315:   bits <5> Rt32;
  1316:   let Inst{20-16} = Rt32{4-0};
  1317:   bits <5> Vxx32;
  1318:   let Inst{4-0} = Vxx32{4-0};
  1319: }
  1320: class Enc_5e87ce : OpcodeHexagon {
  1321:   bits <16> Ii;
  1322:   let Inst{23-22} = Ii{15-14};
  1323:   let Inst{20-16} = Ii{13-9};
  1324:   let Inst{13-5} = Ii{8-0};
  1325:   bits <5> Rd32;
  1326:   let Inst{4-0} = Rd32{4-0};
  1327: }
  1328: class Enc_5eac98 : OpcodeHexagon {
  1329:   bits <6> Ii;
  1330:   let Inst{13-8} = Ii{5-0};
  1331:   bits <5> Rss32;
  1332:   let Inst{20-16} = Rss32{4-0};
  1333:   bits <5> Rdd32;
  1334:   let Inst{4-0} = Rdd32{4-0};
  1335: }
  1336: class Enc_5eb169 : OpcodeHexagon {
  1337:   bits <3> Ii;
  1338:   let Inst{10-8} = Ii{2-0};
  1339:   bits <5> Vdd32;
  1340:   let Inst{4-0} = Vdd32{4-0};
  1341:   bits <5> Rx32;
  1342:   let Inst{20-16} = Rx32{4-0};
  1343: }
  1344: class Enc_607661 : OpcodeHexagon {
  1345:   bits <6> Ii;
  1346:   let Inst{12-7} = Ii{5-0};
  1347:   bits <5> Rd32;
  1348:   let Inst{4-0} = Rd32{4-0};
  1349: }
  1350: class Enc_6185fe : OpcodeHexagon {
  1351:   bits <2> Ii;
  1352:   let Inst{13-13} = Ii{1-1};
  1353:   let Inst{7-7} = Ii{0-0};
  1354:   bits <6> II;
  1355:   let Inst{11-8} = II{5-2};
  1356:   let Inst{6-5} = II{1-0};
  1357:   bits <5> Rt32;
  1358:   let Inst{20-16} = Rt32{4-0};
  1359:   bits <5> Rdd32;
  1360:   let Inst{4-0} = Rdd32{4-0};
  1361: }
  1362: class Enc_61f0b0 : OpcodeHexagon {
  1363:   bits <5> Rs32;
  1364:   let Inst{20-16} = Rs32{4-0};
  1365:   bits <5> Rt32;
  1366:   let Inst{12-8} = Rt32{4-0};
  1367:   bits <5> Rxx32;
  1368:   let Inst{4-0} = Rxx32{4-0};
  1369: }
  1370: class Enc_621fba : OpcodeHexagon {
  1371:   bits <5> Rs32;
  1372:   let Inst{20-16} = Rs32{4-0};
  1373:   bits <5> Gd32;
  1374:   let Inst{4-0} = Gd32{4-0};
  1375: }
  1376: class Enc_625deb : OpcodeHexagon {
  1377:   bits <4> Ii;
  1378:   let Inst{10-8} = Ii{3-1};
  1379:   bits <4> Rs16;
  1380:   let Inst{7-4} = Rs16{3-0};
  1381:   bits <4> Rt16;
  1382:   let Inst{3-0} = Rt16{3-0};
  1383: }
  1384: class Enc_6339d5 : OpcodeHexagon {
  1385:   bits <2> Ii;
  1386:   let Inst{13-13} = Ii{1-1};
  1387:   let Inst{7-7} = Ii{0-0};
  1388:   bits <2> Pv4;
  1389:   let Inst{6-5} = Pv4{1-0};
  1390:   bits <5> Rs32;
  1391:   let Inst{20-16} = Rs32{4-0};
  1392:   bits <5> Ru32;
  1393:   let Inst{12-8} = Ru32{4-0};
  1394:   bits <5> Rt32;
  1395:   let Inst{4-0} = Rt32{4-0};
  1396: }
  1397: class Enc_634460 : OpcodeHexagon {
  1398:   bits <4> Ii;
  1399:   let Inst{13-13} = Ii{3-3};
  1400:   let Inst{10-8} = Ii{2-0};
  1401:   bits <5> Rt32;
  1402:   let Inst{20-16} = Rt32{4-0};
  1403:   bits <5> Vdd32;
  1404:   let Inst{4-0} = Vdd32{4-0};
  1405: }
  1406: class Enc_63eaeb : OpcodeHexagon {
  1407:   bits <2> Ii;
  1408:   let Inst{1-0} = Ii{1-0};
  1409:   bits <4> Rs16;
  1410:   let Inst{7-4} = Rs16{3-0};
  1411: }
  1412: class Enc_6413b6 : OpcodeHexagon {
  1413:   bits <11> Ii;
  1414:   let Inst{21-20} = Ii{10-9};
  1415:   let Inst{7-1} = Ii{8-2};
  1416:   bits <3> Ns8;
  1417:   let Inst{18-16} = Ns8{2-0};
  1418:   bits <0> n1;
  1419: }
  1420: class Enc_645d54 : OpcodeHexagon {
  1421:   bits <2> Ii;
  1422:   let Inst{13-13} = Ii{1-1};
  1423:   let Inst{5-5} = Ii{0-0};
  1424:   bits <5> Rss32;
  1425:   let Inst{20-16} = Rss32{4-0};
  1426:   bits <5> Rt32;
  1427:   let Inst{12-8} = Rt32{4-0};
  1428:   bits <5> Rdd32;
  1429:   let Inst{4-0} = Rdd32{4-0};
  1430: }
  1431: class Enc_65d691 : OpcodeHexagon {
  1432:   bits <2> Ps4;
  1433:   let Inst{17-16} = Ps4{1-0};
  1434:   bits <2> Pd4;
  1435:   let Inst{1-0} = Pd4{1-0};
  1436: }
  1437: class Enc_65f095 : OpcodeHexagon {
  1438:   bits <6> Ii;
  1439:   let Inst{6-3} = Ii{5-2};
  1440:   bits <2> Pv4;
  1441:   let Inst{1-0} = Pv4{1-0};
  1442:   bits <3> Nt8;
  1443:   let Inst{10-8} = Nt8{2-0};
  1444:   bits <5> Rx32;
  1445:   let Inst{20-16} = Rx32{4-0};
  1446: }
  1447: class Enc_667b39 : OpcodeHexagon {
  1448:   bits <5> Css32;
  1449:   let Inst{20-16} = Css32{4-0};
  1450:   bits <5> Rdd32;
  1451:   let Inst{4-0} = Rdd32{4-0};
  1452: }
  1453: class Enc_668704 : OpcodeHexagon {
  1454:   bits <11> Ii;
  1455:   let Inst{21-20} = Ii{10-9};
  1456:   let Inst{7-1} = Ii{8-2};
  1457:   bits <4> Rs16;
  1458:   let Inst{19-16} = Rs16{3-0};
  1459:   bits <0> n1;
  1460: }
  1461: class Enc_66bce1 : OpcodeHexagon {
  1462:   bits <11> Ii;
  1463:   let Inst{21-20} = Ii{10-9};
  1464:   let Inst{7-1} = Ii{8-2};
  1465:   bits <4> Rs16;
  1466:   let Inst{19-16} = Rs16{3-0};
  1467:   bits <4> Rd16;
  1468:   let Inst{11-8} = Rd16{3-0};
  1469: }
  1470: class Enc_690862 : OpcodeHexagon {
  1471:   bits <13> Ii;
  1472:   let Inst{26-25} = Ii{12-11};
  1473:   let Inst{13-13} = Ii{10-10};
  1474:   let Inst{7-0} = Ii{9-2};
  1475:   bits <5> Rs32;
  1476:   let Inst{20-16} = Rs32{4-0};
  1477:   bits <3> Nt8;
  1478:   let Inst{10-8} = Nt8{2-0};
  1479: }
  1480: class Enc_691712 : OpcodeHexagon {
  1481:   bits <2> Pv4;
  1482:   let Inst{12-11} = Pv4{1-0};
  1483:   bits <1> Mu2;
  1484:   let Inst{13-13} = Mu2{0-0};
  1485:   bits <5> Rx32;
  1486:   let Inst{20-16} = Rx32{4-0};
  1487: }
  1488: class Enc_69d63b : OpcodeHexagon {
  1489:   bits <11> Ii;
  1490:   let Inst{21-20} = Ii{10-9};
  1491:   let Inst{7-1} = Ii{8-2};
  1492:   bits <3> Ns8;
  1493:   let Inst{18-16} = Ns8{2-0};
  1494: }
  1495: class Enc_6a5972 : OpcodeHexagon {
  1496:   bits <11> Ii;
  1497:   let Inst{21-20} = Ii{10-9};
  1498:   let Inst{7-1} = Ii{8-2};
  1499:   bits <4> Rs16;
  1500:   let Inst{19-16} = Rs16{3-0};
```
- EN: It declares types such as Enc_5bdd42, Enc_5c124a, Enc_5ccba9, Enc_5cd7e9, ... (30 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_5bdd42, Enc_5c124a, Enc_5ccba9, Enc_5cd7e9, Enc_5d6c34, ... (30 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_5bdd42, Enc_5c124a, Enc_5ccba9, Enc_5cd7e9, ... (30 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_5bdd42, Enc_5c124a, Enc_5ccba9, Enc_5cd7e9, Enc_5d6c34, ... (30 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 1501-1750 / 第 1501-1750 行

```tablegen
  1501:   bits <4> Rt16;
  1502:   let Inst{11-8} = Rt16{3-0};
  1503: }
  1504: class Enc_6b197f : OpcodeHexagon {
  1505:   bits <4> Ii;
  1506:   let Inst{8-5} = Ii{3-0};
  1507:   bits <5> Ryy32;
  1508:   let Inst{4-0} = Ryy32{4-0};
  1509:   bits <5> Rx32;
  1510:   let Inst{20-16} = Rx32{4-0};
  1511: }
  1512: class Enc_6baed4 : OpcodeHexagon {
  1513:   bits <3> Ii;
  1514:   let Inst{10-8} = Ii{2-0};
  1515:   bits <2> Pv4;
  1516:   let Inst{12-11} = Pv4{1-0};
  1517:   bits <5> Rx32;
  1518:   let Inst{20-16} = Rx32{4-0};
  1519: }
  1520: class Enc_6c9440 : OpcodeHexagon {
  1521:   bits <10> Ii;
  1522:   let Inst{21-21} = Ii{9-9};
  1523:   let Inst{13-5} = Ii{8-0};
  1524:   bits <5> Rd32;
  1525:   let Inst{4-0} = Rd32{4-0};
  1526: }
  1527: class Enc_6c9ee0 : OpcodeHexagon {
  1528:   bits <3> Ii;
  1529:   let Inst{10-8} = Ii{2-0};
  1530:   bits <5> Rx32;
  1531:   let Inst{20-16} = Rx32{4-0};
  1532: }
  1533: class Enc_6f70ca : OpcodeHexagon {
  1534:   bits <8> Ii;
  1535:   let Inst{8-4} = Ii{7-3};
  1536: }
  1537: class Enc_6f83e7 : OpcodeHexagon {
  1538:   bits <2> Qv4;
  1539:   let Inst{23-22} = Qv4{1-0};
  1540:   bits <5> Vd32;
  1541:   let Inst{4-0} = Vd32{4-0};
  1542: }
  1543: class Enc_70b24b : OpcodeHexagon {
  1544:   bits <6> Ii;
  1545:   let Inst{8-5} = Ii{5-2};
  1546:   bits <1> Mu2;
  1547:   let Inst{13-13} = Mu2{0-0};
  1548:   bits <5> Rdd32;
  1549:   let Inst{4-0} = Rdd32{4-0};
  1550:   bits <5> Rx32;
  1551:   let Inst{20-16} = Rx32{4-0};
  1552: }
  1553: class Enc_70fb07 : OpcodeHexagon {
  1554:   bits <6> Ii;
  1555:   let Inst{13-8} = Ii{5-0};
  1556:   bits <5> Rss32;
  1557:   let Inst{20-16} = Rss32{4-0};
  1558:   bits <5> Rxx32;
  1559:   let Inst{4-0} = Rxx32{4-0};
  1560: }
  1561: class Enc_71bb9b : OpcodeHexagon {
  1562:   bits <5> Vu32;
  1563:   let Inst{12-8} = Vu32{4-0};
  1564:   bits <5> Vv32;
  1565:   let Inst{20-16} = Vv32{4-0};
  1566:   bits <5> Vdd32;
  1567:   let Inst{4-0} = Vdd32{4-0};
  1568: }
  1569: class Enc_71f1b4 : OpcodeHexagon {
  1570:   bits <6> Ii;
  1571:   let Inst{8-5} = Ii{5-2};
  1572:   bits <5> Rdd32;
  1573:   let Inst{4-0} = Rdd32{4-0};
  1574:   bits <5> Rx32;
  1575:   let Inst{20-16} = Rx32{4-0};
  1576: }
  1577: class Enc_7222b7 : OpcodeHexagon {
  1578:   bits <5> Rt32;
  1579:   let Inst{20-16} = Rt32{4-0};
  1580:   bits <2> Qd4;
  1581:   let Inst{1-0} = Qd4{1-0};
  1582: }
  1583: class Enc_724154 : OpcodeHexagon {
  1584:   bits <6> II;
  1585:   let Inst{5-0} = II{5-0};
  1586:   bits <3> Nt8;
  1587:   let Inst{10-8} = Nt8{2-0};
  1588:   bits <5> Re32;
  1589:   let Inst{20-16} = Re32{4-0};
  1590: }
  1591: class Enc_729ff7 : OpcodeHexagon {
  1592:   bits <3> Ii;
  1593:   let Inst{7-5} = Ii{2-0};
  1594:   bits <5> Rtt32;
  1595:   let Inst{12-8} = Rtt32{4-0};
  1596:   bits <5> Rss32;
  1597:   let Inst{20-16} = Rss32{4-0};
  1598:   bits <5> Rdd32;
  1599:   let Inst{4-0} = Rdd32{4-0};
  1600: }
  1601: class Enc_733b27 : OpcodeHexagon {
  1602:   bits <5> Ii;
  1603:   let Inst{8-5} = Ii{4-1};
  1604:   bits <2> Pt4;
  1605:   let Inst{10-9} = Pt4{1-0};
  1606:   bits <5> Rd32;
  1607:   let Inst{4-0} = Rd32{4-0};
  1608:   bits <5> Rx32;
  1609:   let Inst{20-16} = Rx32{4-0};
  1610: }
  1611: class Enc_736575 : OpcodeHexagon {
  1612:   bits <11> Ii;
  1613:   let Inst{21-20} = Ii{10-9};
  1614:   let Inst{7-1} = Ii{8-2};
  1615:   bits <4> Rs16;
  1616:   let Inst{19-16} = Rs16{3-0};
  1617:   bits <0> n1;
  1618: }
  1619: class Enc_74aef2 : OpcodeHexagon {
  1620:   bits <4> Ii;
  1621:   let Inst{8-5} = Ii{3-0};
  1622:   bits <1> Mu2;
  1623:   let Inst{13-13} = Mu2{0-0};
  1624:   bits <5> Ryy32;
  1625:   let Inst{4-0} = Ryy32{4-0};
  1626:   bits <5> Rx32;
  1627:   let Inst{20-16} = Rx32{4-0};
  1628: }
  1629: class Enc_74d4e5 : OpcodeHexagon {
  1630:   bits <1> Mu2;
  1631:   let Inst{13-13} = Mu2{0-0};
  1632:   bits <5> Rd32;
  1633:   let Inst{4-0} = Rd32{4-0};
  1634:   bits <5> Rx32;
  1635:   let Inst{20-16} = Rx32{4-0};
  1636: }
  1637: class Enc_770858 : OpcodeHexagon {
  1638:   bits <2> Ps4;
  1639:   let Inst{6-5} = Ps4{1-0};
  1640:   bits <5> Vu32;
  1641:   let Inst{12-8} = Vu32{4-0};
  1642:   bits <5> Vd32;
  1643:   let Inst{4-0} = Vd32{4-0};
  1644: }
  1645: class Enc_784502 : OpcodeHexagon {
  1646:   bits <3> Ii;
  1647:   let Inst{10-8} = Ii{2-0};
  1648:   bits <2> Pv4;
  1649:   let Inst{12-11} = Pv4{1-0};
  1650:   bits <3> Os8;
  1651:   let Inst{2-0} = Os8{2-0};
  1652:   bits <5> Rx32;
  1653:   let Inst{20-16} = Rx32{4-0};
  1654: }
  1655: class Enc_78cbf0 : OpcodeHexagon {
  1656:   bits <18> Ii;
  1657:   let Inst{26-25} = Ii{17-16};
  1658:   let Inst{20-16} = Ii{15-11};
  1659:   let Inst{13-13} = Ii{10-10};
  1660:   let Inst{7-0} = Ii{9-2};
  1661:   bits <3> Nt8;
  1662:   let Inst{10-8} = Nt8{2-0};
  1663: }
  1664: class Enc_78e566 : OpcodeHexagon {
  1665:   bits <2> Pt4;
  1666:   let Inst{9-8} = Pt4{1-0};
  1667:   bits <5> Rdd32;
  1668:   let Inst{4-0} = Rdd32{4-0};
  1669: }
  1670: class Enc_79b8c8 : OpcodeHexagon {
  1671:   bits <6> Ii;
  1672:   let Inst{6-3} = Ii{5-2};
  1673:   bits <1> Mu2;
  1674:   let Inst{13-13} = Mu2{0-0};
  1675:   bits <5> Rt32;
  1676:   let Inst{12-8} = Rt32{4-0};
  1677:   bits <5> Rx32;
  1678:   let Inst{20-16} = Rx32{4-0};
  1679: }
  1680: class Enc_7a0ea6 : OpcodeHexagon {
  1681:   bits <4> Rd16;
  1682:   let Inst{3-0} = Rd16{3-0};
  1683:   bits <0> n1;
  1684: }
  1685: class Enc_7b523d : OpcodeHexagon {
  1686:   bits <5> Vu32;
  1687:   let Inst{12-8} = Vu32{4-0};
  1688:   bits <5> Vv32;
  1689:   let Inst{23-19} = Vv32{4-0};
  1690:   bits <3> Rt8;
  1691:   let Inst{18-16} = Rt8{2-0};
  1692:   bits <5> Vxx32;
  1693:   let Inst{4-0} = Vxx32{4-0};
  1694: }
  1695: class Enc_7b7ba8 : OpcodeHexagon {
  1696:   bits <2> Qu4;
  1697:   let Inst{9-8} = Qu4{1-0};
  1698:   bits <5> Rt32;
  1699:   let Inst{20-16} = Rt32{4-0};
  1700:   bits <5> Vd32;
  1701:   let Inst{4-0} = Vd32{4-0};
  1702: }
  1703: class Enc_7d1542 : OpcodeHexagon {
  1704:   bits <7> Ss128;
  1705:   let Inst{22-16} = Ss128{6-0};
  1706:   bits <5> Rd32;
  1707:   let Inst{4-0} = Rd32{4-0};
  1708: }
  1709: class Enc_7e5a82 : OpcodeHexagon {
  1710:   bits <5> Ii;
  1711:   let Inst{12-8} = Ii{4-0};
  1712:   bits <5> Rss32;
  1713:   let Inst{20-16} = Rss32{4-0};
  1714:   bits <5> Rdd32;
  1715:   let Inst{4-0} = Rdd32{4-0};
  1716: }
  1717: class Enc_7eaeb6 : OpcodeHexagon {
  1718:   bits <6> Ii;
  1719:   let Inst{6-3} = Ii{5-2};
  1720:   bits <2> Pv4;
  1721:   let Inst{1-0} = Pv4{1-0};
  1722:   bits <5> Rt32;
  1723:   let Inst{12-8} = Rt32{4-0};
  1724:   bits <5> Rx32;
  1725:   let Inst{20-16} = Rx32{4-0};
  1726: }
  1727: class Enc_7eb485 : OpcodeHexagon {
  1728:   bits <2> Ii;
  1729:   let Inst{13-13} = Ii{1-1};
  1730:   let Inst{6-6} = Ii{0-0};
  1731:   bits <6> II;
  1732:   let Inst{5-0} = II{5-0};
  1733:   bits <5> Ru32;
  1734:   let Inst{20-16} = Ru32{4-0};
  1735:   bits <3> Nt8;
  1736:   let Inst{10-8} = Nt8{2-0};
  1737: }
  1738: class Enc_7eee72 : OpcodeHexagon {
  1739:   bits <1> Mu2;
  1740:   let Inst{13-13} = Mu2{0-0};
  1741:   bits <5> Rdd32;
  1742:   let Inst{4-0} = Rdd32{4-0};
  1743:   bits <5> Rx32;
  1744:   let Inst{20-16} = Rx32{4-0};
  1745: }
  1746: class Enc_7f1a05 : OpcodeHexagon {
  1747:   bits <5> Ru32;
  1748:   let Inst{4-0} = Ru32{4-0};
  1749:   bits <5> Rs32;
  1750:   let Inst{20-16} = Rs32{4-0};
```
- EN: It declares types such as Enc_6b197f, Enc_6baed4, Enc_6c9440, Enc_6c9ee0, ... (31 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_6b197f, Enc_6baed4, Enc_6c9440, Enc_6c9ee0, Enc_6f70ca, ... (31 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_6b197f, Enc_6baed4, Enc_6c9440, Enc_6c9ee0, ... (31 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_6b197f, Enc_6baed4, Enc_6c9440, Enc_6c9ee0, Enc_6f70ca, ... (31 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 1751-2000 / 第 1751-2000 行

```tablegen
  1751:   bits <5> Ry32;
  1752:   let Inst{12-8} = Ry32{4-0};
  1753: }
  1754: class Enc_7fa7f6 : OpcodeHexagon {
  1755:   bits <6> II;
  1756:   let Inst{11-8} = II{5-2};
  1757:   let Inst{6-5} = II{1-0};
  1758:   bits <5> Rdd32;
  1759:   let Inst{4-0} = Rdd32{4-0};
  1760:   bits <5> Re32;
  1761:   let Inst{20-16} = Re32{4-0};
  1762: }
  1763: class Enc_800e04 : OpcodeHexagon {
  1764:   bits <11> Ii;
  1765:   let Inst{21-20} = Ii{10-9};
  1766:   let Inst{7-1} = Ii{8-2};
  1767:   bits <4> Rs16;
  1768:   let Inst{19-16} = Rs16{3-0};
  1769:   bits <0> n1;
  1770: }
  1771: class Enc_80296d : OpcodeHexagon {
  1772:   bits <5> Rs32;
  1773:   let Inst{12-8} = Rs32{4-0};
  1774:   bits <5> Rtt32;
  1775:   let Inst{20-16} = Rtt32{4-0};
  1776:   bits <5> Rd32;
  1777:   let Inst{4-0} = Rd32{4-0};
  1778: }
  1779: class Enc_802dc0 : OpcodeHexagon {
  1780:   bits <1> Ii;
  1781:   let Inst{8-8} = Ii{0-0};
  1782:   bits <2> Qv4;
  1783:   let Inst{23-22} = Qv4{1-0};
  1784: }
  1785: class Enc_81ac1d : OpcodeHexagon {
  1786:   bits <24> Ii;
  1787:   let Inst{24-16} = Ii{23-15};
  1788:   let Inst{13-1} = Ii{14-2};
  1789: }
  1790: class Enc_8203bb : OpcodeHexagon {
  1791:   bits <6> Ii;
  1792:   let Inst{12-7} = Ii{5-0};
  1793:   bits <8> II;
  1794:   let Inst{13-13} = II{7-7};
  1795:   let Inst{6-0} = II{6-0};
  1796:   bits <5> Rs32;
  1797:   let Inst{20-16} = Rs32{4-0};
  1798: }
  1799: class Enc_829a68 : OpcodeHexagon {
  1800:   bits <1> Mu2;
  1801:   let Inst{13-13} = Mu2{0-0};
  1802:   bits <5> Vdd32;
  1803:   let Inst{4-0} = Vdd32{4-0};
  1804:   bits <5> Rx32;
  1805:   let Inst{20-16} = Rx32{4-0};
  1806: }
  1807: class Enc_830e5d : OpcodeHexagon {
  1808:   bits <8> Ii;
  1809:   let Inst{12-5} = Ii{7-0};
  1810:   bits <8> II;
  1811:   let Inst{22-16} = II{7-1};
  1812:   let Inst{13-13} = II{0-0};
  1813:   bits <2> Pu4;
  1814:   let Inst{24-23} = Pu4{1-0};
  1815:   bits <5> Rd32;
  1816:   let Inst{4-0} = Rd32{4-0};
  1817: }
  1818: class Enc_831a7d : OpcodeHexagon {
  1819:   bits <5> Rss32;
  1820:   let Inst{20-16} = Rss32{4-0};
  1821:   bits <5> Rtt32;
  1822:   let Inst{12-8} = Rtt32{4-0};
  1823:   bits <5> Rxx32;
  1824:   let Inst{4-0} = Rxx32{4-0};
  1825:   bits <2> Pe4;
  1826:   let Inst{6-5} = Pe4{1-0};
  1827: }
  1828: class Enc_83ee64 : OpcodeHexagon {
  1829:   bits <5> Ii;
  1830:   let Inst{12-8} = Ii{4-0};
  1831:   bits <5> Rs32;
  1832:   let Inst{20-16} = Rs32{4-0};
  1833:   bits <2> Pd4;
  1834:   let Inst{1-0} = Pd4{1-0};
  1835: }
  1836: class Enc_84b2cd : OpcodeHexagon {
  1837:   bits <8> Ii;
  1838:   let Inst{12-7} = Ii{7-2};
  1839:   bits <5> II;
  1840:   let Inst{4-0} = II{4-0};
  1841:   bits <5> Rs32;
  1842:   let Inst{20-16} = Rs32{4-0};
  1843: }
  1844: class Enc_84bff1 : OpcodeHexagon {
  1845:   bits <2> Ii;
  1846:   let Inst{13-13} = Ii{1-1};
  1847:   let Inst{7-7} = Ii{0-0};
  1848:   bits <5> Rs32;
  1849:   let Inst{20-16} = Rs32{4-0};
  1850:   bits <5> Rt32;
  1851:   let Inst{12-8} = Rt32{4-0};
  1852:   bits <5> Rdd32;
  1853:   let Inst{4-0} = Rdd32{4-0};
  1854: }
  1855: class Enc_84d359 : OpcodeHexagon {
  1856:   bits <4> Ii;
  1857:   let Inst{3-0} = Ii{3-0};
  1858:   bits <4> Rs16;
  1859:   let Inst{7-4} = Rs16{3-0};
  1860: }
  1861: class Enc_85bf58 : OpcodeHexagon {
  1862:   bits <7> Ii;
  1863:   let Inst{6-3} = Ii{6-3};
  1864:   bits <5> Rtt32;
  1865:   let Inst{12-8} = Rtt32{4-0};
  1866:   bits <5> Rx32;
  1867:   let Inst{20-16} = Rx32{4-0};
  1868: }
  1869: class Enc_864a5a : OpcodeHexagon {
  1870:   bits <9> Ii;
  1871:   let Inst{12-8} = Ii{8-4};
  1872:   let Inst{4-3} = Ii{3-2};
  1873:   bits <5> Rs32;
  1874:   let Inst{20-16} = Rs32{4-0};
  1875: }
  1876: class Enc_865390 : OpcodeHexagon {
  1877:   bits <3> Ii;
  1878:   let Inst{10-8} = Ii{2-0};
  1879:   bits <2> Pv4;
  1880:   let Inst{12-11} = Pv4{1-0};
  1881:   bits <5> Vs32;
  1882:   let Inst{4-0} = Vs32{4-0};
  1883:   bits <5> Rx32;
  1884:   let Inst{20-16} = Rx32{4-0};
  1885: }
  1886: class Enc_86a14b : OpcodeHexagon {
  1887:   bits <8> Ii;
  1888:   let Inst{7-3} = Ii{7-3};
  1889:   bits <3> Rdd8;
  1890:   let Inst{2-0} = Rdd8{2-0};
  1891: }
  1892: class Enc_87c142 : OpcodeHexagon {
  1893:   bits <7> Ii;
  1894:   let Inst{8-4} = Ii{6-2};
  1895:   bits <4> Rt16;
  1896:   let Inst{3-0} = Rt16{3-0};
  1897: }
  1898: class Enc_88c16c : OpcodeHexagon {
  1899:   bits <5> Rss32;
  1900:   let Inst{20-16} = Rss32{4-0};
  1901:   bits <5> Rtt32;
  1902:   let Inst{12-8} = Rtt32{4-0};
  1903:   bits <5> Rxx32;
  1904:   let Inst{4-0} = Rxx32{4-0};
  1905: }
  1906: class Enc_88d4d9 : OpcodeHexagon {
  1907:   bits <2> Pu4;
  1908:   let Inst{9-8} = Pu4{1-0};
  1909:   bits <5> Rs32;
  1910:   let Inst{20-16} = Rs32{4-0};
  1911: }
  1912: class Enc_890909 : OpcodeHexagon {
  1913:   bits <5> Rs32;
  1914:   let Inst{20-16} = Rs32{4-0};
  1915:   bits <5> Rd32;
  1916:   let Inst{4-0} = Rd32{4-0};
  1917:   bits <2> Pe4;
  1918:   let Inst{6-5} = Pe4{1-0};
  1919: }
  1920: class Enc_895bd9 : OpcodeHexagon {
  1921:   bits <2> Qu4;
  1922:   let Inst{9-8} = Qu4{1-0};
  1923:   bits <5> Rt32;
  1924:   let Inst{20-16} = Rt32{4-0};
  1925:   bits <5> Vx32;
  1926:   let Inst{4-0} = Vx32{4-0};
  1927: }
  1928: class Enc_8b8927 : OpcodeHexagon {
  1929:   bits <5> Rt32;
  1930:   let Inst{20-16} = Rt32{4-0};
  1931:   bits <1> Mu2;
  1932:   let Inst{13-13} = Mu2{0-0};
  1933:   bits <5> Vv32;
  1934:   let Inst{4-0} = Vv32{4-0};
  1935: }
  1936: class Enc_8b8d61 : OpcodeHexagon {
  1937:   bits <6> Ii;
  1938:   let Inst{22-21} = Ii{5-4};
  1939:   let Inst{13-13} = Ii{3-3};
  1940:   let Inst{7-5} = Ii{2-0};
  1941:   bits <5> Rs32;
  1942:   let Inst{20-16} = Rs32{4-0};
  1943:   bits <5> Ru32;
  1944:   let Inst{4-0} = Ru32{4-0};
  1945:   bits <5> Rd32;
  1946:   let Inst{12-8} = Rd32{4-0};
  1947: }
  1948: class Enc_8bcba4 : OpcodeHexagon {
  1949:   bits <6> II;
  1950:   let Inst{5-0} = II{5-0};
  1951:   bits <5> Rt32;
  1952:   let Inst{12-8} = Rt32{4-0};
  1953:   bits <5> Re32;
  1954:   let Inst{20-16} = Re32{4-0};
  1955: }
  1956: class Enc_8c2412 : OpcodeHexagon {
  1957:   bits <2> Ps4;
  1958:   let Inst{6-5} = Ps4{1-0};
  1959:   bits <5> Vu32;
  1960:   let Inst{12-8} = Vu32{4-0};
  1961:   bits <5> Vv32;
  1962:   let Inst{20-16} = Vv32{4-0};
  1963:   bits <5> Vdd32;
  1964:   let Inst{4-0} = Vdd32{4-0};
  1965: }
  1966: class Enc_8c6530 : OpcodeHexagon {
  1967:   bits <5> Rtt32;
  1968:   let Inst{12-8} = Rtt32{4-0};
  1969:   bits <5> Rss32;
  1970:   let Inst{20-16} = Rss32{4-0};
  1971:   bits <2> Pu4;
  1972:   let Inst{6-5} = Pu4{1-0};
  1973:   bits <5> Rdd32;
  1974:   let Inst{4-0} = Rdd32{4-0};
  1975: }
  1976: class Enc_8d8a30 : OpcodeHexagon {
  1977:   bits <4> Ii;
  1978:   let Inst{13-13} = Ii{3-3};
  1979:   let Inst{10-8} = Ii{2-0};
  1980:   bits <2> Pv4;
  1981:   let Inst{12-11} = Pv4{1-0};
  1982:   bits <5> Rt32;
  1983:   let Inst{20-16} = Rt32{4-0};
  1984:   bits <5> Vd32;
  1985:   let Inst{4-0} = Vd32{4-0};
  1986: }
  1987: class Enc_8dbdfe : OpcodeHexagon {
  1988:   bits <8> Ii;
  1989:   let Inst{13-13} = Ii{7-7};
  1990:   let Inst{7-3} = Ii{6-2};
  1991:   bits <2> Pv4;
  1992:   let Inst{1-0} = Pv4{1-0};
  1993:   bits <5> Rs32;
  1994:   let Inst{20-16} = Rs32{4-0};
  1995:   bits <3> Nt8;
  1996:   let Inst{10-8} = Nt8{2-0};
  1997: }
  1998: class Enc_8dbe85 : OpcodeHexagon {
  1999:   bits <1> Mu2;
  2000:   let Inst{13-13} = Mu2{0-0};
```
- EN: It declares types such as Enc_7fa7f6, Enc_800e04, Enc_80296d, Enc_802dc0, ... (30 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_7fa7f6, Enc_800e04, Enc_80296d, Enc_802dc0, Enc_81ac1d, ... (30 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_7fa7f6, Enc_800e04, Enc_80296d, Enc_802dc0, ... (30 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_7fa7f6, Enc_800e04, Enc_80296d, Enc_802dc0, Enc_81ac1d, ... (30 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 2001-2250 / 第 2001-2250 行

```tablegen
  2001:   bits <3> Nt8;
  2002:   let Inst{10-8} = Nt8{2-0};
  2003:   bits <5> Rx32;
  2004:   let Inst{20-16} = Rx32{4-0};
  2005: }
  2006: class Enc_8dec2e : OpcodeHexagon {
  2007:   bits <5> Ii;
  2008:   let Inst{12-8} = Ii{4-0};
  2009:   bits <5> Rss32;
  2010:   let Inst{20-16} = Rss32{4-0};
  2011:   bits <5> Rd32;
  2012:   let Inst{4-0} = Rd32{4-0};
  2013: }
  2014: class Enc_8df4be : OpcodeHexagon {
  2015:   bits <17> Ii;
  2016:   let Inst{26-25} = Ii{16-15};
  2017:   let Inst{20-16} = Ii{14-10};
  2018:   let Inst{13-5} = Ii{9-1};
  2019:   bits <5> Rd32;
  2020:   let Inst{4-0} = Rd32{4-0};
  2021: }
  2022: class Enc_8e583a : OpcodeHexagon {
  2023:   bits <11> Ii;
  2024:   let Inst{21-20} = Ii{10-9};
  2025:   let Inst{7-1} = Ii{8-2};
  2026:   bits <4> Rs16;
  2027:   let Inst{19-16} = Rs16{3-0};
  2028:   bits <0> n1;
  2029: }
  2030: class Enc_8f7633 : OpcodeHexagon {
  2031:   bits <5> Rs32;
  2032:   let Inst{20-16} = Rs32{4-0};
  2033:   bits <7> Sd128;
  2034:   let Inst{6-0} = Sd128{6-0};
  2035: }
  2036: class Enc_90cd8b : OpcodeHexagon {
  2037:   bits <5> Rss32;
  2038:   let Inst{20-16} = Rss32{4-0};
  2039:   bits <5> Rd32;
  2040:   let Inst{4-0} = Rd32{4-0};
  2041: }
  2042: class Enc_91b9fe : OpcodeHexagon {
  2043:   bits <5> Ii;
  2044:   let Inst{6-3} = Ii{4-1};
  2045:   bits <1> Mu2;
  2046:   let Inst{13-13} = Mu2{0-0};
  2047:   bits <3> Nt8;
  2048:   let Inst{10-8} = Nt8{2-0};
  2049:   bits <5> Rx32;
  2050:   let Inst{20-16} = Rx32{4-0};
  2051: }
  2052: class Enc_927852 : OpcodeHexagon {
  2053:   bits <5> Rss32;
  2054:   let Inst{20-16} = Rss32{4-0};
  2055:   bits <5> Rt32;
  2056:   let Inst{12-8} = Rt32{4-0};
  2057:   bits <5> Rdd32;
  2058:   let Inst{4-0} = Rdd32{4-0};
  2059: }
  2060: class Enc_928ca1 : OpcodeHexagon {
  2061:   bits <1> Mu2;
  2062:   let Inst{13-13} = Mu2{0-0};
  2063:   bits <5> Rtt32;
  2064:   let Inst{12-8} = Rtt32{4-0};
  2065:   bits <5> Rx32;
  2066:   let Inst{20-16} = Rx32{4-0};
  2067: }
  2068: class Enc_935d9b : OpcodeHexagon {
  2069:   bits <5> Ii;
  2070:   let Inst{6-3} = Ii{4-1};
  2071:   bits <1> Mu2;
  2072:   let Inst{13-13} = Mu2{0-0};
  2073:   bits <5> Rt32;
  2074:   let Inst{12-8} = Rt32{4-0};
  2075:   bits <5> Rx32;
  2076:   let Inst{20-16} = Rx32{4-0};
  2077: }
  2078: class Enc_93af4c : OpcodeHexagon {
  2079:   bits <7> Ii;
  2080:   let Inst{10-4} = Ii{6-0};
  2081:   bits <4> Rx16;
  2082:   let Inst{3-0} = Rx16{3-0};
  2083: }
  2084: class Enc_95441f : OpcodeHexagon {
  2085:   bits <5> Vu32;
  2086:   let Inst{12-8} = Vu32{4-0};
  2087:   bits <5> Vv32;
  2088:   let Inst{20-16} = Vv32{4-0};
  2089:   bits <2> Qd4;
  2090:   let Inst{1-0} = Qd4{1-0};
  2091: }
  2092: class Enc_96ce4f : OpcodeHexagon {
  2093:   bits <4> Ii;
  2094:   let Inst{6-3} = Ii{3-0};
  2095:   bits <1> Mu2;
  2096:   let Inst{13-13} = Mu2{0-0};
  2097:   bits <3> Nt8;
  2098:   let Inst{10-8} = Nt8{2-0};
  2099:   bits <5> Rx32;
  2100:   let Inst{20-16} = Rx32{4-0};
  2101: }
  2102: class Enc_97d666 : OpcodeHexagon {
  2103:   bits <4> Rs16;
  2104:   let Inst{7-4} = Rs16{3-0};
  2105:   bits <4> Rd16;
  2106:   let Inst{3-0} = Rd16{3-0};
  2107: }
  2108: class Enc_989021 : OpcodeHexagon {
  2109:   bits <5> Rt32;
  2110:   let Inst{20-16} = Rt32{4-0};
  2111:   bits <5> Vy32;
  2112:   let Inst{12-8} = Vy32{4-0};
  2113:   bits <5> Vx32;
  2114:   let Inst{4-0} = Vx32{4-0};
  2115: }
  2116: class Enc_98c0b8 : OpcodeHexagon {
  2117:   bits <2> Ii;
  2118:   let Inst{13-13} = Ii{1-1};
  2119:   let Inst{7-7} = Ii{0-0};
  2120:   bits <2> Pv4;
  2121:   let Inst{6-5} = Pv4{1-0};
  2122:   bits <5> Rs32;
  2123:   let Inst{20-16} = Rs32{4-0};
  2124:   bits <5> Rt32;
  2125:   let Inst{12-8} = Rt32{4-0};
  2126:   bits <5> Rdd32;
  2127:   let Inst{4-0} = Rdd32{4-0};
  2128: }
  2129: class Enc_9a33d5 : OpcodeHexagon {
  2130:   bits <7> Ii;
  2131:   let Inst{6-3} = Ii{6-3};
  2132:   bits <2> Pv4;
  2133:   let Inst{1-0} = Pv4{1-0};
  2134:   bits <5> Rtt32;
  2135:   let Inst{12-8} = Rtt32{4-0};
  2136:   bits <5> Rx32;
  2137:   let Inst{20-16} = Rx32{4-0};
  2138: }
  2139: class Enc_9ac432 : OpcodeHexagon {
  2140:   bits <2> Ps4;
  2141:   let Inst{17-16} = Ps4{1-0};
  2142:   bits <2> Pt4;
  2143:   let Inst{9-8} = Pt4{1-0};
  2144:   bits <2> Pu4;
  2145:   let Inst{7-6} = Pu4{1-0};
  2146:   bits <2> Pd4;
  2147:   let Inst{1-0} = Pd4{1-0};
  2148: }
  2149: class Enc_9b0bc1 : OpcodeHexagon {
  2150:   bits <2> Pu4;
  2151:   let Inst{6-5} = Pu4{1-0};
  2152:   bits <5> Rt32;
  2153:   let Inst{12-8} = Rt32{4-0};
  2154:   bits <5> Rs32;
  2155:   let Inst{20-16} = Rs32{4-0};
  2156:   bits <5> Rd32;
  2157:   let Inst{4-0} = Rd32{4-0};
  2158: }
  2159: class Enc_9be1de : OpcodeHexagon {
  2160:   bits <2> Qs4;
  2161:   let Inst{6-5} = Qs4{1-0};
  2162:   bits <5> Rt32;
  2163:   let Inst{20-16} = Rt32{4-0};
  2164:   bits <1> Mu2;
  2165:   let Inst{13-13} = Mu2{0-0};
  2166:   bits <5> Vv32;
  2167:   let Inst{12-8} = Vv32{4-0};
  2168:   bits <5> Vw32;
  2169:   let Inst{4-0} = Vw32{4-0};
  2170: }
  2171: class Enc_9cdba7 : OpcodeHexagon {
  2172:   bits <8> Ii;
  2173:   let Inst{12-5} = Ii{7-0};
  2174:   bits <5> Rs32;
  2175:   let Inst{20-16} = Rs32{4-0};
  2176:   bits <5> Rdd32;
  2177:   let Inst{4-0} = Rdd32{4-0};
  2178: }
  2179: class Enc_9d1247 : OpcodeHexagon {
  2180:   bits <7> Ii;
  2181:   let Inst{8-5} = Ii{6-3};
  2182:   bits <2> Pt4;
  2183:   let Inst{10-9} = Pt4{1-0};
  2184:   bits <5> Rdd32;
  2185:   let Inst{4-0} = Rdd32{4-0};
  2186:   bits <5> Rx32;
  2187:   let Inst{20-16} = Rx32{4-0};
  2188: }
  2189: class Enc_9e2e1c : OpcodeHexagon {
  2190:   bits <5> Ii;
  2191:   let Inst{8-5} = Ii{4-1};
  2192:   bits <1> Mu2;
  2193:   let Inst{13-13} = Mu2{0-0};
  2194:   bits <5> Ryy32;
  2195:   let Inst{4-0} = Ryy32{4-0};
  2196:   bits <5> Rx32;
  2197:   let Inst{20-16} = Rx32{4-0};
  2198: }
  2199: class Enc_9e4c3f : OpcodeHexagon {
  2200:   bits <6> II;
  2201:   let Inst{13-8} = II{5-0};
  2202:   bits <11> Ii;
  2203:   let Inst{21-20} = Ii{10-9};
  2204:   let Inst{7-1} = Ii{8-2};
  2205:   bits <4> Rd16;
  2206:   let Inst{19-16} = Rd16{3-0};
  2207: }
  2208: class Enc_9e9047 : OpcodeHexagon {
  2209:   bits <2> Pt4;
  2210:   let Inst{9-8} = Pt4{1-0};
  2211:   bits <5> Rs32;
  2212:   let Inst{20-16} = Rs32{4-0};
  2213: }
  2214: class Enc_9ea4cf : OpcodeHexagon {
  2215:   bits <2> Ii;
  2216:   let Inst{13-13} = Ii{1-1};
  2217:   let Inst{6-6} = Ii{0-0};
  2218:   bits <6> II;
  2219:   let Inst{5-0} = II{5-0};
  2220:   bits <5> Ru32;
  2221:   let Inst{20-16} = Ru32{4-0};
  2222:   bits <5> Rt32;
  2223:   let Inst{12-8} = Rt32{4-0};
  2224: }
  2225: class Enc_9fae8a : OpcodeHexagon {
  2226:   bits <6> Ii;
  2227:   let Inst{13-8} = Ii{5-0};
  2228:   bits <5> Rs32;
  2229:   let Inst{20-16} = Rs32{4-0};
  2230:   bits <5> Rd32;
  2231:   let Inst{4-0} = Rd32{4-0};
  2232: }
  2233: class Enc_a05677 : OpcodeHexagon {
  2234:   bits <5> Ii;
  2235:   let Inst{12-8} = Ii{4-0};
  2236:   bits <5> Rs32;
  2237:   let Inst{20-16} = Rs32{4-0};
  2238:   bits <5> Rd32;
  2239:   let Inst{4-0} = Rd32{4-0};
  2240: }
  2241: class Enc_a1640c : OpcodeHexagon {
  2242:   bits <6> Ii;
  2243:   let Inst{13-8} = Ii{5-0};
  2244:   bits <5> Rss32;
  2245:   let Inst{20-16} = Rss32{4-0};
  2246:   bits <5> Rd32;
  2247:   let Inst{4-0} = Rd32{4-0};
  2248: }
  2249: class Enc_a198f6 : OpcodeHexagon {
  2250:   bits <7> Ii;
```
- EN: It declares types such as Enc_8dec2e, Enc_8df4be, Enc_8e583a, Enc_8f7633, ... (29 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_8dec2e, Enc_8df4be, Enc_8e583a, Enc_8f7633, Enc_90cd8b, ... (29 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_8dec2e, Enc_8df4be, Enc_8e583a, Enc_8f7633, ... (29 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_8dec2e, Enc_8df4be, Enc_8e583a, Enc_8f7633, Enc_90cd8b, ... (29 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 2251-2500 / 第 2251-2500 行

```tablegen
  2251:   let Inst{10-5} = Ii{6-1};
  2252:   bits <2> Pt4;
  2253:   let Inst{12-11} = Pt4{1-0};
  2254:   bits <5> Rs32;
  2255:   let Inst{20-16} = Rs32{4-0};
  2256:   bits <5> Rd32;
  2257:   let Inst{4-0} = Rd32{4-0};
  2258: }
  2259: class Enc_a1e29d : OpcodeHexagon {
  2260:   bits <5> Ii;
  2261:   let Inst{12-8} = Ii{4-0};
  2262:   bits <5> II;
  2263:   let Inst{22-21} = II{4-3};
  2264:   let Inst{7-5} = II{2-0};
  2265:   bits <5> Rs32;
  2266:   let Inst{20-16} = Rs32{4-0};
  2267:   bits <5> Rx32;
  2268:   let Inst{4-0} = Rx32{4-0};
  2269: }
  2270: class Enc_a21d47 : OpcodeHexagon {
  2271:   bits <6> Ii;
  2272:   let Inst{10-5} = Ii{5-0};
  2273:   bits <2> Pt4;
  2274:   let Inst{12-11} = Pt4{1-0};
  2275:   bits <5> Rs32;
  2276:   let Inst{20-16} = Rs32{4-0};
  2277:   bits <5> Rd32;
  2278:   let Inst{4-0} = Rd32{4-0};
  2279: }
  2280: class Enc_a255dc : OpcodeHexagon {
  2281:   bits <3> Ii;
  2282:   let Inst{10-8} = Ii{2-0};
  2283:   bits <5> Vd32;
  2284:   let Inst{4-0} = Vd32{4-0};
  2285:   bits <5> Rx32;
  2286:   let Inst{20-16} = Rx32{4-0};
  2287: }
  2288: class Enc_a27588 : OpcodeHexagon {
  2289:   bits <11> Ii;
  2290:   let Inst{26-25} = Ii{10-9};
  2291:   let Inst{13-5} = Ii{8-0};
  2292:   bits <5> Rs32;
  2293:   let Inst{20-16} = Rs32{4-0};
  2294:   bits <5> Ryy32;
  2295:   let Inst{4-0} = Ryy32{4-0};
  2296: }
  2297: class Enc_a30110 : OpcodeHexagon {
  2298:   bits <5> Vu32;
  2299:   let Inst{12-8} = Vu32{4-0};
  2300:   bits <5> Vv32;
  2301:   let Inst{23-19} = Vv32{4-0};
  2302:   bits <3> Rt8;
  2303:   let Inst{18-16} = Rt8{2-0};
  2304:   bits <5> Vd32;
  2305:   let Inst{4-0} = Vd32{4-0};
  2306: }
  2307: class Enc_a33d04 : OpcodeHexagon {
  2308:   bits <5> Vuu32;
  2309:   let Inst{12-8} = Vuu32{4-0};
  2310:   bits <5> Vd32;
  2311:   let Inst{4-0} = Vd32{4-0};
  2312: }
  2313: class Enc_a42857 : OpcodeHexagon {
  2314:   bits <11> Ii;
  2315:   let Inst{21-20} = Ii{10-9};
  2316:   let Inst{7-1} = Ii{8-2};
  2317:   bits <4> Rs16;
  2318:   let Inst{19-16} = Rs16{3-0};
  2319:   bits <0> n1;
  2320: }
  2321: class Enc_a4ef14 : OpcodeHexagon {
  2322:   bits <5> Rd32;
  2323:   let Inst{4-0} = Rd32{4-0};
  2324: }
  2325: class Enc_a51a9a : OpcodeHexagon {
  2326:   bits <8> Ii;
  2327:   let Inst{12-8} = Ii{7-3};
  2328:   let Inst{4-2} = Ii{2-0};
  2329: }
  2330: class Enc_a56825 : OpcodeHexagon {
  2331:   bits <5> Rss32;
  2332:   let Inst{20-16} = Rss32{4-0};
  2333:   bits <5> Rtt32;
  2334:   let Inst{12-8} = Rtt32{4-0};
  2335:   bits <5> Rdd32;
  2336:   let Inst{4-0} = Rdd32{4-0};
  2337: }
  2338: class Enc_a568d4 : OpcodeHexagon {
  2339:   bits <5> Rt32;
  2340:   let Inst{12-8} = Rt32{4-0};
  2341:   bits <5> Rs32;
  2342:   let Inst{20-16} = Rs32{4-0};
  2343:   bits <5> Rx32;
  2344:   let Inst{4-0} = Rx32{4-0};
  2345: }
  2346: class Enc_a5ed8a : OpcodeHexagon {
  2347:   bits <5> Rt32;
  2348:   let Inst{20-16} = Rt32{4-0};
  2349:   bits <5> Vd32;
  2350:   let Inst{4-0} = Vd32{4-0};
  2351: }
  2352: class Enc_a641d0 : OpcodeHexagon {
  2353:   bits <5> Rt32;
  2354:   let Inst{20-16} = Rt32{4-0};
  2355:   bits <1> Mu2;
  2356:   let Inst{13-13} = Mu2{0-0};
  2357:   bits <5> Vvv32;
  2358:   let Inst{12-8} = Vvv32{4-0};
  2359:   bits <5> Vw32;
  2360:   let Inst{4-0} = Vw32{4-0};
  2361: }
  2362: class Enc_a6853f : OpcodeHexagon {
  2363:   bits <11> Ii;
  2364:   let Inst{21-20} = Ii{10-9};
  2365:   let Inst{7-1} = Ii{8-2};
  2366:   bits <3> Ns8;
  2367:   let Inst{18-16} = Ns8{2-0};
  2368:   bits <0> n1;
  2369: }
  2370: class Enc_a6ce9c : OpcodeHexagon {
  2371:   bits <6> Ii;
  2372:   let Inst{3-0} = Ii{5-2};
  2373:   bits <4> Rs16;
  2374:   let Inst{7-4} = Rs16{3-0};
  2375: }
  2376: class Enc_a705fc : OpcodeHexagon {
  2377:   bits <5> Rss32;
  2378:   let Inst{20-16} = Rss32{4-0};
  2379:   bits <7> Sdd128;
  2380:   let Inst{6-0} = Sdd128{6-0};
  2381: }
  2382: class Enc_a7341a : OpcodeHexagon {
  2383:   bits <5> Vu32;
  2384:   let Inst{12-8} = Vu32{4-0};
  2385:   bits <5> Vv32;
  2386:   let Inst{20-16} = Vv32{4-0};
  2387:   bits <5> Vx32;
  2388:   let Inst{4-0} = Vx32{4-0};
  2389: }
  2390: class Enc_a75aa6 : OpcodeHexagon {
  2391:   bits <5> Rs32;
  2392:   let Inst{20-16} = Rs32{4-0};
  2393:   bits <5> Rt32;
  2394:   let Inst{12-8} = Rt32{4-0};
  2395:   bits <1> Mu2;
  2396:   let Inst{13-13} = Mu2{0-0};
  2397: }
  2398: class Enc_a7b8e8 : OpcodeHexagon {
  2399:   bits <6> Ii;
  2400:   let Inst{22-21} = Ii{5-4};
  2401:   let Inst{13-13} = Ii{3-3};
  2402:   let Inst{7-5} = Ii{2-0};
  2403:   bits <5> Rs32;
  2404:   let Inst{20-16} = Rs32{4-0};
  2405:   bits <5> Rt32;
  2406:   let Inst{12-8} = Rt32{4-0};
  2407:   bits <5> Rd32;
  2408:   let Inst{4-0} = Rd32{4-0};
  2409: }
  2410: class Enc_a803e0 : OpcodeHexagon {
  2411:   bits <7> Ii;
  2412:   let Inst{12-7} = Ii{6-1};
  2413:   bits <8> II;
  2414:   let Inst{13-13} = II{7-7};
  2415:   let Inst{6-0} = II{6-0};
  2416:   bits <5> Rs32;
  2417:   let Inst{20-16} = Rs32{4-0};
  2418: }
  2419: class Enc_a90628 : OpcodeHexagon {
  2420:   bits <2> Qv4;
  2421:   let Inst{23-22} = Qv4{1-0};
  2422:   bits <5> Vu32;
  2423:   let Inst{12-8} = Vu32{4-0};
  2424:   bits <5> Vx32;
  2425:   let Inst{4-0} = Vx32{4-0};
  2426: }
  2427: class Enc_a94f3b : OpcodeHexagon {
  2428:   bits <5> Rs32;
  2429:   let Inst{20-16} = Rs32{4-0};
  2430:   bits <5> Rt32;
  2431:   let Inst{12-8} = Rt32{4-0};
  2432:   bits <5> Rd32;
  2433:   let Inst{4-0} = Rd32{4-0};
  2434:   bits <2> Pe4;
  2435:   let Inst{6-5} = Pe4{1-0};
  2436: }
  2437: class Enc_aad80c : OpcodeHexagon {
  2438:   bits <5> Vuu32;
  2439:   let Inst{12-8} = Vuu32{4-0};
  2440:   bits <5> Rt32;
  2441:   let Inst{20-16} = Rt32{4-0};
  2442:   bits <5> Vdd32;
  2443:   let Inst{4-0} = Vdd32{4-0};
  2444: }
  2445: class Enc_acd6ed : OpcodeHexagon {
  2446:   bits <9> Ii;
  2447:   let Inst{10-5} = Ii{8-3};
  2448:   bits <2> Pt4;
  2449:   let Inst{12-11} = Pt4{1-0};
  2450:   bits <5> Rs32;
  2451:   let Inst{20-16} = Rs32{4-0};
  2452:   bits <5> Rdd32;
  2453:   let Inst{4-0} = Rdd32{4-0};
  2454: }
  2455: class Enc_ad1831 : OpcodeHexagon {
  2456:   bits <16> Ii;
  2457:   let Inst{26-25} = Ii{15-14};
  2458:   let Inst{20-16} = Ii{13-9};
  2459:   let Inst{13-13} = Ii{8-8};
  2460:   let Inst{7-0} = Ii{7-0};
  2461:   bits <3> Nt8;
  2462:   let Inst{10-8} = Nt8{2-0};
  2463: }
  2464: class Enc_ad1c74 : OpcodeHexagon {
  2465:   bits <11> Ii;
  2466:   let Inst{21-20} = Ii{10-9};
  2467:   let Inst{7-1} = Ii{8-2};
  2468:   bits <4> Rs16;
  2469:   let Inst{19-16} = Rs16{3-0};
  2470: }
  2471: class Enc_ad9bef : OpcodeHexagon {
  2472:   bits <5> Vu32;
  2473:   let Inst{12-8} = Vu32{4-0};
  2474:   bits <5> Rtt32;
  2475:   let Inst{20-16} = Rtt32{4-0};
  2476:   bits <5> Vxx32;
  2477:   let Inst{4-0} = Vxx32{4-0};
  2478: }
  2479: class Enc_adf111 : OpcodeHexagon {
  2480:   bits <5> Vu32;
  2481:   let Inst{12-8} = Vu32{4-0};
  2482:   bits <5> Rt32;
  2483:   let Inst{20-16} = Rt32{4-0};
  2484:   bits <2> Qx4;
  2485:   let Inst{1-0} = Qx4{1-0};
  2486: }
  2487: class Enc_b00112 : OpcodeHexagon {
  2488:   bits <5> Rss32;
  2489:   let Inst{20-16} = Rss32{4-0};
  2490:   bits <5> Rtt32;
  2491:   let Inst{12-8} = Rtt32{4-0};
  2492: }
  2493: class Enc_b025d6 : OpcodeHexagon {
  2494:   bits <3> Ii;
  2495:   let Inst{10-8} = Ii{2-0};
  2496:   bits <5> Vss32;
  2497:   let Inst{4-0} = Vss32{4-0};
  2498:   bits <5> Rx32;
  2499:   let Inst{20-16} = Rx32{4-0};
  2500: }
```
- EN: It declares types such as Enc_a1e29d, Enc_a21d47, Enc_a255dc, Enc_a27588, ... (30 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_a1e29d, Enc_a21d47, Enc_a255dc, Enc_a27588, Enc_a30110, ... (30 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_a1e29d, Enc_a21d47, Enc_a255dc, Enc_a27588, ... (30 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_a1e29d, Enc_a21d47, Enc_a255dc, Enc_a27588, Enc_a30110, ... (30 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 2501-2750 / 第 2501-2750 行

```tablegen
  2501: class Enc_b05839 : OpcodeHexagon {
  2502:   bits <7> Ii;
  2503:   let Inst{8-5} = Ii{6-3};
  2504:   bits <1> Mu2;
  2505:   let Inst{13-13} = Mu2{0-0};
  2506:   bits <5> Rdd32;
  2507:   let Inst{4-0} = Rdd32{4-0};
  2508:   bits <5> Rx32;
  2509:   let Inst{20-16} = Rx32{4-0};
  2510: }
  2511: class Enc_b087ac : OpcodeHexagon {
  2512:   bits <5> Vu32;
  2513:   let Inst{12-8} = Vu32{4-0};
  2514:   bits <5> Rt32;
  2515:   let Inst{20-16} = Rt32{4-0};
  2516:   bits <5> Vd32;
  2517:   let Inst{4-0} = Vd32{4-0};
  2518: }
  2519: class Enc_b0e9d8 : OpcodeHexagon {
  2520:   bits <10> Ii;
  2521:   let Inst{21-21} = Ii{9-9};
  2522:   let Inst{13-5} = Ii{8-0};
  2523:   bits <5> Rs32;
  2524:   let Inst{20-16} = Rs32{4-0};
  2525:   bits <5> Rx32;
  2526:   let Inst{4-0} = Rx32{4-0};
  2527: }
  2528: class Enc_b15941 : OpcodeHexagon {
  2529:   bits <4> Ii;
  2530:   let Inst{6-3} = Ii{3-0};
  2531:   bits <1> Mu2;
  2532:   let Inst{13-13} = Mu2{0-0};
  2533:   bits <5> Rt32;
  2534:   let Inst{12-8} = Rt32{4-0};
  2535:   bits <5> Rx32;
  2536:   let Inst{20-16} = Rx32{4-0};
  2537: }
  2538: class Enc_b1e1fb : OpcodeHexagon {
  2539:   bits <11> Ii;
  2540:   let Inst{21-20} = Ii{10-9};
  2541:   let Inst{7-1} = Ii{8-2};
  2542:   bits <4> Rs16;
  2543:   let Inst{19-16} = Rs16{3-0};
  2544:   bits <0> n1;
  2545: }
  2546: class Enc_b388cf : OpcodeHexagon {
  2547:   bits <5> Ii;
  2548:   let Inst{12-8} = Ii{4-0};
  2549:   bits <5> II;
  2550:   let Inst{22-21} = II{4-3};
  2551:   let Inst{7-5} = II{2-0};
  2552:   bits <5> Rs32;
  2553:   let Inst{20-16} = Rs32{4-0};
  2554:   bits <5> Rd32;
  2555:   let Inst{4-0} = Rd32{4-0};
  2556: }
  2557: class Enc_b38ffc : OpcodeHexagon {
  2558:   bits <4> Ii;
  2559:   let Inst{11-8} = Ii{3-0};
  2560:   bits <4> Rs16;
  2561:   let Inst{7-4} = Rs16{3-0};
  2562:   bits <4> Rt16;
  2563:   let Inst{3-0} = Rt16{3-0};
  2564: }
  2565: class Enc_b43b67 : OpcodeHexagon {
  2566:   bits <5> Vu32;
  2567:   let Inst{12-8} = Vu32{4-0};
  2568:   bits <5> Vv32;
  2569:   let Inst{20-16} = Vv32{4-0};
  2570:   bits <5> Vd32;
  2571:   let Inst{4-0} = Vd32{4-0};
  2572:   bits <2> Qx4;
  2573:   let Inst{6-5} = Qx4{1-0};
  2574: }
  2575: class Enc_b4e6cf : OpcodeHexagon {
  2576:   bits <10> Ii;
  2577:   let Inst{21-21} = Ii{9-9};
  2578:   let Inst{13-5} = Ii{8-0};
  2579:   bits <5> Ru32;
  2580:   let Inst{4-0} = Ru32{4-0};
  2581:   bits <5> Rx32;
  2582:   let Inst{20-16} = Rx32{4-0};
  2583: }
  2584: class Enc_b62ef7 : OpcodeHexagon {
  2585:   bits <3> Ii;
  2586:   let Inst{10-8} = Ii{2-0};
  2587:   bits <5> Vs32;
  2588:   let Inst{4-0} = Vs32{4-0};
  2589:   bits <5> Rx32;
  2590:   let Inst{20-16} = Rx32{4-0};
  2591: }
  2592: class Enc_b72622 : OpcodeHexagon {
  2593:   bits <2> Ii;
  2594:   let Inst{13-13} = Ii{1-1};
  2595:   let Inst{5-5} = Ii{0-0};
  2596:   bits <5> Rss32;
  2597:   let Inst{20-16} = Rss32{4-0};
  2598:   bits <5> Rt32;
  2599:   let Inst{12-8} = Rt32{4-0};
  2600:   bits <5> Rxx32;
  2601:   let Inst{4-0} = Rxx32{4-0};
  2602: }
  2603: class Enc_b78edd : OpcodeHexagon {
  2604:   bits <11> Ii;
  2605:   let Inst{21-20} = Ii{10-9};
  2606:   let Inst{7-1} = Ii{8-2};
  2607:   bits <4> Rs16;
  2608:   let Inst{19-16} = Rs16{3-0};
  2609:   bits <0> n1;
  2610: }
  2611: class Enc_b7fad3 : OpcodeHexagon {
  2612:   bits <2> Pv4;
  2613:   let Inst{9-8} = Pv4{1-0};
  2614:   bits <5> Rs32;
  2615:   let Inst{20-16} = Rs32{4-0};
  2616:   bits <5> Rdd32;
  2617:   let Inst{4-0} = Rdd32{4-0};
  2618: }
  2619: class Enc_b8309d : OpcodeHexagon {
  2620:   bits <9> Ii;
  2621:   let Inst{8-3} = Ii{8-3};
  2622:   bits <3> Rtt8;
  2623:   let Inst{2-0} = Rtt8{2-0};
  2624: }
  2625: class Enc_b84c4c : OpcodeHexagon {
  2626:   bits <6> Ii;
  2627:   let Inst{13-8} = Ii{5-0};
  2628:   bits <6> II;
  2629:   let Inst{23-21} = II{5-3};
  2630:   let Inst{7-5} = II{2-0};
  2631:   bits <5> Rss32;
  2632:   let Inst{20-16} = Rss32{4-0};
  2633:   bits <5> Rdd32;
  2634:   let Inst{4-0} = Rdd32{4-0};
  2635: }
  2636: class Enc_b886fd : OpcodeHexagon {
  2637:   bits <5> Ii;
  2638:   let Inst{6-3} = Ii{4-1};
  2639:   bits <2> Pv4;
  2640:   let Inst{1-0} = Pv4{1-0};
  2641:   bits <5> Rt32;
  2642:   let Inst{12-8} = Rt32{4-0};
  2643:   bits <5> Rx32;
  2644:   let Inst{20-16} = Rx32{4-0};
  2645: }
  2646: class Enc_b8c967 : OpcodeHexagon {
  2647:   bits <8> Ii;
  2648:   let Inst{12-5} = Ii{7-0};
  2649:   bits <5> Rs32;
  2650:   let Inst{20-16} = Rs32{4-0};
  2651:   bits <5> Rd32;
  2652:   let Inst{4-0} = Rd32{4-0};
  2653: }
  2654: class Enc_b909d2 : OpcodeHexagon {
  2655:   bits <11> Ii;
  2656:   let Inst{21-20} = Ii{10-9};
  2657:   let Inst{7-1} = Ii{8-2};
  2658:   bits <4> Rs16;
  2659:   let Inst{19-16} = Rs16{3-0};
  2660:   bits <0> n1;
  2661: }
  2662: class Enc_b91167 : OpcodeHexagon {
  2663:   bits <2> Ii;
  2664:   let Inst{6-5} = Ii{1-0};
  2665:   bits <5> Vuu32;
  2666:   let Inst{12-8} = Vuu32{4-0};
  2667:   bits <5> Vvv32;
  2668:   let Inst{20-16} = Vvv32{4-0};
  2669:   bits <5> Vdd32;
  2670:   let Inst{4-0} = Vdd32{4-0};
  2671: }
  2672: class Enc_b97f71 : OpcodeHexagon {
  2673:   bits <6> Ii;
  2674:   let Inst{8-5} = Ii{5-2};
  2675:   bits <2> Pt4;
  2676:   let Inst{10-9} = Pt4{1-0};
  2677:   bits <5> Rd32;
  2678:   let Inst{4-0} = Rd32{4-0};
  2679:   bits <5> Rx32;
  2680:   let Inst{20-16} = Rx32{4-0};
  2681: }
  2682: class Enc_b98b95 : OpcodeHexagon {
  2683:   bits <4> Ii;
  2684:   let Inst{13-13} = Ii{3-3};
  2685:   let Inst{10-8} = Ii{2-0};
  2686:   bits <5> Rt32;
  2687:   let Inst{20-16} = Rt32{4-0};
  2688:   bits <5> Vss32;
  2689:   let Inst{4-0} = Vss32{4-0};
  2690: }
  2691: class Enc_b9c5fb : OpcodeHexagon {
  2692:   bits <5> Rss32;
  2693:   let Inst{20-16} = Rss32{4-0};
  2694:   bits <5> Rdd32;
  2695:   let Inst{4-0} = Rdd32{4-0};
  2696: }
  2697: class Enc_bc03e5 : OpcodeHexagon {
  2698:   bits <17> Ii;
  2699:   let Inst{26-25} = Ii{16-15};
  2700:   let Inst{20-16} = Ii{14-10};
  2701:   let Inst{13-13} = Ii{9-9};
  2702:   let Inst{7-0} = Ii{8-1};
  2703:   bits <3> Nt8;
  2704:   let Inst{10-8} = Nt8{2-0};
  2705: }
  2706: class Enc_bd0b33 : OpcodeHexagon {
  2707:   bits <10> Ii;
  2708:   let Inst{21-21} = Ii{9-9};
  2709:   let Inst{13-5} = Ii{8-0};
  2710:   bits <5> Rs32;
  2711:   let Inst{20-16} = Rs32{4-0};
  2712:   bits <2> Pd4;
  2713:   let Inst{1-0} = Pd4{1-0};
  2714: }
  2715: class Enc_bd1cbc : OpcodeHexagon {
  2716:   bits <5> Ii;
  2717:   let Inst{8-5} = Ii{4-1};
  2718:   bits <5> Ryy32;
  2719:   let Inst{4-0} = Ryy32{4-0};
  2720:   bits <5> Rx32;
  2721:   let Inst{20-16} = Rx32{4-0};
  2722: }
  2723: class Enc_bd6011 : OpcodeHexagon {
  2724:   bits <5> Rt32;
  2725:   let Inst{12-8} = Rt32{4-0};
  2726:   bits <5> Rs32;
  2727:   let Inst{20-16} = Rs32{4-0};
  2728:   bits <5> Rd32;
  2729:   let Inst{4-0} = Rd32{4-0};
  2730: }
  2731: class Enc_bd811a : OpcodeHexagon {
  2732:   bits <5> Rs32;
  2733:   let Inst{20-16} = Rs32{4-0};
  2734:   bits <5> Cd32;
  2735:   let Inst{4-0} = Cd32{4-0};
  2736: }
  2737: class Enc_bddee3 : OpcodeHexagon {
  2738:   bits <5> Vu32;
  2739:   let Inst{12-8} = Vu32{4-0};
  2740:   bits <5> Vyyyy32;
  2741:   let Inst{4-0} = Vyyyy32{4-0};
  2742:   bits <3> Rx8;
  2743:   let Inst{18-16} = Rx8{2-0};
  2744: }
  2745: class Enc_be32a5 : OpcodeHexagon {
  2746:   bits <5> Rs32;
  2747:   let Inst{20-16} = Rs32{4-0};
  2748:   bits <5> Rt32;
  2749:   let Inst{12-8} = Rt32{4-0};
  2750:   bits <5> Rdd32;
```
- EN: It declares types such as Enc_b05839, Enc_b087ac, Enc_b0e9d8, Enc_b15941, ... (29 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_b05839, Enc_b087ac, Enc_b0e9d8, Enc_b15941, Enc_b1e1fb, ... (29 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_b05839, Enc_b087ac, Enc_b0e9d8, Enc_b15941, ... (29 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_b05839, Enc_b087ac, Enc_b0e9d8, Enc_b15941, Enc_b1e1fb, ... (29 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 2751-3000 / 第 2751-3000 行

```tablegen
  2751:   let Inst{4-0} = Rdd32{4-0};
  2752: }
  2753: class Enc_bea5da : OpcodeHexagon {
  2754:   bits <10> Ii;
  2755:   let Inst{17-16} = Ii{9-8};
  2756:   let Inst{12-8} = Ii{7-3};
  2757:   let Inst{4-2} = Ii{2-0};
  2758: }
  2759: class Enc_bfbf03 : OpcodeHexagon {
  2760:   bits <2> Qs4;
  2761:   let Inst{9-8} = Qs4{1-0};
  2762:   bits <2> Qd4;
  2763:   let Inst{1-0} = Qd4{1-0};
  2764: }
  2765: class Enc_c0cdde : OpcodeHexagon {
  2766:   bits <9> Ii;
  2767:   let Inst{13-5} = Ii{8-0};
  2768:   bits <5> Rs32;
  2769:   let Inst{20-16} = Rs32{4-0};
  2770:   bits <2> Pd4;
  2771:   let Inst{1-0} = Pd4{1-0};
  2772: }
  2773: class Enc_c175d0 : OpcodeHexagon {
  2774:   bits <4> Ii;
  2775:   let Inst{11-8} = Ii{3-0};
  2776:   bits <4> Rs16;
  2777:   let Inst{7-4} = Rs16{3-0};
  2778:   bits <4> Rd16;
  2779:   let Inst{3-0} = Rd16{3-0};
  2780: }
  2781: class Enc_c1d806 : OpcodeHexagon {
  2782:   bits <5> Vu32;
  2783:   let Inst{12-8} = Vu32{4-0};
  2784:   bits <5> Vv32;
  2785:   let Inst{20-16} = Vv32{4-0};
  2786:   bits <5> Vd32;
  2787:   let Inst{4-0} = Vd32{4-0};
  2788:   bits <2> Qe4;
  2789:   let Inst{6-5} = Qe4{1-0};
  2790: }
  2791: class Enc_c2b48e : OpcodeHexagon {
  2792:   bits <5> Rs32;
  2793:   let Inst{20-16} = Rs32{4-0};
  2794:   bits <5> Rt32;
  2795:   let Inst{12-8} = Rt32{4-0};
  2796:   bits <2> Pd4;
  2797:   let Inst{1-0} = Pd4{1-0};
  2798: }
  2799: class Enc_c31910 : OpcodeHexagon {
  2800:   bits <8> Ii;
  2801:   let Inst{23-21} = Ii{7-5};
  2802:   let Inst{13-13} = Ii{4-4};
  2803:   let Inst{7-5} = Ii{3-1};
  2804:   let Inst{3-3} = Ii{0-0};
  2805:   bits <5> II;
  2806:   let Inst{12-8} = II{4-0};
  2807:   bits <5> Rx32;
  2808:   let Inst{20-16} = Rx32{4-0};
  2809: }
  2810: class Enc_c4dc92 : OpcodeHexagon {
  2811:   bits <2> Qv4;
  2812:   let Inst{23-22} = Qv4{1-0};
  2813:   bits <5> Vu32;
  2814:   let Inst{12-8} = Vu32{4-0};
  2815:   bits <5> Vd32;
  2816:   let Inst{4-0} = Vd32{4-0};
  2817: }
  2818: class Enc_c6220b : OpcodeHexagon {
  2819:   bits <2> Ii;
  2820:   let Inst{13-13} = Ii{1-1};
  2821:   let Inst{7-7} = Ii{0-0};
  2822:   bits <5> Rs32;
  2823:   let Inst{20-16} = Rs32{4-0};
  2824:   bits <5> Ru32;
  2825:   let Inst{12-8} = Ru32{4-0};
  2826:   bits <3> Nt8;
  2827:   let Inst{2-0} = Nt8{2-0};
  2828: }
  2829: class Enc_c7a204 : OpcodeHexagon {
  2830:   bits <6> II;
  2831:   let Inst{5-0} = II{5-0};
  2832:   bits <5> Rtt32;
  2833:   let Inst{12-8} = Rtt32{4-0};
  2834:   bits <5> Re32;
  2835:   let Inst{20-16} = Re32{4-0};
  2836: }
  2837: class Enc_c7cd90 : OpcodeHexagon {
  2838:   bits <4> Ii;
  2839:   let Inst{6-3} = Ii{3-0};
  2840:   bits <3> Nt8;
  2841:   let Inst{10-8} = Nt8{2-0};
  2842:   bits <5> Rx32;
  2843:   let Inst{20-16} = Rx32{4-0};
  2844: }
  2845: class Enc_c85e2a : OpcodeHexagon {
  2846:   bits <5> Ii;
  2847:   let Inst{12-8} = Ii{4-0};
  2848:   bits <5> II;
  2849:   let Inst{22-21} = II{4-3};
  2850:   let Inst{7-5} = II{2-0};
  2851:   bits <5> Rd32;
  2852:   let Inst{4-0} = Rd32{4-0};
  2853: }
  2854: class Enc_c89067 : OpcodeHexagon {
  2855:   bits <5> Rtt32;
  2856:   let Inst{20-16} = Rtt32{4-0};
  2857:   bits <5> Rdd32;
  2858:   let Inst{4-0} = Rdd32{4-0};
  2859:   bits <5> Rx32;
  2860:   let Inst{12-8} = Rx32{4-0};
  2861: }
  2862: class Enc_c90aca : OpcodeHexagon {
  2863:   bits <8> Ii;
  2864:   let Inst{12-5} = Ii{7-0};
  2865:   bits <5> Rs32;
  2866:   let Inst{20-16} = Rs32{4-0};
  2867:   bits <5> Rx32;
  2868:   let Inst{4-0} = Rx32{4-0};
  2869: }
  2870: class Enc_c9a18e : OpcodeHexagon {
  2871:   bits <11> Ii;
  2872:   let Inst{21-20} = Ii{10-9};
  2873:   let Inst{7-1} = Ii{8-2};
  2874:   bits <3> Ns8;
  2875:   let Inst{18-16} = Ns8{2-0};
  2876:   bits <5> Rt32;
  2877:   let Inst{12-8} = Rt32{4-0};
  2878: }
  2879: class Enc_c9e3bc : OpcodeHexagon {
  2880:   bits <4> Ii;
  2881:   let Inst{13-13} = Ii{3-3};
  2882:   let Inst{10-8} = Ii{2-0};
  2883:   bits <5> Rt32;
  2884:   let Inst{20-16} = Rt32{4-0};
  2885:   bits <5> Vs32;
  2886:   let Inst{4-0} = Vs32{4-0};
  2887: }
  2888: class Enc_ca3887 : OpcodeHexagon {
  2889:   bits <5> Rs32;
  2890:   let Inst{20-16} = Rs32{4-0};
  2891:   bits <5> Rt32;
  2892:   let Inst{12-8} = Rt32{4-0};
  2893: }
  2894: class Enc_cb4b4e : OpcodeHexagon {
  2895:   bits <2> Pu4;
  2896:   let Inst{6-5} = Pu4{1-0};
  2897:   bits <5> Rs32;
  2898:   let Inst{20-16} = Rs32{4-0};
  2899:   bits <5> Rt32;
  2900:   let Inst{12-8} = Rt32{4-0};
  2901:   bits <5> Rdd32;
  2902:   let Inst{4-0} = Rdd32{4-0};
  2903: }
  2904: class Enc_cb785b : OpcodeHexagon {
  2905:   bits <5> Vu32;
  2906:   let Inst{12-8} = Vu32{4-0};
  2907:   bits <5> Rtt32;
  2908:   let Inst{20-16} = Rtt32{4-0};
  2909:   bits <5> Vdd32;
  2910:   let Inst{4-0} = Vdd32{4-0};
  2911: }
  2912: class Enc_cb9321 : OpcodeHexagon {
  2913:   bits <16> Ii;
  2914:   let Inst{27-21} = Ii{15-9};
  2915:   let Inst{13-5} = Ii{8-0};
  2916:   bits <5> Rs32;
  2917:   let Inst{20-16} = Rs32{4-0};
  2918:   bits <5> Rd32;
  2919:   let Inst{4-0} = Rd32{4-0};
  2920: }
  2921: class Enc_cc449f : OpcodeHexagon {
  2922:   bits <4> Ii;
  2923:   let Inst{6-3} = Ii{3-0};
  2924:   bits <2> Pv4;
  2925:   let Inst{1-0} = Pv4{1-0};
  2926:   bits <5> Rt32;
  2927:   let Inst{12-8} = Rt32{4-0};
  2928:   bits <5> Rx32;
  2929:   let Inst{20-16} = Rx32{4-0};
  2930: }
  2931: class Enc_cc857d : OpcodeHexagon {
  2932:   bits <5> Vuu32;
  2933:   let Inst{12-8} = Vuu32{4-0};
  2934:   bits <5> Rt32;
  2935:   let Inst{20-16} = Rt32{4-0};
  2936:   bits <5> Vx32;
  2937:   let Inst{4-0} = Vx32{4-0};
  2938: }
  2939: class Enc_cd4705 : OpcodeHexagon {
  2940:   bits <3> Ii;
  2941:   let Inst{7-5} = Ii{2-0};
  2942:   bits <5> Vu32;
  2943:   let Inst{12-8} = Vu32{4-0};
  2944:   bits <5> Vv32;
  2945:   let Inst{20-16} = Vv32{4-0};
  2946:   bits <5> Vx32;
  2947:   let Inst{4-0} = Vx32{4-0};
  2948: }
  2949: class Enc_cd82bc : OpcodeHexagon {
  2950:   bits <4> Ii;
  2951:   let Inst{21-21} = Ii{3-3};
  2952:   let Inst{7-5} = Ii{2-0};
  2953:   bits <6> II;
  2954:   let Inst{13-8} = II{5-0};
  2955:   bits <5> Rs32;
  2956:   let Inst{20-16} = Rs32{4-0};
  2957:   bits <5> Rx32;
  2958:   let Inst{4-0} = Rx32{4-0};
  2959: }
  2960: class Enc_cda00a : OpcodeHexagon {
  2961:   bits <12> Ii;
  2962:   let Inst{19-16} = Ii{11-8};
  2963:   let Inst{12-5} = Ii{7-0};
  2964:   bits <2> Pu4;
  2965:   let Inst{22-21} = Pu4{1-0};
  2966:   bits <5> Rd32;
  2967:   let Inst{4-0} = Rd32{4-0};
  2968: }
  2969: class Enc_ce6828 : OpcodeHexagon {
  2970:   bits <14> Ii;
  2971:   let Inst{26-25} = Ii{13-12};
  2972:   let Inst{13-13} = Ii{11-11};
  2973:   let Inst{7-0} = Ii{10-3};
  2974:   bits <5> Rs32;
  2975:   let Inst{20-16} = Rs32{4-0};
  2976:   bits <5> Rtt32;
  2977:   let Inst{12-8} = Rtt32{4-0};
  2978: }
  2979: class Enc_cf1927 : OpcodeHexagon {
  2980:   bits <1> Mu2;
  2981:   let Inst{13-13} = Mu2{0-0};
  2982:   bits <3> Os8;
  2983:   let Inst{2-0} = Os8{2-0};
  2984:   bits <5> Rx32;
  2985:   let Inst{20-16} = Rx32{4-0};
  2986: }
  2987: class Enc_d0fe02 : OpcodeHexagon {
  2988:   bits <5> Rxx32;
  2989:   let Inst{20-16} = Rxx32{4-0};
  2990:   bits <0> sgp10;
  2991: }
  2992: class Enc_d15d19 : OpcodeHexagon {
  2993:   bits <1> Mu2;
  2994:   let Inst{13-13} = Mu2{0-0};
  2995:   bits <5> Vs32;
  2996:   let Inst{4-0} = Vs32{4-0};
  2997:   bits <5> Rx32;
  2998:   let Inst{20-16} = Rx32{4-0};
  2999: }
  3000: class Enc_d2216a : OpcodeHexagon {
```
- EN: It declares types such as Enc_bea5da, Enc_bfbf03, Enc_c0cdde, Enc_c175d0, ... (30 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_bea5da, Enc_bfbf03, Enc_c0cdde, Enc_c175d0, Enc_c1d806, ... (30 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_bea5da, Enc_bfbf03, Enc_c0cdde, Enc_c175d0, ... (30 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_bea5da, Enc_bfbf03, Enc_c0cdde, Enc_c175d0, Enc_c1d806, ... (30 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 3001-3250 / 第 3001-3250 行

```tablegen
  3001:   bits <5> Rss32;
  3002:   let Inst{20-16} = Rss32{4-0};
  3003:   bits <5> Rtt32;
  3004:   let Inst{12-8} = Rtt32{4-0};
  3005:   bits <5> Rd32;
  3006:   let Inst{4-0} = Rd32{4-0};
  3007: }
  3008: class Enc_d2c7f1 : OpcodeHexagon {
  3009:   bits <5> Rtt32;
  3010:   let Inst{12-8} = Rtt32{4-0};
  3011:   bits <5> Rss32;
  3012:   let Inst{20-16} = Rss32{4-0};
  3013:   bits <5> Rdd32;
  3014:   let Inst{4-0} = Rdd32{4-0};
  3015:   bits <2> Pe4;
  3016:   let Inst{6-5} = Pe4{1-0};
  3017: }
  3018: class Enc_d44e31 : OpcodeHexagon {
  3019:   bits <6> Ii;
  3020:   let Inst{12-7} = Ii{5-0};
  3021:   bits <5> Rs32;
  3022:   let Inst{20-16} = Rs32{4-0};
  3023:   bits <5> Rt32;
  3024:   let Inst{4-0} = Rt32{4-0};
  3025: }
  3026: class Enc_d483b9 : OpcodeHexagon {
  3027:   bits <1> Ii;
  3028:   let Inst{5-5} = Ii{0-0};
  3029:   bits <5> Vuu32;
  3030:   let Inst{12-8} = Vuu32{4-0};
  3031:   bits <5> Rt32;
  3032:   let Inst{20-16} = Rt32{4-0};
  3033:   bits <5> Vxx32;
  3034:   let Inst{4-0} = Vxx32{4-0};
  3035: }
  3036: class Enc_d50cd3 : OpcodeHexagon {
  3037:   bits <3> Ii;
  3038:   let Inst{7-5} = Ii{2-0};
  3039:   bits <5> Rss32;
  3040:   let Inst{20-16} = Rss32{4-0};
  3041:   bits <5> Rtt32;
  3042:   let Inst{12-8} = Rtt32{4-0};
  3043:   bits <5> Rdd32;
  3044:   let Inst{4-0} = Rdd32{4-0};
  3045: }
  3046: class Enc_d5c73f : OpcodeHexagon {
  3047:   bits <1> Mu2;
  3048:   let Inst{13-13} = Mu2{0-0};
  3049:   bits <5> Rt32;
  3050:   let Inst{12-8} = Rt32{4-0};
  3051:   bits <5> Rx32;
  3052:   let Inst{20-16} = Rx32{4-0};
  3053: }
  3054: class Enc_d6990d : OpcodeHexagon {
  3055:   bits <5> Vuu32;
  3056:   let Inst{12-8} = Vuu32{4-0};
  3057:   bits <5> Rt32;
  3058:   let Inst{20-16} = Rt32{4-0};
  3059:   bits <5> Vxx32;
  3060:   let Inst{4-0} = Vxx32{4-0};
  3061: }
  3062: class Enc_d7a65e : OpcodeHexagon {
  3063:   bits <6> Ii;
  3064:   let Inst{12-7} = Ii{5-0};
  3065:   bits <6> II;
  3066:   let Inst{13-13} = II{5-5};
  3067:   let Inst{4-0} = II{4-0};
  3068:   bits <2> Pv4;
  3069:   let Inst{6-5} = Pv4{1-0};
  3070:   bits <5> Rs32;
  3071:   let Inst{20-16} = Rs32{4-0};
  3072: }
  3073: class Enc_d7bc34 : OpcodeHexagon {
  3074:   bits <5> Vu32;
  3075:   let Inst{12-8} = Vu32{4-0};
  3076:   bits <3> Rt8;
  3077:   let Inst{18-16} = Rt8{2-0};
  3078:   bits <5> Vyyyy32;
  3079:   let Inst{4-0} = Vyyyy32{4-0};
  3080: }
  3081: class Enc_d7dc10 : OpcodeHexagon {
  3082:   bits <5> Rs32;
  3083:   let Inst{20-16} = Rs32{4-0};
  3084:   bits <5> Rtt32;
  3085:   let Inst{12-8} = Rtt32{4-0};
  3086:   bits <2> Pd4;
  3087:   let Inst{1-0} = Pd4{1-0};
  3088: }
  3089: class Enc_da664b : OpcodeHexagon {
  3090:   bits <2> Ii;
  3091:   let Inst{13-13} = Ii{1-1};
  3092:   let Inst{7-7} = Ii{0-0};
  3093:   bits <5> Rs32;
  3094:   let Inst{20-16} = Rs32{4-0};
  3095:   bits <5> Rt32;
  3096:   let Inst{12-8} = Rt32{4-0};
  3097:   bits <5> Rd32;
  3098:   let Inst{4-0} = Rd32{4-0};
  3099: }
  3100: class Enc_da8d43 : OpcodeHexagon {
  3101:   bits <6> Ii;
  3102:   let Inst{13-13} = Ii{5-5};
  3103:   let Inst{7-3} = Ii{4-0};
  3104:   bits <2> Pv4;
  3105:   let Inst{1-0} = Pv4{1-0};
  3106:   bits <5> Rs32;
  3107:   let Inst{20-16} = Rs32{4-0};
  3108:   bits <5> Rt32;
  3109:   let Inst{12-8} = Rt32{4-0};
  3110: }
  3111: class Enc_daea09 : OpcodeHexagon {
  3112:   bits <17> Ii;
  3113:   let Inst{23-22} = Ii{16-15};
  3114:   let Inst{20-16} = Ii{14-10};
  3115:   let Inst{13-13} = Ii{9-9};
  3116:   let Inst{7-1} = Ii{8-2};
  3117:   bits <2> Pu4;
  3118:   let Inst{9-8} = Pu4{1-0};
  3119: }
  3120: class Enc_db40cd : OpcodeHexagon {
  3121:   bits <6> Ii;
  3122:   let Inst{6-3} = Ii{5-2};
  3123:   bits <5> Rt32;
  3124:   let Inst{12-8} = Rt32{4-0};
  3125:   bits <5> Rx32;
  3126:   let Inst{20-16} = Rx32{4-0};
  3127: }
  3128: class Enc_dbd70c : OpcodeHexagon {
  3129:   bits <5> Rss32;
  3130:   let Inst{20-16} = Rss32{4-0};
  3131:   bits <5> Rtt32;
  3132:   let Inst{12-8} = Rtt32{4-0};
  3133:   bits <2> Pu4;
  3134:   let Inst{6-5} = Pu4{1-0};
  3135:   bits <5> Rdd32;
  3136:   let Inst{4-0} = Rdd32{4-0};
  3137: }
  3138: class Enc_dd766a : OpcodeHexagon {
  3139:   bits <5> Vu32;
  3140:   let Inst{12-8} = Vu32{4-0};
  3141:   bits <5> Vdd32;
  3142:   let Inst{4-0} = Vdd32{4-0};
  3143: }
  3144: class Enc_de0214 : OpcodeHexagon {
  3145:   bits <12> Ii;
  3146:   let Inst{26-25} = Ii{11-10};
  3147:   let Inst{13-5} = Ii{9-1};
  3148:   bits <5> Rs32;
  3149:   let Inst{20-16} = Rs32{4-0};
  3150:   bits <5> Rd32;
  3151:   let Inst{4-0} = Rd32{4-0};
  3152: }
  3153: class Enc_de5ea0 : OpcodeHexagon {
  3154:   bits <5> Vuu32;
  3155:   let Inst{12-8} = Vuu32{4-0};
  3156:   bits <5> Vv32;
  3157:   let Inst{20-16} = Vv32{4-0};
  3158:   bits <5> Vd32;
  3159:   let Inst{4-0} = Vd32{4-0};
  3160: }
  3161: class Enc_e07374 : OpcodeHexagon {
  3162:   bits <5> Rs32;
  3163:   let Inst{20-16} = Rs32{4-0};
  3164:   bits <5> Rtt32;
  3165:   let Inst{12-8} = Rtt32{4-0};
  3166:   bits <5> Rd32;
  3167:   let Inst{4-0} = Rd32{4-0};
  3168: }
  3169: class Enc_e0820b : OpcodeHexagon {
  3170:   bits <5> Vu32;
  3171:   let Inst{12-8} = Vu32{4-0};
  3172:   bits <5> Vv32;
  3173:   let Inst{20-16} = Vv32{4-0};
  3174:   bits <2> Qs4;
  3175:   let Inst{6-5} = Qs4{1-0};
  3176:   bits <5> Vd32;
  3177:   let Inst{4-0} = Vd32{4-0};
  3178: }
  3179: class Enc_e0a47a : OpcodeHexagon {
  3180:   bits <4> Ii;
  3181:   let Inst{8-5} = Ii{3-0};
  3182:   bits <1> Mu2;
  3183:   let Inst{13-13} = Mu2{0-0};
  3184:   bits <5> Rd32;
  3185:   let Inst{4-0} = Rd32{4-0};
  3186:   bits <5> Rx32;
  3187:   let Inst{20-16} = Rx32{4-0};
  3188: }
  3189: class Enc_e26546 : OpcodeHexagon {
  3190:   bits <5> Ii;
  3191:   let Inst{6-3} = Ii{4-1};
  3192:   bits <3> Nt8;
  3193:   let Inst{10-8} = Nt8{2-0};
  3194:   bits <5> Rx32;
  3195:   let Inst{20-16} = Rx32{4-0};
  3196: }
  3197: class Enc_e32517 : OpcodeHexagon {
  3198:   bits <7> Sss128;
  3199:   let Inst{22-16} = Sss128{6-0};
  3200:   bits <5> Rdd32;
  3201:   let Inst{4-0} = Rdd32{4-0};
  3202: }
  3203: class Enc_e38e1f : OpcodeHexagon {
  3204:   bits <8> Ii;
  3205:   let Inst{12-5} = Ii{7-0};
  3206:   bits <2> Pu4;
  3207:   let Inst{22-21} = Pu4{1-0};
  3208:   bits <5> Rs32;
  3209:   let Inst{20-16} = Rs32{4-0};
  3210:   bits <5> Rd32;
  3211:   let Inst{4-0} = Rd32{4-0};
  3212: }
  3213: class Enc_e39bb2 : OpcodeHexagon {
  3214:   bits <6> Ii;
  3215:   let Inst{9-4} = Ii{5-0};
  3216:   bits <4> Rd16;
  3217:   let Inst{3-0} = Rd16{3-0};
  3218: }
  3219: class Enc_e3b0c4 : OpcodeHexagon {
  3220: 
  3221: }
  3222: class Enc_e66a97 : OpcodeHexagon {
  3223:   bits <7> Ii;
  3224:   let Inst{12-7} = Ii{6-1};
  3225:   bits <5> II;
  3226:   let Inst{4-0} = II{4-0};
  3227:   bits <5> Rs32;
  3228:   let Inst{20-16} = Rs32{4-0};
  3229: }
  3230: class Enc_e6abcf : OpcodeHexagon {
  3231:   bits <5> Rs32;
  3232:   let Inst{20-16} = Rs32{4-0};
  3233:   bits <5> Rtt32;
  3234:   let Inst{12-8} = Rtt32{4-0};
  3235: }
  3236: class Enc_e6c957 : OpcodeHexagon {
  3237:   bits <10> Ii;
  3238:   let Inst{21-21} = Ii{9-9};
  3239:   let Inst{13-5} = Ii{8-0};
  3240:   bits <5> Rdd32;
  3241:   let Inst{4-0} = Rdd32{4-0};
  3242: }
  3243: class Enc_e7581c : OpcodeHexagon {
  3244:   bits <5> Vu32;
  3245:   let Inst{12-8} = Vu32{4-0};
  3246:   bits <5> Vd32;
  3247:   let Inst{4-0} = Vd32{4-0};
  3248: }
  3249: class Enc_e83554 : OpcodeHexagon {
  3250:   bits <5> Ii;
```
- EN: It declares types such as Enc_d2c7f1, Enc_d44e31, Enc_d483b9, Enc_d50cd3, ... (30 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_d2c7f1, Enc_d44e31, Enc_d483b9, Enc_d50cd3, Enc_d5c73f, ... (30 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_d2c7f1, Enc_d44e31, Enc_d483b9, Enc_d50cd3, ... (30 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_d2c7f1, Enc_d44e31, Enc_d483b9, Enc_d50cd3, Enc_d5c73f, ... (30 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 3251-3500 / 第 3251-3500 行

```tablegen
  3251:   let Inst{8-5} = Ii{4-1};
  3252:   bits <1> Mu2;
  3253:   let Inst{13-13} = Mu2{0-0};
  3254:   bits <5> Rd32;
  3255:   let Inst{4-0} = Rd32{4-0};
  3256:   bits <5> Rx32;
  3257:   let Inst{20-16} = Rx32{4-0};
  3258: }
  3259: class Enc_e8c45e : OpcodeHexagon {
  3260:   bits <7> Ii;
  3261:   let Inst{13-13} = Ii{6-6};
  3262:   let Inst{7-3} = Ii{5-1};
  3263:   bits <2> Pv4;
  3264:   let Inst{1-0} = Pv4{1-0};
  3265:   bits <5> Rs32;
  3266:   let Inst{20-16} = Rs32{4-0};
  3267:   bits <5> Rt32;
  3268:   let Inst{12-8} = Rt32{4-0};
  3269: }
  3270: class Enc_e90a15 : OpcodeHexagon {
  3271:   bits <11> Ii;
  3272:   let Inst{21-20} = Ii{10-9};
  3273:   let Inst{7-1} = Ii{8-2};
  3274:   bits <3> Ns8;
  3275:   let Inst{18-16} = Ns8{2-0};
  3276:   bits <0> n1;
  3277: }
  3278: class Enc_e957fb : OpcodeHexagon {
  3279:   bits <12> Ii;
  3280:   let Inst{26-25} = Ii{11-10};
  3281:   let Inst{13-13} = Ii{9-9};
  3282:   let Inst{7-0} = Ii{8-1};
  3283:   bits <5> Rs32;
  3284:   let Inst{20-16} = Rs32{4-0};
  3285:   bits <5> Rt32;
  3286:   let Inst{12-8} = Rt32{4-0};
  3287: }
  3288: class Enc_ea23e4 : OpcodeHexagon {
  3289:   bits <5> Rtt32;
  3290:   let Inst{12-8} = Rtt32{4-0};
  3291:   bits <5> Rss32;
  3292:   let Inst{20-16} = Rss32{4-0};
  3293:   bits <5> Rdd32;
  3294:   let Inst{4-0} = Rdd32{4-0};
  3295: }
  3296: class Enc_ea4c54 : OpcodeHexagon {
  3297:   bits <2> Pu4;
  3298:   let Inst{6-5} = Pu4{1-0};
  3299:   bits <5> Rs32;
  3300:   let Inst{20-16} = Rs32{4-0};
  3301:   bits <5> Rt32;
  3302:   let Inst{12-8} = Rt32{4-0};
  3303:   bits <5> Rd32;
  3304:   let Inst{4-0} = Rd32{4-0};
  3305: }
  3306: class Enc_eaa9f8 : OpcodeHexagon {
  3307:   bits <5> Vu32;
  3308:   let Inst{12-8} = Vu32{4-0};
  3309:   bits <5> Vv32;
  3310:   let Inst{20-16} = Vv32{4-0};
  3311:   bits <2> Qx4;
  3312:   let Inst{1-0} = Qx4{1-0};
  3313: }
  3314: class Enc_eafd18 : OpcodeHexagon {
  3315:   bits <5> II;
  3316:   let Inst{12-8} = II{4-0};
  3317:   bits <11> Ii;
  3318:   let Inst{21-20} = Ii{10-9};
  3319:   let Inst{7-1} = Ii{8-2};
  3320:   bits <3> Ns8;
  3321:   let Inst{18-16} = Ns8{2-0};
  3322: }
  3323: class Enc_eca7c8 : OpcodeHexagon {
  3324:   bits <2> Ii;
  3325:   let Inst{13-13} = Ii{1-1};
  3326:   let Inst{7-7} = Ii{0-0};
  3327:   bits <5> Rs32;
  3328:   let Inst{20-16} = Rs32{4-0};
  3329:   bits <5> Ru32;
  3330:   let Inst{12-8} = Ru32{4-0};
  3331:   bits <5> Rt32;
  3332:   let Inst{4-0} = Rt32{4-0};
  3333: }
  3334: class Enc_ecbcc8 : OpcodeHexagon {
  3335:   bits <5> Rs32;
  3336:   let Inst{20-16} = Rs32{4-0};
  3337: }
  3338: class Enc_ed48be : OpcodeHexagon {
  3339:   bits <2> Ii;
  3340:   let Inst{6-5} = Ii{1-0};
  3341:   bits <3> Rdd8;
  3342:   let Inst{2-0} = Rdd8{2-0};
  3343: }
  3344: class Enc_ed5027 : OpcodeHexagon {
  3345:   bits <5> Rss32;
  3346:   let Inst{20-16} = Rss32{4-0};
  3347:   bits <5> Gdd32;
  3348:   let Inst{4-0} = Gdd32{4-0};
  3349: }
  3350: class Enc_ee5ed0 : OpcodeHexagon {
  3351:   bits <4> Rs16;
  3352:   let Inst{7-4} = Rs16{3-0};
  3353:   bits <4> Rd16;
  3354:   let Inst{3-0} = Rd16{3-0};
  3355:   bits <0> n1;
  3356: }
  3357: class Enc_ef601b : OpcodeHexagon {
  3358:   bits <4> Ii;
  3359:   let Inst{13-13} = Ii{3-3};
  3360:   let Inst{10-8} = Ii{2-0};
  3361:   bits <2> Pv4;
  3362:   let Inst{12-11} = Pv4{1-0};
  3363:   bits <5> Rt32;
  3364:   let Inst{20-16} = Rt32{4-0};
  3365: }
  3366: class Enc_efaed8 : OpcodeHexagon {
  3367:   bits <1> Ii;
  3368:   let Inst{8-8} = Ii{0-0};
  3369: }
  3370: class Enc_f0cca7 : OpcodeHexagon {
  3371:   bits <8> Ii;
  3372:   let Inst{12-5} = Ii{7-0};
  3373:   bits <6> II;
  3374:   let Inst{20-16} = II{5-1};
  3375:   let Inst{13-13} = II{0-0};
  3376:   bits <5> Rdd32;
  3377:   let Inst{4-0} = Rdd32{4-0};
  3378: }
  3379: class Enc_f20719 : OpcodeHexagon {
  3380:   bits <7> Ii;
  3381:   let Inst{12-7} = Ii{6-1};
  3382:   bits <6> II;
  3383:   let Inst{13-13} = II{5-5};
  3384:   let Inst{4-0} = II{4-0};
  3385:   bits <2> Pv4;
  3386:   let Inst{6-5} = Pv4{1-0};
  3387:   bits <5> Rs32;
  3388:   let Inst{20-16} = Rs32{4-0};
  3389: }
  3390: class Enc_f37377 : OpcodeHexagon {
  3391:   bits <8> Ii;
  3392:   let Inst{12-7} = Ii{7-2};
  3393:   bits <8> II;
  3394:   let Inst{13-13} = II{7-7};
  3395:   let Inst{6-0} = II{6-0};
  3396:   bits <5> Rs32;
  3397:   let Inst{20-16} = Rs32{4-0};
  3398: }
  3399: class Enc_f394d3 : OpcodeHexagon {
  3400:   bits <6> II;
  3401:   let Inst{11-8} = II{5-2};
  3402:   let Inst{6-5} = II{1-0};
  3403:   bits <5> Ryy32;
  3404:   let Inst{4-0} = Ryy32{4-0};
  3405:   bits <5> Re32;
  3406:   let Inst{20-16} = Re32{4-0};
  3407: }
  3408: class Enc_f3f408 : OpcodeHexagon {
  3409:   bits <4> Ii;
  3410:   let Inst{13-13} = Ii{3-3};
  3411:   let Inst{10-8} = Ii{2-0};
  3412:   bits <5> Rt32;
  3413:   let Inst{20-16} = Rt32{4-0};
  3414:   bits <5> Vd32;
  3415:   let Inst{4-0} = Vd32{4-0};
  3416: }
  3417: class Enc_f4413a : OpcodeHexagon {
  3418:   bits <4> Ii;
  3419:   let Inst{8-5} = Ii{3-0};
  3420:   bits <2> Pt4;
  3421:   let Inst{10-9} = Pt4{1-0};
  3422:   bits <5> Rd32;
  3423:   let Inst{4-0} = Rd32{4-0};
  3424:   bits <5> Rx32;
  3425:   let Inst{20-16} = Rx32{4-0};
  3426: }
  3427: class Enc_f44229 : OpcodeHexagon {
  3428:   bits <7> Ii;
  3429:   let Inst{13-13} = Ii{6-6};
  3430:   let Inst{7-3} = Ii{5-1};
  3431:   bits <2> Pv4;
  3432:   let Inst{1-0} = Pv4{1-0};
  3433:   bits <5> Rs32;
  3434:   let Inst{20-16} = Rs32{4-0};
  3435:   bits <3> Nt8;
  3436:   let Inst{10-8} = Nt8{2-0};
  3437: }
  3438: class Enc_f4f57b : OpcodeHexagon {
  3439:   bits <2> Ii;
  3440:   let Inst{6-5} = Ii{1-0};
  3441:   bits <5> Vuu32;
  3442:   let Inst{12-8} = Vuu32{4-0};
  3443:   bits <5> Vvv32;
  3444:   let Inst{20-16} = Vvv32{4-0};
  3445:   bits <5> Vxx32;
  3446:   let Inst{4-0} = Vxx32{4-0};
  3447: }
  3448: class Enc_f55a0c : OpcodeHexagon {
  3449:   bits <6> Ii;
  3450:   let Inst{11-8} = Ii{5-2};
  3451:   bits <4> Rs16;
  3452:   let Inst{7-4} = Rs16{3-0};
  3453:   bits <4> Rt16;
  3454:   let Inst{3-0} = Rt16{3-0};
  3455: }
  3456: class Enc_f5e933 : OpcodeHexagon {
  3457:   bits <2> Ps4;
  3458:   let Inst{17-16} = Ps4{1-0};
  3459:   bits <5> Rd32;
  3460:   let Inst{4-0} = Rd32{4-0};
  3461: }
  3462: class Enc_f6fe0b : OpcodeHexagon {
  3463:   bits <11> Ii;
  3464:   let Inst{21-20} = Ii{10-9};
  3465:   let Inst{7-1} = Ii{8-2};
  3466:   bits <4> Rs16;
  3467:   let Inst{19-16} = Rs16{3-0};
  3468:   bits <0> n1;
  3469: }
  3470: class Enc_f7430e : OpcodeHexagon {
  3471:   bits <4> Ii;
  3472:   let Inst{13-13} = Ii{3-3};
  3473:   let Inst{10-8} = Ii{2-0};
  3474:   bits <2> Pv4;
  3475:   let Inst{12-11} = Pv4{1-0};
  3476:   bits <5> Rt32;
  3477:   let Inst{20-16} = Rt32{4-0};
  3478:   bits <3> Os8;
  3479:   let Inst{2-0} = Os8{2-0};
  3480: }
  3481: class Enc_f77fbc : OpcodeHexagon {
  3482:   bits <4> Ii;
  3483:   let Inst{13-13} = Ii{3-3};
  3484:   let Inst{10-8} = Ii{2-0};
  3485:   bits <5> Rt32;
  3486:   let Inst{20-16} = Rt32{4-0};
  3487:   bits <3> Os8;
  3488:   let Inst{2-0} = Os8{2-0};
  3489: }
  3490: class Enc_f79415 : OpcodeHexagon {
  3491:   bits <2> Ii;
  3492:   let Inst{13-13} = Ii{1-1};
  3493:   let Inst{6-6} = Ii{0-0};
  3494:   bits <6> II;
  3495:   let Inst{5-0} = II{5-0};
  3496:   bits <5> Ru32;
  3497:   let Inst{20-16} = Ru32{4-0};
  3498:   bits <5> Rtt32;
  3499:   let Inst{12-8} = Rtt32{4-0};
  3500: }
```
- EN: It declares types such as Enc_e8c45e, Enc_e90a15, Enc_e957fb, Enc_ea23e4, ... (28 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_e8c45e, Enc_e90a15, Enc_e957fb, Enc_ea23e4, Enc_ea4c54, ... (28 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_e8c45e, Enc_e90a15, Enc_e957fb, Enc_ea23e4, ... (28 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_e8c45e, Enc_e90a15, Enc_e957fb, Enc_ea23e4, Enc_ea4c54, ... (28 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 3501-3602 / 第 3501-3602 行

```tablegen
  3501: class Enc_f7ea77 : OpcodeHexagon {
  3502:   bits <11> Ii;
  3503:   let Inst{21-20} = Ii{10-9};
  3504:   let Inst{7-1} = Ii{8-2};
  3505:   bits <3> Ns8;
  3506:   let Inst{18-16} = Ns8{2-0};
  3507:   bits <0> n1;
  3508: }
  3509: class Enc_f82302 : OpcodeHexagon {
  3510:   bits <11> Ii;
  3511:   let Inst{21-20} = Ii{10-9};
  3512:   let Inst{7-1} = Ii{8-2};
  3513:   bits <3> Ns8;
  3514:   let Inst{18-16} = Ns8{2-0};
  3515:   bits <0> n1;
  3516: }
  3517: class Enc_f82eaf : OpcodeHexagon {
  3518:   bits <8> Ii;
  3519:   let Inst{10-5} = Ii{7-2};
  3520:   bits <2> Pt4;
  3521:   let Inst{12-11} = Pt4{1-0};
  3522:   bits <5> Rs32;
  3523:   let Inst{20-16} = Rs32{4-0};
  3524:   bits <5> Rd32;
  3525:   let Inst{4-0} = Rd32{4-0};
  3526: }
  3527: class Enc_f8c1c4 : OpcodeHexagon {
  3528:   bits <2> Pv4;
  3529:   let Inst{12-11} = Pv4{1-0};
  3530:   bits <1> Mu2;
  3531:   let Inst{13-13} = Mu2{0-0};
  3532:   bits <5> Vd32;
  3533:   let Inst{4-0} = Vd32{4-0};
  3534:   bits <5> Rx32;
  3535:   let Inst{20-16} = Rx32{4-0};
  3536: }
  3537: class Enc_f8ecf9 : OpcodeHexagon {
  3538:   bits <5> Vuu32;
  3539:   let Inst{12-8} = Vuu32{4-0};
  3540:   bits <5> Vvv32;
  3541:   let Inst{20-16} = Vvv32{4-0};
  3542:   bits <5> Vdd32;
  3543:   let Inst{4-0} = Vdd32{4-0};
  3544: }
  3545: class Enc_fa3ba4 : OpcodeHexagon {
  3546:   bits <14> Ii;
  3547:   let Inst{26-25} = Ii{13-12};
  3548:   let Inst{13-5} = Ii{11-3};
  3549:   bits <5> Rs32;
  3550:   let Inst{20-16} = Rs32{4-0};
  3551:   bits <5> Rdd32;
  3552:   let Inst{4-0} = Rdd32{4-0};
  3553: }
  3554: class Enc_fb6577 : OpcodeHexagon {
  3555:   bits <2> Pu4;
  3556:   let Inst{9-8} = Pu4{1-0};
  3557:   bits <5> Rs32;
  3558:   let Inst{20-16} = Rs32{4-0};
  3559:   bits <5> Rd32;
  3560:   let Inst{4-0} = Rd32{4-0};
  3561: }
  3562: class Enc_fc4562 : OpcodeHexagon {
  3563:   bits <5> Rs32;
  3564:   let Inst{12-8} = Rs32{4-0};
  3565:   bits <5> Rtt32;
  3566:   let Inst{20-16} = Rtt32{4-0};
  3567:   bits <5> Rdd32;
  3568:   let Inst{4-0} = Rdd32{4-0};
  3569: }
  3570: class Enc_fcf7a7 : OpcodeHexagon {
  3571:   bits <5> Rss32;
  3572:   let Inst{20-16} = Rss32{4-0};
  3573:   bits <5> Rtt32;
  3574:   let Inst{12-8} = Rtt32{4-0};
  3575:   bits <2> Pd4;
  3576:   let Inst{1-0} = Pd4{1-0};
  3577: }
  3578: class Enc_fda92c : OpcodeHexagon {
  3579:   bits <17> Ii;
  3580:   let Inst{26-25} = Ii{16-15};
  3581:   let Inst{20-16} = Ii{14-10};
  3582:   let Inst{13-13} = Ii{9-9};
  3583:   let Inst{7-0} = Ii{8-1};
  3584:   bits <5> Rt32;
  3585:   let Inst{12-8} = Rt32{4-0};
  3586: }
  3587: class Enc_fef969 : OpcodeHexagon {
  3588:   bits <6> Ii;
  3589:   let Inst{20-16} = Ii{5-1};
  3590:   let Inst{5-5} = Ii{0-0};
  3591:   bits <5> Rt32;
  3592:   let Inst{12-8} = Rt32{4-0};
  3593:   bits <5> Rd32;
  3594:   let Inst{4-0} = Rd32{4-0};
  3595: }
  3596: class Enc_ff3442 : OpcodeHexagon {
  3597:   bits <4> Ii;
  3598:   let Inst{13-13} = Ii{3-3};
  3599:   let Inst{10-8} = Ii{2-0};
  3600:   bits <5> Rt32;
  3601:   let Inst{20-16} = Rt32{4-0};
  3602: }
```
- EN: It declares types such as Enc_f7ea77, Enc_f82302, Enc_f82eaf, Enc_f8c1c4, ... (12 total), which carry the state or API of this component. It defines generated/declarative TableGen records like Enc_f7ea77, Enc_f82302, Enc_f82eaf, Enc_f8c1c4, Enc_f8ecf9, ... (12 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 Enc_f7ea77, Enc_f82302, Enc_f82eaf, Enc_f8c1c4, ... (12 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Enc_f7ea77, Enc_f82302, Enc_f82eaf, Enc_f8c1c4, Enc_f8ecf9, ... (12 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录
- generated target metadata / 生成的目标元数据

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
