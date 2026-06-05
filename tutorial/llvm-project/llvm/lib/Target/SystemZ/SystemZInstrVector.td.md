# SystemZInstrVector.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrVector.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //==- SystemZInstrVector.td - SystemZ Vector instructions ------*- tblgen-*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: // Move instructions
  11: //===----------------------------------------------------------------------===//
  12: 
  13: let Predicates = [FeatureVector] in {
  14:   // Register move.
  15:   let isMoveReg = 1 in {
  16:     def VLR : UnaryVRRa<"vlr", 0xE756, null_frag, v128any, v128any>;
  17:     def VLR16 : UnaryAliasVRR<null_frag, v16hb, v16hb>;
  18:     def VLR32 : UnaryAliasVRR<null_frag, v32sb, v32sb>;
  19:     def VLR64 : UnaryAliasVRR<null_frag, v64db, v64db>;
  20:   }
  21: 
  22:   // Load GR from VR element.
  23:   def VLGV  : BinaryVRScGeneric<"vlgv", 0xE721>;
  24:   def VLGVB : BinaryVRSc<"vlgvb", 0xE721, null_frag, v128b, 0>;
  25:   def VLGVH : BinaryVRSc<"vlgvh", 0xE721, null_frag, v128h, 1>;
  26:   def VLGVF : BinaryVRSc<"vlgvf", 0xE721, null_frag, v128f, 2>;
  27:   def VLGVG : BinaryVRSc<"vlgvg", 0xE721, z_vector_extract, v128g, 3>;
  28: 
  29:   // Load VR element from GR.
  30:   def VLVG  : TernaryVRSbGeneric<"vlvg", 0xE722>;
  31:   def VLVGB : TernaryVRSb<"vlvgb", 0xE722, z_vector_insert,
  32:                           v128b, v128b, GR32, 0>;
  33:   def VLVGH : TernaryVRSb<"vlvgh", 0xE722, z_vector_insert,
  34:                           v128h, v128h, GR32, 1>;
  35:   def VLVGF : TernaryVRSb<"vlvgf", 0xE722, z_vector_insert,
  36:                           v128f, v128f, GR32, 2>;
  37:   def VLVGG : TernaryVRSb<"vlvgg", 0xE722, z_vector_insert,
  38:                           v128g, v128g, GR64, 3>;
  39: 
  40:   // Load VR from GRs disjoint.
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `VLR`, `VLR16`, `VLR32`, `VLR64`, `VLGV`, `VLGVB`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `VLR`, `VLR16`, `VLR32`, `VLR64`, `VLGV`, `VLGVB` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41:   def VLVGP : BinaryVRRf<"vlvgp", 0xE762, z_join_dwords, v128g>;
  42:   def VLVGP32 : BinaryAliasVRRf<GR32>;
  43: }
  44: 
  45: // Extractions always assign to the full GR64, even if the element would
  46: // fit in the lower 32 bits.  Sub-i64 extracts therefore need to take a
  47: // subreg of the result.
  48: class VectorExtractSubreg<ValueType type, Instruction insn>
  49:   : Pat<(i32 (z_vector_extract (type VR128:$vec), shift12only:$index)),
  50:         (EXTRACT_SUBREG (insn VR128:$vec, shift12only:$index), subreg_l32)>;
  51: 
  52: def : VectorExtractSubreg<v16i8, VLGVB>;
  53: def : VectorExtractSubreg<v8i16, VLGVH>;
  54: def : VectorExtractSubreg<v4i32, VLGVF>;
  55: 
  56: //===----------------------------------------------------------------------===//
  57: // Immediate instructions
  58: //===----------------------------------------------------------------------===//
  59: 
  60: let Predicates = [FeatureVector] in {
  61:   let isAsCheapAsAMove = 1, isMoveImm = 1, isReMaterializable = 1 in {
  62: 
  63:     // Generate byte mask.
  64:     def VZERO : InherentVRIa<"vzero", 0xE744, 0>;
  65:     def VONE  : InherentVRIa<"vone", 0xE744, 0xffff>;
  66:     def VGBM  : UnaryVRIa<"vgbm", 0xE744, z_byte_mask, v128b, imm32zx16_timm>;
  67: 
  68:     // Generate mask.
  69:     def VGM  : BinaryVRIbGeneric<"vgm", 0xE746>;
  70:     def VGMB : BinaryVRIb<"vgmb", 0xE746, z_rotate_mask, v128b, 0>;
  71:     def VGMH : BinaryVRIb<"vgmh", 0xE746, z_rotate_mask, v128h, 1>;
  72:     def VGMF : BinaryVRIb<"vgmf", 0xE746, z_rotate_mask, v128f, 2>;
  73:     def VGMG : BinaryVRIb<"vgmg", 0xE746, z_rotate_mask, v128g, 3>;
  74: 
  75:     // Replicate immediate.
  76:     def VREPI  : UnaryVRIaGeneric<"vrepi", 0xE745, imm32sx16>;
  77:     def VREPIB : UnaryVRIa<"vrepib", 0xE745, z_replicate, v128b, imm32sx16_timm, 0>;
  78:     def VREPIH : UnaryVRIa<"vrepih", 0xE745, z_replicate, v128h, imm32sx16_timm, 1>;
  79:     def VREPIF : UnaryVRIa<"vrepif", 0xE745, z_replicate, v128f, imm32sx16_timm, 2>;
  80:     def VREPIG : UnaryVRIa<"vrepig", 0xE745, z_replicate, v128g, imm32sx16_timm, 3>;
```
- **EN**: This block declares or refines TableGen records such as `VLVGP`, `VLVGP32`, `VectorExtractSubreg`, `VZERO`, `VONE`, `VGBM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `VLVGP`, `VLVGP32`, `VectorExtractSubreg`, `VZERO`, `VONE`, `VGBM` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 81-120 / 第 81-120 行
```tablegen
  81:   }
  82: 
  83:   // Load element immediate.
  84:   //
  85:   // We want these instructions to be used ahead of VLVG* where possible.
  86:   // However, VLVG* takes a variable BD-format index whereas VLEI takes
  87:   // a plain immediate index.  This means that VLVG* has an extra "base"
  88:   // register operand and is 3 units more complex.  Bumping the complexity
  89:   // of the VLEI* instructions by 4 means that they are strictly better
  90:   // than VLVG* in cases where both forms match.
  91:   let AddedComplexity = 4 in {
  92:     def VLEIB : TernaryVRIa<"vleib", 0xE740, z_vector_insert,
  93:                             v128b, v128b, imm32sx16trunc, imm32zx4>;
  94:     def VLEIH : TernaryVRIa<"vleih", 0xE741, z_vector_insert,
  95:                             v128h, v128h, imm32sx16trunc, imm32zx3>;
  96:     def VLEIF : TernaryVRIa<"vleif", 0xE743, z_vector_insert,
  97:                             v128f, v128f, imm32sx16, imm32zx2>;
  98:     def VLEIG : TernaryVRIa<"vleig", 0xE742, z_vector_insert,
  99:                             v128g, v128g, imm64sx16, imm32zx1>;
 100:   }
 101: }
 102: 
 103: //===----------------------------------------------------------------------===//
 104: // Loads
 105: //===----------------------------------------------------------------------===//
 106: 
 107: let Predicates = [FeatureVector] in {
 108:   // Load.
 109:   let SimpleBDXLoad = 1 in
 110:       defm VL : UnaryVRXAlign<"vl", 0xE706>;
 111: 
 112:   // Load to block boundary.  The number of loaded bytes is only known
 113:   // at run time.  The instruction is really polymorphic, but v128b matches
 114:   // the return type of the associated intrinsic.
 115:   def VLBB : BinaryVRX<"vlbb", 0xE707, int_s390_vlbb, v128b, 0>;
 116: 
 117:   // Load count to block boundary.
 118:   let Defs = [CC] in
 119:     def LCBB : InstRXE<0xE727, (outs GR32:$R1),
 120:                                (ins (bdxaddr12only $B2, $D2, $X2):$XBD2, imm32zx4:$M3),
```
- **EN**: This block declares or refines TableGen records such as `VLEIB`, `VLEIH`, `VLEIF`, `VLEIG`, `VL`, `VLBB`.
- **CN**: 该代码块声明或细化了 `VLEIB`, `VLEIH`, `VLEIF`, `VLEIG`, `VL`, `VLBB` 等 TableGen 记录。

### Lines 121-160 / 第 121-160 行
```tablegen
 121:                        "lcbb\t$R1, $XBD2, $M3",
 122:                        [(set GR32:$R1, (int_s390_lcbb bdxaddr12only:$XBD2,
 123:                                                       imm32zx4_timm:$M3))]>;
 124: 
 125:   // Load with length.  The number of loaded bytes is only known at run time.
 126:   def VLL : BinaryVRSb<"vll", 0xE737, int_s390_vll, 0>;
 127: 
 128:   // Load multiple.
 129:   defm VLM : LoadMultipleVRSaAlign<"vlm", 0xE736>;
 130: 
 131:   // Load and replicate
 132:   def VLREP  : UnaryVRXGeneric<"vlrep", 0xE705>;
 133:   def VLREPB : UnaryVRX<"vlrepb", 0xE705, z_replicate_loadi8,  v128b, 1, 0>;
 134:   def VLREPH : UnaryVRX<"vlreph", 0xE705, z_replicate_loadi16, v128h, 2, 1>;
 135:   def VLREPF : UnaryVRX<"vlrepf", 0xE705, z_replicate_loadi32, v128f, 4, 2>;
 136:   def VLREPG : UnaryVRX<"vlrepg", 0xE705, z_replicate_loadi64, v128g, 8, 3>;
 137:   def : Pat<(v8f16 (z_replicate_loadf16 bdxaddr12only:$addr)),
 138:             (VLREPH bdxaddr12only:$addr)>;
 139:   def : Pat<(v4f32 (z_replicate_loadf32 bdxaddr12only:$addr)),
 140:             (VLREPF bdxaddr12only:$addr)>;
 141:   def : Pat<(v2f64 (z_replicate_loadf64 bdxaddr12only:$addr)),
 142:             (VLREPG bdxaddr12only:$addr)>;
 143: 
 144:   // Use VLREP to load subvectors.  These patterns use "12pair" because
 145:   // LEY and LDY offer full 20-bit displacement fields.  It's often better
 146:   // to use those instructions rather than force a 20-bit displacement
 147:   // into a GPR temporary.
 148:   let mayLoad = 1, SimpleBDXLoad = 1, canFoldAsLoad = 1,
 149:       isReMaterializable = 1 in {
 150:     def VL16 : UnaryAliasVRX<z_load, v16hb, bdxaddr12pair>;
 151:     def VL32 : UnaryAliasVRX<z_load, v32sb, bdxaddr12pair>;
 152:     def VL64 : UnaryAliasVRX<z_load, v64db, bdxaddr12pair>;
 153:   }
 154: 
 155:   // Load logical element and zero.
 156:   def VLLEZ  : UnaryVRXGeneric<"vllez", 0xE704>;
 157:   def VLLEZB : UnaryVRX<"vllezb", 0xE704, z_vllezi8,  v128b, 1, 0>;
 158:   def VLLEZH : UnaryVRX<"vllezh", 0xE704, z_vllezi16, v128h, 2, 1>;
 159:   def VLLEZF : UnaryVRX<"vllezf", 0xE704, z_vllezi32, v128f, 4, 2>;
 160:   def VLLEZG : UnaryVRX<"vllezg", 0xE704, z_vllezi64, v128g, 8, 3>;
```
- **EN**: This block declares or refines TableGen records such as `VLL`, `VLM`, `VLREP`, `VLREPB`, `VLREPH`, `VLREPF`.
- **CN**: 该代码块声明或细化了 `VLL`, `VLM`, `VLREP`, `VLREPB`, `VLREPH`, `VLREPF` 等 TableGen 记录。

### Lines 161-200 / 第 161-200 行
```tablegen
 161:   def : Pat<(z_vllezf32 bdxaddr12only:$addr),
 162:             (VLLEZF bdxaddr12only:$addr)>;
 163:   def : Pat<(z_vllezf64 bdxaddr12only:$addr),
 164:             (VLLEZG bdxaddr12only:$addr)>;
 165:   let Predicates = [FeatureVectorEnhancements1] in {
 166:     def VLLEZLF : UnaryVRX<"vllezlf", 0xE704, z_vllezli32, v128f, 4, 6>;
 167:     def : Pat<(z_vllezlf32 bdxaddr12only:$addr),
 168:               (VLLEZLF bdxaddr12only:$addr)>;
 169:   }
 170: 
 171:   // Load element.
 172:   def VLEB : TernaryVRX<"vleb", 0xE700, z_vlei8,  v128b, v128b, 1, imm32zx4>;
 173:   def VLEH : TernaryVRX<"vleh", 0xE701, z_vlei16, v128h, v128h, 2, imm32zx3>;
 174:   def VLEF : TernaryVRX<"vlef", 0xE703, z_vlei32, v128f, v128f, 4, imm32zx2>;
 175:   def VLEG : TernaryVRX<"vleg", 0xE702, z_vlei64, v128g, v128g, 8, imm32zx1>;
 176:   def : Pat<(z_vlef32 (v4f32 VR128:$val), bdxaddr12only:$addr, imm32zx2:$index),
 177:             (VLEF VR128:$val, bdxaddr12only:$addr, imm32zx2:$index)>;
 178:   def : Pat<(z_vlef64 (v2f64 VR128:$val), bdxaddr12only:$addr, imm32zx1:$index),
 179:             (VLEG VR128:$val, bdxaddr12only:$addr, imm32zx1:$index)>;
 180: 
 181:   // Gather element.
 182:   def VGEF : TernaryVRV<"vgef", 0xE713, 4, imm32zx2>;
 183:   def VGEG : TernaryVRV<"vgeg", 0xE712, 8, imm32zx1>;
 184: }
 185: 
 186: let Predicates = [FeatureVectorPackedDecimal] in {
 187:   // Load rightmost with length.  The number of loaded bytes is only known
 188:   // at run time.  Note that while the instruction will accept immediate
 189:   // lengths larger that 15 at runtime, those will always result in a trap,
 190:   // so we never emit them here.
 191:   def VLRL : BinaryVSI<"vlrl", 0xE635, null_frag, 0>;
 192:   def VLRLR : BinaryVRSd<"vlrlr", 0xE637, int_s390_vlrl, 0>;
 193:   def : Pat<(int_s390_vlrl imm32zx4:$len, bdaddr12only:$addr),
 194:             (VLRL bdaddr12only:$addr, imm32zx4:$len)>;
 195: }
 196: 
 197: // Use replicating loads if we're inserting a single element into an
 198: // undefined vector.  This avoids a false dependency on the previous
 199: // register contents.
 200: multiclass ReplicatePeephole<Instruction vlrep, ValueType vectype,
```
- **EN**: This block declares or refines TableGen records such as `VLLEZLF`, `VLEB`, `VLEH`, `VLEF`, `VLEG`, `VGEF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `VLLEZLF`, `VLEB`, `VLEH`, `VLEF`, `VLEG`, `VGEF` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 201-240 / 第 201-240 行
```tablegen
 201:                              SDPatternOperator load, ValueType scalartype> {
 202:   def : Pat<(vectype (z_vector_insert
 203:                       (undef), (scalartype (load bdxaddr12only:$addr)), 0)),
 204:             (vlrep bdxaddr12only:$addr)>;
 205:   def : Pat<(vectype (scalar_to_vector
 206:                       (scalartype (load bdxaddr12only:$addr)))),
 207:             (vlrep bdxaddr12only:$addr)>;
 208: }
 209: defm : ReplicatePeephole<VLREPB, v16i8, z_anyextloadi8, i32>;
 210: defm : ReplicatePeephole<VLREPH, v8i16, z_anyextloadi16, i32>;
 211: defm : ReplicatePeephole<VLREPF, v4i32, z_load, i32>;
 212: defm : ReplicatePeephole<VLREPG, v2i64, z_load, i64>;
 213: defm : ReplicatePeephole<VLREPF, v4f32, z_load, f32>;
 214: defm : ReplicatePeephole<VLREPG, v2f64, z_load, f64>;
 215: 
 216: //===----------------------------------------------------------------------===//
 217: // Stores
 218: //===----------------------------------------------------------------------===//
 219: 
 220: let Predicates = [FeatureVector] in {
 221:   // Store.
 222:   let SimpleBDXStore = 1 in
 223:       defm VST : StoreVRXAlign<"vst", 0xE70E>;
 224: 
 225:   // Store with length.  The number of stored bytes is only known at run time.
 226:   def VSTL : StoreLengthVRSb<"vstl", 0xE73F, int_s390_vstl, 0>;
 227: 
 228:   // Store multiple.
 229:   defm VSTM : StoreMultipleVRSaAlign<"vstm", 0xE73E>;
 230: 
 231:   // Store element.
 232:   def VSTEB : StoreBinaryVRX<"vsteb", 0xE708, z_vstei8,  v128b, 1, imm32zx4>;
 233:   def VSTEH : StoreBinaryVRX<"vsteh", 0xE709, z_vstei16, v128h, 2, imm32zx3>;
 234:   def VSTEF : StoreBinaryVRX<"vstef", 0xE70B, z_vstei32, v128f, 4, imm32zx2>;
 235:   def VSTEG : StoreBinaryVRX<"vsteg", 0xE70A, z_vstei64, v128g, 8, imm32zx1>;
 236:   def : Pat<(z_vstef16 (v8f16 VR128:$val), bdxaddr12only:$addr,
 237:                        imm32zx3:$index),
 238:             (VSTEH VR128:$val, bdxaddr12only:$addr, imm32zx2:$index)>;
 239:   def : Pat<(z_vstef32 (v4f32 VR128:$val), bdxaddr12only:$addr,
 240:                        imm32zx2:$index),
```
- **EN**: This block declares or refines TableGen records such as `VST`, `VSTL`, `VSTM`, `VSTEB`, `VSTEH`, `VSTEF`.
- **CN**: 该代码块声明或细化了 `VST`, `VSTL`, `VSTM`, `VSTEB`, `VSTEH`, `VSTEF` 等 TableGen 记录。

### Lines 241-280 / 第 241-280 行
```tablegen
 241:             (VSTEF VR128:$val, bdxaddr12only:$addr, imm32zx2:$index)>;
 242:   def : Pat<(z_vstef64 (v2f64 VR128:$val), bdxaddr12only:$addr,
 243:                        imm32zx1:$index),
 244:             (VSTEG VR128:$val, bdxaddr12only:$addr, imm32zx1:$index)>;
 245: 
 246:   // Use VSTE to store subvectors.  These patterns use "12pair" because
 247:   // STEY and STDY offer full 20-bit displacement fields.  It's often better
 248:   // to use those instructions rather than force a 20-bit displacement
 249:   // into a GPR temporary.
 250:   let mayStore = 1, SimpleBDXStore = 1 in {
 251:     def VST16 : StoreAliasVRX<store, v16hb, bdxaddr12pair>;
 252:     def VST32 : StoreAliasVRX<store, v32sb, bdxaddr12pair>;
 253:     def VST64 : StoreAliasVRX<store, v64db, bdxaddr12pair>;
 254:   }
 255: 
 256:   // Scatter element.
 257:   def VSCEF : StoreBinaryVRV<"vscef", 0xE71B, 4, imm32zx2>;
 258:   def VSCEG : StoreBinaryVRV<"vsceg", 0xE71A, 8, imm32zx1>;
 259: }
 260: 
 261: let Predicates = [FeatureVectorPackedDecimal] in {
 262:   // Store rightmost with length.  The number of stored bytes is only known
 263:   // at run time.  Note that while the instruction will accept immediate
 264:   // lengths larger that 15 at runtime, those will always result in a trap,
 265:   // so we never emit them here.
 266:   def VSTRL : StoreLengthVSI<"vstrl", 0xE63D, null_frag, 0>;
 267:   def VSTRLR : StoreLengthVRSd<"vstrlr", 0xE63F, int_s390_vstrl, 0>;
 268:   def : Pat<(int_s390_vstrl VR128:$val, imm32zx4:$len, bdaddr12only:$addr),
 269:             (VSTRL VR128:$val, bdaddr12only:$addr, imm32zx4:$len)>;
 270: }
 271: 
 272: //===----------------------------------------------------------------------===//
 273: // Byte swaps
 274: //===----------------------------------------------------------------------===//
 275: 
 276: let Predicates = [FeatureVectorEnhancements2] in {
 277:   // Load byte-reversed elements.
 278:   def VLBR  : UnaryVRXGeneric<"vlbr", 0xE606>;
 279:   def VLBRH : UnaryVRX<"vlbrh", 0xE606, z_loadbswap, v128h, 16, 1>;
 280:   def VLBRF : UnaryVRX<"vlbrf", 0xE606, z_loadbswap, v128f, 16, 2>;
```
- **EN**: This block declares or refines TableGen records such as `VST16`, `VST32`, `VST64`, `VSCEF`, `VSCEG`, `VSTRL`.
- **CN**: 该代码块声明或细化了 `VST16`, `VST32`, `VST64`, `VSCEF`, `VSCEG`, `VSTRL` 等 TableGen 记录。

### Lines 281-320 / 第 281-320 行
```tablegen
 281:   def VLBRG : UnaryVRX<"vlbrg", 0xE606, z_loadbswap, v128g, 16, 3>;
 282:   def VLBRQ : UnaryVRX<"vlbrq", 0xE606, z_loadbswap, v128q, 16, 4>;
 283: 
 284:   // Load elements reversed.
 285:   def VLER  : UnaryVRXGeneric<"vler", 0xE607>;
 286:   def VLERH : UnaryVRX<"vlerh", 0xE607, z_loadeswap, v128h, 16, 1>;
 287:   def VLERF : UnaryVRX<"vlerf", 0xE607, z_loadeswap, v128f, 16, 2>;
 288:   def VLERG : UnaryVRX<"vlerg", 0xE607, z_loadeswap, v128g, 16, 3>;
 289:   def : Pat<(v8f16 (z_loadeswap bdxaddr12only:$addr)),
 290:             (VLERH bdxaddr12only:$addr)>;
 291:   def : Pat<(v4f32 (z_loadeswap bdxaddr12only:$addr)),
 292:             (VLERF bdxaddr12only:$addr)>;
 293:   def : Pat<(v2f64 (z_loadeswap bdxaddr12only:$addr)),
 294:             (VLERG bdxaddr12only:$addr)>;
 295:   def : Pat<(v16i8 (z_loadeswap bdxaddr12only:$addr)),
 296:             (VLBRQ bdxaddr12only:$addr)>;
 297: 
 298:   // Load byte-reversed element.
 299:   def VLEBRH : TernaryVRX<"vlebrh", 0xE601, z_vlebri16, v128h, v128h, 2, imm32zx3>;
 300:   def VLEBRF : TernaryVRX<"vlebrf", 0xE603, z_vlebri32, v128f, v128f, 4, imm32zx2>;
 301:   def VLEBRG : TernaryVRX<"vlebrg", 0xE602, z_vlebri64, v128g, v128g, 8, imm32zx1>;
 302: 
 303:   // Load byte-reversed element and zero.
 304:   def VLLEBRZ  : UnaryVRXGeneric<"vllebrz", 0xE604>;
 305:   def VLLEBRZH : UnaryVRX<"vllebrzh", 0xE604, z_vllebrzi16, v128h, 2, 1>;
 306:   def VLLEBRZF : UnaryVRX<"vllebrzf", 0xE604, z_vllebrzi32, v128f, 4, 2>;
 307:   def VLLEBRZG : UnaryVRX<"vllebrzg", 0xE604, z_vllebrzi64, v128g, 8, 3>;
 308:   def VLLEBRZE : UnaryVRX<"vllebrze", 0xE604, z_vllebrzli32, v128f, 4, 6>;
 309:   def : InstAlias<"lerv\t$V1, $XBD2",
 310:                   (VLLEBRZE VR128:$V1, bdxaddr12only:$XBD2), 0>;
 311:   def : InstAlias<"ldrv\t$V1, $XBD2",
 312:                   (VLLEBRZG VR128:$V1, bdxaddr12only:$XBD2), 0>;
 313: 
 314:   // Load byte-reversed element and replicate.
 315:   def VLBRREP  : UnaryVRXGeneric<"vlbrrep", 0xE605>;
 316:   def VLBRREPH : UnaryVRX<"vlbrreph", 0xE605, z_replicate_loadbswapi16, v128h, 2, 1>;
 317:   def VLBRREPF : UnaryVRX<"vlbrrepf", 0xE605, z_replicate_loadbswapi32, v128f, 4, 2>;
 318:   def VLBRREPG : UnaryVRX<"vlbrrepg", 0xE605, z_replicate_loadbswapi64, v128g, 8, 3>;
 319: 
 320:   // Store byte-reversed elements.
```
- **EN**: This block declares or refines TableGen records such as `VLBRG`, `VLBRQ`, `VLER`, `VLERH`, `VLERF`, `VLERG`.
- **CN**: 该代码块声明或细化了 `VLBRG`, `VLBRQ`, `VLER`, `VLERH`, `VLERF`, `VLERG` 等 TableGen 记录。

### Lines 321-360 / 第 321-360 行
```tablegen
 321:   def VSTBR  : StoreVRXGeneric<"vstbr", 0xE60E>;
 322:   def VSTBRH : StoreVRX<"vstbrh", 0xE60E, z_storebswap, v128h, 16, 1>;
 323:   def VSTBRF : StoreVRX<"vstbrf", 0xE60E, z_storebswap, v128f, 16, 2>;
 324:   def VSTBRG : StoreVRX<"vstbrg", 0xE60E, z_storebswap, v128g, 16, 3>;
 325:   def VSTBRQ : StoreVRX<"vstbrq", 0xE60E, z_storebswap, v128q, 16, 4>;
 326: 
 327:   // Store elements reversed.
 328:   def VSTER  : StoreVRXGeneric<"vster", 0xE60F>;
 329:   def VSTERH : StoreVRX<"vsterh", 0xE60F, z_storeeswap, v128h, 16, 1>;
 330:   def VSTERF : StoreVRX<"vsterf", 0xE60F, z_storeeswap, v128f, 16, 2>;
 331:   def VSTERG : StoreVRX<"vsterg", 0xE60F, z_storeeswap, v128g, 16, 3>;
 332:   def : Pat<(z_storeeswap (v8f16 VR128:$val), bdxaddr12only:$addr),
 333:             (VSTERH VR128:$val, bdxaddr12only:$addr)>;
 334:   def : Pat<(z_storeeswap (v4f32 VR128:$val), bdxaddr12only:$addr),
 335:             (VSTERF VR128:$val, bdxaddr12only:$addr)>;
 336:   def : Pat<(z_storeeswap (v2f64 VR128:$val), bdxaddr12only:$addr),
 337:             (VSTERG VR128:$val, bdxaddr12only:$addr)>;
 338:   def : Pat<(z_storeeswap (v16i8 VR128:$val), bdxaddr12only:$addr),
 339:             (VSTBRQ VR128:$val, bdxaddr12only:$addr)>;
 340: 
 341:   // Store byte-reversed element.
 342:   def VSTEBRH : StoreBinaryVRX<"vstebrh", 0xE609, z_vstebri16, v128h, 2, imm32zx3>;
 343:   def VSTEBRF : StoreBinaryVRX<"vstebrf", 0xE60B, z_vstebri32, v128f, 4, imm32zx2>;
 344:   def VSTEBRG : StoreBinaryVRX<"vstebrg", 0xE60A, z_vstebri64, v128g, 8, imm32zx1>;
 345:   def : InstAlias<"sterv\t$V1, $XBD2",
 346:                   (VSTEBRF VR128:$V1, bdxaddr12only:$XBD2, 0), 0>;
 347:   def : InstAlias<"stdrv\t$V1, $XBD2",
 348:                   (VSTEBRG VR128:$V1, bdxaddr12only:$XBD2, 0), 0>;
 349: }
 350: 
 351: //===----------------------------------------------------------------------===//
 352: // Selects and permutes
 353: //===----------------------------------------------------------------------===//
 354: 
 355: let Predicates = [FeatureVector] in {
 356:   // Merge high.
 357:   def VMRH:   BinaryVRRcGeneric<"vmrh", 0xE761>;
 358:   def VMRHB : BinaryVRRc<"vmrhb", 0xE761, z_merge_high, v128b, v128b, 0>;
 359:   def VMRHH : BinaryVRRc<"vmrhh", 0xE761, z_merge_high, v128h, v128h, 1>;
 360:   def VMRHF : BinaryVRRc<"vmrhf", 0xE761, z_merge_high, v128f, v128f, 2>;
```
- **EN**: This block declares or refines TableGen records such as `VSTBR`, `VSTBRH`, `VSTBRF`, `VSTBRG`, `VSTBRQ`, `VSTER`.
- **CN**: 该代码块声明或细化了 `VSTBR`, `VSTBRH`, `VSTBRF`, `VSTBRG`, `VSTBRQ`, `VSTER` 等 TableGen 记录。

### Lines 361-400 / 第 361-400 行
```tablegen
 361:   def VMRHG : BinaryVRRc<"vmrhg", 0xE761, z_merge_high, v128g, v128g, 3>;
 362:   def : BinaryRRWithType<VMRHH, VR128, z_merge_high, v8f16>;
 363:   def : BinaryRRWithType<VMRHF, VR128, z_merge_high, v4f32>;
 364:   def : BinaryRRWithType<VMRHG, VR128, z_merge_high, v2f64>;
 365: 
 366:   // Merge low.
 367:   def VMRL:   BinaryVRRcGeneric<"vmrl", 0xE760>;
 368:   def VMRLB : BinaryVRRc<"vmrlb", 0xE760, z_merge_low, v128b, v128b, 0>;
 369:   def VMRLH : BinaryVRRc<"vmrlh", 0xE760, z_merge_low, v128h, v128h, 1>;
 370:   def VMRLF : BinaryVRRc<"vmrlf", 0xE760, z_merge_low, v128f, v128f, 2>;
 371:   def VMRLG : BinaryVRRc<"vmrlg", 0xE760, z_merge_low, v128g, v128g, 3>;
 372:   def : BinaryRRWithType<VMRLH, VR128, z_merge_low, v8f16>;
 373:   def : BinaryRRWithType<VMRLF, VR128, z_merge_low, v4f32>;
 374:   def : BinaryRRWithType<VMRLG, VR128, z_merge_low, v2f64>;
 375: 
 376:   // Permute.
 377:   def VPERM : TernaryVRRe<"vperm", 0xE78C, z_permute, v128b, v128b>;
 378: 
 379:   // Permute doubleword immediate.
 380:   def VPDI : TernaryVRRc<"vpdi", 0xE784, z_permute_dwords, v128g, v128g>;
 381: 
 382:   // Bit Permute.
 383:   let Predicates = [FeatureVectorEnhancements1] in
 384:     def VBPERM : BinaryVRRc<"vbperm", 0xE785, int_s390_vbperm, v128g, v128b>;
 385: 
 386:   // Replicate.
 387:   def VREP:   BinaryVRIcGeneric<"vrep", 0xE74D>;
 388:   def VREPB : BinaryVRIc<"vrepb", 0xE74D, z_splat, v128b, v128b, 0>;
 389:   def VREPH : BinaryVRIc<"vreph", 0xE74D, z_splat, v128h, v128h, 1>;
 390:   def VREPF : BinaryVRIc<"vrepf", 0xE74D, z_splat, v128f, v128f, 2>;
 391:   def VREPG : BinaryVRIc<"vrepg", 0xE74D, z_splat, v128g, v128g, 3>;
 392:   def : Pat<(v8f16 (z_splat VR128:$vec, imm32zx16_timm:$index)),
 393:             (VREPH VR128:$vec, imm32zx16:$index)>;
 394:   def : Pat<(v4f32 (z_splat VR128:$vec, imm32zx16_timm:$index)),
 395:             (VREPF VR128:$vec, imm32zx16:$index)>;
 396:   def : Pat<(v2f64 (z_splat VR128:$vec, imm32zx16_timm:$index)),
 397:             (VREPG VR128:$vec, imm32zx16:$index)>;
 398: 
 399:   // Select.
 400:   def VSEL : TernaryVRRe<"vsel", 0xE78D, null_frag, v128any, v128any>;
```
- **EN**: This block declares or refines TableGen records such as `VMRHG`, `VMRL:`, `VMRLB`, `VMRLH`, `VMRLF`, `VMRLG`.
- **CN**: 该代码块声明或细化了 `VMRHG`, `VMRL:`, `VMRLB`, `VMRLH`, `VMRLF`, `VMRLG` 等 TableGen 记录。

### Lines 401-440 / 第 401-440 行
```tablegen
 401: 
 402:   // Blend.
 403:   let Predicates = [FeatureVectorEnhancements3] in {
 404:     def VBLEND  : TernaryVRRdGeneric<"vblend", 0xE789>;
 405:     def VBLENDB : TernaryVRRd<"vblendb", 0xE789, null_frag, v128b, v128b, 0>;
 406:     def VBLENDH : TernaryVRRd<"vblendh", 0xE789, null_frag, v128h, v128h, 1>;
 407:     def VBLENDF : TernaryVRRd<"vblendf", 0xE789, null_frag, v128f, v128f, 2>;
 408:     def VBLENDG : TernaryVRRd<"vblendg", 0xE789, null_frag, v128g, v128g, 3>;
 409:     def VBLENDQ : TernaryVRRd<"vblendq", 0xE789, null_frag, v128q, v128q, 4>;
 410:   }
 411: }
 412: 
 413: //===----------------------------------------------------------------------===//
 414: // Widening and narrowing
 415: //===----------------------------------------------------------------------===//
 416: 
 417: let Predicates = [FeatureVector] in {
 418:   // Pack
 419:   def VPK  : BinaryVRRcGeneric<"vpk", 0xE794>;
 420:   def VPKH : BinaryVRRc<"vpkh", 0xE794, z_pack, v128b, v128h, 1>;
 421:   def VPKF : BinaryVRRc<"vpkf", 0xE794, z_pack, v128h, v128f, 2>;
 422:   def VPKG : BinaryVRRc<"vpkg", 0xE794, z_pack, v128f, v128g, 3>;
 423: 
 424:   // Pack saturate.
 425:   def  VPKS  : BinaryVRRbSPairGeneric<"vpks", 0xE797>;
 426:   defm VPKSH : BinaryVRRbSPair<"vpksh", 0xE797, int_s390_vpksh, z_packs_cc,
 427:                                v128b, v128h, 1>;
 428:   defm VPKSF : BinaryVRRbSPair<"vpksf", 0xE797, int_s390_vpksf, z_packs_cc,
 429:                                v128h, v128f, 2>;
 430:   defm VPKSG : BinaryVRRbSPair<"vpksg", 0xE797, int_s390_vpksg, z_packs_cc,
 431:                                v128f, v128g, 3>;
 432: 
 433:   // Pack saturate logical.
 434:   def  VPKLS  : BinaryVRRbSPairGeneric<"vpkls", 0xE795>;
 435:   defm VPKLSH : BinaryVRRbSPair<"vpklsh", 0xE795, int_s390_vpklsh, z_packls_cc,
 436:                                 v128b, v128h, 1>;
 437:   defm VPKLSF : BinaryVRRbSPair<"vpklsf", 0xE795, int_s390_vpklsf, z_packls_cc,
 438:                                 v128h, v128f, 2>;
 439:   defm VPKLSG : BinaryVRRbSPair<"vpklsg", 0xE795, int_s390_vpklsg, z_packls_cc,
 440:                                 v128f, v128g, 3>;
```
- **EN**: This block declares or refines TableGen records such as `VBLEND`, `VBLENDB`, `VBLENDH`, `VBLENDF`, `VBLENDG`, `VBLENDQ`.
- **CN**: 该代码块声明或细化了 `VBLEND`, `VBLENDB`, `VBLENDH`, `VBLENDF`, `VBLENDG`, `VBLENDQ` 等 TableGen 记录。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: 
 442:   // Sign-extend to doubleword.
 443:   def VSEG  : UnaryVRRaGeneric<"vseg", 0xE75F>;
 444:   def VSEGB : UnaryVRRa<"vsegb", 0xE75F, z_vsei8,  v128g, v128g, 0>;
 445:   def VSEGH : UnaryVRRa<"vsegh", 0xE75F, z_vsei16, v128g, v128g, 1>;
 446:   def VSEGF : UnaryVRRa<"vsegf", 0xE75F, z_vsei32, v128g, v128g, 2>;
 447:   def : Pat<(z_vsei8_by_parts  (v16i8 VR128:$src)), (VSEGB VR128:$src)>;
 448:   def : Pat<(z_vsei16_by_parts (v8i16 VR128:$src)), (VSEGH VR128:$src)>;
 449:   def : Pat<(z_vsei32_by_parts (v4i32 VR128:$src)), (VSEGF VR128:$src)>;
 450: 
 451:   // Generate element masks.
 452:   let Predicates = [FeatureVectorEnhancements3] in {
 453:     def VGEM  : UnaryVRRaGeneric<"vgem", 0xE754>;
 454:     def VGEMB : UnaryVRRa<"vgemb", 0xE754, int_s390_vgemb, v128b, v128h, 0>;
 455:     def VGEMH : UnaryVRRa<"vgemh", 0xE754, int_s390_vgemh, v128h, v128b, 1>;
 456:     def VGEMF : UnaryVRRa<"vgemf", 0xE754, int_s390_vgemf, v128f, v128b, 2>;
 457:     def VGEMG : UnaryVRRa<"vgemg", 0xE754, int_s390_vgemg, v128g, v128b, 3>;
 458:     def VGEMQ : UnaryVRRa<"vgemq", 0xE754, int_s390_vgemq, v128q, v128b, 4>;
 459:   }
 460: 
 461:   // Unpack high.
 462:   def VUPH  : UnaryVRRaGeneric<"vuph", 0xE7D7>;
 463:   def VUPHB : UnaryVRRa<"vuphb", 0xE7D7, z_unpack_high, v128h, v128b, 0>;
 464:   def VUPHH : UnaryVRRa<"vuphh", 0xE7D7, z_unpack_high, v128f, v128h, 1>;
 465:   def VUPHF : UnaryVRRa<"vuphf", 0xE7D7, z_unpack_high, v128g, v128f, 2>;
 466:   let Predicates = [FeatureVectorEnhancements3] in
 467:     def VUPHG : UnaryVRRa<"vuphg", 0xE7D7, z_unpack_high, v128q, v128g, 3>;
 468: 
 469:   // Unpack logical high.
 470:   def VUPLH  : UnaryVRRaGeneric<"vuplh", 0xE7D5>;
 471:   def VUPLHB : UnaryVRRa<"vuplhb", 0xE7D5, z_unpackl_high, v128h, v128b, 0>;
 472:   def VUPLHH : UnaryVRRa<"vuplhh", 0xE7D5, z_unpackl_high, v128f, v128h, 1>;
 473:   def VUPLHF : UnaryVRRa<"vuplhf", 0xE7D5, z_unpackl_high, v128g, v128f, 2>;
 474:   let Predicates = [FeatureVectorEnhancements3] in
 475:     def VUPLHG : UnaryVRRa<"vuplhg", 0xE7D5, z_unpackl_high, v128q, v128g, 3>;
 476: 
 477:   // Unpack low.
 478:   def VUPL   : UnaryVRRaGeneric<"vupl", 0xE7D6>;
 479:   def VUPLB  : UnaryVRRa<"vuplb",  0xE7D6, z_unpack_low, v128h, v128b, 0>;
 480:   def VUPLHW : UnaryVRRa<"vuplhw", 0xE7D6, z_unpack_low, v128f, v128h, 1>;
```
- **EN**: This block declares or refines TableGen records such as `VSEG`, `VSEGB`, `VSEGH`, `VSEGF`, `VGEM`, `VGEMB`.
- **CN**: 该代码块声明或细化了 `VSEG`, `VSEGB`, `VSEGH`, `VSEGF`, `VGEM`, `VGEMB` 等 TableGen 记录。

### Lines 481-520 / 第 481-520 行
```tablegen
 481:   def VUPLF  : UnaryVRRa<"vuplf",  0xE7D6, z_unpack_low, v128g, v128f, 2>;
 482:   let Predicates = [FeatureVectorEnhancements3] in
 483:     def VUPLG  : UnaryVRRa<"vuplg",  0xE7D6, z_unpack_low, v128q, v128g, 3>;
 484: 
 485:   // Unpack logical low.
 486:   def VUPLL  : UnaryVRRaGeneric<"vupll", 0xE7D4>;
 487:   def VUPLLB : UnaryVRRa<"vupllb", 0xE7D4, z_unpackl_low, v128h, v128b, 0>;
 488:   def VUPLLH : UnaryVRRa<"vupllh", 0xE7D4, z_unpackl_low, v128f, v128h, 1>;
 489:   def VUPLLF : UnaryVRRa<"vupllf", 0xE7D4, z_unpackl_low, v128g, v128f, 2>;
 490:   let Predicates = [FeatureVectorEnhancements3] in
 491:     def VUPLLG : UnaryVRRa<"vupllg", 0xE7D4, z_unpackl_low, v128q, v128g, 3>;
 492: }
 493: 
 494: //===----------------------------------------------------------------------===//
 495: // Instantiating generic operations for specific types.
 496: //===----------------------------------------------------------------------===//
 497: 
 498: multiclass GenericVectorOps<ValueType type, ValueType inttype> {
 499:   let Predicates = [FeatureVector] in {
 500:     def : Pat<(type (load bdxaddr12only:$addr)),
 501:               (VL bdxaddr12only:$addr)>;
 502:     def : Pat<(store (type VR128:$src), bdxaddr12only:$addr),
 503:               (VST VR128:$src, bdxaddr12only:$addr)>;
 504:     def : Pat<(type (vselect (inttype VR128:$x), VR128:$y, VR128:$z)),
 505:               (VSEL VR128:$y, VR128:$z, VR128:$x)>;
 506:     def : Pat<(type (vselect (inttype (z_vnot VR128:$x)), VR128:$y, VR128:$z)),
 507:               (VSEL VR128:$z, VR128:$y, VR128:$x)>;
 508:   }
 509: }
 510: 
 511: defm : GenericVectorOps<v16i8, v16i8>;
 512: defm : GenericVectorOps<v8i16, v8i16>;
 513: defm : GenericVectorOps<v4i32, v4i32>;
 514: defm : GenericVectorOps<v2i64, v2i64>;
 515: defm : GenericVectorOps<v8f16, v8i16>;
 516: defm : GenericVectorOps<v4f32, v4i32>;
 517: defm : GenericVectorOps<v2f64, v2i64>;
 518: 
 519: multiclass BlendVectorOps<ValueType type, ValueType inttype,
 520:                           Instruction blend> {
```
- **EN**: This block declares or refines TableGen records such as `VUPLF`, `VUPLG`, `VUPLL`, `VUPLLB`, `VUPLLH`, `VUPLLF`.
- **CN**: 该代码块声明或细化了 `VUPLF`, `VUPLG`, `VUPLL`, `VUPLLB`, `VUPLLH`, `VUPLLF` 等 TableGen 记录。

### Lines 521-560 / 第 521-560 行
```tablegen
 521:   let Predicates = [FeatureVectorEnhancements3] in {
 522:     def : Pat<(type (vselect (inttype (z_vicmpl_zero VR128:$x)),
 523:                              VR128:$y, VR128:$z)),
 524:               (blend VR128:$y, VR128:$z, VR128:$x)>;
 525:     def : Pat<(type (vselect (inttype (z_vnot (z_vicmpl_zero VR128:$x))),
 526:                              VR128:$y, VR128:$z)),
 527:               (blend VR128:$z, VR128:$y, VR128:$x)>;
 528:   }
 529: }
 530: 
 531: defm : BlendVectorOps<v16i8, v16i8, VBLENDB>;
 532: defm : BlendVectorOps<v8i16, v8i16, VBLENDH>;
 533: defm : BlendVectorOps<v4i32, v4i32, VBLENDF>;
 534: defm : BlendVectorOps<v2i64, v2i64, VBLENDG>;
 535: defm : BlendVectorOps<v8f16, v8i16, VBLENDH>;
 536: defm : BlendVectorOps<v4f32, v4i32, VBLENDF>;
 537: defm : BlendVectorOps<v2f64, v2i64, VBLENDG>;
 538: 
 539: let Predicates = [FeatureVectorEnhancements3] in {
 540:     def : Pat<(i128 (or (and VR128:$y, (z_vicmph 0, VR128:$x)),
 541:                         (and VR128:$z, (not (z_vicmph 0, VR128:$x))))),
 542:               (VBLENDQ VR128:$y, VR128:$z, VR128:$x)>;
 543: }
 544: 
 545: //===----------------------------------------------------------------------===//
 546: // Integer arithmetic
 547: //===----------------------------------------------------------------------===//
 548: 
 549: let Predicates = [FeatureVector] in {
 550:   let isCommutable = 1 in {
 551:     // Add.
 552:     def VA  : BinaryVRRcGeneric<"va", 0xE7F3>;
 553:     def VAB : BinaryVRRc<"vab", 0xE7F3, add, v128b, v128b, 0>;
 554:     def VAH : BinaryVRRc<"vah", 0xE7F3, add, v128h, v128h, 1>;
 555:     def VAF : BinaryVRRc<"vaf", 0xE7F3, add, v128f, v128f, 2>;
 556:     def VAG : BinaryVRRc<"vag", 0xE7F3, add, v128g, v128g, 3>;
 557:     def VAQ : BinaryVRRc<"vaq", 0xE7F3, add, v128q, v128q, 4>;
 558:   }
 559: 
 560:   let isCommutable = 1 in {
```
- **EN**: This block declares or refines TableGen records such as `VA`, `VAB`, `VAH`, `VAF`, `VAG`, `VAQ`.
- **CN**: 该代码块声明或细化了 `VA`, `VAB`, `VAH`, `VAF`, `VAG`, `VAQ` 等 TableGen 记录。

### Lines 561-600 / 第 561-600 行
```tablegen
 561:     // Add compute carry.
 562:     def VACC  : BinaryVRRcGeneric<"vacc", 0xE7F1>;
 563:     def VACCB : BinaryVRRc<"vaccb", 0xE7F1, z_vacc, v128b, v128b, 0>;
 564:     def VACCH : BinaryVRRc<"vacch", 0xE7F1, z_vacc, v128h, v128h, 1>;
 565:     def VACCF : BinaryVRRc<"vaccf", 0xE7F1, z_vacc, v128f, v128f, 2>;
 566:     def VACCG : BinaryVRRc<"vaccg", 0xE7F1, z_vacc, v128g, v128g, 3>;
 567:     def VACCQ : BinaryVRRc<"vaccq", 0xE7F1, z_vacc, v128q, v128q, 4>;
 568: 
 569:     // Add with carry.
 570:     def VAC  : TernaryVRRdGeneric<"vac", 0xE7BB>;
 571:     def VACQ : TernaryVRRd<"vacq", 0xE7BB, z_vac, v128q, v128q, 4>;
 572: 
 573:     // Add with carry compute carry.
 574:     def VACCC  : TernaryVRRdGeneric<"vaccc", 0xE7B9>;
 575:     def VACCCQ : TernaryVRRd<"vacccq", 0xE7B9, z_vaccc, v128q, v128q, 4>;
 576:   }
 577: 
 578:   // And.
 579:   let isCommutable = 1 in
 580:     def VN : BinaryVRRc<"vn", 0xE768, null_frag, v128any, v128any>;
 581: 
 582:   // And with complement.
 583:   def VNC : BinaryVRRc<"vnc", 0xE769, null_frag, v128any, v128any>;
 584: 
 585:   let isCommutable = 1 in {
 586:     // Average.
 587:     def VAVG  : BinaryVRRcGeneric<"vavg", 0xE7F2>;
 588:     def VAVGB : BinaryVRRc<"vavgb", 0xE7F2, int_s390_vavgb, v128b, v128b, 0>;
 589:     def VAVGH : BinaryVRRc<"vavgh", 0xE7F2, int_s390_vavgh, v128h, v128h, 1>;
 590:     def VAVGF : BinaryVRRc<"vavgf", 0xE7F2, int_s390_vavgf, v128f, v128f, 2>;
 591:     def VAVGG : BinaryVRRc<"vavgg", 0xE7F2, int_s390_vavgg, v128g, v128g, 3>;
 592:     let Predicates = [FeatureVectorEnhancements3] in
 593:       def VAVGQ : BinaryVRRc<"vavgq", 0xE7F2, int_s390_vavgq, v128q, v128q, 4>;
 594: 
 595:     // Average logical.
 596:     def VAVGL  : BinaryVRRcGeneric<"vavgl", 0xE7F0>;
 597:     def VAVGLB : BinaryVRRc<"vavglb", 0xE7F0, int_s390_vavglb, v128b, v128b, 0>;
 598:     def VAVGLH : BinaryVRRc<"vavglh", 0xE7F0, int_s390_vavglh, v128h, v128h, 1>;
 599:     def VAVGLF : BinaryVRRc<"vavglf", 0xE7F0, int_s390_vavglf, v128f, v128f, 2>;
 600:     def VAVGLG : BinaryVRRc<"vavglg", 0xE7F0, int_s390_vavglg, v128g, v128g, 3>;
```
- **EN**: This block declares or refines TableGen records such as `VACC`, `VACCB`, `VACCH`, `VACCF`, `VACCG`, `VACCQ`.
- **CN**: 该代码块声明或细化了 `VACC`, `VACCB`, `VACCH`, `VACCF`, `VACCG`, `VACCQ` 等 TableGen 记录。

### Lines 601-640 / 第 601-640 行
```tablegen
 601:     let Predicates = [FeatureVectorEnhancements3] in
 602:       def VAVGLQ : BinaryVRRc<"vavglq", 0xE7F0, int_s390_vavglq, v128q, v128q, 4>;
 603:   }
 604: 
 605:   // Checksum.
 606:   def VCKSM : BinaryVRRc<"vcksm", 0xE766, int_s390_vcksm, v128f, v128f>;
 607: 
 608:   // Count leading zeros.
 609:   def VCLZ  : UnaryVRRaGeneric<"vclz", 0xE753>;
 610:   def VCLZB : UnaryVRRa<"vclzb", 0xE753, ctlz, v128b, v128b, 0>;
 611:   def VCLZH : UnaryVRRa<"vclzh", 0xE753, ctlz, v128h, v128h, 1>;
 612:   def VCLZF : UnaryVRRa<"vclzf", 0xE753, ctlz, v128f, v128f, 2>;
 613:   def VCLZG : UnaryVRRa<"vclzg", 0xE753, ctlz, v128g, v128g, 3>;
 614:   let Predicates = [FeatureVectorEnhancements3] in
 615:     def VCLZQ : UnaryVRRa<"vclzq", 0xE753, ctlz, v128q, v128q, 4>;
 616: 
 617:   // Count trailing zeros.
 618:   def VCTZ  : UnaryVRRaGeneric<"vctz", 0xE752>;
 619:   def VCTZB : UnaryVRRa<"vctzb", 0xE752, cttz, v128b, v128b, 0>;
 620:   def VCTZH : UnaryVRRa<"vctzh", 0xE752, cttz, v128h, v128h, 1>;
 621:   def VCTZF : UnaryVRRa<"vctzf", 0xE752, cttz, v128f, v128f, 2>;
 622:   def VCTZG : UnaryVRRa<"vctzg", 0xE752, cttz, v128g, v128g, 3>;
 623:   let Predicates = [FeatureVectorEnhancements3] in
 624:     def VCTZQ : UnaryVRRa<"vctzq", 0xE752, cttz, v128q, v128q, 4>;
 625: 
 626:   // Divide.
 627:   let Predicates = [FeatureVectorEnhancements3] in {
 628:     let hasSideEffects = 1 in {
 629:       def VD  : TernaryVRRcIntGeneric<"vd", 0xE7B2>;
 630:       def VDF : TernaryVRRcInt<"vdf", 0xE7B2, null_frag, v128f, v128f, 2>;
 631:       def VDG : TernaryVRRcInt<"vdg", 0xE7B2, null_frag, v128g, v128g, 3>;
 632:       def VDQ : TernaryVRRcInt<"vdq", 0xE7B2, null_frag, v128q, v128q, 4>;
 633:     }
 634:     def : Pat<(v4i32 (sdiv VR128:$x, VR128:$y)), (VDF VR128:$x, VR128:$y, 0)>;
 635:     def : Pat<(v2i64 (sdiv VR128:$x, VR128:$y)), (VDG VR128:$x, VR128:$y, 0)>;
 636:     def : Pat<(i128 (sdiv VR128:$x, VR128:$y)), (VDQ VR128:$x, VR128:$y, 0)>;
 637:   }
 638: 
 639:   // Divide logical.
 640:   let Predicates = [FeatureVectorEnhancements3] in {
```
- **EN**: This block declares or refines TableGen records such as `VAVGLQ`, `VCKSM`, `VCLZ`, `VCLZB`, `VCLZH`, `VCLZF`.
- **CN**: 该代码块声明或细化了 `VAVGLQ`, `VCKSM`, `VCLZ`, `VCLZB`, `VCLZH`, `VCLZF` 等 TableGen 记录。

### Lines 641-680 / 第 641-680 行
```tablegen
 641:     let hasSideEffects = 1 in {
 642:       def VDL  : TernaryVRRcIntGeneric<"vdl", 0xE7B0>;
 643:       def VDLF : TernaryVRRcInt<"vdlf", 0xE7B0, null_frag, v128f, v128f, 2>;
 644:       def VDLG : TernaryVRRcInt<"vdlg", 0xE7B0, null_frag, v128g, v128g, 3>;
 645:       def VDLQ : TernaryVRRcInt<"vdlq", 0xE7B0, null_frag, v128q, v128q, 4>;
 646:     }
 647:     def : Pat<(v4i32 (udiv VR128:$x, VR128:$y)), (VDLF VR128:$x, VR128:$y, 0)>;
 648:     def : Pat<(v2i64 (udiv VR128:$x, VR128:$y)), (VDLG VR128:$x, VR128:$y, 0)>;
 649:     def : Pat<(i128 (udiv VR128:$x, VR128:$y)), (VDLQ VR128:$x, VR128:$y, 0)>;
 650:   }
 651: 
 652:   // Evaluate.
 653:   let Predicates = [FeatureVectorEnhancements3] in
 654:     def VEVAL : QuaternaryVRIk<"veval", 0xE788, int_s390_veval, v128b>;
 655: 
 656:   let isCommutable = 1 in {
 657:     // Not exclusive or.
 658:     let Predicates = [FeatureVectorEnhancements1] in
 659:       def VNX : BinaryVRRc<"vnx", 0xE76C, null_frag, v128any, v128any>;
 660: 
 661:     // Exclusive or.
 662:     def VX : BinaryVRRc<"vx", 0xE76D, null_frag, v128any, v128any>;
 663:   }
 664: 
 665:   // Galois field multiply sum.
 666:   def VGFM  : BinaryVRRcGeneric<"vgfm", 0xE7B4>;
 667:   def VGFMB : BinaryVRRc<"vgfmb", 0xE7B4, int_s390_vgfmb, v128h, v128b, 0>;
 668:   def VGFMH : BinaryVRRc<"vgfmh", 0xE7B4, int_s390_vgfmh, v128f, v128h, 1>;
 669:   def VGFMF : BinaryVRRc<"vgfmf", 0xE7B4, int_s390_vgfmf, v128g, v128f, 2>;
 670:   def VGFMG : BinaryVRRc<"vgfmg", 0xE7B4, int_s390_vgfmg, v128q, v128g, 3>;
 671: 
 672:   // Galois field multiply sum and accumulate.
 673:   def VGFMA  : TernaryVRRdGeneric<"vgfma", 0xE7BC>;
 674:   def VGFMAB : TernaryVRRd<"vgfmab", 0xE7BC, int_s390_vgfmab, v128h, v128b, 0>;
 675:   def VGFMAH : TernaryVRRd<"vgfmah", 0xE7BC, int_s390_vgfmah, v128f, v128h, 1>;
 676:   def VGFMAF : TernaryVRRd<"vgfmaf", 0xE7BC, int_s390_vgfmaf, v128g, v128f, 2>;
 677:   def VGFMAG : TernaryVRRd<"vgfmag", 0xE7BC, int_s390_vgfmag, v128q, v128g, 3>;
 678: 
 679:   // Load complement.
 680:   def VLC  : UnaryVRRaGeneric<"vlc", 0xE7DE>;
```
- **EN**: This block declares or refines TableGen records such as `VDL`, `VDLF`, `VDLG`, `VDLQ`, `VEVAL`, `VNX`.
- **CN**: 该代码块声明或细化了 `VDL`, `VDLF`, `VDLG`, `VDLQ`, `VEVAL`, `VNX` 等 TableGen 记录。

### Lines 681-720 / 第 681-720 行
```tablegen
 681:   def VLCB : UnaryVRRa<"vlcb", 0xE7DE, z_vneg, v128b, v128b, 0>;
 682:   def VLCH : UnaryVRRa<"vlch", 0xE7DE, z_vneg, v128h, v128h, 1>;
 683:   def VLCF : UnaryVRRa<"vlcf", 0xE7DE, z_vneg, v128f, v128f, 2>;
 684:   def VLCG : UnaryVRRa<"vlcg", 0xE7DE, z_vneg, v128g, v128g, 3>;
 685:   let Predicates = [FeatureVectorEnhancements3] in
 686:     def VLCQ : UnaryVRRa<"vlcq", 0xE7DE, ineg, v128q, v128q, 4>;
 687: 
 688:   // Load positive.
 689:   def VLP  : UnaryVRRaGeneric<"vlp", 0xE7DF>;
 690:   def VLPB : UnaryVRRa<"vlpb", 0xE7DF, abs, v128b, v128b, 0>;
 691:   def VLPH : UnaryVRRa<"vlph", 0xE7DF, abs, v128h, v128h, 1>;
 692:   def VLPF : UnaryVRRa<"vlpf", 0xE7DF, abs, v128f, v128f, 2>;
 693:   def VLPG : UnaryVRRa<"vlpg", 0xE7DF, abs, v128g, v128g, 3>;
 694:   let Predicates = [FeatureVectorEnhancements3] in
 695:     def VLPQ : UnaryVRRa<"vlpq", 0xE7DF, abs, v128q, v128q, 4>;
 696: 
 697:   let isCommutable = 1 in {
 698:     // Maximum.
 699:     def VMX  : BinaryVRRcGeneric<"vmx", 0xE7FF>;
 700:     def VMXB : BinaryVRRc<"vmxb", 0xE7FF, smax, v128b, v128b, 0>;
 701:     def VMXH : BinaryVRRc<"vmxh", 0xE7FF, smax, v128h, v128h, 1>;
 702:     def VMXF : BinaryVRRc<"vmxf", 0xE7FF, smax, v128f, v128f, 2>;
 703:     def VMXG : BinaryVRRc<"vmxg", 0xE7FF, smax, v128g, v128g, 3>;
 704:     let Predicates = [FeatureVectorEnhancements3] in
 705:       def VMXQ : BinaryVRRc<"vmxq", 0xE7FF, smax, v128q, v128q, 4>;
 706: 
 707:     // Maximum logical.
 708:     def VMXL  : BinaryVRRcGeneric<"vmxl", 0xE7FD>;
 709:     def VMXLB : BinaryVRRc<"vmxlb", 0xE7FD, umax, v128b, v128b, 0>;
 710:     def VMXLH : BinaryVRRc<"vmxlh", 0xE7FD, umax, v128h, v128h, 1>;
 711:     def VMXLF : BinaryVRRc<"vmxlf", 0xE7FD, umax, v128f, v128f, 2>;
 712:     def VMXLG : BinaryVRRc<"vmxlg", 0xE7FD, umax, v128g, v128g, 3>;
 713:     let Predicates = [FeatureVectorEnhancements3] in
 714:       def VMXLQ : BinaryVRRc<"vmxlq", 0xE7FD, umax, v128q, v128q, 4>;
 715:   }
 716: 
 717:   let isCommutable = 1 in {
 718:     // Minimum.
 719:     def VMN  : BinaryVRRcGeneric<"vmn", 0xE7FE>;
 720:     def VMNB : BinaryVRRc<"vmnb", 0xE7FE, smin, v128b, v128b, 0>;
```
- **EN**: This block declares or refines TableGen records such as `VLCB`, `VLCH`, `VLCF`, `VLCG`, `VLCQ`, `VLP`.
- **CN**: 该代码块声明或细化了 `VLCB`, `VLCH`, `VLCF`, `VLCG`, `VLCQ`, `VLP` 等 TableGen 记录。

### Lines 721-760 / 第 721-760 行
```tablegen
 721:     def VMNH : BinaryVRRc<"vmnh", 0xE7FE, smin, v128h, v128h, 1>;
 722:     def VMNF : BinaryVRRc<"vmnf", 0xE7FE, smin, v128f, v128f, 2>;
 723:     def VMNG : BinaryVRRc<"vmng", 0xE7FE, smin, v128g, v128g, 3>;
 724:     let Predicates = [FeatureVectorEnhancements3] in
 725:       def VMNQ : BinaryVRRc<"vmnq", 0xE7FE, smin, v128q, v128q, 4>;
 726: 
 727:     // Minimum logical.
 728:     def VMNL  : BinaryVRRcGeneric<"vmnl", 0xE7FC>;
 729:     def VMNLB : BinaryVRRc<"vmnlb", 0xE7FC, umin, v128b, v128b, 0>;
 730:     def VMNLH : BinaryVRRc<"vmnlh", 0xE7FC, umin, v128h, v128h, 1>;
 731:     def VMNLF : BinaryVRRc<"vmnlf", 0xE7FC, umin, v128f, v128f, 2>;
 732:     def VMNLG : BinaryVRRc<"vmnlg", 0xE7FC, umin, v128g, v128g, 3>;
 733:     let Predicates = [FeatureVectorEnhancements3] in
 734:       def VMNLQ : BinaryVRRc<"vmnlq", 0xE7FC, umin, v128q, v128q, 4>;
 735:   }
 736: 
 737:   let isCommutable = 1 in {
 738:     // Multiply and add low.
 739:     def VMAL   : TernaryVRRdGeneric<"vmal", 0xE7AA>;
 740:     def VMALB  : TernaryVRRd<"vmalb",  0xE7AA, z_muladd<mul>, v128b, v128b, 0>;
 741:     def VMALHW : TernaryVRRd<"vmalhw", 0xE7AA, z_muladd<mul>, v128h, v128h, 1>;
 742:     def VMALF  : TernaryVRRd<"vmalf",  0xE7AA, z_muladd<mul>, v128f, v128f, 2>;
 743:     let Predicates = [FeatureVectorEnhancements3] in {
 744:       def VMALG : TernaryVRRd<"vmalg",  0xE7AA, z_muladd<mul>, v128g, v128g, 3>;
 745:       def VMALQ : TernaryVRRd<"vmalq",  0xE7AA, z_muladd<mul>, v128q, v128q, 4>;
 746:     }
 747: 
 748:     // Multiply and add high.
 749:     def VMAH  : TernaryVRRdGeneric<"vmah", 0xE7AB>;
 750:     def VMAHB : TernaryVRRd<"vmahb", 0xE7AB, z_vmah, v128b, v128b, 0>;
 751:     def VMAHH : TernaryVRRd<"vmahh", 0xE7AB, z_vmah, v128h, v128h, 1>;
 752:     def VMAHF : TernaryVRRd<"vmahf", 0xE7AB, z_vmah, v128f, v128f, 2>;
 753:     let Predicates = [FeatureVectorEnhancements3] in {
 754:       def VMAHG : TernaryVRRd<"vmahg", 0xE7AB, z_vmah, v128g, v128g, 3>;
 755:       def VMAHQ : TernaryVRRd<"vmahq", 0xE7AB, z_vmah, v128q, v128q, 4>;
 756:     }
 757: 
 758:     // Multiply and add logical high.
 759:     def VMALH  : TernaryVRRdGeneric<"vmalh", 0xE7A9>;
 760:     def VMALHB : TernaryVRRd<"vmalhb", 0xE7A9, z_vmalh, v128b, v128b, 0>;
```
- **EN**: This block declares or refines TableGen records such as `VMNH`, `VMNF`, `VMNG`, `VMNQ`, `VMNL`, `VMNLB`.
- **CN**: 该代码块声明或细化了 `VMNH`, `VMNF`, `VMNG`, `VMNQ`, `VMNL`, `VMNLB` 等 TableGen 记录。

### Lines 761-800 / 第 761-800 行
```tablegen
 761:     def VMALHH : TernaryVRRd<"vmalhh", 0xE7A9, z_vmalh, v128h, v128h, 1>;
 762:     def VMALHF : TernaryVRRd<"vmalhf", 0xE7A9, z_vmalh, v128f, v128f, 2>;
 763:     let Predicates = [FeatureVectorEnhancements3] in {
 764:       def VMALHG : TernaryVRRd<"vmalhg", 0xE7A9, z_vmalh, v128g, v128g, 3>;
 765:       def VMALHQ : TernaryVRRd<"vmalhq", 0xE7A9, z_vmalh, v128q, v128q, 4>;
 766:     }
 767: 
 768:     // Multiply and add even.
 769:     def VMAE  : TernaryVRRdGeneric<"vmae", 0xE7AE>;
 770:     def VMAEB : TernaryVRRd<"vmaeb", 0xE7AE, z_muladd<z_vme>, v128h, v128b, 0>;
 771:     def VMAEH : TernaryVRRd<"vmaeh", 0xE7AE, z_muladd<z_vme>, v128f, v128h, 1>;
 772:     def VMAEF : TernaryVRRd<"vmaef", 0xE7AE, z_muladd<z_vme>, v128g, v128f, 2>;
 773:     let Predicates = [FeatureVectorEnhancements3] in
 774:       def VMAEG : TernaryVRRd<"vmaeg", 0xE7AE, z_muladd<z_vme>, v128q, v128g, 3>;
 775: 
 776:     // Multiply and add logical even.
 777:     def VMALE  : TernaryVRRdGeneric<"vmale", 0xE7AC>;
 778:     def VMALEB : TernaryVRRd<"vmaleb", 0xE7AC, z_muladd<z_vmle>, v128h, v128b, 0>;
 779:     def VMALEH : TernaryVRRd<"vmaleh", 0xE7AC, z_muladd<z_vmle>, v128f, v128h, 1>;
 780:     def VMALEF : TernaryVRRd<"vmalef", 0xE7AC, z_muladd<z_vmle>, v128g, v128f, 2>;
 781:     let Predicates = [FeatureVectorEnhancements3] in
 782:       def VMALEG : TernaryVRRd<"vmaleg", 0xE7AC, z_muladd<z_vmle>, v128q, v128g, 3>;
 783: 
 784:     // Multiply and add odd.
 785:     def VMAO  : TernaryVRRdGeneric<"vmao", 0xE7AF>;
 786:     def VMAOB : TernaryVRRd<"vmaob", 0xE7AF, z_muladd<z_vmo>, v128h, v128b, 0>;
 787:     def VMAOH : TernaryVRRd<"vmaoh", 0xE7AF, z_muladd<z_vmo>, v128f, v128h, 1>;
 788:     def VMAOF : TernaryVRRd<"vmaof", 0xE7AF, z_muladd<z_vmo>, v128g, v128f, 2>;
 789:     let Predicates = [FeatureVectorEnhancements3] in
 790:       def VMAOG : TernaryVRRd<"vmaog", 0xE7AF, z_muladd<z_vmo>, v128q, v128g, 3>;
 791: 
 792:     // Multiply and add logical odd.
 793:     def VMALO  : TernaryVRRdGeneric<"vmalo", 0xE7AD>;
 794:     def VMALOB : TernaryVRRd<"vmalob", 0xE7AD, z_muladd<z_vmlo>, v128h, v128b, 0>;
 795:     def VMALOH : TernaryVRRd<"vmaloh", 0xE7AD, z_muladd<z_vmlo>, v128f, v128h, 1>;
 796:     def VMALOF : TernaryVRRd<"vmalof", 0xE7AD, z_muladd<z_vmlo>, v128g, v128f, 2>;
 797:     let Predicates = [FeatureVectorEnhancements3] in
 798:       def VMALOG : TernaryVRRd<"vmalog", 0xE7AD, z_muladd<z_vmlo>, v128q, v128g, 3>;
 799:   }
 800: 
```
- **EN**: This block declares or refines TableGen records such as `VMALHH`, `VMALHF`, `VMALHG`, `VMALHQ`, `VMAE`, `VMAEB`.
- **CN**: 该代码块声明或细化了 `VMALHH`, `VMALHF`, `VMALHG`, `VMALHQ`, `VMAE`, `VMAEB` 等 TableGen 记录。

### Lines 801-840 / 第 801-840 行
```tablegen
 801:   let isCommutable = 1 in {
 802:     // Multiply high.
 803:     def VMH  : BinaryVRRcGeneric<"vmh", 0xE7A3>;
 804:     def VMHB : BinaryVRRc<"vmhb", 0xE7A3, mulhs, v128b, v128b, 0>;
 805:     def VMHH : BinaryVRRc<"vmhh", 0xE7A3, mulhs, v128h, v128h, 1>;
 806:     def VMHF : BinaryVRRc<"vmhf", 0xE7A3, mulhs, v128f, v128f, 2>;
 807:     let Predicates = [FeatureVectorEnhancements3] in {
 808:       def VMHG : BinaryVRRc<"vmhg", 0xE7A3, mulhs, v128g, v128g, 3>;
 809:       def VMHQ : BinaryVRRc<"vmhq", 0xE7A3, mulhs, v128q, v128q, 4>;
 810:     }
 811: 
 812:     // Multiply logical high.
 813:     def VMLH  : BinaryVRRcGeneric<"vmlh", 0xE7A1>;
 814:     def VMLHB : BinaryVRRc<"vmlhb", 0xE7A1, mulhu, v128b, v128b, 0>;
 815:     def VMLHH : BinaryVRRc<"vmlhh", 0xE7A1, mulhu, v128h, v128h, 1>;
 816:     def VMLHF : BinaryVRRc<"vmlhf", 0xE7A1, mulhu, v128f, v128f, 2>;
 817:     let Predicates = [FeatureVectorEnhancements3] in {
 818:       def VMLHG : BinaryVRRc<"vmlhg", 0xE7A1, mulhu, v128g, v128g, 3>;
 819:       def VMLHQ : BinaryVRRc<"vmlhq", 0xE7A1, mulhu, v128q, v128q, 4>;
 820:     }
 821: 
 822:     // Multiply low.
 823:     def VML   : BinaryVRRcGeneric<"vml", 0xE7A2>;
 824:     def VMLB  : BinaryVRRc<"vmlb",  0xE7A2, mul, v128b, v128b, 0>;
 825:     def VMLHW : BinaryVRRc<"vmlhw", 0xE7A2, mul, v128h, v128h, 1>;
 826:     def VMLF  : BinaryVRRc<"vmlf",  0xE7A2, mul, v128f, v128f, 2>;
 827:     let Predicates = [FeatureVectorEnhancements3] in {
 828:       def VMLG : BinaryVRRc<"vmlg",  0xE7A2, mul, v128g, v128g, 3>;
 829:       def VMLQ : BinaryVRRc<"vmlq",  0xE7A2, mul, v128q, v128q, 4>;
 830:     }
 831: 
 832:     // Multiply even.
 833:     def VME  : BinaryVRRcGeneric<"vme", 0xE7A6>;
 834:     def VMEB : BinaryVRRc<"vmeb", 0xE7A6, z_vme, v128h, v128b, 0>;
 835:     def VMEH : BinaryVRRc<"vmeh", 0xE7A6, z_vme, v128f, v128h, 1>;
 836:     def VMEF : BinaryVRRc<"vmef", 0xE7A6, z_vme, v128g, v128f, 2>;
 837:     let Predicates = [FeatureVectorEnhancements3] in
 838:       def VMEG : BinaryVRRc<"vmeg", 0xE7A6, z_vme, v128q, v128g, 3>;
 839: 
 840:     // Multiply logical even.
```
- **EN**: This block declares or refines TableGen records such as `VMH`, `VMHB`, `VMHH`, `VMHF`, `VMHG`, `VMHQ`.
- **CN**: 该代码块声明或细化了 `VMH`, `VMHB`, `VMHH`, `VMHF`, `VMHG`, `VMHQ` 等 TableGen 记录。

### Lines 841-880 / 第 841-880 行
```tablegen
 841:     def VMLE  : BinaryVRRcGeneric<"vmle", 0xE7A4>;
 842:     def VMLEB : BinaryVRRc<"vmleb", 0xE7A4, z_vmle, v128h, v128b, 0>;
 843:     def VMLEH : BinaryVRRc<"vmleh", 0xE7A4, z_vmle, v128f, v128h, 1>;
 844:     def VMLEF : BinaryVRRc<"vmlef", 0xE7A4, z_vmle, v128g, v128f, 2>;
 845:     let Predicates = [FeatureVectorEnhancements3] in
 846:       def VMLEG : BinaryVRRc<"vmleg", 0xE7A4, z_vmle, v128q, v128g, 3>;
 847: 
 848:     // Multiply odd.
 849:     def VMO  : BinaryVRRcGeneric<"vmo", 0xE7A7>;
 850:     def VMOB : BinaryVRRc<"vmob", 0xE7A7, z_vmo, v128h, v128b, 0>;
 851:     def VMOH : BinaryVRRc<"vmoh", 0xE7A7, z_vmo, v128f, v128h, 1>;
 852:     def VMOF : BinaryVRRc<"vmof", 0xE7A7, z_vmo, v128g, v128f, 2>;
 853:     let Predicates = [FeatureVectorEnhancements3] in
 854:       def VMOG : BinaryVRRc<"vmog", 0xE7A7, z_vmo, v128q, v128g, 3>;
 855: 
 856:     // Multiply logical odd.
 857:     def VMLO  : BinaryVRRcGeneric<"vmlo", 0xE7A5>;
 858:     def VMLOB : BinaryVRRc<"vmlob", 0xE7A5, z_vmlo, v128h, v128b, 0>;
 859:     def VMLOH : BinaryVRRc<"vmloh", 0xE7A5, z_vmlo, v128f, v128h, 1>;
 860:     def VMLOF : BinaryVRRc<"vmlof", 0xE7A5, z_vmlo, v128g, v128f, 2>;
 861:     let Predicates = [FeatureVectorEnhancements3] in
 862:       def VMLOG : BinaryVRRc<"vmlog", 0xE7A5, z_vmlo, v128q, v128g, 3>;
 863:   }
 864: 
 865:   // Multiply sum logical.
 866:   let Predicates = [FeatureVectorEnhancements1], isCommutable = 1 in {
 867:     def VMSL  : QuaternaryVRRdGeneric<"vmsl", 0xE7B8>;
 868:     def VMSLG : QuaternaryVRRd<"vmslg", 0xE7B8, int_s390_vmslg,
 869:                                v128q, v128g, v128g, v128q, 3>;
 870:   }
 871: 
 872:   // Nand.
 873:   let Predicates = [FeatureVectorEnhancements1], isCommutable = 1 in
 874:     def VNN : BinaryVRRc<"vnn", 0xE76E, null_frag, v128any, v128any>;
 875: 
 876:   // Nor.
 877:   let isCommutable = 1 in
 878:     def VNO : BinaryVRRc<"vno", 0xE76B, null_frag, v128any, v128any>;
 879:   def : InstAlias<"vnot\t$V1, $V2", (VNO VR128:$V1, VR128:$V2, VR128:$V2), 0>;
 880: 
```
- **EN**: This block declares or refines TableGen records such as `VMLE`, `VMLEB`, `VMLEH`, `VMLEF`, `VMLEG`, `VMO`.
- **CN**: 该代码块声明或细化了 `VMLE`, `VMLEB`, `VMLEH`, `VMLEF`, `VMLEG`, `VMO` 等 TableGen 记录。

### Lines 881-920 / 第 881-920 行
```tablegen
 881:   // Or.
 882:   let isCommutable = 1 in
 883:     def VO : BinaryVRRc<"vo", 0xE76A, null_frag, v128any, v128any>;
 884: 
 885:   // Or with complement.
 886:   let Predicates = [FeatureVectorEnhancements1] in
 887:     def VOC : BinaryVRRc<"voc", 0xE76F, null_frag, v128any, v128any>;
 888: 
 889:   // Population count.
 890:   def VPOPCT : UnaryVRRaGeneric<"vpopct", 0xE750>;
 891:   def : Pat<(v16i8 (z_popcnt VR128:$x)), (VPOPCT VR128:$x, 0)>;
 892:   let Predicates = [FeatureVectorEnhancements1] in {
 893:     def VPOPCTB : UnaryVRRa<"vpopctb", 0xE750, ctpop, v128b, v128b, 0>;
 894:     def VPOPCTH : UnaryVRRa<"vpopcth", 0xE750, ctpop, v128h, v128h, 1>;
 895:     def VPOPCTF : UnaryVRRa<"vpopctf", 0xE750, ctpop, v128f, v128f, 2>;
 896:     def VPOPCTG : UnaryVRRa<"vpopctg", 0xE750, ctpop, v128g, v128g, 3>;
 897:   }
 898: 
 899:   // Remainder.
 900:   let Predicates = [FeatureVectorEnhancements3] in {
 901:     let hasSideEffects = 1 in {
 902:       def VR  : TernaryVRRcIntGeneric<"vr", 0xE7B3>;
 903:       def VRF : TernaryVRRcInt<"vrf", 0xE7B3, null_frag, v128f, v128f, 2>;
 904:       def VRG : TernaryVRRcInt<"vrg", 0xE7B3, null_frag, v128g, v128g, 3>;
 905:       def VRQ : TernaryVRRcInt<"vrq", 0xE7B3, null_frag, v128q, v128q, 4>;
 906:     }
 907:     def : Pat<(v4i32 (srem VR128:$x, VR128:$y)), (VRF VR128:$x, VR128:$y, 0)>;
 908:     def : Pat<(v2i64 (srem VR128:$x, VR128:$y)), (VRG VR128:$x, VR128:$y, 0)>;
 909:     def : Pat<(i128 (srem VR128:$x, VR128:$y)), (VRQ VR128:$x, VR128:$y, 0)>;
 910:   }
 911: 
 912:   // Remainder logical.
 913:   let Predicates = [FeatureVectorEnhancements3] in {
 914:     let hasSideEffects = 1 in {
 915:       def VRL  : TernaryVRRcIntGeneric<"vrl", 0xE7B1>;
 916:       def VRLF : TernaryVRRcInt<"vrlf", 0xE7B1, null_frag, v128f, v128f, 2>;
 917:       def VRLG : TernaryVRRcInt<"vrlg", 0xE7B1, null_frag, v128g, v128g, 3>;
 918:       def VRLQ : TernaryVRRcInt<"vrlq", 0xE7B1, null_frag, v128q, v128q, 4>;
 919:     }
 920:     def : Pat<(v4i32 (urem VR128:$x, VR128:$y)), (VRLF VR128:$x, VR128:$y, 0)>;
```
- **EN**: This block declares or refines TableGen records such as `VO`, `VOC`, `VPOPCT`, `VPOPCTB`, `VPOPCTH`, `VPOPCTF`.
- **CN**: 该代码块声明或细化了 `VO`, `VOC`, `VPOPCT`, `VPOPCTB`, `VPOPCTH`, `VPOPCTF` 等 TableGen 记录。

### Lines 921-960 / 第 921-960 行
```tablegen
 921:     def : Pat<(v2i64 (urem VR128:$x, VR128:$y)), (VRLG VR128:$x, VR128:$y, 0)>;
 922:     def : Pat<(i128 (urem VR128:$x, VR128:$y)), (VRLQ VR128:$x, VR128:$y, 0)>;
 923:   }
 924: 
 925:   // Element rotate left logical (with vector shift amount).
 926:   def VERLLV  : BinaryVRRcGeneric<"verllv", 0xE773>;
 927:   def VERLLVB : BinaryVRRc<"verllvb", 0xE773, rotl, v128b, v128b, 0>;
 928:   def VERLLVH : BinaryVRRc<"verllvh", 0xE773, rotl, v128h, v128h, 1>;
 929:   def VERLLVF : BinaryVRRc<"verllvf", 0xE773, rotl, v128f, v128f, 2>;
 930:   def VERLLVG : BinaryVRRc<"verllvg", 0xE773, rotl, v128g, v128g, 3>;
 931: 
 932:   // Element rotate left logical (with scalar shift amount).
 933:   def VERLL  : BinaryVRSaGeneric<"verll", 0xE733>;
 934:   def VERLLB : BinaryVRSa<"verllb", 0xE733, z_vrotl_by_scalar, v128b, v128b, 0>;
 935:   def VERLLH : BinaryVRSa<"verllh", 0xE733, z_vrotl_by_scalar, v128h, v128h, 1>;
 936:   def VERLLF : BinaryVRSa<"verllf", 0xE733, z_vrotl_by_scalar, v128f, v128f, 2>;
 937:   def VERLLG : BinaryVRSa<"verllg", 0xE733, z_vrotl_by_scalar, v128g, v128g, 3>;
 938: 
 939:   // Element rotate and insert under mask.
 940:   def VERIM  : QuaternaryVRIdGeneric<"verim", 0xE772>;
 941:   def VERIMB : QuaternaryVRId<"verimb", 0xE772, int_s390_verimb, v128b, v128b, 0>;
 942:   def VERIMH : QuaternaryVRId<"verimh", 0xE772, int_s390_verimh, v128h, v128h, 1>;
 943:   def VERIMF : QuaternaryVRId<"verimf", 0xE772, int_s390_verimf, v128f, v128f, 2>;
 944:   def VERIMG : QuaternaryVRId<"verimg", 0xE772, int_s390_verimg, v128g, v128g, 3>;
 945: 
 946:   // Element shift left (with vector shift amount).
 947:   def VESLV  : BinaryVRRcGeneric<"veslv", 0xE770>;
 948:   def VESLVB : BinaryVRRc<"veslvb", 0xE770, z_vshl, v128b, v128b, 0>;
 949:   def VESLVH : BinaryVRRc<"veslvh", 0xE770, z_vshl, v128h, v128h, 1>;
 950:   def VESLVF : BinaryVRRc<"veslvf", 0xE770, z_vshl, v128f, v128f, 2>;
 951:   def VESLVG : BinaryVRRc<"veslvg", 0xE770, z_vshl, v128g, v128g, 3>;
 952: 
 953:   // Element shift left (with scalar shift amount).
 954:   def VESL  : BinaryVRSaGeneric<"vesl", 0xE730>;
 955:   def VESLB : BinaryVRSa<"veslb", 0xE730, z_vshl_by_scalar, v128b, v128b, 0>;
 956:   def VESLH : BinaryVRSa<"veslh", 0xE730, z_vshl_by_scalar, v128h, v128h, 1>;
 957:   def VESLF : BinaryVRSa<"veslf", 0xE730, z_vshl_by_scalar, v128f, v128f, 2>;
 958:   def VESLG : BinaryVRSa<"veslg", 0xE730, z_vshl_by_scalar, v128g, v128g, 3>;
 959: 
 960:   // Element shift right arithmetic (with vector shift amount).
```
- **EN**: This block declares or refines TableGen records such as `VERLLV`, `VERLLVB`, `VERLLVH`, `VERLLVF`, `VERLLVG`, `VERLL`.
- **CN**: 该代码块声明或细化了 `VERLLV`, `VERLLVB`, `VERLLVH`, `VERLLVF`, `VERLLVG`, `VERLL` 等 TableGen 记录。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961:   def VESRAV  : BinaryVRRcGeneric<"vesrav", 0xE77A>;
 962:   def VESRAVB : BinaryVRRc<"vesravb", 0xE77A, z_vsra, v128b, v128b, 0>;
 963:   def VESRAVH : BinaryVRRc<"vesravh", 0xE77A, z_vsra, v128h, v128h, 1>;
 964:   def VESRAVF : BinaryVRRc<"vesravf", 0xE77A, z_vsra, v128f, v128f, 2>;
 965:   def VESRAVG : BinaryVRRc<"vesravg", 0xE77A, z_vsra, v128g, v128g, 3>;
 966: 
 967:   // Element shift right arithmetic (with scalar shift amount).
 968:   def VESRA  : BinaryVRSaGeneric<"vesra", 0xE73A>;
 969:   def VESRAB : BinaryVRSa<"vesrab", 0xE73A, z_vsra_by_scalar, v128b, v128b, 0>;
 970:   def VESRAH : BinaryVRSa<"vesrah", 0xE73A, z_vsra_by_scalar, v128h, v128h, 1>;
 971:   def VESRAF : BinaryVRSa<"vesraf", 0xE73A, z_vsra_by_scalar, v128f, v128f, 2>;
 972:   def VESRAG : BinaryVRSa<"vesrag", 0xE73A, z_vsra_by_scalar, v128g, v128g, 3>;
 973: 
 974:   // Element shift right logical (with vector shift amount).
 975:   def VESRLV  : BinaryVRRcGeneric<"vesrlv", 0xE778>;
 976:   def VESRLVB : BinaryVRRc<"vesrlvb", 0xE778, z_vsrl, v128b, v128b, 0>;
 977:   def VESRLVH : BinaryVRRc<"vesrlvh", 0xE778, z_vsrl, v128h, v128h, 1>;
 978:   def VESRLVF : BinaryVRRc<"vesrlvf", 0xE778, z_vsrl, v128f, v128f, 2>;
 979:   def VESRLVG : BinaryVRRc<"vesrlvg", 0xE778, z_vsrl, v128g, v128g, 3>;
 980: 
 981:   // Element shift right logical (with scalar shift amount).
 982:   def VESRL  : BinaryVRSaGeneric<"vesrl", 0xE738>;
 983:   def VESRLB : BinaryVRSa<"vesrlb", 0xE738, z_vsrl_by_scalar, v128b, v128b, 0>;
 984:   def VESRLH : BinaryVRSa<"vesrlh", 0xE738, z_vsrl_by_scalar, v128h, v128h, 1>;
 985:   def VESRLF : BinaryVRSa<"vesrlf", 0xE738, z_vsrl_by_scalar, v128f, v128f, 2>;
 986:   def VESRLG : BinaryVRSa<"vesrlg", 0xE738, z_vsrl_by_scalar, v128g, v128g, 3>;
 987: 
 988:   // Shift left.
 989:   def VSL : BinaryVRRc<"vsl", 0xE774, int_s390_vsl, v128b, v128b>;
 990: 
 991:   // Shift left by byte.
 992:   def VSLB : BinaryVRRc<"vslb", 0xE775, int_s390_vslb, v128b, v128b>;
 993: 
 994:   // Shift left double by byte.
 995:   def VSLDB : TernaryVRId<"vsldb", 0xE777, z_shl_double, v128b, v128b, 0>;
 996:   def : Pat<(int_s390_vsldb VR128:$x, VR128:$y, imm32zx8_timm:$z),
 997:             (VSLDB VR128:$x, VR128:$y, imm32zx8:$z)>;
 998: 
 999:   // Shift left double by bit.
1000:   let Predicates = [FeatureVectorEnhancements2] in {
```
- **EN**: This block declares or refines TableGen records such as `VESRAV`, `VESRAVB`, `VESRAVH`, `VESRAVF`, `VESRAVG`, `VESRA`.
- **CN**: 该代码块声明或细化了 `VESRAV`, `VESRAVB`, `VESRAVH`, `VESRAVF`, `VESRAVG`, `VESRA` 等 TableGen 记录。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001:     def VSLD : TernaryVRId<"vsld", 0xE786, z_shl_double_bit, v128b, v128b, 0>;
1002:     def : Pat<(int_s390_vsld VR128:$x, VR128:$y, imm32zx8_timm:$z),
1003:               (VSLD VR128:$x, VR128:$y, imm32zx8:$z)>;
1004:   }
1005: 
1006:   // Shift right arithmetic.
1007:   def VSRA : BinaryVRRc<"vsra", 0xE77E, int_s390_vsra, v128b, v128b>;
1008: 
1009:   // Shift right arithmetic by byte.
1010:   def VSRAB : BinaryVRRc<"vsrab", 0xE77F, int_s390_vsrab, v128b, v128b>;
1011: 
1012:   // Shift right logical.
1013:   def VSRL : BinaryVRRc<"vsrl", 0xE77C, int_s390_vsrl, v128b, v128b>;
1014: 
1015:   // Shift right logical by byte.
1016:   def VSRLB : BinaryVRRc<"vsrlb", 0xE77D, int_s390_vsrlb, v128b, v128b>;
1017: 
1018:   // Shift right double by bit.
1019:   let Predicates = [FeatureVectorEnhancements2] in {
1020:     def VSRD : TernaryVRId<"vsrd", 0xE787, z_shr_double_bit, v128b, v128b, 0>;
1021:     def : Pat<(int_s390_vsrd VR128:$x, VR128:$y, imm32zx8_timm:$z),
1022:               (VSRD VR128:$x, VR128:$y, imm32zx8:$z)>;
1023:   }
1024: 
1025:   // Subtract.
1026:   def VS  : BinaryVRRcGeneric<"vs", 0xE7F7>;
1027:   def VSB : BinaryVRRc<"vsb", 0xE7F7, sub, v128b, v128b, 0>;
1028:   def VSH : BinaryVRRc<"vsh", 0xE7F7, sub, v128h, v128h, 1>;
1029:   def VSF : BinaryVRRc<"vsf", 0xE7F7, sub, v128f, v128f, 2>;
1030:   def VSG : BinaryVRRc<"vsg", 0xE7F7, sub, v128g, v128g, 3>;
1031:   def VSQ : BinaryVRRc<"vsq", 0xE7F7, sub, v128q, v128q, 4>;
1032: 
1033:   // Subtract compute borrow indication.
1034:   def VSCBI  : BinaryVRRcGeneric<"vscbi", 0xE7F5>;
1035:   def VSCBIB : BinaryVRRc<"vscbib", 0xE7F5, z_vscbi, v128b, v128b, 0>;
1036:   def VSCBIH : BinaryVRRc<"vscbih", 0xE7F5, z_vscbi, v128h, v128h, 1>;
1037:   def VSCBIF : BinaryVRRc<"vscbif", 0xE7F5, z_vscbi, v128f, v128f, 2>;
1038:   def VSCBIG : BinaryVRRc<"vscbig", 0xE7F5, z_vscbi, v128g, v128g, 3>;
1039:   def VSCBIQ : BinaryVRRc<"vscbiq", 0xE7F5, z_vscbi, v128q, v128q, 4>;
1040: 
```
- **EN**: This block declares or refines TableGen records such as `VSLD`, `VSRA`, `VSRAB`, `VSRL`, `VSRLB`, `VSRD`.
- **CN**: 该代码块声明或细化了 `VSLD`, `VSRA`, `VSRAB`, `VSRL`, `VSRLB`, `VSRD` 等 TableGen 记录。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041:   // Subtract with borrow indication.
1042:   def VSBI  : TernaryVRRdGeneric<"vsbi", 0xE7BF>;
1043:   def VSBIQ : TernaryVRRd<"vsbiq", 0xE7BF, z_vsbi, v128q, v128q, 4>;
1044: 
1045:   // Subtract with borrow compute borrow indication.
1046:   def VSBCBI  : TernaryVRRdGeneric<"vsbcbi", 0xE7BD>;
1047:   def VSBCBIQ : TernaryVRRd<"vsbcbiq", 0xE7BD, z_vsbcbi, v128q, v128q, 4>;
1048: 
1049:   // Sum across doubleword.
1050:   def VSUMG  : BinaryVRRcGeneric<"vsumg", 0xE765>;
1051:   def VSUMGH : BinaryVRRc<"vsumgh", 0xE765, z_vsum, v128g, v128h, 1>;
1052:   def VSUMGF : BinaryVRRc<"vsumgf", 0xE765, z_vsum, v128g, v128f, 2>;
1053: 
1054:   // Sum across quadword.
1055:   def VSUMQ  : BinaryVRRcGeneric<"vsumq", 0xE767>;
1056:   def VSUMQF : BinaryVRRc<"vsumqf", 0xE767, z_vsum, v128q, v128f, 2>;
1057:   def VSUMQG : BinaryVRRc<"vsumqg", 0xE767, z_vsum, v128q, v128g, 3>;
1058: 
1059:   // Sum across word.
1060:   def VSUM  : BinaryVRRcGeneric<"vsum", 0xE764>;
1061:   def VSUMB : BinaryVRRc<"vsumb", 0xE764, z_vsum, v128f, v128b, 0>;
1062:   def VSUMH : BinaryVRRc<"vsumh", 0xE764, z_vsum, v128f, v128h, 1>;
1063: }
1064: 
1065: // Instantiate the bitwise ops for type TYPE.
1066: multiclass BitwiseVectorOps<ValueType type, SDPatternOperator not_op> {
1067:   let Predicates = [FeatureVector] in {
1068:     def : Pat<(type (and VR128:$x, VR128:$y)), (VN VR128:$x, VR128:$y)>;
1069:     def : Pat<(type (and VR128:$x, (not_op VR128:$y))),
1070:               (VNC VR128:$x, VR128:$y)>;
1071:     def : Pat<(type (or VR128:$x, VR128:$y)), (VO VR128:$x, VR128:$y)>;
1072:     def : Pat<(type (xor VR128:$x, VR128:$y)), (VX VR128:$x, VR128:$y)>;
1073:     def : Pat<(type (or (and VR128:$x, VR128:$z),
1074:                         (and VR128:$y, (not_op VR128:$z)))),
1075:               (VSEL VR128:$x, VR128:$y, VR128:$z)>;
1076:     def : Pat<(type (not_op (or VR128:$x, VR128:$y))),
1077:               (VNO VR128:$x, VR128:$y)>;
1078:     def : Pat<(type (not_op VR128:$x)), (VNO VR128:$x, VR128:$x)>;
1079:   }
1080:   let Predicates = [FeatureVectorEnhancements1] in {
```
- **EN**: This block declares or refines TableGen records such as `VSBI`, `VSBIQ`, `VSBCBI`, `VSBCBIQ`, `VSUMG`, `VSUMGH`.
- **CN**: 该代码块声明或细化了 `VSBI`, `VSBIQ`, `VSBCBI`, `VSBCBIQ`, `VSUMG`, `VSUMGH` 等 TableGen 记录。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081:     def : Pat<(type (not_op (xor VR128:$x, VR128:$y))),
1082:               (VNX VR128:$x, VR128:$y)>;
1083:     def : Pat<(type (not_op (and VR128:$x, VR128:$y))),
1084:               (VNN VR128:$x, VR128:$y)>;
1085:     def : Pat<(type (or VR128:$x, (not_op VR128:$y))),
1086:               (VOC VR128:$x, VR128:$y)>;
1087:   }
1088:   let Predicates = [FeatureVectorEnhancements3] in {
1089:     def : Pat<(type (and VR128:$x, (and VR128:$y, VR128:$z))),
1090:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 1)>;
1091:     def : Pat<(type (and (not_op VR128:$z), (and VR128:$x, VR128:$y))),
1092:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 2)>;
1093:     def : Pat<(type (and VR128:$x, (xor VR128:$y, VR128:$z))),
1094:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 6)>;
1095:     def : Pat<(type (and VR128:$x, (or VR128:$y, VR128:$z))),
1096:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 7)>;
1097:     def : Pat<(type (and VR128:$x, (not_op (or VR128:$y, VR128:$z)))),
1098:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 8)>;
1099:     def : Pat<(type (and VR128:$x, (not_op (xor VR128:$y, VR128:$z)))),
1100:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 9)>;
1101:     def : Pat<(type (and VR128:$x, (or VR128:$y, (not_op VR128:$z)))),
1102:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 11)>;
1103:     def : Pat<(type (and VR128:$x, (not_op (and VR128:$y, VR128:$z)))),
1104:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 14)>;
1105:     def : Pat<(type (and (or VR128:$x, VR128:$y), (xor VR128:$z, (and VR128:$x, VR128:$y)))),
1106:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 22)>;
1107:     def : Pat<(type (or (and VR128:$x, VR128:$y), (and VR128:$z, (or VR128:$x, VR128:$y)))),
1108:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 23)>;
1109:     def : Pat<(type (and (xor VR128:$x, VR128:$y), (xor VR128:$x, VR128:$z))),
1110:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 24)>;
1111:     def : Pat<(type (and (or VR128:$x, VR128:$y), (not_op (xor VR128:$y, VR128:$z)))),
1112:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 25)>;
1113:     def : Pat<(type (and (or VR128:$x, VR128:$y), (xor VR128:$x, VR128:$z))),
1114:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 26)>;
1115:     def : Pat<(type (and (or VR128:$x, VR128:$z), (or VR128:$y, (not_op VR128:$z)))),
1116:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 27)>;
1117:     def : Pat<(type (xor VR128:$x, (and VR128:$y, VR128:$z))),
1118:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 30)>;
1119:     def : Pat<(type (or VR128:$x, (and VR128:$y, VR128:$z))),
1120:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 31)>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121:     def : Pat<(type (and (not_op VR128:$z), (xor VR128:$x, VR128:$y))),
1122:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 40)>;
1123:     def : Pat<(type (and (or VR128:$x, VR128:$y), (not_op (xor VR128:$z, (and VR128:$x, VR128:$y))))),
1124:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 41)>;
1125:     def : Pat<(type (and (not_op VR128:$z), (or VR128:$x, VR128:$y))),
1126:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 42)>;
1127:     def : Pat<(type (or (and VR128:$x, VR128:$y), (and (not_op VR128:$z), (or VR128:$x, VR128:$y)))),
1128:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 43)>;
1129:     def : Pat<(type (xor VR128:$y, (or VR128:$x, (and VR128:$y, VR128:$z)))),
1130:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 44)>;
1131:     def : Pat<(type (xor VR128:$x, (and VR128:$y, (not_op VR128:$z)))),
1132:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 45)>;
1133:     def : Pat<(type (and (or VR128:$x, VR128:$y), (not_op (and VR128:$y, VR128:$z)))),
1134:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 46)>;
1135:     def : Pat<(type (or VR128:$x, (and VR128:$y, (not_op VR128:$z)))),
1136:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 47)>;
1137:     def : Pat<(type (or (xor VR128:$x, VR128:$y), (and VR128:$x, VR128:$z))),
1138:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 61)>;
1139:     def : Pat<(type (or (xor VR128:$x, VR128:$y), (and VR128:$x, (not_op VR128:$z)))),
1140:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 62)>;
1141:     def : Pat<(type (xor (or VR128:$x, VR128:$y), (or VR128:$z, (and VR128:$x, VR128:$y)))),
1142:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 104)>;
1143:     def : Pat<(type (xor VR128:$x, (xor VR128:$y, VR128:$z))),
1144:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 105)>;
1145:     def : Pat<(type (xor VR128:$z, (or VR128:$x, VR128:$y))),
1146:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 106)>;
1147:     def : Pat<(type (or (and VR128:$x, VR128:$y), (xor VR128:$z, (or VR128:$x, VR128:$y)))),
1148:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 107)>;
1149:     def : Pat<(type (or (xor VR128:$y, VR128:$z), (and VR128:$x, (not_op VR128:$y)))),
1150:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 110)>;
1151:     def : Pat<(type (or VR128:$x, (xor VR128:$y, VR128:$z))),
1152:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 111)>;
1153:     def : Pat<(type (or (xor VR128:$x, VR128:$y), (xor VR128:$x, VR128:$z))),
1154:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 126)>;
1155:     def : Pat<(type (or VR128:$x, (or VR128:$y, VR128:$z))),
1156:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 127)>;
1157:     def : Pat<(type (not_op (or VR128:$x, (or VR128:$y, VR128:$z)))),
1158:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 128)>;
1159:     def : Pat<(type (not_op (or (xor VR128:$x, VR128:$y), (xor VR128:$x, VR128:$z)))),
1160:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 129)>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161:     def : Pat<(type (not_op (or VR128:$z, (xor VR128:$x, VR128:$y)))),
1162:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 130)>;
1163:     def : Pat<(type (and (not_op (xor VR128:$x, VR128:$y)), (or VR128:$x, (not_op VR128:$z)))),
1164:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 131)>;
1165:     def : Pat<(type (xor (or VR128:$y, VR128:$z), (or (not_op VR128:$x), (and VR128:$y, VR128:$z)))),
1166:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 134)>;
1167:     def : Pat<(type (not_op (xor VR128:$x, (or VR128:$y, VR128:$z)))),
1168:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 135)>;
1169:     def : Pat<(type (or (not_op (or VR128:$y, VR128:$z)), (and VR128:$x, (and VR128:$y, VR128:$z)))),
1170:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 137)>;
1171:     def : Pat<(type (and (not_op VR128:$z), (or VR128:$x, (not_op VR128:$y)))),
1172:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 138)>;
1173:     def : Pat<(type (or (and VR128:$x, VR128:$y), (not_op (or VR128:$y, VR128:$z)))),
1174:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 139)>;
1175:     def : Pat<(type (or (not_op (or VR128:$y, VR128:$z)), (and VR128:$x, (xor VR128:$y, VR128:$z)))),
1176:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 142)>;
1177:     def : Pat<(type (or VR128:$x, (not_op (or VR128:$y, VR128:$z)))),
1178:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 143)>;
1179:     def : Pat<(type (not_op (xor VR128:$x, (xor VR128:$y, VR128:$z)))),
1180:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 150)>;
1181:     def : Pat<(type (or (and VR128:$x, VR128:$y), (not_op (xor VR128:$z, (or VR128:$x, VR128:$y))))),
1182:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 151)>;
1183:     def : Pat<(type (not_op (or (and VR128:$x, VR128:$y), (xor VR128:$y, VR128:$z)))),
1184:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 152)>;
1185:     def : Pat<(type (xor VR128:$z, (or VR128:$x, (not_op VR128:$y)))),
1186:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 154)>;
1187:     def : Pat<(type (or (and VR128:$x, VR128:$y), (not_op (xor VR128:$y, VR128:$z)))),
1188:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 155)>;
1189:     def : Pat<(type (or (not_op (or VR128:$y, VR128:$z)), (xor VR128:$x, (and VR128:$y, VR128:$z)))),
1190:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 158)>;
1191:     def : Pat<(type (or VR128:$x, (not_op (xor VR128:$y, VR128:$z)))),
1192:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 159)>;
1193:     def : Pat<(type (not_op (or VR128:$z, (and VR128:$x, VR128:$y)))),
1194:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 168)>;
1195:     def : Pat<(type (not_op (xor VR128:$z, (and VR128:$x, VR128:$y)))),
1196:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 169)>;
1197:     def : Pat<(type (or (not_op VR128:$z), (and VR128:$x, VR128:$y))),
1198:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 171)>;
1199:     def : Pat<(type (and (not_op (and VR128:$x, VR128:$y)), (or VR128:$x, (not_op VR128:$z)))),
1200:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 172)>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201:     def : Pat<(type (not_op (and (xor VR128:$x, VR128:$z), (or VR128:$y, VR128:$z)))),
1202:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 173)>;
1203:     def : Pat<(type (or (not_op VR128:$z), (and VR128:$x, (not_op VR128:$y)))),
1204:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 174)>;
1205:     def : Pat<(type (or (xor VR128:$x, VR128:$y), (not_op (or VR128:$x, VR128:$z)))),
1206:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 188)>;
1207:     def : Pat<(type (not_op (and (xor VR128:$x, VR128:$z), (xor VR128:$y, VR128:$z)))),
1208:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 189)>;
1209:     def : Pat<(type (or (not_op VR128:$z), (xor VR128:$x, VR128:$y))),
1210:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 190)>;
1211:     def : Pat<(type (or (not_op VR128:$z), (or VR128:$x, VR128:$y))),
1212:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 191)>;
1213:     def : Pat<(type (or (not_op (or VR128:$x, VR128:$y)), (and (not_op VR128:$z), (xor VR128:$x, VR128:$y)))),
1214:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 232)>;
1215:     def : Pat<(type (xor (not_op (and VR128:$x, VR128:$y)), (and VR128:$z, (or VR128:$x, VR128:$y)))),
1216:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 233)>;
1217:     def : Pat<(type (not_op (and VR128:$z, (or VR128:$x, VR128:$y)))),
1218:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 234)>;
1219:     def : Pat<(type (not_op (and VR128:$z, (xor VR128:$x, VR128:$y)))),
1220:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 235)>;
1221:     def : Pat<(type (or VR128:$x, (not_op (and VR128:$y, VR128:$z)))),
1222:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 239)>;
1223:     def : Pat<(type (not_op (and VR128:$x, (and VR128:$y, VR128:$z)))),
1224:               (VEVAL VR128:$x, VR128:$y, VR128:$z, 254)>;
1225:   }
1226: }
1227: 
1228: defm : BitwiseVectorOps<v16i8, z_vnot>;
1229: defm : BitwiseVectorOps<v8i16, z_vnot>;
1230: defm : BitwiseVectorOps<v4i32, z_vnot>;
1231: defm : BitwiseVectorOps<v2i64, z_vnot>;
1232: defm : BitwiseVectorOps<i128, not>;
1233: 
1234: // Instantiate additional patterns for absolute-related expressions on
1235: // type TYPE.  LC is the negate instruction for TYPE and LP is the absolute
1236: // instruction.
1237: multiclass IntegerAbsoluteVectorOps<ValueType type, Instruction lc,
1238:                                     Instruction lp, int shift> {
1239:   let Predicates = [FeatureVector] in {
1240:     def : Pat<(type (vselect (type (z_vicmph_zero VR128:$x)),
```
- **EN**: This block declares or refines TableGen records such as `IntegerAbsoluteVectorOps`.
- **CN**: 该代码块声明或细化了 `IntegerAbsoluteVectorOps` 等 TableGen 记录。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241:                              (z_vneg VR128:$x), VR128:$x)),
1242:               (lc (lp VR128:$x))>;
1243:     def : Pat<(type (vselect (type (z_vnot (z_vicmph_zero VR128:$x))),
1244:                              VR128:$x, (z_vneg VR128:$x))),
1245:               (lc (lp VR128:$x))>;
1246:     def : Pat<(type (vselect (type (z_vicmpl_zero VR128:$x)),
1247:                              VR128:$x, (z_vneg VR128:$x))),
1248:               (lc (lp VR128:$x))>;
1249:     def : Pat<(type (vselect (type (z_vnot (z_vicmpl_zero VR128:$x))),
1250:                              (z_vneg VR128:$x), VR128:$x)),
1251:               (lc (lp VR128:$x))>;
1252:     def : Pat<(type (or (and (z_vsra_by_scalar VR128:$x, (i32 shift)),
1253:                              (z_vneg VR128:$x)),
1254:                         (and (z_vnot (z_vsra_by_scalar VR128:$x, (i32 shift))),
1255:                              VR128:$x))),
1256:               (lp VR128:$x)>;
1257:     def : Pat<(type (or (and (z_vsra_by_scalar VR128:$x, (i32 shift)),
1258:                              VR128:$x),
1259:                         (and (z_vnot (z_vsra_by_scalar VR128:$x, (i32 shift))),
1260:                              (z_vneg VR128:$x)))),
1261:               (lc (lp VR128:$x))>;
1262:   }
1263: }
1264: 
1265: defm : IntegerAbsoluteVectorOps<v16i8, VLCB, VLPB, 7>;
1266: defm : IntegerAbsoluteVectorOps<v8i16, VLCH, VLPH, 15>;
1267: defm : IntegerAbsoluteVectorOps<v4i32, VLCF, VLPF, 31>;
1268: defm : IntegerAbsoluteVectorOps<v2i64, VLCG, VLPG, 63>;
1269: 
1270: // Instantiate packs/packu: recognize a saturating truncation and convert
1271: // into the corresponding packs/packu instruction.
1272: multiclass SignedSaturatingTruncate<ValueType input, ValueType output,
1273:                                     Instruction packs> {
1274:   def : Pat<
1275:     (output (z_pack
1276:       (smin (smax (input VR128:$a), ssat_trunc_min_vec), ssat_trunc_max_vec),
1277:       (smin (smax (input VR128:$b), ssat_trunc_min_vec), ssat_trunc_max_vec)
1278:     )),
1279:     (packs VR128:$a, VR128:$b)
1280:   >;
```
- **EN**: This block declares or refines TableGen records such as `SignedSaturatingTruncate`.
- **CN**: 该代码块声明或细化了 `SignedSaturatingTruncate` 等 TableGen 记录。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281: 
1282:   def : Pat<
1283:     (output (z_pack
1284:       (smax (smin (input VR128:$a), ssat_trunc_max_vec), ssat_trunc_min_vec),
1285:       (smax (smin (input VR128:$b), ssat_trunc_max_vec), ssat_trunc_min_vec)
1286:     )),
1287:     (packs VR128:$a, VR128:$b)
1288:   >;
1289: }
1290: 
1291: defm : SignedSaturatingTruncate<v8i16, v16i8, VPKSH>;
1292: defm : SignedSaturatingTruncate<v4i32, v8i16, VPKSF>;
1293: defm : SignedSaturatingTruncate<v2i64, v4i32, VPKSG>;
1294: 
1295: multiclass UnsignedSaturatingTruncate<ValueType input, ValueType output,
1296:                                       Instruction packu> {
1297:   def : Pat<
1298:     (output (z_pack
1299:       (umin (input VR128:$a), usat_trunc_max_vec),
1300:       (umin (input VR128:$b), usat_trunc_max_vec)
1301:     )),
1302:     (packu VR128:$a, VR128:$b)
1303:   >;
1304: }
1305: 
1306: defm : UnsignedSaturatingTruncate<v8i16, v16i8, VPKLSH>;
1307: defm : UnsignedSaturatingTruncate<v4i32, v8i16, VPKLSF>;
1308: defm : UnsignedSaturatingTruncate<v2i64, v4i32, VPKLSG>;
1309: 
1310: // Instantiate comparison patterns to recognize VACC/VSCBI for TYPE.
1311: multiclass IntegerComputeCarryOrBorrow<ValueType type,
1312:                                        Instruction vacc, Instruction vscbi> {
1313:   let Predicates = [FeatureVector] in {
1314:     def : Pat<(z_vzext1 (type (z_vicmphl VR128:$x, (add VR128:$x, VR128:$y)))),
1315:               (vacc VR128:$x, VR128:$y)>;
1316:     def : Pat<(z_vzext1 (type (z_vicmphl VR128:$y, (add VR128:$x, VR128:$y)))),
1317:               (vacc VR128:$x, VR128:$y)>;
1318:     def : Pat<(z_vzext1 (z_vnot (type (z_vicmphl VR128:$y, VR128:$x)))),
1319:               (vscbi VR128:$x, VR128:$y)>;
1320:   }
```
- **EN**: This block declares or refines TableGen records such as `UnsignedSaturatingTruncate`, `IntegerComputeCarryOrBorrow`.
- **CN**: 该代码块声明或细化了 `UnsignedSaturatingTruncate`, `IntegerComputeCarryOrBorrow` 等 TableGen 记录。

### Lines 1321-1360 / 第 1321-1360 行
```tablegen
1321: }
1322: defm : IntegerComputeCarryOrBorrow<v16i8, VACCB, VSCBIB>;
1323: defm : IntegerComputeCarryOrBorrow<v8i16, VACCH, VSCBIH>;
1324: defm : IntegerComputeCarryOrBorrow<v4i32, VACCF, VSCBIF>;
1325: defm : IntegerComputeCarryOrBorrow<v2i64, VACCG, VSCBIG>;
1326: 
1327: // Instantiate full-vector shifts.
1328: multiclass FullVectorShiftOps<SDPatternOperator shift,
1329:                               Instruction sbit, Instruction sbyte> {
1330:   let Predicates = [FeatureVector] in {
1331:     def : Pat<(shift (i128 VR128:$x), imm32nobytes:$amt),
1332:               (sbit VR128:$x, (VREPIB (UIMM8 imm:$amt)))>;
1333:     def : Pat<(shift (i128 VR128:$x), imm32nobits:$amt),
1334:               (sbyte VR128:$x, (VREPIB (UIMM8 imm:$amt)))>;
1335:     def : Pat<(shift (i128 VR128:$x), imm32:$amt),
1336:               (sbit (sbyte VR128:$x, (VREPIB (UIMM8 imm:$amt))),
1337:                     (VREPIB (UIMM8 imm:$amt)))>;
1338:     def : Pat<(shift (i128 VR128:$x), GR32:$amt),
1339:               (sbit (sbyte VR128:$x, (VREPB (VLVGP32 GR32:$amt, GR32:$amt), 15)),
1340:                     (VREPB (VLVGP32 GR32:$amt, GR32:$amt), 15))>;
1341:   }
1342: }
1343: defm : FullVectorShiftOps<vshiftop<shl>, VSL, VSLB>;
1344: defm : FullVectorShiftOps<vshiftop<srl>, VSRL, VSRLB>;
1345: defm : FullVectorShiftOps<vshiftop<sra>, VSRA, VSRAB>;
1346: 
1347: //===----------------------------------------------------------------------===//
1348: // Integer comparison
1349: //===----------------------------------------------------------------------===//
1350: 
1351: let Predicates = [FeatureVector] in {
1352:   // Element compare.
1353:   let Defs = [CC] in {
1354:     def VEC  : CompareVRRaGeneric<"vec", 0xE7DB>;
1355:     def VECB : CompareVRRa<"vecb", 0xE7DB, null_frag, v128b, 0>;
1356:     def VECH : CompareVRRa<"vech", 0xE7DB, null_frag, v128h, 1>;
1357:     def VECF : CompareVRRa<"vecf", 0xE7DB, null_frag, v128f, 2>;
1358:     def VECG : CompareVRRa<"vecg", 0xE7DB, null_frag, v128g, 3>;
1359:     let Predicates = [FeatureVectorEnhancements3] in
1360:       def VECQ : CompareVRRa<"vecq", 0xE7DB, z_scmp, v128q, 4>;
```
- **EN**: This block declares or refines TableGen records such as `FullVectorShiftOps`, `VEC`, `VECB`, `VECH`, `VECF`, `VECG`.
- **CN**: 该代码块声明或细化了 `FullVectorShiftOps`, `VEC`, `VECB`, `VECH`, `VECF`, `VECG` 等 TableGen 记录。

### Lines 1361-1400 / 第 1361-1400 行
```tablegen
1361:   }
1362: 
1363:   // Element compare logical.
1364:   let Defs = [CC] in {
1365:     def VECL  : CompareVRRaGeneric<"vecl", 0xE7D9>;
1366:     def VECLB : CompareVRRa<"veclb", 0xE7D9, null_frag, v128b, 0>;
1367:     def VECLH : CompareVRRa<"veclh", 0xE7D9, null_frag, v128h, 1>;
1368:     def VECLF : CompareVRRa<"veclf", 0xE7D9, null_frag, v128f, 2>;
1369:     def VECLG : CompareVRRa<"veclg", 0xE7D9, null_frag, v128g, 3>;
1370:     let Predicates = [FeatureVectorEnhancements3] in
1371:       def VECLQ : CompareVRRa<"veclq", 0xE7D9, z_ucmp, v128q, 4>;
1372:   }
1373: 
1374:   // Compare equal.
1375:   def  VCEQ  : BinaryVRRbSPairGeneric<"vceq", 0xE7F8>;
1376:   defm VCEQB : BinaryVRRbSPair<"vceqb", 0xE7F8, z_vicmpe, z_vicmpes,
1377:                                v128b, v128b, 0>;
1378:   defm VCEQH : BinaryVRRbSPair<"vceqh", 0xE7F8, z_vicmpe, z_vicmpes,
1379:                                v128h, v128h, 1>;
1380:   defm VCEQF : BinaryVRRbSPair<"vceqf", 0xE7F8, z_vicmpe, z_vicmpes,
1381:                                v128f, v128f, 2>;
1382:   defm VCEQG : BinaryVRRbSPair<"vceqg", 0xE7F8, z_vicmpe, z_vicmpes,
1383:                                v128g, v128g, 3>;
1384:   let Predicates = [FeatureVectorEnhancements3] in
1385:     defm VCEQQ : BinaryVRRbSPair<"vceqq", 0xE7F8, z_vicmpe, z_vicmpes,
1386:                                  v128q, v128q, 4>;
1387: 
1388:   // Compare high.
1389:   def  VCH  : BinaryVRRbSPairGeneric<"vch", 0xE7FB>;
1390:   defm VCHB : BinaryVRRbSPair<"vchb", 0xE7FB, z_vicmph, z_vicmphs,
1391:                               v128b, v128b, 0>;
1392:   defm VCHH : BinaryVRRbSPair<"vchh", 0xE7FB, z_vicmph, z_vicmphs,
1393:                               v128h, v128h, 1>;
1394:   defm VCHF : BinaryVRRbSPair<"vchf", 0xE7FB, z_vicmph, z_vicmphs,
1395:                               v128f, v128f, 2>;
1396:   defm VCHG : BinaryVRRbSPair<"vchg", 0xE7FB, z_vicmph, z_vicmphs,
1397:                               v128g, v128g, 3>;
1398:   let Predicates = [FeatureVectorEnhancements3] in
1399:     defm VCHQ : BinaryVRRbSPair<"vchq", 0xE7FB, z_vicmph, z_vicmphs,
1400:                                 v128q, v128q, 4>;
```
- **EN**: This block declares or refines TableGen records such as `VECL`, `VECLB`, `VECLH`, `VECLF`, `VECLG`, `VECLQ`.
- **CN**: 该代码块声明或细化了 `VECL`, `VECLB`, `VECLH`, `VECLF`, `VECLG`, `VECLQ` 等 TableGen 记录。

### Lines 1401-1440 / 第 1401-1440 行
```tablegen
1401: 
1402:   // Compare high logical.
1403:   def  VCHL  : BinaryVRRbSPairGeneric<"vchl", 0xE7F9>;
1404:   defm VCHLB : BinaryVRRbSPair<"vchlb", 0xE7F9, z_vicmphl, z_vicmphls,
1405:                                v128b, v128b, 0>;
1406:   defm VCHLH : BinaryVRRbSPair<"vchlh", 0xE7F9, z_vicmphl, z_vicmphls,
1407:                                v128h, v128h, 1>;
1408:   defm VCHLF : BinaryVRRbSPair<"vchlf", 0xE7F9, z_vicmphl, z_vicmphls,
1409:                                v128f, v128f, 2>;
1410:   defm VCHLG : BinaryVRRbSPair<"vchlg", 0xE7F9, z_vicmphl, z_vicmphls,
1411:                                v128g, v128g, 3>;
1412:   let Predicates = [FeatureVectorEnhancements3] in
1413:     defm VCHLQ : BinaryVRRbSPair<"vchlq", 0xE7F9, z_vicmphl, z_vicmphls,
1414:                                  v128q, v128q, 4>;
1415: 
1416:   // Test under mask.
1417:   let Defs = [CC] in
1418:     def VTM : CompareVRRa<"vtm", 0xE7D8, z_vtm, v128b, 0>;
1419: }
1420: 
1421: //===----------------------------------------------------------------------===//
1422: // Floating-point arithmetic
1423: //===----------------------------------------------------------------------===//
1424: 
1425: // See comments in SystemZInstrFP.td for the suppression flags and
1426: // rounding modes.
1427: multiclass VectorRounding<Instruction insn, TypedReg tr> {
1428:   def : FPConversion<insn, any_frint,      tr, tr, 0, 0>;
1429:   def : FPConversion<insn, any_fnearbyint, tr, tr, 4, 0>;
1430:   def : FPConversion<insn, any_ffloor,     tr, tr, 4, 7>;
1431:   def : FPConversion<insn, any_fceil,      tr, tr, 4, 6>;
1432:   def : FPConversion<insn, any_ftrunc,     tr, tr, 4, 5>;
1433:   def : FPConversion<insn, any_froundeven, tr, tr, 4, 4>;
1434:   def : FPConversion<insn, any_fround,     tr, tr, 4, 1>;
1435: }
1436: 
1437: let Predicates = [FeatureVector] in {
1438:   // Add.
1439:   let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1 in {
1440:     def VFA   : BinaryVRRcFloatGeneric<"vfa", 0xE7E3>;
```
- **EN**: This block declares or refines TableGen records such as `VCHL`, `VCHLB`, `VCHLH`, `VCHLF`, `VCHLG`, `VCHLQ`.
- **CN**: 该代码块声明或细化了 `VCHL`, `VCHLB`, `VCHLH`, `VCHLF`, `VCHLG`, `VCHLQ` 等 TableGen 记录。

### Lines 1441-1480 / 第 1441-1480 行
```tablegen
1441:     def VFADB : BinaryVRRc<"vfadb", 0xE7E3, any_fadd, v128db, v128db, 3, 0>;
1442:     def WFADB : BinaryVRRc<"wfadb", 0xE7E3, any_fadd, v64db, v64db, 3, 8, 0,
1443:                            "adbr">;
1444:     let Predicates = [FeatureVectorEnhancements1] in {
1445:       def VFASB : BinaryVRRc<"vfasb", 0xE7E3, any_fadd, v128sb, v128sb, 2, 0>;
1446:       def WFASB : BinaryVRRc<"wfasb", 0xE7E3, any_fadd, v32sb, v32sb, 2, 8, 0,
1447:                              "aebr">;
1448:       def WFAXB : BinaryVRRc<"wfaxb", 0xE7E3, any_fadd, v128xb, v128xb, 4, 8>;
1449:     }
1450:   }
1451: 
1452:   // Convert from fixed.
1453:   let Uses = [FPC], mayRaiseFPException = 1 in {
1454:     def VCDG  : TernaryVRRaFloatGeneric<"vcdg", 0xE7C3>;
1455:     def VCDGB : TernaryVRRa<"vcdgb", 0xE7C3, null_frag, v128db, v128g, 3, 0>;
1456:     def WCDGB : TernaryVRRa<"wcdgb", 0xE7C3, null_frag, v64db, v64g, 3, 8>;
1457:   }
1458:   def : FPConversion<VCDGB, any_sint_to_fp, v128db, v128g, 0, 0>;
1459:   let Predicates = [FeatureVectorEnhancements2] in {
1460:     let Uses = [FPC], mayRaiseFPException = 1 in {
1461:       let isAsmParserOnly = 1 in
1462:         def VCFPS  : TernaryVRRaFloatGeneric<"vcfps", 0xE7C3>;
1463:       def VCEFB : TernaryVRRa<"vcefb", 0xE7C3, null_frag, v128sb, v128g, 2, 0>;
1464:       def WCEFB : TernaryVRRa<"wcefb", 0xE7C3, null_frag, v32sb, v32f, 2, 8>;
1465:     }
1466:     def : FPConversion<VCEFB, any_sint_to_fp, v128sb, v128f, 0, 0>;
1467:   }
1468: 
1469:   // Convert from logical.
1470:   let Uses = [FPC], mayRaiseFPException = 1 in {
1471:     def VCDLG  : TernaryVRRaFloatGeneric<"vcdlg", 0xE7C1>;
1472:     def VCDLGB : TernaryVRRa<"vcdlgb", 0xE7C1, null_frag, v128db, v128g, 3, 0>;
1473:     def WCDLGB : TernaryVRRa<"wcdlgb", 0xE7C1, null_frag, v64db, v64g, 3, 8>;
1474:   }
1475:   def : FPConversion<VCDLGB, any_uint_to_fp, v128db, v128g, 0, 0>;
1476:   let Predicates = [FeatureVectorEnhancements2] in {
1477:     let Uses = [FPC], mayRaiseFPException = 1 in {
1478:       let isAsmParserOnly = 1 in
1479:         def VCFPL  : TernaryVRRaFloatGeneric<"vcfpl", 0xE7C1>;
1480:       def VCELFB : TernaryVRRa<"vcelfb", 0xE7C1, null_frag, v128sb, v128g, 2, 0>;
```
- **EN**: This block declares or refines TableGen records such as `VFADB`, `WFADB`, `VFASB`, `WFASB`, `WFAXB`, `VCDG`.
- **CN**: 该代码块声明或细化了 `VFADB`, `WFADB`, `VFASB`, `WFASB`, `WFAXB`, `VCDG` 等 TableGen 记录。

### Lines 1481-1520 / 第 1481-1520 行
```tablegen
1481:       def WCELFB : TernaryVRRa<"wcelfb", 0xE7C1, null_frag, v32sb, v32f, 2, 8>;
1482:     }
1483:     def : FPConversion<VCELFB, any_uint_to_fp, v128sb, v128f, 0, 0>;
1484:   }
1485: 
1486:   // Convert to fixed.
1487:   let Uses = [FPC], mayRaiseFPException = 1 in {
1488:     def VCGD  : TernaryVRRaFloatGeneric<"vcgd", 0xE7C2>;
1489:     def VCGDB : TernaryVRRa<"vcgdb", 0xE7C2, null_frag, v128g, v128db, 3, 0>;
1490:     def WCGDB : TernaryVRRa<"wcgdb", 0xE7C2, null_frag, v64g, v64db, 3, 8>;
1491:   }
1492:   // Rounding mode should agree with SystemZInstrFP.td.
1493:   def : FPConversion<VCGDB, any_fp_to_sint, v128g, v128db, 0, 5>;
1494:   let Predicates = [FeatureVectorEnhancements2] in {
1495:     let Uses = [FPC], mayRaiseFPException = 1 in {
1496:       let isAsmParserOnly = 1 in
1497:         def VCSFP  : TernaryVRRaFloatGeneric<"vcsfp", 0xE7C2>;
1498:       def VCFEB : TernaryVRRa<"vcfeb", 0xE7C2, null_frag, v128sb, v128g, 2, 0>;
1499:       def WCFEB : TernaryVRRa<"wcfeb", 0xE7C2, null_frag, v32sb, v32f, 2, 8>;
1500:     }
1501:     // Rounding mode should agree with SystemZInstrFP.td.
1502:     def : FPConversion<VCFEB, any_fp_to_sint, v128f, v128sb, 0, 5>;
1503:   }
1504: 
1505:   // Convert to logical.
1506:   let Uses = [FPC], mayRaiseFPException = 1 in {
1507:     def VCLGD  : TernaryVRRaFloatGeneric<"vclgd", 0xE7C0>;
1508:     def VCLGDB : TernaryVRRa<"vclgdb", 0xE7C0, null_frag, v128g, v128db, 3, 0>;
1509:     def WCLGDB : TernaryVRRa<"wclgdb", 0xE7C0, null_frag, v64g, v64db, 3, 8>;
1510:   }
1511:   // Rounding mode should agree with SystemZInstrFP.td.
1512:   def : FPConversion<VCLGDB, any_fp_to_uint, v128g, v128db, 0, 5>;
1513:   let Predicates = [FeatureVectorEnhancements2] in {
1514:     let Uses = [FPC], mayRaiseFPException = 1 in {
1515:       let isAsmParserOnly = 1 in
1516:         def VCLFP  : TernaryVRRaFloatGeneric<"vclfp", 0xE7C0>;
1517:       def VCLFEB : TernaryVRRa<"vclfeb", 0xE7C0, null_frag, v128sb, v128g, 2, 0>;
1518:       def WCLFEB : TernaryVRRa<"wclfeb", 0xE7C0, null_frag, v32sb, v32f, 2, 8>;
1519:     }
1520:     // Rounding mode should agree with SystemZInstrFP.td.
```
- **EN**: This block declares or refines TableGen records such as `WCELFB`, `VCGD`, `VCGDB`, `WCGDB`, `VCSFP`, `VCFEB`.
- **CN**: 该代码块声明或细化了 `WCELFB`, `VCGD`, `VCGDB`, `WCGDB`, `VCSFP`, `VCFEB` 等 TableGen 记录。

### Lines 1521-1560 / 第 1521-1560 行
```tablegen
1521:     def : FPConversion<VCLFEB, any_fp_to_uint, v128f, v128sb, 0, 5>;
1522:   }
1523: 
1524:   // Divide.
1525:   let Uses = [FPC], mayRaiseFPException = 1 in {
1526:     def VFD   : BinaryVRRcFloatGeneric<"vfd", 0xE7E5>;
1527:     def VFDDB : BinaryVRRc<"vfddb", 0xE7E5, any_fdiv, v128db, v128db, 3, 0>;
1528:     def WFDDB : BinaryVRRc<"wfddb", 0xE7E5, any_fdiv, v64db, v64db, 3, 8, 0,
1529:                            "ddbr">;
1530:     let Predicates = [FeatureVectorEnhancements1] in {
1531:       def VFDSB : BinaryVRRc<"vfdsb", 0xE7E5, any_fdiv, v128sb, v128sb, 2, 0>;
1532:       def WFDSB : BinaryVRRc<"wfdsb", 0xE7E5, any_fdiv, v32sb, v32sb, 2, 8, 0,
1533:                              "debr">;
1534:       def WFDXB : BinaryVRRc<"wfdxb", 0xE7E5, any_fdiv, v128xb, v128xb, 4, 8>;
1535:     }
1536:   }
1537: 
1538:   // Load FP integer.
1539:   let Uses = [FPC], mayRaiseFPException = 1 in {
1540:     def VFI   : TernaryVRRaFloatGeneric<"vfi", 0xE7C7>;
1541:     def VFIDB : TernaryVRRa<"vfidb", 0xE7C7, int_s390_vfidb, v128db, v128db, 3, 0>;
1542:     def WFIDB : TernaryVRRa<"wfidb", 0xE7C7, null_frag, v64db, v64db, 3, 8>;
1543:   }
1544:   defm : VectorRounding<VFIDB, v128db>;
1545:   defm : VectorRounding<WFIDB, v64db>;
1546:   let Predicates = [FeatureVectorEnhancements1] in {
1547:     let Uses = [FPC], mayRaiseFPException = 1 in {
1548:       def VFISB : TernaryVRRa<"vfisb", 0xE7C7, int_s390_vfisb, v128sb, v128sb, 2, 0>;
1549:       def WFISB : TernaryVRRa<"wfisb", 0xE7C7, null_frag, v32sb, v32sb, 2, 8>;
1550:       def WFIXB : TernaryVRRa<"wfixb", 0xE7C7, null_frag, v128xb, v128xb, 4, 8>;
1551:     }
1552:     defm : VectorRounding<VFISB, v128sb>;
1553:     defm : VectorRounding<WFISB, v32sb>;
1554:     defm : VectorRounding<WFIXB, v128xb>;
1555:   }
1556: 
1557:   // Load lengthened.
1558:   let Uses = [FPC], mayRaiseFPException = 1 in {
1559:     def VLDE  : UnaryVRRaFloatGeneric<"vlde", 0xE7C4>;
1560:     def VLDEB : UnaryVRRa<"vldeb", 0xE7C4, z_any_vextend, v128db, v128sb, 2, 0>;
```
- **EN**: This block declares or refines TableGen records such as `VFD`, `VFDDB`, `WFDDB`, `VFDSB`, `WFDSB`, `WFDXB`.
- **CN**: 该代码块声明或细化了 `VFD`, `VFDDB`, `WFDDB`, `VFDSB`, `WFDSB`, `WFDXB` 等 TableGen 记录。

### Lines 1561-1600 / 第 1561-1600 行
```tablegen
1561:     def WLDEB : UnaryVRRa<"wldeb", 0xE7C4, any_fpextend, v64db, v32sb, 2, 8, 0,
1562:                           "ldebr">;
1563:   }
1564:   let Predicates = [FeatureVectorEnhancements1] in {
1565:     let Uses = [FPC], mayRaiseFPException = 1 in {
1566:       let isAsmParserOnly = 1 in {
1567:         def VFLL  : UnaryVRRaFloatGeneric<"vfll", 0xE7C4>;
1568:         def VFLLS : UnaryVRRa<"vflls", 0xE7C4, null_frag, v128db, v128sb, 2, 0>;
1569:         def WFLLS : UnaryVRRa<"wflls", 0xE7C4, null_frag, v64db, v32sb, 2, 8>;
1570:       }
1571:       def WFLLD : UnaryVRRa<"wflld", 0xE7C4, any_fpextend, v128xb, v64db, 3, 8>;
1572:     }
1573:     def : Pat<(f128 (any_fpextend (f32 VR32:$src))),
1574:               (WFLLD (WLDEB VR32:$src))>;
1575:   }
1576: 
1577:   // Load rounded.
1578:   let Uses = [FPC], mayRaiseFPException = 1 in {
1579:     def VLED  : TernaryVRRaFloatGeneric<"vled", 0xE7C5>;
1580:     def VLEDB : TernaryVRRa<"vledb", 0xE7C5, null_frag, v128sb, v128db, 3, 0>;
1581:     def WLEDB : TernaryVRRa<"wledb", 0xE7C5, null_frag, v32sb, v64db, 3, 8>;
1582:   }
1583:   def : Pat<(v4f32 (z_any_vround (v2f64 VR128:$src))), (VLEDB VR128:$src, 0, 0)>;
1584:   def : FPConversion<WLEDB, any_fpround, v32sb, v64db, 0, 0>;
1585:   let Predicates = [FeatureVectorEnhancements1] in {
1586:     let Uses = [FPC], mayRaiseFPException = 1 in {
1587:       let isAsmParserOnly = 1 in {
1588:         def VFLR  : TernaryVRRaFloatGeneric<"vflr", 0xE7C5>;
1589:         def VFLRD : TernaryVRRa<"vflrd", 0xE7C5, null_frag, v128sb, v128db, 3, 0>;
1590:         def WFLRD : TernaryVRRa<"wflrd", 0xE7C5, null_frag, v32sb, v64db, 3, 8>;
1591:       }
1592:       def WFLRX : TernaryVRRa<"wflrx", 0xE7C5, null_frag, v64db, v128xb, 4, 8>;
1593:     }
1594:     def : FPConversion<WFLRX, any_fpround, v64db, v128xb, 0, 0>;
1595:     def : Pat<(f32 (any_fpround (f128 VR128:$src))),
1596:               (WLEDB (WFLRX VR128:$src, 0, 3), 0, 0)>;
1597:   }
1598: 
1599:   // Maximum.
1600:   multiclass VectorMax<Instruction insn, TypedReg tr> {
```
- **EN**: This block declares or refines TableGen records such as `WLDEB`, `VFLL`, `VFLLS`, `WFLLS`, `WFLLD`, `VLED`.
- **CN**: 该代码块声明或细化了 `WLDEB`, `VFLL`, `VFLLS`, `WFLLS`, `WFLLD`, `VLED` 等 TableGen 记录。

### Lines 1601-1640 / 第 1601-1640 行
```tablegen
1601:     def : FPMinMax<insn, any_fmaxnum, tr, 4>;
1602:     def : FPMinMax<insn, fmaximumnum, tr, 4>;
1603:     def : FPMinMax<insn, any_fmaximum, tr, 1>;
1604:   }
1605:   let Predicates = [FeatureVectorEnhancements1] in {
1606:     let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1 in {
1607:       def VFMAX   : TernaryVRRcFloatGeneric<"vfmax", 0xE7EF>;
1608:       def VFMAXDB : TernaryVRRcFloat<"vfmaxdb", 0xE7EF, int_s390_vfmaxdb,
1609:                                      v128db, v128db, 3, 0>;
1610:       def WFMAXDB : TernaryVRRcFloat<"wfmaxdb", 0xE7EF, null_frag,
1611:                                      v64db, v64db, 3, 8>;
1612:       def VFMAXSB : TernaryVRRcFloat<"vfmaxsb", 0xE7EF, int_s390_vfmaxsb,
1613:                                      v128sb, v128sb, 2, 0>;
1614:       def WFMAXSB : TernaryVRRcFloat<"wfmaxsb", 0xE7EF, null_frag,
1615:                                      v32sb, v32sb, 2, 8>;
1616:       def WFMAXXB : TernaryVRRcFloat<"wfmaxxb", 0xE7EF, null_frag,
1617:                                      v128xb, v128xb, 4, 8>;
1618:     }
1619:     defm : VectorMax<VFMAXDB, v128db>;
1620:     defm : VectorMax<WFMAXDB, v64db>;
1621:     defm : VectorMax<VFMAXSB, v128sb>;
1622:     defm : VectorMax<WFMAXSB, v32sb>;
1623:     defm : VectorMax<WFMAXXB, v128xb>;
1624:   }
1625: 
1626:   // Minimum.
1627:   multiclass VectorMin<Instruction insn, TypedReg tr> {
1628:     def : FPMinMax<insn, any_fminnum, tr, 4>;
1629:     def : FPMinMax<insn, fminimumnum, tr, 4>;
1630:     def : FPMinMax<insn, any_fminimum, tr, 1>;
1631:   }
1632:   let Predicates = [FeatureVectorEnhancements1] in {
1633:     let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1 in {
1634:       def VFMIN   : TernaryVRRcFloatGeneric<"vfmin", 0xE7EE>;
1635:       def VFMINDB : TernaryVRRcFloat<"vfmindb", 0xE7EE, int_s390_vfmindb,
1636:                                      v128db, v128db, 3, 0>;
1637:       def WFMINDB : TernaryVRRcFloat<"wfmindb", 0xE7EE, null_frag,
1638:                                      v64db, v64db, 3, 8>;
1639:       def VFMINSB : TernaryVRRcFloat<"vfminsb", 0xE7EE, int_s390_vfminsb,
1640:                                      v128sb, v128sb, 2, 0>;
```
- **EN**: This block declares or refines TableGen records such as `VFMAX`, `VFMAXDB`, `WFMAXDB`, `VFMAXSB`, `WFMAXSB`, `WFMAXXB`.
- **CN**: 该代码块声明或细化了 `VFMAX`, `VFMAXDB`, `WFMAXDB`, `VFMAXSB`, `WFMAXSB`, `WFMAXXB` 等 TableGen 记录。

### Lines 1641-1680 / 第 1641-1680 行
```tablegen
1641:       def WFMINSB : TernaryVRRcFloat<"wfminsb", 0xE7EE, null_frag,
1642:                                      v32sb, v32sb, 2, 8>;
1643:       def WFMINXB : TernaryVRRcFloat<"wfminxb", 0xE7EE, null_frag,
1644:                                      v128xb, v128xb, 4, 8>;
1645:     }
1646:     defm : VectorMin<VFMINDB, v128db>;
1647:     defm : VectorMin<WFMINDB, v64db>;
1648:     defm : VectorMin<VFMINSB, v128sb>;
1649:     defm : VectorMin<WFMINSB, v32sb>;
1650:     defm : VectorMin<WFMINXB, v128xb>;
1651:   }
1652: 
1653:   // Multiply.
1654:   let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1 in {
1655:     def VFM   : BinaryVRRcFloatGeneric<"vfm", 0xE7E7>;
1656:     def VFMDB : BinaryVRRc<"vfmdb", 0xE7E7, any_fmul, v128db, v128db, 3, 0>;
1657:     def WFMDB : BinaryVRRc<"wfmdb", 0xE7E7, any_fmul, v64db, v64db, 3, 8, 0,
1658:                            "mdbr">;
1659:     let Predicates = [FeatureVectorEnhancements1] in {
1660:       def VFMSB : BinaryVRRc<"vfmsb", 0xE7E7, any_fmul, v128sb, v128sb, 2, 0>;
1661:       def WFMSB : BinaryVRRc<"wfmsb", 0xE7E7, any_fmul, v32sb, v32sb, 2, 8, 0,
1662:                              "meebr">;
1663:       def WFMXB : BinaryVRRc<"wfmxb", 0xE7E7, any_fmul, v128xb, v128xb, 4, 8>;
1664:     }
1665:   }
1666: 
1667:   // Multiply and add.
1668:   let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1 in {
1669:     def VFMA   : TernaryVRReFloatGeneric<"vfma", 0xE78F>;
1670:     def VFMADB : TernaryVRRe<"vfmadb", 0xE78F, any_fma, v128db, v128db, 0, 3>;
1671:     def WFMADB : TernaryVRRe<"wfmadb", 0xE78F, any_fma, v64db, v64db, 8, 3,
1672:                              "madbr">;
1673:     let Predicates = [FeatureVectorEnhancements1] in {
1674:       def VFMASB : TernaryVRRe<"vfmasb", 0xE78F, any_fma, v128sb, v128sb, 0, 2>;
1675:       def WFMASB : TernaryVRRe<"wfmasb", 0xE78F, any_fma, v32sb, v32sb, 8, 2,
1676:                                "maebr">;
1677:       def WFMAXB : TernaryVRRe<"wfmaxb", 0xE78F, any_fma, v128xb, v128xb, 8, 4>;
1678:     }
1679:   }
1680: 
```
- **EN**: This block declares or refines TableGen records such as `WFMINSB`, `WFMINXB`, `VFM`, `VFMDB`, `WFMDB`, `VFMSB`.
- **CN**: 该代码块声明或细化了 `WFMINSB`, `WFMINXB`, `VFM`, `VFMDB`, `WFMDB`, `VFMSB` 等 TableGen 记录。

### Lines 1681-1720 / 第 1681-1720 行
```tablegen
1681:   // Multiply and subtract.
1682:   let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1 in {
1683:     def VFMS   : TernaryVRReFloatGeneric<"vfms", 0xE78E>;
1684:     def VFMSDB : TernaryVRRe<"vfmsdb", 0xE78E, any_fms, v128db, v128db, 0, 3>;
1685:     def WFMSDB : TernaryVRRe<"wfmsdb", 0xE78E, any_fms, v64db, v64db, 8, 3,
1686:                              "msdbr">;
1687:     let Predicates = [FeatureVectorEnhancements1] in {
1688:       def VFMSSB : TernaryVRRe<"vfmssb", 0xE78E, any_fms, v128sb, v128sb, 0, 2>;
1689:       def WFMSSB : TernaryVRRe<"wfmssb", 0xE78E, any_fms, v32sb, v32sb, 8, 2,
1690:                                "msebr">;
1691:       def WFMSXB : TernaryVRRe<"wfmsxb", 0xE78E, any_fms, v128xb, v128xb, 8, 4>;
1692:     }
1693:   }
1694: 
1695:   // Negative multiply and add.
1696:   let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1,
1697:       Predicates = [FeatureVectorEnhancements1] in {
1698:     def VFNMA   : TernaryVRReFloatGeneric<"vfnma", 0xE79F>;
1699:     def VFNMADB : TernaryVRRe<"vfnmadb", 0xE79F, any_fnma, v128db, v128db, 0, 3>;
1700:     def WFNMADB : TernaryVRRe<"wfnmadb", 0xE79F, any_fnma, v64db, v64db, 8, 3>;
1701:     def VFNMASB : TernaryVRRe<"vfnmasb", 0xE79F, any_fnma, v128sb, v128sb, 0, 2>;
1702:     def WFNMASB : TernaryVRRe<"wfnmasb", 0xE79F, any_fnma, v32sb, v32sb, 8, 2>;
1703:     def WFNMAXB : TernaryVRRe<"wfnmaxb", 0xE79F, any_fnma, v128xb, v128xb, 8, 4>;
1704:   }
1705: 
1706:   // Negative multiply and subtract.
1707:   let Uses = [FPC], mayRaiseFPException = 1, isCommutable = 1,
1708:       Predicates = [FeatureVectorEnhancements1] in {
1709:     def VFNMS   : TernaryVRReFloatGeneric<"vfnms", 0xE79E>;
1710:     def VFNMSDB : TernaryVRRe<"vfnmsdb", 0xE79E, any_fnms, v128db, v128db, 0, 3>;
1711:     def WFNMSDB : TernaryVRRe<"wfnmsdb", 0xE79E, any_fnms, v64db, v64db, 8, 3>;
1712:     def VFNMSSB : TernaryVRRe<"vfnmssb", 0xE79E, any_fnms, v128sb, v128sb, 0, 2>;
1713:     def WFNMSSB : TernaryVRRe<"wfnmssb", 0xE79E, any_fnms, v32sb, v32sb, 8, 2>;
1714:     def WFNMSXB : TernaryVRRe<"wfnmsxb", 0xE79E, any_fnms, v128xb, v128xb, 8, 4>;
1715:   }
1716: 
1717:   // Perform sign operation.
1718:   def VFPSO   : BinaryVRRaFloatGeneric<"vfpso", 0xE7CC>;
1719:   def VFPSODB : BinaryVRRa<"vfpsodb", 0xE7CC, null_frag, v128db, v128db, 3, 0>;
1720:   def WFPSODB : BinaryVRRa<"wfpsodb", 0xE7CC, null_frag, v64db, v64db, 3, 8>;
```
- **EN**: This block declares or refines TableGen records such as `VFMS`, `VFMSDB`, `WFMSDB`, `VFMSSB`, `WFMSSB`, `WFMSXB`.
- **CN**: 该代码块声明或细化了 `VFMS`, `VFMSDB`, `WFMSDB`, `VFMSSB`, `WFMSSB`, `WFMSXB` 等 TableGen 记录。

### Lines 1721-1760 / 第 1721-1760 行
```tablegen
1721:   let Predicates = [FeatureVectorEnhancements1] in {
1722:     def VFPSOSB : BinaryVRRa<"vfpsosb", 0xE7CC, null_frag, v128sb, v128sb, 2, 0>;
1723:     def WFPSOSB : BinaryVRRa<"wfpsosb", 0xE7CC, null_frag, v32sb, v32sb, 2, 8>;
1724:     def WFPSOXB : BinaryVRRa<"wfpsoxb", 0xE7CC, null_frag, v128xb, v128xb, 4, 8>;
1725:   }
1726: 
1727:   // Load complement.
1728:   def VFLCDB : UnaryVRRa<"vflcdb", 0xE7CC, fneg, v128db, v128db, 3, 0, 0>;
1729:   def WFLCDB : UnaryVRRa<"wflcdb", 0xE7CC, fneg, v64db, v64db, 3, 8, 0>;
1730:   let Predicates = [FeatureVectorEnhancements1] in {
1731:     def VFLCSB : UnaryVRRa<"vflcsb", 0xE7CC, fneg, v128sb, v128sb, 2, 0, 0>;
1732:     def WFLCSB : UnaryVRRa<"wflcsb", 0xE7CC, fneg, v32sb, v32sb, 2, 8, 0>;
1733:     def WFLCXB : UnaryVRRa<"wflcxb", 0xE7CC, fneg, v128xb, v128xb, 4, 8, 0>;
1734:   }
1735: 
1736:   // Load negative.
1737:   def VFLNDB : UnaryVRRa<"vflndb", 0xE7CC, fnabs, v128db, v128db, 3, 0, 1>;
1738:   def WFLNDB : UnaryVRRa<"wflndb", 0xE7CC, fnabs, v64db, v64db, 3, 8, 1>;
1739:   let Predicates = [FeatureVectorEnhancements1] in {
1740:     def VFLNSB : UnaryVRRa<"vflnsb", 0xE7CC, fnabs, v128sb, v128sb, 2, 0, 1>;
1741:     def WFLNSB : UnaryVRRa<"wflnsb", 0xE7CC, fnabs, v32sb, v32sb, 2, 8, 1>;
1742:     def WFLNXB : UnaryVRRa<"wflnxb", 0xE7CC, fnabs, v128xb, v128xb, 4, 8, 1>;
1743:   }
1744: 
1745:   // Load positive.
1746:   def VFLPDB : UnaryVRRa<"vflpdb", 0xE7CC, fabs, v128db, v128db, 3, 0, 2>;
1747:   def WFLPDB : UnaryVRRa<"wflpdb", 0xE7CC, fabs, v64db, v64db, 3, 8, 2>;
1748:   let Predicates = [FeatureVectorEnhancements1] in {
1749:     def VFLPSB : UnaryVRRa<"vflpsb", 0xE7CC, fabs, v128sb, v128sb, 2, 0, 2>;
1750:     def WFLPSB : UnaryVRRa<"wflpsb", 0xE7CC, fabs, v32sb, v32sb, 2, 8, 2>;
1751:     def WFLPXB : UnaryVRRa<"wflpxb", 0xE7CC, fabs, v128xb, v128xb, 4, 8, 2>;
1752:   }
1753: 
1754:   // Square root.
1755:   let Uses = [FPC], mayRaiseFPException = 1 in {
1756:     def VFSQ   : UnaryVRRaFloatGeneric<"vfsq", 0xE7CE>;
1757:     def VFSQDB : UnaryVRRa<"vfsqdb", 0xE7CE, any_fsqrt, v128db, v128db, 3, 0>;
1758:     def WFSQDB : UnaryVRRa<"wfsqdb", 0xE7CE, any_fsqrt, v64db, v64db, 3, 8, 0,
1759:                            "sqdbr">;
1760:     let Predicates = [FeatureVectorEnhancements1] in {
```
- **EN**: This block declares or refines TableGen records such as `VFPSOSB`, `WFPSOSB`, `WFPSOXB`, `VFLCDB`, `WFLCDB`, `VFLCSB`.
- **CN**: 该代码块声明或细化了 `VFPSOSB`, `WFPSOSB`, `WFPSOXB`, `VFLCDB`, `WFLCDB`, `VFLCSB` 等 TableGen 记录。

### Lines 1761-1800 / 第 1761-1800 行
```tablegen
1761:       def VFSQSB : UnaryVRRa<"vfsqsb", 0xE7CE, any_fsqrt, v128sb, v128sb, 2, 0>;
1762:       def WFSQSB : UnaryVRRa<"wfsqsb", 0xE7CE, any_fsqrt, v32sb, v32sb, 2, 8, 0,
1763:                              "sqebr">;
1764:       def WFSQXB : UnaryVRRa<"wfsqxb", 0xE7CE, any_fsqrt, v128xb, v128xb, 4, 8>;
1765:     }
1766:   }
1767: 
1768:   // Subtract.
1769:   let Uses = [FPC], mayRaiseFPException = 1 in {
1770:     def VFS   : BinaryVRRcFloatGeneric<"vfs", 0xE7E2>;
1771:     def VFSDB : BinaryVRRc<"vfsdb", 0xE7E2, any_fsub, v128db, v128db, 3, 0>;
1772:     def WFSDB : BinaryVRRc<"wfsdb", 0xE7E2, any_fsub, v64db, v64db, 3, 8, 0,
1773:                            "sdbr">;
1774:     let Predicates = [FeatureVectorEnhancements1] in {
1775:       def VFSSB : BinaryVRRc<"vfssb", 0xE7E2, any_fsub, v128sb, v128sb, 2, 0>;
1776:       def WFSSB : BinaryVRRc<"wfssb", 0xE7E2, any_fsub, v32sb, v32sb, 2, 8, 0,
1777:                              "sebr">;
1778:       def WFSXB : BinaryVRRc<"wfsxb", 0xE7E2, any_fsub, v128xb, v128xb, 4, 8>;
1779:     }
1780:   }
1781: 
1782:   // Test data class immediate.
1783:   let Defs = [CC] in {
1784:     def VFTCI   : BinaryVRIeFloatGeneric<"vftci", 0xE74A>;
1785:     def VFTCIDB : BinaryVRIe<"vftcidb", 0xE74A, z_vftci, v128g, v128db, 3, 0>;
1786:     def WFTCIDB : BinaryVRIe<"wftcidb", 0xE74A, null_frag, v64g, v64db, 3, 8>;
1787:     let Predicates = [FeatureVectorEnhancements1] in {
1788:       def VFTCISB : BinaryVRIe<"vftcisb", 0xE74A, z_vftci, v128f, v128sb, 2, 0>;
1789:       def WFTCISB : BinaryVRIe<"wftcisb", 0xE74A, null_frag, v32f, v32sb, 2, 8>;
1790:       def WFTCIXB : BinaryVRIe<"wftcixb", 0xE74A, null_frag, v128q, v128xb, 4, 8>;
1791:     }
1792:   }
1793: }
1794: 
1795: //===----------------------------------------------------------------------===//
1796: // Floating-point comparison
1797: //===----------------------------------------------------------------------===//
1798: 
1799: let Predicates = [FeatureVector] in {
1800:   // Compare scalar.
```
- **EN**: This block declares or refines TableGen records such as `VFSQSB`, `WFSQSB`, `WFSQXB`, `VFS`, `VFSDB`, `WFSDB`.
- **CN**: 该代码块声明或细化了 `VFSQSB`, `WFSQSB`, `WFSQXB`, `VFS`, `VFSDB`, `WFSDB` 等 TableGen 记录。

### Lines 1801-1840 / 第 1801-1840 行
```tablegen
1801:   let Uses = [FPC], mayRaiseFPException = 1, Defs = [CC] in {
1802:     def WFC   : CompareVRRaFloatGeneric<"wfc", 0xE7CB>;
1803:     def WFCDB : CompareVRRa<"wfcdb", 0xE7CB, z_any_fcmp, v64db, 3, "cdbr">;
1804:     let Predicates = [FeatureVectorEnhancements1] in {
1805:       def WFCSB : CompareVRRa<"wfcsb", 0xE7CB, z_any_fcmp, v32sb, 2, "cebr">;
1806:       def WFCXB : CompareVRRa<"wfcxb", 0xE7CB, z_any_fcmp, v128xb, 4>;
1807:     }
1808:   }
1809: 
1810:   // Compare and signal scalar.
1811:   let Uses = [FPC], mayRaiseFPException = 1, Defs = [CC] in {
1812:     def WFK   : CompareVRRaFloatGeneric<"wfk", 0xE7CA>;
1813:     def WFKDB : CompareVRRa<"wfkdb", 0xE7CA, z_strict_fcmps, v64db, 3, "kdbr">;
1814:     let Predicates = [FeatureVectorEnhancements1] in {
1815:       def WFKSB : CompareVRRa<"wfksb", 0xE7CA, z_strict_fcmps, v32sb, 2, "kebr">;
1816:       def WFKXB : CompareVRRa<"wfkxb", 0xE7CA, z_strict_fcmps, v128xb, 4>;
1817:     }
1818:   }
1819: 
1820:   // Compare equal.
1821:   let Uses = [FPC], mayRaiseFPException = 1 in {
1822:     def  VFCE   : BinaryVRRcSPairFloatGeneric<"vfce", 0xE7E8>;
1823:     defm VFCEDB : BinaryVRRcSPair<"vfcedb", 0xE7E8, z_any_vfcmpe, z_vfcmpes,
1824:                                   v128g, v128db, 3, 0>;
1825:     defm WFCEDB : BinaryVRRcSPair<"wfcedb", 0xE7E8, null_frag, null_frag,
1826:                                   v64g, v64db, 3, 8>;
1827:     let Predicates = [FeatureVectorEnhancements1] in {
1828:       defm VFCESB : BinaryVRRcSPair<"vfcesb", 0xE7E8, z_any_vfcmpe, z_vfcmpes,
1829:                                     v128f, v128sb, 2, 0>;
1830:       defm WFCESB : BinaryVRRcSPair<"wfcesb", 0xE7E8, null_frag, null_frag,
1831:                                     v32f, v32sb, 2, 8>;
1832:       defm WFCEXB : BinaryVRRcSPair<"wfcexb", 0xE7E8, null_frag, null_frag,
1833:                                     v128q, v128xb, 4, 8>;
1834:     }
1835:   }
1836: 
1837:   // Compare and signal equal.
1838:   let Uses = [FPC], mayRaiseFPException = 1,
1839:       Predicates = [FeatureVectorEnhancements1] in {
1840:     defm VFKEDB : BinaryVRRcSPair<"vfkedb", 0xE7E8, z_strict_vfcmpes, null_frag,
```
- **EN**: This block declares or refines TableGen records such as `WFC`, `WFCDB`, `WFCSB`, `WFCXB`, `WFK`, `WFKDB`.
- **CN**: 该代码块声明或细化了 `WFC`, `WFCDB`, `WFCSB`, `WFCXB`, `WFK`, `WFKDB` 等 TableGen 记录。

### Lines 1841-1880 / 第 1841-1880 行
```tablegen
1841:                                   v128g, v128db, 3, 4>;
1842:     defm WFKEDB : BinaryVRRcSPair<"wfkedb", 0xE7E8, null_frag, null_frag,
1843:                                   v64g, v64db, 3, 12>;
1844:     defm VFKESB : BinaryVRRcSPair<"vfkesb", 0xE7E8, z_strict_vfcmpes, null_frag,
1845:                                   v128f, v128sb, 2, 4>;
1846:     defm WFKESB : BinaryVRRcSPair<"wfkesb", 0xE7E8, null_frag, null_frag,
1847:                                   v32f, v32sb, 2, 12>;
1848:     defm WFKEXB : BinaryVRRcSPair<"wfkexb", 0xE7E8, null_frag, null_frag,
1849:                                   v128q, v128xb, 4, 12>;
1850:   }
1851: 
1852:   // Compare high.
1853:   let Uses = [FPC], mayRaiseFPException = 1 in {
1854:     def  VFCH   : BinaryVRRcSPairFloatGeneric<"vfch", 0xE7EB>;
1855:     defm VFCHDB : BinaryVRRcSPair<"vfchdb", 0xE7EB, z_any_vfcmph, z_vfcmphs,
1856:                                   v128g, v128db, 3, 0>;
1857:     defm WFCHDB : BinaryVRRcSPair<"wfchdb", 0xE7EB, null_frag, null_frag,
1858:                                   v64g, v64db, 3, 8>;
1859:     let Predicates = [FeatureVectorEnhancements1] in {
1860:       defm VFCHSB : BinaryVRRcSPair<"vfchsb", 0xE7EB, z_any_vfcmph, z_vfcmphs,
1861:                                     v128f, v128sb, 2, 0>;
1862:       defm WFCHSB : BinaryVRRcSPair<"wfchsb", 0xE7EB, null_frag, null_frag,
1863:                                     v32f, v32sb, 2, 8>;
1864:       defm WFCHXB : BinaryVRRcSPair<"wfchxb", 0xE7EB, null_frag, null_frag,
1865:                                     v128q, v128xb, 4, 8>;
1866:     }
1867:   }
1868: 
1869:   // Compare and signal high.
1870:   let Uses = [FPC], mayRaiseFPException = 1,
1871:       Predicates = [FeatureVectorEnhancements1] in {
1872:     defm VFKHDB : BinaryVRRcSPair<"vfkhdb", 0xE7EB, z_strict_vfcmphs, null_frag,
1873:                                   v128g, v128db, 3, 4>;
1874:     defm WFKHDB : BinaryVRRcSPair<"wfkhdb", 0xE7EB, null_frag, null_frag,
1875:                                   v64g, v64db, 3, 12>;
1876:     defm VFKHSB : BinaryVRRcSPair<"vfkhsb", 0xE7EB, z_strict_vfcmphs, null_frag,
1877:                                   v128f, v128sb, 2, 4>;
1878:     defm WFKHSB : BinaryVRRcSPair<"wfkhsb", 0xE7EB, null_frag, null_frag,
1879:                                   v32f, v32sb, 2, 12>;
1880:     defm WFKHXB : BinaryVRRcSPair<"wfkhxb", 0xE7EB, null_frag, null_frag,
```
- **EN**: This block declares or refines TableGen records such as `WFKEDB`, `VFKESB`, `WFKESB`, `WFKEXB`, `VFCH`, `VFCHDB`.
- **CN**: 该代码块声明或细化了 `WFKEDB`, `VFKESB`, `WFKESB`, `WFKEXB`, `VFCH`, `VFCHDB` 等 TableGen 记录。

### Lines 1881-1920 / 第 1881-1920 行
```tablegen
1881:                                   v128q, v128xb, 4, 12>;
1882:   }
1883: 
1884:   // Compare high or equal.
1885:   let Uses = [FPC], mayRaiseFPException = 1 in {
1886:     def  VFCHE   : BinaryVRRcSPairFloatGeneric<"vfche", 0xE7EA>;
1887:     defm VFCHEDB : BinaryVRRcSPair<"vfchedb", 0xE7EA, z_any_vfcmphe, z_vfcmphes,
1888:                                    v128g, v128db, 3, 0>;
1889:     defm WFCHEDB : BinaryVRRcSPair<"wfchedb", 0xE7EA, null_frag, null_frag,
1890:                                    v64g, v64db, 3, 8>;
1891:     let Predicates = [FeatureVectorEnhancements1] in {
1892:       defm VFCHESB : BinaryVRRcSPair<"vfchesb", 0xE7EA, z_any_vfcmphe, z_vfcmphes,
1893:                                      v128f, v128sb, 2, 0>;
1894:       defm WFCHESB : BinaryVRRcSPair<"wfchesb", 0xE7EA, null_frag, null_frag,
1895:                                      v32f, v32sb, 2, 8>;
1896:       defm WFCHEXB : BinaryVRRcSPair<"wfchexb", 0xE7EA, null_frag, null_frag,
1897:                                      v128q, v128xb, 4, 8>;
1898:     }
1899:   }
1900: 
1901:   // Compare and signal high or equal.
1902:   let Uses = [FPC], mayRaiseFPException = 1,
1903:       Predicates = [FeatureVectorEnhancements1] in {
1904:     defm VFKHEDB : BinaryVRRcSPair<"vfkhedb", 0xE7EA, z_strict_vfcmphes, null_frag,
1905:                                    v128g, v128db, 3, 4>;
1906:     defm WFKHEDB : BinaryVRRcSPair<"wfkhedb", 0xE7EA, null_frag, null_frag,
1907:                                    v64g, v64db, 3, 12>;
1908:     defm VFKHESB : BinaryVRRcSPair<"vfkhesb", 0xE7EA, z_strict_vfcmphes, null_frag,
1909:                                    v128f, v128sb, 2, 4>;
1910:     defm WFKHESB : BinaryVRRcSPair<"wfkhesb", 0xE7EA, null_frag, null_frag,
1911:                                    v32f, v32sb, 2, 12>;
1912:     defm WFKHEXB : BinaryVRRcSPair<"wfkhexb", 0xE7EA, null_frag, null_frag,
1913:                                    v128q, v128xb, 4, 12>;
1914:   }
1915: }
1916: 
1917: //===----------------------------------------------------------------------===//
1918: // Support for 128-bit integer values in vector registers
1919: //===----------------------------------------------------------------------===//
1920: 
```
- **EN**: This block declares or refines TableGen records such as `VFCHE`, `VFCHEDB`, `WFCHEDB`, `VFCHESB`, `WFCHESB`, `WFCHEXB`.
- **CN**: 该代码块声明或细化了 `VFCHE`, `VFCHEDB`, `WFCHEDB`, `VFCHESB`, `WFCHESB`, `WFCHEXB` 等 TableGen 记录。

### Lines 1921-1960 / 第 1921-1960 行
```tablegen
1921: // Loads and stores.
1922: let Predicates = [FeatureVector] in {
1923:   def : Pat<(i128 (load bdxaddr12only:$addr)),
1924:             (VL bdxaddr12only:$addr)>;
1925:   def : Pat<(store (i128 VR128:$src), bdxaddr12only:$addr),
1926:             (VST VR128:$src, bdxaddr12only:$addr)>;
1927: }
1928: 
1929: // Full i128 move from GPR pair.
1930: let Predicates = [FeatureVector] in
1931:   def : Pat<(i128 (or (zext GR64:$x), (shl (anyext GR64:$y), (i32 64)))),
1932:             (VLVGP GR64:$y, GR64:$x)>;
1933: 
1934: // Any-extensions from GPR to i128.
1935: let Predicates = [FeatureVector] in {
1936:   def : Pat<(i128 (anyext GR32:$x)), (VLVGP32 GR32:$x, GR32:$x)>;
1937:   def : Pat<(i128 (anyext GR64:$x)), (VLVGP GR64:$x, GR64:$x)>;
1938: }
1939: 
1940: // Any-extending loads into i128.
1941: let Predicates = [FeatureVector] in {
1942:   def : Pat<(i128 (z_extloadi8 bdxaddr12only:$addr)),
1943:             (VLREPB bdxaddr12only:$addr)>;
1944:   def : Pat<(i128 (z_extloadi16 bdxaddr12only:$addr)),
1945:             (VLREPH bdxaddr12only:$addr)>;
1946:   def : Pat<(i128 (z_extloadi32 bdxaddr12only:$addr)),
1947:             (VLREPF bdxaddr12only:$addr)>;
1948:   def : Pat<(i128 (z_extloadi64 bdxaddr12only:$addr)),
1949:             (VLREPG bdxaddr12only:$addr)>;
1950: }
1951: 
1952: // Truncations from i128 to GPR.
1953: let Predicates = [FeatureVector] in {
1954:   def : Pat<(i32 (trunc (i128 VR128:$vec))),
1955:             (EXTRACT_SUBREG (VLGVF VR128:$vec, zero_reg, 3), subreg_l32)>;
1956:   def : Pat<(i32 (trunc (srl (i128 VR128:$vec), (i32 32)))),
1957:             (EXTRACT_SUBREG (VLGVF VR128:$vec, zero_reg, 2), subreg_l32)>;
1958:   def : Pat<(i32 (trunc (srl (i128 VR128:$vec), (i32 64)))),
1959:             (EXTRACT_SUBREG (VLGVF VR128:$vec, zero_reg, 1), subreg_l32)>;
1960:   def : Pat<(i32 (trunc (srl (i128 VR128:$vec), (i32 96)))),
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1961-2000 / 第 1961-2000 行
```tablegen
1961:             (EXTRACT_SUBREG (VLGVF VR128:$vec, zero_reg, 0), subreg_l32)>;
1962:   def : Pat<(i64 (trunc (i128 VR128:$vec))),
1963:             (VLGVG VR128:$vec, zero_reg, 1)>;
1964:   def : Pat<(i64 (trunc (srl (i128 VR128:$vec), (i32 64)))),
1965:             (VLGVG VR128:$vec, zero_reg, 0)>;
1966: }
1967: 
1968: // Truncating stores from i128.
1969: let Predicates = [FeatureVector] in {
1970:   def : Pat<(truncstorei8 (i128 VR128:$x), bdxaddr12only:$addr),
1971:             (VSTEB VR128:$x, bdxaddr12only:$addr, 15)>;
1972:   def : Pat<(truncstorei16 (i128 VR128:$x), bdxaddr12only:$addr),
1973:             (VSTEH VR128:$x, bdxaddr12only:$addr, 7)>;
1974:   def : Pat<(truncstorei32 (i128 VR128:$x), bdxaddr12only:$addr),
1975:             (VSTEF VR128:$x, bdxaddr12only:$addr, 3)>;
1976:   def : Pat<(truncstorei32 (srl (i128 VR128:$x), (i32 32)), bdxaddr12only:$addr),
1977:             (VSTEF VR128:$x, bdxaddr12only:$addr, 2)>;
1978:   def : Pat<(truncstorei32 (srl (i128 VR128:$x), (i32 64)), bdxaddr12only:$addr),
1979:             (VSTEF VR128:$x, bdxaddr12only:$addr, 1)>;
1980:   def : Pat<(truncstorei32 (srl (i128 VR128:$x), (i32 96)), bdxaddr12only:$addr),
1981:             (VSTEF VR128:$x, bdxaddr12only:$addr, 0)>;
1982:   def : Pat<(truncstorei64 (i128 VR128:$x), bdxaddr12only:$addr),
1983:             (VSTEG VR128:$x, bdxaddr12only:$addr, 1)>;
1984:   def : Pat<(truncstorei64 (srl (i128 VR128:$x), (i32 64)), bdxaddr12only:$addr),
1985:             (VSTEG VR128:$x, bdxaddr12only:$addr, 0)>;
1986: }
1987: 
1988: // Zero-extensions from GPR to i128.
1989: let Predicates = [FeatureVector] in {
1990:   def : Pat<(i128 (zext8 (anyext GR32:$x))),
1991:             (VLVGB (VGBM 0), GR32:$x, zero_reg, 15)>;
1992:   def : Pat<(i128 (zext16 (anyext GR32:$x))),
1993:             (VLVGH (VGBM 0), GR32:$x, zero_reg, 7)>;
1994:   def : Pat<(i128 (zext GR32:$x)),
1995:             (VLVGF (VGBM 0), GR32:$x, zero_reg, 3)>;
1996:   def : Pat<(i128 (zext GR64:$x)),
1997:             (VLVGG (VGBM 0), GR64:$x, zero_reg, 1)>;
1998: }
1999: 
2000: // Zero-extending loads into i128.
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2001-2040 / 第 2001-2040 行
```tablegen
2001: let Predicates = [FeatureVector] in {
2002:   def : Pat<(i128 (z_zextloadi8 bdxaddr12only:$addr)),
2003:             (VLEB (VGBM 0), bdxaddr12only:$addr, 15)>;
2004:   def : Pat<(i128 (z_zextloadi16 bdxaddr12only:$addr)),
2005:             (VLEH (VGBM 0), bdxaddr12only:$addr, 7)>;
2006:   def : Pat<(i128 (z_zextloadi32 bdxaddr12only:$addr)),
2007:             (VLEF (VGBM 0), bdxaddr12only:$addr, 3)>;
2008:   def : Pat<(i128 (z_zextloadi64 bdxaddr12only:$addr)),
2009:             (VLEG (VGBM 0), bdxaddr12only:$addr, 1)>;
2010: }
2011: 
2012: // Zero-extensions from VR element to i128 on z17.
2013: let Predicates = [FeatureVectorEnhancements3] in {
2014:   def : Pat<(i128 (zext (i64 (z_vector_extract (v2i64 VR128:$src), 0)))),
2015:             (VUPLHG VR128:$src)>;
2016:   def : Pat<(i128 (zext (i64 (z_vector_extract (v2i64 VR128:$src), 1)))),
2017:             (VUPLLG VR128:$src)>;
2018:   def : Pat<(i128 (zext (i32 (z_vector_extract (v4i32 VR128:$src), 0)))),
2019:             (VUPLHG (VUPLHF VR128:$src))>;
2020:   def : Pat<(i128 (zext (i32 (z_vector_extract (v4i32 VR128:$src), 1)))),
2021:             (VUPLHG (VUPLLF VR128:$src))>;
2022:   def : Pat<(i128 (zext (i32 (z_vector_extract (v4i32 VR128:$src), 2)))),
2023:             (VUPLLG (VUPLHF VR128:$src))>;
2024:   def : Pat<(i128 (zext (i32 (z_vector_extract (v4i32 VR128:$src), 3)))),
2025:             (VUPLLG (VUPLLF VR128:$src))>;
2026: }
2027: 
2028: // In-register i128 sign-extensions on z17.
2029: let Predicates = [FeatureVectorEnhancements3] in {
2030:   def : Pat<(i128 (sext_inreg VR128:$x, i8)), (VUPLG (VSEGB VR128:$x))>;
2031:   def : Pat<(i128 (sext_inreg VR128:$x, i16)), (VUPLG (VSEGH VR128:$x))>;
2032:   def : Pat<(i128 (sext_inreg VR128:$x, i32)), (VUPLG (VSEGF VR128:$x))>;
2033:   def : Pat<(i128 (sext_inreg VR128:$x, i64)), (VUPLG VR128:$x)>;
2034: }
2035: 
2036: // In-register i128 sign-extensions.
2037: let Predicates = [FeatureVector] in {
2038:   def : Pat<(i128 (sext_inreg VR128:$x, i8)),
2039:             (VSRAB (VREPB VR128:$x, 15), (VREPIB 120))>;
2040:   def : Pat<(i128 (sext_inreg VR128:$x, i16)),
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2041-2080 / 第 2041-2080 行
```tablegen
2041:             (VSRAB (VREPH VR128:$x, 7), (VREPIB 112))>;
2042:   def : Pat<(i128 (sext_inreg VR128:$x, i32)),
2043:             (VSRAB (VREPF VR128:$x, 3), (VREPIB 96))>;
2044:   def : Pat<(i128 (sext_inreg VR128:$x, i64)),
2045:             (VSRAB (VREPG VR128:$x, 1), (VREPIB 64))>;
2046: }
2047: 
2048: // Sign-extensions from GPR to i128 on z17.
2049: let Predicates = [FeatureVectorEnhancements3] in {
2050:   def : Pat<(i128 (sext_inreg (anyext GR32:$x), i8)),
2051:             (VUPLG (VLVGP (LGBR (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$x, subreg_l32)),
2052:                           (LGBR (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$x, subreg_l32))))>;
2053:   def : Pat<(i128 (sext_inreg (anyext GR32:$x), i16)),
2054:             (VUPLG (VLVGP (LGHR (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$x, subreg_l32)),
2055:                           (LGHR (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GR32:$x, subreg_l32))))>;
2056:   def : Pat<(i128 (sext GR32:$x)),
2057:             (VUPLG (VLVGP (LGFR GR32:$x), (LGFR GR32:$x)))>;
2058:   def : Pat<(i128 (sext GR64:$x)),
2059:             (VUPLG (VLVGP GR64:$x, GR64:$x))>;
2060: }
2061: 
2062: // Sign-extensions from GPR to i128.
2063: let Predicates = [FeatureVector] in {
2064:   def : Pat<(i128 (sext_inreg (anyext GR32:$x), i8)),
2065:             (VLVGP (SRAG (LGBR (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
2066:                                  GR32:$x, subreg_l32)), zero_reg, 63),
2067:                    (LGBR (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
2068:                             GR32:$x, subreg_l32)))>;
2069:   def : Pat<(i128 (sext_inreg (anyext GR32:$x), i16)),
2070:             (VLVGP (SRAG (LGHR (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
2071:                                   GR32:$x, subreg_l32)), zero_reg, 63),
2072:                    (LGHR (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
2073:                             GR32:$x, subreg_l32)))>;
2074:   def : Pat<(i128 (sext GR32:$x)),
2075:             (VLVGP (SRAG (LGFR GR32:$x), zero_reg, 63), (LGFR GR32:$x))>;
2076:   def : Pat<(i128 (sext GR64:$x)),
2077:             (VLVGP (SRAG GR64:$x, zero_reg, 63), GR64:$x)>;
2078: }
2079: 
2080: // Sign-extending loads into i128.
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2081-2120 / 第 2081-2120 行
```tablegen
2081: let Predicates = [FeatureVector] in {
2082:   def : Pat<(i128 (z_sextloadi8 bdxaddr12only:$addr)),
2083:             (VSRAB (VLREPB bdxaddr12only:$addr), (VREPIB 120))>;
2084:   def : Pat<(i128 (z_sextloadi16 bdxaddr12only:$addr)),
2085:             (VSRAB (VLREPH bdxaddr12only:$addr), (VREPIB 112))>;
2086:   def : Pat<(i128 (z_sextloadi32 bdxaddr12only:$addr)),
2087:             (VSRAB (VLREPF bdxaddr12only:$addr), (VREPIB 96))>;
2088:   def : Pat<(i128 (z_sextloadi64 bdxaddr12only:$addr)),
2089:             (VSRAB (VLREPG bdxaddr12only:$addr), (VREPIB 64))>;
2090: }
2091: 
2092: // Sign-extensions from VR element to i128 on z17.
2093: let Predicates = [FeatureVectorEnhancements3] in {
2094:   def : Pat<(i128 (sext (i64 (z_vector_extract (v2i64 VR128:$src), 0)))),
2095:             (VUPHG VR128:$src)>;
2096:   def : Pat<(i128 (sext (i64 (z_vector_extract (v2i64 VR128:$src), 1)))),
2097:             (VUPLG VR128:$src)>;
2098:   def : Pat<(i128 (sext (i32 (z_vector_extract (v4i32 VR128:$src), 0)))),
2099:             (VUPHG (VUPHF VR128:$src))>;
2100:   def : Pat<(i128 (sext (i32 (z_vector_extract (v4i32 VR128:$src), 1)))),
2101:             (VUPHG (VUPLF VR128:$src))>;
2102:   def : Pat<(i128 (sext (i32 (z_vector_extract (v4i32 VR128:$src), 2)))),
2103:             (VUPLG (VUPHF VR128:$src))>;
2104:   def : Pat<(i128 (sext (i32 (z_vector_extract (v4i32 VR128:$src), 3)))),
2105:             (VUPLG (VUPLF VR128:$src))>;
2106: }
2107: 
2108: // i128 comparison pseudo-instructions.
2109: let Predicates = [FeatureVector], Defs = [CC],
2110:     usesCustomInserter = 1, hasNoSchedulingInfo = 1 in {
2111:   def SCmp128Hi : Pseudo<(outs), (ins VR128:$src1, VR128:$src2),
2112:                          [(set CC, (z_scmp128hi (i128 VR128:$src1),
2113:                                                 (i128 VR128:$src2)))]>;
2114:   def UCmp128Hi : Pseudo<(outs), (ins VR128:$src1, VR128:$src2),
2115:                          [(set CC, (z_ucmp128hi (i128 VR128:$src1),
2116:                                                 (i128 VR128:$src2)))]>;
2117: }
2118: 
2119: // i128 select pseudo-instructions.
2120: let Predicates = [FeatureVector] in
```
- **EN**: This block declares or refines TableGen records such as `SCmp128Hi`, `UCmp128Hi`.
- **CN**: 该代码块声明或细化了 `SCmp128Hi`, `UCmp128Hi` 等 TableGen 记录。

### Lines 2121-2160 / 第 2121-2160 行
```tablegen
2121:   def Select128 : SelectWrapper<i128, VR128>;
2122: 
2123: //===----------------------------------------------------------------------===//
2124: // Conversions
2125: //===----------------------------------------------------------------------===//
2126: 
2127: let Predicates = [FeatureVector] in {
2128: def : Pat<(v16i8 (bitconvert (v8i16 VR128:$src))), (v16i8 VR128:$src)>;
2129: def : Pat<(v16i8 (bitconvert (v4i32 VR128:$src))), (v16i8 VR128:$src)>;
2130: def : Pat<(v16i8 (bitconvert (v2i64 VR128:$src))), (v16i8 VR128:$src)>;
2131: def : Pat<(v16i8 (bitconvert (i128  VR128:$src))), (v16i8 VR128:$src)>;
2132: def : Pat<(v16i8 (bitconvert (v8f16 VR128:$src))), (v16i8 VR128:$src)>;
2133: def : Pat<(v16i8 (bitconvert (v4f32 VR128:$src))), (v16i8 VR128:$src)>;
2134: def : Pat<(v16i8 (bitconvert (v2f64 VR128:$src))), (v16i8 VR128:$src)>;
2135: def : Pat<(v16i8 (bitconvert (f128  VR128:$src))), (v16i8 VR128:$src)>;
2136: 
2137: def : Pat<(v8i16 (bitconvert (v16i8 VR128:$src))), (v8i16 VR128:$src)>;
2138: def : Pat<(v8i16 (bitconvert (v4i32 VR128:$src))), (v8i16 VR128:$src)>;
2139: def : Pat<(v8i16 (bitconvert (v2i64 VR128:$src))), (v8i16 VR128:$src)>;
2140: def : Pat<(v8i16 (bitconvert (i128  VR128:$src))), (v8i16 VR128:$src)>;
2141: def : Pat<(v8i16 (bitconvert (v8f16 VR128:$src))), (v8i16 VR128:$src)>;
2142: def : Pat<(v8i16 (bitconvert (v4f32 VR128:$src))), (v8i16 VR128:$src)>;
2143: def : Pat<(v8i16 (bitconvert (v2f64 VR128:$src))), (v8i16 VR128:$src)>;
2144: def : Pat<(v8i16 (bitconvert (f128  VR128:$src))), (v8i16 VR128:$src)>;
2145: 
2146: def : Pat<(v4i32 (bitconvert (v16i8 VR128:$src))), (v4i32 VR128:$src)>;
2147: def : Pat<(v4i32 (bitconvert (v8i16 VR128:$src))), (v4i32 VR128:$src)>;
2148: def : Pat<(v4i32 (bitconvert (v2i64 VR128:$src))), (v4i32 VR128:$src)>;
2149: def : Pat<(v4i32 (bitconvert (i128  VR128:$src))), (v4i32 VR128:$src)>;
2150: def : Pat<(v4i32 (bitconvert (v8f16 VR128:$src))), (v4i32 VR128:$src)>;
2151: def : Pat<(v4i32 (bitconvert (v4f32 VR128:$src))), (v4i32 VR128:$src)>;
2152: def : Pat<(v4i32 (bitconvert (v2f64 VR128:$src))), (v4i32 VR128:$src)>;
2153: def : Pat<(v4i32 (bitconvert (f128  VR128:$src))), (v4i32 VR128:$src)>;
2154: 
2155: def : Pat<(v2i64 (bitconvert (v16i8 VR128:$src))), (v2i64 VR128:$src)>;
2156: def : Pat<(v2i64 (bitconvert (v8i16 VR128:$src))), (v2i64 VR128:$src)>;
2157: def : Pat<(v2i64 (bitconvert (v4i32 VR128:$src))), (v2i64 VR128:$src)>;
2158: def : Pat<(v2i64 (bitconvert (i128  VR128:$src))), (v2i64 VR128:$src)>;
2159: def : Pat<(v2i64 (bitconvert (v8f16 VR128:$src))), (v2i64 VR128:$src)>;
2160: def : Pat<(v2i64 (bitconvert (v4f32 VR128:$src))), (v2i64 VR128:$src)>;
```
- **EN**: This block declares or refines TableGen records such as `Select128`.
- **CN**: 该代码块声明或细化了 `Select128` 等 TableGen 记录。

### Lines 2161-2200 / 第 2161-2200 行
```tablegen
2161: def : Pat<(v2i64 (bitconvert (v2f64 VR128:$src))), (v2i64 VR128:$src)>;
2162: def : Pat<(v2i64 (bitconvert (f128  VR128:$src))), (v2i64 VR128:$src)>;
2163: 
2164: def : Pat<(v8f16 (bitconvert (v16i8 VR128:$src))), (v8f16 VR128:$src)>;
2165: def : Pat<(v8f16 (bitconvert (v8i16 VR128:$src))), (v8f16 VR128:$src)>;
2166: def : Pat<(v8f16 (bitconvert (v4i32 VR128:$src))), (v8f16 VR128:$src)>;
2167: def : Pat<(v8f16 (bitconvert (v2i64 VR128:$src))), (v8f16 VR128:$src)>;
2168: def : Pat<(v8f16 (bitconvert (i128  VR128:$src))), (v8f16 VR128:$src)>;
2169: def : Pat<(v8f16 (bitconvert (v4f32 VR128:$src))), (v8f16 VR128:$src)>;
2170: def : Pat<(v8f16 (bitconvert (v2f64 VR128:$src))), (v8f16 VR128:$src)>;
2171: def : Pat<(v8f16 (bitconvert (f128  VR128:$src))), (v8f16 VR128:$src)>;
2172: 
2173: def : Pat<(v4f32 (bitconvert (v16i8 VR128:$src))), (v4f32 VR128:$src)>;
2174: def : Pat<(v4f32 (bitconvert (v8i16 VR128:$src))), (v4f32 VR128:$src)>;
2175: def : Pat<(v4f32 (bitconvert (v4i32 VR128:$src))), (v4f32 VR128:$src)>;
2176: def : Pat<(v4f32 (bitconvert (i128  VR128:$src))), (v4f32 VR128:$src)>;
2177: def : Pat<(v4f32 (bitconvert (v2i64 VR128:$src))), (v4f32 VR128:$src)>;
2178: def : Pat<(v4f32 (bitconvert (v8f16 VR128:$src))), (v4f32 VR128:$src)>;
2179: def : Pat<(v4f32 (bitconvert (v2f64 VR128:$src))), (v4f32 VR128:$src)>;
2180: def : Pat<(v4f32 (bitconvert (f128  VR128:$src))), (v4f32 VR128:$src)>;
2181: 
2182: def : Pat<(v2f64 (bitconvert (v16i8 VR128:$src))), (v2f64 VR128:$src)>;
2183: def : Pat<(v2f64 (bitconvert (v8i16 VR128:$src))), (v2f64 VR128:$src)>;
2184: def : Pat<(v2f64 (bitconvert (v4i32 VR128:$src))), (v2f64 VR128:$src)>;
2185: def : Pat<(v2f64 (bitconvert (i128  VR128:$src))), (v2f64 VR128:$src)>;
2186: def : Pat<(v2f64 (bitconvert (v2i64 VR128:$src))), (v2f64 VR128:$src)>;
2187: def : Pat<(v2f64 (bitconvert (v8f16 VR128:$src))), (v2f64 VR128:$src)>;
2188: def : Pat<(v2f64 (bitconvert (v4f32 VR128:$src))), (v2f64 VR128:$src)>;
2189: def : Pat<(v2f64 (bitconvert (f128  VR128:$src))), (v2f64 VR128:$src)>;
2190: 
2191: def : Pat<(f128  (bitconvert (v16i8 VR128:$src))), (f128  VR128:$src)>;
2192: def : Pat<(f128  (bitconvert (v8i16 VR128:$src))), (f128  VR128:$src)>;
2193: def : Pat<(f128  (bitconvert (v4i32 VR128:$src))), (f128  VR128:$src)>;
2194: def : Pat<(f128  (bitconvert (v2i64 VR128:$src))), (f128  VR128:$src)>;
2195: def : Pat<(f128  (bitconvert (i128  VR128:$src))), (f128  VR128:$src)>;
2196: def : Pat<(f128  (bitconvert (v8f16 VR128:$src))), (f128  VR128:$src)>;
2197: def : Pat<(f128  (bitconvert (v4f32 VR128:$src))), (f128  VR128:$src)>;
2198: def : Pat<(f128  (bitconvert (v2f64 VR128:$src))), (f128  VR128:$src)>;
2199: 
2200: def : Pat<(i128  (bitconvert (v16i8 VR128:$src))), (i128  VR128:$src)>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2201-2240 / 第 2201-2240 行
```tablegen
2201: def : Pat<(i128  (bitconvert (v8i16 VR128:$src))), (i128  VR128:$src)>;
2202: def : Pat<(i128  (bitconvert (v4i32 VR128:$src))), (i128  VR128:$src)>;
2203: def : Pat<(i128  (bitconvert (v2i64 VR128:$src))), (i128  VR128:$src)>;
2204: def : Pat<(i128  (bitconvert (v8f16 VR128:$src))), (i128  VR128:$src)>;
2205: def : Pat<(i128  (bitconvert (v4f32 VR128:$src))), (i128  VR128:$src)>;
2206: def : Pat<(i128  (bitconvert (v2f64 VR128:$src))), (i128  VR128:$src)>;
2207: def : Pat<(i128  (bitconvert (f128  VR128:$src))), (i128  VR128:$src)>;
2208: } // End Predicates = [FeatureVector]
2209: 
2210: //===----------------------------------------------------------------------===//
2211: // Replicating scalars
2212: //===----------------------------------------------------------------------===//
2213: 
2214: // Define patterns for replicating a scalar GR32 into a vector of type TYPE.
2215: // INDEX is 8 minus the element size in bytes.
2216: class VectorReplicateScalar<ValueType type, Instruction insn, bits<16> index>
2217:   : Pat<(type (z_replicate GR32:$scalar)),
2218:         (insn (VLVGP32 GR32:$scalar, GR32:$scalar), index)>;
2219: 
2220: def : VectorReplicateScalar<v16i8, VREPB, 7>;
2221: def : VectorReplicateScalar<v8i16, VREPH, 3>;
2222: def : VectorReplicateScalar<v4i32, VREPF, 1>;
2223: 
2224: // i64 replications are just a single instruction.
2225: def : Pat<(v2i64 (z_replicate GR64:$scalar)),
2226:           (VLVGP GR64:$scalar, GR64:$scalar)>;
2227: 
2228: //===----------------------------------------------------------------------===//
2229: // Floating-point insertion and extraction
2230: //===----------------------------------------------------------------------===//
2231: 
2232: // Moving 32-bit values between GPRs and FPRs can be done using VLVGF
2233: // and VLGVF.
2234: let Predicates = [FeatureVector] in {
2235:   def LEFR : UnaryAliasVRS<VR32, GR32>;
2236:   def LFER : UnaryAliasVRS<GR64, VR32>;
2237:   def : Pat<(f32 (bitconvert (i32 GR32:$src))), (LEFR GR32:$src)>;
2238:   def : Pat<(i32 (bitconvert (f32 VR32:$src))),
2239:             (EXTRACT_SUBREG (LFER VR32:$src), subreg_l32)>;
2240:   def LEFR_16 : UnaryAliasVRS<VR16, GR32>;
```
- **EN**: This block declares or refines TableGen records such as `VectorReplicateScalar`, `LEFR`, `LFER`, `LEFR_16`.
- **CN**: 该代码块声明或细化了 `VectorReplicateScalar`, `LEFR`, `LFER`, `LEFR_16` 等 TableGen 记录。

### Lines 2241-2280 / 第 2241-2280 行
```tablegen
2241:   def LFER_16 : UnaryAliasVRS<GR32, VR16>;
2242: }
2243: 
2244: // Floating-point values are stored in element 0 of the corresponding
2245: // vector register.  Scalar to vector conversion is just a subreg and
2246: // scalar replication can just replicate element 0 of the vector register.
2247: multiclass ScalarToVectorFP<Instruction vrep, ValueType vt, RegisterOperand cls,
2248:                             SubRegIndex subreg> {
2249:   def : Pat<(vt (scalar_to_vector cls:$scalar)),
2250:             (INSERT_SUBREG (vt (IMPLICIT_DEF)), cls:$scalar, subreg)>;
2251:   def : Pat<(vt (z_replicate cls:$scalar)),
2252:             (vrep (INSERT_SUBREG (vt (IMPLICIT_DEF)), cls:$scalar,
2253:                                  subreg), 0)>;
2254: }
2255: defm : ScalarToVectorFP<VREPH, v8f16, FP16, subreg_h16>;
2256: defm : ScalarToVectorFP<VREPF, v4f32, FP32, subreg_h32>;
2257: defm : ScalarToVectorFP<VREPG, v2f64, FP64, subreg_h64>;
2258: 
2259: // Match v2f64 insertions.  The AddedComplexity counters the 3 added by
2260: // TableGen for the base register operand in VLVG-based integer insertions
2261: // and ensures that this version is strictly better.
2262: let AddedComplexity = 4 in {
2263:   def : Pat<(z_vector_insert (v2f64 VR128:$vec), FP64:$elt, 0),
2264:             (VPDI (INSERT_SUBREG (v2f64 (IMPLICIT_DEF)), FP64:$elt,
2265:                                  subreg_h64), VR128:$vec, 1)>;
2266:   def : Pat<(z_vector_insert (v2f64 VR128:$vec), FP64:$elt, 1),
2267:             (VPDI VR128:$vec, (INSERT_SUBREG (v2f64 (IMPLICIT_DEF)), FP64:$elt,
2268:                                              subreg_h64), 0)>;
2269: }
2270: 
2271: // We extract floating-point element X by replicating (for elements other
2272: // than 0) and then taking a high subreg.  The AddedComplexity counters the
2273: // 3 added by TableGen for the base register operand in VLGV-based integer
2274: // extractions and ensures that this version is strictly better.
2275: let AddedComplexity = 4 in {
2276:   def : Pat<(f16 (z_vector_extract (v8f16 VR128:$vec), 0)),
2277:             (EXTRACT_SUBREG VR128:$vec, subreg_h16)>;
2278:   def : Pat<(f16 (z_vector_extract (v8f16 VR128:$vec), imm32zx3:$index)),
2279:             (EXTRACT_SUBREG (VREPH VR128:$vec, imm32zx2:$index), subreg_h16)>;
2280: 
```
- **EN**: This block declares or refines TableGen records such as `LFER_16`, `ScalarToVectorFP`.
- **CN**: 该代码块声明或细化了 `LFER_16`, `ScalarToVectorFP` 等 TableGen 记录。

### Lines 2281-2320 / 第 2281-2320 行
```tablegen
2281:   def : Pat<(f32 (z_vector_extract (v4f32 VR128:$vec), 0)),
2282:             (EXTRACT_SUBREG VR128:$vec, subreg_h32)>;
2283:   def : Pat<(f32 (z_vector_extract (v4f32 VR128:$vec), imm32zx2:$index)),
2284:             (EXTRACT_SUBREG (VREPF VR128:$vec, imm32zx2:$index), subreg_h32)>;
2285: 
2286:   def : Pat<(f64 (z_vector_extract (v2f64 VR128:$vec), 0)),
2287:             (EXTRACT_SUBREG VR128:$vec, subreg_h64)>;
2288:   def : Pat<(f64 (z_vector_extract (v2f64 VR128:$vec), imm32zx1:$index)),
2289:             (EXTRACT_SUBREG (VREPG VR128:$vec, imm32zx1:$index), subreg_h64)>;
2290: }
2291: 
2292: //===----------------------------------------------------------------------===//
2293: // Support for 128-bit floating-point values in vector registers
2294: //===----------------------------------------------------------------------===//
2295: 
2296: let Predicates = [FeatureVectorEnhancements1] in {
2297:   def : Pat<(f128 (load bdxaddr12only:$addr)),
2298:             (VL bdxaddr12only:$addr)>;
2299:   def : Pat<(store (f128 VR128:$src), bdxaddr12only:$addr),
2300:             (VST VR128:$src, bdxaddr12only:$addr)>;
2301: 
2302:   def : Pat<(f128 fpimm0), (VZERO)>;
2303:   def : Pat<(f128 fpimmneg0), (WFLNXB (VZERO))>;
2304: }
2305: 
2306: //===----------------------------------------------------------------------===//
2307: // String instructions
2308: //===----------------------------------------------------------------------===//
2309: 
2310: let Predicates = [FeatureVector] in {
2311:   defm VFAE  : TernaryOptVRRbSPairGeneric<"vfae", 0xE782>;
2312:   defm VFAEB : TernaryOptVRRbSPair<"vfaeb", 0xE782, int_s390_vfaeb,
2313:                                    z_vfae_cc, v128b, v128b, 0>;
2314:   defm VFAEH : TernaryOptVRRbSPair<"vfaeh", 0xE782, int_s390_vfaeh,
2315:                                    z_vfae_cc, v128h, v128h, 1>;
2316:   defm VFAEF : TernaryOptVRRbSPair<"vfaef", 0xE782, int_s390_vfaef,
2317:                                    z_vfae_cc, v128f, v128f, 2>;
2318:   defm VFAEZB : TernaryOptVRRbSPair<"vfaezb", 0xE782, int_s390_vfaezb,
2319:                                     z_vfaez_cc, v128b, v128b, 0, 2>;
2320:   defm VFAEZH : TernaryOptVRRbSPair<"vfaezh", 0xE782, int_s390_vfaezh,
```
- **EN**: This block declares or refines TableGen records such as `VFAE`, `VFAEB`, `VFAEH`, `VFAEF`, `VFAEZB`, `VFAEZH`.
- **CN**: 该代码块声明或细化了 `VFAE`, `VFAEB`, `VFAEH`, `VFAEF`, `VFAEZB`, `VFAEZH` 等 TableGen 记录。

### Lines 2321-2360 / 第 2321-2360 行
```tablegen
2321:                                     z_vfaez_cc, v128h, v128h, 1, 2>;
2322:   defm VFAEZF : TernaryOptVRRbSPair<"vfaezf", 0xE782, int_s390_vfaezf,
2323:                                     z_vfaez_cc, v128f, v128f, 2, 2>;
2324: 
2325:   defm VFEE  : BinaryExtraVRRbSPairGeneric<"vfee", 0xE780>;
2326:   defm VFEEB : BinaryExtraVRRbSPair<"vfeeb", 0xE780, int_s390_vfeeb,
2327:                                     z_vfee_cc, v128b, v128b, 0>;
2328:   defm VFEEH : BinaryExtraVRRbSPair<"vfeeh", 0xE780, int_s390_vfeeh,
2329:                                     z_vfee_cc, v128h, v128h, 1>;
2330:   defm VFEEF : BinaryExtraVRRbSPair<"vfeef", 0xE780, int_s390_vfeef,
2331:                                     z_vfee_cc, v128f, v128f, 2>;
2332:   defm VFEEZB : BinaryVRRbSPair<"vfeezb", 0xE780, int_s390_vfeezb,
2333:                                 z_vfeez_cc, v128b, v128b, 0, 2>;
2334:   defm VFEEZH : BinaryVRRbSPair<"vfeezh", 0xE780, int_s390_vfeezh,
2335:                                 z_vfeez_cc, v128h, v128h, 1, 2>;
2336:   defm VFEEZF : BinaryVRRbSPair<"vfeezf", 0xE780, int_s390_vfeezf,
2337:                                 z_vfeez_cc, v128f, v128f, 2, 2>;
2338: 
2339:   defm VFENE  : BinaryExtraVRRbSPairGeneric<"vfene", 0xE781>;
2340:   defm VFENEB : BinaryExtraVRRbSPair<"vfeneb", 0xE781, int_s390_vfeneb,
2341:                                      z_vfene_cc, v128b, v128b, 0>;
2342:   defm VFENEH : BinaryExtraVRRbSPair<"vfeneh", 0xE781, int_s390_vfeneh,
2343:                                      z_vfene_cc, v128h, v128h, 1>;
2344:   defm VFENEF : BinaryExtraVRRbSPair<"vfenef", 0xE781, int_s390_vfenef,
2345:                                      z_vfene_cc, v128f, v128f, 2>;
2346:   defm VFENEZB : BinaryVRRbSPair<"vfenezb", 0xE781, int_s390_vfenezb,
2347:                                  z_vfenez_cc, v128b, v128b, 0, 2>;
2348:   defm VFENEZH : BinaryVRRbSPair<"vfenezh", 0xE781, int_s390_vfenezh,
2349:                                  z_vfenez_cc, v128h, v128h, 1, 2>;
2350:   defm VFENEZF : BinaryVRRbSPair<"vfenezf", 0xE781, int_s390_vfenezf,
2351:                                  z_vfenez_cc, v128f, v128f, 2, 2>;
2352: 
2353:   defm VISTR  : UnaryExtraVRRaSPairGeneric<"vistr", 0xE75C>;
2354:   defm VISTRB : UnaryExtraVRRaSPair<"vistrb", 0xE75C, int_s390_vistrb,
2355:                                     z_vistr_cc, v128b, v128b, 0>;
2356:   defm VISTRH : UnaryExtraVRRaSPair<"vistrh", 0xE75C, int_s390_vistrh,
2357:                                     z_vistr_cc, v128h, v128h, 1>;
2358:   defm VISTRF : UnaryExtraVRRaSPair<"vistrf", 0xE75C, int_s390_vistrf,
2359:                                     z_vistr_cc, v128f, v128f, 2>;
2360: 
```
- **EN**: This block declares or refines TableGen records such as `VFAEZF`, `VFEE`, `VFEEB`, `VFEEH`, `VFEEF`, `VFEEZB`.
- **CN**: 该代码块声明或细化了 `VFAEZF`, `VFEE`, `VFEEB`, `VFEEH`, `VFEEF`, `VFEEZB` 等 TableGen 记录。

### Lines 2361-2400 / 第 2361-2400 行
```tablegen
2361:   defm VSTRC  : QuaternaryOptVRRdSPairGeneric<"vstrc", 0xE78A>;
2362:   defm VSTRCB : QuaternaryOptVRRdSPair<"vstrcb", 0xE78A, int_s390_vstrcb,
2363:                                        z_vstrc_cc, v128b, v128b, 0>;
2364:   defm VSTRCH : QuaternaryOptVRRdSPair<"vstrch", 0xE78A, int_s390_vstrch,
2365:                                        z_vstrc_cc, v128h, v128h, 1>;
2366:   defm VSTRCF : QuaternaryOptVRRdSPair<"vstrcf", 0xE78A, int_s390_vstrcf,
2367:                                        z_vstrc_cc, v128f, v128f, 2>;
2368:   defm VSTRCZB : QuaternaryOptVRRdSPair<"vstrczb", 0xE78A, int_s390_vstrczb,
2369:                                         z_vstrcz_cc, v128b, v128b, 0, 2>;
2370:   defm VSTRCZH : QuaternaryOptVRRdSPair<"vstrczh", 0xE78A, int_s390_vstrczh,
2371:                                         z_vstrcz_cc, v128h, v128h, 1, 2>;
2372:   defm VSTRCZF : QuaternaryOptVRRdSPair<"vstrczf", 0xE78A, int_s390_vstrczf,
2373:                                         z_vstrcz_cc, v128f, v128f, 2, 2>;
2374: }
2375: 
2376: let Predicates = [FeatureVectorEnhancements2] in {
2377:   defm VSTRS  : TernaryExtraVRRdGeneric<"vstrs", 0xE78B>;
2378:   defm VSTRSB : TernaryExtraVRRd<"vstrsb", 0xE78B,
2379:                                  z_vstrs_cc, v128b, v128b, 0>;
2380:   defm VSTRSH : TernaryExtraVRRd<"vstrsh", 0xE78B,
2381:                                  z_vstrs_cc, v128b, v128h, 1>;
2382:   defm VSTRSF : TernaryExtraVRRd<"vstrsf", 0xE78B,
2383:                                  z_vstrs_cc, v128b, v128f, 2>;
2384:   let Defs = [CC] in {
2385:     def VSTRSZB : TernaryVRRd<"vstrszb", 0xE78B,
2386:                               z_vstrsz_cc, v128b, v128b, 0, 2>;
2387:     def VSTRSZH : TernaryVRRd<"vstrszh", 0xE78B,
2388:                               z_vstrsz_cc, v128b, v128h, 1, 2>;
2389:     def VSTRSZF : TernaryVRRd<"vstrszf", 0xE78B,
2390:                               z_vstrsz_cc, v128b, v128f, 2, 2>;
2391:   }
2392: }
2393: 
2394: //===----------------------------------------------------------------------===//
2395: // NNP assist instructions
2396: //===----------------------------------------------------------------------===//
2397: 
2398: let Predicates = [FeatureVector, FeatureNNPAssist] in {
2399:   let Uses = [FPC], mayRaiseFPException = 1 in
2400:     def VCFN : UnaryVRRaFloatGeneric<"vcfn", 0xE65D>;
```
- **EN**: This block declares or refines TableGen records such as `VSTRC`, `VSTRCB`, `VSTRCH`, `VSTRCF`, `VSTRCZB`, `VSTRCZH`.
- **CN**: 该代码块声明或细化了 `VSTRC`, `VSTRCB`, `VSTRCH`, `VSTRCF`, `VSTRCZB`, `VSTRCZH` 等 TableGen 记录。

### Lines 2401-2440 / 第 2401-2440 行
```tablegen
2401:   def : Pat<(int_s390_vcfn VR128:$x, imm32zx4_timm:$m),
2402:             (VCFN VR128:$x, 1, imm32zx4:$m)>;
2403: 
2404:   let Uses = [FPC], mayRaiseFPException = 1 in
2405:     def VCLFNL : UnaryVRRaFloatGeneric<"vclfnl", 0xE65E>;
2406:   def : Pat<(int_s390_vclfnls VR128:$x, imm32zx4_timm:$m),
2407:             (VCLFNL VR128:$x, 2, imm32zx4:$m)>;
2408: 
2409:   let Uses = [FPC], mayRaiseFPException = 1 in
2410:     def VCLFNH : UnaryVRRaFloatGeneric<"vclfnh", 0xE656>;
2411:   def : Pat<(int_s390_vclfnhs VR128:$x, imm32zx4_timm:$m),
2412:             (VCLFNH VR128:$x, 2, imm32zx4:$m)>;
2413: 
2414:   let Uses = [FPC], mayRaiseFPException = 1 in
2415:     def VCNF : UnaryVRRaFloatGeneric<"vcnf", 0xE655>;
2416:   def : Pat<(int_s390_vcnf VR128:$x, imm32zx4_timm:$m),
2417:             (VCNF VR128:$x, imm32zx4:$m, 1)>;
2418: 
2419:   let Uses = [FPC], mayRaiseFPException = 1 in
2420:     def VCRNF : BinaryVRRcFloatGeneric<"vcrnf", 0xE675>;
2421:   def : Pat<(int_s390_vcrnfs VR128:$x, VR128:$y, imm32zx4_timm:$m),
2422:             (VCRNF VR128:$x, VR128:$y, imm32zx4:$m, 2)>;
2423: }
2424: 
2425: //===----------------------------------------------------------------------===//
2426: // Packed-decimal instructions
2427: //===----------------------------------------------------------------------===//
2428: 
2429: let Predicates = [FeatureVectorPackedDecimal] in {
2430:   def VLIP : BinaryVRIh<"vlip", 0xE649>;
2431: 
2432:   def VPKZ : BinaryVSI<"vpkz", 0xE634, null_frag, 0>;
2433:   def VUPKZ : StoreLengthVSI<"vupkz", 0xE63C, null_frag, 0>;
2434: 
2435:   let Defs = [CC] in {
2436:     let Predicates = [FeatureVectorPackedDecimalEnhancement] in {
2437:       def VCVBOpt : TernaryVRRi<"vcvb", 0xE650, GR32>;
2438:       def VCVBGOpt : TernaryVRRi<"vcvbg", 0xE652, GR64>;
2439:     }
2440:     def VCVB : BinaryVRRi<"vcvb", 0xE650, GR32>;
```
- **EN**: This block declares or refines TableGen records such as `VCLFNL`, `VCLFNH`, `VCNF`, `VCRNF`, `VLIP`, `VPKZ`.
- **CN**: 该代码块声明或细化了 `VCLFNL`, `VCLFNH`, `VCNF`, `VCRNF`, `VLIP`, `VPKZ` 等 TableGen 记录。

### Lines 2441-2480 / 第 2441-2480 行
```tablegen
2441:     def VCVBG : BinaryVRRi<"vcvbg", 0xE652, GR64>;
2442:     def VCVD : TernaryVRIi<"vcvd", 0xE658, GR32>;
2443:     def VCVDG : TernaryVRIi<"vcvdg", 0xE65A, GR64>;
2444: 
2445:     def VAP : QuaternaryVRIf<"vap", 0xE671>;
2446:     def VSP : QuaternaryVRIf<"vsp", 0xE673>;
2447: 
2448:     def VMP : QuaternaryVRIf<"vmp", 0xE678>;
2449:     def VMSP : QuaternaryVRIf<"vmsp", 0xE679>;
2450: 
2451:     def VDP : QuaternaryVRIf<"vdp", 0xE67A>;
2452:     def VRP : QuaternaryVRIf<"vrp", 0xE67B>;
2453:     def VSDP : QuaternaryVRIf<"vsdp", 0xE67E>;
2454: 
2455:     def VSRP : QuaternaryVRIg<"vsrp", 0xE659>;
2456:     def VPSOP : QuaternaryVRIg<"vpsop", 0xE65B>;
2457: 
2458:     def VTP : TestVRRg<"vtp", 0xE65F>;
2459:     def VCP : CompareVRRh<"vcp", 0xE677>;
2460:   }
2461: }
2462: 
2463: let Predicates = [FeatureVectorPackedDecimalEnhancement2] in {
2464:   def VSCHP : BinaryExtraVRRbGeneric<"vschp", 0xE674>;
2465:   def VSCHSP : BinaryExtraVRRb<"vschsp", 0xE674, 2>;
2466:   def VSCHDP : BinaryExtraVRRb<"vschdp", 0xE674, 3>;
2467:   def VSCHXP : BinaryExtraVRRb<"vschxp", 0xE674, 4>;
2468: 
2469:   def VSCSHP : BinaryVRRb<"vscshp", 0xE67C, null_frag, v128b, v128b>;
2470: 
2471:   def VCSPH : TernaryVRRj<"vcsph", 0xE67D>;
2472: 
2473:   let Defs = [CC] in
2474:     def VCLZDP : BinaryVRRk<"vclzdp", 0xE651>;
2475: 
2476:   let Defs = [CC] in
2477:     def VSRPR : QuaternaryVRIf<"vsrpr", 0xE672>;
2478: 
2479:   let Defs = [CC] in {
2480:     def VPKZR : QuaternaryVRIf<"vpkzr", 0xE670>;
```
- **EN**: This block declares or refines TableGen records such as `VCVBG`, `VCVD`, `VCVDG`, `VAP`, `VSP`, `VMP`.
- **CN**: 该代码块声明或细化了 `VCVBG`, `VCVD`, `VCVDG`, `VAP`, `VSP`, `VMP` 等 TableGen 记录。

### Lines 2481-2495 / 第 2481-2495 行
```tablegen
2481:     def VUPKZH : BinaryVRRk<"vupkzh", 0xE654>;
2482:     def VUPKZL : BinaryVRRk<"vupkzl", 0xE65C>;
2483:   }
2484: }
2485: 
2486: let Predicates = [FeatureVectorPackedDecimalEnhancement3] in {
2487:   def VCVBQ : BinaryVRRk<"vcvbq", 0xE64E>;
2488:   let Defs = [CC] in
2489:     def VCVDQ : TernaryVRIj<"vcvdq", 0xE64A>;
2490: 
2491:   let Defs = [CC] in {
2492:     def VTPOpt : TestExtraVRRg<"vtp", 0xE65F>;
2493:     def VTZ : TestExtraVRIl<"vtz", 0xE67F>;
2494:   }
2495: }
```
- **EN**: This block declares or refines TableGen records such as `VUPKZH`, `VUPKZL`, `VCVBQ`, `VCVDQ`, `VTPOpt`, `VTZ`.
- **CN**: 该代码块声明或细化了 `VUPKZH`, `VUPKZL`, `VCVBQ`, `VCVDQ`, `VTPOpt`, `VTZ` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
