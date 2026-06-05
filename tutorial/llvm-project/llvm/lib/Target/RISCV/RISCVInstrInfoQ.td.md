# RISCVInstrInfoQ.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoQ.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoQ.td - RISC-V 'Q' instructions -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Q',
// Quad-Precision Floating-Point instruction set extension.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-22: TableGen record QExt / TableGen 记录 QExt
```tablegen

//===----------------------------------------------------------------------===//
// Operand and SDNode transformation definitions.
//===----------------------------------------------------------------------===//

def QExt : ExtInfo<"", "", [HasStdExtQ], f128, FPR128, FPR32, FPR64, ?>;

defvar QExts = [QExt];
defvar QExtsRV64 = [QExt];
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 23-35: TableGen record FLQ / TableGen 记录 FLQ
```tablegen
//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtQ] in {
  def FLQ : FPLoad_r<0b100, "flq", FPR128, WriteFLD128>;

  // Operands for stores are in the order srcreg, base, offset rather than
  // reflecting the order these fields are specified in the instruction
  // encoding.
  def FSQ : FPStore_r<0b100, "fsq", FPR128, WriteFST128>;
} // Predicates = [HasStdExtQ]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 36-47: Bulk record instantiation Ext / 批量记录实例化 Ext
```tablegen
foreach Ext = QExts in {
  let SchedRW = [WriteFMA128, ReadFMA128, ReadFMA128, ReadFMA128Addend] in {
    defm FMADD_Q : FPFMA_rrr_frm_m<OPC_MADD, 0b11, "fmadd.q", Ext>;
    defm FMSUB_Q : FPFMA_rrr_frm_m<OPC_MSUB, 0b11, "fmsub.q", Ext>;
    defm FNMSUB_Q : FPFMA_rrr_frm_m<OPC_NMSUB, 0b11, "fnmsub.q", Ext>;
    defm FNMADD_Q : FPFMA_rrr_frm_m<OPC_NMADD, 0b11, "fnmadd.q", Ext>;
  }

  let SchedRW = [WriteFAdd128, ReadFAdd128, ReadFAdd128] in {
    defm FADD_Q : FPALU_rr_frm_m<0b0000011, "fadd.q", Ext>;
    defm FSUB_Q : FPALU_rr_frm_m<0b0000111, "fsub.q", Ext>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 48-58: Bulk record instantiation FMUL_Q / 批量记录实例化 FMUL_Q
```tablegen

  let SchedRW = [WriteFMul128, ReadFMul128, ReadFMul128] in 
  defm FMUL_Q : FPALU_rr_frm_m<0b0001011, "fmul.q", Ext>;

  let SchedRW = [WriteFDiv128, ReadFDiv128, ReadFDiv128] in 
  defm FDIV_Q : FPALU_rr_frm_m<0b0001111, "fdiv.q", Ext>;

  defm FSQRT_Q : FPUnaryOp_r_frm_m<0b0101111, 0b00000, Ext, Ext.PrimaryTy,
                                   Ext.PrimaryTy, "fsqrt.q">,
                 Sched<[WriteFSqrt128, ReadFSqrt128]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 59-69: Bulk record instantiation FSGNJ_Q / 批量记录实例化 FSGNJ_Q
```tablegen
  let SchedRW = [WriteFSGNJ128, ReadFSGNJ128, ReadFSGNJ128],
      mayRaiseFPException = 0 in {
    defm FSGNJ_Q : FPALU_rr_m<0b0010011, 0b000, "fsgnj.q", Ext>;
    defm FSGNJN_Q : FPALU_rr_m<0b0010011, 0b001, "fsgnjn.q", Ext>;
    defm FSGNJX_Q : FPALU_rr_m<0b0010011, 0b010, "fsgnjx.q", Ext>;
  }

  let SchedRW = [WriteFMinMax128, ReadFMinMax128, ReadFMinMax128] in {
    defm FMIN_Q : FPALU_rr_m<0b0010111, 0b000, "fmin.q", Ext, Commutable = 1>;
    defm FMAX_Q : FPALU_rr_m<0b0010111, 0b001, "fmax.q", Ext, Commutable = 1>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 70-79: Bulk record instantiation FCVT_S_Q / 批量记录实例化 FCVT_S_Q
```tablegen

  defm FCVT_S_Q : FPUnaryOp_r_frm_m<0b0100000, 0b00011, Ext, Ext.F32Ty,
                                    Ext.PrimaryTy, "fcvt.s.q">,
                  Sched<[WriteFCvtF128ToF32, ReadFCvtF128ToF32]>;

  defm FCVT_Q_S : FPUnaryOp_r_frmlegacy_m<0b0100011, 0b00000, Ext,
                                          Ext.PrimaryTy, Ext.F32Ty, 
                                          "fcvt.q.s">,
                  Sched<[WriteFCvtF32ToF128, ReadFCvtF32ToF128]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 80-93: Bulk record instantiation FCVT_D_Q / 批量记录实例化 FCVT_D_Q
```tablegen
  defm FCVT_D_Q : FPUnaryOp_r_frm_m<0b0100001, 0b00011, Ext, Ext.F64Ty,
                                    Ext.PrimaryTy, "fcvt.d.q">,
                  Sched<[WriteFCvtF128ToF64, ReadFCvtF128ToF64]>;

  defm FCVT_Q_D : FPUnaryOp_r_frmlegacy_m<0b0100011, 0b00001, Ext,
                                          Ext.PrimaryTy, Ext.F64Ty, 
                                          "fcvt.q.d">,
                  Sched<[WriteFCvtF64ToF128, ReadFCvtF64ToF128]>;

  let SchedRW = [WriteFCmp128, ReadFCmp128, ReadFCmp128] in {
    defm FEQ_Q : FPCmp_rr_m<0b1010011, 0b010, "feq.q", Ext, Commutable = 1>;
    defm FLT_Q : FPCmp_rr_m<0b1010011, 0b001, "flt.q", Ext>;
    defm FLE_Q : FPCmp_rr_m<0b1010011, 0b000, "fle.q", Ext>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 94-104: Bulk record instantiation FCLASS_Q / 批量记录实例化 FCLASS_Q
```tablegen

  let mayRaiseFPException = 0 in 
  defm FCLASS_Q : FPUnaryOp_r_m<0b1110011, 0b00000, 0b001, Ext, GPR,
                                Ext.PrimaryTy, "fclass.q">,
                  Sched<[WriteFClass128, ReadFClass128]>;

  let IsSignExtendingOpW = 1 in 
  defm FCVT_W_Q  : FPUnaryOp_r_frm_m<0b1100011, 0b00000, Ext, GPR,
                                     Ext.PrimaryTy, "fcvt.w.q">,
                   Sched<[WriteFCvtF128ToI32, ReadFCvtF128ToI32]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 105-114: Bulk record instantiation FCVT_WU_Q / 批量记录实例化 FCVT_WU_Q
```tablegen
  let IsSignExtendingOpW = 1 in 
  defm FCVT_WU_Q : FPUnaryOp_r_frm_m<0b1100011, 0b00001, Ext, GPR,
                                     Ext.PrimaryTy, "fcvt.wu.q">,
                   Sched<[WriteFCvtF128ToI32, ReadFCvtF128ToI32]>;

  let mayRaiseFPException = 0 in 
  defm FCVT_Q_W  : FPUnaryOp_r_frmlegacy_m<0b1101011, 0b00000, Ext,
                                           Ext.PrimaryTy, GPR, "fcvt.q.w">,
                   Sched<[WriteFCvtI32ToF128, ReadFCvtI32ToF128]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 115-125: Bulk record instantiation FCVT_Q_WU / 批量记录实例化 FCVT_Q_WU
```tablegen
  let mayRaiseFPException = 0 in 
  defm FCVT_Q_WU : FPUnaryOp_r_frmlegacy_m<0b1101011, 0b00001, Ext,
                                           Ext.PrimaryTy, GPR, "fcvt.q.wu">,
                   Sched<[WriteFCvtI32ToF128, ReadFCvtI32ToF128]>;
} // foreach Ext = QExts

foreach Ext = QExtsRV64 in {
  defm FCVT_L_Q  : FPUnaryOp_r_frm_m<0b1100011, 0b00010, Ext, GPR, 
                                     Ext.PrimaryTy, "fcvt.l.q", [IsRV64]>,
                   Sched<[WriteFCvtF128ToI64, ReadFCvtF128ToI64]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 126-135: Bulk record instantiation FCVT_LU_Q / 批量记录实例化 FCVT_LU_Q
```tablegen
  defm FCVT_LU_Q : FPUnaryOp_r_frm_m<0b1100011, 0b00011, Ext, GPR,
                                     Ext.PrimaryTy, "fcvt.lu.q", [IsRV64]>,
                   Sched<[WriteFCvtF128ToI64, ReadFCvtF128ToI64]>;

  let mayRaiseFPException = 0 in 
  defm FCVT_Q_L : FPUnaryOp_r_frmlegacy_m<0b1101011, 0b00010, Ext,
                                          Ext.PrimaryTy, GPR, "fcvt.q.l",
                                          [IsRV64]>,
                  Sched<[WriteFCvtI64ToF128, ReadFCvtI64ToF128]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 136-145: Bulk record instantiation FCVT_Q_LU / 批量记录实例化 FCVT_Q_LU
```tablegen
  let mayRaiseFPException = 0 in 
  defm FCVT_Q_LU : FPUnaryOp_r_frmlegacy_m<0b1101011, 0b00011, Ext,
                                           Ext.PrimaryTy, GPR, "fcvt.q.lu",
                                           [IsRV64]>,
                   Sched<[WriteFCvtI64ToF128, ReadFCvtI64ToF128]>;
} // foreach Ext = QExtsRV64

//===----------------------------------------------------------------------===//
// Assembler Pseudo Instructions (User-Level ISA, Version 2.2, Chapter 20)
//===----------------------------------------------------------------------===//
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 146-157: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasStdExtQ] in {
  def : InstAlias<"flq $rd, (${rs1})",  (FLQ FPR128:$rd,  GPR:$rs1, 0), 0>;
  def : InstAlias<"fsq $rs2, (${rs1})", (FSQ FPR128:$rs2, GPR:$rs1, 0), 0>;

  def : InstAlias<"fmv.q $rd, $rs",  (FSGNJ_Q FPR128:$rd, FPR128:$rs,
                                      FPR128:$rs)>;
  def : InstAlias<"fabs.q $rd, $rs", (FSGNJX_Q FPR128:$rd, FPR128:$rs,
                                      FPR128:$rs)>;
  def : InstAlias<"fneg.q $rd, $rs", (FSGNJN_Q FPR128:$rd, FPR128:$rs,
                                      FPR128:$rs)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 158-167: TableGen record PseudoFLQ / TableGen 记录 PseudoFLQ
```tablegen
  // fgt.q/fge.q are recognised by the GNU assembler but the canonical
  // flt.q/fle.q forms will always be printed. Therefore, set a zero weight.
  def : InstAlias<"fgt.q $rd, $rs, $rt",
                  (FLT_Q GPR:$rd, FPR128:$rt, FPR128:$rs), 0>;
  def : InstAlias<"fge.q $rd, $rs, $rt",
                  (FLE_Q GPR:$rd, FPR128:$rt, FPR128:$rs), 0>;

  def PseudoFLQ : PseudoFloatLoad<"flq", FPR128>;
  def PseudoFSQ : PseudoStore<"fsq", FPR128>;
} // Predicates = [HasStdExtQ]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
