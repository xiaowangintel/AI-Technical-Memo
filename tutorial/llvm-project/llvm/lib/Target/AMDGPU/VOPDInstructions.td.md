# VOPDInstructions.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/VOPDInstructions.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines VOPDInstructions records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 VOPDInstructions 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, comments, and TableGen overview
```tablegen
//===-- VOPDInstructions.td - Vector Instruction Definitions --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Encodings
//===----------------------------------------------------------------------===//

class VOPDe<bits<4> opX, bits<5> opY> : Enc64 {
  bits<9> src0X;
  bits<8> vsrc1X;
  bits<8> vdstX;
  bits<9> src0Y;
  bits<8> vsrc1Y;
  bits<8> vdstY;

  let Inst{8-0} = src0X;
  let Inst{16-9} = vsrc1X;
  let Inst{21-17} = opY;
  let Inst{25-22} = opX;
  let Inst{31-26} = 0x32; // encoding
  let Inst{40-32} = src0Y;
  let Inst{48-41} = vsrc1Y;
  let Inst{55-49} = vdstY{7-1};
  let Inst{63-56} = vdstX;
}

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `VOPDe`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`VOPDe`。

### Lines 32-52: Defines TableGen class VOPD_MADKe
```tablegen
class VOPD_MADKe<bits<4> opX, bits<5> opY> : Enc96 {
  bits<9> src0X;
  bits<8> vsrc1X;
  bits<8> vdstX;
  bits<9> src0Y;
  bits<8> vsrc1Y;
  bits<8> vdstY;
  bits<32> imm;

  let Inst{8-0} = src0X;
  let Inst{16-9} = vsrc1X;
  let Inst{21-17} = opY;
  let Inst{25-22} = opX;
  let Inst{31-26} = 0x32; // encoding
  let Inst{40-32} = src0Y;
  let Inst{48-41} = vsrc1Y;
  let Inst{55-49} = vdstY{7-1};
  let Inst{63-56} = vdstX;
  let Inst{95-64} = imm;
}

```
**EN:** This section contains concrete logic for TableGen class VOPD_MADKe. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VOPD_MADKe`.
**CN:** 本节包含与 TableGen class VOPD_MADKe 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VOPD_MADKe`。

### Lines 53-83: Defines TableGen class VOPD3e
```tablegen
class VOPD3e<bits<6> opX, bits<6> opY, VOP_Pseudo VDX, VOP_Pseudo VDY> : Enc96 {
  bits<9> src0X;
  bits<8> vsrc1X;
  bits<8> vsrc2X;
  bits<8> vdstX;
  bits<9> src0Y;
  bits<8> vsrc1Y;
  bits<8> vsrc2Y;
  bits<8> vdstY;
  // neg modifiers
  bit src0X_modifiers;
  bit src0Y_modifiers;
  bit vsrc1X_modifiers;
  bit vsrc1Y_modifiers;
  bit vsrc2X_modifiers;
  bit vsrc2Y_modifiers;
  bits<8> bitop3;

  let Inst{8-0} = src0X;
  let Inst{17-12} = opY;
  let Inst{23-18} = opX;
  let Inst{31-24} = 0xcf; // encoding
  let Inst{40-32} = src0Y;
  let Inst{41} = !if(VDX.Pfl.HasModifiers, src0X_modifiers, 0);
  let Inst{42} = !if(!and(VDX.Pfl.HasSrc1, VDX.Pfl.HasModifiers), vsrc1X_modifiers, 0);
  let Inst{43} = !if(!and(VDX.Pfl.HasVOPD3Src2, VDX.Pfl.HasModifiers), vsrc2X_modifiers, 0);
  let Inst{44} = !if(VDY.Pfl.HasModifiers, src0Y_modifiers, 0);
  let Inst{45} = !if(!and(VDY.Pfl.HasSrc1, VDY.Pfl.HasModifiers), vsrc1Y_modifiers, 0);
  let Inst{46} = !if(!and(VDY.Pfl.HasVOPD3Src2, VDY.Pfl.HasModifiers), vsrc2Y_modifiers, 0);
  let Inst{55-48} = !if(!eq(!find(VDX.Pfl.AsmVOPD3X, "$vsrc1X"), -1), ?, vsrc1X);

```
**EN:** This section contains concrete logic for TableGen class VOPD3e. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VOPD3e`.
**CN:** 本节包含与 TableGen class VOPD3e 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VOPD3e`。

### Lines 84-107: Defines TableGen class GFXGenD
```tablegen
  // Despite the vsrc operand name, SGPRs can be used for vsrc2X for
  // V_DUAL_CNDMASK_B32
  let Inst{63-56} = !if(!eq(!find(VDX.Pfl.AsmVOPD3X, "$vsrc2X"), -1), ?, vsrc2X);
  let Inst{71-64} = vdstX;
  let Inst{79-72} = !if(!eq(!find(VDY.Pfl.AsmVOPD3Y, "$vsrc1Y"), -1), ?, vsrc1Y);
  let Inst{87-80} = !if(!ne(!find(VDY.Pfl.AsmVOPD3Y, "bitop"), -1), bitop3,
                        !if(!eq(!find(VDY.Pfl.AsmVOPD3Y, "$vsrc2Y"), -1), ?, vsrc2Y));
  let Inst{95-88} = vdstY;
}

//===----------------------------------------------------------------------===//
// VOPD classes
//===----------------------------------------------------------------------===//


class GFXGenD<GFXGen Gen, list<string> DXPseudos, list<string> DYPseudos,
              Predicate subtargetPred = Gen.AssemblerPredicate> :
    GFXGen<Gen.AssemblerPredicate, Gen.DecoderNamespace, Gen.Suffix,
           Gen.Subtarget> {
  list<string> VOPDXPseudos = DXPseudos;
  list<string> VOPDYPseudos = DYPseudos;
  Predicate SubtargetPredicate = subtargetPred;
}

```
**EN:** This section contains concrete logic for TableGen class GFXGenD. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GFXGenD`.
**CN:** 本节包含与 TableGen class GFXGenD 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GFXGenD`。

### Lines 108-141: Defines TableGen class VOPD_Base
```tablegen
class VOPD_Base<dag outs, dag ins, string asm, VOP_Pseudo VDX, VOP_Pseudo VDY,
                VOPD_Component XasVC, VOPD_Component YasVC, GFXGenD Gen>
    : VOPAnyCommon<outs, ins, asm, []>,
      VOP<NAME>,
      SIMCInstr<NAME, Gen.Subtarget> {
  // Fields for table indexing
  Instruction Opcode = !cast<Instruction>(NAME);
  bits<6> OpX = XasVC.VOPDOp;
  bits<6> OpY = YasVC.VOPDOp;
  bits<4> SubTgt = Gen.Subtarget;

  let VALU = 1;

  let DecoderNamespace = Gen.DecoderNamespace;
  let AssemblerPredicate = Gen.AssemblerPredicate;
  let WaveSizePredicate = isWave32;
  let isCodeGenOnly = 0;
  let SubtargetPredicate = Gen.SubtargetPredicate;
  let AsmMatchConverter  = "cvtVOPD";
  let Size = 8;
  let ReadsModeReg = !or(VDX.ReadsModeReg, VDY.ReadsModeReg);
  let mayRaiseFPException = ReadsModeReg;

  // V_DUAL_FMAC and V_DUAL_DOT2ACC_F32_F16 and V_DUAL_DOT2ACC_F32_BF16 need a
  // dummy src2 tied to dst for passes to track its uses. Its presence does not
  // affect VOPD formation rules because the rules for src2 and dst are the
  // same. src2X and src2Y should not be encoded.
  bit hasSrc2AccX = !or(!eq(VDX.Mnemonic, "v_fmac_f32"), !eq(VDX.Mnemonic, "v_dot2c_f32_f16"), !eq(VDX.Mnemonic, "v_dot2c_f32_bf16"));
  bit hasSrc2AccY = !or(!eq(VDY.Mnemonic, "v_fmac_f32"), !eq(VDY.Mnemonic, "v_dot2c_f32_f16"), !eq(VDY.Mnemonic, "v_dot2c_f32_bf16"));
  string ConstraintsX = !if(hasSrc2AccX, "$src2X = $vdstX", "");
  string ConstraintsY = !if(hasSrc2AccY, "$src2Y = $vdstY", "");
  let Constraints =
      ConstraintsX # !if(!and(hasSrc2AccX, hasSrc2AccY), ", ", "") # ConstraintsY;

```
**EN:** This section contains concrete logic for TableGen class VOPD_Base. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VOPD_Base`.
**CN:** 本节包含与 TableGen class VOPD_Base 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VOPD_Base`。

### Lines 142-164: Defines TableGen class VOPD
```tablegen
  let Uses = RegListUnion<VDX.Uses, VDY.Uses>.ret;
  let Defs = RegListUnion<VDX.Defs, VDY.Defs>.ret;
  let SchedRW = !listconcat(VDX.SchedRW, VDY.SchedRW);
}

class VOPD<dag outs, dag ins, string asm, VOP_Pseudo VDX, VOP_Pseudo VDY,
           VOPD_Component XasVC, VOPD_Component YasVC, GFXGenD Gen>
    : VOPD_Base<outs, ins, asm, VDX, VDY, XasVC, YasVC, Gen>,
      VOPDe<XasVC.VOPDOp{3-0}, YasVC.VOPDOp{4-0}> {
  let Inst{16-9} = !if (!eq(VDX.Mnemonic, "v_mov_b32"), ?, vsrc1X);
  let Inst{48-41} = !if (!eq(VDY.Mnemonic, "v_mov_b32"), ?, vsrc1Y);
}

class VOPD_MADK<dag outs, dag ins, string asm, VOP_Pseudo VDX, VOP_Pseudo VDY,
                VOPD_Component XasVC, VOPD_Component YasVC, GFXGenD Gen>
    : VOPD_Base<outs, ins, asm, VDX, VDY, XasVC, YasVC, Gen>,
      VOPD_MADKe<XasVC.VOPDOp{3-0}, YasVC.VOPDOp{4-0}> {
  let Inst{16-9} = !if (!eq(VDX.Mnemonic, "v_mov_b32"), ?, vsrc1X);
  let Inst{48-41} = !if (!eq(VDY.Mnemonic, "v_mov_b32"), ?, vsrc1Y);
  let Size = 12;
  let FixedSize = 1;
}

```
**EN:** This section contains concrete logic for TableGen class VOPD. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VOPD`, `VOPD_MADK`.
**CN:** 本节包含与 TableGen class VOPD 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VOPD`, `VOPD_MADK`。

### Lines 165-196: Defines TableGen class VOPD3
```tablegen
class VOPD3<dag outs, dag ins, string asm, VOP_Pseudo VDX, VOP_Pseudo VDY,
            VOPD_Component XasVC, VOPD_Component YasVC, GFXGenD Gen>
    : VOPD_Base<outs, ins, asm, VDX, VDY, XasVC, YasVC, Gen>,
      VOPD3e<XasVC.VOPDOp, YasVC.VOPDOp, VDX, VDY> {
  let VOPD3 = 1;
  let Size = 12;
  // VOPD3 uses promoted form of VOP2 instructions, so V_CNDMASK_B32 is not
  // limited to VCC src2 only, and a real SGPR will be used as an operand
  // instead.
  defvar UsesX = !if(!eq(VDX, V_CNDMASK_B32_e32), !filter(x, VDX.Uses, !ne(x, VCC)), VDX.Uses);
  defvar UsesY = !if(!eq(VDY, V_CNDMASK_B32_e32), !filter(x, VDY.Uses, !ne(x, VCC)), VDY.Uses);
  let Uses = RegListUnion<UsesX, UsesY>.ret;
}

defvar VOPDPseudosCommon = [
  "V_FMAC_F32_e32", "V_FMAAK_F32", "V_FMAMK_F32", "V_MUL_F32_e32",
  "V_ADD_F32_e32", "V_SUB_F32_e32", "V_SUBREV_F32_e32", "V_MUL_LEGACY_F32_e32",
  "V_MOV_B32_e32", "V_CNDMASK_B32_e32", "V_MAX_F32_e32", "V_MIN_F32_e32",
  "V_DOT2C_F32_F16_e32", "V_DOT2C_F32_BF16_e32"
];
defvar VOPDYOnlyPseudosCommon = ["V_ADD_U32_e32", "V_LSHLREV_B32_e32"];
defvar VOPDYOnlyPseudosGFX11_12 = ["V_AND_B32_e32"];
defvar VOPDYOnlyPseudosGFX1250 = ["V_MAX_I32_e32", "V_MIN_I32_e32",
                                  "V_SUB_U32_e32", "V_LSHRREV_B32_e32",
                                  "V_ASHRREV_I32_e32"];

defvar VOPDXPseudosGFX11 = VOPDPseudosCommon;
defvar VOPDXPseudosGFX12 = VOPDPseudosCommon;
defvar VOPDYPseudosGFX11 = !listconcat(VOPDXPseudosGFX11, VOPDYOnlyPseudosCommon, VOPDYOnlyPseudosGFX11_12);
defvar VOPDYPseudosGFX12 = !listconcat(VOPDXPseudosGFX12, VOPDYOnlyPseudosCommon, VOPDYOnlyPseudosGFX11_12);
defvar VOPDYPseudosGFX1250 = !listconcat(VOPDXPseudosGFX12, VOPDYOnlyPseudosCommon, VOPDYOnlyPseudosGFX1250);

```
**EN:** This section contains concrete logic for TableGen class VOPD3. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VOPD3`.
**CN:** 本节包含与 TableGen class VOPD3 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VOPD3`。

### Lines 197-214: Defines TableGen record GFX11GenD
```tablegen
def GFX11GenD : GFXGenD<GFX11Not11_70Gen, VOPDXPseudosGFX11, VOPDYPseudosGFX11>;
def GFX1170GenD : GFXGenD<GFX1170Gen, VOPDXPseudosGFX11, VOPDYPseudosGFX11>;
def GFX12GenD : GFXGenD<GFX12Not12_50Gen, VOPDXPseudosGFX12, VOPDYPseudosGFX12>;
def GFX1250GenD : GFXGenD<GFX1250Gen, VOPDXPseudosGFX12, VOPDYPseudosGFX1250>;
def GFX13GenD : GFXGenD<GFX13Gen, VOPDXPseudosGFX12, VOPDYPseudosGFX1250>;


def VOPDDstYOperand : RegisterOperand<VGPR_32, "printRegularOperand"> {
  let DecoderMethod = "decodeOperandVOPDDstY";
}

class getRenamed<string VOPDName, GFXGen Gen> {
  string ret = !cond(!eq(Gen.Subtarget, GFX11Gen.Subtarget) : VOPDName,
                     !eq(VOPDName, "v_dual_max_f32")        : "v_dual_max_num_f32",
                     !eq(VOPDName, "v_dual_min_f32")        : "v_dual_min_num_f32",
                     true                                   : VOPDName);
}

```
**EN:** This section contains concrete logic for TableGen record GFX11GenD. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GFX11GenD`, `GFX1170GenD`, `GFX12GenD`.
**CN:** 本节包含与 TableGen record GFX11GenD 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GFX11GenD`, `GFX1170GenD`, `GFX12GenD`。

### Lines 215-248: Defines TableGen record OpName
```tablegen
foreach Gen = [GFX11GenD, GFX1170GenD, GFX12GenD, GFX1250GenD, GFX13GenD] in {
  foreach x = Gen.VOPDXPseudos in {
    foreach y = Gen.VOPDYPseudos in {
      defvar xInst = !cast<VOP_Pseudo>(x);
      defvar yInst = !cast<VOP_Pseudo>(y);
      defvar XasVC = !cast<VOPD_Component>(x);
      defvar YasVC = !cast<VOPD_Component>(y);
      defvar xAsmName = getRenamed<XasVC.VOPDName, Gen>.ret;
      defvar yAsmName = getRenamed<YasVC.VOPDName, Gen>.ret;
      defvar isMADK = !or(!eq(x, "V_FMAAK_F32"), !eq(x, "V_FMAMK_F32"),
                          !eq(y, "V_FMAAK_F32"), !eq(y, "V_FMAMK_F32"));
      defvar isOpXMADK = !or(!eq(x, "V_FMAAK_F32"), !eq(x, "V_FMAMK_F32"));
      defvar isOpYMADK = !or(!eq(y, "V_FMAAK_F32"), !eq(y, "V_FMAMK_F32"));
      defvar OpName = "V_DUAL_" # !substr(x,2) # "_X_" # !substr(y,2) # Gen.Suffix;
      defvar outs = (outs VGPROp_32:$vdstX, VOPDDstYOperand:$vdstY);
      if !or(isOpXMADK, isOpYMADK) then {
        // If Both X and Y are MADK, the mandatory literal of X additionally must
        // use an alternate operand format which defers to the 'real' Y literal.
        defvar isOpXYMADK = !and(isOpXMADK, isOpYMADK);
        defvar X_MADK_Pfl = !cast<VOP_MADK_Base>(xInst.Pfl);
        defvar asm = xAsmName #" "#
                     !if(isOpXYMADK, X_MADK_Pfl.AsmVOPDX_immX, xInst.Pfl.AsmVOPDX)#
                     " :: "# yAsmName #" "# yInst.Pfl.AsmVOPDY;
        defvar ins = !con(!if(isOpXYMADK, xInst.Pfl.InsVOPDX_immX, xInst.Pfl.InsVOPDX),
                          yInst.Pfl.InsVOPDY);
        def OpName : VOPD_MADK<outs, ins, asm, xInst, yInst, XasVC, YasVC, Gen>;
      } else {
        defvar ins = !con(xInst.Pfl.InsVOPDX, yInst.Pfl.InsVOPDY);
        defvar asm = xAsmName #" "# xInst.Pfl.AsmVOPDX #" :: "# yAsmName #" "# yInst.Pfl.AsmVOPDY;
        def OpName : VOPD<outs, ins, asm, xInst, yInst, XasVC, YasVC, Gen>;
      }
    }
  }
}
```
**EN:** This section contains concrete logic for TableGen record OpName. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `OpName`.
**CN:** 本节包含与 TableGen record OpName 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`OpName`。

### Lines 249-274: Defines TableGen record GFX1250GenD3
```tablegen

defvar VOPD3XPseudosExtraGFX13 = ["V_ADD_U32_e32", "V_LSHLREV_B32_e32", "V_FMA_F32_e64", "V_SUB_U32_e32",
                                  "V_LSHRREV_B32_e32", "V_ASHRREV_I32_e32"];
defvar VOPD3XPseudosExtraGFX1250 = ["V_FMA_F64_e64", "V_ADD_F64_pseudo_e32",
                                    "V_MUL_F64_pseudo_e32", "V_MAX_NUM_F64_e32", "V_MIN_NUM_F64_e32"];
defvar VOPD3XPseudosGFX13 = !listconcat(
                              !filter(x, VOPDXPseudosGFX12, !and(!eq(!find(x, "FMAAK"), -1),
                                                                 !eq(!find(x, "FMAMK"), -1))),
                              VOPD3XPseudosExtraGFX13);
defvar VOPD3XPseudosGFX1250 = !listconcat(VOPD3XPseudosGFX13, VOPD3XPseudosExtraGFX1250);
defvar VOPD3YPseudosExtra = ["V_BITOP3_B32_e64", "V_FMA_F32_e64"];
defvar VOPD3YPseudosGFX1250 = !listconcat(
                                !filter(x, VOPDYPseudosGFX1250, !and(!eq(!find(x, "FMAAK"), -1),
                                                                     !eq(!find(x, "FMAMK"), -1))),
                                VOPD3YPseudosExtra);

def GFX1250GenD3 : GFXGenD<GFX1250Gen, VOPD3XPseudosGFX1250, VOPD3YPseudosGFX1250>;

def GFX13GenD3 : GFXGenD<GFX13Gen, VOPD3XPseudosGFX13, VOPD3YPseudosGFX1250>;

class getOpcMap<string OPName> {
  defvar BaseName = !substr(OPName,2);
  string ret = !cond(!eq(BaseName, "BITOP3_B32_e64")   : "BITOP2_B32_e64",
                     1 : BaseName);
}

```
**EN:** This section contains concrete logic for TableGen record GFX1250GenD3. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GFX1250GenD3`, `GFX13GenD3`, `getOpcMap`.
**CN:** 本节包含与 TableGen record GFX1250GenD3 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GFX1250GenD3`, `GFX13GenD3`, `getOpcMap`。

### Lines 275-291: Defines TableGen record OpName
```tablegen
foreach Gen = [GFX1250GenD3, GFX13GenD3] in {
  foreach x = Gen.VOPDXPseudos in {
    foreach y = Gen.VOPDYPseudos in {
      defvar xInst = !cast<VOP_Pseudo>(x);
      defvar yInst = !cast<VOP_Pseudo>(y);
      defvar XasVC = !cast<VOPD_Component>(x);
      defvar YasVC = !cast<VOPD_Component>(y);
      defvar xAsmName = getRenamed<XasVC.VOPDName, Gen>.ret;
      defvar yAsmName = getRenamed<YasVC.VOPDName, Gen>.ret;
      defvar OpName = "V_DUAL_" # getOpcMap<x>.ret # "_X_" # getOpcMap<y>.ret # "_e96" # Gen.Suffix;
      defvar asm = xAsmName # xInst.Pfl.AsmVOPD3X #" :: "# yAsmName #" "# yInst.Pfl.AsmVOPD3Y;
      defvar ins = !con(xInst.Pfl.InsVOPD3X, yInst.Pfl.InsVOPD3Y);
      defvar outs = (outs xInst.Pfl.DstRC:$vdstX, yInst.Pfl.DstRC:$vdstY);
      def OpName : VOPD3<outs, ins, asm, xInst, yInst, XasVC, YasVC, Gen>;
    }
  }
}
```
**EN:** This section contains concrete logic for TableGen record OpName. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `OpName`.
**CN:** 本节包含与 TableGen record OpName 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`OpName`。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `VOPDe`, `VOPD_MADKe`, `VOPD3e`, `GFXGenD`, `VOPD_Base`, `VOPD`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理; predicates and constraints / 谓词与约束
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
