# RISCV.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCV.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCV.td - Describe the RISC-V Target Machine ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 9-16: Included TableGen fragments / 引入的 TableGen 片段
```tablegen
include "llvm/Target/Target.td"

//===----------------------------------------------------------------------===//
// RISC-V subtarget features and instruction predicates.
//===----------------------------------------------------------------------===//

include "RISCVFeatures.td"
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 17-25: Included TableGen fragments / 引入的 TableGen 片段
```tablegen
//===----------------------------------------------------------------------===//
// RISC-V profiles supported.
//===----------------------------------------------------------------------===//

include "RISCVProfiles.td"

//===----------------------------------------------------------------------===//
// Named operands for CSR instructions.
//===----------------------------------------------------------------------===//
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 26-38: Included TableGen fragments / 引入的 TableGen 片段
```tablegen

include "RISCVSystemOperands.td"

//===----------------------------------------------------------------------===//
// Registers, calling conventions, instruction descriptions.
//===----------------------------------------------------------------------===//

include "RISCVRegisterInfo.td"
include "RISCVSchedule.td"
include "RISCVCallingConv.td"
include "RISCVInstrInfo.td"
include "GISel/RISCVRegisterBanks.td"
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 39-47: Included TableGen fragments / 引入的 TableGen 片段
```tablegen
//===----------------------------------------------------------------------===//
// Instruction predicates
//===----------------------------------------------------------------------===//

include "RISCVInstrPredicates.td"

//===----------------------------------------------------------------------===//
// RISC-V macro fusions.
//===----------------------------------------------------------------------===//
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 48-63: Included TableGen fragments / 引入的 TableGen 片段
```tablegen

include "RISCVMacroFusion.td"

//===----------------------------------------------------------------------===//
// RISC-V Scheduling Models
//===----------------------------------------------------------------------===//
include "RISCVSchedAndes45.td"
include "RISCVSchedGenericOOO.td"
include "RISCVSchedMIPSP8700.td"
include "RISCVSchedRocket.td"
include "RISCVSchedSiFive7.td"
include "RISCVSchedSiFiveP400.td"
include "RISCVSchedSiFiveP500.td"
include "RISCVSchedSiFiveP600.td"
include "RISCVSchedSiFiveP800.td"
include "RISCVSchedSpacemitX60.td"
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 64-71: Included TableGen fragments / 引入的 TableGen 片段
```tablegen
include "RISCVSchedSpacemitX100.td"
include "RISCVSchedSyntacoreSCR1.td"
include "RISCVSchedSyntacoreSCR345.td"
include "RISCVSchedSyntacoreSCR7.td"
include "RISCVSchedTTAscalonX.td"
include "RISCVSchedXiangShanNanHu.td"

//===----------------------------------------------------------------------===//
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 72-79: Included TableGen fragments / 引入的 TableGen 片段
```tablegen
// RISC-V processors supported.
//===----------------------------------------------------------------------===//

include "RISCVProcessors.td"

//===----------------------------------------------------------------------===//
// Pfm Counters
//===----------------------------------------------------------------------===//
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 80-89: Included TableGen fragments / 引入的 TableGen 片段
```tablegen

include "RISCVPfmCounters.td"

//===----------------------------------------------------------------------===//
// Define the RISC-V target.
//===----------------------------------------------------------------------===//

def RISCVInstrInfo : InstrInfo {
  let guessInstructionProperties = 0;
}
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 90-98: TableGen record RISCVAsmParser / TableGen 记录 RISCVAsmParser
```tablegen

def RISCVAsmParser : AsmParser {
  let ShouldEmitMatchRegisterAltName = 1;
  let AllowDuplicateRegisterNames = 1;
}

def RISCVAsmWriter : AsmWriter {
  int PassSubtarget = 1;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 99-107: Bulk record instantiation RISCV / 批量记录实例化 RISCV
```tablegen

defm : RemapAllTargetPseudoPointerOperands<GPR>;

def RISCV : Target {
  let InstructionSet = RISCVInstrInfo;
  let AssemblyParsers = [RISCVAsmParser];
  let AssemblyWriters = [RISCVAsmWriter];
  let AllowRegisterRenaming = 1;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- `llvm/Target/Target.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVFeatures.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVProfiles.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSystemOperands.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVRegisterInfo.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSchedule.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVCallingConv.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `GISel/RISCVRegisterBanks.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrPredicates.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVMacroFusion.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSchedAndes45.td` — Directly referenced by this file. / 该文件直接引用的依赖。
