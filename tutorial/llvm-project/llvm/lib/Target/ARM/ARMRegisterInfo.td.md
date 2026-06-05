# ARMRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMRegisterInfo.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMRegisterInfo` in LLVM TableGen DSL for the ARM backend, covering register definitions, allocation constraints, and register utilities.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMRegisterInfo`，涵盖寄存器定义、分配约束以及寄存器工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- ARMRegisterInfo.td - ARM Register defs -------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-9
```tablegen
include "ARMSystemRegister.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 11-13
```tablegen
//===----------------------------------------------------------------------===//
//  Declarations that describe the ARM register file
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 15-23
```tablegen
// Registers are identified with 4-bit ID numbers.
class ARMReg<bits<16> Enc, string n, list<Register> subregs = [],
             list<string> altNames = []> : Register<n, altNames> {
  let HWEncoding = Enc;
  let Namespace = "ARM";
  let SubRegs = subregs;
  // All bits of ARM registers with sub-registers are covered by sub-registers.
  let CoveredBySubRegs = 1;
}
```
- EN: Declares reusable TableGen class `ARMReg` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `ARMReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 25-28
```tablegen
class ARMFReg<bits<16> Enc, string n> : Register<n> {
  let HWEncoding = Enc;
  let Namespace = "ARM";
}
```
- EN: Declares reusable TableGen class `ARMFReg` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `ARMFReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 30-33
```tablegen
let Namespace = "ARM",
    FallbackRegAltNameIndex = NoRegAltName in {
  def RegNamesRaw : RegAltNameIndex;
}
```
- EN: Defines TableGen record `RegNamesRaw` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RegNamesRaw`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 35-38
```tablegen
// Subregister indices.
let Namespace = "ARM" in {
def qqsub_0 : SubRegIndex<256>;
def qqsub_1 : SubRegIndex<256, 256>;
```
- EN: Defines TableGen record `qqsub_0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `qqsub_0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 40-44
```tablegen
// Note: Code depends on these having consecutive numbers.
def qsub_0 : SubRegIndex<128>;
def qsub_1 : SubRegIndex<128, 128>;
def qsub_2 : ComposedSubRegIndex<qqsub_1, qsub_0>;
def qsub_3 : ComposedSubRegIndex<qqsub_1, qsub_1>;
```
- EN: Defines TableGen record `qsub_0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `qsub_0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 46-53
```tablegen
def dsub_0 : SubRegIndex<64>;
def dsub_1 : SubRegIndex<64, 64>;
def dsub_2 : ComposedSubRegIndex<qsub_1, dsub_0>;
def dsub_3 : ComposedSubRegIndex<qsub_1, dsub_1>;
def dsub_4 : ComposedSubRegIndex<qsub_2, dsub_0>;
def dsub_5 : ComposedSubRegIndex<qsub_2, dsub_1>;
def dsub_6 : ComposedSubRegIndex<qsub_3, dsub_0>;
def dsub_7 : ComposedSubRegIndex<qsub_3, dsub_1>;
```
- EN: Defines TableGen record `dsub_0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `dsub_0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 55-70
```tablegen
def ssub_0  : SubRegIndex<32>;
def ssub_1  : SubRegIndex<32, 32>;
def ssub_2  : ComposedSubRegIndex<dsub_1, ssub_0>;
def ssub_3  : ComposedSubRegIndex<dsub_1, ssub_1>;
def ssub_4  : ComposedSubRegIndex<dsub_2, ssub_0>;
def ssub_5  : ComposedSubRegIndex<dsub_2, ssub_1>;
def ssub_6  : ComposedSubRegIndex<dsub_3, ssub_0>;
def ssub_7  : ComposedSubRegIndex<dsub_3, ssub_1>;
def ssub_8  : ComposedSubRegIndex<dsub_4, ssub_0>;
def ssub_9  : ComposedSubRegIndex<dsub_4, ssub_1>;
def ssub_10 : ComposedSubRegIndex<dsub_5, ssub_0>;
def ssub_11 : ComposedSubRegIndex<dsub_5, ssub_1>;
def ssub_12 : ComposedSubRegIndex<dsub_6, ssub_0>;
def ssub_13 : ComposedSubRegIndex<dsub_6, ssub_1>;
def ssub_14 : ComposedSubRegIndex<dsub_7, ssub_0>;
def ssub_15 : ComposedSubRegIndex<dsub_7, ssub_1>;
```
- EN: Defines TableGen record `ssub_0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ssub_0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 72-76
```tablegen
def gsub_0 : SubRegIndex<32>;
def gsub_1 : SubRegIndex<32, 32>;
// Let TableGen synthesize the remaining 12 ssub_* indices.
// We don't need to name them.
}
```
- EN: Defines TableGen record `gsub_0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `gsub_0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 78-95
```tablegen
// Integer registers
def R0  : ARMReg< 0, "r0">,  DwarfRegNum<[0]>;
def R1  : ARMReg< 1, "r1">,  DwarfRegNum<[1]>;
def R2  : ARMReg< 2, "r2">,  DwarfRegNum<[2]>;
def R3  : ARMReg< 3, "r3">,  DwarfRegNum<[3]>;
def R4  : ARMReg< 4, "r4">,  DwarfRegNum<[4]>;
def R5  : ARMReg< 5, "r5">,  DwarfRegNum<[5]>;
def R6  : ARMReg< 6, "r6">,  DwarfRegNum<[6]>;
def R7  : ARMReg< 7, "r7">,  DwarfRegNum<[7]>;
// These require 32-bit instructions.
let CostPerUse = [1] in {
def R8  : ARMReg< 8, "r8">,  DwarfRegNum<[8]>;
def R9  : ARMReg< 9, "r9">,  DwarfRegNum<[9]>;
def R10 : ARMReg<10, "r10">, DwarfRegNum<[10]>;
def R11 : ARMReg<11, "r11">, DwarfRegNum<[11]>;
def R12 : ARMReg<12, "r12">, DwarfRegNum<[12]>;
let RegAltNameIndices = [RegNamesRaw] in {
def SP  : ARMReg<13, "sp", [], ["r13"]>,  DwarfRegNum<[13]>;
```
- EN: Defines TableGen record `R0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `R0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 96-99
```tablegen
def LR  : ARMReg<14, "lr", [], ["r14"]>,  DwarfRegNum<[14]>;
def PC  : ARMReg<15, "pc", [], ["r15"]>,  DwarfRegNum<[15]>;
}
}
```
- EN: Defines TableGen record `LR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 101-117
```tablegen
// Float registers
def S0  : ARMFReg< 0, "s0">;  def S1  : ARMFReg< 1, "s1">;
def S2  : ARMFReg< 2, "s2">;  def S3  : ARMFReg< 3, "s3">;
def S4  : ARMFReg< 4, "s4">;  def S5  : ARMFReg< 5, "s5">;
def S6  : ARMFReg< 6, "s6">;  def S7  : ARMFReg< 7, "s7">;
def S8  : ARMFReg< 8, "s8">;  def S9  : ARMFReg< 9, "s9">;
def S10 : ARMFReg<10, "s10">; def S11 : ARMFReg<11, "s11">;
def S12 : ARMFReg<12, "s12">; def S13 : ARMFReg<13, "s13">;
def S14 : ARMFReg<14, "s14">; def S15 : ARMFReg<15, "s15">;
def S16 : ARMFReg<16, "s16">; def S17 : ARMFReg<17, "s17">;
def S18 : ARMFReg<18, "s18">; def S19 : ARMFReg<19, "s19">;
def S20 : ARMFReg<20, "s20">; def S21 : ARMFReg<21, "s21">;
def S22 : ARMFReg<22, "s22">; def S23 : ARMFReg<23, "s23">;
def S24 : ARMFReg<24, "s24">; def S25 : ARMFReg<25, "s25">;
def S26 : ARMFReg<26, "s26">; def S27 : ARMFReg<27, "s27">;
def S28 : ARMFReg<28, "s28">; def S29 : ARMFReg<29, "s29">;
def S30 : ARMFReg<30, "s30">; def S31 : ARMFReg<31, "s31">;
```
- EN: Defines TableGen record `S0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 119-136
```tablegen
// Aliases of the F* registers used to hold 64-bit fp values (doubles)
let SubRegIndices = [ssub_0, ssub_1] in {
def D0  : ARMReg< 0,  "d0", [S0,   S1]>, DwarfRegNum<[256]>;
def D1  : ARMReg< 1,  "d1", [S2,   S3]>, DwarfRegNum<[257]>;
def D2  : ARMReg< 2,  "d2", [S4,   S5]>, DwarfRegNum<[258]>;
def D3  : ARMReg< 3,  "d3", [S6,   S7]>, DwarfRegNum<[259]>;
def D4  : ARMReg< 4,  "d4", [S8,   S9]>, DwarfRegNum<[260]>;
def D5  : ARMReg< 5,  "d5", [S10, S11]>, DwarfRegNum<[261]>;
def D6  : ARMReg< 6,  "d6", [S12, S13]>, DwarfRegNum<[262]>;
def D7  : ARMReg< 7,  "d7", [S14, S15]>, DwarfRegNum<[263]>;
def D8  : ARMReg< 8,  "d8", [S16, S17]>, DwarfRegNum<[264]>;
def D9  : ARMReg< 9,  "d9", [S18, S19]>, DwarfRegNum<[265]>;
def D10 : ARMReg<10, "d10", [S20, S21]>, DwarfRegNum<[266]>;
def D11 : ARMReg<11, "d11", [S22, S23]>, DwarfRegNum<[267]>;
def D12 : ARMReg<12, "d12", [S24, S25]>, DwarfRegNum<[268]>;
def D13 : ARMReg<13, "d13", [S26, S27]>, DwarfRegNum<[269]>;
def D14 : ARMReg<14, "d14", [S28, S29]>, DwarfRegNum<[270]>;
def D15 : ARMReg<15, "d15", [S30, S31]>, DwarfRegNum<[271]>;
```
- EN: Defines TableGen record `D0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 137-137
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 139-155
```tablegen
// VFP3 defines 16 additional double registers
def D16 : ARMFReg<16, "d16">, DwarfRegNum<[272]>;
def D17 : ARMFReg<17, "d17">, DwarfRegNum<[273]>;
def D18 : ARMFReg<18, "d18">, DwarfRegNum<[274]>;
def D19 : ARMFReg<19, "d19">, DwarfRegNum<[275]>;
def D20 : ARMFReg<20, "d20">, DwarfRegNum<[276]>;
def D21 : ARMFReg<21, "d21">, DwarfRegNum<[277]>;
def D22 : ARMFReg<22, "d22">, DwarfRegNum<[278]>;
def D23 : ARMFReg<23, "d23">, DwarfRegNum<[279]>;
def D24 : ARMFReg<24, "d24">, DwarfRegNum<[280]>;
def D25 : ARMFReg<25, "d25">, DwarfRegNum<[281]>;
def D26 : ARMFReg<26, "d26">, DwarfRegNum<[282]>;
def D27 : ARMFReg<27, "d27">, DwarfRegNum<[283]>;
def D28 : ARMFReg<28, "d28">, DwarfRegNum<[284]>;
def D29 : ARMFReg<29, "d29">, DwarfRegNum<[285]>;
def D30 : ARMFReg<30, "d30">, DwarfRegNum<[286]>;
def D31 : ARMFReg<31, "d31">, DwarfRegNum<[287]>;
```
- EN: Defines TableGen record `D16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 157-174
```tablegen
// Advanced SIMD (NEON) defines 16 quad-word aliases
let SubRegIndices = [dsub_0, dsub_1] in {
def Q0  : ARMReg< 0,  "q0", [D0,   D1]>;
def Q1  : ARMReg< 1,  "q1", [D2,   D3]>;
def Q2  : ARMReg< 2,  "q2", [D4,   D5]>;
def Q3  : ARMReg< 3,  "q3", [D6,   D7]>;
def Q4  : ARMReg< 4,  "q4", [D8,   D9]>;
def Q5  : ARMReg< 5,  "q5", [D10, D11]>;
def Q6  : ARMReg< 6,  "q6", [D12, D13]>;
def Q7  : ARMReg< 7,  "q7", [D14, D15]>;
}
let SubRegIndices = [dsub_0, dsub_1] in {
def Q8  : ARMReg< 8,  "q8", [D16, D17]>;
def Q9  : ARMReg< 9,  "q9", [D18, D19]>;
def Q10 : ARMReg<10, "q10", [D20, D21]>;
def Q11 : ARMReg<11, "q11", [D22, D23]>;
def Q12 : ARMReg<12, "q12", [D24, D25]>;
def Q13 : ARMReg<13, "q13", [D26, D27]>;
```
- EN: Defines TableGen record `Q0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Q0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 175-177
```tablegen
def Q14 : ARMReg<14, "q14", [D28, D29]>;
def Q15 : ARMReg<15, "q15", [D30, D31]>;
}
```
- EN: Defines TableGen record `Q14` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Q14`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 179-196
```tablegen
// Current Program Status Register.
// We model fpscr with three registers. FPSCR models the control bits and will be
// reserved. FPSCR_RM models rounding mode control bits and will be reserved.
// FPSCR_NZCV models the flag bits and will be unreserved. APSR_NZCV
// models the APSR when it's accessed by some special instructions. In such cases
// it has the same encoding as PC.
def CPSR       : ARMReg<0,  "cpsr">;
def APSR       : ARMReg<15, "apsr">;
def APSR_NZCV  : ARMReg<15, "apsr_nzcv">;
def SPSR       : ARMReg<2,  "spsr">;
def FPSCR      : ARMReg<3,  "fpscr">;
def FPSCR_NZCV : ARMReg<3,  "fpscr_nzcv"> {
  let Aliases = [FPSCR];
}
def FPSCR_RM : ARMReg<3,  "fpscr_rm"> {
  let Aliases = [FPSCR];
}
def ITSTATE    : ARMReg<4, "itstate">;
```
- EN: Defines TableGen record `CPSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CPSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 198-214
```tablegen
// Special Registers - only available in privileged mode.
def FPSID   : ARMReg<0,  "fpsid">;
def MVFR2   : ARMReg<5,  "mvfr2">;
def MVFR1   : ARMReg<6,  "mvfr1">;
def MVFR0   : ARMReg<7,  "mvfr0">;
def FPEXC   : ARMReg<8,  "fpexc">;
def FPINST  : ARMReg<9,  "fpinst">;
def FPINST2 : ARMReg<10, "fpinst2">;
// These encodings aren't actual instruction encodings, their encoding depends
// on the instruction they are used in and for VPR 64 was chosen such that it
// always comes last in spr_reglist_with_vpr.
def VPR     : ARMReg<64, "vpr">;
def FPSCR_NZCVQC
            : ARMReg<2, "fpscr_nzcvqc">;
def P0      : ARMReg<13, "p0">;
def FPCXTNS : ARMReg<14, "fpcxtns">;
def FPCXTS  : ARMReg<15, "fpcxts">;
```
- EN: Defines TableGen record `FPSID` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FPSID`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 216-216
```tablegen
def ZR  : ARMReg<15, "zr">,  DwarfRegNum<[15]>;
```
- EN: Defines TableGen record `ZR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ZR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 218-218
```tablegen
def RA_AUTH_CODE : ARMReg<12, "ra_auth_code">, DwarfRegNum<[143]>;
```
- EN: Defines TableGen record `RA_AUTH_CODE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RA_AUTH_CODE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 220-237
```tablegen
// Register classes.
//
// pc  == Program Counter
// lr  == Link Register
// sp  == Stack Pointer
// r12 == ip (scratch)
// r7  == Frame Pointer (thumb-style backtraces)
// r9  == May be reserved as Thread Register
// r11 == Frame Pointer (arm-style backtraces)
// r10 == Stack Limit
//
def GPR : RegisterClass<"ARM", [i32], 32, (add (sequence "R%u", 0, 12),
                                               SP, LR, PC)> {
  // Allocate LR as the first CSR since it is always saved anyway.
  // For Thumb1 mode, we don't want to allocate hi regs at all, as we don't
  // know how to spill them. If we make our prologue/epilogue code smarter at
  // some point, we can go back to using the above allocation orders for the
  // Thumb1 instructions that know how to use hi regs.
```
- EN: Defines TableGen record `GPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 238-244
```tablegen
  let AltOrders = [(add LR, GPR), (trunc GPR, 8),
                   (add (trunc GPR, 8), R12, LR, (shl GPR, 8))];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().getGPRAllocationOrder(MF);
  }];
  let DiagnosticString = "operand must be a register in range [r0, r15]";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 246-261
```tablegen
// Register set that excludes registers that are reserved for procedure calls.
// This is used for pseudo-instructions that are actually implemented using a
// procedure call.
def GPRnoip : RegisterClass<"ARM", [i32], 32, (sub GPR, R12, LR)> {
  // Allocate LR as the first CSR since it is always saved anyway.
  // For Thumb1 mode, we don't want to allocate hi regs at all, as we don't
  // know how to spill them. If we make our prologue/epilogue code smarter at
  // some point, we can go back to using the above allocation orders for the
  // Thumb1 instructions that know how to use hi regs.
  let AltOrders = [(add GPRnoip, GPRnoip), (trunc GPRnoip, 8),
                   (add (trunc GPRnoip, 8), (shl GPRnoip, 8))];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().getGPRAllocationOrder(MF);
  }];
  let DiagnosticString = "operand must be a register in range [r0, r14]";
}
```
- EN: Defines TableGen record `GPRnoip` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRnoip`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 263-273
```tablegen
// GPRs without the PC.  Some ARM instructions do not allow the PC in
// certain operand slots, particularly as the destination.  Primarily
// useful for disassembly.
def GPRnopc : RegisterClass<"ARM", [i32], 32, (sub GPR, PC)> {
  let AltOrders = [(add LR, GPRnopc), (trunc GPRnopc, 8),
                   (add (trunc GPRnopc, 8), R12, LR, (shl GPRnopc, 8))];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().getGPRAllocationOrder(MF);
  }];
  let DiagnosticString = "operand must be a register in range [r0, r14]";
}
```
- EN: Defines TableGen record `GPRnopc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRnopc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 275-284
```tablegen
// GPRs without the PC but with APSR. Some instructions allow accessing the
// APSR, while actually encoding PC in the register field. This is useful
// for assembly and disassembly only.
def GPRwithAPSR : RegisterClass<"ARM", [i32], 32, (add (sub GPR, PC), APSR_NZCV)> {
  let AltOrders = [(add LR, GPRnopc), (trunc GPRnopc, 8)];
  let AltOrderSelect = [{
      return 1 + MF.getSubtarget<ARMSubtarget>().isThumb1Only();
  }];
  let DiagnosticString = "operand must be a register in range [r0, r14] or apsr_nzcv";
}
```
- EN: Defines TableGen record `GPRwithAPSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRwithAPSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 286-294
```tablegen
// GPRs without the SP register. Used for BXAUT and AUTG
def GPRnosp : RegisterClass<"ARM", [i32], 32, (add (sequence "R%u", 0, 12), LR, PC)> {
  let AltOrders = [(add LR, GPRnosp), (trunc GPRnosp, 8),
                   (add (trunc GPRnosp, 8), R12, LR, (shl GPRnosp, 8))];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().getGPRAllocationOrder(MF);
  }];
  let DiagnosticString = "operand must be a register in range [r0, r12] or LR or PC";
}
```
- EN: Defines TableGen record `GPRnosp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRnosp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 296-299
```tablegen
// GPRs without the PC and SP registers but with APSR. Used by CLRM instruction.
def GPRwithAPSRnosp : RegisterClass<"ARM", [i32], 32, (add (sequence "R%u", 0, 12), LR, APSR)> {
  let isAllocatable = 0;
}
```
- EN: Defines TableGen record `GPRwithAPSRnosp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRwithAPSRnosp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 301-307
```tablegen
def GPRwithZR : RegisterClass<"ARM", [i32], 32, (add (sub GPR, PC), ZR)> {
  let AltOrders = [(add LR, GPRwithZR), (trunc GPRwithZR, 8)];
  let AltOrderSelect = [{
      return 1 + MF.getSubtarget<ARMSubtarget>().isThumb1Only();
  }];
  let DiagnosticString = "operand must be a register in range [r0, r14] or zr";
}
```
- EN: Defines TableGen record `GPRwithZR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRwithZR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 309-315
```tablegen
def GPRwithZRnosp : RegisterClass<"ARM", [i32], 32, (sub GPRwithZR, SP)> {
  let AltOrders = [(add LR, GPRwithZRnosp), (trunc GPRwithZRnosp, 8)];
  let AltOrderSelect = [{
      return 1 + MF.getSubtarget<ARMSubtarget>().isThumb1Only();
  }];
  let DiagnosticString = "operand must be a register in range [r0, r12] or r14 or zr";
}
```
- EN: Defines TableGen record `GPRwithZRnosp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRwithZRnosp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 317-324
```tablegen
// GPRsp - Only the SP is legal. Used by Thumb1 instructions that want the
// implied SP argument list.
// FIXME: It would be better to not use this at all and refactor the
// instructions to not have SP an an explicit argument. That makes
// frame index resolution a bit trickier, though.
def GPRsp : RegisterClass<"ARM", [i32], 32, (add SP)> {
  let DiagnosticString = "operand must be a register sp";
}
```
- EN: Defines TableGen record `GPRsp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRsp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 326-328
```tablegen
// GPRlr - Only LR is legal. Used by ARMv8.1-M Low Overhead Loop instructions
// where LR is the only legal loop counter register.
def GPRlr : RegisterClass<"ARM", [i32], 32, (add LR)>;
```
- EN: Defines TableGen record `GPRlr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRlr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 330-341
```tablegen
// restricted GPR register class. Many Thumb2 instructions allow the full
// register range for operands, but have undefined behaviours when PC
// or SP (R13 or R15) are used. The ARM ISA refers to these operands
// via the BadReg() pseudo-code description.
def rGPR : RegisterClass<"ARM", [i32], 32, (sub GPR, SP, PC)> {
  let AltOrders = [(add LR, rGPR), (trunc rGPR, 8),
                   (add (trunc rGPR, 8), R12, LR, (shl rGPR, 8))];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().getGPRAllocationOrder(MF);
  }];
  let DiagnosticType = "rGPR";
}
```
- EN: Defines TableGen record `rGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 343-352
```tablegen
// GPRs without the PC and SP but with APSR_NZCV.Some instructions allow
// accessing the APSR_NZCV, while actually encoding PC in the register field.
// This is useful for assembly and disassembly only.
// Currently used by the CDE extension.
def GPRwithAPSR_NZCVnosp
  : RegisterClass<"ARM", [i32], 32, (add (sequence "R%u", 0, 12), LR, APSR_NZCV)> {
  let isAllocatable = 0;
  let DiagnosticString =
    "operand must be a register in the range [r0, r12], r14 or apsr_nzcv";
}
```
- EN: Defines TableGen record `GPRwithAPSR_NZCVnosp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRwithAPSR_NZCVnosp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 354-358
```tablegen
// Thumb registers are R0-R7 normally. Some instructions can still use
// the general GPR register class above (MOV, e.g.)
def tGPR : RegisterClass<"ARM", [i32], 32, (trunc GPR, 8)> {
  let DiagnosticString = "operand must be a register in range [r0, r7]";
}
```
- EN: Declares reusable TableGen class `above` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `above`，通常用于抽象共享字段、谓词或编码结构。

### Lines 360-362
```tablegen
// Thumb registers R0-R7 and the PC. Some instructions like TBB or THH allow
// the PC to be used as a destination operand as well.
def tGPRwithpc : RegisterClass<"ARM", [i32], 32, (add tGPR, PC)>;
```
- EN: Defines TableGen record `tGPRwithpc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tGPRwithpc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 364-367
```tablegen
// The high registers in thumb mode, R8-R15.
def hGPR : RegisterClass<"ARM", [i32], 32, (sub GPR, tGPR)> {
  let DiagnosticString = "operand must be a register in range [r8, r15]";
}
```
- EN: Defines TableGen record `hGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `hGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 369-378
```tablegen
// For tail calls, we can't use callee-saved registers, as they are restored
// to the saved value before the tail call, which would clobber a call address.
// Note, getMinimalPhysRegClass(R0) returns tGPR because of the names of
// this class and the preceding one(!)  This is what we want.
def tcGPR : RegisterClass<"ARM", [i32], 32, (add R0, R1, R2, R3, R12)> {
  let AltOrders = [(and tcGPR, tGPR)];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().isThumb1Only();
  }];
}
```
- EN: Declares reusable TableGen class `and` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `and`，通常用于抽象共享字段、谓词或编码结构。

### Lines 380-388
```tablegen
// Some pointer authentication instructions require the use of R12. When return
// address signing is enabled, authentication of the caller's return address
// must be performed before a tail call is made. Therefore, indirect tail call
// jump cannot be from R12.
// FIXME: All PACBTI instruction currently implemented in the compiler
// implicitly use R12. When instructions that allow PAC to be placed in a
// specific register are implemented the restriction needs to be updated to
// make sure that PACBTI signature and indirect tail call both use a different register.
def tcGPRnotr12 : RegisterClass<"ARM", [i32], 32, (add R0, R1, R2, R3)>;
```
- EN: Defines TableGen record `tcGPRnotr12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tcGPRnotr12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 390-397
```tablegen
def tGPROdd : RegisterClass<"ARM", [i32], 32, (add R1, R3, R5, R7, R9, R11)> {
  let AltOrders = [(and tGPROdd, tGPR)];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().isThumb1Only();
  }];
  let DiagnosticString =
    "operand must be an odd-numbered register in range [r1,r11]";
}
```
- EN: Defines TableGen record `tGPROdd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tGPROdd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 399-405
```tablegen
def tGPREven : RegisterClass<"ARM", [i32], 32, (add R0, R2, R4, R6, R8, R10, R12, LR)> {
  let AltOrders = [(and tGPREven, tGPR)];
  let AltOrderSelect = [{
      return MF.getSubtarget<ARMSubtarget>().isThumb1Only();
  }];
  let DiagnosticString = "operand must be an even-numbered register";
}
```
- EN: Defines TableGen record `tGPREven` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tGPREven`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 407-411
```tablegen
// Condition code registers.
def CCR : RegisterClass<"ARM", [i32], 32, (add CPSR)> {
  let CopyCost = -1;  // Don't allow copying of status registers.
  let isAllocatable = 0;
}
```
- EN: Defines TableGen record `CCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 413-416
```tablegen
// MVE Condition code register.
def VCCR : RegisterClass<"ARM", [i32, v16i1, v8i1, v4i1, v2i1], 32, (add VPR)> {
//  let CopyCost = -1;  // Don't allow copying of status registers.
}
```
- EN: Defines TableGen record `VCCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 418-422
```tablegen
// FPSCR, when the flags at the top of it are used as the input or
// output to an instruction such as MVE VADC.
def cl_FPSCR_NZCV : RegisterClass<"ARM", [i32], 32, (add FPSCR_NZCV)> {
  let CopyCost = -1;
}
```
- EN: Defines TableGen record `cl_FPSCR_NZCV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `cl_FPSCR_NZCV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 424-429
```tablegen
// This RegisterClass is required to add FPSCR and FPEXC into a calling
// convention.
def FP_STATUS_REGS : RegisterClass<"ARM", [i32], 32, (add FPSCR, FPEXC)> {
  let CopyCost = -1;  // Don't allow copying of status registers.
  let isAllocatable = 0;
}
```
- EN: Defines TableGen record `FP_STATUS_REGS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FP_STATUS_REGS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 431-445
```tablegen
// Scalar single precision floating point register class..
// FIXME: Allocation order changed to s0, s2, ... or s0, s4, ... as a quick hack
// to avoid partial-write dependencies on D or Q (depending on platform)
// registers (S registers are renamed as portions of D/Q registers).
def SPR : RegisterClass<"ARM", [f32], 32, (sequence "S%u", 0, 31)> {
  let AltOrders = [(add (decimate SPR, 2), SPR),
                   (add (decimate SPR, 4),
                        (decimate SPR, 2),
                        (decimate (rotl SPR, 1), 4),
                        (decimate (rotl SPR, 1), 2))];
  let AltOrderSelect = [{
    return 1 + MF.getSubtarget<ARMSubtarget>().useStride4VFPs();
  }];
  let DiagnosticString = "operand must be a register in range [s0, s31]";
}
```
- EN: Defines TableGen record `SPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 447-457
```tablegen
def HPR : RegisterClass<"ARM", [f16, bf16], 32, (sequence "S%u", 0, 31)> {
  let AltOrders = [(add (decimate HPR, 2), SPR),
                   (add (decimate HPR, 4),
                        (decimate HPR, 2),
                        (decimate (rotl HPR, 1), 4),
                        (decimate (rotl HPR, 1), 2))];
  let AltOrderSelect = [{
    return 1 + MF.getSubtarget<ARMSubtarget>().useStride4VFPs();
  }];
  let DiagnosticString = "operand must be a register in range [s0, s31]";
}
```
- EN: Defines TableGen record `HPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 459-463
```tablegen
// Subset of SPR which can be used as a source of NEON scalars for 16-bit
// operations
def SPR_8 : RegisterClass<"ARM", [f32], 32, (sequence "S%u", 0, 15)> {
  let DiagnosticString = "operand must be a register in range [s0, s15]";
}
```
- EN: Defines TableGen record `SPR_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SPR_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 465-479
```tablegen
// Scalar double precision floating point / generic 64-bit vector register
// class.
// ARM requires only word alignment for double. It's more performant if it
// is double-word alignment though.
def DPR : RegisterClass<"ARM", [f64, v8i8, v4i16, v2i32, v1i64, v2f32, v4f16, v4bf16], 64,
                        (sequence "D%u", 0, 31)> {
  // Allocate non-VFP2 registers D16-D31 first, and prefer even registers on
  // Darwin platforms.
  let AltOrders = [(rotl DPR, 16),
                   (add (decimate (rotl DPR, 16), 2), (rotl DPR, 16))];
  let AltOrderSelect = [{
    return 1 + MF.getSubtarget<ARMSubtarget>().useStride4VFPs();
  }];
  let DiagnosticType = "DPR";
}
```
- EN: Defines TableGen record `DPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 481-486
```tablegen
// Scalar single and double precision floating point and VPR register class,
// this is only used for parsing, don't use it anywhere else as the size and
// types don't match!
def FPWithVPR : RegisterClass<"ARM", [f32], 32, (add SPR, DPR, VPR)> {
    let isAllocatable = 0;
}
```
- EN: Defines TableGen record `FPWithVPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FPWithVPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 488-493
```tablegen
// Subset of DPR that are accessible with VFP2 (and so that also have
// 32-bit SPR subregs).
def DPR_VFP2 : RegisterClass<"ARM", [f64, v8i8, v4i16, v2i32, v1i64, v2f32, v4f16, v4bf16], 64,
                             (trunc DPR, 16)> {
  let DiagnosticString = "operand must be a register in range [d0, d15]";
}
```
- EN: Defines TableGen record `DPR_VFP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DPR_VFP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 495-500
```tablegen
// Subset of DPR which can be used as a source of NEON scalars for 16-bit
// operations
def DPR_8 : RegisterClass<"ARM", [f64, v8i8, v4i16, v2i32, v1i64, v2f32, v4f16, v4bf16], 64,
                          (trunc DPR, 8)> {
  let DiagnosticString = "operand must be a register in range [d0, d7]";
}
```
- EN: Defines TableGen record `DPR_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DPR_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 502-511
```tablegen
// Generic 128-bit vector register class.
def QPR : RegisterClass<"ARM", [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64, v8f16, v8bf16], 128,
                        (sequence "Q%u", 0, 15)> {
  // Allocate non-VFP2 aliases Q8-Q15 first.
  let AltOrders = [(rotl QPR, 8), (trunc QPR, 8)];
  let AltOrderSelect = [{
    return 1 + MF.getSubtarget<ARMSubtarget>().hasMVEIntegerOps();
  }];
  let DiagnosticString = "operand must be a register in range [q0, q15]";
}
```
- EN: Defines TableGen record `QPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `QPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 513-517
```tablegen
// Subset of QPR that have 32-bit SPR subregs.
def QPR_VFP2 : RegisterClass<"ARM", [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
                             128, (trunc QPR, 8)> {
  let DiagnosticString = "operand must be a register in range [q0, q7]";
}
```
- EN: Defines TableGen record `QPR_VFP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `QPR_VFP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 519-523
```tablegen
// Subset of QPR that have DPR_8 and SPR_8 subregs.
def QPR_8 : RegisterClass<"ARM", [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
                           128, (trunc QPR, 4)> {
  let DiagnosticString = "operand must be a register in range [q0, q3]";
}
```
- EN: Defines TableGen record `QPR_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `QPR_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 525-529
```tablegen
// MVE 128-bit vector register class. This class is only really needed for
// parsing assembly, since we still have to truncate the register set in the QPR
// class anyway.
def MQPR : RegisterClass<"ARM", [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64, v8f16],
                         128, (trunc QPR, 8)>;
```
- EN: Declares reusable TableGen class `is` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `is`，通常用于抽象共享字段、谓词或编码结构。

### Lines 531-537
```tablegen
// Pseudo-registers representing odd-even pairs of D registers. The even-odd
// pairs are already represented by the Q registers.
// These are needed by NEON instructions requiring two consecutive D registers.
// There is no D31_D0 register as that is always an UNPREDICTABLE encoding.
def TuplesOE2D : RegisterTuples<[dsub_0, dsub_1],
                                [(decimate (shl DPR, 1), 2),
                                 (decimate (shl DPR, 2), 2)]>;
```
- EN: Defines TableGen record `TuplesOE2D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TuplesOE2D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 539-550
```tablegen
// Register class representing a pair of consecutive D registers.
// Use the Q registers for the even-odd pairs.
def DPair : RegisterClass<"ARM", [v16i8, v8i16, v4i32, v2i64, v4f32, v2f64],
                          128, (interleave QPR, TuplesOE2D)> {
  // Allocate starting at non-VFP2 registers D16-D31 first.
  // Prefer even-odd pairs as they are easier to copy.
  let AltOrders = [(add (rotl QPR, 8),  (rotl DPair, 16)),
                   (add (trunc QPR, 8), (trunc DPair, 16))];
  let AltOrderSelect = [{
    return 1 + MF.getSubtarget<ARMSubtarget>().hasMVEIntegerOps();
  }];
}
```
- EN: Declares reusable TableGen class `representing` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `representing`，通常用于抽象共享字段、谓词或编码结构。

### Lines 552-556
```tablegen
// Pseudo-registers representing even-odd pairs of GPRs from R1 to R13/SP.
// These are needed by instructions (e.g. ldrexd/strexd) requiring even-odd GPRs.
def Tuples2Rnosp : RegisterTuples<[gsub_0, gsub_1],
                                  [(add R0, R2, R4, R6, R8, R10),
                                   (add R1, R3, R5, R7, R9, R11)]>;
```
- EN: Defines TableGen record `Tuples2Rnosp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples2Rnosp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 558-559
```tablegen
def Tuples2Rsp   : RegisterTuples<[gsub_0, gsub_1],
                                  [(add R12), (add SP)]>;
```
- EN: Defines TableGen record `Tuples2Rsp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples2Rsp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 561-564
```tablegen
// Register class representing a pair of even-odd GPRs.
def GPRPair : RegisterClass<"ARM", [untyped], 64, (add Tuples2Rnosp, Tuples2Rsp)> {
  let Size = 64; // 2 x 32 bits, we have no predefined type of that size.
}
```
- EN: Declares reusable TableGen class `representing` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `representing`，通常用于抽象共享字段、谓词或编码结构。

### Lines 566-569
```tablegen
// Register class representing a pair of even-odd GPRs, except (R12, SP).
def GPRPairnosp : RegisterClass<"ARM", [untyped], 64, (add Tuples2Rnosp)> {
  let Size = 64; // 2 x 32 bits, we have no predefined type of that size.
}
```
- EN: Declares reusable TableGen class `representing` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `representing`，通常用于抽象共享字段、谓词或编码结构。

### Lines 571-575
```tablegen
// Pseudo-registers representing 3 consecutive D registers.
def Tuples3D : RegisterTuples<[dsub_0, dsub_1, dsub_2],
                              [(shl DPR, 0),
                               (shl DPR, 1),
                               (shl DPR, 2)]>;
```
- EN: Defines TableGen record `Tuples3D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples3D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 577-580
```tablegen
// 3 consecutive D registers.
def DTriple : RegisterClass<"ARM", [untyped], 64, (add Tuples3D)> {
  let Size = 192; // 3 x 64 bits, we have no predefined type of that size.
}
```
- EN: Defines TableGen record `DTriple` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DTriple`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 582-585
```tablegen
// Pseudo 256-bit registers to represent pairs of Q registers. These should
// never be present in the emitted code.
// These are used for NEON load / store instructions, e.g., vld4, vst3.
def Tuples2Q : RegisterTuples<[qsub_0, qsub_1], [(shl QPR, 0), (shl QPR, 1)]>;
```
- EN: Defines TableGen record `Tuples2Q` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples2Q`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 587-593
```tablegen
// Pseudo 256-bit vector register class to model pairs of Q registers
// (4 consecutive D registers).
def QQPR : RegisterClass<"ARM", [v4i64], 256, (add Tuples2Q)> {
  // Allocate non-VFP2 aliases first.
  let AltOrders = [(rotl QQPR, 8)];
  let AltOrderSelect = [{ return 1; }];
}
```
- EN: Declares reusable TableGen class `to` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `to`，通常用于抽象共享字段、谓词或编码结构。

### Lines 595-596
```tablegen
// Same as QQPR but for MVE, containing the 7 register pairs made up from Q0-Q7.
def MQQPR : RegisterClass<"ARM", [v4i64], 256, (trunc QQPR, 7)>;
```
- EN: Defines TableGen record `MQQPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MQQPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 598-603
```tablegen
// Tuples of 4 D regs that isn't also a pair of Q regs.
def TuplesOE4D : RegisterTuples<[dsub_0, dsub_1, dsub_2, dsub_3],
                                [(decimate (shl DPR, 1), 2),
                                 (decimate (shl DPR, 2), 2),
                                 (decimate (shl DPR, 3), 2),
                                 (decimate (shl DPR, 4), 2)]>;
```
- EN: Defines TableGen record `TuplesOE4D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TuplesOE4D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 605-607
```tablegen
// 4 consecutive D registers.
def DQuad : RegisterClass<"ARM", [v4i64], 256,
                          (interleave Tuples2Q, TuplesOE4D)>;
```
- EN: Defines TableGen record `DQuad` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DQuad`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 609-611
```tablegen
// Pseudo 512-bit registers to represent four consecutive Q registers.
def Tuples2QQ : RegisterTuples<[qqsub_0, qqsub_1],
                               [(shl QQPR, 0), (shl QQPR, 2)]>;
```
- EN: Defines TableGen record `Tuples2QQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples2QQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 613-619
```tablegen
// Pseudo 512-bit vector register class to model 4 consecutive Q registers
// (8 consecutive D registers).
def QQQQPR : RegisterClass<"ARM", [v8i64], 256, (add Tuples2QQ)> {
  // Allocate non-VFP2 aliases first.
  let AltOrders = [(rotl QQQQPR, 8)];
  let AltOrderSelect = [{ return 1; }];
}
```
- EN: Declares reusable TableGen class `to` for `ARMRegisterInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMRegisterInfo` 声明可复用的 TableGen 类 `to`，通常用于抽象共享字段、谓词或编码结构。

### Lines 621-622
```tablegen
// Same as QQPR but for MVE, containing the 5 register quads made up from Q0-Q7.
def MQQQQPR : RegisterClass<"ARM", [v8i64], 256, (trunc QQQQPR, 5)>;
```
- EN: Defines TableGen record `MQQQQPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MQQQQPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 625-628
```tablegen
// Pseudo-registers representing 2-spaced consecutive D registers.
def Tuples2DSpc : RegisterTuples<[dsub_0, dsub_2],
                                 [(shl DPR, 0),
                                  (shl DPR, 2)]>;
```
- EN: Defines TableGen record `Tuples2DSpc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples2DSpc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 630-631
```tablegen
// Spaced pairs of D registers.
def DPairSpc : RegisterClass<"ARM", [v2i64], 64, (add Tuples2DSpc)>;
```
- EN: Defines TableGen record `DPairSpc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DPairSpc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 633-636
```tablegen
def Tuples3DSpc : RegisterTuples<[dsub_0, dsub_2, dsub_4],
                                 [(shl DPR, 0),
                                  (shl DPR, 2),
                                  (shl DPR, 4)]>;
```
- EN: Defines TableGen record `Tuples3DSpc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples3DSpc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 638-641
```tablegen
// Spaced triples of D registers.
def DTripleSpc : RegisterClass<"ARM", [untyped], 64, (add Tuples3DSpc)> {
  let Size = 192; // 3 x 64 bits, we have no predefined type of that size.
}
```
- EN: Defines TableGen record `DTripleSpc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DTripleSpc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 643-647
```tablegen
def Tuples4DSpc : RegisterTuples<[dsub_0, dsub_2, dsub_4, dsub_6],
                                 [(shl DPR, 0),
                                  (shl DPR, 2),
                                  (shl DPR, 4),
                                  (shl DPR, 6)]>;
```
- EN: Defines TableGen record `Tuples4DSpc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Tuples4DSpc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 649-650
```tablegen
// Spaced quads of D registers.
def DQuadSpc : RegisterClass<"ARM", [v4i64], 64, (add Tuples3DSpc)>;
```
- EN: Defines TableGen record `DQuadSpc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DQuadSpc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 652-653
```tablegen
// FP context payload
def FPCXTRegs : RegisterClass<"ARM", [i32], 32, (add FPCXTNS)>;
```
- EN: Defines TableGen record `FPCXTRegs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FPCXTRegs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: register definitions, allocation constraints, and register utilities.
  - CN: 核心职责：寄存器定义、分配约束以及寄存器工具。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。

## Dependencies / 依赖关系

- EN: TableGen includes: `ARMSystemRegister.td`.
  - CN: TableGen 包含项：`ARMSystemRegister.td`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
