# M68kInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 M68k 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //===-- M68kInstrFormats.td - M68k Instruction Formats -----*- tablegen -*-===//
   2: //                     The LLVM Compiler Infrastructure
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //===----------------------------------------------------------------------===//
   7: ///
   8: /// \file
   9: /// This file contains M68k instruction formats.
  10: ///
  11: /// Since M68k has quite a lot memory addressing modes there are more
  12: /// instruction prefixes than just i, r and m:
  13: /// TSF  Since     Form                     Letter  Description
  14: ///  00   M68000    Dn or An                 r       any register
  15: ///  01   M68000    Dn                       d       data register direct
  16: ///  02   M68000    An                       a       address register direct
  17: ///  03   M68000    (An)                     j       address register indirect
  18: ///  04   M68000    (An)+                    o       address register indirect with postincrement
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: ///  05   M68000    -(An)                    e       address register indirect with predecrement
  20: ///  06   M68000    (d16,An)                 p       address register indirect with displacement
  21: ///  10   M68000    (d8,An,Xn.L)             f       address register indirect with index and scale = 1
  22: ///  07   M68000    (d8,An,Xn.W)             F       address register indirect with index and scale = 1
  23: ///  12   M68020    (d8,An,Xn.L,SCALE)       g       address register indirect with index
  24: ///  11   M68020    (d8,An,Xn.W,SCALE)       G       address register indirect with index
  25: ///  14   M68020    ([bd,An],Xn.L,SCALE,od)  u       memory indirect postindexed mode
  26: ///  13   M68020    ([bd,An],Xn.W,SCALE,od)  U       memory indirect postindexed mode
  27: ///  16   M68020    ([bd,An,Xn.L,SCALE],od)  v       memory indirect preindexed mode
  28: ///  15   M68020    ([bd,An,Xn.W,SCALE],od)  V       memory indirect preindexed mode
  29: ///  20   M68000    abs.L                    b       absolute long address
  30: ///  17   M68000    abs.W                    B       absolute short address
  31: ///  21   M68000    (d16,PC)                 q       program counter with displacement
  32: ///  23   M68000    (d8,PC,Xn.L)             k       program counter with index and scale = 1
  33: ///  22   M68000    (d8,PC,Xn.W)             K       program counter with index and scale = 1
  34: ///  25   M68020    (d8,PC,Xn.L,SCALE)       l       program counter with index
  35: ///  24   M68020    (d8,PC,Xn.W,SCALE)       L       program counter with index
  36: ///  27   M68020    ([bd,PC],Xn.L,SCALE,od)  x       program counter memory indirect postindexed mode
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 37-54 / 第 37-54 行
```tablegen
  37: ///  26   M68020    ([bd,PC],Xn.W,SCALE,od)  X       program counter memory indirect postindexed mode
  38: ///  31   M68020    ([bd,PC,Xn.L,SCALE],od)  y       program counter memory indirect preindexed mode
  39: ///  30   M68020    ([bd,PC,Xn.W,SCALE],od)  Y       program counter memory indirect preindexed mode
  40: ///  32   M68000    #immediate               i       immediate data
  41: ///
  42: /// NOTE that long form is always lowercase, word variants are capitalized
  43: ///
  44: /// Operand can be qualified with size where appropriate to force a particular
  45: /// instruction encoding, e.g.:
  46: ///    (i8,An,Xn.W)             f8      1 extension word
  47: ///    (i16,An,Xn.W)            f16     2 extension words
  48: ///    (i32,An,Xn.W)            f32     3 extension words
  49: ///
  50: /// Form without size qualifier will adapt to operand size automatically, e.g.:
  51: ///    (i,An,Xn.W)              f       1, 2 or 3 extension words
  52: ///
  53: /// Some forms already imply a particular size of their operands, e.g.:
  54: ///    (i,An)                   p       1 extension word and i is 16bit
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: ///
  56: /// Operand order follows x86 Intel order(destination before source), e.g.:
  57: ///    MOV8df                   MOVE (4,A0,D0), D1
  58: ///
  59: /// Number after instruction mnemonics determines the size of the data
  60: ///
  61: //===----------------------------------------------------------------------===//
  62: 
  63: /// ??? Is it possible to use this stuff for disassembling?
  64: /// NOTE 1: In case of conditional beads(DA, DAReg), cond part is able to
  65: /// consume any bit, though a more general instructions must be chosen, e.g.
  66: /// d -> r, a -> r
  67: 
  68: //===----------------------------------------------------------------------===//
  69: // Encoding primitives
  70: //===----------------------------------------------------------------------===//
  71: 
  72: class MxEncMemOp {
```
- **EN**: This block declares or refines TableGen records such as `MxEncMemOp`.
- **CN**: 该代码块声明或细化了 `MxEncMemOp` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:   dag EA = (ascend);
  74:   dag Supplement = (ascend);
  75: }
  76: 
  77: class MxEncBriefExt<string reg_opnd, string disp_opnd,
  78:                     bit size_w_l = false, int scale = 1,
  79:                     string disp_encoder = ""> {
  80:   dag Value = (descend
  81:     // D/A + REGISTER
  82:     (operand "$"#reg_opnd, 4),
  83:     // W/L
  84:     size_w_l,
  85:     // SCALE
  86:     !cond(
  87:       !eq(scale, 1) : 0b00,
  88:       !eq(scale, 2) : 0b01,
  89:       !eq(scale, 4) : 0b10,
  90:       !eq(scale, 8) : 0b11
```
- **EN**: This block declares or refines TableGen records such as `MxEncBriefExt`.
- **CN**: 该代码块声明或细化了 `MxEncBriefExt` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91:     ),
  92:     0b0,
  93:     // Displacement
  94:     (operand "$"#disp_opnd, 8, (encoder disp_encoder))
  95:   );
  96: }
  97: 
  98: class MxEncAddrMode_d<string reg_opnd> : MxEncMemOp {
  99:   let EA = (descend /*MODE*/0b000,
 100:                     /*REGISTER*/(operand "$"#reg_opnd, 3));
 101: }
 102: 
 103: class MxEncAddrMode_a<string reg_opnd> : MxEncMemOp {
 104:   let EA = (descend /*MODE*/0b001,
 105:                     /*REGISTER*/(operand "$"#reg_opnd, 3));
 106: }
 107: 
 108: class MxEncAddrMode_r<string reg_opnd> : MxEncMemOp {
```
- **EN**: This block declares or refines TableGen records such as `MxEncAddrMode_d`, `MxEncAddrMode_a`, `MxEncAddrMode_r`.
- **CN**: 该代码块声明或细化了 `MxEncAddrMode_d`, `MxEncAddrMode_a`, `MxEncAddrMode_r` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109:   let EA = (descend /*MODE without the last bit*/0b00,
 110:                     /*REGISTER with D/A bit*/(operand "$"#reg_opnd, 4));
 111: }
 112: 
 113: class MxEncAddrMode_k<string opnd_name> : MxEncMemOp {
 114:   let EA = (descend /*MODE*/0b111,
 115:                     /*REGISTER*/0b011);
 116: 
 117:   let Supplement = MxEncBriefExt<opnd_name#".index", opnd_name#".disp",
 118:                                  /*W/L*/true, /*SCALE*/1,
 119:                                  "encodePCRelImm<8>">.Value;
 120: }
 121: 
 122: class MxEncAddrMode_q<string opnd_name> : MxEncMemOp {
 123:   let EA = (descend /*MODE*/0b111,
 124:                      /*REGISTER*/0b010);
 125: 
 126:   // 16-bit Displacement
```
- **EN**: This block declares or refines TableGen records such as `MxEncAddrMode_k`, `MxEncAddrMode_q`.
- **CN**: 该代码块声明或细化了 `MxEncAddrMode_k`, `MxEncAddrMode_q` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127:   let Supplement = (operand "$"#opnd_name, 16,
 128:                             (encoder "encodePCRelImm<16>"));
 129: }
 130: 
 131: class MxEncAddrMode_p<string opnd_name> : MxEncMemOp {
 132:   let EA = (descend /*MODE*/0b101,
 133:                      /*REGISTER*/(operand "$"#opnd_name#".reg", 3));
 134: 
 135:   // 16-bit Displacement
 136:   let Supplement = (operand "$"#opnd_name#".disp", 16,
 137:                             (encoder "encodeRelocImm<16>"));
 138: }
 139: 
 140: class MxEncAddrMode_f<string opnd_name> : MxEncMemOp {
 141:   let EA = (descend /*MODE*/0b110,
 142:                      /*REGISTER*/(operand "$"#opnd_name#".reg", 3));
 143: 
 144:   let Supplement = MxEncBriefExt<opnd_name#".index", opnd_name#".disp",
```
- **EN**: This block declares or refines TableGen records such as `MxEncAddrMode_p`, `MxEncAddrMode_f`.
- **CN**: 该代码块声明或细化了 `MxEncAddrMode_p`, `MxEncAddrMode_f` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145:                                  /*W/L*/true, /*SCALE*/1,
 146:                                  "encodeRelocImm<8>">.Value;
 147: }
 148: 
 149: class MxEncAddrMode_j<string reg_opnd> : MxEncMemOp {
 150:   let EA = (descend /*MODE*/0b010,
 151:                      /*REGISTER*/(operand "$"#reg_opnd, 3));
 152: }
 153: 
 154: class MxEncAddrMode_i<string opnd_name, int size> : MxEncMemOp {
 155:   let EA = (descend /*MODE*/0b111,
 156:                      /*REGISTER*/0b100);
 157: 
 158:   // Immediate
 159:   let Supplement =
 160:     !cond(
 161:       !eq(size, 8)  : (descend 0b00000000, (operand "$"#opnd_name, 8,
 162: 	                   (encoder "encodeRelocImm<8>"))),
```
- **EN**: This block declares or refines TableGen records such as `MxEncAddrMode_j`, `MxEncAddrMode_i`.
- **CN**: 该代码块声明或细化了 `MxEncAddrMode_j`, `MxEncAddrMode_i` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163:       !eq(size, 16) : (operand "$"#opnd_name, 16,
 164:                            (encoder "encodeRelocImm<16>")),
 165:       !eq(size, 32) : (operand "$"#opnd_name, 32,
 166:                            (encoder "encodeRelocImm<32>"),
 167:                            (decoder "DecodeImm32"))
 168:     );
 169: }
 170: 
 171: // abs.W -> size_w_l = false
 172: // abs.L -> size_w_l = true
 173: class MxEncAddrMode_abs<string opnd_name, bit size_w_l = false> : MxEncMemOp {
 174:   let EA = (descend /*MODE*/0b111,
 175:                     // Wrap the REGISTER part in another dag to make sure
 176:                     // the dag assigned to EA only has two arguments. Such
 177:                     // that it's easier for MOV instructions to reverse
 178:                     // on its destination part.
 179:                     /*REGISTER*/(descend 0b00, size_w_l));
 180: 
```
- **EN**: This block declares or refines TableGen records such as `MxEncAddrMode_abs`.
- **CN**: 该代码块声明或细化了 `MxEncAddrMode_abs` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181:   // Absolute address
 182:   let Supplement = !if(size_w_l,
 183:     // abs.L
 184:     (operand "$"#opnd_name, 32, (encoder "encodeRelocImm<32>"),
 185:                                 (decoder "DecodeImm32")),
 186:     // abs.W
 187:     (operand "$"#opnd_name, 16, (encoder "encodeRelocImm<16>"))
 188:   );
 189: }
 190: 
 191: class MxEncAddrMode_o<string reg_opnd> : MxEncMemOp {
 192:   let EA = (descend /*MODE*/0b011,
 193:                     /*REGISTER*/(operand "$"#reg_opnd, 3));
 194: }
 195: 
 196: class MxEncAddrMode_e<string reg_opnd> : MxEncMemOp {
 197:   let EA = (descend /*MODE*/0b100,
 198:                     /*REGISTER*/(operand "$"#reg_opnd, 3));
```
- **EN**: This block declares or refines TableGen records such as `MxEncAddrMode_o`, `MxEncAddrMode_e`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `MxEncAddrMode_o`, `MxEncAddrMode_e` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: }
 200: 
 201: class MxEncSize<bits<2> value> {
 202:   bits<2> Value = value;
 203: }
 204: def MxEncSize8  : MxEncSize<0b00>;
 205: def MxEncSize16 : MxEncSize<0b01>;
 206: def MxEncSize32 : MxEncSize<0b10>;
 207: def MxEncSize64 : MxEncSize<0b11>;
 208: 
 209: // M68k INSTRUCTION. Most instructions specify the location of an operand by
 210: // using the effective address field in the operation word. The effective address
 211: // is composed of two 3-bit fields: the mode field and the register field. The
 212: // value in the mode field selects the different address modes. The register
 213: // field contains the number of a register.  The effective address field may
 214: // require additional information to fully specify the operand. This additional
 215: // information, called the effective address extension, is contained in the
 216: // following word or words and is considered part of the instruction. The
```
- **EN**: This block declares or refines TableGen records such as `MxEncSize`, `MxEncSize8`, `MxEncSize16`, `MxEncSize32`, `MxEncSize64`.
- **CN**: 该代码块声明或细化了 `MxEncSize`, `MxEncSize8`, `MxEncSize16`, `MxEncSize32`, `MxEncSize64` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217: // effective address modes are grouped into three categories: register direct,
 218: // memory addressing, and special.
 219: class MxInst<dag outs, dag ins,
 220:              string asmStr = "",
 221:              list<dag> pattern = [],
 222:              InstrItinClass itin = NoItinerary>
 223:     : Instruction {
 224:   let Namespace      = "M68k";
 225:   let OutOperandList = outs;
 226:   let InOperandList  = ins;
 227:   let AsmString      = asmStr;
 228:   let Pattern        = pattern;
 229:   let Itinerary      = itin;
 230: 
 231:   dag Inst = (ascend);
 232: 
 233:   // Number of bytes
 234:   let Size = 0;
```
- **EN**: This block declares or refines TableGen records such as `MxInst`.
- **CN**: 该代码块声明或细化了 `MxInst` 等 TableGen 记录。

### Lines 235-243 / 第 235-243 行
```tablegen
 235: 
 236:   let UseLogicalOperandMappings = 1;
 237: }
 238: 
 239: // M68k PSEUDO INSTRUCTION
 240: class MxPseudo<dag outs, dag ins, list<dag> pattern = []>
 241:     : MxInst<outs, ins, "; error: this should not be emitted", pattern> {
 242:   let isPseudo = 1;
 243: }
```
- **EN**: This block declares or refines TableGen records such as `MxPseudo`.
- **CN**: 该代码块声明或细化了 `MxPseudo` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
