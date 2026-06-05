# XtensaDSPInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaDSPInstrInfo.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines instruction records, encodings, and SelectionDAG patterns in TableGen DSL for this backend.
  - **CN**: 使用 TableGen DSL 定义该后端的指令记录、编码和 SelectionDAG 模式。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```tablegen
//===- XtensaDSPInstrInfo.td - Xtensa Target Description ---*- tablegen -*-===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 10-22
```tablegen
//
// This file describes the Xtensa DSP instructions in TableGen format.
//
//===----------------------------------------------------------------------===//

// Multiply
class UMUL_AA<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x07, (outs), (ins AR:$s, AR:$t),
             instrAsm#"\t$s, $t", []>, Requires<[HasMAC16]> {
  let r = 0;
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Declares TableGen records such as `UMUL_AA`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `UMUL_AA` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 23-34
```tablegen
def UMUL_AA_LL : UMUL_AA<0x00, "umul.aa.ll">;
def UMUL_AA_HL : UMUL_AA<0x01, "umul.aa.hl">;
def UMUL_AA_LH : UMUL_AA<0x02, "umul.aa.lh">;
def UMUL_AA_HH : UMUL_AA<0x03, "umul.aa.hh">;

class MUL_AA<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x07, (outs), (ins AR:$s, AR:$t),
             instrAsm#"\t$s, $t", []>, Requires<[HasMAC16]> {
  let r = 0;
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Declares TableGen records such as `UMUL_AA_LL`, `UMUL_AA_HL`, `UMUL_AA_LH`, `UMUL_AA_HH`, ...; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `UMUL_AA_LL`, `UMUL_AA_HL`, `UMUL_AA_LH`, `UMUL_AA_HH`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 35-44
```tablegen
def MUL_AA_LL : MUL_AA<0x04, "mul.aa.ll">;
def MUL_AA_HL : MUL_AA<0x05, "mul.aa.hl">;
def MUL_AA_LH : MUL_AA<0x06, "mul.aa.lh">;
def MUL_AA_HH : MUL_AA<0x07, "mul.aa.hh">;

class MUL_AD<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x03, (outs), (ins AR:$s, MR23:$y),
             instrAsm#"\t$s, $y", []>, Requires<[HasMAC16]> {
  bits<2> y;

```
- **EN**: Declares TableGen records such as `MUL_AA_LL`, `MUL_AA_HL`, `MUL_AA_LH`, `MUL_AA_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MUL_AA_LL`, `MUL_AA_HL`, `MUL_AA_LH`, `MUL_AA_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 45-56
```tablegen
  let r = 0;
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;
  let Defs = [M1, M2, ACCLO, ACCHI];
}

def MUL_AD_LL : MUL_AD<0x04, "mul.ad.ll">;
def MUL_AD_HL : MUL_AD<0x05, "mul.ad.hl">;
def MUL_AD_LH : MUL_AD<0x06, "mul.ad.lh">;
def MUL_AD_HH : MUL_AD<0x07, "mul.ad.hh">;

```
- **EN**: Declares TableGen records such as `MUL_AD_LL`, `MUL_AD_HL`, `MUL_AD_LH`, `MUL_AD_HH`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `MUL_AD_LL`, `MUL_AD_HL`, `MUL_AD_LH`, `MUL_AD_HH` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 57-68
```tablegen
class MUL_DA<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x06, (outs), (ins MR01:$x, AR:$t),
             instrAsm#"\t$x, $t", []>, Requires<[HasMAC16]> {
  bits<2> x;

  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = 0;
  let s = 0;
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Declares TableGen records such as `MUL_DA`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `MUL_DA` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 69-79
```tablegen
def MUL_DA_LL : MUL_DA<0x04, "mul.da.ll">;
def MUL_DA_HL : MUL_DA<0x05, "mul.da.hl">;
def MUL_DA_LH : MUL_DA<0x06, "mul.da.lh">;
def MUL_DA_HH : MUL_DA<0x07, "mul.da.hh">;

class MUL_DD<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x02, (outs), (ins MR01:$x, MR23:$y),
             instrAsm#"\t$x, $y", []>, Requires<[HasMAC16]> {
  bits<2> x;
  bits<2> y;

```
- **EN**: Declares TableGen records such as `MUL_DA_LL`, `MUL_DA_HL`, `MUL_DA_LH`, `MUL_DA_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MUL_DA_LL`, `MUL_DA_HL`, `MUL_DA_LH`, `MUL_DA_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 80-89
```tablegen
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = 0;
  let s = 0;
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 90-101
```tablegen
def MUL_DD_LL : MUL_DD<0x04, "mul.dd.ll">;
def MUL_DD_HL : MUL_DD<0x05, "mul.dd.hl">;
def MUL_DD_LH : MUL_DD<0x06, "mul.dd.lh">;
def MUL_DD_HH : MUL_DD<0x07, "mul.dd.hh">;

class MULA_AA<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x07, (outs), (ins AR:$s, AR:$t),
             instrAsm#"\t$s, $t", []>, Requires<[HasMAC16]> {
  let r = 0;
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Declares TableGen records such as `MUL_DD_LL`, `MUL_DD_HL`, `MUL_DD_LH`, `MUL_DD_HH`, ...; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `MUL_DD_LL`, `MUL_DD_HL`, `MUL_DD_LH`, `MUL_DD_HH`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 102-111
```tablegen
def MULA_AA_LL : MULA_AA<0x08, "mula.aa.ll">;
def MULA_AA_HL : MULA_AA<0x09, "mula.aa.hl">;
def MULA_AA_LH : MULA_AA<0x0A, "mula.aa.lh">;
def MULA_AA_HH : MULA_AA<0x0B, "mula.aa.hh">;

class MULA_AD<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x03, (outs), (ins AR:$s, MR23:$y),
             instrAsm#"\t$s, $y", []>, Requires<[HasMAC16]> {
  bits<2> y;

```
- **EN**: Declares TableGen records such as `MULA_AA_LL`, `MULA_AA_HL`, `MULA_AA_LH`, `MULA_AA_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULA_AA_LL`, `MULA_AA_HL`, `MULA_AA_LH`, `MULA_AA_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 112-120
```tablegen
  let r = 0;
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;

  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 121-130
```tablegen
def MULA_AD_LL : MULA_AD<0x08, "mula.ad.ll">;
def MULA_AD_HL : MULA_AD<0x09, "mula.ad.hl">;
def MULA_AD_LH : MULA_AD<0x0A, "mula.ad.lh">;
def MULA_AD_HH : MULA_AD<0x0B, "mula.ad.hh">;

class MULA_DA<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x06, (outs), (ins MR01:$x, AR:$t),
             instrAsm#"\t$x, $t", []>, Requires<[HasMAC16]> {
  bits<2> x;

```
- **EN**: Declares TableGen records such as `MULA_AD_LL`, `MULA_AD_HL`, `MULA_AD_LH`, `MULA_AD_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULA_AD_LL`, `MULA_AD_HL`, `MULA_AD_LH`, `MULA_AD_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 131-139
```tablegen
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = 0;
  let s = 0;

  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 140-150
```tablegen
def MULA_DA_LL : MULA_DA<0x08, "mula.da.ll">;
def MULA_DA_HL : MULA_DA<0x09, "mula.da.hl">;
def MULA_DA_LH : MULA_DA<0x0A, "mula.da.lh">;
def MULA_DA_HH : MULA_DA<0x0B, "mula.da.hh">;

class MULA_DD<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x02, (outs), (ins MR01:$x, MR23:$y),
             instrAsm#"\t$x, $y", []>, Requires<[HasMAC16]> {
  bits<2> x;
  bits<2> y;

```
- **EN**: Declares TableGen records such as `MULA_DA_LL`, `MULA_DA_HL`, `MULA_DA_LH`, `MULA_DA_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULA_DA_LL`, `MULA_DA_HL`, `MULA_DA_LH`, `MULA_DA_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 151-162
```tablegen
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = 0;
  let s = 0;
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;

  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 163-175
```tablegen
def MULA_DD_LL : MULA_DD<0x08, "mula.dd.ll">;
def MULA_DD_HL : MULA_DD<0x09, "mula.dd.hl">;
def MULA_DD_LH : MULA_DD<0x0A, "mula.dd.lh">;
def MULA_DD_HH : MULA_DD<0x0B, "mula.dd.hh">;

class MULS_AA<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x07, (outs), (ins AR:$s, AR:$t),
             instrAsm#"\t$s, $t", []>, Requires<[HasMAC16]> {
  let r = 0;
  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Declares TableGen records such as `MULA_DD_LL`, `MULA_DD_HL`, `MULA_DD_LH`, `MULA_DD_HH`, ...; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `MULA_DD_LL`, `MULA_DD_HL`, `MULA_DD_LH`, `MULA_DD_HH`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 176-185
```tablegen
def MULS_AA_LL : MULS_AA<0x0C, "muls.aa.ll">;
def MULS_AA_HL : MULS_AA<0x0D, "muls.aa.hl">;
def MULS_AA_LH : MULS_AA<0x0E, "muls.aa.lh">;
def MULS_AA_HH : MULS_AA<0x0F, "muls.aa.hh">;

class MULS_AD<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x03, (outs), (ins AR:$s, MR23:$y),
             instrAsm#"\t$s, $y", []>, Requires<[HasMAC16]> {
  bits<2> y;

```
- **EN**: Declares TableGen records such as `MULS_AA_LL`, `MULS_AA_HL`, `MULS_AA_LH`, `MULS_AA_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULS_AA_LL`, `MULS_AA_HL`, `MULS_AA_LH`, `MULS_AA_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 186-194
```tablegen
  let r = 0;
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;

  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 195-204
```tablegen
def MULS_AD_LL : MULS_AD<0x0C, "muls.ad.ll">;
def MULS_AD_HL : MULS_AD<0x0D, "muls.ad.hl">;
def MULS_AD_LH : MULS_AD<0x0E, "muls.ad.lh">;
def MULS_AD_HH : MULS_AD<0x0F, "muls.ad.hh">;

class MULS_DA<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x06, (outs), (ins MR01:$x, AR:$t),
             instrAsm#"\t$x, $t", []>, Requires<[HasMAC16]> {
  bits<2> x;

```
- **EN**: Declares TableGen records such as `MULS_AD_LL`, `MULS_AD_HL`, `MULS_AD_LH`, `MULS_AD_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULS_AD_LL`, `MULS_AD_HL`, `MULS_AD_LH`, `MULS_AD_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 205-213
```tablegen
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = 0;
  let s = 0;

  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 214-224
```tablegen
def MULS_DA_LL : MULS_DA<0x0C, "muls.da.ll">;
def MULS_DA_HL : MULS_DA<0x0D, "muls.da.hl">;
def MULS_DA_LH : MULS_DA<0x0E, "muls.da.lh">;
def MULS_DA_HH : MULS_DA<0x0F, "muls.da.hh">;

class MULS_DD<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x02, (outs), (ins MR01:$x, MR23:$y),
             instrAsm#"\t$x, $y", []>, Requires<[HasMAC16]> {
  bits<2> x;
  bits<2> y;

```
- **EN**: Declares TableGen records such as `MULS_DA_LL`, `MULS_DA_HL`, `MULS_DA_LH`, `MULS_DA_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULS_DA_LL`, `MULS_DA_HL`, `MULS_DA_LH`, `MULS_DA_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 225-236
```tablegen
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = 0;
  let s = 0;
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;

  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 237-250
```tablegen
def MULS_DD_LL : MULS_DD<0x0C, "muls.dd.ll">;
def MULS_DD_HL : MULS_DD<0x0D, "muls.dd.hl">;
def MULS_DD_LH : MULS_DD<0x0E, "muls.dd.lh">;
def MULS_DD_HH : MULS_DD<0x0F, "muls.dd.hh">;

//===----------------------------------------------------------------------===//
// Multiply-accumulate with load

class MULA_DA_LDDEC<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x05, (outs MR:$w, AR:$d), (ins AR:$s, MR01:$x, AR:$t),
             instrAsm#"\t $w, $s, $x, $t", []>, Requires<[HasMAC16]> {
  bits<2> x;
  bits<2> w;

```
- **EN**: Declares TableGen records such as `MULS_DD_LL`, `MULS_DD_HL`, `MULS_DD_LH`, `MULS_DD_HH`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULS_DD_LL`, `MULS_DD_HL`, `MULS_DD_LH`, `MULS_DD_HH`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 251-259
```tablegen
  let Constraints = "$s = $d";
  let mayLoad = 1;
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = w{1-0};
  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 260-270
```tablegen
def MULA_DA_LL_LDDEC : MULA_DA_LDDEC<0x08, "mula.da.ll.lddec">;
def MULA_DA_HL_LDDEC : MULA_DA_LDDEC<0x09, "mula.da.hl.lddec">;
def MULA_DA_LH_LDDEC : MULA_DA_LDDEC<0x0A, "mula.da.lh.lddec">;
def MULA_DA_HH_LDDEC : MULA_DA_LDDEC<0x0B, "mula.da.hh.lddec">;

class MULA_DA_LDINC<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x04, (outs MR:$w, AR:$d), (ins AR:$s, MR:$x, AR:$t),
             instrAsm#"\t $w, $s, $x, $t", []>, Requires<[HasMAC16]> {
  bits<1> x;
  bits<2> w;

```
- **EN**: Declares TableGen records such as `MULA_DA_LL_LDDEC`, `MULA_DA_HL_LDDEC`, `MULA_DA_LH_LDDEC`, `MULA_DA_HH_LDDEC`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULA_DA_LL_LDDEC`, `MULA_DA_HL_LDDEC`, `MULA_DA_LH_LDDEC`, `MULA_DA_HH_LDDEC`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 271-279
```tablegen
  let Constraints = "$s = $d";
  let mayLoad = 1;
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = w{1-0};
  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 280-291
```tablegen
def MULA_DA_LL_LDINC: MULA_DA_LDINC<0x08, "mula.da.ll.ldinc">;
def MULA_DA_HL_LDINC: MULA_DA_LDINC<0x09, "mula.da.hl.ldinc">;
def MULA_DA_LH_LDINC: MULA_DA_LDINC<0x0A, "mula.da.lh.ldinc">;
def MULA_DA_HH_LDINC: MULA_DA_LDINC<0x0B, "mula.da.hh.ldinc">;

class MULA_DD_LDDEC<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x01, (outs MR:$w, AR:$d), (ins AR:$s, MR01:$x, MR23:$y),
             instrAsm#"\t $w, $s, $x, $y", []>, Requires<[HasMAC16]> {
  bits<2> x;
  bits<2> y;
  bits<2> w;

```
- **EN**: Declares TableGen records such as `MULA_DA_LL_LDINC`, `MULA_DA_HL_LDINC`, `MULA_DA_LH_LDINC`, `MULA_DA_HH_LDINC`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULA_DA_LL_LDINC`, `MULA_DA_HL_LDINC`, `MULA_DA_LH_LDINC`, `MULA_DA_HH_LDINC`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 292-303
```tablegen
  let Constraints = "$s = $d";
  let mayLoad = 1;
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = w{1-0};
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;
  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 304-315
```tablegen
def MULA_DD_LL_LDDEC : MULA_DD_LDDEC<0x08, "mula.dd.ll.lddec">;
def MULA_DD_HL_LDDEC : MULA_DD_LDDEC<0x09, "mula.dd.hl.lddec">;
def MULA_DD_LH_LDDEC : MULA_DD_LDDEC<0x0A, "mula.dd.lh.lddec">;
def MULA_DD_HH_LDDEC : MULA_DD_LDDEC<0x0B, "mula.dd.hh.lddec">;

class MULA_DD_LDINC<bits<4> oper1, string instrAsm>
  : RRR_Inst<0x04, oper1, 0x00, (outs MR:$w, AR:$d), (ins AR:$s, MR01:$x, MR23:$y),
             instrAsm#"\t $w, $s, $x, $y", []>, Requires<[HasMAC16]> {
  bits<2> x;
  bits<2> y;
  bits<2> w;

```
- **EN**: Declares TableGen records such as `MULA_DD_LL_LDDEC`, `MULA_DD_HL_LDDEC`, `MULA_DD_LH_LDDEC`, `MULA_DD_HH_LDDEC`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULA_DD_LL_LDDEC`, `MULA_DD_HL_LDDEC`, `MULA_DD_LH_LDDEC`, `MULA_DD_HH_LDDEC`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 316-327
```tablegen
  let Constraints = "$s = $d";
  let mayLoad = 1;
  let r{3} = 0;
  let r{2} = x{0};
  let r{1-0} = w{1-0};
  let t{3} = 0;
  let t{2} = y{0};
  let t{1-0} = 0;
  let Uses = [ACCLO, ACCHI];
  let Defs = [M1, M2, ACCLO, ACCHI];
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 328-336
```tablegen
def MULA_DD_LL_LDINC : MULA_DD_LDINC<0x08, "mula.dd.ll.ldinc">;
def MULA_DD_HL_LDINC : MULA_DD_LDINC<0x09, "mula.dd.hl.ldinc">;
def MULA_DD_LH_LDINC : MULA_DD_LDINC<0x0A, "mula.dd.lh.ldinc">;
def MULA_DD_HH_LDINC : MULA_DD_LDINC<0x0B, "mula.dd.hh.ldinc">;

def LDDEC : RRR_Inst<0x04, 0x00, 0x09, (outs MR:$w, AR:$d), (ins AR:$s),
                    "lddec\t $w, $s", []>, Requires<[HasMAC16]> {
  bits<2> w;

```
- **EN**: Declares TableGen records such as `MULA_DD_LL_LDINC`, `MULA_DD_HL_LDINC`, `MULA_DD_LH_LDINC`, `MULA_DD_HH_LDINC`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `MULA_DD_LL_LDINC`, `MULA_DD_HL_LDINC`, `MULA_DD_LH_LDINC`, `MULA_DD_HH_LDINC`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 337-347
```tablegen
  let Constraints = "$s = $d";
  let mayLoad = 1;
  let r{3-2} = 0;
  let r{1-0} = w{1-0};
  let t = 0x00;
}

def LDINC : RRR_Inst<0x04, 0x00, 0x08, (outs MR:$w, AR:$d), (ins AR:$s),
                    "ldinc\t $w, $s", []>, Requires<[HasMAC16]> {
  bits<2> w;

```
- **EN**: Declares TableGen records such as `LDINC`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `LDINC` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 348-353
```tablegen
  let Constraints = "$s = $d";
  let mayLoad = 1;
  let r{3-2} = 0;
  let r{1-0} = w{1-0};
  let t = 0;
}
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
