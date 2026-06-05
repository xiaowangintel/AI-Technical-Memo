# LoongArchLASXInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchLASXInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 LoongArch 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //=- LoongArchLASXInstrInfo.td - LoongArch LASX instructions -*- tablegen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the Advanced SIMD extension instructions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: def SDT_LoongArchXVPERM : SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisSameAs<0, 1>,
  14:                                                SDTCisVec<2>, SDTCisInt<2>]>;
  15: def SDT_LoongArchXVREPLVE0 : SDTypeProfile<1, 1, [SDTCisVec<0>,
  16:                                                   SDTCisSameAs<0, 1>]>;
  17: 
  18: // Target nodes.
  19: 
  20: // Vector Shuffle
  21: def loongarch_xvpermi: SDNode<"LoongArchISD::XVPERMI", SDT_LoongArchV1RUimm>;
  22: def loongarch_xvperm: SDNode<"LoongArchISD::XVPERM", SDT_LoongArchXVPERM>;
  23: def loongarch_xvreplve0: SDNode<"LoongArchISD::XVREPLVE0", SDT_LoongArchXVREPLVE0>;
  24: def loongarch_xvreplve0q: SDNode<"LoongArchISD::XVREPLVE0Q", SDT_LoongArchXVREPLVE0>;
  25: def loongarch_xvinsve0 : SDNode<"LoongArchISD::XVINSVE0", SDT_LoongArchV2RUimm>;
  26: 
  27: // Vector mask set by condition
  28: def loongarch_xvmskltz: SDNode<"LoongArchISD::XVMSKLTZ", SDT_LoongArchVMSKCOND>;
  29: def loongarch_xvmskgez: SDNode<"LoongArchISD::XVMSKGEZ", SDT_LoongArchVMSKCOND>;
  30: def loongarch_xvmskeqz: SDNode<"LoongArchISD::XVMSKEQZ", SDT_LoongArchVMSKCOND>;
  31: def loongarch_xvmsknez: SDNode<"LoongArchISD::XVMSKNEZ", SDT_LoongArchVMSKCOND>;
  32: 
  33: def lasxsplati8
  34:   : PatFrag<(ops node:$e0),
  35:             (v32i8 (build_vector node:$e0, node:$e0, node:$e0, node:$e0,
  36:                                  node:$e0, node:$e0, node:$e0, node:$e0,
  37:                                  node:$e0, node:$e0, node:$e0, node:$e0,
  38:                                  node:$e0, node:$e0, node:$e0, node:$e0,
  39:                                  node:$e0, node:$e0, node:$e0, node:$e0,
  40:                                  node:$e0, node:$e0, node:$e0, node:$e0,
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SDT_LoongArchXVPERM`, `SDT_LoongArchXVREPLVE0`, `loongarch_xvpermi:`, `loongarch_xvperm:`, `loongarch_xvreplve0:`, `loongarch_xvreplve0q:`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SDT_LoongArchXVPERM`, `SDT_LoongArchXVREPLVE0`, `loongarch_xvpermi:`, `loongarch_xvperm:`, `loongarch_xvreplve0:`, `loongarch_xvreplve0q:` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41:                                  node:$e0, node:$e0, node:$e0, node:$e0,
  42:                                  node:$e0, node:$e0, node:$e0, node:$e0))>;
  43: def lasxsplati16
  44:   : PatFrag<(ops node:$e0),
  45:             (v16i16 (build_vector node:$e0, node:$e0, node:$e0, node:$e0,
  46:                                   node:$e0, node:$e0, node:$e0, node:$e0,
  47:                                   node:$e0, node:$e0, node:$e0, node:$e0,
  48:                                   node:$e0, node:$e0, node:$e0, node:$e0))>;
  49: def lasxsplati32
  50:   : PatFrag<(ops node:$e0),
  51:             (v8i32 (build_vector node:$e0, node:$e0, node:$e0, node:$e0,
  52:                                  node:$e0, node:$e0, node:$e0, node:$e0))>;
  53: def lasxsplati64
  54:   : PatFrag<(ops node:$e0),
  55:             (v4i64 (build_vector node:$e0, node:$e0, node:$e0, node:$e0))>;
  56: def lasxsplatf32
  57:   : PatFrag<(ops node:$e0),
  58:             (v8f32 (build_vector node:$e0, node:$e0, node:$e0, node:$e0,
  59:                                  node:$e0, node:$e0, node:$e0, node:$e0))>;
  60: def lasxsplatf64
  61:   : PatFrag<(ops node:$e0),
  62:             (v4f64 (build_vector node:$e0, node:$e0, node:$e0, node:$e0))>;
  63: 
  64: //===----------------------------------------------------------------------===//
  65: // Instruction class templates
  66: //===----------------------------------------------------------------------===//
  67: 
  68: class LASX1RI13_XI<bits<32> op, Operand ImmOpnd = simm13>
  69:     : Fmt1RI13_XI<op, (outs LASX256:$xd), (ins ImmOpnd:$imm13), "$xd, $imm13">;
  70: 
  71: class LASX2R_XX<bits<32> op>
  72:     : Fmt2R_XX<op, (outs LASX256:$xd), (ins LASX256:$xj), "$xd, $xj">;
  73: 
  74: class LASX2R_XR<bits<32> op>
  75:     : Fmt2R_XR<op, (outs LASX256:$xd), (ins GPR:$rj), "$xd, $rj">;
  76: 
  77: class LASX2R_CX<bits<32> op>
  78:     : Fmt2R_CX<op, (outs CFR:$cd), (ins LASX256:$xj), "$cd, $xj">;
  79: 
  80: class LASX2RI1_XXI<bits<32> op, Operand ImmOpnd = uimm1>
```
- **EN**: This block declares or refines TableGen records such as `lasxsplati16`, `lasxsplati32`, `lasxsplati64`, `lasxsplatf32`, `lasxsplatf64`, `LASX1RI13_XI`.
- **CN**: 该代码块声明或细化了 `lasxsplati16`, `lasxsplati32`, `lasxsplati64`, `lasxsplatf32`, `lasxsplatf64`, `LASX1RI13_XI` 等 TableGen 记录。

### Lines 81-120 / 第 81-120 行
```tablegen
  81:     : Fmt2RI1_XXI<op, (outs LASX256:$xd), (ins LASX256:$xj, ImmOpnd:$imm1),
  82:                   "$xd, $xj, $imm1">;
  83: 
  84: class LASX2RI2_XXI<bits<32> op, Operand ImmOpnd = uimm2>
  85:     : Fmt2RI2_XXI<op, (outs LASX256:$xd), (ins LASX256:$xj, ImmOpnd:$imm2),
  86:                   "$xd, $xj, $imm2">;
  87: 
  88: class LASX2RI2_RXI<bits<32> op, Operand ImmOpnd = uimm2>
  89:     : Fmt2RI2_RXI<op, (outs GPR:$rd), (ins LASX256:$xj, ImmOpnd:$imm2),
  90:                   "$rd, $xj, $imm2">;
  91: 
  92: class LASX2RI3_XXI<bits<32> op, Operand ImmOpnd = uimm3>
  93:     : Fmt2RI3_XXI<op, (outs LASX256:$xd), (ins LASX256:$xj, ImmOpnd:$imm3),
  94:                   "$xd, $xj, $imm3">;
  95: 
  96: class LASX2RI3_RXI<bits<32> op, Operand ImmOpnd = uimm3>
  97:     : Fmt2RI3_RXI<op, (outs GPR:$rd), (ins LASX256:$xj, ImmOpnd:$imm3),
  98:                   "$rd, $xj, $imm3">;
  99: 
 100: class LASX2RI4_XXI<bits<32> op, Operand ImmOpnd = uimm4>
 101:     : Fmt2RI4_XXI<op, (outs LASX256:$xd), (ins LASX256:$xj, ImmOpnd:$imm4),
 102:                   "$xd, $xj, $imm4">;
 103: 
 104: class LASX2RI4_XRI<bits<32> op, Operand ImmOpnd = uimm4>
 105:     : Fmt2RI4_XRI<op, (outs LASX256:$xd), (ins GPR:$rj, ImmOpnd:$imm4),
 106:                   "$xd, $rj, $imm4">;
 107: 
 108: class LASX2RI4_RXI<bits<32> op, Operand ImmOpnd = uimm4>
 109:     : Fmt2RI4_RXI<op, (outs GPR:$rd), (ins LASX256:$xj, ImmOpnd:$imm4),
 110:                   "$rd, $xj, $imm4">;
 111: 
 112: class LASX2RI5_XXI<bits<32> op, Operand ImmOpnd = uimm5>
 113:     : Fmt2RI5_XXI<op, (outs LASX256:$xd), (ins LASX256:$xj, ImmOpnd:$imm5),
 114:                   "$xd, $xj, $imm5">;
 115: 
 116: class LASX2RI6_XXI<bits<32> op, Operand ImmOpnd = uimm6>
 117:     : Fmt2RI6_XXI<op, (outs LASX256:$xd), (ins LASX256:$xj, ImmOpnd:$imm6),
 118:                   "$xd, $xj, $imm6">;
 119: 
 120: class LASX2RI8_XXI<bits<32> op, Operand ImmOpnd = uimm8>
```
- **EN**: This block declares or refines TableGen records such as `LASX2RI2_XXI`, `LASX2RI2_RXI`, `LASX2RI3_XXI`, `LASX2RI3_RXI`, `LASX2RI4_XXI`, `LASX2RI4_XRI`.
- **CN**: 该代码块声明或细化了 `LASX2RI2_XXI`, `LASX2RI2_RXI`, `LASX2RI3_XXI`, `LASX2RI3_RXI`, `LASX2RI4_XXI`, `LASX2RI4_XRI` 等 TableGen 记录。

### Lines 121-160 / 第 121-160 行
```tablegen
 121:     : Fmt2RI8_XXI<op, (outs LASX256:$xd), (ins LASX256:$xj, ImmOpnd:$imm8),
 122:                   "$xd, $xj, $imm8">;
 123: 
 124: class LASX2RI8I2_XRII<bits<32> op, Operand ImmOpnd = simm8,
 125:                      Operand IdxOpnd = uimm2>
 126:     : Fmt2RI8I2_XRII<op, (outs),
 127:                      (ins LASX256:$xd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm2),
 128:                      "$xd, $rj, $imm8, $imm2">;
 129: class LASX2RI8I3_XRII<bits<32> op, Operand ImmOpnd = simm8,
 130:                      Operand IdxOpnd = uimm3>
 131:     : Fmt2RI8I3_XRII<op, (outs),
 132:                      (ins LASX256:$xd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm3),
 133:                      "$xd, $rj, $imm8, $imm3">;
 134: class LASX2RI8I4_XRII<bits<32> op, Operand ImmOpnd = simm8,
 135:                      Operand IdxOpnd = uimm4>
 136:     : Fmt2RI8I4_XRII<op, (outs),
 137:                      (ins LASX256:$xd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm4),
 138:                      "$xd, $rj, $imm8, $imm4">;
 139: class LASX2RI8I5_XRII<bits<32> op, Operand ImmOpnd = simm8,
 140:                      Operand IdxOpnd = uimm5>
 141:     : Fmt2RI8I5_XRII<op, (outs),
 142:                      (ins LASX256:$xd, GPR:$rj, ImmOpnd:$imm8, IdxOpnd:$imm5),
 143:                      "$xd, $rj, $imm8, $imm5">;
 144: 
 145: class LASX3R_XXX<bits<32> op>
 146:     : Fmt3R_XXX<op, (outs LASX256:$xd), (ins LASX256:$xj, LASX256:$xk),
 147:                 "$xd, $xj, $xk">;
 148: 
 149: class LASX3R_XXR<bits<32> op>
 150:     : Fmt3R_XXR<op, (outs LASX256:$xd), (ins LASX256:$xj, GPR:$rk),
 151:                 "$xd, $xj, $rk">;
 152: 
 153: class LASX4R_XXXX<bits<32> op>
 154:     : Fmt4R_XXXX<op, (outs LASX256:$xd),
 155:                  (ins LASX256:$xj, LASX256:$xk, LASX256:$xa),
 156:                  "$xd, $xj, $xk, $xa">;
 157: 
 158: let Constraints = "$xd = $dst" in {
 159: 
 160: class LASX2RI2_XXXI<bits<32> op, Operand ImmOpnd = uimm2>
```
- **EN**: This block declares or refines TableGen records such as `LASX2RI8I2_XRII`, `LASX2RI8I3_XRII`, `LASX2RI8I4_XRII`, `LASX2RI8I5_XRII`, `LASX3R_XXX`, `LASX3R_XXR`.
- **CN**: 该代码块声明或细化了 `LASX2RI8I2_XRII`, `LASX2RI8I3_XRII`, `LASX2RI8I4_XRII`, `LASX2RI8I5_XRII`, `LASX3R_XXX`, `LASX3R_XXR` 等 TableGen 记录。

### Lines 161-200 / 第 161-200 行
```tablegen
 161:     : Fmt2RI2_XXI<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, ImmOpnd:$imm2),
 162:                   "$xd, $xj, $imm2">;
 163: class LASX2RI3_XXXI<bits<32> op, Operand ImmOpnd = uimm3>
 164:     : Fmt2RI3_XXI<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, ImmOpnd:$imm3),
 165:                   "$xd, $xj, $imm3">;
 166: 
 167: class LASX2RI2_XXRI<bits<32> op, Operand ImmOpnd = uimm2>
 168:     : Fmt2RI2_XRI<op, (outs LASX256:$dst), (ins LASX256:$xd, GPR:$rj, ImmOpnd:$imm2),
 169:                   "$xd, $rj, $imm2">;
 170: class LASX2RI3_XXRI<bits<32> op, Operand ImmOpnd = uimm3>
 171:     : Fmt2RI3_XRI<op, (outs LASX256:$dst), (ins LASX256:$xd, GPR:$rj, ImmOpnd:$imm3),
 172:                   "$xd, $rj, $imm3">;
 173: 
 174: class LASX2RI4_XXXI<bits<32> op, Operand ImmOpnd = uimm4>
 175:     : Fmt2RI4_XXI<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, ImmOpnd:$imm4),
 176:                   "$xd, $xj, $imm4">;
 177: class LASX2RI5_XXXI<bits<32> op, Operand ImmOpnd = uimm5>
 178:     : Fmt2RI5_XXI<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, ImmOpnd:$imm5),
 179:                   "$xd, $xj, $imm5">;
 180: class LASX2RI6_XXXI<bits<32> op, Operand ImmOpnd = uimm6>
 181:     : Fmt2RI6_XXI<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, ImmOpnd:$imm6),
 182:                   "$xd, $xj, $imm6">;
 183: class LASX2RI7_XXXI<bits<32> op, Operand ImmOpnd = uimm7>
 184:     : Fmt2RI7_XXI<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, ImmOpnd:$imm7),
 185:                   "$xd, $xj, $imm7">;
 186: 
 187: class LASX2RI8_XXXI<bits<32> op, Operand ImmOpnd = uimm8>
 188:     : Fmt2RI8_XXI<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, ImmOpnd:$imm8),
 189:                   "$xd, $xj, $imm8">;
 190: 
 191: class LASX3R_XXXX<bits<32> op>
 192:     : Fmt3R_XXX<op, (outs LASX256:$dst), (ins LASX256:$xd, LASX256:$xj, LASX256:$xk),
 193:                 "$xd, $xj, $xk">;
 194: 
 195: } // Constraints = "$xd = $dst"
 196: 
 197: class LASX2RI9_Load<bits<32> op, Operand ImmOpnd = simm9_lsl3>
 198:     : Fmt2RI9_XRI<op, (outs LASX256:$xd), (ins GPR:$rj, ImmOpnd:$imm9),
 199:                   "$xd, $rj, $imm9">;
 200: class LASX2RI10_Load<bits<32> op, Operand ImmOpnd = simm10_lsl2>
```
- **EN**: This block declares or refines TableGen records such as `LASX2RI3_XXXI`, `LASX2RI2_XXRI`, `LASX2RI3_XXRI`, `LASX2RI4_XXXI`, `LASX2RI5_XXXI`, `LASX2RI6_XXXI`.
- **CN**: 该代码块声明或细化了 `LASX2RI3_XXXI`, `LASX2RI2_XXRI`, `LASX2RI3_XXRI`, `LASX2RI4_XXXI`, `LASX2RI5_XXXI`, `LASX2RI6_XXXI` 等 TableGen 记录。

### Lines 201-240 / 第 201-240 行
```tablegen
 201:     : Fmt2RI10_XRI<op, (outs LASX256:$xd), (ins GPR:$rj, ImmOpnd:$imm10),
 202:                   "$xd, $rj, $imm10">;
 203: class LASX2RI11_Load<bits<32> op, Operand ImmOpnd = simm11_lsl1>
 204:     : Fmt2RI11_XRI<op, (outs LASX256:$xd), (ins GPR:$rj, ImmOpnd:$imm11),
 205:                   "$xd, $rj, $imm11">;
 206: class LASX2RI12_Load<bits<32> op, Operand ImmOpnd = simm12_addlike>
 207:     : Fmt2RI12_XRI<op, (outs LASX256:$xd), (ins GPR:$rj, ImmOpnd:$imm12),
 208:                   "$xd, $rj, $imm12">;
 209: class LASX2RI12_Store<bits<32> op, Operand ImmOpnd = simm12_addlike>
 210:     : Fmt2RI12_XRI<op, (outs), (ins LASX256:$xd, GPR:$rj, ImmOpnd:$imm12),
 211:                   "$xd, $rj, $imm12">;
 212: 
 213: class LASX3R_Load<bits<32> op>
 214:     : Fmt3R_XRR<op, (outs LASX256:$xd), (ins GPR:$rj, GPR:$rk),
 215:                 "$xd, $rj, $rk">;
 216: class LASX3R_Store<bits<32> op>
 217:     : Fmt3R_XRR<op, (outs), (ins LASX256:$xd, GPR:$rj, GPR:$rk),
 218:                 "$xd, $rj, $rk">;
 219: 
 220: //===----------------------------------------------------------------------===//
 221: // Instructions
 222: //===----------------------------------------------------------------------===//
 223: 
 224: let hasSideEffects = 0, Predicates = [HasExtLASX] in {
 225: 
 226: let mayLoad = 0, mayStore = 0 in {
 227: def XVADD_B : LASX3R_XXX<0x740a0000>;
 228: def XVADD_H : LASX3R_XXX<0x740a8000>;
 229: def XVADD_W : LASX3R_XXX<0x740b0000>;
 230: def XVADD_D : LASX3R_XXX<0x740b8000>;
 231: def XVADD_Q : LASX3R_XXX<0x752d0000>;
 232: 
 233: def XVSUB_B : LASX3R_XXX<0x740c0000>;
 234: def XVSUB_H : LASX3R_XXX<0x740c8000>;
 235: def XVSUB_W : LASX3R_XXX<0x740d0000>;
 236: def XVSUB_D : LASX3R_XXX<0x740d8000>;
 237: def XVSUB_Q : LASX3R_XXX<0x752d8000>;
 238: 
 239: def XVADDI_BU : LASX2RI5_XXI<0x768a0000>;
 240: def XVADDI_HU : LASX2RI5_XXI<0x768a8000>;
```
- **EN**: This block declares or refines TableGen records such as `LASX2RI11_Load`, `LASX2RI12_Load`, `LASX2RI12_Store`, `LASX3R_Load`, `LASX3R_Store`, `XVADD_B`.
- **CN**: 该代码块声明或细化了 `LASX2RI11_Load`, `LASX2RI12_Load`, `LASX2RI12_Store`, `LASX3R_Load`, `LASX3R_Store`, `XVADD_B` 等 TableGen 记录。

### Lines 241-280 / 第 241-280 行
```tablegen
 241: def XVADDI_WU : LASX2RI5_XXI<0x768b0000>;
 242: def XVADDI_DU : LASX2RI5_XXI<0x768b8000>;
 243: 
 244: def XVSUBI_BU : LASX2RI5_XXI<0x768c0000>;
 245: def XVSUBI_HU : LASX2RI5_XXI<0x768c8000>;
 246: def XVSUBI_WU : LASX2RI5_XXI<0x768d0000>;
 247: def XVSUBI_DU : LASX2RI5_XXI<0x768d8000>;
 248: 
 249: def XVNEG_B : LASX2R_XX<0x769c3000>;
 250: def XVNEG_H : LASX2R_XX<0x769c3400>;
 251: def XVNEG_W : LASX2R_XX<0x769c3800>;
 252: def XVNEG_D : LASX2R_XX<0x769c3c00>;
 253: 
 254: def XVSADD_B : LASX3R_XXX<0x74460000>;
 255: def XVSADD_H : LASX3R_XXX<0x74468000>;
 256: def XVSADD_W : LASX3R_XXX<0x74470000>;
 257: def XVSADD_D : LASX3R_XXX<0x74478000>;
 258: def XVSADD_BU : LASX3R_XXX<0x744a0000>;
 259: def XVSADD_HU : LASX3R_XXX<0x744a8000>;
 260: def XVSADD_WU : LASX3R_XXX<0x744b0000>;
 261: def XVSADD_DU : LASX3R_XXX<0x744b8000>;
 262: 
 263: def XVSSUB_B : LASX3R_XXX<0x74480000>;
 264: def XVSSUB_H : LASX3R_XXX<0x74488000>;
 265: def XVSSUB_W : LASX3R_XXX<0x74490000>;
 266: def XVSSUB_D : LASX3R_XXX<0x74498000>;
 267: def XVSSUB_BU : LASX3R_XXX<0x744c0000>;
 268: def XVSSUB_HU : LASX3R_XXX<0x744c8000>;
 269: def XVSSUB_WU : LASX3R_XXX<0x744d0000>;
 270: def XVSSUB_DU : LASX3R_XXX<0x744d8000>;
 271: 
 272: def XVHADDW_H_B : LASX3R_XXX<0x74540000>;
 273: def XVHADDW_W_H : LASX3R_XXX<0x74548000>;
 274: def XVHADDW_D_W : LASX3R_XXX<0x74550000>;
 275: def XVHADDW_Q_D : LASX3R_XXX<0x74558000>;
 276: def XVHADDW_HU_BU : LASX3R_XXX<0x74580000>;
 277: def XVHADDW_WU_HU : LASX3R_XXX<0x74588000>;
 278: def XVHADDW_DU_WU : LASX3R_XXX<0x74590000>;
 279: def XVHADDW_QU_DU : LASX3R_XXX<0x74598000>;
 280: 
```
- **EN**: This block declares or refines TableGen records such as `XVADDI_WU`, `XVADDI_DU`, `XVSUBI_BU`, `XVSUBI_HU`, `XVSUBI_WU`, `XVSUBI_DU`.
- **CN**: 该代码块声明或细化了 `XVADDI_WU`, `XVADDI_DU`, `XVSUBI_BU`, `XVSUBI_HU`, `XVSUBI_WU`, `XVSUBI_DU` 等 TableGen 记录。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: def XVHSUBW_H_B : LASX3R_XXX<0x74560000>;
 282: def XVHSUBW_W_H : LASX3R_XXX<0x74568000>;
 283: def XVHSUBW_D_W : LASX3R_XXX<0x74570000>;
 284: def XVHSUBW_Q_D : LASX3R_XXX<0x74578000>;
 285: def XVHSUBW_HU_BU : LASX3R_XXX<0x745a0000>;
 286: def XVHSUBW_WU_HU : LASX3R_XXX<0x745a8000>;
 287: def XVHSUBW_DU_WU : LASX3R_XXX<0x745b0000>;
 288: def XVHSUBW_QU_DU : LASX3R_XXX<0x745b8000>;
 289: 
 290: def XVADDWEV_H_B : LASX3R_XXX<0x741e0000>;
 291: def XVADDWEV_W_H : LASX3R_XXX<0x741e8000>;
 292: def XVADDWEV_D_W : LASX3R_XXX<0x741f0000>;
 293: def XVADDWEV_Q_D : LASX3R_XXX<0x741f8000>;
 294: def XVADDWOD_H_B : LASX3R_XXX<0x74220000>;
 295: def XVADDWOD_W_H : LASX3R_XXX<0x74228000>;
 296: def XVADDWOD_D_W : LASX3R_XXX<0x74230000>;
 297: def XVADDWOD_Q_D : LASX3R_XXX<0x74238000>;
 298: 
 299: def XVSUBWEV_H_B : LASX3R_XXX<0x74200000>;
 300: def XVSUBWEV_W_H : LASX3R_XXX<0x74208000>;
 301: def XVSUBWEV_D_W : LASX3R_XXX<0x74210000>;
 302: def XVSUBWEV_Q_D : LASX3R_XXX<0x74218000>;
 303: def XVSUBWOD_H_B : LASX3R_XXX<0x74240000>;
 304: def XVSUBWOD_W_H : LASX3R_XXX<0x74248000>;
 305: def XVSUBWOD_D_W : LASX3R_XXX<0x74250000>;
 306: def XVSUBWOD_Q_D : LASX3R_XXX<0x74258000>;
 307: 
 308: def XVADDWEV_H_BU : LASX3R_XXX<0x742e0000>;
 309: def XVADDWEV_W_HU : LASX3R_XXX<0x742e8000>;
 310: def XVADDWEV_D_WU : LASX3R_XXX<0x742f0000>;
 311: def XVADDWEV_Q_DU : LASX3R_XXX<0x742f8000>;
 312: def XVADDWOD_H_BU : LASX3R_XXX<0x74320000>;
 313: def XVADDWOD_W_HU : LASX3R_XXX<0x74328000>;
 314: def XVADDWOD_D_WU : LASX3R_XXX<0x74330000>;
 315: def XVADDWOD_Q_DU : LASX3R_XXX<0x74338000>;
 316: 
 317: def XVSUBWEV_H_BU : LASX3R_XXX<0x74300000>;
 318: def XVSUBWEV_W_HU : LASX3R_XXX<0x74308000>;
 319: def XVSUBWEV_D_WU : LASX3R_XXX<0x74310000>;
 320: def XVSUBWEV_Q_DU : LASX3R_XXX<0x74318000>;
```
- **EN**: This block declares or refines TableGen records such as `XVHSUBW_H_B`, `XVHSUBW_W_H`, `XVHSUBW_D_W`, `XVHSUBW_Q_D`, `XVHSUBW_HU_BU`, `XVHSUBW_WU_HU`.
- **CN**: 该代码块声明或细化了 `XVHSUBW_H_B`, `XVHSUBW_W_H`, `XVHSUBW_D_W`, `XVHSUBW_Q_D`, `XVHSUBW_HU_BU`, `XVHSUBW_WU_HU` 等 TableGen 记录。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: def XVSUBWOD_H_BU : LASX3R_XXX<0x74340000>;
 322: def XVSUBWOD_W_HU : LASX3R_XXX<0x74348000>;
 323: def XVSUBWOD_D_WU : LASX3R_XXX<0x74350000>;
 324: def XVSUBWOD_Q_DU : LASX3R_XXX<0x74358000>;
 325: 
 326: def XVADDWEV_H_BU_B : LASX3R_XXX<0x743e0000>;
 327: def XVADDWEV_W_HU_H : LASX3R_XXX<0x743e8000>;
 328: def XVADDWEV_D_WU_W : LASX3R_XXX<0x743f0000>;
 329: def XVADDWEV_Q_DU_D : LASX3R_XXX<0x743f8000>;
 330: def XVADDWOD_H_BU_B : LASX3R_XXX<0x74400000>;
 331: def XVADDWOD_W_HU_H : LASX3R_XXX<0x74408000>;
 332: def XVADDWOD_D_WU_W : LASX3R_XXX<0x74410000>;
 333: def XVADDWOD_Q_DU_D : LASX3R_XXX<0x74418000>;
 334: 
 335: def XVAVG_B : LASX3R_XXX<0x74640000>;
 336: def XVAVG_H : LASX3R_XXX<0x74648000>;
 337: def XVAVG_W : LASX3R_XXX<0x74650000>;
 338: def XVAVG_D : LASX3R_XXX<0x74658000>;
 339: def XVAVG_BU : LASX3R_XXX<0x74660000>;
 340: def XVAVG_HU : LASX3R_XXX<0x74668000>;
 341: def XVAVG_WU : LASX3R_XXX<0x74670000>;
 342: def XVAVG_DU : LASX3R_XXX<0x74678000>;
 343: def XVAVGR_B : LASX3R_XXX<0x74680000>;
 344: def XVAVGR_H : LASX3R_XXX<0x74688000>;
 345: def XVAVGR_W : LASX3R_XXX<0x74690000>;
 346: def XVAVGR_D : LASX3R_XXX<0x74698000>;
 347: def XVAVGR_BU : LASX3R_XXX<0x746a0000>;
 348: def XVAVGR_HU : LASX3R_XXX<0x746a8000>;
 349: def XVAVGR_WU : LASX3R_XXX<0x746b0000>;
 350: def XVAVGR_DU : LASX3R_XXX<0x746b8000>;
 351: 
 352: def XVABSD_B : LASX3R_XXX<0x74600000>;
 353: def XVABSD_H : LASX3R_XXX<0x74608000>;
 354: def XVABSD_W : LASX3R_XXX<0x74610000>;
 355: def XVABSD_D : LASX3R_XXX<0x74618000>;
 356: def XVABSD_BU : LASX3R_XXX<0x74620000>;
 357: def XVABSD_HU : LASX3R_XXX<0x74628000>;
 358: def XVABSD_WU : LASX3R_XXX<0x74630000>;
 359: def XVABSD_DU : LASX3R_XXX<0x74638000>;
 360: 
```
- **EN**: This block declares or refines TableGen records such as `XVSUBWOD_H_BU`, `XVSUBWOD_W_HU`, `XVSUBWOD_D_WU`, `XVSUBWOD_Q_DU`, `XVADDWEV_H_BU_B`, `XVADDWEV_W_HU_H`.
- **CN**: 该代码块声明或细化了 `XVSUBWOD_H_BU`, `XVSUBWOD_W_HU`, `XVSUBWOD_D_WU`, `XVSUBWOD_Q_DU`, `XVADDWEV_H_BU_B`, `XVADDWEV_W_HU_H` 等 TableGen 记录。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: def XVADDA_B : LASX3R_XXX<0x745c0000>;
 362: def XVADDA_H : LASX3R_XXX<0x745c8000>;
 363: def XVADDA_W : LASX3R_XXX<0x745d0000>;
 364: def XVADDA_D : LASX3R_XXX<0x745d8000>;
 365: 
 366: def XVMAX_B : LASX3R_XXX<0x74700000>;
 367: def XVMAX_H : LASX3R_XXX<0x74708000>;
 368: def XVMAX_W : LASX3R_XXX<0x74710000>;
 369: def XVMAX_D : LASX3R_XXX<0x74718000>;
 370: def XVMAXI_B : LASX2RI5_XXI<0x76900000, simm5>;
 371: def XVMAXI_H : LASX2RI5_XXI<0x76908000, simm5>;
 372: def XVMAXI_W : LASX2RI5_XXI<0x76910000, simm5>;
 373: def XVMAXI_D : LASX2RI5_XXI<0x76918000, simm5>;
 374: def XVMAX_BU : LASX3R_XXX<0x74740000>;
 375: def XVMAX_HU : LASX3R_XXX<0x74748000>;
 376: def XVMAX_WU : LASX3R_XXX<0x74750000>;
 377: def XVMAX_DU : LASX3R_XXX<0x74758000>;
 378: def XVMAXI_BU : LASX2RI5_XXI<0x76940000>;
 379: def XVMAXI_HU : LASX2RI5_XXI<0x76948000>;
 380: def XVMAXI_WU : LASX2RI5_XXI<0x76950000>;
 381: def XVMAXI_DU : LASX2RI5_XXI<0x76958000>;
 382: 
 383: def XVMIN_B : LASX3R_XXX<0x74720000>;
 384: def XVMIN_H : LASX3R_XXX<0x74728000>;
 385: def XVMIN_W : LASX3R_XXX<0x74730000>;
 386: def XVMIN_D : LASX3R_XXX<0x74738000>;
 387: def XVMINI_B : LASX2RI5_XXI<0x76920000, simm5>;
 388: def XVMINI_H : LASX2RI5_XXI<0x76928000, simm5>;
 389: def XVMINI_W : LASX2RI5_XXI<0x76930000, simm5>;
 390: def XVMINI_D : LASX2RI5_XXI<0x76938000, simm5>;
 391: def XVMIN_BU : LASX3R_XXX<0x74760000>;
 392: def XVMIN_HU : LASX3R_XXX<0x74768000>;
 393: def XVMIN_WU : LASX3R_XXX<0x74770000>;
 394: def XVMIN_DU : LASX3R_XXX<0x74778000>;
 395: def XVMINI_BU : LASX2RI5_XXI<0x76960000>;
 396: def XVMINI_HU : LASX2RI5_XXI<0x76968000>;
 397: def XVMINI_WU : LASX2RI5_XXI<0x76970000>;
 398: def XVMINI_DU : LASX2RI5_XXI<0x76978000>;
 399: 
 400: def XVMUL_B : LASX3R_XXX<0x74840000>;
```
- **EN**: This block declares or refines TableGen records such as `XVADDA_B`, `XVADDA_H`, `XVADDA_W`, `XVADDA_D`, `XVMAX_B`, `XVMAX_H`.
- **CN**: 该代码块声明或细化了 `XVADDA_B`, `XVADDA_H`, `XVADDA_W`, `XVADDA_D`, `XVMAX_B`, `XVMAX_H` 等 TableGen 记录。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: def XVMUL_H : LASX3R_XXX<0x74848000>;
 402: def XVMUL_W : LASX3R_XXX<0x74850000>;
 403: def XVMUL_D : LASX3R_XXX<0x74858000>;
 404: 
 405: def XVMUH_B : LASX3R_XXX<0x74860000>;
 406: def XVMUH_H : LASX3R_XXX<0x74868000>;
 407: def XVMUH_W : LASX3R_XXX<0x74870000>;
 408: def XVMUH_D : LASX3R_XXX<0x74878000>;
 409: def XVMUH_BU : LASX3R_XXX<0x74880000>;
 410: def XVMUH_HU : LASX3R_XXX<0x74888000>;
 411: def XVMUH_WU : LASX3R_XXX<0x74890000>;
 412: def XVMUH_DU : LASX3R_XXX<0x74898000>;
 413: 
 414: def XVMULWEV_H_B : LASX3R_XXX<0x74900000>;
 415: def XVMULWEV_W_H : LASX3R_XXX<0x74908000>;
 416: def XVMULWEV_D_W : LASX3R_XXX<0x74910000>;
 417: def XVMULWEV_Q_D : LASX3R_XXX<0x74918000>;
 418: def XVMULWOD_H_B : LASX3R_XXX<0x74920000>;
 419: def XVMULWOD_W_H : LASX3R_XXX<0x74928000>;
 420: def XVMULWOD_D_W : LASX3R_XXX<0x74930000>;
 421: def XVMULWOD_Q_D : LASX3R_XXX<0x74938000>;
 422: def XVMULWEV_H_BU : LASX3R_XXX<0x74980000>;
 423: def XVMULWEV_W_HU : LASX3R_XXX<0x74988000>;
 424: def XVMULWEV_D_WU : LASX3R_XXX<0x74990000>;
 425: def XVMULWEV_Q_DU : LASX3R_XXX<0x74998000>;
 426: def XVMULWOD_H_BU : LASX3R_XXX<0x749a0000>;
 427: def XVMULWOD_W_HU : LASX3R_XXX<0x749a8000>;
 428: def XVMULWOD_D_WU : LASX3R_XXX<0x749b0000>;
 429: def XVMULWOD_Q_DU : LASX3R_XXX<0x749b8000>;
 430: def XVMULWEV_H_BU_B : LASX3R_XXX<0x74a00000>;
 431: def XVMULWEV_W_HU_H : LASX3R_XXX<0x74a08000>;
 432: def XVMULWEV_D_WU_W : LASX3R_XXX<0x74a10000>;
 433: def XVMULWEV_Q_DU_D : LASX3R_XXX<0x74a18000>;
 434: def XVMULWOD_H_BU_B : LASX3R_XXX<0x74a20000>;
 435: def XVMULWOD_W_HU_H : LASX3R_XXX<0x74a28000>;
 436: def XVMULWOD_D_WU_W : LASX3R_XXX<0x74a30000>;
 437: def XVMULWOD_Q_DU_D : LASX3R_XXX<0x74a38000>;
 438: 
 439: def XVMADD_B : LASX3R_XXXX<0x74a80000>;
 440: def XVMADD_H : LASX3R_XXXX<0x74a88000>;
```
- **EN**: This block declares or refines TableGen records such as `XVMUL_H`, `XVMUL_W`, `XVMUL_D`, `XVMUH_B`, `XVMUH_H`, `XVMUH_W`.
- **CN**: 该代码块声明或细化了 `XVMUL_H`, `XVMUL_W`, `XVMUL_D`, `XVMUH_B`, `XVMUH_H`, `XVMUH_W` 等 TableGen 记录。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: def XVMADD_W : LASX3R_XXXX<0x74a90000>;
 442: def XVMADD_D : LASX3R_XXXX<0x74a98000>;
 443: 
 444: def XVMSUB_B : LASX3R_XXXX<0x74aa0000>;
 445: def XVMSUB_H : LASX3R_XXXX<0x74aa8000>;
 446: def XVMSUB_W : LASX3R_XXXX<0x74ab0000>;
 447: def XVMSUB_D : LASX3R_XXXX<0x74ab8000>;
 448: 
 449: def XVMADDWEV_H_B : LASX3R_XXXX<0x74ac0000>;
 450: def XVMADDWEV_W_H : LASX3R_XXXX<0x74ac8000>;
 451: def XVMADDWEV_D_W : LASX3R_XXXX<0x74ad0000>;
 452: def XVMADDWEV_Q_D : LASX3R_XXXX<0x74ad8000>;
 453: def XVMADDWOD_H_B : LASX3R_XXXX<0x74ae0000>;
 454: def XVMADDWOD_W_H : LASX3R_XXXX<0x74ae8000>;
 455: def XVMADDWOD_D_W : LASX3R_XXXX<0x74af0000>;
 456: def XVMADDWOD_Q_D : LASX3R_XXXX<0x74af8000>;
 457: def XVMADDWEV_H_BU : LASX3R_XXXX<0x74b40000>;
 458: def XVMADDWEV_W_HU : LASX3R_XXXX<0x74b48000>;
 459: def XVMADDWEV_D_WU : LASX3R_XXXX<0x74b50000>;
 460: def XVMADDWEV_Q_DU : LASX3R_XXXX<0x74b58000>;
 461: def XVMADDWOD_H_BU : LASX3R_XXXX<0x74b60000>;
 462: def XVMADDWOD_W_HU : LASX3R_XXXX<0x74b68000>;
 463: def XVMADDWOD_D_WU : LASX3R_XXXX<0x74b70000>;
 464: def XVMADDWOD_Q_DU : LASX3R_XXXX<0x74b78000>;
 465: def XVMADDWEV_H_BU_B : LASX3R_XXXX<0x74bc0000>;
 466: def XVMADDWEV_W_HU_H : LASX3R_XXXX<0x74bc8000>;
 467: def XVMADDWEV_D_WU_W : LASX3R_XXXX<0x74bd0000>;
 468: def XVMADDWEV_Q_DU_D : LASX3R_XXXX<0x74bd8000>;
 469: def XVMADDWOD_H_BU_B : LASX3R_XXXX<0x74be0000>;
 470: def XVMADDWOD_W_HU_H : LASX3R_XXXX<0x74be8000>;
 471: def XVMADDWOD_D_WU_W : LASX3R_XXXX<0x74bf0000>;
 472: def XVMADDWOD_Q_DU_D : LASX3R_XXXX<0x74bf8000>;
 473: 
 474: def XVDIV_B : LASX3R_XXX<0x74e00000>;
 475: def XVDIV_H : LASX3R_XXX<0x74e08000>;
 476: def XVDIV_W : LASX3R_XXX<0x74e10000>;
 477: def XVDIV_D : LASX3R_XXX<0x74e18000>;
 478: def XVDIV_BU : LASX3R_XXX<0x74e40000>;
 479: def XVDIV_HU : LASX3R_XXX<0x74e48000>;
 480: def XVDIV_WU : LASX3R_XXX<0x74e50000>;
```
- **EN**: This block declares or refines TableGen records such as `XVMADD_W`, `XVMADD_D`, `XVMSUB_B`, `XVMSUB_H`, `XVMSUB_W`, `XVMSUB_D`.
- **CN**: 该代码块声明或细化了 `XVMADD_W`, `XVMADD_D`, `XVMSUB_B`, `XVMSUB_H`, `XVMSUB_W`, `XVMSUB_D` 等 TableGen 记录。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: def XVDIV_DU : LASX3R_XXX<0x74e58000>;
 482: 
 483: def XVMOD_B : LASX3R_XXX<0x74e20000>;
 484: def XVMOD_H : LASX3R_XXX<0x74e28000>;
 485: def XVMOD_W : LASX3R_XXX<0x74e30000>;
 486: def XVMOD_D : LASX3R_XXX<0x74e38000>;
 487: def XVMOD_BU : LASX3R_XXX<0x74e60000>;
 488: def XVMOD_HU : LASX3R_XXX<0x74e68000>;
 489: def XVMOD_WU : LASX3R_XXX<0x74e70000>;
 490: def XVMOD_DU : LASX3R_XXX<0x74e78000>;
 491: 
 492: def XVSAT_B : LASX2RI3_XXI<0x77242000>;
 493: def XVSAT_H : LASX2RI4_XXI<0x77244000>;
 494: def XVSAT_W : LASX2RI5_XXI<0x77248000>;
 495: def XVSAT_D : LASX2RI6_XXI<0x77250000>;
 496: def XVSAT_BU : LASX2RI3_XXI<0x77282000>;
 497: def XVSAT_HU : LASX2RI4_XXI<0x77284000>;
 498: def XVSAT_WU : LASX2RI5_XXI<0x77288000>;
 499: def XVSAT_DU : LASX2RI6_XXI<0x77290000>;
 500: 
 501: def XVEXTH_H_B : LASX2R_XX<0x769ee000>;
 502: def XVEXTH_W_H : LASX2R_XX<0x769ee400>;
 503: def XVEXTH_D_W : LASX2R_XX<0x769ee800>;
 504: def XVEXTH_Q_D : LASX2R_XX<0x769eec00>;
 505: def XVEXTH_HU_BU : LASX2R_XX<0x769ef000>;
 506: def XVEXTH_WU_HU : LASX2R_XX<0x769ef400>;
 507: def XVEXTH_DU_WU : LASX2R_XX<0x769ef800>;
 508: def XVEXTH_QU_DU : LASX2R_XX<0x769efc00>;
 509: 
 510: def VEXT2XV_H_B : LASX2R_XX<0x769f1000>;
 511: def VEXT2XV_W_B : LASX2R_XX<0x769f1400>;
 512: def VEXT2XV_D_B : LASX2R_XX<0x769f1800>;
 513: def VEXT2XV_W_H : LASX2R_XX<0x769f1c00>;
 514: def VEXT2XV_D_H : LASX2R_XX<0x769f2000>;
 515: def VEXT2XV_D_W : LASX2R_XX<0x769f2400>;
 516: def VEXT2XV_HU_BU : LASX2R_XX<0x769f2800>;
 517: def VEXT2XV_WU_BU : LASX2R_XX<0x769f2c00>;
 518: def VEXT2XV_DU_BU : LASX2R_XX<0x769f3000>;
 519: def VEXT2XV_WU_HU : LASX2R_XX<0x769f3400>;
 520: def VEXT2XV_DU_HU : LASX2R_XX<0x769f3800>;
```
- **EN**: This block declares or refines TableGen records such as `XVDIV_DU`, `XVMOD_B`, `XVMOD_H`, `XVMOD_W`, `XVMOD_D`, `XVMOD_BU`.
- **CN**: 该代码块声明或细化了 `XVDIV_DU`, `XVMOD_B`, `XVMOD_H`, `XVMOD_W`, `XVMOD_D`, `XVMOD_BU` 等 TableGen 记录。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: def VEXT2XV_DU_WU : LASX2R_XX<0x769f3c00>;
 522: 
 523: def XVHSELI_D : LASX2RI5_XXI<0x769f8000>;
 524: 
 525: def XVSIGNCOV_B : LASX3R_XXX<0x752e0000>;
 526: def XVSIGNCOV_H : LASX3R_XXX<0x752e8000>;
 527: def XVSIGNCOV_W : LASX3R_XXX<0x752f0000>;
 528: def XVSIGNCOV_D : LASX3R_XXX<0x752f8000>;
 529: 
 530: def XVMSKLTZ_B : LASX2R_XX<0x769c4000>;
 531: def XVMSKLTZ_H : LASX2R_XX<0x769c4400>;
 532: def XVMSKLTZ_W : LASX2R_XX<0x769c4800>;
 533: def XVMSKLTZ_D : LASX2R_XX<0x769c4c00>;
 534: 
 535: def XVMSKGEZ_B : LASX2R_XX<0x769c5000>;
 536: 
 537: def XVMSKNZ_B : LASX2R_XX<0x769c6000>;
 538: 
 539: def XVLDI : LASX1RI13_XI<0x77e00000>;
 540: 
 541: def XVAND_V : LASX3R_XXX<0x75260000>;
 542: def XVOR_V : LASX3R_XXX<0x75268000>;
 543: def XVXOR_V : LASX3R_XXX<0x75270000>;
 544: def XVNOR_V : LASX3R_XXX<0x75278000>;
 545: def XVANDN_V : LASX3R_XXX<0x75280000>;
 546: def XVORN_V : LASX3R_XXX<0x75288000>;
 547: 
 548: def XVANDI_B : LASX2RI8_XXI<0x77d00000>;
 549: def XVORI_B : LASX2RI8_XXI<0x77d40000>;
 550: def XVXORI_B : LASX2RI8_XXI<0x77d80000>;
 551: def XVNORI_B : LASX2RI8_XXI<0x77dc0000>;
 552: 
 553: def XVSLL_B : LASX3R_XXX<0x74e80000>;
 554: def XVSLL_H : LASX3R_XXX<0x74e88000>;
 555: def XVSLL_W : LASX3R_XXX<0x74e90000>;
 556: def XVSLL_D : LASX3R_XXX<0x74e98000>;
 557: def XVSLLI_B : LASX2RI3_XXI<0x772c2000>;
 558: def XVSLLI_H : LASX2RI4_XXI<0x772c4000>;
 559: def XVSLLI_W : LASX2RI5_XXI<0x772c8000>;
 560: def XVSLLI_D : LASX2RI6_XXI<0x772d0000>;
```
- **EN**: This block declares or refines TableGen records such as `VEXT2XV_DU_WU`, `XVHSELI_D`, `XVSIGNCOV_B`, `XVSIGNCOV_H`, `XVSIGNCOV_W`, `XVSIGNCOV_D`.
- **CN**: 该代码块声明或细化了 `VEXT2XV_DU_WU`, `XVHSELI_D`, `XVSIGNCOV_B`, `XVSIGNCOV_H`, `XVSIGNCOV_W`, `XVSIGNCOV_D` 等 TableGen 记录。

### Lines 561-600 / 第 561-600 行
```tablegen
 561: 
 562: def XVSRL_B : LASX3R_XXX<0x74ea0000>;
 563: def XVSRL_H : LASX3R_XXX<0x74ea8000>;
 564: def XVSRL_W : LASX3R_XXX<0x74eb0000>;
 565: def XVSRL_D : LASX3R_XXX<0x74eb8000>;
 566: def XVSRLI_B : LASX2RI3_XXI<0x77302000>;
 567: def XVSRLI_H : LASX2RI4_XXI<0x77304000>;
 568: def XVSRLI_W : LASX2RI5_XXI<0x77308000>;
 569: def XVSRLI_D : LASX2RI6_XXI<0x77310000>;
 570: 
 571: def XVSRA_B : LASX3R_XXX<0x74ec0000>;
 572: def XVSRA_H : LASX3R_XXX<0x74ec8000>;
 573: def XVSRA_W : LASX3R_XXX<0x74ed0000>;
 574: def XVSRA_D : LASX3R_XXX<0x74ed8000>;
 575: def XVSRAI_B : LASX2RI3_XXI<0x77342000>;
 576: def XVSRAI_H : LASX2RI4_XXI<0x77344000>;
 577: def XVSRAI_W : LASX2RI5_XXI<0x77348000>;
 578: def XVSRAI_D : LASX2RI6_XXI<0x77350000>;
 579: 
 580: def XVROTR_B : LASX3R_XXX<0x74ee0000>;
 581: def XVROTR_H : LASX3R_XXX<0x74ee8000>;
 582: def XVROTR_W : LASX3R_XXX<0x74ef0000>;
 583: def XVROTR_D : LASX3R_XXX<0x74ef8000>;
 584: def XVROTRI_B : LASX2RI3_XXI<0x76a02000>;
 585: def XVROTRI_H : LASX2RI4_XXI<0x76a04000>;
 586: def XVROTRI_W : LASX2RI5_XXI<0x76a08000>;
 587: def XVROTRI_D : LASX2RI6_XXI<0x76a10000>;
 588: 
 589: def XVSLLWIL_H_B : LASX2RI3_XXI<0x77082000>;
 590: def XVSLLWIL_W_H : LASX2RI4_XXI<0x77084000>;
 591: def XVSLLWIL_D_W : LASX2RI5_XXI<0x77088000>;
 592: def XVEXTL_Q_D : LASX2R_XX<0x77090000>;
 593: def XVSLLWIL_HU_BU : LASX2RI3_XXI<0x770c2000>;
 594: def XVSLLWIL_WU_HU : LASX2RI4_XXI<0x770c4000>;
 595: def XVSLLWIL_DU_WU : LASX2RI5_XXI<0x770c8000>;
 596: def XVEXTL_QU_DU : LASX2R_XX<0x770d0000>;
 597: 
 598: def XVSRLR_B : LASX3R_XXX<0x74f00000>;
 599: def XVSRLR_H : LASX3R_XXX<0x74f08000>;
 600: def XVSRLR_W : LASX3R_XXX<0x74f10000>;
```
- **EN**: This block declares or refines TableGen records such as `XVSRL_B`, `XVSRL_H`, `XVSRL_W`, `XVSRL_D`, `XVSRLI_B`, `XVSRLI_H`.
- **CN**: 该代码块声明或细化了 `XVSRL_B`, `XVSRL_H`, `XVSRL_W`, `XVSRL_D`, `XVSRLI_B`, `XVSRLI_H` 等 TableGen 记录。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: def XVSRLR_D : LASX3R_XXX<0x74f18000>;
 602: def XVSRLRI_B : LASX2RI3_XXI<0x76a42000>;
 603: def XVSRLRI_H : LASX2RI4_XXI<0x76a44000>;
 604: def XVSRLRI_W : LASX2RI5_XXI<0x76a48000>;
 605: def XVSRLRI_D : LASX2RI6_XXI<0x76a50000>;
 606: 
 607: def XVSRAR_B : LASX3R_XXX<0x74f20000>;
 608: def XVSRAR_H : LASX3R_XXX<0x74f28000>;
 609: def XVSRAR_W : LASX3R_XXX<0x74f30000>;
 610: def XVSRAR_D : LASX3R_XXX<0x74f38000>;
 611: def XVSRARI_B : LASX2RI3_XXI<0x76a82000>;
 612: def XVSRARI_H : LASX2RI4_XXI<0x76a84000>;
 613: def XVSRARI_W : LASX2RI5_XXI<0x76a88000>;
 614: def XVSRARI_D : LASX2RI6_XXI<0x76a90000>;
 615: 
 616: def XVSRLN_B_H : LASX3R_XXX<0x74f48000>;
 617: def XVSRLN_H_W : LASX3R_XXX<0x74f50000>;
 618: def XVSRLN_W_D : LASX3R_XXX<0x74f58000>;
 619: def XVSRAN_B_H : LASX3R_XXX<0x74f68000>;
 620: def XVSRAN_H_W : LASX3R_XXX<0x74f70000>;
 621: def XVSRAN_W_D : LASX3R_XXX<0x74f78000>;
 622: 
 623: def XVSRLNI_B_H : LASX2RI4_XXXI<0x77404000>;
 624: def XVSRLNI_H_W : LASX2RI5_XXXI<0x77408000>;
 625: def XVSRLNI_W_D : LASX2RI6_XXXI<0x77410000>;
 626: def XVSRLNI_D_Q : LASX2RI7_XXXI<0x77420000>;
 627: def XVSRANI_B_H : LASX2RI4_XXXI<0x77584000>;
 628: def XVSRANI_H_W : LASX2RI5_XXXI<0x77588000>;
 629: def XVSRANI_W_D : LASX2RI6_XXXI<0x77590000>;
 630: def XVSRANI_D_Q : LASX2RI7_XXXI<0x775a0000>;
 631: 
 632: def XVSRLRN_B_H : LASX3R_XXX<0x74f88000>;
 633: def XVSRLRN_H_W : LASX3R_XXX<0x74f90000>;
 634: def XVSRLRN_W_D : LASX3R_XXX<0x74f98000>;
 635: def XVSRARN_B_H : LASX3R_XXX<0x74fa8000>;
 636: def XVSRARN_H_W : LASX3R_XXX<0x74fb0000>;
 637: def XVSRARN_W_D : LASX3R_XXX<0x74fb8000>;
 638: 
 639: def XVSRLRNI_B_H : LASX2RI4_XXXI<0x77444000>;
 640: def XVSRLRNI_H_W : LASX2RI5_XXXI<0x77448000>;
```
- **EN**: This block declares or refines TableGen records such as `XVSRLR_D`, `XVSRLRI_B`, `XVSRLRI_H`, `XVSRLRI_W`, `XVSRLRI_D`, `XVSRAR_B`.
- **CN**: 该代码块声明或细化了 `XVSRLR_D`, `XVSRLRI_B`, `XVSRLRI_H`, `XVSRLRI_W`, `XVSRLRI_D`, `XVSRAR_B` 等 TableGen 记录。

### Lines 641-680 / 第 641-680 行
```tablegen
 641: def XVSRLRNI_W_D : LASX2RI6_XXXI<0x77450000>;
 642: def XVSRLRNI_D_Q : LASX2RI7_XXXI<0x77460000>;
 643: def XVSRARNI_B_H : LASX2RI4_XXXI<0x775c4000>;
 644: def XVSRARNI_H_W : LASX2RI5_XXXI<0x775c8000>;
 645: def XVSRARNI_W_D : LASX2RI6_XXXI<0x775d0000>;
 646: def XVSRARNI_D_Q : LASX2RI7_XXXI<0x775e0000>;
 647: 
 648: def XVSSRLN_B_H : LASX3R_XXX<0x74fc8000>;
 649: def XVSSRLN_H_W : LASX3R_XXX<0x74fd0000>;
 650: def XVSSRLN_W_D : LASX3R_XXX<0x74fd8000>;
 651: def XVSSRAN_B_H : LASX3R_XXX<0x74fe8000>;
 652: def XVSSRAN_H_W : LASX3R_XXX<0x74ff0000>;
 653: def XVSSRAN_W_D : LASX3R_XXX<0x74ff8000>;
 654: def XVSSRLN_BU_H : LASX3R_XXX<0x75048000>;
 655: def XVSSRLN_HU_W : LASX3R_XXX<0x75050000>;
 656: def XVSSRLN_WU_D : LASX3R_XXX<0x75058000>;
 657: def XVSSRAN_BU_H : LASX3R_XXX<0x75068000>;
 658: def XVSSRAN_HU_W : LASX3R_XXX<0x75070000>;
 659: def XVSSRAN_WU_D : LASX3R_XXX<0x75078000>;
 660: 
 661: def XVSSRLNI_B_H : LASX2RI4_XXXI<0x77484000>;
 662: def XVSSRLNI_H_W : LASX2RI5_XXXI<0x77488000>;
 663: def XVSSRLNI_W_D : LASX2RI6_XXXI<0x77490000>;
 664: def XVSSRLNI_D_Q : LASX2RI7_XXXI<0x774a0000>;
 665: def XVSSRANI_B_H : LASX2RI4_XXXI<0x77604000>;
 666: def XVSSRANI_H_W : LASX2RI5_XXXI<0x77608000>;
 667: def XVSSRANI_W_D : LASX2RI6_XXXI<0x77610000>;
 668: def XVSSRANI_D_Q : LASX2RI7_XXXI<0x77620000>;
 669: def XVSSRLNI_BU_H : LASX2RI4_XXXI<0x774c4000>;
 670: def XVSSRLNI_HU_W : LASX2RI5_XXXI<0x774c8000>;
 671: def XVSSRLNI_WU_D : LASX2RI6_XXXI<0x774d0000>;
 672: def XVSSRLNI_DU_Q : LASX2RI7_XXXI<0x774e0000>;
 673: def XVSSRANI_BU_H : LASX2RI4_XXXI<0x77644000>;
 674: def XVSSRANI_HU_W : LASX2RI5_XXXI<0x77648000>;
 675: def XVSSRANI_WU_D : LASX2RI6_XXXI<0x77650000>;
 676: def XVSSRANI_DU_Q : LASX2RI7_XXXI<0x77660000>;
 677: 
 678: def XVSSRLRN_B_H : LASX3R_XXX<0x75008000>;
 679: def XVSSRLRN_H_W : LASX3R_XXX<0x75010000>;
 680: def XVSSRLRN_W_D : LASX3R_XXX<0x75018000>;
```
- **EN**: This block declares or refines TableGen records such as `XVSRLRNI_W_D`, `XVSRLRNI_D_Q`, `XVSRARNI_B_H`, `XVSRARNI_H_W`, `XVSRARNI_W_D`, `XVSRARNI_D_Q`.
- **CN**: 该代码块声明或细化了 `XVSRLRNI_W_D`, `XVSRLRNI_D_Q`, `XVSRARNI_B_H`, `XVSRARNI_H_W`, `XVSRARNI_W_D`, `XVSRARNI_D_Q` 等 TableGen 记录。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: def XVSSRARN_B_H : LASX3R_XXX<0x75028000>;
 682: def XVSSRARN_H_W : LASX3R_XXX<0x75030000>;
 683: def XVSSRARN_W_D : LASX3R_XXX<0x75038000>;
 684: def XVSSRLRN_BU_H : LASX3R_XXX<0x75088000>;
 685: def XVSSRLRN_HU_W : LASX3R_XXX<0x75090000>;
 686: def XVSSRLRN_WU_D : LASX3R_XXX<0x75098000>;
 687: def XVSSRARN_BU_H : LASX3R_XXX<0x750a8000>;
 688: def XVSSRARN_HU_W : LASX3R_XXX<0x750b0000>;
 689: def XVSSRARN_WU_D : LASX3R_XXX<0x750b8000>;
 690: 
 691: def XVSSRLRNI_B_H : LASX2RI4_XXXI<0x77504000>;
 692: def XVSSRLRNI_H_W : LASX2RI5_XXXI<0x77508000>;
 693: def XVSSRLRNI_W_D : LASX2RI6_XXXI<0x77510000>;
 694: def XVSSRLRNI_D_Q : LASX2RI7_XXXI<0x77520000>;
 695: def XVSSRARNI_B_H : LASX2RI4_XXXI<0x77684000>;
 696: def XVSSRARNI_H_W : LASX2RI5_XXXI<0x77688000>;
 697: def XVSSRARNI_W_D : LASX2RI6_XXXI<0x77690000>;
 698: def XVSSRARNI_D_Q : LASX2RI7_XXXI<0x776a0000>;
 699: def XVSSRLRNI_BU_H : LASX2RI4_XXXI<0x77544000>;
 700: def XVSSRLRNI_HU_W : LASX2RI5_XXXI<0x77548000>;
 701: def XVSSRLRNI_WU_D : LASX2RI6_XXXI<0x77550000>;
 702: def XVSSRLRNI_DU_Q : LASX2RI7_XXXI<0x77560000>;
 703: def XVSSRARNI_BU_H : LASX2RI4_XXXI<0x776c4000>;
 704: def XVSSRARNI_HU_W : LASX2RI5_XXXI<0x776c8000>;
 705: def XVSSRARNI_WU_D : LASX2RI6_XXXI<0x776d0000>;
 706: def XVSSRARNI_DU_Q : LASX2RI7_XXXI<0x776e0000>;
 707: 
 708: def XVCLO_B : LASX2R_XX<0x769c0000>;
 709: def XVCLO_H : LASX2R_XX<0x769c0400>;
 710: def XVCLO_W : LASX2R_XX<0x769c0800>;
 711: def XVCLO_D : LASX2R_XX<0x769c0c00>;
 712: def XVCLZ_B : LASX2R_XX<0x769c1000>;
 713: def XVCLZ_H : LASX2R_XX<0x769c1400>;
 714: def XVCLZ_W : LASX2R_XX<0x769c1800>;
 715: def XVCLZ_D : LASX2R_XX<0x769c1c00>;
 716: 
 717: def XVPCNT_B : LASX2R_XX<0x769c2000>;
 718: def XVPCNT_H : LASX2R_XX<0x769c2400>;
 719: def XVPCNT_W : LASX2R_XX<0x769c2800>;
 720: def XVPCNT_D : LASX2R_XX<0x769c2c00>;
```
- **EN**: This block declares or refines TableGen records such as `XVSSRARN_B_H`, `XVSSRARN_H_W`, `XVSSRARN_W_D`, `XVSSRLRN_BU_H`, `XVSSRLRN_HU_W`, `XVSSRLRN_WU_D`.
- **CN**: 该代码块声明或细化了 `XVSSRARN_B_H`, `XVSSRARN_H_W`, `XVSSRARN_W_D`, `XVSSRLRN_BU_H`, `XVSSRLRN_HU_W`, `XVSSRLRN_WU_D` 等 TableGen 记录。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: 
 722: def XVBITCLR_B : LASX3R_XXX<0x750c0000>;
 723: def XVBITCLR_H : LASX3R_XXX<0x750c8000>;
 724: def XVBITCLR_W : LASX3R_XXX<0x750d0000>;
 725: def XVBITCLR_D : LASX3R_XXX<0x750d8000>;
 726: def XVBITCLRI_B : LASX2RI3_XXI<0x77102000>;
 727: def XVBITCLRI_H : LASX2RI4_XXI<0x77104000>;
 728: def XVBITCLRI_W : LASX2RI5_XXI<0x77108000>;
 729: def XVBITCLRI_D : LASX2RI6_XXI<0x77110000>;
 730: 
 731: def XVBITSET_B : LASX3R_XXX<0x750e0000>;
 732: def XVBITSET_H : LASX3R_XXX<0x750e8000>;
 733: def XVBITSET_W : LASX3R_XXX<0x750f0000>;
 734: def XVBITSET_D : LASX3R_XXX<0x750f8000>;
 735: def XVBITSETI_B : LASX2RI3_XXI<0x77142000>;
 736: def XVBITSETI_H : LASX2RI4_XXI<0x77144000>;
 737: def XVBITSETI_W : LASX2RI5_XXI<0x77148000>;
 738: def XVBITSETI_D : LASX2RI6_XXI<0x77150000>;
 739: 
 740: def XVBITREV_B : LASX3R_XXX<0x75100000>;
 741: def XVBITREV_H : LASX3R_XXX<0x75108000>;
 742: def XVBITREV_W : LASX3R_XXX<0x75110000>;
 743: def XVBITREV_D : LASX3R_XXX<0x75118000>;
 744: def XVBITREVI_B : LASX2RI3_XXI<0x77182000>;
 745: def XVBITREVI_H : LASX2RI4_XXI<0x77184000>;
 746: def XVBITREVI_W : LASX2RI5_XXI<0x77188000>;
 747: def XVBITREVI_D : LASX2RI6_XXI<0x77190000>;
 748: 
 749: def XVFRSTP_B : LASX3R_XXXX<0x752b0000>;
 750: def XVFRSTP_H : LASX3R_XXXX<0x752b8000>;
 751: def XVFRSTPI_B : LASX2RI5_XXXI<0x769a0000>;
 752: def XVFRSTPI_H : LASX2RI5_XXXI<0x769a8000>;
 753: 
 754: def XVFADD_S : LASX3R_XXX<0x75308000>;
 755: def XVFADD_D : LASX3R_XXX<0x75310000>;
 756: def XVFSUB_S : LASX3R_XXX<0x75328000>;
 757: def XVFSUB_D : LASX3R_XXX<0x75330000>;
 758: def XVFMUL_S : LASX3R_XXX<0x75388000>;
 759: def XVFMUL_D : LASX3R_XXX<0x75390000>;
 760: def XVFDIV_S : LASX3R_XXX<0x753a8000>;
```
- **EN**: This block declares or refines TableGen records such as `XVBITCLR_B`, `XVBITCLR_H`, `XVBITCLR_W`, `XVBITCLR_D`, `XVBITCLRI_B`, `XVBITCLRI_H`.
- **CN**: 该代码块声明或细化了 `XVBITCLR_B`, `XVBITCLR_H`, `XVBITCLR_W`, `XVBITCLR_D`, `XVBITCLRI_B`, `XVBITCLRI_H` 等 TableGen 记录。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: def XVFDIV_D : LASX3R_XXX<0x753b0000>;
 762: 
 763: def XVFMADD_S : LASX4R_XXXX<0x0a100000>;
 764: def XVFMADD_D : LASX4R_XXXX<0x0a200000>;
 765: def XVFMSUB_S : LASX4R_XXXX<0x0a500000>;
 766: def XVFMSUB_D : LASX4R_XXXX<0x0a600000>;
 767: def XVFNMADD_S : LASX4R_XXXX<0x0a900000>;
 768: def XVFNMADD_D : LASX4R_XXXX<0x0aa00000>;
 769: def XVFNMSUB_S : LASX4R_XXXX<0x0ad00000>;
 770: def XVFNMSUB_D : LASX4R_XXXX<0x0ae00000>;
 771: 
 772: def XVFMAX_S : LASX3R_XXX<0x753c8000>;
 773: def XVFMAX_D : LASX3R_XXX<0x753d0000>;
 774: def XVFMIN_S : LASX3R_XXX<0x753e8000>;
 775: def XVFMIN_D : LASX3R_XXX<0x753f0000>;
 776: 
 777: def XVFMAXA_S : LASX3R_XXX<0x75408000>;
 778: def XVFMAXA_D : LASX3R_XXX<0x75410000>;
 779: def XVFMINA_S : LASX3R_XXX<0x75428000>;
 780: def XVFMINA_D : LASX3R_XXX<0x75430000>;
 781: 
 782: def XVFLOGB_S : LASX2R_XX<0x769cc400>;
 783: def XVFLOGB_D : LASX2R_XX<0x769cc800>;
 784: 
 785: def XVFCLASS_S : LASX2R_XX<0x769cd400>;
 786: def XVFCLASS_D : LASX2R_XX<0x769cd800>;
 787: 
 788: def XVFSQRT_S : LASX2R_XX<0x769ce400>;
 789: def XVFSQRT_D : LASX2R_XX<0x769ce800>;
 790: def XVFRECIP_S : LASX2R_XX<0x769cf400>;
 791: def XVFRECIP_D : LASX2R_XX<0x769cf800>;
 792: def XVFRSQRT_S : LASX2R_XX<0x769d0400>;
 793: def XVFRSQRT_D : LASX2R_XX<0x769d0800>;
 794: def XVFRECIPE_S : LASX2R_XX<0x769d1400>;
 795: def XVFRECIPE_D : LASX2R_XX<0x769d1800>;
 796: def XVFRSQRTE_S : LASX2R_XX<0x769d2400>;
 797: def XVFRSQRTE_D : LASX2R_XX<0x769d2800>;
 798: 
 799: def XVFCVTL_S_H : LASX2R_XX<0x769de800>;
 800: def XVFCVTH_S_H : LASX2R_XX<0x769dec00>;
```
- **EN**: This block declares or refines TableGen records such as `XVFDIV_D`, `XVFMADD_S`, `XVFMADD_D`, `XVFMSUB_S`, `XVFMSUB_D`, `XVFNMADD_S`.
- **CN**: 该代码块声明或细化了 `XVFDIV_D`, `XVFMADD_S`, `XVFMADD_D`, `XVFMSUB_S`, `XVFMSUB_D`, `XVFNMADD_S` 等 TableGen 记录。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: def XVFCVTL_D_S : LASX2R_XX<0x769df000>;
 802: def XVFCVTH_D_S : LASX2R_XX<0x769df400>;
 803: def XVFCVT_H_S : LASX3R_XXX<0x75460000>;
 804: def XVFCVT_S_D : LASX3R_XXX<0x75468000>;
 805: 
 806: def XVFRINTRNE_S : LASX2R_XX<0x769d7400>;
 807: def XVFRINTRNE_D : LASX2R_XX<0x769d7800>;
 808: def XVFRINTRZ_S : LASX2R_XX<0x769d6400>;
 809: def XVFRINTRZ_D : LASX2R_XX<0x769d6800>;
 810: def XVFRINTRP_S : LASX2R_XX<0x769d5400>;
 811: def XVFRINTRP_D : LASX2R_XX<0x769d5800>;
 812: def XVFRINTRM_S : LASX2R_XX<0x769d4400>;
 813: def XVFRINTRM_D : LASX2R_XX<0x769d4800>;
 814: def XVFRINT_S : LASX2R_XX<0x769d3400>;
 815: def XVFRINT_D : LASX2R_XX<0x769d3800>;
 816: 
 817: def XVFTINTRNE_W_S : LASX2R_XX<0x769e5000>;
 818: def XVFTINTRNE_L_D : LASX2R_XX<0x769e5400>;
 819: def XVFTINTRZ_W_S : LASX2R_XX<0x769e4800>;
 820: def XVFTINTRZ_L_D : LASX2R_XX<0x769e4c00>;
 821: def XVFTINTRP_W_S : LASX2R_XX<0x769e4000>;
 822: def XVFTINTRP_L_D : LASX2R_XX<0x769e4400>;
 823: def XVFTINTRM_W_S : LASX2R_XX<0x769e3800>;
 824: def XVFTINTRM_L_D : LASX2R_XX<0x769e3c00>;
 825: def XVFTINT_W_S : LASX2R_XX<0x769e3000>;
 826: def XVFTINT_L_D : LASX2R_XX<0x769e3400>;
 827: def XVFTINTRZ_WU_S : LASX2R_XX<0x769e7000>;
 828: def XVFTINTRZ_LU_D : LASX2R_XX<0x769e7400>;
 829: def XVFTINT_WU_S : LASX2R_XX<0x769e5800>;
 830: def XVFTINT_LU_D : LASX2R_XX<0x769e5c00>;
 831: 
 832: def XVFTINTRNE_W_D : LASX3R_XXX<0x754b8000>;
 833: def XVFTINTRZ_W_D : LASX3R_XXX<0x754b0000>;
 834: def XVFTINTRP_W_D : LASX3R_XXX<0x754a8000>;
 835: def XVFTINTRM_W_D : LASX3R_XXX<0x754a0000>;
 836: def XVFTINT_W_D : LASX3R_XXX<0x75498000>;
 837: 
 838: def XVFTINTRNEL_L_S : LASX2R_XX<0x769ea000>;
 839: def XVFTINTRNEH_L_S : LASX2R_XX<0x769ea400>;
 840: def XVFTINTRZL_L_S : LASX2R_XX<0x769e9800>;
```
- **EN**: This block declares or refines TableGen records such as `XVFCVTL_D_S`, `XVFCVTH_D_S`, `XVFCVT_H_S`, `XVFCVT_S_D`, `XVFRINTRNE_S`, `XVFRINTRNE_D`.
- **CN**: 该代码块声明或细化了 `XVFCVTL_D_S`, `XVFCVTH_D_S`, `XVFCVT_H_S`, `XVFCVT_S_D`, `XVFRINTRNE_S`, `XVFRINTRNE_D` 等 TableGen 记录。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: def XVFTINTRZH_L_S : LASX2R_XX<0x769e9c00>;
 842: def XVFTINTRPL_L_S : LASX2R_XX<0x769e9000>;
 843: def XVFTINTRPH_L_S : LASX2R_XX<0x769e9400>;
 844: def XVFTINTRML_L_S : LASX2R_XX<0x769e8800>;
 845: def XVFTINTRMH_L_S : LASX2R_XX<0x769e8c00>;
 846: def XVFTINTL_L_S : LASX2R_XX<0x769e8000>;
 847: def XVFTINTH_L_S : LASX2R_XX<0x769e8400>;
 848: 
 849: def XVFFINT_S_W : LASX2R_XX<0x769e0000>;
 850: def XVFFINT_D_L : LASX2R_XX<0x769e0800>;
 851: def XVFFINT_S_WU : LASX2R_XX<0x769e0400>;
 852: def XVFFINT_D_LU : LASX2R_XX<0x769e0c00>;
 853: def XVFFINTL_D_W : LASX2R_XX<0x769e1000>;
 854: def XVFFINTH_D_W : LASX2R_XX<0x769e1400>;
 855: def XVFFINT_S_L : LASX3R_XXX<0x75480000>;
 856: 
 857: def XVSEQ_B : LASX3R_XXX<0x74000000>;
 858: def XVSEQ_H : LASX3R_XXX<0x74008000>;
 859: def XVSEQ_W : LASX3R_XXX<0x74010000>;
 860: def XVSEQ_D : LASX3R_XXX<0x74018000>;
 861: def XVSEQI_B : LASX2RI5_XXI<0x76800000, simm5>;
 862: def XVSEQI_H : LASX2RI5_XXI<0x76808000, simm5>;
 863: def XVSEQI_W : LASX2RI5_XXI<0x76810000, simm5>;
 864: def XVSEQI_D : LASX2RI5_XXI<0x76818000, simm5>;
 865: 
 866: def XVSLE_B : LASX3R_XXX<0x74020000>;
 867: def XVSLE_H : LASX3R_XXX<0x74028000>;
 868: def XVSLE_W : LASX3R_XXX<0x74030000>;
 869: def XVSLE_D : LASX3R_XXX<0x74038000>;
 870: def XVSLEI_B : LASX2RI5_XXI<0x76820000, simm5>;
 871: def XVSLEI_H : LASX2RI5_XXI<0x76828000, simm5>;
 872: def XVSLEI_W : LASX2RI5_XXI<0x76830000, simm5>;
 873: def XVSLEI_D : LASX2RI5_XXI<0x76838000, simm5>;
 874: 
 875: def XVSLE_BU : LASX3R_XXX<0x74040000>;
 876: def XVSLE_HU : LASX3R_XXX<0x74048000>;
 877: def XVSLE_WU : LASX3R_XXX<0x74050000>;
 878: def XVSLE_DU : LASX3R_XXX<0x74058000>;
 879: def XVSLEI_BU : LASX2RI5_XXI<0x76840000>;
 880: def XVSLEI_HU : LASX2RI5_XXI<0x76848000>;
```
- **EN**: This block declares or refines TableGen records such as `XVFTINTRZH_L_S`, `XVFTINTRPL_L_S`, `XVFTINTRPH_L_S`, `XVFTINTRML_L_S`, `XVFTINTRMH_L_S`, `XVFTINTL_L_S`.
- **CN**: 该代码块声明或细化了 `XVFTINTRZH_L_S`, `XVFTINTRPL_L_S`, `XVFTINTRPH_L_S`, `XVFTINTRML_L_S`, `XVFTINTRMH_L_S`, `XVFTINTL_L_S` 等 TableGen 记录。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: def XVSLEI_WU : LASX2RI5_XXI<0x76850000>;
 882: def XVSLEI_DU : LASX2RI5_XXI<0x76858000>;
 883: 
 884: def XVSLT_B : LASX3R_XXX<0x74060000>;
 885: def XVSLT_H : LASX3R_XXX<0x74068000>;
 886: def XVSLT_W : LASX3R_XXX<0x74070000>;
 887: def XVSLT_D : LASX3R_XXX<0x74078000>;
 888: def XVSLTI_B : LASX2RI5_XXI<0x76860000, simm5>;
 889: def XVSLTI_H : LASX2RI5_XXI<0x76868000, simm5>;
 890: def XVSLTI_W : LASX2RI5_XXI<0x76870000, simm5>;
 891: def XVSLTI_D : LASX2RI5_XXI<0x76878000, simm5>;
 892: 
 893: def XVSLT_BU : LASX3R_XXX<0x74080000>;
 894: def XVSLT_HU : LASX3R_XXX<0x74088000>;
 895: def XVSLT_WU : LASX3R_XXX<0x74090000>;
 896: def XVSLT_DU : LASX3R_XXX<0x74098000>;
 897: def XVSLTI_BU : LASX2RI5_XXI<0x76880000>;
 898: def XVSLTI_HU : LASX2RI5_XXI<0x76888000>;
 899: def XVSLTI_WU : LASX2RI5_XXI<0x76890000>;
 900: def XVSLTI_DU : LASX2RI5_XXI<0x76898000>;
 901: 
 902: def XVFCMP_CAF_S : LASX3R_XXX<0x0c900000>;
 903: def XVFCMP_SAF_S : LASX3R_XXX<0x0c908000>;
 904: def XVFCMP_CLT_S : LASX3R_XXX<0x0c910000>;
 905: def XVFCMP_SLT_S : LASX3R_XXX<0x0c918000>;
 906: def XVFCMP_CEQ_S : LASX3R_XXX<0x0c920000>;
 907: def XVFCMP_SEQ_S : LASX3R_XXX<0x0c928000>;
 908: def XVFCMP_CLE_S : LASX3R_XXX<0x0c930000>;
 909: def XVFCMP_SLE_S : LASX3R_XXX<0x0c938000>;
 910: def XVFCMP_CUN_S : LASX3R_XXX<0x0c940000>;
 911: def XVFCMP_SUN_S : LASX3R_XXX<0x0c948000>;
 912: def XVFCMP_CULT_S : LASX3R_XXX<0x0c950000>;
 913: def XVFCMP_SULT_S : LASX3R_XXX<0x0c958000>;
 914: def XVFCMP_CUEQ_S : LASX3R_XXX<0x0c960000>;
 915: def XVFCMP_SUEQ_S : LASX3R_XXX<0x0c968000>;
 916: def XVFCMP_CULE_S : LASX3R_XXX<0x0c970000>;
 917: def XVFCMP_SULE_S : LASX3R_XXX<0x0c978000>;
 918: def XVFCMP_CNE_S : LASX3R_XXX<0x0c980000>;
 919: def XVFCMP_SNE_S : LASX3R_XXX<0x0c988000>;
 920: def XVFCMP_COR_S : LASX3R_XXX<0x0c9a0000>;
```
- **EN**: This block declares or refines TableGen records such as `XVSLEI_WU`, `XVSLEI_DU`, `XVSLT_B`, `XVSLT_H`, `XVSLT_W`, `XVSLT_D`.
- **CN**: 该代码块声明或细化了 `XVSLEI_WU`, `XVSLEI_DU`, `XVSLT_B`, `XVSLT_H`, `XVSLT_W`, `XVSLT_D` 等 TableGen 记录。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: def XVFCMP_SOR_S : LASX3R_XXX<0x0c9a8000>;
 922: def XVFCMP_CUNE_S : LASX3R_XXX<0x0c9c0000>;
 923: def XVFCMP_SUNE_S : LASX3R_XXX<0x0c9c8000>;
 924: 
 925: def XVFCMP_CAF_D : LASX3R_XXX<0x0ca00000>;
 926: def XVFCMP_SAF_D : LASX3R_XXX<0x0ca08000>;
 927: def XVFCMP_CLT_D : LASX3R_XXX<0x0ca10000>;
 928: def XVFCMP_SLT_D : LASX3R_XXX<0x0ca18000>;
 929: def XVFCMP_CEQ_D : LASX3R_XXX<0x0ca20000>;
 930: def XVFCMP_SEQ_D : LASX3R_XXX<0x0ca28000>;
 931: def XVFCMP_CLE_D : LASX3R_XXX<0x0ca30000>;
 932: def XVFCMP_SLE_D : LASX3R_XXX<0x0ca38000>;
 933: def XVFCMP_CUN_D : LASX3R_XXX<0x0ca40000>;
 934: def XVFCMP_SUN_D : LASX3R_XXX<0x0ca48000>;
 935: def XVFCMP_CULT_D : LASX3R_XXX<0x0ca50000>;
 936: def XVFCMP_SULT_D : LASX3R_XXX<0x0ca58000>;
 937: def XVFCMP_CUEQ_D : LASX3R_XXX<0x0ca60000>;
 938: def XVFCMP_SUEQ_D : LASX3R_XXX<0x0ca68000>;
 939: def XVFCMP_CULE_D : LASX3R_XXX<0x0ca70000>;
 940: def XVFCMP_SULE_D : LASX3R_XXX<0x0ca78000>;
 941: def XVFCMP_CNE_D : LASX3R_XXX<0x0ca80000>;
 942: def XVFCMP_SNE_D : LASX3R_XXX<0x0ca88000>;
 943: def XVFCMP_COR_D : LASX3R_XXX<0x0caa0000>;
 944: def XVFCMP_SOR_D : LASX3R_XXX<0x0caa8000>;
 945: def XVFCMP_CUNE_D : LASX3R_XXX<0x0cac0000>;
 946: def XVFCMP_SUNE_D : LASX3R_XXX<0x0cac8000>;
 947: 
 948: def XVBITSEL_V : LASX4R_XXXX<0x0d200000>;
 949: 
 950: def XVBITSELI_B : LASX2RI8_XXXI<0x77c40000>;
 951: 
 952: def XVSETEQZ_V : LASX2R_CX<0x769c9800>;
 953: def XVSETNEZ_V : LASX2R_CX<0x769c9c00>;
 954: def XVSETANYEQZ_B : LASX2R_CX<0x769ca000>;
 955: def XVSETANYEQZ_H : LASX2R_CX<0x769ca400>;
 956: def XVSETANYEQZ_W : LASX2R_CX<0x769ca800>;
 957: def XVSETANYEQZ_D : LASX2R_CX<0x769cac00>;
 958: def XVSETALLNEZ_B : LASX2R_CX<0x769cb000>;
 959: def XVSETALLNEZ_H : LASX2R_CX<0x769cb400>;
 960: def XVSETALLNEZ_W : LASX2R_CX<0x769cb800>;
```
- **EN**: This block declares or refines TableGen records such as `XVFCMP_SOR_S`, `XVFCMP_CUNE_S`, `XVFCMP_SUNE_S`, `XVFCMP_CAF_D`, `XVFCMP_SAF_D`, `XVFCMP_CLT_D`.
- **CN**: 该代码块声明或细化了 `XVFCMP_SOR_S`, `XVFCMP_CUNE_S`, `XVFCMP_SUNE_S`, `XVFCMP_CAF_D`, `XVFCMP_SAF_D`, `XVFCMP_CLT_D` 等 TableGen 记录。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: def XVSETALLNEZ_D : LASX2R_CX<0x769cbc00>;
 962: 
 963: def XVINSGR2VR_W : LASX2RI3_XXRI<0x76ebc000>;
 964: def XVINSGR2VR_D : LASX2RI2_XXRI<0x76ebe000>;
 965: def XVPICKVE2GR_W : LASX2RI3_RXI<0x76efc000>;
 966: def XVPICKVE2GR_D : LASX2RI2_RXI<0x76efe000>;
 967: def XVPICKVE2GR_WU : LASX2RI3_RXI<0x76f3c000>;
 968: def XVPICKVE2GR_DU : LASX2RI2_RXI<0x76f3e000>;
 969: 
 970: let isReMaterializable = 1 in {
 971: def XVREPLGR2VR_B : LASX2R_XR<0x769f0000>;
 972: def XVREPLGR2VR_H : LASX2R_XR<0x769f0400>;
 973: def XVREPLGR2VR_W : LASX2R_XR<0x769f0800>;
 974: def XVREPLGR2VR_D : LASX2R_XR<0x769f0c00>;
 975: }
 976: 
 977: def XVREPLVE_B : LASX3R_XXR<0x75220000>;
 978: def XVREPLVE_H : LASX3R_XXR<0x75228000>;
 979: def XVREPLVE_W : LASX3R_XXR<0x75230000>;
 980: def XVREPLVE_D : LASX3R_XXR<0x75238000>;
 981: def XVREPL128VEI_B : LASX2RI4_XXI<0x76f78000>;
 982: def XVREPL128VEI_H : LASX2RI3_XXI<0x76f7c000>;
 983: def XVREPL128VEI_W : LASX2RI2_XXI<0x76f7e000>;
 984: def XVREPL128VEI_D : LASX2RI1_XXI<0x76f7f000>;
 985: 
 986: def XVREPLVE0_B : LASX2R_XX<0x77070000>;
 987: def XVREPLVE0_H : LASX2R_XX<0x77078000>;
 988: def XVREPLVE0_W : LASX2R_XX<0x7707c000>;
 989: def XVREPLVE0_D : LASX2R_XX<0x7707e000>;
 990: def XVREPLVE0_Q : LASX2R_XX<0x7707f000>;
 991: 
 992: def XVINSVE0_W : LASX2RI3_XXXI<0x76ffc000>;
 993: def XVINSVE0_D : LASX2RI2_XXXI<0x76ffe000>;
 994: 
 995: def XVPICKVE_W : LASX2RI3_XXI<0x7703c000>;
 996: def XVPICKVE_D : LASX2RI2_XXI<0x7703e000>;
 997: 
 998: def XVBSLL_V : LASX2RI5_XXI<0x768e0000>;
 999: def XVBSRL_V : LASX2RI5_XXI<0x768e8000>;
1000: 
```
- **EN**: This block declares or refines TableGen records such as `XVSETALLNEZ_D`, `XVINSGR2VR_W`, `XVINSGR2VR_D`, `XVPICKVE2GR_W`, `XVPICKVE2GR_D`, `XVPICKVE2GR_WU`.
- **CN**: 该代码块声明或细化了 `XVSETALLNEZ_D`, `XVINSGR2VR_W`, `XVINSGR2VR_D`, `XVPICKVE2GR_W`, `XVPICKVE2GR_D`, `XVPICKVE2GR_WU` 等 TableGen 记录。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: def XVPACKEV_B : LASX3R_XXX<0x75160000>;
1002: def XVPACKEV_H : LASX3R_XXX<0x75168000>;
1003: def XVPACKEV_W : LASX3R_XXX<0x75170000>;
1004: def XVPACKEV_D : LASX3R_XXX<0x75178000>;
1005: def XVPACKOD_B : LASX3R_XXX<0x75180000>;
1006: def XVPACKOD_H : LASX3R_XXX<0x75188000>;
1007: def XVPACKOD_W : LASX3R_XXX<0x75190000>;
1008: def XVPACKOD_D : LASX3R_XXX<0x75198000>;
1009: 
1010: def XVPICKEV_B : LASX3R_XXX<0x751e0000>;
1011: def XVPICKEV_H : LASX3R_XXX<0x751e8000>;
1012: def XVPICKEV_W : LASX3R_XXX<0x751f0000>;
1013: def XVPICKEV_D : LASX3R_XXX<0x751f8000>;
1014: def XVPICKOD_B : LASX3R_XXX<0x75200000>;
1015: def XVPICKOD_H : LASX3R_XXX<0x75208000>;
1016: def XVPICKOD_W : LASX3R_XXX<0x75210000>;
1017: def XVPICKOD_D : LASX3R_XXX<0x75218000>;
1018: 
1019: def XVILVL_B : LASX3R_XXX<0x751a0000>;
1020: def XVILVL_H : LASX3R_XXX<0x751a8000>;
1021: def XVILVL_W : LASX3R_XXX<0x751b0000>;
1022: def XVILVL_D : LASX3R_XXX<0x751b8000>;
1023: def XVILVH_B : LASX3R_XXX<0x751c0000>;
1024: def XVILVH_H : LASX3R_XXX<0x751c8000>;
1025: def XVILVH_W : LASX3R_XXX<0x751d0000>;
1026: def XVILVH_D : LASX3R_XXX<0x751d8000>;
1027: 
1028: def XVSHUF_B : LASX4R_XXXX<0x0d600000>;
1029: 
1030: def XVSHUF_H : LASX3R_XXXX<0x757a8000>;
1031: def XVSHUF_W : LASX3R_XXXX<0x757b0000>;
1032: def XVSHUF_D : LASX3R_XXXX<0x757b8000>;
1033: 
1034: def XVPERM_W : LASX3R_XXX<0x757d0000>;
1035: 
1036: def XVSHUF4I_B : LASX2RI8_XXI<0x77900000>;
1037: def XVSHUF4I_H : LASX2RI8_XXI<0x77940000>;
1038: def XVSHUF4I_W : LASX2RI8_XXI<0x77980000>;
1039: def XVSHUF4I_D : LASX2RI8_XXXI<0x779c0000>;
1040: 
```
- **EN**: This block declares or refines TableGen records such as `XVPACKEV_B`, `XVPACKEV_H`, `XVPACKEV_W`, `XVPACKEV_D`, `XVPACKOD_B`, `XVPACKOD_H`.
- **CN**: 该代码块声明或细化了 `XVPACKEV_B`, `XVPACKEV_H`, `XVPACKEV_W`, `XVPACKEV_D`, `XVPACKOD_B`, `XVPACKOD_H` 等 TableGen 记录。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: def XVPERMI_W : LASX2RI8_XXXI<0x77e40000>;
1042: def XVPERMI_D : LASX2RI8_XXI<0x77e80000>;
1043: def XVPERMI_Q : LASX2RI8_XXXI<0x77ec0000>;
1044: 
1045: def XVEXTRINS_D : LASX2RI8_XXXI<0x77800000>;
1046: def XVEXTRINS_W : LASX2RI8_XXXI<0x77840000>;
1047: def XVEXTRINS_H : LASX2RI8_XXXI<0x77880000>;
1048: def XVEXTRINS_B : LASX2RI8_XXXI<0x778c0000>;
1049: } // mayLoad = 0, mayStore = 0
1050: 
1051: let mayLoad = 1, mayStore = 0 in {
1052: def XVLD : LASX2RI12_Load<0x2c800000>;
1053: def XVLDX : LASX3R_Load<0x38480000>;
1054: 
1055: def XVLDREPL_B : LASX2RI12_Load<0x32800000>;
1056: def XVLDREPL_H : LASX2RI11_Load<0x32400000>;
1057: def XVLDREPL_W : LASX2RI10_Load<0x32200000>;
1058: def XVLDREPL_D : LASX2RI9_Load<0x32100000>;
1059: } // mayLoad = 1, mayStore = 0
1060: 
1061: let mayLoad = 0, mayStore = 1 in {
1062: def XVST : LASX2RI12_Store<0x2cc00000>;
1063: def XVSTX : LASX3R_Store<0x384c0000>;
1064: 
1065: def XVSTELM_B : LASX2RI8I5_XRII<0x33800000>;
1066: def XVSTELM_H : LASX2RI8I4_XRII<0x33400000, simm8_lsl1>;
1067: def XVSTELM_W : LASX2RI8I3_XRII<0x33200000, simm8_lsl2>;
1068: def XVSTELM_D : LASX2RI8I2_XRII<0x33100000, simm8_lsl3>;
1069: } // mayLoad = 0, mayStore = 1
1070: 
1071: } // hasSideEffects = 0, Predicates = [HasExtLASX]
1072: 
1073: /// Pseudo-instructions
1074: 
1075: let Predicates = [HasExtLASX] in {
1076: 
1077: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCodeGenOnly = 0,
1078:     isAsmParserOnly = 1 in {
1079: def PseudoXVREPLI_B : Pseudo<(outs LASX256:$xd), (ins simm10:$imm), [],
1080:                              "xvrepli.b", "$xd, $imm">;
```
- **EN**: This block declares or refines TableGen records such as `XVPERMI_W`, `XVPERMI_D`, `XVPERMI_Q`, `XVEXTRINS_D`, `XVEXTRINS_W`, `XVEXTRINS_H`.
- **CN**: 该代码块声明或细化了 `XVPERMI_W`, `XVPERMI_D`, `XVPERMI_Q`, `XVEXTRINS_D`, `XVEXTRINS_W`, `XVEXTRINS_H` 等 TableGen 记录。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: def PseudoXVREPLI_H : Pseudo<(outs LASX256:$xd), (ins simm10:$imm), [],
1082:                              "xvrepli.h", "$xd, $imm">;
1083: def PseudoXVREPLI_W : Pseudo<(outs LASX256:$xd), (ins simm10:$imm), [],
1084:                              "xvrepli.w", "$xd, $imm">;
1085: def PseudoXVREPLI_D : Pseudo<(outs LASX256:$xd), (ins simm10:$imm), [],
1086:                              "xvrepli.d", "$xd, $imm">;
1087: }
1088: 
1089: def PseudoXVBNZ_B : VecCond<loongarch_vall_nonzero, v32i8, LASX256>;
1090: def PseudoXVBNZ_H : VecCond<loongarch_vall_nonzero, v16i16, LASX256>;
1091: def PseudoXVBNZ_W : VecCond<loongarch_vall_nonzero, v8i32, LASX256>;
1092: def PseudoXVBNZ_D : VecCond<loongarch_vall_nonzero, v4i64, LASX256>;
1093: def PseudoXVBNZ : VecCond<loongarch_vany_nonzero, v32i8, LASX256>;
1094: 
1095: def PseudoXVBZ_B : VecCond<loongarch_vall_zero, v32i8, LASX256>;
1096: def PseudoXVBZ_H : VecCond<loongarch_vall_zero, v16i16, LASX256>;
1097: def PseudoXVBZ_W : VecCond<loongarch_vall_zero, v8i32, LASX256>;
1098: def PseudoXVBZ_D : VecCond<loongarch_vall_zero, v4i64, LASX256>;
1099: def PseudoXVBZ : VecCond<loongarch_vany_zero, v32i8, LASX256>;
1100: 
1101: let usesCustomInserter = 1, Constraints = "$xd = $dst" in {
1102: def PseudoXVINSGR2VR_B
1103:   : Pseudo<(outs LASX256:$dst), (ins LASX256:$xd, GPR:$rj, uimm5:$imm)>;
1104: def PseudoXVINSGR2VR_H
1105:   : Pseudo<(outs LASX256:$dst), (ins LASX256:$xd, GPR:$rj, uimm4:$imm)>;
1106: } //  usesCustomInserter = 1, Constraints = "$xd = $dst"
1107: 
1108: let usesCustomInserter = 1, hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
1109: def PseudoXVMSKLTZ_B : Pseudo<(outs GPR:$rd), (ins LASX256:$vj)>;
1110: def PseudoXVMSKLTZ_H : Pseudo<(outs GPR:$rd), (ins LASX256:$vj)>;
1111: def PseudoXVMSKLTZ_W : Pseudo<(outs GPR:$rd), (ins LASX256:$vj)>;
1112: def PseudoXVMSKLTZ_D : Pseudo<(outs GPR:$rd), (ins LASX256:$vj)>;
1113: def PseudoXVMSKGEZ_B : Pseudo<(outs GPR:$rd), (ins LASX256:$vj)>;
1114: def PseudoXVMSKEQZ_B : Pseudo<(outs GPR:$rd), (ins LASX256:$vj)>;
1115: def PseudoXVMSKNEZ_B : Pseudo<(outs GPR:$rd), (ins LASX256:$vj)>;
1116: } // usesCustomInserter = 1, hasSideEffects = 0, mayLoad = 0, mayStore = 0
1117: 
1118: } // Predicates = [HasExtLASX]
1119: 
1120: multiclass PatXr<SDPatternOperator OpNode, string Inst> {
```
- **EN**: This block declares or refines TableGen records such as `PseudoXVREPLI_H`, `PseudoXVREPLI_W`, `PseudoXVREPLI_D`, `PseudoXVBNZ_B`, `PseudoXVBNZ_H`, `PseudoXVBNZ_W`.
- **CN**: 该代码块声明或细化了 `PseudoXVREPLI_H`, `PseudoXVREPLI_W`, `PseudoXVREPLI_D`, `PseudoXVBNZ_B`, `PseudoXVBNZ_H`, `PseudoXVBNZ_W` 等 TableGen 记录。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121:   def : Pat<(v32i8 (OpNode (v32i8 LASX256:$xj))),
1122:             (!cast<LAInst>(Inst#"_B") LASX256:$xj)>;
1123:   def : Pat<(v16i16 (OpNode (v16i16 LASX256:$xj))),
1124:             (!cast<LAInst>(Inst#"_H") LASX256:$xj)>;
1125:   def : Pat<(v8i32 (OpNode (v8i32 LASX256:$xj))),
1126:             (!cast<LAInst>(Inst#"_W") LASX256:$xj)>;
1127:   def : Pat<(v4i64 (OpNode (v4i64 LASX256:$xj))),
1128:             (!cast<LAInst>(Inst#"_D") LASX256:$xj)>;
1129: }
1130: 
1131: multiclass PatXrF<SDPatternOperator OpNode, string Inst> {
1132:   def : Pat<(v8f32 (OpNode (v8f32 LASX256:$xj))),
1133:             (!cast<LAInst>(Inst#"_S") LASX256:$xj)>;
1134:   def : Pat<(v4f64 (OpNode (v4f64 LASX256:$xj))),
1135:             (!cast<LAInst>(Inst#"_D") LASX256:$xj)>;
1136: }
1137: 
1138: multiclass PatXrXr<SDPatternOperator OpNode, string Inst> {
1139:   def : Pat<(OpNode (v32i8 LASX256:$xj), (v32i8 LASX256:$xk)),
1140:             (!cast<LAInst>(Inst#"_B") LASX256:$xj, LASX256:$xk)>;
1141:   def : Pat<(OpNode (v16i16 LASX256:$xj), (v16i16 LASX256:$xk)),
1142:             (!cast<LAInst>(Inst#"_H") LASX256:$xj, LASX256:$xk)>;
1143:   def : Pat<(OpNode (v8i32 LASX256:$xj), (v8i32 LASX256:$xk)),
1144:             (!cast<LAInst>(Inst#"_W") LASX256:$xj, LASX256:$xk)>;
1145:   def : Pat<(OpNode (v4i64 LASX256:$xj), (v4i64 LASX256:$xk)),
1146:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, LASX256:$xk)>;
1147: }
1148: 
1149: multiclass PatXrXrF<SDPatternOperator OpNode, string Inst> {
1150:   def : Pat<(OpNode (v8f32 LASX256:$xj), (v8f32 LASX256:$xk)),
1151:             (!cast<LAInst>(Inst#"_S") LASX256:$xj, LASX256:$xk)>;
1152:   def : Pat<(OpNode (v4f64 LASX256:$xj), (v4f64 LASX256:$xk)),
1153:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, LASX256:$xk)>;
1154: }
1155: 
1156: multiclass PatXrXrU<SDPatternOperator OpNode, string Inst> {
1157:   def : Pat<(OpNode (v32i8 LASX256:$xj), (v32i8 LASX256:$xk)),
1158:             (!cast<LAInst>(Inst#"_BU") LASX256:$xj, LASX256:$xk)>;
1159:   def : Pat<(OpNode (v16i16 LASX256:$xj), (v16i16 LASX256:$xk)),
1160:             (!cast<LAInst>(Inst#"_HU") LASX256:$xj, LASX256:$xk)>;
```
- **EN**: This block declares or refines TableGen records such as `PatXrF`, `PatXrXr`, `PatXrXrF`, `PatXrXrU`.
- **CN**: 该代码块声明或细化了 `PatXrF`, `PatXrXr`, `PatXrXrF`, `PatXrXrU` 等 TableGen 记录。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161:   def : Pat<(OpNode (v8i32 LASX256:$xj), (v8i32 LASX256:$xk)),
1162:             (!cast<LAInst>(Inst#"_WU") LASX256:$xj, LASX256:$xk)>;
1163:   def : Pat<(OpNode (v4i64 LASX256:$xj), (v4i64 LASX256:$xk)),
1164:             (!cast<LAInst>(Inst#"_DU") LASX256:$xj, LASX256:$xk)>;
1165: }
1166: 
1167: multiclass PatXrSimm5<SDPatternOperator OpNode, string Inst> {
1168:   def : Pat<(OpNode (v32i8 LASX256:$xj), (v32i8 (SplatPat_simm5 simm5:$imm))),
1169:             (!cast<LAInst>(Inst#"_B") LASX256:$xj, simm5:$imm)>;
1170:   def : Pat<(OpNode (v16i16 LASX256:$xj), (v16i16 (SplatPat_simm5 simm5:$imm))),
1171:             (!cast<LAInst>(Inst#"_H") LASX256:$xj, simm5:$imm)>;
1172:   def : Pat<(OpNode (v8i32 LASX256:$xj), (v8i32 (SplatPat_simm5 simm5:$imm))),
1173:             (!cast<LAInst>(Inst#"_W") LASX256:$xj, simm5:$imm)>;
1174:   def : Pat<(OpNode (v4i64 LASX256:$xj), (v4i64 (SplatPat_simm5 simm5:$imm))),
1175:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, simm5:$imm)>;
1176: }
1177: 
1178: multiclass PatXrUimm5<SDPatternOperator OpNode, string Inst> {
1179:   def : Pat<(OpNode (v32i8 LASX256:$xj), (v32i8 (SplatPat_uimm5 uimm5:$imm))),
1180:             (!cast<LAInst>(Inst#"_BU") LASX256:$xj, uimm5:$imm)>;
1181:   def : Pat<(OpNode (v16i16 LASX256:$xj), (v16i16 (SplatPat_uimm5 uimm5:$imm))),
1182:             (!cast<LAInst>(Inst#"_HU") LASX256:$xj, uimm5:$imm)>;
1183:   def : Pat<(OpNode (v8i32 LASX256:$xj), (v8i32 (SplatPat_uimm5 uimm5:$imm))),
1184:             (!cast<LAInst>(Inst#"_WU") LASX256:$xj, uimm5:$imm)>;
1185:   def : Pat<(OpNode (v4i64 LASX256:$xj), (v4i64 (SplatPat_uimm5 uimm5:$imm))),
1186:             (!cast<LAInst>(Inst#"_DU") LASX256:$xj, uimm5:$imm)>;
1187: }
1188: 
1189: multiclass PatXrNimm5<SDPatternOperator OpNode, string Inst> {
1190:   def : Pat<(OpNode (v32i8 LASX256:$xj), (v32i8 (SplatPat_nimm5 uimm5:$imm))),
1191:             (!cast<LAInst>(Inst#"_BU") LASX256:$xj, uimm5:$imm)>;
1192:   def : Pat<(OpNode (v16i16 LASX256:$xj), (v16i16 (SplatPat_nimm5 uimm5:$imm))),
1193:             (!cast<LAInst>(Inst#"_HU") LASX256:$xj, uimm5:$imm)>;
1194:   def : Pat<(OpNode (v8i32 LASX256:$xj), (v8i32 (SplatPat_nimm5 uimm5:$imm))),
1195:             (!cast<LAInst>(Inst#"_WU") LASX256:$xj, uimm5:$imm)>;
1196:   def : Pat<(OpNode (v4i64 LASX256:$xj), (v4i64 (SplatPat_nimm5 uimm5:$imm))),
1197:             (!cast<LAInst>(Inst#"_DU") LASX256:$xj, uimm5:$imm)>;
1198: }
1199: 
1200: multiclass PatXrXrXr<SDPatternOperator OpNode, string Inst> {
```
- **EN**: This block declares or refines TableGen records such as `PatXrSimm5`, `PatXrUimm5`, `PatXrNimm5`, `PatXrXrXr`.
- **CN**: 该代码块声明或细化了 `PatXrSimm5`, `PatXrUimm5`, `PatXrNimm5`, `PatXrXrXr` 等 TableGen 记录。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201:   def : Pat<(OpNode (v32i8 LASX256:$xd), (v32i8 LASX256:$xj),
1202:                     (v32i8 LASX256:$xk)),
1203:             (!cast<LAInst>(Inst#"_B") LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
1204:   def : Pat<(OpNode (v16i16 LASX256:$xd), (v16i16 LASX256:$xj),
1205:                     (v16i16 LASX256:$xk)),
1206:             (!cast<LAInst>(Inst#"_H") LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
1207:   def : Pat<(OpNode (v8i32 LASX256:$xd), (v8i32 LASX256:$xj),
1208:                     (v8i32 LASX256:$xk)),
1209:             (!cast<LAInst>(Inst#"_W") LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
1210:   def : Pat<(OpNode (v4i64 LASX256:$xd), (v4i64 LASX256:$xj),
1211:                     (v4i64 LASX256:$xk)),
1212:             (!cast<LAInst>(Inst#"_D") LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
1213: }
1214: 
1215: multiclass PatXrXrW<SDPatternOperator OpNode, string Inst> {
1216:   def : Pat<(OpNode(v32i8 LASX256:$vj), (v32i8 LASX256:$vk)),
1217:             (!cast<LAInst>(Inst#"_H_B") LASX256:$vj, LASX256:$vk)>;
1218:   def : Pat<(OpNode(v16i16 LASX256:$vj), (v16i16 LASX256:$vk)),
1219:             (!cast<LAInst>(Inst#"_W_H") LASX256:$vj, LASX256:$vk)>;
1220:   def : Pat<(OpNode(v8i32 LASX256:$vj), (v8i32 LASX256:$vk)),
1221:             (!cast<LAInst>(Inst#"_D_W") LASX256:$vj, LASX256:$vk)>;
1222:   def : Pat<(OpNode(v4i64 LASX256:$vj), (v4i64 LASX256:$vk)),
1223:             (!cast<LAInst>(Inst#"_Q_D") LASX256:$vj, LASX256:$vk)>;
1224: }
1225: 
1226: multiclass PatShiftXrXr<SDPatternOperator OpNode, string Inst> {
1227:   def : Pat<(OpNode (v32i8 LASX256:$xj), (and vsplati8_imm_eq_7,
1228:                                               (v32i8 LASX256:$xk))),
1229:             (!cast<LAInst>(Inst#"_B") LASX256:$xj, LASX256:$xk)>;
1230:   def : Pat<(OpNode (v16i16 LASX256:$xj), (and vsplati16_imm_eq_15,
1231:                                                (v16i16 LASX256:$xk))),
1232:             (!cast<LAInst>(Inst#"_H") LASX256:$xj, LASX256:$xk)>;
1233:   def : Pat<(OpNode (v8i32 LASX256:$xj), (and vsplati32_imm_eq_31,
1234:                                               (v8i32 LASX256:$xk))),
1235:             (!cast<LAInst>(Inst#"_W") LASX256:$xj, LASX256:$xk)>;
1236:   def : Pat<(OpNode (v4i64 LASX256:$xj), (and vsplati64_imm_eq_63,
1237:                                               (v4i64 LASX256:$xk))),
1238:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, LASX256:$xk)>;
1239: }
1240: 
```
- **EN**: This block declares or refines TableGen records such as `PatXrXrW`, `PatShiftXrXr`.
- **CN**: 该代码块声明或细化了 `PatXrXrW`, `PatShiftXrXr` 等 TableGen 记录。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241: multiclass PatShiftXrSplatUimm<SDPatternOperator OpNode, string Inst> {
1242:   def : Pat<(OpNode (v32i8 LASX256:$xj), (v32i8 (SplatPat_uimm3 uimm3:$imm))),
1243:             (!cast<LAInst>(Inst#"_B") LASX256:$xj, uimm3:$imm)>;
1244:   def : Pat<(OpNode (v16i16 LASX256:$xj), (v16i16 (SplatPat_uimm4 uimm4:$imm))),
1245:             (!cast<LAInst>(Inst#"_H") LASX256:$xj, uimm4:$imm)>;
1246:   def : Pat<(OpNode (v8i32 LASX256:$xj), (v8i32 (SplatPat_uimm5 uimm5:$imm))),
1247:             (!cast<LAInst>(Inst#"_W") LASX256:$xj, uimm5:$imm)>;
1248:   def : Pat<(OpNode (v4i64 LASX256:$xj), (v4i64 (SplatPat_uimm6 uimm6:$imm))),
1249:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, uimm6:$imm)>;
1250: }
1251: 
1252: multiclass PatShiftXrUimm<SDPatternOperator OpNode, string Inst> {
1253:   def : Pat<(OpNode(v32i8 LASX256:$vj), uimm3:$imm),
1254:             (!cast<LAInst>(Inst#"_B") LASX256:$vj, uimm3:$imm)>;
1255:   def : Pat<(OpNode(v16i16 LASX256:$vj), uimm4:$imm),
1256:             (!cast<LAInst>(Inst#"_H") LASX256:$vj, uimm4:$imm)>;
1257:   def : Pat<(OpNode(v8i32 LASX256:$vj), uimm5:$imm),
1258:             (!cast<LAInst>(Inst#"_W") LASX256:$vj, uimm5:$imm)>;
1259:   def : Pat<(OpNode(v4i64 LASX256:$vj), uimm6:$imm),
1260:             (!cast<LAInst>(Inst#"_D") LASX256:$vj, uimm6:$imm)>;
1261: }
1262: 
1263: multiclass PatCCXrSimm5<CondCode CC, string Inst> {
1264:   def : Pat<(v32i8 (setcc (v32i8 LASX256:$xj),
1265:                           (v32i8 (SplatPat_simm5 simm5:$imm)), CC)),
1266:             (!cast<LAInst>(Inst#"_B") LASX256:$xj, simm5:$imm)>;
1267:   def : Pat<(v16i16 (setcc (v16i16 LASX256:$xj),
1268:                            (v16i16 (SplatPat_simm5 simm5:$imm)), CC)),
1269:             (!cast<LAInst>(Inst#"_H") LASX256:$xj, simm5:$imm)>;
1270:   def : Pat<(v8i32 (setcc (v8i32 LASX256:$xj),
1271:                           (v8i32 (SplatPat_simm5 simm5:$imm)), CC)),
1272:             (!cast<LAInst>(Inst#"_W") LASX256:$xj, simm5:$imm)>;
1273:   def : Pat<(v4i64 (setcc (v4i64 LASX256:$xj),
1274:                           (v4i64 (SplatPat_simm5 simm5:$imm)), CC)),
1275:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, simm5:$imm)>;
1276: }
1277: 
1278: multiclass PatCCXrUimm5<CondCode CC, string Inst> {
1279:   def : Pat<(v32i8 (setcc (v32i8 LASX256:$xj),
1280:                           (v32i8 (SplatPat_uimm5 uimm5:$imm)), CC)),
```
- **EN**: This block declares or refines TableGen records such as `PatShiftXrSplatUimm`, `PatShiftXrUimm`, `PatCCXrSimm5`, `PatCCXrUimm5`.
- **CN**: 该代码块声明或细化了 `PatShiftXrSplatUimm`, `PatShiftXrUimm`, `PatCCXrSimm5`, `PatCCXrUimm5` 等 TableGen 记录。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281:             (!cast<LAInst>(Inst#"_BU") LASX256:$xj, uimm5:$imm)>;
1282:   def : Pat<(v16i16 (setcc (v16i16 LASX256:$xj),
1283:                            (v16i16 (SplatPat_uimm5 uimm5:$imm)), CC)),
1284:             (!cast<LAInst>(Inst#"_HU") LASX256:$xj, uimm5:$imm)>;
1285:   def : Pat<(v8i32 (setcc (v8i32 LASX256:$xj),
1286:                           (v8i32 (SplatPat_uimm5 uimm5:$imm)), CC)),
1287:             (!cast<LAInst>(Inst#"_WU") LASX256:$xj, uimm5:$imm)>;
1288:   def : Pat<(v4i64 (setcc (v4i64 LASX256:$xj),
1289:                           (v4i64 (SplatPat_uimm5 uimm5:$imm)), CC)),
1290:             (!cast<LAInst>(Inst#"_DU") LASX256:$xj, uimm5:$imm)>;
1291: }
1292: 
1293: multiclass PatCCXrXr<CondCode CC, string Inst> {
1294:   def : Pat<(v32i8 (setcc (v32i8 LASX256:$xj), (v32i8 LASX256:$xk), CC)),
1295:             (!cast<LAInst>(Inst#"_B") LASX256:$xj, LASX256:$xk)>;
1296:   def : Pat<(v16i16 (setcc (v16i16 LASX256:$xj), (v16i16 LASX256:$xk), CC)),
1297:             (!cast<LAInst>(Inst#"_H") LASX256:$xj, LASX256:$xk)>;
1298:   def : Pat<(v8i32 (setcc (v8i32 LASX256:$xj), (v8i32 LASX256:$xk), CC)),
1299:             (!cast<LAInst>(Inst#"_W") LASX256:$xj, LASX256:$xk)>;
1300:   def : Pat<(v4i64 (setcc (v4i64 LASX256:$xj), (v4i64 LASX256:$xk), CC)),
1301:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, LASX256:$xk)>;
1302: }
1303: 
1304: multiclass PatCCXrXrU<CondCode CC, string Inst> {
1305:   def : Pat<(v32i8 (setcc (v32i8 LASX256:$xj), (v32i8 LASX256:$xk), CC)),
1306:             (!cast<LAInst>(Inst#"_BU") LASX256:$xj, LASX256:$xk)>;
1307:   def : Pat<(v16i16 (setcc (v16i16 LASX256:$xj), (v16i16 LASX256:$xk), CC)),
1308:             (!cast<LAInst>(Inst#"_HU") LASX256:$xj, LASX256:$xk)>;
1309:   def : Pat<(v8i32 (setcc (v8i32 LASX256:$xj), (v8i32 LASX256:$xk), CC)),
1310:             (!cast<LAInst>(Inst#"_WU") LASX256:$xj, LASX256:$xk)>;
1311:   def : Pat<(v4i64 (setcc (v4i64 LASX256:$xj), (v4i64 LASX256:$xk), CC)),
1312:             (!cast<LAInst>(Inst#"_DU") LASX256:$xj, LASX256:$xk)>;
1313: }
1314: 
1315: multiclass PatCCXrXrF<CondCode CC, string Inst> {
1316:   def : Pat<(v8i32 (setcc (v8f32 LASX256:$xj), (v8f32 LASX256:$xk), CC)),
1317:             (!cast<LAInst>(Inst#"_S") LASX256:$xj, LASX256:$xk)>;
1318:   def : Pat<(v4i64 (setcc (v4f64 LASX256:$xj), (v4f64 LASX256:$xk), CC)),
1319:             (!cast<LAInst>(Inst#"_D") LASX256:$xj, LASX256:$xk)>;
1320: }
```
- **EN**: This block declares or refines TableGen records such as `PatCCXrXr`, `PatCCXrXrU`, `PatCCXrXrF`.
- **CN**: 该代码块声明或细化了 `PatCCXrXr`, `PatCCXrXrU`, `PatCCXrXrF` 等 TableGen 记录。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321: 
1322: multiclass PairInsertExtractPatV8<ValueType vecty, ValueType elemty> {
1323:   foreach imm1 = 0...3 in {
1324:     foreach imm2 = 0...3 in {
1325:       defvar Imm = !or(!shl(imm2, 4), imm1);
1326:       def : Pat<(vector_insert (vector_insert vecty:$xd,
1327:                     (elemty (vector_extract vecty:$xj, imm1)), imm2),
1328:                     (elemty (vector_extract vecty:$xj, !add(imm1, 4))),
1329:                     !add(imm2, 4)),
1330:                 (XVEXTRINS_W $xd, $xj, Imm)>;
1331:     }
1332:   }
1333: }
1334: 
1335: multiclass PairInsertExtractPatV4<ValueType vecty, ValueType elemty> {
1336:   foreach imm1 = 0...1 in {
1337:     foreach imm2 = 0...1 in {
1338:       defvar Imm = !or(!shl(imm2, 4), imm1);
1339:       def : Pat<(vector_insert (vector_insert vecty:$xd,
1340:                     (elemty (vector_extract vecty:$xj, imm1)), imm2),
1341:                     (elemty (vector_extract vecty:$xj, !add(imm1, 2))),
1342:                     !add(imm2, 2)),
1343:                 (XVEXTRINS_D $xd, $xj, Imm)>;
1344:     }
1345:   }
1346: }
1347: 
1348: let Predicates = [HasExtLASX] in {
1349: 
1350: // XVADD_{B/H/W/D}
1351: defm : PatXrXr<add, "XVADD">;
1352: // XVSUB_{B/H/W/D}
1353: defm : PatXrXr<sub, "XVSUB">;
1354: 
1355: // XVADDI_{B/H/W/D}U
1356: defm : PatXrNimm5<sub, "XVADDI">;
1357: defm : PatXrUimm5<add, "XVADDI">;
1358: // XVSUBI_{B/H/W/D}U
1359: defm : PatXrNimm5<add, "XVSUBI">;
1360: defm : PatXrUimm5<sub, "XVSUBI">;
```
- **EN**: This block declares or refines TableGen records such as `PairInsertExtractPatV8`, `PairInsertExtractPatV4`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `PairInsertExtractPatV8`, `PairInsertExtractPatV4` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361: 
1362: // XVNEG_{B/H/W/D}
1363: def : Pat<(sub immAllZerosV, (v32i8 LASX256:$xj)), (XVNEG_B LASX256:$xj)>;
1364: def : Pat<(sub immAllZerosV, (v16i16 LASX256:$xj)), (XVNEG_H LASX256:$xj)>;
1365: def : Pat<(sub immAllZerosV, (v8i32 LASX256:$xj)), (XVNEG_W LASX256:$xj)>;
1366: def : Pat<(sub immAllZerosV, (v4i64 LASX256:$xj)), (XVNEG_D LASX256:$xj)>;
1367: 
1368: // XVMAX[I]_{B/H/W/D}[U]
1369: defm : PatXrXr<smax, "XVMAX">;
1370: defm : PatXrXrU<umax, "XVMAX">;
1371: defm : PatXrSimm5<smax, "XVMAXI">;
1372: defm : PatXrUimm5<umax, "XVMAXI">;
1373: 
1374: // XVMIN[I]_{B/H/W/D}[U]
1375: defm : PatXrXr<smin, "XVMIN">;
1376: defm : PatXrXrU<umin, "XVMIN">;
1377: defm : PatXrSimm5<smin, "XVMINI">;
1378: defm : PatXrUimm5<umin, "XVMINI">;
1379: 
1380: // XVMUL_{B/H/W/D}
1381: defm : PatXrXr<mul, "XVMUL">;
1382: 
1383: // XVMUH_{B/H/W/D}[U]
1384: defm : PatXrXr<mulhs, "XVMUH">;
1385: defm : PatXrXrU<mulhu, "XVMUH">;
1386: 
1387: // XVMADD_{B/H/W/D}
1388: defm : PatXrXrXr<muladd, "XVMADD">;
1389: // XVMSUB_{B/H/W/D}
1390: defm : PatXrXrXr<mulsub, "XVMSUB">;
1391: 
1392: // XVDIV_{B/H/W/D}[U]
1393: defm : PatXrXr<sdiv, "XVDIV">;
1394: defm : PatXrXrU<udiv, "XVDIV">;
1395: 
1396: // XVMOD_{B/H/W/D}[U]
1397: defm : PatXrXr<srem, "XVMOD">;
1398: defm : PatXrXrU<urem, "XVMOD">;
1399: 
1400: // XVAND_V
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401: foreach vt = [v32i8, v16i16, v8i32, v4i64] in
1402: def : Pat<(and (vt LASX256:$xj), (vt LASX256:$xk)),
1403:           (XVAND_V LASX256:$xj, LASX256:$xk)>;
1404: // XVOR_V
1405: foreach vt = [v32i8, v16i16, v8i32, v4i64] in
1406: def : Pat<(or (vt LASX256:$xj), (vt LASX256:$xk)),
1407:           (XVOR_V LASX256:$xj, LASX256:$xk)>;
1408: // XVXOR_V
1409: foreach vt = [v32i8, v16i16, v8i32, v4i64] in
1410: def : Pat<(xor (vt LASX256:$xj), (vt LASX256:$xk)),
1411:           (XVXOR_V LASX256:$xj, LASX256:$xk)>;
1412: // XVNOR_V
1413: foreach vt = [v32i8, v16i16, v8i32, v4i64] in
1414: def : Pat<(vnot (or (vt LASX256:$xj), (vt LASX256:$xk))),
1415:           (XVNOR_V LASX256:$xj, LASX256:$xk)>;
1416: // XVANDN_V
1417: foreach vt = [v32i8, v16i16, v8i32, v4i64] in
1418: def : Pat<(loongarch_vandn (vt LASX256:$xj), (vt LASX256:$xk)),
1419:           (XVANDN_V LASX256:$xj, LASX256:$xk)>;
1420: // XVORN_V
1421: foreach vt = [v32i8, v16i16, v8i32, v4i64] in
1422: def : Pat<(or (vt LASX256:$xj), (vt (vnot LASX256:$xk))),
1423:           (XVORN_V LASX256:$xj, LASX256:$xk)>;
1424: 
1425: // XVANDI_B
1426: def : Pat<(and (v32i8 LASX256:$xj), (v32i8 (SplatPat_uimm8 uimm8:$imm))),
1427:           (XVANDI_B LASX256:$xj, uimm8:$imm)>;
1428: foreach vt = [v16i16, v8i32, v4i64] in
1429: def : Pat<(and (vt LASX256:$xj), (vt (vsplat_i8_uimm8 grlenimm:$imm))),
1430:           (XVANDI_B LASX256:$xj, grlenimm:$imm)>;
1431: // XVORI_B
1432: def : Pat<(or (v32i8 LASX256:$xj), (v32i8 (SplatPat_uimm8 uimm8:$imm))),
1433:           (XVORI_B LASX256:$xj, uimm8:$imm)>;
1434: foreach vt = [v16i16, v8i32, v4i64] in
1435: def : Pat<(or (vt LASX256:$xj), (vt (vsplat_i8_uimm8 grlenimm:$imm))),
1436:           (XVORI_B LASX256:$xj, grlenimm:$imm)>;
1437: // XVXORI_B
1438: def : Pat<(xor (v32i8 LASX256:$xj), (v32i8 (SplatPat_uimm8 uimm8:$imm))),
1439:           (XVXORI_B LASX256:$xj, uimm8:$imm)>;
1440: foreach vt = [v16i16, v8i32, v4i64] in
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441: def : Pat<(xor (vt LASX256:$xj), (vt (vsplat_i8_uimm8 grlenimm:$imm))),
1442:           (XVXORI_B LASX256:$xj, grlenimm:$imm)>;
1443: // XVNORI_B
1444: def : Pat<(vnot (or (v32i8 LASX256:$xj), (v32i8 (SplatPat_uimm8 uimm8:$imm)))),
1445:           (XVNORI_B LASX256:$xj, uimm8:$imm)>;
1446: foreach vt = [v16i16, v8i32, v4i64] in
1447: def : Pat<(vnot (or (vt LASX256:$xj), (vt (vsplat_i8_uimm8 grlenimm:$imm)))),
1448:           (XVNORI_B LASX256:$xj, grlenimm:$imm)>;
1449: 
1450: // XVBSLL_V
1451: foreach vt = [v32i8, v16i16, v8i32, v4i64, v8f32,
1452:               v4f64] in def : Pat<(loongarch_vbsll(vt LASX256:$xj), uimm5:$imm),
1453:                                   (XVBSLL_V LASX256:$xj, uimm5:$imm)>;
1454: 
1455: // XVBSRL_V
1456: foreach vt = [v32i8, v16i16, v8i32, v4i64, v8f32,
1457:               v4f64] in def : Pat<(loongarch_vbsrl(vt LASX256:$xj), uimm5:$imm),
1458:                                   (XVBSRL_V LASX256:$xj, uimm5:$imm)>;
1459: 
1460: // XVSLL[I]_{B/H/W/D}
1461: defm : PatXrXr<shl, "XVSLL">;
1462: defm : PatShiftXrXr<shl, "XVSLL">;
1463: defm : PatShiftXrSplatUimm<shl, "XVSLLI">;
1464: defm : PatShiftXrUimm<loongarch_vslli, "XVSLLI">;
1465: 
1466: // XVSRL[I]_{B/H/W/D}
1467: defm : PatXrXr<srl, "XVSRL">;
1468: defm : PatShiftXrXr<srl, "XVSRL">;
1469: defm : PatShiftXrSplatUimm<srl, "XVSRLI">;
1470: defm : PatShiftXrUimm<loongarch_vsrli, "XVSRLI">;
1471: 
1472: // XVSRLR[I]_{B/H/W/D}
1473: defm : PatXrXr<loongarch_vsrlr, "XVSRLR">;
1474: defm : PatShiftXrSplatUimm<loongarch_vsrlr, "XVSRLRI">;
1475: 
1476: // XVSRA[I]_{B/H/W/D}
1477: defm : PatXrXr<sra, "XVSRA">;
1478: defm : PatShiftXrXr<sra, "XVSRA">;
1479: defm : PatShiftXrSplatUimm<sra, "XVSRAI">;
1480: 
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481: // XVSRAR[I]_{B/H/W/D}
1482: defm : PatXrXr<loongarch_vsrar, "XVSRAR">;
1483: defm : PatShiftXrSplatUimm<loongarch_vsrar, "XVSRARI">;
1484: 
1485: // XVROTR[I]_{B/H/W/D}
1486: defm : PatXrXr<rotr, "XVROTR">;
1487: defm : PatShiftXrXr<rotr, "XVROTR">;
1488: defm : PatShiftXrSplatUimm<rotr, "XVROTRI">;
1489: 
1490: // XVCLO_{B/H/W/D}
1491: def : Pat<(ctlz (vnot v32i8:$xj)), (XVCLO_B v32i8:$xj)>;
1492: def : Pat<(ctlz (vnot v16i16:$xj)), (XVCLO_H v16i16:$xj)>;
1493: def : Pat<(ctlz (vnot v8i32:$xj)), (XVCLO_W v8i32:$xj)>;
1494: def : Pat<(ctlz (vnot v4i64:$xj)), (XVCLO_D v4i64:$xj)>;
1495: 
1496: // XVCLZ_{B/H/W/D}
1497: defm : PatXr<ctlz, "XVCLZ">;
1498: 
1499: // XVPCNT_{B/H/W/D}
1500: defm : PatXr<ctpop, "XVPCNT">;
1501: 
1502: // XVBITCLR_{B/H/W/D}
1503: def : Pat<(loongarch_vandn (v32i8 (shl vsplat_imm_eq_1, v32i8:$xk)), v32i8:$xj),
1504:           (v32i8 (XVBITCLR_B v32i8:$xj, v32i8:$xk))>;
1505: def : Pat<(loongarch_vandn (v16i16 (shl vsplat_imm_eq_1, v16i16:$xk)), v16i16:$xj),
1506:           (v16i16 (XVBITCLR_H v16i16:$xj, v16i16:$xk))>;
1507: def : Pat<(loongarch_vandn (v8i32 (shl vsplat_imm_eq_1, v8i32:$xk)), v8i32:$xj),
1508:           (v8i32 (XVBITCLR_W v8i32:$xj, v8i32:$xk))>;
1509: def : Pat<(loongarch_vandn (v4i64 (shl vsplat_imm_eq_1, v4i64:$xk)), v4i64:$xj),
1510:           (v4i64 (XVBITCLR_D v4i64:$xj, v4i64:$xk))>;
1511: def : Pat<(loongarch_vandn (v32i8 (shl vsplat_imm_eq_1,
1512:                                   (vsplati8imm7 v32i8:$xk))), v32i8:$xj),
1513:           (v32i8 (XVBITCLR_B v32i8:$xj, v32i8:$xk))>;
1514: def : Pat<(loongarch_vandn (v16i16 (shl vsplat_imm_eq_1,
1515:                                    (vsplati16imm15 v16i16:$xk))), v16i16:$xj),
1516:           (v16i16 (XVBITCLR_H v16i16:$xj, v16i16:$xk))>;
1517: def : Pat<(loongarch_vandn (v8i32 (shl vsplat_imm_eq_1,
1518:                                   (vsplati32imm31 v8i32:$xk))), v8i32:$xj),
1519:           (v8i32 (XVBITCLR_W v8i32:$xj, v8i32:$xk))>;
1520: def : Pat<(loongarch_vandn (v4i64 (shl vsplat_imm_eq_1,
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521:                                   (vsplati64imm63 v4i64:$xk))), v4i64:$xj),
1522:           (v4i64 (XVBITCLR_D v4i64:$xj, v4i64:$xk))>;
1523: 
1524: // XVBITCLRI_{B/H/W/D}
1525: def : Pat<(and (v32i8 LASX256:$xj), (v32i8 (vsplat_uimm_inv_pow2 uimm3:$imm))),
1526:           (XVBITCLRI_B LASX256:$xj, uimm3:$imm)>;
1527: def : Pat<(and (v16i16 LASX256:$xj), (v16i16 (vsplat_uimm_inv_pow2 uimm4:$imm))),
1528:           (XVBITCLRI_H LASX256:$xj, uimm4:$imm)>;
1529: def : Pat<(and (v8i32 LASX256:$xj), (v8i32 (vsplat_uimm_inv_pow2 uimm5:$imm))),
1530:           (XVBITCLRI_W LASX256:$xj, uimm5:$imm)>;
1531: def : Pat<(and (v4i64 LASX256:$xj), (v4i64 (vsplat_uimm_inv_pow2 uimm6:$imm))),
1532:           (XVBITCLRI_D LASX256:$xj, uimm6:$imm)>;
1533: 
1534: foreach vt = [v16i16, v8i32, v4i64] in {
1535:   def : Pat<(and (vt LASX256:$xj), (vt (vsplat_i8_inv_pow2 grlenimm:$imm))),
1536:             (XVBITCLRI_B LASX256:$xj, grlenimm:$imm)>;
1537:   def : Pat<(and (vt LASX256:$xj), (vt (vsplat_i16_inv_pow2 grlenimm:$imm))),
1538:             (XVBITCLRI_H LASX256:$xj, grlenimm:$imm)>;
1539:   def : Pat<(and (vt LASX256:$xj), (vt (vsplat_i32_inv_pow2 grlenimm:$imm))),
1540:             (XVBITCLRI_W LASX256:$xj, grlenimm:$imm)>;
1541: }
1542: 
1543: // XVBITSET_{B/H/W/D}
1544: def : Pat<(or v32i8:$xj, (shl vsplat_imm_eq_1, v32i8:$xk)),
1545:           (v32i8 (XVBITSET_B v32i8:$xj, v32i8:$xk))>;
1546: def : Pat<(or v16i16:$xj, (shl vsplat_imm_eq_1, v16i16:$xk)),
1547:           (v16i16 (XVBITSET_H v16i16:$xj, v16i16:$xk))>;
1548: def : Pat<(or v8i32:$xj, (shl vsplat_imm_eq_1, v8i32:$xk)),
1549:           (v8i32 (XVBITSET_W v8i32:$xj, v8i32:$xk))>;
1550: def : Pat<(or v4i64:$xj, (shl vsplat_imm_eq_1, v4i64:$xk)),
1551:           (v4i64 (XVBITSET_D v4i64:$xj, v4i64:$xk))>;
1552: def : Pat<(or v32i8:$xj, (shl vsplat_imm_eq_1, (vsplati8imm7 v32i8:$xk))),
1553:           (v32i8 (XVBITSET_B v32i8:$xj, v32i8:$xk))>;
1554: def : Pat<(or v16i16:$xj, (shl vsplat_imm_eq_1, (vsplati16imm15 v16i16:$xk))),
1555:           (v16i16 (XVBITSET_H v16i16:$xj, v16i16:$xk))>;
1556: def : Pat<(or v8i32:$xj, (shl vsplat_imm_eq_1, (vsplati32imm31 v8i32:$xk))),
1557:           (v8i32 (XVBITSET_W v8i32:$xj, v8i32:$xk))>;
1558: def : Pat<(or v4i64:$xj, (shl vsplat_imm_eq_1, (vsplati64imm63 v4i64:$xk))),
1559:           (v4i64 (XVBITSET_D v4i64:$xj, v4i64:$xk))>;
1560: 
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561: // XVBITSETI_{B/H/W/D}
1562: def : Pat<(or (v32i8 LASX256:$xj), (v32i8 (vsplat_uimm_pow2 uimm3:$imm))),
1563:           (XVBITSETI_B LASX256:$xj, uimm3:$imm)>;
1564: def : Pat<(or (v16i16 LASX256:$xj), (v16i16 (vsplat_uimm_pow2 uimm4:$imm))),
1565:           (XVBITSETI_H LASX256:$xj, uimm4:$imm)>;
1566: def : Pat<(or (v8i32 LASX256:$xj), (v8i32 (vsplat_uimm_pow2 uimm5:$imm))),
1567:           (XVBITSETI_W LASX256:$xj, uimm5:$imm)>;
1568: def : Pat<(or (v4i64 LASX256:$xj), (v4i64 (vsplat_uimm_pow2 uimm6:$imm))),
1569:           (XVBITSETI_D LASX256:$xj, uimm6:$imm)>;
1570: 
1571: foreach vt = [v16i16, v8i32, v4i64] in {
1572:   def : Pat<(or (vt LASX256:$xj), (vt (vsplat_i8_pow2 grlenimm:$imm))),
1573:             (XVBITSETI_B LASX256:$xj, grlenimm:$imm)>;
1574:   def : Pat<(or (vt LASX256:$xj), (vt (vsplat_i16_pow2 grlenimm:$imm))),
1575:             (XVBITSETI_H LASX256:$xj, grlenimm:$imm)>;
1576:   def : Pat<(or (vt LASX256:$xj), (vt (vsplat_i32_pow2 grlenimm:$imm))),
1577:             (XVBITSETI_W LASX256:$xj, grlenimm:$imm)>;
1578: }
1579: 
1580: // XVBITREV_{B/H/W/D}
1581: def : Pat<(xor v32i8:$xj, (shl vsplat_imm_eq_1, v32i8:$xk)),
1582:           (v32i8 (XVBITREV_B v32i8:$xj, v32i8:$xk))>;
1583: def : Pat<(xor v16i16:$xj, (shl vsplat_imm_eq_1, v16i16:$xk)),
1584:           (v16i16 (XVBITREV_H v16i16:$xj, v16i16:$xk))>;
1585: def : Pat<(xor v8i32:$xj, (shl vsplat_imm_eq_1, v8i32:$xk)),
1586:           (v8i32 (XVBITREV_W v8i32:$xj, v8i32:$xk))>;
1587: def : Pat<(xor v4i64:$xj, (shl vsplat_imm_eq_1, v4i64:$xk)),
1588:           (v4i64 (XVBITREV_D v4i64:$xj, v4i64:$xk))>;
1589: def : Pat<(xor v32i8:$xj, (shl vsplat_imm_eq_1, (vsplati8imm7 v32i8:$xk))),
1590:           (v32i8 (XVBITREV_B v32i8:$xj, v32i8:$xk))>;
1591: def : Pat<(xor v16i16:$xj, (shl vsplat_imm_eq_1, (vsplati16imm15 v16i16:$xk))),
1592:           (v16i16 (XVBITREV_H v16i16:$xj, v16i16:$xk))>;
1593: def : Pat<(xor v8i32:$xj, (shl vsplat_imm_eq_1, (vsplati32imm31 v8i32:$xk))),
1594:           (v8i32 (XVBITREV_W v8i32:$xj, v8i32:$xk))>;
1595: def : Pat<(xor v4i64:$xj, (shl vsplat_imm_eq_1, (vsplati64imm63 v4i64:$xk))),
1596:           (v4i64 (XVBITREV_D v4i64:$xj, v4i64:$xk))>;
1597: 
1598: // XVBITREVI_{B/H/W/D}
1599: def : Pat<(xor (v32i8 LASX256:$xj), (v32i8 (vsplat_uimm_pow2 uimm3:$imm))),
1600:           (XVBITREVI_B LASX256:$xj, uimm3:$imm)>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601: def : Pat<(xor (v16i16 LASX256:$xj), (v16i16 (vsplat_uimm_pow2 uimm4:$imm))),
1602:           (XVBITREVI_H LASX256:$xj, uimm4:$imm)>;
1603: def : Pat<(xor (v8i32 LASX256:$xj), (v8i32 (vsplat_uimm_pow2 uimm5:$imm))),
1604:           (XVBITREVI_W LASX256:$xj, uimm5:$imm)>;
1605: def : Pat<(xor (v4i64 LASX256:$xj), (v4i64 (vsplat_uimm_pow2 uimm6:$imm))),
1606:           (XVBITREVI_D LASX256:$xj, uimm6:$imm)>;
1607: 
1608: foreach vt = [v16i16, v8i32, v4i64] in {
1609:   def : Pat<(xor (vt LASX256:$xj), (vt (vsplat_i8_pow2 grlenimm:$imm))),
1610:             (XVBITREVI_B LASX256:$xj, grlenimm:$imm)>;
1611:   def : Pat<(xor (vt LASX256:$xj), (vt (vsplat_i16_pow2 grlenimm:$imm))),
1612:             (XVBITREVI_H LASX256:$xj, grlenimm:$imm)>;
1613:   def : Pat<(xor (vt LASX256:$xj), (vt (vsplat_i32_pow2 grlenimm:$imm))),
1614:             (XVBITREVI_W LASX256:$xj, grlenimm:$imm)>;
1615: }
1616: 
1617: // XVBITSEL_V
1618: foreach vt = [v32i8, v16i16, v8i32, v4i64] in
1619:   def : Pat<(vt (or (and LASX256:$va, LASX256:$vk),
1620:                     (loongarch_vandn LASX256:$va, LASX256:$vj))),
1621:             (vt (XVBITSEL_V LASX256:$vj, LASX256:$vk, LASX256:$va))>;
1622: 
1623: // XVBITSELI_B
1624: def : Pat<(v32i8 (or (and LASX256:$vd, (v32i8 (SplatPat_uimm8 uimm8:$imm))),
1625:                      (loongarch_vandn LASX256:$vd, LASX256:$vj))),
1626:           (XVBITSELI_B LASX256:$vd, LASX256:$vj, uimm8:$imm)>;
1627: 
1628: // Vector bswaps
1629: def : Pat<(bswap (v16i16 LASX256:$xj)), (XVSHUF4I_B LASX256:$xj, 0b10110001)>;
1630: def : Pat<(bswap (v8i32 LASX256:$xj)), (XVSHUF4I_B LASX256:$xj, 0b00011011)>;
1631: def : Pat<(bswap (v4i64 LASX256:$xj)),
1632:           (XVSHUF4I_W (XVSHUF4I_B LASX256:$xj, 0b00011011), 0b10110001)>;
1633: 
1634: // XVHADDW_{H_B/W_H/D_W/Q_D}
1635: defm : PatXrXrW<loongarch_vhaddw, "XVHADDW">;
1636: 
1637: // XVFADD_{S/D}
1638: defm : PatXrXrF<fadd, "XVFADD">;
1639: 
1640: // XVFSUB_{S/D}
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1641-1680 / 第 1641-1680 行
```tablegen
1641: defm : PatXrXrF<fsub, "XVFSUB">;
1642: 
1643: // XVFMUL_{S/D}
1644: defm : PatXrXrF<fmul, "XVFMUL">;
1645: 
1646: // XVFDIV_{S/D}
1647: defm : PatXrXrF<fdiv, "XVFDIV">;
1648: 
1649: // XVFMAX_{S/D}, XVFMIN_{S/D}
1650: defm : PatXrXrF<fmaxnum, "XVFMAX">;
1651: defm : PatXrXrF<fminnum, "XVFMIN">;
1652: 
1653: // XVFMADD_{S/D}
1654: def : Pat<(fma v8f32:$xj, v8f32:$xk, v8f32:$xa),
1655:           (XVFMADD_S v8f32:$xj, v8f32:$xk, v8f32:$xa)>;
1656: def : Pat<(fma v4f64:$xj, v4f64:$xk, v4f64:$xa),
1657:           (XVFMADD_D v4f64:$xj, v4f64:$xk, v4f64:$xa)>;
1658: 
1659: // XVFMSUB_{S/D}
1660: def : Pat<(fma v8f32:$xj, v8f32:$xk, (fneg v8f32:$xa)),
1661:           (XVFMSUB_S v8f32:$xj, v8f32:$xk, v8f32:$xa)>;
1662: def : Pat<(fma v4f64:$xj, v4f64:$xk, (fneg v4f64:$xa)),
1663:           (XVFMSUB_D v4f64:$xj, v4f64:$xk, v4f64:$xa)>;
1664: 
1665: // XVFNMADD_{S/D}
1666: def : Pat<(fneg (fma v8f32:$xj, v8f32:$xk, v8f32:$xa)),
1667:           (XVFNMADD_S v8f32:$xj, v8f32:$xk, v8f32:$xa)>;
1668: def : Pat<(fneg (fma v4f64:$xj, v4f64:$xk, v4f64:$xa)),
1669:           (XVFNMADD_D v4f64:$xj, v4f64:$xk, v4f64:$xa)>;
1670: def : Pat<(fma_nsz (fneg v8f32:$xj), v8f32:$xk, (fneg v8f32:$xa)),
1671:           (XVFNMADD_S v8f32:$xj, v8f32:$xk, v8f32:$xa)>;
1672: def : Pat<(fma_nsz (fneg v4f64:$xj), v4f64:$xk, (fneg v4f64:$xa)),
1673:           (XVFNMADD_D v4f64:$xj, v4f64:$xk, v4f64:$xa)>;
1674: 
1675: // XVFNMSUB_{S/D}
1676: def : Pat<(fneg (fma v8f32:$xj, v8f32:$xk, (fneg v8f32:$xa))),
1677:           (XVFNMSUB_S v8f32:$xj, v8f32:$xk, v8f32:$xa)>;
1678: def : Pat<(fneg (fma v4f64:$xj, v4f64:$xk, (fneg v4f64:$xa))),
1679:           (XVFNMSUB_D v4f64:$xj, v4f64:$xk, v4f64:$xa)>;
1680: def : Pat<(fma_nsz (fneg v8f32:$xj), v8f32:$xk, v8f32:$xa),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1681-1720 / 第 1681-1720 行
```tablegen
1681:           (XVFNMSUB_S v8f32:$xj, v8f32:$xk, v8f32:$xa)>;
1682: def : Pat<(fma_nsz (fneg v4f64:$xj), v4f64:$xk, v4f64:$xa),
1683:           (XVFNMSUB_D v4f64:$xj, v4f64:$xk, v4f64:$xa)>;
1684: 
1685: // XVFSQRT_{S/D}
1686: defm : PatXrF<fsqrt, "XVFSQRT">;
1687: 
1688: // XVRECIP_{S/D}
1689: def : Pat<(fdiv vsplatf32_fpimm_eq_1, v8f32:$xj),
1690:           (XVFRECIP_S v8f32:$xj)>;
1691: def : Pat<(fdiv vsplatf64_fpimm_eq_1, v4f64:$xj),
1692:           (XVFRECIP_D v4f64:$xj)>;
1693: 
1694: // XVFRSQRT_{S/D}
1695: def : Pat<(fdiv vsplatf32_fpimm_eq_1, (fsqrt v8f32:$xj)),
1696:           (XVFRSQRT_S v8f32:$xj)>;
1697: def : Pat<(fdiv vsplatf64_fpimm_eq_1, (fsqrt v4f64:$xj)),
1698:           (XVFRSQRT_D v4f64:$xj)>;
1699: 
1700: // XVSEQ[I]_{B/H/W/D}
1701: defm : PatCCXrSimm5<SETEQ, "XVSEQI">;
1702: defm : PatCCXrXr<SETEQ, "XVSEQ">;
1703: 
1704: // XVSLE[I]_{B/H/W/D}[U]
1705: defm : PatCCXrSimm5<SETLE, "XVSLEI">;
1706: defm : PatCCXrUimm5<SETULE, "XVSLEI">;
1707: defm : PatCCXrXr<SETLE, "XVSLE">;
1708: defm : PatCCXrXrU<SETULE, "XVSLE">;
1709: 
1710: // XVSLT[I]_{B/H/W/D}[U]
1711: defm : PatCCXrSimm5<SETLT, "XVSLTI">;
1712: defm : PatCCXrUimm5<SETULT, "XVSLTI">;
1713: defm : PatCCXrXr<SETLT, "XVSLT">;
1714: defm : PatCCXrXrU<SETULT, "XVSLT">;
1715: 
1716: // XVFCMP.cond.{S/D}
1717: defm : PatCCXrXrF<SETEQ, "XVFCMP_CEQ">;
1718: defm : PatCCXrXrF<SETOEQ, "XVFCMP_CEQ">;
1719: defm : PatCCXrXrF<SETUEQ, "XVFCMP_CUEQ">;
1720: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1721-1760 / 第 1721-1760 行
```tablegen
1721: defm : PatCCXrXrF<SETLE, "XVFCMP_CLE">;
1722: defm : PatCCXrXrF<SETOLE, "XVFCMP_CLE">;
1723: defm : PatCCXrXrF<SETULE, "XVFCMP_CULE">;
1724: 
1725: defm : PatCCXrXrF<SETLT, "XVFCMP_CLT">;
1726: defm : PatCCXrXrF<SETOLT, "XVFCMP_CLT">;
1727: defm : PatCCXrXrF<SETULT, "XVFCMP_CULT">;
1728: 
1729: defm : PatCCXrXrF<SETNE, "XVFCMP_CNE">;
1730: defm : PatCCXrXrF<SETONE, "XVFCMP_CNE">;
1731: defm : PatCCXrXrF<SETUNE, "XVFCMP_CUNE">;
1732: 
1733: defm : PatCCXrXrF<SETO, "XVFCMP_COR">;
1734: defm : PatCCXrXrF<SETUO, "XVFCMP_CUN">;
1735: 
1736: // Insert two elements extracted from vector into vector. (The positions
1737: // of the two elements must be same in the source or destination vector's
1738: // front and back 128bits.)
1739: // 2*XVPICKVE2GR_{W/D} + 2*XVINSGR2VR_{W/D} -> XVEXTRINS_{W/D}
1740: // XVPERMI_D + 2*XVPICKVE2GR_{B/H} + 2*PseudoXVINSGR2VR_{B/H} -> XVEXTRINS_{W/D}
1741: foreach imm1 = 0...15 in {
1742:   foreach imm2 = 0...15 in {
1743:     defvar Imm = !or(!shl(imm2, 4), imm1);
1744:     def : Pat<(vector_insert (vector_insert v32i8:$xd,
1745:                   (GRLenVT (vector_extract v32i8:$xj, imm1)), imm2),
1746:                   (GRLenVT (vector_extract v32i8:$xj, !add(imm1, 16))),
1747:                   !add(imm2, 16)),
1748:               (XVEXTRINS_B $xd, $xj, Imm)>;
1749:   }
1750: }
1751: 
1752: foreach imm1 = 0...7 in {
1753:   foreach imm2 = 0...7 in {
1754:     defvar Imm = !or(!shl(imm2, 4), imm1);
1755:     def : Pat<(vector_insert (vector_insert v16i16:$xd,
1756:                   (GRLenVT (vector_extract v16i16:$xj, imm1)), imm2),
1757:                   (GRLenVT (vector_extract v16i16:$xj, !add(imm1, 8))),
1758:                   !add(imm2, 8)),
1759:               (XVEXTRINS_H $xd, $xj, Imm)>;
1760:   }
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1761-1800 / 第 1761-1800 行
```tablegen
1761: }
1762: 
1763: defm : PairInsertExtractPatV8<v8i32, GRLenVT>;
1764: defm : PairInsertExtractPatV8<v8f32, f32>;
1765: defm : PairInsertExtractPatV4<v4i64, GRLenVT>;
1766: defm : PairInsertExtractPatV4<v4f64, f64>;
1767: 
1768: def : Pat<(vector_insert v8i32:$xd, (GRLenVT(vector_extract v8i32:$xj, 0)),
1769:               uimm3:$imm),
1770:           (XVINSVE0_W v8i32:$xd, v8i32:$xj, uimm3:$imm)>;
1771: 
1772: def : Pat<(vector_insert v4i64:$xd, (GRLenVT(vector_extract v4i64:$xj, 0)),
1773:               uimm2:$imm),
1774:           (XVINSVE0_D v4i64:$xd, v4i64:$xj, uimm2:$imm)>;
1775: 
1776: def : Pat<(vector_insert v8i32:$xd,
1777:               (GRLenVT(vector_extract v8i32:$xj, uimm3:$imm1)), uimm3:$imm2),
1778:           (XVINSVE0_W v8i32:$xd, (XVPICKVE_W v8i32:$xj, uimm3:$imm1),
1779:               uimm3:$imm2)>;
1780: 
1781: def : Pat<(vector_insert v4i64:$xd,
1782:               (GRLenVT(vector_extract v4i64:$xj, uimm2:$imm1)), uimm2:$imm2),
1783:           (XVINSVE0_D v4i64:$xd, (XVPICKVE_D v4i64:$xj, uimm2:$imm1),
1784:               uimm2:$imm2)>;
1785: 
1786: // PseudoXVINSGR2VR_{B/H}
1787: def : Pat<(vector_insert v32i8:$xd, GRLenVT:$rj, uimm5:$imm),
1788:           (PseudoXVINSGR2VR_B v32i8:$xd, GRLenVT:$rj, uimm5:$imm)>;
1789: def : Pat<(vector_insert v16i16:$xd, GRLenVT:$rj, uimm4:$imm),
1790:           (PseudoXVINSGR2VR_H v16i16:$xd, GRLenVT:$rj, uimm4:$imm)>;
1791: 
1792: // XVINSGR2VR_{W/D}
1793: def : Pat<(vector_insert v8i32:$xd, GRLenVT:$rj, uimm3:$imm),
1794:           (XVINSGR2VR_W v8i32:$xd, GRLenVT:$rj, uimm3:$imm)>;
1795: def : Pat<(vector_insert v4i64:$xd, GRLenVT:$rj, uimm2:$imm),
1796:           (XVINSGR2VR_D v4i64:$xd, GRLenVT:$rj, uimm2:$imm)>;
1797: def : Pat<(vector_insert v8f32:$xd, (loongarch_movgr2fr_w_la64 GPR:$rj),
1798:               uimm3:$imm),
1799:           (XVINSGR2VR_W v8f32:$xd, GPR:$rj, uimm3:$imm)>;
1800: def : Pat<(vector_insert v4f64:$xd, (f64(bitconvert i64:$rj)), uimm2:$imm),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1801-1840 / 第 1801-1840 行
```tablegen
1801:           (XVINSGR2VR_D v4f64:$xd, GPR:$rj, uimm2:$imm)>;
1802: 
1803: // XVINSVE0_{W/D}
1804: def : Pat<(loongarch_xvinsve0 v8i32:$xd, v8i32:$xj, uimm3:$imm),
1805:           (XVINSVE0_W v8i32:$xd, v8i32:$xj, uimm3:$imm)>;
1806: def : Pat<(loongarch_xvinsve0 v4i64:$xd, v4i64:$xj, uimm2:$imm),
1807:           (XVINSVE0_D v4i64:$xd, v4i64:$xj, uimm2:$imm)>;
1808: def : Pat<(loongarch_xvinsve0 v8f32:$xd, v8f32:$xj, uimm3:$imm),
1809:           (XVINSVE0_W v8f32:$xd, v8f32:$xj, uimm3:$imm)>;
1810: def : Pat<(loongarch_xvinsve0 v4f64:$xd, v4f64:$xj, uimm2:$imm),
1811:           (XVINSVE0_D v4f64:$xd, v4f64:$xj, uimm2:$imm)>;
1812: def : Pat<(vector_insert v8f32:$xd, FPR32:$fj, uimm3:$imm),
1813:           (XVINSVE0_W v8f32:$xd, (SUBREG_TO_REG FPR32:$fj, sub_32),
1814:               uimm3:$imm)>;
1815: def : Pat<(vector_insert v4f64:$xd, FPR64:$fj, uimm2:$imm),
1816:           (XVINSVE0_D v4f64:$xd, (SUBREG_TO_REG FPR64:$fj, sub_64),
1817:               uimm2:$imm)>;
1818: 
1819: // scalar_to_vector
1820: def : Pat<(v8f32 (scalar_to_vector FPR32:$fj)),
1821:           (SUBREG_TO_REG FPR32:$fj, sub_32)>;
1822: def : Pat<(v4f64 (scalar_to_vector FPR64:$fj)),
1823:           (SUBREG_TO_REG FPR64:$fj, sub_64)>;
1824: 
1825: // XVPICKVE2GR_W[U]
1826: def : Pat<(loongarch_vpick_sext_elt v8i32:$xd, uimm3:$imm, i32),
1827:           (XVPICKVE2GR_W v8i32:$xd, uimm3:$imm)>;
1828: def : Pat<(loongarch_vpick_zext_elt v8i32:$xd, uimm3:$imm, i32),
1829:           (XVPICKVE2GR_WU v8i32:$xd, uimm3:$imm)>;
1830: 
1831: // XVREPLGR2VR_{B/H/W/D}
1832: def : Pat<(lasxsplati8 GPR:$rj), (XVREPLGR2VR_B GPR:$rj)>;
1833: def : Pat<(lasxsplati16 GPR:$rj), (XVREPLGR2VR_H GPR:$rj)>;
1834: def : Pat<(lasxsplati32 GPR:$rj), (XVREPLGR2VR_W GPR:$rj)>;
1835: def : Pat<(lasxsplati64 GPR:$rj), (XVREPLGR2VR_D GPR:$rj)>;
1836: 
1837: def : Pat<(v32i8 (loongarch_vreplgr2vr GRLenVT:$rj)),
1838:           (v32i8 (XVREPLGR2VR_B GRLenVT:$rj))>;
1839: def : Pat<(v16i16 (loongarch_vreplgr2vr GRLenVT:$rj)),
1840:           (v16i16 (XVREPLGR2VR_H GRLenVT:$rj))>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1841-1880 / 第 1841-1880 行
```tablegen
1841: def : Pat<(v8i32 (loongarch_vreplgr2vr GRLenVT:$rj)),
1842:           (v8i32 (XVREPLGR2VR_W GRLenVT:$rj))>;
1843: def : Pat<(v4i64 (loongarch_vreplgr2vr GRLenVT:$rj)),
1844:           (v4i64 (XVREPLGR2VR_D GRLenVT:$rj))>;
1845: 
1846: // XVREPLVE_{B/H/W/D}
1847: def : Pat<(loongarch_vreplve v32i8:$xj, GRLenVT:$rk),
1848:           (XVREPLVE_B v32i8:$xj, GRLenVT:$rk)>;
1849: def : Pat<(loongarch_vreplve v16i16:$xj, GRLenVT:$rk),
1850:           (XVREPLVE_H v16i16:$xj, GRLenVT:$rk)>;
1851: def : Pat<(loongarch_vreplve v8i32:$xj, GRLenVT:$rk),
1852:           (XVREPLVE_W v8i32:$xj, GRLenVT:$rk)>;
1853: def : Pat<(loongarch_vreplve v4i64:$xj, GRLenVT:$rk),
1854:           (XVREPLVE_D v4i64:$xj, GRLenVT:$rk)>;
1855: 
1856: // XVSHUF_{B/H/W/D}
1857: def : Pat<(loongarch_vshuf v32i8:$xa, v32i8:$xj, v32i8:$xk),
1858:           (XVSHUF_B v32i8:$xj, v32i8:$xk, v32i8:$xa)>;
1859: def : Pat<(loongarch_vshuf v16i16:$xd, v16i16:$xj, v16i16:$xk),
1860:           (XVSHUF_H v16i16:$xd, v16i16:$xj, v16i16:$xk)>;
1861: def : Pat<(loongarch_vshuf v8i32:$xd, v8i32:$xj, v8i32:$xk),
1862:           (XVSHUF_W v8i32:$xd, v8i32:$xj, v8i32:$xk)>;
1863: def : Pat<(loongarch_vshuf v4i64:$xd, v4i64:$xj, v4i64:$xk),
1864:           (XVSHUF_D v4i64:$xd, v4i64:$xj, v4i64:$xk)>;
1865: def : Pat<(loongarch_vshuf v8i32:$xd, v8f32:$xj, v8f32:$xk),
1866:           (XVSHUF_W v8i32:$xd, v8f32:$xj, v8f32:$xk)>;
1867: def : Pat<(loongarch_vshuf v4i64:$xd, v4f64:$xj, v4f64:$xk),
1868:           (XVSHUF_D v4i64:$xd, v4f64:$xj, v4f64:$xk)>;
1869: 
1870: // XVPICKEV_{B/H/W/D}
1871: def : Pat<(loongarch_vpickev v32i8:$xj, v32i8:$xk),
1872:           (XVPICKEV_B v32i8:$xj, v32i8:$xk)>;
1873: def : Pat<(loongarch_vpickev v16i16:$xj, v16i16:$xk),
1874:           (XVPICKEV_H v16i16:$xj, v16i16:$xk)>;
1875: def : Pat<(loongarch_vpickev v8i32:$xj, v8i32:$xk),
1876:           (XVPICKEV_W v8i32:$xj, v8i32:$xk)>;
1877: def : Pat<(loongarch_vpickev v4i64:$xj, v4i64:$xk),
1878:           (XVPICKEV_D v4i64:$xj, v4i64:$xk)>;
1879: def : Pat<(loongarch_vpickev v8f32:$xj, v8f32:$xk),
1880:           (XVPICKEV_W v8f32:$xj, v8f32:$xk)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1881-1920 / 第 1881-1920 行
```tablegen
1881: def : Pat<(loongarch_vpickev v4f64:$xj, v4f64:$xk),
1882:           (XVPICKEV_D v4f64:$xj, v4f64:$xk)>;
1883: 
1884: // XVPICKOD_{B/H/W/D}
1885: def : Pat<(loongarch_vpickod v32i8:$xj, v32i8:$xk),
1886:           (XVPICKOD_B v32i8:$xj, v32i8:$xk)>;
1887: def : Pat<(loongarch_vpickod v16i16:$xj, v16i16:$xk),
1888:           (XVPICKOD_H v16i16:$xj, v16i16:$xk)>;
1889: def : Pat<(loongarch_vpickod v8i32:$xj, v8i32:$xk),
1890:           (XVPICKOD_W v8i32:$xj, v8i32:$xk)>;
1891: def : Pat<(loongarch_vpickod v4i64:$xj, v4i64:$xk),
1892:           (XVPICKOD_D v4i64:$xj, v4i64:$xk)>;
1893: def : Pat<(loongarch_vpickod v8f32:$xj, v8f32:$xk),
1894:           (XVPICKOD_W v8f32:$xj, v8f32:$xk)>;
1895: def : Pat<(loongarch_vpickod v4f64:$xj, v4f64:$xk),
1896:           (XVPICKOD_D v4f64:$xj, v4f64:$xk)>;
1897: 
1898: // XVPACKEV_{B/H/W/D}
1899: def : Pat<(loongarch_vpackev v32i8:$xj, v32i8:$xk),
1900:           (XVPACKEV_B v32i8:$xj, v32i8:$xk)>;
1901: def : Pat<(loongarch_vpackev v16i16:$xj, v16i16:$xk),
1902:           (XVPACKEV_H v16i16:$xj, v16i16:$xk)>;
1903: def : Pat<(loongarch_vpackev v8i32:$xj, v8i32:$xk),
1904:           (XVPACKEV_W v8i32:$xj, v8i32:$xk)>;
1905: def : Pat<(loongarch_vpackev v4i64:$xj, v4i64:$xk),
1906:           (XVPACKEV_D v4i64:$xj, v4i64:$xk)>;
1907: def : Pat<(loongarch_vpackev v8f32:$xj, v8f32:$xk),
1908:           (XVPACKEV_W v8f32:$xj, v8f32:$xk)>;
1909: def : Pat<(loongarch_vpackev v4f64:$xj, v4f64:$xk),
1910:           (XVPACKEV_D v4f64:$xj, v4f64:$xk)>;
1911: 
1912: // XVPACKOD_{B/H/W/D}
1913: def : Pat<(loongarch_vpackod v32i8:$xj, v32i8:$xk),
1914:           (XVPACKOD_B v32i8:$xj, v32i8:$xk)>;
1915: def : Pat<(loongarch_vpackod v16i16:$xj, v16i16:$xk),
1916:           (XVPACKOD_H v16i16:$xj, v16i16:$xk)>;
1917: def : Pat<(loongarch_vpackod v8i32:$xj, v8i32:$xk),
1918:           (XVPACKOD_W v8i32:$xj, v8i32:$xk)>;
1919: def : Pat<(loongarch_vpackod v4i64:$xj, v4i64:$xk),
1920:           (XVPACKOD_D v4i64:$xj, v4i64:$xk)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1921-1960 / 第 1921-1960 行
```tablegen
1921: def : Pat<(loongarch_vpackod v8f32:$xj, v8f32:$xk),
1922:           (XVPACKOD_W v8f32:$xj, v8f32:$xk)>;
1923: def : Pat<(loongarch_vpackod v4f64:$xj, v4f64:$xk),
1924:           (XVPACKOD_D v4f64:$xj, v4f64:$xk)>;
1925: 
1926: // XVILVL_{B/H/W/D}
1927: def : Pat<(loongarch_vilvl v32i8:$xj, v32i8:$xk),
1928:           (XVILVL_B v32i8:$xj, v32i8:$xk)>;
1929: def : Pat<(loongarch_vilvl v16i16:$xj, v16i16:$xk),
1930:           (XVILVL_H v16i16:$xj, v16i16:$xk)>;
1931: def : Pat<(loongarch_vilvl v8i32:$xj, v8i32:$xk),
1932:           (XVILVL_W v8i32:$xj, v8i32:$xk)>;
1933: def : Pat<(loongarch_vilvl v4i64:$xj, v4i64:$xk),
1934:           (XVILVL_D v4i64:$xj, v4i64:$xk)>;
1935: def : Pat<(loongarch_vilvl v8f32:$xj, v8f32:$xk),
1936:           (XVILVL_W v8f32:$xj, v8f32:$xk)>;
1937: def : Pat<(loongarch_vilvl v4f64:$xj, v4f64:$xk),
1938:           (XVILVL_D v4f64:$xj, v4f64:$xk)>;
1939: 
1940: // XVILVH_{B/H/W/D}
1941: def : Pat<(loongarch_vilvh v32i8:$xj, v32i8:$xk),
1942:           (XVILVH_B v32i8:$xj, v32i8:$xk)>;
1943: def : Pat<(loongarch_vilvh v16i16:$xj, v16i16:$xk),
1944:           (XVILVH_H v16i16:$xj, v16i16:$xk)>;
1945: def : Pat<(loongarch_vilvh v8i32:$xj, v8i32:$xk),
1946:           (XVILVH_W v8i32:$xj, v8i32:$xk)>;
1947: def : Pat<(loongarch_vilvh v4i64:$xj, v4i64:$xk),
1948:           (XVILVH_D v4i64:$xj, v4i64:$xk)>;
1949: def : Pat<(loongarch_vilvh v8f32:$xj, v8f32:$xk),
1950:           (XVILVH_W v8f32:$xj, v8f32:$xk)>;
1951: def : Pat<(loongarch_vilvh v4f64:$xj, v4f64:$xk),
1952:           (XVILVH_D v4f64:$xj, v4f64:$xk)>;
1953: 
1954: // XVSHUF4I_{B/H/W}
1955: def : Pat<(loongarch_vshuf4i v32i8:$xj, immZExt8:$ui8),
1956:           (XVSHUF4I_B v32i8:$xj, immZExt8:$ui8)>;
1957: def : Pat<(loongarch_vshuf4i v16i16:$xj, immZExt8:$ui8),
1958:         (XVSHUF4I_H v16i16:$xj, immZExt8:$ui8)>;
1959: def : Pat<(loongarch_vshuf4i v8i32:$xj, immZExt8:$ui8),
1960:         (XVSHUF4I_W v8i32:$xj, immZExt8:$ui8)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1961-2000 / 第 1961-2000 行
```tablegen
1961: def : Pat<(loongarch_vshuf4i v8f32:$xj, immZExt8:$ui8),
1962:         (XVSHUF4I_W v8f32:$xj, immZExt8:$ui8)>;
1963: def : Pat<(loongarch_vshuf4i_d v4i64:$xj, v4i64:$xk, immZExt8:$ui8),
1964:           (XVSHUF4I_D v4i64:$xj, v4i64:$xk, immZExt8:$ui8)>;
1965: def : Pat<(loongarch_vshuf4i_d v4f64:$xj, v4f64:$xk, immZExt8:$ui8),
1966:           (XVSHUF4I_D v4f64:$xj, v4f64:$xk, immZExt8:$ui8)>;
1967: 
1968: // XVREPL128VEI_{B/H/W/D}
1969: def : Pat<(loongarch_vreplvei v32i8:$xj, immZExt4:$ui4),
1970:           (XVREPL128VEI_B v32i8:$xj, immZExt4:$ui4)>;
1971: def : Pat<(loongarch_vreplvei v16i16:$xj, immZExt3:$ui3),
1972:         (XVREPL128VEI_H v16i16:$xj, immZExt3:$ui3)>;
1973: def : Pat<(loongarch_vreplvei v8i32:$xj, immZExt2:$ui2),
1974:         (XVREPL128VEI_W v8i32:$xj, immZExt2:$ui2)>;
1975: def : Pat<(loongarch_vreplvei v4i64:$xj, immZExt1:$ui1),
1976:         (XVREPL128VEI_D v4i64:$xj, immZExt1:$ui1)>;
1977: def : Pat<(loongarch_vreplvei v8f32:$xj, immZExt2:$ui2),
1978:         (XVREPL128VEI_W v8f32:$xj, immZExt2:$ui2)>;
1979: def : Pat<(loongarch_vreplvei v4f64:$xj, immZExt1:$ui1),
1980:         (XVREPL128VEI_D v4f64:$xj, immZExt1:$ui1)>;
1981: 
1982: // XVPERMI_{W/D}
1983: def : Pat<(loongarch_vpermi v8i32:$xj, v8i32:$xk, immZExt8:$ui8),
1984:           (XVPERMI_W v8i32:$xj, v8i32:$xk, immZExt8:$ui8)>;
1985: def : Pat<(loongarch_vpermi v8f32:$xj, v8f32:$xk, immZExt8:$ui8),
1986:           (XVPERMI_W v8f32:$xj, v8f32:$xk, immZExt8:$ui8)>;
1987: def : Pat<(loongarch_xvpermi v4i64:$xj, immZExt8: $ui8),
1988:           (XVPERMI_D v4i64:$xj, immZExt8: $ui8)>;
1989: def : Pat<(loongarch_xvpermi v4f64:$xj, immZExt8: $ui8),
1990:           (XVPERMI_D v4f64:$xj, immZExt8: $ui8)>;
1991: 
1992: // XVPERM_W
1993: def : Pat<(loongarch_xvperm v8i32:$xj, v8i32:$xk),
1994:           (XVPERM_W v8i32:$xj, v8i32:$xk)>;
1995: def : Pat<(loongarch_xvperm v8f32:$xj, v8i32:$xk),
1996:           (XVPERM_W v8f32:$xj, v8i32:$xk)>;
1997: 
1998: // XVREPLVE0_{B/H/W/D/Q}
1999: def : Pat<(loongarch_xvreplve0 v32i8:$xj),
2000:           (XVREPLVE0_B v32i8:$xj)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2001-2040 / 第 2001-2040 行
```tablegen
2001: def : Pat<(loongarch_xvreplve0 v16i16:$xj),
2002:           (XVREPLVE0_H v16i16:$xj)>;
2003: def : Pat<(loongarch_xvreplve0 v8i32:$xj),
2004:           (XVREPLVE0_W v8i32:$xj)>;
2005: def : Pat<(loongarch_xvreplve0 v4i64:$xj),
2006:           (XVREPLVE0_D v4i64:$xj)>;
2007: def : Pat<(loongarch_xvreplve0 v8f32:$xj),
2008:           (XVREPLVE0_W v8f32:$xj)>;
2009: def : Pat<(loongarch_xvreplve0 v4f64:$xj),
2010:           (XVREPLVE0_D v4f64:$xj)>;
2011: def : Pat<(lasxsplatf32 FPR32:$fj),
2012:           (XVREPLVE0_W (SUBREG_TO_REG FPR32:$fj, sub_32))>;
2013: def : Pat<(lasxsplatf64 FPR64:$fj),
2014:           (XVREPLVE0_D (SUBREG_TO_REG FPR64:$fj, sub_64))>;
2015: foreach vt = [v32i8, v16i16, v8i32, v4i64, v8f32, v4f64] in
2016:   def : Pat<(vt (loongarch_xvreplve0q LASX256:$xj)),
2017:             (XVREPLVE0_Q LASX256:$xj)>;
2018: 
2019: // VSTELM
2020: defm : VstelmPat<truncstorei8, v32i8, XVSTELM_B, simm8, uimm5>;
2021: defm : VstelmPat<truncstorei16, v16i16, XVSTELM_H, simm8_lsl1, uimm4>;
2022: defm : VstelmPat<truncstorei32, v8i32, XVSTELM_W, simm8_lsl2, uimm3>;
2023: defm : VstelmPat<store, v4i64, XVSTELM_D, simm8_lsl3, uimm2>;
2024: defm : VstelmPat<store, v8f32, XVSTELM_W, simm8_lsl2, uimm3, f32>;
2025: defm : VstelmPat<store, v4f64, XVSTELM_D, simm8_lsl3, uimm2, f64>;
2026: 
2027: // Loads/Stores
2028: foreach vt = [v32i8, v16i16, v8i32, v4i64, v8f32, v4f64] in {
2029:   defm : LdPat<load, XVLD, vt>;
2030:   def  : RegRegLdPat<load, XVLDX, vt>;
2031:   defm : StPat<store, XVST, LASX256, vt>;
2032:   def  : RegRegStPat<store, XVSTX, LASX256, vt>;
2033: }
2034: 
2035: // Bitcast float/double element extracted from vector to integer.
2036: def : Pat<(loongarch_movfr2gr_s_la64 (f32 (vector_extract v8f32:$xj, uimm3:$imm))),
2037:           (XVPICKVE2GR_W v8f32:$xj, uimm3:$imm)>;
2038: def : Pat<(i64 (bitconvert (f64 (vector_extract v4f64:$xj, uimm2:$imm)))),
2039:           (XVPICKVE2GR_D v4f64:$xj, uimm2:$imm)>;
2040: 
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2041-2080 / 第 2041-2080 行
```tablegen
2041: // Vector extraction with constant index.
2042: foreach imm = 16...31 in {
2043:   defvar Imm = !and(imm, 15);
2044:   def : Pat<(GRLenVT (vector_extract v32i8:$xj, imm)),
2045:             (VPICKVE2GR_B (EXTRACT_SUBREG (XVPERMI_D v32i8:$xj, 14), sub_128),
2046:                 Imm)>;
2047: }
2048: foreach imm = 8...15 in {
2049:   defvar Imm = !and(imm, 7);
2050:   def : Pat<(GRLenVT (vector_extract v16i16:$xj, imm)),
2051:             (VPICKVE2GR_H (EXTRACT_SUBREG (XVPERMI_D v16i16:$xj, 14), sub_128),
2052:                 Imm)>;
2053: }
2054: def : Pat<(GRLenVT (vector_extract v32i8:$xj, uimm4:$imm)),
2055:           (VPICKVE2GR_B (EXTRACT_SUBREG v32i8:$xj, sub_128), uimm4:$imm)>;
2056: def : Pat<(GRLenVT (vector_extract v16i16:$xj, uimm3:$imm)),
2057:           (VPICKVE2GR_H (EXTRACT_SUBREG v16i16:$xj, sub_128), uimm3:$imm)>;
2058: def : Pat<(GRLenVT (vector_extract v8i32:$xj, uimm3:$imm)),
2059:           (XVPICKVE2GR_W v8i32:$xj, uimm3:$imm)>;
2060: def : Pat<(i64 (vector_extract v4i64:$xj, uimm2:$imm)),
2061:           (XVPICKVE2GR_D v4i64:$xj, uimm2:$imm)>;
2062: def : Pat<(f32(vector_extract v8f32:$xj, uimm3:$imm)),
2063:           (EXTRACT_SUBREG(XVPICKVE_W v8f32:$xj, uimm3:$imm), sub_32)>;
2064: def : Pat<(f64(vector_extract v4f64:$xj, uimm2:$imm)),
2065:           (EXTRACT_SUBREG(XVPICKVE_D v4f64:$xj, uimm2:$imm), sub_64)>;
2066: 
2067: // vselect
2068: def : Pat<(v32i8 (vselect LASX256:$xd, (v32i8 (SplatPat_uimm8 uimm8:$imm)),
2069:                           LASX256:$xj)),
2070:           (XVBITSELI_B LASX256:$xd, LASX256:$xj, uimm8:$imm)>;
2071: foreach vt = [v32i8, v16i16, v8i32, v4i64, v8f32, v4f64] in
2072:   def  : Pat<(vt (vselect LASX256:$xa, LASX256:$xk, LASX256:$xj)),
2073:              (XVBITSEL_V LASX256:$xj, LASX256:$xk, LASX256:$xa)>;
2074: 
2075: // fneg
2076: def : Pat<(fneg (v8f32 LASX256:$xj)), (XVBITREVI_W LASX256:$xj, 31)>;
2077: def : Pat<(fneg (v4f64 LASX256:$xj)), (XVBITREVI_D LASX256:$xj, 63)>;
2078: 
2079: // XVFFINT_{S_W/D_L}
2080: def : Pat<(v8f32 (sint_to_fp v8i32:$vj)), (XVFFINT_S_W v8i32:$vj)>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2081-2120 / 第 2081-2120 行
```tablegen
2081: def : Pat<(v4f64 (sint_to_fp v4i64:$vj)), (XVFFINT_D_L v4i64:$vj)>;
2082: def : Pat<(v4f64 (sint_to_fp v4i32:$vj)),
2083:           (XVFFINT_D_L (VEXT2XV_D_W (SUBREG_TO_REG v4i32:$vj,
2084:                                                    sub_128)))>;
2085: def : Pat<(v4f32 (sint_to_fp v4i64:$vj)),
2086:           (EXTRACT_SUBREG (XVFCVT_S_D (XVPERMI_D (XVFFINT_D_L v4i64:$vj), 238),
2087:                                       (XVFFINT_D_L v4i64:$vj)),
2088:                           sub_128)>;
2089: 
2090: // XVFFINT_{S_WU/D_LU}
2091: def : Pat<(v8f32 (uint_to_fp v8i32:$vj)), (XVFFINT_S_WU v8i32:$vj)>;
2092: def : Pat<(v4f64 (uint_to_fp v4i64:$vj)), (XVFFINT_D_LU v4i64:$vj)>;
2093: def : Pat<(v4f64 (uint_to_fp v4i32:$vj)),
2094:           (XVFFINT_D_LU (VEXT2XV_DU_WU (SUBREG_TO_REG v4i32:$vj,
2095:                                                       sub_128)))>;
2096: def : Pat<(v4f32 (uint_to_fp v4i64:$vj)),
2097:           (EXTRACT_SUBREG (XVFCVT_S_D (XVPERMI_D (XVFFINT_D_LU v4i64:$vj), 238),
2098:                                        (XVFFINT_D_LU v4i64:$vj)),
2099:                           sub_128)>;
2100: 
2101: // XVFTINTRZ_{W_S/L_D}
2102: def : Pat<(v8i32 (fp_to_sint v8f32:$vj)), (XVFTINTRZ_W_S v8f32:$vj)>;
2103: def : Pat<(v4i64 (fp_to_sint v4f64:$vj)), (XVFTINTRZ_L_D v4f64:$vj)>;
2104: def : Pat<(v4i64(fp_to_sint v4f32:$vj)), (VEXT2XV_D_W(SUBREG_TO_REG
2105:                                              (VFTINTRZ_W_S v4f32:$vj),
2106:                                              sub_128))>;
2107: def : Pat<(v4i32(fp_to_sint v4f64:$vj)),
2108:           (EXTRACT_SUBREG(XVPICKEV_W(XVPERMI_D(XVFTINTRZ_L_D v4f64:$vj), 238),
2109:                (XVFTINTRZ_L_D v4f64:$vj)),
2110:               sub_128)>;
2111: 
2112: // XVFTINTRZ_{W_SU/L_DU}
2113: def : Pat<(v8i32 (fp_to_uint v8f32:$vj)), (XVFTINTRZ_WU_S v8f32:$vj)>;
2114: def : Pat<(v4i64 (fp_to_uint v4f64:$vj)), (XVFTINTRZ_LU_D v4f64:$vj)>;
2115: def : Pat<(v4i64(fp_to_uint v4f32:$vj)), (VEXT2XV_DU_WU(SUBREG_TO_REG
2116:                                              (VFTINTRZ_WU_S v4f32:$vj),
2117:                                              sub_128))>;
2118: def : Pat<(v4i32(fp_to_uint v4f64:$vj)),
2119:           (EXTRACT_SUBREG(XVPICKEV_W(XVPERMI_D(XVFTINTRZ_LU_D v4f64:$vj), 238),
2120:                (XVFTINTRZ_LU_D v4f64:$vj)),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2121-2160 / 第 2121-2160 行
```tablegen
2121:               sub_128)>;
2122: 
2123: // XVAVG_{B/H/W/D/BU/HU/WU/DU}, XVAVGR_{B/H/W/D/BU/HU/WU/DU}
2124: defm : VAvgPat<sra, "XVAVG_B", v32i8>;
2125: defm : VAvgPat<sra, "XVAVG_H", v16i16>;
2126: defm : VAvgPat<sra, "XVAVG_W", v8i32>;
2127: defm : VAvgPat<sra, "XVAVG_D", v4i64>;
2128: defm : VAvgPat<srl, "XVAVG_BU", v32i8>;
2129: defm : VAvgPat<srl, "XVAVG_HU", v16i16>;
2130: defm : VAvgPat<srl, "XVAVG_WU", v8i32>;
2131: defm : VAvgPat<srl, "XVAVG_DU", v4i64>;
2132: defm : VAvgrPat<sra, "XVAVGR_B", v32i8>;
2133: defm : VAvgrPat<sra, "XVAVGR_H", v16i16>;
2134: defm : VAvgrPat<sra, "XVAVGR_W", v8i32>;
2135: defm : VAvgrPat<sra, "XVAVGR_D", v4i64>;
2136: defm : VAvgrPat<srl, "XVAVGR_BU", v32i8>;
2137: defm : VAvgrPat<srl, "XVAVGR_HU", v16i16>;
2138: defm : VAvgrPat<srl, "XVAVGR_WU", v8i32>;
2139: defm : VAvgrPat<srl, "XVAVGR_DU", v4i64>;
2140: defm : PatXrXr<avgfloors, "XVAVG">;
2141: defm : PatXrXr<avgceils, "XVAVGR">;
2142: defm : PatXrXrU<avgflooru, "XVAVG">;
2143: defm : PatXrXrU<avgceilu, "XVAVGR">;
2144: 
2145: // abs
2146: def : Pat<(abs v32i8:$xj), (XVSIGNCOV_B v32i8:$xj, v32i8:$xj)>;
2147: def : Pat<(abs v16i16:$xj), (XVSIGNCOV_H v16i16:$xj, v16i16:$xj)>;
2148: def : Pat<(abs v8i32:$xj), (XVSIGNCOV_W v8i32:$xj, v8i32:$xj)>;
2149: def : Pat<(abs v4i64:$xj), (XVSIGNCOV_D v4i64:$xj, v4i64:$xj)>;
2150: 
2151: // XVABSD_{B/H/W/D}[U]
2152: defm : PatXrXr<abds, "XVABSD">;
2153: defm : PatXrXrU<abdu, "XVABSD">;
2154: 
2155: // XVADDA_{B/H/W/D}
2156: def : Pat<(add (v32i8 (abs v32i8:$xj)), (v32i8 (abs v32i8:$xk))),
2157:           (XVADDA_B v32i8:$xj, v32i8:$xk)>;
2158: def : Pat<(add (v16i16 (abs v16i16:$xj)), (v16i16 (abs v16i16:$xk))),
2159:           (XVADDA_H v16i16:$xj, v16i16:$xk)>;
2160: def : Pat<(add (v8i32 (abs v8i32:$xj)), (v8i32 (abs v8i32:$xk))),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2161-2200 / 第 2161-2200 行
```tablegen
2161:           (XVADDA_W v8i32:$xj, v8i32:$xk)>;
2162: def : Pat<(add (v4i64 (abs v4i64:$xj)), (v4i64 (abs v4i64:$xk))),
2163:           (XVADDA_D v4i64:$xj, v4i64:$xk)>;
2164: 
2165: // XVSADD_{B/H/W/D}[U], XVSSUB_{B/H/W/D}[U]
2166: defm : PatXrXr<saddsat, "XVSADD">;
2167: defm : PatXrXr<ssubsat, "XVSSUB">;
2168: defm : PatXrXrU<uaddsat, "XVSADD">;
2169: defm : PatXrXrU<usubsat, "XVSSUB">;
2170: 
2171: // Vector mask set by condition
2172: def : Pat<(loongarch_xvmskltz (v32i8 LASX256:$vj)), (PseudoXVMSKLTZ_B LASX256:$vj)>;
2173: def : Pat<(loongarch_xvmskltz (v16i16 LASX256:$vj)), (PseudoXVMSKLTZ_H LASX256:$vj)>;
2174: def : Pat<(loongarch_xvmskltz (v8i32 LASX256:$vj)), (PseudoXVMSKLTZ_W LASX256:$vj)>;
2175: def : Pat<(loongarch_xvmskltz (v4i64 LASX256:$vj)), (PseudoXVMSKLTZ_D LASX256:$vj)>;
2176: def : Pat<(loongarch_xvmskgez (v32i8 LASX256:$vj)), (PseudoXVMSKGEZ_B LASX256:$vj)>;
2177: def : Pat<(loongarch_xvmskeqz (v32i8 LASX256:$vj)), (PseudoXVMSKEQZ_B LASX256:$vj)>;
2178: def : Pat<(loongarch_xvmsknez (v32i8 LASX256:$vj)), (PseudoXVMSKNEZ_B LASX256:$vj)>;
2179: 
2180: // Subvector tricks
2181: // Patterns for insert_subvector/extract_subvector
2182: multiclass subvector_subreg_lowering<RegisterClass subRC, ValueType subVT,
2183:                                      RegisterClass RC, ValueType VT,
2184:                                      int hiIdx, SubRegIndex subIdx> {
2185:   // A 128-bit subvector extract from the first 256-bit vector position is a
2186:   // subregister copy that needs no instruction. Likewise, a 128-bit subvector
2187:   // insert to the first 256-bit vector position is a subregister copy that needs
2188:   // no instruction.
2189:   def : Pat<(subVT (extract_subvector (VT RC:$src), (iPTR 0))),
2190:             (subVT (EXTRACT_SUBREG RC:$src, subIdx))>;
2191:   def : Pat<(VT (insert_subvector undef_or_freeze_undef, subRC:$src, (iPTR 0))),
2192:             (VT (INSERT_SUBREG (IMPLICIT_DEF), subRC:$src, subIdx))>;
2193: 
2194:   def : Pat<(subVT (extract_subvector (VT RC:$src), (iPTR hiIdx))),
2195:             (subVT (EXTRACT_SUBREG (XVPERMI_Q (IMPLICIT_DEF), RC:$src, 1), subIdx))>;
2196:   def : Pat<(VT (insert_subvector RC:$vd, subRC:$vj, (iPTR 0))),
2197:             (VT (XVPERMI_Q RC:$vd, (INSERT_SUBREG (IMPLICIT_DEF), subRC:$vj, subIdx), 48))>;
2198:   def : Pat<(VT (insert_subvector RC:$vd, subRC:$vj, (iPTR hiIdx))),
2199:             (VT (XVPERMI_Q RC:$vd, (INSERT_SUBREG (IMPLICIT_DEF), subRC:$vj, subIdx), 2))>;
2200: }
```
- **EN**: This block declares or refines TableGen records such as `subvector_subreg_lowering`.
- **CN**: 该代码块声明或细化了 `subvector_subreg_lowering` 等 TableGen 记录。

### Lines 2201-2240 / 第 2201-2240 行
```tablegen
2201: 
2202: defm : subvector_subreg_lowering<LSX128, v4i32, LASX256, v8i32,  4,  sub_128>;
2203: defm : subvector_subreg_lowering<LSX128, v4f32, LASX256, v8f32,  4,  sub_128>;
2204: defm : subvector_subreg_lowering<LSX128, v2i64, LASX256, v4i64,  2,  sub_128>;
2205: defm : subvector_subreg_lowering<LSX128, v2f64, LASX256, v4f64,  2,  sub_128>;
2206: defm : subvector_subreg_lowering<LSX128, v8i16, LASX256, v16i16, 8,  sub_128>;
2207: defm : subvector_subreg_lowering<LSX128, v16i8, LASX256, v32i8,  16, sub_128>;
2208: 
2209: // LASX and LSX conversion
2210: def : Pat<(int_loongarch_lasx_cast_128_s (v4f32 LSX128:$src)),
2211:           (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$src, sub_128)>;
2212: def : Pat<(int_loongarch_lasx_cast_128_d (v2f64 LSX128:$src)),
2213:           (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$src, sub_128)>;
2214: def : Pat<(int_loongarch_lasx_cast_128 (v2i64 LSX128:$src)),
2215:           (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$src, sub_128)>;
2216: def : Pat<(int_loongarch_lasx_extract_128_lo_s (v8f32 LASX256:$src)),
2217:           (EXTRACT_SUBREG LASX256:$src, sub_128)>;
2218: def : Pat<(int_loongarch_lasx_extract_128_lo_d (v4f64 LASX256:$src)),
2219:           (EXTRACT_SUBREG LASX256:$src, sub_128)>;
2220: def : Pat<(int_loongarch_lasx_extract_128_lo (v4i64 LASX256:$src)),
2221:           (EXTRACT_SUBREG LASX256:$src, sub_128)>;
2222: def : Pat<(int_loongarch_lasx_extract_128_hi_s (v8f32 LASX256:$src)),
2223:           (EXTRACT_SUBREG (XVPERMI_Q (IMPLICIT_DEF), LASX256:$src, 1), sub_128)>;
2224: def : Pat<(int_loongarch_lasx_extract_128_hi_d (v4f64 LASX256:$src)),
2225:           (EXTRACT_SUBREG (XVPERMI_Q (IMPLICIT_DEF), LASX256:$src, 1), sub_128)>;
2226: def : Pat<(int_loongarch_lasx_extract_128_hi (v4i64 LASX256:$src)),
2227:           (EXTRACT_SUBREG (XVPERMI_Q (IMPLICIT_DEF), LASX256:$src, 1), sub_128)>;
2228: def : Pat<(int_loongarch_lasx_insert_128_lo_s (v8f32 LASX256:$src), (v4f32 LSX128:$lo)),
2229:           (XVPERMI_Q LASX256:$src, (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$lo, sub_128), 48)>;
2230: def : Pat<(int_loongarch_lasx_insert_128_lo_d (v4f64 LASX256:$src), (v2f64 LSX128:$lo)),
2231:           (XVPERMI_Q LASX256:$src, (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$lo, sub_128), 48)>;
2232: def : Pat<(int_loongarch_lasx_insert_128_lo (v4i64 LASX256:$src), (v2i64 LSX128:$lo)),
2233:           (XVPERMI_Q LASX256:$src, (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$lo, sub_128), 48)>;
2234: def : Pat<(int_loongarch_lasx_insert_128_hi_s (v8f32 LASX256:$src), (v4f32 LSX128:$lo)),
2235:           (XVPERMI_Q LASX256:$src, (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$lo, sub_128), 2)>;
2236: def : Pat<(int_loongarch_lasx_insert_128_hi_d (v4f64 LASX256:$src), (v2f64 LSX128:$lo)),
2237:           (XVPERMI_Q LASX256:$src, (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$lo, sub_128), 2)>;
2238: def : Pat<(int_loongarch_lasx_insert_128_hi (v4i64 LASX256:$src), (v2i64 LSX128:$lo)),
2239:           (XVPERMI_Q LASX256:$src, (INSERT_SUBREG (IMPLICIT_DEF), LSX128:$lo, sub_128), 2)>;
2240: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2241-2280 / 第 2241-2280 行
```tablegen
2241: // Sign extensions
2242: def : Pat<(v4i64 (sext v4i32:$vj)),
2243:           (v4i64 (VEXT2XV_D_W (SUBREG_TO_REG v4i32:$vj, sub_128)))>;
2244: def : Pat<(v8i32 (sext v8i16:$vj)),
2245:           (v8i32 (VEXT2XV_W_H (SUBREG_TO_REG v8i16:$vj, sub_128)))>;
2246: def : Pat<(v16i16 (sext v16i8:$vj)),
2247:           (v16i16 (VEXT2XV_H_B (SUBREG_TO_REG v16i8:$vj, sub_128)))>;
2248: 
2249: def : Pat<(v2i64 (sext_invec v16i8:$vj)),
2250:           (v2i64 (EXTRACT_SUBREG (VEXT2XV_D_B (SUBREG_TO_REG v16i8:$vj, sub_128)),
2251:                                  sub_128))>;
2252: def : Pat<(v2i64 (sext_invec v8i16:$vj)),
2253:           (v2i64 (EXTRACT_SUBREG (VEXT2XV_D_H (SUBREG_TO_REG v8i16:$vj, sub_128)),
2254:                                  sub_128))>;
2255: def : Pat<(v2i64 (sext_invec v4i32:$vj)),
2256:           (v2i64 (EXTRACT_SUBREG (VEXT2XV_D_W (SUBREG_TO_REG v4i32:$vj, sub_128)),
2257:                                  sub_128))>;
2258: def : Pat<(v4i32 (sext_invec v16i8:$vj)),
2259:           (v4i32 (EXTRACT_SUBREG (VEXT2XV_W_B (SUBREG_TO_REG v16i8:$vj, sub_128)),
2260:                                  sub_128))>;
2261: def : Pat<(v4i32 (sext_invec v8i16:$vj)),
2262:           (v4i32 (EXTRACT_SUBREG (VEXT2XV_W_H (SUBREG_TO_REG v8i16:$vj, sub_128)),
2263:                                  sub_128))>;
2264: def : Pat<(v4i64 (sext_invec v32i8:$xj)), (v4i64 (VEXT2XV_D_B v32i8:$xj))>;
2265: def : Pat<(v4i64 (sext_invec v16i16:$xj)), (v4i64 (VEXT2XV_D_H v16i16:$xj))>;
2266: def : Pat<(v4i64 (sext_invec v8i32:$xj)), (v4i64 (VEXT2XV_D_W v8i32:$xj))>;
2267: def : Pat<(v8i16 (sext_invec v16i8:$vj)),
2268:           (v8i16 (EXTRACT_SUBREG (VEXT2XV_H_B (SUBREG_TO_REG v16i8:$vj, sub_128)),
2269:                                  sub_128))>;
2270: def : Pat<(v16i16 (sext_invec v32i8:$xj)), (v16i16 (VEXT2XV_H_B v32i8:$xj))>;
2271: def : Pat<(v8i32 (sext_invec v32i8:$xj)), (v8i32 (VEXT2XV_W_B v32i8:$xj))>;
2272: def : Pat<(v8i32 (sext_invec v16i16:$xj)), (v8i32 (VEXT2XV_W_H v16i16:$xj))>;
2273: def : Pat<(v8i32 (sext_invec v16i8:$vj)),
2274:           (v8i32 (VEXT2XV_W_B (SUBREG_TO_REG v16i8:$vj, sub_128)))>;
2275: 
2276: // Zero extensions
2277: def : Pat<(v4i64 (zext v4i32:$vj)),
2278:           (v4i64 (VEXT2XV_DU_WU (SUBREG_TO_REG v4i32:$vj, sub_128)))>;
2279: def : Pat<(v8i32 (zext v8i16:$vj)),
2280:           (v8i32 (VEXT2XV_WU_HU (SUBREG_TO_REG v8i16:$vj, sub_128)))>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2281-2320 / 第 2281-2320 行
```tablegen
2281: def : Pat<(v16i16 (zext v16i8:$vj)),
2282:           (v16i16 (VEXT2XV_HU_BU (SUBREG_TO_REG v16i8:$vj, sub_128)))>;
2283: 
2284: def : Pat<(v2i64 (zext_invec v16i8:$vj)),
2285:           (v2i64 (EXTRACT_SUBREG (VEXT2XV_DU_BU (SUBREG_TO_REG v16i8:$vj, sub_128)),
2286:                                  sub_128))>;
2287: def : Pat<(v2i64 (zext_invec v8i16:$vj)),
2288:           (v2i64 (EXTRACT_SUBREG (VEXT2XV_DU_HU (SUBREG_TO_REG v8i16:$vj, sub_128)),
2289:                                  sub_128))>;
2290: def : Pat<(v2i64 (zext_invec v4i32:$vj)),
2291:           (v2i64 (EXTRACT_SUBREG (VEXT2XV_DU_WU (SUBREG_TO_REG v4i32:$vj, sub_128)),
2292:                                  sub_128))>;
2293: def : Pat<(v4i32 (zext_invec v16i8:$vj)),
2294:           (v4i32 (EXTRACT_SUBREG (VEXT2XV_WU_BU (SUBREG_TO_REG v16i8:$vj, sub_128)),
2295:                                  sub_128))>;
2296: def : Pat<(v4i32 (zext_invec v8i16:$vj)),
2297:           (v4i32 (EXTRACT_SUBREG (VEXT2XV_WU_HU (SUBREG_TO_REG v8i16:$vj, sub_128)),
2298:                                  sub_128))>;
2299: def : Pat<(v4i64 (zext_invec v32i8:$xj)), (v4i64 (VEXT2XV_DU_BU v32i8:$xj))>;
2300: def : Pat<(v4i64 (zext_invec v16i16:$xj)), (v4i64 (VEXT2XV_DU_HU v16i16:$xj))>;
2301: def : Pat<(v4i64 (zext_invec v8i32:$xj)), (v4i64 (VEXT2XV_DU_WU v8i32:$xj))>;
2302: def : Pat<(v8i16 (zext_invec v16i8:$vj)),
2303:           (v8i16 (EXTRACT_SUBREG (VEXT2XV_HU_BU (SUBREG_TO_REG v16i8:$vj, sub_128)),
2304:                                  sub_128))>;
2305: def : Pat<(v16i16 (zext_invec v32i8:$xj)), (v16i16 (VEXT2XV_HU_BU v32i8:$xj))>;
2306: def : Pat<(v8i32 (zext_invec v32i8:$xj)), (v8i32 (VEXT2XV_WU_BU v32i8:$xj))>;
2307: def : Pat<(v8i32 (zext_invec v16i16:$xj)), (v8i32 (VEXT2XV_WU_HU v16i16:$xj))>;
2308: def : Pat<(v8i32 (zext_invec v16i8:$vj)),
2309:           (v8i32 (VEXT2XV_WU_BU (SUBREG_TO_REG v16i8:$vj, sub_128)))>;
2310: 
2311: } // Predicates = [HasExtLASX]
2312: 
2313: /// Intrinsic pattern
2314: 
2315: class deriveLASXIntrinsic<string Inst> {
2316:   Intrinsic ret = !cast<Intrinsic>(!tolower("int_loongarch_lasx_"#Inst));
2317: }
2318: 
2319: let Predicates = [HasExtLASX] in {
2320: 
```
- **EN**: This block declares or refines TableGen records such as `deriveLASXIntrinsic`.
- **CN**: 该代码块声明或细化了 `deriveLASXIntrinsic` 等 TableGen 记录。

### Lines 2321-2360 / 第 2321-2360 行
```tablegen
2321: // vty: v32i8/v16i16/v8i32/v4i64
2322: // Pat<(Intrinsic vty:$xj, vty:$xk),
2323: //     (LAInst vty:$xj, vty:$xk)>;
2324: foreach Inst = ["XVSADD_B", "XVSADD_BU", "XVSSUB_B", "XVSSUB_BU",
2325:                 "XVHADDW_H_B", "XVHADDW_HU_BU", "XVHSUBW_H_B", "XVHSUBW_HU_BU",
2326:                 "XVADDWEV_H_B", "XVADDWOD_H_B", "XVSUBWEV_H_B", "XVSUBWOD_H_B",
2327:                 "XVADDWEV_H_BU", "XVADDWOD_H_BU", "XVSUBWEV_H_BU", "XVSUBWOD_H_BU",
2328:                 "XVADDWEV_H_BU_B", "XVADDWOD_H_BU_B",
2329:                 "XVAVG_B", "XVAVG_BU", "XVAVGR_B", "XVAVGR_BU",
2330:                 "XVABSD_B", "XVABSD_BU", "XVADDA_B", "XVMUH_B", "XVMUH_BU",
2331:                 "XVMULWEV_H_B", "XVMULWOD_H_B", "XVMULWEV_H_BU", "XVMULWOD_H_BU",
2332:                 "XVMULWEV_H_BU_B", "XVMULWOD_H_BU_B", "XVSIGNCOV_B",
2333:                 "XVANDN_V", "XVORN_V", "XVROTR_B", "XVSRLR_B", "XVSRAR_B",
2334:                 "XVSEQ_B", "XVSLE_B", "XVSLE_BU", "XVSLT_B", "XVSLT_BU",
2335:                 "XVPACKEV_B", "XVPACKOD_B", "XVPICKEV_B", "XVPICKOD_B",
2336:                 "XVILVL_B", "XVILVH_B"] in
2337:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2338:                (v32i8 LASX256:$xj), (v32i8 LASX256:$xk)),
2339:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk)>;
2340: foreach Inst = ["XVSADD_H", "XVSADD_HU", "XVSSUB_H", "XVSSUB_HU",
2341:                 "XVHADDW_W_H", "XVHADDW_WU_HU", "XVHSUBW_W_H", "XVHSUBW_WU_HU",
2342:                 "XVADDWEV_W_H", "XVADDWOD_W_H", "XVSUBWEV_W_H", "XVSUBWOD_W_H",
2343:                 "XVADDWEV_W_HU", "XVADDWOD_W_HU", "XVSUBWEV_W_HU", "XVSUBWOD_W_HU",
2344:                 "XVADDWEV_W_HU_H", "XVADDWOD_W_HU_H",
2345:                 "XVAVG_H", "XVAVG_HU", "XVAVGR_H", "XVAVGR_HU",
2346:                 "XVABSD_H", "XVABSD_HU", "XVADDA_H", "XVMUH_H", "XVMUH_HU",
2347:                 "XVMULWEV_W_H", "XVMULWOD_W_H", "XVMULWEV_W_HU", "XVMULWOD_W_HU",
2348:                 "XVMULWEV_W_HU_H", "XVMULWOD_W_HU_H", "XVSIGNCOV_H", "XVROTR_H",
2349:                 "XVSRLR_H", "XVSRAR_H", "XVSRLN_B_H", "XVSRAN_B_H", "XVSRLRN_B_H",
2350:                 "XVSRARN_B_H", "XVSSRLN_B_H", "XVSSRAN_B_H", "XVSSRLN_BU_H",
2351:                 "XVSSRAN_BU_H", "XVSSRLRN_B_H", "XVSSRARN_B_H", "XVSSRLRN_BU_H",
2352:                 "XVSSRARN_BU_H",
2353:                 "XVSEQ_H", "XVSLE_H", "XVSLE_HU", "XVSLT_H", "XVSLT_HU",
2354:                 "XVPACKEV_H", "XVPACKOD_H", "XVPICKEV_H", "XVPICKOD_H",
2355:                 "XVILVL_H", "XVILVH_H"] in
2356:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2357:                (v16i16 LASX256:$xj), (v16i16 LASX256:$xk)),
2358:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk)>;
2359: foreach Inst = ["XVSADD_W", "XVSADD_WU", "XVSSUB_W", "XVSSUB_WU",
2360:                 "XVHADDW_D_W", "XVHADDW_DU_WU", "XVHSUBW_D_W", "XVHSUBW_DU_WU",
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2361-2400 / 第 2361-2400 行
```tablegen
2361:                 "XVADDWEV_D_W", "XVADDWOD_D_W", "XVSUBWEV_D_W", "XVSUBWOD_D_W",
2362:                 "XVADDWEV_D_WU", "XVADDWOD_D_WU", "XVSUBWEV_D_WU", "XVSUBWOD_D_WU",
2363:                 "XVADDWEV_D_WU_W", "XVADDWOD_D_WU_W",
2364:                 "XVAVG_W", "XVAVG_WU", "XVAVGR_W", "XVAVGR_WU",
2365:                 "XVABSD_W", "XVABSD_WU", "XVADDA_W", "XVMUH_W", "XVMUH_WU",
2366:                 "XVMULWEV_D_W", "XVMULWOD_D_W", "XVMULWEV_D_WU", "XVMULWOD_D_WU",
2367:                 "XVMULWEV_D_WU_W", "XVMULWOD_D_WU_W", "XVSIGNCOV_W", "XVROTR_W",
2368:                 "XVSRLR_W", "XVSRAR_W", "XVSRLN_H_W", "XVSRAN_H_W", "XVSRLRN_H_W",
2369:                 "XVSRARN_H_W", "XVSSRLN_H_W", "XVSSRAN_H_W", "XVSSRLN_HU_W",
2370:                 "XVSSRAN_HU_W", "XVSSRLRN_H_W", "XVSSRARN_H_W", "XVSSRLRN_HU_W",
2371:                 "XVSSRARN_HU_W",
2372:                 "XVSEQ_W", "XVSLE_W", "XVSLE_WU", "XVSLT_W", "XVSLT_WU",
2373:                 "XVPACKEV_W", "XVPACKOD_W", "XVPICKEV_W", "XVPICKOD_W",
2374:                 "XVILVL_W", "XVILVH_W", "XVPERM_W"] in
2375:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2376:                (v8i32 LASX256:$xj), (v8i32 LASX256:$xk)),
2377:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk)>;
2378: foreach Inst = ["XVADD_Q", "XVSUB_Q",
2379:                 "XVSADD_D", "XVSADD_DU", "XVSSUB_D", "XVSSUB_DU",
2380:                 "XVHADDW_Q_D", "XVHADDW_QU_DU", "XVHSUBW_Q_D", "XVHSUBW_QU_DU",
2381:                 "XVADDWEV_Q_D", "XVADDWOD_Q_D", "XVSUBWEV_Q_D", "XVSUBWOD_Q_D",
2382:                 "XVADDWEV_Q_DU", "XVADDWOD_Q_DU", "XVSUBWEV_Q_DU", "XVSUBWOD_Q_DU",
2383:                 "XVADDWEV_Q_DU_D", "XVADDWOD_Q_DU_D",
2384:                 "XVAVG_D", "XVAVG_DU", "XVAVGR_D", "XVAVGR_DU",
2385:                 "XVABSD_D", "XVABSD_DU", "XVADDA_D", "XVMUH_D", "XVMUH_DU",
2386:                 "XVMULWEV_Q_D", "XVMULWOD_Q_D", "XVMULWEV_Q_DU", "XVMULWOD_Q_DU",
2387:                 "XVMULWEV_Q_DU_D", "XVMULWOD_Q_DU_D", "XVSIGNCOV_D", "XVROTR_D",
2388:                 "XVSRLR_D", "XVSRAR_D", "XVSRLN_W_D", "XVSRAN_W_D", "XVSRLRN_W_D",
2389:                 "XVSRARN_W_D", "XVSSRLN_W_D", "XVSSRAN_W_D", "XVSSRLN_WU_D",
2390:                 "XVSSRAN_WU_D", "XVSSRLRN_W_D", "XVSSRARN_W_D", "XVSSRLRN_WU_D",
2391:                 "XVSSRARN_WU_D", "XVFFINT_S_L",
2392:                 "XVSEQ_D", "XVSLE_D", "XVSLE_DU", "XVSLT_D", "XVSLT_DU",
2393:                 "XVPACKEV_D", "XVPACKOD_D", "XVPICKEV_D", "XVPICKOD_D",
2394:                 "XVILVL_D", "XVILVH_D"] in
2395:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2396:                (v4i64 LASX256:$xj), (v4i64 LASX256:$xk)),
2397:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk)>;
2398: 
2399: // vty: v32i8/v16i16/v8i32/v4i64
2400: // Pat<(Intrinsic vty:$xd, vty:$xj, vty:$xk),
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2401-2440 / 第 2401-2440 行
```tablegen
2401: //     (LAInst vty:$xd, vty:$xj, vty:$xk)>;
2402: foreach Inst = ["XVMADDWEV_H_B", "XVMADDWOD_H_B", "XVMADDWEV_H_BU",
2403:                 "XVMADDWOD_H_BU", "XVMADDWEV_H_BU_B", "XVMADDWOD_H_BU_B"] in
2404:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2405:                (v16i16 LASX256:$xd), (v32i8 LASX256:$xj), (v32i8 LASX256:$xk)),
2406:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2407: foreach Inst = ["XVMADDWEV_W_H", "XVMADDWOD_W_H", "XVMADDWEV_W_HU",
2408:                 "XVMADDWOD_W_HU", "XVMADDWEV_W_HU_H", "XVMADDWOD_W_HU_H"] in
2409:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2410:                (v8i32 LASX256:$xd), (v16i16 LASX256:$xj), (v16i16 LASX256:$xk)),
2411:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2412: foreach Inst = ["XVMADDWEV_D_W", "XVMADDWOD_D_W", "XVMADDWEV_D_WU",
2413:                 "XVMADDWOD_D_WU", "XVMADDWEV_D_WU_W", "XVMADDWOD_D_WU_W"] in
2414:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2415:                (v4i64 LASX256:$xd), (v8i32 LASX256:$xj), (v8i32 LASX256:$xk)),
2416:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2417: foreach Inst = ["XVMADDWEV_Q_D", "XVMADDWOD_Q_D", "XVMADDWEV_Q_DU",
2418:                 "XVMADDWOD_Q_DU", "XVMADDWEV_Q_DU_D", "XVMADDWOD_Q_DU_D"] in
2419:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2420:                (v4i64 LASX256:$xd), (v4i64 LASX256:$xj), (v4i64 LASX256:$xk)),
2421:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2422: 
2423: // vty: v32i8/v16i16/v8i32/v4i64
2424: // Pat<(Intrinsic vty:$xj),
2425: //     (LAInst vty:$xj)>;
2426: foreach Inst = ["XVEXTH_H_B", "XVEXTH_HU_BU",
2427:                 "XVMSKLTZ_B", "XVMSKGEZ_B", "XVMSKNZ_B",
2428:                 "XVCLO_B", "VEXT2XV_H_B", "VEXT2XV_HU_BU",
2429:                 "VEXT2XV_W_B", "VEXT2XV_WU_BU", "VEXT2XV_D_B",
2430:                 "VEXT2XV_DU_BU", "XVREPLVE0_B", "XVREPLVE0_Q"] in
2431:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v32i8 LASX256:$xj)),
2432:             (!cast<LAInst>(Inst) LASX256:$xj)>;
2433: foreach Inst = ["XVEXTH_W_H", "XVEXTH_WU_HU", "XVMSKLTZ_H",
2434:                 "XVCLO_H", "XVFCVTL_S_H", "XVFCVTH_S_H",
2435:                 "VEXT2XV_W_H", "VEXT2XV_WU_HU", "VEXT2XV_D_H",
2436:                 "VEXT2XV_DU_HU", "XVREPLVE0_H"] in
2437:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v16i16 LASX256:$xj)),
2438:             (!cast<LAInst>(Inst) LASX256:$xj)>;
2439: foreach Inst = ["XVEXTH_D_W", "XVEXTH_DU_WU", "XVMSKLTZ_W",
2440:                 "XVCLO_W", "XVFFINT_S_W", "XVFFINT_S_WU",
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2441-2480 / 第 2441-2480 行
```tablegen
2441:                 "XVFFINTL_D_W", "XVFFINTH_D_W",
2442:                 "VEXT2XV_D_W", "VEXT2XV_DU_WU", "XVREPLVE0_W"] in
2443:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v8i32 LASX256:$xj)),
2444:             (!cast<LAInst>(Inst) LASX256:$xj)>;
2445: foreach Inst = ["XVEXTH_Q_D", "XVEXTH_QU_DU", "XVMSKLTZ_D",
2446:                 "XVEXTL_Q_D", "XVEXTL_QU_DU",
2447:                 "XVCLO_D", "XVFFINT_D_L", "XVFFINT_D_LU",
2448:                 "XVREPLVE0_D"] in
2449:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v4i64 LASX256:$xj)),
2450:             (!cast<LAInst>(Inst) LASX256:$xj)>;
2451: 
2452: // Pat<(Intrinsic timm:$imm)
2453: //     (LAInst timm:$imm)>;
2454: def : Pat<(int_loongarch_lasx_xvldi timm:$imm),
2455:           (XVLDI (to_valid_timm timm:$imm))>;
2456: foreach Inst = ["XVREPLI_B", "XVREPLI_H", "XVREPLI_W", "XVREPLI_D"] in
2457:   def : Pat<(deriveLASXIntrinsic<Inst>.ret timm:$imm),
2458:             (!cast<LAInst>("Pseudo"#Inst) (to_valid_timm timm:$imm))>;
2459: 
2460: // vty: v32i8/v16i16/v8i32/v4i64
2461: // Pat<(Intrinsic vty:$xj, timm:$imm)
2462: //     (LAInst vty:$xj, timm:$imm)>;
2463: foreach Inst = ["XVSAT_B", "XVSAT_BU", "XVNORI_B", "XVROTRI_B", "XVSLLWIL_H_B",
2464:                 "XVSLLWIL_HU_BU", "XVSRLRI_B", "XVSRARI_B",
2465:                 "XVSEQI_B", "XVSLEI_B", "XVSLEI_BU", "XVSLTI_B", "XVSLTI_BU",
2466:                 "XVREPL128VEI_B", "XVBSLL_V", "XVBSRL_V", "XVSHUF4I_B"] in
2467:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v32i8 LASX256:$xj), timm:$imm),
2468:             (!cast<LAInst>(Inst) LASX256:$xj, (to_valid_timm timm:$imm))>;
2469: foreach Inst = ["XVSAT_H", "XVSAT_HU", "XVROTRI_H", "XVSLLWIL_W_H",
2470:                 "XVSLLWIL_WU_HU", "XVSRLRI_H", "XVSRARI_H",
2471:                 "XVSEQI_H", "XVSLEI_H", "XVSLEI_HU", "XVSLTI_H", "XVSLTI_HU",
2472:                 "XVREPL128VEI_H", "XVSHUF4I_H"] in
2473:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v16i16 LASX256:$xj), timm:$imm),
2474:             (!cast<LAInst>(Inst) LASX256:$xj, (to_valid_timm timm:$imm))>;
2475: foreach Inst = ["XVSAT_W", "XVSAT_WU", "XVROTRI_W", "XVSLLWIL_D_W",
2476:                 "XVSLLWIL_DU_WU", "XVSRLRI_W", "XVSRARI_W",
2477:                 "XVSEQI_W", "XVSLEI_W", "XVSLEI_WU", "XVSLTI_W", "XVSLTI_WU",
2478:                 "XVREPL128VEI_W", "XVSHUF4I_W", "XVPICKVE_W"] in
2479:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v8i32 LASX256:$xj), timm:$imm),
2480:             (!cast<LAInst>(Inst) LASX256:$xj, (to_valid_timm timm:$imm))>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2481-2520 / 第 2481-2520 行
```tablegen
2481: foreach Inst = ["XVSAT_D", "XVSAT_DU", "XVROTRI_D", "XVSRLRI_D", "XVSRARI_D",
2482:                 "XVSEQI_D", "XVSLEI_D", "XVSLEI_DU", "XVSLTI_D", "XVSLTI_DU",
2483:                 "XVPICKVE2GR_D", "XVPICKVE2GR_DU",
2484:                 "XVREPL128VEI_D", "XVPERMI_D", "XVPICKVE_D"] in
2485:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v4i64 LASX256:$xj), timm:$imm),
2486:             (!cast<LAInst>(Inst) LASX256:$xj, (to_valid_timm timm:$imm))>;
2487: 
2488: // vty: v32i8/v16i16/v8i32/v4i64
2489: // Pat<(Intrinsic vty:$xd, vty:$xj, timm:$imm)
2490: //     (LAInst vty:$xd, vty:$xj, timm:$imm)>;
2491: foreach Inst = ["XVSRLNI_B_H", "XVSRANI_B_H", "XVSRLRNI_B_H", "XVSRARNI_B_H",
2492:                 "XVSSRLNI_B_H", "XVSSRANI_B_H", "XVSSRLNI_BU_H", "XVSSRANI_BU_H",
2493:                 "XVSSRLRNI_B_H", "XVSSRARNI_B_H", "XVSSRLRNI_BU_H", "XVSSRARNI_BU_H",
2494:                 "XVFRSTPI_B", "XVBITSELI_B", "XVEXTRINS_B", "XVPERMI_Q"] in
2495:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2496:                (v32i8 LASX256:$xd), (v32i8 LASX256:$xj), timm:$imm),
2497:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj,
2498:                (to_valid_timm timm:$imm))>;
2499: foreach Inst = ["XVSRLNI_H_W", "XVSRANI_H_W", "XVSRLRNI_H_W", "XVSRARNI_H_W",
2500:                 "XVSSRLNI_H_W", "XVSSRANI_H_W", "XVSSRLNI_HU_W", "XVSSRANI_HU_W",
2501:                 "XVSSRLRNI_H_W", "XVSSRARNI_H_W", "XVSSRLRNI_HU_W", "XVSSRARNI_HU_W",
2502:                 "XVFRSTPI_H", "XVEXTRINS_H"] in
2503:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2504:                (v16i16 LASX256:$xd), (v16i16 LASX256:$xj), timm:$imm),
2505:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj,
2506:                (to_valid_timm timm:$imm))>;
2507: foreach Inst = ["XVSRLNI_W_D", "XVSRANI_W_D", "XVSRLRNI_W_D", "XVSRARNI_W_D",
2508:                 "XVSSRLNI_W_D", "XVSSRANI_W_D", "XVSSRLNI_WU_D", "XVSSRANI_WU_D",
2509:                 "XVSSRLRNI_W_D", "XVSSRARNI_W_D", "XVSSRLRNI_WU_D", "XVSSRARNI_WU_D",
2510:                 "XVPERMI_W", "XVEXTRINS_W", "XVINSVE0_W"] in
2511:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2512:                (v8i32 LASX256:$xd), (v8i32 LASX256:$xj), timm:$imm),
2513:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj,
2514:                (to_valid_timm timm:$imm))>;
2515: foreach Inst = ["XVSRLNI_D_Q", "XVSRANI_D_Q", "XVSRLRNI_D_Q", "XVSRARNI_D_Q",
2516:                 "XVSSRLNI_D_Q", "XVSSRANI_D_Q", "XVSSRLNI_DU_Q", "XVSSRANI_DU_Q",
2517:                 "XVSSRLRNI_D_Q", "XVSSRARNI_D_Q", "XVSSRLRNI_DU_Q", "XVSSRARNI_DU_Q",
2518:                 "XVSHUF4I_D", "XVEXTRINS_D", "XVINSVE0_D"] in
2519:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2520:                (v4i64 LASX256:$xd), (v4i64 LASX256:$xj), timm:$imm),
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2521-2560 / 第 2521-2560 行
```tablegen
2521:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj,
2522:                (to_valid_timm timm:$imm))>;
2523: 
2524: // vty: v32i8/v16i16/v8i32/v4i64
2525: // Pat<(Intrinsic vty:$xd, vty:$xj, vty:$xk),
2526: //     (LAInst vty:$xd, vty:$xj, vty:$xk)>;
2527: foreach Inst = ["XVFRSTP_B", "XVBITSEL_V", "XVSHUF_B"] in
2528:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2529:                (v32i8 LASX256:$xd), (v32i8 LASX256:$xj), (v32i8 LASX256:$xk)),
2530:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2531: foreach Inst = ["XVFRSTP_H", "XVSHUF_H"] in
2532:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2533:                (v16i16 LASX256:$xd), (v16i16 LASX256:$xj), (v16i16 LASX256:$xk)),
2534:             (!cast<LAInst>(Inst) LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2535: def : Pat<(int_loongarch_lasx_xvshuf_w (v8i32 LASX256:$xd), (v8i32 LASX256:$xj),
2536:                                      (v8i32 LASX256:$xk)),
2537:           (XVSHUF_W LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2538: def : Pat<(int_loongarch_lasx_xvshuf_d (v4i64 LASX256:$xd), (v4i64 LASX256:$xj),
2539:                                      (v4i64 LASX256:$xk)),
2540:           (XVSHUF_D LASX256:$xd, LASX256:$xj, LASX256:$xk)>;
2541: 
2542: // vty: v8f32/v4f64
2543: // Pat<(Intrinsic vty:$xj, vty:$xk, vty:$xa),
2544: //     (LAInst vty:$xj, vty:$xk, vty:$xa)>;
2545: foreach Inst = ["XVFMSUB_S", "XVFNMADD_S", "XVFNMSUB_S"] in
2546:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2547:                (v8f32 LASX256:$xj), (v8f32 LASX256:$xk), (v8f32 LASX256:$xa)),
2548:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk, LASX256:$xa)>;
2549: foreach Inst = ["XVFMSUB_D", "XVFNMADD_D", "XVFNMSUB_D"] in
2550:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2551:                (v4f64 LASX256:$xj), (v4f64 LASX256:$xk), (v4f64 LASX256:$xa)),
2552:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk, LASX256:$xa)>;
2553: 
2554: // vty: v8f32/v4f64
2555: // Pat<(Intrinsic vty:$xj, vty:$xk),
2556: //     (LAInst vty:$xj, vty:$xk)>;
2557: foreach Inst = ["XVFMAX_S", "XVFMIN_S", "XVFMAXA_S", "XVFMINA_S", "XVFCVT_H_S",
2558:                 "XVFCMP_CAF_S", "XVFCMP_CUN_S", "XVFCMP_CEQ_S", "XVFCMP_CUEQ_S",
2559:                 "XVFCMP_CLT_S", "XVFCMP_CULT_S", "XVFCMP_CLE_S", "XVFCMP_CULE_S",
2560:                 "XVFCMP_CNE_S", "XVFCMP_COR_S", "XVFCMP_CUNE_S",
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2561-2600 / 第 2561-2600 行
```tablegen
2561:                 "XVFCMP_SAF_S", "XVFCMP_SUN_S", "XVFCMP_SEQ_S", "XVFCMP_SUEQ_S",
2562:                 "XVFCMP_SLT_S", "XVFCMP_SULT_S", "XVFCMP_SLE_S", "XVFCMP_SULE_S",
2563:                 "XVFCMP_SNE_S", "XVFCMP_SOR_S", "XVFCMP_SUNE_S"] in
2564:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2565:                (v8f32 LASX256:$xj), (v8f32 LASX256:$xk)),
2566:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk)>;
2567: foreach Inst = ["XVFMAX_D", "XVFMIN_D", "XVFMAXA_D", "XVFMINA_D", "XVFCVT_S_D",
2568:                 "XVFTINTRNE_W_D", "XVFTINTRZ_W_D", "XVFTINTRP_W_D", "XVFTINTRM_W_D",
2569:                 "XVFTINT_W_D",
2570:                 "XVFCMP_CAF_D", "XVFCMP_CUN_D", "XVFCMP_CEQ_D", "XVFCMP_CUEQ_D",
2571:                 "XVFCMP_CLT_D", "XVFCMP_CULT_D", "XVFCMP_CLE_D", "XVFCMP_CULE_D",
2572:                 "XVFCMP_CNE_D", "XVFCMP_COR_D", "XVFCMP_CUNE_D",
2573:                 "XVFCMP_SAF_D", "XVFCMP_SUN_D", "XVFCMP_SEQ_D", "XVFCMP_SUEQ_D",
2574:                 "XVFCMP_SLT_D", "XVFCMP_SULT_D", "XVFCMP_SLE_D", "XVFCMP_SULE_D",
2575:                 "XVFCMP_SNE_D", "XVFCMP_SOR_D", "XVFCMP_SUNE_D"] in
2576:   def : Pat<(deriveLASXIntrinsic<Inst>.ret
2577:                (v4f64 LASX256:$xj), (v4f64 LASX256:$xk)),
2578:             (!cast<LAInst>(Inst) LASX256:$xj, LASX256:$xk)>;
2579: 
2580: // vty: v8f32/v4f64
2581: // Pat<(Intrinsic vty:$xj),
2582: //     (LAInst vty:$xj)>;
2583: foreach Inst = ["XVFLOGB_S", "XVFCLASS_S", "XVFSQRT_S", "XVFRECIP_S", "XVFRSQRT_S",
2584:                 "XVFRINT_S", "XVFCVTL_D_S", "XVFCVTH_D_S",
2585:                 "XVFRINTRNE_S", "XVFRINTRZ_S", "XVFRINTRP_S", "XVFRINTRM_S",
2586:                 "XVFTINTRNE_W_S", "XVFTINTRZ_W_S", "XVFTINTRP_W_S", "XVFTINTRM_W_S",
2587:                 "XVFTINT_W_S", "XVFTINTRZ_WU_S", "XVFTINT_WU_S",
2588:                 "XVFTINTRNEL_L_S", "XVFTINTRNEH_L_S", "XVFTINTRZL_L_S",
2589:                 "XVFTINTRZH_L_S", "XVFTINTRPL_L_S", "XVFTINTRPH_L_S",
2590:                 "XVFTINTRML_L_S", "XVFTINTRMH_L_S", "XVFTINTL_L_S",
2591:                 "XVFTINTH_L_S"] in
2592:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v8f32 LASX256:$xj)),
2593:             (!cast<LAInst>(Inst) LASX256:$xj)>;
2594: foreach Inst = ["XVFLOGB_D", "XVFCLASS_D", "XVFSQRT_D", "XVFRECIP_D", "XVFRSQRT_D",
2595:                 "XVFRINT_D",
2596:                 "XVFRINTRNE_D", "XVFRINTRZ_D", "XVFRINTRP_D", "XVFRINTRM_D",
2597:                 "XVFTINTRNE_L_D", "XVFTINTRZ_L_D", "XVFTINTRP_L_D", "XVFTINTRM_L_D",
2598:                 "XVFTINT_L_D", "XVFTINTRZ_LU_D", "XVFTINT_LU_D"] in
2599:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v4f64 LASX256:$xj)),
2600:             (!cast<LAInst>(Inst) LASX256:$xj)>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2601-2640 / 第 2601-2640 行
```tablegen
2601: 
2602: // 256-Bit vector FP approximate reciprocal operation
2603: let Predicates = [HasFrecipe] in {
2604: foreach Inst = ["XVFRECIPE_S", "XVFRSQRTE_S"] in
2605:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v8f32 LASX256:$xj)),
2606:             (!cast<LAInst>(Inst) LASX256:$xj)>;
2607: foreach Inst = ["XVFRECIPE_D", "XVFRSQRTE_D"] in
2608:   def : Pat<(deriveLASXIntrinsic<Inst>.ret (v4f64 LASX256:$xj)),
2609:             (!cast<LAInst>(Inst) LASX256:$xj)>;
2610: 
2611: def : Pat<(loongarch_vfrecipe v8f32:$src),
2612:           (XVFRECIPE_S v8f32:$src)>;
2613: def : Pat<(loongarch_vfrecipe v4f64:$src),
2614:           (XVFRECIPE_D v4f64:$src)>;
2615: def : Pat<(loongarch_vfrsqrte v8f32:$src),
2616:           (XVFRSQRTE_S v8f32:$src)>;
2617: def : Pat<(loongarch_vfrsqrte v4f64:$src),
2618:           (XVFRSQRTE_D v4f64:$src)>;
2619: }
2620: 
2621: def : Pat<(int_loongarch_lasx_xvpickve_w_f v8f32:$xj, timm:$imm),
2622:           (XVPICKVE_W v8f32:$xj, (to_valid_timm timm:$imm))>;
2623: def : Pat<(int_loongarch_lasx_xvpickve_d_f v4f64:$xj, timm:$imm),
2624:           (XVPICKVE_D v4f64:$xj, (to_valid_timm timm:$imm))>;
2625: 
2626: // Vector floating-point conversion
2627: defm : PatXrF<fceil, "XVFRINTRP">;
2628: defm : PatXrF<ffloor, "XVFRINTRM">;
2629: defm : PatXrF<ftrunc, "XVFRINTRZ">;
2630: defm : PatXrF<froundeven, "XVFRINTRNE">;
2631: def : Pat<(v8f32 (loongarch_vfcvt_s_d (v4f64 LASX256:$xj), (v4f64 LASX256:$xk))),
2632:           (XVFCVT_S_D LASX256:$xj, LASX256:$xk)>;
2633: def : Pat<(v4f64 (loongarch_vfcvtl_d_s (v8f32 LASX256:$xj))),
2634:           (XVFCVTL_D_S LASX256:$xj)>;
2635: def : Pat<(v4f64 (loongarch_vfcvth_d_s (v8f32 LASX256:$xj))),
2636:           (XVFCVTH_D_S LASX256:$xj)>;
2637: 
2638: // load
2639: def : Pat<(int_loongarch_lasx_xvld GPR:$rj, timm:$imm),
2640:           (XVLD GPR:$rj, (to_valid_timm timm:$imm))>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2641-2680 / 第 2641-2680 行
```tablegen
2641: def : Pat<(int_loongarch_lasx_xvldx GPR:$rj, GPR:$rk),
2642:           (XVLDX GPR:$rj, GPR:$rk)>;
2643: 
2644: // xvldrepl
2645: def : Pat<(int_loongarch_lasx_xvldrepl_b GPR:$rj, timm:$imm),
2646:           (XVLDREPL_B GPR:$rj, (to_valid_timm timm:$imm))>;
2647: def : Pat<(int_loongarch_lasx_xvldrepl_h GPR:$rj, timm:$imm),
2648:           (XVLDREPL_H GPR:$rj, (to_valid_timm timm:$imm))>;
2649: def : Pat<(int_loongarch_lasx_xvldrepl_w GPR:$rj, timm:$imm),
2650:           (XVLDREPL_W GPR:$rj, (to_valid_timm timm:$imm))>;
2651: def : Pat<(int_loongarch_lasx_xvldrepl_d GPR:$rj, timm:$imm),
2652:           (XVLDREPL_D GPR:$rj, (to_valid_timm timm:$imm))>;
2653: 
2654: defm : VldreplPat<v32i8, XVLDREPL_B, simm12_addlike>;
2655: defm : VldreplPat<v16i16, XVLDREPL_H, simm11_lsl1>;
2656: defm : VldreplPat<v8i32, XVLDREPL_W, simm10_lsl2>;
2657: defm : VldreplPat<v4i64, XVLDREPL_D, simm9_lsl3>;
2658: defm : VldreplPat<v8f32, XVLDREPL_W, simm10_lsl2>;
2659: defm : VldreplPat<v4f64, XVLDREPL_D, simm9_lsl3>;
2660: 
2661: // store
2662: def : Pat<(int_loongarch_lasx_xvst LASX256:$xd, GPR:$rj, timm:$imm),
2663:           (XVST LASX256:$xd, GPR:$rj, (to_valid_timm timm:$imm))>;
2664: def : Pat<(int_loongarch_lasx_xvstx LASX256:$xd, GPR:$rj, GPR:$rk),
2665:           (XVSTX LASX256:$xd, GPR:$rj, GPR:$rk)>;
2666: 
2667: def : Pat<(int_loongarch_lasx_xvstelm_b v32i8:$xd, GPR:$rj, timm:$imm, timm:$idx),
2668:           (XVSTELM_B v32i8:$xd, GPR:$rj, (to_valid_timm timm:$imm),
2669:                     (to_valid_timm timm:$idx))>;
2670: def : Pat<(int_loongarch_lasx_xvstelm_h v16i16:$xd, GPR:$rj, timm:$imm, timm:$idx),
2671:           (XVSTELM_H v16i16:$xd, GPR:$rj, (to_valid_timm timm:$imm),
2672:                     (to_valid_timm timm:$idx))>;
2673: def : Pat<(int_loongarch_lasx_xvstelm_w v8i32:$xd, GPR:$rj, timm:$imm, timm:$idx),
2674:           (XVSTELM_W v8i32:$xd, GPR:$rj, (to_valid_timm timm:$imm),
2675:                     (to_valid_timm timm:$idx))>;
2676: def : Pat<(int_loongarch_lasx_xvstelm_d v4i64:$xd, GPR:$rj, timm:$imm, timm:$idx),
2677:           (XVSTELM_D v4i64:$xd, GPR:$rj, (to_valid_timm timm:$imm),
2678:                     (to_valid_timm timm:$idx))>;
2679: 
2680: } // Predicates = [HasExtLASX]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
