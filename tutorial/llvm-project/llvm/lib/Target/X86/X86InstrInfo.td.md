# X86InstrInfo.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrInfo.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrInfo.td - Main X86 Instruction Properties --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the X86 properties of the instructions which are needed
// for code generation, machine code emission, and analysis.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// X86 Pattern fragments.
//
include "X86InstrFragments.td"
include "X86InstrFragmentsSIMD.td"

//===----------------------------------------------------------------------===//
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-40: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// X86 Predicate Definitions.
//
include "X86InstrPredicates.td"

//===----------------------------------------------------------------------===//
// X86 Operand Definitions.
//
include "X86InstrOperands.td"

//===----------------------------------------------------------------------===//
// X86 Instruction Format Definitions.
//
include "X86InstrFormats.td"

//===----------------------------------------------------------------------===//
// X86 Instruction utilities.
//
include "X86InstrUtils.td"

//===----------------------------------------------------------------------===//
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 41-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// Global ISel
//
include "X86InstrGISel.td"

//===----------------------------------------------------------------------===//
// Subsystems.
//===----------------------------------------------------------------------===//

include "X86InstrMisc.td"
include "X86InstrTBM.td"
include "X86InstrArithmetic.td"
include "X86InstrCMovSetCC.td"
include "X86InstrConditionalCompare.td"
include "X86InstrExtension.td"
include "X86InstrControl.td"
include "X86InstrShiftRotate.td"

// X87 Floating Point Stack.
include "X86InstrFPStack.td"

```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-80: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen

// FMA - Fused Multiply-Add support (requires FMA)
include "X86InstrFMA.td"

// XOP
include "X86InstrXOP.td"

// SSE, MMX and 3DNow! vector support.
include "X86InstrSSE.td"
include "X86InstrAVX512.td"
include "X86InstrAVX10.td"
include "X86InstrMMX.td"
include "X86Instr3DNow.td"

include "X86InstrVMX.td"
include "X86InstrSVM.td"
include "X86InstrSNP.td"
include "X86InstrTSX.td"
include "X86InstrSGX.td"
include "X86InstrTDX.td"
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 81-99: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen

// Key Locker instructions
include "X86InstrKL.td"

// AMX instructions
include "X86InstrAMX.td"

// RAO-INT instructions
include "X86InstrRAOINT.td"

// System instructions.
include "X86InstrSystem.td"

// Compiler Pseudo Instructions and Pat Patterns
include "X86InstrCompiler.td"
include "X86InstrVecCompiler.td"

// Assembler mnemonic/instruction aliases
include "X86InstrAsmAlias.td"
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- TableGen includes: X86InstrFragments.td, X86InstrFragmentsSIMD.td, X86InstrPredicates.td, X86InstrOperands.td, X86InstrFormats.td, X86InstrUtils.td, X86InstrGISel.td, X86InstrMisc.td. / TableGen 包含：X86InstrFragments.td, X86InstrFragmentsSIMD.td, X86InstrPredicates.td, X86InstrOperands.td, X86InstrFormats.td, X86InstrUtils.td, X86InstrGISel.td, X86InstrMisc.td。
- Additional TableGen include dependencies: 28 more files. / 额外 TableGen 包含依赖：还有 28 个文件。
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
