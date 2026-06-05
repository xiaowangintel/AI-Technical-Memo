# RISCVSchedule.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSchedule.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for processor scheduling resources, latencies, and pipeline behavior for RISC-V cores. / 使用 TableGen 定义RISC-V 处理器内核的调度资源、时延与流水线行为。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVSchedule.td - RISC-V Scheduling Definitions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Define TII for use in SchedPredicates that target MachineInstr.
def : PredicateProlog<[{
  [[maybe_unused]] const RISCVInstrInfo *TII =
    static_cast<const RISCVInstrInfo*>(SchedModel->getInstrInfo());
}]>;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-42: TableGen record operands / TableGen 记录 operands
```tablegen
/// Define scheduler resources associated with def operands.
def WriteIALU       : SchedWrite;    // 32 or 64-bit integer ALU operations
def WriteIALU32     : SchedWrite;    // 32-bit integer ALU operations on RV64I
def WriteShiftImm   : SchedWrite;    // 32 or 64-bit shift by immediate operations
def WriteShiftImm32 : SchedWrite;    // 32-bit shift by immediate operations on RV64Ix
def WriteShiftReg   : SchedWrite;    // 32 or 64-bit shift by immediate operations
def WriteShiftReg32 : SchedWrite;    // 32-bit shift by immediate operations on RV64Ix
def WriteIDiv       : SchedWrite;    // 32-bit or 64-bit divide
def WriteIDiv32     : SchedWrite;    // 32-bit divide on RV64I
def WriteIRem       : SchedWrite;    // 32-bit or 64-bit remainder
def WriteIRem32     : SchedWrite;    // 32-bit remainder on RV64I
def WriteIMul       : SchedWrite;    // 32-bit or 64-bit multiply
def WriteIMul32     : SchedWrite;    // 32-bit multiply on RV64I
def WriteJmp        : SchedWrite;    // Jump
def WriteJal        : SchedWrite;    // Jump and link
def WriteJalr       : SchedWrite;    // Jump and link register
def WriteNop        : SchedWrite;
def WriteLDB        : SchedWrite;    // Load byte
def WriteLDH        : SchedWrite;    // Load half-word
def WriteLDW        : SchedWrite;    // Load word
def WriteLDD        : SchedWrite;    // Load double-word
def WriteCSR        : SchedWrite;    // CSR instructions
def WriteSTB        : SchedWrite;    // Store byte
def WriteSTH        : SchedWrite;    // Store half-word
def WriteSTW        : SchedWrite;    // Store word
def WriteSTD        : SchedWrite;    // Store double-word
def WriteAtomicB    : SchedWrite;    //Atomic memory operation byte size
def WriteAtomicH    : SchedWrite;    //Atomic memory operation halfword size
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 43-69: TableGen record WriteAtomicW / TableGen 记录 WriteAtomicW
```tablegen
def WriteAtomicW    : SchedWrite;    //Atomic memory operation word size
def WriteAtomicD    : SchedWrite;    //Atomic memory operation double word size
def WriteAtomicLDW  : SchedWrite;    // Atomic load word
def WriteAtomicLDD  : SchedWrite;    // Atomic load double word
def WriteAtomicSTW  : SchedWrite;    // Atomic store word
def WriteAtomicSTD  : SchedWrite;    // Atomic store double word
def WriteFAdd16     : SchedWrite;    // 16-bit floating point addition/subtraction
def WriteFAdd32     : SchedWrite;    // 32-bit floating point addition/subtraction
def WriteFAdd64     : SchedWrite;    // 64-bit floating point addition/subtraction
def WriteFAdd128    : SchedWrite;    // 128-bit floating point addition/subtraction
def WriteFMul16     : SchedWrite;    // 16-bit floating point multiply
def WriteFMul32     : SchedWrite;    // 32-bit floating point multiply
def WriteFMul64     : SchedWrite;    // 64-bit floating point multiply
def WriteFMul128    : SchedWrite;    // 128-bit floating point multiply
def WriteFMA16      : SchedWrite;    // 16-bit floating point fused multiply-add
def WriteFMA32      : SchedWrite;    // 32-bit floating point fused multiply-add
def WriteFMA64      : SchedWrite;    // 64-bit floating point fused multiply-add
def WriteFMA128     : SchedWrite;    // 128-bit floating point fused multiply-add
def WriteFDiv16     : SchedWrite;    // 16-bit floating point divide
def WriteFDiv32     : SchedWrite;    // 32-bit floating point divide
def WriteFDiv64     : SchedWrite;    // 64-bit floating point divide
def WriteFDiv128    : SchedWrite;    // 128-bit floating point divide
def WriteFSqrt16    : SchedWrite;    // 16-bit floating point sqrt
def WriteFSqrt32    : SchedWrite;    // 32-bit floating point sqrt
def WriteFSqrt64    : SchedWrite;    // 64-bit floating point sqrt
def WriteFSqrt128   : SchedWrite;    // 128-bit floating point sqrt
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 70-89: TableGen record WriteFCvtI32ToF16 / TableGen 记录 WriteFCvtI32ToF16
```tablegen
// Integer to float conversions
def WriteFCvtI32ToF16  : SchedWrite;
def WriteFCvtI32ToF32  : SchedWrite;
def WriteFCvtI32ToF64  : SchedWrite;
def WriteFCvtI32ToF128 : SchedWrite;
def WriteFCvtI64ToF16  : SchedWrite;    // RV64I only
def WriteFCvtI64ToF32  : SchedWrite;    // RV64I only
def WriteFCvtI64ToF64  : SchedWrite;    // RV64I only
def WriteFCvtI64ToF128 : SchedWrite;    // RV64I only

// Float to integer conversions
def WriteFCvtF16ToI32  : SchedWrite;
def WriteFCvtF16ToI64  : SchedWrite;    // RV64I only
def WriteFCvtF32ToI32  : SchedWrite;
def WriteFCvtF32ToI64  : SchedWrite;    // RV64I only
def WriteFCvtF64ToI32  : SchedWrite;
def WriteFCvtF64ToI64  : SchedWrite;    // RV64I only
def WriteFCvtF128ToI32 : SchedWrite;
def WriteFCvtF128ToI64 : SchedWrite;    // RV64I only
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 90-107: TableGen record WriteFCvtF32ToF64 / TableGen 记录 WriteFCvtF32ToF64
```tablegen
// Float to float conversions
def WriteFCvtF32ToF64  : SchedWrite;
def WriteFCvtF64ToF32  : SchedWrite;
def WriteFCvtF16ToF32  : SchedWrite;
def WriteFCvtF32ToF16  : SchedWrite;
def WriteFCvtF16ToF64  : SchedWrite;
def WriteFCvtF64ToF16  : SchedWrite;
def WriteFCvtF128ToF32 : SchedWrite;
def WriteFCvtF128ToF64 : SchedWrite;
def WriteFCvtF32ToF128 : SchedWrite;
def WriteFCvtF64ToF128 : SchedWrite;

// Zfa fround instructions.
def WriteFRoundF32     : SchedWrite;
def WriteFRoundF64     : SchedWrite;
def WriteFRoundF16     : SchedWrite;
def WriteFRoundF128    : SchedWrite;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 108-124: TableGen record WriteFClass16 / TableGen 记录 WriteFClass16
```tablegen
def WriteFClass16   : SchedWrite;    // 16-bit floating point classify
def WriteFClass32   : SchedWrite;    // 32-bit floating point classify
def WriteFClass64   : SchedWrite;    // 64-bit floating point classify
def WriteFClass128  : SchedWrite;    // 128-bit floating point classify
def WriteFCmp16     : SchedWrite;    // 16-bit floating point compare
def WriteFCmp32     : SchedWrite;    // 32-bit floating point compare
def WriteFCmp64     : SchedWrite;    // 64-bit floating point compare
def WriteFCmp128    : SchedWrite;    // 128-bit floating point compare
def WriteFSGNJ16    : SchedWrite;    // 16-bit floating point sign-injection
def WriteFSGNJ32    : SchedWrite;    // 32-bit floating point sign-injection
def WriteFSGNJ64    : SchedWrite;    // 64-bit floating point sign-injection
def WriteFSGNJ128   : SchedWrite;    // 128-bit floating point sign-injection
def WriteFMinMax16  : SchedWrite;    // 16-bit floating point min or max
def WriteFMinMax32  : SchedWrite;    // 32-bit floating point min or max
def WriteFMinMax64  : SchedWrite;    // 64-bit floating point min or max
def WriteFMinMax128 : SchedWrite;    // 128-bit floating point min or max
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 125-145: TableGen record WriteFMovF16ToI16 / TableGen 记录 WriteFMovF16ToI16
```tablegen
def WriteFMovF16ToI16     : SchedWrite;
def WriteFMovI16ToF16     : SchedWrite;
def WriteFMovF32ToI32     : SchedWrite;
def WriteFMovI32ToF32     : SchedWrite;
def WriteFMovF64ToI64     : SchedWrite;    // RV64I only
def WriteFMovI64ToF64     : SchedWrite;    // RV64I only

def WriteFLI16        : SchedWrite;    // Floating point constant load
def WriteFLI32        : SchedWrite;    // Floating point constant load
def WriteFLI64        : SchedWrite;    // Floating point constant load
def WriteFLI128       : SchedWrite;    // Floating point constant load

def WriteFLD16        : SchedWrite;    // Floating point sp load
def WriteFLD32        : SchedWrite;    // Floating point sp load
def WriteFLD64        : SchedWrite;    // Floating point dp load
def WriteFLD128       : SchedWrite;    // Floating point qp load
def WriteFST16        : SchedWrite;    // Floating point sp store
def WriteFST32        : SchedWrite;    // Floating point sp store
def WriteFST64        : SchedWrite;    // Floating point dp store
def WriteFST128       : SchedWrite;    // Floating point qp store
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 146-173: TableGen record WriteSFB / TableGen 记录 WriteSFB
```tablegen
// short forward branch for Bullet
def WriteSFB        : SchedWrite;
def ReadSFBJmp      : SchedRead;
def ReadSFBALU      : SchedRead;

/// Define scheduler resources associated with use operands.
def ReadJmp         : SchedRead;
def ReadJalr        : SchedRead;
def ReadCSR         : SchedRead;
def ReadMemBase     : SchedRead;
def ReadFMemBase    : SchedRead;
def ReadStoreData   : SchedRead;
def ReadFStoreData  : SchedRead;
def ReadIALU        : SchedRead;
def ReadIALU32      : SchedRead;    // 32-bit integer ALU operations on RV64I
def ReadShiftImm    : SchedRead;
def ReadShiftImm32  : SchedRead;    // 32-bit shift by immediate operations on RV64Ix
def ReadShiftReg    : SchedRead;
def ReadShiftReg32  : SchedRead;    // 32-bit shift by register operations on RV64Ix
def ReadIDiv        : SchedRead;
def ReadIDiv32      : SchedRead;
def ReadIRem        : SchedRead;
def ReadIRem32      : SchedRead;
def ReadIMul        : SchedRead;
def ReadIMul32      : SchedRead;
def ReadAtomicBA    : SchedRead;
def ReadAtomicBD    : SchedRead;
def ReadAtomicHA    : SchedRead;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 174-201: TableGen record ReadAtomicHD / TableGen 记录 ReadAtomicHD
```tablegen
def ReadAtomicHD    : SchedRead;
def ReadAtomicWA    : SchedRead;
def ReadAtomicWD    : SchedRead;
def ReadAtomicDA    : SchedRead;
def ReadAtomicDD    : SchedRead;
def ReadAtomicLDW   : SchedRead;    // Atomic load word
def ReadAtomicLDD   : SchedRead;    // Atomic load double word
def ReadAtomicSTW   : SchedRead;    // Atomic store word
def ReadAtomicSTD   : SchedRead;    // Atomic store double word
def ReadFAdd16      : SchedRead;    // 16-bit floating point addition/subtraction
def ReadFAdd32      : SchedRead;    // 32-bit floating point addition/subtraction
def ReadFAdd64      : SchedRead;    // 64-bit floating point addition/subtraction
def ReadFAdd128     : SchedRead;    // 128-bit floating point addition/subtraction
def ReadFMul16      : SchedRead;    // 16-bit floating point multiply
def ReadFMul32      : SchedRead;    // 32-bit floating point multiply
def ReadFMul64      : SchedRead;    // 64-bit floating point multiply
def ReadFMul128     : SchedRead;    // 128-bit floating point multiply
def ReadFMA16       : SchedRead;    // 16-bit floating point fused multiply-add
def ReadFMA16Addend : SchedRead;    // 16-bit floating point fused multiply-add (addend)
def ReadFMA32       : SchedRead;    // 32-bit floating point fused multiply-add
def ReadFMA32Addend : SchedRead;    // 32-bit floating point fused multiply-add (addend)
def ReadFMA64       : SchedRead;    // 64-bit floating point fused multiply-add
def ReadFMA64Addend : SchedRead;    // 64-bit floating point fused multiply-add (addend)
def ReadFMA128      : SchedRead;    // 128-bit floating point fused multiply-add
def ReadFMA128Addend: SchedRead;    // 128-bit floating point fused multiply-add (addend)
def ReadFDiv16      : SchedRead;    // 16-bit floating point divide
def ReadFDiv32      : SchedRead;    // 32-bit floating point divide
def ReadFDiv64      : SchedRead;    // 64-bit floating point divide
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 202-229: TableGen record ReadFDiv128 / TableGen 记录 ReadFDiv128
```tablegen
def ReadFDiv128     : SchedRead;    // 128-bit floating point divide
def ReadFSqrt16     : SchedRead;    // 16-bit floating point sqrt
def ReadFSqrt32     : SchedRead;    // 32-bit floating point sqrt
def ReadFSqrt64     : SchedRead;    // 64-bit floating point sqrt
def ReadFSqrt128    : SchedRead;    // 128-bit floating point sqrt
def ReadFCmp16      : SchedRead;
def ReadFCmp32      : SchedRead;
def ReadFCmp64      : SchedRead;
def ReadFCmp128     : SchedRead;
def ReadFSGNJ16     : SchedRead;
def ReadFSGNJ32     : SchedRead;
def ReadFSGNJ64     : SchedRead;
def ReadFSGNJ128    : SchedRead;
def ReadFMinMax16   : SchedRead;
def ReadFMinMax32   : SchedRead;
def ReadFMinMax64   : SchedRead;
def ReadFMinMax128  : SchedRead;
def ReadFCvtF16ToI32     : SchedRead;
def ReadFCvtF16ToI64     : SchedRead;
def ReadFCvtF32ToI32     : SchedRead;
def ReadFCvtF32ToI64     : SchedRead;
def ReadFCvtF64ToI32     : SchedRead;
def ReadFCvtF64ToI64     : SchedRead;
def ReadFCvtF128ToI32    : SchedRead;
def ReadFCvtF128ToI64    : SchedRead;
def ReadFCvtI32ToF16     : SchedRead;
def ReadFCvtI32ToF32     : SchedRead;
def ReadFCvtI32ToF64     : SchedRead;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 230-251: TableGen record ReadFCvtI32ToF128 / TableGen 记录 ReadFCvtI32ToF128
```tablegen
def ReadFCvtI32ToF128    : SchedRead;
def ReadFCvtI64ToF16     : SchedRead;
def ReadFCvtI64ToF32     : SchedRead;
def ReadFCvtI64ToF64     : SchedRead;
def ReadFCvtI64ToF128    : SchedRead;
def ReadFMovF16ToI16     : SchedRead;
def ReadFMovI16ToF16     : SchedRead;
def ReadFMovF32ToI32     : SchedRead;
def ReadFMovI32ToF32     : SchedRead;
def ReadFMovF64ToI64     : SchedRead;
def ReadFMovI64ToF64     : SchedRead;
def ReadFCvtF32ToF64     : SchedRead;
def ReadFCvtF64ToF32     : SchedRead;
def ReadFCvtF16ToF32     : SchedRead;
def ReadFCvtF32ToF16     : SchedRead;
def ReadFCvtF16ToF64     : SchedRead;
def ReadFCvtF64ToF16     : SchedRead;
def ReadFCvtF128ToF32    : SchedRead;
def ReadFCvtF128ToF64    : SchedRead;
def ReadFCvtF32ToF128    : SchedRead;
def ReadFCvtF64ToF128    : SchedRead;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 252-278: Reusable TableGen multiclass ReadFRoundF16 / 可复用的 TableGen 多类 ReadFRoundF16
```tablegen
def ReadFRoundF16        : SchedRead;
def ReadFRoundF32        : SchedRead;
def ReadFRoundF64        : SchedRead;
def ReadFRoundF128       : SchedRead;

def ReadFClass16         : SchedRead;
def ReadFClass32         : SchedRead;
def ReadFClass64         : SchedRead;
def ReadFClass128        : SchedRead;

// For CPUs that support Zfhmin, but not Zfh.
multiclass UnsupportedSchedZfh {
let Unsupported = true in {
def : WriteRes<WriteFAdd16, []>;
def : WriteRes<WriteFClass16, []>;
def : WriteRes<WriteFCvtI64ToF16, []>;
def : WriteRes<WriteFCvtI32ToF16, []>;
def : WriteRes<WriteFCvtF16ToI64, []>;
def : WriteRes<WriteFCvtF16ToI32, []>;
def : WriteRes<WriteFDiv16, []>;
def : WriteRes<WriteFCmp16, []>;
def : WriteRes<WriteFMA16, []>;
def : WriteRes<WriteFMinMax16, []>;
def : WriteRes<WriteFMul16, []>;
def : WriteRes<WriteFSGNJ16, []>;
def : WriteRes<WriteFSqrt16, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 279-294: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFAdd16, 0>;
def : ReadAdvance<ReadFClass16, 0>;
def : ReadAdvance<ReadFCvtI64ToF16, 0>;
def : ReadAdvance<ReadFCvtI32ToF16, 0>;
def : ReadAdvance<ReadFCvtF16ToI64, 0>;
def : ReadAdvance<ReadFCvtF16ToI32, 0>;
def : ReadAdvance<ReadFDiv16, 0>;
def : ReadAdvance<ReadFCmp16, 0>;
def : ReadAdvance<ReadFMA16, 0>;
def : ReadAdvance<ReadFMA16Addend, 0>;
def : ReadAdvance<ReadFMinMax16, 0>;
def : ReadAdvance<ReadFMul16, 0>;
def : ReadAdvance<ReadFSGNJ16, 0>;
def : ReadAdvance<ReadFSqrt16, 0>;
} // Unsupported = true
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 295-315: Reusable TableGen multiclass UnsupportedSchedZfhmin / 可复用的 TableGen 多类 UnsupportedSchedZfhmin
```tablegen

// For CPUs that support neither Zfhmin or Zfh.
multiclass UnsupportedSchedZfhmin : UnsupportedSchedZfh {
let Unsupported = true in {
def : WriteRes<WriteFCvtF16ToF64, []>;
def : WriteRes<WriteFCvtF64ToF16, []>;
def : WriteRes<WriteFCvtF16ToF32, []>;
def : WriteRes<WriteFCvtF32ToF16, []>;
def : WriteRes<WriteFLD16, []>;
def : WriteRes<WriteFMovI16ToF16, []>;
def : WriteRes<WriteFMovF16ToI16, []>;
def : WriteRes<WriteFST16, []>;

def : ReadAdvance<ReadFCvtF16ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF16, 0>;
def : ReadAdvance<ReadFCvtF16ToF32, 0>;
def : ReadAdvance<ReadFCvtF32ToF16, 0>;
def : ReadAdvance<ReadFMovI16ToF16, 0>;
def : ReadAdvance<ReadFMovF16ToI16, 0>;
} // Unsupported = true
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 316-338: Reusable TableGen multiclass UnsupportedSchedQ / 可复用的 TableGen 多类 UnsupportedSchedQ
```tablegen

multiclass UnsupportedSchedQ {
let Unsupported = true in {
def : WriteRes<WriteFST128, []>;
def : WriteRes<WriteFLD128, []>;
def : WriteRes<WriteFAdd128, []>;
def : WriteRes<WriteFSGNJ128, []>;
def : WriteRes<WriteFMinMax128, []>;
def : WriteRes<WriteFCvtI32ToF128, []>;
def : WriteRes<WriteFCvtI64ToF128, []>;
def : WriteRes<WriteFCvtF128ToI32, []>;
def : WriteRes<WriteFCvtF128ToI64, []>;
def : WriteRes<WriteFCvtF32ToF128, []>;
def : WriteRes<WriteFCvtF128ToF32, []>;
def : WriteRes<WriteFCvtF64ToF128, []>;
def : WriteRes<WriteFCvtF128ToF64, []>;
def : WriteRes<WriteFClass128, []>;
def : WriteRes<WriteFCmp128, []>;
def : WriteRes<WriteFMul128, []>;
def : WriteRes<WriteFMA128, []>;
def : WriteRes<WriteFDiv128, []>;
def : WriteRes<WriteFSqrt128, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 339-358: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFAdd128, 0>;
def : ReadAdvance<ReadFMul128, 0>;
def : ReadAdvance<ReadFMA128, 0>;
def : ReadAdvance<ReadFMA128Addend, 0>;
def : ReadAdvance<ReadFDiv128, 0>;
def : ReadAdvance<ReadFSqrt128, 0>;
def : ReadAdvance<ReadFCmp128, 0>;
def : ReadAdvance<ReadFSGNJ128, 0>;
def : ReadAdvance<ReadFMinMax128, 0>;
def : ReadAdvance<ReadFCvtF128ToI32, 0>;
def : ReadAdvance<ReadFCvtF128ToI64, 0>;
def : ReadAdvance<ReadFCvtI32ToF128, 0>;
def : ReadAdvance<ReadFCvtI64ToF128, 0>;
def : ReadAdvance<ReadFCvtF32ToF128, 0>;
def : ReadAdvance<ReadFCvtF128ToF32, 0>;
def : ReadAdvance<ReadFCvtF64ToF128, 0>;
def : ReadAdvance<ReadFCvtF128ToF64, 0>;
def : ReadAdvance<ReadFClass128, 0>;
} // Unsupported = true
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 359-381: Reusable TableGen multiclass UnsupportedSchedD / 可复用的 TableGen 多类 UnsupportedSchedD
```tablegen

multiclass UnsupportedSchedD : UnsupportedSchedQ {
let Unsupported = true in {
def : WriteRes<WriteFST64, []>;
def : WriteRes<WriteFLD64, []>;
def : WriteRes<WriteFAdd64, []>;
def : WriteRes<WriteFSGNJ64, []>;
def : WriteRes<WriteFMinMax64, []>;
def : WriteRes<WriteFCvtI32ToF64, []>;
def : WriteRes<WriteFCvtI64ToF64, []>;
def : WriteRes<WriteFCvtF64ToI32, []>;
def : WriteRes<WriteFCvtF64ToI64, []>;
def : WriteRes<WriteFCvtF32ToF64, []>;
def : WriteRes<WriteFCvtF64ToF32, []>;
def : WriteRes<WriteFClass64, []>;
def : WriteRes<WriteFCmp64, []>;
def : WriteRes<WriteFMovF64ToI64, []>;
def : WriteRes<WriteFMovI64ToF64, []>;
def : WriteRes<WriteFMul64, []>;
def : WriteRes<WriteFMA64, []>;
def : WriteRes<WriteFDiv64, []>;
def : WriteRes<WriteFSqrt64, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 382-401: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFAdd64, 0>;
def : ReadAdvance<ReadFMul64, 0>;
def : ReadAdvance<ReadFMA64, 0>;
def : ReadAdvance<ReadFMA64Addend, 0>;
def : ReadAdvance<ReadFDiv64, 0>;
def : ReadAdvance<ReadFSqrt64, 0>;
def : ReadAdvance<ReadFCmp64, 0>;
def : ReadAdvance<ReadFSGNJ64, 0>;
def : ReadAdvance<ReadFMinMax64, 0>;
def : ReadAdvance<ReadFCvtF64ToI32, 0>;
def : ReadAdvance<ReadFCvtF64ToI64, 0>;
def : ReadAdvance<ReadFCvtI32ToF64, 0>;
def : ReadAdvance<ReadFCvtI64ToF64, 0>;
def : ReadAdvance<ReadFCvtF32ToF64, 0>;
def : ReadAdvance<ReadFCvtF64ToF32, 0>;
def : ReadAdvance<ReadFMovF64ToI64, 0>;
def : ReadAdvance<ReadFMovI64ToF64, 0>;
def : ReadAdvance<ReadFClass64, 0>;
} // Unsupported = true
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 402-423: Reusable TableGen multiclass UnsupportedSchedF / 可复用的 TableGen 多类 UnsupportedSchedF
```tablegen

// For CPUs with no floating point.
multiclass UnsupportedSchedF : UnsupportedSchedD, UnsupportedSchedZfhmin {
let Unsupported = true in {
def : WriteRes<WriteFST32, []>;
def : WriteRes<WriteFLD32, []>;
def : WriteRes<WriteFAdd32, []>;
def : WriteRes<WriteFSGNJ32, []>;
def : WriteRes<WriteFMinMax32, []>;
def : WriteRes<WriteFCvtI32ToF32, []>;
def : WriteRes<WriteFCvtI64ToF32, []>;
def : WriteRes<WriteFCvtF32ToI32, []>;
def : WriteRes<WriteFCvtF32ToI64, []>;
def : WriteRes<WriteFClass32, []>;
def : WriteRes<WriteFCmp32, []>;
def : WriteRes<WriteFMovF32ToI32, []>;
def : WriteRes<WriteFMovI32ToF32, []>;
def : WriteRes<WriteFMul32, []>;
def : WriteRes<WriteFMA32, []>;
def : WriteRes<WriteFDiv32, []>;
def : WriteRes<WriteFSqrt32, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 424-443: TableGen record definition / TableGen 记录定义
```tablegen
def : ReadAdvance<ReadFAdd32, 0>;
def : ReadAdvance<ReadFMul32, 0>;
def : ReadAdvance<ReadFMA32, 0>;
def : ReadAdvance<ReadFMA32Addend, 0>;
def : ReadAdvance<ReadFDiv32, 0>;
def : ReadAdvance<ReadFSqrt32, 0>;
def : ReadAdvance<ReadFCmp32, 0>;
def : ReadAdvance<ReadFSGNJ32, 0>;
def : ReadAdvance<ReadFMinMax32, 0>;
def : ReadAdvance<ReadFCvtF32ToI32, 0>;
def : ReadAdvance<ReadFCvtF32ToI64, 0>;
def : ReadAdvance<ReadFCvtI32ToF32, 0>;
def : ReadAdvance<ReadFCvtI64ToF32, 0>;
def : ReadAdvance<ReadFMovF32ToI32, 0>;
def : ReadAdvance<ReadFMovI32ToF32, 0>;
def : ReadAdvance<ReadFClass32, 0>;
def : ReadAdvance<ReadFStoreData, 0>;
def : ReadAdvance<ReadFMemBase, 0>;
} // Unsupported = true
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 444-458: Reusable TableGen multiclass UnsupportedSchedSFB / 可复用的 TableGen 多类 UnsupportedSchedSFB
```tablegen

multiclass UnsupportedSchedSFB {
let Unsupported = true in {
def : WriteRes<WriteSFB, []>;

def : ReadAdvance<ReadSFBJmp, 0>;
def : ReadAdvance<ReadSFBALU, 0>;
} // Unsupported = true
}

multiclass UnsupportedSchedZfaWithQ {
let Unsupported = true in {
def : WriteRes<WriteFRoundF128, []>;
def : WriteRes<WriteFLI128, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 459-476: Reusable TableGen multiclass UnsupportedSchedZfaWithD / 可复用的 TableGen 多类 UnsupportedSchedZfaWithD
```tablegen
def : ReadAdvance<ReadFRoundF128, 0>;
}
}

multiclass UnsupportedSchedZfaWithD : UnsupportedSchedZfaWithQ {
let Unsupported = true in {
def : WriteRes<WriteFRoundF64, []>;
def : WriteRes<WriteFLI64, []>;

def : ReadAdvance<ReadFRoundF64, 0>;
}
}

multiclass UnsupportedSchedZfaWithZfh {
let Unsupported = true in {
def : WriteRes<WriteFRoundF16, []>;
def : WriteRes<WriteFLI16, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 477-490: Reusable TableGen multiclass UnsupportedSchedZfa / 可复用的 TableGen 多类 UnsupportedSchedZfa
```tablegen
def : ReadAdvance<ReadFRoundF16, 0>;
}
}

multiclass UnsupportedSchedZfa : UnsupportedSchedZfaWithD,
                                 UnsupportedSchedZfaWithZfh {
let Unsupported = true in {
def : WriteRes<WriteFRoundF32, []>;
def : WriteRes<WriteFLI32, []>;

def : ReadAdvance<ReadFRoundF32, 0>;
} // Unsupported = true
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 491-511: Reusable TableGen multiclass UnsupportedSchedZabha / 可复用的 TableGen 多类 UnsupportedSchedZabha
```tablegen
multiclass UnsupportedSchedZabha {
let Unsupported = true in {
def : WriteRes<WriteAtomicB, []>;
def : WriteRes<WriteAtomicH, []>;

def : ReadAdvance<ReadAtomicBA, 0>;
def : ReadAdvance<ReadAtomicBD, 0>;
def : ReadAdvance<ReadAtomicHA, 0>;
def : ReadAdvance<ReadAtomicHD, 0>;
} // Unsupported = true
}

multiclass UnsupportedSchedA {
let Unsupported = true in {
def : WriteRes<WriteAtomicW, []>;
def : WriteRes<WriteAtomicD, []>;
def : WriteRes<WriteAtomicLDW, []>;
def : WriteRes<WriteAtomicLDD, []>;
def : WriteRes<WriteAtomicSTW, []>;
def : WriteRes<WriteAtomicSTD, []>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 512-528: Included TableGen fragments / 引入的 TableGen 片段
```tablegen
def : ReadAdvance<ReadAtomicWA, 0>;
def : ReadAdvance<ReadAtomicWD, 0>;
def : ReadAdvance<ReadAtomicDA, 0>;
def : ReadAdvance<ReadAtomicDD, 0>;
def : ReadAdvance<ReadAtomicLDW, 0>;
def : ReadAdvance<ReadAtomicLDD, 0>;
def : ReadAdvance<ReadAtomicSTW, 0>;
def : ReadAdvance<ReadAtomicSTD, 0>;
} // Unsupported = true
}

// Include the scheduler resources for other instruction extensions.
include "RISCVScheduleZb.td"
include "RISCVScheduleV.td"
include "RISCVScheduleXSf.td"
include "RISCVScheduleZvk.td"
```
**EN:** This TableGen block composes the file from reusable fragments so generated backend records can share common definitions.

**CN:** 该 TableGen 区段通过复用片段来组合文件，使生成的后端记录能够共享公共定义。

### Lines 529-538: Reusable TableGen multiclass UnsupportedSchedXsf / 可复用的 TableGen 多类 UnsupportedSchedXsf
```tablegen
// Vendor Extensions
multiclass UnsupportedSchedXsf {
  defm : UnsupportedSchedXsfvcp;
  defm : UnsupportedSchedXSfvfexp;
  defm : UnsupportedSchedXSfvfexpa;
  defm : UnsupportedSchedXSfvfnrclipxfqf;
  defm : UnsupportedSchedXSfvfwmaccqqq;
  defm : UnsupportedSchedXSfvqmaccdod;
  defm : UnsupportedSchedXSfvqmaccqoq;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

## Key Concepts / 关键概念
- **Scheduling models** / **调度模型**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- `RISCVScheduleZb.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVScheduleV.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVScheduleXSf.td` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVScheduleZvk.td` — Directly referenced by this file. / 该文件直接引用的依赖。
