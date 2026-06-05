# RISCVCallingConv.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVCallingConv.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for calling-convention assignments and lowering metadata for RISC-V. / 使用 TableGen 定义RISC-V 的调用约定分配规则与下降元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVCallingConv.td - Calling Conventions RISC-V ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This describes the calling conventions for the RISC-V architecture.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-20: TableGen record CSR_ILP32E_LP64E / TableGen 记录 CSR_ILP32E_LP64E
```tablegen

// The RISC-V calling convention is handled with custom code in
// RISCVISelLowering.cpp (CC_RISCV).

def CSR_ILP32E_LP64E : CalleeSavedRegs<(add X1, X8, X9)>;

def CSR_ILP32_LP64
    : CalleeSavedRegs<(add CSR_ILP32E_LP64E, (sequence "X%u", 18, 27))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 21-28: TableGen record CSR_ILP32F_LP64F / TableGen 记录 CSR_ILP32F_LP64F
```tablegen
def CSR_ILP32F_LP64F
    : CalleeSavedRegs<(add CSR_ILP32_LP64,
                       F8_F, F9_F, (sequence "F%u_F", 18, 27))>;

def CSR_ILP32D_LP64D
    : CalleeSavedRegs<(add CSR_ILP32_LP64,
                       F8_D, F9_D, (sequence "F%u_D", 18, 27))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 29-38: TableGen record CSR_ILP32_LP64_V / TableGen 记录 CSR_ILP32_LP64_V
```tablegen
defvar CSR_V = (add (sequence "V%u", 1, 7), (sequence "V%u", 24, 31),
                     V2M2, V4M2, V6M2, V24M2, V26M2, V28M2, V30M2,
                     V4M4, V24M4, V28M4, V24M8);

def CSR_ILP32_LP64_V
    : CalleeSavedRegs<(add CSR_ILP32_LP64, CSR_V)>;

def CSR_ILP32F_LP64F_V
    : CalleeSavedRegs<(add CSR_ILP32F_LP64F, CSR_V)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 39-46: TableGen record CSR_ILP32D_LP64D_V / TableGen 记录 CSR_ILP32D_LP64D_V
```tablegen
def CSR_ILP32D_LP64D_V
    : CalleeSavedRegs<(add CSR_ILP32D_LP64D, CSR_V)>;

// Needed for implementation of RISCVRegisterInfo::getNoPreservedMask()
def CSR_NoRegs : CalleeSavedRegs<(add)>;

def CSR_IPRA : CalleeSavedRegs<(add X1)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 47-54: TableGen record CSR_Interrupt / TableGen 记录 CSR_Interrupt
```tablegen
// Interrupt handler needs to save/restore all registers that are used,
// both Caller and Callee saved registers.
def CSR_Interrupt : CalleeSavedRegs<(add X1, (sequence "X%u", 5, 31))>;

// Same as CSR_Interrupt, but including all 32-bit FP registers.
def CSR_XLEN_F32_Interrupt: CalleeSavedRegs<(add CSR_Interrupt,
                                             (sequence "F%u_F", 0, 31))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 55-63: TableGen record CSR_XLEN_F64_Interrupt: / TableGen 记录 CSR_XLEN_F64_Interrupt:
```tablegen
// Same as CSR_Interrupt, but including all 64-bit FP registers.
def CSR_XLEN_F64_Interrupt: CalleeSavedRegs<(add CSR_Interrupt,
                                             (sequence "F%u_D", 0, 31))>;

defvar VREGS = (add (sequence "V%u", 0, 31),
                    (sequence "V%uM2", 0, 31, 2),
                    (sequence "V%uM4", 0, 31, 4),
                    (sequence "V%uM8", 0, 31, 8));
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 64-74: TableGen record CSR_XLEN_V_Interrupt: / TableGen 记录 CSR_XLEN_V_Interrupt:
```tablegen
// Same as CSR_Interrupt, but including all vector registers.
def CSR_XLEN_V_Interrupt: CalleeSavedRegs<(add CSR_Interrupt, VREGS)>;

// Same as CSR_Interrupt, but including all 32-bit FP registers and all vector
// registers.
def CSR_XLEN_F32_V_Interrupt: CalleeSavedRegs<(add CSR_XLEN_F32_Interrupt, VREGS)>;

// Same as CSR_Interrupt, but including all 64-bit FP registers and all vector
// registers.
def CSR_XLEN_F64_V_Interrupt: CalleeSavedRegs<(add CSR_XLEN_F64_Interrupt, VREGS)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 75-82: TableGen record CSR_Interrupt_RVE / TableGen 记录 CSR_Interrupt_RVE
```tablegen
// Same as CSR_Interrupt, but excluding X16-X31.
def CSR_Interrupt_RVE : CalleeSavedRegs<(sub CSR_Interrupt,
                                        (sequence "X%u", 16, 31))>;

// Same as CSR_XLEN_F32_Interrupt, but excluding X16-X31.
def CSR_XLEN_F32_Interrupt_RVE: CalleeSavedRegs<(sub CSR_XLEN_F32_Interrupt,
                                                 (sequence "X%u", 16, 31))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 83-90: TableGen record CSR_XLEN_F64_Interrupt_RVE: / TableGen 记录 CSR_XLEN_F64_Interrupt_RVE:
```tablegen
// Same as CSR_XLEN_F64_Interrupt, but excluding X16-X31.
def CSR_XLEN_F64_Interrupt_RVE: CalleeSavedRegs<(sub CSR_XLEN_F64_Interrupt,
                                                 (sequence "X%u", 16, 31))>;

// Same as CSR_XLEN_V_Interrupt, but excluding X16-X31.
def CSR_XLEN_V_Interrupt_RVE: CalleeSavedRegs<(sub CSR_XLEN_V_Interrupt,
                                               (sequence "X%u", 16, 31))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 91-98: TableGen record CSR_XLEN_F32_V_Interrupt_RVE: / TableGen 记录 CSR_XLEN_F32_V_Interrupt_RVE:
```tablegen
// Same as CSR_XLEN_F32_V_Interrupt, but excluding X16-X31.
def CSR_XLEN_F32_V_Interrupt_RVE: CalleeSavedRegs<(sub CSR_XLEN_F32_V_Interrupt,
                                                   (sequence "X%u", 16, 31))>;

// Same as CSR_XLEN_F64_V_Interrupt, but excluding X16-X31.
def CSR_XLEN_F64_V_Interrupt_RVE: CalleeSavedRegs<(sub CSR_XLEN_F64_V_Interrupt,
                                                   (sequence "X%u", 16, 31))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 99-101: TableGen record CSR_RT_MostRegs / TableGen 记录 CSR_RT_MostRegs
```tablegen
def CSR_RT_MostRegs : CalleeSavedRegs<(sub CSR_Interrupt, X6, X7, X28)>;
def CSR_RT_MostRegs_RVE : CalleeSavedRegs<(sub CSR_RT_MostRegs,
                                               (sequence "X%u", 16, 31))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Calling convention rules** / **调用约定规则**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
