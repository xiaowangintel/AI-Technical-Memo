# LoongArchFloatInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchFloatInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 LoongArch 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: // LoongArchFloatInstrFormats.td - LoongArch FP Instr Formats -*- tablegen -*-//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Describe LoongArch floating-point instructions format
  11: //
  12: //  opcode       - operation code.
  13: //  fd           - destination register operand.
  14: //  {c/f}{j/k/a} - source register operand.
  15: //  immN         - immediate data operand.
  16: //
  17: //===----------------------------------------------------------------------===//
  18: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: // Some FP instructions are defined twice, for accepting FPR32 and FPR64, but
  20: // with the same mnemonic. Also some are codegen-only definitions that
  21: // nevertheless require a "normal" mnemonic.
  22: //
  23: // In order to accommodate these needs, the instruction defs have names
  24: // suffixed with `_x[SD]` or `_64`, that will get trimmed before the mnemonics
  25: // are derived.
  26: class deriveFPInsnMnemonic<string name> {
  27:   string ret = deriveInsnMnemonic<!subst("_64", "",
  28:                                          !subst("_xD", "",
  29:                                                 !subst("_xS", "", name)))>.ret;
  30: }
  31: 
  32: // 2R-type
  33: // <opcode | fj | fd>
  34: class FPFmt2R<bits<32> op, dag outs, dag ins, string opnstr,
  35:               list<dag> pattern = []>
  36:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
```
- **EN**: This block declares or refines TableGen records such as `deriveFPInsnMnemonic`, `FPFmt2R`.
- **CN**: 该代码块声明或细化了 `deriveFPInsnMnemonic`, `FPFmt2R` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37:   bits<5> fj;
  38:   bits<5> fd;
  39: 
  40:   let Inst{31-0} = op;
  41:   let Inst{9-5} = fj;
  42:   let Inst{4-0} = fd;
  43: }
  44: 
  45: // 3R-type
  46: // <opcode | fk | fj | fd>
  47: class FPFmt3R<bits<32> op, dag outs, dag ins, string opnstr,
  48:               list<dag> pattern = []>
  49:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
  50:   bits<5> fk;
  51:   bits<5> fj;
  52:   bits<5> fd;
  53: 
  54:   let Inst{31-0} = op;
```
- **EN**: This block declares or refines TableGen records such as `FPFmt3R`.
- **CN**: 该代码块声明或细化了 `FPFmt3R` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55:   let Inst{14-10} = fk;
  56:   let Inst{9-5} = fj;
  57:   let Inst{4-0} = fd;
  58: }
  59: 
  60: // 4R-type
  61: // <opcode | fa | fk | fj | fd>
  62: class FPFmt4R<bits<32> op, dag outs, dag ins, string opnstr,
  63:               list<dag> pattern = []>
  64:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
  65:   bits<5> fa;
  66:   bits<5> fk;
  67:   bits<5> fj;
  68:   bits<5> fd;
  69: 
  70:   let Inst{31-0} = op;
  71:   let Inst{19-15} = fa;
  72:   let Inst{14-10} = fk;
```
- **EN**: This block declares or refines TableGen records such as `FPFmt4R`.
- **CN**: 该代码块声明或细化了 `FPFmt4R` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:   let Inst{9-5} = fj;
  74:   let Inst{4-0} = fd;
  75: }
  76: 
  77: // 2RI12-type
  78: // <opcode | I12 | rj | fd>
  79: class FPFmt2RI12<bits<32> op, dag outs, dag ins, string opnstr,
  80:                  list<dag> pattern = []>
  81:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
  82:   bits<12> imm12;
  83:   bits<5> rj;
  84:   bits<5> fd;
  85: 
  86:   let Inst{31-0} = op;
  87:   let Inst{21-10} = imm12;
  88:   let Inst{9-5} = rj;
  89:   let Inst{4-0} = fd;
  90: }
```
- **EN**: This block declares or refines TableGen records such as `FPFmt2RI12`.
- **CN**: 该代码块声明或细化了 `FPFmt2RI12` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: 
  92: // FmtFCMP
  93: // <opcode | fk | fj | cd>
  94: class FPFmtFCMP<bits<32> op, dag outs, dag ins, string opnstr,
  95:                 list<dag> pattern = []>
  96:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
  97:   bits<5> fk;
  98:   bits<5> fj;
  99:   bits<3> cd;
 100: 
 101:   let Inst{31-0} = op;
 102:   let Inst{14-10} = fk;
 103:   let Inst{9-5} = fj;
 104:   let Inst{2-0} = cd;
 105: }
 106: 
 107: // FPFmtBR
 108: // <opcode | I21[15:0] | cj | I21[20:16]>
```
- **EN**: This block declares or refines TableGen records such as `FPFmtFCMP`.
- **CN**: 该代码块声明或细化了 `FPFmtFCMP` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: class FPFmtBR<bits<32> op, dag outs, dag ins, string opnstr,
 110:               list<dag> pattern = []>
 111:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
 112:   bits<21> imm21;
 113:   bits<3> cj;
 114: 
 115:   let Inst{31-0} = op;
 116:   let Inst{25-10} = imm21{15-0};
 117:   let Inst{7-5} = cj;
 118:   let Inst{4-0} = imm21{20-16};
 119: }
 120: 
 121: // FmtFSEL
 122: // <opcode | ca | fk | fj | fd>
 123: class FPFmtFSEL<bits<32> op, dag outs, dag ins, string opnstr,
 124:                 list<dag> pattern = []>
 125:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
 126:   bits<3> ca;
```
- **EN**: This block declares or refines TableGen records such as `FPFmtBR`, `FPFmtFSEL`.
- **CN**: 该代码块声明或细化了 `FPFmtBR`, `FPFmtFSEL` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127:   bits<5> fk;
 128:   bits<5> fj;
 129:   bits<5> fd;
 130: 
 131:   let Inst{31-0} = op;
 132:   let Inst{17-15} = ca;
 133:   let Inst{14-10} = fk;
 134:   let Inst{9-5} = fj;
 135:   let Inst{4-0} = fd;
 136: }
 137: 
 138: // FPFmtMOV
 139: // <opcode | src | dst>
 140: class FPFmtMOV<bits<32> op, dag outs, dag ins, string opnstr,
 141:                list<dag> pattern = []>
 142:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
 143:   bits<5> src;
 144:   bits<5> dst;
```
- **EN**: This block declares or refines TableGen records such as `FPFmtMOV`.
- **CN**: 该代码块声明或细化了 `FPFmtMOV` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: 
 146:   let Inst{31-0} = op;
 147:   let Inst{9-5} = src;
 148:   let Inst{4-0} = dst;
 149: }
 150: 
 151: // FPFmtMEM
 152: // <opcode | rk | rj | fd>
 153: class FPFmtMEM<bits<32> op, dag outs, dag ins, string opnstr,
 154:                list<dag> pattern = []>
 155:     : LAInst<outs, ins, deriveFPInsnMnemonic<NAME>.ret, opnstr, pattern> {
 156:   bits<5> rk;
 157:   bits<5> rj;
 158:   bits<5> fd;
 159: 
 160:   let Inst{31-0} = op;
 161:   let Inst{14-10} = rk;
 162:   let Inst{9-5} = rj;
```
- **EN**: This block declares or refines TableGen records such as `FPFmtMEM`.
- **CN**: 该代码块声明或细化了 `FPFmtMEM` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163:   let Inst{4-0} = fd;
 164: }
 165: 
 166: //===----------------------------------------------------------------------===//
 167: // Instruction class templates
 168: //===----------------------------------------------------------------------===//
 169: 
 170: let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
 171: class FP_ALU_2R<bits<32> op, RegisterClass rc = FPR32>
 172:     : FPFmt2R<op, (outs rc:$fd), (ins rc:$fj), "$fd, $fj">;
 173: 
 174: class FP_ALU_3R<bits<32> op, RegisterClass rc = FPR32>
 175:     : FPFmt3R<op, (outs rc:$fd), (ins rc:$fj, rc:$fk), "$fd, $fj, $fk">;
 176: 
 177: class FP_ALU_4R<bits<32> op, RegisterClass rc = FPR32>
 178:     : FPFmt4R<op, (outs rc:$fd), (ins rc:$fj, rc:$fk, rc:$fa),
 179:               "$fd, $fj, $fk, $fa">;
 180: } // hasSideEffects = 0, mayLoad = 0, mayStore = 0
```
- **EN**: This block declares or refines TableGen records such as `FP_ALU_2R`, `FP_ALU_3R`, `FP_ALU_4R`.
- **CN**: 该代码块声明或细化了 `FP_ALU_2R`, `FP_ALU_3R`, `FP_ALU_4R` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: 
 182: let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
 183: class FP_CMP<bits<32> op, RegisterClass rc = FPR32>
 184:     : FPFmtFCMP<op, (outs CFR:$cd), (ins rc:$fj, rc:$fk), "$cd, $fj, $fk">;
 185: 
 186: class FP_CONV<bits<32> op, RegisterClass rcd = FPR32, RegisterClass rcs = FPR32>
 187:     : FPFmt2R<op, (outs rcd:$fd), (ins rcs:$fj), "$fd, $fj">;
 188: 
 189: class FP_MOV<bits<32> op, RegisterClass rcd = FPR32, RegisterClass rcs = FPR32>
 190:     : FPFmtMOV<op, (outs rcd:$dst), (ins rcs:$src), "$dst, $src">;
 191: 
 192: class FP_SEL<bits<32> op, RegisterClass rc = FPR32>
 193:     : FPFmtFSEL<op, (outs rc:$fd), (ins rc:$fj, rc:$fk, CFR:$ca),
 194:                 "$fd, $fj, $fk, $ca">;
 195: 
 196: class FP_BRANCH<bits<32> opcode>
 197:     : FPFmtBR<opcode, (outs), (ins CFR:$cj, simm21_lsl2:$imm21),
 198:               "$cj, $imm21"> {
```
- **EN**: This block declares or refines TableGen records such as `FP_CMP`, `FP_CONV`, `FP_MOV`, `FP_SEL`, `FP_BRANCH`.
- **CN**: 该代码块声明或细化了 `FP_CMP`, `FP_CONV`, `FP_MOV`, `FP_SEL`, `FP_BRANCH` 等 TableGen 记录。

### Lines 199-216 / 第 199-216 行
```tablegen
 199:   let isBranch = 1;
 200:   let isTerminator = 1;
 201: }
 202: } // hasSideEffects = 0, mayLoad = 0, mayStore = 0
 203: 
 204: let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in {
 205: class FP_LOAD_3R<bits<32> op, RegisterClass rc = FPR32>
 206:     : FPFmtMEM<op, (outs rc:$fd), (ins GPR:$rj, GPR:$rk),
 207:                "$fd, $rj, $rk">;
 208: class FP_LOAD_2RI12<bits<32> op, RegisterClass rc = FPR32>
 209:     : FPFmt2RI12<op, (outs rc:$fd), (ins GPR:$rj, simm12_addlike:$imm12),
 210:                  "$fd, $rj, $imm12">;
 211: } // hasSideEffects = 0, mayLoad = 1, mayStore = 0
 212: 
 213: let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in {
 214: class FP_STORE_3R<bits<32> op, RegisterClass rc = FPR32>
 215:     : FPFmtMEM<op, (outs), (ins rc:$fd, GPR:$rj, GPR:$rk),
 216:                "$fd, $rj, $rk">;
```
- **EN**: This block declares or refines TableGen records such as `FP_LOAD_3R`, `FP_LOAD_2RI12`, `FP_STORE_3R`.
- **CN**: 该代码块声明或细化了 `FP_LOAD_3R`, `FP_LOAD_2RI12`, `FP_STORE_3R` 等 TableGen 记录。

### Lines 217-232 / 第 217-232 行
```tablegen
 217: class FP_STORE_2RI12<bits<32> op, RegisterClass rc = FPR32>
 218:     : FPFmt2RI12<op, (outs), (ins rc:$fd, GPR:$rj, simm12_addlike:$imm12),
 219:                  "$fd, $rj, $imm12">;
 220: } // hasSideEffects = 0, mayLoad = 0, mayStore = 1
 221: 
 222: // This class is used to define `SET_CFR_{FALSE,TRUE}` instructions which are
 223: // used to expand `PseudoCopyCFR`.
 224: class SET_CFR<bits<32> op, string opcstr>
 225:     : FP_CMP<op> {
 226:   let isCodeGenOnly = 1;
 227:   let fj = 0; // fa0
 228:   let fk = 0; // fa0
 229:   let AsmString = opcstr # "\t$cd, $$fa0, $$fa0";
 230:   let OutOperandList = (outs CFR:$cd);
 231:   let InOperandList = (ins);
 232: }
```
- **EN**: This block declares or refines TableGen records such as `FP_STORE_2RI12`, `SET_CFR`.
- **CN**: 该代码块声明或细化了 `FP_STORE_2RI12`, `SET_CFR` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
