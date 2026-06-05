# SparcInstrVIS.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcInstrVIS.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines backend metadata in TableGen DSL that is later expanded into generated tables and matcher code.
  - **CN**: 使用 TableGen DSL 定义后端元数据，随后会展开为生成的表和匹配代码。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===---- SparcInstrVIS.td - Visual Instruction Set extensions (VIS) -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains instruction formats, definitions and patterns needed for
// VIS, VIS II, VIS III instructions on SPARC.
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-21
```tablegen

// VIS Instruction Format.
class VISInstFormat<bits<9> opfval, dag outs, dag ins, string asmstr>
      : F3_3<0b10, 0b110110, opfval, outs, ins, asmstr, []>;

class VISInst<bits<9> opfval, string OpcStr, RegisterClass RC = DFPRegs>
       : VISInstFormat<opfval,
        (outs RC:$rd), (ins RC:$rs1, RC:$rs2),
        !strconcat(OpcStr, " $rs1, $rs2, $rd")>;

```
- **EN**: Declares TableGen records such as `VISInstFormat`, `VISInst`; this block describes instruction encoding bits.
- **CN**: 声明了 `VISInstFormat`, `VISInst` 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 22-32
```tablegen
// VIS Instruction with integer destination register.
class VISInstID<bits<9> opfval, string OpcStr>
       : VISInstFormat<opfval,
        (outs I64Regs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
        !strconcat(OpcStr, " $rs1, $rs2, $rd")>;

// For VIS Instructions with no operand.
let rd = 0, rs1 = 0, rs2 = 0 in
class VISInst0<bits<9> opfval, string asmstr>
       : VISInstFormat<opfval, (outs), (ins), asmstr>;

```
- **EN**: Declares TableGen records such as `VISInstID`, `VISInst0`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `VISInstID`, `VISInst0` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 33-46
```tablegen
// For VIS Instructions with only rs1, rd operands.
let rs2 = 0 in
class VISInst1<bits<9> opfval, string OpcStr, RegisterClass RC = DFPRegs>
       : VISInstFormat<opfval,
        (outs RC:$rd), (ins RC:$rs1),
        !strconcat(OpcStr, " $rs1, $rd")>;

// For VIS Instructions with only rs2, rd operands.
let rs1 = 0 in
class VISInst2<bits<9> opfval, string OpcStr, RegisterClass RC = DFPRegs>
       : VISInstFormat<opfval,
        (outs RC:$rd), (ins RC:$rs2),
        !strconcat(OpcStr, " $rs2, $rd")>;

```
- **EN**: Declares TableGen records such as `VISInst1`, `VISInst2`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `VISInst1`, `VISInst2` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 47-56
```tablegen
// For VIS Instructions with only rd operand.
let rs1 = 0, rs2 = 0 in
class VISInstD<bits<9> opfval, string OpcStr, RegisterClass RC = DFPRegs>
       : VISInstFormat<opfval,
        (outs RC:$rd), (ins),
        !strconcat(OpcStr, " $rd")>;

// VIS 1 Instructions
let Predicates = [HasVIS] in {

```
- **EN**: Declares TableGen records such as `VISInstD`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `VISInstD` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 57-65
```tablegen
def FPADD16     : VISInst<0b001010000, "fpadd16">;
def FPADD16S    : VISInst<0b001010001, "fpadd16s", FPRegs>;
def FPADD32     : VISInst<0b001010010, "fpadd32">;
def FPADD32S    : VISInst<0b001010011, "fpadd32s", FPRegs>;
def FPSUB16     : VISInst<0b001010100, "fpsub16">;
def FPSUB16S    : VISInst<0b001010101, "fpsub16s", FPRegs>;
def FPSUB32     : VISInst<0b001010110, "fpsub32">;
def FPSUB32S    : VISInst<0b001010111, "fpsub32s", FPRegs>;

```
- **EN**: Declares TableGen records such as `FPADD16`, `FPADD16S`, `FPADD32`, `FPADD32S`, ... for the backend description.
- **CN**: 为后端描述声明了 `FPADD16`, `FPADD16S`, `FPADD32`, `FPADD32S`, ... 等 TableGen 记录。

### Lines 66-77
```tablegen
def FPACK16     : VISInst2<0b000111011, "fpack16">;
def FPACK32     : VISInst <0b000111010, "fpack32">;
let rs1 = 0 in
def FPACKFIX    : VISInstFormat<0b000111101,
                  (outs FPRegs:$rd), (ins DFPRegs:$rs2), "fpackfix $rs2, $rd">;
let rs1 = 0 in
def FEXPAND     : VISInstFormat<0b001001101,
                  (outs DFPRegs:$rd), (ins FPRegs:$rs2), "fexpand $rs2, $rd">;
def FPMERGE     : VISInstFormat<0b001001011,
                  (outs DFPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                  "fpmerge $rs1, $rs2, $rd">;

```
- **EN**: Declares TableGen records such as `FPACK16`, `FPACK32`, `FPACKFIX`, `FEXPAND`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `FPACK16`, `FPACK32`, `FPACKFIX`, `FEXPAND`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 78-95
```tablegen
def FMUL8X16    : VISInstFormat<0b000110001,
                  (outs DFPRegs:$rd), (ins FPRegs:$rs1, DFPRegs:$rs2),
                  "fmul8x16 $rs1, $rs2, $rd">;
def FMUL8X16AU  : VISInstFormat<0b000110011,
                  (outs DFPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                  "fmul8x16au $rs1, $rs2, $rd">;
def FMUL8X16AL  : VISInstFormat<0b000110101,
                  (outs DFPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                  "fmul8x16al $rs1, $rs2, $rd">;
def FMUL8SUX16  : VISInst<0b000110110, "fmul8sux16">;
def FMUL8ULX16  : VISInst<0b000110111, "fmul8ulx16">;
def FMULD8SUX16 : VISInstFormat<0b000111000,
                  (outs DFPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                  "fmuld8sux16 $rs1, $rs2, $rd">;
def FMULD8ULX16 : VISInstFormat<0b000111001,
                  (outs DFPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                  "fmuld8ulx16 $rs1, $rs2, $rd">;

```
- **EN**: Declares TableGen records such as `FMUL8X16`, `FMUL8X16AU`, `FMUL8X16AL`, `FMUL8SUX16`, ... for the backend description.
- **CN**: 为后端描述声明了 `FMUL8X16`, `FMUL8X16AU`, `FMUL8X16AL`, `FMUL8SUX16`, ... 等 TableGen 记录。

### Lines 96-113
```tablegen
def ALIGNADDR   : VISInst<0b000011000, "alignaddr", I64Regs>;
def ALIGNADDRL  : VISInst<0b000011010, "alignaddrl", I64Regs>;
def FALIGNADATA : VISInst<0b001001000, "faligndata">;

def FZERO       : VISInstD<0b001100000, "fzero">;
def FZEROS      : VISInstD<0b001100001, "fzeros", FPRegs>;
def FONE        : VISInstD<0b001111110, "fone">;
def FONES       : VISInstD<0b001111111, "fones", FPRegs>;
def FSRC1       : VISInst1<0b001110100, "fsrc1">;
def FSRC1S      : VISInst1<0b001110101, "fsrc1s", FPRegs>;
def FSRC2       : VISInst2<0b001111000, "fsrc2">;
def FSRC2S      : VISInst2<0b001111001, "fsrc2s", FPRegs>;
def FNOT1       : VISInst1<0b001101010, "fnot1">;
def FNOT1S      : VISInst1<0b001101011, "fnot1s", FPRegs>;
def FNOT2       : VISInst2<0b001100110, "fnot2">;
def FNOT2S      : VISInst2<0b001100111, "fnot2s", FPRegs>;
def FOR         : VISInst<0b001111100,  "for">;
def FORS        : VISInst<0b001111101,  "fors",  FPRegs>;
```
- **EN**: Declares TableGen records such as `ALIGNADDR`, `ALIGNADDRL`, `FALIGNADATA`, `FZERO`, ... for the backend description.
- **CN**: 为后端描述声明了 `ALIGNADDR`, `ALIGNADDRL`, `FALIGNADATA`, `FZERO`, ... 等 TableGen 记录。

### Lines 114-124
```tablegen
def FNOR        : VISInst<0b001100010,  "fnor">;
def FNORS       : VISInst<0b001100011,  "fnors", FPRegs>;
def FAND        : VISInst<0b001110000,  "fand">;
def FANDS       : VISInst<0b001110001,  "fands", FPRegs>;
def FNAND       : VISInst<0b001101110,  "fnand">;
def FNANDS      : VISInst<0b001101111,  "fnands", FPRegs>;
def FXOR        : VISInst<0b001101100,  "fxor">;
def FXORS       : VISInst<0b001101101,  "fxors", FPRegs>;
def FXNOR       : VISInst<0b001110010,  "fxnor">;
def FXNORS      : VISInst<0b001110011,  "fxnors", FPRegs>;

```
- **EN**: Declares TableGen records such as `FNOR`, `FNORS`, `FAND`, `FANDS`, ... for the backend description.
- **CN**: 为后端描述声明了 `FNOR`, `FNORS`, `FAND`, `FANDS`, ... 等 TableGen 记录。

### Lines 125-133
```tablegen
def FORNOT1     : VISInst<0b001111010,  "fornot1">;
def FORNOT1S    : VISInst<0b001111011,  "fornot1s",  FPRegs>;
def FORNOT2     : VISInst<0b001110110,  "fornot2">;
def FORNOT2S    : VISInst<0b001110111,  "fornot2s",  FPRegs>;
def FANDNOT1    : VISInst<0b001101000,  "fandnot1">;
def FANDNOT1S   : VISInst<0b001101001,  "fandnot1s", FPRegs>;
def FANDNOT2    : VISInst<0b001100100,  "fandnot2">;
def FANDNOT2S   : VISInst<0b001100101,  "fandnot2s", FPRegs>;

```
- **EN**: Declares TableGen records such as `FORNOT1`, `FORNOT1S`, `FORNOT2`, `FORNOT2S`, ... for the backend description.
- **CN**: 为后端描述声明了 `FORNOT1`, `FORNOT1S`, `FORNOT2`, `FORNOT2S`, ... 等 TableGen 记录。

### Lines 134-142
```tablegen
def FCMPGT16    : VISInstID<0b000101000,  "fcmpgt16">;
def FCMPGT32    : VISInstID<0b000101100,  "fcmpgt32">;
def FCMPLE16    : VISInstID<0b000100000,  "fcmple16">;
def FCMPLE32    : VISInstID<0b000100100,  "fcmple32">;
def FCMPNE16    : VISInstID<0b000100010,  "fcmpne16">;
def FCMPNE32    : VISInstID<0b000100110,  "fcmpne32">;
def FCMPEQ16    : VISInstID<0b000101010,  "fcmpeq16">;
def FCMPEQ32    : VISInstID<0b000101110,  "fcmpeq32">;

```
- **EN**: Declares TableGen records such as `FCMPGT16`, `FCMPGT32`, `FCMPLE16`, `FCMPLE32`, ... for the backend description.
- **CN**: 为后端描述声明了 `FCMPGT16`, `FCMPGT32`, `FCMPLE16`, `FCMPLE32`, ... 等 TableGen 记录。

### Lines 143-152
```tablegen

def EDGE8       : VISInst<0b000000000,  "edge8",   I64Regs>;
def EDGE8L      : VISInst<0b000000010,  "edge8l",  I64Regs>;
def EDGE16      : VISInst<0b000000100,  "edge16",  I64Regs>;
def EDGE16L     : VISInst<0b000000110,  "edge16l", I64Regs>;
def EDGE32      : VISInst<0b000001000,  "edge32",  I64Regs>;
def EDGE32L     : VISInst<0b000001010,  "edge32l", I64Regs>;

def PDIST       : VISInst<0b000111110, "pdist">;

```
- **EN**: Declares TableGen records such as `EDGE8`, `EDGE8L`, `EDGE16`, `EDGE16L`, ... for the backend description.
- **CN**: 为后端描述声明了 `EDGE8`, `EDGE8L`, `EDGE16`, `EDGE16L`, ... 等 TableGen 记录。

### Lines 153-161
```tablegen
def ARRAY8      : VISInst<0b000010000, "array8",  I64Regs>;
def ARRAY16     : VISInst<0b000010010, "array16", I64Regs>;
def ARRAY32     : VISInst<0b000010100, "array32", I64Regs>;

def SHUTDOWN    : VISInst0<0b010000000, "shutdown">;

} // Predicates = [HasVIS]


```
- **EN**: Declares TableGen records such as `ARRAY8`, `ARRAY16`, `ARRAY32`, `SHUTDOWN` for the backend description.
- **CN**: 为后端描述声明了 `ARRAY8`, `ARRAY16`, `ARRAY32`, `SHUTDOWN` 等 TableGen 记录。

### Lines 162-171
```tablegen
// VIS 2 Instructions.
let Predicates = [HasVIS2] in {

def BMASK     : VISInst<0b000011001, "bmask", I64Regs>;
def BSHUFFLE  : VISInst<0b001001100, "bshuffle">;

let rd = 0, rs1 = 0 in
def SIAM      : F3_3_siam<0b10, 0b110110, 0b010000001, (outs),
                (ins i32imm:$mode), "siam $mode", []>;

```
- **EN**: Declares TableGen records such as `BMASK`, `BSHUFFLE`, `SIAM`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `BMASK`, `BSHUFFLE`, `SIAM` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 172-180
```tablegen
def EDGE8N    : VISInst<0b000000001,  "edge8n",   I64Regs>;
def EDGE8LN   : VISInst<0b000000011,  "edge8ln",  I64Regs>;
def EDGE16N   : VISInst<0b000000101,  "edge16n",  I64Regs>;
def EDGE16LN  : VISInst<0b000000111,  "edge16ln", I64Regs>;
def EDGE32N   : VISInst<0b000001001,  "edge32n",  I64Regs>;
def EDGE32LN  : VISInst<0b000001011,  "edge32ln", I64Regs>;
} // Predicates = [HasVIS2]


```
- **EN**: Declares TableGen records such as `EDGE8N`, `EDGE8LN`, `EDGE16N`, `EDGE16LN`, ... for the backend description.
- **CN**: 为后端描述声明了 `EDGE8N`, `EDGE8LN`, `EDGE16N`, `EDGE16LN`, ... 等 TableGen 记录。

### Lines 181-189
```tablegen
// VIS 3 Instructions.
let Predicates = [HasVIS3] in {

let Uses = [ICC] in
def ADDXC : VISInst<0b000010001, "addxc", I64Regs>;

let Defs = [ICC], Uses = [ICC] in
def ADDXCCC : VISInst<0b000010011, "addxccc", I64Regs>;

```
- **EN**: Declares TableGen records such as `ADDXC`, `ADDXCCC`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `ADDXC`, `ADDXCCC` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 190-199
```tablegen
let rd = 0, rs1 = 0 in {
def CMASK8  : VISInstFormat<0b000011011, (outs), (ins I64Regs:$rs2),
              "cmask8 $rs2">;
def CMASK16  : VISInstFormat<0b000011101, (outs), (ins I64Regs:$rs2),
              "cmask16 $rs2">;
def CMASK32  : VISInstFormat<0b000011111, (outs), (ins I64Regs:$rs2),
              "cmask32 $rs2">;

}

```
- **EN**: Declares TableGen records such as `CMASK8`, `CMASK16`, `CMASK32`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `CMASK8`, `CMASK16`, `CMASK32` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 200-217
```tablegen
def FCHKSM16 : VISInst<0b001000100, "fchksm16">;

def FHADDS   : F3_3<0b10, 0b110100, 0b001100001,
                    (outs FPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                    "fhadds $rs1, $rs2, $rd", []>;
def FHADDD   : F3_3<0b10, 0b110100, 0b001100010,
                    (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                    "fhaddd $rs1, $rs2, $rd", []>;
def FHSUBS   : F3_3<0b10, 0b110100, 0b001100101,
                    (outs FPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                    "fhsubs $rs1, $rs2, $rd", []>;
def FHSUBD   : F3_3<0b10, 0b110100, 0b001100110,
                    (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                    "fhsubd $rs1, $rs2, $rd", []>;
def FLCMPS   : VISInstFormat<0b101010001, (outs FCCRegs:$rd),
                     (ins FPRegs:$rs1, FPRegs:$rs2),
                     "flcmps $rd, $rs1, $rs2">;
def FLCMPD   : VISInstFormat<0b101010010, (outs FCCRegs:$rd),
```
- **EN**: Declares TableGen records such as `FCHKSM16`, `FHADDS`, `FHADDD`, `FHSUBS`, ... for the backend description.
- **CN**: 为后端描述声明了 `FCHKSM16`, `FHADDS`, `FHADDD`, `FHSUBS`, ... 等 TableGen 记录。

### Lines 218-235
```tablegen
                     (ins DFPRegs:$rs1, DFPRegs:$rs2),
                     "flcmpd $rd, $rs1, $rs2">;

def FMEAN16  : VISInst<0b001000000, "fmean16">;

def FNADDS   : F3_3<0b10, 0b110100, 0b001010001,
                    (outs FPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                    "fnadds $rs1, $rs2, $rd", []>;
def FNADDD   : F3_3<0b10, 0b110100, 0b001010010,
                    (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                    "fnaddd $rs1, $rs2, $rd", []>;
def FNHADDS  : F3_3<0b10, 0b110100, 0b001110001,
                    (outs FPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                    "fnhadds $rs1, $rs2, $rd", []>;
def FNHADDD  : F3_3<0b10, 0b110100, 0b001110010,
                    (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                    "fnhaddd $rs1, $rs2, $rd", []>;

```
- **EN**: Declares TableGen records such as `FMEAN16`, `FNADDS`, `FNADDD`, `FNHADDS`, ... for the backend description.
- **CN**: 为后端描述声明了 `FMEAN16`, `FNADDS`, `FNADDD`, `FNHADDS`, ... 等 TableGen 记录。

### Lines 236-245
```tablegen
def FNMULS   : F3_3<0b10, 0b110100, 0b001011001,
                    (outs FPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                    "fnmuls $rs1, $rs2, $rd", []>;
def FNMULD   : F3_3<0b10, 0b110100, 0b001011010,
                    (outs DFPRegs:$rd), (ins DFPRegs:$rs1, DFPRegs:$rs2),
                    "fnmuld $rs1, $rs2, $rd", []>;
def FNSMULD  : F3_3<0b10, 0b110100, 0b001111001,
                    (outs DFPRegs:$rd), (ins FPRegs:$rs1, FPRegs:$rs2),
                    "fnsmuld $rs1, $rs2, $rd", []>;

```
- **EN**: Declares TableGen records such as `FNMULS`, `FNMULD`, `FNSMULD` for the backend description.
- **CN**: 为后端描述声明了 `FNMULS`, `FNMULD`, `FNSMULD` 等 TableGen 记录。

### Lines 246-256
```tablegen
def FPADD64   : VISInst<0b001000010, "fpadd64">;

def FSLL16    : VISInst<0b000100001, "fsll16">;
def FSRL16    : VISInst<0b000100011, "fsrl16">;
def FSLL32    : VISInst<0b000100101, "fsll32">;
def FSRL32    : VISInst<0b000100111, "fsrl32">;
def FSLAS16   : VISInst<0b000101001, "fslas16">;
def FSRA16    : VISInst<0b000101011, "fsra16">;
def FSLAS32   : VISInst<0b000101101, "fslas32">;
def FSRA32    : VISInst<0b000101111, "fsra32">;

```
- **EN**: Declares TableGen records such as `FPADD64`, `FSLL16`, `FSRL16`, `FSLL32`, ... for the backend description.
- **CN**: 为后端描述声明了 `FPADD64`, `FSLL16`, `FSRL16`, `FSLL32`, ... 等 TableGen 记录。

### Lines 257-273
```tablegen
let rs1 = 0 in
def LZCNT     : VISInstFormat<0b000010111, (outs I64Regs:$rd),
                   (ins I64Regs:$rs2), "lzcnt $rs2, $rd">;

let rs1 = 0 in {
def MOVSTOSW : VISInstFormat<0b100010011, (outs IntRegs:$rd),
                   (ins FPRegs:$rs2), "movstosw $rs2, $rd">;
def MOVSTOUW : VISInstFormat<0b100010001, (outs IntRegs:$rd),
                   (ins FPRegs:$rs2), "movstouw $rs2, $rd">;
def MOVDTOX  : VISInstFormat<0b100010000, (outs I64Regs:$rd),
                   (ins DFPRegs:$rs2), "movdtox $rs2, $rd">;
def MOVWTOS  :  VISInstFormat<0b100011001, (outs FPRegs:$rd),
                   (ins IntRegs:$rs2), "movwtos $rs2, $rd">;
def MOVXTOD  :  VISInstFormat<0b100011000, (outs DFPRegs:$rd),
                   (ins I64Regs:$rs2), "movxtod $rs2, $rd">;
}

```
- **EN**: Declares TableGen records such as `LZCNT`, `MOVSTOSW`, `MOVSTOUW`, `MOVDTOX`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `LZCNT`, `MOVSTOSW`, `MOVSTOUW`, `MOVDTOX`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 274-286
```tablegen
def PDISTN   : VISInstID<0b000111111, "pdistn">;

def UMULXHI  : VISInst<0b000010110, "umulxhi", I64Regs>;
def XMULX    : VISInst<0b100010101, "xmulx",   I64Regs>;
def XMULXHI  : VISInst<0b100010110, "xmulxhi", I64Regs>;
} // Predicates = [IsVIS3]

// FP immediate patterns.
def fpimm0 : FPImmLeaf<fAny, [{return Imm.isExactlyValue(+0.0);}]>;
def fpnegimm0 : FPImmLeaf<fAny, [{return Imm.isExactlyValue(-0.0);}]>;
def fpimmhalf : FPImmLeaf<fAny, [{return Imm.isExactlyValue(+0.5);}]>;
def fpnegimmhalf : FPImmLeaf<fAny, [{return Imm.isExactlyValue(-0.5);}]>;

```
- **EN**: Declares TableGen records such as `PDISTN`, `UMULXHI`, `XMULX`, `XMULXHI`, ...; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `PDISTN`, `UMULXHI`, `XMULX`, `XMULXHI`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 287-295
```tablegen
// VIS instruction patterns.
let Predicates = [HasVIS] in {
// Zero immediate.
def : Pat<(f64 fpimm0), (FZERO)>;
def : Pat<(f32 fpimm0), (FZEROS)>;
def : Pat<(f64 fpnegimm0), (FNEGD (FZERO))>;
def : Pat<(f32 fpnegimm0), (FNEGS (FZEROS))>;
} // Predicates = [HasVIS]

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 296-307
```tablegen
// VIS3 instruction patterns.
let Predicates = [HasVIS3] in {
// +/-0.5 immediate.
// This is needed to enable halving instructions.
// FIXME generalize this to arbitrary immediates.
// SET/MOVWTOS or SETX/MOVXTOD pair should let us materialize FP constants
// faster than constant pool loading.
def : Pat<(f32 fpimmhalf), (MOVWTOS (SETHIi 0x0FC000))>;
def : Pat<(f32 fpnegimmhalf), (MOVWTOS (SETHIi 0x2FC000))>;
def : Pat<(f64 fpimmhalf), (MOVXTOD (SLLXri (SETHIi 0x0FF800), 32))>;
def : Pat<(f64 fpnegimmhalf), (MOVXTOD (SLLXri (SETHIi 0x2FF800), 32))>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 308-319
```tablegen
def : Pat<(i64 (adde i64:$lhs, i64:$rhs)), (ADDXCCC $lhs, $rhs)>;

def : Pat<(i64 (mulhu i64:$lhs, i64:$rhs)), (UMULXHI $lhs, $rhs)>;
// Signed "MULXHI".
// Based on the formula presented in OSA2011 §7.140, but with bitops to select
// the values to be added.
// TODO: This expansion should probably be moved to DAG legalization phase.
def : Pat<(i64 (mulhs i64:$lhs, i64:$rhs)),
      (SUBrr (UMULXHI $lhs, $rhs),
             (ADDrr (ANDrr (SRAXri $lhs, 63), $rhs),
                    (ANDrr (SRAXri $rhs, 63), $lhs)))>;

```
- **EN**: Implements logic around `Pat<`, `SUBrr`, `ADDrr`, `ANDrr`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `Pat<`, `SUBrr`, `ADDrr`, `ANDrr` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 320-329
```tablegen
def : Pat<(i64 (ctlz i64:$src)), (LZCNT $src)>;
def : Pat<(i64 (ctlz_zero_poison i64:$src)), (LZCNT $src)>;
// 32-bit LZCNT.
// The zero extension will leave us with 32 extra leading zeros,
// so we need to compensate for it.
// FIXME remove this when the codegen supports using 64-bit values directly
// in V8+ mode.
def : Pat<(i32 (ctlz i32:$src)), (ADDri (LZCNT (SRLri $src, 0)), (i32 -32))>;
def : Pat<(i32 (ctlz_zero_poison i32:$src)), (ADDri (LZCNT (SRLri $src, 0)), (i32 -32))>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 330-344
```tablegen
def : Pat<(i32 (bitconvert f32:$src)), (MOVSTOUW $src)>;
def : Pat<(i64 (zanyext (i32 (bitconvert f32:$src)))), (MOVSTOUW $src)>;
def : Pat<(i64 (sext (i32 (bitconvert f32:$src)))), (MOVSTOSW $src)>;
def : Pat<(f32 (bitconvert i32:$src)), (MOVWTOS $src)>;
def : Pat<(i64 (bitconvert f64:$src)), (MOVDTOX $src)>;
def : Pat<(f64 (bitconvert i64:$src)), (MOVXTOD $src)>;

// OP-then-neg FP operations.
// TODO handle equivalent patterns like `rs1*-rs2`.
def : Pat<(f32 (fneg (fadd f32:$rs1, f32:$rs2))), (FNADDS $rs1, $rs2)>;
def : Pat<(f64 (fneg (fadd f64:$rs1, f64:$rs2))), (FNADDD $rs1, $rs2)>;
def : Pat<(f32 (fneg (fmul f32:$rs1, f32:$rs2))), (FNMULS $rs1, $rs2)>;
def : Pat<(f64 (fneg (fmul f64:$rs1, f64:$rs2))), (FNMULD $rs1, $rs2)>;
def : Pat<(f64 (fneg (fmul (fpextend f32:$rs1), (fpextend f32:$rs2)))), (FNSMULD $rs1, $rs2)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 345-352
```tablegen
// Op-then-halve FP operations.
def : Pat<(f32 (fmul (fadd f32:$rs1, f32:$rs2), fpimmhalf)), (FHADDS $rs1, $rs2)>;
def : Pat<(f64 (fmul (fadd f64:$rs1, f64:$rs2), fpimmhalf)), (FHADDD $rs1, $rs2)>;
def : Pat<(f32 (fmul (fsub f32:$rs1, f32:$rs2), fpimmhalf)), (FHSUBS $rs1, $rs2)>;
def : Pat<(f64 (fmul (fsub f64:$rs1, f64:$rs2), fpimmhalf)), (FHSUBD $rs1, $rs2)>;
def : Pat<(f32 (fmul (fadd f32:$rs1, f32:$rs2), fpnegimmhalf)), (FNHADDS $rs1, $rs2)>;
def : Pat<(f64 (fmul (fadd f64:$rs1, f64:$rs2), fpnegimmhalf)), (FNHADDD $rs1, $rs2)>;
} // Predicates = [HasVIS3]
```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
