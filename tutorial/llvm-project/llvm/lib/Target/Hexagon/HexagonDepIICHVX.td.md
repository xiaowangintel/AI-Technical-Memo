# HexagonDepIICHVX.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonDepIICHVX.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon instruction itinerary/scheduling classes using TableGen DSL.
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
    11: def tc_0390c1ca : InstrItinClass;
    12: def tc_04da405a : InstrItinClass;
    13: def tc_05ca8cfd : InstrItinClass;
    14: def tc_08a4f1b6 : InstrItinClass;
    15: def tc_0afc8be9 : InstrItinClass;
    16: def tc_0b04c6c7 : InstrItinClass;
    17: def tc_0ec46cf9 : InstrItinClass;
    18: def tc_131f1c81 : InstrItinClass;
    19: def tc_1381a97c : InstrItinClass;
    20: def tc_15fdf750 : InstrItinClass;
    21: def tc_16ff9ef8 : InstrItinClass;
    22: def tc_191381c1 : InstrItinClass;
    23: def tc_1ad8a370 : InstrItinClass;
    24: def tc_1ba8a0cd : InstrItinClass;
    25: def tc_20a4bbec : InstrItinClass;
    26: def tc_227864f7 : InstrItinClass;
    27: def tc_257f6f7c : InstrItinClass;
    28: def tc_26a377fe : InstrItinClass;
    29: def tc_2a698a03 : InstrItinClass;
    30: def tc_2b4c548e : InstrItinClass;
    31: def tc_2c745bb8 : InstrItinClass;
    32: def tc_2d4051cd : InstrItinClass;
    33: def tc_2e8f5f6e : InstrItinClass;
    34: def tc_309dbb4f : InstrItinClass;
    35: def tc_37820f4c : InstrItinClass;
    36: def tc_3904b926 : InstrItinClass;
    37: def tc_3aacf4a8 : InstrItinClass;
    38: def tc_3ad719fb : InstrItinClass;
    39: def tc_3c56e5ce : InstrItinClass;
    40: def tc_3c8c15d0 : InstrItinClass;
    41: def tc_3ce09744 : InstrItinClass;
    42: def tc_3e2aaafc : InstrItinClass;
    43: def tc_447d9895 : InstrItinClass;
    44: def tc_453fe68d : InstrItinClass;
    45: def tc_46d6c3e0 : InstrItinClass;
    46: def tc_4942646a : InstrItinClass;
    47: def tc_51d0ecc3 : InstrItinClass;
    48: def tc_52447ecc : InstrItinClass;
    49: def tc_531b383c : InstrItinClass;
    50: def tc_540c3da3 : InstrItinClass;
    51: def tc_54a0dc47 : InstrItinClass;
    52: def tc_561aaa58 : InstrItinClass;
    53: def tc_56c4f9fe : InstrItinClass;
    54: def tc_56e64202 : InstrItinClass;
    55: def tc_58d21193 : InstrItinClass;
    56: def tc_57a4709c : InstrItinClass;
    57: def tc_5bf8afbb : InstrItinClass;
    58: def tc_5cdf8c84 : InstrItinClass;
    59: def tc_61bf7c03 : InstrItinClass;
    60: def tc_649072c2 : InstrItinClass;
    61: def tc_660769f1 : InstrItinClass;
    62: def tc_663c80a7 : InstrItinClass;
    63: def tc_6942b6e0 : InstrItinClass;
    64: def tc_6e7fa133 : InstrItinClass;
    65: def tc_7095ecba : InstrItinClass;
    66: def tc_71646d06 : InstrItinClass;
    67: def tc_7177e272 : InstrItinClass;
    68: def tc_718b5c53 : InstrItinClass;
    69: def tc_7273323b : InstrItinClass;
    70: def tc_72e2b393 : InstrItinClass;
    71: def tc_73efe966 : InstrItinClass;
    72: def tc_7417e785 : InstrItinClass;
    73: def tc_767c4e9d : InstrItinClass;
    74: def tc_7d68d5c2 : InstrItinClass;
    75: def tc_7e6a3e89 : InstrItinClass;
    76: def tc_8772086c : InstrItinClass;
    77: def tc_87adc037 : InstrItinClass;
    78: def tc_8e420e4d : InstrItinClass;
    79: def tc_90bcc1db : InstrItinClass;
    80: def tc_933f2b39 : InstrItinClass;
    81: def tc_946013d8 : InstrItinClass;
    82: def tc_9a1cab75 : InstrItinClass;
    83: def tc_9aff7a2a : InstrItinClass;
    84: def tc_9d1dc972 : InstrItinClass;
    85: def tc_9f363d21 : InstrItinClass;
    86: def tc_a02a10a8 : InstrItinClass;
    87: def tc_a0dbea28 : InstrItinClass;
    88: def tc_a19b9305 : InstrItinClass;
    89: def tc_a28f32b5 : InstrItinClass;
    90: def tc_a69eeee1 : InstrItinClass;
    91: def tc_a7e6707d : InstrItinClass;
    92: def tc_ab23f776 : InstrItinClass;
    93: def tc_abe8c3b2 : InstrItinClass;
    94: def tc_ac4046bc : InstrItinClass;
    95: def tc_af25efd9 : InstrItinClass;
    96: def tc_b091f1c6 : InstrItinClass;
    97: def tc_b28e51aa : InstrItinClass;
    98: def tc_b4416217 : InstrItinClass;
    99: def tc_b9db8205 : InstrItinClass;
   100: def tc_bb599486 : InstrItinClass;
   101: def tc_c0749f3c : InstrItinClass;
   102: def tc_c127de3a : InstrItinClass;
   103: def tc_c4edf264 : InstrItinClass;
   104: def tc_c5dba46e : InstrItinClass;
   105: def tc_c7039829 : InstrItinClass;
   106: def tc_cd94bfe0 : InstrItinClass;
   107: def tc_cda936da : InstrItinClass;
   108: def tc_d8287c14 : InstrItinClass;
   109: def tc_db5555f3 : InstrItinClass;
   110: def tc_dcca380f : InstrItinClass;
   111: def tc_dd5b0695 : InstrItinClass;
   112: def tc_df80eeb0 : InstrItinClass;
   113: def tc_e2d2e9e5 : InstrItinClass;
   114: def tc_e2fdd6e6 : InstrItinClass;
   115: def tc_e35c1e93 : InstrItinClass;
   116: def tc_e3f68a46 : InstrItinClass;
   117: def tc_e675c45a : InstrItinClass;
   118: def tc_e699ae41 : InstrItinClass;
   119: def tc_e99d4c2e : InstrItinClass;
   120: def tc_f175e046 : InstrItinClass;
   121: def tc_f1de44ef : InstrItinClass;
   122: def tc_f21e8abb : InstrItinClass;
   123: 
   124: class DepHVXItinV55 {
   125:   list<InstrItinData> DepHVXItinV55_list = [
   126:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
   127:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   128:        InstrStage<1, [CVI_LD], 0>,
   129:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
   130:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
   131:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
   132: 
   133:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
   134:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   135:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
   136:       [HVX_FWD, HVX_FWD]>,
   137: 
   138:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
   139:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   140:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
   141:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   142: 
   143:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
   144:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   145:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
   146:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   147: 
   148:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
   149:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   150:        InstrStage<1, [CVI_MPY01]>], [9, 5],
   151:       [HVX_FWD, HVX_FWD]>,
   152: 
   153:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
   154:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   155:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
   156:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   157: 
   158:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
   159:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   160:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
   161:       [HVX_FWD, HVX_FWD]>,
   162: 
   163:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
   164:       [InstrStage<1, [SLOT0], 0>,
   165:        InstrStage<1, [SLOT1], 0>,
   166:        InstrStage<1, [CVI_ST], 0>,
   167:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
   168:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   169: 
   170:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
   171:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   172:        InstrStage<1, [CVI_ALL]>], [],
   173:       []>,
   174: 
   175:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
   176:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   177:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
   178:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
   179:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   180: 
   181:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
   182:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   183:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
   184:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   185: 
   186:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
   187:       [InstrStage<1, [SLOT0], 0>,
   188:        InstrStage<1, [CVI_ST], 0>,
   189:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
   190:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   191: 
   192:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
   193:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   194:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
   195:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
   196: 
   197:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
   198:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   199:        InstrStage<1, [CVI_LD], 0>,
   200:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
   201:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   202: 
   203:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
   204:       [InstrStage<1, [SLOT0], 0>,
   205:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
   206:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   207: 
   208:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
   209:       [InstrStage<1, [SLOT0], 0>,
   210:        InstrStage<1, [CVI_ST], 0>,
   211:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
   212:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
   213:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   214: 
   215:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
   216:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   217:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
   218:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   219: 
   220:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
   221:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   222:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
   223:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
   224: 
   225:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
   226:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   227:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
   228:       [HVX_FWD, HVX_FWD]>,
   229: 
   230:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
   231:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   232:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
   233:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   234: 
   235:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
   236:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   237:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
   238:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   239: 
   240:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
   241:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   242:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
   243:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   244: 
   245:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
   246:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   247:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
   248:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   249: 
   250:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
```
- EN: It declares types such as DepHVXItinV55, which carry the state or API of this component. It defines generated/declarative TableGen records like tc_0390c1ca, tc_04da405a, tc_05ca8cfd, tc_08a4f1b6, tc_0afc8be9, ... (113 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 DepHVXItinV55 等类型，用来承载该组件的状态或接口。 这里定义了 tc_0390c1ca, tc_04da405a, tc_05ca8cfd, tc_08a4f1b6, tc_0afc8be9, ... (113 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-500 / 第 251-500 行

```tablegen
   251:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   252:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
   253:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   254: 
   255:     InstrItinData <tc_37820f4c, /*SLOT23,VX_DV*/
   256:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   257:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
   258:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   259: 
   260:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
   261:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   262:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
   263:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   264: 
   265:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
   266:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   267:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
   268:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
   269: 
   270:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
   271:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   272:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
   273:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   274: 
   275:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
   276:       [InstrStage<1, [SLOT0], 0>,
   277:        InstrStage<1, [SLOT1], 0>,
   278:        InstrStage<1, [CVI_LD], 0>,
   279:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
   280:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   281: 
   282:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
   283:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   284:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
   285:       [HVX_FWD, HVX_FWD]>,
   286: 
   287:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
   288:       [InstrStage<1, [SLOT0], 0>,
   289:        InstrStage<1, [CVI_ST]>], [1, 2],
   290:       [Hex_FWD, Hex_FWD]>,
   291: 
   292:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
   293:       [InstrStage<1, [SLOT0], 0>,
   294:        InstrStage<1, [CVI_ST], 0>,
   295:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
   296:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   297: 
   298:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
   299:       [InstrStage<1, [SLOT0], 0>,
   300:        InstrStage<1, [CVI_ST], 0>,
   301:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
   302:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   303: 
   304:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
   305:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   306:        InstrStage<1, [CVI_LD], 0>,
   307:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
   308:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   309: 
   310:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
   311:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   312:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
   313:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   314: 
   315:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
   316:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   317:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
   318:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   319: 
   320:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
   321:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   322:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
   323:       [HVX_FWD, HVX_FWD]>,
   324: 
   325:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
   326:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   327:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
   328:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
   329: 
   330:     InstrItinData <tc_531b383c, /*SLOT0123*/
   331:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
   332:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   333: 
   334:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
   335:       [InstrStage<1, [SLOT0], 0>,
   336:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
   337:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
   338: 
   339:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
   340:       [InstrStage<1, [SLOT0], 0>,
   341:        InstrStage<1, [CVI_ST], 0>,
   342:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
   343:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   344: 
   345:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
   346:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   347:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
   348:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   349: 
   350:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
   351:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   352:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
   353:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   354: 
   355:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
   356:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   357:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
   358:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   359: 
   360:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
   361:       [InstrStage<1, [SLOT0], 0>,
   362:        InstrStage<1, [CVI_ST], 0>,
   363:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
   364:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
   365: 
   366:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
   367:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   368:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
   369:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   370: 
   371:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
   372:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   373:        InstrStage<1, [CVI_XLANE]>], [9, 2],
   374:       [HVX_FWD, Hex_FWD]>,
   375: 
   376:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
   377:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   378:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
   379:       [HVX_FWD, HVX_FWD]>,
   380: 
   381:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
   382:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   383:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
   384:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   385: 
   386:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
   387:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   388:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
   389:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   390: 
   391:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
   392:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   393:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
   394:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   395: 
   396:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
   397:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   398:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
   399:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   400: 
   401:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
   402:       [InstrStage<1, [SLOT0], 0>,
   403:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
   404:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   405: 
   406:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
   407:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   408:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
   409:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   410: 
   411:     InstrItinData <tc_7095ecba, /*SLOT1,LOAD,VA_DV*/
   412:       [InstrStage<1, [SLOT1], 0>,
   413:        InstrStage<1, [CVI_LD], 0>,
   414:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
   415:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
   416: 
   417:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
   418:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   419:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
   420:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   421: 
   422:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
   423:       [InstrStage<1, [SLOT0], 0>,
   424:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
   425:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   426: 
   427:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
   428:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   429:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
   430:       [HVX_FWD]>,
   431: 
   432:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
   433:       [InstrStage<1, [SLOT0], 0>,
   434:        InstrStage<1, [CVI_ST], 0>,
   435:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
   436:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
   437: 
   438:     InstrItinData <tc_72e2b393, /*SLOT23,VX_DV*/
   439:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   440:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
   441:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   442: 
   443:     InstrItinData <tc_73efe966, /*SLOT23,VX_DV*/
   444:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   445:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
   446:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   447: 
   448:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
   449:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   450:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
   451:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   452: 
   453:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
   454:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   455:        InstrStage<1, [CVI_ALL]>], [3, 2],
   456:       [HVX_FWD, Hex_FWD]>,
   457: 
   458:     InstrItinData <tc_7d68d5c2, /*SLOT1,LOAD,VA*/
   459:       [InstrStage<1, [SLOT1], 0>,
   460:        InstrStage<1, [CVI_LD], 0>,
   461:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
   462:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   463: 
   464:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
   465:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   466:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
   467:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   468: 
   469:     InstrItinData <tc_8772086c, /*SLOT0123,VS*/
   470:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   471:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
   472:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   473: 
   474:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
   475:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   476:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
   477:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   478: 
   479:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
   480:       [InstrStage<1, [SLOT0], 0>,
   481:        InstrStage<1, [CVI_ST], 0>,
   482:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
   483:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
   484: 
   485:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
   486:       [InstrStage<1, [SLOT2], 0>,
   487:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
   488:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   489: 
   490:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
   491:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   492:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
   493:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   494: 
   495:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
   496:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   497:        InstrStage<1, [CVI_XLANE]>], [9, 5],
   498:       [HVX_FWD, HVX_FWD]>,
   499: 
   500:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 501-750 / 第 501-750 行

```tablegen
   501:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   502:        InstrStage<1, [CVI_LD], 0>,
   503:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
   504:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
   505:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   506: 
   507:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
   508:       [InstrStage<1, [SLOT0], 0>,
   509:        InstrStage<1, [CVI_ST], 0>,
   510:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
   511:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
   512:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
   513: 
   514:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
   515:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   516:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
   517:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   518: 
   519:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
   520:       [InstrStage<1, [SLOT0], 0>,
   521:        InstrStage<1, [CVI_ST], 0>,
   522:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
   523:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
   524: 
   525:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
   526:       [InstrStage<1, [SLOT0], 0>,
   527:        InstrStage<1, [CVI_ST], 0>,
   528:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
   529:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   530: 
   531:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
   532:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   533:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
   534:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   535: 
   536:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
   537:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   538:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
   539:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   540: 
   541:     InstrItinData <tc_a28f32b5, /*SLOT1,LOAD,VA*/
   542:       [InstrStage<1, [SLOT1], 0>,
   543:        InstrStage<1, [CVI_LD], 0>,
   544:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
   545:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
   546: 
   547:     InstrItinData <tc_a69eeee1, /*SLOT1,LOAD,VA_DV*/
   548:       [InstrStage<1, [SLOT1], 0>,
   549:        InstrStage<1, [CVI_LD], 0>,
   550:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
   551:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   552: 
   553:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
   554:       [InstrStage<1, [SLOT0], 0>,
   555:        InstrStage<1, [SLOT1], 0>,
   556:        InstrStage<1, [CVI_LD], 0>,
   557:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
   558:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
   559: 
   560:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
   561:       [InstrStage<1, [SLOT0], 0>,
   562:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
   563:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
   564: 
   565:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
   566:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   567:        InstrStage<1, [CVI_LD], 0>,
   568:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
   569:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   570: 
   571:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
   572:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   573:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
   574:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   575: 
   576:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
   577:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   578:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
   579:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
   580: 
   581:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
   582:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   583:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
   584:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   585: 
   586:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
   587:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   588:        InstrStage<1, [CVI_ALL]>], [2],
   589:       [Hex_FWD]>,
   590: 
   591:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
   592:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   593:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
   594:       [HVX_FWD, HVX_FWD]>,
   595: 
   596:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
   597:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   598:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
   599:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   600: 
   601:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
   602:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   603:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
   604:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   605: 
   606:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
   607:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   608:        InstrStage<1, [CVI_LD], 0>,
   609:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
   610:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
   611: 
   612:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
   613:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   614:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
   615:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   616: 
   617:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
   618:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   619:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
   620:       [HVX_FWD, Hex_FWD]>,
   621: 
   622:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
   623:       [InstrStage<1, [SLOT0], 0>,
   624:        InstrStage<1, [CVI_ST], 0>,
   625:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
   626:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
   627: 
   628:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
   629:       [InstrStage<1, [SLOT0], 0>,
   630:        InstrStage<1, [SLOT1], 0>,
   631:        InstrStage<1, [CVI_ST], 0>,
   632:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
   633:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   634: 
   635:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
   636:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   637:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
   638:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
   639:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   640: 
   641:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
   642:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   643:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
   644:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   645: 
   646:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
   647:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   648:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
   649:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   650: 
   651:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
   652:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   653:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
   654:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   655: 
   656:     InstrItinData <tc_dcca380f, /*SLOT23,VX_DV*/
   657:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   658:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
   659:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   660: 
   661:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
   662:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   663:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
   664:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   665: 
   666:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
   667:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   668:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
   669:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   670: 
   671:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
   672:       [InstrStage<1, [SLOT0], 0>,
   673:        InstrStage<1, [SLOT1], 0>,
   674:        InstrStage<1, [CVI_ST], 0>,
   675:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
   676:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   677: 
   678:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
   679:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
   680:       [HVX_FWD, HVX_FWD]>,
   681: 
   682:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
   683:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   684:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
   685:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   686: 
   687:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
   688:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   689:        InstrStage<1, [CVI_ALL]>], [3],
   690:       [HVX_FWD]>,
   691: 
   692:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
   693:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   694:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
   695:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
   696: 
   697:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
   698:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   699:        InstrStage<1, [CVI_ZW]>], [1, 2],
   700:       [Hex_FWD, Hex_FWD]>,
   701: 
   702:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
   703:       [InstrStage<1, [SLOT0], 0>,
   704:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
   705:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   706: 
   707:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
   708:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   709:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
   710:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   711: 
   712:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
   713:       [InstrStage<1, [SLOT2], 0>,
   714:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
   715:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   716: 
   717:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
   718:       [InstrStage<1, [SLOT0], 0>,
   719:        InstrStage<1, [SLOT1], 0>,
   720:        InstrStage<1, [CVI_ST], 0>,
   721:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
   722:       [Hex_FWD, Hex_FWD, HVX_FWD]>
   723:   ];
   724: }
   725: 
   726: class DepHVXItinV60 {
   727:   list<InstrItinData> DepHVXItinV60_list = [
   728:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
   729:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   730:        InstrStage<1, [CVI_LD], 0>,
   731:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
   732:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
   733:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
   734: 
   735:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
   736:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   737:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
   738:       [HVX_FWD, HVX_FWD]>,
   739: 
   740:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
   741:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   742:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
   743:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   744: 
   745:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
   746:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   747:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
   748:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   749: 
   750:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
```
- EN: It declares types such as DepHVXItinV60, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV60; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV60 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV60 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 751-1000 / 第 751-1000 行

```tablegen
   751:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   752:        InstrStage<1, [CVI_MPY01]>], [9, 5],
   753:       [HVX_FWD, HVX_FWD]>,
   754: 
   755:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
   756:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   757:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
   758:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   759: 
   760:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
   761:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   762:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
   763:       [HVX_FWD, HVX_FWD]>,
   764: 
   765:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
   766:       [InstrStage<1, [SLOT0], 0>,
   767:        InstrStage<1, [SLOT1], 0>,
   768:        InstrStage<1, [CVI_ST], 0>,
   769:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
   770:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   771: 
   772:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
   773:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   774:        InstrStage<1, [CVI_ALL]>], [],
   775:       []>,
   776: 
   777:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
   778:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   779:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
   780:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
   781:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   782: 
   783:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
   784:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   785:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
   786:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   787: 
   788:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
   789:       [InstrStage<1, [SLOT0], 0>,
   790:        InstrStage<1, [CVI_ST], 0>,
   791:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
   792:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   793: 
   794:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
   795:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   796:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
   797:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
   798: 
   799:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
   800:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   801:        InstrStage<1, [CVI_LD], 0>,
   802:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
   803:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   804: 
   805:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
   806:       [InstrStage<1, [SLOT0], 0>,
   807:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
   808:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   809: 
   810:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
   811:       [InstrStage<1, [SLOT0], 0>,
   812:        InstrStage<1, [CVI_ST], 0>,
   813:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
   814:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
   815:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   816: 
   817:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
   818:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   819:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
   820:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   821: 
   822:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
   823:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   824:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
   825:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
   826: 
   827:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
   828:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   829:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
   830:       [HVX_FWD, HVX_FWD]>,
   831: 
   832:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
   833:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   834:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
   835:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   836: 
   837:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
   838:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   839:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
   840:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   841: 
   842:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
   843:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   844:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
   845:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   846: 
   847:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
   848:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   849:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
   850:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   851: 
   852:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
   853:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   854:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
   855:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   856: 
   857:     InstrItinData <tc_37820f4c, /*SLOT23,VX_DV*/
   858:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   859:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
   860:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
   861: 
   862:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
   863:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   864:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
   865:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   866: 
   867:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
   868:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   869:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
   870:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
   871: 
   872:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
   873:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   874:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
   875:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   876: 
   877:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
   878:       [InstrStage<1, [SLOT0], 0>,
   879:        InstrStage<1, [SLOT1], 0>,
   880:        InstrStage<1, [CVI_LD], 0>,
   881:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
   882:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   883: 
   884:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
   885:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   886:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
   887:       [HVX_FWD, HVX_FWD]>,
   888: 
   889:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
   890:       [InstrStage<1, [SLOT0], 0>,
   891:        InstrStage<1, [CVI_ST]>], [1, 2],
   892:       [Hex_FWD, Hex_FWD]>,
   893: 
   894:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
   895:       [InstrStage<1, [SLOT0], 0>,
   896:        InstrStage<1, [CVI_ST], 0>,
   897:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
   898:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   899: 
   900:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
   901:       [InstrStage<1, [SLOT0], 0>,
   902:        InstrStage<1, [CVI_ST], 0>,
   903:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
   904:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   905: 
   906:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
   907:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   908:        InstrStage<1, [CVI_LD], 0>,
   909:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
   910:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   911: 
   912:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
   913:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   914:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
   915:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   916: 
   917:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
   918:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   919:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
   920:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   921: 
   922:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
   923:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   924:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
   925:       [HVX_FWD, HVX_FWD]>,
   926: 
   927:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
   928:       [InstrStage<1, [SLOT0, SLOT1], 0>,
   929:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
   930:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
   931: 
   932:     InstrItinData <tc_531b383c, /*SLOT0123*/
   933:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
   934:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   935: 
   936:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
   937:       [InstrStage<1, [SLOT0], 0>,
   938:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
   939:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
   940: 
   941:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
   942:       [InstrStage<1, [SLOT0], 0>,
   943:        InstrStage<1, [CVI_ST], 0>,
   944:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
   945:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
   946: 
   947:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
   948:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   949:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
   950:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   951: 
   952:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
   953:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   954:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
   955:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
   956: 
   957:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
   958:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   959:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
   960:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   961: 
   962:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
   963:       [InstrStage<1, [SLOT0], 0>,
   964:        InstrStage<1, [CVI_ST], 0>,
   965:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
   966:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
   967: 
   968:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
   969:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   970:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
   971:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   972: 
   973:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
   974:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
   975:        InstrStage<1, [CVI_XLANE]>], [9, 2],
   976:       [HVX_FWD, Hex_FWD]>,
   977: 
   978:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
   979:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   980:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
   981:       [HVX_FWD, HVX_FWD]>,
   982: 
   983:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
   984:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   985:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
   986:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   987: 
   988:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
   989:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   990:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
   991:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
   992: 
   993:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
   994:       [InstrStage<1, [SLOT2, SLOT3], 0>,
   995:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
   996:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
   997: 
   998:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
   999:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1000:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1001-1250 / 第 1001-1250 行

```tablegen
  1001:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1002: 
  1003:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  1004:       [InstrStage<1, [SLOT0], 0>,
  1005:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  1006:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1007: 
  1008:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  1009:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1010:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  1011:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1012: 
  1013:     InstrItinData <tc_7095ecba, /*SLOT1,LOAD,VA_DV*/
  1014:       [InstrStage<1, [SLOT1], 0>,
  1015:        InstrStage<1, [CVI_LD], 0>,
  1016:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  1017:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1018: 
  1019:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  1020:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1021:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  1022:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1023: 
  1024:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  1025:       [InstrStage<1, [SLOT0], 0>,
  1026:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  1027:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1028: 
  1029:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  1030:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1031:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  1032:       [HVX_FWD]>,
  1033: 
  1034:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  1035:       [InstrStage<1, [SLOT0], 0>,
  1036:        InstrStage<1, [CVI_ST], 0>,
  1037:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  1038:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1039: 
  1040:     InstrItinData <tc_72e2b393, /*SLOT23,VX_DV*/
  1041:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1042:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  1043:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1044: 
  1045:     InstrItinData <tc_73efe966, /*SLOT23,VX_DV*/
  1046:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1047:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  1048:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1049: 
  1050:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  1051:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1052:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  1053:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1054: 
  1055:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  1056:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1057:        InstrStage<1, [CVI_ALL]>], [3, 2],
  1058:       [HVX_FWD, Hex_FWD]>,
  1059: 
  1060:     InstrItinData <tc_7d68d5c2, /*SLOT1,LOAD,VA*/
  1061:       [InstrStage<1, [SLOT1], 0>,
  1062:        InstrStage<1, [CVI_LD], 0>,
  1063:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  1064:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1065: 
  1066:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  1067:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1068:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  1069:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1070: 
  1071:     InstrItinData <tc_8772086c, /*SLOT0123,VS*/
  1072:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1073:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  1074:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1075: 
  1076:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  1077:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1078:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  1079:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1080: 
  1081:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  1082:       [InstrStage<1, [SLOT0], 0>,
  1083:        InstrStage<1, [CVI_ST], 0>,
  1084:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  1085:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1086: 
  1087:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  1088:       [InstrStage<1, [SLOT2], 0>,
  1089:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  1090:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1091: 
  1092:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  1093:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1094:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  1095:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1096: 
  1097:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  1098:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1099:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  1100:       [HVX_FWD, HVX_FWD]>,
  1101: 
  1102:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  1103:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1104:        InstrStage<1, [CVI_LD], 0>,
  1105:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  1106:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  1107:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1108: 
  1109:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  1110:       [InstrStage<1, [SLOT0], 0>,
  1111:        InstrStage<1, [CVI_ST], 0>,
  1112:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  1113:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  1114:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1115: 
  1116:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  1117:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1118:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  1119:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1120: 
  1121:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  1122:       [InstrStage<1, [SLOT0], 0>,
  1123:        InstrStage<1, [CVI_ST], 0>,
  1124:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  1125:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1126: 
  1127:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  1128:       [InstrStage<1, [SLOT0], 0>,
  1129:        InstrStage<1, [CVI_ST], 0>,
  1130:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  1131:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1132: 
  1133:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  1134:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1135:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  1136:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1137: 
  1138:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  1139:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1140:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  1141:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1142: 
  1143:     InstrItinData <tc_a28f32b5, /*SLOT1,LOAD,VA*/
  1144:       [InstrStage<1, [SLOT1], 0>,
  1145:        InstrStage<1, [CVI_LD], 0>,
  1146:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  1147:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1148: 
  1149:     InstrItinData <tc_a69eeee1, /*SLOT1,LOAD,VA_DV*/
  1150:       [InstrStage<1, [SLOT1], 0>,
  1151:        InstrStage<1, [CVI_LD], 0>,
  1152:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  1153:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1154: 
  1155:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  1156:       [InstrStage<1, [SLOT0], 0>,
  1157:        InstrStage<1, [SLOT1], 0>,
  1158:        InstrStage<1, [CVI_LD], 0>,
  1159:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  1160:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  1161: 
  1162:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  1163:       [InstrStage<1, [SLOT0], 0>,
  1164:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  1165:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1166: 
  1167:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  1168:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1169:        InstrStage<1, [CVI_LD], 0>,
  1170:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  1171:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1172: 
  1173:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  1174:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1175:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  1176:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1177: 
  1178:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  1179:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1180:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  1181:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1182: 
  1183:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  1184:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1185:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  1186:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1187: 
  1188:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  1189:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1190:        InstrStage<1, [CVI_ALL]>], [2],
  1191:       [Hex_FWD]>,
  1192: 
  1193:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  1194:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1195:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  1196:       [HVX_FWD, HVX_FWD]>,
  1197: 
  1198:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  1199:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1200:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  1201:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1202: 
  1203:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  1204:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1205:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  1206:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1207: 
  1208:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  1209:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1210:        InstrStage<1, [CVI_LD], 0>,
  1211:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  1212:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  1213: 
  1214:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  1215:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1216:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  1217:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1218: 
  1219:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  1220:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1221:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  1222:       [HVX_FWD, Hex_FWD]>,
  1223: 
  1224:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  1225:       [InstrStage<1, [SLOT0], 0>,
  1226:        InstrStage<1, [CVI_ST], 0>,
  1227:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  1228:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1229: 
  1230:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  1231:       [InstrStage<1, [SLOT0], 0>,
  1232:        InstrStage<1, [SLOT1], 0>,
  1233:        InstrStage<1, [CVI_ST], 0>,
  1234:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  1235:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1236: 
  1237:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  1238:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1239:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  1240:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  1241:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1242: 
  1243:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  1244:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1245:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  1246:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1247: 
  1248:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  1249:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1250:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1251-1500 / 第 1251-1500 行

```tablegen
  1251:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1252: 
  1253:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  1254:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1255:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  1256:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1257: 
  1258:     InstrItinData <tc_dcca380f, /*SLOT23,VX_DV*/
  1259:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1260:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  1261:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1262: 
  1263:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  1264:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1265:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  1266:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1267: 
  1268:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  1269:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1270:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  1271:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1272: 
  1273:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  1274:       [InstrStage<1, [SLOT0], 0>,
  1275:        InstrStage<1, [SLOT1], 0>,
  1276:        InstrStage<1, [CVI_ST], 0>,
  1277:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  1278:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1279: 
  1280:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  1281:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
  1282:       [HVX_FWD, HVX_FWD]>,
  1283: 
  1284:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  1285:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1286:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  1287:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1288: 
  1289:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  1290:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1291:        InstrStage<1, [CVI_ALL]>], [3],
  1292:       [HVX_FWD]>,
  1293: 
  1294:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  1295:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1296:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  1297:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  1298: 
  1299:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  1300:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1301:        InstrStage<1, [CVI_ZW]>], [1, 2],
  1302:       [Hex_FWD, Hex_FWD]>,
  1303: 
  1304:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  1305:       [InstrStage<1, [SLOT0], 0>,
  1306:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  1307:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1308: 
  1309:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  1310:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1311:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  1312:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1313: 
  1314:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  1315:       [InstrStage<1, [SLOT2], 0>,
  1316:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  1317:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1318: 
  1319:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  1320:       [InstrStage<1, [SLOT0], 0>,
  1321:        InstrStage<1, [SLOT1], 0>,
  1322:        InstrStage<1, [CVI_ST], 0>,
  1323:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  1324:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  1325:   ];
  1326: }
  1327: 
  1328: class DepHVXItinV62 {
  1329:   list<InstrItinData> DepHVXItinV62_list = [
  1330:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  1331:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1332:        InstrStage<1, [CVI_LD], 0>,
  1333:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  1334:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  1335:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  1336: 
  1337:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  1338:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1339:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  1340:       [HVX_FWD, HVX_FWD]>,
  1341: 
  1342:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  1343:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1344:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  1345:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1346: 
  1347:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  1348:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1349:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  1350:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1351: 
  1352:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  1353:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1354:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  1355:       [HVX_FWD, HVX_FWD]>,
  1356: 
  1357:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  1358:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1359:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  1360:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1361: 
  1362:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  1363:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1364:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  1365:       [HVX_FWD, HVX_FWD]>,
  1366: 
  1367:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  1368:       [InstrStage<1, [SLOT0], 0>,
  1369:        InstrStage<1, [SLOT1], 0>,
  1370:        InstrStage<1, [CVI_ST], 0>,
  1371:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  1372:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1373: 
  1374:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  1375:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1376:        InstrStage<1, [CVI_ALL]>], [],
  1377:       []>,
  1378: 
  1379:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  1380:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1381:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  1382:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  1383:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1384: 
  1385:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  1386:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1387:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  1388:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1389: 
  1390:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  1391:       [InstrStage<1, [SLOT0], 0>,
  1392:        InstrStage<1, [CVI_ST], 0>,
  1393:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  1394:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1395: 
  1396:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  1397:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1398:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  1399:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  1400: 
  1401:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  1402:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1403:        InstrStage<1, [CVI_LD], 0>,
  1404:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  1405:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1406: 
  1407:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  1408:       [InstrStage<1, [SLOT0], 0>,
  1409:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  1410:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1411: 
  1412:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  1413:       [InstrStage<1, [SLOT0], 0>,
  1414:        InstrStage<1, [CVI_ST], 0>,
  1415:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  1416:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  1417:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1418: 
  1419:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  1420:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1421:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  1422:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1423: 
  1424:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  1425:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1426:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  1427:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  1428: 
  1429:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  1430:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1431:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  1432:       [HVX_FWD, HVX_FWD]>,
  1433: 
  1434:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  1435:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1436:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  1437:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1438: 
  1439:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  1440:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1441:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  1442:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1443: 
  1444:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  1445:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1446:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  1447:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1448: 
  1449:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  1450:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1451:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  1452:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1453: 
  1454:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  1455:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1456:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  1457:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1458: 
  1459:     InstrItinData <tc_37820f4c, /*SLOT23,VX_DV*/
  1460:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1461:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  1462:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1463: 
  1464:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  1465:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1466:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  1467:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1468: 
  1469:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  1470:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1471:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  1472:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  1473: 
  1474:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  1475:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1476:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  1477:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1478: 
  1479:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  1480:       [InstrStage<1, [SLOT0], 0>,
  1481:        InstrStage<1, [SLOT1], 0>,
  1482:        InstrStage<1, [CVI_LD], 0>,
  1483:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  1484:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1485: 
  1486:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  1487:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1488:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  1489:       [HVX_FWD, HVX_FWD]>,
  1490: 
  1491:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  1492:       [InstrStage<1, [SLOT0], 0>,
  1493:        InstrStage<1, [CVI_ST]>], [1, 2],
  1494:       [Hex_FWD, Hex_FWD]>,
  1495: 
  1496:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  1497:       [InstrStage<1, [SLOT0], 0>,
  1498:        InstrStage<1, [CVI_ST], 0>,
  1499:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  1500:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
```
- EN: It declares types such as DepHVXItinV62, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV62; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV62 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV62 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 1501-1750 / 第 1501-1750 行

```tablegen
  1501: 
  1502:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  1503:       [InstrStage<1, [SLOT0], 0>,
  1504:        InstrStage<1, [CVI_ST], 0>,
  1505:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  1506:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1507: 
  1508:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  1509:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1510:        InstrStage<1, [CVI_LD], 0>,
  1511:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  1512:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1513: 
  1514:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  1515:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1516:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  1517:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1518: 
  1519:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  1520:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1521:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  1522:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1523: 
  1524:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  1525:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1526:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  1527:       [HVX_FWD, HVX_FWD]>,
  1528: 
  1529:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  1530:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1531:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  1532:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  1533: 
  1534:     InstrItinData <tc_531b383c, /*SLOT0123*/
  1535:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
  1536:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1537: 
  1538:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  1539:       [InstrStage<1, [SLOT0], 0>,
  1540:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  1541:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  1542: 
  1543:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  1544:       [InstrStage<1, [SLOT0], 0>,
  1545:        InstrStage<1, [CVI_ST], 0>,
  1546:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  1547:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1548: 
  1549:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  1550:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1551:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  1552:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1553: 
  1554:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  1555:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1556:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  1557:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1558: 
  1559:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  1560:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1561:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  1562:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1563: 
  1564:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  1565:       [InstrStage<1, [SLOT0], 0>,
  1566:        InstrStage<1, [CVI_ST], 0>,
  1567:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  1568:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1569: 
  1570:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  1571:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1572:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  1573:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1574: 
  1575:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  1576:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1577:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  1578:       [HVX_FWD, Hex_FWD]>,
  1579: 
  1580:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  1581:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1582:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  1583:       [HVX_FWD, HVX_FWD]>,
  1584: 
  1585:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  1586:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1587:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  1588:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1589: 
  1590:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  1591:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1592:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  1593:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1594: 
  1595:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  1596:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1597:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  1598:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1599: 
  1600:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  1601:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1602:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  1603:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1604: 
  1605:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  1606:       [InstrStage<1, [SLOT0], 0>,
  1607:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  1608:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1609: 
  1610:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  1611:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1612:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  1613:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1614: 
  1615:     InstrItinData <tc_7095ecba, /*SLOT1,LOAD,VA_DV*/
  1616:       [InstrStage<1, [SLOT1], 0>,
  1617:        InstrStage<1, [CVI_LD], 0>,
  1618:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  1619:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1620: 
  1621:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  1622:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1623:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  1624:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1625: 
  1626:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  1627:       [InstrStage<1, [SLOT0], 0>,
  1628:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  1629:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1630: 
  1631:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  1632:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1633:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  1634:       [HVX_FWD]>,
  1635: 
  1636:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  1637:       [InstrStage<1, [SLOT0], 0>,
  1638:        InstrStage<1, [CVI_ST], 0>,
  1639:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  1640:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1641: 
  1642:     InstrItinData <tc_72e2b393, /*SLOT23,VX_DV*/
  1643:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1644:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  1645:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1646: 
  1647:     InstrItinData <tc_73efe966, /*SLOT23,VX_DV*/
  1648:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1649:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  1650:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1651: 
  1652:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  1653:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1654:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  1655:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1656: 
  1657:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  1658:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1659:        InstrStage<1, [CVI_ALL]>], [3, 2],
  1660:       [HVX_FWD, Hex_FWD]>,
  1661: 
  1662:     InstrItinData <tc_7d68d5c2, /*SLOT1,LOAD,VA*/
  1663:       [InstrStage<1, [SLOT1], 0>,
  1664:        InstrStage<1, [CVI_LD], 0>,
  1665:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  1666:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1667: 
  1668:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  1669:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1670:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  1671:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1672: 
  1673:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  1674:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1675:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  1676:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1677: 
  1678:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  1679:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1680:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  1681:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1682: 
  1683:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  1684:       [InstrStage<1, [SLOT0], 0>,
  1685:        InstrStage<1, [CVI_ST], 0>,
  1686:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  1687:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1688: 
  1689:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  1690:       [InstrStage<1, [SLOT2], 0>,
  1691:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  1692:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1693: 
  1694:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  1695:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1696:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  1697:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1698: 
  1699:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  1700:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1701:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  1702:       [HVX_FWD, HVX_FWD]>,
  1703: 
  1704:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  1705:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1706:        InstrStage<1, [CVI_LD], 0>,
  1707:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  1708:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  1709:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1710: 
  1711:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  1712:       [InstrStage<1, [SLOT0], 0>,
  1713:        InstrStage<1, [CVI_ST], 0>,
  1714:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  1715:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  1716:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1717: 
  1718:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  1719:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1720:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  1721:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1722: 
  1723:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  1724:       [InstrStage<1, [SLOT0], 0>,
  1725:        InstrStage<1, [CVI_ST], 0>,
  1726:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  1727:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1728: 
  1729:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  1730:       [InstrStage<1, [SLOT0], 0>,
  1731:        InstrStage<1, [CVI_ST], 0>,
  1732:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  1733:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1734: 
  1735:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  1736:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1737:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  1738:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1739: 
  1740:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  1741:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1742:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  1743:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1744: 
  1745:     InstrItinData <tc_a28f32b5, /*SLOT1,LOAD,VA*/
  1746:       [InstrStage<1, [SLOT1], 0>,
  1747:        InstrStage<1, [CVI_LD], 0>,
  1748:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  1749:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1750: 
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1751-2000 / 第 1751-2000 行

```tablegen
  1751:     InstrItinData <tc_a69eeee1, /*SLOT1,LOAD,VA_DV*/
  1752:       [InstrStage<1, [SLOT1], 0>,
  1753:        InstrStage<1, [CVI_LD], 0>,
  1754:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  1755:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1756: 
  1757:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  1758:       [InstrStage<1, [SLOT0], 0>,
  1759:        InstrStage<1, [SLOT1], 0>,
  1760:        InstrStage<1, [CVI_LD], 0>,
  1761:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  1762:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  1763: 
  1764:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  1765:       [InstrStage<1, [SLOT0], 0>,
  1766:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  1767:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1768: 
  1769:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  1770:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1771:        InstrStage<1, [CVI_LD], 0>,
  1772:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  1773:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1774: 
  1775:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  1776:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1777:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  1778:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1779: 
  1780:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  1781:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1782:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  1783:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  1784: 
  1785:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  1786:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1787:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  1788:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1789: 
  1790:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  1791:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1792:        InstrStage<1, [CVI_ALL]>], [2],
  1793:       [Hex_FWD]>,
  1794: 
  1795:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  1796:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1797:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  1798:       [HVX_FWD, HVX_FWD]>,
  1799: 
  1800:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  1801:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1802:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  1803:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1804: 
  1805:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  1806:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1807:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  1808:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1809: 
  1810:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  1811:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1812:        InstrStage<1, [CVI_LD], 0>,
  1813:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  1814:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  1815: 
  1816:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  1817:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1818:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  1819:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1820: 
  1821:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  1822:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1823:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  1824:       [HVX_FWD, Hex_FWD]>,
  1825: 
  1826:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  1827:       [InstrStage<1, [SLOT0], 0>,
  1828:        InstrStage<1, [CVI_ST], 0>,
  1829:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  1830:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  1831: 
  1832:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  1833:       [InstrStage<1, [SLOT0], 0>,
  1834:        InstrStage<1, [SLOT1], 0>,
  1835:        InstrStage<1, [CVI_ST], 0>,
  1836:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  1837:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1838: 
  1839:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  1840:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1841:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  1842:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  1843:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1844: 
  1845:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  1846:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1847:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  1848:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1849: 
  1850:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  1851:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1852:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  1853:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1854: 
  1855:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  1856:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1857:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  1858:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1859: 
  1860:     InstrItinData <tc_dcca380f, /*SLOT23,VX_DV*/
  1861:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1862:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  1863:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1864: 
  1865:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  1866:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1867:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  1868:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1869: 
  1870:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  1871:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1872:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  1873:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1874: 
  1875:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  1876:       [InstrStage<1, [SLOT0], 0>,
  1877:        InstrStage<1, [SLOT1], 0>,
  1878:        InstrStage<1, [CVI_ST], 0>,
  1879:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  1880:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1881: 
  1882:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  1883:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
  1884:       [HVX_FWD, HVX_FWD]>,
  1885: 
  1886:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  1887:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1888:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  1889:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1890: 
  1891:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  1892:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1893:        InstrStage<1, [CVI_ALL]>], [3],
  1894:       [HVX_FWD]>,
  1895: 
  1896:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  1897:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1898:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  1899:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  1900: 
  1901:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  1902:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1903:        InstrStage<1, [CVI_ZW]>], [1, 2],
  1904:       [Hex_FWD, Hex_FWD]>,
  1905: 
  1906:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  1907:       [InstrStage<1, [SLOT0], 0>,
  1908:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  1909:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1910: 
  1911:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  1912:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1913:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  1914:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1915: 
  1916:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  1917:       [InstrStage<1, [SLOT2], 0>,
  1918:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  1919:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1920: 
  1921:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  1922:       [InstrStage<1, [SLOT0], 0>,
  1923:        InstrStage<1, [SLOT1], 0>,
  1924:        InstrStage<1, [CVI_ST], 0>,
  1925:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  1926:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  1927:   ];
  1928: }
  1929: 
  1930: class DepHVXItinV65 {
  1931:   list<InstrItinData> DepHVXItinV65_list = [
  1932:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  1933:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  1934:        InstrStage<1, [CVI_LD], 0>,
  1935:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  1936:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  1937:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  1938: 
  1939:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  1940:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1941:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  1942:       [HVX_FWD, HVX_FWD]>,
  1943: 
  1944:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  1945:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1946:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  1947:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  1948: 
  1949:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  1950:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1951:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  1952:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  1953: 
  1954:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  1955:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1956:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  1957:       [HVX_FWD, HVX_FWD]>,
  1958: 
  1959:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  1960:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1961:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  1962:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  1963: 
  1964:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  1965:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1966:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  1967:       [HVX_FWD, HVX_FWD]>,
  1968: 
  1969:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  1970:       [InstrStage<1, [SLOT0], 0>,
  1971:        InstrStage<1, [SLOT1], 0>,
  1972:        InstrStage<1, [CVI_ST], 0>,
  1973:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  1974:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1975: 
  1976:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  1977:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1978:        InstrStage<1, [CVI_ALL]>], [],
  1979:       []>,
  1980: 
  1981:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  1982:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  1983:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  1984:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  1985:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1986: 
  1987:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  1988:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  1989:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  1990:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  1991: 
  1992:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  1993:       [InstrStage<1, [SLOT0], 0>,
  1994:        InstrStage<1, [CVI_ST], 0>,
  1995:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  1996:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  1997: 
  1998:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  1999:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2000:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
```
- EN: It declares types such as DepHVXItinV65, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV65; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV65 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV65 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 2001-2250 / 第 2001-2250 行

```tablegen
  2001:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  2002: 
  2003:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  2004:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2005:        InstrStage<1, [CVI_LD], 0>,
  2006:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  2007:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2008: 
  2009:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  2010:       [InstrStage<1, [SLOT0], 0>,
  2011:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  2012:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2013: 
  2014:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  2015:       [InstrStage<1, [SLOT0], 0>,
  2016:        InstrStage<1, [CVI_ST], 0>,
  2017:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  2018:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  2019:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2020: 
  2021:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  2022:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2023:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  2024:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2025: 
  2026:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  2027:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2028:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  2029:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  2030: 
  2031:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  2032:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2033:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  2034:       [HVX_FWD, HVX_FWD]>,
  2035: 
  2036:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  2037:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2038:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  2039:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2040: 
  2041:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  2042:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2043:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  2044:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2045: 
  2046:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  2047:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2048:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  2049:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2050: 
  2051:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  2052:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2053:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  2054:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2055: 
  2056:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  2057:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2058:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  2059:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2060: 
  2061:     InstrItinData <tc_37820f4c, /*SLOT23,VX_DV*/
  2062:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2063:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  2064:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2065: 
  2066:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  2067:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2068:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  2069:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2070: 
  2071:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  2072:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2073:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  2074:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  2075: 
  2076:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  2077:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2078:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  2079:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2080: 
  2081:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  2082:       [InstrStage<1, [SLOT0], 0>,
  2083:        InstrStage<1, [SLOT1], 0>,
  2084:        InstrStage<1, [CVI_LD], 0>,
  2085:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  2086:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2087: 
  2088:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  2089:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2090:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  2091:       [HVX_FWD, HVX_FWD]>,
  2092: 
  2093:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  2094:       [InstrStage<1, [SLOT0], 0>,
  2095:        InstrStage<1, [CVI_ST]>], [1, 2],
  2096:       [Hex_FWD, Hex_FWD]>,
  2097: 
  2098:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  2099:       [InstrStage<1, [SLOT0], 0>,
  2100:        InstrStage<1, [CVI_ST], 0>,
  2101:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  2102:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2103: 
  2104:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  2105:       [InstrStage<1, [SLOT0], 0>,
  2106:        InstrStage<1, [CVI_ST], 0>,
  2107:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  2108:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2109: 
  2110:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  2111:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2112:        InstrStage<1, [CVI_LD], 0>,
  2113:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  2114:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2115: 
  2116:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  2117:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2118:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  2119:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2120: 
  2121:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  2122:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2123:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  2124:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2125: 
  2126:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  2127:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2128:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  2129:       [HVX_FWD, HVX_FWD]>,
  2130: 
  2131:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  2132:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2133:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  2134:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  2135: 
  2136:     InstrItinData <tc_531b383c, /*SLOT0123*/
  2137:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
  2138:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2139: 
  2140:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  2141:       [InstrStage<1, [SLOT0], 0>,
  2142:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  2143:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  2144: 
  2145:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  2146:       [InstrStage<1, [SLOT0], 0>,
  2147:        InstrStage<1, [CVI_ST], 0>,
  2148:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  2149:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2150: 
  2151:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  2152:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2153:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  2154:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2155: 
  2156:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  2157:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2158:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  2159:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2160: 
  2161:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  2162:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2163:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  2164:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2165: 
  2166:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  2167:       [InstrStage<1, [SLOT0], 0>,
  2168:        InstrStage<1, [CVI_ST], 0>,
  2169:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  2170:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2171: 
  2172:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  2173:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2174:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  2175:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2176: 
  2177:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  2178:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2179:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  2180:       [HVX_FWD, Hex_FWD]>,
  2181: 
  2182:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  2183:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2184:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  2185:       [HVX_FWD, HVX_FWD]>,
  2186: 
  2187:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  2188:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2189:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  2190:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2191: 
  2192:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  2193:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2194:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  2195:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2196: 
  2197:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  2198:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2199:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  2200:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2201: 
  2202:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  2203:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2204:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  2205:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2206: 
  2207:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  2208:       [InstrStage<1, [SLOT0], 0>,
  2209:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  2210:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2211: 
  2212:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  2213:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2214:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  2215:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2216: 
  2217:     InstrItinData <tc_7095ecba, /*SLOT1,LOAD,VA_DV*/
  2218:       [InstrStage<1, [SLOT1], 0>,
  2219:        InstrStage<1, [CVI_LD], 0>,
  2220:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  2221:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2222: 
  2223:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  2224:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2225:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  2226:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2227: 
  2228:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  2229:       [InstrStage<1, [SLOT0], 0>,
  2230:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  2231:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2232: 
  2233:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  2234:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2235:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  2236:       [HVX_FWD]>,
  2237: 
  2238:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  2239:       [InstrStage<1, [SLOT0], 0>,
  2240:        InstrStage<1, [CVI_ST], 0>,
  2241:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  2242:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2243: 
  2244:     InstrItinData <tc_72e2b393, /*SLOT23,VX_DV*/
  2245:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2246:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  2247:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2248: 
  2249:     InstrItinData <tc_73efe966, /*SLOT23,VX_DV*/
  2250:       [InstrStage<1, [SLOT2, SLOT3], 0>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2251-2500 / 第 2251-2500 行

```tablegen
  2251:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  2252:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2253: 
  2254:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  2255:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2256:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  2257:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2258: 
  2259:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  2260:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2261:        InstrStage<1, [CVI_ALL]>], [3, 2],
  2262:       [HVX_FWD, Hex_FWD]>,
  2263: 
  2264:     InstrItinData <tc_7d68d5c2, /*SLOT1,LOAD,VA*/
  2265:       [InstrStage<1, [SLOT1], 0>,
  2266:        InstrStage<1, [CVI_LD], 0>,
  2267:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  2268:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2269: 
  2270:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  2271:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2272:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  2273:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2274: 
  2275:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  2276:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2277:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  2278:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2279: 
  2280:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  2281:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2282:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  2283:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2284: 
  2285:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  2286:       [InstrStage<1, [SLOT0], 0>,
  2287:        InstrStage<1, [CVI_ST], 0>,
  2288:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  2289:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2290: 
  2291:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  2292:       [InstrStage<1, [SLOT2], 0>,
  2293:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  2294:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2295: 
  2296:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  2297:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2298:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  2299:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2300: 
  2301:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  2302:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2303:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  2304:       [HVX_FWD, HVX_FWD]>,
  2305: 
  2306:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  2307:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2308:        InstrStage<1, [CVI_LD], 0>,
  2309:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  2310:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  2311:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2312: 
  2313:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  2314:       [InstrStage<1, [SLOT0], 0>,
  2315:        InstrStage<1, [CVI_ST], 0>,
  2316:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  2317:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  2318:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2319: 
  2320:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  2321:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2322:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  2323:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2324: 
  2325:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  2326:       [InstrStage<1, [SLOT0], 0>,
  2327:        InstrStage<1, [CVI_ST], 0>,
  2328:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  2329:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2330: 
  2331:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  2332:       [InstrStage<1, [SLOT0], 0>,
  2333:        InstrStage<1, [CVI_ST], 0>,
  2334:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  2335:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2336: 
  2337:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  2338:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2339:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  2340:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2341: 
  2342:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  2343:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2344:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  2345:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2346: 
  2347:     InstrItinData <tc_a28f32b5, /*SLOT1,LOAD,VA*/
  2348:       [InstrStage<1, [SLOT1], 0>,
  2349:        InstrStage<1, [CVI_LD], 0>,
  2350:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  2351:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2352: 
  2353:     InstrItinData <tc_a69eeee1, /*SLOT1,LOAD,VA_DV*/
  2354:       [InstrStage<1, [SLOT1], 0>,
  2355:        InstrStage<1, [CVI_LD], 0>,
  2356:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  2357:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2358: 
  2359:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  2360:       [InstrStage<1, [SLOT0], 0>,
  2361:        InstrStage<1, [SLOT1], 0>,
  2362:        InstrStage<1, [CVI_LD], 0>,
  2363:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  2364:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  2365: 
  2366:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  2367:       [InstrStage<1, [SLOT0], 0>,
  2368:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  2369:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2370: 
  2371:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  2372:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2373:        InstrStage<1, [CVI_LD], 0>,
  2374:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  2375:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2376: 
  2377:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  2378:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2379:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  2380:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2381: 
  2382:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  2383:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2384:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  2385:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2386: 
  2387:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  2388:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2389:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  2390:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2391: 
  2392:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  2393:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2394:        InstrStage<1, [CVI_ALL]>], [2],
  2395:       [Hex_FWD]>,
  2396: 
  2397:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  2398:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2399:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  2400:       [HVX_FWD, HVX_FWD]>,
  2401: 
  2402:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  2403:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2404:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  2405:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2406: 
  2407:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  2408:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2409:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  2410:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2411: 
  2412:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  2413:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2414:        InstrStage<1, [CVI_LD], 0>,
  2415:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  2416:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  2417: 
  2418:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  2419:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2420:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  2421:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2422: 
  2423:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  2424:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2425:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  2426:       [HVX_FWD, Hex_FWD]>,
  2427: 
  2428:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  2429:       [InstrStage<1, [SLOT0], 0>,
  2430:        InstrStage<1, [CVI_ST], 0>,
  2431:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  2432:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2433: 
  2434:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  2435:       [InstrStage<1, [SLOT0], 0>,
  2436:        InstrStage<1, [SLOT1], 0>,
  2437:        InstrStage<1, [CVI_ST], 0>,
  2438:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  2439:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2440: 
  2441:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  2442:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2443:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  2444:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  2445:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2446: 
  2447:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  2448:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2449:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  2450:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2451: 
  2452:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  2453:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2454:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  2455:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2456: 
  2457:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  2458:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2459:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  2460:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2461: 
  2462:     InstrItinData <tc_dcca380f, /*SLOT23,VX_DV*/
  2463:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2464:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  2465:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2466: 
  2467:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  2468:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2469:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  2470:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2471: 
  2472:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  2473:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2474:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  2475:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2476: 
  2477:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  2478:       [InstrStage<1, [SLOT0], 0>,
  2479:        InstrStage<1, [SLOT1], 0>,
  2480:        InstrStage<1, [CVI_ST], 0>,
  2481:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  2482:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2483: 
  2484:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  2485:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
  2486:       [HVX_FWD, HVX_FWD]>,
  2487: 
  2488:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  2489:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2490:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  2491:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2492: 
  2493:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  2494:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2495:        InstrStage<1, [CVI_ALL]>], [3],
  2496:       [HVX_FWD]>,
  2497: 
  2498:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  2499:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2500:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2501-2750 / 第 2501-2750 行

```tablegen
  2501:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  2502: 
  2503:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  2504:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2505:        InstrStage<1, [CVI_ZW]>], [1, 2],
  2506:       [Hex_FWD, Hex_FWD]>,
  2507: 
  2508:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  2509:       [InstrStage<1, [SLOT0], 0>,
  2510:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  2511:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2512: 
  2513:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  2514:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2515:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  2516:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2517: 
  2518:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  2519:       [InstrStage<1, [SLOT2], 0>,
  2520:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  2521:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2522: 
  2523:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  2524:       [InstrStage<1, [SLOT0], 0>,
  2525:        InstrStage<1, [SLOT1], 0>,
  2526:        InstrStage<1, [CVI_ST], 0>,
  2527:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  2528:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  2529:   ];
  2530: }
  2531: 
  2532: class DepHVXItinV66 {
  2533:   list<InstrItinData> DepHVXItinV66_list = [
  2534:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  2535:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2536:        InstrStage<1, [CVI_LD], 0>,
  2537:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  2538:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  2539:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  2540: 
  2541:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  2542:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2543:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  2544:       [HVX_FWD, HVX_FWD]>,
  2545: 
  2546:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  2547:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2548:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  2549:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2550: 
  2551:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  2552:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2553:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  2554:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2555: 
  2556:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  2557:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2558:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  2559:       [HVX_FWD, HVX_FWD]>,
  2560: 
  2561:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  2562:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2563:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  2564:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2565: 
  2566:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  2567:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2568:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  2569:       [HVX_FWD, HVX_FWD]>,
  2570: 
  2571:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  2572:       [InstrStage<1, [SLOT0], 0>,
  2573:        InstrStage<1, [SLOT1], 0>,
  2574:        InstrStage<1, [CVI_ST], 0>,
  2575:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  2576:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2577: 
  2578:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  2579:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2580:        InstrStage<1, [CVI_ALL]>], [],
  2581:       []>,
  2582: 
  2583:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  2584:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2585:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  2586:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  2587:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2588: 
  2589:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  2590:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2591:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  2592:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2593: 
  2594:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  2595:       [InstrStage<1, [SLOT0], 0>,
  2596:        InstrStage<1, [CVI_ST], 0>,
  2597:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  2598:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2599: 
  2600:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  2601:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2602:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  2603:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  2604: 
  2605:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  2606:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2607:        InstrStage<1, [CVI_LD], 0>,
  2608:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  2609:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2610: 
  2611:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  2612:       [InstrStage<1, [SLOT0], 0>,
  2613:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  2614:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2615: 
  2616:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  2617:       [InstrStage<1, [SLOT0], 0>,
  2618:        InstrStage<1, [CVI_ST], 0>,
  2619:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  2620:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  2621:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2622: 
  2623:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  2624:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2625:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  2626:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2627: 
  2628:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  2629:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2630:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  2631:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  2632: 
  2633:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  2634:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2635:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  2636:       [HVX_FWD, HVX_FWD]>,
  2637: 
  2638:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  2639:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2640:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  2641:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2642: 
  2643:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  2644:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2645:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  2646:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2647: 
  2648:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  2649:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2650:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  2651:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2652: 
  2653:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  2654:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2655:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  2656:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2657: 
  2658:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  2659:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2660:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  2661:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2662: 
  2663:     InstrItinData <tc_37820f4c, /*SLOT23,VX_DV*/
  2664:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2665:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  2666:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2667: 
  2668:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  2669:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2670:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  2671:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2672: 
  2673:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  2674:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2675:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  2676:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  2677: 
  2678:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  2679:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2680:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  2681:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2682: 
  2683:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  2684:       [InstrStage<1, [SLOT0], 0>,
  2685:        InstrStage<1, [SLOT1], 0>,
  2686:        InstrStage<1, [CVI_LD], 0>,
  2687:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  2688:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2689: 
  2690:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  2691:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2692:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  2693:       [HVX_FWD, HVX_FWD]>,
  2694: 
  2695:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  2696:       [InstrStage<1, [SLOT0], 0>,
  2697:        InstrStage<1, [CVI_ST]>], [1, 2],
  2698:       [Hex_FWD, Hex_FWD]>,
  2699: 
  2700:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  2701:       [InstrStage<1, [SLOT0], 0>,
  2702:        InstrStage<1, [CVI_ST], 0>,
  2703:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  2704:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2705: 
  2706:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  2707:       [InstrStage<1, [SLOT0], 0>,
  2708:        InstrStage<1, [CVI_ST], 0>,
  2709:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  2710:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2711: 
  2712:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  2713:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2714:        InstrStage<1, [CVI_LD], 0>,
  2715:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  2716:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2717: 
  2718:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  2719:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2720:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  2721:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2722: 
  2723:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  2724:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2725:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  2726:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2727: 
  2728:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  2729:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2730:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  2731:       [HVX_FWD, HVX_FWD]>,
  2732: 
  2733:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  2734:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2735:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  2736:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  2737: 
  2738:     InstrItinData <tc_531b383c, /*SLOT0123*/
  2739:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
  2740:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2741: 
  2742:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  2743:       [InstrStage<1, [SLOT0], 0>,
  2744:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  2745:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  2746: 
  2747:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  2748:       [InstrStage<1, [SLOT0], 0>,
  2749:        InstrStage<1, [CVI_ST], 0>,
  2750:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
```
- EN: It declares types such as DepHVXItinV66, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV66; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV66 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV66 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 2751-3000 / 第 2751-3000 行

```tablegen
  2751:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2752: 
  2753:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  2754:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2755:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  2756:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2757: 
  2758:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  2759:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2760:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  2761:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2762: 
  2763:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  2764:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2765:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  2766:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2767: 
  2768:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  2769:       [InstrStage<1, [SLOT0], 0>,
  2770:        InstrStage<1, [CVI_ST], 0>,
  2771:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  2772:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2773: 
  2774:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  2775:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2776:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  2777:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2778: 
  2779:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  2780:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2781:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  2782:       [HVX_FWD, Hex_FWD]>,
  2783: 
  2784:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  2785:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2786:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  2787:       [HVX_FWD, HVX_FWD]>,
  2788: 
  2789:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  2790:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2791:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  2792:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2793: 
  2794:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  2795:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2796:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  2797:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2798: 
  2799:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  2800:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2801:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  2802:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2803: 
  2804:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  2805:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2806:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  2807:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2808: 
  2809:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  2810:       [InstrStage<1, [SLOT0], 0>,
  2811:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  2812:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2813: 
  2814:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  2815:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2816:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  2817:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2818: 
  2819:     InstrItinData <tc_7095ecba, /*SLOT1,LOAD,VA_DV*/
  2820:       [InstrStage<1, [SLOT1], 0>,
  2821:        InstrStage<1, [CVI_LD], 0>,
  2822:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  2823:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2824: 
  2825:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  2826:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2827:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  2828:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2829: 
  2830:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  2831:       [InstrStage<1, [SLOT0], 0>,
  2832:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  2833:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2834: 
  2835:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  2836:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2837:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  2838:       [HVX_FWD]>,
  2839: 
  2840:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  2841:       [InstrStage<1, [SLOT0], 0>,
  2842:        InstrStage<1, [CVI_ST], 0>,
  2843:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  2844:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2845: 
  2846:     InstrItinData <tc_72e2b393, /*SLOT23,VX_DV*/
  2847:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2848:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  2849:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2850: 
  2851:     InstrItinData <tc_73efe966, /*SLOT23,VX_DV*/
  2852:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2853:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  2854:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2855: 
  2856:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  2857:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2858:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  2859:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2860: 
  2861:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  2862:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2863:        InstrStage<1, [CVI_ALL]>], [3, 2],
  2864:       [HVX_FWD, Hex_FWD]>,
  2865: 
  2866:     InstrItinData <tc_7d68d5c2, /*SLOT1,LOAD,VA*/
  2867:       [InstrStage<1, [SLOT1], 0>,
  2868:        InstrStage<1, [CVI_LD], 0>,
  2869:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  2870:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2871: 
  2872:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  2873:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2874:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  2875:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2876: 
  2877:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  2878:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2879:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  2880:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  2881: 
  2882:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  2883:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2884:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  2885:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2886: 
  2887:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  2888:       [InstrStage<1, [SLOT0], 0>,
  2889:        InstrStage<1, [CVI_ST], 0>,
  2890:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  2891:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2892: 
  2893:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  2894:       [InstrStage<1, [SLOT2], 0>,
  2895:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  2896:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2897: 
  2898:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  2899:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2900:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  2901:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2902: 
  2903:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  2904:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2905:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  2906:       [HVX_FWD, HVX_FWD]>,
  2907: 
  2908:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  2909:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2910:        InstrStage<1, [CVI_LD], 0>,
  2911:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  2912:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  2913:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2914: 
  2915:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  2916:       [InstrStage<1, [SLOT0], 0>,
  2917:        InstrStage<1, [CVI_ST], 0>,
  2918:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  2919:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  2920:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2921: 
  2922:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  2923:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2924:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  2925:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2926: 
  2927:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  2928:       [InstrStage<1, [SLOT0], 0>,
  2929:        InstrStage<1, [CVI_ST], 0>,
  2930:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  2931:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2932: 
  2933:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  2934:       [InstrStage<1, [SLOT0], 0>,
  2935:        InstrStage<1, [CVI_ST], 0>,
  2936:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  2937:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2938: 
  2939:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  2940:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2941:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  2942:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2943: 
  2944:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  2945:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2946:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  2947:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  2948: 
  2949:     InstrItinData <tc_a28f32b5, /*SLOT1,LOAD,VA*/
  2950:       [InstrStage<1, [SLOT1], 0>,
  2951:        InstrStage<1, [CVI_LD], 0>,
  2952:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  2953:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2954: 
  2955:     InstrItinData <tc_a69eeee1, /*SLOT1,LOAD,VA_DV*/
  2956:       [InstrStage<1, [SLOT1], 0>,
  2957:        InstrStage<1, [CVI_LD], 0>,
  2958:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  2959:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  2960: 
  2961:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  2962:       [InstrStage<1, [SLOT0], 0>,
  2963:        InstrStage<1, [SLOT1], 0>,
  2964:        InstrStage<1, [CVI_LD], 0>,
  2965:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  2966:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  2967: 
  2968:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  2969:       [InstrStage<1, [SLOT0], 0>,
  2970:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  2971:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  2972: 
  2973:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  2974:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  2975:        InstrStage<1, [CVI_LD], 0>,
  2976:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  2977:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2978: 
  2979:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  2980:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2981:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  2982:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  2983: 
  2984:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  2985:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2986:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  2987:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  2988: 
  2989:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  2990:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2991:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  2992:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  2993: 
  2994:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  2995:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2996:        InstrStage<1, [CVI_ALL]>], [2],
  2997:       [Hex_FWD]>,
  2998: 
  2999:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  3000:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 3001-3250 / 第 3001-3250 行

```tablegen
  3001:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  3002:       [HVX_FWD, HVX_FWD]>,
  3003: 
  3004:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  3005:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3006:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  3007:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3008: 
  3009:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  3010:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3011:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  3012:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3013: 
  3014:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  3015:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3016:        InstrStage<1, [CVI_LD], 0>,
  3017:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  3018:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  3019: 
  3020:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  3021:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3022:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  3023:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3024: 
  3025:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  3026:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3027:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  3028:       [HVX_FWD, Hex_FWD]>,
  3029: 
  3030:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  3031:       [InstrStage<1, [SLOT0], 0>,
  3032:        InstrStage<1, [CVI_ST], 0>,
  3033:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  3034:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  3035: 
  3036:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  3037:       [InstrStage<1, [SLOT0], 0>,
  3038:        InstrStage<1, [SLOT1], 0>,
  3039:        InstrStage<1, [CVI_ST], 0>,
  3040:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  3041:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3042: 
  3043:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  3044:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3045:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  3046:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  3047:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3048: 
  3049:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  3050:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3051:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  3052:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3053: 
  3054:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  3055:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3056:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  3057:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3058: 
  3059:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  3060:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3061:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  3062:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3063: 
  3064:     InstrItinData <tc_dcca380f, /*SLOT23,VX_DV*/
  3065:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3066:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  3067:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3068: 
  3069:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  3070:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3071:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  3072:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3073: 
  3074:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  3075:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3076:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  3077:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3078: 
  3079:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  3080:       [InstrStage<1, [SLOT0], 0>,
  3081:        InstrStage<1, [SLOT1], 0>,
  3082:        InstrStage<1, [CVI_ST], 0>,
  3083:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  3084:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3085: 
  3086:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  3087:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
  3088:       [HVX_FWD, HVX_FWD]>,
  3089: 
  3090:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  3091:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3092:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  3093:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3094: 
  3095:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  3096:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3097:        InstrStage<1, [CVI_ALL]>], [3],
  3098:       [HVX_FWD]>,
  3099: 
  3100:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  3101:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3102:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  3103:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  3104: 
  3105:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  3106:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3107:        InstrStage<1, [CVI_ZW]>], [1, 2],
  3108:       [Hex_FWD, Hex_FWD]>,
  3109: 
  3110:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  3111:       [InstrStage<1, [SLOT0], 0>,
  3112:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  3113:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3114: 
  3115:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  3116:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3117:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  3118:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3119: 
  3120:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  3121:       [InstrStage<1, [SLOT2], 0>,
  3122:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  3123:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3124: 
  3125:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  3126:       [InstrStage<1, [SLOT0], 0>,
  3127:        InstrStage<1, [SLOT1], 0>,
  3128:        InstrStage<1, [CVI_ST], 0>,
  3129:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  3130:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  3131:   ];
  3132: }
  3133: 
  3134: class DepHVXItinV67 {
  3135:   list<InstrItinData> DepHVXItinV67_list = [
  3136:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  3137:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3138:        InstrStage<1, [CVI_LD], 0>,
  3139:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  3140:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  3141:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  3142: 
  3143:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  3144:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3145:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  3146:       [HVX_FWD, HVX_FWD]>,
  3147: 
  3148:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  3149:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3150:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  3151:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3152: 
  3153:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  3154:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3155:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  3156:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3157: 
  3158:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  3159:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3160:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  3161:       [HVX_FWD, HVX_FWD]>,
  3162: 
  3163:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  3164:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3165:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  3166:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3167: 
  3168:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  3169:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3170:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  3171:       [HVX_FWD, HVX_FWD]>,
  3172: 
  3173:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  3174:       [InstrStage<1, [SLOT0], 0>,
  3175:        InstrStage<1, [SLOT1], 0>,
  3176:        InstrStage<1, [CVI_ST], 0>,
  3177:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  3178:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3179: 
  3180:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  3181:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3182:        InstrStage<1, [CVI_ALL]>], [],
  3183:       []>,
  3184: 
  3185:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  3186:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3187:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  3188:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  3189:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3190: 
  3191:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  3192:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3193:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  3194:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3195: 
  3196:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  3197:       [InstrStage<1, [SLOT0], 0>,
  3198:        InstrStage<1, [CVI_ST], 0>,
  3199:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  3200:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3201: 
  3202:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  3203:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3204:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  3205:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  3206: 
  3207:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  3208:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3209:        InstrStage<1, [CVI_LD], 0>,
  3210:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  3211:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3212: 
  3213:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  3214:       [InstrStage<1, [SLOT0], 0>,
  3215:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  3216:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3217: 
  3218:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  3219:       [InstrStage<1, [SLOT0], 0>,
  3220:        InstrStage<1, [CVI_ST], 0>,
  3221:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  3222:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  3223:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3224: 
  3225:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  3226:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3227:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  3228:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3229: 
  3230:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  3231:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3232:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  3233:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  3234: 
  3235:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  3236:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3237:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  3238:       [HVX_FWD, HVX_FWD]>,
  3239: 
  3240:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  3241:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3242:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  3243:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3244: 
  3245:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  3246:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3247:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  3248:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3249: 
  3250:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
```
- EN: It declares types such as DepHVXItinV67, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV67; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV67 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV67 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 3251-3500 / 第 3251-3500 行

```tablegen
  3251:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3252:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  3253:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3254: 
  3255:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  3256:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3257:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  3258:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3259: 
  3260:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  3261:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3262:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  3263:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3264: 
  3265:     InstrItinData <tc_37820f4c, /*SLOT23,VX_DV*/
  3266:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3267:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  3268:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3269: 
  3270:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  3271:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3272:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  3273:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3274: 
  3275:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  3276:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3277:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  3278:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  3279: 
  3280:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  3281:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3282:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  3283:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3284: 
  3285:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  3286:       [InstrStage<1, [SLOT0], 0>,
  3287:        InstrStage<1, [SLOT1], 0>,
  3288:        InstrStage<1, [CVI_LD], 0>,
  3289:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  3290:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3291: 
  3292:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  3293:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3294:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  3295:       [HVX_FWD, HVX_FWD]>,
  3296: 
  3297:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  3298:       [InstrStage<1, [SLOT0], 0>,
  3299:        InstrStage<1, [CVI_ST]>], [1, 2],
  3300:       [Hex_FWD, Hex_FWD]>,
  3301: 
  3302:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  3303:       [InstrStage<1, [SLOT0], 0>,
  3304:        InstrStage<1, [CVI_ST], 0>,
  3305:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  3306:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3307: 
  3308:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  3309:       [InstrStage<1, [SLOT0], 0>,
  3310:        InstrStage<1, [CVI_ST], 0>,
  3311:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  3312:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3313: 
  3314:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  3315:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3316:        InstrStage<1, [CVI_LD], 0>,
  3317:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  3318:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3319: 
  3320:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  3321:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3322:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  3323:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3324: 
  3325:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  3326:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3327:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  3328:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3329: 
  3330:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  3331:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3332:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  3333:       [HVX_FWD, HVX_FWD]>,
  3334: 
  3335:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  3336:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3337:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  3338:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  3339: 
  3340:     InstrItinData <tc_531b383c, /*SLOT0123*/
  3341:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
  3342:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3343: 
  3344:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  3345:       [InstrStage<1, [SLOT0], 0>,
  3346:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  3347:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  3348: 
  3349:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  3350:       [InstrStage<1, [SLOT0], 0>,
  3351:        InstrStage<1, [CVI_ST], 0>,
  3352:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  3353:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3354: 
  3355:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  3356:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3357:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  3358:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3359: 
  3360:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  3361:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3362:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  3363:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3364: 
  3365:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  3366:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3367:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  3368:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3369: 
  3370:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  3371:       [InstrStage<1, [SLOT0], 0>,
  3372:        InstrStage<1, [CVI_ST], 0>,
  3373:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  3374:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  3375: 
  3376:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  3377:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3378:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  3379:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3380: 
  3381:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  3382:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3383:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  3384:       [HVX_FWD, Hex_FWD]>,
  3385: 
  3386:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  3387:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3388:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  3389:       [HVX_FWD, HVX_FWD]>,
  3390: 
  3391:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  3392:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3393:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  3394:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3395: 
  3396:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  3397:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3398:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  3399:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3400: 
  3401:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  3402:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3403:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  3404:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3405: 
  3406:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  3407:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3408:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  3409:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3410: 
  3411:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  3412:       [InstrStage<1, [SLOT0], 0>,
  3413:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  3414:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3415: 
  3416:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  3417:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3418:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  3419:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3420: 
  3421:     InstrItinData <tc_7095ecba, /*SLOT1,LOAD,VA_DV*/
  3422:       [InstrStage<1, [SLOT1], 0>,
  3423:        InstrStage<1, [CVI_LD], 0>,
  3424:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  3425:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  3426: 
  3427:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  3428:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3429:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  3430:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3431: 
  3432:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  3433:       [InstrStage<1, [SLOT0], 0>,
  3434:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  3435:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3436: 
  3437:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  3438:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3439:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  3440:       [HVX_FWD]>,
  3441: 
  3442:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  3443:       [InstrStage<1, [SLOT0], 0>,
  3444:        InstrStage<1, [CVI_ST], 0>,
  3445:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  3446:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  3447: 
  3448:     InstrItinData <tc_72e2b393, /*SLOT23,VX_DV*/
  3449:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3450:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  3451:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3452: 
  3453:     InstrItinData <tc_73efe966, /*SLOT23,VX_DV*/
  3454:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3455:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  3456:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3457: 
  3458:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  3459:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3460:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  3461:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3462: 
  3463:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  3464:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3465:        InstrStage<1, [CVI_ALL]>], [3, 2],
  3466:       [HVX_FWD, Hex_FWD]>,
  3467: 
  3468:     InstrItinData <tc_7d68d5c2, /*SLOT1,LOAD,VA*/
  3469:       [InstrStage<1, [SLOT1], 0>,
  3470:        InstrStage<1, [CVI_LD], 0>,
  3471:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  3472:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3473: 
  3474:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  3475:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3476:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  3477:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3478: 
  3479:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  3480:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3481:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  3482:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3483: 
  3484:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  3485:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3486:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  3487:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3488: 
  3489:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  3490:       [InstrStage<1, [SLOT0], 0>,
  3491:        InstrStage<1, [CVI_ST], 0>,
  3492:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  3493:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  3494: 
  3495:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  3496:       [InstrStage<1, [SLOT2], 0>,
  3497:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  3498:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3499: 
  3500:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 3501-3750 / 第 3501-3750 行

```tablegen
  3501:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3502:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  3503:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3504: 
  3505:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  3506:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3507:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  3508:       [HVX_FWD, HVX_FWD]>,
  3509: 
  3510:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  3511:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3512:        InstrStage<1, [CVI_LD], 0>,
  3513:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  3514:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  3515:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3516: 
  3517:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  3518:       [InstrStage<1, [SLOT0], 0>,
  3519:        InstrStage<1, [CVI_ST], 0>,
  3520:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  3521:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  3522:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  3523: 
  3524:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  3525:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3526:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  3527:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3528: 
  3529:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  3530:       [InstrStage<1, [SLOT0], 0>,
  3531:        InstrStage<1, [CVI_ST], 0>,
  3532:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  3533:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  3534: 
  3535:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  3536:       [InstrStage<1, [SLOT0], 0>,
  3537:        InstrStage<1, [CVI_ST], 0>,
  3538:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  3539:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3540: 
  3541:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  3542:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3543:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  3544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3545: 
  3546:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  3547:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3548:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  3549:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3550: 
  3551:     InstrItinData <tc_a28f32b5, /*SLOT1,LOAD,VA*/
  3552:       [InstrStage<1, [SLOT1], 0>,
  3553:        InstrStage<1, [CVI_LD], 0>,
  3554:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  3555:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  3556: 
  3557:     InstrItinData <tc_a69eeee1, /*SLOT1,LOAD,VA_DV*/
  3558:       [InstrStage<1, [SLOT1], 0>,
  3559:        InstrStage<1, [CVI_LD], 0>,
  3560:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  3561:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3562: 
  3563:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  3564:       [InstrStage<1, [SLOT0], 0>,
  3565:        InstrStage<1, [SLOT1], 0>,
  3566:        InstrStage<1, [CVI_LD], 0>,
  3567:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  3568:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  3569: 
  3570:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  3571:       [InstrStage<1, [SLOT0], 0>,
  3572:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  3573:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  3574: 
  3575:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  3576:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3577:        InstrStage<1, [CVI_LD], 0>,
  3578:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  3579:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3580: 
  3581:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  3582:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3583:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  3584:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3585: 
  3586:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  3587:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3588:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  3589:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  3590: 
  3591:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  3592:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3593:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  3594:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3595: 
  3596:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  3597:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3598:        InstrStage<1, [CVI_ALL]>], [2],
  3599:       [Hex_FWD]>,
  3600: 
  3601:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  3602:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3603:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  3604:       [HVX_FWD, HVX_FWD]>,
  3605: 
  3606:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  3607:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3608:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  3609:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3610: 
  3611:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  3612:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3613:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  3614:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3615: 
  3616:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  3617:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3618:        InstrStage<1, [CVI_LD], 0>,
  3619:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  3620:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  3621: 
  3622:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  3623:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3624:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  3625:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3626: 
  3627:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  3628:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3629:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  3630:       [HVX_FWD, Hex_FWD]>,
  3631: 
  3632:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  3633:       [InstrStage<1, [SLOT0], 0>,
  3634:        InstrStage<1, [CVI_ST], 0>,
  3635:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  3636:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  3637: 
  3638:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  3639:       [InstrStage<1, [SLOT0], 0>,
  3640:        InstrStage<1, [SLOT1], 0>,
  3641:        InstrStage<1, [CVI_ST], 0>,
  3642:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  3643:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3644: 
  3645:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  3646:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3647:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  3648:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  3649:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3650: 
  3651:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  3652:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3653:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  3654:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3655: 
  3656:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  3657:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3658:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  3659:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3660: 
  3661:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  3662:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3663:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  3664:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3665: 
  3666:     InstrItinData <tc_dcca380f, /*SLOT23,VX_DV*/
  3667:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3668:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  3669:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3670: 
  3671:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  3672:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3673:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  3674:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3675: 
  3676:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  3677:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3678:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  3679:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3680: 
  3681:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  3682:       [InstrStage<1, [SLOT0], 0>,
  3683:        InstrStage<1, [SLOT1], 0>,
  3684:        InstrStage<1, [CVI_ST], 0>,
  3685:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  3686:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3687: 
  3688:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  3689:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
  3690:       [HVX_FWD, HVX_FWD]>,
  3691: 
  3692:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  3693:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3694:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  3695:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3696: 
  3697:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  3698:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3699:        InstrStage<1, [CVI_ALL]>], [3],
  3700:       [HVX_FWD]>,
  3701: 
  3702:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  3703:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3704:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  3705:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  3706: 
  3707:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  3708:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3709:        InstrStage<1, [CVI_ZW]>], [1, 2],
  3710:       [Hex_FWD, Hex_FWD]>,
  3711: 
  3712:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  3713:       [InstrStage<1, [SLOT0], 0>,
  3714:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  3715:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3716: 
  3717:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  3718:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3719:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  3720:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3721: 
  3722:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  3723:       [InstrStage<1, [SLOT2], 0>,
  3724:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  3725:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3726: 
  3727:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  3728:       [InstrStage<1, [SLOT0], 0>,
  3729:        InstrStage<1, [SLOT1], 0>,
  3730:        InstrStage<1, [CVI_ST], 0>,
  3731:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  3732:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  3733:   ];
  3734: }
  3735: 
  3736: class DepHVXItinV68 {
  3737:   list<InstrItinData> DepHVXItinV68_list = [
  3738:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  3739:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3740:        InstrStage<1, [CVI_LD], 0>,
  3741:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  3742:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  3743:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  3744: 
  3745:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  3746:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3747:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  3748:       [HVX_FWD, HVX_FWD]>,
  3749: 
  3750:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
```
- EN: It declares types such as DepHVXItinV68, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV68; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV68 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV68 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 3751-4000 / 第 3751-4000 行

```tablegen
  3751:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3752:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  3753:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3754: 
  3755:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  3756:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3757:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  3758:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3759: 
  3760:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  3761:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3762:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  3763:       [HVX_FWD, HVX_FWD]>,
  3764: 
  3765:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  3766:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3767:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  3768:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3769: 
  3770:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  3771:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3772:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  3773:       [HVX_FWD, HVX_FWD]>,
  3774: 
  3775:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  3776:       [InstrStage<1, [SLOT0], 0>,
  3777:        InstrStage<1, [SLOT1], 0>,
  3778:        InstrStage<1, [CVI_ST], 0>,
  3779:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  3780:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3781: 
  3782:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  3783:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3784:        InstrStage<1, [CVI_ALL]>], [],
  3785:       []>,
  3786: 
  3787:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  3788:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3789:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  3790:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  3791:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3792: 
  3793:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  3794:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3795:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  3796:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3797: 
  3798:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  3799:       [InstrStage<1, [SLOT0], 0>,
  3800:        InstrStage<1, [CVI_ST], 0>,
  3801:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  3802:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3803: 
  3804:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  3805:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3806:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  3807:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  3808: 
  3809:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  3810:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3811:        InstrStage<1, [CVI_LD], 0>,
  3812:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  3813:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3814: 
  3815:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  3816:       [InstrStage<1, [SLOT0], 0>,
  3817:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  3818:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3819: 
  3820:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  3821:       [InstrStage<1, [SLOT0], 0>,
  3822:        InstrStage<1, [CVI_ST], 0>,
  3823:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  3824:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  3825:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3826: 
  3827:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  3828:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3829:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  3830:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3831: 
  3832:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  3833:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3834:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  3835:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  3836: 
  3837:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  3838:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3839:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  3840:       [HVX_FWD, HVX_FWD]>,
  3841: 
  3842:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  3843:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3844:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  3845:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3846: 
  3847:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  3848:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3849:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  3850:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3851: 
  3852:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  3853:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3854:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  3855:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3856: 
  3857:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  3858:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3859:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  3860:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3861: 
  3862:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  3863:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3864:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  3865:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3866: 
  3867:     InstrItinData <tc_37820f4c, /*SLOT23,VX_DV*/
  3868:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3869:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  3870:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  3871: 
  3872:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  3873:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3874:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  3875:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3876: 
  3877:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  3878:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3879:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  3880:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  3881: 
  3882:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  3883:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3884:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  3885:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3886: 
  3887:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  3888:       [InstrStage<1, [SLOT0], 0>,
  3889:        InstrStage<1, [SLOT1], 0>,
  3890:        InstrStage<1, [CVI_LD], 0>,
  3891:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  3892:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3893: 
  3894:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  3895:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3896:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  3897:       [HVX_FWD, HVX_FWD]>,
  3898: 
  3899:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  3900:       [InstrStage<1, [SLOT0], 0>,
  3901:        InstrStage<1, [CVI_ST]>], [1, 2],
  3902:       [Hex_FWD, Hex_FWD]>,
  3903: 
  3904:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  3905:       [InstrStage<1, [SLOT0], 0>,
  3906:        InstrStage<1, [CVI_ST], 0>,
  3907:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  3908:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3909: 
  3910:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  3911:       [InstrStage<1, [SLOT0], 0>,
  3912:        InstrStage<1, [CVI_ST], 0>,
  3913:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  3914:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3915: 
  3916:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  3917:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3918:        InstrStage<1, [CVI_LD], 0>,
  3919:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  3920:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3921: 
  3922:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  3923:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3924:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  3925:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3926: 
  3927:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  3928:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3929:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  3930:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3931: 
  3932:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  3933:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3934:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  3935:       [HVX_FWD, HVX_FWD]>,
  3936: 
  3937:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  3938:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  3939:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  3940:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  3941: 
  3942:     InstrItinData <tc_531b383c, /*SLOT0123*/
  3943:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
  3944:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3945: 
  3946:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  3947:       [InstrStage<1, [SLOT0], 0>,
  3948:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  3949:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  3950: 
  3951:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  3952:       [InstrStage<1, [SLOT0], 0>,
  3953:        InstrStage<1, [CVI_ST], 0>,
  3954:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  3955:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  3956: 
  3957:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  3958:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3959:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  3960:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3961: 
  3962:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  3963:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3964:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  3965:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  3966: 
  3967:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  3968:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3969:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  3970:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3971: 
  3972:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  3973:       [InstrStage<1, [SLOT0], 0>,
  3974:        InstrStage<1, [CVI_ST], 0>,
  3975:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  3976:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  3977: 
  3978:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  3979:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3980:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  3981:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  3982: 
  3983:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  3984:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  3985:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  3986:       [HVX_FWD, Hex_FWD]>,
  3987: 
  3988:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  3989:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3990:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  3991:       [HVX_FWD, HVX_FWD]>,
  3992: 
  3993:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  3994:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3995:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  3996:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  3997: 
  3998:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  3999:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4000:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 4001-4250 / 第 4001-4250 行

```tablegen
  4001:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4002: 
  4003:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  4004:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4005:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  4006:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4007: 
  4008:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  4009:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4010:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  4011:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4012: 
  4013:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  4014:       [InstrStage<1, [SLOT0], 0>,
  4015:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  4016:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4017: 
  4018:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  4019:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4020:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  4021:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4022: 
  4023:     InstrItinData <tc_7095ecba, /*SLOT01,LOAD,VA_DV*/
  4024:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4025:        InstrStage<1, [CVI_LD], 0>,
  4026:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  4027:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4028: 
  4029:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  4030:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4031:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  4032:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4033: 
  4034:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  4035:       [InstrStage<1, [SLOT0], 0>,
  4036:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  4037:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4038: 
  4039:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  4040:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4041:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  4042:       [HVX_FWD]>,
  4043: 
  4044:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  4045:       [InstrStage<1, [SLOT0], 0>,
  4046:        InstrStage<1, [CVI_ST], 0>,
  4047:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  4048:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4049: 
  4050:     InstrItinData <tc_72e2b393, /*SLOT23,VX_DV*/
  4051:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4052:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  4053:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4054: 
  4055:     InstrItinData <tc_73efe966, /*SLOT23,VX_DV*/
  4056:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4057:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  4058:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4059: 
  4060:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  4061:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4062:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  4063:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4064: 
  4065:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  4066:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4067:        InstrStage<1, [CVI_ALL]>], [3, 2],
  4068:       [HVX_FWD, Hex_FWD]>,
  4069: 
  4070:     InstrItinData <tc_7d68d5c2, /*SLOT01,LOAD,VA*/
  4071:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4072:        InstrStage<1, [CVI_LD], 0>,
  4073:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  4074:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4075: 
  4076:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  4077:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4078:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  4079:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4080: 
  4081:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  4082:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4083:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  4084:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4085: 
  4086:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  4087:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4088:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  4089:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4090: 
  4091:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  4092:       [InstrStage<1, [SLOT0], 0>,
  4093:        InstrStage<1, [CVI_ST], 0>,
  4094:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  4095:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4096: 
  4097:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  4098:       [InstrStage<1, [SLOT2], 0>,
  4099:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  4100:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4101: 
  4102:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  4103:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4104:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  4105:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4106: 
  4107:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  4108:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4109:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  4110:       [HVX_FWD, HVX_FWD]>,
  4111: 
  4112:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  4113:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4114:        InstrStage<1, [CVI_LD], 0>,
  4115:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  4116:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  4117:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4118: 
  4119:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  4120:       [InstrStage<1, [SLOT0], 0>,
  4121:        InstrStage<1, [CVI_ST], 0>,
  4122:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  4123:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  4124:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4125: 
  4126:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  4127:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4128:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  4129:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4130: 
  4131:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  4132:       [InstrStage<1, [SLOT0], 0>,
  4133:        InstrStage<1, [CVI_ST], 0>,
  4134:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  4135:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4136: 
  4137:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  4138:       [InstrStage<1, [SLOT0], 0>,
  4139:        InstrStage<1, [CVI_ST], 0>,
  4140:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  4141:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4142: 
  4143:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  4144:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4145:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  4146:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4147: 
  4148:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  4149:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4150:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  4151:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4152: 
  4153:     InstrItinData <tc_a28f32b5, /*SLOT01,LOAD,VA*/
  4154:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4155:        InstrStage<1, [CVI_LD], 0>,
  4156:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  4157:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4158: 
  4159:     InstrItinData <tc_a69eeee1, /*SLOT01,LOAD,VA_DV*/
  4160:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4161:        InstrStage<1, [CVI_LD], 0>,
  4162:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  4163:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4164: 
  4165:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  4166:       [InstrStage<1, [SLOT0], 0>,
  4167:        InstrStage<1, [SLOT1], 0>,
  4168:        InstrStage<1, [CVI_LD], 0>,
  4169:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  4170:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  4171: 
  4172:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  4173:       [InstrStage<1, [SLOT0], 0>,
  4174:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  4175:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4176: 
  4177:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  4178:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4179:        InstrStage<1, [CVI_LD], 0>,
  4180:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  4181:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4182: 
  4183:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  4184:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4185:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  4186:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4187: 
  4188:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  4189:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4190:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  4191:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4192: 
  4193:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  4194:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4195:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  4196:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4197: 
  4198:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  4199:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4200:        InstrStage<1, [CVI_ALL]>], [2],
  4201:       [Hex_FWD]>,
  4202: 
  4203:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  4204:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4205:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  4206:       [HVX_FWD, HVX_FWD]>,
  4207: 
  4208:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  4209:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4210:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  4211:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4212: 
  4213:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  4214:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4215:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  4216:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4217: 
  4218:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  4219:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4220:        InstrStage<1, [CVI_LD], 0>,
  4221:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  4222:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  4223: 
  4224:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  4225:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4226:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  4227:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4228: 
  4229:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  4230:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4231:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  4232:       [HVX_FWD, Hex_FWD]>,
  4233: 
  4234:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  4235:       [InstrStage<1, [SLOT0], 0>,
  4236:        InstrStage<1, [CVI_ST], 0>,
  4237:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  4238:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4239: 
  4240:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  4241:       [InstrStage<1, [SLOT0], 0>,
  4242:        InstrStage<1, [SLOT1], 0>,
  4243:        InstrStage<1, [CVI_ST], 0>,
  4244:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  4245:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4246: 
  4247:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  4248:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4249:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  4250:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 4251-4500 / 第 4251-4500 行

```tablegen
  4251:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4252: 
  4253:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  4254:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4255:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  4256:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4257: 
  4258:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  4259:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4260:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  4261:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4262: 
  4263:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  4264:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4265:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  4266:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4267: 
  4268:     InstrItinData <tc_dcca380f, /*SLOT23,VX_DV*/
  4269:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4270:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  4271:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4272: 
  4273:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  4274:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4275:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  4276:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4277: 
  4278:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  4279:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4280:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  4281:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4282: 
  4283:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  4284:       [InstrStage<1, [SLOT0], 0>,
  4285:        InstrStage<1, [SLOT1], 0>,
  4286:        InstrStage<1, [CVI_ST], 0>,
  4287:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  4288:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4289: 
  4290:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  4291:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
  4292:       [HVX_FWD, HVX_FWD]>,
  4293: 
  4294:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  4295:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4296:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  4297:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4298: 
  4299:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  4300:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4301:        InstrStage<1, [CVI_ALL]>], [3],
  4302:       [HVX_FWD]>,
  4303: 
  4304:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  4305:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4306:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  4307:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  4308: 
  4309:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  4310:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4311:        InstrStage<1, [CVI_ZW]>], [1, 2],
  4312:       [Hex_FWD, Hex_FWD]>,
  4313: 
  4314:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  4315:       [InstrStage<1, [SLOT0], 0>,
  4316:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  4317:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4318: 
  4319:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  4320:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4321:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  4322:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4323: 
  4324:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  4325:       [InstrStage<1, [SLOT2], 0>,
  4326:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  4327:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4328: 
  4329:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  4330:       [InstrStage<1, [SLOT0], 0>,
  4331:        InstrStage<1, [SLOT1], 0>,
  4332:        InstrStage<1, [CVI_ST], 0>,
  4333:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  4334:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  4335:   ];
  4336: }
  4337: 
  4338: class DepHVXItinV69 {
  4339:   list<InstrItinData> DepHVXItinV69_list = [
  4340:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  4341:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4342:        InstrStage<1, [CVI_LD], 0>,
  4343:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  4344:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  4345:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  4346: 
  4347:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  4348:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4349:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  4350:       [HVX_FWD, HVX_FWD]>,
  4351: 
  4352:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  4353:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4354:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  4355:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4356: 
  4357:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  4358:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4359:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  4360:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4361: 
  4362:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  4363:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4364:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  4365:       [HVX_FWD, HVX_FWD]>,
  4366: 
  4367:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  4368:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4369:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  4370:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4371: 
  4372:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  4373:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4374:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  4375:       [HVX_FWD, HVX_FWD]>,
  4376: 
  4377:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  4378:       [InstrStage<1, [SLOT0], 0>,
  4379:        InstrStage<1, [SLOT1], 0>,
  4380:        InstrStage<1, [CVI_ST], 0>,
  4381:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  4382:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4383: 
  4384:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  4385:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4386:        InstrStage<1, [CVI_ALL]>], [],
  4387:       []>,
  4388: 
  4389:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  4390:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4391:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  4392:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  4393:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4394: 
  4395:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  4396:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4397:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  4398:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4399: 
  4400:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  4401:       [InstrStage<1, [SLOT0], 0>,
  4402:        InstrStage<1, [CVI_ST], 0>,
  4403:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  4404:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4405: 
  4406:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  4407:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4408:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  4409:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  4410: 
  4411:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  4412:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4413:        InstrStage<1, [CVI_LD], 0>,
  4414:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  4415:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4416: 
  4417:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  4418:       [InstrStage<1, [SLOT0], 0>,
  4419:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  4420:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4421: 
  4422:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  4423:       [InstrStage<1, [SLOT0], 0>,
  4424:        InstrStage<1, [CVI_ST], 0>,
  4425:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  4426:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  4427:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4428: 
  4429:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  4430:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4431:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  4432:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4433: 
  4434:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  4435:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4436:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  4437:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  4438: 
  4439:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  4440:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4441:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  4442:       [HVX_FWD, HVX_FWD]>,
  4443: 
  4444:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  4445:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4446:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  4447:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4448: 
  4449:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  4450:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4451:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  4452:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4453: 
  4454:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  4455:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4456:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  4457:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4458: 
  4459:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  4460:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4461:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  4462:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4463: 
  4464:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  4465:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4466:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  4467:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4468: 
  4469:     InstrItinData <tc_37820f4c, /*SLOT23,VX*/
  4470:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4471:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  4472:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4473: 
  4474:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  4475:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4476:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  4477:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4478: 
  4479:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  4480:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4481:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  4482:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  4483: 
  4484:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  4485:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4486:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  4487:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4488: 
  4489:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  4490:       [InstrStage<1, [SLOT0], 0>,
  4491:        InstrStage<1, [SLOT1], 0>,
  4492:        InstrStage<1, [CVI_LD], 0>,
  4493:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  4494:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4495: 
  4496:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  4497:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4498:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  4499:       [HVX_FWD, HVX_FWD]>,
  4500: 
```
- EN: It declares types such as DepHVXItinV69, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV69; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV69 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV69 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 4501-4750 / 第 4501-4750 行

```tablegen
  4501:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  4502:       [InstrStage<1, [SLOT0], 0>,
  4503:        InstrStage<1, [CVI_ST]>], [1, 2],
  4504:       [Hex_FWD, Hex_FWD]>,
  4505: 
  4506:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  4507:       [InstrStage<1, [SLOT0], 0>,
  4508:        InstrStage<1, [CVI_ST], 0>,
  4509:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  4510:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4511: 
  4512:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  4513:       [InstrStage<1, [SLOT0], 0>,
  4514:        InstrStage<1, [CVI_ST], 0>,
  4515:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  4516:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4517: 
  4518:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  4519:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4520:        InstrStage<1, [CVI_LD], 0>,
  4521:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  4522:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4523: 
  4524:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  4525:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4526:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  4527:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4528: 
  4529:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  4530:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4531:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  4532:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4533: 
  4534:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  4535:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4536:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  4537:       [HVX_FWD, HVX_FWD]>,
  4538: 
  4539:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  4540:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4541:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  4542:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  4543: 
  4544:     InstrItinData <tc_531b383c, /*SLOT0123*/
  4545:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7, 7],
  4546:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4547: 
  4548:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  4549:       [InstrStage<1, [SLOT0], 0>,
  4550:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  4551:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  4552: 
  4553:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  4554:       [InstrStage<1, [SLOT0], 0>,
  4555:        InstrStage<1, [CVI_ST], 0>,
  4556:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  4557:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4558: 
  4559:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  4560:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4561:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  4562:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4563: 
  4564:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  4565:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4566:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  4567:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4568: 
  4569:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  4570:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4571:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  4572:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4573: 
  4574:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  4575:       [InstrStage<1, [SLOT0], 0>,
  4576:        InstrStage<1, [CVI_ST], 0>,
  4577:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  4578:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4579: 
  4580:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  4581:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4582:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  4583:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4584: 
  4585:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  4586:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4587:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  4588:       [HVX_FWD, Hex_FWD]>,
  4589: 
  4590:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  4591:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4592:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  4593:       [HVX_FWD, HVX_FWD]>,
  4594: 
  4595:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  4596:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4597:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  4598:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4599: 
  4600:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  4601:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4602:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  4603:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4604: 
  4605:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  4606:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4607:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  4608:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4609: 
  4610:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  4611:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4612:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  4613:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4614: 
  4615:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  4616:       [InstrStage<1, [SLOT0], 0>,
  4617:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  4618:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4619: 
  4620:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  4621:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4622:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  4623:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4624: 
  4625:     InstrItinData <tc_7095ecba, /*SLOT01,LOAD,VA_DV*/
  4626:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4627:        InstrStage<1, [CVI_LD], 0>,
  4628:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  4629:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4630: 
  4631:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  4632:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4633:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  4634:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4635: 
  4636:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  4637:       [InstrStage<1, [SLOT0], 0>,
  4638:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  4639:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4640: 
  4641:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  4642:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4643:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  4644:       [HVX_FWD]>,
  4645: 
  4646:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  4647:       [InstrStage<1, [SLOT0], 0>,
  4648:        InstrStage<1, [CVI_ST], 0>,
  4649:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  4650:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4651: 
  4652:     InstrItinData <tc_72e2b393, /*SLOT23,VX*/
  4653:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4654:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  4655:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4656: 
  4657:     InstrItinData <tc_73efe966, /*SLOT23,VX*/
  4658:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4659:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  4660:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4661: 
  4662:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  4663:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4664:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  4665:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4666: 
  4667:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  4668:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4669:        InstrStage<1, [CVI_ALL]>], [3, 2],
  4670:       [HVX_FWD, Hex_FWD]>,
  4671: 
  4672:     InstrItinData <tc_7d68d5c2, /*SLOT01,LOAD,VA*/
  4673:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4674:        InstrStage<1, [CVI_LD], 0>,
  4675:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  4676:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4677: 
  4678:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  4679:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4680:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  4681:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4682: 
  4683:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  4684:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4685:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  4686:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4687: 
  4688:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  4689:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4690:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  4691:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4692: 
  4693:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  4694:       [InstrStage<1, [SLOT0], 0>,
  4695:        InstrStage<1, [CVI_ST], 0>,
  4696:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  4697:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4698: 
  4699:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  4700:       [InstrStage<1, [SLOT2], 0>,
  4701:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  4702:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4703: 
  4704:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  4705:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4706:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  4707:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4708: 
  4709:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  4710:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4711:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  4712:       [HVX_FWD, HVX_FWD]>,
  4713: 
  4714:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  4715:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4716:        InstrStage<1, [CVI_LD], 0>,
  4717:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  4718:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  4719:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4720: 
  4721:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  4722:       [InstrStage<1, [SLOT0], 0>,
  4723:        InstrStage<1, [CVI_ST], 0>,
  4724:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  4725:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  4726:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4727: 
  4728:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  4729:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4730:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  4731:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4732: 
  4733:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  4734:       [InstrStage<1, [SLOT0], 0>,
  4735:        InstrStage<1, [CVI_ST], 0>,
  4736:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  4737:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4738: 
  4739:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  4740:       [InstrStage<1, [SLOT0], 0>,
  4741:        InstrStage<1, [CVI_ST], 0>,
  4742:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  4743:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4744: 
  4745:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  4746:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4747:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  4748:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4749: 
  4750:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 4751-5000 / 第 4751-5000 行

```tablegen
  4751:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4752:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  4753:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4754: 
  4755:     InstrItinData <tc_a28f32b5, /*SLOT01,LOAD,VA*/
  4756:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4757:        InstrStage<1, [CVI_LD], 0>,
  4758:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  4759:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4760: 
  4761:     InstrItinData <tc_a69eeee1, /*SLOT01,LOAD,VA_DV*/
  4762:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4763:        InstrStage<1, [CVI_LD], 0>,
  4764:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  4765:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4766: 
  4767:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  4768:       [InstrStage<1, [SLOT0], 0>,
  4769:        InstrStage<1, [SLOT1], 0>,
  4770:        InstrStage<1, [CVI_LD], 0>,
  4771:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  4772:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  4773: 
  4774:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  4775:       [InstrStage<1, [SLOT0], 0>,
  4776:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  4777:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4778: 
  4779:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  4780:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4781:        InstrStage<1, [CVI_LD], 0>,
  4782:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  4783:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4784: 
  4785:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  4786:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4787:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  4788:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4789: 
  4790:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  4791:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4792:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  4793:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  4794: 
  4795:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  4796:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4797:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  4798:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4799: 
  4800:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  4801:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4802:        InstrStage<1, [CVI_ALL]>], [2],
  4803:       [Hex_FWD]>,
  4804: 
  4805:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  4806:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4807:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  4808:       [HVX_FWD, HVX_FWD]>,
  4809: 
  4810:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  4811:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4812:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  4813:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4814: 
  4815:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  4816:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4817:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  4818:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4819: 
  4820:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  4821:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4822:        InstrStage<1, [CVI_LD], 0>,
  4823:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  4824:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  4825: 
  4826:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  4827:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4828:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  4829:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4830: 
  4831:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  4832:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4833:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  4834:       [HVX_FWD, Hex_FWD]>,
  4835: 
  4836:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  4837:       [InstrStage<1, [SLOT0], 0>,
  4838:        InstrStage<1, [CVI_ST], 0>,
  4839:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  4840:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  4841: 
  4842:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  4843:       [InstrStage<1, [SLOT0], 0>,
  4844:        InstrStage<1, [SLOT1], 0>,
  4845:        InstrStage<1, [CVI_ST], 0>,
  4846:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  4847:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4848: 
  4849:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  4850:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4851:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  4852:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  4853:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4854: 
  4855:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  4856:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4857:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  4858:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4859: 
  4860:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  4861:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4862:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  4863:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4864: 
  4865:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  4866:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4867:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  4868:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4869: 
  4870:     InstrItinData <tc_dcca380f, /*SLOT23,VX*/
  4871:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4872:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  4873:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4874: 
  4875:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  4876:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4877:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  4878:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4879: 
  4880:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  4881:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4882:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  4883:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4884: 
  4885:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  4886:       [InstrStage<1, [SLOT0], 0>,
  4887:        InstrStage<1, [SLOT1], 0>,
  4888:        InstrStage<1, [CVI_ST], 0>,
  4889:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  4890:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4891: 
  4892:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  4893:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 7],
  4894:       [HVX_FWD, HVX_FWD]>,
  4895: 
  4896:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  4897:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4898:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  4899:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4900: 
  4901:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  4902:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4903:        InstrStage<1, [CVI_ALL]>], [3],
  4904:       [HVX_FWD]>,
  4905: 
  4906:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  4907:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4908:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  4909:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  4910: 
  4911:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  4912:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4913:        InstrStage<1, [CVI_ZW]>], [1, 2],
  4914:       [Hex_FWD, Hex_FWD]>,
  4915: 
  4916:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  4917:       [InstrStage<1, [SLOT0], 0>,
  4918:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  4919:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4920: 
  4921:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  4922:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4923:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  4924:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4925: 
  4926:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  4927:       [InstrStage<1, [SLOT2], 0>,
  4928:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  4929:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4930: 
  4931:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  4932:       [InstrStage<1, [SLOT0], 0>,
  4933:        InstrStage<1, [SLOT1], 0>,
  4934:        InstrStage<1, [CVI_ST], 0>,
  4935:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  4936:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  4937:   ];
  4938: }
  4939: 
  4940: class DepHVXItinV71 {
  4941:   list<InstrItinData> DepHVXItinV71_list = [
  4942:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  4943:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  4944:        InstrStage<1, [CVI_LD], 0>,
  4945:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  4946:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  4947:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  4948: 
  4949:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  4950:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4951:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  4952:       [HVX_FWD, HVX_FWD]>,
  4953: 
  4954:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  4955:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4956:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  4957:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  4958: 
  4959:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  4960:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4961:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  4962:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  4963: 
  4964:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  4965:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4966:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  4967:       [HVX_FWD, HVX_FWD]>,
  4968: 
  4969:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  4970:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4971:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  4972:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  4973: 
  4974:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  4975:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4976:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  4977:       [HVX_FWD, HVX_FWD]>,
  4978: 
  4979:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  4980:       [InstrStage<1, [SLOT0], 0>,
  4981:        InstrStage<1, [SLOT1], 0>,
  4982:        InstrStage<1, [CVI_ST], 0>,
  4983:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  4984:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  4985: 
  4986:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  4987:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4988:        InstrStage<1, [CVI_ALL]>], [],
  4989:       []>,
  4990: 
  4991:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  4992:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  4993:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  4994:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  4995:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  4996: 
  4997:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  4998:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  4999:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  5000:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepHVXItinV71, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV71; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV71 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV71 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 5001-5250 / 第 5001-5250 行

```tablegen
  5001: 
  5002:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  5003:       [InstrStage<1, [SLOT0], 0>,
  5004:        InstrStage<1, [CVI_ST], 0>,
  5005:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  5006:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5007: 
  5008:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  5009:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5010:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  5011:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  5012: 
  5013:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  5014:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5015:        InstrStage<1, [CVI_LD], 0>,
  5016:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  5017:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5018: 
  5019:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  5020:       [InstrStage<1, [SLOT0], 0>,
  5021:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  5022:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5023: 
  5024:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  5025:       [InstrStage<1, [SLOT0], 0>,
  5026:        InstrStage<1, [CVI_ST], 0>,
  5027:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  5028:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  5029:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5030: 
  5031:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  5032:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5033:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  5034:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5035: 
  5036:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  5037:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5038:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  5039:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  5040: 
  5041:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  5042:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5043:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  5044:       [HVX_FWD, HVX_FWD]>,
  5045: 
  5046:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  5047:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5048:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  5049:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5050: 
  5051:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  5052:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5053:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  5054:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5055: 
  5056:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  5057:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5058:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  5059:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5060: 
  5061:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  5062:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5063:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  5064:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5065: 
  5066:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  5067:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5068:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  5069:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5070: 
  5071:     InstrItinData <tc_37820f4c, /*SLOT23,VX*/
  5072:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5073:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  5074:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5075: 
  5076:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  5077:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5078:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  5079:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5080: 
  5081:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  5082:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5083:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  5084:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  5085: 
  5086:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  5087:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5088:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  5089:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5090: 
  5091:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  5092:       [InstrStage<1, [SLOT0], 0>,
  5093:        InstrStage<1, [SLOT1], 0>,
  5094:        InstrStage<1, [CVI_LD], 0>,
  5095:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  5096:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5097: 
  5098:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  5099:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5100:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  5101:       [HVX_FWD, HVX_FWD]>,
  5102: 
  5103:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  5104:       [InstrStage<1, [SLOT0], 0>,
  5105:        InstrStage<1, [CVI_ST]>], [1, 2],
  5106:       [Hex_FWD, Hex_FWD]>,
  5107: 
  5108:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  5109:       [InstrStage<1, [SLOT0], 0>,
  5110:        InstrStage<1, [CVI_ST], 0>,
  5111:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  5112:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5113: 
  5114:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  5115:       [InstrStage<1, [SLOT0], 0>,
  5116:        InstrStage<1, [CVI_ST], 0>,
  5117:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  5118:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5119: 
  5120:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  5121:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5122:        InstrStage<1, [CVI_LD], 0>,
  5123:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  5124:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5125: 
  5126:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  5127:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5128:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  5129:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5130: 
  5131:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  5132:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5133:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  5134:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5135: 
  5136:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  5137:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5138:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  5139:       [HVX_FWD, HVX_FWD]>,
  5140: 
  5141:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  5142:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5143:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  5144:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  5145: 
  5146:     InstrItinData <tc_531b383c, /*SLOT0123*/
  5147:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5, 5],
  5148:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5149: 
  5150:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  5151:       [InstrStage<1, [SLOT0], 0>,
  5152:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  5153:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  5154: 
  5155:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  5156:       [InstrStage<1, [SLOT0], 0>,
  5157:        InstrStage<1, [CVI_ST], 0>,
  5158:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  5159:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5160: 
  5161:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  5162:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5163:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  5164:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5165: 
  5166:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  5167:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5168:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  5169:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5170: 
  5171:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  5172:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5173:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  5174:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5175: 
  5176:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  5177:       [InstrStage<1, [SLOT0], 0>,
  5178:        InstrStage<1, [CVI_ST], 0>,
  5179:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  5180:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5181: 
  5182:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  5183:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5184:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  5185:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5186: 
  5187:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  5188:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5189:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  5190:       [HVX_FWD, Hex_FWD]>,
  5191: 
  5192:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  5193:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5194:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  5195:       [HVX_FWD, HVX_FWD]>,
  5196: 
  5197:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  5198:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5199:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  5200:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5201: 
  5202:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  5203:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5204:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  5205:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5206: 
  5207:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  5208:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5209:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  5210:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5211: 
  5212:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  5213:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5214:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  5215:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5216: 
  5217:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  5218:       [InstrStage<1, [SLOT0], 0>,
  5219:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  5220:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5221: 
  5222:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  5223:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5224:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  5225:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5226: 
  5227:     InstrItinData <tc_7095ecba, /*SLOT01,LOAD,VA_DV*/
  5228:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5229:        InstrStage<1, [CVI_LD], 0>,
  5230:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  5231:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5232: 
  5233:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  5234:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5235:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  5236:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5237: 
  5238:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  5239:       [InstrStage<1, [SLOT0], 0>,
  5240:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  5241:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5242: 
  5243:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  5244:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5245:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  5246:       [HVX_FWD]>,
  5247: 
  5248:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  5249:       [InstrStage<1, [SLOT0], 0>,
  5250:        InstrStage<1, [CVI_ST], 0>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 5251-5500 / 第 5251-5500 行

```tablegen
  5251:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  5252:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5253: 
  5254:     InstrItinData <tc_72e2b393, /*SLOT23,VX*/
  5255:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5256:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  5257:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5258: 
  5259:     InstrItinData <tc_73efe966, /*SLOT23,VX*/
  5260:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5261:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  5262:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5263: 
  5264:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  5265:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5266:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  5267:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5268: 
  5269:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  5270:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5271:        InstrStage<1, [CVI_ALL]>], [3, 2],
  5272:       [HVX_FWD, Hex_FWD]>,
  5273: 
  5274:     InstrItinData <tc_7d68d5c2, /*SLOT01,LOAD,VA*/
  5275:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5276:        InstrStage<1, [CVI_LD], 0>,
  5277:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  5278:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5279: 
  5280:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  5281:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5282:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  5283:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5284: 
  5285:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  5286:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5287:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  5288:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5289: 
  5290:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  5291:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5292:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  5293:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5294: 
  5295:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  5296:       [InstrStage<1, [SLOT0], 0>,
  5297:        InstrStage<1, [CVI_ST], 0>,
  5298:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  5299:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5300: 
  5301:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  5302:       [InstrStage<1, [SLOT2], 0>,
  5303:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  5304:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5305: 
  5306:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  5307:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5308:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  5309:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5310: 
  5311:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  5312:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5313:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  5314:       [HVX_FWD, HVX_FWD]>,
  5315: 
  5316:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  5317:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5318:        InstrStage<1, [CVI_LD], 0>,
  5319:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  5320:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  5321:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5322: 
  5323:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  5324:       [InstrStage<1, [SLOT0], 0>,
  5325:        InstrStage<1, [CVI_ST], 0>,
  5326:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  5327:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  5328:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5329: 
  5330:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  5331:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5332:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  5333:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5334: 
  5335:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  5336:       [InstrStage<1, [SLOT0], 0>,
  5337:        InstrStage<1, [CVI_ST], 0>,
  5338:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  5339:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5340: 
  5341:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  5342:       [InstrStage<1, [SLOT0], 0>,
  5343:        InstrStage<1, [CVI_ST], 0>,
  5344:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  5345:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5346: 
  5347:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  5348:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5349:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  5350:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5351: 
  5352:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  5353:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5354:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  5355:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5356: 
  5357:     InstrItinData <tc_a28f32b5, /*SLOT01,LOAD,VA*/
  5358:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5359:        InstrStage<1, [CVI_LD], 0>,
  5360:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  5361:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5362: 
  5363:     InstrItinData <tc_a69eeee1, /*SLOT01,LOAD,VA_DV*/
  5364:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5365:        InstrStage<1, [CVI_LD], 0>,
  5366:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  5367:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5368: 
  5369:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  5370:       [InstrStage<1, [SLOT0], 0>,
  5371:        InstrStage<1, [SLOT1], 0>,
  5372:        InstrStage<1, [CVI_LD], 0>,
  5373:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  5374:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  5375: 
  5376:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  5377:       [InstrStage<1, [SLOT0], 0>,
  5378:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  5379:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5380: 
  5381:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  5382:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5383:        InstrStage<1, [CVI_LD], 0>,
  5384:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  5385:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5386: 
  5387:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  5388:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5389:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  5390:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5391: 
  5392:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  5393:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5394:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  5395:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5396: 
  5397:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  5398:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5399:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  5400:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5401: 
  5402:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  5403:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5404:        InstrStage<1, [CVI_ALL]>], [2],
  5405:       [Hex_FWD]>,
  5406: 
  5407:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  5408:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5409:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  5410:       [HVX_FWD, HVX_FWD]>,
  5411: 
  5412:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  5413:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5414:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  5415:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5416: 
  5417:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  5418:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5419:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  5420:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5421: 
  5422:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  5423:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5424:        InstrStage<1, [CVI_LD], 0>,
  5425:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  5426:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  5427: 
  5428:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  5429:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5430:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  5431:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5432: 
  5433:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  5434:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5435:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  5436:       [HVX_FWD, Hex_FWD]>,
  5437: 
  5438:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  5439:       [InstrStage<1, [SLOT0], 0>,
  5440:        InstrStage<1, [CVI_ST], 0>,
  5441:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  5442:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5443: 
  5444:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  5445:       [InstrStage<1, [SLOT0], 0>,
  5446:        InstrStage<1, [SLOT1], 0>,
  5447:        InstrStage<1, [CVI_ST], 0>,
  5448:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  5449:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5450: 
  5451:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  5452:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5453:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  5454:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  5455:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5456: 
  5457:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  5458:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5459:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  5460:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5461: 
  5462:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  5463:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5464:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  5465:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5466: 
  5467:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  5468:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5469:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  5470:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5471: 
  5472:     InstrItinData <tc_dcca380f, /*SLOT23,VX*/
  5473:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5474:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  5475:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5476: 
  5477:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  5478:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5479:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  5480:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5481: 
  5482:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  5483:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5484:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  5485:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5486: 
  5487:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  5488:       [InstrStage<1, [SLOT0], 0>,
  5489:        InstrStage<1, [SLOT1], 0>,
  5490:        InstrStage<1, [CVI_ST], 0>,
  5491:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  5492:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5493: 
  5494:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  5495:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5],
  5496:       [HVX_FWD, HVX_FWD]>,
  5497: 
  5498:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  5499:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5500:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 5501-5750 / 第 5501-5750 行

```tablegen
  5501:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5502: 
  5503:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  5504:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5505:        InstrStage<1, [CVI_ALL]>], [3],
  5506:       [HVX_FWD]>,
  5507: 
  5508:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  5509:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5510:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  5511:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  5512: 
  5513:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  5514:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5515:        InstrStage<1, [CVI_ZW]>], [1, 2],
  5516:       [Hex_FWD, Hex_FWD]>,
  5517: 
  5518:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  5519:       [InstrStage<1, [SLOT0], 0>,
  5520:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  5521:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5522: 
  5523:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  5524:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5525:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  5526:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5527: 
  5528:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  5529:       [InstrStage<1, [SLOT2], 0>,
  5530:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  5531:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5532: 
  5533:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  5534:       [InstrStage<1, [SLOT0], 0>,
  5535:        InstrStage<1, [SLOT1], 0>,
  5536:        InstrStage<1, [CVI_ST], 0>,
  5537:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  5538:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  5539:   ];
  5540: }
  5541: 
  5542: class DepHVXItinV73 {
  5543:   list<InstrItinData> DepHVXItinV73_list = [
  5544:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  5545:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5546:        InstrStage<1, [CVI_LD], 0>,
  5547:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  5548:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  5549:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  5550: 
  5551:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  5552:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5553:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  5554:       [HVX_FWD, HVX_FWD]>,
  5555: 
  5556:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  5557:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5558:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  5559:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5560: 
  5561:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  5562:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5563:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  5564:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5565: 
  5566:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  5567:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5568:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  5569:       [HVX_FWD, HVX_FWD]>,
  5570: 
  5571:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  5572:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5573:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  5574:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5575: 
  5576:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  5577:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5578:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  5579:       [HVX_FWD, HVX_FWD]>,
  5580: 
  5581:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  5582:       [InstrStage<1, [SLOT0], 0>,
  5583:        InstrStage<1, [SLOT1], 0>,
  5584:        InstrStage<1, [CVI_ST], 0>,
  5585:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  5586:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5587: 
  5588:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  5589:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5590:        InstrStage<1, [CVI_ALL]>], [],
  5591:       []>,
  5592: 
  5593:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  5594:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5595:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  5596:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  5597:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5598: 
  5599:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  5600:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5601:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  5602:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5603: 
  5604:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  5605:       [InstrStage<1, [SLOT0], 0>,
  5606:        InstrStage<1, [CVI_ST], 0>,
  5607:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  5608:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5609: 
  5610:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  5611:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5612:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  5613:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  5614: 
  5615:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  5616:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5617:        InstrStage<1, [CVI_LD], 0>,
  5618:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  5619:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5620: 
  5621:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  5622:       [InstrStage<1, [SLOT0], 0>,
  5623:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  5624:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5625: 
  5626:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  5627:       [InstrStage<1, [SLOT0], 0>,
  5628:        InstrStage<1, [CVI_ST], 0>,
  5629:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  5630:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  5631:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5632: 
  5633:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  5634:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5635:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  5636:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5637: 
  5638:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  5639:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5640:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  5641:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  5642: 
  5643:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  5644:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5645:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  5646:       [HVX_FWD, HVX_FWD]>,
  5647: 
  5648:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  5649:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5650:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  5651:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5652: 
  5653:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  5654:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5655:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  5656:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5657: 
  5658:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  5659:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5660:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  5661:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5662: 
  5663:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  5664:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5665:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  5666:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5667: 
  5668:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  5669:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5670:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  5671:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5672: 
  5673:     InstrItinData <tc_37820f4c, /*SLOT23,VX*/
  5674:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5675:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  5676:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5677: 
  5678:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  5679:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5680:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  5681:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5682: 
  5683:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  5684:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5685:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  5686:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  5687: 
  5688:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  5689:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5690:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  5691:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5692: 
  5693:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  5694:       [InstrStage<1, [SLOT0], 0>,
  5695:        InstrStage<1, [SLOT1], 0>,
  5696:        InstrStage<1, [CVI_LD], 0>,
  5697:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  5698:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5699: 
  5700:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  5701:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5702:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  5703:       [HVX_FWD, HVX_FWD]>,
  5704: 
  5705:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  5706:       [InstrStage<1, [SLOT0], 0>,
  5707:        InstrStage<1, [CVI_ST]>], [1, 2],
  5708:       [Hex_FWD, Hex_FWD]>,
  5709: 
  5710:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  5711:       [InstrStage<1, [SLOT0], 0>,
  5712:        InstrStage<1, [CVI_ST], 0>,
  5713:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  5714:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5715: 
  5716:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  5717:       [InstrStage<1, [SLOT0], 0>,
  5718:        InstrStage<1, [CVI_ST], 0>,
  5719:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  5720:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5721: 
  5722:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  5723:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5724:        InstrStage<1, [CVI_LD], 0>,
  5725:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  5726:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5727: 
  5728:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  5729:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5730:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  5731:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5732: 
  5733:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  5734:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5735:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  5736:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5737: 
  5738:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  5739:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5740:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  5741:       [HVX_FWD, HVX_FWD]>,
  5742: 
  5743:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  5744:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5745:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  5746:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  5747: 
  5748:     InstrItinData <tc_531b383c, /*SLOT0123*/
  5749:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5, 5],
  5750:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
```
- EN: It declares types such as DepHVXItinV73, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV73; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV73 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV73 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 5751-6000 / 第 5751-6000 行

```tablegen
  5751: 
  5752:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  5753:       [InstrStage<1, [SLOT0], 0>,
  5754:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  5755:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  5756: 
  5757:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  5758:       [InstrStage<1, [SLOT0], 0>,
  5759:        InstrStage<1, [CVI_ST], 0>,
  5760:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  5761:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5762: 
  5763:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  5764:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5765:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  5766:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5767: 
  5768:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  5769:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5770:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  5771:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5772: 
  5773:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  5774:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5775:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  5776:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5777: 
  5778:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  5779:       [InstrStage<1, [SLOT0], 0>,
  5780:        InstrStage<1, [CVI_ST], 0>,
  5781:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  5782:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5783: 
  5784:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  5785:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5786:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  5787:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5788: 
  5789:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  5790:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5791:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  5792:       [HVX_FWD, Hex_FWD]>,
  5793: 
  5794:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  5795:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5796:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  5797:       [HVX_FWD, HVX_FWD]>,
  5798: 
  5799:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  5800:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5801:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  5802:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5803: 
  5804:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  5805:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5806:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  5807:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5808: 
  5809:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  5810:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5811:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  5812:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5813: 
  5814:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  5815:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5816:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  5817:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5818: 
  5819:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  5820:       [InstrStage<1, [SLOT0], 0>,
  5821:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  5822:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5823: 
  5824:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  5825:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5826:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  5827:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5828: 
  5829:     InstrItinData <tc_7095ecba, /*SLOT01,LOAD,VA_DV*/
  5830:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5831:        InstrStage<1, [CVI_LD], 0>,
  5832:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  5833:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5834: 
  5835:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  5836:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5837:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  5838:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5839: 
  5840:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  5841:       [InstrStage<1, [SLOT0], 0>,
  5842:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  5843:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5844: 
  5845:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  5846:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5847:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  5848:       [HVX_FWD]>,
  5849: 
  5850:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  5851:       [InstrStage<1, [SLOT0], 0>,
  5852:        InstrStage<1, [CVI_ST], 0>,
  5853:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  5854:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5855: 
  5856:     InstrItinData <tc_72e2b393, /*SLOT23,VX*/
  5857:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5858:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  5859:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5860: 
  5861:     InstrItinData <tc_73efe966, /*SLOT23,VX*/
  5862:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5863:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  5864:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5865: 
  5866:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  5867:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5868:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  5869:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5870: 
  5871:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  5872:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5873:        InstrStage<1, [CVI_ALL]>], [3, 2],
  5874:       [HVX_FWD, Hex_FWD]>,
  5875: 
  5876:     InstrItinData <tc_7d68d5c2, /*SLOT01,LOAD,VA*/
  5877:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5878:        InstrStage<1, [CVI_LD], 0>,
  5879:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  5880:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5881: 
  5882:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  5883:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5884:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  5885:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5886: 
  5887:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  5888:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5889:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  5890:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  5891: 
  5892:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  5893:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5894:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  5895:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5896: 
  5897:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  5898:       [InstrStage<1, [SLOT0], 0>,
  5899:        InstrStage<1, [CVI_ST], 0>,
  5900:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  5901:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5902: 
  5903:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  5904:       [InstrStage<1, [SLOT2], 0>,
  5905:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  5906:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5907: 
  5908:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  5909:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5910:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  5911:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5912: 
  5913:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  5914:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5915:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  5916:       [HVX_FWD, HVX_FWD]>,
  5917: 
  5918:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  5919:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5920:        InstrStage<1, [CVI_LD], 0>,
  5921:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  5922:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  5923:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5924: 
  5925:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  5926:       [InstrStage<1, [SLOT0], 0>,
  5927:        InstrStage<1, [CVI_ST], 0>,
  5928:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  5929:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  5930:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5931: 
  5932:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  5933:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5934:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  5935:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  5936: 
  5937:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  5938:       [InstrStage<1, [SLOT0], 0>,
  5939:        InstrStage<1, [CVI_ST], 0>,
  5940:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  5941:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5942: 
  5943:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  5944:       [InstrStage<1, [SLOT0], 0>,
  5945:        InstrStage<1, [CVI_ST], 0>,
  5946:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  5947:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5948: 
  5949:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  5950:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5951:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  5952:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5953: 
  5954:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  5955:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5956:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  5957:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  5958: 
  5959:     InstrItinData <tc_a28f32b5, /*SLOT01,LOAD,VA*/
  5960:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5961:        InstrStage<1, [CVI_LD], 0>,
  5962:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  5963:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5964: 
  5965:     InstrItinData <tc_a69eeee1, /*SLOT01,LOAD,VA_DV*/
  5966:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5967:        InstrStage<1, [CVI_LD], 0>,
  5968:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  5969:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  5970: 
  5971:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  5972:       [InstrStage<1, [SLOT0], 0>,
  5973:        InstrStage<1, [SLOT1], 0>,
  5974:        InstrStage<1, [CVI_LD], 0>,
  5975:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  5976:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  5977: 
  5978:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  5979:       [InstrStage<1, [SLOT0], 0>,
  5980:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  5981:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  5982: 
  5983:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  5984:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  5985:        InstrStage<1, [CVI_LD], 0>,
  5986:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  5987:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5988: 
  5989:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  5990:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  5991:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  5992:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  5993: 
  5994:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  5995:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  5996:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  5997:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  5998: 
  5999:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  6000:       [InstrStage<1, [SLOT2, SLOT3], 0>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 6001-6250 / 第 6001-6250 行

```tablegen
  6001:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  6002:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6003: 
  6004:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  6005:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6006:        InstrStage<1, [CVI_ALL]>], [2],
  6007:       [Hex_FWD]>,
  6008: 
  6009:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  6010:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6011:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  6012:       [HVX_FWD, HVX_FWD]>,
  6013: 
  6014:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  6015:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6016:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  6017:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6018: 
  6019:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  6020:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6021:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  6022:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6023: 
  6024:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  6025:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6026:        InstrStage<1, [CVI_LD], 0>,
  6027:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  6028:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  6029: 
  6030:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  6031:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6032:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  6033:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6034: 
  6035:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  6036:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6037:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  6038:       [HVX_FWD, Hex_FWD]>,
  6039: 
  6040:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  6041:       [InstrStage<1, [SLOT0], 0>,
  6042:        InstrStage<1, [CVI_ST], 0>,
  6043:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  6044:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  6045: 
  6046:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  6047:       [InstrStage<1, [SLOT0], 0>,
  6048:        InstrStage<1, [SLOT1], 0>,
  6049:        InstrStage<1, [CVI_ST], 0>,
  6050:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  6051:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6052: 
  6053:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  6054:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6055:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  6056:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  6057:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6058: 
  6059:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  6060:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6061:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  6062:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6063: 
  6064:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  6065:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6066:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  6067:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6068: 
  6069:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  6070:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6071:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  6072:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6073: 
  6074:     InstrItinData <tc_dcca380f, /*SLOT23,VX*/
  6075:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6076:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  6077:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6078: 
  6079:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  6080:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6081:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  6082:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6083: 
  6084:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  6085:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6086:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  6087:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6088: 
  6089:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  6090:       [InstrStage<1, [SLOT0], 0>,
  6091:        InstrStage<1, [SLOT1], 0>,
  6092:        InstrStage<1, [CVI_ST], 0>,
  6093:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  6094:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6095: 
  6096:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  6097:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5],
  6098:       [HVX_FWD, HVX_FWD]>,
  6099: 
  6100:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  6101:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6102:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  6103:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6104: 
  6105:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  6106:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6107:        InstrStage<1, [CVI_ALL]>], [3],
  6108:       [HVX_FWD]>,
  6109: 
  6110:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  6111:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6112:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  6113:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  6114: 
  6115:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  6116:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6117:        InstrStage<1, [CVI_ZW]>], [1, 2],
  6118:       [Hex_FWD, Hex_FWD]>,
  6119: 
  6120:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  6121:       [InstrStage<1, [SLOT0], 0>,
  6122:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  6123:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6124: 
  6125:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  6126:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6127:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  6128:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6129: 
  6130:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  6131:       [InstrStage<1, [SLOT2], 0>,
  6132:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  6133:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6134: 
  6135:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  6136:       [InstrStage<1, [SLOT0], 0>,
  6137:        InstrStage<1, [SLOT1], 0>,
  6138:        InstrStage<1, [CVI_ST], 0>,
  6139:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  6140:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  6141:   ];
  6142: }
  6143: 
  6144: class DepHVXItinV75 {
  6145:   list<InstrItinData> DepHVXItinV75_list = [
  6146:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  6147:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6148:        InstrStage<1, [CVI_LD], 0>,
  6149:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  6150:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  6151:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  6152: 
  6153:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  6154:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6155:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  6156:       [HVX_FWD, HVX_FWD]>,
  6157: 
  6158:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  6159:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6160:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  6161:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6162: 
  6163:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  6164:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6165:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  6166:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6167: 
  6168:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  6169:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6170:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  6171:       [HVX_FWD, HVX_FWD]>,
  6172: 
  6173:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  6174:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6175:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  6176:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6177: 
  6178:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  6179:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6180:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  6181:       [HVX_FWD, HVX_FWD]>,
  6182: 
  6183:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  6184:       [InstrStage<1, [SLOT0], 0>,
  6185:        InstrStage<1, [SLOT1], 0>,
  6186:        InstrStage<1, [CVI_ST], 0>,
  6187:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  6188:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6189: 
  6190:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  6191:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6192:        InstrStage<1, [CVI_ALL]>], [],
  6193:       []>,
  6194: 
  6195:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  6196:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6197:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  6198:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  6199:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6200: 
  6201:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  6202:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6203:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  6204:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6205: 
  6206:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  6207:       [InstrStage<1, [SLOT0], 0>,
  6208:        InstrStage<1, [CVI_ST], 0>,
  6209:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  6210:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6211: 
  6212:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  6213:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6214:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  6215:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  6216: 
  6217:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  6218:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6219:        InstrStage<1, [CVI_LD], 0>,
  6220:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  6221:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6222: 
  6223:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  6224:       [InstrStage<1, [SLOT0], 0>,
  6225:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  6226:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6227: 
  6228:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  6229:       [InstrStage<1, [SLOT0], 0>,
  6230:        InstrStage<1, [CVI_ST], 0>,
  6231:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  6232:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  6233:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6234: 
  6235:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  6236:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6237:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  6238:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6239: 
  6240:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  6241:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6242:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  6243:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  6244: 
  6245:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  6246:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6247:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  6248:       [HVX_FWD, HVX_FWD]>,
  6249: 
  6250:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
```
- EN: It declares types such as DepHVXItinV75, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV75; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV75 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV75 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 6251-6500 / 第 6251-6500 行

```tablegen
  6251:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6252:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  6253:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6254: 
  6255:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  6256:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6257:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  6258:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6259: 
  6260:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  6261:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6262:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  6263:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6264: 
  6265:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  6266:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6267:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  6268:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6269: 
  6270:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  6271:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6272:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  6273:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6274: 
  6275:     InstrItinData <tc_37820f4c, /*SLOT23,VX*/
  6276:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6277:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  6278:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6279: 
  6280:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  6281:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6282:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  6283:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6284: 
  6285:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  6286:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6287:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  6288:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  6289: 
  6290:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  6291:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6292:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  6293:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6294: 
  6295:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  6296:       [InstrStage<1, [SLOT0], 0>,
  6297:        InstrStage<1, [SLOT1], 0>,
  6298:        InstrStage<1, [CVI_LD], 0>,
  6299:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  6300:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6301: 
  6302:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  6303:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6304:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  6305:       [HVX_FWD, HVX_FWD]>,
  6306: 
  6307:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  6308:       [InstrStage<1, [SLOT0], 0>,
  6309:        InstrStage<1, [CVI_ST]>], [1, 2],
  6310:       [Hex_FWD, Hex_FWD]>,
  6311: 
  6312:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  6313:       [InstrStage<1, [SLOT0], 0>,
  6314:        InstrStage<1, [CVI_ST], 0>,
  6315:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  6316:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6317: 
  6318:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  6319:       [InstrStage<1, [SLOT0], 0>,
  6320:        InstrStage<1, [CVI_ST], 0>,
  6321:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  6322:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6323: 
  6324:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  6325:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6326:        InstrStage<1, [CVI_LD], 0>,
  6327:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  6328:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6329: 
  6330:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  6331:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6332:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  6333:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6334: 
  6335:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  6336:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6337:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  6338:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6339: 
  6340:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  6341:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6342:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  6343:       [HVX_FWD, HVX_FWD]>,
  6344: 
  6345:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  6346:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6347:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  6348:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  6349: 
  6350:     InstrItinData <tc_531b383c, /*SLOT0123*/
  6351:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5, 5],
  6352:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6353: 
  6354:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  6355:       [InstrStage<1, [SLOT0], 0>,
  6356:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  6357:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  6358: 
  6359:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  6360:       [InstrStage<1, [SLOT0], 0>,
  6361:        InstrStage<1, [CVI_ST], 0>,
  6362:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  6363:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6364: 
  6365:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  6366:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6367:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  6368:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6369: 
  6370:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  6371:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6372:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  6373:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6374: 
  6375:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  6376:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6377:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  6378:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6379: 
  6380:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  6381:       [InstrStage<1, [SLOT0], 0>,
  6382:        InstrStage<1, [CVI_ST], 0>,
  6383:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  6384:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  6385: 
  6386:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  6387:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6388:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  6389:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6390: 
  6391:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  6392:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6393:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  6394:       [HVX_FWD, Hex_FWD]>,
  6395: 
  6396:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  6397:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6398:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  6399:       [HVX_FWD, HVX_FWD]>,
  6400: 
  6401:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  6402:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6403:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  6404:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6405: 
  6406:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  6407:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6408:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  6409:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6410: 
  6411:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  6412:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6413:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  6414:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6415: 
  6416:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  6417:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6418:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  6419:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6420: 
  6421:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  6422:       [InstrStage<1, [SLOT0], 0>,
  6423:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  6424:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6425: 
  6426:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  6427:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6428:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  6429:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6430: 
  6431:     InstrItinData <tc_7095ecba, /*SLOT01,LOAD,VA_DV*/
  6432:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6433:        InstrStage<1, [CVI_LD], 0>,
  6434:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  6435:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  6436: 
  6437:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  6438:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6439:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  6440:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6441: 
  6442:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  6443:       [InstrStage<1, [SLOT0], 0>,
  6444:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  6445:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6446: 
  6447:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  6448:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6449:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  6450:       [HVX_FWD]>,
  6451: 
  6452:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  6453:       [InstrStage<1, [SLOT0], 0>,
  6454:        InstrStage<1, [CVI_ST], 0>,
  6455:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  6456:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  6457: 
  6458:     InstrItinData <tc_72e2b393, /*SLOT23,VX*/
  6459:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6460:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  6461:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6462: 
  6463:     InstrItinData <tc_73efe966, /*SLOT23,VX*/
  6464:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6465:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  6466:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6467: 
  6468:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  6469:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6470:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  6471:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6472: 
  6473:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  6474:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6475:        InstrStage<1, [CVI_ALL]>], [3, 2],
  6476:       [HVX_FWD, Hex_FWD]>,
  6477: 
  6478:     InstrItinData <tc_7d68d5c2, /*SLOT01,LOAD,VA*/
  6479:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6480:        InstrStage<1, [CVI_LD], 0>,
  6481:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  6482:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6483: 
  6484:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  6485:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6486:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  6487:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6488: 
  6489:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  6490:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6491:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  6492:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6493: 
  6494:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  6495:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6496:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  6497:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6498: 
  6499:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  6500:       [InstrStage<1, [SLOT0], 0>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 6501-6750 / 第 6501-6750 行

```tablegen
  6501:        InstrStage<1, [CVI_ST], 0>,
  6502:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  6503:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  6504: 
  6505:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  6506:       [InstrStage<1, [SLOT2], 0>,
  6507:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  6508:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6509: 
  6510:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  6511:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6512:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  6513:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6514: 
  6515:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  6516:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6517:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  6518:       [HVX_FWD, HVX_FWD]>,
  6519: 
  6520:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  6521:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6522:        InstrStage<1, [CVI_LD], 0>,
  6523:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  6524:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  6525:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6526: 
  6527:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  6528:       [InstrStage<1, [SLOT0], 0>,
  6529:        InstrStage<1, [CVI_ST], 0>,
  6530:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  6531:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  6532:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  6533: 
  6534:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  6535:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6536:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  6537:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6538: 
  6539:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  6540:       [InstrStage<1, [SLOT0], 0>,
  6541:        InstrStage<1, [CVI_ST], 0>,
  6542:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  6543:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  6544: 
  6545:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  6546:       [InstrStage<1, [SLOT0], 0>,
  6547:        InstrStage<1, [CVI_ST], 0>,
  6548:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  6549:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6550: 
  6551:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  6552:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6553:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  6554:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6555: 
  6556:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  6557:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6558:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  6559:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6560: 
  6561:     InstrItinData <tc_a28f32b5, /*SLOT01,LOAD,VA*/
  6562:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6563:        InstrStage<1, [CVI_LD], 0>,
  6564:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  6565:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  6566: 
  6567:     InstrItinData <tc_a69eeee1, /*SLOT01,LOAD,VA_DV*/
  6568:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6569:        InstrStage<1, [CVI_LD], 0>,
  6570:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  6571:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6572: 
  6573:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  6574:       [InstrStage<1, [SLOT0], 0>,
  6575:        InstrStage<1, [SLOT1], 0>,
  6576:        InstrStage<1, [CVI_LD], 0>,
  6577:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  6578:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  6579: 
  6580:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  6581:       [InstrStage<1, [SLOT0], 0>,
  6582:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  6583:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  6584: 
  6585:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  6586:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6587:        InstrStage<1, [CVI_LD], 0>,
  6588:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  6589:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6590: 
  6591:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  6592:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6593:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  6594:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6595: 
  6596:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  6597:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6598:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  6599:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  6600: 
  6601:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  6602:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6603:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  6604:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6605: 
  6606:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  6607:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6608:        InstrStage<1, [CVI_ALL]>], [2],
  6609:       [Hex_FWD]>,
  6610: 
  6611:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  6612:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6613:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  6614:       [HVX_FWD, HVX_FWD]>,
  6615: 
  6616:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  6617:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6618:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  6619:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6620: 
  6621:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  6622:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6623:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  6624:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6625: 
  6626:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  6627:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6628:        InstrStage<1, [CVI_LD], 0>,
  6629:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  6630:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  6631: 
  6632:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  6633:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6634:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  6635:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6636: 
  6637:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  6638:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6639:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  6640:       [HVX_FWD, Hex_FWD]>,
  6641: 
  6642:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  6643:       [InstrStage<1, [SLOT0], 0>,
  6644:        InstrStage<1, [CVI_ST], 0>,
  6645:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  6646:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  6647: 
  6648:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  6649:       [InstrStage<1, [SLOT0], 0>,
  6650:        InstrStage<1, [SLOT1], 0>,
  6651:        InstrStage<1, [CVI_ST], 0>,
  6652:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  6653:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6654: 
  6655:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  6656:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6657:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  6658:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  6659:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6660: 
  6661:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  6662:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6663:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  6664:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6665: 
  6666:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  6667:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6668:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  6669:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6670: 
  6671:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  6672:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6673:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  6674:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6675: 
  6676:     InstrItinData <tc_dcca380f, /*SLOT23,VX*/
  6677:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6678:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  6679:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6680: 
  6681:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  6682:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6683:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  6684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6685: 
  6686:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  6687:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6688:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  6689:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6690: 
  6691:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  6692:       [InstrStage<1, [SLOT0], 0>,
  6693:        InstrStage<1, [SLOT1], 0>,
  6694:        InstrStage<1, [CVI_ST], 0>,
  6695:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  6696:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6697: 
  6698:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  6699:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5],
  6700:       [HVX_FWD, HVX_FWD]>,
  6701: 
  6702:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  6703:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6704:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  6705:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6706: 
  6707:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  6708:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6709:        InstrStage<1, [CVI_ALL]>], [3],
  6710:       [HVX_FWD]>,
  6711: 
  6712:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  6713:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6714:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  6715:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  6716: 
  6717:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  6718:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6719:        InstrStage<1, [CVI_ZW]>], [1, 2],
  6720:       [Hex_FWD, Hex_FWD]>,
  6721: 
  6722:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  6723:       [InstrStage<1, [SLOT0], 0>,
  6724:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  6725:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6726: 
  6727:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  6728:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6729:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  6730:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6731: 
  6732:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  6733:       [InstrStage<1, [SLOT2], 0>,
  6734:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  6735:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6736: 
  6737:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  6738:       [InstrStage<1, [SLOT0], 0>,
  6739:        InstrStage<1, [SLOT1], 0>,
  6740:        InstrStage<1, [CVI_ST], 0>,
  6741:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  6742:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  6743:   ];
  6744: }
  6745: 
  6746: class DepHVXItinV79 {
  6747:   list<InstrItinData> DepHVXItinV79_list = [
  6748:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  6749:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6750:        InstrStage<1, [CVI_LD], 0>,
```
- EN: It declares types such as DepHVXItinV79, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV79; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV79 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV79 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 6751-7000 / 第 6751-7000 行

```tablegen
  6751:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  6752:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  6753:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  6754: 
  6755:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  6756:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6757:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  6758:       [HVX_FWD, HVX_FWD]>,
  6759: 
  6760:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  6761:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6762:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  6763:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6764: 
  6765:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  6766:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6767:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  6768:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6769: 
  6770:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  6771:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6772:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  6773:       [HVX_FWD, HVX_FWD]>,
  6774: 
  6775:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  6776:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6777:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  6778:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  6779: 
  6780:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  6781:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6782:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  6783:       [HVX_FWD, HVX_FWD]>,
  6784: 
  6785:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  6786:       [InstrStage<1, [SLOT0], 0>,
  6787:        InstrStage<1, [SLOT1], 0>,
  6788:        InstrStage<1, [CVI_ST], 0>,
  6789:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  6790:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6791: 
  6792:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  6793:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6794:        InstrStage<1, [CVI_ALL]>], [],
  6795:       []>,
  6796: 
  6797:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  6798:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6799:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  6800:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  6801:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6802: 
  6803:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  6804:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6805:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  6806:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6807: 
  6808:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  6809:       [InstrStage<1, [SLOT0], 0>,
  6810:        InstrStage<1, [CVI_ST], 0>,
  6811:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  6812:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6813: 
  6814:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  6815:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6816:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  6817:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  6818: 
  6819:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  6820:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6821:        InstrStage<1, [CVI_LD], 0>,
  6822:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  6823:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6824: 
  6825:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  6826:       [InstrStage<1, [SLOT0], 0>,
  6827:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  6828:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6829: 
  6830:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  6831:       [InstrStage<1, [SLOT0], 0>,
  6832:        InstrStage<1, [CVI_ST], 0>,
  6833:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  6834:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  6835:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6836: 
  6837:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  6838:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6839:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  6840:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6841: 
  6842:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  6843:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6844:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  6845:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  6846: 
  6847:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  6848:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6849:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  6850:       [HVX_FWD, HVX_FWD]>,
  6851: 
  6852:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  6853:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6854:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  6855:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6856: 
  6857:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  6858:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6859:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  6860:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6861: 
  6862:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  6863:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6864:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  6865:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6866: 
  6867:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  6868:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6869:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  6870:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6871: 
  6872:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  6873:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6874:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  6875:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6876: 
  6877:     InstrItinData <tc_37820f4c, /*SLOT23,VX*/
  6878:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6879:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  6880:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  6881: 
  6882:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  6883:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6884:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  6885:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6886: 
  6887:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  6888:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6889:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  6890:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  6891: 
  6892:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  6893:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6894:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  6895:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6896: 
  6897:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  6898:       [InstrStage<1, [SLOT0], 0>,
  6899:        InstrStage<1, [SLOT1], 0>,
  6900:        InstrStage<1, [CVI_LD], 0>,
  6901:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  6902:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6903: 
  6904:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  6905:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6906:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  6907:       [HVX_FWD, HVX_FWD]>,
  6908: 
  6909:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  6910:       [InstrStage<1, [SLOT0], 0>,
  6911:        InstrStage<1, [CVI_ST]>], [1, 2],
  6912:       [Hex_FWD, Hex_FWD]>,
  6913: 
  6914:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  6915:       [InstrStage<1, [SLOT0], 0>,
  6916:        InstrStage<1, [CVI_ST], 0>,
  6917:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  6918:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6919: 
  6920:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  6921:       [InstrStage<1, [SLOT0], 0>,
  6922:        InstrStage<1, [CVI_ST], 0>,
  6923:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  6924:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6925: 
  6926:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  6927:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6928:        InstrStage<1, [CVI_LD], 0>,
  6929:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  6930:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6931: 
  6932:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  6933:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6934:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  6935:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6936: 
  6937:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  6938:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  6939:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  6940:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6941: 
  6942:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  6943:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6944:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  6945:       [HVX_FWD, HVX_FWD]>,
  6946: 
  6947:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  6948:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  6949:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  6950:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  6951: 
  6952:     InstrItinData <tc_531b383c, /*SLOT0123*/
  6953:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5, 5],
  6954:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6955: 
  6956:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  6957:       [InstrStage<1, [SLOT0], 0>,
  6958:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  6959:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  6960: 
  6961:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  6962:       [InstrStage<1, [SLOT0], 0>,
  6963:        InstrStage<1, [CVI_ST], 0>,
  6964:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  6965:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  6966: 
  6967:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  6968:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6969:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  6970:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6971: 
  6972:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  6973:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6974:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  6975:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  6976: 
  6977:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  6978:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6979:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  6980:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6981: 
  6982:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  6983:       [InstrStage<1, [SLOT0], 0>,
  6984:        InstrStage<1, [CVI_ST], 0>,
  6985:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  6986:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  6987: 
  6988:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  6989:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6990:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  6991:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  6992: 
  6993:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  6994:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  6995:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  6996:       [HVX_FWD, Hex_FWD]>,
  6997: 
  6998:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  6999:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7000:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 7001-7250 / 第 7001-7250 行

```tablegen
  7001:       [HVX_FWD, HVX_FWD]>,
  7002: 
  7003:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  7004:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7005:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  7006:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7007: 
  7008:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  7009:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7010:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  7011:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7012: 
  7013:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  7014:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7015:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  7016:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7017: 
  7018:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  7019:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7020:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  7021:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7022: 
  7023:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  7024:       [InstrStage<1, [SLOT0], 0>,
  7025:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  7026:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7027: 
  7028:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  7029:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7030:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  7031:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7032: 
  7033:     InstrItinData <tc_7095ecba, /*SLOT01,LOAD,VA_DV*/
  7034:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7035:        InstrStage<1, [CVI_LD], 0>,
  7036:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  7037:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7038: 
  7039:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  7040:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7041:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  7042:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7043: 
  7044:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  7045:       [InstrStage<1, [SLOT0], 0>,
  7046:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  7047:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7048: 
  7049:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  7050:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7051:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  7052:       [HVX_FWD]>,
  7053: 
  7054:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  7055:       [InstrStage<1, [SLOT0], 0>,
  7056:        InstrStage<1, [CVI_ST], 0>,
  7057:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  7058:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7059: 
  7060:     InstrItinData <tc_72e2b393, /*SLOT23,VX*/
  7061:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7062:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  7063:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7064: 
  7065:     InstrItinData <tc_73efe966, /*SLOT23,VX*/
  7066:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7067:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  7068:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7069: 
  7070:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  7071:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7072:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  7073:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7074: 
  7075:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  7076:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7077:        InstrStage<1, [CVI_ALL]>], [3, 2],
  7078:       [HVX_FWD, Hex_FWD]>,
  7079: 
  7080:     InstrItinData <tc_7d68d5c2, /*SLOT01,LOAD,VA*/
  7081:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7082:        InstrStage<1, [CVI_LD], 0>,
  7083:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  7084:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7085: 
  7086:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  7087:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7088:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  7089:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7090: 
  7091:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  7092:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7093:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  7094:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7095: 
  7096:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  7097:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7098:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  7099:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7100: 
  7101:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  7102:       [InstrStage<1, [SLOT0], 0>,
  7103:        InstrStage<1, [CVI_ST], 0>,
  7104:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  7105:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7106: 
  7107:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  7108:       [InstrStage<1, [SLOT2], 0>,
  7109:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  7110:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7111: 
  7112:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  7113:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7114:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  7115:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7116: 
  7117:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  7118:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7119:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  7120:       [HVX_FWD, HVX_FWD]>,
  7121: 
  7122:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  7123:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7124:        InstrStage<1, [CVI_LD], 0>,
  7125:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  7126:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  7127:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7128: 
  7129:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  7130:       [InstrStage<1, [SLOT0], 0>,
  7131:        InstrStage<1, [CVI_ST], 0>,
  7132:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  7133:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  7134:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7135: 
  7136:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  7137:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7138:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  7139:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7140: 
  7141:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  7142:       [InstrStage<1, [SLOT0], 0>,
  7143:        InstrStage<1, [CVI_ST], 0>,
  7144:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  7145:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7146: 
  7147:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  7148:       [InstrStage<1, [SLOT0], 0>,
  7149:        InstrStage<1, [CVI_ST], 0>,
  7150:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  7151:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7152: 
  7153:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  7154:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7155:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  7156:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7157: 
  7158:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  7159:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7160:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  7161:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7162: 
  7163:     InstrItinData <tc_a28f32b5, /*SLOT01,LOAD,VA*/
  7164:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7165:        InstrStage<1, [CVI_LD], 0>,
  7166:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  7167:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7168: 
  7169:     InstrItinData <tc_a69eeee1, /*SLOT01,LOAD,VA_DV*/
  7170:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7171:        InstrStage<1, [CVI_LD], 0>,
  7172:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  7173:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7174: 
  7175:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  7176:       [InstrStage<1, [SLOT0], 0>,
  7177:        InstrStage<1, [SLOT1], 0>,
  7178:        InstrStage<1, [CVI_LD], 0>,
  7179:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  7180:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  7181: 
  7182:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  7183:       [InstrStage<1, [SLOT0], 0>,
  7184:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  7185:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7186: 
  7187:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  7188:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7189:        InstrStage<1, [CVI_LD], 0>,
  7190:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  7191:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7192: 
  7193:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  7194:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7195:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  7196:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7197: 
  7198:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  7199:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7200:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  7201:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7202: 
  7203:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  7204:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7205:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  7206:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7207: 
  7208:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  7209:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7210:        InstrStage<1, [CVI_ALL]>], [2],
  7211:       [Hex_FWD]>,
  7212: 
  7213:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  7214:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7215:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  7216:       [HVX_FWD, HVX_FWD]>,
  7217: 
  7218:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  7219:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7220:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  7221:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7222: 
  7223:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  7224:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7225:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  7226:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7227: 
  7228:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  7229:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7230:        InstrStage<1, [CVI_LD], 0>,
  7231:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  7232:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  7233: 
  7234:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  7235:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7236:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  7237:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7238: 
  7239:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  7240:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7241:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  7242:       [HVX_FWD, Hex_FWD]>,
  7243: 
  7244:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  7245:       [InstrStage<1, [SLOT0], 0>,
  7246:        InstrStage<1, [CVI_ST], 0>,
  7247:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  7248:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7249: 
  7250:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 7251-7500 / 第 7251-7500 行

```tablegen
  7251:       [InstrStage<1, [SLOT0], 0>,
  7252:        InstrStage<1, [SLOT1], 0>,
  7253:        InstrStage<1, [CVI_ST], 0>,
  7254:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  7255:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7256: 
  7257:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  7258:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7259:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  7260:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  7261:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7262: 
  7263:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  7264:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7265:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  7266:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7267: 
  7268:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  7269:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7270:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  7271:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7272: 
  7273:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  7274:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7275:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  7276:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7277: 
  7278:     InstrItinData <tc_dcca380f, /*SLOT23,VX*/
  7279:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7280:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  7281:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7282: 
  7283:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  7284:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7285:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  7286:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7287: 
  7288:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  7289:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7290:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  7291:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7292: 
  7293:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  7294:       [InstrStage<1, [SLOT0], 0>,
  7295:        InstrStage<1, [SLOT1], 0>,
  7296:        InstrStage<1, [CVI_ST], 0>,
  7297:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  7298:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7299: 
  7300:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  7301:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5],
  7302:       [HVX_FWD, HVX_FWD]>,
  7303: 
  7304:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  7305:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7306:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  7307:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7308: 
  7309:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  7310:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7311:        InstrStage<1, [CVI_ALL]>], [3],
  7312:       [HVX_FWD]>,
  7313: 
  7314:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  7315:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7316:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  7317:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  7318: 
  7319:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  7320:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7321:        InstrStage<1, [CVI_ZW]>], [1, 2],
  7322:       [Hex_FWD, Hex_FWD]>,
  7323: 
  7324:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  7325:       [InstrStage<1, [SLOT0], 0>,
  7326:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  7327:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7328: 
  7329:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  7330:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7331:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  7332:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7333: 
  7334:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  7335:       [InstrStage<1, [SLOT2], 0>,
  7336:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  7337:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7338: 
  7339:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  7340:       [InstrStage<1, [SLOT0], 0>,
  7341:        InstrStage<1, [SLOT1], 0>,
  7342:        InstrStage<1, [CVI_ST], 0>,
  7343:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  7344:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  7345:   ];
  7346: }
  7347: 
  7348: class DepHVXItinV81 {
  7349:   list<InstrItinData> DepHVXItinV81_list = [
  7350:     InstrItinData <tc_0390c1ca, /*SLOT01,LOAD,VA,VX_DV*/
  7351:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7352:        InstrStage<1, [CVI_LD], 0>,
  7353:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  7354:        InstrStage<1, [CVI_MPY01]>], [9, 1, 2],
  7355:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  7356: 
  7357:     InstrItinData <tc_04da405a, /*SLOT0123,VP_VS*/
  7358:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7359:        InstrStage<1, [CVI_XLSHF]>], [9, 5],
  7360:       [HVX_FWD, HVX_FWD]>,
  7361: 
  7362:     InstrItinData <tc_05ca8cfd, /*SLOT0123,VS*/
  7363:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7364:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5],
  7365:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7366: 
  7367:     InstrItinData <tc_08a4f1b6, /*SLOT23,VX_DV*/
  7368:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7369:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5],
  7370:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7371: 
  7372:     InstrItinData <tc_0afc8be9, /*SLOT23,VX_DV*/
  7373:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7374:        InstrStage<1, [CVI_MPY01]>], [9, 5],
  7375:       [HVX_FWD, HVX_FWD]>,
  7376: 
  7377:     InstrItinData <tc_0b04c6c7, /*SLOT23,VX_DV*/
  7378:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7379:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  7380:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7381: 
  7382:     InstrItinData <tc_0ec46cf9, /*SLOT0123,VA*/
  7383:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7384:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7],
  7385:       [HVX_FWD, HVX_FWD]>,
  7386: 
  7387:     InstrItinData <tc_131f1c81, /*SLOT0,NOSLOT1,STORE,VP*/
  7388:       [InstrStage<1, [SLOT0], 0>,
  7389:        InstrStage<1, [SLOT1], 0>,
  7390:        InstrStage<1, [CVI_ST], 0>,
  7391:        InstrStage<1, [CVI_XLANE]>], [2, 1, 2, 5],
  7392:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7393: 
  7394:     InstrItinData <tc_1381a97c, /*SLOT0123,4SLOT*/
  7395:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7396:        InstrStage<1, [CVI_ALL]>], [],
  7397:       []>,
  7398: 
  7399:     InstrItinData <tc_15fdf750, /*SLOT23,VS_VX*/
  7400:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7401:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  7402:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 7, 5, 2],
  7403:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7404: 
  7405:     InstrItinData <tc_16ff9ef8, /*SLOT0123,VS*/
  7406:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7407:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 5, 2],
  7408:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7409: 
  7410:     InstrItinData <tc_191381c1, /*SLOT0,STORE,VA*/
  7411:       [InstrStage<1, [SLOT0], 0>,
  7412:        InstrStage<1, [CVI_ST], 0>,
  7413:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 7, 1, 2, 7],
  7414:       [Hex_FWD, HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7415: 
  7416:     InstrItinData <tc_1ad8a370, /*SLOT23,VX_DV*/
  7417:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7418:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2, 2],
  7419:       [HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  7420: 
  7421:     InstrItinData <tc_1ba8a0cd, /*SLOT01,LOAD,VA*/
  7422:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7423:        InstrStage<1, [CVI_LD], 0>,
  7424:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 1, 2],
  7425:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7426: 
  7427:     InstrItinData <tc_20a4bbec, /*SLOT0,STORE*/
  7428:       [InstrStage<1, [SLOT0], 0>,
  7429:        InstrStage<1, [CVI_ST]>], [3, 1, 2],
  7430:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7431: 
  7432:     InstrItinData <tc_227864f7, /*SLOT0,STORE,VA,VX_DV*/
  7433:       [InstrStage<1, [SLOT0], 0>,
  7434:        InstrStage<1, [CVI_ST], 0>,
  7435:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  7436:        InstrStage<1, [CVI_MPY01]>], [3, 1, 2, 5],
  7437:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7438: 
  7439:     InstrItinData <tc_257f6f7c, /*SLOT0123,VA*/
  7440:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7441:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 7],
  7442:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7443: 
  7444:     InstrItinData <tc_26a377fe, /*SLOT23,4SLOT_MPY*/
  7445:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7446:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 5, 2],
  7447:       [HVX_FWD, Hex_FWD, HVX_FWD, Hex_FWD]>,
  7448: 
  7449:     InstrItinData <tc_2a698a03, /*SLOT0123,VSorVP*/
  7450:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7451:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5],
  7452:       [HVX_FWD, HVX_FWD]>,
  7453: 
  7454:     InstrItinData <tc_2b4c548e, /*SLOT23,VX_DV*/
  7455:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7456:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  7457:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7458: 
  7459:     InstrItinData <tc_2c745bb8, /*SLOT0123,VP_VS*/
  7460:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7461:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5],
  7462:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7463: 
  7464:     InstrItinData <tc_2d4051cd, /*SLOT23,4SLOT_MPY*/
  7465:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7466:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 3, 7, 5, 2],
  7467:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7468: 
  7469:     InstrItinData <tc_2e8f5f6e, /*SLOT23,VX*/
  7470:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7471:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7, 2],
  7472:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7473: 
  7474:     InstrItinData <tc_309dbb4f, /*SLOT0123,VS*/
  7475:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7476:        InstrStage<1, [CVI_SHIFT]>], [9, 7, 5, 2],
  7477:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7478: 
  7479:     InstrItinData <tc_37820f4c, /*SLOT23,VX*/
  7480:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7481:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  7482:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7483: 
  7484:     InstrItinData <tc_3904b926, /*SLOT01,LOAD*/
  7485:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7486:        InstrStage<1, [CVI_LD]>], [9, 2, 1, 2],
  7487:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7488: 
  7489:     InstrItinData <tc_3aacf4a8, /*SLOT0123,VA*/
  7490:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7491:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 7],
  7492:       [HVX_FWD, Hex_FWD, HVX_FWD]>,
  7493: 
  7494:     InstrItinData <tc_3ad719fb, /*SLOT01,ZW*/
  7495:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7496:        InstrStage<1, [CVI_ZW]>], [3, 2, 1, 2],
  7497:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7498: 
  7499:     InstrItinData <tc_3c56e5ce, /*SLOT0,NOSLOT1,LOAD,VP*/
  7500:       [InstrStage<1, [SLOT0], 0>,
```
- EN: It declares types such as DepHVXItinV81, which carry the state or API of this component. It defines generated/declarative TableGen records like DepHVXItinV81; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepHVXItinV81 等类型，用来承载该组件的状态或接口。 这里定义了 DepHVXItinV81 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 7501-7750 / 第 7501-7750 行

```tablegen
  7501:        InstrStage<1, [SLOT1], 0>,
  7502:        InstrStage<1, [CVI_LD], 0>,
  7503:        InstrStage<1, [CVI_XLANE]>], [9, 3, 1, 2],
  7504:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7505: 
  7506:     InstrItinData <tc_3c8c15d0, /*SLOT23,VX*/
  7507:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7508:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5],
  7509:       [HVX_FWD, HVX_FWD]>,
  7510: 
  7511:     InstrItinData <tc_3ce09744, /*SLOT0,STORE*/
  7512:       [InstrStage<1, [SLOT0], 0>,
  7513:        InstrStage<1, [CVI_ST]>], [1, 2],
  7514:       [Hex_FWD, Hex_FWD]>,
  7515: 
  7516:     InstrItinData <tc_3e2aaafc, /*SLOT0,STORE,VA*/
  7517:       [InstrStage<1, [SLOT0], 0>,
  7518:        InstrStage<1, [CVI_ST], 0>,
  7519:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 1, 2, 7],
  7520:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7521: 
  7522:     InstrItinData <tc_447d9895, /*SLOT0,STORE,VA*/
  7523:       [InstrStage<1, [SLOT0], 0>,
  7524:        InstrStage<1, [CVI_ST], 0>,
  7525:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  7526:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7527: 
  7528:     InstrItinData <tc_453fe68d, /*SLOT01,LOAD,VA*/
  7529:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7530:        InstrStage<1, [CVI_LD], 0>,
  7531:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 3, 2, 1, 2],
  7532:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7533: 
  7534:     InstrItinData <tc_46d6c3e0, /*SLOT0123,VP*/
  7535:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7536:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5],
  7537:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7538: 
  7539:     InstrItinData <tc_4942646a, /*SLOT23,VX*/
  7540:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7541:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5, 2],
  7542:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7543: 
  7544:     InstrItinData <tc_51d0ecc3, /*SLOT0123,VS*/
  7545:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7546:        InstrStage<1, [CVI_SHIFT]>], [9, 5],
  7547:       [HVX_FWD, HVX_FWD]>,
  7548: 
  7549:     InstrItinData <tc_52447ecc, /*SLOT01,LOAD*/
  7550:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7551:        InstrStage<1, [CVI_LD]>], [9, 1, 2],
  7552:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  7553: 
  7554:     InstrItinData <tc_531b383c, /*SLOT0123*/
  7555:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5, 5],
  7556:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7557: 
  7558:     InstrItinData <tc_540c3da3, /*SLOT0,VA*/
  7559:       [InstrStage<1, [SLOT0], 0>,
  7560:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [4, 7, 1],
  7561:       [Hex_FWD, HVX_FWD, Hex_FWD]>,
  7562: 
  7563:     InstrItinData <tc_54a0dc47, /*SLOT0,STORE,VA*/
  7564:       [InstrStage<1, [SLOT0], 0>,
  7565:        InstrStage<1, [CVI_ST], 0>,
  7566:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [3, 2, 1, 2, 7],
  7567:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7568: 
  7569:     InstrItinData <tc_561aaa58, /*SLOT0123,VP_VS*/
  7570:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7571:        InstrStage<1, [CVI_XLSHF]>], [9, 9, 5, 5, 2],
  7572:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7573: 
  7574:     InstrItinData <tc_56c4f9fe, /*SLOT0123,VA*/
  7575:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7576:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  7577:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7578: 
  7579:     InstrItinData <tc_56e64202, /*SLOT0123,VP*/
  7580:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7581:        InstrStage<1, [CVI_XLANE]>], [9, 5, 5, 2],
  7582:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7583: 
  7584:     InstrItinData <tc_58d21193, /*SLOT0,STORE,VA_DV*/
  7585:       [InstrStage<1, [SLOT0], 0>,
  7586:        InstrStage<1, [CVI_ST], 0>,
  7587:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7, 7],
  7588:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7589: 
  7590:     InstrItinData <tc_57a4709c, /*SLOT0123,VA*/
  7591:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7592:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7, 2],
  7593:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7594: 
  7595:     InstrItinData <tc_5bf8afbb, /*SLOT0123,VP*/
  7596:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7597:        InstrStage<1, [CVI_XLANE]>], [9, 2],
  7598:       [HVX_FWD, Hex_FWD]>,
  7599: 
  7600:     InstrItinData <tc_5cdf8c84, /*SLOT23,VX*/
  7601:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7602:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7],
  7603:       [HVX_FWD, HVX_FWD]>,
  7604: 
  7605:     InstrItinData <tc_61bf7c03, /*SLOT23,4SLOT_MPY*/
  7606:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7607:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 5, 2],
  7608:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7609: 
  7610:     InstrItinData <tc_649072c2, /*SLOT23,VX*/
  7611:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7612:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  7613:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7614: 
  7615:     InstrItinData <tc_660769f1, /*SLOT23,VX_DV*/
  7616:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7617:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2],
  7618:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7619: 
  7620:     InstrItinData <tc_663c80a7, /*SLOT01,LOAD*/
  7621:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7622:        InstrStage<1, [CVI_LD]>], [9, 3, 1, 2],
  7623:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7624: 
  7625:     InstrItinData <tc_6942b6e0, /*SLOT0,STORE*/
  7626:       [InstrStage<1, [SLOT0], 0>,
  7627:        InstrStage<1, [CVI_ST]>], [3, 1, 2, 5],
  7628:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7629: 
  7630:     InstrItinData <tc_6e7fa133, /*SLOT0123,VP*/
  7631:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7632:        InstrStage<1, [CVI_XLANE]>], [9, 5, 2],
  7633:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7634: 
  7635:     InstrItinData <tc_7095ecba, /*SLOT01,LOAD,VA_DV*/
  7636:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7637:        InstrStage<1, [CVI_LD], 0>,
  7638:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7],
  7639:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7640: 
  7641:     InstrItinData <tc_71646d06, /*SLOT0123,VA_DV*/
  7642:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7643:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7, 7],
  7644:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7645: 
  7646:     InstrItinData <tc_7177e272, /*SLOT0,STORE*/
  7647:       [InstrStage<1, [SLOT0], 0>,
  7648:        InstrStage<1, [CVI_ST]>], [2, 1, 2, 5],
  7649:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7650: 
  7651:     InstrItinData <tc_718b5c53, /*SLOT0123,VA_DV*/
  7652:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7653:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9],
  7654:       [HVX_FWD]>,
  7655: 
  7656:     InstrItinData <tc_7273323b, /*SLOT0,STORE,VA_DV*/
  7657:       [InstrStage<1, [SLOT0], 0>,
  7658:        InstrStage<1, [CVI_ST], 0>,
  7659:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [1, 2, 7, 7],
  7660:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7661: 
  7662:     InstrItinData <tc_72e2b393, /*SLOT23,VX*/
  7663:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7664:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  7665:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7666: 
  7667:     InstrItinData <tc_73efe966, /*SLOT23,VX*/
  7668:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7669:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  7670:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7671: 
  7672:     InstrItinData <tc_7417e785, /*SLOT0123,VS*/
  7673:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7674:        InstrStage<1, [CVI_SHIFT]>], [9, 5, 2],
  7675:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7676: 
  7677:     InstrItinData <tc_767c4e9d, /*SLOT0123,4SLOT*/
  7678:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7679:        InstrStage<1, [CVI_ALL]>], [3, 2],
  7680:       [HVX_FWD, Hex_FWD]>,
  7681: 
  7682:     InstrItinData <tc_7d68d5c2, /*SLOT01,LOAD,VA*/
  7683:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7684:        InstrStage<1, [CVI_LD], 0>,
  7685:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7],
  7686:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7687: 
  7688:     InstrItinData <tc_7e6a3e89, /*SLOT0123,VA*/
  7689:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7690:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7, 7],
  7691:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7692: 
  7693:     InstrItinData <tc_8772086c, /*SLOT0123,VA*/
  7694:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7695:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 7, 7],
  7696:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7697: 
  7698:     InstrItinData <tc_87adc037, /*SLOT0123,VP_VS*/
  7699:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7700:        InstrStage<1, [CVI_XLSHF]>], [9, 5, 5, 2],
  7701:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7702: 
  7703:     InstrItinData <tc_8e420e4d, /*SLOT0,STORE,VA*/
  7704:       [InstrStage<1, [SLOT0], 0>,
  7705:        InstrStage<1, [CVI_ST], 0>,
  7706:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [7, 1, 2, 7, 7],
  7707:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7708: 
  7709:     InstrItinData <tc_90bcc1db, /*SLOT2,VX_DV*/
  7710:       [InstrStage<1, [SLOT2], 0>,
  7711:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5, 2],
  7712:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7713: 
  7714:     InstrItinData <tc_933f2b39, /*SLOT23,4SLOT_MPY*/
  7715:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7716:        InstrStage<1, [CVI_ALL_NOMEM]>], [9, 7, 5, 2],
  7717:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7718: 
  7719:     InstrItinData <tc_946013d8, /*SLOT0123,VP*/
  7720:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7721:        InstrStage<1, [CVI_XLANE]>], [9, 5],
  7722:       [HVX_FWD, HVX_FWD]>,
  7723: 
  7724:     InstrItinData <tc_9a1cab75, /*SLOT01,LOAD,VA,VX_DV*/
  7725:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7726:        InstrStage<1, [CVI_LD], 0>,
  7727:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  7728:        InstrStage<1, [CVI_MPY01]>], [9, 3, 1, 2],
  7729:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7730: 
  7731:     InstrItinData <tc_9aff7a2a, /*SLOT0,STORE,VA,VX_DV*/
  7732:       [InstrStage<1, [SLOT0], 0>,
  7733:        InstrStage<1, [CVI_ST], 0>,
  7734:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE], 0>,
  7735:        InstrStage<1, [CVI_MPY01]>], [1, 2, 5],
  7736:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7737: 
  7738:     InstrItinData <tc_9d1dc972, /*SLOT0123,VP_VS*/
  7739:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7740:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5, 2],
  7741:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7742: 
  7743:     InstrItinData <tc_9f363d21, /*SLOT0,STORE,VA*/
  7744:       [InstrStage<1, [SLOT0], 0>,
  7745:        InstrStage<1, [CVI_ST], 0>,
  7746:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7, 7],
  7747:       [Hex_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7748: 
  7749:     InstrItinData <tc_a02a10a8, /*SLOT0,STORE,VA*/
  7750:       [InstrStage<1, [SLOT0], 0>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 7751-7948 / 第 7751-7948 行

```tablegen
  7751:        InstrStage<1, [CVI_ST], 0>,
  7752:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [2, 1, 2, 7],
  7753:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7754: 
  7755:     InstrItinData <tc_a0dbea28, /*SLOT01,ZW*/
  7756:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7757:        InstrStage<1, [CVI_ZW]>], [3, 1, 2],
  7758:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7759: 
  7760:     InstrItinData <tc_a19b9305, /*SLOT23,VX*/
  7761:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7762:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 5],
  7763:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7764: 
  7765:     InstrItinData <tc_a28f32b5, /*SLOT01,LOAD,VA*/
  7766:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7767:        InstrStage<1, [CVI_LD], 0>,
  7768:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  7769:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7770: 
  7771:     InstrItinData <tc_a69eeee1, /*SLOT01,LOAD,VA_DV*/
  7772:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7773:        InstrStage<1, [CVI_LD], 0>,
  7774:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [7, 1, 2, 7],
  7775:       [HVX_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7776: 
  7777:     InstrItinData <tc_a7e6707d, /*SLOT0,NOSLOT1,LOAD,VP*/
  7778:       [InstrStage<1, [SLOT0], 0>,
  7779:        InstrStage<1, [SLOT1], 0>,
  7780:        InstrStage<1, [CVI_LD], 0>,
  7781:        InstrStage<1, [CVI_XLANE]>], [9, 1, 2],
  7782:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  7783: 
  7784:     InstrItinData <tc_ab23f776, /*SLOT0,STORE*/
  7785:       [InstrStage<1, [SLOT0], 0>,
  7786:        InstrStage<1, [CVI_ST]>], [1, 2, 5],
  7787:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7788: 
  7789:     InstrItinData <tc_abe8c3b2, /*SLOT01,LOAD,VA*/
  7790:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7791:        InstrStage<1, [CVI_LD], 0>,
  7792:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 2, 1, 2],
  7793:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7794: 
  7795:     InstrItinData <tc_ac4046bc, /*SLOT23,VX*/
  7796:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7797:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 2],
  7798:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7799: 
  7800:     InstrItinData <tc_af25efd9, /*SLOT0123,VA_DV*/
  7801:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7802:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 2, 7, 7],
  7803:       [HVX_FWD, Hex_FWD, HVX_FWD, HVX_FWD]>,
  7804: 
  7805:     InstrItinData <tc_b091f1c6, /*SLOT23,VX*/
  7806:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7807:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 5, 2],
  7808:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7809: 
  7810:     InstrItinData <tc_b28e51aa, /*SLOT0123,4SLOT*/
  7811:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7812:        InstrStage<1, [CVI_ALL]>], [2],
  7813:       [Hex_FWD]>,
  7814: 
  7815:     InstrItinData <tc_b4416217, /*SLOT0123,VA_DV*/
  7816:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7817:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7],
  7818:       [HVX_FWD, HVX_FWD]>,
  7819: 
  7820:     InstrItinData <tc_b9db8205, /*SLOT01,LOAD*/
  7821:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7822:        InstrStage<1, [CVI_LD]>], [9, 3, 2, 1, 2],
  7823:       [HVX_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7824: 
  7825:     InstrItinData <tc_bb599486, /*SLOT23,VX_DV*/
  7826:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7827:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 5, 2],
  7828:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7829: 
  7830:     InstrItinData <tc_c0749f3c, /*SLOT01,LOAD,VA*/
  7831:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7832:        InstrStage<1, [CVI_LD], 0>,
  7833:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 1, 2],
  7834:       [HVX_FWD, Hex_FWD, Hex_FWD]>,
  7835: 
  7836:     InstrItinData <tc_c127de3a, /*SLOT23,VX*/
  7837:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7838:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5],
  7839:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7840: 
  7841:     InstrItinData <tc_c4edf264, /*SLOT23,VX*/
  7842:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7843:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 2],
  7844:       [HVX_FWD, Hex_FWD]>,
  7845: 
  7846:     InstrItinData <tc_c5dba46e, /*SLOT0,STORE,VA*/
  7847:       [InstrStage<1, [SLOT0], 0>,
  7848:        InstrStage<1, [CVI_ST], 0>,
  7849:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [1, 2, 7],
  7850:       [Hex_FWD, Hex_FWD, HVX_FWD]>,
  7851: 
  7852:     InstrItinData <tc_c7039829, /*SLOT0,NOSLOT1,STORE,VP*/
  7853:       [InstrStage<1, [SLOT0], 0>,
  7854:        InstrStage<1, [SLOT1], 0>,
  7855:        InstrStage<1, [CVI_ST], 0>,
  7856:        InstrStage<1, [CVI_XLANE]>], [3, 2, 1, 2, 5],
  7857:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7858: 
  7859:     InstrItinData <tc_cd94bfe0, /*SLOT23,VS_VX*/
  7860:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7861:        InstrStage<1, [CVI_MPY0, CVI_MPY1], 0>,
  7862:        InstrStage<1, [CVI_SHIFT, CVI_XLANE]>], [9, 5, 2],
  7863:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7864: 
  7865:     InstrItinData <tc_cda936da, /*SLOT23,VX*/
  7866:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7867:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 7, 7],
  7868:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7869: 
  7870:     InstrItinData <tc_d8287c14, /*SLOT23,VX_DV*/
  7871:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7872:        InstrStage<1, [CVI_MPY01]>], [9, 5, 5],
  7873:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7874: 
  7875:     InstrItinData <tc_db5555f3, /*SLOT0123,VA_DV*/
  7876:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7877:        InstrStage<1, [CVI_MPY01, CVI_XLSHF]>], [9, 7, 7],
  7878:       [HVX_FWD, HVX_FWD, HVX_FWD]>,
  7879: 
  7880:     InstrItinData <tc_dcca380f, /*SLOT23,VX*/
  7881:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7882:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 2],
  7883:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7884: 
  7885:     InstrItinData <tc_dd5b0695, /*SLOT01,ZW*/
  7886:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7887:        InstrStage<1, [CVI_ZW]>], [2, 1, 2],
  7888:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7889: 
  7890:     InstrItinData <tc_df80eeb0, /*SLOT0123,VP_VS*/
  7891:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7892:        InstrStage<1, [CVI_XLSHF]>], [9, 7, 5, 5],
  7893:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7894: 
  7895:     InstrItinData <tc_e2d2e9e5, /*SLOT0,NOSLOT1,STORE,VP*/
  7896:       [InstrStage<1, [SLOT0], 0>,
  7897:        InstrStage<1, [SLOT1], 0>,
  7898:        InstrStage<1, [CVI_ST], 0>,
  7899:        InstrStage<1, [CVI_XLANE]>], [3, 1, 2, 5],
  7900:       [Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7901: 
  7902:     InstrItinData <tc_e2fdd6e6, /*SLOT0123*/
  7903:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [9, 5],
  7904:       [HVX_FWD, HVX_FWD]>,
  7905: 
  7906:     InstrItinData <tc_e35c1e93, /*SLOT0123,VA*/
  7907:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7908:        InstrStage<1, [CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE]>], [9, 9, 7, 7],
  7909:       [HVX_FWD, HVX_FWD, HVX_FWD, HVX_FWD]>,
  7910: 
  7911:     InstrItinData <tc_e3f68a46, /*SLOT0123,4SLOT*/
  7912:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  7913:        InstrStage<1, [CVI_ALL]>], [3],
  7914:       [HVX_FWD]>,
  7915: 
  7916:     InstrItinData <tc_e675c45a, /*SLOT23,VX_DV*/
  7917:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7918:        InstrStage<1, [CVI_MPY01]>], [9, 7, 5, 2, 2],
  7919:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD, Hex_FWD]>,
  7920: 
  7921:     InstrItinData <tc_e699ae41, /*SLOT01,ZW*/
  7922:       [InstrStage<1, [SLOT0, SLOT1], 0>,
  7923:        InstrStage<1, [CVI_ZW]>], [1, 2],
  7924:       [Hex_FWD, Hex_FWD]>,
  7925: 
  7926:     InstrItinData <tc_e99d4c2e, /*SLOT0,STORE*/
  7927:       [InstrStage<1, [SLOT0], 0>,
  7928:        InstrStage<1, [CVI_ST]>], [3, 2, 1, 2, 5],
  7929:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, HVX_FWD]>,
  7930: 
  7931:     InstrItinData <tc_f175e046, /*SLOT23,VX*/
  7932:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  7933:        InstrStage<1, [CVI_MPY0, CVI_MPY1]>], [9, 5, 5, 2],
  7934:       [HVX_FWD, HVX_FWD, HVX_FWD, Hex_FWD]>,
  7935: 
  7936:     InstrItinData <tc_f1de44ef, /*SLOT2,VX_DV*/
  7937:       [InstrStage<1, [SLOT2], 0>,
  7938:        InstrStage<1, [CVI_MPY01]>], [9, 5, 2],
  7939:       [HVX_FWD, HVX_FWD, Hex_FWD]>,
  7940: 
  7941:     InstrItinData <tc_f21e8abb, /*SLOT0,NOSLOT1,STORE,VP*/
  7942:       [InstrStage<1, [SLOT0], 0>,
  7943:        InstrStage<1, [SLOT1], 0>,
  7944:        InstrStage<1, [CVI_ST], 0>,
  7945:        InstrStage<1, [CVI_XLANE]>], [1, 2, 5],
  7946:       [Hex_FWD, Hex_FWD, HVX_FWD]>
  7947:   ];
  7948: }
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录
- generated target metadata / 生成的目标元数据

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
