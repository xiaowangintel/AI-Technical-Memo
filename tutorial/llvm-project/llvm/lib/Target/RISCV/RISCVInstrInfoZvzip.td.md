# RISCVInstrInfoZvzip.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZvzip.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===---- RISCVInstrInfoZvzip.td - 'Zvzip' instructions ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the Zvzip standard extension
// for reordering structured data in vector registers.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-25: TableGen record VZIP_VV / TableGen 记录 VZIP_VV
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZvzip], Constraints = "@earlyclobber $vd" in {
  let VS1VS2Constraint = WidenV in
  def VZIP_VV : VALUVV<0b111110, OPMVV, "vzip.vv">;

  let VS1VS2Constraint = VS2Constraint in {
    def VUNZIPE_V : VALUVs2<0b010010, 0b01011, OPMVV, "vunzipe.v">;
    def VUNZIPO_V : VALUVs2<0b010010, 0b01111, OPMVV, "vunzipo.v">;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 26-40: TableGen record VPAIRE_VV / TableGen 记录 VPAIRE_VV
```tablegen

  let VS1VS2Constraint = Vrgather in {
    def VPAIRE_VV : VALUVV<0b001111, OPIVV, "vpaire.vv">;
    def VPAIRO_VV : VALUVV<0b001111, OPMVV, "vpairo.vv">;
  }
} // Predicates = [HasStdExtZvzip]

defset list<VTypeInfoToWide> AllZvzipVectors = {
  def : VTypeInfoToWide<VI8MF8,  VI8MF4>;
  def : VTypeInfoToWide<VI8MF4,  VI8MF2>;
  def : VTypeInfoToWide<VI8MF2,  VI8M1>;
  def : VTypeInfoToWide<VI8M1,   VI8M2>;
  def : VTypeInfoToWide<VI8M2,   VI8M4>;
  def : VTypeInfoToWide<VI8M4,   VI8M8>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 41-51: TableGen record definition / TableGen 记录定义
```tablegen
  def : VTypeInfoToWide<VI16MF4, VI16MF2>;
  def : VTypeInfoToWide<VI16MF2, VI16M1>;
  def : VTypeInfoToWide<VI16M1,  VI16M2>;
  def : VTypeInfoToWide<VI16M2,  VI16M4>;
  def : VTypeInfoToWide<VI16M4,  VI16M8>;

  def : VTypeInfoToWide<VI32MF2, VI32M1>;
  def : VTypeInfoToWide<VI32M1,  VI32M2>;
  def : VTypeInfoToWide<VI32M2,  VI32M4>;
  def : VTypeInfoToWide<VI32M4,  VI32M8>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 52-62: TableGen record definition / TableGen 记录定义
```tablegen
  def : VTypeInfoToWide<VI64M1,  VI64M2>;
  def : VTypeInfoToWide<VI64M2,  VI64M4>;
  def : VTypeInfoToWide<VI64M4,  VI64M8>;

  // Floating-point 16-bit
  def : VTypeInfoToWide<VF16MF4, VF16MF2>;
  def : VTypeInfoToWide<VF16MF2, VF16M1>;
  def : VTypeInfoToWide<VF16M1,  VF16M2>;
  def : VTypeInfoToWide<VF16M2,  VF16M4>;
  def : VTypeInfoToWide<VF16M4,  VF16M8>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 63-75: TableGen record definition / TableGen 记录定义
```tablegen
  // BF16
  def : VTypeInfoToWide<VBF16MF4, VBF16MF2>;
  def : VTypeInfoToWide<VBF16MF2, VBF16M1>;
  def : VTypeInfoToWide<VBF16M1,  VBF16M2>;
  def : VTypeInfoToWide<VBF16M2,  VBF16M4>;
  def : VTypeInfoToWide<VBF16M4,  VBF16M8>;

  // Floating-point 32-bit
  def : VTypeInfoToWide<VF32MF2, VF32M1>;
  def : VTypeInfoToWide<VF32M1,  VF32M2>;
  def : VTypeInfoToWide<VF32M2,  VF32M4>;
  def : VTypeInfoToWide<VF32M4,  VF32M8>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 76-87: Reusable TableGen multiclass VPseudoVZIP / 可复用的 TableGen 多类 VPseudoVZIP
```tablegen
  // Floating-point 64-bit
  def : VTypeInfoToWide<VF64M1,  VF64M2>;
  def : VTypeInfoToWide<VF64M2,  VF64M4>;
  def : VTypeInfoToWide<VF64M4,  VF64M8>;
}

multiclass VPseudoVZIP {
  foreach m = MxListW in
    defm "" : VPseudoBinaryW_VV<m, Commutable=0>,
              SchedBinary<"WriteVIWALUV", "ReadVIWALUV", "ReadVIWALUV", m.MX,
                          forcePassthruRead=true>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 88-100: Reusable TableGen multiclass VPseudoVUNZIP / 可复用的 TableGen 多类 VPseudoVUNZIP
```tablegen

multiclass VPseudoVUNZIP {
  foreach m = MxListW in {
    defvar mx = m.MX;
    let VLMul = m.value in {
      def "_V_" # mx : VPseudoUnaryNoMask<m.vrclass, m.wvrclass>,
                       SchedUnary<"WriteVIALUV", "ReadVIALUV", mx,
                                  forcePassthruRead=true>;
      def "_V_" # mx # "_MASK" :
        VPseudoUnaryMask<m.vrclass, m.wvrclass>,
        RISCVMaskedPseudo<MaskIdx=2>,
        SchedUnary<"WriteVIALUV", "ReadVIALUV", mx, forcePassthruRead=true>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 101-111: Bulk record instantiation PseudoVZIP / 批量记录实例化 PseudoVZIP
```tablegen
  }
}

let Predicates = [HasStdExtZvzip],
    Constraints = "@earlyclobber $rd, $rd = $passthru" in {
  defm PseudoVZIP    : VPseudoVZIP;
  defm PseudoVUNZIPE : VPseudoVUNZIP;
  defm PseudoVUNZIPO : VPseudoVUNZIP;
  defm PseudoVPAIRE  : VPseudoVALU_VV;
  defm PseudoVPAIRO  : VPseudoVALU_VV;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 112-126: Reusable TableGen multiclass VPatVUnzipIntrinsic / 可复用的 TableGen 多类 VPatVUnzipIntrinsic
```tablegen

multiclass VPatVUnzipIntrinsic<string intrinsic_name,
                               string instruction_name> {
  foreach VtiToWti = AllZvzipVectors in {
    defvar vti = VtiToWti.Vti;
    defvar wti = VtiToWti.Wti;
    let Predicates = !listconcat(GetVTypeMinimalPredicates<wti>.Predicates,
                                 [HasStdExtZvzip]) in {
      def : VPatUnaryNoMask<intrinsic_name, instruction_name, "V",
                            vti.Vector, wti.Vector, vti.Log2SEW,
                            vti.LMul, vti.RegClass, wti.RegClass>;
      def : VPatUnaryMask<intrinsic_name, instruction_name, "V",
                          vti.Vector, wti.Vector, vti.Mask, vti.Log2SEW,
                          vti.LMul, vti.RegClass, wti.RegClass>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 127-141: Bulk record instantiation / 批量记录实例化
```tablegen
  }
}

defm : VPatVUnzipIntrinsic<"int_riscv_vunzipe", "PseudoVUNZIPE">;
defm : VPatVUnzipIntrinsic<"int_riscv_vunzipo", "PseudoVUNZIPO">;
defm : VPatBinaryV_VV<"int_riscv_vpaire", "PseudoVPAIRE", AllVectors,
                      ExtraPreds = [HasStdExtZvzip],
                      requireMinimal = true>;
defm : VPatBinaryV_VV<"int_riscv_vpairo", "PseudoVPAIRO", AllVectors,
                      ExtraPreds = [HasStdExtZvzip],
                      requireMinimal = true>;
defm : VPatBinaryW_VV<"int_riscv_vzip", "PseudoVZIP", AllZvzipVectors,
                      ExtraPreds = [HasStdExtZvzip],
                      requireMinimal = true>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 142-158: TableGen record SDT_RISCVZip_VL / TableGen 记录 SDT_RISCVZip_VL
```tablegen
// (vd (op vs2, vs1, passthru, mask, vl))
def SDT_RISCVZip_VL : SDTypeProfile<1, 5, [SDTCisVec<0>, SDTCisVec<1>,
                                           SDTCisSubVecOfVec<1, 0>,
                                           SDTCisSameAs<1, 2>,
                                           SDTCisSameAs<0, 3>,
                                           SDTCVecEltisVT<4, i1>,
                                           SDTCisSameNumEltsAs<1, 4>,
                                           SDTCisVT<5, XLenVT>]>;

def SDT_RISCVVecBinOp_VL : SDTypeProfile<1, 5, [SDTCisSameAs<0, 1>,
                                                SDTCisSameAs<0, 2>,
                                                SDTCisVec<0>,
                                                SDTCisSameAs<0, 3>,
                                                SDTCVecEltisVT<4, i1>,
                                                SDTCisSameNumEltsAs<0, 4>,
                                                SDTCisVT<5, XLenVT>]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 159-175: Reusable TableGen multiclass vpaire_vl / 可复用的 TableGen 多类 vpaire_vl
```tablegen
let HasPassthruOp = true, HasMaskOp = true in {
def vpaire_vl  : RVSDNode<"VPAIRE_VL", SDT_RISCVVecBinOp_VL>;
def vpairo_vl  : RVSDNode<"VPAIRO_VL", SDT_RISCVVecBinOp_VL>;
def vzip_vl    : RVSDNode<"VZIP_VL", SDT_RISCVZip_VL>;
} // HasPassthruOp = true, HasMaskOp = true

multiclass VPatVZIP<SDPatternOperator vop, string instruction_name> {
  foreach VtiToWti = AllZvzipVectors in {
    defvar vti = VtiToWti.Vti;
    defvar wti = VtiToWti.Wti;
    let Predicates = !listconcat([HasStdExtZvzip],
                                 GetVTypeMinimalPredicates<vti>.Predicates) in {
      def : VPatBinaryVL_V<vop, instruction_name, "VV",
                           wti.Vector, vti.Vector, vti.Vector, vti.Mask,
                           vti.Log2SEW, vti.LMul, wti.RegClass, vti.RegClass,
                           vti.RegClass>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 176-185: Bulk record instantiation / 批量记录实例化
```tablegen
  }
}

defm : VPatVZIP<vzip_vl, "PseudoVZIP">;
defm : VPatBinaryVL_VV<vpaire_vl, "PseudoVPAIRE", AllVectors,
                       ExtraPreds = [HasStdExtZvzip],
                       requireMinimal = true>;
defm : VPatBinaryVL_VV<vpairo_vl, "PseudoVPAIRO", AllVectors,
                       ExtraPreds = [HasStdExtZvzip],
                       requireMinimal = true>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
