# RISCVInstrInfoZvfbf.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZvfbf.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZvfbf.td - 'Zvfbf*' instructions -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Zvfbfmin'
// extension, providing vector conversion instructions for BFloat16.
// This version is still experimental as the 'Zvfbfmin' extension hasn't been
// ratified yet.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-29: TableGen record VFWCVTBF16_F_F_V / TableGen 记录 VFWCVTBF16_F_F_V
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let ElementsDependOn = EltDepsNone in {

let Predicates = [HasStdExtZvfbfminOrZvfofp8min],
    Constraints = "@earlyclobber $vd",
    mayRaiseFPException = true in {
let VS1VS2Constraint = WidenCvt, DestEEW = EEWSEWx2 in
def VFWCVTBF16_F_F_V : VWCVTF_FV_VS2<"vfwcvtbf16.f.f.v", 0b010010, 0b01101>;
let Uses = [FRM, VL, VTYPE] in
def VFNCVTBF16_F_F_W : VNCVTF_FV_VS2<"vfncvtbf16.f.f.w", 0b010010, 0b11101>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 30-45: Bulk record instantiation VFWMACCBF16_V / 批量记录实例化 VFWMACCBF16_V
```tablegen

let Predicates = [HasStdExtZvfbfwma],
    Uses = [FRM, VL, VTYPE], mayRaiseFPException = true,
    DestEEW = EEWSEWx2 in {
defm VFWMACCBF16_V : VWMAC_FV_V_F<"vfwmaccbf16", 0b111011>;
}

} // ElementsDependOn = EltDepsNone

//===----------------------------------------------------------------------===//
// Pseudo instructions
//===----------------------------------------------------------------------===//
let Predicates = [HasStdExtZvfbfmin] in {
  defm PseudoVFWCVTBF16_F_F : VPseudoVWCVTD_V;
  defm PseudoVFNCVTBF16_F_F : VPseudoVNCVTD_W_RM;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 46-64: Reusable TableGen multiclass PseudoVFWMACCBF16 / 可复用的 TableGen 多类 PseudoVFWMACCBF16
```tablegen

let mayRaiseFPException = true, Predicates = [HasStdExtZvfbfwma] in
  defm PseudoVFWMACCBF16 : VPseudoVWMAC_VV_VF_BF_RM;

defset list<VTypeInfoToWide> AllWidenableIntToBF16Vectors = {
  def : VTypeInfoToWide<VI8MF8, VBF16MF4>;
  def : VTypeInfoToWide<VI8MF4, VBF16MF2>;
  def : VTypeInfoToWide<VI8MF2, VBF16M1>;
  def : VTypeInfoToWide<VI8M1, VBF16M2>;
  def : VTypeInfoToWide<VI8M2, VBF16M4>;
  def : VTypeInfoToWide<VI8M4, VBF16M8>;
}

multiclass VPseudoVALU_VV_VF_RM_BF16 {
  foreach m = MxListF in {
    defm "" : VPseudoBinaryFV_VV_RM<m, 16/*sew*/>,
              SchedBinary<"WriteVFALUV", "ReadVFALUV", "ReadVFALUV", m.MX,
                          16/*sew*/, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 65-80: Reusable TableGen multiclass m / 可复用的 TableGen 多类 m
```tablegen

  defvar f = SCALAR_F16;
  foreach m = f.MxList in {
    defm "" : VPseudoBinaryV_VF_RM<m, f, f.SEW>,
              SchedBinary<"WriteVFALUF", "ReadVFALUV", "ReadVFALUF", m.MX,
                          f.SEW, forcePassthruRead=true>;
  }
}

multiclass VPseudoVALU_VF_RM_BF16 {
  defvar f = SCALAR_F16;
  foreach m = f.MxList in {
    defm "" : VPseudoBinaryV_VF_RM<m, f, f.SEW>,
              SchedBinary<"WriteVFALUF", "ReadVFALUV", "ReadVFALUF", m.MX,
                          f.SEW, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 81-95: Reusable TableGen multiclass VPseudoVFWALU_VV_VF_RM_BF16 / 可复用的 TableGen 多类 VPseudoVFWALU_VV_VF_RM_BF16
```tablegen
}

multiclass VPseudoVFWALU_VV_VF_RM_BF16 {
  foreach m = MxListFW in {
    defm "" : VPseudoBinaryW_VV_RM<m, sew=16>,
              SchedBinary<"WriteVFWALUV", "ReadVFWALUV", "ReadVFWALUV", m.MX,
                          16/*sew*/, forcePassthruRead=true>;
  }

  defvar f = SCALAR_F16;
  foreach m = f.MxListFW in {
    defm "" : VPseudoBinaryW_VF_RM<m, f, sew=f.SEW>,
              SchedBinary<"WriteVFWALUF", "ReadVFWALUV", "ReadVFWALUF", m.MX,
                        f.SEW, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 96-109: Reusable TableGen multiclass VPseudoVFWALU_WV_WF_RM_BF16 / 可复用的 TableGen 多类 VPseudoVFWALU_WV_WF_RM_BF16
```tablegen
}

multiclass VPseudoVFWALU_WV_WF_RM_BF16 {
  foreach m = MxListFW in {
    defm "" : VPseudoBinaryW_WV_RM<m, sew=16>,
              SchedBinary<"WriteVFWALUV", "ReadVFWALUV", "ReadVFWALUV", m.MX,
                          16/*sew*/, forcePassthruRead=true>;
  }
  defvar f = SCALAR_F16;
  foreach m = f.MxListFW in {
    defm "" : VPseudoBinaryW_WF_RM<m, f, sew=f.SEW>,
              SchedBinary<"WriteVFWALUF", "ReadVFWALUV", "ReadVFWALUF", m.MX,
                          f.SEW, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 110-124: Reusable TableGen multiclass VPseudoVFMUL_VV_VF_RM_BF16 / 可复用的 TableGen 多类 VPseudoVFMUL_VV_VF_RM_BF16
```tablegen
}

multiclass VPseudoVFMUL_VV_VF_RM_BF16 {
  foreach m = MxListF in {
    defm "" : VPseudoBinaryFV_VV_RM<m, 16/*sew*/>,
              SchedBinary<"WriteVFMulV", "ReadVFMulV", "ReadVFMulV", m.MX,
                          16/*sew*/, forcePassthruRead=true>;
  }

  defvar f = SCALAR_F16;
  foreach m = f.MxList in {
    defm "" : VPseudoBinaryV_VF_RM<m, f, f.SEW>,
              SchedBinary<"WriteVFMulF", "ReadVFMulV", "ReadVFMulF", m.MX,
                          f.SEW, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 125-139: Reusable TableGen multiclass VPseudoVWMUL_VV_VF_RM_BF16 / 可复用的 TableGen 多类 VPseudoVWMUL_VV_VF_RM_BF16
```tablegen
}

multiclass VPseudoVWMUL_VV_VF_RM_BF16 {
  foreach m = MxListFW in {
    defm "" : VPseudoBinaryW_VV_RM<m, sew=16>,
              SchedBinary<"WriteVFWMulV", "ReadVFWMulV", "ReadVFWMulV", m.MX,
                          16/*sew*/, forcePassthruRead=true>;
  }

  defvar f = SCALAR_F16;
  foreach m = f.MxListFW in {
    defm "" : VPseudoBinaryW_VF_RM<m, f, sew=f.SEW>,
              SchedBinary<"WriteVFWMulF", "ReadVFWMulV", "ReadVFWMulF", m.MX,
                          f.SEW, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 140-154: Reusable TableGen multiclass VPseudoVMAC_VV_VF_AAXA_RM_BF16 / 可复用的 TableGen 多类 VPseudoVMAC_VV_VF_AAXA_RM_BF16
```tablegen
}

multiclass VPseudoVMAC_VV_VF_AAXA_RM_BF16 {
  foreach m = MxListF in {
    defm "" : VPseudoTernaryV_VV_AAXA_RM<m, 16/*sew*/>,
              SchedTernary<"WriteVFMulAddV", "ReadVFMulAddV", "ReadVFMulAddV", 
                           "ReadVFMulAddV", m.MX, 16/*sew*/>;
  }

  defvar f = SCALAR_F16;
  foreach m = f.MxList in {
    defm "" : VPseudoTernaryV_VF_AAXA_RM<m, f, f.SEW>,
              SchedTernary<"WriteVFMulAddF", "ReadVFMulAddV", "ReadVFMulAddF", 
                           "ReadVFMulAddV", m.MX, f.SEW>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 155-169: Reusable TableGen multiclass VPseudoVWMAC_VV_VF_RM_BF16 / 可复用的 TableGen 多类 VPseudoVWMAC_VV_VF_RM_BF16
```tablegen
}

multiclass VPseudoVWMAC_VV_VF_RM_BF16 {
  foreach m = MxListFW in {
    defm "" : VPseudoTernaryW_VV_RM<m, sew=16>,
              SchedTernary<"WriteVFWMulAddV", "ReadVFWMulAddV",
                           "ReadVFWMulAddV", "ReadVFWMulAddV", m.MX, 16/*sew*/>;
  }

  defvar f = SCALAR_F16;
  foreach m = f.MxListFW in {
    defm "" : VPseudoTernaryW_VF_RM<m, f, sew=f.SEW>,
              SchedTernary<"WriteVFWMulAddF", "ReadVFWMulAddV",
                           "ReadVFWMulAddF", "ReadVFWMulAddV", m.MX, f.SEW>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 170-185: Reusable TableGen multiclass VPseudoVRCP_V_BF16 / 可复用的 TableGen 多类 VPseudoVRCP_V_BF16
```tablegen
}

multiclass VPseudoVRCP_V_BF16 {
  foreach m = MxListF in {
    defvar mx = m.MX;
    let VLMul = m.value in {
      def "_V_" # mx # "_E16"
          : VPseudoUnaryNoMask<m.vrclass, m.vrclass>,
            SchedUnary<"WriteVFRecpV", "ReadVFRecpV", mx, 16/*sew*/,
                       forcePassthruRead=true>;
      def "_V_" # mx # "_E16_MASK"
          : VPseudoUnaryMask<m.vrclass, m.vrclass>,
            RISCVMaskedPseudo<MaskIdx = 2>,
            SchedUnary<"WriteVFRecpV", "ReadVFRecpV", mx, 16/*sew*/,
                       forcePassthruRead=true>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 186-202: Reusable TableGen multiclass VPseudoVRCP_V_RM_BF16 / 可复用的 TableGen 多类 VPseudoVRCP_V_RM_BF16
```tablegen
  }
}

multiclass VPseudoVRCP_V_RM_BF16 {
  foreach m = MxListF in {
    defvar mx = m.MX;
    let VLMul = m.value in {
      def "_V_" # mx # "_E16"
          : VPseudoUnaryNoMaskRoundingMode<m.vrclass, m.vrclass>,
            SchedUnary<"WriteVFRecpV", "ReadVFRecpV", mx, 16/*sew*/,
                       forcePassthruRead=true>;
      def "_V_" # mx # "_E16_MASK"
          : VPseudoUnaryMaskRoundingMode<m.vrclass, m.vrclass>,
            RISCVMaskedPseudo<MaskIdx = 2>,
            SchedUnary<"WriteVFRecpV", "ReadVFRecpV", mx, 16/*sew*/,
                       forcePassthruRead=true>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 203-218: Reusable TableGen multiclass VPseudoVMAX_VV_VF_BF16 / 可复用的 TableGen 多类 VPseudoVMAX_VV_VF_BF16
```tablegen
  }
}

multiclass VPseudoVMAX_VV_VF_BF16 {
  foreach m = MxListF in {
    defm "" : VPseudoBinaryV_VV<m, sew=16>,
              SchedBinary<"WriteVFMinMaxV", "ReadVFMinMaxV", "ReadVFMinMaxV", 
                          m.MX, 16/*sew*/, forcePassthruRead=true>;
  }

  defvar f = SCALAR_F16;
  foreach m = f.MxList in {
    defm "" : VPseudoBinaryV_VF<m, f, f.SEW>,
              SchedBinary<"WriteVFMinMaxF", "ReadVFMinMaxV", "ReadVFMinMaxF", 
                          m.MX, f.SEW, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 219-233: Reusable TableGen multiclass VPseudoVSGNJ_VV_VF_BF16 / 可复用的 TableGen 多类 VPseudoVSGNJ_VV_VF_BF16
```tablegen
}

multiclass VPseudoVSGNJ_VV_VF_BF16 {
  foreach m = MxListF in {
    defm "" : VPseudoBinaryV_VV<m, sew=16>,
              SchedBinary<"WriteVFSgnjV", "ReadVFSgnjV", "ReadVFSgnjV", m.MX,
                          16/*sew*/, forcePassthruRead=true>;
  }

  defvar f = SCALAR_F16;
  foreach m = f.MxList in {
    defm "" : VPseudoBinaryV_VF<m, f, f.SEW>,
              SchedBinary<"WriteVFSgnjF", "ReadVFSgnjV", "ReadVFSgnjF", m.MX,
                          f.SEW, forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 234-252: Reusable TableGen multiclass VPseudoVWCVTF_V_BF16 / 可复用的 TableGen 多类 VPseudoVWCVTF_V_BF16
```tablegen
}

multiclass VPseudoVWCVTF_V_BF16 {
  defvar constraint = "@earlyclobber $rd";
  foreach m = MxListW in
    defm _V : VPseudoConversion<m.wvrclass, m.vrclass, m, constraint, sew=8,
                                TargetConstraintType=3>,
              SchedUnary<"WriteVFWCvtIToFV", "ReadVFWCvtIToFV", m.MX, 8/*sew*/,
                         forcePassthruRead=true>;
}

multiclass VPseudoVWCVTD_V_BF16 {
  defvar constraint = "@earlyclobber $rd";
  foreach m = MxListFW in
    defm _V : VPseudoConversion<m.wvrclass, m.vrclass, m, constraint, sew=16,
                                TargetConstraintType=3>,
              SchedUnary<"WriteVFWCvtFToFV", "ReadVFWCvtFToFV", m.MX, 16/*sew*/,
                         forcePassthruRead=true>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 253-271: Reusable TableGen multiclass VPseudoVNCVTD_W_BF16 / 可复用的 TableGen 多类 VPseudoVNCVTD_W_BF16
```tablegen

multiclass VPseudoVNCVTD_W_BF16 {
  defvar constraint = "@earlyclobber $rd";
  foreach m = MxListFW in
    defm _W : VPseudoConversion<m.vrclass, m.wvrclass, m, constraint, sew=16,
                                TargetConstraintType=2>,
              SchedUnary<"WriteVFNCvtFToFV", "ReadVFNCvtFToFV", m.MX, 16/*sew*/,
                         forcePassthruRead=true>;
}

multiclass VPseudoVNCVTD_W_RM_BF16 {
  defvar constraint = "@earlyclobber $rd";
  foreach m = MxListFW in
    defm _W : VPseudoConversionRoundingMode<m.vrclass, m.wvrclass, m,
                                            constraint, sew=16,
                                            TargetConstraintType=2>,
              SchedUnary<"WriteVFNCvtFToFV", "ReadVFNCvtFToFV", m.MX, 16/*sew*/,
                         forcePassthruRead=true>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 272-285: Bulk record instantiation PseudoVFADD_ALT / 批量记录实例化 PseudoVFADD_ALT
```tablegen

let Predicates = [HasVInstructionsBF16], AltFmtType = IS_ALTFMT in {
let mayRaiseFPException = true in {
defm PseudoVFADD_ALT : VPseudoVALU_VV_VF_RM_BF16;
defm PseudoVFSUB_ALT  : VPseudoVALU_VV_VF_RM_BF16;
defm PseudoVFRSUB_ALT : VPseudoVALU_VF_RM_BF16;
}

let mayRaiseFPException = true in {
defm PseudoVFWADD_ALT : VPseudoVFWALU_VV_VF_RM_BF16;
defm PseudoVFWSUB_ALT : VPseudoVFWALU_VV_VF_RM_BF16;
defm PseudoVFWADD_ALT : VPseudoVFWALU_WV_WF_RM_BF16;
defm PseudoVFWSUB_ALT : VPseudoVFWALU_WV_WF_RM_BF16;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 286-302: Bulk record instantiation PseudoVFMUL_ALT / 批量记录实例化 PseudoVFMUL_ALT
```tablegen

let mayRaiseFPException = true in
defm PseudoVFMUL_ALT : VPseudoVFMUL_VV_VF_RM_BF16;

let mayRaiseFPException = true in
defm PseudoVFWMUL_ALT : VPseudoVWMUL_VV_VF_RM_BF16;

let mayRaiseFPException = true in {
defm PseudoVFMACC_ALT  : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
defm PseudoVFNMACC_ALT : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
defm PseudoVFMSAC_ALT  : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
defm PseudoVFNMSAC_ALT : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
defm PseudoVFMADD_ALT  : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
defm PseudoVFNMADD_ALT : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
defm PseudoVFMSUB_ALT  : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
defm PseudoVFNMSUB_ALT : VPseudoVMAC_VV_VF_AAXA_RM_BF16;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 303-316: Bulk record instantiation PseudoVFWMACC_ALT / 批量记录实例化 PseudoVFWMACC_ALT
```tablegen

let mayRaiseFPException = true in {
defm PseudoVFWMACC_ALT  : VPseudoVWMAC_VV_VF_RM_BF16;
defm PseudoVFWNMACC_ALT : VPseudoVWMAC_VV_VF_RM_BF16;
defm PseudoVFWMSAC_ALT  : VPseudoVWMAC_VV_VF_RM_BF16;
defm PseudoVFWNMSAC_ALT : VPseudoVWMAC_VV_VF_RM_BF16;
}

let mayRaiseFPException = true in
defm PseudoVFRSQRT7_ALT : VPseudoVRCP_V_BF16;

let mayRaiseFPException = true in
defm PseudoVFREC7_ALT : VPseudoVRCP_V_RM_BF16;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 317-333: Bulk record instantiation PseudoVFMIN_ALT / 批量记录实例化 PseudoVFMIN_ALT
```tablegen
let mayRaiseFPException = true in {
defm PseudoVFMIN_ALT : VPseudoVMAX_VV_VF_BF16;
defm PseudoVFMAX_ALT : VPseudoVMAX_VV_VF_BF16;
}

defm PseudoVFSGNJ_ALT  : VPseudoVSGNJ_VV_VF_BF16;
defm PseudoVFSGNJN_ALT : VPseudoVSGNJ_VV_VF_BF16;
defm PseudoVFSGNJX_ALT : VPseudoVSGNJ_VV_VF_BF16;

let mayRaiseFPException = true in {
defm PseudoVMFEQ_ALT : VPseudoVCMPM_VV_VF;
defm PseudoVMFNE_ALT : VPseudoVCMPM_VV_VF;
defm PseudoVMFLT_ALT : VPseudoVCMPM_VV_VF;
defm PseudoVMFLE_ALT : VPseudoVCMPM_VV_VF;
defm PseudoVMFGT_ALT : VPseudoVCMPM_VF;
defm PseudoVMFGE_ALT : VPseudoVCMPM_VF;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 334-347: Bulk record instantiation PseudoVFCLASS_ALT / 批量记录实例化 PseudoVFCLASS_ALT
```tablegen

defm PseudoVFCLASS_ALT : VPseudoVCLS_V;

defm PseudoVFMERGE_ALT : VPseudoVMRG_FM;

defm PseudoVFMV_V_ALT : VPseudoVMV_F;

let mayRaiseFPException = true in {
defm PseudoVFWCVT_F_XU_ALT : VPseudoVWCVTF_V_BF16;
defm PseudoVFWCVT_F_X_ALT  : VPseudoVWCVTF_V_BF16;

defm PseudoVFWCVT_F_F_ALT  : VPseudoVWCVTD_V_BF16;
} // mayRaiseFPException = true
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 348-361: Bulk record instantiation PseudoVFNCVT_XU_F_ALT / 批量记录实例化 PseudoVFNCVT_XU_F_ALT
```tablegen
let mayRaiseFPException = true in {
let hasSideEffects = 0, hasPostISelHook = 1 in {
defm PseudoVFNCVT_XU_F_ALT : VPseudoVNCVTI_W_RM;
defm PseudoVFNCVT_X_F_ALT  : VPseudoVNCVTI_W_RM;
}

defm PseudoVFNCVT_RTZ_XU_F_ALT : VPseudoVNCVTI_W;
defm PseudoVFNCVT_RTZ_X_F_ALT  : VPseudoVNCVTI_W;

defm PseudoVFNCVT_F_F_ALT  : VPseudoVNCVTD_W_RM_BF16;

defm PseudoVFNCVT_ROD_F_F_ALT : VPseudoVNCVTD_W_BF16;
} // mayRaiseFPException = true
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 362-375: TableGen record definition / TableGen 记录定义
```tablegen
let mayLoad = 0, mayStore = 0, hasSideEffects = 0 in {
  defvar f = SCALAR_F16;
  let HasSEWOp = 1, BaseInstr = VFMV_F_S in
  def "PseudoVFMV_" # f.FX # "_S_ALT" :
    RISCVVPseudo<(outs f.fprclass:$rd), (ins VR:$rs2, sew:$sew)>,
    Sched<[WriteVMovFS, ReadVMovFS]>;
  let HasVLOp = 1, HasSEWOp = 1, BaseInstr = VFMV_S_F, isReMaterializable = 1,
      Constraints = "$rd = $passthru" in
  def "PseudoVFMV_S_" # f.FX # "_ALT" :
    RISCVVPseudo<(outs VR:$rd),
                 (ins VR:$passthru, f.fprclass:$rs1, AVL:$vl, sew:$sew)>,
    Sched<[WriteVMovSF, ReadVMovSF_V, ReadVMovSF_F]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 376-392: Reusable TableGen multiclass PseudoVFSLIDE1UP_ALT / 可复用的 TableGen 多类 PseudoVFSLIDE1UP_ALT
```tablegen
defm PseudoVFSLIDE1UP_ALT   : VPseudoVSLD1_VF<"@earlyclobber $rd">;
defm PseudoVFSLIDE1DOWN_ALT : VPseudoVSLD1_VF;
} // Predicates = [HasVInstructionsBF16], AltFmtType = IS_ALTFMT

//===----------------------------------------------------------------------===//
// Patterns
//===----------------------------------------------------------------------===//
multiclass VPatConversionWF_VF_BF<string intrinsic, string instruction,
                                  bit isSEWAware = 0> {
  foreach fvtiToFWti = AllWidenableBF16ToFloatVectors in
  {
    defvar fvti = fvtiToFWti.Vti;
    defvar fwti = fvtiToFWti.Wti;
    defm : VPatConversion<intrinsic, instruction, "V",
                          fwti.Vector, fvti.Vector, fwti.Mask, fvti.Log2SEW,
                          fvti.LMul, fwti.RegClass, fvti.RegClass, isSEWAware>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 393-406: Reusable TableGen multiclass VPatConversionVF_WF_BF_RM / 可复用的 TableGen 多类 VPatConversionVF_WF_BF_RM
```tablegen
}

multiclass VPatConversionVF_WF_BF_RM<string intrinsic, string instruction,
                                     bit isSEWAware = 0> {
  foreach fvtiToFWti = AllWidenableBF16ToFloatVectors in {
    defvar fvti = fvtiToFWti.Vti;
    defvar fwti = fvtiToFWti.Wti;
    defm : VPatConversionRoundingMode<intrinsic, instruction, "W",
                                      fvti.Vector, fwti.Vector, fvti.Mask, fvti.Log2SEW,
                                      fvti.LMul, fvti.RegClass, fwti.RegClass,
                                      isSEWAware>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 407-427: Reusable TableGen multiclass VPatConversionVI_VF_BF16 / 可复用的 TableGen 多类 VPatConversionVI_VF_BF16
```tablegen
let Predicates = [HasStdExtZvfbfmin] in {
  defm : VPatConversionWF_VF_BF<"int_riscv_vfwcvtbf16_f_f_v",
                                "PseudoVFWCVTBF16_F_F", isSEWAware=1>;
  defm : VPatConversionVF_WF_BF_RM<"int_riscv_vfncvtbf16_f_f_w",
                                   "PseudoVFNCVTBF16_F_F", isSEWAware=1>;
}

let Predicates = [HasStdExtZvfbfwma] in {
  defm : VPatTernaryW_VV_VX_RM<"int_riscv_vfwmaccbf16", "PseudoVFWMACCBF16",
                               AllWidenableBF16ToFloatVectors, isSEWAware=1>;
}

multiclass VPatConversionVI_VF_BF16<string intrinsic, string instruction> {
  foreach fvti = AllBF16Vectors in {
    defvar ivti = GetIntVTypeInfo<fvti>.Vti;
    let Predicates = !listconcat(GetVTypePredicates<fvti>.Predicates,
                                 GetVTypePredicates<ivti>.Predicates) in
    defm : VPatConversion<intrinsic, instruction, "V",
                          ivti.Vector, fvti.Vector, ivti.Mask, fvti.Log2SEW,
                          fvti.LMul, ivti.RegClass, fvti.RegClass>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 428-441: Reusable TableGen multiclass VPatConversionWF_VI_BF16 / 可复用的 TableGen 多类 VPatConversionWF_VI_BF16
```tablegen
}

multiclass VPatConversionWF_VI_BF16<string intrinsic, string instruction,
                                    bit isSEWAware = 0> {
  foreach vtiToWti = AllWidenableIntToBF16Vectors in {
    defvar vti = vtiToWti.Vti;
    defvar fwti = vtiToWti.Wti;
    let Predicates = !listconcat(GetVTypePredicates<vti>.Predicates,
                                 GetVTypePredicates<fwti>.Predicates) in
    defm : VPatConversion<intrinsic, instruction, "V",
                          fwti.Vector, vti.Vector, fwti.Mask, vti.Log2SEW,
                          vti.LMul, fwti.RegClass, vti.RegClass, isSEWAware>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 442-455: Reusable TableGen multiclass VPatConversionWF_VF_BF16 / 可复用的 TableGen 多类 VPatConversionWF_VF_BF16
```tablegen

multiclass VPatConversionWF_VF_BF16<string intrinsic, string instruction,
                                    bit isSEWAware = 0> {
  foreach fvtiToFWti = AllWidenableBF16ToFloatVectors in {
    defvar fvti = fvtiToFWti.Vti;
    defvar fwti = fvtiToFWti.Wti;
    let Predicates = !listconcat(GetVTypeMinimalPredicates<fvti>.Predicates,
                                 GetVTypeMinimalPredicates<fwti>.Predicates) in
    defm : VPatConversion<intrinsic, instruction, "V",
                          fwti.Vector, fvti.Vector, fwti.Mask, fvti.Log2SEW,
                          fvti.LMul, fwti.RegClass, fvti.RegClass, isSEWAware>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 456-477: Reusable TableGen multiclass VPatConversionVI_WF_BF16 / 可复用的 TableGen 多类 VPatConversionVI_WF_BF16
```tablegen
multiclass VPatConversionVI_WF_BF16<string intrinsic, string instruction> {
  foreach vtiToWti = AllWidenableIntToBF16Vectors in {
    defvar vti = vtiToWti.Vti;
    defvar fwti = vtiToWti.Wti;
    let Predicates = !listconcat(GetVTypePredicates<vti>.Predicates,
                                 GetVTypePredicates<fwti>.Predicates) in
    defm : VPatConversion<intrinsic, instruction, "W",
                          vti.Vector, fwti.Vector, vti.Mask, vti.Log2SEW,
                          vti.LMul, vti.RegClass, fwti.RegClass>;
  }
}

multiclass VPatConversionVI_WF_RM_BF16<string intrinsic, string instruction> {
  foreach vtiToWti = AllWidenableIntToBF16Vectors in {
    defvar vti = vtiToWti.Vti;
    defvar fwti = vtiToWti.Wti;
    let Predicates = !listconcat(GetVTypePredicates<vti>.Predicates,
                                 GetVTypePredicates<fwti>.Predicates) in
    defm : VPatConversionRoundingMode<intrinsic, instruction, "W",
                                      vti.Vector, fwti.Vector, vti.Mask, vti.Log2SEW,
                                      vti.LMul, vti.RegClass, fwti.RegClass>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 478-491: Reusable TableGen multiclass VPatConversionVF_WF_BF16 / 可复用的 TableGen 多类 VPatConversionVF_WF_BF16
```tablegen
}

multiclass VPatConversionVF_WF_BF16<string intrinsic, string instruction,
                                    bit isSEWAware = 0> {
  foreach fvtiToFWti = AllWidenableBF16ToFloatVectors in {
    defvar fvti = fvtiToFWti.Vti;
    defvar fwti = fvtiToFWti.Wti;
    let Predicates = !listconcat(GetVTypePredicates<fvti>.Predicates,
                                 GetVTypePredicates<fwti>.Predicates) in
    defm : VPatConversion<intrinsic, instruction, "W",
                          fvti.Vector, fwti.Vector, fvti.Mask, fvti.Log2SEW,
                          fvti.LMul, fvti.RegClass, fwti.RegClass, isSEWAware>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 492-519: Bulk record instantiation / 批量记录实例化
```tablegen

let Predicates = [HasVInstructionsBF16] in {
defm : VPatBinaryV_VV_VX_RM<"int_riscv_vfadd", "PseudoVFADD_ALT",
                            AllBF16Vectors, isSEWAware = 1>;
defm : VPatBinaryV_VV_VX_RM<"int_riscv_vfsub", "PseudoVFSUB_ALT",
                            AllBF16Vectors, isSEWAware = 1>;
defm : VPatBinaryV_VX_RM<"int_riscv_vfrsub", "PseudoVFRSUB_ALT",
                         AllBF16Vectors, isSEWAware = 1>;
defm : VPatBinaryW_VV_VX_RM<"int_riscv_vfwadd", "PseudoVFWADD_ALT",
                            AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatBinaryW_VV_VX_RM<"int_riscv_vfwsub", "PseudoVFWSUB_ALT",
                            AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatBinaryW_WV_WX_RM<"int_riscv_vfwadd_w", "PseudoVFWADD_ALT",
                            AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatBinaryW_WV_WX_RM<"int_riscv_vfwsub_w", "PseudoVFWSUB_ALT",
                            AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatBinaryV_VV_VX_RM<"int_riscv_vfmul", "PseudoVFMUL_ALT",
                            AllBF16Vectors, isSEWAware=1>;
defm : VPatBinaryW_VV_VX_RM<"int_riscv_vfwmul", "PseudoVFWMUL_ALT",
                            AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfmacc", "PseudoVFMACC_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfnmacc", "PseudoVFNMACC_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfmsac", "PseudoVFMSAC_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfnmsac", "PseudoVFNMSAC_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 520-547: Bulk record instantiation / 批量记录实例化
```tablegen
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfmadd", "PseudoVFMADD_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfnmadd", "PseudoVFNMADD_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfmsub", "PseudoVFMSUB_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
defm : VPatTernaryV_VV_VX_AAXA_RM<"int_riscv_vfnmsub", "PseudoVFNMSUB_ALT",
                                  AllBF16Vectors, isSEWAware=1>;
defm : VPatTernaryW_VV_VX_RM<"int_riscv_vfwmacc", "PseudoVFWMACC_ALT",
                             AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatTernaryW_VV_VX_RM<"int_riscv_vfwnmacc", "PseudoVFWNMACC_ALT",
                             AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatTernaryW_VV_VX_RM<"int_riscv_vfwmsac", "PseudoVFWMSAC_ALT",
                             AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatTernaryW_VV_VX_RM<"int_riscv_vfwnmsac", "PseudoVFWNMSAC_ALT",
                             AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatUnaryV_V<"int_riscv_vfrsqrt7", "PseudoVFRSQRT7_ALT",
                    AllBF16Vectors, isSEWAware=1>;
defm : VPatUnaryV_V_RM<"int_riscv_vfrec7", "PseudoVFREC7_ALT",
                       AllBF16Vectors, isSEWAware=1>;
defm : VPatBinaryV_VV_VX<"int_riscv_vfmin", "PseudoVFMIN_ALT",
                         AllBF16Vectors, isSEWAware=1>;
defm : VPatBinaryV_VV_VX<"int_riscv_vfmax", "PseudoVFMAX_ALT",
                         AllBF16Vectors, isSEWAware=1>;
defm : VPatBinaryV_VV_VX<"int_riscv_vfsgnj", "PseudoVFSGNJ_ALT",
                         AllBF16Vectors, isSEWAware=1>;
defm : VPatBinaryV_VV_VX<"int_riscv_vfsgnjn", "PseudoVFSGNJN_ALT",
                         AllBF16Vectors, isSEWAware=1>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 548-567: Bulk record instantiation vti / 批量记录实例化 vti
```tablegen
defm : VPatBinaryV_VV_VX<"int_riscv_vfsgnjx", "PseudoVFSGNJX_ALT",
                         AllBF16Vectors, isSEWAware=1>;
defm : VPatBinaryM_VV_VX<"int_riscv_vmfeq", "PseudoVMFEQ_ALT", AllBF16Vectors>;
defm : VPatBinaryM_VV_VX<"int_riscv_vmfle", "PseudoVMFLE_ALT", AllBF16Vectors>;
defm : VPatBinaryM_VV_VX<"int_riscv_vmflt", "PseudoVMFLT_ALT", AllBF16Vectors>;
defm : VPatBinaryM_VV_VX<"int_riscv_vmfne", "PseudoVMFNE_ALT", AllBF16Vectors>;
defm : VPatBinaryM_VX<"int_riscv_vmfgt", "PseudoVMFGT_ALT", AllBF16Vectors>;
defm : VPatBinaryM_VX<"int_riscv_vmfge", "PseudoVMFGE_ALT", AllBF16Vectors>;
defm : VPatBinarySwappedM_VV<"int_riscv_vmfgt", "PseudoVMFLT_ALT", AllBF16Vectors>;
defm : VPatBinarySwappedM_VV<"int_riscv_vmfge", "PseudoVMFLE_ALT", AllBF16Vectors>;
defm : VPatConversionVI_VF_BF16<"int_riscv_vfclass", "PseudoVFCLASS_ALT">;
foreach vti = AllBF16Vectors in {
  let Predicates = GetVTypePredicates<vti>.Predicates in
    defm : VPatBinaryCarryInTAIL<"int_riscv_vfmerge", "PseudoVFMERGE_ALT",
                                 "V"#vti.ScalarSuffix#"M",
                                 vti.Vector,
                                 vti.Vector, vti.Scalar, vti.Mask,
                                 vti.Log2SEW, vti.LMul, vti.RegClass,
                                 vti.RegClass, vti.ScalarRegClass>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 568-584: Bulk record instantiation / 批量记录实例化
```tablegen
defm : VPatConversionWF_VI_BF16<"int_riscv_vfwcvt_f_xu_v", "PseudoVFWCVT_F_XU_ALT",
                                isSEWAware=1>;
defm : VPatConversionWF_VI_BF16<"int_riscv_vfwcvt_f_x_v", "PseudoVFWCVT_F_X_ALT",
                                isSEWAware=1>;
defm : VPatConversionWF_VF_BF16<"int_riscv_vfwcvt_f_f_v", "PseudoVFWCVT_F_F_ALT",
                                isSEWAware=1>;
defm : VPatConversionVI_WF_RM_BF16<"int_riscv_vfncvt_xu_f_w", "PseudoVFNCVT_XU_F_ALT">;
defm : VPatConversionVI_WF_RM_BF16<"int_riscv_vfncvt_x_f_w", "PseudoVFNCVT_X_F_ALT">;
defm : VPatConversionVI_WF_BF16<"int_riscv_vfncvt_rtz_xu_f_w", "PseudoVFNCVT_RTZ_XU_F_ALT">;
defm : VPatConversionVI_WF_BF16<"int_riscv_vfncvt_rtz_x_f_w", "PseudoVFNCVT_RTZ_X_F_ALT">;
defm : VPatConversionVF_WF_RM<"int_riscv_vfncvt_f_f_w", "PseudoVFNCVT_F_F_ALT",
                              AllWidenableBF16ToFloatVectors, isSEWAware=1>;
defm : VPatConversionVF_WF_BF16<"int_riscv_vfncvt_rod_f_f_w", "PseudoVFNCVT_ROD_F_F_ALT",
                                isSEWAware=1>;
defm : VPatBinaryV_VX<"int_riscv_vfslide1up", "PseudoVFSLIDE1UP_ALT", AllBF16Vectors>;
defm : VPatBinaryV_VX<"int_riscv_vfslide1down", "PseudoVFSLIDE1DOWN_ALT", AllBF16Vectors>;
} // Predicates = [HasVInstructionsBF16]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
