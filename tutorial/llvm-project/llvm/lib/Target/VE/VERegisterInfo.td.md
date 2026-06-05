# VERegisterInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VERegisterInfo.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines physical registers, register classes, and related metadata in TableGen DSL.
  - **CN**: 使用 TableGen DSL 定义物理寄存器、寄存器类及其相关元数据。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- VERegisterInfo.td - VE Register defs ---------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-21
```tablegen

//===----------------------------------------------------------------------===//
//  Declarations that describe the VE register file
//===----------------------------------------------------------------------===//

class VEReg<bits<7> enc, string n, list<Register> subregs = [],
            list<string> altNames = [], list<Register> aliases = []>
        : Register<n, altNames> {
  let HWEncoding{15-7} = 0;
  let HWEncoding{6-0} = enc;
  let Namespace = "VE";
  let SubRegs = subregs;
  let Aliases = aliases;
}
```
- **EN**: Declares TableGen records such as `VEReg`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `VEReg` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 22-28
```tablegen

class VEMiscReg<bits<6> enc, string n>: Register<n> {
  let HWEncoding{15-6} = 0;
  let HWEncoding{5-0} = enc;
  let Namespace = "VE";
}

```
- **EN**: Declares TableGen records such as `VEMiscReg`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `VEMiscReg` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 29-38
```tablegen
class VEVecReg<bits<8> enc, string n, list<Register> subregs = [],
               list<string> altNames = [], list<Register> aliases = []>
        : Register<n, altNames> {
  let HWEncoding{15-8} = 0;
  let HWEncoding{7-0} = enc;
  let Namespace = "VE";
  let SubRegs = subregs;
  let Aliases = aliases;
}

```
- **EN**: Declares TableGen records such as `VEVecReg`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `VEVecReg` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 39-48
```tablegen
class VEMaskReg<bits<4> enc, string n, list<Register> subregs = [],
                list<string> altNames = [], list<Register> aliases = []>
        : Register<n, altNames> {
  let HWEncoding{15-4} = 0;
  let HWEncoding{3-0} = enc;
  let Namespace = "VE";
  let SubRegs = subregs;
  let Aliases = aliases;
}

```
- **EN**: Declares TableGen records such as `VEMaskReg`; this block specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `VEMaskReg` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 49-58
```tablegen
let Namespace = "VE" in {
  def sub_i32     : SubRegIndex<32, 32>;        // Low 32 bit (32..63)
  def sub_f32     : SubRegIndex<32>;            // High 32 bit (0..31)
  def sub_even    : SubRegIndex<64>;            // High 64 bit (0..63)
  def sub_odd     : SubRegIndex<64, 64>;        // Low 64 bit (64..127)
  def sub_vm_even : SubRegIndex<256>;           // High 256 bit (0..255)
  def sub_vm_odd  : SubRegIndex<256, 256>;      // Low 256 bit (256..511)
  def AsmName     : RegAltNameIndex;
}

```
- **EN**: Declares TableGen records such as `sub_i32`, `sub_f32`, `sub_even`, `sub_odd`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `sub_i32`, `sub_f32`, `sub_even`, `sub_odd`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 59-67
```tablegen
//-----------------------------------------------------------------------------
// Miscellaneous Registers
//-----------------------------------------------------------------------------

def USRCC : VEMiscReg<0, "usrcc">;      // User clock counter
def PSW : VEMiscReg<1, "psw">;          // Program status word
def SAR : VEMiscReg<2, "sar">;          // Store address register
def PMMR : VEMiscReg<7, "pmmr">;        // Performance monitor mode register

```
- **EN**: Declares TableGen records such as `USRCC`, `PSW`, `SAR`, `PMMR` for the backend description.
- **CN**: 为后端描述声明了 `USRCC`, `PSW`, `SAR`, `PMMR` 等 TableGen 记录。

### Lines 68-75
```tablegen
// Performance monitor configuration registers
foreach I = 0-3 in
  def PMCR#I : VEMiscReg<!add(8,I), "pmcr"#I>;

// Performance monitor counter
foreach I = 0-14 in
  def PMC#I : VEMiscReg<!add(16,I), "pmc"#I>;

```
- **EN**: Declares TableGen records such as `PMCR`, `PMC` for the backend description.
- **CN**: 为后端描述声明了 `PMCR`, `PMC` 等 TableGen 记录。

### Lines 76-85
```tablegen
// Register classes.
def MISC : RegisterClass<"VE", [i64], 64,
                         (add USRCC, PSW, SAR, PMMR,
                              (sequence "PMCR%u", 0, 3),
                              (sequence "PMC%u", 0, 14))>;

//-----------------------------------------------------------------------------
// Instruction Counter Register
//-----------------------------------------------------------------------------

```
- **EN**: Declares TableGen records such as `MISC` for the backend description.
- **CN**: 为后端描述声明了 `MISC` 等 TableGen 记录。

### Lines 86-93
```tablegen
def IC : VEMiscReg<62, "ic">;

//-----------------------------------------------------------------------------
// Vector Length Register
//-----------------------------------------------------------------------------

def VL : VEMiscReg<63, "vl">;

```
- **EN**: Declares TableGen records such as `IC`, `VL` for the backend description.
- **CN**: 为后端描述声明了 `IC`, `VL` 等 TableGen 记录。

### Lines 94-100
```tablegen
// Register classes.
def VLS : RegisterClass<"VE", [i32], 64, (add VL)>;

//-----------------------------------------------------------------------------
// Generic Registers
//-----------------------------------------------------------------------------

```
- **EN**: Declares TableGen records such as `VLS` for the backend description.
- **CN**: 为后端描述声明了 `VLS` 等 TableGen 记录。

### Lines 101-113
```tablegen
let RegAltNameIndices = [AsmName] in {

// Generic integer registers - 32 bits wide
foreach I = 0-63 in
  def SW#I : VEReg<I, "sw"#I, [], ["s"#I]>, DwarfRegNum<[I]>;

// Generic floating point registers - 32 bits wide
//   NOTE: Mark SF#I as alias of SW#I temporary to avoid register allocation
//         problem.
foreach I = 0-63 in
  def SF#I : VEReg<I, "sf"#I, [], ["s"#I], [!cast<VEReg>("SW"#I)]>,
                   DwarfRegNum<[I]>;

```
- **EN**: Declares TableGen records such as `SW`, `SF`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SW`, `SF` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 114-124
```tablegen
// Generic integer registers - 64 bits wide
let SubRegIndices = [sub_i32, sub_f32], CoveredBySubRegs = 1 in {
  // Several registers have specific names, so add them to one of aliases.
  def SX8 : VEReg<8, "s8", [SW8, SF8], ["s8", "sl"]>, DwarfRegNum<[8]>;
  def SX9 : VEReg<9, "s9", [SW9, SF9], ["s9", "fp"]>, DwarfRegNum<[9]>;
  def SX10 : VEReg<10, "s10", [SW10, SF10], ["s10", "lr"]>, DwarfRegNum<[10]>;
  def SX11 : VEReg<11, "s11", [SW11, SF11], ["s11", "sp"]>, DwarfRegNum<[11]>;
  def SX14 : VEReg<14, "s14", [SW14, SF14], ["s14", "tp"]>, DwarfRegNum<[14]>;
  def SX15 : VEReg<15, "s15", [SW15, SF15], ["s15", "got"]>, DwarfRegNum<[15]>;
  def SX16 : VEReg<16, "s16", [SW16, SF16], ["s16", "plt"]>, DwarfRegNum<[16]>;

```
- **EN**: Declares TableGen records such as `SX8`, `SX9`, `SX10`, `SX11`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SX8`, `SX9`, `SX10`, `SX11`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 125-138
```tablegen
  // Other generic registers.
  foreach I = { 0-7, 12-13, 17-63 } in
    def SX#I : VEReg<I, "s"#I, [!cast<VEReg>("SW"#I), !cast<VEReg>("SF"#I)],
                     ["s"#I]>, DwarfRegNum<[I]>;
}

// Aliases of the S* registers used to hold 128-bit for values (long doubles).
// Following foreach represents something like:
//   def Q0 : VEReg<0, "q0", [SX0, SX1], ["s0"]>;
//   def Q1 : VEReg<2, "q2", [SX2, SX3], ["s2"]>;
//   ...
let SubRegIndices = [sub_even, sub_odd], CoveredBySubRegs = 1 in
foreach I = 0-31 in
  def Q#I : VEReg<!shl(I,1), "q"#I,
```
- **EN**: Declares TableGen records such as `SX`, `Q`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SX`, `Q` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 139-146
```tablegen
                  [!cast<VEReg>("SX"#!shl(I,1)),
                   !cast<VEReg>("SX"#!add(!shl(I,1),1))],
                  ["s"#!shl(I,1)]>;

// Vector registers - 64 bits wide 256 elements
foreach I = 0-63 in
  def V#I : VEVecReg<I, "v"#I, [], ["v"#I]>, DwarfRegNum<[!add(64,I)]>;

```
- **EN**: Declares TableGen records such as `V` for the backend description.
- **CN**: 为后端描述声明了 `V` 等 TableGen 记录。

### Lines 147-155
```tablegen
// Vector Index Register
def VIX : VEVecReg<255, "vix", [], ["vix"]>;

// Vector mask registers - 256 bits wide
let isConstant = true in
def VM0 : VEMaskReg<0, "vm0", [], ["vm0"]>, DwarfRegNum<[128]>;
foreach I = 1-15 in
  def VM#I : VEMaskReg<I, "vm"#I, [], ["vm"#I]>, DwarfRegNum<[!add(128,I)]>;

```
- **EN**: Declares TableGen records such as `VIX`, `VM0`, `VM`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `VIX`, `VM0`, `VM` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 156-166
```tablegen
// Aliases of VMs to use as a pair of two VM for packed instructions
let isConstant = true in
def VMP0 : VEMaskReg<0, "vm0", [], ["vm0"]>;

let SubRegIndices = [sub_vm_even, sub_vm_odd], CoveredBySubRegs = 1 in
foreach I = 1-7 in
  def VMP#I : VEMaskReg<!shl(I,1), "vmp"#I,
                        [!cast<VEMaskReg>("VM"#!shl(I,1)),
                         !cast<VEMaskReg>("VM"#!add(!shl(I,1),1))],
                        ["vm"#!shl(I,1)]>;

```
- **EN**: Declares TableGen records such as `VMP0`, `VMP`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `VMP0`, `VMP` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 167-180
```tablegen
} // RegAltNameIndices = [AsmName]

// Register classes.
//
// The register order is defined in terms of the preferred
// allocation order.
def I32 : RegisterClass<"VE", [i32], 32,
                        (add (sequence "SW%u", 0, 7),
                             (sequence "SW%u", 34, 63),
                             (sequence "SW%u", 8, 33))>;
def I64 : RegisterClass<"VE", [i64, f64], 64,
                        (add (sequence "SX%u", 0, 7),
                             (sequence "SX%u", 34, 63),
                             (sequence "SX%u", 8, 33))>;
```
- **EN**: Declares TableGen records such as `I32`, `I64` for the backend description.
- **CN**: 为后端描述声明了 `I32`, `I64` 等 TableGen 记录。

### Lines 181-189
```tablegen
def F32 : RegisterClass<"VE", [f32], 32,
                        (add (sequence "SF%u", 0, 7),
                             (sequence "SF%u", 34, 63),
                             (sequence "SF%u", 8, 33))>;
def F128 : RegisterClass<"VE", [f128], 128,
                        (add (sequence "Q%u", 0, 3),
                             (sequence "Q%u", 17, 31),
                             (sequence "Q%u", 4, 16))>;

```
- **EN**: Declares TableGen records such as `F32`, `F128` for the backend description.
- **CN**: 为后端描述声明了 `F32`, `F128` 等 TableGen 记录。

### Lines 190-196
```tablegen
def V64 : RegisterClass<"VE",
                        [v256f64, // default type for vector registers
                         v512i32, v512f32,
                         v256i64, v256i32, v256f32, /* v256f64, */], 64,
                        (add (sequence "V%u", 0, 63),
                             VIX)>;

```
- **EN**: Declares TableGen records such as `V64` for the backend description.
- **CN**: 为后端描述声明了 `V64` 等 TableGen 记录。

### Lines 197-199
```tablegen
// vm0 is reserved for always true
def VM : RegisterClass<"VE", [v256i1], 64, (sequence "VM%u", 0, 15)>;
def VM512 : RegisterClass<"VE", [v512i1], 64, (sequence "VMP%u", 0, 7)>;
```
- **EN**: Declares TableGen records such as `VM`, `VM512` for the backend description.
- **CN**: 为后端描述声明了 `VM`, `VM512` 等 TableGen 记录。

## Key Concepts / 关键概念

- **Register model / 寄存器模型**:
  - **EN**: Describes registers, classes, and allocation-facing details
  - **CN**: 描述寄存器、寄存器类以及面向分配器的细节
- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
