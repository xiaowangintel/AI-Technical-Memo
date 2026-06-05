# MipsRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsRegisterInfo.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `MipsRegisterInfo` in LLVM TableGen DSL for the Mips backend, covering register definitions, allocation constraints, and register utilities.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsRegisterInfo`，涵盖寄存器定义、分配约束以及寄存器工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- MipsRegisterInfo.td - Mips Register defs -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-22
```tablegen
//===----------------------------------------------------------------------===//
//  Declarations that describe the MIPS register file
//===----------------------------------------------------------------------===//
let Namespace = "Mips" in {
def sub_32     : SubRegIndex<32>;
def sub_64     : SubRegIndex<64>;
def sub_lo     : SubRegIndex<32>;
def sub_hi     : SubRegIndex<32, 32>;
def sub_dsp16_19 : SubRegIndex<4, 16>;
def sub_dsp20    : SubRegIndex<1, 20>;
def sub_dsp21    : SubRegIndex<1, 21>;
def sub_dsp22    : SubRegIndex<1, 22>;
def sub_dsp23    : SubRegIndex<1, 23>;
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 24-26
```tablegen
class Unallocatable {
  bit isAllocatable = 0;
}
```
- EN: Declares reusable TableGen class `Unallocatable` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `Unallocatable`，通常用于抽象共享字段、谓词或编码结构。

### Lines 28-32
```tablegen
// We have banks of 32 registers each.
class MipsReg<bits<16> Enc, string n> : Register<n> {
  let HWEncoding = Enc;
  let Namespace = "Mips";
}
```
- EN: Declares reusable TableGen class `MipsReg` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `MipsReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 34-38
```tablegen
class MipsRegWithSubRegs<bits<16> Enc, string n, list<Register> subregs>
  : RegisterWithSubRegs<n, subregs> {
  let HWEncoding = Enc;
  let Namespace = "Mips";
}
```
- EN: Declares reusable TableGen class `MipsRegWithSubRegs` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `MipsRegWithSubRegs`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-41
```tablegen
// Mips CPU Registers.
class MipsGPRReg<bits<16> Enc, string n> : MipsReg<Enc, n>;
```
- EN: Declares reusable TableGen class `MipsGPRReg` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `MipsGPRReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 43-47
```tablegen
// Mips 64-bit CPU Registers
class Mips64GPRReg<bits<16> Enc, string n, list<Register> subregs>
  : MipsRegWithSubRegs<Enc, n, subregs> {
  let SubRegIndices = [sub_32];
}
```
- EN: Declares reusable TableGen class `Mips64GPRReg` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `Mips64GPRReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 49-50
```tablegen
// Mips 32-bit FPU Registers
class FPR<bits<16> Enc, string n> : MipsReg<Enc, n>;
```
- EN: Declares reusable TableGen class `FPR` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `FPR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 52-57
```tablegen
// Mips 64-bit (aliased) FPU Registers
class AFPR<bits<16> Enc, string n, list<Register> subregs>
  : MipsRegWithSubRegs<Enc, n, subregs> {
  let SubRegIndices = [sub_lo, sub_hi];
  let CoveredBySubRegs = 1;
}
```
- EN: Declares reusable TableGen class `AFPR` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `AFPR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 59-63
```tablegen
class AFPR64<bits<16> Enc, string n, list<Register> subregs>
  : MipsRegWithSubRegs<Enc, n, subregs> {
  let SubRegIndices = [sub_lo, sub_hi];
  let CoveredBySubRegs = 1;
}
```
- EN: Declares reusable TableGen class `AFPR64` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `AFPR64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 65-69
```tablegen
// Mips 128-bit (aliased) MSA Registers
class AFPR128<bits<16> Enc, string n, list<Register> subregs>
  : MipsRegWithSubRegs<Enc, n, subregs> {
  let SubRegIndices = [sub_64];
}
```
- EN: Declares reusable TableGen class `AFPR128` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `AFPR128`，通常用于抽象共享字段、谓词或编码结构。

### Lines 71-76
```tablegen
// Accumulator Registers
class ACCReg<bits<16> Enc, string n, list<Register> subregs>
  : MipsRegWithSubRegs<Enc, n, subregs> {
  let SubRegIndices = [sub_lo, sub_hi];
  let CoveredBySubRegs = 1;
}
```
- EN: Declares reusable TableGen class `ACCReg` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `ACCReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 78-79
```tablegen
// Mips Hardware Registers
class HWR<bits<16> Enc, string n> : MipsReg<Enc, n>;
```
- EN: Declares reusable TableGen class `HWR` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `HWR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 81-83
```tablegen
//===----------------------------------------------------------------------===//
//  Registers
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 85-102
```tablegen
let Namespace = "Mips" in {
  // General Purpose Registers
  let isConstant = true in
  def ZERO : MipsGPRReg< 0, "zero">, DwarfRegNum<[0]>;
  def AT   : MipsGPRReg< 1, "1">,    DwarfRegNum<[1]>;
  def V0   : MipsGPRReg< 2, "2">,    DwarfRegNum<[2]>;
  def V1   : MipsGPRReg< 3, "3">,    DwarfRegNum<[3]>;
  def A0   : MipsGPRReg< 4, "4">,    DwarfRegNum<[4]>;
  def A1   : MipsGPRReg< 5, "5">,    DwarfRegNum<[5]>;
  def A2   : MipsGPRReg< 6, "6">,    DwarfRegNum<[6]>;
  def A3   : MipsGPRReg< 7, "7">,    DwarfRegNum<[7]>;
  def T0   : MipsGPRReg< 8, "8">,    DwarfRegNum<[8]>;
  def T1   : MipsGPRReg< 9, "9">,    DwarfRegNum<[9]>;
  def T2   : MipsGPRReg< 10, "10">,  DwarfRegNum<[10]>;
  def T3   : MipsGPRReg< 11, "11">,  DwarfRegNum<[11]>;
  def T4   : MipsGPRReg< 12, "12">,  DwarfRegNum<[12]>;
  def T5   : MipsGPRReg< 13, "13">,  DwarfRegNum<[13]>;
  def T6   : MipsGPRReg< 14, "14">,  DwarfRegNum<[14]>;
```
- EN: Defines TableGen record `ZERO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ZERO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 103-119
```tablegen
  def T7   : MipsGPRReg< 15, "15">,  DwarfRegNum<[15]>;
  def S0   : MipsGPRReg< 16, "16">,  DwarfRegNum<[16]>;
  def S1   : MipsGPRReg< 17, "17">,  DwarfRegNum<[17]>;
  def S2   : MipsGPRReg< 18, "18">,  DwarfRegNum<[18]>;
  def S3   : MipsGPRReg< 19, "19">,  DwarfRegNum<[19]>;
  def S4   : MipsGPRReg< 20, "20">,  DwarfRegNum<[20]>;
  def S5   : MipsGPRReg< 21, "21">,  DwarfRegNum<[21]>;
  def S6   : MipsGPRReg< 22, "22">,  DwarfRegNum<[22]>;
  def S7   : MipsGPRReg< 23, "23">,  DwarfRegNum<[23]>;
  def T8   : MipsGPRReg< 24, "24">,  DwarfRegNum<[24]>;
  def T9   : MipsGPRReg< 25, "25">,  DwarfRegNum<[25]>;
  def K0   : MipsGPRReg< 26, "26">,  DwarfRegNum<[26]>;
  def K1   : MipsGPRReg< 27, "27">,  DwarfRegNum<[27]>;
  def GP   : MipsGPRReg< 28, "gp">,  DwarfRegNum<[28]>;
  def SP   : MipsGPRReg< 29, "sp">,  DwarfRegNum<[29]>;
  def FP   : MipsGPRReg< 30, "fp">,  DwarfRegNum<[30]>;
  def RA   : MipsGPRReg< 31, "ra">,  DwarfRegNum<[31]>;
```
- EN: Defines TableGen record `T7` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `T7`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 121-138
```tablegen
  // General Purpose 64-bit Registers
  let isConstant = true in
  def ZERO_64 : Mips64GPRReg< 0, "zero", [ZERO]>, DwarfRegNum<[0]>;
  def AT_64   : Mips64GPRReg< 1, "1",    [AT]>, DwarfRegNum<[1]>;
  def V0_64   : Mips64GPRReg< 2, "2",    [V0]>, DwarfRegNum<[2]>;
  def V1_64   : Mips64GPRReg< 3, "3",    [V1]>, DwarfRegNum<[3]>;
  def A0_64   : Mips64GPRReg< 4, "4",    [A0]>, DwarfRegNum<[4]>;
  def A1_64   : Mips64GPRReg< 5, "5",    [A1]>, DwarfRegNum<[5]>;
  def A2_64   : Mips64GPRReg< 6, "6",    [A2]>, DwarfRegNum<[6]>;
  def A3_64   : Mips64GPRReg< 7, "7",    [A3]>, DwarfRegNum<[7]>;
  def T0_64   : Mips64GPRReg< 8, "8",    [T0]>, DwarfRegNum<[8]>;
  def T1_64   : Mips64GPRReg< 9, "9",    [T1]>, DwarfRegNum<[9]>;
  def T2_64   : Mips64GPRReg< 10, "10",  [T2]>, DwarfRegNum<[10]>;
  def T3_64   : Mips64GPRReg< 11, "11",  [T3]>, DwarfRegNum<[11]>;
  def T4_64   : Mips64GPRReg< 12, "12",  [T4]>, DwarfRegNum<[12]>;
  def T5_64   : Mips64GPRReg< 13, "13",  [T5]>, DwarfRegNum<[13]>;
  def T6_64   : Mips64GPRReg< 14, "14",  [T6]>, DwarfRegNum<[14]>;
  def T7_64   : Mips64GPRReg< 15, "15",  [T7]>, DwarfRegNum<[15]>;
```
- EN: Defines TableGen record `ZERO_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ZERO_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 139-154
```tablegen
  def S0_64   : Mips64GPRReg< 16, "16",  [S0]>, DwarfRegNum<[16]>;
  def S1_64   : Mips64GPRReg< 17, "17",  [S1]>, DwarfRegNum<[17]>;
  def S2_64   : Mips64GPRReg< 18, "18",  [S2]>, DwarfRegNum<[18]>;
  def S3_64   : Mips64GPRReg< 19, "19",  [S3]>, DwarfRegNum<[19]>;
  def S4_64   : Mips64GPRReg< 20, "20",  [S4]>, DwarfRegNum<[20]>;
  def S5_64   : Mips64GPRReg< 21, "21",  [S5]>, DwarfRegNum<[21]>;
  def S6_64   : Mips64GPRReg< 22, "22",  [S6]>, DwarfRegNum<[22]>;
  def S7_64   : Mips64GPRReg< 23, "23",  [S7]>, DwarfRegNum<[23]>;
  def T8_64   : Mips64GPRReg< 24, "24",  [T8]>, DwarfRegNum<[24]>;
  def T9_64   : Mips64GPRReg< 25, "25",  [T9]>, DwarfRegNum<[25]>;
  def K0_64   : Mips64GPRReg< 26, "26",  [K0]>, DwarfRegNum<[26]>;
  def K1_64   : Mips64GPRReg< 27, "27",  [K1]>, DwarfRegNum<[27]>;
  def GP_64   : Mips64GPRReg< 28, "gp",  [GP]>, DwarfRegNum<[28]>;
  def SP_64   : Mips64GPRReg< 29, "sp",  [SP]>, DwarfRegNum<[29]>;
  def FP_64   : Mips64GPRReg< 30, "fp",  [FP]>, DwarfRegNum<[30]>;
  def RA_64   : Mips64GPRReg< 31, "ra",  [RA]>, DwarfRegNum<[31]>;
```
- EN: Defines TableGen record `S0_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S0_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 156-158
```tablegen
  /// Mips Single point precision FPU Registers
  foreach I = 0-31 in
  def F#I : FPR<I, "f"#I>, DwarfRegNum<[!add(I, 32)]>;
```
- EN: Defines TableGen record `F` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `F`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 160-162
```tablegen
  // Higher half of 64-bit FP registers.
  foreach I = 0-31 in
  def F_HI#I : FPR<I, "f"#I>, DwarfRegNum<[!add(I, 32)]>;
```
- EN: Defines TableGen record `F_HI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `F_HI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 164-169
```tablegen
  /// Mips Double point precision FPU Registers (aliased
  /// with the single precision to hold 64 bit values)
  foreach I = 0-15 in
  def D#I : AFPR<!shl(I, 1), "f"#!shl(I, 1),
                 [!cast<FPR>("F"#!shl(I, 1)),
                  !cast<FPR>("F"#!add(!shl(I, 1), 1))]>;
```
- EN: Defines TableGen record `D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 171-174
```tablegen
  /// Mips Double point precision FPU Registers in MFP64 mode.
  foreach I = 0-31 in
  def D#I#_64 : AFPR64<I, "f"#I, [!cast<FPR>("F"#I), !cast<FPR>("F_HI"#I)]>,
                DwarfRegNum<[!add(I, 32)]>;
```
- EN: Defines TableGen record `D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 176-180
```tablegen
  /// Mips MSA registers
  /// MSA and FPU cannot both be present unless the FPU has 64-bit registers
  foreach I = 0-31 in
  def W#I : AFPR128<I, "w"#I, [!cast<AFPR64>("D"#I#"_64")]>,
            DwarfRegNum<[!add(I, 32)]>;
```
- EN: Defines TableGen record `W` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `W`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 182-190
```tablegen
  // Hi/Lo registers
  def HI0 : MipsReg<0, "ac0">, DwarfRegNum<[64]>;
  def HI1 : MipsReg<1, "ac1">, DwarfRegNum<[176]>;
  def HI2 : MipsReg<2, "ac2">, DwarfRegNum<[178]>;
  def HI3 : MipsReg<3, "ac3">, DwarfRegNum<[180]>;
  def LO0 : MipsReg<0, "ac0">, DwarfRegNum<[65]>;
  def LO1 : MipsReg<1, "ac1">, DwarfRegNum<[177]>;
  def LO2 : MipsReg<2, "ac2">, DwarfRegNum<[179]>;
  def LO3 : MipsReg<3, "ac3">, DwarfRegNum<[181]>;
```
- EN: Defines TableGen record `HI0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HI0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 192-195
```tablegen
  let SubRegIndices = [sub_32] in {
  def HI0_64  : RegisterWithSubRegs<"hi", [HI0]>;
  def LO0_64  : RegisterWithSubRegs<"lo", [LO0]>;
  }
```
- EN: Defines TableGen record `HI0_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HI0_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 197-199
```tablegen
  // FP control registers.
  foreach I = 0-31 in
  def FCR#I : MipsReg<I, ""#I>;
```
- EN: Defines TableGen record `FCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 201-203
```tablegen
  // FP condition code registers.
  foreach I = 0-7 in
  def FCC#I : MipsReg<I, "fcc"#I>;
```
- EN: Defines TableGen record `FCC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 205-207
```tablegen
  // COP0 registers.
  foreach I = 0-31 in
  def COP0#I : MipsReg<I, ""#I>;
```
- EN: Defines TableGen record `COP0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 209-211
```tablegen
  // COP2 registers.
  foreach I = 0-31 in
  def COP2#I : MipsReg<I, ""#I>;
```
- EN: Defines TableGen record `COP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 213-215
```tablegen
  // COP3 registers.
  foreach I = 0-31 in
  def COP3#I : MipsReg<I, ""#I>;
```
- EN: Defines TableGen record `COP3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 217-218
```tablegen
  // PC register
  def PC : Register<"pc">;
```
- EN: Defines TableGen record `PC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 220-224
```tablegen
  // Hardware registers
  def HWR0 : MipsReg<0, "hwr_cpunum">;
  def HWR1 : MipsReg<1, "hwr_synci_step">;
  def HWR2 : MipsReg<2, "hwr_cc">;
  def HWR3 : MipsReg<3, "hwr_ccres">;
```
- EN: Defines TableGen record `HWR0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HWR0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 226-227
```tablegen
  foreach I = 4-31 in
  def HWR#I : MipsReg<I, ""#I>;
```
- EN: Defines TableGen record `HWR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HWR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 229-232
```tablegen
  // Accum registers
  foreach I = 0-3 in
  def AC#I : ACCReg<I, "ac"#I,
                    [!cast<Register>("LO"#I), !cast<Register>("HI"#I)]>;
```
- EN: Defines TableGen record `AC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 234-234
```tablegen
  def AC0_64 : ACCReg<0, "ac0", [LO0_64, HI0_64]>;
```
- EN: Defines TableGen record `AC0_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AC0_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 236-246
```tablegen
  // DSP-ASE control register fields.
  def DSPPos : Register<"">;
  def DSPSCount : Register<"">;
  def DSPCarry : Register<"">;
  def DSPEFI : Register<"">;
  def DSPOutFlag16_19 : Register<"">;
  def DSPOutFlag20 : Register<"">;
  def DSPOutFlag21 : Register<"">;
  def DSPOutFlag22 : Register<"">;
  def DSPOutFlag23 : Register<"">;
  def DSPCCond : Register<"">;
```
- EN: Defines TableGen record `DSPPos` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSPPos`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 248-252
```tablegen
  let SubRegIndices = [sub_dsp16_19, sub_dsp20, sub_dsp21, sub_dsp22,
                       sub_dsp23] in
  def DSPOutFlag : RegisterWithSubRegs<"", [DSPOutFlag16_19, DSPOutFlag20,
                                            DSPOutFlag21, DSPOutFlag22,
                                            DSPOutFlag23]>;
```
- EN: Defines TableGen record `DSPOutFlag` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSPOutFlag`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 254-267
```tablegen
  // MSA-ASE control registers.
  def MSAIR      : MipsReg<0, "0">;
  def MSACSR     : MipsReg<1, "1">;
  def MSAAccess  : MipsReg<2, "2">;
  def MSASave    : MipsReg<3, "3">;
  def MSAModify  : MipsReg<4, "4">;
  def MSARequest : MipsReg<5, "5">;
  def MSAMap     : MipsReg<6, "6">;
  def MSAUnmap   : MipsReg<7, "7">;
  // MSA-ASE fake control registers.
  // These registers do not exist, but instructions like `cfcmsa`
  // and `ctcmsa` allows to specify them.
  foreach I = 8-31 in
  def MSA#I : MipsReg<I, ""#I>;
```
- EN: Defines TableGen record `MSAIR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSAIR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 269-275
```tablegen
  // Octeon multiplier and product registers
  def MPL0 : MipsReg<0, "mpl0">;
  def MPL1 : MipsReg<1, "mpl1">;
  def MPL2 : MipsReg<2, "mpl2">;
  def P0 : MipsReg<0, "p0">;
  def P1 : MipsReg<1, "p1">;
  def P2 : MipsReg<2, "p2">;
```
- EN: Defines TableGen record `MPL0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MPL0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 277-277
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 279-281
```tablegen
//===----------------------------------------------------------------------===//
// Register Classes
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 283-296
```tablegen
class GPR32Class<list<ValueType> regTypes> :
  RegisterClass<"Mips", regTypes, 32, (add
  // Reserved
  ZERO, AT,
  // Return Values and Arguments
  V0, V1, A0, A1, A2, A3,
  // Not preserved across procedure calls
  T0, T1, T2, T3, T4, T5, T6, T7,
  // Callee save
  S0, S1, S2, S3, S4, S5, S6, S7,
  // Not preserved across procedure calls
  T8, T9,
  // Reserved
  K0, K1, GP, SP, FP, RA)>;
```
- EN: Declares reusable TableGen class `GPR32Class` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `GPR32Class`，通常用于抽象共享字段、谓词或编码结构。

### Lines 298-298
```tablegen
def GPR32 : GPR32Class<[i32]>;
```
- EN: Defines TableGen record `GPR32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 300-302
```tablegen
def GPR32ZERO : RegisterClass<"Mips", [i32], 32, (add
  // Reserved
  ZERO)>;
```
- EN: Defines TableGen record `GPR32ZERO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32ZERO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 304-316
```tablegen
def GPR32NONZERO : RegisterClass<"Mips", [i32], 32, (add
  // Reserved
  AT,
  // Return Values and Arguments
  V0, V1, A0, A1, A2, A3,
  // Not preserved across procedure calls
  T0, T1, T2, T3, T4, T5, T6, T7,
  // Callee save
  S0, S1, S2, S3, S4, S5, S6, S7,
  // Not preserved across procedure calls
  T8, T9,
  // Reserved
  K0, K1, GP, SP, FP, RA)>;
```
- EN: Defines TableGen record `GPR32NONZERO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32NONZERO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 318-318
```tablegen
def DSPR  : GPR32Class<[v4i8, v2i16, i32]>;
```
- EN: Defines TableGen record `DSPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 320-324
```tablegen
def GPRMM16 : RegisterClass<"Mips", [i32], 32, (add
  // Callee save
  S0, S1,
  // Return Values and Arguments
  V0, V1, A0, A1, A2, A3)>;
```
- EN: Defines TableGen record `GPRMM16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 326-332
```tablegen
def GPRMM16Zero : RegisterClass<"Mips", [i32], 32, (add
  // Reserved
  ZERO,
  // Callee save
  S1,
  // Return Values and Arguments
  V0, V1, A0, A1, A2, A3)>;
```
- EN: Defines TableGen record `GPRMM16Zero` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16Zero`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 334-342
```tablegen
def GPRMM16MoveP : RegisterClass<"Mips", [i32], 32, (add
  // Reserved
  ZERO,
  // Callee save
  S1,
  // Return Values and Arguments
  V0, V1,
  // Callee save
  S0, S2, S3, S4)>;
```
- EN: Defines TableGen record `GPRMM16MoveP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16MoveP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 344-346
```tablegen
def GPRMM16MovePPairFirst : RegisterClass<"Mips", [i32], 32, (add
  // Arguments
  A0, A1, A2)>;
```
- EN: Defines TableGen record `GPRMM16MovePPairFirst` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16MovePPairFirst`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 348-352
```tablegen
def GPRMM16MovePPairSecond : RegisterClass<"Mips", [i32], 32, (add
  // Arguments
  A1, A2, A3,
  // Callee save
  S5, S6)>;
```
- EN: Defines TableGen record `GPRMM16MovePPairSecond` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16MovePPairSecond`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 354-366
```tablegen
def GPR64 : RegisterClass<"Mips", [i64], 64, (add
  // Reserved
  ZERO_64, AT_64,
  // Return Values and Arguments
  V0_64, V1_64, A0_64, A1_64, A2_64, A3_64,
  // Not preserved across procedure calls
  T0_64, T1_64, T2_64, T3_64, T4_64, T5_64, T6_64, T7_64,
  // Callee save
  S0_64, S1_64, S2_64, S3_64, S4_64, S5_64, S6_64, S7_64,
  // Not preserved across procedure calls
  T8_64, T9_64,
  // Reserved
  K0_64, K1_64, GP_64, SP_64, FP_64, RA_64)>;
```
- EN: Defines TableGen record `GPR64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 368-372
```tablegen
def CPU16Regs : RegisterClass<"Mips", [i32], 32, (add
  // Return Values and Arguments
  V0, V1, A0, A1, A2, A3,
  // Callee save
  S0, S1)>;
```
- EN: Defines TableGen record `CPU16Regs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CPU16Regs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 374-379
```tablegen
def CPU16RegsPlusSP : RegisterClass<"Mips", [i32], 32, (add
  // Return Values and Arguments
  V0, V1, A0, A1, A2, A3,
  // Callee save
  S0, S1,
  SP)>;
```
- EN: Defines TableGen record `CPU16RegsPlusSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CPU16RegsPlusSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 381-381
```tablegen
def CPURAReg : RegisterClass<"Mips", [i32], 32, (add RA)>, Unallocatable;
```
- EN: Defines TableGen record `CPURAReg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CPURAReg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 383-383
```tablegen
def CPUSPReg : RegisterClass<"Mips", [i32], 32, (add SP)>, Unallocatable;
```
- EN: Defines TableGen record `CPUSPReg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CPUSPReg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 385-399
```tablegen
// 64bit fp:
// * FGR64  - 32 64-bit registers
// * AFGR64 - 16 32-bit even registers (32-bit FP Mode)
//
// 32bit fp:
// * FGR32 - 16 32-bit even registers
// * FGR32 - 32 32-bit registers (single float only mode)
def FGR32 : RegisterClass<"Mips", [f32], 32, (sequence "F%u", 0, 31)> {
  // Do not allocate odd registers when given -mattr=+nooddspreg.
  let AltOrders = [(decimate FGR32, 2)];
  let AltOrderSelect = [{
    const auto & S = MF.getSubtarget<MipsSubtarget>();
    return S.isABI_O32() && !S.useOddSPReg();
  }];
}
```
- EN: Defines TableGen record `FGR32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 401-411
```tablegen
def AFGR64 : RegisterClass<"Mips", [f64], 64, (add
  // Return Values and Arguments
  D0, D1,
  // Not preserved across procedure calls
  D2, D3, D4, D5,
  // Return Values and Arguments
  D6, D7,
  // Not preserved across procedure calls
  D8, D9,
  // Callee save
  D10, D11, D12, D13, D14, D15)>;
```
- EN: Defines TableGen record `AFGR64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AFGR64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 413-420
```tablegen
def FGR64 : RegisterClass<"Mips", [f64], 64, (sequence "D%u_64", 0, 31)> {
  // Do not allocate odd registers when given -mattr=+nooddspreg.
  let AltOrders = [(decimate FGR64, 2)];
  let AltOrderSelect = [{
    const auto & S = MF.getSubtarget<MipsSubtarget>();
    return S.isABI_O32() && !S.useOddSPReg();
  }];
}
```
- EN: Defines TableGen record `FGR64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 422-424
```tablegen
// FP control registers.
def CCR : RegisterClass<"Mips", [i32], 32, (sequence "FCR%u", 0, 31)>,
          Unallocatable;
```
- EN: Defines TableGen record `CCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 426-428
```tablegen
// FP condition code registers.
def FCC : RegisterClass<"Mips", [i32], 32, (sequence "FCC%u", 0, 7)>,
          Unallocatable;
```
- EN: Defines TableGen record `FCC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 430-433
```tablegen
// MIPS32r6/MIPS64r6 store FPU condition codes in normal FGR registers.
// This class allows us to represent this in codegen patterns.
def FGR32CC : RegisterClass<"Mips", [i32], 32, (sequence "F%u", 0, 31)>;
def FGR64CC : RegisterClass<"Mips", [i32, f32, f64], 64, (sequence "D%u_64", 0, 31)>;
```
- EN: Declares reusable TableGen class `allows` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `allows`，通常用于抽象共享字段、谓词或编码结构。

### Lines 435-435
```tablegen
def MSA128F16 : RegisterClass<"Mips", [f16], 128, (sequence "W%u", 0, 31)>;
```
- EN: Defines TableGen record `MSA128F16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128F16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 437-446
```tablegen
def MSA128B: RegisterClass<"Mips", [v16i8], 128,
                           (sequence "W%u", 0, 31)>;
def MSA128H: RegisterClass<"Mips", [v8i16, v8f16], 128,
                           (sequence "W%u", 0, 31)>;
def MSA128W: RegisterClass<"Mips", [v4i32, v4f32], 128,
                           (sequence "W%u", 0, 31)>;
def MSA128D: RegisterClass<"Mips", [v2i64, v2f64], 128,
                           (sequence "W%u", 0, 31)>;
def MSA128WEvens: RegisterClass<"Mips", [v4i32, v4f32], 128,
                                (decimate (sequence "W%u", 0, 31), 2)>;
```
- EN: Defines TableGen record `MSA128B:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128B:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 448-450
```tablegen
def MSACtrl: RegisterClass<"Mips", [i32], 32, (add
  MSAIR, MSACSR, MSAAccess, MSASave, MSAModify, MSARequest, MSAMap, MSAUnmap,
  (sequence "MSA%u", 8, 31))>, Unallocatable;
```
- EN: Defines TableGen record `MSACtrl:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSACtrl:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 452-458
```tablegen
// Hi/Lo Registers
def LO32 : RegisterClass<"Mips", [i32], 32, (add LO0)>;
def HI32 : RegisterClass<"Mips", [i32], 32, (add HI0)>;
def LO32DSP : RegisterClass<"Mips", [i32], 32, (sequence "LO%u", 0, 3)>;
def HI32DSP : RegisterClass<"Mips", [i32], 32, (sequence "HI%u", 0, 3)>;
def LO64 : RegisterClass<"Mips", [i64], 64, (add LO0_64)>;
def HI64 : RegisterClass<"Mips", [i64], 64, (add HI0_64)>;
```
- EN: Defines TableGen record `LO32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LO32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 460-462
```tablegen
// Hardware registers
def HWRegs : RegisterClass<"Mips", [i32], 32, (sequence "HWR%u", 0, 31)>,
             Unallocatable;
```
- EN: Defines TableGen record `HWRegs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HWRegs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 464-467
```tablegen
// Accumulator Registers
def ACC64 : RegisterClass<"Mips", [untyped], 64, (add AC0)> {
  let Size = 64;
}
```
- EN: Defines TableGen record `ACC64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ACC64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 469-471
```tablegen
def ACC128 : RegisterClass<"Mips", [untyped], 128, (add AC0_64)> {
  let Size = 128;
}
```
- EN: Defines TableGen record `ACC128` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ACC128`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 473-475
```tablegen
def ACC64DSP : RegisterClass<"Mips", [untyped], 64, (sequence "AC%u", 0, 3)> {
  let Size = 64;
}
```
- EN: Defines TableGen record `ACC64DSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ACC64DSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 477-477
```tablegen
def DSPCC : RegisterClass<"Mips", [v4i8, v2i16], 32, (add DSPCCond)>;
```
- EN: Defines TableGen record `DSPCC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSPCC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 479-481
```tablegen
// Coprocessor 0 registers.
def COP0 : RegisterClass<"Mips", [i32], 32, (sequence "COP0%u", 0, 31)>,
           Unallocatable;
```
- EN: Defines TableGen record `COP0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 483-485
```tablegen
// Coprocessor 2 registers.
def COP2 : RegisterClass<"Mips", [i32], 32, (sequence "COP2%u", 0, 31)>,
           Unallocatable;
```
- EN: Defines TableGen record `COP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 487-489
```tablegen
// Coprocessor 3 registers.
def COP3 : RegisterClass<"Mips", [i32], 32, (sequence "COP3%u", 0, 31)>,
           Unallocatable;
```
- EN: Defines TableGen record `COP3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 491-496
```tablegen
// Stack pointer and global pointer classes for instructions that are limited
// to a single register such as lwgp/lwsp in microMIPS.
def SP32 : RegisterClass<"Mips", [i32], 32, (add SP)>, Unallocatable;
def SP64 : RegisterClass<"Mips", [i64], 64, (add SP_64)>, Unallocatable;
def GP32 : RegisterClass<"Mips", [i32], 32, (add GP)>, Unallocatable;
def GP64 : RegisterClass<"Mips", [i64], 64, (add GP_64)>, Unallocatable;
```
- EN: Defines TableGen record `SP32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SP32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 498-502
```tablegen
// Octeon multiplier and product registers
def OCTEON_MPL : RegisterClass<"Mips", [i64], 64, (add MPL0, MPL1, MPL2)>,
                 Unallocatable;
def OCTEON_P : RegisterClass<"Mips", [i64], 64, (add P0, P1, P2)>,
               Unallocatable;
```
- EN: Defines TableGen record `OCTEON_MPL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OCTEON_MPL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 504-504
```tablegen
// Register Operands.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 506-508
```tablegen
class MipsAsmRegOperand : AsmOperandClass {
  let ParserMethod = "parseAnyRegister";
}
```
- EN: Declares reusable TableGen class `MipsAsmRegOperand` for `MipsRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsRegisterInfo` 声明可复用的 TableGen 类 `MipsAsmRegOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 510-513
```tablegen
def GPR64AsmOperand : MipsAsmRegOperand {
  let Name = "GPR64AsmReg";
  let PredicateMethod = "isGPRAsmReg";
}
```
- EN: Defines TableGen record `GPR64AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR64AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 515-518
```tablegen
def GPR32ZeroAsmOperand : MipsAsmRegOperand {
  let Name = "GPR32ZeroAsmReg";
  let PredicateMethod = "isGPRZeroAsmReg";
}
```
- EN: Defines TableGen record `GPR32ZeroAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32ZeroAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 520-523
```tablegen
def GPR32NonZeroAsmOperand : MipsAsmRegOperand {
  let Name = "GPR32NonZeroAsmReg";
  let PredicateMethod = "isGPRNonZeroAsmReg";
}
```
- EN: Defines TableGen record `GPR32NonZeroAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32NonZeroAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 525-528
```tablegen
def GPR32AsmOperand : MipsAsmRegOperand {
  let Name = "GPR32AsmReg";
  let PredicateMethod = "isGPRAsmReg";
}
```
- EN: Defines TableGen record `GPR32AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 530-533
```tablegen
def GPRMM16AsmOperand : MipsAsmRegOperand {
  let Name = "GPRMM16AsmReg";
  let PredicateMethod = "isMM16AsmReg";
}
```
- EN: Defines TableGen record `GPRMM16AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 535-538
```tablegen
def GPRMM16AsmOperandZero : MipsAsmRegOperand {
  let Name = "GPRMM16AsmRegZero";
  let PredicateMethod = "isMM16AsmRegZero";
}
```
- EN: Defines TableGen record `GPRMM16AsmOperandZero` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16AsmOperandZero`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 540-543
```tablegen
def GPRMM16AsmOperandMoveP : MipsAsmRegOperand {
  let Name = "GPRMM16AsmRegMoveP";
  let PredicateMethod = "isMM16AsmRegMoveP";
}
```
- EN: Defines TableGen record `GPRMM16AsmOperandMoveP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16AsmOperandMoveP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 545-548
```tablegen
def GPRMM16AsmOperandMovePPairFirst : MipsAsmRegOperand {
  let Name = "GPRMM16AsmRegMovePPairFirst";
  let PredicateMethod = "isMM16AsmRegMovePPairFirst";
}
```
- EN: Defines TableGen record `GPRMM16AsmOperandMovePPairFirst` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16AsmOperandMovePPairFirst`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 550-553
```tablegen
def GPRMM16AsmOperandMovePPairSecond : MipsAsmRegOperand {
  let Name = "GPRMM16AsmRegMovePPairSecond";
  let PredicateMethod = "isMM16AsmRegMovePPairSecond";
}
```
- EN: Defines TableGen record `GPRMM16AsmOperandMovePPairSecond` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16AsmOperandMovePPairSecond`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 555-558
```tablegen
def ACC64DSPAsmOperand : MipsAsmRegOperand {
  let Name = "ACC64DSPAsmReg";
  let PredicateMethod = "isACCAsmReg";
}
```
- EN: Defines TableGen record `ACC64DSPAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ACC64DSPAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 560-563
```tablegen
def HI32DSPAsmOperand : MipsAsmRegOperand {
  let Name = "HI32DSPAsmReg";
  let PredicateMethod = "isACCAsmReg";
}
```
- EN: Defines TableGen record `HI32DSPAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HI32DSPAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 565-568
```tablegen
def LO32DSPAsmOperand : MipsAsmRegOperand {
  let Name = "LO32DSPAsmReg";
  let PredicateMethod = "isACCAsmReg";
}
```
- EN: Defines TableGen record `LO32DSPAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LO32DSPAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 570-572
```tablegen
def CCRAsmOperand : MipsAsmRegOperand {
  let Name = "CCRAsmReg";
}
```
- EN: Defines TableGen record `CCRAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CCRAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 574-577
```tablegen
def AFGR64AsmOperand : MipsAsmRegOperand {
  let Name = "AFGR64AsmReg";
  let PredicateMethod = "isFGRAsmReg";
}
```
- EN: Defines TableGen record `AFGR64AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AFGR64AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 579-582
```tablegen
def StrictlyAFGR64AsmOperand : MipsAsmRegOperand {
  let Name = "StrictlyAFGR64AsmReg";
  let PredicateMethod = "isStrictlyFGRAsmReg";
}
```
- EN: Defines TableGen record `StrictlyAFGR64AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `StrictlyAFGR64AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 584-587
```tablegen
def FGR64AsmOperand : MipsAsmRegOperand {
  let Name = "FGR64AsmReg";
  let PredicateMethod = "isFGRAsmReg";
}
```
- EN: Defines TableGen record `FGR64AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR64AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 589-592
```tablegen
def StrictlyFGR64AsmOperand : MipsAsmRegOperand {
  let Name = "StrictlyFGR64AsmReg";
  let PredicateMethod = "isStrictlyFGRAsmReg";
}
```
- EN: Defines TableGen record `StrictlyFGR64AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `StrictlyFGR64AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 594-597
```tablegen
def FGR32AsmOperand : MipsAsmRegOperand {
  let Name = "FGR32AsmReg";
  let PredicateMethod = "isFGRAsmReg";
}
```
- EN: Defines TableGen record `FGR32AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR32AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 599-602
```tablegen
def StrictlyFGR32AsmOperand : MipsAsmRegOperand {
  let Name = "StrictlyFGR32AsmReg";
  let PredicateMethod = "isStrictlyFGRAsmReg";
}
```
- EN: Defines TableGen record `StrictlyFGR32AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `StrictlyFGR32AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 604-606
```tablegen
def FCCRegsAsmOperand : MipsAsmRegOperand {
  let Name = "FCCAsmReg";
}
```
- EN: Defines TableGen record `FCCRegsAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCCRegsAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 608-610
```tablegen
def MSA128AsmOperand : MipsAsmRegOperand {
  let Name = "MSA128AsmReg";
}
```
- EN: Defines TableGen record `MSA128AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 612-614
```tablegen
def MSACtrlAsmOperand : MipsAsmRegOperand {
  let Name = "MSACtrlAsmReg";
}
```
- EN: Defines TableGen record `MSACtrlAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSACtrlAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 616-618
```tablegen
def GPR32ZeroOpnd : RegisterOperand<GPR32ZERO> {
  let ParserMatchClass = GPR32ZeroAsmOperand;
}
```
- EN: Defines TableGen record `GPR32ZeroOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32ZeroOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 620-622
```tablegen
def GPR32NonZeroOpnd : RegisterOperand<GPR32NONZERO> {
  let ParserMatchClass = GPR32NonZeroAsmOperand;
}
```
- EN: Defines TableGen record `GPR32NonZeroOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32NonZeroOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 624-626
```tablegen
def GPR32Opnd : RegisterOperand<GPR32> {
  let ParserMatchClass = GPR32AsmOperand;
}
```
- EN: Defines TableGen record `GPR32Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR32Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 628-630
```tablegen
def GPRMM16Opnd : RegisterOperand<GPRMM16> {
  let ParserMatchClass = GPRMM16AsmOperand;
}
```
- EN: Defines TableGen record `GPRMM16Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 632-634
```tablegen
def GPRMM16OpndZero : RegisterOperand<GPRMM16Zero> {
  let ParserMatchClass = GPRMM16AsmOperandZero;
}
```
- EN: Defines TableGen record `GPRMM16OpndZero` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16OpndZero`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 636-639
```tablegen
def GPRMM16OpndMoveP : RegisterOperand<GPRMM16MoveP> {
  let ParserMatchClass = GPRMM16AsmOperandMoveP;
  let EncoderMethod = "getMovePRegSingleOpValue";
}
```
- EN: Defines TableGen record `GPRMM16OpndMoveP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16OpndMoveP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 641-643
```tablegen
def GPRMM16OpndMovePPairFirst : RegisterOperand<GPRMM16MovePPairFirst> {
  let ParserMatchClass = GPRMM16AsmOperandMovePPairFirst;
}
```
- EN: Defines TableGen record `GPRMM16OpndMovePPairFirst` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16OpndMovePPairFirst`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 645-647
```tablegen
def GPRMM16OpndMovePPairSecond : RegisterOperand<GPRMM16MovePPairSecond> {
  let ParserMatchClass = GPRMM16AsmOperandMovePPairSecond;
}
```
- EN: Defines TableGen record `GPRMM16OpndMovePPairSecond` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRMM16OpndMovePPairSecond`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 649-651
```tablegen
def GPR64Opnd : RegisterOperand<GPR64> {
  let ParserMatchClass = GPR64AsmOperand;
}
```
- EN: Defines TableGen record `GPR64Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR64Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 653-655
```tablegen
def DSPROpnd : RegisterOperand<DSPR> {
  let ParserMatchClass = GPR32AsmOperand;
}
```
- EN: Defines TableGen record `DSPROpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSPROpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 657-659
```tablegen
def CCROpnd : RegisterOperand<CCR> {
  let ParserMatchClass = CCRAsmOperand;
}
```
- EN: Defines TableGen record `CCROpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CCROpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 661-663
```tablegen
def HWRegsAsmOperand : MipsAsmRegOperand {
  let Name = "HWRegsAsmReg";
}
```
- EN: Defines TableGen record `HWRegsAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HWRegsAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 665-667
```tablegen
def COP0AsmOperand : MipsAsmRegOperand {
  let Name = "COP0AsmReg";
}
```
- EN: Defines TableGen record `COP0AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP0AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 669-671
```tablegen
def COP2AsmOperand : MipsAsmRegOperand {
  let Name = "COP2AsmReg";
}
```
- EN: Defines TableGen record `COP2AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP2AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 673-675
```tablegen
def COP3AsmOperand : MipsAsmRegOperand {
  let Name = "COP3AsmReg";
}
```
- EN: Defines TableGen record `COP3AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP3AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 677-679
```tablegen
def HWRegsOpnd : RegisterOperand<HWRegs> {
  let ParserMatchClass = HWRegsAsmOperand;
}
```
- EN: Defines TableGen record `HWRegsOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HWRegsOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 681-683
```tablegen
def AFGR64Opnd : RegisterOperand<AFGR64> {
  let ParserMatchClass = AFGR64AsmOperand;
}
```
- EN: Defines TableGen record `AFGR64Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AFGR64Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 685-687
```tablegen
def StrictlyAFGR64Opnd : RegisterOperand<AFGR64> {
  let ParserMatchClass = StrictlyAFGR64AsmOperand;
}
```
- EN: Defines TableGen record `StrictlyAFGR64Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `StrictlyAFGR64Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 689-691
```tablegen
def FGR64Opnd : RegisterOperand<FGR64> {
  let ParserMatchClass = FGR64AsmOperand;
}
```
- EN: Defines TableGen record `FGR64Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR64Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 693-695
```tablegen
def StrictlyFGR64Opnd : RegisterOperand<FGR64> {
  let ParserMatchClass = StrictlyFGR64AsmOperand;
}
```
- EN: Defines TableGen record `StrictlyFGR64Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `StrictlyFGR64Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 697-699
```tablegen
def FGR32Opnd : RegisterOperand<FGR32> {
  let ParserMatchClass = FGR32AsmOperand;
}
```
- EN: Defines TableGen record `FGR32Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR32Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 701-703
```tablegen
def StrictlyFGR32Opnd : RegisterOperand<FGR32> {
  let ParserMatchClass = StrictlyFGR32AsmOperand;
}
```
- EN: Defines TableGen record `StrictlyFGR32Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `StrictlyFGR32Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 705-709
```tablegen
def FGR64CCOpnd : RegisterOperand<FGR64CC> {
  // The assembler doesn't use register classes so we can re-use
  // FGR64AsmOperand.
  let ParserMatchClass = FGR64AsmOperand;
}
```
- EN: Defines TableGen record `FGR64CCOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR64CCOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 711-715
```tablegen
def FGR32CCOpnd : RegisterOperand<FGR32CC> {
  // The assembler doesn't use register classes so we can re-use
  // FGR32AsmOperand.
  let ParserMatchClass = FGR32AsmOperand;
}
```
- EN: Defines TableGen record `FGR32CCOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FGR32CCOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 717-719
```tablegen
def FCCRegsOpnd : RegisterOperand<FCC> {
  let ParserMatchClass = FCCRegsAsmOperand;
}
```
- EN: Defines TableGen record `FCCRegsOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCCRegsOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 721-723
```tablegen
def LO32DSPOpnd : RegisterOperand<LO32DSP> {
  let ParserMatchClass = LO32DSPAsmOperand;
}
```
- EN: Defines TableGen record `LO32DSPOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LO32DSPOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 725-727
```tablegen
def HI32DSPOpnd : RegisterOperand<HI32DSP> {
  let ParserMatchClass = HI32DSPAsmOperand;
}
```
- EN: Defines TableGen record `HI32DSPOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HI32DSPOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 729-731
```tablegen
def ACC64DSPOpnd : RegisterOperand<ACC64DSP> {
  let ParserMatchClass = ACC64DSPAsmOperand;
}
```
- EN: Defines TableGen record `ACC64DSPOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ACC64DSPOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 733-735
```tablegen
def COP0Opnd : RegisterOperand<COP0> {
  let ParserMatchClass = COP0AsmOperand;
}
```
- EN: Defines TableGen record `COP0Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP0Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 737-739
```tablegen
def COP2Opnd : RegisterOperand<COP2> {
  let ParserMatchClass = COP2AsmOperand;
}
```
- EN: Defines TableGen record `COP2Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP2Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 741-743
```tablegen
def COP3Opnd : RegisterOperand<COP3> {
  let ParserMatchClass = COP3AsmOperand;
}
```
- EN: Defines TableGen record `COP3Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COP3Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 745-747
```tablegen
def MSA128F16Opnd : RegisterOperand<MSA128F16> {
  let ParserMatchClass = MSA128AsmOperand;
}
```
- EN: Defines TableGen record `MSA128F16Opnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128F16Opnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 749-751
```tablegen
def MSA128BOpnd : RegisterOperand<MSA128B> {
  let ParserMatchClass = MSA128AsmOperand;
}
```
- EN: Defines TableGen record `MSA128BOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128BOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 753-755
```tablegen
def MSA128HOpnd : RegisterOperand<MSA128H> {
  let ParserMatchClass = MSA128AsmOperand;
}
```
- EN: Defines TableGen record `MSA128HOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128HOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 757-759
```tablegen
def MSA128WOpnd : RegisterOperand<MSA128W> {
  let ParserMatchClass = MSA128AsmOperand;
}
```
- EN: Defines TableGen record `MSA128WOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128WOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 761-763
```tablegen
def MSA128DOpnd : RegisterOperand<MSA128D> {
  let ParserMatchClass = MSA128AsmOperand;
}
```
- EN: Defines TableGen record `MSA128DOpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128DOpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 765-767
```tablegen
def MSA128CROpnd : RegisterOperand<MSACtrl> {
  let ParserMatchClass = MSACtrlAsmOperand;
}
```
- EN: Defines TableGen record `MSA128CROpnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSA128CROpnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 769-771
```tablegen
def mips_ptr_rc : RegClassByHwMode<
  [MIPS32, MIPS64],
  [GPR32, GPR64]>;
```
- EN: Defines TableGen record `mips_ptr_rc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mips_ptr_rc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 773-775
```tablegen
def ptr_gpr16mm_rc : RegClassByHwMode<
  [MIPS32, MIPS64],
  [GPRMM16, GPRMM16]>; // FIXME: Why even use this if it's just a constant
```
- EN: Defines TableGen record `ptr_gpr16mm_rc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ptr_gpr16mm_rc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 777-779
```tablegen
def ptr_sp_rc : RegClassByHwMode<
  [MIPS32, MIPS64],
  [SP32, SP64]>;
```
- EN: Defines TableGen record `ptr_sp_rc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ptr_sp_rc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 781-783
```tablegen
def ptr_gp_rc : RegClassByHwMode<
  [MIPS32, MIPS64],
  [GP32, GP64]>;
```
- EN: Defines TableGen record `ptr_gp_rc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ptr_gp_rc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: register definitions, allocation constraints, and register utilities.
  - CN: 核心职责：寄存器定义、分配约束以及寄存器工具。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
