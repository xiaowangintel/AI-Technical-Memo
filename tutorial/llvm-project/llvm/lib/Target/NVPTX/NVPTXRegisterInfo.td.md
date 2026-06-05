# NVPTXRegisterInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXRegisterInfo.td`
- Repository: `llvm-project`
- Purpose (EN): Declarations that describe the PTX register file.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```tablegen
 1: //===-- NVPTXRegisterInfo.td - NVPTX Register defs ---------*- tablegen -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: //===----------------------------------------------------------------------===//
10: //  Declarations that describe the PTX register file
11: //===----------------------------------------------------------------------===//
12:
13: class NVPTXReg<string n> : Register<n> {
14:   let Namespace = "NVPTX";
15: }
16:
17: class NVPTXRegClass<list<ValueType> regTypes, int alignment, dag regList>
18:      : RegisterClass <"NVPTX", regTypes, alignment, regList>;
19:
20: //===----------------------------------------------------------------------===//
21: //  Registers
22: //===----------------------------------------------------------------------===//
23:
24: // Special Registers used as stack pointer
25: def VRFrame32         : NVPTXReg<"%SP">;
26: def VRFrame64         : NVPTXReg<"%SP">;
27: def VRFrameLocal32    : NVPTXReg<"%SPL">;
28: def VRFrameLocal64    : NVPTXReg<"%SPL">;
29:
30: // Special Registers used as the stack
31: def VRDepot  : NVPTXReg<"%Depot">;
32:
33: // We use virtual registers, but define a few physical registers here to keep
34: // SDAG and the MachineInstr layers happy.
35: foreach i = 0...4 in {
36:   def P#i  : NVPTXReg<"%p"#i>;  // Predicate
37:   def RS#i : NVPTXReg<"%rs"#i>; // 16-bit
38:   def R#i  : NVPTXReg<"%r"#i>;  // 32-bit
39:   def RL#i : NVPTXReg<"%rd"#i>; // 64-bit
40:   def RQ#i : NVPTXReg<"%rq"#i>; // 128-bit
41: }
42:
43: foreach i = 0...31 in {
44:   def ENVREG#i : NVPTXReg<"%envreg"#i>;
45: }
46:
47: //===----------------------------------------------------------------------===//
48: //  Register classes.
49: //  NOTE: if you add new vector types for a register, you must update
50: //        NVPTX::packed_types() in NVPTXUtilities.h accordingly!
51: //===----------------------------------------------------------------------===//
52: def B1 : NVPTXRegClass<[i1], 8, (add (sequence "P%u", 0, 4))>;
53: def B16 : NVPTXRegClass<[i16, f16, bf16], 16, (add (sequence "RS%u", 0, 4))>;
54: def B32 : NVPTXRegClass<[i32, v2f16, v2bf16, v2i16, v4i8, f32], 32,
55:                               (add (sequence "R%u", 0, 4),
56:                               VRFrame32, VRFrameLocal32)>;
57: def B64 : NVPTXRegClass<[i64, v2i32, v2f32, f64], 64,
58:                         (add (sequence "RL%u", 0, 4),
59:                          VRFrame64, VRFrameLocal64)>;
60: // 128-bit regs are not defined as general regs in NVPTX. They are used for inlineASM only.
```
- EN: This range uses TableGen DSL to describe records such as NVPTXReg, NVPTXRegClass, VRFrame32, VRFrame64; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 NVPTXReg、NVPTXRegClass、VRFrame32、VRFrame64 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 61-65
```tablegen
61: def B128 : NVPTXRegClass<[i128], 128, (add (sequence "RQ%u", 0, 4))>;
62:
63: // Read NVPTXRegisterInfo.cpp to see how VRFrame and VRDepot are used.
64: def SpecialRegs : NVPTXRegClass<[i32], 32, (add VRFrame32, VRFrameLocal32, VRDepot,
65:                                             (sequence "ENVREG%u", 0, 31))>;
```
- EN: This range uses TableGen DSL to describe records such as B128, SpecialRegs; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 B128、SpecialRegs 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: Register information tracks physical/register-bank constraints and target-specific register properties.
  - CN: 寄存器信息负责描述物理寄存器或寄存器银行约束，以及目标特定属性。
- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXReg, NVPTXRegClass, VRFrame32, VRFrame64, VRFrameLocal32, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXReg, NVPTXRegClass, VRFrame32, VRFrame64, VRFrameLocal32，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
- This file has no explicit textual includes; its effective dependencies come from surrounding generated or linked LLVM components.
- 该文件没有显式文本 include；其实际依赖主要来自周边生成代码或链接到的 LLVM 组件。
