# LanaiInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiInstrFormats.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines reusable TableGen instruction format classes and encoding fields.
- 目的（中文）: 定义可复用的 TableGen 指令格式类与编码字段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- LanaiInstrFormats.td - Lanai Instruction Formats ----*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: class InstLanai<dag outs, dag ins, string asmstr, list<dag> pattern>
  10:     : Instruction {
  11:   field bits<32> Inst;
  12:   let Size = 4;
  13: 
  14:   let Namespace = "Lanai";
  15:   let DecoderNamespace = "Lanai";
  16: 
  17:   bits<4> Opcode;
  18:   let Inst{31 - 28} = Opcode;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as InstLanai, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 InstLanai 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19: 
  20:   dag OutOperandList = outs;
  21:   dag InOperandList = ins;
  22:   let AsmString = asmstr;
  23:   let Pattern = pattern;
  24: }
  25: 
  26: //------------------------------------------------------------------------------
  27: // Register Immediate (RI)
  28: //------------------------------------------------------------------------------
  29: // Encoding:
  30: //           -----------------------------------------------------------------
  31: //           |0.A.A.A| . . . . | . . . . |F.H| . . . . . . . . . . . . . . . |
  32: //           -----------------------------------------------------------------
  33: //            opcode     Rd        Rs1                constant (16)
  34: //
  35: // Action:
  36: //           Rd <- Rs1 op constant
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 37-54

```tablegen
  37: //
  38: // Except for shift instructions, `H' determines whether the constant
  39: // is in the high (1) or low (0) word.  The other halfword is 0x0000,
  40: // except for the `AND' instruction (`AAA' = 100), for which the other
  41: // halfword is 0xFFFF, and shifts (`AAA' = 111), for which the constant is
  42: // sign extended.
  43: //
  44: // `F' determines whether the instruction modifies (1) or does not
  45: // modify (0) the program flags.
  46: //
  47: // `AAA' specifies the operation: `add' (000), `addc' (001), `sub'
  48: // (010), `subb' (011), `and' (100), `or' (101), `xor' (110), or `shift'
  49: // (111).  For the shift, `H' specifies a logical (0) or arithmetic (1)
  50: // shift.  The amount and direction of the shift are determined by the
  51: // sign extended constant interpreted as a two's complement number.  The
  52: // shift operation is defined only for the range of:
  53: //      31 ... 0 -1 ... -31
  54: //      \      / \        /
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 55-72

```tablegen
  55: //        left     right
  56: //        shift    shift
  57: //
  58: // If and only if the `F' bit is 1, RI instructions modify the
  59: // condition bits, `Z' (Zero), `N' (Negative), `V' (oVerflow), and `C'
  60: // (Carry), according to the result.  If the flags are updated, they are
  61: // updated as follows:
  62: // `Z'
  63: //      is set if the result is zero and cleared otherwise.
  64: //
  65: // `N'
  66: //      is set to the most significant bit of the result.
  67: //
  68: // `V'
  69: //      For arithmetic instructions (`add', `addc', `sub', `subb') `V' is
  70: //      set if the sign (most significant) bits of the input operands are
  71: //      the same but different from the sign bit of the result and cleared
  72: //      otherwise.  For other RI instructions, `V' is cleared.
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 73-90

```tablegen
  73: //
  74: // `C'
  75: //      For arithmetic instructions, `C' is set/cleared if there is/is_not
  76: //      a carry generated out of the most significant when performing the
  77: //      twos-complement addition (`sub(a,b) == a + ~b + 1', `subb(a,b) ==
  78: //      a + ~b + `C'').  For left shifts, `C' is set to the least
  79: //      significant bit discarded by the shift operation.  For all other
  80: //      operations, `C' is cleared.
  81: //
  82: // A Jump is accomplished by `Rd' being `pc', and it has one shadow.
  83: //
  84: // The all-0s word is the instruction `R0 <- R0 + 0', which is a no-op.
  85: class InstRI<bits<3> op, dag outs, dag ins, string asmstr,
  86:              list<dag> pattern>
  87:     : InstLanai<outs, ins, asmstr, pattern>, Sched<[WriteALU]> {
  88:   let Itinerary = IIC_ALU;
  89:   bits<5> Rd;
  90:   bits<5> Rs1;
```

- EN: This range defines declarative TableGen records such as InstRI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstRI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91:   bit F;
  92:   bit H;
  93:   bits<16> imm16;
  94: 
  95:   let Opcode{3} = 0;
  96:   let Opcode{2 - 0} = op;
  97:   let Inst{27 - 23} = Rd;
  98:   let Inst{22 - 18} = Rs1;
  99:   let Inst{17} = F;
 100:   let Inst{16} = H;
 101:   let Inst{15 - 0} = imm16;
 102: }
 103: 
 104: //------------------------------------------------------------------------------
 105: // Register Register (RR)
 106: //------------------------------------------------------------------------------
 107: // Encoding:
 108: //           -----------------------------------------------------------------
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109: //           |1.1.0.0| . . . . | . . . . |F.I| . . . . |B.B.B|J.J.J.J.J|D.D.D|
 110: //           -----------------------------------------------------------------
 111: //            opcode     Rd        Rs1           Rs2   \       operation     /
 112: //
 113: // Action:
 114: //           `Rd <- Rs1 op Rs2' iff condition DDDI is true.
 115: //
 116: // `DDDI' is as described for the BR instruction.
 117: //
 118: // `F' determines whether the instruction modifies (1) or does not
 119: // modify (0) the program flags.
 120: //
 121: // `BBB' determines the operation: `add' (000), `addc' (001), `sub'
 122: // (010), `subb' (011), `and' (100), `or' (101), `xor' (110), or "special"
 123: // (111).  The `JJJJJ' field is irrelevant except for special.
 124: //
 125: // `JJJJJ' determines which special operation is performed.  `10---'
 126: // is a logical shift, and `11---' is an arithmetic shift, and ‘00000` is
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 127-144

```tablegen
 127: // the SELECT operation.  The amount and direction of the shift are
 128: // determined by the contents of `Rs2' interpreted as a two's complement
 129: // number (in the same way as shifts in the Register-Immediate
 130: // instructions in *Note RI::).  For the SELECT operation, Rd gets Rs1 if
 131: // condition DDDI is true, Rs2 otherwise. All other `JJJJJ' combinations
 132: // are reserved for instructions that may be defined in the future.
 133: //
 134: // If the `F' bit is 1, RR instructions modify the condition bits, `Z'
 135: // (Zero), `N' (Negative), `V' (oVerflow), and `C' (Carry), according to
 136: // the result.  All RR instructions modify the `Z', `N', and `V' flags.
 137: // Except for arithmetic instructions (`add', `addc', `sub', `subb'), `V'
 138: // is cleared.  Only arithmetic instructions and shifts modify `C'. Right
 139: // shifts clear C.
 140: //
 141: // DDDI is as described in the table for the BR instruction and only used for
 142: // the select instruction.
 143: //
 144: // A Jump is accomplished by `Rd' being `pc', and it has one shadow.
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 145-162

```tablegen
 145: class InstRR<bits<3> op, dag outs, dag ins, string asmstr,
 146:              list<dag> pattern>
 147:     : InstLanai<outs, ins, asmstr, pattern>, Sched<[WriteALU]> {
 148:   let Itinerary = IIC_ALU;
 149:   bits<5> Rd;
 150:   bits<5> Rs1;
 151:   bits<5> Rs2;
 152:   bit F;
 153:   bits<4> DDDI;
 154:   bits<5> JJJJJ;
 155: 
 156:   let Opcode = 0b1100;
 157:   let Inst{27 - 23} = Rd;
 158:   let Inst{22 - 18} = Rs1;
 159:   let Inst{17} = F;
 160:   let Inst{16} = DDDI{0};
 161:   let Inst{15 - 11} = Rs2;
 162:   let Inst{10 - 8} = op;
```

- EN: This range defines declarative TableGen records such as InstRR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstRR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:   let Inst{7 - 3} = JJJJJ;
 164:   let Inst{2 - 0} = DDDI{3 - 1};
 165: }
 166: 
 167: //------------------------------------------------------------------------------
 168: // Register Memory (RM)
 169: //------------------------------------------------------------------------------
 170: // Encoding:
 171: //          -----------------------------------------------------------------
 172: //          |1.0.0.S| . . . . | . . . . |P.Q| . . . . . . . . . . . . . . . |
 173: //          -----------------------------------------------------------------
 174: //           opcode     Rd        Rs1                 constant (16)
 175: //
 176: // Action:
 177: //        Rd <- Memory(ea)      (Load)    see below for the
 178: //        Memory(ea) <- Rd      (Store)   definition of ea.
 179: //
 180: // `S' determines whether the instruction is a Load (0) or a Store (1).
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181: // Loads appear in Rd one cycle after this instruction executes.  If the
 182: // following instruction reads Rd, that instruction will be delayed by 1
 183: // clock cycle.
 184: //
 185: //   PQ      operation
 186: //   --      ------------------------------------------
 187: //   00      ea = Rs1
 188: //   01      ea = Rs1,             Rs1 <- Rs1 + constant
 189: //   10      ea = Rs1 + constant
 190: //   11      ea = Rs1 + constant,  Rs1 <- Rs1 + constant
 191: //
 192: // The constant is sign-extended for this instruction.
 193: //
 194: // A Jump is accomplished by `Rd' being `pc', and it has *two* delay slots.
 195: class InstRM<bit S, dag outs, dag ins, string asmstr, list<dag> pattern>
 196:     : InstLanai<outs, ins, asmstr, pattern> {
 197:   bits<5> Rd;
 198:   bits<5> Rs1;
```

- EN: This range defines declarative TableGen records such as InstRM, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 InstRM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 199-216

```tablegen
 199:   bits<1> P;
 200:   bits<1> Q;
 201:   bits<16> imm16;
 202:   // Dummy variables to allow multiclass definition of RM and RRM
 203:   bits<2> YL;
 204:   bit E;
 205: 
 206:   let Opcode{3 - 1} = 0b100;
 207:   let Opcode{0} = S;
 208:   let Inst{27 - 23} = Rd;
 209:   let Inst{22 - 18} = Rs1;
 210:   let Inst{17} = P;
 211:   let Inst{16} = Q;
 212:   let Inst{15 - 0} = imm16;
 213: 
 214:   let PostEncoderMethod = "adjustPqBitsRmAndRrm";
 215: }
 216: 
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217: //------------------------------------------------------------------------------
 218: // Register Register Memory (RRM)
 219: //------------------------------------------------------------------------------
 220: // Encoding:
 221: //           -----------------------------------------------------------------
 222: //           |1.0.1.S| . . . . | . . . . |P.Q| . . . . |B.B.B|J.J.J.J.J|Y.L.E|
 223: //           -----------------------------------------------------------------
 224: //            opcode     Rd        Rs1           Rs2   \       operation     /
 225: //
 226: // Action:
 227: //           Rd <- Memory(ea)      (Load)    see below for the
 228: //           Memory(ea) <- Rd      (Store)   definition of ea.
 229: //
 230: // The RRM instruction is identical to the RM (*note RM::.) instruction
 231: // except that:
 232: //
 233: // 1. `Rs1 + constant' is replaced with `Rs1 op Rs2', where `op' is
 234: //    determined in the same way as in the RR instruction (*note RR::.)
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 235-252

```tablegen
 235: //    and
 236: //
 237: // 2. part-word memory accesses are allowed as specified below.
 238: //
 239: //    If `BBB' != 111 (i.e.: For all but shift operations):
 240: //        If `YLE' = 01- => fuLl-word memory access
 241: //        If `YLE' = 00- => half-word memory access
 242: //        If `YLE' = 10- => bYte memory access
 243: //        If `YLE' = --1 => loads are zEro extended
 244: //        If `YLE' = --0 => loads are sign extended
 245: //
 246: //    If `BBB' = 111 (For shift operations):
 247: //        fullword memory access are performed.
 248: //
 249: // All part-word loads write the least significant part of the
 250: // destination register with the higher-order bits zero- or sign-extended.
 251: // All part-word stores store the least significant part-word of the
 252: // source register in the destination memory location.
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 253-270

```tablegen
 253: //
 254: // A Jump is accomplished by `Rd' being `pc', and it has *two* delay slots.
 255: class InstRRM<bit S, dag outs, dag ins, string asmstr,
 256:               list<dag> pattern>
 257:     : InstLanai<outs, ins, asmstr, pattern> {
 258:   bits<5> Rd;
 259:   bits<5> Rs1;
 260:   bits<5> Rs2;
 261:   bits<1> P;
 262:   bits<1> Q;
 263:   bits<3> BBB;
 264:   bits<5> JJJJJ;
 265:   bits<2> YL;
 266:   bit E;
 267: 
 268:   let Opcode{3 - 1} = 0b101;
 269:   let Opcode{0} = S;
 270:   let Inst{27 - 23} = Rd;
```

- EN: This range defines declarative TableGen records such as InstRRM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstRRM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271:   let Inst{22 - 18} = Rs1;
 272:   let Inst{17} = P;
 273:   let Inst{16} = Q;
 274:   let Inst{15 - 11} = Rs2;
 275:   let Inst{10 - 8} = BBB;
 276:   let Inst{7 - 3} = JJJJJ;
 277:   let Inst{2 - 1} = YL;
 278:   let Inst{0} = E;
 279: 
 280:   let PostEncoderMethod = "adjustPqBitsRmAndRrm";
 281: }
 282: 
 283: //------------------------------------------------------------------------------
 284: // Conditional Branch (BR)
 285: //------------------------------------------------------------------------------
 286: // Encoding:
 287: //           -----------------------------------------------------------------
 288: //           |1.1.1.0|D.D.D| . . . . . . . . . . . . . . . . . . . . . . |0.I|
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289: //           -----------------------------------------------------------------
 290: //            opcode condition                   constant (23)
 291: //
 292: // Action:
 293: //            if (condition) { `pc' <- 4*(zero-extended constant) }
 294: //
 295: // The BR instruction is an absolute branch.
 296: // The constant is scaled as shown by its position in the instruction word such
 297: // that it specifies word-aligned addresses in the range [0,2^25-4]
 298: //
 299: // The `DDDI' field selects the condition that causes the branch to be taken.
 300: // (the `I' (Invert sense) bit inverts the sense of the condition):
 301: //
 302: //   DDDI  logical function                        [code, used for...]
 303: //   ----  --------------------------------------  ------------------------
 304: //   0000  1                                       [T, true]
 305: //   0001  0                                       [F, false]
 306: //   0010  C AND Z'                                [HI, high]
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 307-324

```tablegen
 307: //   0011  C' OR Z                                 [LS, low or same]
 308: //   0100  C'                                      [CC, carry cleared]
 309: //   0101  C                                       [CS, carry set]
 310: //   0110  Z'                                      [NE, not equal]
 311: //   0111  Z                                       [EQ, equal]
 312: //   1000  V'                                      [VC, oVerflow cleared]
 313: //   1001  V                                       [VS, oVerflow set]
 314: //   1010  N'                                      [PL, plus]
 315: //   1011  N                                       [MI, minus]
 316: //   1100  (N AND V) OR (N' AND V')                [GE, greater than or equal]
 317: //   1101  (N AND V') OR (N' AND V)                [LT, less than]
 318: //   1110  (N AND V AND Z') OR (N' AND V' AND Z')  [GT, greater than]
 319: //   1111  (Z) OR (N AND V') OR (N' AND V)         [LE, less than or equal]
 320: //
 321: // If the branch is not taken, the BR instruction is a no-op.  If the branch is
 322: // taken, the processor starts executing instructions at the branch target
 323: // address *after* the processor has executed one more instruction.  That is,
 324: // the branch has one “branch delay slot”.  Be very careful if you find yourself
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 325-342

```tablegen
 325: // wanting to put a branch in a branch delays slot!
 326: class InstBR<dag outs, dag ins, string asmstr, list<dag> pattern>
 327:     : InstLanai<outs, ins, asmstr, pattern> {
 328:   let Itinerary = IIC_ALU;
 329:   bits<25> addr;
 330:   bits<4> DDDI;
 331: 
 332:   let Opcode = 0b1110;
 333:   let Inst{27 - 25} = DDDI{3 - 1};
 334:   let Inst{24 - 0} = addr;
 335:   // These instructions overwrite the last two address bits (which are assumed
 336:   // and ensured to be 0).
 337:   let Inst{1} = 0;
 338:   let Inst{0} = DDDI{0};
 339: }
 340: 
 341: //------------------------------------------------------------------------------
 342: // Conditional Branch Relative (BRR)
```

- EN: This range defines declarative TableGen records such as InstBR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstBR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 343-360

```tablegen
 343: //------------------------------------------------------------------------------
 344: // Encoding:
 345: //           -----------------------------------------------------------------
 346: //           |1.1.1.0|D.D.D|1|-| . . . . |-.-| . . . . . . . . . . . . . |1.I|
 347: //           -----------------------------------------------------------------
 348: //            opcode condition     Rs1           constant (14)
 349: // Action:
 350: //           if (condition) { ‘pc’ <- Rs1 + 4*sign-extended constant) }
 351: //
 352: // BRR behaves like BR, except the branch target address is a 16-bit PC relative
 353: // offset.
 354: class InstBRR<dag outs, dag ins, string asmstr, list<dag> pattern>
 355:     : InstLanai<outs, ins, asmstr, pattern> {
 356:   bits<4> DDDI;
 357:   bits<5> Rs1;
 358:   bits<16> imm16;
 359: 
 360:   let Opcode = 0b1110;
```

- EN: This range defines declarative TableGen records such as InstBRR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstBRR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361:   let Inst{27 - 25} = DDDI{3 - 1};
 362:   let Inst{24} = 1;
 363:   let Inst{22 - 18} = Rs1;
 364:   let Inst{17 - 16} = 0;
 365:   let Inst{15 - 0} = imm16;
 366:   // Overwrite last two bits which have to be zero
 367:   let Inst{1} = 1;
 368:   let Inst{0} = DDDI{0};
 369: 
 370:   // Set don't cares to zero
 371:   let Inst{23} = 0;
 372: }
 373: 
 374: //------------------------------------------------------------------------------
 375: // Conditional Set (SCC)
 376: //------------------------------------------------------------------------------
 377: // Encoding:
 378: //           -----------------------------------------------------------------
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 379-396

```tablegen
 379: //           |1.1.1.0|D.D.D|0.-| . . . . |-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-|1.I|
 380: //           -----------------------------------------------------------------
 381: //            opcode condition     Rs1
 382: //
 383: // Action:
 384: //       Rs1 <- logical function result
 385: //
 386: // SCC sets dst_reg to the boolean result of computing the logical function
 387: // specified by DDDI, as described in the table for the BR instruction.
 388: class InstSCC<dag outs, dag ins, string asmstr,
 389:               list<dag> pattern>
 390:     : InstLanai<outs, ins, asmstr, pattern> {
 391:   let Itinerary = IIC_ALU;
 392:   bits<5> Rs1; // dst_reg in documentation
 393:   bits<4> DDDI;
 394: 
 395:   let Opcode = 0b1110;
 396:   let Inst{27 - 25} = DDDI{3 - 1};
```

- EN: This range defines declarative TableGen records such as InstSCC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstSCC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397:   let Inst{24} = 0;
 398:   let Inst{22 - 18} = Rs1;
 399:   let Inst{1} = 1;
 400:   let Inst{0} = DDDI{0};
 401: 
 402:   // Set don't cares to zero
 403:   let Inst{23} = 0;
 404:   let Inst{17 - 2} = 0;
 405: }
 406: 
 407: //------------------------------------------------------------------------------
 408: // Special Load/Store (SLS)
 409: //------------------------------------------------------------------------------
 410: //
 411: // Encoding:
 412: //           -----------------------------------------------------------------
 413: //           |1.1.1.1| . . . . | . . . . |0.S| . . . . . . . . . . . . . . . |
 414: //           -----------------------------------------------------------------
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415: //            opcode     Rd    addr 5msb's            address 16 lsb's
 416: //
 417: // Action:
 418: //           If S = 0 (LOAD):   Rd <- Memory(address);
 419: //           If S = 1 (STORE):  Memory(address) <- Rd
 420: //
 421: // The timing is the same as for RM (*note RM::.) and RRM (*note
 422: // RRM::.) instructions.  The two low-order bits of the 21-bit address are
 423: // ignored.  The address is zero extended.  Fullword memory accesses are
 424: // performed.
 425: class InstSLS<bit S, dag outs, dag ins, string asmstr, list<dag> pattern>
 426:     : InstLanai<outs, ins, asmstr, pattern> {
 427:   bits<5> Rd;
 428:   bits<5> msb;
 429:   bits<16> lsb;
 430: 
 431:   let Opcode = 0b1111;
 432:   let Inst{27 - 23} = Rd;
```

- EN: This range defines declarative TableGen records such as InstSLS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstSLS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 433-450

```tablegen
 433:   let Inst{22 - 18} = msb;
 434:   let Inst{17} = 0;
 435:   let Inst{16} = S;
 436:   let Inst{15 - 0} = lsb;
 437: }
 438: 
 439: //------------------------------------------------------------------------------
 440: // Special Load Immediate (SLI)
 441: //------------------------------------------------------------------------------
 442: // Encoding:
 443: //           -----------------------------------------------------------------
 444: //           |1.1.1.1| . . . . | . . . . |1.0| . . . . . . . . . . . . . . . |
 445: //           -----------------------------------------------------------------
 446: //            opcode     Rd    const 5msb's          constant 16 lsb's
 447: //
 448: // Action:
 449: //           Rd <- constant
 450: //
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451: // The 21-bit constant is zero-extended.  The timing is the same as the
 452: // RM instruction (*note RM::.).
 453: class InstSLI<dag outs, dag ins, string asmstr, list<dag> pattern>
 454:     : InstLanai<outs, ins, asmstr, pattern> {
 455:   bits<5> Rd;
 456:   bits<5> msb;
 457:   bits<16> lsb;
 458: 
 459:   let Opcode = 0b1111;
 460:   let Inst{27 - 23} = Rd;
 461:   let Inst{22 - 18} = msb;
 462:   let Inst{17} = 1;
 463:   let Inst{16} = 0;
 464:   let Inst{15 - 0} = lsb;
 465: }
 466: 
 467: //------------------------------------------------------------------------------
 468: // Special Part-Word Load/Store (SPLS)
```

- EN: This range defines declarative TableGen records such as InstSLI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 InstSLI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469: //------------------------------------------------------------------------------
 470: // Encoding:
 471: //        -----------------------------------------------------------------
 472: //        |1.1.1.1| . . . . | . . . . |1.1.0.Y.S.E.P.Q| . . . . . . . . . |
 473: //        -----------------------------------------------------------------
 474: //         opcode     Rd        Rs1                       constant (10)
 475: //
 476: // Action:
 477: //        If `YS' = 11  (bYte     Store):
 478: //             Memory(ea) <- (least significant byte of Rr)
 479: //        If `YS' = 01  (halfword Store):
 480: //             Memory(ea) <- (least significant half-word of Rr)
 481: //        If `YS' = 10  (bYte     load):  Rr <- Memory(ea)
 482: //        If `YS' = 00  (halfword load):  Rr <- Memory(ea)
 483: //             [Note: here ea is determined as in the RM instruction. ]
 484: //        If `SE' = 01 then the value is zEro extended
 485: //             before being loaded into Rd.
 486: //        If `SE' = 00 then the value is sign extended
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 487-504

```tablegen
 487: //             before being loaded into Rd.
 488: //
 489: // `P' and `Q' are used to determine `ea' as in the RM instruction. The
 490: // constant is sign extended.  The timing is the same as the RM and RRM
 491: // instructions.  *Note RM:: and *Note RRM::.
 492: //
 493: // All part-word loads write the part-word into the least significant
 494: // part of the destination register, with the higher-order bits zero- or
 495: // sign-extended.  All part-word stores store the least significant
 496: // part-word of the source register into the destination memory location.
 497: class InstSPLS<dag outs, dag ins, string asmstr,
 498:                list<dag> pattern>
 499:     : InstLanai<outs, ins, asmstr, pattern> {
 500:   bits<5> Rd;
 501:   bits<5> Rs1;
 502:   bits<5> msb;
 503:   bit Y;
 504:   bit S;
```

- EN: This range defines declarative TableGen records such as InstSPLS, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 InstSPLS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 505-522

```tablegen
 505:   bit E;
 506:   bits<1> P;
 507:   bits<1> Q;
 508:   bits<10> imm10;
 509: 
 510:   let Opcode = 0b1111;
 511:   let Inst{27 - 23} = Rd;
 512:   let Inst{22 - 18} = Rs1;
 513:   let Inst{17 - 15} = 0b110;
 514:   let Inst{14} = Y;
 515:   let Inst{13} = S;
 516:   let Inst{12} = E;
 517:   let Inst{11} = P;
 518:   let Inst{10} = Q;
 519:   let Inst{9 - 0} = imm10;
 520: 
 521:   let PostEncoderMethod = "adjustPqBitsSpls";
 522: }
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 523-540

```tablegen
 523: 
 524: //------------------------------------------------------------------------------
 525: // Special instructions (popc, leadz, trailz)
 526: //------------------------------------------------------------------------------
 527: // Encoding:
 528: //         -----------------------------------------------------------------
 529: //         |1.1.0.1|    Rd   |   Rs1   |F.-| . . . . | . . | . . . . | OP  |
 530: //         -----------------------------------------------------------------
 531: //          opcode      Rd       Rs1
 532: // Action:
 533: //         Rd <- Perform action encoded in OP on Rs1
 534: //   OP is one of:
 535: //      0b001 POPC   Population count;
 536: //      0b010 LEADZ  Count number of leading zeros;
 537: //      0b011 TRAILZ Count number of trailing zeros;
 538: class InstSpecial<bits<3> op, dag outs, dag ins, string asmstr,
 539:                   list<dag> pattern> : InstLanai<outs, ins, asmstr,
 540:                   pattern>, Sched<[WriteALU]> {
```

- EN: This range defines declarative TableGen records such as InstSpecial, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 InstSpecial 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 541-558

```tablegen
 541:   let Itinerary = IIC_ALU;
 542:   bit F;
 543:   bits<5> Rd;
 544:   bits<5> Rs1;
 545: 
 546:   let Opcode = 0b1101;
 547:   let Inst{27 - 23} = Rd;
 548:   let Inst{22 - 18} = Rs1;
 549:   let Inst{17} = F;
 550:   let Inst{16 - 3} = 0;
 551:   let Inst{2 - 0} = op;
 552: }
 553: 
 554: // Pseudo instructions
 555: class Pseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
 556:     : InstLanai<outs, ins, asmstr, pattern> {
 557:   let Inst{15 - 0} = 0;
 558:   let isPseudo = 1;
```

- EN: This range defines declarative TableGen records such as Pseudo, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 Pseudo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 559-559

```tablegen
 559: }
```

- EN: This range continues the implementation of the backend component described by LanaiInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Instruction format fields / 指令格式字段
- Reusable TableGen classes / 可复用 TableGen 类
- Pseudo-instruction handling / 伪指令处理
- Scheduling model / 调度模型

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
