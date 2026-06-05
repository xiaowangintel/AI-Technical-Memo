# RISCVInstrInfoZvabd.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZvabd.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZvabd.td - 'Zvabd' instructions ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// This file describes the RISC-V instructions for 'Zvabd' (Vector Absolute
/// Difference).
///
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-22: TableGen record VABS_V / TableGen 记录 VABS_V
```tablegen

//===----------------------------------------------------------------------===//
// Instruction Definitions
//===----------------------------------------------------------------------===//
let Predicates = [HasStdExtZvabd], ElementsDependOn = EltDepsNone in {
  def VABS_V : VALUVs2<0b010010, 0b10000, OPMVV, "vabs.v">;

  def VABD_VV  : VALUVV<0b010001, OPMVV, "vabd.vv">;
  def VABDU_VV : VALUVV<0b010011, OPMVV, "vabdu.vv">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 23-31: TableGen record VWABDA_VV / TableGen 记录 VWABDA_VV
```tablegen
  let Constraints = "@earlyclobber $vd", VS1VS2Constraint = WidenV in {
    def VWABDA_VV  : VALUVV<0b010101, OPMVV, "vwabda.vv">;
    def VWABDAU_VV : VALUVV<0b010110, OPMVV, "vwabdau.vv">;
  } // Constraints = "@earlyclobber $vd", VS1VS2Constraint = WidenV
} // Predicates = [HasStdExtZvabd], ElementsDependOn = EltDepsNone

//===----------------------------------------------------------------------===//
// Pseudos
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 32-42: Reusable TableGen multiclass PseudoVABS / 可复用的 TableGen 多类 PseudoVABS
```tablegen
multiclass PseudoVABS {
  foreach m = MxList in {
    defvar mx = m.MX;
    let VLMul = m.value in {
      def "_V_" # mx : VPseudoUnaryNoMask<m.vrclass, m.vrclass>,
                       SchedUnary<"WriteVIALUV", "ReadVIALUV", mx, forcePassthruRead=true>;
      def "_V_" # mx # "_MASK" :
        VPseudoUnaryMask<m.vrclass, m.vrclass>,
        RISCVMaskedPseudo<MaskIdx=2>,
        SchedUnary<"WriteVIALUV", "ReadVIALUV", mx, forcePassthruRead=true>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 43-52: Reusable TableGen multiclass VPseudoVWABD_VV / 可复用的 TableGen 多类 VPseudoVWABD_VV
```tablegen
  }
}

multiclass VPseudoVWABD_VV {
  foreach m = MxListW in {
    defvar mx = m.MX;
    defm "" : VPseudoTernaryW_VV<m, Commutable = 1>,
              SchedTernary<"WriteVIWMulAddV", "ReadVIWMulAddV",
                           "ReadVIWMulAddV", "ReadVIWMulAddV", mx>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 53-62: Bulk record instantiation PseudoVABS / 批量记录实例化 PseudoVABS
```tablegen
}

let Predicates = [HasStdExtZvabd] in {
  defm PseudoVABS : PseudoVABS;
  defm PseudoVABD : VPseudoVALU_VV<Commutable = 1>;
  defm PseudoVABDU : VPseudoVALU_VV<Commutable = 1>;
  defm PseudoVWABDA : VPseudoVWABD_VV;
  defm PseudoVWABDAU : VPseudoVWABD_VV;
} // Predicates = [HasStdExtZvabd]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 63-73: TableGen record riscv_abs_vl / TableGen 记录 riscv_abs_vl
```tablegen
//===----------------------------------------------------------------------===//
// CodeGen Patterns
//===----------------------------------------------------------------------===//
let HasPassthruOp = true, HasMaskOp = true in {
def riscv_abs_vl  : RVSDNode<"ABS_VL", SDT_RISCVIntUnOp_VL>;
def riscv_abds_vl : RVSDNode<"ABDS_VL", SDT_RISCVIntBinOp_VL, [SDNPCommutative]>;
def riscv_abdu_vl : RVSDNode<"ABDU_VL", SDT_RISCVIntBinOp_VL, [SDNPCommutative]>;
def rvv_vwabda_vl  : RVSDNode<"VWABDA_VL", SDT_RISCVVWIntTernOp_VL, [SDNPCommutative]>;
def rvv_vwabdau_vl : RVSDNode<"VWABDAU_VL", SDT_RISCVVWIntTernOp_VL, [SDNPCommutative]>;
} // let HasPassthruOp = true, HasMaskOp = true
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 74-81: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
// These instructions are defined for SEW=8 and SEW=16, otherwise the instruction
// encoding is reserved.
defvar ABDIntVectors = !filter(vti, AllIntegerVectors, !or(!eq(vti.SEW, 8),
                                                           !eq(vti.SEW, 16)));
defvar ABDAIntVectors = !filter(vtiTowti, AllWidenableIntVectors,
                                          !or(!eq(vtiTowti.Vti.SEW, 8),
                                              !eq(vtiTowti.Vti.SEW, 16)));
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 82-94: Bulk record instantiation / 批量记录实例化
```tablegen
let Predicates = [HasStdExtZvabd] in {
defm : VPatBinarySDNode_VV<abds, "PseudoVABD", ABDIntVectors>;
defm : VPatBinarySDNode_VV<abdu, "PseudoVABDU", ABDIntVectors>;

defm : VPatBinaryVL_VV<riscv_abds_vl, "PseudoVABD", ABDIntVectors>;
defm : VPatBinaryVL_VV<riscv_abdu_vl, "PseudoVABDU", ABDIntVectors>;

defm : VPatUnaryV_V<"int_riscv_vabs", "PseudoVABS", AllIntegerVectors>;
defm : VPatBinaryV_VV<"int_riscv_vabd", "PseudoVABD", ABDIntVectors>;
defm : VPatBinaryV_VV<"int_riscv_vabdu", "PseudoVABDU", ABDIntVectors>;
defm : VPatTernaryW_VV<"int_riscv_vwabda", "PseudoVWABDA", ABDAIntVectors>;
defm : VPatTernaryW_VV<"int_riscv_vwabdau", "PseudoVWABDAU", ABDAIntVectors>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 95-103: Bulk record instantiation vti / 批量记录实例化 vti
```tablegen
foreach vti = AllIntegerVectors in {
  def : Pat<(vti.Vector (abs (vti.Vector vti.RegClass:$rs2))),
            (!cast<Instruction>("PseudoVABS_V_"#vti.LMul.MX)
                    (vti.Vector (IMPLICIT_DEF)),
                    vti.RegClass:$rs2, vti.AVL, vti.Log2SEW, TA_MA)>;
}

defm : VPatUnaryVL_V<riscv_abs_vl, "PseudoVABS", HasStdExtZvabd>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 104-106: Bulk record instantiation / 批量记录实例化
```tablegen
defm : VPatWidenMultiplyAddVL_VV<rvv_vwabda_vl, "PseudoVWABDA", ABDAIntVectors>;
defm : VPatWidenMultiplyAddVL_VV<rvv_vwabdau_vl, "PseudoVWABDAU", ABDAIntVectors>;
} // Predicates = [HasStdExtZvabd]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
