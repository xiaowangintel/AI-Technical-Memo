# HexagonIntrinsics.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonIntrinsics.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon intrinsic declarations and mappings using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```tablegen
     1: //===-- HexagonIntrinsics.td - Instruction intrinsics ------*- tablegen -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // These intrinsic patterns are not auto-generated.
    10: 
    11: class T_R_pat <InstHexagon MI, Intrinsic IntID>
    12:   : Pat <(IntID I32:$Rs),
    13:          (MI I32:$Rs)>;
    14: 
    15: class T_RR_pat <InstHexagon MI, Intrinsic IntID>
    16:   : Pat <(IntID I32:$Rs, I32:$Rt),
    17:          (MI I32:$Rs, I32:$Rt)>;
    18: 
    19: class T_RP_pat <InstHexagon MI, Intrinsic IntID>
    20:   : Pat <(IntID I32:$Rs, I64:$Rt),
    21:          (MI I32:$Rs, I64:$Rt)>;
    22: 
    23: def: Pat<(int_hexagon_A2_add IntRegs:$Rs, IntRegs:$Rt),
    24:          (A2_add IntRegs:$Rs, IntRegs:$Rt)>;
    25: def: Pat<(int_hexagon_A2_addi IntRegs:$Rs, timm:$s16),
    26:          (A2_addi IntRegs:$Rs, imm:$s16)>;
    27: def: Pat<(int_hexagon_A2_addp DoubleRegs:$Rs, DoubleRegs:$Rt),
    28:          (A2_addp DoubleRegs:$Rs, DoubleRegs:$Rt)>;
    29: 
    30: def: Pat<(int_hexagon_A2_sub IntRegs:$Rs, IntRegs:$Rt),
    31:          (A2_sub IntRegs:$Rs, IntRegs:$Rt)>;
    32: def: Pat<(int_hexagon_A2_subri timm:$s10, IntRegs:$Rs),
    33:          (A2_subri imm:$s10, IntRegs:$Rs)>;
    34: def: Pat<(int_hexagon_A2_subp DoubleRegs:$Rs, DoubleRegs:$Rt),
    35:          (A2_subp DoubleRegs:$Rs, DoubleRegs:$Rt)>;
    36: 
    37: def: Pat<(int_hexagon_M2_mpyi IntRegs:$Rs, IntRegs:$Rt),
    38:          (M2_mpyi IntRegs:$Rs, IntRegs:$Rt)>;
    39: def: Pat<(int_hexagon_M2_mpyui IntRegs:$Rs, IntRegs:$Rt), // Same as M2_mpyi
    40:          (M2_mpyi IntRegs:$Rs, IntRegs:$Rt)>;
    41: def: Pat<(int_hexagon_M2_mpysmi IntRegs:$Rs, imm:$s9),
    42:          (M2_mpysmi IntRegs:$Rs, imm:$s9)>;
    43: def: Pat<(int_hexagon_M2_dpmpyss_s0 IntRegs:$Rs, IntRegs:$Rt),
    44:          (M2_dpmpyss_s0 IntRegs:$Rs, IntRegs:$Rt)>;
    45: def: Pat<(int_hexagon_M2_dpmpyuu_s0 IntRegs:$Rs, IntRegs:$Rt),
    46:          (M2_dpmpyuu_s0 IntRegs:$Rs, IntRegs:$Rt)>;
    47: 
    48: def: Pat<(int_hexagon_S2_asl_i_r IntRegs:$Rs, timm:$u5),
    49:          (S2_asl_i_r IntRegs:$Rs, imm:$u5)>;
    50: def: Pat<(int_hexagon_S2_lsr_i_r IntRegs:$Rs, timm:$u5),
```
- EN: It declares types such as T_R_pat, T_RR_pat, T_RP_pat, which carry the state or API of this component. It defines declarative TableGen records like T_R_pat, T_RR_pat, T_RP_pat; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonIntrinsics, showing how the code connects to sibling backend components.
- CN: 这里声明了 T_R_pat, T_RR_pat, T_RP_pat 等类型，用来承载该组件的状态或接口。 这里定义了 T_R_pat, T_RR_pat, T_RP_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonIntrinsics，说明了它与同级后端组件的连接关系。

### Lines 51-100 / 第 51-100 行

```tablegen
    51:          (S2_lsr_i_r IntRegs:$Rs, imm:$u5)>;
    52: def: Pat<(int_hexagon_S2_asr_i_r IntRegs:$Rs, timm:$u5),
    53:          (S2_asr_i_r IntRegs:$Rs, imm:$u5)>;
    54: def: Pat<(int_hexagon_S2_asl_i_p DoubleRegs:$Rs, timm:$u6),
    55:          (S2_asl_i_p DoubleRegs:$Rs, imm:$u6)>;
    56: def: Pat<(int_hexagon_S2_lsr_i_p DoubleRegs:$Rs, timm:$u6),
    57:          (S2_lsr_i_p DoubleRegs:$Rs, imm:$u6)>;
    58: def: Pat<(int_hexagon_S2_asr_i_p DoubleRegs:$Rs, timm:$u6),
    59:          (S2_asr_i_p DoubleRegs:$Rs, imm:$u6)>;
    60: 
    61: def: Pat<(int_hexagon_A2_and IntRegs:$Rs, IntRegs:$Rt),
    62:          (A2_and IntRegs:$Rs, IntRegs:$Rt)>;
    63: def: Pat<(int_hexagon_A2_andir IntRegs:$Rs, timm:$s10),
    64:          (A2_andir IntRegs:$Rs, imm:$s10)>;
    65: def: Pat<(int_hexagon_A2_or IntRegs:$Rs, IntRegs:$Rt),
    66:          (A2_or IntRegs:$Rs, IntRegs:$Rt)>;
    67: def: Pat<(int_hexagon_A2_orir IntRegs:$Rs, timm:$s10),
    68:          (A2_orir IntRegs:$Rs, imm:$s10)>;
    69: def: Pat<(int_hexagon_A2_xor IntRegs:$Rs, IntRegs:$Rt),
    70:          (A2_xor IntRegs:$Rs, IntRegs:$Rt)>;
    71: 
    72: def: Pat<(int_hexagon_A2_sxtb IntRegs:$Rs),
    73:          (A2_sxtb IntRegs:$Rs)>;
    74: def: Pat<(int_hexagon_A2_sxth IntRegs:$Rs),
    75:          (A2_sxth IntRegs:$Rs)>;
    76: def: Pat<(int_hexagon_A2_zxtb IntRegs:$Rs),
    77:          (A2_zxtb IntRegs:$Rs)>;
    78: def: Pat<(int_hexagon_A2_zxth IntRegs:$Rs),
    79:          (A2_zxth IntRegs:$Rs)>;
    80: 
    81: // Assembler mapped from Rd32=not(Rs32) to Rd32=sub(#-1,Rs32)
    82: def : Pat <(int_hexagon_A2_not I32:$Rs),
    83:            (A2_subri -1, I32:$Rs)>;
    84: 
    85: // Assembler mapped from Rd32=neg(Rs32) to Rd32=sub(#0,Rs32)
    86: def : Pat <(int_hexagon_A2_neg I32:$Rs),
    87:            (A2_subri 0, I32:$Rs)>;
    88: 
    89: // Make sure the patterns with zero immediate value has higher complexity
    90: // otherwise, we need to updated the predicates for immediates to exclude zero
    91: let AddedComplexity = 200 in {
    92: def : Pat <(int_hexagon_S2_asr_i_r_rnd_goodsyntax I32:$Rs, (i32 0)),
    93:            (A2_tfr I32:$Rs)>;
    94: def : Pat <(int_hexagon_S2_asr_i_p_rnd_goodsyntax I64:$Rs, (i32 0)),
    95:            (A2_combinew (HiReg I64:$Rs), (LoReg I64:$Rs))>;
    96: def : Pat <(int_hexagon_S5_vasrhrnd_goodsyntax I64:$Rs, (i32 0)),
    97:            (A2_combinew (HiReg I64:$Rs), (LoReg I64:$Rs))>;
    98: def : Pat <(int_hexagon_S5_asrhub_rnd_sat_goodsyntax I64:$Rs, (i32 0)),
    99:            (S2_vsathub I64:$Rs)>;
   100: }
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-150 / 第 101-150 行

```tablegen
   101: 
   102: def : Pat <(int_hexagon_S2_asr_i_r_rnd_goodsyntax I32:$Rs, u5_0ImmPred_timm:$imm),
   103:            (S2_asr_i_r_rnd I32:$Rs, (UDEC1 u5_0ImmPred:$imm))>;
   104: def : Pat <(int_hexagon_S2_asr_i_p_rnd_goodsyntax I64:$Rs, u6_0ImmPred_timm:$imm),
   105:            (S2_asr_i_p_rnd I64:$Rs, (UDEC1 u6_0ImmPred:$imm))>;
   106: def : Pat <(int_hexagon_S5_vasrhrnd_goodsyntax I64:$Rs, u4_0ImmPred_timm:$imm),
   107:            (S5_vasrhrnd I64:$Rs, (UDEC1 u4_0ImmPred:$imm))>;
   108: def : Pat <(int_hexagon_S5_asrhub_rnd_sat_goodsyntax I64:$Rs, u4_0ImmPred_timm:$imm),
   109:            (S5_asrhub_rnd_sat I64:$Rs, (UDEC1 u4_0ImmPred:$imm))>;
   110: 
   111: def ImmExt64: SDNodeXForm<imm, [{
   112:   int64_t V = N->getSExtValue();
   113:   return CurDAG->getTargetConstant(V, SDLoc(N), MVT::i64);
   114: }]>;
   115: 
   116: // A2_tfrpi has an operand of type i64. This is necessary, since it is
   117: // generated from "(set I64:$Rd, imm)". That pattern would not appear
   118: // in the DAG, if the immediate was not a 64-bit value.
   119: // The builtin for A2_tfrpi, on the other hand, takes a 32-bit value,
   120: // which makes it impossible to simply replace it with the instruction.
   121: // To connect the builtin with the instruction, the builtin's operand
   122: // needs to be extended to the right type.
   123: 
   124: def : Pat<(int_hexagon_A2_tfrpi timm:$Is),
   125:           (A2_tfrpi (ImmExt64 $Is))>;
   126: 
   127: def : Pat <(int_hexagon_C2_cmpgei I32:$src1, s32_0ImmPred_timm:$src2),
   128:            (C2_tfrpr (C2_cmpgti I32:$src1, (SDEC1 s32_0ImmPred:$src2)))>;
   129: 
   130: def : Pat <(int_hexagon_C2_cmpgeui I32:$src1, u32_0ImmPred_timm:$src2),
   131:            (C2_tfrpr (C2_cmpgtui I32:$src1, (UDEC1 u32_0ImmPred:$src2)))>;
   132: 
   133: def : Pat <(int_hexagon_C2_cmpgeui I32:$src, 0),
   134:            (C2_tfrpr (C2_cmpeq I32:$src, I32:$src))>;
   135: def : Pat <(int_hexagon_C2_cmplt I32:$src1, I32:$src2),
   136:            (C2_tfrpr (C2_cmpgt I32:$src2, I32:$src1))>;
   137: def : Pat <(int_hexagon_C2_cmpltu I32:$src1, I32:$src2),
   138:            (C2_tfrpr (C2_cmpgtu I32:$src2, I32:$src1))>;
   139: 
   140: //===----------------------------------------------------------------------===//
   141: // Template 'def pat' to map tableidx[bhwd] intrinsics to :raw instructions.
   142: //===----------------------------------------------------------------------===//
   143: class S2op_tableidx_pat <Intrinsic IntID, InstHexagon OutputInst,
   144:                          SDNodeXForm XformImm>
   145:   : Pat <(IntID I32:$src1, I32:$src2, u4_0ImmPred_timm:$src3, u5_0ImmPred_timm:$src4),
   146:          (OutputInst I32:$src1, I32:$src2, u4_0ImmPred:$src3,
   147:                      (XformImm u5_0ImmPred:$src4))>;
   148: 
   149: def SDEC2 : SDNodeXForm<imm, [{
   150:   int32_t V = N->getSExtValue();
```
- EN: It declares types such as S2op_tableidx_pat, which carry the state or API of this component. It defines declarative TableGen records like ImmExt64, S2op_tableidx_pat, SDEC2; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSExtValue, getTargetConstant, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 S2op_tableidx_pat 等类型，用来承载该组件的状态或接口。 这里定义了 ImmExt64, S2op_tableidx_pat, SDEC2 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSExtValue, getTargetConstant 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```tablegen
   151:   return CurDAG->getSignedTargetConstant(V-2, SDLoc(N), MVT::i32);
   152: }]>;
   153: 
   154: def SDEC3 : SDNodeXForm<imm, [{
   155:   int32_t V = N->getSExtValue();
   156:   return CurDAG->getSignedTargetConstant(V-3, SDLoc(N), MVT::i32);
   157: }]>;
   158: 
   159: // Table Index : Extract and insert bits.
   160: // Map to the real hardware instructions after subtracting appropriate
   161: // values from the 4th input operand. Please note that subtraction is not
   162: // needed for int_hexagon_S2_tableidxb_goodsyntax.
   163: 
   164: def : S2op_tableidx_pat <int_hexagon_S2_tableidxb_goodsyntax, S2_tableidxb,
   165:                          IdImm>;
   166: def : S2op_tableidx_pat <int_hexagon_S2_tableidxh_goodsyntax, S2_tableidxh,
   167:                          SDEC1>;
   168: def : S2op_tableidx_pat <int_hexagon_S2_tableidxw_goodsyntax, S2_tableidxw,
   169:                          SDEC2>;
   170: def : S2op_tableidx_pat <int_hexagon_S2_tableidxd_goodsyntax, S2_tableidxd,
   171:                          SDEC3>;
   172: 
   173: // Load/store locked.
   174: def : T_R_pat<L2_loadw_locked, int_hexagon_L2_loadw_locked>;
   175: def : T_R_pat<L4_loadd_locked, int_hexagon_L4_loadd_locked>;
   176: 
   177: def : Pat<(int_hexagon_S2_storew_locked I32:$Rs, I32:$Rt),
   178:           (C2_tfrpr (S2_storew_locked I32:$Rs, I32:$Rt))>;
   179: def : Pat<(int_hexagon_S4_stored_locked I32:$Rs, I64:$Rt),
   180:           (C2_tfrpr (S4_stored_locked I32:$Rs, I64:$Rt))>;
   181: 
   182: //*******************************************************************
   183: //           ST
   184: //*******************************************************************
   185: 
   186: class T_stb_pat <InstHexagon MI, Intrinsic IntID, PatLeaf Val>
   187:   : Pat<(IntID I32:$Rs, Val:$Rt, I32:$Ru),
   188:         (MI I32:$Rs, I32:$Ru, Val:$Rt)>;
   189: 
   190: def : T_stb_pat <S2_storerh_pbr, int_hexagon_S2_storerh_pbr, I32>;
   191: def : T_stb_pat <S2_storerb_pbr, int_hexagon_S2_storerb_pbr, I32>;
   192: def : T_stb_pat <S2_storeri_pbr, int_hexagon_S2_storeri_pbr, I32>;
   193: def : T_stb_pat <S2_storerf_pbr, int_hexagon_S2_storerf_pbr, I32>;
   194: def : T_stb_pat <S2_storerd_pbr, int_hexagon_S2_storerd_pbr, I64>;
   195: 
   196: class T_stc_pat <InstHexagon MI, Intrinsic IntID, PatLeaf Imm, PatLeaf Val>
   197:   : Pat<(IntID I32:$Rs, Val:$Rt, I32:$Ru, Imm:$s),
   198:         (MI I32:$Rs, Imm:$s, I32:$Ru, Val:$Rt)>;
   199: 
   200: def: T_stc_pat<S2_storerb_pci, int_hexagon_circ_stb,   s4_0ImmPred_timm, I32>;
```
- EN: It declares types such as T_stb_pat, T_stc_pat, which carry the state or API of this component. It defines declarative TableGen records like SDEC3, T_stb_pat, T_stc_pat; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSignedTargetConstant, getSExtValue, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 T_stb_pat, T_stc_pat 等类型，用来承载该组件的状态或接口。 这里定义了 SDEC3, T_stb_pat, T_stc_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSignedTargetConstant, getSExtValue 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```tablegen
   201: def: T_stc_pat<S2_storerh_pci, int_hexagon_circ_sth,   s4_1ImmPred_timm, I32>;
   202: def: T_stc_pat<S2_storeri_pci, int_hexagon_circ_stw,   s4_2ImmPred_timm, I32>;
   203: def: T_stc_pat<S2_storerd_pci, int_hexagon_circ_std,   s4_3ImmPred_timm, I64>;
   204: def: T_stc_pat<S2_storerf_pci, int_hexagon_circ_sthhi, s4_1ImmPred_timm, I32>;
   205: 
   206: multiclass MaskedStore <InstHexagon MI, Intrinsic IntID> {
   207:   def : Pat<(IntID HvxQR:$src1, IntRegs:$src2, HvxVR:$src3),
   208:             (MI HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>,
   209:         Requires<[UseHVX]>;
   210: 
   211:   def : Pat<(!cast<Intrinsic>(IntID#"_128B") HvxQR:$src1, IntRegs:$src2,
   212:                                              HvxVR:$src3),
   213:             (MI HvxQR:$src1, IntRegs:$src2, 0, HvxVR:$src3)>,
   214:         Requires<[UseHVX]>;
   215: }
   216: 
   217: defm : MaskedStore <V6_vS32b_qpred_ai, int_hexagon_V6_vmaskedstoreq>;
   218: defm : MaskedStore <V6_vS32b_nqpred_ai, int_hexagon_V6_vmaskedstorenq>;
   219: defm : MaskedStore <V6_vS32b_nt_qpred_ai, int_hexagon_V6_vmaskedstorentq>;
   220: defm : MaskedStore <V6_vS32b_nt_nqpred_ai, int_hexagon_V6_vmaskedstorentnq>;
   221: 
   222: defm : MaskedStore <V6_vS32b_qpred_ai, int_hexagon_V6_vS32b_qpred_ai>;
   223: defm : MaskedStore <V6_vS32b_nqpred_ai, int_hexagon_V6_vS32b_nqpred_ai>;
   224: defm : MaskedStore <V6_vS32b_nt_qpred_ai, int_hexagon_V6_vS32b_nt_qpred_ai>;
   225: defm : MaskedStore <V6_vS32b_nt_nqpred_ai, int_hexagon_V6_vS32b_nt_nqpred_ai>;
   226: 
   227: //*******************************************************************
   228: //           SYSTEM
   229: //*******************************************************************
   230: 
   231: def: T_R_pat<Y2_dccleana,    int_hexagon_Y2_dccleana>;
   232: def: T_R_pat<Y2_dccleaninva, int_hexagon_Y2_dccleaninva>;
   233: def: T_R_pat<Y2_dcinva,      int_hexagon_Y2_dcinva>;
   234: def: T_R_pat<Y2_dczeroa,     int_hexagon_Y2_dczeroa>;
   235: 
   236: def: T_RR_pat<Y4_l2fetch,    int_hexagon_Y4_l2fetch>;
   237: def: T_RP_pat<Y5_l2fetch,    int_hexagon_Y5_l2fetch>;
   238: 
   239: def: Pat<(int_hexagon_Y2_dcfetch I32:$Rt), (Y2_dcfetchbo I32:$Rt, 0)>;
   240: 
   241: //
   242: // Patterns for optimizing code generations for HVX.
   243: 
   244: def u3_64_ImmPred  : PatLeaf<(i32 imm), [{
   245:   int64_t v = (int64_t)(64 - N->getSExtValue());
   246:   return isUInt<3>(v);
   247: }]>;
   248: 
   249: def u3_128_ImmPred  : PatLeaf<(i32 imm), [{
   250:   int64_t v = (int64_t)(128 - N->getSExtValue());
```
- EN: It defines declarative TableGen records like MaskedStore, u3_64_ImmPred, u3_128_ImmPred; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSExtValue, isUInt<3>, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 MaskedStore, u3_64_ImmPred, u3_128_ImmPred 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSExtValue, isUInt<3> 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-300 / 第 251-300 行

```tablegen
   251:   return isUInt<3>(v);
   252: }]>;
   253: 
   254: def SUB_64_VAL : SDNodeXForm<imm, [{
   255:    int32_t Imm = N->getSExtValue();
   256:    return CurDAG->getTargetConstant(64 - Imm, SDLoc(N), MVT::i32);
   257: }]>;
   258: 
   259: def SUB_128_VAL : SDNodeXForm<imm, [{
   260:    int32_t Imm = N->getSExtValue();
   261:    return CurDAG->getTargetConstant(128 - Imm, SDLoc(N), MVT::i32);
   262: }]>;
   263: 
   264: let AddedComplexity = 100 in {
   265: def : Pat <(v16i32 (int_hexagon_V6_lo (v32i32 HvxWR:$src1))),
   266:            (v16i32 (EXTRACT_SUBREG (v32i32 HvxWR:$src1), vsub_lo))>,
   267:            Requires<[UseHVX]>;
   268: 
   269: def : Pat <(v16i32 (int_hexagon_V6_hi (v32i32 HvxWR:$src1))),
   270:            (v16i32 (EXTRACT_SUBREG (v32i32 HvxWR:$src1), vsub_hi))>,
   271:            Requires<[UseHVX]>;
   272: 
   273: def : Pat <(v32i32 (int_hexagon_V6_lo_128B (v64i32 HvxWR:$src1))),
   274:            (v32i32 (EXTRACT_SUBREG (v64i32 HvxWR:$src1), vsub_lo))>,
   275:            Requires<[UseHVX]>;
   276: 
   277: def : Pat <(v32i32 (int_hexagon_V6_hi_128B (v64i32 HvxWR:$src1))),
   278:            (v32i32 (EXTRACT_SUBREG (v64i32 HvxWR:$src1), vsub_hi))>,
   279:            Requires<[UseHVX]>;
   280: }
   281: 
   282: def: Pat<(v64i16 (trunc v64i32:$Vdd)),
   283:          (v64i16 (V6_vpackwh_sat
   284:                  (v32i32 (V6_hi HvxWR:$Vdd)),
   285:                  (v32i32 (V6_lo HvxWR:$Vdd))))>,
   286:      Requires<[UseHVX]>;
   287: 
   288: multiclass T_VI_pat <InstHexagon MI, Intrinsic IntID> {
   289:   def: Pat<(IntID HvxVR:$src1, u3_0ImmPred:$src2),
   290:            (MI    HvxVR:$src1, HvxVR:$src1, u3_0ImmPred:$src2)>,
   291:        Requires<[UseHVX]>;
   292: 
   293:   def: Pat<(!cast<Intrinsic>(IntID#"_128B") HvxVR:$src1, u3_0ImmPred:$src2),
   294:            (MI                 HvxVR:$src1, HvxVR:$src1, u3_0ImmPred:$src2)>,
   295:        Requires<[UseHVX]>;
   296: }
   297: 
   298: multiclass T_VI_inv_pat <InstHexagon MI, Intrinsic IntID> {
   299:   def: Pat<(IntID HvxVR:$src1, u3_64_ImmPred:$src2),
   300:            (MI    HvxVR:$src1, HvxVR:$src1,
```
- EN: It defines declarative TableGen records like SUB_64_VAL, SUB_128_VAL, T_VI_pat, T_VI_inv_pat; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as isUInt<3>, getSExtValue, getTargetConstant, translating Hexagon-specific policy into reusable code paths.
- CN: 这里定义了 SUB_64_VAL, SUB_128_VAL, T_VI_pat, T_VI_inv_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 isUInt<3>, getSExtValue, getTargetConstant 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 301-350 / 第 301-350 行

```tablegen
   301:                   (SUB_64_VAL u3_64_ImmPred:$src2))>,
   302:        Requires<[UseHVX]>;
   303: 
   304:   def: Pat<(!cast<Intrinsic>(IntID#"_128B") HvxVR:$src1, u3_128_ImmPred:$src2),
   305:            (MI HvxVR:$src1, HvxVR:$src1, (SUB_128_VAL u3_128_ImmPred:$src2))>,
   306:        Requires<[UseHVX]>;
   307: }
   308: 
   309: multiclass T_VVI_pat <InstHexagon MI, Intrinsic IntID> {
   310:   def: Pat<(IntID HvxVR:$src1, HvxVR:$src2, u3_0ImmPred:$src3),
   311:            (MI    HvxVR:$src1, HvxVR:$src2, u3_0ImmPred:$src3)>,
   312:        Requires<[UseHVX]>;
   313: 
   314:   def: Pat<(!cast<Intrinsic>(IntID#"_128B") HvxVR:$src1, HvxVR:$src2,
   315:                                             u3_0ImmPred:$src3),
   316:            (MI                              HvxVR:$src1, HvxVR:$src2,
   317:                                             u3_0ImmPred:$src3)>,
   318:        Requires<[UseHVX]>;
   319: }
   320: 
   321: multiclass T_VVI_inv_pat <InstHexagon MI, Intrinsic IntID> {
   322:   def: Pat<(IntID HvxVR:$src1, HvxVR:$src2, u3_64_ImmPred:$src3),
   323:            (MI    HvxVR:$src1, HvxVR:$src2,
   324:                                     (SUB_64_VAL u3_64_ImmPred:$src3))>,
   325:        Requires<[UseHVX]>;
   326: 
   327:   def: Pat<(!cast<Intrinsic>(IntID#"_128B") HvxVR:$src1, HvxVR:$src2,
   328:                                             u3_128_ImmPred:$src3),
   329:            (MI                              HvxVR:$src1, HvxVR:$src2,
   330:                                           (SUB_128_VAL u3_128_ImmPred:$src3))>,
   331:        Requires<[UseHVX]>;
   332: }
   333: 
   334: multiclass T_VVR_pat <InstHexagon MI, Intrinsic IntID> {
   335:   def: Pat<(IntID HvxVR:$src1, HvxVR:$src2, IntRegs:$src3),
   336:            (MI    HvxVR:$src1, HvxVR:$src2, IntRegs:$src3)>,
   337:        Requires<[UseHVX]>;
   338: 
   339:   def: Pat<(!cast<Intrinsic>(IntID#"_128B") HvxVR:$src1, HvxVR:$src2,
   340:                                             IntRegs:$src3),
   341:            (MI                              HvxVR:$src1, HvxVR:$src2,
   342:                                             IntRegs:$src3)>,
   343:        Requires<[UseHVX]>;
   344: }
   345: 
   346: defm : T_VI_pat <V6_valignbi, int_hexagon_V6_vror>;
   347: defm : T_VI_inv_pat <V6_vlalignbi, int_hexagon_V6_vror>;
   348: 
   349: defm : T_VVI_pat <V6_valignbi, int_hexagon_V6_valignb>;
   350: defm : T_VVI_inv_pat <V6_vlalignbi, int_hexagon_V6_valignbi>;
```
- EN: It defines declarative TableGen records like T_VVI_pat, T_VVI_inv_pat, T_VVR_pat; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 T_VVI_pat, T_VVI_inv_pat, T_VVR_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 351-400 / 第 351-400 行

```tablegen
   351: defm : T_VVI_inv_pat <V6_vlalignbi, int_hexagon_V6_valignb>;
   352: defm : T_VVR_pat <V6_valignb, int_hexagon_V6_valignbi>;
   353: defm : T_VVI_pat <V6_vlalignbi, int_hexagon_V6_vlalignb>;
   354: defm : T_VVI_inv_pat <V6_valignbi, int_hexagon_V6_vlalignbi>;
   355: defm : T_VVI_inv_pat <V6_valignbi, int_hexagon_V6_vlalignb>;
   356: defm : T_VVR_pat <V6_vlalignb, int_hexagon_V6_vlalignbi>;
   357: 
   358: def: Pat<(int_hexagon_V6_vd0),
   359:          (V6_vd0)>, Requires<[UseHVXV60, UseHVX64B]>;
   360: def: Pat<(int_hexagon_V6_vd0_128B ),
   361:          (V6_vd0)>, Requires<[UseHVXV60, UseHVX128B]>;
   362: 
   363: def: Pat<(int_hexagon_V6_vdd0),
   364:          (V6_vdd0)>, Requires<[UseHVXV65, UseHVX64B]>;
   365: def: Pat<(int_hexagon_V6_vdd0_128B),
   366:          (V6_vdd0)>, Requires<[UseHVXV65, UseHVX128B]>;
   367: 
   368: 
   369: multiclass T_VP_pat<InstHexagon MI, Intrinsic IntID> {
   370:   def: Pat<(IntID HvxVR:$Vu, DoubleRegs:$Rt),
   371:            (MI    HvxVR:$Vu, DoubleRegs:$Rt)>;
   372:   def: Pat<(!cast<Intrinsic>(IntID#"_128B")
   373:                   HvxVR:$Vu, DoubleRegs:$Rt),
   374:            (MI    HvxVR:$Vu, DoubleRegs:$Rt)>;
   375: }
   376: 
   377: multiclass T_WVP_pat<InstHexagon MI, Intrinsic IntID> {
   378:   def: Pat<(IntID HvxWR:$Vx, HvxVR:$Vu, DoubleRegs:$Rt),
   379:            (MI    HvxWR:$Vx, HvxVR:$Vu, DoubleRegs:$Rt)>;
   380:   def: Pat<(!cast<Intrinsic>(IntID#"_128B")
   381:                   HvxWR:$Vx, HvxVR:$Vu, DoubleRegs:$Rt),
   382:            (MI    HvxWR:$Vx, HvxVR:$Vu, DoubleRegs:$Rt)>;
   383: }
   384: 
   385: // These are actually only in V65.
   386: let Predicates = [UseHVXV65, UseHVX] in {
   387:   defm: T_VP_pat<V6_vrmpyub_rtt,        int_hexagon_V6_vrmpyub_rtt>;
   388:   defm: T_VP_pat<V6_vrmpybub_rtt,       int_hexagon_V6_vrmpybub_rtt>;
   389: 
   390:   defm: T_WVP_pat<V6_vrmpyub_rtt_acc,   int_hexagon_V6_vrmpyub_rtt_acc>;
   391:   defm: T_WVP_pat<V6_vrmpybub_rtt_acc,  int_hexagon_V6_vrmpybub_rtt_acc>;
   392: }
   393: 
   394: 
   395: multiclass T_pRI_pat<InstHexagon MI, Intrinsic IntID> {
   396:   def: Pat<(IntID PredRegs:$P, IntRegs:$R, timm:$s),
   397:            (MI    PredRegs:$P, IntRegs:$R, imm:$s)>;
   398:   def: Pat<(!cast<Intrinsic>(IntID#"_128B")
   399:                   PredRegs:$P, IntRegs:$R, timm:$s),
   400:            (MI    PredRegs:$P, IntRegs:$R, imm:$s)>;
```
- EN: It defines declarative TableGen records like T_VP_pat, T_WVP_pat, T_pRI_pat; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 T_VP_pat, T_WVP_pat, T_pRI_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 401-450 / 第 401-450 行

```tablegen
   401: }
   402: 
   403: multiclass T_pRM_pat<InstHexagon MI, Intrinsic IntID> {
   404:   def: Pat<(IntID PredRegs:$P, IntRegs:$R, ModRegs:$M),
   405:            (MI    PredRegs:$P, IntRegs:$R, ModRegs:$M)>;
   406:   def: Pat<(!cast<Intrinsic>(IntID#"_128B")
   407:                   PredRegs:$P, IntRegs:$R, ModRegs:$M),
   408:            (MI    PredRegs:$P, IntRegs:$R, ModRegs:$M)>;
   409: }
   410: 
   411: let Predicates = [UseHVXV62, UseHVX] in {
   412:   defm: T_pRI_pat<V6_vL32b_pred_ai,       int_hexagon_V6_vL32b_pred_ai>;
   413:   defm: T_pRI_pat<V6_vL32b_npred_ai,      int_hexagon_V6_vL32b_npred_ai>;
   414:   defm: T_pRI_pat<V6_vL32b_pred_pi,       int_hexagon_V6_vL32b_pred_pi>;
   415:   defm: T_pRI_pat<V6_vL32b_npred_pi,      int_hexagon_V6_vL32b_npred_pi>;
   416:   defm: T_pRI_pat<V6_vL32b_nt_pred_ai,    int_hexagon_V6_vL32b_nt_pred_ai>;
   417:   defm: T_pRI_pat<V6_vL32b_nt_npred_ai,   int_hexagon_V6_vL32b_nt_npred_ai>;
   418:   defm: T_pRI_pat<V6_vL32b_nt_pred_pi,    int_hexagon_V6_vL32b_nt_pred_pi>;
   419:   defm: T_pRI_pat<V6_vL32b_nt_npred_pi,   int_hexagon_V6_vL32b_nt_npred_pi>;
   420: 
   421:   defm: T_pRM_pat<V6_vL32b_pred_ppu,      int_hexagon_V6_vL32b_pred_ppu>;
   422:   defm: T_pRM_pat<V6_vL32b_npred_ppu,     int_hexagon_V6_vL32b_npred_ppu>;
   423:   defm: T_pRM_pat<V6_vL32b_nt_pred_ppu,   int_hexagon_V6_vL32b_nt_pred_ppu>;
   424:   defm: T_pRM_pat<V6_vL32b_nt_npred_ppu,  int_hexagon_V6_vL32b_nt_npred_ppu>;
   425: }
   426: 
   427: multiclass T_pRIV_pat<InstHexagon MI, Intrinsic IntID> {
   428:   def: Pat<(IntID PredRegs:$P, IntRegs:$R, timm:$s, HvxVR:$V),
   429:            (MI    PredRegs:$P, IntRegs:$R, imm:$s, HvxVR:$V)>;
   430:   def: Pat<(!cast<Intrinsic>(IntID#"_128B")
   431:                   PredRegs:$P, IntRegs:$R, timm:$s, HvxVR:$V),
   432:            (MI    PredRegs:$P, IntRegs:$R, imm:$s, HvxVR:$V)>;
   433: }
   434: 
   435: multiclass T_pRMV_pat<InstHexagon MI, Intrinsic IntID> {
   436:   def: Pat<(IntID PredRegs:$P, IntRegs:$R, ModRegs:$M, HvxVR:$V),
   437:            (MI    PredRegs:$P, IntRegs:$R, ModRegs:$M, HvxVR:$V)>;
   438:   def: Pat<(!cast<Intrinsic>(IntID#"_128B")
   439:                   PredRegs:$P, IntRegs:$R, ModRegs:$M, HvxVR:$V),
   440:            (MI    PredRegs:$P, IntRegs:$R, ModRegs:$M, HvxVR:$V)>;
   441: }
   442: 
   443: let Predicates = [UseHVXV60, UseHVX] in {
   444:   defm: T_pRIV_pat<V6_vS32b_pred_ai,      int_hexagon_V6_vS32b_pred_ai>;
   445:   defm: T_pRIV_pat<V6_vS32b_npred_ai,     int_hexagon_V6_vS32b_npred_ai>;
   446:   defm: T_pRIV_pat<V6_vS32b_pred_pi,      int_hexagon_V6_vS32b_pred_pi>;
   447:   defm: T_pRIV_pat<V6_vS32b_npred_pi,     int_hexagon_V6_vS32b_npred_pi>;
   448:   defm: T_pRIV_pat<V6_vS32Ub_pred_ai,     int_hexagon_V6_vS32Ub_pred_ai>;
   449:   defm: T_pRIV_pat<V6_vS32Ub_npred_ai,    int_hexagon_V6_vS32Ub_npred_ai>;
   450:   defm: T_pRIV_pat<V6_vS32Ub_pred_pi,     int_hexagon_V6_vS32Ub_pred_pi>;
```
- EN: It defines declarative TableGen records like T_pRM_pat, T_pRIV_pat, T_pRMV_pat; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 T_pRM_pat, T_pRIV_pat, T_pRMV_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 451-465 / 第 451-465 行

```tablegen
   451:   defm: T_pRIV_pat<V6_vS32Ub_npred_pi,    int_hexagon_V6_vS32Ub_npred_pi>;
   452:   defm: T_pRIV_pat<V6_vS32b_nt_pred_ai,   int_hexagon_V6_vS32b_nt_pred_ai>;
   453:   defm: T_pRIV_pat<V6_vS32b_nt_npred_ai,  int_hexagon_V6_vS32b_nt_npred_ai>;
   454:   defm: T_pRIV_pat<V6_vS32b_nt_pred_pi,   int_hexagon_V6_vS32b_nt_pred_pi>;
   455:   defm: T_pRIV_pat<V6_vS32b_nt_npred_pi,  int_hexagon_V6_vS32b_nt_npred_pi>;
   456: 
   457:   defm: T_pRMV_pat<V6_vS32b_pred_ppu,     int_hexagon_V6_vS32b_pred_ppu>;
   458:   defm: T_pRMV_pat<V6_vS32b_npred_ppu,    int_hexagon_V6_vS32b_npred_ppu>;
   459:   defm: T_pRMV_pat<V6_vS32Ub_pred_ppu,    int_hexagon_V6_vS32Ub_pred_ppu>;
   460:   defm: T_pRMV_pat<V6_vS32Ub_npred_ppu,   int_hexagon_V6_vS32Ub_npred_ppu>;
   461:   defm: T_pRMV_pat<V6_vS32b_nt_pred_ppu,  int_hexagon_V6_vS32b_nt_pred_ppu>;
   462:   defm: T_pRMV_pat<V6_vS32b_nt_npred_ppu, int_hexagon_V6_vS32b_nt_npred_ppu>;
   463: }
   464: 
   465: include "HexagonDepMapAsm2Intrin.td"
```
- EN: It includes TableGen fragments such as HexagonDepMapAsm2Intrin.td so records can build on shared target definitions. Notable Hexagon symbols referenced here include HexagonDepMapAsm2Intrin, showing how the code connects to sibling backend components.
- CN: 这里包含了 HexagonDepMapAsm2Intrin.td 等 TableGen 片段，使记录可以复用共享目标定义。 这里引用的重要 Hexagon 符号包括 HexagonDepMapAsm2Intrin，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- intrinsic mapping / Intrinsic 映射
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Included TableGen files / 包含的 TableGen 文件: `HexagonDepMapAsm2Intrin.td`
- Hexagon symbols / Hexagon 符号: `HexagonIntrinsics, HexagonDepMapAsm2Intrin`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
