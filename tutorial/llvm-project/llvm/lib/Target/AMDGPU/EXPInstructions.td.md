# EXPInstructions.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/EXPInstructions.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines EXPInstructions records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 EXPInstructions 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, comments, and TableGen overview
```tablegen
//===-- EXPInstructions.td - Export Instruction Definitions ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// EXP classes
//===----------------------------------------------------------------------===//

class EXPCommon<bit _row, bit _done, string asm = ""> : InstSI<
  (outs),
  (ins exp_tgt:$tgt,
       ExpSrc0:$src0, ExpSrc1:$src1, ExpSrc2:$src2, ExpSrc3:$src3,
       exp_vm:$vm, exp_compr:$compr, i32imm:$en),
  asm> {
  let EXP = 1;
  let EXP_CNT = 1;
  let mayLoad = _done;
  let mayStore = 1;
  let maybeAtomic = 0;
  let UseNamedOperandTable = 1;
```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `EXPCommon`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`EXPCommon`。

### Lines 25-46: Defines TableGen class EXP_Pseudo
```tablegen
  let Uses = !if(_row, [EXEC, M0], [EXEC]);
  let SchedRW = [WriteExport];
  let DisableWQM = 1;

  bit row = _row;
  bit done = _done;
}

class EXP_Pseudo<bit row, bit done>
  : EXPCommon<row, done>, SIMCInstr<NAME, SIEncodingFamily.NONE> {
  let isPseudo = 1;
  let isCodeGenOnly = 1;
}

// Real instruction with optional asm operands "compr" and "vm".
class EXP_Real_ComprVM<EXP_Pseudo ps, int subtarget>
  : EXPCommon<0, ps.done, "exp$tgt, $src0, $src1, $src2, $src3"
                       #!if(ps.done, " done", "")#"$compr$vm">,
    SIMCInstr<ps.PseudoInstr, subtarget> {
  let AsmMatchConverter = "cvtExp";
}

```
**EN:** This section contains concrete logic for TableGen class EXP_Pseudo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `EXP_Pseudo`, `EXP_Real_ComprVM`.
**CN:** 本节包含与 TableGen class EXP_Pseudo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`EXP_Pseudo`, `EXP_Real_ComprVM`。

### Lines 47-67: Defines TableGen class EXP_Real_Row
```tablegen
// Real instruction with optional asm operand "row_en".
class EXP_Real_Row<EXP_Pseudo ps, int subtarget, string name = "exp">
  : EXPCommon<ps.row, ps.done, name#"$tgt, $src0, $src1, $src2, $src3"
                         #!if(ps.done, " done", "")#!if(ps.row, " row_en", "")>,
    SIMCInstr<ps.PseudoInstr, subtarget> {
  let AsmMatchConverter = "cvtExp";
}

//===----------------------------------------------------------------------===//
// EXP Instructions
//===----------------------------------------------------------------------===//

// DONE variants have mayLoad = 1.
// ROW variants have an implicit use of M0.
let SubtargetPredicate = HasExportInsts in {
def EXP          : EXP_Pseudo<0, 0>;
def EXP_DONE     : EXP_Pseudo<0, 1>;
def EXP_ROW      : EXP_Pseudo<1, 0>;
def EXP_ROW_DONE : EXP_Pseudo<1, 1>;
} // let SubtargetPredicate = HasExportInsts

```
**EN:** This section contains concrete logic for TableGen class EXP_Real_Row. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `EXP_Real_Row`, `EXP`, `EXP_DONE`.
**CN:** 本节包含与 TableGen class EXP_Real_Row 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`EXP_Real_Row`, `EXP`, `EXP_DONE`。

### Lines 68-90: Defines TableGen multiclass EXP_Real_si
```tablegen
//===----------------------------------------------------------------------===//
// SI, VI, GFX10.
//===----------------------------------------------------------------------===//

multiclass EXP_Real_si {
  defvar ps = !cast<EXP_Pseudo>(NAME);
  def _si : EXP_Real_ComprVM<ps, SIEncodingFamily.SI>, EXPe_ComprVM {
    let AssemblerPredicate = isGFX6GFX7;
    let DecoderNamespace = "GFX6GFX7";
    let done = ps.done;
  }
}

multiclass EXP_Real_vi {
  defvar ps = !cast<EXP_Pseudo>(NAME);
  def _vi : EXP_Real_ComprVM<ps, SIEncodingFamily.VI>, EXPe_vi {
    let AssemblerPredicate = isGFX8GFX9;
    let SubtargetPredicate = isNotGFX90APlus;
    let DecoderNamespace = "GFX8";
    let done = ps.done;
  }
}

```
**EN:** This section contains concrete logic for TableGen multiclass EXP_Real_si. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `EXP_Real_si`, `_si`, `EXP_Real_vi`.
**CN:** 本节包含与 TableGen multiclass EXP_Real_si 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`EXP_Real_si`, `_si`, `EXP_Real_vi`。

### Lines 91-106: Defines TableGen multiclass EXP_Real_gfx10
```tablegen
multiclass EXP_Real_gfx10 {
  defvar ps = !cast<EXP_Pseudo>(NAME);
  def _gfx10 : EXP_Real_ComprVM<ps, SIEncodingFamily.GFX10>, EXPe_ComprVM {
    let AssemblerPredicate = isGFX10Only;
    let DecoderNamespace = "GFX10";
    let done = ps.done;
  }
}

defm EXP      : EXP_Real_si, EXP_Real_vi, EXP_Real_gfx10;
defm EXP_DONE : EXP_Real_si, EXP_Real_vi, EXP_Real_gfx10;

//===----------------------------------------------------------------------===//
// GFX11, GFX12, GFX13.
//===----------------------------------------------------------------------===//

```
**EN:** This section contains concrete logic for TableGen multiclass EXP_Real_gfx10. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `EXP_Real_gfx10`, `_gfx10`, `EXP`.
**CN:** 本节包含与 TableGen multiclass EXP_Real_gfx10 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`EXP_Real_gfx10`, `_gfx10`, `EXP`。

### Lines 107-128: Defines TableGen multiclass EXP_Real_gfx11
```tablegen
multiclass EXP_Real_gfx11 {
  defvar ps = !cast<EXP_Pseudo>(NAME);
  def _gfx11 : EXP_Real_Row<ps, SIEncodingFamily.GFX11>, EXPe_Row {
    let AssemblerPredicate = isGFX11Only;
    let DecoderNamespace = "GFX11";
    let row = ps.row;
    let done = ps.done;
  }
}

multiclass VEXPORT_Real<GFXGen Gen> {
  defvar ps = !cast<EXP_Pseudo>(NAME);
  def Gen.Suffix : EXP_Real_Row<ps, Gen.Subtarget, "export">,
    EXPe_Row, MnemonicAlias<"exp", "export">, Requires<[Gen.AssemblerPredicate]> {
    let AssemblerPredicate = Gen.AssemblerPredicate;
    let OtherPredicates = [HasExportInsts];
    let DecoderNamespace = Gen.DecoderNamespace;
    let row = ps.row;
    let done = ps.done;
  }
}

```
**EN:** This section contains concrete logic for TableGen multiclass EXP_Real_gfx11. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `EXP_Real_gfx11`, `_gfx11`, `VEXPORT_Real`.
**CN:** 本节包含与 TableGen multiclass EXP_Real_gfx11 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`EXP_Real_gfx11`, `_gfx11`, `VEXPORT_Real`。

### Lines 129-146: Defines TableGen record group EXP
```tablegen
defm EXP          : EXP_Real_gfx11, VEXPORT_Real<GFX12Not12_50Gen>, VEXPORT_Real<GFX13Gen>;
defm EXP_DONE     : EXP_Real_gfx11, VEXPORT_Real<GFX12Not12_50Gen>, VEXPORT_Real<GFX13Gen>;
defm EXP_ROW      : EXP_Real_gfx11, VEXPORT_Real<GFX12Not12_50Gen>, VEXPORT_Real<GFX13Gen>;
defm EXP_ROW_DONE : EXP_Real_gfx11, VEXPORT_Real<GFX12Not12_50Gen>, VEXPORT_Real<GFX13Gen>;

//===----------------------------------------------------------------------===//
// EXP Patterns
//===----------------------------------------------------------------------===//

class ExpPattern<ValueType vt, Instruction Inst, int done_val> : GCNPat<
  (int_amdgcn_exp timm:$tgt, timm:$en,
                  (vt ExpSrc0:$src0), (vt ExpSrc1:$src1),
                  (vt ExpSrc2:$src2), (vt ExpSrc3:$src3),
                  done_val, timm:$vm),
  (Inst timm:$tgt, ExpSrc0:$src0, ExpSrc1:$src1,
        ExpSrc2:$src2, ExpSrc3:$src3, timm:$vm, 0, timm:$en)
>;

```
**EN:** This section contains concrete logic for TableGen record group EXP. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `EXP`, `EXP_DONE`, `EXP_ROW`.
**CN:** 本节包含与 TableGen record group EXP 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`EXP`, `EXP_DONE`, `EXP_ROW`。

### Lines 147-163: Defines TableGen class ExpRowPattern
```tablegen
class ExpRowPattern<ValueType vt, Instruction Inst, int done_val> : GCNPat<
  (int_amdgcn_exp_row timm:$tgt, timm:$en,
                      (vt ExpSrc0:$src0), (vt ExpSrc1:$src1),
                      (vt ExpSrc2:$src2), (vt ExpSrc3:$src3),
                      done_val, M0),
  (Inst timm:$tgt, ExpSrc0:$src0, ExpSrc1:$src1,
        ExpSrc2:$src2, ExpSrc3:$src3, 0, 0, timm:$en)
>;

class ExpComprPattern<ValueType vt, Instruction Inst, int done_val> : GCNPat<
  (int_amdgcn_exp_compr timm:$tgt, timm:$en,
                        (vt ExpSrc0:$src0), (vt ExpSrc1:$src1),
                        done_val, timm:$vm),
  (Inst timm:$tgt, ExpSrc0:$src0, ExpSrc1:$src1,
        (IMPLICIT_DEF), (IMPLICIT_DEF), timm:$vm, 1, timm:$en)
>;

```
**EN:** This section contains concrete logic for TableGen class ExpRowPattern. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `ExpRowPattern`, `ExpComprPattern`.
**CN:** 本节包含与 TableGen class ExpRowPattern 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`ExpRowPattern`, `ExpComprPattern`。

### Lines 164-180: TableGen records and backend metadata
```tablegen
// FIXME: The generated DAG matcher seems to have strange behavior
// with a 1-bit literal to match, so use a -1 for checking a true
// 1-bit value.
def : ExpPattern<i32, EXP, 0>;
def : ExpPattern<i32, EXP_DONE, -1>;
def : ExpPattern<f32, EXP, 0>;
def : ExpPattern<f32, EXP_DONE, -1>;

def : ExpRowPattern<i32, EXP_ROW, 0>;
def : ExpRowPattern<i32, EXP_ROW_DONE, -1>;
def : ExpRowPattern<f32, EXP_ROW, 0>;
def : ExpRowPattern<f32, EXP_ROW_DONE, -1>;

def : ExpComprPattern<v2i16, EXP, 0>;
def : ExpComprPattern<v2i16, EXP_DONE, -1>;
def : ExpComprPattern<v2f16, EXP, 0>;
def : ExpComprPattern<v2f16, EXP_DONE, -1>;
```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `EXPCommon`, `EXP_Pseudo`, `EXP_Real_ComprVM`, `EXP_Real_Row`, `EXP`, `EXP_DONE`
- **Main themes / 核心主题**: alias analysis / 别名分析; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
