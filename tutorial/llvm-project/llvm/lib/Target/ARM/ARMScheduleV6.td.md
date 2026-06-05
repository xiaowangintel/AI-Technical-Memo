# ARMScheduleV6.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMScheduleV6.td`
- Repository: `llvm-project`
- Purpose (EN): This file defines the itinerary class data for the ARM v6 processors.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMScheduleV6`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- ARMScheduleV6.td - ARM v6 Scheduling Definitions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the itinerary class data for the ARM v6 processors.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-16
```tablegen
// Model based on ARM1176
//
// Functional Units
def V6_Pipe : FuncUnit; // pipeline
```
- EN: Defines TableGen record `V6_Pipe` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `V6_Pipe`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 18-35
```tablegen
// Scheduling information derived from "ARM1176JZF-S Technical Reference Manual"
//
def ARMV6Itineraries : ProcessorItineraries<
  [V6_Pipe], [], [
  //
  // No operand cycles
  InstrItinData<IIC_iALUx    , [InstrStage<1, [V6_Pipe]>]>,
  //
  // Binary Instructions that produce a result
  InstrItinData<IIC_iALUi    , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  InstrItinData<IIC_iALUr    , [InstrStage<1, [V6_Pipe]>], [2, 2, 2]>,
  InstrItinData<IIC_iALUsi   , [InstrStage<1, [V6_Pipe]>], [2, 2, 1]>,
  InstrItinData<IIC_iALUsr   , [InstrStage<2, [V6_Pipe]>], [3, 3, 2, 1]>,
  //
  // Bitwise Instructions that produce a result
  InstrItinData<IIC_iBITi    , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  InstrItinData<IIC_iBITr    , [InstrStage<1, [V6_Pipe]>], [2, 2, 2]>,
  InstrItinData<IIC_iBITsi   , [InstrStage<1, [V6_Pipe]>], [2, 2, 1]>,
```
- EN: Defines TableGen record `ARMV6Itineraries` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMV6Itineraries`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 36-53
```tablegen
  InstrItinData<IIC_iBITsr   , [InstrStage<2, [V6_Pipe]>], [3, 3, 2, 1]>,
  //
  // Unary Instructions that produce a result
  InstrItinData<IIC_iUNAr    , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  InstrItinData<IIC_iUNAsi   , [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  //
  // Zero and sign extension instructions
  InstrItinData<IIC_iEXTr    , [InstrStage<1, [V6_Pipe]>], [1, 1]>,
  InstrItinData<IIC_iEXTAr   , [InstrStage<1, [V6_Pipe]>], [2, 2, 1]>,
  InstrItinData<IIC_iEXTAsr  , [InstrStage<2, [V6_Pipe]>], [3, 3, 2, 1]>,
  //
  // Compare instructions
  InstrItinData<IIC_iCMPi    , [InstrStage<1, [V6_Pipe]>], [2]>,
  InstrItinData<IIC_iCMPr    , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  InstrItinData<IIC_iCMPsi   , [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  InstrItinData<IIC_iCMPsr   , [InstrStage<2, [V6_Pipe]>], [3, 2, 1]>,
  //
  // Test instructions
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 54-71
```tablegen
  InstrItinData<IIC_iTSTi    , [InstrStage<1, [V6_Pipe]>], [2]>,
  InstrItinData<IIC_iTSTr    , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  InstrItinData<IIC_iTSTsi   , [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  InstrItinData<IIC_iTSTsr   , [InstrStage<2, [V6_Pipe]>], [3, 2, 1]>,
  //
  // Move instructions, unconditional
  InstrItinData<IIC_iMOVi    , [InstrStage<1, [V6_Pipe]>], [2]>,
  InstrItinData<IIC_iMOVr    , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  InstrItinData<IIC_iMOVsi   , [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  InstrItinData<IIC_iMOVsr   , [InstrStage<2, [V6_Pipe]>], [3, 2, 1]>,
  InstrItinData<IIC_iMOVix2  , [InstrStage<1, [V6_Pipe]>,
                                InstrStage<1, [V6_Pipe]>], [2]>,
  InstrItinData<IIC_iMOVix2addpc,[InstrStage<1, [V6_Pipe]>,
                                  InstrStage<1, [V6_Pipe]>,
                                  InstrStage<1, [V6_Pipe]>], [3]>,
  InstrItinData<IIC_iMOVix2ld , [InstrStage<1, [V6_Pipe]>,
                                 InstrStage<1, [V6_Pipe]>,
                                 InstrStage<1, [V6_Pipe]>], [5]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 72-85
```tablegen
  //
  // Move instructions, conditional
  InstrItinData<IIC_iCMOVi   , [InstrStage<1, [V6_Pipe]>], [3]>,
  InstrItinData<IIC_iCMOVr   , [InstrStage<1, [V6_Pipe]>], [3, 2]>,
  InstrItinData<IIC_iCMOVsi  , [InstrStage<1, [V6_Pipe]>], [3, 1]>,
  InstrItinData<IIC_iCMOVsr  , [InstrStage<1, [V6_Pipe]>], [4, 2, 1]>,
  InstrItinData<IIC_iCMOVix2 , [InstrStage<1, [V6_Pipe]>,
                                InstrStage<1, [V6_Pipe]>], [4]>,
  //
  // MVN instructions
  InstrItinData<IIC_iMVNi    , [InstrStage<1, [V6_Pipe]>], [2]>,
  InstrItinData<IIC_iMVNr    , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  InstrItinData<IIC_iMVNsi   , [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  InstrItinData<IIC_iMVNsr   , [InstrStage<2, [V6_Pipe]>], [3, 2, 1]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 87-94
```tablegen
  // Integer multiply pipeline
  //
  InstrItinData<IIC_iMUL16   , [InstrStage<1, [V6_Pipe]>], [4, 1, 1]>,
  InstrItinData<IIC_iMAC16   , [InstrStage<1, [V6_Pipe]>], [4, 1, 1, 2]>,
  InstrItinData<IIC_iMUL32   , [InstrStage<2, [V6_Pipe]>], [5, 1, 1]>,
  InstrItinData<IIC_iMAC32   , [InstrStage<2, [V6_Pipe]>], [5, 1, 1, 2]>,
  InstrItinData<IIC_iMUL64   , [InstrStage<3, [V6_Pipe]>], [6, 1, 1]>,
  InstrItinData<IIC_iMAC64   , [InstrStage<3, [V6_Pipe]>], [6, 1, 1, 2]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 96-113
```tablegen
  // Integer load pipeline
  //
  // Immediate offset
  InstrItinData<IIC_iLoad_i   , [InstrStage<1, [V6_Pipe]>], [4, 1]>,
  InstrItinData<IIC_iLoad_bh_i, [InstrStage<1, [V6_Pipe]>], [4, 1]>,
  InstrItinData<IIC_iLoad_d_i , [InstrStage<1, [V6_Pipe]>], [4, 1]>,
  //
  // Register offset
  InstrItinData<IIC_iLoad_r   , [InstrStage<1, [V6_Pipe]>], [4, 1, 1]>,
  InstrItinData<IIC_iLoad_bh_r, [InstrStage<1, [V6_Pipe]>], [4, 1, 1]>,
  InstrItinData<IIC_iLoad_d_r , [InstrStage<1, [V6_Pipe]>], [4, 1, 1]>,
  //
  // Scaled register offset, issues over 2 cycles
  InstrItinData<IIC_iLoad_si   , [InstrStage<2, [V6_Pipe]>], [5, 2, 1]>,
  InstrItinData<IIC_iLoad_bh_si, [InstrStage<2, [V6_Pipe]>], [5, 2, 1]>,
  //
  // Immediate offset with update
  InstrItinData<IIC_iLoad_iu   , [InstrStage<1, [V6_Pipe]>], [4, 2, 1]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 114-123
```tablegen
  InstrItinData<IIC_iLoad_bh_iu, [InstrStage<1, [V6_Pipe]>], [4, 2, 1]>,
  //
  // Register offset with update
  InstrItinData<IIC_iLoad_ru   , [InstrStage<1, [V6_Pipe]>], [4, 2, 1, 1]>,
  InstrItinData<IIC_iLoad_bh_ru, [InstrStage<1, [V6_Pipe]>], [4, 2, 1, 1]>,
  InstrItinData<IIC_iLoad_d_ru , [InstrStage<1, [V6_Pipe]>], [4, 2, 1, 1]>,
  //
  // Scaled register offset with update, issues over 2 cycles
  InstrItinData<IIC_iLoad_siu,   [InstrStage<2, [V6_Pipe]>], [5, 2, 2, 1]>,
  InstrItinData<IIC_iLoad_bh_siu,[InstrStage<2, [V6_Pipe]>], [5, 2, 2, 1]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 125-134
```tablegen
  //
  // Load multiple, def is the 5th operand.
  InstrItinData<IIC_iLoad_m  , [InstrStage<3, [V6_Pipe]>], [1, 1, 1, 1, 4]>,
  //
  // Load multiple + update, defs are the 1st and 5th operands.
  InstrItinData<IIC_iLoad_mu , [InstrStage<3, [V6_Pipe]>], [2, 1, 1, 1, 4]>,
  //
  // Load multiple plus branch
  InstrItinData<IIC_iLoad_mBr, [InstrStage<3, [V6_Pipe]>,
                                InstrStage<1, [V6_Pipe]>], [1, 2, 1, 1, 4]>,
```
- EN: Defines TableGen record `is` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `is`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 136-139
```tablegen
  //
  // iLoadi + iALUr for t2LDRpci_pic.
  InstrItinData<IIC_iLoadiALU, [InstrStage<1, [V6_Pipe]>,
                                InstrStage<1, [V6_Pipe]>], [3, 1]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 141-147
```tablegen
  //
  // Pop, def is the 3rd operand.
  InstrItinData<IIC_iPop     , [InstrStage<3, [V6_Pipe]>], [1, 1, 4]>,
  //
  // Pop + branch, def is the 3rd operand.
  InstrItinData<IIC_iPop_Br,   [InstrStage<3, [V6_Pipe]>,
                                InstrStage<1, [V6_Pipe]>], [1, 2, 4]>,
```
- EN: Defines TableGen record `is` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `is`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 149-166
```tablegen
  // Integer store pipeline
  //
  // Immediate offset
  InstrItinData<IIC_iStore_i   , [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  InstrItinData<IIC_iStore_bh_i, [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  InstrItinData<IIC_iStore_d_i , [InstrStage<1, [V6_Pipe]>], [2, 1]>,
  //
  // Register offset
  InstrItinData<IIC_iStore_r   , [InstrStage<1, [V6_Pipe]>], [2, 1, 1]>,
  InstrItinData<IIC_iStore_bh_r, [InstrStage<1, [V6_Pipe]>], [2, 1, 1]>,
  InstrItinData<IIC_iStore_d_r , [InstrStage<1, [V6_Pipe]>], [2, 1, 1]>,
  //
  // Scaled register offset, issues over 2 cycles
  InstrItinData<IIC_iStore_si   , [InstrStage<2, [V6_Pipe]>], [2, 2, 1]>,
  InstrItinData<IIC_iStore_bh_si, [InstrStage<2, [V6_Pipe]>], [2, 2, 1]>,
  //
  // Immediate offset with update
  InstrItinData<IIC_iStore_iu   , [InstrStage<1, [V6_Pipe]>], [2, 2, 1]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 167-182
```tablegen
  InstrItinData<IIC_iStore_bh_iu, [InstrStage<1, [V6_Pipe]>], [2, 2, 1]>,
  //
  // Register offset with update
  InstrItinData<IIC_iStore_ru,   [InstrStage<1, [V6_Pipe]>], [2, 2, 1, 1]>,
  InstrItinData<IIC_iStore_bh_ru,[InstrStage<1, [V6_Pipe]>], [2, 2, 1, 1]>,
  InstrItinData<IIC_iStore_d_ru, [InstrStage<1, [V6_Pipe]>], [2, 2, 1, 1]>,
  //
  // Scaled register offset with update, issues over 2 cycles
  InstrItinData<IIC_iStore_siu,   [InstrStage<2, [V6_Pipe]>], [2, 2, 2, 1]>,
  InstrItinData<IIC_iStore_bh_siu,[InstrStage<2, [V6_Pipe]>], [2, 2, 2, 1]>,
  //
  // Store multiple
  InstrItinData<IIC_iStore_m  , [InstrStage<3, [V6_Pipe]>]>,
  //
  // Store multiple + update
  InstrItinData<IIC_iStore_mu , [InstrStage<3, [V6_Pipe]>], [2]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 184-187
```tablegen
  // Branch
  //
  // no delay slots, so the latency of a branch is unimportant
  InstrItinData<IIC_Br      , [InstrStage<1, [V6_Pipe]>]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 189-206
```tablegen
  // VFP
  // Issue through integer pipeline, and execute in NEON unit. We assume
  // RunFast mode so that NFP pipeline is used for single-precision when
  // possible.
  //
  // FP Special Register to Integer Register File Move
  InstrItinData<IIC_fpSTAT , [InstrStage<1, [V6_Pipe]>], [3]>,
  //
  // Single-precision FP Unary
  InstrItinData<IIC_fpUNA32 , [InstrStage<1, [V6_Pipe]>], [5, 2]>,
  //
  // Double-precision FP Unary
  InstrItinData<IIC_fpUNA64 , [InstrStage<1, [V6_Pipe]>], [5, 2]>,
  //
  // Single-precision FP Compare
  InstrItinData<IIC_fpCMP32 , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  //
  // Double-precision FP Compare
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 207-224
```tablegen
  InstrItinData<IIC_fpCMP64 , [InstrStage<1, [V6_Pipe]>], [2, 2]>,
  //
  // Single to Double FP Convert
  InstrItinData<IIC_fpCVTSD , [InstrStage<1, [V6_Pipe]>], [5, 2]>,
  //
  // Double to Single FP Convert
  InstrItinData<IIC_fpCVTDS , [InstrStage<1, [V6_Pipe]>], [5, 2]>,
  //
  // Single-Precision FP to Integer Convert
  InstrItinData<IIC_fpCVTSI , [InstrStage<1, [V6_Pipe]>], [9, 2]>,
  //
  // Double-Precision FP to Integer Convert
  InstrItinData<IIC_fpCVTDI , [InstrStage<1, [V6_Pipe]>], [9, 2]>,
  //
  // Integer to Single-Precision FP Convert
  InstrItinData<IIC_fpCVTIS , [InstrStage<1, [V6_Pipe]>], [9, 2]>,
  //
  // Integer to Double-Precision FP Convert
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 225-242
```tablegen
  InstrItinData<IIC_fpCVTID , [InstrStage<1, [V6_Pipe]>], [9, 2]>,
  //
  // Single-precision FP ALU
  InstrItinData<IIC_fpALU32 , [InstrStage<1, [V6_Pipe]>], [9, 2, 2]>,
  //
  // Double-precision FP ALU
  InstrItinData<IIC_fpALU64 , [InstrStage<1, [V6_Pipe]>], [9, 2, 2]>,
  //
  // Single-precision FP Multiply
  InstrItinData<IIC_fpMUL32 , [InstrStage<1, [V6_Pipe]>], [9, 2, 2]>,
  //
  // Double-precision FP Multiply
  InstrItinData<IIC_fpMUL64 , [InstrStage<2, [V6_Pipe]>], [9, 2, 2]>,
  //
  // Single-precision FP MAC
  InstrItinData<IIC_fpMAC32 , [InstrStage<1, [V6_Pipe]>], [9, 2, 2, 2]>,
  //
  // Double-precision FP MAC
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 243-260
```tablegen
  InstrItinData<IIC_fpMAC64 , [InstrStage<2, [V6_Pipe]>], [9, 2, 2, 2]>,
  //
  // Single-precision Fused FP MAC
  InstrItinData<IIC_fpFMAC32, [InstrStage<1, [V6_Pipe]>], [9, 2, 2, 2]>,
  //
  // Double-precision Fused FP MAC
  InstrItinData<IIC_fpFMAC64, [InstrStage<2, [V6_Pipe]>], [9, 2, 2, 2]>,
  //
  // Single-precision FP DIV
  InstrItinData<IIC_fpDIV32 , [InstrStage<15, [V6_Pipe]>], [20, 2, 2]>,
  //
  // Double-precision FP DIV
  InstrItinData<IIC_fpDIV64 , [InstrStage<29, [V6_Pipe]>], [34, 2, 2]>,
  //
  // Single-precision FP SQRT
  InstrItinData<IIC_fpSQRT32 , [InstrStage<15, [V6_Pipe]>], [20, 2, 2]>,
  //
  // Double-precision FP SQRT
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 261-278
```tablegen
  InstrItinData<IIC_fpSQRT64 , [InstrStage<29, [V6_Pipe]>], [34, 2, 2]>,
  //
  // Integer to Single-precision Move
  InstrItinData<IIC_fpMOVIS,  [InstrStage<1, [V6_Pipe]>], [10, 1]>,
  //
  // Integer to Double-precision Move
  InstrItinData<IIC_fpMOVID,  [InstrStage<1, [V6_Pipe]>], [10, 1, 1]>,
  //
  // Single-precision to Integer Move
  InstrItinData<IIC_fpMOVSI,  [InstrStage<1, [V6_Pipe]>], [10, 1]>,
  //
  // Double-precision to Integer Move
  InstrItinData<IIC_fpMOVDI,  [InstrStage<1, [V6_Pipe]>], [10, 10, 1]>,
  //
  // Single-precision FP Load
  InstrItinData<IIC_fpLoad32 , [InstrStage<1, [V6_Pipe]>], [5, 2, 2]>,
  //
  // Double-precision FP Load
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 279-296
```tablegen
  InstrItinData<IIC_fpLoad64 , [InstrStage<1, [V6_Pipe]>], [5, 2, 2]>,
  //
  // FP Load Multiple
  InstrItinData<IIC_fpLoad_m , [InstrStage<3, [V6_Pipe]>], [2, 1, 1, 5]>,
  //
  // FP Load Multiple + update
  InstrItinData<IIC_fpLoad_mu, [InstrStage<3, [V6_Pipe]>], [3, 2, 1, 1, 5]>,
  //
  // Single-precision FP Store
  InstrItinData<IIC_fpStore32 , [InstrStage<1, [V6_Pipe]>], [2, 2, 2]>,
  //
  // Double-precision FP Store
  // use FU_Issue to enforce the 1 load/store per cycle limit
  InstrItinData<IIC_fpStore64 , [InstrStage<1, [V6_Pipe]>], [2, 2, 2]>,
  //
  // FP Store Multiple
  InstrItinData<IIC_fpStore_m, [InstrStage<3, [V6_Pipe]>], [2, 2, 2, 2]>,
  //
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 297-299
```tablegen
  // FP Store Multiple + update
  InstrItinData<IIC_fpStore_mu,[InstrStage<3, [V6_Pipe]>], [3, 2, 2, 2, 2]>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
