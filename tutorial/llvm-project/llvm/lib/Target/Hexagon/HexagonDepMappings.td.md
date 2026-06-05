# HexagonDepMappings.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonDepMappings.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon generated mapping tables used by the backend using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。 内容以生成表项为主。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。
- Generation status / 生成状态: The file appears to be auto-generated or largely generated from upstream target data. / 该文件看起来是自动生成的，或主要由上游目标数据生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

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
    11: def A2_negAlias : InstAlias<"$Rd32 = neg($Rs32)", (A2_subri IntRegs:$Rd32, 0, IntRegs:$Rs32)>;
    12: def A2_notAlias : InstAlias<"$Rd32 = not($Rs32)", (A2_subri IntRegs:$Rd32, -1, IntRegs:$Rs32)>;
    13: def A2_tfrfAlias : InstAlias<"if (!$Pu4) $Rd32 = $Rs32", (A2_paddif IntRegs:$Rd32, PredRegs:$Pu4, IntRegs:$Rs32, 0)>;
    14: def A2_tfrfnewAlias : InstAlias<"if (!$Pu4.new) $Rd32 = $Rs32", (A2_paddifnew IntRegs:$Rd32, PredRegs:$Pu4, IntRegs:$Rs32, 0)>;
    15: def A2_tfrtAlias : InstAlias<"if ($Pu4) $Rd32 = $Rs32", (A2_paddit IntRegs:$Rd32, PredRegs:$Pu4, IntRegs:$Rs32, 0)>;
    16: def A2_tfrtnewAlias : InstAlias<"if ($Pu4.new) $Rd32 = $Rs32", (A2_padditnew IntRegs:$Rd32, PredRegs:$Pu4, IntRegs:$Rs32, 0)>;
    17: def A2_vaddb_mapAlias : InstAlias<"$Rdd32 = vaddb($Rss32,$Rtt32)", (A2_vaddub DoubleRegs:$Rdd32, DoubleRegs:$Rss32, DoubleRegs:$Rtt32)>;
    18: def A2_vsubb_mapAlias : InstAlias<"$Rdd32 = vsubb($Rss32,$Rtt32)", (A2_vsubub DoubleRegs:$Rdd32, DoubleRegs:$Rss32, DoubleRegs:$Rtt32)>;
    19: def A2_zxtbAlias : InstAlias<"$Rd32 = zxtb($Rs32)", (A2_andir IntRegs:$Rd32, IntRegs:$Rs32, 255)>;
    20: def C2_cmpltAlias : InstAlias<"$Pd4 = cmp.lt($Rs32,$Rt32)", (C2_cmpgt PredRegs:$Pd4, IntRegs:$Rt32, IntRegs:$Rs32)>;
    21: def C2_cmpltuAlias : InstAlias<"$Pd4 = cmp.ltu($Rs32,$Rt32)", (C2_cmpgtu PredRegs:$Pd4, IntRegs:$Rt32, IntRegs:$Rs32)>;
    22: def C2_pxfer_mapAlias : InstAlias<"$Pd4 = $Ps4", (C2_or PredRegs:$Pd4, PredRegs:$Ps4, PredRegs:$Ps4)>;
    23: def J2_jumpf_nopred_mapAlias : InstAlias<"if (!$Pu4) jump $Ii", (J2_jumpf PredRegs:$Pu4, b30_2Imm:$Ii)>;
    24: def J2_jumprf_nopred_mapAlias : InstAlias<"if (!$Pu4) jumpr $Rs32", (J2_jumprf PredRegs:$Pu4, IntRegs:$Rs32)>;
    25: def J2_jumprt_nopred_mapAlias : InstAlias<"if ($Pu4) jumpr $Rs32", (J2_jumprt PredRegs:$Pu4, IntRegs:$Rs32)>;
    26: def J2_jumpt_nopred_mapAlias : InstAlias<"if ($Pu4) jump $Ii", (J2_jumpt PredRegs:$Pu4, b30_2Imm:$Ii)>;
    27: def J2_trap1_noregmapAlias : InstAlias<"trap1(#$Ii)", (J2_trap1 R0, u8_0Imm:$Ii)>;
    28: def L2_loadalignb_zomapAlias : InstAlias<"$Ryy32 = memb_fifo($Rs32)", (L2_loadalignb_io DoubleRegs:$Ryy32, IntRegs:$Rs32, 0)>;
    29: def L2_loadalignh_zomapAlias : InstAlias<"$Ryy32 = memh_fifo($Rs32)", (L2_loadalignh_io DoubleRegs:$Ryy32, IntRegs:$Rs32, 0)>;
    30: def L2_loadbsw2_zomapAlias : InstAlias<"$Rd32 = membh($Rs32)", (L2_loadbsw2_io IntRegs:$Rd32, IntRegs:$Rs32, 0)>;
    31: def L2_loadbsw4_zomapAlias : InstAlias<"$Rdd32 = membh($Rs32)", (L2_loadbsw4_io DoubleRegs:$Rdd32, IntRegs:$Rs32, 0)>;
    32: def L2_loadbzw2_zomapAlias : InstAlias<"$Rd32 = memubh($Rs32)", (L2_loadbzw2_io IntRegs:$Rd32, IntRegs:$Rs32, 0)>;
    33: def L2_loadbzw4_zomapAlias : InstAlias<"$Rdd32 = memubh($Rs32)", (L2_loadbzw4_io DoubleRegs:$Rdd32, IntRegs:$Rs32, 0)>;
    34: def L2_loadrb_zomapAlias : InstAlias<"$Rd32 = memb($Rs32)", (L2_loadrb_io IntRegs:$Rd32, IntRegs:$Rs32, 0)>;
    35: def L2_loadrd_zomapAlias : InstAlias<"$Rdd32 = memd($Rs32)", (L2_loadrd_io DoubleRegs:$Rdd32, IntRegs:$Rs32, 0)>;
    36: def L2_loadrh_zomapAlias : InstAlias<"$Rd32 = memh($Rs32)", (L2_loadrh_io IntRegs:$Rd32, IntRegs:$Rs32, 0)>;
    37: def L2_loadri_zomapAlias : InstAlias<"$Rd32 = memw($Rs32)", (L2_loadri_io IntRegs:$Rd32, IntRegs:$Rs32, 0)>;
    38: def L2_loadrub_zomapAlias : InstAlias<"$Rd32 = memub($Rs32)", (L2_loadrub_io IntRegs:$Rd32, IntRegs:$Rs32, 0)>;
    39: def L2_loadruh_zomapAlias : InstAlias<"$Rd32 = memuh($Rs32)", (L2_loadruh_io IntRegs:$Rd32, IntRegs:$Rs32, 0)>;
    40: def L2_ploadrbf_zomapAlias : InstAlias<"if (!$Pt4) $Rd32 = memb($Rs32)", (L2_ploadrbf_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    41: def L2_ploadrbfnew_zomapAlias : InstAlias<"if (!$Pt4.new) $Rd32 = memb($Rs32)", (L2_ploadrbfnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    42: def L2_ploadrbt_zomapAlias : InstAlias<"if ($Pt4) $Rd32 = memb($Rs32)", (L2_ploadrbt_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    43: def L2_ploadrbtnew_zomapAlias : InstAlias<"if ($Pt4.new) $Rd32 = memb($Rs32)", (L2_ploadrbtnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    44: def L2_ploadrdf_zomapAlias : InstAlias<"if (!$Pt4) $Rdd32 = memd($Rs32)", (L2_ploadrdf_io DoubleRegs:$Rdd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    45: def L2_ploadrdfnew_zomapAlias : InstAlias<"if (!$Pt4.new) $Rdd32 = memd($Rs32)", (L2_ploadrdfnew_io DoubleRegs:$Rdd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    46: def L2_ploadrdt_zomapAlias : InstAlias<"if ($Pt4) $Rdd32 = memd($Rs32)", (L2_ploadrdt_io DoubleRegs:$Rdd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    47: def L2_ploadrdtnew_zomapAlias : InstAlias<"if ($Pt4.new) $Rdd32 = memd($Rs32)", (L2_ploadrdtnew_io DoubleRegs:$Rdd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    48: def L2_ploadrhf_zomapAlias : InstAlias<"if (!$Pt4) $Rd32 = memh($Rs32)", (L2_ploadrhf_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    49: def L2_ploadrhfnew_zomapAlias : InstAlias<"if (!$Pt4.new) $Rd32 = memh($Rs32)", (L2_ploadrhfnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    50: def L2_ploadrht_zomapAlias : InstAlias<"if ($Pt4) $Rd32 = memh($Rs32)", (L2_ploadrht_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    51: def L2_ploadrhtnew_zomapAlias : InstAlias<"if ($Pt4.new) $Rd32 = memh($Rs32)", (L2_ploadrhtnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    52: def L2_ploadrif_zomapAlias : InstAlias<"if (!$Pt4) $Rd32 = memw($Rs32)", (L2_ploadrif_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    53: def L2_ploadrifnew_zomapAlias : InstAlias<"if (!$Pt4.new) $Rd32 = memw($Rs32)", (L2_ploadrifnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    54: def L2_ploadrit_zomapAlias : InstAlias<"if ($Pt4) $Rd32 = memw($Rs32)", (L2_ploadrit_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    55: def L2_ploadritnew_zomapAlias : InstAlias<"if ($Pt4.new) $Rd32 = memw($Rs32)", (L2_ploadritnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    56: def L2_ploadrubf_zomapAlias : InstAlias<"if (!$Pt4) $Rd32 = memub($Rs32)", (L2_ploadrubf_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    57: def L2_ploadrubfnew_zomapAlias : InstAlias<"if (!$Pt4.new) $Rd32 = memub($Rs32)", (L2_ploadrubfnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    58: def L2_ploadrubt_zomapAlias : InstAlias<"if ($Pt4) $Rd32 = memub($Rs32)", (L2_ploadrubt_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    59: def L2_ploadrubtnew_zomapAlias : InstAlias<"if ($Pt4.new) $Rd32 = memub($Rs32)", (L2_ploadrubtnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    60: def L2_ploadruhf_zomapAlias : InstAlias<"if (!$Pt4) $Rd32 = memuh($Rs32)", (L2_ploadruhf_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    61: def L2_ploadruhfnew_zomapAlias : InstAlias<"if (!$Pt4.new) $Rd32 = memuh($Rs32)", (L2_ploadruhfnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    62: def L2_ploadruht_zomapAlias : InstAlias<"if ($Pt4) $Rd32 = memuh($Rs32)", (L2_ploadruht_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    63: def L2_ploadruhtnew_zomapAlias : InstAlias<"if ($Pt4.new) $Rd32 = memuh($Rs32)", (L2_ploadruhtnew_io IntRegs:$Rd32, PredRegs:$Pt4, IntRegs:$Rs32, 0)>;
    64: def L4_add_memopb_zomapAlias : InstAlias<"memb($Rs32) += $Rt32", (L4_add_memopb_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    65: def L4_add_memoph_zomapAlias : InstAlias<"memh($Rs32) += $Rt32", (L4_add_memoph_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    66: def L4_add_memopw_zomapAlias : InstAlias<"memw($Rs32) += $Rt32", (L4_add_memopw_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    67: def L4_and_memopb_zomapAlias : InstAlias<"memb($Rs32) &= $Rt32", (L4_and_memopb_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    68: def L4_and_memoph_zomapAlias : InstAlias<"memh($Rs32) &= $Rt32", (L4_and_memoph_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    69: def L4_and_memopw_zomapAlias : InstAlias<"memw($Rs32) &= $Rt32", (L4_and_memopw_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    70: def L4_iadd_memopb_zomapAlias : InstAlias<"memb($Rs32) += #$II", (L4_iadd_memopb_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    71: def L4_iadd_memoph_zomapAlias : InstAlias<"memh($Rs32) += #$II", (L4_iadd_memoph_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    72: def L4_iadd_memopw_zomapAlias : InstAlias<"memw($Rs32) += #$II", (L4_iadd_memopw_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    73: def L4_iand_memopb_zomapAlias : InstAlias<"memb($Rs32) = clrbit(#$II)", (L4_iand_memopb_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    74: def L4_iand_memoph_zomapAlias : InstAlias<"memh($Rs32) = clrbit(#$II)", (L4_iand_memoph_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    75: def L4_iand_memopw_zomapAlias : InstAlias<"memw($Rs32) = clrbit(#$II)", (L4_iand_memopw_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    76: def L4_ior_memopb_zomapAlias : InstAlias<"memb($Rs32) = setbit(#$II)", (L4_ior_memopb_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    77: def L4_ior_memoph_zomapAlias : InstAlias<"memh($Rs32) = setbit(#$II)", (L4_ior_memoph_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    78: def L4_ior_memopw_zomapAlias : InstAlias<"memw($Rs32) = setbit(#$II)", (L4_ior_memopw_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    79: def L4_isub_memopb_zomapAlias : InstAlias<"memb($Rs32) -= #$II", (L4_isub_memopb_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    80: def L4_isub_memoph_zomapAlias : InstAlias<"memh($Rs32) -= #$II", (L4_isub_memoph_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    81: def L4_isub_memopw_zomapAlias : InstAlias<"memw($Rs32) -= #$II", (L4_isub_memopw_io IntRegs:$Rs32, 0, u5_0Imm:$II)>;
    82: def L4_or_memopb_zomapAlias : InstAlias<"memb($Rs32) |= $Rt32", (L4_or_memopb_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    83: def L4_or_memoph_zomapAlias : InstAlias<"memh($Rs32) |= $Rt32", (L4_or_memoph_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    84: def L4_or_memopw_zomapAlias : InstAlias<"memw($Rs32) |= $Rt32", (L4_or_memopw_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    85: def L4_return_map_to_raw_fAlias : InstAlias<"if (!$Pv4) dealloc_return", (L4_return_f D15, PredRegs:$Pv4, R30)>;
    86: def L4_return_map_to_raw_fnew_pntAlias : InstAlias<"if (!$Pv4.new) dealloc_return:nt", (L4_return_fnew_pnt D15, PredRegs:$Pv4, R30)>;
    87: def L4_return_map_to_raw_fnew_ptAlias : InstAlias<"if (!$Pv4.new) dealloc_return:t", (L4_return_fnew_pt D15, PredRegs:$Pv4, R30)>;
    88: def L4_return_map_to_raw_tAlias : InstAlias<"if ($Pv4) dealloc_return", (L4_return_t D15, PredRegs:$Pv4, R30)>;
    89: def L4_return_map_to_raw_tnew_pntAlias : InstAlias<"if ($Pv4.new) dealloc_return:nt", (L4_return_tnew_pnt D15, PredRegs:$Pv4, R30)>;
    90: def L4_return_map_to_raw_tnew_ptAlias : InstAlias<"if ($Pv4.new) dealloc_return:t", (L4_return_tnew_pt D15, PredRegs:$Pv4, R30)>;
    91: def L4_sub_memopb_zomapAlias : InstAlias<"memb($Rs32) -= $Rt32", (L4_sub_memopb_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    92: def L4_sub_memoph_zomapAlias : InstAlias<"memh($Rs32) -= $Rt32", (L4_sub_memoph_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    93: def L4_sub_memopw_zomapAlias : InstAlias<"memw($Rs32) -= $Rt32", (L4_sub_memopw_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
    94: def L6_deallocframe_map_to_rawAlias : InstAlias<"deallocframe", (L2_deallocframe D15, R30)>;
    95: def L6_return_map_to_rawAlias : InstAlias<"dealloc_return", (L4_return D15, R30)>;
    96: def M2_mpyuiAlias : InstAlias<"$Rd32 = mpyui($Rs32,$Rt32)", (M2_mpyi IntRegs:$Rd32, IntRegs:$Rs32, IntRegs:$Rt32)>;
    97: def M7_vdmpyAlias : InstAlias<"$Rdd32 = vdmpyw($Rss32,$Rtt32)", (M7_dcmpyrwc DoubleRegs:$Rdd32, DoubleRegs:$Rss32, DoubleRegs:$Rtt32)>;
    98: def M7_vdmpy_accAlias : InstAlias<"$Rxx32 += vdmpyw($Rss32,$Rtt32)", (M7_dcmpyrwc_acc DoubleRegs:$Rxx32, DoubleRegs:$Rss32, DoubleRegs:$Rtt32)>;
    99: def S2_pstorerbf_zomapAlias : InstAlias<"if (!$Pv4) memb($Rs32) = $Rt32", (S2_pstorerbf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   100: def S2_pstorerbnewf_zomapAlias : InstAlias<"if (!$Pv4) memb($Rs32) = $Nt8.new", (S2_pstorerbnewf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
```
- EN: It defines generated/declarative TableGen records like A2_negAlias, A2_notAlias, A2_tfrfAlias, A2_tfrfnewAlias, A2_tfrtAlias, ... (90 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 A2_negAlias, A2_notAlias, A2_tfrfAlias, A2_tfrfnewAlias, A2_tfrtAlias, ... (90 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-200 / 第 101-200 行

```tablegen
   101: def S2_pstorerbnewt_zomapAlias : InstAlias<"if ($Pv4) memb($Rs32) = $Nt8.new", (S2_pstorerbnewt_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   102: def S2_pstorerbt_zomapAlias : InstAlias<"if ($Pv4) memb($Rs32) = $Rt32", (S2_pstorerbt_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   103: def S2_pstorerdf_zomapAlias : InstAlias<"if (!$Pv4) memd($Rs32) = $Rtt32", (S2_pstorerdf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, DoubleRegs:$Rtt32)>;
   104: def S2_pstorerdt_zomapAlias : InstAlias<"if ($Pv4) memd($Rs32) = $Rtt32", (S2_pstorerdt_io PredRegs:$Pv4, IntRegs:$Rs32, 0, DoubleRegs:$Rtt32)>;
   105: def S2_pstorerff_zomapAlias : InstAlias<"if (!$Pv4) memh($Rs32) = $Rt32.h", (S2_pstorerff_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   106: def S2_pstorerft_zomapAlias : InstAlias<"if ($Pv4) memh($Rs32) = $Rt32.h", (S2_pstorerft_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   107: def S2_pstorerhf_zomapAlias : InstAlias<"if (!$Pv4) memh($Rs32) = $Rt32", (S2_pstorerhf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   108: def S2_pstorerhnewf_zomapAlias : InstAlias<"if (!$Pv4) memh($Rs32) = $Nt8.new", (S2_pstorerhnewf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   109: def S2_pstorerhnewt_zomapAlias : InstAlias<"if ($Pv4) memh($Rs32) = $Nt8.new", (S2_pstorerhnewt_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   110: def S2_pstorerht_zomapAlias : InstAlias<"if ($Pv4) memh($Rs32) = $Rt32", (S2_pstorerht_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   111: def S2_pstorerif_zomapAlias : InstAlias<"if (!$Pv4) memw($Rs32) = $Rt32", (S2_pstorerif_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   112: def S2_pstorerinewf_zomapAlias : InstAlias<"if (!$Pv4) memw($Rs32) = $Nt8.new", (S2_pstorerinewf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   113: def S2_pstorerinewt_zomapAlias : InstAlias<"if ($Pv4) memw($Rs32) = $Nt8.new", (S2_pstorerinewt_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   114: def S2_pstorerit_zomapAlias : InstAlias<"if ($Pv4) memw($Rs32) = $Rt32", (S2_pstorerit_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   115: def S2_storerb_zomapAlias : InstAlias<"memb($Rs32) = $Rt32", (S2_storerb_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   116: def S2_storerbnew_zomapAlias : InstAlias<"memb($Rs32) = $Nt8.new", (S2_storerbnew_io IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   117: def S2_storerd_zomapAlias : InstAlias<"memd($Rs32) = $Rtt32", (S2_storerd_io IntRegs:$Rs32, 0, DoubleRegs:$Rtt32)>;
   118: def S2_storerf_zomapAlias : InstAlias<"memh($Rs32) = $Rt32.h", (S2_storerf_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   119: def S2_storerh_zomapAlias : InstAlias<"memh($Rs32) = $Rt32", (S2_storerh_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   120: def S2_storerhnew_zomapAlias : InstAlias<"memh($Rs32) = $Nt8.new", (S2_storerhnew_io IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   121: def S2_storeri_zomapAlias : InstAlias<"memw($Rs32) = $Rt32", (S2_storeri_io IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   122: def S2_storerinew_zomapAlias : InstAlias<"memw($Rs32) = $Nt8.new", (S2_storerinew_io IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   123: def S2_tableidxb_goodsyntaxAlias : InstAlias<"$Rx32 = tableidxb($Rs32,#$Ii,#$II)", (S2_tableidxb IntRegs:$Rx32, IntRegs:$Rs32, u4_0Imm:$Ii, u5_0Imm:$II)>;
   124: def S4_pstorerbfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memb($Rs32) = $Rt32", (S4_pstorerbfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   125: def S4_pstorerbnewfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memb($Rs32) = $Nt8.new", (S4_pstorerbnewfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   126: def S4_pstorerbnewtnew_zomapAlias : InstAlias<"if ($Pv4.new) memb($Rs32) = $Nt8.new", (S4_pstorerbnewtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   127: def S4_pstorerbtnew_zomapAlias : InstAlias<"if ($Pv4.new) memb($Rs32) = $Rt32", (S4_pstorerbtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   128: def S4_pstorerdfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memd($Rs32) = $Rtt32", (S4_pstorerdfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, DoubleRegs:$Rtt32)>;
   129: def S4_pstorerdtnew_zomapAlias : InstAlias<"if ($Pv4.new) memd($Rs32) = $Rtt32", (S4_pstorerdtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, DoubleRegs:$Rtt32)>;
   130: def S4_pstorerffnew_zomapAlias : InstAlias<"if (!$Pv4.new) memh($Rs32) = $Rt32.h", (S4_pstorerffnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   131: def S4_pstorerftnew_zomapAlias : InstAlias<"if ($Pv4.new) memh($Rs32) = $Rt32.h", (S4_pstorerftnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   132: def S4_pstorerhfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memh($Rs32) = $Rt32", (S4_pstorerhfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   133: def S4_pstorerhnewfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memh($Rs32) = $Nt8.new", (S4_pstorerhnewfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   134: def S4_pstorerhnewtnew_zomapAlias : InstAlias<"if ($Pv4.new) memh($Rs32) = $Nt8.new", (S4_pstorerhnewtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   135: def S4_pstorerhtnew_zomapAlias : InstAlias<"if ($Pv4.new) memh($Rs32) = $Rt32", (S4_pstorerhtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   136: def S4_pstorerifnew_zomapAlias : InstAlias<"if (!$Pv4.new) memw($Rs32) = $Rt32", (S4_pstorerifnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   137: def S4_pstorerinewfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memw($Rs32) = $Nt8.new", (S4_pstorerinewfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   138: def S4_pstorerinewtnew_zomapAlias : InstAlias<"if ($Pv4.new) memw($Rs32) = $Nt8.new", (S4_pstorerinewtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Nt8)>;
   139: def S4_pstoreritnew_zomapAlias : InstAlias<"if ($Pv4.new) memw($Rs32) = $Rt32", (S4_pstoreritnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, IntRegs:$Rt32)>;
   140: def S4_storeirb_zomapAlias : InstAlias<"memb($Rs32) = #$II", (S4_storeirb_io IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   141: def S4_storeirbf_zomapAlias : InstAlias<"if (!$Pv4) memb($Rs32) = #$II", (S4_storeirbf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   142: def S4_storeirbfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memb($Rs32) = #$II", (S4_storeirbfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   143: def S4_storeirbt_zomapAlias : InstAlias<"if ($Pv4) memb($Rs32) = #$II", (S4_storeirbt_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   144: def S4_storeirbtnew_zomapAlias : InstAlias<"if ($Pv4.new) memb($Rs32) = #$II", (S4_storeirbtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   145: def S4_storeirh_zomapAlias : InstAlias<"memh($Rs32) = #$II", (S4_storeirh_io IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   146: def S4_storeirhf_zomapAlias : InstAlias<"if (!$Pv4) memh($Rs32) = #$II", (S4_storeirhf_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   147: def S4_storeirhfnew_zomapAlias : InstAlias<"if (!$Pv4.new) memh($Rs32) = #$II", (S4_storeirhfnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   148: def S4_storeirht_zomapAlias : InstAlias<"if ($Pv4) memh($Rs32) = #$II", (S4_storeirht_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   149: def S4_storeirhtnew_zomapAlias : InstAlias<"if ($Pv4.new) memh($Rs32) = #$II", (S4_storeirhtnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   150: def S4_storeiri_zomapAlias : InstAlias<"memw($Rs32) = #$II", (S4_storeiri_io IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   151: def S4_storeirif_zomapAlias : InstAlias<"if (!$Pv4) memw($Rs32) = #$II", (S4_storeirif_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   152: def S4_storeirifnew_zomapAlias : InstAlias<"if (!$Pv4.new) memw($Rs32) = #$II", (S4_storeirifnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   153: def S4_storeirit_zomapAlias : InstAlias<"if ($Pv4) memw($Rs32) = #$II", (S4_storeirit_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   154: def S4_storeiritnew_zomapAlias : InstAlias<"if ($Pv4.new) memw($Rs32) = #$II", (S4_storeiritnew_io PredRegs:$Pv4, IntRegs:$Rs32, 0, s32_0Imm:$II)>;
   155: def S6_allocframe_to_rawAlias : InstAlias<"allocframe(#$Ii)", (S2_allocframe R29, u11_3Imm:$Ii)>;
   156: def V6_MAP_equbAlias : InstAlias<"$Qd4 = vcmp.eq($Vu32.ub,$Vv32.ub)", (V6_veqb HvxQR:$Qd4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   157: def V6_MAP_equb_andAlias : InstAlias<"$Qx4 &= vcmp.eq($Vu32.ub,$Vv32.ub)", (V6_veqb_and HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   158: def V6_MAP_equb_iorAlias : InstAlias<"$Qx4 |= vcmp.eq($Vu32.ub,$Vv32.ub)", (V6_veqb_or HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   159: def V6_MAP_equb_xorAlias : InstAlias<"$Qx4 ^= vcmp.eq($Vu32.ub,$Vv32.ub)", (V6_veqb_xor HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   160: def V6_MAP_equhAlias : InstAlias<"$Qd4 = vcmp.eq($Vu32.uh,$Vv32.uh)", (V6_veqh HvxQR:$Qd4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   161: def V6_MAP_equh_andAlias : InstAlias<"$Qx4 &= vcmp.eq($Vu32.uh,$Vv32.uh)", (V6_veqh_and HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   162: def V6_MAP_equh_iorAlias : InstAlias<"$Qx4 |= vcmp.eq($Vu32.uh,$Vv32.uh)", (V6_veqh_or HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   163: def V6_MAP_equh_xorAlias : InstAlias<"$Qx4 ^= vcmp.eq($Vu32.uh,$Vv32.uh)", (V6_veqh_xor HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   164: def V6_MAP_equwAlias : InstAlias<"$Qd4 = vcmp.eq($Vu32.uw,$Vv32.uw)", (V6_veqw HvxQR:$Qd4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   165: def V6_MAP_equw_andAlias : InstAlias<"$Qx4 &= vcmp.eq($Vu32.uw,$Vv32.uw)", (V6_veqw_and HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   166: def V6_MAP_equw_iorAlias : InstAlias<"$Qx4 |= vcmp.eq($Vu32.uw,$Vv32.uw)", (V6_veqw_or HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   167: def V6_MAP_equw_xorAlias : InstAlias<"$Qx4 ^= vcmp.eq($Vu32.uw,$Vv32.uw)", (V6_veqw_xor HvxQR:$Qx4, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   168: def V6_extractw_altAlias : InstAlias<"$Rd32.w = vextract($Vu32,$Rs32)", (V6_extractw IntRegs:$Rd32, HvxVR:$Vu32, IntRegs:$Rs32)>, Requires<[UseHVX]>;
   169: def V6_ld0Alias : InstAlias<"$Vd32 = vmem($Rt32)", (V6_vL32b_ai HvxVR:$Vd32, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   170: def V6_ldcnp0Alias : InstAlias<"if (!$Pv4) $Vd32.cur = vmem($Rt32)", (V6_vL32b_cur_npred_pi HvxVR:$Vd32, IntRegs:$Rt32, PredRegs:$Pv4, 0)>, Requires<[UseHVX]>;
   171: def V6_ldcnpnt0Alias : InstAlias<"if (!$Pv4) $Vd32.cur = vmem($Rt32):nt", (V6_vL32b_nt_cur_npred_pi HvxVR:$Vd32, IntRegs:$Rt32, PredRegs:$Pv4, 0)>, Requires<[UseHVX]>;
   172: def V6_ldcp0Alias : InstAlias<"if ($Pv4) $Vd32.cur = vmem($Rt32)", (V6_vL32b_cur_pred_pi HvxVR:$Vd32, IntRegs:$Rt32, PredRegs:$Pv4, 0)>, Requires<[UseHVX]>;
   173: def V6_ldcpnt0Alias : InstAlias<"if ($Pv4) $Vd32.cur = vmem($Rt32):nt", (V6_vL32b_nt_cur_pred_pi HvxVR:$Vd32, IntRegs:$Rt32, PredRegs:$Pv4, 0)>, Requires<[UseHVX]>;
   174: def V6_ldnp0Alias : InstAlias<"if (!$Pv4) $Vd32 = vmem($Rt32)", (V6_vL32b_npred_pi HvxVR:$Vd32, IntRegs:$Rt32, PredRegs:$Pv4, 0)>, Requires<[UseHVX]>;
   175: def V6_ldnpnt0Alias : InstAlias<"if (!$Pv4) $Vd32 = vmem($Rt32):nt", (V6_vL32b_nt_npred_pi HvxVR:$Vd32, IntRegs:$Rt32, PredRegs:$Pv4, 0)>, Requires<[UseHVX]>;
   176: def V6_ldnt0Alias : InstAlias<"$Vd32 = vmem($Rt32):nt", (V6_vL32b_nt_ai HvxVR:$Vd32, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   177: def V6_ldp0Alias : InstAlias<"if ($Pv4) $Vd32 = vmem($Rt32)", (V6_vL32b_pred_ai HvxVR:$Vd32, PredRegs:$Pv4, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   178: def V6_ldpnt0Alias : InstAlias<"if ($Pv4) $Vd32 = vmem($Rt32):nt", (V6_vL32b_nt_pred_ai HvxVR:$Vd32, PredRegs:$Pv4, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   179: def V6_ldtnp0Alias : InstAlias<"if (!$Pv4) $Vd32.tmp = vmem($Rt32)", (V6_vL32b_npred_ai HvxVR:$Vd32, PredRegs:$Pv4, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   180: def V6_ldtnpnt0Alias : InstAlias<"if (!$Pv4) $Vd32.tmp = vmem($Rt32):nt", (V6_vL32b_nt_npred_ai HvxVR:$Vd32, PredRegs:$Pv4, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   181: def V6_ldtp0Alias : InstAlias<"if ($Pv4) $Vd32.tmp = vmem($Rt32)", (V6_vL32b_tmp_pred_ai HvxVR:$Vd32, PredRegs:$Pv4, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   182: def V6_ldtpnt0Alias : InstAlias<"if ($Pv4) $Vd32.tmp = vmem($Rt32):nt", (V6_vL32b_nt_tmp_pred_ai HvxVR:$Vd32, PredRegs:$Pv4, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   183: def V6_ldu0Alias : InstAlias<"$Vd32 = vmemu($Rt32)", (V6_vL32Ub_ai HvxVR:$Vd32, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   184: def V6_st0Alias : InstAlias<"vmem($Rt32) = $Vs32", (V6_vS32b_ai IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   185: def V6_stn0Alias : InstAlias<"vmem($Rt32) = $Os8.new", (V6_vS32b_new_ai IntRegs:$Rt32, 0, HvxVR:$Os8)>, Requires<[UseHVX]>;
   186: def V6_stnnt0Alias : InstAlias<"vmem($Rt32):nt = $Os8.new", (V6_vS32b_nt_new_ai IntRegs:$Rt32, 0, HvxVR:$Os8)>, Requires<[UseHVX]>;
   187: def V6_stnp0Alias : InstAlias<"if (!$Pv4) vmem($Rt32) = $Vs32", (V6_vS32b_npred_ai PredRegs:$Pv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   188: def V6_stnpnt0Alias : InstAlias<"if (!$Pv4) vmem($Rt32):nt = $Vs32", (V6_vS32b_nt_npred_ai PredRegs:$Pv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   189: def V6_stnq0Alias : InstAlias<"if (!$Qv4) vmem($Rt32) = $Vs32", (V6_vS32b_nqpred_ai HvxQR:$Qv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   190: def V6_stnqnt0Alias : InstAlias<"if (!$Qv4) vmem($Rt32):nt = $Vs32", (V6_vS32b_nt_nqpred_ai HvxQR:$Qv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   191: def V6_stnt0Alias : InstAlias<"vmem($Rt32):nt = $Vs32", (V6_vS32b_nt_ai IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   192: def V6_stp0Alias : InstAlias<"if ($Pv4) vmem($Rt32) = $Vs32", (V6_vS32b_pred_ai PredRegs:$Pv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   193: def V6_stpnt0Alias : InstAlias<"if ($Pv4) vmem($Rt32):nt = $Vs32", (V6_vS32b_nt_pred_ai PredRegs:$Pv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   194: def V6_stq0Alias : InstAlias<"if ($Qv4) vmem($Rt32) = $Vs32", (V6_vS32b_qpred_ai HvxQR:$Qv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   195: def V6_stqnt0Alias : InstAlias<"if ($Qv4) vmem($Rt32):nt = $Vs32", (V6_vS32b_nt_qpred_ai HvxQR:$Qv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   196: def V6_stu0Alias : InstAlias<"vmemu($Rt32) = $Vs32", (V6_vS32Ub_ai IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   197: def V6_stunp0Alias : InstAlias<"if (!$Pv4) vmemu($Rt32) = $Vs32", (V6_vS32Ub_npred_ai PredRegs:$Pv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   198: def V6_stup0Alias : InstAlias<"if ($Pv4) vmemu($Rt32) = $Vs32", (V6_vS32Ub_pred_ai PredRegs:$Pv4, IntRegs:$Rt32, 0, HvxVR:$Vs32)>, Requires<[UseHVX]>;
   199: def V6_v6mpyhubs10_altAlias : InstAlias<"$Vdd32.w = v6mpy($Vuu32.ub,$Vvv32.b10,#$Ii):h", (V6_v6mpyhubs10 HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32, u2_0Imm:$Ii)>, Requires<[UseHVX]>;
   200: def V6_v6mpyvubs10_altAlias : InstAlias<"$Vdd32.w = v6mpy($Vuu32.ub,$Vvv32.b10,#$Ii):v", (V6_v6mpyvubs10 HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32, u2_0Imm:$Ii)>, Requires<[UseHVX]>;
```
- EN: It defines generated/declarative TableGen records like S2_pstorerbnewt_zomapAlias, S2_pstorerbt_zomapAlias, S2_pstorerdf_zomapAlias, S2_pstorerdt_zomapAlias, S2_pstorerff_zomapAlias, ... (100 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 S2_pstorerbnewt_zomapAlias, S2_pstorerbt_zomapAlias, S2_pstorerdf_zomapAlias, S2_pstorerdt_zomapAlias, S2_pstorerff_zomapAlias, ... (100 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 201-300 / 第 201-300 行

```tablegen
   201: def V6_vabsb_altAlias : InstAlias<"$Vd32 = vabsb($Vu32)", (V6_vabsb HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   202: def V6_vabsb_sat_altAlias : InstAlias<"$Vd32 = vabsb($Vu32):sat", (V6_vabsb_sat HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   203: def V6_vabsdiffh_altAlias : InstAlias<"$Vd32 = vabsdiffh($Vu32,$Vv32)", (V6_vabsdiffh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   204: def V6_vabsdiffub_altAlias : InstAlias<"$Vd32 = vabsdiffub($Vu32,$Vv32)", (V6_vabsdiffub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   205: def V6_vabsdiffuh_altAlias : InstAlias<"$Vd32 = vabsdiffuh($Vu32,$Vv32)", (V6_vabsdiffuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   206: def V6_vabsdiffw_altAlias : InstAlias<"$Vd32 = vabsdiffw($Vu32,$Vv32)", (V6_vabsdiffw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   207: def V6_vabsh_altAlias : InstAlias<"$Vd32 = vabsh($Vu32)", (V6_vabsh HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   208: def V6_vabsh_sat_altAlias : InstAlias<"$Vd32 = vabsh($Vu32):sat", (V6_vabsh_sat HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   209: def V6_vabsub_altAlias : InstAlias<"$Vd32.ub = vabs($Vu32.b)", (V6_vabsb HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   210: def V6_vabsuh_altAlias : InstAlias<"$Vd32.uh = vabs($Vu32.h)", (V6_vabsh HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   211: def V6_vabsuw_altAlias : InstAlias<"$Vd32.uw = vabs($Vu32.w)", (V6_vabsw HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   212: def V6_vabsw_altAlias : InstAlias<"$Vd32 = vabsw($Vu32)", (V6_vabsw HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   213: def V6_vabsw_sat_altAlias : InstAlias<"$Vd32 = vabsw($Vu32):sat", (V6_vabsw_sat HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   214: def V6_vaddb_altAlias : InstAlias<"$Vd32 = vaddb($Vu32,$Vv32)", (V6_vaddb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   215: def V6_vaddb_dv_altAlias : InstAlias<"$Vdd32 = vaddb($Vuu32,$Vvv32)", (V6_vaddb_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   216: def V6_vaddbnq_altAlias : InstAlias<"if (!$Qv4.b) $Vx32.b += $Vu32.b", (V6_vaddbnq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   217: def V6_vaddbq_altAlias : InstAlias<"if ($Qv4.b) $Vx32.b += $Vu32.b", (V6_vaddbq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   218: def V6_vaddbsat_altAlias : InstAlias<"$Vd32 = vaddb($Vu32,$Vv32):sat", (V6_vaddbsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   219: def V6_vaddbsat_dv_altAlias : InstAlias<"$Vdd32 = vaddb($Vuu32,$Vvv32):sat", (V6_vaddbsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   220: def V6_vaddh_altAlias : InstAlias<"$Vd32 = vaddh($Vu32,$Vv32)", (V6_vaddh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   221: def V6_vaddh_dv_altAlias : InstAlias<"$Vdd32 = vaddh($Vuu32,$Vvv32)", (V6_vaddh_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   222: def V6_vaddhnq_altAlias : InstAlias<"if (!$Qv4.h) $Vx32.h += $Vu32.h", (V6_vaddhnq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   223: def V6_vaddhq_altAlias : InstAlias<"if ($Qv4.h) $Vx32.h += $Vu32.h", (V6_vaddhq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   224: def V6_vaddhsat_altAlias : InstAlias<"$Vd32 = vaddh($Vu32,$Vv32):sat", (V6_vaddhsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   225: def V6_vaddhsat_dv_altAlias : InstAlias<"$Vdd32 = vaddh($Vuu32,$Vvv32):sat", (V6_vaddhsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   226: def V6_vaddhw_acc_altAlias : InstAlias<"$Vxx32 += vaddh($Vu32,$Vv32)", (V6_vaddhw_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   227: def V6_vaddhw_altAlias : InstAlias<"$Vdd32 = vaddh($Vu32,$Vv32)", (V6_vaddhw HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   228: def V6_vaddubh_acc_altAlias : InstAlias<"$Vxx32 += vaddub($Vu32,$Vv32)", (V6_vaddubh_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   229: def V6_vaddubh_altAlias : InstAlias<"$Vdd32 = vaddub($Vu32,$Vv32)", (V6_vaddubh HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   230: def V6_vaddubsat_altAlias : InstAlias<"$Vd32 = vaddub($Vu32,$Vv32):sat", (V6_vaddubsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   231: def V6_vaddubsat_dv_altAlias : InstAlias<"$Vdd32 = vaddub($Vuu32,$Vvv32):sat", (V6_vaddubsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   232: def V6_vadduhsat_altAlias : InstAlias<"$Vd32 = vadduh($Vu32,$Vv32):sat", (V6_vadduhsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   233: def V6_vadduhsat_dv_altAlias : InstAlias<"$Vdd32 = vadduh($Vuu32,$Vvv32):sat", (V6_vadduhsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   234: def V6_vadduhw_acc_altAlias : InstAlias<"$Vxx32 += vadduh($Vu32,$Vv32)", (V6_vadduhw_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   235: def V6_vadduhw_altAlias : InstAlias<"$Vdd32 = vadduh($Vu32,$Vv32)", (V6_vadduhw HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   236: def V6_vadduwsat_altAlias : InstAlias<"$Vd32 = vadduw($Vu32,$Vv32):sat", (V6_vadduwsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   237: def V6_vadduwsat_dv_altAlias : InstAlias<"$Vdd32 = vadduw($Vuu32,$Vvv32):sat", (V6_vadduwsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   238: def V6_vaddw_altAlias : InstAlias<"$Vd32 = vaddw($Vu32,$Vv32)", (V6_vaddw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   239: def V6_vaddw_dv_altAlias : InstAlias<"$Vdd32 = vaddw($Vuu32,$Vvv32)", (V6_vaddw_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   240: def V6_vaddwnq_altAlias : InstAlias<"if (!$Qv4.w) $Vx32.w += $Vu32.w", (V6_vaddwnq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   241: def V6_vaddwq_altAlias : InstAlias<"if ($Qv4.w) $Vx32.w += $Vu32.w", (V6_vaddwq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   242: def V6_vaddwsat_altAlias : InstAlias<"$Vd32 = vaddw($Vu32,$Vv32):sat", (V6_vaddwsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   243: def V6_vaddwsat_dv_altAlias : InstAlias<"$Vdd32 = vaddw($Vuu32,$Vvv32):sat", (V6_vaddwsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   244: def V6_vandnqrt_acc_altAlias : InstAlias<"$Vx32.ub |= vand(!$Qu4.ub,$Rt32.ub)", (V6_vandnqrt_acc HvxVR:$Vx32, HvxQR:$Qu4, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   245: def V6_vandnqrt_altAlias : InstAlias<"$Vd32.ub = vand(!$Qu4.ub,$Rt32.ub)", (V6_vandnqrt HvxVR:$Vd32, HvxQR:$Qu4, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   246: def V6_vandqrt_acc_altAlias : InstAlias<"$Vx32.ub |= vand($Qu4.ub,$Rt32.ub)", (V6_vandqrt_acc HvxVR:$Vx32, HvxQR:$Qu4, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   247: def V6_vandqrt_altAlias : InstAlias<"$Vd32.ub = vand($Qu4.ub,$Rt32.ub)", (V6_vandqrt HvxVR:$Vd32, HvxQR:$Qu4, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   248: def V6_vandvrt_acc_altAlias : InstAlias<"$Qx4.ub |= vand($Vu32.ub,$Rt32.ub)", (V6_vandvrt_acc HvxQR:$Qx4, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   249: def V6_vandvrt_altAlias : InstAlias<"$Qd4.ub = vand($Vu32.ub,$Rt32.ub)", (V6_vandvrt HvxQR:$Qd4, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   250: def V6_vaslh_acc_altAlias : InstAlias<"$Vx32 += vaslh($Vu32,$Rt32)", (V6_vaslh_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   251: def V6_vaslh_altAlias : InstAlias<"$Vd32 = vaslh($Vu32,$Rt32)", (V6_vaslh HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   252: def V6_vaslhv_altAlias : InstAlias<"$Vd32 = vaslh($Vu32,$Vv32)", (V6_vaslhv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   253: def V6_vaslw_acc_altAlias : InstAlias<"$Vx32 += vaslw($Vu32,$Rt32)", (V6_vaslw_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   254: def V6_vaslw_altAlias : InstAlias<"$Vd32 = vaslw($Vu32,$Rt32)", (V6_vaslw HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   255: def V6_vaslwv_altAlias : InstAlias<"$Vd32 = vaslw($Vu32,$Vv32)", (V6_vaslwv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   256: def V6_vasr_into_altAlias : InstAlias<"$Vxx32 = vasrinto($Vu32,$Vv32)", (V6_vasr_into HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   257: def V6_vasrh_acc_altAlias : InstAlias<"$Vx32 += vasrh($Vu32,$Rt32)", (V6_vasrh_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   258: def V6_vasrh_altAlias : InstAlias<"$Vd32 = vasrh($Vu32,$Rt32)", (V6_vasrh HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   259: def V6_vasrhv_altAlias : InstAlias<"$Vd32 = vasrh($Vu32,$Vv32)", (V6_vasrhv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   260: def V6_vasrw_acc_altAlias : InstAlias<"$Vx32 += vasrw($Vu32,$Rt32)", (V6_vasrw_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   261: def V6_vasrw_altAlias : InstAlias<"$Vd32 = vasrw($Vu32,$Rt32)", (V6_vasrw HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   262: def V6_vasrwv_altAlias : InstAlias<"$Vd32 = vasrw($Vu32,$Vv32)", (V6_vasrwv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   263: def V6_vavgb_altAlias : InstAlias<"$Vd32 = vavgb($Vu32,$Vv32)", (V6_vavgb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   264: def V6_vavgbrnd_altAlias : InstAlias<"$Vd32 = vavgb($Vu32,$Vv32):rnd", (V6_vavgbrnd HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   265: def V6_vavgh_altAlias : InstAlias<"$Vd32 = vavgh($Vu32,$Vv32)", (V6_vavgh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   266: def V6_vavghrnd_altAlias : InstAlias<"$Vd32 = vavgh($Vu32,$Vv32):rnd", (V6_vavghrnd HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   267: def V6_vavgub_altAlias : InstAlias<"$Vd32 = vavgub($Vu32,$Vv32)", (V6_vavgub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   268: def V6_vavgubrnd_altAlias : InstAlias<"$Vd32 = vavgub($Vu32,$Vv32):rnd", (V6_vavgubrnd HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   269: def V6_vavguh_altAlias : InstAlias<"$Vd32 = vavguh($Vu32,$Vv32)", (V6_vavguh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   270: def V6_vavguhrnd_altAlias : InstAlias<"$Vd32 = vavguh($Vu32,$Vv32):rnd", (V6_vavguhrnd HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   271: def V6_vavguw_altAlias : InstAlias<"$Vd32 = vavguw($Vu32,$Vv32)", (V6_vavguw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   272: def V6_vavguwrnd_altAlias : InstAlias<"$Vd32 = vavguw($Vu32,$Vv32):rnd", (V6_vavguwrnd HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   273: def V6_vavgw_altAlias : InstAlias<"$Vd32 = vavgw($Vu32,$Vv32)", (V6_vavgw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   274: def V6_vavgwrnd_altAlias : InstAlias<"$Vd32 = vavgw($Vu32,$Vv32):rnd", (V6_vavgwrnd HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   275: def V6_vcl0h_altAlias : InstAlias<"$Vd32 = vcl0h($Vu32)", (V6_vcl0h HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   276: def V6_vcl0w_altAlias : InstAlias<"$Vd32 = vcl0w($Vu32)", (V6_vcl0w HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   277: def V6_vd0Alias : InstAlias<"$Vd32 = #0", (V6_vxor HvxVR:$Vd32, HvxVR:$Vd32, HvxVR:$Vd32)>, Requires<[UseHVX]>;
   278: def V6_vdd0Alias : InstAlias<"$Vdd32 = #0", (V6_vsubw_dv HvxWR:$Vdd32, W15, W15)>, Requires<[UseHVX]>;
   279: def V6_vdealb4w_altAlias : InstAlias<"$Vd32 = vdealb4w($Vu32,$Vv32)", (V6_vdealb4w HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   280: def V6_vdealb_altAlias : InstAlias<"$Vd32 = vdealb($Vu32)", (V6_vdealb HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   281: def V6_vdealh_altAlias : InstAlias<"$Vd32 = vdealh($Vu32)", (V6_vdealh HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   282: def V6_vdmpybus_acc_altAlias : InstAlias<"$Vx32 += vdmpybus($Vu32,$Rt32)", (V6_vdmpybus_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   283: def V6_vdmpybus_altAlias : InstAlias<"$Vd32 = vdmpybus($Vu32,$Rt32)", (V6_vdmpybus HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   284: def V6_vdmpybus_dv_acc_altAlias : InstAlias<"$Vxx32 += vdmpybus($Vuu32,$Rt32)", (V6_vdmpybus_dv_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   285: def V6_vdmpybus_dv_altAlias : InstAlias<"$Vdd32 = vdmpybus($Vuu32,$Rt32)", (V6_vdmpybus_dv HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   286: def V6_vdmpyhb_acc_altAlias : InstAlias<"$Vx32 += vdmpyhb($Vu32,$Rt32)", (V6_vdmpyhb_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   287: def V6_vdmpyhb_altAlias : InstAlias<"$Vd32 = vdmpyhb($Vu32,$Rt32)", (V6_vdmpyhb HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   288: def V6_vdmpyhb_dv_acc_altAlias : InstAlias<"$Vxx32 += vdmpyhb($Vuu32,$Rt32)", (V6_vdmpyhb_dv_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   289: def V6_vdmpyhb_dv_altAlias : InstAlias<"$Vdd32 = vdmpyhb($Vuu32,$Rt32)", (V6_vdmpyhb_dv HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   290: def V6_vdmpyhisat_acc_altAlias : InstAlias<"$Vx32 += vdmpyh($Vuu32,$Rt32):sat", (V6_vdmpyhisat_acc HvxVR:$Vx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   291: def V6_vdmpyhisat_altAlias : InstAlias<"$Vd32 = vdmpyh($Vuu32,$Rt32):sat", (V6_vdmpyhisat HvxVR:$Vd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   292: def V6_vdmpyhsat_acc_altAlias : InstAlias<"$Vx32 += vdmpyh($Vu32,$Rt32):sat", (V6_vdmpyhsat_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   293: def V6_vdmpyhsat_altAlias : InstAlias<"$Vd32 = vdmpyh($Vu32,$Rt32):sat", (V6_vdmpyhsat HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   294: def V6_vdmpyhsuisat_acc_altAlias : InstAlias<"$Vx32 += vdmpyhsu($Vuu32,$Rt32,#1):sat", (V6_vdmpyhsuisat_acc HvxVR:$Vx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   295: def V6_vdmpyhsuisat_altAlias : InstAlias<"$Vd32 = vdmpyhsu($Vuu32,$Rt32,#1):sat", (V6_vdmpyhsuisat HvxVR:$Vd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   296: def V6_vdmpyhsusat_acc_altAlias : InstAlias<"$Vx32 += vdmpyhsu($Vu32,$Rt32):sat", (V6_vdmpyhsusat_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   297: def V6_vdmpyhsusat_altAlias : InstAlias<"$Vd32 = vdmpyhsu($Vu32,$Rt32):sat", (V6_vdmpyhsusat HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   298: def V6_vdmpyhvsat_acc_altAlias : InstAlias<"$Vx32 += vdmpyh($Vu32,$Vv32):sat", (V6_vdmpyhvsat_acc HvxVR:$Vx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   299: def V6_vdmpyhvsat_altAlias : InstAlias<"$Vd32 = vdmpyh($Vu32,$Vv32):sat", (V6_vdmpyhvsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   300: def V6_vdsaduh_acc_altAlias : InstAlias<"$Vxx32 += vdsaduh($Vuu32,$Rt32)", (V6_vdsaduh_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
```
- EN: It defines generated/declarative TableGen records like V6_vabsb_altAlias, V6_vabsb_sat_altAlias, V6_vabsdiffh_altAlias, V6_vabsdiffub_altAlias, V6_vabsdiffuh_altAlias, ... (100 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 V6_vabsb_altAlias, V6_vabsb_sat_altAlias, V6_vabsdiffh_altAlias, V6_vabsdiffub_altAlias, V6_vabsdiffuh_altAlias, ... (100 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 301-400 / 第 301-400 行

```tablegen
   301: def V6_vdsaduh_altAlias : InstAlias<"$Vdd32 = vdsaduh($Vuu32,$Rt32)", (V6_vdsaduh HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   302: def V6_vlsrh_altAlias : InstAlias<"$Vd32 = vlsrh($Vu32,$Rt32)", (V6_vlsrh HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   303: def V6_vlsrhv_altAlias : InstAlias<"$Vd32 = vlsrh($Vu32,$Vv32)", (V6_vlsrhv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   304: def V6_vlsrw_altAlias : InstAlias<"$Vd32 = vlsrw($Vu32,$Rt32)", (V6_vlsrw HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   305: def V6_vlsrwv_altAlias : InstAlias<"$Vd32 = vlsrw($Vu32,$Vv32)", (V6_vlsrwv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   306: def V6_vmaxb_altAlias : InstAlias<"$Vd32 = vmaxb($Vu32,$Vv32)", (V6_vmaxb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   307: def V6_vmaxh_altAlias : InstAlias<"$Vd32 = vmaxh($Vu32,$Vv32)", (V6_vmaxh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   308: def V6_vmaxub_altAlias : InstAlias<"$Vd32 = vmaxub($Vu32,$Vv32)", (V6_vmaxub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   309: def V6_vmaxuh_altAlias : InstAlias<"$Vd32 = vmaxuh($Vu32,$Vv32)", (V6_vmaxuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   310: def V6_vmaxw_altAlias : InstAlias<"$Vd32 = vmaxw($Vu32,$Vv32)", (V6_vmaxw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   311: def V6_vminb_altAlias : InstAlias<"$Vd32 = vminb($Vu32,$Vv32)", (V6_vminb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   312: def V6_vminh_altAlias : InstAlias<"$Vd32 = vminh($Vu32,$Vv32)", (V6_vminh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   313: def V6_vminub_altAlias : InstAlias<"$Vd32 = vminub($Vu32,$Vv32)", (V6_vminub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   314: def V6_vminuh_altAlias : InstAlias<"$Vd32 = vminuh($Vu32,$Vv32)", (V6_vminuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   315: def V6_vminw_altAlias : InstAlias<"$Vd32 = vminw($Vu32,$Vv32)", (V6_vminw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   316: def V6_vmpabus_acc_altAlias : InstAlias<"$Vxx32 += vmpabus($Vuu32,$Rt32)", (V6_vmpabus_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   317: def V6_vmpabus_altAlias : InstAlias<"$Vdd32 = vmpabus($Vuu32,$Rt32)", (V6_vmpabus HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   318: def V6_vmpabusv_altAlias : InstAlias<"$Vdd32 = vmpabus($Vuu32,$Vvv32)", (V6_vmpabusv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   319: def V6_vmpabuu_acc_altAlias : InstAlias<"$Vxx32 += vmpabuu($Vuu32,$Rt32)", (V6_vmpabuu_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   320: def V6_vmpabuu_altAlias : InstAlias<"$Vdd32 = vmpabuu($Vuu32,$Rt32)", (V6_vmpabuu HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   321: def V6_vmpabuuv_altAlias : InstAlias<"$Vdd32 = vmpabuu($Vuu32,$Vvv32)", (V6_vmpabuuv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   322: def V6_vmpahb_acc_altAlias : InstAlias<"$Vxx32 += vmpahb($Vuu32,$Rt32)", (V6_vmpahb_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   323: def V6_vmpahb_altAlias : InstAlias<"$Vdd32 = vmpahb($Vuu32,$Rt32)", (V6_vmpahb HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   324: def V6_vmpauhb_acc_altAlias : InstAlias<"$Vxx32 += vmpauhb($Vuu32,$Rt32)", (V6_vmpauhb_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   325: def V6_vmpauhb_altAlias : InstAlias<"$Vdd32 = vmpauhb($Vuu32,$Rt32)", (V6_vmpauhb HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   326: def V6_vmpybus_acc_altAlias : InstAlias<"$Vxx32 += vmpybus($Vu32,$Rt32)", (V6_vmpybus_acc HvxWR:$Vxx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   327: def V6_vmpybus_altAlias : InstAlias<"$Vdd32 = vmpybus($Vu32,$Rt32)", (V6_vmpybus HvxWR:$Vdd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   328: def V6_vmpybusv_acc_altAlias : InstAlias<"$Vxx32 += vmpybus($Vu32,$Vv32)", (V6_vmpybusv_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   329: def V6_vmpybusv_altAlias : InstAlias<"$Vdd32 = vmpybus($Vu32,$Vv32)", (V6_vmpybusv HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   330: def V6_vmpybv_acc_altAlias : InstAlias<"$Vxx32 += vmpyb($Vu32,$Vv32)", (V6_vmpybv_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   331: def V6_vmpybv_altAlias : InstAlias<"$Vdd32 = vmpyb($Vu32,$Vv32)", (V6_vmpybv HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   332: def V6_vmpyewuh_altAlias : InstAlias<"$Vd32 = vmpyewuh($Vu32,$Vv32)", (V6_vmpyewuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   333: def V6_vmpyh_acc_altAlias : InstAlias<"$Vxx32 += vmpyh($Vu32,$Rt32)", (V6_vmpyh_acc HvxWR:$Vxx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   334: def V6_vmpyh_altAlias : InstAlias<"$Vdd32 = vmpyh($Vu32,$Rt32)", (V6_vmpyh HvxWR:$Vdd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   335: def V6_vmpyhsat_acc_altAlias : InstAlias<"$Vxx32 += vmpyh($Vu32,$Rt32):sat", (V6_vmpyhsat_acc HvxWR:$Vxx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   336: def V6_vmpyhsrs_altAlias : InstAlias<"$Vd32 = vmpyh($Vu32,$Rt32):<<1:rnd:sat", (V6_vmpyhsrs HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   337: def V6_vmpyhss_altAlias : InstAlias<"$Vd32 = vmpyh($Vu32,$Rt32):<<1:sat", (V6_vmpyhss HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   338: def V6_vmpyhus_acc_altAlias : InstAlias<"$Vxx32 += vmpyhus($Vu32,$Vv32)", (V6_vmpyhus_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   339: def V6_vmpyhus_altAlias : InstAlias<"$Vdd32 = vmpyhus($Vu32,$Vv32)", (V6_vmpyhus HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   340: def V6_vmpyhv_acc_altAlias : InstAlias<"$Vxx32 += vmpyh($Vu32,$Vv32)", (V6_vmpyhv_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   341: def V6_vmpyhv_altAlias : InstAlias<"$Vdd32 = vmpyh($Vu32,$Vv32)", (V6_vmpyhv HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   342: def V6_vmpyhvsrs_altAlias : InstAlias<"$Vd32 = vmpyh($Vu32,$Vv32):<<1:rnd:sat", (V6_vmpyhvsrs HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   343: def V6_vmpyiewh_acc_altAlias : InstAlias<"$Vx32 += vmpyiewh($Vu32,$Vv32)", (V6_vmpyiewh_acc HvxVR:$Vx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   344: def V6_vmpyiewuh_acc_altAlias : InstAlias<"$Vx32 += vmpyiewuh($Vu32,$Vv32)", (V6_vmpyiewuh_acc HvxVR:$Vx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   345: def V6_vmpyiewuh_altAlias : InstAlias<"$Vd32 = vmpyiewuh($Vu32,$Vv32)", (V6_vmpyiewuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   346: def V6_vmpyih_acc_altAlias : InstAlias<"$Vx32 += vmpyih($Vu32,$Vv32)", (V6_vmpyih_acc HvxVR:$Vx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   347: def V6_vmpyih_altAlias : InstAlias<"$Vd32 = vmpyih($Vu32,$Vv32)", (V6_vmpyih HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   348: def V6_vmpyihb_acc_altAlias : InstAlias<"$Vx32 += vmpyihb($Vu32,$Rt32)", (V6_vmpyihb_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   349: def V6_vmpyihb_altAlias : InstAlias<"$Vd32 = vmpyihb($Vu32,$Rt32)", (V6_vmpyihb HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   350: def V6_vmpyiowh_altAlias : InstAlias<"$Vd32 = vmpyiowh($Vu32,$Vv32)", (V6_vmpyiowh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   351: def V6_vmpyiwb_acc_altAlias : InstAlias<"$Vx32 += vmpyiwb($Vu32,$Rt32)", (V6_vmpyiwb_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   352: def V6_vmpyiwb_altAlias : InstAlias<"$Vd32 = vmpyiwb($Vu32,$Rt32)", (V6_vmpyiwb HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   353: def V6_vmpyiwh_acc_altAlias : InstAlias<"$Vx32 += vmpyiwh($Vu32,$Rt32)", (V6_vmpyiwh_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   354: def V6_vmpyiwh_altAlias : InstAlias<"$Vd32 = vmpyiwh($Vu32,$Rt32)", (V6_vmpyiwh HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   355: def V6_vmpyiwub_acc_altAlias : InstAlias<"$Vx32 += vmpyiwub($Vu32,$Rt32)", (V6_vmpyiwub_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   356: def V6_vmpyiwub_altAlias : InstAlias<"$Vd32 = vmpyiwub($Vu32,$Rt32)", (V6_vmpyiwub HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   357: def V6_vmpyowh_altAlias : InstAlias<"$Vd32 = vmpyowh($Vu32,$Vv32):<<1:sat", (V6_vmpyowh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   358: def V6_vmpyowh_rnd_altAlias : InstAlias<"$Vd32 = vmpyowh($Vu32,$Vv32):<<1:rnd:sat", (V6_vmpyowh_rnd HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   359: def V6_vmpyub_acc_altAlias : InstAlias<"$Vxx32 += vmpyub($Vu32,$Rt32)", (V6_vmpyub_acc HvxWR:$Vxx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   360: def V6_vmpyub_altAlias : InstAlias<"$Vdd32 = vmpyub($Vu32,$Rt32)", (V6_vmpyub HvxWR:$Vdd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   361: def V6_vmpyubv_acc_altAlias : InstAlias<"$Vxx32 += vmpyub($Vu32,$Vv32)", (V6_vmpyubv_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   362: def V6_vmpyubv_altAlias : InstAlias<"$Vdd32 = vmpyub($Vu32,$Vv32)", (V6_vmpyubv HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   363: def V6_vmpyuh_acc_altAlias : InstAlias<"$Vxx32 += vmpyuh($Vu32,$Rt32)", (V6_vmpyuh_acc HvxWR:$Vxx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   364: def V6_vmpyuh_altAlias : InstAlias<"$Vdd32 = vmpyuh($Vu32,$Rt32)", (V6_vmpyuh HvxWR:$Vdd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   365: def V6_vmpyuhv_acc_altAlias : InstAlias<"$Vxx32 += vmpyuh($Vu32,$Vv32)", (V6_vmpyuhv_acc HvxWR:$Vxx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   366: def V6_vmpyuhv_altAlias : InstAlias<"$Vdd32 = vmpyuh($Vu32,$Vv32)", (V6_vmpyuhv HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   367: def V6_vnavgb_altAlias : InstAlias<"$Vd32 = vnavgb($Vu32,$Vv32)", (V6_vnavgb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   368: def V6_vnavgh_altAlias : InstAlias<"$Vd32 = vnavgh($Vu32,$Vv32)", (V6_vnavgh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   369: def V6_vnavgub_altAlias : InstAlias<"$Vd32 = vnavgub($Vu32,$Vv32)", (V6_vnavgub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   370: def V6_vnavgw_altAlias : InstAlias<"$Vd32 = vnavgw($Vu32,$Vv32)", (V6_vnavgw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   371: def V6_vnormamth_altAlias : InstAlias<"$Vd32 = vnormamth($Vu32)", (V6_vnormamth HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   372: def V6_vnormamtw_altAlias : InstAlias<"$Vd32 = vnormamtw($Vu32)", (V6_vnormamtw HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   373: def V6_vpackeb_altAlias : InstAlias<"$Vd32 = vpackeb($Vu32,$Vv32)", (V6_vpackeb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   374: def V6_vpackeh_altAlias : InstAlias<"$Vd32 = vpackeh($Vu32,$Vv32)", (V6_vpackeh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   375: def V6_vpackhb_sat_altAlias : InstAlias<"$Vd32 = vpackhb($Vu32,$Vv32):sat", (V6_vpackhb_sat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   376: def V6_vpackhub_sat_altAlias : InstAlias<"$Vd32 = vpackhub($Vu32,$Vv32):sat", (V6_vpackhub_sat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   377: def V6_vpackob_altAlias : InstAlias<"$Vd32 = vpackob($Vu32,$Vv32)", (V6_vpackob HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   378: def V6_vpackoh_altAlias : InstAlias<"$Vd32 = vpackoh($Vu32,$Vv32)", (V6_vpackoh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   379: def V6_vpackwh_sat_altAlias : InstAlias<"$Vd32 = vpackwh($Vu32,$Vv32):sat", (V6_vpackwh_sat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   380: def V6_vpackwuh_sat_altAlias : InstAlias<"$Vd32 = vpackwuh($Vu32,$Vv32):sat", (V6_vpackwuh_sat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   381: def V6_vpopcounth_altAlias : InstAlias<"$Vd32 = vpopcounth($Vu32)", (V6_vpopcounth HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   382: def V6_vrmpybub_rtt_acc_altAlias : InstAlias<"$Vxx32.w += vrmpy($Vu32.b,$Rtt32.ub)", (V6_vrmpybub_rtt_acc HvxWR:$Vxx32, HvxVR:$Vu32, DoubleRegs:$Rtt32)>, Requires<[UseHVX]>;
   383: def V6_vrmpybub_rtt_altAlias : InstAlias<"$Vdd32.w = vrmpy($Vu32.b,$Rtt32.ub)", (V6_vrmpybub_rtt HvxWR:$Vdd32, HvxVR:$Vu32, DoubleRegs:$Rtt32)>, Requires<[UseHVX]>;
   384: def V6_vrmpybus_acc_altAlias : InstAlias<"$Vx32 += vrmpybus($Vu32,$Rt32)", (V6_vrmpybus_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   385: def V6_vrmpybus_altAlias : InstAlias<"$Vd32 = vrmpybus($Vu32,$Rt32)", (V6_vrmpybus HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   386: def V6_vrmpybusi_acc_altAlias : InstAlias<"$Vxx32 += vrmpybus($Vuu32,$Rt32,#$Ii)", (V6_vrmpybusi_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32, u1_0Imm:$Ii)>, Requires<[UseHVX]>;
   387: def V6_vrmpybusi_altAlias : InstAlias<"$Vdd32 = vrmpybus($Vuu32,$Rt32,#$Ii)", (V6_vrmpybusi HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32, u1_0Imm:$Ii)>, Requires<[UseHVX]>;
   388: def V6_vrmpybusv_acc_altAlias : InstAlias<"$Vx32 += vrmpybus($Vu32,$Vv32)", (V6_vrmpybusv_acc HvxVR:$Vx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   389: def V6_vrmpybusv_altAlias : InstAlias<"$Vd32 = vrmpybus($Vu32,$Vv32)", (V6_vrmpybusv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   390: def V6_vrmpybv_acc_altAlias : InstAlias<"$Vx32 += vrmpyb($Vu32,$Vv32)", (V6_vrmpybv_acc HvxVR:$Vx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   391: def V6_vrmpybv_altAlias : InstAlias<"$Vd32 = vrmpyb($Vu32,$Vv32)", (V6_vrmpybv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   392: def V6_vrmpyub_acc_altAlias : InstAlias<"$Vx32 += vrmpyub($Vu32,$Rt32)", (V6_vrmpyub_acc HvxVR:$Vx32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   393: def V6_vrmpyub_altAlias : InstAlias<"$Vd32 = vrmpyub($Vu32,$Rt32)", (V6_vrmpyub HvxVR:$Vd32, HvxVR:$Vu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   394: def V6_vrmpyub_rtt_acc_altAlias : InstAlias<"$Vxx32.uw += vrmpy($Vu32.ub,$Rtt32.ub)", (V6_vrmpyub_rtt_acc HvxWR:$Vxx32, HvxVR:$Vu32, DoubleRegs:$Rtt32)>, Requires<[UseHVX]>;
   395: def V6_vrmpyub_rtt_altAlias : InstAlias<"$Vdd32.uw = vrmpy($Vu32.ub,$Rtt32.ub)", (V6_vrmpyub_rtt HvxWR:$Vdd32, HvxVR:$Vu32, DoubleRegs:$Rtt32)>, Requires<[UseHVX]>;
   396: def V6_vrmpyubi_acc_altAlias : InstAlias<"$Vxx32 += vrmpyub($Vuu32,$Rt32,#$Ii)", (V6_vrmpyubi_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32, u1_0Imm:$Ii)>, Requires<[UseHVX]>;
   397: def V6_vrmpyubi_altAlias : InstAlias<"$Vdd32 = vrmpyub($Vuu32,$Rt32,#$Ii)", (V6_vrmpyubi HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32, u1_0Imm:$Ii)>, Requires<[UseHVX]>;
   398: def V6_vrmpyubv_acc_altAlias : InstAlias<"$Vx32 += vrmpyub($Vu32,$Vv32)", (V6_vrmpyubv_acc HvxVR:$Vx32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   399: def V6_vrmpyubv_altAlias : InstAlias<"$Vd32 = vrmpyub($Vu32,$Vv32)", (V6_vrmpyubv HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   400: def V6_vrotr_altAlias : InstAlias<"$Vd32 = vrotr($Vu32,$Vv32)", (V6_vrotr HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
```
- EN: It defines generated/declarative TableGen records like V6_vdsaduh_altAlias, V6_vlsrh_altAlias, V6_vlsrhv_altAlias, V6_vlsrw_altAlias, V6_vlsrwv_altAlias, ... (100 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 V6_vdsaduh_altAlias, V6_vlsrh_altAlias, V6_vlsrhv_altAlias, V6_vlsrw_altAlias, V6_vlsrwv_altAlias, ... (100 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 401-475 / 第 401-475 行

```tablegen
   401: def V6_vroundhb_altAlias : InstAlias<"$Vd32 = vroundhb($Vu32,$Vv32):sat", (V6_vroundhb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   402: def V6_vroundhub_altAlias : InstAlias<"$Vd32 = vroundhub($Vu32,$Vv32):sat", (V6_vroundhub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   403: def V6_vrounduhub_altAlias : InstAlias<"$Vd32 = vrounduhub($Vu32,$Vv32):sat", (V6_vrounduhub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   404: def V6_vrounduwuh_altAlias : InstAlias<"$Vd32 = vrounduwuh($Vu32,$Vv32):sat", (V6_vrounduwuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   405: def V6_vroundwh_altAlias : InstAlias<"$Vd32 = vroundwh($Vu32,$Vv32):sat", (V6_vroundwh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   406: def V6_vroundwuh_altAlias : InstAlias<"$Vd32 = vroundwuh($Vu32,$Vv32):sat", (V6_vroundwuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   407: def V6_vrsadubi_acc_altAlias : InstAlias<"$Vxx32 += vrsadub($Vuu32,$Rt32,#$Ii)", (V6_vrsadubi_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32, u1_0Imm:$Ii)>, Requires<[UseHVX]>;
   408: def V6_vrsadubi_altAlias : InstAlias<"$Vdd32 = vrsadub($Vuu32,$Rt32,#$Ii)", (V6_vrsadubi HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32, u1_0Imm:$Ii)>, Requires<[UseHVX]>;
   409: def V6_vsathub_altAlias : InstAlias<"$Vd32 = vsathub($Vu32,$Vv32)", (V6_vsathub HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   410: def V6_vsatuwuh_altAlias : InstAlias<"$Vd32 = vsatuwuh($Vu32,$Vv32)", (V6_vsatuwuh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   411: def V6_vsatwh_altAlias : InstAlias<"$Vd32 = vsatwh($Vu32,$Vv32)", (V6_vsatwh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   412: def V6_vsb_altAlias : InstAlias<"$Vdd32 = vsxtb($Vu32)", (V6_vsb HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   413: def V6_vscattermh_add_altAlias : InstAlias<"vscatter($Rt32,$Mu2,$Vv32.h) += $Vw32.h", (V6_vscattermh_add IntRegs:$Rt32, ModRegs:$Mu2, HvxVR:$Vv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   414: def V6_vscattermh_altAlias : InstAlias<"vscatter($Rt32,$Mu2,$Vv32.h) = $Vw32.h", (V6_vscattermh IntRegs:$Rt32, ModRegs:$Mu2, HvxVR:$Vv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   415: def V6_vscattermhq_altAlias : InstAlias<"if ($Qs4) vscatter($Rt32,$Mu2,$Vv32.h) = $Vw32.h", (V6_vscattermhq HvxQR:$Qs4, IntRegs:$Rt32, ModRegs:$Mu2, HvxVR:$Vv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   416: def V6_vscattermw_add_altAlias : InstAlias<"vscatter($Rt32,$Mu2,$Vv32.w) += $Vw32.w", (V6_vscattermw_add IntRegs:$Rt32, ModRegs:$Mu2, HvxVR:$Vv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   417: def V6_vscattermw_altAlias : InstAlias<"vscatter($Rt32,$Mu2,$Vv32.w) = $Vw32.w", (V6_vscattermw IntRegs:$Rt32, ModRegs:$Mu2, HvxVR:$Vv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   418: def V6_vscattermwh_add_altAlias : InstAlias<"vscatter($Rt32,$Mu2,$Vvv32.w) += $Vw32.h", (V6_vscattermhw_add IntRegs:$Rt32, ModRegs:$Mu2, HvxWR:$Vvv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   419: def V6_vscattermwh_altAlias : InstAlias<"vscatter($Rt32,$Mu2,$Vvv32.w) = $Vw32.h", (V6_vscattermhw IntRegs:$Rt32, ModRegs:$Mu2, HvxWR:$Vvv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   420: def V6_vscattermwhq_altAlias : InstAlias<"if ($Qs4) vscatter($Rt32,$Mu2,$Vvv32.w) = $Vw32.h", (V6_vscattermhwq HvxQR:$Qs4, IntRegs:$Rt32, ModRegs:$Mu2, HvxWR:$Vvv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   421: def V6_vscattermwq_altAlias : InstAlias<"if ($Qs4) vscatter($Rt32,$Mu2,$Vv32.w) = $Vw32.w", (V6_vscattermwq HvxQR:$Qs4, IntRegs:$Rt32, ModRegs:$Mu2, HvxVR:$Vv32, HvxVR:$Vw32)>, Requires<[UseHVX]>;
   422: def V6_vsh_altAlias : InstAlias<"$Vdd32 = vsxth($Vu32)", (V6_vsh HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   423: def V6_vshufeh_altAlias : InstAlias<"$Vd32 = vshuffeh($Vu32,$Vv32)", (V6_vshufeh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   424: def V6_vshuffb_altAlias : InstAlias<"$Vd32 = vshuffb($Vu32)", (V6_vshuffb HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   425: def V6_vshuffeb_altAlias : InstAlias<"$Vd32 = vshuffeb($Vu32,$Vv32)", (V6_vshuffeb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   426: def V6_vshuffh_altAlias : InstAlias<"$Vd32 = vshuffh($Vu32)", (V6_vshuffh HvxVR:$Vd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   427: def V6_vshuffob_altAlias : InstAlias<"$Vd32 = vshuffob($Vu32,$Vv32)", (V6_vshuffob HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   428: def V6_vshufoeb_altAlias : InstAlias<"$Vdd32 = vshuffoeb($Vu32,$Vv32)", (V6_vshufoeb HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   429: def V6_vshufoeh_altAlias : InstAlias<"$Vdd32 = vshuffoeh($Vu32,$Vv32)", (V6_vshufoeh HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   430: def V6_vshufoh_altAlias : InstAlias<"$Vd32 = vshuffoh($Vu32,$Vv32)", (V6_vshufoh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   431: def V6_vsubb_altAlias : InstAlias<"$Vd32 = vsubb($Vu32,$Vv32)", (V6_vsubb HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   432: def V6_vsubb_dv_altAlias : InstAlias<"$Vdd32 = vsubb($Vuu32,$Vvv32)", (V6_vsubb_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   433: def V6_vsubbnq_altAlias : InstAlias<"if (!$Qv4.b) $Vx32.b -= $Vu32.b", (V6_vsubbnq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   434: def V6_vsubbq_altAlias : InstAlias<"if ($Qv4.b) $Vx32.b -= $Vu32.b", (V6_vsubbq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   435: def V6_vsubbsat_altAlias : InstAlias<"$Vd32 = vsubb($Vu32,$Vv32):sat", (V6_vsubbsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   436: def V6_vsubbsat_dv_altAlias : InstAlias<"$Vdd32 = vsubb($Vuu32,$Vvv32):sat", (V6_vsubbsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   437: def V6_vsubh_altAlias : InstAlias<"$Vd32 = vsubh($Vu32,$Vv32)", (V6_vsubh HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   438: def V6_vsubh_dv_altAlias : InstAlias<"$Vdd32 = vsubh($Vuu32,$Vvv32)", (V6_vsubh_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   439: def V6_vsubhnq_altAlias : InstAlias<"if (!$Qv4.h) $Vx32.h -= $Vu32.h", (V6_vsubhnq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   440: def V6_vsubhq_altAlias : InstAlias<"if ($Qv4.h) $Vx32.h -= $Vu32.h", (V6_vsubhq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   441: def V6_vsubhsat_altAlias : InstAlias<"$Vd32 = vsubh($Vu32,$Vv32):sat", (V6_vsubhsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   442: def V6_vsubhsat_dv_altAlias : InstAlias<"$Vdd32 = vsubh($Vuu32,$Vvv32):sat", (V6_vsubhsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   443: def V6_vsubhw_altAlias : InstAlias<"$Vdd32 = vsubh($Vu32,$Vv32)", (V6_vsubhw HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   444: def V6_vsububh_altAlias : InstAlias<"$Vdd32 = vsubub($Vu32,$Vv32)", (V6_vsububh HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   445: def V6_vsububsat_altAlias : InstAlias<"$Vd32 = vsubub($Vu32,$Vv32):sat", (V6_vsububsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   446: def V6_vsububsat_dv_altAlias : InstAlias<"$Vdd32 = vsubub($Vuu32,$Vvv32):sat", (V6_vsububsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   447: def V6_vsubuhsat_altAlias : InstAlias<"$Vd32 = vsubuh($Vu32,$Vv32):sat", (V6_vsubuhsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   448: def V6_vsubuhsat_dv_altAlias : InstAlias<"$Vdd32 = vsubuh($Vuu32,$Vvv32):sat", (V6_vsubuhsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   449: def V6_vsubuhw_altAlias : InstAlias<"$Vdd32 = vsubuh($Vu32,$Vv32)", (V6_vsubuhw HvxWR:$Vdd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   450: def V6_vsubuwsat_altAlias : InstAlias<"$Vd32 = vsubuw($Vu32,$Vv32):sat", (V6_vsubuwsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   451: def V6_vsubuwsat_dv_altAlias : InstAlias<"$Vdd32 = vsubuw($Vuu32,$Vvv32):sat", (V6_vsubuwsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   452: def V6_vsubw_altAlias : InstAlias<"$Vd32 = vsubw($Vu32,$Vv32)", (V6_vsubw HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   453: def V6_vsubw_dv_altAlias : InstAlias<"$Vdd32 = vsubw($Vuu32,$Vvv32)", (V6_vsubw_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   454: def V6_vsubwnq_altAlias : InstAlias<"if (!$Qv4.w) $Vx32.w -= $Vu32.w", (V6_vsubwnq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   455: def V6_vsubwq_altAlias : InstAlias<"if ($Qv4.w) $Vx32.w -= $Vu32.w", (V6_vsubwq HvxVR:$Vx32, HvxQR:$Qv4, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   456: def V6_vsubwsat_altAlias : InstAlias<"$Vd32 = vsubw($Vu32,$Vv32):sat", (V6_vsubwsat HvxVR:$Vd32, HvxVR:$Vu32, HvxVR:$Vv32)>, Requires<[UseHVX]>;
   457: def V6_vsubwsat_dv_altAlias : InstAlias<"$Vdd32 = vsubw($Vuu32,$Vvv32):sat", (V6_vsubwsat_dv HvxWR:$Vdd32, HvxWR:$Vuu32, HvxWR:$Vvv32)>, Requires<[UseHVX]>;
   458: def V6_vtmpyb_acc_altAlias : InstAlias<"$Vxx32 += vtmpyb($Vuu32,$Rt32)", (V6_vtmpyb_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   459: def V6_vtmpyb_altAlias : InstAlias<"$Vdd32 = vtmpyb($Vuu32,$Rt32)", (V6_vtmpyb HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   460: def V6_vtmpybus_acc_altAlias : InstAlias<"$Vxx32 += vtmpybus($Vuu32,$Rt32)", (V6_vtmpybus_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   461: def V6_vtmpybus_altAlias : InstAlias<"$Vdd32 = vtmpybus($Vuu32,$Rt32)", (V6_vtmpybus HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   462: def V6_vtmpyhb_acc_altAlias : InstAlias<"$Vxx32 += vtmpyhb($Vuu32,$Rt32)", (V6_vtmpyhb_acc HvxWR:$Vxx32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   463: def V6_vtmpyhb_altAlias : InstAlias<"$Vdd32 = vtmpyhb($Vuu32,$Rt32)", (V6_vtmpyhb HvxWR:$Vdd32, HvxWR:$Vuu32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   464: def V6_vtran2x2_mapAlias : InstAlias<"vtrans2x2($Vy32,$Vx32,$Rt32)", (V6_vshuff HvxVR:$Vy32, HvxVR:$Vx32, IntRegs:$Rt32)>, Requires<[UseHVX]>;
   465: def V6_vunpackb_altAlias : InstAlias<"$Vdd32 = vunpackb($Vu32)", (V6_vunpackb HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   466: def V6_vunpackh_altAlias : InstAlias<"$Vdd32 = vunpackh($Vu32)", (V6_vunpackh HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   467: def V6_vunpackoh_altAlias : InstAlias<"$Vxx32 |= vunpackoh($Vu32)", (V6_vunpackoh HvxWR:$Vxx32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   468: def V6_vunpackub_altAlias : InstAlias<"$Vdd32 = vunpackub($Vu32)", (V6_vunpackub HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   469: def V6_vunpackuh_altAlias : InstAlias<"$Vdd32 = vunpackuh($Vu32)", (V6_vunpackuh HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   470: def V6_vzb_altAlias : InstAlias<"$Vdd32 = vzxtb($Vu32)", (V6_vzb HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   471: def V6_vzh_altAlias : InstAlias<"$Vdd32 = vzxth($Vu32)", (V6_vzh HvxWR:$Vdd32, HvxVR:$Vu32)>, Requires<[UseHVX]>;
   472: def V6_zld0Alias : InstAlias<"z = vmem($Rt32)", (V6_zLd_ai IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   473: def V6_zldp0Alias : InstAlias<"if ($Pv4) z = vmem($Rt32)", (V6_zLd_pred_ai PredRegs:$Pv4, IntRegs:$Rt32, 0)>, Requires<[UseHVX]>;
   474: def Y2_crswap_oldAlias : InstAlias<"crswap($Rx32,sgp)", (Y2_crswap0 IntRegs:$Rx32)>;
   475: def Y2_dcfetchAlias : InstAlias<"dcfetch($Rs32)", (Y2_dcfetchbo IntRegs:$Rs32, 0)>;
```
- EN: It defines generated/declarative TableGen records like V6_vroundhb_altAlias, V6_vroundhub_altAlias, V6_vrounduhub_altAlias, V6_vrounduwuh_altAlias, V6_vroundwh_altAlias, ... (75 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 V6_vroundhb_altAlias, V6_vroundhub_altAlias, V6_vrounduhub_altAlias, V6_vrounduwuh_altAlias, V6_vroundwh_altAlias, ... (75 total) 等生成式/声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录
- generated target metadata / 生成的目标元数据

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
