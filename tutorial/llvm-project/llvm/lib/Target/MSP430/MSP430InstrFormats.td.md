# MSP430InstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430InstrFormats.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines reusable instruction encoding formats in TableGen DSL.
  - **CN**: 使用 TableGen DSL 定义可复用的指令编码格式。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```tablegen
//===-- MSP430InstrFormats.td - MSP430 Instruction Formats -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 10-23
```tablegen
//  Describe MSP430 instructions format here
//

class SourceMode<bits<2> val> {
  bits<2> Value = val;
}

def SrcReg      : SourceMode<0>; // r
def SrcMem      : SourceMode<1>; // m
def SrcIndReg   : SourceMode<2>; // n
def SrcPostInc  : SourceMode<3>; // p
def SrcImm      : SourceMode<3>; // i
//  SrcCGImm    : SourceMode< >; // c

```
- **EN**: Declares TableGen records such as `SourceMode`, `SrcReg`, `SrcMem`, `SrcIndReg`, ...; this block describes instruction encoding bits.
- **CN**: 声明了 `SourceMode`, `SrcReg`, `SrcMem`, `SrcIndReg`, ... 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 24-34
```tablegen
class DestMode<bit val> {
  bit Value = val;
}

def DstReg      : DestMode<0>;   // r
def DstMem      : DestMode<1>;   // m

// Generic MSP430 Format
class MSP430Inst<dag outs, dag ins, int size, string asmstr> : Instruction {
  field bits<48> Inst;

```
- **EN**: Declares TableGen records such as `DestMode`, `DstReg`, `DstMem`, `MSP430Inst`; this block describes instruction encoding bits.
- **CN**: 声明了 `DestMode`, `DstReg`, `DstMem`, `MSP430Inst` 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 35-43
```tablegen
  let Namespace = "MSP430";

  dag OutOperandList = outs;
  dag InOperandList  = ins;

  let AsmString = asmstr;
  let Size = size;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 44-52
```tablegen
// MSP430 Double Operand (Format I) Instructions
class IForm<bits<4> opcode, DestMode ad, bit bw, SourceMode as, int size,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, size, asmstr> {
  let Pattern = pattern;

  bits<4> rs;
  bits<4> rd;

```
- **EN**: Declares TableGen records such as `IForm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `IForm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 53-65
```tablegen
  let Inst{15-12} = opcode;
  let Inst{11-8}  = rs;
  let Inst{7}     = ad.Value;
  let Inst{6}     = bw;
  let Inst{5-4}   = as.Value;
  let Inst{3-0}   = rd;
}

// 8 bit IForm instructions
class IForm8<bits<4> opcode, DestMode dest, SourceMode src, int size,
             dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm<opcode, dest, 1, src, size, outs, ins, asmstr, pattern>;

```
- **EN**: Declares TableGen records such as `IForm8`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `IForm8` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 66-80
```tablegen
class I8rr<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstReg, SrcReg, 2, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Alpha";
}

class I8ri<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstReg, SrcImm, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<16> imm;
  let Inst{31-16} = imm;
  let rs = 0b0000;
}

```
- **EN**: Declares TableGen records such as `I8rr`, `I8ri`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8rr`, `I8ri` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 81-89
```tablegen
class I8rc<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 2, asmstr> {
  let DecoderNamespace = "Beta";
  let Pattern = pattern;

  bits<6> imm;
  bits<4> rd;

```
- **EN**: Declares TableGen records such as `I8rc`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8rc` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 90-106
```tablegen
  let Inst{15-12} = opcode;
  let Inst{11-8}  = imm{3-0};
  let Inst{7}     = DstReg.Value;
  let Inst{6}     = 1;
  let Inst{5-4}   = imm{5-4};
  let Inst{3-0}   = rd;
}

class I8rm<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstReg, SrcMem, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<20> src;
  let rs = src{3-0};
  let Inst{31-16} = src{19-4};
}

```
- **EN**: Declares TableGen records such as `I8rm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8rm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 107-118
```tablegen
class I8rn<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstReg, SrcIndReg, 2, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
}

class I8rp<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstReg, SrcPostInc, 2, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
}

```
- **EN**: Declares TableGen records such as `I8rn`, `I8rp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8rn`, `I8rp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 119-127
```tablegen
class I8mr<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstMem, SrcReg, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Alpha";
  bits<20> dst;
  let rd = dst{3-0};
  let Inst{31-16} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I8mr`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8mr` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 128-139
```tablegen
class I8mi<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstMem, SrcImm, 6, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<16> imm;
  bits<20> dst;
  let rs = 0b0000;
  let Inst{31-16} = imm;
  let rd = dst{3-0};
  let Inst{47-32} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I8mi`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8mi` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 140-148
```tablegen
class I8mc<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 4, asmstr> {
  let DecoderNamespace = "Beta";
  let Pattern = pattern;

  bits<6> imm;
  bits<20> dst;

```
- **EN**: Declares TableGen records such as `I8mc`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8mc` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 149-157
```tablegen
  let Inst{31-16} = dst{19-4};
  let Inst{15-12} = opcode;
  let Inst{11-8}  = imm{3-0};
  let Inst{7}     = DstMem.Value;
  let Inst{6}     = 1;
  let Inst{5-4}   = imm{5-4};
  let Inst{3-0}   = dst{3-0};
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 158-169
```tablegen
class I8mm<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstMem, SrcMem, 6, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<20> src;
  bits<20> dst;
  let rs = src{3-0};
  let Inst{31-16} = src{19-4};
  let rd = dst{3-0};
  let Inst{47-32} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I8mm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8mm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 170-178
```tablegen
class I8mn<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstMem, SrcIndReg, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
  bits<20> dst;
  let rd = dst{3-0};
  let Inst{31-16} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I8mn`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8mn` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 179-187
```tablegen
class I8mp<bits<4> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm8<opcode, DstMem, SrcPostInc, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
  bits<20> dst;
  let rd = dst{3-0};
  let Inst{31-16} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I8mp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I8mp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 188-198
```tablegen
// 16 bit IForm instructions
class IForm16<bits<4> opcode, DestMode dest, SourceMode src, int size,
              dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm<opcode, dest, 0, src, size, outs, ins, asmstr, pattern>;

class I16rr<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstReg, SrcReg, 2, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Alpha";
}

```
- **EN**: Declares TableGen records such as `IForm16`, `I16rr`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `IForm16`, `I16rr` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 199-207
```tablegen
class I16ri<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstReg, SrcImm, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<16> imm;
  let Inst{31-16} = imm;
  let rs = 0b0000;
}

```
- **EN**: Declares TableGen records such as `I16ri`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16ri` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 208-216
```tablegen
class I16rc<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 2, asmstr> {
  let DecoderNamespace = "Beta";
  let Pattern = pattern;

  bits<6> imm;
  bits<4> rd;

```
- **EN**: Declares TableGen records such as `I16rc`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16rc` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 217-233
```tablegen
  let Inst{15-12} = opcode;
  let Inst{11-8}  = imm{3-0};
  let Inst{7}     = DstReg.Value;
  let Inst{6}     = 0;
  let Inst{5-4}   = imm{5-4};
  let Inst{3-0}   = rd;
}

class I16rm<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstReg, SrcMem, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<20> src;
  let rs = src{3-0};
  let Inst{31-16} = src{19-4};
}

```
- **EN**: Declares TableGen records such as `I16rm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16rm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 234-245
```tablegen
class I16rn<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstReg, SrcIndReg, 2, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
}

class I16rp<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstReg, SrcPostInc, 2, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
}

```
- **EN**: Declares TableGen records such as `I16rn`, `I16rp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16rn`, `I16rp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 246-254
```tablegen
class I16mr<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstMem, SrcReg, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Alpha";
  bits<20> dst;
  let rd = dst{3-0};
  let Inst{31-16} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I16mr`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16mr` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 255-266
```tablegen
class I16mi<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstMem, SrcImm, 6, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<16> imm;
  bits<20> dst;
  let Inst{31-16} = imm;
  let rs = 0b0000;
  let rd = dst{3-0};
  let Inst{47-32} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I16mi`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16mi` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 267-275
```tablegen
class I16mc<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 4, asmstr> {
  let DecoderNamespace = "Beta";
  let Pattern = pattern;

  bits<6> imm;
  bits<20> dst;

```
- **EN**: Declares TableGen records such as `I16mc`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16mc` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 276-284
```tablegen
  let Inst{31-16} = dst{19-4};
  let Inst{15-12} = opcode;
  let Inst{11-8}  = imm{3-0};
  let Inst{7}     = DstMem.Value;
  let Inst{6}     = 0;
  let Inst{5-4}   = imm{5-4};
  let Inst{3-0}   = dst{3-0};
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 285-296
```tablegen
class I16mm<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstMem, SrcMem, 6, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Gamma";
  bits<20> src;
  bits<20> dst;
  let rs = src{3-0};
  let Inst{31-16} = src{19-4};
  let rd = dst{3-0};
  let Inst{47-32} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I16mm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16mm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 297-305
```tablegen
class I16mn<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstMem, SrcIndReg, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
  bits<20> dst;
  let rd = dst{3-0};
  let Inst{31-16} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I16mn`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16mn` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 306-314
```tablegen
class I16mp<bits<4> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IForm16<opcode, DstMem, SrcPostInc, 4, outs, ins, asmstr, pattern> {
  let DecoderNamespace = "Delta";
  bits<20> dst;
  let rd = dst{3-0};
  let Inst{31-16} = dst{19-4};
}

```
- **EN**: Declares TableGen records such as `I16mp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `I16mp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 315-329
```tablegen
// MSP430 Single Operand (Format II) Instructions
class IIForm<bits<3> opcode, bit bw, SourceMode as, int size,
             dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, size, asmstr> {
  let Pattern = pattern;

  bits<4> rs;

  let Inst{15-10} = 0b000100;
  let Inst{9-7}   = opcode;
  let Inst{6}     = bw;
  let Inst{5-4}   = as.Value;
  let Inst{3-0}   = rs;
}

```
- **EN**: Declares TableGen records such as `IIForm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `IIForm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 330-338
```tablegen
// 8 bit IIForm instructions
class IIForm8<bits<3> opcode, SourceMode src, int size,
              dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm<opcode, 1, src, size, outs, ins, asmstr, pattern>;

class II8r<bits<3> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm8<opcode, SrcReg, 2, outs, ins, asmstr, pattern>;

```
- **EN**: Declares TableGen records such as `IIForm8`, `II8r`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `IIForm8`, `II8r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 339-354
```tablegen
class II8m<bits<3> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm8<opcode, SrcMem, 4, outs, ins, asmstr, pattern> {
  bits<20> src;
  let rs = src{3-0};
  let Inst{31-16} = src{19-4};
}

class II8i<bits<3> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm8<opcode, SrcImm, 4, outs, ins, asmstr, pattern> {
  bits<16> imm;
  let rs = 0b0000;
  let Inst{31-16} = imm;
}

```
- **EN**: Declares TableGen records such as `II8m`, `II8i`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `II8m`, `II8i` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 355-367
```tablegen
class II8c<bits<3> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 2, asmstr> {
  let Pattern = pattern;

  bits<6> imm;

  let Inst{15-10} = 0b000100;
  let Inst{9-7}   = opcode;
  let Inst{6}     = 1;
  let Inst{5-0}   = imm;
}

```
- **EN**: Declares TableGen records such as `II8c`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `II8c` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 368-380
```tablegen
class II8n<bits<3> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm8<opcode, SrcIndReg, 2, outs, ins, asmstr, pattern>;

class II8p<bits<3> opcode,
           dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm8<opcode, SrcPostInc, 2, outs, ins, asmstr, pattern>;

// 16 bit IIForm instructions
class IIForm16<bits<3> opcode, SourceMode src, int size,
               dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm<opcode, 0, src, size, outs, ins, asmstr, pattern>;

```
- **EN**: Declares TableGen records such as `II8n`, `II8p`, `IIForm16`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `II8n`, `II8p`, `IIForm16` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 381-392
```tablegen
class II16r<bits<3> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm16<opcode, SrcReg, 2, outs, ins, asmstr, pattern>;

class II16m<bits<3> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm16<opcode, SrcMem, 4, outs, ins, asmstr, pattern> {
  bits<20> src;
  let rs = src{3-0};
  let Inst{31-16} = src{19-4};
}

```
- **EN**: Declares TableGen records such as `II16r`, `II16m`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `II16r`, `II16m` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 393-405
```tablegen
class II16i<bits<3> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm16<opcode, SrcImm, 4, outs, ins, asmstr, pattern> {
  bits<16> imm;
  let rs = 0b0000;
  let Inst{31-16} = imm;
}

class II16c<bits<3> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 2, asmstr> {
  let Pattern = pattern;

```
- **EN**: Declares TableGen records such as `II16i`, `II16c`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `II16i`, `II16c` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 406-417
```tablegen
  bits<6> imm;

  let Inst{15-10} = 0b000100;
  let Inst{9-7}   = opcode;
  let Inst{6}     = 0;
  let Inst{5-0}   = imm;
}

class II16n<bits<3> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm16<opcode, SrcIndReg, 2, outs, ins, asmstr, pattern>;

```
- **EN**: Declares TableGen records such as `II16n`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `II16n` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 418-426
```tablegen
class II16p<bits<3> opcode,
            dag outs, dag ins, string asmstr, list<dag> pattern>
  : IIForm16<opcode, SrcPostInc, 2, outs, ins, asmstr, pattern>;

// MSP430 Conditional Jumps Instructions
class CJForm<dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 2, asmstr> {
  let Pattern = pattern;
  
```
- **EN**: Declares TableGen records such as `II16p`, `CJForm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `II16p`, `CJForm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 427-439
```tablegen
  bits<3> cond;
  bits<10> dst;

  let Inst{15-13} = 0b001;
  let Inst{12-10} = cond;
  let Inst{9-0} = dst;
}

// Pseudo instructions
class Pseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
  : MSP430Inst<outs, ins, 0, asmstr> {
  let Pattern = pattern;
}
```
- **EN**: Declares TableGen records such as `Pseudo`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `Pseudo` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
