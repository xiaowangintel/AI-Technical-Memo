# M68kInstrControl.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrControl.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 M68k 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //===-- M68kInstrControl.td - Control Flow Instructions ----*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file describes the M68k jump, return, call, and related instructions.
  11: /// Here is the current status of the file:
  12: ///
  13: ///  Machine:
  14: ///
  15: ///       BRA   [x]     BSR  [~]     Bcc [~]     DBcc [ ]     FBcc [ ]
  16: ///       FDBcc [ ]     FNOP [ ]     FPn [ ]     FScc [ ]     FTST [ ]
  17: ///       JMP   [~]     JSR  [x]     NOP [x]     RTD  [!]     RTR  [ ]
  18: ///       RTS   [x]     Scc  [~]     TST [ ]
  19: ///
  20: ///  Pseudo:
  21: ///
  22: ///          RET [x]
  23: ///    TCRETURNj [x]   TCRETURNq [x]
  24: ///     TAILJMPj [x]    TAILJMPq [x]
  25: ///
  26: ///  Map:
  27: ///
  28: ///   [ ] - was not touched at all
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 29-56 / 第 29-56 行
```tablegen
  29: ///   [!] - requires extarnal stuff implemented
  30: ///   [~] - in progress but usable
  31: ///   [x] - done
  32: ///
  33: ///
  34: ///                                   NOTE
  35: ///      Though branch and jump instructions are using memory operands they
  36: ///      DO NOT read the jump address from memory, they just calculate EA
  37: ///      and jump there.
  38: ///
  39: //===----------------------------------------------------------------------===//
  40: 
  41: //===----------------------------------------------------------------------===//
  42: // NOP
  43: //===----------------------------------------------------------------------===//
  44: 
  45: let hasSideEffects = 0 in {
  46:   def NOP : MxInst<(outs), (ins), "nop", []> {
  47:     let Inst = (descend 0b0100, 0b1110, 0b0111, 0b0001);
  48:   }
  49: }
  50: 
  51: 
  52: //===----------------------------------------------------------------------===//
  53: // Conditions
  54: //===----------------------------------------------------------------------===//
  55: 
  56: /// CC—Carry clear      GE—Greater than or equal
```
- **EN**: This block declares or refines TableGen records such as `NOP`.
- **CN**: 该代码块声明或细化了 `NOP` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57: /// LS—Lower or same    PL—Plus
  58: /// CS—Carry set        GT—Greater than
  59: /// LT—Less than        T—Always true*
  60: /// EQ—Equal            HI—Higher
  61: /// MI—Minus            VC—Overflow clear
  62: /// F—Never true*       LE—Less than or equal
  63: /// NE—Not equal        VS—Overflow set
  64: ///
  65: /// *Not applicable to the Bcc instructions.
  66: class MxEncCondOp<bits<4> cond> {
  67:   dag Value = (descend cond);
  68: }
  69: 
  70: def MxCCt  : MxEncCondOp<0b0000>;
  71: def MxCCf  : MxEncCondOp<0b0001>;
  72: def MxCChi : MxEncCondOp<0b0010>;
  73: def MxCCls : MxEncCondOp<0b0011>;
  74: def MxCCcc : MxEncCondOp<0b0100>;
  75: def MxCCcs : MxEncCondOp<0b0101>;
  76: def MxCCne : MxEncCondOp<0b0110>;
  77: def MxCCeq : MxEncCondOp<0b0111>;
  78: def MxCCvc : MxEncCondOp<0b1000>;
  79: def MxCCvs : MxEncCondOp<0b1001>;
  80: def MxCCpl : MxEncCondOp<0b1010>;
  81: def MxCCmi : MxEncCondOp<0b1011>;
  82: def MxCCge : MxEncCondOp<0b1100>;
  83: def MxCClt : MxEncCondOp<0b1101>;
  84: def MxCCgt : MxEncCondOp<0b1110>;
```
- **EN**: This block declares or refines TableGen records such as `MxEncCondOp`, `MxCCt`, `MxCCf`, `MxCChi`, `MxCCls`, `MxCCcc`.
- **CN**: 该代码块声明或细化了 `MxEncCondOp`, `MxCCt`, `MxCCf`, `MxCChi`, `MxCCls`, `MxCCcc` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85: def MxCCle : MxEncCondOp<0b1111>;
  86: 
  87: 
  88: 
  89: /// --------------------------------+---------+---------
  90: ///  F  E  D  C | B  A  9  8 | 7  6 | 5  4  3 | 2  1  0
  91: /// --------------------------------+---------+---------
  92: ///  0  1  0  1 | CONDITION  | 1  1 |   MODE  |   REG
  93: /// ----------------------------------------------------
  94: 
  95: let Uses = [CCR] in {
  96: class MxSccR<string CC>
  97:     : MxInst<(outs MxDRD8:$dst), (ins), "s"#CC#"\t$dst",
  98:              [(set i8:$dst, (MxSetCC !cast<PatLeaf>("MxCOND"#CC), CCR))]> {
  99:   let Inst = (descend 0b0101, !cast<MxEncCondOp>("MxCC"#CC).Value, 0b11,
 100:               /*MODE without last bit*/0b00,
 101:               /*REGISTER prefixed with D/A bit*/(operand "$dst", 4));
 102: }
 103: 
 104: class MxSccM<string CC, MxOperand MEMOpd, ComplexPattern MEMPat, MxEncMemOp DST_ENC>
 105:     : MxInst<(outs), (ins MEMOpd:$dst), "s"#CC#"\t$dst",
 106:              [(store (MxSetCC !cast<PatLeaf>("MxCOND"#CC), CCR), MEMPat:$dst)]> {
 107:   let Inst =
 108:     (ascend
 109:       (descend 0b0101, !cast<MxEncCondOp>("MxCC"#CC).Value, 0b11, DST_ENC.EA),
 110:       DST_ENC.Supplement
 111:     );
 112: }
```
- **EN**: This block declares or refines TableGen records such as `MxCCle`, `MxSccR`, `MxSccM`.
- **CN**: 该代码块声明或细化了 `MxCCle`, `MxSccR`, `MxSccM` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113: }
 114: 
 115: foreach cc = [ "cc", "ls", "lt", "eq", "mi", "f", "ne", "ge",
 116:                "cs", "pl", "gt", "t", "hi", "vc", "le", "vs"] in {
 117: def SET#"d8"#cc : MxSccR<cc>;
 118: def SET#"j8"#cc : MxSccM<cc, MxType8.JOp, MxType8.JPat, MxEncAddrMode_j<"dst">>;
 119: def SET#"p8"#cc : MxSccM<cc, MxType8.POp, MxType8.PPat, MxEncAddrMode_p<"dst">>;
 120: }
 121: 
 122: //===----------------------------------------------------------------------===//
 123: // Jumps
 124: //===----------------------------------------------------------------------===//
 125: 
 126: ///------------------------------+---------+---------
 127: /// F  E  D  C  B  A  9  8  7  6 | 5  4  3 | 2  1  0
 128: ///------------------------------+---------+---------
 129: /// 0  1  0  0  1  1  1  0  1  1 |  MODE   |   REG
 130: ///------------------------------+---------+---------
 131: let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1 in
 132: class MxJMP<MxOperand LOCOp, MxEncMemOp DST_ENC>
 133:     : MxInst<(outs), (ins LOCOp:$dst), "jmp\t$dst", [(brind iPTR:$dst)]> {
 134:   let Inst =
 135:     (ascend
 136:       (descend 0b0100, 0b1110, 0b11, DST_ENC.EA),
 137:       DST_ENC.Supplement
 138:     );
 139: }
 140: 
```
- **EN**: This block declares or refines TableGen records such as `SET`, `SET`, `SET`, `MxJMP`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `SET`, `SET`, `SET`, `MxJMP` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: def JMP32j : MxJMP<MxARI32, MxEncAddrMode_j<"dst">>;
 142: 
 143: 
 144: // FIXME Support 16 bit indirect jump.
 145: // Currently M68k does not allow 16 bit indirect jumps use sext operands
 146: // def JMP16r     : MxInst<(outs), (ins M68k_ARI16:$dst),
 147: //                             "jmp\t$dst",
 148: //                             [(brind AR16:$dst)]>;
 149: 
 150: //===----------------------------------------------------------------------===//
 151: // Branches
 152: //===----------------------------------------------------------------------===//
 153: 
 154: /// --------------------------------------------------
 155: ///  F  E  D  C | B  A  9  8 | 7  6  5  4  3  2  1  0
 156: /// --------------------------------------------------
 157: ///  0  1  1  0 | CONDITION |   8-BIT DISPLACEMENT
 158: /// --------------------------------------------------
 159: ///  16-BIT DISPLACEMENT IF 8-BIT DISPLACEMENT = $00
 160: /// --------------------------------------------------
 161: ///  32-BIT DISPLACEMENT IF 8-BIT DISPLACEMENT = $FF
 162: /// --------------------------------------------------
 163: let isBranch = 1, isTerminator = 1, Uses = [CCR] in
 164: class MxBcc<string cc, Operand TARGET, dag disp_8, dag disp_16_32>
 165:     : MxInst<(outs), (ins TARGET:$dst), "b"#cc#"\t$dst", []> {
 166:   // FIXME: If we want to avoid supplying disp_16_32 with empty
 167:   //        (ascend) for 16/32 bits variants, we can use conditional
 168:   //        bang operator like this:
```
- **EN**: This block declares or refines TableGen records such as `JMP32j`, `MxBcc`.
- **CN**: 该代码块声明或细化了 `JMP32j`, `MxBcc` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169:   //        ```
 170:   //        class MxBcc<string cc, Operand TARGET, int SIZE>
 171:   //        ...
 172:   //        let Inst = !cond(
 173:   //            !eq(SIZE, 8):   /* encoding for Bcc8  */
 174:   //            !eq(SIZE, 16):  /* encoding for Bcc16 */
 175:   //            !eq(SIZE, 32):  /* encoding for Bcc32 */
 176:   //        );
 177:   let Inst =
 178:       (ascend
 179:         (descend 0b0110, !cast<MxEncCondOp>("MxCC"#cc).Value, disp_8),
 180:         disp_16_32
 181:       );
 182: 
 183:   let Predicates = !if(!eq(TARGET, MxBrTarget32), [AtLeastM68020], []);
 184: }
 185: 
 186: foreach cc = [ "cc", "ls", "lt", "eq", "mi", "ne", "ge",
 187:                "cs", "pl", "gt", "hi", "vc", "le", "vs"] in {
 188:   def B#cc#"8"
 189:     : MxBcc<cc, MxBrTarget8,
 190:             (operand "$dst", 8, (encoder "encodePCRelImm<8>")), (ascend)>;
 191: 
 192:   def B#cc#"16"
 193:     : MxBcc<cc, MxBrTarget16, (descend 0b0000, 0b0000),
 194:             (operand "$dst", 16, (encoder "encodePCRelImm<16>"))>;
 195: 
 196:   def B#cc#"32"
```
- **EN**: This block declares or refines TableGen records such as `B`, `B`, `B`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `B`, `B`, `B` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```tablegen
 197:     : MxBcc<cc, MxBrTarget32, (descend 0b1111, 0b1111),
 198:             (operand "$dst", 32, (encoder "encodePCRelImm<32>"))>;
 199: }
 200: 
 201: foreach cc = [ "cc", "ls", "lt", "eq", "mi", "ne", "ge",
 202:                "cs", "pl", "gt", "hi", "vc", "le", "vs"] in {
 203: def : Pat<(MxBrCond bb:$target, !cast<PatLeaf>("MxCOND"#cc), CCR),
 204:           (!cast<Instruction>("B"#cc#"8") MxBrTarget8:$target)>;
 205: }
 206: 
 207: /// -------------------------------------------------
 208: ///  F  E  D  C  B  A  9  8 | 7  6  5  4  3  2  1  0
 209: /// -------------------------------------------------
 210: ///  0  1  1  0  0  0  0  0 |   8-BIT DISPLACEMENT
 211: /// -------------------------------------------------
 212: ///  16-BIT DISPLACEMENT IF 8-BIT DISPLACEMENT = $00
 213: /// -------------------------------------------------
 214: ///  32-BIT DISPLACEMENT IF 8-BIT DISPLACEMENT = $FF
 215: /// -------------------------------------------------
 216: let isBranch = 1, isTerminator = 1, isBarrier = 1 in
 217: class MxBra<Operand TARGET, dag disp_8, dag disp_16_32>
 218:     : MxInst<(outs), (ins TARGET:$dst), "bra\t$dst", []> {
 219:   let Inst =
 220:     (ascend
 221:       (descend 0b0110, 0b0000, disp_8),
 222:       disp_16_32
 223:     );
 224:   
```
- **EN**: This block declares or refines TableGen records such as `MxBra`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `MxBra` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 225-252 / 第 225-252 行
```tablegen
 225:   let Predicates = !if(!eq(TARGET, MxBrTarget32), [AtLeastM68020], []);
 226: }
 227: 
 228: def BRA8  : MxBra<MxBrTarget8,
 229:                   (operand "$dst", 8, (encoder "encodePCRelImm<8>")), (ascend)>;
 230: 
 231: def BRA16 : MxBra<MxBrTarget16, (descend 0b0000, 0b0000),
 232:                   (operand "$dst", 16, (encoder "encodePCRelImm<16>"))>;
 233: 
 234: def BRA32 : MxBra<MxBrTarget32, (descend 0b1111, 0b1111),
 235:                   (operand "$dst", 32, (encoder "encodePCRelImm<32>"),
 236:                                        (decoder "DecodeImm32"))>;
 237: 
 238: def : Pat<(br bb:$target), (BRA8 MxBrTarget8:$target)>;
 239: 
 240: /// -------------------------------------------------
 241: ///  F  E  D  C  B  A  9  8 | 7  6  5  4  3  2  1  0
 242: /// -------------------------------------------------
 243: ///  0  1  1  0  0  0  0  1 |   8-BIT DISPLACEMENT
 244: /// -------------------------------------------------
 245: ///  16-BIT DISPLACEMENT IF 8-BIT DISPLACEMENT = $00
 246: /// -------------------------------------------------
 247: ///  32-BIT DISPLACEMENT IF 8-BIT DISPLACEMENT = $FF
 248: /// -------------------------------------------------
 249: 
 250: let isBranch = 1, isTerminator = 1 in
 251: class MxBsr<Operand TARGET, MxType TYPE, dag disp_8, dag disp_16_32>
 252:     : MxInst<(outs), (ins TARGET:$dst), "bsr."#TYPE.Prefix#"\t$dst"> {
```
- **EN**: This block declares or refines TableGen records such as `BRA8`, `BRA16`, `BRA32`, `MxBsr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `BRA8`, `BRA16`, `BRA32`, `MxBsr` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```tablegen
 253:   let Inst = (ascend
 254:                 (descend 0b0110, 0b0001, disp_8),
 255:                  disp_16_32
 256:               );
 257:   let Predicates = !if(!eq(TARGET, MxBrTarget32), [AtLeastM68020], []);
 258: }
 259: 
 260: def BSR8 : MxBsr<MxBrTarget8, MxType8,
 261:                 (operand "$dst", 8, (encoder "encodePCRelImm<8>")), (ascend)>;
 262: 
 263: def BSR16 : MxBsr<MxBrTarget16, MxType16, (descend 0b0000, 0b0000),
 264:                 (operand "$dst", 16, (encoder "encodePCRelImm<16>"))>;
 265: 
 266: def BSR32 : MxBsr<MxBrTarget32, MxType32, (descend 0b1111, 0b1111),
 267:                 (operand "$dst", 32, (encoder "encodePCRelImm<32>"),
 268:                                      (decoder "DecodeImm32"))>;
 269: 
 270: //===----------------------------------------------------------------------===//
 271: // Call
 272: //===----------------------------------------------------------------------===//
 273: 
 274: // All calls clobber the non-callee saved registers. %SP is marked as
 275: // a use to prevent stack-pointer assignments that appear immediately
 276: // before calls from potentially appearing dead. Uses for argument
 277: // registers are added manually.
 278: let Uses = [SP] in
 279: let isCall = 1 in
 280: ///------------------------------+---------+---------
```
- **EN**: This block declares or refines TableGen records such as `BSR8`, `BSR16`, `BSR32`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `BSR8`, `BSR16`, `BSR32` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-308 / 第 281-308 行
```tablegen
 281: /// F  E  D  C  B  A  9  8  7  6 | 5  4  3 | 2  1  0
 282: ///------------------------------+---------+---------
 283: /// 0  1  0  0  1  1  1  0  1  0 |  MODE   |   REG
 284: ///------------------------------+---------+---------
 285: class MxCall<MxOperand LOCOp, MxEncMemOp DST_ENC>
 286:     : MxInst<(outs), (ins LOCOp:$dst), "jsr\t$dst", []> {
 287:   let Inst =
 288:     (ascend
 289:       (descend 0b0100, 0b1110, 0b10, DST_ENC.EA),
 290:       DST_ENC.Supplement
 291:     );
 292: }
 293: 
 294: def CALLk : MxCall<MxPCI32, MxEncAddrMode_k<"dst">>;
 295: def CALLq : MxCall<MxPCD32, MxEncAddrMode_q<"dst">>;
 296: def CALLf : MxCall<MxARII32, MxEncAddrMode_f<"dst">>;
 297: def CALLp : MxCall<MxARID32, MxEncAddrMode_p<"dst">>;
 298: def CALLb : MxCall<MxAL32,  MxEncAddrMode_abs<"dst", true>>;
 299: def CALLj : MxCall<MxARI32, MxEncAddrMode_j<"dst">>;
 300: 
 301: multiclass CallPat<MxCall callOp, Predicate pred> {
 302:   let Predicates = [pred] in {
 303:     def : Pat<(MxCall (i32 tglobaladdr:$dst)),  (callOp tglobaladdr:$dst)>;
 304:     def : Pat<(MxCall (i32 texternalsym:$dst)), (callOp texternalsym:$dst)>;
 305:     def : Pat<(MxCall (i32 imm:$dst)),          (callOp imm:$dst)>;
 306:   }
 307: }
 308: 
```
- **EN**: This block declares or refines TableGen records such as `MxCall`, `CALLk`, `CALLq`, `CALLf`, `CALLp`, `CALLb`.
- **CN**: 该代码块声明或细化了 `MxCall`, `CALLk`, `CALLq`, `CALLf`, `CALLp`, `CALLb` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309: defm : CallPat<CALLq, IsPIC>;
 310: defm : CallPat<CALLb, IsNotPIC>;
 311: 
 312: def : Pat<(MxCall iPTR:$dst), (CALLj MxARI32:$dst)>;
 313: 
 314: //===----------------------------------------------------------------------===//
 315: // Tail Call
 316: //===----------------------------------------------------------------------===//
 317: 
 318: let isCodeGenOnly = 1 in {
 319: let Uses = [SP] in {
 320: let isCall = 1, isTerminator = 1, isBarrier = 1 in {
 321: 
 322: let isReturn = 1 in
 323: def TCRETURNq : MxPseudo<(outs), (ins MxPCD32:$dst,    i32imm:$adj)>;
 324: def TAILJMPq  : MxPseudo<(outs), (ins MxPCD32:$dst)>;
 325: 
 326: // NOTE j does not mean load and jump M68k jmp just calculates EA and jumps
 327: // and it is using Mem form like (An) thus j letter.
 328: let isReturn = 1 in
 329: def TCRETURNj : MxPseudo<(outs), (ins MxARI32_TC:$dst, i32imm:$adj)>;
 330: def TAILJMPj  : MxPseudo<(outs), (ins MxARI32_TC:$dst)>;
 331: } // isCall = 1, isTerminator = 1, isBarrier = 1
 332: } // Uses = [SP]
 333: } // isCodeGenOnly = 1
 334: 
 335: //===----------------------------------------------------------------------===//
 336: // Return
```
- **EN**: This block declares or refines TableGen records such as `TCRETURNq`, `TAILJMPq`, `TCRETURNj`, `TAILJMPj`.
- **CN**: 该代码块声明或细化了 `TCRETURNq`, `TAILJMPq`, `TCRETURNj`, `TAILJMPj` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337: //===----------------------------------------------------------------------===//
 338: 
 339: let isTerminator = 1, isReturn = 1, isBarrier = 1, hasCtrlDep = 1 in {
 340: 
 341: def RTS : MxInst<(outs), (ins), "rts", []> {
 342:   let Inst = (descend 0b0100, 0b1110, 0b0111, 0b0101);
 343: }
 344: 
 345: def RTE: MxInst<(outs), (ins), "rte", []> {
 346:   let Inst = (descend 0b0100, 0b1110, 0b0111, 0b0011);
 347: }
 348: 
 349: let isCodeGenOnly = 1 in
 350: def RET : MxPseudo<(outs), (ins i32imm:$adj, variable_ops),
 351:                    [(MxRet timm:$adj)]>;
 352: } // isTerminator = 1, isReturn = 1, isBarrier = 1, hasCtrlDep = 1
 353: 
 354: //===----------------------------------------------------------------------===//
 355: // SETCC_C Patterns
 356: //===----------------------------------------------------------------------===//
 357: 
 358: // Use subx to materialize carry bit.
 359: let Uses = [CCR], Defs = [CCR], isPseudo = 1 in {
 360: // FIXME These are pseudo ops that should be replaced with Pat<> patterns.
 361: // However, Pat<> can't replicate the destination reg into the inputs of the
 362: // result.
 363: def SETCS_C8d : MxPseudo<(outs MxDRD8:$dst), (ins),
 364:                          [(set MxDRD8:$dst, (MxSetCC_C MxCONDcs, CCR))]>;
```
- **EN**: This block declares or refines TableGen records such as `RTS`, `RTE:`, `RET`, `SETCS_C8d`.
- **CN**: 该代码块声明或细化了 `RTS`, `RTE:`, `RET`, `SETCS_C8d` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365: def SETCS_C16d : MxPseudo<(outs MxDRD16:$dst), (ins),
 366:                           [(set MxDRD16:$dst, (MxSetCC_C MxCONDcs, CCR))]>;
 367: def SETCS_C32d : MxPseudo<(outs MxXRD32:$dst), (ins),
 368:                           [(set MxXRD32:$dst, (MxSetCC_C MxCONDcs, CCR))]>;
 369: } // Uses = [CCR], Defs = [CCR], isPseudo = 1
 370: 
 371: 
 372: def : Pat<(i16 (anyext (i8 (MxSetCC_C MxCONDcs, CCR)))), (SETCS_C16d)>;
 373: def : Pat<(i32 (anyext (i8 (MxSetCC_C MxCONDcs, CCR)))), (SETCS_C32d)>;
 374: 
 375: def : Pat<(i16 (sext (i8 (MxSetCC_C MxCONDcs, CCR)))), (SETCS_C16d)>;
 376: def : Pat<(i32 (sext (i8 (MxSetCC_C MxCONDcs, CCR)))), (SETCS_C32d)>;
 377: 
 378: // We canonicalize 'scs' to "(and (subx reg,reg), 1)" on the hope that the and
 379: // will be eliminated and that the subx can be extended up to a wider type.  When
 380: // this happens, it is great.  However, if we are left with an 8-bit subx and an
 381: // and, we might as well just match it as a setb.
 382: def : Pat<(and (i8 (MxSetCC_C MxCONDcs, CCR)), 1), (SETd8cs)>;
 383: 
 384: // (add OP, SETB) -> (addx OP, (move 0))
 385: def : Pat<(add (and (i8 (MxSetCC_C MxCONDcs, CCR)), 1), MxDRD8:$op),
 386:           (ADDX8dd MxDRD8:$op, (MOV8di 0))>;
 387: def : Pat<(add (and (i32 (MxSetCC_C MxCONDcs, CCR)), 1), MxXRD32:$op),
 388:           (ADDX32dd MxDRD32:$op, (MOV32ri 0))>;
 389: 
 390: // (sub OP, SETB) -> (subx OP, (move 0))
 391: def : Pat<(sub MxDRD8:$op, (and (i8 (MxSetCC_C MxCONDcs, CCR)), 1)),
 392:           (SUBX8dd MxDRD8:$op, (MOV8di 0))>;
```
- **EN**: This block declares or refines TableGen records such as `SETCS_C16d`, `SETCS_C32d`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `SETCS_C16d`, `SETCS_C32d` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-420 / 第 393-420 行
```tablegen
 393: def : Pat<(sub MxXRD32:$op, (and (i32 (MxSetCC_C MxCONDcs, CCR)), 1)),
 394:           (SUBX32dd MxDRD32:$op, (MOV32ri 0))>;
 395: 
 396: // (sub OP, SETCC_CARRY) -> (addx OP, (move 0))
 397: def : Pat<(sub MxDRD8:$op, (i8 (MxSetCC_C MxCONDcs, CCR))),
 398:           (ADDX8dd MxDRD8:$op, (MOV8di 0))>;
 399: def : Pat<(sub MxXRD32:$op, (i32 (MxSetCC_C MxCONDcs, CCR))),
 400:           (ADDX32dd MxDRD32:$op, (MOV32ri 0))>;
 401: 
 402: //===------------===//
 403: // Trap / Breakpoint
 404: //===------------===//
 405: 
 406: let RenderMethod = "addImmOperands", ParserMethod = "parseImm" in {
 407:   def MxTrapImm : AsmOperandClass {
 408:     let Name = "MxTrapImm";
 409:     let PredicateMethod = "isTrapImm";
 410:   }
 411: 
 412:   def MxBkptImm : AsmOperandClass {
 413:     let Name = "MxBkptImm";
 414:     let PredicateMethod = "isBkptImm";
 415:   }
 416: }
 417: 
 418: let ParserMatchClass = MxTrapImm in
 419: def MxTrapimm : MxOp<i8,  MxSize8,  "i">;
 420: 
```
- **EN**: This block declares or refines TableGen records such as `MxTrapImm`, `MxBkptImm`, `MxTrapimm`.
- **CN**: 该代码块声明或细化了 `MxTrapImm`, `MxBkptImm`, `MxTrapimm` 等 TableGen 记录。

### Lines 421-438 / 第 421-438 行
```tablegen
 421: let ParserMatchClass = MxBkptImm in
 422: def MxBkptimm : MxOp<i8,  MxSize8,  "i">;
 423: 
 424: def TRAP : MxInst<(outs), (ins MxTrapimm:$vect), "trap\t$vect", []> {
 425:   let Inst = (descend 0b0100, 0b1110, 0b0100, (operand "$vect", 4));
 426: }
 427: 
 428: def TRAPV : MxInst<(outs), (ins), "trapv", []> {
 429:   let Inst = (descend 0b0100, 0b1110, 0b0111, 0b0110);
 430: }
 431: 
 432: def BKPT : MxInst<(outs), (ins MxBkptimm:$vect), "bkpt\t$vect", []> {
 433:   let Inst = (descend 0b0100, 0b1000, 0b0100, 0b1 , (operand "$vect", 3));
 434: }
 435: 
 436: def ILLEGAL : MxInst<(outs), (ins), "illegal", []> {
 437:   let Inst = (descend 0b0100, 0b1010, 0b1111, 0b1100);
 438: }
```
- **EN**: This block declares or refines TableGen records such as `MxBkptimm`, `TRAP`, `TRAPV`, `BKPT`, `ILLEGAL`.
- **CN**: 该代码块声明或细化了 `MxBkptimm`, `TRAP`, `TRAPV`, `BKPT`, `ILLEGAL` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
