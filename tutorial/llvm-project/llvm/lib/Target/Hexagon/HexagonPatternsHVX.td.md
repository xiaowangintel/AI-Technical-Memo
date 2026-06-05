# HexagonPatternsHVX.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonPatternsHVX.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon HVX-specific instruction-selection patterns using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。 重点涉及指令语义与选择。 重点涉及 HVX/向量处理。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```tablegen
     1: //===- HexagonPatternsHVX.td - Selection Patterns for HVX --*- tablegen -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: def HQ8:    PatLeaf<(VecQ8   HvxQR:$R)>;
    10: def HQ16:   PatLeaf<(VecQ16  HvxQR:$R)>;
    11: def HQ32:   PatLeaf<(VecQ32  HvxQR:$R)>;
    12: 
    13: def HVI8:   PatLeaf<(VecI8   HvxVR:$R)>;
    14: def HVI16:  PatLeaf<(VecI16  HvxVR:$R)>;
    15: def HVI32:  PatLeaf<(VecI32  HvxVR:$R)>;
    16: def HVF16:  PatLeaf<(VecF16  HvxVR:$R)>;
    17: def HVF32:  PatLeaf<(VecF32  HvxVR:$R)>;
    18: def HVBF16: PatLeaf<(VecBF16 HvxVR:$R)>;
    19: 
    20: def HWI8:   PatLeaf<(VecPI8  HvxWR:$R)>;
    21: def HWI16:  PatLeaf<(VecPI16 HvxWR:$R)>;
    22: def HWI32:  PatLeaf<(VecPI32 HvxWR:$R)>;
    23: def HWF16:  PatLeaf<(VecPF16 HvxWR:$R)>;
    24: def HWF32:  PatLeaf<(VecPF32 HvxWR:$R)>;
    25: def HWBF16: PatLeaf<(VecPBF16 HvxWR:$R)>;
    26: 
    27: def SDTVecUnaryOp:
    28:   SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisVec<1>]>;
    29: 
    30: def SDTVecBinOp:
    31:   SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisVec<1>, SDTCisSameAs<1,2>]>;
    32: 
    33: def SDTHexagonVEXTRACTW: SDTypeProfile<1, 2,
    34:   [SDTCisVT<0, i32>, SDTCisVec<1>, SDTCisVT<2, i32>]>;
    35: def HexagonVEXTRACTW : SDNode<"HexagonISD::VEXTRACTW", SDTHexagonVEXTRACTW>;
    36: 
    37: def SDTHexagonVINSERTW0: SDTypeProfile<1, 2,
    38:   [SDTCisVec<0>, SDTCisSameAs<0, 1>, SDTCisVT<2, i32>]>;
    39: def HexagonVINSERTW0: SDNode<"HexagonISD::VINSERTW0", SDTHexagonVINSERTW0>;
    40: 
    41: def HwLen2: SDNodeXForm<imm, [{
    42:   const auto &ST = CurDAG->getSubtarget<HexagonSubtarget>();
    43:   return CurDAG->getTargetConstant(ST.getVectorLength()/2, SDLoc(N), MVT::i32);
    44: }]>;
    45: 
    46: def Q2V: OutPatFrag<(ops node:$Qs), (V6_vandqrt $Qs, (ToI32 -1))>;
    47: 
    48: def Combinev: OutPatFrag<(ops node:$Vs, node:$Vt),
    49:   (REG_SEQUENCE HvxWR, $Vs, vsub_hi, $Vt, vsub_lo)>;
    50: 
    51: def Combineq: OutPatFrag<(ops node:$Qs, node:$Qt),
    52:   (V6_vandvrt
    53:     (V6_vor
    54:       (V6_vror (V6_vpackeb (V6_vd0), (Q2V $Qs)),
    55:                (ToI32 (HwLen2 (i32 0)))),  // Half the vector length
    56:       (V6_vpackeb (V6_vd0), (Q2V $Qt))),
    57:     (ToI32 -1))>;
    58: 
    59: def LoVec: OutPatFrag<(ops node:$Vs), (EXTRACT_SUBREG $Vs, vsub_lo)>;
    60: def HiVec: OutPatFrag<(ops node:$Vs), (EXTRACT_SUBREG $Vs, vsub_hi)>;
    61: 
    62: def HexagonQCAT:       SDNode<"HexagonISD::QCAT",       SDTVecBinOp>;
    63: def HexagonQTRUE:      SDNode<"HexagonISD::QTRUE",      SDTVecLeaf>;
    64: def HexagonQFALSE:     SDNode<"HexagonISD::QFALSE",     SDTVecLeaf>;
    65: 
    66: def vzero:  PatFrags<(ops), [(splat_vector (i32 0)), (splat_vector (f32zero))]>;
    67: def qtrue:  PatFrag<(ops), (HexagonQTRUE)>;
    68: def qfalse: PatFrag<(ops), (HexagonQFALSE)>;
    69: def qcat:   PatFrag<(ops node:$Qs, node:$Qt),
    70:                     (HexagonQCAT node:$Qs, node:$Qt)>;
    71: 
    72: def qnot:     PatFrag<(ops node:$Qs), (xor node:$Qs, qtrue)>;
    73: 
    74: def VSxtb: OutPatFrag<(ops node:$Vs), (V6_vunpackb  $Vs)>;
    75: def VSxth: OutPatFrag<(ops node:$Vs), (V6_vunpackh  $Vs)>;
    76: def VZxtb: OutPatFrag<(ops node:$Vs), (V6_vunpackub $Vs)>;
    77: def VZxth: OutPatFrag<(ops node:$Vs), (V6_vunpackuh $Vs)>;
    78: 
    79: def VShuff: OutPatFrag<(ops node:$Vs, node:$S),
    80:                        (V6_vshuffvdd (HiVec $Vs), (LoVec $Vs), (A2_tfrsi $S))>;
    81: 
    82: def VDeal: OutPatFrag<(ops node:$Vs, node:$S),
    83:                       (V6_vdealvdd (HiVec $Vs), (LoVec $Vs), (A2_tfrsi $S))>;
    84: 
    85: class VSubi<InstHexagon VSub, InstHexagon VSplati>:
    86:   OutPatFrag<(ops node:$Imm, node:$Vs), (VSub (VSplati (i32 $Imm)), $Vs)>;
    87: 
    88: def VSubib: VSubi<V6_vsubb, PS_vsplatib>;
    89: def VSubih: VSubi<V6_vsubh, PS_vsplatih>;
    90: def VSubiw: VSubi<V6_vsubw, PS_vsplatiw>;
    91: 
    92: def VNegb: OutPatFrag<(ops node:$Vs), (VSubib 0, $Vs)>;
    93: def VNegh: OutPatFrag<(ops node:$Vs), (VSubih 0, $Vs)>;
    94: def VNegw: OutPatFrag<(ops node:$Vs), (VSubiw 0, $Vs)>;
    95: 
    96: class pf3<SDNode Op>: PatFrag<(ops node:$a, node:$b, node:$c),
    97:                               (Op node:$a, node:$b, node:$c)>;
    98: 
    99: def Mfshl: pf3<HexagonMFSHL>;
   100: def Mfshr: pf3<HexagonMFSHR>;
```
- EN: It declares types such as VSubi, pf3, which carry the state or API of this component. It defines declarative TableGen records like HQ8, HQ16, HQ32, HVI8, HVI16, ... (51 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSubtarget<HexagonSubtarget>, getTargetConstant, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 VSubi, pf3 等类型，用来承载该组件的状态或接口。 这里定义了 HQ8, HQ16, HQ32, HVI8, HVI16, ... (51 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSubtarget<HexagonSubtarget>, getTargetConstant 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-200 / 第 101-200 行

```tablegen
   101: 
   102: def IsVecOff : PatLeaf<(i32 imm), [{
   103:   int32_t V = N->getSExtValue();
   104:   int32_t VecSize = HRI->getSpillSize(Hexagon::HvxVRRegClass);
   105:   assert(isPowerOf2_32(VecSize));
   106:   if ((uint32_t(V) & (uint32_t(VecSize)-1)) != 0)
   107:     return false;
   108:   int32_t L = Log2_32(VecSize);
   109:   return isInt<4>(V >> L);
   110: }]>;
   111: 
   112: 
   113: def alignedload: PatFrag<(ops node:$a), (load $a), [{
   114:   return isAlignedMemNode(cast<MemSDNode>(N));
   115: }]>;
   116: 
   117: def unalignedload: PatFrag<(ops node:$a), (load $a), [{
   118:   return !isAlignedMemNode(cast<MemSDNode>(N));
   119: }]>;
   120: 
   121: def alignedstore: PatFrag<(ops node:$v, node:$a), (store $v, $a), [{
   122:   return isAlignedMemNode(cast<MemSDNode>(N));
   123: }]>;
   124: 
   125: def unalignedstore: PatFrag<(ops node:$v, node:$a), (store $v, $a), [{
   126:   return !isAlignedMemNode(cast<MemSDNode>(N));
   127: }]>;
   128: 
   129: 
   130: // HVX loads
   131: 
   132: multiclass HvxLdfi_pat<InstHexagon MI, PatFrag Load, ValueType ResType,
   133:                        PatFrag ImmPred> {
   134:   def: Pat<(ResType (Load (add (i32 AddrFI:$fi), ImmPred:$Off))),
   135:            (MI AddrFI:$fi, imm:$Off)>;
   136:   def: Pat<(ResType (Load (IsOrAdd (i32 AddrFI:$fi), ImmPred:$Off))),
   137:            (MI AddrFI:$fi, imm:$Off)>;
   138:   def: Pat<(ResType (Load AddrFI:$fi)), (ResType (MI AddrFI:$fi, 0))>;
   139: }
   140: 
   141: multiclass HvxLdgi_pat<InstHexagon MI, PatFrag Load, ValueType ResType,
   142:                      PatFrag ImmPred> {
   143:   def: Pat<(ResType (Load (add I32:$Rt, ImmPred:$Off))),
   144:            (MI I32:$Rt, imm:$Off)>;
   145:   def: Pat<(ResType (Load I32:$Rt)),
   146:            (MI I32:$Rt, 0)>;
   147: }
   148: 
   149: multiclass HvxLdc_pat<InstHexagon MI, PatFrag Load, ValueType ResType> {
   150:   // The HVX selection code for shuffles can generate vector constants.
   151:   // Calling "Select" on the resulting loads from CP fails without these
   152:   // patterns.
   153:   def: Pat<(ResType (Load (HexagonCP tconstpool:$Addr))),
   154:            (MI (ToI32 imm:$Addr), 0)>;
   155:   def: Pat<(ResType (Load (HexagonAtPcrel tconstpool:$Addr))),
   156:            (MI (C4_addipc imm:$Addr), 0)>;
   157: }
   158: 
   159: multiclass HvxLd_pat<InstHexagon MI, PatFrag Load, ValueType ResType,
   160:                      PatFrag ImmPred> {
   161:   defm: HvxLdfi_pat<MI, Load, ResType, ImmPred>;
   162:   defm: HvxLdgi_pat<MI, Load, ResType, ImmPred>;
   163:   defm: HvxLdc_pat <MI, Load, ResType>;
   164: }
   165: 
   166: // Aligned loads: everything, plus loads with valignaddr node.
   167: multiclass HvxLda_pat<InstHexagon MI, PatFrag Load, ValueType ResType,
   168:                       PatFrag ImmPred> {
   169:   let AddedComplexity = 50 in {
   170:     def: Pat<(ResType (Load (valignaddr I32:$Rt))),
   171:              (MI I32:$Rt, 0)>;
   172:     def: Pat<(ResType (Load (add (valignaddr I32:$Rt), ImmPred:$Off))),
   173:              (MI I32:$Rt, imm:$Off)>;
   174:   }
   175:   defm: HvxLd_pat<MI, Load, ResType, ImmPred>;
   176: }
   177: 
   178: let Predicates = [UseHVX] in {
   179:   // alignedload will match a non-temporal load as well, so try non-temporal
   180:   // first.
   181:   defm: HvxLda_pat<V6_vL32b_nt_ai, alignednontemporalload, VecI8,  IsVecOff>;
   182:   defm: HvxLda_pat<V6_vL32b_nt_ai, alignednontemporalload, VecI16, IsVecOff>;
   183:   defm: HvxLda_pat<V6_vL32b_nt_ai, alignednontemporalload, VecI32, IsVecOff>;
   184:   defm: HvxLda_pat<V6_vL32b_ai,               alignedload, VecI8,  IsVecOff>;
   185:   defm: HvxLda_pat<V6_vL32b_ai,               alignedload, VecI16, IsVecOff>;
   186:   defm: HvxLda_pat<V6_vL32b_ai,               alignedload, VecI32, IsVecOff>;
   187:   defm: HvxLd_pat<V6_vL32Ub_ai,             unalignedload, VecI8,  IsVecOff>;
   188:   defm: HvxLd_pat<V6_vL32Ub_ai,             unalignedload, VecI16, IsVecOff>;
   189:   defm: HvxLd_pat<V6_vL32Ub_ai,             unalignedload, VecI32, IsVecOff>;
   190: }
   191: 
   192: let Predicates = [UseHVXV68] in {
   193:   defm : HvxLda_pat<V6_vL32b_nt_ai, alignednontemporalload, VecBF16, IsVecOff>;
   194:   defm : HvxLda_pat<V6_vL32b_nt_ai, alignednontemporalload, VecF16, IsVecOff>;
   195:   defm : HvxLda_pat<V6_vL32b_nt_ai, alignednontemporalload, VecF32, IsVecOff>;
   196:   defm : HvxLda_pat<V6_vL32b_ai, alignedload, VecBF16, IsVecOff>;
   197:   defm : HvxLda_pat<V6_vL32b_ai, alignedload, VecF16, IsVecOff>;
   198:   defm : HvxLda_pat<V6_vL32b_ai, alignedload, VecF32, IsVecOff>;
   199:   defm : HvxLd_pat<V6_vL32Ub_ai, unalignedload, VecBF16, IsVecOff>;
   200:   defm : HvxLd_pat<V6_vL32Ub_ai, unalignedload, VecF16, IsVecOff>;
```
- EN: It defines declarative TableGen records like IsVecOff, alignedload, unalignedload, alignedstore, unalignedstore, ... (10 total); these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getSExtValue, getSpillSize, assert, Log2_32, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里定义了 IsVecOff, alignedload, unalignedload, alignedstore, unalignedstore, ... (10 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getSExtValue, getSpillSize, assert, Log2_32, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-300 / 第 201-300 行

```tablegen
   201:   defm : HvxLd_pat<V6_vL32Ub_ai, unalignedload, VecF32, IsVecOff>;
   202: }
   203: 
   204: // HVX stores
   205: 
   206: multiclass HvxStfi_pat<InstHexagon MI, PatFrag Store, PatFrag Value,
   207:                        PatFrag ImmPred> {
   208:   def: Pat<(Store Value:$Vs, (add (i32 AddrFI:$fi), ImmPred:$Off)),
   209:            (MI AddrFI:$fi, imm:$Off, Value:$Vs)>;
   210:   def: Pat<(Store Value:$Vs, (IsOrAdd (i32 AddrFI:$fi), ImmPred:$Off)),
   211:            (MI AddrFI:$fi, imm:$Off, Value:$Vs)>;
   212:   def: Pat<(Store Value:$Vs, AddrFI:$fi),
   213:            (MI AddrFI:$fi, 0, Value:$Vs)>;
   214: }
   215: 
   216: multiclass HvxStgi_pat<InstHexagon MI, PatFrag Store, PatFrag Value,
   217:                        PatFrag ImmPred> {
   218:   def: Pat<(Store Value:$Vs, (add I32:$Rt, ImmPred:$Off)),
   219:            (MI I32:$Rt, imm:$Off, Value:$Vs)>;
   220:   def: Pat<(Store Value:$Vs, (IsOrAdd I32:$Rt, ImmPred:$Off)),
   221:            (MI I32:$Rt, imm:$Off, Value:$Vs)>;
   222:   def: Pat<(Store Value:$Vs, I32:$Rt),
   223:            (MI I32:$Rt, 0, Value:$Vs)>;
   224: }
   225: 
   226: multiclass HvxSt_pat<InstHexagon MI, PatFrag Store, PatFrag Value,
   227:                      PatFrag ImmPred> {
   228:   defm: HvxStfi_pat<MI, Store, Value, ImmPred>;
   229:   defm: HvxStgi_pat<MI, Store, Value, ImmPred>;
   230: }
   231: 
   232: let Predicates = [UseHVX] in {
   233:   // alignedstore will match a non-temporal store as well, so try non-temporal
   234:   // first.
   235:   defm: HvxSt_pat<V6_vS32b_nt_ai, alignednontemporalstore,  HVI8, IsVecOff>;
   236:   defm: HvxSt_pat<V6_vS32b_nt_ai, alignednontemporalstore, HVI16, IsVecOff>;
   237:   defm: HvxSt_pat<V6_vS32b_nt_ai, alignednontemporalstore, HVI32, IsVecOff>;
   238:   defm: HvxSt_pat<V6_vS32b_ai,               alignedstore,  HVI8, IsVecOff>;
   239:   defm: HvxSt_pat<V6_vS32b_ai,               alignedstore, HVI16, IsVecOff>;
   240:   defm: HvxSt_pat<V6_vS32b_ai,               alignedstore, HVI32, IsVecOff>;
   241:   defm: HvxSt_pat<V6_vS32Ub_ai,            unalignedstore,  HVI8, IsVecOff>;
   242:   defm: HvxSt_pat<V6_vS32Ub_ai,            unalignedstore, HVI16, IsVecOff>;
   243:   defm: HvxSt_pat<V6_vS32Ub_ai,            unalignedstore, HVI32, IsVecOff>;
   244: }
   245: 
   246: let Predicates = [UseHVXV68] in {
   247:   defm: HvxSt_pat<V6_vS32b_nt_ai, alignednontemporalstore, HVBF16, IsVecOff>;
   248:   defm: HvxSt_pat<V6_vS32b_nt_ai, alignednontemporalstore, HVF16, IsVecOff>;
   249:   defm: HvxSt_pat<V6_vS32b_nt_ai, alignednontemporalstore, HVF32, IsVecOff>;
   250:   defm: HvxSt_pat<V6_vS32b_ai,               alignedstore, HVBF16, IsVecOff>;
   251:   defm: HvxSt_pat<V6_vS32b_ai,               alignedstore, HVF16, IsVecOff>;
   252:   defm: HvxSt_pat<V6_vS32b_ai,               alignedstore, HVF32, IsVecOff>;
   253:   defm: HvxSt_pat<V6_vS32Ub_ai,            unalignedstore, HVBF16, IsVecOff>;
   254:   defm: HvxSt_pat<V6_vS32Ub_ai,            unalignedstore, HVF16, IsVecOff>;
   255:   defm: HvxSt_pat<V6_vS32Ub_ai,            unalignedstore, HVF32, IsVecOff>;
   256: }
   257: 
   258: // Bitcasts between same-size vector types are no-ops, except for the
   259: // actual type change.
   260: let Predicates = [UseHVX] in {
   261:   defm: NopCast_pat<VecI8,   VecI16,  HvxVR>;
   262:   defm: NopCast_pat<VecI8,   VecI32,  HvxVR>;
   263:   defm: NopCast_pat<VecI16,  VecI32,  HvxVR>;
   264: 
   265:   defm: NopCast_pat<VecPI8,  VecPI16, HvxWR>;
   266:   defm: NopCast_pat<VecPI8,  VecPI32, HvxWR>;
   267:   defm: NopCast_pat<VecPI16, VecPI32, HvxWR>;
   268: }
   269: 
   270: let Predicates = [UseHVX, UseHVXFloatingPoint] in {
   271:   defm: NopCast_pat<VecI8,   VecF16,  HvxVR>;
   272:   defm: NopCast_pat<VecI8,   VecBF16, HvxVR>;
   273:   defm: NopCast_pat<VecI8,   VecF32,  HvxVR>;
   274:   defm: NopCast_pat<VecI16,  VecF16,  HvxVR>;
   275:   defm: NopCast_pat<VecI16,  VecBF16, HvxVR>;
   276:   defm: NopCast_pat<VecI16,  VecF32,  HvxVR>;
   277:   defm: NopCast_pat<VecI32,  VecF16,  HvxVR>;
   278:   defm: NopCast_pat<VecI32,  VecBF16, HvxVR>;
   279:   defm: NopCast_pat<VecI32,  VecF32,  HvxVR>;
   280:   defm: NopCast_pat<VecF16,  VecF32,  HvxVR>;
   281: 
   282:   defm: NopCast_pat<VecPI8,  VecPF16, HvxWR>;
   283:   defm: NopCast_pat<VecPI8,  VecPBF16, HvxWR>;
   284:   defm: NopCast_pat<VecPI8,  VecPF32, HvxWR>;
   285:   defm: NopCast_pat<VecPI16, VecPF16, HvxWR>;
   286:   defm: NopCast_pat<VecPI16, VecPBF16, HvxWR>;
   287:   defm: NopCast_pat<VecPI16, VecPF32, HvxWR>;
   288:   defm: NopCast_pat<VecPI32, VecPF16, HvxWR>;
   289:   defm: NopCast_pat<VecPI32, VecPBF16, HvxWR>;
   290:   defm: NopCast_pat<VecPI32, VecPF32, HvxWR>;
   291:   defm: NopCast_pat<VecPF16, VecPF32, HvxWR>;
   292: }
   293: 
   294: let Predicates = [UseHVX] in {
   295:   let AddedComplexity = 100 in {
   296:     // These should be preferred over a vsplat of 0.
   297:     def: Pat<(VecI8   vzero), (V6_vd0)>;
   298:     def: Pat<(VecI16  vzero), (V6_vd0)>;
   299:     def: Pat<(VecI32  vzero), (V6_vd0)>;
   300:     def: Pat<(VecPI8  vzero), (PS_vdd0)>;
```
- EN: It defines declarative TableGen records like HvxStfi_pat, HvxStgi_pat, HvxSt_pat; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 HvxStfi_pat, HvxStgi_pat, HvxSt_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-400 / 第 301-400 行

```tablegen
   301:     def: Pat<(VecPI16 vzero), (PS_vdd0)>;
   302:     def: Pat<(VecPI32 vzero), (PS_vdd0)>;
   303:     def: Pat<(VecPF32 vzero), (PS_vdd0)>;
   304: 
   305:     def: Pat<(concat_vectors  (VecI8 vzero),  (VecI8 vzero)), (PS_vdd0)>;
   306:     def: Pat<(concat_vectors (VecI16 vzero), (VecI16 vzero)), (PS_vdd0)>;
   307:     def: Pat<(concat_vectors (VecI32 vzero), (VecI32 vzero)), (PS_vdd0)>;
   308:   }
   309: 
   310:   def: Pat<(VecPI8 (concat_vectors HVI8:$Vs, HVI8:$Vt)),
   311:            (Combinev HvxVR:$Vt, HvxVR:$Vs)>;
   312:   def: Pat<(VecPI16 (concat_vectors HVI16:$Vs, HVI16:$Vt)),
   313:            (Combinev HvxVR:$Vt, HvxVR:$Vs)>;
   314:   def: Pat<(VecPI32 (concat_vectors HVI32:$Vs, HVI32:$Vt)),
   315:            (Combinev HvxVR:$Vt, HvxVR:$Vs)>;
   316: 
   317:   def: Pat<(VecQ8  (qcat HQ16:$Qs, HQ16:$Qt)), (Combineq $Qt, $Qs)>;
   318:   def: Pat<(VecQ16 (qcat HQ32:$Qs, HQ32:$Qt)), (Combineq $Qt, $Qs)>;
   319: 
   320:   def: Pat<(HexagonVEXTRACTW HVI8:$Vu, I32:$Rs),
   321:            (V6_extractw HvxVR:$Vu, I32:$Rs)>;
   322:   def: Pat<(HexagonVEXTRACTW HVI16:$Vu, I32:$Rs),
   323:            (V6_extractw HvxVR:$Vu, I32:$Rs)>;
   324:   def: Pat<(HexagonVEXTRACTW HVI32:$Vu, I32:$Rs),
   325:            (V6_extractw HvxVR:$Vu, I32:$Rs)>;
   326: 
   327:   def: Pat<(HexagonVINSERTW0 HVI8:$Vu,  I32:$Rt),
   328:            (V6_vinsertwr HvxVR:$Vu, I32:$Rt)>;
   329:   def: Pat<(HexagonVINSERTW0 HVI16:$Vu, I32:$Rt),
   330:            (V6_vinsertwr HvxVR:$Vu, I32:$Rt)>;
   331:   def: Pat<(HexagonVINSERTW0 HVI32:$Vu, I32:$Rt),
   332:            (V6_vinsertwr HvxVR:$Vu, I32:$Rt)>;
   333: }
   334: 
   335: let Predicates = [UseHVX, UseHVXFloatingPoint] in {
   336:   let AddedComplexity = 100 in {
   337:     def: Pat<(VecF16  vzero), (V6_vd0)>;
   338:     def: Pat<(VecBF16  vzero), (V6_vd0)>;
   339:     def: Pat<(VecF32  vzero), (V6_vd0)>;
   340:     def: Pat<(VecPF16 vzero), (PS_vdd0)>;
   341:     def: Pat<(VecPBF16 vzero), (PS_vdd0)>;
   342:     def: Pat<(VecPF32 vzero), (PS_vdd0)>;
   343: 
   344:     def: Pat<(concat_vectors (VecF16 vzero), (VecF16 vzero)), (PS_vdd0)>;
   345:     def : Pat<(concat_vectors (VecBF16 vzero), (VecBF16 vzero)), (PS_vdd0)>;
   346:     def: Pat<(concat_vectors (VecF32 vzero), (VecF32 vzero)), (PS_vdd0)>;
   347:   }
   348: 
   349:   def: Pat<(VecPF16 (concat_vectors HVF16:$Vs, HVF16:$Vt)),
   350:            (Combinev HvxVR:$Vt, HvxVR:$Vs)>;
   351:   def: Pat<(VecPBF16 (concat_vectors HVBF16:$Vs, HVBF16:$Vt)),
   352:            (Combinev HvxVR:$Vt, HvxVR:$Vs)>;
   353:   def: Pat<(VecPF32 (concat_vectors HVF32:$Vs, HVF32:$Vt)),
   354:            (Combinev HvxVR:$Vt, HvxVR:$Vs)>;
   355: 
   356:   def: Pat<(HexagonVINSERTW0 HVF16:$Vu, I32:$Rt),
   357:            (V6_vinsertwr HvxVR:$Vu, I32:$Rt)>;
   358:   def: Pat<(HexagonVINSERTW0 HVBF16:$Vu, I32:$Rt),
   359:            (V6_vinsertwr HvxVR:$Vu, I32:$Rt)>;
   360:   def: Pat<(HexagonVINSERTW0 HVF32:$Vu, I32:$Rt),
   361:            (V6_vinsertwr HvxVR:$Vu, I32:$Rt)>;
   362: }
   363: 
   364: def Rep: OutPatFrag<(ops node:$N), (Combinev $N, $N)>;
   365: 
   366: let Predicates = [UseHVX] in {
   367:   let AddedComplexity = 10 in {
   368:     def: Pat<(VecI8   (splat_vector u8_0ImmPred:$V)),  (PS_vsplatib imm:$V)>;
   369:     def: Pat<(VecI16  (splat_vector u16_0ImmPred:$V)), (PS_vsplatih imm:$V)>;
   370:     def: Pat<(VecI32  (splat_vector anyimm:$V)),       (PS_vsplatiw imm:$V)>;
   371:     def: Pat<(VecPI8  (splat_vector u8_0ImmPred:$V)),  (Rep (PS_vsplatib imm:$V))>;
   372:     def: Pat<(VecPI16 (splat_vector u16_0ImmPred:$V)), (Rep (PS_vsplatih imm:$V))>;
   373:     def: Pat<(VecPI32 (splat_vector anyimm:$V)),       (Rep (PS_vsplatiw imm:$V))>;
   374:   }
   375:   def: Pat<(VecI8   (splat_vector I32:$Rs)), (PS_vsplatrb $Rs)>;
   376:   def: Pat<(VecI16  (splat_vector I32:$Rs)), (PS_vsplatrh $Rs)>;
   377:   def: Pat<(VecI32  (splat_vector I32:$Rs)), (PS_vsplatrw $Rs)>;
   378:   def: Pat<(VecPI8  (splat_vector I32:$Rs)), (Rep (PS_vsplatrb $Rs))>;
   379:   def: Pat<(VecPI16 (splat_vector I32:$Rs)), (Rep (PS_vsplatrh $Rs))>;
   380:   def: Pat<(VecPI32 (splat_vector I32:$Rs)), (Rep (PS_vsplatrw $Rs))>;
   381: }
   382: let Predicates = [UseHVXV68, UseHVXFloatingPoint] in {
   383:   let AddedComplexity = 30 in {
   384:     def: Pat<(VecF16  (splat_vector u16_0ImmPred:$V)), (PS_vsplatih imm:$V)>;
   385:     def: Pat<(VecBF16  (splat_vector u16_0ImmPred:$V)), (PS_vsplatih imm:$V)>;
   386:     def: Pat<(VecF32  (splat_vector anyint:$V)),       (PS_vsplatiw imm:$V)>;
   387:     def: Pat<(VecF32  (splat_vector f32ImmPred:$V)),   (PS_vsplatiw (ftoi $V))>;
   388:   }
   389:   let AddedComplexity = 20 in {
   390:     def: Pat<(VecF16  (splat_vector I32:$Rs)), (PS_vsplatrh $Rs)>;
   391:     def: Pat<(VecBF16  (splat_vector I32:$Rs)), (PS_vsplatrh $Rs)>;
   392:     def: Pat<(VecF32  (splat_vector I32:$Rs)), (PS_vsplatrw $Rs)>;
   393:     def: Pat<(VecF32  (splat_vector F32:$Rs)), (PS_vsplatrw $Rs)>;
   394:   }
   395: }
   396: 
   397: class Vneg1<ValueType VecTy>
   398:   : PatFrag<(ops), (VecTy (splat_vector (i32 -1)))>;
   399: 
   400: class Vnot<ValueType VecTy>
```
- EN: It declares types such as Vneg1, Vnot, which carry the state or API of this component. It defines declarative TableGen records like Rep, Vneg1, Vnot; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonVEXTRACTW, HexagonVINSERTW0, showing how the code connects to sibling backend components.
- CN: 这里声明了 Vneg1, Vnot 等类型，用来承载该组件的状态或接口。 这里定义了 Rep, Vneg1, Vnot 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonVEXTRACTW, HexagonVINSERTW0，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```tablegen
   401:   : PatFrag<(ops node:$Vs), (xor $Vs, Vneg1<VecTy>)>;
   402: 
   403: class ExtOp_pat<InstHexagon MI, PatFrag Op, PatFrag Ext, ValueType ResType,
   404:                  PatFrag VPred, int Shuff>
   405:   : Pat<(ResType (Op (Ext VPred:$Vs), (Ext VPred:$Vt))),
   406:         (VShuff (MI VPred:$Vs, VPred:$Vt), Shuff)>;
   407: 
   408: class VOpAcc_pat<InstHexagon MI, PatFrag Op, PatFrag Ext, ValueType ResType,
   409:                   PatFrag VxPred, PatFrag VsPred, int Shuff>
   410:   : Pat<(ResType (add VxPred:$Vx, (Op (Ext VsPred:$Vs), (Ext VsPred:$Vt)))),
   411:         (VShuff (MI (VDeal $Vx, Shuff), VsPred:$Vs, VsPred:$Vt), Shuff)>;
   412: 
   413: let Predicates = [UseHVX] in {
   414:   let AddedComplexity = 200 in {
   415:     def : ExtOp_pat<V6_vaddubh, Add, Zext, VecPI16, HVI8, -2>;
   416:     def : ExtOp_pat<V6_vadduhw, Add, Zext, VecPI32, HVI16, -4>;
   417:     def : ExtOp_pat<V6_vaddhw, Add, Sext, VecPI32, HVI16, -4>;
   418: 
   419:     def : ExtOp_pat<V6_vsububh, Sub, Zext, VecPI16, HVI8, -2>;
   420:     def : ExtOp_pat<V6_vsubuhw, Sub, Zext, VecPI32, HVI16, -4>;
   421:     def : ExtOp_pat<V6_vsubhw, Sub, Sext, VecPI32, HVI16, -4>;
   422: 
   423:     def : ExtOp_pat<V6_vmpybv, Mul, Sext, VecPI16, HVI8, -2>;
   424:     def : ExtOp_pat<V6_vmpyhv, Mul, Sext, VecPI32, HVI16, -4>;
   425:     def : ExtOp_pat<V6_vmpyubv, Mul, Zext, VecPI16, HVI8, -2>;
   426:     def : ExtOp_pat<V6_vmpyuhv, Mul, Zext, VecPI32, HVI16, -4>;
   427: 
   428:     // The first operand in V6_vmpybusv is unsigned.
   429:     def : Pat<(VecPI16 (mul (VecPI16 (zext HVI8:$Vs)),
   430:                             (VecPI16 (sext HVI8:$Vv)))),
   431:               (VShuff (V6_vmpybusv HVI8:$Vs, HVI8:$Vv), -2)>;
   432: 
   433:     // The second operand in V6_vmpyhus is unsigned.
   434:     def : Pat<(VecPI32 (mul (VecPI32 (sext HVI16:$Vs)),
   435:                             (VecPI32 (zext HVI16:$Vv)))),
   436:               (VShuff (V6_vmpyhus HVI16:$Vs, HVI16:$Vv), -4)>;
   437: 
   438:     def : VOpAcc_pat<V6_vaddubh_acc, Add, Zext, VecPI16, HWI16, HVI8, -2>;
   439:     def : VOpAcc_pat<V6_vadduhw_acc, Add, Zext, VecPI32, HWI32, HVI16, -4>;
   440:     def : VOpAcc_pat<V6_vaddhw_acc, Add, Sext, VecPI32, HWI32, HVI16, -4>;
   441: 
   442:     def : VOpAcc_pat<V6_vmpybv_acc, Mul, Sext, VecPI16, HWI16, HVI8, -2>;
   443:     def : VOpAcc_pat<V6_vmpyubv_acc, Mul, Zext, VecPI16, HWI16, HVI8, -2>;
   444:     def : VOpAcc_pat<V6_vmpyhv_acc, Mul, Sext, VecPI32, HWI32, HVI16, -4>;
   445:     def : VOpAcc_pat<V6_vmpyuhv_acc, Mul, Zext, VecPI32, HWI32, HVI16, -4>;
   446: 
   447:     // The second operand in V6_vmpybusv_acc is unsigned.
   448:     def : Pat<(VecPI16 (add HWI16:$Vx , (mul (VecPI16 (zext HVI8:$Vs)),
   449:                                              (VecPI16 (sext HVI8:$Vt))))),
   450:               (VShuff (V6_vmpybusv_acc (VDeal $Vx, -2),
   451:                                        HVI8:$Vs, HVI8:$Vt), -2)>;
   452: 
   453:     // The third operand in V6_vmpyhus_acc is unsigned.
   454:     def : Pat<(add HWI32:$Vx, (mul (VecPI32 (sext HVI16:$Vs)),
   455:                                    (VecPI32 (zext HVI16:$Vt)))),
   456:               (VShuff (V6_vmpyhus_acc (VDeal $Vx, -4),
   457:                                       HVI16:$Vs, HVI16:$Vt), -4)>;
   458:   }
   459: 
   460:   def : Pat<(VecI32 (partial_reduce_umla VecI32:$Acc, HVI8:$A, HVI8:$B)),
   461:             (V6_vrmpyubv_acc $Acc, $A, $B)>;
   462:   def : Pat<(VecI32 (partial_reduce_smla VecI32:$Acc, HVI8:$A, HVI8:$B)),
   463:             (V6_vrmpybv_acc $Acc, $A, $B)>;
   464:   def : Pat<(VecI32 (partial_reduce_sumla VecI32:$Acc, HVI8:$A, HVI8:$B)),
   465:             (V6_vrmpybusv_acc $Acc, $B, $A)>;
   466: }
   467: 
   468: let Predicates = [UseHVX] in {
   469:   let AddedComplexity = 200 in {
   470:     def: Pat<(Vnot<VecI8>   HVI8:$Vs), (V6_vnot HvxVR:$Vs)>;
   471:     def: Pat<(Vnot<VecI16> HVI16:$Vs), (V6_vnot HvxVR:$Vs)>;
   472:     def: Pat<(Vnot<VecI32> HVI32:$Vs), (V6_vnot HvxVR:$Vs)>;
   473:   }
   474: 
   475:   def: OpR_RR_pat<V6_vaddb,    Add,   VecI8,  HVI8>;
   476:   def: OpR_RR_pat<V6_vaddh,    Add,  VecI16, HVI16>;
   477:   def: OpR_RR_pat<V6_vaddw,    Add,  VecI32, HVI32>;
   478:   def: OpR_RR_pat<V6_vaddb_dv, Add,  VecPI8,  HWI8>;
   479:   def: OpR_RR_pat<V6_vaddh_dv, Add, VecPI16, HWI16>;
   480:   def: OpR_RR_pat<V6_vaddw_dv, Add, VecPI32, HWI32>;
   481:   def: OpR_RR_pat<V6_vsubb,    Sub,   VecI8,  HVI8>;
   482:   def: OpR_RR_pat<V6_vsubh,    Sub,  VecI16, HVI16>;
   483:   def: OpR_RR_pat<V6_vsubw,    Sub,  VecI32, HVI32>;
   484:   def: OpR_RR_pat<V6_vsubb_dv, Sub,  VecPI8,  HWI8>;
   485:   def: OpR_RR_pat<V6_vsubh_dv, Sub, VecPI16, HWI16>;
   486:   def: OpR_RR_pat<V6_vsubw_dv, Sub, VecPI32, HWI32>;
   487:   def: OpR_RR_pat<V6_vand,     And,   VecI8,  HVI8>;
   488:   def: OpR_RR_pat<V6_vand,     And,  VecI16, HVI16>;
   489:   def: OpR_RR_pat<V6_vand,     And,  VecI32, HVI32>;
   490:   def: OpR_RR_pat<V6_vor,       Or,   VecI8,  HVI8>;
   491:   def: OpR_RR_pat<V6_vor,       Or,  VecI16, HVI16>;
   492:   def: OpR_RR_pat<V6_vor,       Or,  VecI32, HVI32>;
   493:   def: OpR_RR_pat<V6_vxor,     Xor,   VecI8,  HVI8>;
   494:   def: OpR_RR_pat<V6_vxor,     Xor,  VecI16, HVI16>;
   495:   def: OpR_RR_pat<V6_vxor,     Xor,  VecI32, HVI32>;
   496: 
   497:   def: OpR_RR_pat<V6_vminb,   Smin,   VecI8,  HVI8>;
   498:   def: OpR_RR_pat<V6_vmaxb,   Smax,   VecI8,  HVI8>;
   499:   def: OpR_RR_pat<V6_vminub,  Umin,   VecI8,  HVI8>;
   500:   def: OpR_RR_pat<V6_vmaxub,  Umax,   VecI8,  HVI8>;
```
- EN: It declares types such as ExtOp_pat, VOpAcc_pat, which carry the state or API of this component. It defines declarative TableGen records like ExtOp_pat, VOpAcc_pat; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 ExtOp_pat, VOpAcc_pat 等类型，用来承载该组件的状态或接口。 这里定义了 ExtOp_pat, VOpAcc_pat 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 501-600 / 第 501-600 行

```tablegen
   501:   def: OpR_RR_pat<V6_vminh,   Smin,  VecI16, HVI16>;
   502:   def: OpR_RR_pat<V6_vmaxh,   Smax,  VecI16, HVI16>;
   503:   def: OpR_RR_pat<V6_vminuh,  Umin,  VecI16, HVI16>;
   504:   def: OpR_RR_pat<V6_vmaxuh,  Umax,  VecI16, HVI16>;
   505:   def: OpR_RR_pat<V6_vminw,   Smin,  VecI32, HVI32>;
   506:   def: OpR_RR_pat<V6_vmaxw,   Smax,  VecI32, HVI32>;
   507: 
   508:   def: Pat<(vselect HQ8:$Qu, HVI8:$Vs, HVI8:$Vt),
   509:            (V6_vmux HvxQR:$Qu, HvxVR:$Vs, HvxVR:$Vt)>;
   510:   def: Pat<(vselect HQ16:$Qu, HVI16:$Vs, HVI16:$Vt),
   511:            (V6_vmux HvxQR:$Qu, HvxVR:$Vs, HvxVR:$Vt)>;
   512:   def: Pat<(vselect HQ32:$Qu, HVI32:$Vs, HVI32:$Vt),
   513:            (V6_vmux HvxQR:$Qu, HvxVR:$Vs, HvxVR:$Vt)>;
   514: 
   515:   def: Pat<(vselect (qnot HQ8:$Qu), HVI8:$Vs, HVI8:$Vt),
   516:            (V6_vmux HvxQR:$Qu, HvxVR:$Vt, HvxVR:$Vs)>;
   517:   def: Pat<(vselect (qnot HQ16:$Qu), HVI16:$Vs, HVI16:$Vt),
   518:            (V6_vmux HvxQR:$Qu, HvxVR:$Vt, HvxVR:$Vs)>;
   519:   def: Pat<(vselect (qnot HQ32:$Qu), HVI32:$Vs, HVI32:$Vt),
   520:            (V6_vmux HvxQR:$Qu, HvxVR:$Vt, HvxVR:$Vs)>;
   521: }
   522: 
   523: let Predicates = [UseHVX] in {
   524:   def: OpR_RR_pat_sat<V6_vaddubsat,    uaddsat, VecI8,   HVI8>;
   525:   def: OpR_RR_pat_sat<V6_vadduhsat,    uaddsat, VecI16,  HVI16>;
   526:   def: OpR_RR_pat_sat<V6_vadduwsat,    uaddsat, VecI32,  HVI32>;
   527:   def: OpR_RR_pat_sat<V6_vaddbsat,     saddsat, VecI8,   HVI8>;
   528:   def: OpR_RR_pat_sat<V6_vaddhsat,     saddsat, VecI16,  HVI16>;
   529:   def: OpR_RR_pat_sat<V6_vaddwsat,     saddsat, VecI32,  HVI32>;
   530:   def: OpR_RR_pat_sat<V6_vaddubsat_dv, uaddsat, VecPI8,  HWI8>;
   531:   def: OpR_RR_pat_sat<V6_vadduhsat_dv, uaddsat, VecPI16, HWI16>;
   532:   def: OpR_RR_pat_sat<V6_vadduwsat_dv, uaddsat, VecPI32, HWI32>;
   533:   def: OpR_RR_pat_sat<V6_vaddbsat_dv,  saddsat, VecPI8,  HWI8>;
   534:   def: OpR_RR_pat_sat<V6_vaddhsat_dv,  saddsat, VecPI16, HWI16>;
   535:   def: OpR_RR_pat_sat<V6_vaddwsat_dv,  saddsat, VecPI32, HWI32>;
   536: }
   537: 
   538: let Predicates = [UseHVX] in {
   539:   def: OpR_RR_pat_sat<V6_vsububsat,    usubsat, VecI8,   HVI8>;
   540:   def: OpR_RR_pat_sat<V6_vsubuhsat,    usubsat, VecI16,  HVI16>;
   541:   def: OpR_RR_pat_sat<V6_vsubuwsat,    usubsat, VecI32,  HVI32>;
   542:   def: OpR_RR_pat_sat<V6_vsubbsat,     ssubsat, VecI8,   HVI8>;
   543:   def: OpR_RR_pat_sat<V6_vsubhsat,     ssubsat, VecI16,  HVI16>;
   544:   def: OpR_RR_pat_sat<V6_vsubwsat,     ssubsat, VecI32,  HVI32>;
   545:   def: OpR_RR_pat_sat<V6_vsububsat_dv, usubsat, VecPI8,  HWI8>;
   546:   def: OpR_RR_pat_sat<V6_vsubuhsat_dv, usubsat, VecPI16, HWI16>;
   547:   def: OpR_RR_pat_sat<V6_vsubuwsat_dv, usubsat, VecPI32, HWI32>;
   548:   def: OpR_RR_pat_sat<V6_vsubbsat_dv,  ssubsat, VecPI8,  HWI8>;
   549:   def: OpR_RR_pat_sat<V6_vsubhsat_dv,  ssubsat, VecPI16, HWI16>;
   550:   def: OpR_RR_pat_sat<V6_vsubwsat_dv,  ssubsat, VecPI32, HWI32>;
   551: }
   552: 
   553: // For now, we always deal with vector floating point in SF mode.
   554: class OpR_RR_pat_conv<InstHexagon MI, PatFrag Op, ValueType ResType,
   555:                       PatFrag RsPred, PatFrag RtPred = RsPred>
   556:   : Pat<(ResType (Op RsPred:$Rs, RtPred:$Rt)),
   557:         (V6_vconv_sf_qf32 (VecF32 (MI RsPred:$Rs, RtPred:$Rt)))>;
   558: 
   559: class OpR_RR_pat_conv_hf<InstHexagon MI, PatFrag Op, ValueType ResType,
   560:                       PatFrag RsPred, PatFrag RtPred = RsPred>
   561:   : Pat<(ResType (Op RsPred:$Rs, RtPred:$Rt)),
   562:         (V6_vconv_hf_qf16 (VecF16 (MI RsPred:$Rs, RtPred:$Rt)))>;
   563: 
   564: let Predicates = [UseHVXV68, UseHVXQFloat] in {
   565:   def: OpR_RR_pat_conv_hf<V6_vsub_hf,        pf2<fsub>,  VecF16, HVF16>;
   566:   def: OpR_RR_pat_conv_hf<V6_vadd_hf,        pf2<fadd>,  VecF16, HVF16>;
   567:   def: OpR_RR_pat_conv_hf<V6_vmpy_qf16_hf,   pf2<fmul>,  VecF16, HVF16>;
   568:   def: OpR_RR_pat_conv<V6_vsub_sf,        pf2<fsub>,  VecF32, HVF32>;
   569:   def: OpR_RR_pat_conv<V6_vadd_sf,        pf2<fadd>,  VecF32, HVF32>;
   570:   def: OpR_RR_pat_conv<V6_vmpy_qf32_sf,   pf2<fmul>,  VecF32, HVF32>;
   571: 
   572:   // For now we assume that the fp32 register is always coming in as IEEE float
   573:   // since the qfloat arithmetic instructions above always generate the
   574:   // accompanying conversions as part of their pattern
   575:   def: Pat<(VecF16 (pf1<fpround> HWF32:$Vuu)),
   576:            (V6_vdealh (V6_vconv_hf_qf32
   577:              (VecPF32 (Combinev (V6_vadd_sf (HiVec HvxWR:$Vuu), (V6_vd0)),
   578:                                 (V6_vadd_sf (LoVec HvxWR:$Vuu), (V6_vd0))
   579:              ))))>;
   580:   // fpextend for QFloat is handled manually in HexagonISelLoweringHVX.cpp.
   581: }
   582: 
   583: // HVX IEEE arithmetic Instructions
   584: let Predicates = [UseHVXV68, UseHVXIEEEFP] in {
   585:   def: Pat<(fadd HVF16:$Rs, HVF16:$Rt),
   586:            (V6_vadd_hf_hf HVF16:$Rs, HVF16:$Rt)>;
   587:   def: Pat<(fadd HVF32:$Rs, HVF32:$Rt),
   588:            (V6_vadd_sf_sf HVF32:$Rs, HVF32:$Rt)>;
   589:   def: Pat<(fsub HVF16:$Rs, HVF16:$Rt),
   590:            (V6_vsub_hf_hf HVF16:$Rs, HVF16:$Rt)>;
   591:   def: Pat<(fsub HVF32:$Rs, HVF32:$Rt),
   592:            (V6_vsub_sf_sf HVF32:$Rs, HVF32:$Rt)>;
   593:   def: Pat<(fmul HVF16:$Rs, HVF16:$Rt),
   594:            (V6_vmpy_hf_hf HVF16:$Rs, HVF16:$Rt)>;
   595:   def: Pat<(fmul HVF32:$Rs, HVF32:$Rt),
   596:            (V6_vmpy_sf_sf HVF32:$Rs, HVF32:$Rt)>;
   597: 
   598:   def: Pat<(VecF16 (pf1<fpround> HWF32:$Vuu)),
   599:            (V6_vdealh (V6_vcvt_hf_sf (HiVec HvxWR:$Vuu), (LoVec HvxWR:$Vuu)))>;
   600:   def: Pat<(VecPF32 (pf1<fpextend> HVF16:$Vu)),
```
- EN: It declares types such as OpR_RR_pat_conv, OpR_RR_pat_conv_hf, which carry the state or API of this component. It defines declarative TableGen records like OpR_RR_pat_conv, OpR_RR_pat_conv_hf; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonISelLoweringHVX, showing how the code connects to sibling backend components.
- CN: 这里声明了 OpR_RR_pat_conv, OpR_RR_pat_conv_hf 等类型，用来承载该组件的状态或接口。 这里定义了 OpR_RR_pat_conv, OpR_RR_pat_conv_hf 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonISelLoweringHVX，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```tablegen
   601:            (V6_vcvt_sf_hf (V6_vshuffh HvxVR:$Vu))>;
   602: 
   603:   def: OpR_R_pat<V6_vcvt_h_hf,  Fptosi, VecI16, HVF16>;
   604:   def: OpR_R_pat<V6_vcvt_uh_hf, Fptoui, VecI16, HVF16>;
   605:   def: OpR_R_pat<V6_vcvt_hf_h,  Sitofp, VecF16, HVI16>;
   606:   def: OpR_R_pat<V6_vcvt_hf_uh, Uitofp, VecF16, HVI16>;
   607: 
   608:   def: Pat<(VecI8 (Fptosi HWF16:$Vu)),
   609:            (V6_vcvt_b_hf (HiVec $Vu), (LoVec $Vu))>;
   610:   def: Pat<(VecI8 (Fptoui HWF16:$Vu)),
   611:            (V6_vcvt_ub_hf (HiVec $Vu), (LoVec $Vu))>;
   612:   def: Pat<(VecPF16 (Sitofp HVI8:$Vu)), (V6_vcvt_hf_b HvxVR:$Vu)>;
   613:   def: Pat<(VecPF16 (Uitofp HVI8:$Vu)), (V6_vcvt_hf_ub HvxVR:$Vu)>;
   614: }
   615: 
   616: let Predicates = [UseHVXV81] in {
   617:   def : Pat<(VecBF16 (pf1<fpround> HWF32:$Vuu)),
   618:             (V6_vpackwuh_sat (V6_vmux
   619:                  (V6_veqsf (HiVec HvxWR:$Vuu), (HiVec HvxWR:$Vuu)),
   620:                  (V6_vlsrw (V6_vmux (V6_veqw (V6_vand (HiVec HvxWR:$Vuu),
   621:                                          (PS_vsplatiw (i32 0x1FFFF))),
   622:                                 (PS_vsplatiw (i32 0x08000))),
   623:                       (HiVec HvxWR:$Vuu),
   624:                       (V6_vaddw (HiVec HvxWR:$Vuu),
   625:                                 (V6_vand (HiVec HvxWR:$Vuu),
   626:                                     (PS_vsplatiw (i32 0x8000))))),
   627:                                   (A2_tfrsi 16)),
   628:                  (PS_vsplatih (i32 0x7fff))),
   629:                 (V6_vmux (V6_veqsf (LoVec HvxWR:$Vuu), (LoVec HvxWR:$Vuu)),
   630:                     (V6_vlsrw (V6_vmux (V6_veqw (V6_vand (LoVec HvxWR:$Vuu),
   631:                                             (PS_vsplatiw (i32 0x1FFFF))),
   632:                                    (PS_vsplatiw (i32 0x08000))),
   633:                          (LoVec HvxWR:$Vuu),
   634:                          (V6_vaddw (LoVec HvxWR:$Vuu),
   635:                                    (V6_vand (LoVec HvxWR:$Vuu),
   636:                                        (PS_vsplatiw (i32 0x8000))))),
   637:                         (A2_tfrsi 16)),
   638:                     (PS_vsplatih (i32 0x7fff))))>;
   639: }
   640: 
   641: let Predicates = [UseHVXV73, UseHVXQFloat] in {
   642:   def : Pat<(VecF32 (Sitofp HVI32:$Vu)), (V6_vconv_sf_w HvxVR:$Vu)>;
   643: }
   644: 
   645: let Predicates = [UseHVXV68, UseHVXFloatingPoint] in {
   646:   def: Pat<(vselect HQ16:$Qu, HVF16:$Vs, HVF16:$Vt),
   647:            (V6_vmux HvxQR:$Qu, HvxVR:$Vs, HvxVR:$Vt)>;
   648:   def: Pat<(vselect (qnot HQ16:$Qu), HVF16:$Vs, HVF16:$Vt),
   649:            (V6_vmux HvxQR:$Qu, HvxVR:$Vt, HvxVR:$Vs)>;
   650: 
   651:   def: Pat<(vselect HQ32:$Qu, HVF32:$Vs, HVF32:$Vt),
   652:            (V6_vmux HvxQR:$Qu, HvxVR:$Vs, HvxVR:$Vt)>;
   653:   def: Pat<(vselect (qnot HQ32:$Qu), HVF32:$Vs, HVF32:$Vt),
   654:            (V6_vmux HvxQR:$Qu, HvxVR:$Vt, HvxVR:$Vs)>;
   655: }
   656: 
   657: let Predicates = [UseHVXV81, UseHVXFloatingPoint] in {
   658:   def : Pat<(vselect HQ16:$Qu, HVBF16:$Vs, HVBF16:$Vt),
   659:             (V6_vmux HvxQR:$Qu, HvxVR:$Vs, HvxVR:$Vt)>;
   660:   def : Pat<(vselect (qnot HQ16:$Qu), HVBF16:$Vs, HVBF16:$Vt),
   661:             (V6_vmux HvxQR:$Qu, HvxVR:$Vt, HvxVR:$Vs)>;
   662: }
   663: 
   664: let Predicates = [UseHVXV68, UseHVX128B, UseHVXQFloat] in {
   665:   let AddedComplexity = 220 in {
   666:     defm: MinMax_pats<V6_vmin_hf, V6_vmax_hf, vselect,  setgt, VecQ16, HVF16>;
   667:     defm: MinMax_pats<V6_vmin_hf, V6_vmax_hf, vselect, setogt, VecQ16, HVF16>;
   668:     defm: MinMax_pats<V6_vmin_sf, V6_vmax_sf, vselect,  setgt, VecQ32, HVF32>;
   669:     defm: MinMax_pats<V6_vmin_sf, V6_vmax_sf, vselect, setogt, VecQ32, HVF32>;
   670:   }
   671:   def: OpR_RR_pat<V6_vmin_hf, pf2<fminimumnum>, VecF16, HVF16>;
   672:   def: OpR_RR_pat<V6_vmax_hf, pf2<fmaximumnum>, VecF16, HVF16>;
   673:   def: OpR_RR_pat<V6_vmin_sf, pf2<fminimumnum>, VecF32, HVF32>;
   674:   def: OpR_RR_pat<V6_vmax_sf, pf2<fmaximumnum>, VecF32, HVF32>;
   675: }
   676: 
   677: let Predicates = [UseHVXV68, UseHVX128B, UseHVXIEEEFP] in {
   678:   let AddedComplexity = 220 in {
   679:     defm: MinMax_pats<V6_vfmin_hf, V6_vfmax_hf, vselect,  setgt, VecQ16, HVF16>;
   680:     defm: MinMax_pats<V6_vfmin_hf, V6_vfmax_hf, vselect, setogt, VecQ16, HVF16>;
   681:     defm: MinMax_pats<V6_vfmin_sf, V6_vfmax_sf, vselect,  setgt, VecQ32, HVF32>;
   682:     defm: MinMax_pats<V6_vfmin_sf, V6_vfmax_sf, vselect, setogt, VecQ32, HVF32>;
   683:   }
   684:   def: OpR_RR_pat<V6_vfmin_hf, pf2<fminimumnum>, VecF16, HVF16>;
   685:   def: OpR_RR_pat<V6_vfmax_hf, pf2<fmaximumnum>, VecF16, HVF16>;
   686:   def: OpR_RR_pat<V6_vfmin_sf, pf2<fminimumnum>, VecF32, HVF32>;
   687:   def: OpR_RR_pat<V6_vfmax_sf, pf2<fmaximumnum>, VecF32, HVF32>;
   688: }
   689: 
   690: let Predicates = [UseHVX] in {
   691:   // For i8 vectors Vs = (a0, a1, ...), Vt = (b0, b1, ...),
   692:   // V6_vmpybv Vs, Vt produces a pair of i16 vectors Hi:Lo,
   693:   // where Lo = (a0*b0, a2*b2, ...), Hi = (a1*b1, a3*b3, ...).
   694:   def: Pat<(mul HVI8:$Vs, HVI8:$Vt),
   695:            (V6_vshuffeb (HiVec (V6_vmpybv HvxVR:$Vs, HvxVR:$Vt)),
   696:                         (LoVec (V6_vmpybv HvxVR:$Vs, HvxVR:$Vt)))>;
   697:   def: Pat<(mul HVI16:$Vs, HVI16:$Vt),
   698:            (V6_vmpyih HvxVR:$Vs, HvxVR:$Vt)>;
   699:   def: Pat<(mul HVI32:$Vs, HVI32:$Vt),
   700:            (V6_vmpyiewuh_acc (V6_vmpyieoh HvxVR:$Vs, HvxVR:$Vt),
```
- EN: This range continues the declarative TableGen description for Hexagon target data.
- CN: 这一段继续给出 Hexagon 目标数据的声明式 TableGen 描述。

### Lines 701-800 / 第 701-800 行

```tablegen
   701:                              HvxVR:$Vs, HvxVR:$Vt)>;
   702: }
   703: 
   704: let Predicates = [UseHVX] in {
   705:   def: Pat<(VecPI16 (sext HVI8:$Vs)),  (VSxtb $Vs)>;
   706:   def: Pat<(VecPI32 (sext HVI16:$Vs)), (VSxth $Vs)>;
   707:   def: Pat<(VecPI16 (zext HVI8:$Vs)),  (VZxtb $Vs)>;
   708:   def: Pat<(VecPI32 (zext HVI16:$Vs)), (VZxth $Vs)>;
   709: 
   710:   def: Pat<(VecI16 (sext_invec HVI8:$Vs)),  (LoVec (VSxtb $Vs))>;
   711:   def: Pat<(VecI32 (sext_invec HVI16:$Vs)), (LoVec (VSxth $Vs))>;
   712:   def: Pat<(VecI32 (sext_invec HVI8:$Vs)),
   713:            (LoVec (VSxth (LoVec (VSxtb $Vs))))>;
   714:   def: Pat<(VecPI16 (sext_invec HWI8:$Vss)),  (VSxtb (LoVec $Vss))>;
   715:   def: Pat<(VecPI32 (sext_invec HWI16:$Vss)), (VSxth (LoVec $Vss))>;
   716:   def: Pat<(VecPI32 (sext_invec HWI8:$Vss)),
   717:            (VSxth (LoVec (VSxtb (LoVec $Vss))))>;
   718: 
   719:   def: Pat<(VecI16 (zext_invec HVI8:$Vs)),  (LoVec (VZxtb $Vs))>;
   720:   def: Pat<(VecI32 (zext_invec HVI16:$Vs)), (LoVec (VZxth $Vs))>;
   721:   def: Pat<(VecI32 (zext_invec HVI8:$Vs)),
   722:            (LoVec (VZxth (LoVec (VZxtb $Vs))))>;
   723:   def: Pat<(VecPI16 (zext_invec HWI8:$Vss)),  (VZxtb (LoVec $Vss))>;
   724:   def: Pat<(VecPI32 (zext_invec HWI16:$Vss)), (VZxth (LoVec $Vss))>;
   725:   def: Pat<(VecPI32 (zext_invec HWI8:$Vss)),
   726:            (VZxth (LoVec (VZxtb (LoVec $Vss))))>;
   727: 
   728:   def: Pat<(VecI8 (trunc HWI16:$Vss)),
   729:            (V6_vpackeb (HiVec $Vss), (LoVec $Vss))>;
   730:   def: Pat<(VecI16 (trunc HWI32:$Vss)),
   731:            (V6_vpackeh (HiVec $Vss), (LoVec $Vss))>;
   732:   // Pattern for (v32i8 (trunc v32i32:$Vs)) after widening:
   733:   def: Pat<(VecI8 (trunc
   734:               (concat_vectors
   735:                 (VecI16 (trunc (concat_vectors HVI32:$Vs, undef))),
   736:                 undef))),
   737:            (V6_vdealb4w (IMPLICIT_DEF), HvxVR:$Vs)>;
   738: 
   739:   def: Pat<(VecQ8 (trunc HVI8:$Vs)),
   740:            (V6_vandvrt HvxVR:$Vs, (ToI32 0x01010101))>;
   741:   def: Pat<(VecQ16 (trunc HVI16:$Vs)),
   742:            (V6_vandvrt HvxVR:$Vs, (ToI32 0x01010101))>;
   743:   def: Pat<(VecQ32 (trunc HVI32:$Vs)),
   744:            (V6_vandvrt HvxVR:$Vs, (ToI32 0x01010101))>;
   745:   def: Pat<(VecQ8 (trunc HWI16:$Vss)),
   746:            (Combineq(VecQ16(V6_vandvrt (HiVec $Vss), (ToI32 0x01010101))),
   747:            (VecQ16 (V6_vandvrt (LoVec $Vss), (ToI32 0x01010101))))>;
   748:   def: Pat<(VecQ16 (trunc HWI32:$Vss)),
   749:            (Combineq(VecQ32(V6_vandvrt (HiVec $Vss), (ToI32 0x01010101))),
   750:            (VecQ32 (V6_vandvrt (LoVec $Vss), (ToI32 0x01010101))))>;
   751: }
   752: 
   753: let Predicates = [UseHVX] in {
   754:   // The "source" types are not legal, and there are no parameterized
   755:   // definitions for them, but they are length-specific.
   756:   let Predicates = [UseHVX,UseHVX64B] in {
   757:     def: Pat<(VecI16 (sext_inreg HVI16:$Vs, v32i8)),
   758:              (V6_vasrh (V6_vaslh HVI16:$Vs, (ToI32 8)), (ToI32 8))>;
   759:     def: Pat<(VecI32 (sext_inreg HVI32:$Vs, v16i8)),
   760:              (V6_vasrw (V6_vaslw HVI32:$Vs, (ToI32 24)), (ToI32 24))>;
   761:     def: Pat<(VecI32 (sext_inreg HVI32:$Vs, v16i16)),
   762:              (V6_vasrw (V6_vaslw HVI32:$Vs, (ToI32 16)), (ToI32 16))>;
   763:   }
   764:   let Predicates = [UseHVX,UseHVX128B] in {
   765:     def: Pat<(VecI16 (sext_inreg HVI16:$Vs, v64i8)),
   766:              (V6_vasrh (V6_vaslh HVI16:$Vs, (ToI32 8)), (ToI32 8))>;
   767:     def: Pat<(VecI32 (sext_inreg HVI32:$Vs, v32i8)),
   768:              (V6_vasrw (V6_vaslw HVI32:$Vs, (ToI32 24)), (ToI32 24))>;
   769:     def: Pat<(VecI32 (sext_inreg HVI32:$Vs, v32i16)),
   770:              (V6_vasrw (V6_vaslw HVI32:$Vs, (ToI32 16)), (ToI32 16))>;
   771:   }
   772: 
   773:   // Take a pair of vectors Vt:Vs and shift them towards LSB by (Rt & HwLen).
   774:   def: Pat<(VecI8 (valign HVI8:$Vt, HVI8:$Vs, I32:$Rt)),
   775:            (LoVec (V6_valignb HvxVR:$Vt, HvxVR:$Vs, I32:$Rt))>;
   776:   def: Pat<(VecI16 (valign HVI16:$Vt, HVI16:$Vs, I32:$Rt)),
   777:            (LoVec (V6_valignb HvxVR:$Vt, HvxVR:$Vs, I32:$Rt))>;
   778:   def: Pat<(VecI32 (valign HVI32:$Vt, HVI32:$Vs, I32:$Rt)),
   779:            (LoVec (V6_valignb HvxVR:$Vt, HvxVR:$Vs, I32:$Rt))>;
   780: 
   781:   def: Pat<(HexagonVASL HVI8:$Vs, I32:$Rt),
   782:            (V6_vshuffeb (V6_vaslh (HiVec (V6_vzb HvxVR:$Vs)), I32:$Rt),
   783:                         (V6_vaslh (LoVec (V6_vzb HvxVR:$Vs)), I32:$Rt))>;
   784:   def: Pat<(HexagonVASR HVI8:$Vs, I32:$Rt),
   785:            (V6_vshuffeb (V6_vasrh (HiVec (V6_vsb HvxVR:$Vs)), I32:$Rt),
   786:                         (V6_vasrh (LoVec (V6_vsb HvxVR:$Vs)), I32:$Rt))>;
   787:   def: Pat<(HexagonVLSR HVI8:$Vs, I32:$Rt),
   788:            (V6_vshuffeb (V6_vlsrh (HiVec (V6_vzb HvxVR:$Vs)), I32:$Rt),
   789:                         (V6_vlsrh (LoVec (V6_vzb HvxVR:$Vs)), I32:$Rt))>;
   790: 
   791:   def: Pat<(HexagonVASL HVI16:$Vs, I32:$Rt), (V6_vaslh HvxVR:$Vs, I32:$Rt)>;
   792:   def: Pat<(HexagonVASL HVI32:$Vs, I32:$Rt), (V6_vaslw HvxVR:$Vs, I32:$Rt)>;
   793:   def: Pat<(HexagonVASR HVI16:$Vs, I32:$Rt), (V6_vasrh HvxVR:$Vs, I32:$Rt)>;
   794:   def: Pat<(HexagonVASR HVI32:$Vs, I32:$Rt), (V6_vasrw HvxVR:$Vs, I32:$Rt)>;
   795:   def: Pat<(HexagonVLSR HVI16:$Vs, I32:$Rt), (V6_vlsrh HvxVR:$Vs, I32:$Rt)>;
   796:   def: Pat<(HexagonVLSR HVI32:$Vs, I32:$Rt), (V6_vlsrw HvxVR:$Vs, I32:$Rt)>;
   797: 
   798:   def: Pat<(add HVI32:$Vx, (HexagonVASL HVI32:$Vu, I32:$Rt)),
   799:            (V6_vaslw_acc HvxVR:$Vx, HvxVR:$Vu, I32:$Rt)>;
   800:   def: Pat<(add HVI32:$Vx, (HexagonVASR HVI32:$Vu, I32:$Rt)),
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVASL, HexagonVASR, HexagonVLSR, showing how the code connects to sibling backend components.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVASL, HexagonVASR, HexagonVLSR，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```tablegen
   801:            (V6_vasrw_acc HvxVR:$Vx, HvxVR:$Vu, I32:$Rt)>;
   802: 
   803:   def: Pat<(shl HVI8:$Vs, HVI8:$Vt),
   804:            (V6_vshuffeb (V6_vaslhv (HiVec (V6_vzb $Vs)), (HiVec (V6_vzb $Vt))),
   805:                         (V6_vaslhv (LoVec (V6_vzb $Vs)), (LoVec (V6_vzb $Vt))))>;
   806:   def: Pat<(sra HVI8:$Vs, HVI8:$Vt),
   807:            (V6_vshuffeb (V6_vasrhv (HiVec (V6_vsb $Vs)), (HiVec (V6_vzb $Vt))),
   808:                         (V6_vasrhv (LoVec (V6_vsb $Vs)), (LoVec (V6_vzb $Vt))))>;
   809:   def: Pat<(srl HVI8:$Vs, HVI8:$Vt),
   810:            (V6_vshuffeb (V6_vlsrhv (HiVec (V6_vzb $Vs)), (HiVec (V6_vzb $Vt))),
   811:                         (V6_vlsrhv (LoVec (V6_vzb $Vs)), (LoVec (V6_vzb $Vt))))>;
   812: 
   813:   def: Pat<(shl HVI16:$Vs, HVI16:$Vt), (V6_vaslhv HvxVR:$Vs, HvxVR:$Vt)>;
   814:   def: Pat<(shl HVI32:$Vs, HVI32:$Vt), (V6_vaslwv HvxVR:$Vs, HvxVR:$Vt)>;
   815:   def: Pat<(sra HVI16:$Vs, HVI16:$Vt), (V6_vasrhv HvxVR:$Vs, HvxVR:$Vt)>;
   816:   def: Pat<(sra HVI32:$Vs, HVI32:$Vt), (V6_vasrwv HvxVR:$Vs, HvxVR:$Vt)>;
   817:   def: Pat<(srl HVI16:$Vs, HVI16:$Vt), (V6_vlsrhv HvxVR:$Vs, HvxVR:$Vt)>;
   818:   def: Pat<(srl HVI32:$Vs, HVI32:$Vt), (V6_vlsrwv HvxVR:$Vs, HvxVR:$Vt)>;
   819: 
   820:   // Mfshl hi, lo, amt
   821:   def: Pat<(Mfshl HVI8:$Vu, HVI8:$Vv, HVI8:$Vs),
   822:            (V6_vshuffob (V6_vaslhv (HiVec (V6_vshufoeb $Vu, $Vv)),
   823:                                    (HiVec (V6_vzb $Vs))),
   824:                         (V6_vaslhv (LoVec (V6_vshufoeb $Vu, $Vv)),
   825:                                    (LoVec (V6_vzb $Vs))))>;
   826:   let Predicates = [UseHVX,UseHVXV60] in {
   827:     // V60 doesn't produce 0 on shifts by bitwidth, e.g. Vv.h << 16-0
   828:     def: Pat<(Mfshl HVI16:$Vu, HVI16:$Vv, HVI16:$Vs),
   829:              (V6_vmux (V6_veqh $Vs, (V6_vd0)),
   830:                       $Vu,
   831:                       (V6_vor (V6_vaslhv $Vu, $Vs),
   832:                               (V6_vlsrhv $Vv, (VSubih 16, $Vs))))>;
   833:     def: Pat<(Mfshl HVI32:$Vu, HVI32:$Vv, HVI32:$Vs),
   834:              (V6_vmux (V6_veqw (V6_vand $Vs, (PS_vsplatiw (i32 31))), (V6_vd0)),
   835:                       $Vu,
   836:                       (V6_vor (V6_vaslwv $Vu, $Vs),
   837:                               (V6_vlsrwv $Vv, (VSubiw 32, $Vs))))>;
   838:   }
   839:   let Predicates = [UseHVX,UseHVXV62], AddedComplexity = 10 in {
   840:     // Do it as (Vu << Vs) | (Vv >> (BW-Vs)).
   841:     // For Vs == 0 becomes Vu | (Vv >> -BW), since the shift amount is
   842:     // sign-extended. Then this becomes Vu | (Vv << BW) == Vu.
   843:     def: Pat<(Mfshl HVI16:$Vu, HVI16:$Vv, HVI16:$Vs),
   844:              (V6_vor (V6_vaslhv $Vu, $Vs),
   845:                      (V6_vlsrhv $Vv, (VSubih 16, $Vs)))>;
   846:     def: Pat<(Mfshl HVI32:$Vu, HVI32:$Vv, HVI32:$Vs),
   847:              (V6_vor (V6_vaslwv $Vu, $Vs),
   848:                      (V6_vlsrwv $Vv, (VSubiw 32, $Vs)))>;
   849:   }
   850:   let Predicates = [UseHVX,UseHVXV66], AddedComplexity = 20 in {
   851:     // Assume Vs > 0 (and within bit width)
   852:     // Vx[1]:Vx[0] = V6_vasr_into Vx[0], Vv, Vs
   853:     //   -->  (Vx[0]:Vx[0] & (ffffffff << -Vs)) | (Vv:00000000 << -Vs)
   854:     // i.e. Vx[1] = insert ((Vv << -Vs) -> Vx[0])
   855:     def: Pat<(Mfshl HVI32:$Vu, HVI32:$Vv, HVI32:$Vs),
   856:              (HiVec (V6_vasr_into (Combinev (VecI32 (IMPLICIT_DEF)),
   857:                                             (V6_vlsrwv $Vv, (VSubiw 32, $Vs))),
   858:                                   $Vu,
   859:                                   (V6_vsubw (V6_vd0), $Vs)))>;
   860:   }
   861: 
   862:   // Mfshr hi, lo, amt
   863:   def: Pat<(Mfshr HVI8:$Vu, HVI8:$Vv, HVI8:$Vs),
   864:            (V6_vshuffeb (V6_vlsrhv (HiVec (V6_vshufoeb $Vu, $Vv)),
   865:                                    (HiVec (V6_vzb $Vs))),
   866:                         (V6_vlsrhv (LoVec (V6_vshufoeb $Vu, $Vv)),
   867:                                    (LoVec (V6_vzb $Vs))))>;
   868:   let Predicates = [UseHVX,UseHVXV60] in {
   869:     def: Pat<(Mfshr HVI16:$Vu, HVI16:$Vv, HVI16:$Vs),
   870:              (V6_vmux (V6_veqh $Vs, (V6_vd0)),
   871:                       $Vv,
   872:                       (V6_vor (V6_vaslhv $Vu, (VSubih 16, $Vs)),
   873:                               (V6_vlsrhv $Vv, $Vs)))>;
   874:     def: Pat<(Mfshr HVI32:$Vu, HVI32:$Vv, HVI32:$Vs),
   875:              (V6_vmux (V6_veqw $Vs, (V6_vd0)),
   876:                       $Vv,
   877:                       (V6_vor (V6_vaslwv $Vu, (VSubiw 32, $Vs)),
   878:                               (V6_vlsrwv $Vv, $Vs)))>;
   879:   }
   880:   let Predicates = [UseHVX,UseHVXV62], AddedComplexity = 10 in {
   881:     // Do it as (Vu >> -(BW-Vs)) | (Vv >> Vs).
   882:     // For Vs == 0 becomes (Vu << BW) | Vs == 0 | Vv
   883:     def: Pat<(Mfshr HVI16:$Vu, HVI16:$Vv, HVI16:$Vs),
   884:              (V6_vor (V6_vlsrhv $Vu, (V6_vsubh $Vs, (PS_vsplatih (i32 16)))),
   885:                      (V6_vlsrhv $Vv, $Vs))>;
   886:     def: Pat<(Mfshr HVI32:$Vu, HVI32:$Vv, HVI32:$Vs),
   887:              (V6_vor (V6_vlsrwv $Vu, (V6_vsubw $Vs, (PS_vsplatiw (i32 32)))),
   888:                      (V6_vlsrwv $Vv, $Vs))>;
   889:   }
   890:   let Predicates = [UseHVX,UseHVXV66], AddedComplexity = 20 in {
   891:     // Assume Vs > 0 (and within bit width)
   892:     // Vx[1]:Vx[0] = V6_vasr_into Vx[0], Vv, Vs
   893:     //   -->  (Vx[0]:Vx[0] & (ffffffff >> Vs)) | (Vv:00000000 >> Vs)
   894:     // i.e. Vx[0] = insert ((Vv >> Vs) -> Vx[0])
   895:     def: Pat<(Mfshr HVI32:$Vu, HVI32:$Vv, HVI32:$Vs),
   896:              (LoVec (V6_vasr_into (Combinev (VecI32 (IMPLICIT_DEF)),
   897:                                             (V6_vlsrwv $Vv, $Vs)),
   898:                                   $Vu,
   899:                                   $Vs))>;
   900:   }
```
- EN: This range continues the declarative TableGen description for Hexagon target data.
- CN: 这一段继续给出 Hexagon 目标数据的声明式 TableGen 描述。

### Lines 901-1000 / 第 901-1000 行

```tablegen
   901: 
   902:   def: Pat<(VecI16 (bswap HVI16:$Vs)),
   903:            (V6_vdelta HvxVR:$Vs, (PS_vsplatib (i32 0x01)))>;
   904:   def: Pat<(VecI32 (bswap HVI32:$Vs)),
   905:            (V6_vdelta HvxVR:$Vs, (PS_vsplatib (i32 0x03)))>;
   906: 
   907:   def: Pat<(VecI8 (ctpop HVI8:$Vs)),
   908:            (V6_vshuffeb (V6_vpopcounth (HiVec (V6_vzb HvxVR:$Vs))),
   909:                         (V6_vpopcounth (LoVec (V6_vzb HvxVR:$Vs))))>;
   910:   def: Pat<(VecI16 (ctpop HVI16:$Vs)), (V6_vpopcounth HvxVR:$Vs)>;
   911:   def: Pat<(VecI32 (ctpop HVI32:$Vs)),
   912:            (V6_vaddw (LoVec (V6_vzh (V6_vpopcounth HvxVR:$Vs))),
   913:                      (HiVec (V6_vzh (V6_vpopcounth HvxVR:$Vs))))>;
   914: 
   915:   def: Pat<(VecI8 (ctlz HVI8:$Vs)),
   916:            (V6_vsubb (V6_vshuffeb (V6_vcl0h (HiVec (V6_vzb HvxVR:$Vs))),
   917:                                   (V6_vcl0h (LoVec (V6_vzb HvxVR:$Vs)))),
   918:                      (PS_vsplatib (i32 0x08)))>;
   919: 
   920:   def: Pat<(VecI16 (ctlz HVI16:$Vs)), (V6_vcl0h HvxVR:$Vs)>;
   921:   def: Pat<(VecI32 (ctlz HVI32:$Vs)), (V6_vcl0w HvxVR:$Vs)>;
   922: }
   923: 
   924: class HvxSel_pat<InstHexagon MI, PatFrag RegPred>
   925:   : Pat<(select I1:$Pu, RegPred:$Vs, RegPred:$Vt),
   926:         (MI I1:$Pu, RegPred:$Vs, RegPred:$Vt)>;
   927: 
   928: let Predicates = [UseHVX] in {
   929:   def: HvxSel_pat<PS_vselect, HVI8>;
   930:   def: HvxSel_pat<PS_vselect, HVI16>;
   931:   def: HvxSel_pat<PS_vselect, HVI32>;
   932:   def: HvxSel_pat<PS_wselect, HWI8>;
   933:   def: HvxSel_pat<PS_wselect, HWI16>;
   934:   def: HvxSel_pat<PS_wselect, HWI32>;
   935: }
   936: 
   937: def V2Q: OutPatFrag<(ops node:$Vs), (V6_vandvrt $Vs, (ToI32 -1))>;
   938: 
   939: let Predicates = [UseHVX] in {
   940:   def: Pat<(select I1:$Pu, VecQ8:$Qs, VecQ8:$Qt),
   941:            (V2Q (PS_vselect $Pu, (Q2V $Qs), (Q2V $Qt)))>;
   942:   def: Pat<(select I1:$Pu, VecQ16:$Qs, VecQ16:$Qt),
   943:            (V2Q (PS_vselect $Pu, (Q2V $Qs), (Q2V $Qt)))>;
   944:   def: Pat<(select I1:$Pu, VecQ32:$Qs, VecQ32:$Qt),
   945:            (V2Q (PS_vselect $Pu, (Q2V $Qs), (Q2V $Qt)))>;
   946: }
   947: 
   948: let Predicates = [UseHVX] in {
   949:   def: Pat<(VecQ8   (qtrue)), (PS_qtrue)>;
   950:   def: Pat<(VecQ16  (qtrue)), (PS_qtrue)>;
   951:   def: Pat<(VecQ32  (qtrue)), (PS_qtrue)>;
   952:   def: Pat<(VecQ8  (qfalse)), (PS_qfalse)>;
   953:   def: Pat<(VecQ16 (qfalse)), (PS_qfalse)>;
   954:   def: Pat<(VecQ32 (qfalse)), (PS_qfalse)>;
   955: 
   956:   def: Pat<(vnot  HQ8:$Qs), (V6_pred_not HvxQR:$Qs)>;
   957:   def: Pat<(vnot HQ16:$Qs), (V6_pred_not HvxQR:$Qs)>;
   958:   def: Pat<(vnot HQ32:$Qs), (V6_pred_not HvxQR:$Qs)>;
   959:   def: Pat<(qnot  HQ8:$Qs), (V6_pred_not HvxQR:$Qs)>;
   960:   def: Pat<(qnot HQ16:$Qs), (V6_pred_not HvxQR:$Qs)>;
   961:   def: Pat<(qnot HQ32:$Qs), (V6_pred_not HvxQR:$Qs)>;
   962: 
   963:   def: OpR_RR_pat<V6_pred_and,  And,  VecQ8,   HQ8>;
   964:   def: OpR_RR_pat<V6_pred_and,  And, VecQ16,  HQ16>;
   965:   def: OpR_RR_pat<V6_pred_and,  And, VecQ32,  HQ32>;
   966:   def: OpR_RR_pat<V6_pred_or,    Or,  VecQ8,   HQ8>;
   967:   def: OpR_RR_pat<V6_pred_or,    Or, VecQ16,  HQ16>;
   968:   def: OpR_RR_pat<V6_pred_or,    Or, VecQ32,  HQ32>;
   969:   def: OpR_RR_pat<V6_pred_xor,  Xor,  VecQ8,   HQ8>;
   970:   def: OpR_RR_pat<V6_pred_xor,  Xor, VecQ16,  HQ16>;
   971:   def: OpR_RR_pat<V6_pred_xor,  Xor, VecQ32,  HQ32>;
   972: 
   973:   def: OpR_RR_pat<V6_pred_and_n,  VNot2<And, qnot>,  VecQ8,   HQ8>;
   974:   def: OpR_RR_pat<V6_pred_and_n,  VNot2<And, qnot>, VecQ16,  HQ16>;
   975:   def: OpR_RR_pat<V6_pred_and_n,  VNot2<And, qnot>, VecQ32,  HQ32>;
   976:   def: OpR_RR_pat<V6_pred_or_n,    VNot2<Or, qnot>,  VecQ8,   HQ8>;
   977:   def: OpR_RR_pat<V6_pred_or_n,    VNot2<Or, qnot>, VecQ16,  HQ16>;
   978:   def: OpR_RR_pat<V6_pred_or_n,    VNot2<Or, qnot>, VecQ32,  HQ32>;
   979: 
   980:   def: OpR_RR_pat<V6_veqb,      seteq,  VecQ8,  HVI8>;
   981:   def: OpR_RR_pat<V6_veqh,      seteq, VecQ16, HVI16>;
   982:   def: OpR_RR_pat<V6_veqw,      seteq, VecQ32, HVI32>;
   983:   def: OpR_RR_pat<V6_vgtb,      setgt,  VecQ8,  HVI8>;
   984:   def: OpR_RR_pat<V6_vgth,      setgt, VecQ16, HVI16>;
   985:   def: OpR_RR_pat<V6_vgtw,      setgt, VecQ32, HVI32>;
   986:   def: OpR_RR_pat<V6_vgtub,    setugt,  VecQ8,  HVI8>;
   987:   def: OpR_RR_pat<V6_vgtuh,    setugt, VecQ16, HVI16>;
   988:   def: OpR_RR_pat<V6_vgtuw,    setugt, VecQ32, HVI32>;
   989: 
   990:   def: AccRRR_pat<V6_veqb_and,    And,  seteq,    HQ8,  HVI8,  HVI8>;
   991:   def: AccRRR_pat<V6_veqb_or,      Or,  seteq,    HQ8,  HVI8,  HVI8>;
   992:   def: AccRRR_pat<V6_veqb_xor,    Xor,  seteq,    HQ8,  HVI8,  HVI8>;
   993:   def: AccRRR_pat<V6_veqh_and,    And,  seteq,   HQ16, HVI16, HVI16>;
   994:   def: AccRRR_pat<V6_veqh_or,      Or,  seteq,   HQ16, HVI16, HVI16>;
   995:   def: AccRRR_pat<V6_veqh_xor,    Xor,  seteq,   HQ16, HVI16, HVI16>;
   996:   def: AccRRR_pat<V6_veqw_and,    And,  seteq,   HQ32, HVI32, HVI32>;
   997:   def: AccRRR_pat<V6_veqw_or,      Or,  seteq,   HQ32, HVI32, HVI32>;
   998:   def: AccRRR_pat<V6_veqw_xor,    Xor,  seteq,   HQ32, HVI32, HVI32>;
   999: 
  1000:   def: AccRRR_pat<V6_vgtb_and,    And,  setgt,    HQ8,  HVI8,  HVI8>;
```
- EN: It declares types such as HvxSel_pat, which carry the state or API of this component. It defines declarative TableGen records like HvxSel_pat, V2Q; these records are consumed by TableGen instead of executed directly.
- CN: 这里声明了 HvxSel_pat 等类型，用来承载该组件的状态或接口。 这里定义了 HvxSel_pat, V2Q 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 1001-1100 / 第 1001-1100 行

```tablegen
  1001:   def: AccRRR_pat<V6_vgtb_or,      Or,  setgt,    HQ8,  HVI8,  HVI8>;
  1002:   def: AccRRR_pat<V6_vgtb_xor,    Xor,  setgt,    HQ8,  HVI8,  HVI8>;
  1003:   def: AccRRR_pat<V6_vgth_and,    And,  setgt,   HQ16, HVI16, HVI16>;
  1004:   def: AccRRR_pat<V6_vgth_or,      Or,  setgt,   HQ16, HVI16, HVI16>;
  1005:   def: AccRRR_pat<V6_vgth_xor,    Xor,  setgt,   HQ16, HVI16, HVI16>;
  1006:   def: AccRRR_pat<V6_vgtw_and,    And,  setgt,   HQ32, HVI32, HVI32>;
  1007:   def: AccRRR_pat<V6_vgtw_or,      Or,  setgt,   HQ32, HVI32, HVI32>;
  1008:   def: AccRRR_pat<V6_vgtw_xor,    Xor,  setgt,   HQ32, HVI32, HVI32>;
  1009: 
  1010:   def: AccRRR_pat<V6_vgtub_and,   And, setugt,    HQ8,  HVI8,  HVI8>;
  1011:   def: AccRRR_pat<V6_vgtub_or,     Or, setugt,    HQ8,  HVI8,  HVI8>;
  1012:   def: AccRRR_pat<V6_vgtub_xor,   Xor, setugt,    HQ8,  HVI8,  HVI8>;
  1013:   def: AccRRR_pat<V6_vgtuh_and,   And, setugt,   HQ16, HVI16, HVI16>;
  1014:   def: AccRRR_pat<V6_vgtuh_or,     Or, setugt,   HQ16, HVI16, HVI16>;
  1015:   def: AccRRR_pat<V6_vgtuh_xor,   Xor, setugt,   HQ16, HVI16, HVI16>;
  1016:   def: AccRRR_pat<V6_vgtuw_and,   And, setugt,   HQ32, HVI32, HVI32>;
  1017:   def: AccRRR_pat<V6_vgtuw_or,     Or, setugt,   HQ32, HVI32, HVI32>;
  1018:   def: AccRRR_pat<V6_vgtuw_xor,   Xor, setugt,   HQ32, HVI32, HVI32>;
  1019: }
  1020: 
  1021: let Predicates = [UseHVXV68, UseHVXFloatingPoint] in {
  1022:   def: OpR_RR_pat<V6_veqh,              seteq,  VecQ16, HVF16>;
  1023:   def: OpR_RR_pat<V6_veqh,             setoeq,  VecQ16, HVF16>;
  1024:   def: OpR_RR_pat<V6_veqh,             setueq,  VecQ16, HVF16>;
  1025:   def: OpR_RR_pat<V6_vgthf,             setgt,  VecQ16, HVF16>;
  1026:   def: OpR_RR_pat<V6_vgthf,            setogt,  VecQ16, HVF16>;
  1027:   def: OpR_RR_pat<V6_vgthf,            setugt,  VecQ16, HVF16>;
  1028: 
  1029:   def: OpR_RR_pat<V6_veqw,              seteq,  VecQ32, HVF32>;
  1030:   def: OpR_RR_pat<V6_veqw,             setoeq,  VecQ32, HVF32>;
  1031:   def: OpR_RR_pat<V6_veqw,             setueq,  VecQ32, HVF32>;
  1032:   def: OpR_RR_pat<V6_vgtsf,             setgt,  VecQ32, HVF32>;
  1033:   def: OpR_RR_pat<V6_vgtsf,            setogt,  VecQ32, HVF32>;
  1034:   def: OpR_RR_pat<V6_vgtsf,            setugt,  VecQ32, HVF32>;
  1035: 
  1036:   def: AccRRR_pat<V6_veqh_and,    And,          seteq,  HQ16, HVF16, HVF16>;
  1037:   def: AccRRR_pat<V6_veqh_or,      Or,          seteq,  HQ16, HVF16, HVF16>;
  1038:   def: AccRRR_pat<V6_veqh_xor,    Xor,          seteq,  HQ16, HVF16, HVF16>;
  1039:   def: AccRRR_pat<V6_veqh_and,    And,         setoeq,  HQ16, HVF16, HVF16>;
  1040:   def: AccRRR_pat<V6_veqh_or,      Or,         setoeq,  HQ16, HVF16, HVF16>;
  1041:   def: AccRRR_pat<V6_veqh_xor,    Xor,         setoeq,  HQ16, HVF16, HVF16>;
  1042:   def: AccRRR_pat<V6_veqh_and,    And,         setueq,  HQ16, HVF16, HVF16>;
  1043:   def: AccRRR_pat<V6_veqh_or,      Or,         setueq,  HQ16, HVF16, HVF16>;
  1044:   def: AccRRR_pat<V6_veqh_xor,    Xor,         setueq,  HQ16, HVF16, HVF16>;
  1045:   def: AccRRR_pat<V6_vgthf_and,   And,          setgt,  HQ16, HVF16, HVF16>;
  1046:   def: AccRRR_pat<V6_vgthf_or,     Or,          setgt,  HQ16, HVF16, HVF16>;
  1047:   def: AccRRR_pat<V6_vgthf_xor,   Xor,          setgt,  HQ16, HVF16, HVF16>;
  1048:   def: AccRRR_pat<V6_vgthf_and,   And,         setogt,  HQ16, HVF16, HVF16>;
  1049:   def: AccRRR_pat<V6_vgthf_or,     Or,         setogt,  HQ16, HVF16, HVF16>;
  1050:   def: AccRRR_pat<V6_vgthf_xor,   Xor,         setogt,  HQ16, HVF16, HVF16>;
  1051:   def: AccRRR_pat<V6_vgthf_and,   And,         setugt,  HQ16, HVF16, HVF16>;
  1052:   def: AccRRR_pat<V6_vgthf_or,     Or,         setugt,  HQ16, HVF16, HVF16>;
  1053:   def: AccRRR_pat<V6_vgthf_xor,   Xor,         setugt,  HQ16, HVF16, HVF16>;
  1054: 
  1055:   def: AccRRR_pat<V6_veqw_and,    And,          seteq,  HQ32, HVF32, HVF32>;
  1056:   def: AccRRR_pat<V6_veqw_or,      Or,          seteq,  HQ32, HVF32, HVF32>;
  1057:   def: AccRRR_pat<V6_veqw_xor,    Xor,          seteq,  HQ32, HVF32, HVF32>;
  1058:   def: AccRRR_pat<V6_veqw_and,    And,         setoeq,  HQ32, HVF32, HVF32>;
  1059:   def: AccRRR_pat<V6_veqw_or,      Or,         setoeq,  HQ32, HVF32, HVF32>;
  1060:   def: AccRRR_pat<V6_veqw_xor,    Xor,         setoeq,  HQ32, HVF32, HVF32>;
  1061:   def: AccRRR_pat<V6_veqw_and,    And,         setueq,  HQ32, HVF32, HVF32>;
  1062:   def: AccRRR_pat<V6_veqw_or,      Or,         setueq,  HQ32, HVF32, HVF32>;
  1063:   def: AccRRR_pat<V6_veqw_xor,    Xor,         setueq,  HQ32, HVF32, HVF32>;
  1064:   def: AccRRR_pat<V6_vgtsf_and,   And,          setgt,  HQ32, HVF32, HVF32>;
  1065:   def: AccRRR_pat<V6_vgtsf_or,     Or,          setgt,  HQ32, HVF32, HVF32>;
  1066:   def: AccRRR_pat<V6_vgtsf_xor,   Xor,          setgt,  HQ32, HVF32, HVF32>;
  1067:   def: AccRRR_pat<V6_vgtsf_and,   And,         setogt,  HQ32, HVF32, HVF32>;
  1068:   def: AccRRR_pat<V6_vgtsf_or,     Or,         setogt,  HQ32, HVF32, HVF32>;
  1069:   def: AccRRR_pat<V6_vgtsf_xor,   Xor,         setogt,  HQ32, HVF32, HVF32>;
  1070:   def: AccRRR_pat<V6_vgtsf_and,   And,         setugt,  HQ32, HVF32, HVF32>;
  1071:   def: AccRRR_pat<V6_vgtsf_or,     Or,         setugt,  HQ32, HVF32, HVF32>;
  1072:   def: AccRRR_pat<V6_vgtsf_xor,   Xor,         setugt,  HQ32, HVF32, HVF32>;
  1073: 
  1074:   def: Pat<(VecQ16 (setone HVF16:$Vt, HVF16:$Vu)),
  1075:            (V6_pred_not (V6_veqh HvxVR:$Vt, HvxVR:$Vu))>;
  1076: 
  1077:   def: Pat<(VecQ32 (setone HVF32:$Vt, HVF32:$Vu)),
  1078:            (V6_pred_not (V6_veqw HvxVR:$Vt, HvxVR:$Vu))>;
  1079: }
  1080: 
  1081: // Multiply high for non-i32 types
  1082: def: Pat<(VecI8  (mulhs  HVI8:$Vu,  HVI8:$Vv)),
  1083:          (V6_vshuffob (HiVec (V6_vmpybv $Vu, $Vv)),
  1084:                       (LoVec (V6_vmpybv $Vu, $Vv)))>;
  1085: def: Pat<(VecI16 (mulhs HVI16:$Vu, HVI16:$Vv)),
  1086:          (V6_vshufoh (HiVec (V6_vmpyhv $Vu, $Vv)),
  1087:                      (LoVec (V6_vmpyhv $Vu, $Vv)))>;
  1088: def: Pat<(VecI8  (mulhu  HVI8:$Vu,  HVI8:$Vv)),
  1089:          (V6_vshuffob (HiVec (V6_vmpyubv $Vu, $Vv)),
  1090:                       (LoVec (V6_vmpyubv $Vu, $Vv)))>;
  1091: def: Pat<(VecI16 (mulhu HVI16:$Vu, HVI16:$Vv)),
  1092:          (V6_vshufoh (HiVec (V6_vmpyuhv $Vu, $Vv)),
  1093:                      (LoVec (V6_vmpyuhv $Vu, $Vv)))>;
  1094: let Predicates = [UseHVXV69], AddedComplexity = 20 in {
  1095:   def: Pat<(VecI16 (mulhu HVI16:$Vu, HVI16:$Vv)),
  1096:            (V6_vmpyuhvs $Vu, $Vv)>;
  1097: }
  1098: 
  1099: let Predicates = [UseHVXV60] in {
  1100:   // V60 doesn't have vabsb or byte shifts.
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1101-1147 / 第 1101-1147 行

```tablegen
  1101:   // Do the "mask = x >> width-1; abs = (x + mask) ^ mask" trick.
  1102:   // v31:30.h = vsxt(Inp.b)             ; generate masks in odd bytes in
  1103:   //                                    ; interleaved half-words
  1104:   // v29:28.b = vshuffoe(v31.b,v30.b)   ; collect odd/even bytes, masks = v29
  1105:   // v27.b    = vadd(Inp.b,v29.b)       ; x + masks
  1106:   // Abs      = vxor(v27,v29)           ;   ^ masks
  1107:   def: Pat<(VecI8 (abs HVI8:$Vs)),
  1108:     (V6_vxor HvxVR:$Vs,
  1109:       (V6_vaddb HvxVR:$Vs,
  1110:         (HiVec
  1111:           (V6_vshufoeb
  1112:             (HiVec (V6_vsb HvxVR:$Vs)),
  1113:             (LoVec (V6_vsb HvxVR:$Vs))))))>;
  1114: }
  1115: 
  1116: let Predicates = [UseHVXV62], AddedComplexity = 20 in {
  1117:   def: Pat<(VecI8 (abs HVI8:$Vs)), (V6_vabsb HvxVR:$Vs)>;
  1118: }
  1119: 
  1120: def: Pat<(VecI16 (abs HVI16:$Vs)), (V6_vabsh HvxVR:$Vs)>;
  1121: def: Pat<(VecI32 (abs HVI32:$Vs)), (V6_vabsw HvxVR:$Vs)>;
  1122: 
  1123: // If a node takes an MVT type as a parameter, the argument must be
  1124: // a name of a member of MVT.
  1125: multiclass Saturates<ValueType HvxTy_i8, ValueType HvxTy_i16> {
  1126:   def: Pat<(VecI8 (ssat HWI16:$Vss, HvxTy_i8)),
  1127:            (V6_vpackhb_sat (HiVec $Vss), (LoVec $Vss))>;
  1128:   def: Pat<(VecI8 (ssat (concat_vectors HWI32:$Vss, HWI32:$Vtt), HvxTy_i8)),
  1129:            (V6_vpackhb_sat (V6_vpackwh_sat (HiVec $Vtt), (LoVec $Vtt)),
  1130:                            (V6_vpackwh_sat (HiVec $Vss), (LoVec $Vss)))>;
  1131:   def: Pat<(VecI16 (ssat HWI32:$Vss, HvxTy_i16)),
  1132:            (V6_vpackwh_sat (HiVec $Vss), (LoVec $Vss))>;
  1133: 
  1134:   def: Pat<(VecI8 (usat HWI16:$Vss, HvxTy_i8)),
  1135:            (V6_vpackhub_sat (HiVec $Vss), (LoVec $Vss))>;
  1136:   def: Pat<(VecI8 (usat (concat_vectors HWI32:$Vss, HWI32:$Vtt), HvxTy_i8)),
  1137:            (V6_vpackhub_sat (V6_vpackwuh_sat (HiVec $Vtt), (LoVec $Vtt)),
  1138:                             (V6_vpackwuh_sat (HiVec $Vss), (LoVec $Vss)))>;
  1139:   def: Pat<(VecI16 (usat HWI32:$Vss, HvxTy_i16)),
  1140:            (V6_vpackwuh_sat (HiVec $Vss), (LoVec $Vss))>;
  1141: }
  1142: let Predicates = [UseHVX64B] in {
  1143:   defm: Saturates<v64i8, v32i16>;
  1144: }
  1145: let Predicates = [UseHVX128B] in {
  1146:   defm: Saturates<v128i8, v64i16>;
  1147: }
```
- EN: It defines declarative TableGen records like Saturates; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as vsxt, vshuffoe, vadd, vxor, translating Hexagon-specific policy into reusable code paths.
- CN: 这里定义了 Saturates 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 vsxt, vshuffoe, vadd, vxor 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- subtarget feature gating / 子目标特性控制
- pattern-driven instruction selection / 基于模式的指令选择
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonPatternsHVX, HexagonVEXTRACTW, HexagonISD, HexagonVINSERTW0, HexagonSubtarget, HexagonQCAT, HexagonQTRUE, HexagonQFALSE, HexagonMFSHL, HexagonMFSHR, ... (16 total)`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
