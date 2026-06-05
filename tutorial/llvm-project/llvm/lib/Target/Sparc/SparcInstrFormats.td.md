# SparcInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcInstrFormats.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines reusable instruction encoding formats in TableGen DSL.
  - **CN**: 使用 TableGen DSL 定义可复用的指令编码格式。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```tablegen
//===-- SparcInstrFormats.td - Sparc Instruction Formats ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

class InstSP<dag outs, dag ins, string asmstr, list<dag> pattern,
             InstrItinClass itin = NoItinerary>
   : Instruction {
  field bits<32> Inst;

```
- **EN**: Declares TableGen records such as `InstSP`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `InstSP` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 14-24
```tablegen
  let Namespace = "SP";
  let Size = 4;

  bits<2> op;
  let Inst{31-30} = op;               // Top two bits are the 'op' field

  dag OutOperandList = outs;
  dag InOperandList = ins;
  let AsmString   = asmstr;
  let Pattern = pattern;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 25-33
```tablegen
  let DecoderNamespace = "Sparc";

  let Itinerary = itin;
}

//===----------------------------------------------------------------------===//
// Format #2 instruction classes in the Sparc
//===----------------------------------------------------------------------===//

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 34-44
```tablegen
// Format 2 instructions
class F2<dag outs, dag ins, string asmstr, list<dag> pattern,
         InstrItinClass itin = NoItinerary>
   : InstSP<outs, ins, asmstr, pattern, itin> {
  bits<3>  op2;
  bits<22> imm22;
  let op          = 0;    // op = 0
  let Inst{24-22} = op2;
  let Inst{21-0}  = imm22;
}

```
- **EN**: Declares TableGen records such as `F2`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F2` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 45-53
```tablegen
// Specific F2 classes: SparcV8 manual, page 44
//
class F2_1<bits<3> op2Val, dag outs, dag ins, string asmstr, list<dag> pattern,
           InstrItinClass itin = NoItinerary>
   : F2<outs, ins, asmstr, pattern, itin> {
  bits<5>  rd;

  let op2         = op2Val;

```
- **EN**: Declares TableGen records such as `F2_1`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F2_1` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 54-62
```tablegen
  let Inst{29-25} = rd;
}

class F2_2<bits<3> op2Val, bit annul, dag outs, dag ins, string asmstr,
           list<dag> pattern, InstrItinClass itin = NoItinerary>
   : F2<outs, ins, asmstr, pattern, itin> {
  bits<4>   cond;
  let op2         = op2Val;

```
- **EN**: Declares TableGen records such as `F2_2`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F2_2` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 63-74
```tablegen
  let Inst{29}    = annul;
  let Inst{28-25} = cond;
}

class F2_3<bits<3> op2Val, bit annul, bit pred,
           dag outs, dag ins, string asmstr, list<dag> pattern,
           InstrItinClass itin = NoItinerary>
   : InstSP<outs, ins, asmstr, pattern, itin> {
  bits<2>  cc;
  bits<4>  cond;
  bits<19> imm19;

```
- **EN**: Declares TableGen records such as `F2_3`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F2_3` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 75-84
```tablegen
  let op          = 0;    // op = 0

  let Inst{29}    = annul;
  let Inst{28-25} = cond;
  let Inst{24-22} = op2Val;
  let Inst{21-20} = cc;
  let Inst{19}    = pred;
  let Inst{18-0}  = imm19;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 85-93
```tablegen
class F2_4<bit annul, bit pred, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
   : InstSP<outs, ins, asmstr, pattern, itin> {
  bits<16> imm16;
  bits<5>  rs1;
  bits<3>  rcond;

  let op          = 0;    // op = 0

```
- **EN**: Declares TableGen records such as `F2_4`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F2_4` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 94-103
```tablegen
  let Inst{29}    = annul;
  let Inst{28}    = 0;
  let Inst{27-25} = rcond;
  let Inst{24-22} = 0b011;
  let Inst{21-20} = imm16{15-14};
  let Inst{19}    = pred;
  let Inst{18-14} = rs1;
  let Inst{13-0}  = imm16{13-0};
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 104-113
```tablegen
class F2_5<bit cc, dag outs, dag ins, string asmstr,
           list<dag> pattern = [], InstrItinClass itin = NoItinerary>
    : InstSP<outs, ins, asmstr, pattern, itin> {
  bits<10> imm10;
  bits<5>  rs1;
  bits<5>  rs2;
  bits<4>  cond;

  let op          = 0; // op = 0

```
- **EN**: Declares TableGen records such as `F2_5`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F2_5` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 114-125
```tablegen
  let Inst{29}    = cond{3};
  let Inst{28}    = 1;
  let Inst{27-25} = cond{2-0};
  let Inst{24-22} = 0b011;
  let Inst{21}    = cc;
  let Inst{20-19} = imm10{9-8};
  let Inst{18-14} = rs1;
  let Inst{13}    = 0; // i = 0
  let Inst{12-5}  = imm10{7-0};
  let Inst{4-0}   = rs2;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 126-135
```tablegen
class F2_6<bit cc, dag outs, dag ins, string asmstr,
           list<dag> pattern = [], InstrItinClass itin = NoItinerary>
    : InstSP<outs, ins, asmstr, pattern, itin> {
  bits<10> imm10;
  bits<5>  rs1;
  bits<5>  simm5;
  bits<4>  cond;

  let op          = 0; // op = 0

```
- **EN**: Declares TableGen records such as `F2_6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F2_6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 136-147
```tablegen
  let Inst{29}    = cond{3};
  let Inst{28}    = 1;
  let Inst{27-25} = cond{2-0};
  let Inst{24-22} = 0b011;
  let Inst{21}    = cc;
  let Inst{20-19} = imm10{9-8};
  let Inst{18-14} = rs1;
  let Inst{13}    = 1; // i = 1
  let Inst{12-5}  = imm10{7-0};
  let Inst{4-0}   = simm5;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 148-163
```tablegen
//===----------------------------------------------------------------------===//
// Format #3 instruction classes in the Sparc
//===----------------------------------------------------------------------===//

class F3<dag outs, dag ins, string asmstr, list<dag> pattern,
         InstrItinClass itin = NoItinerary>
   : InstSP<outs, ins, asmstr, pattern, itin> {
  bits<5> rd;
  bits<6> op3;
  bits<5> rs1;
  let op{1} = 1;   // Op = 2 or 3
  let Inst{29-25} = rd;
  let Inst{24-19} = op3;
  let Inst{18-14} = rs1;
}

```
- **EN**: Declares TableGen records such as `F3`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 164-174
```tablegen
// Specific F3 classes: SparcV8 manual, page 44
//
class F3_1_asi<bits<2> opVal, bits<6> op3val, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
   : F3<outs, ins, asmstr, pattern, itin> {
  bits<8> asi;
  bits<5> rs2;

  let op         = opVal;
  let op3        = op3val;

```
- **EN**: Declares TableGen records such as `F3_1_asi`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_1_asi` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 175-187
```tablegen
  let Inst{13}   = 0;     // i field = 0
  let Inst{12-5} = asi;   // address space identifier
  let Inst{4-0}  = rs2;
}

// CAS instructions does not use an immediate even when i=1
class F3_1_cas_asi<bits<2> opVal, bits<6> op3val, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
   : F3_1_asi<opVal, op3val, outs, ins, asmstr, pattern, itin> {
  let asi = 0;
  let Inst{13}   = 1;     // i field = 1
}

```
- **EN**: Declares TableGen records such as `F3_1_cas_asi`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_1_cas_asi` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 188-198
```tablegen
class F3_1<bits<2> opVal, bits<6> op3val, dag outs, dag ins, string asmstr,
       list<dag> pattern, InstrItinClass itin = IIC_iu_instr>
  : F3_1_asi<opVal, op3val, outs, ins, asmstr, pattern, itin> {
  let asi = 0;
}

class F3_2<bits<2> opVal, bits<6> op3val, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = IIC_iu_instr>
   : F3<outs, ins, asmstr, pattern, itin> {
  bits<13> simm13;

```
- **EN**: Declares TableGen records such as `F3_1`, `F3_2`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_1`, `F3_2` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 199-211
```tablegen
  let op         = opVal;
  let op3        = op3val;

  let Inst{13}   = 1;     // i field = 1
  let Inst{12-0} = simm13;
}

// floating-point
class F3_3<bits<2> opVal, bits<6> op3val, bits<9> opfval, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
   : F3<outs, ins, asmstr, pattern, itin> {
  bits<5> rs2;

```
- **EN**: Declares TableGen records such as `F3_3`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_3` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 212-224
```tablegen
  let op         = opVal;
  let op3        = op3val;

  let Inst{13-5} = opfval;   // fp opcode
  let Inst{4-0}  = rs2;
}

// floating-point unary operations.
class F3_3u<bits<2> opVal, bits<6> op3val, bits<9> opfval, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
   : F3<outs, ins, asmstr, pattern, itin> {
  bits<5> rs2;

```
- **EN**: Declares TableGen records such as `F3_3u`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_3u` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 225-238
```tablegen
  let op         = opVal;
  let op3        = op3val;
  let rs1        = 0;

  let Inst{13-5} = opfval;   // fp opcode
  let Inst{4-0}  = rs2;
}

// floating-point compares.
class F3_3c<bits<2> opVal, bits<6> op3val, bits<9> opfval, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
   : F3<outs, ins, asmstr, pattern, itin> {
  bits<5> rs2;

```
- **EN**: Declares TableGen records such as `F3_3c`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_3c` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 239-251
```tablegen
  let op         = opVal;
  let op3        = op3val;

  let Inst{13-5} = opfval;   // fp opcode
  let Inst{4-0}  = rs2;
}

// SIAM instruction
class F3_3_siam<bits<2> opVal, bits<6> op3val, bits<9> opfval, dag outs, dag ins,
           string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
   : F3<outs, ins, asmstr, pattern, itin> {
  bits<3> mode;

```
- **EN**: Declares TableGen records such as `F3_3_siam`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_3_siam` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 252-260
```tablegen
  let op         = opVal;
  let op3        = op3val;
  let rd         = 0;
  let rs1        = 0;
  let Inst{13-5} = opfval;   // fp opcode
  let Inst{4-3}  = 0;
  let Inst{2-0}  = mode;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 261-270
```tablegen
// Shift by register rs2.
class F3_Sr<bits<2> opVal, bits<6> op3val, bit xVal, dag outs, dag ins,
            string asmstr, list<dag> pattern, InstrItinClass itin = IIC_iu_instr>
   : F3<outs, ins, asmstr, pattern, itin> {
  bit x = xVal;           // 1 for 64-bit shifts.
  bits<5> rs2;

  let op         = opVal;
  let op3        = op3val;

```
- **EN**: Declares TableGen records such as `F3_Sr`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_Sr` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 271-282
```tablegen
  let Inst{13}   = 0;     // i field = 0
  let Inst{12}   = x;     // extended registers.
  let Inst{4-0}  = rs2;
}

// Shift by immediate.
class F3_Si<bits<2> opVal, bits<6> op3val, bit xVal, dag outs, dag ins,
            string asmstr, list<dag> pattern, InstrItinClass itin = IIC_iu_instr>
   : F3<outs, ins, asmstr, pattern, itin> {
  bit x = xVal;           // 1 for 64-bit shifts.
  bits<6> shcnt;          // shcnt32 / shcnt64.

```
- **EN**: Declares TableGen records such as `F3_Si`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_Si` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 283-300
```tablegen
  let op         = opVal;
  let op3        = op3val;

  let Inst{13}   = 1;     // i field = 1
  let Inst{12}   = x;     // extended registers.
  let Inst{5-0}  = shcnt;
}

// Define rr and ri shift instructions with patterns.
multiclass F3_S<string OpcStr, bits<6> Op3Val, bit XVal, SDNode OpNode,
                ValueType VT, Operand SIT, RegisterClass RC,
                InstrItinClass itin = IIC_iu_instr> {
  def rr : F3_Sr<2, Op3Val, XVal, (outs RC:$rd), (ins RC:$rs1, IntRegs:$rs2),
                 !strconcat(OpcStr, " $rs1, $rs2, $rd"),
                 [(set VT:$rd, (OpNode VT:$rs1, i32:$rs2))],
                 itin>;
  def ri : F3_Si<2, Op3Val, XVal, (outs RC:$rd), (ins RC:$rs1, SIT:$shcnt),
                 !strconcat(OpcStr, " $rs1, $shcnt, $rd"),
```
- **EN**: Declares TableGen records such as `F3_S`, `rr`, `ri`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F3_S`, `rr`, `ri` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 301-312
```tablegen
                 [(set VT:$rd, (OpNode VT:$rs1, (i32 imm:$shcnt)))],
                 itin>;
}

// 4-operand instructions.
class F3_4<bits<6> op3val, bits<4> op5val, dag outs, dag ins,
           string asmstr, list<dag> pattern = [], InstrItinClass itin = NoItinerary>
   : F3<outs, ins, asmstr, pattern, itin> {
  bits<4> op5;
  bits<5> rs3;
  bits<5> rs2;

```
- **EN**: Declares TableGen records such as `F3_4`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `F3_4` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 313-321
```tablegen
  let op         = 2;
  let op3        = op3val;
  let op5        = op5val;

  let Inst{13-9} = rs3;
  let Inst{8-5}  = op5;
  let Inst{4-0}  = rs2;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 322-331
```tablegen
class F4<bits<6> op3, dag outs, dag ins, string asmstr, list<dag> pattern,
         InstrItinClass itin = NoItinerary>
   : InstSP<outs, ins, asmstr, pattern, itin> {
  bits<5> rd;

  let op          = 2;
  let Inst{29-25} = rd;
  let Inst{24-19} = op3;
}

```
- **EN**: Declares TableGen records such as `F4`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F4` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 332-341
```tablegen

class F4_1<bits<6> op3, dag outs, dag ins,
           string asmstr, list<dag> pattern,
           InstrItinClass itin = NoItinerary>
   : F4<op3, outs, ins, asmstr, pattern, itin> {
  bit    intcc;
  bits<2> cc;
  bits<4> cond;
  bits<5> rs2;

```
- **EN**: Declares TableGen records such as `F4_1`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `F4_1` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 342-357
```tablegen
  let Inst{4-0}   = rs2;
  let Inst{12-11} = cc;
  let Inst{13}    = 0;
  let Inst{17-14} = cond;
  let Inst{18}    = intcc;
}

class F4_2<bits<6> op3, dag outs, dag ins,
            string asmstr, list<dag> pattern,
            InstrItinClass itin = NoItinerary>
   : F4<op3, outs, ins, asmstr, pattern, itin> {
  bit      intcc;
  bits<2>  cc;
  bits<4>  cond;
  bits<11> simm11;

```
- **EN**: Declares TableGen records such as `F4_2`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F4_2` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 358-373
```tablegen
  let Inst{10-0}  = simm11;
  let Inst{12-11} = cc;
  let Inst{13}    = 1;
  let Inst{17-14} = cond;
  let Inst{18}    = intcc;
}

class F4_3<bits<6> op3, bits<6> opf_low, dag outs, dag ins,
           string asmstr, list<dag> pattern,
           InstrItinClass itin = NoItinerary>
   : F4<op3, outs, ins, asmstr, pattern, itin> {
  bits<4> cond;
  bit     intcc;
  bits<2> opf_cc;
  bits<5> rs2;

```
- **EN**: Declares TableGen records such as `F4_3`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F4_3` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 374-391
```tablegen
  let Inst{18}     = 0;
  let Inst{17-14}  = cond;
  let Inst{13}     = intcc;
  let Inst{12-11}  = opf_cc;
  let Inst{10-5}   = opf_low;
  let Inst{4-0}    = rs2;
}

class F4_4r<bits<6> op3, bits<5> opf_low, dag outs, dag ins,
            string asmstr, list<dag> pattern,
            InstrItinClass itin = NoItinerary>
   : F4<op3, outs, ins, asmstr, pattern, itin> {
  bits<5> rs1;
  bits<5> rs2;
  bits<3> rcond;
  let Inst{18-14} = rs1;
  let Inst{13}    = 0;  // IsImm
  let Inst{12-10} = rcond;
```
- **EN**: Declares TableGen records such as `F4_4r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F4_4r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 392-409
```tablegen
  let Inst{9-5}   = opf_low;
  let Inst{4-0}   = rs2;
}


class F4_4i<bits<6> op3, dag outs, dag ins,
            string asmstr, list<dag> pattern,
           InstrItinClass itin = NoItinerary>
   : F4<op3, outs, ins, asmstr, pattern, itin> {
  bits<5>  rs1;
  bits<10> simm10;
  bits<3>  rcond;
  let Inst{18-14} = rs1;
  let Inst{13}    = 1;  // IsImm
  let Inst{12-10} = rcond;
  let Inst{9-0}   = simm10;
}

```
- **EN**: Declares TableGen records such as `F4_4i`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `F4_4i` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 410-424
```tablegen

class TRAPSP<bits<6> op3Val, bit isimm, dag outs, dag ins,
             string asmstr, list<dag> pattern,
             InstrItinClass itin = NoItinerary>
   : F3<outs, ins, asmstr, pattern, itin> {
   bits<4> cond;
   bits<2> cc;

   let op = 0b10;
   let rd{4} = 0;
   let rd{3-0} = cond;
   let op3 = op3Val;
   let Inst{13} = isimm;
   let Inst{12-11} = cc;

```
- **EN**: Declares TableGen records such as `TRAPSP`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `TRAPSP` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 425-436
```tablegen
}

class TRAPSPrr<bits<6> op3Val, dag outs, dag ins,
               string asmstr, list<dag> pattern,
               InstrItinClass itin = NoItinerary>
   : TRAPSP<op3Val, 0, outs, ins, asmstr, pattern, itin> {
   bits<5> rs2;

   let Inst{10-5} = 0;
   let Inst{4-0}  = rs2;
}

```
- **EN**: Declares TableGen records such as `TRAPSPrr`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `TRAPSPrr` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 437-446
```tablegen
class TRAPSPri<bits<6> op3Val, dag outs, dag ins,
               string asmstr, list<dag> pattern,
               InstrItinClass itin = NoItinerary>
   : TRAPSP<op3Val, 1, outs, ins, asmstr, pattern, itin> {
   bits<8> imm;

   let Inst{10-8} = 0;
   let Inst{7-0}  = imm;
}

```
- **EN**: Declares TableGen records such as `TRAPSPri`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `TRAPSPri` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 447-453
```tablegen
// Pseudo-instructions for alternate assembly syntax (never used by codegen).
// These are aliases that require C++ handling to convert to the target
// instruction, while InstAliases can be handled directly by tblgen.
class AsmPseudoInst<dag outs, dag ins, string asm>
  : InstSP<outs, ins, asm, []> {
  let isPseudo = 1;
}
```
- **EN**: Declares TableGen records such as `AsmPseudoInst`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `AsmPseudoInst` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
