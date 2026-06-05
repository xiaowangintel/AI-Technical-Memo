# HexagonDepMapAsm2Intrin.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonDepMapAsm2Intrin.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon generated mappings between assembler mnemonics and intrinsics using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。 内容以生成表项为主。
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
    11: 
    12: // V5 Scalar Instructions.
    13: 
    14: def: Pat<(int_hexagon_A2_abs IntRegs:$src1),
    15:          (A2_abs IntRegs:$src1)>, Requires<[HasV5]>;
    16: def: Pat<(int_hexagon_A2_absp DoubleRegs:$src1),
    17:          (A2_absp DoubleRegs:$src1)>, Requires<[HasV5]>;
    18: def: Pat<(int_hexagon_A2_abssat IntRegs:$src1),
    19:          (A2_abssat IntRegs:$src1)>, Requires<[HasV5]>;
    20: def: Pat<(int_hexagon_A2_add IntRegs:$src1, IntRegs:$src2),
    21:          (A2_add IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    22: def: Pat<(int_hexagon_A2_addh_h16_hh IntRegs:$src1, IntRegs:$src2),
    23:          (A2_addh_h16_hh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    24: def: Pat<(int_hexagon_A2_addh_h16_hl IntRegs:$src1, IntRegs:$src2),
    25:          (A2_addh_h16_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    26: def: Pat<(int_hexagon_A2_addh_h16_lh IntRegs:$src1, IntRegs:$src2),
    27:          (A2_addh_h16_lh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    28: def: Pat<(int_hexagon_A2_addh_h16_ll IntRegs:$src1, IntRegs:$src2),
    29:          (A2_addh_h16_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    30: def: Pat<(int_hexagon_A2_addh_h16_sat_hh IntRegs:$src1, IntRegs:$src2),
    31:          (A2_addh_h16_sat_hh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    32: def: Pat<(int_hexagon_A2_addh_h16_sat_hl IntRegs:$src1, IntRegs:$src2),
    33:          (A2_addh_h16_sat_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    34: def: Pat<(int_hexagon_A2_addh_h16_sat_lh IntRegs:$src1, IntRegs:$src2),
    35:          (A2_addh_h16_sat_lh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    36: def: Pat<(int_hexagon_A2_addh_h16_sat_ll IntRegs:$src1, IntRegs:$src2),
    37:          (A2_addh_h16_sat_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    38: def: Pat<(int_hexagon_A2_addh_l16_hl IntRegs:$src1, IntRegs:$src2),
    39:          (A2_addh_l16_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    40: def: Pat<(int_hexagon_A2_addh_l16_ll IntRegs:$src1, IntRegs:$src2),
    41:          (A2_addh_l16_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    42: def: Pat<(int_hexagon_A2_addh_l16_sat_hl IntRegs:$src1, IntRegs:$src2),
    43:          (A2_addh_l16_sat_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    44: def: Pat<(int_hexagon_A2_addh_l16_sat_ll IntRegs:$src1, IntRegs:$src2),
    45:          (A2_addh_l16_sat_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    46: def: Pat<(int_hexagon_A2_addi IntRegs:$src1, s32_0ImmPred_timm:$src2),
    47:          (A2_addi IntRegs:$src1, s32_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
    48: def: Pat<(int_hexagon_A2_addp DoubleRegs:$src1, DoubleRegs:$src2),
    49:          (A2_addp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    50: def: Pat<(int_hexagon_A2_addpsat DoubleRegs:$src1, DoubleRegs:$src2),
    51:          (A2_addpsat DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    52: def: Pat<(int_hexagon_A2_addsat IntRegs:$src1, IntRegs:$src2),
    53:          (A2_addsat IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    54: def: Pat<(int_hexagon_A2_addsp IntRegs:$src1, DoubleRegs:$src2),
    55:          (A2_addsp IntRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    56: def: Pat<(int_hexagon_A2_and IntRegs:$src1, IntRegs:$src2),
    57:          (A2_and IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    58: def: Pat<(int_hexagon_A2_andir IntRegs:$src1, s32_0ImmPred_timm:$src2),
    59:          (A2_andir IntRegs:$src1, s32_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
    60: def: Pat<(int_hexagon_A2_andp DoubleRegs:$src1, DoubleRegs:$src2),
    61:          (A2_andp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    62: def: Pat<(int_hexagon_A2_aslh IntRegs:$src1),
    63:          (A2_aslh IntRegs:$src1)>, Requires<[HasV5]>;
    64: def: Pat<(int_hexagon_A2_asrh IntRegs:$src1),
    65:          (A2_asrh IntRegs:$src1)>, Requires<[HasV5]>;
    66: def: Pat<(int_hexagon_A2_combine_hh IntRegs:$src1, IntRegs:$src2),
    67:          (A2_combine_hh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    68: def: Pat<(int_hexagon_A2_combine_hl IntRegs:$src1, IntRegs:$src2),
    69:          (A2_combine_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    70: def: Pat<(int_hexagon_A2_combine_lh IntRegs:$src1, IntRegs:$src2),
    71:          (A2_combine_lh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    72: def: Pat<(int_hexagon_A2_combine_ll IntRegs:$src1, IntRegs:$src2),
    73:          (A2_combine_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    74: def: Pat<(int_hexagon_A2_combineii s32_0ImmPred_timm:$src1, s8_0ImmPred_timm:$src2),
    75:          (A2_combineii s32_0ImmPred_timm:$src1, s8_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
    76: def: Pat<(int_hexagon_A2_combinew IntRegs:$src1, IntRegs:$src2),
    77:          (A2_combinew IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    78: def: Pat<(int_hexagon_A2_max IntRegs:$src1, IntRegs:$src2),
    79:          (A2_max IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    80: def: Pat<(int_hexagon_A2_maxp DoubleRegs:$src1, DoubleRegs:$src2),
    81:          (A2_maxp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    82: def: Pat<(int_hexagon_A2_maxu IntRegs:$src1, IntRegs:$src2),
    83:          (A2_maxu IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    84: def: Pat<(int_hexagon_A2_maxup DoubleRegs:$src1, DoubleRegs:$src2),
    85:          (A2_maxup DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    86: def: Pat<(int_hexagon_A2_min IntRegs:$src1, IntRegs:$src2),
    87:          (A2_min IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    88: def: Pat<(int_hexagon_A2_minp DoubleRegs:$src1, DoubleRegs:$src2),
    89:          (A2_minp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    90: def: Pat<(int_hexagon_A2_minu IntRegs:$src1, IntRegs:$src2),
    91:          (A2_minu IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
    92: def: Pat<(int_hexagon_A2_minup DoubleRegs:$src1, DoubleRegs:$src2),
    93:          (A2_minup DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
    94: def: Pat<(int_hexagon_A2_negp DoubleRegs:$src1),
    95:          (A2_negp DoubleRegs:$src1)>, Requires<[HasV5]>;
    96: def: Pat<(int_hexagon_A2_negsat IntRegs:$src1),
    97:          (A2_negsat IntRegs:$src1)>, Requires<[HasV5]>;
    98: def: Pat<(int_hexagon_A2_notp DoubleRegs:$src1),
    99:          (A2_notp DoubleRegs:$src1)>, Requires<[HasV5]>;
   100: def: Pat<(int_hexagon_A2_or IntRegs:$src1, IntRegs:$src2),
   101:          (A2_or IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   102: def: Pat<(int_hexagon_A2_orir IntRegs:$src1, s32_0ImmPred_timm:$src2),
   103:          (A2_orir IntRegs:$src1, s32_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   104: def: Pat<(int_hexagon_A2_orp DoubleRegs:$src1, DoubleRegs:$src2),
   105:          (A2_orp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   106: def: Pat<(int_hexagon_A2_roundsat DoubleRegs:$src1),
   107:          (A2_roundsat DoubleRegs:$src1)>, Requires<[HasV5]>;
   108: def: Pat<(int_hexagon_A2_sat DoubleRegs:$src1),
   109:          (A2_sat DoubleRegs:$src1)>, Requires<[HasV5]>;
   110: def: Pat<(int_hexagon_A2_satb IntRegs:$src1),
   111:          (A2_satb IntRegs:$src1)>, Requires<[HasV5]>;
   112: def: Pat<(int_hexagon_A2_sath IntRegs:$src1),
   113:          (A2_sath IntRegs:$src1)>, Requires<[HasV5]>;
   114: def: Pat<(int_hexagon_A2_satub IntRegs:$src1),
   115:          (A2_satub IntRegs:$src1)>, Requires<[HasV5]>;
   116: def: Pat<(int_hexagon_A2_satuh IntRegs:$src1),
   117:          (A2_satuh IntRegs:$src1)>, Requires<[HasV5]>;
   118: def: Pat<(int_hexagon_A2_sub IntRegs:$src1, IntRegs:$src2),
   119:          (A2_sub IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   120: def: Pat<(int_hexagon_A2_subh_h16_hh IntRegs:$src1, IntRegs:$src2),
   121:          (A2_subh_h16_hh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   122: def: Pat<(int_hexagon_A2_subh_h16_hl IntRegs:$src1, IntRegs:$src2),
   123:          (A2_subh_h16_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   124: def: Pat<(int_hexagon_A2_subh_h16_lh IntRegs:$src1, IntRegs:$src2),
   125:          (A2_subh_h16_lh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   126: def: Pat<(int_hexagon_A2_subh_h16_ll IntRegs:$src1, IntRegs:$src2),
   127:          (A2_subh_h16_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   128: def: Pat<(int_hexagon_A2_subh_h16_sat_hh IntRegs:$src1, IntRegs:$src2),
   129:          (A2_subh_h16_sat_hh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   130: def: Pat<(int_hexagon_A2_subh_h16_sat_hl IntRegs:$src1, IntRegs:$src2),
   131:          (A2_subh_h16_sat_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   132: def: Pat<(int_hexagon_A2_subh_h16_sat_lh IntRegs:$src1, IntRegs:$src2),
   133:          (A2_subh_h16_sat_lh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   134: def: Pat<(int_hexagon_A2_subh_h16_sat_ll IntRegs:$src1, IntRegs:$src2),
   135:          (A2_subh_h16_sat_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   136: def: Pat<(int_hexagon_A2_subh_l16_hl IntRegs:$src1, IntRegs:$src2),
   137:          (A2_subh_l16_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   138: def: Pat<(int_hexagon_A2_subh_l16_ll IntRegs:$src1, IntRegs:$src2),
   139:          (A2_subh_l16_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   140: def: Pat<(int_hexagon_A2_subh_l16_sat_hl IntRegs:$src1, IntRegs:$src2),
   141:          (A2_subh_l16_sat_hl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   142: def: Pat<(int_hexagon_A2_subh_l16_sat_ll IntRegs:$src1, IntRegs:$src2),
   143:          (A2_subh_l16_sat_ll IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   144: def: Pat<(int_hexagon_A2_subp DoubleRegs:$src1, DoubleRegs:$src2),
   145:          (A2_subp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   146: def: Pat<(int_hexagon_A2_subri s32_0ImmPred_timm:$src1, IntRegs:$src2),
   147:          (A2_subri s32_0ImmPred_timm:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   148: def: Pat<(int_hexagon_A2_subsat IntRegs:$src1, IntRegs:$src2),
   149:          (A2_subsat IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   150: def: Pat<(int_hexagon_A2_svaddh IntRegs:$src1, IntRegs:$src2),
   151:          (A2_svaddh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   152: def: Pat<(int_hexagon_A2_svaddhs IntRegs:$src1, IntRegs:$src2),
   153:          (A2_svaddhs IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   154: def: Pat<(int_hexagon_A2_svadduhs IntRegs:$src1, IntRegs:$src2),
   155:          (A2_svadduhs IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   156: def: Pat<(int_hexagon_A2_svavgh IntRegs:$src1, IntRegs:$src2),
   157:          (A2_svavgh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   158: def: Pat<(int_hexagon_A2_svavghs IntRegs:$src1, IntRegs:$src2),
   159:          (A2_svavghs IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   160: def: Pat<(int_hexagon_A2_svnavgh IntRegs:$src1, IntRegs:$src2),
   161:          (A2_svnavgh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   162: def: Pat<(int_hexagon_A2_svsubh IntRegs:$src1, IntRegs:$src2),
   163:          (A2_svsubh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   164: def: Pat<(int_hexagon_A2_svsubhs IntRegs:$src1, IntRegs:$src2),
   165:          (A2_svsubhs IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   166: def: Pat<(int_hexagon_A2_svsubuhs IntRegs:$src1, IntRegs:$src2),
   167:          (A2_svsubuhs IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   168: def: Pat<(int_hexagon_A2_swiz IntRegs:$src1),
   169:          (A2_swiz IntRegs:$src1)>, Requires<[HasV5]>;
   170: def: Pat<(int_hexagon_A2_sxtb IntRegs:$src1),
   171:          (A2_sxtb IntRegs:$src1)>, Requires<[HasV5]>;
   172: def: Pat<(int_hexagon_A2_sxth IntRegs:$src1),
   173:          (A2_sxth IntRegs:$src1)>, Requires<[HasV5]>;
   174: def: Pat<(int_hexagon_A2_sxtw IntRegs:$src1),
   175:          (A2_sxtw IntRegs:$src1)>, Requires<[HasV5]>;
   176: def: Pat<(int_hexagon_A2_tfr IntRegs:$src1),
   177:          (A2_tfr IntRegs:$src1)>, Requires<[HasV5]>;
   178: def: Pat<(int_hexagon_A2_tfrih IntRegs:$src1, u16_0ImmPred_timm:$src2),
   179:          (A2_tfrih IntRegs:$src1, u16_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   180: def: Pat<(int_hexagon_A2_tfril IntRegs:$src1, u16_0ImmPred_timm:$src2),
   181:          (A2_tfril IntRegs:$src1, u16_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   182: def: Pat<(int_hexagon_A2_tfrp DoubleRegs:$src1),
   183:          (A2_tfrp DoubleRegs:$src1)>, Requires<[HasV5]>;
   184: def: Pat<(int_hexagon_A2_tfrsi s32_0ImmPred_timm:$src1),
   185:          (A2_tfrsi s32_0ImmPred_timm:$src1)>, Requires<[HasV5]>;
   186: def: Pat<(int_hexagon_A2_vabsh DoubleRegs:$src1),
   187:          (A2_vabsh DoubleRegs:$src1)>, Requires<[HasV5]>;
   188: def: Pat<(int_hexagon_A2_vabshsat DoubleRegs:$src1),
   189:          (A2_vabshsat DoubleRegs:$src1)>, Requires<[HasV5]>;
   190: def: Pat<(int_hexagon_A2_vabsw DoubleRegs:$src1),
   191:          (A2_vabsw DoubleRegs:$src1)>, Requires<[HasV5]>;
   192: def: Pat<(int_hexagon_A2_vabswsat DoubleRegs:$src1),
   193:          (A2_vabswsat DoubleRegs:$src1)>, Requires<[HasV5]>;
   194: def: Pat<(int_hexagon_A2_vaddb_map DoubleRegs:$src1, DoubleRegs:$src2),
   195:          (A2_vaddub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   196: def: Pat<(int_hexagon_A2_vaddh DoubleRegs:$src1, DoubleRegs:$src2),
   197:          (A2_vaddh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   198: def: Pat<(int_hexagon_A2_vaddhs DoubleRegs:$src1, DoubleRegs:$src2),
   199:          (A2_vaddhs DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   200: def: Pat<(int_hexagon_A2_vaddub DoubleRegs:$src1, DoubleRegs:$src2),
   201:          (A2_vaddub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   202: def: Pat<(int_hexagon_A2_vaddubs DoubleRegs:$src1, DoubleRegs:$src2),
   203:          (A2_vaddubs DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   204: def: Pat<(int_hexagon_A2_vadduhs DoubleRegs:$src1, DoubleRegs:$src2),
   205:          (A2_vadduhs DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   206: def: Pat<(int_hexagon_A2_vaddw DoubleRegs:$src1, DoubleRegs:$src2),
   207:          (A2_vaddw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   208: def: Pat<(int_hexagon_A2_vaddws DoubleRegs:$src1, DoubleRegs:$src2),
   209:          (A2_vaddws DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   210: def: Pat<(int_hexagon_A2_vavgh DoubleRegs:$src1, DoubleRegs:$src2),
   211:          (A2_vavgh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   212: def: Pat<(int_hexagon_A2_vavghcr DoubleRegs:$src1, DoubleRegs:$src2),
   213:          (A2_vavghcr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   214: def: Pat<(int_hexagon_A2_vavghr DoubleRegs:$src1, DoubleRegs:$src2),
   215:          (A2_vavghr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   216: def: Pat<(int_hexagon_A2_vavgub DoubleRegs:$src1, DoubleRegs:$src2),
   217:          (A2_vavgub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   218: def: Pat<(int_hexagon_A2_vavgubr DoubleRegs:$src1, DoubleRegs:$src2),
   219:          (A2_vavgubr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   220: def: Pat<(int_hexagon_A2_vavguh DoubleRegs:$src1, DoubleRegs:$src2),
   221:          (A2_vavguh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   222: def: Pat<(int_hexagon_A2_vavguhr DoubleRegs:$src1, DoubleRegs:$src2),
   223:          (A2_vavguhr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   224: def: Pat<(int_hexagon_A2_vavguw DoubleRegs:$src1, DoubleRegs:$src2),
   225:          (A2_vavguw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   226: def: Pat<(int_hexagon_A2_vavguwr DoubleRegs:$src1, DoubleRegs:$src2),
   227:          (A2_vavguwr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   228: def: Pat<(int_hexagon_A2_vavgw DoubleRegs:$src1, DoubleRegs:$src2),
   229:          (A2_vavgw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   230: def: Pat<(int_hexagon_A2_vavgwcr DoubleRegs:$src1, DoubleRegs:$src2),
   231:          (A2_vavgwcr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   232: def: Pat<(int_hexagon_A2_vavgwr DoubleRegs:$src1, DoubleRegs:$src2),
   233:          (A2_vavgwr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   234: def: Pat<(int_hexagon_A2_vcmpbeq DoubleRegs:$src1, DoubleRegs:$src2),
   235:          (C2_tfrpr (A2_vcmpbeq DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   236: def: Pat<(int_hexagon_A2_vcmpbgtu DoubleRegs:$src1, DoubleRegs:$src2),
   237:          (C2_tfrpr (A2_vcmpbgtu DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   238: def: Pat<(int_hexagon_A2_vcmpheq DoubleRegs:$src1, DoubleRegs:$src2),
   239:          (C2_tfrpr (A2_vcmpheq DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   240: def: Pat<(int_hexagon_A2_vcmphgt DoubleRegs:$src1, DoubleRegs:$src2),
   241:          (C2_tfrpr (A2_vcmphgt DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   242: def: Pat<(int_hexagon_A2_vcmphgtu DoubleRegs:$src1, DoubleRegs:$src2),
   243:          (C2_tfrpr (A2_vcmphgtu DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   244: def: Pat<(int_hexagon_A2_vcmpweq DoubleRegs:$src1, DoubleRegs:$src2),
   245:          (C2_tfrpr (A2_vcmpweq DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   246: def: Pat<(int_hexagon_A2_vcmpwgt DoubleRegs:$src1, DoubleRegs:$src2),
   247:          (C2_tfrpr (A2_vcmpwgt DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   248: def: Pat<(int_hexagon_A2_vcmpwgtu DoubleRegs:$src1, DoubleRegs:$src2),
   249:          (C2_tfrpr (A2_vcmpwgtu DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   250: def: Pat<(int_hexagon_A2_vconj DoubleRegs:$src1),
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 251-500 / 第 251-500 行

```tablegen
   251:          (A2_vconj DoubleRegs:$src1)>, Requires<[HasV5]>;
   252: def: Pat<(int_hexagon_A2_vmaxb DoubleRegs:$src1, DoubleRegs:$src2),
   253:          (A2_vmaxb DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   254: def: Pat<(int_hexagon_A2_vmaxh DoubleRegs:$src1, DoubleRegs:$src2),
   255:          (A2_vmaxh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   256: def: Pat<(int_hexagon_A2_vmaxub DoubleRegs:$src1, DoubleRegs:$src2),
   257:          (A2_vmaxub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   258: def: Pat<(int_hexagon_A2_vmaxuh DoubleRegs:$src1, DoubleRegs:$src2),
   259:          (A2_vmaxuh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   260: def: Pat<(int_hexagon_A2_vmaxuw DoubleRegs:$src1, DoubleRegs:$src2),
   261:          (A2_vmaxuw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   262: def: Pat<(int_hexagon_A2_vmaxw DoubleRegs:$src1, DoubleRegs:$src2),
   263:          (A2_vmaxw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   264: def: Pat<(int_hexagon_A2_vminb DoubleRegs:$src1, DoubleRegs:$src2),
   265:          (A2_vminb DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   266: def: Pat<(int_hexagon_A2_vminh DoubleRegs:$src1, DoubleRegs:$src2),
   267:          (A2_vminh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   268: def: Pat<(int_hexagon_A2_vminub DoubleRegs:$src1, DoubleRegs:$src2),
   269:          (A2_vminub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   270: def: Pat<(int_hexagon_A2_vminuh DoubleRegs:$src1, DoubleRegs:$src2),
   271:          (A2_vminuh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   272: def: Pat<(int_hexagon_A2_vminuw DoubleRegs:$src1, DoubleRegs:$src2),
   273:          (A2_vminuw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   274: def: Pat<(int_hexagon_A2_vminw DoubleRegs:$src1, DoubleRegs:$src2),
   275:          (A2_vminw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   276: def: Pat<(int_hexagon_A2_vnavgh DoubleRegs:$src1, DoubleRegs:$src2),
   277:          (A2_vnavgh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   278: def: Pat<(int_hexagon_A2_vnavghcr DoubleRegs:$src1, DoubleRegs:$src2),
   279:          (A2_vnavghcr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   280: def: Pat<(int_hexagon_A2_vnavghr DoubleRegs:$src1, DoubleRegs:$src2),
   281:          (A2_vnavghr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   282: def: Pat<(int_hexagon_A2_vnavgw DoubleRegs:$src1, DoubleRegs:$src2),
   283:          (A2_vnavgw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   284: def: Pat<(int_hexagon_A2_vnavgwcr DoubleRegs:$src1, DoubleRegs:$src2),
   285:          (A2_vnavgwcr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   286: def: Pat<(int_hexagon_A2_vnavgwr DoubleRegs:$src1, DoubleRegs:$src2),
   287:          (A2_vnavgwr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   288: def: Pat<(int_hexagon_A2_vraddub DoubleRegs:$src1, DoubleRegs:$src2),
   289:          (A2_vraddub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   290: def: Pat<(int_hexagon_A2_vraddub_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   291:          (A2_vraddub_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   292: def: Pat<(int_hexagon_A2_vrsadub DoubleRegs:$src1, DoubleRegs:$src2),
   293:          (A2_vrsadub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   294: def: Pat<(int_hexagon_A2_vrsadub_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   295:          (A2_vrsadub_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   296: def: Pat<(int_hexagon_A2_vsubb_map DoubleRegs:$src1, DoubleRegs:$src2),
   297:          (A2_vsubub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   298: def: Pat<(int_hexagon_A2_vsubh DoubleRegs:$src1, DoubleRegs:$src2),
   299:          (A2_vsubh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   300: def: Pat<(int_hexagon_A2_vsubhs DoubleRegs:$src1, DoubleRegs:$src2),
   301:          (A2_vsubhs DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   302: def: Pat<(int_hexagon_A2_vsubub DoubleRegs:$src1, DoubleRegs:$src2),
   303:          (A2_vsubub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   304: def: Pat<(int_hexagon_A2_vsububs DoubleRegs:$src1, DoubleRegs:$src2),
   305:          (A2_vsububs DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   306: def: Pat<(int_hexagon_A2_vsubuhs DoubleRegs:$src1, DoubleRegs:$src2),
   307:          (A2_vsubuhs DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   308: def: Pat<(int_hexagon_A2_vsubw DoubleRegs:$src1, DoubleRegs:$src2),
   309:          (A2_vsubw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   310: def: Pat<(int_hexagon_A2_vsubws DoubleRegs:$src1, DoubleRegs:$src2),
   311:          (A2_vsubws DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   312: def: Pat<(int_hexagon_A2_xor IntRegs:$src1, IntRegs:$src2),
   313:          (A2_xor IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   314: def: Pat<(int_hexagon_A2_xorp DoubleRegs:$src1, DoubleRegs:$src2),
   315:          (A2_xorp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   316: def: Pat<(int_hexagon_A2_zxtb IntRegs:$src1),
   317:          (A2_zxtb IntRegs:$src1)>, Requires<[HasV5]>;
   318: def: Pat<(int_hexagon_A2_zxth IntRegs:$src1),
   319:          (A2_zxth IntRegs:$src1)>, Requires<[HasV5]>;
   320: def: Pat<(int_hexagon_A4_andn IntRegs:$src1, IntRegs:$src2),
   321:          (A4_andn IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   322: def: Pat<(int_hexagon_A4_andnp DoubleRegs:$src1, DoubleRegs:$src2),
   323:          (A4_andnp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   324: def: Pat<(int_hexagon_A4_bitsplit IntRegs:$src1, IntRegs:$src2),
   325:          (A4_bitsplit IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   326: def: Pat<(int_hexagon_A4_bitspliti IntRegs:$src1, u5_0ImmPred_timm:$src2),
   327:          (A4_bitspliti IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   328: def: Pat<(int_hexagon_A4_boundscheck IntRegs:$src1, DoubleRegs:$src2),
   329:          (C2_tfrpr (A4_boundscheck IntRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   330: def: Pat<(int_hexagon_A4_cmpbeq IntRegs:$src1, IntRegs:$src2),
   331:          (C2_tfrpr (A4_cmpbeq IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   332: def: Pat<(int_hexagon_A4_cmpbeqi IntRegs:$src1, u8_0ImmPred_timm:$src2),
   333:          (C2_tfrpr (A4_cmpbeqi IntRegs:$src1, u8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   334: def: Pat<(int_hexagon_A4_cmpbgt IntRegs:$src1, IntRegs:$src2),
   335:          (C2_tfrpr (A4_cmpbgt IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   336: def: Pat<(int_hexagon_A4_cmpbgti IntRegs:$src1, s8_0ImmPred_timm:$src2),
   337:          (C2_tfrpr (A4_cmpbgti IntRegs:$src1, s8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   338: def: Pat<(int_hexagon_A4_cmpbgtu IntRegs:$src1, IntRegs:$src2),
   339:          (C2_tfrpr (A4_cmpbgtu IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   340: def: Pat<(int_hexagon_A4_cmpbgtui IntRegs:$src1, u32_0ImmPred_timm:$src2),
   341:          (C2_tfrpr (A4_cmpbgtui IntRegs:$src1, u32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   342: def: Pat<(int_hexagon_A4_cmpheq IntRegs:$src1, IntRegs:$src2),
   343:          (C2_tfrpr (A4_cmpheq IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   344: def: Pat<(int_hexagon_A4_cmpheqi IntRegs:$src1, s32_0ImmPred_timm:$src2),
   345:          (C2_tfrpr (A4_cmpheqi IntRegs:$src1, s32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   346: def: Pat<(int_hexagon_A4_cmphgt IntRegs:$src1, IntRegs:$src2),
   347:          (C2_tfrpr (A4_cmphgt IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   348: def: Pat<(int_hexagon_A4_cmphgti IntRegs:$src1, s32_0ImmPred_timm:$src2),
   349:          (C2_tfrpr (A4_cmphgti IntRegs:$src1, s32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   350: def: Pat<(int_hexagon_A4_cmphgtu IntRegs:$src1, IntRegs:$src2),
   351:          (C2_tfrpr (A4_cmphgtu IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   352: def: Pat<(int_hexagon_A4_cmphgtui IntRegs:$src1, u32_0ImmPred_timm:$src2),
   353:          (C2_tfrpr (A4_cmphgtui IntRegs:$src1, u32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   354: def: Pat<(int_hexagon_A4_combineir s32_0ImmPred_timm:$src1, IntRegs:$src2),
   355:          (A4_combineir s32_0ImmPred_timm:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   356: def: Pat<(int_hexagon_A4_combineri IntRegs:$src1, s32_0ImmPred_timm:$src2),
   357:          (A4_combineri IntRegs:$src1, s32_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   358: def: Pat<(int_hexagon_A4_cround_ri IntRegs:$src1, u5_0ImmPred_timm:$src2),
   359:          (A4_cround_ri IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   360: def: Pat<(int_hexagon_A4_cround_rr IntRegs:$src1, IntRegs:$src2),
   361:          (A4_cround_rr IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   362: def: Pat<(int_hexagon_A4_modwrapu IntRegs:$src1, IntRegs:$src2),
   363:          (A4_modwrapu IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   364: def: Pat<(int_hexagon_A4_orn IntRegs:$src1, IntRegs:$src2),
   365:          (A4_orn IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   366: def: Pat<(int_hexagon_A4_ornp DoubleRegs:$src1, DoubleRegs:$src2),
   367:          (A4_ornp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   368: def: Pat<(int_hexagon_A4_rcmpeq IntRegs:$src1, IntRegs:$src2),
   369:          (A4_rcmpeq IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   370: def: Pat<(int_hexagon_A4_rcmpeqi IntRegs:$src1, s32_0ImmPred_timm:$src2),
   371:          (A4_rcmpeqi IntRegs:$src1, s32_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   372: def: Pat<(int_hexagon_A4_rcmpneq IntRegs:$src1, IntRegs:$src2),
   373:          (A4_rcmpneq IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   374: def: Pat<(int_hexagon_A4_rcmpneqi IntRegs:$src1, s32_0ImmPred_timm:$src2),
   375:          (A4_rcmpneqi IntRegs:$src1, s32_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   376: def: Pat<(int_hexagon_A4_round_ri IntRegs:$src1, u5_0ImmPred_timm:$src2),
   377:          (A4_round_ri IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   378: def: Pat<(int_hexagon_A4_round_ri_sat IntRegs:$src1, u5_0ImmPred_timm:$src2),
   379:          (A4_round_ri_sat IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   380: def: Pat<(int_hexagon_A4_round_rr IntRegs:$src1, IntRegs:$src2),
   381:          (A4_round_rr IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   382: def: Pat<(int_hexagon_A4_round_rr_sat IntRegs:$src1, IntRegs:$src2),
   383:          (A4_round_rr_sat IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   384: def: Pat<(int_hexagon_A4_tlbmatch DoubleRegs:$src1, IntRegs:$src2),
   385:          (C2_tfrpr (A4_tlbmatch DoubleRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   386: def: Pat<(int_hexagon_A4_vcmpbeq_any DoubleRegs:$src1, DoubleRegs:$src2),
   387:          (C2_tfrpr (A4_vcmpbeq_any DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   388: def: Pat<(int_hexagon_A4_vcmpbeqi DoubleRegs:$src1, u8_0ImmPred_timm:$src2),
   389:          (C2_tfrpr (A4_vcmpbeqi DoubleRegs:$src1, u8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   390: def: Pat<(int_hexagon_A4_vcmpbgt DoubleRegs:$src1, DoubleRegs:$src2),
   391:          (C2_tfrpr (A4_vcmpbgt DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   392: def: Pat<(int_hexagon_A4_vcmpbgti DoubleRegs:$src1, s8_0ImmPred_timm:$src2),
   393:          (C2_tfrpr (A4_vcmpbgti DoubleRegs:$src1, s8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   394: def: Pat<(int_hexagon_A4_vcmpbgtui DoubleRegs:$src1, u7_0ImmPred_timm:$src2),
   395:          (C2_tfrpr (A4_vcmpbgtui DoubleRegs:$src1, u7_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   396: def: Pat<(int_hexagon_A4_vcmpheqi DoubleRegs:$src1, s8_0ImmPred_timm:$src2),
   397:          (C2_tfrpr (A4_vcmpheqi DoubleRegs:$src1, s8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   398: def: Pat<(int_hexagon_A4_vcmphgti DoubleRegs:$src1, s8_0ImmPred_timm:$src2),
   399:          (C2_tfrpr (A4_vcmphgti DoubleRegs:$src1, s8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   400: def: Pat<(int_hexagon_A4_vcmphgtui DoubleRegs:$src1, u7_0ImmPred_timm:$src2),
   401:          (C2_tfrpr (A4_vcmphgtui DoubleRegs:$src1, u7_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   402: def: Pat<(int_hexagon_A4_vcmpweqi DoubleRegs:$src1, s8_0ImmPred_timm:$src2),
   403:          (C2_tfrpr (A4_vcmpweqi DoubleRegs:$src1, s8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   404: def: Pat<(int_hexagon_A4_vcmpwgti DoubleRegs:$src1, s8_0ImmPred_timm:$src2),
   405:          (C2_tfrpr (A4_vcmpwgti DoubleRegs:$src1, s8_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   406: def: Pat<(int_hexagon_A4_vcmpwgtui DoubleRegs:$src1, u7_0ImmPred_timm:$src2),
   407:          (C2_tfrpr (A4_vcmpwgtui DoubleRegs:$src1, u7_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   408: def: Pat<(int_hexagon_A4_vrmaxh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   409:          (A4_vrmaxh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   410: def: Pat<(int_hexagon_A4_vrmaxuh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   411:          (A4_vrmaxuh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   412: def: Pat<(int_hexagon_A4_vrmaxuw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   413:          (A4_vrmaxuw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   414: def: Pat<(int_hexagon_A4_vrmaxw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   415:          (A4_vrmaxw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   416: def: Pat<(int_hexagon_A4_vrminh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   417:          (A4_vrminh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   418: def: Pat<(int_hexagon_A4_vrminuh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   419:          (A4_vrminuh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   420: def: Pat<(int_hexagon_A4_vrminuw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   421:          (A4_vrminuw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   422: def: Pat<(int_hexagon_A4_vrminw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
   423:          (A4_vrminw DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   424: def: Pat<(int_hexagon_A5_vaddhubs DoubleRegs:$src1, DoubleRegs:$src2),
   425:          (A5_vaddhubs DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   426: def: Pat<(int_hexagon_C2_all8 PredRegs:$src1),
   427:          (C2_tfrpr (C2_all8 (C2_tfrrp PredRegs:$src1)))>, Requires<[HasV5]>;
   428: def: Pat<(int_hexagon_C2_and PredRegs:$src1, PredRegs:$src2),
   429:          (C2_tfrpr (C2_and (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2)))>, Requires<[HasV5]>;
   430: def: Pat<(int_hexagon_C2_andn PredRegs:$src1, PredRegs:$src2),
   431:          (C2_tfrpr (C2_andn (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2)))>, Requires<[HasV5]>;
   432: def: Pat<(int_hexagon_C2_any8 PredRegs:$src1),
   433:          (C2_tfrpr (C2_any8 (C2_tfrrp PredRegs:$src1)))>, Requires<[HasV5]>;
   434: def: Pat<(int_hexagon_C2_bitsclr IntRegs:$src1, IntRegs:$src2),
   435:          (C2_tfrpr (C2_bitsclr IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   436: def: Pat<(int_hexagon_C2_bitsclri IntRegs:$src1, u6_0ImmPred_timm:$src2),
   437:          (C2_tfrpr (C2_bitsclri IntRegs:$src1, u6_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   438: def: Pat<(int_hexagon_C2_bitsset IntRegs:$src1, IntRegs:$src2),
   439:          (C2_tfrpr (C2_bitsset IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   440: def: Pat<(int_hexagon_C2_cmpeq IntRegs:$src1, IntRegs:$src2),
   441:          (C2_tfrpr (C2_cmpeq IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   442: def: Pat<(int_hexagon_C2_cmpeqi IntRegs:$src1, s32_0ImmPred_timm:$src2),
   443:          (C2_tfrpr (C2_cmpeqi IntRegs:$src1, s32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   444: def: Pat<(int_hexagon_C2_cmpeqp DoubleRegs:$src1, DoubleRegs:$src2),
   445:          (C2_tfrpr (C2_cmpeqp DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   446: def: Pat<(int_hexagon_C2_cmpgt IntRegs:$src1, IntRegs:$src2),
   447:          (C2_tfrpr (C2_cmpgt IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   448: def: Pat<(int_hexagon_C2_cmpgti IntRegs:$src1, s32_0ImmPred_timm:$src2),
   449:          (C2_tfrpr (C2_cmpgti IntRegs:$src1, s32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   450: def: Pat<(int_hexagon_C2_cmpgtp DoubleRegs:$src1, DoubleRegs:$src2),
   451:          (C2_tfrpr (C2_cmpgtp DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   452: def: Pat<(int_hexagon_C2_cmpgtu IntRegs:$src1, IntRegs:$src2),
   453:          (C2_tfrpr (C2_cmpgtu IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   454: def: Pat<(int_hexagon_C2_cmpgtui IntRegs:$src1, u32_0ImmPred_timm:$src2),
   455:          (C2_tfrpr (C2_cmpgtui IntRegs:$src1, u32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   456: def: Pat<(int_hexagon_C2_cmpgtup DoubleRegs:$src1, DoubleRegs:$src2),
   457:          (C2_tfrpr (C2_cmpgtup DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   458: def: Pat<(int_hexagon_C2_mask PredRegs:$src1),
   459:          (C2_mask (C2_tfrrp PredRegs:$src1))>, Requires<[HasV5]>;
   460: def: Pat<(int_hexagon_C2_mux PredRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   461:          (C2_mux (C2_tfrrp PredRegs:$src1), IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   462: def: Pat<(int_hexagon_C2_muxii PredRegs:$src1, s32_0ImmPred_timm:$src2, s8_0ImmPred_timm:$src3),
   463:          (C2_muxii (C2_tfrrp PredRegs:$src1), s32_0ImmPred_timm:$src2, s8_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
   464: def: Pat<(int_hexagon_C2_muxir PredRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3),
   465:          (C2_muxir (C2_tfrrp PredRegs:$src1), IntRegs:$src2, s32_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
   466: def: Pat<(int_hexagon_C2_muxri PredRegs:$src1, s32_0ImmPred_timm:$src2, IntRegs:$src3),
   467:          (C2_muxri (C2_tfrrp PredRegs:$src1), s32_0ImmPred_timm:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   468: def: Pat<(int_hexagon_C2_not PredRegs:$src1),
   469:          (C2_tfrpr (C2_not (C2_tfrrp PredRegs:$src1)))>, Requires<[HasV5]>;
   470: def: Pat<(int_hexagon_C2_or PredRegs:$src1, PredRegs:$src2),
   471:          (C2_tfrpr (C2_or (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2)))>, Requires<[HasV5]>;
   472: def: Pat<(int_hexagon_C2_orn PredRegs:$src1, PredRegs:$src2),
   473:          (C2_tfrpr (C2_orn (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2)))>, Requires<[HasV5]>;
   474: def: Pat<(int_hexagon_C2_pxfer_map PredRegs:$src1),
   475:          (C2_tfrpr (C2_pxfer_map (C2_tfrrp PredRegs:$src1)))>, Requires<[HasV5]>;
   476: def: Pat<(int_hexagon_C2_tfrpr PredRegs:$src1),
   477:          (C2_tfrpr (C2_tfrrp PredRegs:$src1))>, Requires<[HasV5]>;
   478: def: Pat<(int_hexagon_C2_tfrrp IntRegs:$src1),
   479:          (C2_tfrpr (C2_tfrrp IntRegs:$src1))>, Requires<[HasV5]>;
   480: def: Pat<(int_hexagon_C2_vitpack PredRegs:$src1, PredRegs:$src2),
   481:          (C2_vitpack (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2))>, Requires<[HasV5]>;
   482: def: Pat<(int_hexagon_C2_vmux PredRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   483:          (C2_vmux (C2_tfrrp PredRegs:$src1), DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   484: def: Pat<(int_hexagon_C2_xor PredRegs:$src1, PredRegs:$src2),
   485:          (C2_tfrpr (C2_xor (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2)))>, Requires<[HasV5]>;
   486: def: Pat<(int_hexagon_C4_and_and PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   487:          (C2_tfrpr (C4_and_and (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   488: def: Pat<(int_hexagon_C4_and_andn PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   489:          (C2_tfrpr (C4_and_andn (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   490: def: Pat<(int_hexagon_C4_and_or PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   491:          (C2_tfrpr (C4_and_or (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   492: def: Pat<(int_hexagon_C4_and_orn PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   493:          (C2_tfrpr (C4_and_orn (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   494: def: Pat<(int_hexagon_C4_cmplte IntRegs:$src1, IntRegs:$src2),
   495:          (C2_tfrpr (C4_cmplte IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   496: def: Pat<(int_hexagon_C4_cmpltei IntRegs:$src1, s32_0ImmPred_timm:$src2),
   497:          (C2_tfrpr (C4_cmpltei IntRegs:$src1, s32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   498: def: Pat<(int_hexagon_C4_cmplteu IntRegs:$src1, IntRegs:$src2),
   499:          (C2_tfrpr (C4_cmplteu IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   500: def: Pat<(int_hexagon_C4_cmplteui IntRegs:$src1, u32_0ImmPred_timm:$src2),
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 501-750 / 第 501-750 行

```tablegen
   501:          (C2_tfrpr (C4_cmplteui IntRegs:$src1, u32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   502: def: Pat<(int_hexagon_C4_cmpneq IntRegs:$src1, IntRegs:$src2),
   503:          (C2_tfrpr (C4_cmpneq IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   504: def: Pat<(int_hexagon_C4_cmpneqi IntRegs:$src1, s32_0ImmPred_timm:$src2),
   505:          (C2_tfrpr (C4_cmpneqi IntRegs:$src1, s32_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   506: def: Pat<(int_hexagon_C4_fastcorner9 PredRegs:$src1, PredRegs:$src2),
   507:          (C2_tfrpr (C4_fastcorner9 (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2)))>, Requires<[HasV5]>;
   508: def: Pat<(int_hexagon_C4_fastcorner9_not PredRegs:$src1, PredRegs:$src2),
   509:          (C2_tfrpr (C4_fastcorner9_not (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2)))>, Requires<[HasV5]>;
   510: def: Pat<(int_hexagon_C4_nbitsclr IntRegs:$src1, IntRegs:$src2),
   511:          (C2_tfrpr (C4_nbitsclr IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   512: def: Pat<(int_hexagon_C4_nbitsclri IntRegs:$src1, u6_0ImmPred_timm:$src2),
   513:          (C2_tfrpr (C4_nbitsclri IntRegs:$src1, u6_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   514: def: Pat<(int_hexagon_C4_nbitsset IntRegs:$src1, IntRegs:$src2),
   515:          (C2_tfrpr (C4_nbitsset IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   516: def: Pat<(int_hexagon_C4_or_and PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   517:          (C2_tfrpr (C4_or_and (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   518: def: Pat<(int_hexagon_C4_or_andn PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   519:          (C2_tfrpr (C4_or_andn (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   520: def: Pat<(int_hexagon_C4_or_or PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   521:          (C2_tfrpr (C4_or_or (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   522: def: Pat<(int_hexagon_C4_or_orn PredRegs:$src1, PredRegs:$src2, PredRegs:$src3),
   523:          (C2_tfrpr (C4_or_orn (C2_tfrrp PredRegs:$src1), (C2_tfrrp PredRegs:$src2), (C2_tfrrp PredRegs:$src3)))>, Requires<[HasV5]>;
   524: def: Pat<(int_hexagon_F2_conv_d2df DoubleRegs:$src1),
   525:          (F2_conv_d2df DoubleRegs:$src1)>, Requires<[HasV5]>;
   526: def: Pat<(int_hexagon_F2_conv_d2sf DoubleRegs:$src1),
   527:          (F2_conv_d2sf DoubleRegs:$src1)>, Requires<[HasV5]>;
   528: def: Pat<(int_hexagon_F2_conv_df2d DoubleRegs:$src1),
   529:          (F2_conv_df2d DoubleRegs:$src1)>, Requires<[HasV5]>;
   530: def: Pat<(int_hexagon_F2_conv_df2d_chop DoubleRegs:$src1),
   531:          (F2_conv_df2d_chop DoubleRegs:$src1)>, Requires<[HasV5]>;
   532: def: Pat<(int_hexagon_F2_conv_df2sf DoubleRegs:$src1),
   533:          (F2_conv_df2sf DoubleRegs:$src1)>, Requires<[HasV5]>;
   534: def: Pat<(int_hexagon_F2_conv_df2ud DoubleRegs:$src1),
   535:          (F2_conv_df2ud DoubleRegs:$src1)>, Requires<[HasV5]>;
   536: def: Pat<(int_hexagon_F2_conv_df2ud_chop DoubleRegs:$src1),
   537:          (F2_conv_df2ud_chop DoubleRegs:$src1)>, Requires<[HasV5]>;
   538: def: Pat<(int_hexagon_F2_conv_df2uw DoubleRegs:$src1),
   539:          (F2_conv_df2uw DoubleRegs:$src1)>, Requires<[HasV5]>;
   540: def: Pat<(int_hexagon_F2_conv_df2uw_chop DoubleRegs:$src1),
   541:          (F2_conv_df2uw_chop DoubleRegs:$src1)>, Requires<[HasV5]>;
   542: def: Pat<(int_hexagon_F2_conv_df2w DoubleRegs:$src1),
   543:          (F2_conv_df2w DoubleRegs:$src1)>, Requires<[HasV5]>;
   544: def: Pat<(int_hexagon_F2_conv_df2w_chop DoubleRegs:$src1),
   545:          (F2_conv_df2w_chop DoubleRegs:$src1)>, Requires<[HasV5]>;
   546: def: Pat<(int_hexagon_F2_conv_sf2d IntRegs:$src1),
   547:          (F2_conv_sf2d IntRegs:$src1)>, Requires<[HasV5]>;
   548: def: Pat<(int_hexagon_F2_conv_sf2d_chop IntRegs:$src1),
   549:          (F2_conv_sf2d_chop IntRegs:$src1)>, Requires<[HasV5]>;
   550: def: Pat<(int_hexagon_F2_conv_sf2df IntRegs:$src1),
   551:          (F2_conv_sf2df IntRegs:$src1)>, Requires<[HasV5]>;
   552: def: Pat<(int_hexagon_F2_conv_sf2ud IntRegs:$src1),
   553:          (F2_conv_sf2ud IntRegs:$src1)>, Requires<[HasV5]>;
   554: def: Pat<(int_hexagon_F2_conv_sf2ud_chop IntRegs:$src1),
   555:          (F2_conv_sf2ud_chop IntRegs:$src1)>, Requires<[HasV5]>;
   556: def: Pat<(int_hexagon_F2_conv_sf2uw IntRegs:$src1),
   557:          (F2_conv_sf2uw IntRegs:$src1)>, Requires<[HasV5]>;
   558: def: Pat<(int_hexagon_F2_conv_sf2uw_chop IntRegs:$src1),
   559:          (F2_conv_sf2uw_chop IntRegs:$src1)>, Requires<[HasV5]>;
   560: def: Pat<(int_hexagon_F2_conv_sf2w IntRegs:$src1),
   561:          (F2_conv_sf2w IntRegs:$src1)>, Requires<[HasV5]>;
   562: def: Pat<(int_hexagon_F2_conv_sf2w_chop IntRegs:$src1),
   563:          (F2_conv_sf2w_chop IntRegs:$src1)>, Requires<[HasV5]>;
   564: def: Pat<(int_hexagon_F2_conv_ud2df DoubleRegs:$src1),
   565:          (F2_conv_ud2df DoubleRegs:$src1)>, Requires<[HasV5]>;
   566: def: Pat<(int_hexagon_F2_conv_ud2sf DoubleRegs:$src1),
   567:          (F2_conv_ud2sf DoubleRegs:$src1)>, Requires<[HasV5]>;
   568: def: Pat<(int_hexagon_F2_conv_uw2df IntRegs:$src1),
   569:          (F2_conv_uw2df IntRegs:$src1)>, Requires<[HasV5]>;
   570: def: Pat<(int_hexagon_F2_conv_uw2sf IntRegs:$src1),
   571:          (F2_conv_uw2sf IntRegs:$src1)>, Requires<[HasV5]>;
   572: def: Pat<(int_hexagon_F2_conv_w2df IntRegs:$src1),
   573:          (F2_conv_w2df IntRegs:$src1)>, Requires<[HasV5]>;
   574: def: Pat<(int_hexagon_F2_conv_w2sf IntRegs:$src1),
   575:          (F2_conv_w2sf IntRegs:$src1)>, Requires<[HasV5]>;
   576: def: Pat<(int_hexagon_F2_dfclass DoubleRegs:$src1, u5_0ImmPred_timm:$src2),
   577:          (C2_tfrpr (F2_dfclass DoubleRegs:$src1, u5_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   578: def: Pat<(int_hexagon_F2_dfcmpeq DoubleRegs:$src1, DoubleRegs:$src2),
   579:          (C2_tfrpr (F2_dfcmpeq DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   580: def: Pat<(int_hexagon_F2_dfcmpge DoubleRegs:$src1, DoubleRegs:$src2),
   581:          (C2_tfrpr (F2_dfcmpge DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   582: def: Pat<(int_hexagon_F2_dfcmpgt DoubleRegs:$src1, DoubleRegs:$src2),
   583:          (C2_tfrpr (F2_dfcmpgt DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   584: def: Pat<(int_hexagon_F2_dfcmpuo DoubleRegs:$src1, DoubleRegs:$src2),
   585:          (C2_tfrpr (F2_dfcmpuo DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV5]>;
   586: def: Pat<(int_hexagon_F2_dfimm_n u10_0ImmPred_timm:$src1),
   587:          (F2_dfimm_n u10_0ImmPred_timm:$src1)>, Requires<[HasV5]>;
   588: def: Pat<(int_hexagon_F2_dfimm_p u10_0ImmPred_timm:$src1),
   589:          (F2_dfimm_p u10_0ImmPred_timm:$src1)>, Requires<[HasV5]>;
   590: def: Pat<(int_hexagon_F2_sfadd IntRegs:$src1, IntRegs:$src2),
   591:          (F2_sfadd IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   592: def: Pat<(int_hexagon_F2_sfclass IntRegs:$src1, u5_0ImmPred_timm:$src2),
   593:          (C2_tfrpr (F2_sfclass IntRegs:$src1, u5_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
   594: def: Pat<(int_hexagon_F2_sfcmpeq IntRegs:$src1, IntRegs:$src2),
   595:          (C2_tfrpr (F2_sfcmpeq IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   596: def: Pat<(int_hexagon_F2_sfcmpge IntRegs:$src1, IntRegs:$src2),
   597:          (C2_tfrpr (F2_sfcmpge IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   598: def: Pat<(int_hexagon_F2_sfcmpgt IntRegs:$src1, IntRegs:$src2),
   599:          (C2_tfrpr (F2_sfcmpgt IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   600: def: Pat<(int_hexagon_F2_sfcmpuo IntRegs:$src1, IntRegs:$src2),
   601:          (C2_tfrpr (F2_sfcmpuo IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
   602: def: Pat<(int_hexagon_F2_sffixupd IntRegs:$src1, IntRegs:$src2),
   603:          (F2_sffixupd IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   604: def: Pat<(int_hexagon_F2_sffixupn IntRegs:$src1, IntRegs:$src2),
   605:          (F2_sffixupn IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   606: def: Pat<(int_hexagon_F2_sffixupr IntRegs:$src1),
   607:          (F2_sffixupr IntRegs:$src1)>, Requires<[HasV5]>;
   608: def: Pat<(int_hexagon_F2_sffma IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   609:          (F2_sffma IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   610: def: Pat<(int_hexagon_F2_sffma_lib IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   611:          (F2_sffma_lib IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   612: def: Pat<(int_hexagon_F2_sffma_sc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3, PredRegs:$src4),
   613:          (F2_sffma_sc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3, (C2_tfrrp PredRegs:$src4))>, Requires<[HasV5]>;
   614: def: Pat<(int_hexagon_F2_sffms IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   615:          (F2_sffms IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   616: def: Pat<(int_hexagon_F2_sffms_lib IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   617:          (F2_sffms_lib IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   618: def: Pat<(int_hexagon_F2_sfimm_n u10_0ImmPred_timm:$src1),
   619:          (F2_sfimm_n u10_0ImmPred_timm:$src1)>, Requires<[HasV5]>;
   620: def: Pat<(int_hexagon_F2_sfimm_p u10_0ImmPred_timm:$src1),
   621:          (F2_sfimm_p u10_0ImmPred_timm:$src1)>, Requires<[HasV5]>;
   622: def: Pat<(int_hexagon_F2_sfmax IntRegs:$src1, IntRegs:$src2),
   623:          (F2_sfmax IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   624: def: Pat<(int_hexagon_F2_sfmin IntRegs:$src1, IntRegs:$src2),
   625:          (F2_sfmin IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   626: def: Pat<(int_hexagon_F2_sfmpy IntRegs:$src1, IntRegs:$src2),
   627:          (F2_sfmpy IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   628: def: Pat<(int_hexagon_F2_sfsub IntRegs:$src1, IntRegs:$src2),
   629:          (F2_sfsub IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   630: def: Pat<(int_hexagon_M2_acci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   631:          (M2_acci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   632: def: Pat<(int_hexagon_M2_accii IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3),
   633:          (M2_accii IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
   634: def: Pat<(int_hexagon_M2_cmaci_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   635:          (M2_cmaci_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   636: def: Pat<(int_hexagon_M2_cmacr_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   637:          (M2_cmacr_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   638: def: Pat<(int_hexagon_M2_cmacs_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   639:          (M2_cmacs_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   640: def: Pat<(int_hexagon_M2_cmacs_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   641:          (M2_cmacs_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   642: def: Pat<(int_hexagon_M2_cmacsc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   643:          (M2_cmacsc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   644: def: Pat<(int_hexagon_M2_cmacsc_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   645:          (M2_cmacsc_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   646: def: Pat<(int_hexagon_M2_cmpyi_s0 IntRegs:$src1, IntRegs:$src2),
   647:          (M2_cmpyi_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   648: def: Pat<(int_hexagon_M2_cmpyr_s0 IntRegs:$src1, IntRegs:$src2),
   649:          (M2_cmpyr_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   650: def: Pat<(int_hexagon_M2_cmpyrs_s0 IntRegs:$src1, IntRegs:$src2),
   651:          (M2_cmpyrs_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   652: def: Pat<(int_hexagon_M2_cmpyrs_s1 IntRegs:$src1, IntRegs:$src2),
   653:          (M2_cmpyrs_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   654: def: Pat<(int_hexagon_M2_cmpyrsc_s0 IntRegs:$src1, IntRegs:$src2),
   655:          (M2_cmpyrsc_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   656: def: Pat<(int_hexagon_M2_cmpyrsc_s1 IntRegs:$src1, IntRegs:$src2),
   657:          (M2_cmpyrsc_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   658: def: Pat<(int_hexagon_M2_cmpys_s0 IntRegs:$src1, IntRegs:$src2),
   659:          (M2_cmpys_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   660: def: Pat<(int_hexagon_M2_cmpys_s1 IntRegs:$src1, IntRegs:$src2),
   661:          (M2_cmpys_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   662: def: Pat<(int_hexagon_M2_cmpysc_s0 IntRegs:$src1, IntRegs:$src2),
   663:          (M2_cmpysc_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   664: def: Pat<(int_hexagon_M2_cmpysc_s1 IntRegs:$src1, IntRegs:$src2),
   665:          (M2_cmpysc_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   666: def: Pat<(int_hexagon_M2_cnacs_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   667:          (M2_cnacs_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   668: def: Pat<(int_hexagon_M2_cnacs_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   669:          (M2_cnacs_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   670: def: Pat<(int_hexagon_M2_cnacsc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   671:          (M2_cnacsc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   672: def: Pat<(int_hexagon_M2_cnacsc_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   673:          (M2_cnacsc_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   674: def: Pat<(int_hexagon_M2_dpmpyss_acc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   675:          (M2_dpmpyss_acc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   676: def: Pat<(int_hexagon_M2_dpmpyss_nac_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   677:          (M2_dpmpyss_nac_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   678: def: Pat<(int_hexagon_M2_dpmpyss_rnd_s0 IntRegs:$src1, IntRegs:$src2),
   679:          (M2_dpmpyss_rnd_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   680: def: Pat<(int_hexagon_M2_dpmpyss_s0 IntRegs:$src1, IntRegs:$src2),
   681:          (M2_dpmpyss_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   682: def: Pat<(int_hexagon_M2_dpmpyuu_acc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   683:          (M2_dpmpyuu_acc_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   684: def: Pat<(int_hexagon_M2_dpmpyuu_nac_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   685:          (M2_dpmpyuu_nac_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   686: def: Pat<(int_hexagon_M2_dpmpyuu_s0 IntRegs:$src1, IntRegs:$src2),
   687:          (M2_dpmpyuu_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   688: def: Pat<(int_hexagon_M2_hmmpyh_rs1 IntRegs:$src1, IntRegs:$src2),
   689:          (M2_hmmpyh_rs1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   690: def: Pat<(int_hexagon_M2_hmmpyh_s1 IntRegs:$src1, IntRegs:$src2),
   691:          (M2_hmmpyh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   692: def: Pat<(int_hexagon_M2_hmmpyl_rs1 IntRegs:$src1, IntRegs:$src2),
   693:          (M2_hmmpyl_rs1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   694: def: Pat<(int_hexagon_M2_hmmpyl_s1 IntRegs:$src1, IntRegs:$src2),
   695:          (M2_hmmpyl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   696: def: Pat<(int_hexagon_M2_maci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   697:          (M2_maci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   698: def: Pat<(int_hexagon_M2_macsin IntRegs:$src1, IntRegs:$src2, u32_0ImmPred_timm:$src3),
   699:          (M2_macsin IntRegs:$src1, IntRegs:$src2, u32_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
   700: def: Pat<(int_hexagon_M2_macsip IntRegs:$src1, IntRegs:$src2, u32_0ImmPred_timm:$src3),
   701:          (M2_macsip IntRegs:$src1, IntRegs:$src2, u32_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
   702: def: Pat<(int_hexagon_M2_mmachs_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   703:          (M2_mmachs_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   704: def: Pat<(int_hexagon_M2_mmachs_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   705:          (M2_mmachs_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   706: def: Pat<(int_hexagon_M2_mmachs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   707:          (M2_mmachs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   708: def: Pat<(int_hexagon_M2_mmachs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   709:          (M2_mmachs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   710: def: Pat<(int_hexagon_M2_mmacls_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   711:          (M2_mmacls_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   712: def: Pat<(int_hexagon_M2_mmacls_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   713:          (M2_mmacls_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   714: def: Pat<(int_hexagon_M2_mmacls_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   715:          (M2_mmacls_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   716: def: Pat<(int_hexagon_M2_mmacls_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   717:          (M2_mmacls_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   718: def: Pat<(int_hexagon_M2_mmacuhs_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   719:          (M2_mmacuhs_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   720: def: Pat<(int_hexagon_M2_mmacuhs_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   721:          (M2_mmacuhs_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   722: def: Pat<(int_hexagon_M2_mmacuhs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   723:          (M2_mmacuhs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   724: def: Pat<(int_hexagon_M2_mmacuhs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   725:          (M2_mmacuhs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   726: def: Pat<(int_hexagon_M2_mmaculs_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   727:          (M2_mmaculs_rs0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   728: def: Pat<(int_hexagon_M2_mmaculs_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   729:          (M2_mmaculs_rs1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   730: def: Pat<(int_hexagon_M2_mmaculs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   731:          (M2_mmaculs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   732: def: Pat<(int_hexagon_M2_mmaculs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
   733:          (M2_mmaculs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
   734: def: Pat<(int_hexagon_M2_mmpyh_rs0 DoubleRegs:$src1, DoubleRegs:$src2),
   735:          (M2_mmpyh_rs0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   736: def: Pat<(int_hexagon_M2_mmpyh_rs1 DoubleRegs:$src1, DoubleRegs:$src2),
   737:          (M2_mmpyh_rs1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   738: def: Pat<(int_hexagon_M2_mmpyh_s0 DoubleRegs:$src1, DoubleRegs:$src2),
   739:          (M2_mmpyh_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   740: def: Pat<(int_hexagon_M2_mmpyh_s1 DoubleRegs:$src1, DoubleRegs:$src2),
   741:          (M2_mmpyh_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   742: def: Pat<(int_hexagon_M2_mmpyl_rs0 DoubleRegs:$src1, DoubleRegs:$src2),
   743:          (M2_mmpyl_rs0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   744: def: Pat<(int_hexagon_M2_mmpyl_rs1 DoubleRegs:$src1, DoubleRegs:$src2),
   745:          (M2_mmpyl_rs1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   746: def: Pat<(int_hexagon_M2_mmpyl_s0 DoubleRegs:$src1, DoubleRegs:$src2),
   747:          (M2_mmpyl_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   748: def: Pat<(int_hexagon_M2_mmpyl_s1 DoubleRegs:$src1, DoubleRegs:$src2),
   749:          (M2_mmpyl_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   750: def: Pat<(int_hexagon_M2_mmpyuh_rs0 DoubleRegs:$src1, DoubleRegs:$src2),
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 751-1000 / 第 751-1000 行

```tablegen
   751:          (M2_mmpyuh_rs0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   752: def: Pat<(int_hexagon_M2_mmpyuh_rs1 DoubleRegs:$src1, DoubleRegs:$src2),
   753:          (M2_mmpyuh_rs1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   754: def: Pat<(int_hexagon_M2_mmpyuh_s0 DoubleRegs:$src1, DoubleRegs:$src2),
   755:          (M2_mmpyuh_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   756: def: Pat<(int_hexagon_M2_mmpyuh_s1 DoubleRegs:$src1, DoubleRegs:$src2),
   757:          (M2_mmpyuh_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   758: def: Pat<(int_hexagon_M2_mmpyul_rs0 DoubleRegs:$src1, DoubleRegs:$src2),
   759:          (M2_mmpyul_rs0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   760: def: Pat<(int_hexagon_M2_mmpyul_rs1 DoubleRegs:$src1, DoubleRegs:$src2),
   761:          (M2_mmpyul_rs1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   762: def: Pat<(int_hexagon_M2_mmpyul_s0 DoubleRegs:$src1, DoubleRegs:$src2),
   763:          (M2_mmpyul_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   764: def: Pat<(int_hexagon_M2_mmpyul_s1 DoubleRegs:$src1, DoubleRegs:$src2),
   765:          (M2_mmpyul_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
   766: def: Pat<(int_hexagon_M2_mpy_acc_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   767:          (M2_mpy_acc_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   768: def: Pat<(int_hexagon_M2_mpy_acc_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   769:          (M2_mpy_acc_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   770: def: Pat<(int_hexagon_M2_mpy_acc_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   771:          (M2_mpy_acc_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   772: def: Pat<(int_hexagon_M2_mpy_acc_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   773:          (M2_mpy_acc_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   774: def: Pat<(int_hexagon_M2_mpy_acc_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   775:          (M2_mpy_acc_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   776: def: Pat<(int_hexagon_M2_mpy_acc_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   777:          (M2_mpy_acc_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   778: def: Pat<(int_hexagon_M2_mpy_acc_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   779:          (M2_mpy_acc_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   780: def: Pat<(int_hexagon_M2_mpy_acc_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   781:          (M2_mpy_acc_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   782: def: Pat<(int_hexagon_M2_mpy_acc_sat_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   783:          (M2_mpy_acc_sat_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   784: def: Pat<(int_hexagon_M2_mpy_acc_sat_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   785:          (M2_mpy_acc_sat_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   786: def: Pat<(int_hexagon_M2_mpy_acc_sat_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   787:          (M2_mpy_acc_sat_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   788: def: Pat<(int_hexagon_M2_mpy_acc_sat_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   789:          (M2_mpy_acc_sat_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   790: def: Pat<(int_hexagon_M2_mpy_acc_sat_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   791:          (M2_mpy_acc_sat_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   792: def: Pat<(int_hexagon_M2_mpy_acc_sat_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   793:          (M2_mpy_acc_sat_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   794: def: Pat<(int_hexagon_M2_mpy_acc_sat_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   795:          (M2_mpy_acc_sat_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   796: def: Pat<(int_hexagon_M2_mpy_acc_sat_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   797:          (M2_mpy_acc_sat_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   798: def: Pat<(int_hexagon_M2_mpy_hh_s0 IntRegs:$src1, IntRegs:$src2),
   799:          (M2_mpy_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   800: def: Pat<(int_hexagon_M2_mpy_hh_s1 IntRegs:$src1, IntRegs:$src2),
   801:          (M2_mpy_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   802: def: Pat<(int_hexagon_M2_mpy_hl_s0 IntRegs:$src1, IntRegs:$src2),
   803:          (M2_mpy_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   804: def: Pat<(int_hexagon_M2_mpy_hl_s1 IntRegs:$src1, IntRegs:$src2),
   805:          (M2_mpy_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   806: def: Pat<(int_hexagon_M2_mpy_lh_s0 IntRegs:$src1, IntRegs:$src2),
   807:          (M2_mpy_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   808: def: Pat<(int_hexagon_M2_mpy_lh_s1 IntRegs:$src1, IntRegs:$src2),
   809:          (M2_mpy_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   810: def: Pat<(int_hexagon_M2_mpy_ll_s0 IntRegs:$src1, IntRegs:$src2),
   811:          (M2_mpy_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   812: def: Pat<(int_hexagon_M2_mpy_ll_s1 IntRegs:$src1, IntRegs:$src2),
   813:          (M2_mpy_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   814: def: Pat<(int_hexagon_M2_mpy_nac_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   815:          (M2_mpy_nac_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   816: def: Pat<(int_hexagon_M2_mpy_nac_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   817:          (M2_mpy_nac_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   818: def: Pat<(int_hexagon_M2_mpy_nac_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   819:          (M2_mpy_nac_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   820: def: Pat<(int_hexagon_M2_mpy_nac_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   821:          (M2_mpy_nac_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   822: def: Pat<(int_hexagon_M2_mpy_nac_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   823:          (M2_mpy_nac_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   824: def: Pat<(int_hexagon_M2_mpy_nac_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   825:          (M2_mpy_nac_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   826: def: Pat<(int_hexagon_M2_mpy_nac_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   827:          (M2_mpy_nac_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   828: def: Pat<(int_hexagon_M2_mpy_nac_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   829:          (M2_mpy_nac_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   830: def: Pat<(int_hexagon_M2_mpy_nac_sat_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   831:          (M2_mpy_nac_sat_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   832: def: Pat<(int_hexagon_M2_mpy_nac_sat_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   833:          (M2_mpy_nac_sat_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   834: def: Pat<(int_hexagon_M2_mpy_nac_sat_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   835:          (M2_mpy_nac_sat_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   836: def: Pat<(int_hexagon_M2_mpy_nac_sat_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   837:          (M2_mpy_nac_sat_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   838: def: Pat<(int_hexagon_M2_mpy_nac_sat_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   839:          (M2_mpy_nac_sat_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   840: def: Pat<(int_hexagon_M2_mpy_nac_sat_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   841:          (M2_mpy_nac_sat_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   842: def: Pat<(int_hexagon_M2_mpy_nac_sat_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   843:          (M2_mpy_nac_sat_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   844: def: Pat<(int_hexagon_M2_mpy_nac_sat_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   845:          (M2_mpy_nac_sat_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   846: def: Pat<(int_hexagon_M2_mpy_rnd_hh_s0 IntRegs:$src1, IntRegs:$src2),
   847:          (M2_mpy_rnd_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   848: def: Pat<(int_hexagon_M2_mpy_rnd_hh_s1 IntRegs:$src1, IntRegs:$src2),
   849:          (M2_mpy_rnd_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   850: def: Pat<(int_hexagon_M2_mpy_rnd_hl_s0 IntRegs:$src1, IntRegs:$src2),
   851:          (M2_mpy_rnd_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   852: def: Pat<(int_hexagon_M2_mpy_rnd_hl_s1 IntRegs:$src1, IntRegs:$src2),
   853:          (M2_mpy_rnd_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   854: def: Pat<(int_hexagon_M2_mpy_rnd_lh_s0 IntRegs:$src1, IntRegs:$src2),
   855:          (M2_mpy_rnd_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   856: def: Pat<(int_hexagon_M2_mpy_rnd_lh_s1 IntRegs:$src1, IntRegs:$src2),
   857:          (M2_mpy_rnd_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   858: def: Pat<(int_hexagon_M2_mpy_rnd_ll_s0 IntRegs:$src1, IntRegs:$src2),
   859:          (M2_mpy_rnd_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   860: def: Pat<(int_hexagon_M2_mpy_rnd_ll_s1 IntRegs:$src1, IntRegs:$src2),
   861:          (M2_mpy_rnd_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   862: def: Pat<(int_hexagon_M2_mpy_sat_hh_s0 IntRegs:$src1, IntRegs:$src2),
   863:          (M2_mpy_sat_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   864: def: Pat<(int_hexagon_M2_mpy_sat_hh_s1 IntRegs:$src1, IntRegs:$src2),
   865:          (M2_mpy_sat_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   866: def: Pat<(int_hexagon_M2_mpy_sat_hl_s0 IntRegs:$src1, IntRegs:$src2),
   867:          (M2_mpy_sat_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   868: def: Pat<(int_hexagon_M2_mpy_sat_hl_s1 IntRegs:$src1, IntRegs:$src2),
   869:          (M2_mpy_sat_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   870: def: Pat<(int_hexagon_M2_mpy_sat_lh_s0 IntRegs:$src1, IntRegs:$src2),
   871:          (M2_mpy_sat_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   872: def: Pat<(int_hexagon_M2_mpy_sat_lh_s1 IntRegs:$src1, IntRegs:$src2),
   873:          (M2_mpy_sat_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   874: def: Pat<(int_hexagon_M2_mpy_sat_ll_s0 IntRegs:$src1, IntRegs:$src2),
   875:          (M2_mpy_sat_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   876: def: Pat<(int_hexagon_M2_mpy_sat_ll_s1 IntRegs:$src1, IntRegs:$src2),
   877:          (M2_mpy_sat_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   878: def: Pat<(int_hexagon_M2_mpy_sat_rnd_hh_s0 IntRegs:$src1, IntRegs:$src2),
   879:          (M2_mpy_sat_rnd_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   880: def: Pat<(int_hexagon_M2_mpy_sat_rnd_hh_s1 IntRegs:$src1, IntRegs:$src2),
   881:          (M2_mpy_sat_rnd_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   882: def: Pat<(int_hexagon_M2_mpy_sat_rnd_hl_s0 IntRegs:$src1, IntRegs:$src2),
   883:          (M2_mpy_sat_rnd_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   884: def: Pat<(int_hexagon_M2_mpy_sat_rnd_hl_s1 IntRegs:$src1, IntRegs:$src2),
   885:          (M2_mpy_sat_rnd_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   886: def: Pat<(int_hexagon_M2_mpy_sat_rnd_lh_s0 IntRegs:$src1, IntRegs:$src2),
   887:          (M2_mpy_sat_rnd_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   888: def: Pat<(int_hexagon_M2_mpy_sat_rnd_lh_s1 IntRegs:$src1, IntRegs:$src2),
   889:          (M2_mpy_sat_rnd_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   890: def: Pat<(int_hexagon_M2_mpy_sat_rnd_ll_s0 IntRegs:$src1, IntRegs:$src2),
   891:          (M2_mpy_sat_rnd_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   892: def: Pat<(int_hexagon_M2_mpy_sat_rnd_ll_s1 IntRegs:$src1, IntRegs:$src2),
   893:          (M2_mpy_sat_rnd_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   894: def: Pat<(int_hexagon_M2_mpy_up IntRegs:$src1, IntRegs:$src2),
   895:          (M2_mpy_up IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   896: def: Pat<(int_hexagon_M2_mpy_up_s1 IntRegs:$src1, IntRegs:$src2),
   897:          (M2_mpy_up_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   898: def: Pat<(int_hexagon_M2_mpy_up_s1_sat IntRegs:$src1, IntRegs:$src2),
   899:          (M2_mpy_up_s1_sat IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   900: def: Pat<(int_hexagon_M2_mpyd_acc_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   901:          (M2_mpyd_acc_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   902: def: Pat<(int_hexagon_M2_mpyd_acc_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   903:          (M2_mpyd_acc_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   904: def: Pat<(int_hexagon_M2_mpyd_acc_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   905:          (M2_mpyd_acc_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   906: def: Pat<(int_hexagon_M2_mpyd_acc_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   907:          (M2_mpyd_acc_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   908: def: Pat<(int_hexagon_M2_mpyd_acc_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   909:          (M2_mpyd_acc_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   910: def: Pat<(int_hexagon_M2_mpyd_acc_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   911:          (M2_mpyd_acc_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   912: def: Pat<(int_hexagon_M2_mpyd_acc_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   913:          (M2_mpyd_acc_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   914: def: Pat<(int_hexagon_M2_mpyd_acc_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   915:          (M2_mpyd_acc_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   916: def: Pat<(int_hexagon_M2_mpyd_hh_s0 IntRegs:$src1, IntRegs:$src2),
   917:          (M2_mpyd_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   918: def: Pat<(int_hexagon_M2_mpyd_hh_s1 IntRegs:$src1, IntRegs:$src2),
   919:          (M2_mpyd_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   920: def: Pat<(int_hexagon_M2_mpyd_hl_s0 IntRegs:$src1, IntRegs:$src2),
   921:          (M2_mpyd_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   922: def: Pat<(int_hexagon_M2_mpyd_hl_s1 IntRegs:$src1, IntRegs:$src2),
   923:          (M2_mpyd_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   924: def: Pat<(int_hexagon_M2_mpyd_lh_s0 IntRegs:$src1, IntRegs:$src2),
   925:          (M2_mpyd_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   926: def: Pat<(int_hexagon_M2_mpyd_lh_s1 IntRegs:$src1, IntRegs:$src2),
   927:          (M2_mpyd_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   928: def: Pat<(int_hexagon_M2_mpyd_ll_s0 IntRegs:$src1, IntRegs:$src2),
   929:          (M2_mpyd_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   930: def: Pat<(int_hexagon_M2_mpyd_ll_s1 IntRegs:$src1, IntRegs:$src2),
   931:          (M2_mpyd_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   932: def: Pat<(int_hexagon_M2_mpyd_nac_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   933:          (M2_mpyd_nac_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   934: def: Pat<(int_hexagon_M2_mpyd_nac_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   935:          (M2_mpyd_nac_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   936: def: Pat<(int_hexagon_M2_mpyd_nac_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   937:          (M2_mpyd_nac_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   938: def: Pat<(int_hexagon_M2_mpyd_nac_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   939:          (M2_mpyd_nac_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   940: def: Pat<(int_hexagon_M2_mpyd_nac_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   941:          (M2_mpyd_nac_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   942: def: Pat<(int_hexagon_M2_mpyd_nac_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   943:          (M2_mpyd_nac_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   944: def: Pat<(int_hexagon_M2_mpyd_nac_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   945:          (M2_mpyd_nac_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   946: def: Pat<(int_hexagon_M2_mpyd_nac_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   947:          (M2_mpyd_nac_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   948: def: Pat<(int_hexagon_M2_mpyd_rnd_hh_s0 IntRegs:$src1, IntRegs:$src2),
   949:          (M2_mpyd_rnd_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   950: def: Pat<(int_hexagon_M2_mpyd_rnd_hh_s1 IntRegs:$src1, IntRegs:$src2),
   951:          (M2_mpyd_rnd_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   952: def: Pat<(int_hexagon_M2_mpyd_rnd_hl_s0 IntRegs:$src1, IntRegs:$src2),
   953:          (M2_mpyd_rnd_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   954: def: Pat<(int_hexagon_M2_mpyd_rnd_hl_s1 IntRegs:$src1, IntRegs:$src2),
   955:          (M2_mpyd_rnd_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   956: def: Pat<(int_hexagon_M2_mpyd_rnd_lh_s0 IntRegs:$src1, IntRegs:$src2),
   957:          (M2_mpyd_rnd_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   958: def: Pat<(int_hexagon_M2_mpyd_rnd_lh_s1 IntRegs:$src1, IntRegs:$src2),
   959:          (M2_mpyd_rnd_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   960: def: Pat<(int_hexagon_M2_mpyd_rnd_ll_s0 IntRegs:$src1, IntRegs:$src2),
   961:          (M2_mpyd_rnd_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   962: def: Pat<(int_hexagon_M2_mpyd_rnd_ll_s1 IntRegs:$src1, IntRegs:$src2),
   963:          (M2_mpyd_rnd_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   964: def: Pat<(int_hexagon_M2_mpyi IntRegs:$src1, IntRegs:$src2),
   965:          (M2_mpyi IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   966: def: Pat<(int_hexagon_M2_mpysmi IntRegs:$src1, m32_0ImmPred_timm:$src2),
   967:          (M2_mpysmi IntRegs:$src1, m32_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
   968: def: Pat<(int_hexagon_M2_mpysu_up IntRegs:$src1, IntRegs:$src2),
   969:          (M2_mpysu_up IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   970: def: Pat<(int_hexagon_M2_mpyu_acc_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   971:          (M2_mpyu_acc_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   972: def: Pat<(int_hexagon_M2_mpyu_acc_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   973:          (M2_mpyu_acc_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   974: def: Pat<(int_hexagon_M2_mpyu_acc_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   975:          (M2_mpyu_acc_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   976: def: Pat<(int_hexagon_M2_mpyu_acc_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   977:          (M2_mpyu_acc_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   978: def: Pat<(int_hexagon_M2_mpyu_acc_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   979:          (M2_mpyu_acc_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   980: def: Pat<(int_hexagon_M2_mpyu_acc_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   981:          (M2_mpyu_acc_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   982: def: Pat<(int_hexagon_M2_mpyu_acc_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   983:          (M2_mpyu_acc_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   984: def: Pat<(int_hexagon_M2_mpyu_acc_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
   985:          (M2_mpyu_acc_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
   986: def: Pat<(int_hexagon_M2_mpyu_hh_s0 IntRegs:$src1, IntRegs:$src2),
   987:          (M2_mpyu_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   988: def: Pat<(int_hexagon_M2_mpyu_hh_s1 IntRegs:$src1, IntRegs:$src2),
   989:          (M2_mpyu_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   990: def: Pat<(int_hexagon_M2_mpyu_hl_s0 IntRegs:$src1, IntRegs:$src2),
   991:          (M2_mpyu_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   992: def: Pat<(int_hexagon_M2_mpyu_hl_s1 IntRegs:$src1, IntRegs:$src2),
   993:          (M2_mpyu_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   994: def: Pat<(int_hexagon_M2_mpyu_lh_s0 IntRegs:$src1, IntRegs:$src2),
   995:          (M2_mpyu_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   996: def: Pat<(int_hexagon_M2_mpyu_lh_s1 IntRegs:$src1, IntRegs:$src2),
   997:          (M2_mpyu_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
   998: def: Pat<(int_hexagon_M2_mpyu_ll_s0 IntRegs:$src1, IntRegs:$src2),
   999:          (M2_mpyu_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1000: def: Pat<(int_hexagon_M2_mpyu_ll_s1 IntRegs:$src1, IntRegs:$src2),
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1001-1250 / 第 1001-1250 行

```tablegen
  1001:          (M2_mpyu_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1002: def: Pat<(int_hexagon_M2_mpyu_nac_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1003:          (M2_mpyu_nac_hh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1004: def: Pat<(int_hexagon_M2_mpyu_nac_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1005:          (M2_mpyu_nac_hh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1006: def: Pat<(int_hexagon_M2_mpyu_nac_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1007:          (M2_mpyu_nac_hl_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1008: def: Pat<(int_hexagon_M2_mpyu_nac_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1009:          (M2_mpyu_nac_hl_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1010: def: Pat<(int_hexagon_M2_mpyu_nac_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1011:          (M2_mpyu_nac_lh_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1012: def: Pat<(int_hexagon_M2_mpyu_nac_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1013:          (M2_mpyu_nac_lh_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1014: def: Pat<(int_hexagon_M2_mpyu_nac_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1015:          (M2_mpyu_nac_ll_s0 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1016: def: Pat<(int_hexagon_M2_mpyu_nac_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1017:          (M2_mpyu_nac_ll_s1 IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1018: def: Pat<(int_hexagon_M2_mpyu_up IntRegs:$src1, IntRegs:$src2),
  1019:          (M2_mpyu_up IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1020: def: Pat<(int_hexagon_M2_mpyud_acc_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1021:          (M2_mpyud_acc_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1022: def: Pat<(int_hexagon_M2_mpyud_acc_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1023:          (M2_mpyud_acc_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1024: def: Pat<(int_hexagon_M2_mpyud_acc_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1025:          (M2_mpyud_acc_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1026: def: Pat<(int_hexagon_M2_mpyud_acc_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1027:          (M2_mpyud_acc_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1028: def: Pat<(int_hexagon_M2_mpyud_acc_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1029:          (M2_mpyud_acc_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1030: def: Pat<(int_hexagon_M2_mpyud_acc_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1031:          (M2_mpyud_acc_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1032: def: Pat<(int_hexagon_M2_mpyud_acc_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1033:          (M2_mpyud_acc_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1034: def: Pat<(int_hexagon_M2_mpyud_acc_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1035:          (M2_mpyud_acc_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1036: def: Pat<(int_hexagon_M2_mpyud_hh_s0 IntRegs:$src1, IntRegs:$src2),
  1037:          (M2_mpyud_hh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1038: def: Pat<(int_hexagon_M2_mpyud_hh_s1 IntRegs:$src1, IntRegs:$src2),
  1039:          (M2_mpyud_hh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1040: def: Pat<(int_hexagon_M2_mpyud_hl_s0 IntRegs:$src1, IntRegs:$src2),
  1041:          (M2_mpyud_hl_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1042: def: Pat<(int_hexagon_M2_mpyud_hl_s1 IntRegs:$src1, IntRegs:$src2),
  1043:          (M2_mpyud_hl_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1044: def: Pat<(int_hexagon_M2_mpyud_lh_s0 IntRegs:$src1, IntRegs:$src2),
  1045:          (M2_mpyud_lh_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1046: def: Pat<(int_hexagon_M2_mpyud_lh_s1 IntRegs:$src1, IntRegs:$src2),
  1047:          (M2_mpyud_lh_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1048: def: Pat<(int_hexagon_M2_mpyud_ll_s0 IntRegs:$src1, IntRegs:$src2),
  1049:          (M2_mpyud_ll_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1050: def: Pat<(int_hexagon_M2_mpyud_ll_s1 IntRegs:$src1, IntRegs:$src2),
  1051:          (M2_mpyud_ll_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1052: def: Pat<(int_hexagon_M2_mpyud_nac_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1053:          (M2_mpyud_nac_hh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1054: def: Pat<(int_hexagon_M2_mpyud_nac_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1055:          (M2_mpyud_nac_hh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1056: def: Pat<(int_hexagon_M2_mpyud_nac_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1057:          (M2_mpyud_nac_hl_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1058: def: Pat<(int_hexagon_M2_mpyud_nac_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1059:          (M2_mpyud_nac_hl_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1060: def: Pat<(int_hexagon_M2_mpyud_nac_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1061:          (M2_mpyud_nac_lh_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1062: def: Pat<(int_hexagon_M2_mpyud_nac_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1063:          (M2_mpyud_nac_lh_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1064: def: Pat<(int_hexagon_M2_mpyud_nac_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1065:          (M2_mpyud_nac_ll_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1066: def: Pat<(int_hexagon_M2_mpyud_nac_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1067:          (M2_mpyud_nac_ll_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1068: def: Pat<(int_hexagon_M2_mpyui IntRegs:$src1, IntRegs:$src2),
  1069:          (M2_mpyui IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1070: def: Pat<(int_hexagon_M2_nacci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1071:          (M2_nacci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1072: def: Pat<(int_hexagon_M2_naccii IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3),
  1073:          (M2_naccii IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1074: def: Pat<(int_hexagon_M2_subacc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1075:          (M2_subacc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1076: def: Pat<(int_hexagon_M2_vabsdiffh DoubleRegs:$src1, DoubleRegs:$src2),
  1077:          (M2_vabsdiffh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1078: def: Pat<(int_hexagon_M2_vabsdiffw DoubleRegs:$src1, DoubleRegs:$src2),
  1079:          (M2_vabsdiffw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1080: def: Pat<(int_hexagon_M2_vcmac_s0_sat_i DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1081:          (M2_vcmac_s0_sat_i DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1082: def: Pat<(int_hexagon_M2_vcmac_s0_sat_r DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1083:          (M2_vcmac_s0_sat_r DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1084: def: Pat<(int_hexagon_M2_vcmpy_s0_sat_i DoubleRegs:$src1, DoubleRegs:$src2),
  1085:          (M2_vcmpy_s0_sat_i DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1086: def: Pat<(int_hexagon_M2_vcmpy_s0_sat_r DoubleRegs:$src1, DoubleRegs:$src2),
  1087:          (M2_vcmpy_s0_sat_r DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1088: def: Pat<(int_hexagon_M2_vcmpy_s1_sat_i DoubleRegs:$src1, DoubleRegs:$src2),
  1089:          (M2_vcmpy_s1_sat_i DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1090: def: Pat<(int_hexagon_M2_vcmpy_s1_sat_r DoubleRegs:$src1, DoubleRegs:$src2),
  1091:          (M2_vcmpy_s1_sat_r DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1092: def: Pat<(int_hexagon_M2_vdmacs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1093:          (M2_vdmacs_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1094: def: Pat<(int_hexagon_M2_vdmacs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1095:          (M2_vdmacs_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1096: def: Pat<(int_hexagon_M2_vdmpyrs_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1097:          (M2_vdmpyrs_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1098: def: Pat<(int_hexagon_M2_vdmpyrs_s1 DoubleRegs:$src1, DoubleRegs:$src2),
  1099:          (M2_vdmpyrs_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1100: def: Pat<(int_hexagon_M2_vdmpys_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1101:          (M2_vdmpys_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1102: def: Pat<(int_hexagon_M2_vdmpys_s1 DoubleRegs:$src1, DoubleRegs:$src2),
  1103:          (M2_vdmpys_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1104: def: Pat<(int_hexagon_M2_vmac2 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1105:          (M2_vmac2 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1106: def: Pat<(int_hexagon_M2_vmac2es DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1107:          (M2_vmac2es DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1108: def: Pat<(int_hexagon_M2_vmac2es_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1109:          (M2_vmac2es_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1110: def: Pat<(int_hexagon_M2_vmac2es_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1111:          (M2_vmac2es_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1112: def: Pat<(int_hexagon_M2_vmac2s_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1113:          (M2_vmac2s_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1114: def: Pat<(int_hexagon_M2_vmac2s_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1115:          (M2_vmac2s_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1116: def: Pat<(int_hexagon_M2_vmac2su_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1117:          (M2_vmac2su_s0 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1118: def: Pat<(int_hexagon_M2_vmac2su_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1119:          (M2_vmac2su_s1 DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1120: def: Pat<(int_hexagon_M2_vmpy2es_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1121:          (M2_vmpy2es_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1122: def: Pat<(int_hexagon_M2_vmpy2es_s1 DoubleRegs:$src1, DoubleRegs:$src2),
  1123:          (M2_vmpy2es_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1124: def: Pat<(int_hexagon_M2_vmpy2s_s0 IntRegs:$src1, IntRegs:$src2),
  1125:          (M2_vmpy2s_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1126: def: Pat<(int_hexagon_M2_vmpy2s_s0pack IntRegs:$src1, IntRegs:$src2),
  1127:          (M2_vmpy2s_s0pack IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1128: def: Pat<(int_hexagon_M2_vmpy2s_s1 IntRegs:$src1, IntRegs:$src2),
  1129:          (M2_vmpy2s_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1130: def: Pat<(int_hexagon_M2_vmpy2s_s1pack IntRegs:$src1, IntRegs:$src2),
  1131:          (M2_vmpy2s_s1pack IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1132: def: Pat<(int_hexagon_M2_vmpy2su_s0 IntRegs:$src1, IntRegs:$src2),
  1133:          (M2_vmpy2su_s0 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1134: def: Pat<(int_hexagon_M2_vmpy2su_s1 IntRegs:$src1, IntRegs:$src2),
  1135:          (M2_vmpy2su_s1 IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1136: def: Pat<(int_hexagon_M2_vraddh DoubleRegs:$src1, DoubleRegs:$src2),
  1137:          (M2_vraddh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1138: def: Pat<(int_hexagon_M2_vradduh DoubleRegs:$src1, DoubleRegs:$src2),
  1139:          (M2_vradduh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1140: def: Pat<(int_hexagon_M2_vrcmaci_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1141:          (M2_vrcmaci_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1142: def: Pat<(int_hexagon_M2_vrcmaci_s0c DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1143:          (M2_vrcmaci_s0c DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1144: def: Pat<(int_hexagon_M2_vrcmacr_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1145:          (M2_vrcmacr_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1146: def: Pat<(int_hexagon_M2_vrcmacr_s0c DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1147:          (M2_vrcmacr_s0c DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1148: def: Pat<(int_hexagon_M2_vrcmpyi_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1149:          (M2_vrcmpyi_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1150: def: Pat<(int_hexagon_M2_vrcmpyi_s0c DoubleRegs:$src1, DoubleRegs:$src2),
  1151:          (M2_vrcmpyi_s0c DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1152: def: Pat<(int_hexagon_M2_vrcmpyr_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1153:          (M2_vrcmpyr_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1154: def: Pat<(int_hexagon_M2_vrcmpyr_s0c DoubleRegs:$src1, DoubleRegs:$src2),
  1155:          (M2_vrcmpyr_s0c DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1156: def: Pat<(int_hexagon_M2_vrcmpys_acc_s1 DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1157:          (M2_vrcmpys_acc_s1 DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1158: def: Pat<(int_hexagon_M2_vrcmpys_s1 DoubleRegs:$src1, IntRegs:$src2),
  1159:          (M2_vrcmpys_s1 DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1160: def: Pat<(int_hexagon_M2_vrcmpys_s1rp DoubleRegs:$src1, IntRegs:$src2),
  1161:          (M2_vrcmpys_s1rp DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1162: def: Pat<(int_hexagon_M2_vrmac_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1163:          (M2_vrmac_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1164: def: Pat<(int_hexagon_M2_vrmpy_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1165:          (M2_vrmpy_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1166: def: Pat<(int_hexagon_M2_xor_xacc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1167:          (M2_xor_xacc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1168: def: Pat<(int_hexagon_M4_and_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1169:          (M4_and_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1170: def: Pat<(int_hexagon_M4_and_andn IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1171:          (M4_and_andn IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1172: def: Pat<(int_hexagon_M4_and_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1173:          (M4_and_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1174: def: Pat<(int_hexagon_M4_and_xor IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1175:          (M4_and_xor IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1176: def: Pat<(int_hexagon_M4_cmpyi_wh DoubleRegs:$src1, IntRegs:$src2),
  1177:          (M4_cmpyi_wh DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1178: def: Pat<(int_hexagon_M4_cmpyi_whc DoubleRegs:$src1, IntRegs:$src2),
  1179:          (M4_cmpyi_whc DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1180: def: Pat<(int_hexagon_M4_cmpyr_wh DoubleRegs:$src1, IntRegs:$src2),
  1181:          (M4_cmpyr_wh DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1182: def: Pat<(int_hexagon_M4_cmpyr_whc DoubleRegs:$src1, IntRegs:$src2),
  1183:          (M4_cmpyr_whc DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1184: def: Pat<(int_hexagon_M4_mac_up_s1_sat IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1185:          (M4_mac_up_s1_sat IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1186: def: Pat<(int_hexagon_M4_mpyri_addi u32_0ImmPred_timm:$src1, IntRegs:$src2, u6_0ImmPred_timm:$src3),
  1187:          (M4_mpyri_addi u32_0ImmPred_timm:$src1, IntRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1188: def: Pat<(int_hexagon_M4_mpyri_addr IntRegs:$src1, IntRegs:$src2, u32_0ImmPred_timm:$src3),
  1189:          (M4_mpyri_addr IntRegs:$src1, IntRegs:$src2, u32_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1190: def: Pat<(int_hexagon_M4_mpyri_addr_u2 IntRegs:$src1, u6_2ImmPred_timm:$src2, IntRegs:$src3),
  1191:          (M4_mpyri_addr_u2 IntRegs:$src1, u6_2ImmPred_timm:$src2, IntRegs:$src3)>, Requires<[UseCompound, HasV5]>;
  1192: def: Pat<(int_hexagon_M4_mpyrr_addi u32_0ImmPred_timm:$src1, IntRegs:$src2, IntRegs:$src3),
  1193:          (M4_mpyrr_addi u32_0ImmPred_timm:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[UseCompound, HasV5]>;
  1194: def: Pat<(int_hexagon_M4_mpyrr_addr IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1195:          (M4_mpyrr_addr IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[UseCompound, HasV5]>;
  1196: def: Pat<(int_hexagon_M4_nac_up_s1_sat IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1197:          (M4_nac_up_s1_sat IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1198: def: Pat<(int_hexagon_M4_or_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1199:          (M4_or_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1200: def: Pat<(int_hexagon_M4_or_andn IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1201:          (M4_or_andn IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1202: def: Pat<(int_hexagon_M4_or_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1203:          (M4_or_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1204: def: Pat<(int_hexagon_M4_or_xor IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1205:          (M4_or_xor IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1206: def: Pat<(int_hexagon_M4_pmpyw IntRegs:$src1, IntRegs:$src2),
  1207:          (M4_pmpyw IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1208: def: Pat<(int_hexagon_M4_pmpyw_acc DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1209:          (M4_pmpyw_acc DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1210: def: Pat<(int_hexagon_M4_vpmpyh IntRegs:$src1, IntRegs:$src2),
  1211:          (M4_vpmpyh IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1212: def: Pat<(int_hexagon_M4_vpmpyh_acc DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1213:          (M4_vpmpyh_acc DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1214: def: Pat<(int_hexagon_M4_vrmpyeh_acc_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1215:          (M4_vrmpyeh_acc_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1216: def: Pat<(int_hexagon_M4_vrmpyeh_acc_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1217:          (M4_vrmpyeh_acc_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1218: def: Pat<(int_hexagon_M4_vrmpyeh_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1219:          (M4_vrmpyeh_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1220: def: Pat<(int_hexagon_M4_vrmpyeh_s1 DoubleRegs:$src1, DoubleRegs:$src2),
  1221:          (M4_vrmpyeh_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1222: def: Pat<(int_hexagon_M4_vrmpyoh_acc_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1223:          (M4_vrmpyoh_acc_s0 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1224: def: Pat<(int_hexagon_M4_vrmpyoh_acc_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1225:          (M4_vrmpyoh_acc_s1 DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1226: def: Pat<(int_hexagon_M4_vrmpyoh_s0 DoubleRegs:$src1, DoubleRegs:$src2),
  1227:          (M4_vrmpyoh_s0 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1228: def: Pat<(int_hexagon_M4_vrmpyoh_s1 DoubleRegs:$src1, DoubleRegs:$src2),
  1229:          (M4_vrmpyoh_s1 DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1230: def: Pat<(int_hexagon_M4_xor_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1231:          (M4_xor_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1232: def: Pat<(int_hexagon_M4_xor_andn IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1233:          (M4_xor_andn IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1234: def: Pat<(int_hexagon_M4_xor_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1235:          (M4_xor_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1236: def: Pat<(int_hexagon_M4_xor_xacc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1237:          (M4_xor_xacc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1238: def: Pat<(int_hexagon_M5_vdmacbsu DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1239:          (M5_vdmacbsu DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1240: def: Pat<(int_hexagon_M5_vdmpybsu DoubleRegs:$src1, DoubleRegs:$src2),
  1241:          (M5_vdmpybsu DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1242: def: Pat<(int_hexagon_M5_vmacbsu DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1243:          (M5_vmacbsu DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1244: def: Pat<(int_hexagon_M5_vmacbuu DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1245:          (M5_vmacbuu DoubleRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1246: def: Pat<(int_hexagon_M5_vmpybsu IntRegs:$src1, IntRegs:$src2),
  1247:          (M5_vmpybsu IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1248: def: Pat<(int_hexagon_M5_vmpybuu IntRegs:$src1, IntRegs:$src2),
  1249:          (M5_vmpybuu IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1250: def: Pat<(int_hexagon_M5_vrmacbsu DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1251-1500 / 第 1251-1500 行

```tablegen
  1251:          (M5_vrmacbsu DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1252: def: Pat<(int_hexagon_M5_vrmacbuu DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1253:          (M5_vrmacbuu DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1254: def: Pat<(int_hexagon_M5_vrmpybsu DoubleRegs:$src1, DoubleRegs:$src2),
  1255:          (M5_vrmpybsu DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1256: def: Pat<(int_hexagon_M5_vrmpybuu DoubleRegs:$src1, DoubleRegs:$src2),
  1257:          (M5_vrmpybuu DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1258: def: Pat<(int_hexagon_S2_addasl_rrri IntRegs:$src1, IntRegs:$src2, u3_0ImmPred_timm:$src3),
  1259:          (S2_addasl_rrri IntRegs:$src1, IntRegs:$src2, u3_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1260: def: Pat<(int_hexagon_S2_asl_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2),
  1261:          (S2_asl_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1262: def: Pat<(int_hexagon_S2_asl_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1263:          (S2_asl_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1264: def: Pat<(int_hexagon_S2_asl_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1265:          (S2_asl_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1266: def: Pat<(int_hexagon_S2_asl_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1267:          (S2_asl_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1268: def: Pat<(int_hexagon_S2_asl_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1269:          (S2_asl_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1270: def: Pat<(int_hexagon_S2_asl_i_p_xacc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1271:          (S2_asl_i_p_xacc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1272: def: Pat<(int_hexagon_S2_asl_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1273:          (S2_asl_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1274: def: Pat<(int_hexagon_S2_asl_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1275:          (S2_asl_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1276: def: Pat<(int_hexagon_S2_asl_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1277:          (S2_asl_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1278: def: Pat<(int_hexagon_S2_asl_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1279:          (S2_asl_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1280: def: Pat<(int_hexagon_S2_asl_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1281:          (S2_asl_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1282: def: Pat<(int_hexagon_S2_asl_i_r_sat IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1283:          (S2_asl_i_r_sat IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1284: def: Pat<(int_hexagon_S2_asl_i_r_xacc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1285:          (S2_asl_i_r_xacc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1286: def: Pat<(int_hexagon_S2_asl_i_vh DoubleRegs:$src1, u4_0ImmPred_timm:$src2),
  1287:          (S2_asl_i_vh DoubleRegs:$src1, u4_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1288: def: Pat<(int_hexagon_S2_asl_i_vw DoubleRegs:$src1, u5_0ImmPred_timm:$src2),
  1289:          (S2_asl_i_vw DoubleRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1290: def: Pat<(int_hexagon_S2_asl_r_p DoubleRegs:$src1, IntRegs:$src2),
  1291:          (S2_asl_r_p DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1292: def: Pat<(int_hexagon_S2_asl_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1293:          (S2_asl_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1294: def: Pat<(int_hexagon_S2_asl_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1295:          (S2_asl_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1296: def: Pat<(int_hexagon_S2_asl_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1297:          (S2_asl_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1298: def: Pat<(int_hexagon_S2_asl_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1299:          (S2_asl_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1300: def: Pat<(int_hexagon_S2_asl_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1301:          (S2_asl_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1302: def: Pat<(int_hexagon_S2_asl_r_r IntRegs:$src1, IntRegs:$src2),
  1303:          (S2_asl_r_r IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1304: def: Pat<(int_hexagon_S2_asl_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1305:          (S2_asl_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1306: def: Pat<(int_hexagon_S2_asl_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1307:          (S2_asl_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1308: def: Pat<(int_hexagon_S2_asl_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1309:          (S2_asl_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1310: def: Pat<(int_hexagon_S2_asl_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1311:          (S2_asl_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1312: def: Pat<(int_hexagon_S2_asl_r_r_sat IntRegs:$src1, IntRegs:$src2),
  1313:          (S2_asl_r_r_sat IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1314: def: Pat<(int_hexagon_S2_asl_r_vh DoubleRegs:$src1, IntRegs:$src2),
  1315:          (S2_asl_r_vh DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1316: def: Pat<(int_hexagon_S2_asl_r_vw DoubleRegs:$src1, IntRegs:$src2),
  1317:          (S2_asl_r_vw DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1318: def: Pat<(int_hexagon_S2_asr_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2),
  1319:          (S2_asr_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1320: def: Pat<(int_hexagon_S2_asr_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1321:          (S2_asr_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1322: def: Pat<(int_hexagon_S2_asr_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1323:          (S2_asr_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1324: def: Pat<(int_hexagon_S2_asr_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1325:          (S2_asr_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1326: def: Pat<(int_hexagon_S2_asr_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1327:          (S2_asr_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1328: def: Pat<(int_hexagon_S2_asr_i_p_rnd DoubleRegs:$src1, u6_0ImmPred_timm:$src2),
  1329:          (S2_asr_i_p_rnd DoubleRegs:$src1, u6_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1330: def: Pat<(int_hexagon_S2_asr_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1331:          (S2_asr_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1332: def: Pat<(int_hexagon_S2_asr_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1333:          (S2_asr_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1334: def: Pat<(int_hexagon_S2_asr_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1335:          (S2_asr_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1336: def: Pat<(int_hexagon_S2_asr_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1337:          (S2_asr_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1338: def: Pat<(int_hexagon_S2_asr_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1339:          (S2_asr_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1340: def: Pat<(int_hexagon_S2_asr_i_r_rnd IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1341:          (S2_asr_i_r_rnd IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1342: def: Pat<(int_hexagon_S2_asr_i_svw_trun DoubleRegs:$src1, u5_0ImmPred_timm:$src2),
  1343:          (S2_asr_i_svw_trun DoubleRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1344: def: Pat<(int_hexagon_S2_asr_i_vh DoubleRegs:$src1, u4_0ImmPred_timm:$src2),
  1345:          (S2_asr_i_vh DoubleRegs:$src1, u4_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1346: def: Pat<(int_hexagon_S2_asr_i_vw DoubleRegs:$src1, u5_0ImmPred_timm:$src2),
  1347:          (S2_asr_i_vw DoubleRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1348: def: Pat<(int_hexagon_S2_asr_r_p DoubleRegs:$src1, IntRegs:$src2),
  1349:          (S2_asr_r_p DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1350: def: Pat<(int_hexagon_S2_asr_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1351:          (S2_asr_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1352: def: Pat<(int_hexagon_S2_asr_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1353:          (S2_asr_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1354: def: Pat<(int_hexagon_S2_asr_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1355:          (S2_asr_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1356: def: Pat<(int_hexagon_S2_asr_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1357:          (S2_asr_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1358: def: Pat<(int_hexagon_S2_asr_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1359:          (S2_asr_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1360: def: Pat<(int_hexagon_S2_asr_r_r IntRegs:$src1, IntRegs:$src2),
  1361:          (S2_asr_r_r IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1362: def: Pat<(int_hexagon_S2_asr_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1363:          (S2_asr_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1364: def: Pat<(int_hexagon_S2_asr_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1365:          (S2_asr_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1366: def: Pat<(int_hexagon_S2_asr_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1367:          (S2_asr_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1368: def: Pat<(int_hexagon_S2_asr_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1369:          (S2_asr_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1370: def: Pat<(int_hexagon_S2_asr_r_r_sat IntRegs:$src1, IntRegs:$src2),
  1371:          (S2_asr_r_r_sat IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1372: def: Pat<(int_hexagon_S2_asr_r_svw_trun DoubleRegs:$src1, IntRegs:$src2),
  1373:          (S2_asr_r_svw_trun DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1374: def: Pat<(int_hexagon_S2_asr_r_vh DoubleRegs:$src1, IntRegs:$src2),
  1375:          (S2_asr_r_vh DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1376: def: Pat<(int_hexagon_S2_asr_r_vw DoubleRegs:$src1, IntRegs:$src2),
  1377:          (S2_asr_r_vw DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1378: def: Pat<(int_hexagon_S2_brev IntRegs:$src1),
  1379:          (S2_brev IntRegs:$src1)>, Requires<[HasV5]>;
  1380: def: Pat<(int_hexagon_S2_brevp DoubleRegs:$src1),
  1381:          (S2_brevp DoubleRegs:$src1)>, Requires<[HasV5]>;
  1382: def: Pat<(int_hexagon_S2_cl0 IntRegs:$src1),
  1383:          (S2_cl0 IntRegs:$src1)>, Requires<[HasV5]>;
  1384: def: Pat<(int_hexagon_S2_cl0p DoubleRegs:$src1),
  1385:          (S2_cl0p DoubleRegs:$src1)>, Requires<[HasV5]>;
  1386: def: Pat<(int_hexagon_S2_cl1 IntRegs:$src1),
  1387:          (S2_cl1 IntRegs:$src1)>, Requires<[HasV5]>;
  1388: def: Pat<(int_hexagon_S2_cl1p DoubleRegs:$src1),
  1389:          (S2_cl1p DoubleRegs:$src1)>, Requires<[HasV5]>;
  1390: def: Pat<(int_hexagon_S2_clb IntRegs:$src1),
  1391:          (S2_clb IntRegs:$src1)>, Requires<[HasV5]>;
  1392: def: Pat<(int_hexagon_S2_clbnorm IntRegs:$src1),
  1393:          (S2_clbnorm IntRegs:$src1)>, Requires<[HasV5]>;
  1394: def: Pat<(int_hexagon_S2_clbp DoubleRegs:$src1),
  1395:          (S2_clbp DoubleRegs:$src1)>, Requires<[HasV5]>;
  1396: def: Pat<(int_hexagon_S2_clrbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1397:          (S2_clrbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1398: def: Pat<(int_hexagon_S2_clrbit_r IntRegs:$src1, IntRegs:$src2),
  1399:          (S2_clrbit_r IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1400: def: Pat<(int_hexagon_S2_ct0 IntRegs:$src1),
  1401:          (S2_ct0 IntRegs:$src1)>, Requires<[HasV5]>;
  1402: def: Pat<(int_hexagon_S2_ct0p DoubleRegs:$src1),
  1403:          (S2_ct0p DoubleRegs:$src1)>, Requires<[HasV5]>;
  1404: def: Pat<(int_hexagon_S2_ct1 IntRegs:$src1),
  1405:          (S2_ct1 IntRegs:$src1)>, Requires<[HasV5]>;
  1406: def: Pat<(int_hexagon_S2_ct1p DoubleRegs:$src1),
  1407:          (S2_ct1p DoubleRegs:$src1)>, Requires<[HasV5]>;
  1408: def: Pat<(int_hexagon_S2_deinterleave DoubleRegs:$src1),
  1409:          (S2_deinterleave DoubleRegs:$src1)>, Requires<[HasV5]>;
  1410: def: Pat<(int_hexagon_S2_extractu IntRegs:$src1, u5_0ImmPred_timm:$src2, u5_0ImmPred_timm:$src3),
  1411:          (S2_extractu IntRegs:$src1, u5_0ImmPred_timm:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1412: def: Pat<(int_hexagon_S2_extractu_rp IntRegs:$src1, DoubleRegs:$src2),
  1413:          (S2_extractu_rp IntRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1414: def: Pat<(int_hexagon_S2_extractup DoubleRegs:$src1, u6_0ImmPred_timm:$src2, u6_0ImmPred_timm:$src3),
  1415:          (S2_extractup DoubleRegs:$src1, u6_0ImmPred_timm:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1416: def: Pat<(int_hexagon_S2_extractup_rp DoubleRegs:$src1, DoubleRegs:$src2),
  1417:          (S2_extractup_rp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1418: def: Pat<(int_hexagon_S2_insert IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3, u5_0ImmPred_timm:$src4),
  1419:          (S2_insert IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3, u5_0ImmPred_timm:$src4)>, Requires<[HasV5]>;
  1420: def: Pat<(int_hexagon_S2_insert_rp IntRegs:$src1, IntRegs:$src2, DoubleRegs:$src3),
  1421:          (S2_insert_rp IntRegs:$src1, IntRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1422: def: Pat<(int_hexagon_S2_insertp DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3, u6_0ImmPred_timm:$src4),
  1423:          (S2_insertp DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3, u6_0ImmPred_timm:$src4)>, Requires<[HasV5]>;
  1424: def: Pat<(int_hexagon_S2_insertp_rp DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1425:          (S2_insertp_rp DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV5]>;
  1426: def: Pat<(int_hexagon_S2_interleave DoubleRegs:$src1),
  1427:          (S2_interleave DoubleRegs:$src1)>, Requires<[HasV5]>;
  1428: def: Pat<(int_hexagon_S2_lfsp DoubleRegs:$src1, DoubleRegs:$src2),
  1429:          (S2_lfsp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1430: def: Pat<(int_hexagon_S2_lsl_r_p DoubleRegs:$src1, IntRegs:$src2),
  1431:          (S2_lsl_r_p DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1432: def: Pat<(int_hexagon_S2_lsl_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1433:          (S2_lsl_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1434: def: Pat<(int_hexagon_S2_lsl_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1435:          (S2_lsl_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1436: def: Pat<(int_hexagon_S2_lsl_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1437:          (S2_lsl_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1438: def: Pat<(int_hexagon_S2_lsl_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1439:          (S2_lsl_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1440: def: Pat<(int_hexagon_S2_lsl_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1441:          (S2_lsl_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1442: def: Pat<(int_hexagon_S2_lsl_r_r IntRegs:$src1, IntRegs:$src2),
  1443:          (S2_lsl_r_r IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1444: def: Pat<(int_hexagon_S2_lsl_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1445:          (S2_lsl_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1446: def: Pat<(int_hexagon_S2_lsl_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1447:          (S2_lsl_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1448: def: Pat<(int_hexagon_S2_lsl_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1449:          (S2_lsl_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1450: def: Pat<(int_hexagon_S2_lsl_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1451:          (S2_lsl_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1452: def: Pat<(int_hexagon_S2_lsl_r_vh DoubleRegs:$src1, IntRegs:$src2),
  1453:          (S2_lsl_r_vh DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1454: def: Pat<(int_hexagon_S2_lsl_r_vw DoubleRegs:$src1, IntRegs:$src2),
  1455:          (S2_lsl_r_vw DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1456: def: Pat<(int_hexagon_S2_lsr_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2),
  1457:          (S2_lsr_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1458: def: Pat<(int_hexagon_S2_lsr_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1459:          (S2_lsr_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1460: def: Pat<(int_hexagon_S2_lsr_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1461:          (S2_lsr_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1462: def: Pat<(int_hexagon_S2_lsr_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1463:          (S2_lsr_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1464: def: Pat<(int_hexagon_S2_lsr_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1465:          (S2_lsr_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1466: def: Pat<(int_hexagon_S2_lsr_i_p_xacc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1467:          (S2_lsr_i_p_xacc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1468: def: Pat<(int_hexagon_S2_lsr_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1469:          (S2_lsr_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1470: def: Pat<(int_hexagon_S2_lsr_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1471:          (S2_lsr_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1472: def: Pat<(int_hexagon_S2_lsr_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1473:          (S2_lsr_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1474: def: Pat<(int_hexagon_S2_lsr_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1475:          (S2_lsr_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1476: def: Pat<(int_hexagon_S2_lsr_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1477:          (S2_lsr_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1478: def: Pat<(int_hexagon_S2_lsr_i_r_xacc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1479:          (S2_lsr_i_r_xacc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1480: def: Pat<(int_hexagon_S2_lsr_i_vh DoubleRegs:$src1, u4_0ImmPred_timm:$src2),
  1481:          (S2_lsr_i_vh DoubleRegs:$src1, u4_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1482: def: Pat<(int_hexagon_S2_lsr_i_vw DoubleRegs:$src1, u5_0ImmPred_timm:$src2),
  1483:          (S2_lsr_i_vw DoubleRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1484: def: Pat<(int_hexagon_S2_lsr_r_p DoubleRegs:$src1, IntRegs:$src2),
  1485:          (S2_lsr_r_p DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1486: def: Pat<(int_hexagon_S2_lsr_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1487:          (S2_lsr_r_p_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1488: def: Pat<(int_hexagon_S2_lsr_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1489:          (S2_lsr_r_p_and DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1490: def: Pat<(int_hexagon_S2_lsr_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1491:          (S2_lsr_r_p_nac DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1492: def: Pat<(int_hexagon_S2_lsr_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1493:          (S2_lsr_r_p_or DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1494: def: Pat<(int_hexagon_S2_lsr_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1495:          (S2_lsr_r_p_xor DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1496: def: Pat<(int_hexagon_S2_lsr_r_r IntRegs:$src1, IntRegs:$src2),
  1497:          (S2_lsr_r_r IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1498: def: Pat<(int_hexagon_S2_lsr_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1499:          (S2_lsr_r_r_acc IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1500: def: Pat<(int_hexagon_S2_lsr_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1501-1750 / 第 1501-1750 行

```tablegen
  1501:          (S2_lsr_r_r_and IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1502: def: Pat<(int_hexagon_S2_lsr_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1503:          (S2_lsr_r_r_nac IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1504: def: Pat<(int_hexagon_S2_lsr_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1505:          (S2_lsr_r_r_or IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1506: def: Pat<(int_hexagon_S2_lsr_r_vh DoubleRegs:$src1, IntRegs:$src2),
  1507:          (S2_lsr_r_vh DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1508: def: Pat<(int_hexagon_S2_lsr_r_vw DoubleRegs:$src1, IntRegs:$src2),
  1509:          (S2_lsr_r_vw DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1510: def: Pat<(int_hexagon_S2_packhl IntRegs:$src1, IntRegs:$src2),
  1511:          (S2_packhl IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1512: def: Pat<(int_hexagon_S2_parityp DoubleRegs:$src1, DoubleRegs:$src2),
  1513:          (S2_parityp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1514: def: Pat<(int_hexagon_S2_setbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1515:          (S2_setbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1516: def: Pat<(int_hexagon_S2_setbit_r IntRegs:$src1, IntRegs:$src2),
  1517:          (S2_setbit_r IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1518: def: Pat<(int_hexagon_S2_shuffeb DoubleRegs:$src1, DoubleRegs:$src2),
  1519:          (S2_shuffeb DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1520: def: Pat<(int_hexagon_S2_shuffeh DoubleRegs:$src1, DoubleRegs:$src2),
  1521:          (S2_shuffeh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1522: def: Pat<(int_hexagon_S2_shuffob DoubleRegs:$src1, DoubleRegs:$src2),
  1523:          (S2_shuffob DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1524: def: Pat<(int_hexagon_S2_shuffoh DoubleRegs:$src1, DoubleRegs:$src2),
  1525:          (S2_shuffoh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1526: def: Pat<(int_hexagon_S2_svsathb IntRegs:$src1),
  1527:          (S2_svsathb IntRegs:$src1)>, Requires<[HasV5]>;
  1528: def: Pat<(int_hexagon_S2_svsathub IntRegs:$src1),
  1529:          (S2_svsathub IntRegs:$src1)>, Requires<[HasV5]>;
  1530: def: Pat<(int_hexagon_S2_togglebit_i IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1531:          (S2_togglebit_i IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1532: def: Pat<(int_hexagon_S2_togglebit_r IntRegs:$src1, IntRegs:$src2),
  1533:          (S2_togglebit_r IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1534: def: Pat<(int_hexagon_S2_tstbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1535:          (C2_tfrpr (S2_tstbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
  1536: def: Pat<(int_hexagon_S2_tstbit_r IntRegs:$src1, IntRegs:$src2),
  1537:          (C2_tfrpr (S2_tstbit_r IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
  1538: def: Pat<(int_hexagon_S2_valignib DoubleRegs:$src1, DoubleRegs:$src2, u3_0ImmPred_timm:$src3),
  1539:          (S2_valignib DoubleRegs:$src1, DoubleRegs:$src2, u3_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1540: def: Pat<(int_hexagon_S2_valignrb DoubleRegs:$src1, DoubleRegs:$src2, PredRegs:$src3),
  1541:          (S2_valignrb DoubleRegs:$src1, DoubleRegs:$src2, (C2_tfrrp PredRegs:$src3))>, Requires<[HasV5]>;
  1542: def: Pat<(int_hexagon_S2_vcnegh DoubleRegs:$src1, IntRegs:$src2),
  1543:          (S2_vcnegh DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1544: def: Pat<(int_hexagon_S2_vcrotate DoubleRegs:$src1, IntRegs:$src2),
  1545:          (S2_vcrotate DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1546: def: Pat<(int_hexagon_S2_vrcnegh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3),
  1547:          (S2_vrcnegh DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3)>, Requires<[HasV5]>;
  1548: def: Pat<(int_hexagon_S2_vrndpackwh DoubleRegs:$src1),
  1549:          (S2_vrndpackwh DoubleRegs:$src1)>, Requires<[HasV5]>;
  1550: def: Pat<(int_hexagon_S2_vrndpackwhs DoubleRegs:$src1),
  1551:          (S2_vrndpackwhs DoubleRegs:$src1)>, Requires<[HasV5]>;
  1552: def: Pat<(int_hexagon_S2_vsathb DoubleRegs:$src1),
  1553:          (S2_vsathb DoubleRegs:$src1)>, Requires<[HasV5]>;
  1554: def: Pat<(int_hexagon_S2_vsathb_nopack DoubleRegs:$src1),
  1555:          (S2_vsathb_nopack DoubleRegs:$src1)>, Requires<[HasV5]>;
  1556: def: Pat<(int_hexagon_S2_vsathub DoubleRegs:$src1),
  1557:          (S2_vsathub DoubleRegs:$src1)>, Requires<[HasV5]>;
  1558: def: Pat<(int_hexagon_S2_vsathub_nopack DoubleRegs:$src1),
  1559:          (S2_vsathub_nopack DoubleRegs:$src1)>, Requires<[HasV5]>;
  1560: def: Pat<(int_hexagon_S2_vsatwh DoubleRegs:$src1),
  1561:          (S2_vsatwh DoubleRegs:$src1)>, Requires<[HasV5]>;
  1562: def: Pat<(int_hexagon_S2_vsatwh_nopack DoubleRegs:$src1),
  1563:          (S2_vsatwh_nopack DoubleRegs:$src1)>, Requires<[HasV5]>;
  1564: def: Pat<(int_hexagon_S2_vsatwuh DoubleRegs:$src1),
  1565:          (S2_vsatwuh DoubleRegs:$src1)>, Requires<[HasV5]>;
  1566: def: Pat<(int_hexagon_S2_vsatwuh_nopack DoubleRegs:$src1),
  1567:          (S2_vsatwuh_nopack DoubleRegs:$src1)>, Requires<[HasV5]>;
  1568: def: Pat<(int_hexagon_S2_vsplatrb IntRegs:$src1),
  1569:          (S2_vsplatrb IntRegs:$src1)>, Requires<[HasV5]>;
  1570: def: Pat<(int_hexagon_S2_vsplatrh IntRegs:$src1),
  1571:          (S2_vsplatrh IntRegs:$src1)>, Requires<[HasV5]>;
  1572: def: Pat<(int_hexagon_S2_vspliceib DoubleRegs:$src1, DoubleRegs:$src2, u3_0ImmPred_timm:$src3),
  1573:          (S2_vspliceib DoubleRegs:$src1, DoubleRegs:$src2, u3_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1574: def: Pat<(int_hexagon_S2_vsplicerb DoubleRegs:$src1, DoubleRegs:$src2, PredRegs:$src3),
  1575:          (S2_vsplicerb DoubleRegs:$src1, DoubleRegs:$src2, (C2_tfrrp PredRegs:$src3))>, Requires<[HasV5]>;
  1576: def: Pat<(int_hexagon_S2_vsxtbh IntRegs:$src1),
  1577:          (S2_vsxtbh IntRegs:$src1)>, Requires<[HasV5]>;
  1578: def: Pat<(int_hexagon_S2_vsxthw IntRegs:$src1),
  1579:          (S2_vsxthw IntRegs:$src1)>, Requires<[HasV5]>;
  1580: def: Pat<(int_hexagon_S2_vtrunehb DoubleRegs:$src1),
  1581:          (S2_vtrunehb DoubleRegs:$src1)>, Requires<[HasV5]>;
  1582: def: Pat<(int_hexagon_S2_vtrunewh DoubleRegs:$src1, DoubleRegs:$src2),
  1583:          (S2_vtrunewh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1584: def: Pat<(int_hexagon_S2_vtrunohb DoubleRegs:$src1),
  1585:          (S2_vtrunohb DoubleRegs:$src1)>, Requires<[HasV5]>;
  1586: def: Pat<(int_hexagon_S2_vtrunowh DoubleRegs:$src1, DoubleRegs:$src2),
  1587:          (S2_vtrunowh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1588: def: Pat<(int_hexagon_S2_vzxtbh IntRegs:$src1),
  1589:          (S2_vzxtbh IntRegs:$src1)>, Requires<[HasV5]>;
  1590: def: Pat<(int_hexagon_S2_vzxthw IntRegs:$src1),
  1591:          (S2_vzxthw IntRegs:$src1)>, Requires<[HasV5]>;
  1592: def: Pat<(int_hexagon_S4_addaddi IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3),
  1593:          (S4_addaddi IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1594: def: Pat<(int_hexagon_S4_addi_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1595:          (S4_addi_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1596: def: Pat<(int_hexagon_S4_addi_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1597:          (S4_addi_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1598: def: Pat<(int_hexagon_S4_andi_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1599:          (S4_andi_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1600: def: Pat<(int_hexagon_S4_andi_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1601:          (S4_andi_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1602: def: Pat<(int_hexagon_S4_clbaddi IntRegs:$src1, s6_0ImmPred_timm:$src2),
  1603:          (S4_clbaddi IntRegs:$src1, s6_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1604: def: Pat<(int_hexagon_S4_clbpaddi DoubleRegs:$src1, s6_0ImmPred_timm:$src2),
  1605:          (S4_clbpaddi DoubleRegs:$src1, s6_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1606: def: Pat<(int_hexagon_S4_clbpnorm DoubleRegs:$src1),
  1607:          (S4_clbpnorm DoubleRegs:$src1)>, Requires<[HasV5]>;
  1608: def: Pat<(int_hexagon_S4_extract IntRegs:$src1, u5_0ImmPred_timm:$src2, u5_0ImmPred_timm:$src3),
  1609:          (S4_extract IntRegs:$src1, u5_0ImmPred_timm:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1610: def: Pat<(int_hexagon_S4_extract_rp IntRegs:$src1, DoubleRegs:$src2),
  1611:          (S4_extract_rp IntRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1612: def: Pat<(int_hexagon_S4_extractp DoubleRegs:$src1, u6_0ImmPred_timm:$src2, u6_0ImmPred_timm:$src3),
  1613:          (S4_extractp DoubleRegs:$src1, u6_0ImmPred_timm:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1614: def: Pat<(int_hexagon_S4_extractp_rp DoubleRegs:$src1, DoubleRegs:$src2),
  1615:          (S4_extractp_rp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1616: def: Pat<(int_hexagon_S4_lsli s6_0ImmPred_timm:$src1, IntRegs:$src2),
  1617:          (S4_lsli s6_0ImmPred_timm:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1618: def: Pat<(int_hexagon_S4_ntstbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1619:          (C2_tfrpr (S4_ntstbit_i IntRegs:$src1, u5_0ImmPred_timm:$src2))>, Requires<[HasV5]>;
  1620: def: Pat<(int_hexagon_S4_ntstbit_r IntRegs:$src1, IntRegs:$src2),
  1621:          (C2_tfrpr (S4_ntstbit_r IntRegs:$src1, IntRegs:$src2))>, Requires<[HasV5]>;
  1622: def: Pat<(int_hexagon_S4_or_andi IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3),
  1623:          (S4_or_andi IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1624: def: Pat<(int_hexagon_S4_or_andix IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3),
  1625:          (S4_or_andix IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1626: def: Pat<(int_hexagon_S4_or_ori IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3),
  1627:          (S4_or_ori IntRegs:$src1, IntRegs:$src2, s32_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1628: def: Pat<(int_hexagon_S4_ori_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1629:          (S4_ori_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1630: def: Pat<(int_hexagon_S4_ori_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1631:          (S4_ori_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1632: def: Pat<(int_hexagon_S4_parity IntRegs:$src1, IntRegs:$src2),
  1633:          (S4_parity IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1634: def: Pat<(int_hexagon_S4_subaddi IntRegs:$src1, s32_0ImmPred_timm:$src2, IntRegs:$src3),
  1635:          (S4_subaddi IntRegs:$src1, s32_0ImmPred_timm:$src2, IntRegs:$src3)>, Requires<[UseCompound, HasV5]>;
  1636: def: Pat<(int_hexagon_S4_subi_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1637:          (S4_subi_asl_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1638: def: Pat<(int_hexagon_S4_subi_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1639:          (S4_subi_lsr_ri u32_0ImmPred_timm:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[UseCompound, HasV5]>;
  1640: def: Pat<(int_hexagon_S4_vrcrotate DoubleRegs:$src1, IntRegs:$src2, u2_0ImmPred_timm:$src3),
  1641:          (S4_vrcrotate DoubleRegs:$src1, IntRegs:$src2, u2_0ImmPred_timm:$src3)>, Requires<[HasV5]>;
  1642: def: Pat<(int_hexagon_S4_vrcrotate_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3, u2_0ImmPred_timm:$src4),
  1643:          (S4_vrcrotate_acc DoubleRegs:$src1, DoubleRegs:$src2, IntRegs:$src3, u2_0ImmPred_timm:$src4)>, Requires<[HasV5]>;
  1644: def: Pat<(int_hexagon_S4_vxaddsubh DoubleRegs:$src1, DoubleRegs:$src2),
  1645:          (S4_vxaddsubh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1646: def: Pat<(int_hexagon_S4_vxaddsubhr DoubleRegs:$src1, DoubleRegs:$src2),
  1647:          (S4_vxaddsubhr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1648: def: Pat<(int_hexagon_S4_vxaddsubw DoubleRegs:$src1, DoubleRegs:$src2),
  1649:          (S4_vxaddsubw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1650: def: Pat<(int_hexagon_S4_vxsubaddh DoubleRegs:$src1, DoubleRegs:$src2),
  1651:          (S4_vxsubaddh DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1652: def: Pat<(int_hexagon_S4_vxsubaddhr DoubleRegs:$src1, DoubleRegs:$src2),
  1653:          (S4_vxsubaddhr DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1654: def: Pat<(int_hexagon_S4_vxsubaddw DoubleRegs:$src1, DoubleRegs:$src2),
  1655:          (S4_vxsubaddw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1656: def: Pat<(int_hexagon_S5_asrhub_sat DoubleRegs:$src1, u4_0ImmPred_timm:$src2),
  1657:          (S5_asrhub_sat DoubleRegs:$src1, u4_0ImmPred_timm:$src2)>, Requires<[HasV5]>;
  1658: def: Pat<(int_hexagon_S5_popcountp DoubleRegs:$src1),
  1659:          (S5_popcountp DoubleRegs:$src1)>, Requires<[HasV5]>;
  1660: def: Pat<(int_hexagon_Y2_dccleana IntRegs:$src1),
  1661:          (Y2_dccleana IntRegs:$src1)>, Requires<[HasV5]>;
  1662: def: Pat<(int_hexagon_Y2_dccleaninva IntRegs:$src1),
  1663:          (Y2_dccleaninva IntRegs:$src1)>, Requires<[HasV5]>;
  1664: def: Pat<(int_hexagon_Y2_dcinva IntRegs:$src1),
  1665:          (Y2_dcinva IntRegs:$src1)>, Requires<[HasV5]>;
  1666: def: Pat<(int_hexagon_Y2_dczeroa IntRegs:$src1),
  1667:          (Y2_dczeroa IntRegs:$src1)>, Requires<[HasV5]>;
  1668: def: Pat<(int_hexagon_Y4_l2fetch IntRegs:$src1, IntRegs:$src2),
  1669:          (Y4_l2fetch IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV5]>;
  1670: def: Pat<(int_hexagon_Y5_l2fetch IntRegs:$src1, DoubleRegs:$src2),
  1671:          (Y5_l2fetch IntRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV5]>;
  1672: 
  1673: // V60 Scalar Instructions.
  1674: 
  1675: def: Pat<(int_hexagon_S6_rol_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2),
  1676:          (S6_rol_i_p DoubleRegs:$src1, u6_0ImmPred_timm:$src2)>, Requires<[HasV60]>;
  1677: def: Pat<(int_hexagon_S6_rol_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1678:          (S6_rol_i_p_acc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1679: def: Pat<(int_hexagon_S6_rol_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1680:          (S6_rol_i_p_and DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1681: def: Pat<(int_hexagon_S6_rol_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1682:          (S6_rol_i_p_nac DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1683: def: Pat<(int_hexagon_S6_rol_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1684:          (S6_rol_i_p_or DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1685: def: Pat<(int_hexagon_S6_rol_i_p_xacc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3),
  1686:          (S6_rol_i_p_xacc DoubleRegs:$src1, DoubleRegs:$src2, u6_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1687: def: Pat<(int_hexagon_S6_rol_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1688:          (S6_rol_i_r IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV60]>;
  1689: def: Pat<(int_hexagon_S6_rol_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1690:          (S6_rol_i_r_acc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1691: def: Pat<(int_hexagon_S6_rol_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1692:          (S6_rol_i_r_and IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1693: def: Pat<(int_hexagon_S6_rol_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1694:          (S6_rol_i_r_nac IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1695: def: Pat<(int_hexagon_S6_rol_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1696:          (S6_rol_i_r_or IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1697: def: Pat<(int_hexagon_S6_rol_i_r_xacc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3),
  1698:          (S6_rol_i_r_xacc IntRegs:$src1, IntRegs:$src2, u5_0ImmPred_timm:$src3)>, Requires<[HasV60]>;
  1699: 
  1700: // V62 Scalar Instructions.
  1701: 
  1702: def: Pat<(int_hexagon_M6_vabsdiffb DoubleRegs:$src1, DoubleRegs:$src2),
  1703:          (M6_vabsdiffb DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV62]>;
  1704: def: Pat<(int_hexagon_M6_vabsdiffub DoubleRegs:$src1, DoubleRegs:$src2),
  1705:          (M6_vabsdiffub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV62]>;
  1706: def: Pat<(int_hexagon_S6_vsplatrbp IntRegs:$src1),
  1707:          (S6_vsplatrbp IntRegs:$src1)>, Requires<[HasV62]>;
  1708: def: Pat<(int_hexagon_S6_vtrunehb_ppp DoubleRegs:$src1, DoubleRegs:$src2),
  1709:          (S6_vtrunehb_ppp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV62]>;
  1710: def: Pat<(int_hexagon_S6_vtrunohb_ppp DoubleRegs:$src1, DoubleRegs:$src2),
  1711:          (S6_vtrunohb_ppp DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV62]>;
  1712: 
  1713: // V65 Scalar Instructions.
  1714: 
  1715: def: Pat<(int_hexagon_A6_vcmpbeq_notany DoubleRegs:$src1, DoubleRegs:$src2),
  1716:          (C2_tfrpr (A6_vcmpbeq_notany DoubleRegs:$src1, DoubleRegs:$src2))>, Requires<[HasV65]>;
  1717: 
  1718: // V66 Scalar Instructions.
  1719: 
  1720: def: Pat<(int_hexagon_F2_dfadd DoubleRegs:$src1, DoubleRegs:$src2),
  1721:          (F2_dfadd DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV66]>;
  1722: def: Pat<(int_hexagon_F2_dfsub DoubleRegs:$src1, DoubleRegs:$src2),
  1723:          (F2_dfsub DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV66]>;
  1724: def: Pat<(int_hexagon_M2_mnaci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3),
  1725:          (M2_mnaci IntRegs:$src1, IntRegs:$src2, IntRegs:$src3)>, Requires<[HasV66]>;
  1726: def: Pat<(int_hexagon_S2_mask u5_0ImmPred_timm:$src1, u5_0ImmPred_timm:$src2),
  1727:          (S2_mask u5_0ImmPred_timm:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV66]>;
  1728: 
  1729: // V67 Scalar Instructions.
  1730: 
  1731: def: Pat<(int_hexagon_A7_clip IntRegs:$src1, u5_0ImmPred_timm:$src2),
  1732:          (A7_clip IntRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV67]>;
  1733: def: Pat<(int_hexagon_A7_croundd_ri DoubleRegs:$src1, u6_0ImmPred_timm:$src2),
  1734:          (A7_croundd_ri DoubleRegs:$src1, u6_0ImmPred_timm:$src2)>, Requires<[HasV67]>;
  1735: def: Pat<(int_hexagon_A7_croundd_rr DoubleRegs:$src1, IntRegs:$src2),
  1736:          (A7_croundd_rr DoubleRegs:$src1, IntRegs:$src2)>, Requires<[HasV67]>;
  1737: def: Pat<(int_hexagon_A7_vclip DoubleRegs:$src1, u5_0ImmPred_timm:$src2),
  1738:          (A7_vclip DoubleRegs:$src1, u5_0ImmPred_timm:$src2)>, Requires<[HasV67]>;
  1739: def: Pat<(int_hexagon_F2_dfmax DoubleRegs:$src1, DoubleRegs:$src2),
  1740:          (F2_dfmax DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1741: def: Pat<(int_hexagon_F2_dfmin DoubleRegs:$src1, DoubleRegs:$src2),
  1742:          (F2_dfmin DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1743: def: Pat<(int_hexagon_F2_dfmpyfix DoubleRegs:$src1, DoubleRegs:$src2),
  1744:          (F2_dfmpyfix DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1745: def: Pat<(int_hexagon_F2_dfmpyhh DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1746:          (F2_dfmpyhh DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV67]>;
  1747: def: Pat<(int_hexagon_F2_dfmpylh DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1748:          (F2_dfmpylh DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV67]>;
  1749: def: Pat<(int_hexagon_F2_dfmpyll DoubleRegs:$src1, DoubleRegs:$src2),
  1750:          (F2_dfmpyll DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 1751-2000 / 第 1751-2000 行

```tablegen
  1751: def: Pat<(int_hexagon_M7_dcmpyiw DoubleRegs:$src1, DoubleRegs:$src2),
  1752:          (M7_dcmpyiw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1753: def: Pat<(int_hexagon_M7_dcmpyiw_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1754:          (M7_dcmpyiw_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV67]>;
  1755: def: Pat<(int_hexagon_M7_dcmpyiwc DoubleRegs:$src1, DoubleRegs:$src2),
  1756:          (M7_dcmpyiwc DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1757: def: Pat<(int_hexagon_M7_dcmpyiwc_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1758:          (M7_dcmpyiwc_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV67]>;
  1759: def: Pat<(int_hexagon_M7_dcmpyrw DoubleRegs:$src1, DoubleRegs:$src2),
  1760:          (M7_dcmpyrw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1761: def: Pat<(int_hexagon_M7_dcmpyrw_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1762:          (M7_dcmpyrw_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV67]>;
  1763: def: Pat<(int_hexagon_M7_dcmpyrwc DoubleRegs:$src1, DoubleRegs:$src2),
  1764:          (M7_dcmpyrwc DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1765: def: Pat<(int_hexagon_M7_dcmpyrwc_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1766:          (M7_dcmpyrwc_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV67]>;
  1767: def: Pat<(int_hexagon_M7_vdmpy DoubleRegs:$src1, DoubleRegs:$src2),
  1768:          (M7_dcmpyrwc DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1769: def: Pat<(int_hexagon_M7_vdmpy_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3),
  1770:          (M7_dcmpyrwc_acc DoubleRegs:$src1, DoubleRegs:$src2, DoubleRegs:$src3)>, Requires<[HasV67]>;
  1771: def: Pat<(int_hexagon_M7_wcmpyiw DoubleRegs:$src1, DoubleRegs:$src2),
  1772:          (M7_wcmpyiw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1773: def: Pat<(int_hexagon_M7_wcmpyiw_rnd DoubleRegs:$src1, DoubleRegs:$src2),
  1774:          (M7_wcmpyiw_rnd DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1775: def: Pat<(int_hexagon_M7_wcmpyiwc DoubleRegs:$src1, DoubleRegs:$src2),
  1776:          (M7_wcmpyiwc DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1777: def: Pat<(int_hexagon_M7_wcmpyiwc_rnd DoubleRegs:$src1, DoubleRegs:$src2),
  1778:          (M7_wcmpyiwc_rnd DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1779: def: Pat<(int_hexagon_M7_wcmpyrw DoubleRegs:$src1, DoubleRegs:$src2),
  1780:          (M7_wcmpyrw DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1781: def: Pat<(int_hexagon_M7_wcmpyrw_rnd DoubleRegs:$src1, DoubleRegs:$src2),
  1782:          (M7_wcmpyrw_rnd DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1783: def: Pat<(int_hexagon_M7_wcmpyrwc DoubleRegs:$src1, DoubleRegs:$src2),
  1784:          (M7_wcmpyrwc DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1785: def: Pat<(int_hexagon_M7_wcmpyrwc_rnd DoubleRegs:$src1, DoubleRegs:$src2),
  1786:          (M7_wcmpyrwc_rnd DoubleRegs:$src1, DoubleRegs:$src2)>, Requires<[HasV67]>;
  1787: 
  1788: // V68 Scalar Instructions.
  1789: 
  1790: def: Pat<(int_hexagon_Y6_dmlink IntRegs:$src1, IntRegs:$src2),
  1791:          (Y6_dmlink IntRegs:$src1, IntRegs:$src2)>, Requires<[HasV68]>;
  1792: def: Pat<(int_hexagon_Y6_dmpause ),
  1793:          (Y6_dmpause )>, Requires<[HasV68]>;
  1794: def: Pat<(int_hexagon_Y6_dmpoll ),
  1795:          (Y6_dmpoll )>, Requires<[HasV68]>;
  1796: def: Pat<(int_hexagon_Y6_dmresume IntRegs:$src1),
  1797:          (Y6_dmresume IntRegs:$src1)>, Requires<[HasV68]>;
  1798: def: Pat<(int_hexagon_Y6_dmstart IntRegs:$src1),
  1799:          (Y6_dmstart IntRegs:$src1)>, Requires<[HasV68]>;
  1800: def: Pat<(int_hexagon_Y6_dmwait ),
  1801:          (Y6_dmwait )>, Requires<[HasV68]>;
  1802: 
  1803: // V60 HVX Instructions.
  1804: 
  1805: def: Pat<(int_hexagon_V6_extractw HvxVR:$src1, IntRegs:$src2),
  1806:          (V6_extractw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1807: def: Pat<(int_hexagon_V6_extractw_128B HvxVR:$src1, IntRegs:$src2),
  1808:          (V6_extractw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1809: def: Pat<(int_hexagon_V6_hi HvxWR:$src1),
  1810:          (V6_hi HvxWR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1811: def: Pat<(int_hexagon_V6_hi_128B HvxWR:$src1),
  1812:          (V6_hi HvxWR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1813: def: Pat<(int_hexagon_V6_lo HvxWR:$src1),
  1814:          (V6_lo HvxWR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1815: def: Pat<(int_hexagon_V6_lo_128B HvxWR:$src1),
  1816:          (V6_lo HvxWR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1817: def: Pat<(int_hexagon_V6_lvsplatw IntRegs:$src1),
  1818:          (V6_lvsplatw IntRegs:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1819: def: Pat<(int_hexagon_V6_lvsplatw_128B IntRegs:$src1),
  1820:          (V6_lvsplatw IntRegs:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1821: def: Pat<(int_hexagon_V6_pred_and HvxQR:$src1, HvxQR:$src2),
  1822:          (V6_pred_and HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1823: def: Pat<(int_hexagon_V6_pred_and_128B HvxQR:$src1, HvxQR:$src2),
  1824:          (V6_pred_and HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1825: def: Pat<(int_hexagon_V6_pred_and_n HvxQR:$src1, HvxQR:$src2),
  1826:          (V6_pred_and_n HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1827: def: Pat<(int_hexagon_V6_pred_and_n_128B HvxQR:$src1, HvxQR:$src2),
  1828:          (V6_pred_and_n HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1829: def: Pat<(int_hexagon_V6_pred_not HvxQR:$src1),
  1830:          (V6_pred_not HvxQR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1831: def: Pat<(int_hexagon_V6_pred_not_128B HvxQR:$src1),
  1832:          (V6_pred_not HvxQR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1833: def: Pat<(int_hexagon_V6_pred_or HvxQR:$src1, HvxQR:$src2),
  1834:          (V6_pred_or HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1835: def: Pat<(int_hexagon_V6_pred_or_128B HvxQR:$src1, HvxQR:$src2),
  1836:          (V6_pred_or HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1837: def: Pat<(int_hexagon_V6_pred_or_n HvxQR:$src1, HvxQR:$src2),
  1838:          (V6_pred_or_n HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1839: def: Pat<(int_hexagon_V6_pred_or_n_128B HvxQR:$src1, HvxQR:$src2),
  1840:          (V6_pred_or_n HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1841: def: Pat<(int_hexagon_V6_pred_scalar2 IntRegs:$src1),
  1842:          (V6_pred_scalar2 IntRegs:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1843: def: Pat<(int_hexagon_V6_pred_scalar2_128B IntRegs:$src1),
  1844:          (V6_pred_scalar2 IntRegs:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1845: def: Pat<(int_hexagon_V6_pred_xor HvxQR:$src1, HvxQR:$src2),
  1846:          (V6_pred_xor HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1847: def: Pat<(int_hexagon_V6_pred_xor_128B HvxQR:$src1, HvxQR:$src2),
  1848:          (V6_pred_xor HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1849: def: Pat<(int_hexagon_V6_vS32b_nqpred_ai HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1850:          (V6_vS32b_nqpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1851: def: Pat<(int_hexagon_V6_vS32b_nqpred_ai_128B HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1852:          (V6_vS32b_nqpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1853: def: Pat<(int_hexagon_V6_vS32b_nt_nqpred_ai HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1854:          (V6_vS32b_nt_nqpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1855: def: Pat<(int_hexagon_V6_vS32b_nt_nqpred_ai_128B HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1856:          (V6_vS32b_nt_nqpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1857: def: Pat<(int_hexagon_V6_vS32b_nt_qpred_ai HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1858:          (V6_vS32b_nt_qpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1859: def: Pat<(int_hexagon_V6_vS32b_nt_qpred_ai_128B HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1860:          (V6_vS32b_nt_qpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1861: def: Pat<(int_hexagon_V6_vS32b_qpred_ai HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1862:          (V6_vS32b_qpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1863: def: Pat<(int_hexagon_V6_vS32b_qpred_ai_128B HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
  1864:          (V6_vS32b_qpred_ai HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1865: def: Pat<(int_hexagon_V6_vabsdiffh HvxVR:$src1, HvxVR:$src2),
  1866:          (V6_vabsdiffh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1867: def: Pat<(int_hexagon_V6_vabsdiffh_128B HvxVR:$src1, HvxVR:$src2),
  1868:          (V6_vabsdiffh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1869: def: Pat<(int_hexagon_V6_vabsdiffub HvxVR:$src1, HvxVR:$src2),
  1870:          (V6_vabsdiffub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1871: def: Pat<(int_hexagon_V6_vabsdiffub_128B HvxVR:$src1, HvxVR:$src2),
  1872:          (V6_vabsdiffub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1873: def: Pat<(int_hexagon_V6_vabsdiffuh HvxVR:$src1, HvxVR:$src2),
  1874:          (V6_vabsdiffuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1875: def: Pat<(int_hexagon_V6_vabsdiffuh_128B HvxVR:$src1, HvxVR:$src2),
  1876:          (V6_vabsdiffuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1877: def: Pat<(int_hexagon_V6_vabsdiffw HvxVR:$src1, HvxVR:$src2),
  1878:          (V6_vabsdiffw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1879: def: Pat<(int_hexagon_V6_vabsdiffw_128B HvxVR:$src1, HvxVR:$src2),
  1880:          (V6_vabsdiffw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1881: def: Pat<(int_hexagon_V6_vabsh HvxVR:$src1),
  1882:          (V6_vabsh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1883: def: Pat<(int_hexagon_V6_vabsh_128B HvxVR:$src1),
  1884:          (V6_vabsh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1885: def: Pat<(int_hexagon_V6_vabsh_sat HvxVR:$src1),
  1886:          (V6_vabsh_sat HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1887: def: Pat<(int_hexagon_V6_vabsh_sat_128B HvxVR:$src1),
  1888:          (V6_vabsh_sat HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1889: def: Pat<(int_hexagon_V6_vabsw HvxVR:$src1),
  1890:          (V6_vabsw HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1891: def: Pat<(int_hexagon_V6_vabsw_128B HvxVR:$src1),
  1892:          (V6_vabsw HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1893: def: Pat<(int_hexagon_V6_vabsw_sat HvxVR:$src1),
  1894:          (V6_vabsw_sat HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  1895: def: Pat<(int_hexagon_V6_vabsw_sat_128B HvxVR:$src1),
  1896:          (V6_vabsw_sat HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  1897: def: Pat<(int_hexagon_V6_vaddb HvxVR:$src1, HvxVR:$src2),
  1898:          (V6_vaddb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1899: def: Pat<(int_hexagon_V6_vaddb_128B HvxVR:$src1, HvxVR:$src2),
  1900:          (V6_vaddb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1901: def: Pat<(int_hexagon_V6_vaddb_dv HvxWR:$src1, HvxWR:$src2),
  1902:          (V6_vaddb_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1903: def: Pat<(int_hexagon_V6_vaddb_dv_128B HvxWR:$src1, HvxWR:$src2),
  1904:          (V6_vaddb_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1905: def: Pat<(int_hexagon_V6_vaddbnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1906:          (V6_vaddbnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1907: def: Pat<(int_hexagon_V6_vaddbnq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1908:          (V6_vaddbnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1909: def: Pat<(int_hexagon_V6_vaddbq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1910:          (V6_vaddbq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1911: def: Pat<(int_hexagon_V6_vaddbq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1912:          (V6_vaddbq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1913: def: Pat<(int_hexagon_V6_vaddh HvxVR:$src1, HvxVR:$src2),
  1914:          (V6_vaddh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1915: def: Pat<(int_hexagon_V6_vaddh_128B HvxVR:$src1, HvxVR:$src2),
  1916:          (V6_vaddh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1917: def: Pat<(int_hexagon_V6_vaddh_dv HvxWR:$src1, HvxWR:$src2),
  1918:          (V6_vaddh_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1919: def: Pat<(int_hexagon_V6_vaddh_dv_128B HvxWR:$src1, HvxWR:$src2),
  1920:          (V6_vaddh_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1921: def: Pat<(int_hexagon_V6_vaddhnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1922:          (V6_vaddhnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1923: def: Pat<(int_hexagon_V6_vaddhnq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1924:          (V6_vaddhnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1925: def: Pat<(int_hexagon_V6_vaddhq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1926:          (V6_vaddhq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1927: def: Pat<(int_hexagon_V6_vaddhq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1928:          (V6_vaddhq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1929: def: Pat<(int_hexagon_V6_vaddhsat HvxVR:$src1, HvxVR:$src2),
  1930:          (V6_vaddhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1931: def: Pat<(int_hexagon_V6_vaddhsat_128B HvxVR:$src1, HvxVR:$src2),
  1932:          (V6_vaddhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1933: def: Pat<(int_hexagon_V6_vaddhsat_dv HvxWR:$src1, HvxWR:$src2),
  1934:          (V6_vaddhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1935: def: Pat<(int_hexagon_V6_vaddhsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  1936:          (V6_vaddhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1937: def: Pat<(int_hexagon_V6_vaddhw HvxVR:$src1, HvxVR:$src2),
  1938:          (V6_vaddhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1939: def: Pat<(int_hexagon_V6_vaddhw_128B HvxVR:$src1, HvxVR:$src2),
  1940:          (V6_vaddhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1941: def: Pat<(int_hexagon_V6_vaddubh HvxVR:$src1, HvxVR:$src2),
  1942:          (V6_vaddubh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1943: def: Pat<(int_hexagon_V6_vaddubh_128B HvxVR:$src1, HvxVR:$src2),
  1944:          (V6_vaddubh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1945: def: Pat<(int_hexagon_V6_vaddubsat HvxVR:$src1, HvxVR:$src2),
  1946:          (V6_vaddubsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1947: def: Pat<(int_hexagon_V6_vaddubsat_128B HvxVR:$src1, HvxVR:$src2),
  1948:          (V6_vaddubsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1949: def: Pat<(int_hexagon_V6_vaddubsat_dv HvxWR:$src1, HvxWR:$src2),
  1950:          (V6_vaddubsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1951: def: Pat<(int_hexagon_V6_vaddubsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  1952:          (V6_vaddubsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1953: def: Pat<(int_hexagon_V6_vadduhsat HvxVR:$src1, HvxVR:$src2),
  1954:          (V6_vadduhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1955: def: Pat<(int_hexagon_V6_vadduhsat_128B HvxVR:$src1, HvxVR:$src2),
  1956:          (V6_vadduhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1957: def: Pat<(int_hexagon_V6_vadduhsat_dv HvxWR:$src1, HvxWR:$src2),
  1958:          (V6_vadduhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1959: def: Pat<(int_hexagon_V6_vadduhsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  1960:          (V6_vadduhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1961: def: Pat<(int_hexagon_V6_vadduhw HvxVR:$src1, HvxVR:$src2),
  1962:          (V6_vadduhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1963: def: Pat<(int_hexagon_V6_vadduhw_128B HvxVR:$src1, HvxVR:$src2),
  1964:          (V6_vadduhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1965: def: Pat<(int_hexagon_V6_vaddw HvxVR:$src1, HvxVR:$src2),
  1966:          (V6_vaddw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1967: def: Pat<(int_hexagon_V6_vaddw_128B HvxVR:$src1, HvxVR:$src2),
  1968:          (V6_vaddw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1969: def: Pat<(int_hexagon_V6_vaddw_dv HvxWR:$src1, HvxWR:$src2),
  1970:          (V6_vaddw_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1971: def: Pat<(int_hexagon_V6_vaddw_dv_128B HvxWR:$src1, HvxWR:$src2),
  1972:          (V6_vaddw_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1973: def: Pat<(int_hexagon_V6_vaddwnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1974:          (V6_vaddwnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1975: def: Pat<(int_hexagon_V6_vaddwnq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1976:          (V6_vaddwnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1977: def: Pat<(int_hexagon_V6_vaddwq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1978:          (V6_vaddwq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1979: def: Pat<(int_hexagon_V6_vaddwq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  1980:          (V6_vaddwq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1981: def: Pat<(int_hexagon_V6_vaddwsat HvxVR:$src1, HvxVR:$src2),
  1982:          (V6_vaddwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1983: def: Pat<(int_hexagon_V6_vaddwsat_128B HvxVR:$src1, HvxVR:$src2),
  1984:          (V6_vaddwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1985: def: Pat<(int_hexagon_V6_vaddwsat_dv HvxWR:$src1, HvxWR:$src2),
  1986:          (V6_vaddwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1987: def: Pat<(int_hexagon_V6_vaddwsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  1988:          (V6_vaddwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  1989: def: Pat<(int_hexagon_V6_valignb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  1990:          (V6_valignb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1991: def: Pat<(int_hexagon_V6_valignb_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  1992:          (V6_valignb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1993: def: Pat<(int_hexagon_V6_valignbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  1994:          (V6_valignbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  1995: def: Pat<(int_hexagon_V6_valignbi_128B HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  1996:          (V6_valignbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  1997: def: Pat<(int_hexagon_V6_vand HvxVR:$src1, HvxVR:$src2),
  1998:          (V6_vand HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  1999: def: Pat<(int_hexagon_V6_vand_128B HvxVR:$src1, HvxVR:$src2),
  2000:          (V6_vand HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2001-2250 / 第 2001-2250 行

```tablegen
  2001: def: Pat<(int_hexagon_V6_vandqrt HvxQR:$src1, IntRegs:$src2),
  2002:          (V6_vandqrt HvxQR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2003: def: Pat<(int_hexagon_V6_vandqrt_128B HvxQR:$src1, IntRegs:$src2),
  2004:          (V6_vandqrt HvxQR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2005: def: Pat<(int_hexagon_V6_vandqrt_acc HvxVR:$src1, HvxQR:$src2, IntRegs:$src3),
  2006:          (V6_vandqrt_acc HvxVR:$src1, HvxQR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2007: def: Pat<(int_hexagon_V6_vandqrt_acc_128B HvxVR:$src1, HvxQR:$src2, IntRegs:$src3),
  2008:          (V6_vandqrt_acc HvxVR:$src1, HvxQR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2009: def: Pat<(int_hexagon_V6_vandvrt HvxVR:$src1, IntRegs:$src2),
  2010:          (V6_vandvrt HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2011: def: Pat<(int_hexagon_V6_vandvrt_128B HvxVR:$src1, IntRegs:$src2),
  2012:          (V6_vandvrt HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2013: def: Pat<(int_hexagon_V6_vandvrt_acc HvxQR:$src1, HvxVR:$src2, IntRegs:$src3),
  2014:          (V6_vandvrt_acc HvxQR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2015: def: Pat<(int_hexagon_V6_vandvrt_acc_128B HvxQR:$src1, HvxVR:$src2, IntRegs:$src3),
  2016:          (V6_vandvrt_acc HvxQR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2017: def: Pat<(int_hexagon_V6_vaslh HvxVR:$src1, IntRegs:$src2),
  2018:          (V6_vaslh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2019: def: Pat<(int_hexagon_V6_vaslh_128B HvxVR:$src1, IntRegs:$src2),
  2020:          (V6_vaslh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2021: def: Pat<(int_hexagon_V6_vaslhv HvxVR:$src1, HvxVR:$src2),
  2022:          (V6_vaslhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2023: def: Pat<(int_hexagon_V6_vaslhv_128B HvxVR:$src1, HvxVR:$src2),
  2024:          (V6_vaslhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2025: def: Pat<(int_hexagon_V6_vaslw HvxVR:$src1, IntRegs:$src2),
  2026:          (V6_vaslw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2027: def: Pat<(int_hexagon_V6_vaslw_128B HvxVR:$src1, IntRegs:$src2),
  2028:          (V6_vaslw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2029: def: Pat<(int_hexagon_V6_vaslw_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2030:          (V6_vaslw_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2031: def: Pat<(int_hexagon_V6_vaslw_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2032:          (V6_vaslw_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2033: def: Pat<(int_hexagon_V6_vaslwv HvxVR:$src1, HvxVR:$src2),
  2034:          (V6_vaslwv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2035: def: Pat<(int_hexagon_V6_vaslwv_128B HvxVR:$src1, HvxVR:$src2),
  2036:          (V6_vaslwv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2037: def: Pat<(int_hexagon_V6_vasrh HvxVR:$src1, IntRegs:$src2),
  2038:          (V6_vasrh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2039: def: Pat<(int_hexagon_V6_vasrh_128B HvxVR:$src1, IntRegs:$src2),
  2040:          (V6_vasrh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2041: def: Pat<(int_hexagon_V6_vasrhbrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2042:          (V6_vasrhbrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2043: def: Pat<(int_hexagon_V6_vasrhbrndsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2044:          (V6_vasrhbrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2045: def: Pat<(int_hexagon_V6_vasrhubrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2046:          (V6_vasrhubrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2047: def: Pat<(int_hexagon_V6_vasrhubrndsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2048:          (V6_vasrhubrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2049: def: Pat<(int_hexagon_V6_vasrhubsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2050:          (V6_vasrhubsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2051: def: Pat<(int_hexagon_V6_vasrhubsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2052:          (V6_vasrhubsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2053: def: Pat<(int_hexagon_V6_vasrhv HvxVR:$src1, HvxVR:$src2),
  2054:          (V6_vasrhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2055: def: Pat<(int_hexagon_V6_vasrhv_128B HvxVR:$src1, HvxVR:$src2),
  2056:          (V6_vasrhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2057: def: Pat<(int_hexagon_V6_vasrw HvxVR:$src1, IntRegs:$src2),
  2058:          (V6_vasrw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2059: def: Pat<(int_hexagon_V6_vasrw_128B HvxVR:$src1, IntRegs:$src2),
  2060:          (V6_vasrw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2061: def: Pat<(int_hexagon_V6_vasrw_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2062:          (V6_vasrw_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2063: def: Pat<(int_hexagon_V6_vasrw_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2064:          (V6_vasrw_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2065: def: Pat<(int_hexagon_V6_vasrwh HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2066:          (V6_vasrwh HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2067: def: Pat<(int_hexagon_V6_vasrwh_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2068:          (V6_vasrwh HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2069: def: Pat<(int_hexagon_V6_vasrwhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2070:          (V6_vasrwhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2071: def: Pat<(int_hexagon_V6_vasrwhrndsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2072:          (V6_vasrwhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2073: def: Pat<(int_hexagon_V6_vasrwhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2074:          (V6_vasrwhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2075: def: Pat<(int_hexagon_V6_vasrwhsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2076:          (V6_vasrwhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2077: def: Pat<(int_hexagon_V6_vasrwuhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2078:          (V6_vasrwuhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2079: def: Pat<(int_hexagon_V6_vasrwuhsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2080:          (V6_vasrwuhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2081: def: Pat<(int_hexagon_V6_vasrwv HvxVR:$src1, HvxVR:$src2),
  2082:          (V6_vasrwv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2083: def: Pat<(int_hexagon_V6_vasrwv_128B HvxVR:$src1, HvxVR:$src2),
  2084:          (V6_vasrwv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2085: def: Pat<(int_hexagon_V6_vassign HvxVR:$src1),
  2086:          (V6_vassign HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2087: def: Pat<(int_hexagon_V6_vassign_128B HvxVR:$src1),
  2088:          (V6_vassign HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2089: def: Pat<(int_hexagon_V6_vassignp HvxWR:$src1),
  2090:          (V6_vassignp HvxWR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2091: def: Pat<(int_hexagon_V6_vassignp_128B HvxWR:$src1),
  2092:          (V6_vassignp HvxWR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2093: def: Pat<(int_hexagon_V6_vavgh HvxVR:$src1, HvxVR:$src2),
  2094:          (V6_vavgh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2095: def: Pat<(int_hexagon_V6_vavgh_128B HvxVR:$src1, HvxVR:$src2),
  2096:          (V6_vavgh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2097: def: Pat<(int_hexagon_V6_vavghrnd HvxVR:$src1, HvxVR:$src2),
  2098:          (V6_vavghrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2099: def: Pat<(int_hexagon_V6_vavghrnd_128B HvxVR:$src1, HvxVR:$src2),
  2100:          (V6_vavghrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2101: def: Pat<(int_hexagon_V6_vavgub HvxVR:$src1, HvxVR:$src2),
  2102:          (V6_vavgub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2103: def: Pat<(int_hexagon_V6_vavgub_128B HvxVR:$src1, HvxVR:$src2),
  2104:          (V6_vavgub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2105: def: Pat<(int_hexagon_V6_vavgubrnd HvxVR:$src1, HvxVR:$src2),
  2106:          (V6_vavgubrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2107: def: Pat<(int_hexagon_V6_vavgubrnd_128B HvxVR:$src1, HvxVR:$src2),
  2108:          (V6_vavgubrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2109: def: Pat<(int_hexagon_V6_vavguh HvxVR:$src1, HvxVR:$src2),
  2110:          (V6_vavguh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2111: def: Pat<(int_hexagon_V6_vavguh_128B HvxVR:$src1, HvxVR:$src2),
  2112:          (V6_vavguh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2113: def: Pat<(int_hexagon_V6_vavguhrnd HvxVR:$src1, HvxVR:$src2),
  2114:          (V6_vavguhrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2115: def: Pat<(int_hexagon_V6_vavguhrnd_128B HvxVR:$src1, HvxVR:$src2),
  2116:          (V6_vavguhrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2117: def: Pat<(int_hexagon_V6_vavgw HvxVR:$src1, HvxVR:$src2),
  2118:          (V6_vavgw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2119: def: Pat<(int_hexagon_V6_vavgw_128B HvxVR:$src1, HvxVR:$src2),
  2120:          (V6_vavgw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2121: def: Pat<(int_hexagon_V6_vavgwrnd HvxVR:$src1, HvxVR:$src2),
  2122:          (V6_vavgwrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2123: def: Pat<(int_hexagon_V6_vavgwrnd_128B HvxVR:$src1, HvxVR:$src2),
  2124:          (V6_vavgwrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2125: def: Pat<(int_hexagon_V6_vcl0h HvxVR:$src1),
  2126:          (V6_vcl0h HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2127: def: Pat<(int_hexagon_V6_vcl0h_128B HvxVR:$src1),
  2128:          (V6_vcl0h HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2129: def: Pat<(int_hexagon_V6_vcl0w HvxVR:$src1),
  2130:          (V6_vcl0w HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2131: def: Pat<(int_hexagon_V6_vcl0w_128B HvxVR:$src1),
  2132:          (V6_vcl0w HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2133: def: Pat<(int_hexagon_V6_vcombine HvxVR:$src1, HvxVR:$src2),
  2134:          (V6_vcombine HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2135: def: Pat<(int_hexagon_V6_vcombine_128B HvxVR:$src1, HvxVR:$src2),
  2136:          (V6_vcombine HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2137: def: Pat<(int_hexagon_V6_vd0 ),
  2138:          (V6_vd0 )>, Requires<[UseHVXV60, UseHVX64B]>;
  2139: def: Pat<(int_hexagon_V6_vd0_128B ),
  2140:          (V6_vd0 )>, Requires<[UseHVXV60, UseHVX128B]>;
  2141: def: Pat<(int_hexagon_V6_vdealb HvxVR:$src1),
  2142:          (V6_vdealb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2143: def: Pat<(int_hexagon_V6_vdealb_128B HvxVR:$src1),
  2144:          (V6_vdealb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2145: def: Pat<(int_hexagon_V6_vdealb4w HvxVR:$src1, HvxVR:$src2),
  2146:          (V6_vdealb4w HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2147: def: Pat<(int_hexagon_V6_vdealb4w_128B HvxVR:$src1, HvxVR:$src2),
  2148:          (V6_vdealb4w HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2149: def: Pat<(int_hexagon_V6_vdealh HvxVR:$src1),
  2150:          (V6_vdealh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2151: def: Pat<(int_hexagon_V6_vdealh_128B HvxVR:$src1),
  2152:          (V6_vdealh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2153: def: Pat<(int_hexagon_V6_vdealvdd HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2154:          (V6_vdealvdd HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2155: def: Pat<(int_hexagon_V6_vdealvdd_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2156:          (V6_vdealvdd HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2157: def: Pat<(int_hexagon_V6_vdelta HvxVR:$src1, HvxVR:$src2),
  2158:          (V6_vdelta HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2159: def: Pat<(int_hexagon_V6_vdelta_128B HvxVR:$src1, HvxVR:$src2),
  2160:          (V6_vdelta HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2161: def: Pat<(int_hexagon_V6_vdmpybus HvxVR:$src1, IntRegs:$src2),
  2162:          (V6_vdmpybus HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2163: def: Pat<(int_hexagon_V6_vdmpybus_128B HvxVR:$src1, IntRegs:$src2),
  2164:          (V6_vdmpybus HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2165: def: Pat<(int_hexagon_V6_vdmpybus_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2166:          (V6_vdmpybus_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2167: def: Pat<(int_hexagon_V6_vdmpybus_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2168:          (V6_vdmpybus_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2169: def: Pat<(int_hexagon_V6_vdmpybus_dv HvxWR:$src1, IntRegs:$src2),
  2170:          (V6_vdmpybus_dv HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2171: def: Pat<(int_hexagon_V6_vdmpybus_dv_128B HvxWR:$src1, IntRegs:$src2),
  2172:          (V6_vdmpybus_dv HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2173: def: Pat<(int_hexagon_V6_vdmpybus_dv_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2174:          (V6_vdmpybus_dv_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2175: def: Pat<(int_hexagon_V6_vdmpybus_dv_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2176:          (V6_vdmpybus_dv_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2177: def: Pat<(int_hexagon_V6_vdmpyhb HvxVR:$src1, IntRegs:$src2),
  2178:          (V6_vdmpyhb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2179: def: Pat<(int_hexagon_V6_vdmpyhb_128B HvxVR:$src1, IntRegs:$src2),
  2180:          (V6_vdmpyhb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2181: def: Pat<(int_hexagon_V6_vdmpyhb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2182:          (V6_vdmpyhb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2183: def: Pat<(int_hexagon_V6_vdmpyhb_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2184:          (V6_vdmpyhb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2185: def: Pat<(int_hexagon_V6_vdmpyhb_dv HvxWR:$src1, IntRegs:$src2),
  2186:          (V6_vdmpyhb_dv HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2187: def: Pat<(int_hexagon_V6_vdmpyhb_dv_128B HvxWR:$src1, IntRegs:$src2),
  2188:          (V6_vdmpyhb_dv HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2189: def: Pat<(int_hexagon_V6_vdmpyhb_dv_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2190:          (V6_vdmpyhb_dv_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2191: def: Pat<(int_hexagon_V6_vdmpyhb_dv_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2192:          (V6_vdmpyhb_dv_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2193: def: Pat<(int_hexagon_V6_vdmpyhisat HvxWR:$src1, IntRegs:$src2),
  2194:          (V6_vdmpyhisat HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2195: def: Pat<(int_hexagon_V6_vdmpyhisat_128B HvxWR:$src1, IntRegs:$src2),
  2196:          (V6_vdmpyhisat HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2197: def: Pat<(int_hexagon_V6_vdmpyhisat_acc HvxVR:$src1, HvxWR:$src2, IntRegs:$src3),
  2198:          (V6_vdmpyhisat_acc HvxVR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2199: def: Pat<(int_hexagon_V6_vdmpyhisat_acc_128B HvxVR:$src1, HvxWR:$src2, IntRegs:$src3),
  2200:          (V6_vdmpyhisat_acc HvxVR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2201: def: Pat<(int_hexagon_V6_vdmpyhsat HvxVR:$src1, IntRegs:$src2),
  2202:          (V6_vdmpyhsat HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2203: def: Pat<(int_hexagon_V6_vdmpyhsat_128B HvxVR:$src1, IntRegs:$src2),
  2204:          (V6_vdmpyhsat HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2205: def: Pat<(int_hexagon_V6_vdmpyhsat_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2206:          (V6_vdmpyhsat_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2207: def: Pat<(int_hexagon_V6_vdmpyhsat_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2208:          (V6_vdmpyhsat_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2209: def: Pat<(int_hexagon_V6_vdmpyhsuisat HvxWR:$src1, IntRegs:$src2),
  2210:          (V6_vdmpyhsuisat HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2211: def: Pat<(int_hexagon_V6_vdmpyhsuisat_128B HvxWR:$src1, IntRegs:$src2),
  2212:          (V6_vdmpyhsuisat HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2213: def: Pat<(int_hexagon_V6_vdmpyhsuisat_acc HvxVR:$src1, HvxWR:$src2, IntRegs:$src3),
  2214:          (V6_vdmpyhsuisat_acc HvxVR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2215: def: Pat<(int_hexagon_V6_vdmpyhsuisat_acc_128B HvxVR:$src1, HvxWR:$src2, IntRegs:$src3),
  2216:          (V6_vdmpyhsuisat_acc HvxVR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2217: def: Pat<(int_hexagon_V6_vdmpyhsusat HvxVR:$src1, IntRegs:$src2),
  2218:          (V6_vdmpyhsusat HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2219: def: Pat<(int_hexagon_V6_vdmpyhsusat_128B HvxVR:$src1, IntRegs:$src2),
  2220:          (V6_vdmpyhsusat HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2221: def: Pat<(int_hexagon_V6_vdmpyhsusat_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2222:          (V6_vdmpyhsusat_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2223: def: Pat<(int_hexagon_V6_vdmpyhsusat_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2224:          (V6_vdmpyhsusat_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2225: def: Pat<(int_hexagon_V6_vdmpyhvsat HvxVR:$src1, HvxVR:$src2),
  2226:          (V6_vdmpyhvsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2227: def: Pat<(int_hexagon_V6_vdmpyhvsat_128B HvxVR:$src1, HvxVR:$src2),
  2228:          (V6_vdmpyhvsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2229: def: Pat<(int_hexagon_V6_vdmpyhvsat_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2230:          (V6_vdmpyhvsat_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2231: def: Pat<(int_hexagon_V6_vdmpyhvsat_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2232:          (V6_vdmpyhvsat_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2233: def: Pat<(int_hexagon_V6_vdsaduh HvxWR:$src1, IntRegs:$src2),
  2234:          (V6_vdsaduh HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2235: def: Pat<(int_hexagon_V6_vdsaduh_128B HvxWR:$src1, IntRegs:$src2),
  2236:          (V6_vdsaduh HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2237: def: Pat<(int_hexagon_V6_vdsaduh_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2238:          (V6_vdsaduh_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2239: def: Pat<(int_hexagon_V6_vdsaduh_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2240:          (V6_vdsaduh_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2241: def: Pat<(int_hexagon_V6_veqb HvxVR:$src1, HvxVR:$src2),
  2242:          (V6_veqb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2243: def: Pat<(int_hexagon_V6_veqb_128B HvxVR:$src1, HvxVR:$src2),
  2244:          (V6_veqb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2245: def: Pat<(int_hexagon_V6_veqb_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2246:          (V6_veqb_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2247: def: Pat<(int_hexagon_V6_veqb_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2248:          (V6_veqb_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2249: def: Pat<(int_hexagon_V6_veqb_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2250:          (V6_veqb_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2251-2500 / 第 2251-2500 行

```tablegen
  2251: def: Pat<(int_hexagon_V6_veqb_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2252:          (V6_veqb_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2253: def: Pat<(int_hexagon_V6_veqb_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2254:          (V6_veqb_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2255: def: Pat<(int_hexagon_V6_veqb_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2256:          (V6_veqb_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2257: def: Pat<(int_hexagon_V6_veqh HvxVR:$src1, HvxVR:$src2),
  2258:          (V6_veqh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2259: def: Pat<(int_hexagon_V6_veqh_128B HvxVR:$src1, HvxVR:$src2),
  2260:          (V6_veqh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2261: def: Pat<(int_hexagon_V6_veqh_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2262:          (V6_veqh_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2263: def: Pat<(int_hexagon_V6_veqh_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2264:          (V6_veqh_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2265: def: Pat<(int_hexagon_V6_veqh_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2266:          (V6_veqh_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2267: def: Pat<(int_hexagon_V6_veqh_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2268:          (V6_veqh_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2269: def: Pat<(int_hexagon_V6_veqh_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2270:          (V6_veqh_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2271: def: Pat<(int_hexagon_V6_veqh_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2272:          (V6_veqh_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2273: def: Pat<(int_hexagon_V6_veqw HvxVR:$src1, HvxVR:$src2),
  2274:          (V6_veqw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2275: def: Pat<(int_hexagon_V6_veqw_128B HvxVR:$src1, HvxVR:$src2),
  2276:          (V6_veqw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2277: def: Pat<(int_hexagon_V6_veqw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2278:          (V6_veqw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2279: def: Pat<(int_hexagon_V6_veqw_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2280:          (V6_veqw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2281: def: Pat<(int_hexagon_V6_veqw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2282:          (V6_veqw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2283: def: Pat<(int_hexagon_V6_veqw_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2284:          (V6_veqw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2285: def: Pat<(int_hexagon_V6_veqw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2286:          (V6_veqw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2287: def: Pat<(int_hexagon_V6_veqw_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2288:          (V6_veqw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2289: def: Pat<(int_hexagon_V6_vgtb HvxVR:$src1, HvxVR:$src2),
  2290:          (V6_vgtb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2291: def: Pat<(int_hexagon_V6_vgtb_128B HvxVR:$src1, HvxVR:$src2),
  2292:          (V6_vgtb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2293: def: Pat<(int_hexagon_V6_vgtb_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2294:          (V6_vgtb_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2295: def: Pat<(int_hexagon_V6_vgtb_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2296:          (V6_vgtb_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2297: def: Pat<(int_hexagon_V6_vgtb_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2298:          (V6_vgtb_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2299: def: Pat<(int_hexagon_V6_vgtb_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2300:          (V6_vgtb_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2301: def: Pat<(int_hexagon_V6_vgtb_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2302:          (V6_vgtb_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2303: def: Pat<(int_hexagon_V6_vgtb_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2304:          (V6_vgtb_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2305: def: Pat<(int_hexagon_V6_vgth HvxVR:$src1, HvxVR:$src2),
  2306:          (V6_vgth HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2307: def: Pat<(int_hexagon_V6_vgth_128B HvxVR:$src1, HvxVR:$src2),
  2308:          (V6_vgth HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2309: def: Pat<(int_hexagon_V6_vgth_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2310:          (V6_vgth_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2311: def: Pat<(int_hexagon_V6_vgth_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2312:          (V6_vgth_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2313: def: Pat<(int_hexagon_V6_vgth_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2314:          (V6_vgth_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2315: def: Pat<(int_hexagon_V6_vgth_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2316:          (V6_vgth_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2317: def: Pat<(int_hexagon_V6_vgth_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2318:          (V6_vgth_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2319: def: Pat<(int_hexagon_V6_vgth_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2320:          (V6_vgth_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2321: def: Pat<(int_hexagon_V6_vgtub HvxVR:$src1, HvxVR:$src2),
  2322:          (V6_vgtub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2323: def: Pat<(int_hexagon_V6_vgtub_128B HvxVR:$src1, HvxVR:$src2),
  2324:          (V6_vgtub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2325: def: Pat<(int_hexagon_V6_vgtub_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2326:          (V6_vgtub_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2327: def: Pat<(int_hexagon_V6_vgtub_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2328:          (V6_vgtub_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2329: def: Pat<(int_hexagon_V6_vgtub_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2330:          (V6_vgtub_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2331: def: Pat<(int_hexagon_V6_vgtub_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2332:          (V6_vgtub_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2333: def: Pat<(int_hexagon_V6_vgtub_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2334:          (V6_vgtub_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2335: def: Pat<(int_hexagon_V6_vgtub_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2336:          (V6_vgtub_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2337: def: Pat<(int_hexagon_V6_vgtuh HvxVR:$src1, HvxVR:$src2),
  2338:          (V6_vgtuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2339: def: Pat<(int_hexagon_V6_vgtuh_128B HvxVR:$src1, HvxVR:$src2),
  2340:          (V6_vgtuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2341: def: Pat<(int_hexagon_V6_vgtuh_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2342:          (V6_vgtuh_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2343: def: Pat<(int_hexagon_V6_vgtuh_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2344:          (V6_vgtuh_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2345: def: Pat<(int_hexagon_V6_vgtuh_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2346:          (V6_vgtuh_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2347: def: Pat<(int_hexagon_V6_vgtuh_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2348:          (V6_vgtuh_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2349: def: Pat<(int_hexagon_V6_vgtuh_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2350:          (V6_vgtuh_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2351: def: Pat<(int_hexagon_V6_vgtuh_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2352:          (V6_vgtuh_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2353: def: Pat<(int_hexagon_V6_vgtuw HvxVR:$src1, HvxVR:$src2),
  2354:          (V6_vgtuw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2355: def: Pat<(int_hexagon_V6_vgtuw_128B HvxVR:$src1, HvxVR:$src2),
  2356:          (V6_vgtuw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2357: def: Pat<(int_hexagon_V6_vgtuw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2358:          (V6_vgtuw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2359: def: Pat<(int_hexagon_V6_vgtuw_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2360:          (V6_vgtuw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2361: def: Pat<(int_hexagon_V6_vgtuw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2362:          (V6_vgtuw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2363: def: Pat<(int_hexagon_V6_vgtuw_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2364:          (V6_vgtuw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2365: def: Pat<(int_hexagon_V6_vgtuw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2366:          (V6_vgtuw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2367: def: Pat<(int_hexagon_V6_vgtuw_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2368:          (V6_vgtuw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2369: def: Pat<(int_hexagon_V6_vgtw HvxVR:$src1, HvxVR:$src2),
  2370:          (V6_vgtw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2371: def: Pat<(int_hexagon_V6_vgtw_128B HvxVR:$src1, HvxVR:$src2),
  2372:          (V6_vgtw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2373: def: Pat<(int_hexagon_V6_vgtw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2374:          (V6_vgtw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2375: def: Pat<(int_hexagon_V6_vgtw_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2376:          (V6_vgtw_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2377: def: Pat<(int_hexagon_V6_vgtw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2378:          (V6_vgtw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2379: def: Pat<(int_hexagon_V6_vgtw_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2380:          (V6_vgtw_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2381: def: Pat<(int_hexagon_V6_vgtw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2382:          (V6_vgtw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2383: def: Pat<(int_hexagon_V6_vgtw_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2384:          (V6_vgtw_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2385: def: Pat<(int_hexagon_V6_vinsertwr HvxVR:$src1, IntRegs:$src2),
  2386:          (V6_vinsertwr HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2387: def: Pat<(int_hexagon_V6_vinsertwr_128B HvxVR:$src1, IntRegs:$src2),
  2388:          (V6_vinsertwr HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2389: def: Pat<(int_hexagon_V6_vlalignb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2390:          (V6_vlalignb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2391: def: Pat<(int_hexagon_V6_vlalignb_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2392:          (V6_vlalignb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2393: def: Pat<(int_hexagon_V6_vlalignbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  2394:          (V6_vlalignbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2395: def: Pat<(int_hexagon_V6_vlalignbi_128B HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  2396:          (V6_vlalignbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2397: def: Pat<(int_hexagon_V6_vlsrh HvxVR:$src1, IntRegs:$src2),
  2398:          (V6_vlsrh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2399: def: Pat<(int_hexagon_V6_vlsrh_128B HvxVR:$src1, IntRegs:$src2),
  2400:          (V6_vlsrh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2401: def: Pat<(int_hexagon_V6_vlsrhv HvxVR:$src1, HvxVR:$src2),
  2402:          (V6_vlsrhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2403: def: Pat<(int_hexagon_V6_vlsrhv_128B HvxVR:$src1, HvxVR:$src2),
  2404:          (V6_vlsrhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2405: def: Pat<(int_hexagon_V6_vlsrw HvxVR:$src1, IntRegs:$src2),
  2406:          (V6_vlsrw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2407: def: Pat<(int_hexagon_V6_vlsrw_128B HvxVR:$src1, IntRegs:$src2),
  2408:          (V6_vlsrw HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2409: def: Pat<(int_hexagon_V6_vlsrwv HvxVR:$src1, HvxVR:$src2),
  2410:          (V6_vlsrwv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2411: def: Pat<(int_hexagon_V6_vlsrwv_128B HvxVR:$src1, HvxVR:$src2),
  2412:          (V6_vlsrwv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2413: def: Pat<(int_hexagon_V6_vlutvvb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2414:          (V6_vlutvvb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2415: def: Pat<(int_hexagon_V6_vlutvvb_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2416:          (V6_vlutvvb HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2417: def: Pat<(int_hexagon_V6_vlutvvb_oracc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4),
  2418:          (V6_vlutvvb_oracc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4)>, Requires<[UseHVXV60, UseHVX64B]>;
  2419: def: Pat<(int_hexagon_V6_vlutvvb_oracc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4),
  2420:          (V6_vlutvvb_oracc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4)>, Requires<[UseHVXV60, UseHVX128B]>;
  2421: def: Pat<(int_hexagon_V6_vlutvwh HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2422:          (V6_vlutvwh HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2423: def: Pat<(int_hexagon_V6_vlutvwh_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2424:          (V6_vlutvwh HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2425: def: Pat<(int_hexagon_V6_vlutvwh_oracc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4),
  2426:          (V6_vlutvwh_oracc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4)>, Requires<[UseHVXV60, UseHVX64B]>;
  2427: def: Pat<(int_hexagon_V6_vlutvwh_oracc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4),
  2428:          (V6_vlutvwh_oracc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, IntRegsLow8:$src4)>, Requires<[UseHVXV60, UseHVX128B]>;
  2429: def: Pat<(int_hexagon_V6_vmaxh HvxVR:$src1, HvxVR:$src2),
  2430:          (V6_vmaxh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2431: def: Pat<(int_hexagon_V6_vmaxh_128B HvxVR:$src1, HvxVR:$src2),
  2432:          (V6_vmaxh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2433: def: Pat<(int_hexagon_V6_vmaxub HvxVR:$src1, HvxVR:$src2),
  2434:          (V6_vmaxub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2435: def: Pat<(int_hexagon_V6_vmaxub_128B HvxVR:$src1, HvxVR:$src2),
  2436:          (V6_vmaxub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2437: def: Pat<(int_hexagon_V6_vmaxuh HvxVR:$src1, HvxVR:$src2),
  2438:          (V6_vmaxuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2439: def: Pat<(int_hexagon_V6_vmaxuh_128B HvxVR:$src1, HvxVR:$src2),
  2440:          (V6_vmaxuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2441: def: Pat<(int_hexagon_V6_vmaxw HvxVR:$src1, HvxVR:$src2),
  2442:          (V6_vmaxw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2443: def: Pat<(int_hexagon_V6_vmaxw_128B HvxVR:$src1, HvxVR:$src2),
  2444:          (V6_vmaxw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2445: def: Pat<(int_hexagon_V6_vminh HvxVR:$src1, HvxVR:$src2),
  2446:          (V6_vminh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2447: def: Pat<(int_hexagon_V6_vminh_128B HvxVR:$src1, HvxVR:$src2),
  2448:          (V6_vminh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2449: def: Pat<(int_hexagon_V6_vminub HvxVR:$src1, HvxVR:$src2),
  2450:          (V6_vminub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2451: def: Pat<(int_hexagon_V6_vminub_128B HvxVR:$src1, HvxVR:$src2),
  2452:          (V6_vminub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2453: def: Pat<(int_hexagon_V6_vminuh HvxVR:$src1, HvxVR:$src2),
  2454:          (V6_vminuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2455: def: Pat<(int_hexagon_V6_vminuh_128B HvxVR:$src1, HvxVR:$src2),
  2456:          (V6_vminuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2457: def: Pat<(int_hexagon_V6_vminw HvxVR:$src1, HvxVR:$src2),
  2458:          (V6_vminw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2459: def: Pat<(int_hexagon_V6_vminw_128B HvxVR:$src1, HvxVR:$src2),
  2460:          (V6_vminw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2461: def: Pat<(int_hexagon_V6_vmpabus HvxWR:$src1, IntRegs:$src2),
  2462:          (V6_vmpabus HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2463: def: Pat<(int_hexagon_V6_vmpabus_128B HvxWR:$src1, IntRegs:$src2),
  2464:          (V6_vmpabus HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2465: def: Pat<(int_hexagon_V6_vmpabus_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2466:          (V6_vmpabus_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2467: def: Pat<(int_hexagon_V6_vmpabus_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2468:          (V6_vmpabus_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2469: def: Pat<(int_hexagon_V6_vmpabusv HvxWR:$src1, HvxWR:$src2),
  2470:          (V6_vmpabusv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2471: def: Pat<(int_hexagon_V6_vmpabusv_128B HvxWR:$src1, HvxWR:$src2),
  2472:          (V6_vmpabusv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2473: def: Pat<(int_hexagon_V6_vmpabuuv HvxWR:$src1, HvxWR:$src2),
  2474:          (V6_vmpabuuv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2475: def: Pat<(int_hexagon_V6_vmpabuuv_128B HvxWR:$src1, HvxWR:$src2),
  2476:          (V6_vmpabuuv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2477: def: Pat<(int_hexagon_V6_vmpahb HvxWR:$src1, IntRegs:$src2),
  2478:          (V6_vmpahb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2479: def: Pat<(int_hexagon_V6_vmpahb_128B HvxWR:$src1, IntRegs:$src2),
  2480:          (V6_vmpahb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2481: def: Pat<(int_hexagon_V6_vmpahb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2482:          (V6_vmpahb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2483: def: Pat<(int_hexagon_V6_vmpahb_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2484:          (V6_vmpahb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2485: def: Pat<(int_hexagon_V6_vmpybus HvxVR:$src1, IntRegs:$src2),
  2486:          (V6_vmpybus HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2487: def: Pat<(int_hexagon_V6_vmpybus_128B HvxVR:$src1, IntRegs:$src2),
  2488:          (V6_vmpybus HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2489: def: Pat<(int_hexagon_V6_vmpybus_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2490:          (V6_vmpybus_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2491: def: Pat<(int_hexagon_V6_vmpybus_acc_128B HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2492:          (V6_vmpybus_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2493: def: Pat<(int_hexagon_V6_vmpybusv HvxVR:$src1, HvxVR:$src2),
  2494:          (V6_vmpybusv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2495: def: Pat<(int_hexagon_V6_vmpybusv_128B HvxVR:$src1, HvxVR:$src2),
  2496:          (V6_vmpybusv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2497: def: Pat<(int_hexagon_V6_vmpybusv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2498:          (V6_vmpybusv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2499: def: Pat<(int_hexagon_V6_vmpybusv_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2500:          (V6_vmpybusv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2501-2750 / 第 2501-2750 行

```tablegen
  2501: def: Pat<(int_hexagon_V6_vmpybv HvxVR:$src1, HvxVR:$src2),
  2502:          (V6_vmpybv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2503: def: Pat<(int_hexagon_V6_vmpybv_128B HvxVR:$src1, HvxVR:$src2),
  2504:          (V6_vmpybv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2505: def: Pat<(int_hexagon_V6_vmpybv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2506:          (V6_vmpybv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2507: def: Pat<(int_hexagon_V6_vmpybv_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2508:          (V6_vmpybv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2509: def: Pat<(int_hexagon_V6_vmpyewuh HvxVR:$src1, HvxVR:$src2),
  2510:          (V6_vmpyewuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2511: def: Pat<(int_hexagon_V6_vmpyewuh_128B HvxVR:$src1, HvxVR:$src2),
  2512:          (V6_vmpyewuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2513: def: Pat<(int_hexagon_V6_vmpyh HvxVR:$src1, IntRegs:$src2),
  2514:          (V6_vmpyh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2515: def: Pat<(int_hexagon_V6_vmpyh_128B HvxVR:$src1, IntRegs:$src2),
  2516:          (V6_vmpyh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2517: def: Pat<(int_hexagon_V6_vmpyhsat_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2518:          (V6_vmpyhsat_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2519: def: Pat<(int_hexagon_V6_vmpyhsat_acc_128B HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2520:          (V6_vmpyhsat_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2521: def: Pat<(int_hexagon_V6_vmpyhsrs HvxVR:$src1, IntRegs:$src2),
  2522:          (V6_vmpyhsrs HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2523: def: Pat<(int_hexagon_V6_vmpyhsrs_128B HvxVR:$src1, IntRegs:$src2),
  2524:          (V6_vmpyhsrs HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2525: def: Pat<(int_hexagon_V6_vmpyhss HvxVR:$src1, IntRegs:$src2),
  2526:          (V6_vmpyhss HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2527: def: Pat<(int_hexagon_V6_vmpyhss_128B HvxVR:$src1, IntRegs:$src2),
  2528:          (V6_vmpyhss HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2529: def: Pat<(int_hexagon_V6_vmpyhus HvxVR:$src1, HvxVR:$src2),
  2530:          (V6_vmpyhus HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2531: def: Pat<(int_hexagon_V6_vmpyhus_128B HvxVR:$src1, HvxVR:$src2),
  2532:          (V6_vmpyhus HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2533: def: Pat<(int_hexagon_V6_vmpyhus_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2534:          (V6_vmpyhus_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2535: def: Pat<(int_hexagon_V6_vmpyhus_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2536:          (V6_vmpyhus_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2537: def: Pat<(int_hexagon_V6_vmpyhv HvxVR:$src1, HvxVR:$src2),
  2538:          (V6_vmpyhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2539: def: Pat<(int_hexagon_V6_vmpyhv_128B HvxVR:$src1, HvxVR:$src2),
  2540:          (V6_vmpyhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2541: def: Pat<(int_hexagon_V6_vmpyhv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2542:          (V6_vmpyhv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2543: def: Pat<(int_hexagon_V6_vmpyhv_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2544:          (V6_vmpyhv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2545: def: Pat<(int_hexagon_V6_vmpyhvsrs HvxVR:$src1, HvxVR:$src2),
  2546:          (V6_vmpyhvsrs HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2547: def: Pat<(int_hexagon_V6_vmpyhvsrs_128B HvxVR:$src1, HvxVR:$src2),
  2548:          (V6_vmpyhvsrs HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2549: def: Pat<(int_hexagon_V6_vmpyieoh HvxVR:$src1, HvxVR:$src2),
  2550:          (V6_vmpyieoh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2551: def: Pat<(int_hexagon_V6_vmpyieoh_128B HvxVR:$src1, HvxVR:$src2),
  2552:          (V6_vmpyieoh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2553: def: Pat<(int_hexagon_V6_vmpyiewh_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2554:          (V6_vmpyiewh_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2555: def: Pat<(int_hexagon_V6_vmpyiewh_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2556:          (V6_vmpyiewh_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2557: def: Pat<(int_hexagon_V6_vmpyiewuh HvxVR:$src1, HvxVR:$src2),
  2558:          (V6_vmpyiewuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2559: def: Pat<(int_hexagon_V6_vmpyiewuh_128B HvxVR:$src1, HvxVR:$src2),
  2560:          (V6_vmpyiewuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2561: def: Pat<(int_hexagon_V6_vmpyiewuh_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2562:          (V6_vmpyiewuh_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2563: def: Pat<(int_hexagon_V6_vmpyiewuh_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2564:          (V6_vmpyiewuh_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2565: def: Pat<(int_hexagon_V6_vmpyih HvxVR:$src1, HvxVR:$src2),
  2566:          (V6_vmpyih HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2567: def: Pat<(int_hexagon_V6_vmpyih_128B HvxVR:$src1, HvxVR:$src2),
  2568:          (V6_vmpyih HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2569: def: Pat<(int_hexagon_V6_vmpyih_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2570:          (V6_vmpyih_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2571: def: Pat<(int_hexagon_V6_vmpyih_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2572:          (V6_vmpyih_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2573: def: Pat<(int_hexagon_V6_vmpyihb HvxVR:$src1, IntRegs:$src2),
  2574:          (V6_vmpyihb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2575: def: Pat<(int_hexagon_V6_vmpyihb_128B HvxVR:$src1, IntRegs:$src2),
  2576:          (V6_vmpyihb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2577: def: Pat<(int_hexagon_V6_vmpyihb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2578:          (V6_vmpyihb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2579: def: Pat<(int_hexagon_V6_vmpyihb_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2580:          (V6_vmpyihb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2581: def: Pat<(int_hexagon_V6_vmpyiowh HvxVR:$src1, HvxVR:$src2),
  2582:          (V6_vmpyiowh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2583: def: Pat<(int_hexagon_V6_vmpyiowh_128B HvxVR:$src1, HvxVR:$src2),
  2584:          (V6_vmpyiowh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2585: def: Pat<(int_hexagon_V6_vmpyiwb HvxVR:$src1, IntRegs:$src2),
  2586:          (V6_vmpyiwb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2587: def: Pat<(int_hexagon_V6_vmpyiwb_128B HvxVR:$src1, IntRegs:$src2),
  2588:          (V6_vmpyiwb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2589: def: Pat<(int_hexagon_V6_vmpyiwb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2590:          (V6_vmpyiwb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2591: def: Pat<(int_hexagon_V6_vmpyiwb_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2592:          (V6_vmpyiwb_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2593: def: Pat<(int_hexagon_V6_vmpyiwh HvxVR:$src1, IntRegs:$src2),
  2594:          (V6_vmpyiwh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2595: def: Pat<(int_hexagon_V6_vmpyiwh_128B HvxVR:$src1, IntRegs:$src2),
  2596:          (V6_vmpyiwh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2597: def: Pat<(int_hexagon_V6_vmpyiwh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2598:          (V6_vmpyiwh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2599: def: Pat<(int_hexagon_V6_vmpyiwh_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2600:          (V6_vmpyiwh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2601: def: Pat<(int_hexagon_V6_vmpyowh HvxVR:$src1, HvxVR:$src2),
  2602:          (V6_vmpyowh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2603: def: Pat<(int_hexagon_V6_vmpyowh_128B HvxVR:$src1, HvxVR:$src2),
  2604:          (V6_vmpyowh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2605: def: Pat<(int_hexagon_V6_vmpyowh_rnd HvxVR:$src1, HvxVR:$src2),
  2606:          (V6_vmpyowh_rnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2607: def: Pat<(int_hexagon_V6_vmpyowh_rnd_128B HvxVR:$src1, HvxVR:$src2),
  2608:          (V6_vmpyowh_rnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2609: def: Pat<(int_hexagon_V6_vmpyowh_rnd_sacc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2610:          (V6_vmpyowh_rnd_sacc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2611: def: Pat<(int_hexagon_V6_vmpyowh_rnd_sacc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2612:          (V6_vmpyowh_rnd_sacc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2613: def: Pat<(int_hexagon_V6_vmpyowh_sacc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2614:          (V6_vmpyowh_sacc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2615: def: Pat<(int_hexagon_V6_vmpyowh_sacc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2616:          (V6_vmpyowh_sacc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2617: def: Pat<(int_hexagon_V6_vmpyub HvxVR:$src1, IntRegs:$src2),
  2618:          (V6_vmpyub HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2619: def: Pat<(int_hexagon_V6_vmpyub_128B HvxVR:$src1, IntRegs:$src2),
  2620:          (V6_vmpyub HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2621: def: Pat<(int_hexagon_V6_vmpyub_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2622:          (V6_vmpyub_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2623: def: Pat<(int_hexagon_V6_vmpyub_acc_128B HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2624:          (V6_vmpyub_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2625: def: Pat<(int_hexagon_V6_vmpyubv HvxVR:$src1, HvxVR:$src2),
  2626:          (V6_vmpyubv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2627: def: Pat<(int_hexagon_V6_vmpyubv_128B HvxVR:$src1, HvxVR:$src2),
  2628:          (V6_vmpyubv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2629: def: Pat<(int_hexagon_V6_vmpyubv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2630:          (V6_vmpyubv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2631: def: Pat<(int_hexagon_V6_vmpyubv_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2632:          (V6_vmpyubv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2633: def: Pat<(int_hexagon_V6_vmpyuh HvxVR:$src1, IntRegs:$src2),
  2634:          (V6_vmpyuh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2635: def: Pat<(int_hexagon_V6_vmpyuh_128B HvxVR:$src1, IntRegs:$src2),
  2636:          (V6_vmpyuh HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2637: def: Pat<(int_hexagon_V6_vmpyuh_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2638:          (V6_vmpyuh_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2639: def: Pat<(int_hexagon_V6_vmpyuh_acc_128B HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  2640:          (V6_vmpyuh_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2641: def: Pat<(int_hexagon_V6_vmpyuhv HvxVR:$src1, HvxVR:$src2),
  2642:          (V6_vmpyuhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2643: def: Pat<(int_hexagon_V6_vmpyuhv_128B HvxVR:$src1, HvxVR:$src2),
  2644:          (V6_vmpyuhv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2645: def: Pat<(int_hexagon_V6_vmpyuhv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2646:          (V6_vmpyuhv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2647: def: Pat<(int_hexagon_V6_vmpyuhv_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  2648:          (V6_vmpyuhv_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2649: def: Pat<(int_hexagon_V6_vmux HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2650:          (V6_vmux HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2651: def: Pat<(int_hexagon_V6_vmux_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2652:          (V6_vmux HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2653: def: Pat<(int_hexagon_V6_vnavgh HvxVR:$src1, HvxVR:$src2),
  2654:          (V6_vnavgh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2655: def: Pat<(int_hexagon_V6_vnavgh_128B HvxVR:$src1, HvxVR:$src2),
  2656:          (V6_vnavgh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2657: def: Pat<(int_hexagon_V6_vnavgub HvxVR:$src1, HvxVR:$src2),
  2658:          (V6_vnavgub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2659: def: Pat<(int_hexagon_V6_vnavgub_128B HvxVR:$src1, HvxVR:$src2),
  2660:          (V6_vnavgub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2661: def: Pat<(int_hexagon_V6_vnavgw HvxVR:$src1, HvxVR:$src2),
  2662:          (V6_vnavgw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2663: def: Pat<(int_hexagon_V6_vnavgw_128B HvxVR:$src1, HvxVR:$src2),
  2664:          (V6_vnavgw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2665: def: Pat<(int_hexagon_V6_vnormamth HvxVR:$src1),
  2666:          (V6_vnormamth HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2667: def: Pat<(int_hexagon_V6_vnormamth_128B HvxVR:$src1),
  2668:          (V6_vnormamth HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2669: def: Pat<(int_hexagon_V6_vnormamtw HvxVR:$src1),
  2670:          (V6_vnormamtw HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2671: def: Pat<(int_hexagon_V6_vnormamtw_128B HvxVR:$src1),
  2672:          (V6_vnormamtw HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2673: def: Pat<(int_hexagon_V6_vnot HvxVR:$src1),
  2674:          (V6_vnot HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2675: def: Pat<(int_hexagon_V6_vnot_128B HvxVR:$src1),
  2676:          (V6_vnot HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2677: def: Pat<(int_hexagon_V6_vor HvxVR:$src1, HvxVR:$src2),
  2678:          (V6_vor HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2679: def: Pat<(int_hexagon_V6_vor_128B HvxVR:$src1, HvxVR:$src2),
  2680:          (V6_vor HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2681: def: Pat<(int_hexagon_V6_vpackeb HvxVR:$src1, HvxVR:$src2),
  2682:          (V6_vpackeb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2683: def: Pat<(int_hexagon_V6_vpackeb_128B HvxVR:$src1, HvxVR:$src2),
  2684:          (V6_vpackeb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2685: def: Pat<(int_hexagon_V6_vpackeh HvxVR:$src1, HvxVR:$src2),
  2686:          (V6_vpackeh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2687: def: Pat<(int_hexagon_V6_vpackeh_128B HvxVR:$src1, HvxVR:$src2),
  2688:          (V6_vpackeh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2689: def: Pat<(int_hexagon_V6_vpackhb_sat HvxVR:$src1, HvxVR:$src2),
  2690:          (V6_vpackhb_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2691: def: Pat<(int_hexagon_V6_vpackhb_sat_128B HvxVR:$src1, HvxVR:$src2),
  2692:          (V6_vpackhb_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2693: def: Pat<(int_hexagon_V6_vpackhub_sat HvxVR:$src1, HvxVR:$src2),
  2694:          (V6_vpackhub_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2695: def: Pat<(int_hexagon_V6_vpackhub_sat_128B HvxVR:$src1, HvxVR:$src2),
  2696:          (V6_vpackhub_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2697: def: Pat<(int_hexagon_V6_vpackob HvxVR:$src1, HvxVR:$src2),
  2698:          (V6_vpackob HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2699: def: Pat<(int_hexagon_V6_vpackob_128B HvxVR:$src1, HvxVR:$src2),
  2700:          (V6_vpackob HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2701: def: Pat<(int_hexagon_V6_vpackoh HvxVR:$src1, HvxVR:$src2),
  2702:          (V6_vpackoh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2703: def: Pat<(int_hexagon_V6_vpackoh_128B HvxVR:$src1, HvxVR:$src2),
  2704:          (V6_vpackoh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2705: def: Pat<(int_hexagon_V6_vpackwh_sat HvxVR:$src1, HvxVR:$src2),
  2706:          (V6_vpackwh_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2707: def: Pat<(int_hexagon_V6_vpackwh_sat_128B HvxVR:$src1, HvxVR:$src2),
  2708:          (V6_vpackwh_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2709: def: Pat<(int_hexagon_V6_vpackwuh_sat HvxVR:$src1, HvxVR:$src2),
  2710:          (V6_vpackwuh_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2711: def: Pat<(int_hexagon_V6_vpackwuh_sat_128B HvxVR:$src1, HvxVR:$src2),
  2712:          (V6_vpackwuh_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2713: def: Pat<(int_hexagon_V6_vpopcounth HvxVR:$src1),
  2714:          (V6_vpopcounth HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2715: def: Pat<(int_hexagon_V6_vpopcounth_128B HvxVR:$src1),
  2716:          (V6_vpopcounth HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2717: def: Pat<(int_hexagon_V6_vrdelta HvxVR:$src1, HvxVR:$src2),
  2718:          (V6_vrdelta HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2719: def: Pat<(int_hexagon_V6_vrdelta_128B HvxVR:$src1, HvxVR:$src2),
  2720:          (V6_vrdelta HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2721: def: Pat<(int_hexagon_V6_vrmpybus HvxVR:$src1, IntRegs:$src2),
  2722:          (V6_vrmpybus HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2723: def: Pat<(int_hexagon_V6_vrmpybus_128B HvxVR:$src1, IntRegs:$src2),
  2724:          (V6_vrmpybus HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2725: def: Pat<(int_hexagon_V6_vrmpybus_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2726:          (V6_vrmpybus_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2727: def: Pat<(int_hexagon_V6_vrmpybus_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2728:          (V6_vrmpybus_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2729: def: Pat<(int_hexagon_V6_vrmpybusi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3),
  2730:          (V6_vrmpybusi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2731: def: Pat<(int_hexagon_V6_vrmpybusi_128B HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3),
  2732:          (V6_vrmpybusi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2733: def: Pat<(int_hexagon_V6_vrmpybusi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4),
  2734:          (V6_vrmpybusi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4)>, Requires<[UseHVXV60, UseHVX64B]>;
  2735: def: Pat<(int_hexagon_V6_vrmpybusi_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4),
  2736:          (V6_vrmpybusi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4)>, Requires<[UseHVXV60, UseHVX128B]>;
  2737: def: Pat<(int_hexagon_V6_vrmpybusv HvxVR:$src1, HvxVR:$src2),
  2738:          (V6_vrmpybusv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2739: def: Pat<(int_hexagon_V6_vrmpybusv_128B HvxVR:$src1, HvxVR:$src2),
  2740:          (V6_vrmpybusv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2741: def: Pat<(int_hexagon_V6_vrmpybusv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2742:          (V6_vrmpybusv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2743: def: Pat<(int_hexagon_V6_vrmpybusv_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2744:          (V6_vrmpybusv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2745: def: Pat<(int_hexagon_V6_vrmpybv HvxVR:$src1, HvxVR:$src2),
  2746:          (V6_vrmpybv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2747: def: Pat<(int_hexagon_V6_vrmpybv_128B HvxVR:$src1, HvxVR:$src2),
  2748:          (V6_vrmpybv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2749: def: Pat<(int_hexagon_V6_vrmpybv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2750:          (V6_vrmpybv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 2751-3000 / 第 2751-3000 行

```tablegen
  2751: def: Pat<(int_hexagon_V6_vrmpybv_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2752:          (V6_vrmpybv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2753: def: Pat<(int_hexagon_V6_vrmpyub HvxVR:$src1, IntRegs:$src2),
  2754:          (V6_vrmpyub HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2755: def: Pat<(int_hexagon_V6_vrmpyub_128B HvxVR:$src1, IntRegs:$src2),
  2756:          (V6_vrmpyub HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2757: def: Pat<(int_hexagon_V6_vrmpyub_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2758:          (V6_vrmpyub_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2759: def: Pat<(int_hexagon_V6_vrmpyub_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  2760:          (V6_vrmpyub_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2761: def: Pat<(int_hexagon_V6_vrmpyubi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3),
  2762:          (V6_vrmpyubi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2763: def: Pat<(int_hexagon_V6_vrmpyubi_128B HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3),
  2764:          (V6_vrmpyubi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2765: def: Pat<(int_hexagon_V6_vrmpyubi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4),
  2766:          (V6_vrmpyubi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4)>, Requires<[UseHVXV60, UseHVX64B]>;
  2767: def: Pat<(int_hexagon_V6_vrmpyubi_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4),
  2768:          (V6_vrmpyubi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4)>, Requires<[UseHVXV60, UseHVX128B]>;
  2769: def: Pat<(int_hexagon_V6_vrmpyubv HvxVR:$src1, HvxVR:$src2),
  2770:          (V6_vrmpyubv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2771: def: Pat<(int_hexagon_V6_vrmpyubv_128B HvxVR:$src1, HvxVR:$src2),
  2772:          (V6_vrmpyubv HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2773: def: Pat<(int_hexagon_V6_vrmpyubv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2774:          (V6_vrmpyubv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2775: def: Pat<(int_hexagon_V6_vrmpyubv_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  2776:          (V6_vrmpyubv_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2777: def: Pat<(int_hexagon_V6_vror HvxVR:$src1, IntRegs:$src2),
  2778:          (V6_vror HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2779: def: Pat<(int_hexagon_V6_vror_128B HvxVR:$src1, IntRegs:$src2),
  2780:          (V6_vror HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2781: def: Pat<(int_hexagon_V6_vroundhb HvxVR:$src1, HvxVR:$src2),
  2782:          (V6_vroundhb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2783: def: Pat<(int_hexagon_V6_vroundhb_128B HvxVR:$src1, HvxVR:$src2),
  2784:          (V6_vroundhb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2785: def: Pat<(int_hexagon_V6_vroundhub HvxVR:$src1, HvxVR:$src2),
  2786:          (V6_vroundhub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2787: def: Pat<(int_hexagon_V6_vroundhub_128B HvxVR:$src1, HvxVR:$src2),
  2788:          (V6_vroundhub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2789: def: Pat<(int_hexagon_V6_vroundwh HvxVR:$src1, HvxVR:$src2),
  2790:          (V6_vroundwh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2791: def: Pat<(int_hexagon_V6_vroundwh_128B HvxVR:$src1, HvxVR:$src2),
  2792:          (V6_vroundwh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2793: def: Pat<(int_hexagon_V6_vroundwuh HvxVR:$src1, HvxVR:$src2),
  2794:          (V6_vroundwuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2795: def: Pat<(int_hexagon_V6_vroundwuh_128B HvxVR:$src1, HvxVR:$src2),
  2796:          (V6_vroundwuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2797: def: Pat<(int_hexagon_V6_vrsadubi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3),
  2798:          (V6_vrsadubi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2799: def: Pat<(int_hexagon_V6_vrsadubi_128B HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3),
  2800:          (V6_vrsadubi HvxWR:$src1, IntRegs:$src2, u1_0ImmPred_timm:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2801: def: Pat<(int_hexagon_V6_vrsadubi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4),
  2802:          (V6_vrsadubi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4)>, Requires<[UseHVXV60, UseHVX64B]>;
  2803: def: Pat<(int_hexagon_V6_vrsadubi_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4),
  2804:          (V6_vrsadubi_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3, u1_0ImmPred_timm:$src4)>, Requires<[UseHVXV60, UseHVX128B]>;
  2805: def: Pat<(int_hexagon_V6_vsathub HvxVR:$src1, HvxVR:$src2),
  2806:          (V6_vsathub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2807: def: Pat<(int_hexagon_V6_vsathub_128B HvxVR:$src1, HvxVR:$src2),
  2808:          (V6_vsathub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2809: def: Pat<(int_hexagon_V6_vsatwh HvxVR:$src1, HvxVR:$src2),
  2810:          (V6_vsatwh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2811: def: Pat<(int_hexagon_V6_vsatwh_128B HvxVR:$src1, HvxVR:$src2),
  2812:          (V6_vsatwh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2813: def: Pat<(int_hexagon_V6_vsb HvxVR:$src1),
  2814:          (V6_vsb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2815: def: Pat<(int_hexagon_V6_vsb_128B HvxVR:$src1),
  2816:          (V6_vsb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2817: def: Pat<(int_hexagon_V6_vsh HvxVR:$src1),
  2818:          (V6_vsh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2819: def: Pat<(int_hexagon_V6_vsh_128B HvxVR:$src1),
  2820:          (V6_vsh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2821: def: Pat<(int_hexagon_V6_vshufeh HvxVR:$src1, HvxVR:$src2),
  2822:          (V6_vshufeh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2823: def: Pat<(int_hexagon_V6_vshufeh_128B HvxVR:$src1, HvxVR:$src2),
  2824:          (V6_vshufeh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2825: def: Pat<(int_hexagon_V6_vshuffb HvxVR:$src1),
  2826:          (V6_vshuffb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2827: def: Pat<(int_hexagon_V6_vshuffb_128B HvxVR:$src1),
  2828:          (V6_vshuffb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2829: def: Pat<(int_hexagon_V6_vshuffeb HvxVR:$src1, HvxVR:$src2),
  2830:          (V6_vshuffeb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2831: def: Pat<(int_hexagon_V6_vshuffeb_128B HvxVR:$src1, HvxVR:$src2),
  2832:          (V6_vshuffeb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2833: def: Pat<(int_hexagon_V6_vshuffh HvxVR:$src1),
  2834:          (V6_vshuffh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2835: def: Pat<(int_hexagon_V6_vshuffh_128B HvxVR:$src1),
  2836:          (V6_vshuffh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2837: def: Pat<(int_hexagon_V6_vshuffob HvxVR:$src1, HvxVR:$src2),
  2838:          (V6_vshuffob HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2839: def: Pat<(int_hexagon_V6_vshuffob_128B HvxVR:$src1, HvxVR:$src2),
  2840:          (V6_vshuffob HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2841: def: Pat<(int_hexagon_V6_vshuffvdd HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2842:          (V6_vshuffvdd HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2843: def: Pat<(int_hexagon_V6_vshuffvdd_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  2844:          (V6_vshuffvdd HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2845: def: Pat<(int_hexagon_V6_vshufoeb HvxVR:$src1, HvxVR:$src2),
  2846:          (V6_vshufoeb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2847: def: Pat<(int_hexagon_V6_vshufoeb_128B HvxVR:$src1, HvxVR:$src2),
  2848:          (V6_vshufoeb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2849: def: Pat<(int_hexagon_V6_vshufoeh HvxVR:$src1, HvxVR:$src2),
  2850:          (V6_vshufoeh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2851: def: Pat<(int_hexagon_V6_vshufoeh_128B HvxVR:$src1, HvxVR:$src2),
  2852:          (V6_vshufoeh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2853: def: Pat<(int_hexagon_V6_vshufoh HvxVR:$src1, HvxVR:$src2),
  2854:          (V6_vshufoh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2855: def: Pat<(int_hexagon_V6_vshufoh_128B HvxVR:$src1, HvxVR:$src2),
  2856:          (V6_vshufoh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2857: def: Pat<(int_hexagon_V6_vsubb HvxVR:$src1, HvxVR:$src2),
  2858:          (V6_vsubb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2859: def: Pat<(int_hexagon_V6_vsubb_128B HvxVR:$src1, HvxVR:$src2),
  2860:          (V6_vsubb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2861: def: Pat<(int_hexagon_V6_vsubb_dv HvxWR:$src1, HvxWR:$src2),
  2862:          (V6_vsubb_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2863: def: Pat<(int_hexagon_V6_vsubb_dv_128B HvxWR:$src1, HvxWR:$src2),
  2864:          (V6_vsubb_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2865: def: Pat<(int_hexagon_V6_vsubbnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2866:          (V6_vsubbnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2867: def: Pat<(int_hexagon_V6_vsubbnq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2868:          (V6_vsubbnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2869: def: Pat<(int_hexagon_V6_vsubbq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2870:          (V6_vsubbq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2871: def: Pat<(int_hexagon_V6_vsubbq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2872:          (V6_vsubbq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2873: def: Pat<(int_hexagon_V6_vsubh HvxVR:$src1, HvxVR:$src2),
  2874:          (V6_vsubh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2875: def: Pat<(int_hexagon_V6_vsubh_128B HvxVR:$src1, HvxVR:$src2),
  2876:          (V6_vsubh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2877: def: Pat<(int_hexagon_V6_vsubh_dv HvxWR:$src1, HvxWR:$src2),
  2878:          (V6_vsubh_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2879: def: Pat<(int_hexagon_V6_vsubh_dv_128B HvxWR:$src1, HvxWR:$src2),
  2880:          (V6_vsubh_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2881: def: Pat<(int_hexagon_V6_vsubhnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2882:          (V6_vsubhnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2883: def: Pat<(int_hexagon_V6_vsubhnq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2884:          (V6_vsubhnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2885: def: Pat<(int_hexagon_V6_vsubhq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2886:          (V6_vsubhq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2887: def: Pat<(int_hexagon_V6_vsubhq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2888:          (V6_vsubhq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2889: def: Pat<(int_hexagon_V6_vsubhsat HvxVR:$src1, HvxVR:$src2),
  2890:          (V6_vsubhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2891: def: Pat<(int_hexagon_V6_vsubhsat_128B HvxVR:$src1, HvxVR:$src2),
  2892:          (V6_vsubhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2893: def: Pat<(int_hexagon_V6_vsubhsat_dv HvxWR:$src1, HvxWR:$src2),
  2894:          (V6_vsubhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2895: def: Pat<(int_hexagon_V6_vsubhsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  2896:          (V6_vsubhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2897: def: Pat<(int_hexagon_V6_vsubhw HvxVR:$src1, HvxVR:$src2),
  2898:          (V6_vsubhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2899: def: Pat<(int_hexagon_V6_vsubhw_128B HvxVR:$src1, HvxVR:$src2),
  2900:          (V6_vsubhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2901: def: Pat<(int_hexagon_V6_vsububh HvxVR:$src1, HvxVR:$src2),
  2902:          (V6_vsububh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2903: def: Pat<(int_hexagon_V6_vsububh_128B HvxVR:$src1, HvxVR:$src2),
  2904:          (V6_vsububh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2905: def: Pat<(int_hexagon_V6_vsububsat HvxVR:$src1, HvxVR:$src2),
  2906:          (V6_vsububsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2907: def: Pat<(int_hexagon_V6_vsububsat_128B HvxVR:$src1, HvxVR:$src2),
  2908:          (V6_vsububsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2909: def: Pat<(int_hexagon_V6_vsububsat_dv HvxWR:$src1, HvxWR:$src2),
  2910:          (V6_vsububsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2911: def: Pat<(int_hexagon_V6_vsububsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  2912:          (V6_vsububsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2913: def: Pat<(int_hexagon_V6_vsubuhsat HvxVR:$src1, HvxVR:$src2),
  2914:          (V6_vsubuhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2915: def: Pat<(int_hexagon_V6_vsubuhsat_128B HvxVR:$src1, HvxVR:$src2),
  2916:          (V6_vsubuhsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2917: def: Pat<(int_hexagon_V6_vsubuhsat_dv HvxWR:$src1, HvxWR:$src2),
  2918:          (V6_vsubuhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2919: def: Pat<(int_hexagon_V6_vsubuhsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  2920:          (V6_vsubuhsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2921: def: Pat<(int_hexagon_V6_vsubuhw HvxVR:$src1, HvxVR:$src2),
  2922:          (V6_vsubuhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2923: def: Pat<(int_hexagon_V6_vsubuhw_128B HvxVR:$src1, HvxVR:$src2),
  2924:          (V6_vsubuhw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2925: def: Pat<(int_hexagon_V6_vsubw HvxVR:$src1, HvxVR:$src2),
  2926:          (V6_vsubw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2927: def: Pat<(int_hexagon_V6_vsubw_128B HvxVR:$src1, HvxVR:$src2),
  2928:          (V6_vsubw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2929: def: Pat<(int_hexagon_V6_vsubw_dv HvxWR:$src1, HvxWR:$src2),
  2930:          (V6_vsubw_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2931: def: Pat<(int_hexagon_V6_vsubw_dv_128B HvxWR:$src1, HvxWR:$src2),
  2932:          (V6_vsubw_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2933: def: Pat<(int_hexagon_V6_vsubwnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2934:          (V6_vsubwnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2935: def: Pat<(int_hexagon_V6_vsubwnq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2936:          (V6_vsubwnq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2937: def: Pat<(int_hexagon_V6_vsubwq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2938:          (V6_vsubwq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2939: def: Pat<(int_hexagon_V6_vsubwq_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2940:          (V6_vsubwq HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2941: def: Pat<(int_hexagon_V6_vsubwsat HvxVR:$src1, HvxVR:$src2),
  2942:          (V6_vsubwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2943: def: Pat<(int_hexagon_V6_vsubwsat_128B HvxVR:$src1, HvxVR:$src2),
  2944:          (V6_vsubwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2945: def: Pat<(int_hexagon_V6_vsubwsat_dv HvxWR:$src1, HvxWR:$src2),
  2946:          (V6_vsubwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2947: def: Pat<(int_hexagon_V6_vsubwsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  2948:          (V6_vsubwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2949: def: Pat<(int_hexagon_V6_vswap HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2950:          (V6_vswap HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2951: def: Pat<(int_hexagon_V6_vswap_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  2952:          (V6_vswap HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2953: def: Pat<(int_hexagon_V6_vtmpyb HvxWR:$src1, IntRegs:$src2),
  2954:          (V6_vtmpyb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2955: def: Pat<(int_hexagon_V6_vtmpyb_128B HvxWR:$src1, IntRegs:$src2),
  2956:          (V6_vtmpyb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2957: def: Pat<(int_hexagon_V6_vtmpyb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2958:          (V6_vtmpyb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2959: def: Pat<(int_hexagon_V6_vtmpyb_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2960:          (V6_vtmpyb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2961: def: Pat<(int_hexagon_V6_vtmpybus HvxWR:$src1, IntRegs:$src2),
  2962:          (V6_vtmpybus HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2963: def: Pat<(int_hexagon_V6_vtmpybus_128B HvxWR:$src1, IntRegs:$src2),
  2964:          (V6_vtmpybus HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2965: def: Pat<(int_hexagon_V6_vtmpybus_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2966:          (V6_vtmpybus_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2967: def: Pat<(int_hexagon_V6_vtmpybus_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2968:          (V6_vtmpybus_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2969: def: Pat<(int_hexagon_V6_vtmpyhb HvxWR:$src1, IntRegs:$src2),
  2970:          (V6_vtmpyhb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2971: def: Pat<(int_hexagon_V6_vtmpyhb_128B HvxWR:$src1, IntRegs:$src2),
  2972:          (V6_vtmpyhb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2973: def: Pat<(int_hexagon_V6_vtmpyhb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2974:          (V6_vtmpyhb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX64B]>;
  2975: def: Pat<(int_hexagon_V6_vtmpyhb_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  2976:          (V6_vtmpyhb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV60, UseHVX128B]>;
  2977: def: Pat<(int_hexagon_V6_vunpackb HvxVR:$src1),
  2978:          (V6_vunpackb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2979: def: Pat<(int_hexagon_V6_vunpackb_128B HvxVR:$src1),
  2980:          (V6_vunpackb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2981: def: Pat<(int_hexagon_V6_vunpackh HvxVR:$src1),
  2982:          (V6_vunpackh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2983: def: Pat<(int_hexagon_V6_vunpackh_128B HvxVR:$src1),
  2984:          (V6_vunpackh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2985: def: Pat<(int_hexagon_V6_vunpackob HvxWR:$src1, HvxVR:$src2),
  2986:          (V6_vunpackob HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2987: def: Pat<(int_hexagon_V6_vunpackob_128B HvxWR:$src1, HvxVR:$src2),
  2988:          (V6_vunpackob HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2989: def: Pat<(int_hexagon_V6_vunpackoh HvxWR:$src1, HvxVR:$src2),
  2990:          (V6_vunpackoh HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  2991: def: Pat<(int_hexagon_V6_vunpackoh_128B HvxWR:$src1, HvxVR:$src2),
  2992:          (V6_vunpackoh HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  2993: def: Pat<(int_hexagon_V6_vunpackub HvxVR:$src1),
  2994:          (V6_vunpackub HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2995: def: Pat<(int_hexagon_V6_vunpackub_128B HvxVR:$src1),
  2996:          (V6_vunpackub HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  2997: def: Pat<(int_hexagon_V6_vunpackuh HvxVR:$src1),
  2998:          (V6_vunpackuh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  2999: def: Pat<(int_hexagon_V6_vunpackuh_128B HvxVR:$src1),
  3000:          (V6_vunpackuh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 3001-3250 / 第 3001-3250 行

```tablegen
  3001: def: Pat<(int_hexagon_V6_vxor HvxVR:$src1, HvxVR:$src2),
  3002:          (V6_vxor HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX64B]>;
  3003: def: Pat<(int_hexagon_V6_vxor_128B HvxVR:$src1, HvxVR:$src2),
  3004:          (V6_vxor HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV60, UseHVX128B]>;
  3005: def: Pat<(int_hexagon_V6_vzb HvxVR:$src1),
  3006:          (V6_vzb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  3007: def: Pat<(int_hexagon_V6_vzb_128B HvxVR:$src1),
  3008:          (V6_vzb HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  3009: def: Pat<(int_hexagon_V6_vzh HvxVR:$src1),
  3010:          (V6_vzh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX64B]>;
  3011: def: Pat<(int_hexagon_V6_vzh_128B HvxVR:$src1),
  3012:          (V6_vzh HvxVR:$src1)>, Requires<[UseHVXV60, UseHVX128B]>;
  3013: 
  3014: // V62 HVX Instructions.
  3015: 
  3016: def: Pat<(int_hexagon_V6_lvsplatb IntRegs:$src1),
  3017:          (V6_lvsplatb IntRegs:$src1)>, Requires<[UseHVXV62, UseHVX64B]>;
  3018: def: Pat<(int_hexagon_V6_lvsplatb_128B IntRegs:$src1),
  3019:          (V6_lvsplatb IntRegs:$src1)>, Requires<[UseHVXV62, UseHVX128B]>;
  3020: def: Pat<(int_hexagon_V6_lvsplath IntRegs:$src1),
  3021:          (V6_lvsplath IntRegs:$src1)>, Requires<[UseHVXV62, UseHVX64B]>;
  3022: def: Pat<(int_hexagon_V6_lvsplath_128B IntRegs:$src1),
  3023:          (V6_lvsplath IntRegs:$src1)>, Requires<[UseHVXV62, UseHVX128B]>;
  3024: def: Pat<(int_hexagon_V6_pred_scalar2v2 IntRegs:$src1),
  3025:          (V6_pred_scalar2v2 IntRegs:$src1)>, Requires<[UseHVXV62, UseHVX64B]>;
  3026: def: Pat<(int_hexagon_V6_pred_scalar2v2_128B IntRegs:$src1),
  3027:          (V6_pred_scalar2v2 IntRegs:$src1)>, Requires<[UseHVXV62, UseHVX128B]>;
  3028: def: Pat<(int_hexagon_V6_shuffeqh HvxQR:$src1, HvxQR:$src2),
  3029:          (V6_shuffeqh HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3030: def: Pat<(int_hexagon_V6_shuffeqh_128B HvxQR:$src1, HvxQR:$src2),
  3031:          (V6_shuffeqh HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3032: def: Pat<(int_hexagon_V6_shuffeqw HvxQR:$src1, HvxQR:$src2),
  3033:          (V6_shuffeqw HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3034: def: Pat<(int_hexagon_V6_shuffeqw_128B HvxQR:$src1, HvxQR:$src2),
  3035:          (V6_shuffeqw HvxQR:$src1, HvxQR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3036: def: Pat<(int_hexagon_V6_vaddbsat HvxVR:$src1, HvxVR:$src2),
  3037:          (V6_vaddbsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3038: def: Pat<(int_hexagon_V6_vaddbsat_128B HvxVR:$src1, HvxVR:$src2),
  3039:          (V6_vaddbsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3040: def: Pat<(int_hexagon_V6_vaddbsat_dv HvxWR:$src1, HvxWR:$src2),
  3041:          (V6_vaddbsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3042: def: Pat<(int_hexagon_V6_vaddbsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  3043:          (V6_vaddbsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3044: def: Pat<(int_hexagon_V6_vaddcarry HvxVR:$src1, HvxVR:$src2, HvxQR:$src3),
  3045:          (V6_vaddcarry HvxVR:$src1, HvxVR:$src2, HvxQR:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3046: def: Pat<(int_hexagon_V6_vaddcarry_128B HvxVR:$src1, HvxVR:$src2, HvxQR:$src3),
  3047:          (V6_vaddcarry HvxVR:$src1, HvxVR:$src2, HvxQR:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3048: def: Pat<(int_hexagon_V6_vaddclbh HvxVR:$src1, HvxVR:$src2),
  3049:          (V6_vaddclbh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3050: def: Pat<(int_hexagon_V6_vaddclbh_128B HvxVR:$src1, HvxVR:$src2),
  3051:          (V6_vaddclbh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3052: def: Pat<(int_hexagon_V6_vaddclbw HvxVR:$src1, HvxVR:$src2),
  3053:          (V6_vaddclbw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3054: def: Pat<(int_hexagon_V6_vaddclbw_128B HvxVR:$src1, HvxVR:$src2),
  3055:          (V6_vaddclbw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3056: def: Pat<(int_hexagon_V6_vaddhw_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3057:          (V6_vaddhw_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3058: def: Pat<(int_hexagon_V6_vaddhw_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3059:          (V6_vaddhw_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3060: def: Pat<(int_hexagon_V6_vaddubh_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3061:          (V6_vaddubh_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3062: def: Pat<(int_hexagon_V6_vaddubh_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3063:          (V6_vaddubh_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3064: def: Pat<(int_hexagon_V6_vaddububb_sat HvxVR:$src1, HvxVR:$src2),
  3065:          (V6_vaddububb_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3066: def: Pat<(int_hexagon_V6_vaddububb_sat_128B HvxVR:$src1, HvxVR:$src2),
  3067:          (V6_vaddububb_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3068: def: Pat<(int_hexagon_V6_vadduhw_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3069:          (V6_vadduhw_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3070: def: Pat<(int_hexagon_V6_vadduhw_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3071:          (V6_vadduhw_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3072: def: Pat<(int_hexagon_V6_vadduwsat HvxVR:$src1, HvxVR:$src2),
  3073:          (V6_vadduwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3074: def: Pat<(int_hexagon_V6_vadduwsat_128B HvxVR:$src1, HvxVR:$src2),
  3075:          (V6_vadduwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3076: def: Pat<(int_hexagon_V6_vadduwsat_dv HvxWR:$src1, HvxWR:$src2),
  3077:          (V6_vadduwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3078: def: Pat<(int_hexagon_V6_vadduwsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  3079:          (V6_vadduwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3080: def: Pat<(int_hexagon_V6_vandnqrt HvxQR:$src1, IntRegs:$src2),
  3081:          (V6_vandnqrt HvxQR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3082: def: Pat<(int_hexagon_V6_vandnqrt_128B HvxQR:$src1, IntRegs:$src2),
  3083:          (V6_vandnqrt HvxQR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3084: def: Pat<(int_hexagon_V6_vandnqrt_acc HvxVR:$src1, HvxQR:$src2, IntRegs:$src3),
  3085:          (V6_vandnqrt_acc HvxVR:$src1, HvxQR:$src2, IntRegs:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3086: def: Pat<(int_hexagon_V6_vandnqrt_acc_128B HvxVR:$src1, HvxQR:$src2, IntRegs:$src3),
  3087:          (V6_vandnqrt_acc HvxVR:$src1, HvxQR:$src2, IntRegs:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3088: def: Pat<(int_hexagon_V6_vandvnqv HvxQR:$src1, HvxVR:$src2),
  3089:          (V6_vandvnqv HvxQR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3090: def: Pat<(int_hexagon_V6_vandvnqv_128B HvxQR:$src1, HvxVR:$src2),
  3091:          (V6_vandvnqv HvxQR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3092: def: Pat<(int_hexagon_V6_vandvqv HvxQR:$src1, HvxVR:$src2),
  3093:          (V6_vandvqv HvxQR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3094: def: Pat<(int_hexagon_V6_vandvqv_128B HvxQR:$src1, HvxVR:$src2),
  3095:          (V6_vandvqv HvxQR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3096: def: Pat<(int_hexagon_V6_vasrhbsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3097:          (V6_vasrhbsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3098: def: Pat<(int_hexagon_V6_vasrhbsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3099:          (V6_vasrhbsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3100: def: Pat<(int_hexagon_V6_vasruwuhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3101:          (V6_vasruwuhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3102: def: Pat<(int_hexagon_V6_vasruwuhrndsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3103:          (V6_vasruwuhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3104: def: Pat<(int_hexagon_V6_vasrwuhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3105:          (V6_vasrwuhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3106: def: Pat<(int_hexagon_V6_vasrwuhrndsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3107:          (V6_vasrwuhrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3108: def: Pat<(int_hexagon_V6_vlsrb HvxVR:$src1, IntRegs:$src2),
  3109:          (V6_vlsrb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3110: def: Pat<(int_hexagon_V6_vlsrb_128B HvxVR:$src1, IntRegs:$src2),
  3111:          (V6_vlsrb HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3112: def: Pat<(int_hexagon_V6_vlutvvb_nm HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3113:          (V6_vlutvvb_nm HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3114: def: Pat<(int_hexagon_V6_vlutvvb_nm_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3115:          (V6_vlutvvb_nm HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3116: def: Pat<(int_hexagon_V6_vlutvvb_oracci HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4),
  3117:          (V6_vlutvvb_oracci HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4)>, Requires<[UseHVXV62, UseHVX64B]>;
  3118: def: Pat<(int_hexagon_V6_vlutvvb_oracci_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4),
  3119:          (V6_vlutvvb_oracci HvxVR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4)>, Requires<[UseHVXV62, UseHVX128B]>;
  3120: def: Pat<(int_hexagon_V6_vlutvvbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  3121:          (V6_vlutvvbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3122: def: Pat<(int_hexagon_V6_vlutvvbi_128B HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  3123:          (V6_vlutvvbi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3124: def: Pat<(int_hexagon_V6_vlutvwh_nm HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3125:          (V6_vlutvwh_nm HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3126: def: Pat<(int_hexagon_V6_vlutvwh_nm_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3127:          (V6_vlutvwh_nm HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3128: def: Pat<(int_hexagon_V6_vlutvwh_oracci HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4),
  3129:          (V6_vlutvwh_oracci HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4)>, Requires<[UseHVXV62, UseHVX64B]>;
  3130: def: Pat<(int_hexagon_V6_vlutvwh_oracci_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4),
  3131:          (V6_vlutvwh_oracci HvxWR:$src1, HvxVR:$src2, HvxVR:$src3, u3_0ImmPred_timm:$src4)>, Requires<[UseHVXV62, UseHVX128B]>;
  3132: def: Pat<(int_hexagon_V6_vlutvwhi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  3133:          (V6_vlutvwhi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3134: def: Pat<(int_hexagon_V6_vlutvwhi_128B HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3),
  3135:          (V6_vlutvwhi HvxVR:$src1, HvxVR:$src2, u3_0ImmPred_timm:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3136: def: Pat<(int_hexagon_V6_vmaxb HvxVR:$src1, HvxVR:$src2),
  3137:          (V6_vmaxb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3138: def: Pat<(int_hexagon_V6_vmaxb_128B HvxVR:$src1, HvxVR:$src2),
  3139:          (V6_vmaxb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3140: def: Pat<(int_hexagon_V6_vminb HvxVR:$src1, HvxVR:$src2),
  3141:          (V6_vminb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3142: def: Pat<(int_hexagon_V6_vminb_128B HvxVR:$src1, HvxVR:$src2),
  3143:          (V6_vminb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3144: def: Pat<(int_hexagon_V6_vmpauhb HvxWR:$src1, IntRegs:$src2),
  3145:          (V6_vmpauhb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3146: def: Pat<(int_hexagon_V6_vmpauhb_128B HvxWR:$src1, IntRegs:$src2),
  3147:          (V6_vmpauhb HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3148: def: Pat<(int_hexagon_V6_vmpauhb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  3149:          (V6_vmpauhb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3150: def: Pat<(int_hexagon_V6_vmpauhb_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  3151:          (V6_vmpauhb_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3152: def: Pat<(int_hexagon_V6_vmpyewuh_64 HvxVR:$src1, HvxVR:$src2),
  3153:          (V6_vmpyewuh_64 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3154: def: Pat<(int_hexagon_V6_vmpyewuh_64_128B HvxVR:$src1, HvxVR:$src2),
  3155:          (V6_vmpyewuh_64 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3156: def: Pat<(int_hexagon_V6_vmpyiwub HvxVR:$src1, IntRegs:$src2),
  3157:          (V6_vmpyiwub HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3158: def: Pat<(int_hexagon_V6_vmpyiwub_128B HvxVR:$src1, IntRegs:$src2),
  3159:          (V6_vmpyiwub HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3160: def: Pat<(int_hexagon_V6_vmpyiwub_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3161:          (V6_vmpyiwub_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3162: def: Pat<(int_hexagon_V6_vmpyiwub_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3163:          (V6_vmpyiwub_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3164: def: Pat<(int_hexagon_V6_vmpyowh_64_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3165:          (V6_vmpyowh_64_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3166: def: Pat<(int_hexagon_V6_vmpyowh_64_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3167:          (V6_vmpyowh_64_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3168: def: Pat<(int_hexagon_V6_vrounduhub HvxVR:$src1, HvxVR:$src2),
  3169:          (V6_vrounduhub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3170: def: Pat<(int_hexagon_V6_vrounduhub_128B HvxVR:$src1, HvxVR:$src2),
  3171:          (V6_vrounduhub HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3172: def: Pat<(int_hexagon_V6_vrounduwuh HvxVR:$src1, HvxVR:$src2),
  3173:          (V6_vrounduwuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3174: def: Pat<(int_hexagon_V6_vrounduwuh_128B HvxVR:$src1, HvxVR:$src2),
  3175:          (V6_vrounduwuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3176: def: Pat<(int_hexagon_V6_vsatuwuh HvxVR:$src1, HvxVR:$src2),
  3177:          (V6_vsatuwuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3178: def: Pat<(int_hexagon_V6_vsatuwuh_128B HvxVR:$src1, HvxVR:$src2),
  3179:          (V6_vsatuwuh HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3180: def: Pat<(int_hexagon_V6_vsubbsat HvxVR:$src1, HvxVR:$src2),
  3181:          (V6_vsubbsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3182: def: Pat<(int_hexagon_V6_vsubbsat_128B HvxVR:$src1, HvxVR:$src2),
  3183:          (V6_vsubbsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3184: def: Pat<(int_hexagon_V6_vsubbsat_dv HvxWR:$src1, HvxWR:$src2),
  3185:          (V6_vsubbsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3186: def: Pat<(int_hexagon_V6_vsubbsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  3187:          (V6_vsubbsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3188: def: Pat<(int_hexagon_V6_vsubcarry HvxVR:$src1, HvxVR:$src2, HvxQR:$src3),
  3189:          (V6_vsubcarry HvxVR:$src1, HvxVR:$src2, HvxQR:$src3)>, Requires<[UseHVXV62, UseHVX64B]>;
  3190: def: Pat<(int_hexagon_V6_vsubcarry_128B HvxVR:$src1, HvxVR:$src2, HvxQR:$src3),
  3191:          (V6_vsubcarry HvxVR:$src1, HvxVR:$src2, HvxQR:$src3)>, Requires<[UseHVXV62, UseHVX128B]>;
  3192: def: Pat<(int_hexagon_V6_vsubububb_sat HvxVR:$src1, HvxVR:$src2),
  3193:          (V6_vsubububb_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3194: def: Pat<(int_hexagon_V6_vsubububb_sat_128B HvxVR:$src1, HvxVR:$src2),
  3195:          (V6_vsubububb_sat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3196: def: Pat<(int_hexagon_V6_vsubuwsat HvxVR:$src1, HvxVR:$src2),
  3197:          (V6_vsubuwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3198: def: Pat<(int_hexagon_V6_vsubuwsat_128B HvxVR:$src1, HvxVR:$src2),
  3199:          (V6_vsubuwsat HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3200: def: Pat<(int_hexagon_V6_vsubuwsat_dv HvxWR:$src1, HvxWR:$src2),
  3201:          (V6_vsubuwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX64B]>;
  3202: def: Pat<(int_hexagon_V6_vsubuwsat_dv_128B HvxWR:$src1, HvxWR:$src2),
  3203:          (V6_vsubuwsat_dv HvxWR:$src1, HvxWR:$src2)>, Requires<[UseHVXV62, UseHVX128B]>;
  3204: 
  3205: // V65 HVX Instructions.
  3206: 
  3207: def: Pat<(int_hexagon_V6_vabsb HvxVR:$src1),
  3208:          (V6_vabsb HvxVR:$src1)>, Requires<[UseHVXV65, UseHVX64B]>;
  3209: def: Pat<(int_hexagon_V6_vabsb_128B HvxVR:$src1),
  3210:          (V6_vabsb HvxVR:$src1)>, Requires<[UseHVXV65, UseHVX128B]>;
  3211: def: Pat<(int_hexagon_V6_vabsb_sat HvxVR:$src1),
  3212:          (V6_vabsb_sat HvxVR:$src1)>, Requires<[UseHVXV65, UseHVX64B]>;
  3213: def: Pat<(int_hexagon_V6_vabsb_sat_128B HvxVR:$src1),
  3214:          (V6_vabsb_sat HvxVR:$src1)>, Requires<[UseHVXV65, UseHVX128B]>;
  3215: def: Pat<(int_hexagon_V6_vaslh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3216:          (V6_vaslh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3217: def: Pat<(int_hexagon_V6_vaslh_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3218:          (V6_vaslh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3219: def: Pat<(int_hexagon_V6_vasrh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3220:          (V6_vasrh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3221: def: Pat<(int_hexagon_V6_vasrh_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3222:          (V6_vasrh_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3223: def: Pat<(int_hexagon_V6_vasruhubrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3224:          (V6_vasruhubrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3225: def: Pat<(int_hexagon_V6_vasruhubrndsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3226:          (V6_vasruhubrndsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3227: def: Pat<(int_hexagon_V6_vasruhubsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3228:          (V6_vasruhubsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3229: def: Pat<(int_hexagon_V6_vasruhubsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3230:          (V6_vasruhubsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3231: def: Pat<(int_hexagon_V6_vasruwuhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3232:          (V6_vasruwuhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3233: def: Pat<(int_hexagon_V6_vasruwuhsat_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3234:          (V6_vasruwuhsat HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3235: def: Pat<(int_hexagon_V6_vavgb HvxVR:$src1, HvxVR:$src2),
  3236:          (V6_vavgb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3237: def: Pat<(int_hexagon_V6_vavgb_128B HvxVR:$src1, HvxVR:$src2),
  3238:          (V6_vavgb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
  3239: def: Pat<(int_hexagon_V6_vavgbrnd HvxVR:$src1, HvxVR:$src2),
  3240:          (V6_vavgbrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3241: def: Pat<(int_hexagon_V6_vavgbrnd_128B HvxVR:$src1, HvxVR:$src2),
  3242:          (V6_vavgbrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
  3243: def: Pat<(int_hexagon_V6_vavguw HvxVR:$src1, HvxVR:$src2),
  3244:          (V6_vavguw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3245: def: Pat<(int_hexagon_V6_vavguw_128B HvxVR:$src1, HvxVR:$src2),
  3246:          (V6_vavguw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
  3247: def: Pat<(int_hexagon_V6_vavguwrnd HvxVR:$src1, HvxVR:$src2),
  3248:          (V6_vavguwrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3249: def: Pat<(int_hexagon_V6_vavguwrnd_128B HvxVR:$src1, HvxVR:$src2),
  3250:          (V6_vavguwrnd HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 3251-3500 / 第 3251-3500 行

```tablegen
  3251: def: Pat<(int_hexagon_V6_vdd0 ),
  3252:          (V6_vdd0 )>, Requires<[UseHVXV65, UseHVX64B]>;
  3253: def: Pat<(int_hexagon_V6_vdd0_128B ),
  3254:          (V6_vdd0 )>, Requires<[UseHVXV65, UseHVX128B]>;
  3255: def: Pat<(int_hexagon_V6_vlut4 HvxVR:$src1, DoubleRegs:$src2),
  3256:          (V6_vlut4 HvxVR:$src1, DoubleRegs:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3257: def: Pat<(int_hexagon_V6_vlut4_128B HvxVR:$src1, DoubleRegs:$src2),
  3258:          (V6_vlut4 HvxVR:$src1, DoubleRegs:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
  3259: def: Pat<(int_hexagon_V6_vmpabuu HvxWR:$src1, IntRegs:$src2),
  3260:          (V6_vmpabuu HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3261: def: Pat<(int_hexagon_V6_vmpabuu_128B HvxWR:$src1, IntRegs:$src2),
  3262:          (V6_vmpabuu HvxWR:$src1, IntRegs:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
  3263: def: Pat<(int_hexagon_V6_vmpabuu_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  3264:          (V6_vmpabuu_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3265: def: Pat<(int_hexagon_V6_vmpabuu_acc_128B HvxWR:$src1, HvxWR:$src2, IntRegs:$src3),
  3266:          (V6_vmpabuu_acc HvxWR:$src1, HvxWR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3267: def: Pat<(int_hexagon_V6_vmpahhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3),
  3268:          (V6_vmpahhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3269: def: Pat<(int_hexagon_V6_vmpahhsat_128B HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3),
  3270:          (V6_vmpahhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3271: def: Pat<(int_hexagon_V6_vmpauhuhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3),
  3272:          (V6_vmpauhuhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3273: def: Pat<(int_hexagon_V6_vmpauhuhsat_128B HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3),
  3274:          (V6_vmpauhuhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3275: def: Pat<(int_hexagon_V6_vmpsuhuhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3),
  3276:          (V6_vmpsuhuhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3277: def: Pat<(int_hexagon_V6_vmpsuhuhsat_128B HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3),
  3278:          (V6_vmpsuhuhsat HvxVR:$src1, HvxVR:$src2, DoubleRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3279: def: Pat<(int_hexagon_V6_vmpyh_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  3280:          (V6_vmpyh_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3281: def: Pat<(int_hexagon_V6_vmpyh_acc_128B HvxWR:$src1, HvxVR:$src2, IntRegs:$src3),
  3282:          (V6_vmpyh_acc HvxWR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3283: def: Pat<(int_hexagon_V6_vmpyuhe HvxVR:$src1, IntRegs:$src2),
  3284:          (V6_vmpyuhe HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3285: def: Pat<(int_hexagon_V6_vmpyuhe_128B HvxVR:$src1, IntRegs:$src2),
  3286:          (V6_vmpyuhe HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
  3287: def: Pat<(int_hexagon_V6_vmpyuhe_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3288:          (V6_vmpyuhe_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX64B]>;
  3289: def: Pat<(int_hexagon_V6_vmpyuhe_acc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3290:          (V6_vmpyuhe_acc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV65, UseHVX128B]>;
  3291: def: Pat<(int_hexagon_V6_vnavgb HvxVR:$src1, HvxVR:$src2),
  3292:          (V6_vnavgb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX64B]>;
  3293: def: Pat<(int_hexagon_V6_vnavgb_128B HvxVR:$src1, HvxVR:$src2),
  3294:          (V6_vnavgb HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV65, UseHVX128B]>;
  3295: def: Pat<(int_hexagon_V6_vprefixqb HvxQR:$src1),
  3296:          (V6_vprefixqb HvxQR:$src1)>, Requires<[UseHVXV65, UseHVX64B]>;
  3297: def: Pat<(int_hexagon_V6_vprefixqb_128B HvxQR:$src1),
  3298:          (V6_vprefixqb HvxQR:$src1)>, Requires<[UseHVXV65, UseHVX128B]>;
  3299: def: Pat<(int_hexagon_V6_vprefixqh HvxQR:$src1),
  3300:          (V6_vprefixqh HvxQR:$src1)>, Requires<[UseHVXV65, UseHVX64B]>;
  3301: def: Pat<(int_hexagon_V6_vprefixqh_128B HvxQR:$src1),
  3302:          (V6_vprefixqh HvxQR:$src1)>, Requires<[UseHVXV65, UseHVX128B]>;
  3303: def: Pat<(int_hexagon_V6_vprefixqw HvxQR:$src1),
  3304:          (V6_vprefixqw HvxQR:$src1)>, Requires<[UseHVXV65, UseHVX64B]>;
  3305: def: Pat<(int_hexagon_V6_vprefixqw_128B HvxQR:$src1),
  3306:          (V6_vprefixqw HvxQR:$src1)>, Requires<[UseHVXV65, UseHVX128B]>;
  3307: def: Pat<(int_hexagon_V6_vscattermh IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3308:          (V6_vscattermh IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX64B]>;
  3309: def: Pat<(int_hexagon_V6_vscattermh_128B IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3310:          (V6_vscattermh IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX128B]>;
  3311: def: Pat<(int_hexagon_V6_vscattermh_add IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3312:          (V6_vscattermh_add IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX64B]>;
  3313: def: Pat<(int_hexagon_V6_vscattermh_add_128B IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3314:          (V6_vscattermh_add IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX128B]>;
  3315: def: Pat<(int_hexagon_V6_vscattermhq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5),
  3316:          (V6_vscattermhq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5)>, Requires<[UseHVXV65, UseHVX64B]>;
  3317: def: Pat<(int_hexagon_V6_vscattermhq_128B HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5),
  3318:          (V6_vscattermhq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5)>, Requires<[UseHVXV65, UseHVX128B]>;
  3319: def: Pat<(int_hexagon_V6_vscattermhw IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4),
  3320:          (V6_vscattermhw IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX64B]>;
  3321: def: Pat<(int_hexagon_V6_vscattermhw_128B IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4),
  3322:          (V6_vscattermhw IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX128B]>;
  3323: def: Pat<(int_hexagon_V6_vscattermhw_add IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4),
  3324:          (V6_vscattermhw_add IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX64B]>;
  3325: def: Pat<(int_hexagon_V6_vscattermhw_add_128B IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4),
  3326:          (V6_vscattermhw_add IntRegs:$src1, ModRegs:$src2, HvxWR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX128B]>;
  3327: def: Pat<(int_hexagon_V6_vscattermhwq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxWR:$src4, HvxVR:$src5),
  3328:          (V6_vscattermhwq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxWR:$src4, HvxVR:$src5)>, Requires<[UseHVXV65, UseHVX64B]>;
  3329: def: Pat<(int_hexagon_V6_vscattermhwq_128B HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxWR:$src4, HvxVR:$src5),
  3330:          (V6_vscattermhwq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxWR:$src4, HvxVR:$src5)>, Requires<[UseHVXV65, UseHVX128B]>;
  3331: def: Pat<(int_hexagon_V6_vscattermw IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3332:          (V6_vscattermw IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX64B]>;
  3333: def: Pat<(int_hexagon_V6_vscattermw_128B IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3334:          (V6_vscattermw IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX128B]>;
  3335: def: Pat<(int_hexagon_V6_vscattermw_add IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3336:          (V6_vscattermw_add IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX64B]>;
  3337: def: Pat<(int_hexagon_V6_vscattermw_add_128B IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4),
  3338:          (V6_vscattermw_add IntRegs:$src1, ModRegs:$src2, HvxVR:$src3, HvxVR:$src4)>, Requires<[UseHVXV65, UseHVX128B]>;
  3339: def: Pat<(int_hexagon_V6_vscattermwq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5),
  3340:          (V6_vscattermwq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5)>, Requires<[UseHVXV65, UseHVX64B]>;
  3341: def: Pat<(int_hexagon_V6_vscattermwq_128B HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5),
  3342:          (V6_vscattermwq HvxQR:$src1, IntRegs:$src2, ModRegs:$src3, HvxVR:$src4, HvxVR:$src5)>, Requires<[UseHVXV65, UseHVX128B]>;
  3343: 
  3344: // V66 HVX Instructions.
  3345: 
  3346: def: Pat<(int_hexagon_V6_vaddcarryo HvxVR:$src1, HvxVR:$src2),
  3347:          (V6_vaddcarryo HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX64B]>;
  3348: def: Pat<(int_hexagon_V6_vaddcarryo_128B HvxVR:$src1, HvxVR:$src2),
  3349:          (V6_vaddcarryo HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX128B]>;
  3350: def: Pat<(int_hexagon_V6_vaddcarrysat HvxVR:$src1, HvxVR:$src2, HvxQR:$src3),
  3351:          (V6_vaddcarrysat HvxVR:$src1, HvxVR:$src2, HvxQR:$src3)>, Requires<[UseHVXV66, UseHVX64B]>;
  3352: def: Pat<(int_hexagon_V6_vaddcarrysat_128B HvxVR:$src1, HvxVR:$src2, HvxQR:$src3),
  3353:          (V6_vaddcarrysat HvxVR:$src1, HvxVR:$src2, HvxQR:$src3)>, Requires<[UseHVXV66, UseHVX128B]>;
  3354: def: Pat<(int_hexagon_V6_vasr_into HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3355:          (V6_vasr_into HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV66, UseHVX64B]>;
  3356: def: Pat<(int_hexagon_V6_vasr_into_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3357:          (V6_vasr_into HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV66, UseHVX128B]>;
  3358: def: Pat<(int_hexagon_V6_vrotr HvxVR:$src1, HvxVR:$src2),
  3359:          (V6_vrotr HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX64B]>;
  3360: def: Pat<(int_hexagon_V6_vrotr_128B HvxVR:$src1, HvxVR:$src2),
  3361:          (V6_vrotr HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX128B]>;
  3362: def: Pat<(int_hexagon_V6_vsatdw HvxVR:$src1, HvxVR:$src2),
  3363:          (V6_vsatdw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX64B]>;
  3364: def: Pat<(int_hexagon_V6_vsatdw_128B HvxVR:$src1, HvxVR:$src2),
  3365:          (V6_vsatdw HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX128B]>;
  3366: def: Pat<(int_hexagon_V6_vsubcarryo HvxVR:$src1, HvxVR:$src2),
  3367:          (V6_vsubcarryo HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX64B]>;
  3368: def: Pat<(int_hexagon_V6_vsubcarryo_128B HvxVR:$src1, HvxVR:$src2),
  3369:          (V6_vsubcarryo HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV66, UseHVX128B]>;
  3370: 
  3371: // V68 HVX Instructions.
  3372: 
  3373: def: Pat<(int_hexagon_V6_v6mpyhubs10 HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3),
  3374:          (V6_v6mpyhubs10 HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3)>, Requires<[UseHVXV68, UseHVX64B]>;
  3375: def: Pat<(int_hexagon_V6_v6mpyhubs10_128B HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3),
  3376:          (V6_v6mpyhubs10 HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3)>, Requires<[UseHVXV68, UseHVX128B]>;
  3377: def: Pat<(int_hexagon_V6_v6mpyhubs10_vxx HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4),
  3378:          (V6_v6mpyhubs10_vxx HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4)>, Requires<[UseHVXV68, UseHVX64B]>;
  3379: def: Pat<(int_hexagon_V6_v6mpyhubs10_vxx_128B HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4),
  3380:          (V6_v6mpyhubs10_vxx HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4)>, Requires<[UseHVXV68, UseHVX128B]>;
  3381: def: Pat<(int_hexagon_V6_v6mpyvubs10 HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3),
  3382:          (V6_v6mpyvubs10 HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3)>, Requires<[UseHVXV68, UseHVX64B]>;
  3383: def: Pat<(int_hexagon_V6_v6mpyvubs10_128B HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3),
  3384:          (V6_v6mpyvubs10 HvxWR:$src1, HvxWR:$src2, u2_0ImmPred_timm:$src3)>, Requires<[UseHVXV68, UseHVX128B]>;
  3385: def: Pat<(int_hexagon_V6_v6mpyvubs10_vxx HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4),
  3386:          (V6_v6mpyvubs10_vxx HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4)>, Requires<[UseHVXV68, UseHVX64B]>;
  3387: def: Pat<(int_hexagon_V6_v6mpyvubs10_vxx_128B HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4),
  3388:          (V6_v6mpyvubs10_vxx HvxWR:$src1, HvxWR:$src2, HvxWR:$src3, u2_0ImmPred_timm:$src4)>, Requires<[UseHVXV68, UseHVX128B]>;
  3389: def: Pat<(int_hexagon_V6_vabs_hf HvxVR:$src1),
  3390:          (V6_vabs_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3391: def: Pat<(int_hexagon_V6_vabs_hf_128B HvxVR:$src1),
  3392:          (V6_vabs_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3393: def: Pat<(int_hexagon_V6_vabs_sf HvxVR:$src1),
  3394:          (V6_vabs_sf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3395: def: Pat<(int_hexagon_V6_vabs_sf_128B HvxVR:$src1),
  3396:          (V6_vabs_sf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3397: def: Pat<(int_hexagon_V6_vadd_hf HvxVR:$src1, HvxVR:$src2),
  3398:          (V6_vadd_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3399: def: Pat<(int_hexagon_V6_vadd_hf_128B HvxVR:$src1, HvxVR:$src2),
  3400:          (V6_vadd_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3401: def: Pat<(int_hexagon_V6_vadd_hf_hf HvxVR:$src1, HvxVR:$src2),
  3402:          (V6_vadd_hf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3403: def: Pat<(int_hexagon_V6_vadd_hf_hf_128B HvxVR:$src1, HvxVR:$src2),
  3404:          (V6_vadd_hf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3405: def: Pat<(int_hexagon_V6_vadd_qf16 HvxVR:$src1, HvxVR:$src2),
  3406:          (V6_vadd_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3407: def: Pat<(int_hexagon_V6_vadd_qf16_128B HvxVR:$src1, HvxVR:$src2),
  3408:          (V6_vadd_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3409: def: Pat<(int_hexagon_V6_vadd_qf16_mix HvxVR:$src1, HvxVR:$src2),
  3410:          (V6_vadd_qf16_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3411: def: Pat<(int_hexagon_V6_vadd_qf16_mix_128B HvxVR:$src1, HvxVR:$src2),
  3412:          (V6_vadd_qf16_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3413: def: Pat<(int_hexagon_V6_vadd_qf32 HvxVR:$src1, HvxVR:$src2),
  3414:          (V6_vadd_qf32 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3415: def: Pat<(int_hexagon_V6_vadd_qf32_128B HvxVR:$src1, HvxVR:$src2),
  3416:          (V6_vadd_qf32 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3417: def: Pat<(int_hexagon_V6_vadd_qf32_mix HvxVR:$src1, HvxVR:$src2),
  3418:          (V6_vadd_qf32_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3419: def: Pat<(int_hexagon_V6_vadd_qf32_mix_128B HvxVR:$src1, HvxVR:$src2),
  3420:          (V6_vadd_qf32_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3421: def: Pat<(int_hexagon_V6_vadd_sf HvxVR:$src1, HvxVR:$src2),
  3422:          (V6_vadd_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3423: def: Pat<(int_hexagon_V6_vadd_sf_128B HvxVR:$src1, HvxVR:$src2),
  3424:          (V6_vadd_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3425: def: Pat<(int_hexagon_V6_vadd_sf_hf HvxVR:$src1, HvxVR:$src2),
  3426:          (V6_vadd_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3427: def: Pat<(int_hexagon_V6_vadd_sf_hf_128B HvxVR:$src1, HvxVR:$src2),
  3428:          (V6_vadd_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3429: def: Pat<(int_hexagon_V6_vadd_sf_sf HvxVR:$src1, HvxVR:$src2),
  3430:          (V6_vadd_sf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3431: def: Pat<(int_hexagon_V6_vadd_sf_sf_128B HvxVR:$src1, HvxVR:$src2),
  3432:          (V6_vadd_sf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3433: def: Pat<(int_hexagon_V6_vassign_fp HvxVR:$src1),
  3434:          (V6_vassign_fp HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3435: def: Pat<(int_hexagon_V6_vassign_fp_128B HvxVR:$src1),
  3436:          (V6_vassign_fp HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3437: def: Pat<(int_hexagon_V6_vconv_hf_qf16 HvxVR:$src1),
  3438:          (V6_vconv_hf_qf16 HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3439: def: Pat<(int_hexagon_V6_vconv_hf_qf16_128B HvxVR:$src1),
  3440:          (V6_vconv_hf_qf16 HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3441: def: Pat<(int_hexagon_V6_vconv_hf_qf32 HvxWR:$src1),
  3442:          (V6_vconv_hf_qf32 HvxWR:$src1)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3443: def: Pat<(int_hexagon_V6_vconv_hf_qf32_128B HvxWR:$src1),
  3444:          (V6_vconv_hf_qf32 HvxWR:$src1)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3445: def: Pat<(int_hexagon_V6_vconv_sf_qf32 HvxVR:$src1),
  3446:          (V6_vconv_sf_qf32 HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3447: def: Pat<(int_hexagon_V6_vconv_sf_qf32_128B HvxVR:$src1),
  3448:          (V6_vconv_sf_qf32 HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3449: def: Pat<(int_hexagon_V6_vcvt_b_hf HvxVR:$src1, HvxVR:$src2),
  3450:          (V6_vcvt_b_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3451: def: Pat<(int_hexagon_V6_vcvt_b_hf_128B HvxVR:$src1, HvxVR:$src2),
  3452:          (V6_vcvt_b_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3453: def: Pat<(int_hexagon_V6_vcvt_h_hf HvxVR:$src1),
  3454:          (V6_vcvt_h_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3455: def: Pat<(int_hexagon_V6_vcvt_h_hf_128B HvxVR:$src1),
  3456:          (V6_vcvt_h_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3457: def: Pat<(int_hexagon_V6_vcvt_hf_b HvxVR:$src1),
  3458:          (V6_vcvt_hf_b HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3459: def: Pat<(int_hexagon_V6_vcvt_hf_b_128B HvxVR:$src1),
  3460:          (V6_vcvt_hf_b HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3461: def: Pat<(int_hexagon_V6_vcvt_hf_h HvxVR:$src1),
  3462:          (V6_vcvt_hf_h HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3463: def: Pat<(int_hexagon_V6_vcvt_hf_h_128B HvxVR:$src1),
  3464:          (V6_vcvt_hf_h HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3465: def: Pat<(int_hexagon_V6_vcvt_hf_sf HvxVR:$src1, HvxVR:$src2),
  3466:          (V6_vcvt_hf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3467: def: Pat<(int_hexagon_V6_vcvt_hf_sf_128B HvxVR:$src1, HvxVR:$src2),
  3468:          (V6_vcvt_hf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3469: def: Pat<(int_hexagon_V6_vcvt_hf_ub HvxVR:$src1),
  3470:          (V6_vcvt_hf_ub HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3471: def: Pat<(int_hexagon_V6_vcvt_hf_ub_128B HvxVR:$src1),
  3472:          (V6_vcvt_hf_ub HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3473: def: Pat<(int_hexagon_V6_vcvt_hf_uh HvxVR:$src1),
  3474:          (V6_vcvt_hf_uh HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3475: def: Pat<(int_hexagon_V6_vcvt_hf_uh_128B HvxVR:$src1),
  3476:          (V6_vcvt_hf_uh HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3477: def: Pat<(int_hexagon_V6_vcvt_sf_hf HvxVR:$src1),
  3478:          (V6_vcvt_sf_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3479: def: Pat<(int_hexagon_V6_vcvt_sf_hf_128B HvxVR:$src1),
  3480:          (V6_vcvt_sf_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3481: def: Pat<(int_hexagon_V6_vcvt_ub_hf HvxVR:$src1, HvxVR:$src2),
  3482:          (V6_vcvt_ub_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3483: def: Pat<(int_hexagon_V6_vcvt_ub_hf_128B HvxVR:$src1, HvxVR:$src2),
  3484:          (V6_vcvt_ub_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3485: def: Pat<(int_hexagon_V6_vcvt_uh_hf HvxVR:$src1),
  3486:          (V6_vcvt_uh_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3487: def: Pat<(int_hexagon_V6_vcvt_uh_hf_128B HvxVR:$src1),
  3488:          (V6_vcvt_uh_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3489: def: Pat<(int_hexagon_V6_vdmpy_sf_hf HvxVR:$src1, HvxVR:$src2),
  3490:          (V6_vdmpy_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3491: def: Pat<(int_hexagon_V6_vdmpy_sf_hf_128B HvxVR:$src1, HvxVR:$src2),
  3492:          (V6_vdmpy_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3493: def: Pat<(int_hexagon_V6_vdmpy_sf_hf_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  3494:          (V6_vdmpy_sf_hf_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B]>;
  3495: def: Pat<(int_hexagon_V6_vdmpy_sf_hf_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  3496:          (V6_vdmpy_sf_hf_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B]>;
  3497: def: Pat<(int_hexagon_V6_vfmax_hf HvxVR:$src1, HvxVR:$src2),
  3498:          (V6_vfmax_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3499: def: Pat<(int_hexagon_V6_vfmax_hf_128B HvxVR:$src1, HvxVR:$src2),
  3500:          (V6_vfmax_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 3501-3750 / 第 3501-3750 行

```tablegen
  3501: def: Pat<(int_hexagon_V6_vfmax_sf HvxVR:$src1, HvxVR:$src2),
  3502:          (V6_vfmax_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3503: def: Pat<(int_hexagon_V6_vfmax_sf_128B HvxVR:$src1, HvxVR:$src2),
  3504:          (V6_vfmax_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3505: def: Pat<(int_hexagon_V6_vfmin_hf HvxVR:$src1, HvxVR:$src2),
  3506:          (V6_vfmin_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3507: def: Pat<(int_hexagon_V6_vfmin_hf_128B HvxVR:$src1, HvxVR:$src2),
  3508:          (V6_vfmin_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3509: def: Pat<(int_hexagon_V6_vfmin_sf HvxVR:$src1, HvxVR:$src2),
  3510:          (V6_vfmin_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3511: def: Pat<(int_hexagon_V6_vfmin_sf_128B HvxVR:$src1, HvxVR:$src2),
  3512:          (V6_vfmin_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3513: def: Pat<(int_hexagon_V6_vfneg_hf HvxVR:$src1),
  3514:          (V6_vfneg_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3515: def: Pat<(int_hexagon_V6_vfneg_hf_128B HvxVR:$src1),
  3516:          (V6_vfneg_hf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3517: def: Pat<(int_hexagon_V6_vfneg_sf HvxVR:$src1),
  3518:          (V6_vfneg_sf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX64B]>;
  3519: def: Pat<(int_hexagon_V6_vfneg_sf_128B HvxVR:$src1),
  3520:          (V6_vfneg_sf HvxVR:$src1)>, Requires<[UseHVXV68, UseHVX128B]>;
  3521: def: Pat<(int_hexagon_V6_vgthf HvxVR:$src1, HvxVR:$src2),
  3522:          (V6_vgthf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3523: def: Pat<(int_hexagon_V6_vgthf_128B HvxVR:$src1, HvxVR:$src2),
  3524:          (V6_vgthf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3525: def: Pat<(int_hexagon_V6_vgthf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3526:          (V6_vgthf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3527: def: Pat<(int_hexagon_V6_vgthf_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3528:          (V6_vgthf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3529: def: Pat<(int_hexagon_V6_vgthf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3530:          (V6_vgthf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3531: def: Pat<(int_hexagon_V6_vgthf_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3532:          (V6_vgthf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3533: def: Pat<(int_hexagon_V6_vgthf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3534:          (V6_vgthf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3535: def: Pat<(int_hexagon_V6_vgthf_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3536:          (V6_vgthf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3537: def: Pat<(int_hexagon_V6_vgtsf HvxVR:$src1, HvxVR:$src2),
  3538:          (V6_vgtsf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3539: def: Pat<(int_hexagon_V6_vgtsf_128B HvxVR:$src1, HvxVR:$src2),
  3540:          (V6_vgtsf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3541: def: Pat<(int_hexagon_V6_vgtsf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3542:          (V6_vgtsf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3543: def: Pat<(int_hexagon_V6_vgtsf_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3544:          (V6_vgtsf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3545: def: Pat<(int_hexagon_V6_vgtsf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3546:          (V6_vgtsf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3547: def: Pat<(int_hexagon_V6_vgtsf_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3548:          (V6_vgtsf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3549: def: Pat<(int_hexagon_V6_vgtsf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3550:          (V6_vgtsf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3551: def: Pat<(int_hexagon_V6_vgtsf_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3552:          (V6_vgtsf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3553: def: Pat<(int_hexagon_V6_vmax_hf HvxVR:$src1, HvxVR:$src2),
  3554:          (V6_vmax_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3555: def: Pat<(int_hexagon_V6_vmax_hf_128B HvxVR:$src1, HvxVR:$src2),
  3556:          (V6_vmax_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3557: def: Pat<(int_hexagon_V6_vmax_sf HvxVR:$src1, HvxVR:$src2),
  3558:          (V6_vmax_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3559: def: Pat<(int_hexagon_V6_vmax_sf_128B HvxVR:$src1, HvxVR:$src2),
  3560:          (V6_vmax_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3561: def: Pat<(int_hexagon_V6_vmin_hf HvxVR:$src1, HvxVR:$src2),
  3562:          (V6_vmin_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3563: def: Pat<(int_hexagon_V6_vmin_hf_128B HvxVR:$src1, HvxVR:$src2),
  3564:          (V6_vmin_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3565: def: Pat<(int_hexagon_V6_vmin_sf HvxVR:$src1, HvxVR:$src2),
  3566:          (V6_vmin_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3567: def: Pat<(int_hexagon_V6_vmin_sf_128B HvxVR:$src1, HvxVR:$src2),
  3568:          (V6_vmin_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3569: def: Pat<(int_hexagon_V6_vmpy_hf_hf HvxVR:$src1, HvxVR:$src2),
  3570:          (V6_vmpy_hf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3571: def: Pat<(int_hexagon_V6_vmpy_hf_hf_128B HvxVR:$src1, HvxVR:$src2),
  3572:          (V6_vmpy_hf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3573: def: Pat<(int_hexagon_V6_vmpy_hf_hf_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  3574:          (V6_vmpy_hf_hf_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B]>;
  3575: def: Pat<(int_hexagon_V6_vmpy_hf_hf_acc_128B HvxVR:$src1, HvxVR:$src2, HvxVR:$src3),
  3576:          (V6_vmpy_hf_hf_acc HvxVR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B]>;
  3577: def: Pat<(int_hexagon_V6_vmpy_qf16 HvxVR:$src1, HvxVR:$src2),
  3578:          (V6_vmpy_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3579: def: Pat<(int_hexagon_V6_vmpy_qf16_128B HvxVR:$src1, HvxVR:$src2),
  3580:          (V6_vmpy_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3581: def: Pat<(int_hexagon_V6_vmpy_qf16_hf HvxVR:$src1, HvxVR:$src2),
  3582:          (V6_vmpy_qf16_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3583: def: Pat<(int_hexagon_V6_vmpy_qf16_hf_128B HvxVR:$src1, HvxVR:$src2),
  3584:          (V6_vmpy_qf16_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3585: def: Pat<(int_hexagon_V6_vmpy_qf16_mix_hf HvxVR:$src1, HvxVR:$src2),
  3586:          (V6_vmpy_qf16_mix_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3587: def: Pat<(int_hexagon_V6_vmpy_qf16_mix_hf_128B HvxVR:$src1, HvxVR:$src2),
  3588:          (V6_vmpy_qf16_mix_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3589: def: Pat<(int_hexagon_V6_vmpy_qf32 HvxVR:$src1, HvxVR:$src2),
  3590:          (V6_vmpy_qf32 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3591: def: Pat<(int_hexagon_V6_vmpy_qf32_128B HvxVR:$src1, HvxVR:$src2),
  3592:          (V6_vmpy_qf32 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3593: def: Pat<(int_hexagon_V6_vmpy_qf32_hf HvxVR:$src1, HvxVR:$src2),
  3594:          (V6_vmpy_qf32_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3595: def: Pat<(int_hexagon_V6_vmpy_qf32_hf_128B HvxVR:$src1, HvxVR:$src2),
  3596:          (V6_vmpy_qf32_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3597: def: Pat<(int_hexagon_V6_vmpy_qf32_mix_hf HvxVR:$src1, HvxVR:$src2),
  3598:          (V6_vmpy_qf32_mix_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3599: def: Pat<(int_hexagon_V6_vmpy_qf32_mix_hf_128B HvxVR:$src1, HvxVR:$src2),
  3600:          (V6_vmpy_qf32_mix_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3601: def: Pat<(int_hexagon_V6_vmpy_qf32_qf16 HvxVR:$src1, HvxVR:$src2),
  3602:          (V6_vmpy_qf32_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3603: def: Pat<(int_hexagon_V6_vmpy_qf32_qf16_128B HvxVR:$src1, HvxVR:$src2),
  3604:          (V6_vmpy_qf32_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3605: def: Pat<(int_hexagon_V6_vmpy_qf32_sf HvxVR:$src1, HvxVR:$src2),
  3606:          (V6_vmpy_qf32_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3607: def: Pat<(int_hexagon_V6_vmpy_qf32_sf_128B HvxVR:$src1, HvxVR:$src2),
  3608:          (V6_vmpy_qf32_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3609: def: Pat<(int_hexagon_V6_vmpy_sf_hf HvxVR:$src1, HvxVR:$src2),
  3610:          (V6_vmpy_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3611: def: Pat<(int_hexagon_V6_vmpy_sf_hf_128B HvxVR:$src1, HvxVR:$src2),
  3612:          (V6_vmpy_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3613: def: Pat<(int_hexagon_V6_vmpy_sf_hf_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3614:          (V6_vmpy_sf_hf_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX64B]>;
  3615: def: Pat<(int_hexagon_V6_vmpy_sf_hf_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3616:          (V6_vmpy_sf_hf_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV68, UseHVX128B]>;
  3617: def: Pat<(int_hexagon_V6_vmpy_sf_sf HvxVR:$src1, HvxVR:$src2),
  3618:          (V6_vmpy_sf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3619: def: Pat<(int_hexagon_V6_vmpy_sf_sf_128B HvxVR:$src1, HvxVR:$src2),
  3620:          (V6_vmpy_sf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3621: def: Pat<(int_hexagon_V6_vsub_hf HvxVR:$src1, HvxVR:$src2),
  3622:          (V6_vsub_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3623: def: Pat<(int_hexagon_V6_vsub_hf_128B HvxVR:$src1, HvxVR:$src2),
  3624:          (V6_vsub_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3625: def: Pat<(int_hexagon_V6_vsub_hf_hf HvxVR:$src1, HvxVR:$src2),
  3626:          (V6_vsub_hf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3627: def: Pat<(int_hexagon_V6_vsub_hf_hf_128B HvxVR:$src1, HvxVR:$src2),
  3628:          (V6_vsub_hf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3629: def: Pat<(int_hexagon_V6_vsub_qf16 HvxVR:$src1, HvxVR:$src2),
  3630:          (V6_vsub_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3631: def: Pat<(int_hexagon_V6_vsub_qf16_128B HvxVR:$src1, HvxVR:$src2),
  3632:          (V6_vsub_qf16 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3633: def: Pat<(int_hexagon_V6_vsub_qf16_mix HvxVR:$src1, HvxVR:$src2),
  3634:          (V6_vsub_qf16_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3635: def: Pat<(int_hexagon_V6_vsub_qf16_mix_128B HvxVR:$src1, HvxVR:$src2),
  3636:          (V6_vsub_qf16_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3637: def: Pat<(int_hexagon_V6_vsub_qf32 HvxVR:$src1, HvxVR:$src2),
  3638:          (V6_vsub_qf32 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3639: def: Pat<(int_hexagon_V6_vsub_qf32_128B HvxVR:$src1, HvxVR:$src2),
  3640:          (V6_vsub_qf32 HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3641: def: Pat<(int_hexagon_V6_vsub_qf32_mix HvxVR:$src1, HvxVR:$src2),
  3642:          (V6_vsub_qf32_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3643: def: Pat<(int_hexagon_V6_vsub_qf32_mix_128B HvxVR:$src1, HvxVR:$src2),
  3644:          (V6_vsub_qf32_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3645: def: Pat<(int_hexagon_V6_vsub_sf HvxVR:$src1, HvxVR:$src2),
  3646:          (V6_vsub_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B, UseHVXQFloat]>;
  3647: def: Pat<(int_hexagon_V6_vsub_sf_128B HvxVR:$src1, HvxVR:$src2),
  3648:          (V6_vsub_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B, UseHVXQFloat]>;
  3649: def: Pat<(int_hexagon_V6_vsub_sf_hf HvxVR:$src1, HvxVR:$src2),
  3650:          (V6_vsub_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3651: def: Pat<(int_hexagon_V6_vsub_sf_hf_128B HvxVR:$src1, HvxVR:$src2),
  3652:          (V6_vsub_sf_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3653: def: Pat<(int_hexagon_V6_vsub_sf_sf HvxVR:$src1, HvxVR:$src2),
  3654:          (V6_vsub_sf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX64B]>;
  3655: def: Pat<(int_hexagon_V6_vsub_sf_sf_128B HvxVR:$src1, HvxVR:$src2),
  3656:          (V6_vsub_sf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV68, UseHVX128B]>;
  3657: 
  3658: // V69 HVX Instructions.
  3659: 
  3660: def: Pat<(int_hexagon_V6_vasrvuhubrndsat HvxWR:$src1, HvxVR:$src2),
  3661:          (V6_vasrvuhubrndsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX64B]>;
  3662: def: Pat<(int_hexagon_V6_vasrvuhubrndsat_128B HvxWR:$src1, HvxVR:$src2),
  3663:          (V6_vasrvuhubrndsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX128B]>;
  3664: def: Pat<(int_hexagon_V6_vasrvuhubsat HvxWR:$src1, HvxVR:$src2),
  3665:          (V6_vasrvuhubsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX64B]>;
  3666: def: Pat<(int_hexagon_V6_vasrvuhubsat_128B HvxWR:$src1, HvxVR:$src2),
  3667:          (V6_vasrvuhubsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX128B]>;
  3668: def: Pat<(int_hexagon_V6_vasrvwuhrndsat HvxWR:$src1, HvxVR:$src2),
  3669:          (V6_vasrvwuhrndsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX64B]>;
  3670: def: Pat<(int_hexagon_V6_vasrvwuhrndsat_128B HvxWR:$src1, HvxVR:$src2),
  3671:          (V6_vasrvwuhrndsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX128B]>;
  3672: def: Pat<(int_hexagon_V6_vasrvwuhsat HvxWR:$src1, HvxVR:$src2),
  3673:          (V6_vasrvwuhsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX64B]>;
  3674: def: Pat<(int_hexagon_V6_vasrvwuhsat_128B HvxWR:$src1, HvxVR:$src2),
  3675:          (V6_vasrvwuhsat HvxWR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX128B]>;
  3676: def: Pat<(int_hexagon_V6_vmpyuhvs HvxVR:$src1, HvxVR:$src2),
  3677:          (V6_vmpyuhvs HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX64B]>;
  3678: def: Pat<(int_hexagon_V6_vmpyuhvs_128B HvxVR:$src1, HvxVR:$src2),
  3679:          (V6_vmpyuhvs HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV69, UseHVX128B]>;
  3680: 
  3681: // V73 HVX Instructions.
  3682: 
  3683: def: Pat<(int_hexagon_V6_vadd_sf_bf HvxVR:$src1, HvxVR:$src2),
  3684:          (V6_vadd_sf_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX64B]>;
  3685: def: Pat<(int_hexagon_V6_vadd_sf_bf_128B HvxVR:$src1, HvxVR:$src2),
  3686:          (V6_vadd_sf_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX128B]>;
  3687: def: Pat<(int_hexagon_V6_vconv_h_hf HvxVR:$src1),
  3688:          (V6_vconv_h_hf HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX64B]>;
  3689: def: Pat<(int_hexagon_V6_vconv_h_hf_128B HvxVR:$src1),
  3690:          (V6_vconv_h_hf HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX128B]>;
  3691: def: Pat<(int_hexagon_V6_vconv_hf_h HvxVR:$src1),
  3692:          (V6_vconv_hf_h HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX64B]>;
  3693: def: Pat<(int_hexagon_V6_vconv_hf_h_128B HvxVR:$src1),
  3694:          (V6_vconv_hf_h HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX128B]>;
  3695: def: Pat<(int_hexagon_V6_vconv_sf_w HvxVR:$src1),
  3696:          (V6_vconv_sf_w HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX64B]>;
  3697: def: Pat<(int_hexagon_V6_vconv_sf_w_128B HvxVR:$src1),
  3698:          (V6_vconv_sf_w HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX128B]>;
  3699: def: Pat<(int_hexagon_V6_vconv_w_sf HvxVR:$src1),
  3700:          (V6_vconv_w_sf HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX64B]>;
  3701: def: Pat<(int_hexagon_V6_vconv_w_sf_128B HvxVR:$src1),
  3702:          (V6_vconv_w_sf HvxVR:$src1)>, Requires<[UseHVXV73, UseHVX128B]>;
  3703: def: Pat<(int_hexagon_V6_vcvt_bf_sf HvxVR:$src1, HvxVR:$src2),
  3704:          (V6_vcvt_bf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX64B]>;
  3705: def: Pat<(int_hexagon_V6_vcvt_bf_sf_128B HvxVR:$src1, HvxVR:$src2),
  3706:          (V6_vcvt_bf_sf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX128B]>;
  3707: def: Pat<(int_hexagon_V6_vgtbf HvxVR:$src1, HvxVR:$src2),
  3708:          (V6_vgtbf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX64B, UseHVXQFloat]>;
  3709: def: Pat<(int_hexagon_V6_vgtbf_128B HvxVR:$src1, HvxVR:$src2),
  3710:          (V6_vgtbf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX128B, UseHVXQFloat]>;
  3711: def: Pat<(int_hexagon_V6_vgtbf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3712:          (V6_vgtbf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX64B, UseHVXQFloat]>;
  3713: def: Pat<(int_hexagon_V6_vgtbf_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3714:          (V6_vgtbf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX128B, UseHVXQFloat]>;
  3715: def: Pat<(int_hexagon_V6_vgtbf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3716:          (V6_vgtbf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX64B, UseHVXQFloat]>;
  3717: def: Pat<(int_hexagon_V6_vgtbf_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3718:          (V6_vgtbf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX128B, UseHVXQFloat]>;
  3719: def: Pat<(int_hexagon_V6_vgtbf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3720:          (V6_vgtbf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX64B, UseHVXQFloat]>;
  3721: def: Pat<(int_hexagon_V6_vgtbf_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3722:          (V6_vgtbf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX128B, UseHVXQFloat]>;
  3723: def: Pat<(int_hexagon_V6_vmax_bf HvxVR:$src1, HvxVR:$src2),
  3724:          (V6_vmax_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX64B]>;
  3725: def: Pat<(int_hexagon_V6_vmax_bf_128B HvxVR:$src1, HvxVR:$src2),
  3726:          (V6_vmax_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX128B]>;
  3727: def: Pat<(int_hexagon_V6_vmin_bf HvxVR:$src1, HvxVR:$src2),
  3728:          (V6_vmin_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX64B]>;
  3729: def: Pat<(int_hexagon_V6_vmin_bf_128B HvxVR:$src1, HvxVR:$src2),
  3730:          (V6_vmin_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX128B]>;
  3731: def: Pat<(int_hexagon_V6_vmpy_sf_bf HvxVR:$src1, HvxVR:$src2),
  3732:          (V6_vmpy_sf_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX64B]>;
  3733: def: Pat<(int_hexagon_V6_vmpy_sf_bf_128B HvxVR:$src1, HvxVR:$src2),
  3734:          (V6_vmpy_sf_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX128B]>;
  3735: def: Pat<(int_hexagon_V6_vmpy_sf_bf_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3736:          (V6_vmpy_sf_bf_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX64B]>;
  3737: def: Pat<(int_hexagon_V6_vmpy_sf_bf_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3738:          (V6_vmpy_sf_bf_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV73, UseHVX128B]>;
  3739: def: Pat<(int_hexagon_V6_vsub_sf_bf HvxVR:$src1, HvxVR:$src2),
  3740:          (V6_vsub_sf_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX64B]>;
  3741: def: Pat<(int_hexagon_V6_vsub_sf_bf_128B HvxVR:$src1, HvxVR:$src2),
  3742:          (V6_vsub_sf_bf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV73, UseHVX128B]>;
  3743: 
  3744: // V79 HVX Instructions.
  3745: 
  3746: def: Pat<(int_hexagon_V6_get_qfext HvxVR:$src1, IntRegs:$src2),
  3747:          (V6_get_qfext HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX64B, UseHVXQFloat]>;
  3748: def: Pat<(int_hexagon_V6_get_qfext_128B HvxVR:$src1, IntRegs:$src2),
  3749:          (V6_get_qfext HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX128B, UseHVXQFloat]>;
  3750: def: Pat<(int_hexagon_V6_get_qfext_oracc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

### Lines 3751-3956 / 第 3751-3956 行

```tablegen
  3751:          (V6_get_qfext_oracc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV79, UseHVX64B, UseHVXQFloat]>;
  3752: def: Pat<(int_hexagon_V6_get_qfext_oracc_128B HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
  3753:          (V6_get_qfext_oracc HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>, Requires<[UseHVXV79, UseHVX128B, UseHVXQFloat]>;
  3754: def: Pat<(int_hexagon_V6_set_qfext HvxVR:$src1, IntRegs:$src2),
  3755:          (V6_set_qfext HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX64B, UseHVXQFloat]>;
  3756: def: Pat<(int_hexagon_V6_set_qfext_128B HvxVR:$src1, IntRegs:$src2),
  3757:          (V6_set_qfext HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX128B, UseHVXQFloat]>;
  3758: def: Pat<(int_hexagon_V6_vabs_f8 HvxVR:$src1),
  3759:          (V6_vabs_f8 HvxVR:$src1)>, Requires<[HasV79, UseHVX64B]>;
  3760: def: Pat<(int_hexagon_V6_vabs_f8_128B HvxVR:$src1),
  3761:          (V6_vabs_f8 HvxVR:$src1)>, Requires<[HasV79, UseHVX128B]>;
  3762: def: Pat<(int_hexagon_V6_vadd_hf_f8 HvxVR:$src1, HvxVR:$src2),
  3763:          (V6_vadd_hf_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX64B]>;
  3764: def: Pat<(int_hexagon_V6_vadd_hf_f8_128B HvxVR:$src1, HvxVR:$src2),
  3765:          (V6_vadd_hf_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX128B]>;
  3766: def: Pat<(int_hexagon_V6_vcvt2_b_hf HvxVR:$src1, HvxVR:$src2),
  3767:          (V6_vcvt2_b_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV79, UseHVX64B]>;
  3768: def: Pat<(int_hexagon_V6_vcvt2_b_hf_128B HvxVR:$src1, HvxVR:$src2),
  3769:          (V6_vcvt2_b_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV79, UseHVX128B]>;
  3770: def: Pat<(int_hexagon_V6_vcvt2_hf_b HvxVR:$src1),
  3771:          (V6_vcvt2_hf_b HvxVR:$src1)>, Requires<[UseHVXV79, UseHVX64B]>;
  3772: def: Pat<(int_hexagon_V6_vcvt2_hf_b_128B HvxVR:$src1),
  3773:          (V6_vcvt2_hf_b HvxVR:$src1)>, Requires<[UseHVXV79, UseHVX128B]>;
  3774: def: Pat<(int_hexagon_V6_vcvt2_hf_ub HvxVR:$src1),
  3775:          (V6_vcvt2_hf_ub HvxVR:$src1)>, Requires<[UseHVXV79, UseHVX64B]>;
  3776: def: Pat<(int_hexagon_V6_vcvt2_hf_ub_128B HvxVR:$src1),
  3777:          (V6_vcvt2_hf_ub HvxVR:$src1)>, Requires<[UseHVXV79, UseHVX128B]>;
  3778: def: Pat<(int_hexagon_V6_vcvt2_ub_hf HvxVR:$src1, HvxVR:$src2),
  3779:          (V6_vcvt2_ub_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV79, UseHVX64B]>;
  3780: def: Pat<(int_hexagon_V6_vcvt2_ub_hf_128B HvxVR:$src1, HvxVR:$src2),
  3781:          (V6_vcvt2_ub_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV79, UseHVX128B]>;
  3782: def: Pat<(int_hexagon_V6_vcvt_f8_hf HvxVR:$src1, HvxVR:$src2),
  3783:          (V6_vcvt_f8_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX64B]>;
  3784: def: Pat<(int_hexagon_V6_vcvt_f8_hf_128B HvxVR:$src1, HvxVR:$src2),
  3785:          (V6_vcvt_f8_hf HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX128B]>;
  3786: def: Pat<(int_hexagon_V6_vcvt_hf_f8 HvxVR:$src1),
  3787:          (V6_vcvt_hf_f8 HvxVR:$src1)>, Requires<[HasV79, UseHVX64B]>;
  3788: def: Pat<(int_hexagon_V6_vcvt_hf_f8_128B HvxVR:$src1),
  3789:          (V6_vcvt_hf_f8 HvxVR:$src1)>, Requires<[HasV79, UseHVX128B]>;
  3790: def: Pat<(int_hexagon_V6_vfmax_f8 HvxVR:$src1, HvxVR:$src2),
  3791:          (V6_vfmax_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX64B]>;
  3792: def: Pat<(int_hexagon_V6_vfmax_f8_128B HvxVR:$src1, HvxVR:$src2),
  3793:          (V6_vfmax_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX128B]>;
  3794: def: Pat<(int_hexagon_V6_vfmin_f8 HvxVR:$src1, HvxVR:$src2),
  3795:          (V6_vfmin_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX64B]>;
  3796: def: Pat<(int_hexagon_V6_vfmin_f8_128B HvxVR:$src1, HvxVR:$src2),
  3797:          (V6_vfmin_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX128B]>;
  3798: def: Pat<(int_hexagon_V6_vfneg_f8 HvxVR:$src1),
  3799:          (V6_vfneg_f8 HvxVR:$src1)>, Requires<[HasV79, UseHVX64B]>;
  3800: def: Pat<(int_hexagon_V6_vfneg_f8_128B HvxVR:$src1),
  3801:          (V6_vfneg_f8 HvxVR:$src1)>, Requires<[HasV79, UseHVX128B]>;
  3802: def: Pat<(int_hexagon_V6_vmerge_qf HvxVR:$src1, HvxVR:$src2),
  3803:          (V6_vmerge_qf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV79, UseHVX64B, UseHVXQFloat]>;
  3804: def: Pat<(int_hexagon_V6_vmerge_qf_128B HvxVR:$src1, HvxVR:$src2),
  3805:          (V6_vmerge_qf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV79, UseHVX128B, UseHVXQFloat]>;
  3806: def: Pat<(int_hexagon_V6_vmpy_hf_f8 HvxVR:$src1, HvxVR:$src2),
  3807:          (V6_vmpy_hf_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX64B]>;
  3808: def: Pat<(int_hexagon_V6_vmpy_hf_f8_128B HvxVR:$src1, HvxVR:$src2),
  3809:          (V6_vmpy_hf_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX128B]>;
  3810: def: Pat<(int_hexagon_V6_vmpy_hf_f8_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3811:          (V6_vmpy_hf_f8_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[HasV79, UseHVX64B]>;
  3812: def: Pat<(int_hexagon_V6_vmpy_hf_f8_acc_128B HvxWR:$src1, HvxVR:$src2, HvxVR:$src3),
  3813:          (V6_vmpy_hf_f8_acc HvxWR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[HasV79, UseHVX128B]>;
  3814: def: Pat<(int_hexagon_V6_vmpy_rt_hf HvxVR:$src1, IntRegs:$src2),
  3815:          (V6_vmpy_rt_hf HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX64B, UseHVXQFloat]>;
  3816: def: Pat<(int_hexagon_V6_vmpy_rt_hf_128B HvxVR:$src1, IntRegs:$src2),
  3817:          (V6_vmpy_rt_hf HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX128B, UseHVXQFloat]>;
  3818: def: Pat<(int_hexagon_V6_vmpy_rt_qf16 HvxVR:$src1, IntRegs:$src2),
  3819:          (V6_vmpy_rt_qf16 HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX64B, UseHVXQFloat]>;
  3820: def: Pat<(int_hexagon_V6_vmpy_rt_qf16_128B HvxVR:$src1, IntRegs:$src2),
  3821:          (V6_vmpy_rt_qf16 HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX128B, UseHVXQFloat]>;
  3822: def: Pat<(int_hexagon_V6_vmpy_rt_sf HvxVR:$src1, IntRegs:$src2),
  3823:          (V6_vmpy_rt_sf HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX64B, UseHVXQFloat]>;
  3824: def: Pat<(int_hexagon_V6_vmpy_rt_sf_128B HvxVR:$src1, IntRegs:$src2),
  3825:          (V6_vmpy_rt_sf HvxVR:$src1, IntRegs:$src2)>, Requires<[UseHVXV79, UseHVX128B, UseHVXQFloat]>;
  3826: def: Pat<(int_hexagon_V6_vsub_hf_f8 HvxVR:$src1, HvxVR:$src2),
  3827:          (V6_vsub_hf_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX64B]>;
  3828: def: Pat<(int_hexagon_V6_vsub_hf_f8_128B HvxVR:$src1, HvxVR:$src2),
  3829:          (V6_vsub_hf_f8 HvxVR:$src1, HvxVR:$src2)>, Requires<[HasV79, UseHVX128B]>;
  3830: 
  3831: // V81 HVX Instructions.
  3832: 
  3833: def: Pat<(int_hexagon_V6_vabs_qf16_hf HvxVR:$src1),
  3834:          (V6_vabs_qf16_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3835: def: Pat<(int_hexagon_V6_vabs_qf16_hf_128B HvxVR:$src1),
  3836:          (V6_vabs_qf16_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3837: def: Pat<(int_hexagon_V6_vabs_qf16_qf16 HvxVR:$src1),
  3838:          (V6_vabs_qf16_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3839: def: Pat<(int_hexagon_V6_vabs_qf16_qf16_128B HvxVR:$src1),
  3840:          (V6_vabs_qf16_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3841: def: Pat<(int_hexagon_V6_vabs_qf32_qf32 HvxVR:$src1),
  3842:          (V6_vabs_qf32_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3843: def: Pat<(int_hexagon_V6_vabs_qf32_qf32_128B HvxVR:$src1),
  3844:          (V6_vabs_qf32_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3845: def: Pat<(int_hexagon_V6_vabs_qf32_sf HvxVR:$src1),
  3846:          (V6_vabs_qf32_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3847: def: Pat<(int_hexagon_V6_vabs_qf32_sf_128B HvxVR:$src1),
  3848:          (V6_vabs_qf32_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3849: def: Pat<(int_hexagon_V6_valign4 HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3850:          (V6_valign4 HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV81, UseHVX64B]>;
  3851: def: Pat<(int_hexagon_V6_valign4_128B HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3),
  3852:          (V6_valign4 HvxVR:$src1, HvxVR:$src2, IntRegsLow8:$src3)>, Requires<[UseHVXV81, UseHVX128B]>;
  3853: def: Pat<(int_hexagon_V6_vconv_bf_qf32 HvxWR:$src1),
  3854:          (V6_vconv_bf_qf32 HvxWR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3855: def: Pat<(int_hexagon_V6_vconv_bf_qf32_128B HvxWR:$src1),
  3856:          (V6_vconv_bf_qf32 HvxWR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3857: def: Pat<(int_hexagon_V6_vconv_f8_qf16 HvxVR:$src1),
  3858:          (V6_vconv_f8_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3859: def: Pat<(int_hexagon_V6_vconv_f8_qf16_128B HvxVR:$src1),
  3860:          (V6_vconv_f8_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3861: def: Pat<(int_hexagon_V6_vconv_h_hf_rnd HvxVR:$src1),
  3862:          (V6_vconv_h_hf_rnd HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B]>;
  3863: def: Pat<(int_hexagon_V6_vconv_h_hf_rnd_128B HvxVR:$src1),
  3864:          (V6_vconv_h_hf_rnd HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B]>;
  3865: def: Pat<(int_hexagon_V6_vconv_qf16_f8 HvxVR:$src1),
  3866:          (V6_vconv_qf16_f8 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3867: def: Pat<(int_hexagon_V6_vconv_qf16_f8_128B HvxVR:$src1),
  3868:          (V6_vconv_qf16_f8 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3869: def: Pat<(int_hexagon_V6_vconv_qf16_hf HvxVR:$src1),
  3870:          (V6_vconv_qf16_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3871: def: Pat<(int_hexagon_V6_vconv_qf16_hf_128B HvxVR:$src1),
  3872:          (V6_vconv_qf16_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3873: def: Pat<(int_hexagon_V6_vconv_qf16_qf16 HvxVR:$src1),
  3874:          (V6_vconv_qf16_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3875: def: Pat<(int_hexagon_V6_vconv_qf16_qf16_128B HvxVR:$src1),
  3876:          (V6_vconv_qf16_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3877: def: Pat<(int_hexagon_V6_vconv_qf32_qf32 HvxVR:$src1),
  3878:          (V6_vconv_qf32_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3879: def: Pat<(int_hexagon_V6_vconv_qf32_qf32_128B HvxVR:$src1),
  3880:          (V6_vconv_qf32_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3881: def: Pat<(int_hexagon_V6_vconv_qf32_sf HvxVR:$src1),
  3882:          (V6_vconv_qf32_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3883: def: Pat<(int_hexagon_V6_vconv_qf32_sf_128B HvxVR:$src1),
  3884:          (V6_vconv_qf32_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3885: def: Pat<(int_hexagon_V6_veqhf HvxVR:$src1, HvxVR:$src2),
  3886:          (V6_veqhf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3887: def: Pat<(int_hexagon_V6_veqhf_128B HvxVR:$src1, HvxVR:$src2),
  3888:          (V6_veqhf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3889: def: Pat<(int_hexagon_V6_veqhf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3890:          (V6_veqhf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3891: def: Pat<(int_hexagon_V6_veqhf_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3892:          (V6_veqhf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3893: def: Pat<(int_hexagon_V6_veqhf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3894:          (V6_veqhf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3895: def: Pat<(int_hexagon_V6_veqhf_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3896:          (V6_veqhf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3897: def: Pat<(int_hexagon_V6_veqhf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3898:          (V6_veqhf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3899: def: Pat<(int_hexagon_V6_veqhf_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3900:          (V6_veqhf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3901: def: Pat<(int_hexagon_V6_veqsf HvxVR:$src1, HvxVR:$src2),
  3902:          (V6_veqsf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3903: def: Pat<(int_hexagon_V6_veqsf_128B HvxVR:$src1, HvxVR:$src2),
  3904:          (V6_veqsf HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3905: def: Pat<(int_hexagon_V6_veqsf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3906:          (V6_veqsf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3907: def: Pat<(int_hexagon_V6_veqsf_and_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3908:          (V6_veqsf_and HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3909: def: Pat<(int_hexagon_V6_veqsf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3910:          (V6_veqsf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3911: def: Pat<(int_hexagon_V6_veqsf_or_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3912:          (V6_veqsf_or HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3913: def: Pat<(int_hexagon_V6_veqsf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3914:          (V6_veqsf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3915: def: Pat<(int_hexagon_V6_veqsf_xor_128B HvxQR:$src1, HvxVR:$src2, HvxVR:$src3),
  3916:          (V6_veqsf_xor HvxQR:$src1, HvxVR:$src2, HvxVR:$src3)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3917: def: Pat<(int_hexagon_V6_vilog2_hf HvxVR:$src1),
  3918:          (V6_vilog2_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3919: def: Pat<(int_hexagon_V6_vilog2_hf_128B HvxVR:$src1),
  3920:          (V6_vilog2_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3921: def: Pat<(int_hexagon_V6_vilog2_qf16 HvxVR:$src1),
  3922:          (V6_vilog2_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3923: def: Pat<(int_hexagon_V6_vilog2_qf16_128B HvxVR:$src1),
  3924:          (V6_vilog2_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3925: def: Pat<(int_hexagon_V6_vilog2_qf32 HvxVR:$src1),
  3926:          (V6_vilog2_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3927: def: Pat<(int_hexagon_V6_vilog2_qf32_128B HvxVR:$src1),
  3928:          (V6_vilog2_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3929: def: Pat<(int_hexagon_V6_vilog2_sf HvxVR:$src1),
  3930:          (V6_vilog2_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3931: def: Pat<(int_hexagon_V6_vilog2_sf_128B HvxVR:$src1),
  3932:          (V6_vilog2_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3933: def: Pat<(int_hexagon_V6_vneg_qf16_hf HvxVR:$src1),
  3934:          (V6_vneg_qf16_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3935: def: Pat<(int_hexagon_V6_vneg_qf16_hf_128B HvxVR:$src1),
  3936:          (V6_vneg_qf16_hf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3937: def: Pat<(int_hexagon_V6_vneg_qf16_qf16 HvxVR:$src1),
  3938:          (V6_vneg_qf16_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3939: def: Pat<(int_hexagon_V6_vneg_qf16_qf16_128B HvxVR:$src1),
  3940:          (V6_vneg_qf16_qf16 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3941: def: Pat<(int_hexagon_V6_vneg_qf32_qf32 HvxVR:$src1),
  3942:          (V6_vneg_qf32_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3943: def: Pat<(int_hexagon_V6_vneg_qf32_qf32_128B HvxVR:$src1),
  3944:          (V6_vneg_qf32_qf32 HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3945: def: Pat<(int_hexagon_V6_vneg_qf32_sf HvxVR:$src1),
  3946:          (V6_vneg_qf32_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3947: def: Pat<(int_hexagon_V6_vneg_qf32_sf_128B HvxVR:$src1),
  3948:          (V6_vneg_qf32_sf HvxVR:$src1)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3949: def: Pat<(int_hexagon_V6_vsub_hf_mix HvxVR:$src1, HvxVR:$src2),
  3950:          (V6_vsub_hf_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3951: def: Pat<(int_hexagon_V6_vsub_hf_mix_128B HvxVR:$src1, HvxVR:$src2),
  3952:          (V6_vsub_hf_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
  3953: def: Pat<(int_hexagon_V6_vsub_sf_mix HvxVR:$src1, HvxVR:$src2),
  3954:          (V6_vsub_sf_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX64B, UseHVXQFloat]>;
  3955: def: Pat<(int_hexagon_V6_vsub_sf_mix_128B HvxVR:$src1, HvxVR:$src2),
  3956:          (V6_vsub_sf_mix HvxVR:$src1, HvxVR:$src2)>, Requires<[UseHVXV81, UseHVX128B, UseHVXQFloat]>;
```
- EN: Because the file is auto-generated, this range mainly enumerates metadata entries rather than bespoke algorithms.
- CN: 由于文件是自动生成的，这一段主要枚举元数据表项，而不是手写算法。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录
- generated target metadata / 生成的目标元数据

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
