# RISCVInstrInfoZfbfmin.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZfbfmin.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZfbfmin.td - 'Zfbfmin' instructions --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Zfbfmin'
// extension, providing scalar conversion instructions for BFloat16.
// This version is still experimental as the 'Zfbfmin' extension hasn't been
// ratified yet.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-26: TableGen record FCVT_BF16_S / TableGen 记录 FCVT_BF16_S
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZfbfmin] in {
def FCVT_BF16_S : FPUnaryOp_r_frm<0b0100010, 0b01000, FPR16, FPR32, "fcvt.bf16.s">,
                  Sched<[WriteFCvtF32ToF16, ReadFCvtF32ToF16]>;
def FCVT_S_BF16 : FPUnaryOp_r_frmlegacy<0b0100000, 0b00110, FPR32, FPR16, "fcvt.s.bf16">,
                  Sched<[WriteFCvtF16ToF32, ReadFCvtF16ToF32]>;
} // Predicates = [HasStdExtZfbfmin]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 27-36: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Pseudo-instructions and codegen patterns
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZfbfmin] in {
def : Pat<(riscv_selectcc (XLenVT GPR:$lhs), GPR:$rhs, cond:$cc,
                          (bf16 FPR16:$truev), FPR16:$falsev),
          (Select_FPR16_Using_CC_GPR GPR:$lhs, GPR:$rhs,
           (CCtoRISCVCC $cc), FPR16:$truev, FPR16:$falsev)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 37-46: TableGen record definition / TableGen 记录定义
```tablegen
// Explicitly select 0 in the condition to X0. The register coalescer doesn't
// always do it.
def : Pat<(riscv_selectcc (XLenVT GPR:$lhs), 0, cond:$cc,
                          (bf16 FPR16:$truev), FPR16:$falsev),
          (Select_FPR16_Using_CC_GPR GPR:$lhs, (XLenVT X0),
           (CCtoRISCVCC $cc), FPR16:$truev, FPR16:$falsev)>;

/// Loads
def : LdPat<load, FLH, bf16>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 47-56: TableGen record definition / TableGen 记录定义
```tablegen
/// Stores
def : StPat<store, FSH, FPR16, bf16>;

/// Float conversion operations
// f32 -> bf16, bf16 -> f32
def : Pat<(bf16 (fpround FPR32:$rs1)),
          (FCVT_BF16_S FPR32:$rs1, FRM_DYN)>;
def : Pat<(fpextend (bf16 FPR16:$rs1)),
          (FCVT_S_BF16 FPR16:$rs1, FRM_RNE)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 57-71: TableGen record definition / TableGen 记录定义
```tablegen
// Moves (no conversion)
def : Pat<(bf16 (riscv_fmv_h_x GPR:$src)), (FMV_H_X GPR:$src)>;
def : Pat<(riscv_fmv_x_anyexth (bf16 FPR16:$src)), (FMV_X_H FPR16:$src)>;
def : Pat<(riscv_fmv_x_signexth (bf16 FPR16:$src)), (FMV_X_H FPR16:$src)>;
} // Predicates = [HasStdExtZfbfmin]

let Predicates = [HasStdExtZfbfmin, HasStdExtD] in {
// f64 -> bf16
// FIXME: This pattern double rounds.
def : Pat<(bf16 (fpround FPR64:$rs1)),
          (FCVT_BF16_S (FCVT_S_D FPR64:$rs1, FRM_DYN), FRM_DYN)>;
// bf16 -> f64
def : Pat<(fpextend (bf16 FPR16:$rs1)),
          (FCVT_D_S (FCVT_S_BF16 FPR16:$rs1, FRM_DYN), FRM_RNE)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 72-78: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasStdExtZfhmin, HasStdExtZfbfmin] in {
// bf16 -> f16
def : Pat<(f16 (bitconvert (bf16 FPR16:$src))), (f16 FPR16:$src)>;
// f16 -> bf16
def : Pat<(bf16 (bitconvert (f16 FPR16:$src))), (bf16 FPR16:$src)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
