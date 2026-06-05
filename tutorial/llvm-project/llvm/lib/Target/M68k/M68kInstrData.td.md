# M68kInstrData.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrData.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 M68k 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //===-- M68kInstrData.td - M68k Data Movement Instructions -*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file describes the Motorola 680x0 data movement instructions which are
  11: /// the basic means of transferring and storing addresses and data. Here is the
  12: /// current status of the file:
  13: ///
  14: ///  Machine:
  15: ///
  16: ///     EXG   [ ]     FMOVE [ ]     FSMOVE [ ]     FDMOVE [ ]     FMOVEM [ ]
  17: ///     LEA   [~]     PEA   [ ]     MOVE   [~]     MOVE16 [ ]     MOVEA  [ ]
  18: ///     MOVEM [ ]     MOVEP [ ]     MOVEQ  [ ]     LINK   [~]     UNLK   [~]
  19: ///
  20: ///  Pseudo:
  21: ///
  22: ///     MOVI  [x]     MOVSX [x]     MOVZX [x]     MOVX   [x]
  23: ///
  24: ///  Map:
  25: ///
  26: ///   [ ] - was not touched at all
  27: ///   [!] - requires extarnal stuff implemented
  28: ///   [~] - in progress but usable
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 29-56 / 第 29-56 行
```tablegen
  29: ///   [x] - done
  30: ///
  31: //===----------------------------------------------------------------------===//
  32: 
  33: //===----------------------------------------------------------------------===//
  34: // MOVE
  35: //===----------------------------------------------------------------------===//
  36: 
  37: /// -----------------------------------------------------
  38: ///  F  E | D  C | B  A  9 | 8  7  6 | 5  4  3 | 2  1  0
  39: /// -----------------------------------------------------
  40: ///       |      |    DESTINATION    |       SOURCE
  41: ///  0  0 | SIZE |   REG   |   MODE  |   MODE  |   REG
  42: /// -----------------------------------------------------
  43: ///
  44: /// NOTE Move requires EA X version for direct register destination(0)
  45: 
  46: // MOVE has a different size encoding.
  47: class MxMoveSize<bits<2> value> {
  48:   bits<2> Value = value;
  49: }
  50: def MxMoveSize8  : MxMoveSize<0b01>;
  51: def MxMoveSize16 : MxMoveSize<0b11>;
  52: def MxMoveSize32 : MxMoveSize<0b10>;
  53: 
  54: class MxMoveEncoding<MxMoveSize size, MxEncMemOp dst_enc, MxEncMemOp src_enc> {
  55:   dag Value = (ascend
  56:     (descend 0b00, size.Value,
```
- **EN**: This block declares or refines TableGen records such as `MxMoveSize`, `MxMoveSize8`, `MxMoveSize16`, `MxMoveSize32`, `MxMoveEncoding`.
- **CN**: 该代码块声明或细化了 `MxMoveSize`, `MxMoveSize8`, `MxMoveSize16`, `MxMoveSize32`, `MxMoveEncoding` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57:              !cond(
  58:                !eq(!getdagop(dst_enc.EA), descend): !setdagop(dst_enc.EA, ascend),
  59:                !eq(!getdagop(dst_enc.EA), ascend): !setdagop(dst_enc.EA, descend)),
  60:              src_enc.EA),
  61:     // Source extension
  62:     src_enc.Supplement,
  63:     // Destination extension
  64:     dst_enc.Supplement
  65:   );
  66: }
  67: 
  68: // Special encoding for Xn
  69: class MxMoveEncAddrMode_r<string reg_opnd> : MxEncMemOp {
  70:   let EA = (descend (descend 0b00, (slice "$"#reg_opnd, 3, 3)),
  71:                     (operand "$"#reg_opnd, 3));
  72: }
  73: 
  74: // TODO: Generalize and adopt this utility in other .td files as well.
  75: multiclass MxMoveOperandEncodings<string opnd_name> {
  76:   // Dn
  77:   def MxMove#NAME#OpEnc_d : MxEncAddrMode_d<opnd_name>;
  78:   // An
  79:   def MxMove#NAME#OpEnc_a : MxEncAddrMode_a<opnd_name>;
  80:   // Xn
  81:   def MxMove#NAME#OpEnc_r : MxMoveEncAddrMode_r<opnd_name>;
  82:   // (An)+
  83:   def MxMove#NAME#OpEnc_o : MxEncAddrMode_o<opnd_name>;
  84:   // -(An)
```
- **EN**: This block declares or refines TableGen records such as `MxMoveEncAddrMode_r`, `MxMoveOperandEncodings`, `MxMove`, `MxMove`, `MxMove`, `MxMove`.
- **CN**: 该代码块声明或细化了 `MxMoveEncAddrMode_r`, `MxMoveOperandEncodings`, `MxMove`, `MxMove`, `MxMove`, `MxMove` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85:   def MxMove#NAME#OpEnc_e : MxEncAddrMode_e<opnd_name>;
  86:   // (i,PC,Xn)
  87:   def MxMove#NAME#OpEnc_k : MxEncAddrMode_k<opnd_name>;
  88:   // (i,PC)
  89:   def MxMove#NAME#OpEnc_q : MxEncAddrMode_q<opnd_name>;
  90:   // (i,An,Xn)
  91:   def MxMove#NAME#OpEnc_f : MxEncAddrMode_f<opnd_name>;
  92:   // (i,An)
  93:   def MxMove#NAME#OpEnc_p : MxEncAddrMode_p<opnd_name>;
  94:   // (ABS).L
  95:   def MxMove#NAME#OpEnc_b : MxEncAddrMode_abs<opnd_name, /*W/L=*/true>;
  96:   // (An)
  97:   def MxMove#NAME#OpEnc_j : MxEncAddrMode_j<opnd_name>;
  98: }
  99: 
 100: defm Src : MxMoveOperandEncodings<"src">;
 101: defm Dst : MxMoveOperandEncodings<"dst">;
 102: 
 103: defvar MxMoveSupportedAMs = ["o", "e", "k", "q", "f", "p", "b", "j"];
 104: 
 105: let Defs = [CCR] in
 106: class MxMove<string size, dag outs, dag ins, list<dag> pattern, MxMoveEncoding enc>
 107:     : MxInst<outs, ins, "move."#size#"\t$src, $dst", pattern> {
 108:   let Inst = enc.Value;
 109: }
 110: 
 111: // R <- R
 112: class MxMove_RR<MxType TYPE, string DST_REG, string SRC_REG,
```
- **EN**: This block declares or refines TableGen records such as `MxMove`, `MxMove`, `MxMove`, `MxMove`, `MxMove`, `MxMove`.
- **CN**: 该代码块声明或细化了 `MxMove`, `MxMove`, `MxMove`, `MxMove`, `MxMove`, `MxMove` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113:                 MxMoveEncoding ENC,
 114:                 MxOpBundle DST = !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#DST_REG),
 115:                 MxOpBundle SRC = !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#SRC_REG)>
 116:     : MxMove<TYPE.Prefix,
 117:              (outs DST.Op:$dst), (ins SRC.Op:$src),
 118:              [(null_frag)], ENC>;
 119: 
 120: foreach DST_REG = ["r", "a"] in {
 121:   foreach SRC_REG = ["r", "a"] in
 122:   foreach TYPE = [MxType16, MxType32] in
 123:   def MOV # TYPE.Size # DST_REG # SRC_REG # TYPE.Postfix
 124:       : MxMove_RR<TYPE, DST_REG, SRC_REG,
 125:                   MxMoveEncoding<!cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 126:                                  !cast<MxEncMemOp>("MxMoveDstOpEnc_"#DST_REG),
 127:                                  !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#SRC_REG)>>;
 128: } // foreach DST_REG
 129: foreach TYPE = [MxType8, MxType16, MxType32] in
 130: def MOV # TYPE.Size # dd # TYPE.Postfix
 131:     : MxMove_RR<TYPE, "d", "d",
 132:                 MxMoveEncoding<!cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 133:                                MxMoveDstOpEnc_d, MxMoveSrcOpEnc_d>>;
 134: 
 135: // M <- R
 136: let mayStore = 1 in {
 137: class MxMove_MR<MxType TYPE, MxOpBundle DST, string SRC_REG, MxMoveEncoding ENC,
 138:                 MxOpBundle SRC = !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#SRC_REG)>
 139:     : MxMove<TYPE.Prefix, (outs), (ins DST.Op:$dst, SRC.Op:$src),
 140:              [(store TYPE.VT:$src, DST.Pat:$dst)], ENC>;
```
- **EN**: This block declares or refines TableGen records such as `MOV`, `MOV`, `MxMove_MR`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MOV`, `MOV`, `MxMove_MR` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: 
 142: class MxMove_MI<MxType TYPE, MxOpBundle DST, MxMoveEncoding ENC,
 143:                 MxImmOpBundle SRC = !cast<MxImmOpBundle>("MxOp"#TYPE.Size#"AddrMode_i")>
 144:     : MxMove<TYPE.Prefix, (outs), (ins DST.Op:$dst, SRC.Op:$src),
 145:              [(store SRC.ImmPat:$src, DST.Pat:$dst)], ENC>;
 146: } // let mayStore = 1
 147: 
 148: foreach REG = ["r", "a", "d"] in
 149: foreach AM = MxMoveSupportedAMs in {
 150:   foreach TYPE = !if(!eq(REG, "d"), [MxType8, MxType16, MxType32], [MxType16, MxType32]) in
 151:   def MOV # TYPE.Size # AM # REG # TYPE.Postfix
 152:       : MxMove_MR<TYPE, !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#AM), REG,
 153:                   MxMoveEncoding<!cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 154:                                  !cast<MxEncMemOp>("MxMoveDstOpEnc_"#AM),
 155:                                  !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#REG)>>;
 156: } // foreach AM
 157: 
 158: foreach AM = MxMoveSupportedAMs in {
 159:   foreach TYPE = [MxType8, MxType16, MxType32] in
 160:   def MOV # TYPE.Size # AM # i # TYPE.Postfix
 161:       : MxMove_MI<TYPE, !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#AM),
 162:                   MxMoveEncoding<!cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 163:                                  !cast<MxEncMemOp>("MxMoveDstOpEnc_"#AM),
 164:                                  MxEncAddrMode_i<"src", TYPE.Size>>>;
 165: } // foreach AM
 166: 
 167: // R <- I
 168: // No pattern, as all immediate -> register moves are matched to the MOVI pseudo
```
- **EN**: This block declares or refines TableGen records such as `MxMove_MI`, `MOV`, `MOV`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxMove_MI`, `MOV`, `MOV` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 169-196 / 第 169-196 行
```tablegen
 169: class MxMove_RI<MxType TYPE, string DST_REG, MxMoveEncoding ENC,
 170:                 MxImmOpBundle SRC = !cast<MxImmOpBundle>("MxOp"#TYPE.Size#"AddrMode_i"),
 171:                 MxOpBundle DST = !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#DST_REG)>
 172:     : MxMove<TYPE.Prefix, (outs DST.Op:$dst), (ins SRC.Op:$src),
 173:               [(null_frag)], ENC>;
 174: 
 175: foreach REG = ["r", "a", "d"] in {
 176:   foreach TYPE = !if(!eq(REG, "d"), [MxType8, MxType16, MxType32], [MxType16, MxType32]) in
 177:   def MOV # TYPE.Size # REG # i # TYPE.Postfix
 178:       : MxMove_RI<TYPE, REG,
 179:                   MxMoveEncoding<!cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 180:                                  !cast<MxEncMemOp>("MxMoveDstOpEnc_"#REG),
 181:                                  MxEncAddrMode_i<"src", TYPE.Size>>>;
 182: } // foreach REG
 183: 
 184: // R <- M
 185: let mayLoad = 1 in
 186: class MxMove_RM<MxType TYPE, string DST_REG, MxOpBundle SRC, MxEncMemOp SRC_ENC,
 187:                 MxMoveSize SIZE_ENC = !cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 188:                 MxOpBundle DST = !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#DST_REG),
 189:                 MxEncMemOp DST_ENC = !cast<MxEncMemOp>("MxMoveDstOpEnc_"#DST_REG)>
 190:     : MxMove<TYPE.Prefix, (outs DST.Op:$dst), (ins SRC.Op:$src),
 191:              [(set TYPE.VT:$dst, (TYPE.Load SRC.Pat:$src))],
 192:              MxMoveEncoding<SIZE_ENC, DST_ENC, SRC_ENC>>;
 193: 
 194: foreach REG = ["r", "a", "d"] in
 195: foreach AM = MxMoveSupportedAMs in {
 196:   foreach TYPE = !if(!eq(REG, "d"), [MxType8, MxType16, MxType32], [MxType16, MxType32]) in
```
- **EN**: This block declares or refines TableGen records such as `MxMove_RI`, `MOV`, `MxMove_RM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxMove_RI`, `MOV`, `MxMove_RM` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```tablegen
 197:   def MOV # TYPE.Size # REG # AM # TYPE.Postfix
 198:       : MxMove_RM<TYPE, REG, !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#AM),
 199:                   !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#AM)>;
 200: } // foreach AM
 201: 
 202: // Tail call version
 203: let Pattern = [(null_frag)] in {
 204:   foreach REG = ["r", "a"] in
 205:   foreach AM = MxMoveSupportedAMs in {
 206:     foreach TYPE = [MxType16, MxType32] in
 207:     def MOV # TYPE.Size # REG # AM # _TC
 208:         : MxMove_RM<TYPE, REG, !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#AM),
 209:                     !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#AM)> {
 210:       let isCodeGenOnly = true;
 211:     }
 212:   } // foreach AM
 213: } // let Pattern
 214: 
 215: let mayLoad = 1, mayStore = 1 in
 216: class MxMove_MM<MxType TYPE, MxOpBundle DST, MxOpBundle SRC,
 217:                 MxEncMemOp DST_ENC, MxEncMemOp SRC_ENC>
 218:     : MxMove<TYPE.Prefix, (outs), (ins DST.Op:$dst, SRC.Op:$src),
 219:              [(store (TYPE.Load SRC.Pat:$src), DST.Pat:$dst)],
 220:              MxMoveEncoding<!cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 221:                             DST_ENC, SRC_ENC>>;
 222: 
 223: // Use a store-load patfrag that can reject unsafe load chains (e.g. callseq).
 224: let mayLoad = 1, mayStore = 1 in
```
- **EN**: This block declares or refines TableGen records such as `MOV`, `MOV`, `MxMove_MM`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MOV`, `MOV`, `MxMove_MM` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 225-252 / 第 225-252 行
```tablegen
 225: class MxMove_MM_Safe<MxType TYPE, PatFrag StoreLoad,
 226:                      MxOpBundle DST, MxOpBundle SRC,
 227:                      MxEncMemOp DST_ENC, MxEncMemOp SRC_ENC>
 228:     : MxMove<TYPE.Prefix, (outs), (ins DST.Op:$dst, SRC.Op:$src),
 229:              [(StoreLoad DST.Pat:$dst, SRC.Pat:$src)],
 230:              MxMoveEncoding<!cast<MxMoveSize>("MxMoveSize"#TYPE.Size),
 231:                             DST_ENC, SRC_ENC>>;
 232: 
 233: foreach DST_AM = MxMoveSupportedAMs in
 234: foreach SRC_AM = MxMoveSupportedAMs in {
 235:   def MOV8 # DST_AM # SRC_AM # !cast<MxType>("MxType8").Postfix
 236:       : MxMove_MM_Safe<MxType8, Mxstoreloadi8_safe,
 237:                        !cast<MxOpBundle>("MxOp8AddrMode_"#DST_AM),
 238:                        !cast<MxOpBundle>("MxOp8AddrMode_"#SRC_AM),
 239:                        !cast<MxEncMemOp>("MxMoveDstOpEnc_"#DST_AM),
 240:                        !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#SRC_AM)>;
 241: 
 242:   def MOV16 # DST_AM # SRC_AM # !cast<MxType>("MxType16").Postfix
 243:       : MxMove_MM_Safe<MxType16, Mxstoreloadi16_safe,
 244:                        !cast<MxOpBundle>("MxOp16AddrMode_"#DST_AM),
 245:                        !cast<MxOpBundle>("MxOp16AddrMode_"#SRC_AM),
 246:                        !cast<MxEncMemOp>("MxMoveDstOpEnc_"#DST_AM),
 247:                        !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#SRC_AM)>;
 248: 
 249:   def MOV32 # DST_AM # SRC_AM # !cast<MxType>("MxType32").Postfix
 250:       : MxMove_MM_Safe<MxType32, Mxstoreloadi32_safe,
 251:                        !cast<MxOpBundle>("MxOp32AddrMode_"#DST_AM),
 252:                        !cast<MxOpBundle>("MxOp32AddrMode_"#SRC_AM),
```
- **EN**: This block declares or refines TableGen records such as `MxMove_MM_Safe`, `MOV8`, `MOV16`, `MOV32`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxMove_MM_Safe`, `MOV8`, `MOV16`, `MOV32` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 253-280 / 第 253-280 行
```tablegen
 253:                        !cast<MxEncMemOp>("MxMoveDstOpEnc_"#DST_AM),
 254:                        !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#SRC_AM)>;
 255: } // foreach SRC_AM
 256: 
 257: // Store ABS(basically pointer) as Immdiate to Mem
 258: def : Pat<(store   MxType32.BPat :$src, MxType32.PPat :$dst),
 259:           (MOV32pi MxType32.POp  :$dst, MxType32.IOp  :$src)>;
 260: 
 261: def : Pat<(store   MxType32.BPat :$src, MxType32.FPat :$dst),
 262:           (MOV32fi MxType32.FOp  :$dst, MxType32.IOp  :$src)>;
 263: 
 264: def : Pat<(store   MxType32.BPat :$src, MxType32.BPat :$dst),
 265:           (MOV32bi MxType32.BOp  :$dst, MxType32.IOp  :$src)>;
 266: 
 267: def : Pat<(store   MxType32.BPat :$src, MxType32.JPat :$dst),
 268:           (MOV32ji MxType32.JOp  :$dst, MxType32.IOp  :$src)>;
 269: 
 270: //===----------------------------------------------------------------------===//
 271: // MOVEQ
 272: //===----------------------------------------------------------------------===//
 273: 
 274: /// ------------+---------+---+-----------------------
 275: ///  F  E  D  C | B  A  9 | 8 | 7  6  5  4  3  2  1  0
 276: /// ------------+---------+---+-----------------------
 277: ///  0  1  1  1 |   REG   | 0 |          DATA
 278: /// ------------+---------+---+-----------------------
 279: 
 280: // No pattern, as all immediate -> register moves are matched to the MOVI pseudo
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-308 / 第 281-308 行
```tablegen
 281: let Defs = [CCR] in
 282: def MOVQ : MxInst<(outs MxDRD32:$dst), (ins Mxi8imm:$imm),
 283:                   "moveq\t$imm, $dst",
 284:                   [(null_frag)]> {
 285:   let Inst = (descend 0b0111, (operand "$dst", 3), 0b0, (operand "$imm", 8));
 286: }
 287: 
 288: //===----------------------------------------------------------------------===//
 289: // MOVEM
 290: //
 291: // The mask is already pre-processed by the save/restore spill hook
 292: //===----------------------------------------------------------------------===//
 293: 
 294: // Direction
 295: defvar MxMOVEM_MR = false;
 296: defvar MxMOVEM_RM = true;
 297: 
 298: // Size
 299: defvar MxMOVEM_W = false;
 300: defvar MxMOVEM_L = true;
 301: 
 302: /// ---------------+-------------+-------------+---------
 303: ///  F  E  D  C  B | A | 9  8  7 | 6 | 5  4  3 | 2  1  0
 304: /// ---------------+---+---------+---+---------+---------
 305: ///  0  1  0  0  1 | D | 0  0  1 | S |   MODE  |   REG
 306: /// ---------------+---+---------+---+---------+---------
 307: ///                  REGISTER LIST MASK
 308: /// -----------------------------------------------------
```
- **EN**: This block declares or refines TableGen records such as `MOVQ`.
- **CN**: 该代码块声明或细化了 `MOVQ` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309: /// D - direction(RM,MR)
 310: /// S - size(W,L)
 311: class MxMOVEMEncoding<MxEncMemOp opnd_enc, bit size, bit direction,
 312:                       string mask_op_name> {
 313:   dag Value = (ascend
 314:     (descend 0b01001, direction, 0b001, size, opnd_enc.EA),
 315:     // Mask
 316:     (operand "$"#mask_op_name, 16),
 317:     opnd_enc.Supplement
 318:   );
 319: }
 320: 
 321: let mayStore = 1 in
 322: class MxMOVEM_MR<MxType TYPE, bit SIZE_ENC,
 323:                  MxOperand MEMOp, MxEncMemOp MEM_ENC,
 324:                  MxOp MASKOp>
 325:     : MxInst<(outs), (ins MEMOp:$dst, MASKOp:$mask),
 326:              "movem."#TYPE.Prefix#"\t$mask, $dst", []> {
 327:   let Inst = MxMOVEMEncoding<MEM_ENC, SIZE_ENC, MxMOVEM_MR, "mask">.Value;
 328: }
 329: 
 330: foreach AM = MxMoveSupportedAMs in {
 331:   foreach TYPE = [MxType16, MxType32] in
 332:   def MOVM # TYPE.Size # AM # m # TYPE.Postfix
 333:       : MxMOVEM_MR<TYPE, !if(!eq(TYPE, MxType16), MxMOVEM_W, MxMOVEM_L),
 334:                    !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#AM).Op,
 335:                    !cast<MxEncMemOp>("MxMoveDstOpEnc_"#AM),
 336:                    !if(!eq(AM, "e"), MxInverseMoveMask, MxMoveMask)>;
```
- **EN**: This block declares or refines TableGen records such as `MxMOVEMEncoding`, `MxMOVEM_MR`, `MOVM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxMOVEMEncoding`, `MxMOVEM_MR`, `MOVM` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 337-364 / 第 337-364 行
```tablegen
 337: } // foreach AM
 338: 
 339: let mayLoad = 1 in
 340: class MxMOVEM_RM<MxType TYPE, bit SIZE_ENC,
 341:                  MxOperand MEMOp, MxEncMemOp MEM_ENC,
 342:                  MxOp MASKOp>
 343:     : MxInst<(outs), (ins MASKOp:$mask, MEMOp:$src),
 344:              "movem."#TYPE.Prefix#"\t$src, $mask", []> {
 345:   let Inst = MxMOVEMEncoding<MEM_ENC, SIZE_ENC, MxMOVEM_RM, "mask">.Value;
 346: }
 347: 
 348: foreach AM = MxMoveSupportedAMs in {
 349:   foreach TYPE = [MxType16, MxType32] in
 350:   def MOVM # TYPE.Size # m # AM # TYPE.Postfix
 351:       : MxMOVEM_RM<TYPE, !if(!eq(TYPE, MxType16), MxMOVEM_W, MxMOVEM_L),
 352:                    !cast<MxOpBundle>("MxOp"#TYPE.Size#"AddrMode_"#AM).Op,
 353:                    !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#AM),
 354:                    !if(!eq(AM, "e"), MxInverseMoveMask, MxMoveMask)>;
 355: } // foreach AM
 356: 
 357: // Pseudo versions. These a required by virtual register spill/restore since
 358: // the mask requires real register to encode. These instruction will be expanded
 359: // into real MOVEM after RA finishes.
 360: let mayStore = 1 in
 361: class MxMOVEM_MR_Pseudo<MxType TYPE, MxOperand MEMOp>
 362:     : MxPseudo<(outs), (ins MEMOp:$dst, TYPE.ROp:$reg)>;
 363: let mayLoad = 1 in
 364: class MxMOVEM_RM_Pseudo<MxType TYPE, MxOperand MEMOp>
```
- **EN**: This block declares or refines TableGen records such as `MxMOVEM_RM`, `MOVM`, `MxMOVEM_MR_Pseudo`, `MxMOVEM_RM_Pseudo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxMOVEM_RM`, `MOVM`, `MxMOVEM_MR_Pseudo`, `MxMOVEM_RM_Pseudo` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 365-392 / 第 365-392 行
```tablegen
 365:     : MxPseudo<(outs TYPE.ROp:$dst), (ins MEMOp:$src)>;
 366: 
 367: // Mem <- Reg
 368: def MOVM16jm_P : MxMOVEM_MR_Pseudo<MxType16r, MxType16.JOp>;
 369: def MOVM32jm_P : MxMOVEM_MR_Pseudo<MxType32r, MxType32.JOp>;
 370: 
 371: def MOVM16pm_P : MxMOVEM_MR_Pseudo<MxType16r, MxType16.POp>;
 372: def MOVM32pm_P : MxMOVEM_MR_Pseudo<MxType32r, MxType32.POp>;
 373: 
 374: // Reg <- Mem
 375: def MOVM16mj_P : MxMOVEM_RM_Pseudo<MxType16r, MxType16.JOp>;
 376: def MOVM32mj_P : MxMOVEM_RM_Pseudo<MxType32r, MxType32.JOp>;
 377: 
 378: def MOVM16mp_P : MxMOVEM_RM_Pseudo<MxType16r, MxType16.POp>;
 379: def MOVM32mp_P : MxMOVEM_RM_Pseudo<MxType32r, MxType32.POp>;
 380: 
 381: 
 382: //===----------------------------------------------------------------------===//
 383: // MOVE to/from SR/CCR
 384: //===----------------------------------------------------------------------===//
 385: 
 386: /// Move to CCR
 387: /// --------------------------------------------------
 388: ///  F  E  D  C  B  A  9  8  7  6 | 5  4  3 | 2  1  0
 389: /// --------------------------------------------------
 390: ///                               | EFFECTIVE ADDRESS
 391: ///  0  1  0  0  0  1  0  0  1  1 |   MODE  |   REG
 392: /// --------------------------------------------------
```
- **EN**: This block declares or refines TableGen records such as `MOVM16jm_P`, `MOVM32jm_P`, `MOVM16pm_P`, `MOVM32pm_P`, `MOVM16mj_P`, `MOVM32mj_P`.
- **CN**: 该代码块声明或细化了 `MOVM16jm_P`, `MOVM32jm_P`, `MOVM16pm_P`, `MOVM32pm_P`, `MOVM16mj_P`, `MOVM32mj_P` 等 TableGen 记录。

### Lines 393-420 / 第 393-420 行
```tablegen
 393: let Defs = [CCR] in {
 394: class MxMoveToCCR<MxOperand MEMOp, MxEncMemOp SRC_ENC>
 395:     : MxInst<(outs CCRC:$dst), (ins MEMOp:$src), "move.w\t$src, $dst", []> {
 396:   let Inst = (ascend
 397:     (descend 0b0100010011, SRC_ENC.EA),
 398:     SRC_ENC.Supplement
 399:   );
 400: }
 401: 
 402: class MxMoveToCCRPseudo<MxOperand MEMOp>
 403:     : MxPseudo<(outs CCRC:$dst), (ins MEMOp:$src)>;
 404: } // let Defs = [CCR]
 405: 
 406: let mayLoad = 1 in
 407: foreach AM = MxMoveSupportedAMs in {
 408:   def MOV16c # AM : MxMoveToCCR<!cast<MxOpBundle>("MxOp16AddrMode_"#AM).Op,
 409:                                 !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#AM)>;
 410:   def MOV8c # AM  : MxMoveToCCRPseudo<!cast<MxOpBundle>("MxOp8AddrMode_"#AM).Op>;
 411: } // foreach AM
 412: 
 413: // Only data register is allowed.
 414: def MOV16cd : MxMoveToCCR<MxOp16AddrMode_d.Op, MxMoveSrcOpEnc_d>;
 415: 
 416: /// Move from CCR
 417: /// --------------------------------------------------
 418: ///  F  E  D  C  B  A  9  8  7  6 | 5  4  3 | 2  1  0
 419: /// --------------------------------------------------
 420: ///                               | EFFECTIVE ADDRESS
```
- **EN**: This block declares or refines TableGen records such as `MxMoveToCCR`, `MxMoveToCCRPseudo`, `MOV16c`, `MOV8c`, `MOV16cd`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxMoveToCCR`, `MxMoveToCCRPseudo`, `MOV16c`, `MOV8c`, `MOV16cd` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 421-448 / 第 421-448 行
```tablegen
 421: ///  0  1  0  0  0  0  1  0  1  1 |   MODE  |   REG
 422: /// --------------------------------------------------
 423: let Uses = [CCR] in {
 424: class MxMoveFromCCR_R
 425:     : MxInst<(outs MxDRD16:$dst), (ins CCRC:$src), "move.w\t$src, $dst", []>,
 426:       Requires<[ AtLeastM68010 ]> {
 427:   let Inst = (descend 0b0100001011, MxEncAddrMode_d<"dst">.EA);
 428: }
 429: 
 430: class MxMoveFromCCR_M<MxOperand MEMOp, MxEncMemOp DST_ENC>
 431:     : MxInst<(outs), (ins MEMOp:$dst, CCRC:$src), "move.w\t$src, $dst", []>,
 432:       Requires<[ AtLeastM68010 ]> {
 433:   let Inst = (ascend
 434:     (descend 0b0100001011, DST_ENC.EA),
 435:     DST_ENC.Supplement
 436:   );
 437: }
 438: 
 439: class MxMoveFromCCRPseudo<MxOperand MEMOp>
 440:     : MxPseudo<(outs), (ins MEMOp:$dst, CCRC:$src)>;
 441: class MxMoveFromCCR_RPseudo<MxOperand MEMOp>
 442:     : MxPseudo<(outs MEMOp:$dst), (ins CCRC:$src)>;
 443: } // let Uses = [CCR]
 444: 
 445: let mayStore = 1 in
 446: foreach AM = MxMoveSupportedAMs in {
 447:   def MOV16 # AM # c
 448:     : MxMoveFromCCR_M<!cast<MxOpBundle>("MxOp16AddrMode_"#AM).Op,
```
- **EN**: This block declares or refines TableGen records such as `MxMoveFromCCR_R`, `MxMoveFromCCR_M`, `MxMoveFromCCRPseudo`, `MxMoveFromCCR_RPseudo`, `MOV16`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxMoveFromCCR_R`, `MxMoveFromCCR_M`, `MxMoveFromCCRPseudo`, `MxMoveFromCCR_RPseudo`, `MOV16` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 449-476 / 第 449-476 行
```tablegen
 449:                       !cast<MxEncMemOp>("MxMoveDstOpEnc_"#AM)>;
 450:   def MOV8 # AM # c
 451:     : MxMoveFromCCRPseudo<!cast<MxOpBundle>("MxOp8AddrMode_"#AM).Op>;
 452: } // foreach AM
 453: 
 454: // Only data register is allowed.
 455: def MOV16dc : MxMoveFromCCR_R;
 456: 
 457: /// Move to SR
 458: /// --------------------------------------------------
 459: ///  F  E  D  C  B  A  9  8  7  6 | 5  4  3 | 2  1  0
 460: /// --------------------------------------------------
 461: ///                               | EFFECTIVE ADDRESS
 462: ///  0  1  0  0  0  1  1  0  1  1 |   MODE  |   REG
 463: /// --------------------------------------------------
 464: let Defs = [SR] in {
 465: class MxMoveToSR<MxOperand MEMOp, MxEncMemOp SRC_ENC>
 466:     : MxInst<(outs SRC:$dst), (ins MEMOp:$src), "move.w\t$src, $dst", []> {
 467:   let Inst = (ascend
 468:     (descend 0b0100011011, SRC_ENC.EA),
 469:     SRC_ENC.Supplement
 470:   );
 471: }
 472: } // let Defs = [SR]
 473: 
 474: let mayLoad = 1 in
 475: foreach AM = MxMoveSupportedAMs in {
 476:   def MOV16s # AM : MxMoveToSR<!cast<MxOpBundle>("MxOp16AddrMode_"#AM).Op,
```
- **EN**: This block declares or refines TableGen records such as `MOV8`, `MOV16dc`, `MxMoveToSR`, `MOV16s`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MOV8`, `MOV16dc`, `MxMoveToSR`, `MOV16s` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 477-504 / 第 477-504 行
```tablegen
 477:                                 !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#AM)>;
 478: } // foreach AM
 479: 
 480: def MOV16sd : MxMoveToSR<MxOp16AddrMode_d.Op, MxMoveSrcOpEnc_d>;
 481: 
 482: /// Move from SR
 483: /// --------------------------------------------------
 484: ///  F  E  D  C  B  A  9  8  7  6 | 5  4  3 | 2  1  0
 485: /// --------------------------------------------------
 486: ///                               | EFFECTIVE ADDRESS
 487: ///  0  1  0  0  0  0  0  0  1  1 |   MODE  |   REG
 488: /// --------------------------------------------------
 489: let Uses = [SR] in {
 490: class MxMoveFromSR_R
 491:     : MxInst<(outs MxDRD16:$dst), (ins SRC:$src), "move.w\t$src, $dst", []> {
 492:   let Inst = (descend 0b0100000011, MxEncAddrMode_d<"dst">.EA);
 493: }
 494: 
 495: class MxMoveFromSR_M<MxOperand MEMOp, MxEncMemOp DST_ENC>
 496:     : MxInst<(outs), (ins MEMOp:$dst, SRC:$src), "move.w\t$src, $dst", []> {
 497:   let Inst = (ascend
 498:     (descend 0b0100000011, DST_ENC.EA),
 499:     DST_ENC.Supplement
 500:   );
 501: }
 502: } // let Uses = [SR]
 503: 
 504: let mayStore = 1 in
```
- **EN**: This block declares or refines TableGen records such as `MOV16sd`, `MxMoveFromSR_R`, `MxMoveFromSR_M`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MOV16sd`, `MxMoveFromSR_R`, `MxMoveFromSR_M` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 505-532 / 第 505-532 行
```tablegen
 505: foreach AM = MxMoveSupportedAMs in {
 506:   def MOV16 # AM # s
 507:     : MxMoveFromSR_M<!cast<MxOpBundle>("MxOp16AddrMode_"#AM).Op,
 508:                       !cast<MxEncMemOp>("MxMoveDstOpEnc_"#AM)>;
 509: } // foreach AM
 510: 
 511: def MOV16ds : MxMoveFromSR_R;
 512: 
 513: //===----------------------------------------------------------------------===//
 514: // LEA
 515: //===----------------------------------------------------------------------===//
 516: 
 517: /// ----------------------------------------------------
 518: ///  F  E  D  C | B  A  9 | 8  7  6 | 5  4  3 | 2  1  0
 519: /// ----------------------------------------------------
 520: ///  0  1  0  0 | DST REG | 1  1  1 |   MODE  |   REG
 521: /// ----------------------------------------------------
 522: class MxLEA<MxOpBundle SRC, MxEncMemOp SRC_ENC>
 523:     : MxInst<(outs MxARD32:$dst), (ins SRC.Op:$src),
 524:              "lea\t$src, $dst", [(set i32:$dst, SRC.Pat:$src)]> {
 525:   let Inst = (ascend
 526:     (descend 0b0100, (operand "$dst", 3), 0b111, SRC_ENC.EA),
 527:     SRC_ENC.Supplement
 528:   );
 529: }
 530: 
 531: foreach AM = ["p", "f", "b", "q", "k"] in
 532: def LEA32 # AM : MxLEA<!cast<MxOpBundle>("MxOp32AddrMode_"#AM),
```
- **EN**: This block declares or refines TableGen records such as `MOV16`, `MOV16ds`, `MxLEA`, `LEA32`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MOV16`, `MOV16ds`, `MxLEA`, `LEA32` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 533-560 / 第 533-560 行
```tablegen
 533:                        !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#AM)>;
 534: 
 535: //===----------------------------------------------------------------------===//
 536: // LINK/UNLK
 537: //===----------------------------------------------------------------------===//
 538: 
 539: let Uses = [SP], Defs = [SP] in {
 540: let mayStore = 1 in {
 541: 
 542: def LINK16 : MxInst<(outs), (ins MxARD16:$src, Mxi16imm:$disp), "link.w\t$src, $disp", []> {
 543:   let Inst = (ascend
 544:     (descend 0b0100111001010, (operand "$src", 3)),
 545:     (operand "$disp", 16)
 546:   );
 547: }
 548: 
 549: def LINK32 : MxInst<(outs), (ins MxARD16:$src, Mxi32imm:$disp), "link.l\t$src, $disp", []> {
 550:   let Inst = (ascend
 551:     (descend 0b0100100000001, (operand "$src", 3)),
 552:     (slice "$disp", 31, 16),
 553:     (slice "$disp", 15, 0)
 554:   );
 555: }
 556: 
 557: def UNLK : MxInst<(outs), (ins MxARD32:$src), "unlk\t$src", []> {
 558:   let Inst = (descend 0b0100111001011, (operand "$src", 3));
 559: }
 560: 
```
- **EN**: This block declares or refines TableGen records such as `LINK16`, `LINK32`, `UNLK`.
- **CN**: 该代码块声明或细化了 `LINK16`, `LINK32`, `UNLK` 等 TableGen 记录。

### Lines 561-588 / 第 561-588 行
```tablegen
 561: } // let mayStore = 1
 562: } // let Uses = [SP], Defs = [SP]
 563: 
 564: //===----------------------------------------------------------------------===//
 565: // Pseudos
 566: //===----------------------------------------------------------------------===//
 567: 
 568: /// Pushe/Pop to/from SP for simplicity
 569: let Uses = [SP], Defs = [SP], hasSideEffects = 0 in {
 570: 
 571: // SP <- SP - <size>; (SP) <- Dn
 572: let mayStore = 1 in {
 573: def PUSH8d  : MxPseudo<(outs), (ins DR8:$reg)>;
 574: def PUSH16d : MxPseudo<(outs), (ins DR16:$reg)>;
 575: def PUSH32r : MxPseudo<(outs), (ins XR32:$reg)>;
 576: } // let mayStore = 1
 577: 
 578: // Dn <- (SP); SP <- SP + <size>
 579: let mayLoad = 1 in {
 580: def POP8d  : MxPseudo<(outs DR8:$reg),  (ins)>;
 581: def POP16d : MxPseudo<(outs DR16:$reg), (ins)>;
 582: def POP32r : MxPseudo<(outs XR32:$reg), (ins)>;
 583: } // let mayLoad = 1
 584: 
 585: } // let Uses/Defs = [SP], hasSideEffects = 0
 586: 
 587: 
 588: let Defs = [CCR] in {
```
- **EN**: This block declares or refines TableGen records such as `PUSH8d`, `PUSH16d`, `PUSH32r`, `POP8d`, `POP16d`, `POP32r`.
- **CN**: 该代码块声明或细化了 `PUSH8d`, `PUSH16d`, `PUSH32r`, `POP8d`, `POP16d`, `POP32r` 等 TableGen 记录。

### Lines 589-616 / 第 589-616 行
```tablegen
 589: class MxPseudoMove_RR<MxType DST, MxType SRC, list<dag> PAT = []>
 590:     : MxPseudo<(outs DST.ROp:$dst), (ins SRC.ROp:$src), PAT>;
 591: 
 592: class MxPseudoMove_RM<MxType DST, MxOperand SRCOpd, list<dag> PAT = []>
 593:     : MxPseudo<(outs DST.ROp:$dst), (ins SRCOpd:$src), PAT>;
 594: 
 595: 
 596: // These Pseudos handle loading immediates to registers.
 597: // They are expanded post-RA into either move or moveq instructions,
 598: // depending on size, destination register class, and immediate value.
 599: // This is done with pseudoinstructions in order to not constrain RA to
 600: // data registers if moveq matches.
 601: class MxPseudoMove_DI<MxType TYPE>
 602:     : MxPseudo<(outs TYPE.ROp:$dst), (ins TYPE.IOp:$src),
 603:                [(set TYPE.ROp:$dst, imm:$src)]>;
 604: 
 605: // i8 imm -> reg can always be converted to moveq,
 606: // but we still emit a pseudo for consistency.
 607: def MOVI8di  : MxPseudoMove_DI<MxType8d>;
 608: def MOVI16ri : MxPseudoMove_DI<MxType16r>;
 609: def MOVI32ri : MxPseudoMove_DI<MxType32r>;
 610: } // let Defs = [CCR]
 611: 
 612: /// This group of Pseudos is analogues to the real x86 extending moves, but
 613: /// since M68k does not have those we need to emulate. These instructions
 614: /// will be expanded right after RA completed because we need to know precisely
 615: /// what registers are allocated for the operands and if they overlap we just
 616: /// extend the value if the registers are completely different we need to move
```
- **EN**: This block declares or refines TableGen records such as `MxPseudoMove_RR`, `MxPseudoMove_RM`, `MxPseudoMove_DI`, `MOVI8di`, `MOVI16ri`, `MOVI32ri`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxPseudoMove_RR`, `MxPseudoMove_RM`, `MxPseudoMove_DI`, `MOVI8di`, `MOVI16ri`, `MOVI32ri` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 617-644 / 第 617-644 行
```tablegen
 617: /// first.
 618: foreach EXT = ["S", "Z"] in {
 619:   let hasSideEffects = 0 in {
 620: 
 621:     def MOV#EXT#Xd16d8  : MxPseudoMove_RR<MxType16d,  MxType8d>;
 622:     def MOV#EXT#Xd32d8  : MxPseudoMove_RR<MxType32d,  MxType8d>;
 623:     def MOV#EXT#Xd32d16 : MxPseudoMove_RR<MxType32r, MxType16r>;
 624: 
 625:     let mayLoad = 1 in {
 626: 
 627:       def MOV#EXT#Xd16j8   : MxPseudoMove_RM<MxType16d,  MxType8.JOp>;
 628:       def MOV#EXT#Xd32j8   : MxPseudoMove_RM<MxType32d,  MxType8.JOp>;
 629:       def MOV#EXT#Xd32j16  : MxPseudoMove_RM<MxType32d, MxType16.JOp>;
 630: 
 631:       def MOV#EXT#Xd16p8   : MxPseudoMove_RM<MxType16d,  MxType8.POp>;
 632:       def MOV#EXT#Xd32p8   : MxPseudoMove_RM<MxType32d,  MxType8.POp>;
 633:       def MOV#EXT#Xd32p16  : MxPseudoMove_RM<MxType32d, MxType16.POp>;
 634: 
 635:       def MOV#EXT#Xd16f8   : MxPseudoMove_RM<MxType16d,  MxType8.FOp>;
 636:       def MOV#EXT#Xd32f8   : MxPseudoMove_RM<MxType32d,  MxType8.FOp>;
 637:       def MOV#EXT#Xd32f16  : MxPseudoMove_RM<MxType32d, MxType16.FOp>;
 638: 
 639:       def MOV#EXT#Xd16q8   : MxPseudoMove_RM<MxType16d,  MxType8.QOp>;
 640:       def MOV#EXT#Xd32q8   : MxPseudoMove_RM<MxType32d,  MxType8.QOp>;
 641:       def MOV#EXT#Xd32q16  : MxPseudoMove_RM<MxType32d,  MxType16.QOp>;
 642: 
 643:     }
 644:   }
```
- **EN**: This block declares or refines TableGen records such as `MOV`, `MOV`, `MOV`, `MOV`, `MOV`, `MOV`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MOV`, `MOV`, `MOV`, `MOV`, `MOV`, `MOV` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 645-672 / 第 645-672 行
```tablegen
 645: }
 646: 
 647: /// This group of instructions is similar to the group above but DOES NOT do
 648: /// any value extension, they just load a smaller register into the lower part
 649: /// of another register if operands' real registers are different or does
 650: /// nothing if they are the same.
 651: def MOVXd16d8  : MxPseudoMove_RR<MxType16d,  MxType8d>;
 652: def MOVXd32d8  : MxPseudoMove_RR<MxType32d,  MxType8d>;
 653: def MOVXd32d16 : MxPseudoMove_RR<MxType32r, MxType16r>;
 654: 
 655: //===----------------------------------------------------------------------===//
 656: // Extend/Truncate Patterns
 657: //===----------------------------------------------------------------------===//
 658: 
 659: // i16 <- sext i8
 660: def: Pat<(i16 (sext i8:$src)),
 661:           (EXTRACT_SUBREG (MOVSXd32d8 MxDRD8:$src), MxSubRegIndex16Lo)>;
 662: def: Pat<(MxSExtLoadi16i8 MxCP_ARI:$src),
 663:           (EXTRACT_SUBREG (MOVSXd32j8 MxARI8:$src), MxSubRegIndex16Lo)>;
 664: def: Pat<(MxSExtLoadi16i8 MxCP_ARID:$src),
 665:           (EXTRACT_SUBREG (MOVSXd32p8 MxARID8:$src), MxSubRegIndex16Lo)>;
 666: def: Pat<(MxSExtLoadi16i8 MxCP_ARII:$src),
 667:           (EXTRACT_SUBREG (MOVSXd32f8 MxARII8:$src), MxSubRegIndex16Lo)>;
 668: def: Pat<(MxSExtLoadi16i8 MxCP_PCD:$src), (MOVSXd16q8 MxPCD8:$src)>;
 669: 
 670: // i32 <- sext i8
 671: def: Pat<(i32 (sext i8:$src)), (MOVSXd32d8 MxDRD8:$src)>;
 672: def: Pat<(MxSExtLoadi32i8 MxCP_ARI :$src), (MOVSXd32j8 MxARI8 :$src)>;
```
- **EN**: This block declares or refines TableGen records such as `MOVXd16d8`, `MOVXd32d8`, `MOVXd32d16`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MOVXd16d8`, `MOVXd32d8`, `MOVXd32d16` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 673-700 / 第 673-700 行
```tablegen
 673: def: Pat<(MxSExtLoadi32i8 MxCP_ARID:$src), (MOVSXd32p8 MxARID8:$src)>;
 674: def: Pat<(MxSExtLoadi32i8 MxCP_ARII:$src), (MOVSXd32f8 MxARII8:$src)>;
 675: def: Pat<(MxSExtLoadi32i8 MxCP_PCD:$src),  (MOVSXd32q8 MxPCD8:$src)>;
 676: 
 677: // i32 <- sext i16
 678: def: Pat<(i32 (sext i16:$src)), (MOVSXd32d16 MxDRD16:$src)>;
 679: def: Pat<(MxSExtLoadi32i16 MxCP_ARI :$src), (MOVSXd32j16 MxARI16 :$src)>;
 680: def: Pat<(MxSExtLoadi32i16 MxCP_ARID:$src), (MOVSXd32p16 MxARID16:$src)>;
 681: def: Pat<(MxSExtLoadi32i16 MxCP_ARII:$src), (MOVSXd32f16 MxARII16:$src)>;
 682: def: Pat<(MxSExtLoadi32i16 MxCP_PCD:$src),  (MOVSXd32q16 MxPCD16:$src)>;
 683: 
 684: // i16 <- zext i8
 685: def: Pat<(i16 (zext i8:$src)),
 686:           (EXTRACT_SUBREG (MOVZXd32d8 MxDRD8:$src), MxSubRegIndex16Lo)>;
 687: def: Pat<(MxZExtLoadi16i8 MxCP_ARI:$src),
 688:           (EXTRACT_SUBREG (MOVZXd32j8 MxARI8:$src), MxSubRegIndex16Lo)>;
 689: def: Pat<(MxZExtLoadi16i8 MxCP_ARID:$src),
 690:           (EXTRACT_SUBREG (MOVZXd32p8 MxARID8:$src), MxSubRegIndex16Lo)>;
 691: def: Pat<(MxZExtLoadi16i8 MxCP_ARII:$src),
 692:           (EXTRACT_SUBREG (MOVZXd32f8 MxARII8:$src), MxSubRegIndex16Lo)>;
 693: def: Pat<(MxZExtLoadi16i8 MxCP_PCD :$src), (MOVZXd16q8 MxPCD8 :$src)>;
 694: 
 695: // i32 <- zext i8
 696: def: Pat<(i32 (zext i8:$src)), (MOVZXd32d8 MxDRD8:$src)>;
 697: def: Pat<(MxZExtLoadi32i8 MxCP_ARI :$src), (MOVZXd32j8 MxARI8 :$src)>;
 698: def: Pat<(MxZExtLoadi32i8 MxCP_ARID:$src), (MOVZXd32p8 MxARID8:$src)>;
 699: def: Pat<(MxZExtLoadi32i8 MxCP_ARII:$src), (MOVZXd32f8 MxARII8:$src)>;
 700: def: Pat<(MxZExtLoadi32i8 MxCP_PCD :$src), (MOVZXd32q8 MxPCD8 :$src)>;
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 701-728 / 第 701-728 行
```tablegen
 701: 
 702: // i32 <- zext i16
 703: def: Pat<(i32 (zext i16:$src)), (MOVZXd32d16 MxDRD16:$src)>;
 704: def: Pat<(MxZExtLoadi32i16 MxCP_ARI :$src), (MOVZXd32j16 MxARI16 :$src)>;
 705: def: Pat<(MxZExtLoadi32i16 MxCP_ARID:$src), (MOVZXd32p16 MxARID16:$src)>;
 706: def: Pat<(MxZExtLoadi32i16 MxCP_ARII:$src), (MOVZXd32f16 MxARII16:$src)>;
 707: def: Pat<(MxZExtLoadi32i16 MxCP_PCD :$src), (MOVZXd32q16 MxPCD16 :$src)>;
 708: 
 709: // i16 <- anyext i8
 710: def: Pat<(i16 (anyext i8:$src)),
 711:           (EXTRACT_SUBREG (MOVZXd32d8 MxDRD8:$src), MxSubRegIndex16Lo)>;
 712: def: Pat<(MxExtLoadi16i8 MxCP_ARI:$src),
 713:           (EXTRACT_SUBREG (MOVZXd32j8 MxARI8:$src), MxSubRegIndex16Lo)>;
 714: def: Pat<(MxExtLoadi16i8 MxCP_ARID:$src),
 715:           (EXTRACT_SUBREG (MOVZXd32p8 MxARID8:$src), MxSubRegIndex16Lo)>;
 716: def: Pat<(MxExtLoadi16i8 MxCP_ARII:$src),
 717:           (EXTRACT_SUBREG (MOVZXd32f8 MxARII8:$src), MxSubRegIndex16Lo)>;
 718: def: Pat<(MxExtLoadi16i8 MxCP_PCD:$src),
 719:           (EXTRACT_SUBREG (MOVZXd32q8 MxPCD8:$src), MxSubRegIndex16Lo)>;
 720: 
 721: // i32 <- anyext i8
 722: def: Pat<(i32 (anyext i8:$src)), (MOVZXd32d8 MxDRD8:$src)>;
 723: def: Pat<(MxExtLoadi32i8 MxCP_ARI :$src), (MOVZXd32j8 MxARI8 :$src)>;
 724: def: Pat<(MxExtLoadi32i8 MxCP_ARID:$src), (MOVZXd32p8 MxARID8:$src)>;
 725: def: Pat<(MxExtLoadi32i8 MxCP_ARII:$src), (MOVZXd32f8 MxARII8:$src)>;
 726: def: Pat<(MxExtLoadi32i8 MxCP_PCD:$src), (MOVZXd32q8 MxPCD8:$src)>;
 727: 
 728: // i32 <- anyext i16
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 729-756 / 第 729-756 行
```tablegen
 729: def: Pat<(i32 (anyext i16:$src)), (MOVZXd32d16 MxDRD16:$src)>;
 730: def: Pat<(MxExtLoadi32i16 MxCP_ARI :$src), (MOVZXd32j16 MxARI16 :$src)>;
 731: def: Pat<(MxExtLoadi32i16 MxCP_ARID:$src), (MOVZXd32p16 MxARID16:$src)>;
 732: def: Pat<(MxExtLoadi32i16 MxCP_ARII:$src), (MOVZXd32f16 MxARII16:$src)>;
 733: def: Pat<(MxExtLoadi32i16 MxCP_PCD:$src), (MOVZXd32q16 MxPCD16:$src)>;
 734: 
 735: // trunc patterns
 736: def : Pat<(i16 (trunc i32:$src)),
 737:           (EXTRACT_SUBREG MxXRD32:$src, MxSubRegIndex16Lo)>;
 738: def : Pat<(i8  (trunc i32:$src)),
 739:           (EXTRACT_SUBREG MxXRD32:$src, MxSubRegIndex8Lo)>;
 740: def : Pat<(i8  (trunc i16:$src)),
 741:           (EXTRACT_SUBREG MxXRD16:$src, MxSubRegIndex8Lo)>;
 742: 
 743: //===----------------------------------------------------------------------===//
 744: // FMOVE
 745: //===----------------------------------------------------------------------===//
 746: 
 747: let Defs = [FPS] in
 748: class MxFMove<string size, dag outs, dag ins, list<dag> pattern,
 749:               string rounding = "">
 750:     : MxInst<outs, ins,
 751:              "f"#rounding#"move."#size#"\t$src, $dst", pattern> {
 752:   // Only FMOVE uses FPC
 753:   let Uses = !if(!eq(rounding, ""), [FPC], []);
 754: 
 755:   // FSMOVE and FDMOVE are only available after M68040
 756:   let Predicates = [!if(!eq(rounding, ""), AtLeastM68881, AtLeastM68040)];
```
- **EN**: This block declares or refines TableGen records such as `MxFMove`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxFMove` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 757-784 / 第 757-784 行
```tablegen
 757: }
 758: 
 759: // FPDR <- FPDR
 760: class MxFMove_FF<string rounding, int size,
 761:                  MxOpBundle Opnd = !cast<MxOpBundle>("MxOp"#size#"AddrMode_fpr")>
 762:     : MxFMove<"x", (outs Opnd.Op:$dst), (ins Opnd.Op:$src),
 763:               [(null_frag)], rounding> {
 764:   let Inst = (ascend
 765:     (descend 0b1111,
 766:       /*COPROCESSOR ID*/0b001,
 767:       0b000,
 768:       /*MODE + REGISTER*/0b000000
 769:     ),
 770:     (descend 0b0, /* R/M */0b0, 0b0,
 771:       /*SOURCE SPECIFIER*/
 772:       (operand "$src", 3),
 773:       /*DESTINATION*/
 774:       (operand "$dst", 3),
 775:       /*OPMODE*/
 776:       !cond(!eq(rounding, "s"): 0b1000000,
 777:             !eq(rounding, "d"): 0b1000100,
 778:             true: 0b0000000)
 779:     )
 780:   );
 781: }
 782: 
 783: foreach rounding = ["", "s", "d"] in {
 784:   def F # !toupper(rounding) # MOV80fp_fp : MxFMove_FF<rounding, 80>;
```
- **EN**: This block declares or refines TableGen records such as `MxFMove_FF`, `F`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxFMove_FF`, `F` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 785-812 / 第 785-812 行
```tablegen
 785: 
 786:   // We don't have `fmove.s` or `fmove.d` because values will be converted to
 787:   // f80 upon storing into the register, but FMOV32/64fp_fp are still needed
 788:   // to make codegen easier.
 789:   let isCodeGenOnly = true in
 790:   foreach size = [32, 64] in
 791:     def F # !toupper(rounding) # MOV # size # fp_fp : MxFMove_FF<rounding, size>;
 792: }
 793: // Direction
 794: defvar MxFMove_FP_EA = false;
 795: defvar MxFMove_EA_FP = true;
 796: 
 797: // Encoding scheme for FPSYS <-> R/M
 798: class MxEncFSysMove<bit dir, MxEncMemOp EAEnc, string fsys_reg> {
 799:   dag Value = (ascend
 800:     (descend 0b1111,
 801:       /*COPROCESSOR ID*/0b001,
 802:       0b000,
 803:       /*MODE + REGISTER*/
 804:       EAEnc.EA
 805:     ),
 806:     (descend 0b10, /*dir*/ dir,
 807:       /*REGISTER SELECT*/
 808:       (operand "$"#fsys_reg, 3, (encoder "encodeFPSYSSelect")),
 809:       0b0000000000
 810:     )
 811:   );
 812: }
```
- **EN**: This block declares or refines TableGen records such as `F`, `MxEncFSysMove`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `F`, `MxEncFSysMove` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 813-838 / 第 813-838 行
```tablegen
 813: 
 814: // FPSYS <-> R
 815: class MxFMove_FSYS_R<string src_reg,
 816:                      MxOpBundle SrcOpnd = !cast<MxOpBundle>("MxOp32AddrMode_"#src_reg),
 817:                      MxOpBundle DstOpnd = !cond(!eq(src_reg, "d"): MxOp32AddrMode_fpcs,
 818:                                                 !eq(src_reg, "a"): MxOp32AddrMode_fpi),
 819:                      MxEncMemOp SrcEnc = !cast<MxEncMemOp>("MxMoveSrcOpEnc_"#src_reg)>
 820:     : MxFMove<"l", (outs DstOpnd.Op:$dst), (ins SrcOpnd.Op:$src),
 821:               [(null_frag)]> {
 822:   let Inst = MxEncFSysMove<MxFMove_FP_EA, SrcEnc, "dst">.Value;
 823: }
 824: 
 825: class MxFMove_R_FSYS<string dst_reg,
 826:                      MxOpBundle SrcOpnd = !cond(!eq(dst_reg, "d"): MxOp32AddrMode_fpcs,
 827:                                                 !eq(dst_reg, "a"): MxOp32AddrMode_fpi),
 828:                      MxOpBundle DstOpnd = !cast<MxOpBundle>("MxOp32AddrMode_"#dst_reg),
 829:                      MxEncMemOp DstEnc = !cast<MxEncMemOp>("MxMoveDstOpEnc_"#dst_reg)>
 830:     : MxFMove<"l", (outs DstOpnd.Op:$dst), (ins SrcOpnd.Op:$src),
 831:               [(null_frag)]> {
 832:   let Inst = MxEncFSysMove<MxFMove_EA_FP, DstEnc, "src">.Value;
 833: }
 834: 
 835: def FMOVE32fpcs_d : MxFMove_FSYS_R<"d">;
 836: def FMOVE32d_fpcs : MxFMove_R_FSYS<"d">;
 837: def FMOVE32fpi_a  : MxFMove_FSYS_R<"a">;
 838: def FMOVE32a_fpi  : MxFMove_R_FSYS<"a">;
```
- **EN**: This block declares or refines TableGen records such as `MxFMove_FSYS_R`, `MxFMove_R_FSYS`, `FMOVE32fpcs_d`, `FMOVE32d_fpcs`, `FMOVE32fpi_a`, `FMOVE32a_fpi`.
- **CN**: 该代码块声明或细化了 `MxFMove_FSYS_R`, `MxFMove_R_FSYS`, `FMOVE32fpcs_d`, `FMOVE32d_fpcs`, `FMOVE32fpi_a`, `FMOVE32a_fpi` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
