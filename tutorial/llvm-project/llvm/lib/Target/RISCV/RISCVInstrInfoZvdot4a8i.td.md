# RISCVInstrInfoZvdot4a8i.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZvdot4a8i.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//==-- RISCVInstrInfoZvdot4a8i.td - 'Zvdot4a8i' instructions -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Zvdot4a8i'
// extension.
// This version is still experimental as the 'Zvdot4a8i' extension hasn't been
// ratified yet.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-28: TableGen class VDOT4AVV<bits<6> / TableGen 类 VDOT4AVV<bits<6>
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

class VDOT4AVV<bits<6> funct6, RISCVVFormat opv, string opcodestr>
    : RVInstVV<funct6, opv, (outs VR:$vd_wb),
                (ins VR:$vd, VR:$vs2, VR:$vs1, VMaskOp:$vm),
                opcodestr, "$vd, $vs2, $vs1$vm"> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$vd = $vd_wb";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 29-38: TableGen class VDOT4AVX<bits<6> / TableGen 类 VDOT4AVX<bits<6>
```tablegen

class VDOT4AVX<bits<6> funct6, RISCVVFormat opv, string opcodestr>
    : RVInstVX<funct6, opv, (outs VR:$vd_wb),
                (ins VR:$vd, VR:$vs2, GPR:$rs1, VMaskOp:$vm),
                opcodestr, "$vd, $vs2, $rs1$vm"> {
  let mayLoad = 0;
  let mayStore = 0;
  let hasSideEffects = 0;
  let Constraints = "$vd = $vd_wb";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 39-49: TableGen record VDOT4A_VV / TableGen 记录 VDOT4A_VV
```tablegen

let Predicates = [HasStdExtZvdot4a8i] in {
  def VDOT4A_VV   : VDOT4AVV<0b101100, OPMVV, "vdot4a.vv">;
  def VDOT4A_VX   : VDOT4AVX<0b101100, OPMVX, "vdot4a.vx">;
  def VDOT4AU_VV  : VDOT4AVV<0b101000, OPMVV, "vdot4au.vv">;
  def VDOT4AU_VX  : VDOT4AVX<0b101000, OPMVX, "vdot4au.vx">;
  def VDOT4ASU_VV : VDOT4AVV<0b101010, OPMVV, "vdot4asu.vv">;
  def VDOT4ASU_VX : VDOT4AVX<0b101010, OPMVX, "vdot4asu.vx">;
  def VDOT4AUS_VX : VDOT4AVX<0b101110, OPMVX, "vdot4aus.vx">;
} // Predicates = [HasStdExtZvdot4a8i]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 50-59: TableGen record riscv_vdot4a_vl / TableGen 记录 riscv_vdot4a_vl
```tablegen
//===----------------------------------------------------------------------===//
// Helpers to define the VL patterns.
//===----------------------------------------------------------------------===//

let HasPassthruOp = true, HasMaskOp = true in {
  def riscv_vdot4a_vl : RVSDNode<"VDOT4A_VL", SDT_RISCVIntBinOp_VL>;
  def riscv_vdot4au_vl : RVSDNode<"VDOT4AU_VL", SDT_RISCVIntBinOp_VL>;
  def riscv_vdot4asu_vl : RVSDNode<"VDOT4ASU_VL", SDT_RISCVIntBinOp_VL>;
} // let HasPassthruOp = true, HasMaskOp = true
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 60-72: Reusable TableGen multiclass VPseudoVDOT4A_VV_VX / 可复用的 TableGen 多类 VPseudoVDOT4A_VV_VX
```tablegen
//===----------------------------------------------------------------------===//
// Pseudo Instructions for CodeGen
//===----------------------------------------------------------------------===//

multiclass VPseudoVDOT4A_VV_VX<bit Commutable = 0> {
  foreach m = MxSet<32>.m in {
    defm "" : VPseudoBinaryV_VV<m, Commutable=Commutable>,
              SchedBinary<"WriteVIMulAddV", "ReadVIMulAddV", "ReadVIMulAddV", m.MX,
                          forcePassthruRead=true>;
    defm "" : VPseudoBinaryV_VX<m>,
              SchedBinary<"WriteVIMulAddX", "ReadVIMulAddV", "ReadVIMulAddX", m.MX,
                          forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 73-83: Bulk record instantiation PseudoVDOT4A / 批量记录实例化 PseudoVDOT4A
```tablegen
}

let Predicates = [HasStdExtZvdot4a8i], mayLoad = 0, mayStore = 0,
    hasSideEffects = 0 in {
  defm PseudoVDOT4A : VPseudoVDOT4A_VV_VX<Commutable=1>;
  defm PseudoVDOT4AU : VPseudoVDOT4A_VV_VX<Commutable=1>;
  defm PseudoVDOT4ASU : VPseudoVDOT4A_VV_VX;
  // VDOT4AUS does not have a VV variant
  foreach m = MxListVF4 in {
    defm "PseudoVDOT4AUS_VX" : VPseudoTernaryWithPolicy<m.vrclass, m.vrclass, GPR, m>;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 84-94: Bulk record instantiation / 批量记录实例化
```tablegen
}

//===----------------------------------------------------------------------===//
// Patterns.
//===----------------------------------------------------------------------===//

defvar AllE32Vectors = [VI32MF2, VI32M1, VI32M2, VI32M4, VI32M8];
defm : VPatBinaryVL_VV_VX<riscv_vdot4a_vl, "PseudoVDOT4A", AllE32Vectors, ExtraPreds=[HasStdExtZvdot4a8i]>;
defm : VPatBinaryVL_VV_VX<riscv_vdot4au_vl, "PseudoVDOT4AU", AllE32Vectors, ExtraPreds=[HasStdExtZvdot4a8i]>;
defm : VPatBinaryVL_VV_VX<riscv_vdot4asu_vl, "PseudoVDOT4ASU", AllE32Vectors, ExtraPreds=[HasStdExtZvdot4a8i]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 95-108: Reusable TableGen multiclass VPatTernaryV_VX_AAAX / 可复用的 TableGen 多类 VPatTernaryV_VX_AAAX
```tablegen
// These VPat definitions are for vdot4a because they have a different operand
// order with other ternary instructions (i.e. vop.vx vd, vs2, rs1)
multiclass VPatTernaryV_VX_AAAX<string intrinsic, string instruction,
                                list<VTypeInfo> vtilist,
                                list<Predicate> ExtraPreds> {
  foreach vti = vtilist in
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    defm : VPatTernaryWithPolicy<intrinsic, instruction,
                                 "V"#vti.ScalarSuffix,
                                 vti.Vector, vti.Vector, vti.Scalar,
                                 vti.Mask, vti.Log2SEW, vti.LMul,
                                 vti.RegClass, vti.RegClass,
                                 vti.ScalarRegClass>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 109-121: Reusable TableGen multiclass VPatTernaryV_VV_AAAX / 可复用的 TableGen 多类 VPatTernaryV_VV_AAAX
```tablegen

multiclass VPatTernaryV_VV_AAAX<string intrinsic, string instruction,
                                list<VTypeInfo> vtilist,
                                list<Predicate> ExtraPreds> {
  foreach vti = vtilist in
    let Predicates = !listconcat(ExtraPreds, GetVTypePredicates<vti>.Predicates) in
    defm : VPatTernaryWithPolicy<intrinsic, instruction,
                                 "VV",
                                 vti.Vector, vti.Vector, vti.Vector,
                                 vti.Mask, vti.Log2SEW, vti.LMul,
                                 vti.RegClass, vti.RegClass,
                                 vti.RegClass>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 122-132: Reusable TableGen multiclass VPatTernaryV_VV_VX_AAAX / 可复用的 TableGen 多类 VPatTernaryV_VV_VX_AAAX
```tablegen

multiclass VPatTernaryV_VV_VX_AAAX<string intrinsic, string instruction,
                                   list<VTypeInfo> vtilist,
                                   list<Predicate> ExtraPreds>
    : VPatTernaryV_VV_AAAX<intrinsic, instruction, vtilist, ExtraPreds>,
      VPatTernaryV_VX_AAAX<intrinsic, instruction, vtilist, ExtraPreds>;

defm : VPatTernaryV_VV_VX_AAAX<"int_riscv_vdot4a", "PseudoVDOT4A", AllE32Vectors, [HasStdExtZvdot4a8i]>;
defm : VPatTernaryV_VV_VX_AAAX<"int_riscv_vdot4au", "PseudoVDOT4AU", AllE32Vectors, [HasStdExtZvdot4a8i]>;
defm : VPatTernaryV_VV_VX_AAAX<"int_riscv_vdot4asu", "PseudoVDOT4ASU", AllE32Vectors, [HasStdExtZvdot4a8i]>;
defm : VPatTernaryV_VX_AAAX<"int_riscv_vdot4aus", "PseudoVDOT4AUS", AllE32Vectors, [HasStdExtZvdot4a8i]>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
