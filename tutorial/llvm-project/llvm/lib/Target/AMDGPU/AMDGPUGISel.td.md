# AMDGPUGISel.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUGISel.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines AMDGPUGISel records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 AMDGPUGISel 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, comments, and TableGen overview
```tablegen
//===-- AMDGPUGIsel.td - AMDGPU GlobalISel Patterns---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This files contains patterns that should only be used by GlobalISel.  For
// example patterns for V_* instructions that have S_* equivalents.
// SelectionDAG does not support selecting V_* instructions.
//===----------------------------------------------------------------------===//

include "AMDGPU.td"
include "AMDGPUCombine.td"

def sd_vsrc0 : ComplexPattern<i32, 1, "">;
def gi_vsrc0 :
    GIComplexOperandMatcher<s32, "selectVSRC0">,
    GIComplexPatternEquiv<sd_vsrc0>;

def sd_vcsrc : ComplexPattern<i32, 1, "">;
def gi_vcsrc :
    GIComplexOperandMatcher<s32, "selectVCSRC">,
    GIComplexPatternEquiv<sd_vcsrc>;

def gi_vop3mods0 :
    GIComplexOperandMatcher<s32, "selectVOP3Mods0">,
    GIComplexPatternEquiv<VOP3Mods0>;

def gi_vop3mods :
    GIComplexOperandMatcher<s32, "selectVOP3Mods">,
    GIComplexPatternEquiv<VOP3Mods>;

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `sd_vsrc0`, `gi_vsrc0`, `sd_vcsrc`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`sd_vsrc0`, `gi_vsrc0`, `sd_vcsrc`。

### Lines 34-65: Defines TableGen record gi_vop3modsnoncanonicalizing
```tablegen
def gi_vop3modsnoncanonicalizing :
    GIComplexOperandMatcher<s32, "selectVOP3ModsNonCanonicalizing">,
    GIComplexPatternEquiv<VOP3ModsNonCanonicalizing>;

def gi_vop3_no_mods :
    GIComplexOperandMatcher<s32, "selectVOP3NoMods">,
    GIComplexPatternEquiv<VOP3NoMods>;

def gi_vop3omods :
    GIComplexOperandMatcher<s32, "selectVOP3OMods">,
    GIComplexPatternEquiv<VOP3OMods>;

def gi_vop3pmods :
    GIComplexOperandMatcher<s32, "selectVOP3PMods">,
    GIComplexPatternEquiv<VOP3PMods>;

def gi_vop3pmodsdot :
    GIComplexOperandMatcher<s32, "selectVOP3PModsDOT">,
    GIComplexPatternEquiv<VOP3PModsDOT>;

def gi_vop3pnomodsdot :
    GIComplexOperandMatcher<s32, "selectVOP3PNoModsDOT">,
    GIComplexPatternEquiv<VOP3PNoModsDOT>;

def gi_vop3pmodsf32 :
    GIComplexOperandMatcher<s32, "selectVOP3PModsF32">,
    GIComplexPatternEquiv<VOP3PModsF32>;

def gi_vop3pnomodsf32 :
    GIComplexOperandMatcher<s32, "selectVOP3PNoModsF32">,
    GIComplexPatternEquiv<VOP3PNoModsF32>;

```
**EN:** This section contains concrete logic for TableGen record gi_vop3modsnoncanonicalizing. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_vop3modsnoncanonicalizing`, `gi_vop3_no_mods`, `gi_vop3omods`.
**CN:** 本节包含与 TableGen record gi_vop3modsnoncanonicalizing 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_vop3modsnoncanonicalizing`, `gi_vop3_no_mods`, `gi_vop3omods`。

### Lines 66-97: Defines TableGen record gi_wmmaopselvop3pmods
```tablegen
def gi_wmmaopselvop3pmods :
    GIComplexOperandMatcher<s32, "selectWMMAOpSelVOP3PMods">,
    GIComplexPatternEquiv<WMMAOpSelVOP3PMods>;

def gi_wmmavisrc :
    GIComplexOperandMatcher<s32, "selectWMMAVISrc">,
    GIComplexPatternEquiv<WMMAVISrc>;

def gi_wmmamods :
    GIComplexOperandMatcher<s32, "selectWMMAModsF32NegAbs">,
    GIComplexPatternEquiv<WMMAModsF32NegAbs>;

def gi_wmmamodsf16Neg :
    GIComplexOperandMatcher<s32, "selectWMMAModsF16Neg">,
    GIComplexPatternEquiv<WMMAModsF16Neg>;

def gi_wmmamodsf16NegAbs :
    GIComplexOperandMatcher<s32, "selectWMMAModsF16NegAbs">,
    GIComplexPatternEquiv<WMMAModsF16NegAbs>;

def gi_swmmacindex8 :
    GIComplexOperandMatcher<s32, "selectSWMMACIndex8">,
    GIComplexPatternEquiv<SWMMACIndex8>;

def gi_swmmacindex16 :
    GIComplexOperandMatcher<s32, "selectSWMMACIndex16">,
    GIComplexPatternEquiv<SWMMACIndex16>;

def gi_swmmacindex32 :
    GIComplexOperandMatcher<s64, "selectSWMMACIndex32">,
    GIComplexPatternEquiv<SWMMACIndex32>;

```
**EN:** This section contains concrete logic for TableGen record gi_wmmaopselvop3pmods. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_wmmaopselvop3pmods`, `gi_wmmavisrc`, `gi_wmmamods`.
**CN:** 本节包含与 TableGen record gi_wmmaopselvop3pmods 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_wmmaopselvop3pmods`, `gi_wmmavisrc`, `gi_wmmamods`。

### Lines 98-130: Defines TableGen record gi_vop3opselmods
```tablegen
def gi_vop3opselmods :
    GIComplexOperandMatcher<s32, "selectVOP3OpSelMods">,
    GIComplexPatternEquiv<VOP3OpSelMods>;

def gi_vinterpmods :
    GIComplexOperandMatcher<s32, "selectVINTERPMods">,
    GIComplexPatternEquiv<VINTERPMods>;

def gi_vinterpmods_hi :
    GIComplexOperandMatcher<s32, "selectVINTERPModsHi">,
    GIComplexPatternEquiv<VINTERPModsHi>;

// FIXME: Why do we have both VOP3OpSel and VOP3OpSelMods?
def gi_vop3opsel :
    GIComplexOperandMatcher<s32, "selectVOP3OpSelMods">,
    GIComplexPatternEquiv<VOP3OpSel>;

def gi_smrd_imm :
    GIComplexOperandMatcher<s64, "selectSmrdImm">,
    GIComplexPatternEquiv<SMRDImm>;

def gi_smrd_imm32 :
    GIComplexOperandMatcher<s64, "selectSmrdImm32">,
    GIComplexPatternEquiv<SMRDImm32>;

def gi_smrd_sgpr :
    GIComplexOperandMatcher<s64, "selectSmrdSgpr">,
    GIComplexPatternEquiv<SMRDSgpr>;

def gi_smrd_sgpr_imm :
    GIComplexOperandMatcher<s64, "selectSmrdSgprImm">,
    GIComplexPatternEquiv<SMRDSgprImm>;

```
**EN:** This section contains concrete logic for TableGen record gi_vop3opselmods. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_vop3opselmods`, `gi_vinterpmods`, `gi_vinterpmods_hi`.
**CN:** 本节包含与 TableGen record gi_vop3opselmods 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_vop3opselmods`, `gi_vinterpmods`, `gi_vinterpmods_hi`。

### Lines 131-163: Defines TableGen record gi_flat_offset
```tablegen
def gi_flat_offset :
    GIComplexOperandMatcher<s64, "selectFlatOffset">,
    GIComplexPatternEquiv<FlatOffset>;
def gi_global_offset :
    GIComplexOperandMatcher<s64, "selectGlobalOffset">,
    GIComplexPatternEquiv<GlobalOffset>;
def gi_global_saddr :
    GIComplexOperandMatcher<s64, "selectGlobalSAddr">,
    GIComplexPatternEquiv<GlobalSAddr>;
def gi_global_saddr_cpol :
    GIComplexOperandMatcher<s64, "selectGlobalSAddrCPol">,
    GIComplexPatternEquiv<GlobalSAddrCPol>;
def gi_global_saddr_cpol_m0 :
    GIComplexOperandMatcher<s64, "selectGlobalSAddrCPolM0">,
    GIComplexPatternEquiv<GlobalSAddrCPolM0>;
def gi_global_saddr_glc :
    GIComplexOperandMatcher<s64, "selectGlobalSAddrGLC">,
    GIComplexPatternEquiv<GlobalSAddrGLC>;
def gi_global_saddr_no_ioffset :
    GIComplexOperandMatcher<s64, "selectGlobalSAddrNoIOffset">,
    GIComplexPatternEquiv<GlobalSAddrNoIOffset>;
def gi_global_saddr_no_ioffset_m0 :
    GIComplexOperandMatcher<s64, "selectGlobalSAddrNoIOffsetM0">,
    GIComplexPatternEquiv<GlobalSAddrNoIOffsetM0>;

def gi_mubuf_scratch_offset :
    GIComplexOperandMatcher<s32, "selectMUBUFScratchOffset">,
    GIComplexPatternEquiv<MUBUFScratchOffset>;

def gi_buf_soffset :
    GIComplexOperandMatcher<s32, "selectBUFSOffset">,
    GIComplexPatternEquiv<BUFSOffset>;

```
**EN:** This section contains concrete logic for TableGen record gi_flat_offset. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_flat_offset`, `gi_global_offset`, `gi_global_saddr`.
**CN:** 本节包含与 TableGen record gi_flat_offset 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_flat_offset`, `gi_global_offset`, `gi_global_saddr`。

### Lines 164-195: Defines TableGen record gi_mubuf_scratch_offen
```tablegen
def gi_mubuf_scratch_offen :
    GIComplexOperandMatcher<s32, "selectMUBUFScratchOffen">,
    GIComplexPatternEquiv<MUBUFScratchOffen>;

def gi_flat_scratch_offset :
    GIComplexOperandMatcher<s32, "selectScratchOffset">,
    GIComplexPatternEquiv<ScratchOffset>;

def gi_flat_scratch_saddr :
    GIComplexOperandMatcher<s32, "selectScratchSAddr">,
    GIComplexPatternEquiv<ScratchSAddr>;

def gi_flat_scratch_svaddr :
    GIComplexOperandMatcher<s32, "selectScratchSVAddr">,
    GIComplexPatternEquiv<ScratchSVAddr>;

def gi_ds_1addr_1offset :
    GIComplexOperandMatcher<s32, "selectDS1Addr1Offset">,
    GIComplexPatternEquiv<DS1Addr1Offset>;

def gi_ds_64bit_4byte_aligned :
    GIComplexOperandMatcher<s64, "selectDS64Bit4ByteAligned">,
    GIComplexPatternEquiv<DS64Bit4ByteAligned>;

def gi_ds_128bit_8byte_aligned :
    GIComplexOperandMatcher<s64, "selectDS128Bit8ByteAligned">,
    GIComplexPatternEquiv<DS128Bit8ByteAligned>;

def gi_mubuf_addr64 :
    GIComplexOperandMatcher<s64, "selectMUBUFAddr64">,
    GIComplexPatternEquiv<MUBUFAddr64>;

```
**EN:** This section contains concrete logic for TableGen record gi_mubuf_scratch_offen. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_mubuf_scratch_offen`, `gi_flat_scratch_offset`, `gi_flat_scratch_saddr`.
**CN:** 本节包含与 TableGen record gi_mubuf_scratch_offen 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_mubuf_scratch_offen`, `gi_flat_scratch_offset`, `gi_flat_scratch_saddr`。

### Lines 196-224: Defines TableGen record gi_mubuf_offset
```tablegen
def gi_mubuf_offset :
    GIComplexOperandMatcher<s64, "selectMUBUFOffset">,
    GIComplexPatternEquiv<MUBUFOffset>;

def gi_smrd_buffer_imm :
    GIComplexOperandMatcher<s64, "selectSMRDBufferImm">,
    GIComplexPatternEquiv<SMRDBufferImm>;

def gi_smrd_buffer_imm32 :
    GIComplexOperandMatcher<s64, "selectSMRDBufferImm32">,
    GIComplexPatternEquiv<SMRDBufferImm32>;

def gi_smrd_buffer_sgpr_imm :
    GIComplexOperandMatcher<s64, "selectSMRDBufferSgprImm">,
    GIComplexPatternEquiv<SMRDBufferSgprImm>;

def gi_vop3_mad_mix_mods :
    GIComplexOperandMatcher<s64, "selectVOP3PMadMixMods">,
    GIComplexPatternEquiv<VOP3PMadMixMods>;

def gi_vop3_mad_mix_mods_ext :
    GIComplexOperandMatcher<s64, "selectVOP3PMadMixModsExt">,
    GIComplexPatternEquiv<VOP3PMadMixModsExt>;

// Separate load nodes are defined to glue m0 initialization in
// SelectionDAG. The GISel selector can just insert m0 initialization
// directly before selecting a glue-less load, so hide this
// distinction.

```
**EN:** This section contains concrete logic for TableGen record gi_mubuf_offset. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_mubuf_offset`, `gi_smrd_buffer_imm`, `gi_smrd_buffer_imm32`.
**CN:** 本节包含与 TableGen record gi_mubuf_offset 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_mubuf_offset`, `gi_smrd_buffer_imm`, `gi_smrd_buffer_imm32`。

### Lines 225-258: Adjusts TableGen properties and predicates
```tablegen
def : GINodeEquiv<G_LOAD, AMDGPUld_glue> {
  let CheckMMOIsNonAtomic = 1;
  let IfSignExtend = G_SEXTLOAD;
  let IfZeroExtend = G_ZEXTLOAD;
}

def : GINodeEquiv<G_STORE, AMDGPUst_glue> {
  let CheckMMOIsNonAtomic = 1;
}

def : GINodeEquiv<G_LOAD, AMDGPUatomic_ld_glue> {
  bit CheckMMOIsAtomic = 1;
  let IfSignExtend = G_SEXTLOAD;
  let IfZeroExtend = G_ZEXTLOAD;
}

def : GINodeEquiv<G_STORE, AMDGPUatomic_st_glue> {
  bit CheckMMOIsAtomic = 1;
}


def : GINodeEquiv<G_ATOMIC_CMPXCHG, atomic_cmp_swap_glue>;
def : GINodeEquiv<G_ATOMICRMW_XCHG, atomic_swap_glue>;
def : GINodeEquiv<G_ATOMICRMW_ADD, atomic_load_add_glue>;
def : GINodeEquiv<G_ATOMICRMW_SUB, atomic_load_sub_glue>;
def : GINodeEquiv<G_ATOMICRMW_AND, atomic_load_and_glue>;
def : GINodeEquiv<G_ATOMICRMW_OR, atomic_load_or_glue>;
def : GINodeEquiv<G_ATOMICRMW_XOR, atomic_load_xor_glue>;
def : GINodeEquiv<G_ATOMICRMW_MIN, atomic_load_min_glue>;
def : GINodeEquiv<G_ATOMICRMW_MAX, atomic_load_max_glue>;
def : GINodeEquiv<G_ATOMICRMW_UMIN, atomic_load_umin_glue>;
def : GINodeEquiv<G_ATOMICRMW_UMAX, atomic_load_umax_glue>;
def : GINodeEquiv<G_ATOMICRMW_FADD, atomic_load_fadd_glue>;

```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 259-292: TableGen records and backend metadata
```tablegen
def : GINodeEquiv<G_AMDGPU_FFBH_U32, AMDGPUffbh_u32_impl>;
def : GINodeEquiv<G_AMDGPU_FFBL_B32, AMDGPUffbl_b32_impl>;
def : GINodeEquiv<G_AMDGPU_FMIN_LEGACY, AMDGPUfmin_legacy>;
def : GINodeEquiv<G_AMDGPU_FMAX_LEGACY, AMDGPUfmax_legacy>;
def : GINodeEquiv<G_AMDGPU_RCP_IFLAG, AMDGPUrcp_iflag>;

def : GINodeEquiv<G_AMDGPU_CVT_F32_UBYTE0, AMDGPUcvt_f32_ubyte0>;
def : GINodeEquiv<G_AMDGPU_CVT_F32_UBYTE1, AMDGPUcvt_f32_ubyte1>;
def : GINodeEquiv<G_AMDGPU_CVT_F32_UBYTE2, AMDGPUcvt_f32_ubyte2>;
def : GINodeEquiv<G_AMDGPU_CVT_F32_UBYTE3, AMDGPUcvt_f32_ubyte3>;

def : GINodeEquiv<G_AMDGPU_CVT_PK_I16_I32, AMDGPUpk_i16_i32_impl>;
def : GINodeEquiv<G_AMDGPU_SMED3, AMDGPUsmed3>;
def : GINodeEquiv<G_AMDGPU_UMED3, AMDGPUumed3>;
def : GINodeEquiv<G_AMDGPU_FMED3, AMDGPUfmed3_impl>;
def : GINodeEquiv<G_AMDGPU_CLAMP, AMDGPUclamp>;

def : GINodeEquiv<G_AMDGPU_ATOMIC_CMPXCHG, AMDGPUatomic_cmp_swap>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD, SIbuffer_load>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_USHORT, SIbuffer_load_ushort>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_UBYTE, SIbuffer_load_ubyte>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_SSHORT, SIbuffer_load_short>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_SBYTE, SIbuffer_load_byte>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_TFE, SIbuffer_load_tfe>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_USHORT_TFE, SIbuffer_load_ushort_tfe>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_UBYTE_TFE, SIbuffer_load_ubyte_tfe>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_SSHORT_TFE, SIbuffer_load_short_tfe>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_SBYTE_TFE, SIbuffer_load_byte_tfe>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_FORMAT, SIbuffer_load_format>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_FORMAT_TFE, SIbuffer_load_format_tfe>;
def : GINodeEquiv<G_AMDGPU_BUFFER_LOAD_FORMAT_D16, SIbuffer_load_format_d16>;
def : GINodeEquiv<G_AMDGPU_TBUFFER_LOAD_FORMAT, SItbuffer_load>;
def : GINodeEquiv<G_AMDGPU_TBUFFER_LOAD_FORMAT_D16, SItbuffer_load_d16>;
def : GINodeEquiv<G_AMDGPU_BUFFER_STORE, SIbuffer_store>;
```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 293-326: TableGen records and backend metadata
```tablegen
def : GINodeEquiv<G_AMDGPU_BUFFER_STORE_SHORT, SIbuffer_store_short>;
def : GINodeEquiv<G_AMDGPU_BUFFER_STORE_BYTE, SIbuffer_store_byte>;
def : GINodeEquiv<G_AMDGPU_BUFFER_STORE_FORMAT, SIbuffer_store_format>;
def : GINodeEquiv<G_AMDGPU_BUFFER_STORE_FORMAT_D16, SIbuffer_store_format_d16>;
def : GINodeEquiv<G_AMDGPU_TBUFFER_STORE_FORMAT, SItbuffer_store>;
def : GINodeEquiv<G_AMDGPU_TBUFFER_STORE_FORMAT_D16, SItbuffer_store_d16>;

// FIXME: Check MMO is atomic
def : GINodeEquiv<G_ATOMICRMW_UINC_WRAP, atomic_load_uinc_wrap_glue>;
def : GINodeEquiv<G_ATOMICRMW_UDEC_WRAP, atomic_load_udec_wrap_glue>;
def : GINodeEquiv<G_ATOMICRMW_USUB_COND, atomic_load_usub_cond_glue>;
def : GINodeEquiv<G_ATOMICRMW_USUB_SAT, atomic_load_usub_sat_glue>;
def : GINodeEquiv<G_ATOMICRMW_FMIN, atomic_load_fmin_glue>;
def : GINodeEquiv<G_ATOMICRMW_FMAX, atomic_load_fmax_glue>;

def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_SWAP, SIbuffer_atomic_swap>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_ADD, SIbuffer_atomic_add>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_SUB, SIbuffer_atomic_sub>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_SMIN, SIbuffer_atomic_smin>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_UMIN, SIbuffer_atomic_umin>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_SMAX, SIbuffer_atomic_smax>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_UMAX, SIbuffer_atomic_umax>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_AND, SIbuffer_atomic_and>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_OR, SIbuffer_atomic_or>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_XOR, SIbuffer_atomic_xor>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_INC, SIbuffer_atomic_inc>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_DEC, SIbuffer_atomic_dec>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_FADD, SIbuffer_atomic_fadd>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_FMIN, SIbuffer_atomic_fmin>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_FMAX, SIbuffer_atomic_fmax>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_CMPSWAP, SIbuffer_atomic_cmpswap>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_COND_SUB_U32, SIbuffer_atomic_cond_sub_u32>;
def : GINodeEquiv<G_AMDGPU_BUFFER_ATOMIC_SUB_CLAMP_U32, SIbuffer_atomic_csub>;
def : GINodeEquiv<G_AMDGPU_S_BUFFER_LOAD, SIsbuffer_load>;
```
**EN:** This section describes backend metadata in TableGen form so LLVM can generate derived tables, predicates, or target-specific records during the build.
**CN:** 本节以 TableGen 形式描述后端元数据，使 LLVM 能在构建期间生成派生表、谓词或目标专用记录。

### Lines 327-359: Defines TableGen class GISelSop2Pat
```tablegen
def : GINodeEquiv<G_AMDGPU_S_BUFFER_LOAD_SBYTE, SIsbuffer_load_byte>;
def : GINodeEquiv<G_AMDGPU_S_BUFFER_LOAD_UBYTE, SIsbuffer_load_ubyte>;
def : GINodeEquiv<G_AMDGPU_S_BUFFER_LOAD_SSHORT, SIsbuffer_load_short>;
def : GINodeEquiv<G_AMDGPU_S_BUFFER_LOAD_USHORT, SIsbuffer_load_ushort>;
def : GINodeEquiv<G_AMDGPU_S_BUFFER_PREFETCH, SIsbuffer_prefetch>;

def : GINodeEquiv<G_AMDGPU_LOAD_D16_LO, SIload_d16_lo>;
def : GINodeEquiv<G_AMDGPU_LOAD_D16_LO_U8, SIload_d16_lo_u8>;
def : GINodeEquiv<G_AMDGPU_LOAD_D16_LO_I8, SIload_d16_lo_i8>;
def : GINodeEquiv<G_AMDGPU_LOAD_D16_HI, SIload_d16_hi>;
def : GINodeEquiv<G_AMDGPU_LOAD_D16_HI_U8, SIload_d16_hi_u8>;
def : GINodeEquiv<G_AMDGPU_LOAD_D16_HI_I8, SIload_d16_hi_i8>;

def : GINodeEquiv<G_AMDGPU_WHOLE_WAVE_FUNC_SETUP, AMDGPUwhole_wave_setup>;
// G_AMDGPU_WHOLE_WAVE_FUNC_RETURN is simpler than AMDGPUwhole_wave_return,
// so we don't mark it as equivalent.

def : GINodeEquiv<G_AMDGPU_SPONENTRY, sponentry>;

def : GINodeEquiv<G_AMDGPU_FLAT_LOAD_MONITOR, AMDGPUflat_load_monitor>;
def : GINodeEquiv<G_AMDGPU_GLOBAL_LOAD_MONITOR, AMDGPUglobal_load_monitor>;


class GISelSop2Pat <
  SDPatternOperator node,
  Instruction inst,
  ValueType dst_vt,
  ValueType src0_vt = dst_vt, ValueType src1_vt = src0_vt>   : GCNPat <

  (dst_vt (node (src0_vt SReg_32:$src0), (src1_vt SReg_32:$src1))),
  (inst src0_vt:$src0, src1_vt:$src1)
>;

```
**EN:** This section contains concrete logic for TableGen class GISelSop2Pat. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GISelSop2Pat`.
**CN:** 本节包含与 TableGen class GISelSop2Pat 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GISelSop2Pat`。

### Lines 360-389: Defines TableGen class GISelVop2Pat
```tablegen
class GISelVop2Pat <
  SDPatternOperator node,
  Instruction inst,
  ValueType dst_vt,
  ValueType src0_vt = dst_vt, ValueType src1_vt = src0_vt>   : GCNPat <

  (dst_vt (node (src0_vt (sd_vsrc0 src0_vt:$src0)), (src1_vt VGPR_32:$src1))),
  (inst src0_vt:$src0, src1_vt:$src1)
>;

class GISelVop2CommutePat <
  SDPatternOperator node,
  Instruction inst,
  ValueType dst_vt,
  ValueType src0_vt = dst_vt, ValueType src1_vt = src0_vt>   : GCNPat <

  (dst_vt (node (src1_vt VGPR_32:$src1), (src0_vt (sd_vsrc0 src0_vt:$src0)))),
  (inst src0_vt:$src0, src1_vt:$src1)
>;

class GISelVop3Pat2 <
  SDPatternOperator node,
  Instruction inst,
  ValueType dst_vt,
  ValueType src0_vt = dst_vt, ValueType src1_vt = src0_vt>   : GCNPat <

  (dst_vt (node (src0_vt (sd_vcsrc src0_vt:$src0)), (src1_vt (sd_vcsrc src1_vt:$src1)))),
  (inst src0_vt:$src0, src1_vt:$src1)
>;

```
**EN:** This section contains concrete logic for TableGen class GISelVop2Pat. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GISelVop2Pat`, `GISelVop2CommutePat`, `GISelVop3Pat2`.
**CN:** 本节包含与 TableGen class GISelVop2Pat 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GISelVop2Pat`, `GISelVop2CommutePat`, `GISelVop3Pat2`。

### Lines 390-417: Defines TableGen class GISelVop3Pat2CommutePat
```tablegen
class GISelVop3Pat2CommutePat <
  SDPatternOperator node,
  Instruction inst,
  ValueType dst_vt,
  ValueType src0_vt = dst_vt, ValueType src1_vt = src0_vt>   : GCNPat <

  (dst_vt (node (src0_vt (sd_vcsrc src0_vt:$src0)), (src1_vt (sd_vcsrc src1_vt:$src1)))),
  (inst src0_vt:$src1, src1_vt:$src0)
>;

class GISelVop3Pat2ModsPat <
  SDPatternOperator node,
  Instruction inst,
  ValueType dst_vt,
  ValueType src0_vt = dst_vt, ValueType src1_vt = src0_vt> : GCNPat <

  (dst_vt (node (src0_vt (VOP3Mods0 src0_vt:$src0, i32:$src0_modifiers, i1:$clamp, i32:$omods)),
                (src1_vt (VOP3Mods src1_vt:$src1, i32:$src1_modifiers)))),
  (inst i32:$src0_modifiers, src0_vt:$src0,
        i32:$src1_modifiers, src1_vt:$src1, $clamp, $omods)
>;

multiclass GISelVop2IntrPat <
  SDPatternOperator node, Instruction inst,
  ValueType dst_vt, ValueType src_vt = dst_vt> {

  def : GISelVop2Pat <node, inst, dst_vt, src_vt>;

```
**EN:** This section contains concrete logic for TableGen class GISelVop3Pat2CommutePat. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GISelVop3Pat2CommutePat`, `GISelVop3Pat2ModsPat`, `GISelVop2IntrPat`.
**CN:** 本节包含与 TableGen class GISelVop3Pat2CommutePat 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GISelVop3Pat2CommutePat`, `GISelVop3Pat2ModsPat`, `GISelVop2IntrPat`。

### Lines 418-450: Defines TableGen record gi_as_i32timm
```tablegen
  // FIXME: Intrinsics aren't marked as commutable, so we need to add an explicit
  // pattern to handle commuting.  This is another reason why legalizing to a
  // generic machine instruction may be better that matching the intrinsic
  // directly.
  def : GISelVop2CommutePat <node, inst, dst_vt, src_vt>;
}

// Since GlobalISel is more flexible then SelectionDAG, I think we can get
// away with adding patterns for integer types and not legalizing all
// loads and stores to vector types.  This should help simplify the load/store
// legalization.
foreach Ty = [i64, p0, p1, p4] in {
  defm : SMRD_Pattern <"S_LOAD_DWORDX2",  Ty>;
}

def gi_as_i32timm : GICustomOperandRenderer<"renderTruncTImm">,
  GISDNodeXFormEquiv<as_i32timm>;

def gi_as_i16timm : GICustomOperandRenderer<"renderTruncTImm">,
  GISDNodeXFormEquiv<as_i16timm>;

def gi_as_i8timm : GICustomOperandRenderer<"renderTruncTImm">,
  GISDNodeXFormEquiv<as_i8timm>;

def gi_as_i1timm : GICustomOperandRenderer<"renderTruncTImm">,
  GISDNodeXFormEquiv<as_i1timm>;

def gi_as_i1timm_zext : GICustomOperandRenderer<"renderZextBoolTImm">,
  GISDNodeXFormEquiv<as_i1timm_zext>;

def gi_NegateImm : GICustomOperandRenderer<"renderNegateImm">,
  GISDNodeXFormEquiv<NegateImm>;

```
**EN:** This section contains concrete logic for TableGen record gi_as_i32timm. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_as_i32timm`, `gi_as_i16timm`, `gi_as_i8timm`.
**CN:** 本节包含与 TableGen record gi_as_i32timm 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_as_i32timm`, `gi_as_i16timm`, `gi_as_i8timm`。

### Lines 451-483: Defines TableGen record gi_bitcast_fpimm_to_i32
```tablegen
def gi_bitcast_fpimm_to_i32 : GICustomOperandRenderer<"renderBitcastFPImm32">,
  GISDNodeXFormEquiv<bitcast_fpimm_to_i32>;
def gi_bitcast_fpimm_to_i64 : GICustomOperandRenderer<"renderBitcastFPImm64">,
  GISDNodeXFormEquiv<bitcast_fpimm_to_i64>;

def gi_IMMCountTrailingOnes : GICustomOperandRenderer<"renderCountTrailingOnesImm">,
  GISDNodeXFormEquiv<IMMCountTrailingOnes>;

def gi_extract_cpol : GICustomOperandRenderer<"renderExtractCPol">,
  GISDNodeXFormEquiv<extract_cpol>;

def gi_extract_swz : GICustomOperandRenderer<"renderExtractSWZ">,
  GISDNodeXFormEquiv<extract_swz>;

def gi_extract_cpol_set_glc : GICustomOperandRenderer<"renderExtractCpolSetGLC">,
  GISDNodeXFormEquiv<extract_cpol_set_glc>;

def gi_frameindex_to_targetframeindex : GICustomOperandRenderer<"renderFrameIndex">,
  GISDNodeXFormEquiv<frameindex_to_targetframeindex>;

def gi_fp_pow2_to_exponent : GICustomOperandRenderer<"renderFPPow2ToExponent">,
  GISDNodeXFormEquiv<FPPow2ToExponentXForm>;

def gi_as_hw_round_mode : GICustomOperandRenderer<"renderRoundMode">,
  GISDNodeXFormEquiv<as_hw_round_mode>;

def gi_VOP3PModsNeg : GICustomOperandRenderer<"renderVOP3PModsNeg">,
  GISDNodeXFormEquiv<VOP3PModsNeg>;
def gi_VOP3PModsNegs : GICustomOperandRenderer<"renderVOP3PModsNegs">,
  GISDNodeXFormEquiv<VOP3PModsNegs>;
def gi_VOP3PModsNegAbs : GICustomOperandRenderer<"renderVOP3PModsNegAbs">,
  GISDNodeXFormEquiv<VOP3PModsNegAbs>;

```
**EN:** This section contains concrete logic for TableGen record gi_bitcast_fpimm_to_i32. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_bitcast_fpimm_to_i32`, `gi_bitcast_fpimm_to_i64`, `gi_IMMCountTrailingOnes`.
**CN:** 本节包含与 TableGen record gi_bitcast_fpimm_to_i32 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_bitcast_fpimm_to_i32`, `gi_bitcast_fpimm_to_i64`, `gi_IMMCountTrailingOnes`。

### Lines 484-488: Defines TableGen record gi_prefetch_loc
```tablegen
def gi_prefetch_loc : GICustomOperandRenderer<"renderPrefetchLoc">,
  GISDNodeXFormEquiv<PrefetchLoc>;

def gi_MFMALdScaleModifierOp : GICustomOperandRenderer<"renderScaledMAIIntrinsicOperand">,
  GISDNodeXFormEquiv<MFMALdScaleXForm>;
```
**EN:** This section contains concrete logic for TableGen record gi_prefetch_loc. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `gi_prefetch_loc`, `gi_MFMALdScaleModifierOp`.
**CN:** 本节包含与 TableGen record gi_prefetch_loc 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`gi_prefetch_loc`, `gi_MFMALdScaleModifierOp`。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `sd_vsrc0`, `gi_vsrc0`, `sd_vcsrc`, `gi_vcsrc`, `gi_vop3mods0`, `gi_vop3mods`
- **Main themes / 核心主题**: instruction semantics / 指令语义; SelectionDAG processing / SelectionDAG 处理; GlobalISel support / GlobalISel 支持
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- `AMDGPU.td`
- `AMDGPUCombine.td`
