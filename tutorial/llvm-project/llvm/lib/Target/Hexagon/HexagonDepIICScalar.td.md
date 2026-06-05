# HexagonDepIICScalar.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonDepIICScalar.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon instruction itinerary/scheduling classes using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。 重点涉及指令语义与选择。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。
- Generation status / 生成状态: The file appears to be auto-generated or largely generated from upstream target data. / 该文件看起来是自动生成的，或主要由上游目标数据生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-500 / 第 1-500 行

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
    11: def tc_011e0e9d : InstrItinClass;
    12: def tc_01d44cb2 : InstrItinClass;
    13: def tc_01e1be3b : InstrItinClass;
    14: def tc_02fe1c65 : InstrItinClass;
    15: def tc_0655b949 : InstrItinClass;
    16: def tc_075c8dd8 : InstrItinClass;
    17: def tc_0a195f2c : InstrItinClass;
    18: def tc_0a43be35 : InstrItinClass;
    19: def tc_0a6c20ae : InstrItinClass;
    20: def tc_0ba0d5da : InstrItinClass;
    21: def tc_0dfac0a7 : InstrItinClass;
    22: def tc_0fac1eb8 : InstrItinClass;
    23: def tc_112d30d6 : InstrItinClass;
    24: def tc_1242dc2a : InstrItinClass;
    25: def tc_1248597c : InstrItinClass;
    26: def tc_139ef484 : InstrItinClass;
    27: def tc_14ab4f41 : InstrItinClass;
    28: def tc_151bf368 : InstrItinClass;
    29: def tc_158aa3f7 : InstrItinClass;
    30: def tc_197dce51 : InstrItinClass;
    31: def tc_1981450d : InstrItinClass;
    32: def tc_1c2c7a4a : InstrItinClass;
    33: def tc_1c7522a8 : InstrItinClass;
    34: def tc_1d41f8b7 : InstrItinClass;
    35: def tc_1fcb8495 : InstrItinClass;
    36: def tc_1fe4ab69 : InstrItinClass;
    37: def tc_20131976 : InstrItinClass;
    38: def tc_2237d952 : InstrItinClass;
    39: def tc_23708a21 : InstrItinClass;
    40: def tc_2471c1c8 : InstrItinClass;
    41: def tc_24e109c7 : InstrItinClass;
    42: def tc_24f426ab : InstrItinClass;
    43: def tc_27106296 : InstrItinClass;
    44: def tc_280f7fe1 : InstrItinClass;
    45: def tc_28e55c6f : InstrItinClass;
    46: def tc_2c13e7f5 : InstrItinClass;
    47: def tc_2c3e17fc : InstrItinClass;
    48: def tc_2f573607 : InstrItinClass;
    49: def tc_33e7e673 : InstrItinClass;
    50: def tc_362b0be2 : InstrItinClass;
    51: def tc_38382228 : InstrItinClass;
    52: def tc_388f9897 : InstrItinClass;
    53: def tc_38e0bae9 : InstrItinClass;
    54: def tc_3d14a17b : InstrItinClass;
    55: def tc_3edca78f : InstrItinClass;
    56: def tc_3fbf1042 : InstrItinClass;
    57: def tc_407e96f9 : InstrItinClass;
    58: def tc_40d64c94 : InstrItinClass;
    59: def tc_4222e6bf : InstrItinClass;
    60: def tc_42ff66ba : InstrItinClass;
    61: def tc_442395f3 : InstrItinClass;
    62: def tc_449acf79 : InstrItinClass;
    63: def tc_44d5a428 : InstrItinClass;
    64: def tc_44fffc58 : InstrItinClass;
    65: def tc_45791fb8 : InstrItinClass;
    66: def tc_45f9d1be : InstrItinClass;
    67: def tc_46c18ecf : InstrItinClass;
    68: def tc_49fdfd4b : InstrItinClass;
    69: def tc_4a55d03c : InstrItinClass;
    70: def tc_4abdbdc6 : InstrItinClass;
    71: def tc_4ac61d92 : InstrItinClass;
    72: def tc_4bf903b0 : InstrItinClass;
    73: def tc_503ce0f3 : InstrItinClass;
    74: def tc_512b1653 : InstrItinClass;
    75: def tc_53c851ab : InstrItinClass;
    76: def tc_54f0cee2 : InstrItinClass;
    77: def tc_5502c366 : InstrItinClass;
    78: def tc_55255f2b : InstrItinClass;
    79: def tc_556f6577 : InstrItinClass;
    80: def tc_55a9a350 : InstrItinClass;
    81: def tc_55b33fda : InstrItinClass;
    82: def tc_56a124a7 : InstrItinClass;
    83: def tc_57a55b54 : InstrItinClass;
    84: def tc_5944960d : InstrItinClass;
    85: def tc_59a7822c : InstrItinClass;
    86: def tc_5a222e89 : InstrItinClass;
    87: def tc_5a4b5e58 : InstrItinClass;
    88: def tc_5b347363 : InstrItinClass;
    89: def tc_5ceb2f9e : InstrItinClass;
    90: def tc_5da50c4b : InstrItinClass;
    91: def tc_5deb5e47 : InstrItinClass;
    92: def tc_5e4cf0e8 : InstrItinClass;
    93: def tc_5f2afaf7 : InstrItinClass;
    94: def tc_60e324ff : InstrItinClass;
    95: def tc_63567288 : InstrItinClass;
    96: def tc_64b00d8a : InstrItinClass;
    97: def tc_651cbe02 : InstrItinClass;
    98: def tc_65279839 : InstrItinClass;
    99: def tc_65cbd974 : InstrItinClass;
   100: def tc_69bfb303 : InstrItinClass;
   101: def tc_6aa823ab : InstrItinClass;
   102: def tc_6ae3426b : InstrItinClass;
   103: def tc_6d861a95 : InstrItinClass;
   104: def tc_6e20402a : InstrItinClass;
   105: def tc_6f42bc60 : InstrItinClass;
   106: def tc_6fb52018 : InstrItinClass;
   107: def tc_6fc5dbea : InstrItinClass;
   108: def tc_711c805f : InstrItinClass;
   109: def tc_713b66bf : InstrItinClass;
   110: def tc_7401744f : InstrItinClass;
   111: def tc_7476d766 : InstrItinClass;
   112: def tc_74a42bda : InstrItinClass;
   113: def tc_759e57be : InstrItinClass;
   114: def tc_76bb5435 : InstrItinClass;
   115: def tc_77f94a5e : InstrItinClass;
   116: def tc_788b1d09 : InstrItinClass;
   117: def tc_78f87ed3 : InstrItinClass;
   118: def tc_7af3a37e : InstrItinClass;
   119: def tc_7b9187d3 : InstrItinClass;
   120: def tc_7c28bd7e : InstrItinClass;
   121: def tc_7c31e19a : InstrItinClass;
   122: def tc_7c6d32e4 : InstrItinClass;
   123: def tc_7d6a2568 : InstrItinClass;
   124: def tc_7dc63b5c : InstrItinClass;
   125: def tc_7f58404a : InstrItinClass;
   126: def tc_7f7f45f5 : InstrItinClass;
   127: def tc_7f8ae742 : InstrItinClass;
   128: def tc_8035e91f : InstrItinClass;
   129: def tc_822c3c68 : InstrItinClass;
   130: def tc_829d8a86 : InstrItinClass;
   131: def tc_838c4d7a : InstrItinClass;
   132: def tc_84a7500d : InstrItinClass;
   133: def tc_86173609 : InstrItinClass;
   134: def tc_887d1bb7 : InstrItinClass;
   135: def tc_8a6d0d94 : InstrItinClass;
   136: def tc_8a825db2 : InstrItinClass;
   137: def tc_8b5bd4f5 : InstrItinClass;
   138: def tc_8e82e8ca : InstrItinClass;
   139: def tc_8f36a2fd : InstrItinClass;
   140: def tc_9124c04f : InstrItinClass;
   141: def tc_92240447 : InstrItinClass;
   142: def tc_934753bb : InstrItinClass;
   143: def tc_937dd41c : InstrItinClass;
   144: def tc_9406230a : InstrItinClass;
   145: def tc_95a33176 : InstrItinClass;
   146: def tc_95f43c5e : InstrItinClass;
   147: def tc_96ef76ef : InstrItinClass;
   148: def tc_975a4e54 : InstrItinClass;
   149: def tc_9783714b : InstrItinClass;
   150: def tc_9b20a062 : InstrItinClass;
   151: def tc_9b34f5e0 : InstrItinClass;
   152: def tc_9b3c0462 : InstrItinClass;
   153: def tc_9bcfb2ee : InstrItinClass;
   154: def tc_9c52f549 : InstrItinClass;
   155: def tc_9e27f2f9 : InstrItinClass;
   156: def tc_9e72dc89 : InstrItinClass;
   157: def tc_9edb7c77 : InstrItinClass;
   158: def tc_9edefe01 : InstrItinClass;
   159: def tc_9f6cd987 : InstrItinClass;
   160: def tc_a08b630b : InstrItinClass;
   161: def tc_a1297125 : InstrItinClass;
   162: def tc_a154b476 : InstrItinClass;
   163: def tc_a2b365d2 : InstrItinClass;
   164: def tc_a3070909 : InstrItinClass;
   165: def tc_a32e03e7 : InstrItinClass;
   166: def tc_a38c45dc : InstrItinClass;
   167: def tc_a4e22bbd : InstrItinClass;
   168: def tc_a4ee89db : InstrItinClass;
   169: def tc_a724463d : InstrItinClass;
   170: def tc_a7a13fac : InstrItinClass;
   171: def tc_a7bdb22c : InstrItinClass;
   172: def tc_a9edeffa : InstrItinClass;
   173: def tc_abfd9a6d : InstrItinClass;
   174: def tc_ac65613f : InstrItinClass;
   175: def tc_addc37a8 : InstrItinClass;
   176: def tc_ae5babd7 : InstrItinClass;
   177: def tc_aee6250c : InstrItinClass;
   178: def tc_af6af259 : InstrItinClass;
   179: def tc_b1ae5f67 : InstrItinClass;
   180: def tc_b2196a3f : InstrItinClass;
   181: def tc_b3d46584 : InstrItinClass;
   182: def tc_b4dc7630 : InstrItinClass;
   183: def tc_b7c4062a : InstrItinClass;
   184: def tc_b837298f : InstrItinClass;
   185: def tc_b9bec29e : InstrItinClass;
   186: def tc_ba9255a6 : InstrItinClass;
   187: def tc_bb07f2c5 : InstrItinClass;
   188: def tc_bb78483e : InstrItinClass;
   189: def tc_bb831a7c : InstrItinClass;
   190: def tc_bf2ffc0f : InstrItinClass;
   191: def tc_c20701f0 : InstrItinClass;
   192: def tc_c21d7447 : InstrItinClass;
   193: def tc_c57d9f39 : InstrItinClass;
   194: def tc_c818ff7f : InstrItinClass;
   195: def tc_ce59038e : InstrItinClass;
   196: def tc_cfa0e29b : InstrItinClass;
   197: def tc_d03278fd : InstrItinClass;
   198: def tc_d234b61a : InstrItinClass;
   199: def tc_d33e5eee : InstrItinClass;
   200: def tc_d3632d88 : InstrItinClass;
   201: def tc_d45ba9cd : InstrItinClass;
   202: def tc_d57d649c : InstrItinClass;
   203: def tc_d61dfdc3 : InstrItinClass;
   204: def tc_d68dca5c : InstrItinClass;
   205: def tc_d71ea8fa : InstrItinClass;
   206: def tc_d7718fbe : InstrItinClass;
   207: def tc_db596beb : InstrItinClass;
   208: def tc_db96aa6b : InstrItinClass;
   209: def tc_dc51281d : InstrItinClass;
   210: def tc_decdde8a : InstrItinClass;
   211: def tc_df5d53f9 : InstrItinClass;
   212: def tc_e3d699e3 : InstrItinClass;
   213: def tc_e60def48 : InstrItinClass;
   214: def tc_e9170fb7 : InstrItinClass;
   215: def tc_ed03645c : InstrItinClass;
   216: def tc_ed3f8d2a : InstrItinClass;
   217: def tc_eed07714 : InstrItinClass;
   218: def tc_eeda4109 : InstrItinClass;
   219: def tc_ef921005 : InstrItinClass;
   220: def tc_f098b237 : InstrItinClass;
   221: def tc_f0cdeccf : InstrItinClass;
   222: def tc_f0e8e832 : InstrItinClass;
   223: def tc_f34c1c21 : InstrItinClass;
   224: def tc_f38f92e1 : InstrItinClass;
   225: def tc_f529831b : InstrItinClass;
   226: def tc_f6e2aff9 : InstrItinClass;
   227: def tc_f7569068 : InstrItinClass;
   228: def tc_f97707c1 : InstrItinClass;
   229: def tc_f999c66e : InstrItinClass;
   230: def tc_fae9dfa5 : InstrItinClass;
   231: def tc_fedb7e19 : InstrItinClass;
   232: 
   233: class DepScalarItinV5 {
   234:   list<InstrItinData> DepScalarItinV5_list = [
   235:     InstrItinData <tc_011e0e9d, [InstrStage<1, [SLOT0]>]>,
   236:     InstrItinData <tc_01d44cb2, [InstrStage<1, [SLOT2, SLOT3]>]>,
   237:     InstrItinData <tc_01e1be3b, [InstrStage<1, [SLOT2, SLOT3]>]>,
   238:     InstrItinData <tc_02fe1c65, [InstrStage<1, [SLOT2, SLOT3]>]>,
   239:     InstrItinData <tc_0655b949, [InstrStage<1, [SLOT0, SLOT1]>]>,
   240:     InstrItinData <tc_075c8dd8, [InstrStage<1, [SLOT0, SLOT1]>]>,
   241:     InstrItinData <tc_0a195f2c, [InstrStage<1, [SLOT2, SLOT3]>]>,
   242:     InstrItinData <tc_0a6c20ae, [InstrStage<1, [SLOT0]>]>,
   243:     InstrItinData <tc_0ba0d5da, [InstrStage<1, [SLOT2]>]>,
   244:     InstrItinData <tc_0dfac0a7, [InstrStage<1, [SLOT2, SLOT3]>]>,
   245:     InstrItinData <tc_0fac1eb8, [InstrStage<1, [SLOT0]>]>,
   246:     InstrItinData <tc_112d30d6, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   247:     InstrItinData <tc_1242dc2a, [InstrStage<1, [SLOT0]>]>,
   248:     InstrItinData <tc_1248597c, [InstrStage<1, [SLOT3]>]>,
   249:     InstrItinData <tc_14ab4f41, [InstrStage<1, [SLOT0]>]>,
   250:     InstrItinData <tc_151bf368, [InstrStage<1, [SLOT2, SLOT3]>]>,
   251:     InstrItinData <tc_158aa3f7, [InstrStage<1, [SLOT0]>]>,
   252:     InstrItinData <tc_197dce51, [InstrStage<1, [SLOT3]>]>,
   253:     InstrItinData <tc_1981450d, [InstrStage<1, [SLOT0]>]>,
   254:     InstrItinData <tc_1c2c7a4a, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   255:     InstrItinData <tc_1c7522a8, [InstrStage<1, [SLOT0, SLOT1]>]>,
   256:     InstrItinData <tc_1d41f8b7, [InstrStage<1, [SLOT2, SLOT3]>]>,
   257:     InstrItinData <tc_1fcb8495, [InstrStage<1, [SLOT2, SLOT3]>]>,
   258:     InstrItinData <tc_1fe4ab69, [InstrStage<1, [SLOT0, SLOT1]>]>,
   259:     InstrItinData <tc_20131976, [InstrStage<1, [SLOT2, SLOT3]>]>,
   260:     InstrItinData <tc_2237d952, [InstrStage<1, [SLOT0]>]>,
   261:     InstrItinData <tc_23708a21, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   262:     InstrItinData <tc_2471c1c8, [InstrStage<1, [SLOT0]>]>,
   263:     InstrItinData <tc_24e109c7, [InstrStage<1, [SLOT0]>]>,
   264:     InstrItinData <tc_24f426ab, [InstrStage<1, [SLOT2, SLOT3]>]>,
   265:     InstrItinData <tc_27106296, [InstrStage<1, [SLOT3]>]>,
   266:     InstrItinData <tc_280f7fe1, [InstrStage<1, [SLOT0, SLOT1]>]>,
   267:     InstrItinData <tc_28e55c6f, [InstrStage<1, [SLOT3]>]>,
   268:     InstrItinData <tc_2c13e7f5, [InstrStage<1, [SLOT2, SLOT3]>]>,
   269:     InstrItinData <tc_2c3e17fc, [InstrStage<1, [SLOT3]>]>,
   270:     InstrItinData <tc_2f573607, [InstrStage<1, [SLOT2]>]>,
   271:     InstrItinData <tc_33e7e673, [InstrStage<1, [SLOT2]>]>,
   272:     InstrItinData <tc_362b0be2, [InstrStage<1, [SLOT2]>]>,
   273:     InstrItinData <tc_38382228, [InstrStage<1, [SLOT2, SLOT3]>]>,
   274:     InstrItinData <tc_388f9897, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   275:     InstrItinData <tc_38e0bae9, [InstrStage<1, [SLOT2, SLOT3]>]>,
   276:     InstrItinData <tc_3d14a17b, [InstrStage<1, [SLOT0, SLOT1]>]>,
   277:     InstrItinData <tc_3edca78f, [InstrStage<1, [SLOT3]>]>,
   278:     InstrItinData <tc_3fbf1042, [InstrStage<1, [SLOT0, SLOT1]>]>,
   279:     InstrItinData <tc_407e96f9, [InstrStage<1, [SLOT2, SLOT3]>]>,
   280:     InstrItinData <tc_40d64c94, [InstrStage<1, [SLOT0]>]>,
   281:     InstrItinData <tc_4222e6bf, [InstrStage<1, [SLOT0, SLOT1]>]>,
   282:     InstrItinData <tc_42ff66ba, [InstrStage<1, [SLOT2]>]>,
   283:     InstrItinData <tc_442395f3, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   284:     InstrItinData <tc_449acf79, [InstrStage<1, [SLOT0, SLOT1]>]>,
   285:     InstrItinData <tc_44d5a428, [InstrStage<1, [SLOT0, SLOT1]>]>,
   286:     InstrItinData <tc_44fffc58, [InstrStage<1, [SLOT2, SLOT3]>]>,
   287:     InstrItinData <tc_45791fb8, [InstrStage<1, [SLOT0, SLOT1]>]>,
   288:     InstrItinData <tc_45f9d1be, [InstrStage<1, [SLOT2]>]>,
   289:     InstrItinData <tc_49fdfd4b, [InstrStage<1, [SLOT3]>]>,
   290:     InstrItinData <tc_4a55d03c, [InstrStage<1, [SLOT2, SLOT3]>]>,
   291:     InstrItinData <tc_4abdbdc6, [InstrStage<1, [SLOT3]>]>,
   292:     InstrItinData <tc_4ac61d92, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   293:     InstrItinData <tc_4bf903b0, [InstrStage<1, [SLOT0]>]>,
   294:     InstrItinData <tc_503ce0f3, [InstrStage<1, [SLOT2, SLOT3]>]>,
   295:     InstrItinData <tc_53c851ab, [InstrStage<1, [SLOT2]>]>,
   296:     InstrItinData <tc_5502c366, [InstrStage<1, [SLOT2, SLOT3]>]>,
   297:     InstrItinData <tc_55255f2b, [InstrStage<1, [SLOT3]>]>,
   298:     InstrItinData <tc_556f6577, [InstrStage<1, [SLOT2, SLOT3]>]>,
   299:     InstrItinData <tc_55a9a350, [InstrStage<1, [SLOT0]>]>,
   300:     InstrItinData <tc_55b33fda, [InstrStage<1, [SLOT2, SLOT3]>]>,
   301:     InstrItinData <tc_56a124a7, [InstrStage<1, [SLOT2, SLOT3]>]>,
   302:     InstrItinData <tc_57a55b54, [InstrStage<1, [SLOT3]>]>,
   303:     InstrItinData <tc_5944960d, [InstrStage<1, [SLOT0, SLOT1]>]>,
   304:     InstrItinData <tc_59a7822c, [InstrStage<1, [SLOT0, SLOT1]>]>,
   305:     InstrItinData <tc_5a4b5e58, [InstrStage<1, [SLOT3]>]>,
   306:     InstrItinData <tc_5b347363, [InstrStage<1, [SLOT0, SLOT1]>]>,
   307:     InstrItinData <tc_5ceb2f9e, [InstrStage<1, [SLOT0, SLOT1]>]>,
   308:     InstrItinData <tc_5da50c4b, [InstrStage<1, [SLOT2, SLOT3]>]>,
   309:     InstrItinData <tc_5deb5e47, [InstrStage<1, [SLOT0]>]>,
   310:     InstrItinData <tc_5e4cf0e8, [InstrStage<1, [SLOT2, SLOT3]>]>,
   311:     InstrItinData <tc_5f2afaf7, [InstrStage<1, [SLOT0, SLOT1]>]>,
   312:     InstrItinData <tc_60e324ff, [InstrStage<1, [SLOT2]>]>,
   313:     InstrItinData <tc_63567288, [InstrStage<1, [SLOT0, SLOT1]>]>,
   314:     InstrItinData <tc_64b00d8a, [InstrStage<1, [SLOT0]>]>,
   315:     InstrItinData <tc_651cbe02, [InstrStage<1, [SLOT2, SLOT3]>]>,
   316:     InstrItinData <tc_65279839, [InstrStage<1, [SLOT2, SLOT3]>]>,
   317:     InstrItinData <tc_65cbd974, [InstrStage<1, [SLOT0, SLOT1]>]>,
   318:     InstrItinData <tc_69bfb303, [InstrStage<1, [SLOT2, SLOT3]>]>,
   319:     InstrItinData <tc_6ae3426b, [InstrStage<1, [SLOT3]>]>,
   320:     InstrItinData <tc_6d861a95, [InstrStage<1, [SLOT3]>]>,
   321:     InstrItinData <tc_6e20402a, [InstrStage<1, [SLOT0]>]>,
   322:     InstrItinData <tc_6f42bc60, [InstrStage<1, [SLOT0]>]>,
   323:     InstrItinData <tc_6fc5dbea, [InstrStage<1, [SLOT2, SLOT3]>]>,
   324:     InstrItinData <tc_711c805f, [InstrStage<1, [SLOT2, SLOT3]>]>,
   325:     InstrItinData <tc_713b66bf, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   326:     InstrItinData <tc_7401744f, [InstrStage<1, [SLOT2, SLOT3]>]>,
   327:     InstrItinData <tc_7476d766, [InstrStage<1, [SLOT3]>]>,
   328:     InstrItinData <tc_74a42bda, [InstrStage<1, [SLOT0, SLOT1]>]>,
   329:     InstrItinData <tc_76bb5435, [InstrStage<1, [SLOT0, SLOT1]>]>,
   330:     InstrItinData <tc_77f94a5e, [InstrStage<1, [SLOT0]>]>,
   331:     InstrItinData <tc_788b1d09, [InstrStage<1, [SLOT2, SLOT3]>]>,
   332:     InstrItinData <tc_7af3a37e, [InstrStage<1, [SLOT0]>]>,
   333:     InstrItinData <tc_7b9187d3, [InstrStage<1, [SLOT0]>]>,
   334:     InstrItinData <tc_7c31e19a, [InstrStage<1, [SLOT0, SLOT1]>]>,
   335:     InstrItinData <tc_7c6d32e4, [InstrStage<1, [SLOT0, SLOT1]>]>,
   336:     InstrItinData <tc_7dc63b5c, [InstrStage<1, [SLOT3]>]>,
   337:     InstrItinData <tc_7f7f45f5, [InstrStage<1, [SLOT2, SLOT3]>]>,
   338:     InstrItinData <tc_7f8ae742, [InstrStage<1, [SLOT2, SLOT3]>]>,
   339:     InstrItinData <tc_8035e91f, [InstrStage<1, [SLOT0, SLOT1]>]>,
   340:     InstrItinData <tc_822c3c68, [InstrStage<1, [SLOT0, SLOT1]>]>,
   341:     InstrItinData <tc_829d8a86, [InstrStage<1, [SLOT0]>]>,
   342:     InstrItinData <tc_838c4d7a, [InstrStage<1, [SLOT0, SLOT1]>]>,
   343:     InstrItinData <tc_84a7500d, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   344:     InstrItinData <tc_86173609, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   345:     InstrItinData <tc_887d1bb7, [InstrStage<1, [SLOT0, SLOT1]>]>,
   346:     InstrItinData <tc_8a6d0d94, [InstrStage<1, [SLOT0, SLOT1]>]>,
   347:     InstrItinData <tc_8a825db2, [InstrStage<1, [SLOT2, SLOT3]>]>,
   348:     InstrItinData <tc_8b5bd4f5, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   349:     InstrItinData <tc_8e82e8ca, [InstrStage<1, [SLOT0, SLOT1]>]>,
   350:     InstrItinData <tc_8f36a2fd, [InstrStage<1, [SLOT0, SLOT1]>]>,
   351:     InstrItinData <tc_9124c04f, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   352:     InstrItinData <tc_92240447, [InstrStage<1, [SLOT0]>]>,
   353:     InstrItinData <tc_934753bb, [InstrStage<1, [SLOT0]>]>,
   354:     InstrItinData <tc_937dd41c, [InstrStage<1, [SLOT0, SLOT1]>]>,
   355:     InstrItinData <tc_9406230a, [InstrStage<1, [SLOT3]>]>,
   356:     InstrItinData <tc_95a33176, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   357:     InstrItinData <tc_95f43c5e, [InstrStage<1, [SLOT2]>]>,
   358:     InstrItinData <tc_96ef76ef, [InstrStage<1, [SLOT0]>]>,
   359:     InstrItinData <tc_975a4e54, [InstrStage<1, [SLOT0]>]>,
   360:     InstrItinData <tc_9783714b, [InstrStage<1, [SLOT2, SLOT3]>]>,
   361:     InstrItinData <tc_9b34f5e0, [InstrStage<1, [SLOT2]>]>,
   362:     InstrItinData <tc_9b3c0462, [InstrStage<1, [SLOT2, SLOT3]>]>,
   363:     InstrItinData <tc_9bcfb2ee, [InstrStage<1, [SLOT0]>]>,
   364:     InstrItinData <tc_9c52f549, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   365:     InstrItinData <tc_9e27f2f9, [InstrStage<1, [SLOT2, SLOT3]>]>,
   366:     InstrItinData <tc_9e72dc89, [InstrStage<1, [SLOT2, SLOT3]>]>,
   367:     InstrItinData <tc_9edb7c77, [InstrStage<1, [SLOT2, SLOT3]>]>,
   368:     InstrItinData <tc_9edefe01, [InstrStage<1, [SLOT0, SLOT1]>]>,
   369:     InstrItinData <tc_9f6cd987, [InstrStage<1, [SLOT2, SLOT3]>]>,
   370:     InstrItinData <tc_a08b630b, [InstrStage<1, [SLOT2, SLOT3]>]>,
   371:     InstrItinData <tc_a1297125, [InstrStage<1, [SLOT2, SLOT3]>]>,
   372:     InstrItinData <tc_a154b476, [InstrStage<1, [SLOT2, SLOT3]>]>,
   373:     InstrItinData <tc_a2b365d2, [InstrStage<1, [SLOT0, SLOT1]>]>,
   374:     InstrItinData <tc_a3070909, [InstrStage<1, [SLOT0]>]>,
   375:     InstrItinData <tc_a32e03e7, [InstrStage<1, [SLOT0, SLOT1]>]>,
   376:     InstrItinData <tc_a38c45dc, [InstrStage<1, [SLOT2, SLOT3]>]>,
   377:     InstrItinData <tc_a4e22bbd, [InstrStage<1, [SLOT2, SLOT3]>]>,
   378:     InstrItinData <tc_a4ee89db, [InstrStage<1, [SLOT0]>]>,
   379:     InstrItinData <tc_a7a13fac, [InstrStage<1, [SLOT2, SLOT3]>]>,
   380:     InstrItinData <tc_a7bdb22c, [InstrStage<1, [SLOT2, SLOT3]>]>,
   381:     InstrItinData <tc_a9edeffa, [InstrStage<1, [SLOT0, SLOT1]>]>,
   382:     InstrItinData <tc_abfd9a6d, [InstrStage<1, [SLOT0, SLOT1]>]>,
   383:     InstrItinData <tc_ac65613f, [InstrStage<1, [SLOT0, SLOT1]>]>,
   384:     InstrItinData <tc_addc37a8, [InstrStage<1, [SLOT0]>]>,
   385:     InstrItinData <tc_ae5babd7, [InstrStage<1, [SLOT0, SLOT1]>]>,
   386:     InstrItinData <tc_aee6250c, [InstrStage<1, [SLOT0, SLOT1]>]>,
   387:     InstrItinData <tc_af6af259, [InstrStage<1, [SLOT0, SLOT1]>]>,
   388:     InstrItinData <tc_b1ae5f67, [InstrStage<1, [SLOT0]>]>,
   389:     InstrItinData <tc_b4dc7630, [InstrStage<1, [SLOT0, SLOT1]>]>,
   390:     InstrItinData <tc_b7c4062a, [InstrStage<1, [SLOT0, SLOT1]>]>,
   391:     InstrItinData <tc_b837298f, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   392:     InstrItinData <tc_ba9255a6, [InstrStage<1, [SLOT0, SLOT1]>]>,
   393:     InstrItinData <tc_bb07f2c5, [InstrStage<1, [SLOT0, SLOT1]>]>,
   394:     InstrItinData <tc_bb831a7c, [InstrStage<1, [SLOT2, SLOT3]>]>,
   395:     InstrItinData <tc_bf2ffc0f, [InstrStage<1, [SLOT0, SLOT1]>]>,
   396:     InstrItinData <tc_c20701f0, [InstrStage<1, [SLOT2, SLOT3]>]>,
   397:     InstrItinData <tc_c21d7447, [InstrStage<1, [SLOT2, SLOT3]>]>,
   398:     InstrItinData <tc_c57d9f39, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   399:     InstrItinData <tc_c818ff7f, [InstrStage<1, [SLOT0]>]>,
   400:     InstrItinData <tc_ce59038e, [InstrStage<1, [SLOT0]>]>,
   401:     InstrItinData <tc_cfa0e29b, [InstrStage<1, [SLOT0]>]>,
   402:     InstrItinData <tc_d03278fd, [InstrStage<1, [SLOT0, SLOT1]>]>,
   403:     InstrItinData <tc_d33e5eee, [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
   404:     InstrItinData <tc_d3632d88, [InstrStage<1, [SLOT2, SLOT3]>]>,
   405:     InstrItinData <tc_d45ba9cd, [InstrStage<1, [SLOT0]>]>,
   406:     InstrItinData <tc_d57d649c, [InstrStage<1, [SLOT2]>]>,
   407:     InstrItinData <tc_d61dfdc3, [InstrStage<1, [SLOT2, SLOT3]>]>,
   408:     InstrItinData <tc_d68dca5c, [InstrStage<1, [SLOT2, SLOT3]>]>,
   409:     InstrItinData <tc_d7718fbe, [InstrStage<1, [SLOT3]>]>,
   410:     InstrItinData <tc_db596beb, [InstrStage<1, [SLOT2, SLOT3]>]>,
   411:     InstrItinData <tc_db96aa6b, [InstrStage<1, [SLOT0]>]>,
   412:     InstrItinData <tc_dc51281d, [InstrStage<1, [SLOT2]>]>,
   413:     InstrItinData <tc_decdde8a, [InstrStage<1, [SLOT2, SLOT3]>]>,
   414:     InstrItinData <tc_df5d53f9, [InstrStage<1, [SLOT0]>]>,
   415:     InstrItinData <tc_e3d699e3, [InstrStage<1, [SLOT2, SLOT3]>]>,
   416:     InstrItinData <tc_e60def48, [InstrStage<1, [SLOT2]>]>,
   417:     InstrItinData <tc_e9170fb7, [InstrStage<1, [SLOT0, SLOT1]>]>,
   418:     InstrItinData <tc_ed03645c, [InstrStage<1, [SLOT2]>]>,
   419:     InstrItinData <tc_eed07714, [InstrStage<1, [SLOT0, SLOT1]>]>,
   420:     InstrItinData <tc_eeda4109, [InstrStage<1, [SLOT2, SLOT3]>]>,
   421:     InstrItinData <tc_ef921005, [InstrStage<1, [SLOT2, SLOT3]>]>,
   422:     InstrItinData <tc_f098b237, [InstrStage<1, [SLOT2, SLOT3]>]>,
   423:     InstrItinData <tc_f0cdeccf, [InstrStage<1, [SLOT2, SLOT3]>]>,
   424:     InstrItinData <tc_f0e8e832, [InstrStage<1, [SLOT2, SLOT3]>]>,
   425:     InstrItinData <tc_f34c1c21, [InstrStage<1, [SLOT2, SLOT3]>]>,
   426:     InstrItinData <tc_f38f92e1, [InstrStage<1, [SLOT0]>]>,
   427:     InstrItinData <tc_f529831b, [InstrStage<1, [SLOT0]>]>,
   428:     InstrItinData <tc_f6e2aff9, [InstrStage<1, [SLOT0]>]>,
   429:     InstrItinData <tc_f7569068, [InstrStage<1, [SLOT2, SLOT3]>]>,
   430:     InstrItinData <tc_f97707c1, [InstrStage<1, [SLOT2]>]>,
   431:     InstrItinData <tc_f999c66e, [InstrStage<1, [SLOT2, SLOT3]>]>,
   432:     InstrItinData <tc_fae9dfa5, [InstrStage<1, [SLOT3]>]>,
   433:     InstrItinData <tc_fedb7e19, [InstrStage<1, [SLOT0, SLOT1]>]>  ];
   434: }
   435: 
   436: class DepScalarItinV55 {
   437:   list<InstrItinData> DepScalarItinV55_list = [
   438:     InstrItinData <tc_011e0e9d, /*tc_st*/
   439:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
   440:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   441: 
   442:     InstrItinData <tc_01d44cb2, /*tc_1*/
   443:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
   444:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   445: 
   446:     InstrItinData <tc_01e1be3b, /*tc_3x*/
   447:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
   448:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   449: 
   450:     InstrItinData <tc_02fe1c65, /*tc_3x*/
   451:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
   452:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   453: 
   454:     InstrItinData <tc_0655b949, /*tc_st*/
   455:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
   456:       [Hex_FWD, Hex_FWD]>,
   457: 
   458:     InstrItinData <tc_075c8dd8, /*tc_ld*/
   459:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
   460:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   461: 
   462:     InstrItinData <tc_0a195f2c, /*tc_4x*/
   463:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
   464:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   465: 
   466:     InstrItinData <tc_0a6c20ae, /*tc_st*/
   467:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
   468:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   469: 
   470:     InstrItinData <tc_0ba0d5da, /*tc_2early*/
   471:       [InstrStage<1, [SLOT2]>], [1],
   472:       [Hex_FWD]>,
   473: 
   474:     InstrItinData <tc_0dfac0a7, /*tc_2*/
   475:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
   476:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   477: 
   478:     InstrItinData <tc_0fac1eb8, /*tc_st*/
   479:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
   480:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   481: 
   482:     InstrItinData <tc_112d30d6, /*tc_1*/
   483:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
   484:       [Hex_FWD]>,
   485: 
   486:     InstrItinData <tc_1242dc2a, /*tc_ld*/
   487:       [InstrStage<1, [SLOT0]>], [2],
   488:       [Hex_FWD]>,
   489: 
   490:     InstrItinData <tc_1248597c, /*tc_3x*/
   491:       [InstrStage<1, [SLOT3]>], [2, 2],
   492:       [Hex_FWD, Hex_FWD]>,
   493: 
   494:     InstrItinData <tc_139ef484, /*tc_3stall*/
   495:       [InstrStage<1, [SLOT2]>], [1, 1],
   496:       [Hex_FWD, Hex_FWD]>,
   497: 
   498:     InstrItinData <tc_14ab4f41, /*tc_3stall*/
   499:       [InstrStage<1, [SLOT0]>], [4, 3, 1],
   500:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV5, DepScalarItinV55, which carry the state or API of this component. It defines generated/declarative TableGen records like tc_011e0e9d, tc_01d44cb2, tc_01e1be3b, tc_02fe1c65, tc_0655b949, ... (223 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 DepScalarItinV5, DepScalarItinV55 等类型，用来承载该组件的状态或接口。 这里定义了 tc_011e0e9d, tc_01d44cb2, tc_01e1be3b, tc_02fe1c65, tc_0655b949, ... (223 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 501-1000 / 第 501-1000 行

```tablegen
   501: 
   502:     InstrItinData <tc_151bf368, /*tc_2early*/
   503:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
   504:       [Hex_FWD, Hex_FWD]>,
   505: 
   506:     InstrItinData <tc_158aa3f7, /*tc_st*/
   507:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
   508:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   509: 
   510:     InstrItinData <tc_197dce51, /*tc_3x*/
   511:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
   512:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   513: 
   514:     InstrItinData <tc_1981450d, /*tc_3stall*/
   515:       [InstrStage<1, [SLOT0]>], [3],
   516:       [Hex_FWD]>,
   517: 
   518:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
   519:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
   520:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   521: 
   522:     InstrItinData <tc_1c7522a8, /*tc_ld*/
   523:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
   524:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   525: 
   526:     InstrItinData <tc_1d41f8b7, /*tc_3stall*/
   527:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1, 1, 2],
   528:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   529: 
   530:     InstrItinData <tc_1fcb8495, /*tc_2*/
   531:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
   532:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   533: 
   534:     InstrItinData <tc_1fe4ab69, /*tc_st*/
   535:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 2],
   536:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   537: 
   538:     InstrItinData <tc_20131976, /*tc_2*/
   539:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
   540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   541: 
   542:     InstrItinData <tc_2237d952, /*tc_ld*/
   543:       [InstrStage<1, [SLOT0]>], [1, 2],
   544:       [Hex_FWD, Hex_FWD]>,
   545: 
   546:     InstrItinData <tc_23708a21, /*tc_2early*/
   547:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
   548:       []>,
   549: 
   550:     InstrItinData <tc_2471c1c8, /*tc_ld*/
   551:       [InstrStage<1, [SLOT0]>], [4, 1],
   552:       [Hex_FWD, Hex_FWD]>,
   553: 
   554:     InstrItinData <tc_24e109c7, /*tc_3stall*/
   555:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
   556:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   557: 
   558:     InstrItinData <tc_24f426ab, /*tc_2early*/
   559:       [InstrStage<1, [SLOT2, SLOT3]>], [1, 2, 2],
   560:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   561: 
   562:     InstrItinData <tc_27106296, /*tc_3x*/
   563:       [InstrStage<1, [SLOT3]>], [4, 2, 2],
   564:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   565: 
   566:     InstrItinData <tc_280f7fe1, /*tc_st*/
   567:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 2],
   568:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   569: 
   570:     InstrItinData <tc_28e55c6f, /*tc_3x*/
   571:       [InstrStage<1, [SLOT3]>], [1, 1],
   572:       [Hex_FWD, Hex_FWD]>,
   573: 
   574:     InstrItinData <tc_2c13e7f5, /*tc_2*/
   575:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
   576:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   577: 
   578:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
   579:       [InstrStage<1, [SLOT3]>], [1],
   580:       [Hex_FWD]>,
   581: 
   582:     InstrItinData <tc_2f573607, /*tc_2early*/
   583:       [InstrStage<1, [SLOT2]>], [2, 1],
   584:       [Hex_FWD, Hex_FWD]>,
   585: 
   586:     InstrItinData <tc_33e7e673, /*tc_2early*/
   587:       [InstrStage<1, [SLOT2]>], [],
   588:       []>,
   589: 
   590:     InstrItinData <tc_362b0be2, /*tc_2early*/
   591:       [InstrStage<1, [SLOT2]>], [1],
   592:       [Hex_FWD]>,
   593: 
   594:     InstrItinData <tc_38382228, /*tc_3x*/
   595:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
   596:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   597: 
   598:     InstrItinData <tc_388f9897, /*tc_1*/
   599:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
   600:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   601: 
   602:     InstrItinData <tc_38e0bae9, /*tc_3*/
   603:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1, 1, 1],
   604:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   605: 
   606:     InstrItinData <tc_3d14a17b, /*tc_1*/
   607:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
   608:       [Hex_FWD, Hex_FWD]>,
   609: 
   610:     InstrItinData <tc_3edca78f, /*tc_2*/
   611:       [InstrStage<1, [SLOT3]>], [4, 2],
   612:       [Hex_FWD, Hex_FWD]>,
   613: 
   614:     InstrItinData <tc_3fbf1042, /*tc_1*/
   615:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
   616:       [Hex_FWD]>,
   617: 
   618:     InstrItinData <tc_407e96f9, /*tc_1*/
   619:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
   620:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   621: 
   622:     InstrItinData <tc_40d64c94, /*tc_3stall*/
   623:       [InstrStage<1, [SLOT0]>], [4, 2],
   624:       [Hex_FWD, Hex_FWD]>,
   625: 
   626:     InstrItinData <tc_4222e6bf, /*tc_ld*/
   627:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
   628:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   629: 
   630:     InstrItinData <tc_42ff66ba, /*tc_2early*/
   631:       [InstrStage<1, [SLOT2]>], [2, 1],
   632:       [Hex_FWD, Hex_FWD]>,
   633: 
   634:     InstrItinData <tc_442395f3, /*tc_1*/
   635:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2, 2],
   636:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   637: 
   638:     InstrItinData <tc_449acf79, /*tc_st*/
   639:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 3, 1, 2, 2],
   640:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   641: 
   642:     InstrItinData <tc_44d5a428, /*tc_st*/
   643:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
   644:       [Hex_FWD, Hex_FWD]>,
   645: 
   646:     InstrItinData <tc_44fffc58, /*tc_2early*/
   647:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
   648:       [Hex_FWD]>,
   649: 
   650:     InstrItinData <tc_45791fb8, /*tc_ld*/
   651:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
   652:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   653: 
   654:     InstrItinData <tc_45f9d1be, /*tc_2early*/
   655:       [InstrStage<1, [SLOT2]>], [2],
   656:       [Hex_FWD]>,
   657: 
   658:     InstrItinData <tc_46c18ecf, /*tc_3x*/
   659:       [InstrStage<1, [SLOT3]>], [4, 1],
   660:       [Hex_FWD, Hex_FWD]>,
   661: 
   662:     InstrItinData <tc_49fdfd4b, /*tc_3x*/
   663:       [InstrStage<1, [SLOT3]>], [4, 1],
   664:       [Hex_FWD, Hex_FWD]>,
   665: 
   666:     InstrItinData <tc_4a55d03c, /*tc_2early*/
   667:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 1],
   668:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   669: 
   670:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
   671:       [InstrStage<1, [SLOT3]>], [2, 2],
   672:       [Hex_FWD, Hex_FWD]>,
   673: 
   674:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
   675:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
   676:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   677: 
   678:     InstrItinData <tc_4bf903b0, /*tc_st*/
   679:       [InstrStage<1, [SLOT0]>], [3],
   680:       [Hex_FWD]>,
   681: 
   682:     InstrItinData <tc_503ce0f3, /*tc_3x*/
   683:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
   684:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   685: 
   686:     InstrItinData <tc_512b1653, /*tc_st*/
   687:       [InstrStage<1, [SLOT0]>], [1, 2],
   688:       [Hex_FWD, Hex_FWD]>,
   689: 
   690:     InstrItinData <tc_53c851ab, /*tc_2early*/
   691:       [InstrStage<1, [SLOT2]>], [3, 2, 2],
   692:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   693: 
   694:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
   695:       [InstrStage<1, [SLOT3]>], [1],
   696:       [Hex_FWD]>,
   697: 
   698:     InstrItinData <tc_5502c366, /*tc_1*/
   699:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
   700:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   701: 
   702:     InstrItinData <tc_55255f2b, /*tc_3x*/
   703:       [InstrStage<1, [SLOT3]>], [],
   704:       []>,
   705: 
   706:     InstrItinData <tc_556f6577, /*tc_3x*/
   707:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
   708:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   709: 
   710:     InstrItinData <tc_55a9a350, /*tc_st*/
   711:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
   712:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   713: 
   714:     InstrItinData <tc_55b33fda, /*tc_2early*/
   715:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1],
   716:       [Hex_FWD, Hex_FWD]>,
   717: 
   718:     InstrItinData <tc_56a124a7, /*tc_2early*/
   719:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
   720:       [Hex_FWD, Hex_FWD]>,
   721: 
   722:     InstrItinData <tc_57a55b54, /*tc_2early*/
   723:       [InstrStage<1, [SLOT3]>], [1, 2],
   724:       [Hex_FWD, Hex_FWD]>,
   725: 
   726:     InstrItinData <tc_5944960d, /*tc_ld*/
   727:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
   728:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   729: 
   730:     InstrItinData <tc_59a7822c, /*tc_2early*/
   731:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
   732:       [Hex_FWD, Hex_FWD]>,
   733: 
   734:     InstrItinData <tc_5a222e89, /*tc_2early*/
   735:       [InstrStage<1, [SLOT2]>], [1, 1],
   736:       [Hex_FWD, Hex_FWD]>,
   737: 
   738:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
   739:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
   740:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   741: 
   742:     InstrItinData <tc_5b347363, /*tc_1*/
   743:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
   744:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   745: 
   746:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
   747:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
   748:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   749: 
   750:     InstrItinData <tc_5da50c4b, /*tc_1*/
   751:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
   752:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   753: 
   754:     InstrItinData <tc_5deb5e47, /*tc_st*/
   755:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
   756:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   757: 
   758:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
   759:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
   760:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   761: 
   762:     InstrItinData <tc_5f2afaf7, /*tc_ld*/
   763:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 3, 1, 2],
   764:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   765: 
   766:     InstrItinData <tc_60e324ff, /*tc_2early*/
   767:       [InstrStage<1, [SLOT2]>], [1],
   768:       [Hex_FWD]>,
   769: 
   770:     InstrItinData <tc_63567288, /*tc_1*/
   771:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
   772:       [Hex_FWD]>,
   773: 
   774:     InstrItinData <tc_64b00d8a, /*tc_ld*/
   775:       [InstrStage<1, [SLOT0]>], [4, 1],
   776:       [Hex_FWD, Hex_FWD]>,
   777: 
   778:     InstrItinData <tc_651cbe02, /*tc_2early*/
   779:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
   780:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   781: 
   782:     InstrItinData <tc_65279839, /*tc_3x*/
   783:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
   784:       [Hex_FWD, Hex_FWD]>,
   785: 
   786:     InstrItinData <tc_65cbd974, /*tc_st*/
   787:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
   788:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   789: 
   790:     InstrItinData <tc_69bfb303, /*tc_2early*/
   791:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
   792:       [Hex_FWD, Hex_FWD]>,
   793: 
   794:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
   795:       [InstrStage<1, [SLOT3]>], [4, 1],
   796:       [Hex_FWD, Hex_FWD]>,
   797: 
   798:     InstrItinData <tc_6ae3426b, /*tc_3x*/
   799:       [InstrStage<1, [SLOT3]>], [4, 1],
   800:       [Hex_FWD, Hex_FWD]>,
   801: 
   802:     InstrItinData <tc_6d861a95, /*tc_3x*/
   803:       [InstrStage<1, [SLOT3]>], [2, 1],
   804:       [Hex_FWD, Hex_FWD]>,
   805: 
   806:     InstrItinData <tc_6e20402a, /*tc_st*/
   807:       [InstrStage<1, [SLOT0]>], [2, 3],
   808:       [Hex_FWD, Hex_FWD]>,
   809: 
   810:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
   811:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
   812:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   813: 
   814:     InstrItinData <tc_6fb52018, /*tc_3stall*/
   815:       [InstrStage<1, [SLOT0]>], [1, 1],
   816:       [Hex_FWD, Hex_FWD]>,
   817: 
   818:     InstrItinData <tc_6fc5dbea, /*tc_1*/
   819:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
   820:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   821: 
   822:     InstrItinData <tc_711c805f, /*tc_2early*/
   823:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
   824:       [Hex_FWD, Hex_FWD]>,
   825: 
   826:     InstrItinData <tc_713b66bf, /*tc_1*/
   827:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
   828:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   829: 
   830:     InstrItinData <tc_7401744f, /*tc_2*/
   831:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
   832:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   833: 
   834:     InstrItinData <tc_7476d766, /*tc_3x*/
   835:       [InstrStage<1, [SLOT3]>], [4, 2],
   836:       [Hex_FWD, Hex_FWD]>,
   837: 
   838:     InstrItinData <tc_74a42bda, /*tc_ld*/
   839:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
   840:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   841: 
   842:     InstrItinData <tc_759e57be, /*tc_3stall*/
   843:       [InstrStage<1, [SLOT2]>], [4, 1],
   844:       [Hex_FWD, Hex_FWD]>,
   845: 
   846:     InstrItinData <tc_76bb5435, /*tc_ld*/
   847:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
   848:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   849: 
   850:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
   851:       [InstrStage<1, [SLOT2]>], [1],
   852:       [Hex_FWD]>,
   853: 
   854:     InstrItinData <tc_77f94a5e, /*tc_st*/
   855:       [InstrStage<1, [SLOT0]>], [],
   856:       []>,
   857: 
   858:     InstrItinData <tc_788b1d09, /*tc_3*/
   859:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 1],
   860:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   861: 
   862:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
   863:       [InstrStage<1, [SLOT0]>], [],
   864:       []>,
   865: 
   866:     InstrItinData <tc_7af3a37e, /*tc_st*/
   867:       [InstrStage<1, [SLOT0]>], [1, 3],
   868:       [Hex_FWD, Hex_FWD]>,
   869: 
   870:     InstrItinData <tc_7b9187d3, /*tc_3stall*/
   871:       [InstrStage<1, [SLOT0]>], [3, 2],
   872:       [Hex_FWD, Hex_FWD]>,
   873: 
   874:     InstrItinData <tc_7c28bd7e, /*tc_st*/
   875:       [InstrStage<1, [SLOT0]>], [3],
   876:       [Hex_FWD]>,
   877: 
   878:     InstrItinData <tc_7c31e19a, /*tc_st*/
   879:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
   880:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   881: 
   882:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
   883:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
   884:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   885: 
   886:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
   887:       [InstrStage<1, [SLOT3]>], [4, 2],
   888:       [Hex_FWD, Hex_FWD]>,
   889: 
   890:     InstrItinData <tc_7f58404a, /*tc_3stall*/
   891:       [InstrStage<1, [SLOT3]>], [],
   892:       []>,
   893: 
   894:     InstrItinData <tc_7f7f45f5, /*tc_3x*/
   895:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1],
   896:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   897: 
   898:     InstrItinData <tc_7f8ae742, /*tc_3x*/
   899:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
   900:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   901: 
   902:     InstrItinData <tc_8035e91f, /*tc_st*/
   903:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
   904:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   905: 
   906:     InstrItinData <tc_822c3c68, /*tc_ld*/
   907:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
   908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   909: 
   910:     InstrItinData <tc_829d8a86, /*tc_st*/
   911:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
   912:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   913: 
   914:     InstrItinData <tc_838c4d7a, /*tc_st*/
   915:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
   916:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   917: 
   918:     InstrItinData <tc_84a7500d, /*tc_2*/
   919:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
   920:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   921: 
   922:     InstrItinData <tc_86173609, /*tc_1*/
   923:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2],
   924:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   925: 
   926:     InstrItinData <tc_887d1bb7, /*tc_st*/
   927:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 2],
   928:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   929: 
   930:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
   931:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
   932:       [Hex_FWD, Hex_FWD]>,
   933: 
   934:     InstrItinData <tc_8a825db2, /*tc_2*/
   935:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
   936:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   937: 
   938:     InstrItinData <tc_8b5bd4f5, /*tc_1*/
   939:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
   940:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   941: 
   942:     InstrItinData <tc_8e82e8ca, /*tc_st*/
   943:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 2],
   944:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   945: 
   946:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
   947:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
   948:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   949: 
   950:     InstrItinData <tc_9124c04f, /*tc_1*/
   951:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
   952:       [Hex_FWD, Hex_FWD]>,
   953: 
   954:     InstrItinData <tc_92240447, /*tc_st*/
   955:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
   956:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   957: 
   958:     InstrItinData <tc_934753bb, /*tc_ld*/
   959:       [InstrStage<1, [SLOT0]>], [4, 2, 2],
   960:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   961: 
   962:     InstrItinData <tc_937dd41c, /*tc_ld*/
   963:       [InstrStage<1, [SLOT0, SLOT1]>], [],
   964:       []>,
   965: 
   966:     InstrItinData <tc_9406230a, /*tc_3x*/
   967:       [InstrStage<1, [SLOT3]>], [2, 1],
   968:       [Hex_FWD, Hex_FWD]>,
   969: 
   970:     InstrItinData <tc_95a33176, /*tc_2*/
   971:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
   972:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   973: 
   974:     InstrItinData <tc_95f43c5e, /*tc_3*/
   975:       [InstrStage<1, [SLOT2]>], [1],
   976:       [Hex_FWD]>,
   977: 
   978:     InstrItinData <tc_96ef76ef, /*tc_st*/
   979:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
   980:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
   981: 
   982:     InstrItinData <tc_975a4e54, /*tc_3stall*/
   983:       [InstrStage<1, [SLOT0]>], [1, 3, 2],
   984:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
   985: 
   986:     InstrItinData <tc_9783714b, /*tc_3x*/
   987:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1],
   988:       [Hex_FWD, Hex_FWD]>,
   989: 
   990:     InstrItinData <tc_9b20a062, /*tc_3stall*/
   991:       [InstrStage<1, [SLOT2]>], [4, 1],
   992:       [Hex_FWD, Hex_FWD]>,
   993: 
   994:     InstrItinData <tc_9b34f5e0, /*tc_2early*/
   995:       [InstrStage<1, [SLOT2]>], [],
   996:       []>,
   997: 
   998:     InstrItinData <tc_9b3c0462, /*tc_2*/
   999:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  1000:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1001-1500 / 第 1001-1500 行

```tablegen
  1001: 
  1002:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  1003:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  1004:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1005: 
  1006:     InstrItinData <tc_9c52f549, /*tc_2early*/
  1007:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 1],
  1008:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1009: 
  1010:     InstrItinData <tc_9e27f2f9, /*tc_2early*/
  1011:       [InstrStage<1, [SLOT2, SLOT3]>], [1, 1, 2],
  1012:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1013: 
  1014:     InstrItinData <tc_9e72dc89, /*tc_3*/
  1015:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 1],
  1016:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1017: 
  1018:     InstrItinData <tc_9edb7c77, /*tc_3*/
  1019:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 1, 2],
  1020:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1021: 
  1022:     InstrItinData <tc_9edefe01, /*tc_st*/
  1023:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 2],
  1024:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1025: 
  1026:     InstrItinData <tc_9f6cd987, /*tc_1*/
  1027:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  1028:       [Hex_FWD, Hex_FWD]>,
  1029: 
  1030:     InstrItinData <tc_a08b630b, /*tc_2*/
  1031:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  1032:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1033: 
  1034:     InstrItinData <tc_a1297125, /*tc_2early*/
  1035:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 2],
  1036:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1037: 
  1038:     InstrItinData <tc_a154b476, /*tc_3x*/
  1039:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  1040:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1041: 
  1042:     InstrItinData <tc_a2b365d2, /*tc_st*/
  1043:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  1044:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1045: 
  1046:     InstrItinData <tc_a3070909, /*tc_3stall*/
  1047:       [InstrStage<1, [SLOT0]>], [1, 1],
  1048:       [Hex_FWD, Hex_FWD]>,
  1049: 
  1050:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  1051:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  1052:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1053: 
  1054:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  1055:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  1056:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1057: 
  1058:     InstrItinData <tc_a4e22bbd, /*tc_1*/
  1059:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  1060:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1061: 
  1062:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  1063:       [InstrStage<1, [SLOT0]>], [],
  1064:       []>,
  1065: 
  1066:     InstrItinData <tc_a724463d, /*tc_3stall*/
  1067:       [InstrStage<1, [SLOT0]>], [4, 1],
  1068:       [Hex_FWD, Hex_FWD]>,
  1069: 
  1070:     InstrItinData <tc_a7a13fac, /*tc_2early*/
  1071:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  1072:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1073: 
  1074:     InstrItinData <tc_a7bdb22c, /*tc_1*/
  1075:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  1076:       [Hex_FWD, Hex_FWD]>,
  1077: 
  1078:     InstrItinData <tc_a9edeffa, /*tc_st*/
  1079:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  1080:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1081: 
  1082:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  1083:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  1084:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1085: 
  1086:     InstrItinData <tc_ac65613f, /*tc_ld*/
  1087:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  1088:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1089: 
  1090:     InstrItinData <tc_addc37a8, /*tc_st*/
  1091:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  1092:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1093: 
  1094:     InstrItinData <tc_ae5babd7, /*tc_st*/
  1095:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  1096:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1097: 
  1098:     InstrItinData <tc_aee6250c, /*tc_ld*/
  1099:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  1100:       [Hex_FWD, Hex_FWD]>,
  1101: 
  1102:     InstrItinData <tc_af6af259, /*tc_ld*/
  1103:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  1104:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1105: 
  1106:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  1107:       [InstrStage<1, [SLOT0]>], [1],
  1108:       [Hex_FWD]>,
  1109: 
  1110:     InstrItinData <tc_b4dc7630, /*tc_st*/
  1111:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 2],
  1112:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1113: 
  1114:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  1115:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  1116:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1117: 
  1118:     InstrItinData <tc_b837298f, /*tc_1*/
  1119:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  1120:       []>,
  1121: 
  1122:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  1123:       [InstrStage<1, [SLOT2]>], [],
  1124:       []>,
  1125: 
  1126:     InstrItinData <tc_ba9255a6, /*tc_st*/
  1127:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 2],
  1128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1129: 
  1130:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  1131:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  1132:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1133: 
  1134:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  1135:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  1136:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1137: 
  1138:     InstrItinData <tc_bb831a7c, /*tc_1*/
  1139:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2, 2],
  1140:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1141: 
  1142:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  1143:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  1144:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1145: 
  1146:     InstrItinData <tc_c20701f0, /*tc_3x*/
  1147:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  1148:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1149: 
  1150:     InstrItinData <tc_c21d7447, /*tc_3x*/
  1151:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  1152:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1153: 
  1154:     InstrItinData <tc_c57d9f39, /*tc_1*/
  1155:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  1156:       [Hex_FWD, Hex_FWD]>,
  1157: 
  1158:     InstrItinData <tc_c818ff7f, /*tc_3stall*/
  1159:       [InstrStage<1, [SLOT0]>], [],
  1160:       []>,
  1161: 
  1162:     InstrItinData <tc_ce59038e, /*tc_st*/
  1163:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  1164:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1165: 
  1166:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  1167:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  1168:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1169: 
  1170:     InstrItinData <tc_d03278fd, /*tc_st*/
  1171:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  1172:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1173: 
  1174:     InstrItinData <tc_d234b61a, /*tc_st*/
  1175:       [InstrStage<1, [SLOT0]>], [1],
  1176:       [Hex_FWD]>,
  1177: 
  1178:     InstrItinData <tc_d33e5eee, /*tc_2early*/
  1179:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 2],
  1180:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1181: 
  1182:     InstrItinData <tc_d3632d88, /*tc_2*/
  1183:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  1184:       [Hex_FWD, Hex_FWD]>,
  1185: 
  1186:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  1187:       [InstrStage<1, [SLOT0]>], [1],
  1188:       [Hex_FWD]>,
  1189: 
  1190:     InstrItinData <tc_d57d649c, /*tc_2early*/
  1191:       [InstrStage<1, [SLOT2]>], [2],
  1192:       [Hex_FWD]>,
  1193: 
  1194:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  1195:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  1196:       [Hex_FWD, Hex_FWD]>,
  1197: 
  1198:     InstrItinData <tc_d68dca5c, /*tc_2early*/
  1199:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  1200:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1201: 
  1202:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  1203:       [InstrStage<1, [SLOT3]>], [2, 1],
  1204:       [Hex_FWD, Hex_FWD]>,
  1205: 
  1206:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  1207:       [InstrStage<1, [SLOT3]>], [1],
  1208:       [Hex_FWD]>,
  1209: 
  1210:     InstrItinData <tc_db596beb, /*tc_3stall*/
  1211:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  1212:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1213: 
  1214:     InstrItinData <tc_db96aa6b, /*tc_st*/
  1215:       [InstrStage<1, [SLOT0]>], [1],
  1216:       [Hex_FWD]>,
  1217: 
  1218:     InstrItinData <tc_dc51281d, /*tc_2early*/
  1219:       [InstrStage<1, [SLOT2]>], [2, 1],
  1220:       [Hex_FWD, Hex_FWD]>,
  1221: 
  1222:     InstrItinData <tc_decdde8a, /*tc_2early*/
  1223:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  1224:       [Hex_FWD]>,
  1225: 
  1226:     InstrItinData <tc_df5d53f9, /*tc_3stall*/
  1227:       [InstrStage<1, [SLOT0]>], [4, 2, 1],
  1228:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1229: 
  1230:     InstrItinData <tc_e3d699e3, /*tc_1*/
  1231:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  1232:       [Hex_FWD, Hex_FWD]>,
  1233: 
  1234:     InstrItinData <tc_e60def48, /*tc_2early*/
  1235:       [InstrStage<1, [SLOT2]>], [1],
  1236:       [Hex_FWD]>,
  1237: 
  1238:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  1239:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  1240:       [Hex_FWD, Hex_FWD]>,
  1241: 
  1242:     InstrItinData <tc_ed03645c, /*tc_2early*/
  1243:       [InstrStage<1, [SLOT2]>], [3, 1],
  1244:       [Hex_FWD, Hex_FWD]>,
  1245: 
  1246:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  1247:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  1248:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1249: 
  1250:     InstrItinData <tc_eed07714, /*tc_ld*/
  1251:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  1252:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1253: 
  1254:     InstrItinData <tc_eeda4109, /*tc_2early*/
  1255:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  1256:       [Hex_FWD, Hex_FWD]>,
  1257: 
  1258:     InstrItinData <tc_ef921005, /*tc_1*/
  1259:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  1260:       [Hex_FWD, Hex_FWD]>,
  1261: 
  1262:     InstrItinData <tc_f098b237, /*tc_1*/
  1263:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1264:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1265: 
  1266:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  1267:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  1268:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1269: 
  1270:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  1271:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  1272:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1273: 
  1274:     InstrItinData <tc_f34c1c21, /*tc_1*/
  1275:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1276:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1277: 
  1278:     InstrItinData <tc_f38f92e1, /*tc_3stall*/
  1279:       [InstrStage<1, [SLOT0]>], [2],
  1280:       [Hex_FWD]>,
  1281: 
  1282:     InstrItinData <tc_f529831b, /*tc_st*/
  1283:       [InstrStage<1, [SLOT0]>], [3, 3, 1, 2, 3],
  1284:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1285: 
  1286:     InstrItinData <tc_f6e2aff9, /*tc_3stall*/
  1287:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  1288:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1289: 
  1290:     InstrItinData <tc_f7569068, /*tc_3x*/
  1291:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1, 1],
  1292:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1293: 
  1294:     InstrItinData <tc_f97707c1, /*tc_1*/
  1295:       [InstrStage<1, [SLOT2]>], [2],
  1296:       [Hex_FWD]>,
  1297: 
  1298:     InstrItinData <tc_f999c66e, /*tc_2early*/
  1299:       [InstrStage<1, [SLOT2, SLOT3]>], [1, 2],
  1300:       [Hex_FWD, Hex_FWD]>,
  1301: 
  1302:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  1303:       [InstrStage<1, [SLOT3]>], [4, 2],
  1304:       [Hex_FWD, Hex_FWD]>,
  1305: 
  1306:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  1307:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  1308:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  1309:   ];
  1310: }
  1311: 
  1312: class DepScalarItinV60 {
  1313:   list<InstrItinData> DepScalarItinV60_list = [
  1314:     InstrItinData <tc_011e0e9d, /*tc_st*/
  1315:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  1316:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1317: 
  1318:     InstrItinData <tc_01d44cb2, /*tc_1*/
  1319:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1320:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1321: 
  1322:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  1323:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  1324:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1325: 
  1326:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  1327:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  1328:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1329: 
  1330:     InstrItinData <tc_0655b949, /*tc_st*/
  1331:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  1332:       [Hex_FWD, Hex_FWD]>,
  1333: 
  1334:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  1335:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  1336:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1337: 
  1338:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  1339:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  1340:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1341: 
  1342:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  1343:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  1344:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1345: 
  1346:     InstrItinData <tc_0ba0d5da, /*tc_2early*/
  1347:       [InstrStage<1, [SLOT2]>], [1],
  1348:       [Hex_FWD]>,
  1349: 
  1350:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  1351:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  1352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1353: 
  1354:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  1355:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  1356:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1357: 
  1358:     InstrItinData <tc_112d30d6, /*tc_1*/
  1359:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  1360:       [Hex_FWD]>,
  1361: 
  1362:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  1363:       [InstrStage<1, [SLOT0]>], [2],
  1364:       [Hex_FWD]>,
  1365: 
  1366:     InstrItinData <tc_1248597c, /*tc_3x*/
  1367:       [InstrStage<1, [SLOT3]>], [2, 2],
  1368:       [Hex_FWD, Hex_FWD]>,
  1369: 
  1370:     InstrItinData <tc_139ef484, /*tc_3stall*/
  1371:       [InstrStage<1, [SLOT2]>], [1, 1],
  1372:       [Hex_FWD, Hex_FWD]>,
  1373: 
  1374:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  1375:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  1376:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1377: 
  1378:     InstrItinData <tc_151bf368, /*tc_2early*/
  1379:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  1380:       [Hex_FWD, Hex_FWD]>,
  1381: 
  1382:     InstrItinData <tc_158aa3f7, /*tc_st*/
  1383:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  1384:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1385: 
  1386:     InstrItinData <tc_197dce51, /*tc_3x*/
  1387:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  1388:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1389: 
  1390:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  1391:       [InstrStage<1, [SLOT0]>], [3],
  1392:       [Hex_FWD]>,
  1393: 
  1394:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  1395:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  1396:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1397: 
  1398:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  1399:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  1400:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1401: 
  1402:     InstrItinData <tc_1d41f8b7, /*tc_3stall*/
  1403:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1, 1, 2],
  1404:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1405: 
  1406:     InstrItinData <tc_1fcb8495, /*tc_2*/
  1407:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  1408:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1409: 
  1410:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  1411:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 2],
  1412:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1413: 
  1414:     InstrItinData <tc_20131976, /*tc_1*/
  1415:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1416:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1417: 
  1418:     InstrItinData <tc_2237d952, /*tc_ld*/
  1419:       [InstrStage<1, [SLOT0]>], [1, 2],
  1420:       [Hex_FWD, Hex_FWD]>,
  1421: 
  1422:     InstrItinData <tc_23708a21, /*tc_2early*/
  1423:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  1424:       []>,
  1425: 
  1426:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  1427:       [InstrStage<1, [SLOT0]>], [4, 1],
  1428:       [Hex_FWD, Hex_FWD]>,
  1429: 
  1430:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  1431:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  1432:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1433: 
  1434:     InstrItinData <tc_24f426ab, /*tc_2early*/
  1435:       [InstrStage<1, [SLOT2, SLOT3]>], [1, 2, 2],
  1436:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1437: 
  1438:     InstrItinData <tc_27106296, /*tc_3x*/
  1439:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  1440:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1441: 
  1442:     InstrItinData <tc_280f7fe1, /*tc_st*/
  1443:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 2],
  1444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1445: 
  1446:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  1447:       [InstrStage<1, [SLOT3]>], [1, 1],
  1448:       [Hex_FWD, Hex_FWD]>,
  1449: 
  1450:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  1451:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  1452:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1453: 
  1454:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  1455:       [InstrStage<1, [SLOT3]>], [1],
  1456:       [Hex_FWD]>,
  1457: 
  1458:     InstrItinData <tc_2f573607, /*tc_2early*/
  1459:       [InstrStage<1, [SLOT2]>], [2, 1],
  1460:       [Hex_FWD, Hex_FWD]>,
  1461: 
  1462:     InstrItinData <tc_33e7e673, /*tc_2early*/
  1463:       [InstrStage<1, [SLOT2]>], [],
  1464:       []>,
  1465: 
  1466:     InstrItinData <tc_362b0be2, /*tc_2early*/
  1467:       [InstrStage<1, [SLOT2]>], [1],
  1468:       [Hex_FWD]>,
  1469: 
  1470:     InstrItinData <tc_38382228, /*tc_3x*/
  1471:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  1472:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1473: 
  1474:     InstrItinData <tc_388f9897, /*tc_1*/
  1475:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  1476:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1477: 
  1478:     InstrItinData <tc_38e0bae9, /*tc_3stall*/
  1479:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1, 1, 1],
  1480:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1481: 
  1482:     InstrItinData <tc_3d14a17b, /*tc_1*/
  1483:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  1484:       [Hex_FWD, Hex_FWD]>,
  1485: 
  1486:     InstrItinData <tc_3edca78f, /*tc_2*/
  1487:       [InstrStage<1, [SLOT3]>], [4, 2],
  1488:       [Hex_FWD, Hex_FWD]>,
  1489: 
  1490:     InstrItinData <tc_3fbf1042, /*tc_1*/
  1491:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  1492:       [Hex_FWD]>,
  1493: 
  1494:     InstrItinData <tc_407e96f9, /*tc_1*/
  1495:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1496:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1497: 
  1498:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  1499:       [InstrStage<1, [SLOT0]>], [3, 2],
  1500:       [Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV60, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV60; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV60 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV60 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 1501-2000 / 第 1501-2000 行

```tablegen
  1501: 
  1502:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  1503:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  1504:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1505: 
  1506:     InstrItinData <tc_42ff66ba, /*tc_2early*/
  1507:       [InstrStage<1, [SLOT2]>], [2, 1],
  1508:       [Hex_FWD, Hex_FWD]>,
  1509: 
  1510:     InstrItinData <tc_442395f3, /*tc_1*/
  1511:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2, 2],
  1512:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1513: 
  1514:     InstrItinData <tc_449acf79, /*tc_st*/
  1515:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 3, 1, 2, 2],
  1516:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1517: 
  1518:     InstrItinData <tc_44d5a428, /*tc_st*/
  1519:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  1520:       [Hex_FWD, Hex_FWD]>,
  1521: 
  1522:     InstrItinData <tc_44fffc58, /*tc_2early*/
  1523:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  1524:       [Hex_FWD]>,
  1525: 
  1526:     InstrItinData <tc_45791fb8, /*tc_ld*/
  1527:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  1528:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1529: 
  1530:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  1531:       [InstrStage<1, [SLOT2]>], [2],
  1532:       [Hex_FWD]>,
  1533: 
  1534:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  1535:       [InstrStage<1, [SLOT3]>], [4, 1],
  1536:       [Hex_FWD, Hex_FWD]>,
  1537: 
  1538:     InstrItinData <tc_49fdfd4b, /*tc_3x*/
  1539:       [InstrStage<1, [SLOT3]>], [4, 1],
  1540:       [Hex_FWD, Hex_FWD]>,
  1541: 
  1542:     InstrItinData <tc_4a55d03c, /*tc_2early*/
  1543:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 1],
  1544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1545: 
  1546:     InstrItinData <tc_4abdbdc6, /*tc_3stall*/
  1547:       [InstrStage<1, [SLOT3]>], [2, 2],
  1548:       [Hex_FWD, Hex_FWD]>,
  1549: 
  1550:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  1551:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  1552:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1553: 
  1554:     InstrItinData <tc_4bf903b0, /*tc_st*/
  1555:       [InstrStage<1, [SLOT0]>], [3],
  1556:       [Hex_FWD]>,
  1557: 
  1558:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  1559:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  1560:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1561: 
  1562:     InstrItinData <tc_512b1653, /*tc_st*/
  1563:       [InstrStage<1, [SLOT0]>], [1, 2],
  1564:       [Hex_FWD, Hex_FWD]>,
  1565: 
  1566:     InstrItinData <tc_53c851ab, /*tc_2early*/
  1567:       [InstrStage<1, [SLOT2]>], [3, 2, 2],
  1568:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1569: 
  1570:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  1571:       [InstrStage<1, [SLOT3]>], [1],
  1572:       [Hex_FWD]>,
  1573: 
  1574:     InstrItinData <tc_5502c366, /*tc_1*/
  1575:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1576:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1577: 
  1578:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  1579:       [InstrStage<1, [SLOT3]>], [],
  1580:       []>,
  1581: 
  1582:     InstrItinData <tc_556f6577, /*tc_3x*/
  1583:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  1584:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1585: 
  1586:     InstrItinData <tc_55a9a350, /*tc_st*/
  1587:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  1588:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1589: 
  1590:     InstrItinData <tc_55b33fda, /*tc_2early*/
  1591:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1],
  1592:       [Hex_FWD, Hex_FWD]>,
  1593: 
  1594:     InstrItinData <tc_56a124a7, /*tc_2early*/
  1595:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  1596:       [Hex_FWD, Hex_FWD]>,
  1597: 
  1598:     InstrItinData <tc_57a55b54, /*tc_2early*/
  1599:       [InstrStage<1, [SLOT3]>], [1, 2],
  1600:       [Hex_FWD, Hex_FWD]>,
  1601: 
  1602:     InstrItinData <tc_5944960d, /*tc_ld*/
  1603:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  1604:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1605: 
  1606:     InstrItinData <tc_59a7822c, /*tc_2early*/
  1607:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  1608:       [Hex_FWD, Hex_FWD]>,
  1609: 
  1610:     InstrItinData <tc_5a222e89, /*tc_2early*/
  1611:       [InstrStage<1, [SLOT2]>], [1, 1],
  1612:       [Hex_FWD, Hex_FWD]>,
  1613: 
  1614:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  1615:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  1616:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1617: 
  1618:     InstrItinData <tc_5b347363, /*tc_1*/
  1619:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  1620:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1621: 
  1622:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  1623:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  1624:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1625: 
  1626:     InstrItinData <tc_5da50c4b, /*tc_1*/
  1627:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1628:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1629: 
  1630:     InstrItinData <tc_5deb5e47, /*tc_st*/
  1631:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  1632:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1633: 
  1634:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  1635:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  1636:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1637: 
  1638:     InstrItinData <tc_5f2afaf7, /*tc_ld*/
  1639:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 3, 1, 2],
  1640:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1641: 
  1642:     InstrItinData <tc_60e324ff, /*tc_2early*/
  1643:       [InstrStage<1, [SLOT2]>], [1],
  1644:       [Hex_FWD]>,
  1645: 
  1646:     InstrItinData <tc_63567288, /*tc_1*/
  1647:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  1648:       [Hex_FWD]>,
  1649: 
  1650:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  1651:       [InstrStage<1, [SLOT0]>], [4, 1],
  1652:       [Hex_FWD, Hex_FWD]>,
  1653: 
  1654:     InstrItinData <tc_651cbe02, /*tc_2early*/
  1655:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  1656:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1657: 
  1658:     InstrItinData <tc_65279839, /*tc_2*/
  1659:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  1660:       [Hex_FWD, Hex_FWD]>,
  1661: 
  1662:     InstrItinData <tc_65cbd974, /*tc_st*/
  1663:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  1664:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1665: 
  1666:     InstrItinData <tc_69bfb303, /*tc_2early*/
  1667:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  1668:       [Hex_FWD, Hex_FWD]>,
  1669: 
  1670:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  1671:       [InstrStage<1, [SLOT3]>], [4, 1],
  1672:       [Hex_FWD, Hex_FWD]>,
  1673: 
  1674:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  1675:       [InstrStage<1, [SLOT3]>], [4, 1],
  1676:       [Hex_FWD, Hex_FWD]>,
  1677: 
  1678:     InstrItinData <tc_6d861a95, /*tc_3stall*/
  1679:       [InstrStage<1, [SLOT3]>], [2, 1],
  1680:       [Hex_FWD, Hex_FWD]>,
  1681: 
  1682:     InstrItinData <tc_6e20402a, /*tc_st*/
  1683:       [InstrStage<1, [SLOT0]>], [2, 3],
  1684:       [Hex_FWD, Hex_FWD]>,
  1685: 
  1686:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  1687:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  1688:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1689: 
  1690:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  1691:       [InstrStage<1, [SLOT0]>], [1, 1],
  1692:       [Hex_FWD, Hex_FWD]>,
  1693: 
  1694:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  1695:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  1696:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1697: 
  1698:     InstrItinData <tc_711c805f, /*tc_2early*/
  1699:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  1700:       [Hex_FWD, Hex_FWD]>,
  1701: 
  1702:     InstrItinData <tc_713b66bf, /*tc_1*/
  1703:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  1704:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1705: 
  1706:     InstrItinData <tc_7401744f, /*tc_2*/
  1707:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  1708:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1709: 
  1710:     InstrItinData <tc_7476d766, /*tc_3x*/
  1711:       [InstrStage<1, [SLOT3]>], [4, 2],
  1712:       [Hex_FWD, Hex_FWD]>,
  1713: 
  1714:     InstrItinData <tc_74a42bda, /*tc_ld*/
  1715:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  1716:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1717: 
  1718:     InstrItinData <tc_759e57be, /*tc_3stall*/
  1719:       [InstrStage<1, [SLOT2]>], [4, 1],
  1720:       [Hex_FWD, Hex_FWD]>,
  1721: 
  1722:     InstrItinData <tc_76bb5435, /*tc_ld*/
  1723:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  1724:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1725: 
  1726:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  1727:       [InstrStage<1, [SLOT2]>], [1],
  1728:       [Hex_FWD]>,
  1729: 
  1730:     InstrItinData <tc_77f94a5e, /*tc_st*/
  1731:       [InstrStage<1, [SLOT0]>], [],
  1732:       []>,
  1733: 
  1734:     InstrItinData <tc_788b1d09, /*tc_3x*/
  1735:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  1736:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1737: 
  1738:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  1739:       [InstrStage<1, [SLOT0]>], [],
  1740:       []>,
  1741: 
  1742:     InstrItinData <tc_7af3a37e, /*tc_st*/
  1743:       [InstrStage<1, [SLOT0]>], [1, 3],
  1744:       [Hex_FWD, Hex_FWD]>,
  1745: 
  1746:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  1747:       [InstrStage<1, [SLOT0]>], [3, 2],
  1748:       [Hex_FWD, Hex_FWD]>,
  1749: 
  1750:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  1751:       [InstrStage<1, [SLOT0]>], [3],
  1752:       [Hex_FWD]>,
  1753: 
  1754:     InstrItinData <tc_7c31e19a, /*tc_st*/
  1755:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  1756:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1757: 
  1758:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  1759:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  1760:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1761: 
  1762:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  1763:       [InstrStage<1, [SLOT3]>], [4, 1],
  1764:       [Hex_FWD, Hex_FWD]>,
  1765: 
  1766:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  1767:       [InstrStage<1, [SLOT3]>], [],
  1768:       []>,
  1769: 
  1770:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  1771:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  1772:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1773: 
  1774:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  1775:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  1776:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1777: 
  1778:     InstrItinData <tc_8035e91f, /*tc_st*/
  1779:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  1780:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1781: 
  1782:     InstrItinData <tc_822c3c68, /*tc_ld*/
  1783:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  1784:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1785: 
  1786:     InstrItinData <tc_829d8a86, /*tc_st*/
  1787:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  1788:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1789: 
  1790:     InstrItinData <tc_838c4d7a, /*tc_st*/
  1791:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  1792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1793: 
  1794:     InstrItinData <tc_84a7500d, /*tc_1*/
  1795:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  1796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1797: 
  1798:     InstrItinData <tc_86173609, /*tc_1*/
  1799:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2],
  1800:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1801: 
  1802:     InstrItinData <tc_887d1bb7, /*tc_st*/
  1803:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 2],
  1804:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1805: 
  1806:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  1807:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  1808:       [Hex_FWD, Hex_FWD]>,
  1809: 
  1810:     InstrItinData <tc_8a825db2, /*tc_2*/
  1811:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  1812:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1813: 
  1814:     InstrItinData <tc_8b5bd4f5, /*tc_1*/
  1815:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  1816:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1817: 
  1818:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  1819:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 2],
  1820:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1821: 
  1822:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  1823:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  1824:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1825: 
  1826:     InstrItinData <tc_9124c04f, /*tc_1*/
  1827:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  1828:       [Hex_FWD, Hex_FWD]>,
  1829: 
  1830:     InstrItinData <tc_92240447, /*tc_st*/
  1831:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  1832:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1833: 
  1834:     InstrItinData <tc_934753bb, /*tc_ld*/
  1835:       [InstrStage<1, [SLOT0]>], [4, 2, 2],
  1836:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1837: 
  1838:     InstrItinData <tc_937dd41c, /*tc_ld*/
  1839:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  1840:       []>,
  1841: 
  1842:     InstrItinData <tc_9406230a, /*tc_3x*/
  1843:       [InstrStage<1, [SLOT3]>], [2, 1],
  1844:       [Hex_FWD, Hex_FWD]>,
  1845: 
  1846:     InstrItinData <tc_95a33176, /*tc_2*/
  1847:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  1848:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1849: 
  1850:     InstrItinData <tc_95f43c5e, /*tc_3*/
  1851:       [InstrStage<1, [SLOT2]>], [1],
  1852:       [Hex_FWD]>,
  1853: 
  1854:     InstrItinData <tc_96ef76ef, /*tc_st*/
  1855:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  1856:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1857: 
  1858:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  1859:       [InstrStage<1, [SLOT0]>], [2, 3, 2],
  1860:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1861: 
  1862:     InstrItinData <tc_9783714b, /*tc_4x*/
  1863:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  1864:       [Hex_FWD, Hex_FWD]>,
  1865: 
  1866:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  1867:       [InstrStage<1, [SLOT2]>], [4, 1],
  1868:       [Hex_FWD, Hex_FWD]>,
  1869: 
  1870:     InstrItinData <tc_9b34f5e0, /*tc_2early*/
  1871:       [InstrStage<1, [SLOT2]>], [],
  1872:       []>,
  1873: 
  1874:     InstrItinData <tc_9b3c0462, /*tc_2*/
  1875:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  1876:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1877: 
  1878:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  1879:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  1880:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1881: 
  1882:     InstrItinData <tc_9c52f549, /*tc_2early*/
  1883:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 1],
  1884:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1885: 
  1886:     InstrItinData <tc_9e27f2f9, /*tc_2early*/
  1887:       [InstrStage<1, [SLOT2, SLOT3]>], [1, 1, 2],
  1888:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1889: 
  1890:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  1891:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  1892:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1893: 
  1894:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  1895:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  1896:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1897: 
  1898:     InstrItinData <tc_9edefe01, /*tc_st*/
  1899:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 2],
  1900:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1901: 
  1902:     InstrItinData <tc_9f6cd987, /*tc_1*/
  1903:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  1904:       [Hex_FWD, Hex_FWD]>,
  1905: 
  1906:     InstrItinData <tc_a08b630b, /*tc_2*/
  1907:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  1908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1909: 
  1910:     InstrItinData <tc_a1297125, /*tc_2early*/
  1911:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 2],
  1912:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1913: 
  1914:     InstrItinData <tc_a154b476, /*tc_3x*/
  1915:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  1916:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1917: 
  1918:     InstrItinData <tc_a2b365d2, /*tc_st*/
  1919:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  1920:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1921: 
  1922:     InstrItinData <tc_a3070909, /*tc_3stall*/
  1923:       [InstrStage<1, [SLOT0]>], [1, 1],
  1924:       [Hex_FWD, Hex_FWD]>,
  1925: 
  1926:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  1927:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  1928:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1929: 
  1930:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  1931:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  1932:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1933: 
  1934:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  1935:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  1936:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1937: 
  1938:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  1939:       [InstrStage<1, [SLOT0]>], [],
  1940:       []>,
  1941: 
  1942:     InstrItinData <tc_a724463d, /*tc_3stall*/
  1943:       [InstrStage<1, [SLOT0]>], [4, 1],
  1944:       [Hex_FWD, Hex_FWD]>,
  1945: 
  1946:     InstrItinData <tc_a7a13fac, /*tc_2early*/
  1947:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  1948:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1949: 
  1950:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  1951:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  1952:       [Hex_FWD, Hex_FWD]>,
  1953: 
  1954:     InstrItinData <tc_a9edeffa, /*tc_st*/
  1955:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  1956:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1957: 
  1958:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  1959:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  1960:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1961: 
  1962:     InstrItinData <tc_ac65613f, /*tc_ld*/
  1963:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  1964:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1965: 
  1966:     InstrItinData <tc_addc37a8, /*tc_st*/
  1967:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  1968:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1969: 
  1970:     InstrItinData <tc_ae5babd7, /*tc_st*/
  1971:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  1972:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  1973: 
  1974:     InstrItinData <tc_aee6250c, /*tc_ld*/
  1975:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  1976:       [Hex_FWD, Hex_FWD]>,
  1977: 
  1978:     InstrItinData <tc_af6af259, /*tc_ld*/
  1979:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  1980:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1981: 
  1982:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  1983:       [InstrStage<1, [SLOT0]>], [1],
  1984:       [Hex_FWD]>,
  1985: 
  1986:     InstrItinData <tc_b4dc7630, /*tc_st*/
  1987:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 2],
  1988:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1989: 
  1990:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  1991:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  1992:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  1993: 
  1994:     InstrItinData <tc_b837298f, /*tc_1*/
  1995:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  1996:       []>,
  1997: 
  1998:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  1999:       [InstrStage<1, [SLOT2]>], [],
  2000:       []>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2001-2500 / 第 2001-2500 行

```tablegen
  2001: 
  2002:     InstrItinData <tc_ba9255a6, /*tc_st*/
  2003:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 2],
  2004:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2005: 
  2006:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  2007:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  2008:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2009: 
  2010:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  2011:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  2012:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2013: 
  2014:     InstrItinData <tc_bb831a7c, /*tc_2*/
  2015:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  2016:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2017: 
  2018:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  2019:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  2020:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2021: 
  2022:     InstrItinData <tc_c20701f0, /*tc_2*/
  2023:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2024:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2025: 
  2026:     InstrItinData <tc_c21d7447, /*tc_3x*/
  2027:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  2028:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2029: 
  2030:     InstrItinData <tc_c57d9f39, /*tc_1*/
  2031:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  2032:       [Hex_FWD, Hex_FWD]>,
  2033: 
  2034:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  2035:       [InstrStage<1, [SLOT0]>], [],
  2036:       []>,
  2037: 
  2038:     InstrItinData <tc_ce59038e, /*tc_st*/
  2039:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  2040:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2041: 
  2042:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  2043:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  2044:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2045: 
  2046:     InstrItinData <tc_d03278fd, /*tc_st*/
  2047:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  2048:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2049: 
  2050:     InstrItinData <tc_d234b61a, /*tc_st*/
  2051:       [InstrStage<1, [SLOT0]>], [1],
  2052:       [Hex_FWD]>,
  2053: 
  2054:     InstrItinData <tc_d33e5eee, /*tc_2early*/
  2055:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 2],
  2056:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2057: 
  2058:     InstrItinData <tc_d3632d88, /*tc_2*/
  2059:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  2060:       [Hex_FWD, Hex_FWD]>,
  2061: 
  2062:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  2063:       [InstrStage<1, [SLOT0]>], [1],
  2064:       [Hex_FWD]>,
  2065: 
  2066:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  2067:       [InstrStage<1, [SLOT2]>], [2],
  2068:       [Hex_FWD]>,
  2069: 
  2070:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  2071:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  2072:       [Hex_FWD, Hex_FWD]>,
  2073: 
  2074:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  2075:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  2076:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2077: 
  2078:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  2079:       [InstrStage<1, [SLOT3]>], [2, 1],
  2080:       [Hex_FWD, Hex_FWD]>,
  2081: 
  2082:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  2083:       [InstrStage<1, [SLOT3]>], [1],
  2084:       [Hex_FWD]>,
  2085: 
  2086:     InstrItinData <tc_db596beb, /*tc_3stall*/
  2087:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  2088:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2089: 
  2090:     InstrItinData <tc_db96aa6b, /*tc_st*/
  2091:       [InstrStage<1, [SLOT0]>], [1],
  2092:       [Hex_FWD]>,
  2093: 
  2094:     InstrItinData <tc_dc51281d, /*tc_2early*/
  2095:       [InstrStage<1, [SLOT2]>], [2, 1],
  2096:       [Hex_FWD, Hex_FWD]>,
  2097: 
  2098:     InstrItinData <tc_decdde8a, /*tc_2early*/
  2099:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  2100:       [Hex_FWD]>,
  2101: 
  2102:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  2103:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  2104:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2105: 
  2106:     InstrItinData <tc_e3d699e3, /*tc_2*/
  2107:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  2108:       [Hex_FWD, Hex_FWD]>,
  2109: 
  2110:     InstrItinData <tc_e60def48, /*tc_2early*/
  2111:       [InstrStage<1, [SLOT2]>], [1],
  2112:       [Hex_FWD]>,
  2113: 
  2114:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  2115:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  2116:       [Hex_FWD, Hex_FWD]>,
  2117: 
  2118:     InstrItinData <tc_ed03645c, /*tc_2early*/
  2119:       [InstrStage<1, [SLOT2]>], [3, 1],
  2120:       [Hex_FWD, Hex_FWD]>,
  2121: 
  2122:     InstrItinData <tc_eed07714, /*tc_ld*/
  2123:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  2124:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2125: 
  2126:     InstrItinData <tc_eeda4109, /*tc_2early*/
  2127:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  2128:       [Hex_FWD, Hex_FWD]>,
  2129: 
  2130:     InstrItinData <tc_ef921005, /*tc_1*/
  2131:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  2132:       [Hex_FWD, Hex_FWD]>,
  2133: 
  2134:     InstrItinData <tc_f098b237, /*tc_1*/
  2135:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  2136:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2137: 
  2138:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  2139:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  2140:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2141: 
  2142:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  2143:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  2144:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2145: 
  2146:     InstrItinData <tc_f34c1c21, /*tc_2*/
  2147:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2148:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2149: 
  2150:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  2151:       [InstrStage<1, [SLOT0]>], [2],
  2152:       [Hex_FWD]>,
  2153: 
  2154:     InstrItinData <tc_f529831b, /*tc_st*/
  2155:       [InstrStage<1, [SLOT0]>], [3, 3, 1, 2, 3],
  2156:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2157: 
  2158:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  2159:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  2160:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2161: 
  2162:     InstrItinData <tc_f7569068, /*tc_4x*/
  2163:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  2164:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2165: 
  2166:     InstrItinData <tc_f97707c1, /*tc_1*/
  2167:       [InstrStage<1, [SLOT2]>], [2],
  2168:       [Hex_FWD]>,
  2169: 
  2170:     InstrItinData <tc_f999c66e, /*tc_2early*/
  2171:       [InstrStage<1, [SLOT2, SLOT3]>], [1, 2],
  2172:       [Hex_FWD, Hex_FWD]>,
  2173: 
  2174:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  2175:       [InstrStage<1, [SLOT3]>], [4, 2],
  2176:       [Hex_FWD, Hex_FWD]>,
  2177: 
  2178:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  2179:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  2180:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  2181:   ];
  2182: }
  2183: 
  2184: class DepScalarItinV60se {
  2185:   list<InstrItinData> DepScalarItinV60se_list = [
  2186:     InstrItinData <tc_011e0e9d, /*tc_st*/
  2187:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  2188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2189: 
  2190:     InstrItinData <tc_01d44cb2, /*tc_1*/
  2191:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  2192:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2193: 
  2194:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  2195:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  2196:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2197: 
  2198:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  2199:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  2200:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2201: 
  2202:     InstrItinData <tc_0655b949, /*tc_st*/
  2203:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  2204:       [Hex_FWD, Hex_FWD]>,
  2205: 
  2206:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  2207:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  2208:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2209: 
  2210:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  2211:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  2212:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2213: 
  2214:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  2215:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  2216:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2217: 
  2218:     InstrItinData <tc_0ba0d5da, /*tc_2early*/
  2219:       [InstrStage<1, [SLOT2]>], [1],
  2220:       [Hex_FWD]>,
  2221: 
  2222:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  2223:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2224:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2225: 
  2226:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  2227:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  2228:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2229: 
  2230:     InstrItinData <tc_112d30d6, /*tc_1*/
  2231:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  2232:       [Hex_FWD]>,
  2233: 
  2234:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  2235:       [InstrStage<1, [SLOT0]>], [2],
  2236:       [Hex_FWD]>,
  2237: 
  2238:     InstrItinData <tc_1248597c, /*tc_3x*/
  2239:       [InstrStage<1, [SLOT3]>], [2, 2],
  2240:       [Hex_FWD, Hex_FWD]>,
  2241: 
  2242:     InstrItinData <tc_139ef484, /*tc_3stall*/
  2243:       [InstrStage<1, [SLOT2]>], [1, 1],
  2244:       [Hex_FWD, Hex_FWD]>,
  2245: 
  2246:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  2247:       [InstrStage<1, [SLOT0], 0>,
  2248:        InstrStage<1, [CVI_ST]>], [3, 3, 2],
  2249:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2250: 
  2251:     InstrItinData <tc_151bf368, /*tc_2early*/
  2252:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  2253:       [Hex_FWD, Hex_FWD]>,
  2254: 
  2255:     InstrItinData <tc_158aa3f7, /*tc_st*/
  2256:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  2257:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2258: 
  2259:     InstrItinData <tc_197dce51, /*tc_3x*/
  2260:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  2261:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2262: 
  2263:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  2264:       [InstrStage<1, [SLOT0], 0>,
  2265:        InstrStage<1, [CVI_ST]>], [3],
  2266:       [Hex_FWD]>,
  2267: 
  2268:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  2269:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  2270:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2271: 
  2272:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  2273:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  2274:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2275: 
  2276:     InstrItinData <tc_1d41f8b7, /*tc_3stall*/
  2277:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1, 1, 2],
  2278:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2279: 
  2280:     InstrItinData <tc_1fcb8495, /*tc_2*/
  2281:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2282:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2283: 
  2284:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  2285:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 2],
  2286:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2287: 
  2288:     InstrItinData <tc_20131976, /*tc_1*/
  2289:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  2290:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2291: 
  2292:     InstrItinData <tc_2237d952, /*tc_ld*/
  2293:       [InstrStage<1, [SLOT0]>], [1, 2],
  2294:       [Hex_FWD, Hex_FWD]>,
  2295: 
  2296:     InstrItinData <tc_23708a21, /*tc_2early*/
  2297:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3], 0>,
  2298:        InstrStage<1, [CVI_ST]>], [],
  2299:       []>,
  2300: 
  2301:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  2302:       [InstrStage<1, [SLOT0]>], [4, 1],
  2303:       [Hex_FWD, Hex_FWD]>,
  2304: 
  2305:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  2306:       [InstrStage<1, [SLOT0], 0>,
  2307:        InstrStage<1, [CVI_ST]>], [3, 2, 2],
  2308:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2309: 
  2310:     InstrItinData <tc_24f426ab, /*tc_2early*/
  2311:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2312:        InstrStage<1, [CVI_ST]>], [1, 2, 2],
  2313:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2314: 
  2315:     InstrItinData <tc_27106296, /*tc_3x*/
  2316:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  2317:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2318: 
  2319:     InstrItinData <tc_280f7fe1, /*tc_st*/
  2320:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 2],
  2321:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2322: 
  2323:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  2324:       [InstrStage<1, [SLOT3]>], [1, 1],
  2325:       [Hex_FWD, Hex_FWD]>,
  2326: 
  2327:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  2328:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  2329:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2330: 
  2331:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  2332:       [InstrStage<1, [SLOT3]>], [1],
  2333:       [Hex_FWD]>,
  2334: 
  2335:     InstrItinData <tc_2f573607, /*tc_2early*/
  2336:       [InstrStage<1, [SLOT2], 0>,
  2337:        InstrStage<1, [CVI_ST]>], [2, 1],
  2338:       [Hex_FWD, Hex_FWD]>,
  2339: 
  2340:     InstrItinData <tc_33e7e673, /*tc_2early*/
  2341:       [InstrStage<1, [SLOT2], 0>,
  2342:        InstrStage<1, [CVI_ST]>], [],
  2343:       []>,
  2344: 
  2345:     InstrItinData <tc_362b0be2, /*tc_2early*/
  2346:       [InstrStage<1, [SLOT2], 0>,
  2347:        InstrStage<1, [CVI_ST]>], [1],
  2348:       [Hex_FWD]>,
  2349: 
  2350:     InstrItinData <tc_38382228, /*tc_3x*/
  2351:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  2352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2353: 
  2354:     InstrItinData <tc_388f9897, /*tc_1*/
  2355:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  2356:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2357: 
  2358:     InstrItinData <tc_38e0bae9, /*tc_3stall*/
  2359:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 1, 1, 1],
  2360:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2361: 
  2362:     InstrItinData <tc_3d14a17b, /*tc_1*/
  2363:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  2364:       [Hex_FWD, Hex_FWD]>,
  2365: 
  2366:     InstrItinData <tc_3edca78f, /*tc_2*/
  2367:       [InstrStage<1, [SLOT3]>], [4, 2],
  2368:       [Hex_FWD, Hex_FWD]>,
  2369: 
  2370:     InstrItinData <tc_3fbf1042, /*tc_1*/
  2371:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  2372:       [Hex_FWD]>,
  2373: 
  2374:     InstrItinData <tc_407e96f9, /*tc_1*/
  2375:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  2376:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2377: 
  2378:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  2379:       [InstrStage<1, [SLOT0], 0>,
  2380:        InstrStage<1, [CVI_ST]>], [3, 2],
  2381:       [Hex_FWD, Hex_FWD]>,
  2382: 
  2383:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  2384:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  2385:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2386: 
  2387:     InstrItinData <tc_42ff66ba, /*tc_2early*/
  2388:       [InstrStage<1, [SLOT2], 0>,
  2389:        InstrStage<1, [CVI_ST]>], [2, 1],
  2390:       [Hex_FWD, Hex_FWD]>,
  2391: 
  2392:     InstrItinData <tc_442395f3, /*tc_1*/
  2393:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2, 2],
  2394:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2395: 
  2396:     InstrItinData <tc_449acf79, /*tc_st*/
  2397:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 3, 1, 2, 2],
  2398:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2399: 
  2400:     InstrItinData <tc_44d5a428, /*tc_st*/
  2401:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  2402:       [Hex_FWD, Hex_FWD]>,
  2403: 
  2404:     InstrItinData <tc_44fffc58, /*tc_2early*/
  2405:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2406:        InstrStage<1, [CVI_ST]>], [2],
  2407:       [Hex_FWD]>,
  2408: 
  2409:     InstrItinData <tc_45791fb8, /*tc_ld*/
  2410:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  2411:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2412: 
  2413:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  2414:       [InstrStage<1, [SLOT2], 0>,
  2415:        InstrStage<1, [CVI_ST]>], [2],
  2416:       [Hex_FWD]>,
  2417: 
  2418:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  2419:       [InstrStage<1, [SLOT3]>], [4, 1],
  2420:       [Hex_FWD, Hex_FWD]>,
  2421: 
  2422:     InstrItinData <tc_49fdfd4b, /*tc_3x*/
  2423:       [InstrStage<1, [SLOT3]>], [4, 1],
  2424:       [Hex_FWD, Hex_FWD]>,
  2425: 
  2426:     InstrItinData <tc_4a55d03c, /*tc_2early*/
  2427:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 1],
  2428:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2429: 
  2430:     InstrItinData <tc_4abdbdc6, /*tc_3stall*/
  2431:       [InstrStage<1, [SLOT3]>], [2, 2],
  2432:       [Hex_FWD, Hex_FWD]>,
  2433: 
  2434:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  2435:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  2436:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2437: 
  2438:     InstrItinData <tc_4bf903b0, /*tc_st*/
  2439:       [InstrStage<1, [SLOT0]>], [3],
  2440:       [Hex_FWD]>,
  2441: 
  2442:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  2443:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  2444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2445: 
  2446:     InstrItinData <tc_512b1653, /*tc_st*/
  2447:       [InstrStage<1, [SLOT0]>], [1, 2],
  2448:       [Hex_FWD, Hex_FWD]>,
  2449: 
  2450:     InstrItinData <tc_53c851ab, /*tc_2early*/
  2451:       [InstrStage<1, [SLOT2], 0>,
  2452:        InstrStage<1, [CVI_ST]>], [3, 2, 2],
  2453:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2454: 
  2455:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  2456:       [InstrStage<1, [SLOT3]>], [1],
  2457:       [Hex_FWD]>,
  2458: 
  2459:     InstrItinData <tc_5502c366, /*tc_1*/
  2460:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2461:        InstrStage<1, [CVI_ST]>], [3, 2, 2],
  2462:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2463: 
  2464:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  2465:       [InstrStage<1, [SLOT3]>], [],
  2466:       []>,
  2467: 
  2468:     InstrItinData <tc_556f6577, /*tc_3x*/
  2469:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  2470:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2471: 
  2472:     InstrItinData <tc_55a9a350, /*tc_st*/
  2473:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  2474:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2475: 
  2476:     InstrItinData <tc_55b33fda, /*tc_2early*/
  2477:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1],
  2478:       [Hex_FWD, Hex_FWD]>,
  2479: 
  2480:     InstrItinData <tc_56a124a7, /*tc_2early*/
  2481:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2482:        InstrStage<1, [CVI_ST]>], [2, 2],
  2483:       [Hex_FWD, Hex_FWD]>,
  2484: 
  2485:     InstrItinData <tc_57a55b54, /*tc_2early*/
  2486:       [InstrStage<1, [SLOT3], 0>,
  2487:        InstrStage<1, [CVI_ST]>], [1, 2],
  2488:       [Hex_FWD, Hex_FWD]>,
  2489: 
  2490:     InstrItinData <tc_5944960d, /*tc_ld*/
  2491:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  2492:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2493: 
  2494:     InstrItinData <tc_59a7822c, /*tc_2early*/
  2495:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  2496:       [Hex_FWD, Hex_FWD]>,
  2497: 
  2498:     InstrItinData <tc_5a222e89, /*tc_2early*/
  2499:       [InstrStage<1, [SLOT2]>], [1, 1],
  2500:       [Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV60se, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV60se; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV60se 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV60se 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 2501-3000 / 第 2501-3000 行

```tablegen
  2501: 
  2502:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  2503:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  2504:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2505: 
  2506:     InstrItinData <tc_5b347363, /*tc_1*/
  2507:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  2508:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2509: 
  2510:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  2511:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  2512:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2513: 
  2514:     InstrItinData <tc_5da50c4b, /*tc_1*/
  2515:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  2516:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2517: 
  2518:     InstrItinData <tc_5deb5e47, /*tc_st*/
  2519:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  2520:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2521: 
  2522:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  2523:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  2524:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2525: 
  2526:     InstrItinData <tc_5f2afaf7, /*tc_ld*/
  2527:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 3, 1, 2],
  2528:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2529: 
  2530:     InstrItinData <tc_60e324ff, /*tc_2early*/
  2531:       [InstrStage<1, [SLOT2], 0>,
  2532:        InstrStage<1, [CVI_ST]>], [1],
  2533:       [Hex_FWD]>,
  2534: 
  2535:     InstrItinData <tc_63567288, /*tc_1*/
  2536:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  2537:       [Hex_FWD]>,
  2538: 
  2539:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  2540:       [InstrStage<1, [SLOT0]>], [4, 1],
  2541:       [Hex_FWD, Hex_FWD]>,
  2542: 
  2543:     InstrItinData <tc_651cbe02, /*tc_2early*/
  2544:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  2545:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2546: 
  2547:     InstrItinData <tc_65279839, /*tc_2*/
  2548:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  2549:       [Hex_FWD, Hex_FWD]>,
  2550: 
  2551:     InstrItinData <tc_65cbd974, /*tc_st*/
  2552:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  2553:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2554: 
  2555:     InstrItinData <tc_69bfb303, /*tc_2early*/
  2556:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2557:        InstrStage<1, [CVI_ST]>], [2, 2],
  2558:       [Hex_FWD, Hex_FWD]>,
  2559: 
  2560:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  2561:       [InstrStage<1, [SLOT3]>], [4, 1],
  2562:       [Hex_FWD, Hex_FWD]>,
  2563: 
  2564:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  2565:       [InstrStage<1, [SLOT3]>], [4, 1],
  2566:       [Hex_FWD, Hex_FWD]>,
  2567: 
  2568:     InstrItinData <tc_6d861a95, /*tc_3stall*/
  2569:       [InstrStage<1, [SLOT3]>], [2, 1],
  2570:       [Hex_FWD, Hex_FWD]>,
  2571: 
  2572:     InstrItinData <tc_6e20402a, /*tc_st*/
  2573:       [InstrStage<1, [SLOT0]>], [2, 3],
  2574:       [Hex_FWD, Hex_FWD]>,
  2575: 
  2576:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  2577:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  2578:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2579: 
  2580:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  2581:       [InstrStage<1, [SLOT0]>], [1, 1],
  2582:       [Hex_FWD, Hex_FWD]>,
  2583: 
  2584:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  2585:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  2586:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2587: 
  2588:     InstrItinData <tc_711c805f, /*tc_2early*/
  2589:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2590:        InstrStage<1, [CVI_ST]>], [2, 2],
  2591:       [Hex_FWD, Hex_FWD]>,
  2592: 
  2593:     InstrItinData <tc_713b66bf, /*tc_1*/
  2594:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  2595:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2596: 
  2597:     InstrItinData <tc_7401744f, /*tc_2*/
  2598:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  2599:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2600: 
  2601:     InstrItinData <tc_7476d766, /*tc_3x*/
  2602:       [InstrStage<1, [SLOT3]>], [4, 2],
  2603:       [Hex_FWD, Hex_FWD]>,
  2604: 
  2605:     InstrItinData <tc_74a42bda, /*tc_ld*/
  2606:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  2607:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2608: 
  2609:     InstrItinData <tc_759e57be, /*tc_3stall*/
  2610:       [InstrStage<1, [SLOT2]>], [4, 1],
  2611:       [Hex_FWD, Hex_FWD]>,
  2612: 
  2613:     InstrItinData <tc_76bb5435, /*tc_ld*/
  2614:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  2615:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2616: 
  2617:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  2618:       [InstrStage<1, [SLOT2]>], [1],
  2619:       [Hex_FWD]>,
  2620: 
  2621:     InstrItinData <tc_77f94a5e, /*tc_st*/
  2622:       [InstrStage<1, [SLOT0]>], [],
  2623:       []>,
  2624: 
  2625:     InstrItinData <tc_788b1d09, /*tc_3x*/
  2626:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  2627:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2628: 
  2629:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  2630:       [InstrStage<1, [SLOT0]>], [],
  2631:       []>,
  2632: 
  2633:     InstrItinData <tc_7af3a37e, /*tc_st*/
  2634:       [InstrStage<1, [SLOT0]>], [1, 3],
  2635:       [Hex_FWD, Hex_FWD]>,
  2636: 
  2637:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  2638:       [InstrStage<1, [SLOT0], 0>,
  2639:        InstrStage<1, [CVI_ST]>], [3, 2],
  2640:       [Hex_FWD, Hex_FWD]>,
  2641: 
  2642:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  2643:       [InstrStage<1, [SLOT0]>], [3],
  2644:       [Hex_FWD]>,
  2645: 
  2646:     InstrItinData <tc_7c31e19a, /*tc_st*/
  2647:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  2648:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2649: 
  2650:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  2651:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  2652:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2653: 
  2654:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  2655:       [InstrStage<1, [SLOT3]>], [4, 1],
  2656:       [Hex_FWD, Hex_FWD]>,
  2657: 
  2658:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  2659:       [InstrStage<1, [SLOT3]>], [],
  2660:       []>,
  2661: 
  2662:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  2663:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  2664:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2665: 
  2666:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  2667:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  2668:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2669: 
  2670:     InstrItinData <tc_8035e91f, /*tc_st*/
  2671:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  2672:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2673: 
  2674:     InstrItinData <tc_822c3c68, /*tc_ld*/
  2675:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  2676:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2677: 
  2678:     InstrItinData <tc_829d8a86, /*tc_st*/
  2679:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  2680:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2681: 
  2682:     InstrItinData <tc_838c4d7a, /*tc_st*/
  2683:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  2684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2685: 
  2686:     InstrItinData <tc_84a7500d, /*tc_1*/
  2687:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  2688:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2689: 
  2690:     InstrItinData <tc_86173609, /*tc_1*/
  2691:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2],
  2692:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2693: 
  2694:     InstrItinData <tc_887d1bb7, /*tc_st*/
  2695:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 2],
  2696:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2697: 
  2698:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  2699:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  2700:       [Hex_FWD, Hex_FWD]>,
  2701: 
  2702:     InstrItinData <tc_8a825db2, /*tc_2*/
  2703:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2704:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2705: 
  2706:     InstrItinData <tc_8b5bd4f5, /*tc_1*/
  2707:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  2708:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2709: 
  2710:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  2711:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 2],
  2712:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2713: 
  2714:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  2715:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  2716:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2717: 
  2718:     InstrItinData <tc_9124c04f, /*tc_1*/
  2719:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  2720:       [Hex_FWD, Hex_FWD]>,
  2721: 
  2722:     InstrItinData <tc_92240447, /*tc_st*/
  2723:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  2724:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2725: 
  2726:     InstrItinData <tc_934753bb, /*tc_ld*/
  2727:       [InstrStage<1, [SLOT0]>], [4, 2, 2],
  2728:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2729: 
  2730:     InstrItinData <tc_937dd41c, /*tc_ld*/
  2731:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  2732:       []>,
  2733: 
  2734:     InstrItinData <tc_9406230a, /*tc_3x*/
  2735:       [InstrStage<1, [SLOT3]>], [2, 1],
  2736:       [Hex_FWD, Hex_FWD]>,
  2737: 
  2738:     InstrItinData <tc_95a33176, /*tc_2*/
  2739:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  2740:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2741: 
  2742:     InstrItinData <tc_95f43c5e, /*tc_3*/
  2743:       [InstrStage<1, [SLOT2], 0>,
  2744:        InstrStage<1, [CVI_ST]>], [1],
  2745:       [Hex_FWD]>,
  2746: 
  2747:     InstrItinData <tc_96ef76ef, /*tc_st*/
  2748:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  2749:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2750: 
  2751:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  2752:       [InstrStage<1, [SLOT0], 0>,
  2753:        InstrStage<1, [CVI_ST]>], [2, 3, 2],
  2754:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2755: 
  2756:     InstrItinData <tc_9783714b, /*tc_4x*/
  2757:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  2758:       [Hex_FWD, Hex_FWD]>,
  2759: 
  2760:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  2761:       [InstrStage<1, [SLOT2]>], [4, 1],
  2762:       [Hex_FWD, Hex_FWD]>,
  2763: 
  2764:     InstrItinData <tc_9b34f5e0, /*tc_2early*/
  2765:       [InstrStage<1, [SLOT2]>], [],
  2766:       []>,
  2767: 
  2768:     InstrItinData <tc_9b3c0462, /*tc_2*/
  2769:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2770:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2771: 
  2772:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  2773:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  2774:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2775: 
  2776:     InstrItinData <tc_9c52f549, /*tc_2early*/
  2777:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 1],
  2778:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2779: 
  2780:     InstrItinData <tc_9e27f2f9, /*tc_2early*/
  2781:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2782:        InstrStage<1, [CVI_ST]>], [1, 1, 2],
  2783:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2784: 
  2785:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  2786:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  2787:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2788: 
  2789:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  2790:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  2791:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2792: 
  2793:     InstrItinData <tc_9edefe01, /*tc_st*/
  2794:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 2],
  2795:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2796: 
  2797:     InstrItinData <tc_9f6cd987, /*tc_1*/
  2798:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  2799:       [Hex_FWD, Hex_FWD]>,
  2800: 
  2801:     InstrItinData <tc_a08b630b, /*tc_2*/
  2802:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2803:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2804: 
  2805:     InstrItinData <tc_a1297125, /*tc_2early*/
  2806:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 2],
  2807:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2808: 
  2809:     InstrItinData <tc_a154b476, /*tc_3x*/
  2810:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  2811:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2812: 
  2813:     InstrItinData <tc_a2b365d2, /*tc_st*/
  2814:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  2815:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2816: 
  2817:     InstrItinData <tc_a3070909, /*tc_3stall*/
  2818:       [InstrStage<1, [SLOT0]>], [1, 1],
  2819:       [Hex_FWD, Hex_FWD]>,
  2820: 
  2821:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  2822:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  2823:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2824: 
  2825:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  2826:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  2827:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2828: 
  2829:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  2830:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  2831:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2832: 
  2833:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  2834:       [InstrStage<1, [SLOT0], 0>,
  2835:        InstrStage<1, [CVI_ST]>], [],
  2836:       []>,
  2837: 
  2838:     InstrItinData <tc_a724463d, /*tc_3stall*/
  2839:       [InstrStage<1, [SLOT0]>], [4, 1],
  2840:       [Hex_FWD, Hex_FWD]>,
  2841: 
  2842:     InstrItinData <tc_a7a13fac, /*tc_2early*/
  2843:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  2844:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2845: 
  2846:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  2847:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  2848:       [Hex_FWD, Hex_FWD]>,
  2849: 
  2850:     InstrItinData <tc_a9edeffa, /*tc_st*/
  2851:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  2852:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2853: 
  2854:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  2855:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  2856:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2857: 
  2858:     InstrItinData <tc_ac65613f, /*tc_ld*/
  2859:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  2860:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2861: 
  2862:     InstrItinData <tc_addc37a8, /*tc_st*/
  2863:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  2864:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2865: 
  2866:     InstrItinData <tc_ae5babd7, /*tc_st*/
  2867:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  2868:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2869: 
  2870:     InstrItinData <tc_aee6250c, /*tc_ld*/
  2871:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  2872:       [Hex_FWD, Hex_FWD]>,
  2873: 
  2874:     InstrItinData <tc_af6af259, /*tc_ld*/
  2875:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  2876:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2877: 
  2878:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  2879:       [InstrStage<1, [SLOT0]>], [1],
  2880:       [Hex_FWD]>,
  2881: 
  2882:     InstrItinData <tc_b4dc7630, /*tc_st*/
  2883:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 2],
  2884:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2885: 
  2886:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  2887:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  2888:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2889: 
  2890:     InstrItinData <tc_b837298f, /*tc_1*/
  2891:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  2892:       []>,
  2893: 
  2894:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  2895:       [InstrStage<1, [SLOT2]>], [],
  2896:       []>,
  2897: 
  2898:     InstrItinData <tc_ba9255a6, /*tc_st*/
  2899:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 2],
  2900:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2901: 
  2902:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  2903:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  2904:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2905: 
  2906:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  2907:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  2908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2909: 
  2910:     InstrItinData <tc_bb831a7c, /*tc_2*/
  2911:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  2912:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2913: 
  2914:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  2915:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  2916:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2917: 
  2918:     InstrItinData <tc_c20701f0, /*tc_2*/
  2919:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  2920:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2921: 
  2922:     InstrItinData <tc_c21d7447, /*tc_3x*/
  2923:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  2924:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2925: 
  2926:     InstrItinData <tc_c57d9f39, /*tc_1*/
  2927:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  2928:       [Hex_FWD, Hex_FWD]>,
  2929: 
  2930:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  2931:       [InstrStage<1, [SLOT0], 0>,
  2932:        InstrStage<1, [CVI_ST]>], [],
  2933:       []>,
  2934: 
  2935:     InstrItinData <tc_ce59038e, /*tc_st*/
  2936:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  2937:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2938: 
  2939:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  2940:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  2941:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2942: 
  2943:     InstrItinData <tc_d03278fd, /*tc_st*/
  2944:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  2945:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  2946: 
  2947:     InstrItinData <tc_d234b61a, /*tc_st*/
  2948:       [InstrStage<1, [SLOT0]>], [1],
  2949:       [Hex_FWD]>,
  2950: 
  2951:     InstrItinData <tc_d33e5eee, /*tc_2early*/
  2952:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 2],
  2953:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2954: 
  2955:     InstrItinData <tc_d3632d88, /*tc_2*/
  2956:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  2957:       [Hex_FWD, Hex_FWD]>,
  2958: 
  2959:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  2960:       [InstrStage<1, [SLOT0]>], [1],
  2961:       [Hex_FWD]>,
  2962: 
  2963:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  2964:       [InstrStage<1, [SLOT2], 0>,
  2965:        InstrStage<1, [CVI_ST]>], [2],
  2966:       [Hex_FWD]>,
  2967: 
  2968:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  2969:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  2970:       [Hex_FWD, Hex_FWD]>,
  2971: 
  2972:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  2973:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  2974:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2975: 
  2976:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  2977:       [InstrStage<1, [SLOT3]>], [2, 1],
  2978:       [Hex_FWD, Hex_FWD]>,
  2979: 
  2980:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  2981:       [InstrStage<1, [SLOT3]>], [1],
  2982:       [Hex_FWD]>,
  2983: 
  2984:     InstrItinData <tc_db596beb, /*tc_3stall*/
  2985:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  2986:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  2987: 
  2988:     InstrItinData <tc_db96aa6b, /*tc_st*/
  2989:       [InstrStage<1, [SLOT0]>], [1],
  2990:       [Hex_FWD]>,
  2991: 
  2992:     InstrItinData <tc_dc51281d, /*tc_2early*/
  2993:       [InstrStage<1, [SLOT2], 0>,
  2994:        InstrStage<1, [CVI_ST]>], [2, 1],
  2995:       [Hex_FWD, Hex_FWD]>,
  2996: 
  2997:     InstrItinData <tc_decdde8a, /*tc_2early*/
  2998:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  2999:        InstrStage<1, [CVI_ST]>], [2],
  3000:       [Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 3001-3500 / 第 3001-3500 行

```tablegen
  3001: 
  3002:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  3003:       [InstrStage<1, [SLOT0], 0>,
  3004:        InstrStage<1, [CVI_ST]>], [3, 2, 2],
  3005:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3006: 
  3007:     InstrItinData <tc_e3d699e3, /*tc_2*/
  3008:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  3009:       [Hex_FWD, Hex_FWD]>,
  3010: 
  3011:     InstrItinData <tc_e60def48, /*tc_2early*/
  3012:       [InstrStage<1, [SLOT2], 0>,
  3013:        InstrStage<1, [CVI_ST]>], [1],
  3014:       [Hex_FWD]>,
  3015: 
  3016:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  3017:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  3018:       [Hex_FWD, Hex_FWD]>,
  3019: 
  3020:     InstrItinData <tc_ed03645c, /*tc_2early*/
  3021:       [InstrStage<1, [SLOT2], 0>,
  3022:        InstrStage<1, [CVI_ST]>], [3, 1],
  3023:       [Hex_FWD, Hex_FWD]>,
  3024: 
  3025:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  3026:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  3027:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3028: 
  3029:     InstrItinData <tc_eed07714, /*tc_ld*/
  3030:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  3031:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3032: 
  3033:     InstrItinData <tc_eeda4109, /*tc_2early*/
  3034:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3035:        InstrStage<1, [CVI_ST]>], [3, 2],
  3036:       [Hex_FWD, Hex_FWD]>,
  3037: 
  3038:     InstrItinData <tc_ef921005, /*tc_1*/
  3039:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  3040:       [Hex_FWD, Hex_FWD]>,
  3041: 
  3042:     InstrItinData <tc_f098b237, /*tc_1*/
  3043:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  3044:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3045: 
  3046:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  3047:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  3048:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3049: 
  3050:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  3051:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  3052:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3053: 
  3054:     InstrItinData <tc_f34c1c21, /*tc_2*/
  3055:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3056:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3057: 
  3058:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  3059:       [InstrStage<1, [SLOT0], 0>,
  3060:        InstrStage<1, [CVI_ST]>], [2],
  3061:       [Hex_FWD]>,
  3062: 
  3063:     InstrItinData <tc_f529831b, /*tc_st*/
  3064:       [InstrStage<1, [SLOT0]>], [3, 3, 1, 2, 3],
  3065:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3066: 
  3067:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  3068:       [InstrStage<1, [SLOT0], 0>,
  3069:        InstrStage<1, [CVI_ST]>], [3, 2, 2],
  3070:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3071: 
  3072:     InstrItinData <tc_f7569068, /*tc_4x*/
  3073:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  3074:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3075: 
  3076:     InstrItinData <tc_f97707c1, /*tc_1*/
  3077:       [InstrStage<1, [SLOT2], 0>,
  3078:        InstrStage<1, [CVI_ST]>], [2],
  3079:       [Hex_FWD]>,
  3080: 
  3081:     InstrItinData <tc_f999c66e, /*tc_2early*/
  3082:       [InstrStage<1, [SLOT2, SLOT3], 0>,
  3083:        InstrStage<1, [CVI_ST]>], [1, 2],
  3084:       [Hex_FWD, Hex_FWD]>,
  3085: 
  3086:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  3087:       [InstrStage<1, [SLOT3]>], [4, 2],
  3088:       [Hex_FWD, Hex_FWD]>,
  3089: 
  3090:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  3091:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  3092:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  3093:   ];
  3094: }
  3095: 
  3096: class DepScalarItinV62 {
  3097:   list<InstrItinData> DepScalarItinV62_list = [
  3098:     InstrItinData <tc_011e0e9d, /*tc_st*/
  3099:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  3100:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3101: 
  3102:     InstrItinData <tc_01d44cb2, /*tc_2*/
  3103:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3104:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3105: 
  3106:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  3107:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  3108:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3109: 
  3110:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  3111:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  3112:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3113: 
  3114:     InstrItinData <tc_0655b949, /*tc_st*/
  3115:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  3116:       [Hex_FWD, Hex_FWD]>,
  3117: 
  3118:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  3119:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  3120:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3121: 
  3122:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  3123:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  3124:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3125: 
  3126:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  3127:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  3128:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3129: 
  3130:     InstrItinData <tc_0ba0d5da, /*tc_2early*/
  3131:       [InstrStage<1, [SLOT2]>], [1],
  3132:       [Hex_FWD]>,
  3133: 
  3134:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  3135:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3136:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3137: 
  3138:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  3139:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  3140:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3141: 
  3142:     InstrItinData <tc_112d30d6, /*tc_1*/
  3143:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  3144:       [Hex_FWD]>,
  3145: 
  3146:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  3147:       [InstrStage<1, [SLOT0]>], [2],
  3148:       [Hex_FWD]>,
  3149: 
  3150:     InstrItinData <tc_1248597c, /*tc_3x*/
  3151:       [InstrStage<1, [SLOT3]>], [2, 2],
  3152:       [Hex_FWD, Hex_FWD]>,
  3153: 
  3154:     InstrItinData <tc_139ef484, /*tc_3stall*/
  3155:       [InstrStage<1, [SLOT2]>], [1, 1],
  3156:       [Hex_FWD, Hex_FWD]>,
  3157: 
  3158:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  3159:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  3160:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3161: 
  3162:     InstrItinData <tc_151bf368, /*tc_2early*/
  3163:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  3164:       [Hex_FWD, Hex_FWD]>,
  3165: 
  3166:     InstrItinData <tc_158aa3f7, /*tc_st*/
  3167:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  3168:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3169: 
  3170:     InstrItinData <tc_197dce51, /*tc_3x*/
  3171:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  3172:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3173: 
  3174:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  3175:       [InstrStage<1, [SLOT0]>], [3],
  3176:       [Hex_FWD]>,
  3177: 
  3178:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  3179:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  3180:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3181: 
  3182:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  3183:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  3184:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3185: 
  3186:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  3187:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  3188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3189: 
  3190:     InstrItinData <tc_1fcb8495, /*tc_2*/
  3191:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3192:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3193: 
  3194:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  3195:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 2],
  3196:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3197: 
  3198:     InstrItinData <tc_20131976, /*tc_1*/
  3199:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  3200:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3201: 
  3202:     InstrItinData <tc_2237d952, /*tc_ld*/
  3203:       [InstrStage<1, [SLOT0]>], [1, 2],
  3204:       [Hex_FWD, Hex_FWD]>,
  3205: 
  3206:     InstrItinData <tc_23708a21, /*tc_2early*/
  3207:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  3208:       []>,
  3209: 
  3210:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  3211:       [InstrStage<1, [SLOT0]>], [4, 1],
  3212:       [Hex_FWD, Hex_FWD]>,
  3213: 
  3214:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  3215:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  3216:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3217: 
  3218:     InstrItinData <tc_24f426ab, /*tc_2early*/
  3219:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [1, 2, 2],
  3220:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3221: 
  3222:     InstrItinData <tc_27106296, /*tc_3x*/
  3223:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  3224:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3225: 
  3226:     InstrItinData <tc_280f7fe1, /*tc_st*/
  3227:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 2],
  3228:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3229: 
  3230:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  3231:       [InstrStage<1, [SLOT3]>], [1, 1],
  3232:       [Hex_FWD, Hex_FWD]>,
  3233: 
  3234:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  3235:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  3236:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3237: 
  3238:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  3239:       [InstrStage<1, [SLOT3]>], [1],
  3240:       [Hex_FWD]>,
  3241: 
  3242:     InstrItinData <tc_2f573607, /*tc_2early*/
  3243:       [InstrStage<1, [SLOT2]>], [2, 1],
  3244:       [Hex_FWD, Hex_FWD]>,
  3245: 
  3246:     InstrItinData <tc_33e7e673, /*tc_2early*/
  3247:       [InstrStage<1, [SLOT2]>], [],
  3248:       []>,
  3249: 
  3250:     InstrItinData <tc_362b0be2, /*tc_3*/
  3251:       [InstrStage<1, [SLOT2]>], [1],
  3252:       [Hex_FWD]>,
  3253: 
  3254:     InstrItinData <tc_38382228, /*tc_3x*/
  3255:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  3256:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3257: 
  3258:     InstrItinData <tc_388f9897, /*tc_1*/
  3259:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  3260:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3261: 
  3262:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  3263:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
  3264:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3265: 
  3266:     InstrItinData <tc_3d14a17b, /*tc_1*/
  3267:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  3268:       [Hex_FWD, Hex_FWD]>,
  3269: 
  3270:     InstrItinData <tc_3edca78f, /*tc_2*/
  3271:       [InstrStage<1, [SLOT3]>], [4, 2],
  3272:       [Hex_FWD, Hex_FWD]>,
  3273: 
  3274:     InstrItinData <tc_3fbf1042, /*tc_1*/
  3275:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  3276:       [Hex_FWD]>,
  3277: 
  3278:     InstrItinData <tc_407e96f9, /*tc_1*/
  3279:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  3280:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3281: 
  3282:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  3283:       [InstrStage<1, [SLOT0]>], [3, 2],
  3284:       [Hex_FWD, Hex_FWD]>,
  3285: 
  3286:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  3287:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  3288:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3289: 
  3290:     InstrItinData <tc_42ff66ba, /*tc_2early*/
  3291:       [InstrStage<1, [SLOT2]>], [2, 1],
  3292:       [Hex_FWD, Hex_FWD]>,
  3293: 
  3294:     InstrItinData <tc_442395f3, /*tc_1*/
  3295:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2, 2],
  3296:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3297: 
  3298:     InstrItinData <tc_449acf79, /*tc_st*/
  3299:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 3, 1, 2, 2],
  3300:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3301: 
  3302:     InstrItinData <tc_44d5a428, /*tc_st*/
  3303:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  3304:       [Hex_FWD, Hex_FWD]>,
  3305: 
  3306:     InstrItinData <tc_44fffc58, /*tc_3*/
  3307:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  3308:       [Hex_FWD]>,
  3309: 
  3310:     InstrItinData <tc_45791fb8, /*tc_ld*/
  3311:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  3312:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3313: 
  3314:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  3315:       [InstrStage<1, [SLOT2]>], [2],
  3316:       [Hex_FWD]>,
  3317: 
  3318:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  3319:       [InstrStage<1, [SLOT3]>], [4, 1],
  3320:       [Hex_FWD, Hex_FWD]>,
  3321: 
  3322:     InstrItinData <tc_49fdfd4b, /*tc_3x*/
  3323:       [InstrStage<1, [SLOT3]>], [4, 1],
  3324:       [Hex_FWD, Hex_FWD]>,
  3325: 
  3326:     InstrItinData <tc_4a55d03c, /*tc_2early*/
  3327:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 1],
  3328:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3329: 
  3330:     InstrItinData <tc_4abdbdc6, /*tc_3stall*/
  3331:       [InstrStage<1, [SLOT3]>], [2, 2],
  3332:       [Hex_FWD, Hex_FWD]>,
  3333: 
  3334:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  3335:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  3336:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3337: 
  3338:     InstrItinData <tc_4bf903b0, /*tc_st*/
  3339:       [InstrStage<1, [SLOT0]>], [3],
  3340:       [Hex_FWD]>,
  3341: 
  3342:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  3343:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  3344:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3345: 
  3346:     InstrItinData <tc_512b1653, /*tc_st*/
  3347:       [InstrStage<1, [SLOT0]>], [1, 2],
  3348:       [Hex_FWD, Hex_FWD]>,
  3349: 
  3350:     InstrItinData <tc_53c851ab, /*tc_2early*/
  3351:       [InstrStage<1, [SLOT2]>], [3, 2, 2],
  3352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3353: 
  3354:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  3355:       [InstrStage<1, [SLOT3]>], [1],
  3356:       [Hex_FWD]>,
  3357: 
  3358:     InstrItinData <tc_5502c366, /*tc_1*/
  3359:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  3360:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3361: 
  3362:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  3363:       [InstrStage<1, [SLOT3]>], [],
  3364:       []>,
  3365: 
  3366:     InstrItinData <tc_556f6577, /*tc_3x*/
  3367:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  3368:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3369: 
  3370:     InstrItinData <tc_55a9a350, /*tc_st*/
  3371:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  3372:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3373: 
  3374:     InstrItinData <tc_55b33fda, /*tc_2early*/
  3375:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1],
  3376:       [Hex_FWD, Hex_FWD]>,
  3377: 
  3378:     InstrItinData <tc_56a124a7, /*tc_2early*/
  3379:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  3380:       [Hex_FWD, Hex_FWD]>,
  3381: 
  3382:     InstrItinData <tc_57a55b54, /*tc_2early*/
  3383:       [InstrStage<1, [SLOT3]>], [1, 2],
  3384:       [Hex_FWD, Hex_FWD]>,
  3385: 
  3386:     InstrItinData <tc_5944960d, /*tc_ld*/
  3387:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  3388:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3389: 
  3390:     InstrItinData <tc_59a7822c, /*tc_2early*/
  3391:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  3392:       [Hex_FWD, Hex_FWD]>,
  3393: 
  3394:     InstrItinData <tc_5a222e89, /*tc_2early*/
  3395:       [InstrStage<1, [SLOT2]>], [1, 1],
  3396:       [Hex_FWD, Hex_FWD]>,
  3397: 
  3398:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  3399:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  3400:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3401: 
  3402:     InstrItinData <tc_5b347363, /*tc_1*/
  3403:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  3404:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3405: 
  3406:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  3407:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  3408:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3409: 
  3410:     InstrItinData <tc_5da50c4b, /*tc_1*/
  3411:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  3412:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3413: 
  3414:     InstrItinData <tc_5deb5e47, /*tc_st*/
  3415:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  3416:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3417: 
  3418:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  3419:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  3420:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3421: 
  3422:     InstrItinData <tc_5f2afaf7, /*tc_ld*/
  3423:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 3, 1, 2],
  3424:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3425: 
  3426:     InstrItinData <tc_60e324ff, /*tc_2early*/
  3427:       [InstrStage<1, [SLOT2]>], [1],
  3428:       [Hex_FWD]>,
  3429: 
  3430:     InstrItinData <tc_63567288, /*tc_1*/
  3431:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  3432:       [Hex_FWD]>,
  3433: 
  3434:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  3435:       [InstrStage<1, [SLOT0]>], [4, 1],
  3436:       [Hex_FWD, Hex_FWD]>,
  3437: 
  3438:     InstrItinData <tc_651cbe02, /*tc_2early*/
  3439:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  3440:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3441: 
  3442:     InstrItinData <tc_65279839, /*tc_2*/
  3443:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  3444:       [Hex_FWD, Hex_FWD]>,
  3445: 
  3446:     InstrItinData <tc_65cbd974, /*tc_st*/
  3447:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  3448:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3449: 
  3450:     InstrItinData <tc_69bfb303, /*tc_3*/
  3451:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  3452:       [Hex_FWD, Hex_FWD]>,
  3453: 
  3454:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  3455:       [InstrStage<1, [SLOT3]>], [4, 1],
  3456:       [Hex_FWD, Hex_FWD]>,
  3457: 
  3458:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  3459:       [InstrStage<1, [SLOT3]>], [4, 1],
  3460:       [Hex_FWD, Hex_FWD]>,
  3461: 
  3462:     InstrItinData <tc_6d861a95, /*tc_3stall*/
  3463:       [InstrStage<1, [SLOT3]>], [2, 1],
  3464:       [Hex_FWD, Hex_FWD]>,
  3465: 
  3466:     InstrItinData <tc_6e20402a, /*tc_st*/
  3467:       [InstrStage<1, [SLOT0]>], [2, 3],
  3468:       [Hex_FWD, Hex_FWD]>,
  3469: 
  3470:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  3471:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  3472:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3473: 
  3474:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  3475:       [InstrStage<1, [SLOT0]>], [1, 1],
  3476:       [Hex_FWD, Hex_FWD]>,
  3477: 
  3478:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  3479:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  3480:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3481: 
  3482:     InstrItinData <tc_711c805f, /*tc_2early*/
  3483:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  3484:       [Hex_FWD, Hex_FWD]>,
  3485: 
  3486:     InstrItinData <tc_713b66bf, /*tc_1*/
  3487:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  3488:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3489: 
  3490:     InstrItinData <tc_7401744f, /*tc_2*/
  3491:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  3492:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3493: 
  3494:     InstrItinData <tc_7476d766, /*tc_3x*/
  3495:       [InstrStage<1, [SLOT3]>], [4, 2],
  3496:       [Hex_FWD, Hex_FWD]>,
  3497: 
  3498:     InstrItinData <tc_74a42bda, /*tc_ld*/
  3499:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  3500:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV62, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV62; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV62 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV62 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 3501-4000 / 第 3501-4000 行

```tablegen
  3501: 
  3502:     InstrItinData <tc_759e57be, /*tc_3stall*/
  3503:       [InstrStage<1, [SLOT2]>], [4, 1],
  3504:       [Hex_FWD, Hex_FWD]>,
  3505: 
  3506:     InstrItinData <tc_76bb5435, /*tc_ld*/
  3507:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  3508:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3509: 
  3510:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  3511:       [InstrStage<1, [SLOT2]>], [1],
  3512:       [Hex_FWD]>,
  3513: 
  3514:     InstrItinData <tc_77f94a5e, /*tc_st*/
  3515:       [InstrStage<1, [SLOT0]>], [],
  3516:       []>,
  3517: 
  3518:     InstrItinData <tc_788b1d09, /*tc_3x*/
  3519:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  3520:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3521: 
  3522:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  3523:       [InstrStage<1, [SLOT0]>], [],
  3524:       []>,
  3525: 
  3526:     InstrItinData <tc_7af3a37e, /*tc_st*/
  3527:       [InstrStage<1, [SLOT0]>], [1, 3],
  3528:       [Hex_FWD, Hex_FWD]>,
  3529: 
  3530:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  3531:       [InstrStage<1, [SLOT0]>], [3, 2],
  3532:       [Hex_FWD, Hex_FWD]>,
  3533: 
  3534:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  3535:       [InstrStage<1, [SLOT0]>], [3],
  3536:       [Hex_FWD]>,
  3537: 
  3538:     InstrItinData <tc_7c31e19a, /*tc_st*/
  3539:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  3540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3541: 
  3542:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  3543:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  3544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3545: 
  3546:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  3547:       [InstrStage<1, [SLOT3]>], [4, 1],
  3548:       [Hex_FWD, Hex_FWD]>,
  3549: 
  3550:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  3551:       [InstrStage<1, [SLOT3]>], [],
  3552:       []>,
  3553: 
  3554:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  3555:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  3556:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3557: 
  3558:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  3559:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  3560:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3561: 
  3562:     InstrItinData <tc_8035e91f, /*tc_st*/
  3563:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  3564:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3565: 
  3566:     InstrItinData <tc_822c3c68, /*tc_ld*/
  3567:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  3568:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3569: 
  3570:     InstrItinData <tc_829d8a86, /*tc_st*/
  3571:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  3572:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3573: 
  3574:     InstrItinData <tc_838c4d7a, /*tc_st*/
  3575:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  3576:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3577: 
  3578:     InstrItinData <tc_84a7500d, /*tc_1*/
  3579:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  3580:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3581: 
  3582:     InstrItinData <tc_86173609, /*tc_1*/
  3583:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 3, 2],
  3584:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3585: 
  3586:     InstrItinData <tc_887d1bb7, /*tc_st*/
  3587:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 2],
  3588:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3589: 
  3590:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  3591:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  3592:       [Hex_FWD, Hex_FWD]>,
  3593: 
  3594:     InstrItinData <tc_8a825db2, /*tc_2*/
  3595:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3596:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3597: 
  3598:     InstrItinData <tc_8b5bd4f5, /*tc_1*/
  3599:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  3600:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3601: 
  3602:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  3603:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 2],
  3604:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3605: 
  3606:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  3607:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  3608:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3609: 
  3610:     InstrItinData <tc_9124c04f, /*tc_1*/
  3611:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  3612:       [Hex_FWD, Hex_FWD]>,
  3613: 
  3614:     InstrItinData <tc_92240447, /*tc_st*/
  3615:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  3616:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3617: 
  3618:     InstrItinData <tc_934753bb, /*tc_ld*/
  3619:       [InstrStage<1, [SLOT0]>], [4, 2, 2],
  3620:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3621: 
  3622:     InstrItinData <tc_937dd41c, /*tc_ld*/
  3623:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  3624:       []>,
  3625: 
  3626:     InstrItinData <tc_9406230a, /*tc_3x*/
  3627:       [InstrStage<1, [SLOT3]>], [2, 1],
  3628:       [Hex_FWD, Hex_FWD]>,
  3629: 
  3630:     InstrItinData <tc_95a33176, /*tc_2*/
  3631:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  3632:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3633: 
  3634:     InstrItinData <tc_95f43c5e, /*tc_3*/
  3635:       [InstrStage<1, [SLOT2]>], [1],
  3636:       [Hex_FWD]>,
  3637: 
  3638:     InstrItinData <tc_96ef76ef, /*tc_st*/
  3639:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  3640:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3641: 
  3642:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  3643:       [InstrStage<1, [SLOT0]>], [2, 3, 2],
  3644:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3645: 
  3646:     InstrItinData <tc_9783714b, /*tc_4x*/
  3647:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  3648:       [Hex_FWD, Hex_FWD]>,
  3649: 
  3650:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  3651:       [InstrStage<1, [SLOT2]>], [4, 1],
  3652:       [Hex_FWD, Hex_FWD]>,
  3653: 
  3654:     InstrItinData <tc_9b34f5e0, /*tc_2early*/
  3655:       [InstrStage<1, [SLOT2]>], [],
  3656:       []>,
  3657: 
  3658:     InstrItinData <tc_9b3c0462, /*tc_2*/
  3659:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3660:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3661: 
  3662:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  3663:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  3664:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3665: 
  3666:     InstrItinData <tc_9c52f549, /*tc_2early*/
  3667:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 1],
  3668:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3669: 
  3670:     InstrItinData <tc_9e27f2f9, /*tc_2early*/
  3671:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [1, 1, 2],
  3672:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3673: 
  3674:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  3675:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  3676:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3677: 
  3678:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  3679:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  3680:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3681: 
  3682:     InstrItinData <tc_9edefe01, /*tc_st*/
  3683:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 2],
  3684:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3685: 
  3686:     InstrItinData <tc_9f6cd987, /*tc_1*/
  3687:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  3688:       [Hex_FWD, Hex_FWD]>,
  3689: 
  3690:     InstrItinData <tc_a08b630b, /*tc_2*/
  3691:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3692:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3693: 
  3694:     InstrItinData <tc_a1297125, /*tc_2early*/
  3695:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 1, 2],
  3696:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3697: 
  3698:     InstrItinData <tc_a154b476, /*tc_3x*/
  3699:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  3700:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3701: 
  3702:     InstrItinData <tc_a2b365d2, /*tc_st*/
  3703:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  3704:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3705: 
  3706:     InstrItinData <tc_a3070909, /*tc_3stall*/
  3707:       [InstrStage<1, [SLOT0]>], [1, 1],
  3708:       [Hex_FWD, Hex_FWD]>,
  3709: 
  3710:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  3711:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  3712:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3713: 
  3714:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  3715:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  3716:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3717: 
  3718:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  3719:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  3720:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3721: 
  3722:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  3723:       [InstrStage<1, [SLOT0]>], [],
  3724:       []>,
  3725: 
  3726:     InstrItinData <tc_a724463d, /*tc_3stall*/
  3727:       [InstrStage<1, [SLOT0]>], [4, 1],
  3728:       [Hex_FWD, Hex_FWD]>,
  3729: 
  3730:     InstrItinData <tc_a7a13fac, /*tc_2early*/
  3731:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  3732:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3733: 
  3734:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  3735:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  3736:       [Hex_FWD, Hex_FWD]>,
  3737: 
  3738:     InstrItinData <tc_a9edeffa, /*tc_st*/
  3739:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  3740:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3741: 
  3742:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  3743:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  3744:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3745: 
  3746:     InstrItinData <tc_ac65613f, /*tc_ld*/
  3747:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  3748:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3749: 
  3750:     InstrItinData <tc_addc37a8, /*tc_st*/
  3751:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  3752:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3753: 
  3754:     InstrItinData <tc_ae5babd7, /*tc_st*/
  3755:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  3756:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3757: 
  3758:     InstrItinData <tc_aee6250c, /*tc_ld*/
  3759:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  3760:       [Hex_FWD, Hex_FWD]>,
  3761: 
  3762:     InstrItinData <tc_af6af259, /*tc_ld*/
  3763:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  3764:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3765: 
  3766:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  3767:       [InstrStage<1, [SLOT0]>], [1],
  3768:       [Hex_FWD]>,
  3769: 
  3770:     InstrItinData <tc_b4dc7630, /*tc_st*/
  3771:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 2],
  3772:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3773: 
  3774:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  3775:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  3776:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3777: 
  3778:     InstrItinData <tc_b837298f, /*tc_1*/
  3779:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  3780:       []>,
  3781: 
  3782:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  3783:       [InstrStage<1, [SLOT2]>], [],
  3784:       []>,
  3785: 
  3786:     InstrItinData <tc_ba9255a6, /*tc_st*/
  3787:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 2],
  3788:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3789: 
  3790:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  3791:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  3792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3793: 
  3794:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  3795:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  3796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3797: 
  3798:     InstrItinData <tc_bb831a7c, /*tc_2*/
  3799:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  3800:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3801: 
  3802:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  3803:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  3804:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3805: 
  3806:     InstrItinData <tc_c20701f0, /*tc_2*/
  3807:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3808:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3809: 
  3810:     InstrItinData <tc_c21d7447, /*tc_3x*/
  3811:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  3812:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3813: 
  3814:     InstrItinData <tc_c57d9f39, /*tc_1*/
  3815:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  3816:       [Hex_FWD, Hex_FWD]>,
  3817: 
  3818:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  3819:       [InstrStage<1, [SLOT0]>], [],
  3820:       []>,
  3821: 
  3822:     InstrItinData <tc_ce59038e, /*tc_st*/
  3823:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  3824:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3825: 
  3826:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  3827:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  3828:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3829: 
  3830:     InstrItinData <tc_d03278fd, /*tc_st*/
  3831:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  3832:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3833: 
  3834:     InstrItinData <tc_d234b61a, /*tc_st*/
  3835:       [InstrStage<1, [SLOT0]>], [1],
  3836:       [Hex_FWD]>,
  3837: 
  3838:     InstrItinData <tc_d33e5eee, /*tc_2early*/
  3839:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 1, 2],
  3840:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3841: 
  3842:     InstrItinData <tc_d3632d88, /*tc_2*/
  3843:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  3844:       [Hex_FWD, Hex_FWD]>,
  3845: 
  3846:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  3847:       [InstrStage<1, [SLOT0]>], [1],
  3848:       [Hex_FWD]>,
  3849: 
  3850:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  3851:       [InstrStage<1, [SLOT2]>], [2],
  3852:       [Hex_FWD]>,
  3853: 
  3854:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  3855:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  3856:       [Hex_FWD, Hex_FWD]>,
  3857: 
  3858:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  3859:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  3860:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3861: 
  3862:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  3863:       [InstrStage<1, [SLOT3]>], [2, 1],
  3864:       [Hex_FWD, Hex_FWD]>,
  3865: 
  3866:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  3867:       [InstrStage<1, [SLOT3]>], [1],
  3868:       [Hex_FWD]>,
  3869: 
  3870:     InstrItinData <tc_db596beb, /*tc_3x*/
  3871:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  3872:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3873: 
  3874:     InstrItinData <tc_db96aa6b, /*tc_st*/
  3875:       [InstrStage<1, [SLOT0]>], [1],
  3876:       [Hex_FWD]>,
  3877: 
  3878:     InstrItinData <tc_dc51281d, /*tc_3*/
  3879:       [InstrStage<1, [SLOT2]>], [2, 1],
  3880:       [Hex_FWD, Hex_FWD]>,
  3881: 
  3882:     InstrItinData <tc_decdde8a, /*tc_2early*/
  3883:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  3884:       [Hex_FWD]>,
  3885: 
  3886:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  3887:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  3888:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3889: 
  3890:     InstrItinData <tc_e3d699e3, /*tc_2*/
  3891:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  3892:       [Hex_FWD, Hex_FWD]>,
  3893: 
  3894:     InstrItinData <tc_e60def48, /*tc_2early*/
  3895:       [InstrStage<1, [SLOT2]>], [1],
  3896:       [Hex_FWD]>,
  3897: 
  3898:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  3899:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  3900:       [Hex_FWD, Hex_FWD]>,
  3901: 
  3902:     InstrItinData <tc_ed03645c, /*tc_2early*/
  3903:       [InstrStage<1, [SLOT2]>], [3, 1],
  3904:       [Hex_FWD, Hex_FWD]>,
  3905: 
  3906:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  3907:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  3908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3909: 
  3910:     InstrItinData <tc_eed07714, /*tc_ld*/
  3911:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  3912:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3913: 
  3914:     InstrItinData <tc_eeda4109, /*tc_2early*/
  3915:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  3916:       [Hex_FWD, Hex_FWD]>,
  3917: 
  3918:     InstrItinData <tc_ef921005, /*tc_1*/
  3919:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  3920:       [Hex_FWD, Hex_FWD]>,
  3921: 
  3922:     InstrItinData <tc_f098b237, /*tc_1*/
  3923:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  3924:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3925: 
  3926:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  3927:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  3928:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3929: 
  3930:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  3931:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  3932:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3933: 
  3934:     InstrItinData <tc_f34c1c21, /*tc_2*/
  3935:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3936:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3937: 
  3938:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  3939:       [InstrStage<1, [SLOT0]>], [2],
  3940:       [Hex_FWD]>,
  3941: 
  3942:     InstrItinData <tc_f529831b, /*tc_st*/
  3943:       [InstrStage<1, [SLOT0]>], [3, 3, 1, 2, 3],
  3944:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3945: 
  3946:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  3947:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  3948:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3949: 
  3950:     InstrItinData <tc_f7569068, /*tc_4x*/
  3951:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  3952:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3953: 
  3954:     InstrItinData <tc_f97707c1, /*tc_1*/
  3955:       [InstrStage<1, [SLOT2]>], [2],
  3956:       [Hex_FWD]>,
  3957: 
  3958:     InstrItinData <tc_f999c66e, /*tc_2early*/
  3959:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [1, 2],
  3960:       [Hex_FWD, Hex_FWD]>,
  3961: 
  3962:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  3963:       [InstrStage<1, [SLOT3]>], [4, 2],
  3964:       [Hex_FWD, Hex_FWD]>,
  3965: 
  3966:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  3967:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  3968:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  3969:   ];
  3970: }
  3971: 
  3972: class DepScalarItinV65 {
  3973:   list<InstrItinData> DepScalarItinV65_list = [
  3974:     InstrItinData <tc_011e0e9d, /*tc_st*/
  3975:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  3976:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3977: 
  3978:     InstrItinData <tc_01d44cb2, /*tc_2*/
  3979:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  3980:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3981: 
  3982:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  3983:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  3984:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3985: 
  3986:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  3987:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  3988:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  3989: 
  3990:     InstrItinData <tc_0655b949, /*tc_st*/
  3991:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  3992:       [Hex_FWD, Hex_FWD]>,
  3993: 
  3994:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  3995:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  3996:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  3997: 
  3998:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  3999:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  4000:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV65, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV65; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV65 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV65 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 4001-4500 / 第 4001-4500 行

```tablegen
  4001: 
  4002:     InstrItinData <tc_0a43be35, /*tc_3x*/
  4003:       [InstrStage<1, [SLOT3]>], [1],
  4004:       [Hex_FWD]>,
  4005: 
  4006:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  4007:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  4008:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4009: 
  4010:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
  4011:       [InstrStage<1, [SLOT2]>], [1],
  4012:       [Hex_FWD]>,
  4013: 
  4014:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  4015:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4016:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4017: 
  4018:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  4019:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  4020:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4021: 
  4022:     InstrItinData <tc_112d30d6, /*tc_1*/
  4023:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  4024:       [Hex_FWD]>,
  4025: 
  4026:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  4027:       [InstrStage<1, [SLOT0]>], [2],
  4028:       [Hex_FWD]>,
  4029: 
  4030:     InstrItinData <tc_1248597c, /*tc_3x*/
  4031:       [InstrStage<1, [SLOT3]>], [2, 2],
  4032:       [Hex_FWD, Hex_FWD]>,
  4033: 
  4034:     InstrItinData <tc_139ef484, /*tc_3stall*/
  4035:       [InstrStage<1, [SLOT2]>], [1, 1],
  4036:       [Hex_FWD, Hex_FWD]>,
  4037: 
  4038:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  4039:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
  4040:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4041: 
  4042:     InstrItinData <tc_151bf368, /*tc_1*/
  4043:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  4044:       [Hex_FWD, Hex_FWD]>,
  4045: 
  4046:     InstrItinData <tc_158aa3f7, /*tc_st*/
  4047:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  4048:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4049: 
  4050:     InstrItinData <tc_197dce51, /*tc_3x*/
  4051:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  4052:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4053: 
  4054:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  4055:       [InstrStage<1, [SLOT0]>], [3],
  4056:       [Hex_FWD]>,
  4057: 
  4058:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  4059:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  4060:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4061: 
  4062:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  4063:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  4064:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4065: 
  4066:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  4067:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  4068:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4069: 
  4070:     InstrItinData <tc_1fcb8495, /*tc_2*/
  4071:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4072:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4073: 
  4074:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  4075:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 2],
  4076:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4077: 
  4078:     InstrItinData <tc_20131976, /*tc_1*/
  4079:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4080:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4081: 
  4082:     InstrItinData <tc_2237d952, /*tc_ld*/
  4083:       [InstrStage<1, [SLOT0]>], [1, 2],
  4084:       [Hex_FWD, Hex_FWD]>,
  4085: 
  4086:     InstrItinData <tc_23708a21, /*tc_1*/
  4087:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  4088:       []>,
  4089: 
  4090:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  4091:       [InstrStage<1, [SLOT0]>], [4, 1],
  4092:       [Hex_FWD, Hex_FWD]>,
  4093: 
  4094:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  4095:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  4096:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4097: 
  4098:     InstrItinData <tc_24f426ab, /*tc_1*/
  4099:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  4100:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4101: 
  4102:     InstrItinData <tc_27106296, /*tc_3x*/
  4103:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  4104:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4105: 
  4106:     InstrItinData <tc_280f7fe1, /*tc_st*/
  4107:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 2],
  4108:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4109: 
  4110:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  4111:       [InstrStage<1, [SLOT3]>], [1, 1],
  4112:       [Hex_FWD, Hex_FWD]>,
  4113: 
  4114:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  4115:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  4116:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4117: 
  4118:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  4119:       [InstrStage<1, [SLOT3]>], [1],
  4120:       [Hex_FWD]>,
  4121: 
  4122:     InstrItinData <tc_2f573607, /*tc_1*/
  4123:       [InstrStage<1, [SLOT2]>], [2, 2],
  4124:       [Hex_FWD, Hex_FWD]>,
  4125: 
  4126:     InstrItinData <tc_33e7e673, /*tc_2early*/
  4127:       [InstrStage<1, [SLOT2]>], [],
  4128:       []>,
  4129: 
  4130:     InstrItinData <tc_362b0be2, /*tc_3*/
  4131:       [InstrStage<1, [SLOT2]>], [1],
  4132:       [Hex_FWD]>,
  4133: 
  4134:     InstrItinData <tc_38382228, /*tc_3x*/
  4135:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  4136:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4137: 
  4138:     InstrItinData <tc_388f9897, /*tc_1*/
  4139:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  4140:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4141: 
  4142:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  4143:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
  4144:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4145: 
  4146:     InstrItinData <tc_3d14a17b, /*tc_1*/
  4147:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  4148:       [Hex_FWD, Hex_FWD]>,
  4149: 
  4150:     InstrItinData <tc_3edca78f, /*tc_2*/
  4151:       [InstrStage<1, [SLOT3]>], [4, 2],
  4152:       [Hex_FWD, Hex_FWD]>,
  4153: 
  4154:     InstrItinData <tc_3fbf1042, /*tc_1*/
  4155:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  4156:       [Hex_FWD]>,
  4157: 
  4158:     InstrItinData <tc_407e96f9, /*tc_1*/
  4159:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4160:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4161: 
  4162:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  4163:       [InstrStage<1, [SLOT0]>], [3, 1],
  4164:       [Hex_FWD, Hex_FWD]>,
  4165: 
  4166:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  4167:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  4168:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4169: 
  4170:     InstrItinData <tc_42ff66ba, /*tc_1*/
  4171:       [InstrStage<1, [SLOT2]>], [2, 2],
  4172:       [Hex_FWD, Hex_FWD]>,
  4173: 
  4174:     InstrItinData <tc_442395f3, /*tc_2latepred*/
  4175:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
  4176:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4177: 
  4178:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
  4179:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
  4180:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4181: 
  4182:     InstrItinData <tc_44d5a428, /*tc_st*/
  4183:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  4184:       [Hex_FWD, Hex_FWD]>,
  4185: 
  4186:     InstrItinData <tc_44fffc58, /*tc_3*/
  4187:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  4188:       [Hex_FWD]>,
  4189: 
  4190:     InstrItinData <tc_45791fb8, /*tc_ld*/
  4191:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  4192:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4193: 
  4194:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  4195:       [InstrStage<1, [SLOT2]>], [2],
  4196:       [Hex_FWD]>,
  4197: 
  4198:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  4199:       [InstrStage<1, [SLOT3]>], [4, 1],
  4200:       [Hex_FWD, Hex_FWD]>,
  4201: 
  4202:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
  4203:       [InstrStage<1, [SLOT3]>], [4, 1],
  4204:       [Hex_FWD, Hex_FWD]>,
  4205: 
  4206:     InstrItinData <tc_4a55d03c, /*tc_1*/
  4207:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4208:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4209: 
  4210:     InstrItinData <tc_4abdbdc6, /*tc_3stall*/
  4211:       [InstrStage<1, [SLOT3]>], [2, 2],
  4212:       [Hex_FWD, Hex_FWD]>,
  4213: 
  4214:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  4215:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  4216:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4217: 
  4218:     InstrItinData <tc_4bf903b0, /*tc_st*/
  4219:       [InstrStage<1, [SLOT0]>], [3],
  4220:       [Hex_FWD]>,
  4221: 
  4222:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  4223:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  4224:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4225: 
  4226:     InstrItinData <tc_512b1653, /*tc_st*/
  4227:       [InstrStage<1, [SLOT0]>], [1, 2],
  4228:       [Hex_FWD, Hex_FWD]>,
  4229: 
  4230:     InstrItinData <tc_53c851ab, /*tc_3stall*/
  4231:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
  4232:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4233: 
  4234:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  4235:       [InstrStage<1, [SLOT3]>], [1],
  4236:       [Hex_FWD]>,
  4237: 
  4238:     InstrItinData <tc_5502c366, /*tc_1*/
  4239:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4240:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4241: 
  4242:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  4243:       [InstrStage<1, [SLOT3]>], [],
  4244:       []>,
  4245: 
  4246:     InstrItinData <tc_556f6577, /*tc_3x*/
  4247:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  4248:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4249: 
  4250:     InstrItinData <tc_55a9a350, /*tc_st*/
  4251:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  4252:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4253: 
  4254:     InstrItinData <tc_55b33fda, /*tc_1*/
  4255:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  4256:       [Hex_FWD, Hex_FWD]>,
  4257: 
  4258:     InstrItinData <tc_56a124a7, /*tc_1*/
  4259:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  4260:       [Hex_FWD, Hex_FWD]>,
  4261: 
  4262:     InstrItinData <tc_57a55b54, /*tc_1*/
  4263:       [InstrStage<1, [SLOT3]>], [2, 2],
  4264:       [Hex_FWD, Hex_FWD]>,
  4265: 
  4266:     InstrItinData <tc_5944960d, /*tc_ld*/
  4267:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  4268:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4269: 
  4270:     InstrItinData <tc_59a7822c, /*tc_1*/
  4271:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  4272:       [Hex_FWD, Hex_FWD]>,
  4273: 
  4274:     InstrItinData <tc_5a222e89, /*tc_2early*/
  4275:       [InstrStage<1, [SLOT2]>], [1, 1],
  4276:       [Hex_FWD, Hex_FWD]>,
  4277: 
  4278:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  4279:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  4280:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4281: 
  4282:     InstrItinData <tc_5b347363, /*tc_1*/
  4283:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  4284:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4285: 
  4286:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  4287:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  4288:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4289: 
  4290:     InstrItinData <tc_5da50c4b, /*tc_1*/
  4291:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4292:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4293: 
  4294:     InstrItinData <tc_5deb5e47, /*tc_st*/
  4295:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  4296:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4297: 
  4298:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  4299:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  4300:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4301: 
  4302:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
  4303:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
  4304:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4305: 
  4306:     InstrItinData <tc_60e324ff, /*tc_1*/
  4307:       [InstrStage<1, [SLOT2]>], [2],
  4308:       [Hex_FWD]>,
  4309: 
  4310:     InstrItinData <tc_63567288, /*tc_2latepred*/
  4311:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
  4312:       [Hex_FWD]>,
  4313: 
  4314:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  4315:       [InstrStage<1, [SLOT0]>], [4, 1],
  4316:       [Hex_FWD, Hex_FWD]>,
  4317: 
  4318:     InstrItinData <tc_651cbe02, /*tc_1*/
  4319:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4320:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4321: 
  4322:     InstrItinData <tc_65279839, /*tc_2*/
  4323:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  4324:       [Hex_FWD, Hex_FWD]>,
  4325: 
  4326:     InstrItinData <tc_65cbd974, /*tc_st*/
  4327:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  4328:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4329: 
  4330:     InstrItinData <tc_69bfb303, /*tc_3*/
  4331:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  4332:       [Hex_FWD, Hex_FWD]>,
  4333: 
  4334:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  4335:       [InstrStage<1, [SLOT3]>], [4, 1],
  4336:       [Hex_FWD, Hex_FWD]>,
  4337: 
  4338:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  4339:       [InstrStage<1, [SLOT3]>], [4, 1],
  4340:       [Hex_FWD, Hex_FWD]>,
  4341: 
  4342:     InstrItinData <tc_6d861a95, /*tc_3stall*/
  4343:       [InstrStage<1, [SLOT3]>], [2, 1],
  4344:       [Hex_FWD, Hex_FWD]>,
  4345: 
  4346:     InstrItinData <tc_6e20402a, /*tc_st*/
  4347:       [InstrStage<1, [SLOT0]>], [2, 3],
  4348:       [Hex_FWD, Hex_FWD]>,
  4349: 
  4350:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  4351:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  4352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4353: 
  4354:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  4355:       [InstrStage<1, [SLOT0]>], [1, 1],
  4356:       [Hex_FWD, Hex_FWD]>,
  4357: 
  4358:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  4359:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  4360:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4361: 
  4362:     InstrItinData <tc_711c805f, /*tc_1*/
  4363:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  4364:       [Hex_FWD, Hex_FWD]>,
  4365: 
  4366:     InstrItinData <tc_713b66bf, /*tc_1*/
  4367:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  4368:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4369: 
  4370:     InstrItinData <tc_7401744f, /*tc_2*/
  4371:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  4372:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4373: 
  4374:     InstrItinData <tc_7476d766, /*tc_3stall*/
  4375:       [InstrStage<1, [SLOT3]>], [4, 2],
  4376:       [Hex_FWD, Hex_FWD]>,
  4377: 
  4378:     InstrItinData <tc_74a42bda, /*tc_ld*/
  4379:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  4380:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4381: 
  4382:     InstrItinData <tc_759e57be, /*tc_3stall*/
  4383:       [InstrStage<1, [SLOT2]>], [4, 1],
  4384:       [Hex_FWD, Hex_FWD]>,
  4385: 
  4386:     InstrItinData <tc_76bb5435, /*tc_ld*/
  4387:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  4388:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4389: 
  4390:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  4391:       [InstrStage<1, [SLOT2]>], [1],
  4392:       [Hex_FWD]>,
  4393: 
  4394:     InstrItinData <tc_77f94a5e, /*tc_st*/
  4395:       [InstrStage<1, [SLOT0]>], [],
  4396:       []>,
  4397: 
  4398:     InstrItinData <tc_788b1d09, /*tc_3x*/
  4399:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  4400:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4401: 
  4402:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  4403:       [InstrStage<1, [SLOT0]>], [],
  4404:       []>,
  4405: 
  4406:     InstrItinData <tc_7af3a37e, /*tc_st*/
  4407:       [InstrStage<1, [SLOT0]>], [1, 3],
  4408:       [Hex_FWD, Hex_FWD]>,
  4409: 
  4410:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  4411:       [InstrStage<1, [SLOT0]>], [3, 2],
  4412:       [Hex_FWD, Hex_FWD]>,
  4413: 
  4414:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  4415:       [InstrStage<1, [SLOT0]>], [3],
  4416:       [Hex_FWD]>,
  4417: 
  4418:     InstrItinData <tc_7c31e19a, /*tc_st*/
  4419:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  4420:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4421: 
  4422:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  4423:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  4424:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4425: 
  4426:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  4427:       [InstrStage<1, [SLOT3]>], [4, 1],
  4428:       [Hex_FWD, Hex_FWD]>,
  4429: 
  4430:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  4431:       [InstrStage<1, [SLOT3]>], [],
  4432:       []>,
  4433: 
  4434:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  4435:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  4436:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4437: 
  4438:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  4439:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  4440:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4441: 
  4442:     InstrItinData <tc_8035e91f, /*tc_st*/
  4443:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  4444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4445: 
  4446:     InstrItinData <tc_822c3c68, /*tc_ld*/
  4447:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  4448:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4449: 
  4450:     InstrItinData <tc_829d8a86, /*tc_st*/
  4451:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  4452:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4453: 
  4454:     InstrItinData <tc_838c4d7a, /*tc_st*/
  4455:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  4456:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4457: 
  4458:     InstrItinData <tc_84a7500d, /*tc_1*/
  4459:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  4460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4461: 
  4462:     InstrItinData <tc_86173609, /*tc_2latepred*/
  4463:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  4464:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4465: 
  4466:     InstrItinData <tc_887d1bb7, /*tc_st*/
  4467:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 2],
  4468:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4469: 
  4470:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  4471:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  4472:       [Hex_FWD, Hex_FWD]>,
  4473: 
  4474:     InstrItinData <tc_8a825db2, /*tc_2*/
  4475:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4476:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4477: 
  4478:     InstrItinData <tc_8b5bd4f5, /*tc_1*/
  4479:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  4480:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4481: 
  4482:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  4483:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 2],
  4484:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4485: 
  4486:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  4487:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  4488:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4489: 
  4490:     InstrItinData <tc_9124c04f, /*tc_1*/
  4491:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  4492:       [Hex_FWD, Hex_FWD]>,
  4493: 
  4494:     InstrItinData <tc_92240447, /*tc_st*/
  4495:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  4496:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4497: 
  4498:     InstrItinData <tc_934753bb, /*tc_ld*/
  4499:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
  4500:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 4501-5000 / 第 4501-5000 行

```tablegen
  4501: 
  4502:     InstrItinData <tc_937dd41c, /*tc_ld*/
  4503:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  4504:       []>,
  4505: 
  4506:     InstrItinData <tc_9406230a, /*tc_3x*/
  4507:       [InstrStage<1, [SLOT3]>], [2, 1],
  4508:       [Hex_FWD, Hex_FWD]>,
  4509: 
  4510:     InstrItinData <tc_95a33176, /*tc_2*/
  4511:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  4512:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4513: 
  4514:     InstrItinData <tc_95f43c5e, /*tc_3*/
  4515:       [InstrStage<1, [SLOT2]>], [1],
  4516:       [Hex_FWD]>,
  4517: 
  4518:     InstrItinData <tc_96ef76ef, /*tc_st*/
  4519:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  4520:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4521: 
  4522:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  4523:       [InstrStage<1, [SLOT0]>], [2, 3, 2],
  4524:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4525: 
  4526:     InstrItinData <tc_9783714b, /*tc_4x*/
  4527:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  4528:       [Hex_FWD, Hex_FWD]>,
  4529: 
  4530:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  4531:       [InstrStage<1, [SLOT2]>], [4, 1],
  4532:       [Hex_FWD, Hex_FWD]>,
  4533: 
  4534:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
  4535:       [InstrStage<1, [SLOT2]>], [],
  4536:       []>,
  4537: 
  4538:     InstrItinData <tc_9b3c0462, /*tc_2*/
  4539:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4541: 
  4542:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  4543:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  4544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4545: 
  4546:     InstrItinData <tc_9c52f549, /*tc_1*/
  4547:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  4548:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4549: 
  4550:     InstrItinData <tc_9e27f2f9, /*tc_1*/
  4551:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  4552:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4553: 
  4554:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  4555:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  4556:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4557: 
  4558:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  4559:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  4560:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4561: 
  4562:     InstrItinData <tc_9edefe01, /*tc_st*/
  4563:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 2],
  4564:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4565: 
  4566:     InstrItinData <tc_9f6cd987, /*tc_1*/
  4567:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  4568:       [Hex_FWD, Hex_FWD]>,
  4569: 
  4570:     InstrItinData <tc_a08b630b, /*tc_2*/
  4571:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4573: 
  4574:     InstrItinData <tc_a1297125, /*tc_1*/
  4575:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4576:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4577: 
  4578:     InstrItinData <tc_a154b476, /*tc_3x*/
  4579:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  4580:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4581: 
  4582:     InstrItinData <tc_a2b365d2, /*tc_st*/
  4583:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  4584:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4585: 
  4586:     InstrItinData <tc_a3070909, /*tc_3stall*/
  4587:       [InstrStage<1, [SLOT0]>], [1, 1],
  4588:       [Hex_FWD, Hex_FWD]>,
  4589: 
  4590:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  4591:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  4592:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4593: 
  4594:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  4595:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  4596:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4597: 
  4598:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  4599:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  4600:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4601: 
  4602:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  4603:       [InstrStage<1, [SLOT0]>], [],
  4604:       []>,
  4605: 
  4606:     InstrItinData <tc_a724463d, /*tc_3stall*/
  4607:       [InstrStage<1, [SLOT0]>], [4, 1],
  4608:       [Hex_FWD, Hex_FWD]>,
  4609: 
  4610:     InstrItinData <tc_a7a13fac, /*tc_1*/
  4611:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  4612:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4613: 
  4614:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  4615:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  4616:       [Hex_FWD, Hex_FWD]>,
  4617: 
  4618:     InstrItinData <tc_a9edeffa, /*tc_st*/
  4619:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  4620:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4621: 
  4622:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  4623:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  4624:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4625: 
  4626:     InstrItinData <tc_ac65613f, /*tc_ld*/
  4627:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  4628:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4629: 
  4630:     InstrItinData <tc_addc37a8, /*tc_st*/
  4631:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  4632:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4633: 
  4634:     InstrItinData <tc_ae5babd7, /*tc_st*/
  4635:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  4636:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4637: 
  4638:     InstrItinData <tc_aee6250c, /*tc_ld*/
  4639:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  4640:       [Hex_FWD, Hex_FWD]>,
  4641: 
  4642:     InstrItinData <tc_af6af259, /*tc_ld*/
  4643:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  4644:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4645: 
  4646:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  4647:       [InstrStage<1, [SLOT0]>], [1],
  4648:       [Hex_FWD]>,
  4649: 
  4650:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
  4651:       [InstrStage<1, [SLOT3]>], [1, 1],
  4652:       [Hex_FWD, Hex_FWD]>,
  4653: 
  4654:     InstrItinData <tc_b3d46584, /*tc_st*/
  4655:       [InstrStage<1, [SLOT0]>], [],
  4656:       []>,
  4657: 
  4658:     InstrItinData <tc_b4dc7630, /*tc_st*/
  4659:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 2],
  4660:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4661: 
  4662:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  4663:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  4664:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4665: 
  4666:     InstrItinData <tc_b837298f, /*tc_1*/
  4667:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  4668:       []>,
  4669: 
  4670:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  4671:       [InstrStage<1, [SLOT2]>], [],
  4672:       []>,
  4673: 
  4674:     InstrItinData <tc_ba9255a6, /*tc_st*/
  4675:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 2],
  4676:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4677: 
  4678:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  4679:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  4680:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4681: 
  4682:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  4683:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  4684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4685: 
  4686:     InstrItinData <tc_bb831a7c, /*tc_2*/
  4687:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  4688:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4689: 
  4690:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  4691:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  4692:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4693: 
  4694:     InstrItinData <tc_c20701f0, /*tc_2*/
  4695:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4696:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4697: 
  4698:     InstrItinData <tc_c21d7447, /*tc_3x*/
  4699:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  4700:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4701: 
  4702:     InstrItinData <tc_c57d9f39, /*tc_1*/
  4703:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  4704:       [Hex_FWD, Hex_FWD]>,
  4705: 
  4706:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  4707:       [InstrStage<1, [SLOT0]>], [],
  4708:       []>,
  4709: 
  4710:     InstrItinData <tc_ce59038e, /*tc_st*/
  4711:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  4712:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4713: 
  4714:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  4715:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  4716:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4717: 
  4718:     InstrItinData <tc_d03278fd, /*tc_st*/
  4719:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  4720:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4721: 
  4722:     InstrItinData <tc_d234b61a, /*tc_st*/
  4723:       [InstrStage<1, [SLOT0]>], [1],
  4724:       [Hex_FWD]>,
  4725: 
  4726:     InstrItinData <tc_d33e5eee, /*tc_1*/
  4727:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  4728:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4729: 
  4730:     InstrItinData <tc_d3632d88, /*tc_2*/
  4731:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  4732:       [Hex_FWD, Hex_FWD]>,
  4733: 
  4734:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  4735:       [InstrStage<1, [SLOT0]>], [1],
  4736:       [Hex_FWD]>,
  4737: 
  4738:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  4739:       [InstrStage<1, [SLOT2]>], [2],
  4740:       [Hex_FWD]>,
  4741: 
  4742:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  4743:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  4744:       [Hex_FWD, Hex_FWD]>,
  4745: 
  4746:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  4747:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  4748:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4749: 
  4750:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  4751:       [InstrStage<1, [SLOT3]>], [2, 1],
  4752:       [Hex_FWD, Hex_FWD]>,
  4753: 
  4754:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  4755:       [InstrStage<1, [SLOT3]>], [1],
  4756:       [Hex_FWD]>,
  4757: 
  4758:     InstrItinData <tc_db596beb, /*tc_3x*/
  4759:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  4760:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4761: 
  4762:     InstrItinData <tc_db96aa6b, /*tc_st*/
  4763:       [InstrStage<1, [SLOT0]>], [1],
  4764:       [Hex_FWD]>,
  4765: 
  4766:     InstrItinData <tc_dc51281d, /*tc_3*/
  4767:       [InstrStage<1, [SLOT2]>], [2, 1],
  4768:       [Hex_FWD, Hex_FWD]>,
  4769: 
  4770:     InstrItinData <tc_decdde8a, /*tc_1*/
  4771:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  4772:       [Hex_FWD]>,
  4773: 
  4774:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  4775:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
  4776:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4777: 
  4778:     InstrItinData <tc_e3d699e3, /*tc_2*/
  4779:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  4780:       [Hex_FWD, Hex_FWD]>,
  4781: 
  4782:     InstrItinData <tc_e60def48, /*tc_1*/
  4783:       [InstrStage<1, [SLOT2]>], [2],
  4784:       [Hex_FWD]>,
  4785: 
  4786:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  4787:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  4788:       [Hex_FWD, Hex_FWD]>,
  4789: 
  4790:     InstrItinData <tc_ed03645c, /*tc_1*/
  4791:       [InstrStage<1, [SLOT2]>], [3, 2],
  4792:       [Hex_FWD, Hex_FWD]>,
  4793: 
  4794:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  4795:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  4796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4797: 
  4798:     InstrItinData <tc_eed07714, /*tc_ld*/
  4799:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  4800:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4801: 
  4802:     InstrItinData <tc_eeda4109, /*tc_1*/
  4803:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  4804:       [Hex_FWD, Hex_FWD]>,
  4805: 
  4806:     InstrItinData <tc_ef921005, /*tc_1*/
  4807:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  4808:       [Hex_FWD, Hex_FWD]>,
  4809: 
  4810:     InstrItinData <tc_f098b237, /*tc_1*/
  4811:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  4812:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4813: 
  4814:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  4815:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  4816:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4817: 
  4818:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  4819:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  4820:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4821: 
  4822:     InstrItinData <tc_f34c1c21, /*tc_2*/
  4823:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4824:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4825: 
  4826:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  4827:       [InstrStage<1, [SLOT0]>], [2],
  4828:       [Hex_FWD]>,
  4829: 
  4830:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
  4831:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
  4832:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4833: 
  4834:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  4835:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  4836:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4837: 
  4838:     InstrItinData <tc_f7569068, /*tc_4x*/
  4839:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  4840:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4841: 
  4842:     InstrItinData <tc_f97707c1, /*tc_1*/
  4843:       [InstrStage<1, [SLOT2]>], [2],
  4844:       [Hex_FWD]>,
  4845: 
  4846:     InstrItinData <tc_f999c66e, /*tc_1*/
  4847:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  4848:       [Hex_FWD, Hex_FWD]>,
  4849: 
  4850:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  4851:       [InstrStage<1, [SLOT3]>], [4, 2],
  4852:       [Hex_FWD, Hex_FWD]>,
  4853: 
  4854:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  4855:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  4856:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  4857:   ];
  4858: }
  4859: 
  4860: class DepScalarItinV66 {
  4861:   list<InstrItinData> DepScalarItinV66_list = [
  4862:     InstrItinData <tc_011e0e9d, /*tc_st*/
  4863:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  4864:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4865: 
  4866:     InstrItinData <tc_01d44cb2, /*tc_2*/
  4867:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4868:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4869: 
  4870:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  4871:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  4872:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4873: 
  4874:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  4875:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  4876:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4877: 
  4878:     InstrItinData <tc_0655b949, /*tc_st*/
  4879:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
  4880:       [Hex_FWD, Hex_FWD]>,
  4881: 
  4882:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  4883:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  4884:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4885: 
  4886:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  4887:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  4888:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4889: 
  4890:     InstrItinData <tc_0a43be35, /*tc_3x*/
  4891:       [InstrStage<1, [SLOT3]>], [1],
  4892:       [Hex_FWD]>,
  4893: 
  4894:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  4895:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  4896:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4897: 
  4898:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
  4899:       [InstrStage<1, [SLOT2]>], [1],
  4900:       [Hex_FWD]>,
  4901: 
  4902:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  4903:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4904:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4905: 
  4906:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  4907:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  4908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4909: 
  4910:     InstrItinData <tc_112d30d6, /*tc_1*/
  4911:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  4912:       [Hex_FWD]>,
  4913: 
  4914:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  4915:       [InstrStage<1, [SLOT0]>], [2],
  4916:       [Hex_FWD]>,
  4917: 
  4918:     InstrItinData <tc_1248597c, /*tc_3x*/
  4919:       [InstrStage<1, [SLOT3]>], [2, 2],
  4920:       [Hex_FWD, Hex_FWD]>,
  4921: 
  4922:     InstrItinData <tc_139ef484, /*tc_3stall*/
  4923:       [InstrStage<1, [SLOT2]>], [1, 1],
  4924:       [Hex_FWD, Hex_FWD]>,
  4925: 
  4926:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  4927:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
  4928:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4929: 
  4930:     InstrItinData <tc_151bf368, /*tc_1*/
  4931:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  4932:       [Hex_FWD, Hex_FWD]>,
  4933: 
  4934:     InstrItinData <tc_158aa3f7, /*tc_st*/
  4935:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  4936:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4937: 
  4938:     InstrItinData <tc_197dce51, /*tc_3x*/
  4939:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  4940:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4941: 
  4942:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  4943:       [InstrStage<1, [SLOT0]>], [3],
  4944:       [Hex_FWD]>,
  4945: 
  4946:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  4947:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  4948:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4949: 
  4950:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  4951:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  4952:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4953: 
  4954:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  4955:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  4956:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4957: 
  4958:     InstrItinData <tc_1fcb8495, /*tc_2*/
  4959:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4960:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4961: 
  4962:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  4963:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
  4964:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4965: 
  4966:     InstrItinData <tc_20131976, /*tc_2*/
  4967:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  4968:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4969: 
  4970:     InstrItinData <tc_2237d952, /*tc_ld*/
  4971:       [InstrStage<1, [SLOT0]>], [1, 2],
  4972:       [Hex_FWD, Hex_FWD]>,
  4973: 
  4974:     InstrItinData <tc_23708a21, /*tc_1*/
  4975:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  4976:       []>,
  4977: 
  4978:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  4979:       [InstrStage<1, [SLOT0]>], [4, 1],
  4980:       [Hex_FWD, Hex_FWD]>,
  4981: 
  4982:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  4983:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  4984:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4985: 
  4986:     InstrItinData <tc_24f426ab, /*tc_1*/
  4987:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  4988:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4989: 
  4990:     InstrItinData <tc_27106296, /*tc_3x*/
  4991:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  4992:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  4993: 
  4994:     InstrItinData <tc_280f7fe1, /*tc_st*/
  4995:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
  4996:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  4997: 
  4998:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  4999:       [InstrStage<1, [SLOT3]>], [1, 1],
  5000:       [Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV66, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV66; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV66 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV66 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 5001-5500 / 第 5001-5500 行

```tablegen
  5001: 
  5002:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  5003:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  5004:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5005: 
  5006:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  5007:       [InstrStage<1, [SLOT3]>], [1],
  5008:       [Hex_FWD]>,
  5009: 
  5010:     InstrItinData <tc_2f573607, /*tc_1*/
  5011:       [InstrStage<1, [SLOT2]>], [2, 2],
  5012:       [Hex_FWD, Hex_FWD]>,
  5013: 
  5014:     InstrItinData <tc_33e7e673, /*tc_2early*/
  5015:       [InstrStage<1, [SLOT2]>], [],
  5016:       []>,
  5017: 
  5018:     InstrItinData <tc_362b0be2, /*tc_3*/
  5019:       [InstrStage<1, [SLOT2]>], [1],
  5020:       [Hex_FWD]>,
  5021: 
  5022:     InstrItinData <tc_38382228, /*tc_3x*/
  5023:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  5024:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5025: 
  5026:     InstrItinData <tc_388f9897, /*tc_1*/
  5027:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  5028:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5029: 
  5030:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  5031:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
  5032:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5033: 
  5034:     InstrItinData <tc_3d14a17b, /*tc_1*/
  5035:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  5036:       [Hex_FWD, Hex_FWD]>,
  5037: 
  5038:     InstrItinData <tc_3edca78f, /*tc_2*/
  5039:       [InstrStage<1, [SLOT3]>], [4, 2],
  5040:       [Hex_FWD, Hex_FWD]>,
  5041: 
  5042:     InstrItinData <tc_3fbf1042, /*tc_1*/
  5043:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  5044:       [Hex_FWD]>,
  5045: 
  5046:     InstrItinData <tc_407e96f9, /*tc_1*/
  5047:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5048:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5049: 
  5050:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  5051:       [InstrStage<1, [SLOT0]>], [3, 1],
  5052:       [Hex_FWD, Hex_FWD]>,
  5053: 
  5054:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  5055:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  5056:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5057: 
  5058:     InstrItinData <tc_42ff66ba, /*tc_1*/
  5059:       [InstrStage<1, [SLOT2]>], [2, 2],
  5060:       [Hex_FWD, Hex_FWD]>,
  5061: 
  5062:     InstrItinData <tc_442395f3, /*tc_2latepred*/
  5063:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
  5064:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5065: 
  5066:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
  5067:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
  5068:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5069: 
  5070:     InstrItinData <tc_44d5a428, /*tc_st*/
  5071:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  5072:       [Hex_FWD, Hex_FWD]>,
  5073: 
  5074:     InstrItinData <tc_44fffc58, /*tc_3*/
  5075:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  5076:       [Hex_FWD]>,
  5077: 
  5078:     InstrItinData <tc_45791fb8, /*tc_ld*/
  5079:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  5080:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5081: 
  5082:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  5083:       [InstrStage<1, [SLOT2]>], [2],
  5084:       [Hex_FWD]>,
  5085: 
  5086:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  5087:       [InstrStage<1, [SLOT3]>], [4, 1],
  5088:       [Hex_FWD, Hex_FWD]>,
  5089: 
  5090:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
  5091:       [InstrStage<1, [SLOT3]>], [4, 1],
  5092:       [Hex_FWD, Hex_FWD]>,
  5093: 
  5094:     InstrItinData <tc_4a55d03c, /*tc_1*/
  5095:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5096:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5097: 
  5098:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
  5099:       [InstrStage<1, [SLOT3]>], [2, 2],
  5100:       [Hex_FWD, Hex_FWD]>,
  5101: 
  5102:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  5103:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  5104:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5105: 
  5106:     InstrItinData <tc_4bf903b0, /*tc_st*/
  5107:       [InstrStage<1, [SLOT0]>], [3],
  5108:       [Hex_FWD]>,
  5109: 
  5110:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  5111:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  5112:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5113: 
  5114:     InstrItinData <tc_512b1653, /*tc_st*/
  5115:       [InstrStage<1, [SLOT0]>], [1, 2],
  5116:       [Hex_FWD, Hex_FWD]>,
  5117: 
  5118:     InstrItinData <tc_53c851ab, /*tc_3stall*/
  5119:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
  5120:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5121: 
  5122:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  5123:       [InstrStage<1, [SLOT3]>], [1],
  5124:       [Hex_FWD]>,
  5125: 
  5126:     InstrItinData <tc_5502c366, /*tc_1*/
  5127:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5129: 
  5130:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  5131:       [InstrStage<1, [SLOT3]>], [],
  5132:       []>,
  5133: 
  5134:     InstrItinData <tc_556f6577, /*tc_3x*/
  5135:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  5136:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5137: 
  5138:     InstrItinData <tc_55a9a350, /*tc_st*/
  5139:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  5140:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5141: 
  5142:     InstrItinData <tc_55b33fda, /*tc_1*/
  5143:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  5144:       [Hex_FWD, Hex_FWD]>,
  5145: 
  5146:     InstrItinData <tc_56a124a7, /*tc_1*/
  5147:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  5148:       [Hex_FWD, Hex_FWD]>,
  5149: 
  5150:     InstrItinData <tc_57a55b54, /*tc_1*/
  5151:       [InstrStage<1, [SLOT3]>], [2, 2],
  5152:       [Hex_FWD, Hex_FWD]>,
  5153: 
  5154:     InstrItinData <tc_5944960d, /*tc_ld*/
  5155:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  5156:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5157: 
  5158:     InstrItinData <tc_59a7822c, /*tc_1*/
  5159:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  5160:       [Hex_FWD, Hex_FWD]>,
  5161: 
  5162:     InstrItinData <tc_5a222e89, /*tc_2early*/
  5163:       [InstrStage<1, [SLOT2]>], [1, 1],
  5164:       [Hex_FWD, Hex_FWD]>,
  5165: 
  5166:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  5167:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  5168:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5169: 
  5170:     InstrItinData <tc_5b347363, /*tc_1*/
  5171:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  5172:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5173: 
  5174:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  5175:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  5176:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5177: 
  5178:     InstrItinData <tc_5da50c4b, /*tc_1*/
  5179:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5180:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5181: 
  5182:     InstrItinData <tc_5deb5e47, /*tc_st*/
  5183:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  5184:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5185: 
  5186:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  5187:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  5188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5189: 
  5190:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
  5191:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
  5192:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5193: 
  5194:     InstrItinData <tc_60e324ff, /*tc_1*/
  5195:       [InstrStage<1, [SLOT2]>], [2],
  5196:       [Hex_FWD]>,
  5197: 
  5198:     InstrItinData <tc_63567288, /*tc_2latepred*/
  5199:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
  5200:       [Hex_FWD]>,
  5201: 
  5202:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  5203:       [InstrStage<1, [SLOT0]>], [4, 1],
  5204:       [Hex_FWD, Hex_FWD]>,
  5205: 
  5206:     InstrItinData <tc_651cbe02, /*tc_1*/
  5207:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5208:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5209: 
  5210:     InstrItinData <tc_65279839, /*tc_2*/
  5211:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  5212:       [Hex_FWD, Hex_FWD]>,
  5213: 
  5214:     InstrItinData <tc_65cbd974, /*tc_st*/
  5215:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  5216:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5217: 
  5218:     InstrItinData <tc_69bfb303, /*tc_3*/
  5219:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  5220:       [Hex_FWD, Hex_FWD]>,
  5221: 
  5222:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  5223:       [InstrStage<1, [SLOT3]>], [4, 1],
  5224:       [Hex_FWD, Hex_FWD]>,
  5225: 
  5226:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  5227:       [InstrStage<1, [SLOT3]>], [4, 1],
  5228:       [Hex_FWD, Hex_FWD]>,
  5229: 
  5230:     InstrItinData <tc_6d861a95, /*tc_3x*/
  5231:       [InstrStage<1, [SLOT3]>], [2, 1],
  5232:       [Hex_FWD, Hex_FWD]>,
  5233: 
  5234:     InstrItinData <tc_6e20402a, /*tc_st*/
  5235:       [InstrStage<1, [SLOT0]>], [2, 3],
  5236:       [Hex_FWD, Hex_FWD]>,
  5237: 
  5238:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  5239:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  5240:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5241: 
  5242:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  5243:       [InstrStage<1, [SLOT0]>], [1, 1],
  5244:       [Hex_FWD, Hex_FWD]>,
  5245: 
  5246:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  5247:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  5248:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5249: 
  5250:     InstrItinData <tc_711c805f, /*tc_1*/
  5251:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  5252:       [Hex_FWD, Hex_FWD]>,
  5253: 
  5254:     InstrItinData <tc_713b66bf, /*tc_1*/
  5255:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  5256:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5257: 
  5258:     InstrItinData <tc_7401744f, /*tc_2*/
  5259:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  5260:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5261: 
  5262:     InstrItinData <tc_7476d766, /*tc_3stall*/
  5263:       [InstrStage<1, [SLOT3]>], [4, 2],
  5264:       [Hex_FWD, Hex_FWD]>,
  5265: 
  5266:     InstrItinData <tc_74a42bda, /*tc_ld*/
  5267:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  5268:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5269: 
  5270:     InstrItinData <tc_759e57be, /*tc_3stall*/
  5271:       [InstrStage<1, [SLOT2]>], [4, 1],
  5272:       [Hex_FWD, Hex_FWD]>,
  5273: 
  5274:     InstrItinData <tc_76bb5435, /*tc_ld*/
  5275:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  5276:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5277: 
  5278:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  5279:       [InstrStage<1, [SLOT2]>], [1],
  5280:       [Hex_FWD]>,
  5281: 
  5282:     InstrItinData <tc_77f94a5e, /*tc_st*/
  5283:       [InstrStage<1, [SLOT0]>], [],
  5284:       []>,
  5285: 
  5286:     InstrItinData <tc_788b1d09, /*tc_3x*/
  5287:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  5288:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5289: 
  5290:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  5291:       [InstrStage<1, [SLOT0]>], [],
  5292:       []>,
  5293: 
  5294:     InstrItinData <tc_7af3a37e, /*tc_st*/
  5295:       [InstrStage<1, [SLOT0]>], [1, 3],
  5296:       [Hex_FWD, Hex_FWD]>,
  5297: 
  5298:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  5299:       [InstrStage<1, [SLOT0]>], [3, 2],
  5300:       [Hex_FWD, Hex_FWD]>,
  5301: 
  5302:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  5303:       [InstrStage<1, [SLOT0]>], [3],
  5304:       [Hex_FWD]>,
  5305: 
  5306:     InstrItinData <tc_7c31e19a, /*tc_st*/
  5307:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  5308:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5309: 
  5310:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  5311:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  5312:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5313: 
  5314:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  5315:       [InstrStage<1, [SLOT3]>], [4, 1],
  5316:       [Hex_FWD, Hex_FWD]>,
  5317: 
  5318:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  5319:       [InstrStage<1, [SLOT3]>], [],
  5320:       []>,
  5321: 
  5322:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  5323:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  5324:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5325: 
  5326:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  5327:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  5328:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5329: 
  5330:     InstrItinData <tc_8035e91f, /*tc_st*/
  5331:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
  5332:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5333: 
  5334:     InstrItinData <tc_822c3c68, /*tc_ld*/
  5335:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  5336:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5337: 
  5338:     InstrItinData <tc_829d8a86, /*tc_st*/
  5339:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  5340:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5341: 
  5342:     InstrItinData <tc_838c4d7a, /*tc_st*/
  5343:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  5344:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5345: 
  5346:     InstrItinData <tc_84a7500d, /*tc_2*/
  5347:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  5348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5349: 
  5350:     InstrItinData <tc_86173609, /*tc_2latepred*/
  5351:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  5352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5353: 
  5354:     InstrItinData <tc_887d1bb7, /*tc_st*/
  5355:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
  5356:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5357: 
  5358:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  5359:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  5360:       [Hex_FWD, Hex_FWD]>,
  5361: 
  5362:     InstrItinData <tc_8a825db2, /*tc_2*/
  5363:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5364:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5365: 
  5366:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
  5367:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  5368:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5369: 
  5370:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  5371:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
  5372:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5373: 
  5374:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  5375:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  5376:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5377: 
  5378:     InstrItinData <tc_9124c04f, /*tc_1*/
  5379:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  5380:       [Hex_FWD, Hex_FWD]>,
  5381: 
  5382:     InstrItinData <tc_92240447, /*tc_st*/
  5383:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  5384:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5385: 
  5386:     InstrItinData <tc_934753bb, /*tc_ld*/
  5387:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
  5388:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5389: 
  5390:     InstrItinData <tc_937dd41c, /*tc_ld*/
  5391:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  5392:       []>,
  5393: 
  5394:     InstrItinData <tc_9406230a, /*tc_3x*/
  5395:       [InstrStage<1, [SLOT3]>], [2, 1],
  5396:       [Hex_FWD, Hex_FWD]>,
  5397: 
  5398:     InstrItinData <tc_95a33176, /*tc_2*/
  5399:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  5400:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5401: 
  5402:     InstrItinData <tc_95f43c5e, /*tc_3*/
  5403:       [InstrStage<1, [SLOT2]>], [1],
  5404:       [Hex_FWD]>,
  5405: 
  5406:     InstrItinData <tc_96ef76ef, /*tc_st*/
  5407:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  5408:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5409: 
  5410:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  5411:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  5412:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5413: 
  5414:     InstrItinData <tc_9783714b, /*tc_4x*/
  5415:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  5416:       [Hex_FWD, Hex_FWD]>,
  5417: 
  5418:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  5419:       [InstrStage<1, [SLOT2]>], [4, 1],
  5420:       [Hex_FWD, Hex_FWD]>,
  5421: 
  5422:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
  5423:       [InstrStage<1, [SLOT2]>], [],
  5424:       []>,
  5425: 
  5426:     InstrItinData <tc_9b3c0462, /*tc_2*/
  5427:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5428:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5429: 
  5430:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  5431:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  5432:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5433: 
  5434:     InstrItinData <tc_9c52f549, /*tc_1*/
  5435:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  5436:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5437: 
  5438:     InstrItinData <tc_9e27f2f9, /*tc_1*/
  5439:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  5440:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5441: 
  5442:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  5443:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  5444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5445: 
  5446:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  5447:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  5448:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5449: 
  5450:     InstrItinData <tc_9edefe01, /*tc_st*/
  5451:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
  5452:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5453: 
  5454:     InstrItinData <tc_9f6cd987, /*tc_1*/
  5455:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  5456:       [Hex_FWD, Hex_FWD]>,
  5457: 
  5458:     InstrItinData <tc_a08b630b, /*tc_2*/
  5459:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5461: 
  5462:     InstrItinData <tc_a1297125, /*tc_1*/
  5463:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5464:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5465: 
  5466:     InstrItinData <tc_a154b476, /*tc_3x*/
  5467:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  5468:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5469: 
  5470:     InstrItinData <tc_a2b365d2, /*tc_st*/
  5471:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
  5472:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5473: 
  5474:     InstrItinData <tc_a3070909, /*tc_3stall*/
  5475:       [InstrStage<1, [SLOT0]>], [1, 1],
  5476:       [Hex_FWD, Hex_FWD]>,
  5477: 
  5478:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  5479:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  5480:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5481: 
  5482:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  5483:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  5484:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5485: 
  5486:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  5487:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  5488:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5489: 
  5490:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  5491:       [InstrStage<1, [SLOT0]>], [],
  5492:       []>,
  5493: 
  5494:     InstrItinData <tc_a724463d, /*tc_3stall*/
  5495:       [InstrStage<1, [SLOT0]>], [4, 1],
  5496:       [Hex_FWD, Hex_FWD]>,
  5497: 
  5498:     InstrItinData <tc_a7a13fac, /*tc_1*/
  5499:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  5500:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 5501-6000 / 第 5501-6000 行

```tablegen
  5501: 
  5502:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  5503:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  5504:       [Hex_FWD, Hex_FWD]>,
  5505: 
  5506:     InstrItinData <tc_a9edeffa, /*tc_st*/
  5507:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  5508:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5509: 
  5510:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  5511:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  5512:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5513: 
  5514:     InstrItinData <tc_ac65613f, /*tc_ld*/
  5515:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  5516:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5517: 
  5518:     InstrItinData <tc_addc37a8, /*tc_st*/
  5519:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  5520:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5521: 
  5522:     InstrItinData <tc_ae5babd7, /*tc_st*/
  5523:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  5524:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5525: 
  5526:     InstrItinData <tc_aee6250c, /*tc_ld*/
  5527:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  5528:       [Hex_FWD, Hex_FWD]>,
  5529: 
  5530:     InstrItinData <tc_af6af259, /*tc_ld*/
  5531:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  5532:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5533: 
  5534:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  5535:       [InstrStage<1, [SLOT0]>], [1],
  5536:       [Hex_FWD]>,
  5537: 
  5538:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
  5539:       [InstrStage<1, [SLOT3]>], [1, 1],
  5540:       [Hex_FWD, Hex_FWD]>,
  5541: 
  5542:     InstrItinData <tc_b3d46584, /*tc_st*/
  5543:       [InstrStage<1, [SLOT0]>], [],
  5544:       []>,
  5545: 
  5546:     InstrItinData <tc_b4dc7630, /*tc_st*/
  5547:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
  5548:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5549: 
  5550:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  5551:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  5552:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5553: 
  5554:     InstrItinData <tc_b837298f, /*tc_1*/
  5555:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  5556:       []>,
  5557: 
  5558:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  5559:       [InstrStage<1, [SLOT2]>], [],
  5560:       []>,
  5561: 
  5562:     InstrItinData <tc_ba9255a6, /*tc_st*/
  5563:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
  5564:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5565: 
  5566:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  5567:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
  5568:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5569: 
  5570:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  5571:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  5572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5573: 
  5574:     InstrItinData <tc_bb831a7c, /*tc_2*/
  5575:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  5576:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5577: 
  5578:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  5579:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  5580:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5581: 
  5582:     InstrItinData <tc_c20701f0, /*tc_2*/
  5583:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5584:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5585: 
  5586:     InstrItinData <tc_c21d7447, /*tc_3x*/
  5587:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  5588:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5589: 
  5590:     InstrItinData <tc_c57d9f39, /*tc_1*/
  5591:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  5592:       [Hex_FWD, Hex_FWD]>,
  5593: 
  5594:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  5595:       [InstrStage<1, [SLOT0]>], [],
  5596:       []>,
  5597: 
  5598:     InstrItinData <tc_ce59038e, /*tc_st*/
  5599:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  5600:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5601: 
  5602:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  5603:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  5604:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5605: 
  5606:     InstrItinData <tc_d03278fd, /*tc_st*/
  5607:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  5608:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5609: 
  5610:     InstrItinData <tc_d234b61a, /*tc_st*/
  5611:       [InstrStage<1, [SLOT0]>], [1],
  5612:       [Hex_FWD]>,
  5613: 
  5614:     InstrItinData <tc_d33e5eee, /*tc_1*/
  5615:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  5616:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5617: 
  5618:     InstrItinData <tc_d3632d88, /*tc_2*/
  5619:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  5620:       [Hex_FWD, Hex_FWD]>,
  5621: 
  5622:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  5623:       [InstrStage<1, [SLOT0]>], [1],
  5624:       [Hex_FWD]>,
  5625: 
  5626:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  5627:       [InstrStage<1, [SLOT2]>], [2],
  5628:       [Hex_FWD]>,
  5629: 
  5630:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  5631:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  5632:       [Hex_FWD, Hex_FWD]>,
  5633: 
  5634:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  5635:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  5636:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5637: 
  5638:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  5639:       [InstrStage<1, [SLOT3]>], [2, 1],
  5640:       [Hex_FWD, Hex_FWD]>,
  5641: 
  5642:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  5643:       [InstrStage<1, [SLOT3]>], [1],
  5644:       [Hex_FWD]>,
  5645: 
  5646:     InstrItinData <tc_db596beb, /*tc_3x*/
  5647:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  5648:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5649: 
  5650:     InstrItinData <tc_db96aa6b, /*tc_st*/
  5651:       [InstrStage<1, [SLOT0]>], [1],
  5652:       [Hex_FWD]>,
  5653: 
  5654:     InstrItinData <tc_dc51281d, /*tc_3*/
  5655:       [InstrStage<1, [SLOT2]>], [2, 1],
  5656:       [Hex_FWD, Hex_FWD]>,
  5657: 
  5658:     InstrItinData <tc_decdde8a, /*tc_1*/
  5659:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  5660:       [Hex_FWD]>,
  5661: 
  5662:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  5663:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
  5664:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5665: 
  5666:     InstrItinData <tc_e3d699e3, /*tc_2*/
  5667:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  5668:       [Hex_FWD, Hex_FWD]>,
  5669: 
  5670:     InstrItinData <tc_e60def48, /*tc_1*/
  5671:       [InstrStage<1, [SLOT2]>], [2],
  5672:       [Hex_FWD]>,
  5673: 
  5674:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  5675:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  5676:       [Hex_FWD, Hex_FWD]>,
  5677: 
  5678:     InstrItinData <tc_ed03645c, /*tc_1*/
  5679:       [InstrStage<1, [SLOT2]>], [3, 2],
  5680:       [Hex_FWD, Hex_FWD]>,
  5681: 
  5682:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  5683:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  5684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5685: 
  5686:     InstrItinData <tc_eed07714, /*tc_ld*/
  5687:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  5688:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5689: 
  5690:     InstrItinData <tc_eeda4109, /*tc_1*/
  5691:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  5692:       [Hex_FWD, Hex_FWD]>,
  5693: 
  5694:     InstrItinData <tc_ef921005, /*tc_1*/
  5695:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  5696:       [Hex_FWD, Hex_FWD]>,
  5697: 
  5698:     InstrItinData <tc_f098b237, /*tc_2*/
  5699:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5700:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5701: 
  5702:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  5703:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  5704:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5705: 
  5706:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  5707:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  5708:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5709: 
  5710:     InstrItinData <tc_f34c1c21, /*tc_2*/
  5711:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5712:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5713: 
  5714:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  5715:       [InstrStage<1, [SLOT0]>], [2],
  5716:       [Hex_FWD]>,
  5717: 
  5718:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
  5719:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
  5720:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5721: 
  5722:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  5723:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  5724:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5725: 
  5726:     InstrItinData <tc_f7569068, /*tc_4x*/
  5727:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  5728:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5729: 
  5730:     InstrItinData <tc_f97707c1, /*tc_1*/
  5731:       [InstrStage<1, [SLOT2]>], [2],
  5732:       [Hex_FWD]>,
  5733: 
  5734:     InstrItinData <tc_f999c66e, /*tc_1*/
  5735:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  5736:       [Hex_FWD, Hex_FWD]>,
  5737: 
  5738:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  5739:       [InstrStage<1, [SLOT3]>], [4, 2],
  5740:       [Hex_FWD, Hex_FWD]>,
  5741: 
  5742:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  5743:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  5744:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  5745:   ];
  5746: }
  5747: 
  5748: class DepScalarItinV67 {
  5749:   list<InstrItinData> DepScalarItinV67_list = [
  5750:     InstrItinData <tc_011e0e9d, /*tc_st*/
  5751:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  5752:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5753: 
  5754:     InstrItinData <tc_01d44cb2, /*tc_2*/
  5755:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5756:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5757: 
  5758:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  5759:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  5760:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5761: 
  5762:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  5763:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  5764:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5765: 
  5766:     InstrItinData <tc_0655b949, /*tc_st*/
  5767:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
  5768:       [Hex_FWD, Hex_FWD]>,
  5769: 
  5770:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  5771:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  5772:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5773: 
  5774:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  5775:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  5776:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5777: 
  5778:     InstrItinData <tc_0a43be35, /*tc_3x*/
  5779:       [InstrStage<1, [SLOT3]>], [1],
  5780:       [Hex_FWD]>,
  5781: 
  5782:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  5783:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  5784:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5785: 
  5786:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
  5787:       [InstrStage<1, [SLOT2]>], [1],
  5788:       [Hex_FWD]>,
  5789: 
  5790:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  5791:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5793: 
  5794:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  5795:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  5796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5797: 
  5798:     InstrItinData <tc_112d30d6, /*tc_1*/
  5799:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  5800:       [Hex_FWD]>,
  5801: 
  5802:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  5803:       [InstrStage<1, [SLOT0]>], [2],
  5804:       [Hex_FWD]>,
  5805: 
  5806:     InstrItinData <tc_1248597c, /*tc_3x*/
  5807:       [InstrStage<1, [SLOT3]>], [2, 2],
  5808:       [Hex_FWD, Hex_FWD]>,
  5809: 
  5810:     InstrItinData <tc_139ef484, /*tc_3stall*/
  5811:       [InstrStage<1, [SLOT2]>], [1, 1],
  5812:       [Hex_FWD, Hex_FWD]>,
  5813: 
  5814:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  5815:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
  5816:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5817: 
  5818:     InstrItinData <tc_151bf368, /*tc_1*/
  5819:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  5820:       [Hex_FWD, Hex_FWD]>,
  5821: 
  5822:     InstrItinData <tc_158aa3f7, /*tc_st*/
  5823:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  5824:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5825: 
  5826:     InstrItinData <tc_197dce51, /*tc_3x*/
  5827:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  5828:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5829: 
  5830:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  5831:       [InstrStage<1, [SLOT0]>], [3],
  5832:       [Hex_FWD]>,
  5833: 
  5834:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  5835:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  5836:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5837: 
  5838:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  5839:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  5840:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5841: 
  5842:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  5843:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  5844:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5845: 
  5846:     InstrItinData <tc_1fcb8495, /*tc_2*/
  5847:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5848:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5849: 
  5850:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  5851:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
  5852:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5853: 
  5854:     InstrItinData <tc_20131976, /*tc_2*/
  5855:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  5856:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5857: 
  5858:     InstrItinData <tc_2237d952, /*tc_ld*/
  5859:       [InstrStage<1, [SLOT0]>], [1, 2],
  5860:       [Hex_FWD, Hex_FWD]>,
  5861: 
  5862:     InstrItinData <tc_23708a21, /*tc_1*/
  5863:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  5864:       []>,
  5865: 
  5866:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  5867:       [InstrStage<1, [SLOT0]>], [4, 1],
  5868:       [Hex_FWD, Hex_FWD]>,
  5869: 
  5870:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  5871:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  5872:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5873: 
  5874:     InstrItinData <tc_24f426ab, /*tc_1*/
  5875:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  5876:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5877: 
  5878:     InstrItinData <tc_27106296, /*tc_3x*/
  5879:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  5880:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5881: 
  5882:     InstrItinData <tc_280f7fe1, /*tc_st*/
  5883:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
  5884:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5885: 
  5886:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  5887:       [InstrStage<1, [SLOT3]>], [1, 1],
  5888:       [Hex_FWD, Hex_FWD]>,
  5889: 
  5890:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  5891:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  5892:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5893: 
  5894:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  5895:       [InstrStage<1, [SLOT3]>], [1],
  5896:       [Hex_FWD]>,
  5897: 
  5898:     InstrItinData <tc_2f573607, /*tc_1*/
  5899:       [InstrStage<1, [SLOT2]>], [2, 2],
  5900:       [Hex_FWD, Hex_FWD]>,
  5901: 
  5902:     InstrItinData <tc_33e7e673, /*tc_2early*/
  5903:       [InstrStage<1, [SLOT2]>], [],
  5904:       []>,
  5905: 
  5906:     InstrItinData <tc_362b0be2, /*tc_3*/
  5907:       [InstrStage<1, [SLOT2]>], [1],
  5908:       [Hex_FWD]>,
  5909: 
  5910:     InstrItinData <tc_38382228, /*tc_3x*/
  5911:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  5912:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5913: 
  5914:     InstrItinData <tc_388f9897, /*tc_1*/
  5915:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  5916:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5917: 
  5918:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  5919:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
  5920:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5921: 
  5922:     InstrItinData <tc_3d14a17b, /*tc_1*/
  5923:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  5924:       [Hex_FWD, Hex_FWD]>,
  5925: 
  5926:     InstrItinData <tc_3edca78f, /*tc_2*/
  5927:       [InstrStage<1, [SLOT3]>], [4, 2],
  5928:       [Hex_FWD, Hex_FWD]>,
  5929: 
  5930:     InstrItinData <tc_3fbf1042, /*tc_1*/
  5931:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  5932:       [Hex_FWD]>,
  5933: 
  5934:     InstrItinData <tc_407e96f9, /*tc_1*/
  5935:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5936:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5937: 
  5938:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  5939:       [InstrStage<1, [SLOT0]>], [3, 1],
  5940:       [Hex_FWD, Hex_FWD]>,
  5941: 
  5942:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  5943:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  5944:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5945: 
  5946:     InstrItinData <tc_42ff66ba, /*tc_1*/
  5947:       [InstrStage<1, [SLOT2]>], [2, 2],
  5948:       [Hex_FWD, Hex_FWD]>,
  5949: 
  5950:     InstrItinData <tc_442395f3, /*tc_2latepred*/
  5951:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
  5952:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5953: 
  5954:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
  5955:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
  5956:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5957: 
  5958:     InstrItinData <tc_44d5a428, /*tc_st*/
  5959:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  5960:       [Hex_FWD, Hex_FWD]>,
  5961: 
  5962:     InstrItinData <tc_44fffc58, /*tc_3*/
  5963:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  5964:       [Hex_FWD]>,
  5965: 
  5966:     InstrItinData <tc_45791fb8, /*tc_ld*/
  5967:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  5968:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  5969: 
  5970:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  5971:       [InstrStage<1, [SLOT2]>], [2],
  5972:       [Hex_FWD]>,
  5973: 
  5974:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  5975:       [InstrStage<1, [SLOT3]>], [4, 1],
  5976:       [Hex_FWD, Hex_FWD]>,
  5977: 
  5978:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
  5979:       [InstrStage<1, [SLOT3]>], [4, 1],
  5980:       [Hex_FWD, Hex_FWD]>,
  5981: 
  5982:     InstrItinData <tc_4a55d03c, /*tc_1*/
  5983:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  5984:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5985: 
  5986:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
  5987:       [InstrStage<1, [SLOT3]>], [2, 2],
  5988:       [Hex_FWD, Hex_FWD]>,
  5989: 
  5990:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  5991:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  5992:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  5993: 
  5994:     InstrItinData <tc_4bf903b0, /*tc_st*/
  5995:       [InstrStage<1, [SLOT0]>], [3],
  5996:       [Hex_FWD]>,
  5997: 
  5998:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  5999:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  6000:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV67, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV67; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV67 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV67 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 6001-6500 / 第 6001-6500 行

```tablegen
  6001: 
  6002:     InstrItinData <tc_512b1653, /*tc_st*/
  6003:       [InstrStage<1, [SLOT0]>], [1, 2],
  6004:       [Hex_FWD, Hex_FWD]>,
  6005: 
  6006:     InstrItinData <tc_53c851ab, /*tc_3stall*/
  6007:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
  6008:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6009: 
  6010:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  6011:       [InstrStage<1, [SLOT3]>], [1],
  6012:       [Hex_FWD]>,
  6013: 
  6014:     InstrItinData <tc_5502c366, /*tc_1*/
  6015:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6016:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6017: 
  6018:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  6019:       [InstrStage<1, [SLOT3]>], [],
  6020:       []>,
  6021: 
  6022:     InstrItinData <tc_556f6577, /*tc_3x*/
  6023:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  6024:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6025: 
  6026:     InstrItinData <tc_55a9a350, /*tc_st*/
  6027:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  6028:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6029: 
  6030:     InstrItinData <tc_55b33fda, /*tc_1*/
  6031:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  6032:       [Hex_FWD, Hex_FWD]>,
  6033: 
  6034:     InstrItinData <tc_56a124a7, /*tc_1*/
  6035:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  6036:       [Hex_FWD, Hex_FWD]>,
  6037: 
  6038:     InstrItinData <tc_57a55b54, /*tc_1*/
  6039:       [InstrStage<1, [SLOT3]>], [2, 2],
  6040:       [Hex_FWD, Hex_FWD]>,
  6041: 
  6042:     InstrItinData <tc_5944960d, /*tc_ld*/
  6043:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  6044:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6045: 
  6046:     InstrItinData <tc_59a7822c, /*tc_1*/
  6047:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  6048:       [Hex_FWD, Hex_FWD]>,
  6049: 
  6050:     InstrItinData <tc_5a222e89, /*tc_2early*/
  6051:       [InstrStage<1, [SLOT2]>], [1, 1],
  6052:       [Hex_FWD, Hex_FWD]>,
  6053: 
  6054:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  6055:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  6056:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6057: 
  6058:     InstrItinData <tc_5b347363, /*tc_1*/
  6059:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  6060:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6061: 
  6062:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  6063:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  6064:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6065: 
  6066:     InstrItinData <tc_5da50c4b, /*tc_1*/
  6067:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6068:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6069: 
  6070:     InstrItinData <tc_5deb5e47, /*tc_st*/
  6071:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  6072:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6073: 
  6074:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  6075:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  6076:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6077: 
  6078:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
  6079:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
  6080:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6081: 
  6082:     InstrItinData <tc_60e324ff, /*tc_1*/
  6083:       [InstrStage<1, [SLOT2]>], [2],
  6084:       [Hex_FWD]>,
  6085: 
  6086:     InstrItinData <tc_63567288, /*tc_2latepred*/
  6087:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
  6088:       [Hex_FWD]>,
  6089: 
  6090:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  6091:       [InstrStage<1, [SLOT0]>], [4, 1],
  6092:       [Hex_FWD, Hex_FWD]>,
  6093: 
  6094:     InstrItinData <tc_651cbe02, /*tc_1*/
  6095:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6096:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6097: 
  6098:     InstrItinData <tc_65279839, /*tc_2*/
  6099:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  6100:       [Hex_FWD, Hex_FWD]>,
  6101: 
  6102:     InstrItinData <tc_65cbd974, /*tc_st*/
  6103:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  6104:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6105: 
  6106:     InstrItinData <tc_69bfb303, /*tc_3*/
  6107:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  6108:       [Hex_FWD, Hex_FWD]>,
  6109: 
  6110:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  6111:       [InstrStage<1, [SLOT3]>], [4, 1],
  6112:       [Hex_FWD, Hex_FWD]>,
  6113: 
  6114:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  6115:       [InstrStage<1, [SLOT3]>], [4, 1],
  6116:       [Hex_FWD, Hex_FWD]>,
  6117: 
  6118:     InstrItinData <tc_6d861a95, /*tc_3x*/
  6119:       [InstrStage<1, [SLOT3]>], [2, 1],
  6120:       [Hex_FWD, Hex_FWD]>,
  6121: 
  6122:     InstrItinData <tc_6e20402a, /*tc_st*/
  6123:       [InstrStage<1, [SLOT0]>], [2, 3],
  6124:       [Hex_FWD, Hex_FWD]>,
  6125: 
  6126:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  6127:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  6128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6129: 
  6130:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  6131:       [InstrStage<1, [SLOT0]>], [1, 1],
  6132:       [Hex_FWD, Hex_FWD]>,
  6133: 
  6134:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  6135:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  6136:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6137: 
  6138:     InstrItinData <tc_711c805f, /*tc_1*/
  6139:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  6140:       [Hex_FWD, Hex_FWD]>,
  6141: 
  6142:     InstrItinData <tc_713b66bf, /*tc_1*/
  6143:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  6144:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6145: 
  6146:     InstrItinData <tc_7401744f, /*tc_2*/
  6147:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  6148:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6149: 
  6150:     InstrItinData <tc_7476d766, /*tc_3stall*/
  6151:       [InstrStage<1, [SLOT3]>], [4, 2],
  6152:       [Hex_FWD, Hex_FWD]>,
  6153: 
  6154:     InstrItinData <tc_74a42bda, /*tc_ld*/
  6155:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  6156:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6157: 
  6158:     InstrItinData <tc_759e57be, /*tc_3stall*/
  6159:       [InstrStage<1, [SLOT2]>], [4, 1],
  6160:       [Hex_FWD, Hex_FWD]>,
  6161: 
  6162:     InstrItinData <tc_76bb5435, /*tc_ld*/
  6163:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  6164:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6165: 
  6166:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  6167:       [InstrStage<1, [SLOT2]>], [1],
  6168:       [Hex_FWD]>,
  6169: 
  6170:     InstrItinData <tc_77f94a5e, /*tc_st*/
  6171:       [InstrStage<1, [SLOT0]>], [],
  6172:       []>,
  6173: 
  6174:     InstrItinData <tc_788b1d09, /*tc_3x*/
  6175:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  6176:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6177: 
  6178:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  6179:       [InstrStage<1, [SLOT0]>], [],
  6180:       []>,
  6181: 
  6182:     InstrItinData <tc_7af3a37e, /*tc_st*/
  6183:       [InstrStage<1, [SLOT0]>], [1, 3],
  6184:       [Hex_FWD, Hex_FWD]>,
  6185: 
  6186:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  6187:       [InstrStage<1, [SLOT0]>], [3, 2],
  6188:       [Hex_FWD, Hex_FWD]>,
  6189: 
  6190:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  6191:       [InstrStage<1, [SLOT0]>], [3],
  6192:       [Hex_FWD]>,
  6193: 
  6194:     InstrItinData <tc_7c31e19a, /*tc_st*/
  6195:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  6196:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6197: 
  6198:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  6199:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  6200:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6201: 
  6202:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  6203:       [InstrStage<1, [SLOT3]>], [4, 1],
  6204:       [Hex_FWD, Hex_FWD]>,
  6205: 
  6206:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  6207:       [InstrStage<1, [SLOT3]>], [],
  6208:       []>,
  6209: 
  6210:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  6211:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  6212:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6213: 
  6214:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  6215:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  6216:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6217: 
  6218:     InstrItinData <tc_8035e91f, /*tc_st*/
  6219:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
  6220:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6221: 
  6222:     InstrItinData <tc_822c3c68, /*tc_ld*/
  6223:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  6224:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6225: 
  6226:     InstrItinData <tc_829d8a86, /*tc_st*/
  6227:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  6228:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6229: 
  6230:     InstrItinData <tc_838c4d7a, /*tc_st*/
  6231:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  6232:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6233: 
  6234:     InstrItinData <tc_84a7500d, /*tc_2*/
  6235:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  6236:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6237: 
  6238:     InstrItinData <tc_86173609, /*tc_2latepred*/
  6239:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  6240:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6241: 
  6242:     InstrItinData <tc_887d1bb7, /*tc_st*/
  6243:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
  6244:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6245: 
  6246:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  6247:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  6248:       [Hex_FWD, Hex_FWD]>,
  6249: 
  6250:     InstrItinData <tc_8a825db2, /*tc_2*/
  6251:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6252:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6253: 
  6254:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
  6255:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  6256:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6257: 
  6258:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  6259:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
  6260:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6261: 
  6262:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  6263:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  6264:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6265: 
  6266:     InstrItinData <tc_9124c04f, /*tc_1*/
  6267:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  6268:       [Hex_FWD, Hex_FWD]>,
  6269: 
  6270:     InstrItinData <tc_92240447, /*tc_st*/
  6271:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  6272:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6273: 
  6274:     InstrItinData <tc_934753bb, /*tc_ld*/
  6275:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
  6276:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6277: 
  6278:     InstrItinData <tc_937dd41c, /*tc_ld*/
  6279:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  6280:       []>,
  6281: 
  6282:     InstrItinData <tc_9406230a, /*tc_3x*/
  6283:       [InstrStage<1, [SLOT3]>], [2, 1],
  6284:       [Hex_FWD, Hex_FWD]>,
  6285: 
  6286:     InstrItinData <tc_95a33176, /*tc_2*/
  6287:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  6288:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6289: 
  6290:     InstrItinData <tc_95f43c5e, /*tc_3*/
  6291:       [InstrStage<1, [SLOT2]>], [1],
  6292:       [Hex_FWD]>,
  6293: 
  6294:     InstrItinData <tc_96ef76ef, /*tc_st*/
  6295:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  6296:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6297: 
  6298:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  6299:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  6300:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6301: 
  6302:     InstrItinData <tc_9783714b, /*tc_4x*/
  6303:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  6304:       [Hex_FWD, Hex_FWD]>,
  6305: 
  6306:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  6307:       [InstrStage<1, [SLOT2]>], [4, 1],
  6308:       [Hex_FWD, Hex_FWD]>,
  6309: 
  6310:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
  6311:       [InstrStage<1, [SLOT2]>], [],
  6312:       []>,
  6313: 
  6314:     InstrItinData <tc_9b3c0462, /*tc_2*/
  6315:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6316:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6317: 
  6318:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  6319:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  6320:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6321: 
  6322:     InstrItinData <tc_9c52f549, /*tc_1*/
  6323:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  6324:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6325: 
  6326:     InstrItinData <tc_9e27f2f9, /*tc_1*/
  6327:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  6328:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6329: 
  6330:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  6331:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  6332:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6333: 
  6334:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  6335:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  6336:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6337: 
  6338:     InstrItinData <tc_9edefe01, /*tc_st*/
  6339:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
  6340:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6341: 
  6342:     InstrItinData <tc_9f6cd987, /*tc_1*/
  6343:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  6344:       [Hex_FWD, Hex_FWD]>,
  6345: 
  6346:     InstrItinData <tc_a08b630b, /*tc_2*/
  6347:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6349: 
  6350:     InstrItinData <tc_a1297125, /*tc_1*/
  6351:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6353: 
  6354:     InstrItinData <tc_a154b476, /*tc_3x*/
  6355:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  6356:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6357: 
  6358:     InstrItinData <tc_a2b365d2, /*tc_st*/
  6359:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
  6360:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6361: 
  6362:     InstrItinData <tc_a3070909, /*tc_3stall*/
  6363:       [InstrStage<1, [SLOT0]>], [1, 1],
  6364:       [Hex_FWD, Hex_FWD]>,
  6365: 
  6366:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  6367:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  6368:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6369: 
  6370:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  6371:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  6372:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6373: 
  6374:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  6375:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  6376:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6377: 
  6378:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  6379:       [InstrStage<1, [SLOT0]>], [],
  6380:       []>,
  6381: 
  6382:     InstrItinData <tc_a724463d, /*tc_3stall*/
  6383:       [InstrStage<1, [SLOT0]>], [4, 1],
  6384:       [Hex_FWD, Hex_FWD]>,
  6385: 
  6386:     InstrItinData <tc_a7a13fac, /*tc_1*/
  6387:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  6388:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6389: 
  6390:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  6391:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  6392:       [Hex_FWD, Hex_FWD]>,
  6393: 
  6394:     InstrItinData <tc_a9edeffa, /*tc_st*/
  6395:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  6396:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6397: 
  6398:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  6399:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  6400:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6401: 
  6402:     InstrItinData <tc_ac65613f, /*tc_ld*/
  6403:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  6404:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6405: 
  6406:     InstrItinData <tc_addc37a8, /*tc_st*/
  6407:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  6408:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6409: 
  6410:     InstrItinData <tc_ae5babd7, /*tc_st*/
  6411:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  6412:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6413: 
  6414:     InstrItinData <tc_aee6250c, /*tc_ld*/
  6415:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  6416:       [Hex_FWD, Hex_FWD]>,
  6417: 
  6418:     InstrItinData <tc_af6af259, /*tc_ld*/
  6419:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  6420:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6421: 
  6422:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  6423:       [InstrStage<1, [SLOT0]>], [1],
  6424:       [Hex_FWD]>,
  6425: 
  6426:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
  6427:       [InstrStage<1, [SLOT3]>], [1, 1],
  6428:       [Hex_FWD, Hex_FWD]>,
  6429: 
  6430:     InstrItinData <tc_b3d46584, /*tc_st*/
  6431:       [InstrStage<1, [SLOT0]>], [],
  6432:       []>,
  6433: 
  6434:     InstrItinData <tc_b3d46584, /*tc_st*/
  6435:       [InstrStage<1, [SLOT0]>], [],
  6436:       []>,
  6437: 
  6438:     InstrItinData <tc_b4dc7630, /*tc_st*/
  6439:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
  6440:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6441: 
  6442:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  6443:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  6444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6445: 
  6446:     InstrItinData <tc_b837298f, /*tc_1*/
  6447:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  6448:       []>,
  6449: 
  6450:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  6451:       [InstrStage<1, [SLOT2]>], [],
  6452:       []>,
  6453: 
  6454:     InstrItinData <tc_ba9255a6, /*tc_st*/
  6455:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
  6456:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6457: 
  6458:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  6459:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
  6460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6461: 
  6462:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  6463:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  6464:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6465: 
  6466:     InstrItinData <tc_bb831a7c, /*tc_2*/
  6467:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  6468:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6469: 
  6470:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  6471:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  6472:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6473: 
  6474:     InstrItinData <tc_c20701f0, /*tc_2*/
  6475:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6476:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6477: 
  6478:     InstrItinData <tc_c21d7447, /*tc_3x*/
  6479:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  6480:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6481: 
  6482:     InstrItinData <tc_c57d9f39, /*tc_1*/
  6483:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  6484:       [Hex_FWD, Hex_FWD]>,
  6485: 
  6486:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  6487:       [InstrStage<1, [SLOT0]>], [],
  6488:       []>,
  6489: 
  6490:     InstrItinData <tc_ce59038e, /*tc_st*/
  6491:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  6492:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6493: 
  6494:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  6495:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  6496:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6497: 
  6498:     InstrItinData <tc_d03278fd, /*tc_st*/
  6499:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  6500:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 6501-7000 / 第 6501-7000 行

```tablegen
  6501: 
  6502:     InstrItinData <tc_d234b61a, /*tc_st*/
  6503:       [InstrStage<1, [SLOT0]>], [1],
  6504:       [Hex_FWD]>,
  6505: 
  6506:     InstrItinData <tc_d33e5eee, /*tc_1*/
  6507:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  6508:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6509: 
  6510:     InstrItinData <tc_d3632d88, /*tc_2*/
  6511:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  6512:       [Hex_FWD, Hex_FWD]>,
  6513: 
  6514:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  6515:       [InstrStage<1, [SLOT0]>], [1],
  6516:       [Hex_FWD]>,
  6517: 
  6518:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  6519:       [InstrStage<1, [SLOT2]>], [2],
  6520:       [Hex_FWD]>,
  6521: 
  6522:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  6523:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  6524:       [Hex_FWD, Hex_FWD]>,
  6525: 
  6526:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  6527:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  6528:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6529: 
  6530:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  6531:       [InstrStage<1, [SLOT3]>], [2, 1],
  6532:       [Hex_FWD, Hex_FWD]>,
  6533: 
  6534:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  6535:       [InstrStage<1, [SLOT3]>], [1],
  6536:       [Hex_FWD]>,
  6537: 
  6538:     InstrItinData <tc_db596beb, /*tc_3x*/
  6539:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  6540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6541: 
  6542:     InstrItinData <tc_db96aa6b, /*tc_st*/
  6543:       [InstrStage<1, [SLOT0]>], [1],
  6544:       [Hex_FWD]>,
  6545: 
  6546:     InstrItinData <tc_dc51281d, /*tc_3*/
  6547:       [InstrStage<1, [SLOT2]>], [2, 1],
  6548:       [Hex_FWD, Hex_FWD]>,
  6549: 
  6550:     InstrItinData <tc_decdde8a, /*tc_1*/
  6551:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  6552:       [Hex_FWD]>,
  6553: 
  6554:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  6555:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
  6556:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6557: 
  6558:     InstrItinData <tc_e3d699e3, /*tc_2*/
  6559:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  6560:       [Hex_FWD, Hex_FWD]>,
  6561: 
  6562:     InstrItinData <tc_e60def48, /*tc_1*/
  6563:       [InstrStage<1, [SLOT2]>], [2],
  6564:       [Hex_FWD]>,
  6565: 
  6566:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  6567:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  6568:       [Hex_FWD, Hex_FWD]>,
  6569: 
  6570:     InstrItinData <tc_ed03645c, /*tc_1*/
  6571:       [InstrStage<1, [SLOT2]>], [3, 2],
  6572:       [Hex_FWD, Hex_FWD]>,
  6573: 
  6574:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  6575:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  6576:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6577: 
  6578:     InstrItinData <tc_eed07714, /*tc_ld*/
  6579:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  6580:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6581: 
  6582:     InstrItinData <tc_eeda4109, /*tc_1*/
  6583:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  6584:       [Hex_FWD, Hex_FWD]>,
  6585: 
  6586:     InstrItinData <tc_ef921005, /*tc_1*/
  6587:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  6588:       [Hex_FWD, Hex_FWD]>,
  6589: 
  6590:     InstrItinData <tc_f098b237, /*tc_2*/
  6591:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6592:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6593: 
  6594:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  6595:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  6596:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6597: 
  6598:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  6599:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  6600:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6601: 
  6602:     InstrItinData <tc_f34c1c21, /*tc_2*/
  6603:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6604:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6605: 
  6606:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  6607:       [InstrStage<1, [SLOT0]>], [2],
  6608:       [Hex_FWD]>,
  6609: 
  6610:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
  6611:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
  6612:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6613: 
  6614:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  6615:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  6616:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6617: 
  6618:     InstrItinData <tc_f7569068, /*tc_4x*/
  6619:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  6620:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6621: 
  6622:     InstrItinData <tc_f97707c1, /*tc_1*/
  6623:       [InstrStage<1, [SLOT2]>], [2],
  6624:       [Hex_FWD]>,
  6625: 
  6626:     InstrItinData <tc_f999c66e, /*tc_1*/
  6627:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  6628:       [Hex_FWD, Hex_FWD]>,
  6629: 
  6630:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  6631:       [InstrStage<1, [SLOT3]>], [4, 2],
  6632:       [Hex_FWD, Hex_FWD]>,
  6633: 
  6634:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  6635:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  6636:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  6637:   ];
  6638: }
  6639: 
  6640: class DepScalarItinV67T {
  6641:   list<InstrItinData> DepScalarItinV67T_list = [
  6642:     InstrItinData <tc_011e0e9d, /*tc_st*/
  6643:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  6644:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6645: 
  6646:     InstrItinData <tc_01d44cb2, /*tc_2*/
  6647:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6648:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6649: 
  6650:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  6651:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  6652:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6653: 
  6654:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  6655:       [InstrStage<1, [SLOT3]>], [5, 1, 1],
  6656:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6657: 
  6658:     InstrItinData <tc_0655b949, /*tc_st*/
  6659:       [InstrStage<1, [SLOT0]>], [2, 3],
  6660:       [Hex_FWD, Hex_FWD]>,
  6661: 
  6662:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  6663:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2],
  6664:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6665: 
  6666:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  6667:       [InstrStage<1, [SLOT3]>], [5, 2, 1, 1],
  6668:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6669: 
  6670:     InstrItinData <tc_0a43be35, /*tc_3x*/
  6671:       [InstrStage<1, [SLOT3]>], [1],
  6672:       [Hex_FWD]>,
  6673: 
  6674:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  6675:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  6676:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6677: 
  6678:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
  6679:       [InstrStage<1, [SLOT2]>], [1],
  6680:       [Hex_FWD]>,
  6681: 
  6682:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  6683:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6685: 
  6686:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  6687:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  6688:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6689: 
  6690:     InstrItinData <tc_112d30d6, /*tc_1*/
  6691:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2],
  6692:       [Hex_FWD]>,
  6693: 
  6694:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  6695:       [InstrStage<1, [SLOT0]>], [2],
  6696:       [Hex_FWD]>,
  6697: 
  6698:     InstrItinData <tc_1248597c, /*tc_3x*/
  6699:       [InstrStage<1, [SLOT3]>], [2, 2],
  6700:       [Hex_FWD, Hex_FWD]>,
  6701: 
  6702:     InstrItinData <tc_139ef484, /*tc_3stall*/
  6703:       [InstrStage<1, [SLOT2]>], [1, 1],
  6704:       [Hex_FWD, Hex_FWD]>,
  6705: 
  6706:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  6707:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
  6708:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6709: 
  6710:     InstrItinData <tc_151bf368, /*tc_1*/
  6711:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  6712:       [Hex_FWD, Hex_FWD]>,
  6713: 
  6714:     InstrItinData <tc_158aa3f7, /*tc_st*/
  6715:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  6716:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6717: 
  6718:     InstrItinData <tc_197dce51, /*tc_3x*/
  6719:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  6720:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6721: 
  6722:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  6723:       [InstrStage<1, [SLOT0]>], [3],
  6724:       [Hex_FWD]>,
  6725: 
  6726:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  6727:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2, 2],
  6728:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6729: 
  6730:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  6731:       [InstrStage<1, [SLOT0]>], [4, 3, 2, 1, 2],
  6732:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6733: 
  6734:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  6735:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  6736:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6737: 
  6738:     InstrItinData <tc_1fcb8495, /*tc_2*/
  6739:       [InstrStage<1, [SLOT3]>], [4, 2, 2],
  6740:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6741: 
  6742:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  6743:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  6744:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6745: 
  6746:     InstrItinData <tc_20131976, /*tc_2*/
  6747:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  6748:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6749: 
  6750:     InstrItinData <tc_2237d952, /*tc_ld*/
  6751:       [InstrStage<1, [SLOT0]>], [1, 2],
  6752:       [Hex_FWD, Hex_FWD]>,
  6753: 
  6754:     InstrItinData <tc_23708a21, /*tc_1*/
  6755:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [],
  6756:       []>,
  6757: 
  6758:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  6759:       [InstrStage<1, [SLOT0]>], [4, 1],
  6760:       [Hex_FWD, Hex_FWD]>,
  6761: 
  6762:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  6763:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  6764:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6765: 
  6766:     InstrItinData <tc_24f426ab, /*tc_1*/
  6767:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2, 2],
  6768:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6769: 
  6770:     InstrItinData <tc_27106296, /*tc_3x*/
  6771:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  6772:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6773: 
  6774:     InstrItinData <tc_280f7fe1, /*tc_st*/
  6775:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  6776:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6777: 
  6778:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  6779:       [InstrStage<1, [SLOT3]>], [1, 1],
  6780:       [Hex_FWD, Hex_FWD]>,
  6781: 
  6782:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  6783:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2],
  6784:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6785: 
  6786:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  6787:       [InstrStage<1, [SLOT3]>], [1],
  6788:       [Hex_FWD]>,
  6789: 
  6790:     InstrItinData <tc_2f573607, /*tc_1*/
  6791:       [InstrStage<1, [SLOT2]>], [2, 2],
  6792:       [Hex_FWD, Hex_FWD]>,
  6793: 
  6794:     InstrItinData <tc_33e7e673, /*tc_2early*/
  6795:       [InstrStage<1, [SLOT2]>], [],
  6796:       []>,
  6797: 
  6798:     InstrItinData <tc_362b0be2, /*tc_3*/
  6799:       [InstrStage<1, [SLOT2]>], [1],
  6800:       [Hex_FWD]>,
  6801: 
  6802:     InstrItinData <tc_38382228, /*tc_3x*/
  6803:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  6804:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6805: 
  6806:     InstrItinData <tc_388f9897, /*tc_1*/
  6807:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  6808:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6809: 
  6810:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  6811:       [InstrStage<1, [SLOT3]>], [4, 4, 2, 1, 1],
  6812:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6813: 
  6814:     InstrItinData <tc_3d14a17b, /*tc_1*/
  6815:       [InstrStage<1, [SLOT0]>], [3, 2],
  6816:       [Hex_FWD, Hex_FWD]>,
  6817: 
  6818:     InstrItinData <tc_3edca78f, /*tc_2*/
  6819:       [InstrStage<1, [SLOT3]>], [4, 2],
  6820:       [Hex_FWD, Hex_FWD]>,
  6821: 
  6822:     InstrItinData <tc_3fbf1042, /*tc_1*/
  6823:       [InstrStage<1, [SLOT0]>], [3],
  6824:       [Hex_FWD]>,
  6825: 
  6826:     InstrItinData <tc_407e96f9, /*tc_1*/
  6827:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6828:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6829: 
  6830:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  6831:       [InstrStage<1, [SLOT0]>], [3, 1],
  6832:       [Hex_FWD, Hex_FWD]>,
  6833: 
  6834:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  6835:       [InstrStage<1, [SLOT0]>], [4, 1, 2],
  6836:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6837: 
  6838:     InstrItinData <tc_42ff66ba, /*tc_1*/
  6839:       [InstrStage<1, [SLOT2]>], [2, 2],
  6840:       [Hex_FWD, Hex_FWD]>,
  6841: 
  6842:     InstrItinData <tc_442395f3, /*tc_2latepred*/
  6843:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 3, 2, 2],
  6844:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6845: 
  6846:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
  6847:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 1],
  6848:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6849: 
  6850:     InstrItinData <tc_44d5a428, /*tc_st*/
  6851:       [InstrStage<1, [SLOT0]>], [1, 2],
  6852:       [Hex_FWD, Hex_FWD]>,
  6853: 
  6854:     InstrItinData <tc_44fffc58, /*tc_3*/
  6855:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  6856:       [Hex_FWD]>,
  6857: 
  6858:     InstrItinData <tc_45791fb8, /*tc_ld*/
  6859:       [InstrStage<1, [SLOT0]>], [4, 2, 1, 1, 2],
  6860:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6861: 
  6862:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  6863:       [InstrStage<1, [SLOT2]>], [2],
  6864:       [Hex_FWD]>,
  6865: 
  6866:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  6867:       [InstrStage<1, [SLOT3]>], [4, 1],
  6868:       [Hex_FWD, Hex_FWD]>,
  6869: 
  6870:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
  6871:       [InstrStage<1, [SLOT3]>], [4, 1],
  6872:       [Hex_FWD, Hex_FWD]>,
  6873: 
  6874:     InstrItinData <tc_4a55d03c, /*tc_1*/
  6875:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6876:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6877: 
  6878:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
  6879:       [InstrStage<1, [SLOT3]>], [2, 2],
  6880:       [Hex_FWD, Hex_FWD]>,
  6881: 
  6882:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  6883:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  6884:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6885: 
  6886:     InstrItinData <tc_4bf903b0, /*tc_st*/
  6887:       [InstrStage<1, [SLOT0]>], [3],
  6888:       [Hex_FWD]>,
  6889: 
  6890:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  6891:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 1],
  6892:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6893: 
  6894:     InstrItinData <tc_512b1653, /*tc_st*/
  6895:       [InstrStage<1, [SLOT0]>], [1, 2],
  6896:       [Hex_FWD, Hex_FWD]>,
  6897: 
  6898:     InstrItinData <tc_53c851ab, /*tc_3stall*/
  6899:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
  6900:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6901: 
  6902:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  6903:       [InstrStage<1, [SLOT3]>], [1],
  6904:       [Hex_FWD]>,
  6905: 
  6906:     InstrItinData <tc_5502c366, /*tc_1*/
  6907:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6909: 
  6910:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  6911:       [InstrStage<1, [SLOT3]>], [],
  6912:       []>,
  6913: 
  6914:     InstrItinData <tc_556f6577, /*tc_3x*/
  6915:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  6916:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6917: 
  6918:     InstrItinData <tc_55a9a350, /*tc_st*/
  6919:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  6920:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6921: 
  6922:     InstrItinData <tc_55b33fda, /*tc_1*/
  6923:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  6924:       [Hex_FWD, Hex_FWD]>,
  6925: 
  6926:     InstrItinData <tc_56a124a7, /*tc_1*/
  6927:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2],
  6928:       [Hex_FWD, Hex_FWD]>,
  6929: 
  6930:     InstrItinData <tc_57a55b54, /*tc_1*/
  6931:       [InstrStage<1, [SLOT3]>], [2, 2],
  6932:       [Hex_FWD, Hex_FWD]>,
  6933: 
  6934:     InstrItinData <tc_5944960d, /*tc_ld*/
  6935:       [InstrStage<1, [SLOT0]>], [1, 1, 2],
  6936:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6937: 
  6938:     InstrItinData <tc_59a7822c, /*tc_1*/
  6939:       [InstrStage<1, [SLOT0]>], [2, 2],
  6940:       [Hex_FWD, Hex_FWD]>,
  6941: 
  6942:     InstrItinData <tc_5a222e89, /*tc_2early*/
  6943:       [InstrStage<1, [SLOT2]>], [1, 1],
  6944:       [Hex_FWD, Hex_FWD]>,
  6945: 
  6946:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  6947:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  6948:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6949: 
  6950:     InstrItinData <tc_5b347363, /*tc_1*/
  6951:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  6952:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6953: 
  6954:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  6955:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 2],
  6956:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6957: 
  6958:     InstrItinData <tc_5da50c4b, /*tc_1*/
  6959:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6960:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6961: 
  6962:     InstrItinData <tc_5deb5e47, /*tc_st*/
  6963:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  6964:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6965: 
  6966:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  6967:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2],
  6968:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6969: 
  6970:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
  6971:       [InstrStage<1, [SLOT0]>], [4, 4, 3, 1, 2],
  6972:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6973: 
  6974:     InstrItinData <tc_60e324ff, /*tc_1*/
  6975:       [InstrStage<1, [SLOT2]>], [2],
  6976:       [Hex_FWD]>,
  6977: 
  6978:     InstrItinData <tc_63567288, /*tc_2latepred*/
  6979:       [InstrStage<1, [SLOT0]>], [4],
  6980:       [Hex_FWD]>,
  6981: 
  6982:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  6983:       [InstrStage<1, [SLOT0]>], [4, 1],
  6984:       [Hex_FWD, Hex_FWD]>,
  6985: 
  6986:     InstrItinData <tc_651cbe02, /*tc_1*/
  6987:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  6988:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  6989: 
  6990:     InstrItinData <tc_65279839, /*tc_2*/
  6991:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  6992:       [Hex_FWD, Hex_FWD]>,
  6993: 
  6994:     InstrItinData <tc_65cbd974, /*tc_st*/
  6995:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2],
  6996:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  6997: 
  6998:     InstrItinData <tc_69bfb303, /*tc_3*/
  6999:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  7000:       [Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV67T, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV67T; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV67T 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV67T 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 7001-7500 / 第 7001-7500 行

```tablegen
  7001: 
  7002:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  7003:       [InstrStage<1, [SLOT3]>], [4, 1],
  7004:       [Hex_FWD, Hex_FWD]>,
  7005: 
  7006:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  7007:       [InstrStage<1, [SLOT3]>], [4, 1],
  7008:       [Hex_FWD, Hex_FWD]>,
  7009: 
  7010:     InstrItinData <tc_6d861a95, /*tc_3x*/
  7011:       [InstrStage<1, [SLOT3]>], [2, 1],
  7012:       [Hex_FWD, Hex_FWD]>,
  7013: 
  7014:     InstrItinData <tc_6e20402a, /*tc_st*/
  7015:       [InstrStage<1, [SLOT0]>], [2, 3],
  7016:       [Hex_FWD, Hex_FWD]>,
  7017: 
  7018:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  7019:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  7020:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7021: 
  7022:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  7023:       [InstrStage<1, [SLOT0]>], [1, 1],
  7024:       [Hex_FWD, Hex_FWD]>,
  7025: 
  7026:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  7027:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  7028:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7029: 
  7030:     InstrItinData <tc_711c805f, /*tc_1*/
  7031:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2],
  7032:       [Hex_FWD, Hex_FWD]>,
  7033: 
  7034:     InstrItinData <tc_713b66bf, /*tc_1*/
  7035:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2],
  7036:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7037: 
  7038:     InstrItinData <tc_7401744f, /*tc_2*/
  7039:       [InstrStage<1, [SLOT3]>], [4, 4, 2, 2],
  7040:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7041: 
  7042:     InstrItinData <tc_7476d766, /*tc_3stall*/
  7043:       [InstrStage<1, [SLOT3]>], [4, 2],
  7044:       [Hex_FWD, Hex_FWD]>,
  7045: 
  7046:     InstrItinData <tc_74a42bda, /*tc_ld*/
  7047:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  7048:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7049: 
  7050:     InstrItinData <tc_759e57be, /*tc_3stall*/
  7051:       [InstrStage<1, [SLOT2]>], [4, 1],
  7052:       [Hex_FWD, Hex_FWD]>,
  7053: 
  7054:     InstrItinData <tc_76bb5435, /*tc_ld*/
  7055:       [InstrStage<1, [SLOT0]>], [4, 3, 2, 1, 2, 2],
  7056:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7057: 
  7058:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  7059:       [InstrStage<1, [SLOT2]>], [1],
  7060:       [Hex_FWD]>,
  7061: 
  7062:     InstrItinData <tc_77f94a5e, /*tc_st*/
  7063:       [InstrStage<1, [SLOT0]>], [],
  7064:       []>,
  7065: 
  7066:     InstrItinData <tc_788b1d09, /*tc_3x*/
  7067:       [InstrStage<1, [SLOT3]>], [4, 1, 1, 2],
  7068:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7069: 
  7070:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  7071:       [InstrStage<1, [SLOT0]>], [],
  7072:       []>,
  7073: 
  7074:     InstrItinData <tc_7af3a37e, /*tc_st*/
  7075:       [InstrStage<1, [SLOT0]>], [1, 3],
  7076:       [Hex_FWD, Hex_FWD]>,
  7077: 
  7078:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  7079:       [InstrStage<1, [SLOT0]>], [3, 2],
  7080:       [Hex_FWD, Hex_FWD]>,
  7081: 
  7082:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  7083:       [InstrStage<1, [SLOT0]>], [3],
  7084:       [Hex_FWD]>,
  7085: 
  7086:     InstrItinData <tc_7c31e19a, /*tc_st*/
  7087:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  7088:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7089: 
  7090:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  7091:       [InstrStage<1, [SLOT0]>], [4, 2, 2],
  7092:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7093: 
  7094:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  7095:       [InstrStage<1, [SLOT3]>], [4, 1],
  7096:       [Hex_FWD, Hex_FWD]>,
  7097: 
  7098:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  7099:       [InstrStage<1, [SLOT3]>], [],
  7100:       []>,
  7101: 
  7102:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  7103:       [InstrStage<1, [SLOT3]>], [5, 5, 1],
  7104:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7105: 
  7106:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  7107:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  7108:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7109: 
  7110:     InstrItinData <tc_8035e91f, /*tc_st*/
  7111:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  7112:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7113: 
  7114:     InstrItinData <tc_822c3c68, /*tc_ld*/
  7115:       [InstrStage<1, [SLOT0]>], [4, 3, 2],
  7116:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7117: 
  7118:     InstrItinData <tc_829d8a86, /*tc_st*/
  7119:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  7120:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7121: 
  7122:     InstrItinData <tc_838c4d7a, /*tc_st*/
  7123:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  7124:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7125: 
  7126:     InstrItinData <tc_84a7500d, /*tc_2*/
  7127:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 2, 2],
  7128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7129: 
  7130:     InstrItinData <tc_86173609, /*tc_2latepred*/
  7131:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 3, 2],
  7132:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7133: 
  7134:     InstrItinData <tc_887d1bb7, /*tc_st*/
  7135:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  7136:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7137: 
  7138:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  7139:       [InstrStage<1, [SLOT0]>], [4, 2],
  7140:       [Hex_FWD, Hex_FWD]>,
  7141: 
  7142:     InstrItinData <tc_8a825db2, /*tc_2*/
  7143:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7144:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7145: 
  7146:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
  7147:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 2, 2],
  7148:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7149: 
  7150:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  7151:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  7152:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7153: 
  7154:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  7155:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  7156:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7157: 
  7158:     InstrItinData <tc_9124c04f, /*tc_1*/
  7159:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  7160:       [Hex_FWD, Hex_FWD]>,
  7161: 
  7162:     InstrItinData <tc_92240447, /*tc_st*/
  7163:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  7164:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7165: 
  7166:     InstrItinData <tc_934753bb, /*tc_ld*/
  7167:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
  7168:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7169: 
  7170:     InstrItinData <tc_937dd41c, /*tc_ld*/
  7171:       [InstrStage<1, [SLOT0]>], [],
  7172:       []>,
  7173: 
  7174:     InstrItinData <tc_9406230a, /*tc_3x*/
  7175:       [InstrStage<1, [SLOT3]>], [2, 1],
  7176:       [Hex_FWD, Hex_FWD]>,
  7177: 
  7178:     InstrItinData <tc_95a33176, /*tc_2*/
  7179:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 2, 2],
  7180:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7181: 
  7182:     InstrItinData <tc_95f43c5e, /*tc_3*/
  7183:       [InstrStage<1, [SLOT2]>], [1],
  7184:       [Hex_FWD]>,
  7185: 
  7186:     InstrItinData <tc_96ef76ef, /*tc_st*/
  7187:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  7188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7189: 
  7190:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  7191:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  7192:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7193: 
  7194:     InstrItinData <tc_9783714b, /*tc_4x*/
  7195:       [InstrStage<1, [SLOT3]>], [5, 1],
  7196:       [Hex_FWD, Hex_FWD]>,
  7197: 
  7198:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  7199:       [InstrStage<1, [SLOT2]>], [4, 1],
  7200:       [Hex_FWD, Hex_FWD]>,
  7201: 
  7202:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
  7203:       [InstrStage<1, [SLOT2]>], [],
  7204:       []>,
  7205: 
  7206:     InstrItinData <tc_9b3c0462, /*tc_2*/
  7207:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7208:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7209: 
  7210:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  7211:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  7212:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7213: 
  7214:     InstrItinData <tc_9c52f549, /*tc_1*/
  7215:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2],
  7216:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7217: 
  7218:     InstrItinData <tc_9e27f2f9, /*tc_1*/
  7219:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2, 2],
  7220:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7221: 
  7222:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  7223:       [InstrStage<1, [SLOT3]>], [5, 2, 1, 1],
  7224:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7225: 
  7226:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  7227:       [InstrStage<1, [SLOT3]>], [5, 2, 1, 1, 2],
  7228:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7229: 
  7230:     InstrItinData <tc_9edefe01, /*tc_st*/
  7231:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  7232:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7233: 
  7234:     InstrItinData <tc_9f6cd987, /*tc_1*/
  7235:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  7236:       [Hex_FWD, Hex_FWD]>,
  7237: 
  7238:     InstrItinData <tc_a08b630b, /*tc_2*/
  7239:       [InstrStage<1, [SLOT3]>], [4, 2, 2],
  7240:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7241: 
  7242:     InstrItinData <tc_a1297125, /*tc_1*/
  7243:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  7244:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7245: 
  7246:     InstrItinData <tc_a154b476, /*tc_3x*/
  7247:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 2],
  7248:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7249: 
  7250:     InstrItinData <tc_a2b365d2, /*tc_st*/
  7251:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  7252:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7253: 
  7254:     InstrItinData <tc_a3070909, /*tc_3stall*/
  7255:       [InstrStage<1, [SLOT0]>], [1, 1],
  7256:       [Hex_FWD, Hex_FWD]>,
  7257: 
  7258:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  7259:       [InstrStage<1, [SLOT0]>], [4, 2, 1, 2, 2],
  7260:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7261: 
  7262:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  7263:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1, 2],
  7264:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7265: 
  7266:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  7267:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2],
  7268:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7269: 
  7270:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  7271:       [InstrStage<1, [SLOT0]>], [],
  7272:       []>,
  7273: 
  7274:     InstrItinData <tc_a724463d, /*tc_3stall*/
  7275:       [InstrStage<1, [SLOT0]>], [4, 1],
  7276:       [Hex_FWD, Hex_FWD]>,
  7277: 
  7278:     InstrItinData <tc_a7a13fac, /*tc_1*/
  7279:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  7280:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7281: 
  7282:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  7283:       [InstrStage<1, [SLOT3]>], [4, 2],
  7284:       [Hex_FWD, Hex_FWD]>,
  7285: 
  7286:     InstrItinData <tc_a9edeffa, /*tc_st*/
  7287:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  7288:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7289: 
  7290:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  7291:       [InstrStage<1, [SLOT0]>], [4, 1, 2, 2],
  7292:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7293: 
  7294:     InstrItinData <tc_ac65613f, /*tc_ld*/
  7295:       [InstrStage<1, [SLOT0]>], [4, 3, 2, 2],
  7296:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7297: 
  7298:     InstrItinData <tc_addc37a8, /*tc_st*/
  7299:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  7300:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7301: 
  7302:     InstrItinData <tc_ae5babd7, /*tc_st*/
  7303:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  7304:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7305: 
  7306:     InstrItinData <tc_aee6250c, /*tc_ld*/
  7307:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  7308:       [Hex_FWD, Hex_FWD]>,
  7309: 
  7310:     InstrItinData <tc_af6af259, /*tc_ld*/
  7311:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 1],
  7312:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7313: 
  7314:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  7315:       [InstrStage<1, [SLOT0]>], [1],
  7316:       [Hex_FWD]>,
  7317: 
  7318:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
  7319:       [InstrStage<1, [SLOT3]>], [1, 1],
  7320:       [Hex_FWD, Hex_FWD]>,
  7321: 
  7322:     InstrItinData <tc_b3d46584, /*tc_st*/
  7323:       [InstrStage<1, [SLOT0]>], [],
  7324:       []>,
  7325: 
  7326:     InstrItinData <tc_b4dc7630, /*tc_st*/
  7327:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  7328:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7329: 
  7330:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  7331:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 1, 2],
  7332:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7333: 
  7334:     InstrItinData <tc_b837298f, /*tc_1*/
  7335:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [],
  7336:       []>,
  7337: 
  7338:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  7339:       [InstrStage<1, [SLOT2]>], [],
  7340:       []>,
  7341: 
  7342:     InstrItinData <tc_ba9255a6, /*tc_st*/
  7343:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  7344:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7345: 
  7346:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  7347:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  7348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7349: 
  7350:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  7351:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  7352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7353: 
  7354:     InstrItinData <tc_bb831a7c, /*tc_2*/
  7355:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2, 2],
  7356:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7357: 
  7358:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  7359:       [InstrStage<1, [SLOT0]>], [4, 1, 1, 2],
  7360:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7361: 
  7362:     InstrItinData <tc_c20701f0, /*tc_2*/
  7363:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7364:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7365: 
  7366:     InstrItinData <tc_c21d7447, /*tc_3x*/
  7367:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  7368:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7369: 
  7370:     InstrItinData <tc_c57d9f39, /*tc_1*/
  7371:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2],
  7372:       [Hex_FWD, Hex_FWD]>,
  7373: 
  7374:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  7375:       [InstrStage<1, [SLOT0]>], [],
  7376:       []>,
  7377: 
  7378:     InstrItinData <tc_ce59038e, /*tc_st*/
  7379:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  7380:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7381: 
  7382:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  7383:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  7384:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7385: 
  7386:     InstrItinData <tc_d03278fd, /*tc_st*/
  7387:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 2],
  7388:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7389: 
  7390:     InstrItinData <tc_d234b61a, /*tc_st*/
  7391:       [InstrStage<1, [SLOT0]>], [1],
  7392:       [Hex_FWD]>,
  7393: 
  7394:     InstrItinData <tc_d33e5eee, /*tc_1*/
  7395:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2],
  7396:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7397: 
  7398:     InstrItinData <tc_d3632d88, /*tc_2*/
  7399:       [InstrStage<1, [SLOT3]>], [4, 2],
  7400:       [Hex_FWD, Hex_FWD]>,
  7401: 
  7402:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  7403:       [InstrStage<1, [SLOT0]>], [1],
  7404:       [Hex_FWD]>,
  7405: 
  7406:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  7407:       [InstrStage<1, [SLOT2]>], [2],
  7408:       [Hex_FWD]>,
  7409: 
  7410:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  7411:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  7412:       [Hex_FWD, Hex_FWD]>,
  7413: 
  7414:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  7415:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  7416:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7417: 
  7418:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  7419:       [InstrStage<1, [SLOT3]>], [2, 1],
  7420:       [Hex_FWD, Hex_FWD]>,
  7421: 
  7422:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  7423:       [InstrStage<1, [SLOT3]>], [1],
  7424:       [Hex_FWD]>,
  7425: 
  7426:     InstrItinData <tc_db596beb, /*tc_3x*/
  7427:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  7428:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7429: 
  7430:     InstrItinData <tc_db96aa6b, /*tc_st*/
  7431:       [InstrStage<1, [SLOT0]>], [1],
  7432:       [Hex_FWD]>,
  7433: 
  7434:     InstrItinData <tc_dc51281d, /*tc_3*/
  7435:       [InstrStage<1, [SLOT2]>], [2, 1],
  7436:       [Hex_FWD, Hex_FWD]>,
  7437: 
  7438:     InstrItinData <tc_decdde8a, /*tc_1*/
  7439:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2],
  7440:       [Hex_FWD]>,
  7441: 
  7442:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  7443:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
  7444:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7445: 
  7446:     InstrItinData <tc_e3d699e3, /*tc_2*/
  7447:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  7448:       [Hex_FWD, Hex_FWD]>,
  7449: 
  7450:     InstrItinData <tc_e60def48, /*tc_1*/
  7451:       [InstrStage<1, [SLOT2]>], [2],
  7452:       [Hex_FWD]>,
  7453: 
  7454:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  7455:       [InstrStage<1, [SLOT0]>], [4, 1],
  7456:       [Hex_FWD, Hex_FWD]>,
  7457: 
  7458:     InstrItinData <tc_ed03645c, /*tc_1*/
  7459:       [InstrStage<1, [SLOT2]>], [3, 2],
  7460:       [Hex_FWD, Hex_FWD]>,
  7461: 
  7462:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  7463:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  7464:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7465: 
  7466:     InstrItinData <tc_eed07714, /*tc_ld*/
  7467:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  7468:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7469: 
  7470:     InstrItinData <tc_eeda4109, /*tc_1*/
  7471:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2],
  7472:       [Hex_FWD, Hex_FWD]>,
  7473: 
  7474:     InstrItinData <tc_ef921005, /*tc_1*/
  7475:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  7476:       [Hex_FWD, Hex_FWD]>,
  7477: 
  7478:     InstrItinData <tc_f098b237, /*tc_2*/
  7479:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7480:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7481: 
  7482:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  7483:       [InstrStage<1, [SLOT3]>], [4, 1, 1, 2],
  7484:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7485: 
  7486:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  7487:       [InstrStage<1, [SLOT3]>], [5, 1, 1],
  7488:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7489: 
  7490:     InstrItinData <tc_f34c1c21, /*tc_2*/
  7491:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7492:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7493: 
  7494:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  7495:       [InstrStage<1, [SLOT0]>], [2],
  7496:       [Hex_FWD]>,
  7497: 
  7498:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
  7499:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
  7500:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 7501-8000 / 第 7501-8000 行

```tablegen
  7501: 
  7502:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  7503:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  7504:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7505: 
  7506:     InstrItinData <tc_f7569068, /*tc_4x*/
  7507:       [InstrStage<1, [SLOT3]>], [5, 5, 1, 1],
  7508:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7509: 
  7510:     InstrItinData <tc_f97707c1, /*tc_1*/
  7511:       [InstrStage<1, [SLOT2]>], [2],
  7512:       [Hex_FWD]>,
  7513: 
  7514:     InstrItinData <tc_f999c66e, /*tc_1*/
  7515:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2],
  7516:       [Hex_FWD, Hex_FWD]>,
  7517: 
  7518:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  7519:       [InstrStage<1, [SLOT3]>], [4, 2],
  7520:       [Hex_FWD, Hex_FWD]>,
  7521: 
  7522:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  7523:       [InstrStage<1, [SLOT0]>], [4, 2, 1, 2],
  7524:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  7525:   ];
  7526: }
  7527: 
  7528: class DepScalarItinV68 {
  7529:   list<InstrItinData> DepScalarItinV68_list = [
  7530:     InstrItinData <tc_011e0e9d, /*tc_st*/
  7531:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  7532:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7533: 
  7534:     InstrItinData <tc_01d44cb2, /*tc_2*/
  7535:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7536:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7537: 
  7538:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  7539:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  7540:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7541: 
  7542:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  7543:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  7544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7545: 
  7546:     InstrItinData <tc_0655b949, /*tc_st*/
  7547:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
  7548:       [Hex_FWD, Hex_FWD]>,
  7549: 
  7550:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  7551:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  7552:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7553: 
  7554:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  7555:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  7556:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7557: 
  7558:     InstrItinData <tc_0a43be35, /*tc_3x*/
  7559:       [InstrStage<1, [SLOT3]>], [1],
  7560:       [Hex_FWD]>,
  7561: 
  7562:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  7563:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  7564:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7565: 
  7566:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
  7567:       [InstrStage<1, [SLOT2]>], [1],
  7568:       [Hex_FWD]>,
  7569: 
  7570:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  7571:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7573: 
  7574:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  7575:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  7576:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7577: 
  7578:     InstrItinData <tc_112d30d6, /*tc_1*/
  7579:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  7580:       [Hex_FWD]>,
  7581: 
  7582:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  7583:       [InstrStage<1, [SLOT0]>], [2],
  7584:       [Hex_FWD]>,
  7585: 
  7586:     InstrItinData <tc_1248597c, /*tc_3x*/
  7587:       [InstrStage<1, [SLOT3]>], [2, 2],
  7588:       [Hex_FWD, Hex_FWD]>,
  7589: 
  7590:     InstrItinData <tc_139ef484, /*tc_3stall*/
  7591:       [InstrStage<1, [SLOT2]>], [1, 1],
  7592:       [Hex_FWD, Hex_FWD]>,
  7593: 
  7594:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  7595:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
  7596:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7597: 
  7598:     InstrItinData <tc_151bf368, /*tc_1*/
  7599:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  7600:       [Hex_FWD, Hex_FWD]>,
  7601: 
  7602:     InstrItinData <tc_158aa3f7, /*tc_st*/
  7603:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  7604:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7605: 
  7606:     InstrItinData <tc_197dce51, /*tc_3x*/
  7607:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  7608:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7609: 
  7610:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  7611:       [InstrStage<1, [SLOT0]>], [3],
  7612:       [Hex_FWD]>,
  7613: 
  7614:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  7615:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  7616:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7617: 
  7618:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  7619:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  7620:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7621: 
  7622:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  7623:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  7624:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7625: 
  7626:     InstrItinData <tc_1fcb8495, /*tc_2*/
  7627:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7628:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7629: 
  7630:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  7631:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
  7632:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7633: 
  7634:     InstrItinData <tc_20131976, /*tc_2*/
  7635:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  7636:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7637: 
  7638:     InstrItinData <tc_2237d952, /*tc_ld*/
  7639:       [InstrStage<1, [SLOT0]>], [1, 2],
  7640:       [Hex_FWD, Hex_FWD]>,
  7641: 
  7642:     InstrItinData <tc_23708a21, /*tc_1*/
  7643:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  7644:       []>,
  7645: 
  7646:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  7647:       [InstrStage<1, [SLOT0]>], [4, 1],
  7648:       [Hex_FWD, Hex_FWD]>,
  7649: 
  7650:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  7651:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  7652:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7653: 
  7654:     InstrItinData <tc_24f426ab, /*tc_1*/
  7655:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  7656:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7657: 
  7658:     InstrItinData <tc_27106296, /*tc_3x*/
  7659:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  7660:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7661: 
  7662:     InstrItinData <tc_280f7fe1, /*tc_st*/
  7663:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
  7664:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7665: 
  7666:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  7667:       [InstrStage<1, [SLOT3]>], [1, 1],
  7668:       [Hex_FWD, Hex_FWD]>,
  7669: 
  7670:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  7671:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  7672:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7673: 
  7674:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  7675:       [InstrStage<1, [SLOT3]>], [1],
  7676:       [Hex_FWD]>,
  7677: 
  7678:     InstrItinData <tc_2f573607, /*tc_1*/
  7679:       [InstrStage<1, [SLOT2]>], [2, 2],
  7680:       [Hex_FWD, Hex_FWD]>,
  7681: 
  7682:     InstrItinData <tc_33e7e673, /*tc_2early*/
  7683:       [InstrStage<1, [SLOT2]>], [],
  7684:       []>,
  7685: 
  7686:     InstrItinData <tc_362b0be2, /*tc_3*/
  7687:       [InstrStage<1, [SLOT2]>], [1],
  7688:       [Hex_FWD]>,
  7689: 
  7690:     InstrItinData <tc_38382228, /*tc_3x*/
  7691:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  7692:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7693: 
  7694:     InstrItinData <tc_388f9897, /*tc_1*/
  7695:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  7696:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7697: 
  7698:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  7699:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
  7700:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7701: 
  7702:     InstrItinData <tc_3d14a17b, /*tc_1*/
  7703:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  7704:       [Hex_FWD, Hex_FWD]>,
  7705: 
  7706:     InstrItinData <tc_3edca78f, /*tc_2*/
  7707:       [InstrStage<1, [SLOT3]>], [4, 2],
  7708:       [Hex_FWD, Hex_FWD]>,
  7709: 
  7710:     InstrItinData <tc_3fbf1042, /*tc_1*/
  7711:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  7712:       [Hex_FWD]>,
  7713: 
  7714:     InstrItinData <tc_407e96f9, /*tc_1*/
  7715:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  7716:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7717: 
  7718:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  7719:       [InstrStage<1, [SLOT0]>], [3, 1],
  7720:       [Hex_FWD, Hex_FWD]>,
  7721: 
  7722:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  7723:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  7724:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7725: 
  7726:     InstrItinData <tc_42ff66ba, /*tc_1*/
  7727:       [InstrStage<1, [SLOT2]>], [2, 2],
  7728:       [Hex_FWD, Hex_FWD]>,
  7729: 
  7730:     InstrItinData <tc_442395f3, /*tc_2latepred*/
  7731:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
  7732:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7733: 
  7734:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
  7735:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
  7736:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7737: 
  7738:     InstrItinData <tc_44d5a428, /*tc_st*/
  7739:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  7740:       [Hex_FWD, Hex_FWD]>,
  7741: 
  7742:     InstrItinData <tc_44fffc58, /*tc_3*/
  7743:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  7744:       [Hex_FWD]>,
  7745: 
  7746:     InstrItinData <tc_45791fb8, /*tc_ld*/
  7747:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  7748:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7749: 
  7750:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  7751:       [InstrStage<1, [SLOT2]>], [2],
  7752:       [Hex_FWD]>,
  7753: 
  7754:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  7755:       [InstrStage<1, [SLOT3]>], [4, 1],
  7756:       [Hex_FWD, Hex_FWD]>,
  7757: 
  7758:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
  7759:       [InstrStage<1, [SLOT3]>], [4, 1],
  7760:       [Hex_FWD, Hex_FWD]>,
  7761: 
  7762:     InstrItinData <tc_4a55d03c, /*tc_1*/
  7763:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  7764:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7765: 
  7766:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
  7767:       [InstrStage<1, [SLOT3]>], [2, 2],
  7768:       [Hex_FWD, Hex_FWD]>,
  7769: 
  7770:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  7771:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  7772:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7773: 
  7774:     InstrItinData <tc_4bf903b0, /*tc_st*/
  7775:       [InstrStage<1, [SLOT0]>], [3],
  7776:       [Hex_FWD]>,
  7777: 
  7778:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  7779:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  7780:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7781: 
  7782:     InstrItinData <tc_512b1653, /*tc_st*/
  7783:       [InstrStage<1, [SLOT0]>], [1, 2],
  7784:       [Hex_FWD, Hex_FWD]>,
  7785: 
  7786:     InstrItinData <tc_53c851ab, /*tc_3stall*/
  7787:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
  7788:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7789: 
  7790:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  7791:       [InstrStage<1, [SLOT3]>], [1],
  7792:       [Hex_FWD]>,
  7793: 
  7794:     InstrItinData <tc_5502c366, /*tc_1*/
  7795:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  7796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7797: 
  7798:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  7799:       [InstrStage<1, [SLOT3]>], [],
  7800:       []>,
  7801: 
  7802:     InstrItinData <tc_556f6577, /*tc_3x*/
  7803:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  7804:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7805: 
  7806:     InstrItinData <tc_55a9a350, /*tc_st*/
  7807:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  7808:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7809: 
  7810:     InstrItinData <tc_55b33fda, /*tc_1*/
  7811:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  7812:       [Hex_FWD, Hex_FWD]>,
  7813: 
  7814:     InstrItinData <tc_56a124a7, /*tc_1*/
  7815:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  7816:       [Hex_FWD, Hex_FWD]>,
  7817: 
  7818:     InstrItinData <tc_57a55b54, /*tc_1*/
  7819:       [InstrStage<1, [SLOT3]>], [2, 2],
  7820:       [Hex_FWD, Hex_FWD]>,
  7821: 
  7822:     InstrItinData <tc_5944960d, /*tc_ld*/
  7823:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  7824:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7825: 
  7826:     InstrItinData <tc_59a7822c, /*tc_1*/
  7827:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  7828:       [Hex_FWD, Hex_FWD]>,
  7829: 
  7830:     InstrItinData <tc_5a222e89, /*tc_2early*/
  7831:       [InstrStage<1, [SLOT2]>], [1, 1],
  7832:       [Hex_FWD, Hex_FWD]>,
  7833: 
  7834:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  7835:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  7836:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7837: 
  7838:     InstrItinData <tc_5b347363, /*tc_1*/
  7839:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  7840:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7841: 
  7842:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  7843:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  7844:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7845: 
  7846:     InstrItinData <tc_5da50c4b, /*tc_1*/
  7847:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  7848:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7849: 
  7850:     InstrItinData <tc_5deb5e47, /*tc_st*/
  7851:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  7852:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7853: 
  7854:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  7855:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  7856:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7857: 
  7858:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
  7859:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
  7860:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7861: 
  7862:     InstrItinData <tc_60e324ff, /*tc_1*/
  7863:       [InstrStage<1, [SLOT2]>], [2],
  7864:       [Hex_FWD]>,
  7865: 
  7866:     InstrItinData <tc_63567288, /*tc_2latepred*/
  7867:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
  7868:       [Hex_FWD]>,
  7869: 
  7870:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  7871:       [InstrStage<1, [SLOT0]>], [4, 1],
  7872:       [Hex_FWD, Hex_FWD]>,
  7873: 
  7874:     InstrItinData <tc_651cbe02, /*tc_1*/
  7875:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  7876:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7877: 
  7878:     InstrItinData <tc_65279839, /*tc_2*/
  7879:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  7880:       [Hex_FWD, Hex_FWD]>,
  7881: 
  7882:     InstrItinData <tc_65cbd974, /*tc_st*/
  7883:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  7884:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7885: 
  7886:     InstrItinData <tc_69bfb303, /*tc_3*/
  7887:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  7888:       [Hex_FWD, Hex_FWD]>,
  7889: 
  7890:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  7891:       [InstrStage<1, [SLOT3]>], [4, 1],
  7892:       [Hex_FWD, Hex_FWD]>,
  7893: 
  7894:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  7895:       [InstrStage<1, [SLOT3]>], [4, 1],
  7896:       [Hex_FWD, Hex_FWD]>,
  7897: 
  7898:     InstrItinData <tc_6d861a95, /*tc_3x*/
  7899:       [InstrStage<1, [SLOT3]>], [2, 1],
  7900:       [Hex_FWD, Hex_FWD]>,
  7901: 
  7902:     InstrItinData <tc_6e20402a, /*tc_st*/
  7903:       [InstrStage<1, [SLOT0]>], [2, 3],
  7904:       [Hex_FWD, Hex_FWD]>,
  7905: 
  7906:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  7907:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  7908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7909: 
  7910:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  7911:       [InstrStage<1, [SLOT0]>], [1, 1],
  7912:       [Hex_FWD, Hex_FWD]>,
  7913: 
  7914:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  7915:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  7916:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7917: 
  7918:     InstrItinData <tc_711c805f, /*tc_1*/
  7919:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  7920:       [Hex_FWD, Hex_FWD]>,
  7921: 
  7922:     InstrItinData <tc_713b66bf, /*tc_1*/
  7923:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  7924:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7925: 
  7926:     InstrItinData <tc_7401744f, /*tc_2*/
  7927:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  7928:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7929: 
  7930:     InstrItinData <tc_7476d766, /*tc_3stall*/
  7931:       [InstrStage<1, [SLOT3]>], [4, 2],
  7932:       [Hex_FWD, Hex_FWD]>,
  7933: 
  7934:     InstrItinData <tc_74a42bda, /*tc_ld*/
  7935:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  7936:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7937: 
  7938:     InstrItinData <tc_759e57be, /*tc_3stall*/
  7939:       [InstrStage<1, [SLOT2]>], [4, 1],
  7940:       [Hex_FWD, Hex_FWD]>,
  7941: 
  7942:     InstrItinData <tc_76bb5435, /*tc_ld*/
  7943:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  7944:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7945: 
  7946:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  7947:       [InstrStage<1, [SLOT2]>], [1],
  7948:       [Hex_FWD]>,
  7949: 
  7950:     InstrItinData <tc_77f94a5e, /*tc_st*/
  7951:       [InstrStage<1, [SLOT0]>], [],
  7952:       []>,
  7953: 
  7954:     InstrItinData <tc_788b1d09, /*tc_3x*/
  7955:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  7956:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7957: 
  7958:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  7959:       [InstrStage<1, [SLOT0]>], [],
  7960:       []>,
  7961: 
  7962:     InstrItinData <tc_7af3a37e, /*tc_st*/
  7963:       [InstrStage<1, [SLOT0]>], [1, 3],
  7964:       [Hex_FWD, Hex_FWD]>,
  7965: 
  7966:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  7967:       [InstrStage<1, [SLOT0]>], [3, 2],
  7968:       [Hex_FWD, Hex_FWD]>,
  7969: 
  7970:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  7971:       [InstrStage<1, [SLOT0]>], [3],
  7972:       [Hex_FWD]>,
  7973: 
  7974:     InstrItinData <tc_7c31e19a, /*tc_st*/
  7975:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  7976:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7977: 
  7978:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  7979:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  7980:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7981: 
  7982:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  7983:       [InstrStage<1, [SLOT3]>], [4, 1],
  7984:       [Hex_FWD, Hex_FWD]>,
  7985: 
  7986:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  7987:       [InstrStage<1, [SLOT3]>], [],
  7988:       []>,
  7989: 
  7990:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  7991:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  7992:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  7993: 
  7994:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  7995:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  7996:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  7997: 
  7998:     InstrItinData <tc_8035e91f, /*tc_st*/
  7999:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
  8000:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV68, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV68; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV68 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV68 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 8001-8500 / 第 8001-8500 行

```tablegen
  8001: 
  8002:     InstrItinData <tc_822c3c68, /*tc_ld*/
  8003:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  8004:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8005: 
  8006:     InstrItinData <tc_829d8a86, /*tc_st*/
  8007:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  8008:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8009: 
  8010:     InstrItinData <tc_838c4d7a, /*tc_st*/
  8011:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  8012:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8013: 
  8014:     InstrItinData <tc_84a7500d, /*tc_2*/
  8015:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  8016:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8017: 
  8018:     InstrItinData <tc_86173609, /*tc_2latepred*/
  8019:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  8020:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8021: 
  8022:     InstrItinData <tc_887d1bb7, /*tc_st*/
  8023:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
  8024:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8025: 
  8026:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  8027:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  8028:       [Hex_FWD, Hex_FWD]>,
  8029: 
  8030:     InstrItinData <tc_8a825db2, /*tc_2*/
  8031:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8032:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8033: 
  8034:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
  8035:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  8036:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8037: 
  8038:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  8039:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
  8040:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8041: 
  8042:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  8043:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  8044:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8045: 
  8046:     InstrItinData <tc_9124c04f, /*tc_1*/
  8047:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  8048:       [Hex_FWD, Hex_FWD]>,
  8049: 
  8050:     InstrItinData <tc_92240447, /*tc_st*/
  8051:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  8052:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8053: 
  8054:     InstrItinData <tc_934753bb, /*tc_ld*/
  8055:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
  8056:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8057: 
  8058:     InstrItinData <tc_937dd41c, /*tc_ld*/
  8059:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  8060:       []>,
  8061: 
  8062:     InstrItinData <tc_9406230a, /*tc_3x*/
  8063:       [InstrStage<1, [SLOT3]>], [2, 1],
  8064:       [Hex_FWD, Hex_FWD]>,
  8065: 
  8066:     InstrItinData <tc_95a33176, /*tc_2*/
  8067:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  8068:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8069: 
  8070:     InstrItinData <tc_95f43c5e, /*tc_3*/
  8071:       [InstrStage<1, [SLOT2]>], [1],
  8072:       [Hex_FWD]>,
  8073: 
  8074:     InstrItinData <tc_96ef76ef, /*tc_st*/
  8075:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  8076:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8077: 
  8078:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  8079:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  8080:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8081: 
  8082:     InstrItinData <tc_9783714b, /*tc_4x*/
  8083:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  8084:       [Hex_FWD, Hex_FWD]>,
  8085: 
  8086:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  8087:       [InstrStage<1, [SLOT2]>], [4, 1],
  8088:       [Hex_FWD, Hex_FWD]>,
  8089: 
  8090:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
  8091:       [InstrStage<1, [SLOT2]>], [],
  8092:       []>,
  8093: 
  8094:     InstrItinData <tc_9b3c0462, /*tc_2*/
  8095:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8096:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8097: 
  8098:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  8099:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  8100:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8101: 
  8102:     InstrItinData <tc_9c52f549, /*tc_1*/
  8103:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  8104:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8105: 
  8106:     InstrItinData <tc_9e27f2f9, /*tc_1*/
  8107:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  8108:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8109: 
  8110:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  8111:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  8112:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8113: 
  8114:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  8115:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  8116:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8117: 
  8118:     InstrItinData <tc_9edefe01, /*tc_st*/
  8119:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
  8120:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8121: 
  8122:     InstrItinData <tc_9f6cd987, /*tc_1*/
  8123:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  8124:       [Hex_FWD, Hex_FWD]>,
  8125: 
  8126:     InstrItinData <tc_a08b630b, /*tc_2*/
  8127:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8129: 
  8130:     InstrItinData <tc_a1297125, /*tc_1*/
  8131:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  8132:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8133: 
  8134:     InstrItinData <tc_a154b476, /*tc_3x*/
  8135:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  8136:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8137: 
  8138:     InstrItinData <tc_a2b365d2, /*tc_st*/
  8139:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
  8140:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8141: 
  8142:     InstrItinData <tc_a3070909, /*tc_3stall*/
  8143:       [InstrStage<1, [SLOT0]>], [1, 1],
  8144:       [Hex_FWD, Hex_FWD]>,
  8145: 
  8146:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  8147:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  8148:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8149: 
  8150:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  8151:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  8152:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8153: 
  8154:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  8155:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  8156:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8157: 
  8158:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  8159:       [InstrStage<1, [SLOT0]>], [],
  8160:       []>,
  8161: 
  8162:     InstrItinData <tc_a724463d, /*tc_3stall*/
  8163:       [InstrStage<1, [SLOT0]>], [4, 1],
  8164:       [Hex_FWD, Hex_FWD]>,
  8165: 
  8166:     InstrItinData <tc_a7a13fac, /*tc_1*/
  8167:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  8168:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8169: 
  8170:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  8171:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  8172:       [Hex_FWD, Hex_FWD]>,
  8173: 
  8174:     InstrItinData <tc_a9edeffa, /*tc_st*/
  8175:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  8176:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8177: 
  8178:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  8179:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  8180:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8181: 
  8182:     InstrItinData <tc_ac65613f, /*tc_ld*/
  8183:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  8184:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8185: 
  8186:     InstrItinData <tc_addc37a8, /*tc_st*/
  8187:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  8188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8189: 
  8190:     InstrItinData <tc_ae5babd7, /*tc_st*/
  8191:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  8192:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8193: 
  8194:     InstrItinData <tc_aee6250c, /*tc_ld*/
  8195:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  8196:       [Hex_FWD, Hex_FWD]>,
  8197: 
  8198:     InstrItinData <tc_af6af259, /*tc_ld*/
  8199:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  8200:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8201: 
  8202:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  8203:       [InstrStage<1, [SLOT0]>], [1],
  8204:       [Hex_FWD]>,
  8205: 
  8206:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
  8207:       [InstrStage<1, [SLOT3]>], [1, 1],
  8208:       [Hex_FWD, Hex_FWD]>,
  8209: 
  8210:     InstrItinData <tc_b3d46584, /*tc_st*/
  8211:       [InstrStage<1, [SLOT0]>], [],
  8212:       []>,
  8213: 
  8214:     InstrItinData <tc_b4dc7630, /*tc_st*/
  8215:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
  8216:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8217: 
  8218:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  8219:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  8220:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8221: 
  8222:     InstrItinData <tc_b837298f, /*tc_1*/
  8223:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  8224:       []>,
  8225: 
  8226:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  8227:       [InstrStage<1, [SLOT2]>], [],
  8228:       []>,
  8229: 
  8230:     InstrItinData <tc_ba9255a6, /*tc_st*/
  8231:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
  8232:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8233: 
  8234:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  8235:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
  8236:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8237: 
  8238:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  8239:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  8240:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8241: 
  8242:     InstrItinData <tc_bb831a7c, /*tc_2*/
  8243:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  8244:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8245: 
  8246:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  8247:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  8248:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8249: 
  8250:     InstrItinData <tc_c20701f0, /*tc_2*/
  8251:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8252:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8253: 
  8254:     InstrItinData <tc_c21d7447, /*tc_3x*/
  8255:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  8256:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8257: 
  8258:     InstrItinData <tc_c57d9f39, /*tc_1*/
  8259:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  8260:       [Hex_FWD, Hex_FWD]>,
  8261: 
  8262:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  8263:       [InstrStage<1, [SLOT0]>], [],
  8264:       []>,
  8265: 
  8266:     InstrItinData <tc_ce59038e, /*tc_st*/
  8267:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  8268:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8269: 
  8270:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  8271:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  8272:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8273: 
  8274:     InstrItinData <tc_d03278fd, /*tc_st*/
  8275:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  8276:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8277: 
  8278:     InstrItinData <tc_d234b61a, /*tc_st*/
  8279:       [InstrStage<1, [SLOT0]>], [1],
  8280:       [Hex_FWD]>,
  8281: 
  8282:     InstrItinData <tc_d33e5eee, /*tc_1*/
  8283:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  8284:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8285: 
  8286:     InstrItinData <tc_d3632d88, /*tc_2*/
  8287:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  8288:       [Hex_FWD, Hex_FWD]>,
  8289: 
  8290:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  8291:       [InstrStage<1, [SLOT0]>], [1],
  8292:       [Hex_FWD]>,
  8293: 
  8294:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  8295:       [InstrStage<1, [SLOT2]>], [2],
  8296:       [Hex_FWD]>,
  8297: 
  8298:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  8299:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  8300:       [Hex_FWD, Hex_FWD]>,
  8301: 
  8302:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  8303:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  8304:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8305: 
  8306:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  8307:       [InstrStage<1, [SLOT3]>], [2, 1],
  8308:       [Hex_FWD, Hex_FWD]>,
  8309: 
  8310:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  8311:       [InstrStage<1, [SLOT3]>], [1],
  8312:       [Hex_FWD]>,
  8313: 
  8314:     InstrItinData <tc_db596beb, /*tc_3x*/
  8315:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  8316:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8317: 
  8318:     InstrItinData <tc_db96aa6b, /*tc_st*/
  8319:       [InstrStage<1, [SLOT0]>], [1],
  8320:       [Hex_FWD]>,
  8321: 
  8322:     InstrItinData <tc_dc51281d, /*tc_3*/
  8323:       [InstrStage<1, [SLOT2]>], [2, 1],
  8324:       [Hex_FWD, Hex_FWD]>,
  8325: 
  8326:     InstrItinData <tc_decdde8a, /*tc_1*/
  8327:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  8328:       [Hex_FWD]>,
  8329: 
  8330:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  8331:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
  8332:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8333: 
  8334:     InstrItinData <tc_e3d699e3, /*tc_2*/
  8335:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  8336:       [Hex_FWD, Hex_FWD]>,
  8337: 
  8338:     InstrItinData <tc_e60def48, /*tc_1*/
  8339:       [InstrStage<1, [SLOT2]>], [2],
  8340:       [Hex_FWD]>,
  8341: 
  8342:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  8343:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  8344:       [Hex_FWD, Hex_FWD]>,
  8345: 
  8346:     InstrItinData <tc_ed03645c, /*tc_1*/
  8347:       [InstrStage<1, [SLOT2]>], [3, 2],
  8348:       [Hex_FWD, Hex_FWD]>,
  8349: 
  8350:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  8351:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  8352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8353: 
  8354:     InstrItinData <tc_eed07714, /*tc_ld*/
  8355:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  8356:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8357: 
  8358:     InstrItinData <tc_eeda4109, /*tc_1*/
  8359:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  8360:       [Hex_FWD, Hex_FWD]>,
  8361: 
  8362:     InstrItinData <tc_ef921005, /*tc_1*/
  8363:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  8364:       [Hex_FWD, Hex_FWD]>,
  8365: 
  8366:     InstrItinData <tc_f098b237, /*tc_2*/
  8367:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8368:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8369: 
  8370:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  8371:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  8372:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8373: 
  8374:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  8375:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  8376:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8377: 
  8378:     InstrItinData <tc_f34c1c21, /*tc_2*/
  8379:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8380:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8381: 
  8382:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  8383:       [InstrStage<1, [SLOT0]>], [2],
  8384:       [Hex_FWD]>,
  8385: 
  8386:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
  8387:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
  8388:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8389: 
  8390:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  8391:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  8392:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8393: 
  8394:     InstrItinData <tc_f7569068, /*tc_4x*/
  8395:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  8396:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8397: 
  8398:     InstrItinData <tc_f97707c1, /*tc_1*/
  8399:       [InstrStage<1, [SLOT2]>], [2],
  8400:       [Hex_FWD]>,
  8401: 
  8402:     InstrItinData <tc_f999c66e, /*tc_1*/
  8403:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  8404:       [Hex_FWD, Hex_FWD]>,
  8405: 
  8406:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  8407:       [InstrStage<1, [SLOT3]>], [4, 2],
  8408:       [Hex_FWD, Hex_FWD]>,
  8409: 
  8410:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  8411:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  8412:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  8413:   ];
  8414: }
  8415: 
  8416: class DepScalarItinV69 {
  8417:   list<InstrItinData> DepScalarItinV69_list = [
  8418:     InstrItinData <tc_011e0e9d, /*tc_st*/
  8419:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  8420:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8421: 
  8422:     InstrItinData <tc_01d44cb2, /*tc_2*/
  8423:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8424:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8425: 
  8426:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  8427:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  8428:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8429: 
  8430:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  8431:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  8432:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8433: 
  8434:     InstrItinData <tc_0655b949, /*tc_st*/
  8435:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
  8436:       [Hex_FWD, Hex_FWD]>,
  8437: 
  8438:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  8439:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  8440:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8441: 
  8442:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  8443:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  8444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8445: 
  8446:     InstrItinData <tc_0a43be35, /*tc_3x*/
  8447:       [InstrStage<1, [SLOT3]>], [1],
  8448:       [Hex_FWD]>,
  8449: 
  8450:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  8451:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  8452:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8453: 
  8454:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
  8455:       [InstrStage<1, [SLOT2]>], [1],
  8456:       [Hex_FWD]>,
  8457: 
  8458:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  8459:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8461: 
  8462:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  8463:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  8464:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8465: 
  8466:     InstrItinData <tc_112d30d6, /*tc_1*/
  8467:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  8468:       [Hex_FWD]>,
  8469: 
  8470:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  8471:       [InstrStage<1, [SLOT0]>], [2],
  8472:       [Hex_FWD]>,
  8473: 
  8474:     InstrItinData <tc_1248597c, /*tc_3x*/
  8475:       [InstrStage<1, [SLOT3]>], [2, 2],
  8476:       [Hex_FWD, Hex_FWD]>,
  8477: 
  8478:     InstrItinData <tc_139ef484, /*tc_3stall*/
  8479:       [InstrStage<1, [SLOT2]>], [1, 1],
  8480:       [Hex_FWD, Hex_FWD]>,
  8481: 
  8482:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  8483:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
  8484:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8485: 
  8486:     InstrItinData <tc_151bf368, /*tc_1*/
  8487:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  8488:       [Hex_FWD, Hex_FWD]>,
  8489: 
  8490:     InstrItinData <tc_158aa3f7, /*tc_st*/
  8491:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  8492:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8493: 
  8494:     InstrItinData <tc_197dce51, /*tc_3x*/
  8495:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  8496:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8497: 
  8498:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  8499:       [InstrStage<1, [SLOT0]>], [3],
  8500:       [Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV69, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV69; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV69 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV69 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 8501-9000 / 第 8501-9000 行

```tablegen
  8501: 
  8502:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  8503:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  8504:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8505: 
  8506:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  8507:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  8508:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8509: 
  8510:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  8511:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  8512:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8513: 
  8514:     InstrItinData <tc_1fcb8495, /*tc_2*/
  8515:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8516:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8517: 
  8518:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  8519:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
  8520:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8521: 
  8522:     InstrItinData <tc_20131976, /*tc_2*/
  8523:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8524:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8525: 
  8526:     InstrItinData <tc_2237d952, /*tc_ld*/
  8527:       [InstrStage<1, [SLOT0]>], [1, 2],
  8528:       [Hex_FWD, Hex_FWD]>,
  8529: 
  8530:     InstrItinData <tc_23708a21, /*tc_1*/
  8531:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  8532:       []>,
  8533: 
  8534:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  8535:       [InstrStage<1, [SLOT0]>], [4, 1],
  8536:       [Hex_FWD, Hex_FWD]>,
  8537: 
  8538:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  8539:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  8540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8541: 
  8542:     InstrItinData <tc_24f426ab, /*tc_1*/
  8543:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  8544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8545: 
  8546:     InstrItinData <tc_27106296, /*tc_3x*/
  8547:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  8548:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8549: 
  8550:     InstrItinData <tc_280f7fe1, /*tc_st*/
  8551:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
  8552:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8553: 
  8554:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  8555:       [InstrStage<1, [SLOT3]>], [1, 1],
  8556:       [Hex_FWD, Hex_FWD]>,
  8557: 
  8558:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  8559:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  8560:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8561: 
  8562:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  8563:       [InstrStage<1, [SLOT3]>], [1],
  8564:       [Hex_FWD]>,
  8565: 
  8566:     InstrItinData <tc_2f573607, /*tc_1*/
  8567:       [InstrStage<1, [SLOT2]>], [2, 2],
  8568:       [Hex_FWD, Hex_FWD]>,
  8569: 
  8570:     InstrItinData <tc_33e7e673, /*tc_2early*/
  8571:       [InstrStage<1, [SLOT2]>], [],
  8572:       []>,
  8573: 
  8574:     InstrItinData <tc_362b0be2, /*tc_3*/
  8575:       [InstrStage<1, [SLOT2]>], [1],
  8576:       [Hex_FWD]>,
  8577: 
  8578:     InstrItinData <tc_38382228, /*tc_3x*/
  8579:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  8580:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8581: 
  8582:     InstrItinData <tc_388f9897, /*tc_1*/
  8583:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  8584:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8585: 
  8586:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  8587:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
  8588:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8589: 
  8590:     InstrItinData <tc_3d14a17b, /*tc_1*/
  8591:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  8592:       [Hex_FWD, Hex_FWD]>,
  8593: 
  8594:     InstrItinData <tc_3edca78f, /*tc_2*/
  8595:       [InstrStage<1, [SLOT3]>], [4, 2],
  8596:       [Hex_FWD, Hex_FWD]>,
  8597: 
  8598:     InstrItinData <tc_3fbf1042, /*tc_1*/
  8599:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  8600:       [Hex_FWD]>,
  8601: 
  8602:     InstrItinData <tc_407e96f9, /*tc_1*/
  8603:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  8604:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8605: 
  8606:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  8607:       [InstrStage<1, [SLOT0]>], [3, 1],
  8608:       [Hex_FWD, Hex_FWD]>,
  8609: 
  8610:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  8611:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  8612:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8613: 
  8614:     InstrItinData <tc_42ff66ba, /*tc_1*/
  8615:       [InstrStage<1, [SLOT2]>], [2, 2],
  8616:       [Hex_FWD, Hex_FWD]>,
  8617: 
  8618:     InstrItinData <tc_442395f3, /*tc_2latepred*/
  8619:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
  8620:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8621: 
  8622:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
  8623:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
  8624:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8625: 
  8626:     InstrItinData <tc_44d5a428, /*tc_st*/
  8627:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  8628:       [Hex_FWD, Hex_FWD]>,
  8629: 
  8630:     InstrItinData <tc_44fffc58, /*tc_3*/
  8631:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  8632:       [Hex_FWD]>,
  8633: 
  8634:     InstrItinData <tc_45791fb8, /*tc_ld*/
  8635:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  8636:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8637: 
  8638:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  8639:       [InstrStage<1, [SLOT2]>], [2],
  8640:       [Hex_FWD]>,
  8641: 
  8642:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  8643:       [InstrStage<1, [SLOT3]>], [4, 1],
  8644:       [Hex_FWD, Hex_FWD]>,
  8645: 
  8646:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
  8647:       [InstrStage<1, [SLOT3]>], [4, 1],
  8648:       [Hex_FWD, Hex_FWD]>,
  8649: 
  8650:     InstrItinData <tc_4a55d03c, /*tc_1*/
  8651:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  8652:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8653: 
  8654:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
  8655:       [InstrStage<1, [SLOT3]>], [2, 2],
  8656:       [Hex_FWD, Hex_FWD]>,
  8657: 
  8658:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  8659:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  8660:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8661: 
  8662:     InstrItinData <tc_4bf903b0, /*tc_st*/
  8663:       [InstrStage<1, [SLOT0]>], [3],
  8664:       [Hex_FWD]>,
  8665: 
  8666:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  8667:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  8668:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8669: 
  8670:     InstrItinData <tc_512b1653, /*tc_st*/
  8671:       [InstrStage<1, [SLOT0]>], [1, 2],
  8672:       [Hex_FWD, Hex_FWD]>,
  8673: 
  8674:     InstrItinData <tc_53c851ab, /*tc_3stall*/
  8675:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
  8676:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8677: 
  8678:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  8679:       [InstrStage<1, [SLOT3]>], [1],
  8680:       [Hex_FWD]>,
  8681: 
  8682:     InstrItinData <tc_5502c366, /*tc_1*/
  8683:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  8684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8685: 
  8686:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  8687:       [InstrStage<1, [SLOT3]>], [],
  8688:       []>,
  8689: 
  8690:     InstrItinData <tc_556f6577, /*tc_3x*/
  8691:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  8692:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8693: 
  8694:     InstrItinData <tc_55a9a350, /*tc_st*/
  8695:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  8696:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8697: 
  8698:     InstrItinData <tc_55b33fda, /*tc_1*/
  8699:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  8700:       [Hex_FWD, Hex_FWD]>,
  8701: 
  8702:     InstrItinData <tc_56a124a7, /*tc_1*/
  8703:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  8704:       [Hex_FWD, Hex_FWD]>,
  8705: 
  8706:     InstrItinData <tc_57a55b54, /*tc_1*/
  8707:       [InstrStage<1, [SLOT3]>], [2, 2],
  8708:       [Hex_FWD, Hex_FWD]>,
  8709: 
  8710:     InstrItinData <tc_5944960d, /*tc_ld*/
  8711:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  8712:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8713: 
  8714:     InstrItinData <tc_59a7822c, /*tc_1*/
  8715:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  8716:       [Hex_FWD, Hex_FWD]>,
  8717: 
  8718:     InstrItinData <tc_5a222e89, /*tc_2early*/
  8719:       [InstrStage<1, [SLOT2]>], [1, 1],
  8720:       [Hex_FWD, Hex_FWD]>,
  8721: 
  8722:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  8723:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  8724:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8725: 
  8726:     InstrItinData <tc_5b347363, /*tc_1*/
  8727:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  8728:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8729: 
  8730:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  8731:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  8732:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8733: 
  8734:     InstrItinData <tc_5da50c4b, /*tc_1*/
  8735:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  8736:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8737: 
  8738:     InstrItinData <tc_5deb5e47, /*tc_st*/
  8739:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  8740:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8741: 
  8742:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  8743:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  8744:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8745: 
  8746:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
  8747:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
  8748:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8749: 
  8750:     InstrItinData <tc_60e324ff, /*tc_1*/
  8751:       [InstrStage<1, [SLOT2]>], [2],
  8752:       [Hex_FWD]>,
  8753: 
  8754:     InstrItinData <tc_63567288, /*tc_2latepred*/
  8755:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
  8756:       [Hex_FWD]>,
  8757: 
  8758:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  8759:       [InstrStage<1, [SLOT0]>], [4, 1],
  8760:       [Hex_FWD, Hex_FWD]>,
  8761: 
  8762:     InstrItinData <tc_651cbe02, /*tc_1*/
  8763:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  8764:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8765: 
  8766:     InstrItinData <tc_65279839, /*tc_2*/
  8767:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  8768:       [Hex_FWD, Hex_FWD]>,
  8769: 
  8770:     InstrItinData <tc_65cbd974, /*tc_st*/
  8771:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  8772:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8773: 
  8774:     InstrItinData <tc_69bfb303, /*tc_3*/
  8775:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  8776:       [Hex_FWD, Hex_FWD]>,
  8777: 
  8778:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  8779:       [InstrStage<1, [SLOT3]>], [4, 1],
  8780:       [Hex_FWD, Hex_FWD]>,
  8781: 
  8782:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  8783:       [InstrStage<1, [SLOT3]>], [4, 1],
  8784:       [Hex_FWD, Hex_FWD]>,
  8785: 
  8786:     InstrItinData <tc_6d861a95, /*tc_3x*/
  8787:       [InstrStage<1, [SLOT3]>], [2, 1],
  8788:       [Hex_FWD, Hex_FWD]>,
  8789: 
  8790:     InstrItinData <tc_6e20402a, /*tc_st*/
  8791:       [InstrStage<1, [SLOT0]>], [2, 3],
  8792:       [Hex_FWD, Hex_FWD]>,
  8793: 
  8794:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  8795:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  8796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8797: 
  8798:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  8799:       [InstrStage<1, [SLOT0]>], [1, 1],
  8800:       [Hex_FWD, Hex_FWD]>,
  8801: 
  8802:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  8803:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  8804:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8805: 
  8806:     InstrItinData <tc_711c805f, /*tc_1*/
  8807:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  8808:       [Hex_FWD, Hex_FWD]>,
  8809: 
  8810:     InstrItinData <tc_713b66bf, /*tc_1*/
  8811:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  8812:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8813: 
  8814:     InstrItinData <tc_7401744f, /*tc_2*/
  8815:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  8816:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8817: 
  8818:     InstrItinData <tc_7476d766, /*tc_3stall*/
  8819:       [InstrStage<1, [SLOT3]>], [4, 2],
  8820:       [Hex_FWD, Hex_FWD]>,
  8821: 
  8822:     InstrItinData <tc_74a42bda, /*tc_ld*/
  8823:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  8824:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8825: 
  8826:     InstrItinData <tc_759e57be, /*tc_3stall*/
  8827:       [InstrStage<1, [SLOT2]>], [4, 1],
  8828:       [Hex_FWD, Hex_FWD]>,
  8829: 
  8830:     InstrItinData <tc_76bb5435, /*tc_ld*/
  8831:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  8832:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8833: 
  8834:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  8835:       [InstrStage<1, [SLOT2]>], [1],
  8836:       [Hex_FWD]>,
  8837: 
  8838:     InstrItinData <tc_77f94a5e, /*tc_st*/
  8839:       [InstrStage<1, [SLOT0]>], [],
  8840:       []>,
  8841: 
  8842:     InstrItinData <tc_788b1d09, /*tc_3x*/
  8843:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  8844:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8845: 
  8846:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  8847:       [InstrStage<1, [SLOT0]>], [],
  8848:       []>,
  8849: 
  8850:     InstrItinData <tc_7af3a37e, /*tc_st*/
  8851:       [InstrStage<1, [SLOT0]>], [1, 3],
  8852:       [Hex_FWD, Hex_FWD]>,
  8853: 
  8854:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  8855:       [InstrStage<1, [SLOT0]>], [3, 2],
  8856:       [Hex_FWD, Hex_FWD]>,
  8857: 
  8858:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  8859:       [InstrStage<1, [SLOT0]>], [3],
  8860:       [Hex_FWD]>,
  8861: 
  8862:     InstrItinData <tc_7c31e19a, /*tc_st*/
  8863:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  8864:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8865: 
  8866:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  8867:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  8868:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8869: 
  8870:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  8871:       [InstrStage<1, [SLOT3]>], [4, 1],
  8872:       [Hex_FWD, Hex_FWD]>,
  8873: 
  8874:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  8875:       [InstrStage<1, [SLOT3]>], [],
  8876:       []>,
  8877: 
  8878:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  8879:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  8880:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8881: 
  8882:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  8883:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  8884:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8885: 
  8886:     InstrItinData <tc_8035e91f, /*tc_st*/
  8887:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
  8888:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8889: 
  8890:     InstrItinData <tc_822c3c68, /*tc_ld*/
  8891:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  8892:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8893: 
  8894:     InstrItinData <tc_829d8a86, /*tc_st*/
  8895:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  8896:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8897: 
  8898:     InstrItinData <tc_838c4d7a, /*tc_st*/
  8899:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  8900:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8901: 
  8902:     InstrItinData <tc_84a7500d, /*tc_2*/
  8903:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  8904:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8905: 
  8906:     InstrItinData <tc_86173609, /*tc_2latepred*/
  8907:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  8908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8909: 
  8910:     InstrItinData <tc_887d1bb7, /*tc_st*/
  8911:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
  8912:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8913: 
  8914:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  8915:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  8916:       [Hex_FWD, Hex_FWD]>,
  8917: 
  8918:     InstrItinData <tc_8a825db2, /*tc_2*/
  8919:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8920:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8921: 
  8922:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
  8923:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  8924:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8925: 
  8926:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  8927:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
  8928:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8929: 
  8930:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  8931:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  8932:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8933: 
  8934:     InstrItinData <tc_9124c04f, /*tc_1*/
  8935:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  8936:       [Hex_FWD, Hex_FWD]>,
  8937: 
  8938:     InstrItinData <tc_92240447, /*tc_st*/
  8939:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  8940:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8941: 
  8942:     InstrItinData <tc_934753bb, /*tc_ld*/
  8943:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
  8944:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8945: 
  8946:     InstrItinData <tc_937dd41c, /*tc_ld*/
  8947:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  8948:       []>,
  8949: 
  8950:     InstrItinData <tc_9406230a, /*tc_3x*/
  8951:       [InstrStage<1, [SLOT3]>], [2, 1],
  8952:       [Hex_FWD, Hex_FWD]>,
  8953: 
  8954:     InstrItinData <tc_95a33176, /*tc_2*/
  8955:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  8956:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8957: 
  8958:     InstrItinData <tc_95f43c5e, /*tc_3*/
  8959:       [InstrStage<1, [SLOT2]>], [1],
  8960:       [Hex_FWD]>,
  8961: 
  8962:     InstrItinData <tc_96ef76ef, /*tc_st*/
  8963:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  8964:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  8965: 
  8966:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  8967:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  8968:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8969: 
  8970:     InstrItinData <tc_9783714b, /*tc_4x*/
  8971:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  8972:       [Hex_FWD, Hex_FWD]>,
  8973: 
  8974:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  8975:       [InstrStage<1, [SLOT2]>], [4, 1],
  8976:       [Hex_FWD, Hex_FWD]>,
  8977: 
  8978:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
  8979:       [InstrStage<1, [SLOT2]>], [],
  8980:       []>,
  8981: 
  8982:     InstrItinData <tc_9b3c0462, /*tc_2*/
  8983:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  8984:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8985: 
  8986:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  8987:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  8988:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8989: 
  8990:     InstrItinData <tc_9c52f549, /*tc_1*/
  8991:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  8992:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8993: 
  8994:     InstrItinData <tc_9e27f2f9, /*tc_1*/
  8995:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  8996:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  8997: 
  8998:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  8999:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  9000:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 9001-9500 / 第 9001-9500 行

```tablegen
  9001: 
  9002:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  9003:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  9004:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9005: 
  9006:     InstrItinData <tc_9edefe01, /*tc_st*/
  9007:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
  9008:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9009: 
  9010:     InstrItinData <tc_9f6cd987, /*tc_1*/
  9011:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  9012:       [Hex_FWD, Hex_FWD]>,
  9013: 
  9014:     InstrItinData <tc_a08b630b, /*tc_2*/
  9015:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9016:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9017: 
  9018:     InstrItinData <tc_a1297125, /*tc_1*/
  9019:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  9020:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9021: 
  9022:     InstrItinData <tc_a154b476, /*tc_3x*/
  9023:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  9024:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9025: 
  9026:     InstrItinData <tc_a2b365d2, /*tc_st*/
  9027:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
  9028:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9029: 
  9030:     InstrItinData <tc_a3070909, /*tc_3stall*/
  9031:       [InstrStage<1, [SLOT0]>], [1, 1],
  9032:       [Hex_FWD, Hex_FWD]>,
  9033: 
  9034:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  9035:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  9036:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9037: 
  9038:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  9039:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  9040:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9041: 
  9042:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  9043:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  9044:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9045: 
  9046:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  9047:       [InstrStage<1, [SLOT0]>], [],
  9048:       []>,
  9049: 
  9050:     InstrItinData <tc_a724463d, /*tc_3stall*/
  9051:       [InstrStage<1, [SLOT0]>], [4, 1],
  9052:       [Hex_FWD, Hex_FWD]>,
  9053: 
  9054:     InstrItinData <tc_a7a13fac, /*tc_1*/
  9055:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  9056:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9057: 
  9058:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  9059:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  9060:       [Hex_FWD, Hex_FWD]>,
  9061: 
  9062:     InstrItinData <tc_a9edeffa, /*tc_st*/
  9063:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  9064:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9065: 
  9066:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  9067:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  9068:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9069: 
  9070:     InstrItinData <tc_ac65613f, /*tc_ld*/
  9071:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  9072:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9073: 
  9074:     InstrItinData <tc_addc37a8, /*tc_st*/
  9075:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  9076:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9077: 
  9078:     InstrItinData <tc_ae5babd7, /*tc_st*/
  9079:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  9080:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9081: 
  9082:     InstrItinData <tc_aee6250c, /*tc_ld*/
  9083:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  9084:       [Hex_FWD, Hex_FWD]>,
  9085: 
  9086:     InstrItinData <tc_af6af259, /*tc_ld*/
  9087:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  9088:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9089: 
  9090:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  9091:       [InstrStage<1, [SLOT0]>], [1],
  9092:       [Hex_FWD]>,
  9093: 
  9094:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
  9095:       [InstrStage<1, [SLOT3]>], [1, 1],
  9096:       [Hex_FWD, Hex_FWD]>,
  9097: 
  9098:     InstrItinData <tc_b3d46584, /*tc_st*/
  9099:       [InstrStage<1, [SLOT0]>], [],
  9100:       []>,
  9101: 
  9102:     InstrItinData <tc_b4dc7630, /*tc_st*/
  9103:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
  9104:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9105: 
  9106:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  9107:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  9108:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9109: 
  9110:     InstrItinData <tc_b837298f, /*tc_1*/
  9111:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  9112:       []>,
  9113: 
  9114:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
  9115:       [InstrStage<1, [SLOT2]>], [],
  9116:       []>,
  9117: 
  9118:     InstrItinData <tc_ba9255a6, /*tc_st*/
  9119:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
  9120:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9121: 
  9122:     InstrItinData <tc_bb07f2c5, /*tc_st*/
  9123:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
  9124:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9125: 
  9126:     InstrItinData <tc_bb78483e, /*tc_3stall*/
  9127:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  9128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9129: 
  9130:     InstrItinData <tc_bb831a7c, /*tc_2*/
  9131:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
  9132:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9133: 
  9134:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
  9135:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
  9136:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9137: 
  9138:     InstrItinData <tc_c20701f0, /*tc_2*/
  9139:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9140:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9141: 
  9142:     InstrItinData <tc_c21d7447, /*tc_3x*/
  9143:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  9144:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9145: 
  9146:     InstrItinData <tc_c57d9f39, /*tc_1*/
  9147:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  9148:       [Hex_FWD, Hex_FWD]>,
  9149: 
  9150:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
  9151:       [InstrStage<1, [SLOT0]>], [],
  9152:       []>,
  9153: 
  9154:     InstrItinData <tc_ce59038e, /*tc_st*/
  9155:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
  9156:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9157: 
  9158:     InstrItinData <tc_cfa0e29b, /*tc_st*/
  9159:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
  9160:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9161: 
  9162:     InstrItinData <tc_d03278fd, /*tc_st*/
  9163:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
  9164:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9165: 
  9166:     InstrItinData <tc_d234b61a, /*tc_st*/
  9167:       [InstrStage<1, [SLOT0]>], [1],
  9168:       [Hex_FWD]>,
  9169: 
  9170:     InstrItinData <tc_d33e5eee, /*tc_1*/
  9171:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  9172:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9173: 
  9174:     InstrItinData <tc_d3632d88, /*tc_2*/
  9175:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  9176:       [Hex_FWD, Hex_FWD]>,
  9177: 
  9178:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
  9179:       [InstrStage<1, [SLOT0]>], [1],
  9180:       [Hex_FWD]>,
  9181: 
  9182:     InstrItinData <tc_d57d649c, /*tc_3stall*/
  9183:       [InstrStage<1, [SLOT2]>], [2],
  9184:       [Hex_FWD]>,
  9185: 
  9186:     InstrItinData <tc_d61dfdc3, /*tc_2*/
  9187:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  9188:       [Hex_FWD, Hex_FWD]>,
  9189: 
  9190:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
  9191:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  9192:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9193: 
  9194:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
  9195:       [InstrStage<1, [SLOT3]>], [2, 1],
  9196:       [Hex_FWD, Hex_FWD]>,
  9197: 
  9198:     InstrItinData <tc_d7718fbe, /*tc_3x*/
  9199:       [InstrStage<1, [SLOT3]>], [1],
  9200:       [Hex_FWD]>,
  9201: 
  9202:     InstrItinData <tc_db596beb, /*tc_3x*/
  9203:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  9204:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9205: 
  9206:     InstrItinData <tc_db96aa6b, /*tc_st*/
  9207:       [InstrStage<1, [SLOT0]>], [1],
  9208:       [Hex_FWD]>,
  9209: 
  9210:     InstrItinData <tc_dc51281d, /*tc_3*/
  9211:       [InstrStage<1, [SLOT2]>], [2, 1],
  9212:       [Hex_FWD, Hex_FWD]>,
  9213: 
  9214:     InstrItinData <tc_decdde8a, /*tc_1*/
  9215:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  9216:       [Hex_FWD]>,
  9217: 
  9218:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
  9219:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
  9220:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9221: 
  9222:     InstrItinData <tc_e3d699e3, /*tc_2*/
  9223:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  9224:       [Hex_FWD, Hex_FWD]>,
  9225: 
  9226:     InstrItinData <tc_e60def48, /*tc_1*/
  9227:       [InstrStage<1, [SLOT2]>], [2],
  9228:       [Hex_FWD]>,
  9229: 
  9230:     InstrItinData <tc_e9170fb7, /*tc_ld*/
  9231:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  9232:       [Hex_FWD, Hex_FWD]>,
  9233: 
  9234:     InstrItinData <tc_ed03645c, /*tc_1*/
  9235:       [InstrStage<1, [SLOT2]>], [3, 2],
  9236:       [Hex_FWD, Hex_FWD]>,
  9237: 
  9238:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
  9239:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  9240:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9241: 
  9242:     InstrItinData <tc_eed07714, /*tc_ld*/
  9243:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  9244:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9245: 
  9246:     InstrItinData <tc_eeda4109, /*tc_1*/
  9247:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  9248:       [Hex_FWD, Hex_FWD]>,
  9249: 
  9250:     InstrItinData <tc_ef921005, /*tc_1*/
  9251:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  9252:       [Hex_FWD, Hex_FWD]>,
  9253: 
  9254:     InstrItinData <tc_f098b237, /*tc_2*/
  9255:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9256:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9257: 
  9258:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
  9259:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  9260:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9261: 
  9262:     InstrItinData <tc_f0e8e832, /*tc_4x*/
  9263:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  9264:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9265: 
  9266:     InstrItinData <tc_f34c1c21, /*tc_2*/
  9267:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9268:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9269: 
  9270:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
  9271:       [InstrStage<1, [SLOT0]>], [2],
  9272:       [Hex_FWD]>,
  9273: 
  9274:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
  9275:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
  9276:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9277: 
  9278:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
  9279:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
  9280:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9281: 
  9282:     InstrItinData <tc_f7569068, /*tc_4x*/
  9283:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
  9284:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9285: 
  9286:     InstrItinData <tc_f97707c1, /*tc_1*/
  9287:       [InstrStage<1, [SLOT2]>], [2],
  9288:       [Hex_FWD]>,
  9289: 
  9290:     InstrItinData <tc_f999c66e, /*tc_1*/
  9291:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  9292:       [Hex_FWD, Hex_FWD]>,
  9293: 
  9294:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
  9295:       [InstrStage<1, [SLOT3]>], [4, 2],
  9296:       [Hex_FWD, Hex_FWD]>,
  9297: 
  9298:     InstrItinData <tc_fedb7e19, /*tc_ld*/
  9299:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
  9300:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
  9301:   ];
  9302: }
  9303: 
  9304: class DepScalarItinV71 {
  9305:   list<InstrItinData> DepScalarItinV71_list = [
  9306:     InstrItinData <tc_011e0e9d, /*tc_st*/
  9307:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
  9308:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9309: 
  9310:     InstrItinData <tc_01d44cb2, /*tc_2*/
  9311:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9312:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9313: 
  9314:     InstrItinData <tc_01e1be3b, /*tc_3x*/
  9315:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  9316:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9317: 
  9318:     InstrItinData <tc_02fe1c65, /*tc_4x*/
  9319:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
  9320:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9321: 
  9322:     InstrItinData <tc_0655b949, /*tc_st*/
  9323:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
  9324:       [Hex_FWD, Hex_FWD]>,
  9325: 
  9326:     InstrItinData <tc_075c8dd8, /*tc_ld*/
  9327:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
  9328:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9329: 
  9330:     InstrItinData <tc_0a195f2c, /*tc_4x*/
  9331:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  9332:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9333: 
  9334:     InstrItinData <tc_0a43be35, /*tc_3x*/
  9335:       [InstrStage<1, [SLOT3]>], [1],
  9336:       [Hex_FWD]>,
  9337: 
  9338:     InstrItinData <tc_0a6c20ae, /*tc_st*/
  9339:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
  9340:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9341: 
  9342:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
  9343:       [InstrStage<1, [SLOT2]>], [1],
  9344:       [Hex_FWD]>,
  9345: 
  9346:     InstrItinData <tc_0dfac0a7, /*tc_2*/
  9347:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9349: 
  9350:     InstrItinData <tc_0fac1eb8, /*tc_st*/
  9351:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
  9352:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9353: 
  9354:     InstrItinData <tc_112d30d6, /*tc_1*/
  9355:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
  9356:       [Hex_FWD]>,
  9357: 
  9358:     InstrItinData <tc_1242dc2a, /*tc_ld*/
  9359:       [InstrStage<1, [SLOT0]>], [2],
  9360:       [Hex_FWD]>,
  9361: 
  9362:     InstrItinData <tc_1248597c, /*tc_3x*/
  9363:       [InstrStage<1, [SLOT3]>], [2, 2],
  9364:       [Hex_FWD, Hex_FWD]>,
  9365: 
  9366:     InstrItinData <tc_139ef484, /*tc_3stall*/
  9367:       [InstrStage<1, [SLOT2]>], [1, 1],
  9368:       [Hex_FWD, Hex_FWD]>,
  9369: 
  9370:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
  9371:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
  9372:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9373: 
  9374:     InstrItinData <tc_151bf368, /*tc_1*/
  9375:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  9376:       [Hex_FWD, Hex_FWD]>,
  9377: 
  9378:     InstrItinData <tc_158aa3f7, /*tc_st*/
  9379:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
  9380:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9381: 
  9382:     InstrItinData <tc_197dce51, /*tc_3x*/
  9383:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
  9384:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9385: 
  9386:     InstrItinData <tc_1981450d, /*tc_newvjump*/
  9387:       [InstrStage<1, [SLOT0]>], [3],
  9388:       [Hex_FWD]>,
  9389: 
  9390:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
  9391:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
  9392:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9393: 
  9394:     InstrItinData <tc_1c7522a8, /*tc_ld*/
  9395:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
  9396:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9397: 
  9398:     InstrItinData <tc_1d41f8b7, /*tc_1*/
  9399:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
  9400:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9401: 
  9402:     InstrItinData <tc_1fcb8495, /*tc_2*/
  9403:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9404:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9405: 
  9406:     InstrItinData <tc_1fe4ab69, /*tc_st*/
  9407:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
  9408:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9409: 
  9410:     InstrItinData <tc_20131976, /*tc_2*/
  9411:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9412:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9413: 
  9414:     InstrItinData <tc_2237d952, /*tc_ld*/
  9415:       [InstrStage<1, [SLOT0]>], [1, 2],
  9416:       [Hex_FWD, Hex_FWD]>,
  9417: 
  9418:     InstrItinData <tc_23708a21, /*tc_1*/
  9419:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
  9420:       []>,
  9421: 
  9422:     InstrItinData <tc_2471c1c8, /*tc_ld*/
  9423:       [InstrStage<1, [SLOT0]>], [4, 1],
  9424:       [Hex_FWD, Hex_FWD]>,
  9425: 
  9426:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
  9427:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  9428:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9429: 
  9430:     InstrItinData <tc_24f426ab, /*tc_1*/
  9431:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  9432:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9433: 
  9434:     InstrItinData <tc_27106296, /*tc_3x*/
  9435:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
  9436:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9437: 
  9438:     InstrItinData <tc_280f7fe1, /*tc_st*/
  9439:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
  9440:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9441: 
  9442:     InstrItinData <tc_28e55c6f, /*tc_3x*/
  9443:       [InstrStage<1, [SLOT3]>], [1, 1],
  9444:       [Hex_FWD, Hex_FWD]>,
  9445: 
  9446:     InstrItinData <tc_2c13e7f5, /*tc_2*/
  9447:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  9448:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9449: 
  9450:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
  9451:       [InstrStage<1, [SLOT3]>], [1],
  9452:       [Hex_FWD]>,
  9453: 
  9454:     InstrItinData <tc_2f573607, /*tc_1*/
  9455:       [InstrStage<1, [SLOT2]>], [2, 2],
  9456:       [Hex_FWD, Hex_FWD]>,
  9457: 
  9458:     InstrItinData <tc_33e7e673, /*tc_2early*/
  9459:       [InstrStage<1, [SLOT2]>], [],
  9460:       []>,
  9461: 
  9462:     InstrItinData <tc_362b0be2, /*tc_3*/
  9463:       [InstrStage<1, [SLOT2]>], [1],
  9464:       [Hex_FWD]>,
  9465: 
  9466:     InstrItinData <tc_38382228, /*tc_3x*/
  9467:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
  9468:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9469: 
  9470:     InstrItinData <tc_388f9897, /*tc_1*/
  9471:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  9472:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9473: 
  9474:     InstrItinData <tc_38e0bae9, /*tc_3x*/
  9475:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
  9476:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9477: 
  9478:     InstrItinData <tc_3d14a17b, /*tc_1*/
  9479:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
  9480:       [Hex_FWD, Hex_FWD]>,
  9481: 
  9482:     InstrItinData <tc_3edca78f, /*tc_2*/
  9483:       [InstrStage<1, [SLOT3]>], [4, 2],
  9484:       [Hex_FWD, Hex_FWD]>,
  9485: 
  9486:     InstrItinData <tc_3fbf1042, /*tc_1*/
  9487:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
  9488:       [Hex_FWD]>,
  9489: 
  9490:     InstrItinData <tc_407e96f9, /*tc_1*/
  9491:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  9492:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9493: 
  9494:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
  9495:       [InstrStage<1, [SLOT0]>], [3, 1],
  9496:       [Hex_FWD, Hex_FWD]>,
  9497: 
  9498:     InstrItinData <tc_4222e6bf, /*tc_ld*/
  9499:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
  9500:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV71, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV71; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV71 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV71 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 9501-10000 / 第 9501-10000 行

```tablegen
  9501: 
  9502:     InstrItinData <tc_42ff66ba, /*tc_1*/
  9503:       [InstrStage<1, [SLOT2]>], [2, 2],
  9504:       [Hex_FWD, Hex_FWD]>,
  9505: 
  9506:     InstrItinData <tc_442395f3, /*tc_2latepred*/
  9507:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
  9508:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9509: 
  9510:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
  9511:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
  9512:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9513: 
  9514:     InstrItinData <tc_44d5a428, /*tc_st*/
  9515:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
  9516:       [Hex_FWD, Hex_FWD]>,
  9517: 
  9518:     InstrItinData <tc_44fffc58, /*tc_3*/
  9519:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
  9520:       [Hex_FWD]>,
  9521: 
  9522:     InstrItinData <tc_45791fb8, /*tc_ld*/
  9523:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
  9524:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9525: 
  9526:     InstrItinData <tc_45f9d1be, /*tc_2early*/
  9527:       [InstrStage<1, [SLOT2]>], [2],
  9528:       [Hex_FWD]>,
  9529: 
  9530:     InstrItinData <tc_46c18ecf, /*tc_3x*/
  9531:       [InstrStage<1, [SLOT3]>], [4, 1],
  9532:       [Hex_FWD, Hex_FWD]>,
  9533: 
  9534:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
  9535:       [InstrStage<1, [SLOT3]>], [4, 1],
  9536:       [Hex_FWD, Hex_FWD]>,
  9537: 
  9538:     InstrItinData <tc_4a55d03c, /*tc_1*/
  9539:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  9540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9541: 
  9542:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
  9543:       [InstrStage<1, [SLOT3]>], [2, 2],
  9544:       [Hex_FWD, Hex_FWD]>,
  9545: 
  9546:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
  9547:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  9548:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9549: 
  9550:     InstrItinData <tc_4bf903b0, /*tc_st*/
  9551:       [InstrStage<1, [SLOT0]>], [3],
  9552:       [Hex_FWD]>,
  9553: 
  9554:     InstrItinData <tc_503ce0f3, /*tc_3x*/
  9555:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
  9556:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9557: 
  9558:     InstrItinData <tc_512b1653, /*tc_st*/
  9559:       [InstrStage<1, [SLOT0]>], [1, 2],
  9560:       [Hex_FWD, Hex_FWD]>,
  9561: 
  9562:     InstrItinData <tc_53c851ab, /*tc_3stall*/
  9563:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
  9564:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9565: 
  9566:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
  9567:       [InstrStage<1, [SLOT3]>], [1],
  9568:       [Hex_FWD]>,
  9569: 
  9570:     InstrItinData <tc_5502c366, /*tc_1*/
  9571:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  9572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9573: 
  9574:     InstrItinData <tc_55255f2b, /*tc_3stall*/
  9575:       [InstrStage<1, [SLOT3]>], [],
  9576:       []>,
  9577: 
  9578:     InstrItinData <tc_556f6577, /*tc_3x*/
  9579:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
  9580:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9581: 
  9582:     InstrItinData <tc_55a9a350, /*tc_st*/
  9583:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
  9584:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9585: 
  9586:     InstrItinData <tc_55b33fda, /*tc_1*/
  9587:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  9588:       [Hex_FWD, Hex_FWD]>,
  9589: 
  9590:     InstrItinData <tc_56a124a7, /*tc_1*/
  9591:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  9592:       [Hex_FWD, Hex_FWD]>,
  9593: 
  9594:     InstrItinData <tc_57a55b54, /*tc_1*/
  9595:       [InstrStage<1, [SLOT3]>], [2, 2],
  9596:       [Hex_FWD, Hex_FWD]>,
  9597: 
  9598:     InstrItinData <tc_5944960d, /*tc_ld*/
  9599:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
  9600:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9601: 
  9602:     InstrItinData <tc_59a7822c, /*tc_1*/
  9603:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
  9604:       [Hex_FWD, Hex_FWD]>,
  9605: 
  9606:     InstrItinData <tc_5a222e89, /*tc_2early*/
  9607:       [InstrStage<1, [SLOT2]>], [1, 1],
  9608:       [Hex_FWD, Hex_FWD]>,
  9609: 
  9610:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
  9611:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
  9612:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9613: 
  9614:     InstrItinData <tc_5b347363, /*tc_1*/
  9615:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
  9616:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9617: 
  9618:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
  9619:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
  9620:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9621: 
  9622:     InstrItinData <tc_5da50c4b, /*tc_1*/
  9623:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  9624:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9625: 
  9626:     InstrItinData <tc_5deb5e47, /*tc_st*/
  9627:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  9628:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9629: 
  9630:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
  9631:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  9632:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9633: 
  9634:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
  9635:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
  9636:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9637: 
  9638:     InstrItinData <tc_60e324ff, /*tc_1*/
  9639:       [InstrStage<1, [SLOT2]>], [2],
  9640:       [Hex_FWD]>,
  9641: 
  9642:     InstrItinData <tc_63567288, /*tc_2latepred*/
  9643:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
  9644:       [Hex_FWD]>,
  9645: 
  9646:     InstrItinData <tc_64b00d8a, /*tc_ld*/
  9647:       [InstrStage<1, [SLOT0]>], [4, 1],
  9648:       [Hex_FWD, Hex_FWD]>,
  9649: 
  9650:     InstrItinData <tc_651cbe02, /*tc_1*/
  9651:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  9652:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9653: 
  9654:     InstrItinData <tc_65279839, /*tc_2*/
  9655:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  9656:       [Hex_FWD, Hex_FWD]>,
  9657: 
  9658:     InstrItinData <tc_65cbd974, /*tc_st*/
  9659:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
  9660:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9661: 
  9662:     InstrItinData <tc_69bfb303, /*tc_3*/
  9663:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
  9664:       [Hex_FWD, Hex_FWD]>,
  9665: 
  9666:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
  9667:       [InstrStage<1, [SLOT3]>], [4, 1],
  9668:       [Hex_FWD, Hex_FWD]>,
  9669: 
  9670:     InstrItinData <tc_6ae3426b, /*tc_3x*/
  9671:       [InstrStage<1, [SLOT3]>], [4, 1],
  9672:       [Hex_FWD, Hex_FWD]>,
  9673: 
  9674:     InstrItinData <tc_6d861a95, /*tc_3x*/
  9675:       [InstrStage<1, [SLOT3]>], [2, 1],
  9676:       [Hex_FWD, Hex_FWD]>,
  9677: 
  9678:     InstrItinData <tc_6e20402a, /*tc_st*/
  9679:       [InstrStage<1, [SLOT0]>], [2, 3],
  9680:       [Hex_FWD, Hex_FWD]>,
  9681: 
  9682:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
  9683:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
  9684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9685: 
  9686:     InstrItinData <tc_6fb52018, /*tc_3stall*/
  9687:       [InstrStage<1, [SLOT0]>], [1, 1],
  9688:       [Hex_FWD, Hex_FWD]>,
  9689: 
  9690:     InstrItinData <tc_6fc5dbea, /*tc_1*/
  9691:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  9692:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9693: 
  9694:     InstrItinData <tc_711c805f, /*tc_1*/
  9695:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
  9696:       [Hex_FWD, Hex_FWD]>,
  9697: 
  9698:     InstrItinData <tc_713b66bf, /*tc_1*/
  9699:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  9700:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9701: 
  9702:     InstrItinData <tc_7401744f, /*tc_2*/
  9703:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
  9704:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9705: 
  9706:     InstrItinData <tc_7476d766, /*tc_3stall*/
  9707:       [InstrStage<1, [SLOT3]>], [4, 2],
  9708:       [Hex_FWD, Hex_FWD]>,
  9709: 
  9710:     InstrItinData <tc_74a42bda, /*tc_ld*/
  9711:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
  9712:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9713: 
  9714:     InstrItinData <tc_759e57be, /*tc_3stall*/
  9715:       [InstrStage<1, [SLOT2]>], [4, 1],
  9716:       [Hex_FWD, Hex_FWD]>,
  9717: 
  9718:     InstrItinData <tc_76bb5435, /*tc_ld*/
  9719:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
  9720:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9721: 
  9722:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
  9723:       [InstrStage<1, [SLOT2]>], [1],
  9724:       [Hex_FWD]>,
  9725: 
  9726:     InstrItinData <tc_77f94a5e, /*tc_st*/
  9727:       [InstrStage<1, [SLOT0]>], [],
  9728:       []>,
  9729: 
  9730:     InstrItinData <tc_788b1d09, /*tc_3x*/
  9731:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
  9732:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9733: 
  9734:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
  9735:       [InstrStage<1, [SLOT0]>], [],
  9736:       []>,
  9737: 
  9738:     InstrItinData <tc_7af3a37e, /*tc_st*/
  9739:       [InstrStage<1, [SLOT0]>], [1, 3],
  9740:       [Hex_FWD, Hex_FWD]>,
  9741: 
  9742:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
  9743:       [InstrStage<1, [SLOT0]>], [3, 2],
  9744:       [Hex_FWD, Hex_FWD]>,
  9745: 
  9746:     InstrItinData <tc_7c28bd7e, /*tc_st*/
  9747:       [InstrStage<1, [SLOT0]>], [3],
  9748:       [Hex_FWD]>,
  9749: 
  9750:     InstrItinData <tc_7c31e19a, /*tc_st*/
  9751:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  9752:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9753: 
  9754:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
  9755:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
  9756:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9757: 
  9758:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
  9759:       [InstrStage<1, [SLOT3]>], [4, 1],
  9760:       [Hex_FWD, Hex_FWD]>,
  9761: 
  9762:     InstrItinData <tc_7f58404a, /*tc_3stall*/
  9763:       [InstrStage<1, [SLOT3]>], [],
  9764:       []>,
  9765: 
  9766:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
  9767:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
  9768:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9769: 
  9770:     InstrItinData <tc_7f8ae742, /*tc_3x*/
  9771:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
  9772:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9773: 
  9774:     InstrItinData <tc_8035e91f, /*tc_st*/
  9775:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
  9776:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9777: 
  9778:     InstrItinData <tc_822c3c68, /*tc_ld*/
  9779:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
  9780:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9781: 
  9782:     InstrItinData <tc_829d8a86, /*tc_st*/
  9783:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
  9784:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9785: 
  9786:     InstrItinData <tc_838c4d7a, /*tc_st*/
  9787:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
  9788:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9789: 
  9790:     InstrItinData <tc_84a7500d, /*tc_2*/
  9791:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  9792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9793: 
  9794:     InstrItinData <tc_86173609, /*tc_2latepred*/
  9795:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
  9796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9797: 
  9798:     InstrItinData <tc_887d1bb7, /*tc_st*/
  9799:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
  9800:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9801: 
  9802:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
  9803:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
  9804:       [Hex_FWD, Hex_FWD]>,
  9805: 
  9806:     InstrItinData <tc_8a825db2, /*tc_2*/
  9807:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9808:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9809: 
  9810:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
  9811:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  9812:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9813: 
  9814:     InstrItinData <tc_8e82e8ca, /*tc_st*/
  9815:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
  9816:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9817: 
  9818:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
  9819:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
  9820:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9821: 
  9822:     InstrItinData <tc_9124c04f, /*tc_1*/
  9823:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
  9824:       [Hex_FWD, Hex_FWD]>,
  9825: 
  9826:     InstrItinData <tc_92240447, /*tc_st*/
  9827:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
  9828:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9829: 
  9830:     InstrItinData <tc_934753bb, /*tc_ld*/
  9831:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
  9832:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9833: 
  9834:     InstrItinData <tc_937dd41c, /*tc_ld*/
  9835:       [InstrStage<1, [SLOT0, SLOT1]>], [],
  9836:       []>,
  9837: 
  9838:     InstrItinData <tc_9406230a, /*tc_3x*/
  9839:       [InstrStage<1, [SLOT3]>], [2, 1],
  9840:       [Hex_FWD, Hex_FWD]>,
  9841: 
  9842:     InstrItinData <tc_95a33176, /*tc_2*/
  9843:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
  9844:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9845: 
  9846:     InstrItinData <tc_95f43c5e, /*tc_3*/
  9847:       [InstrStage<1, [SLOT2]>], [1],
  9848:       [Hex_FWD]>,
  9849: 
  9850:     InstrItinData <tc_96ef76ef, /*tc_st*/
  9851:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
  9852:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9853: 
  9854:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
  9855:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
  9856:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9857: 
  9858:     InstrItinData <tc_9783714b, /*tc_4x*/
  9859:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
  9860:       [Hex_FWD, Hex_FWD]>,
  9861: 
  9862:     InstrItinData <tc_9b20a062, /*tc_3stall*/
  9863:       [InstrStage<1, [SLOT2]>], [4, 1],
  9864:       [Hex_FWD, Hex_FWD]>,
  9865: 
  9866:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
  9867:       [InstrStage<1, [SLOT2]>], [],
  9868:       []>,
  9869: 
  9870:     InstrItinData <tc_9b3c0462, /*tc_2*/
  9871:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9872:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9873: 
  9874:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
  9875:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
  9876:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9877: 
  9878:     InstrItinData <tc_9c52f549, /*tc_1*/
  9879:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
  9880:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9881: 
  9882:     InstrItinData <tc_9e27f2f9, /*tc_1*/
  9883:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
  9884:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9885: 
  9886:     InstrItinData <tc_9e72dc89, /*tc_4x*/
  9887:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
  9888:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9889: 
  9890:     InstrItinData <tc_9edb7c77, /*tc_4x*/
  9891:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
  9892:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9893: 
  9894:     InstrItinData <tc_9edefe01, /*tc_st*/
  9895:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
  9896:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9897: 
  9898:     InstrItinData <tc_9f6cd987, /*tc_1*/
  9899:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
  9900:       [Hex_FWD, Hex_FWD]>,
  9901: 
  9902:     InstrItinData <tc_a08b630b, /*tc_2*/
  9903:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
  9904:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9905: 
  9906:     InstrItinData <tc_a1297125, /*tc_1*/
  9907:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
  9908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9909: 
  9910:     InstrItinData <tc_a154b476, /*tc_3x*/
  9911:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
  9912:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9913: 
  9914:     InstrItinData <tc_a2b365d2, /*tc_st*/
  9915:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
  9916:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9917: 
  9918:     InstrItinData <tc_a3070909, /*tc_3stall*/
  9919:       [InstrStage<1, [SLOT0]>], [1, 1],
  9920:       [Hex_FWD, Hex_FWD]>,
  9921: 
  9922:     InstrItinData <tc_a32e03e7, /*tc_ld*/
  9923:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
  9924:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9925: 
  9926:     InstrItinData <tc_a38c45dc, /*tc_3x*/
  9927:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
  9928:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9929: 
  9930:     InstrItinData <tc_a4e22bbd, /*tc_2*/
  9931:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
  9932:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9933: 
  9934:     InstrItinData <tc_a4ee89db, /*tc_2early*/
  9935:       [InstrStage<1, [SLOT0]>], [],
  9936:       []>,
  9937: 
  9938:     InstrItinData <tc_a724463d, /*tc_3stall*/
  9939:       [InstrStage<1, [SLOT0]>], [4, 1],
  9940:       [Hex_FWD, Hex_FWD]>,
  9941: 
  9942:     InstrItinData <tc_a7a13fac, /*tc_1*/
  9943:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
  9944:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9945: 
  9946:     InstrItinData <tc_a7bdb22c, /*tc_2*/
  9947:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
  9948:       [Hex_FWD, Hex_FWD]>,
  9949: 
  9950:     InstrItinData <tc_a9edeffa, /*tc_st*/
  9951:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  9952:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9953: 
  9954:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
  9955:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
  9956:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9957: 
  9958:     InstrItinData <tc_ac65613f, /*tc_ld*/
  9959:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
  9960:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9961: 
  9962:     InstrItinData <tc_addc37a8, /*tc_st*/
  9963:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
  9964:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9965: 
  9966:     InstrItinData <tc_ae5babd7, /*tc_st*/
  9967:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
  9968:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
  9969: 
  9970:     InstrItinData <tc_aee6250c, /*tc_ld*/
  9971:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
  9972:       [Hex_FWD, Hex_FWD]>,
  9973: 
  9974:     InstrItinData <tc_af6af259, /*tc_ld*/
  9975:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
  9976:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9977: 
  9978:     InstrItinData <tc_b1ae5f67, /*tc_st*/
  9979:       [InstrStage<1, [SLOT0]>], [1],
  9980:       [Hex_FWD]>,
  9981: 
  9982:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
  9983:       [InstrStage<1, [SLOT3]>], [1, 1],
  9984:       [Hex_FWD, Hex_FWD]>,
  9985: 
  9986:     InstrItinData <tc_b3d46584, /*tc_st*/
  9987:       [InstrStage<1, [SLOT0]>], [],
  9988:       []>,
  9989: 
  9990:     InstrItinData <tc_b4dc7630, /*tc_st*/
  9991:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
  9992:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9993: 
  9994:     InstrItinData <tc_b7c4062a, /*tc_ld*/
  9995:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
  9996:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
  9997: 
  9998:     InstrItinData <tc_b837298f, /*tc_1*/
  9999:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 10000:       []>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 10001-10500 / 第 10001-10500 行

```tablegen
 10001: 
 10002:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
 10003:       [InstrStage<1, [SLOT2]>], [],
 10004:       []>,
 10005: 
 10006:     InstrItinData <tc_ba9255a6, /*tc_st*/
 10007:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
 10008:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10009: 
 10010:     InstrItinData <tc_bb07f2c5, /*tc_st*/
 10011:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
 10012:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10013: 
 10014:     InstrItinData <tc_bb78483e, /*tc_3stall*/
 10015:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 10016:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10017: 
 10018:     InstrItinData <tc_bb831a7c, /*tc_2*/
 10019:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
 10020:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10021: 
 10022:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
 10023:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
 10024:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10025: 
 10026:     InstrItinData <tc_c20701f0, /*tc_2*/
 10027:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10028:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10029: 
 10030:     InstrItinData <tc_c21d7447, /*tc_3x*/
 10031:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 10032:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10033: 
 10034:     InstrItinData <tc_c57d9f39, /*tc_1*/
 10035:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 10036:       [Hex_FWD, Hex_FWD]>,
 10037: 
 10038:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
 10039:       [InstrStage<1, [SLOT0]>], [],
 10040:       []>,
 10041: 
 10042:     InstrItinData <tc_ce59038e, /*tc_st*/
 10043:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
 10044:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10045: 
 10046:     InstrItinData <tc_cfa0e29b, /*tc_st*/
 10047:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
 10048:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10049: 
 10050:     InstrItinData <tc_d03278fd, /*tc_st*/
 10051:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
 10052:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10053: 
 10054:     InstrItinData <tc_d234b61a, /*tc_st*/
 10055:       [InstrStage<1, [SLOT0]>], [1],
 10056:       [Hex_FWD]>,
 10057: 
 10058:     InstrItinData <tc_d33e5eee, /*tc_1*/
 10059:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 10060:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10061: 
 10062:     InstrItinData <tc_d3632d88, /*tc_2*/
 10063:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 10064:       [Hex_FWD, Hex_FWD]>,
 10065: 
 10066:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
 10067:       [InstrStage<1, [SLOT0]>], [1],
 10068:       [Hex_FWD]>,
 10069: 
 10070:     InstrItinData <tc_d57d649c, /*tc_3stall*/
 10071:       [InstrStage<1, [SLOT2]>], [2],
 10072:       [Hex_FWD]>,
 10073: 
 10074:     InstrItinData <tc_d61dfdc3, /*tc_2*/
 10075:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 10076:       [Hex_FWD, Hex_FWD]>,
 10077: 
 10078:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
 10079:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 10080:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10081: 
 10082:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
 10083:       [InstrStage<1, [SLOT3]>], [2, 1],
 10084:       [Hex_FWD, Hex_FWD]>,
 10085: 
 10086:     InstrItinData <tc_d7718fbe, /*tc_3x*/
 10087:       [InstrStage<1, [SLOT3]>], [1],
 10088:       [Hex_FWD]>,
 10089: 
 10090:     InstrItinData <tc_db596beb, /*tc_3x*/
 10091:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 10092:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10093: 
 10094:     InstrItinData <tc_db96aa6b, /*tc_st*/
 10095:       [InstrStage<1, [SLOT0]>], [1],
 10096:       [Hex_FWD]>,
 10097: 
 10098:     InstrItinData <tc_dc51281d, /*tc_3*/
 10099:       [InstrStage<1, [SLOT2]>], [2, 1],
 10100:       [Hex_FWD, Hex_FWD]>,
 10101: 
 10102:     InstrItinData <tc_decdde8a, /*tc_1*/
 10103:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 10104:       [Hex_FWD]>,
 10105: 
 10106:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
 10107:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
 10108:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10109: 
 10110:     InstrItinData <tc_e3d699e3, /*tc_2*/
 10111:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 10112:       [Hex_FWD, Hex_FWD]>,
 10113: 
 10114:     InstrItinData <tc_e60def48, /*tc_1*/
 10115:       [InstrStage<1, [SLOT2]>], [2],
 10116:       [Hex_FWD]>,
 10117: 
 10118:     InstrItinData <tc_e9170fb7, /*tc_ld*/
 10119:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 10120:       [Hex_FWD, Hex_FWD]>,
 10121: 
 10122:     InstrItinData <tc_ed03645c, /*tc_1*/
 10123:       [InstrStage<1, [SLOT2]>], [3, 2],
 10124:       [Hex_FWD, Hex_FWD]>,
 10125: 
 10126:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
 10127:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 10128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10129: 
 10130:     InstrItinData <tc_eed07714, /*tc_ld*/
 10131:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 10132:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10133: 
 10134:     InstrItinData <tc_eeda4109, /*tc_1*/
 10135:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 10136:       [Hex_FWD, Hex_FWD]>,
 10137: 
 10138:     InstrItinData <tc_ef921005, /*tc_1*/
 10139:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 10140:       [Hex_FWD, Hex_FWD]>,
 10141: 
 10142:     InstrItinData <tc_f098b237, /*tc_2*/
 10143:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10144:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10145: 
 10146:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
 10147:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 10148:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10149: 
 10150:     InstrItinData <tc_f0e8e832, /*tc_4x*/
 10151:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 10152:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10153: 
 10154:     InstrItinData <tc_f34c1c21, /*tc_2*/
 10155:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10156:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10157: 
 10158:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
 10159:       [InstrStage<1, [SLOT0]>], [2],
 10160:       [Hex_FWD]>,
 10161: 
 10162:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
 10163:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
 10164:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10165: 
 10166:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
 10167:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
 10168:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10169: 
 10170:     InstrItinData <tc_f7569068, /*tc_4x*/
 10171:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
 10172:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10173: 
 10174:     InstrItinData <tc_f97707c1, /*tc_1*/
 10175:       [InstrStage<1, [SLOT2]>], [2],
 10176:       [Hex_FWD]>,
 10177: 
 10178:     InstrItinData <tc_f999c66e, /*tc_1*/
 10179:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 10180:       [Hex_FWD, Hex_FWD]>,
 10181: 
 10182:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
 10183:       [InstrStage<1, [SLOT3]>], [4, 2],
 10184:       [Hex_FWD, Hex_FWD]>,
 10185: 
 10186:     InstrItinData <tc_fedb7e19, /*tc_ld*/
 10187:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
 10188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
 10189:   ];
 10190: }
 10191: 
 10192: class DepScalarItinV71T {
 10193:   list<InstrItinData> DepScalarItinV71T_list = [
 10194:     InstrItinData <tc_011e0e9d, /*tc_st*/
 10195:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
 10196:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10197: 
 10198:     InstrItinData <tc_01d44cb2, /*tc_2*/
 10199:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10200:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10201: 
 10202:     InstrItinData <tc_01e1be3b, /*tc_3x*/
 10203:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
 10204:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10205: 
 10206:     InstrItinData <tc_02fe1c65, /*tc_4x*/
 10207:       [InstrStage<1, [SLOT3]>], [5, 1, 1],
 10208:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10209: 
 10210:     InstrItinData <tc_0655b949, /*tc_st*/
 10211:       [InstrStage<1, [SLOT0]>], [2, 3],
 10212:       [Hex_FWD, Hex_FWD]>,
 10213: 
 10214:     InstrItinData <tc_075c8dd8, /*tc_ld*/
 10215:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2],
 10216:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10217: 
 10218:     InstrItinData <tc_0a195f2c, /*tc_4x*/
 10219:       [InstrStage<1, [SLOT3]>], [5, 2, 1, 1],
 10220:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10221: 
 10222:     InstrItinData <tc_0a43be35, /*tc_3x*/
 10223:       [InstrStage<1, [SLOT3]>], [1],
 10224:       [Hex_FWD]>,
 10225: 
 10226:     InstrItinData <tc_0a6c20ae, /*tc_st*/
 10227:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
 10228:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10229: 
 10230:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
 10231:       [InstrStage<1, [SLOT2]>], [1],
 10232:       [Hex_FWD]>,
 10233: 
 10234:     InstrItinData <tc_0dfac0a7, /*tc_2*/
 10235:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10236:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10237: 
 10238:     InstrItinData <tc_0fac1eb8, /*tc_st*/
 10239:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
 10240:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10241: 
 10242:     InstrItinData <tc_112d30d6, /*tc_1*/
 10243:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2],
 10244:       [Hex_FWD]>,
 10245: 
 10246:     InstrItinData <tc_1242dc2a, /*tc_ld*/
 10247:       [InstrStage<1, [SLOT0]>], [2],
 10248:       [Hex_FWD]>,
 10249: 
 10250:     InstrItinData <tc_1248597c, /*tc_3x*/
 10251:       [InstrStage<1, [SLOT3]>], [2, 2],
 10252:       [Hex_FWD, Hex_FWD]>,
 10253: 
 10254:     InstrItinData <tc_139ef484, /*tc_3stall*/
 10255:       [InstrStage<1, [SLOT2]>], [1, 1],
 10256:       [Hex_FWD, Hex_FWD]>,
 10257: 
 10258:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
 10259:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
 10260:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10261: 
 10262:     InstrItinData <tc_151bf368, /*tc_1*/
 10263:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 10264:       [Hex_FWD, Hex_FWD]>,
 10265: 
 10266:     InstrItinData <tc_158aa3f7, /*tc_st*/
 10267:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
 10268:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10269: 
 10270:     InstrItinData <tc_197dce51, /*tc_3x*/
 10271:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
 10272:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10273: 
 10274:     InstrItinData <tc_1981450d, /*tc_newvjump*/
 10275:       [InstrStage<1, [SLOT0]>], [3],
 10276:       [Hex_FWD]>,
 10277: 
 10278:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
 10279:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2, 2],
 10280:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10281: 
 10282:     InstrItinData <tc_1c7522a8, /*tc_ld*/
 10283:       [InstrStage<1, [SLOT0]>], [4, 3, 2, 1, 2],
 10284:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10285: 
 10286:     InstrItinData <tc_1d41f8b7, /*tc_1*/
 10287:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
 10288:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10289: 
 10290:     InstrItinData <tc_1fcb8495, /*tc_2*/
 10291:       [InstrStage<1, [SLOT3]>], [4, 2, 2],
 10292:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10293: 
 10294:     InstrItinData <tc_1fe4ab69, /*tc_st*/
 10295:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
 10296:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10297: 
 10298:     InstrItinData <tc_20131976, /*tc_2*/
 10299:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10300:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10301: 
 10302:     InstrItinData <tc_2237d952, /*tc_ld*/
 10303:       [InstrStage<1, [SLOT0]>], [1, 2],
 10304:       [Hex_FWD, Hex_FWD]>,
 10305: 
 10306:     InstrItinData <tc_23708a21, /*tc_1*/
 10307:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [],
 10308:       []>,
 10309: 
 10310:     InstrItinData <tc_2471c1c8, /*tc_ld*/
 10311:       [InstrStage<1, [SLOT0]>], [4, 1],
 10312:       [Hex_FWD, Hex_FWD]>,
 10313: 
 10314:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
 10315:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 10316:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10317: 
 10318:     InstrItinData <tc_24f426ab, /*tc_1*/
 10319:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2, 2],
 10320:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10321: 
 10322:     InstrItinData <tc_27106296, /*tc_3x*/
 10323:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
 10324:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10325: 
 10326:     InstrItinData <tc_280f7fe1, /*tc_st*/
 10327:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
 10328:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10329: 
 10330:     InstrItinData <tc_28e55c6f, /*tc_3x*/
 10331:       [InstrStage<1, [SLOT3]>], [1, 1],
 10332:       [Hex_FWD, Hex_FWD]>,
 10333: 
 10334:     InstrItinData <tc_2c13e7f5, /*tc_2*/
 10335:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2],
 10336:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10337: 
 10338:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
 10339:       [InstrStage<1, [SLOT3]>], [1],
 10340:       [Hex_FWD]>,
 10341: 
 10342:     InstrItinData <tc_2f573607, /*tc_1*/
 10343:       [InstrStage<1, [SLOT2]>], [2, 2],
 10344:       [Hex_FWD, Hex_FWD]>,
 10345: 
 10346:     InstrItinData <tc_33e7e673, /*tc_2early*/
 10347:       [InstrStage<1, [SLOT2]>], [],
 10348:       []>,
 10349: 
 10350:     InstrItinData <tc_362b0be2, /*tc_3*/
 10351:       [InstrStage<1, [SLOT2]>], [1],
 10352:       [Hex_FWD]>,
 10353: 
 10354:     InstrItinData <tc_38382228, /*tc_3x*/
 10355:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
 10356:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10357: 
 10358:     InstrItinData <tc_388f9897, /*tc_1*/
 10359:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 10360:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10361: 
 10362:     InstrItinData <tc_38e0bae9, /*tc_3x*/
 10363:       [InstrStage<1, [SLOT3]>], [4, 4, 2, 1, 1],
 10364:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10365: 
 10366:     InstrItinData <tc_3d14a17b, /*tc_1*/
 10367:       [InstrStage<1, [SLOT0]>], [3, 2],
 10368:       [Hex_FWD, Hex_FWD]>,
 10369: 
 10370:     InstrItinData <tc_3edca78f, /*tc_2*/
 10371:       [InstrStage<1, [SLOT3]>], [4, 2],
 10372:       [Hex_FWD, Hex_FWD]>,
 10373: 
 10374:     InstrItinData <tc_3fbf1042, /*tc_1*/
 10375:       [InstrStage<1, [SLOT0]>], [3],
 10376:       [Hex_FWD]>,
 10377: 
 10378:     InstrItinData <tc_407e96f9, /*tc_1*/
 10379:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 10380:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10381: 
 10382:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
 10383:       [InstrStage<1, [SLOT0]>], [3, 1],
 10384:       [Hex_FWD, Hex_FWD]>,
 10385: 
 10386:     InstrItinData <tc_4222e6bf, /*tc_ld*/
 10387:       [InstrStage<1, [SLOT0]>], [4, 1, 2],
 10388:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10389: 
 10390:     InstrItinData <tc_42ff66ba, /*tc_1*/
 10391:       [InstrStage<1, [SLOT2]>], [2, 2],
 10392:       [Hex_FWD, Hex_FWD]>,
 10393: 
 10394:     InstrItinData <tc_442395f3, /*tc_2latepred*/
 10395:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 3, 2, 2],
 10396:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10397: 
 10398:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
 10399:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 1],
 10400:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10401: 
 10402:     InstrItinData <tc_44d5a428, /*tc_st*/
 10403:       [InstrStage<1, [SLOT0]>], [1, 2],
 10404:       [Hex_FWD, Hex_FWD]>,
 10405: 
 10406:     InstrItinData <tc_44fffc58, /*tc_3*/
 10407:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 10408:       [Hex_FWD]>,
 10409: 
 10410:     InstrItinData <tc_45791fb8, /*tc_ld*/
 10411:       [InstrStage<1, [SLOT0]>], [4, 2, 1, 1, 2],
 10412:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10413: 
 10414:     InstrItinData <tc_45f9d1be, /*tc_2early*/
 10415:       [InstrStage<1, [SLOT2]>], [2],
 10416:       [Hex_FWD]>,
 10417: 
 10418:     InstrItinData <tc_46c18ecf, /*tc_3x*/
 10419:       [InstrStage<1, [SLOT3]>], [4, 1],
 10420:       [Hex_FWD, Hex_FWD]>,
 10421: 
 10422:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
 10423:       [InstrStage<1, [SLOT3]>], [4, 1],
 10424:       [Hex_FWD, Hex_FWD]>,
 10425: 
 10426:     InstrItinData <tc_4a55d03c, /*tc_1*/
 10427:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 10428:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10429: 
 10430:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
 10431:       [InstrStage<1, [SLOT3]>], [2, 2],
 10432:       [Hex_FWD, Hex_FWD]>,
 10433: 
 10434:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
 10435:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 10436:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10437: 
 10438:     InstrItinData <tc_4bf903b0, /*tc_st*/
 10439:       [InstrStage<1, [SLOT0]>], [3],
 10440:       [Hex_FWD]>,
 10441: 
 10442:     InstrItinData <tc_503ce0f3, /*tc_3x*/
 10443:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 1],
 10444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10445: 
 10446:     InstrItinData <tc_512b1653, /*tc_st*/
 10447:       [InstrStage<1, [SLOT0]>], [1, 2],
 10448:       [Hex_FWD, Hex_FWD]>,
 10449: 
 10450:     InstrItinData <tc_53c851ab, /*tc_3stall*/
 10451:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
 10452:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10453: 
 10454:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
 10455:       [InstrStage<1, [SLOT3]>], [1],
 10456:       [Hex_FWD]>,
 10457: 
 10458:     InstrItinData <tc_5502c366, /*tc_1*/
 10459:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 10460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10461: 
 10462:     InstrItinData <tc_55255f2b, /*tc_3stall*/
 10463:       [InstrStage<1, [SLOT3]>], [],
 10464:       []>,
 10465: 
 10466:     InstrItinData <tc_556f6577, /*tc_3x*/
 10467:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 10468:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10469: 
 10470:     InstrItinData <tc_55a9a350, /*tc_st*/
 10471:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
 10472:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10473: 
 10474:     InstrItinData <tc_55b33fda, /*tc_1*/
 10475:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 10476:       [Hex_FWD, Hex_FWD]>,
 10477: 
 10478:     InstrItinData <tc_56a124a7, /*tc_1*/
 10479:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2],
 10480:       [Hex_FWD, Hex_FWD]>,
 10481: 
 10482:     InstrItinData <tc_57a55b54, /*tc_1*/
 10483:       [InstrStage<1, [SLOT3]>], [2, 2],
 10484:       [Hex_FWD, Hex_FWD]>,
 10485: 
 10486:     InstrItinData <tc_5944960d, /*tc_ld*/
 10487:       [InstrStage<1, [SLOT0]>], [1, 1, 2],
 10488:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10489: 
 10490:     InstrItinData <tc_59a7822c, /*tc_1*/
 10491:       [InstrStage<1, [SLOT0]>], [2, 2],
 10492:       [Hex_FWD, Hex_FWD]>,
 10493: 
 10494:     InstrItinData <tc_5a222e89, /*tc_2early*/
 10495:       [InstrStage<1, [SLOT2]>], [1, 1],
 10496:       [Hex_FWD, Hex_FWD]>,
 10497: 
 10498:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
 10499:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 10500:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV71T, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV71T; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV71T 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV71T 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 10501-11000 / 第 10501-11000 行

```tablegen
 10501: 
 10502:     InstrItinData <tc_5b347363, /*tc_1*/
 10503:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
 10504:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10505: 
 10506:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
 10507:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 2],
 10508:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10509: 
 10510:     InstrItinData <tc_5da50c4b, /*tc_1*/
 10511:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 10512:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10513: 
 10514:     InstrItinData <tc_5deb5e47, /*tc_st*/
 10515:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 10516:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10517: 
 10518:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
 10519:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2],
 10520:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10521: 
 10522:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
 10523:       [InstrStage<1, [SLOT0]>], [4, 4, 3, 1, 2],
 10524:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10525: 
 10526:     InstrItinData <tc_60e324ff, /*tc_1*/
 10527:       [InstrStage<1, [SLOT2]>], [2],
 10528:       [Hex_FWD]>,
 10529: 
 10530:     InstrItinData <tc_63567288, /*tc_2latepred*/
 10531:       [InstrStage<1, [SLOT0]>], [4],
 10532:       [Hex_FWD]>,
 10533: 
 10534:     InstrItinData <tc_64b00d8a, /*tc_ld*/
 10535:       [InstrStage<1, [SLOT0]>], [4, 1],
 10536:       [Hex_FWD, Hex_FWD]>,
 10537: 
 10538:     InstrItinData <tc_651cbe02, /*tc_1*/
 10539:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 10540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10541: 
 10542:     InstrItinData <tc_65279839, /*tc_2*/
 10543:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 10544:       [Hex_FWD, Hex_FWD]>,
 10545: 
 10546:     InstrItinData <tc_65cbd974, /*tc_st*/
 10547:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2],
 10548:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10549: 
 10550:     InstrItinData <tc_69bfb303, /*tc_3*/
 10551:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
 10552:       [Hex_FWD, Hex_FWD]>,
 10553: 
 10554:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
 10555:       [InstrStage<1, [SLOT3]>], [4, 1],
 10556:       [Hex_FWD, Hex_FWD]>,
 10557: 
 10558:     InstrItinData <tc_6ae3426b, /*tc_3x*/
 10559:       [InstrStage<1, [SLOT3]>], [4, 1],
 10560:       [Hex_FWD, Hex_FWD]>,
 10561: 
 10562:     InstrItinData <tc_6d861a95, /*tc_3x*/
 10563:       [InstrStage<1, [SLOT3]>], [2, 1],
 10564:       [Hex_FWD, Hex_FWD]>,
 10565: 
 10566:     InstrItinData <tc_6e20402a, /*tc_st*/
 10567:       [InstrStage<1, [SLOT0]>], [2, 3],
 10568:       [Hex_FWD, Hex_FWD]>,
 10569: 
 10570:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
 10571:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 10572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10573: 
 10574:     InstrItinData <tc_6fb52018, /*tc_3stall*/
 10575:       [InstrStage<1, [SLOT0]>], [1, 1],
 10576:       [Hex_FWD, Hex_FWD]>,
 10577: 
 10578:     InstrItinData <tc_6fc5dbea, /*tc_1*/
 10579:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 10580:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10581: 
 10582:     InstrItinData <tc_711c805f, /*tc_1*/
 10583:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2],
 10584:       [Hex_FWD, Hex_FWD]>,
 10585: 
 10586:     InstrItinData <tc_713b66bf, /*tc_1*/
 10587:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2],
 10588:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10589: 
 10590:     InstrItinData <tc_7401744f, /*tc_2*/
 10591:       [InstrStage<1, [SLOT3]>], [4, 4, 2, 2],
 10592:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10593: 
 10594:     InstrItinData <tc_7476d766, /*tc_3stall*/
 10595:       [InstrStage<1, [SLOT3]>], [4, 2],
 10596:       [Hex_FWD, Hex_FWD]>,
 10597: 
 10598:     InstrItinData <tc_74a42bda, /*tc_ld*/
 10599:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
 10600:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10601: 
 10602:     InstrItinData <tc_759e57be, /*tc_3stall*/
 10603:       [InstrStage<1, [SLOT2]>], [4, 1],
 10604:       [Hex_FWD, Hex_FWD]>,
 10605: 
 10606:     InstrItinData <tc_76bb5435, /*tc_ld*/
 10607:       [InstrStage<1, [SLOT0]>], [4, 3, 2, 1, 2, 2],
 10608:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10609: 
 10610:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
 10611:       [InstrStage<1, [SLOT2]>], [1],
 10612:       [Hex_FWD]>,
 10613: 
 10614:     InstrItinData <tc_77f94a5e, /*tc_st*/
 10615:       [InstrStage<1, [SLOT0]>], [],
 10616:       []>,
 10617: 
 10618:     InstrItinData <tc_788b1d09, /*tc_3x*/
 10619:       [InstrStage<1, [SLOT3]>], [4, 1, 1, 2],
 10620:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10621: 
 10622:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
 10623:       [InstrStage<1, [SLOT0]>], [],
 10624:       []>,
 10625: 
 10626:     InstrItinData <tc_7af3a37e, /*tc_st*/
 10627:       [InstrStage<1, [SLOT0]>], [1, 3],
 10628:       [Hex_FWD, Hex_FWD]>,
 10629: 
 10630:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
 10631:       [InstrStage<1, [SLOT0]>], [3, 2],
 10632:       [Hex_FWD, Hex_FWD]>,
 10633: 
 10634:     InstrItinData <tc_7c28bd7e, /*tc_st*/
 10635:       [InstrStage<1, [SLOT0]>], [3],
 10636:       [Hex_FWD]>,
 10637: 
 10638:     InstrItinData <tc_7c31e19a, /*tc_st*/
 10639:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
 10640:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10641: 
 10642:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
 10643:       [InstrStage<1, [SLOT0]>], [4, 2, 2],
 10644:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10645: 
 10646:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
 10647:       [InstrStage<1, [SLOT3]>], [4, 1],
 10648:       [Hex_FWD, Hex_FWD]>,
 10649: 
 10650:     InstrItinData <tc_7f58404a, /*tc_3stall*/
 10651:       [InstrStage<1, [SLOT3]>], [],
 10652:       []>,
 10653: 
 10654:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
 10655:       [InstrStage<1, [SLOT3]>], [5, 5, 1],
 10656:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10657: 
 10658:     InstrItinData <tc_7f8ae742, /*tc_3x*/
 10659:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
 10660:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10661: 
 10662:     InstrItinData <tc_8035e91f, /*tc_st*/
 10663:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
 10664:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10665: 
 10666:     InstrItinData <tc_822c3c68, /*tc_ld*/
 10667:       [InstrStage<1, [SLOT0]>], [4, 3, 2],
 10668:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10669: 
 10670:     InstrItinData <tc_829d8a86, /*tc_st*/
 10671:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
 10672:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10673: 
 10674:     InstrItinData <tc_838c4d7a, /*tc_st*/
 10675:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 10676:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10677: 
 10678:     InstrItinData <tc_84a7500d, /*tc_2*/
 10679:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 2, 2],
 10680:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10681: 
 10682:     InstrItinData <tc_86173609, /*tc_2latepred*/
 10683:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 3, 2],
 10684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10685: 
 10686:     InstrItinData <tc_887d1bb7, /*tc_st*/
 10687:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
 10688:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10689: 
 10690:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
 10691:       [InstrStage<1, [SLOT0]>], [4, 2],
 10692:       [Hex_FWD, Hex_FWD]>,
 10693: 
 10694:     InstrItinData <tc_8a825db2, /*tc_2*/
 10695:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10696:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10697: 
 10698:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
 10699:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 2, 2],
 10700:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10701: 
 10702:     InstrItinData <tc_8e82e8ca, /*tc_st*/
 10703:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
 10704:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10705: 
 10706:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
 10707:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 10708:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10709: 
 10710:     InstrItinData <tc_9124c04f, /*tc_1*/
 10711:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 10712:       [Hex_FWD, Hex_FWD]>,
 10713: 
 10714:     InstrItinData <tc_92240447, /*tc_st*/
 10715:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
 10716:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10717: 
 10718:     InstrItinData <tc_934753bb, /*tc_ld*/
 10719:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
 10720:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10721: 
 10722:     InstrItinData <tc_937dd41c, /*tc_ld*/
 10723:       [InstrStage<1, [SLOT0]>], [],
 10724:       []>,
 10725: 
 10726:     InstrItinData <tc_9406230a, /*tc_3x*/
 10727:       [InstrStage<1, [SLOT3]>], [2, 1],
 10728:       [Hex_FWD, Hex_FWD]>,
 10729: 
 10730:     InstrItinData <tc_95a33176, /*tc_2*/
 10731:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [4, 2, 2],
 10732:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10733: 
 10734:     InstrItinData <tc_95f43c5e, /*tc_3*/
 10735:       [InstrStage<1, [SLOT2]>], [1],
 10736:       [Hex_FWD]>,
 10737: 
 10738:     InstrItinData <tc_96ef76ef, /*tc_st*/
 10739:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
 10740:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10741: 
 10742:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
 10743:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 10744:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10745: 
 10746:     InstrItinData <tc_9783714b, /*tc_4x*/
 10747:       [InstrStage<1, [SLOT3]>], [5, 1],
 10748:       [Hex_FWD, Hex_FWD]>,
 10749: 
 10750:     InstrItinData <tc_9b20a062, /*tc_3stall*/
 10751:       [InstrStage<1, [SLOT2]>], [4, 1],
 10752:       [Hex_FWD, Hex_FWD]>,
 10753: 
 10754:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
 10755:       [InstrStage<1, [SLOT2]>], [],
 10756:       []>,
 10757: 
 10758:     InstrItinData <tc_9b3c0462, /*tc_2*/
 10759:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10760:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10761: 
 10762:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
 10763:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 10764:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10765: 
 10766:     InstrItinData <tc_9c52f549, /*tc_1*/
 10767:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2],
 10768:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10769: 
 10770:     InstrItinData <tc_9e27f2f9, /*tc_1*/
 10771:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2, 2],
 10772:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10773: 
 10774:     InstrItinData <tc_9e72dc89, /*tc_4x*/
 10775:       [InstrStage<1, [SLOT3]>], [5, 2, 1, 1],
 10776:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10777: 
 10778:     InstrItinData <tc_9edb7c77, /*tc_4x*/
 10779:       [InstrStage<1, [SLOT3]>], [5, 2, 1, 1, 2],
 10780:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10781: 
 10782:     InstrItinData <tc_9edefe01, /*tc_st*/
 10783:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
 10784:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10785: 
 10786:     InstrItinData <tc_9f6cd987, /*tc_1*/
 10787:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 10788:       [Hex_FWD, Hex_FWD]>,
 10789: 
 10790:     InstrItinData <tc_a08b630b, /*tc_2*/
 10791:       [InstrStage<1, [SLOT3]>], [4, 2, 2],
 10792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10793: 
 10794:     InstrItinData <tc_a1297125, /*tc_1*/
 10795:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 10796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10797: 
 10798:     InstrItinData <tc_a154b476, /*tc_3x*/
 10799:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 2],
 10800:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10801: 
 10802:     InstrItinData <tc_a2b365d2, /*tc_st*/
 10803:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
 10804:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10805: 
 10806:     InstrItinData <tc_a3070909, /*tc_3stall*/
 10807:       [InstrStage<1, [SLOT0]>], [1, 1],
 10808:       [Hex_FWD, Hex_FWD]>,
 10809: 
 10810:     InstrItinData <tc_a32e03e7, /*tc_ld*/
 10811:       [InstrStage<1, [SLOT0]>], [4, 2, 1, 2, 2],
 10812:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10813: 
 10814:     InstrItinData <tc_a38c45dc, /*tc_3x*/
 10815:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1, 2],
 10816:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10817: 
 10818:     InstrItinData <tc_a4e22bbd, /*tc_2*/
 10819:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2],
 10820:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10821: 
 10822:     InstrItinData <tc_a4ee89db, /*tc_2early*/
 10823:       [InstrStage<1, [SLOT0]>], [],
 10824:       []>,
 10825: 
 10826:     InstrItinData <tc_a724463d, /*tc_3stall*/
 10827:       [InstrStage<1, [SLOT0]>], [4, 1],
 10828:       [Hex_FWD, Hex_FWD]>,
 10829: 
 10830:     InstrItinData <tc_a7a13fac, /*tc_1*/
 10831:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 10832:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10833: 
 10834:     InstrItinData <tc_a7bdb22c, /*tc_2*/
 10835:       [InstrStage<1, [SLOT3]>], [4, 2],
 10836:       [Hex_FWD, Hex_FWD]>,
 10837: 
 10838:     InstrItinData <tc_a9edeffa, /*tc_st*/
 10839:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 10840:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10841: 
 10842:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
 10843:       [InstrStage<1, [SLOT0]>], [4, 1, 2, 2],
 10844:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10845: 
 10846:     InstrItinData <tc_ac65613f, /*tc_ld*/
 10847:       [InstrStage<1, [SLOT0]>], [4, 3, 2, 2],
 10848:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10849: 
 10850:     InstrItinData <tc_addc37a8, /*tc_st*/
 10851:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
 10852:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10853: 
 10854:     InstrItinData <tc_ae5babd7, /*tc_st*/
 10855:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 10856:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10857: 
 10858:     InstrItinData <tc_aee6250c, /*tc_ld*/
 10859:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 10860:       [Hex_FWD, Hex_FWD]>,
 10861: 
 10862:     InstrItinData <tc_af6af259, /*tc_ld*/
 10863:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 1],
 10864:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10865: 
 10866:     InstrItinData <tc_b1ae5f67, /*tc_st*/
 10867:       [InstrStage<1, [SLOT0]>], [1],
 10868:       [Hex_FWD]>,
 10869: 
 10870:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
 10871:       [InstrStage<1, [SLOT3]>], [1, 1],
 10872:       [Hex_FWD, Hex_FWD]>,
 10873: 
 10874:     InstrItinData <tc_b3d46584, /*tc_st*/
 10875:       [InstrStage<1, [SLOT0]>], [],
 10876:       []>,
 10877: 
 10878:     InstrItinData <tc_b4dc7630, /*tc_st*/
 10879:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
 10880:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10881: 
 10882:     InstrItinData <tc_b7c4062a, /*tc_ld*/
 10883:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 1, 2],
 10884:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10885: 
 10886:     InstrItinData <tc_b837298f, /*tc_1*/
 10887:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [],
 10888:       []>,
 10889: 
 10890:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
 10891:       [InstrStage<1, [SLOT2]>], [],
 10892:       []>,
 10893: 
 10894:     InstrItinData <tc_ba9255a6, /*tc_st*/
 10895:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
 10896:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10897: 
 10898:     InstrItinData <tc_bb07f2c5, /*tc_st*/
 10899:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
 10900:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10901: 
 10902:     InstrItinData <tc_bb78483e, /*tc_3stall*/
 10903:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 10904:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10905: 
 10906:     InstrItinData <tc_bb831a7c, /*tc_2*/
 10907:       [InstrStage<1, [SLOT3]>], [4, 2, 2, 2, 2],
 10908:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10909: 
 10910:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
 10911:       [InstrStage<1, [SLOT0]>], [4, 1, 1, 2],
 10912:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10913: 
 10914:     InstrItinData <tc_c20701f0, /*tc_2*/
 10915:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 10916:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10917: 
 10918:     InstrItinData <tc_c21d7447, /*tc_3x*/
 10919:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 10920:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10921: 
 10922:     InstrItinData <tc_c57d9f39, /*tc_1*/
 10923:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2],
 10924:       [Hex_FWD, Hex_FWD]>,
 10925: 
 10926:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
 10927:       [InstrStage<1, [SLOT0]>], [],
 10928:       []>,
 10929: 
 10930:     InstrItinData <tc_ce59038e, /*tc_st*/
 10931:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
 10932:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10933: 
 10934:     InstrItinData <tc_cfa0e29b, /*tc_st*/
 10935:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
 10936:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10937: 
 10938:     InstrItinData <tc_d03278fd, /*tc_st*/
 10939:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 2],
 10940:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 10941: 
 10942:     InstrItinData <tc_d234b61a, /*tc_st*/
 10943:       [InstrStage<1, [SLOT0]>], [1],
 10944:       [Hex_FWD]>,
 10945: 
 10946:     InstrItinData <tc_d33e5eee, /*tc_1*/
 10947:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2, 2],
 10948:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10949: 
 10950:     InstrItinData <tc_d3632d88, /*tc_2*/
 10951:       [InstrStage<1, [SLOT3]>], [4, 2],
 10952:       [Hex_FWD, Hex_FWD]>,
 10953: 
 10954:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
 10955:       [InstrStage<1, [SLOT0]>], [1],
 10956:       [Hex_FWD]>,
 10957: 
 10958:     InstrItinData <tc_d57d649c, /*tc_3stall*/
 10959:       [InstrStage<1, [SLOT2]>], [2],
 10960:       [Hex_FWD]>,
 10961: 
 10962:     InstrItinData <tc_d61dfdc3, /*tc_2*/
 10963:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 10964:       [Hex_FWD, Hex_FWD]>,
 10965: 
 10966:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
 10967:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 10968:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10969: 
 10970:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
 10971:       [InstrStage<1, [SLOT3]>], [2, 1],
 10972:       [Hex_FWD, Hex_FWD]>,
 10973: 
 10974:     InstrItinData <tc_d7718fbe, /*tc_3x*/
 10975:       [InstrStage<1, [SLOT3]>], [1],
 10976:       [Hex_FWD]>,
 10977: 
 10978:     InstrItinData <tc_db596beb, /*tc_3x*/
 10979:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 10980:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10981: 
 10982:     InstrItinData <tc_db96aa6b, /*tc_st*/
 10983:       [InstrStage<1, [SLOT0]>], [1],
 10984:       [Hex_FWD]>,
 10985: 
 10986:     InstrItinData <tc_dc51281d, /*tc_3*/
 10987:       [InstrStage<1, [SLOT2]>], [2, 1],
 10988:       [Hex_FWD, Hex_FWD]>,
 10989: 
 10990:     InstrItinData <tc_decdde8a, /*tc_1*/
 10991:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2],
 10992:       [Hex_FWD]>,
 10993: 
 10994:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
 10995:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
 10996:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 10997: 
 10998:     InstrItinData <tc_e3d699e3, /*tc_2*/
 10999:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 11000:       [Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 11001-11500 / 第 11001-11500 行

```tablegen
 11001: 
 11002:     InstrItinData <tc_e60def48, /*tc_1*/
 11003:       [InstrStage<1, [SLOT2]>], [2],
 11004:       [Hex_FWD]>,
 11005: 
 11006:     InstrItinData <tc_e9170fb7, /*tc_ld*/
 11007:       [InstrStage<1, [SLOT0]>], [4, 1],
 11008:       [Hex_FWD, Hex_FWD]>,
 11009: 
 11010:     InstrItinData <tc_ed03645c, /*tc_1*/
 11011:       [InstrStage<1, [SLOT2]>], [3, 2],
 11012:       [Hex_FWD, Hex_FWD]>,
 11013: 
 11014:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
 11015:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 11016:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11017: 
 11018:     InstrItinData <tc_eed07714, /*tc_ld*/
 11019:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 11020:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11021: 
 11022:     InstrItinData <tc_eeda4109, /*tc_1*/
 11023:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [3, 2],
 11024:       [Hex_FWD, Hex_FWD]>,
 11025: 
 11026:     InstrItinData <tc_ef921005, /*tc_1*/
 11027:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 11028:       [Hex_FWD, Hex_FWD]>,
 11029: 
 11030:     InstrItinData <tc_f098b237, /*tc_2*/
 11031:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11032:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11033: 
 11034:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
 11035:       [InstrStage<1, [SLOT3]>], [4, 1, 1, 2],
 11036:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11037: 
 11038:     InstrItinData <tc_f0e8e832, /*tc_4x*/
 11039:       [InstrStage<1, [SLOT3]>], [5, 1, 1],
 11040:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11041: 
 11042:     InstrItinData <tc_f34c1c21, /*tc_2*/
 11043:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11044:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11045: 
 11046:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
 11047:       [InstrStage<1, [SLOT0]>], [2],
 11048:       [Hex_FWD]>,
 11049: 
 11050:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
 11051:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
 11052:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11053: 
 11054:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
 11055:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
 11056:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11057: 
 11058:     InstrItinData <tc_f7569068, /*tc_4x*/
 11059:       [InstrStage<1, [SLOT3]>], [5, 5, 1, 1],
 11060:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11061: 
 11062:     InstrItinData <tc_f97707c1, /*tc_1*/
 11063:       [InstrStage<1, [SLOT2]>], [2],
 11064:       [Hex_FWD]>,
 11065: 
 11066:     InstrItinData <tc_f999c66e, /*tc_1*/
 11067:       [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 2],
 11068:       [Hex_FWD, Hex_FWD]>,
 11069: 
 11070:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
 11071:       [InstrStage<1, [SLOT3]>], [4, 2],
 11072:       [Hex_FWD, Hex_FWD]>,
 11073: 
 11074:     InstrItinData <tc_fedb7e19, /*tc_ld*/
 11075:       [InstrStage<1, [SLOT0]>], [4, 2, 1, 2],
 11076:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
 11077:   ];
 11078: }
 11079: 
 11080: class DepScalarItinV73 {
 11081:   list<InstrItinData> DepScalarItinV73_list = [
 11082:     InstrItinData <tc_011e0e9d, /*tc_st*/
 11083:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
 11084:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11085: 
 11086:     InstrItinData <tc_01d44cb2, /*tc_2*/
 11087:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11088:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11089: 
 11090:     InstrItinData <tc_01e1be3b, /*tc_3x*/
 11091:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 11092:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11093: 
 11094:     InstrItinData <tc_02fe1c65, /*tc_4x*/
 11095:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 11096:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11097: 
 11098:     InstrItinData <tc_0655b949, /*tc_st*/
 11099:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
 11100:       [Hex_FWD, Hex_FWD]>,
 11101: 
 11102:     InstrItinData <tc_075c8dd8, /*tc_ld*/
 11103:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
 11104:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11105: 
 11106:     InstrItinData <tc_0a195f2c, /*tc_4x*/
 11107:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 11108:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11109: 
 11110:     InstrItinData <tc_0a43be35, /*tc_3x*/
 11111:       [InstrStage<1, [SLOT3]>], [1],
 11112:       [Hex_FWD]>,
 11113: 
 11114:     InstrItinData <tc_0a6c20ae, /*tc_st*/
 11115:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
 11116:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11117: 
 11118:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
 11119:       [InstrStage<1, [SLOT2]>], [1],
 11120:       [Hex_FWD]>,
 11121: 
 11122:     InstrItinData <tc_0dfac0a7, /*tc_2*/
 11123:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11124:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11125: 
 11126:     InstrItinData <tc_0fac1eb8, /*tc_st*/
 11127:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
 11128:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11129: 
 11130:     InstrItinData <tc_112d30d6, /*tc_1*/
 11131:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 11132:       [Hex_FWD]>,
 11133: 
 11134:     InstrItinData <tc_1242dc2a, /*tc_ld*/
 11135:       [InstrStage<1, [SLOT0]>], [2],
 11136:       [Hex_FWD]>,
 11137: 
 11138:     InstrItinData <tc_1248597c, /*tc_3x*/
 11139:       [InstrStage<1, [SLOT3]>], [2, 2],
 11140:       [Hex_FWD, Hex_FWD]>,
 11141: 
 11142:     InstrItinData <tc_139ef484, /*tc_3stall*/
 11143:       [InstrStage<1, [SLOT2]>], [1, 1],
 11144:       [Hex_FWD, Hex_FWD]>,
 11145: 
 11146:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
 11147:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
 11148:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11149: 
 11150:     InstrItinData <tc_151bf368, /*tc_1*/
 11151:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 11152:       [Hex_FWD, Hex_FWD]>,
 11153: 
 11154:     InstrItinData <tc_158aa3f7, /*tc_st*/
 11155:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
 11156:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11157: 
 11158:     InstrItinData <tc_197dce51, /*tc_3x*/
 11159:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
 11160:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11161: 
 11162:     InstrItinData <tc_1981450d, /*tc_newvjump*/
 11163:       [InstrStage<1, [SLOT0]>], [3],
 11164:       [Hex_FWD]>,
 11165: 
 11166:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
 11167:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
 11168:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11169: 
 11170:     InstrItinData <tc_1c7522a8, /*tc_ld*/
 11171:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
 11172:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11173: 
 11174:     InstrItinData <tc_1d41f8b7, /*tc_1*/
 11175:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
 11176:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11177: 
 11178:     InstrItinData <tc_1fcb8495, /*tc_2*/
 11179:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11180:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11181: 
 11182:     InstrItinData <tc_1fe4ab69, /*tc_st*/
 11183:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
 11184:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11185: 
 11186:     InstrItinData <tc_20131976, /*tc_2*/
 11187:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11188:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11189: 
 11190:     InstrItinData <tc_2237d952, /*tc_ld*/
 11191:       [InstrStage<1, [SLOT0]>], [1, 2],
 11192:       [Hex_FWD, Hex_FWD]>,
 11193: 
 11194:     InstrItinData <tc_23708a21, /*tc_1*/
 11195:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 11196:       []>,
 11197: 
 11198:     InstrItinData <tc_2471c1c8, /*tc_ld*/
 11199:       [InstrStage<1, [SLOT0]>], [4, 1],
 11200:       [Hex_FWD, Hex_FWD]>,
 11201: 
 11202:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
 11203:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 11204:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11205: 
 11206:     InstrItinData <tc_24f426ab, /*tc_1*/
 11207:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 11208:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11209: 
 11210:     InstrItinData <tc_27106296, /*tc_3x*/
 11211:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
 11212:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11213: 
 11214:     InstrItinData <tc_280f7fe1, /*tc_st*/
 11215:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
 11216:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11217: 
 11218:     InstrItinData <tc_28e55c6f, /*tc_3x*/
 11219:       [InstrStage<1, [SLOT3]>], [1, 1],
 11220:       [Hex_FWD, Hex_FWD]>,
 11221: 
 11222:     InstrItinData <tc_2c13e7f5, /*tc_2*/
 11223:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 11224:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11225: 
 11226:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
 11227:       [InstrStage<1, [SLOT3]>], [1],
 11228:       [Hex_FWD]>,
 11229: 
 11230:     InstrItinData <tc_2f573607, /*tc_1*/
 11231:       [InstrStage<1, [SLOT2]>], [2, 2],
 11232:       [Hex_FWD, Hex_FWD]>,
 11233: 
 11234:     InstrItinData <tc_33e7e673, /*tc_2early*/
 11235:       [InstrStage<1, [SLOT2]>], [],
 11236:       []>,
 11237: 
 11238:     InstrItinData <tc_362b0be2, /*tc_3*/
 11239:       [InstrStage<1, [SLOT2]>], [1],
 11240:       [Hex_FWD]>,
 11241: 
 11242:     InstrItinData <tc_38382228, /*tc_3x*/
 11243:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
 11244:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11245: 
 11246:     InstrItinData <tc_388f9897, /*tc_1*/
 11247:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 11248:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11249: 
 11250:     InstrItinData <tc_38e0bae9, /*tc_3x*/
 11251:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
 11252:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11253: 
 11254:     InstrItinData <tc_3d14a17b, /*tc_1*/
 11255:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
 11256:       [Hex_FWD, Hex_FWD]>,
 11257: 
 11258:     InstrItinData <tc_3edca78f, /*tc_2*/
 11259:       [InstrStage<1, [SLOT3]>], [4, 2],
 11260:       [Hex_FWD, Hex_FWD]>,
 11261: 
 11262:     InstrItinData <tc_3fbf1042, /*tc_1*/
 11263:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
 11264:       [Hex_FWD]>,
 11265: 
 11266:     InstrItinData <tc_407e96f9, /*tc_1*/
 11267:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 11268:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11269: 
 11270:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
 11271:       [InstrStage<1, [SLOT0]>], [3, 1],
 11272:       [Hex_FWD, Hex_FWD]>,
 11273: 
 11274:     InstrItinData <tc_4222e6bf, /*tc_ld*/
 11275:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 11276:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11277: 
 11278:     InstrItinData <tc_42ff66ba, /*tc_1*/
 11279:       [InstrStage<1, [SLOT2]>], [2, 2],
 11280:       [Hex_FWD, Hex_FWD]>,
 11281: 
 11282:     InstrItinData <tc_442395f3, /*tc_2latepred*/
 11283:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
 11284:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11285: 
 11286:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
 11287:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
 11288:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11289: 
 11290:     InstrItinData <tc_44d5a428, /*tc_st*/
 11291:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
 11292:       [Hex_FWD, Hex_FWD]>,
 11293: 
 11294:     InstrItinData <tc_44fffc58, /*tc_3*/
 11295:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 11296:       [Hex_FWD]>,
 11297: 
 11298:     InstrItinData <tc_45791fb8, /*tc_ld*/
 11299:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
 11300:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11301: 
 11302:     InstrItinData <tc_45f9d1be, /*tc_2early*/
 11303:       [InstrStage<1, [SLOT2]>], [2],
 11304:       [Hex_FWD]>,
 11305: 
 11306:     InstrItinData <tc_46c18ecf, /*tc_3x*/
 11307:       [InstrStage<1, [SLOT3]>], [4, 1],
 11308:       [Hex_FWD, Hex_FWD]>,
 11309: 
 11310:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
 11311:       [InstrStage<1, [SLOT3]>], [4, 1],
 11312:       [Hex_FWD, Hex_FWD]>,
 11313: 
 11314:     InstrItinData <tc_4a55d03c, /*tc_1*/
 11315:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 11316:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11317: 
 11318:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
 11319:       [InstrStage<1, [SLOT3]>], [2, 2],
 11320:       [Hex_FWD, Hex_FWD]>,
 11321: 
 11322:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
 11323:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 11324:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11325: 
 11326:     InstrItinData <tc_4bf903b0, /*tc_st*/
 11327:       [InstrStage<1, [SLOT0]>], [3],
 11328:       [Hex_FWD]>,
 11329: 
 11330:     InstrItinData <tc_503ce0f3, /*tc_3x*/
 11331:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
 11332:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11333: 
 11334:     InstrItinData <tc_512b1653, /*tc_st*/
 11335:       [InstrStage<1, [SLOT0]>], [1, 2],
 11336:       [Hex_FWD, Hex_FWD]>,
 11337: 
 11338:     InstrItinData <tc_53c851ab, /*tc_3stall*/
 11339:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
 11340:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11341: 
 11342:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
 11343:       [InstrStage<1, [SLOT3]>], [1],
 11344:       [Hex_FWD]>,
 11345: 
 11346:     InstrItinData <tc_5502c366, /*tc_1*/
 11347:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 11348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11349: 
 11350:     InstrItinData <tc_55255f2b, /*tc_3stall*/
 11351:       [InstrStage<1, [SLOT3]>], [],
 11352:       []>,
 11353: 
 11354:     InstrItinData <tc_556f6577, /*tc_3x*/
 11355:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 11356:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11357: 
 11358:     InstrItinData <tc_55a9a350, /*tc_st*/
 11359:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
 11360:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11361: 
 11362:     InstrItinData <tc_55b33fda, /*tc_1*/
 11363:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 11364:       [Hex_FWD, Hex_FWD]>,
 11365: 
 11366:     InstrItinData <tc_56a124a7, /*tc_1*/
 11367:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 11368:       [Hex_FWD, Hex_FWD]>,
 11369: 
 11370:     InstrItinData <tc_57a55b54, /*tc_1*/
 11371:       [InstrStage<1, [SLOT3]>], [2, 2],
 11372:       [Hex_FWD, Hex_FWD]>,
 11373: 
 11374:     InstrItinData <tc_5944960d, /*tc_ld*/
 11375:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
 11376:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11377: 
 11378:     InstrItinData <tc_59a7822c, /*tc_1*/
 11379:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
 11380:       [Hex_FWD, Hex_FWD]>,
 11381: 
 11382:     InstrItinData <tc_5a222e89, /*tc_2early*/
 11383:       [InstrStage<1, [SLOT2]>], [1, 1],
 11384:       [Hex_FWD, Hex_FWD]>,
 11385: 
 11386:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
 11387:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 11388:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11389: 
 11390:     InstrItinData <tc_5b347363, /*tc_1*/
 11391:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
 11392:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11393: 
 11394:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
 11395:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
 11396:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11397: 
 11398:     InstrItinData <tc_5da50c4b, /*tc_1*/
 11399:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 11400:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11401: 
 11402:     InstrItinData <tc_5deb5e47, /*tc_st*/
 11403:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 11404:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11405: 
 11406:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
 11407:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 11408:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11409: 
 11410:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
 11411:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
 11412:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11413: 
 11414:     InstrItinData <tc_60e324ff, /*tc_1*/
 11415:       [InstrStage<1, [SLOT2]>], [2],
 11416:       [Hex_FWD]>,
 11417: 
 11418:     InstrItinData <tc_63567288, /*tc_2latepred*/
 11419:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
 11420:       [Hex_FWD]>,
 11421: 
 11422:     InstrItinData <tc_64b00d8a, /*tc_ld*/
 11423:       [InstrStage<1, [SLOT0]>], [4, 1],
 11424:       [Hex_FWD, Hex_FWD]>,
 11425: 
 11426:     InstrItinData <tc_651cbe02, /*tc_1*/
 11427:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 11428:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11429: 
 11430:     InstrItinData <tc_65279839, /*tc_2*/
 11431:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 11432:       [Hex_FWD, Hex_FWD]>,
 11433: 
 11434:     InstrItinData <tc_65cbd974, /*tc_st*/
 11435:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
 11436:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11437: 
 11438:     InstrItinData <tc_69bfb303, /*tc_3*/
 11439:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
 11440:       [Hex_FWD, Hex_FWD]>,
 11441: 
 11442:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
 11443:       [InstrStage<1, [SLOT3]>], [4, 1],
 11444:       [Hex_FWD, Hex_FWD]>,
 11445: 
 11446:     InstrItinData <tc_6ae3426b, /*tc_3x*/
 11447:       [InstrStage<1, [SLOT3]>], [4, 1],
 11448:       [Hex_FWD, Hex_FWD]>,
 11449: 
 11450:     InstrItinData <tc_6d861a95, /*tc_3x*/
 11451:       [InstrStage<1, [SLOT3]>], [2, 1],
 11452:       [Hex_FWD, Hex_FWD]>,
 11453: 
 11454:     InstrItinData <tc_6e20402a, /*tc_st*/
 11455:       [InstrStage<1, [SLOT0]>], [2, 3],
 11456:       [Hex_FWD, Hex_FWD]>,
 11457: 
 11458:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
 11459:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 11460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11461: 
 11462:     InstrItinData <tc_6fb52018, /*tc_3stall*/
 11463:       [InstrStage<1, [SLOT0]>], [1, 1],
 11464:       [Hex_FWD, Hex_FWD]>,
 11465: 
 11466:     InstrItinData <tc_6fc5dbea, /*tc_1*/
 11467:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 11468:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11469: 
 11470:     InstrItinData <tc_711c805f, /*tc_1*/
 11471:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 11472:       [Hex_FWD, Hex_FWD]>,
 11473: 
 11474:     InstrItinData <tc_713b66bf, /*tc_1*/
 11475:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 11476:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11477: 
 11478:     InstrItinData <tc_7401744f, /*tc_2*/
 11479:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
 11480:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11481: 
 11482:     InstrItinData <tc_7476d766, /*tc_3stall*/
 11483:       [InstrStage<1, [SLOT3]>], [4, 2],
 11484:       [Hex_FWD, Hex_FWD]>,
 11485: 
 11486:     InstrItinData <tc_74a42bda, /*tc_ld*/
 11487:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
 11488:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11489: 
 11490:     InstrItinData <tc_759e57be, /*tc_3stall*/
 11491:       [InstrStage<1, [SLOT2]>], [4, 1],
 11492:       [Hex_FWD, Hex_FWD]>,
 11493: 
 11494:     InstrItinData <tc_76bb5435, /*tc_ld*/
 11495:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
 11496:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11497: 
 11498:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
 11499:       [InstrStage<1, [SLOT2]>], [1],
 11500:       [Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV73, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV73; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV73 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV73 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 11501-12000 / 第 11501-12000 行

```tablegen
 11501: 
 11502:     InstrItinData <tc_77f94a5e, /*tc_st*/
 11503:       [InstrStage<1, [SLOT0]>], [],
 11504:       []>,
 11505: 
 11506:     InstrItinData <tc_788b1d09, /*tc_3x*/
 11507:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 11508:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11509: 
 11510:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
 11511:       [InstrStage<1, [SLOT0]>], [],
 11512:       []>,
 11513: 
 11514:     InstrItinData <tc_7af3a37e, /*tc_st*/
 11515:       [InstrStage<1, [SLOT0]>], [1, 3],
 11516:       [Hex_FWD, Hex_FWD]>,
 11517: 
 11518:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
 11519:       [InstrStage<1, [SLOT0]>], [3, 2],
 11520:       [Hex_FWD, Hex_FWD]>,
 11521: 
 11522:     InstrItinData <tc_7c28bd7e, /*tc_st*/
 11523:       [InstrStage<1, [SLOT0]>], [3],
 11524:       [Hex_FWD]>,
 11525: 
 11526:     InstrItinData <tc_7c31e19a, /*tc_st*/
 11527:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 11528:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11529: 
 11530:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
 11531:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
 11532:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11533: 
 11534:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
 11535:       [InstrStage<1, [SLOT3]>], [4, 1],
 11536:       [Hex_FWD, Hex_FWD]>,
 11537: 
 11538:     InstrItinData <tc_7f58404a, /*tc_3stall*/
 11539:       [InstrStage<1, [SLOT3]>], [],
 11540:       []>,
 11541: 
 11542:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
 11543:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
 11544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11545: 
 11546:     InstrItinData <tc_7f8ae742, /*tc_3x*/
 11547:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 11548:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11549: 
 11550:     InstrItinData <tc_8035e91f, /*tc_st*/
 11551:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
 11552:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11553: 
 11554:     InstrItinData <tc_822c3c68, /*tc_ld*/
 11555:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
 11556:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11557: 
 11558:     InstrItinData <tc_829d8a86, /*tc_st*/
 11559:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
 11560:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11561: 
 11562:     InstrItinData <tc_838c4d7a, /*tc_st*/
 11563:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 11564:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11565: 
 11566:     InstrItinData <tc_84a7500d, /*tc_2*/
 11567:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 11568:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11569: 
 11570:     InstrItinData <tc_86173609, /*tc_2latepred*/
 11571:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 11572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11573: 
 11574:     InstrItinData <tc_887d1bb7, /*tc_st*/
 11575:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
 11576:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11577: 
 11578:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
 11579:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
 11580:       [Hex_FWD, Hex_FWD]>,
 11581: 
 11582:     InstrItinData <tc_8a825db2, /*tc_2*/
 11583:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11584:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11585: 
 11586:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
 11587:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 11588:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11589: 
 11590:     InstrItinData <tc_8e82e8ca, /*tc_st*/
 11591:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
 11592:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11593: 
 11594:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
 11595:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
 11596:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11597: 
 11598:     InstrItinData <tc_9124c04f, /*tc_1*/
 11599:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 11600:       [Hex_FWD, Hex_FWD]>,
 11601: 
 11602:     InstrItinData <tc_92240447, /*tc_st*/
 11603:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
 11604:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11605: 
 11606:     InstrItinData <tc_934753bb, /*tc_ld*/
 11607:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
 11608:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11609: 
 11610:     InstrItinData <tc_937dd41c, /*tc_ld*/
 11611:       [InstrStage<1, [SLOT0, SLOT1]>], [],
 11612:       []>,
 11613: 
 11614:     InstrItinData <tc_9406230a, /*tc_3x*/
 11615:       [InstrStage<1, [SLOT3]>], [2, 1],
 11616:       [Hex_FWD, Hex_FWD]>,
 11617: 
 11618:     InstrItinData <tc_95a33176, /*tc_2*/
 11619:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 11620:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11621: 
 11622:     InstrItinData <tc_95f43c5e, /*tc_3*/
 11623:       [InstrStage<1, [SLOT2]>], [1],
 11624:       [Hex_FWD]>,
 11625: 
 11626:     InstrItinData <tc_96ef76ef, /*tc_st*/
 11627:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
 11628:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11629: 
 11630:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
 11631:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 11632:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11633: 
 11634:     InstrItinData <tc_9783714b, /*tc_4x*/
 11635:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
 11636:       [Hex_FWD, Hex_FWD]>,
 11637: 
 11638:     InstrItinData <tc_9b20a062, /*tc_3stall*/
 11639:       [InstrStage<1, [SLOT2]>], [4, 1],
 11640:       [Hex_FWD, Hex_FWD]>,
 11641: 
 11642:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
 11643:       [InstrStage<1, [SLOT2]>], [],
 11644:       []>,
 11645: 
 11646:     InstrItinData <tc_9b3c0462, /*tc_2*/
 11647:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11648:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11649: 
 11650:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
 11651:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 11652:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11653: 
 11654:     InstrItinData <tc_9c52f549, /*tc_1*/
 11655:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 11656:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11657: 
 11658:     InstrItinData <tc_9e27f2f9, /*tc_1*/
 11659:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 11660:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11661: 
 11662:     InstrItinData <tc_9e72dc89, /*tc_4x*/
 11663:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 11664:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11665: 
 11666:     InstrItinData <tc_9edb7c77, /*tc_4x*/
 11667:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
 11668:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11669: 
 11670:     InstrItinData <tc_9edefe01, /*tc_st*/
 11671:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
 11672:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11673: 
 11674:     InstrItinData <tc_9f6cd987, /*tc_1*/
 11675:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 11676:       [Hex_FWD, Hex_FWD]>,
 11677: 
 11678:     InstrItinData <tc_a08b630b, /*tc_2*/
 11679:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11680:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11681: 
 11682:     InstrItinData <tc_a1297125, /*tc_1*/
 11683:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 11684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11685: 
 11686:     InstrItinData <tc_a154b476, /*tc_3x*/
 11687:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
 11688:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11689: 
 11690:     InstrItinData <tc_a2b365d2, /*tc_st*/
 11691:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
 11692:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11693: 
 11694:     InstrItinData <tc_a3070909, /*tc_3stall*/
 11695:       [InstrStage<1, [SLOT0]>], [1, 1],
 11696:       [Hex_FWD, Hex_FWD]>,
 11697: 
 11698:     InstrItinData <tc_a32e03e7, /*tc_ld*/
 11699:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
 11700:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11701: 
 11702:     InstrItinData <tc_a38c45dc, /*tc_3x*/
 11703:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
 11704:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11705: 
 11706:     InstrItinData <tc_a4e22bbd, /*tc_2*/
 11707:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 11708:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11709: 
 11710:     InstrItinData <tc_a4ee89db, /*tc_2early*/
 11711:       [InstrStage<1, [SLOT0]>], [],
 11712:       []>,
 11713: 
 11714:     InstrItinData <tc_a724463d, /*tc_3stall*/
 11715:       [InstrStage<1, [SLOT0]>], [4, 1],
 11716:       [Hex_FWD, Hex_FWD]>,
 11717: 
 11718:     InstrItinData <tc_a7a13fac, /*tc_1*/
 11719:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 11720:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11721: 
 11722:     InstrItinData <tc_a7bdb22c, /*tc_2*/
 11723:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 11724:       [Hex_FWD, Hex_FWD]>,
 11725: 
 11726:     InstrItinData <tc_a9edeffa, /*tc_st*/
 11727:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 11728:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11729: 
 11730:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
 11731:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
 11732:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11733: 
 11734:     InstrItinData <tc_ac65613f, /*tc_ld*/
 11735:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
 11736:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11737: 
 11738:     InstrItinData <tc_addc37a8, /*tc_st*/
 11739:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
 11740:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11741: 
 11742:     InstrItinData <tc_ae5babd7, /*tc_st*/
 11743:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 11744:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11745: 
 11746:     InstrItinData <tc_aee6250c, /*tc_ld*/
 11747:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 11748:       [Hex_FWD, Hex_FWD]>,
 11749: 
 11750:     InstrItinData <tc_af6af259, /*tc_ld*/
 11751:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
 11752:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11753: 
 11754:     InstrItinData <tc_b1ae5f67, /*tc_st*/
 11755:       [InstrStage<1, [SLOT0]>], [1],
 11756:       [Hex_FWD]>,
 11757: 
 11758:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
 11759:       [InstrStage<1, [SLOT3]>], [1, 1],
 11760:       [Hex_FWD, Hex_FWD]>,
 11761: 
 11762:     InstrItinData <tc_b3d46584, /*tc_st*/
 11763:       [InstrStage<1, [SLOT0]>], [],
 11764:       []>,
 11765: 
 11766:     InstrItinData <tc_b4dc7630, /*tc_st*/
 11767:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
 11768:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11769: 
 11770:     InstrItinData <tc_b7c4062a, /*tc_ld*/
 11771:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
 11772:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11773: 
 11774:     InstrItinData <tc_b837298f, /*tc_1*/
 11775:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 11776:       []>,
 11777: 
 11778:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
 11779:       [InstrStage<1, [SLOT2]>], [],
 11780:       []>,
 11781: 
 11782:     InstrItinData <tc_ba9255a6, /*tc_st*/
 11783:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
 11784:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11785: 
 11786:     InstrItinData <tc_bb07f2c5, /*tc_st*/
 11787:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
 11788:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11789: 
 11790:     InstrItinData <tc_bb78483e, /*tc_3stall*/
 11791:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 11792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11793: 
 11794:     InstrItinData <tc_bb831a7c, /*tc_2*/
 11795:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
 11796:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11797: 
 11798:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
 11799:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
 11800:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11801: 
 11802:     InstrItinData <tc_c20701f0, /*tc_2*/
 11803:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11804:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11805: 
 11806:     InstrItinData <tc_c21d7447, /*tc_3x*/
 11807:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 11808:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11809: 
 11810:     InstrItinData <tc_c57d9f39, /*tc_1*/
 11811:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 11812:       [Hex_FWD, Hex_FWD]>,
 11813: 
 11814:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
 11815:       [InstrStage<1, [SLOT0]>], [],
 11816:       []>,
 11817: 
 11818:     InstrItinData <tc_ce59038e, /*tc_st*/
 11819:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
 11820:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11821: 
 11822:     InstrItinData <tc_cfa0e29b, /*tc_st*/
 11823:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
 11824:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11825: 
 11826:     InstrItinData <tc_d03278fd, /*tc_st*/
 11827:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
 11828:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11829: 
 11830:     InstrItinData <tc_d234b61a, /*tc_st*/
 11831:       [InstrStage<1, [SLOT0]>], [1],
 11832:       [Hex_FWD]>,
 11833: 
 11834:     InstrItinData <tc_d33e5eee, /*tc_1*/
 11835:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 11836:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11837: 
 11838:     InstrItinData <tc_d3632d88, /*tc_2*/
 11839:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 11840:       [Hex_FWD, Hex_FWD]>,
 11841: 
 11842:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
 11843:       [InstrStage<1, [SLOT0]>], [1],
 11844:       [Hex_FWD]>,
 11845: 
 11846:     InstrItinData <tc_d57d649c, /*tc_3stall*/
 11847:       [InstrStage<1, [SLOT2]>], [2],
 11848:       [Hex_FWD]>,
 11849: 
 11850:     InstrItinData <tc_d61dfdc3, /*tc_2*/
 11851:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 11852:       [Hex_FWD, Hex_FWD]>,
 11853: 
 11854:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
 11855:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 11856:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11857: 
 11858:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
 11859:       [InstrStage<1, [SLOT3]>], [2, 1],
 11860:       [Hex_FWD, Hex_FWD]>,
 11861: 
 11862:     InstrItinData <tc_d7718fbe, /*tc_3x*/
 11863:       [InstrStage<1, [SLOT3]>], [1],
 11864:       [Hex_FWD]>,
 11865: 
 11866:     InstrItinData <tc_db596beb, /*tc_3x*/
 11867:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 11868:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11869: 
 11870:     InstrItinData <tc_db96aa6b, /*tc_st*/
 11871:       [InstrStage<1, [SLOT0]>], [1],
 11872:       [Hex_FWD]>,
 11873: 
 11874:     InstrItinData <tc_dc51281d, /*tc_3*/
 11875:       [InstrStage<1, [SLOT2]>], [2, 1],
 11876:       [Hex_FWD, Hex_FWD]>,
 11877: 
 11878:     InstrItinData <tc_decdde8a, /*tc_1*/
 11879:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 11880:       [Hex_FWD]>,
 11881: 
 11882:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
 11883:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
 11884:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11885: 
 11886:     InstrItinData <tc_e3d699e3, /*tc_2*/
 11887:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 11888:       [Hex_FWD, Hex_FWD]>,
 11889: 
 11890:     InstrItinData <tc_e60def48, /*tc_1*/
 11891:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 11892:       [Hex_FWD]>,
 11893: 
 11894:     InstrItinData <tc_e9170fb7, /*tc_ld*/
 11895:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 11896:       [Hex_FWD, Hex_FWD]>,
 11897: 
 11898:     InstrItinData <tc_ed03645c, /*tc_1*/
 11899:       [InstrStage<1, [SLOT2]>], [3, 2],
 11900:       [Hex_FWD, Hex_FWD]>,
 11901: 
 11902:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
 11903:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 11904:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11905: 
 11906:     InstrItinData <tc_eed07714, /*tc_ld*/
 11907:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 11908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11909: 
 11910:     InstrItinData <tc_eeda4109, /*tc_1*/
 11911:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 11912:       [Hex_FWD, Hex_FWD]>,
 11913: 
 11914:     InstrItinData <tc_ef921005, /*tc_1*/
 11915:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 11916:       [Hex_FWD, Hex_FWD]>,
 11917: 
 11918:     InstrItinData <tc_f098b237, /*tc_2*/
 11919:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11920:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11921: 
 11922:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
 11923:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 11924:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11925: 
 11926:     InstrItinData <tc_f0e8e832, /*tc_4x*/
 11927:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 11928:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11929: 
 11930:     InstrItinData <tc_f34c1c21, /*tc_2*/
 11931:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11932:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11933: 
 11934:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
 11935:       [InstrStage<1, [SLOT0]>], [2],
 11936:       [Hex_FWD]>,
 11937: 
 11938:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
 11939:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
 11940:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11941: 
 11942:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
 11943:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
 11944:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11945: 
 11946:     InstrItinData <tc_f7569068, /*tc_4x*/
 11947:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
 11948:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11949: 
 11950:     InstrItinData <tc_f97707c1, /*tc_1*/
 11951:       [InstrStage<1, [SLOT2]>], [2],
 11952:       [Hex_FWD]>,
 11953: 
 11954:     InstrItinData <tc_f999c66e, /*tc_1*/
 11955:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 11956:       [Hex_FWD, Hex_FWD]>,
 11957: 
 11958:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
 11959:       [InstrStage<1, [SLOT3]>], [4, 2],
 11960:       [Hex_FWD, Hex_FWD]>,
 11961: 
 11962:     InstrItinData <tc_fedb7e19, /*tc_ld*/
 11963:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
 11964:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
 11965:   ];
 11966: }
 11967: 
 11968: class DepScalarItinV75 {
 11969:   list<InstrItinData> DepScalarItinV75_list = [
 11970:     InstrItinData <tc_011e0e9d, /*tc_st*/
 11971:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
 11972:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11973: 
 11974:     InstrItinData <tc_01d44cb2, /*tc_2*/
 11975:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 11976:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11977: 
 11978:     InstrItinData <tc_01e1be3b, /*tc_3x*/
 11979:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 11980:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11981: 
 11982:     InstrItinData <tc_02fe1c65, /*tc_4x*/
 11983:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 11984:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 11985: 
 11986:     InstrItinData <tc_0655b949, /*tc_st*/
 11987:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
 11988:       [Hex_FWD, Hex_FWD]>,
 11989: 
 11990:     InstrItinData <tc_075c8dd8, /*tc_ld*/
 11991:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
 11992:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11993: 
 11994:     InstrItinData <tc_0a195f2c, /*tc_4x*/
 11995:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 11996:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 11997: 
 11998:     InstrItinData <tc_0a43be35, /*tc_3x*/
 11999:       [InstrStage<1, [SLOT3]>], [1],
 12000:       [Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV75, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV75; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV75 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV75 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 12001-12500 / 第 12001-12500 行

```tablegen
 12001: 
 12002:     InstrItinData <tc_0a6c20ae, /*tc_st*/
 12003:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
 12004:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12005: 
 12006:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
 12007:       [InstrStage<1, [SLOT2]>], [1],
 12008:       [Hex_FWD]>,
 12009: 
 12010:     InstrItinData <tc_0dfac0a7, /*tc_2*/
 12011:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12012:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12013: 
 12014:     InstrItinData <tc_0fac1eb8, /*tc_st*/
 12015:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
 12016:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12017: 
 12018:     InstrItinData <tc_112d30d6, /*tc_1*/
 12019:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 12020:       [Hex_FWD]>,
 12021: 
 12022:     InstrItinData <tc_1242dc2a, /*tc_ld*/
 12023:       [InstrStage<1, [SLOT0]>], [2],
 12024:       [Hex_FWD]>,
 12025: 
 12026:     InstrItinData <tc_1248597c, /*tc_3x*/
 12027:       [InstrStage<1, [SLOT3]>], [2, 2],
 12028:       [Hex_FWD, Hex_FWD]>,
 12029: 
 12030:     InstrItinData <tc_139ef484, /*tc_3stall*/
 12031:       [InstrStage<1, [SLOT2]>], [1, 1],
 12032:       [Hex_FWD, Hex_FWD]>,
 12033: 
 12034:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
 12035:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
 12036:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12037: 
 12038:     InstrItinData <tc_151bf368, /*tc_1*/
 12039:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 12040:       [Hex_FWD, Hex_FWD]>,
 12041: 
 12042:     InstrItinData <tc_158aa3f7, /*tc_st*/
 12043:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
 12044:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12045: 
 12046:     InstrItinData <tc_197dce51, /*tc_3x*/
 12047:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
 12048:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12049: 
 12050:     InstrItinData <tc_1981450d, /*tc_newvjump*/
 12051:       [InstrStage<1, [SLOT0]>], [3],
 12052:       [Hex_FWD]>,
 12053: 
 12054:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
 12055:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
 12056:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12057: 
 12058:     InstrItinData <tc_1c7522a8, /*tc_ld*/
 12059:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
 12060:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12061: 
 12062:     InstrItinData <tc_1d41f8b7, /*tc_1*/
 12063:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
 12064:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12065: 
 12066:     InstrItinData <tc_1fcb8495, /*tc_2*/
 12067:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12068:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12069: 
 12070:     InstrItinData <tc_1fe4ab69, /*tc_st*/
 12071:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
 12072:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12073: 
 12074:     InstrItinData <tc_20131976, /*tc_2*/
 12075:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12076:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12077: 
 12078:     InstrItinData <tc_2237d952, /*tc_ld*/
 12079:       [InstrStage<1, [SLOT0]>], [1, 2],
 12080:       [Hex_FWD, Hex_FWD]>,
 12081: 
 12082:     InstrItinData <tc_23708a21, /*tc_1*/
 12083:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 12084:       []>,
 12085: 
 12086:     InstrItinData <tc_2471c1c8, /*tc_ld*/
 12087:       [InstrStage<1, [SLOT0]>], [4, 1],
 12088:       [Hex_FWD, Hex_FWD]>,
 12089: 
 12090:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
 12091:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 12092:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12093: 
 12094:     InstrItinData <tc_24f426ab, /*tc_1*/
 12095:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 12096:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12097: 
 12098:     InstrItinData <tc_27106296, /*tc_3x*/
 12099:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
 12100:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12101: 
 12102:     InstrItinData <tc_280f7fe1, /*tc_st*/
 12103:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
 12104:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12105: 
 12106:     InstrItinData <tc_28e55c6f, /*tc_3x*/
 12107:       [InstrStage<1, [SLOT3]>], [1, 1],
 12108:       [Hex_FWD, Hex_FWD]>,
 12109: 
 12110:     InstrItinData <tc_2c13e7f5, /*tc_2*/
 12111:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 12112:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12113: 
 12114:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
 12115:       [InstrStage<1, [SLOT3]>], [1],
 12116:       [Hex_FWD]>,
 12117: 
 12118:     InstrItinData <tc_2f573607, /*tc_1*/
 12119:       [InstrStage<1, [SLOT2]>], [2, 2],
 12120:       [Hex_FWD, Hex_FWD]>,
 12121: 
 12122:     InstrItinData <tc_33e7e673, /*tc_2early*/
 12123:       [InstrStage<1, [SLOT2]>], [],
 12124:       []>,
 12125: 
 12126:     InstrItinData <tc_362b0be2, /*tc_3*/
 12127:       [InstrStage<1, [SLOT2]>], [1],
 12128:       [Hex_FWD]>,
 12129: 
 12130:     InstrItinData <tc_38382228, /*tc_3x*/
 12131:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
 12132:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12133: 
 12134:     InstrItinData <tc_388f9897, /*tc_1*/
 12135:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 12136:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12137: 
 12138:     InstrItinData <tc_38e0bae9, /*tc_3x*/
 12139:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
 12140:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12141: 
 12142:     InstrItinData <tc_3d14a17b, /*tc_1*/
 12143:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
 12144:       [Hex_FWD, Hex_FWD]>,
 12145: 
 12146:     InstrItinData <tc_3edca78f, /*tc_2*/
 12147:       [InstrStage<1, [SLOT3]>], [4, 2],
 12148:       [Hex_FWD, Hex_FWD]>,
 12149: 
 12150:     InstrItinData <tc_3fbf1042, /*tc_1*/
 12151:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
 12152:       [Hex_FWD]>,
 12153: 
 12154:     InstrItinData <tc_407e96f9, /*tc_1*/
 12155:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 12156:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12157: 
 12158:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
 12159:       [InstrStage<1, [SLOT0]>], [3, 1],
 12160:       [Hex_FWD, Hex_FWD]>,
 12161: 
 12162:     InstrItinData <tc_4222e6bf, /*tc_ld*/
 12163:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 12164:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12165: 
 12166:     InstrItinData <tc_42ff66ba, /*tc_1*/
 12167:       [InstrStage<1, [SLOT2]>], [2, 2],
 12168:       [Hex_FWD, Hex_FWD]>,
 12169: 
 12170:     InstrItinData <tc_442395f3, /*tc_2latepred*/
 12171:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
 12172:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12173: 
 12174:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
 12175:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
 12176:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12177: 
 12178:     InstrItinData <tc_44d5a428, /*tc_st*/
 12179:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
 12180:       [Hex_FWD, Hex_FWD]>,
 12181: 
 12182:     InstrItinData <tc_44fffc58, /*tc_3*/
 12183:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 12184:       [Hex_FWD]>,
 12185: 
 12186:     InstrItinData <tc_45791fb8, /*tc_ld*/
 12187:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
 12188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12189: 
 12190:     InstrItinData <tc_45f9d1be, /*tc_2early*/
 12191:       [InstrStage<1, [SLOT2]>], [2],
 12192:       [Hex_FWD]>,
 12193: 
 12194:     InstrItinData <tc_46c18ecf, /*tc_3x*/
 12195:       [InstrStage<1, [SLOT3]>], [4, 1],
 12196:       [Hex_FWD, Hex_FWD]>,
 12197: 
 12198:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
 12199:       [InstrStage<1, [SLOT3]>], [4, 1],
 12200:       [Hex_FWD, Hex_FWD]>,
 12201: 
 12202:     InstrItinData <tc_4a55d03c, /*tc_1*/
 12203:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 12204:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12205: 
 12206:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
 12207:       [InstrStage<1, [SLOT3]>], [2, 2],
 12208:       [Hex_FWD, Hex_FWD]>,
 12209: 
 12210:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
 12211:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 12212:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12213: 
 12214:     InstrItinData <tc_4bf903b0, /*tc_st*/
 12215:       [InstrStage<1, [SLOT0]>], [3],
 12216:       [Hex_FWD]>,
 12217: 
 12218:     InstrItinData <tc_503ce0f3, /*tc_3x*/
 12219:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
 12220:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12221: 
 12222:     InstrItinData <tc_512b1653, /*tc_st*/
 12223:       [InstrStage<1, [SLOT0]>], [1, 2],
 12224:       [Hex_FWD, Hex_FWD]>,
 12225: 
 12226:     InstrItinData <tc_53c851ab, /*tc_3stall*/
 12227:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
 12228:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12229: 
 12230:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
 12231:       [InstrStage<1, [SLOT3]>], [1],
 12232:       [Hex_FWD]>,
 12233: 
 12234:     InstrItinData <tc_5502c366, /*tc_1*/
 12235:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 12236:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12237: 
 12238:     InstrItinData <tc_55255f2b, /*tc_3stall*/
 12239:       [InstrStage<1, [SLOT3]>], [],
 12240:       []>,
 12241: 
 12242:     InstrItinData <tc_556f6577, /*tc_3x*/
 12243:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 12244:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12245: 
 12246:     InstrItinData <tc_55a9a350, /*tc_st*/
 12247:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
 12248:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12249: 
 12250:     InstrItinData <tc_55b33fda, /*tc_1*/
 12251:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 12252:       [Hex_FWD, Hex_FWD]>,
 12253: 
 12254:     InstrItinData <tc_56a124a7, /*tc_1*/
 12255:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 12256:       [Hex_FWD, Hex_FWD]>,
 12257: 
 12258:     InstrItinData <tc_57a55b54, /*tc_1*/
 12259:       [InstrStage<1, [SLOT3]>], [2, 2],
 12260:       [Hex_FWD, Hex_FWD]>,
 12261: 
 12262:     InstrItinData <tc_5944960d, /*tc_ld*/
 12263:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
 12264:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12265: 
 12266:     InstrItinData <tc_59a7822c, /*tc_1*/
 12267:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
 12268:       [Hex_FWD, Hex_FWD]>,
 12269: 
 12270:     InstrItinData <tc_5a222e89, /*tc_2early*/
 12271:       [InstrStage<1, [SLOT2]>], [1, 1],
 12272:       [Hex_FWD, Hex_FWD]>,
 12273: 
 12274:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
 12275:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 12276:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12277: 
 12278:     InstrItinData <tc_5b347363, /*tc_1*/
 12279:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
 12280:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12281: 
 12282:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
 12283:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
 12284:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12285: 
 12286:     InstrItinData <tc_5da50c4b, /*tc_1*/
 12287:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 12288:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12289: 
 12290:     InstrItinData <tc_5deb5e47, /*tc_st*/
 12291:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 12292:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12293: 
 12294:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
 12295:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 12296:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12297: 
 12298:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
 12299:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
 12300:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12301: 
 12302:     InstrItinData <tc_60e324ff, /*tc_1*/
 12303:       [InstrStage<1, [SLOT2]>], [2],
 12304:       [Hex_FWD]>,
 12305: 
 12306:     InstrItinData <tc_63567288, /*tc_2latepred*/
 12307:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
 12308:       [Hex_FWD]>,
 12309: 
 12310:     InstrItinData <tc_64b00d8a, /*tc_ld*/
 12311:       [InstrStage<1, [SLOT0]>], [4, 1],
 12312:       [Hex_FWD, Hex_FWD]>,
 12313: 
 12314:     InstrItinData <tc_651cbe02, /*tc_1*/
 12315:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 12316:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12317: 
 12318:     InstrItinData <tc_65279839, /*tc_2*/
 12319:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 12320:       [Hex_FWD, Hex_FWD]>,
 12321: 
 12322:     InstrItinData <tc_65cbd974, /*tc_st*/
 12323:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
 12324:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12325: 
 12326:     InstrItinData <tc_69bfb303, /*tc_3*/
 12327:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
 12328:       [Hex_FWD, Hex_FWD]>,
 12329: 
 12330:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
 12331:       [InstrStage<1, [SLOT3]>], [4, 1],
 12332:       [Hex_FWD, Hex_FWD]>,
 12333: 
 12334:     InstrItinData <tc_6ae3426b, /*tc_3x*/
 12335:       [InstrStage<1, [SLOT3]>], [4, 1],
 12336:       [Hex_FWD, Hex_FWD]>,
 12337: 
 12338:     InstrItinData <tc_6d861a95, /*tc_3x*/
 12339:       [InstrStage<1, [SLOT3]>], [2, 1],
 12340:       [Hex_FWD, Hex_FWD]>,
 12341: 
 12342:     InstrItinData <tc_6e20402a, /*tc_st*/
 12343:       [InstrStage<1, [SLOT0]>], [2, 3],
 12344:       [Hex_FWD, Hex_FWD]>,
 12345: 
 12346:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
 12347:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 12348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12349: 
 12350:     InstrItinData <tc_6fb52018, /*tc_3stall*/
 12351:       [InstrStage<1, [SLOT0]>], [1, 1],
 12352:       [Hex_FWD, Hex_FWD]>,
 12353: 
 12354:     InstrItinData <tc_6fc5dbea, /*tc_1*/
 12355:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 12356:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12357: 
 12358:     InstrItinData <tc_711c805f, /*tc_1*/
 12359:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 12360:       [Hex_FWD, Hex_FWD]>,
 12361: 
 12362:     InstrItinData <tc_713b66bf, /*tc_1*/
 12363:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 12364:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12365: 
 12366:     InstrItinData <tc_7401744f, /*tc_2*/
 12367:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
 12368:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12369: 
 12370:     InstrItinData <tc_7476d766, /*tc_3stall*/
 12371:       [InstrStage<1, [SLOT3]>], [4, 2],
 12372:       [Hex_FWD, Hex_FWD]>,
 12373: 
 12374:     InstrItinData <tc_74a42bda, /*tc_ld*/
 12375:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
 12376:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12377: 
 12378:     InstrItinData <tc_759e57be, /*tc_3stall*/
 12379:       [InstrStage<1, [SLOT2]>], [4, 1],
 12380:       [Hex_FWD, Hex_FWD]>,
 12381: 
 12382:     InstrItinData <tc_76bb5435, /*tc_ld*/
 12383:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
 12384:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12385: 
 12386:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
 12387:       [InstrStage<1, [SLOT2]>], [1],
 12388:       [Hex_FWD]>,
 12389: 
 12390:     InstrItinData <tc_77f94a5e, /*tc_st*/
 12391:       [InstrStage<1, [SLOT0]>], [],
 12392:       []>,
 12393: 
 12394:     InstrItinData <tc_788b1d09, /*tc_3x*/
 12395:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 12396:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12397: 
 12398:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
 12399:       [InstrStage<1, [SLOT0]>], [],
 12400:       []>,
 12401: 
 12402:     InstrItinData <tc_7af3a37e, /*tc_st*/
 12403:       [InstrStage<1, [SLOT0]>], [1, 3],
 12404:       [Hex_FWD, Hex_FWD]>,
 12405: 
 12406:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
 12407:       [InstrStage<1, [SLOT0]>], [3, 2],
 12408:       [Hex_FWD, Hex_FWD]>,
 12409: 
 12410:     InstrItinData <tc_7c28bd7e, /*tc_st*/
 12411:       [InstrStage<1, [SLOT0]>], [3],
 12412:       [Hex_FWD]>,
 12413: 
 12414:     InstrItinData <tc_7c31e19a, /*tc_st*/
 12415:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 12416:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12417: 
 12418:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
 12419:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
 12420:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12421: 
 12422:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
 12423:       [InstrStage<1, [SLOT3]>], [4, 1],
 12424:       [Hex_FWD, Hex_FWD]>,
 12425: 
 12426:     InstrItinData <tc_7f58404a, /*tc_3stall*/
 12427:       [InstrStage<1, [SLOT3]>], [],
 12428:       []>,
 12429: 
 12430:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
 12431:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
 12432:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12433: 
 12434:     InstrItinData <tc_7f8ae742, /*tc_3x*/
 12435:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 12436:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12437: 
 12438:     InstrItinData <tc_8035e91f, /*tc_st*/
 12439:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
 12440:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12441: 
 12442:     InstrItinData <tc_822c3c68, /*tc_ld*/
 12443:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
 12444:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12445: 
 12446:     InstrItinData <tc_829d8a86, /*tc_st*/
 12447:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
 12448:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12449: 
 12450:     InstrItinData <tc_838c4d7a, /*tc_st*/
 12451:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 12452:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12453: 
 12454:     InstrItinData <tc_84a7500d, /*tc_2*/
 12455:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 12456:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12457: 
 12458:     InstrItinData <tc_86173609, /*tc_2latepred*/
 12459:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 12460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12461: 
 12462:     InstrItinData <tc_887d1bb7, /*tc_st*/
 12463:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
 12464:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12465: 
 12466:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
 12467:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
 12468:       [Hex_FWD, Hex_FWD]>,
 12469: 
 12470:     InstrItinData <tc_8a825db2, /*tc_2*/
 12471:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12472:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12473: 
 12474:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
 12475:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 12476:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12477: 
 12478:     InstrItinData <tc_8e82e8ca, /*tc_st*/
 12479:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
 12480:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12481: 
 12482:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
 12483:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
 12484:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12485: 
 12486:     InstrItinData <tc_9124c04f, /*tc_1*/
 12487:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 12488:       [Hex_FWD, Hex_FWD]>,
 12489: 
 12490:     InstrItinData <tc_92240447, /*tc_st*/
 12491:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
 12492:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12493: 
 12494:     InstrItinData <tc_934753bb, /*tc_ld*/
 12495:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
 12496:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12497: 
 12498:     InstrItinData <tc_937dd41c, /*tc_ld*/
 12499:       [InstrStage<1, [SLOT0, SLOT1]>], [],
 12500:       []>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 12501-13000 / 第 12501-13000 行

```tablegen
 12501: 
 12502:     InstrItinData <tc_9406230a, /*tc_3x*/
 12503:       [InstrStage<1, [SLOT3]>], [2, 1],
 12504:       [Hex_FWD, Hex_FWD]>,
 12505: 
 12506:     InstrItinData <tc_95a33176, /*tc_2*/
 12507:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 12508:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12509: 
 12510:     InstrItinData <tc_95f43c5e, /*tc_3*/
 12511:       [InstrStage<1, [SLOT2]>], [1],
 12512:       [Hex_FWD]>,
 12513: 
 12514:     InstrItinData <tc_96ef76ef, /*tc_st*/
 12515:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
 12516:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12517: 
 12518:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
 12519:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 12520:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12521: 
 12522:     InstrItinData <tc_9783714b, /*tc_4x*/
 12523:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
 12524:       [Hex_FWD, Hex_FWD]>,
 12525: 
 12526:     InstrItinData <tc_9b20a062, /*tc_3stall*/
 12527:       [InstrStage<1, [SLOT2]>], [4, 1],
 12528:       [Hex_FWD, Hex_FWD]>,
 12529: 
 12530:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
 12531:       [InstrStage<1, [SLOT2]>], [],
 12532:       []>,
 12533: 
 12534:     InstrItinData <tc_9b3c0462, /*tc_2*/
 12535:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12536:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12537: 
 12538:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
 12539:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 12540:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12541: 
 12542:     InstrItinData <tc_9c52f549, /*tc_1*/
 12543:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 12544:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12545: 
 12546:     InstrItinData <tc_9e27f2f9, /*tc_1*/
 12547:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 12548:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12549: 
 12550:     InstrItinData <tc_9e72dc89, /*tc_4x*/
 12551:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 12552:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12553: 
 12554:     InstrItinData <tc_9edb7c77, /*tc_4x*/
 12555:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
 12556:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12557: 
 12558:     InstrItinData <tc_9edefe01, /*tc_st*/
 12559:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
 12560:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12561: 
 12562:     InstrItinData <tc_9f6cd987, /*tc_1*/
 12563:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 12564:       [Hex_FWD, Hex_FWD]>,
 12565: 
 12566:     InstrItinData <tc_a08b630b, /*tc_2*/
 12567:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12568:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12569: 
 12570:     InstrItinData <tc_a1297125, /*tc_1*/
 12571:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 12572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12573: 
 12574:     InstrItinData <tc_a154b476, /*tc_3x*/
 12575:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
 12576:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12577: 
 12578:     InstrItinData <tc_a2b365d2, /*tc_st*/
 12579:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
 12580:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12581: 
 12582:     InstrItinData <tc_a3070909, /*tc_3stall*/
 12583:       [InstrStage<1, [SLOT0]>], [1, 1],
 12584:       [Hex_FWD, Hex_FWD]>,
 12585: 
 12586:     InstrItinData <tc_a32e03e7, /*tc_ld*/
 12587:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
 12588:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12589: 
 12590:     InstrItinData <tc_a38c45dc, /*tc_3x*/
 12591:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
 12592:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12593: 
 12594:     InstrItinData <tc_a4e22bbd, /*tc_2*/
 12595:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 12596:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12597: 
 12598:     InstrItinData <tc_a4ee89db, /*tc_2early*/
 12599:       [InstrStage<1, [SLOT0]>], [],
 12600:       []>,
 12601: 
 12602:     InstrItinData <tc_a724463d, /*tc_3stall*/
 12603:       [InstrStage<1, [SLOT0]>], [4, 1],
 12604:       [Hex_FWD, Hex_FWD]>,
 12605: 
 12606:     InstrItinData <tc_a7a13fac, /*tc_1*/
 12607:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 12608:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12609: 
 12610:     InstrItinData <tc_a7bdb22c, /*tc_2*/
 12611:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 12612:       [Hex_FWD, Hex_FWD]>,
 12613: 
 12614:     InstrItinData <tc_a9edeffa, /*tc_st*/
 12615:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 12616:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12617: 
 12618:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
 12619:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
 12620:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12621: 
 12622:     InstrItinData <tc_ac65613f, /*tc_ld*/
 12623:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
 12624:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12625: 
 12626:     InstrItinData <tc_addc37a8, /*tc_st*/
 12627:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
 12628:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12629: 
 12630:     InstrItinData <tc_ae5babd7, /*tc_st*/
 12631:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 12632:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12633: 
 12634:     InstrItinData <tc_aee6250c, /*tc_ld*/
 12635:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 12636:       [Hex_FWD, Hex_FWD]>,
 12637: 
 12638:     InstrItinData <tc_af6af259, /*tc_ld*/
 12639:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
 12640:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12641: 
 12642:     InstrItinData <tc_b1ae5f67, /*tc_st*/
 12643:       [InstrStage<1, [SLOT0]>], [1],
 12644:       [Hex_FWD]>,
 12645: 
 12646:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
 12647:       [InstrStage<1, [SLOT3]>], [1, 1],
 12648:       [Hex_FWD, Hex_FWD]>,
 12649: 
 12650:     InstrItinData <tc_b3d46584, /*tc_st*/
 12651:       [InstrStage<1, [SLOT0]>], [],
 12652:       []>,
 12653: 
 12654:     InstrItinData <tc_b4dc7630, /*tc_st*/
 12655:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
 12656:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12657: 
 12658:     InstrItinData <tc_b7c4062a, /*tc_ld*/
 12659:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
 12660:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12661: 
 12662:     InstrItinData <tc_b837298f, /*tc_1*/
 12663:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 12664:       []>,
 12665: 
 12666:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
 12667:       [InstrStage<1, [SLOT2]>], [],
 12668:       []>,
 12669: 
 12670:     InstrItinData <tc_ba9255a6, /*tc_st*/
 12671:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
 12672:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12673: 
 12674:     InstrItinData <tc_bb07f2c5, /*tc_st*/
 12675:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
 12676:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12677: 
 12678:     InstrItinData <tc_bb78483e, /*tc_3stall*/
 12679:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 12680:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12681: 
 12682:     InstrItinData <tc_bb831a7c, /*tc_2*/
 12683:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
 12684:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12685: 
 12686:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
 12687:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
 12688:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12689: 
 12690:     InstrItinData <tc_c20701f0, /*tc_2*/
 12691:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12692:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12693: 
 12694:     InstrItinData <tc_c21d7447, /*tc_3x*/
 12695:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 12696:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12697: 
 12698:     InstrItinData <tc_c57d9f39, /*tc_1*/
 12699:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 12700:       [Hex_FWD, Hex_FWD]>,
 12701: 
 12702:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
 12703:       [InstrStage<1, [SLOT0]>], [],
 12704:       []>,
 12705: 
 12706:     InstrItinData <tc_ce59038e, /*tc_st*/
 12707:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
 12708:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12709: 
 12710:     InstrItinData <tc_cfa0e29b, /*tc_st*/
 12711:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
 12712:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12713: 
 12714:     InstrItinData <tc_d03278fd, /*tc_st*/
 12715:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
 12716:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12717: 
 12718:     InstrItinData <tc_d234b61a, /*tc_st*/
 12719:       [InstrStage<1, [SLOT0]>], [1],
 12720:       [Hex_FWD]>,
 12721: 
 12722:     InstrItinData <tc_d33e5eee, /*tc_1*/
 12723:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 12724:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12725: 
 12726:     InstrItinData <tc_d3632d88, /*tc_2*/
 12727:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 12728:       [Hex_FWD, Hex_FWD]>,
 12729: 
 12730:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
 12731:       [InstrStage<1, [SLOT0]>], [1],
 12732:       [Hex_FWD]>,
 12733: 
 12734:     InstrItinData <tc_d57d649c, /*tc_3stall*/
 12735:       [InstrStage<1, [SLOT2]>], [2],
 12736:       [Hex_FWD]>,
 12737: 
 12738:     InstrItinData <tc_d61dfdc3, /*tc_2*/
 12739:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 12740:       [Hex_FWD, Hex_FWD]>,
 12741: 
 12742:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
 12743:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 12744:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12745: 
 12746:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
 12747:       [InstrStage<1, [SLOT3]>], [2, 1],
 12748:       [Hex_FWD, Hex_FWD]>,
 12749: 
 12750:     InstrItinData <tc_d7718fbe, /*tc_3x*/
 12751:       [InstrStage<1, [SLOT3]>], [1],
 12752:       [Hex_FWD]>,
 12753: 
 12754:     InstrItinData <tc_db596beb, /*tc_3x*/
 12755:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 12756:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12757: 
 12758:     InstrItinData <tc_db96aa6b, /*tc_st*/
 12759:       [InstrStage<1, [SLOT0]>], [1],
 12760:       [Hex_FWD]>,
 12761: 
 12762:     InstrItinData <tc_dc51281d, /*tc_3*/
 12763:       [InstrStage<1, [SLOT2]>], [2, 1],
 12764:       [Hex_FWD, Hex_FWD]>,
 12765: 
 12766:     InstrItinData <tc_decdde8a, /*tc_1*/
 12767:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 12768:       [Hex_FWD]>,
 12769: 
 12770:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
 12771:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
 12772:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12773: 
 12774:     InstrItinData <tc_e3d699e3, /*tc_2*/
 12775:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 12776:       [Hex_FWD, Hex_FWD]>,
 12777: 
 12778:     InstrItinData <tc_e60def48, /*tc_1*/
 12779:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 12780:       [Hex_FWD]>,
 12781: 
 12782:     InstrItinData <tc_e9170fb7, /*tc_ld*/
 12783:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 12784:       [Hex_FWD, Hex_FWD]>,
 12785: 
 12786:     InstrItinData <tc_ed03645c, /*tc_1*/
 12787:       [InstrStage<1, [SLOT2]>], [3, 2],
 12788:       [Hex_FWD, Hex_FWD]>,
 12789: 
 12790:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
 12791:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 12792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12793: 
 12794:     InstrItinData <tc_eed07714, /*tc_ld*/
 12795:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 12796:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12797: 
 12798:     InstrItinData <tc_eeda4109, /*tc_1*/
 12799:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 12800:       [Hex_FWD, Hex_FWD]>,
 12801: 
 12802:     InstrItinData <tc_ef921005, /*tc_1*/
 12803:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 12804:       [Hex_FWD, Hex_FWD]>,
 12805: 
 12806:     InstrItinData <tc_f098b237, /*tc_2*/
 12807:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12808:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12809: 
 12810:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
 12811:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 12812:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12813: 
 12814:     InstrItinData <tc_f0e8e832, /*tc_4x*/
 12815:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 12816:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12817: 
 12818:     InstrItinData <tc_f34c1c21, /*tc_2*/
 12819:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12820:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12821: 
 12822:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
 12823:       [InstrStage<1, [SLOT0]>], [2],
 12824:       [Hex_FWD]>,
 12825: 
 12826:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
 12827:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
 12828:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12829: 
 12830:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
 12831:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
 12832:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12833: 
 12834:     InstrItinData <tc_f7569068, /*tc_4x*/
 12835:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
 12836:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12837: 
 12838:     InstrItinData <tc_f97707c1, /*tc_1*/
 12839:       [InstrStage<1, [SLOT2]>], [2],
 12840:       [Hex_FWD]>,
 12841: 
 12842:     InstrItinData <tc_f999c66e, /*tc_1*/
 12843:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 12844:       [Hex_FWD, Hex_FWD]>,
 12845: 
 12846:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
 12847:       [InstrStage<1, [SLOT3]>], [4, 2],
 12848:       [Hex_FWD, Hex_FWD]>,
 12849: 
 12850:     InstrItinData <tc_fedb7e19, /*tc_ld*/
 12851:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
 12852:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
 12853:   ];
 12854: }
 12855: 
 12856: class DepScalarItinV79 {
 12857:   list<InstrItinData> DepScalarItinV79_list = [
 12858:     InstrItinData <tc_011e0e9d, /*tc_st*/
 12859:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
 12860:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12861: 
 12862:     InstrItinData <tc_01d44cb2, /*tc_2*/
 12863:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12864:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12865: 
 12866:     InstrItinData <tc_01e1be3b, /*tc_3x*/
 12867:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 12868:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12869: 
 12870:     InstrItinData <tc_02fe1c65, /*tc_4x*/
 12871:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 12872:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12873: 
 12874:     InstrItinData <tc_0655b949, /*tc_st*/
 12875:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
 12876:       [Hex_FWD, Hex_FWD]>,
 12877: 
 12878:     InstrItinData <tc_075c8dd8, /*tc_ld*/
 12879:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
 12880:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12881: 
 12882:     InstrItinData <tc_0a195f2c, /*tc_4x*/
 12883:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 12884:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12885: 
 12886:     InstrItinData <tc_0a43be35, /*tc_3x*/
 12887:       [InstrStage<1, [SLOT3]>], [1],
 12888:       [Hex_FWD]>,
 12889: 
 12890:     InstrItinData <tc_0a6c20ae, /*tc_st*/
 12891:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
 12892:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12893: 
 12894:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
 12895:       [InstrStage<1, [SLOT2]>], [1],
 12896:       [Hex_FWD]>,
 12897: 
 12898:     InstrItinData <tc_0dfac0a7, /*tc_2*/
 12899:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12900:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12901: 
 12902:     InstrItinData <tc_0fac1eb8, /*tc_st*/
 12903:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
 12904:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12905: 
 12906:     InstrItinData <tc_112d30d6, /*tc_1*/
 12907:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 12908:       [Hex_FWD]>,
 12909: 
 12910:     InstrItinData <tc_1242dc2a, /*tc_ld*/
 12911:       [InstrStage<1, [SLOT0]>], [2],
 12912:       [Hex_FWD]>,
 12913: 
 12914:     InstrItinData <tc_1248597c, /*tc_3x*/
 12915:       [InstrStage<1, [SLOT3]>], [2, 2],
 12916:       [Hex_FWD, Hex_FWD]>,
 12917: 
 12918:     InstrItinData <tc_139ef484, /*tc_3stall*/
 12919:       [InstrStage<1, [SLOT2]>], [1, 1],
 12920:       [Hex_FWD, Hex_FWD]>,
 12921: 
 12922:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
 12923:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
 12924:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12925: 
 12926:     InstrItinData <tc_151bf368, /*tc_1*/
 12927:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 12928:       [Hex_FWD, Hex_FWD]>,
 12929: 
 12930:     InstrItinData <tc_158aa3f7, /*tc_st*/
 12931:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
 12932:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12933: 
 12934:     InstrItinData <tc_197dce51, /*tc_3x*/
 12935:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
 12936:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12937: 
 12938:     InstrItinData <tc_1981450d, /*tc_newvjump*/
 12939:       [InstrStage<1, [SLOT0]>], [3],
 12940:       [Hex_FWD]>,
 12941: 
 12942:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
 12943:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
 12944:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12945: 
 12946:     InstrItinData <tc_1c7522a8, /*tc_ld*/
 12947:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
 12948:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12949: 
 12950:     InstrItinData <tc_1d41f8b7, /*tc_1*/
 12951:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
 12952:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12953: 
 12954:     InstrItinData <tc_1fcb8495, /*tc_2*/
 12955:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12956:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12957: 
 12958:     InstrItinData <tc_1fe4ab69, /*tc_st*/
 12959:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
 12960:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12961: 
 12962:     InstrItinData <tc_20131976, /*tc_2*/
 12963:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 12964:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12965: 
 12966:     InstrItinData <tc_2237d952, /*tc_ld*/
 12967:       [InstrStage<1, [SLOT0]>], [1, 2],
 12968:       [Hex_FWD, Hex_FWD]>,
 12969: 
 12970:     InstrItinData <tc_23708a21, /*tc_1*/
 12971:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 12972:       []>,
 12973: 
 12974:     InstrItinData <tc_2471c1c8, /*tc_ld*/
 12975:       [InstrStage<1, [SLOT0]>], [4, 1],
 12976:       [Hex_FWD, Hex_FWD]>,
 12977: 
 12978:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
 12979:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 12980:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12981: 
 12982:     InstrItinData <tc_24f426ab, /*tc_1*/
 12983:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 12984:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12985: 
 12986:     InstrItinData <tc_27106296, /*tc_3x*/
 12987:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
 12988:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 12989: 
 12990:     InstrItinData <tc_280f7fe1, /*tc_st*/
 12991:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
 12992:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 12993: 
 12994:     InstrItinData <tc_28e55c6f, /*tc_3x*/
 12995:       [InstrStage<1, [SLOT3]>], [1, 1],
 12996:       [Hex_FWD, Hex_FWD]>,
 12997: 
 12998:     InstrItinData <tc_2c13e7f5, /*tc_2*/
 12999:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 13000:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV79, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV79; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV79 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV79 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 13001-13500 / 第 13001-13500 行

```tablegen
 13001: 
 13002:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
 13003:       [InstrStage<1, [SLOT3]>], [1],
 13004:       [Hex_FWD]>,
 13005: 
 13006:     InstrItinData <tc_2f573607, /*tc_1*/
 13007:       [InstrStage<1, [SLOT2]>], [2, 2],
 13008:       [Hex_FWD, Hex_FWD]>,
 13009: 
 13010:     InstrItinData <tc_33e7e673, /*tc_2early*/
 13011:       [InstrStage<1, [SLOT2]>], [],
 13012:       []>,
 13013: 
 13014:     InstrItinData <tc_362b0be2, /*tc_3*/
 13015:       [InstrStage<1, [SLOT2]>], [1],
 13016:       [Hex_FWD]>,
 13017: 
 13018:     InstrItinData <tc_38382228, /*tc_3x*/
 13019:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
 13020:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13021: 
 13022:     InstrItinData <tc_388f9897, /*tc_1*/
 13023:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 13024:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13025: 
 13026:     InstrItinData <tc_38e0bae9, /*tc_3x*/
 13027:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
 13028:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13029: 
 13030:     InstrItinData <tc_3d14a17b, /*tc_1*/
 13031:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
 13032:       [Hex_FWD, Hex_FWD]>,
 13033: 
 13034:     InstrItinData <tc_3edca78f, /*tc_2*/
 13035:       [InstrStage<1, [SLOT3]>], [4, 2],
 13036:       [Hex_FWD, Hex_FWD]>,
 13037: 
 13038:     InstrItinData <tc_3fbf1042, /*tc_1*/
 13039:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
 13040:       [Hex_FWD]>,
 13041: 
 13042:     InstrItinData <tc_407e96f9, /*tc_1*/
 13043:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13044:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13045: 
 13046:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
 13047:       [InstrStage<1, [SLOT0]>], [3, 1],
 13048:       [Hex_FWD, Hex_FWD]>,
 13049: 
 13050:     InstrItinData <tc_4222e6bf, /*tc_ld*/
 13051:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 13052:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13053: 
 13054:     InstrItinData <tc_42ff66ba, /*tc_1*/
 13055:       [InstrStage<1, [SLOT2]>], [2, 2],
 13056:       [Hex_FWD, Hex_FWD]>,
 13057: 
 13058:     InstrItinData <tc_442395f3, /*tc_2latepred*/
 13059:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
 13060:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13061: 
 13062:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
 13063:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
 13064:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13065: 
 13066:     InstrItinData <tc_44d5a428, /*tc_st*/
 13067:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
 13068:       [Hex_FWD, Hex_FWD]>,
 13069: 
 13070:     InstrItinData <tc_44fffc58, /*tc_3*/
 13071:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 13072:       [Hex_FWD]>,
 13073: 
 13074:     InstrItinData <tc_45791fb8, /*tc_ld*/
 13075:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
 13076:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13077: 
 13078:     InstrItinData <tc_45f9d1be, /*tc_2early*/
 13079:       [InstrStage<1, [SLOT2]>], [2],
 13080:       [Hex_FWD]>,
 13081: 
 13082:     InstrItinData <tc_46c18ecf, /*tc_3x*/
 13083:       [InstrStage<1, [SLOT3]>], [4, 1],
 13084:       [Hex_FWD, Hex_FWD]>,
 13085: 
 13086:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
 13087:       [InstrStage<1, [SLOT3]>], [4, 1],
 13088:       [Hex_FWD, Hex_FWD]>,
 13089: 
 13090:     InstrItinData <tc_4a55d03c, /*tc_1*/
 13091:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13092:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13093: 
 13094:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
 13095:       [InstrStage<1, [SLOT3]>], [2, 2],
 13096:       [Hex_FWD, Hex_FWD]>,
 13097: 
 13098:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
 13099:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 13100:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13101: 
 13102:     InstrItinData <tc_4bf903b0, /*tc_st*/
 13103:       [InstrStage<1, [SLOT0]>], [3],
 13104:       [Hex_FWD]>,
 13105: 
 13106:     InstrItinData <tc_503ce0f3, /*tc_3x*/
 13107:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
 13108:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13109: 
 13110:     InstrItinData <tc_512b1653, /*tc_st*/
 13111:       [InstrStage<1, [SLOT0]>], [1, 2],
 13112:       [Hex_FWD, Hex_FWD]>,
 13113: 
 13114:     InstrItinData <tc_53c851ab, /*tc_3stall*/
 13115:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
 13116:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13117: 
 13118:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
 13119:       [InstrStage<1, [SLOT3]>], [1],
 13120:       [Hex_FWD]>,
 13121: 
 13122:     InstrItinData <tc_5502c366, /*tc_1*/
 13123:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13124:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13125: 
 13126:     InstrItinData <tc_55255f2b, /*tc_3stall*/
 13127:       [InstrStage<1, [SLOT3]>], [],
 13128:       []>,
 13129: 
 13130:     InstrItinData <tc_556f6577, /*tc_3x*/
 13131:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 13132:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13133: 
 13134:     InstrItinData <tc_55a9a350, /*tc_st*/
 13135:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
 13136:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13137: 
 13138:     InstrItinData <tc_55b33fda, /*tc_1*/
 13139:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 13140:       [Hex_FWD, Hex_FWD]>,
 13141: 
 13142:     InstrItinData <tc_56a124a7, /*tc_1*/
 13143:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 13144:       [Hex_FWD, Hex_FWD]>,
 13145: 
 13146:     InstrItinData <tc_57a55b54, /*tc_1*/
 13147:       [InstrStage<1, [SLOT3]>], [2, 2],
 13148:       [Hex_FWD, Hex_FWD]>,
 13149: 
 13150:     InstrItinData <tc_5944960d, /*tc_ld*/
 13151:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
 13152:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13153: 
 13154:     InstrItinData <tc_59a7822c, /*tc_1*/
 13155:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
 13156:       [Hex_FWD, Hex_FWD]>,
 13157: 
 13158:     InstrItinData <tc_5a222e89, /*tc_2early*/
 13159:       [InstrStage<1, [SLOT2]>], [1, 1],
 13160:       [Hex_FWD, Hex_FWD]>,
 13161: 
 13162:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
 13163:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 13164:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13165: 
 13166:     InstrItinData <tc_5b347363, /*tc_1*/
 13167:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
 13168:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13169: 
 13170:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
 13171:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
 13172:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13173: 
 13174:     InstrItinData <tc_5da50c4b, /*tc_1*/
 13175:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13176:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13177: 
 13178:     InstrItinData <tc_5deb5e47, /*tc_st*/
 13179:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 13180:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13181: 
 13182:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
 13183:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 13184:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13185: 
 13186:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
 13187:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
 13188:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13189: 
 13190:     InstrItinData <tc_60e324ff, /*tc_1*/
 13191:       [InstrStage<1, [SLOT2]>], [2],
 13192:       [Hex_FWD]>,
 13193: 
 13194:     InstrItinData <tc_63567288, /*tc_2latepred*/
 13195:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
 13196:       [Hex_FWD]>,
 13197: 
 13198:     InstrItinData <tc_64b00d8a, /*tc_ld*/
 13199:       [InstrStage<1, [SLOT0]>], [4, 1],
 13200:       [Hex_FWD, Hex_FWD]>,
 13201: 
 13202:     InstrItinData <tc_651cbe02, /*tc_1*/
 13203:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13204:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13205: 
 13206:     InstrItinData <tc_65279839, /*tc_2*/
 13207:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 13208:       [Hex_FWD, Hex_FWD]>,
 13209: 
 13210:     InstrItinData <tc_65cbd974, /*tc_st*/
 13211:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
 13212:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13213: 
 13214:     InstrItinData <tc_69bfb303, /*tc_3*/
 13215:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
 13216:       [Hex_FWD, Hex_FWD]>,
 13217: 
 13218:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
 13219:       [InstrStage<1, [SLOT3]>], [4, 1],
 13220:       [Hex_FWD, Hex_FWD]>,
 13221: 
 13222:     InstrItinData <tc_6ae3426b, /*tc_3x*/
 13223:       [InstrStage<1, [SLOT3]>], [4, 1],
 13224:       [Hex_FWD, Hex_FWD]>,
 13225: 
 13226:     InstrItinData <tc_6d861a95, /*tc_3x*/
 13227:       [InstrStage<1, [SLOT3]>], [2, 1],
 13228:       [Hex_FWD, Hex_FWD]>,
 13229: 
 13230:     InstrItinData <tc_6e20402a, /*tc_st*/
 13231:       [InstrStage<1, [SLOT0]>], [2, 3],
 13232:       [Hex_FWD, Hex_FWD]>,
 13233: 
 13234:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
 13235:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 13236:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13237: 
 13238:     InstrItinData <tc_6fb52018, /*tc_3stall*/
 13239:       [InstrStage<1, [SLOT0]>], [1, 1],
 13240:       [Hex_FWD, Hex_FWD]>,
 13241: 
 13242:     InstrItinData <tc_6fc5dbea, /*tc_1*/
 13243:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 13244:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13245: 
 13246:     InstrItinData <tc_711c805f, /*tc_1*/
 13247:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 13248:       [Hex_FWD, Hex_FWD]>,
 13249: 
 13250:     InstrItinData <tc_713b66bf, /*tc_1*/
 13251:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 13252:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13253: 
 13254:     InstrItinData <tc_7401744f, /*tc_2*/
 13255:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
 13256:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13257: 
 13258:     InstrItinData <tc_7476d766, /*tc_3stall*/
 13259:       [InstrStage<1, [SLOT3]>], [4, 2],
 13260:       [Hex_FWD, Hex_FWD]>,
 13261: 
 13262:     InstrItinData <tc_74a42bda, /*tc_ld*/
 13263:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
 13264:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13265: 
 13266:     InstrItinData <tc_759e57be, /*tc_3stall*/
 13267:       [InstrStage<1, [SLOT2]>], [4, 1],
 13268:       [Hex_FWD, Hex_FWD]>,
 13269: 
 13270:     InstrItinData <tc_76bb5435, /*tc_ld*/
 13271:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
 13272:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13273: 
 13274:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
 13275:       [InstrStage<1, [SLOT2]>], [1],
 13276:       [Hex_FWD]>,
 13277: 
 13278:     InstrItinData <tc_77f94a5e, /*tc_st*/
 13279:       [InstrStage<1, [SLOT0]>], [],
 13280:       []>,
 13281: 
 13282:     InstrItinData <tc_788b1d09, /*tc_3x*/
 13283:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 13284:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13285: 
 13286:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
 13287:       [InstrStage<1, [SLOT0]>], [],
 13288:       []>,
 13289: 
 13290:     InstrItinData <tc_7af3a37e, /*tc_st*/
 13291:       [InstrStage<1, [SLOT0]>], [1, 3],
 13292:       [Hex_FWD, Hex_FWD]>,
 13293: 
 13294:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
 13295:       [InstrStage<1, [SLOT0]>], [3, 2],
 13296:       [Hex_FWD, Hex_FWD]>,
 13297: 
 13298:     InstrItinData <tc_7c28bd7e, /*tc_st*/
 13299:       [InstrStage<1, [SLOT0]>], [3],
 13300:       [Hex_FWD]>,
 13301: 
 13302:     InstrItinData <tc_7c31e19a, /*tc_st*/
 13303:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 13304:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13305: 
 13306:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
 13307:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
 13308:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13309: 
 13310:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
 13311:       [InstrStage<1, [SLOT3]>], [4, 1],
 13312:       [Hex_FWD, Hex_FWD]>,
 13313: 
 13314:     InstrItinData <tc_7f58404a, /*tc_3stall*/
 13315:       [InstrStage<1, [SLOT3]>], [],
 13316:       []>,
 13317: 
 13318:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
 13319:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
 13320:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13321: 
 13322:     InstrItinData <tc_7f8ae742, /*tc_3x*/
 13323:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 13324:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13325: 
 13326:     InstrItinData <tc_8035e91f, /*tc_st*/
 13327:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
 13328:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13329: 
 13330:     InstrItinData <tc_822c3c68, /*tc_ld*/
 13331:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
 13332:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13333: 
 13334:     InstrItinData <tc_829d8a86, /*tc_st*/
 13335:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
 13336:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13337: 
 13338:     InstrItinData <tc_838c4d7a, /*tc_st*/
 13339:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 13340:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13341: 
 13342:     InstrItinData <tc_84a7500d, /*tc_2*/
 13343:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 13344:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13345: 
 13346:     InstrItinData <tc_86173609, /*tc_2latepred*/
 13347:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 13348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13349: 
 13350:     InstrItinData <tc_887d1bb7, /*tc_st*/
 13351:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
 13352:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13353: 
 13354:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
 13355:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
 13356:       [Hex_FWD, Hex_FWD]>,
 13357: 
 13358:     InstrItinData <tc_8a825db2, /*tc_2*/
 13359:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13360:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13361: 
 13362:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
 13363:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 13364:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13365: 
 13366:     InstrItinData <tc_8e82e8ca, /*tc_st*/
 13367:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
 13368:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13369: 
 13370:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
 13371:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
 13372:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13373: 
 13374:     InstrItinData <tc_9124c04f, /*tc_1*/
 13375:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 13376:       [Hex_FWD, Hex_FWD]>,
 13377: 
 13378:     InstrItinData <tc_92240447, /*tc_st*/
 13379:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
 13380:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13381: 
 13382:     InstrItinData <tc_934753bb, /*tc_ld*/
 13383:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
 13384:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13385: 
 13386:     InstrItinData <tc_937dd41c, /*tc_ld*/
 13387:       [InstrStage<1, [SLOT0, SLOT1]>], [],
 13388:       []>,
 13389: 
 13390:     InstrItinData <tc_9406230a, /*tc_3x*/
 13391:       [InstrStage<1, [SLOT3]>], [2, 1],
 13392:       [Hex_FWD, Hex_FWD]>,
 13393: 
 13394:     InstrItinData <tc_95a33176, /*tc_2*/
 13395:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 13396:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13397: 
 13398:     InstrItinData <tc_95f43c5e, /*tc_3*/
 13399:       [InstrStage<1, [SLOT2]>], [1],
 13400:       [Hex_FWD]>,
 13401: 
 13402:     InstrItinData <tc_96ef76ef, /*tc_st*/
 13403:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
 13404:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13405: 
 13406:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
 13407:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 13408:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13409: 
 13410:     InstrItinData <tc_9783714b, /*tc_4x*/
 13411:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
 13412:       [Hex_FWD, Hex_FWD]>,
 13413: 
 13414:     InstrItinData <tc_9b20a062, /*tc_3stall*/
 13415:       [InstrStage<1, [SLOT2]>], [4, 1],
 13416:       [Hex_FWD, Hex_FWD]>,
 13417: 
 13418:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
 13419:       [InstrStage<1, [SLOT2]>], [],
 13420:       []>,
 13421: 
 13422:     InstrItinData <tc_9b3c0462, /*tc_2*/
 13423:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13424:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13425: 
 13426:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
 13427:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 13428:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13429: 
 13430:     InstrItinData <tc_9c52f549, /*tc_1*/
 13431:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 13432:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13433: 
 13434:     InstrItinData <tc_9e27f2f9, /*tc_1*/
 13435:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 13436:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13437: 
 13438:     InstrItinData <tc_9e72dc89, /*tc_4x*/
 13439:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 13440:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13441: 
 13442:     InstrItinData <tc_9edb7c77, /*tc_4x*/
 13443:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
 13444:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13445: 
 13446:     InstrItinData <tc_9edefe01, /*tc_st*/
 13447:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
 13448:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13449: 
 13450:     InstrItinData <tc_9f6cd987, /*tc_1*/
 13451:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 13452:       [Hex_FWD, Hex_FWD]>,
 13453: 
 13454:     InstrItinData <tc_a08b630b, /*tc_2*/
 13455:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13456:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13457: 
 13458:     InstrItinData <tc_a1297125, /*tc_1*/
 13459:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13460:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13461: 
 13462:     InstrItinData <tc_a154b476, /*tc_3x*/
 13463:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
 13464:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13465: 
 13466:     InstrItinData <tc_a2b365d2, /*tc_st*/
 13467:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
 13468:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13469: 
 13470:     InstrItinData <tc_a3070909, /*tc_3stall*/
 13471:       [InstrStage<1, [SLOT0]>], [1, 1],
 13472:       [Hex_FWD, Hex_FWD]>,
 13473: 
 13474:     InstrItinData <tc_a32e03e7, /*tc_ld*/
 13475:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
 13476:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13477: 
 13478:     InstrItinData <tc_a38c45dc, /*tc_3x*/
 13479:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
 13480:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13481: 
 13482:     InstrItinData <tc_a4e22bbd, /*tc_2*/
 13483:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 13484:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13485: 
 13486:     InstrItinData <tc_a4ee89db, /*tc_2early*/
 13487:       [InstrStage<1, [SLOT0]>], [],
 13488:       []>,
 13489: 
 13490:     InstrItinData <tc_a724463d, /*tc_3stall*/
 13491:       [InstrStage<1, [SLOT0]>], [4, 1],
 13492:       [Hex_FWD, Hex_FWD]>,
 13493: 
 13494:     InstrItinData <tc_a7a13fac, /*tc_1*/
 13495:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 13496:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13497: 
 13498:     InstrItinData <tc_a7bdb22c, /*tc_2*/
 13499:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 13500:       [Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 13501-14000 / 第 13501-14000 行

```tablegen
 13501: 
 13502:     InstrItinData <tc_a9edeffa, /*tc_st*/
 13503:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 13504:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13505: 
 13506:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
 13507:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
 13508:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13509: 
 13510:     InstrItinData <tc_ac65613f, /*tc_ld*/
 13511:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
 13512:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13513: 
 13514:     InstrItinData <tc_addc37a8, /*tc_st*/
 13515:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
 13516:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13517: 
 13518:     InstrItinData <tc_ae5babd7, /*tc_st*/
 13519:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 13520:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13521: 
 13522:     InstrItinData <tc_aee6250c, /*tc_ld*/
 13523:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 13524:       [Hex_FWD, Hex_FWD]>,
 13525: 
 13526:     InstrItinData <tc_af6af259, /*tc_ld*/
 13527:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
 13528:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13529: 
 13530:     InstrItinData <tc_b1ae5f67, /*tc_st*/
 13531:       [InstrStage<1, [SLOT0]>], [1],
 13532:       [Hex_FWD]>,
 13533: 
 13534:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
 13535:       [InstrStage<1, [SLOT3]>], [1, 1],
 13536:       [Hex_FWD, Hex_FWD]>,
 13537: 
 13538:     InstrItinData <tc_b3d46584, /*tc_st*/
 13539:       [InstrStage<1, [SLOT0]>], [],
 13540:       []>,
 13541: 
 13542:     InstrItinData <tc_b4dc7630, /*tc_st*/
 13543:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
 13544:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13545: 
 13546:     InstrItinData <tc_b7c4062a, /*tc_ld*/
 13547:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
 13548:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13549: 
 13550:     InstrItinData <tc_b837298f, /*tc_1*/
 13551:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 13552:       []>,
 13553: 
 13554:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
 13555:       [InstrStage<1, [SLOT2]>], [],
 13556:       []>,
 13557: 
 13558:     InstrItinData <tc_ba9255a6, /*tc_st*/
 13559:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
 13560:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13561: 
 13562:     InstrItinData <tc_bb07f2c5, /*tc_st*/
 13563:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
 13564:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13565: 
 13566:     InstrItinData <tc_bb78483e, /*tc_3stall*/
 13567:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 13568:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13569: 
 13570:     InstrItinData <tc_bb831a7c, /*tc_2*/
 13571:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
 13572:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13573: 
 13574:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
 13575:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
 13576:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13577: 
 13578:     InstrItinData <tc_c20701f0, /*tc_2*/
 13579:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13580:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13581: 
 13582:     InstrItinData <tc_c21d7447, /*tc_3x*/
 13583:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 13584:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13585: 
 13586:     InstrItinData <tc_c57d9f39, /*tc_1*/
 13587:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 13588:       [Hex_FWD, Hex_FWD]>,
 13589: 
 13590:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
 13591:       [InstrStage<1, [SLOT0]>], [],
 13592:       []>,
 13593: 
 13594:     InstrItinData <tc_ce59038e, /*tc_st*/
 13595:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
 13596:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13597: 
 13598:     InstrItinData <tc_cfa0e29b, /*tc_st*/
 13599:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
 13600:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13601: 
 13602:     InstrItinData <tc_d03278fd, /*tc_st*/
 13603:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
 13604:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13605: 
 13606:     InstrItinData <tc_d234b61a, /*tc_st*/
 13607:       [InstrStage<1, [SLOT0]>], [1],
 13608:       [Hex_FWD]>,
 13609: 
 13610:     InstrItinData <tc_d33e5eee, /*tc_1*/
 13611:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 13612:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13613: 
 13614:     InstrItinData <tc_d3632d88, /*tc_2*/
 13615:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 13616:       [Hex_FWD, Hex_FWD]>,
 13617: 
 13618:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
 13619:       [InstrStage<1, [SLOT0]>], [1],
 13620:       [Hex_FWD]>,
 13621: 
 13622:     InstrItinData <tc_d57d649c, /*tc_3stall*/
 13623:       [InstrStage<1, [SLOT2]>], [2],
 13624:       [Hex_FWD]>,
 13625: 
 13626:     InstrItinData <tc_d61dfdc3, /*tc_2*/
 13627:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 13628:       [Hex_FWD, Hex_FWD]>,
 13629: 
 13630:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
 13631:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 13632:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13633: 
 13634:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
 13635:       [InstrStage<1, [SLOT3]>], [2, 1],
 13636:       [Hex_FWD, Hex_FWD]>,
 13637: 
 13638:     InstrItinData <tc_d7718fbe, /*tc_3x*/
 13639:       [InstrStage<1, [SLOT3]>], [1],
 13640:       [Hex_FWD]>,
 13641: 
 13642:     InstrItinData <tc_db596beb, /*tc_3x*/
 13643:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 13644:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13645: 
 13646:     InstrItinData <tc_db96aa6b, /*tc_st*/
 13647:       [InstrStage<1, [SLOT0]>], [1],
 13648:       [Hex_FWD]>,
 13649: 
 13650:     InstrItinData <tc_dc51281d, /*tc_3*/
 13651:       [InstrStage<1, [SLOT2]>], [2, 1],
 13652:       [Hex_FWD, Hex_FWD]>,
 13653: 
 13654:     InstrItinData <tc_decdde8a, /*tc_1*/
 13655:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 13656:       [Hex_FWD]>,
 13657: 
 13658:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
 13659:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
 13660:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13661: 
 13662:     InstrItinData <tc_e3d699e3, /*tc_2*/
 13663:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 13664:       [Hex_FWD, Hex_FWD]>,
 13665: 
 13666:     InstrItinData <tc_e60def48, /*tc_1*/
 13667:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 13668:       [Hex_FWD]>,
 13669: 
 13670:     InstrItinData <tc_e9170fb7, /*tc_ld*/
 13671:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 13672:       [Hex_FWD, Hex_FWD]>,
 13673: 
 13674:     InstrItinData <tc_ed03645c, /*tc_1*/
 13675:       [InstrStage<1, [SLOT2]>], [3, 2],
 13676:       [Hex_FWD, Hex_FWD]>,
 13677: 
 13678:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
 13679:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 13680:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13681: 
 13682:     InstrItinData <tc_eed07714, /*tc_ld*/
 13683:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 13684:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13685: 
 13686:     InstrItinData <tc_eeda4109, /*tc_1*/
 13687:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 13688:       [Hex_FWD, Hex_FWD]>,
 13689: 
 13690:     InstrItinData <tc_ef921005, /*tc_1*/
 13691:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 13692:       [Hex_FWD, Hex_FWD]>,
 13693: 
 13694:     InstrItinData <tc_f098b237, /*tc_2*/
 13695:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13696:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13697: 
 13698:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
 13699:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 13700:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13701: 
 13702:     InstrItinData <tc_f0e8e832, /*tc_4x*/
 13703:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 13704:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13705: 
 13706:     InstrItinData <tc_f34c1c21, /*tc_2*/
 13707:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13708:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13709: 
 13710:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
 13711:       [InstrStage<1, [SLOT0]>], [2],
 13712:       [Hex_FWD]>,
 13713: 
 13714:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
 13715:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
 13716:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13717: 
 13718:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
 13719:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
 13720:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13721: 
 13722:     InstrItinData <tc_f7569068, /*tc_4x*/
 13723:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
 13724:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13725: 
 13726:     InstrItinData <tc_f97707c1, /*tc_1*/
 13727:       [InstrStage<1, [SLOT2]>], [2],
 13728:       [Hex_FWD]>,
 13729: 
 13730:     InstrItinData <tc_f999c66e, /*tc_1*/
 13731:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 13732:       [Hex_FWD, Hex_FWD]>,
 13733: 
 13734:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
 13735:       [InstrStage<1, [SLOT3]>], [4, 2],
 13736:       [Hex_FWD, Hex_FWD]>,
 13737: 
 13738:     InstrItinData <tc_fedb7e19, /*tc_ld*/
 13739:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
 13740:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
 13741:   ];
 13742: }
 13743: 
 13744: class DepScalarItinV81 {
 13745:   list<InstrItinData> DepScalarItinV81_list = [
 13746:     InstrItinData <tc_011e0e9d, /*tc_st*/
 13747:       [InstrStage<1, [SLOT0]>], [2, 1, 2, 3],
 13748:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13749: 
 13750:     InstrItinData <tc_01d44cb2, /*tc_2*/
 13751:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13752:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13753: 
 13754:     InstrItinData <tc_01e1be3b, /*tc_3x*/
 13755:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 13756:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13757: 
 13758:     InstrItinData <tc_02fe1c65, /*tc_4x*/
 13759:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 13760:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13761: 
 13762:     InstrItinData <tc_0655b949, /*tc_st*/
 13763:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 3],
 13764:       [Hex_FWD, Hex_FWD]>,
 13765: 
 13766:     InstrItinData <tc_075c8dd8, /*tc_ld*/
 13767:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2],
 13768:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13769: 
 13770:     InstrItinData <tc_0a195f2c, /*tc_4x*/
 13771:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 13772:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13773: 
 13774:     InstrItinData <tc_0a43be35, /*tc_3x*/
 13775:       [InstrStage<1, [SLOT3]>], [1],
 13776:       [Hex_FWD]>,
 13777: 
 13778:     InstrItinData <tc_0a6c20ae, /*tc_st*/
 13779:       [InstrStage<1, [SLOT0]>], [2, 1, 1, 2, 3],
 13780:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13781: 
 13782:     InstrItinData <tc_0ba0d5da, /*tc_3stall*/
 13783:       [InstrStage<1, [SLOT2]>], [1],
 13784:       [Hex_FWD]>,
 13785: 
 13786:     InstrItinData <tc_0dfac0a7, /*tc_2*/
 13787:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13788:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13789: 
 13790:     InstrItinData <tc_0fac1eb8, /*tc_st*/
 13791:       [InstrStage<1, [SLOT0]>], [3, 2, 3],
 13792:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13793: 
 13794:     InstrItinData <tc_112d30d6, /*tc_1*/
 13795:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 13796:       [Hex_FWD]>,
 13797: 
 13798:     InstrItinData <tc_1242dc2a, /*tc_ld*/
 13799:       [InstrStage<1, [SLOT0]>], [2],
 13800:       [Hex_FWD]>,
 13801: 
 13802:     InstrItinData <tc_1248597c, /*tc_3x*/
 13803:       [InstrStage<1, [SLOT3]>], [2, 2],
 13804:       [Hex_FWD, Hex_FWD]>,
 13805: 
 13806:     InstrItinData <tc_139ef484, /*tc_3stall*/
 13807:       [InstrStage<1, [SLOT2]>], [1, 1],
 13808:       [Hex_FWD, Hex_FWD]>,
 13809: 
 13810:     InstrItinData <tc_14ab4f41, /*tc_newvjump*/
 13811:       [InstrStage<1, [SLOT0]>], [3, 3, 1],
 13812:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13813: 
 13814:     InstrItinData <tc_151bf368, /*tc_1*/
 13815:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 13816:       [Hex_FWD, Hex_FWD]>,
 13817: 
 13818:     InstrItinData <tc_158aa3f7, /*tc_st*/
 13819:       [InstrStage<1, [SLOT0]>], [1, 2, 2],
 13820:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13821: 
 13822:     InstrItinData <tc_197dce51, /*tc_3x*/
 13823:       [InstrStage<1, [SLOT3]>], [4, 2, 1, 1],
 13824:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13825: 
 13826:     InstrItinData <tc_1981450d, /*tc_newvjump*/
 13827:       [InstrStage<1, [SLOT0]>], [3],
 13828:       [Hex_FWD]>,
 13829: 
 13830:     InstrItinData <tc_1c2c7a4a, /*tc_1*/
 13831:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2, 2],
 13832:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13833: 
 13834:     InstrItinData <tc_1c7522a8, /*tc_ld*/
 13835:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2],
 13836:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13837: 
 13838:     InstrItinData <tc_1d41f8b7, /*tc_1*/
 13839:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 4, 2, 2, 2],
 13840:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13841: 
 13842:     InstrItinData <tc_1fcb8495, /*tc_2*/
 13843:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13844:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13845: 
 13846:     InstrItinData <tc_1fe4ab69, /*tc_st*/
 13847:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 1, 2, 3],
 13848:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13849: 
 13850:     InstrItinData <tc_20131976, /*tc_2*/
 13851:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 13852:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13853: 
 13854:     InstrItinData <tc_2237d952, /*tc_ld*/
 13855:       [InstrStage<1, [SLOT0]>], [1, 2],
 13856:       [Hex_FWD, Hex_FWD]>,
 13857: 
 13858:     InstrItinData <tc_23708a21, /*tc_1*/
 13859:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 13860:       []>,
 13861: 
 13862:     InstrItinData <tc_2471c1c8, /*tc_ld*/
 13863:       [InstrStage<1, [SLOT0]>], [4, 1],
 13864:       [Hex_FWD, Hex_FWD]>,
 13865: 
 13866:     InstrItinData <tc_24e109c7, /*tc_newvjump*/
 13867:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 13868:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13869: 
 13870:     InstrItinData <tc_24f426ab, /*tc_1*/
 13871:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 13872:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13873: 
 13874:     InstrItinData <tc_27106296, /*tc_3x*/
 13875:       [InstrStage<1, [SLOT3]>], [4, 1, 2],
 13876:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13877: 
 13878:     InstrItinData <tc_280f7fe1, /*tc_st*/
 13879:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2, 3],
 13880:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13881: 
 13882:     InstrItinData <tc_28e55c6f, /*tc_3x*/
 13883:       [InstrStage<1, [SLOT3]>], [1, 1],
 13884:       [Hex_FWD, Hex_FWD]>,
 13885: 
 13886:     InstrItinData <tc_2c13e7f5, /*tc_2*/
 13887:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 13888:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13889: 
 13890:     InstrItinData <tc_2c3e17fc, /*tc_3x*/
 13891:       [InstrStage<1, [SLOT3]>], [1],
 13892:       [Hex_FWD]>,
 13893: 
 13894:     InstrItinData <tc_2f573607, /*tc_1*/
 13895:       [InstrStage<1, [SLOT2]>], [2, 2],
 13896:       [Hex_FWD, Hex_FWD]>,
 13897: 
 13898:     InstrItinData <tc_33e7e673, /*tc_2early*/
 13899:       [InstrStage<1, [SLOT2]>], [],
 13900:       []>,
 13901: 
 13902:     InstrItinData <tc_362b0be2, /*tc_3*/
 13903:       [InstrStage<1, [SLOT2]>], [1],
 13904:       [Hex_FWD]>,
 13905: 
 13906:     InstrItinData <tc_38382228, /*tc_3x*/
 13907:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 2],
 13908:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13909: 
 13910:     InstrItinData <tc_388f9897, /*tc_1*/
 13911:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 13912:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13913: 
 13914:     InstrItinData <tc_38e0bae9, /*tc_3x*/
 13915:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 1, 1],
 13916:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13917: 
 13918:     InstrItinData <tc_3d14a17b, /*tc_1*/
 13919:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2],
 13920:       [Hex_FWD, Hex_FWD]>,
 13921: 
 13922:     InstrItinData <tc_3edca78f, /*tc_2*/
 13923:       [InstrStage<1, [SLOT3]>], [4, 2],
 13924:       [Hex_FWD, Hex_FWD]>,
 13925: 
 13926:     InstrItinData <tc_3fbf1042, /*tc_1*/
 13927:       [InstrStage<1, [SLOT0, SLOT1]>], [3],
 13928:       [Hex_FWD]>,
 13929: 
 13930:     InstrItinData <tc_407e96f9, /*tc_1*/
 13931:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13932:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13933: 
 13934:     InstrItinData <tc_40d64c94, /*tc_newvjump*/
 13935:       [InstrStage<1, [SLOT0]>], [3, 1],
 13936:       [Hex_FWD, Hex_FWD]>,
 13937: 
 13938:     InstrItinData <tc_4222e6bf, /*tc_ld*/
 13939:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 13940:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13941: 
 13942:     InstrItinData <tc_42ff66ba, /*tc_1*/
 13943:       [InstrStage<1, [SLOT2]>], [2, 2],
 13944:       [Hex_FWD, Hex_FWD]>,
 13945: 
 13946:     InstrItinData <tc_442395f3, /*tc_2latepred*/
 13947:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2, 2],
 13948:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13949: 
 13950:     InstrItinData <tc_449acf79, /*tc_latepredstaia*/
 13951:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 1],
 13952:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13953: 
 13954:     InstrItinData <tc_44d5a428, /*tc_st*/
 13955:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2],
 13956:       [Hex_FWD, Hex_FWD]>,
 13957: 
 13958:     InstrItinData <tc_44fffc58, /*tc_3*/
 13959:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 13960:       [Hex_FWD]>,
 13961: 
 13962:     InstrItinData <tc_45791fb8, /*tc_ld*/
 13963:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 1, 2],
 13964:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13965: 
 13966:     InstrItinData <tc_45f9d1be, /*tc_2early*/
 13967:       [InstrStage<1, [SLOT2]>], [2],
 13968:       [Hex_FWD]>,
 13969: 
 13970:     InstrItinData <tc_46c18ecf, /*tc_3x*/
 13971:       [InstrStage<1, [SLOT3]>], [4, 1],
 13972:       [Hex_FWD, Hex_FWD]>,
 13973: 
 13974:     InstrItinData <tc_49fdfd4b, /*tc_3stall*/
 13975:       [InstrStage<1, [SLOT3]>], [4, 1],
 13976:       [Hex_FWD, Hex_FWD]>,
 13977: 
 13978:     InstrItinData <tc_4a55d03c, /*tc_1*/
 13979:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 13980:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13981: 
 13982:     InstrItinData <tc_4abdbdc6, /*tc_3x*/
 13983:       [InstrStage<1, [SLOT3]>], [2, 2],
 13984:       [Hex_FWD, Hex_FWD]>,
 13985: 
 13986:     InstrItinData <tc_4ac61d92, /*tc_2latepred*/
 13987:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 13988:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 13989: 
 13990:     InstrItinData <tc_4bf903b0, /*tc_st*/
 13991:       [InstrStage<1, [SLOT0]>], [3],
 13992:       [Hex_FWD]>,
 13993: 
 13994:     InstrItinData <tc_503ce0f3, /*tc_3x*/
 13995:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 1],
 13996:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 13997: 
 13998:     InstrItinData <tc_512b1653, /*tc_st*/
 13999:       [InstrStage<1, [SLOT0]>], [1, 2],
 14000:       [Hex_FWD, Hex_FWD]>,
```
- EN: It declares types such as DepScalarItinV81, which carry the state or API of this component. It defines generated/declarative TableGen records like DepScalarItinV81; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 DepScalarItinV81 等类型，用来承载该组件的状态或接口。 这里定义了 DepScalarItinV81 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 14001-14500 / 第 14001-14500 行

```tablegen
 14001: 
 14002:     InstrItinData <tc_53c851ab, /*tc_3stall*/
 14003:       [InstrStage<1, [SLOT2]>], [4, 1, 2],
 14004:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14005: 
 14006:     InstrItinData <tc_54f0cee2, /*tc_3stall*/
 14007:       [InstrStage<1, [SLOT3]>], [1],
 14008:       [Hex_FWD]>,
 14009: 
 14010:     InstrItinData <tc_5502c366, /*tc_1*/
 14011:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 14012:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14013: 
 14014:     InstrItinData <tc_55255f2b, /*tc_3stall*/
 14015:       [InstrStage<1, [SLOT3]>], [],
 14016:       []>,
 14017: 
 14018:     InstrItinData <tc_556f6577, /*tc_3x*/
 14019:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 14020:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14021: 
 14022:     InstrItinData <tc_55a9a350, /*tc_st*/
 14023:       [InstrStage<1, [SLOT0]>], [1, 2, 2, 3],
 14024:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14025: 
 14026:     InstrItinData <tc_55b33fda, /*tc_1*/
 14027:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 14028:       [Hex_FWD, Hex_FWD]>,
 14029: 
 14030:     InstrItinData <tc_56a124a7, /*tc_1*/
 14031:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 14032:       [Hex_FWD, Hex_FWD]>,
 14033: 
 14034:     InstrItinData <tc_57a55b54, /*tc_1*/
 14035:       [InstrStage<1, [SLOT3]>], [2, 2],
 14036:       [Hex_FWD, Hex_FWD]>,
 14037: 
 14038:     InstrItinData <tc_5944960d, /*tc_ld*/
 14039:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 1, 2],
 14040:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14041: 
 14042:     InstrItinData <tc_59a7822c, /*tc_1*/
 14043:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2],
 14044:       [Hex_FWD, Hex_FWD]>,
 14045: 
 14046:     InstrItinData <tc_5a222e89, /*tc_2early*/
 14047:       [InstrStage<1, [SLOT2]>], [1, 1],
 14048:       [Hex_FWD, Hex_FWD]>,
 14049: 
 14050:     InstrItinData <tc_5a4b5e58, /*tc_3x*/
 14051:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 14052:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14053: 
 14054:     InstrItinData <tc_5b347363, /*tc_1*/
 14055:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 2],
 14056:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14057: 
 14058:     InstrItinData <tc_5ceb2f9e, /*tc_ld*/
 14059:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 2, 2],
 14060:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14061: 
 14062:     InstrItinData <tc_5da50c4b, /*tc_1*/
 14063:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 14064:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14065: 
 14066:     InstrItinData <tc_5deb5e47, /*tc_st*/
 14067:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 14068:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14069: 
 14070:     InstrItinData <tc_5e4cf0e8, /*tc_2*/
 14071:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 14072:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14073: 
 14074:     InstrItinData <tc_5f2afaf7, /*tc_latepredldaia*/
 14075:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 4, 3, 1, 2],
 14076:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14077: 
 14078:     InstrItinData <tc_60e324ff, /*tc_1*/
 14079:       [InstrStage<1, [SLOT2]>], [2],
 14080:       [Hex_FWD]>,
 14081: 
 14082:     InstrItinData <tc_63567288, /*tc_2latepred*/
 14083:       [InstrStage<1, [SLOT0, SLOT1]>], [4],
 14084:       [Hex_FWD]>,
 14085: 
 14086:     InstrItinData <tc_64b00d8a, /*tc_ld*/
 14087:       [InstrStage<1, [SLOT0]>], [4, 1],
 14088:       [Hex_FWD, Hex_FWD]>,
 14089: 
 14090:     InstrItinData <tc_651cbe02, /*tc_1*/
 14091:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 14092:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14093: 
 14094:     InstrItinData <tc_65279839, /*tc_2*/
 14095:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 14096:       [Hex_FWD, Hex_FWD]>,
 14097: 
 14098:     InstrItinData <tc_65cbd974, /*tc_st*/
 14099:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2],
 14100:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14101: 
 14102:     InstrItinData <tc_69bfb303, /*tc_3*/
 14103:       [InstrStage<1, [SLOT2, SLOT3]>], [2, 2],
 14104:       [Hex_FWD, Hex_FWD]>,
 14105: 
 14106:     InstrItinData <tc_6aa823ab, /*tc_3stall*/
 14107:       [InstrStage<1, [SLOT3]>], [4, 1],
 14108:       [Hex_FWD, Hex_FWD]>,
 14109: 
 14110:     InstrItinData <tc_6ae3426b, /*tc_3x*/
 14111:       [InstrStage<1, [SLOT3]>], [4, 1],
 14112:       [Hex_FWD, Hex_FWD]>,
 14113: 
 14114:     InstrItinData <tc_6d861a95, /*tc_3x*/
 14115:       [InstrStage<1, [SLOT3]>], [2, 1],
 14116:       [Hex_FWD, Hex_FWD]>,
 14117: 
 14118:     InstrItinData <tc_6e20402a, /*tc_st*/
 14119:       [InstrStage<1, [SLOT0]>], [2, 3],
 14120:       [Hex_FWD, Hex_FWD]>,
 14121: 
 14122:     InstrItinData <tc_6f42bc60, /*tc_3stall*/
 14123:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 14124:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14125: 
 14126:     InstrItinData <tc_6fb52018, /*tc_3stall*/
 14127:       [InstrStage<1, [SLOT0]>], [1, 1],
 14128:       [Hex_FWD, Hex_FWD]>,
 14129: 
 14130:     InstrItinData <tc_6fc5dbea, /*tc_1*/
 14131:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 14132:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14133: 
 14134:     InstrItinData <tc_711c805f, /*tc_1*/
 14135:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 14136:       [Hex_FWD, Hex_FWD]>,
 14137: 
 14138:     InstrItinData <tc_713b66bf, /*tc_1*/
 14139:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 14140:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14141: 
 14142:     InstrItinData <tc_7401744f, /*tc_2*/
 14143:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 4, 2, 2],
 14144:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14145: 
 14146:     InstrItinData <tc_7476d766, /*tc_3stall*/
 14147:       [InstrStage<1, [SLOT3]>], [4, 2],
 14148:       [Hex_FWD, Hex_FWD]>,
 14149: 
 14150:     InstrItinData <tc_74a42bda, /*tc_ld*/
 14151:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2],
 14152:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14153: 
 14154:     InstrItinData <tc_759e57be, /*tc_3stall*/
 14155:       [InstrStage<1, [SLOT2]>], [4, 1],
 14156:       [Hex_FWD, Hex_FWD]>,
 14157: 
 14158:     InstrItinData <tc_76bb5435, /*tc_ld*/
 14159:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 1, 2, 2],
 14160:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14161: 
 14162:     InstrItinData <tc_7d6a2568, /*tc_3stall*/
 14163:       [InstrStage<1, [SLOT2]>], [1],
 14164:       [Hex_FWD]>,
 14165: 
 14166:     InstrItinData <tc_77f94a5e, /*tc_st*/
 14167:       [InstrStage<1, [SLOT0]>], [],
 14168:       []>,
 14169: 
 14170:     InstrItinData <tc_788b1d09, /*tc_3x*/
 14171:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 14172:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14173: 
 14174:     InstrItinData <tc_78f87ed3, /*tc_3stall*/
 14175:       [InstrStage<1, [SLOT0]>], [],
 14176:       []>,
 14177: 
 14178:     InstrItinData <tc_7af3a37e, /*tc_st*/
 14179:       [InstrStage<1, [SLOT0]>], [1, 3],
 14180:       [Hex_FWD, Hex_FWD]>,
 14181: 
 14182:     InstrItinData <tc_7b9187d3, /*tc_newvjump*/
 14183:       [InstrStage<1, [SLOT0]>], [3, 2],
 14184:       [Hex_FWD, Hex_FWD]>,
 14185: 
 14186:     InstrItinData <tc_7c28bd7e, /*tc_st*/
 14187:       [InstrStage<1, [SLOT0]>], [3],
 14188:       [Hex_FWD]>,
 14189: 
 14190:     InstrItinData <tc_7c31e19a, /*tc_st*/
 14191:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 14192:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14193: 
 14194:     InstrItinData <tc_7c6d32e4, /*tc_ld*/
 14195:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 2],
 14196:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14197: 
 14198:     InstrItinData <tc_7dc63b5c, /*tc_3x*/
 14199:       [InstrStage<1, [SLOT3]>], [4, 1],
 14200:       [Hex_FWD, Hex_FWD]>,
 14201: 
 14202:     InstrItinData <tc_7f58404a, /*tc_3stall*/
 14203:       [InstrStage<1, [SLOT3]>], [],
 14204:       []>,
 14205: 
 14206:     InstrItinData <tc_7f7f45f5, /*tc_4x*/
 14207:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1],
 14208:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14209: 
 14210:     InstrItinData <tc_7f8ae742, /*tc_3x*/
 14211:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1],
 14212:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14213: 
 14214:     InstrItinData <tc_8035e91f, /*tc_st*/
 14215:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 3],
 14216:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14217: 
 14218:     InstrItinData <tc_822c3c68, /*tc_ld*/
 14219:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2],
 14220:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14221: 
 14222:     InstrItinData <tc_829d8a86, /*tc_st*/
 14223:       [InstrStage<1, [SLOT0]>], [3, 1, 1, 2, 3],
 14224:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14225: 
 14226:     InstrItinData <tc_838c4d7a, /*tc_st*/
 14227:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2],
 14228:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14229: 
 14230:     InstrItinData <tc_84a7500d, /*tc_2*/
 14231:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 14232:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14233: 
 14234:     InstrItinData <tc_86173609, /*tc_2latepred*/
 14235:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 3, 2],
 14236:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14237: 
 14238:     InstrItinData <tc_887d1bb7, /*tc_st*/
 14239:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 2, 3],
 14240:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14241: 
 14242:     InstrItinData <tc_8a6d0d94, /*tc_ld*/
 14243:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2],
 14244:       [Hex_FWD, Hex_FWD]>,
 14245: 
 14246:     InstrItinData <tc_8a825db2, /*tc_2*/
 14247:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 14248:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14249: 
 14250:     InstrItinData <tc_8b5bd4f5, /*tc_2*/
 14251:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 14252:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14253: 
 14254:     InstrItinData <tc_8e82e8ca, /*tc_st*/
 14255:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 1, 2, 3],
 14256:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14257: 
 14258:     InstrItinData <tc_8f36a2fd, /*tc_ld*/
 14259:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1],
 14260:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14261: 
 14262:     InstrItinData <tc_9124c04f, /*tc_1*/
 14263:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 14264:       [Hex_FWD, Hex_FWD]>,
 14265: 
 14266:     InstrItinData <tc_92240447, /*tc_st*/
 14267:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 3],
 14268:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14269: 
 14270:     InstrItinData <tc_934753bb, /*tc_ld*/
 14271:       [InstrStage<1, [SLOT0]>], [3, 1, 2],
 14272:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14273: 
 14274:     InstrItinData <tc_937dd41c, /*tc_ld*/
 14275:       [InstrStage<1, [SLOT0, SLOT1]>], [],
 14276:       []>,
 14277: 
 14278:     InstrItinData <tc_9406230a, /*tc_3x*/
 14279:       [InstrStage<1, [SLOT3]>], [2, 1],
 14280:       [Hex_FWD, Hex_FWD]>,
 14281: 
 14282:     InstrItinData <tc_95a33176, /*tc_2*/
 14283:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [4, 2, 2],
 14284:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14285: 
 14286:     InstrItinData <tc_95f43c5e, /*tc_3*/
 14287:       [InstrStage<1, [SLOT2]>], [1],
 14288:       [Hex_FWD]>,
 14289: 
 14290:     InstrItinData <tc_96ef76ef, /*tc_st*/
 14291:       [InstrStage<1, [SLOT0]>], [1, 1, 2, 3],
 14292:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14293: 
 14294:     InstrItinData <tc_975a4e54, /*tc_newvjump*/
 14295:       [InstrStage<1, [SLOT0]>], [3, 3, 2],
 14296:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14297: 
 14298:     InstrItinData <tc_9783714b, /*tc_4x*/
 14299:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1],
 14300:       [Hex_FWD, Hex_FWD]>,
 14301: 
 14302:     InstrItinData <tc_9b20a062, /*tc_3stall*/
 14303:       [InstrStage<1, [SLOT2]>], [4, 1],
 14304:       [Hex_FWD, Hex_FWD]>,
 14305: 
 14306:     InstrItinData <tc_9b34f5e0, /*tc_3stall*/
 14307:       [InstrStage<1, [SLOT2]>], [],
 14308:       []>,
 14309: 
 14310:     InstrItinData <tc_9b3c0462, /*tc_2*/
 14311:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 14312:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14313: 
 14314:     InstrItinData <tc_9bcfb2ee, /*tc_st*/
 14315:       [InstrStage<1, [SLOT0]>], [1, 2, 3],
 14316:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14317: 
 14318:     InstrItinData <tc_9c52f549, /*tc_1*/
 14319:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 14320:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14321: 
 14322:     InstrItinData <tc_9e27f2f9, /*tc_1*/
 14323:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2, 2],
 14324:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14325: 
 14326:     InstrItinData <tc_9e72dc89, /*tc_4x*/
 14327:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1],
 14328:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14329: 
 14330:     InstrItinData <tc_9edb7c77, /*tc_4x*/
 14331:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 2, 1, 1, 2],
 14332:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14333: 
 14334:     InstrItinData <tc_9edefe01, /*tc_st*/
 14335:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 1, 2, 3],
 14336:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14337: 
 14338:     InstrItinData <tc_9f6cd987, /*tc_1*/
 14339:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 14340:       [Hex_FWD, Hex_FWD]>,
 14341: 
 14342:     InstrItinData <tc_a08b630b, /*tc_2*/
 14343:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 14344:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14345: 
 14346:     InstrItinData <tc_a1297125, /*tc_1*/
 14347:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2],
 14348:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14349: 
 14350:     InstrItinData <tc_a154b476, /*tc_3x*/
 14351:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 2],
 14352:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14353: 
 14354:     InstrItinData <tc_a2b365d2, /*tc_st*/
 14355:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 3],
 14356:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14357: 
 14358:     InstrItinData <tc_a3070909, /*tc_3stall*/
 14359:       [InstrStage<1, [SLOT0]>], [1, 1],
 14360:       [Hex_FWD, Hex_FWD]>,
 14361: 
 14362:     InstrItinData <tc_a32e03e7, /*tc_ld*/
 14363:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2, 2],
 14364:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14365: 
 14366:     InstrItinData <tc_a38c45dc, /*tc_3x*/
 14367:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 1, 1, 2],
 14368:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14369: 
 14370:     InstrItinData <tc_a4e22bbd, /*tc_2*/
 14371:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2],
 14372:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14373: 
 14374:     InstrItinData <tc_a4ee89db, /*tc_2early*/
 14375:       [InstrStage<1, [SLOT0]>], [],
 14376:       []>,
 14377: 
 14378:     InstrItinData <tc_a724463d, /*tc_3stall*/
 14379:       [InstrStage<1, [SLOT0]>], [4, 1],
 14380:       [Hex_FWD, Hex_FWD]>,
 14381: 
 14382:     InstrItinData <tc_a7a13fac, /*tc_1*/
 14383:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2, 2, 2],
 14384:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14385: 
 14386:     InstrItinData <tc_a7bdb22c, /*tc_2*/
 14387:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 14388:       [Hex_FWD, Hex_FWD]>,
 14389: 
 14390:     InstrItinData <tc_a9edeffa, /*tc_st*/
 14391:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 14392:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14393: 
 14394:     InstrItinData <tc_abfd9a6d, /*tc_ld*/
 14395:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2, 2],
 14396:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14397: 
 14398:     InstrItinData <tc_ac65613f, /*tc_ld*/
 14399:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 2, 2],
 14400:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14401: 
 14402:     InstrItinData <tc_addc37a8, /*tc_st*/
 14403:       [InstrStage<1, [SLOT0]>], [3, 1, 2, 2, 3],
 14404:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14405: 
 14406:     InstrItinData <tc_ae5babd7, /*tc_st*/
 14407:       [InstrStage<1, [SLOT0, SLOT1]>], [1, 2, 3],
 14408:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14409: 
 14410:     InstrItinData <tc_aee6250c, /*tc_ld*/
 14411:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 14412:       [Hex_FWD, Hex_FWD]>,
 14413: 
 14414:     InstrItinData <tc_af6af259, /*tc_ld*/
 14415:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1],
 14416:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14417: 
 14418:     InstrItinData <tc_b1ae5f67, /*tc_st*/
 14419:       [InstrStage<1, [SLOT0]>], [1],
 14420:       [Hex_FWD]>,
 14421: 
 14422:     InstrItinData <tc_b2196a3f, /*tc_3stall*/
 14423:       [InstrStage<1, [SLOT3]>], [1, 1],
 14424:       [Hex_FWD, Hex_FWD]>,
 14425: 
 14426:     InstrItinData <tc_b3d46584, /*tc_st*/
 14427:       [InstrStage<1, [SLOT0]>], [],
 14428:       []>,
 14429: 
 14430:     InstrItinData <tc_b4dc7630, /*tc_st*/
 14431:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 1, 2, 2, 3],
 14432:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14433: 
 14434:     InstrItinData <tc_b7c4062a, /*tc_ld*/
 14435:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 3, 1, 1, 2],
 14436:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14437: 
 14438:     InstrItinData <tc_b837298f, /*tc_1*/
 14439:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [],
 14440:       []>,
 14441: 
 14442:     InstrItinData <tc_b9bec29e, /*tc_3stall*/
 14443:       [InstrStage<1, [SLOT2]>], [],
 14444:       []>,
 14445: 
 14446:     InstrItinData <tc_ba9255a6, /*tc_st*/
 14447:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 2, 3],
 14448:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14449: 
 14450:     InstrItinData <tc_bb07f2c5, /*tc_st*/
 14451:       [InstrStage<1, [SLOT0, SLOT1]>], [3, 2, 3],
 14452:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14453: 
 14454:     InstrItinData <tc_bb78483e, /*tc_3stall*/
 14455:       [InstrStage<1, [SLOT3]>], [4, 1, 1],
 14456:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14457: 
 14458:     InstrItinData <tc_bb831a7c, /*tc_2*/
 14459:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2, 2, 2],
 14460:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14461: 
 14462:     InstrItinData <tc_bf2ffc0f, /*tc_ld*/
 14463:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 1, 2],
 14464:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14465: 
 14466:     InstrItinData <tc_c20701f0, /*tc_2*/
 14467:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 14468:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14469: 
 14470:     InstrItinData <tc_c21d7447, /*tc_3x*/
 14471:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 14472:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14473: 
 14474:     InstrItinData <tc_c57d9f39, /*tc_1*/
 14475:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 14476:       [Hex_FWD, Hex_FWD]>,
 14477: 
 14478:     InstrItinData <tc_c818ff7f, /*tc_newvjump*/
 14479:       [InstrStage<1, [SLOT0]>], [],
 14480:       []>,
 14481: 
 14482:     InstrItinData <tc_ce59038e, /*tc_st*/
 14483:       [InstrStage<1, [SLOT0]>], [3, 2, 1, 2, 3],
 14484:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14485: 
 14486:     InstrItinData <tc_cfa0e29b, /*tc_st*/
 14487:       [InstrStage<1, [SLOT0]>], [2, 2, 3],
 14488:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14489: 
 14490:     InstrItinData <tc_d03278fd, /*tc_st*/
 14491:       [InstrStage<1, [SLOT0, SLOT1]>], [2, 1, 2, 2],
 14492:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14493: 
 14494:     InstrItinData <tc_d234b61a, /*tc_st*/
 14495:       [InstrStage<1, [SLOT0]>], [1],
 14496:       [Hex_FWD]>,
 14497: 
 14498:     InstrItinData <tc_d33e5eee, /*tc_1*/
 14499:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2, 2],
 14500:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 14501-14630 / 第 14501-14630 行

```tablegen
 14501: 
 14502:     InstrItinData <tc_d3632d88, /*tc_2*/
 14503:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 14504:       [Hex_FWD, Hex_FWD]>,
 14505: 
 14506:     InstrItinData <tc_d45ba9cd, /*tc_ld*/
 14507:       [InstrStage<1, [SLOT0]>], [1],
 14508:       [Hex_FWD]>,
 14509: 
 14510:     InstrItinData <tc_d57d649c, /*tc_3stall*/
 14511:       [InstrStage<1, [SLOT2]>], [2],
 14512:       [Hex_FWD]>,
 14513: 
 14514:     InstrItinData <tc_d61dfdc3, /*tc_2*/
 14515:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 14516:       [Hex_FWD, Hex_FWD]>,
 14517: 
 14518:     InstrItinData <tc_d68dca5c, /*tc_3stall*/
 14519:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 14520:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14521: 
 14522:     InstrItinData <tc_d71ea8fa, /*tc_3x*/
 14523:       [InstrStage<1, [SLOT3]>], [2, 1],
 14524:       [Hex_FWD, Hex_FWD]>,
 14525: 
 14526:     InstrItinData <tc_d7718fbe, /*tc_3x*/
 14527:       [InstrStage<1, [SLOT3]>], [1],
 14528:       [Hex_FWD]>,
 14529: 
 14530:     InstrItinData <tc_db596beb, /*tc_3x*/
 14531:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1],
 14532:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14533: 
 14534:     InstrItinData <tc_db96aa6b, /*tc_st*/
 14535:       [InstrStage<1, [SLOT0]>], [1],
 14536:       [Hex_FWD]>,
 14537: 
 14538:     InstrItinData <tc_dc51281d, /*tc_3*/
 14539:       [InstrStage<1, [SLOT2]>], [2, 1],
 14540:       [Hex_FWD, Hex_FWD]>,
 14541: 
 14542:     InstrItinData <tc_decdde8a, /*tc_1*/
 14543:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2],
 14544:       [Hex_FWD]>,
 14545: 
 14546:     InstrItinData <tc_df5d53f9, /*tc_newvjump*/
 14547:       [InstrStage<1, [SLOT0]>], [3, 2, 1],
 14548:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14549: 
 14550:     InstrItinData <tc_e3d699e3, /*tc_2*/
 14551:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2],
 14552:       [Hex_FWD, Hex_FWD]>,
 14553: 
 14554:     InstrItinData <tc_e60def48, /*tc_1*/
 14555:       [InstrStage<1, [SLOT2, SLOT3]>], [2],
 14556:       [Hex_FWD]>,
 14557: 
 14558:     InstrItinData <tc_e9170fb7, /*tc_ld*/
 14559:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1],
 14560:       [Hex_FWD, Hex_FWD]>,
 14561: 
 14562:     InstrItinData <tc_ed03645c, /*tc_1*/
 14563:       [InstrStage<1, [SLOT2]>], [3, 2],
 14564:       [Hex_FWD, Hex_FWD]>,
 14565: 
 14566:     InstrItinData <tc_ed3f8d2a, /*tc_ld*/
 14567:       [InstrStage<1, [SLOT0]>], [4, 1, 1],
 14568:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14569: 
 14570:     InstrItinData <tc_eed07714, /*tc_ld*/
 14571:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 1, 2],
 14572:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14573: 
 14574:     InstrItinData <tc_eeda4109, /*tc_1*/
 14575:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [3, 2],
 14576:       [Hex_FWD, Hex_FWD]>,
 14577: 
 14578:     InstrItinData <tc_ef921005, /*tc_1*/
 14579:       [InstrStage<1, [SLOT2, SLOT3]>], [3, 2],
 14580:       [Hex_FWD, Hex_FWD]>,
 14581: 
 14582:     InstrItinData <tc_f098b237, /*tc_2*/
 14583:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 14584:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14585: 
 14586:     InstrItinData <tc_f0cdeccf, /*tc_3x*/
 14587:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 1, 1, 2],
 14588:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14589: 
 14590:     InstrItinData <tc_f0e8e832, /*tc_4x*/
 14591:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 1, 1],
 14592:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14593: 
 14594:     InstrItinData <tc_f34c1c21, /*tc_2*/
 14595:       [InstrStage<1, [SLOT2, SLOT3]>], [4, 2, 2],
 14596:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14597: 
 14598:     InstrItinData <tc_f38f92e1, /*tc_newvjump*/
 14599:       [InstrStage<1, [SLOT0]>], [2],
 14600:       [Hex_FWD]>,
 14601: 
 14602:     InstrItinData <tc_f529831b, /*tc_latepredstaia*/
 14603:       [InstrStage<1, [SLOT0]>], [4, 3, 1, 2, 3],
 14604:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14605: 
 14606:     InstrItinData <tc_f6e2aff9, /*tc_newvjump*/
 14607:       [InstrStage<1, [SLOT0]>], [3, 2, 2],
 14608:       [Hex_FWD, Hex_FWD, Hex_FWD]>,
 14609: 
 14610:     InstrItinData <tc_f7569068, /*tc_4x*/
 14611:       [InstrStage<1, [SLOT2, SLOT3]>], [5, 5, 1, 1],
 14612:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>,
 14613: 
 14614:     InstrItinData <tc_f97707c1, /*tc_1*/
 14615:       [InstrStage<1, [SLOT2]>], [2],
 14616:       [Hex_FWD]>,
 14617: 
 14618:     InstrItinData <tc_f999c66e, /*tc_1*/
 14619:       [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>], [2, 2],
 14620:       [Hex_FWD, Hex_FWD]>,
 14621: 
 14622:     InstrItinData <tc_fae9dfa5, /*tc_3x*/
 14623:       [InstrStage<1, [SLOT3]>], [4, 2],
 14624:       [Hex_FWD, Hex_FWD]>,
 14625: 
 14626:     InstrItinData <tc_fedb7e19, /*tc_ld*/
 14627:       [InstrStage<1, [SLOT0, SLOT1]>], [4, 2, 1, 2],
 14628:       [Hex_FWD, Hex_FWD, Hex_FWD, Hex_FWD]>
 14629:   ];
 14630: }
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

## Key Concepts / 关键概念

- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录
- generated target metadata / 生成的目标元数据

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
