# LoongArchLSXInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchLSXInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 LoongArch 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //===- LoongArchLSXInstrInfo.td - LoongArch LSX instructions -*- tablegen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the SIMD extension instructions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: def SDT_LoongArchVreplve : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisVec<0>,
  14:                                          SDTCisInt<1>, SDTCisVec<1>,
  15:                                          SDTCisSameAs<0, 1>, SDTCisInt<2>]>;
  16: def SDT_LoongArchVecCond : SDTypeProfile<1, 1, [SDTCisInt<0>, SDTCisVec<1>]>;
  17: 
  18: def SDT_LoongArchVShuf : SDTypeProfile<1, 3, [SDTCisVec<0>,
  19:                                        SDTCisInt<1>, SDTCisVec<1>,
  20:                                        SDTCisSameAs<0, 2>,
  21:                                        SDTCisSameAs<2, 3>]>;
  22: def SDT_LoongArchV2R : SDTypeProfile<1, 2, [SDTCisVec<0>,
  23:                                      SDTCisSameAs<0, 1>, SDTCisSameAs<1, 2>]>;
  24: def SDT_LoongArchV1RUimm: SDTypeProfile<1, 2, [SDTCisVec<0>,
  25:                                         SDTCisSameAs<0,1>, SDTCisVT<2, GRLenVT>]>;
  26: def SDT_LoongArchV2RUimm
  27:     : SDTypeProfile<1, 3,
  28:                     [SDTCisVec<0>, SDTCisSameAs<0, 1>, SDTCisSameAs<1, 2>,
  29:                      SDTCisVT<3, GRLenVT>]>;
  30: def SDT_LoongArchVreplgr2vr : SDTypeProfile<1, 1, [SDTCisInt<0>, SDTCisVec<0>, SDTCisInt<1>]>;
  31: def SDT_LoongArchVFRECIPE : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisVec<0>, SDTCisSameAs<0, 1>]>;
  32: def SDT_LoongArchVFRSQRTE : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisVec<0>, SDTCisSameAs<0, 1>]>;
  33: def SDT_LoongArchVLDREPL : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisPtrTy<1>]>;
  34: def SDT_LoongArchVMSKCOND : SDTypeProfile<1, 1, [SDTCisInt<0>, SDTCisVec<1>]>;
  35: def SDT_LoongArchVFCVT_S_D : SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisFP<0>,
  36:                                                   SDTCisVec<1>, SDTCisFP<1>, SDTCisSameAs<1, 2>]>;
  37: def SDT_LoongArchVFCVTLH_D_S : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisFP<0>,
  38:                                                     SDTCisVec<1>, SDTCisFP<1>]>;
  39: 
  40: // Target nodes.
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SDT_LoongArchVreplve`, `SDT_LoongArchVecCond`, `SDT_LoongArchVShuf`, `SDT_LoongArchV2R`, `SDT_LoongArchV1RUimm:`, `SDT_LoongArchV2RUimm`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SDT_LoongArchVreplve`, `SDT_LoongArchVecCond`, `SDT_LoongArchVShuf`, `SDT_LoongArchV2R`, `SDT_LoongArchV1RUimm:`, `SDT_LoongArchV2RUimm` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41: 
  42: // Vector Shuffle
  43: def loongarch_vreplve : SDNode<"LoongArchISD::VREPLVE", SDT_LoongArchVreplve>;
  44: 
  45: // Vector comparisons
  46: def loongarch_vall_nonzero : SDNode<"LoongArchISD::VALL_NONZERO",
  47:                                     SDT_LoongArchVecCond>;
  48: def loongarch_vany_nonzero : SDNode<"LoongArchISD::VANY_NONZERO",
  49:                                     SDT_LoongArchVecCond>;
  50: def loongarch_vall_zero : SDNode<"LoongArchISD::VALL_ZERO",
  51:                                  SDT_LoongArchVecCond>;
  52: def loongarch_vany_zero : SDNode<"LoongArchISD::VANY_ZERO",
  53:                                  SDT_LoongArchVecCond>;
  54: 
  55: // Extended vector element extraction
  56: def loongarch_vpick_sext_elt : SDNode<"LoongArchISD::VPICK_SEXT_ELT",
  57:                                       SDTypeProfile<1, 3, [SDTCisPtrTy<2>]>>;
  58: def loongarch_vpick_zext_elt : SDNode<"LoongArchISD::VPICK_ZEXT_ELT",
  59:                                       SDTypeProfile<1, 3, [SDTCisPtrTy<2>]>>;
  60: 
  61: // Vector Shuffle
  62: def loongarch_vshuf: SDNode<"LoongArchISD::VSHUF", SDT_LoongArchVShuf>;
  63: def loongarch_vpickev: SDNode<"LoongArchISD::VPICKEV", SDT_LoongArchV2R>;
  64: def loongarch_vpickod: SDNode<"LoongArchISD::VPICKOD", SDT_LoongArchV2R>;
  65: def loongarch_vpackev: SDNode<"LoongArchISD::VPACKEV", SDT_LoongArchV2R>;
  66: def loongarch_vpackod: SDNode<"LoongArchISD::VPACKOD", SDT_LoongArchV2R>;
  67: def loongarch_vilvl: SDNode<"LoongArchISD::VILVL", SDT_LoongArchV2R>;
  68: def loongarch_vilvh: SDNode<"LoongArchISD::VILVH", SDT_LoongArchV2R>;
  69: def loongarch_vandn: SDNode<"LoongArchISD::VANDN", SDT_LoongArchV2R>;
  70: 
  71: def loongarch_vshuf4i: SDNode<"LoongArchISD::VSHUF4I", SDT_LoongArchV1RUimm>;
  72: def loongarch_vshuf4i_d : SDNode<"LoongArchISD::VSHUF4I_D", SDT_LoongArchV2RUimm>;
  73: def loongarch_vreplvei: SDNode<"LoongArchISD::VREPLVEI", SDT_LoongArchV1RUimm>;
  74: def loongarch_vreplgr2vr: SDNode<"LoongArchISD::VREPLGR2VR", SDT_LoongArchVreplgr2vr>;
  75: def loongarch_vpermi : SDNode<"LoongArchISD::VPERMI", SDT_LoongArchV2RUimm>;
  76: 
  77: def loongarch_vfrecipe: SDNode<"LoongArchISD::FRECIPE", SDT_LoongArchVFRECIPE>;
  78: def loongarch_vfrsqrte: SDNode<"LoongArchISD::FRSQRTE", SDT_LoongArchVFRSQRTE>;
  79: 
  80: // Vector logicial left / right shift by immediate
```
- **EN**: This block declares or refines TableGen records such as `loongarch_vreplve`, `loongarch_vall_nonzero`, `loongarch_vany_nonzero`, `loongarch_vall_zero`, `loongarch_vany_zero`, `loongarch_vpick_sext_elt`.
- **CN**: 该代码块声明或细化了 `loongarch_vreplve`, `loongarch_vall_nonzero`, `loongarch_vany_nonzero`, `loongarch_vall_zero`, `loongarch_vany_zero`, `loongarch_vpick_sext_elt` 等 TableGen 记录。

### Lines 81-120 / 第 81-120 行
```tablegen
  81: def loongarch_vslli : SDNode<"LoongArchISD::VSLLI", SDT_LoongArchV1RUimm>;
  82: def loongarch_vsrli : SDNode<"LoongArchISD::VSRLI", SDT_LoongArchV1RUimm>;
  83: 
  84: // Vector byte logicial left / right shift
  85: def loongarch_vbsll : SDNode<"LoongArchISD::VBSLL", SDT_LoongArchV1RUimm>;
  86: def loongarch_vbsrl : SDNode<"LoongArchISD::VBSRL", SDT_LoongArchV1RUimm>;
  87: 
  88: // Vector Horizontal Addition with Widening
  89: def loongarch_vhaddw : SDNode<"LoongArchISD::VHADDW", SDT_LoongArchV2R>;
  90: 
  91: // Scalar load broadcast to vector
  92: def loongarch_vldrepl
  93:     : SDNode<"LoongArchISD::VLDREPL",
  94:              SDT_LoongArchVLDREPL, [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
  95: 
  96: // Vector mask set by condition
  97: def loongarch_vmskltz: SDNode<"LoongArchISD::VMSKLTZ", SDT_LoongArchVMSKCOND>;
  98: def loongarch_vmskgez: SDNode<"LoongArchISD::VMSKGEZ", SDT_LoongArchVMSKCOND>;
  99: def loongarch_vmskeqz: SDNode<"LoongArchISD::VMSKEQZ", SDT_LoongArchVMSKCOND>;
 100: def loongarch_vmsknez: SDNode<"LoongArchISD::VMSKNEZ", SDT_LoongArchVMSKCOND>;
 101: 
 102: def loongarch_vfcvt_s_d: SDNode<"LoongArchISD::VFCVT", SDT_LoongArchVFCVT_S_D>;
 103: def loongarch_vfcvtl_d_s: SDNode<"LoongArchISD::VFCVTL", SDT_LoongArchVFCVTLH_D_S>;
 104: def loongarch_vfcvth_d_s: SDNode<"LoongArchISD::VFCVTH", SDT_LoongArchVFCVTLH_D_S>;
 105: 
 106: // Vector rounded shift
 107: def loongarch_vsrlr: SDNode<"LoongArchISD::VSRLR", SDT_LoongArchV2R>;
 108: def loongarch_vsrar: SDNode<"LoongArchISD::VSRAR", SDT_LoongArchV2R>;
 109: 
 110: def immZExt1 : ImmLeaf<GRLenVT, [{return isUInt<1>(Imm);}]>;
 111: def immZExt2 : ImmLeaf<GRLenVT, [{return isUInt<2>(Imm);}]>;
 112: def immZExt3 : ImmLeaf<GRLenVT, [{return isUInt<3>(Imm);}]>;
 113: def immZExt4 : ImmLeaf<GRLenVT, [{return isUInt<4>(Imm);}]>;
 114: def immZExt8 : ImmLeaf<GRLenVT, [{return isUInt<8>(Imm);}]>;
 115: 
 116: class VecCond<SDPatternOperator OpNode, ValueType TyNode,
 117:               RegisterClass RC = LSX128>
 118:     : Pseudo<(outs GPR:$rd), (ins RC:$vj),
 119:              [(set GPR:$rd, (OpNode (TyNode RC:$vj)))]> {
 120:   let hasSideEffects = 0;
```
- **EN**: This block declares or refines TableGen records such as `loongarch_vslli`, `loongarch_vsrli`, `loongarch_vbsll`, `loongarch_vbsrl`, `loongarch_vhaddw`, `loongarch_vldrepl`.
- **CN**: 该代码块声明或细化了 `loongarch_vslli`, `loongarch_vsrli`, `loongarch_vbsll`, `loongarch_vbsrl`, `loongarch_vhaddw`, `loongarch_vldrepl` 等 TableGen 记录。

### Lines 121-160 / 第 121-160 行
```tablegen
 121:   let mayLoad = 0;
 122:   let mayStore = 0;
 123:   let usesCustomInserter = 1;
 124: }
 125: 
 126: def vsplat_imm_eq_1 : PatFrags<(ops), [(build_vector)], [{
 127:   APInt Imm;
 128:   EVT EltTy = N->getValueType(0).getVectorElementType();
 129: 
 130:   if (N->getOpcode() == ISD::BITCAST)
 131:     N = N->getOperand(0).getNode();
 132: 
 133:   return selectVSplat(N, Imm, EltTy.getSizeInBits()) &&
 134:          Imm.getBitWidth() == EltTy.getSizeInBits() && Imm == 1;
 135: }]>;
 136: 
 137: def vsplati8_imm_eq_7 : PatFrags<(ops), [(build_vector)], [{
 138:   APInt Imm;
 139:   EVT EltTy = N->getValueType(0).getVectorElementType();
 140: 
 141:   if (N->getOpcode() == ISD::BITCAST)
 142:     N = N->getOperand(0).getNode();
 143: 
 144:   return selectVSplat(N, Imm, EltTy.getSizeInBits()) &&
 145:          Imm.getBitWidth() == EltTy.getSizeInBits() && Imm == 7;
 146: }]>;
 147: def vsplati16_imm_eq_15 : PatFrags<(ops), [(build_vector)], [{
 148:   APInt Imm;
 149:   EVT EltTy = N->getValueType(0).getVectorElementType();
 150: 
 151:   if (N->getOpcode() == ISD::BITCAST)
 152:     N = N->getOperand(0).getNode();
 153: 
 154:   return selectVSplat(N, Imm, EltTy.getSizeInBits()) &&
 155:          Imm.getBitWidth() == EltTy.getSizeInBits() && Imm == 15;
 156: }]>;
 157: def vsplati32_imm_eq_31 : PatFrags<(ops), [(build_vector)], [{
 158:   APInt Imm;
 159:   EVT EltTy = N->getValueType(0).getVectorElementType();
 160: 
```
- **EN**: This block declares or refines TableGen records such as `vsplat_imm_eq_1`, `vsplati8_imm_eq_7`, `vsplati16_imm_eq_15`, `vsplati32_imm_eq_31`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `vsplat_imm_eq_1`, `vsplati8_imm_eq_7`, `vsplati16_imm_eq_15`, `vsplati32_imm_eq_31` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 161-200 / 第 161-200 行
```tablegen
 161:   if (N->getOpcode() == ISD::BITCAST)
 162:     N = N->getOperand(0).getNode();
 163: 
 164:   return selectVSplat(N, Imm, EltTy.getSizeInBits()) &&
 165:          Imm.getBitWidth() == EltTy.getSizeInBits() && Imm == 31;
 166: }]>;
 167: def vsplati64_imm_eq_63 : PatFrags<(ops), [(build_vector)], [{
 168:   APInt Imm;
 169:   EVT EltTy = N->getValueType(0).getVectorElementType();
 170: 
 171:   if (N->getOpcode() == ISD::BITCAST)
 172:     N = N->getOperand(0).getNode();
 173: 
 174:   return selectVSplat(N, Imm, EltTy.getSizeInBits()) &&
 175:          Imm.getBitWidth() == EltTy.getSizeInBits() && Imm == 63;
 176: }]>;
 177: 
 178: def vsplatf32_fpimm_eq_1
 179:   : PatFrags<(ops), [(bitconvert (v4i32 (build_vector))),
 180:                      (bitconvert (v8i32 (build_vector)))], [{
 181:   APInt Imm;
 182:   EVT EltTy = N->getValueType(0).getVectorElementType();
 183:   N = N->getOperand(0).getNode();
 184: 
 185:   return selectVSplat(N, Imm, EltTy.getSizeInBits()) &&
 186:          Imm.getBitWidth() == 32 &&
 187:          Imm.getBitWidth() == EltTy.getSizeInBits() &&
 188:          Imm == APFloat(+1.0f).bitcastToAPInt();
 189: }]>;
 190: def vsplatf64_fpimm_eq_1
 191:   : PatFrags<(ops), [(bitconvert (v2i64 (build_vector))),
 192:                      (bitconvert (v4i64 (build_vector)))], [{
 193:   APInt Imm;
 194:   EVT EltTy = N->getValueType(0).getVectorElementType();
 195:   N = N->getOperand(0).getNode();
 196: 
 197:   return selectVSplat(N, Imm, EltTy.getSizeInBits()) &&
 198:          Imm.getBitWidth() == EltTy.getSizeInBits() &&
 199:          Imm == APFloat(+1.0).bitcastToAPInt();
 200: }]>;
```
- **EN**: This block declares or refines TableGen records such as `vsplati64_imm_eq_63`, `vsplatf32_fpimm_eq_1`, `vsplatf64_fpimm_eq_1`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `vsplati64_imm_eq_63`, `vsplatf32_fpimm_eq_1`, `vsplatf64_fpimm_eq_1` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 201-240 / 第 201-240 行
```tablegen
 201: 
 202: def vsplati8imm7   : PatFrag<(ops node:$reg),
 203:                              (and node:$reg, vsplati8_imm_eq_7)>;
 204: def vsplati16imm15 : PatFrag<(ops node:$reg),
 205:                              (and node:$reg, vsplati16_imm_eq_15)>;
 206: def vsplati32imm31 : PatFrag<(ops node:$reg),
 207:                              (and node:$reg, vsplati32_imm_eq_31)>;
 208: def vsplati64imm63 : PatFrag<(ops node:$reg),
 209:                              (and node:$reg, vsplati64_imm_eq_63)>;
 210: 
 211: foreach N = [3, 4, 5, 6, 8] in
 212:   def SplatPat_uimm#N : ComplexPattern<vAny, 1, "selectVSplatImm<"#N#", 0>",
 213:                                        [build_vector, bitconvert], [], 2>;
 214: 
 215: foreach N = [5] in
 216:   def SplatPat_simm#N : ComplexPattern<vAny, 1, "selectVSplatImm<"#N#", 0, true>",
 217:                                        [build_vector, bitconvert]>;
 218: 
 219: foreach N = [5] in
 220:   def SplatPat_nimm#N : ComplexPattern<vAny, 1, "selectVSplatImmNeg<"#N#">",
 221:                                        [build_vector, bitconvert]>;
 222: 
 223: def vsplat_i8_uimm8 : ComplexPattern<vAny, 1, "selectVSplatImm<8, 8>",
 224:                                      [build_vector, bitconvert]>;
 225: 
 226: def vsplat_uimm_inv_pow2 : ComplexPattern<vAny, 1, "selectVSplatUimmInvPow2",
 227:                                           [build_vector, bitconvert]>;
 228: 
 229: foreach N = [8, 16, 32] in
 230: def vsplat_i#N#_inv_pow2 : ComplexPattern<vAny, 1, "selectVSplatUimmInvPow2<"#N#">",
 231:                                           [build_vector, bitconvert]>;
 232: 
 233: def vsplat_uimm_pow2 : ComplexPattern<vAny, 1, "selectVSplatUimmPow2",
 234:                                       [build_vector, bitconvert]>;
 235: 
 236: foreach N = [8, 16, 32] in
 237: def vsplat_i#N#_pow2 : ComplexPattern<vAny, 1, "selectVSplatUimmPow2<"#N#">",
 238:                                       [build_vector, bitconvert]>;
 239: 
 240: def muladd : PatFrag<(ops node:$vd, node:$vj, node:$vk),
```
- **EN**: This block declares or refines TableGen records such as `vsplati8imm7`, `vsplati16imm15`, `vsplati32imm31`, `vsplati64imm63`, `SplatPat_uimm`, `SplatPat_simm`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `vsplati8imm7`, `vsplati16imm15`, `vsplati32imm31`, `vsplati64imm63`, `SplatPat_uimm`, `SplatPat_simm` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 241-280 / 第 241-280 行
```tablegen
 241:                      (add node:$vd, (mul node:$vj, node:$vk))>;
 242: 
 243: def mulsub : PatFrag<(ops node:$vd, node:$vj, node:$vk),
 244:                      (sub node:$vd, (mul node:$vj, node:$vk))>;
 245: 
 246: def lsxsplati8  : PatFrag<(ops node:$e0),
 247:                           (v16i8 (build_vector node:$e0, node:$e0,
 248:                                                node:$e0, node:$e0,
 249:                                                node:$e0, node:$e0,
 250:                                                node:$e0, node:$e0,
 251:                                                node:$e0, node:$e0,
 252:                                                node:$e0, node:$e0,
 253:                                                node:$e0, node:$e0,
 254:                                                node:$e0, node:$e0))>;
 255: def lsxsplati16 : PatFrag<(ops node:$e0),
 256:                           (v8i16 (build_vector node:$e0, node:$e0,
 257:                                                node:$e0, node:$e0,
 258:                                                node:$e0, node:$e0,
 259:                                                node:$e0, node:$e0))>;
 260: def lsxsplati32 : PatFrag<(ops node:$e0),
 261:                           (v4i32 (build_vector node:$e0, node:$e0,
 262:                                                node:$e0, node:$e0))>;
 263: def lsxsplati64 : PatFrag<(ops node:$e0),
 264:                           (v2i64 (build_vector node:$e0, node:$e0))>;
 265: def lsxsplatf32 : PatFrag<(ops node:$e0),
 266:                           (v4f32 (build_vector node:$e0, node:$e0,
 267:                                                node:$e0, node:$e0))>;
 268: def lsxsplatf64 : PatFrag<(ops node:$e0),
 269:                           (v2f64 (build_vector node:$e0, node:$e0))>;
 270: 
 271: def to_valid_timm : SDNodeXForm<timm, [{
 272:   auto CN = cast<ConstantSDNode>(N);
 273:   return CurDAG->getSignedTargetConstant(CN->getSExtValue(), SDLoc(N),
 274:                                          Subtarget->getGRLenVT());
 275: }]>;
 276: 
 277: // FP immediate of VLDI patterns.
 278: def f32imm_vldi : PatLeaf<(fpimm), [{
 279:   const auto &TLI =
 280:       *static_cast<const LoongArchTargetLowering*>(getTargetLowering());
```
- **EN**: This block declares or refines TableGen records such as `mulsub`, `lsxsplati8`, `lsxsplati16`, `lsxsplati32`, `lsxsplati64`, `lsxsplatf32`.
- **CN**: 该代码块声明或细化了 `mulsub`, `lsxsplati8`, `lsxsplati16`, `lsxsplati32`, `lsxsplati64`, `lsxsplatf32` 等 TableGen 记录。

### Lines 281-320 / 第 281-320 行
```tablegen
 281:   return TLI.isFPImmVLDILegal(N->getValueAPF(), MVT::f32);
 282: }]>;
 283: def f64imm_vldi : PatLeaf<(fpimm), [{
 284:   const auto &TLI =
 285:       *static_cast<const LoongArchTargetLowering*>(getTargetLowering());
 286:   return TLI.isFPImmVLDILegal(N->getValueAPF(), MVT::f64);
 287: }]>;
 288: 
 289: def to_f32imm_vldi : SDNodeXForm<fpimm, [{
 290:   uint64_t x = N->getValueAPF().bitcastToAPInt().getZExtValue();
 291:   x = (0b11011 << 8) | (((x >> 24) & 0xc0) ^ 0x40) | ((x >> 19) & 0x3f);
 292:   return CurDAG->getSignedTargetConstant(SignExtend32<13>(x), SDLoc(N),
 293:                                          MVT::i32);
 294: }]>;
 295: def to_f64imm_vldi : SDNodeXForm<fpimm, [{
 296:   uint64_t x = N->getValueAPF().bitcastToAPInt().getZExtValue();
 297:   x = (0b11100 << 8) | (((x >> 56) & 0xc0) ^ 0x40) | ((x >> 48) & 0x3f);
 298:   return CurDAG->getSignedTargetConstant(SignExtend32<13>(x), SDLoc(N),
 299:                                          MVT::i32);
 300: }]>;
 301: 
 302: //===----------------------------------------------------------------------===//
 303: // Instruction class templates
 304: //===----------------------------------------------------------------------===//
 305: 
 306: class LSX1RI13_VI<bits<32> op, Operand ImmOpnd = simm13>
 307:     : Fmt1RI13_VI<op, (outs LSX128:$vd), (ins ImmOpnd:$imm13), "$vd, $imm13">;
 308: 
 309: class LSX2R_VV<bits<32> op>
 310:     : Fmt2R_VV<op, (outs LSX128:$vd), (ins LSX128:$vj), "$vd, $vj">;
 311: 
 312: class LSX2R_VR<bits<32> op>
 313:     : Fmt2R_VR<op, (outs LSX128:$vd), (ins GPR:$rj), "$vd, $rj">;
 314: 
 315: class LSX2R_CV<bits<32> op>
 316:     : Fmt2R_CV<op, (outs CFR:$cd), (ins LSX128:$vj), "$cd, $vj">;
 317: 
 318: class LSX2RI1_VVI<bits<32> op, Operand ImmOpnd = uimm1>
 319:     : Fmt2RI1_VVI<op, (outs LSX128:$vd), (ins LSX128:$vj, ImmOpnd:$imm1),
 320:                   "$vd, $vj, $imm1">;
```
- **EN**: This block declares or refines TableGen records such as `f64imm_vldi`, `to_f32imm_vldi`, `to_f64imm_vldi`, `LSX1RI13_VI`, `LSX2R_VV`, `LSX2R_VR`.
- **CN**: 该代码块声明或细化了 `f64imm_vldi`, `to_f32imm_vldi`, `to_f64imm_vldi`, `LSX1RI13_VI`, `LSX2R_VV`, `LSX2R_VR` 等 TableGen 记录。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: 
 322: class LSX2RI1_RVI<bits<32> op, Operand ImmOpnd = uimm1>
 323:     : Fmt2RI1_RVI<op, (outs GPR:$rd), (ins LSX128:$vj, ImmOpnd:$imm1),
 324:                   "$rd, $vj, $imm1">;
 325: 
 326: class LSX2RI2_VVI<bits<32> op, Operand ImmOpnd = uimm2>
 327:     : Fmt2RI2_VVI<op, (outs LSX128:$vd), (ins LSX128:$vj, ImmOpnd:$imm2),
 328:                   "$vd, $vj, $imm2">;
 329: 
 330: class LSX2RI2_RVI<bits<32> op, Operand ImmOpnd = uimm2>
 331:     : Fmt2RI2_RVI<op, (outs GPR:$rd), (ins LSX128:$vj, ImmOpnd:$imm2),
 332:                   "$rd, $vj, $imm2">;
 333: 
 334: class LSX2RI3_VVI<bits<32> op, Operand ImmOpnd = uimm3>
 335:     : Fmt2RI3_VVI<op, (outs LSX128:$vd), (ins LSX128:$vj, ImmOpnd:$imm3),
 336:                   "$vd, $vj, $imm3">;
 337: 
 338: class LSX2RI3_RVI<bits<32> op, Operand ImmOpnd = uimm3>
 339:     : Fmt2RI3_RVI<op, (outs GPR:$rd), (ins LSX128:$vj, ImmOpnd:$imm3),
 340:                   "$rd, $vj, $imm3">;
 341: 
 342: class LSX2RI4_VVI<bits<32> op, Operand ImmOpnd = uimm4>
 343:     : Fmt2RI4_VVI<op, (outs LSX128:$vd), (ins LSX128:$vj, ImmOpnd:$imm4),
 344:                   "$vd, $vj, $imm4">;
 345: 
 346: class LSX2RI4_RVI<bits<32> op, Operand ImmOpnd = uimm4>
 347:     : Fmt2RI4_RVI<op, (outs GPR:$rd), (ins LSX128:$vj, ImmOpnd:$imm4),
 348:                   "$rd, $vj, $imm4">;
 349: 
 350: class LSX2RI5_VVI<bits<32> op, Operand ImmOpnd = uimm5>
 351:     : Fmt2RI5_VVI<op, (outs LSX128:$vd), (ins LSX128:$vj, ImmOpnd:$imm5),
 352:                   "$vd, $vj, $imm5">;
 353: 
 354: class LSX2RI6_VVI<bits<32> op, Operand ImmOpnd = uimm6>
 355:     : Fmt2RI6_VVI<op, (outs LSX128:$vd), (ins LSX128:$vj, ImmOpnd:$imm6),
 356:                   "$vd, $vj, $imm6">;
 357: 
 358: class LSX2RI8_VVI<bits<32> op, Operand ImmOpnd = uimm8>
 359:     : Fmt2RI8_VVI<op, (outs LSX128:$vd), (ins LSX128:$vj, ImmOpnd:$imm8),
 360:                   "$vd, $vj, $imm8">;
```
- **EN**: This block declares or refines TableGen records such as `LSX2RI1_RVI`, `LSX2RI2_VVI`, `LSX2RI2_RVI`, `LSX2RI3_VVI`, `LSX2RI3_RVI`, `LSX2RI4_VVI`.
- **CN**: 该代码块声明或细化了 `LSX2RI1_RVI`, `LSX2RI2_VVI`, `LSX2RI2_RVI`, `LSX2RI3_VVI`, `LSX2RI3_RVI`, `LSX2RI4_VVI` 等 TableGen 记录。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: 
 362: class LSX2RI8I1_VRII<bits<32> op, Operand ImmOpnd = simm8,
 363:                      Operand IdxOpnd = uimm1>
 364:     : Fmt2RI8I1_VRII<op, (outs),
 365:                      (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm1),
 366:                      "$vd, $rj, $imm8, $imm1">;
 367: class LSX2RI8I2_VRII<bits<32> op, Operand ImmOpnd = simm8,
 368:                      Operand IdxOpnd = uimm2>
 369:     : Fmt2RI8I2_VRII<op, (outs),
 370:                      (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm2),
 371:                      "$vd, $rj, $imm8, $imm2">;
 372: class LSX2RI8I3_VRII<bits<32> op, Operand ImmOpnd = simm8,
 373:                      Operand IdxOpnd = uimm3>
 374:     : Fmt2RI8I3_VRII<op, (outs),
 375:                      (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm3),
 376:                      "$vd, $rj, $imm8, $imm3">;
 377: class LSX2RI8I4_VRII<bits<32> op, Operand ImmOpnd = simm8,
 378:                      Operand IdxOpnd = uimm4>
 379:     : Fmt2RI8I4_VRII<op, (outs),
 380:                      (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm4),
 381:                      "$vd, $rj, $imm8, $imm4">;
 382: 
 383: class LSX3R_VVV<bits<32> op>
 384:     : Fmt3R_VVV<op, (outs LSX128:$vd), (ins LSX128:$vj, LSX128:$vk),
 385:                 "$vd, $vj, $vk">;
 386: 
 387: class LSX3R_VVR<bits<32> op>
 388:     : Fmt3R_VVR<op, (outs LSX128:$vd), (ins LSX128:$vj, GPR:$rk),
 389:                 "$vd, $vj, $rk">;
 390: 
 391: class LSX4R_VVVV<bits<32> op>
 392:     : Fmt4R_VVVV<op, (outs LSX128:$vd),
 393:                  (ins LSX128:$vj, LSX128:$vk, LSX128:$va),
 394:                  "$vd, $vj, $vk, $va">;
 395: 
 396: let Constraints = "$vd = $dst" in {
 397: 
 398: class LSX2RI1_VVRI<bits<32> op, Operand ImmOpnd = uimm1>
 399:     : Fmt2RI1_VRI<op, (outs LSX128:$dst), (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm1),
 400:                   "$vd, $rj, $imm1">;
```
- **EN**: This block declares or refines TableGen records such as `LSX2RI8I1_VRII`, `LSX2RI8I2_VRII`, `LSX2RI8I3_VRII`, `LSX2RI8I4_VRII`, `LSX3R_VVV`, `LSX3R_VVR`.
- **CN**: 该代码块声明或细化了 `LSX2RI8I1_VRII`, `LSX2RI8I2_VRII`, `LSX2RI8I3_VRII`, `LSX2RI8I4_VRII`, `LSX3R_VVV`, `LSX3R_VVR` 等 TableGen 记录。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: class LSX2RI2_VVRI<bits<32> op, Operand ImmOpnd = uimm2>
 402:     : Fmt2RI2_VRI<op, (outs LSX128:$dst), (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm2),
 403:                   "$vd, $rj, $imm2">;
 404: class LSX2RI3_VVRI<bits<32> op, Operand ImmOpnd = uimm3>
 405:     : Fmt2RI3_VRI<op, (outs LSX128:$dst), (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm3),
 406:                   "$vd, $rj, $imm3">;
 407: class LSX2RI4_VVRI<bits<32> op, Operand ImmOpnd = uimm4>
 408:     : Fmt2RI4_VRI<op, (outs LSX128:$dst), (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm4),
 409:                   "$vd, $rj, $imm4">;
 410: 
 411: class LSX2RI4_VVVI<bits<32> op, Operand ImmOpnd = uimm4>
 412:     : Fmt2RI4_VVI<op, (outs LSX128:$dst), (ins LSX128:$vd, LSX128:$vj, ImmOpnd:$imm4),
 413:                   "$vd, $vj, $imm4">;
 414: class LSX2RI5_VVVI<bits<32> op, Operand ImmOpnd = uimm5>
 415:     : Fmt2RI5_VVI<op, (outs LSX128:$dst), (ins LSX128:$vd, LSX128:$vj, ImmOpnd:$imm5),
 416:                   "$vd, $vj, $imm5">;
 417: class LSX2RI6_VVVI<bits<32> op, Operand ImmOpnd = uimm6>
 418:     : Fmt2RI6_VVI<op, (outs LSX128:$dst), (ins LSX128:$vd, LSX128:$vj, ImmOpnd:$imm6),
 419:                   "$vd, $vj, $imm6">;
 420: class LSX2RI7_VVVI<bits<32> op, Operand ImmOpnd = uimm7>
 421:     : Fmt2RI7_VVI<op, (outs LSX128:$dst), (ins LSX128:$vd, LSX128:$vj, ImmOpnd:$imm7),
 422:                   "$vd, $vj, $imm7">;
 423: 
 424: class LSX2RI8_VVVI<bits<32> op, Operand ImmOpnd = uimm8>
 425:     : Fmt2RI8_VVI<op, (outs LSX128:$dst), (ins LSX128:$vd, LSX128:$vj, ImmOpnd:$imm8),
 426:                   "$vd, $vj, $imm8">;
 427: 
 428: class LSX3R_VVVV<bits<32> op>
 429:     : Fmt3R_VVV<op, (outs LSX128:$dst), (ins LSX128:$vd, LSX128:$vj, LSX128:$vk),
 430:                 "$vd, $vj, $vk">;
 431: 
 432: } // Constraints = "$vd = $dst"
 433: 
 434: class LSX2RI9_Load<bits<32> op, Operand ImmOpnd = simm9_lsl3>
 435:     : Fmt2RI9_VRI<op, (outs LSX128:$vd), (ins GPR:$rj, ImmOpnd:$imm9),
 436:                   "$vd, $rj, $imm9">;
 437: class LSX2RI10_Load<bits<32> op, Operand ImmOpnd = simm10_lsl2>
 438:     : Fmt2RI10_VRI<op, (outs LSX128:$vd), (ins GPR:$rj, ImmOpnd:$imm10),
 439:                   "$vd, $rj, $imm10">;
 440: class LSX2RI11_Load<bits<32> op, Operand ImmOpnd = simm11_lsl1>
```
- **EN**: This block declares or refines TableGen records such as `LSX2RI2_VVRI`, `LSX2RI3_VVRI`, `LSX2RI4_VVRI`, `LSX2RI4_VVVI`, `LSX2RI5_VVVI`, `LSX2RI6_VVVI`.
- **CN**: 该代码块声明或细化了 `LSX2RI2_VVRI`, `LSX2RI3_VVRI`, `LSX2RI4_VVRI`, `LSX2RI4_VVVI`, `LSX2RI5_VVVI`, `LSX2RI6_VVVI` 等 TableGen 记录。

### Lines 441-480 / 第 441-480 行
```tablegen
 441:     : Fmt2RI11_VRI<op, (outs LSX128:$vd), (ins GPR:$rj, ImmOpnd:$imm11),
 442:                   "$vd, $rj, $imm11">;
 443: class LSX2RI12_Load<bits<32> op, Operand ImmOpnd = simm12_addlike>
 444:     : Fmt2RI12_VRI<op, (outs LSX128:$vd), (ins GPR:$rj, ImmOpnd:$imm12),
 445:                   "$vd, $rj, $imm12">;
 446: class LSX2RI12_Store<bits<32> op, Operand ImmOpnd = simm12_addlike>
 447:     : Fmt2RI12_VRI<op, (outs), (ins LSX128:$vd, GPR:$rj, ImmOpnd:$imm12),
 448:                   "$vd, $rj, $imm12">;
 449: 
 450: class LSX3R_Load<bits<32> op>
 451:     : Fmt3R_VRR<op, (outs LSX128:$vd), (ins GPR:$rj, GPR:$rk),
 452:                 "$vd, $rj, $rk">;
 453: class LSX3R_Store<bits<32> op>
 454:     : Fmt3R_VRR<op, (outs), (ins LSX128:$vd, GPR:$rj, GPR:$rk),
 455:                 "$vd, $rj, $rk">;
 456: 
 457: //===----------------------------------------------------------------------===//
 458: // Instructions
 459: //===----------------------------------------------------------------------===//
 460: 
 461: let hasSideEffects = 0, Predicates = [HasExtLSX] in {
 462: 
 463: let mayLoad = 0, mayStore = 0 in {
 464: 
 465: def VADD_B : LSX3R_VVV<0x700a0000>;
 466: def VADD_H : LSX3R_VVV<0x700a8000>;
 467: def VADD_W : LSX3R_VVV<0x700b0000>;
 468: def VADD_D : LSX3R_VVV<0x700b8000>;
 469: def VADD_Q : LSX3R_VVV<0x712d0000>;
 470: 
 471: def VSUB_B : LSX3R_VVV<0x700c0000>;
 472: def VSUB_H : LSX3R_VVV<0x700c8000>;
 473: def VSUB_W : LSX3R_VVV<0x700d0000>;
 474: def VSUB_D : LSX3R_VVV<0x700d8000>;
 475: def VSUB_Q : LSX3R_VVV<0x712d8000>;
 476: 
 477: def VADDI_BU : LSX2RI5_VVI<0x728a0000>;
 478: def VADDI_HU : LSX2RI5_VVI<0x728a8000>;
 479: def VADDI_WU : LSX2RI5_VVI<0x728b0000>;
 480: def VADDI_DU : LSX2RI5_VVI<0x728b8000>;
```
- **EN**: This block declares or refines TableGen records such as `LSX2RI12_Load`, `LSX2RI12_Store`, `LSX3R_Load`, `LSX3R_Store`, `VADD_B`, `VADD_H`.
- **CN**: 该代码块声明或细化了 `LSX2RI12_Load`, `LSX2RI12_Store`, `LSX3R_Load`, `LSX3R_Store`, `VADD_B`, `VADD_H` 等 TableGen 记录。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: 
 482: def VSUBI_BU : LSX2RI5_VVI<0x728c0000>;
 483: def VSUBI_HU : LSX2RI5_VVI<0x728c8000>;
 484: def VSUBI_WU : LSX2RI5_VVI<0x728d0000>;
 485: def VSUBI_DU : LSX2RI5_VVI<0x728d8000>;
 486: 
 487: def VNEG_B : LSX2R_VV<0x729c3000>;
 488: def VNEG_H : LSX2R_VV<0x729c3400>;
 489: def VNEG_W : LSX2R_VV<0x729c3800>;
 490: def VNEG_D : LSX2R_VV<0x729c3c00>;
 491: 
 492: def VSADD_B : LSX3R_VVV<0x70460000>;
 493: def VSADD_H : LSX3R_VVV<0x70468000>;
 494: def VSADD_W : LSX3R_VVV<0x70470000>;
 495: def VSADD_D : LSX3R_VVV<0x70478000>;
 496: def VSADD_BU : LSX3R_VVV<0x704a0000>;
 497: def VSADD_HU : LSX3R_VVV<0x704a8000>;
 498: def VSADD_WU : LSX3R_VVV<0x704b0000>;
 499: def VSADD_DU : LSX3R_VVV<0x704b8000>;
 500: 
 501: def VSSUB_B : LSX3R_VVV<0x70480000>;
 502: def VSSUB_H : LSX3R_VVV<0x70488000>;
 503: def VSSUB_W : LSX3R_VVV<0x70490000>;
 504: def VSSUB_D : LSX3R_VVV<0x70498000>;
 505: def VSSUB_BU : LSX3R_VVV<0x704c0000>;
 506: def VSSUB_HU : LSX3R_VVV<0x704c8000>;
 507: def VSSUB_WU : LSX3R_VVV<0x704d0000>;
 508: def VSSUB_DU : LSX3R_VVV<0x704d8000>;
 509: 
 510: def VHADDW_H_B : LSX3R_VVV<0x70540000>;
 511: def VHADDW_W_H : LSX3R_VVV<0x70548000>;
 512: def VHADDW_D_W : LSX3R_VVV<0x70550000>;
 513: def VHADDW_Q_D : LSX3R_VVV<0x70558000>;
 514: def VHADDW_HU_BU : LSX3R_VVV<0x70580000>;
 515: def VHADDW_WU_HU : LSX3R_VVV<0x70588000>;
 516: def VHADDW_DU_WU : LSX3R_VVV<0x70590000>;
 517: def VHADDW_QU_DU : LSX3R_VVV<0x70598000>;
 518: 
 519: def VHSUBW_H_B : LSX3R_VVV<0x70560000>;
 520: def VHSUBW_W_H : LSX3R_VVV<0x70568000>;
```
- **EN**: This block declares or refines TableGen records such as `VSUBI_BU`, `VSUBI_HU`, `VSUBI_WU`, `VSUBI_DU`, `VNEG_B`, `VNEG_H`.
- **CN**: 该代码块声明或细化了 `VSUBI_BU`, `VSUBI_HU`, `VSUBI_WU`, `VSUBI_DU`, `VNEG_B`, `VNEG_H` 等 TableGen 记录。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: def VHSUBW_D_W : LSX3R_VVV<0x70570000>;
 522: def VHSUBW_Q_D : LSX3R_VVV<0x70578000>;
 523: def VHSUBW_HU_BU : LSX3R_VVV<0x705a0000>;
 524: def VHSUBW_WU_HU : LSX3R_VVV<0x705a8000>;
 525: def VHSUBW_DU_WU : LSX3R_VVV<0x705b0000>;
 526: def VHSUBW_QU_DU : LSX3R_VVV<0x705b8000>;
 527: 
 528: def VADDWEV_H_B : LSX3R_VVV<0x701e0000>;
 529: def VADDWEV_W_H : LSX3R_VVV<0x701e8000>;
 530: def VADDWEV_D_W : LSX3R_VVV<0x701f0000>;
 531: def VADDWEV_Q_D : LSX3R_VVV<0x701f8000>;
 532: def VADDWOD_H_B : LSX3R_VVV<0x70220000>;
 533: def VADDWOD_W_H : LSX3R_VVV<0x70228000>;
 534: def VADDWOD_D_W : LSX3R_VVV<0x70230000>;
 535: def VADDWOD_Q_D : LSX3R_VVV<0x70238000>;
 536: 
 537: def VSUBWEV_H_B : LSX3R_VVV<0x70200000>;
 538: def VSUBWEV_W_H : LSX3R_VVV<0x70208000>;
 539: def VSUBWEV_D_W : LSX3R_VVV<0x70210000>;
 540: def VSUBWEV_Q_D : LSX3R_VVV<0x70218000>;
 541: def VSUBWOD_H_B : LSX3R_VVV<0x70240000>;
 542: def VSUBWOD_W_H : LSX3R_VVV<0x70248000>;
 543: def VSUBWOD_D_W : LSX3R_VVV<0x70250000>;
 544: def VSUBWOD_Q_D : LSX3R_VVV<0x70258000>;
 545: 
 546: def VADDWEV_H_BU : LSX3R_VVV<0x702e0000>;
 547: def VADDWEV_W_HU : LSX3R_VVV<0x702e8000>;
 548: def VADDWEV_D_WU : LSX3R_VVV<0x702f0000>;
 549: def VADDWEV_Q_DU : LSX3R_VVV<0x702f8000>;
 550: def VADDWOD_H_BU : LSX3R_VVV<0x70320000>;
 551: def VADDWOD_W_HU : LSX3R_VVV<0x70328000>;
 552: def VADDWOD_D_WU : LSX3R_VVV<0x70330000>;
 553: def VADDWOD_Q_DU : LSX3R_VVV<0x70338000>;
 554: 
 555: def VSUBWEV_H_BU : LSX3R_VVV<0x70300000>;
 556: def VSUBWEV_W_HU : LSX3R_VVV<0x70308000>;
 557: def VSUBWEV_D_WU : LSX3R_VVV<0x70310000>;
 558: def VSUBWEV_Q_DU : LSX3R_VVV<0x70318000>;
 559: def VSUBWOD_H_BU : LSX3R_VVV<0x70340000>;
 560: def VSUBWOD_W_HU : LSX3R_VVV<0x70348000>;
```
- **EN**: This block declares or refines TableGen records such as `VHSUBW_D_W`, `VHSUBW_Q_D`, `VHSUBW_HU_BU`, `VHSUBW_WU_HU`, `VHSUBW_DU_WU`, `VHSUBW_QU_DU`.
- **CN**: 该代码块声明或细化了 `VHSUBW_D_W`, `VHSUBW_Q_D`, `VHSUBW_HU_BU`, `VHSUBW_WU_HU`, `VHSUBW_DU_WU`, `VHSUBW_QU_DU` 等 TableGen 记录。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: def VSUBWOD_D_WU : LSX3R_VVV<0x70350000>;
 562: def VSUBWOD_Q_DU : LSX3R_VVV<0x70358000>;
 563: 
 564: def VADDWEV_H_BU_B : LSX3R_VVV<0x703e0000>;
 565: def VADDWEV_W_HU_H : LSX3R_VVV<0x703e8000>;
 566: def VADDWEV_D_WU_W : LSX3R_VVV<0x703f0000>;
 567: def VADDWEV_Q_DU_D : LSX3R_VVV<0x703f8000>;
 568: def VADDWOD_H_BU_B : LSX3R_VVV<0x70400000>;
 569: def VADDWOD_W_HU_H : LSX3R_VVV<0x70408000>;
 570: def VADDWOD_D_WU_W : LSX3R_VVV<0x70410000>;
 571: def VADDWOD_Q_DU_D : LSX3R_VVV<0x70418000>;
 572: 
 573: def VAVG_B : LSX3R_VVV<0x70640000>;
 574: def VAVG_H : LSX3R_VVV<0x70648000>;
 575: def VAVG_W : LSX3R_VVV<0x70650000>;
 576: def VAVG_D : LSX3R_VVV<0x70658000>;
 577: def VAVG_BU : LSX3R_VVV<0x70660000>;
 578: def VAVG_HU : LSX3R_VVV<0x70668000>;
 579: def VAVG_WU : LSX3R_VVV<0x70670000>;
 580: def VAVG_DU : LSX3R_VVV<0x70678000>;
 581: def VAVGR_B : LSX3R_VVV<0x70680000>;
 582: def VAVGR_H : LSX3R_VVV<0x70688000>;
 583: def VAVGR_W : LSX3R_VVV<0x70690000>;
 584: def VAVGR_D : LSX3R_VVV<0x70698000>;
 585: def VAVGR_BU : LSX3R_VVV<0x706a0000>;
 586: def VAVGR_HU : LSX3R_VVV<0x706a8000>;
 587: def VAVGR_WU : LSX3R_VVV<0x706b0000>;
 588: def VAVGR_DU : LSX3R_VVV<0x706b8000>;
 589: 
 590: def VABSD_B : LSX3R_VVV<0x70600000>;
 591: def VABSD_H : LSX3R_VVV<0x70608000>;
 592: def VABSD_W : LSX3R_VVV<0x70610000>;
 593: def VABSD_D : LSX3R_VVV<0x70618000>;
 594: def VABSD_BU : LSX3R_VVV<0x70620000>;
 595: def VABSD_HU : LSX3R_VVV<0x70628000>;
 596: def VABSD_WU : LSX3R_VVV<0x70630000>;
 597: def VABSD_DU : LSX3R_VVV<0x70638000>;
 598: 
 599: def VADDA_B : LSX3R_VVV<0x705c0000>;
 600: def VADDA_H : LSX3R_VVV<0x705c8000>;
```
- **EN**: This block declares or refines TableGen records such as `VSUBWOD_D_WU`, `VSUBWOD_Q_DU`, `VADDWEV_H_BU_B`, `VADDWEV_W_HU_H`, `VADDWEV_D_WU_W`, `VADDWEV_Q_DU_D`.
- **CN**: 该代码块声明或细化了 `VSUBWOD_D_WU`, `VSUBWOD_Q_DU`, `VADDWEV_H_BU_B`, `VADDWEV_W_HU_H`, `VADDWEV_D_WU_W`, `VADDWEV_Q_DU_D` 等 TableGen 记录。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: def VADDA_W : LSX3R_VVV<0x705d0000>;
 602: def VADDA_D : LSX3R_VVV<0x705d8000>;
 603: 
 604: def VMAX_B : LSX3R_VVV<0x70700000>;
 605: def VMAX_H : LSX3R_VVV<0x70708000>;
 606: def VMAX_W : LSX3R_VVV<0x70710000>;
 607: def VMAX_D : LSX3R_VVV<0x70718000>;
 608: def VMAXI_B : LSX2RI5_VVI<0x72900000, simm5>;
 609: def VMAXI_H : LSX2RI5_VVI<0x72908000, simm5>;
 610: def VMAXI_W : LSX2RI5_VVI<0x72910000, simm5>;
 611: def VMAXI_D : LSX2RI5_VVI<0x72918000, simm5>;
 612: def VMAX_BU : LSX3R_VVV<0x70740000>;
 613: def VMAX_HU : LSX3R_VVV<0x70748000>;
 614: def VMAX_WU : LSX3R_VVV<0x70750000>;
 615: def VMAX_DU : LSX3R_VVV<0x70758000>;
 616: def VMAXI_BU : LSX2RI5_VVI<0x72940000>;
 617: def VMAXI_HU : LSX2RI5_VVI<0x72948000>;
 618: def VMAXI_WU : LSX2RI5_VVI<0x72950000>;
 619: def VMAXI_DU : LSX2RI5_VVI<0x72958000>;
 620: 
 621: def VMIN_B : LSX3R_VVV<0x70720000>;
 622: def VMIN_H : LSX3R_VVV<0x70728000>;
 623: def VMIN_W : LSX3R_VVV<0x70730000>;
 624: def VMIN_D : LSX3R_VVV<0x70738000>;
 625: def VMINI_B : LSX2RI5_VVI<0x72920000, simm5>;
 626: def VMINI_H : LSX2RI5_VVI<0x72928000, simm5>;
 627: def VMINI_W : LSX2RI5_VVI<0x72930000, simm5>;
 628: def VMINI_D : LSX2RI5_VVI<0x72938000, simm5>;
 629: def VMIN_BU : LSX3R_VVV<0x70760000>;
 630: def VMIN_HU : LSX3R_VVV<0x70768000>;
 631: def VMIN_WU : LSX3R_VVV<0x70770000>;
 632: def VMIN_DU : LSX3R_VVV<0x70778000>;
 633: def VMINI_BU : LSX2RI5_VVI<0x72960000>;
 634: def VMINI_HU : LSX2RI5_VVI<0x72968000>;
 635: def VMINI_WU : LSX2RI5_VVI<0x72970000>;
 636: def VMINI_DU : LSX2RI5_VVI<0x72978000>;
 637: 
 638: def VMUL_B : LSX3R_VVV<0x70840000>;
 639: def VMUL_H : LSX3R_VVV<0x70848000>;
 640: def VMUL_W : LSX3R_VVV<0x70850000>;
```
- **EN**: This block declares or refines TableGen records such as `VADDA_W`, `VADDA_D`, `VMAX_B`, `VMAX_H`, `VMAX_W`, `VMAX_D`.
- **CN**: 该代码块声明或细化了 `VADDA_W`, `VADDA_D`, `VMAX_B`, `VMAX_H`, `VMAX_W`, `VMAX_D` 等 TableGen 记录。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: def VMUL_D : LSX3R_VVV<0x70858000>;
 642: 
 643: def VMUH_B : LSX3R_VVV<0x70860000>;
 644: def VMUH_H : LSX3R_VVV<0x70868000>;
 645: def VMUH_W : LSX3R_VVV<0x70870000>;
 646: def VMUH_D : LSX3R_VVV<0x70878000>;
 647: def VMUH_BU : LSX3R_VVV<0x70880000>;
 648: def VMUH_HU : LSX3R_VVV<0x70888000>;
 649: def VMUH_WU : LSX3R_VVV<0x70890000>;
 650: def VMUH_DU : LSX3R_VVV<0x70898000>;
 651: 
 652: def VMULWEV_H_B : LSX3R_VVV<0x70900000>;
 653: def VMULWEV_W_H : LSX3R_VVV<0x70908000>;
 654: def VMULWEV_D_W : LSX3R_VVV<0x70910000>;
 655: def VMULWEV_Q_D : LSX3R_VVV<0x70918000>;
 656: def VMULWOD_H_B : LSX3R_VVV<0x70920000>;
 657: def VMULWOD_W_H : LSX3R_VVV<0x70928000>;
 658: def VMULWOD_D_W : LSX3R_VVV<0x70930000>;
 659: def VMULWOD_Q_D : LSX3R_VVV<0x70938000>;
 660: def VMULWEV_H_BU : LSX3R_VVV<0x70980000>;
 661: def VMULWEV_W_HU : LSX3R_VVV<0x70988000>;
 662: def VMULWEV_D_WU : LSX3R_VVV<0x70990000>;
 663: def VMULWEV_Q_DU : LSX3R_VVV<0x70998000>;
 664: def VMULWOD_H_BU : LSX3R_VVV<0x709a0000>;
 665: def VMULWOD_W_HU : LSX3R_VVV<0x709a8000>;
 666: def VMULWOD_D_WU : LSX3R_VVV<0x709b0000>;
 667: def VMULWOD_Q_DU : LSX3R_VVV<0x709b8000>;
 668: def VMULWEV_H_BU_B : LSX3R_VVV<0x70a00000>;
 669: def VMULWEV_W_HU_H : LSX3R_VVV<0x70a08000>;
 670: def VMULWEV_D_WU_W : LSX3R_VVV<0x70a10000>;
 671: def VMULWEV_Q_DU_D : LSX3R_VVV<0x70a18000>;
 672: def VMULWOD_H_BU_B : LSX3R_VVV<0x70a20000>;
 673: def VMULWOD_W_HU_H : LSX3R_VVV<0x70a28000>;
 674: def VMULWOD_D_WU_W : LSX3R_VVV<0x70a30000>;
 675: def VMULWOD_Q_DU_D : LSX3R_VVV<0x70a38000>;
 676: 
 677: def VMADD_B : LSX3R_VVVV<0x70a80000>;
 678: def VMADD_H : LSX3R_VVVV<0x70a88000>;
 679: def VMADD_W : LSX3R_VVVV<0x70a90000>;
 680: def VMADD_D : LSX3R_VVVV<0x70a98000>;
```
- **EN**: This block declares or refines TableGen records such as `VMUL_D`, `VMUH_B`, `VMUH_H`, `VMUH_W`, `VMUH_D`, `VMUH_BU`.
- **CN**: 该代码块声明或细化了 `VMUL_D`, `VMUH_B`, `VMUH_H`, `VMUH_W`, `VMUH_D`, `VMUH_BU` 等 TableGen 记录。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: 
 682: def VMSUB_B : LSX3R_VVVV<0x70aa0000>;
 683: def VMSUB_H : LSX3R_VVVV<0x70aa8000>;
 684: def VMSUB_W : LSX3R_VVVV<0x70ab0000>;
 685: def VMSUB_D : LSX3R_VVVV<0x70ab8000>;
 686: 
 687: def VMADDWEV_H_B : LSX3R_VVVV<0x70ac0000>;
 688: def VMADDWEV_W_H : LSX3R_VVVV<0x70ac8000>;
 689: def VMADDWEV_D_W : LSX3R_VVVV<0x70ad0000>;
 690: def VMADDWEV_Q_D : LSX3R_VVVV<0x70ad8000>;
 691: def VMADDWOD_H_B : LSX3R_VVVV<0x70ae0000>;
 692: def VMADDWOD_W_H : LSX3R_VVVV<0x70ae8000>;
 693: def VMADDWOD_D_W : LSX3R_VVVV<0x70af0000>;
 694: def VMADDWOD_Q_D : LSX3R_VVVV<0x70af8000>;
 695: def VMADDWEV_H_BU : LSX3R_VVVV<0x70b40000>;
 696: def VMADDWEV_W_HU : LSX3R_VVVV<0x70b48000>;
 697: def VMADDWEV_D_WU : LSX3R_VVVV<0x70b50000>;
 698: def VMADDWEV_Q_DU : LSX3R_VVVV<0x70b58000>;
 699: def VMADDWOD_H_BU : LSX3R_VVVV<0x70b60000>;
 700: def VMADDWOD_W_HU : LSX3R_VVVV<0x70b68000>;
 701: def VMADDWOD_D_WU : LSX3R_VVVV<0x70b70000>;
 702: def VMADDWOD_Q_DU : LSX3R_VVVV<0x70b78000>;
 703: def VMADDWEV_H_BU_B : LSX3R_VVVV<0x70bc0000>;
 704: def VMADDWEV_W_HU_H : LSX3R_VVVV<0x70bc8000>;
 705: def VMADDWEV_D_WU_W : LSX3R_VVVV<0x70bd0000>;
 706: def VMADDWEV_Q_DU_D : LSX3R_VVVV<0x70bd8000>;
 707: def VMADDWOD_H_BU_B : LSX3R_VVVV<0x70be0000>;
 708: def VMADDWOD_W_HU_H : LSX3R_VVVV<0x70be8000>;
 709: def VMADDWOD_D_WU_W : LSX3R_VVVV<0x70bf0000>;
 710: def VMADDWOD_Q_DU_D : LSX3R_VVVV<0x70bf8000>;
 711: 
 712: def VDIV_B : LSX3R_VVV<0x70e00000>;
 713: def VDIV_H : LSX3R_VVV<0x70e08000>;
 714: def VDIV_W : LSX3R_VVV<0x70e10000>;
 715: def VDIV_D : LSX3R_VVV<0x70e18000>;
 716: def VDIV_BU : LSX3R_VVV<0x70e40000>;
 717: def VDIV_HU : LSX3R_VVV<0x70e48000>;
 718: def VDIV_WU : LSX3R_VVV<0x70e50000>;
 719: def VDIV_DU : LSX3R_VVV<0x70e58000>;
 720: 
```
- **EN**: This block declares or refines TableGen records such as `VMSUB_B`, `VMSUB_H`, `VMSUB_W`, `VMSUB_D`, `VMADDWEV_H_B`, `VMADDWEV_W_H`.
- **CN**: 该代码块声明或细化了 `VMSUB_B`, `VMSUB_H`, `VMSUB_W`, `VMSUB_D`, `VMADDWEV_H_B`, `VMADDWEV_W_H` 等 TableGen 记录。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: def VMOD_B : LSX3R_VVV<0x70e20000>;
 722: def VMOD_H : LSX3R_VVV<0x70e28000>;
 723: def VMOD_W : LSX3R_VVV<0x70e30000>;
 724: def VMOD_D : LSX3R_VVV<0x70e38000>;
 725: def VMOD_BU : LSX3R_VVV<0x70e60000>;
 726: def VMOD_HU : LSX3R_VVV<0x70e68000>;
 727: def VMOD_WU : LSX3R_VVV<0x70e70000>;
 728: def VMOD_DU : LSX3R_VVV<0x70e78000>;
 729: 
 730: def VSAT_B : LSX2RI3_VVI<0x73242000>;
 731: def VSAT_H : LSX2RI4_VVI<0x73244000>;
 732: def VSAT_W : LSX2RI5_VVI<0x73248000>;
 733: def VSAT_D : LSX2RI6_VVI<0x73250000>;
 734: def VSAT_BU : LSX2RI3_VVI<0x73282000>;
 735: def VSAT_HU : LSX2RI4_VVI<0x73284000>;
 736: def VSAT_WU : LSX2RI5_VVI<0x73288000>;
 737: def VSAT_DU : LSX2RI6_VVI<0x73290000>;
 738: 
 739: def VEXTH_H_B : LSX2R_VV<0x729ee000>;
 740: def VEXTH_W_H : LSX2R_VV<0x729ee400>;
 741: def VEXTH_D_W : LSX2R_VV<0x729ee800>;
 742: def VEXTH_Q_D : LSX2R_VV<0x729eec00>;
 743: def VEXTH_HU_BU : LSX2R_VV<0x729ef000>;
 744: def VEXTH_WU_HU : LSX2R_VV<0x729ef400>;
 745: def VEXTH_DU_WU : LSX2R_VV<0x729ef800>;
 746: def VEXTH_QU_DU : LSX2R_VV<0x729efc00>;
 747: 
 748: def VSIGNCOV_B : LSX3R_VVV<0x712e0000>;
 749: def VSIGNCOV_H : LSX3R_VVV<0x712e8000>;
 750: def VSIGNCOV_W : LSX3R_VVV<0x712f0000>;
 751: def VSIGNCOV_D : LSX3R_VVV<0x712f8000>;
 752: 
 753: def VMSKLTZ_B : LSX2R_VV<0x729c4000>;
 754: def VMSKLTZ_H : LSX2R_VV<0x729c4400>;
 755: def VMSKLTZ_W : LSX2R_VV<0x729c4800>;
 756: def VMSKLTZ_D : LSX2R_VV<0x729c4c00>;
 757: 
 758: def VMSKGEZ_B : LSX2R_VV<0x729c5000>;
 759: 
 760: def VMSKNZ_B : LSX2R_VV<0x729c6000>;
```
- **EN**: This block declares or refines TableGen records such as `VMOD_B`, `VMOD_H`, `VMOD_W`, `VMOD_D`, `VMOD_BU`, `VMOD_HU`.
- **CN**: 该代码块声明或细化了 `VMOD_B`, `VMOD_H`, `VMOD_W`, `VMOD_D`, `VMOD_BU`, `VMOD_HU` 等 TableGen 记录。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: 
 762: let isReMaterializable = 1, isAsCheapAsAMove = 1 in {
 763: def VLDI : LSX1RI13_VI<0x73e00000>;
 764: }
 765: 
 766: def VAND_V : LSX3R_VVV<0x71260000>;
 767: def VOR_V : LSX3R_VVV<0x71268000>;
 768: def VXOR_V : LSX3R_VVV<0x71270000>;
 769: def VNOR_V : LSX3R_VVV<0x71278000>;
 770: def VANDN_V : LSX3R_VVV<0x71280000>;
 771: def VORN_V : LSX3R_VVV<0x71288000>;
 772: 
 773: def VANDI_B : LSX2RI8_VVI<0x73d00000>;
 774: def VORI_B : LSX2RI8_VVI<0x73d40000>;
 775: def VXORI_B : LSX2RI8_VVI<0x73d80000>;
 776: def VNORI_B : LSX2RI8_VVI<0x73dc0000>;
 777: 
 778: def VSLL_B : LSX3R_VVV<0x70e80000>;
 779: def VSLL_H : LSX3R_VVV<0x70e88000>;
 780: def VSLL_W : LSX3R_VVV<0x70e90000>;
 781: def VSLL_D : LSX3R_VVV<0x70e98000>;
 782: def VSLLI_B : LSX2RI3_VVI<0x732c2000>;
 783: def VSLLI_H : LSX2RI4_VVI<0x732c4000>;
 784: def VSLLI_W : LSX2RI5_VVI<0x732c8000>;
 785: def VSLLI_D : LSX2RI6_VVI<0x732d0000>;
 786: 
 787: def VSRL_B : LSX3R_VVV<0x70ea0000>;
 788: def VSRL_H : LSX3R_VVV<0x70ea8000>;
 789: def VSRL_W : LSX3R_VVV<0x70eb0000>;
 790: def VSRL_D : LSX3R_VVV<0x70eb8000>;
 791: def VSRLI_B : LSX2RI3_VVI<0x73302000>;
 792: def VSRLI_H : LSX2RI4_VVI<0x73304000>;
 793: def VSRLI_W : LSX2RI5_VVI<0x73308000>;
 794: def VSRLI_D : LSX2RI6_VVI<0x73310000>;
 795: 
 796: def VSRA_B : LSX3R_VVV<0x70ec0000>;
 797: def VSRA_H : LSX3R_VVV<0x70ec8000>;
 798: def VSRA_W : LSX3R_VVV<0x70ed0000>;
 799: def VSRA_D : LSX3R_VVV<0x70ed8000>;
 800: def VSRAI_B : LSX2RI3_VVI<0x73342000>;
```
- **EN**: This block declares or refines TableGen records such as `VLDI`, `VAND_V`, `VOR_V`, `VXOR_V`, `VNOR_V`, `VANDN_V`.
- **CN**: 该代码块声明或细化了 `VLDI`, `VAND_V`, `VOR_V`, `VXOR_V`, `VNOR_V`, `VANDN_V` 等 TableGen 记录。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: def VSRAI_H : LSX2RI4_VVI<0x73344000>;
 802: def VSRAI_W : LSX2RI5_VVI<0x73348000>;
 803: def VSRAI_D : LSX2RI6_VVI<0x73350000>;
 804: 
 805: def VROTR_B : LSX3R_VVV<0x70ee0000>;
 806: def VROTR_H : LSX3R_VVV<0x70ee8000>;
 807: def VROTR_W : LSX3R_VVV<0x70ef0000>;
 808: def VROTR_D : LSX3R_VVV<0x70ef8000>;
 809: def VROTRI_B : LSX2RI3_VVI<0x72a02000>;
 810: def VROTRI_H : LSX2RI4_VVI<0x72a04000>;
 811: def VROTRI_W : LSX2RI5_VVI<0x72a08000>;
 812: def VROTRI_D : LSX2RI6_VVI<0x72a10000>;
 813: 
 814: def VSLLWIL_H_B : LSX2RI3_VVI<0x73082000>;
 815: def VSLLWIL_W_H : LSX2RI4_VVI<0x73084000>;
 816: def VSLLWIL_D_W : LSX2RI5_VVI<0x73088000>;
 817: def VEXTL_Q_D : LSX2R_VV<0x73090000>;
 818: def VSLLWIL_HU_BU : LSX2RI3_VVI<0x730c2000>;
 819: def VSLLWIL_WU_HU : LSX2RI4_VVI<0x730c4000>;
 820: def VSLLWIL_DU_WU : LSX2RI5_VVI<0x730c8000>;
 821: def VEXTL_QU_DU : LSX2R_VV<0x730d0000>;
 822: 
 823: def VSRLR_B : LSX3R_VVV<0x70f00000>;
 824: def VSRLR_H : LSX3R_VVV<0x70f08000>;
 825: def VSRLR_W : LSX3R_VVV<0x70f10000>;
 826: def VSRLR_D : LSX3R_VVV<0x70f18000>;
 827: def VSRLRI_B : LSX2RI3_VVI<0x72a42000>;
 828: def VSRLRI_H : LSX2RI4_VVI<0x72a44000>;
 829: def VSRLRI_W : LSX2RI5_VVI<0x72a48000>;
 830: def VSRLRI_D : LSX2RI6_VVI<0x72a50000>;
 831: 
 832: def VSRAR_B : LSX3R_VVV<0x70f20000>;
 833: def VSRAR_H : LSX3R_VVV<0x70f28000>;
 834: def VSRAR_W : LSX3R_VVV<0x70f30000>;
 835: def VSRAR_D : LSX3R_VVV<0x70f38000>;
 836: def VSRARI_B : LSX2RI3_VVI<0x72a82000>;
 837: def VSRARI_H : LSX2RI4_VVI<0x72a84000>;
 838: def VSRARI_W : LSX2RI5_VVI<0x72a88000>;
 839: def VSRARI_D : LSX2RI6_VVI<0x72a90000>;
 840: 
```
- **EN**: This block declares or refines TableGen records such as `VSRAI_H`, `VSRAI_W`, `VSRAI_D`, `VROTR_B`, `VROTR_H`, `VROTR_W`.
- **CN**: 该代码块声明或细化了 `VSRAI_H`, `VSRAI_W`, `VSRAI_D`, `VROTR_B`, `VROTR_H`, `VROTR_W` 等 TableGen 记录。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: def VSRLN_B_H : LSX3R_VVV<0x70f48000>;
 842: def VSRLN_H_W : LSX3R_VVV<0x70f50000>;
 843: def VSRLN_W_D : LSX3R_VVV<0x70f58000>;
 844: def VSRAN_B_H : LSX3R_VVV<0x70f68000>;
 845: def VSRAN_H_W : LSX3R_VVV<0x70f70000>;
 846: def VSRAN_W_D : LSX3R_VVV<0x70f78000>;
 847: 
 848: def VSRLNI_B_H : LSX2RI4_VVVI<0x73404000>;
 849: def VSRLNI_H_W : LSX2RI5_VVVI<0x73408000>;
 850: def VSRLNI_W_D : LSX2RI6_VVVI<0x73410000>;
 851: def VSRLNI_D_Q : LSX2RI7_VVVI<0x73420000>;
 852: def VSRANI_B_H : LSX2RI4_VVVI<0x73584000>;
 853: def VSRANI_H_W : LSX2RI5_VVVI<0x73588000>;
 854: def VSRANI_W_D : LSX2RI6_VVVI<0x73590000>;
 855: def VSRANI_D_Q : LSX2RI7_VVVI<0x735a0000>;
 856: 
 857: def VSRLRN_B_H : LSX3R_VVV<0x70f88000>;
 858: def VSRLRN_H_W : LSX3R_VVV<0x70f90000>;
 859: def VSRLRN_W_D : LSX3R_VVV<0x70f98000>;
 860: def VSRARN_B_H : LSX3R_VVV<0x70fa8000>;
 861: def VSRARN_H_W : LSX3R_VVV<0x70fb0000>;
 862: def VSRARN_W_D : LSX3R_VVV<0x70fb8000>;
 863: 
 864: def VSRLRNI_B_H : LSX2RI4_VVVI<0x73444000>;
 865: def VSRLRNI_H_W : LSX2RI5_VVVI<0x73448000>;
 866: def VSRLRNI_W_D : LSX2RI6_VVVI<0x73450000>;
 867: def VSRLRNI_D_Q : LSX2RI7_VVVI<0x73460000>;
 868: def VSRARNI_B_H : LSX2RI4_VVVI<0x735c4000>;
 869: def VSRARNI_H_W : LSX2RI5_VVVI<0x735c8000>;
 870: def VSRARNI_W_D : LSX2RI6_VVVI<0x735d0000>;
 871: def VSRARNI_D_Q : LSX2RI7_VVVI<0x735e0000>;
 872: 
 873: def VSSRLN_B_H : LSX3R_VVV<0x70fc8000>;
 874: def VSSRLN_H_W : LSX3R_VVV<0x70fd0000>;
 875: def VSSRLN_W_D : LSX3R_VVV<0x70fd8000>;
 876: def VSSRAN_B_H : LSX3R_VVV<0x70fe8000>;
 877: def VSSRAN_H_W : LSX3R_VVV<0x70ff0000>;
 878: def VSSRAN_W_D : LSX3R_VVV<0x70ff8000>;
 879: def VSSRLN_BU_H : LSX3R_VVV<0x71048000>;
 880: def VSSRLN_HU_W : LSX3R_VVV<0x71050000>;
```
- **EN**: This block declares or refines TableGen records such as `VSRLN_B_H`, `VSRLN_H_W`, `VSRLN_W_D`, `VSRAN_B_H`, `VSRAN_H_W`, `VSRAN_W_D`.
- **CN**: 该代码块声明或细化了 `VSRLN_B_H`, `VSRLN_H_W`, `VSRLN_W_D`, `VSRAN_B_H`, `VSRAN_H_W`, `VSRAN_W_D` 等 TableGen 记录。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: def VSSRLN_WU_D : LSX3R_VVV<0x71058000>;
 882: def VSSRAN_BU_H : LSX3R_VVV<0x71068000>;
 883: def VSSRAN_HU_W : LSX3R_VVV<0x71070000>;
 884: def VSSRAN_WU_D : LSX3R_VVV<0x71078000>;
 885: 
 886: def VSSRLNI_B_H : LSX2RI4_VVVI<0x73484000>;
 887: def VSSRLNI_H_W : LSX2RI5_VVVI<0x73488000>;
 888: def VSSRLNI_W_D : LSX2RI6_VVVI<0x73490000>;
 889: def VSSRLNI_D_Q : LSX2RI7_VVVI<0x734a0000>;
 890: def VSSRANI_B_H : LSX2RI4_VVVI<0x73604000>;
 891: def VSSRANI_H_W : LSX2RI5_VVVI<0x73608000>;
 892: def VSSRANI_W_D : LSX2RI6_VVVI<0x73610000>;
 893: def VSSRANI_D_Q : LSX2RI7_VVVI<0x73620000>;
 894: def VSSRLNI_BU_H : LSX2RI4_VVVI<0x734c4000>;
 895: def VSSRLNI_HU_W : LSX2RI5_VVVI<0x734c8000>;
 896: def VSSRLNI_WU_D : LSX2RI6_VVVI<0x734d0000>;
 897: def VSSRLNI_DU_Q : LSX2RI7_VVVI<0x734e0000>;
 898: def VSSRANI_BU_H : LSX2RI4_VVVI<0x73644000>;
 899: def VSSRANI_HU_W : LSX2RI5_VVVI<0x73648000>;
 900: def VSSRANI_WU_D : LSX2RI6_VVVI<0x73650000>;
 901: def VSSRANI_DU_Q : LSX2RI7_VVVI<0x73660000>;
 902: 
 903: def VSSRLRN_B_H : LSX3R_VVV<0x71008000>;
 904: def VSSRLRN_H_W : LSX3R_VVV<0x71010000>;
 905: def VSSRLRN_W_D : LSX3R_VVV<0x71018000>;
 906: def VSSRARN_B_H : LSX3R_VVV<0x71028000>;
 907: def VSSRARN_H_W : LSX3R_VVV<0x71030000>;
 908: def VSSRARN_W_D : LSX3R_VVV<0x71038000>;
 909: def VSSRLRN_BU_H : LSX3R_VVV<0x71088000>;
 910: def VSSRLRN_HU_W : LSX3R_VVV<0x71090000>;
 911: def VSSRLRN_WU_D : LSX3R_VVV<0x71098000>;
 912: def VSSRARN_BU_H : LSX3R_VVV<0x710a8000>;
 913: def VSSRARN_HU_W : LSX3R_VVV<0x710b0000>;
 914: def VSSRARN_WU_D : LSX3R_VVV<0x710b8000>;
 915: 
 916: def VSSRLRNI_B_H : LSX2RI4_VVVI<0x73504000>;
 917: def VSSRLRNI_H_W : LSX2RI5_VVVI<0x73508000>;
 918: def VSSRLRNI_W_D : LSX2RI6_VVVI<0x73510000>;
 919: def VSSRLRNI_D_Q : LSX2RI7_VVVI<0x73520000>;
 920: def VSSRARNI_B_H : LSX2RI4_VVVI<0x73684000>;
```
- **EN**: This block declares or refines TableGen records such as `VSSRLN_WU_D`, `VSSRAN_BU_H`, `VSSRAN_HU_W`, `VSSRAN_WU_D`, `VSSRLNI_B_H`, `VSSRLNI_H_W`.
- **CN**: 该代码块声明或细化了 `VSSRLN_WU_D`, `VSSRAN_BU_H`, `VSSRAN_HU_W`, `VSSRAN_WU_D`, `VSSRLNI_B_H`, `VSSRLNI_H_W` 等 TableGen 记录。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: def VSSRARNI_H_W : LSX2RI5_VVVI<0x73688000>;
 922: def VSSRARNI_W_D : LSX2RI6_VVVI<0x73690000>;
 923: def VSSRARNI_D_Q : LSX2RI7_VVVI<0x736a0000>;
 924: def VSSRLRNI_BU_H : LSX2RI4_VVVI<0x73544000>;
 925: def VSSRLRNI_HU_W : LSX2RI5_VVVI<0x73548000>;
 926: def VSSRLRNI_WU_D : LSX2RI6_VVVI<0x73550000>;
 927: def VSSRLRNI_DU_Q : LSX2RI7_VVVI<0x73560000>;
 928: def VSSRARNI_BU_H : LSX2RI4_VVVI<0x736c4000>;
 929: def VSSRARNI_HU_W : LSX2RI5_VVVI<0x736c8000>;
 930: def VSSRARNI_WU_D : LSX2RI6_VVVI<0x736d0000>;
 931: def VSSRARNI_DU_Q : LSX2RI7_VVVI<0x736e0000>;
 932: 
 933: def VCLO_B : LSX2R_VV<0x729c0000>;
 934: def VCLO_H : LSX2R_VV<0x729c0400>;
 935: def VCLO_W : LSX2R_VV<0x729c0800>;
 936: def VCLO_D : LSX2R_VV<0x729c0c00>;
 937: def VCLZ_B : LSX2R_VV<0x729c1000>;
 938: def VCLZ_H : LSX2R_VV<0x729c1400>;
 939: def VCLZ_W : LSX2R_VV<0x729c1800>;
 940: def VCLZ_D : LSX2R_VV<0x729c1c00>;
 941: 
 942: def VPCNT_B : LSX2R_VV<0x729c2000>;
 943: def VPCNT_H : LSX2R_VV<0x729c2400>;
 944: def VPCNT_W : LSX2R_VV<0x729c2800>;
 945: def VPCNT_D : LSX2R_VV<0x729c2c00>;
 946: 
 947: def VBITCLR_B : LSX3R_VVV<0x710c0000>;
 948: def VBITCLR_H : LSX3R_VVV<0x710c8000>;
 949: def VBITCLR_W : LSX3R_VVV<0x710d0000>;
 950: def VBITCLR_D : LSX3R_VVV<0x710d8000>;
 951: def VBITCLRI_B : LSX2RI3_VVI<0x73102000>;
 952: def VBITCLRI_H : LSX2RI4_VVI<0x73104000>;
 953: def VBITCLRI_W : LSX2RI5_VVI<0x73108000>;
 954: def VBITCLRI_D : LSX2RI6_VVI<0x73110000>;
 955: 
 956: def VBITSET_B : LSX3R_VVV<0x710e0000>;
 957: def VBITSET_H : LSX3R_VVV<0x710e8000>;
 958: def VBITSET_W : LSX3R_VVV<0x710f0000>;
 959: def VBITSET_D : LSX3R_VVV<0x710f8000>;
 960: def VBITSETI_B : LSX2RI3_VVI<0x73142000>;
```
- **EN**: This block declares or refines TableGen records such as `VSSRARNI_H_W`, `VSSRARNI_W_D`, `VSSRARNI_D_Q`, `VSSRLRNI_BU_H`, `VSSRLRNI_HU_W`, `VSSRLRNI_WU_D`.
- **CN**: 该代码块声明或细化了 `VSSRARNI_H_W`, `VSSRARNI_W_D`, `VSSRARNI_D_Q`, `VSSRLRNI_BU_H`, `VSSRLRNI_HU_W`, `VSSRLRNI_WU_D` 等 TableGen 记录。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: def VBITSETI_H : LSX2RI4_VVI<0x73144000>;
 962: def VBITSETI_W : LSX2RI5_VVI<0x73148000>;
 963: def VBITSETI_D : LSX2RI6_VVI<0x73150000>;
 964: 
 965: def VBITREV_B : LSX3R_VVV<0x71100000>;
 966: def VBITREV_H : LSX3R_VVV<0x71108000>;
 967: def VBITREV_W : LSX3R_VVV<0x71110000>;
 968: def VBITREV_D : LSX3R_VVV<0x71118000>;
 969: def VBITREVI_B : LSX2RI3_VVI<0x73182000>;
 970: def VBITREVI_H : LSX2RI4_VVI<0x73184000>;
 971: def VBITREVI_W : LSX2RI5_VVI<0x73188000>;
 972: def VBITREVI_D : LSX2RI6_VVI<0x73190000>;
 973: 
 974: def VFRSTP_B : LSX3R_VVVV<0x712b0000>;
 975: def VFRSTP_H : LSX3R_VVVV<0x712b8000>;
 976: def VFRSTPI_B : LSX2RI5_VVVI<0x729a0000>;
 977: def VFRSTPI_H : LSX2RI5_VVVI<0x729a8000>;
 978: 
 979: def VFADD_S : LSX3R_VVV<0x71308000>;
 980: def VFADD_D : LSX3R_VVV<0x71310000>;
 981: def VFSUB_S : LSX3R_VVV<0x71328000>;
 982: def VFSUB_D : LSX3R_VVV<0x71330000>;
 983: def VFMUL_S : LSX3R_VVV<0x71388000>;
 984: def VFMUL_D : LSX3R_VVV<0x71390000>;
 985: def VFDIV_S : LSX3R_VVV<0x713a8000>;
 986: def VFDIV_D : LSX3R_VVV<0x713b0000>;
 987: 
 988: def VFMADD_S : LSX4R_VVVV<0x09100000>;
 989: def VFMADD_D : LSX4R_VVVV<0x09200000>;
 990: def VFMSUB_S : LSX4R_VVVV<0x09500000>;
 991: def VFMSUB_D : LSX4R_VVVV<0x09600000>;
 992: def VFNMADD_S : LSX4R_VVVV<0x09900000>;
 993: def VFNMADD_D : LSX4R_VVVV<0x09a00000>;
 994: def VFNMSUB_S : LSX4R_VVVV<0x09d00000>;
 995: def VFNMSUB_D : LSX4R_VVVV<0x09e00000>;
 996: 
 997: def VFMAX_S : LSX3R_VVV<0x713c8000>;
 998: def VFMAX_D : LSX3R_VVV<0x713d0000>;
 999: def VFMIN_S : LSX3R_VVV<0x713e8000>;
1000: def VFMIN_D : LSX3R_VVV<0x713f0000>;
```
- **EN**: This block declares or refines TableGen records such as `VBITSETI_H`, `VBITSETI_W`, `VBITSETI_D`, `VBITREV_B`, `VBITREV_H`, `VBITREV_W`.
- **CN**: 该代码块声明或细化了 `VBITSETI_H`, `VBITSETI_W`, `VBITSETI_D`, `VBITREV_B`, `VBITREV_H`, `VBITREV_W` 等 TableGen 记录。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: 
1002: def VFMAXA_S : LSX3R_VVV<0x71408000>;
1003: def VFMAXA_D : LSX3R_VVV<0x71410000>;
1004: def VFMINA_S : LSX3R_VVV<0x71428000>;
1005: def VFMINA_D : LSX3R_VVV<0x71430000>;
1006: 
1007: def VFLOGB_S : LSX2R_VV<0x729cc400>;
1008: def VFLOGB_D : LSX2R_VV<0x729cc800>;
1009: 
1010: def VFCLASS_S : LSX2R_VV<0x729cd400>;
1011: def VFCLASS_D : LSX2R_VV<0x729cd800>;
1012: 
1013: def VFSQRT_S : LSX2R_VV<0x729ce400>;
1014: def VFSQRT_D : LSX2R_VV<0x729ce800>;
1015: def VFRECIP_S : LSX2R_VV<0x729cf400>;
1016: def VFRECIP_D : LSX2R_VV<0x729cf800>;
1017: def VFRSQRT_S : LSX2R_VV<0x729d0400>;
1018: def VFRSQRT_D : LSX2R_VV<0x729d0800>;
1019: def VFRECIPE_S : LSX2R_VV<0x729d1400>;
1020: def VFRECIPE_D : LSX2R_VV<0x729d1800>;
1021: def VFRSQRTE_S : LSX2R_VV<0x729d2400>;
1022: def VFRSQRTE_D : LSX2R_VV<0x729d2800>;
1023: 
1024: def VFCVTL_S_H : LSX2R_VV<0x729de800>;
1025: def VFCVTH_S_H : LSX2R_VV<0x729dec00>;
1026: def VFCVTL_D_S : LSX2R_VV<0x729df000>;
1027: def VFCVTH_D_S : LSX2R_VV<0x729df400>;
1028: def VFCVT_H_S : LSX3R_VVV<0x71460000>;
1029: def VFCVT_S_D : LSX3R_VVV<0x71468000>;
1030: 
1031: def VFRINTRNE_S : LSX2R_VV<0x729d7400>;
1032: def VFRINTRNE_D : LSX2R_VV<0x729d7800>;
1033: def VFRINTRZ_S : LSX2R_VV<0x729d6400>;
1034: def VFRINTRZ_D : LSX2R_VV<0x729d6800>;
1035: def VFRINTRP_S : LSX2R_VV<0x729d5400>;
1036: def VFRINTRP_D : LSX2R_VV<0x729d5800>;
1037: def VFRINTRM_S : LSX2R_VV<0x729d4400>;
1038: def VFRINTRM_D : LSX2R_VV<0x729d4800>;
1039: def VFRINT_S : LSX2R_VV<0x729d3400>;
1040: def VFRINT_D : LSX2R_VV<0x729d3800>;
```
- **EN**: This block declares or refines TableGen records such as `VFMAXA_S`, `VFMAXA_D`, `VFMINA_S`, `VFMINA_D`, `VFLOGB_S`, `VFLOGB_D`.
- **CN**: 该代码块声明或细化了 `VFMAXA_S`, `VFMAXA_D`, `VFMINA_S`, `VFMINA_D`, `VFLOGB_S`, `VFLOGB_D` 等 TableGen 记录。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: 
1042: def VFTINTRNE_W_S : LSX2R_VV<0x729e5000>;
1043: def VFTINTRNE_L_D : LSX2R_VV<0x729e5400>;
1044: def VFTINTRZ_W_S : LSX2R_VV<0x729e4800>;
1045: def VFTINTRZ_L_D : LSX2R_VV<0x729e4c00>;
1046: def VFTINTRP_W_S : LSX2R_VV<0x729e4000>;
1047: def VFTINTRP_L_D : LSX2R_VV<0x729e4400>;
1048: def VFTINTRM_W_S : LSX2R_VV<0x729e3800>;
1049: def VFTINTRM_L_D : LSX2R_VV<0x729e3c00>;
1050: def VFTINT_W_S : LSX2R_VV<0x729e3000>;
1051: def VFTINT_L_D : LSX2R_VV<0x729e3400>;
1052: def VFTINTRZ_WU_S : LSX2R_VV<0x729e7000>;
1053: def VFTINTRZ_LU_D : LSX2R_VV<0x729e7400>;
1054: def VFTINT_WU_S : LSX2R_VV<0x729e5800>;
1055: def VFTINT_LU_D : LSX2R_VV<0x729e5c00>;
1056: 
1057: def VFTINTRNE_W_D : LSX3R_VVV<0x714b8000>;
1058: def VFTINTRZ_W_D : LSX3R_VVV<0x714b0000>;
1059: def VFTINTRP_W_D : LSX3R_VVV<0x714a8000>;
1060: def VFTINTRM_W_D : LSX3R_VVV<0x714a0000>;
1061: def VFTINT_W_D : LSX3R_VVV<0x71498000>;
1062: 
1063: def VFTINTRNEL_L_S : LSX2R_VV<0x729ea000>;
1064: def VFTINTRNEH_L_S : LSX2R_VV<0x729ea400>;
1065: def VFTINTRZL_L_S : LSX2R_VV<0x729e9800>;
1066: def VFTINTRZH_L_S : LSX2R_VV<0x729e9c00>;
1067: def VFTINTRPL_L_S : LSX2R_VV<0x729e9000>;
1068: def VFTINTRPH_L_S : LSX2R_VV<0x729e9400>;
1069: def VFTINTRML_L_S : LSX2R_VV<0x729e8800>;
1070: def VFTINTRMH_L_S : LSX2R_VV<0x729e8c00>;
1071: def VFTINTL_L_S : LSX2R_VV<0x729e8000>;
1072: def VFTINTH_L_S : LSX2R_VV<0x729e8400>;
1073: 
1074: def VFFINT_S_W : LSX2R_VV<0x729e0000>;
1075: def VFFINT_D_L : LSX2R_VV<0x729e0800>;
1076: def VFFINT_S_WU : LSX2R_VV<0x729e0400>;
1077: def VFFINT_D_LU : LSX2R_VV<0x729e0c00>;
1078: def VFFINTL_D_W : LSX2R_VV<0x729e1000>;
1079: def VFFINTH_D_W : LSX2R_VV<0x729e1400>;
1080: def VFFINT_S_L : LSX3R_VVV<0x71480000>;
```
- **EN**: This block declares or refines TableGen records such as `VFTINTRNE_W_S`, `VFTINTRNE_L_D`, `VFTINTRZ_W_S`, `VFTINTRZ_L_D`, `VFTINTRP_W_S`, `VFTINTRP_L_D`.
- **CN**: 该代码块声明或细化了 `VFTINTRNE_W_S`, `VFTINTRNE_L_D`, `VFTINTRZ_W_S`, `VFTINTRZ_L_D`, `VFTINTRP_W_S`, `VFTINTRP_L_D` 等 TableGen 记录。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: 
1082: def VSEQ_B : LSX3R_VVV<0x70000000>;
1083: def VSEQ_H : LSX3R_VVV<0x70008000>;
1084: def VSEQ_W : LSX3R_VVV<0x70010000>;
1085: def VSEQ_D : LSX3R_VVV<0x70018000>;
1086: def VSEQI_B : LSX2RI5_VVI<0x72800000, simm5>;
1087: def VSEQI_H : LSX2RI5_VVI<0x72808000, simm5>;
1088: def VSEQI_W : LSX2RI5_VVI<0x72810000, simm5>;
1089: def VSEQI_D : LSX2RI5_VVI<0x72818000, simm5>;
1090: 
1091: def VSLE_B : LSX3R_VVV<0x70020000>;
1092: def VSLE_H : LSX3R_VVV<0x70028000>;
1093: def VSLE_W : LSX3R_VVV<0x70030000>;
1094: def VSLE_D : LSX3R_VVV<0x70038000>;
1095: def VSLEI_B : LSX2RI5_VVI<0x72820000, simm5>;
1096: def VSLEI_H : LSX2RI5_VVI<0x72828000, simm5>;
1097: def VSLEI_W : LSX2RI5_VVI<0x72830000, simm5>;
1098: def VSLEI_D : LSX2RI5_VVI<0x72838000, simm5>;
1099: 
1100: def VSLE_BU : LSX3R_VVV<0x70040000>;
1101: def VSLE_HU : LSX3R_VVV<0x70048000>;
1102: def VSLE_WU : LSX3R_VVV<0x70050000>;
1103: def VSLE_DU : LSX3R_VVV<0x70058000>;
1104: def VSLEI_BU : LSX2RI5_VVI<0x72840000>;
1105: def VSLEI_HU : LSX2RI5_VVI<0x72848000>;
1106: def VSLEI_WU : LSX2RI5_VVI<0x72850000>;
1107: def VSLEI_DU : LSX2RI5_VVI<0x72858000>;
1108: 
1109: def VSLT_B : LSX3R_VVV<0x70060000>;
1110: def VSLT_H : LSX3R_VVV<0x70068000>;
1111: def VSLT_W : LSX3R_VVV<0x70070000>;
1112: def VSLT_D : LSX3R_VVV<0x70078000>;
1113: def VSLTI_B : LSX2RI5_VVI<0x72860000, simm5>;
1114: def VSLTI_H : LSX2RI5_VVI<0x72868000, simm5>;
1115: def VSLTI_W : LSX2RI5_VVI<0x72870000, simm5>;
1116: def VSLTI_D : LSX2RI5_VVI<0x72878000, simm5>;
1117: 
1118: def VSLT_BU : LSX3R_VVV<0x70080000>;
1119: def VSLT_HU : LSX3R_VVV<0x70088000>;
1120: def VSLT_WU : LSX3R_VVV<0x70090000>;
```
- **EN**: This block declares or refines TableGen records such as `VSEQ_B`, `VSEQ_H`, `VSEQ_W`, `VSEQ_D`, `VSEQI_B`, `VSEQI_H`.
- **CN**: 该代码块声明或细化了 `VSEQ_B`, `VSEQ_H`, `VSEQ_W`, `VSEQ_D`, `VSEQI_B`, `VSEQI_H` 等 TableGen 记录。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: def VSLT_DU : LSX3R_VVV<0x70098000>;
1122: def VSLTI_BU : LSX2RI5_VVI<0x72880000>;
1123: def VSLTI_HU : LSX2RI5_VVI<0x72888000>;
1124: def VSLTI_WU : LSX2RI5_VVI<0x72890000>;
1125: def VSLTI_DU : LSX2RI5_VVI<0x72898000>;
1126: 
1127: def VFCMP_CAF_S : LSX3R_VVV<0x0c500000>;
1128: def VFCMP_SAF_S : LSX3R_VVV<0x0c508000>;
1129: def VFCMP_CLT_S : LSX3R_VVV<0x0c510000>;
1130: def VFCMP_SLT_S : LSX3R_VVV<0x0c518000>;
1131: def VFCMP_CEQ_S : LSX3R_VVV<0x0c520000>;
1132: def VFCMP_SEQ_S : LSX3R_VVV<0x0c528000>;
1133: def VFCMP_CLE_S : LSX3R_VVV<0x0c530000>;
1134: def VFCMP_SLE_S : LSX3R_VVV<0x0c538000>;
1135: def VFCMP_CUN_S : LSX3R_VVV<0x0c540000>;
1136: def VFCMP_SUN_S : LSX3R_VVV<0x0c548000>;
1137: def VFCMP_CULT_S : LSX3R_VVV<0x0c550000>;
1138: def VFCMP_SULT_S : LSX3R_VVV<0x0c558000>;
1139: def VFCMP_CUEQ_S : LSX3R_VVV<0x0c560000>;
1140: def VFCMP_SUEQ_S : LSX3R_VVV<0x0c568000>;
1141: def VFCMP_CULE_S : LSX3R_VVV<0x0c570000>;
1142: def VFCMP_SULE_S : LSX3R_VVV<0x0c578000>;
1143: def VFCMP_CNE_S : LSX3R_VVV<0x0c580000>;
1144: def VFCMP_SNE_S : LSX3R_VVV<0x0c588000>;
1145: def VFCMP_COR_S : LSX3R_VVV<0x0c5a0000>;
1146: def VFCMP_SOR_S : LSX3R_VVV<0x0c5a8000>;
1147: def VFCMP_CUNE_S : LSX3R_VVV<0x0c5c0000>;
1148: def VFCMP_SUNE_S : LSX3R_VVV<0x0c5c8000>;
1149: 
1150: def VFCMP_CAF_D : LSX3R_VVV<0x0c600000>;
1151: def VFCMP_SAF_D : LSX3R_VVV<0x0c608000>;
1152: def VFCMP_CLT_D : LSX3R_VVV<0x0c610000>;
1153: def VFCMP_SLT_D : LSX3R_VVV<0x0c618000>;
1154: def VFCMP_CEQ_D : LSX3R_VVV<0x0c620000>;
1155: def VFCMP_SEQ_D : LSX3R_VVV<0x0c628000>;
1156: def VFCMP_CLE_D : LSX3R_VVV<0x0c630000>;
1157: def VFCMP_SLE_D : LSX3R_VVV<0x0c638000>;
1158: def VFCMP_CUN_D : LSX3R_VVV<0x0c640000>;
1159: def VFCMP_SUN_D : LSX3R_VVV<0x0c648000>;
1160: def VFCMP_CULT_D : LSX3R_VVV<0x0c650000>;
```
- **EN**: This block declares or refines TableGen records such as `VSLT_DU`, `VSLTI_BU`, `VSLTI_HU`, `VSLTI_WU`, `VSLTI_DU`, `VFCMP_CAF_S`.
- **CN**: 该代码块声明或细化了 `VSLT_DU`, `VSLTI_BU`, `VSLTI_HU`, `VSLTI_WU`, `VSLTI_DU`, `VFCMP_CAF_S` 等 TableGen 记录。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: def VFCMP_SULT_D : LSX3R_VVV<0x0c658000>;
1162: def VFCMP_CUEQ_D : LSX3R_VVV<0x0c660000>;
1163: def VFCMP_SUEQ_D : LSX3R_VVV<0x0c668000>;
1164: def VFCMP_CULE_D : LSX3R_VVV<0x0c670000>;
1165: def VFCMP_SULE_D : LSX3R_VVV<0x0c678000>;
1166: def VFCMP_CNE_D : LSX3R_VVV<0x0c680000>;
1167: def VFCMP_SNE_D : LSX3R_VVV<0x0c688000>;
1168: def VFCMP_COR_D : LSX3R_VVV<0x0c6a0000>;
1169: def VFCMP_SOR_D : LSX3R_VVV<0x0c6a8000>;
1170: def VFCMP_CUNE_D : LSX3R_VVV<0x0c6c0000>;
1171: def VFCMP_SUNE_D : LSX3R_VVV<0x0c6c8000>;
1172: 
1173: def VBITSEL_V : LSX4R_VVVV<0x0d100000>;
1174: 
1175: def VBITSELI_B : LSX2RI8_VVVI<0x73c40000>;
1176: 
1177: def VSETEQZ_V : LSX2R_CV<0x729c9800>;
1178: def VSETNEZ_V : LSX2R_CV<0x729c9c00>;
1179: def VSETANYEQZ_B : LSX2R_CV<0x729ca000>;
1180: def VSETANYEQZ_H : LSX2R_CV<0x729ca400>;
1181: def VSETANYEQZ_W : LSX2R_CV<0x729ca800>;
1182: def VSETANYEQZ_D : LSX2R_CV<0x729cac00>;
1183: def VSETALLNEZ_B : LSX2R_CV<0x729cb000>;
1184: def VSETALLNEZ_H : LSX2R_CV<0x729cb400>;
1185: def VSETALLNEZ_W : LSX2R_CV<0x729cb800>;
1186: def VSETALLNEZ_D : LSX2R_CV<0x729cbc00>;
1187: 
1188: def VINSGR2VR_B : LSX2RI4_VVRI<0x72eb8000>;
1189: def VINSGR2VR_H : LSX2RI3_VVRI<0x72ebc000>;
1190: def VINSGR2VR_W : LSX2RI2_VVRI<0x72ebe000>;
1191: def VINSGR2VR_D : LSX2RI1_VVRI<0x72ebf000>;
1192: def VPICKVE2GR_B : LSX2RI4_RVI<0x72ef8000>;
1193: def VPICKVE2GR_H : LSX2RI3_RVI<0x72efc000>;
1194: def VPICKVE2GR_W : LSX2RI2_RVI<0x72efe000>;
1195: def VPICKVE2GR_D : LSX2RI1_RVI<0x72eff000>;
1196: def VPICKVE2GR_BU : LSX2RI4_RVI<0x72f38000>;
1197: def VPICKVE2GR_HU : LSX2RI3_RVI<0x72f3c000>;
1198: def VPICKVE2GR_WU : LSX2RI2_RVI<0x72f3e000>;
1199: def VPICKVE2GR_DU : LSX2RI1_RVI<0x72f3f000>;
1200: 
```
- **EN**: This block declares or refines TableGen records such as `VFCMP_SULT_D`, `VFCMP_CUEQ_D`, `VFCMP_SUEQ_D`, `VFCMP_CULE_D`, `VFCMP_SULE_D`, `VFCMP_CNE_D`.
- **CN**: 该代码块声明或细化了 `VFCMP_SULT_D`, `VFCMP_CUEQ_D`, `VFCMP_SUEQ_D`, `VFCMP_CULE_D`, `VFCMP_SULE_D`, `VFCMP_CNE_D` 等 TableGen 记录。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: let isReMaterializable = 1 in {
1202: def VREPLGR2VR_B : LSX2R_VR<0x729f0000>;
1203: def VREPLGR2VR_H : LSX2R_VR<0x729f0400>;
1204: def VREPLGR2VR_W : LSX2R_VR<0x729f0800>;
1205: def VREPLGR2VR_D : LSX2R_VR<0x729f0c00>;
1206: }
1207: 
1208: def VREPLVE_B : LSX3R_VVR<0x71220000>;
1209: def VREPLVE_H : LSX3R_VVR<0x71228000>;
1210: def VREPLVE_W : LSX3R_VVR<0x71230000>;
1211: def VREPLVE_D : LSX3R_VVR<0x71238000>;
1212: def VREPLVEI_B : LSX2RI4_VVI<0x72f78000>;
1213: def VREPLVEI_H : LSX2RI3_VVI<0x72f7c000>;
1214: def VREPLVEI_W : LSX2RI2_VVI<0x72f7e000>;
1215: def VREPLVEI_D : LSX2RI1_VVI<0x72f7f000>;
1216: 
1217: def VBSLL_V : LSX2RI5_VVI<0x728e0000>;
1218: def VBSRL_V : LSX2RI5_VVI<0x728e8000>;
1219: 
1220: def VPACKEV_B : LSX3R_VVV<0x71160000>;
1221: def VPACKEV_H : LSX3R_VVV<0x71168000>;
1222: def VPACKEV_W : LSX3R_VVV<0x71170000>;
1223: def VPACKEV_D : LSX3R_VVV<0x71178000>;
1224: def VPACKOD_B : LSX3R_VVV<0x71180000>;
1225: def VPACKOD_H : LSX3R_VVV<0x71188000>;
1226: def VPACKOD_W : LSX3R_VVV<0x71190000>;
1227: def VPACKOD_D : LSX3R_VVV<0x71198000>;
1228: 
1229: def VPICKEV_B : LSX3R_VVV<0x711e0000>;
1230: def VPICKEV_H : LSX3R_VVV<0x711e8000>;
1231: def VPICKEV_W : LSX3R_VVV<0x711f0000>;
1232: def VPICKEV_D : LSX3R_VVV<0x711f8000>;
1233: def VPICKOD_B : LSX3R_VVV<0x71200000>;
1234: def VPICKOD_H : LSX3R_VVV<0x71208000>;
1235: def VPICKOD_W : LSX3R_VVV<0x71210000>;
1236: def VPICKOD_D : LSX3R_VVV<0x71218000>;
1237: 
1238: def VILVL_B : LSX3R_VVV<0x711a0000>;
1239: def VILVL_H : LSX3R_VVV<0x711a8000>;
1240: def VILVL_W : LSX3R_VVV<0x711b0000>;
```
- **EN**: This block declares or refines TableGen records such as `VREPLGR2VR_B`, `VREPLGR2VR_H`, `VREPLGR2VR_W`, `VREPLGR2VR_D`, `VREPLVE_B`, `VREPLVE_H`.
- **CN**: 该代码块声明或细化了 `VREPLGR2VR_B`, `VREPLGR2VR_H`, `VREPLGR2VR_W`, `VREPLGR2VR_D`, `VREPLVE_B`, `VREPLVE_H` 等 TableGen 记录。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: def VILVL_D : LSX3R_VVV<0x711b8000>;
1242: def VILVH_B : LSX3R_VVV<0x711c0000>;
1243: def VILVH_H : LSX3R_VVV<0x711c8000>;
1244: def VILVH_W : LSX3R_VVV<0x711d0000>;
1245: def VILVH_D : LSX3R_VVV<0x711d8000>;
1246: 
1247: def VSHUF_B : LSX4R_VVVV<0x0d500000>;
1248: 
1249: def VSHUF_H : LSX3R_VVVV<0x717a8000>;
1250: def VSHUF_W : LSX3R_VVVV<0x717b0000>;
1251: def VSHUF_D : LSX3R_VVVV<0x717b8000>;
1252: 
1253: def VSHUF4I_B : LSX2RI8_VVI<0x73900000>;
1254: def VSHUF4I_H : LSX2RI8_VVI<0x73940000>;
1255: def VSHUF4I_W : LSX2RI8_VVI<0x73980000>;
1256: def VSHUF4I_D : LSX2RI8_VVVI<0x739c0000>;
1257: 
1258: def VPERMI_W : LSX2RI8_VVVI<0x73e40000>;
1259: 
1260: def VEXTRINS_D : LSX2RI8_VVVI<0x73800000>;
1261: def VEXTRINS_W : LSX2RI8_VVVI<0x73840000>;
1262: def VEXTRINS_H : LSX2RI8_VVVI<0x73880000>;
1263: def VEXTRINS_B : LSX2RI8_VVVI<0x738c0000>;
1264: } // mayLoad = 0, mayStore = 0
1265: 
1266: let mayLoad = 1, mayStore = 0 in {
1267: def VLD : LSX2RI12_Load<0x2c000000>;
1268: def VLDX : LSX3R_Load<0x38400000>;
1269: 
1270: def VLDREPL_B : LSX2RI12_Load<0x30800000>;
1271: def VLDREPL_H : LSX2RI11_Load<0x30400000>;
1272: def VLDREPL_W : LSX2RI10_Load<0x30200000>;
1273: def VLDREPL_D : LSX2RI9_Load<0x30100000>;
1274: } // mayLoad = 1, mayStore = 0
1275: 
1276: let mayLoad = 0, mayStore = 1 in {
1277: def VST : LSX2RI12_Store<0x2c400000>;
1278: def VSTX : LSX3R_Store<0x38440000>;
1279: 
1280: def VSTELM_B : LSX2RI8I4_VRII<0x31800000>;
```
- **EN**: This block declares or refines TableGen records such as `VILVL_D`, `VILVH_B`, `VILVH_H`, `VILVH_W`, `VILVH_D`, `VSHUF_B`.
- **CN**: 该代码块声明或细化了 `VILVL_D`, `VILVH_B`, `VILVH_H`, `VILVH_W`, `VILVH_D`, `VSHUF_B` 等 TableGen 记录。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: def VSTELM_H : LSX2RI8I3_VRII<0x31400000, simm8_lsl1>;
1282: def VSTELM_W : LSX2RI8I2_VRII<0x31200000, simm8_lsl2>;
1283: def VSTELM_D : LSX2RI8I1_VRII<0x31100000, simm8_lsl3>;
1284: } // mayLoad = 0, mayStore = 1
1285: 
1286: } // hasSideEffects = 0, Predicates = [HasExtLSX]
1287: 
1288: /// Pseudo-instructions
1289: 
1290: let Predicates = [HasExtLSX] in {
1291: 
1292: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCodeGenOnly = 0,
1293:     isAsmParserOnly = 1 in {
1294: def PseudoVREPLI_B : Pseudo<(outs LSX128:$vd), (ins simm10:$imm), [],
1295:                             "vrepli.b", "$vd, $imm">;
1296: def PseudoVREPLI_H : Pseudo<(outs LSX128:$vd), (ins simm10:$imm), [],
1297:                             "vrepli.h", "$vd, $imm">;
1298: def PseudoVREPLI_W : Pseudo<(outs LSX128:$vd), (ins simm10:$imm), [],
1299:                             "vrepli.w", "$vd, $imm">;
1300: def PseudoVREPLI_D : Pseudo<(outs LSX128:$vd), (ins simm10:$imm), [],
1301:                             "vrepli.d", "$vd, $imm">;
1302: }
1303: 
1304: def PseudoVBNZ_B : VecCond<loongarch_vall_nonzero, v16i8>;
1305: def PseudoVBNZ_H : VecCond<loongarch_vall_nonzero, v8i16>;
1306: def PseudoVBNZ_W : VecCond<loongarch_vall_nonzero, v4i32>;
1307: def PseudoVBNZ_D : VecCond<loongarch_vall_nonzero, v2i64>;
1308: def PseudoVBNZ : VecCond<loongarch_vany_nonzero, v16i8>;
1309: 
1310: def PseudoVBZ_B : VecCond<loongarch_vall_zero, v16i8>;
1311: def PseudoVBZ_H : VecCond<loongarch_vall_zero, v8i16>;
1312: def PseudoVBZ_W : VecCond<loongarch_vall_zero, v4i32>;
1313: def PseudoVBZ_D : VecCond<loongarch_vall_zero, v2i64>;
1314: def PseudoVBZ : VecCond<loongarch_vany_zero, v16i8>;
1315: 
1316: let usesCustomInserter = 1 in {
1317: def PseudoCTPOP_B : Pseudo<(outs GPR:$rd), (ins GPR:$rj),
1318:                            [(set GPR:$rd, (ctpop (and GPR:$rj, 255)))]>;
1319: def PseudoCTPOP_H : Pseudo<(outs GPR:$rd), (ins GPR:$rj),
1320:                            [(set GPR:$rd, (ctpop (loongarch_bstrpick GRLenVT:$rj,
```
- **EN**: This block declares or refines TableGen records such as `VSTELM_H`, `VSTELM_W`, `VSTELM_D`, `PseudoVREPLI_B`, `PseudoVREPLI_H`, `PseudoVREPLI_W`.
- **CN**: 该代码块声明或细化了 `VSTELM_H`, `VSTELM_W`, `VSTELM_D`, `PseudoVREPLI_B`, `PseudoVREPLI_H`, `PseudoVREPLI_W` 等 TableGen 记录。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321:                                               (GRLenVT 15), (GRLenVT 0))))]>;
1322: let Predicates = [IsLA32] in {
1323: def PseudoCTPOP_H_LA32 : Pseudo<(outs GPR:$rd), (ins GPR:$rj),
1324:                                 [(set GPR:$rd, (ctpop (and GPR:$rj, 65535)))]>;
1325: def PseudoCTPOP_W_LA32 : Pseudo<(outs GPR:$rd), (ins GPR:$rj),
1326:                                 [(set GPR:$rd, (ctpop GPR:$rj))]>;
1327: } // Predicates = [IsLA32]
1328: 
1329: let Predicates = [IsLA64] in {
1330: def PseudoCTPOP_W : Pseudo<(outs GPR:$rd), (ins GPR:$rj),
1331:                            [(set GPR:$rd, (ctpop (loongarch_bstrpick i64:$rj,
1332:                                               (i64 31), (i64 0))))]>;
1333: def PseudoCTPOP_D : Pseudo<(outs GPR:$rd), (ins GPR:$rj),
1334:                            [(set GPR:$rd, (ctpop GPR:$rj))]>;
1335: } // Predicates = [IsLA64]
1336: } // usesCustomInserter = 1
1337: 
1338: let usesCustomInserter = 1, hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
1339: def PseudoVMSKLTZ_B : Pseudo<(outs GPR:$rd), (ins LSX128:$vj)>;
1340: def PseudoVMSKLTZ_H : Pseudo<(outs GPR:$rd), (ins LSX128:$vj)>;
1341: def PseudoVMSKLTZ_W : Pseudo<(outs GPR:$rd), (ins LSX128:$vj)>;
1342: def PseudoVMSKLTZ_D : Pseudo<(outs GPR:$rd), (ins LSX128:$vj)>;
1343: def PseudoVMSKGEZ_B : Pseudo<(outs GPR:$rd), (ins LSX128:$vj)>;
1344: def PseudoVMSKEQZ_B : Pseudo<(outs GPR:$rd), (ins LSX128:$vj)>;
1345: def PseudoVMSKNEZ_B : Pseudo<(outs GPR:$rd), (ins LSX128:$vj)>;
1346: } // usesCustomInserter = 1, hasSideEffects = 0, mayLoad = 0, mayStore = 0
1347: 
1348: } // Predicates = [HasExtLSX]
1349: 
1350: multiclass PatVr<SDPatternOperator OpNode, string Inst> {
1351:   def : Pat<(v16i8 (OpNode (v16i8 LSX128:$vj))),
1352:             (!cast<LAInst>(Inst#"_B") LSX128:$vj)>;
1353:   def : Pat<(v8i16 (OpNode (v8i16 LSX128:$vj))),
1354:             (!cast<LAInst>(Inst#"_H") LSX128:$vj)>;
1355:   def : Pat<(v4i32 (OpNode (v4i32 LSX128:$vj))),
1356:             (!cast<LAInst>(Inst#"_W") LSX128:$vj)>;
1357:   def : Pat<(v2i64 (OpNode (v2i64 LSX128:$vj))),
1358:             (!cast<LAInst>(Inst#"_D") LSX128:$vj)>;
1359: }
1360: 
```
- **EN**: This block declares or refines TableGen records such as `PseudoCTPOP_H_LA32`, `PseudoCTPOP_W_LA32`, `PseudoCTPOP_W`, `PseudoCTPOP_D`, `PseudoVMSKLTZ_B`, `PseudoVMSKLTZ_H`.
- **CN**: 该代码块声明或细化了 `PseudoCTPOP_H_LA32`, `PseudoCTPOP_W_LA32`, `PseudoCTPOP_W`, `PseudoCTPOP_D`, `PseudoVMSKLTZ_B`, `PseudoVMSKLTZ_H` 等 TableGen 记录。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361: multiclass PatVrF<SDPatternOperator OpNode, string Inst> {
1362:   def : Pat<(v4f32 (OpNode (v4f32 LSX128:$vj))),
1363:             (!cast<LAInst>(Inst#"_S") LSX128:$vj)>;
1364:   def : Pat<(v2f64 (OpNode (v2f64 LSX128:$vj))),
1365:             (!cast<LAInst>(Inst#"_D") LSX128:$vj)>;
1366: }
1367: 
1368: multiclass PatVrVr<SDPatternOperator OpNode, string Inst> {
1369:   def : Pat<(OpNode (v16i8 LSX128:$vj), (v16i8 LSX128:$vk)),
1370:             (!cast<LAInst>(Inst#"_B") LSX128:$vj, LSX128:$vk)>;
1371:   def : Pat<(OpNode (v8i16 LSX128:$vj), (v8i16 LSX128:$vk)),
1372:             (!cast<LAInst>(Inst#"_H") LSX128:$vj, LSX128:$vk)>;
1373:   def : Pat<(OpNode (v4i32 LSX128:$vj), (v4i32 LSX128:$vk)),
1374:             (!cast<LAInst>(Inst#"_W") LSX128:$vj, LSX128:$vk)>;
1375:   def : Pat<(OpNode (v2i64 LSX128:$vj), (v2i64 LSX128:$vk)),
1376:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, LSX128:$vk)>;
1377: }
1378: 
1379: multiclass PatVrVrF<SDPatternOperator OpNode, string Inst> {
1380:   def : Pat<(OpNode (v4f32 LSX128:$vj), (v4f32 LSX128:$vk)),
1381:             (!cast<LAInst>(Inst#"_S") LSX128:$vj, LSX128:$vk)>;
1382:   def : Pat<(OpNode (v2f64 LSX128:$vj), (v2f64 LSX128:$vk)),
1383:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, LSX128:$vk)>;
1384: }
1385: 
1386: multiclass PatVrVrU<SDPatternOperator OpNode, string Inst> {
1387:   def : Pat<(OpNode (v16i8 LSX128:$vj), (v16i8 LSX128:$vk)),
1388:             (!cast<LAInst>(Inst#"_BU") LSX128:$vj, LSX128:$vk)>;
1389:   def : Pat<(OpNode (v8i16 LSX128:$vj), (v8i16 LSX128:$vk)),
1390:             (!cast<LAInst>(Inst#"_HU") LSX128:$vj, LSX128:$vk)>;
1391:   def : Pat<(OpNode (v4i32 LSX128:$vj), (v4i32 LSX128:$vk)),
1392:             (!cast<LAInst>(Inst#"_WU") LSX128:$vj, LSX128:$vk)>;
1393:   def : Pat<(OpNode (v2i64 LSX128:$vj), (v2i64 LSX128:$vk)),
1394:             (!cast<LAInst>(Inst#"_DU") LSX128:$vj, LSX128:$vk)>;
1395: }
1396: 
1397: multiclass PatVrSimm5<SDPatternOperator OpNode, string Inst> {
1398:   def : Pat<(OpNode (v16i8 LSX128:$vj), (v16i8 (SplatPat_simm5 simm5:$imm))),
1399:             (!cast<LAInst>(Inst#"_B") LSX128:$vj, simm5:$imm)>;
1400:   def : Pat<(OpNode (v8i16 LSX128:$vj), (v8i16 (SplatPat_simm5 simm5:$imm))),
```
- **EN**: This block declares or refines TableGen records such as `PatVrF`, `PatVrVr`, `PatVrVrF`, `PatVrVrU`, `PatVrSimm5`.
- **CN**: 该代码块声明或细化了 `PatVrF`, `PatVrVr`, `PatVrVrF`, `PatVrVrU`, `PatVrSimm5` 等 TableGen 记录。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401:             (!cast<LAInst>(Inst#"_H") LSX128:$vj, simm5:$imm)>;
1402:   def : Pat<(OpNode (v4i32 LSX128:$vj), (v4i32 (SplatPat_simm5 simm5:$imm))),
1403:             (!cast<LAInst>(Inst#"_W") LSX128:$vj, simm5:$imm)>;
1404:   def : Pat<(OpNode (v2i64 LSX128:$vj), (v2i64 (SplatPat_simm5 simm5:$imm))),
1405:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, simm5:$imm)>;
1406: }
1407: 
1408: multiclass PatVrUimm5<SDPatternOperator OpNode, string Inst> {
1409:   def : Pat<(OpNode (v16i8 LSX128:$vj), (v16i8 (SplatPat_uimm5 uimm5:$imm))),
1410:             (!cast<LAInst>(Inst#"_BU") LSX128:$vj, uimm5:$imm)>;
1411:   def : Pat<(OpNode (v8i16 LSX128:$vj), (v8i16 (SplatPat_uimm5 uimm5:$imm))),
1412:             (!cast<LAInst>(Inst#"_HU") LSX128:$vj, uimm5:$imm)>;
1413:   def : Pat<(OpNode (v4i32 LSX128:$vj), (v4i32 (SplatPat_uimm5 uimm5:$imm))),
1414:             (!cast<LAInst>(Inst#"_WU") LSX128:$vj, uimm5:$imm)>;
1415:   def : Pat<(OpNode (v2i64 LSX128:$vj), (v2i64 (SplatPat_uimm5 uimm5:$imm))),
1416:             (!cast<LAInst>(Inst#"_DU") LSX128:$vj, uimm5:$imm)>;
1417: }
1418: 
1419: multiclass PatVrNimm5<SDPatternOperator OpNode, string Inst> {
1420:   def : Pat<(OpNode (v16i8 LSX128:$vj), (v16i8 (SplatPat_nimm5 uimm5:$imm))),
1421:             (!cast<LAInst>(Inst#"_BU") LSX128:$vj, uimm5:$imm)>;
1422:   def : Pat<(OpNode (v8i16 LSX128:$vj), (v8i16 (SplatPat_nimm5 uimm5:$imm))),
1423:             (!cast<LAInst>(Inst#"_HU") LSX128:$vj, uimm5:$imm)>;
1424:   def : Pat<(OpNode (v4i32 LSX128:$vj), (v4i32 (SplatPat_nimm5 uimm5:$imm))),
1425:             (!cast<LAInst>(Inst#"_WU") LSX128:$vj, uimm5:$imm)>;
1426:   def : Pat<(OpNode (v2i64 LSX128:$vj), (v2i64 (SplatPat_nimm5 uimm5:$imm))),
1427:             (!cast<LAInst>(Inst#"_DU") LSX128:$vj, uimm5:$imm)>;
1428: }
1429: 
1430: multiclass PatVrVrVr<SDPatternOperator OpNode, string Inst> {
1431:   def : Pat<(OpNode (v16i8 LSX128:$vd), (v16i8 LSX128:$vj), (v16i8 LSX128:$vk)),
1432:             (!cast<LAInst>(Inst#"_B") LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
1433:   def : Pat<(OpNode (v8i16 LSX128:$vd), (v8i16 LSX128:$vj), (v8i16 LSX128:$vk)),
1434:             (!cast<LAInst>(Inst#"_H") LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
1435:   def : Pat<(OpNode (v4i32 LSX128:$vd), (v4i32 LSX128:$vj), (v4i32 LSX128:$vk)),
1436:             (!cast<LAInst>(Inst#"_W") LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
1437:   def : Pat<(OpNode (v2i64 LSX128:$vd), (v2i64 LSX128:$vj), (v2i64 LSX128:$vk)),
1438:             (!cast<LAInst>(Inst#"_D") LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
1439: }
1440: 
```
- **EN**: This block declares or refines TableGen records such as `PatVrUimm5`, `PatVrNimm5`, `PatVrVrVr`.
- **CN**: 该代码块声明或细化了 `PatVrUimm5`, `PatVrNimm5`, `PatVrVrVr` 等 TableGen 记录。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: multiclass PatVrVrW<SDPatternOperator OpNode, string Inst> {
1442:   def : Pat<(OpNode(v16i8 LSX128:$vj), (v16i8 LSX128:$vk)),
1443:             (!cast<LAInst>(Inst#"_H_B") LSX128:$vj, LSX128:$vk)>;
1444:   def : Pat<(OpNode(v8i16 LSX128:$vj), (v8i16 LSX128:$vk)),
1445:             (!cast<LAInst>(Inst#"_W_H") LSX128:$vj, LSX128:$vk)>;
1446:   def : Pat<(OpNode(v4i32 LSX128:$vj), (v4i32 LSX128:$vk)),
1447:             (!cast<LAInst>(Inst#"_D_W") LSX128:$vj, LSX128:$vk)>;
1448:   def : Pat<(OpNode(v2i64 LSX128:$vj), (v2i64 LSX128:$vk)),
1449:             (!cast<LAInst>(Inst#"_Q_D") LSX128:$vj, LSX128:$vk)>;
1450: }
1451: 
1452: multiclass PatShiftVrVr<SDPatternOperator OpNode, string Inst> {
1453:   def : Pat<(OpNode (v16i8 LSX128:$vj), (and vsplati8_imm_eq_7,
1454:                                              (v16i8 LSX128:$vk))),
1455:             (!cast<LAInst>(Inst#"_B") LSX128:$vj, LSX128:$vk)>;
1456:   def : Pat<(OpNode (v8i16 LSX128:$vj), (and vsplati16_imm_eq_15,
1457:                                              (v8i16 LSX128:$vk))),
1458:             (!cast<LAInst>(Inst#"_H") LSX128:$vj, LSX128:$vk)>;
1459:   def : Pat<(OpNode (v4i32 LSX128:$vj), (and vsplati32_imm_eq_31,
1460:                                              (v4i32 LSX128:$vk))),
1461:             (!cast<LAInst>(Inst#"_W") LSX128:$vj, LSX128:$vk)>;
1462:   def : Pat<(OpNode (v2i64 LSX128:$vj), (and vsplati64_imm_eq_63,
1463:                                              (v2i64 LSX128:$vk))),
1464:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, LSX128:$vk)>;
1465: }
1466: 
1467: multiclass PatShiftVrSplatUimm<SDPatternOperator OpNode, string Inst> {
1468:   def : Pat<(OpNode (v16i8 LSX128:$vj), (v16i8 (SplatPat_uimm3 uimm3:$imm))),
1469:             (!cast<LAInst>(Inst#"_B") LSX128:$vj, uimm3:$imm)>;
1470:   def : Pat<(OpNode (v8i16 LSX128:$vj), (v8i16 (SplatPat_uimm4 uimm4:$imm))),
1471:             (!cast<LAInst>(Inst#"_H") LSX128:$vj, uimm4:$imm)>;
1472:   def : Pat<(OpNode (v4i32 LSX128:$vj), (v4i32 (SplatPat_uimm5 uimm5:$imm))),
1473:             (!cast<LAInst>(Inst#"_W") LSX128:$vj, uimm5:$imm)>;
1474:   def : Pat<(OpNode (v2i64 LSX128:$vj), (v2i64 (SplatPat_uimm6 uimm6:$imm))),
1475:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, uimm6:$imm)>;
1476: }
1477: 
1478: multiclass PatShiftVrUimm<SDPatternOperator OpNode, string Inst> {
1479:   def : Pat<(OpNode(v16i8 LSX128:$vj), uimm3:$imm),
1480:             (!cast<LAInst>(Inst#"_B") LSX128:$vj, uimm3:$imm)>;
```
- **EN**: This block declares or refines TableGen records such as `PatVrVrW`, `PatShiftVrVr`, `PatShiftVrSplatUimm`, `PatShiftVrUimm`.
- **CN**: 该代码块声明或细化了 `PatVrVrW`, `PatShiftVrVr`, `PatShiftVrSplatUimm`, `PatShiftVrUimm` 等 TableGen 记录。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481:   def : Pat<(OpNode(v8i16 LSX128:$vj), uimm4:$imm),
1482:             (!cast<LAInst>(Inst#"_H") LSX128:$vj, uimm4:$imm)>;
1483:   def : Pat<(OpNode(v4i32 LSX128:$vj), uimm5:$imm),
1484:             (!cast<LAInst>(Inst#"_W") LSX128:$vj, uimm5:$imm)>;
1485:   def : Pat<(OpNode(v2i64 LSX128:$vj), uimm6:$imm),
1486:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, uimm6:$imm)>;
1487: }
1488: 
1489: multiclass PatCCVrSimm5<CondCode CC, string Inst> {
1490:   def : Pat<(v16i8 (setcc (v16i8 LSX128:$vj),
1491:                           (v16i8 (SplatPat_simm5 simm5:$imm)), CC)),
1492:             (!cast<LAInst>(Inst#"_B") LSX128:$vj, simm5:$imm)>;
1493:   def : Pat<(v8i16 (setcc (v8i16 LSX128:$vj),
1494:                           (v8i16 (SplatPat_simm5 simm5:$imm)), CC)),
1495:             (!cast<LAInst>(Inst#"_H") LSX128:$vj, simm5:$imm)>;
1496:   def : Pat<(v4i32 (setcc (v4i32 LSX128:$vj),
1497:                           (v4i32 (SplatPat_simm5 simm5:$imm)), CC)),
1498:             (!cast<LAInst>(Inst#"_W") LSX128:$vj, simm5:$imm)>;
1499:   def : Pat<(v2i64 (setcc (v2i64 LSX128:$vj),
1500:                           (v2i64 (SplatPat_simm5 simm5:$imm)), CC)),
1501:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, simm5:$imm)>;
1502: }
1503: 
1504: multiclass PatCCVrUimm5<CondCode CC, string Inst> {
1505:   def : Pat<(v16i8 (setcc (v16i8 LSX128:$vj),
1506:                           (v16i8 (SplatPat_uimm5 uimm5:$imm)), CC)),
1507:             (!cast<LAInst>(Inst#"_BU") LSX128:$vj, uimm5:$imm)>;
1508:   def : Pat<(v8i16 (setcc (v8i16 LSX128:$vj),
1509:                           (v8i16 (SplatPat_uimm5 uimm5:$imm)), CC)),
1510:             (!cast<LAInst>(Inst#"_HU") LSX128:$vj, uimm5:$imm)>;
1511:   def : Pat<(v4i32 (setcc (v4i32 LSX128:$vj),
1512:                           (v4i32 (SplatPat_uimm5 uimm5:$imm)), CC)),
1513:             (!cast<LAInst>(Inst#"_WU") LSX128:$vj, uimm5:$imm)>;
1514:   def : Pat<(v2i64 (setcc (v2i64 LSX128:$vj),
1515:                           (v2i64 (SplatPat_uimm5 uimm5:$imm)), CC)),
1516:             (!cast<LAInst>(Inst#"_DU") LSX128:$vj, uimm5:$imm)>;
1517: }
1518: 
1519: multiclass PatCCVrVr<CondCode CC, string Inst> {
1520:   def : Pat<(v16i8 (setcc (v16i8 LSX128:$vj), (v16i8 LSX128:$vk), CC)),
```
- **EN**: This block declares or refines TableGen records such as `PatCCVrSimm5`, `PatCCVrUimm5`, `PatCCVrVr`.
- **CN**: 该代码块声明或细化了 `PatCCVrSimm5`, `PatCCVrUimm5`, `PatCCVrVr` 等 TableGen 记录。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521:             (!cast<LAInst>(Inst#"_B") LSX128:$vj, LSX128:$vk)>;
1522:   def : Pat<(v8i16 (setcc (v8i16 LSX128:$vj), (v8i16 LSX128:$vk), CC)),
1523:             (!cast<LAInst>(Inst#"_H") LSX128:$vj, LSX128:$vk)>;
1524:   def : Pat<(v4i32 (setcc (v4i32 LSX128:$vj), (v4i32 LSX128:$vk), CC)),
1525:             (!cast<LAInst>(Inst#"_W") LSX128:$vj, LSX128:$vk)>;
1526:   def : Pat<(v2i64 (setcc (v2i64 LSX128:$vj), (v2i64 LSX128:$vk), CC)),
1527:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, LSX128:$vk)>;
1528: }
1529: 
1530: multiclass PatCCVrVrU<CondCode CC, string Inst> {
1531:   def : Pat<(v16i8 (setcc (v16i8 LSX128:$vj), (v16i8 LSX128:$vk), CC)),
1532:             (!cast<LAInst>(Inst#"_BU") LSX128:$vj, LSX128:$vk)>;
1533:   def : Pat<(v8i16 (setcc (v8i16 LSX128:$vj), (v8i16 LSX128:$vk), CC)),
1534:             (!cast<LAInst>(Inst#"_HU") LSX128:$vj, LSX128:$vk)>;
1535:   def : Pat<(v4i32 (setcc (v4i32 LSX128:$vj), (v4i32 LSX128:$vk), CC)),
1536:             (!cast<LAInst>(Inst#"_WU") LSX128:$vj, LSX128:$vk)>;
1537:   def : Pat<(v2i64 (setcc (v2i64 LSX128:$vj), (v2i64 LSX128:$vk), CC)),
1538:             (!cast<LAInst>(Inst#"_DU") LSX128:$vj, LSX128:$vk)>;
1539: }
1540: 
1541: multiclass PatCCVrVrF<CondCode CC, string Inst> {
1542:   def : Pat<(v4i32 (setcc (v4f32 LSX128:$vj), (v4f32 LSX128:$vk), CC)),
1543:             (!cast<LAInst>(Inst#"_S") LSX128:$vj, LSX128:$vk)>;
1544:   def : Pat<(v2i64 (setcc (v2f64 LSX128:$vj), (v2f64 LSX128:$vk), CC)),
1545:             (!cast<LAInst>(Inst#"_D") LSX128:$vj, LSX128:$vk)>;
1546: }
1547: 
1548: multiclass VldreplPat<ValueType vt, LAInst Inst, Operand ImmOpnd> {
1549:   def : Pat<(vt(loongarch_vldrepl BaseAddr:$rj)), (Inst BaseAddr:$rj, 0)>;
1550:   def : Pat<(vt(loongarch_vldrepl(AddrConstant GPR:$rj, ImmOpnd:$imm))),
1551:             (Inst GPR:$rj, ImmOpnd:$imm)>;
1552:   def : Pat<(vt(loongarch_vldrepl(AddLike BaseAddr:$rj, ImmOpnd:$imm))),
1553:             (Inst BaseAddr:$rj, ImmOpnd:$imm)>;
1554: }
1555: 
1556: multiclass VstelmPat<PatFrag StoreOp, ValueType vt, LAInst Inst,
1557:                      Operand ImmOpnd, Operand IdxOpnd, ValueType elt = GRLenVT> {
1558:   def : Pat<(StoreOp(elt(vector_extract vt:$vd, IdxOpnd:$idx)), BaseAddr:$rj),
1559:             (Inst vt:$vd, BaseAddr:$rj, 0, IdxOpnd:$idx)>;
1560: 
```
- **EN**: This block declares or refines TableGen records such as `PatCCVrVrU`, `PatCCVrVrF`, `VldreplPat`, `VstelmPat`.
- **CN**: 该代码块声明或细化了 `PatCCVrVrU`, `PatCCVrVrF`, `VldreplPat`, `VstelmPat` 等 TableGen 记录。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561:   def : Pat<(StoreOp(elt(vector_extract vt:$vd, IdxOpnd:$idx)),
1562:                 (AddrConstant GPR:$rj, ImmOpnd:$imm)),
1563:             (Inst vt:$vd, GPR:$rj, ImmOpnd:$imm, IdxOpnd:$idx)>;
1564: 
1565:   def : Pat<(StoreOp(elt(vector_extract vt:$vd, IdxOpnd:$idx)),
1566:                 (AddLike BaseAddr:$rj, ImmOpnd:$imm)),
1567:             (Inst vt:$vd, BaseAddr:$rj, ImmOpnd:$imm, IdxOpnd:$idx)>;
1568: }
1569: 
1570: multiclass InsertExtractPatV4<ValueType vecty, ValueType elemty> {
1571:   foreach imm1 = 0...3 in {
1572:     foreach imm2 = 0...3 in {
1573:       defvar Imm = !or(!shl(imm2, 4), imm1);
1574:       def : Pat<(vector_insert vecty:$vd,
1575:                     (elemty (vector_extract vecty:$vj, imm1)), imm2),
1576:                 (VEXTRINS_W $vd, $vj, Imm)>;
1577:     }
1578:   }
1579: }
1580: 
1581: multiclass InsertExtractPatV2<ValueType vecty, ValueType elemty> {
1582:   foreach imm1 = 0...1 in {
1583:     foreach imm2 = 0...1 in {
1584:       defvar Imm = !or(!shl(imm2, 4), imm1);
1585:       def : Pat<(vector_insert vecty:$vd,
1586:                     (elemty (vector_extract vecty:$vj, imm1)), imm2),
1587:                 (VEXTRINS_D $vd, $vj, Imm)>;
1588:     }
1589:   }
1590: }
1591: 
1592: multiclass VAvgPat<SDPatternOperator OpNode, string Inst, ValueType vt> {
1593:   def : Pat<(OpNode (vt (add vt:$vj, vt:$vk)), (vt (vsplat_imm_eq_1))),
1594:             (!cast<LAInst>(Inst) vt:$vj, vt:$vk)>;
1595: }
1596: 
1597: multiclass VAvgrPat<SDPatternOperator OpNode, string Inst, ValueType vt> {
1598:   def : Pat<(OpNode (vt (add (vt (add vt:$vj, vt:$vk)),
1599:                              (vt (vsplat_imm_eq_1)))),
1600:                     (vt (vsplat_imm_eq_1))),
```
- **EN**: This block declares or refines TableGen records such as `InsertExtractPatV4`, `InsertExtractPatV2`, `VAvgPat`, `VAvgrPat`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `InsertExtractPatV4`, `InsertExtractPatV2`, `VAvgPat`, `VAvgrPat` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601:             (!cast<LAInst>(Inst) vt:$vj, vt:$vk)>;
1602: }
1603: 
1604: let Predicates = [HasExtLSX] in {
1605: 
1606: // VADD_{B/H/W/D}
1607: defm : PatVrVr<add, "VADD">;
1608: // VSUB_{B/H/W/D}
1609: defm : PatVrVr<sub, "VSUB">;
1610: 
1611: // VADDI_{B/H/W/D}U
1612: defm : PatVrNimm5<sub, "VADDI">;
1613: defm : PatVrUimm5<add, "VADDI">;
1614: // VSUBI_{B/H/W/D}U
1615: defm : PatVrNimm5<add, "VSUBI">;
1616: defm : PatVrUimm5<sub, "VSUBI">;
1617: 
1618: // VNEG_{B/H/W/D}
1619: def : Pat<(sub immAllZerosV, (v16i8 LSX128:$vj)), (VNEG_B LSX128:$vj)>;
1620: def : Pat<(sub immAllZerosV, (v8i16 LSX128:$vj)), (VNEG_H LSX128:$vj)>;
1621: def : Pat<(sub immAllZerosV, (v4i32 LSX128:$vj)), (VNEG_W LSX128:$vj)>;
1622: def : Pat<(sub immAllZerosV, (v2i64 LSX128:$vj)), (VNEG_D LSX128:$vj)>;
1623: 
1624: // VMAX[I]_{B/H/W/D}[U]
1625: defm : PatVrVr<smax, "VMAX">;
1626: defm : PatVrVrU<umax, "VMAX">;
1627: defm : PatVrSimm5<smax, "VMAXI">;
1628: defm : PatVrUimm5<umax, "VMAXI">;
1629: 
1630: // VMIN[I]_{B/H/W/D}[U]
1631: defm : PatVrVr<smin, "VMIN">;
1632: defm : PatVrVrU<umin, "VMIN">;
1633: defm : PatVrSimm5<smin, "VMINI">;
1634: defm : PatVrUimm5<umin, "VMINI">;
1635: 
1636: // VMUL_{B/H/W/D}
1637: defm : PatVrVr<mul, "VMUL">;
1638: 
1639: // VMUH_{B/H/W/D}[U]
1640: defm : PatVrVr<mulhs, "VMUH">;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1641-1680 / 第 1641-1680 行
```tablegen
1641: defm : PatVrVrU<mulhu, "VMUH">;
1642: 
1643: // VMADD_{B/H/W/D}
1644: defm : PatVrVrVr<muladd, "VMADD">;
1645: // VMSUB_{B/H/W/D}
1646: defm : PatVrVrVr<mulsub, "VMSUB">;
1647: 
1648: // VDIV_{B/H/W/D}[U]
1649: defm : PatVrVr<sdiv, "VDIV">;
1650: defm : PatVrVrU<udiv, "VDIV">;
1651: 
1652: // VMOD_{B/H/W/D}[U]
1653: defm : PatVrVr<srem, "VMOD">;
1654: defm : PatVrVrU<urem, "VMOD">;
1655: 
1656: // VAND_V
1657: foreach vt = [v16i8, v8i16, v4i32, v2i64] in
1658: def : Pat<(and (vt LSX128:$vj), (vt LSX128:$vk)),
1659:           (VAND_V LSX128:$vj, LSX128:$vk)>;
1660: // VOR_V
1661: foreach vt = [v16i8, v8i16, v4i32, v2i64] in
1662: def : Pat<(or (vt LSX128:$vj), (vt LSX128:$vk)),
1663:           (VOR_V LSX128:$vj, LSX128:$vk)>;
1664: // VXOR_V
1665: foreach vt = [v16i8, v8i16, v4i32, v2i64] in
1666: def : Pat<(xor (vt LSX128:$vj), (vt LSX128:$vk)),
1667:           (VXOR_V LSX128:$vj, LSX128:$vk)>;
1668: // VNOR_V
1669: foreach vt = [v16i8, v8i16, v4i32, v2i64] in
1670: def : Pat<(vnot (or (vt LSX128:$vj), (vt LSX128:$vk))),
1671:           (VNOR_V LSX128:$vj, LSX128:$vk)>;
1672: // VANDN_V
1673: foreach vt = [v16i8, v8i16, v4i32, v2i64] in
1674: def : Pat<(loongarch_vandn (vt LSX128:$vj), (vt LSX128:$vk)),
1675:           (VANDN_V LSX128:$vj, LSX128:$vk)>;
1676: // VORN_V
1677: foreach vt = [v16i8, v8i16, v4i32, v2i64] in
1678: def : Pat<(or (vt LSX128:$vj), (vt (vnot LSX128:$vk))),
1679:           (VORN_V LSX128:$vj, LSX128:$vk)>;
1680: 
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1681-1720 / 第 1681-1720 行
```tablegen
1681: // VANDI_B
1682: def : Pat<(and (v16i8 LSX128:$vj), (v16i8 (SplatPat_uimm8 uimm8:$imm))),
1683:           (VANDI_B LSX128:$vj, uimm8:$imm)>;
1684: foreach vt = [v8i16, v4i32, v2i64] in
1685: def : Pat<(and (vt LSX128:$vj), (vt (vsplat_i8_uimm8 grlenimm:$imm))),
1686:           (VANDI_B LSX128:$vj, grlenimm:$imm)>;
1687: // VORI_B
1688: def : Pat<(or (v16i8 LSX128:$vj), (v16i8 (SplatPat_uimm8 uimm8:$imm))),
1689:           (VORI_B LSX128:$vj, uimm8:$imm)>;
1690: foreach vt = [v8i16, v4i32, v2i64] in
1691: def : Pat<(or (vt LSX128:$vj), (vt (vsplat_i8_uimm8 grlenimm:$imm))),
1692:           (VORI_B LSX128:$vj, grlenimm:$imm)>;
1693: // VXORI_B
1694: def : Pat<(xor (v16i8 LSX128:$vj), (v16i8 (SplatPat_uimm8 uimm8:$imm))),
1695:           (VXORI_B LSX128:$vj, uimm8:$imm)>;
1696: foreach vt = [v8i16, v4i32, v2i64] in
1697: def : Pat<(xor (vt LSX128:$vj), (vt (vsplat_i8_uimm8 grlenimm:$imm))),
1698:           (VXORI_B LSX128:$vj, grlenimm:$imm)>;
1699: // VNORI_B
1700: def : Pat<(vnot (or (v16i8 LSX128:$vj), (v16i8 (SplatPat_uimm8 uimm8:$imm)))),
1701:           (VNORI_B LSX128:$vj, uimm8:$imm)>;
1702: foreach vt = [v8i16, v4i32, v2i64] in
1703: def : Pat<(vnot (or (vt LSX128:$vj), (vt (vsplat_i8_uimm8 grlenimm:$imm)))),
1704:           (VNORI_B LSX128:$vj, grlenimm:$imm)>;
1705: 
1706: // VBSLL_V
1707: foreach vt = [v16i8, v8i16, v4i32, v2i64, v4f32,
1708:               v2f64] in def : Pat<(loongarch_vbsll(vt LSX128:$vj), uimm5:$imm),
1709:                                   (VBSLL_V LSX128:$vj, uimm5:$imm)>;
1710: 
1711: // VBSRL_V
1712: foreach vt = [v16i8, v8i16, v4i32, v2i64, v4f32,
1713:               v2f64] in def : Pat<(loongarch_vbsrl(vt LSX128:$vj), uimm5:$imm),
1714:                                   (VBSRL_V LSX128:$vj, uimm5:$imm)>;
1715: 
1716: // VSLL[I]_{B/H/W/D}
1717: defm : PatVrVr<shl, "VSLL">;
1718: defm : PatShiftVrVr<shl, "VSLL">;
1719: defm : PatShiftVrSplatUimm<shl, "VSLLI">;
1720: defm : PatShiftVrUimm<loongarch_vslli, "VSLLI">;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1721-1760 / 第 1721-1760 行
```tablegen
1721: 
1722: // VSRL[I]_{B/H/W/D}
1723: defm : PatVrVr<srl, "VSRL">;
1724: defm : PatShiftVrVr<srl, "VSRL">;
1725: defm : PatShiftVrSplatUimm<srl, "VSRLI">;
1726: defm : PatShiftVrUimm<loongarch_vsrli, "VSRLI">;
1727: 
1728: // VSRLR[I]_{B/H/W/D}
1729: defm : PatVrVr<loongarch_vsrlr, "VSRLR">;
1730: defm : PatShiftVrSplatUimm<loongarch_vsrlr, "VSRLRI">;
1731: 
1732: // VSRA[I]_{B/H/W/D}
1733: defm : PatVrVr<sra, "VSRA">;
1734: defm : PatShiftVrVr<sra, "VSRA">;
1735: defm : PatShiftVrSplatUimm<sra, "VSRAI">;
1736: 
1737: // VSRAR[I]_{B/H/W/D}
1738: defm : PatVrVr<loongarch_vsrar, "VSRAR">;
1739: defm : PatShiftVrSplatUimm<loongarch_vsrar, "VSRARI">;
1740: 
1741: // VROTR[I]_{B/H/W/D}
1742: defm : PatVrVr<rotr, "VROTR">;
1743: defm : PatShiftVrVr<rotr, "VROTR">;
1744: defm : PatShiftVrSplatUimm<rotr, "VROTRI">;
1745: 
1746: // VCLO_{B/H/W/D}
1747: def : Pat<(ctlz (vnot v16i8:$vj)), (VCLO_B v16i8:$vj)>;
1748: def : Pat<(ctlz (vnot v8i16:$vj)), (VCLO_H v8i16:$vj)>;
1749: def : Pat<(ctlz (vnot v4i32:$vj)), (VCLO_W v4i32:$vj)>;
1750: def : Pat<(ctlz (vnot v2i64:$vj)), (VCLO_D v2i64:$vj)>;
1751: 
1752: // VCLZ_{B/H/W/D}
1753: defm : PatVr<ctlz, "VCLZ">;
1754: 
1755: // VPCNT_{B/H/W/D}
1756: defm : PatVr<ctpop, "VPCNT">;
1757: 
1758: // VBITCLR_{B/H/W/D}
1759: def : Pat<(loongarch_vandn (v16i8 (shl vsplat_imm_eq_1, v16i8:$vk)), v16i8:$vj),
1760:           (v16i8 (VBITCLR_B v16i8:$vj, v16i8:$vk))>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1761-1800 / 第 1761-1800 行
```tablegen
1761: def : Pat<(loongarch_vandn (v8i16 (shl vsplat_imm_eq_1, v8i16:$vk)), v8i16:$vj),
1762:           (v8i16 (VBITCLR_H v8i16:$vj, v8i16:$vk))>;
1763: def : Pat<(loongarch_vandn (v4i32 (shl vsplat_imm_eq_1, v4i32:$vk)), v4i32:$vj),
1764:           (v4i32 (VBITCLR_W v4i32:$vj, v4i32:$vk))>;
1765: def : Pat<(loongarch_vandn (v2i64 (shl vsplat_imm_eq_1, v2i64:$vk)), v2i64:$vj),
1766:           (v2i64 (VBITCLR_D v2i64:$vj, v2i64:$vk))>;
1767: def : Pat<(loongarch_vandn (v16i8 (shl vsplat_imm_eq_1,
1768:                                   (vsplati8imm7 v16i8:$vk))), v16i8:$vj),
1769:           (v16i8 (VBITCLR_B v16i8:$vj, v16i8:$vk))>;
1770: def : Pat<(loongarch_vandn (v8i16 (shl vsplat_imm_eq_1,
1771:                                   (vsplati16imm15 v8i16:$vk))), v8i16:$vj),
1772:           (v8i16 (VBITCLR_H v8i16:$vj, v8i16:$vk))>;
1773: def : Pat<(loongarch_vandn (v4i32 (shl vsplat_imm_eq_1,
1774:                                   (vsplati32imm31 v4i32:$vk))), v4i32:$vj),
1775:           (v4i32 (VBITCLR_W v4i32:$vj, v4i32:$vk))>;
1776: def : Pat<(loongarch_vandn (v2i64 (shl vsplat_imm_eq_1,
1777:                                   (vsplati64imm63 v2i64:$vk))), v2i64:$vj),
1778:           (v2i64 (VBITCLR_D v2i64:$vj, v2i64:$vk))>;
1779: 
1780: // VBITCLRI_{B/H/W/D}
1781: def : Pat<(and (v16i8 LSX128:$vj), (v16i8 (vsplat_uimm_inv_pow2 uimm3:$imm))),
1782:           (VBITCLRI_B LSX128:$vj, uimm3:$imm)>;
1783: def : Pat<(and (v8i16 LSX128:$vj), (v8i16 (vsplat_uimm_inv_pow2 uimm4:$imm))),
1784:           (VBITCLRI_H LSX128:$vj, uimm4:$imm)>;
1785: def : Pat<(and (v4i32 LSX128:$vj), (v4i32 (vsplat_uimm_inv_pow2 uimm5:$imm))),
1786:           (VBITCLRI_W LSX128:$vj, uimm5:$imm)>;
1787: def : Pat<(and (v2i64 LSX128:$vj), (v2i64 (vsplat_uimm_inv_pow2 uimm6:$imm))),
1788:           (VBITCLRI_D LSX128:$vj, uimm6:$imm)>;
1789: 
1790: foreach vt = [v8i16, v4i32, v2i64] in {
1791:   def : Pat<(and (vt LSX128:$vj), (vt (vsplat_i8_inv_pow2 grlenimm:$imm))),
1792:             (VBITCLRI_B LSX128:$vj, grlenimm:$imm)>;
1793:   def : Pat<(and (vt LSX128:$vj), (vt (vsplat_i16_inv_pow2 grlenimm:$imm))),
1794:             (VBITCLRI_H LSX128:$vj, grlenimm:$imm)>;
1795:   def : Pat<(and (vt LSX128:$vj), (vt (vsplat_i32_inv_pow2 grlenimm:$imm))),
1796:             (VBITCLRI_W LSX128:$vj, grlenimm:$imm)>;
1797: }
1798: 
1799: // VBITSET_{B/H/W/D}
1800: def : Pat<(or v16i8:$vj, (shl vsplat_imm_eq_1, v16i8:$vk)),
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1801-1840 / 第 1801-1840 行
```tablegen
1801:           (v16i8 (VBITSET_B v16i8:$vj, v16i8:$vk))>;
1802: def : Pat<(or v8i16:$vj, (shl vsplat_imm_eq_1, v8i16:$vk)),
1803:           (v8i16 (VBITSET_H v8i16:$vj, v8i16:$vk))>;
1804: def : Pat<(or v4i32:$vj, (shl vsplat_imm_eq_1, v4i32:$vk)),
1805:           (v4i32 (VBITSET_W v4i32:$vj, v4i32:$vk))>;
1806: def : Pat<(or v2i64:$vj, (shl vsplat_imm_eq_1, v2i64:$vk)),
1807:           (v2i64 (VBITSET_D v2i64:$vj, v2i64:$vk))>;
1808: def : Pat<(or v16i8:$vj, (shl vsplat_imm_eq_1, (vsplati8imm7 v16i8:$vk))),
1809:           (v16i8 (VBITSET_B v16i8:$vj, v16i8:$vk))>;
1810: def : Pat<(or v8i16:$vj, (shl vsplat_imm_eq_1, (vsplati16imm15 v8i16:$vk))),
1811:           (v8i16 (VBITSET_H v8i16:$vj, v8i16:$vk))>;
1812: def : Pat<(or v4i32:$vj, (shl vsplat_imm_eq_1, (vsplati32imm31 v4i32:$vk))),
1813:           (v4i32 (VBITSET_W v4i32:$vj, v4i32:$vk))>;
1814: def : Pat<(or v2i64:$vj, (shl vsplat_imm_eq_1, (vsplati64imm63 v2i64:$vk))),
1815:           (v2i64 (VBITSET_D v2i64:$vj, v2i64:$vk))>;
1816: 
1817: // VBITSETI_{B/H/W/D}
1818: def : Pat<(or (v16i8 LSX128:$vj), (v16i8 (vsplat_uimm_pow2 uimm3:$imm))),
1819:           (VBITSETI_B LSX128:$vj, uimm3:$imm)>;
1820: def : Pat<(or (v8i16 LSX128:$vj), (v8i16 (vsplat_uimm_pow2 uimm4:$imm))),
1821:           (VBITSETI_H LSX128:$vj, uimm4:$imm)>;
1822: def : Pat<(or (v4i32 LSX128:$vj), (v4i32 (vsplat_uimm_pow2 uimm5:$imm))),
1823:           (VBITSETI_W LSX128:$vj, uimm5:$imm)>;
1824: def : Pat<(or (v2i64 LSX128:$vj), (v2i64 (vsplat_uimm_pow2 uimm6:$imm))),
1825:           (VBITSETI_D LSX128:$vj, uimm6:$imm)>;
1826: 
1827: foreach vt = [v8i16, v4i32, v2i64] in {
1828:   def : Pat<(or (vt LSX128:$vj), (vt (vsplat_i8_pow2 grlenimm:$imm))),
1829:             (VBITSETI_B LSX128:$vj, grlenimm:$imm)>;
1830:   def : Pat<(or (vt LSX128:$vj), (vt (vsplat_i16_pow2 grlenimm:$imm))),
1831:             (VBITSETI_H LSX128:$vj, grlenimm:$imm)>;
1832:   def : Pat<(or (vt LSX128:$vj), (vt (vsplat_i32_pow2 grlenimm:$imm))),
1833:             (VBITSETI_W LSX128:$vj, grlenimm:$imm)>;
1834: }
1835: 
1836: // VBITREV_{B/H/W/D}
1837: def : Pat<(xor v16i8:$vj, (shl vsplat_imm_eq_1, v16i8:$vk)),
1838:           (v16i8 (VBITREV_B v16i8:$vj, v16i8:$vk))>;
1839: def : Pat<(xor v8i16:$vj, (shl vsplat_imm_eq_1, v8i16:$vk)),
1840:           (v8i16 (VBITREV_H v8i16:$vj, v8i16:$vk))>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1841-1880 / 第 1841-1880 行
```tablegen
1841: def : Pat<(xor v4i32:$vj, (shl vsplat_imm_eq_1, v4i32:$vk)),
1842:           (v4i32 (VBITREV_W v4i32:$vj, v4i32:$vk))>;
1843: def : Pat<(xor v2i64:$vj, (shl vsplat_imm_eq_1, v2i64:$vk)),
1844:           (v2i64 (VBITREV_D v2i64:$vj, v2i64:$vk))>;
1845: def : Pat<(xor v16i8:$vj, (shl vsplat_imm_eq_1, (vsplati8imm7 v16i8:$vk))),
1846:           (v16i8 (VBITREV_B v16i8:$vj, v16i8:$vk))>;
1847: def : Pat<(xor v8i16:$vj, (shl vsplat_imm_eq_1, (vsplati16imm15 v8i16:$vk))),
1848:           (v8i16 (VBITREV_H v8i16:$vj, v8i16:$vk))>;
1849: def : Pat<(xor v4i32:$vj, (shl vsplat_imm_eq_1, (vsplati32imm31 v4i32:$vk))),
1850:           (v4i32 (VBITREV_W v4i32:$vj, v4i32:$vk))>;
1851: def : Pat<(xor v2i64:$vj, (shl vsplat_imm_eq_1, (vsplati64imm63 v2i64:$vk))),
1852:           (v2i64 (VBITREV_D v2i64:$vj, v2i64:$vk))>;
1853: 
1854: // VBITREVI_{B/H/W/D}
1855: def : Pat<(xor (v16i8 LSX128:$vj), (v16i8 (vsplat_uimm_pow2 uimm3:$imm))),
1856:           (VBITREVI_B LSX128:$vj, uimm3:$imm)>;
1857: def : Pat<(xor (v8i16 LSX128:$vj), (v8i16 (vsplat_uimm_pow2 uimm4:$imm))),
1858:           (VBITREVI_H LSX128:$vj, uimm4:$imm)>;
1859: def : Pat<(xor (v4i32 LSX128:$vj), (v4i32 (vsplat_uimm_pow2 uimm5:$imm))),
1860:           (VBITREVI_W LSX128:$vj, uimm5:$imm)>;
1861: def : Pat<(xor (v2i64 LSX128:$vj), (v2i64 (vsplat_uimm_pow2 uimm6:$imm))),
1862:           (VBITREVI_D LSX128:$vj, uimm6:$imm)>;
1863: 
1864: foreach vt = [v8i16, v4i32, v2i64] in {
1865:   def : Pat<(xor (vt LSX128:$vj), (vt (vsplat_i8_pow2 grlenimm:$imm))),
1866:             (VBITREVI_B LSX128:$vj, grlenimm:$imm)>;
1867:   def : Pat<(xor (vt LSX128:$vj), (vt (vsplat_i16_pow2 grlenimm:$imm))),
1868:             (VBITREVI_H LSX128:$vj, grlenimm:$imm)>;
1869:   def : Pat<(xor (vt LSX128:$vj), (vt (vsplat_i32_pow2 grlenimm:$imm))),
1870:             (VBITREVI_W LSX128:$vj, grlenimm:$imm)>;
1871: }
1872: 
1873: // VBITSEL_V
1874: foreach vt = [v16i8, v8i16, v4i32, v2i64] in
1875:   def : Pat<(vt (or (and LSX128:$va, LSX128:$vk),
1876:                     (loongarch_vandn LSX128:$va, LSX128:$vj))),
1877:             (vt (VBITSEL_V LSX128:$vj, LSX128:$vk, LSX128:$va))>;
1878: 
1879: // VBITSELI_B
1880: def : Pat<(v16i8 (or (and LSX128:$vd, (v16i8 (SplatPat_uimm8 uimm8:$imm))),
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1881-1920 / 第 1881-1920 行
```tablegen
1881:                      (loongarch_vandn LSX128:$vd, LSX128:$vj))),
1882:           (VBITSELI_B LSX128:$vd, LSX128:$vj, uimm8:$imm)>;
1883: 
1884: // Vector bswaps
1885: def : Pat<(bswap (v8i16 LSX128:$vj)), (VSHUF4I_B LSX128:$vj, 0b10110001)>;
1886: def : Pat<(bswap (v4i32 LSX128:$vj)), (VSHUF4I_B LSX128:$vj, 0b00011011)>;
1887: def : Pat<(bswap (v2i64 LSX128:$vj)),
1888:           (VSHUF4I_W (VSHUF4I_B LSX128:$vj, 0b00011011), 0b10110001)>;
1889: 
1890: // VHADDW_{H_B/W_H/D_W/Q_D}
1891: defm : PatVrVrW<loongarch_vhaddw, "VHADDW">;
1892: 
1893: // VFADD_{S/D}
1894: defm : PatVrVrF<fadd, "VFADD">;
1895: 
1896: // VFSUB_{S/D}
1897: defm : PatVrVrF<fsub, "VFSUB">;
1898: 
1899: // VFMUL_{S/D}
1900: defm : PatVrVrF<fmul, "VFMUL">;
1901: 
1902: // VFDIV_{S/D}
1903: defm : PatVrVrF<fdiv, "VFDIV">;
1904: 
1905: // VFMAX_{S/D}, VFMIN_{S/D}
1906: defm : PatVrVrF<fmaxnum, "VFMAX">;
1907: defm : PatVrVrF<fminnum, "VFMIN">;
1908: 
1909: // VFMADD_{S/D}
1910: def : Pat<(fma v4f32:$vj, v4f32:$vk, v4f32:$va),
1911:           (VFMADD_S v4f32:$vj, v4f32:$vk, v4f32:$va)>;
1912: def : Pat<(fma v2f64:$vj, v2f64:$vk, v2f64:$va),
1913:           (VFMADD_D v2f64:$vj, v2f64:$vk, v2f64:$va)>;
1914: 
1915: // VFMSUB_{S/D}
1916: def : Pat<(fma v4f32:$vj, v4f32:$vk, (fneg v4f32:$va)),
1917:           (VFMSUB_S v4f32:$vj, v4f32:$vk, v4f32:$va)>;
1918: def : Pat<(fma v2f64:$vj, v2f64:$vk, (fneg v2f64:$va)),
1919:           (VFMSUB_D v2f64:$vj, v2f64:$vk, v2f64:$va)>;
1920: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1921-1960 / 第 1921-1960 行
```tablegen
1921: // VFNMADD_{S/D}
1922: def : Pat<(fneg (fma v4f32:$vj, v4f32:$vk, v4f32:$va)),
1923:           (VFNMADD_S v4f32:$vj, v4f32:$vk, v4f32:$va)>;
1924: def : Pat<(fneg (fma v2f64:$vj, v2f64:$vk, v2f64:$va)),
1925:           (VFNMADD_D v2f64:$vj, v2f64:$vk, v2f64:$va)>;
1926: def : Pat<(fma_nsz (fneg v4f32:$vj), v4f32:$vk, (fneg v4f32:$va)),
1927:           (VFNMADD_S v4f32:$vj, v4f32:$vk, v4f32:$va)>;
1928: def : Pat<(fma_nsz (fneg v2f64:$vj), v2f64:$vk, (fneg v2f64:$va)),
1929:           (VFNMADD_D v2f64:$vj, v2f64:$vk, v2f64:$va)>;
1930: 
1931: // VFNMSUB_{S/D}
1932: def : Pat<(fneg (fma v4f32:$vj, v4f32:$vk, (fneg v4f32:$va))),
1933:           (VFNMSUB_S v4f32:$vj, v4f32:$vk, v4f32:$va)>;
1934: def : Pat<(fneg (fma v2f64:$vj, v2f64:$vk, (fneg v2f64:$va))),
1935:           (VFNMSUB_D v2f64:$vj, v2f64:$vk, v2f64:$va)>;
1936: def : Pat<(fma_nsz (fneg v4f32:$vj), v4f32:$vk, v4f32:$va),
1937:           (VFNMSUB_S v4f32:$vj, v4f32:$vk, v4f32:$va)>;
1938: def : Pat<(fma_nsz (fneg v2f64:$vj), v2f64:$vk, v2f64:$va),
1939:           (VFNMSUB_D v2f64:$vj, v2f64:$vk, v2f64:$va)>;
1940: 
1941: // VFSQRT_{S/D}
1942: defm : PatVrF<fsqrt, "VFSQRT">;
1943: 
1944: // VFRECIP_{S/D}
1945: def : Pat<(fdiv vsplatf32_fpimm_eq_1, v4f32:$vj),
1946:           (VFRECIP_S v4f32:$vj)>;
1947: def : Pat<(fdiv vsplatf64_fpimm_eq_1, v2f64:$vj),
1948:           (VFRECIP_D v2f64:$vj)>;
1949: 
1950: // VFRSQRT_{S/D}
1951: def : Pat<(fdiv vsplatf32_fpimm_eq_1, (fsqrt v4f32:$vj)),
1952:           (VFRSQRT_S v4f32:$vj)>;
1953: def : Pat<(fdiv vsplatf64_fpimm_eq_1, (fsqrt v2f64:$vj)),
1954:           (VFRSQRT_D v2f64:$vj)>;
1955: 
1956: // VSEQ[I]_{B/H/W/D}
1957: defm : PatCCVrSimm5<SETEQ, "VSEQI">;
1958: defm : PatCCVrVr<SETEQ, "VSEQ">;
1959: 
1960: // VSLE[I]_{B/H/W/D}[U]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1961-2000 / 第 1961-2000 行
```tablegen
1961: defm : PatCCVrSimm5<SETLE, "VSLEI">;
1962: defm : PatCCVrUimm5<SETULE, "VSLEI">;
1963: defm : PatCCVrVr<SETLE, "VSLE">;
1964: defm : PatCCVrVrU<SETULE, "VSLE">;
1965: 
1966: // VSLT[I]_{B/H/W/D}[U]
1967: defm : PatCCVrSimm5<SETLT, "VSLTI">;
1968: defm : PatCCVrUimm5<SETULT, "VSLTI">;
1969: defm : PatCCVrVr<SETLT, "VSLT">;
1970: defm : PatCCVrVrU<SETULT, "VSLT">;
1971: 
1972: // VFCMP.cond.{S/D}
1973: defm : PatCCVrVrF<SETEQ, "VFCMP_CEQ">;
1974: defm : PatCCVrVrF<SETOEQ, "VFCMP_CEQ">;
1975: defm : PatCCVrVrF<SETUEQ, "VFCMP_CUEQ">;
1976: 
1977: defm : PatCCVrVrF<SETLE, "VFCMP_CLE">;
1978: defm : PatCCVrVrF<SETOLE, "VFCMP_CLE">;
1979: defm : PatCCVrVrF<SETULE, "VFCMP_CULE">;
1980: 
1981: defm : PatCCVrVrF<SETLT, "VFCMP_CLT">;
1982: defm : PatCCVrVrF<SETOLT, "VFCMP_CLT">;
1983: defm : PatCCVrVrF<SETULT, "VFCMP_CULT">;
1984: 
1985: defm : PatCCVrVrF<SETNE, "VFCMP_CNE">;
1986: defm : PatCCVrVrF<SETONE, "VFCMP_CNE">;
1987: defm : PatCCVrVrF<SETUNE, "VFCMP_CUNE">;
1988: 
1989: defm : PatCCVrVrF<SETO, "VFCMP_COR">;
1990: defm : PatCCVrVrF<SETUO, "VFCMP_CUN">;
1991: 
1992: // Insert element extracted from vector into vector.
1993: // VPICKVE2GR_{B/H/W/D} + VINSGR2VR_{B/H/W/D} -> VEXTRINS_{B/H/W/D}
1994: foreach imm1 = 0...15 in {
1995:   foreach imm2 = 0...15 in {
1996:     defvar Imm = !or(!shl(imm2, 4), imm1);
1997:     def : Pat<(vector_insert v16i8:$vd,
1998:                   (GRLenVT (vector_extract v16i8:$vj, imm1)), imm2),
1999:               (VEXTRINS_B $vd, $vj, Imm)>;
2000:   }
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2001-2040 / 第 2001-2040 行
```tablegen
2001: }
2002: 
2003: foreach imm1 = 0...7 in {
2004:   foreach imm2 = 0...7 in {
2005:     defvar Imm = !or(!shl(imm2, 4), imm1);
2006:     def : Pat<(vector_insert v8i16:$vd,
2007:                   (GRLenVT (vector_extract v8i16:$vj, imm1)), imm2),
2008:               (VEXTRINS_H $vd, $vj, Imm)>;
2009:   }
2010: }
2011: 
2012: defm : InsertExtractPatV4<v4i32, GRLenVT>;
2013: defm : InsertExtractPatV4<v4f32, f32>;
2014: defm : InsertExtractPatV2<v2i64, GRLenVT>;
2015: defm : InsertExtractPatV2<v2f64, f64>;
2016: 
2017: // VINSGR2VR_{B/H/W/D}
2018: def : Pat<(vector_insert v16i8:$vd, GRLenVT:$rj, uimm4:$imm),
2019:           (VINSGR2VR_B v16i8:$vd, GRLenVT:$rj, uimm4:$imm)>;
2020: def : Pat<(vector_insert v8i16:$vd, GRLenVT:$rj, uimm3:$imm),
2021:           (VINSGR2VR_H v8i16:$vd, GRLenVT:$rj, uimm3:$imm)>;
2022: def : Pat<(vector_insert v4i32:$vd, GRLenVT:$rj, uimm2:$imm),
2023:           (VINSGR2VR_W v4i32:$vd, GRLenVT:$rj, uimm2:$imm)>;
2024: def : Pat<(vector_insert v2i64:$vd, GRLenVT:$rj, uimm1:$imm),
2025:           (VINSGR2VR_D v2i64:$vd, GRLenVT:$rj, uimm1:$imm)>;
2026: def : Pat<(vector_insert v4f32:$vd, (loongarch_movgr2fr_w_la64 GPR:$rj), uimm2:$imm),
2027:           (VINSGR2VR_W $vd, $rj, uimm2:$imm)>;
2028: def : Pat<(vector_insert v2f64:$vd, (f64 (bitconvert i64:$rj)), uimm1:$imm),
2029:           (VINSGR2VR_D $vd, $rj, uimm1:$imm)>;
2030: 
2031: // VEXTRINS_{W/D}
2032: foreach imm = 0...3 in {
2033:   defvar Imm = !shl(imm, 4);
2034:   def : Pat<(vector_insert v4f32:$vd, FPR32:$fj, imm),
2035:             (VEXTRINS_W $vd, (SUBREG_TO_REG FPR32:$fj, sub_32), Imm)>;
2036: }
2037: 
2038: foreach imm = 0...1 in {
2039:   defvar Imm = !shl(imm, 4);
2040:   def : Pat<(vector_insert v2f64:$vd, FPR64:$fj, imm),
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2041-2080 / 第 2041-2080 行
```tablegen
2041:             (VEXTRINS_D $vd, (SUBREG_TO_REG FPR64:$fj, sub_64), Imm)>;
2042: }
2043: 
2044: // scalar_to_vector
2045: def : Pat<(v4f32 (scalar_to_vector FPR32:$fj)),
2046:           (SUBREG_TO_REG FPR32:$fj, sub_32)>;
2047: def : Pat<(v2f64 (scalar_to_vector FPR64:$fj)),
2048:           (SUBREG_TO_REG FPR64:$fj, sub_64)>;
2049: 
2050: // VPICKVE2GR_{B/H/W}[U]
2051: def : Pat<(loongarch_vpick_sext_elt v16i8:$vd, uimm4:$imm, i8),
2052:           (VPICKVE2GR_B v16i8:$vd, uimm4:$imm)>;
2053: def : Pat<(loongarch_vpick_sext_elt v8i16:$vd, uimm3:$imm, i16),
2054:           (VPICKVE2GR_H v8i16:$vd, uimm3:$imm)>;
2055: def : Pat<(loongarch_vpick_sext_elt v4i32:$vd, uimm2:$imm, i32),
2056:           (VPICKVE2GR_W v4i32:$vd, uimm2:$imm)>;
2057: 
2058: def : Pat<(loongarch_vpick_zext_elt v16i8:$vd, uimm4:$imm, i8),
2059:           (VPICKVE2GR_BU v16i8:$vd, uimm4:$imm)>;
2060: def : Pat<(loongarch_vpick_zext_elt v8i16:$vd, uimm3:$imm, i16),
2061:           (VPICKVE2GR_HU v8i16:$vd, uimm3:$imm)>;
2062: def : Pat<(loongarch_vpick_zext_elt v4i32:$vd, uimm2:$imm, i32),
2063:           (VPICKVE2GR_WU v4i32:$vd, uimm2:$imm)>;
2064: 
2065: // VREPLGR2VR_{B/H/W/D}
2066: def : Pat<(lsxsplati8 GPR:$rj), (VREPLGR2VR_B GPR:$rj)>;
2067: def : Pat<(lsxsplati16 GPR:$rj), (VREPLGR2VR_H GPR:$rj)>;
2068: def : Pat<(lsxsplati32 GPR:$rj), (VREPLGR2VR_W GPR:$rj)>;
2069: def : Pat<(lsxsplati64 GPR:$rj), (VREPLGR2VR_D GPR:$rj)>;
2070: 
2071: def : Pat<(v16i8 (loongarch_vreplgr2vr GRLenVT:$rj)),
2072:           (v16i8 (VREPLGR2VR_B GRLenVT:$rj))>;
2073: def : Pat<(v8i16 (loongarch_vreplgr2vr GRLenVT:$rj)),
2074:           (v8i16 (VREPLGR2VR_H GRLenVT:$rj))>;
2075: def : Pat<(v4i32 (loongarch_vreplgr2vr GRLenVT:$rj)),
2076:           (v4i32 (VREPLGR2VR_W GRLenVT:$rj))>;
2077: def : Pat<(v2i64 (loongarch_vreplgr2vr GRLenVT:$rj)),
2078:           (v2i64 (VREPLGR2VR_D GRLenVT:$rj))>;
2079: 
2080: // VREPLVE_{B/H/W/D}
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2081-2120 / 第 2081-2120 行
```tablegen
2081: def : Pat<(loongarch_vreplve v16i8:$vj, GRLenVT:$rk),
2082:           (VREPLVE_B v16i8:$vj, GRLenVT:$rk)>;
2083: def : Pat<(loongarch_vreplve v8i16:$vj, GRLenVT:$rk),
2084:           (VREPLVE_H v8i16:$vj, GRLenVT:$rk)>;
2085: def : Pat<(loongarch_vreplve v4i32:$vj, GRLenVT:$rk),
2086:           (VREPLVE_W v4i32:$vj, GRLenVT:$rk)>;
2087: def : Pat<(loongarch_vreplve v2i64:$vj, GRLenVT:$rk),
2088:           (VREPLVE_D v2i64:$vj, GRLenVT:$rk)>;
2089: 
2090: // VSHUF_{B/H/W/D}
2091: def : Pat<(loongarch_vshuf v16i8:$va, v16i8:$vj, v16i8:$vk),
2092:           (VSHUF_B v16i8:$vj, v16i8:$vk, v16i8:$va)>;
2093: def : Pat<(loongarch_vshuf v8i16:$vd, v8i16:$vj, v8i16:$vk),
2094:           (VSHUF_H v8i16:$vd, v8i16:$vj, v8i16:$vk)>;
2095: def : Pat<(loongarch_vshuf v4i32:$vd, v4i32:$vj, v4i32:$vk),
2096:           (VSHUF_W v4i32:$vd, v4i32:$vj, v4i32:$vk)>;
2097: def : Pat<(loongarch_vshuf v2i64:$vd, v2i64:$vj, v2i64:$vk),
2098:           (VSHUF_D v2i64:$vd, v2i64:$vj, v2i64:$vk)>;
2099: def : Pat<(loongarch_vshuf v4i32:$vd, v4f32:$vj, v4f32:$vk),
2100:           (VSHUF_W v4i32:$vd, v4f32:$vj, v4f32:$vk)>;
2101: def : Pat<(loongarch_vshuf v2i64:$vd, v2f64:$vj, v2f64:$vk),
2102:           (VSHUF_D v2i64:$vd, v2f64:$vj, v2f64:$vk)>;
2103: 
2104: // VPICKEV_{B/H/W/D}
2105: def : Pat<(loongarch_vpickev v16i8:$vj, v16i8:$vk),
2106:           (VPICKEV_B v16i8:$vj, v16i8:$vk)>;
2107: def : Pat<(loongarch_vpickev v8i16:$vj, v8i16:$vk),
2108:           (VPICKEV_H v8i16:$vj, v8i16:$vk)>;
2109: def : Pat<(loongarch_vpickev v4i32:$vj, v4i32:$vk),
2110:           (VPICKEV_W v4i32:$vj, v4i32:$vk)>;
2111: def : Pat<(loongarch_vpickev v2i64:$vj, v2i64:$vk),
2112:           (VPICKEV_D v2i64:$vj, v2i64:$vk)>;
2113: def : Pat<(loongarch_vpickev v4f32:$vj, v4f32:$vk),
2114:           (VPICKEV_W v4f32:$vj, v4f32:$vk)>;
2115: def : Pat<(loongarch_vpickev v2f64:$vj, v2f64:$vk),
2116:           (VPICKEV_D v2f64:$vj, v2f64:$vk)>;
2117: 
2118: // VPICKOD_{B/H/W/D}
2119: def : Pat<(loongarch_vpickod v16i8:$vj, v16i8:$vk),
2120:           (VPICKOD_B v16i8:$vj, v16i8:$vk)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2121-2160 / 第 2121-2160 行
```tablegen
2121: def : Pat<(loongarch_vpickod v8i16:$vj, v8i16:$vk),
2122:           (VPICKOD_H v8i16:$vj, v8i16:$vk)>;
2123: def : Pat<(loongarch_vpickod v4i32:$vj, v4i32:$vk),
2124:           (VPICKOD_W v4i32:$vj, v4i32:$vk)>;
2125: def : Pat<(loongarch_vpickod v2i64:$vj, v2i64:$vk),
2126:           (VPICKOD_D v2i64:$vj, v2i64:$vk)>;
2127: def : Pat<(loongarch_vpickod v4f32:$vj, v4f32:$vk),
2128:           (VPICKOD_W v4f32:$vj, v4f32:$vk)>;
2129: def : Pat<(loongarch_vpickod v2f64:$vj, v2f64:$vk),
2130:           (VPICKOD_D v2f64:$vj, v2f64:$vk)>;
2131: 
2132: // VPACKEV_{B/H/W/D}
2133: def : Pat<(loongarch_vpackev v16i8:$vj, v16i8:$vk),
2134:           (VPACKEV_B v16i8:$vj, v16i8:$vk)>;
2135: def : Pat<(loongarch_vpackev v8i16:$vj, v8i16:$vk),
2136:           (VPACKEV_H v8i16:$vj, v8i16:$vk)>;
2137: def : Pat<(loongarch_vpackev v4i32:$vj, v4i32:$vk),
2138:           (VPACKEV_W v4i32:$vj, v4i32:$vk)>;
2139: def : Pat<(loongarch_vpackev v2i64:$vj, v2i64:$vk),
2140:           (VPACKEV_D v2i64:$vj, v2i64:$vk)>;
2141: def : Pat<(loongarch_vpackev v4f32:$vj, v4f32:$vk),
2142:           (VPACKEV_W v4f32:$vj, v4f32:$vk)>;
2143: def : Pat<(loongarch_vpackev v2f64:$vj, v2f64:$vk),
2144:           (VPACKEV_D v2f64:$vj, v2f64:$vk)>;
2145: 
2146: // VPACKOD_{B/H/W/D}
2147: def : Pat<(loongarch_vpackod v16i8:$vj, v16i8:$vk),
2148:           (VPACKOD_B v16i8:$vj, v16i8:$vk)>;
2149: def : Pat<(loongarch_vpackod v8i16:$vj, v8i16:$vk),
2150:           (VPACKOD_H v8i16:$vj, v8i16:$vk)>;
2151: def : Pat<(loongarch_vpackod v4i32:$vj, v4i32:$vk),
2152:           (VPACKOD_W v4i32:$vj, v4i32:$vk)>;
2153: def : Pat<(loongarch_vpackod v2i64:$vj, v2i64:$vk),
2154:           (VPACKOD_D v2i64:$vj, v2i64:$vk)>;
2155: def : Pat<(loongarch_vpackod v4f32:$vj, v4f32:$vk),
2156:           (VPACKOD_W v4f32:$vj, v4f32:$vk)>;
2157: def : Pat<(loongarch_vpackod v2f64:$vj, v2f64:$vk),
2158:           (VPACKOD_D v2f64:$vj, v2f64:$vk)>;
2159: 
2160: // VILVL_{B/H/W/D}
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2161-2200 / 第 2161-2200 行
```tablegen
2161: def : Pat<(loongarch_vilvl v16i8:$vj, v16i8:$vk),
2162:           (VILVL_B v16i8:$vj, v16i8:$vk)>;
2163: def : Pat<(loongarch_vilvl v8i16:$vj, v8i16:$vk),
2164:           (VILVL_H v8i16:$vj, v8i16:$vk)>;
2165: def : Pat<(loongarch_vilvl v4i32:$vj, v4i32:$vk),
2166:           (VILVL_W v4i32:$vj, v4i32:$vk)>;
2167: def : Pat<(loongarch_vilvl v2i64:$vj, v2i64:$vk),
2168:           (VILVL_D v2i64:$vj, v2i64:$vk)>;
2169: def : Pat<(loongarch_vilvl v4f32:$vj, v4f32:$vk),
2170:           (VILVL_W v4f32:$vj, v4f32:$vk)>;
2171: def : Pat<(loongarch_vilvl v2f64:$vj, v2f64:$vk),
2172:           (VILVL_D v2f64:$vj, v2f64:$vk)>;
2173: 
2174: // VILVH_{B/H/W/D}
2175: def : Pat<(loongarch_vilvh v16i8:$vj, v16i8:$vk),
2176:           (VILVH_B v16i8:$vj, v16i8:$vk)>;
2177: def : Pat<(loongarch_vilvh v8i16:$vj, v8i16:$vk),
2178:           (VILVH_H v8i16:$vj, v8i16:$vk)>;
2179: def : Pat<(loongarch_vilvh v4i32:$vj, v4i32:$vk),
2180:           (VILVH_W v4i32:$vj, v4i32:$vk)>;
2181: def : Pat<(loongarch_vilvh v2i64:$vj, v2i64:$vk),
2182:           (VILVH_D v2i64:$vj, v2i64:$vk)>;
2183: def : Pat<(loongarch_vilvh v4f32:$vj, v4f32:$vk),
2184:           (VILVH_W v4f32:$vj, v4f32:$vk)>;
2185: def : Pat<(loongarch_vilvh v2f64:$vj, v2f64:$vk),
2186:           (VILVH_D v2f64:$vj, v2f64:$vk)>;
2187: 
2188: // VSHUF4I_{B/H/W/D}
2189: def : Pat<(loongarch_vshuf4i v16i8:$vj, immZExt8:$ui8),
2190:           (VSHUF4I_B v16i8:$vj, immZExt8:$ui8)>;
2191: def : Pat<(loongarch_vshuf4i v8i16:$vj, immZExt8:$ui8),
2192:           (VSHUF4I_H v8i16:$vj, immZExt8:$ui8)>;
2193: def : Pat<(loongarch_vshuf4i v4i32:$vj, immZExt8:$ui8),
2194:           (VSHUF4I_W v4i32:$vj, immZExt8:$ui8)>;
2195: def : Pat<(loongarch_vshuf4i v4f32:$vj, immZExt8:$ui8),
2196:           (VSHUF4I_W v4f32:$vj, immZExt8:$ui8)>;
2197: def : Pat<(loongarch_vshuf4i_d v2i64:$vj, v2i64:$vk, immZExt8:$ui8),
2198:           (VSHUF4I_D v2i64:$vj, v2i64:$vk, immZExt8:$ui8)>;
2199: def : Pat<(loongarch_vshuf4i_d v2f64:$vj, v2f64:$vk, immZExt8:$ui8),
2200:           (VSHUF4I_D v2f64:$vj, v2f64:$vk, immZExt8:$ui8)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2201-2240 / 第 2201-2240 行
```tablegen
2201: 
2202: // VPERMI_W
2203: def : Pat<(loongarch_vpermi v4i32:$vj, v4i32:$vk, immZExt8:$ui8),
2204:           (VPERMI_W v4i32:$vj, v4i32:$vk, immZExt8:$ui8)>;
2205: def : Pat<(loongarch_vpermi v4f32:$vj, v4f32:$vk, immZExt8:$ui8),
2206:           (VPERMI_W v4f32:$vj, v4f32:$vk, immZExt8:$ui8)>;
2207: 
2208: // VREPLVEI_{B/H/W/D}
2209: def : Pat<(loongarch_vreplvei v16i8:$vj, immZExt4:$ui4),
2210:           (VREPLVEI_B v16i8:$vj, immZExt4:$ui4)>;
2211: def : Pat<(loongarch_vreplvei v8i16:$vj, immZExt3:$ui3),
2212:           (VREPLVEI_H v8i16:$vj, immZExt3:$ui3)>;
2213: def : Pat<(loongarch_vreplvei v4i32:$vj, immZExt2:$ui2),
2214:           (VREPLVEI_W v4i32:$vj, immZExt2:$ui2)>;
2215: def : Pat<(loongarch_vreplvei v2i64:$vj, immZExt1:$ui1),
2216:           (VREPLVEI_D v2i64:$vj, immZExt1:$ui1)>;
2217: def : Pat<(loongarch_vreplvei v4f32:$vj, immZExt2:$ui2),
2218:           (VREPLVEI_W v4f32:$vj, immZExt2:$ui2)>;
2219: def : Pat<(loongarch_vreplvei v2f64:$vj, immZExt1:$ui1),
2220:           (VREPLVEI_D v2f64:$vj, immZExt1:$ui1)>;
2221: 
2222: // VREPLVEI_{W/D}
2223: def : Pat<(lsxsplatf32 FPR32:$fj),
2224:           (VREPLVEI_W (SUBREG_TO_REG FPR32:$fj, sub_32), 0)>;
2225: def : Pat<(lsxsplatf64 FPR64:$fj),
2226:           (VREPLVEI_D (SUBREG_TO_REG FPR64:$fj, sub_64), 0)>;
2227: 
2228: defm : VstelmPat<truncstorei8, v16i8, VSTELM_B, simm8, uimm4>;
2229: defm : VstelmPat<truncstorei16, v8i16, VSTELM_H, simm8_lsl1, uimm3>;
2230: defm : VstelmPat<truncstorei32, v4i32, VSTELM_W, simm8_lsl2, uimm2>;
2231: defm : VstelmPat<store, v2i64, VSTELM_D, simm8_lsl3, uimm1>;
2232: defm : VstelmPat<store, v4f32, VSTELM_W, simm8_lsl2, uimm2, f32>;
2233: defm : VstelmPat<store, v2f64, VSTELM_D, simm8_lsl3, uimm1, f64>;
2234: 
2235: // Loads/Stores
2236: foreach vt = [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64] in {
2237:   defm : LdPat<load, VLD, vt>;
2238:   def  : RegRegLdPat<load, VLDX, vt>;
2239:   defm : StPat<store, VST, LSX128, vt>;
2240:   def  : RegRegStPat<store, VSTX, LSX128, vt>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2241-2280 / 第 2241-2280 行
```tablegen
2241: }
2242: 
2243: // Bitcast float/double element extracted from vector to integer.
2244: def : Pat<(loongarch_movfr2gr_s_la64 (f32 (vector_extract v4f32:$vj, uimm2:$imm))),
2245:           (VPICKVE2GR_W v4f32:$vj, uimm2:$imm)>;
2246: def : Pat<(i64 (bitconvert (f64 (vector_extract v2f64:$vj, uimm1:$imm)))),
2247:           (VPICKVE2GR_D v2f64:$vj, uimm1:$imm)>;
2248: 
2249: // Vector extraction with constant index.
2250: def : Pat<(GRLenVT (vector_extract v16i8:$vj, uimm4:$imm)),
2251:           (VPICKVE2GR_B v16i8:$vj, uimm4:$imm)>;
2252: def : Pat<(GRLenVT (vector_extract v8i16:$vj, uimm3:$imm)),
2253:           (VPICKVE2GR_H v8i16:$vj, uimm3:$imm)>;
2254: def : Pat<(GRLenVT (vector_extract v4i32:$vj, uimm2:$imm)),
2255:           (VPICKVE2GR_W v4i32:$vj, uimm2:$imm)>;
2256: def : Pat<(i64 (vector_extract v2i64:$vj, uimm1:$imm)),
2257:           (VPICKVE2GR_D v2i64:$vj, uimm1:$imm)>;
2258: def : Pat<(f32 (vector_extract v4f32:$vj, uimm2:$imm)),
2259:           (f32 (EXTRACT_SUBREG (VREPLVEI_W v4f32:$vj, uimm2:$imm), sub_32))>;
2260: def : Pat<(f64 (vector_extract v2f64:$vj, uimm1:$imm)),
2261:           (f64 (EXTRACT_SUBREG (VREPLVEI_D v2f64:$vj, uimm1:$imm), sub_64))>;
2262: 
2263: // Vector extraction with variable index.
2264: def : Pat<(GRLenVT (vector_extract v16i8:$vj, GRLenVT:$rk)),
2265:           (SRAI_W (COPY_TO_REGCLASS (f32 (EXTRACT_SUBREG (VREPLVE_B v16i8:$vj,
2266:                                                                     GRLenVT:$rk),
2267:                                                          sub_32)),
2268:                                     GPR), (GRLenVT 24))>;
2269: def : Pat<(GRLenVT (vector_extract v8i16:$vj, GRLenVT:$rk)),
2270:           (SRAI_W (COPY_TO_REGCLASS (f32 (EXTRACT_SUBREG (VREPLVE_H v8i16:$vj,
2271:                                                                     GRLenVT:$rk),
2272:                                                          sub_32)),
2273:                                     GPR), (GRLenVT 16))>;
2274: def : Pat<(GRLenVT (vector_extract v4i32:$vj, GRLenVT:$rk)),
2275:           (COPY_TO_REGCLASS (f32 (EXTRACT_SUBREG (VREPLVE_W v4i32:$vj, GRLenVT:$rk),
2276:                                                  sub_32)),
2277:                             GPR)>;
2278: def : Pat<(GRLenVT (vector_extract v2i64:$vj, GRLenVT:$rk)),
2279:           (COPY_TO_REGCLASS (f64 (EXTRACT_SUBREG (VREPLVE_D v2i64:$vj, GRLenVT:$rk),
2280:                                                  sub_64)),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2281-2320 / 第 2281-2320 行
```tablegen
2281:                             GPR)>;
2282: def : Pat<(f32 (vector_extract v4f32:$vj, GRLenVT:$rk)),
2283:           (f32 (EXTRACT_SUBREG (VREPLVE_W v4f32:$vj, GRLenVT:$rk), sub_32))>;
2284: def : Pat<(f64 (vector_extract v2f64:$vj, GRLenVT:$rk)),
2285:           (f64 (EXTRACT_SUBREG (VREPLVE_D v2f64:$vj, GRLenVT:$rk), sub_64))>;
2286: 
2287: // vselect
2288: def : Pat<(v16i8 (vselect LSX128:$vd, (v16i8 (SplatPat_uimm8 uimm8:$imm)),
2289:                           LSX128:$vj)),
2290:           (VBITSELI_B LSX128:$vd, LSX128:$vj, uimm8:$imm)>;
2291: foreach vt = [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64] in
2292:   def  : Pat<(vt (vselect LSX128:$va, LSX128:$vk, LSX128:$vj)),
2293:              (VBITSEL_V LSX128:$vj, LSX128:$vk, LSX128:$va)>;
2294: 
2295: // fneg
2296: def : Pat<(fneg (v4f32 LSX128:$vj)), (VBITREVI_W LSX128:$vj, 31)>;
2297: def : Pat<(fneg (v2f64 LSX128:$vj)), (VBITREVI_D LSX128:$vj, 63)>;
2298: 
2299: // VFFINT_{S_W/D_L}
2300: def : Pat<(v4f32 (sint_to_fp v4i32:$vj)), (VFFINT_S_W v4i32:$vj)>;
2301: def : Pat<(v2f64 (sint_to_fp v2i64:$vj)), (VFFINT_D_L v2i64:$vj)>;
2302: 
2303: // VFFINT_{S_WU/D_LU}
2304: def : Pat<(v4f32 (uint_to_fp v4i32:$vj)), (VFFINT_S_WU v4i32:$vj)>;
2305: def : Pat<(v2f64 (uint_to_fp v2i64:$vj)), (VFFINT_D_LU v2i64:$vj)>;
2306: 
2307: // VFTINTRZ_{W_S/L_D}
2308: def : Pat<(v4i32 (fp_to_sint v4f32:$vj)), (VFTINTRZ_W_S v4f32:$vj)>;
2309: def : Pat<(v2i64 (fp_to_sint v2f64:$vj)), (VFTINTRZ_L_D v2f64:$vj)>;
2310: 
2311: // VFTINTRZ_{W_SU/L_DU}
2312: def : Pat<(v4i32 (fp_to_uint v4f32:$vj)), (VFTINTRZ_WU_S v4f32:$vj)>;
2313: def : Pat<(v2i64 (fp_to_uint v2f64:$vj)), (VFTINTRZ_LU_D v2f64:$vj)>;
2314: 
2315: // Vector loads floating-point constants
2316: def : Pat<(f32 f32imm_vldi:$in),
2317:           (f32 (EXTRACT_SUBREG (VLDI (to_f32imm_vldi f32imm_vldi:$in)), sub_32))>;
2318: def : Pat<(f64 f64imm_vldi:$in),
2319:           (f64 (EXTRACT_SUBREG (VLDI (to_f64imm_vldi f64imm_vldi:$in)), sub_64))>;
2320: 
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2321-2360 / 第 2321-2360 行
```tablegen
2321: // VAVG_{B/H/W/D/BU/HU/WU/DU}, VAVGR_{B/H/W/D/BU/HU/WU/DU}
2322: defm : VAvgPat<sra, "VAVG_B", v16i8>;
2323: defm : VAvgPat<sra, "VAVG_H", v8i16>;
2324: defm : VAvgPat<sra, "VAVG_W", v4i32>;
2325: defm : VAvgPat<sra, "VAVG_D", v2i64>;
2326: defm : VAvgPat<srl, "VAVG_BU", v16i8>;
2327: defm : VAvgPat<srl, "VAVG_HU", v8i16>;
2328: defm : VAvgPat<srl, "VAVG_WU", v4i32>;
2329: defm : VAvgPat<srl, "VAVG_DU", v2i64>;
2330: defm : VAvgrPat<sra, "VAVGR_B", v16i8>;
2331: defm : VAvgrPat<sra, "VAVGR_H", v8i16>;
2332: defm : VAvgrPat<sra, "VAVGR_W", v4i32>;
2333: defm : VAvgrPat<sra, "VAVGR_D", v2i64>;
2334: defm : VAvgrPat<srl, "VAVGR_BU", v16i8>;
2335: defm : VAvgrPat<srl, "VAVGR_HU", v8i16>;
2336: defm : VAvgrPat<srl, "VAVGR_WU", v4i32>;
2337: defm : VAvgrPat<srl, "VAVGR_DU", v2i64>;
2338: defm : PatVrVr<avgfloors, "VAVG">;
2339: defm : PatVrVr<avgceils, "VAVGR">;
2340: defm : PatVrVrU<avgflooru, "VAVG">;
2341: defm : PatVrVrU<avgceilu, "VAVGR">;
2342: 
2343: // abs
2344: def : Pat<(abs v16i8:$vj), (VSIGNCOV_B v16i8:$vj, v16i8:$vj)>;
2345: def : Pat<(abs v8i16:$vj), (VSIGNCOV_H v8i16:$vj, v8i16:$vj)>;
2346: def : Pat<(abs v4i32:$vj), (VSIGNCOV_W v4i32:$vj, v4i32:$vj)>;
2347: def : Pat<(abs v2i64:$vj), (VSIGNCOV_D v2i64:$vj, v2i64:$vj)>;
2348: 
2349: // VABSD_{B/H/W/D}[U]
2350: defm : PatVrVr<abds, "VABSD">;
2351: defm : PatVrVrU<abdu, "VABSD">;
2352: 
2353: // VADDA_{B/H/W/D}
2354: def : Pat<(add (v16i8 (abs v16i8:$vj)), (v16i8 (abs v16i8:$vk))),
2355:           (VADDA_B v16i8:$vj, v16i8:$vk)>;
2356: def : Pat<(add (v8i16 (abs v8i16:$vj)), (v8i16 (abs v8i16:$vk))),
2357:           (VADDA_H v8i16:$vj, v8i16:$vk)>;
2358: def : Pat<(add (v4i32 (abs v4i32:$vj)), (v4i32 (abs v4i32:$vk))),
2359:           (VADDA_W v4i32:$vj, v4i32:$vk)>;
2360: def : Pat<(add (v2i64 (abs v2i64:$vj)), (v2i64 (abs v2i64:$vk))),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2361-2400 / 第 2361-2400 行
```tablegen
2361:           (VADDA_D v2i64:$vj, v2i64:$vk)>;
2362: 
2363: // VSADD_{B/H/W/D}[U], VSSUB_{B/H/W/D}[U]
2364: defm : PatVrVr<saddsat, "VSADD">;
2365: defm : PatVrVr<ssubsat, "VSSUB">;
2366: defm : PatVrVrU<uaddsat, "VSADD">;
2367: defm : PatVrVrU<usubsat, "VSSUB">;
2368: 
2369: // Vector mask set by condition
2370: def : Pat<(loongarch_vmskltz (v16i8 LSX128:$vj)), (PseudoVMSKLTZ_B LSX128:$vj)>;
2371: def : Pat<(loongarch_vmskltz (v8i16 LSX128:$vj)), (PseudoVMSKLTZ_H LSX128:$vj)>;
2372: def : Pat<(loongarch_vmskltz (v4i32 LSX128:$vj)), (PseudoVMSKLTZ_W LSX128:$vj)>;
2373: def : Pat<(loongarch_vmskltz (v2i64 LSX128:$vj)), (PseudoVMSKLTZ_D LSX128:$vj)>;
2374: def : Pat<(loongarch_vmskgez (v16i8 LSX128:$vj)), (PseudoVMSKGEZ_B LSX128:$vj)>;
2375: def : Pat<(loongarch_vmskeqz (v16i8 LSX128:$vj)), (PseudoVMSKEQZ_B LSX128:$vj)>;
2376: def : Pat<(loongarch_vmsknez (v16i8 LSX128:$vj)), (PseudoVMSKNEZ_B LSX128:$vj)>;
2377: 
2378: } // Predicates = [HasExtLSX]
2379: 
2380: /// Intrinsic pattern
2381: 
2382: class deriveLSXIntrinsic<string Inst> {
2383:   Intrinsic ret = !cast<Intrinsic>(!tolower("int_loongarch_lsx_"#Inst));
2384: }
2385: 
2386: let Predicates = [HasExtLSX] in {
2387: 
2388: // vty: v16i8/v8i16/v4i32/v2i64
2389: // Pat<(Intrinsic vty:$vj, vty:$vk),
2390: //     (LAInst vty:$vj, vty:$vk)>;
2391: foreach Inst = ["VSADD_B", "VSADD_BU", "VSSUB_B", "VSSUB_BU",
2392:                 "VHADDW_H_B", "VHADDW_HU_BU", "VHSUBW_H_B", "VHSUBW_HU_BU",
2393:                 "VADDWEV_H_B", "VADDWOD_H_B", "VSUBWEV_H_B", "VSUBWOD_H_B",
2394:                 "VADDWEV_H_BU", "VADDWOD_H_BU", "VSUBWEV_H_BU", "VSUBWOD_H_BU",
2395:                 "VADDWEV_H_BU_B", "VADDWOD_H_BU_B",
2396:                 "VAVG_B", "VAVG_BU", "VAVGR_B", "VAVGR_BU",
2397:                 "VABSD_B", "VABSD_BU", "VADDA_B", "VMUH_B", "VMUH_BU",
2398:                 "VMULWEV_H_B", "VMULWOD_H_B", "VMULWEV_H_BU", "VMULWOD_H_BU",
2399:                 "VMULWEV_H_BU_B", "VMULWOD_H_BU_B", "VSIGNCOV_B",
2400:                 "VANDN_V", "VORN_V", "VROTR_B", "VSRLR_B", "VSRAR_B",
```
- **EN**: This block declares or refines TableGen records such as `deriveLSXIntrinsic`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `deriveLSXIntrinsic` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2401-2440 / 第 2401-2440 行
```tablegen
2401:                 "VSEQ_B", "VSLE_B", "VSLE_BU", "VSLT_B", "VSLT_BU",
2402:                 "VPACKEV_B", "VPACKOD_B", "VPICKEV_B", "VPICKOD_B",
2403:                 "VILVL_B", "VILVH_B"] in
2404:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2405:                (v16i8 LSX128:$vj), (v16i8 LSX128:$vk)),
2406:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk)>;
2407: foreach Inst = ["VSADD_H", "VSADD_HU", "VSSUB_H", "VSSUB_HU",
2408:                 "VHADDW_W_H", "VHADDW_WU_HU", "VHSUBW_W_H", "VHSUBW_WU_HU",
2409:                 "VADDWEV_W_H", "VADDWOD_W_H", "VSUBWEV_W_H", "VSUBWOD_W_H",
2410:                 "VADDWEV_W_HU", "VADDWOD_W_HU", "VSUBWEV_W_HU", "VSUBWOD_W_HU",
2411:                 "VADDWEV_W_HU_H", "VADDWOD_W_HU_H",
2412:                 "VAVG_H", "VAVG_HU", "VAVGR_H", "VAVGR_HU",
2413:                 "VABSD_H", "VABSD_HU", "VADDA_H", "VMUH_H", "VMUH_HU",
2414:                 "VMULWEV_W_H", "VMULWOD_W_H", "VMULWEV_W_HU", "VMULWOD_W_HU",
2415:                 "VMULWEV_W_HU_H", "VMULWOD_W_HU_H", "VSIGNCOV_H", "VROTR_H",
2416:                 "VSRLR_H", "VSRAR_H", "VSRLN_B_H", "VSRAN_B_H", "VSRLRN_B_H",
2417:                 "VSRARN_B_H", "VSSRLN_B_H", "VSSRAN_B_H", "VSSRLN_BU_H",
2418:                 "VSSRAN_BU_H", "VSSRLRN_B_H", "VSSRARN_B_H", "VSSRLRN_BU_H",
2419:                 "VSSRARN_BU_H",
2420:                 "VSEQ_H", "VSLE_H", "VSLE_HU", "VSLT_H", "VSLT_HU",
2421:                 "VPACKEV_H", "VPACKOD_H", "VPICKEV_H", "VPICKOD_H",
2422:                 "VILVL_H", "VILVH_H"] in
2423:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2424:                (v8i16 LSX128:$vj), (v8i16 LSX128:$vk)),
2425:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk)>;
2426: foreach Inst = ["VSADD_W", "VSADD_WU", "VSSUB_W", "VSSUB_WU",
2427:                 "VHADDW_D_W", "VHADDW_DU_WU", "VHSUBW_D_W", "VHSUBW_DU_WU",
2428:                 "VADDWEV_D_W", "VADDWOD_D_W", "VSUBWEV_D_W", "VSUBWOD_D_W",
2429:                 "VADDWEV_D_WU", "VADDWOD_D_WU", "VSUBWEV_D_WU", "VSUBWOD_D_WU",
2430:                 "VADDWEV_D_WU_W", "VADDWOD_D_WU_W",
2431:                 "VAVG_W", "VAVG_WU", "VAVGR_W", "VAVGR_WU",
2432:                 "VABSD_W", "VABSD_WU", "VADDA_W", "VMUH_W", "VMUH_WU",
2433:                 "VMULWEV_D_W", "VMULWOD_D_W", "VMULWEV_D_WU", "VMULWOD_D_WU",
2434:                 "VMULWEV_D_WU_W", "VMULWOD_D_WU_W", "VSIGNCOV_W", "VROTR_W",
2435:                 "VSRLR_W", "VSRAR_W", "VSRLN_H_W", "VSRAN_H_W", "VSRLRN_H_W",
2436:                 "VSRARN_H_W", "VSSRLN_H_W", "VSSRAN_H_W", "VSSRLN_HU_W",
2437:                 "VSSRAN_HU_W", "VSSRLRN_H_W", "VSSRARN_H_W", "VSSRLRN_HU_W",
2438:                 "VSSRARN_HU_W",
2439:                 "VSEQ_W", "VSLE_W", "VSLE_WU", "VSLT_W", "VSLT_WU",
2440:                 "VPACKEV_W", "VPACKOD_W", "VPICKEV_W", "VPICKOD_W",
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2441-2480 / 第 2441-2480 行
```tablegen
2441:                 "VILVL_W", "VILVH_W"] in
2442:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2443:                (v4i32 LSX128:$vj), (v4i32 LSX128:$vk)),
2444:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk)>;
2445: foreach Inst = ["VADD_Q", "VSUB_Q",
2446:                 "VSADD_D", "VSADD_DU", "VSSUB_D", "VSSUB_DU",
2447:                 "VHADDW_Q_D", "VHADDW_QU_DU", "VHSUBW_Q_D", "VHSUBW_QU_DU",
2448:                 "VADDWEV_Q_D", "VADDWOD_Q_D", "VSUBWEV_Q_D", "VSUBWOD_Q_D",
2449:                 "VADDWEV_Q_DU", "VADDWOD_Q_DU", "VSUBWEV_Q_DU", "VSUBWOD_Q_DU",
2450:                 "VADDWEV_Q_DU_D", "VADDWOD_Q_DU_D",
2451:                 "VAVG_D", "VAVG_DU", "VAVGR_D", "VAVGR_DU",
2452:                 "VABSD_D", "VABSD_DU", "VADDA_D", "VMUH_D", "VMUH_DU",
2453:                 "VMULWEV_Q_D", "VMULWOD_Q_D", "VMULWEV_Q_DU", "VMULWOD_Q_DU",
2454:                 "VMULWEV_Q_DU_D", "VMULWOD_Q_DU_D", "VSIGNCOV_D", "VROTR_D",
2455:                 "VSRLR_D", "VSRAR_D", "VSRLN_W_D", "VSRAN_W_D", "VSRLRN_W_D",
2456:                 "VSRARN_W_D", "VSSRLN_W_D", "VSSRAN_W_D", "VSSRLN_WU_D",
2457:                 "VSSRAN_WU_D", "VSSRLRN_W_D", "VSSRARN_W_D", "VSSRLRN_WU_D",
2458:                 "VSSRARN_WU_D", "VFFINT_S_L",
2459:                 "VSEQ_D", "VSLE_D", "VSLE_DU", "VSLT_D", "VSLT_DU",
2460:                 "VPACKEV_D", "VPACKOD_D", "VPICKEV_D", "VPICKOD_D",
2461:                 "VILVL_D", "VILVH_D"] in
2462:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2463:                (v2i64 LSX128:$vj), (v2i64 LSX128:$vk)),
2464:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk)>;
2465: 
2466: // vty: v16i8/v8i16/v4i32/v2i64
2467: // Pat<(Intrinsic vty:$vd, vty:$vj, vty:$vk),
2468: //     (LAInst vty:$vd, vty:$vj, vty:$vk)>;
2469: foreach Inst = ["VMADDWEV_H_B", "VMADDWOD_H_B", "VMADDWEV_H_BU",
2470:                 "VMADDWOD_H_BU", "VMADDWEV_H_BU_B", "VMADDWOD_H_BU_B"] in
2471:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2472:                (v8i16 LSX128:$vd), (v16i8 LSX128:$vj), (v16i8 LSX128:$vk)),
2473:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2474: foreach Inst = ["VMADDWEV_W_H", "VMADDWOD_W_H", "VMADDWEV_W_HU",
2475:                 "VMADDWOD_W_HU", "VMADDWEV_W_HU_H", "VMADDWOD_W_HU_H"] in
2476:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2477:                (v4i32 LSX128:$vd), (v8i16 LSX128:$vj), (v8i16 LSX128:$vk)),
2478:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2479: foreach Inst = ["VMADDWEV_D_W", "VMADDWOD_D_W", "VMADDWEV_D_WU",
2480:                 "VMADDWOD_D_WU", "VMADDWEV_D_WU_W", "VMADDWOD_D_WU_W"] in
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2481-2520 / 第 2481-2520 行
```tablegen
2481:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2482:                (v2i64 LSX128:$vd), (v4i32 LSX128:$vj), (v4i32 LSX128:$vk)),
2483:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2484: foreach Inst = ["VMADDWEV_Q_D", "VMADDWOD_Q_D", "VMADDWEV_Q_DU",
2485:                 "VMADDWOD_Q_DU", "VMADDWEV_Q_DU_D", "VMADDWOD_Q_DU_D"] in
2486:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2487:                (v2i64 LSX128:$vd), (v2i64 LSX128:$vj), (v2i64 LSX128:$vk)),
2488:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2489: 
2490: // vty: v16i8/v8i16/v4i32/v2i64
2491: // Pat<(Intrinsic vty:$vj),
2492: //     (LAInst vty:$vj)>;
2493: foreach Inst = ["VEXTH_H_B", "VEXTH_HU_BU",
2494:                 "VMSKLTZ_B", "VMSKGEZ_B", "VMSKNZ_B",
2495:                 "VCLO_B"] in
2496:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v16i8 LSX128:$vj)),
2497:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2498: foreach Inst = ["VEXTH_W_H", "VEXTH_WU_HU", "VMSKLTZ_H",
2499:                 "VCLO_H", "VFCVTL_S_H", "VFCVTH_S_H"] in
2500:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v8i16 LSX128:$vj)),
2501:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2502: foreach Inst = ["VEXTH_D_W", "VEXTH_DU_WU", "VMSKLTZ_W",
2503:                 "VCLO_W", "VFFINT_S_W", "VFFINT_S_WU",
2504:                 "VFFINTL_D_W", "VFFINTH_D_W"] in
2505:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v4i32 LSX128:$vj)),
2506:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2507: foreach Inst = ["VEXTH_Q_D", "VEXTH_QU_DU", "VMSKLTZ_D",
2508:                 "VEXTL_Q_D", "VEXTL_QU_DU",
2509:                 "VCLO_D", "VFFINT_D_L", "VFFINT_D_LU"] in
2510:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v2i64 LSX128:$vj)),
2511:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2512: 
2513: // Pat<(Intrinsic timm:$imm)
2514: //     (LAInst timm:$imm)>;
2515: def : Pat<(int_loongarch_lsx_vldi timm:$imm),
2516:           (VLDI (to_valid_timm timm:$imm))>;
2517: foreach Inst = ["VREPLI_B", "VREPLI_H", "VREPLI_W", "VREPLI_D"] in
2518:   def : Pat<(deriveLSXIntrinsic<Inst>.ret timm:$imm),
2519:             (!cast<LAInst>("Pseudo"#Inst) (to_valid_timm timm:$imm))>;
2520: 
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2521-2560 / 第 2521-2560 行
```tablegen
2521: // vty: v16i8/v8i16/v4i32/v2i64
2522: // Pat<(Intrinsic vty:$vj, timm:$imm)
2523: //     (LAInst vty:$vj, timm:$imm)>;
2524: foreach Inst = ["VSAT_B", "VSAT_BU", "VNORI_B", "VROTRI_B", "VSLLWIL_H_B",
2525:                 "VSLLWIL_HU_BU", "VSRLRI_B", "VSRARI_B",
2526:                 "VSEQI_B", "VSLEI_B", "VSLEI_BU", "VSLTI_B", "VSLTI_BU",
2527:                 "VREPLVEI_B", "VBSLL_V", "VBSRL_V", "VSHUF4I_B"] in
2528:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v16i8 LSX128:$vj), timm:$imm),
2529:             (!cast<LAInst>(Inst) LSX128:$vj, (to_valid_timm timm:$imm))>;
2530: foreach Inst = ["VSAT_H", "VSAT_HU", "VROTRI_H", "VSLLWIL_W_H",
2531:                 "VSLLWIL_WU_HU", "VSRLRI_H", "VSRARI_H",
2532:                 "VSEQI_H", "VSLEI_H", "VSLEI_HU", "VSLTI_H", "VSLTI_HU",
2533:                 "VREPLVEI_H", "VSHUF4I_H"] in
2534:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v8i16 LSX128:$vj), timm:$imm),
2535:             (!cast<LAInst>(Inst) LSX128:$vj, (to_valid_timm timm:$imm))>;
2536: foreach Inst = ["VSAT_W", "VSAT_WU", "VROTRI_W", "VSLLWIL_D_W",
2537:                 "VSLLWIL_DU_WU", "VSRLRI_W", "VSRARI_W",
2538:                 "VSEQI_W", "VSLEI_W", "VSLEI_WU", "VSLTI_W", "VSLTI_WU",
2539:                 "VREPLVEI_W", "VSHUF4I_W"] in
2540:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v4i32 LSX128:$vj), timm:$imm),
2541:             (!cast<LAInst>(Inst) LSX128:$vj, (to_valid_timm timm:$imm))>;
2542: foreach Inst = ["VSAT_D", "VSAT_DU", "VROTRI_D", "VSRLRI_D", "VSRARI_D",
2543:                 "VSEQI_D", "VSLEI_D", "VSLEI_DU", "VSLTI_D", "VSLTI_DU",
2544:                 "VPICKVE2GR_D", "VPICKVE2GR_DU",
2545:                 "VREPLVEI_D"] in
2546:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v2i64 LSX128:$vj), timm:$imm),
2547:             (!cast<LAInst>(Inst) LSX128:$vj, (to_valid_timm timm:$imm))>;
2548: 
2549: // vty: v16i8/v8i16/v4i32/v2i64
2550: // Pat<(Intrinsic vty:$vd, vty:$vj, timm:$imm)
2551: //     (LAInst vty:$vd, vty:$vj, timm:$imm)>;
2552: foreach Inst = ["VSRLNI_B_H", "VSRANI_B_H", "VSRLRNI_B_H", "VSRARNI_B_H",
2553:                 "VSSRLNI_B_H", "VSSRANI_B_H", "VSSRLNI_BU_H", "VSSRANI_BU_H",
2554:                 "VSSRLRNI_B_H", "VSSRARNI_B_H", "VSSRLRNI_BU_H", "VSSRARNI_BU_H",
2555:                 "VFRSTPI_B", "VBITSELI_B", "VEXTRINS_B"] in
2556:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2557:                (v16i8 LSX128:$vd), (v16i8 LSX128:$vj), timm:$imm),
2558:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj,
2559:                (to_valid_timm timm:$imm))>;
2560: foreach Inst = ["VSRLNI_H_W", "VSRANI_H_W", "VSRLRNI_H_W", "VSRARNI_H_W",
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2561-2600 / 第 2561-2600 行
```tablegen
2561:                 "VSSRLNI_H_W", "VSSRANI_H_W", "VSSRLNI_HU_W", "VSSRANI_HU_W",
2562:                 "VSSRLRNI_H_W", "VSSRARNI_H_W", "VSSRLRNI_HU_W", "VSSRARNI_HU_W",
2563:                 "VFRSTPI_H", "VEXTRINS_H"] in
2564:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2565:                (v8i16 LSX128:$vd), (v8i16 LSX128:$vj), timm:$imm),
2566:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj,
2567:                (to_valid_timm timm:$imm))>;
2568: foreach Inst = ["VSRLNI_W_D", "VSRANI_W_D", "VSRLRNI_W_D", "VSRARNI_W_D",
2569:                 "VSSRLNI_W_D", "VSSRANI_W_D", "VSSRLNI_WU_D", "VSSRANI_WU_D",
2570:                 "VSSRLRNI_W_D", "VSSRARNI_W_D", "VSSRLRNI_WU_D", "VSSRARNI_WU_D",
2571:                 "VPERMI_W", "VEXTRINS_W"] in
2572:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2573:                (v4i32 LSX128:$vd), (v4i32 LSX128:$vj), timm:$imm),
2574:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj,
2575:                (to_valid_timm timm:$imm))>;
2576: foreach Inst = ["VSRLNI_D_Q", "VSRANI_D_Q", "VSRLRNI_D_Q", "VSRARNI_D_Q",
2577:                 "VSSRLNI_D_Q", "VSSRANI_D_Q", "VSSRLNI_DU_Q", "VSSRANI_DU_Q",
2578:                 "VSSRLRNI_D_Q", "VSSRARNI_D_Q", "VSSRLRNI_DU_Q", "VSSRARNI_DU_Q",
2579:                 "VSHUF4I_D", "VEXTRINS_D"] in
2580:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2581:                (v2i64 LSX128:$vd), (v2i64 LSX128:$vj), timm:$imm),
2582:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj,
2583:                (to_valid_timm timm:$imm))>;
2584: 
2585: // vty: v16i8/v8i16/v4i32/v2i64
2586: // Pat<(Intrinsic vty:$vd, vty:$vj, vty:$vk),
2587: //     (LAInst vty:$vd, vty:$vj, vty:$vk)>;
2588: foreach Inst = ["VFRSTP_B", "VBITSEL_V", "VSHUF_B"] in
2589:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2590:                (v16i8 LSX128:$vd), (v16i8 LSX128:$vj), (v16i8 LSX128:$vk)),
2591:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2592: foreach Inst = ["VFRSTP_H", "VSHUF_H"] in
2593:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2594:                (v8i16 LSX128:$vd), (v8i16 LSX128:$vj), (v8i16 LSX128:$vk)),
2595:             (!cast<LAInst>(Inst) LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2596: def : Pat<(int_loongarch_lsx_vshuf_w (v4i32 LSX128:$vd), (v4i32 LSX128:$vj),
2597:                                      (v4i32 LSX128:$vk)),
2598:           (VSHUF_W LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2599: def : Pat<(int_loongarch_lsx_vshuf_d (v2i64 LSX128:$vd), (v2i64 LSX128:$vj),
2600:                                      (v2i64 LSX128:$vk)),
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2601-2640 / 第 2601-2640 行
```tablegen
2601:           (VSHUF_D LSX128:$vd, LSX128:$vj, LSX128:$vk)>;
2602: 
2603: // vty: v4f32/v2f64
2604: // Pat<(Intrinsic vty:$vj, vty:$vk, vty:$va),
2605: //     (LAInst vty:$vj, vty:$vk, vty:$va)>;
2606: foreach Inst = ["VFMSUB_S", "VFNMADD_S", "VFNMSUB_S"] in
2607:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2608:                (v4f32 LSX128:$vj), (v4f32 LSX128:$vk), (v4f32 LSX128:$va)),
2609:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk, LSX128:$va)>;
2610: foreach Inst = ["VFMSUB_D", "VFNMADD_D", "VFNMSUB_D"] in
2611:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2612:                (v2f64 LSX128:$vj), (v2f64 LSX128:$vk), (v2f64 LSX128:$va)),
2613:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk, LSX128:$va)>;
2614: 
2615: // vty: v4f32/v2f64
2616: // Pat<(Intrinsic vty:$vj, vty:$vk),
2617: //     (LAInst vty:$vj, vty:$vk)>;
2618: foreach Inst = ["VFMAX_S", "VFMIN_S", "VFMAXA_S", "VFMINA_S", "VFCVT_H_S",
2619:                 "VFCMP_CAF_S", "VFCMP_CUN_S", "VFCMP_CEQ_S", "VFCMP_CUEQ_S",
2620:                 "VFCMP_CLT_S", "VFCMP_CULT_S", "VFCMP_CLE_S", "VFCMP_CULE_S",
2621:                 "VFCMP_CNE_S", "VFCMP_COR_S", "VFCMP_CUNE_S",
2622:                 "VFCMP_SAF_S", "VFCMP_SUN_S", "VFCMP_SEQ_S", "VFCMP_SUEQ_S",
2623:                 "VFCMP_SLT_S", "VFCMP_SULT_S", "VFCMP_SLE_S", "VFCMP_SULE_S",
2624:                 "VFCMP_SNE_S", "VFCMP_SOR_S", "VFCMP_SUNE_S"] in
2625:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2626:                (v4f32 LSX128:$vj), (v4f32 LSX128:$vk)),
2627:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk)>;
2628: foreach Inst = ["VFMAX_D", "VFMIN_D", "VFMAXA_D", "VFMINA_D", "VFCVT_S_D",
2629:                 "VFTINTRNE_W_D", "VFTINTRZ_W_D", "VFTINTRP_W_D", "VFTINTRM_W_D",
2630:                 "VFTINT_W_D",
2631:                 "VFCMP_CAF_D", "VFCMP_CUN_D", "VFCMP_CEQ_D", "VFCMP_CUEQ_D",
2632:                 "VFCMP_CLT_D", "VFCMP_CULT_D", "VFCMP_CLE_D", "VFCMP_CULE_D",
2633:                 "VFCMP_CNE_D", "VFCMP_COR_D", "VFCMP_CUNE_D",
2634:                 "VFCMP_SAF_D", "VFCMP_SUN_D", "VFCMP_SEQ_D", "VFCMP_SUEQ_D",
2635:                 "VFCMP_SLT_D", "VFCMP_SULT_D", "VFCMP_SLE_D", "VFCMP_SULE_D",
2636:                 "VFCMP_SNE_D", "VFCMP_SOR_D", "VFCMP_SUNE_D"] in
2637:   def : Pat<(deriveLSXIntrinsic<Inst>.ret
2638:                (v2f64 LSX128:$vj), (v2f64 LSX128:$vk)),
2639:             (!cast<LAInst>(Inst) LSX128:$vj, LSX128:$vk)>;
2640: 
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2641-2680 / 第 2641-2680 行
```tablegen
2641: // vty: v4f32/v2f64
2642: // Pat<(Intrinsic vty:$vj),
2643: //     (LAInst vty:$vj)>;
2644: foreach Inst = ["VFLOGB_S", "VFCLASS_S", "VFSQRT_S", "VFRECIP_S", "VFRSQRT_S",
2645:                 "VFRINT_S", "VFCVTL_D_S", "VFCVTH_D_S",
2646:                 "VFRINTRNE_S", "VFRINTRZ_S", "VFRINTRP_S", "VFRINTRM_S",
2647:                 "VFTINTRNE_W_S", "VFTINTRZ_W_S", "VFTINTRP_W_S", "VFTINTRM_W_S",
2648:                 "VFTINT_W_S", "VFTINTRZ_WU_S", "VFTINT_WU_S",
2649:                 "VFTINTRNEL_L_S", "VFTINTRNEH_L_S", "VFTINTRZL_L_S",
2650:                 "VFTINTRZH_L_S", "VFTINTRPL_L_S", "VFTINTRPH_L_S",
2651:                 "VFTINTRML_L_S", "VFTINTRMH_L_S", "VFTINTL_L_S",
2652:                 "VFTINTH_L_S"] in
2653:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v4f32 LSX128:$vj)),
2654:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2655: foreach Inst = ["VFLOGB_D", "VFCLASS_D", "VFSQRT_D", "VFRECIP_D", "VFRSQRT_D",
2656:                 "VFRINT_D",
2657:                 "VFRINTRNE_D", "VFRINTRZ_D", "VFRINTRP_D", "VFRINTRM_D",
2658:                 "VFTINTRNE_L_D", "VFTINTRZ_L_D", "VFTINTRP_L_D", "VFTINTRM_L_D",
2659:                 "VFTINT_L_D", "VFTINTRZ_LU_D", "VFTINT_LU_D"] in
2660:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v2f64 LSX128:$vj)),
2661:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2662: 
2663: // 128-Bit vector FP approximate reciprocal operation
2664: let Predicates = [HasFrecipe] in {
2665: foreach Inst = ["VFRECIPE_S", "VFRSQRTE_S"] in
2666:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v4f32 LSX128:$vj)),
2667:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2668: foreach Inst = ["VFRECIPE_D", "VFRSQRTE_D"] in
2669:   def : Pat<(deriveLSXIntrinsic<Inst>.ret (v2f64 LSX128:$vj)),
2670:             (!cast<LAInst>(Inst) LSX128:$vj)>;
2671: 
2672: def : Pat<(loongarch_vfrecipe v4f32:$src), 
2673:           (VFRECIPE_S v4f32:$src)>;
2674: def : Pat<(loongarch_vfrecipe v2f64:$src), 
2675:           (VFRECIPE_D v2f64:$src)>;
2676: def : Pat<(loongarch_vfrsqrte v4f32:$src), 
2677:           (VFRSQRTE_S v4f32:$src)>;
2678: def : Pat<(loongarch_vfrsqrte v2f64:$src), 
2679:           (VFRSQRTE_D v2f64:$src)>;
2680: }
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2681-2720 / 第 2681-2720 行
```tablegen
2681: 
2682: // Vector floating-point conversion
2683: def : Pat<(f32 (fceil FPR32:$fj)),
2684:           (f32 (EXTRACT_SUBREG (VFRINTRP_S (VREPLVEI_W
2685:                (SUBREG_TO_REG FPR32:$fj, sub_32), 0)), sub_32))>;
2686: def : Pat<(f64 (fceil FPR64:$fj)),
2687:           (f64 (EXTRACT_SUBREG (VFRINTRP_D (VREPLVEI_D
2688:                (SUBREG_TO_REG FPR64:$fj, sub_64), 0)), sub_64))>;
2689: def : Pat<(f32 (ffloor FPR32:$fj)),
2690:           (f32 (EXTRACT_SUBREG (VFRINTRM_S (VREPLVEI_W
2691:                (SUBREG_TO_REG FPR32:$fj, sub_32), 0)), sub_32))>;
2692: def : Pat<(f64 (ffloor FPR64:$fj)),
2693:           (f64 (EXTRACT_SUBREG (VFRINTRM_D (VREPLVEI_D
2694:                (SUBREG_TO_REG FPR64:$fj, sub_64), 0)), sub_64))>;
2695: def : Pat<(f32 (ftrunc FPR32:$fj)),
2696:           (f32 (EXTRACT_SUBREG (VFRINTRZ_S (VREPLVEI_W
2697:                (SUBREG_TO_REG FPR32:$fj, sub_32), 0)), sub_32))>;
2698: def : Pat<(f64 (ftrunc FPR64:$fj)),
2699:           (f64 (EXTRACT_SUBREG (VFRINTRZ_D (VREPLVEI_D
2700:                (SUBREG_TO_REG FPR64:$fj, sub_64), 0)), sub_64))>;
2701: def : Pat<(f32 (froundeven FPR32:$fj)),
2702:           (f32 (EXTRACT_SUBREG (VFRINTRNE_S (VREPLVEI_W
2703:                (SUBREG_TO_REG FPR32:$fj, sub_32), 0)), sub_32))>;
2704: def : Pat<(f64 (froundeven FPR64:$fj)),
2705:           (f64 (EXTRACT_SUBREG (VFRINTRNE_D (VREPLVEI_D
2706:                (SUBREG_TO_REG FPR64:$fj, sub_64), 0)), sub_64))>;
2707: 
2708: defm : PatVrF<fceil, "VFRINTRP">;
2709: defm : PatVrF<ffloor, "VFRINTRM">;
2710: defm : PatVrF<ftrunc, "VFRINTRZ">;
2711: defm : PatVrF<froundeven, "VFRINTRNE">;
2712: 
2713: def : Pat<(v4f32 (loongarch_vfcvt_s_d (v2f64 LSX128:$vj), (v2f64 LSX128:$vk))),
2714:           (VFCVT_S_D LSX128:$vj, LSX128:$vk)>;
2715: 
2716: def : Pat<(v2f64 (loongarch_vfcvtl_d_s (v4f32 LSX128:$vj))),
2717:           (VFCVTL_D_S LSX128:$vj)>;
2718: def : Pat<(v2f64 (loongarch_vfcvth_d_s (v4f32 LSX128:$vj))),
2719:           (VFCVTH_D_S LSX128:$vj)>;
2720: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2721-2760 / 第 2721-2760 行
```tablegen
2721: // load
2722: def : Pat<(int_loongarch_lsx_vld GPR:$rj, timm:$imm),
2723:           (VLD GPR:$rj, (to_valid_timm timm:$imm))>;
2724: def : Pat<(int_loongarch_lsx_vldx GPR:$rj, GPR:$rk),
2725:           (VLDX GPR:$rj, GPR:$rk)>;
2726: 
2727: // vldrepl
2728: def : Pat<(int_loongarch_lsx_vldrepl_b GPR:$rj, timm:$imm),
2729:           (VLDREPL_B GPR:$rj, (to_valid_timm timm:$imm))>;
2730: def : Pat<(int_loongarch_lsx_vldrepl_h GPR:$rj, timm:$imm),
2731:           (VLDREPL_H GPR:$rj, (to_valid_timm timm:$imm))>;
2732: def : Pat<(int_loongarch_lsx_vldrepl_w GPR:$rj, timm:$imm),
2733:           (VLDREPL_W GPR:$rj, (to_valid_timm timm:$imm))>;
2734: def : Pat<(int_loongarch_lsx_vldrepl_d GPR:$rj, timm:$imm),
2735:           (VLDREPL_D GPR:$rj, (to_valid_timm timm:$imm))>;
2736: 
2737: defm : VldreplPat<v16i8, VLDREPL_B, simm12_addlike>;
2738: defm : VldreplPat<v8i16, VLDREPL_H, simm11_lsl1>;
2739: defm : VldreplPat<v4i32, VLDREPL_W, simm10_lsl2>;
2740: defm : VldreplPat<v2i64, VLDREPL_D, simm9_lsl3>;
2741: defm : VldreplPat<v4f32, VLDREPL_W, simm10_lsl2>;
2742: defm : VldreplPat<v2f64, VLDREPL_D, simm9_lsl3>;
2743: 
2744: // store
2745: def : Pat<(int_loongarch_lsx_vst LSX128:$vd, GPR:$rj, timm:$imm),
2746:           (VST LSX128:$vd, GPR:$rj, (to_valid_timm timm:$imm))>;
2747: def : Pat<(int_loongarch_lsx_vstx LSX128:$vd, GPR:$rj, GPR:$rk),
2748:           (VSTX LSX128:$vd, GPR:$rj, GPR:$rk)>;
2749: 
2750: def : Pat<(int_loongarch_lsx_vstelm_b v16i8:$vd, GPR:$rj, timm:$imm, timm:$idx),
2751:           (VSTELM_B v16i8:$vd, GPR:$rj, (to_valid_timm timm:$imm),
2752:                     (to_valid_timm timm:$idx))>;
2753: def : Pat<(int_loongarch_lsx_vstelm_h v8i16:$vd, GPR:$rj, timm:$imm, timm:$idx),
2754:           (VSTELM_H v8i16:$vd, GPR:$rj, (to_valid_timm timm:$imm),
2755:                     (to_valid_timm timm:$idx))>;
2756: def : Pat<(int_loongarch_lsx_vstelm_w v4i32:$vd, GPR:$rj, timm:$imm, timm:$idx),
2757:           (VSTELM_W v4i32:$vd, GPR:$rj, (to_valid_timm timm:$imm),
2758:                     (to_valid_timm timm:$idx))>;
2759: def : Pat<(int_loongarch_lsx_vstelm_d v2i64:$vd, GPR:$rj, timm:$imm, timm:$idx),
2760:           (VSTELM_D v2i64:$vd, GPR:$rj, (to_valid_timm timm:$imm),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2761-2763 / 第 2761-2763 行
```tablegen
2761:                     (to_valid_timm timm:$idx))>;
2762: 
2763: } // Predicates = [HasExtLSX]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
