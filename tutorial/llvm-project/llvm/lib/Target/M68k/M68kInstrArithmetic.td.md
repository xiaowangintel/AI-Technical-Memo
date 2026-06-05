# M68kInstrArithmetic.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrArithmetic.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 M68k 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //===-- M68kInstrArithmetic.td - Integer Arith Instrs ------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file describes the integer arithmetic instructions in the M68k
  11: /// architecture. Here is the current status of the file:
  12: ///
  13: ///  Machine:
  14: ///
  15: ///    ADD       [~]   ADDA      [~]   ADDI        [~]   ADDQ [ ]   ADDX [~]
  16: ///    CLR       [ ]   CMP       [~]   CMPA        [~]   CMPI [~]   CMPM [ ]
  17: ///    CMP2      [ ]   DIVS/DIVU [~]   DIVSL/DIVUL [ ]   EXT  [~]   EXTB [ ]
  18: ///    MULS/MULU [~]   NEG       [~]   NEGX        [~]   NOT  [~]   SUB  [~]
  19: ///    SUBA      [~]   SUBI      [~]   SUBQ        [ ]   SUBX [~]
  20: ///
  21: ///  Map:
  22: ///
  23: ///   [ ] - was not touched at all
  24: ///   [!] - requires extarnal stuff implemented
  25: ///   [~] - functional implementation
  26: ///   [X] - complete implementation
  27: ///
  28: //===----------------------------------------------------------------------===//
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 29-56 / 第 29-56 行
```tablegen
  29: 
  30: //===----------------------------------------------------------------------===//
  31: // OPMODE Encoding
  32: //===----------------------------------------------------------------------===//
  33: class MxOpModeEncoding<bits<3> encoding> {
  34:   bits<3> Value = encoding;
  35: }
  36: 
  37: // op EA, Dn
  38: def MxOpMode8_d_EA  : MxOpModeEncoding<0b000>;
  39: def MxOpMode16_d_EA : MxOpModeEncoding<0b001>;
  40: def MxOpMode32_d_EA : MxOpModeEncoding<0b010>;
  41: 
  42: // op Dn, EA
  43: def MxOpMode8_EA_d  : MxOpModeEncoding<0b100>;
  44: def MxOpMode16_EA_d : MxOpModeEncoding<0b101>;
  45: def MxOpMode32_EA_d : MxOpModeEncoding<0b110>;
  46: 
  47: // op EA, An
  48: def MxOpMode16_a_EA : MxOpModeEncoding<0b011>;
  49: def MxOpMode32_a_EA : MxOpModeEncoding<0b111>;
  50: 
  51: 
  52: //===----------------------------------------------------------------------===//
  53: // Encoding
  54: //===----------------------------------------------------------------------===//
  55: 
  56: let Defs = [CCR] in {
```
- **EN**: This block declares or refines TableGen records such as `MxOpModeEncoding`, `MxOpMode8_d_EA`, `MxOpMode16_d_EA`, `MxOpMode32_d_EA`, `MxOpMode8_EA_d`, `MxOpMode16_EA_d`.
- **CN**: 该代码块声明或细化了 `MxOpModeEncoding`, `MxOpMode8_d_EA`, `MxOpMode16_d_EA`, `MxOpMode32_d_EA`, `MxOpMode8_EA_d`, `MxOpMode16_EA_d` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57: let Constraints = "$src = $dst" in {
  58: 
  59: /// Encoding for Normal forms
  60: /// ----------------------------------------------------
  61: ///  F  E  D  C | B  A  9 | 8  7  6 | 5  4  3 | 2  1  0
  62: /// ----------------------------------------------------
  63: ///             |         |         | EFFECTIVE ADDRESS
  64: ///  x  x  x  x |   REG   | OP MODE |   MODE  |   REG
  65: /// ----------------------------------------------------
  66: 
  67: // $reg, $ccr <- $reg op $reg
  68: class MxBiArOp_R_RR_xEA<string MN, SDNode NODE, MxType DST_TYPE, MxType SRC_TYPE,
  69:                         bits<4> CMD>
  70:     : MxInst<(outs DST_TYPE.ROp:$dst), (ins DST_TYPE.ROp:$src, SRC_TYPE.ROp:$opd),
  71:              MN#"."#DST_TYPE.Prefix#"\t$opd, $dst",
  72:              [(set DST_TYPE.VT:$dst, CCR, (NODE DST_TYPE.VT:$src, SRC_TYPE.VT:$opd))]> {
  73:   let Inst = (descend
  74:     CMD, (operand "$dst", 3),
  75:     !cast<MxOpModeEncoding>("MxOpMode"#DST_TYPE.Size#"_"#DST_TYPE.RLet#"_EA").Value,
  76:     !cond(
  77:       !eq(SRC_TYPE.RLet, "r") : (descend 0b00, (operand "$opd", 4)),
  78:       !eq(SRC_TYPE.RLet, "d") : (descend 0b000, (operand "$opd", 3))
  79:     )
  80:   );
  81: }
  82: 
  83: /// This Op is similar to the one above except it uses reversed opmode, some
  84: /// commands(e.g. eor) do not support dEA or rEA modes and require EAd for
```
- **EN**: This block declares or refines TableGen records such as `MxBiArOp_R_RR_xEA`.
- **CN**: 该代码块声明或细化了 `MxBiArOp_R_RR_xEA` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85: /// register only operations.
  86: /// NOTE when using dd commands it is irrelevant which opmode to use(as it seems)
  87: /// but some opcodes support address register and some do not which creates this
  88: /// mess.
  89: class MxBiArOp_R_RR_EAd<string MN, SDNode NODE, MxType TYPE, bits<4> CMD>
  90:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src, TYPE.ROp:$opd),
  91:              MN#"."#TYPE.Prefix#"\t$opd, $dst",
  92:              [(set TYPE.VT:$dst, CCR, (NODE TYPE.VT:$src, TYPE.VT:$opd))]> {
  93:   let Inst = (descend
  94:     CMD, (operand "$opd", 3),
  95:     !cast<MxOpModeEncoding>("MxOpMode"#TYPE.Size#"_EA_"#TYPE.RLet).Value,
  96:     /*Destination can only be a data register*/
  97:     /*MODE*/0b000,
  98:     /*REGISTER*/(operand "$dst", 3));
  99: }
 100: 
 101: let mayLoad = 1 in
 102: class MxBiArOp_R_RM<string MN, SDNode NODE, MxType TYPE, MxOperand OPD, ComplexPattern PAT,
 103:                     bits<4> CMD, MxEncMemOp SRC_ENC>
 104:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src, OPD:$opd),
 105:              MN#"."#TYPE.Prefix#"\t$opd, $dst",
 106:              [(set TYPE.VT:$dst, CCR, (NODE TYPE.VT:$src, (TYPE.Load PAT:$opd)))]> {
 107:   let Inst = (ascend
 108:     (descend CMD, (operand "$dst", 3),
 109:              !cast<MxOpModeEncoding>("MxOpMode"#TYPE.Size#"_"#TYPE.RLet#"_EA").Value,
 110:              SRC_ENC.EA),
 111:     SRC_ENC.Supplement
 112:   );
```
- **EN**: This block declares or refines TableGen records such as `MxBiArOp_R_RR_EAd`, `MxBiArOp_R_RM`.
- **CN**: 该代码块声明或细化了 `MxBiArOp_R_RR_EAd`, `MxBiArOp_R_RM` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113: }
 114: 
 115: /// Encoding for Immediate forms
 116: /// ---------------------------------------------------
 117: ///  F  E  D  C  B  A  9  8 | 7  6 | 5  4  3 | 2  1  0
 118: /// ---------------------------------------------------
 119: ///                         |      | EFFECTIVE ADDRESS
 120: ///  x  x  x  x  x  x  x  x | SIZE |   MODE  |   REG
 121: /// ---------------------------------------------------
 122: ///     16-BIT WORD DATA    |     8-BIT BYTE DATA
 123: /// ---------------------------------------------------
 124: ///                 32-BIT LONG DATA
 125: /// ---------------------------------------------------
 126: /// NOTE It is used to store an immediate to memory, imm-to-reg are handled with
 127: /// normal version
 128: 
 129: // $reg <- $reg op $imm
 130: class MxBiArOp_R_RI_xEA<string MN, SDNode NODE, MxType TYPE, bits<4> CMD>
 131:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src, TYPE.IOp:$opd),
 132:              MN#"."#TYPE.Prefix#"\t$opd, $dst",
 133:              [(set TYPE.VT:$dst, CCR, (NODE TYPE.VT:$src, TYPE.IPat:$opd))]> {
 134:   let Inst = (ascend
 135:     (descend CMD, (operand "$dst", 3),
 136:              !cast<MxOpModeEncoding>("MxOpMode"#TYPE.Size#"_"#TYPE.RLet#"_EA").Value,
 137:              MxEncAddrMode_i<"opd", TYPE.Size>.EA),
 138:     MxEncAddrMode_i<"opd", TYPE.Size>.Supplement
 139:   );
 140: }
```
- **EN**: This block declares or refines TableGen records such as `MxBiArOp_R_RI_xEA`.
- **CN**: 该代码块声明或细化了 `MxBiArOp_R_RI_xEA` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: 
 142: // Again, there are two ways to write an immediate to Dn register either dEA
 143: // opmode or using *I encoding, and again some instructions also support address
 144: // registers some do not.
 145: class MxBiArOp_R_RI<string MN, SDNode NODE, MxType TYPE, bits<4> CMD>
 146:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src, TYPE.IOp:$opd),
 147:              MN#"i."#TYPE.Prefix#"\t$opd, $dst",
 148:              [(set TYPE.VT:$dst, CCR, (NODE TYPE.VT:$src, TYPE.IPat:$opd))]> {
 149:   let Inst = (ascend
 150:     (descend 0b0000, CMD,
 151:              !cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
 152:              // The destination cannot be address register, so it's always
 153:              // the MODE for data register direct mode.
 154:              /*MODE*/0b000,
 155:              /*REGISTER*/(operand "$dst", 3)),
 156:     // Source (i.e. immediate value) encoding
 157:     MxEncAddrMode_i<"opd", TYPE.Size>.Supplement
 158:   );
 159: }
 160: } // Constraints
 161: 
 162: let mayLoad = 1, mayStore = 1 in {
 163: 
 164: // FIXME MxBiArOp_FMR/FMI cannot consume CCR from MxAdd/MxSub which leads for
 165: // MxAdd to survive the match and subsequent mismatch.
 166: class MxBiArOp_MR<string MN, MxType TYPE,
 167:                   MxOperand MEMOpd, bits<4> CMD, MxEncMemOp DST_ENC>
 168:     : MxInst<(outs), (ins MEMOpd:$dst, TYPE.ROp:$opd),
```
- **EN**: This block declares or refines TableGen records such as `MxBiArOp_R_RI`, `MxBiArOp_MR`.
- **CN**: 该代码块声明或细化了 `MxBiArOp_R_RI`, `MxBiArOp_MR` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169:              MN#"."#TYPE.Prefix#"\t$opd, $dst", []> {
 170:   let Inst = (ascend
 171:     (descend CMD, (operand "$opd", 3),
 172:              !cast<MxOpModeEncoding>("MxOpMode"#TYPE.Size#"_EA_"#TYPE.RLet).Value,
 173:              DST_ENC.EA),
 174:     DST_ENC.Supplement
 175:   );
 176: }
 177: 
 178: class MxBiArOp_MI<string MN, MxType TYPE,
 179:                   MxOperand MEMOpd, bits<4> CMD, MxEncMemOp DST_ENC>
 180:     : MxInst<(outs), (ins MEMOpd:$dst, TYPE.IOp:$opd),
 181:              MN#"."#TYPE.Prefix#"\t$opd, $dst", []> {
 182:   let Inst = (ascend
 183:     (descend 0b0000, CMD,
 184:              !cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
 185:              DST_ENC.EA),
 186:     // Source (i.e. immediate value) encoding
 187:     MxEncAddrMode_i<"opd", TYPE.Size>.Supplement,
 188:     // Destination encoding
 189:     DST_ENC.Supplement
 190:   );
 191: }
 192: } // mayLoad, mayStore
 193: } // Defs = [CCR]
 194: 
 195: multiclass MxBiArOp_DF<string MN, SDNode NODE, bit isComm,
 196:                        bits<4> CMD, bits<4> CMDI> {
```
- **EN**: This block declares or refines TableGen records such as `MxBiArOp_MI`, `MxBiArOp_DF`.
- **CN**: 该代码块声明或细化了 `MxBiArOp_MI`, `MxBiArOp_DF` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197: 
 198:   foreach SZ = [8, 16, 32] in {
 199:     // op $mem, $reg
 200:     def NAME#SZ#"dk"  : MxBiArOp_R_RM<MN, NODE,
 201:                                       !cast<MxType>("MxType"#SZ#"d"),
 202:                                       !cast<MxType>("MxType"#SZ).KOp,
 203:                                       !cast<MxType>("MxType"#SZ).KPat,
 204:                                       CMD, MxEncAddrMode_k<"opd">>;
 205: 
 206:     def NAME#SZ#"dq"  : MxBiArOp_R_RM<MN, NODE,
 207:                                       !cast<MxType>("MxType"#SZ#"d"),
 208:                                       !cast<MxType>("MxType"#SZ).QOp,
 209:                                       !cast<MxType>("MxType"#SZ).QPat,
 210:                                       CMD, MxEncAddrMode_q<"opd">>;
 211: 
 212:     def NAME#SZ#"dp"  : MxBiArOp_R_RM<MN, NODE,
 213:                                       !cast<MxType>("MxType"#SZ#"d"),
 214:                                       !cast<MxType>("MxType"#SZ).POp,
 215:                                       !cast<MxType>("MxType"#SZ).PPat,
 216:                                       CMD, MxEncAddrMode_p<"opd">>;
 217: 
 218:     def NAME#SZ#"df"  : MxBiArOp_R_RM<MN, NODE,
 219:                                       !cast<MxType>("MxType"#SZ#"d"),
 220:                                       !cast<MxType>("MxType"#SZ).FOp,
 221:                                       !cast<MxType>("MxType"#SZ).FPat,
 222:                                       CMD, MxEncAddrMode_f<"opd">>;
 223: 
 224:     def NAME#SZ#"dj"  : MxBiArOp_R_RM<MN, NODE,
```
- **EN**: This block declares or refines TableGen records such as `NAME`, `NAME`, `NAME`, `NAME`, `NAME`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `NAME`, `NAME`, `NAME`, `NAME`, `NAME` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 225-252 / 第 225-252 行
```tablegen
 225:                                       !cast<MxType>("MxType"#SZ#"d"),
 226:                                       !cast<MxType>("MxType"#SZ).JOp,
 227:                                       !cast<MxType>("MxType"#SZ).JPat,
 228:                                       CMD, MxEncAddrMode_j<"opd">>;
 229:     // op $imm, $reg
 230:     def NAME#SZ#"di"  : MxBiArOp_R_RI_xEA<MN, NODE,
 231:                                           !cast<MxType>("MxType"#SZ#"d"),
 232:                                           CMD>;
 233:     // op $reg, $mem
 234:     def NAME#SZ#"pd"  : MxBiArOp_MR<MN,
 235:                                     !cast<MxType>("MxType"#SZ#"d"),
 236:                                     !cast<MxType>("MxType"#SZ).POp,
 237:                                     CMD, MxEncAddrMode_p<"dst">>;
 238: 
 239:     def NAME#SZ#"fd"  : MxBiArOp_MR<MN,
 240:                                     !cast<MxType>("MxType"#SZ#"d"),
 241:                                     !cast<MxType>("MxType"#SZ).FOp,
 242:                                     CMD, MxEncAddrMode_f<"dst">>;
 243: 
 244:     def NAME#SZ#"jd"  : MxBiArOp_MR<MN,
 245:                                     !cast<MxType>("MxType"#SZ#"d"),
 246:                                     !cast<MxType>("MxType"#SZ).JOp,
 247:                                     CMD, MxEncAddrMode_j<"dst">>;
 248:     // op $imm, $mem
 249:     def NAME#SZ#"pi"  : MxBiArOp_MI<MN,
 250:                                     !cast<MxType>("MxType"#SZ),
 251:                                     !cast<MxType>("MxType"#SZ).POp,
 252:                                     CMDI, MxEncAddrMode_p<"dst">>;
```
- **EN**: This block declares or refines TableGen records such as `NAME`, `NAME`, `NAME`, `NAME`, `NAME`.
- **CN**: 该代码块声明或细化了 `NAME`, `NAME`, `NAME`, `NAME`, `NAME` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253: 
 254:     def NAME#SZ#"fi"  : MxBiArOp_MI<MN,
 255:                                     !cast<MxType>("MxType"#SZ),
 256:                                     !cast<MxType>("MxType"#SZ).FOp,
 257:                                     CMDI, MxEncAddrMode_f<"dst">>;
 258: 
 259:     def NAME#SZ#"ji"  : MxBiArOp_MI<MN,
 260:                                     !cast<MxType>("MxType"#SZ),
 261:                                     !cast<MxType>("MxType"#SZ).JOp,
 262:                                     CMDI, MxEncAddrMode_j<"dst">>;
 263:     // op $reg, $reg
 264:     let isCommutable = isComm in
 265:     def NAME#SZ#"dd" : MxBiArOp_R_RR_xEA<MN, NODE,
 266:                                          !cast<MxType>("MxType"#SZ#"d"),
 267:                                          !cast<MxType>("MxType"#SZ#"d"),
 268:                                          CMD>;
 269:   } // foreach SZ
 270: 
 271:   foreach SZ = [16, 32] in
 272:   def NAME#SZ#"dr" : MxBiArOp_R_RR_xEA<MN, NODE,
 273:                                        !cast<MxType>("MxType"#SZ#"d"),
 274:                                        !cast<MxType>("MxType"#SZ#"r"),
 275:                                        CMD>;
 276: 
 277: } // MxBiArOp_DF
 278: 
 279: 
 280: // These special snowflakes allowed to match address registers but since *A
```
- **EN**: This block declares or refines TableGen records such as `NAME`, `NAME`, `NAME`, `NAME`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `NAME`, `NAME`, `NAME`, `NAME` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-308 / 第 281-308 行
```tablegen
 281: // operations do not produce CCR we should not match them against Mx nodes that
 282: // produce it.
 283: let Pattern = [(null_frag)] in
 284: multiclass MxBiArOp_AF<string MN, SDNode NODE, bits<4> CMD> {
 285: 
 286:   def NAME#"32ak" : MxBiArOp_R_RM<MN, NODE, MxType32a, MxType32.KOp, MxType32.KPat,
 287:                                   CMD, MxEncAddrMode_k<"opd">>;
 288:   def NAME#"32aq" : MxBiArOp_R_RM<MN, NODE, MxType32a, MxType32.QOp, MxType32.QPat,
 289:                                   CMD, MxEncAddrMode_q<"opd">>;
 290:   def NAME#"32af" : MxBiArOp_R_RM<MN, NODE, MxType32a, MxType32.FOp, MxType32.FPat,
 291:                                   CMD, MxEncAddrMode_f<"opd">>;
 292:   def NAME#"32ap" : MxBiArOp_R_RM<MN, NODE, MxType32a, MxType32.POp, MxType32.PPat,
 293:                                   CMD, MxEncAddrMode_p<"opd">>;
 294:   def NAME#"32aj" : MxBiArOp_R_RM<MN, NODE, MxType32a, MxType32.JOp, MxType32.JPat,
 295:                                   CMD, MxEncAddrMode_j<"opd">>;
 296:   def NAME#"32ab" : MxBiArOp_R_RM<MN, NODE, MxType32a, MxType32.BOp, MxType32.BPat,
 297:                                   CMD, MxEncAddrMode_abs<"opd", true>>;
 298:   def NAME#"32ai" : MxBiArOp_R_RI_xEA<MN, NODE, MxType32a, CMD>;
 299: 
 300:   def NAME#"32ar" : MxBiArOp_R_RR_xEA<MN, NODE, MxType32a, MxType32r, CMD>;
 301: 
 302: } // MxBiArOp_AF
 303: 
 304: // NOTE These naturally produce CCR
 305: 
 306: //===----------------------------------------------------------------------===//
 307: // Add/Sub
 308: //===----------------------------------------------------------------------===//
```
- **EN**: This block declares or refines TableGen records such as `MxBiArOp_AF`, `NAME`, `NAME`, `NAME`, `NAME`, `NAME`.
- **CN**: 该代码块声明或细化了 `MxBiArOp_AF`, `NAME`, `NAME`, `NAME`, `NAME`, `NAME` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309: 
 310: defm ADD : MxBiArOp_DF<"add",  MxAdd, 1, 0xD, 0x6>;
 311: defm ADD : MxBiArOp_AF<"adda", MxAdd, 0xD>;
 312: defm SUB : MxBiArOp_DF<"sub",  MxSub, 0, 0x9, 0x4>;
 313: defm SUB : MxBiArOp_AF<"suba", MxSub, 0x9>;
 314: 
 315: // This pattern is used to enable the instruction selector to select ADD32ab
 316: // for global values that are allocated in thread-local storage, i.e.:
 317: //   t8: i32 = ISD::ADD GLOBAL_OFFSET_TABLE, TargetGlobalTLSAddress:i32<ptr @myvar>
 318: //     ====>
 319: //   t8: i32,i8 = ADD32ab GLOBAL_OFFSET_TABLE, TargetGlobalTLSAddress:i32<ptr @myvar>
 320: def : Pat<(add MxARD32:$src, tglobaltlsaddr:$opd), (ADD32ab MxARD32:$src, MxAL32:$opd)>;
 321: 
 322: let Uses = [CCR], Defs = [CCR] in {
 323: let Constraints = "$src = $dst" in {
 324: 
 325: /// Encoding for Extended forms
 326: /// ------------------------------------------------------
 327: ///  F  E  D  C | B  A  9 | 8 | 7  6 | 5  4 | 3 | 2  1  0
 328: /// ------------------------------------------------------
 329: ///  x  x  x  x |  REG Rx | 1 | SIZE | 0  0 | M |  REG Ry
 330: /// ------------------------------------------------------
 331: /// Rx - destination
 332: /// Ry - source
 333: /// M  - address mode switch
 334: 
 335: // $reg, ccr <- $reg op $reg op ccr
 336: class MxBiArOp_R_RRX<string MN, SDNode NODE, MxType TYPE, bits<4> CMD>
```
- **EN**: This block declares or refines TableGen records such as `ADD`, `ADD`, `SUB`, `SUB`, `MxBiArOp_R_RRX`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 该代码块声明或细化了 `ADD`, `ADD`, `SUB`, `SUB`, `MxBiArOp_R_RRX` 等 TableGen 记录。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 337-364 / 第 337-364 行
```tablegen
 337:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src, TYPE.ROp:$opd),
 338:              MN#"."#TYPE.Prefix#"\t$opd, $dst",
 339:              [(set TYPE.VT:$dst, CCR, (NODE TYPE.VT:$src, TYPE.VT:$opd, CCR))]> {
 340:   let Inst = (descend CMD,
 341:     // Destination register
 342:     (operand "$dst", 3),
 343:     0b1,
 344:     // SIZE
 345:     !cond(!eq(TYPE.Size, 8): 0b00,
 346:           !eq(TYPE.Size, 16): 0b01,
 347:           !eq(TYPE.Size, 32): 0b10),
 348:     0b00, /*R/M*/0b0,
 349:     // Source register
 350:     (operand "$opd", 3)
 351:   );
 352: }
 353: } // Constraints
 354: } // Uses, Defs
 355: 
 356: multiclass MxBiArOp_RFF<string MN, SDNode NODE, bit isComm, bits<4> CMD> {
 357: 
 358: let isCommutable = isComm in {
 359:   foreach SZ = [8, 16, 32] in
 360:   def NAME#SZ#"dd"  : MxBiArOp_R_RRX<MN, NODE, !cast<MxType>("MxType"#SZ#"d"), CMD>;
 361: } // isComm
 362: 
 363: } // MxBiArOp_RFF
 364: 
```
- **EN**: This block declares or refines TableGen records such as `MxBiArOp_RFF`, `NAME`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxBiArOp_RFF`, `NAME` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 365-392 / 第 365-392 行
```tablegen
 365: // NOTE These consume and produce CCR
 366: defm ADDX : MxBiArOp_RFF<"addx", MxAddX, 1, 0xD>;
 367: defm SUBX : MxBiArOp_RFF<"subx", MxSubX, 0, 0x9>;
 368: 
 369: 
 370: //===----------------------------------------------------------------------===//
 371: // And/Xor/Or
 372: //===----------------------------------------------------------------------===//
 373: 
 374: defm AND : MxBiArOp_DF<"and", MxAnd, 1, 0xC, 0x2>;
 375: defm OR  : MxBiArOp_DF<"or",  MxOr,  1, 0x8, 0x0>;
 376: 
 377: multiclass MxBiArOp_DF_EAd<string MN, SDNode NODE, bits<4> CMD, bits<4> CMDI> {
 378:   foreach SZ = [8, 16, 32] in {
 379:     let isCommutable = 1 in
 380:     def NAME#SZ#"dd"  : MxBiArOp_R_RR_EAd<MN, NODE,
 381:                                           !cast<MxType>("MxType"#SZ#"d"),
 382:                                           CMD>;
 383: 
 384:     def NAME#SZ#"di"  : MxBiArOp_R_RI<MN, NODE,
 385:                                       !cast<MxType>("MxType"#SZ#"d"),
 386:                                       CMDI>;
 387:   } // foreach SZ
 388: } // MxBiArOp_DF_EAd
 389: 
 390: defm XOR : MxBiArOp_DF_EAd<"eor", MxXor, 0xB, 0xA>;
 391: 
 392: 
```
- **EN**: This block declares or refines TableGen records such as `ADDX`, `SUBX`, `AND`, `OR`, `MxBiArOp_DF_EAd`, `NAME`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `ADDX`, `SUBX`, `AND`, `OR`, `MxBiArOp_DF_EAd`, `NAME` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 393-420 / 第 393-420 行
```tablegen
 393: //===----------------------------------------------------------------------===//
 394: // CMP
 395: //===----------------------------------------------------------------------===//
 396: 
 397: let Defs = [CCR] in {
 398: class MxCmp_RR<MxType LHS_TYPE, MxType RHS_TYPE = LHS_TYPE>
 399:     : MxInst<(outs), (ins LHS_TYPE.ROp:$lhs, RHS_TYPE.ROp:$rhs),
 400:              "cmp."#RHS_TYPE.Prefix#"\t$lhs, $rhs",
 401:              [(set CCR, (MxCmp LHS_TYPE.VT:$lhs, RHS_TYPE.VT:$rhs))]> {
 402:   let Inst = (descend 0b1011,
 403:     // REGISTER
 404:     (operand "$rhs", 3),
 405:     // OPMODE
 406:     !cast<MxOpModeEncoding>("MxOpMode"#RHS_TYPE.Size#"_"#RHS_TYPE.RLet#"_EA").Value,
 407:     // MODE without last bit
 408:     0b00,
 409:     // REGISTER prefixed by D/A bit
 410:     (operand "$lhs", 4)
 411:   );
 412: }
 413: 
 414: class MxCmp_RI<MxType TYPE>
 415:     : MxInst<(outs), (ins TYPE.IOp:$imm, TYPE.ROp:$reg),
 416:              "cmpi."#TYPE.Prefix#"\t$imm, $reg",
 417:              [(set CCR, (MxCmp TYPE.IPat:$imm, TYPE.VT:$reg))]> {
 418:   let Inst = (ascend
 419:     (descend 0b00001100,
 420:              !cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
```
- **EN**: This block declares or refines TableGen records such as `MxCmp_RR`, `MxCmp_RI`.
- **CN**: 该代码块声明或细化了 `MxCmp_RR`, `MxCmp_RI` 等 TableGen 记录。

### Lines 421-448 / 第 421-448 行
```tablegen
 421:              // The destination cannot be address register, so it's always
 422:              // the MODE for data register direct mode.
 423:              /*MODE*/0b000,
 424:              /*REGISTER*/(operand "$reg", 3)),
 425:     // Source (i.e. immediate value) encoding
 426:     MxEncAddrMode_i<"imm", TYPE.Size>.Supplement
 427:   );
 428: }
 429: 
 430: let mayLoad = 1 in {
 431: 
 432: class MxCmp_MI<MxType TYPE, MxOperand MEMOpd, ComplexPattern MEMPat,
 433:                MxEncMemOp MEM_ENC>
 434:     : MxInst<(outs), (ins TYPE.IOp:$imm, MEMOpd:$mem),
 435:              "cmpi."#TYPE.Prefix#"\t$imm, $mem",
 436:              [(set CCR, (MxCmp TYPE.IPat:$imm, (load MEMPat:$mem)))]> {
 437:   let Inst = (ascend
 438:     (descend 0b00001100,
 439:              !cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
 440:              MEM_ENC.EA),
 441:     // Source (i.e. immediate value) encoding
 442:     MxEncAddrMode_i<"imm", TYPE.Size>.Supplement,
 443:     // Destination (i.e. memory operand) encoding
 444:     MEM_ENC.Supplement
 445:   );
 446: }
 447: 
 448: // FIXME: What about abs.W?
```
- **EN**: This block declares or refines TableGen records such as `MxCmp_MI`.
- **CN**: 该代码块声明或细化了 `MxCmp_MI` 等 TableGen 记录。

### Lines 449-476 / 第 449-476 行
```tablegen
 449: class MxCmp_BI<MxType TYPE>
 450:     : MxInst<(outs), (ins TYPE.IOp:$imm, MxAL32:$abs),
 451:              "cmpi."#TYPE.Prefix#"\t$imm, $abs",
 452:              [(set CCR, (MxCmp TYPE.IPat:$imm,
 453:                                (load (i32 (MxWrapper tglobaladdr:$abs)))))]> {
 454:   defvar AbsEncoding = MxEncAddrMode_abs<"abs", true>;
 455:   let Inst = (ascend
 456:     (descend 0b00001100,
 457:              !cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
 458:              AbsEncoding.EA),
 459:     // Source (i.e. immediate value) encoding
 460:     MxEncAddrMode_i<"imm", TYPE.Size>.Supplement,
 461:     // Destination (i.e. memory operand) encoding
 462:     AbsEncoding.Supplement
 463:   );
 464: }
 465: 
 466: class MxCmp_RM<MxType TYPE, MxOperand MEMOpd, ComplexPattern MEMPat,
 467:                MxEncMemOp MEM_ENC>
 468:     : MxInst<(outs), (ins TYPE.ROp:$reg, MEMOpd:$mem),
 469:              "cmp."#TYPE.Prefix#"\t$mem, $reg",
 470:              [(set CCR, (MxCmp (load MEMPat:$mem), TYPE.ROp:$reg))]> {
 471:   let Inst = (ascend
 472:     (descend 0b1011,
 473:       // REGISTER
 474:       (operand "$reg", 3),
 475:       // OPMODE
 476:       !cast<MxOpModeEncoding>("MxOpMode"#TYPE.Size#"_d_EA").Value,
```
- **EN**: This block declares or refines TableGen records such as `MxCmp_BI`, `MxCmp_RM`.
- **CN**: 该代码块声明或细化了 `MxCmp_BI`, `MxCmp_RM` 等 TableGen 记录。

### Lines 477-504 / 第 477-504 行
```tablegen
 477:       MEM_ENC.EA),
 478:     MEM_ENC.Supplement
 479:   );
 480: }
 481: } // let mayLoad = 1
 482: 
 483: } // let Defs = [CCR]
 484: 
 485: multiclass MMxCmp_RM<MxType TYPE> {
 486:   def NAME#TYPE.KOp.Letter : MxCmp_RM<TYPE, TYPE.KOp, TYPE.KPat, MxEncAddrMode_k<"mem">>;
 487:   def NAME#TYPE.QOp.Letter : MxCmp_RM<TYPE, TYPE.QOp, TYPE.QPat, MxEncAddrMode_q<"mem">>;
 488:   def NAME#TYPE.POp.Letter : MxCmp_RM<TYPE, TYPE.POp, TYPE.PPat, MxEncAddrMode_p<"mem">>;
 489:   def NAME#TYPE.FOp.Letter : MxCmp_RM<TYPE, TYPE.FOp, TYPE.FPat, MxEncAddrMode_f<"mem">>;
 490:   def NAME#TYPE.JOp.Letter : MxCmp_RM<TYPE, TYPE.JOp, TYPE.JPat, MxEncAddrMode_j<"mem">>;
 491: }
 492: 
 493: multiclass MMxCmp_MI<MxType TYPE> {
 494:   def NAME#TYPE.KOp.Letter#"i" : MxCmp_MI<TYPE, TYPE.KOp, TYPE.KPat,
 495:                                           MxEncAddrMode_k<"mem">>;
 496:   def NAME#TYPE.QOp.Letter#"i" : MxCmp_MI<TYPE, TYPE.QOp, TYPE.QPat,
 497:                                           MxEncAddrMode_q<"mem">>;
 498:   def NAME#TYPE.POp.Letter#"i" : MxCmp_MI<TYPE, TYPE.POp, TYPE.PPat,
 499:                                           MxEncAddrMode_p<"mem">>;
 500:   def NAME#TYPE.FOp.Letter#"i" : MxCmp_MI<TYPE, TYPE.FOp, TYPE.FPat,
 501:                                           MxEncAddrMode_f<"mem">>;
 502:   def NAME#TYPE.JOp.Letter#"i" : MxCmp_MI<TYPE, TYPE.JOp, TYPE.JPat,
 503:                                           MxEncAddrMode_j<"mem">>;
 504: }
```
- **EN**: This block declares or refines TableGen records such as `MMxCmp_RM`, `NAME`, `NAME`, `NAME`, `NAME`, `NAME`.
- **CN**: 该代码块声明或细化了 `MMxCmp_RM`, `NAME`, `NAME`, `NAME`, `NAME`, `NAME` 等 TableGen 记录。

### Lines 505-532 / 第 505-532 行
```tablegen
 505: 
 506: foreach S = [8, 16, 32] in {
 507:   def CMP#S#di : MxCmp_RI<!cast<MxType>("MxType"#S#"d")>;
 508:   def CMP#S#bi : MxCmp_BI<!cast<MxType>("MxType"#S#"d")>;
 509: } // foreach
 510: 
 511: def CMP8dd : MxCmp_RR<MxType8d>;
 512: foreach S = [16, 32] in {
 513:   def CMP#S#dr : MxCmp_RR<!cast<MxType>("MxType"#S#"r"),
 514:                           !cast<MxType>("MxType"#S#"d")>;
 515: }
 516: 
 517: // cmp mem, Dn
 518: defm CMP8d  : MMxCmp_RM<MxType8d>;
 519: defm CMP16d : MMxCmp_RM<MxType16d>;
 520: defm CMP32d : MMxCmp_RM<MxType32d>;
 521: 
 522: // cmp #imm, mem
 523: defm CMP8  : MMxCmp_MI<MxType8d>;
 524: defm CMP16 : MMxCmp_MI<MxType16d>;
 525: defm CMP32 : MMxCmp_MI<MxType32d>;
 526: 
 527: 
 528: //===----------------------------------------------------------------------===//
 529: // EXT
 530: //===----------------------------------------------------------------------===//
 531: 
 532: /// ---------------------------------------------------
```
- **EN**: This block declares or refines TableGen records such as `CMP`, `CMP`, `CMP8dd`, `CMP`, `CMP8d`, `CMP16d`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `CMP`, `CMP`, `CMP8dd`, `CMP`, `CMP8d`, `CMP16d` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 533-560 / 第 533-560 行
```tablegen
 533: ///  F  E  D  C  B  A  9 | 8  7  6 | 5  4  3 | 2  1  0
 534: /// ---------------------------------------------------
 535: ///  0  1  0  0  1  0  0 |  OPMODE | 0  0  0 |   REG
 536: /// ---------------------------------------------------
 537: let Defs = [CCR] in
 538: let Constraints = "$src = $dst" in
 539: class MxExt<MxType TO, MxType FROM>
 540:     : MxInst<(outs TO.ROp:$dst), (ins TO.ROp:$src),
 541:              "ext."#TO.Prefix#"\t$src", []> {
 542:   let Inst = (descend 0b0100100,
 543:     // OPMODE
 544:     !cond(
 545:       // byte -> word
 546:       !and(!eq(FROM.Size, 8), !eq(TO.Size, 16)): 0b010,
 547:       // word -> long
 548:       !and(!eq(FROM.Size, 16), !eq(TO.Size, 32)): 0b011,
 549:       // byte -> long
 550:       !and(!eq(FROM.Size, 8), !eq(TO.Size, 32)): 0b111
 551:     ),
 552:     0b000,
 553:     // REGISTER
 554:     (operand "$src", 3)
 555:   );
 556: }
 557: 
 558: def EXT16 : MxExt<MxType16d, MxType8d>;
 559: def EXT32 : MxExt<MxType32d, MxType16d>;
 560: 
```
- **EN**: This block declares or refines TableGen records such as `MxExt`, `EXT16`, `EXT32`.
- **CN**: 该代码块声明或细化了 `MxExt`, `EXT16`, `EXT32` 等 TableGen 记录。

### Lines 561-588 / 第 561-588 行
```tablegen
 561: def : Pat<(sext_inreg i16:$src, i8),  (EXT16 $src)>;
 562: def : Pat<(sext_inreg i32:$src, i16), (EXT32 $src)>;
 563: def : Pat<(sext_inreg i32:$src, i8),
 564:           (EXT32 (MOVXd32d16 (EXT16 (EXTRACT_SUBREG $src, MxSubRegIndex16Lo))))>;
 565: 
 566: 
 567: //===----------------------------------------------------------------------===//
 568: // DIV/MUL
 569: //===----------------------------------------------------------------------===//
 570: 
 571: /// Word operation:
 572: /// ----------------------------------------------------
 573: ///  F  E  D  C | B  A  9 | 8  7  6 | 5  4  3 | 2  1  0
 574: /// ----------------------------------------------------
 575: ///             |         |         | EFFECTIVE ADDRESS
 576: ///  x  x  x  x |   REG   | OP MODE |   MODE  |   REG
 577: /// ----------------------------------------------------
 578: let Defs = [CCR] in {
 579: let Constraints = "$src = $dst" in {
 580: // $dreg <- $dreg op $dreg
 581: class MxDiMuOp_DD<string MN, bits<4> CMD, bit SIGNED = false,
 582:                   MxOperand DST, MxOperand OPD>
 583:     : MxInst<(outs DST:$dst), (ins DST:$src, OPD:$opd), MN#"\t$opd, $dst", []> {
 584:   let Inst = (descend CMD,
 585:     // REGISTER
 586:     (operand "$dst", 3),
 587:     !if(SIGNED, 0b111, 0b011),
 588:     /*MODE*/0b000, /*REGISTER*/(operand "$opd", 3)
```
- **EN**: This block declares or refines TableGen records such as `MxDiMuOp_DD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxDiMuOp_DD` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```tablegen
 589:   );
 590: }
 591: 
 592: // $dreg <- $dreg op $dreg
 593: class MxDiMuOp_DD_Long<string MN, SDNode NODE, bits<10> CMD, bit SIGNED = false>
 594:     : MxInst<(outs MxDRD32:$dst), (ins MxDRD32:$src, MxDRD32:$opd), MN#"\t$opd, $dst",
 595:              [(set i32:$dst, CCR, (NODE i32:$src, i32:$opd))]> {
 596:   let Inst = (ascend
 597:     (descend CMD,
 598:       /*MODE*/0b000, /*REGISTER*/(operand "$opd", 3)),
 599:     (descend 0b0,
 600:       // REGISTER
 601:       (operand "$dst", 3),
 602:       !if(SIGNED, 0b1, 0b0),
 603:       /*SIZE*/0b0, 0b0000000,
 604:       // Dr REGISTER
 605:       0b000)
 606:   );
 607: }
 608: 
 609: // $reg <- $reg op $imm
 610: class MxDiMuOp_DI<string MN, bits<4> CMD, bit SIGNED = false,
 611:                   MxOperand DST, MxOperand OPD>
 612:     : MxInst<(outs DST:$dst), (ins DST:$src, OPD:$opd), MN#"\t$opd, $dst", []> {
 613:   // FIXME: Support immediates with different widths.
 614:   defvar ImmEnc = MxEncAddrMode_i<"opd", 16>;
 615:   let Inst = (ascend
 616:     (descend CMD,
```
- **EN**: This block declares or refines TableGen records such as `MxDiMuOp_DD_Long`, `MxDiMuOp_DI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxDiMuOp_DD_Long`, `MxDiMuOp_DI` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 617-644 / 第 617-644 行
```tablegen
 617:       // REGISTER
 618:       (operand "$dst", 3),
 619:       !if(SIGNED, 0b111, 0b011), ImmEnc.EA),
 620:     ImmEnc.Supplement
 621:   );
 622: }
 623: } // let Constraints
 624: } // Defs = [CCR]
 625: 
 626: multiclass MxDiMuOp<string MN, bits<4> CMD> {
 627:   def "S"#NAME#"d32d16" : MxDiMuOp_DD<MN#"s", CMD, /*SIGNED*/true, MxDRD32, MxDRD16>;
 628:   def "U"#NAME#"d32d16" : MxDiMuOp_DD<MN#"u", CMD, /*SIGNED*/false, MxDRD32, MxDRD16>;
 629: 
 630:   def "S"#NAME#"d32i16" : MxDiMuOp_DI<MN#"s", CMD, /*SIGNED*/true, MxDRD32, Mxi16imm>;
 631:   def "U"#NAME#"d32i16" : MxDiMuOp_DI<MN#"u", CMD, /*SIGNED*/false, MxDRD32, Mxi16imm>;
 632: }
 633: 
 634: defm DIV : MxDiMuOp<"div", 0x8>;
 635: 
 636: def SDIVd32d32 : MxDiMuOp_DD_Long<"divs.l", sdiv, 0x131, /*SIGNED*/true>;
 637: def UDIVd32d32 : MxDiMuOp_DD_Long<"divu.l", udiv, 0x131, /*SIGNED*/false>;
 638: 
 639: // This is used to cast immediates to 16-bits for operations which don't
 640: // support smaller immediate sizes.
 641: def as_i16imm : SDNodeXForm<imm, [{
 642:   return CurDAG->getTargetConstant(N->getSExtValue(), SDLoc(N), MVT::i16);
 643: }]>;
 644: 
```
- **EN**: This block declares or refines TableGen records such as `MxDiMuOp`, `DIV`, `SDIVd32d32`, `UDIVd32d32`, `as_i16imm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxDiMuOp`, `DIV`, `SDIVd32d32`, `UDIVd32d32`, `as_i16imm` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 645-672 / 第 645-672 行
```tablegen
 645: // RR i8
 646: def : Pat<(sdiv i8:$dst, i8:$opd),
 647:           (EXTRACT_SUBREG
 648:             (SDIVd32d16 (MOVSXd32d8 $dst), (MOVSXd16d8 $opd)),
 649:              MxSubRegIndex8Lo)>;
 650: 
 651: def : Pat<(udiv i8:$dst, i8:$opd),
 652:           (EXTRACT_SUBREG
 653:             (UDIVd32d16 (MOVZXd32d8 $dst), (MOVZXd16d8 $opd)),
 654:              MxSubRegIndex8Lo)>;
 655: 
 656: def : Pat<(srem i8:$dst, i8:$opd),
 657:           (EXTRACT_SUBREG
 658:             (ASR32di (ASR32di (SDIVd32d16 (MOVSXd32d8 $dst), (MOVSXd16d8 $opd)), 8), 8),
 659:              MxSubRegIndex8Lo)>;
 660: 
 661: def : Pat<(urem i8:$dst, i8:$opd),
 662:           (EXTRACT_SUBREG
 663:             (LSR32di (LSR32di (UDIVd32d16 (MOVZXd32d8 $dst), (MOVZXd16d8 $opd)), 8), 8),
 664:              MxSubRegIndex8Lo)>;
 665: 
 666: // RR i16
 667: def : Pat<(sdiv i16:$dst, i16:$opd),
 668:           (EXTRACT_SUBREG
 669:             (SDIVd32d16 (MOVSXd32d16 $dst), $opd),
 670:              MxSubRegIndex16Lo)>;
 671: 
 672: def : Pat<(udiv i16:$dst, i16:$opd),
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 673-700 / 第 673-700 行
```tablegen
 673:           (EXTRACT_SUBREG
 674:             (UDIVd32d16 (MOVZXd32d16 $dst), $opd),
 675:              MxSubRegIndex16Lo)>;
 676: 
 677: def : Pat<(srem i16:$dst, i16:$opd),
 678:           (EXTRACT_SUBREG
 679:             (ASR32di (ASR32di (SDIVd32d16 (MOVSXd32d16 $dst), $opd), 8), 8),
 680:              MxSubRegIndex16Lo)>;
 681: 
 682: def : Pat<(urem i16:$dst, i16:$opd),
 683:           (EXTRACT_SUBREG
 684:             (LSR32di (LSR32di (UDIVd32d16 (MOVZXd32d16 $dst), $opd), 8), 8),
 685:              MxSubRegIndex16Lo)>;
 686: 
 687: // RI i8
 688: def : Pat<(sdiv i8:$dst, Mxi8immSExt8:$opd),
 689:           (EXTRACT_SUBREG
 690:             (SDIVd32i16 (MOVSXd32d8 $dst), (as_i16imm $opd)),
 691:              MxSubRegIndex8Lo)>;
 692: 
 693: def : Pat<(udiv i8:$dst, Mxi8immSExt8:$opd),
 694:           (EXTRACT_SUBREG
 695:             (UDIVd32i16 (MOVZXd32d8 $dst), (as_i16imm $opd)),
 696:              MxSubRegIndex8Lo)>;
 697: 
 698: def : Pat<(srem i8:$dst, Mxi8immSExt8:$opd),
 699:           (EXTRACT_SUBREG
 700:             (ASR32di (ASR32di (SDIVd32i16 (MOVSXd32d8 $dst), (as_i16imm $opd)), 8), 8),
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 701-728 / 第 701-728 行
```tablegen
 701:              MxSubRegIndex8Lo)>;
 702: 
 703: def : Pat<(urem i8:$dst, Mxi8immSExt8:$opd),
 704:           (EXTRACT_SUBREG
 705:             (LSR32di (LSR32di (UDIVd32i16 (MOVZXd32d8 $dst), (as_i16imm $opd)), 8), 8),
 706:              MxSubRegIndex8Lo)>;
 707: 
 708: // RI i16
 709: def : Pat<(sdiv i16:$dst, Mxi16immSExt16:$opd),
 710:           (EXTRACT_SUBREG
 711:             (SDIVd32i16 (MOVSXd32d16 $dst), imm:$opd),
 712:              MxSubRegIndex16Lo)>;
 713: 
 714: def : Pat<(udiv i16:$dst, Mxi16immSExt16:$opd),
 715:           (EXTRACT_SUBREG
 716:             (UDIVd32i16 (MOVZXd32d16 $dst), imm:$opd),
 717:              MxSubRegIndex16Lo)>;
 718: 
 719: def : Pat<(srem i16:$dst, Mxi16immSExt16:$opd),
 720:           (EXTRACT_SUBREG
 721:             (ASR32di (ASR32di (SDIVd32i16 (MOVSXd32d16 $dst), imm:$opd), 8), 8),
 722:              MxSubRegIndex16Lo)>;
 723: 
 724: def : Pat<(urem i16:$dst, Mxi16immSExt16:$opd),
 725:           (EXTRACT_SUBREG
 726:             (LSR32di (LSR32di (UDIVd32i16 (MOVZXd32d16 $dst), imm:$opd), 8), 8),
 727:              MxSubRegIndex16Lo)>;
 728: 
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 729-756 / 第 729-756 行
```tablegen
 729: 
 730: defm MUL : MxDiMuOp<"mul", 0xC>;
 731: 
 732: def SMULd32d32 : MxDiMuOp_DD_Long<"muls.l", MxSMul, 0x130, /*SIGNED*/true>;
 733: def UMULd32d32 : MxDiMuOp_DD_Long<"mulu.l", MxUMul, 0x130, /*SIGNED*/false>;
 734: 
 735: // RR
 736: def : Pat<(mul i16:$dst, i16:$opd),
 737:           (EXTRACT_SUBREG
 738:             (SMULd32d16 (MOVXd32d16 $dst), $opd),
 739:              MxSubRegIndex16Lo)>;
 740: 
 741: def : Pat<(mulhs i16:$dst, i16:$opd),
 742:           (EXTRACT_SUBREG
 743:             (ASR32di (ASR32di (SMULd32d16 (MOVXd32d16 $dst), $opd), 8), 8),
 744:              MxSubRegIndex16Lo)>;
 745: 
 746: def : Pat<(mulhu i16:$dst, i16:$opd),
 747:           (EXTRACT_SUBREG
 748:             (LSR32di (LSR32di (UMULd32d16 (MOVXd32d16 $dst), $opd), 8), 8),
 749:              MxSubRegIndex16Lo)>;
 750: 
 751: def : Pat<(mul i32:$dst, i32:$opd), (SMULd32d32 $dst, $opd)>;
 752: 
 753: 
 754: // RI
 755: def : Pat<(mul i16:$dst, Mxi16immSExt16:$opd),
 756:           (EXTRACT_SUBREG
```
- **EN**: This block declares or refines TableGen records such as `MUL`, `SMULd32d32`, `UMULd32d32`.
- **CN**: 该代码块声明或细化了 `MUL`, `SMULd32d32`, `UMULd32d32` 等 TableGen 记录。

### Lines 757-784 / 第 757-784 行
```tablegen
 757:             (SMULd32i16 (MOVXd32d16 $dst), imm:$opd),
 758:              MxSubRegIndex16Lo)>;
 759: 
 760: def : Pat<(mulhs i16:$dst, Mxi16immSExt16:$opd),
 761:           (EXTRACT_SUBREG
 762:             (ASR32di (ASR32di (SMULd32i16 (MOVXd32d16 $dst), imm:$opd), 8), 8),
 763:              MxSubRegIndex16Lo)>;
 764: 
 765: def : Pat<(mulhu i16:$dst, Mxi16immSExt16:$opd),
 766:           (EXTRACT_SUBREG
 767:             (LSR32di (LSR32di (UMULd32i16 (MOVXd32d16 $dst), imm:$opd), 8), 8),
 768:              MxSubRegIndex16Lo)>;
 769: 
 770: 
 771: //===----------------------------------------------------------------------===//
 772: // NEG/NEGX/NOT
 773: //===----------------------------------------------------------------------===//
 774: 
 775: /// ------------+------------+------+---------+---------
 776: ///  F  E  D  C | B  A  9  8 | 7  6 | 5  4  3 | 2  1  0
 777: /// ------------+------------+------+-------------------
 778: ///             |            |      | EFFECTIVE ADDRESS
 779: ///  0  1  0  0 | x  x  x  x | SIZE |   MODE  |   REG
 780: /// ------------+------------+------+---------+---------
 781: let Defs = [CCR] in {
 782: let Constraints = "$src = $dst" in {
 783: 
 784: class MxNeg_D<MxType TYPE>
```
- **EN**: This block declares or refines TableGen records such as `MxNeg_D`.
- **CN**: 该代码块声明或细化了 `MxNeg_D` 等 TableGen 记录。

### Lines 785-812 / 第 785-812 行
```tablegen
 785:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src),
 786:              "neg."#TYPE.Prefix#"\t$dst",
 787:              [(set TYPE.VT:$dst, (ineg TYPE.VT:$src))]> {
 788:   let Inst = (descend 0b01000100,
 789:     /*SIZE*/!cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
 790:     //MODE without last bit
 791:     0b00,
 792:     //REGISTER prefixed by D/A bit
 793:     (operand "$dst", 4)
 794:   );
 795: }
 796: 
 797: let Uses = [CCR] in {
 798: class MxNegX_D<MxType TYPE>
 799:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src),
 800:              "negx."#TYPE.Prefix#"\t$dst",
 801:              [(set TYPE.VT:$dst, (MxSubX 0, TYPE.VT:$src, CCR))]> {
 802:   let Inst = (descend 0b01000000,
 803:     /*SIZE*/!cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
 804:     //MODE without last bit
 805:     0b00,
 806:     //REGISTER prefixed by D/A bit
 807:     (operand "$dst", 4)
 808:   );
 809: }
 810: }
 811: 
 812: class MxNot_D<MxType TYPE>
```
- **EN**: This block declares or refines TableGen records such as `MxNegX_D`, `MxNot_D`.
- **CN**: 该代码块声明或细化了 `MxNegX_D`, `MxNot_D` 等 TableGen 记录。

### Lines 813-840 / 第 813-840 行
```tablegen
 813:     : MxInst<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$src),
 814:              "not."#TYPE.Prefix#"\t$dst",
 815:              [(set TYPE.VT:$dst, (not TYPE.VT:$src))]> {
 816:   let Inst = (descend 0b01000110,
 817:     /*SIZE*/!cast<MxEncSize>("MxEncSize"#TYPE.Size).Value,
 818:     //MODE without last bit
 819:     0b00,
 820:     //REGISTER prefixed by D/A bit
 821:     (operand "$dst", 4)
 822:   );
 823: }
 824: 
 825: } // let Constraints
 826: } // let Defs = [CCR]
 827: 
 828: foreach S = [8, 16, 32] in {
 829:   def NEG#S#d  : MxNeg_D<!cast<MxType>("MxType"#S#"d")>;
 830:   def NEGX#S#d : MxNegX_D<!cast<MxType>("MxType"#S#"d")>;
 831:   def NOT#S#d  : MxNot_D<!cast<MxType>("MxType"#S#"d")>;
 832: }
 833: 
 834: def : Pat<(MxSub 0, i8 :$src), (NEG8d  MxDRD8 :$src)>;
 835: def : Pat<(MxSub 0, i16:$src), (NEG16d MxDRD16:$src)>;
 836: def : Pat<(MxSub 0, i32:$src), (NEG32d MxDRD32:$src)>;
 837: // SExt of i1 values.
 838: // Although we specify `ZeroOrNegativeOneBooleanContent` for boolean content,
 839: // we're still adding an AND here as we don't know the origin of the i1 value.
 840: def : Pat<(sext_inreg i8:$src,  i1), (NEG8d  (AND8di  MxDRD8:$src,  1))>;
```
- **EN**: This block declares or refines TableGen records such as `NEG`, `NEGX`, `NOT`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `NEG`, `NEGX`, `NOT` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 841-868 / 第 841-868 行
```tablegen
 841: def : Pat<(sext_inreg i16:$src, i1), (NEG16d (AND16di MxDRD16:$src, 1))>;
 842: def : Pat<(sext_inreg i32:$src, i1), (NEG32d (AND32di MxDRD32:$src, 1))>;
 843: 
 844: //===----------------------------------------------------------------------===//
 845: // no-CCR Patterns
 846: //===----------------------------------------------------------------------===//
 847: 
 848: /// Basically the reason for this stuff is that add and addc share the same
 849: /// operand types constraints for whatever reasons and I had to define a common
 850: /// MxAdd and MxSub instructions that produce CCR and then pattern-map add and addc
 851: /// to it.
 852: /// NOTE On the other hand I see no reason why I cannot just drop explicit CCR
 853: /// result. Anyway works for now, hopefully I will better understand how this stuff
 854: /// is designed later
 855: foreach N = ["add", "addc"] in {
 856: 
 857:   // add reg, reg
 858:   def : Pat<(!cast<SDNode>(N) i8 :$src, i8 :$opd),
 859:             (ADD8dd  MxDRD8 :$src, MxDRD8 :$opd)>;
 860:   def : Pat<(!cast<SDNode>(N) i16:$src, i16:$opd),
 861:             (ADD16dr MxXRD16:$src, MxDRD16:$opd)>;
 862:   def : Pat<(!cast<SDNode>(N) i32:$src, i32:$opd),
 863:             (ADD32dr MxXRD32:$src, MxDRD32:$opd)>;
 864: 
 865:   // add (An), reg
 866:   def : Pat<(!cast<SDNode>(N) MxType8.VT:$src, (Mxloadi8 MxType8.JPat:$opd)),
 867:             (ADD8dj MxDRD8:$src, MxType8.JOp:$opd)>;
 868:   def : Pat<(!cast<SDNode>(N) MxType16.VT:$src, (Mxloadi16 MxType16.JPat:$opd)),
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 869-896 / 第 869-896 行
```tablegen
 869:             (ADD16dj MxDRD16:$src, MxType16.JOp:$opd)>;
 870:   def : Pat<(!cast<SDNode>(N) MxType32.VT:$src, (Mxloadi32 MxType32.JPat:$opd)),
 871:             (ADD32dj MxDRD32:$src, MxType32.JOp:$opd)>;
 872: 
 873:   // add (i,An), reg
 874:   def : Pat<(!cast<SDNode>(N) MxType8.VT:$src, (Mxloadi8 MxType8.PPat:$opd)),
 875:             (ADD8dp MxDRD8:$src, MxType8.POp:$opd)>;
 876:   def : Pat<(!cast<SDNode>(N) MxType16.VT:$src, (Mxloadi16 MxType16.PPat:$opd)),
 877:             (ADD16dp MxDRD16:$src, MxType16.POp:$opd)>;
 878:   def : Pat<(!cast<SDNode>(N) MxType32.VT:$src, (Mxloadi32 MxType32.PPat:$opd)),
 879:             (ADD32dp MxDRD32:$src, MxType32.POp:$opd)>;
 880: 
 881:   // add (i,An,Xn), reg
 882:   def : Pat<(!cast<SDNode>(N) MxType8.VT:$src, (Mxloadi8 MxType8.FPat:$opd)),
 883:             (ADD8df MxDRD8:$src, MxType8.FOp:$opd)>;
 884:   def : Pat<(!cast<SDNode>(N) MxType16.VT:$src, (Mxloadi16 MxType16.FPat:$opd)),
 885:             (ADD16df MxDRD16:$src, MxType16.FOp:$opd)>;
 886:   def : Pat<(!cast<SDNode>(N) MxType32.VT:$src, (Mxloadi32 MxType32.FPat:$opd)),
 887:             (ADD32df MxDRD32:$src, MxType32.FOp:$opd)>;
 888: 
 889:   // add reg, imm
 890:   def : Pat<(!cast<SDNode>(N) i8: $src, Mxi8immSExt8:$opd),
 891:             (ADD8di  MxDRD8 :$src, imm:$opd)>;
 892:   def : Pat<(!cast<SDNode>(N) i16:$src, Mxi16immSExt16:$opd),
 893:             (ADD16di MxDRD16:$src, imm:$opd)>;
 894: 
 895:   // LEAp is more complex and thus will be selected over normal ADD32ri but it cannot
 896:   // be used with data registers, here by adding complexity to a simple ADD32ri insts
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 897-924 / 第 897-924 行
```tablegen
 897:   // we make sure it will be selected over LEAp
 898:   let AddedComplexity = 15 in {
 899:   def : Pat<(!cast<SDNode>(N) i32:$src, Mxi32immSExt32:$opd),
 900:             (ADD32di MxDRD32:$src, imm:$opd)>;
 901:   } // AddedComplexity = 15
 902: 
 903:   // add imm, (An)
 904:   def : Pat<(store (!cast<SDNode>(N) (load MxType8.JPat:$dst), MxType8.IPat:$opd),
 905:                    MxType8.JPat:$dst),
 906:             (ADD8ji MxType8.JOp:$dst, imm:$opd)>;
 907:   def : Pat<(store (!cast<SDNode>(N) (load MxType16.JPat:$dst), MxType16.IPat:$opd),
 908:                    MxType16.JPat:$dst),
 909:             (ADD16ji MxType16.JOp:$dst, imm:$opd)>;
 910:   def : Pat<(store (!cast<SDNode>(N) (load MxType32.JPat:$dst), MxType32.IPat:$opd),
 911:                    MxType32.JPat:$dst),
 912:             (ADD32ji MxType32.JOp:$dst, imm:$opd)>;
 913: 
 914: } // foreach add, addc
 915: 
 916: def : Pat<(adde i8 :$src, i8 :$opd), (ADDX8dd  MxDRD8 :$src, MxDRD8 :$opd)>;
 917: def : Pat<(adde i16:$src, i16:$opd), (ADDX16dd MxDRD16:$src, MxDRD16:$opd)>;
 918: def : Pat<(adde i32:$src, i32:$opd), (ADDX32dd MxDRD32:$src, MxDRD32:$opd)>;
 919: 
 920: 
 921: 
 922: foreach N = ["sub", "subc"] in {
 923: 
 924:   // sub reg, reg
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 925-952 / 第 925-952 行
```tablegen
 925:   def : Pat<(!cast<SDNode>(N) i8 :$src, i8 :$opd),
 926:             (SUB8dd  MxDRD8 :$src, MxDRD8 :$opd)>;
 927:   def : Pat<(!cast<SDNode>(N) i16:$src, i16:$opd),
 928:             (SUB16dd MxDRD16:$src, MxDRD16:$opd)>;
 929:   def : Pat<(!cast<SDNode>(N) i32:$src, i32:$opd),
 930:             (SUB32dd MxDRD32:$src, MxDRD32:$opd)>;
 931: 
 932: 
 933:   // sub (An), reg
 934:   def : Pat<(!cast<SDNode>(N) MxType8.VT:$src, (Mxloadi8 MxType8.JPat:$opd)),
 935:             (SUB8dj MxDRD8:$src, MxType8.JOp:$opd)>;
 936:   def : Pat<(!cast<SDNode>(N) MxType16.VT:$src, (Mxloadi16 MxType16.JPat:$opd)),
 937:             (SUB16dj MxDRD16:$src, MxType16.JOp:$opd)>;
 938:   def : Pat<(!cast<SDNode>(N) MxType32.VT:$src, (Mxloadi32 MxType32.JPat:$opd)),
 939:             (SUB32dj MxDRD32:$src, MxType32.JOp:$opd)>;
 940: 
 941:   // sub (i,An), reg
 942:   def : Pat<(!cast<SDNode>(N) MxType8.VT:$src, (Mxloadi8 MxType8.PPat:$opd)),
 943:             (SUB8dp MxDRD8:$src, MxType8.POp:$opd)>;
 944:   def : Pat<(!cast<SDNode>(N) MxType16.VT:$src, (Mxloadi16 MxType16.PPat:$opd)),
 945:             (SUB16dp MxDRD16:$src, MxType16.POp:$opd)>;
 946:   def : Pat<(!cast<SDNode>(N) MxType32.VT:$src, (Mxloadi32 MxType32.PPat:$opd)),
 947:             (SUB32dp MxDRD32:$src, MxType32.POp:$opd)>;
 948: 
 949:   // sub (i,An,Xn), reg
 950:   def : Pat<(!cast<SDNode>(N) MxType8.VT:$src, (Mxloadi8 MxType8.FPat:$opd)),
 951:             (SUB8df MxDRD8:$src, MxType8.FOp:$opd)>;
 952:   def : Pat<(!cast<SDNode>(N) MxType16.VT:$src, (Mxloadi16 MxType16.FPat:$opd)),
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 953-980 / 第 953-980 行
```tablegen
 953:             (SUB16df MxDRD16:$src, MxType16.FOp:$opd)>;
 954:   def : Pat<(!cast<SDNode>(N) MxType32.VT:$src, (Mxloadi32 MxType32.FPat:$opd)),
 955:             (SUB32df MxDRD32:$src, MxType32.FOp:$opd)>;
 956: 
 957:   // sub reg, imm
 958:   def : Pat<(!cast<SDNode>(N) i8 :$src, Mxi8immSExt8 :$opd),
 959:             (SUB8di  MxDRD8 :$src, imm:$opd)>;
 960:   def : Pat<(!cast<SDNode>(N) i16:$src, Mxi16immSExt16:$opd),
 961:             (SUB16di MxDRD16:$src, imm:$opd)>;
 962:   def : Pat<(!cast<SDNode>(N) i32:$src, Mxi32immSExt32:$opd),
 963:             (SUB32di MxDRD32:$src, imm:$opd)>;
 964: 
 965:   // sub imm, (An)
 966:   def : Pat<(store (!cast<SDNode>(N) (load MxType8.JPat:$dst), MxType8.IPat:$opd),
 967:                    MxType8.JPat:$dst),
 968:             (SUB8ji MxType8.JOp:$dst, imm:$opd)>;
 969:   def : Pat<(store (!cast<SDNode>(N) (load MxType16.JPat:$dst), MxType16.IPat:$opd),
 970:                    MxType16.JPat:$dst),
 971:             (SUB16ji MxType16.JOp:$dst, imm:$opd)>;
 972:   def : Pat<(store (!cast<SDNode>(N) (load MxType32.JPat:$dst), MxType32.IPat:$opd),
 973:                    MxType32.JPat:$dst),
 974:             (SUB32ji MxType32.JOp:$dst, imm:$opd)>;
 975: 
 976: } // foreach sub, subx
 977: 
 978: def : Pat<(sube i8 :$src, i8 :$opd), (SUBX8dd  MxDRD8 :$src, MxDRD8 :$opd)>;
 979: def : Pat<(sube i16:$src, i16:$opd), (SUBX16dd MxDRD16:$src, MxDRD16:$opd)>;
 980: def : Pat<(sube i32:$src, i32:$opd), (SUBX32dd MxDRD32:$src, MxDRD32:$opd)>;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 981-1008 / 第 981-1008 行
```tablegen
 981: 
 982: multiclass BitwisePat<string INST, SDNode OP> {
 983:   // op reg, reg
 984:   def : Pat<(OP i8 :$src, i8 :$opd),
 985:             (!cast<MxInst>(INST#"8dd")  MxDRD8 :$src, MxDRD8 :$opd)>;
 986:   def : Pat<(OP i16:$src, i16:$opd),
 987:             (!cast<MxInst>(INST#"16dd") MxDRD16:$src, MxDRD16:$opd)>;
 988:   def : Pat<(OP i32:$src, i32:$opd),
 989:             (!cast<MxInst>(INST#"32dd") MxDRD32:$src, MxDRD32:$opd)>;
 990:   // op reg, imm
 991:   def : Pat<(OP i8: $src, Mxi8immSExt8 :$opd),
 992:             (!cast<MxInst>(INST#"8di")  MxDRD8 :$src, imm:$opd)>;
 993:   def : Pat<(OP i16:$src, Mxi16immSExt16:$opd),
 994:             (!cast<MxInst>(INST#"16di") MxDRD16:$src, imm:$opd)>;
 995:   def : Pat<(OP i32:$src, Mxi32immSExt32:$opd),
 996:             (!cast<MxInst>(INST#"32di") MxDRD32:$src, imm:$opd)>;
 997: }
 998: 
 999: defm : BitwisePat<"AND", and>;
1000: defm : BitwisePat<"OR",  or>;
1001: defm : BitwisePat<"XOR", xor>;
1002: 
1003: //===----------------------------------------------------------------------===//
1004: // Floating point arithmetic instruction
1005: //===----------------------------------------------------------------------===//
1006: 
1007: let Defs = [FPS] in
1008: class MxFArithBase_FF<dag outs, dag ins, string asm, string rounding,
```
- **EN**: This block declares or refines TableGen records such as `BitwisePat`, `MxFArithBase_FF`.
- **CN**: 该代码块声明或细化了 `BitwisePat`, `MxFArithBase_FF` 等 TableGen 记录。

### Lines 1009-1036 / 第 1009-1036 行
```tablegen
1009:                       list<dag> patterns>
1010:     : MxInst<outs, ins, asm, patterns> {
1011:   let Uses = !if(!eq(rounding, ""), [FPC], []);
1012: 
1013:   let Predicates = !if(!eq(rounding, ""), [AtLeastM68881], [AtLeastM68040]);
1014: }
1015: 
1016: class MxFPOpModeSelector<string rounding, bits<7> single, bits<7> double,
1017:                          bits<7> extended> {
1018:   bits<7> Mode = !cond(!eq(rounding, "s"): single,
1019:                        !eq(rounding, "d"): double,
1020:                        !eq(rounding, ""): extended);
1021: }
1022: 
1023: //===----------------------------------------------------------------------===//
1024: // Unary floating point instruction
1025: //===----------------------------------------------------------------------===//
1026: 
1027: class MxFUnary_FF<MxOpBundle Opnd, string rounding,
1028:                   string mnemonic, bits<7> opmode>
1029:     : MxFArithBase_FF<(outs Opnd.Op:$dst), (ins Opnd.Op:$src),
1030:                     "f"#rounding#mnemonic#".x\t$src, $dst", rounding, [(null_frag)]> {
1031:   let Inst = (ascend
1032:   (descend 0b1111,
1033:     /*COPROCESSOR ID*/0b001,
1034:     0b000,
1035:     /*MODE+REGISTER*/0b000000),
1036:   (descend 0b0, /* R/M */ 0b0, 0b0,
```
- **EN**: This block declares or refines TableGen records such as `MxFPOpModeSelector`, `MxFUnary_FF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxFPOpModeSelector`, `MxFUnary_FF` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1037-1064 / 第 1037-1064 行
```tablegen
1037:     /*SOURCE SPECIFIER*/
1038:     (operand "$src", 3),
1039:     /*DESTINATION*/
1040:     (operand "$dst", 3),
1041:     /*OPMODE*/
1042:     opmode)
1043:   );
1044: }
1045: 
1046: multiclass MxFUnaryOp<string mnemonic, bits<7> single, bits<7> double,
1047:                       bits<7> extended> {
1048:   foreach rounding = ["", "s", "d"] in {
1049:     defvar opmode = MxFPOpModeSelector<rounding, single, double, extended>.Mode;
1050: 
1051:     def F # !toupper(rounding) # !substr(NAME, 1) # "80fp_fp"
1052:       : MxFUnary_FF<MxOp80AddrMode_fpr, rounding, mnemonic, opmode>;
1053:   
1054:     let isCodeGenOnly = 1 in
1055:     foreach size = [32, 64] in
1056:       def F # !toupper(rounding) # !substr(NAME, 1) # size # "fp_fp"
1057:         : MxFUnary_FF<!cast<MxOpBundle>("MxOp"#size#"AddrMode_fpr"),
1058:                       rounding, mnemonic, opmode>;
1059:   }
1060: }
1061: 
1062: defm FABS : MxFUnaryOp<"abs", 0b1011000, 0b1011100, 0b0011000>;
1063: defm FNEG : MxFUnaryOp<"neg", 0b1011010, 0b1011110, 0b0011010>;
1064: 
```
- **EN**: This block declares or refines TableGen records such as `MxFUnaryOp`, `F`, `F`, `FABS`, `FNEG`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxFUnaryOp`, `F`, `F`, `FABS`, `FNEG` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1065-1092 / 第 1065-1092 行
```tablegen
1065: //===----------------------------------------------------------------------===//
1066: // Binary floating point instruction
1067: //===----------------------------------------------------------------------===//
1068: 
1069: let Constraints = "$src = $dst" in
1070: class MxFBinary_FF<MxOpBundle Opnd, string rounding,
1071:                    string mnemonic, bits<7> opmode>
1072:     : MxFArithBase_FF<(outs Opnd.Op:$dst), (ins Opnd.Op:$src, Opnd.Op:$opd),
1073:                       "f"#rounding#mnemonic#".x\t$opd, $dst", rounding, [(null_frag)]> {
1074:   let Inst = (ascend
1075:   (descend 0b1111,
1076:     /*COPROCESSOR ID*/0b001,
1077:     0b000,
1078:     /*MODE+REGISTER*/0b000000),
1079:   (descend 0b0, /* R/M */ 0b0, 0b0,
1080:     /*SOURCE SPECIFIER*/
1081:     (operand "$opd", 3),
1082:     /*DESTINATION*/
1083:     (operand "$dst", 3),
1084:     /*OPMODE*/
1085:     opmode)
1086:   );
1087: }
1088: 
1089: multiclass MxFBinaryOp<string mnemonic, bits<7> single, bits<7> double,
1090:                       bits<7> extended> {
1091:   foreach rounding = ["", "s", "d"] in {
1092:     defvar opmode = MxFPOpModeSelector<rounding, single, double, extended>.Mode;
```
- **EN**: This block declares or refines TableGen records such as `MxFBinary_FF`, `MxFBinaryOp`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxFBinary_FF`, `MxFBinaryOp` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1093-1108 / 第 1093-1108 行
```tablegen
1093: 
1094:     def F # !toupper(rounding) # !substr(NAME, 1) # "80fp_fp"
1095:       : MxFBinary_FF<MxOp80AddrMode_fpr, rounding, mnemonic, opmode>;
1096: 
1097:     let isCodeGenOnly = 1 in
1098:     foreach size = [32, 64] in
1099:       def F # !toupper(rounding) # !substr(NAME, 1) # size # "fp_fp"
1100:         : MxFBinary_FF<!cast<MxOpBundle>("MxOp"#size#"AddrMode_fpr"),
1101:                         rounding, mnemonic, opmode>;
1102:   }
1103: }
1104: 
1105: defm FADD : MxFBinaryOp<"add", 0b1100010, 0b1100110, 0b0100010>;
1106: defm FSUB : MxFBinaryOp<"sub", 0b1101000, 0b1101100, 0b0101000>;
1107: defm FMUL : MxFBinaryOp<"mul", 0b1100011, 0b1100111, 0b0100011>;
1108: defm FDIV : MxFBinaryOp<"div", 0b1100000, 0b1100100, 0b0100000>;
```
- **EN**: This block declares or refines TableGen records such as `F`, `F`, `FADD`, `FSUB`, `FMUL`, `FDIV`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `F`, `F`, `FADD`, `FSUB`, `FMUL`, `FDIV` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
