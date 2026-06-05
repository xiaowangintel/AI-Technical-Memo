# RISCVScheduleZb.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVScheduleZb.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Commentary and design intent / 注释与设计意图
```tablegen
//===- RISCVScheduleB.td - RISC-V Scheduling Definitions B -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

/// Define scheduler resources associated with def operands.
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 11-29: TableGen record WriteSHXADD / TableGen 记录 WriteSHXADD
```tablegen
// Zba extension
def WriteSHXADD      : SchedWrite; // sh1add/sh2add/sh3add
def WriteSHXADD32    : SchedWrite; // sh1add.uw/sh2add.uw/sh3add.uw

// Zbb extension
def WriteRotateImm   : SchedWrite;
def WriteRotateImm32 : SchedWrite;
def WriteRotateReg   : SchedWrite;
def WriteRotateReg32 : SchedWrite;
def WriteCLZ         : SchedWrite;
def WriteCLZ32       : SchedWrite;
def WriteCTZ         : SchedWrite;
def WriteCTZ32       : SchedWrite;
def WriteCPOP        : SchedWrite;
def WriteCPOP32      : SchedWrite;
def WriteREV8        : SchedWrite;
def WriteORCB        : SchedWrite;
def WriteIMinMax     : SchedWrite;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 30-44: TableGen record WriteCLMUL / TableGen 记录 WriteCLMUL
```tablegen
// Zbc extension
def WriteCLMUL       : SchedWrite; // CLMUL/CLMULR/CLMULH

// Zbs extension
def WriteSingleBit   : SchedWrite; // BCLR/BSET/BINV
def WriteSingleBitImm: SchedWrite; // BCLRI/BSETI/BINVI
def WriteBEXT : SchedWrite; // BEXT
def WriteBEXTI : SchedWrite; // BEXTI

// Zbkb extension
def WriteBREV8       : SchedWrite; // brev8
def WritePACK        : SchedWrite; // pack/packh
def WritePACK32      : SchedWrite; // packw
def WriteZIP         : SchedWrite; // zip/unzip
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 45-64: TableGen record WriteXPERM / TableGen 记录 WriteXPERM
```tablegen
// Zbkx extension
def WriteXPERM       : SchedWrite; // xperm4/xperm8

/// Define scheduler resources associated with use operands.

// Zba extension
def ReadSHXADD      : SchedRead; // sh1add/sh2add/sh3add
def ReadSHXADD32    : SchedRead; // sh1add.uw/sh2add.uw/sh3add.uw

// Zbb extension
def ReadRotateImm   : SchedRead;
def ReadRotateImm32 : SchedRead;
def ReadRotateReg   : SchedRead;
def ReadRotateReg32 : SchedRead;
def ReadCLZ         : SchedRead;
def ReadCLZ32       : SchedRead;
def ReadCTZ         : SchedRead;
def ReadCTZ32       : SchedRead;
def ReadCPOP        : SchedRead;
def ReadCPOP32      : SchedRead;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 65-75: TableGen record ReadREV8 / TableGen 记录 ReadREV8
```tablegen
def ReadREV8        : SchedRead;
def ReadORCB        : SchedRead;
def ReadIMinMax     : SchedRead;

// Zbc extension
def ReadCLMUL       : SchedRead; // CLMUL/CLMULR/CLMULH

// Zbs extension
def ReadSingleBit   : SchedRead; // BCLR/BSET/BINV/BEXT
def ReadSingleBitImm: SchedRead; // BCLRI/BSETI/BINVI/BEXTI
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 76-86: TableGen record ReadBREV8 / TableGen 记录 ReadBREV8
```tablegen
// Zbkb extension
def ReadBREV8       : SchedRead; // brev8
def ReadPACK        : SchedRead; // pack/packh
def ReadPACK32      : SchedRead; // packw
def ReadZIP         : SchedRead; // zip/unzip

// Zbkx extension
def ReadXPERM       : SchedRead; // xperm4/xperm8

/// Define default scheduler resources for B.
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 87-96: Reusable TableGen multiclass UnsupportedSchedZba / 可复用的 TableGen 多类 UnsupportedSchedZba
```tablegen
multiclass UnsupportedSchedZba {
let Unsupported = true in {
def : WriteRes<WriteSHXADD, []>;
def : WriteRes<WriteSHXADD32, []>;

def : ReadAdvance<ReadSHXADD, 0>;
def : ReadAdvance<ReadSHXADD32, 0>;
}
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 97-112: Reusable TableGen multiclass UnsupportedSchedZbb / 可复用的 TableGen 多类 UnsupportedSchedZbb
```tablegen
multiclass UnsupportedSchedZbb {
let Unsupported = true in {
def : WriteRes<WriteRotateImm, []>;
def : WriteRes<WriteRotateImm32, []>;
def : WriteRes<WriteRotateReg, []>;
def : WriteRes<WriteRotateReg32, []>;
def : WriteRes<WriteCLZ, []>;
def : WriteRes<WriteCLZ32, []>;
def : WriteRes<WriteCTZ, []>;
def : WriteRes<WriteCTZ32, []>;
def : WriteRes<WriteCPOP, []>;
def : WriteRes<WriteCPOP32, []>;
def : WriteRes<WriteREV8, []>;
def : WriteRes<WriteORCB, []>;
def : WriteRes<WriteIMinMax, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 113-126: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadRotateImm, 0>;
def : ReadAdvance<ReadRotateImm32, 0>;
def : ReadAdvance<ReadRotateReg, 0>;
def : ReadAdvance<ReadRotateReg32, 0>;
def : ReadAdvance<ReadCLZ, 0>;
def : ReadAdvance<ReadCLZ32, 0>;
def : ReadAdvance<ReadCTZ, 0>;
def : ReadAdvance<ReadCTZ32, 0>;
def : ReadAdvance<ReadCPOP, 0>;
def : ReadAdvance<ReadCPOP32, 0>;
def : ReadAdvance<ReadREV8, 0>;
def : ReadAdvance<ReadORCB, 0>;
def : ReadAdvance<ReadIMinMax, 0>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 127-136: Reusable TableGen multiclass UnsupportedSchedZbc / 可复用的 TableGen 多类 UnsupportedSchedZbc
```tablegen
}

multiclass UnsupportedSchedZbc {
let Unsupported = true in {
def : WriteRes<WriteCLMUL, []>;

def : ReadAdvance<ReadCLMUL, 0>;
}
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 137-146: Reusable TableGen multiclass UnsupportedSchedZbs / 可复用的 TableGen 多类 UnsupportedSchedZbs
```tablegen
multiclass UnsupportedSchedZbs {
let Unsupported = true in {
def : WriteRes<WriteSingleBit, []>;
def : WriteRes<WriteSingleBitImm, []>;
def : WriteRes<WriteBEXT, []>;
def : WriteRes<WriteBEXTI, []>;

def : ReadAdvance<ReadSingleBit, 0>;
def : ReadAdvance<ReadSingleBitImm, 0>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 147-160: Reusable TableGen multiclass UnsupportedSchedZbkb / 可复用的 TableGen 多类 UnsupportedSchedZbkb
```tablegen
}

multiclass UnsupportedSchedZbkb {
let Unsupported = true in {
def : WriteRes<WriteBREV8, []>;
def : WriteRes<WritePACK, []>;
def : WriteRes<WritePACK32, []>;
def : WriteRes<WriteZIP, []>;

def : ReadAdvance<ReadBREV8, 0>;
def : ReadAdvance<ReadPACK, 0>;
def : ReadAdvance<ReadPACK32, 0>;
def : ReadAdvance<ReadZIP, 0>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 161-169: Reusable TableGen multiclass UnsupportedSchedZbkx / 可复用的 TableGen 多类 UnsupportedSchedZbkx
```tablegen
}

multiclass UnsupportedSchedZbkx {
let Unsupported = true in {
def : WriteRes<WriteXPERM, []>;

def : ReadAdvance<ReadXPERM, 0>;
}
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
