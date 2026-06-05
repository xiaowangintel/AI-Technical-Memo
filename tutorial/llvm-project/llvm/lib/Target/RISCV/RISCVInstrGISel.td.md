# RISCVInstrGISel.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrGISel.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrGISel.td - RISC-V GISel target pseudos ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
// RISC-V GlobalISel target pseudo instruction definitions. This is kept
// separately from the other tablegen files for organizational purposes, but
// share the same infrastructure.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-25: TableGen class RISCVGenericInstruction / TableGen 类 RISCVGenericInstruction
```tablegen

class RISCVGenericInstruction : GenericInstruction {
  let Namespace = "RISCV";
}

// Pseudo equivalent to a RISCVISD::SRAW.
def G_SRAW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 26-35: TableGen record G_SRLW / TableGen 记录 G_SRLW
```tablegen
def : GINodeEquiv<G_SRAW, riscv_sraw>;

// Pseudo equivalent to a RISCVISD::SRLW.
def G_SRLW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_SRLW, riscv_srlw>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 36-49: TableGen record G_SLLW / TableGen 记录 G_SLLW
```tablegen
// Pseudo equivalent to a RISCVISD::SLLW.
def G_SLLW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_SLLW, riscv_sllw>;

// Pseudo equivalent to a RISCVISD::DIVW.
def G_DIVW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 50-59: TableGen record G_DIVUW / TableGen 记录 G_DIVUW
```tablegen
def : GINodeEquiv<G_DIVW, riscv_divw>;

// Pseudo equivalent to a RISCVISD::DIVUW.
def G_DIVUW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_DIVUW, riscv_divuw>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 60-73: TableGen record G_REMUW / TableGen 记录 G_REMUW
```tablegen
// Pseudo equivalent to a RISCVISD::REMUW.
def G_REMUW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_REMUW, riscv_remuw>;

// Pseudo equivalent to a RISCVISD::RORW.
def G_RORW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 74-83: TableGen record G_ROLW / TableGen 记录 G_ROLW
```tablegen
def : GINodeEquiv<G_RORW, riscv_rorw>;

// Pseudo equivalent to a RISCVISD::ROLW.
def G_ROLW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src1, type0:$src2);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_ROLW, riscv_rolw>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 84-97: TableGen record G_CLZW / TableGen 记录 G_CLZW
```tablegen
// Pseudo equivalent to a RISCVISD::CLZW.
def G_CLZW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_CLZW, riscv_clzw>;

// Pseudo equivalent to a RISCVISD::CTZW.
def G_CTZW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src);
  let hasSideEffects = false;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 98-107: TableGen record G_FCVT_W_RV64 / TableGen 记录 G_FCVT_W_RV64
```tablegen
def : GINodeEquiv<G_CTZW, riscv_ctzw>;

// Pseudo equivalent to a RISCVISD::FCVT_W_RV64.
def G_FCVT_W_RV64 : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$src, untyped_imm_0:$frm);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_FCVT_W_RV64, riscv_fcvt_w_rv64>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 108-121: TableGen record G_FCVT_WU_RV64 / TableGen 记录 G_FCVT_WU_RV64
```tablegen
// Pseudo equivalent to a RISCVISD::FCVT_WU_RV64.
def G_FCVT_WU_RV64 : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$src, untyped_imm_0:$frm);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_FCVT_WU_RV64, riscv_fcvt_wu_rv64>;

// Pseudo equivalent to a RISCVISD::FCLASS.
def G_FCLASS : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$src);
  let hasSideEffects = false;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 122-131: TableGen record G_READ_VLENB / TableGen 记录 G_READ_VLENB
```tablegen
def : GINodeEquiv<G_FCLASS, riscv_fclass>;

// Pseudo equivalent to a RISCVISD::READ_VLENB.
def G_READ_VLENB : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_READ_VLENB, riscv_read_vlenb>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 132-145: TableGen record G_VMCLR_VL / TableGen 记录 G_VMCLR_VL
```tablegen
// Pseudo equivalent to a RISCVISD::VMCLR_VL
def G_VMCLR_VL : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$vl);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_VMCLR_VL, riscv_vmclr_vl>;

// Pseudo equivalent to a RISCVISD::VMSET_VL
def G_VMSET_VL : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type1:$vl);
  let hasSideEffects = false;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 146-161: TableGen record G_SPLAT_VECTOR_SPLIT_I64_VL / TableGen 记录 G_SPLAT_VECTOR_SPLIT_I64_VL
```tablegen
def : GINodeEquiv<G_VMSET_VL, riscv_vmset_vl>;

def G_SPLAT_VECTOR_SPLIT_I64_VL : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$passthru, type1:$hi, type1:$lo, type2:$vl);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_SPLAT_VECTOR_SPLIT_I64_VL, riscv_splat_vector_split_i64_vl>;

// Pseudo equivalent to a RISCVISD::VSLIDEDOWN_VL
def G_VSLIDEDOWN_VL : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$merge, type0:$vec, type1:$idx, type2:$mask,
                       type1:$vl, type1:$policy);
  let hasSideEffects = false;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 162-171: TableGen record G_VMV_V_V_VL / TableGen 记录 G_VMV_V_V_VL
```tablegen
def : GINodeEquiv<G_VSLIDEDOWN_VL, riscv_slidedown_vl>;

// Pseudo equivalent to a RISCVISD::VMV_V_V_VL
def G_VMV_V_V_VL : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$passthru, type0:$vec, type1:$vl);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_VMV_V_V_VL, riscv_vmv_v_v_vl>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 172-186: TableGen record G_VSLIDEUP_VL / TableGen 记录 G_VSLIDEUP_VL
```tablegen
// Pseudo equivalent to a RISCVISD::VSLIDEUP_VL
def G_VSLIDEUP_VL : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$merge, type0:$vec, type1:$idx, type2:$mask,
                       type3:$vl, type4:$policy);
  let hasSideEffects = false;
}
def : GINodeEquiv<G_VSLIDEUP_VL, riscv_slideup_vl>;

// Pseudo equivalent to RISCVISD::CLSW
def G_CLSW : RISCVGenericInstruction {
  let OutOperandList = (outs type0:$dst);
  let InOperandList = (ins type0:$src);
  let hasSideEffects = false;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 187-187: TableGen record definition / TableGen 记录定义
```tablegen
def : GINodeEquiv<G_CLSW , riscv_clsw>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
