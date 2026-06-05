# AMDGPUCallingConv.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUCallingConv.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines AMDGPUCallingConv records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 AMDGPUCallingConv 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: File banner, comments, and TableGen overview
```tablegen
//===---- AMDCallingConv.td - Calling Conventions for Radeon GPUs ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This describes the calling conventions for the AMD Radeon GPUs.
//
//===----------------------------------------------------------------------===//

// Inversion of CCIfInReg
class CCIfNotInReg<CCAction A> : CCIf<"!ArgFlags.isInReg()", A> {}
class CCIfExtend<CCAction A>
  : CCIf<"ArgFlags.isSExt() || ArgFlags.isZExt()", A>;
class CCIfOrigTypeShaderCCIsSGPR<CCAction A>
  : CCIf<[{(!OrigTy->getScalarType()->isFloatTy() &&
            !OrigTy->getScalarType()->isHalfTy()) }], A>;


```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `CCIfNotInReg`, `CCIfExtend`, `CCIfOrigTypeShaderCCIsSGPR`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`CCIfNotInReg`, `CCIfExtend`, `CCIfOrigTypeShaderCCIsSGPR`。

### Lines 22-43: Defines TableGen record CC_SI_Gfx
```tablegen
// Calling convention for SI
def CC_SI_Gfx : CallingConv<[
  // 0-3 are reserved for the stack buffer descriptor
  // 30-31 are reserved for the return address
  // 32 is reserved for the stack pointer
  // 33 is reserved for the frame pointer
  // 34 is reserved for the base pointer
  CCIfInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(4, 30), !cast<Register>("SGPR"#i))  // SGPR4-29
  >>>,

  CCIfNotInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(0, 32), !cast<Register>("VGPR"#i))  // VGPR0-31
  >>>,

  CCIfType<[i32, f32, v2i16, v2f16, i16, f16, i1, bf16, v2bf16], CCAssignToStack<4, 4>>
]>;

def RetCC_SI_Gfx : CallingConv<[
  CCIfType<[i1], CCPromoteToType<i32>>,
  CCIfType<[i1, i16], CCIfExtend<CCPromoteToType<i32>>>,

```
**EN:** This section contains concrete logic for TableGen record CC_SI_Gfx. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `CC_SI_Gfx`, `RetCC_SI_Gfx`.
**CN:** 本节包含与 TableGen record CC_SI_Gfx 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`CC_SI_Gfx`, `RetCC_SI_Gfx`。

### Lines 44-63: Defines TableGen record CC_SI_SHADER
```tablegen
  CCIfNotInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(0, 136), !cast<Register>("VGPR"#i))  // VGPR0-135
  >>>,
]>;

def CC_SI_SHADER : CallingConv<[

  CCIfType<[i1], CCPromoteToType<i32>>,

  CCIfInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(0, 44), !cast<Register>("SGPR"#i))  // SGPR0-43
  >>>,

  // 32*4 + 4 is the minimum for a fetch shader consumer with 32 inputs.
  CCIfNotInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(0, 136), !cast<Register>("VGPR"#i))  // VGPR0-135
  >>>
]>;


```
**EN:** This section contains concrete logic for TableGen record CC_SI_SHADER. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `CC_SI_SHADER`.
**CN:** 本节包含与 TableGen record CC_SI_SHADER 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`CC_SI_SHADER`。

### Lines 64-87: Defines TableGen record RetCC_SI_Shader
```tablegen
def RetCC_SI_Shader : CallingConv<[
  CCIfType<[i1, i16], CCIfExtend<CCPromoteToType<i32>>>,
  CCIfType<[i32, i16, v2i16], CCIfOrigTypeShaderCCIsSGPR<CCAssignToReg<
    !foreach(i, !range(0, 44), !cast<Register>("SGPR"#i))  // SGPR0-43
  >>>,

  // 32*4 + 4 is the minimum for a fetch shader with 32 outputs.
  CCIfType<[f32, f16, v2f16, bf16, v2bf16, i32, i16, v2i16] , CCAssignToReg<
    !foreach(i, !range(0, 136), !cast<Register>("VGPR"#i))  // VGPR0-135
  >>
]>;

def CSR_AMDGPU_VGPRs : CalleeSavedRegs<
  // The CSRs & scratch-registers are interleaved at a split boundary of 8.
  (add (sequence "VGPR%u", 40, 47),
    (sequence "VGPR%u", 56, 63),
    (sequence "VGPR%u", 72, 79),
    (sequence "VGPR%u", 88, 95),
    (sequence "VGPR%u", 104, 111),
    (sequence "VGPR%u", 120, 127),
    (sequence "VGPR%u", 136, 143),
    (sequence "VGPR%u", 152, 159),
    (sequence "VGPR%u", 168, 175),
    (sequence "VGPR%u", 184, 191),
```
**EN:** This section contains concrete logic for TableGen record RetCC_SI_Shader. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `RetCC_SI_Shader`, `CSR_AMDGPU_VGPRs`.
**CN:** 本节包含与 TableGen record RetCC_SI_Shader 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`RetCC_SI_Shader`, `CSR_AMDGPU_VGPRs`。

### Lines 88-109: Defines TableGen record CSR_AMDGPU_AGPRs
```tablegen
    (sequence "VGPR%u", 200, 207),
    (sequence "VGPR%u", 216, 223),
    (sequence "VGPR%u", 232, 239),
    (sequence "VGPR%u", 248, 255))
>;

def CSR_AMDGPU_AGPRs : CalleeSavedRegs<
  (sequence "AGPR%u", 32, 255)
>;

def CSR_AMDGPU_SGPRs : CalleeSavedRegs<
  // Ensure that s30-s31 (return address), s32 (stack pointer), s33 (frame pointer),
  // and s34 (base pointer) are callee-saved. The striped layout starts from s40,
  // with a stripe width of 8. The last stripe is 10 wide instead of 8, to avoid
  // ending with a 2-wide stripe.
  (add (sequence "SGPR%u", 30, 39),
       (sequence "SGPR%u", 48, 55),
       (sequence "SGPR%u", 64, 71),
       (sequence "SGPR%u", 80, 87),
       (sequence "SGPR%u", 96, 105))
>;

```
**EN:** This section contains concrete logic for TableGen record CSR_AMDGPU_AGPRs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `CSR_AMDGPU_AGPRs`, `CSR_AMDGPU_SGPRs`.
**CN:** 本节包含与 TableGen record CSR_AMDGPU_AGPRs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`CSR_AMDGPU_AGPRs`, `CSR_AMDGPU_SGPRs`。

### Lines 110-133: Defines TableGen record CSR_AMDGPU_SI_Gfx_SGPRs
```tablegen
def CSR_AMDGPU_SI_Gfx_SGPRs : CalleeSavedRegs<
  (add (sequence "SGPR%u", 4, 31), (sequence "SGPR%u", 64, 105))
>;

def CSR_AMDGPU : CalleeSavedRegs<
  (add CSR_AMDGPU_VGPRs, CSR_AMDGPU_SGPRs)
>;

def CSR_AMDGPU_GFX90AInsts : CalleeSavedRegs<
  (add CSR_AMDGPU, CSR_AMDGPU_AGPRs)
>;

def CSR_AMDGPU_SI_Gfx : CalleeSavedRegs<
  (add CSR_AMDGPU_VGPRs, CSR_AMDGPU_SI_Gfx_SGPRs)
>;

def CSR_AMDGPU_SI_Gfx_GFX90AInsts : CalleeSavedRegs<
  (add CSR_AMDGPU_SI_Gfx, CSR_AMDGPU_AGPRs)
>;

def CSR_AMDGPU_CS_ChainPreserve : CalleeSavedRegs<
  (sequence "VGPR%u", 8, 255)
>;

```
**EN:** This section contains concrete logic for TableGen record CSR_AMDGPU_SI_Gfx_SGPRs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `CSR_AMDGPU_SI_Gfx_SGPRs`, `CSR_AMDGPU`, `CSR_AMDGPU_GFX90AInsts`.
**CN:** 本节包含与 TableGen record CSR_AMDGPU_SI_Gfx_SGPRs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`CSR_AMDGPU_SI_Gfx_SGPRs`, `CSR_AMDGPU`, `CSR_AMDGPU_GFX90AInsts`。

### Lines 134-151: Defines TableGen record CSR_AMDGPU_NoRegs
```tablegen
def CSR_AMDGPU_NoRegs : CalleeSavedRegs<(add)>;

// Calling convention for leaf functions
def CC_AMDGPU_Func : CallingConv<[
  CCIfByVal<CCPassByVal<4, 4>>,
  CCIfType<[i1], CCPromoteToType<i32>>,
  CCIfType<[i8, i16], CCIfExtend<CCPromoteToType<i32>>>,

  CCIfInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(0, 30), !cast<Register>("SGPR"#i))  // SGPR0-29
  >>>,

  CCIfType<[i32, f32, i16, f16, v2i16, v2f16, i1, bf16, v2bf16], CCAssignToReg<
    !foreach(i, !range(0, 32), !cast<Register>("VGPR"#i))  // VGPR0-31
  >>,
  CCIfType<[i32, f32, v2i16, v2f16, i16, f16, i1, bf16, v2bf16], CCAssignToStack<4, 4>>
]>;

```
**EN:** This section contains concrete logic for TableGen record CSR_AMDGPU_NoRegs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `CSR_AMDGPU_NoRegs`, `CC_AMDGPU_Func`.
**CN:** 本节包含与 TableGen record CSR_AMDGPU_NoRegs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`CSR_AMDGPU_NoRegs`, `CC_AMDGPU_Func`。

### Lines 152-174: Defines TableGen record RetCC_AMDGPU_Func
```tablegen
// Calling convention for leaf functions
def RetCC_AMDGPU_Func : CallingConv<[
  CCIfType<[i1], CCPromoteToType<i32>>,
  CCIfType<[i1, i16], CCIfExtend<CCPromoteToType<i32>>>,
  CCIfType<[i32, f32, i16, f16, v2i16, v2f16, bf16, v2bf16], CCAssignToReg<
    !foreach(i, !range(0, 32), !cast<Register>("VGPR"#i))  // VGPR0-31
  >>,
]>;

def CC_AMDGPU : CallingConv<[
   CCIf<"State.getMachineFunction().getSubtarget<GCNSubtarget>().getGeneration() >= "
          "AMDGPUSubtarget::SOUTHERN_ISLANDS",
        CCDelegateTo<CC_SI_SHADER>>,
   CCIf<"State.getMachineFunction().getSubtarget<GCNSubtarget>().getGeneration() >= "
          "AMDGPUSubtarget::SOUTHERN_ISLANDS && State.getCallingConv() == CallingConv::C",
        CCDelegateTo<CC_AMDGPU_Func>>
]>;

def CC_AMDGPU_CS_CHAIN : CallingConv<[
  CCIfInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(105), !cast<Register>("SGPR"#i))
  >>>,

```
**EN:** This section contains concrete logic for TableGen record RetCC_AMDGPU_Func. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `RetCC_AMDGPU_Func`, `CC_AMDGPU`, `CC_AMDGPU_CS_CHAIN`.
**CN:** 本节包含与 TableGen record RetCC_AMDGPU_Func 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`RetCC_AMDGPU_Func`, `CC_AMDGPU`, `CC_AMDGPU_CS_CHAIN`。

### Lines 175-196: Defines TableGen class RegMask
```tablegen
  CCIfNotInReg<CCIfType<[f32, i32, f16, i16, v2i16, v2f16, bf16, v2bf16] , CCAssignToReg<
    !foreach(i, !range(8, 255), !cast<Register>("VGPR"#i))
  >>>
]>;

// Trivial class to denote when a def is used only to get a RegMask, i.e.
// SaveList is ignored and the def is not used as part of any calling
// convention.
class RegMask<dag mask> : CalleeSavedRegs<mask>;

def AMDGPU_AllVGPRs : RegMask<
  (sequence "VGPR%u", 0, 255)
>;

def AMDGPU_AllAGPRs : RegMask<
  (sequence "AGPR%u", 0, 255)
>;

def AMDGPU_AllVectorRegs : RegMask<
  (add AMDGPU_AllVGPRs, AMDGPU_AllAGPRs)
>;

```
**EN:** This section contains concrete logic for TableGen class RegMask. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `RegMask`, `AMDGPU_AllVGPRs`, `AMDGPU_AllAGPRs`.
**CN:** 本节包含与 TableGen class RegMask 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`RegMask`, `AMDGPU_AllVGPRs`, `AMDGPU_AllAGPRs`。

### Lines 197-199: Defines TableGen record AMDGPU_AllAllocatableSRegs
```tablegen
def AMDGPU_AllAllocatableSRegs : RegMask<
  (add (sequence "SGPR%u", 0, 105), VCC_LO, VCC_HI)
>;
```
**EN:** This section contains concrete logic for TableGen record AMDGPU_AllAllocatableSRegs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU_AllAllocatableSRegs`.
**CN:** 本节包含与 TableGen record AMDGPU_AllAllocatableSRegs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU_AllAllocatableSRegs`。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `CCIfNotInReg`, `CCIfExtend`, `CCIfOrigTypeShaderCCIsSGPR`, `CC_SI_Gfx`, `RetCC_SI_Gfx`, `CC_SI_SHADER`
- **Main themes / 核心主题**: register management / 寄存器管理; LLVM pass integration / LLVM Pass 集成
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
