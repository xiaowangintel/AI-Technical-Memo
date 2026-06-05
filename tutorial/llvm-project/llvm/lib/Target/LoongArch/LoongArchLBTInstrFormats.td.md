# LoongArchLBTInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchLBTInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 LoongArch 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: // LoongArchLBTInstrFormats.td - LoongArch LBT Instr Formats -*- tablegen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Describe LoongArch LBT instructions format
  11: //
  12: //  opcode        - operation code.
  13: //  rd/sd         - destination register operand.
  14: //  rj/rk/sj      - source register operand.
  15: //  immN/ptr      - immediate data operand.
  16: //
  17: //  Note: The definition of "NoDstFmt..." conveys the meaning of no explicit
  18: //  output operand. In other words, there will be no output operand in the
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: //  assembly notation of these instructions. In fact, they always manipulate
  20: //  the "EFLAGS" register.
  21: //  Since these instructions are currently not used for code generation,
  22: //  we do not need to add `let Defs/Uses = [EFLAGS]`.
  23: //===----------------------------------------------------------------------===//
  24: 
  25: // 1R-type (no outs)
  26: // <opcode | rj>
  27: class NoDstFmt1R<bits<32> op>
  28:     : LAInst<(outs), (ins GPR:$rj),
  29:              deriveInsnMnemonic<NAME>.ret, "$rj"> {
  30:   bits<5> rj;
  31: 
  32:   let Inst{31-0} = op;
  33:   let Inst{9-5} = rj;
  34: }
  35: 
  36: // 1RI3-type (no outs)
```
- **EN**: This block declares or refines TableGen records such as `NoDstFmt1R`.
- **CN**: 该代码块声明或细化了 `NoDstFmt1R` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37: // <opcode | I3 | rj>
  38: class NoDstFmt1RI3<bits<32> op>
  39:     : LAInst<(outs), (ins GPR:$rj, uimm3:$imm3),
  40:              deriveInsnMnemonic<NAME>.ret, "$rj, $imm3"> {
  41:   bits<3> imm3;
  42:   bits<5> rj;
  43: 
  44:   let Inst{31-0} = op;
  45:   let Inst{12-10} = imm3;
  46:   let Inst{9-5} = rj;
  47: }
  48: 
  49: // 1RI4-type (no outs)
  50: // <opcode | I4 | rj>
  51: class NoDstFmt1RI4<bits<32> op>
  52:     : LAInst<(outs), (ins GPR:$rj, uimm4:$imm4),
  53:              deriveInsnMnemonic<NAME>.ret, "$rj, $imm4"> {
  54:   bits<4> imm4;
```
- **EN**: This block declares or refines TableGen records such as `NoDstFmt1RI3`, `NoDstFmt1RI4`.
- **CN**: 该代码块声明或细化了 `NoDstFmt1RI3`, `NoDstFmt1RI4` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55:   bits<5> rj;
  56: 
  57:   let Inst{31-0} = op;
  58:   let Inst{13-10} = imm4;
  59:   let Inst{9-5} = rj;
  60: }
  61: 
  62: // 1RI4-type
  63: // <opcode | I4 | rd>
  64: class Fmt1RI4<bits<32> op>
  65:     : LAInst<(outs GPR:$rd), (ins uimm4:$imm4),
  66:              deriveInsnMnemonic<NAME>.ret, "$rd, $imm4"> {
  67:   bits<4> imm4;
  68:   bits<5> rd;
  69: 
  70:   let Inst{31-0} = op;
  71:   let Inst{13-10} = imm4;
  72:   let Inst{4-0} = rd;
```
- **EN**: This block declares or refines TableGen records such as `Fmt1RI4`.
- **CN**: 该代码块声明或细化了 `Fmt1RI4` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73: }
  74: 
  75: // 1RI5-type (no outs)
  76: // <opcode | I5 | rj>
  77: class NoDstFmt1RI5<bits<32> op>
  78:     : LAInst<(outs), (ins GPR:$rj, uimm5:$imm5),
  79:              deriveInsnMnemonic<NAME>.ret, "$rj, $imm5"> {
  80:   bits<5> imm5;
  81:   bits<5> rj;
  82: 
  83:   let Inst{31-0} = op;
  84:   let Inst{14-10} = imm5;
  85:   let Inst{9-5} = rj;
  86: }
  87: 
  88: // 1RI5I4-type (no outs)
  89: // <opcode | rd | I5 | I4>
  90: class NoDstFmt1RI5I4<bits<32> op>
```
- **EN**: This block declares or refines TableGen records such as `NoDstFmt1RI5`, `NoDstFmt1RI5I4`.
- **CN**: 该代码块声明或细化了 `NoDstFmt1RI5`, `NoDstFmt1RI5I4` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91:     : LAInst<(outs), (ins  GPR:$rj, uimm5:$imm5, uimm4:$imm4),
  92:              deriveInsnMnemonic<NAME>.ret, "$rj, $imm5, $imm4"> {
  93:   bits<5> imm5;
  94:   bits<5> rj;
  95:   bits<4> imm4;
  96: 
  97:   let Inst{31-0} = op;
  98:   let Inst{14-10} = imm5;
  99:   let Inst{9-5} = rj;
 100:   let Inst{3-0} = imm4;
 101: }
 102: 
 103: // 1RI5I8-type
 104: // <opcode | rd | I5 | I8>
 105: class Fmt1RI5I8<bits<32> op>
 106:     : LAInst<(outs GPR:$rd), (ins uimm5:$imm5, uimm8:$imm8),
 107:              deriveInsnMnemonic<NAME>.ret, "$rd, $imm5, $imm8"> {
 108:   bits<8> imm8;
```
- **EN**: This block declares or refines TableGen records such as `Fmt1RI5I8`.
- **CN**: 该代码块声明或细化了 `Fmt1RI5I8` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109:   bits<5> imm5;
 110:   bits<5> rd;
 111: 
 112:   let Inst{31-0} = op;
 113:   let Inst{17-10} = imm8;
 114:   let Inst{9-5} = imm5;
 115:   let Inst{4-0} = rd;
 116: }
 117: 
 118: // 1RI6-type (no outs)
 119: // <opcode | I6 | rj>
 120: class NoDstFmt1RI6<bits<32> op>
 121:     : LAInst<(outs), (ins GPR:$rj, uimm6:$imm6),
 122:              deriveInsnMnemonic<NAME>.ret, "$rj, $imm6"> {
 123:   bits<6> imm6;
 124:   bits<5> rj;
 125: 
 126:   let Inst{31-0} = op;
```
- **EN**: This block declares or refines TableGen records such as `NoDstFmt1RI6`.
- **CN**: 该代码块声明或细化了 `NoDstFmt1RI6` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127:   let Inst{15-10} = imm6;
 128:   let Inst{9-5} = rj;
 129: }
 130: 
 131: // 1RI8-type
 132: // <opcode | I8 | rd>
 133: class Fmt1RI8<bits<32> op>
 134:     : LAInst<(outs GPR:$rd), (ins uimm8:$imm8),
 135:              deriveInsnMnemonic<NAME>.ret, "$rd, $imm8"> {
 136:   bits<8> imm8;
 137:   bits<5> rd;
 138: 
 139:   let Inst{31-0} = op;
 140:   let Inst{17-10} = imm8;
 141:   let Inst{4-0} = rd;
 142: }
 143: 
 144: // 2R-type (no outs)
```
- **EN**: This block declares or refines TableGen records such as `Fmt1RI8`.
- **CN**: 该代码块声明或细化了 `Fmt1RI8` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: // <opcode | rk | rj>
 146: class NoDstFmt2R<bits<32> op>
 147:     : LAInst<(outs), (ins GPR:$rj, GPR:$rk),
 148:              deriveInsnMnemonic<NAME>.ret, "$rj, $rk"> {
 149:   bits<5> rk;
 150:   bits<5> rj;
 151: 
 152:   let Inst{31-0} = op;
 153:   let Inst{14-10} = rk;
 154:   let Inst{9-5} = rj;
 155: }
 156: 
 157: // 2RI4-type (no outs)
 158: // <opcode | rk | rj | imm4>
 159: class NoDstFmt2RI4<bits<32> op>
 160:     : LAInst<(outs), (ins GPR:$rj, GPR:$rk, uimm4:$imm4),
 161:              deriveInsnMnemonic<NAME>.ret, "$rj, $rk, $imm4"> {
 162:   bits<4> imm4;
```
- **EN**: This block declares or refines TableGen records such as `NoDstFmt2R`, `NoDstFmt2RI4`.
- **CN**: 该代码块声明或细化了 `NoDstFmt2R`, `NoDstFmt2RI4` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163:   bits<5> rk;
 164:   bits<5> rj;
 165: 
 166:   let Inst{31-0} = op;
 167:   let Inst{14-10} = rk;
 168:   let Inst{9-5} = rj;
 169:   let Inst{3-0} = imm4;
 170: }
 171: 
 172: // 2RI3-type
 173: // <opcode | I3 | rj | rd>
 174: class Fmt2RI3<bits<32> op>
 175:     : LAInst<(outs GPR:$rd), (ins GPR:$rj, uimm3:$imm3),
 176:              deriveInsnMnemonic<NAME>.ret, "$rd, $rj, $imm3"> {
 177:   bits<3> imm3;
 178:   bits<5> rj;
 179:   bits<5> rd;
 180: 
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI3`.
- **CN**: 该代码块声明或细化了 `Fmt2RI3` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181:   let Inst{31-0} = op;
 182:   let Inst{12-10} = imm3;
 183:   let Inst{9-5} = rj;
 184:   let Inst{4-0} = rd;
 185: }
 186: 
 187: // 2RI4-type
 188: // <opcode | I4 | rj | rd>
 189: class Fmt2RI4<bits<32> op>
 190:     : LAInst<(outs GPR:$rd), (ins GPR:$rj, uimm4:$imm4),
 191:              deriveInsnMnemonic<NAME>.ret, "$rd, $rj, $imm4"> {
 192:   bits<4> imm4;
 193:   bits<5> rj;
 194:   bits<5> rd;
 195: 
 196:   let Inst{31-0} = op;
 197:   let Inst{13-10} = imm4;
 198:   let Inst{9-5} = rj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI4`.
- **CN**: 该代码块声明或细化了 `Fmt2RI4` 等 TableGen 记录。

### Lines 199-216 / 第 199-216 行
```tablegen
 199:   let Inst{4-0} = rd;
 200: }
 201: 
 202: // <opcode | rj | sd>
 203: class FmtGR2SCR<bits<32> op>
 204:     : LAInst<(outs SCR:$sd), (ins GPR:$rj), deriveInsnMnemonic<NAME>.ret,
 205:              "$sd, $rj"> {
 206:   bits<5> rj;
 207:   bits<2> sd;
 208: 
 209:   let Inst{31-0} = op;
 210:   let Inst{9-5} = rj;
 211:   let Inst{1-0} = sd;
 212: }
 213: 
 214: // <opcode | sj | rd>
 215: class FmtSCR2GR<bits<32> op>
 216:     : LAInst<(outs GPR:$rd), (ins SCR:$sj), deriveInsnMnemonic<NAME>.ret,
```
- **EN**: This block declares or refines TableGen records such as `FmtGR2SCR`, `FmtSCR2GR`.
- **CN**: 该代码块声明或细化了 `FmtGR2SCR`, `FmtSCR2GR` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217:              "$rd, $sj"> {
 218:   bits<2> sj;
 219:   bits<5> rd;
 220: 
 221:   let Inst{31-0} = op;
 222:   let Inst{6-5} = sj;
 223:   let Inst{4-0} = rd;
 224: }
 225: 
 226: // <opcode | I21[15:0] | I21[20:16]>
 227: class FmtJISCR<bits<32> op>
 228:     : LAInst<(outs), (ins simm21_lsl2:$imm21), deriveInsnMnemonic<NAME>.ret,
 229:               "$imm21"> {
 230:   bits<21> imm21;
 231:   bits<5> rj;
 232: 
 233:   let Inst{31-0} = op;
 234:   let Inst{25-10} = imm21{15-0};
```
- **EN**: This block declares or refines TableGen records such as `FmtJISCR`.
- **CN**: 该代码块声明或细化了 `FmtJISCR` 等 TableGen 记录。

### Lines 235-252 / 第 235-252 行
```tablegen
 235:   let Inst{4-0} = imm21{20-16};
 236: }
 237: 
 238: // <opcode | rd>
 239: class FmtMFTOP<bits<32> op>
 240:     : LAInst<(outs GPR:$rd), (ins), deriveInsnMnemonic<NAME>.ret,
 241:              "$rd"> {
 242:   bits<5> rd;
 243: 
 244:   let Inst{31-0} = op;
 245:   let Inst{4-0} = rd;
 246: }
 247: 
 248: // <opcode | ptr>
 249: class FmtMTTOP<bits<32> op>
 250:     : LAInst<(outs), (ins uimm3:$ptr), deriveInsnMnemonic<NAME>.ret,
 251:              "$ptr"> {
 252:   bits<3> ptr;
```
- **EN**: This block declares or refines TableGen records such as `FmtMFTOP`, `FmtMTTOP`.
- **CN**: 该代码块声明或细化了 `FmtMFTOP`, `FmtMTTOP` 等 TableGen 记录。

### Lines 253-256 / 第 253-256 行
```tablegen
 253: 
 254:   let Inst{31-0} = op;
 255:   let Inst{7-5} = ptr;
 256: }
```
- **EN**: This span continues the file's main responsibility: defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
