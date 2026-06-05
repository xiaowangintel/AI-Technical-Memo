# DSDIRInstructions.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/DSDIRInstructions.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines DSDIRInstructions records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 DSDIRInstructions 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: File banner, comments, and TableGen overview
```tablegen
//===-- DSDIRInstructions.td - LDS/VDS Direct Instruction Definitions -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// LDSDIR/VDSDIR encoding (LDSDIR is gfx11, VDSDIR is gfx12+)
//===----------------------------------------------------------------------===//

class LDSDIRe<bits<2> op, bit is_direct> : Enc32 {
  // encoding fields
  bits<2> attrchan;
  bits<6> attr;
  bits<4> waitvdst;
  bits<8> vdst;

  // encoding
  let Inst{31-24} = 0xce; // encoding
  let Inst{23-22} = 0x0; // reserved
  let Inst{21-20} = op;
  let Inst{19-16} = waitvdst;
  let Inst{15-10} = !if(is_direct, ?, attr);
  let Inst{9-8} = !if(is_direct, ?, attrchan);
  let Inst{7-0} = vdst;
}

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `LDSDIRe`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`LDSDIRe`。

### Lines 30-59: Defines TableGen class VDSDIRe
```tablegen
class VDSDIRe<bits<2> op, bit is_direct> : Enc32 {
  // encoding fields
  bits<2> attrchan;
  bits<6> attr;
  bits<4> waitvdst;
  bits<8> vdst;
  bits<1> waitvsrc;

  // encoding
  let Inst{31-24} = 0xce; // encoding
  let Inst{23} = waitvsrc;
  let Inst{22} = 0x0; // reserved
  let Inst{21-20} = op;
  let Inst{19-16} = waitvdst;
  let Inst{15-10} = !if(is_direct, ?, attr);
  let Inst{9-8} = !if(is_direct, ?, attrchan);
  let Inst{7-0} = vdst;
}

//===----------------------------------------------------------------------===//
// LDSDIR/VDSDIR Classes
//===----------------------------------------------------------------------===//

class LDSDIR_getIns<bit direct> {
  dag ret = !if(direct,
    (ins WaitVDST:$waitvdst),
    (ins InterpAttr:$attr, InterpAttrChan:$attrchan, WaitVDST:$waitvdst)
  );
}

```
**EN:** This section contains concrete logic for TableGen class VDSDIRe. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VDSDIRe`, `LDSDIR_getIns`.
**CN:** 本节包含与 TableGen class VDSDIRe 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VDSDIRe`, `LDSDIR_getIns`。

### Lines 60-88: Defines TableGen class VDSDIR_getIns
```tablegen
class VDSDIR_getIns<bit direct> {
  dag ret = !if(direct,
    (ins WaitVAVDst:$waitvdst, WaitVMVSrc:$waitvsrc),
    (ins InterpAttr:$attr, InterpAttrChan:$attrchan, WaitVAVDst:$waitvdst,
         WaitVMVSrc:$waitvsrc)
  );
}

class DSDIR_Common<string opName, string asm = "", dag ins, bit direct> :
  InstSI<(outs VGPR_32:$vdst), ins, asm> {
  let LDSDIR = 1;
  let EXP_CNT = 1;

  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
  let maybeAtomic = 0;

  string Mnemonic = opName;
  let UseNamedOperandTable = 1;

  let Uses = [M0, EXEC];
  let DisableWQM = 0;
  let SchedRW = [WriteLDS];

  bit is_direct;
  let is_direct = direct;
}

```
**EN:** This section contains concrete logic for TableGen class VDSDIR_getIns. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VDSDIR_getIns`, `DSDIR_Common`.
**CN:** 本节包含与 TableGen class VDSDIR_getIns 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VDSDIR_getIns`, `DSDIR_Common`。

### Lines 89-122: Defines TableGen class DSDIR_Pseudo
```tablegen
class DSDIR_Pseudo<string opName, dag ins, bit direct> :
  DSDIR_Common<opName, "", ins, direct>,
  SIMCInstr<opName, SIEncodingFamily.NONE> {
  let isPseudo = 1;
  let isCodeGenOnly = 1;
}

class LDSDIR_getAsm<bit direct> {
  string ret = !if(direct,
    " $vdst$waitvdst",
    " $vdst, $attr$attrchan$waitvdst"
  );
}

class VDSDIR_getAsm<bit direct> {
  string ret = !if(direct,
    " $vdst$waitvdst$waitvsrc",
    " $vdst, $attr$attrchan$waitvdst$waitvsrc"
  );
}

class DSDIR_Real<DSDIR_Pseudo lds, dag ins, string asm, int subtarget> :
  DSDIR_Common<lds.Mnemonic,
               lds.Mnemonic # asm,
               ins,
               lds.is_direct>,
  SIMCInstr <lds.PseudoInstr, subtarget> {
  let isPseudo = 0;
  let isCodeGenOnly = 0;

  // copy SubtargetPredicate from pseudo.
  let SubtargetPredicate = lds.SubtargetPredicate;
}

```
**EN:** This section contains concrete logic for TableGen class DSDIR_Pseudo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DSDIR_Pseudo`, `LDSDIR_getAsm`, `VDSDIR_getAsm`.
**CN:** 本节包含与 TableGen class DSDIR_Pseudo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DSDIR_Pseudo`, `LDSDIR_getAsm`, `VDSDIR_getAsm`。

### Lines 123-153: Defines TableGen record LDS_DIRECT_LOAD
```tablegen
//===----------------------------------------------------------------------===//
// LDS/VDS Direct Instructions
//===----------------------------------------------------------------------===//

let SubtargetPredicate = isGFX11Only in {

def LDS_DIRECT_LOAD : DSDIR_Pseudo<"lds_direct_load", LDSDIR_getIns<1>.ret, 1>;
def LDS_PARAM_LOAD : DSDIR_Pseudo<"lds_param_load", LDSDIR_getIns<0>.ret, 0>;

def : GCNPat <
  (f32 (int_amdgcn_lds_direct_load M0)),
  (LDS_DIRECT_LOAD 0)
>;

def : GCNPat <
  (f32 (int_amdgcn_lds_param_load timm:$attrchan, timm:$attr, M0)),
  (LDS_PARAM_LOAD timm:$attr, timm:$attrchan, 0)
>;

} // End SubtargetPredicate = isGFX11Only

let SubtargetPredicate = isGFX12PlusNot12_50 in {

def DS_DIRECT_LOAD : DSDIR_Pseudo<"ds_direct_load", VDSDIR_getIns<1>.ret, 1>;
def DS_PARAM_LOAD : DSDIR_Pseudo<"ds_param_load", VDSDIR_getIns<0>.ret, 0>;

def : GCNPat <
  (f32 (int_amdgcn_lds_direct_load M0)),
  (DS_DIRECT_LOAD 0, 1)
>;

```
**EN:** This section contains concrete logic for TableGen record LDS_DIRECT_LOAD. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `LDS_DIRECT_LOAD`, `LDS_PARAM_LOAD`, `DS_DIRECT_LOAD`.
**CN:** 本节包含与 TableGen record LDS_DIRECT_LOAD 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`LDS_DIRECT_LOAD`, `LDS_PARAM_LOAD`, `DS_DIRECT_LOAD`。

### Lines 154-182: Defines TableGen multiclass DSDIR_Real_gfx11
```tablegen
def : GCNPat <
  (f32 (int_amdgcn_lds_param_load timm:$attrchan, timm:$attr, M0)),
  (DS_PARAM_LOAD timm:$attr, timm:$attrchan, 0, 1)
>;

} // End SubtargetPredicate = isGFX12PlusNot12_50.

//===----------------------------------------------------------------------===//
// GFX11
//===----------------------------------------------------------------------===//

multiclass DSDIR_Real_gfx11<bits<2> op> {
  defvar lds = !cast<DSDIR_Pseudo>(NAME);
  def _gfx11 : DSDIR_Real<lds, lds.InOperandList,
                          LDSDIR_getAsm<lds.is_direct>.ret,
                          SIEncodingFamily.GFX11>,
               LDSDIRe<op, lds.is_direct> {
    let AssemblerPredicate = isGFX11Only;
    let DecoderNamespace = "GFX11";
  }
}

defm LDS_PARAM_LOAD : DSDIR_Real_gfx11<0x0>;
defm LDS_DIRECT_LOAD : DSDIR_Real_gfx11<0x1>;

//===----------------------------------------------------------------------===//
// GFX12, GFX13
//===----------------------------------------------------------------------===//

```
**EN:** This section contains concrete logic for TableGen multiclass DSDIR_Real_gfx11. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DSDIR_Real_gfx11`, `_gfx11`, `LDS_PARAM_LOAD`.
**CN:** 本节包含与 TableGen multiclass DSDIR_Real_gfx11 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DSDIR_Real_gfx11`, `_gfx11`, `LDS_PARAM_LOAD`。

### Lines 183-204: Defines TableGen multiclass DSDIR_Real
```tablegen
multiclass DSDIR_Real<GFXGen Gen, bits<2> op> {
  defvar lds = !cast<DSDIR_Pseudo>(NAME);
  def Gen.Suffix : DSDIR_Real<lds, lds.InOperandList,
                              VDSDIR_getAsm<lds.is_direct>.ret,
                              Gen.Subtarget>,
                   VDSDIRe<op, lds.is_direct> {
    let AssemblerPredicate = Gen.AssemblerPredicate;
    let DecoderNamespace = Gen.DecoderNamespace;
  }
}

multiclass DSDIR_Real_gfx12_gfx13<bits<2> op> :
  DSDIR_Real<GFX12Gen, op>,
  DSDIR_Real<GFX13Gen, op>;

defm DS_PARAM_LOAD  : DSDIR_Real_gfx12_gfx13<0x0>;
defm DS_DIRECT_LOAD : DSDIR_Real_gfx12_gfx13<0x1>;

let SubtargetPredicate = isGFX12Plus in {
  def : AMDGPUMnemonicAlias<"lds_param_load", "ds_param_load">;
  def : AMDGPUMnemonicAlias<"lds_direct_load", "ds_direct_load">;
}
```
**EN:** This section contains concrete logic for TableGen multiclass DSDIR_Real. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DSDIR_Real`, `Gen`, `DSDIR_Real_gfx12_gfx13`.
**CN:** 本节包含与 TableGen multiclass DSDIR_Real 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DSDIR_Real`, `Gen`, `DSDIR_Real_gfx12_gfx13`。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `LDSDIRe`, `VDSDIRe`, `LDSDIR_getIns`, `VDSDIR_getIns`, `DSDIR_Common`, `DSDIR_Pseudo`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度; assembly handling / 汇编处理
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
