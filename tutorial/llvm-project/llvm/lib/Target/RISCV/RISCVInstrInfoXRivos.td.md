# RISCVInstrInfoXRivos.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoXRivos.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoXRivos.td ----------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the vendor extensions defined by Rivos Inc.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-20: TableGen class CustomRivosVXI<bits<6> / TableGen 类 CustomRivosVXI<bits<6>
```tablegen

class CustomRivosVXI<bits<6> funct6, RISCVVFormat opv, dag outs, dag ins,
                     string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> imm;
  bits<5> rs1;
  bits<5> vd;
  bit vm = 0;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 21-28: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-26} = funct6;
  let Inst{25} = vm;
  let Inst{24-20} = imm;
  let Inst{19-15} = rs1;
  let Inst{14-12} = opv.Value;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 29-43: TableGen class CustomRivosXVI<bits<6> / TableGen 类 CustomRivosXVI<bits<6>
```tablegen
  let Uses = [VL, VTYPE];
  let VMConstraint = false;
  let Constraints = "$vd = $vd_wb";

  let UseNamedOperandTable = true;
}

class CustomRivosXVI<bits<6> funct6, RISCVVFormat opv, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> imm;
  bits<5> vs2;
  bits<5> rd;
  bit vm = 1;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 44-51: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-26} = funct6;
  let Inst{25} = vm;
  let Inst{24-20} = vs2;
  let Inst{19-15} = imm;
  let Inst{14-12} = opv.Value;
  let Inst{11-7} = rd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 52-60: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Uses = [VL, VTYPE];
  let VMConstraint = false;

  let UseNamedOperandTable = true;
}

//===----------------------------------------------------------------------===//
// XRivosVizip
//===----------------------------------------------------------------------===//
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 61-73: Bulk record instantiation RI_VZIPEVEN_V / 批量记录实例化 RI_VZIPEVEN_V
```tablegen


let Predicates = [HasVendorXRivosVizip], DecoderNamespace = "XRivos",
  Constraints = "@earlyclobber $vd", VS1VS2Constraint = Vrgather,
  Inst{6-0} = OPC_CUSTOM_2.Value, ReadsPastVL = 1,
  ElementsDependOn = EltDepsNone in  {
defm RI_VZIPEVEN_V : VALU_IV_V<"ri.vzipeven", 0b001100>;
defm RI_VZIPODD_V : VALU_IV_V<"ri.vzipodd", 0b011100>;
defm RI_VZIP2A_V : VALU_IV_V<"ri.vzip2a", 0b000100>;
defm RI_VZIP2B_V : VALU_IV_V<"ri.vzip2b", 0b010100>;
defm RI_VUNZIP2A_V : VALU_IV_V<"ri.vunzip2a", 0b001000>;
defm RI_VUNZIP2B_V : VALU_IV_V<"ri.vunzip2b", 0b011000>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 74-83: TableGen record ri_vzipeven_vl / TableGen 记录 ri_vzipeven_vl
```tablegen

// These are modeled after the int binop VL nodes
let HasMaskOp = true in {
  def ri_vzipeven_vl : RVSDNode<"RI_VZIPEVEN_VL", SDT_RISCVIntBinOp_VL>;
  def ri_vzipodd_vl : RVSDNode<"RI_VZIPODD_VL", SDT_RISCVIntBinOp_VL>;
  def ri_vzip2a_vl : RVSDNode<"RI_VZIP2A_VL", SDT_RISCVIntBinOp_VL>;
  def ri_vzip2b_vl : RVSDNode<"RI_VZIP2B_VL", SDT_RISCVIntBinOp_VL>;
  def ri_vunzip2a_vl : RVSDNode<"RI_VUNZIP2A_VL", SDT_RISCVIntBinOp_VL>;
  def ri_vunzip2b_vl : RVSDNode<"RI_VUNZIP2B_VL", SDT_RISCVIntBinOp_VL>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 84-98: Reusable TableGen multiclass RIVPseudoVALU_VV / 可复用的 TableGen 多类 RIVPseudoVALU_VV
```tablegen

multiclass RIVPseudoVALU_VV {
  foreach m = MxList in
    defm "" : VPseudoBinaryV_VV<m, Commutable=0>;
}

let Predicates = [HasVendorXRivosVizip],
    Constraints = "@earlyclobber $rd, $rd = $passthru" in {
defm PseudoRI_VZIPEVEN   : RIVPseudoVALU_VV;
defm PseudoRI_VZIPODD   : RIVPseudoVALU_VV;
defm PseudoRI_VZIP2A   : RIVPseudoVALU_VV;
defm PseudoRI_VZIP2B   : RIVPseudoVALU_VV;
defm PseudoRI_VUNZIP2A   : RIVPseudoVALU_VV;
defm PseudoRI_VUNZIP2B   : RIVPseudoVALU_VV;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 99-109: Reusable TableGen multiclass RIVPatBinaryVL_VV / 可复用的 TableGen 多类 RIVPatBinaryVL_VV
```tablegen

multiclass RIVPatBinaryVL_VV<SDPatternOperator vop, string instruction_name,
                              list<VTypeInfo> vtilist = AllIntegerVectors,
                              bit isSEWAware = false> {
  foreach vti = vtilist in
    let Predicates = GetVTypePredicates<vti>.Predicates in
      def : VPatBinaryVL_V<vop, instruction_name, "VV",
                           vti.Vector, vti.Vector, vti.Vector, vti.Mask,
                           vti.Log2SEW, vti.LMul, vti.RegClass, vti.RegClass,
                           vti.RegClass, isSEWAware>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 110-116: Bulk record instantiation / 批量记录实例化
```tablegen

defm : RIVPatBinaryVL_VV<ri_vzipeven_vl, "PseudoRI_VZIPEVEN">;
defm : RIVPatBinaryVL_VV<ri_vzipodd_vl, "PseudoRI_VZIPODD">;
defm : RIVPatBinaryVL_VV<ri_vzip2a_vl, "PseudoRI_VZIP2A">;
defm : RIVPatBinaryVL_VV<ri_vzip2b_vl, "PseudoRI_VZIP2B">;
defm : RIVPatBinaryVL_VV<ri_vunzip2a_vl, "PseudoRI_VUNZIP2A">;
defm : RIVPatBinaryVL_VV<ri_vunzip2b_vl, "PseudoRI_VUNZIP2B">;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
