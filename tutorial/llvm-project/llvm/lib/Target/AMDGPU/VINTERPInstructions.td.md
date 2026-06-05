# VINTERPInstructions.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/VINTERPInstructions.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines VINTERPInstructions records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 VINTERPInstructions 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: File banner, comments, and TableGen overview
```tablegen
//===-- VINTERPInstructions.td - VINTERP Instruction Definitions ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// VINTERP encoding
//===----------------------------------------------------------------------===//

class VINTERPe : Enc64 {
  bits<11> vdst;
  bits<4> src0_modifiers;
  bits<11> src0;
  bits<3> src1_modifiers;
  bits<11> src1;
  bits<3> src2_modifiers;
  bits<11> src2;
  bits<1> clamp;
  bits<3> waitexp;

  let Inst{31-26} = 0x33; // VOP3P encoding
  let Inst{25-24} = 0x1; // VINTERP sub-encoding

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `VINTERPe`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`VINTERPe`。

### Lines 27-59: Defines TableGen class VINTERPe_gfx11
```tablegen
  let Inst{7-0}   = vdst{7-0};
  let Inst{10-8}  = waitexp;
  // Fields for hi/lo 16-bits of register selection
  let Inst{11}    = src0_modifiers{2};
  let Inst{12}    = src1_modifiers{2};
  let Inst{13}    = src2_modifiers{2};
  let Inst{14}    = src0_modifiers{3};
  let Inst{15}    = clamp;
  let Inst{40-32} = src0{8-0};
  let Inst{49-41} = src1{8-0};
  let Inst{58-50} = src2{8-0};
  let Inst{61}    = src0_modifiers{0}; // neg(0)
  let Inst{62}    = src1_modifiers{0}; // neg(1)
  let Inst{63}    = src2_modifiers{0}; // neg(2)
}

class VINTERPe_gfx11 <bits<7> op> : VINTERPe {
  let Inst{22-16} = op;
}

class VINTERPe_gfx12_gfx13 <bits<7> op> : VINTERPe {
  let Inst{20-16} = op{4-0};
}

//===----------------------------------------------------------------------===//
// VOP3 VINTERP
//===----------------------------------------------------------------------===//

class VINTERP_Pseudo <string OpName, VOPProfile P, list<dag> pattern = []> :
  VOP3_Pseudo<OpName, P, pattern, 0, 0> {
  let AsmMatchConverter = "cvtVINTERP";
  let mayRaiseFPException = 0;

```
**EN:** This section contains concrete logic for TableGen class VINTERPe_gfx11. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VINTERPe_gfx11`, `VINTERPe_gfx12_gfx13`, `VINTERP_Pseudo`.
**CN:** 本节包含与 TableGen class VINTERPe_gfx11 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VINTERPe_gfx11`, `VINTERPe_gfx12_gfx13`, `VINTERP_Pseudo`。

### Lines 60-87: Defines TableGen class VINTERP_Real
```tablegen
  let VOP3_OPSEL = 1;
  let VINTERP = 1;
}

class VINTERP_Real <VOP_Pseudo ps, int EncodingFamily, string asmName> :
  VOP3_Real <ps, EncodingFamily, asmName> {
  let VINTERP = 1;
  let IsSingle = 1;
}

def VOP3_VINTERP_F32 : VOPProfile<[f32, f32, f32, f32]> {
  let HasOpSel = 0;
  let HasModifiers = 1;

  let Src0Mod = FPVRegInputMods;
  let Src1Mod = FPVRegInputMods;
  let Src2Mod = FPVRegInputMods;

  let Outs64 = (outs VGPR_32:$vdst);
  let Ins64 = (ins Src0Mod:$src0_modifiers, VRegSrc_32:$src0,
                   Src1Mod:$src1_modifiers, VRegSrc_32:$src1,
                   Src2Mod:$src2_modifiers, VRegSrc_32:$src2,
                   Clamp:$clamp,
                   WaitEXP:$waitexp);

  let Asm64 = " $vdst, $src0_modifiers, $src1_modifiers, $src2_modifiers$clamp$waitexp";
}

```
**EN:** This section contains concrete logic for TableGen class VINTERP_Real. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VINTERP_Real`, `VOP3_VINTERP_F32`.
**CN:** 本节包含与 TableGen class VINTERP_Real 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VINTERP_Real`, `VOP3_VINTERP_F32`。

### Lines 88-118: Defines TableGen class VOP3_VINTERP_F16_t16
```tablegen
class VOP3_VINTERP_F16_t16 <list<ValueType> ArgVT> : VOPProfile_True16<VOPProfile<ArgVT>> {
  let Src0Mod = FPT16VRegInputMods</*Fake16*/0>;
  let Src1Mod = FPVRegInputMods;
  let Src2Mod = !if(!eq(ArgVT[3].Size, 16), FPT16VRegInputMods</*Fake16*/0>,
                                            FPVRegInputMods);
  let Ins64 = (ins Src0Mod:$src0_modifiers, VRegSrc_16:$src0,
                   Src1Mod:$src1_modifiers, VRegSrc_32:$src1,
                   Src2Mod:$src2_modifiers,
                   !if(!eq(ArgVT[3].Size, 16), VRegSrc_16, VRegSrc_32):$src2,
                   Clamp:$clamp, op_sel0:$op_sel,
                   WaitEXP:$waitexp);

  let Asm64 = "$vdst, $src0_modifiers, $src1_modifiers, $src2_modifiers$clamp$op_sel$waitexp";
}

class VOP3_VINTERP_F16_fake16 <list<ValueType> ArgVT> : VOPProfile_Fake16<VOPProfile<ArgVT>> {
  let Src0Mod = FPT16VRegInputMods</*Fake16*/1>;
  let Src1Mod = FPVRegInputMods;
  let Src2Mod = !if(!eq(ArgVT[3].Size, 16), FPT16VRegInputMods</*Fake16*/1>,
                                            FPVRegInputMods);

  let Ins64 = (ins Src0Mod:$src0_modifiers, VRegSrc_fake16:$src0,
                   Src1Mod:$src1_modifiers, VRegSrc_32:$src1,
                   Src2Mod:$src2_modifiers,
                   !if(!eq(ArgVT[3].Size, 16), VRegSrc_fake16, VRegSrc_32):$src2,
                   Clamp:$clamp, op_sel0:$op_sel,
                   WaitEXP:$waitexp);

  let Asm64 = "$vdst, $src0_modifiers, $src1_modifiers, $src2_modifiers$clamp$op_sel$waitexp";
}

```
**EN:** This section contains concrete logic for TableGen class VOP3_VINTERP_F16_t16. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VOP3_VINTERP_F16_t16`, `VOP3_VINTERP_F16_fake16`.
**CN:** 本节包含与 TableGen class VOP3_VINTERP_F16_t16 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VOP3_VINTERP_F16_t16`, `VOP3_VINTERP_F16_fake16`。

### Lines 119-148: Defines TableGen multiclass VINTERP_t16
```tablegen
//===----------------------------------------------------------------------===//
// VINTERP Pseudo Instructions
//===----------------------------------------------------------------------===//

let SubtargetPredicate = HasVINTERPEncoding in {

multiclass VINTERP_t16<string OpName, list<ValueType> ArgVT> {
  let True16Predicate = UseRealTrue16Insts in {
    def _t16 : VINTERP_Pseudo<OpName#"_t16", VOP3_VINTERP_F16_t16<ArgVT>> ;
  }
  let True16Predicate = UseFakeTrue16Insts in {
    def _fake16 : VINTERP_Pseudo<OpName#"_fake16", VOP3_VINTERP_F16_fake16<ArgVT>> ;
  }
}

let Uses = [M0, EXEC, MODE] in {
def V_INTERP_P10_F32_inreg : VINTERP_Pseudo <"v_interp_p10_f32", VOP3_VINTERP_F32>;
def V_INTERP_P2_F32_inreg : VINTERP_Pseudo <"v_interp_p2_f32", VOP3_VINTERP_F32>;

defm V_INTERP_P10_F16_F32_inreg : VINTERP_t16<"v_interp_p10_f16_f32", [f32, f16, f32, f16]>;
defm V_INTERP_P2_F16_F32_inreg : VINTERP_t16<"v_interp_p2_f16_f32", [f16, f16, f32, f32]>;
} // Uses = [M0, EXEC, MODE]

let Uses = [M0, EXEC] in {
defm V_INTERP_P10_RTZ_F16_F32_inreg : VINTERP_t16<"v_interp_p10_rtz_f16_f32", [f32, f16, f32, f16]>;
defm V_INTERP_P2_RTZ_F16_F32_inreg : VINTERP_t16 <"v_interp_p2_rtz_f16_f32", [f16, f16, f32, f32]>;
} // Uses = [M0, EXEC]

} // SubtargetPredicate = HasVINTERPEncoding.

```
**EN:** This section contains concrete logic for TableGen multiclass VINTERP_t16. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VINTERP_t16`, `_t16`, `_fake16`.
**CN:** 本节包含与 TableGen multiclass VINTERP_t16 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VINTERP_t16`, `_t16`, `_fake16`。

### Lines 149-176: Defines TableGen class VInterpF32Pat
```tablegen
class VInterpF32Pat <SDPatternOperator op, Instruction inst> : GCNPat <
   (f32 (op
      (VINTERPMods f32:$src0, i32:$src0_modifiers),
      (VINTERPMods f32:$src1, i32:$src1_modifiers),
      (VINTERPMods f32:$src2, i32:$src2_modifiers))),
    (inst $src0_modifiers, $src0,
          $src1_modifiers, $src1,
          $src2_modifiers, $src2,
          0, /* clamp */
          7) /* wait_exp */
>;

class VInterpF16Pat <SDPatternOperator op, Instruction inst,
                     ValueType dst_type, bit high,
                     list<ComplexPattern> pat> : GCNPat <
   (dst_type (op
      (pat[0] f32:$src0, i32:$src0_modifiers),
      (pat[1] f32:$src1, i32:$src1_modifiers),
      (pat[2] f32:$src2, i32:$src2_modifiers),
      !if(high, (i1 -1), (i1 0)))),
    (inst $src0_modifiers, $src0,
          $src1_modifiers, $src1,
          $src2_modifiers, $src2,
          0, /* clamp */
          /* op_sel = 0 */
          7) /* wait_exp */
>;

```
**EN:** This section contains concrete logic for TableGen class VInterpF32Pat. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VInterpF32Pat`, `VInterpF16Pat`.
**CN:** 本节包含与 TableGen class VInterpF32Pat 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VInterpF32Pat`, `VInterpF16Pat`。

### Lines 177-209: Defines TableGen multiclass VInterpF16Pat
```tablegen
multiclass VInterpF16Pat <SDPatternOperator op, Instruction inst,
                          ValueType dst_type, list<ComplexPattern> high_pat> {
  def : VInterpF16Pat<op, inst, dst_type, 0,
                      [VINTERPMods, VINTERPMods, VINTERPMods]>;
  def : VInterpF16Pat<op, inst, dst_type, 1, high_pat>;
}

class VInterpF16Pat_t16 <SDPatternOperator op, Instruction inst,
                     ValueType dstVT, bit high, bit isP2> : GCNPat <
   (dstVT (op
      (VINTERPMods f32:$src0, i32:$src0_modifiers),
      (VINTERPMods f32:$src1, i32:$src1_modifiers),
      (VINTERPMods f32:$src2, i32:$src2_modifiers),
      !if(high, (i1 -1), (i1 0)))),
    (inst $src0_modifiers,
          (f16 (EXTRACT_SUBREG VGPR_32:$src0, !if(high, hi16, lo16))),
          $src1_modifiers, VGPR_32:$src1,
          $src2_modifiers,
          !if(isP2, (f32 VGPR_32:$src2),
                    (f16 (EXTRACT_SUBREG VGPR_32:$src2, !if(high, hi16, lo16)))),
          0, /* clamp */
          7) /* wait_exp */
>;

multiclass VInterpF16Pat_t16 <SDPatternOperator op, Instruction inst,
                          ValueType dstVT, bit isP2> {
  def : VInterpF16Pat_t16<op, inst, dstVT, 0, isP2>;
  def : VInterpF16Pat_t16<op, inst, dstVT, 1, isP2>;
}

def : VInterpF32Pat<int_amdgcn_interp_inreg_p10, V_INTERP_P10_F32_inreg>;
def : VInterpF32Pat<int_amdgcn_interp_inreg_p2, V_INTERP_P2_F32_inreg>;

```
**EN:** This section contains concrete logic for TableGen multiclass VInterpF16Pat. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VInterpF16Pat`, `VInterpF16Pat_t16`.
**CN:** 本节包含与 TableGen multiclass VInterpF16Pat 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VInterpF16Pat`, `VInterpF16Pat_t16`。

### Lines 210-239: Adjusts TableGen properties and predicates
```tablegen
let True16Predicate = UseRealTrue16Insts in {
defm : VInterpF16Pat_t16<int_amdgcn_interp_inreg_p10_f16,
                     V_INTERP_P10_F16_F32_inreg_t16, f32, 0>;
defm : VInterpF16Pat_t16<int_amdgcn_interp_inreg_p2_f16,
                     V_INTERP_P2_F16_F32_inreg_t16, f16, 1>;
defm : VInterpF16Pat_t16<int_amdgcn_interp_p10_rtz_f16,
                     V_INTERP_P10_RTZ_F16_F32_inreg_t16, f32, 0>;
defm : VInterpF16Pat_t16<int_amdgcn_interp_p2_rtz_f16,
                     V_INTERP_P2_RTZ_F16_F32_inreg_t16, f16, 1>;
}

let True16Predicate = UseFakeTrue16Insts in {
defm : VInterpF16Pat<int_amdgcn_interp_inreg_p10_f16,
                     V_INTERP_P10_F16_F32_inreg_fake16, f32,
                     [VINTERPModsHi, VINTERPMods, VINTERPModsHi]>;
defm : VInterpF16Pat<int_amdgcn_interp_inreg_p2_f16,
                     V_INTERP_P2_F16_F32_inreg_fake16, f16,
                     [VINTERPModsHi, VINTERPMods, VINTERPMods]>;
defm : VInterpF16Pat<int_amdgcn_interp_p10_rtz_f16,
                     V_INTERP_P10_RTZ_F16_F32_inreg_fake16, f32,
                     [VINTERPModsHi, VINTERPMods, VINTERPModsHi]>;
defm : VInterpF16Pat<int_amdgcn_interp_p2_rtz_f16,
                     V_INTERP_P2_RTZ_F16_F32_inreg_fake16, f16,
                     [VINTERPModsHi, VINTERPMods, VINTERPMods]>;
}

//===----------------------------------------------------------------------===//
// VINTERP Real Instructions
//===----------------------------------------------------------------------===//

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 240-272: Defines TableGen multiclass VINTERP_Real
```tablegen
multiclass VINTERP_Real <GFXGen Gen, bits<7> op, string asmName> {
  defvar ps = !cast<VOP3_Pseudo>(NAME);
  let AssemblerPredicate = Gen.AssemblerPredicate, DecoderNamespace = Gen.DecoderNamespace #
                           !if(ps.Pfl.IsRealTrue16, "", "_FAKE16") in {
    if !eq(Gen.Subtarget, SIEncodingFamily.GFX11) then
    def Gen.Suffix :
      VINTERP_Real<ps, Gen.Subtarget, asmName>,
      VINTERPe_gfx11<op>;
    else // gfx12, gfx13
    def Gen.Suffix :
      VINTERP_Real<ps, Gen.Subtarget, asmName>,
      VINTERPe_gfx12_gfx13<op>;
  }
}

multiclass VINTERP_Real_gfx11_gfx12_gfx13 <bits<7> op, string asmName = !cast<VOP3_Pseudo>(NAME).Mnemonic> :
  VINTERP_Real<GFX11Gen, op, asmName>, VINTERP_Real<GFX12Gen, op, asmName>, VINTERP_Real<GFX13Gen, op, asmName>;

multiclass VINTERP_Real_t16_and_fake16_gfx11_gfx12_gfx13 <bits<7> op, string asmName = !cast<VOP3_Pseudo>(NAME).Mnemonic> {
  defm _t16:    VINTERP_Real_gfx11_gfx12_gfx13<op, asmName>;
  defm _fake16: VINTERP_Real_gfx11_gfx12_gfx13<op, asmName>;
}


defm V_INTERP_P10_F32_inreg : VINTERP_Real_gfx11_gfx12_gfx13<0x000>;
defm V_INTERP_P2_F32_inreg : VINTERP_Real_gfx11_gfx12_gfx13<0x001>;
defm V_INTERP_P10_F16_F32_inreg : VINTERP_Real_t16_and_fake16_gfx11_gfx12_gfx13<0x002, "v_interp_p10_f16_f32">;
defm V_INTERP_P2_F16_F32_inreg : VINTERP_Real_t16_and_fake16_gfx11_gfx12_gfx13<0x003, "v_interp_p2_f16_f32">;
defm V_INTERP_P10_RTZ_F16_F32_inreg : VINTERP_Real_t16_and_fake16_gfx11_gfx12_gfx13<0x004, "v_interp_p10_rtz_f16_f32">;
defm V_INTERP_P2_RTZ_F16_F32_inreg : VINTERP_Real_t16_and_fake16_gfx11_gfx12_gfx13<0x005, "v_interp_p2_rtz_f16_f32">;

let AssemblerPredicate = isGFX11Plus in
def : AMDGPUMnemonicAlias<"v_interp_p2_new_f32", "v_interp_p2_f32">;
```
**EN:** This section contains concrete logic for TableGen multiclass VINTERP_Real. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VINTERP_Real`, `Gen`, `VINTERP_Real_gfx11_gfx12_gfx13`.
**CN:** 本节包含与 TableGen multiclass VINTERP_Real 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VINTERP_Real`, `Gen`, `VINTERP_Real_gfx11_gfx12_gfx13`。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `VINTERPe`, `VINTERPe_gfx11`, `VINTERPe_gfx12_gfx13`, `VINTERP_Pseudo`, `VINTERP_Real`, `VOP3_VINTERP_F32`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
