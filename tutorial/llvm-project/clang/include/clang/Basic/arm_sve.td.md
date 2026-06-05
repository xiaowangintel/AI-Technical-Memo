# arm_sve.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/arm_sve.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: ARM SVE compiler interface.
- **Purpose (CN)**: 声明与 `arm_sve` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 2538

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````tablegen
//===--- arm_sve.td - ARM SVE compiler interface ------------------------===//
//
//  Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
//  See https://llvm.org/LICENSE.txt for license information.
//  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file defines the TableGen definitions from which the ARM SVE header
//  file will be generated.  See:
//
//      https://developer.arm.com/architectures/system-architectures/software-standards/acle
//
//===----------------------------------------------------------------------===//

include "arm_sve_sme_incl.td"

////////////////////////////////////////////////////////////////////////////////
// Loads

// Load one vector (scalar base)
def SVLD1   : MInst<"svld1[_{2}]", "dPc", "csilUcUsUiUlhfdbm", [IsLoad, VerifyRuntimeMode],               MemEltTyDefault, "aarch64_sve_ld1">;
def SVLD1SB : MInst<"svld1sb_{d}", "dPS", "silUsUiUl",         [IsLoad, VerifyRuntimeMode],               MemEltTyInt8,    "aarch64_sve_ld1">;
def SVLD1UB : MInst<"svld1ub_{d}", "dPW", "silUsUiUl",         [IsLoad, IsZExtReturn, VerifyRuntimeMode], MemEltTyInt8,    "aarch64_sve_ld1">;
def SVLD1SH : MInst<"svld1sh_{d}", "dPT", "ilUiUl",            [IsLoad, VerifyRuntimeMode],               MemEltTyInt16,   "aarch64_sve_ld1">;
def SVLD1UH : MInst<"svld1uh_{d}", "dPX", "ilUiUl",            [IsLoad, IsZExtReturn, VerifyRuntimeMode], MemEltTyInt16,   "aarch64_sve_ld1">;
def SVLD1SW : MInst<"svld1sw_{d}", "dPU", "lUl",               [IsLoad, VerifyRuntimeMode],               MemEltTyInt32,   "aarch64_sve_ld1">;
def SVLD1UW : MInst<"svld1uw_{d}", "dPY", "lUl",               [IsLoad, IsZExtReturn, VerifyRuntimeMode], MemEltTyInt32,   "aarch64_sve_ld1">;
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the TableGen definitions from which the ARM SVE header`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the TableGen definitions from which the ARM SVE header`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `file will be generated. See:`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`file will be generated. See:`。
- **L11 EN**: Separator comment used for visual grouping.
  **L11 CN**: 用于视觉分组的分隔注释。
- **L12 EN**: Comment explains nearby logic, constraints, or intent: `https://developer.arm.com/architectures/system-architectures/software-standards/acle`.
  **L12 CN**: 注释解释附近代码的逻辑、约束或设计意图：`https://developer.arm.com/architectures/system-architectures/software-standards/acle`。
- **L13 EN**: Separator comment used for visual grouping.
  **L13 CN**: 用于视觉分组的分隔注释。
- **L14 EN**: Banner comment marking a file or section boundary.
  **L14 CN**: 横幅注释，用于标记文件或章节边界。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Includes TableGen file `"arm_sve_sme_incl.td"` so later records can reuse shared definitions.
  **L16 CN**: 引入 TableGen 文件 `"arm_sve_sme_incl.td"`，以便后续记录复用共享定义。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Separator comment used for visual grouping.
  **L18 CN**: 用于视觉分组的分隔注释。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Loads`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Loads`。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L21 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (scalar base)`.
  **L21 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (scalar base)`。
- **L22 EN**: Declares TableGen def record `SVLD1`.
  **L22 CN**: 声明 TableGen def 记录 `SVLD1`。
- **L23 EN**: Declares TableGen def record `SVLD1SB`.
  **L23 CN**: 声明 TableGen def 记录 `SVLD1SB`。
- **L24 EN**: Declares TableGen def record `SVLD1UB`.
  **L24 CN**: 声明 TableGen def 记录 `SVLD1UB`。
- **L25 EN**: Declares TableGen def record `SVLD1SH`.
  **L25 CN**: 声明 TableGen def 记录 `SVLD1SH`。
- **L26 EN**: Declares TableGen def record `SVLD1UH`.
  **L26 CN**: 声明 TableGen def 记录 `SVLD1UH`。
- **L27 EN**: Declares TableGen def record `SVLD1SW`.
  **L27 CN**: 声明 TableGen def 记录 `SVLD1SW`。
- **L28 EN**: Declares TableGen def record `SVLD1UW`.
  **L28 CN**: 声明 TableGen def 记录 `SVLD1UW`。

### Lines 29-56

````tablegen

// Load one vector (scalar base, VL displacement)
def SVLD1_VNUM   : MInst<"svld1_vnum[_{2}]", "dPcl", "csilUcUsUiUlhfdbm", [IsLoad, VerifyRuntimeMode],               MemEltTyDefault, "aarch64_sve_ld1">;
def SVLD1SB_VNUM : MInst<"svld1sb_vnum_{d}", "dPSl", "silUsUiUl",         [IsLoad, VerifyRuntimeMode],               MemEltTyInt8,    "aarch64_sve_ld1">;
def SVLD1UB_VNUM : MInst<"svld1ub_vnum_{d}", "dPWl", "silUsUiUl",         [IsLoad, IsZExtReturn, VerifyRuntimeMode], MemEltTyInt8,    "aarch64_sve_ld1">;
def SVLD1SH_VNUM : MInst<"svld1sh_vnum_{d}", "dPTl", "ilUiUl",            [IsLoad, VerifyRuntimeMode],               MemEltTyInt16,   "aarch64_sve_ld1">;
def SVLD1UH_VNUM : MInst<"svld1uh_vnum_{d}", "dPXl", "ilUiUl",            [IsLoad, IsZExtReturn, VerifyRuntimeMode], MemEltTyInt16,   "aarch64_sve_ld1">;
def SVLD1SW_VNUM : MInst<"svld1sw_vnum_{d}", "dPUl", "lUl",               [IsLoad, VerifyRuntimeMode],               MemEltTyInt32,   "aarch64_sve_ld1">;
def SVLD1UW_VNUM : MInst<"svld1uw_vnum_{d}", "dPYl", "lUl",               [IsLoad, IsZExtReturn, VerifyRuntimeMode], MemEltTyInt32,   "aarch64_sve_ld1">;

let SMETargetGuard = InvalidMode in {
// Load one vector (vector base)
def SVLD1_GATHER_BASES_U   : MInst<"svld1_gather[_{2}base]_{d}",   "dPu", "ilUiUlfd", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1SB_GATHER_BASES_U : MInst<"svld1sb_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad],               MemEltTyInt8,    "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1UB_GATHER_BASES_U : MInst<"svld1ub_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1SH_GATHER_BASES_U : MInst<"svld1sh_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1UH_GATHER_BASES_U : MInst<"svld1uh_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1SW_GATHER_BASES_U : MInst<"svld1sw_gather[_{2}base]_{d}", "dPu", "lUl",      [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1UW_GATHER_BASES_U : MInst<"svld1uw_gather[_{2}base]_{d}", "dPu", "lUl",      [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ld1_gather_scalar_offset">;

// Load one vector (scalar base, signed vector offset in bytes)
def SVLD1_GATHER_64B_OFFSETS_S   : MInst<"svld1_gather_[{3}]offset[_{d}]", "dPcx", "lUld", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ld1_gather">;
def SVLD1SB_GATHER_64B_OFFSETS_S : MInst<"svld1sb_gather_[{3}]offset_{d}", "dPSx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ld1_gather">;
def SVLD1UB_GATHER_64B_OFFSETS_S : MInst<"svld1ub_gather_[{3}]offset_{d}", "dPWx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ld1_gather">;
def SVLD1SH_GATHER_64B_OFFSETS_S : MInst<"svld1sh_gather_[{3}]offset_{d}", "dPTx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ld1_gather">;
def SVLD1UH_GATHER_64B_OFFSETS_S : MInst<"svld1uh_gather_[{3}]offset_{d}", "dPXx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather">;
def SVLD1SW_GATHER_64B_OFFSETS_S : MInst<"svld1sw_gather_[{3}]offset_{d}", "dPUx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ld1_gather">;
def SVLD1UW_GATHER_64B_OFFSETS_S : MInst<"svld1uw_gather_[{3}]offset_{d}", "dPYx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ld1_gather">;
````
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (scalar base, VL displacement)`.
  **L30 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (scalar base, VL displacement)`。
- **L31 EN**: Declares TableGen def record `SVLD1_VNUM`.
  **L31 CN**: 声明 TableGen def 记录 `SVLD1_VNUM`。
- **L32 EN**: Declares TableGen def record `SVLD1SB_VNUM`.
  **L32 CN**: 声明 TableGen def 记录 `SVLD1SB_VNUM`。
- **L33 EN**: Declares TableGen def record `SVLD1UB_VNUM`.
  **L33 CN**: 声明 TableGen def 记录 `SVLD1UB_VNUM`。
- **L34 EN**: Declares TableGen def record `SVLD1SH_VNUM`.
  **L34 CN**: 声明 TableGen def 记录 `SVLD1SH_VNUM`。
- **L35 EN**: Declares TableGen def record `SVLD1UH_VNUM`.
  **L35 CN**: 声明 TableGen def 记录 `SVLD1UH_VNUM`。
- **L36 EN**: Declares TableGen def record `SVLD1SW_VNUM`.
  **L36 CN**: 声明 TableGen def 记录 `SVLD1SW_VNUM`。
- **L37 EN**: Declares TableGen def record `SVLD1UW_VNUM`.
  **L37 CN**: 声明 TableGen def 记录 `SVLD1UW_VNUM`。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L39 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L39 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L40 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (vector base)`.
  **L40 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (vector base)`。
- **L41 EN**: Declares TableGen def record `SVLD1_GATHER_BASES_U`.
  **L41 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_BASES_U`。
- **L42 EN**: Declares TableGen def record `SVLD1SB_GATHER_BASES_U`.
  **L42 CN**: 声明 TableGen def 记录 `SVLD1SB_GATHER_BASES_U`。
- **L43 EN**: Declares TableGen def record `SVLD1UB_GATHER_BASES_U`.
  **L43 CN**: 声明 TableGen def 记录 `SVLD1UB_GATHER_BASES_U`。
- **L44 EN**: Declares TableGen def record `SVLD1SH_GATHER_BASES_U`.
  **L44 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_BASES_U`。
- **L45 EN**: Declares TableGen def record `SVLD1UH_GATHER_BASES_U`.
  **L45 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_BASES_U`。
- **L46 EN**: Declares TableGen def record `SVLD1SW_GATHER_BASES_U`.
  **L46 CN**: 声明 TableGen def 记录 `SVLD1SW_GATHER_BASES_U`。
- **L47 EN**: Declares TableGen def record `SVLD1UW_GATHER_BASES_U`.
  **L47 CN**: 声明 TableGen def 记录 `SVLD1UW_GATHER_BASES_U`。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L49 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (scalar base, signed vector offset in bytes)`.
  **L49 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (scalar base, signed vector offset in bytes)`。
- **L50 EN**: Declares TableGen def record `SVLD1_GATHER_64B_OFFSETS_S`.
  **L50 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_64B_OFFSETS_S`。
- **L51 EN**: Declares TableGen def record `SVLD1SB_GATHER_64B_OFFSETS_S`.
  **L51 CN**: 声明 TableGen def 记录 `SVLD1SB_GATHER_64B_OFFSETS_S`。
- **L52 EN**: Declares TableGen def record `SVLD1UB_GATHER_64B_OFFSETS_S`.
  **L52 CN**: 声明 TableGen def 记录 `SVLD1UB_GATHER_64B_OFFSETS_S`。
- **L53 EN**: Declares TableGen def record `SVLD1SH_GATHER_64B_OFFSETS_S`.
  **L53 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_64B_OFFSETS_S`。
- **L54 EN**: Declares TableGen def record `SVLD1UH_GATHER_64B_OFFSETS_S`.
  **L54 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_64B_OFFSETS_S`。
- **L55 EN**: Declares TableGen def record `SVLD1SW_GATHER_64B_OFFSETS_S`.
  **L55 CN**: 声明 TableGen def 记录 `SVLD1SW_GATHER_64B_OFFSETS_S`。
- **L56 EN**: Declares TableGen def record `SVLD1UW_GATHER_64B_OFFSETS_S`.
  **L56 CN**: 声明 TableGen def 记录 `SVLD1UW_GATHER_64B_OFFSETS_S`。

### Lines 57-84

````tablegen

def SVLD1_GATHER_32B_OFFSETS_S   : MInst<"svld1_gather_[{3}]offset[_{d}]", "dPcx", "iUif", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ld1_gather_sxtw">;
def SVLD1SB_GATHER_32B_OFFSETS_S : MInst<"svld1sb_gather_[{3}]offset_{d}", "dPSx", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ld1_gather_sxtw">;
def SVLD1UB_GATHER_32B_OFFSETS_S : MInst<"svld1ub_gather_[{3}]offset_{d}", "dPWx", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ld1_gather_sxtw">;
def SVLD1SH_GATHER_32B_OFFSETS_S : MInst<"svld1sh_gather_[{3}]offset_{d}", "dPTx", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ld1_gather_sxtw">;
def SVLD1UH_GATHER_32B_OFFSETS_S : MInst<"svld1uh_gather_[{3}]offset_{d}", "dPXx", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_sxtw">;

// Load one vector (scalar base, unsigned vector offset in bytes)
def SVLD1_GATHER_64B_OFFSETS_U   : MInst<"svld1_gather_[{3}]offset[_{d}]", "dPcu", "lUld", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ld1_gather">;
def SVLD1SB_GATHER_64B_OFFSETS_U : MInst<"svld1sb_gather_[{3}]offset_{d}", "dPSu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ld1_gather">;
def SVLD1UB_GATHER_64B_OFFSETS_U : MInst<"svld1ub_gather_[{3}]offset_{d}", "dPWu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ld1_gather">;
def SVLD1SH_GATHER_64B_OFFSETS_U : MInst<"svld1sh_gather_[{3}]offset_{d}", "dPTu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ld1_gather">;
def SVLD1UH_GATHER_64B_OFFSETS_U : MInst<"svld1uh_gather_[{3}]offset_{d}", "dPXu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather">;
def SVLD1SW_GATHER_64B_OFFSETS_U : MInst<"svld1sw_gather_[{3}]offset_{d}", "dPUu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ld1_gather">;
def SVLD1UW_GATHER_64B_OFFSETS_U : MInst<"svld1uw_gather_[{3}]offset_{d}", "dPYu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ld1_gather">;

def SVLD1_GATHER_32B_OFFSETS_U   : MInst<"svld1_gather_[{3}]offset[_{d}]", "dPcu", "iUif", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ld1_gather_uxtw">;
def SVLD1SB_GATHER_32B_OFFSETS_U : MInst<"svld1sb_gather_[{3}]offset_{d}", "dPSu", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ld1_gather_uxtw">;
def SVLD1UB_GATHER_32B_OFFSETS_U : MInst<"svld1ub_gather_[{3}]offset_{d}", "dPWu", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ld1_gather_uxtw">;
def SVLD1SH_GATHER_32B_OFFSETS_U : MInst<"svld1sh_gather_[{3}]offset_{d}", "dPTu", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ld1_gather_uxtw">;
def SVLD1UH_GATHER_32B_OFFSETS_U : MInst<"svld1uh_gather_[{3}]offset_{d}", "dPXu", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_uxtw">;

// Load one vector (vector base, signed scalar offset in bytes)
def SVLD1_GATHER_OFFSET_S   : MInst<"svld1_gather[_{2}base]_offset_{d}",   "dPul", "ilUiUlfd", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1SB_GATHER_OFFSET_S : MInst<"svld1sb_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1UB_GATHER_OFFSET_S : MInst<"svld1ub_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1SH_GATHER_OFFSET_S : MInst<"svld1sh_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1UH_GATHER_OFFSET_S : MInst<"svld1uh_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_scalar_offset">;
````
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L58 EN**: Declares TableGen def record `SVLD1_GATHER_32B_OFFSETS_S`.
  **L58 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_32B_OFFSETS_S`。
- **L59 EN**: Declares TableGen def record `SVLD1SB_GATHER_32B_OFFSETS_S`.
  **L59 CN**: 声明 TableGen def 记录 `SVLD1SB_GATHER_32B_OFFSETS_S`。
- **L60 EN**: Declares TableGen def record `SVLD1UB_GATHER_32B_OFFSETS_S`.
  **L60 CN**: 声明 TableGen def 记录 `SVLD1UB_GATHER_32B_OFFSETS_S`。
- **L61 EN**: Declares TableGen def record `SVLD1SH_GATHER_32B_OFFSETS_S`.
  **L61 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_32B_OFFSETS_S`。
- **L62 EN**: Declares TableGen def record `SVLD1UH_GATHER_32B_OFFSETS_S`.
  **L62 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_32B_OFFSETS_S`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (scalar base, unsigned vector offset in bytes)`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (scalar base, unsigned vector offset in bytes)`。
- **L65 EN**: Declares TableGen def record `SVLD1_GATHER_64B_OFFSETS_U`.
  **L65 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_64B_OFFSETS_U`。
- **L66 EN**: Declares TableGen def record `SVLD1SB_GATHER_64B_OFFSETS_U`.
  **L66 CN**: 声明 TableGen def 记录 `SVLD1SB_GATHER_64B_OFFSETS_U`。
- **L67 EN**: Declares TableGen def record `SVLD1UB_GATHER_64B_OFFSETS_U`.
  **L67 CN**: 声明 TableGen def 记录 `SVLD1UB_GATHER_64B_OFFSETS_U`。
- **L68 EN**: Declares TableGen def record `SVLD1SH_GATHER_64B_OFFSETS_U`.
  **L68 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_64B_OFFSETS_U`。
- **L69 EN**: Declares TableGen def record `SVLD1UH_GATHER_64B_OFFSETS_U`.
  **L69 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_64B_OFFSETS_U`。
- **L70 EN**: Declares TableGen def record `SVLD1SW_GATHER_64B_OFFSETS_U`.
  **L70 CN**: 声明 TableGen def 记录 `SVLD1SW_GATHER_64B_OFFSETS_U`。
- **L71 EN**: Declares TableGen def record `SVLD1UW_GATHER_64B_OFFSETS_U`.
  **L71 CN**: 声明 TableGen def 记录 `SVLD1UW_GATHER_64B_OFFSETS_U`。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L73 EN**: Declares TableGen def record `SVLD1_GATHER_32B_OFFSETS_U`.
  **L73 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_32B_OFFSETS_U`。
- **L74 EN**: Declares TableGen def record `SVLD1SB_GATHER_32B_OFFSETS_U`.
  **L74 CN**: 声明 TableGen def 记录 `SVLD1SB_GATHER_32B_OFFSETS_U`。
- **L75 EN**: Declares TableGen def record `SVLD1UB_GATHER_32B_OFFSETS_U`.
  **L75 CN**: 声明 TableGen def 记录 `SVLD1UB_GATHER_32B_OFFSETS_U`。
- **L76 EN**: Declares TableGen def record `SVLD1SH_GATHER_32B_OFFSETS_U`.
  **L76 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_32B_OFFSETS_U`。
- **L77 EN**: Declares TableGen def record `SVLD1UH_GATHER_32B_OFFSETS_U`.
  **L77 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_32B_OFFSETS_U`。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (vector base, signed scalar offset in bytes)`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (vector base, signed scalar offset in bytes)`。
- **L80 EN**: Declares TableGen def record `SVLD1_GATHER_OFFSET_S`.
  **L80 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_OFFSET_S`。
- **L81 EN**: Declares TableGen def record `SVLD1SB_GATHER_OFFSET_S`.
  **L81 CN**: 声明 TableGen def 记录 `SVLD1SB_GATHER_OFFSET_S`。
- **L82 EN**: Declares TableGen def record `SVLD1UB_GATHER_OFFSET_S`.
  **L82 CN**: 声明 TableGen def 记录 `SVLD1UB_GATHER_OFFSET_S`。
- **L83 EN**: Declares TableGen def record `SVLD1SH_GATHER_OFFSET_S`.
  **L83 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_OFFSET_S`。
- **L84 EN**: Declares TableGen def record `SVLD1UH_GATHER_OFFSET_S`.
  **L84 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_OFFSET_S`。

### Lines 85-112

````tablegen
def SVLD1SW_GATHER_OFFSET_S : MInst<"svld1sw_gather[_{2}base]_offset_{d}", "dPul", "lUl",      [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1UW_GATHER_OFFSET_S : MInst<"svld1uw_gather[_{2}base]_offset_{d}", "dPul", "lUl",      [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ld1_gather_scalar_offset">;

// Load one vector (scalar base, signed vector index)
def SVLD1_GATHER_64B_INDICES_S   : MInst<"svld1_gather_[{3}]index[_{d}]", "dPcx", "lUld", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ld1_gather_index">;
def SVLD1SH_GATHER_64B_INDICES_S : MInst<"svld1sh_gather_[{3}]index_{d}", "dPTx", "lUl",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ld1_gather_index">;
def SVLD1UH_GATHER_64B_INDICES_S : MInst<"svld1uh_gather_[{3}]index_{d}", "dPXx", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_index">;
def SVLD1SW_GATHER_64B_INDICES_S : MInst<"svld1sw_gather_[{3}]index_{d}", "dPUx", "lUl",  [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ld1_gather_index">;
def SVLD1UW_GATHER_64B_INDICES_S : MInst<"svld1uw_gather_[{3}]index_{d}", "dPYx", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ld1_gather_index">;

def SVLD1_GATHER_32B_INDICES_S   : MInst<"svld1_gather_[{3}]index[_{d}]", "dPcx", "iUif", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ld1_gather_sxtw_index">;
def SVLD1SH_GATHER_32B_INDICES_S : MInst<"svld1sh_gather_[{3}]index_{d}", "dPTx", "iUi",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ld1_gather_sxtw_index">;
def SVLD1UH_GATHER_32B_INDICES_S : MInst<"svld1uh_gather_[{3}]index_{d}", "dPXx", "iUi",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_sxtw_index">;

// Load one vector (scalar base, unsigned vector index)
def SVLD1_GATHER_64B_INDICES_U   : MInst<"svld1_gather_[{3}]index[_{d}]", "dPcu", "lUld", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ld1_gather_index">;
def SVLD1SH_GATHER_64B_INDICES_U : MInst<"svld1sh_gather_[{3}]index_{d}", "dPTu", "lUl",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ld1_gather_index">;
def SVLD1UH_GATHER_64B_INDICES_U : MInst<"svld1uh_gather_[{3}]index_{d}", "dPXu", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_index">;
def SVLD1SW_GATHER_64B_INDICES_U : MInst<"svld1sw_gather_[{3}]index_{d}", "dPUu", "lUl",  [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ld1_gather_index">;
def SVLD1UW_GATHER_64B_INDICES_U : MInst<"svld1uw_gather_[{3}]index_{d}", "dPYu", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ld1_gather_index">;

def SVLD1_GATHER_32B_INDICES_U   : MInst<"svld1_gather_[{3}]index[_{d}]", "dPcu", "iUif", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ld1_gather_uxtw_index">;
def SVLD1SH_GATHER_32B_INDICES_U : MInst<"svld1sh_gather_[{3}]index_{d}", "dPTu", "iUi",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ld1_gather_uxtw_index">;
def SVLD1UH_GATHER_32B_INDICES_U : MInst<"svld1uh_gather_[{3}]index_{d}", "dPXu", "iUi",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_uxtw_index">;

// Load one vector (vector base, signed scalar index)
def SVLD1_GATHER_INDEX_S     : MInst<"svld1_gather[_{2}base]_index_{d}",   "dPul", "ilUiUlfd", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1SH_GATHER_INDEX_S   : MInst<"svld1sh_gather[_{2}base]_index_{d}", "dPul", "ilUiUl",   [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ld1_gather_scalar_offset">;
````
- **L85 EN**: Declares TableGen def record `SVLD1SW_GATHER_OFFSET_S`.
  **L85 CN**: 声明 TableGen def 记录 `SVLD1SW_GATHER_OFFSET_S`。
- **L86 EN**: Declares TableGen def record `SVLD1UW_GATHER_OFFSET_S`.
  **L86 CN**: 声明 TableGen def 记录 `SVLD1UW_GATHER_OFFSET_S`。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L88 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (scalar base, signed vector index)`.
  **L88 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (scalar base, signed vector index)`。
- **L89 EN**: Declares TableGen def record `SVLD1_GATHER_64B_INDICES_S`.
  **L89 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_64B_INDICES_S`。
- **L90 EN**: Declares TableGen def record `SVLD1SH_GATHER_64B_INDICES_S`.
  **L90 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_64B_INDICES_S`。
- **L91 EN**: Declares TableGen def record `SVLD1UH_GATHER_64B_INDICES_S`.
  **L91 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_64B_INDICES_S`。
- **L92 EN**: Declares TableGen def record `SVLD1SW_GATHER_64B_INDICES_S`.
  **L92 CN**: 声明 TableGen def 记录 `SVLD1SW_GATHER_64B_INDICES_S`。
- **L93 EN**: Declares TableGen def record `SVLD1UW_GATHER_64B_INDICES_S`.
  **L93 CN**: 声明 TableGen def 记录 `SVLD1UW_GATHER_64B_INDICES_S`。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L95 EN**: Declares TableGen def record `SVLD1_GATHER_32B_INDICES_S`.
  **L95 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_32B_INDICES_S`。
- **L96 EN**: Declares TableGen def record `SVLD1SH_GATHER_32B_INDICES_S`.
  **L96 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_32B_INDICES_S`。
- **L97 EN**: Declares TableGen def record `SVLD1UH_GATHER_32B_INDICES_S`.
  **L97 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_32B_INDICES_S`。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (scalar base, unsigned vector index)`.
  **L99 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (scalar base, unsigned vector index)`。
- **L100 EN**: Declares TableGen def record `SVLD1_GATHER_64B_INDICES_U`.
  **L100 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_64B_INDICES_U`。
- **L101 EN**: Declares TableGen def record `SVLD1SH_GATHER_64B_INDICES_U`.
  **L101 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_64B_INDICES_U`。
- **L102 EN**: Declares TableGen def record `SVLD1UH_GATHER_64B_INDICES_U`.
  **L102 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_64B_INDICES_U`。
- **L103 EN**: Declares TableGen def record `SVLD1SW_GATHER_64B_INDICES_U`.
  **L103 CN**: 声明 TableGen def 记录 `SVLD1SW_GATHER_64B_INDICES_U`。
- **L104 EN**: Declares TableGen def record `SVLD1UW_GATHER_64B_INDICES_U`.
  **L104 CN**: 声明 TableGen def 记录 `SVLD1UW_GATHER_64B_INDICES_U`。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L106 EN**: Declares TableGen def record `SVLD1_GATHER_32B_INDICES_U`.
  **L106 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_32B_INDICES_U`。
- **L107 EN**: Declares TableGen def record `SVLD1SH_GATHER_32B_INDICES_U`.
  **L107 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_32B_INDICES_U`。
- **L108 EN**: Declares TableGen def record `SVLD1UH_GATHER_32B_INDICES_U`.
  **L108 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_32B_INDICES_U`。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (vector base, signed scalar index)`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (vector base, signed scalar index)`。
- **L111 EN**: Declares TableGen def record `SVLD1_GATHER_INDEX_S`.
  **L111 CN**: 声明 TableGen def 记录 `SVLD1_GATHER_INDEX_S`。
- **L112 EN**: Declares TableGen def record `SVLD1SH_GATHER_INDEX_S`.
  **L112 CN**: 声明 TableGen def 记录 `SVLD1SH_GATHER_INDEX_S`。

### Lines 113-140

````tablegen
def SVLD1UH_GATHER_INDEX_S   : MInst<"svld1uh_gather[_{2}base]_index_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1SW_GATHER_INDEX_S   : MInst<"svld1sw_gather[_{2}base]_index_{d}", "dPul", "lUl",      [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ld1_gather_scalar_offset">;
def SVLD1UW_GATHER_INDEX_S   : MInst<"svld1uw_gather[_{2}base]_index_{d}", "dPul", "lUl",      [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ld1_gather_scalar_offset">;


// First-faulting load one vector (scalar base)
def SVLDFF1   : MInst<"svldff1[_{2}]", "dPc", "csilUcUsUiUlhfdbm", [IsLoad],               MemEltTyDefault, "aarch64_sve_ldff1">;
def SVLDFF1SB : MInst<"svldff1sb_{d}", "dPS", "silUsUiUl",         [IsLoad],               MemEltTyInt8,    "aarch64_sve_ldff1">;
def SVLDFF1UB : MInst<"svldff1ub_{d}", "dPW", "silUsUiUl",         [IsLoad, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1">;
def SVLDFF1SH : MInst<"svldff1sh_{d}", "dPT", "ilUiUl",            [IsLoad],               MemEltTyInt16,   "aarch64_sve_ldff1">;
def SVLDFF1UH : MInst<"svldff1uh_{d}", "dPX", "ilUiUl",            [IsLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1">;
def SVLDFF1SW : MInst<"svldff1sw_{d}", "dPU", "lUl",               [IsLoad],               MemEltTyInt32,   "aarch64_sve_ldff1">;
def SVLDFF1UW : MInst<"svldff1uw_{d}", "dPY", "lUl",               [IsLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1">;

// First-faulting load one vector (scalar base, VL displacement)
def SVLDFF1_VNUM   : MInst<"svldff1_vnum[_{2}]", "dPcl", "csilUcUsUiUlhfdbm", [IsLoad],               MemEltTyDefault, "aarch64_sve_ldff1">;
def SVLDFF1SB_VNUM : MInst<"svldff1sb_vnum_{d}", "dPSl", "silUsUiUl",         [IsLoad],               MemEltTyInt8,    "aarch64_sve_ldff1">;
def SVLDFF1UB_VNUM : MInst<"svldff1ub_vnum_{d}", "dPWl", "silUsUiUl",         [IsLoad, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1">;
def SVLDFF1SH_VNUM : MInst<"svldff1sh_vnum_{d}", "dPTl", "ilUiUl",            [IsLoad],               MemEltTyInt16,   "aarch64_sve_ldff1">;
def SVLDFF1UH_VNUM : MInst<"svldff1uh_vnum_{d}", "dPXl", "ilUiUl",            [IsLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1">;
def SVLDFF1SW_VNUM : MInst<"svldff1sw_vnum_{d}", "dPUl", "lUl",               [IsLoad],               MemEltTyInt32,   "aarch64_sve_ldff1">;
def SVLDFF1UW_VNUM : MInst<"svldff1uw_vnum_{d}", "dPYl", "lUl",               [IsLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1">;
}

let SMETargetGuard = InvalidMode in {
// First-faulting load one vector (vector base)
def SVLDFF1_GATHER_BASES_U   : MInst<"svldff1_gather[_{2}base]_{d}",   "dPu", "ilUiUlfd", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SB_GATHER_BASES_U : MInst<"svldff1sb_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad],               MemEltTyInt8,    "aarch64_sve_ldff1_gather_scalar_offset">;
````
- **L113 EN**: Declares TableGen def record `SVLD1UH_GATHER_INDEX_S`.
  **L113 CN**: 声明 TableGen def 记录 `SVLD1UH_GATHER_INDEX_S`。
- **L114 EN**: Declares TableGen def record `SVLD1SW_GATHER_INDEX_S`.
  **L114 CN**: 声明 TableGen def 记录 `SVLD1SW_GATHER_INDEX_S`。
- **L115 EN**: Declares TableGen def record `SVLD1UW_GATHER_INDEX_S`.
  **L115 CN**: 声明 TableGen def 记录 `SVLD1UW_GATHER_INDEX_S`。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L118 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (scalar base)`.
  **L118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (scalar base)`。
- **L119 EN**: Declares TableGen def record `SVLDFF1`.
  **L119 CN**: 声明 TableGen def 记录 `SVLDFF1`。
- **L120 EN**: Declares TableGen def record `SVLDFF1SB`.
  **L120 CN**: 声明 TableGen def 记录 `SVLDFF1SB`。
- **L121 EN**: Declares TableGen def record `SVLDFF1UB`.
  **L121 CN**: 声明 TableGen def 记录 `SVLDFF1UB`。
- **L122 EN**: Declares TableGen def record `SVLDFF1SH`.
  **L122 CN**: 声明 TableGen def 记录 `SVLDFF1SH`。
- **L123 EN**: Declares TableGen def record `SVLDFF1UH`.
  **L123 CN**: 声明 TableGen def 记录 `SVLDFF1UH`。
- **L124 EN**: Declares TableGen def record `SVLDFF1SW`.
  **L124 CN**: 声明 TableGen def 记录 `SVLDFF1SW`。
- **L125 EN**: Declares TableGen def record `SVLDFF1UW`.
  **L125 CN**: 声明 TableGen def 记录 `SVLDFF1UW`。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L127 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (scalar base, VL displacement)`.
  **L127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (scalar base, VL displacement)`。
- **L128 EN**: Declares TableGen def record `SVLDFF1_VNUM`.
  **L128 CN**: 声明 TableGen def 记录 `SVLDFF1_VNUM`。
- **L129 EN**: Declares TableGen def record `SVLDFF1SB_VNUM`.
  **L129 CN**: 声明 TableGen def 记录 `SVLDFF1SB_VNUM`。
- **L130 EN**: Declares TableGen def record `SVLDFF1UB_VNUM`.
  **L130 CN**: 声明 TableGen def 记录 `SVLDFF1UB_VNUM`。
- **L131 EN**: Declares TableGen def record `SVLDFF1SH_VNUM`.
  **L131 CN**: 声明 TableGen def 记录 `SVLDFF1SH_VNUM`。
- **L132 EN**: Declares TableGen def record `SVLDFF1UH_VNUM`.
  **L132 CN**: 声明 TableGen def 记录 `SVLDFF1UH_VNUM`。
- **L133 EN**: Declares TableGen def record `SVLDFF1SW_VNUM`.
  **L133 CN**: 声明 TableGen def 记录 `SVLDFF1SW_VNUM`。
- **L134 EN**: Declares TableGen def record `SVLDFF1UW_VNUM`.
  **L134 CN**: 声明 TableGen def 记录 `SVLDFF1UW_VNUM`。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L137 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L137 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L138 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (vector base)`.
  **L138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (vector base)`。
- **L139 EN**: Declares TableGen def record `SVLDFF1_GATHER_BASES_U`.
  **L139 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_BASES_U`。
- **L140 EN**: Declares TableGen def record `SVLDFF1SB_GATHER_BASES_U`.
  **L140 CN**: 声明 TableGen def 记录 `SVLDFF1SB_GATHER_BASES_U`。

### Lines 141-168

````tablegen
def SVLDFF1UB_GATHER_BASES_U : MInst<"svldff1ub_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SH_GATHER_BASES_U : MInst<"svldff1sh_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1UH_GATHER_BASES_U : MInst<"svldff1uh_gather[_{2}base]_{d}", "dPu", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SW_GATHER_BASES_U : MInst<"svldff1sw_gather[_{2}base]_{d}", "dPu", "lUl",      [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1UW_GATHER_BASES_U : MInst<"svldff1uw_gather[_{2}base]_{d}", "dPu", "lUl",      [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1_gather_scalar_offset">;

// First-faulting load one vector (scalar base, signed vector offset in bytes)
def SVLDFF1_GATHER_64B_OFFSETS_S   : MInst<"svldff1_gather_[{3}]offset[_{d}]", "dPcx", "lUld", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldff1_gather">;
def SVLDFF1SB_GATHER_64B_OFFSETS_S : MInst<"svldff1sb_gather_[{3}]offset_{d}", "dPSx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldff1_gather">;
def SVLDFF1UB_GATHER_64B_OFFSETS_S : MInst<"svldff1ub_gather_[{3}]offset_{d}", "dPWx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1_gather">;
def SVLDFF1SH_GATHER_64B_OFFSETS_S : MInst<"svldff1sh_gather_[{3}]offset_{d}", "dPTx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldff1_gather">;
def SVLDFF1UH_GATHER_64B_OFFSETS_S : MInst<"svldff1uh_gather_[{3}]offset_{d}", "dPXx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather">;
def SVLDFF1SW_GATHER_64B_OFFSETS_S : MInst<"svldff1sw_gather_[{3}]offset_{d}", "dPUx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ldff1_gather">;
def SVLDFF1UW_GATHER_64B_OFFSETS_S : MInst<"svldff1uw_gather_[{3}]offset_{d}", "dPYx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1_gather">;

def SVLDFF1_GATHER_32B_OFFSETS_S   : MInst<"svldff1_gather_[{3}]offset[_{d}]", "dPcx", "iUif", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldff1_gather_sxtw">;
def SVLDFF1SB_GATHER_32B_OFFSETS_S : MInst<"svldff1sb_gather_[{3}]offset_{d}", "dPSx", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldff1_gather_sxtw">;
def SVLDFF1UB_GATHER_32B_OFFSETS_S : MInst<"svldff1ub_gather_[{3}]offset_{d}", "dPWx", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1_gather_sxtw">;
def SVLDFF1SH_GATHER_32B_OFFSETS_S : MInst<"svldff1sh_gather_[{3}]offset_{d}", "dPTx", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_sxtw">;
def SVLDFF1UH_GATHER_32B_OFFSETS_S : MInst<"svldff1uh_gather_[{3}]offset_{d}", "dPXx", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_sxtw">;

// First-faulting load one vector (scalar base, unsigned vector offset in bytes)
def SVLDFF1_GATHER_64B_OFFSETS_U   : MInst<"svldff1_gather_[{3}]offset[_{d}]", "dPcu", "lUld", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldff1_gather">;
def SVLDFF1SB_GATHER_64B_OFFSETS_U : MInst<"svldff1sb_gather_[{3}]offset_{d}", "dPSu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldff1_gather">;
def SVLDFF1UB_GATHER_64B_OFFSETS_U : MInst<"svldff1ub_gather_[{3}]offset_{d}", "dPWu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1_gather">;
def SVLDFF1SH_GATHER_64B_OFFSETS_U : MInst<"svldff1sh_gather_[{3}]offset_{d}", "dPTu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldff1_gather">;
def SVLDFF1UH_GATHER_64B_OFFSETS_U : MInst<"svldff1uh_gather_[{3}]offset_{d}", "dPXu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather">;
def SVLDFF1SW_GATHER_64B_OFFSETS_U : MInst<"svldff1sw_gather_[{3}]offset_{d}", "dPUu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ldff1_gather">;
````
- **L141 EN**: Declares TableGen def record `SVLDFF1UB_GATHER_BASES_U`.
  **L141 CN**: 声明 TableGen def 记录 `SVLDFF1UB_GATHER_BASES_U`。
- **L142 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_BASES_U`.
  **L142 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_BASES_U`。
- **L143 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_BASES_U`.
  **L143 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_BASES_U`。
- **L144 EN**: Declares TableGen def record `SVLDFF1SW_GATHER_BASES_U`.
  **L144 CN**: 声明 TableGen def 记录 `SVLDFF1SW_GATHER_BASES_U`。
- **L145 EN**: Declares TableGen def record `SVLDFF1UW_GATHER_BASES_U`.
  **L145 CN**: 声明 TableGen def 记录 `SVLDFF1UW_GATHER_BASES_U`。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (scalar base, signed vector offset in bytes)`.
  **L147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (scalar base, signed vector offset in bytes)`。
- **L148 EN**: Declares TableGen def record `SVLDFF1_GATHER_64B_OFFSETS_S`.
  **L148 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_64B_OFFSETS_S`。
- **L149 EN**: Declares TableGen def record `SVLDFF1SB_GATHER_64B_OFFSETS_S`.
  **L149 CN**: 声明 TableGen def 记录 `SVLDFF1SB_GATHER_64B_OFFSETS_S`。
- **L150 EN**: Declares TableGen def record `SVLDFF1UB_GATHER_64B_OFFSETS_S`.
  **L150 CN**: 声明 TableGen def 记录 `SVLDFF1UB_GATHER_64B_OFFSETS_S`。
- **L151 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_64B_OFFSETS_S`.
  **L151 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_64B_OFFSETS_S`。
- **L152 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_64B_OFFSETS_S`.
  **L152 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_64B_OFFSETS_S`。
- **L153 EN**: Declares TableGen def record `SVLDFF1SW_GATHER_64B_OFFSETS_S`.
  **L153 CN**: 声明 TableGen def 记录 `SVLDFF1SW_GATHER_64B_OFFSETS_S`。
- **L154 EN**: Declares TableGen def record `SVLDFF1UW_GATHER_64B_OFFSETS_S`.
  **L154 CN**: 声明 TableGen def 记录 `SVLDFF1UW_GATHER_64B_OFFSETS_S`。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L156 EN**: Declares TableGen def record `SVLDFF1_GATHER_32B_OFFSETS_S`.
  **L156 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_32B_OFFSETS_S`。
- **L157 EN**: Declares TableGen def record `SVLDFF1SB_GATHER_32B_OFFSETS_S`.
  **L157 CN**: 声明 TableGen def 记录 `SVLDFF1SB_GATHER_32B_OFFSETS_S`。
- **L158 EN**: Declares TableGen def record `SVLDFF1UB_GATHER_32B_OFFSETS_S`.
  **L158 CN**: 声明 TableGen def 记录 `SVLDFF1UB_GATHER_32B_OFFSETS_S`。
- **L159 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_32B_OFFSETS_S`.
  **L159 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_32B_OFFSETS_S`。
- **L160 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_32B_OFFSETS_S`.
  **L160 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_32B_OFFSETS_S`。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (scalar base, unsigned vector offset in bytes)`.
  **L162 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (scalar base, unsigned vector offset in bytes)`。
- **L163 EN**: Declares TableGen def record `SVLDFF1_GATHER_64B_OFFSETS_U`.
  **L163 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_64B_OFFSETS_U`。
- **L164 EN**: Declares TableGen def record `SVLDFF1SB_GATHER_64B_OFFSETS_U`.
  **L164 CN**: 声明 TableGen def 记录 `SVLDFF1SB_GATHER_64B_OFFSETS_U`。
- **L165 EN**: Declares TableGen def record `SVLDFF1UB_GATHER_64B_OFFSETS_U`.
  **L165 CN**: 声明 TableGen def 记录 `SVLDFF1UB_GATHER_64B_OFFSETS_U`。
- **L166 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_64B_OFFSETS_U`.
  **L166 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_64B_OFFSETS_U`。
- **L167 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_64B_OFFSETS_U`.
  **L167 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_64B_OFFSETS_U`。
- **L168 EN**: Declares TableGen def record `SVLDFF1SW_GATHER_64B_OFFSETS_U`.
  **L168 CN**: 声明 TableGen def 记录 `SVLDFF1SW_GATHER_64B_OFFSETS_U`。

### Lines 169-196

````tablegen
def SVLDFF1UW_GATHER_64B_OFFSETS_U : MInst<"svldff1uw_gather_[{3}]offset_{d}", "dPYu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1_gather">;

def SVLDFF1_GATHER_32B_OFFSETS_U   : MInst<"svldff1_gather_[{3}]offset[_{d}]", "dPcu", "iUif", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldff1_gather_uxtw">;
def SVLDFF1SB_GATHER_32B_OFFSETS_U : MInst<"svldff1sb_gather_[{3}]offset_{d}", "dPSu", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldff1_gather_uxtw">;
def SVLDFF1UB_GATHER_32B_OFFSETS_U : MInst<"svldff1ub_gather_[{3}]offset_{d}", "dPWu", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1_gather_uxtw">;
def SVLDFF1SH_GATHER_32B_OFFSETS_U : MInst<"svldff1sh_gather_[{3}]offset_{d}", "dPTu", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_uxtw">;
def SVLDFF1UH_GATHER_32B_OFFSETS_U : MInst<"svldff1uh_gather_[{3}]offset_{d}", "dPXu", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_uxtw">;

// First-faulting load one vector (vector base, signed scalar offset in bytes)
def SVLDFF1_GATHER_OFFSET_S   : MInst<"svldff1_gather[_{2}base]_offset_{d}",   "dPul", "ilUiUlfd", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SB_GATHER_OFFSET_S : MInst<"svldff1sb_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1UB_GATHER_OFFSET_S : MInst<"svldff1ub_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SH_GATHER_OFFSET_S : MInst<"svldff1sh_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1UH_GATHER_OFFSET_S : MInst<"svldff1uh_gather[_{2}base]_offset_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SW_GATHER_OFFSET_S : MInst<"svldff1sw_gather[_{2}base]_offset_{d}", "dPul", "lUl",      [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1UW_GATHER_OFFSET_S : MInst<"svldff1uw_gather[_{2}base]_offset_{d}", "dPul", "lUl",      [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1_gather_scalar_offset">;

// First-faulting load one vector (scalar base, signed vector index)
def SVLDFF1_GATHER_64B_INDICES_S   : MInst<"svldff1_gather_[{3}]index[_{d}]", "dPcx", "lUld", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldff1_gather_index">;
def SVLDFF1SH_GATHER_64B_INDICES_S : MInst<"svldff1sh_gather_[{3}]index_{d}", "dPTx", "lUl",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_index">;
def SVLDFF1UH_GATHER_64B_INDICES_S : MInst<"svldff1uh_gather_[{3}]index_{d}", "dPXx", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_index">;
def SVLDFF1SW_GATHER_64B_INDICES_S : MInst<"svldff1sw_gather_[{3}]index_{d}", "dPUx", "lUl",  [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldff1_gather_index">;
def SVLDFF1UW_GATHER_64B_INDICES_S : MInst<"svldff1uw_gather_[{3}]index_{d}", "dPYx", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1_gather_index">;

def SVLDFF1_GATHER_32B_INDICES_S   : MInst<"svldff1_gather_[{3}]index[_{d}]", "dPcx", "iUif", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldff1_gather_sxtw_index">;
def SVLDFF1SH_GATHER_32B_INDICES_S : MInst<"svldff1sh_gather_[{3}]index_{d}", "dPTx", "iUi",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_sxtw_index">;
def SVLDFF1UH_GATHER_32B_INDICES_S : MInst<"svldff1uh_gather_[{3}]index_{d}", "dPXx", "iUi",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_sxtw_index">;

````
- **L169 EN**: Declares TableGen def record `SVLDFF1UW_GATHER_64B_OFFSETS_U`.
  **L169 CN**: 声明 TableGen def 记录 `SVLDFF1UW_GATHER_64B_OFFSETS_U`。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L171 EN**: Declares TableGen def record `SVLDFF1_GATHER_32B_OFFSETS_U`.
  **L171 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_32B_OFFSETS_U`。
- **L172 EN**: Declares TableGen def record `SVLDFF1SB_GATHER_32B_OFFSETS_U`.
  **L172 CN**: 声明 TableGen def 记录 `SVLDFF1SB_GATHER_32B_OFFSETS_U`。
- **L173 EN**: Declares TableGen def record `SVLDFF1UB_GATHER_32B_OFFSETS_U`.
  **L173 CN**: 声明 TableGen def 记录 `SVLDFF1UB_GATHER_32B_OFFSETS_U`。
- **L174 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_32B_OFFSETS_U`.
  **L174 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_32B_OFFSETS_U`。
- **L175 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_32B_OFFSETS_U`.
  **L175 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_32B_OFFSETS_U`。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L177 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (vector base, signed scalar offset in bytes)`.
  **L177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (vector base, signed scalar offset in bytes)`。
- **L178 EN**: Declares TableGen def record `SVLDFF1_GATHER_OFFSET_S`.
  **L178 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_OFFSET_S`。
- **L179 EN**: Declares TableGen def record `SVLDFF1SB_GATHER_OFFSET_S`.
  **L179 CN**: 声明 TableGen def 记录 `SVLDFF1SB_GATHER_OFFSET_S`。
- **L180 EN**: Declares TableGen def record `SVLDFF1UB_GATHER_OFFSET_S`.
  **L180 CN**: 声明 TableGen def 记录 `SVLDFF1UB_GATHER_OFFSET_S`。
- **L181 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_OFFSET_S`.
  **L181 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_OFFSET_S`。
- **L182 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_OFFSET_S`.
  **L182 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_OFFSET_S`。
- **L183 EN**: Declares TableGen def record `SVLDFF1SW_GATHER_OFFSET_S`.
  **L183 CN**: 声明 TableGen def 记录 `SVLDFF1SW_GATHER_OFFSET_S`。
- **L184 EN**: Declares TableGen def record `SVLDFF1UW_GATHER_OFFSET_S`.
  **L184 CN**: 声明 TableGen def 记录 `SVLDFF1UW_GATHER_OFFSET_S`。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (scalar base, signed vector index)`.
  **L186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (scalar base, signed vector index)`。
- **L187 EN**: Declares TableGen def record `SVLDFF1_GATHER_64B_INDICES_S`.
  **L187 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_64B_INDICES_S`。
- **L188 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_64B_INDICES_S`.
  **L188 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_64B_INDICES_S`。
- **L189 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_64B_INDICES_S`.
  **L189 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_64B_INDICES_S`。
- **L190 EN**: Declares TableGen def record `SVLDFF1SW_GATHER_64B_INDICES_S`.
  **L190 CN**: 声明 TableGen def 记录 `SVLDFF1SW_GATHER_64B_INDICES_S`。
- **L191 EN**: Declares TableGen def record `SVLDFF1UW_GATHER_64B_INDICES_S`.
  **L191 CN**: 声明 TableGen def 记录 `SVLDFF1UW_GATHER_64B_INDICES_S`。
- **L192 EN**: Blank line separating nearby declarations or logic blocks.
  **L192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L193 EN**: Declares TableGen def record `SVLDFF1_GATHER_32B_INDICES_S`.
  **L193 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_32B_INDICES_S`。
- **L194 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_32B_INDICES_S`.
  **L194 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_32B_INDICES_S`。
- **L195 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_32B_INDICES_S`.
  **L195 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_32B_INDICES_S`。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 197-224

````tablegen
// First-faulting load one vector (scalar base, unsigned vector index)
def SVLDFF1_GATHER_64B_INDICES_U   : MInst<"svldff1_gather_[{3}]index[_{d}]", "dPcu", "lUld", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldff1_gather_index">;
def SVLDFF1SH_GATHER_64B_INDICES_U : MInst<"svldff1sh_gather_[{3}]index_{d}", "dPTu", "lUl",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_index">;
def SVLDFF1UH_GATHER_64B_INDICES_U : MInst<"svldff1uh_gather_[{3}]index_{d}", "dPXu", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_index">;
def SVLDFF1SW_GATHER_64B_INDICES_U : MInst<"svldff1sw_gather_[{3}]index_{d}", "dPUu", "lUl",  [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldff1_gather_index">;
def SVLDFF1UW_GATHER_64B_INDICES_U : MInst<"svldff1uw_gather_[{3}]index_{d}", "dPYu", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1_gather_index">;

def SVLDFF1_GATHER_32B_INDICES_U   : MInst<"svldff1_gather_[{3}]index[_{d}]", "dPcu", "iUif", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldff1_gather_uxtw_index">;
def SVLDFF1SH_GATHER_32B_INDICES_U : MInst<"svldff1sh_gather_[{3}]index_{d}", "dPTu", "iUi",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_uxtw_index">;
def SVLDFF1UH_GATHER_32B_INDICES_U : MInst<"svldff1uh_gather_[{3}]index_{d}", "dPXu", "iUi",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_uxtw_index">;

// First-faulting load one vector (vector base, signed scalar index)
def SVLDFF1_GATHER_INDEX_S   : MInst<"svldff1_gather[_{2}base]_index_{d}",   "dPul", "ilUiUlfd", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SH_GATHER_INDEX_S : MInst<"svldff1sh_gather[_{2}base]_index_{d}", "dPul", "ilUiUl",   [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1UH_GATHER_INDEX_S : MInst<"svldff1uh_gather[_{2}base]_index_{d}", "dPul", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1SW_GATHER_INDEX_S : MInst<"svldff1sw_gather[_{2}base]_index_{d}", "dPul", "lUl",      [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldff1_gather_scalar_offset">;
def SVLDFF1UW_GATHER_INDEX_S : MInst<"svldff1uw_gather[_{2}base]_index_{d}", "dPul", "lUl",      [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldff1_gather_scalar_offset">;

// Non-faulting load one vector (scalar base)
def SVLDNF1   : MInst<"svldnf1[_{2}]", "dPc", "csilUcUsUiUlhfdbm", [IsLoad],               MemEltTyDefault, "aarch64_sve_ldnf1">;
def SVLDNF1SB : MInst<"svldnf1sb_{d}", "dPS", "silUsUiUl",         [IsLoad],               MemEltTyInt8,    "aarch64_sve_ldnf1">;
def SVLDNF1UB : MInst<"svldnf1ub_{d}", "dPW", "silUsUiUl",         [IsLoad, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldnf1">;
def SVLDNF1SH : MInst<"svldnf1sh_{d}", "dPT", "ilUiUl",            [IsLoad],               MemEltTyInt16,   "aarch64_sve_ldnf1">;
def SVLDNF1UH : MInst<"svldnf1uh_{d}", "dPX", "ilUiUl",            [IsLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnf1">;
def SVLDNF1SW : MInst<"svldnf1sw_{d}", "dPU", "lUl",               [IsLoad],               MemEltTyInt32,   "aarch64_sve_ldnf1">;
def SVLDNF1UW : MInst<"svldnf1uw_{d}", "dPY", "lUl",               [IsLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnf1">;

// Non-faulting load one vector (scalar base, VL displacement)
````
- **L197 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (scalar base, unsigned vector index)`.
  **L197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (scalar base, unsigned vector index)`。
- **L198 EN**: Declares TableGen def record `SVLDFF1_GATHER_64B_INDICES_U`.
  **L198 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_64B_INDICES_U`。
- **L199 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_64B_INDICES_U`.
  **L199 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_64B_INDICES_U`。
- **L200 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_64B_INDICES_U`.
  **L200 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_64B_INDICES_U`。
- **L201 EN**: Declares TableGen def record `SVLDFF1SW_GATHER_64B_INDICES_U`.
  **L201 CN**: 声明 TableGen def 记录 `SVLDFF1SW_GATHER_64B_INDICES_U`。
- **L202 EN**: Declares TableGen def record `SVLDFF1UW_GATHER_64B_INDICES_U`.
  **L202 CN**: 声明 TableGen def 记录 `SVLDFF1UW_GATHER_64B_INDICES_U`。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L204 EN**: Declares TableGen def record `SVLDFF1_GATHER_32B_INDICES_U`.
  **L204 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_32B_INDICES_U`。
- **L205 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_32B_INDICES_U`.
  **L205 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_32B_INDICES_U`。
- **L206 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_32B_INDICES_U`.
  **L206 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_32B_INDICES_U`。
- **L207 EN**: Blank line separating nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L208 EN**: Comment explains nearby logic, constraints, or intent: `First-faulting load one vector (vector base, signed scalar index)`.
  **L208 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First-faulting load one vector (vector base, signed scalar index)`。
- **L209 EN**: Declares TableGen def record `SVLDFF1_GATHER_INDEX_S`.
  **L209 CN**: 声明 TableGen def 记录 `SVLDFF1_GATHER_INDEX_S`。
- **L210 EN**: Declares TableGen def record `SVLDFF1SH_GATHER_INDEX_S`.
  **L210 CN**: 声明 TableGen def 记录 `SVLDFF1SH_GATHER_INDEX_S`。
- **L211 EN**: Declares TableGen def record `SVLDFF1UH_GATHER_INDEX_S`.
  **L211 CN**: 声明 TableGen def 记录 `SVLDFF1UH_GATHER_INDEX_S`。
- **L212 EN**: Declares TableGen def record `SVLDFF1SW_GATHER_INDEX_S`.
  **L212 CN**: 声明 TableGen def 记录 `SVLDFF1SW_GATHER_INDEX_S`。
- **L213 EN**: Declares TableGen def record `SVLDFF1UW_GATHER_INDEX_S`.
  **L213 CN**: 声明 TableGen def 记录 `SVLDFF1UW_GATHER_INDEX_S`。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Comment explains nearby logic, constraints, or intent: `Non-faulting load one vector (scalar base)`.
  **L215 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-faulting load one vector (scalar base)`。
- **L216 EN**: Declares TableGen def record `SVLDNF1`.
  **L216 CN**: 声明 TableGen def 记录 `SVLDNF1`。
- **L217 EN**: Declares TableGen def record `SVLDNF1SB`.
  **L217 CN**: 声明 TableGen def 记录 `SVLDNF1SB`。
- **L218 EN**: Declares TableGen def record `SVLDNF1UB`.
  **L218 CN**: 声明 TableGen def 记录 `SVLDNF1UB`。
- **L219 EN**: Declares TableGen def record `SVLDNF1SH`.
  **L219 CN**: 声明 TableGen def 记录 `SVLDNF1SH`。
- **L220 EN**: Declares TableGen def record `SVLDNF1UH`.
  **L220 CN**: 声明 TableGen def 记录 `SVLDNF1UH`。
- **L221 EN**: Declares TableGen def record `SVLDNF1SW`.
  **L221 CN**: 声明 TableGen def 记录 `SVLDNF1SW`。
- **L222 EN**: Declares TableGen def record `SVLDNF1UW`.
  **L222 CN**: 声明 TableGen def 记录 `SVLDNF1UW`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `Non-faulting load one vector (scalar base, VL displacement)`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-faulting load one vector (scalar base, VL displacement)`。

### Lines 225-252

````tablegen
def SVLDNF1_VNUM   : MInst<"svldnf1_vnum[_{2}]", "dPcl", "csilUcUsUiUlhfdbm", [IsLoad],               MemEltTyDefault, "aarch64_sve_ldnf1">;
def SVLDNF1SB_VNUM : MInst<"svldnf1sb_vnum_{d}", "dPSl", "silUsUiUl",         [IsLoad],               MemEltTyInt8,    "aarch64_sve_ldnf1">;
def SVLDNF1UB_VNUM : MInst<"svldnf1ub_vnum_{d}", "dPWl", "silUsUiUl",         [IsLoad, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldnf1">;
def SVLDNF1SH_VNUM : MInst<"svldnf1sh_vnum_{d}", "dPTl", "ilUiUl",            [IsLoad],               MemEltTyInt16,   "aarch64_sve_ldnf1">;
def SVLDNF1UH_VNUM : MInst<"svldnf1uh_vnum_{d}", "dPXl", "ilUiUl",            [IsLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnf1">;
def SVLDNF1SW_VNUM : MInst<"svldnf1sw_vnum_{d}", "dPUl", "lUl",               [IsLoad],               MemEltTyInt32,   "aarch64_sve_ldnf1">;
def SVLDNF1UW_VNUM : MInst<"svldnf1uw_vnum_{d}", "dPYl", "lUl",               [IsLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnf1">;
}

// Load one vector, unextended load, non-temporal (scalar base)
def SVLDNT1 : MInst<"svldnt1[_{2}]", "dPc", "csilUcUsUiUlhfdbm", [IsLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_ldnt1">;

// Load one vector, unextended load, non-temporal (scalar base, VL displacement)
def SVLDNT1_VNUM : MInst<"svldnt1_vnum[_{2}]", "dPcl", "csilUcUsUiUlhfdbm", [IsLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_ldnt1">;

// Load one quadword and replicate (scalar base)
def SVLD1RQ : MInst<"svld1rq[_{2}]", "dPc", "csilUcUsUiUlhfdbm", [IsLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_ld1rq">;

// Load N-element structure into N vectors (scalar base)
def SVLD2 : SInst<"svld2[_{2}]", "2Pc", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld2_sret", [IsStructLoad, VerifyRuntimeMode]>;
def SVLD3 : SInst<"svld3[_{2}]", "3Pc", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld3_sret", [IsStructLoad, VerifyRuntimeMode]>;
def SVLD4 : SInst<"svld4[_{2}]", "4Pc", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld4_sret", [IsStructLoad, VerifyRuntimeMode]>;

// Load N-element structure into N vectors (scalar base, VL displacement)
def SVLD2_VNUM : SInst<"svld2_vnum[_{2}]", "2Pcl", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld2_sret", [IsStructLoad, VerifyRuntimeMode]>;
def SVLD3_VNUM : SInst<"svld3_vnum[_{2}]", "3Pcl", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld3_sret", [IsStructLoad, VerifyRuntimeMode]>;
def SVLD4_VNUM : SInst<"svld4_vnum[_{2}]", "4Pcl", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld4_sret", [IsStructLoad, VerifyRuntimeMode]>;

````
- **L225 EN**: Declares TableGen def record `SVLDNF1_VNUM`.
  **L225 CN**: 声明 TableGen def 记录 `SVLDNF1_VNUM`。
- **L226 EN**: Declares TableGen def record `SVLDNF1SB_VNUM`.
  **L226 CN**: 声明 TableGen def 记录 `SVLDNF1SB_VNUM`。
- **L227 EN**: Declares TableGen def record `SVLDNF1UB_VNUM`.
  **L227 CN**: 声明 TableGen def 记录 `SVLDNF1UB_VNUM`。
- **L228 EN**: Declares TableGen def record `SVLDNF1SH_VNUM`.
  **L228 CN**: 声明 TableGen def 记录 `SVLDNF1SH_VNUM`。
- **L229 EN**: Declares TableGen def record `SVLDNF1UH_VNUM`.
  **L229 CN**: 声明 TableGen def 记录 `SVLDNF1UH_VNUM`。
- **L230 EN**: Declares TableGen def record `SVLDNF1SW_VNUM`.
  **L230 CN**: 声明 TableGen def 记录 `SVLDNF1SW_VNUM`。
- **L231 EN**: Declares TableGen def record `SVLDNF1UW_VNUM`.
  **L231 CN**: 声明 TableGen def 记录 `SVLDNF1UW_VNUM`。
- **L232 EN**: Closes the current lexical scope or compound statement.
  **L232 CN**: 结束当前词法作用域或复合语句块。
- **L233 EN**: Blank line separating nearby declarations or logic blocks.
  **L233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L234 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector, unextended load, non-temporal (scalar base)`.
  **L234 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector, unextended load, non-temporal (scalar base)`。
- **L235 EN**: Declares TableGen def record `SVLDNT1`.
  **L235 CN**: 声明 TableGen def 记录 `SVLDNT1`。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector, unextended load, non-temporal (scalar base, VL displacement)`.
  **L237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector, unextended load, non-temporal (scalar base, VL displacement)`。
- **L238 EN**: Declares TableGen def record `SVLDNT1_VNUM`.
  **L238 CN**: 声明 TableGen def 记录 `SVLDNT1_VNUM`。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L240 EN**: Comment explains nearby logic, constraints, or intent: `Load one quadword and replicate (scalar base)`.
  **L240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one quadword and replicate (scalar base)`。
- **L241 EN**: Declares TableGen def record `SVLD1RQ`.
  **L241 CN**: 声明 TableGen def 记录 `SVLD1RQ`。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `Load N-element structure into N vectors (scalar base)`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load N-element structure into N vectors (scalar base)`。
- **L244 EN**: Declares TableGen def record `SVLD2`.
  **L244 CN**: 声明 TableGen def 记录 `SVLD2`。
- **L245 EN**: Declares TableGen def record `SVLD3`.
  **L245 CN**: 声明 TableGen def 记录 `SVLD3`。
- **L246 EN**: Declares TableGen def record `SVLD4`.
  **L246 CN**: 声明 TableGen def 记录 `SVLD4`。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L248 EN**: Comment explains nearby logic, constraints, or intent: `Load N-element structure into N vectors (scalar base, VL displacement)`.
  **L248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load N-element structure into N vectors (scalar base, VL displacement)`。
- **L249 EN**: Declares TableGen def record `SVLD2_VNUM`.
  **L249 CN**: 声明 TableGen def 记录 `SVLD2_VNUM`。
- **L250 EN**: Declares TableGen def record `SVLD3_VNUM`.
  **L250 CN**: 声明 TableGen def 记录 `SVLD3_VNUM`。
- **L251 EN**: Declares TableGen def record `SVLD4_VNUM`.
  **L251 CN**: 声明 TableGen def 记录 `SVLD4_VNUM`。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 253-280

````tablegen
// Load one octoword and replicate (scalar base)
let SVETargetGuard = "f64mm", SMETargetGuard = InvalidMode in {
  def SVLD1RO : MInst<"svld1ro[_{2}]", "dPc", "csilUcUsUiUlhfdbm", [IsLoad], MemEltTyDefault, "aarch64_sve_ld1ro">;
}

let SVETargetGuard = "bf16", SMETargetGuard = InvalidMode in {
  def SVBFMMLA       : SInst<"svbfmmla[_{0}]",       "MMdd",  "b", MergeNone, "aarch64_sve_fmmla",        [IsOverloadFirstandLast]>;
}

let SVETargetGuard = "sve-b16mm", SMETargetGuard = InvalidMode in {
  def SVMMLA_BF16 : SInst<"svmmla[_bf16]", "dddd", "b", MergeNone, "aarch64_sve_fmmla", [IsOverloadFirstandLast]>;
}

let SVETargetGuard = "bf16", SMETargetGuard = "bf16" in {
  def SVBFDOT        : SInst<"svbfdot[_{0}]",        "MMdd",  "b", MergeNone, "aarch64_sve_bfdot",           [IsOverloadNone, VerifyRuntimeMode]>;
  def SVBFMLALB      : SInst<"svbfmlalb[_{0}]",      "MMdd",  "b", MergeNone, "aarch64_sve_bfmlalb",         [IsOverloadNone, VerifyRuntimeMode]>;
  def SVBFMLALT      : SInst<"svbfmlalt[_{0}]",      "MMdd",  "b", MergeNone, "aarch64_sve_bfmlalt",         [IsOverloadNone, VerifyRuntimeMode]>;
  def SVBFDOT_N      : SInst<"svbfdot[_n_{0}]",      "MMda",  "b", MergeNone, "aarch64_sve_bfdot",           [IsOverloadNone, VerifyRuntimeMode]>;
  def SVBFMLALB_N    : SInst<"svbfmlalb[_n_{0}]",    "MMda",  "b", MergeNone, "aarch64_sve_bfmlalb",         [IsOverloadNone, VerifyRuntimeMode]>;
  def SVBFMLALT_N    : SInst<"svbfmlalt[_n_{0}]",    "MMda",  "b", MergeNone, "aarch64_sve_bfmlalt",         [IsOverloadNone, VerifyRuntimeMode]>;
  def SVBFDOT_LANE   : SInst<"svbfdot_lane[_{0}]",   "MMddi", "b", MergeNone, "aarch64_sve_bfdot_lane_v2",   [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_3>]>;
  def SVBFMLALB_LANE : SInst<"svbfmlalb_lane[_{0}]", "MMddi", "b", MergeNone, "aarch64_sve_bfmlalb_lane_v2", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
  def SVBFMLALT_LANE : SInst<"svbfmlalt_lane[_{0}]", "MMddi", "b", MergeNone, "aarch64_sve_bfmlalt_lane_v2", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
}

let SVETargetGuard = "sve2p1", SMETargetGuard = InvalidMode in {
  // Contiguous zero-extending load to quadword (single vector).
  def SVLD1UWQ      : MInst<"svld1uwq[_{d}]",      "dPc",  "iUif", [IsLoad], MemEltTyInt32, "aarch64_sve_ld1uwq">;
````
- **L253 EN**: Comment explains nearby logic, constraints, or intent: `Load one octoword and replicate (scalar base)`.
  **L253 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one octoword and replicate (scalar base)`。
- **L254 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "f64mm", SMETargetGuard = InvalidMode in {`.
  **L254 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "f64mm", SMETargetGuard = InvalidMode in {`。
- **L255 EN**: Declares TableGen def record `SVLD1RO`.
  **L255 CN**: 声明 TableGen def 记录 `SVLD1RO`。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Blank line separating nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L258 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "bf16", SMETargetGuard = InvalidMode in {`.
  **L258 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "bf16", SMETargetGuard = InvalidMode in {`。
- **L259 EN**: Declares TableGen def record `SVBFMMLA`.
  **L259 CN**: 声明 TableGen def 记录 `SVBFMMLA`。
- **L260 EN**: Closes the current lexical scope or compound statement.
  **L260 CN**: 结束当前词法作用域或复合语句块。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L262 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-b16mm", SMETargetGuard = InvalidMode in {`.
  **L262 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-b16mm", SMETargetGuard = InvalidMode in {`。
- **L263 EN**: Declares TableGen def record `SVMMLA_BF16`.
  **L263 CN**: 声明 TableGen def 记录 `SVMMLA_BF16`。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L266 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "bf16", SMETargetGuard = "bf16" in {`.
  **L266 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "bf16", SMETargetGuard = "bf16" in {`。
- **L267 EN**: Declares TableGen def record `SVBFDOT`.
  **L267 CN**: 声明 TableGen def 记录 `SVBFDOT`。
- **L268 EN**: Declares TableGen def record `SVBFMLALB`.
  **L268 CN**: 声明 TableGen def 记录 `SVBFMLALB`。
- **L269 EN**: Declares TableGen def record `SVBFMLALT`.
  **L269 CN**: 声明 TableGen def 记录 `SVBFMLALT`。
- **L270 EN**: Declares TableGen def record `SVBFDOT_N`.
  **L270 CN**: 声明 TableGen def 记录 `SVBFDOT_N`。
- **L271 EN**: Declares TableGen def record `SVBFMLALB_N`.
  **L271 CN**: 声明 TableGen def 记录 `SVBFMLALB_N`。
- **L272 EN**: Declares TableGen def record `SVBFMLALT_N`.
  **L272 CN**: 声明 TableGen def 记录 `SVBFMLALT_N`。
- **L273 EN**: Declares TableGen def record `SVBFDOT_LANE`.
  **L273 CN**: 声明 TableGen def 记录 `SVBFDOT_LANE`。
- **L274 EN**: Declares TableGen def record `SVBFMLALB_LANE`.
  **L274 CN**: 声明 TableGen def 记录 `SVBFMLALB_LANE`。
- **L275 EN**: Declares TableGen def record `SVBFMLALT_LANE`.
  **L275 CN**: 声明 TableGen def 记录 `SVBFMLALT_LANE`。
- **L276 EN**: Closes the current lexical scope or compound statement.
  **L276 CN**: 结束当前词法作用域或复合语句块。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L278 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1", SMETargetGuard = InvalidMode in {`.
  **L278 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1", SMETargetGuard = InvalidMode in {`。
- **L279 EN**: Comment explains nearby logic, constraints, or intent: `Contiguous zero-extending load to quadword (single vector).`.
  **L279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Contiguous zero-extending load to quadword (single vector).`。
- **L280 EN**: Declares TableGen def record `SVLD1UWQ`.
  **L280 CN**: 声明 TableGen def 记录 `SVLD1UWQ`。

### Lines 281-308

````tablegen
  def SVLD1UWQ_VNUM : MInst<"svld1uwq_vnum[_{d}]", "dPcl", "iUif", [IsLoad], MemEltTyInt32, "aarch64_sve_ld1uwq">;

  def SVLD1UDQ      : MInst<"svld1udq[_{d}]",      "dPc",  "lUld", [IsLoad], MemEltTyInt64, "aarch64_sve_ld1udq">;
  def SVLD1UDQ_VNUM : MInst<"svld1udq_vnum[_{d}]", "dPcl", "lUld", [IsLoad], MemEltTyInt64, "aarch64_sve_ld1udq">;

  // Load one vector (vector base + scalar offset)
  def SVLD1Q_GATHER_U64BASE_OFFSET : MInst<"svld1q_gather[_{2}base]_offset_{d}", "dPgl", "cUcsUsiUilUlfhdbm", [IsGatherLoad, IsByteIndexed], MemEltTyDefault, "aarch64_sve_ld1q_gather_scalar_offset">;
  def SVLD1Q_GATHER_U64BASE :        MInst<"svld1q_gather[_{2}base]_{d}",         "dPg", "cUcsUsiUilUlfhdbm", [IsGatherLoad, IsByteIndexed], MemEltTyDefault, "aarch64_sve_ld1q_gather_scalar_offset">;

  // Load one vector (scalar base + vector offset)
  def SVLD1Q_GATHER_U64OFFSET : MInst<"svld1q_gather_[{3}]offset[_{d}]", "dPcg", "cUcsUsiUilUlfhdbm", [IsGatherLoad, IsByteIndexed], MemEltTyDefault, "aarch64_sve_ld1q_gather_vector_offset">;

  // Load quadwords (scalar base + vector index)
  def SVLD1Q_GATHER_INDICES_U : MInst<"svld1q_gather_[{3}]index[_{d}]", "dPcg", "sUsiUilUlbhfd", [IsGatherLoad], MemEltTyDefault, "aarch64_sve_ld1q_gather_index">;

  // Load quadwords (vector base + scalar index)
  def SVLD1Q_GATHER_INDEX_S   : MInst<"svld1q_gather[_{2}base]_index_{d}", "dPgl", "sUsiUilUlbhfd", [IsGatherLoad], MemEltTyDefault, "aarch64_sve_ld1q_gather_scalar_offset">;
}

let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {
  // Load N-element structure into N vectors (scalar base)
  def SVLD2Q : SInst<"svld2q[_{2}]", "2Pc", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld2q_sret", [IsStructLoad, VerifyRuntimeMode]>;
  def SVLD3Q : SInst<"svld3q[_{2}]", "3Pc", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld3q_sret", [IsStructLoad, VerifyRuntimeMode]>;
  def SVLD4Q : SInst<"svld4q[_{2}]", "4Pc", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld4q_sret", [IsStructLoad, VerifyRuntimeMode]>;

  // Load N-element structure into N vectors (scalar base, VL displacement)
  def SVLD2Q_VNUM : SInst<"svld2q_vnum[_{2}]", "2Pcl", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld2q_sret", [IsStructLoad, VerifyRuntimeMode]>;
  def SVLD3Q_VNUM : SInst<"svld3q_vnum[_{2}]", "3Pcl", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld3q_sret", [IsStructLoad, VerifyRuntimeMode]>;
````
- **L281 EN**: Declares TableGen def record `SVLD1UWQ_VNUM`.
  **L281 CN**: 声明 TableGen def 记录 `SVLD1UWQ_VNUM`。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L283 EN**: Declares TableGen def record `SVLD1UDQ`.
  **L283 CN**: 声明 TableGen def 记录 `SVLD1UDQ`。
- **L284 EN**: Declares TableGen def record `SVLD1UDQ_VNUM`.
  **L284 CN**: 声明 TableGen def 记录 `SVLD1UDQ_VNUM`。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L286 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (vector base + scalar offset)`.
  **L286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (vector base + scalar offset)`。
- **L287 EN**: Declares TableGen def record `SVLD1Q_GATHER_U64BASE_OFFSET`.
  **L287 CN**: 声明 TableGen def 记录 `SVLD1Q_GATHER_U64BASE_OFFSET`。
- **L288 EN**: Declares TableGen def record `SVLD1Q_GATHER_U64BASE`.
  **L288 CN**: 声明 TableGen def 记录 `SVLD1Q_GATHER_U64BASE`。
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L290 EN**: Comment explains nearby logic, constraints, or intent: `Load one vector (scalar base + vector offset)`.
  **L290 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load one vector (scalar base + vector offset)`。
- **L291 EN**: Declares TableGen def record `SVLD1Q_GATHER_U64OFFSET`.
  **L291 CN**: 声明 TableGen def 记录 `SVLD1Q_GATHER_U64OFFSET`。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L293 EN**: Comment explains nearby logic, constraints, or intent: `Load quadwords (scalar base + vector index)`.
  **L293 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load quadwords (scalar base + vector index)`。
- **L294 EN**: Declares TableGen def record `SVLD1Q_GATHER_INDICES_U`.
  **L294 CN**: 声明 TableGen def 记录 `SVLD1Q_GATHER_INDICES_U`。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L296 EN**: Comment explains nearby logic, constraints, or intent: `Load quadwords (vector base + scalar index)`.
  **L296 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load quadwords (vector base + scalar index)`。
- **L297 EN**: Declares TableGen def record `SVLD1Q_GATHER_INDEX_S`.
  **L297 CN**: 声明 TableGen def 记录 `SVLD1Q_GATHER_INDEX_S`。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L300 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`.
  **L300 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`。
- **L301 EN**: Comment explains nearby logic, constraints, or intent: `Load N-element structure into N vectors (scalar base)`.
  **L301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load N-element structure into N vectors (scalar base)`。
- **L302 EN**: Declares TableGen def record `SVLD2Q`.
  **L302 CN**: 声明 TableGen def 记录 `SVLD2Q`。
- **L303 EN**: Declares TableGen def record `SVLD3Q`.
  **L303 CN**: 声明 TableGen def 记录 `SVLD3Q`。
- **L304 EN**: Declares TableGen def record `SVLD4Q`.
  **L304 CN**: 声明 TableGen def 记录 `SVLD4Q`。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L306 EN**: Comment explains nearby logic, constraints, or intent: `Load N-element structure into N vectors (scalar base, VL displacement)`.
  **L306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load N-element structure into N vectors (scalar base, VL displacement)`。
- **L307 EN**: Declares TableGen def record `SVLD2Q_VNUM`.
  **L307 CN**: 声明 TableGen def 记录 `SVLD2Q_VNUM`。
- **L308 EN**: Declares TableGen def record `SVLD3Q_VNUM`.
  **L308 CN**: 声明 TableGen def 记录 `SVLD3Q_VNUM`。

### Lines 309-336

````tablegen
  def SVLD4Q_VNUM : SInst<"svld4q_vnum[_{2}]", "4Pcl", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_ld4q_sret", [IsStructLoad, VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// Stores

// Store one vector (scalar base)
def SVST1    : MInst<"svst1[_{d}]",  "vPpd", "csilUcUsUiUlhfdbm", [IsStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_st1">;
def SVST1B_S : MInst<"svst1b[_{d}]", "vPAd", "sil",               [IsStore, VerifyRuntimeMode], MemEltTyInt8,    "aarch64_sve_st1">;
def SVST1B_U : MInst<"svst1b[_{d}]", "vPEd", "UsUiUl",            [IsStore, VerifyRuntimeMode], MemEltTyInt8,    "aarch64_sve_st1">;
def SVST1H_S : MInst<"svst1h[_{d}]", "vPBd", "il",                [IsStore, VerifyRuntimeMode], MemEltTyInt16,   "aarch64_sve_st1">;
def SVST1H_U : MInst<"svst1h[_{d}]", "vPFd", "UiUl",              [IsStore, VerifyRuntimeMode], MemEltTyInt16,   "aarch64_sve_st1">;
def SVST1W_S : MInst<"svst1w[_{d}]", "vPCd", "l",                 [IsStore, VerifyRuntimeMode], MemEltTyInt32,   "aarch64_sve_st1">;
def SVST1W_U : MInst<"svst1w[_{d}]", "vPGd", "Ul",                [IsStore, VerifyRuntimeMode], MemEltTyInt32,   "aarch64_sve_st1">;

// Store one vector (scalar base, VL displacement)
def SVST1_VNUM    : MInst<"svst1_vnum[_{d}]",  "vPpld", "csilUcUsUiUlhfdbm", [IsStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_st1">;
def SVST1B_VNUM_S : MInst<"svst1b_vnum[_{d}]", "vPAld", "sil",               [IsStore, VerifyRuntimeMode], MemEltTyInt8,    "aarch64_sve_st1">;
def SVST1B_VNUM_U : MInst<"svst1b_vnum[_{d}]", "vPEld", "UsUiUl",            [IsStore, VerifyRuntimeMode], MemEltTyInt8,    "aarch64_sve_st1">;
def SVST1H_VNUM_S : MInst<"svst1h_vnum[_{d}]", "vPBld", "il",                [IsStore, VerifyRuntimeMode], MemEltTyInt16,   "aarch64_sve_st1">;
def SVST1H_VNUM_U : MInst<"svst1h_vnum[_{d}]", "vPFld", "UiUl",              [IsStore, VerifyRuntimeMode], MemEltTyInt16,   "aarch64_sve_st1">;
def SVST1W_VNUM_S : MInst<"svst1w_vnum[_{d}]", "vPCld", "l",                 [IsStore, VerifyRuntimeMode], MemEltTyInt32,   "aarch64_sve_st1">;
def SVST1W_VNUM_U : MInst<"svst1w_vnum[_{d}]", "vPGld", "Ul",                [IsStore, VerifyRuntimeMode], MemEltTyInt32,   "aarch64_sve_st1">;

let SMETargetGuard = InvalidMode in {
// Store one vector (vector base)
def SVST1_SCATTER_BASES_U     : MInst<"svst1_scatter[_{2}base_{d}]",  "vPud",  "ilUiUlfd", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1_scatter_scalar_offset">;
def SVST1B_SCATTER_BASES_U    : MInst<"svst1b_scatter[_{2}base_{d}]", "vPud",  "ilUiUl",   [IsScatterStore], MemEltTyInt8,    "aarch64_sve_st1_scatter_scalar_offset">;
````
- **L309 EN**: Declares TableGen def record `SVLD4Q_VNUM`.
  **L309 CN**: 声明 TableGen def 记录 `SVLD4Q_VNUM`。
- **L310 EN**: Closes the current lexical scope or compound statement.
  **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L312 EN**: Separator comment used for visual grouping.
  **L312 CN**: 用于视觉分组的分隔注释。
- **L313 EN**: Comment explains nearby logic, constraints, or intent: `Stores`.
  **L313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores`。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (scalar base)`.
  **L315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (scalar base)`。
- **L316 EN**: Declares TableGen def record `SVST1`.
  **L316 CN**: 声明 TableGen def 记录 `SVST1`。
- **L317 EN**: Declares TableGen def record `SVST1B_S`.
  **L317 CN**: 声明 TableGen def 记录 `SVST1B_S`。
- **L318 EN**: Declares TableGen def record `SVST1B_U`.
  **L318 CN**: 声明 TableGen def 记录 `SVST1B_U`。
- **L319 EN**: Declares TableGen def record `SVST1H_S`.
  **L319 CN**: 声明 TableGen def 记录 `SVST1H_S`。
- **L320 EN**: Declares TableGen def record `SVST1H_U`.
  **L320 CN**: 声明 TableGen def 记录 `SVST1H_U`。
- **L321 EN**: Declares TableGen def record `SVST1W_S`.
  **L321 CN**: 声明 TableGen def 记录 `SVST1W_S`。
- **L322 EN**: Declares TableGen def record `SVST1W_U`.
  **L322 CN**: 声明 TableGen def 记录 `SVST1W_U`。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (scalar base, VL displacement)`.
  **L324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (scalar base, VL displacement)`。
- **L325 EN**: Declares TableGen def record `SVST1_VNUM`.
  **L325 CN**: 声明 TableGen def 记录 `SVST1_VNUM`。
- **L326 EN**: Declares TableGen def record `SVST1B_VNUM_S`.
  **L326 CN**: 声明 TableGen def 记录 `SVST1B_VNUM_S`。
- **L327 EN**: Declares TableGen def record `SVST1B_VNUM_U`.
  **L327 CN**: 声明 TableGen def 记录 `SVST1B_VNUM_U`。
- **L328 EN**: Declares TableGen def record `SVST1H_VNUM_S`.
  **L328 CN**: 声明 TableGen def 记录 `SVST1H_VNUM_S`。
- **L329 EN**: Declares TableGen def record `SVST1H_VNUM_U`.
  **L329 CN**: 声明 TableGen def 记录 `SVST1H_VNUM_U`。
- **L330 EN**: Declares TableGen def record `SVST1W_VNUM_S`.
  **L330 CN**: 声明 TableGen def 记录 `SVST1W_VNUM_S`。
- **L331 EN**: Declares TableGen def record `SVST1W_VNUM_U`.
  **L331 CN**: 声明 TableGen def 记录 `SVST1W_VNUM_U`。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L333 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L334 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (vector base)`.
  **L334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (vector base)`。
- **L335 EN**: Declares TableGen def record `SVST1_SCATTER_BASES_U`.
  **L335 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_BASES_U`。
- **L336 EN**: Declares TableGen def record `SVST1B_SCATTER_BASES_U`.
  **L336 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_BASES_U`。

### Lines 337-364

````tablegen
def SVST1H_SCATTER_BASES_U    : MInst<"svst1h_scatter[_{2}base_{d}]", "vPud",  "ilUiUl",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_scalar_offset">;
def SVST1W_SCATTER_BASES_U    : MInst<"svst1w_scatter[_{2}base_{d}]", "vPud",  "lUl",      [IsScatterStore], MemEltTyInt32,   "aarch64_sve_st1_scatter_scalar_offset">;

// Store one vector (scalar base, signed vector offset in bytes)
def SVST1_SCATTER_64B_OFFSETS_S   : MInst<"svst1_scatter_[{3}]offset[_{d}]",  "vPpxd", "lUld", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1_scatter">;
def SVST1B_SCATTER_64B_OFFSETS_SS : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPAxd", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter">;
def SVST1B_SCATTER_64B_OFFSETS_SU : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPExd", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter">;
def SVST1H_SCATTER_64B_OFFSETS_SS : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPBxd", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter">;
def SVST1H_SCATTER_64B_OFFSETS_SU : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPFxd", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter">;
def SVST1W_SCATTER_64B_OFFSETS_SS : MInst<"svst1w_scatter_[{3}]offset[_{d}]", "vPCxd", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_st1_scatter">;
def SVST1W_SCATTER_64B_OFFSETS_SU : MInst<"svst1w_scatter_[{3}]offset[_{d}]", "vPGxd", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_st1_scatter">;

def SVST1_SCATTER_32B_OFFSETS_S   : MInst<"svst1_scatter_[{3}]offset[_{d}]",  "vPpxd", "iUif", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1_scatter_sxtw">;
def SVST1B_SCATTER_32B_OFFSETS_SS : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPAxd", "i",    [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter_sxtw">;
def SVST1B_SCATTER_32B_OFFSETS_SU : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPExd", "Ui",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter_sxtw">;
def SVST1H_SCATTER_32B_OFFSETS_SS : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPBxd", "i",    [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter_sxtw">;
def SVST1H_SCATTER_32B_OFFSETS_SU : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPFxd", "Ui",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter_sxtw">;

// Store one vector (scalar base, unsigned vector offset in bytes)
def SVST1_SCATTER_64B_OFFSETS_U   : MInst<"svst1_scatter_[{3}]offset[_{d}]",  "vPpud", "lUld", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1_scatter">;
def SVST1B_SCATTER_64B_OFFSETS_US : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPAud", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter">;
def SVST1B_SCATTER_64B_OFFSETS_UU : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPEud", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter">;
def SVST1H_SCATTER_64B_OFFSETS_US : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPBud", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter">;
def SVST1H_SCATTER_64B_OFFSETS_UU : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPFud", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter">;
def SVST1W_SCATTER_64B_OFFSETS_US : MInst<"svst1w_scatter_[{3}]offset[_{d}]", "vPCud", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_st1_scatter">;
def SVST1W_SCATTER_64B_OFFSETS_UU : MInst<"svst1w_scatter_[{3}]offset[_{d}]", "vPGud", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_st1_scatter">;

def SVST1_SCATTER_32B_OFFSETS_U   : MInst<"svst1_scatter_[{3}]offset[_{d}]",  "vPpud", "iUif", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1_scatter_uxtw">;
````
- **L337 EN**: Declares TableGen def record `SVST1H_SCATTER_BASES_U`.
  **L337 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_BASES_U`。
- **L338 EN**: Declares TableGen def record `SVST1W_SCATTER_BASES_U`.
  **L338 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_BASES_U`。
- **L339 EN**: Blank line separating nearby declarations or logic blocks.
  **L339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L340 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (scalar base, signed vector offset in bytes)`.
  **L340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (scalar base, signed vector offset in bytes)`。
- **L341 EN**: Declares TableGen def record `SVST1_SCATTER_64B_OFFSETS_S`.
  **L341 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_64B_OFFSETS_S`。
- **L342 EN**: Declares TableGen def record `SVST1B_SCATTER_64B_OFFSETS_SS`.
  **L342 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_64B_OFFSETS_SS`。
- **L343 EN**: Declares TableGen def record `SVST1B_SCATTER_64B_OFFSETS_SU`.
  **L343 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_64B_OFFSETS_SU`。
- **L344 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_OFFSETS_SS`.
  **L344 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_OFFSETS_SS`。
- **L345 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_OFFSETS_SU`.
  **L345 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_OFFSETS_SU`。
- **L346 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_OFFSETS_SS`.
  **L346 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_OFFSETS_SS`。
- **L347 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_OFFSETS_SU`.
  **L347 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_OFFSETS_SU`。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Declares TableGen def record `SVST1_SCATTER_32B_OFFSETS_S`.
  **L349 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_32B_OFFSETS_S`。
- **L350 EN**: Declares TableGen def record `SVST1B_SCATTER_32B_OFFSETS_SS`.
  **L350 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_32B_OFFSETS_SS`。
- **L351 EN**: Declares TableGen def record `SVST1B_SCATTER_32B_OFFSETS_SU`.
  **L351 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_32B_OFFSETS_SU`。
- **L352 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_OFFSETS_SS`.
  **L352 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_OFFSETS_SS`。
- **L353 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_OFFSETS_SU`.
  **L353 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_OFFSETS_SU`。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (scalar base, unsigned vector offset in bytes)`.
  **L355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (scalar base, unsigned vector offset in bytes)`。
- **L356 EN**: Declares TableGen def record `SVST1_SCATTER_64B_OFFSETS_U`.
  **L356 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_64B_OFFSETS_U`。
- **L357 EN**: Declares TableGen def record `SVST1B_SCATTER_64B_OFFSETS_US`.
  **L357 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_64B_OFFSETS_US`。
- **L358 EN**: Declares TableGen def record `SVST1B_SCATTER_64B_OFFSETS_UU`.
  **L358 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_64B_OFFSETS_UU`。
- **L359 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_OFFSETS_US`.
  **L359 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_OFFSETS_US`。
- **L360 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_OFFSETS_UU`.
  **L360 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_OFFSETS_UU`。
- **L361 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_OFFSETS_US`.
  **L361 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_OFFSETS_US`。
- **L362 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_OFFSETS_UU`.
  **L362 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_OFFSETS_UU`。
- **L363 EN**: Blank line separating nearby declarations or logic blocks.
  **L363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L364 EN**: Declares TableGen def record `SVST1_SCATTER_32B_OFFSETS_U`.
  **L364 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_32B_OFFSETS_U`。

### Lines 365-392

````tablegen
def SVST1B_SCATTER_32B_OFFSETS_US : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPAud", "i",    [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter_uxtw">;
def SVST1B_SCATTER_32B_OFFSETS_UU : MInst<"svst1b_scatter_[{3}]offset[_{d}]", "vPEud", "Ui",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter_uxtw">;
def SVST1H_SCATTER_32B_OFFSETS_US : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPBud", "i",    [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter_uxtw">;
def SVST1H_SCATTER_32B_OFFSETS_UU : MInst<"svst1h_scatter_[{3}]offset[_{d}]", "vPFud", "Ui",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter_uxtw">;

// Store one vector (vector base, signed scalar offset in bytes)
def SVST1_SCATTER_OFFSET_S    : MInst<"svst1_scatter[_{2}base]_offset[_{d}]",  "vPuld", "ilUiUlfd", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1_scatter_scalar_offset">;
def SVST1B_SCATTER_OFFSET_S   : MInst<"svst1b_scatter[_{2}base]_offset[_{d}]", "vPuld", "ilUiUl",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_st1_scatter_scalar_offset">;
def SVST1H_SCATTER_OFFSET_S   : MInst<"svst1h_scatter[_{2}base]_offset[_{d}]", "vPuld", "ilUiUl",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_st1_scatter_scalar_offset">;
def SVST1W_SCATTER_OFFSET_S   : MInst<"svst1w_scatter[_{2}base]_offset[_{d}]", "vPuld", "lUl",      [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_st1_scatter_scalar_offset">;

// Store one vector (scalar base, signed vector index)
def SVST1_SCATTER_64B_INDICES_S   : MInst<"svst1_scatter_[{3}]index[_{d}]",  "vPpxd", "lUld", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1_scatter_index">;
def SVST1H_SCATTER_64B_INDICES_SS : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPBxd", "l",    [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_index">;
def SVST1H_SCATTER_64B_INDICES_SU : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPFxd", "Ul",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_index">;
def SVST1W_SCATTER_64B_INDICES_SS : MInst<"svst1w_scatter_[{3}]index[_{d}]", "vPCxd", "l",    [IsScatterStore], MemEltTyInt32,   "aarch64_sve_st1_scatter_index">;
def SVST1W_SCATTER_64B_INDICES_SU : MInst<"svst1w_scatter_[{3}]index[_{d}]", "vPGxd", "Ul",   [IsScatterStore], MemEltTyInt32,   "aarch64_sve_st1_scatter_index">;

def SVST1_SCATTER_32B_INDICES_S   : MInst<"svst1_scatter_[{3}]index[_{d}]",  "vPpxd", "iUif", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1_scatter_sxtw_index">;
def SVST1H_SCATTER_32B_INDICES_SS : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPBxd", "i",    [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_sxtw_index">;
def SVST1H_SCATTER_32B_INDICES_SU : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPFxd", "Ui",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_sxtw_index">;

// Store one vector (scalar base, unsigned vector index)
def SVST1_SCATTER_64B_INDICES_U   : MInst<"svst1_scatter_[{3}]index[_{d}]",  "vPpud", "lUld", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1_scatter_index">;
def SVST1H_SCATTER_64B_INDICES_US : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPBud", "l",    [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_index">;
def SVST1H_SCATTER_64B_INDICES_UU : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPFud", "Ul",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_index">;
def SVST1W_SCATTER_64B_INDICES_US : MInst<"svst1w_scatter_[{3}]index[_{d}]", "vPCud", "l",    [IsScatterStore], MemEltTyInt32,   "aarch64_sve_st1_scatter_index">;
def SVST1W_SCATTER_64B_INDICES_UU : MInst<"svst1w_scatter_[{3}]index[_{d}]", "vPGud", "Ul",   [IsScatterStore], MemEltTyInt32,   "aarch64_sve_st1_scatter_index">;
````
- **L365 EN**: Declares TableGen def record `SVST1B_SCATTER_32B_OFFSETS_US`.
  **L365 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_32B_OFFSETS_US`。
- **L366 EN**: Declares TableGen def record `SVST1B_SCATTER_32B_OFFSETS_UU`.
  **L366 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_32B_OFFSETS_UU`。
- **L367 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_OFFSETS_US`.
  **L367 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_OFFSETS_US`。
- **L368 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_OFFSETS_UU`.
  **L368 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_OFFSETS_UU`。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L370 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (vector base, signed scalar offset in bytes)`.
  **L370 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (vector base, signed scalar offset in bytes)`。
- **L371 EN**: Declares TableGen def record `SVST1_SCATTER_OFFSET_S`.
  **L371 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_OFFSET_S`。
- **L372 EN**: Declares TableGen def record `SVST1B_SCATTER_OFFSET_S`.
  **L372 CN**: 声明 TableGen def 记录 `SVST1B_SCATTER_OFFSET_S`。
- **L373 EN**: Declares TableGen def record `SVST1H_SCATTER_OFFSET_S`.
  **L373 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_OFFSET_S`。
- **L374 EN**: Declares TableGen def record `SVST1W_SCATTER_OFFSET_S`.
  **L374 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_OFFSET_S`。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L376 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (scalar base, signed vector index)`.
  **L376 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (scalar base, signed vector index)`。
- **L377 EN**: Declares TableGen def record `SVST1_SCATTER_64B_INDICES_S`.
  **L377 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_64B_INDICES_S`。
- **L378 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_INDICES_SS`.
  **L378 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_INDICES_SS`。
- **L379 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_INDICES_SU`.
  **L379 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_INDICES_SU`。
- **L380 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_INDICES_SS`.
  **L380 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_INDICES_SS`。
- **L381 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_INDICES_SU`.
  **L381 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_INDICES_SU`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L383 EN**: Declares TableGen def record `SVST1_SCATTER_32B_INDICES_S`.
  **L383 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_32B_INDICES_S`。
- **L384 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_INDICES_SS`.
  **L384 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_INDICES_SS`。
- **L385 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_INDICES_SU`.
  **L385 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_INDICES_SU`。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L387 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (scalar base, unsigned vector index)`.
  **L387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (scalar base, unsigned vector index)`。
- **L388 EN**: Declares TableGen def record `SVST1_SCATTER_64B_INDICES_U`.
  **L388 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_64B_INDICES_U`。
- **L389 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_INDICES_US`.
  **L389 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_INDICES_US`。
- **L390 EN**: Declares TableGen def record `SVST1H_SCATTER_64B_INDICES_UU`.
  **L390 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_64B_INDICES_UU`。
- **L391 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_INDICES_US`.
  **L391 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_INDICES_US`。
- **L392 EN**: Declares TableGen def record `SVST1W_SCATTER_64B_INDICES_UU`.
  **L392 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_64B_INDICES_UU`。

### Lines 393-420

````tablegen

def SVST1_SCATTER_32B_INDICES_U   : MInst<"svst1_scatter_[{3}]index[_{d}]",  "vPpud", "iUif", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1_scatter_uxtw_index">;
def SVST1H_SCATTER_32B_INDICES_US : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPBud", "i",    [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_uxtw_index">;
def SVST1H_SCATTER_32B_INDICES_UU : MInst<"svst1h_scatter_[{3}]index[_{d}]", "vPFud", "Ui",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_uxtw_index">;

// Store one vector (vector base, signed scalar index)
def SVST1_SCATTER_INDEX_S     : MInst<"svst1_scatter[_{2}base]_index[_{d}]",  "vPuld", "ilUiUlfd", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1_scatter_scalar_offset">;
def SVST1H_SCATTER_INDEX_S    : MInst<"svst1h_scatter[_{2}base]_index[_{d}]", "vPuld", "ilUiUl",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_st1_scatter_scalar_offset">;
def SVST1W_SCATTER_INDEX_S    : MInst<"svst1w_scatter[_{2}base]_index[_{d}]", "vPuld", "lUl",      [IsScatterStore], MemEltTyInt32,   "aarch64_sve_st1_scatter_scalar_offset">;
}

// Store N vectors into N-element structure (scalar base)
def SVST2 : SInst<"svst2[_{d}]", "vPp2", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st2", [IsStructStore, VerifyRuntimeMode]>;
def SVST3 : SInst<"svst3[_{d}]", "vPp3", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st3", [IsStructStore, VerifyRuntimeMode]>;
def SVST4 : SInst<"svst4[_{d}]", "vPp4", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st4", [IsStructStore, VerifyRuntimeMode]>;

// Store N vectors into N-element structure (scalar base, VL displacement)
def SVST2_VNUM : SInst<"svst2_vnum[_{d}]", "vPpl2", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st2", [IsStructStore, VerifyRuntimeMode]>;
def SVST3_VNUM : SInst<"svst3_vnum[_{d}]", "vPpl3", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st3", [IsStructStore, VerifyRuntimeMode]>;
def SVST4_VNUM : SInst<"svst4_vnum[_{d}]", "vPpl4", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st4", [IsStructStore, VerifyRuntimeMode]>;

// Store one vector, with no truncation, non-temporal (scalar base)
def SVSTNT1 : MInst<"svstnt1[_{d}]", "vPpd", "csilUcUsUiUlhfdbm", [IsStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_stnt1">;

// Store one vector, with no truncation, non-temporal (scalar base, VL displacement)
def SVSTNT1_VNUM : MInst<"svstnt1_vnum[_{d}]", "vPpld", "csilUcUsUiUlhfdbm", [IsStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_stnt1">;

let SVETargetGuard = "sve2p1", SMETargetGuard = InvalidMode in {
````
- **L393 EN**: Blank line separating nearby declarations or logic blocks.
  **L393 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L394 EN**: Declares TableGen def record `SVST1_SCATTER_32B_INDICES_U`.
  **L394 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_32B_INDICES_U`。
- **L395 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_INDICES_US`.
  **L395 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_INDICES_US`。
- **L396 EN**: Declares TableGen def record `SVST1H_SCATTER_32B_INDICES_UU`.
  **L396 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_32B_INDICES_UU`。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L398 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (vector base, signed scalar index)`.
  **L398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (vector base, signed scalar index)`。
- **L399 EN**: Declares TableGen def record `SVST1_SCATTER_INDEX_S`.
  **L399 CN**: 声明 TableGen def 记录 `SVST1_SCATTER_INDEX_S`。
- **L400 EN**: Declares TableGen def record `SVST1H_SCATTER_INDEX_S`.
  **L400 CN**: 声明 TableGen def 记录 `SVST1H_SCATTER_INDEX_S`。
- **L401 EN**: Declares TableGen def record `SVST1W_SCATTER_INDEX_S`.
  **L401 CN**: 声明 TableGen def 记录 `SVST1W_SCATTER_INDEX_S`。
- **L402 EN**: Closes the current lexical scope or compound statement.
  **L402 CN**: 结束当前词法作用域或复合语句块。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L404 EN**: Comment explains nearby logic, constraints, or intent: `Store N vectors into N-element structure (scalar base)`.
  **L404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store N vectors into N-element structure (scalar base)`。
- **L405 EN**: Declares TableGen def record `SVST2`.
  **L405 CN**: 声明 TableGen def 记录 `SVST2`。
- **L406 EN**: Declares TableGen def record `SVST3`.
  **L406 CN**: 声明 TableGen def 记录 `SVST3`。
- **L407 EN**: Declares TableGen def record `SVST4`.
  **L407 CN**: 声明 TableGen def 记录 `SVST4`。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L409 EN**: Comment explains nearby logic, constraints, or intent: `Store N vectors into N-element structure (scalar base, VL displacement)`.
  **L409 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store N vectors into N-element structure (scalar base, VL displacement)`。
- **L410 EN**: Declares TableGen def record `SVST2_VNUM`.
  **L410 CN**: 声明 TableGen def 记录 `SVST2_VNUM`。
- **L411 EN**: Declares TableGen def record `SVST3_VNUM`.
  **L411 CN**: 声明 TableGen def 记录 `SVST3_VNUM`。
- **L412 EN**: Declares TableGen def record `SVST4_VNUM`.
  **L412 CN**: 声明 TableGen def 记录 `SVST4_VNUM`。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector, with no truncation, non-temporal (scalar base)`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector, with no truncation, non-temporal (scalar base)`。
- **L415 EN**: Declares TableGen def record `SVSTNT1`.
  **L415 CN**: 声明 TableGen def 记录 `SVSTNT1`。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector, with no truncation, non-temporal (scalar base, VL displacement)`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector, with no truncation, non-temporal (scalar base, VL displacement)`。
- **L418 EN**: Declares TableGen def record `SVSTNT1_VNUM`.
  **L418 CN**: 声明 TableGen def 记录 `SVSTNT1_VNUM`。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L420 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1", SMETargetGuard = InvalidMode in {`.
  **L420 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1", SMETargetGuard = InvalidMode in {`。

### Lines 421-448

````tablegen
  // Contiguous truncating store from quadword (single vector).
  def SVST1UWQ      : MInst<"svst1wq[_{d}]",       "vPpd", "iUif", [IsStore], MemEltTyInt32, "aarch64_sve_st1wq">;
  def SVST1UWQ_VNUM : MInst<"svst1wq_vnum[_{d}]", "vPpld", "iUif", [IsStore], MemEltTyInt32, "aarch64_sve_st1wq">;

  def SVST1UDQ      : MInst<"svst1dq[_{d}]",      "vPpd", "lUld",  [IsStore], MemEltTyInt64, "aarch64_sve_st1dq">;
  def SVST1UDQ_VNUM : MInst<"svst1dq_vnum[_{d}]", "vPpld", "lUld", [IsStore], MemEltTyInt64, "aarch64_sve_st1dq">;

  // Store one vector (vector base + scalar offset)
  def SVST1Q_SCATTER_U64BASE_OFFSET : MInst<"svst1q_scatter[_{2}base]_offset[_{d}]",  "vPgld", "cUcsUsiUilUlfhdbm", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1q_scatter_scalar_offset">;
  def SVST1Q_SCATTER_U64BASE :        MInst<"svst1q_scatter[_{2}base][_{d}]",          "vPgd", "cUcsUsiUilUlfhdbm", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1q_scatter_scalar_offset">;

  // Store one vector (scalar base + vector offset)
  def SVST1Q_SCATTER_OFFSETS_U : MInst<"svst1q_scatter_[{3}]offset[_{d}]", "vPpgd", "cUcsUsiUilUlfhdbm", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1q_scatter_vector_offset">;
  def SVST1Q_SCATTER_OFFSETS_S : MInst<"svst1q_scatter_[{3}]offset[_{d}]", "vPp#d", "cUcsUsiUilUlfhdbm", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_st1q_scatter_vector_offset">;

  // Scatter store quadwords (scalar base + vector index)
  def SVST1Q_SCATTER_INDICES_U : MInst<"svst1q_scatter_[{3}]index[_{d}]", "vPpgd", "sUsiUilUlbhfd", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1q_scatter_index">;
  def SVST1Q_SCATTER_INDICES_S : MInst<"svst1q_scatter_[{3}]index[_{d}]", "vPp#d", "sUsiUilUlbhfd", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1q_scatter_index">;

  // Scatter store quadwords (vector base + scalar index)
  def SVST1Q_SCATTER_INDEX_S   : MInst<"svst1q_scatter[_{2}base]_index[_{d}]", "vPgld", "sUsiUilUlbhfd", [IsScatterStore], MemEltTyDefault, "aarch64_sve_st1q_scatter_scalar_offset">;
}

let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {
  // Store N vectors into N-element structure (scalar base)
  def SVST2Q : SInst<"svst2q[_{d}]", "vPc2", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st2q", [IsStructStore, VerifyRuntimeMode]>;
  def SVST3Q : SInst<"svst3q[_{d}]", "vPc3", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st3q", [IsStructStore, VerifyRuntimeMode]>;
  def SVST4Q : SInst<"svst4q[_{d}]", "vPc4", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st4q", [IsStructStore, VerifyRuntimeMode]>;
````
- **L421 EN**: Comment explains nearby logic, constraints, or intent: `Contiguous truncating store from quadword (single vector).`.
  **L421 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Contiguous truncating store from quadword (single vector).`。
- **L422 EN**: Declares TableGen def record `SVST1UWQ`.
  **L422 CN**: 声明 TableGen def 记录 `SVST1UWQ`。
- **L423 EN**: Declares TableGen def record `SVST1UWQ_VNUM`.
  **L423 CN**: 声明 TableGen def 记录 `SVST1UWQ_VNUM`。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L425 EN**: Declares TableGen def record `SVST1UDQ`.
  **L425 CN**: 声明 TableGen def 记录 `SVST1UDQ`。
- **L426 EN**: Declares TableGen def record `SVST1UDQ_VNUM`.
  **L426 CN**: 声明 TableGen def 记录 `SVST1UDQ_VNUM`。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L428 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (vector base + scalar offset)`.
  **L428 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (vector base + scalar offset)`。
- **L429 EN**: Declares TableGen def record `SVST1Q_SCATTER_U64BASE_OFFSET`.
  **L429 CN**: 声明 TableGen def 记录 `SVST1Q_SCATTER_U64BASE_OFFSET`。
- **L430 EN**: Declares TableGen def record `SVST1Q_SCATTER_U64BASE`.
  **L430 CN**: 声明 TableGen def 记录 `SVST1Q_SCATTER_U64BASE`。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Comment explains nearby logic, constraints, or intent: `Store one vector (scalar base + vector offset)`.
  **L432 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store one vector (scalar base + vector offset)`。
- **L433 EN**: Declares TableGen def record `SVST1Q_SCATTER_OFFSETS_U`.
  **L433 CN**: 声明 TableGen def 记录 `SVST1Q_SCATTER_OFFSETS_U`。
- **L434 EN**: Declares TableGen def record `SVST1Q_SCATTER_OFFSETS_S`.
  **L434 CN**: 声明 TableGen def 记录 `SVST1Q_SCATTER_OFFSETS_S`。
- **L435 EN**: Blank line separating nearby declarations or logic blocks.
  **L435 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L436 EN**: Comment explains nearby logic, constraints, or intent: `Scatter store quadwords (scalar base + vector index)`.
  **L436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scatter store quadwords (scalar base + vector index)`。
- **L437 EN**: Declares TableGen def record `SVST1Q_SCATTER_INDICES_U`.
  **L437 CN**: 声明 TableGen def 记录 `SVST1Q_SCATTER_INDICES_U`。
- **L438 EN**: Declares TableGen def record `SVST1Q_SCATTER_INDICES_S`.
  **L438 CN**: 声明 TableGen def 记录 `SVST1Q_SCATTER_INDICES_S`。
- **L439 EN**: Blank line separating nearby declarations or logic blocks.
  **L439 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L440 EN**: Comment explains nearby logic, constraints, or intent: `Scatter store quadwords (vector base + scalar index)`.
  **L440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scatter store quadwords (vector base + scalar index)`。
- **L441 EN**: Declares TableGen def record `SVST1Q_SCATTER_INDEX_S`.
  **L441 CN**: 声明 TableGen def 记录 `SVST1Q_SCATTER_INDEX_S`。
- **L442 EN**: Closes the current lexical scope or compound statement.
  **L442 CN**: 结束当前词法作用域或复合语句块。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L444 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`.
  **L444 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`。
- **L445 EN**: Comment explains nearby logic, constraints, or intent: `Store N vectors into N-element structure (scalar base)`.
  **L445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store N vectors into N-element structure (scalar base)`。
- **L446 EN**: Declares TableGen def record `SVST2Q`.
  **L446 CN**: 声明 TableGen def 记录 `SVST2Q`。
- **L447 EN**: Declares TableGen def record `SVST3Q`.
  **L447 CN**: 声明 TableGen def 记录 `SVST3Q`。
- **L448 EN**: Declares TableGen def record `SVST4Q`.
  **L448 CN**: 声明 TableGen def 记录 `SVST4Q`。

### Lines 449-476

````tablegen

  // Store N vectors into N-element structure (scalar base, VL displacement)
  def SVST2Q_VNUM : SInst<"svst2q_vnum[_{d}]", "vPcl2", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st2q", [IsStructStore, VerifyRuntimeMode]>;
  def SVST3Q_VNUM : SInst<"svst3q_vnum[_{d}]", "vPcl3", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st3q", [IsStructStore, VerifyRuntimeMode]>;
  def SVST4Q_VNUM : SInst<"svst4q_vnum[_{d}]", "vPcl4", "csilUcUsUiUlhfdbm", MergeNone, "aarch64_sve_st4q", [IsStructStore, VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// Prefetches

// Prefetch (Scalar base)
def SVPRFB : MInst<"svprfb", "vPQJ", "c", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt8,  "aarch64_sve_prf">;
def SVPRFH : MInst<"svprfh", "vPQJ", "s", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt16, "aarch64_sve_prf">;
def SVPRFW : MInst<"svprfw", "vPQJ", "i", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt32, "aarch64_sve_prf">;
def SVPRFD : MInst<"svprfd", "vPQJ", "l", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt64, "aarch64_sve_prf">;

// Prefetch (Scalar base, VL displacement)
def SVPRFB_VNUM : MInst<"svprfb_vnum", "vPQlJ", "c", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt8,  "aarch64_sve_prf">;
def SVPRFH_VNUM : MInst<"svprfh_vnum", "vPQlJ", "s", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt16, "aarch64_sve_prf">;
def SVPRFW_VNUM : MInst<"svprfw_vnum", "vPQlJ", "i", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt32, "aarch64_sve_prf">;
def SVPRFD_VNUM : MInst<"svprfd_vnum", "vPQlJ", "l", [IsPrefetch, VerifyRuntimeMode], MemEltTyInt64, "aarch64_sve_prf">;

let SMETargetGuard = InvalidMode in {
// Prefetch (Vector bases)
def SVPRFB_GATHER_BASES : MInst<"svprfb_gather[_{2}base]", "vPdJ", "UiUl", [IsGatherPrefetch], MemEltTyInt8,  "aarch64_sve_prfb_gather_scalar_offset">;
def SVPRFH_GATHER_BASES : MInst<"svprfh_gather[_{2}base]", "vPdJ", "UiUl", [IsGatherPrefetch], MemEltTyInt16, "aarch64_sve_prfh_gather_scalar_offset">;
def SVPRFW_GATHER_BASES : MInst<"svprfw_gather[_{2}base]", "vPdJ", "UiUl", [IsGatherPrefetch], MemEltTyInt32, "aarch64_sve_prfw_gather_scalar_offset">;
def SVPRFD_GATHER_BASES : MInst<"svprfd_gather[_{2}base]", "vPdJ", "UiUl", [IsGatherPrefetch], MemEltTyInt64, "aarch64_sve_prfd_gather_scalar_offset">;
````
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L450 EN**: Comment explains nearby logic, constraints, or intent: `Store N vectors into N-element structure (scalar base, VL displacement)`.
  **L450 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store N vectors into N-element structure (scalar base, VL displacement)`。
- **L451 EN**: Declares TableGen def record `SVST2Q_VNUM`.
  **L451 CN**: 声明 TableGen def 记录 `SVST2Q_VNUM`。
- **L452 EN**: Declares TableGen def record `SVST3Q_VNUM`.
  **L452 CN**: 声明 TableGen def 记录 `SVST3Q_VNUM`。
- **L453 EN**: Declares TableGen def record `SVST4Q_VNUM`.
  **L453 CN**: 声明 TableGen def 记录 `SVST4Q_VNUM`。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L456 EN**: Separator comment used for visual grouping.
  **L456 CN**: 用于视觉分组的分隔注释。
- **L457 EN**: Comment explains nearby logic, constraints, or intent: `Prefetches`.
  **L457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetches`。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L459 EN**: Comment explains nearby logic, constraints, or intent: `Prefetch (Scalar base)`.
  **L459 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetch (Scalar base)`。
- **L460 EN**: Declares TableGen def record `SVPRFB`.
  **L460 CN**: 声明 TableGen def 记录 `SVPRFB`。
- **L461 EN**: Declares TableGen def record `SVPRFH`.
  **L461 CN**: 声明 TableGen def 记录 `SVPRFH`。
- **L462 EN**: Declares TableGen def record `SVPRFW`.
  **L462 CN**: 声明 TableGen def 记录 `SVPRFW`。
- **L463 EN**: Declares TableGen def record `SVPRFD`.
  **L463 CN**: 声明 TableGen def 记录 `SVPRFD`。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Comment explains nearby logic, constraints, or intent: `Prefetch (Scalar base, VL displacement)`.
  **L465 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetch (Scalar base, VL displacement)`。
- **L466 EN**: Declares TableGen def record `SVPRFB_VNUM`.
  **L466 CN**: 声明 TableGen def 记录 `SVPRFB_VNUM`。
- **L467 EN**: Declares TableGen def record `SVPRFH_VNUM`.
  **L467 CN**: 声明 TableGen def 记录 `SVPRFH_VNUM`。
- **L468 EN**: Declares TableGen def record `SVPRFW_VNUM`.
  **L468 CN**: 声明 TableGen def 记录 `SVPRFW_VNUM`。
- **L469 EN**: Declares TableGen def record `SVPRFD_VNUM`.
  **L469 CN**: 声明 TableGen def 记录 `SVPRFD_VNUM`。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L471 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L472 EN**: Comment explains nearby logic, constraints, or intent: `Prefetch (Vector bases)`.
  **L472 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetch (Vector bases)`。
- **L473 EN**: Declares TableGen def record `SVPRFB_GATHER_BASES`.
  **L473 CN**: 声明 TableGen def 记录 `SVPRFB_GATHER_BASES`。
- **L474 EN**: Declares TableGen def record `SVPRFH_GATHER_BASES`.
  **L474 CN**: 声明 TableGen def 记录 `SVPRFH_GATHER_BASES`。
- **L475 EN**: Declares TableGen def record `SVPRFW_GATHER_BASES`.
  **L475 CN**: 声明 TableGen def 记录 `SVPRFW_GATHER_BASES`。
- **L476 EN**: Declares TableGen def record `SVPRFD_GATHER_BASES`.
  **L476 CN**: 声明 TableGen def 记录 `SVPRFD_GATHER_BASES`。

### Lines 477-504

````tablegen

// Prefetch (Scalar base, Vector offsets)
def SVPRFB_GATHER_32B_OFFSETS_S : MInst<"svprfb_gather_[{3}]offset", "vPQdJ", "i",  [IsGatherPrefetch], MemEltTyInt8,  "aarch64_sve_prfb_gather_sxtw_index">;
def SVPRFH_GATHER_32B_OFFSETS_S : MInst<"svprfh_gather_[{3}]index",  "vPQdJ", "i",  [IsGatherPrefetch], MemEltTyInt16, "aarch64_sve_prfh_gather_sxtw_index">;
def SVPRFW_GATHER_32B_OFFSETS_S : MInst<"svprfw_gather_[{3}]index",  "vPQdJ", "i",  [IsGatherPrefetch], MemEltTyInt32, "aarch64_sve_prfw_gather_sxtw_index">;
def SVPRFD_GATHER_32B_OFFSETS_S : MInst<"svprfd_gather_[{3}]index",  "vPQdJ", "i",  [IsGatherPrefetch], MemEltTyInt64, "aarch64_sve_prfd_gather_sxtw_index">;

def SVPRFB_GATHER_64B_OFFSETS_S : MInst<"svprfb_gather_[{3}]offset", "vPQdJ", "l",  [IsGatherPrefetch], MemEltTyInt8,  "aarch64_sve_prfb_gather_index">;
def SVPRFH_GATHER_64B_OFFSETS_S : MInst<"svprfh_gather_[{3}]index",  "vPQdJ", "l",  [IsGatherPrefetch], MemEltTyInt16, "aarch64_sve_prfh_gather_index">;
def SVPRFW_GATHER_64B_OFFSETS_S : MInst<"svprfw_gather_[{3}]index",  "vPQdJ", "l",  [IsGatherPrefetch], MemEltTyInt32, "aarch64_sve_prfw_gather_index">;
def SVPRFD_GATHER_64B_OFFSETS_S : MInst<"svprfd_gather_[{3}]index",  "vPQdJ", "l",  [IsGatherPrefetch], MemEltTyInt64, "aarch64_sve_prfd_gather_index">;

def SVPRFB_GATHER_32B_OFFSETS_U : MInst<"svprfb_gather_[{3}]offset", "vPQdJ", "Ui", [IsGatherPrefetch], MemEltTyInt8,  "aarch64_sve_prfb_gather_uxtw_index">;
def SVPRFH_GATHER_32B_OFFSETS_U : MInst<"svprfh_gather_[{3}]index",  "vPQdJ", "Ui", [IsGatherPrefetch], MemEltTyInt16, "aarch64_sve_prfh_gather_uxtw_index">;
def SVPRFW_GATHER_32B_OFFSETS_U : MInst<"svprfw_gather_[{3}]index",  "vPQdJ", "Ui", [IsGatherPrefetch], MemEltTyInt32, "aarch64_sve_prfw_gather_uxtw_index">;
def SVPRFD_GATHER_32B_OFFSETS_U : MInst<"svprfd_gather_[{3}]index",  "vPQdJ", "Ui", [IsGatherPrefetch], MemEltTyInt64, "aarch64_sve_prfd_gather_uxtw_index">;

def SVPRFB_GATHER_64B_OFFSETS_U : MInst<"svprfb_gather_[{3}]offset", "vPQdJ", "Ul", [IsGatherPrefetch], MemEltTyInt8,  "aarch64_sve_prfb_gather_index">;
def SVPRFH_GATHER_64B_OFFSETS_U : MInst<"svprfh_gather_[{3}]index",  "vPQdJ", "Ul", [IsGatherPrefetch], MemEltTyInt16, "aarch64_sve_prfh_gather_index">;
def SVPRFW_GATHER_64B_OFFSETS_U : MInst<"svprfw_gather_[{3}]index",  "vPQdJ", "Ul", [IsGatherPrefetch], MemEltTyInt32, "aarch64_sve_prfw_gather_index">;
def SVPRFD_GATHER_64B_OFFSETS_U : MInst<"svprfd_gather_[{3}]index",  "vPQdJ", "Ul", [IsGatherPrefetch], MemEltTyInt64, "aarch64_sve_prfd_gather_index">;

// Prefetch (Vector bases, scalar offset)
def SVPRFB_GATHER_BASES_OFFSET : MInst<"svprfb_gather[_{2}base]_offset", "vPdlJ", "UiUl", [IsGatherPrefetch], MemEltTyInt8,  "aarch64_sve_prfb_gather_scalar_offset">;
def SVPRFH_GATHER_BASES_OFFSET : MInst<"svprfh_gather[_{2}base]_index",  "vPdlJ", "UiUl", [IsGatherPrefetch], MemEltTyInt16, "aarch64_sve_prfh_gather_scalar_offset">;
def SVPRFW_GATHER_BASES_OFFSET : MInst<"svprfw_gather[_{2}base]_index",  "vPdlJ", "UiUl", [IsGatherPrefetch], MemEltTyInt32, "aarch64_sve_prfw_gather_scalar_offset">;
def SVPRFD_GATHER_BASES_OFFSET : MInst<"svprfd_gather[_{2}base]_index",  "vPdlJ", "UiUl", [IsGatherPrefetch], MemEltTyInt64, "aarch64_sve_prfd_gather_scalar_offset">;
}
````
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L478 EN**: Comment explains nearby logic, constraints, or intent: `Prefetch (Scalar base, Vector offsets)`.
  **L478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetch (Scalar base, Vector offsets)`。
- **L479 EN**: Declares TableGen def record `SVPRFB_GATHER_32B_OFFSETS_S`.
  **L479 CN**: 声明 TableGen def 记录 `SVPRFB_GATHER_32B_OFFSETS_S`。
- **L480 EN**: Declares TableGen def record `SVPRFH_GATHER_32B_OFFSETS_S`.
  **L480 CN**: 声明 TableGen def 记录 `SVPRFH_GATHER_32B_OFFSETS_S`。
- **L481 EN**: Declares TableGen def record `SVPRFW_GATHER_32B_OFFSETS_S`.
  **L481 CN**: 声明 TableGen def 记录 `SVPRFW_GATHER_32B_OFFSETS_S`。
- **L482 EN**: Declares TableGen def record `SVPRFD_GATHER_32B_OFFSETS_S`.
  **L482 CN**: 声明 TableGen def 记录 `SVPRFD_GATHER_32B_OFFSETS_S`。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Declares TableGen def record `SVPRFB_GATHER_64B_OFFSETS_S`.
  **L484 CN**: 声明 TableGen def 记录 `SVPRFB_GATHER_64B_OFFSETS_S`。
- **L485 EN**: Declares TableGen def record `SVPRFH_GATHER_64B_OFFSETS_S`.
  **L485 CN**: 声明 TableGen def 记录 `SVPRFH_GATHER_64B_OFFSETS_S`。
- **L486 EN**: Declares TableGen def record `SVPRFW_GATHER_64B_OFFSETS_S`.
  **L486 CN**: 声明 TableGen def 记录 `SVPRFW_GATHER_64B_OFFSETS_S`。
- **L487 EN**: Declares TableGen def record `SVPRFD_GATHER_64B_OFFSETS_S`.
  **L487 CN**: 声明 TableGen def 记录 `SVPRFD_GATHER_64B_OFFSETS_S`。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Declares TableGen def record `SVPRFB_GATHER_32B_OFFSETS_U`.
  **L489 CN**: 声明 TableGen def 记录 `SVPRFB_GATHER_32B_OFFSETS_U`。
- **L490 EN**: Declares TableGen def record `SVPRFH_GATHER_32B_OFFSETS_U`.
  **L490 CN**: 声明 TableGen def 记录 `SVPRFH_GATHER_32B_OFFSETS_U`。
- **L491 EN**: Declares TableGen def record `SVPRFW_GATHER_32B_OFFSETS_U`.
  **L491 CN**: 声明 TableGen def 记录 `SVPRFW_GATHER_32B_OFFSETS_U`。
- **L492 EN**: Declares TableGen def record `SVPRFD_GATHER_32B_OFFSETS_U`.
  **L492 CN**: 声明 TableGen def 记录 `SVPRFD_GATHER_32B_OFFSETS_U`。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L494 EN**: Declares TableGen def record `SVPRFB_GATHER_64B_OFFSETS_U`.
  **L494 CN**: 声明 TableGen def 记录 `SVPRFB_GATHER_64B_OFFSETS_U`。
- **L495 EN**: Declares TableGen def record `SVPRFH_GATHER_64B_OFFSETS_U`.
  **L495 CN**: 声明 TableGen def 记录 `SVPRFH_GATHER_64B_OFFSETS_U`。
- **L496 EN**: Declares TableGen def record `SVPRFW_GATHER_64B_OFFSETS_U`.
  **L496 CN**: 声明 TableGen def 记录 `SVPRFW_GATHER_64B_OFFSETS_U`。
- **L497 EN**: Declares TableGen def record `SVPRFD_GATHER_64B_OFFSETS_U`.
  **L497 CN**: 声明 TableGen def 记录 `SVPRFD_GATHER_64B_OFFSETS_U`。
- **L498 EN**: Blank line separating nearby declarations or logic blocks.
  **L498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L499 EN**: Comment explains nearby logic, constraints, or intent: `Prefetch (Vector bases, scalar offset)`.
  **L499 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetch (Vector bases, scalar offset)`。
- **L500 EN**: Declares TableGen def record `SVPRFB_GATHER_BASES_OFFSET`.
  **L500 CN**: 声明 TableGen def 记录 `SVPRFB_GATHER_BASES_OFFSET`。
- **L501 EN**: Declares TableGen def record `SVPRFH_GATHER_BASES_OFFSET`.
  **L501 CN**: 声明 TableGen def 记录 `SVPRFH_GATHER_BASES_OFFSET`。
- **L502 EN**: Declares TableGen def record `SVPRFW_GATHER_BASES_OFFSET`.
  **L502 CN**: 声明 TableGen def 记录 `SVPRFW_GATHER_BASES_OFFSET`。
- **L503 EN**: Declares TableGen def record `SVPRFD_GATHER_BASES_OFFSET`.
  **L503 CN**: 声明 TableGen def 记录 `SVPRFD_GATHER_BASES_OFFSET`。
- **L504 EN**: Closes the current lexical scope or compound statement.
  **L504 CN**: 结束当前词法作用域或复合语句块。

### Lines 505-532

````tablegen

////////////////////////////////////////////////////////////////////////////////
// Address calculations

let SMETargetGuard = InvalidMode in {
def SVADRB : SInst<"svadrb[_{0}base]_[{2}]offset", "uud", "ilUiUl", MergeNone, "aarch64_sve_adrb">;
def SVADRH : SInst<"svadrh[_{0}base]_[{2}]index",  "uud", "ilUiUl", MergeNone, "aarch64_sve_adrh">;
def SVADRW : SInst<"svadrw[_{0}base]_[{2}]index",  "uud", "ilUiUl", MergeNone, "aarch64_sve_adrw">;
def SVADRD : SInst<"svadrd[_{0}base]_[{2}]index",  "uud", "ilUiUl", MergeNone, "aarch64_sve_adrd">;
}

////////////////////////////////////////////////////////////////////////////////
// Scalar to vector

def SVDUPQ_8  : SInst<"svdupq[_n]_{d}", "dssssssssssssssss",  "cUc", MergeNone, "", [VerifyRuntimeMode]>;
def SVDUPQ_16 : SInst<"svdupq[_n]_{d}", "dssssssss",  "sUshb", MergeNone, "", [VerifyRuntimeMode]>;
def SVDUPQ_32 : SInst<"svdupq[_n]_{d}", "dssss",  "iUif", MergeNone, "", [VerifyRuntimeMode]>;
def SVDUPQ_64 : SInst<"svdupq[_n]_{d}", "dss",  "lUld", MergeNone, "", [VerifyRuntimeMode]>;

multiclass svdup_base<string n, string p, MergeType mt, string i> {
  def NAME : SInst<n, p, "csilUcUsUiUlhfdb", mt, i, [VerifyRuntimeMode]>;
}

defm SVDUP   : svdup_base<"svdup[_n]_{d}", "ds",   MergeNone,    "aarch64_sve_dup_x">;
defm SVDUP_M : svdup_base<"svdup[_n]_{d}", "ddPs", MergeOp1,     "aarch64_sve_dup">;
defm SVDUP_X : svdup_base<"svdup[_n]_{d}", "dPs",  MergeAnyExp,  "aarch64_sve_dup">;
defm SVDUP_Z : svdup_base<"svdup[_n]_{d}", "dPs",  MergeZeroExp, "aarch64_sve_dup">;

````
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L506 EN**: Separator comment used for visual grouping.
  **L506 CN**: 用于视觉分组的分隔注释。
- **L507 EN**: Comment explains nearby logic, constraints, or intent: `Address calculations`.
  **L507 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Address calculations`。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L509 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L509 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L510 EN**: Declares TableGen def record `SVADRB`.
  **L510 CN**: 声明 TableGen def 记录 `SVADRB`。
- **L511 EN**: Declares TableGen def record `SVADRH`.
  **L511 CN**: 声明 TableGen def 记录 `SVADRH`。
- **L512 EN**: Declares TableGen def record `SVADRW`.
  **L512 CN**: 声明 TableGen def 记录 `SVADRW`。
- **L513 EN**: Declares TableGen def record `SVADRD`.
  **L513 CN**: 声明 TableGen def 记录 `SVADRD`。
- **L514 EN**: Closes the current lexical scope or compound statement.
  **L514 CN**: 结束当前词法作用域或复合语句块。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L516 EN**: Separator comment used for visual grouping.
  **L516 CN**: 用于视觉分组的分隔注释。
- **L517 EN**: Comment explains nearby logic, constraints, or intent: `Scalar to vector`.
  **L517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar to vector`。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Declares TableGen def record `SVDUPQ_8`.
  **L519 CN**: 声明 TableGen def 记录 `SVDUPQ_8`。
- **L520 EN**: Declares TableGen def record `SVDUPQ_16`.
  **L520 CN**: 声明 TableGen def 记录 `SVDUPQ_16`。
- **L521 EN**: Declares TableGen def record `SVDUPQ_32`.
  **L521 CN**: 声明 TableGen def 记录 `SVDUPQ_32`。
- **L522 EN**: Declares TableGen def record `SVDUPQ_64`.
  **L522 CN**: 声明 TableGen def 记录 `SVDUPQ_64`。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L524 EN**: Declares TableGen multiclass record `svdup_base`.
  **L524 CN**: 声明 TableGen multiclass 记录 `svdup_base`。
- **L525 EN**: Declares TableGen def record `NAME`.
  **L525 CN**: 声明 TableGen def 记录 `NAME`。
- **L526 EN**: Closes the current lexical scope or compound statement.
  **L526 CN**: 结束当前词法作用域或复合语句块。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Declares TableGen defm record `SVDUP`.
  **L528 CN**: 声明 TableGen defm 记录 `SVDUP`。
- **L529 EN**: Declares TableGen defm record `SVDUP_M`.
  **L529 CN**: 声明 TableGen defm 记录 `SVDUP_M`。
- **L530 EN**: Declares TableGen defm record `SVDUP_X`.
  **L530 CN**: 声明 TableGen defm 记录 `SVDUP_X`。
- **L531 EN**: Declares TableGen defm record `SVDUP_Z`.
  **L531 CN**: 声明 TableGen defm 记录 `SVDUP_Z`。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 533-560

````tablegen
def SVINDEX : SInst<"svindex_{d}",   "dss",  "csilUcUsUiUl",    MergeNone,    "aarch64_sve_index", [VerifyRuntimeMode]>;

// Integer arithmetic

multiclass SInstZPZ<string name, string types, string intrinsic> {
  def _M : SInst<name # "[_{d}]", "ddPd", types, MergeOp1,     intrinsic, [VerifyRuntimeMode]>;
  def _X : SInst<name # "[_{d}]", "dPd",  types, MergeAnyExp,  intrinsic, [VerifyRuntimeMode]>;
  def _Z : SInst<name # "[_{d}]", "dPd",  types, MergeZeroExp, intrinsic, [VerifyRuntimeMode]>;
}

defm SVABS : SInstZPZ<"svabs", "csil", "aarch64_sve_abs">;
defm SVNEG : SInstZPZ<"svneg", "csil", "aarch64_sve_neg">;

//------------------------------------------------------------------------------

multiclass SInstZPZZ<string name, string types, string m_intrinsic, string x_intrinsic, list<FlagType> flags=[]> {
  def _M : SInst<name # "[_{d}]",   "dPdd", types, MergeOp1,  m_intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
  def _X : SInst<name # "[_{d}]",   "dPdd", types, MergeAny,  x_intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
  def _Z : SInst<name # "[_{d}]",   "dPdd", types, MergeZero, m_intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;

  def _N_M : SInst<name # "[_n_{d}]", "dPda", types, MergeOp1,  m_intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
  def _N_X : SInst<name # "[_n_{d}]", "dPda", types, MergeAny,  x_intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
  def _N_Z : SInst<name # "[_n_{d}]", "dPda", types, MergeZero, m_intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
}

defm SVABD_S  : SInstZPZZ<"svabd",  "csil",         "aarch64_sve_sabd",  "aarch64_sve_sabd_u">;
defm SVABD_U  : SInstZPZZ<"svabd",  "UcUsUiUl",     "aarch64_sve_uabd",  "aarch64_sve_uabd_u">;
defm SVADD    : SInstZPZZ<"svadd",  "csilUcUsUiUl", "aarch64_sve_add",   "aarch64_sve_add_u">;
````
- **L533 EN**: Declares TableGen def record `SVINDEX`.
  **L533 CN**: 声明 TableGen def 记录 `SVINDEX`。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L535 EN**: Comment explains nearby logic, constraints, or intent: `Integer arithmetic`.
  **L535 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Integer arithmetic`。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Declares TableGen multiclass record `SInstZPZ`.
  **L537 CN**: 声明 TableGen multiclass 记录 `SInstZPZ`。
- **L538 EN**: Declares TableGen def record `_M`.
  **L538 CN**: 声明 TableGen def 记录 `_M`。
- **L539 EN**: Declares TableGen def record `_X`.
  **L539 CN**: 声明 TableGen def 记录 `_X`。
- **L540 EN**: Declares TableGen def record `_Z`.
  **L540 CN**: 声明 TableGen def 记录 `_Z`。
- **L541 EN**: Closes the current lexical scope or compound statement.
  **L541 CN**: 结束当前词法作用域或复合语句块。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L543 EN**: Declares TableGen defm record `SVABS`.
  **L543 CN**: 声明 TableGen defm 记录 `SVABS`。
- **L544 EN**: Declares TableGen defm record `SVNEG`.
  **L544 CN**: 声明 TableGen defm 记录 `SVNEG`。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Separator comment used for visual grouping.
  **L546 CN**: 用于视觉分组的分隔注释。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L548 EN**: Declares TableGen multiclass record `SInstZPZZ`.
  **L548 CN**: 声明 TableGen multiclass 记录 `SInstZPZZ`。
- **L549 EN**: Declares TableGen def record `_M`.
  **L549 CN**: 声明 TableGen def 记录 `_M`。
- **L550 EN**: Declares TableGen def record `_X`.
  **L550 CN**: 声明 TableGen def 记录 `_X`。
- **L551 EN**: Declares TableGen def record `_Z`.
  **L551 CN**: 声明 TableGen def 记录 `_Z`。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L553 EN**: Declares TableGen def record `_N_M`.
  **L553 CN**: 声明 TableGen def 记录 `_N_M`。
- **L554 EN**: Declares TableGen def record `_N_X`.
  **L554 CN**: 声明 TableGen def 记录 `_N_X`。
- **L555 EN**: Declares TableGen def record `_N_Z`.
  **L555 CN**: 声明 TableGen def 记录 `_N_Z`。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L558 EN**: Declares TableGen defm record `SVABD_S`.
  **L558 CN**: 声明 TableGen defm 记录 `SVABD_S`。
- **L559 EN**: Declares TableGen defm record `SVABD_U`.
  **L559 CN**: 声明 TableGen defm 记录 `SVABD_U`。
- **L560 EN**: Declares TableGen defm record `SVADD`.
  **L560 CN**: 声明 TableGen defm 记录 `SVADD`。

### Lines 561-588

````tablegen
defm SVDIV_S  : SInstZPZZ<"svdiv",  "il",           "aarch64_sve_sdiv",  "aarch64_sve_sdiv_u">;
defm SVDIV_U  : SInstZPZZ<"svdiv",  "UiUl",         "aarch64_sve_udiv",  "aarch64_sve_udiv_u">;
defm SVDIVR_S : SInstZPZZ<"svdivr", "il",           "aarch64_sve_sdivr", "aarch64_sve_sdiv_u", [ReverseMergeAnyBinOp]>;
defm SVDIVR_U : SInstZPZZ<"svdivr", "UiUl",         "aarch64_sve_udivr", "aarch64_sve_udiv_u", [ReverseMergeAnyBinOp]>;
defm SVMAX_S  : SInstZPZZ<"svmax",  "csil",         "aarch64_sve_smax",  "aarch64_sve_smax_u">;
defm SVMAX_U  : SInstZPZZ<"svmax",  "UcUsUiUl",     "aarch64_sve_umax",  "aarch64_sve_umax_u">;
defm SVMIN_S  : SInstZPZZ<"svmin",  "csil",         "aarch64_sve_smin",  "aarch64_sve_smin_u">;
defm SVMIN_U  : SInstZPZZ<"svmin",  "UcUsUiUl",     "aarch64_sve_umin",  "aarch64_sve_umin_u">;
defm SVMUL    : SInstZPZZ<"svmul",  "csilUcUsUiUl", "aarch64_sve_mul",   "aarch64_sve_mul_u">;
defm SVMULH_S : SInstZPZZ<"svmulh", "csil",         "aarch64_sve_smulh", "aarch64_sve_smulh_u">;
defm SVMULH_U : SInstZPZZ<"svmulh", "UcUsUiUl",     "aarch64_sve_umulh", "aarch64_sve_umulh_u">;
defm SVSUB    : SInstZPZZ<"svsub",  "csilUcUsUiUl", "aarch64_sve_sub",   "aarch64_sve_sub_u">;
defm SVSUBR   : SInstZPZZ<"svsubr", "csilUcUsUiUl", "aarch64_sve_subr",  "aarch64_sve_sub_u", [ReverseMergeAnyBinOp]>;

//------------------------------------------------------------------------------

multiclass SInstZPZZZ<string name, string types, string m_intrinsic, string x_intrinsic, list<FlagType> flags=[]> {
  def _M : SInst<name # "[_{d}]",   "dPddd", types, MergeOp1,  m_intrinsic, flags>;
  def _X : SInst<name # "[_{d}]",   "dPddd", types, MergeAny,  x_intrinsic, flags>;
  def _Z : SInst<name # "[_{d}]",   "dPddd", types, MergeZero, m_intrinsic, flags>;

  def _N_M : SInst<name # "[_n_{d}]", "dPdda", types, MergeOp1,  m_intrinsic, flags>;
  def _N_X : SInst<name # "[_n_{d}]", "dPdda", types, MergeAny,  x_intrinsic, flags>;
  def _N_Z : SInst<name # "[_n_{d}]", "dPdda", types, MergeZero, m_intrinsic, flags>;
}

defm SVMAD : SInstZPZZZ<"svmad", "csilUcUsUiUl", "aarch64_sve_mad", "aarch64_sve_mla_u", [ReverseMergeAnyAccOp, VerifyRuntimeMode]>;
defm SVMLA : SInstZPZZZ<"svmla", "csilUcUsUiUl", "aarch64_sve_mla", "aarch64_sve_mla_u", [VerifyRuntimeMode]>;
````
- **L561 EN**: Declares TableGen defm record `SVDIV_S`.
  **L561 CN**: 声明 TableGen defm 记录 `SVDIV_S`。
- **L562 EN**: Declares TableGen defm record `SVDIV_U`.
  **L562 CN**: 声明 TableGen defm 记录 `SVDIV_U`。
- **L563 EN**: Declares TableGen defm record `SVDIVR_S`.
  **L563 CN**: 声明 TableGen defm 记录 `SVDIVR_S`。
- **L564 EN**: Declares TableGen defm record `SVDIVR_U`.
  **L564 CN**: 声明 TableGen defm 记录 `SVDIVR_U`。
- **L565 EN**: Declares TableGen defm record `SVMAX_S`.
  **L565 CN**: 声明 TableGen defm 记录 `SVMAX_S`。
- **L566 EN**: Declares TableGen defm record `SVMAX_U`.
  **L566 CN**: 声明 TableGen defm 记录 `SVMAX_U`。
- **L567 EN**: Declares TableGen defm record `SVMIN_S`.
  **L567 CN**: 声明 TableGen defm 记录 `SVMIN_S`。
- **L568 EN**: Declares TableGen defm record `SVMIN_U`.
  **L568 CN**: 声明 TableGen defm 记录 `SVMIN_U`。
- **L569 EN**: Declares TableGen defm record `SVMUL`.
  **L569 CN**: 声明 TableGen defm 记录 `SVMUL`。
- **L570 EN**: Declares TableGen defm record `SVMULH_S`.
  **L570 CN**: 声明 TableGen defm 记录 `SVMULH_S`。
- **L571 EN**: Declares TableGen defm record `SVMULH_U`.
  **L571 CN**: 声明 TableGen defm 记录 `SVMULH_U`。
- **L572 EN**: Declares TableGen defm record `SVSUB`.
  **L572 CN**: 声明 TableGen defm 记录 `SVSUB`。
- **L573 EN**: Declares TableGen defm record `SVSUBR`.
  **L573 CN**: 声明 TableGen defm 记录 `SVSUBR`。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Separator comment used for visual grouping.
  **L575 CN**: 用于视觉分组的分隔注释。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L577 EN**: Declares TableGen multiclass record `SInstZPZZZ`.
  **L577 CN**: 声明 TableGen multiclass 记录 `SInstZPZZZ`。
- **L578 EN**: Declares TableGen def record `_M`.
  **L578 CN**: 声明 TableGen def 记录 `_M`。
- **L579 EN**: Declares TableGen def record `_X`.
  **L579 CN**: 声明 TableGen def 记录 `_X`。
- **L580 EN**: Declares TableGen def record `_Z`.
  **L580 CN**: 声明 TableGen def 记录 `_Z`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Declares TableGen def record `_N_M`.
  **L582 CN**: 声明 TableGen def 记录 `_N_M`。
- **L583 EN**: Declares TableGen def record `_N_X`.
  **L583 CN**: 声明 TableGen def 记录 `_N_X`。
- **L584 EN**: Declares TableGen def record `_N_Z`.
  **L584 CN**: 声明 TableGen def 记录 `_N_Z`。
- **L585 EN**: Closes the current lexical scope or compound statement.
  **L585 CN**: 结束当前词法作用域或复合语句块。
- **L586 EN**: Blank line separating nearby declarations or logic blocks.
  **L586 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L587 EN**: Declares TableGen defm record `SVMAD`.
  **L587 CN**: 声明 TableGen defm 记录 `SVMAD`。
- **L588 EN**: Declares TableGen defm record `SVMLA`.
  **L588 CN**: 声明 TableGen defm 记录 `SVMLA`。

### Lines 589-616

````tablegen
defm SVMLS : SInstZPZZZ<"svmls", "csilUcUsUiUl", "aarch64_sve_mls", "aarch64_sve_mls_u", [VerifyRuntimeMode]>;
defm SVMSB : SInstZPZZZ<"svmsb", "csilUcUsUiUl", "aarch64_sve_msb", "aarch64_sve_mls_u", [ReverseMergeAnyAccOp, VerifyRuntimeMode]>;

//------------------------------------------------------------------------------

def SVDOT_S  : SInst<"svdot[_{0}]",    "ddqq", "il",       MergeNone, "aarch64_sve_sdot", [VerifyRuntimeMode]>;
def SVDOT_U  : SInst<"svdot[_{0}]",    "ddqq", "UiUl",     MergeNone, "aarch64_sve_udot", [VerifyRuntimeMode]>;
def SVQADD_S : SInst<"svqadd[_{d}]",   "ddd",  "csil",     MergeNone, "aarch64_sve_sqadd_x", [VerifyRuntimeMode]>;
def SVQADD_U : SInst<"svqadd[_{d}]",   "ddd",  "UcUsUiUl", MergeNone, "aarch64_sve_uqadd_x", [VerifyRuntimeMode]>;
def SVQSUB_S : SInst<"svqsub[_{d}]",   "ddd",  "csil",     MergeNone, "aarch64_sve_sqsub_x", [VerifyRuntimeMode]>;
def SVQSUB_U : SInst<"svqsub[_{d}]",   "ddd",  "UcUsUiUl", MergeNone, "aarch64_sve_uqsub_x", [VerifyRuntimeMode]>;

def SVDOT_N_S  : SInst<"svdot[_n_{0}]",  "ddqr", "il",       MergeNone, "aarch64_sve_sdot", [VerifyRuntimeMode]>;
def SVDOT_N_U  : SInst<"svdot[_n_{0}]",  "ddqr", "UiUl",     MergeNone, "aarch64_sve_udot", [VerifyRuntimeMode]>;
def SVQADD_N_S : SInst<"svqadd[_n_{d}]", "dda",  "csil",     MergeNone, "aarch64_sve_sqadd_x", [VerifyRuntimeMode]>;
def SVQADD_N_U : SInst<"svqadd[_n_{d}]", "dda",  "UcUsUiUl", MergeNone, "aarch64_sve_uqadd_x", [VerifyRuntimeMode]>;
def SVQSUB_N_S : SInst<"svqsub[_n_{d}]", "dda",  "csil",     MergeNone, "aarch64_sve_sqsub_x", [VerifyRuntimeMode]>;
def SVQSUB_N_U : SInst<"svqsub[_n_{d}]", "dda",  "UcUsUiUl", MergeNone, "aarch64_sve_uqsub_x", [VerifyRuntimeMode]>;

def SVDOT_LANE_S : SInst<"svdot_lane[_{d}]",  "ddqqi",  "il",   MergeNone, "aarch64_sve_sdot_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndexDot, 2>]>;
def SVDOT_LANE_U : SInst<"svdot_lane[_{d}]",  "ddqqi",  "UiUl", MergeNone, "aarch64_sve_udot_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndexDot, 2>]>;

////////////////////////////////////////////////////////////////////////////////
// Logical operations

defm SVAND : SInstZPZZ<"svand", "csilUcUsUiUl", "aarch64_sve_and", "aarch64_sve_and_u">;
defm SVBIC : SInstZPZZ<"svbic", "csilUcUsUiUl", "aarch64_sve_bic", "aarch64_sve_bic_u">;
defm SVEOR : SInstZPZZ<"sveor", "csilUcUsUiUl", "aarch64_sve_eor", "aarch64_sve_eor_u">;
````
- **L589 EN**: Declares TableGen defm record `SVMLS`.
  **L589 CN**: 声明 TableGen defm 记录 `SVMLS`。
- **L590 EN**: Declares TableGen defm record `SVMSB`.
  **L590 CN**: 声明 TableGen defm 记录 `SVMSB`。
- **L591 EN**: Blank line separating nearby declarations or logic blocks.
  **L591 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L592 EN**: Separator comment used for visual grouping.
  **L592 CN**: 用于视觉分组的分隔注释。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Declares TableGen def record `SVDOT_S`.
  **L594 CN**: 声明 TableGen def 记录 `SVDOT_S`。
- **L595 EN**: Declares TableGen def record `SVDOT_U`.
  **L595 CN**: 声明 TableGen def 记录 `SVDOT_U`。
- **L596 EN**: Declares TableGen def record `SVQADD_S`.
  **L596 CN**: 声明 TableGen def 记录 `SVQADD_S`。
- **L597 EN**: Declares TableGen def record `SVQADD_U`.
  **L597 CN**: 声明 TableGen def 记录 `SVQADD_U`。
- **L598 EN**: Declares TableGen def record `SVQSUB_S`.
  **L598 CN**: 声明 TableGen def 记录 `SVQSUB_S`。
- **L599 EN**: Declares TableGen def record `SVQSUB_U`.
  **L599 CN**: 声明 TableGen def 记录 `SVQSUB_U`。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L601 EN**: Declares TableGen def record `SVDOT_N_S`.
  **L601 CN**: 声明 TableGen def 记录 `SVDOT_N_S`。
- **L602 EN**: Declares TableGen def record `SVDOT_N_U`.
  **L602 CN**: 声明 TableGen def 记录 `SVDOT_N_U`。
- **L603 EN**: Declares TableGen def record `SVQADD_N_S`.
  **L603 CN**: 声明 TableGen def 记录 `SVQADD_N_S`。
- **L604 EN**: Declares TableGen def record `SVQADD_N_U`.
  **L604 CN**: 声明 TableGen def 记录 `SVQADD_N_U`。
- **L605 EN**: Declares TableGen def record `SVQSUB_N_S`.
  **L605 CN**: 声明 TableGen def 记录 `SVQSUB_N_S`。
- **L606 EN**: Declares TableGen def record `SVQSUB_N_U`.
  **L606 CN**: 声明 TableGen def 记录 `SVQSUB_N_U`。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L608 EN**: Declares TableGen def record `SVDOT_LANE_S`.
  **L608 CN**: 声明 TableGen def 记录 `SVDOT_LANE_S`。
- **L609 EN**: Declares TableGen def record `SVDOT_LANE_U`.
  **L609 CN**: 声明 TableGen def 记录 `SVDOT_LANE_U`。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L611 EN**: Separator comment used for visual grouping.
  **L611 CN**: 用于视觉分组的分隔注释。
- **L612 EN**: Comment explains nearby logic, constraints, or intent: `Logical operations`.
  **L612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Logical operations`。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Declares TableGen defm record `SVAND`.
  **L614 CN**: 声明 TableGen defm 记录 `SVAND`。
- **L615 EN**: Declares TableGen defm record `SVBIC`.
  **L615 CN**: 声明 TableGen defm 记录 `SVBIC`。
- **L616 EN**: Declares TableGen defm record `SVEOR`.
  **L616 CN**: 声明 TableGen defm 记录 `SVEOR`。

### Lines 617-644

````tablegen
defm SVORR : SInstZPZZ<"svorr", "csilUcUsUiUl", "aarch64_sve_orr", "aarch64_sve_orr_u">;

defm SVCNOT : SInstZPZ<"svcnot", "csilUcUsUiUl", "aarch64_sve_cnot">;
defm SVNOT  : SInstZPZ<"svnot",  "csilUcUsUiUl", "aarch64_sve_not">;

////////////////////////////////////////////////////////////////////////////////
// Shifts

multiclass SInst_SHIFT<string name, string intrinsic, string ts, string wide_ts> {
  def _M : SInst<name # "[_{d}]", "dPdu", ts, MergeOp1,  intrinsic, [VerifyRuntimeMode]>;
  def _X : SInst<name # "[_{d}]", "dPdu", ts, MergeAny,  intrinsic # _u, [VerifyRuntimeMode]>;
  def _Z : SInst<name # "[_{d}]", "dPdu", ts, MergeZero, intrinsic, [VerifyRuntimeMode]>;

  def _N_M : SInst<name # "[_n_{d}]", "dPdL", ts, MergeOp1,  intrinsic, [VerifyRuntimeMode]>;
  def _N_X : SInst<name # "[_n_{d}]", "dPdL", ts, MergeAny,  intrinsic # _u, [VerifyRuntimeMode]>;
  def _N_Z : SInst<name # "[_n_{d}]", "dPdL", ts, MergeZero, intrinsic, [VerifyRuntimeMode]>;

  def _WIDE_M : SInst<name # _wide # "[_{d}]", "dPdg", wide_ts, MergeOp1,  intrinsic # _wide, [VerifyRuntimeMode]>;
  def _WIDE_X : SInst<name # _wide # "[_{d}]", "dPdg", wide_ts, MergeAny,  intrinsic # _wide, [VerifyRuntimeMode]>;
  def _WIDE_Z : SInst<name # _wide # "[_{d}]", "dPdg", wide_ts, MergeZero, intrinsic # _wide, [VerifyRuntimeMode]>;

  def _WIDE_N_M : SInst<name # _wide # "[_n_{d}]", "dPdf", wide_ts, MergeOp1,  intrinsic # _wide, [VerifyRuntimeMode]>;
  def _WIDE_N_X : SInst<name # _wide # "[_n_{d}]", "dPdf", wide_ts, MergeAny,  intrinsic # _wide, [VerifyRuntimeMode]>;
  def _WIDE_N_Z : SInst<name # _wide # "[_n_{d}]", "dPdf", wide_ts, MergeZero, intrinsic # _wide, [VerifyRuntimeMode]>;
}

defm SVASR : SInst_SHIFT<"svasr", "aarch64_sve_asr", "csil", "csi">;
defm SVLSL : SInst_SHIFT<"svlsl", "aarch64_sve_lsl", "csilUcUsUiUl", "csiUcUsUi">;
````
- **L617 EN**: Declares TableGen defm record `SVORR`.
  **L617 CN**: 声明 TableGen defm 记录 `SVORR`。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L619 EN**: Declares TableGen defm record `SVCNOT`.
  **L619 CN**: 声明 TableGen defm 记录 `SVCNOT`。
- **L620 EN**: Declares TableGen defm record `SVNOT`.
  **L620 CN**: 声明 TableGen defm 记录 `SVNOT`。
- **L621 EN**: Blank line separating nearby declarations or logic blocks.
  **L621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L622 EN**: Separator comment used for visual grouping.
  **L622 CN**: 用于视觉分组的分隔注释。
- **L623 EN**: Comment explains nearby logic, constraints, or intent: `Shifts`.
  **L623 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shifts`。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L625 EN**: Declares TableGen multiclass record `SInst_SHIFT`.
  **L625 CN**: 声明 TableGen multiclass 记录 `SInst_SHIFT`。
- **L626 EN**: Declares TableGen def record `_M`.
  **L626 CN**: 声明 TableGen def 记录 `_M`。
- **L627 EN**: Declares TableGen def record `_X`.
  **L627 CN**: 声明 TableGen def 记录 `_X`。
- **L628 EN**: Declares TableGen def record `_Z`.
  **L628 CN**: 声明 TableGen def 记录 `_Z`。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L630 EN**: Declares TableGen def record `_N_M`.
  **L630 CN**: 声明 TableGen def 记录 `_N_M`。
- **L631 EN**: Declares TableGen def record `_N_X`.
  **L631 CN**: 声明 TableGen def 记录 `_N_X`。
- **L632 EN**: Declares TableGen def record `_N_Z`.
  **L632 CN**: 声明 TableGen def 记录 `_N_Z`。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L634 EN**: Declares TableGen def record `_WIDE_M`.
  **L634 CN**: 声明 TableGen def 记录 `_WIDE_M`。
- **L635 EN**: Declares TableGen def record `_WIDE_X`.
  **L635 CN**: 声明 TableGen def 记录 `_WIDE_X`。
- **L636 EN**: Declares TableGen def record `_WIDE_Z`.
  **L636 CN**: 声明 TableGen def 记录 `_WIDE_Z`。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L638 EN**: Declares TableGen def record `_WIDE_N_M`.
  **L638 CN**: 声明 TableGen def 记录 `_WIDE_N_M`。
- **L639 EN**: Declares TableGen def record `_WIDE_N_X`.
  **L639 CN**: 声明 TableGen def 记录 `_WIDE_N_X`。
- **L640 EN**: Declares TableGen def record `_WIDE_N_Z`.
  **L640 CN**: 声明 TableGen def 记录 `_WIDE_N_Z`。
- **L641 EN**: Closes the current lexical scope or compound statement.
  **L641 CN**: 结束当前词法作用域或复合语句块。
- **L642 EN**: Blank line separating nearby declarations or logic blocks.
  **L642 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L643 EN**: Declares TableGen defm record `SVASR`.
  **L643 CN**: 声明 TableGen defm 记录 `SVASR`。
- **L644 EN**: Declares TableGen defm record `SVLSL`.
  **L644 CN**: 声明 TableGen defm 记录 `SVLSL`。

### Lines 645-672

````tablegen
defm SVLSR : SInst_SHIFT<"svlsr", "aarch64_sve_lsr", "UcUsUiUl", "UcUsUi">;

def SVASRD_M : SInst<"svasrd[_n_{d}]", "dPdi", "csil", MergeOp1,  "aarch64_sve_asrd", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVASRD_X : SInst<"svasrd[_n_{d}]", "dPdi", "csil", MergeAny,  "aarch64_sve_asrd", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVASRD_Z : SInst<"svasrd[_n_{d}]", "dPdi", "csil", MergeZero, "aarch64_sve_asrd", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;

def SVINSR : SInst<"svinsr[_n_{d}]", "dds", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_insr", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Integer reductions

def SVADDV_S : SInst<"svaddv[_{d}]", "lPd", "csil",         MergeNone, "aarch64_sve_saddv", [VerifyRuntimeMode]>;
def SVADDV_U : SInst<"svaddv[_{d}]", "nPd", "UcUsUiUl",     MergeNone, "aarch64_sve_uaddv", [VerifyRuntimeMode]>;
def SVANDV   : SInst<"svandv[_{d}]", "sPd", "csilUcUsUiUl", MergeNone, "aarch64_sve_andv", [VerifyRuntimeMode]>;
def SVEORV   : SInst<"sveorv[_{d}]", "sPd", "csilUcUsUiUl", MergeNone, "aarch64_sve_eorv", [VerifyRuntimeMode]>;
def SVMAXV_S : SInst<"svmaxv[_{d}]", "sPd", "csil",         MergeNone, "aarch64_sve_smaxv", [VerifyRuntimeMode]>;
def SVMAXV_U : SInst<"svmaxv[_{d}]", "sPd", "UcUsUiUl",     MergeNone, "aarch64_sve_umaxv", [VerifyRuntimeMode]>;
def SVMINV_S : SInst<"svminv[_{d}]", "sPd", "csil",         MergeNone, "aarch64_sve_sminv", [VerifyRuntimeMode]>;
def SVMINV_U : SInst<"svminv[_{d}]", "sPd", "UcUsUiUl",     MergeNone, "aarch64_sve_uminv", [VerifyRuntimeMode]>;
def SVORV    : SInst<"svorv[_{d}]",  "sPd", "csilUcUsUiUl", MergeNone, "aarch64_sve_orv", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Integer comparisons

def SVCMPEQ : SInst<"svcmpeq[_{d}]", "PPdd", "csilUcUsUiUl", MergeNone, "aarch64_sve_cmpeq", [VerifyRuntimeMode]>;
def SVCMPNE : SInst<"svcmpne[_{d}]", "PPdd", "csilUcUsUiUl", MergeNone, "aarch64_sve_cmpne", [VerifyRuntimeMode]>;
def SVCMPGE : SInst<"svcmpge[_{d}]", "PPdd", "csil",         MergeNone, "aarch64_sve_cmpge", [VerifyRuntimeMode]>;
def SVCMPGT : SInst<"svcmpgt[_{d}]", "PPdd", "csil",         MergeNone, "aarch64_sve_cmpgt", [VerifyRuntimeMode]>;
````
- **L645 EN**: Declares TableGen defm record `SVLSR`.
  **L645 CN**: 声明 TableGen defm 记录 `SVLSR`。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L647 EN**: Declares TableGen def record `SVASRD_M`.
  **L647 CN**: 声明 TableGen def 记录 `SVASRD_M`。
- **L648 EN**: Declares TableGen def record `SVASRD_X`.
  **L648 CN**: 声明 TableGen def 记录 `SVASRD_X`。
- **L649 EN**: Declares TableGen def record `SVASRD_Z`.
  **L649 CN**: 声明 TableGen def 记录 `SVASRD_Z`。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Declares TableGen def record `SVINSR`.
  **L651 CN**: 声明 TableGen def 记录 `SVINSR`。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L653 EN**: Separator comment used for visual grouping.
  **L653 CN**: 用于视觉分组的分隔注释。
- **L654 EN**: Comment explains nearby logic, constraints, or intent: `Integer reductions`.
  **L654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Integer reductions`。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L656 EN**: Declares TableGen def record `SVADDV_S`.
  **L656 CN**: 声明 TableGen def 记录 `SVADDV_S`。
- **L657 EN**: Declares TableGen def record `SVADDV_U`.
  **L657 CN**: 声明 TableGen def 记录 `SVADDV_U`。
- **L658 EN**: Declares TableGen def record `SVANDV`.
  **L658 CN**: 声明 TableGen def 记录 `SVANDV`。
- **L659 EN**: Declares TableGen def record `SVEORV`.
  **L659 CN**: 声明 TableGen def 记录 `SVEORV`。
- **L660 EN**: Declares TableGen def record `SVMAXV_S`.
  **L660 CN**: 声明 TableGen def 记录 `SVMAXV_S`。
- **L661 EN**: Declares TableGen def record `SVMAXV_U`.
  **L661 CN**: 声明 TableGen def 记录 `SVMAXV_U`。
- **L662 EN**: Declares TableGen def record `SVMINV_S`.
  **L662 CN**: 声明 TableGen def 记录 `SVMINV_S`。
- **L663 EN**: Declares TableGen def record `SVMINV_U`.
  **L663 CN**: 声明 TableGen def 记录 `SVMINV_U`。
- **L664 EN**: Declares TableGen def record `SVORV`.
  **L664 CN**: 声明 TableGen def 记录 `SVORV`。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L666 EN**: Separator comment used for visual grouping.
  **L666 CN**: 用于视觉分组的分隔注释。
- **L667 EN**: Comment explains nearby logic, constraints, or intent: `Integer comparisons`.
  **L667 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Integer comparisons`。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Declares TableGen def record `SVCMPEQ`.
  **L669 CN**: 声明 TableGen def 记录 `SVCMPEQ`。
- **L670 EN**: Declares TableGen def record `SVCMPNE`.
  **L670 CN**: 声明 TableGen def 记录 `SVCMPNE`。
- **L671 EN**: Declares TableGen def record `SVCMPGE`.
  **L671 CN**: 声明 TableGen def 记录 `SVCMPGE`。
- **L672 EN**: Declares TableGen def record `SVCMPGT`.
  **L672 CN**: 声明 TableGen def 记录 `SVCMPGT`。

### Lines 673-700

````tablegen
def SVCMPLE : SInst<"svcmple[_{d}]", "PPdd", "csil",         MergeNone, "aarch64_sve_cmpge", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPLT : SInst<"svcmplt[_{d}]", "PPdd", "csil",         MergeNone, "aarch64_sve_cmpgt", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPHI : SInst<"svcmpgt[_{d}]", "PPdd", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphi", [VerifyRuntimeMode]>;
def SVCMPHS : SInst<"svcmpge[_{d}]", "PPdd", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphs", [VerifyRuntimeMode]>;
def SVCMPLO : SInst<"svcmplt[_{d}]", "PPdd", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphi", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPLS : SInst<"svcmple[_{d}]", "PPdd", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphs", [ReverseCompare, VerifyRuntimeMode]>;

def SVCMPEQ_N : SInst<"svcmpeq[_n_{d}]", "PPda", "csilUcUsUiUl", MergeNone, "aarch64_sve_cmpeq", [VerifyRuntimeMode]>;
def SVCMPNE_N : SInst<"svcmpne[_n_{d}]", "PPda", "csilUcUsUiUl", MergeNone, "aarch64_sve_cmpne", [VerifyRuntimeMode]>;
def SVCMPGE_N : SInst<"svcmpge[_n_{d}]", "PPda", "csil",         MergeNone, "aarch64_sve_cmpge", [VerifyRuntimeMode]>;
def SVCMPGT_N : SInst<"svcmpgt[_n_{d}]", "PPda", "csil",         MergeNone, "aarch64_sve_cmpgt", [VerifyRuntimeMode]>;
def SVCMPLE_N : SInst<"svcmple[_n_{d}]", "PPda", "csil",         MergeNone, "aarch64_sve_cmpge", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPLT_N : SInst<"svcmplt[_n_{d}]", "PPda", "csil",         MergeNone, "aarch64_sve_cmpgt", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPHS_N : SInst<"svcmpge[_n_{d}]", "PPda", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphs", [VerifyRuntimeMode]>;
def SVCMPHI_N : SInst<"svcmpgt[_n_{d}]", "PPda", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphi", [VerifyRuntimeMode]>;
def SVCMPLS_N : SInst<"svcmple[_n_{d}]", "PPda", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphs", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPLO_N : SInst<"svcmplt[_n_{d}]", "PPda", "UcUsUiUl",     MergeNone, "aarch64_sve_cmphi", [ReverseCompare, VerifyRuntimeMode]>;

def SVCMPEQ_WIDE : SInst<"svcmpeq_wide[_{d}]", "PPdw", "csi",    MergeNone, "aarch64_sve_cmpeq_wide", [VerifyRuntimeMode]>;
def SVCMPNE_WIDE : SInst<"svcmpne_wide[_{d}]", "PPdw", "csi",    MergeNone, "aarch64_sve_cmpne_wide", [VerifyRuntimeMode]>;
def SVCMPGE_WIDE : SInst<"svcmpge_wide[_{d}]", "PPdw", "csi",    MergeNone, "aarch64_sve_cmpge_wide", [VerifyRuntimeMode]>;
def SVCMPGT_WIDE : SInst<"svcmpgt_wide[_{d}]", "PPdw", "csi",    MergeNone, "aarch64_sve_cmpgt_wide", [VerifyRuntimeMode]>;
def SVCMPLE_WIDE : SInst<"svcmple_wide[_{d}]", "PPdw", "csi",    MergeNone, "aarch64_sve_cmple_wide", [VerifyRuntimeMode]>;
def SVCMPLT_WIDE : SInst<"svcmplt_wide[_{d}]", "PPdw", "csi",    MergeNone, "aarch64_sve_cmplt_wide", [VerifyRuntimeMode]>;
def SVCMPHI_WIDE : SInst<"svcmpgt_wide[_{d}]", "PPdw", "UcUsUi", MergeNone, "aarch64_sve_cmphi_wide", [VerifyRuntimeMode]>;
def SVCMPHS_WIDE : SInst<"svcmpge_wide[_{d}]", "PPdw", "UcUsUi", MergeNone, "aarch64_sve_cmphs_wide", [VerifyRuntimeMode]>;
def SVCMPLO_WIDE : SInst<"svcmplt_wide[_{d}]", "PPdw", "UcUsUi", MergeNone, "aarch64_sve_cmplo_wide", [VerifyRuntimeMode]>;
def SVCMPLS_WIDE : SInst<"svcmple_wide[_{d}]", "PPdw", "UcUsUi", MergeNone, "aarch64_sve_cmpls_wide", [VerifyRuntimeMode]>;
````
- **L673 EN**: Declares TableGen def record `SVCMPLE`.
  **L673 CN**: 声明 TableGen def 记录 `SVCMPLE`。
- **L674 EN**: Declares TableGen def record `SVCMPLT`.
  **L674 CN**: 声明 TableGen def 记录 `SVCMPLT`。
- **L675 EN**: Declares TableGen def record `SVCMPHI`.
  **L675 CN**: 声明 TableGen def 记录 `SVCMPHI`。
- **L676 EN**: Declares TableGen def record `SVCMPHS`.
  **L676 CN**: 声明 TableGen def 记录 `SVCMPHS`。
- **L677 EN**: Declares TableGen def record `SVCMPLO`.
  **L677 CN**: 声明 TableGen def 记录 `SVCMPLO`。
- **L678 EN**: Declares TableGen def record `SVCMPLS`.
  **L678 CN**: 声明 TableGen def 记录 `SVCMPLS`。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L680 EN**: Declares TableGen def record `SVCMPEQ_N`.
  **L680 CN**: 声明 TableGen def 记录 `SVCMPEQ_N`。
- **L681 EN**: Declares TableGen def record `SVCMPNE_N`.
  **L681 CN**: 声明 TableGen def 记录 `SVCMPNE_N`。
- **L682 EN**: Declares TableGen def record `SVCMPGE_N`.
  **L682 CN**: 声明 TableGen def 记录 `SVCMPGE_N`。
- **L683 EN**: Declares TableGen def record `SVCMPGT_N`.
  **L683 CN**: 声明 TableGen def 记录 `SVCMPGT_N`。
- **L684 EN**: Declares TableGen def record `SVCMPLE_N`.
  **L684 CN**: 声明 TableGen def 记录 `SVCMPLE_N`。
- **L685 EN**: Declares TableGen def record `SVCMPLT_N`.
  **L685 CN**: 声明 TableGen def 记录 `SVCMPLT_N`。
- **L686 EN**: Declares TableGen def record `SVCMPHS_N`.
  **L686 CN**: 声明 TableGen def 记录 `SVCMPHS_N`。
- **L687 EN**: Declares TableGen def record `SVCMPHI_N`.
  **L687 CN**: 声明 TableGen def 记录 `SVCMPHI_N`。
- **L688 EN**: Declares TableGen def record `SVCMPLS_N`.
  **L688 CN**: 声明 TableGen def 记录 `SVCMPLS_N`。
- **L689 EN**: Declares TableGen def record `SVCMPLO_N`.
  **L689 CN**: 声明 TableGen def 记录 `SVCMPLO_N`。
- **L690 EN**: Blank line separating nearby declarations or logic blocks.
  **L690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L691 EN**: Declares TableGen def record `SVCMPEQ_WIDE`.
  **L691 CN**: 声明 TableGen def 记录 `SVCMPEQ_WIDE`。
- **L692 EN**: Declares TableGen def record `SVCMPNE_WIDE`.
  **L692 CN**: 声明 TableGen def 记录 `SVCMPNE_WIDE`。
- **L693 EN**: Declares TableGen def record `SVCMPGE_WIDE`.
  **L693 CN**: 声明 TableGen def 记录 `SVCMPGE_WIDE`。
- **L694 EN**: Declares TableGen def record `SVCMPGT_WIDE`.
  **L694 CN**: 声明 TableGen def 记录 `SVCMPGT_WIDE`。
- **L695 EN**: Declares TableGen def record `SVCMPLE_WIDE`.
  **L695 CN**: 声明 TableGen def 记录 `SVCMPLE_WIDE`。
- **L696 EN**: Declares TableGen def record `SVCMPLT_WIDE`.
  **L696 CN**: 声明 TableGen def 记录 `SVCMPLT_WIDE`。
- **L697 EN**: Declares TableGen def record `SVCMPHI_WIDE`.
  **L697 CN**: 声明 TableGen def 记录 `SVCMPHI_WIDE`。
- **L698 EN**: Declares TableGen def record `SVCMPHS_WIDE`.
  **L698 CN**: 声明 TableGen def 记录 `SVCMPHS_WIDE`。
- **L699 EN**: Declares TableGen def record `SVCMPLO_WIDE`.
  **L699 CN**: 声明 TableGen def 记录 `SVCMPLO_WIDE`。
- **L700 EN**: Declares TableGen def record `SVCMPLS_WIDE`.
  **L700 CN**: 声明 TableGen def 记录 `SVCMPLS_WIDE`。

### Lines 701-728

````tablegen

def SVCMPEQ_WIDE_N : SInst<"svcmpeq_wide[_n_{d}]", "PPdj", "csi",    MergeNone, "aarch64_sve_cmpeq_wide", [VerifyRuntimeMode]>;
def SVCMPNE_WIDE_N : SInst<"svcmpne_wide[_n_{d}]", "PPdj", "csi",    MergeNone, "aarch64_sve_cmpne_wide", [VerifyRuntimeMode]>;
def SVCMPGE_WIDE_N : SInst<"svcmpge_wide[_n_{d}]", "PPdj", "csi",    MergeNone, "aarch64_sve_cmpge_wide", [VerifyRuntimeMode]>;
def SVCMPGT_WIDE_N : SInst<"svcmpgt_wide[_n_{d}]", "PPdj", "csi",    MergeNone, "aarch64_sve_cmpgt_wide", [VerifyRuntimeMode]>;
def SVCMPLE_WIDE_N : SInst<"svcmple_wide[_n_{d}]", "PPdj", "csi",    MergeNone, "aarch64_sve_cmple_wide", [VerifyRuntimeMode]>;
def SVCMPLT_WIDE_N : SInst<"svcmplt_wide[_n_{d}]", "PPdj", "csi",    MergeNone, "aarch64_sve_cmplt_wide", [VerifyRuntimeMode]>;
def SVCMPHS_WIDE_N : SInst<"svcmpge_wide[_n_{d}]", "PPdj", "UcUsUi", MergeNone, "aarch64_sve_cmphs_wide", [VerifyRuntimeMode]>;
def SVCMPHI_WIDE_N : SInst<"svcmpgt_wide[_n_{d}]", "PPdj", "UcUsUi", MergeNone, "aarch64_sve_cmphi_wide", [VerifyRuntimeMode]>;
def SVCMPLO_WIDE_N : SInst<"svcmplt_wide[_n_{d}]", "PPdj", "UcUsUi", MergeNone, "aarch64_sve_cmplo_wide", [VerifyRuntimeMode]>;
def SVCMPLS_WIDE_N : SInst<"svcmple_wide[_n_{d}]", "PPdj", "UcUsUi", MergeNone, "aarch64_sve_cmpls_wide", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// While comparisons

def SVWHILELE_S32 : SInst<"svwhilele_{d}[_{1}]", "Pkk", "PcPsPiPl", MergeNone, "aarch64_sve_whilele", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILELE_S64 : SInst<"svwhilele_{d}[_{1}]", "Pll", "PcPsPiPl", MergeNone, "aarch64_sve_whilele", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILELO_U32 : SInst<"svwhilelt_{d}[_{1}]", "Pmm", "PcPsPiPl", MergeNone, "aarch64_sve_whilelo", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILELO_U64 : SInst<"svwhilelt_{d}[_{1}]", "Pnn", "PcPsPiPl", MergeNone, "aarch64_sve_whilelo", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILELS_U32 : SInst<"svwhilele_{d}[_{1}]", "Pmm", "PcPsPiPl", MergeNone, "aarch64_sve_whilels", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILELS_U64 : SInst<"svwhilele_{d}[_{1}]", "Pnn", "PcPsPiPl", MergeNone, "aarch64_sve_whilels", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILELT_S32 : SInst<"svwhilelt_{d}[_{1}]", "Pkk", "PcPsPiPl", MergeNone, "aarch64_sve_whilelt", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILELT_S64 : SInst<"svwhilelt_{d}[_{1}]", "Pll", "PcPsPiPl", MergeNone, "aarch64_sve_whilelt", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Counting bit

multiclass SInstCLS<string name, string types, string intrinsic, list<FlagType> flags=[]> {
````
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Declares TableGen def record `SVCMPEQ_WIDE_N`.
  **L702 CN**: 声明 TableGen def 记录 `SVCMPEQ_WIDE_N`。
- **L703 EN**: Declares TableGen def record `SVCMPNE_WIDE_N`.
  **L703 CN**: 声明 TableGen def 记录 `SVCMPNE_WIDE_N`。
- **L704 EN**: Declares TableGen def record `SVCMPGE_WIDE_N`.
  **L704 CN**: 声明 TableGen def 记录 `SVCMPGE_WIDE_N`。
- **L705 EN**: Declares TableGen def record `SVCMPGT_WIDE_N`.
  **L705 CN**: 声明 TableGen def 记录 `SVCMPGT_WIDE_N`。
- **L706 EN**: Declares TableGen def record `SVCMPLE_WIDE_N`.
  **L706 CN**: 声明 TableGen def 记录 `SVCMPLE_WIDE_N`。
- **L707 EN**: Declares TableGen def record `SVCMPLT_WIDE_N`.
  **L707 CN**: 声明 TableGen def 记录 `SVCMPLT_WIDE_N`。
- **L708 EN**: Declares TableGen def record `SVCMPHS_WIDE_N`.
  **L708 CN**: 声明 TableGen def 记录 `SVCMPHS_WIDE_N`。
- **L709 EN**: Declares TableGen def record `SVCMPHI_WIDE_N`.
  **L709 CN**: 声明 TableGen def 记录 `SVCMPHI_WIDE_N`。
- **L710 EN**: Declares TableGen def record `SVCMPLO_WIDE_N`.
  **L710 CN**: 声明 TableGen def 记录 `SVCMPLO_WIDE_N`。
- **L711 EN**: Declares TableGen def record `SVCMPLS_WIDE_N`.
  **L711 CN**: 声明 TableGen def 记录 `SVCMPLS_WIDE_N`。
- **L712 EN**: Blank line separating nearby declarations or logic blocks.
  **L712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L713 EN**: Separator comment used for visual grouping.
  **L713 CN**: 用于视觉分组的分隔注释。
- **L714 EN**: Comment explains nearby logic, constraints, or intent: `While comparisons`.
  **L714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`While comparisons`。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L716 EN**: Declares TableGen def record `SVWHILELE_S32`.
  **L716 CN**: 声明 TableGen def 记录 `SVWHILELE_S32`。
- **L717 EN**: Declares TableGen def record `SVWHILELE_S64`.
  **L717 CN**: 声明 TableGen def 记录 `SVWHILELE_S64`。
- **L718 EN**: Declares TableGen def record `SVWHILELO_U32`.
  **L718 CN**: 声明 TableGen def 记录 `SVWHILELO_U32`。
- **L719 EN**: Declares TableGen def record `SVWHILELO_U64`.
  **L719 CN**: 声明 TableGen def 记录 `SVWHILELO_U64`。
- **L720 EN**: Declares TableGen def record `SVWHILELS_U32`.
  **L720 CN**: 声明 TableGen def 记录 `SVWHILELS_U32`。
- **L721 EN**: Declares TableGen def record `SVWHILELS_U64`.
  **L721 CN**: 声明 TableGen def 记录 `SVWHILELS_U64`。
- **L722 EN**: Declares TableGen def record `SVWHILELT_S32`.
  **L722 CN**: 声明 TableGen def 记录 `SVWHILELT_S32`。
- **L723 EN**: Declares TableGen def record `SVWHILELT_S64`.
  **L723 CN**: 声明 TableGen def 记录 `SVWHILELT_S64`。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L725 EN**: Separator comment used for visual grouping.
  **L725 CN**: 用于视觉分组的分隔注释。
- **L726 EN**: Comment explains nearby logic, constraints, or intent: `Counting bit`.
  **L726 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Counting bit`。
- **L727 EN**: Blank line separating nearby declarations or logic blocks.
  **L727 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L728 EN**: Declares TableGen multiclass record `SInstCLS`.
  **L728 CN**: 声明 TableGen multiclass 记录 `SInstCLS`。

### Lines 729-756

````tablegen
  def _M : SInst<name # "[_{d}]", "uuPd", types, MergeOp1,     intrinsic, flags>;
  def _X : SInst<name # "[_{d}]", "uPd",  types, MergeAnyExp,  intrinsic, flags>;
  def _Z : SInst<name # "[_{d}]", "uPd",  types, MergeZeroExp, intrinsic, flags>;
}

defm SVCLS : SInstCLS<"svcls", "csil",             "aarch64_sve_cls", [VerifyRuntimeMode]>;
defm SVCLZ : SInstCLS<"svclz", "csilUcUsUiUl",     "aarch64_sve_clz", [VerifyRuntimeMode]>;
defm SVCNT : SInstCLS<"svcnt", "csilUcUsUiUlhfdb", "aarch64_sve_cnt", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Conversion

defm SVEXTB_S : SInstZPZ<"svextb", "sil",    "aarch64_sve_sxtb">;
defm SVEXTB_U : SInstZPZ<"svextb", "UsUiUl", "aarch64_sve_uxtb">;
defm SVEXTH_S : SInstZPZ<"svexth", "il",     "aarch64_sve_sxth">;
defm SVEXTH_U : SInstZPZ<"svexth", "UiUl",   "aarch64_sve_uxth">;
defm SVEXTW_S : SInstZPZ<"svextw", "l",      "aarch64_sve_sxtw">;
defm SVEXTW_U : SInstZPZ<"svextw", "Ul",     "aarch64_sve_uxtw">;

////////////////////////////////////////////////////////////////////////////////
// Reversal

defm SVRBIT : SInstZPZ<"svrbit", "csilUcUsUiUl", "aarch64_sve_rbit">;
defm SVREVB : SInstZPZ<"svrevb", "silUsUiUl",    "aarch64_sve_revb">;
defm SVREVH : SInstZPZ<"svrevh", "ilUiUl",       "aarch64_sve_revh">;
defm SVREVW : SInstZPZ<"svrevw", "lUl",          "aarch64_sve_revw">;

////////////////////////////////////////////////////////////////////////////////
````
- **L729 EN**: Declares TableGen def record `_M`.
  **L729 CN**: 声明 TableGen def 记录 `_M`。
- **L730 EN**: Declares TableGen def record `_X`.
  **L730 CN**: 声明 TableGen def 记录 `_X`。
- **L731 EN**: Declares TableGen def record `_Z`.
  **L731 CN**: 声明 TableGen def 记录 `_Z`。
- **L732 EN**: Closes the current lexical scope or compound statement.
  **L732 CN**: 结束当前词法作用域或复合语句块。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L734 EN**: Declares TableGen defm record `SVCLS`.
  **L734 CN**: 声明 TableGen defm 记录 `SVCLS`。
- **L735 EN**: Declares TableGen defm record `SVCLZ`.
  **L735 CN**: 声明 TableGen defm 记录 `SVCLZ`。
- **L736 EN**: Declares TableGen defm record `SVCNT`.
  **L736 CN**: 声明 TableGen defm 记录 `SVCNT`。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L738 EN**: Separator comment used for visual grouping.
  **L738 CN**: 用于视觉分组的分隔注释。
- **L739 EN**: Comment explains nearby logic, constraints, or intent: `Conversion`.
  **L739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Conversion`。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Declares TableGen defm record `SVEXTB_S`.
  **L741 CN**: 声明 TableGen defm 记录 `SVEXTB_S`。
- **L742 EN**: Declares TableGen defm record `SVEXTB_U`.
  **L742 CN**: 声明 TableGen defm 记录 `SVEXTB_U`。
- **L743 EN**: Declares TableGen defm record `SVEXTH_S`.
  **L743 CN**: 声明 TableGen defm 记录 `SVEXTH_S`。
- **L744 EN**: Declares TableGen defm record `SVEXTH_U`.
  **L744 CN**: 声明 TableGen defm 记录 `SVEXTH_U`。
- **L745 EN**: Declares TableGen defm record `SVEXTW_S`.
  **L745 CN**: 声明 TableGen defm 记录 `SVEXTW_S`。
- **L746 EN**: Declares TableGen defm record `SVEXTW_U`.
  **L746 CN**: 声明 TableGen defm 记录 `SVEXTW_U`。
- **L747 EN**: Blank line separating nearby declarations or logic blocks.
  **L747 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L748 EN**: Separator comment used for visual grouping.
  **L748 CN**: 用于视觉分组的分隔注释。
- **L749 EN**: Comment explains nearby logic, constraints, or intent: `Reversal`.
  **L749 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reversal`。
- **L750 EN**: Blank line separating nearby declarations or logic blocks.
  **L750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L751 EN**: Declares TableGen defm record `SVRBIT`.
  **L751 CN**: 声明 TableGen defm 记录 `SVRBIT`。
- **L752 EN**: Declares TableGen defm record `SVREVB`.
  **L752 CN**: 声明 TableGen defm 记录 `SVREVB`。
- **L753 EN**: Declares TableGen defm record `SVREVH`.
  **L753 CN**: 声明 TableGen defm 记录 `SVREVH`。
- **L754 EN**: Declares TableGen defm record `SVREVW`.
  **L754 CN**: 声明 TableGen defm 记录 `SVREVW`。
- **L755 EN**: Blank line separating nearby declarations or logic blocks.
  **L755 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L756 EN**: Separator comment used for visual grouping.
  **L756 CN**: 用于视觉分组的分隔注释。

### Lines 757-784

````tablegen
// Floating-point arithmetic

defm SVABS_F : SInstZPZ<"svabs", "hfd", "aarch64_sve_fabs">;
defm SVNEG_F : SInstZPZ<"svneg", "hfd", "aarch64_sve_fneg">;

defm SVABD_F  : SInstZPZZ<"svabd",  "hfd", "aarch64_sve_fabd",   "aarch64_sve_fabd_u">;
defm SVADD_F  : SInstZPZZ<"svadd",  "hfd", "aarch64_sve_fadd",   "aarch64_sve_fadd_u">;
defm SVDIV_F  : SInstZPZZ<"svdiv",  "hfd", "aarch64_sve_fdiv",   "aarch64_sve_fdiv_u">;
defm SVDIVR_F : SInstZPZZ<"svdivr", "hfd", "aarch64_sve_fdivr",  "aarch64_sve_fdiv_u", [ReverseMergeAnyBinOp]>;
defm SVMAX_F  : SInstZPZZ<"svmax",  "hfd", "aarch64_sve_fmax",   "aarch64_sve_fmax_u">;
defm SVMAXNM  : SInstZPZZ<"svmaxnm","hfd", "aarch64_sve_fmaxnm", "aarch64_sve_fmaxnm_u">;
defm SVMIN_F  : SInstZPZZ<"svmin",  "hfd", "aarch64_sve_fmin",   "aarch64_sve_fmin_u">;
defm SVMINNM  : SInstZPZZ<"svminnm","hfd", "aarch64_sve_fminnm", "aarch64_sve_fminnm_u">;
defm SVMUL_F  : SInstZPZZ<"svmul",  "hfd", "aarch64_sve_fmul",   "aarch64_sve_fmul_u">;
defm SVMULX   : SInstZPZZ<"svmulx", "hfd", "aarch64_sve_fmulx",  "aarch64_sve_fmulx_u">;
defm SVSUB_F  : SInstZPZZ<"svsub",  "hfd", "aarch64_sve_fsub",   "aarch64_sve_fsub_u">;
defm SVSUBR_F : SInstZPZZ<"svsubr", "hfd", "aarch64_sve_fsubr",  "aarch64_sve_fsub_u", [ReverseMergeAnyBinOp]>;

defm SVRECPX : SInstZPZ<"svrecpx", "hfd", "aarch64_sve_frecpx">;
defm SVRINTA : SInstZPZ<"svrinta", "hfd", "aarch64_sve_frinta">;
defm SVRINTI : SInstZPZ<"svrinti", "hfd", "aarch64_sve_frinti">;
defm SVRINTM : SInstZPZ<"svrintm", "hfd", "aarch64_sve_frintm">;
defm SVRINTN : SInstZPZ<"svrintn", "hfd", "aarch64_sve_frintn">;
defm SVRINTP : SInstZPZ<"svrintp", "hfd", "aarch64_sve_frintp">;
defm SVRINTX : SInstZPZ<"svrintx", "hfd", "aarch64_sve_frintx">;
defm SVRINTZ : SInstZPZ<"svrintz", "hfd", "aarch64_sve_frintz">;
defm SVSQRT  : SInstZPZ<"svsqrt",  "hfd", "aarch64_sve_fsqrt">;

````
- **L757 EN**: Comment explains nearby logic, constraints, or intent: `Floating-point arithmetic`.
  **L757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Floating-point arithmetic`。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Declares TableGen defm record `SVABS_F`.
  **L759 CN**: 声明 TableGen defm 记录 `SVABS_F`。
- **L760 EN**: Declares TableGen defm record `SVNEG_F`.
  **L760 CN**: 声明 TableGen defm 记录 `SVNEG_F`。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L762 EN**: Declares TableGen defm record `SVABD_F`.
  **L762 CN**: 声明 TableGen defm 记录 `SVABD_F`。
- **L763 EN**: Declares TableGen defm record `SVADD_F`.
  **L763 CN**: 声明 TableGen defm 记录 `SVADD_F`。
- **L764 EN**: Declares TableGen defm record `SVDIV_F`.
  **L764 CN**: 声明 TableGen defm 记录 `SVDIV_F`。
- **L765 EN**: Declares TableGen defm record `SVDIVR_F`.
  **L765 CN**: 声明 TableGen defm 记录 `SVDIVR_F`。
- **L766 EN**: Declares TableGen defm record `SVMAX_F`.
  **L766 CN**: 声明 TableGen defm 记录 `SVMAX_F`。
- **L767 EN**: Declares TableGen defm record `SVMAXNM`.
  **L767 CN**: 声明 TableGen defm 记录 `SVMAXNM`。
- **L768 EN**: Declares TableGen defm record `SVMIN_F`.
  **L768 CN**: 声明 TableGen defm 记录 `SVMIN_F`。
- **L769 EN**: Declares TableGen defm record `SVMINNM`.
  **L769 CN**: 声明 TableGen defm 记录 `SVMINNM`。
- **L770 EN**: Declares TableGen defm record `SVMUL_F`.
  **L770 CN**: 声明 TableGen defm 记录 `SVMUL_F`。
- **L771 EN**: Declares TableGen defm record `SVMULX`.
  **L771 CN**: 声明 TableGen defm 记录 `SVMULX`。
- **L772 EN**: Declares TableGen defm record `SVSUB_F`.
  **L772 CN**: 声明 TableGen defm 记录 `SVSUB_F`。
- **L773 EN**: Declares TableGen defm record `SVSUBR_F`.
  **L773 CN**: 声明 TableGen defm 记录 `SVSUBR_F`。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L775 EN**: Declares TableGen defm record `SVRECPX`.
  **L775 CN**: 声明 TableGen defm 记录 `SVRECPX`。
- **L776 EN**: Declares TableGen defm record `SVRINTA`.
  **L776 CN**: 声明 TableGen defm 记录 `SVRINTA`。
- **L777 EN**: Declares TableGen defm record `SVRINTI`.
  **L777 CN**: 声明 TableGen defm 记录 `SVRINTI`。
- **L778 EN**: Declares TableGen defm record `SVRINTM`.
  **L778 CN**: 声明 TableGen defm 记录 `SVRINTM`。
- **L779 EN**: Declares TableGen defm record `SVRINTN`.
  **L779 CN**: 声明 TableGen defm 记录 `SVRINTN`。
- **L780 EN**: Declares TableGen defm record `SVRINTP`.
  **L780 CN**: 声明 TableGen defm 记录 `SVRINTP`。
- **L781 EN**: Declares TableGen defm record `SVRINTX`.
  **L781 CN**: 声明 TableGen defm 记录 `SVRINTX`。
- **L782 EN**: Declares TableGen defm record `SVRINTZ`.
  **L782 CN**: 声明 TableGen defm 记录 `SVRINTZ`。
- **L783 EN**: Declares TableGen defm record `SVSQRT`.
  **L783 CN**: 声明 TableGen defm 记录 `SVSQRT`。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 785-812

````tablegen
let SVETargetGuard = "sve2p2|sme2p2", SMETargetGuard = "sve2p2|sme2p2" in {
defm SVRINT32X : SInstZPZ<"svrint32x", "fd", "aarch64_sve_frint32x">;
defm SVRINT32Z : SInstZPZ<"svrint32z", "fd", "aarch64_sve_frint32z">;
defm SVRINT64X : SInstZPZ<"svrint64x", "fd", "aarch64_sve_frint64x">;
defm SVRINT64Z : SInstZPZ<"svrint64z", "fd", "aarch64_sve_frint64z">;
}

let SMETargetGuard = "sme2,ssve-fexpa" in {
def SVEXPA  : SInst<"svexpa[_{d}]",  "du",   "hfd", MergeNone, "aarch64_sve_fexpa_x", [VerifyRuntimeMode]>;
}

let SMETargetGuard = InvalidMode in {
def SVTMAD  : SInst<"svtmad[_{d}]",  "dddi", "hfd", MergeNone, "aarch64_sve_ftmad_x", [], [ImmCheck<2, ImmCheck0_7>]>;
def SVTSMUL : SInst<"svtsmul[_{d}]", "ddu",  "hfd", MergeNone, "aarch64_sve_ftsmul_x">;
def SVTSSEL : SInst<"svtssel[_{d}]", "ddu",  "hfd", MergeNone, "aarch64_sve_ftssel_x">;
}

def SVSCALE_M : SInst<"svscale[_{d}]",   "dPdx", "hfd", MergeOp1,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
def SVSCALE_X : SInst<"svscale[_{d}]",   "dPdx", "hfd", MergeAny,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
def SVSCALE_Z : SInst<"svscale[_{d}]",   "dPdx", "hfd", MergeZero, "aarch64_sve_fscale", [VerifyRuntimeMode]>;

def SVSCALE_N_M : SInst<"svscale[_n_{d}]", "dPdK", "hfd", MergeOp1,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
def SVSCALE_N_X : SInst<"svscale[_n_{d}]", "dPdK", "hfd", MergeAny,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
def SVSCALE_N_Z : SInst<"svscale[_n_{d}]", "dPdK", "hfd", MergeZero, "aarch64_sve_fscale", [VerifyRuntimeMode]>;

let SVETargetGuard = "sve-bfscale", SMETargetGuard = "sve-bfscale,sme2" in {
  def SVBFSCALE_M : SInst<"svscale[_{d}]",   "dPdx", "b", MergeOp1,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
  def SVBFSCALE_X : SInst<"svscale[_{d}]",   "dPdx", "b", MergeAny,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
````
- **L785 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p2|sme2p2", SMETargetGuard = "sve2p2|sme2p2" in {`.
  **L785 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p2|sme2p2", SMETargetGuard = "sve2p2|sme2p2" in {`。
- **L786 EN**: Declares TableGen defm record `SVRINT32X`.
  **L786 CN**: 声明 TableGen defm 记录 `SVRINT32X`。
- **L787 EN**: Declares TableGen defm record `SVRINT32Z`.
  **L787 CN**: 声明 TableGen defm 记录 `SVRINT32Z`。
- **L788 EN**: Declares TableGen defm record `SVRINT64X`.
  **L788 CN**: 声明 TableGen defm 记录 `SVRINT64X`。
- **L789 EN**: Declares TableGen defm record `SVRINT64Z`.
  **L789 CN**: 声明 TableGen defm 记录 `SVRINT64Z`。
- **L790 EN**: Closes the current lexical scope or compound statement.
  **L790 CN**: 结束当前词法作用域或复合语句块。
- **L791 EN**: Blank line separating nearby declarations or logic blocks.
  **L791 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L792 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,ssve-fexpa" in {`.
  **L792 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,ssve-fexpa" in {`。
- **L793 EN**: Declares TableGen def record `SVEXPA`.
  **L793 CN**: 声明 TableGen def 记录 `SVEXPA`。
- **L794 EN**: Closes the current lexical scope or compound statement.
  **L794 CN**: 结束当前词法作用域或复合语句块。
- **L795 EN**: Blank line separating nearby declarations or logic blocks.
  **L795 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L796 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L796 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L797 EN**: Declares TableGen def record `SVTMAD`.
  **L797 CN**: 声明 TableGen def 记录 `SVTMAD`。
- **L798 EN**: Declares TableGen def record `SVTSMUL`.
  **L798 CN**: 声明 TableGen def 记录 `SVTSMUL`。
- **L799 EN**: Declares TableGen def record `SVTSSEL`.
  **L799 CN**: 声明 TableGen def 记录 `SVTSSEL`。
- **L800 EN**: Closes the current lexical scope or compound statement.
  **L800 CN**: 结束当前词法作用域或复合语句块。
- **L801 EN**: Blank line separating nearby declarations or logic blocks.
  **L801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L802 EN**: Declares TableGen def record `SVSCALE_M`.
  **L802 CN**: 声明 TableGen def 记录 `SVSCALE_M`。
- **L803 EN**: Declares TableGen def record `SVSCALE_X`.
  **L803 CN**: 声明 TableGen def 记录 `SVSCALE_X`。
- **L804 EN**: Declares TableGen def record `SVSCALE_Z`.
  **L804 CN**: 声明 TableGen def 记录 `SVSCALE_Z`。
- **L805 EN**: Blank line separating nearby declarations or logic blocks.
  **L805 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L806 EN**: Declares TableGen def record `SVSCALE_N_M`.
  **L806 CN**: 声明 TableGen def 记录 `SVSCALE_N_M`。
- **L807 EN**: Declares TableGen def record `SVSCALE_N_X`.
  **L807 CN**: 声明 TableGen def 记录 `SVSCALE_N_X`。
- **L808 EN**: Declares TableGen def record `SVSCALE_N_Z`.
  **L808 CN**: 声明 TableGen def 记录 `SVSCALE_N_Z`。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L810 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-bfscale", SMETargetGuard = "sve-bfscale,sme2" in {`.
  **L810 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-bfscale", SMETargetGuard = "sve-bfscale,sme2" in {`。
- **L811 EN**: Declares TableGen def record `SVBFSCALE_M`.
  **L811 CN**: 声明 TableGen def 记录 `SVBFSCALE_M`。
- **L812 EN**: Declares TableGen def record `SVBFSCALE_X`.
  **L812 CN**: 声明 TableGen def 记录 `SVBFSCALE_X`。

### Lines 813-840

````tablegen
  def SVBFSCALE_Z : SInst<"svscale[_{d}]",   "dPdx", "b", MergeZero, "aarch64_sve_fscale", [VerifyRuntimeMode]>;

  def SVBFSCALE_N_M : SInst<"svscale[_n_{d}]", "dPdK", "b", MergeOp1,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
  def SVBFSCALE_N_X : SInst<"svscale[_n_{d}]", "dPdK", "b", MergeAny,  "aarch64_sve_fscale", [VerifyRuntimeMode]>;
  def SVBFSCALE_N_Z : SInst<"svscale[_n_{d}]", "dPdK", "b", MergeZero, "aarch64_sve_fscale", [VerifyRuntimeMode]>;
}

defm SVMAD_F  : SInstZPZZZ<"svmad",  "hfd", "aarch64_sve_fmad",  "aarch64_sve_fmla_u",  [VerifyRuntimeMode, ReverseMergeAnyAccOp]>;
defm SVMLA_F  : SInstZPZZZ<"svmla",  "hfd", "aarch64_sve_fmla",  "aarch64_sve_fmla_u", [VerifyRuntimeMode]>;
defm SVMLS_F  : SInstZPZZZ<"svmls",  "hfd", "aarch64_sve_fmls",  "aarch64_sve_fmls_u", [VerifyRuntimeMode]>;
defm SVMSB_F  : SInstZPZZZ<"svmsb",  "hfd", "aarch64_sve_fmsb",  "aarch64_sve_fmls_u",  [VerifyRuntimeMode, ReverseMergeAnyAccOp]>;
defm SVNMAD_F : SInstZPZZZ<"svnmad", "hfd", "aarch64_sve_fnmad", "aarch64_sve_fnmla_u", [VerifyRuntimeMode, ReverseMergeAnyAccOp]>;
defm SVNMLA_F : SInstZPZZZ<"svnmla", "hfd", "aarch64_sve_fnmla", "aarch64_sve_fnmla_u", [VerifyRuntimeMode]>;
defm SVNMLS_F : SInstZPZZZ<"svnmls", "hfd", "aarch64_sve_fnmls", "aarch64_sve_fnmls_u", [VerifyRuntimeMode]>;
defm SVNMSB_F : SInstZPZZZ<"svnmsb", "hfd", "aarch64_sve_fnmsb", "aarch64_sve_fnmls_u", [VerifyRuntimeMode, ReverseMergeAnyAccOp]>;

def SVCADD_M : SInst<"svcadd[_{d}]", "dPddi",  "hfd", MergeOp1,  "aarch64_sve_fcadd", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckComplexRot90_270>]>;
def SVCADD_X : SInst<"svcadd[_{d}]", "dPddi",  "hfd", MergeAny,  "aarch64_sve_fcadd", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckComplexRot90_270>]>;
def SVCADD_Z : SInst<"svcadd[_{d}]", "dPddi",  "hfd", MergeZero, "aarch64_sve_fcadd", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckComplexRot90_270>]>;
def SVCMLA_M : SInst<"svcmla[_{d}]", "dPdddi", "hfd", MergeOp1,  "aarch64_sve_fcmla", [VerifyRuntimeMode], [ImmCheck<4, ImmCheckComplexRotAll90>]>;
def SVCMLA_X : SInst<"svcmla[_{d}]", "dPdddi", "hfd", MergeAny,  "aarch64_sve_fcmla", [VerifyRuntimeMode], [ImmCheck<4, ImmCheckComplexRotAll90>]>;
def SVCMLA_Z : SInst<"svcmla[_{d}]", "dPdddi", "hfd", MergeZero, "aarch64_sve_fcmla", [VerifyRuntimeMode], [ImmCheck<4, ImmCheckComplexRotAll90>]>;

def SVCMLA_LANE : SInst<"svcmla_lane[_{d}]", "ddddii", "hf",  MergeNone, "aarch64_sve_fcmla_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndexCompRotate, 2>, ImmCheck<4, ImmCheckComplexRotAll90>]>;
def SVMLA_LANE  : SInst<"svmla_lane[_{d}]",  "ddddi",  "hfd", MergeNone, "aarch64_sve_fmla_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLS_LANE  : SInst<"svmls_lane[_{d}]",  "ddddi",  "hfd", MergeNone, "aarch64_sve_fmls_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMUL_LANE  : SInst<"svmul_lane[_{d}]",  "dddi",   "hfd", MergeNone, "aarch64_sve_fmul_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;

````
- **L813 EN**: Declares TableGen def record `SVBFSCALE_Z`.
  **L813 CN**: 声明 TableGen def 记录 `SVBFSCALE_Z`。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Declares TableGen def record `SVBFSCALE_N_M`.
  **L815 CN**: 声明 TableGen def 记录 `SVBFSCALE_N_M`。
- **L816 EN**: Declares TableGen def record `SVBFSCALE_N_X`.
  **L816 CN**: 声明 TableGen def 记录 `SVBFSCALE_N_X`。
- **L817 EN**: Declares TableGen def record `SVBFSCALE_N_Z`.
  **L817 CN**: 声明 TableGen def 记录 `SVBFSCALE_N_Z`。
- **L818 EN**: Closes the current lexical scope or compound statement.
  **L818 CN**: 结束当前词法作用域或复合语句块。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L820 EN**: Declares TableGen defm record `SVMAD_F`.
  **L820 CN**: 声明 TableGen defm 记录 `SVMAD_F`。
- **L821 EN**: Declares TableGen defm record `SVMLA_F`.
  **L821 CN**: 声明 TableGen defm 记录 `SVMLA_F`。
- **L822 EN**: Declares TableGen defm record `SVMLS_F`.
  **L822 CN**: 声明 TableGen defm 记录 `SVMLS_F`。
- **L823 EN**: Declares TableGen defm record `SVMSB_F`.
  **L823 CN**: 声明 TableGen defm 记录 `SVMSB_F`。
- **L824 EN**: Declares TableGen defm record `SVNMAD_F`.
  **L824 CN**: 声明 TableGen defm 记录 `SVNMAD_F`。
- **L825 EN**: Declares TableGen defm record `SVNMLA_F`.
  **L825 CN**: 声明 TableGen defm 记录 `SVNMLA_F`。
- **L826 EN**: Declares TableGen defm record `SVNMLS_F`.
  **L826 CN**: 声明 TableGen defm 记录 `SVNMLS_F`。
- **L827 EN**: Declares TableGen defm record `SVNMSB_F`.
  **L827 CN**: 声明 TableGen defm 记录 `SVNMSB_F`。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Declares TableGen def record `SVCADD_M`.
  **L829 CN**: 声明 TableGen def 记录 `SVCADD_M`。
- **L830 EN**: Declares TableGen def record `SVCADD_X`.
  **L830 CN**: 声明 TableGen def 记录 `SVCADD_X`。
- **L831 EN**: Declares TableGen def record `SVCADD_Z`.
  **L831 CN**: 声明 TableGen def 记录 `SVCADD_Z`。
- **L832 EN**: Declares TableGen def record `SVCMLA_M`.
  **L832 CN**: 声明 TableGen def 记录 `SVCMLA_M`。
- **L833 EN**: Declares TableGen def record `SVCMLA_X`.
  **L833 CN**: 声明 TableGen def 记录 `SVCMLA_X`。
- **L834 EN**: Declares TableGen def record `SVCMLA_Z`.
  **L834 CN**: 声明 TableGen def 记录 `SVCMLA_Z`。
- **L835 EN**: Blank line separating nearby declarations or logic blocks.
  **L835 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L836 EN**: Declares TableGen def record `SVCMLA_LANE`.
  **L836 CN**: 声明 TableGen def 记录 `SVCMLA_LANE`。
- **L837 EN**: Declares TableGen def record `SVMLA_LANE`.
  **L837 CN**: 声明 TableGen def 记录 `SVMLA_LANE`。
- **L838 EN**: Declares TableGen def record `SVMLS_LANE`.
  **L838 CN**: 声明 TableGen def 记录 `SVMLS_LANE`。
- **L839 EN**: Declares TableGen def record `SVMUL_LANE`.
  **L839 CN**: 声明 TableGen def 记录 `SVMUL_LANE`。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 841-868

````tablegen
def SVRECPE  : SInst<"svrecpe[_{d}]",  "dd",  "hfd", MergeNone, "aarch64_sve_frecpe_x", [VerifyRuntimeMode]>;
def SVRECPS  : SInst<"svrecps[_{d}]",  "ddd", "hfd", MergeNone, "aarch64_sve_frecps_x", [VerifyRuntimeMode]>;
def SVRSQRTE : SInst<"svrsqrte[_{d}]", "dd",  "hfd", MergeNone, "aarch64_sve_frsqrte_x", [VerifyRuntimeMode]>;
def SVRSQRTS : SInst<"svrsqrts[_{d}]", "ddd", "hfd", MergeNone, "aarch64_sve_frsqrts_x", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Floating-point reductions

let SMETargetGuard = InvalidMode in {
def SVFADDA : SInst<"svadda[_{d}]",   "sPsd", "hfd", MergeNone, "aarch64_sve_fadda">;
}

def SVFADDV   : SInst<"svaddv[_{d}]",   "sPd",  "hfd", MergeNone, "aarch64_sve_faddv", [VerifyRuntimeMode]>;
def SVFMAXV   : SInst<"svmaxv[_{d}]",   "sPd",  "hfd", MergeNone, "aarch64_sve_fmaxv", [VerifyRuntimeMode]>;
def SVFMAXNMV : SInst<"svmaxnmv[_{d}]", "sPd",  "hfd", MergeNone, "aarch64_sve_fmaxnmv", [VerifyRuntimeMode]>;
def SVFMINV   : SInst<"svminv[_{d}]",   "sPd",  "hfd", MergeNone, "aarch64_sve_fminv", [VerifyRuntimeMode]>;
def SVFMINNMV : SInst<"svminnmv[_{d}]", "sPd",  "hfd", MergeNone, "aarch64_sve_fminnmv", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Floating-point comparisons

def SVACGE  : SInst<"svacge[_{d}]",  "PPdd", "hfd", MergeNone, "aarch64_sve_facge", [VerifyRuntimeMode]>;
def SVACGT  : SInst<"svacgt[_{d}]",  "PPdd", "hfd", MergeNone, "aarch64_sve_facgt", [VerifyRuntimeMode]>;
def SVACLE  : SInst<"svacle[_{d}]",  "PPdd", "hfd", MergeNone, "aarch64_sve_facge", [ReverseCompare, VerifyRuntimeMode]>;
def SVACLT  : SInst<"svaclt[_{d}]",  "PPdd", "hfd", MergeNone, "aarch64_sve_facgt", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPUO : SInst<"svcmpuo[_{d}]", "PPdd", "hfd", MergeNone, "aarch64_sve_fcmpuo", [VerifyRuntimeMode]>;

def SVACGE_N  : SInst<"svacge[_n_{d}]",  "PPda", "hfd", MergeNone, "aarch64_sve_facge", [VerifyRuntimeMode]>;
````
- **L841 EN**: Declares TableGen def record `SVRECPE`.
  **L841 CN**: 声明 TableGen def 记录 `SVRECPE`。
- **L842 EN**: Declares TableGen def record `SVRECPS`.
  **L842 CN**: 声明 TableGen def 记录 `SVRECPS`。
- **L843 EN**: Declares TableGen def record `SVRSQRTE`.
  **L843 CN**: 声明 TableGen def 记录 `SVRSQRTE`。
- **L844 EN**: Declares TableGen def record `SVRSQRTS`.
  **L844 CN**: 声明 TableGen def 记录 `SVRSQRTS`。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L846 EN**: Separator comment used for visual grouping.
  **L846 CN**: 用于视觉分组的分隔注释。
- **L847 EN**: Comment explains nearby logic, constraints, or intent: `Floating-point reductions`.
  **L847 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Floating-point reductions`。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L849 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L850 EN**: Declares TableGen def record `SVFADDA`.
  **L850 CN**: 声明 TableGen def 记录 `SVFADDA`。
- **L851 EN**: Closes the current lexical scope or compound statement.
  **L851 CN**: 结束当前词法作用域或复合语句块。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Declares TableGen def record `SVFADDV`.
  **L853 CN**: 声明 TableGen def 记录 `SVFADDV`。
- **L854 EN**: Declares TableGen def record `SVFMAXV`.
  **L854 CN**: 声明 TableGen def 记录 `SVFMAXV`。
- **L855 EN**: Declares TableGen def record `SVFMAXNMV`.
  **L855 CN**: 声明 TableGen def 记录 `SVFMAXNMV`。
- **L856 EN**: Declares TableGen def record `SVFMINV`.
  **L856 CN**: 声明 TableGen def 记录 `SVFMINV`。
- **L857 EN**: Declares TableGen def record `SVFMINNMV`.
  **L857 CN**: 声明 TableGen def 记录 `SVFMINNMV`。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L859 EN**: Separator comment used for visual grouping.
  **L859 CN**: 用于视觉分组的分隔注释。
- **L860 EN**: Comment explains nearby logic, constraints, or intent: `Floating-point comparisons`.
  **L860 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Floating-point comparisons`。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L862 EN**: Declares TableGen def record `SVACGE`.
  **L862 CN**: 声明 TableGen def 记录 `SVACGE`。
- **L863 EN**: Declares TableGen def record `SVACGT`.
  **L863 CN**: 声明 TableGen def 记录 `SVACGT`。
- **L864 EN**: Declares TableGen def record `SVACLE`.
  **L864 CN**: 声明 TableGen def 记录 `SVACLE`。
- **L865 EN**: Declares TableGen def record `SVACLT`.
  **L865 CN**: 声明 TableGen def 记录 `SVACLT`。
- **L866 EN**: Declares TableGen def record `SVCMPUO`.
  **L866 CN**: 声明 TableGen def 记录 `SVCMPUO`。
- **L867 EN**: Blank line separating nearby declarations or logic blocks.
  **L867 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L868 EN**: Declares TableGen def record `SVACGE_N`.
  **L868 CN**: 声明 TableGen def 记录 `SVACGE_N`。

### Lines 869-896

````tablegen
def SVACGT_N  : SInst<"svacgt[_n_{d}]",  "PPda", "hfd", MergeNone, "aarch64_sve_facgt", [VerifyRuntimeMode]>;
def SVACLE_N  : SInst<"svacle[_n_{d}]",  "PPda", "hfd", MergeNone, "aarch64_sve_facge", [ReverseCompare, VerifyRuntimeMode]>;
def SVACLT_N  : SInst<"svaclt[_n_{d}]",  "PPda", "hfd", MergeNone, "aarch64_sve_facgt", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPUO_N : SInst<"svcmpuo[_n_{d}]", "PPda", "hfd", MergeNone, "aarch64_sve_fcmpuo", [VerifyRuntimeMode]>;

def SVCMPEQ_F : SInst<"svcmpeq[_{d}]", "PPdd", "hfd", MergeNone, "aarch64_sve_fcmpeq", [VerifyRuntimeMode]>;
def SVCMPNE_F : SInst<"svcmpne[_{d}]", "PPdd", "hfd", MergeNone, "aarch64_sve_fcmpne", [VerifyRuntimeMode]>;
def SVCMPGE_F : SInst<"svcmpge[_{d}]", "PPdd", "hfd", MergeNone, "aarch64_sve_fcmpge", [VerifyRuntimeMode]>;
def SVCMPGT_F : SInst<"svcmpgt[_{d}]", "PPdd", "hfd", MergeNone, "aarch64_sve_fcmpgt", [VerifyRuntimeMode]>;
def SVCMPLE_F : SInst<"svcmple[_{d}]", "PPdd", "hfd", MergeNone, "aarch64_sve_fcmpge", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPLT_F : SInst<"svcmplt[_{d}]", "PPdd", "hfd", MergeNone, "aarch64_sve_fcmpgt", [ReverseCompare, VerifyRuntimeMode]>;

def SVCMPEQ_F_N : SInst<"svcmpeq[_n_{d}]", "PPda", "hfd", MergeNone, "aarch64_sve_fcmpeq", [VerifyRuntimeMode]>;
def SVCMPNE_F_N : SInst<"svcmpne[_n_{d}]", "PPda", "hfd", MergeNone, "aarch64_sve_fcmpne", [VerifyRuntimeMode]>;
def SVCMPGE_F_N : SInst<"svcmpge[_n_{d}]", "PPda", "hfd", MergeNone, "aarch64_sve_fcmpge", [VerifyRuntimeMode]>;
def SVCMPGT_F_N : SInst<"svcmpgt[_n_{d}]", "PPda", "hfd", MergeNone, "aarch64_sve_fcmpgt", [VerifyRuntimeMode]>;
def SVCMPLE_F_N : SInst<"svcmple[_n_{d}]", "PPda", "hfd", MergeNone, "aarch64_sve_fcmpge", [ReverseCompare, VerifyRuntimeMode]>;
def SVCMPLT_F_N : SInst<"svcmplt[_n_{d}]", "PPda", "hfd", MergeNone, "aarch64_sve_fcmpgt", [ReverseCompare, VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Floating-point conversions

multiclass SInstCvtMXZ<
    string name, string m_types, string xz_types, string types,
    string intrinsic, list<FlagType> flags = [IsOverloadNone]> {
  def _M : SInst<name, m_types,  types, MergeOp1,     intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
  def _X : SInst<name, xz_types, types, MergeAnyExp,  intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
  def _Z : SInst<name, xz_types, types, MergeZeroExp, intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
````
- **L869 EN**: Declares TableGen def record `SVACGT_N`.
  **L869 CN**: 声明 TableGen def 记录 `SVACGT_N`。
- **L870 EN**: Declares TableGen def record `SVACLE_N`.
  **L870 CN**: 声明 TableGen def 记录 `SVACLE_N`。
- **L871 EN**: Declares TableGen def record `SVACLT_N`.
  **L871 CN**: 声明 TableGen def 记录 `SVACLT_N`。
- **L872 EN**: Declares TableGen def record `SVCMPUO_N`.
  **L872 CN**: 声明 TableGen def 记录 `SVCMPUO_N`。
- **L873 EN**: Blank line separating nearby declarations or logic blocks.
  **L873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L874 EN**: Declares TableGen def record `SVCMPEQ_F`.
  **L874 CN**: 声明 TableGen def 记录 `SVCMPEQ_F`。
- **L875 EN**: Declares TableGen def record `SVCMPNE_F`.
  **L875 CN**: 声明 TableGen def 记录 `SVCMPNE_F`。
- **L876 EN**: Declares TableGen def record `SVCMPGE_F`.
  **L876 CN**: 声明 TableGen def 记录 `SVCMPGE_F`。
- **L877 EN**: Declares TableGen def record `SVCMPGT_F`.
  **L877 CN**: 声明 TableGen def 记录 `SVCMPGT_F`。
- **L878 EN**: Declares TableGen def record `SVCMPLE_F`.
  **L878 CN**: 声明 TableGen def 记录 `SVCMPLE_F`。
- **L879 EN**: Declares TableGen def record `SVCMPLT_F`.
  **L879 CN**: 声明 TableGen def 记录 `SVCMPLT_F`。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L881 EN**: Declares TableGen def record `SVCMPEQ_F_N`.
  **L881 CN**: 声明 TableGen def 记录 `SVCMPEQ_F_N`。
- **L882 EN**: Declares TableGen def record `SVCMPNE_F_N`.
  **L882 CN**: 声明 TableGen def 记录 `SVCMPNE_F_N`。
- **L883 EN**: Declares TableGen def record `SVCMPGE_F_N`.
  **L883 CN**: 声明 TableGen def 记录 `SVCMPGE_F_N`。
- **L884 EN**: Declares TableGen def record `SVCMPGT_F_N`.
  **L884 CN**: 声明 TableGen def 记录 `SVCMPGT_F_N`。
- **L885 EN**: Declares TableGen def record `SVCMPLE_F_N`.
  **L885 CN**: 声明 TableGen def 记录 `SVCMPLE_F_N`。
- **L886 EN**: Declares TableGen def record `SVCMPLT_F_N`.
  **L886 CN**: 声明 TableGen def 记录 `SVCMPLT_F_N`。
- **L887 EN**: Blank line separating nearby declarations or logic blocks.
  **L887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L888 EN**: Separator comment used for visual grouping.
  **L888 CN**: 用于视觉分组的分隔注释。
- **L889 EN**: Comment explains nearby logic, constraints, or intent: `Floating-point conversions`.
  **L889 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Floating-point conversions`。
- **L890 EN**: Blank line separating nearby declarations or logic blocks.
  **L890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L891 EN**: Declares TableGen multiclass record `SInstCvtMXZ`.
  **L891 CN**: 声明 TableGen multiclass 记录 `SInstCvtMXZ`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `string name, string m_types, string xz_types, string types,`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`string name, string m_types, string xz_types, string types,`。
- **L893 EN**: Continues the surrounding expression or declaration: `string intrinsic, list<FlagType> flags = [IsOverloadNone]> {`.
  **L893 CN**: 继续构造周围的表达式或声明：`string intrinsic, list<FlagType> flags = [IsOverloadNone]> {`。
- **L894 EN**: Declares TableGen def record `_M`.
  **L894 CN**: 声明 TableGen def 记录 `_M`。
- **L895 EN**: Declares TableGen def record `_X`.
  **L895 CN**: 声明 TableGen def 记录 `_X`。
- **L896 EN**: Declares TableGen def record `_Z`.
  **L896 CN**: 声明 TableGen def 记录 `_Z`。

### Lines 897-924

````tablegen
}

multiclass SInstCvtMX<string name, string m_types, string xz_types,
                      string types, string intrinsic,
                      list<FlagType> flags = [IsOverloadNone]> {
  def _M : SInst<name, m_types,  types, MergeOp1,     intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
  def _X : SInst<name, xz_types, types, MergeAnyExp,  intrinsic, !listconcat(flags, [VerifyRuntimeMode])>;
}

// svcvt_s##_f16
defm SVFCVTZS_S16_F16 : SInstCvtMXZ<"svcvt_s16[_f16]", "ddPO", "dPO", "s",  "aarch64_sve_fcvtzs", [IsOverloadFirstandLast]>;
defm SVFCVTZS_S32_F16 : SInstCvtMXZ<"svcvt_s32[_f16]", "ddPO", "dPO", "i",  "aarch64_sve_fcvtzs_i32f16">;
defm SVFCVTZS_S64_F16 : SInstCvtMXZ<"svcvt_s64[_f16]", "ddPO", "dPO", "l",  "aarch64_sve_fcvtzs_i64f16">;

// svcvt_s##_f32
defm SVFCVTZS_S32_F32 : SInstCvtMXZ<"svcvt_s32[_f32]", "ddPM", "dPM", "i",  "aarch64_sve_fcvtzs", [IsOverloadFirstandLast]>;
defm SVFCVTZS_S64_F32 : SInstCvtMXZ<"svcvt_s64[_f32]", "ddPM", "dPM", "l",  "aarch64_sve_fcvtzs_i64f32">;

// svcvt_s##_f64
defm SVFCVTZS_S32_F64 : SInstCvtMXZ<"svcvt_s32[_f64]", "ttPd", "tPd", "d",  "aarch64_sve_fcvtzs_i32f64">;
defm SVFCVTZS_S64_F64 : SInstCvtMXZ<"svcvt_s64[_f64]", "ddPN", "dPN", "l",  "aarch64_sve_fcvtzs", [IsOverloadFirstandLast]>;

// svcvt_u##_f16
defm SVFCVTZU_U16_F16 : SInstCvtMXZ<"svcvt_u16[_f16]", "ddPO", "dPO", "Us", "aarch64_sve_fcvtzu", [IsOverloadFirstandLast]>;
defm SVFCVTZU_U32_F16 : SInstCvtMXZ<"svcvt_u32[_f16]", "ddPO", "dPO", "Ui", "aarch64_sve_fcvtzu_i32f16">;
defm SVFCVTZU_U64_F16 : SInstCvtMXZ<"svcvt_u64[_f16]", "ddPO", "dPO", "Ul", "aarch64_sve_fcvtzu_i64f16">;

// svcvt_u##_f32
````
- **L897 EN**: Closes the current lexical scope or compound statement.
  **L897 CN**: 结束当前词法作用域或复合语句块。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L899 EN**: Declares TableGen multiclass record `SInstCvtMX`.
  **L899 CN**: 声明 TableGen multiclass 记录 `SInstCvtMX`。
- **L900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `string types, string intrinsic,`.
  **L900 CN**: 继续一个多行参数列表、初始化器或聚合项：`string types, string intrinsic,`。
- **L901 EN**: Continues the surrounding expression or declaration: `list<FlagType> flags = [IsOverloadNone]> {`.
  **L901 CN**: 继续构造周围的表达式或声明：`list<FlagType> flags = [IsOverloadNone]> {`。
- **L902 EN**: Declares TableGen def record `_M`.
  **L902 CN**: 声明 TableGen def 记录 `_M`。
- **L903 EN**: Declares TableGen def record `_X`.
  **L903 CN**: 声明 TableGen def 记录 `_X`。
- **L904 EN**: Closes the current lexical scope or compound statement.
  **L904 CN**: 结束当前词法作用域或复合语句块。
- **L905 EN**: Blank line separating nearby declarations or logic blocks.
  **L905 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L906 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_s##_f16`.
  **L906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_s##_f16`。
- **L907 EN**: Declares TableGen defm record `SVFCVTZS_S16_F16`.
  **L907 CN**: 声明 TableGen defm 记录 `SVFCVTZS_S16_F16`。
- **L908 EN**: Declares TableGen defm record `SVFCVTZS_S32_F16`.
  **L908 CN**: 声明 TableGen defm 记录 `SVFCVTZS_S32_F16`。
- **L909 EN**: Declares TableGen defm record `SVFCVTZS_S64_F16`.
  **L909 CN**: 声明 TableGen defm 记录 `SVFCVTZS_S64_F16`。
- **L910 EN**: Blank line separating nearby declarations or logic blocks.
  **L910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L911 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_s##_f32`.
  **L911 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_s##_f32`。
- **L912 EN**: Declares TableGen defm record `SVFCVTZS_S32_F32`.
  **L912 CN**: 声明 TableGen defm 记录 `SVFCVTZS_S32_F32`。
- **L913 EN**: Declares TableGen defm record `SVFCVTZS_S64_F32`.
  **L913 CN**: 声明 TableGen defm 记录 `SVFCVTZS_S64_F32`。
- **L914 EN**: Blank line separating nearby declarations or logic blocks.
  **L914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L915 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_s##_f64`.
  **L915 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_s##_f64`。
- **L916 EN**: Declares TableGen defm record `SVFCVTZS_S32_F64`.
  **L916 CN**: 声明 TableGen defm 记录 `SVFCVTZS_S32_F64`。
- **L917 EN**: Declares TableGen defm record `SVFCVTZS_S64_F64`.
  **L917 CN**: 声明 TableGen defm 记录 `SVFCVTZS_S64_F64`。
- **L918 EN**: Blank line separating nearby declarations or logic blocks.
  **L918 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L919 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_u##_f16`.
  **L919 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_u##_f16`。
- **L920 EN**: Declares TableGen defm record `SVFCVTZU_U16_F16`.
  **L920 CN**: 声明 TableGen defm 记录 `SVFCVTZU_U16_F16`。
- **L921 EN**: Declares TableGen defm record `SVFCVTZU_U32_F16`.
  **L921 CN**: 声明 TableGen defm 记录 `SVFCVTZU_U32_F16`。
- **L922 EN**: Declares TableGen defm record `SVFCVTZU_U64_F16`.
  **L922 CN**: 声明 TableGen defm 记录 `SVFCVTZU_U64_F16`。
- **L923 EN**: Blank line separating nearby declarations or logic blocks.
  **L923 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L924 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_u##_f32`.
  **L924 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_u##_f32`。

### Lines 925-952

````tablegen
defm SVFCVTZU_U32_F32 : SInstCvtMXZ<"svcvt_u32[_f32]", "ddPM", "dPM", "Ui", "aarch64_sve_fcvtzu", [IsOverloadFirstandLast]>;
defm SVFCVTZU_U64_F32 : SInstCvtMXZ<"svcvt_u64[_f32]", "ddPM", "dPM", "Ul", "aarch64_sve_fcvtzu_i64f32">;

// svcvt_u##_f64
defm SVFCVTZU_U32_F64 : SInstCvtMXZ<"svcvt_u32[_f64]", "zzPd", "zPd", "d",  "aarch64_sve_fcvtzu_i32f64">;
defm SVFCVTZU_U64_F64 : SInstCvtMXZ<"svcvt_u64[_f64]", "ddPN", "dPN", "Ul", "aarch64_sve_fcvtzu", [IsOverloadFirstandLast]>;

// svcvt_f16_s##
defm SVFCVTZS_F16_S16 : SInstCvtMXZ<"svcvt_f16[_s16]", "OOPd", "OPd", "s",  "aarch64_sve_scvtf", [IsOverloadFirstandLast]>;
defm SVFCVTZS_F16_S32 : SInstCvtMXZ<"svcvt_f16[_s32]", "OOPd", "OPd", "i",  "aarch64_sve_scvtf_f16i32">;
defm SVFCVTZS_F16_S64 : SInstCvtMXZ<"svcvt_f16[_s64]", "OOPd", "OPd", "l",  "aarch64_sve_scvtf_f16i64">;

// svcvt_f32_s##
defm SVFCVTZS_F32_S32 : SInstCvtMXZ<"svcvt_f32[_s32]", "MMPd", "MPd", "i",  "aarch64_sve_scvtf", [IsOverloadFirstandLast]>;
defm SVFCVTZS_F32_S64 : SInstCvtMXZ<"svcvt_f32[_s64]", "MMPd", "MPd", "l",  "aarch64_sve_scvtf_f32i64">;

// svcvt_f64_s##
defm SVFCVTZS_F64_S32 : SInstCvtMXZ<"svcvt_f64[_s32]", "ddPt", "dPt", "d",  "aarch64_sve_scvtf_f64i32">;
defm SVFCVTZS_F64_S64 : SInstCvtMXZ<"svcvt_f64[_s64]", "NNPd", "NPd", "l",  "aarch64_sve_scvtf", [IsOverloadFirstandLast]>;

// svcvt_f16_u##
defm SVFCVTZU_F16_U16 : SInstCvtMXZ<"svcvt_f16[_u16]", "OOPd", "OPd", "Us", "aarch64_sve_ucvtf", [IsOverloadFirstandLast]>;
defm SVFCVTZU_F16_U32 : SInstCvtMXZ<"svcvt_f16[_u32]", "OOPd", "OPd", "Ui", "aarch64_sve_ucvtf_f16i32">;
defm SVFCVTZU_F16_U64 : SInstCvtMXZ<"svcvt_f16[_u64]", "OOPd", "OPd", "Ul", "aarch64_sve_ucvtf_f16i64">;

// svcvt_f32_u##
defm SVFCVTZU_F32_U32 : SInstCvtMXZ<"svcvt_f32[_u32]", "MMPd", "MPd", "Ui", "aarch64_sve_ucvtf", [IsOverloadFirstandLast]>;
defm SVFCVTZU_F32_U64 : SInstCvtMXZ<"svcvt_f32[_u64]", "MMPd", "MPd", "Ul", "aarch64_sve_ucvtf_f32i64">;
````
- **L925 EN**: Declares TableGen defm record `SVFCVTZU_U32_F32`.
  **L925 CN**: 声明 TableGen defm 记录 `SVFCVTZU_U32_F32`。
- **L926 EN**: Declares TableGen defm record `SVFCVTZU_U64_F32`.
  **L926 CN**: 声明 TableGen defm 记录 `SVFCVTZU_U64_F32`。
- **L927 EN**: Blank line separating nearby declarations or logic blocks.
  **L927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L928 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_u##_f64`.
  **L928 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_u##_f64`。
- **L929 EN**: Declares TableGen defm record `SVFCVTZU_U32_F64`.
  **L929 CN**: 声明 TableGen defm 记录 `SVFCVTZU_U32_F64`。
- **L930 EN**: Declares TableGen defm record `SVFCVTZU_U64_F64`.
  **L930 CN**: 声明 TableGen defm 记录 `SVFCVTZU_U64_F64`。
- **L931 EN**: Blank line separating nearby declarations or logic blocks.
  **L931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L932 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f16_s##`.
  **L932 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f16_s##`。
- **L933 EN**: Declares TableGen defm record `SVFCVTZS_F16_S16`.
  **L933 CN**: 声明 TableGen defm 记录 `SVFCVTZS_F16_S16`。
- **L934 EN**: Declares TableGen defm record `SVFCVTZS_F16_S32`.
  **L934 CN**: 声明 TableGen defm 记录 `SVFCVTZS_F16_S32`。
- **L935 EN**: Declares TableGen defm record `SVFCVTZS_F16_S64`.
  **L935 CN**: 声明 TableGen defm 记录 `SVFCVTZS_F16_S64`。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L937 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f32_s##`.
  **L937 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f32_s##`。
- **L938 EN**: Declares TableGen defm record `SVFCVTZS_F32_S32`.
  **L938 CN**: 声明 TableGen defm 记录 `SVFCVTZS_F32_S32`。
- **L939 EN**: Declares TableGen defm record `SVFCVTZS_F32_S64`.
  **L939 CN**: 声明 TableGen defm 记录 `SVFCVTZS_F32_S64`。
- **L940 EN**: Blank line separating nearby declarations or logic blocks.
  **L940 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L941 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f64_s##`.
  **L941 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f64_s##`。
- **L942 EN**: Declares TableGen defm record `SVFCVTZS_F64_S32`.
  **L942 CN**: 声明 TableGen defm 记录 `SVFCVTZS_F64_S32`。
- **L943 EN**: Declares TableGen defm record `SVFCVTZS_F64_S64`.
  **L943 CN**: 声明 TableGen defm 记录 `SVFCVTZS_F64_S64`。
- **L944 EN**: Blank line separating nearby declarations or logic blocks.
  **L944 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L945 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f16_u##`.
  **L945 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f16_u##`。
- **L946 EN**: Declares TableGen defm record `SVFCVTZU_F16_U16`.
  **L946 CN**: 声明 TableGen defm 记录 `SVFCVTZU_F16_U16`。
- **L947 EN**: Declares TableGen defm record `SVFCVTZU_F16_U32`.
  **L947 CN**: 声明 TableGen defm 记录 `SVFCVTZU_F16_U32`。
- **L948 EN**: Declares TableGen defm record `SVFCVTZU_F16_U64`.
  **L948 CN**: 声明 TableGen defm 记录 `SVFCVTZU_F16_U64`。
- **L949 EN**: Blank line separating nearby declarations or logic blocks.
  **L949 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L950 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f32_u##`.
  **L950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f32_u##`。
- **L951 EN**: Declares TableGen defm record `SVFCVTZU_F32_U32`.
  **L951 CN**: 声明 TableGen defm 记录 `SVFCVTZU_F32_U32`。
- **L952 EN**: Declares TableGen defm record `SVFCVTZU_F32_U64`.
  **L952 CN**: 声明 TableGen defm 记录 `SVFCVTZU_F32_U64`。

### Lines 953-980

````tablegen

// svcvt_f64_u##
defm SVFCVTZU_F64_U32 : SInstCvtMXZ<"svcvt_f64[_u32]", "ddPz", "dPz", "d",  "aarch64_sve_ucvtf_f64i32">;
defm SVFCVTZU_F64_U64 : SInstCvtMXZ<"svcvt_f64[_u64]", "NNPd", "NPd", "Ul", "aarch64_sve_ucvtf", [IsOverloadFirstandLast]>;

// svcvt_f16_f##
defm SVFCVT_F16_F32   : SInstCvtMXZ<"svcvt_f16[_f32]", "OOPd", "OPd", "f", "aarch64_sve_fcvt_f16f32">;
defm SVFCVT_F16_F64   : SInstCvtMXZ<"svcvt_f16[_f64]", "OOPd", "OPd", "d", "aarch64_sve_fcvt_f16f64">;

// svcvt_f32_f##
defm SVFCVT_F32_F16   : SInstCvtMXZ<"svcvt_f32[_f16]", "ddPO", "dPO", "f", "aarch64_sve_fcvt_f32f16">;
defm SVFCVT_F32_F64   : SInstCvtMXZ<"svcvt_f32[_f64]", "MMPd", "MPd", "d", "aarch64_sve_fcvt_f32f64">;

// svcvt_f64_f##
defm SVFCVT_F64_F16   : SInstCvtMXZ<"svcvt_f64[_f16]", "ddPO", "dPO", "d", "aarch64_sve_fcvt_f64f16">;
defm SVFCVT_F64_F32   : SInstCvtMXZ<"svcvt_f64[_f32]", "ddPM", "dPM", "d", "aarch64_sve_fcvt_f64f32">;

let SVETargetGuard = "bf16", SMETargetGuard = "bf16" in {
defm SVCVT_BF16_F32    : SInstCvtMXZ<"svcvt_bf16[_f32]", "$$Pd", "$Pd", "f", "aarch64_sve_fcvt_bf16f32_v2">;

def SVCVTNT_BF16_F32   : SInst<"svcvtnt_bf16[_f32]", "$$Pd", "f", MergeOp1, "aarch64_sve_fcvtnt_bf16f32_v2", [IsOverloadNone, VerifyRuntimeMode]>;
//  SVCVTNT_X_BF16_F32 : Implemented as macro by SveEmitter.cpp
}

let SVETargetGuard = "sve2|sme" in {
defm SVCVTLT_F32_F16 : SInstCvtMX<"svcvtlt_f32[_f16]",  "ddPh", "dPh", "f", "aarch64_sve_fcvtlt_f32f16">;
defm SVCVTLT_F64_F32 : SInstCvtMX<"svcvtlt_f64[_f32]",  "ddPh", "dPh", "d", "aarch64_sve_fcvtlt_f64f32">;

````
- **L953 EN**: Blank line separating nearby declarations or logic blocks.
  **L953 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L954 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f64_u##`.
  **L954 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f64_u##`。
- **L955 EN**: Declares TableGen defm record `SVFCVTZU_F64_U32`.
  **L955 CN**: 声明 TableGen defm 记录 `SVFCVTZU_F64_U32`。
- **L956 EN**: Declares TableGen defm record `SVFCVTZU_F64_U64`.
  **L956 CN**: 声明 TableGen defm 记录 `SVFCVTZU_F64_U64`。
- **L957 EN**: Blank line separating nearby declarations or logic blocks.
  **L957 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L958 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f16_f##`.
  **L958 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f16_f##`。
- **L959 EN**: Declares TableGen defm record `SVFCVT_F16_F32`.
  **L959 CN**: 声明 TableGen defm 记录 `SVFCVT_F16_F32`。
- **L960 EN**: Declares TableGen defm record `SVFCVT_F16_F64`.
  **L960 CN**: 声明 TableGen defm 记录 `SVFCVT_F16_F64`。
- **L961 EN**: Blank line separating nearby declarations or logic blocks.
  **L961 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L962 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f32_f##`.
  **L962 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f32_f##`。
- **L963 EN**: Declares TableGen defm record `SVFCVT_F32_F16`.
  **L963 CN**: 声明 TableGen defm 记录 `SVFCVT_F32_F16`。
- **L964 EN**: Declares TableGen defm record `SVFCVT_F32_F64`.
  **L964 CN**: 声明 TableGen defm 记录 `SVFCVT_F32_F64`。
- **L965 EN**: Blank line separating nearby declarations or logic blocks.
  **L965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L966 EN**: Comment explains nearby logic, constraints, or intent: `svcvt_f64_f##`.
  **L966 CN**: 注释解释附近代码的逻辑、约束或设计意图：`svcvt_f64_f##`。
- **L967 EN**: Declares TableGen defm record `SVFCVT_F64_F16`.
  **L967 CN**: 声明 TableGen defm 记录 `SVFCVT_F64_F16`。
- **L968 EN**: Declares TableGen defm record `SVFCVT_F64_F32`.
  **L968 CN**: 声明 TableGen defm 记录 `SVFCVT_F64_F32`。
- **L969 EN**: Blank line separating nearby declarations or logic blocks.
  **L969 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L970 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "bf16", SMETargetGuard = "bf16" in {`.
  **L970 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "bf16", SMETargetGuard = "bf16" in {`。
- **L971 EN**: Declares TableGen defm record `SVCVT_BF16_F32`.
  **L971 CN**: 声明 TableGen defm 记录 `SVCVT_BF16_F32`。
- **L972 EN**: Blank line separating nearby declarations or logic blocks.
  **L972 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L973 EN**: Declares TableGen def record `SVCVTNT_BF16_F32`.
  **L973 CN**: 声明 TableGen def 记录 `SVCVTNT_BF16_F32`。
- **L974 EN**: Comment explains nearby logic, constraints, or intent: `SVCVTNT_X_BF16_F32 : Implemented as macro by SveEmitter.cpp`.
  **L974 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVCVTNT_X_BF16_F32 : Implemented as macro by SveEmitter.cpp`。
- **L975 EN**: Closes the current lexical scope or compound statement.
  **L975 CN**: 结束当前词法作用域或复合语句块。
- **L976 EN**: Blank line separating nearby declarations or logic blocks.
  **L976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L977 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L977 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L978 EN**: Declares TableGen defm record `SVCVTLT_F32_F16`.
  **L978 CN**: 声明 TableGen defm 记录 `SVCVTLT_F32_F16`。
- **L979 EN**: Declares TableGen defm record `SVCVTLT_F64_F32`.
  **L979 CN**: 声明 TableGen defm 记录 `SVCVTLT_F64_F32`。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 981-1008

````tablegen
defm SVCVTX_F32_F64  : SInstCvtMXZ<"svcvtx_f32[_f64]",  "MMPd", "MPd", "d", "aarch64_sve_fcvtx_f32f64">;

def SVCVTNT_F16_F32   : SInst<"svcvtnt_f16[_f32]",  "hhPd", "f", MergeOp1, "aarch64_sve_fcvtnt_f16f32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVCVTNT_F32_F64   : SInst<"svcvtnt_f32[_f64]",  "hhPd", "d", MergeOp1, "aarch64_sve_fcvtnt_f32f64", [IsOverloadNone, VerifyRuntimeMode]>;
//  SVCVTNT_X_F16_F32 : Implemented as macro by SveEmitter.cpp
//  SVCVTNT_X_F32_F64 : Implemented as macro by SveEmitter.cpp

def SVCVTXNT_F32_F64   : SInst<"svcvtxnt_f32[_f64]", "MMPd", "d", MergeOp1, "aarch64_sve_fcvtxnt_f32f64", [IsOverloadNone, VerifyRuntimeMode]>;
//  SVCVTXNT_X_F32_F64 : Implemented as macro by SveEmitter.cpp
}

let SVETargetGuard = "sve2p2|sme2p2", SMETargetGuard = "sve2p2|sme2p2" in {

def SVCVTNT_Z_F16_F32  : SInst<"svcvtnt_f16[_f32]_z",  "hhPd", "f", MergeNone, "aarch64_sve_fcvtnt_z_f16f32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVCVTNT_Z_F32_F64  : SInst<"svcvtnt_f32[_f64]_z",  "hhPd", "d", MergeNone, "aarch64_sve_fcvtnt_z_f32f64", [IsOverloadNone, VerifyRuntimeMode]>;
def SVCVTNT_Z_BF16_F32 : SInst<"svcvtnt_bf16[_f32]_z", "$$Pd", "f", MergeNone, "aarch64_sve_fcvtnt_z_bf16f32", [IsOverloadNone, VerifyRuntimeMode]>;

def SVCVTXNT_Z_F32_F64 : SInst<"svcvtxnt_f32[_f64]_z", "MMPd", "d", MergeNone, "aarch64_sve_fcvtxnt_z_f32f64", [IsOverloadNone, VerifyRuntimeMode]>;

def SVCVTLT_Z_F32_F16  : SInst<"svcvtlt_f32[_f16]", "dPh", "f", MergeZeroExp, "aarch64_sve_fcvtlt_f32f16", [IsOverloadNone, VerifyRuntimeMode]>;
def SVCVTLT_Z_F64_F32  : SInst<"svcvtlt_f64[_f32]", "dPh", "d", MergeZeroExp, "aarch64_sve_fcvtlt_f64f32",  [IsOverloadNone, VerifyRuntimeMode]>;

}
////////////////////////////////////////////////////////////////////////////////
// Permutations and selection

multiclass SVEPerm<string name, string proto, string i> {
  def : SInst<name, proto, "csilUcUsUiUlhfdb", MergeNone, i, [VerifyRuntimeMode]>;
````
- **L981 EN**: Declares TableGen defm record `SVCVTX_F32_F64`.
  **L981 CN**: 声明 TableGen defm 记录 `SVCVTX_F32_F64`。
- **L982 EN**: Blank line separating nearby declarations or logic blocks.
  **L982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L983 EN**: Declares TableGen def record `SVCVTNT_F16_F32`.
  **L983 CN**: 声明 TableGen def 记录 `SVCVTNT_F16_F32`。
- **L984 EN**: Declares TableGen def record `SVCVTNT_F32_F64`.
  **L984 CN**: 声明 TableGen def 记录 `SVCVTNT_F32_F64`。
- **L985 EN**: Comment explains nearby logic, constraints, or intent: `SVCVTNT_X_F16_F32 : Implemented as macro by SveEmitter.cpp`.
  **L985 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVCVTNT_X_F16_F32 : Implemented as macro by SveEmitter.cpp`。
- **L986 EN**: Comment explains nearby logic, constraints, or intent: `SVCVTNT_X_F32_F64 : Implemented as macro by SveEmitter.cpp`.
  **L986 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVCVTNT_X_F32_F64 : Implemented as macro by SveEmitter.cpp`。
- **L987 EN**: Blank line separating nearby declarations or logic blocks.
  **L987 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L988 EN**: Declares TableGen def record `SVCVTXNT_F32_F64`.
  **L988 CN**: 声明 TableGen def 记录 `SVCVTXNT_F32_F64`。
- **L989 EN**: Comment explains nearby logic, constraints, or intent: `SVCVTXNT_X_F32_F64 : Implemented as macro by SveEmitter.cpp`.
  **L989 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVCVTXNT_X_F32_F64 : Implemented as macro by SveEmitter.cpp`。
- **L990 EN**: Closes the current lexical scope or compound statement.
  **L990 CN**: 结束当前词法作用域或复合语句块。
- **L991 EN**: Blank line separating nearby declarations or logic blocks.
  **L991 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L992 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p2|sme2p2", SMETargetGuard = "sve2p2|sme2p2" in {`.
  **L992 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p2|sme2p2", SMETargetGuard = "sve2p2|sme2p2" in {`。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L994 EN**: Declares TableGen def record `SVCVTNT_Z_F16_F32`.
  **L994 CN**: 声明 TableGen def 记录 `SVCVTNT_Z_F16_F32`。
- **L995 EN**: Declares TableGen def record `SVCVTNT_Z_F32_F64`.
  **L995 CN**: 声明 TableGen def 记录 `SVCVTNT_Z_F32_F64`。
- **L996 EN**: Declares TableGen def record `SVCVTNT_Z_BF16_F32`.
  **L996 CN**: 声明 TableGen def 记录 `SVCVTNT_Z_BF16_F32`。
- **L997 EN**: Blank line separating nearby declarations or logic blocks.
  **L997 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L998 EN**: Declares TableGen def record `SVCVTXNT_Z_F32_F64`.
  **L998 CN**: 声明 TableGen def 记录 `SVCVTXNT_Z_F32_F64`。
- **L999 EN**: Blank line separating nearby declarations or logic blocks.
  **L999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1000 EN**: Declares TableGen def record `SVCVTLT_Z_F32_F16`.
  **L1000 CN**: 声明 TableGen def 记录 `SVCVTLT_Z_F32_F16`。
- **L1001 EN**: Declares TableGen def record `SVCVTLT_Z_F64_F32`.
  **L1001 CN**: 声明 TableGen def 记录 `SVCVTLT_Z_F64_F32`。
- **L1002 EN**: Blank line separating nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1003 EN**: Closes the current lexical scope or compound statement.
  **L1003 CN**: 结束当前词法作用域或复合语句块。
- **L1004 EN**: Separator comment used for visual grouping.
  **L1004 CN**: 用于视觉分组的分隔注释。
- **L1005 EN**: Comment explains nearby logic, constraints, or intent: `Permutations and selection`.
  **L1005 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Permutations and selection`。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Declares TableGen multiclass record `SVEPerm`.
  **L1007 CN**: 声明 TableGen multiclass 记录 `SVEPerm`。
- **L1008 EN**: Declares TableGen def record `def`.
  **L1008 CN**: 声明 TableGen def 记录 `def`。

### Lines 1009-1036

````tablegen
}

defm SVCLASTA   : SVEPerm<"svclasta[_{d}]",   "dPdd", "aarch64_sve_clasta">;
defm SVCLASTA_N : SVEPerm<"svclasta[_n_{d}]", "sPsd", "aarch64_sve_clasta_n">;
defm SVCLASTB   : SVEPerm<"svclastb[_{d}]",   "dPdd", "aarch64_sve_clastb">;
defm SVCLASTB_N : SVEPerm<"svclastb[_n_{d}]", "sPsd", "aarch64_sve_clastb_n">;

let SMETargetGuard = "sme2p2" in {
def SVCOMPACT : SInst<"svcompact[_{d}]", "dPd",  "ilUiUlfd", MergeNone, "aarch64_sve_compact", [VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve2p2|sme2p2",  SMETargetGuard = "sme2p2" in {
def SVCOMPACT_BH : SInst<"svcompact[_{d}]", "dPd",  "cUcsUsmbh", MergeNone, "aarch64_sve_compact", [VerifyRuntimeMode]>;
def SVEXPAND  : SInst<"svexpand[_{d}]",  "dPd",  "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_expand",  [VerifyRuntimeMode]>;
}

// Note: svdup_lane is implemented using the intrinsic for TBL to represent a
// splat of any possible lane. It is upto LLVM to pick a more efficient
// instruction such as DUP (indexed) if the lane index fits the range of the
// instruction's immediate.
def SVDUP_LANE  : SInst<"svdup_lane[_{d}]",  "ddL",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_tbl", [VerifyRuntimeMode]>;
def SVDUPQ_LANE : SInst<"svdupq_lane[_{d}]", "ddn",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_dupq_lane", [VerifyRuntimeMode]>;
def SVEXT       : SInst<"svext[_{d}]",       "dddi", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_ext", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckExtract, 1>]>;
defm SVLASTA    : SVEPerm<"svlasta[_{d}]",   "sPd",  "aarch64_sve_lasta">;
defm SVLASTB    : SVEPerm<"svlastb[_{d}]",   "sPd",  "aarch64_sve_lastb">;
def SVREV       : SInst<"svrev[_{d}]",       "dd",   "csilUcUsUiUlhfdb", MergeNone, "vector_reverse", [VerifyRuntimeMode]>;
def SVSEL       : SInst<"svsel[_{d}]",       "dPdd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_sel", [VerifyRuntimeMode]>;
def SVSPLICE    : SInst<"svsplice[_{d}]",    "dPdd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_splice", [VerifyRuntimeMode]>;
````
- **L1009 EN**: Closes the current lexical scope or compound statement.
  **L1009 CN**: 结束当前词法作用域或复合语句块。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1011 EN**: Declares TableGen defm record `SVCLASTA`.
  **L1011 CN**: 声明 TableGen defm 记录 `SVCLASTA`。
- **L1012 EN**: Declares TableGen defm record `SVCLASTA_N`.
  **L1012 CN**: 声明 TableGen defm 记录 `SVCLASTA_N`。
- **L1013 EN**: Declares TableGen defm record `SVCLASTB`.
  **L1013 CN**: 声明 TableGen defm 记录 `SVCLASTB`。
- **L1014 EN**: Declares TableGen defm record `SVCLASTB_N`.
  **L1014 CN**: 声明 TableGen defm 记录 `SVCLASTB_N`。
- **L1015 EN**: Blank line separating nearby declarations or logic blocks.
  **L1015 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1016 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2p2" in {`.
  **L1016 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2p2" in {`。
- **L1017 EN**: Declares TableGen def record `SVCOMPACT`.
  **L1017 CN**: 声明 TableGen def 记录 `SVCOMPACT`。
- **L1018 EN**: Closes the current lexical scope or compound statement.
  **L1018 CN**: 结束当前词法作用域或复合语句块。
- **L1019 EN**: Blank line separating nearby declarations or logic blocks.
  **L1019 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1020 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p2|sme2p2",  SMETargetGuard = "sme2p2" in {`.
  **L1020 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p2|sme2p2",  SMETargetGuard = "sme2p2" in {`。
- **L1021 EN**: Declares TableGen def record `SVCOMPACT_BH`.
  **L1021 CN**: 声明 TableGen def 记录 `SVCOMPACT_BH`。
- **L1022 EN**: Declares TableGen def record `SVEXPAND`.
  **L1022 CN**: 声明 TableGen def 记录 `SVEXPAND`。
- **L1023 EN**: Closes the current lexical scope or compound statement.
  **L1023 CN**: 结束当前词法作用域或复合语句块。
- **L1024 EN**: Blank line separating nearby declarations or logic blocks.
  **L1024 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1025 EN**: Comment highlights an implementation note: `Note: svdup_lane is implemented using the intrinsic for TBL to represent a`.
  **L1025 CN**: 注释强调一条实现说明：`Note: svdup_lane is implemented using the intrinsic for TBL to represent a`。
- **L1026 EN**: Comment explains nearby logic, constraints, or intent: `splat of any possible lane. It is upto LLVM to pick a more efficient`.
  **L1026 CN**: 注释解释附近代码的逻辑、约束或设计意图：`splat of any possible lane. It is upto LLVM to pick a more efficient`。
- **L1027 EN**: Comment explains nearby logic, constraints, or intent: `instruction such as DUP (indexed) if the lane index fits the range of the`.
  **L1027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction such as DUP (indexed) if the lane index fits the range of the`。
- **L1028 EN**: Comment explains nearby logic, constraints, or intent: `instruction's immediate.`.
  **L1028 CN**: 注释解释附近代码的逻辑、约束或设计意图：`instruction's immediate.`。
- **L1029 EN**: Declares TableGen def record `SVDUP_LANE`.
  **L1029 CN**: 声明 TableGen def 记录 `SVDUP_LANE`。
- **L1030 EN**: Declares TableGen def record `SVDUPQ_LANE`.
  **L1030 CN**: 声明 TableGen def 记录 `SVDUPQ_LANE`。
- **L1031 EN**: Declares TableGen def record `SVEXT`.
  **L1031 CN**: 声明 TableGen def 记录 `SVEXT`。
- **L1032 EN**: Declares TableGen defm record `SVLASTA`.
  **L1032 CN**: 声明 TableGen defm 记录 `SVLASTA`。
- **L1033 EN**: Declares TableGen defm record `SVLASTB`.
  **L1033 CN**: 声明 TableGen defm 记录 `SVLASTB`。
- **L1034 EN**: Declares TableGen def record `SVREV`.
  **L1034 CN**: 声明 TableGen def 记录 `SVREV`。
- **L1035 EN**: Declares TableGen def record `SVSEL`.
  **L1035 CN**: 声明 TableGen def 记录 `SVSEL`。
- **L1036 EN**: Declares TableGen def record `SVSPLICE`.
  **L1036 CN**: 声明 TableGen def 记录 `SVSPLICE`。

### Lines 1037-1064

````tablegen
def SVTBL       : SInst<"svtbl[_{d}]",       "ddu",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_tbl", [VerifyRuntimeMode]>;

def SVTRN1     : SInst<"svtrn1[_{d}]",      "ddd",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_trn1", [VerifyRuntimeMode]>;
def SVTRN2     : SInst<"svtrn2[_{d}]",      "ddd",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_trn2", [VerifyRuntimeMode]>;
def SVUNPKHI_S : SInst<"svunpkhi[_{d}]",    "dh",   "sil",              MergeNone, "aarch64_sve_sunpkhi", [VerifyRuntimeMode]>;
def SVUNPKHI_U : SInst<"svunpkhi[_{d}]",    "dh",   "UsUiUl",           MergeNone, "aarch64_sve_uunpkhi", [VerifyRuntimeMode]>;
def SVUNPKLO_S : SInst<"svunpklo[_{d}]",    "dh",   "sil",              MergeNone, "aarch64_sve_sunpklo", [VerifyRuntimeMode]>;
def SVUNPKLO_U : SInst<"svunpklo[_{d}]",    "dh",   "UsUiUl",           MergeNone, "aarch64_sve_uunpklo", [VerifyRuntimeMode]>;
def SVUZP1     : SInst<"svuzp1[_{d}]",      "ddd",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_uzp1", [VerifyRuntimeMode]>;
def SVUZP2     : SInst<"svuzp2[_{d}]",      "ddd",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_uzp2", [VerifyRuntimeMode]>;
def SVZIP1     : SInst<"svzip1[_{d}]",      "ddd",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_zip1", [VerifyRuntimeMode]>;
def SVZIP2     : SInst<"svzip2[_{d}]",      "ddd",  "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_zip2", [VerifyRuntimeMode]>;

def SVREV_B8   : SInst<"svrev_b8",     "PP",   "Pc", MergeNone, "vector_reverse", [VerifyRuntimeMode]>;
def SVREV_B16  : SInst<"svrev_b16",    "PP",   "Pc", MergeNone, "aarch64_sve_rev_b16",  [IsOverloadNone, VerifyRuntimeMode]>;
def SVREV_B32  : SInst<"svrev_b32",    "PP",   "Pc", MergeNone, "aarch64_sve_rev_b32",  [IsOverloadNone, VerifyRuntimeMode]>;
def SVREV_B64  : SInst<"svrev_b64",    "PP",   "Pc", MergeNone, "aarch64_sve_rev_b64",  [IsOverloadNone, VerifyRuntimeMode]>;
def SVSEL_B    : SInst<"svsel[_b]",    "PPPP", "Pc", MergeNone, "aarch64_sve_sel", [VerifyRuntimeMode]>;
def SVTRN1_B8  : SInst<"svtrn1_b8",    "PPP",  "Pc", MergeNone, "aarch64_sve_trn1", [VerifyRuntimeMode]>;
def SVTRN1_B16 : SInst<"svtrn1_b16",   "PPP",  "Pc", MergeNone, "aarch64_sve_trn1_b16", [IsOverloadNone, VerifyRuntimeMode]>;
def SVTRN1_B32 : SInst<"svtrn1_b32",   "PPP",  "Pc", MergeNone, "aarch64_sve_trn1_b32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVTRN1_B64 : SInst<"svtrn1_b64",   "PPP",  "Pc", MergeNone, "aarch64_sve_trn1_b64", [IsOverloadNone, VerifyRuntimeMode]>;
def SVTRN2_B8  : SInst<"svtrn2_b8",    "PPP",  "Pc", MergeNone, "aarch64_sve_trn2", [VerifyRuntimeMode]>;
def SVTRN2_B16 : SInst<"svtrn2_b16",   "PPP",  "Pc", MergeNone, "aarch64_sve_trn2_b16", [IsOverloadNone, VerifyRuntimeMode]>;
def SVTRN2_B32 : SInst<"svtrn2_b32",   "PPP",  "Pc", MergeNone, "aarch64_sve_trn2_b32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVTRN2_B64 : SInst<"svtrn2_b64",   "PPP",  "Pc", MergeNone, "aarch64_sve_trn2_b64", [IsOverloadNone, VerifyRuntimeMode]>;
def SVPUNPKHI  : SInst<"svunpkhi[_b]", "PP",   "Pc", MergeNone, "aarch64_sve_punpkhi", [VerifyRuntimeMode]>;
def SVPUNPKLO  : SInst<"svunpklo[_b]", "PP",   "Pc", MergeNone, "aarch64_sve_punpklo", [VerifyRuntimeMode]>;
````
- **L1037 EN**: Declares TableGen def record `SVTBL`.
  **L1037 CN**: 声明 TableGen def 记录 `SVTBL`。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1039 EN**: Declares TableGen def record `SVTRN1`.
  **L1039 CN**: 声明 TableGen def 记录 `SVTRN1`。
- **L1040 EN**: Declares TableGen def record `SVTRN2`.
  **L1040 CN**: 声明 TableGen def 记录 `SVTRN2`。
- **L1041 EN**: Declares TableGen def record `SVUNPKHI_S`.
  **L1041 CN**: 声明 TableGen def 记录 `SVUNPKHI_S`。
- **L1042 EN**: Declares TableGen def record `SVUNPKHI_U`.
  **L1042 CN**: 声明 TableGen def 记录 `SVUNPKHI_U`。
- **L1043 EN**: Declares TableGen def record `SVUNPKLO_S`.
  **L1043 CN**: 声明 TableGen def 记录 `SVUNPKLO_S`。
- **L1044 EN**: Declares TableGen def record `SVUNPKLO_U`.
  **L1044 CN**: 声明 TableGen def 记录 `SVUNPKLO_U`。
- **L1045 EN**: Declares TableGen def record `SVUZP1`.
  **L1045 CN**: 声明 TableGen def 记录 `SVUZP1`。
- **L1046 EN**: Declares TableGen def record `SVUZP2`.
  **L1046 CN**: 声明 TableGen def 记录 `SVUZP2`。
- **L1047 EN**: Declares TableGen def record `SVZIP1`.
  **L1047 CN**: 声明 TableGen def 记录 `SVZIP1`。
- **L1048 EN**: Declares TableGen def record `SVZIP2`.
  **L1048 CN**: 声明 TableGen def 记录 `SVZIP2`。
- **L1049 EN**: Blank line separating nearby declarations or logic blocks.
  **L1049 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1050 EN**: Declares TableGen def record `SVREV_B8`.
  **L1050 CN**: 声明 TableGen def 记录 `SVREV_B8`。
- **L1051 EN**: Declares TableGen def record `SVREV_B16`.
  **L1051 CN**: 声明 TableGen def 记录 `SVREV_B16`。
- **L1052 EN**: Declares TableGen def record `SVREV_B32`.
  **L1052 CN**: 声明 TableGen def 记录 `SVREV_B32`。
- **L1053 EN**: Declares TableGen def record `SVREV_B64`.
  **L1053 CN**: 声明 TableGen def 记录 `SVREV_B64`。
- **L1054 EN**: Declares TableGen def record `SVSEL_B`.
  **L1054 CN**: 声明 TableGen def 记录 `SVSEL_B`。
- **L1055 EN**: Declares TableGen def record `SVTRN1_B8`.
  **L1055 CN**: 声明 TableGen def 记录 `SVTRN1_B8`。
- **L1056 EN**: Declares TableGen def record `SVTRN1_B16`.
  **L1056 CN**: 声明 TableGen def 记录 `SVTRN1_B16`。
- **L1057 EN**: Declares TableGen def record `SVTRN1_B32`.
  **L1057 CN**: 声明 TableGen def 记录 `SVTRN1_B32`。
- **L1058 EN**: Declares TableGen def record `SVTRN1_B64`.
  **L1058 CN**: 声明 TableGen def 记录 `SVTRN1_B64`。
- **L1059 EN**: Declares TableGen def record `SVTRN2_B8`.
  **L1059 CN**: 声明 TableGen def 记录 `SVTRN2_B8`。
- **L1060 EN**: Declares TableGen def record `SVTRN2_B16`.
  **L1060 CN**: 声明 TableGen def 记录 `SVTRN2_B16`。
- **L1061 EN**: Declares TableGen def record `SVTRN2_B32`.
  **L1061 CN**: 声明 TableGen def 记录 `SVTRN2_B32`。
- **L1062 EN**: Declares TableGen def record `SVTRN2_B64`.
  **L1062 CN**: 声明 TableGen def 记录 `SVTRN2_B64`。
- **L1063 EN**: Declares TableGen def record `SVPUNPKHI`.
  **L1063 CN**: 声明 TableGen def 记录 `SVPUNPKHI`。
- **L1064 EN**: Declares TableGen def record `SVPUNPKLO`.
  **L1064 CN**: 声明 TableGen def 记录 `SVPUNPKLO`。

### Lines 1065-1092

````tablegen
def SVUZP1_B8  : SInst<"svuzp1_b8",    "PPP",  "Pc", MergeNone, "aarch64_sve_uzp1", [VerifyRuntimeMode]>;
def SVUZP1_B16 : SInst<"svuzp1_b16",   "PPP",  "Pc", MergeNone, "aarch64_sve_uzp1_b16", [IsOverloadNone, VerifyRuntimeMode]>;
def SVUZP1_B32 : SInst<"svuzp1_b32",   "PPP",  "Pc", MergeNone, "aarch64_sve_uzp1_b32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVUZP1_B64 : SInst<"svuzp1_b64",   "PPP",  "Pc", MergeNone, "aarch64_sve_uzp1_b64", [IsOverloadNone, VerifyRuntimeMode]>;
def SVUZP2_B8  : SInst<"svuzp2_b8",    "PPP",  "Pc", MergeNone, "aarch64_sve_uzp2", [VerifyRuntimeMode]>;
def SVUZP2_B16 : SInst<"svuzp2_b16",   "PPP",  "Pc", MergeNone, "aarch64_sve_uzp2_b16", [IsOverloadNone, VerifyRuntimeMode]>;
def SVUZP2_B32 : SInst<"svuzp2_b32",   "PPP",  "Pc", MergeNone, "aarch64_sve_uzp2_b32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVUZP2_B64 : SInst<"svuzp2_b64",   "PPP",  "Pc", MergeNone, "aarch64_sve_uzp2_b64", [IsOverloadNone, VerifyRuntimeMode]>;
def SVZIP1_B8  : SInst<"svzip1_b8",    "PPP",  "Pc", MergeNone, "aarch64_sve_zip1", [VerifyRuntimeMode]>;
def SVZIP1_B16 : SInst<"svzip1_b16",   "PPP",  "Pc", MergeNone, "aarch64_sve_zip1_b16", [IsOverloadNone, VerifyRuntimeMode]>;
def SVZIP1_B32 : SInst<"svzip1_b32",   "PPP",  "Pc", MergeNone, "aarch64_sve_zip1_b32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVZIP1_B64 : SInst<"svzip1_b64",   "PPP",  "Pc", MergeNone, "aarch64_sve_zip1_b64", [IsOverloadNone, VerifyRuntimeMode]>;
def SVZIP2_B   : SInst<"svzip2_b8",    "PPP",  "Pc", MergeNone, "aarch64_sve_zip2", [VerifyRuntimeMode]>;
def SVZIP2_B16 : SInst<"svzip2_b16",   "PPP",  "Pc", MergeNone, "aarch64_sve_zip2_b16", [IsOverloadNone, VerifyRuntimeMode]>;
def SVZIP2_B32 : SInst<"svzip2_b32",   "PPP",  "Pc", MergeNone, "aarch64_sve_zip2_b32", [IsOverloadNone, VerifyRuntimeMode]>;
def SVZIP2_B64 : SInst<"svzip2_b64",   "PPP",  "Pc", MergeNone, "aarch64_sve_zip2_b64", [IsOverloadNone, VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Predicate creation

def SVPFALSE : SInst<"svpfalse[_b]", "Pv", "", MergeNone, "", [IsOverloadNone, VerifyRuntimeMode]>;
def SVPTRUE_PAT : SInst<"svptrue_pat_{d}", "PI", "PcPsPiPl", MergeNone, "aarch64_sve_ptrue", [VerifyRuntimeMode]>;
def SVPTRUE     : SInst<"svptrue_{d}",     "Pv", "PcPsPiPl", MergeNone, "aarch64_sve_ptrue", [IsAppendSVALL, VerifyRuntimeMode]>;

def SVDUPQ_B8  : SInst<"svdupq[_n]_{d}", "Pssssssssssssssss", "Pc", MergeNone, "", [VerifyRuntimeMode]>;
def SVDUPQ_B16 : SInst<"svdupq[_n]_{d}", "Pssssssss", "Ps", MergeNone, "", [VerifyRuntimeMode]>;
def SVDUPQ_B32 : SInst<"svdupq[_n]_{d}", "Pssss", "Pi", MergeNone, "", [VerifyRuntimeMode]>;
def SVDUPQ_B64 : SInst<"svdupq[_n]_{d}", "Pss", "Pl", MergeNone, "", [VerifyRuntimeMode]>;
````
- **L1065 EN**: Declares TableGen def record `SVUZP1_B8`.
  **L1065 CN**: 声明 TableGen def 记录 `SVUZP1_B8`。
- **L1066 EN**: Declares TableGen def record `SVUZP1_B16`.
  **L1066 CN**: 声明 TableGen def 记录 `SVUZP1_B16`。
- **L1067 EN**: Declares TableGen def record `SVUZP1_B32`.
  **L1067 CN**: 声明 TableGen def 记录 `SVUZP1_B32`。
- **L1068 EN**: Declares TableGen def record `SVUZP1_B64`.
  **L1068 CN**: 声明 TableGen def 记录 `SVUZP1_B64`。
- **L1069 EN**: Declares TableGen def record `SVUZP2_B8`.
  **L1069 CN**: 声明 TableGen def 记录 `SVUZP2_B8`。
- **L1070 EN**: Declares TableGen def record `SVUZP2_B16`.
  **L1070 CN**: 声明 TableGen def 记录 `SVUZP2_B16`。
- **L1071 EN**: Declares TableGen def record `SVUZP2_B32`.
  **L1071 CN**: 声明 TableGen def 记录 `SVUZP2_B32`。
- **L1072 EN**: Declares TableGen def record `SVUZP2_B64`.
  **L1072 CN**: 声明 TableGen def 记录 `SVUZP2_B64`。
- **L1073 EN**: Declares TableGen def record `SVZIP1_B8`.
  **L1073 CN**: 声明 TableGen def 记录 `SVZIP1_B8`。
- **L1074 EN**: Declares TableGen def record `SVZIP1_B16`.
  **L1074 CN**: 声明 TableGen def 记录 `SVZIP1_B16`。
- **L1075 EN**: Declares TableGen def record `SVZIP1_B32`.
  **L1075 CN**: 声明 TableGen def 记录 `SVZIP1_B32`。
- **L1076 EN**: Declares TableGen def record `SVZIP1_B64`.
  **L1076 CN**: 声明 TableGen def 记录 `SVZIP1_B64`。
- **L1077 EN**: Declares TableGen def record `SVZIP2_B`.
  **L1077 CN**: 声明 TableGen def 记录 `SVZIP2_B`。
- **L1078 EN**: Declares TableGen def record `SVZIP2_B16`.
  **L1078 CN**: 声明 TableGen def 记录 `SVZIP2_B16`。
- **L1079 EN**: Declares TableGen def record `SVZIP2_B32`.
  **L1079 CN**: 声明 TableGen def 记录 `SVZIP2_B32`。
- **L1080 EN**: Declares TableGen def record `SVZIP2_B64`.
  **L1080 CN**: 声明 TableGen def 记录 `SVZIP2_B64`。
- **L1081 EN**: Blank line separating nearby declarations or logic blocks.
  **L1081 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1082 EN**: Separator comment used for visual grouping.
  **L1082 CN**: 用于视觉分组的分隔注释。
- **L1083 EN**: Comment explains nearby logic, constraints, or intent: `Predicate creation`.
  **L1083 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicate creation`。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1085 EN**: Declares TableGen def record `SVPFALSE`.
  **L1085 CN**: 声明 TableGen def 记录 `SVPFALSE`。
- **L1086 EN**: Declares TableGen def record `SVPTRUE_PAT`.
  **L1086 CN**: 声明 TableGen def 记录 `SVPTRUE_PAT`。
- **L1087 EN**: Declares TableGen def record `SVPTRUE`.
  **L1087 CN**: 声明 TableGen def 记录 `SVPTRUE`。
- **L1088 EN**: Blank line separating nearby declarations or logic blocks.
  **L1088 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1089 EN**: Declares TableGen def record `SVDUPQ_B8`.
  **L1089 CN**: 声明 TableGen def 记录 `SVDUPQ_B8`。
- **L1090 EN**: Declares TableGen def record `SVDUPQ_B16`.
  **L1090 CN**: 声明 TableGen def 记录 `SVDUPQ_B16`。
- **L1091 EN**: Declares TableGen def record `SVDUPQ_B32`.
  **L1091 CN**: 声明 TableGen def 记录 `SVDUPQ_B32`。
- **L1092 EN**: Declares TableGen def record `SVDUPQ_B64`.
  **L1092 CN**: 声明 TableGen def 记录 `SVDUPQ_B64`。

### Lines 1093-1120

````tablegen
def SVDUP_N_B  : SInst<"svdup[_n]_{d}",  "Ps", "PcPsPiPl", MergeNone, "", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Predicate operations

def SVAND_B_Z  : SInst<"svand[_b]_z",  "PPPP", "Pc", MergeNone, "aarch64_sve_and_z", [VerifyRuntimeMode]>;
def SVBIC_B_Z  : SInst<"svbic[_b]_z",  "PPPP", "Pc", MergeNone, "aarch64_sve_bic_z", [VerifyRuntimeMode]>;
def SVEOR_B_Z  : SInst<"sveor[_b]_z",  "PPPP", "Pc", MergeNone, "aarch64_sve_eor_z", [VerifyRuntimeMode]>;
def SVMOV_B_Z  : SInst<"svmov[_b]_z",  "PPP",  "Pc", MergeNone, "", [VerifyRuntimeMode]>; // Uses custom expansion
def SVNAND_B_Z : SInst<"svnand[_b]_z", "PPPP", "Pc", MergeNone, "aarch64_sve_nand_z", [VerifyRuntimeMode]>;
def SVNOR_B_Z  : SInst<"svnor[_b]_z",  "PPPP", "Pc", MergeNone, "aarch64_sve_nor_z", [VerifyRuntimeMode]>;
def SVNOT_B_Z  : SInst<"svnot[_b]_z",  "PPP",  "Pc", MergeNone, "", [VerifyRuntimeMode]>; // Uses custom expansion
def SVORN_B_Z  : SInst<"svorn[_b]_z",  "PPPP", "Pc", MergeNone, "aarch64_sve_orn_z", [VerifyRuntimeMode]>;
def SVORR_B_Z  : SInst<"svorr[_b]_z",  "PPPP", "Pc", MergeNone, "aarch64_sve_orr_z", [VerifyRuntimeMode]>;

def SVBRKA    : SInst<"svbrka[_b]_m",  "PPPP", "Pc", MergeNone, "aarch64_sve_brka", [VerifyRuntimeMode]>;
def SVBRKA_Z  : SInst<"svbrka[_b]_z",  "PPP",  "Pc", MergeNone, "aarch64_sve_brka_z", [VerifyRuntimeMode]>;
def SVBRKB    : SInst<"svbrkb[_b]_m",  "PPPP", "Pc", MergeNone, "aarch64_sve_brkb", [VerifyRuntimeMode]>;
def SVBRKB_Z  : SInst<"svbrkb[_b]_z",  "PPP",  "Pc", MergeNone, "aarch64_sve_brkb_z", [VerifyRuntimeMode]>;
def SVBRKN_Z  : SInst<"svbrkn[_b]_z",  "PPPP", "Pc", MergeNone, "aarch64_sve_brkn_z", [VerifyRuntimeMode]>;
def SVBRKPA_Z : SInst<"svbrkpa[_b]_z", "PPPP", "Pc", MergeNone, "aarch64_sve_brkpa_z", [VerifyRuntimeMode]>;
def SVBRKPB_Z : SInst<"svbrkpb[_b]_z", "PPPP", "Pc", MergeNone, "aarch64_sve_brkpb_z", [VerifyRuntimeMode]>;

def SVPFIRST : SInst<"svpfirst[_b]", "PPP", "Pc",       MergeNone, "aarch64_sve_pfirst", [VerifyRuntimeMode]>;
def SVPNEXT  : SInst<"svpnext_{d}",  "PPP", "PcPsPiPl", MergeNone, "aarch64_sve_pnext", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// Testing predicates
````
- **L1093 EN**: Declares TableGen def record `SVDUP_N_B`.
  **L1093 CN**: 声明 TableGen def 记录 `SVDUP_N_B`。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Separator comment used for visual grouping.
  **L1095 CN**: 用于视觉分组的分隔注释。
- **L1096 EN**: Comment explains nearby logic, constraints, or intent: `Predicate operations`.
  **L1096 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicate operations`。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1098 EN**: Declares TableGen def record `SVAND_B_Z`.
  **L1098 CN**: 声明 TableGen def 记录 `SVAND_B_Z`。
- **L1099 EN**: Declares TableGen def record `SVBIC_B_Z`.
  **L1099 CN**: 声明 TableGen def 记录 `SVBIC_B_Z`。
- **L1100 EN**: Declares TableGen def record `SVEOR_B_Z`.
  **L1100 CN**: 声明 TableGen def 记录 `SVEOR_B_Z`。
- **L1101 EN**: Declares TableGen def record `SVMOV_B_Z`.
  **L1101 CN**: 声明 TableGen def 记录 `SVMOV_B_Z`。
- **L1102 EN**: Declares TableGen def record `SVNAND_B_Z`.
  **L1102 CN**: 声明 TableGen def 记录 `SVNAND_B_Z`。
- **L1103 EN**: Declares TableGen def record `SVNOR_B_Z`.
  **L1103 CN**: 声明 TableGen def 记录 `SVNOR_B_Z`。
- **L1104 EN**: Declares TableGen def record `SVNOT_B_Z`.
  **L1104 CN**: 声明 TableGen def 记录 `SVNOT_B_Z`。
- **L1105 EN**: Declares TableGen def record `SVORN_B_Z`.
  **L1105 CN**: 声明 TableGen def 记录 `SVORN_B_Z`。
- **L1106 EN**: Declares TableGen def record `SVORR_B_Z`.
  **L1106 CN**: 声明 TableGen def 记录 `SVORR_B_Z`。
- **L1107 EN**: Blank line separating nearby declarations or logic blocks.
  **L1107 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1108 EN**: Declares TableGen def record `SVBRKA`.
  **L1108 CN**: 声明 TableGen def 记录 `SVBRKA`。
- **L1109 EN**: Declares TableGen def record `SVBRKA_Z`.
  **L1109 CN**: 声明 TableGen def 记录 `SVBRKA_Z`。
- **L1110 EN**: Declares TableGen def record `SVBRKB`.
  **L1110 CN**: 声明 TableGen def 记录 `SVBRKB`。
- **L1111 EN**: Declares TableGen def record `SVBRKB_Z`.
  **L1111 CN**: 声明 TableGen def 记录 `SVBRKB_Z`。
- **L1112 EN**: Declares TableGen def record `SVBRKN_Z`.
  **L1112 CN**: 声明 TableGen def 记录 `SVBRKN_Z`。
- **L1113 EN**: Declares TableGen def record `SVBRKPA_Z`.
  **L1113 CN**: 声明 TableGen def 记录 `SVBRKPA_Z`。
- **L1114 EN**: Declares TableGen def record `SVBRKPB_Z`.
  **L1114 CN**: 声明 TableGen def 记录 `SVBRKPB_Z`。
- **L1115 EN**: Blank line separating nearby declarations or logic blocks.
  **L1115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1116 EN**: Declares TableGen def record `SVPFIRST`.
  **L1116 CN**: 声明 TableGen def 记录 `SVPFIRST`。
- **L1117 EN**: Declares TableGen def record `SVPNEXT`.
  **L1117 CN**: 声明 TableGen def 记录 `SVPNEXT`。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1119 EN**: Separator comment used for visual grouping.
  **L1119 CN**: 用于视觉分组的分隔注释。
- **L1120 EN**: Comment explains nearby logic, constraints, or intent: `Testing predicates`.
  **L1120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Testing predicates`。

### Lines 1121-1148

````tablegen

def SVPTEST_ANY   : SInst<"svptest_any",   "sPP", "Pc", MergeNone, "aarch64_sve_ptest_any", [VerifyRuntimeMode]>;
def SVPTEST_FIRST : SInst<"svptest_first", "sPP", "Pc", MergeNone, "aarch64_sve_ptest_first", [VerifyRuntimeMode]>;
def SVPTEST_LAST  : SInst<"svptest_last",  "sPP", "Pc", MergeNone, "aarch64_sve_ptest_last", [VerifyRuntimeMode]>;

////////////////////////////////////////////////////////////////////////////////
// FFR manipulation

let SMETargetGuard = InvalidMode in {
def SVRDFFR   : SInst<"svrdffr",   "Pv", "Pc", MergeNone, "", [IsOverloadNone]>;
def SVRDFFR_Z : SInst<"svrdffr_z", "PP", "Pc", MergeNone, "", [IsOverloadNone]>;
def SVSETFFR  : SInst<"svsetffr",  "vv", "",   MergeNone, "", [IsOverloadNone]>;
def SVWRFFR   : SInst<"svwrffr",   "vP", "Pc", MergeNone, "", [IsOverloadNone]>;
}

////////////////////////////////////////////////////////////////////////////////
// Counting elements

def SVCNTB_PAT : SInst<"svcntb_pat", "nI", "", MergeNone, "aarch64_sve_cntb", [IsOverloadNone, VerifyRuntimeMode]>;
def SVCNTH_PAT : SInst<"svcnth_pat", "nI", "", MergeNone, "aarch64_sve_cnth", [IsOverloadNone, VerifyRuntimeMode]>;
def SVCNTW_PAT : SInst<"svcntw_pat", "nI", "", MergeNone, "aarch64_sve_cntw", [IsOverloadNone, VerifyRuntimeMode]>;
def SVCNTD_PAT : SInst<"svcntd_pat", "nI", "", MergeNone, "aarch64_sve_cntd", [IsOverloadNone, VerifyRuntimeMode]>;

def SVCNTB : SInst<"svcntb", "nv", "", MergeNone, "aarch64_sve_cntb", [IsAppendSVALL, IsOverloadNone, VerifyRuntimeMode]>;
def SVCNTH : SInst<"svcnth", "nv", "", MergeNone, "aarch64_sve_cnth", [IsAppendSVALL, IsOverloadNone, VerifyRuntimeMode]>;
def SVCNTW : SInst<"svcntw", "nv", "", MergeNone, "aarch64_sve_cntw", [IsAppendSVALL, IsOverloadNone, VerifyRuntimeMode]>;
def SVCNTD : SInst<"svcntd", "nv", "", MergeNone, "aarch64_sve_cntd", [IsAppendSVALL, IsOverloadNone, VerifyRuntimeMode]>;

````
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1122 EN**: Declares TableGen def record `SVPTEST_ANY`.
  **L1122 CN**: 声明 TableGen def 记录 `SVPTEST_ANY`。
- **L1123 EN**: Declares TableGen def record `SVPTEST_FIRST`.
  **L1123 CN**: 声明 TableGen def 记录 `SVPTEST_FIRST`。
- **L1124 EN**: Declares TableGen def record `SVPTEST_LAST`.
  **L1124 CN**: 声明 TableGen def 记录 `SVPTEST_LAST`。
- **L1125 EN**: Blank line separating nearby declarations or logic blocks.
  **L1125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1126 EN**: Separator comment used for visual grouping.
  **L1126 CN**: 用于视觉分组的分隔注释。
- **L1127 EN**: Comment explains nearby logic, constraints, or intent: `FFR manipulation`.
  **L1127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FFR manipulation`。
- **L1128 EN**: Blank line separating nearby declarations or logic blocks.
  **L1128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1129 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = InvalidMode in {`.
  **L1129 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = InvalidMode in {`。
- **L1130 EN**: Declares TableGen def record `SVRDFFR`.
  **L1130 CN**: 声明 TableGen def 记录 `SVRDFFR`。
- **L1131 EN**: Declares TableGen def record `SVRDFFR_Z`.
  **L1131 CN**: 声明 TableGen def 记录 `SVRDFFR_Z`。
- **L1132 EN**: Declares TableGen def record `SVSETFFR`.
  **L1132 CN**: 声明 TableGen def 记录 `SVSETFFR`。
- **L1133 EN**: Declares TableGen def record `SVWRFFR`.
  **L1133 CN**: 声明 TableGen def 记录 `SVWRFFR`。
- **L1134 EN**: Closes the current lexical scope or compound statement.
  **L1134 CN**: 结束当前词法作用域或复合语句块。
- **L1135 EN**: Blank line separating nearby declarations or logic blocks.
  **L1135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1136 EN**: Separator comment used for visual grouping.
  **L1136 CN**: 用于视觉分组的分隔注释。
- **L1137 EN**: Comment explains nearby logic, constraints, or intent: `Counting elements`.
  **L1137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Counting elements`。
- **L1138 EN**: Blank line separating nearby declarations or logic blocks.
  **L1138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1139 EN**: Declares TableGen def record `SVCNTB_PAT`.
  **L1139 CN**: 声明 TableGen def 记录 `SVCNTB_PAT`。
- **L1140 EN**: Declares TableGen def record `SVCNTH_PAT`.
  **L1140 CN**: 声明 TableGen def 记录 `SVCNTH_PAT`。
- **L1141 EN**: Declares TableGen def record `SVCNTW_PAT`.
  **L1141 CN**: 声明 TableGen def 记录 `SVCNTW_PAT`。
- **L1142 EN**: Declares TableGen def record `SVCNTD_PAT`.
  **L1142 CN**: 声明 TableGen def 记录 `SVCNTD_PAT`。
- **L1143 EN**: Blank line separating nearby declarations or logic blocks.
  **L1143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1144 EN**: Declares TableGen def record `SVCNTB`.
  **L1144 CN**: 声明 TableGen def 记录 `SVCNTB`。
- **L1145 EN**: Declares TableGen def record `SVCNTH`.
  **L1145 CN**: 声明 TableGen def 记录 `SVCNTH`。
- **L1146 EN**: Declares TableGen def record `SVCNTW`.
  **L1146 CN**: 声明 TableGen def 记录 `SVCNTW`。
- **L1147 EN**: Declares TableGen def record `SVCNTD`.
  **L1147 CN**: 声明 TableGen def 记录 `SVCNTD`。
- **L1148 EN**: Blank line separating nearby declarations or logic blocks.
  **L1148 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1149-1176

````tablegen
def SVCNTP : SInst<"svcntp_{d}",  "nPP", "PcPsPiPl",         MergeNone, "aarch64_sve_cntp", [VerifyRuntimeMode]>;
def SVLEN  : SInst<"svlen[_{d}]", "nd",  "csilUcUsUiUlhfdb", MergeNone, "", [VerifyRuntimeMode]>;

let SVETargetGuard = "sve2p2|sme2p2",  SMETargetGuard = "sve2p2|sme2p2" in {
  def SVFIRSTP  : SInst<"svfirstp_{d}", "lPP", "PcPsPiPl", MergeNone, "aarch64_sve_firstp", [VerifyRuntimeMode], []>;
  def SVLASTP  : SInst<"svlastp_{d}", "lPP", "PcPsPiPl", MergeNone, "aarch64_sve_lastp", [VerifyRuntimeMode], []>;
}

////////////////////////////////////////////////////////////////////////////////
// Saturating scalar arithmetic

class sat_type<string u, string t> { string U = u; string T = t; }
def SignedByte         : sat_type<"",  "c">;
def SignedHalf         : sat_type<"",  "s">;
def SignedWord         : sat_type<"",  "i">;
def SignedDoubleWord   : sat_type<"",  "l">;
def UnsignedByte       : sat_type<"U", "Uc">;
def UnsignedHalf       : sat_type<"U", "Us">;
def UnsignedWord       : sat_type<"U", "Ui">;
def UnsignedDoubleWord : sat_type<"U", "Ul">;

multiclass SInst_SAT1<string name, string intrinsic, sat_type type> {
  def _N32     : SInst<name # "_pat[_n_{d}]", "ssIi", type.U # "i", MergeNone, intrinsic # "_n32", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck1_16>]>;
  def _N64     : SInst<name # "_pat[_n_{d}]", "ssIi", type.U # "l", MergeNone, intrinsic # "_n64", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck1_16>]>;
  def _N32_ALL : SInst<name # "[_n_{d}]",     "ssi",  type.U # "i", MergeNone, intrinsic # "_n32", [IsOverloadNone, IsInsertOp1SVALL, VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;
  def _N64_ALL : SInst<name # "[_n_{d}]",     "ssi",  type.U # "l", MergeNone, intrinsic # "_n64", [IsOverloadNone, IsInsertOp1SVALL, VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;
}

````
- **L1149 EN**: Declares TableGen def record `SVCNTP`.
  **L1149 CN**: 声明 TableGen def 记录 `SVCNTP`。
- **L1150 EN**: Declares TableGen def record `SVLEN`.
  **L1150 CN**: 声明 TableGen def 记录 `SVLEN`。
- **L1151 EN**: Blank line separating nearby declarations or logic blocks.
  **L1151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1152 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p2|sme2p2",  SMETargetGuard = "sve2p2|sme2p2" in {`.
  **L1152 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p2|sme2p2",  SMETargetGuard = "sve2p2|sme2p2" in {`。
- **L1153 EN**: Declares TableGen def record `SVFIRSTP`.
  **L1153 CN**: 声明 TableGen def 记录 `SVFIRSTP`。
- **L1154 EN**: Declares TableGen def record `SVLASTP`.
  **L1154 CN**: 声明 TableGen def 记录 `SVLASTP`。
- **L1155 EN**: Closes the current lexical scope or compound statement.
  **L1155 CN**: 结束当前词法作用域或复合语句块。
- **L1156 EN**: Blank line separating nearby declarations or logic blocks.
  **L1156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1157 EN**: Separator comment used for visual grouping.
  **L1157 CN**: 用于视觉分组的分隔注释。
- **L1158 EN**: Comment explains nearby logic, constraints, or intent: `Saturating scalar arithmetic`.
  **L1158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Saturating scalar arithmetic`。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1160 EN**: Declares TableGen class record `sat_type`.
  **L1160 CN**: 声明 TableGen class 记录 `sat_type`。
- **L1161 EN**: Declares TableGen def record `SignedByte`.
  **L1161 CN**: 声明 TableGen def 记录 `SignedByte`。
- **L1162 EN**: Declares TableGen def record `SignedHalf`.
  **L1162 CN**: 声明 TableGen def 记录 `SignedHalf`。
- **L1163 EN**: Declares TableGen def record `SignedWord`.
  **L1163 CN**: 声明 TableGen def 记录 `SignedWord`。
- **L1164 EN**: Declares TableGen def record `SignedDoubleWord`.
  **L1164 CN**: 声明 TableGen def 记录 `SignedDoubleWord`。
- **L1165 EN**: Declares TableGen def record `UnsignedByte`.
  **L1165 CN**: 声明 TableGen def 记录 `UnsignedByte`。
- **L1166 EN**: Declares TableGen def record `UnsignedHalf`.
  **L1166 CN**: 声明 TableGen def 记录 `UnsignedHalf`。
- **L1167 EN**: Declares TableGen def record `UnsignedWord`.
  **L1167 CN**: 声明 TableGen def 记录 `UnsignedWord`。
- **L1168 EN**: Declares TableGen def record `UnsignedDoubleWord`.
  **L1168 CN**: 声明 TableGen def 记录 `UnsignedDoubleWord`。
- **L1169 EN**: Blank line separating nearby declarations or logic blocks.
  **L1169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1170 EN**: Declares TableGen multiclass record `SInst_SAT1`.
  **L1170 CN**: 声明 TableGen multiclass 记录 `SInst_SAT1`。
- **L1171 EN**: Declares TableGen def record `_N32`.
  **L1171 CN**: 声明 TableGen def 记录 `_N32`。
- **L1172 EN**: Declares TableGen def record `_N64`.
  **L1172 CN**: 声明 TableGen def 记录 `_N64`。
- **L1173 EN**: Declares TableGen def record `_N32_ALL`.
  **L1173 CN**: 声明 TableGen def 记录 `_N32_ALL`。
- **L1174 EN**: Declares TableGen def record `_N64_ALL`.
  **L1174 CN**: 声明 TableGen def 记录 `_N64_ALL`。
- **L1175 EN**: Closes the current lexical scope or compound statement.
  **L1175 CN**: 结束当前词法作用域或复合语句块。
- **L1176 EN**: Blank line separating nearby declarations or logic blocks.
  **L1176 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1177-1204

````tablegen
multiclass SInst_SAT2<string name, string intrinsic, sat_type type> {
  def ""       : SInst<name # "_pat[_{d}]",   "ddIi", type.T,       MergeNone, intrinsic, [VerifyRuntimeMode], [ImmCheck<2, ImmCheck1_16>]>;
  def _ALL     : SInst<name # "[_{d}]",       "ddi",  type.T,       MergeNone, intrinsic, [IsInsertOp1SVALL, VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;

  def _N32     : SInst<name # "_pat[_n_{d}]", "ssIi", type.U # "i", MergeNone, intrinsic # "_n32", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck1_16>]>;
  def _N64     : SInst<name # "_pat[_n_{d}]", "ssIi", type.U # "l", MergeNone, intrinsic # "_n64", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck1_16>]>;
  def _N32_ALL : SInst<name # "[_n_{d}]",     "ssi",  type.U # "i", MergeNone, intrinsic # "_n32", [IsOverloadNone, IsInsertOp1SVALL, VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;
  def _N64_ALL : SInst<name # "[_n_{d}]",     "ssi",  type.U # "l", MergeNone, intrinsic # "_n64", [IsOverloadNone, IsInsertOp1SVALL, VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;
}

defm SVQDECB_S : SInst_SAT1<"svqdecb", "aarch64_sve_sqdecb", SignedByte>;
defm SVQDECB_U : SInst_SAT1<"svqdecb", "aarch64_sve_uqdecb", UnsignedByte>;
defm SVQDECH_S : SInst_SAT2<"svqdech", "aarch64_sve_sqdech", SignedHalf>;
defm SVQDECH_U : SInst_SAT2<"svqdech", "aarch64_sve_uqdech", UnsignedHalf>;
defm SVQDECW_S : SInst_SAT2<"svqdecw", "aarch64_sve_sqdecw", SignedWord>;
defm SVQDECW_U : SInst_SAT2<"svqdecw", "aarch64_sve_uqdecw", UnsignedWord>;
defm SVQDECD_S : SInst_SAT2<"svqdecd", "aarch64_sve_sqdecd", SignedDoubleWord>;
defm SVQDECD_U : SInst_SAT2<"svqdecd", "aarch64_sve_uqdecd", UnsignedDoubleWord>;

defm SVQINCB_S : SInst_SAT1<"svqincb", "aarch64_sve_sqincb", SignedByte>;
defm SVQINCB_U : SInst_SAT1<"svqincb", "aarch64_sve_uqincb", UnsignedByte>;
defm SVQINCH_S : SInst_SAT2<"svqinch", "aarch64_sve_sqinch", SignedHalf>;
defm SVQINCH_U : SInst_SAT2<"svqinch", "aarch64_sve_uqinch", UnsignedHalf>;
defm SVQINCW_S : SInst_SAT2<"svqincw", "aarch64_sve_sqincw", SignedWord>;
defm SVQINCW_U : SInst_SAT2<"svqincw", "aarch64_sve_uqincw", UnsignedWord>;
defm SVQINCD_S : SInst_SAT2<"svqincd", "aarch64_sve_sqincd", SignedDoubleWord>;
defm SVQINCD_U : SInst_SAT2<"svqincd", "aarch64_sve_uqincd", UnsignedDoubleWord>;

````
- **L1177 EN**: Declares TableGen multiclass record `SInst_SAT2`.
  **L1177 CN**: 声明 TableGen multiclass 记录 `SInst_SAT2`。
- **L1178 EN**: Declares TableGen def record `""`.
  **L1178 CN**: 声明 TableGen def 记录 `""`。
- **L1179 EN**: Declares TableGen def record `_ALL`.
  **L1179 CN**: 声明 TableGen def 记录 `_ALL`。
- **L1180 EN**: Blank line separating nearby declarations or logic blocks.
  **L1180 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1181 EN**: Declares TableGen def record `_N32`.
  **L1181 CN**: 声明 TableGen def 记录 `_N32`。
- **L1182 EN**: Declares TableGen def record `_N64`.
  **L1182 CN**: 声明 TableGen def 记录 `_N64`。
- **L1183 EN**: Declares TableGen def record `_N32_ALL`.
  **L1183 CN**: 声明 TableGen def 记录 `_N32_ALL`。
- **L1184 EN**: Declares TableGen def record `_N64_ALL`.
  **L1184 CN**: 声明 TableGen def 记录 `_N64_ALL`。
- **L1185 EN**: Closes the current lexical scope or compound statement.
  **L1185 CN**: 结束当前词法作用域或复合语句块。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1187 EN**: Declares TableGen defm record `SVQDECB_S`.
  **L1187 CN**: 声明 TableGen defm 记录 `SVQDECB_S`。
- **L1188 EN**: Declares TableGen defm record `SVQDECB_U`.
  **L1188 CN**: 声明 TableGen defm 记录 `SVQDECB_U`。
- **L1189 EN**: Declares TableGen defm record `SVQDECH_S`.
  **L1189 CN**: 声明 TableGen defm 记录 `SVQDECH_S`。
- **L1190 EN**: Declares TableGen defm record `SVQDECH_U`.
  **L1190 CN**: 声明 TableGen defm 记录 `SVQDECH_U`。
- **L1191 EN**: Declares TableGen defm record `SVQDECW_S`.
  **L1191 CN**: 声明 TableGen defm 记录 `SVQDECW_S`。
- **L1192 EN**: Declares TableGen defm record `SVQDECW_U`.
  **L1192 CN**: 声明 TableGen defm 记录 `SVQDECW_U`。
- **L1193 EN**: Declares TableGen defm record `SVQDECD_S`.
  **L1193 CN**: 声明 TableGen defm 记录 `SVQDECD_S`。
- **L1194 EN**: Declares TableGen defm record `SVQDECD_U`.
  **L1194 CN**: 声明 TableGen defm 记录 `SVQDECD_U`。
- **L1195 EN**: Blank line separating nearby declarations or logic blocks.
  **L1195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1196 EN**: Declares TableGen defm record `SVQINCB_S`.
  **L1196 CN**: 声明 TableGen defm 记录 `SVQINCB_S`。
- **L1197 EN**: Declares TableGen defm record `SVQINCB_U`.
  **L1197 CN**: 声明 TableGen defm 记录 `SVQINCB_U`。
- **L1198 EN**: Declares TableGen defm record `SVQINCH_S`.
  **L1198 CN**: 声明 TableGen defm 记录 `SVQINCH_S`。
- **L1199 EN**: Declares TableGen defm record `SVQINCH_U`.
  **L1199 CN**: 声明 TableGen defm 记录 `SVQINCH_U`。
- **L1200 EN**: Declares TableGen defm record `SVQINCW_S`.
  **L1200 CN**: 声明 TableGen defm 记录 `SVQINCW_S`。
- **L1201 EN**: Declares TableGen defm record `SVQINCW_U`.
  **L1201 CN**: 声明 TableGen defm 记录 `SVQINCW_U`。
- **L1202 EN**: Declares TableGen defm record `SVQINCD_S`.
  **L1202 CN**: 声明 TableGen defm 记录 `SVQINCD_S`。
- **L1203 EN**: Declares TableGen defm record `SVQINCD_U`.
  **L1203 CN**: 声明 TableGen defm 记录 `SVQINCD_U`。
- **L1204 EN**: Blank line separating nearby declarations or logic blocks.
  **L1204 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1205-1232

````tablegen
def SVQDECP_S : SInst<"svqdecp[_{d}]", "ddP", "sil",    MergeNone, "aarch64_sve_sqdecp", [VerifyRuntimeMode]>;
def SVQDECP_U : SInst<"svqdecp[_{d}]", "ddP", "UsUiUl", MergeNone, "aarch64_sve_uqdecp", [VerifyRuntimeMode]>;
def SVQINCP_S : SInst<"svqincp[_{d}]", "ddP", "sil",    MergeNone, "aarch64_sve_sqincp", [VerifyRuntimeMode]>;
def SVQINCP_U : SInst<"svqincp[_{d}]", "ddP", "UsUiUl", MergeNone, "aarch64_sve_uqincp", [VerifyRuntimeMode]>;

def SVQDECP_N_S32 : SInst<"svqdecp[_n_s32]_{d}", "kkP", "PcPsPiPl", MergeNone, "aarch64_sve_sqdecp_n32", [VerifyRuntimeMode]>;
def SVQDECP_N_S64 : SInst<"svqdecp[_n_s64]_{d}", "llP", "PcPsPiPl", MergeNone, "aarch64_sve_sqdecp_n64", [VerifyRuntimeMode]>;
def SVQDECP_N_U32 : SInst<"svqdecp[_n_u32]_{d}", "mmP", "PcPsPiPl", MergeNone, "aarch64_sve_uqdecp_n32", [VerifyRuntimeMode]>;
def SVQDECP_N_U64 : SInst<"svqdecp[_n_u64]_{d}", "nnP", "PcPsPiPl", MergeNone, "aarch64_sve_uqdecp_n64", [VerifyRuntimeMode]>;
def SVQINCP_N_S32 : SInst<"svqincp[_n_s32]_{d}", "kkP", "PcPsPiPl", MergeNone, "aarch64_sve_sqincp_n32", [VerifyRuntimeMode]>;
def SVQINCP_N_S64 : SInst<"svqincp[_n_s64]_{d}", "llP", "PcPsPiPl", MergeNone, "aarch64_sve_sqincp_n64", [VerifyRuntimeMode]>;
def SVQINCP_N_U32 : SInst<"svqincp[_n_u32]_{d}", "mmP", "PcPsPiPl", MergeNone, "aarch64_sve_uqincp_n32", [VerifyRuntimeMode]>;
def SVQINCP_N_U64 : SInst<"svqincp[_n_u64]_{d}", "nnP", "PcPsPiPl", MergeNone, "aarch64_sve_uqincp_n64", [VerifyRuntimeMode]>;

let SVETargetGuard = "i8mm", SMETargetGuard = InvalidMode in {
def SVMMLA_S32   : SInst<"svmmla[_s32]",   "ddqq","i",  MergeNone, "aarch64_sve_smmla">;
def SVMMLA_U32   : SInst<"svmmla[_u32]",   "ddqq","Ui", MergeNone, "aarch64_sve_ummla">;
def SVUSMMLA_S32 : SInst<"svusmmla[_s32]", "ddbq","i",  MergeNone, "aarch64_sve_usmmla">;
}

let SVETargetGuard = "i8mm", SMETargetGuard = "i8mm"in {
def SVUSDOT_S   : SInst<"svusdot[_s32]",   "ddbq", "i", MergeNone, "aarch64_sve_usdot", [VerifyRuntimeMode]>;
def SVUSDOT_N_S : SInst<"svusdot[_n_s32]", "ddbr", "i", MergeNone, "aarch64_sve_usdot", [VerifyRuntimeMode]>;
def SVSUDOT_S   : SInst<"svsudot[_s32]",   "ddqb", "i", MergeNone, "aarch64_sve_usdot", [ReverseUSDOT, VerifyRuntimeMode]>;
def SVSUDOT_N_S : SInst<"svsudot[_n_s32]", "ddq@", "i", MergeNone, "aarch64_sve_usdot", [ReverseUSDOT, VerifyRuntimeMode]>;

def SVUSDOT_LANE_S : SInst<"svusdot_lane[_s32]", "ddbqi",  "i", MergeNone, "aarch64_sve_usdot_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndexDot, 2>]>;
def SVSUDOT_LANE_S : SInst<"svsudot_lane[_s32]", "ddqbi",  "i", MergeNone, "aarch64_sve_sudot_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndexDot, 2>]>;
````
- **L1205 EN**: Declares TableGen def record `SVQDECP_S`.
  **L1205 CN**: 声明 TableGen def 记录 `SVQDECP_S`。
- **L1206 EN**: Declares TableGen def record `SVQDECP_U`.
  **L1206 CN**: 声明 TableGen def 记录 `SVQDECP_U`。
- **L1207 EN**: Declares TableGen def record `SVQINCP_S`.
  **L1207 CN**: 声明 TableGen def 记录 `SVQINCP_S`。
- **L1208 EN**: Declares TableGen def record `SVQINCP_U`.
  **L1208 CN**: 声明 TableGen def 记录 `SVQINCP_U`。
- **L1209 EN**: Blank line separating nearby declarations or logic blocks.
  **L1209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1210 EN**: Declares TableGen def record `SVQDECP_N_S32`.
  **L1210 CN**: 声明 TableGen def 记录 `SVQDECP_N_S32`。
- **L1211 EN**: Declares TableGen def record `SVQDECP_N_S64`.
  **L1211 CN**: 声明 TableGen def 记录 `SVQDECP_N_S64`。
- **L1212 EN**: Declares TableGen def record `SVQDECP_N_U32`.
  **L1212 CN**: 声明 TableGen def 记录 `SVQDECP_N_U32`。
- **L1213 EN**: Declares TableGen def record `SVQDECP_N_U64`.
  **L1213 CN**: 声明 TableGen def 记录 `SVQDECP_N_U64`。
- **L1214 EN**: Declares TableGen def record `SVQINCP_N_S32`.
  **L1214 CN**: 声明 TableGen def 记录 `SVQINCP_N_S32`。
- **L1215 EN**: Declares TableGen def record `SVQINCP_N_S64`.
  **L1215 CN**: 声明 TableGen def 记录 `SVQINCP_N_S64`。
- **L1216 EN**: Declares TableGen def record `SVQINCP_N_U32`.
  **L1216 CN**: 声明 TableGen def 记录 `SVQINCP_N_U32`。
- **L1217 EN**: Declares TableGen def record `SVQINCP_N_U64`.
  **L1217 CN**: 声明 TableGen def 记录 `SVQINCP_N_U64`。
- **L1218 EN**: Blank line separating nearby declarations or logic blocks.
  **L1218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1219 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "i8mm", SMETargetGuard = InvalidMode in {`.
  **L1219 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "i8mm", SMETargetGuard = InvalidMode in {`。
- **L1220 EN**: Declares TableGen def record `SVMMLA_S32`.
  **L1220 CN**: 声明 TableGen def 记录 `SVMMLA_S32`。
- **L1221 EN**: Declares TableGen def record `SVMMLA_U32`.
  **L1221 CN**: 声明 TableGen def 记录 `SVMMLA_U32`。
- **L1222 EN**: Declares TableGen def record `SVUSMMLA_S32`.
  **L1222 CN**: 声明 TableGen def 记录 `SVUSMMLA_S32`。
- **L1223 EN**: Closes the current lexical scope or compound statement.
  **L1223 CN**: 结束当前词法作用域或复合语句块。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1225 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "i8mm", SMETargetGuard = "i8mm"in {`.
  **L1225 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "i8mm", SMETargetGuard = "i8mm"in {`。
- **L1226 EN**: Declares TableGen def record `SVUSDOT_S`.
  **L1226 CN**: 声明 TableGen def 记录 `SVUSDOT_S`。
- **L1227 EN**: Declares TableGen def record `SVUSDOT_N_S`.
  **L1227 CN**: 声明 TableGen def 记录 `SVUSDOT_N_S`。
- **L1228 EN**: Declares TableGen def record `SVSUDOT_S`.
  **L1228 CN**: 声明 TableGen def 记录 `SVSUDOT_S`。
- **L1229 EN**: Declares TableGen def record `SVSUDOT_N_S`.
  **L1229 CN**: 声明 TableGen def 记录 `SVSUDOT_N_S`。
- **L1230 EN**: Blank line separating nearby declarations or logic blocks.
  **L1230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1231 EN**: Declares TableGen def record `SVUSDOT_LANE_S`.
  **L1231 CN**: 声明 TableGen def 记录 `SVUSDOT_LANE_S`。
- **L1232 EN**: Declares TableGen def record `SVSUDOT_LANE_S`.
  **L1232 CN**: 声明 TableGen def 记录 `SVSUDOT_LANE_S`。

### Lines 1233-1260

````tablegen
}

let SVETargetGuard = "f32mm", SMETargetGuard = InvalidMode in {
def SVMMLA_F32 : SInst<"svmmla[_f32]", "dddd","f", MergeNone, "aarch64_sve_fmmla", [IsOverloadFirstandLast]>;
}

let SVETargetGuard = "f64mm", SMETargetGuard = InvalidMode in {
def SVMMLA_F64 : SInst<"svmmla[_f64]",  "dddd", "d", MergeNone, "aarch64_sve_fmmla", [IsOverloadFirstandLast]>;

let SVETargetGuard = "sve-f16f32mm", SMETargetGuard = InvalidMode in {
  def SVMMLA_F32_F16  : SInst<"svmmla[_f32_f16]", "ddhh", "f", MergeNone, "aarch64_sve_fmmla", [IsOverloadFirstandLast]>;
}

let SVETargetGuard = "sve2p2,f16mm", SMETargetGuard = InvalidMode in {
  def SVMMLA_F16 : SInst<"svmmla[_f16]", "dddd", "h", MergeNone, "aarch64_sve_fmmla", [IsOverloadFirstandLast]>;
}

let SVETargetGuard = "sve2,f8f32mm", SMETargetGuard = InvalidMode in {
  def SVMMLA_F32_MF8 : SInst<"svmmla[_f32_mf8]", "dd~~>", "f", MergeNone, "aarch64_sve_fp8_fmmla">;
}

let SVETargetGuard = "sve2,f8f16mm", SMETargetGuard = InvalidMode in {
  def SVMMLA_F16_MF8 : SInst<"svmmla[_f16_mf8]", "dd~~>", "h", MergeNone, "aarch64_sve_fp8_fmmla">;
}

def SVTRN1Q : SInst<"svtrn1q[_{d}]", "ddd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_trn1q">;
def SVTRN2Q : SInst<"svtrn2q[_{d}]", "ddd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_trn2q">;
def SVUZP1Q : SInst<"svuzp1q[_{d}]", "ddd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_uzp1q">;
````
- **L1233 EN**: Closes the current lexical scope or compound statement.
  **L1233 CN**: 结束当前词法作用域或复合语句块。
- **L1234 EN**: Blank line separating nearby declarations or logic blocks.
  **L1234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1235 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "f32mm", SMETargetGuard = InvalidMode in {`.
  **L1235 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "f32mm", SMETargetGuard = InvalidMode in {`。
- **L1236 EN**: Declares TableGen def record `SVMMLA_F32`.
  **L1236 CN**: 声明 TableGen def 记录 `SVMMLA_F32`。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Blank line separating nearby declarations or logic blocks.
  **L1238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1239 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "f64mm", SMETargetGuard = InvalidMode in {`.
  **L1239 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "f64mm", SMETargetGuard = InvalidMode in {`。
- **L1240 EN**: Declares TableGen def record `SVMMLA_F64`.
  **L1240 CN**: 声明 TableGen def 记录 `SVMMLA_F64`。
- **L1241 EN**: Blank line separating nearby declarations or logic blocks.
  **L1241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1242 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-f16f32mm", SMETargetGuard = InvalidMode in {`.
  **L1242 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-f16f32mm", SMETargetGuard = InvalidMode in {`。
- **L1243 EN**: Declares TableGen def record `SVMMLA_F32_F16`.
  **L1243 CN**: 声明 TableGen def 记录 `SVMMLA_F32_F16`。
- **L1244 EN**: Closes the current lexical scope or compound statement.
  **L1244 CN**: 结束当前词法作用域或复合语句块。
- **L1245 EN**: Blank line separating nearby declarations or logic blocks.
  **L1245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1246 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p2,f16mm", SMETargetGuard = InvalidMode in {`.
  **L1246 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p2,f16mm", SMETargetGuard = InvalidMode in {`。
- **L1247 EN**: Declares TableGen def record `SVMMLA_F16`.
  **L1247 CN**: 声明 TableGen def 记录 `SVMMLA_F16`。
- **L1248 EN**: Closes the current lexical scope or compound statement.
  **L1248 CN**: 结束当前词法作用域或复合语句块。
- **L1249 EN**: Blank line separating nearby declarations or logic blocks.
  **L1249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1250 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2,f8f32mm", SMETargetGuard = InvalidMode in {`.
  **L1250 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2,f8f32mm", SMETargetGuard = InvalidMode in {`。
- **L1251 EN**: Declares TableGen def record `SVMMLA_F32_MF8`.
  **L1251 CN**: 声明 TableGen def 记录 `SVMMLA_F32_MF8`。
- **L1252 EN**: Closes the current lexical scope or compound statement.
  **L1252 CN**: 结束当前词法作用域或复合语句块。
- **L1253 EN**: Blank line separating nearby declarations or logic blocks.
  **L1253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1254 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2,f8f16mm", SMETargetGuard = InvalidMode in {`.
  **L1254 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2,f8f16mm", SMETargetGuard = InvalidMode in {`。
- **L1255 EN**: Declares TableGen def record `SVMMLA_F16_MF8`.
  **L1255 CN**: 声明 TableGen def 记录 `SVMMLA_F16_MF8`。
- **L1256 EN**: Closes the current lexical scope or compound statement.
  **L1256 CN**: 结束当前词法作用域或复合语句块。
- **L1257 EN**: Blank line separating nearby declarations or logic blocks.
  **L1257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1258 EN**: Declares TableGen def record `SVTRN1Q`.
  **L1258 CN**: 声明 TableGen def 记录 `SVTRN1Q`。
- **L1259 EN**: Declares TableGen def record `SVTRN2Q`.
  **L1259 CN**: 声明 TableGen def 记录 `SVTRN2Q`。
- **L1260 EN**: Declares TableGen def record `SVUZP1Q`.
  **L1260 CN**: 声明 TableGen def 记录 `SVUZP1Q`。

### Lines 1261-1288

````tablegen
def SVUZP2Q : SInst<"svuzp2q[_{d}]", "ddd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_uzp2q">;
def SVZIP1Q : SInst<"svzip1q[_{d}]", "ddd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_zip1q">;
def SVZIP2Q : SInst<"svzip2q[_{d}]", "ddd", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_zip2q">;
}

////////////////////////////////////////////////////////////////////////////////
// Vector creation
def SVUNDEF_1 : SInst<"svundef_{d}",  "dv", "csilUcUsUiUlhfdbm", MergeNone, "", [IsUndef, VerifyRuntimeMode]>;
def SVUNDEF_2 : SInst<"svundef2_{d}", "2v", "csilUcUsUiUlhfdbm", MergeNone, "", [IsUndef, VerifyRuntimeMode]>;
def SVUNDEF_3 : SInst<"svundef3_{d}", "3v", "csilUcUsUiUlhfdbm", MergeNone, "", [IsUndef, VerifyRuntimeMode]>;
def SVUNDEF_4 : SInst<"svundef4_{d}", "4v", "csilUcUsUiUlhfdbm", MergeNone, "", [IsUndef, VerifyRuntimeMode]>;

def SVCREATE_2 : SInst<"svcreate2[_{d}]", "2dd",   "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleCreate, VerifyRuntimeMode]>;
def SVCREATE_3 : SInst<"svcreate3[_{d}]", "3ddd",  "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleCreate, VerifyRuntimeMode]>;
def SVCREATE_4 : SInst<"svcreate4[_{d}]", "4dddd", "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleCreate, VerifyRuntimeMode]>;

let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {
  def SVCREATE_2_B : SInst<"svcreate2[_b]", "2dd",   "Pc", MergeNone, "", [IsTupleCreate, VerifyRuntimeMode]>;
  def SVCREATE_4_B : SInst<"svcreate4[_b]", "4dddd", "Pc", MergeNone, "", [IsTupleCreate, VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// Vector insertion and extraction
def SVGET_2 : SInst<"svget2[_{d}]", "d2i", "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleGet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_1>]>;
def SVGET_3 : SInst<"svget3[_{d}]", "d3i", "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleGet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_2>]>;
def SVGET_4 : SInst<"svget4[_{d}]", "d4i", "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleGet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_3>]>;

def SVSET_2 : SInst<"svset2[_{d}]", "22id", "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleSet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_1>]>;
````
- **L1261 EN**: Declares TableGen def record `SVUZP2Q`.
  **L1261 CN**: 声明 TableGen def 记录 `SVUZP2Q`。
- **L1262 EN**: Declares TableGen def record `SVZIP1Q`.
  **L1262 CN**: 声明 TableGen def 记录 `SVZIP1Q`。
- **L1263 EN**: Declares TableGen def record `SVZIP2Q`.
  **L1263 CN**: 声明 TableGen def 记录 `SVZIP2Q`。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Separator comment used for visual grouping.
  **L1266 CN**: 用于视觉分组的分隔注释。
- **L1267 EN**: Comment explains nearby logic, constraints, or intent: `Vector creation`.
  **L1267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector creation`。
- **L1268 EN**: Declares TableGen def record `SVUNDEF_1`.
  **L1268 CN**: 声明 TableGen def 记录 `SVUNDEF_1`。
- **L1269 EN**: Declares TableGen def record `SVUNDEF_2`.
  **L1269 CN**: 声明 TableGen def 记录 `SVUNDEF_2`。
- **L1270 EN**: Declares TableGen def record `SVUNDEF_3`.
  **L1270 CN**: 声明 TableGen def 记录 `SVUNDEF_3`。
- **L1271 EN**: Declares TableGen def record `SVUNDEF_4`.
  **L1271 CN**: 声明 TableGen def 记录 `SVUNDEF_4`。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1273 EN**: Declares TableGen def record `SVCREATE_2`.
  **L1273 CN**: 声明 TableGen def 记录 `SVCREATE_2`。
- **L1274 EN**: Declares TableGen def record `SVCREATE_3`.
  **L1274 CN**: 声明 TableGen def 记录 `SVCREATE_3`。
- **L1275 EN**: Declares TableGen def record `SVCREATE_4`.
  **L1275 CN**: 声明 TableGen def 记录 `SVCREATE_4`。
- **L1276 EN**: Blank line separating nearby declarations or logic blocks.
  **L1276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1277 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`.
  **L1277 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`。
- **L1278 EN**: Declares TableGen def record `SVCREATE_2_B`.
  **L1278 CN**: 声明 TableGen def 记录 `SVCREATE_2_B`。
- **L1279 EN**: Declares TableGen def record `SVCREATE_4_B`.
  **L1279 CN**: 声明 TableGen def 记录 `SVCREATE_4_B`。
- **L1280 EN**: Closes the current lexical scope or compound statement.
  **L1280 CN**: 结束当前词法作用域或复合语句块。
- **L1281 EN**: Blank line separating nearby declarations or logic blocks.
  **L1281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1282 EN**: Separator comment used for visual grouping.
  **L1282 CN**: 用于视觉分组的分隔注释。
- **L1283 EN**: Comment explains nearby logic, constraints, or intent: `Vector insertion and extraction`.
  **L1283 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector insertion and extraction`。
- **L1284 EN**: Declares TableGen def record `SVGET_2`.
  **L1284 CN**: 声明 TableGen def 记录 `SVGET_2`。
- **L1285 EN**: Declares TableGen def record `SVGET_3`.
  **L1285 CN**: 声明 TableGen def 记录 `SVGET_3`。
- **L1286 EN**: Declares TableGen def record `SVGET_4`.
  **L1286 CN**: 声明 TableGen def 记录 `SVGET_4`。
- **L1287 EN**: Blank line separating nearby declarations or logic blocks.
  **L1287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1288 EN**: Declares TableGen def record `SVSET_2`.
  **L1288 CN**: 声明 TableGen def 记录 `SVSET_2`。

### Lines 1289-1316

````tablegen
def SVSET_3 : SInst<"svset3[_{d}]", "33id", "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleSet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_2>]>;
def SVSET_4 : SInst<"svset4[_{d}]", "44id", "csilUcUsUiUlhfdbm", MergeNone, "", [IsTupleSet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_3>]>;

let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {
  def SVGET_2_B : SInst<"svget2[_b]", "d2i", "Pc", MergeNone, "", [IsTupleGet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_1>]>;
  def SVGET_4_B : SInst<"svget4[_b]", "d4i", "Pc", MergeNone, "", [IsTupleGet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_3>]>;

  def SVSET_2_B : SInst<"svset2[_b]", "22id", "Pc", MergeNone, "", [IsTupleSet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_1>]>;
  def SVSET_4_B : SInst<"svset4[_b]", "44id", "Pc", MergeNone, "", [IsTupleSet, VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_3>]>;

  def SVUNDEF_2_B : Inst<"svundef2_b", "2", "Pc", MergeNone, "", [IsUndef, VerifyRuntimeMode], []>;
  def SVUNDEF_4_B : Inst<"svundef4_b", "4", "Pc", MergeNone, "", [IsUndef, VerifyRuntimeMode], []>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 WhileGE/GT
let SVETargetGuard = "sve2|sme" in {
def SVWHILEGE_S32 : SInst<"svwhilege_{d}[_{1}]", "Pkk", "PcPsPiPl", MergeNone, "aarch64_sve_whilege", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILEGE_S64 : SInst<"svwhilege_{d}[_{1}]", "Pll", "PcPsPiPl", MergeNone, "aarch64_sve_whilege", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILEGT_S32 : SInst<"svwhilegt_{d}[_{1}]", "Pkk", "PcPsPiPl", MergeNone, "aarch64_sve_whilegt", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILEGT_S64 : SInst<"svwhilegt_{d}[_{1}]", "Pll", "PcPsPiPl", MergeNone, "aarch64_sve_whilegt", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILEHI_U32 : SInst<"svwhilegt_{d}[_{1}]", "Pmm", "PcPsPiPl", MergeNone, "aarch64_sve_whilehi", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILEHI_U64 : SInst<"svwhilegt_{d}[_{1}]", "Pnn", "PcPsPiPl", MergeNone, "aarch64_sve_whilehi", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILEHS_U32 : SInst<"svwhilege_{d}[_{1}]", "Pmm", "PcPsPiPl", MergeNone, "aarch64_sve_whilehs", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
def SVWHILEHS_U64 : SInst<"svwhilege_{d}[_{1}]", "Pnn", "PcPsPiPl", MergeNone, "aarch64_sve_whilehs", [IsOverloadWhileOrMultiVecCvt, VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {
````
- **L1289 EN**: Declares TableGen def record `SVSET_3`.
  **L1289 CN**: 声明 TableGen def 记录 `SVSET_3`。
- **L1290 EN**: Declares TableGen def record `SVSET_4`.
  **L1290 CN**: 声明 TableGen def 记录 `SVSET_4`。
- **L1291 EN**: Blank line separating nearby declarations or logic blocks.
  **L1291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1292 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`.
  **L1292 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`。
- **L1293 EN**: Declares TableGen def record `SVGET_2_B`.
  **L1293 CN**: 声明 TableGen def 记录 `SVGET_2_B`。
- **L1294 EN**: Declares TableGen def record `SVGET_4_B`.
  **L1294 CN**: 声明 TableGen def 记录 `SVGET_4_B`。
- **L1295 EN**: Blank line separating nearby declarations or logic blocks.
  **L1295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1296 EN**: Declares TableGen def record `SVSET_2_B`.
  **L1296 CN**: 声明 TableGen def 记录 `SVSET_2_B`。
- **L1297 EN**: Declares TableGen def record `SVSET_4_B`.
  **L1297 CN**: 声明 TableGen def 记录 `SVSET_4_B`。
- **L1298 EN**: Blank line separating nearby declarations or logic blocks.
  **L1298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1299 EN**: Declares TableGen def record `SVUNDEF_2_B`.
  **L1299 CN**: 声明 TableGen def 记录 `SVUNDEF_2_B`。
- **L1300 EN**: Declares TableGen def record `SVUNDEF_4_B`.
  **L1300 CN**: 声明 TableGen def 记录 `SVUNDEF_4_B`。
- **L1301 EN**: Closes the current lexical scope or compound statement.
  **L1301 CN**: 结束当前词法作用域或复合语句块。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1303 EN**: Separator comment used for visual grouping.
  **L1303 CN**: 用于视觉分组的分隔注释。
- **L1304 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 WhileGE/GT`.
  **L1304 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 WhileGE/GT`。
- **L1305 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1305 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1306 EN**: Declares TableGen def record `SVWHILEGE_S32`.
  **L1306 CN**: 声明 TableGen def 记录 `SVWHILEGE_S32`。
- **L1307 EN**: Declares TableGen def record `SVWHILEGE_S64`.
  **L1307 CN**: 声明 TableGen def 记录 `SVWHILEGE_S64`。
- **L1308 EN**: Declares TableGen def record `SVWHILEGT_S32`.
  **L1308 CN**: 声明 TableGen def 记录 `SVWHILEGT_S32`。
- **L1309 EN**: Declares TableGen def record `SVWHILEGT_S64`.
  **L1309 CN**: 声明 TableGen def 记录 `SVWHILEGT_S64`。
- **L1310 EN**: Declares TableGen def record `SVWHILEHI_U32`.
  **L1310 CN**: 声明 TableGen def 记录 `SVWHILEHI_U32`。
- **L1311 EN**: Declares TableGen def record `SVWHILEHI_U64`.
  **L1311 CN**: 声明 TableGen def 记录 `SVWHILEHI_U64`。
- **L1312 EN**: Declares TableGen def record `SVWHILEHS_U32`.
  **L1312 CN**: 声明 TableGen def 记录 `SVWHILEHS_U32`。
- **L1313 EN**: Declares TableGen def record `SVWHILEHS_U64`.
  **L1313 CN**: 声明 TableGen def 记录 `SVWHILEHS_U64`。
- **L1314 EN**: Closes the current lexical scope or compound statement.
  **L1314 CN**: 结束当前词法作用域或复合语句块。
- **L1315 EN**: Blank line separating nearby declarations or logic blocks.
  **L1315 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1316 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`.
  **L1316 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`。

### Lines 1317-1344

````tablegen
  def SVWHILEGE_S64_X2 : SInst<"svwhilege_{d}[_{1}]_x2", "2ll", "PcPsPiPl", MergeNone, "aarch64_sve_whilege_x2", [VerifyRuntimeMode]>;
  def SVWHILEGT_S64_X2 : SInst<"svwhilegt_{d}[_{1}]_x2", "2ll", "PcPsPiPl", MergeNone, "aarch64_sve_whilegt_x2", [VerifyRuntimeMode]>;
  def SVWHILEHI_U64_X2 : SInst<"svwhilegt_{d}[_{1}]_x2", "2nn", "PcPsPiPl", MergeNone, "aarch64_sve_whilehi_x2", [VerifyRuntimeMode]>;
  def SVWHILEHS_U64_X2 : SInst<"svwhilege_{d}[_{1}]_x2", "2nn", "PcPsPiPl", MergeNone, "aarch64_sve_whilehs_x2", [VerifyRuntimeMode]>;
  def SVWHILELE_S64_X2 : SInst<"svwhilele_{d}[_{1}]_x2", "2ll", "PcPsPiPl", MergeNone, "aarch64_sve_whilele_x2", [VerifyRuntimeMode]>;
  def SVWHILELT_S64_X2 : SInst<"svwhilelt_{d}[_{1}]_x2", "2ll", "PcPsPiPl", MergeNone, "aarch64_sve_whilelt_x2", [VerifyRuntimeMode]>;
  def SVWHILELO_U64_X2 : SInst<"svwhilelt_{d}[_{1}]_x2", "2nn", "PcPsPiPl", MergeNone, "aarch64_sve_whilelo_x2", [VerifyRuntimeMode]>;
  def SVWHILELS_U64_X2 : SInst<"svwhilele_{d}[_{1}]_x2", "2nn", "PcPsPiPl", MergeNone, "aarch64_sve_whilels_x2", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Uniform DSP operations

let SVETargetGuard = "sve2|sme" in {
defm SVQADD_S  : SInstZPZZ<"svqadd",  "csli",     "aarch64_sve_sqadd",  "aarch64_sve_sqadd">;
defm SVQADD_U  : SInstZPZZ<"svqadd",  "UcUsUiUl", "aarch64_sve_uqadd",  "aarch64_sve_uqadd">;
defm SVHADD_S  : SInstZPZZ<"svhadd",  "csli",     "aarch64_sve_shadd",  "aarch64_sve_shadd">;
defm SVHADD_U  : SInstZPZZ<"svhadd",  "UcUsUiUl", "aarch64_sve_uhadd",  "aarch64_sve_uhadd">;
defm SVRHADD_S : SInstZPZZ<"svrhadd", "csli",     "aarch64_sve_srhadd", "aarch64_sve_srhadd">;
defm SVRHADD_U : SInstZPZZ<"svrhadd", "UcUsUiUl", "aarch64_sve_urhadd", "aarch64_sve_urhadd">;

defm SVQSUB_S  : SInstZPZZ<"svqsub",  "csli",     "aarch64_sve_sqsub",  "aarch64_sve_sqsub_u">;
defm SVQSUB_U  : SInstZPZZ<"svqsub",  "UcUsUiUl", "aarch64_sve_uqsub",  "aarch64_sve_uqsub_u">;
defm SVQSUBR_S : SInstZPZZ<"svqsubr", "csli",     "aarch64_sve_sqsubr", "aarch64_sve_sqsub_u", [ReverseMergeAnyBinOp]>;
defm SVQSUBR_U : SInstZPZZ<"svqsubr", "UcUsUiUl", "aarch64_sve_uqsubr", "aarch64_sve_uqsub_u", [ReverseMergeAnyBinOp]>;
defm SVHSUB_S  : SInstZPZZ<"svhsub",  "csli",     "aarch64_sve_shsub",  "aarch64_sve_shsub_u">;
defm SVHSUB_U  : SInstZPZZ<"svhsub",  "UcUsUiUl", "aarch64_sve_uhsub",  "aarch64_sve_uhsub_u">;
defm SVHSUBR_S : SInstZPZZ<"svhsubr", "csli",     "aarch64_sve_shsubr", "aarch64_sve_shsub_u", [ReverseMergeAnyBinOp]>;
````
- **L1317 EN**: Declares TableGen def record `SVWHILEGE_S64_X2`.
  **L1317 CN**: 声明 TableGen def 记录 `SVWHILEGE_S64_X2`。
- **L1318 EN**: Declares TableGen def record `SVWHILEGT_S64_X2`.
  **L1318 CN**: 声明 TableGen def 记录 `SVWHILEGT_S64_X2`。
- **L1319 EN**: Declares TableGen def record `SVWHILEHI_U64_X2`.
  **L1319 CN**: 声明 TableGen def 记录 `SVWHILEHI_U64_X2`。
- **L1320 EN**: Declares TableGen def record `SVWHILEHS_U64_X2`.
  **L1320 CN**: 声明 TableGen def 记录 `SVWHILEHS_U64_X2`。
- **L1321 EN**: Declares TableGen def record `SVWHILELE_S64_X2`.
  **L1321 CN**: 声明 TableGen def 记录 `SVWHILELE_S64_X2`。
- **L1322 EN**: Declares TableGen def record `SVWHILELT_S64_X2`.
  **L1322 CN**: 声明 TableGen def 记录 `SVWHILELT_S64_X2`。
- **L1323 EN**: Declares TableGen def record `SVWHILELO_U64_X2`.
  **L1323 CN**: 声明 TableGen def 记录 `SVWHILELO_U64_X2`。
- **L1324 EN**: Declares TableGen def record `SVWHILELS_U64_X2`.
  **L1324 CN**: 声明 TableGen def 记录 `SVWHILELS_U64_X2`。
- **L1325 EN**: Closes the current lexical scope or compound statement.
  **L1325 CN**: 结束当前词法作用域或复合语句块。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1327 EN**: Separator comment used for visual grouping.
  **L1327 CN**: 用于视觉分组的分隔注释。
- **L1328 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Uniform DSP operations`.
  **L1328 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Uniform DSP operations`。
- **L1329 EN**: Blank line separating nearby declarations or logic blocks.
  **L1329 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1330 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1330 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1331 EN**: Declares TableGen defm record `SVQADD_S`.
  **L1331 CN**: 声明 TableGen defm 记录 `SVQADD_S`。
- **L1332 EN**: Declares TableGen defm record `SVQADD_U`.
  **L1332 CN**: 声明 TableGen defm 记录 `SVQADD_U`。
- **L1333 EN**: Declares TableGen defm record `SVHADD_S`.
  **L1333 CN**: 声明 TableGen defm 记录 `SVHADD_S`。
- **L1334 EN**: Declares TableGen defm record `SVHADD_U`.
  **L1334 CN**: 声明 TableGen defm 记录 `SVHADD_U`。
- **L1335 EN**: Declares TableGen defm record `SVRHADD_S`.
  **L1335 CN**: 声明 TableGen defm 记录 `SVRHADD_S`。
- **L1336 EN**: Declares TableGen defm record `SVRHADD_U`.
  **L1336 CN**: 声明 TableGen defm 记录 `SVRHADD_U`。
- **L1337 EN**: Blank line separating nearby declarations or logic blocks.
  **L1337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1338 EN**: Declares TableGen defm record `SVQSUB_S`.
  **L1338 CN**: 声明 TableGen defm 记录 `SVQSUB_S`。
- **L1339 EN**: Declares TableGen defm record `SVQSUB_U`.
  **L1339 CN**: 声明 TableGen defm 记录 `SVQSUB_U`。
- **L1340 EN**: Declares TableGen defm record `SVQSUBR_S`.
  **L1340 CN**: 声明 TableGen defm 记录 `SVQSUBR_S`。
- **L1341 EN**: Declares TableGen defm record `SVQSUBR_U`.
  **L1341 CN**: 声明 TableGen defm 记录 `SVQSUBR_U`。
- **L1342 EN**: Declares TableGen defm record `SVHSUB_S`.
  **L1342 CN**: 声明 TableGen defm 记录 `SVHSUB_S`。
- **L1343 EN**: Declares TableGen defm record `SVHSUB_U`.
  **L1343 CN**: 声明 TableGen defm 记录 `SVHSUB_U`。
- **L1344 EN**: Declares TableGen defm record `SVHSUBR_S`.
  **L1344 CN**: 声明 TableGen defm 记录 `SVHSUBR_S`。

### Lines 1345-1372

````tablegen
defm SVHSUBR_U : SInstZPZZ<"svhsubr", "UcUsUiUl", "aarch64_sve_uhsubr", "aarch64_sve_uhsub_u", [ReverseMergeAnyBinOp]>;

defm SVQABS   : SInstZPZ<"svqabs",   "csil", "aarch64_sve_sqabs">;
defm SVQNEG   : SInstZPZ<"svqneg",   "csil", "aarch64_sve_sqneg">;
defm SVRECPE  : SInstZPZ<"svrecpe",  "Ui",   "aarch64_sve_urecpe">;
defm SVRSQRTE : SInstZPZ<"svrsqrte", "Ui",   "aarch64_sve_ursqrte">;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2.3 - Two-way signed/unsigned absolute difference sum and accumulate long

let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {
  def SVABAL_S : SInst<"svabal[_{d}]", "ddhh", "sil"   , MergeNone, "aarch64_sve_sabal", [VerifyRuntimeMode]>;
  def SVABAL_S_N : SInst<"svabal[_n_{d}]", "ddhR", "sil"   , MergeNone, "aarch64_sve_sabal", [VerifyRuntimeMode]>;

  def SVABAL_U : SInst<"svabal[_{d}]", "ddhh", "UsUiUl", MergeNone, "aarch64_sve_uabal", [VerifyRuntimeMode]>;
  def SVABAL_U_N : SInst<"svabal[_n_{d}]", "ddhR", "UsUiUl", MergeNone, "aarch64_sve_uabal", [VerifyRuntimeMode]>;
}

//------------------------------------------------------------------------------

multiclass SInstZPZxZ<string name, string types, string pat_v, string pat_n, string m_intrinsic, string x_intrinsic, list<FlagType> flags=[]> {
  def _M   : SInst<name # "[_{d}]", pat_v, types, MergeOp1,  m_intrinsic, flags>;
  def _X   : SInst<name # "[_{d}]", pat_v, types, MergeAny,  x_intrinsic, flags>;
  def _Z   : SInst<name # "[_{d}]", pat_v, types, MergeZero, m_intrinsic, flags>;

  def _N_M : SInst<name # "[_n_{d}]", pat_n, types, MergeOp1,  m_intrinsic, flags>;
  def _N_X : SInst<name # "[_n_{d}]", pat_n, types, MergeAny,  x_intrinsic, flags>;
````
- **L1345 EN**: Declares TableGen defm record `SVHSUBR_U`.
  **L1345 CN**: 声明 TableGen defm 记录 `SVHSUBR_U`。
- **L1346 EN**: Blank line separating nearby declarations or logic blocks.
  **L1346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1347 EN**: Declares TableGen defm record `SVQABS`.
  **L1347 CN**: 声明 TableGen defm 记录 `SVQABS`。
- **L1348 EN**: Declares TableGen defm record `SVQNEG`.
  **L1348 CN**: 声明 TableGen defm 记录 `SVQNEG`。
- **L1349 EN**: Declares TableGen defm record `SVRECPE`.
  **L1349 CN**: 声明 TableGen defm 记录 `SVRECPE`。
- **L1350 EN**: Declares TableGen defm record `SVRSQRTE`.
  **L1350 CN**: 声明 TableGen defm 记录 `SVRSQRTE`。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Blank line separating nearby declarations or logic blocks.
  **L1352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1353 EN**: Separator comment used for visual grouping.
  **L1353 CN**: 用于视觉分组的分隔注释。
- **L1354 EN**: Comment explains nearby logic, constraints, or intent: `SVE2.3 - Two-way signed/unsigned absolute difference sum and accumulate long`.
  **L1354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2.3 - Two-way signed/unsigned absolute difference sum and accumulate long`。
- **L1355 EN**: Blank line separating nearby declarations or logic blocks.
  **L1355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1356 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`.
  **L1356 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`。
- **L1357 EN**: Declares TableGen def record `SVABAL_S`.
  **L1357 CN**: 声明 TableGen def 记录 `SVABAL_S`。
- **L1358 EN**: Declares TableGen def record `SVABAL_S_N`.
  **L1358 CN**: 声明 TableGen def 记录 `SVABAL_S_N`。
- **L1359 EN**: Blank line separating nearby declarations or logic blocks.
  **L1359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1360 EN**: Declares TableGen def record `SVABAL_U`.
  **L1360 CN**: 声明 TableGen def 记录 `SVABAL_U`。
- **L1361 EN**: Declares TableGen def record `SVABAL_U_N`.
  **L1361 CN**: 声明 TableGen def 记录 `SVABAL_U_N`。
- **L1362 EN**: Closes the current lexical scope or compound statement.
  **L1362 CN**: 结束当前词法作用域或复合语句块。
- **L1363 EN**: Blank line separating nearby declarations or logic blocks.
  **L1363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1364 EN**: Separator comment used for visual grouping.
  **L1364 CN**: 用于视觉分组的分隔注释。
- **L1365 EN**: Blank line separating nearby declarations or logic blocks.
  **L1365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1366 EN**: Declares TableGen multiclass record `SInstZPZxZ`.
  **L1366 CN**: 声明 TableGen multiclass 记录 `SInstZPZxZ`。
- **L1367 EN**: Declares TableGen def record `_M`.
  **L1367 CN**: 声明 TableGen def 记录 `_M`。
- **L1368 EN**: Declares TableGen def record `_X`.
  **L1368 CN**: 声明 TableGen def 记录 `_X`。
- **L1369 EN**: Declares TableGen def record `_Z`.
  **L1369 CN**: 声明 TableGen def 记录 `_Z`。
- **L1370 EN**: Blank line separating nearby declarations or logic blocks.
  **L1370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1371 EN**: Declares TableGen def record `_N_M`.
  **L1371 CN**: 声明 TableGen def 记录 `_N_M`。
- **L1372 EN**: Declares TableGen def record `_N_X`.
  **L1372 CN**: 声明 TableGen def 记录 `_N_X`。

### Lines 1373-1400

````tablegen
  def _N_Z : SInst<name # "[_n_{d}]", pat_n, types, MergeZero, m_intrinsic, flags>;
}

let SVETargetGuard = "sve2|sme" in {
defm SVQRSHL_S : SInstZPZxZ<"svqrshl", "csil",     "dPdx", "dPdK", "aarch64_sve_sqrshl", "aarch64_sve_sqrshl_u", [VerifyRuntimeMode]>;
defm SVQRSHL_U : SInstZPZxZ<"svqrshl", "UcUsUiUl", "dPdx", "dPdK", "aarch64_sve_uqrshl", "aarch64_sve_uqrshl_u", [VerifyRuntimeMode]>;
defm SVQSHL_S  : SInstZPZxZ<"svqshl",  "csil",     "dPdx", "dPdK", "aarch64_sve_sqshl",  "aarch64_sve_sqshl_u", [VerifyRuntimeMode]>;
defm SVQSHL_U  : SInstZPZxZ<"svqshl",  "UcUsUiUl", "dPdx", "dPdK", "aarch64_sve_uqshl",  "aarch64_sve_uqshl_u", [VerifyRuntimeMode]>;
defm SVRSHL_S  : SInstZPZxZ<"svrshl",  "csil",     "dPdx", "dPdK", "aarch64_sve_srshl",  "aarch64_sve_srshl_u", [VerifyRuntimeMode]>;
defm SVRSHL_U  : SInstZPZxZ<"svrshl",  "UcUsUiUl", "dPdx", "dPdK", "aarch64_sve_urshl",  "aarch64_sve_urshl_u", [VerifyRuntimeMode]>;
defm SVSQADD   : SInstZPZxZ<"svsqadd", "UcUsUiUl", "dPdx", "dPdK", "aarch64_sve_usqadd", "aarch64_sve_usqadd", [VerifyRuntimeMode]>;
defm SVUQADD   : SInstZPZxZ<"svuqadd", "csil",     "dPdu", "dPdL", "aarch64_sve_suqadd", "aarch64_sve_suqadd", [VerifyRuntimeMode]>;

def SVABA_S   : SInst<"svaba[_{d}]",     "dddd", "csil"    , MergeNone, "aarch64_sve_saba", [VerifyRuntimeMode]>;
def SVABA_U   : SInst<"svaba[_{d}]",     "dddd", "UcUsUiUl", MergeNone, "aarch64_sve_uaba", [VerifyRuntimeMode]>;
def SVQDMULH  : SInst<"svqdmulh[_{d}]",  "ddd",  "csil",     MergeNone, "aarch64_sve_sqdmulh", [VerifyRuntimeMode]>;
def SVQRDMULH : SInst<"svqrdmulh[_{d}]", "ddd",  "csil",     MergeNone, "aarch64_sve_sqrdmulh", [VerifyRuntimeMode]>;
def SVQRDMLAH : SInst<"svqrdmlah[_{d}]", "dddd", "csil",     MergeNone, "aarch64_sve_sqrdmlah", [VerifyRuntimeMode]>;
def SVQRDMLSH : SInst<"svqrdmlsh[_{d}]", "dddd", "csil",     MergeNone, "aarch64_sve_sqrdmlsh", [VerifyRuntimeMode]>;

def SVABA_S_N   : SInst<"svaba[_n_{d}]",     "ddda", "csil",     MergeNone, "aarch64_sve_saba", [VerifyRuntimeMode]>;
def SVABA_U_N   : SInst<"svaba[_n_{d}]",     "ddda", "UcUsUiUl", MergeNone, "aarch64_sve_uaba", [VerifyRuntimeMode]>;
def SVQDMULH_N  : SInst<"svqdmulh[_n_{d}]",  "dda",  "csil",     MergeNone, "aarch64_sve_sqdmulh", [VerifyRuntimeMode]>;
def SVQRDMULH_N : SInst<"svqrdmulh[_n_{d}]", "dda",  "csil",     MergeNone, "aarch64_sve_sqrdmulh", [VerifyRuntimeMode]>;
def SVQRDMLAH_N : SInst<"svqrdmlah[_n_{d}]", "ddda", "csil",     MergeNone, "aarch64_sve_sqrdmlah", [VerifyRuntimeMode]>;
def SVQRDMLSH_N : SInst<"svqrdmlsh[_n_{d}]", "ddda", "csil",     MergeNone, "aarch64_sve_sqrdmlsh", [VerifyRuntimeMode]>;

def SVQDMULH_LANE  : SInst<"svqdmulh_lane[_{d}]",  "dddi",  "sil", MergeNone, "aarch64_sve_sqdmulh_lane",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
````
- **L1373 EN**: Declares TableGen def record `_N_Z`.
  **L1373 CN**: 声明 TableGen def 记录 `_N_Z`。
- **L1374 EN**: Closes the current lexical scope or compound statement.
  **L1374 CN**: 结束当前词法作用域或复合语句块。
- **L1375 EN**: Blank line separating nearby declarations or logic blocks.
  **L1375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1376 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1376 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1377 EN**: Declares TableGen defm record `SVQRSHL_S`.
  **L1377 CN**: 声明 TableGen defm 记录 `SVQRSHL_S`。
- **L1378 EN**: Declares TableGen defm record `SVQRSHL_U`.
  **L1378 CN**: 声明 TableGen defm 记录 `SVQRSHL_U`。
- **L1379 EN**: Declares TableGen defm record `SVQSHL_S`.
  **L1379 CN**: 声明 TableGen defm 记录 `SVQSHL_S`。
- **L1380 EN**: Declares TableGen defm record `SVQSHL_U`.
  **L1380 CN**: 声明 TableGen defm 记录 `SVQSHL_U`。
- **L1381 EN**: Declares TableGen defm record `SVRSHL_S`.
  **L1381 CN**: 声明 TableGen defm 记录 `SVRSHL_S`。
- **L1382 EN**: Declares TableGen defm record `SVRSHL_U`.
  **L1382 CN**: 声明 TableGen defm 记录 `SVRSHL_U`。
- **L1383 EN**: Declares TableGen defm record `SVSQADD`.
  **L1383 CN**: 声明 TableGen defm 记录 `SVSQADD`。
- **L1384 EN**: Declares TableGen defm record `SVUQADD`.
  **L1384 CN**: 声明 TableGen defm 记录 `SVUQADD`。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1386 EN**: Declares TableGen def record `SVABA_S`.
  **L1386 CN**: 声明 TableGen def 记录 `SVABA_S`。
- **L1387 EN**: Declares TableGen def record `SVABA_U`.
  **L1387 CN**: 声明 TableGen def 记录 `SVABA_U`。
- **L1388 EN**: Declares TableGen def record `SVQDMULH`.
  **L1388 CN**: 声明 TableGen def 记录 `SVQDMULH`。
- **L1389 EN**: Declares TableGen def record `SVQRDMULH`.
  **L1389 CN**: 声明 TableGen def 记录 `SVQRDMULH`。
- **L1390 EN**: Declares TableGen def record `SVQRDMLAH`.
  **L1390 CN**: 声明 TableGen def 记录 `SVQRDMLAH`。
- **L1391 EN**: Declares TableGen def record `SVQRDMLSH`.
  **L1391 CN**: 声明 TableGen def 记录 `SVQRDMLSH`。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1393 EN**: Declares TableGen def record `SVABA_S_N`.
  **L1393 CN**: 声明 TableGen def 记录 `SVABA_S_N`。
- **L1394 EN**: Declares TableGen def record `SVABA_U_N`.
  **L1394 CN**: 声明 TableGen def 记录 `SVABA_U_N`。
- **L1395 EN**: Declares TableGen def record `SVQDMULH_N`.
  **L1395 CN**: 声明 TableGen def 记录 `SVQDMULH_N`。
- **L1396 EN**: Declares TableGen def record `SVQRDMULH_N`.
  **L1396 CN**: 声明 TableGen def 记录 `SVQRDMULH_N`。
- **L1397 EN**: Declares TableGen def record `SVQRDMLAH_N`.
  **L1397 CN**: 声明 TableGen def 记录 `SVQRDMLAH_N`。
- **L1398 EN**: Declares TableGen def record `SVQRDMLSH_N`.
  **L1398 CN**: 声明 TableGen def 记录 `SVQRDMLSH_N`。
- **L1399 EN**: Blank line separating nearby declarations or logic blocks.
  **L1399 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1400 EN**: Declares TableGen def record `SVQDMULH_LANE`.
  **L1400 CN**: 声明 TableGen def 记录 `SVQDMULH_LANE`。

### Lines 1401-1428

````tablegen
def SVQRDMULH_LANE : SInst<"svqrdmulh_lane[_{d}]", "dddi",  "sil", MergeNone, "aarch64_sve_sqrdmulh_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def SVQRDMLAH_LANE : SInst<"svqrdmlah_lane[_{d}]", "ddddi", "sil", MergeNone, "aarch64_sve_sqrdmlah_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVQRDMLSH_LANE : SInst<"svqrdmlsh_lane[_{d}]", "ddddi", "sil", MergeNone, "aarch64_sve_sqrdmlsh_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;

def SVQSHLU_M  : SInst<"svqshlu[_n_{d}]", "uPdi", "csil",         MergeOp1,  "aarch64_sve_sqshlu", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftLeft,  1>]>;
def SVQSHLU_X  : SInst<"svqshlu[_n_{d}]", "uPdi", "csil",         MergeAny,  "aarch64_sve_sqshlu", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftLeft,  1>]>;
def SVQSHLU_Z  : SInst<"svqshlu[_n_{d}]", "uPdi", "csil",         MergeZero, "aarch64_sve_sqshlu", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftLeft,  1>]>;
def SVRSHR_M_S : SInst<"svrshr[_n_{d}]",  "dPdi", "csil",         MergeOp1,  "aarch64_sve_srshr",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVRSHR_M_U : SInst<"svrshr[_n_{d}]",  "dPdi", "UcUsUiUl",     MergeOp1,  "aarch64_sve_urshr",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVRSHR_X_S : SInst<"svrshr[_n_{d}]",  "dPdi", "csil",         MergeAny,  "aarch64_sve_srshr",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVRSHR_X_U : SInst<"svrshr[_n_{d}]",  "dPdi", "UcUsUiUl",     MergeAny,  "aarch64_sve_urshr",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVRSHR_Z_S : SInst<"svrshr[_n_{d}]",  "dPdi", "csil",         MergeZero, "aarch64_sve_srshr",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVRSHR_Z_U : SInst<"svrshr[_n_{d}]",  "dPdi", "UcUsUiUl",     MergeZero, "aarch64_sve_urshr",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVRSRA_S   : SInst<"svrsra[_n_{d}]",  "dddi", "csil",         MergeNone, "aarch64_sve_srsra",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVRSRA_U   : SInst<"svrsra[_n_{d}]",  "dddi", "UcUsUiUl",     MergeNone, "aarch64_sve_ursra",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVSLI      : SInst<"svsli[_n_{d}]",   "dddi", "csilUcUsUiUl", MergeNone, "aarch64_sve_sli",    [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftLeft,  1>]>;
def SVSRA_S    : SInst<"svsra[_n_{d}]",   "dddi", "csil",         MergeNone, "aarch64_sve_ssra",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVSRA_U    : SInst<"svsra[_n_{d}]",   "dddi", "UcUsUiUl",     MergeNone, "aarch64_sve_usra",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
def SVSRI      : SInst<"svsri[_n_{d}]",   "dddi", "csilUcUsUiUl", MergeNone, "aarch64_sve_sri",    [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Non-widening pairwise arithmetic

multiclass SInstPairwise<string name, string types, string intrinsic, list<FlagType> flags=[]> {
  def _M   : SInst<name # "[_{d}]", "dPdd", types, MergeOp1, intrinsic, flags>;
  def _X   : SInst<name # "[_{d}]", "dPdd", types, MergeAny, intrinsic, flags>;
}
````
- **L1401 EN**: Declares TableGen def record `SVQRDMULH_LANE`.
  **L1401 CN**: 声明 TableGen def 记录 `SVQRDMULH_LANE`。
- **L1402 EN**: Declares TableGen def record `SVQRDMLAH_LANE`.
  **L1402 CN**: 声明 TableGen def 记录 `SVQRDMLAH_LANE`。
- **L1403 EN**: Declares TableGen def record `SVQRDMLSH_LANE`.
  **L1403 CN**: 声明 TableGen def 记录 `SVQRDMLSH_LANE`。
- **L1404 EN**: Blank line separating nearby declarations or logic blocks.
  **L1404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1405 EN**: Declares TableGen def record `SVQSHLU_M`.
  **L1405 CN**: 声明 TableGen def 记录 `SVQSHLU_M`。
- **L1406 EN**: Declares TableGen def record `SVQSHLU_X`.
  **L1406 CN**: 声明 TableGen def 记录 `SVQSHLU_X`。
- **L1407 EN**: Declares TableGen def record `SVQSHLU_Z`.
  **L1407 CN**: 声明 TableGen def 记录 `SVQSHLU_Z`。
- **L1408 EN**: Declares TableGen def record `SVRSHR_M_S`.
  **L1408 CN**: 声明 TableGen def 记录 `SVRSHR_M_S`。
- **L1409 EN**: Declares TableGen def record `SVRSHR_M_U`.
  **L1409 CN**: 声明 TableGen def 记录 `SVRSHR_M_U`。
- **L1410 EN**: Declares TableGen def record `SVRSHR_X_S`.
  **L1410 CN**: 声明 TableGen def 记录 `SVRSHR_X_S`。
- **L1411 EN**: Declares TableGen def record `SVRSHR_X_U`.
  **L1411 CN**: 声明 TableGen def 记录 `SVRSHR_X_U`。
- **L1412 EN**: Declares TableGen def record `SVRSHR_Z_S`.
  **L1412 CN**: 声明 TableGen def 记录 `SVRSHR_Z_S`。
- **L1413 EN**: Declares TableGen def record `SVRSHR_Z_U`.
  **L1413 CN**: 声明 TableGen def 记录 `SVRSHR_Z_U`。
- **L1414 EN**: Declares TableGen def record `SVRSRA_S`.
  **L1414 CN**: 声明 TableGen def 记录 `SVRSRA_S`。
- **L1415 EN**: Declares TableGen def record `SVRSRA_U`.
  **L1415 CN**: 声明 TableGen def 记录 `SVRSRA_U`。
- **L1416 EN**: Declares TableGen def record `SVSLI`.
  **L1416 CN**: 声明 TableGen def 记录 `SVSLI`。
- **L1417 EN**: Declares TableGen def record `SVSRA_S`.
  **L1417 CN**: 声明 TableGen def 记录 `SVSRA_S`。
- **L1418 EN**: Declares TableGen def record `SVSRA_U`.
  **L1418 CN**: 声明 TableGen def 记录 `SVSRA_U`。
- **L1419 EN**: Declares TableGen def record `SVSRI`.
  **L1419 CN**: 声明 TableGen def 记录 `SVSRI`。
- **L1420 EN**: Closes the current lexical scope or compound statement.
  **L1420 CN**: 结束当前词法作用域或复合语句块。
- **L1421 EN**: Blank line separating nearby declarations or logic blocks.
  **L1421 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1422 EN**: Separator comment used for visual grouping.
  **L1422 CN**: 用于视觉分组的分隔注释。
- **L1423 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Non-widening pairwise arithmetic`.
  **L1423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Non-widening pairwise arithmetic`。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Declares TableGen multiclass record `SInstPairwise`.
  **L1425 CN**: 声明 TableGen multiclass 记录 `SInstPairwise`。
- **L1426 EN**: Declares TableGen def record `_M`.
  **L1426 CN**: 声明 TableGen def 记录 `_M`。
- **L1427 EN**: Declares TableGen def record `_X`.
  **L1427 CN**: 声明 TableGen def 记录 `_X`。
- **L1428 EN**: Closes the current lexical scope or compound statement.
  **L1428 CN**: 结束当前词法作用域或复合语句块。

### Lines 1429-1456

````tablegen

let SVETargetGuard = "sve2|sme" in {
defm SVADDP   : SInstPairwise<"svaddp",   "csliUcUsUiUl", "aarch64_sve_addp", [VerifyRuntimeMode]>;
defm SVADDP_F : SInstPairwise<"svaddp",   "hfd",          "aarch64_sve_faddp", [VerifyRuntimeMode]>;
defm SVMAXNMP : SInstPairwise<"svmaxnmp", "hfd",          "aarch64_sve_fmaxnmp", [VerifyRuntimeMode]>;
defm SVMAXP_F : SInstPairwise<"svmaxp",   "hfd",          "aarch64_sve_fmaxp", [VerifyRuntimeMode]>;
defm SVMAXP_S : SInstPairwise<"svmaxp",   "csli",         "aarch64_sve_smaxp", [VerifyRuntimeMode]>;
defm SVMAXP_U : SInstPairwise<"svmaxp",   "UcUsUiUl",     "aarch64_sve_umaxp", [VerifyRuntimeMode]>;
defm SVMINNMP : SInstPairwise<"svminnmp", "hfd",          "aarch64_sve_fminnmp", [VerifyRuntimeMode]>;
defm SVMINP_F : SInstPairwise<"svminp",   "hfd",          "aarch64_sve_fminp", [VerifyRuntimeMode]>;
defm SVMINP_S : SInstPairwise<"svminp",   "csli",         "aarch64_sve_sminp", [VerifyRuntimeMode]>;
defm SVMINP_U : SInstPairwise<"svminp",   "UcUsUiUl",     "aarch64_sve_uminp", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2.3 - Add pairwise within quadword vector segments

let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {
def SVADDQP   : SInst<"svaddqp[_{d}]", "ddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_addqp",
                    [VerifyRuntimeMode]>;
def SVADDSUBP : SInst<"svaddsubp[_{d}]", "ddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_addsubp",
                    [VerifyRuntimeMode]>;
def SVSUBP_M  : SInst<"svsubp[_{d}]", "dPdd", "csilUcUsUiUl", MergeOp1,  "aarch64_sve_subp", [VerifyRuntimeMode]>;
def SVSUBP_X  : SInst<"svsubp[_{d}]", "dPdd", "csilUcUsUiUl", MergeAny,  "aarch64_sve_subp", [VerifyRuntimeMode]>;
def SVSUBP_Z  : SInst<"svsubp[_{d}]", "dPdd", "csilUcUsUiUl", MergeZero, "aarch64_sve_subp", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
````
- **L1429 EN**: Blank line separating nearby declarations or logic blocks.
  **L1429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1430 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1430 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1431 EN**: Declares TableGen defm record `SVADDP`.
  **L1431 CN**: 声明 TableGen defm 记录 `SVADDP`。
- **L1432 EN**: Declares TableGen defm record `SVADDP_F`.
  **L1432 CN**: 声明 TableGen defm 记录 `SVADDP_F`。
- **L1433 EN**: Declares TableGen defm record `SVMAXNMP`.
  **L1433 CN**: 声明 TableGen defm 记录 `SVMAXNMP`。
- **L1434 EN**: Declares TableGen defm record `SVMAXP_F`.
  **L1434 CN**: 声明 TableGen defm 记录 `SVMAXP_F`。
- **L1435 EN**: Declares TableGen defm record `SVMAXP_S`.
  **L1435 CN**: 声明 TableGen defm 记录 `SVMAXP_S`。
- **L1436 EN**: Declares TableGen defm record `SVMAXP_U`.
  **L1436 CN**: 声明 TableGen defm 记录 `SVMAXP_U`。
- **L1437 EN**: Declares TableGen defm record `SVMINNMP`.
  **L1437 CN**: 声明 TableGen defm 记录 `SVMINNMP`。
- **L1438 EN**: Declares TableGen defm record `SVMINP_F`.
  **L1438 CN**: 声明 TableGen defm 记录 `SVMINP_F`。
- **L1439 EN**: Declares TableGen defm record `SVMINP_S`.
  **L1439 CN**: 声明 TableGen defm 记录 `SVMINP_S`。
- **L1440 EN**: Declares TableGen defm record `SVMINP_U`.
  **L1440 CN**: 声明 TableGen defm 记录 `SVMINP_U`。
- **L1441 EN**: Closes the current lexical scope or compound statement.
  **L1441 CN**: 结束当前词法作用域或复合语句块。
- **L1442 EN**: Blank line separating nearby declarations or logic blocks.
  **L1442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1443 EN**: Separator comment used for visual grouping.
  **L1443 CN**: 用于视觉分组的分隔注释。
- **L1444 EN**: Comment explains nearby logic, constraints, or intent: `SVE2.3 - Add pairwise within quadword vector segments`.
  **L1444 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2.3 - Add pairwise within quadword vector segments`。
- **L1445 EN**: Blank line separating nearby declarations or logic blocks.
  **L1445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1446 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`.
  **L1446 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`。
- **L1447 EN**: Declares TableGen def record `SVADDQP`.
  **L1447 CN**: 声明 TableGen def 记录 `SVADDQP`。
- **L1448 EN**: Adds a standalone statement or declaration: `[VerifyRuntimeMode]>;`.
  **L1448 CN**: 添加一条独立语句或声明：`[VerifyRuntimeMode]>;`。
- **L1449 EN**: Declares TableGen def record `SVADDSUBP`.
  **L1449 CN**: 声明 TableGen def 记录 `SVADDSUBP`。
- **L1450 EN**: Adds a standalone statement or declaration: `[VerifyRuntimeMode]>;`.
  **L1450 CN**: 添加一条独立语句或声明：`[VerifyRuntimeMode]>;`。
- **L1451 EN**: Declares TableGen def record `SVSUBP_M`.
  **L1451 CN**: 声明 TableGen def 记录 `SVSUBP_M`。
- **L1452 EN**: Declares TableGen def record `SVSUBP_X`.
  **L1452 CN**: 声明 TableGen def 记录 `SVSUBP_X`。
- **L1453 EN**: Declares TableGen def record `SVSUBP_Z`.
  **L1453 CN**: 声明 TableGen def 记录 `SVSUBP_Z`。
- **L1454 EN**: Closes the current lexical scope or compound statement.
  **L1454 CN**: 结束当前词法作用域或复合语句块。
- **L1455 EN**: Blank line separating nearby declarations or logic blocks.
  **L1455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1456 EN**: Separator comment used for visual grouping.
  **L1456 CN**: 用于视觉分组的分隔注释。

### Lines 1457-1484

````tablegen
// SVE2 - Widening pairwise arithmetic

let SVETargetGuard = "sve2|sme" in {
def SVADALP_S_M : SInst<"svadalp[_{d}]", "dPdh", "sil", MergeOp1,  "aarch64_sve_sadalp", [VerifyRuntimeMode]>;
def SVADALP_S_X : SInst<"svadalp[_{d}]", "dPdh", "sil", MergeAny,  "aarch64_sve_sadalp", [VerifyRuntimeMode]>;
def SVADALP_S_Z : SInst<"svadalp[_{d}]", "dPdh", "sil", MergeZero, "aarch64_sve_sadalp", [VerifyRuntimeMode]>;

def SVADALP_U_M : SInst<"svadalp[_{d}]", "dPdh", "UsUiUl", MergeOp1,  "aarch64_sve_uadalp", [VerifyRuntimeMode]>;
def SVADALP_U_X : SInst<"svadalp[_{d}]", "dPdh", "UsUiUl", MergeAny,  "aarch64_sve_uadalp", [VerifyRuntimeMode]>;
def SVADALP_U_Z : SInst<"svadalp[_{d}]", "dPdh", "UsUiUl", MergeZero, "aarch64_sve_uadalp", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Bitwise ternary logical instructions
//

let SVETargetGuard = "sve2|sme" in {
def SVBCAX  : SInst<"svbcax[_{d}]",  "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_bcax", [VerifyRuntimeMode]>;
def SVBSL   : SInst<"svbsl[_{d}]",   "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_bsl", [VerifyRuntimeMode]>;
def SVBSL1N : SInst<"svbsl1n[_{d}]", "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_bsl1n", [VerifyRuntimeMode]>;
def SVBSL2N : SInst<"svbsl2n[_{d}]", "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_bsl2n", [VerifyRuntimeMode]>;
def SVEOR3  : SInst<"sveor3[_{d}]",  "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_eor3", [VerifyRuntimeMode]>;
def SVNBSL  : SInst<"svnbsl[_{d}]",  "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_nbsl", [VerifyRuntimeMode]>;

def SVBCAX_N  : SInst<"svbcax[_n_{d}]",  "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_bcax", [VerifyRuntimeMode]>;
def SVBSL_N   : SInst<"svbsl[_n_{d}]",   "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_bsl", [VerifyRuntimeMode]>;
def SVBSL1N_N : SInst<"svbsl1n[_n_{d}]", "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_bsl1n", [VerifyRuntimeMode]>;
def SVBSL2N_N : SInst<"svbsl2n[_n_{d}]", "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_bsl2n", [VerifyRuntimeMode]>;
````
- **L1457 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Widening pairwise arithmetic`.
  **L1457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Widening pairwise arithmetic`。
- **L1458 EN**: Blank line separating nearby declarations or logic blocks.
  **L1458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1459 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1459 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1460 EN**: Declares TableGen def record `SVADALP_S_M`.
  **L1460 CN**: 声明 TableGen def 记录 `SVADALP_S_M`。
- **L1461 EN**: Declares TableGen def record `SVADALP_S_X`.
  **L1461 CN**: 声明 TableGen def 记录 `SVADALP_S_X`。
- **L1462 EN**: Declares TableGen def record `SVADALP_S_Z`.
  **L1462 CN**: 声明 TableGen def 记录 `SVADALP_S_Z`。
- **L1463 EN**: Blank line separating nearby declarations or logic blocks.
  **L1463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1464 EN**: Declares TableGen def record `SVADALP_U_M`.
  **L1464 CN**: 声明 TableGen def 记录 `SVADALP_U_M`。
- **L1465 EN**: Declares TableGen def record `SVADALP_U_X`.
  **L1465 CN**: 声明 TableGen def 记录 `SVADALP_U_X`。
- **L1466 EN**: Declares TableGen def record `SVADALP_U_Z`.
  **L1466 CN**: 声明 TableGen def 记录 `SVADALP_U_Z`。
- **L1467 EN**: Closes the current lexical scope or compound statement.
  **L1467 CN**: 结束当前词法作用域或复合语句块。
- **L1468 EN**: Blank line separating nearby declarations or logic blocks.
  **L1468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1469 EN**: Separator comment used for visual grouping.
  **L1469 CN**: 用于视觉分组的分隔注释。
- **L1470 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Bitwise ternary logical instructions`.
  **L1470 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Bitwise ternary logical instructions`。
- **L1471 EN**: Separator comment used for visual grouping.
  **L1471 CN**: 用于视觉分组的分隔注释。
- **L1472 EN**: Blank line separating nearby declarations or logic blocks.
  **L1472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1473 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1473 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1474 EN**: Declares TableGen def record `SVBCAX`.
  **L1474 CN**: 声明 TableGen def 记录 `SVBCAX`。
- **L1475 EN**: Declares TableGen def record `SVBSL`.
  **L1475 CN**: 声明 TableGen def 记录 `SVBSL`。
- **L1476 EN**: Declares TableGen def record `SVBSL1N`.
  **L1476 CN**: 声明 TableGen def 记录 `SVBSL1N`。
- **L1477 EN**: Declares TableGen def record `SVBSL2N`.
  **L1477 CN**: 声明 TableGen def 记录 `SVBSL2N`。
- **L1478 EN**: Declares TableGen def record `SVEOR3`.
  **L1478 CN**: 声明 TableGen def 记录 `SVEOR3`。
- **L1479 EN**: Declares TableGen def record `SVNBSL`.
  **L1479 CN**: 声明 TableGen def 记录 `SVNBSL`。
- **L1480 EN**: Blank line separating nearby declarations or logic blocks.
  **L1480 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1481 EN**: Declares TableGen def record `SVBCAX_N`.
  **L1481 CN**: 声明 TableGen def 记录 `SVBCAX_N`。
- **L1482 EN**: Declares TableGen def record `SVBSL_N`.
  **L1482 CN**: 声明 TableGen def 记录 `SVBSL_N`。
- **L1483 EN**: Declares TableGen def record `SVBSL1N_N`.
  **L1483 CN**: 声明 TableGen def 记录 `SVBSL1N_N`。
- **L1484 EN**: Declares TableGen def record `SVBSL2N_N`.
  **L1484 CN**: 声明 TableGen def 记录 `SVBSL2N_N`。

### Lines 1485-1512

````tablegen
def SVEOR3_N  : SInst<"sveor3[_n_{d}]",  "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_eor3", [VerifyRuntimeMode]>;
def SVNBSL_N  : SInst<"svnbsl[_n_{d}]",  "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_nbsl", [VerifyRuntimeMode]>;
def SVXAR_N   : SInst<"svxar[_n_{d}]",   "dddi", "csilUcUsUiUl", MergeNone, "aarch64_sve_xar", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRight, 1>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Large integer arithmetic

let SVETargetGuard = "sve2|sme" in {
def SVADCLB : SInst<"svadclb[_{d}]", "dddd", "UiUl", MergeNone, "aarch64_sve_adclb", [VerifyRuntimeMode]>;
def SVADCLT : SInst<"svadclt[_{d}]", "dddd", "UiUl", MergeNone, "aarch64_sve_adclt", [VerifyRuntimeMode]>;
def SVSBCLB : SInst<"svsbclb[_{d}]", "dddd", "UiUl", MergeNone, "aarch64_sve_sbclb", [VerifyRuntimeMode]>;
def SVSBCLT : SInst<"svsbclt[_{d}]", "dddd", "UiUl", MergeNone, "aarch64_sve_sbclt", [VerifyRuntimeMode]>;

def SVADCLB_N : SInst<"svadclb[_n_{d}]", "ddda", "UiUl", MergeNone, "aarch64_sve_adclb", [VerifyRuntimeMode]>;
def SVADCLT_N : SInst<"svadclt[_n_{d}]", "ddda", "UiUl", MergeNone, "aarch64_sve_adclt", [VerifyRuntimeMode]>;
def SVSBCLB_N : SInst<"svsbclb[_n_{d}]", "ddda", "UiUl", MergeNone, "aarch64_sve_sbclb", [VerifyRuntimeMode]>;
def SVSBCLT_N : SInst<"svsbclt[_n_{d}]", "ddda", "UiUl", MergeNone, "aarch64_sve_sbclt", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Multiplication by indexed elements

let SVETargetGuard = "sve2|sme" in {
def SVMLA_LANE_2 : SInst<"svmla_lane[_{d}]", "ddddi", "silUsUiUl", MergeNone, "aarch64_sve_mla_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLS_LANE_2 : SInst<"svmls_lane[_{d}]", "ddddi", "silUsUiUl", MergeNone, "aarch64_sve_mls_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMUL_LANE_2 : SInst<"svmul_lane[_{d}]", "dddi",  "silUsUiUl", MergeNone, "aarch64_sve_mul_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
}
````
- **L1485 EN**: Declares TableGen def record `SVEOR3_N`.
  **L1485 CN**: 声明 TableGen def 记录 `SVEOR3_N`。
- **L1486 EN**: Declares TableGen def record `SVNBSL_N`.
  **L1486 CN**: 声明 TableGen def 记录 `SVNBSL_N`。
- **L1487 EN**: Declares TableGen def record `SVXAR_N`.
  **L1487 CN**: 声明 TableGen def 记录 `SVXAR_N`。
- **L1488 EN**: Closes the current lexical scope or compound statement.
  **L1488 CN**: 结束当前词法作用域或复合语句块。
- **L1489 EN**: Blank line separating nearby declarations or logic blocks.
  **L1489 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1490 EN**: Separator comment used for visual grouping.
  **L1490 CN**: 用于视觉分组的分隔注释。
- **L1491 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Large integer arithmetic`.
  **L1491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Large integer arithmetic`。
- **L1492 EN**: Blank line separating nearby declarations or logic blocks.
  **L1492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1493 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1493 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1494 EN**: Declares TableGen def record `SVADCLB`.
  **L1494 CN**: 声明 TableGen def 记录 `SVADCLB`。
- **L1495 EN**: Declares TableGen def record `SVADCLT`.
  **L1495 CN**: 声明 TableGen def 记录 `SVADCLT`。
- **L1496 EN**: Declares TableGen def record `SVSBCLB`.
  **L1496 CN**: 声明 TableGen def 记录 `SVSBCLB`。
- **L1497 EN**: Declares TableGen def record `SVSBCLT`.
  **L1497 CN**: 声明 TableGen def 记录 `SVSBCLT`。
- **L1498 EN**: Blank line separating nearby declarations or logic blocks.
  **L1498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1499 EN**: Declares TableGen def record `SVADCLB_N`.
  **L1499 CN**: 声明 TableGen def 记录 `SVADCLB_N`。
- **L1500 EN**: Declares TableGen def record `SVADCLT_N`.
  **L1500 CN**: 声明 TableGen def 记录 `SVADCLT_N`。
- **L1501 EN**: Declares TableGen def record `SVSBCLB_N`.
  **L1501 CN**: 声明 TableGen def 记录 `SVSBCLB_N`。
- **L1502 EN**: Declares TableGen def record `SVSBCLT_N`.
  **L1502 CN**: 声明 TableGen def 记录 `SVSBCLT_N`。
- **L1503 EN**: Closes the current lexical scope or compound statement.
  **L1503 CN**: 结束当前词法作用域或复合语句块。
- **L1504 EN**: Blank line separating nearby declarations or logic blocks.
  **L1504 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1505 EN**: Separator comment used for visual grouping.
  **L1505 CN**: 用于视觉分组的分隔注释。
- **L1506 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Multiplication by indexed elements`.
  **L1506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Multiplication by indexed elements`。
- **L1507 EN**: Blank line separating nearby declarations or logic blocks.
  **L1507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1508 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1508 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1509 EN**: Declares TableGen def record `SVMLA_LANE_2`.
  **L1509 CN**: 声明 TableGen def 记录 `SVMLA_LANE_2`。
- **L1510 EN**: Declares TableGen def record `SVMLS_LANE_2`.
  **L1510 CN**: 声明 TableGen def 记录 `SVMLS_LANE_2`。
- **L1511 EN**: Declares TableGen def record `SVMUL_LANE_2`.
  **L1511 CN**: 声明 TableGen def 记录 `SVMUL_LANE_2`。
- **L1512 EN**: Closes the current lexical scope or compound statement.
  **L1512 CN**: 结束当前词法作用域或复合语句块。

### Lines 1513-1540

````tablegen

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Uniform complex integer arithmetic
let SVETargetGuard = "sve2|sme" in {
def SVCADD             : SInst<"svcadd[_{d}]",          "dddi",   "csilUcUsUiUl", MergeNone, "aarch64_sve_cadd_x",           [VerifyRuntimeMode], [ImmCheck<2, ImmCheckComplexRot90_270>]>;
def SVSQCADD           : SInst<"svqcadd[_{d}]",         "dddi",   "csil",         MergeNone, "aarch64_sve_sqcadd_x",         [VerifyRuntimeMode], [ImmCheck<2, ImmCheckComplexRot90_270>]>;
def SVCMLA             : SInst<"svcmla[_{d}]",          "ddddi",  "csilUcUsUiUl", MergeNone, "aarch64_sve_cmla_x",           [VerifyRuntimeMode], [ImmCheck<3, ImmCheckComplexRotAll90>]>;
def SVCMLA_LANE_X      : SInst<"svcmla_lane[_{d}]",     "ddddii", "siUsUi",       MergeNone, "aarch64_sve_cmla_lane_x",      [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndexCompRotate, 2>, ImmCheck<4, ImmCheckComplexRotAll90>]>;
def SVSQRDCMLAH_X      : SInst<"svqrdcmlah[_{d}]",      "ddddi",  "csil",         MergeNone, "aarch64_sve_sqrdcmlah_x",      [VerifyRuntimeMode], [ImmCheck<3, ImmCheckComplexRotAll90>]>;
def SVSQRDCMLAH_LANE_X : SInst<"svqrdcmlah_lane[_{d}]", "ddddii", "si",           MergeNone, "aarch64_sve_sqrdcmlah_lane_x", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndexCompRotate, 2>, ImmCheck<4, ImmCheckComplexRotAll90>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Widening DSP operations

multiclass SInstWideDSPAcc<string name, string types, string intrinsic> {
  def    : SInst<name # "[_{d}]",   "ddhh", types, MergeNone, intrinsic, [VerifyRuntimeMode]>;
  def _N : SInst<name # "[_n_{d}]", "ddhR", types, MergeNone, intrinsic, [VerifyRuntimeMode]>;
}

multiclass SInstWideDSPLong<string name, string types, string intrinsic> {
  def    : SInst<name # "[_{d}]",   "dhh", types, MergeNone, intrinsic, [VerifyRuntimeMode]>;
  def _N : SInst<name # "[_n_{d}]", "dhR", types, MergeNone, intrinsic, [VerifyRuntimeMode]>;
}

multiclass SInstWideDSPWide<string name, string types, string intrinsic> {
  def    : SInst<name # "[_{d}]",   "ddh", types, MergeNone, intrinsic, [VerifyRuntimeMode]>;
  def _N : SInst<name # "[_n_{d}]", "ddR", types, MergeNone, intrinsic, [VerifyRuntimeMode]>;
````
- **L1513 EN**: Blank line separating nearby declarations or logic blocks.
  **L1513 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1514 EN**: Separator comment used for visual grouping.
  **L1514 CN**: 用于视觉分组的分隔注释。
- **L1515 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Uniform complex integer arithmetic`.
  **L1515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Uniform complex integer arithmetic`。
- **L1516 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1516 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1517 EN**: Declares TableGen def record `SVCADD`.
  **L1517 CN**: 声明 TableGen def 记录 `SVCADD`。
- **L1518 EN**: Declares TableGen def record `SVSQCADD`.
  **L1518 CN**: 声明 TableGen def 记录 `SVSQCADD`。
- **L1519 EN**: Declares TableGen def record `SVCMLA`.
  **L1519 CN**: 声明 TableGen def 记录 `SVCMLA`。
- **L1520 EN**: Declares TableGen def record `SVCMLA_LANE_X`.
  **L1520 CN**: 声明 TableGen def 记录 `SVCMLA_LANE_X`。
- **L1521 EN**: Declares TableGen def record `SVSQRDCMLAH_X`.
  **L1521 CN**: 声明 TableGen def 记录 `SVSQRDCMLAH_X`。
- **L1522 EN**: Declares TableGen def record `SVSQRDCMLAH_LANE_X`.
  **L1522 CN**: 声明 TableGen def 记录 `SVSQRDCMLAH_LANE_X`。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Blank line separating nearby declarations or logic blocks.
  **L1524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1525 EN**: Separator comment used for visual grouping.
  **L1525 CN**: 用于视觉分组的分隔注释。
- **L1526 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Widening DSP operations`.
  **L1526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Widening DSP operations`。
- **L1527 EN**: Blank line separating nearby declarations or logic blocks.
  **L1527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1528 EN**: Declares TableGen multiclass record `SInstWideDSPAcc`.
  **L1528 CN**: 声明 TableGen multiclass 记录 `SInstWideDSPAcc`。
- **L1529 EN**: Declares TableGen def record `def`.
  **L1529 CN**: 声明 TableGen def 记录 `def`。
- **L1530 EN**: Declares TableGen def record `_N`.
  **L1530 CN**: 声明 TableGen def 记录 `_N`。
- **L1531 EN**: Closes the current lexical scope or compound statement.
  **L1531 CN**: 结束当前词法作用域或复合语句块。
- **L1532 EN**: Blank line separating nearby declarations or logic blocks.
  **L1532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1533 EN**: Declares TableGen multiclass record `SInstWideDSPLong`.
  **L1533 CN**: 声明 TableGen multiclass 记录 `SInstWideDSPLong`。
- **L1534 EN**: Declares TableGen def record `def`.
  **L1534 CN**: 声明 TableGen def 记录 `def`。
- **L1535 EN**: Declares TableGen def record `_N`.
  **L1535 CN**: 声明 TableGen def 记录 `_N`。
- **L1536 EN**: Closes the current lexical scope or compound statement.
  **L1536 CN**: 结束当前词法作用域或复合语句块。
- **L1537 EN**: Blank line separating nearby declarations or logic blocks.
  **L1537 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1538 EN**: Declares TableGen multiclass record `SInstWideDSPWide`.
  **L1538 CN**: 声明 TableGen multiclass 记录 `SInstWideDSPWide`。
- **L1539 EN**: Declares TableGen def record `def`.
  **L1539 CN**: 声明 TableGen def 记录 `def`。
- **L1540 EN**: Declares TableGen def record `_N`.
  **L1540 CN**: 声明 TableGen def 记录 `_N`。

### Lines 1541-1568

````tablegen
}

let SVETargetGuard = "sve2|sme" in {
defm SVABALB_S : SInstWideDSPAcc<"svabalb",   "sil",    "aarch64_sve_sabalb">;
defm SVABALB_U : SInstWideDSPAcc<"svabalb",   "UsUiUl", "aarch64_sve_uabalb">;
defm SVABALT_S : SInstWideDSPAcc<"svabalt",   "sil",    "aarch64_sve_sabalt">;
defm SVABALT_U : SInstWideDSPAcc<"svabalt",   "UsUiUl", "aarch64_sve_uabalt">;
defm SVMLALB_S : SInstWideDSPAcc<"svmlalb",   "sil",    "aarch64_sve_smlalb">;
defm SVMLALB_U : SInstWideDSPAcc<"svmlalb",   "UsUiUl", "aarch64_sve_umlalb">;
defm SVMLALT_S : SInstWideDSPAcc<"svmlalt",   "sil",    "aarch64_sve_smlalt">;
defm SVMLALT_U : SInstWideDSPAcc<"svmlalt",   "UsUiUl", "aarch64_sve_umlalt">;
defm SVMLSLB_S : SInstWideDSPAcc<"svmlslb",   "sil",    "aarch64_sve_smlslb">;
defm SVMLSLB_U : SInstWideDSPAcc<"svmlslb",   "UsUiUl", "aarch64_sve_umlslb">;
defm SVMLSLT_S : SInstWideDSPAcc<"svmlslt",   "sil",    "aarch64_sve_smlslt">;
defm SVMLSLT_U : SInstWideDSPAcc<"svmlslt",   "UsUiUl", "aarch64_sve_umlslt">;
defm SVQDMLALB : SInstWideDSPAcc<"svqdmlalb", "sil",    "aarch64_sve_sqdmlalb">;
defm SVQDMLALT : SInstWideDSPAcc<"svqdmlalt", "sil",    "aarch64_sve_sqdmlalt">;
defm SVQDMLSLB : SInstWideDSPAcc<"svqdmlslb", "sil",    "aarch64_sve_sqdmlslb">;
defm SVQDMLSLT : SInstWideDSPAcc<"svqdmlslt", "sil",    "aarch64_sve_sqdmlslt">;

defm SVABDLB_S : SInstWideDSPLong<"svabdlb",   "sil",    "aarch64_sve_sabdlb">;
defm SVABDLB_U : SInstWideDSPLong<"svabdlb",   "UsUiUl", "aarch64_sve_uabdlb">;
defm SVABDLT_S : SInstWideDSPLong<"svabdlt",   "sil",    "aarch64_sve_sabdlt">;
defm SVABDLT_U : SInstWideDSPLong<"svabdlt",   "UsUiUl", "aarch64_sve_uabdlt">;
defm SVADDLB_S : SInstWideDSPLong<"svaddlb",   "sil",    "aarch64_sve_saddlb">;
defm SVADDLB_U : SInstWideDSPLong<"svaddlb",   "UsUiUl", "aarch64_sve_uaddlb">;
defm SVADDLT_S : SInstWideDSPLong<"svaddlt",   "sil",    "aarch64_sve_saddlt">;
defm SVADDLT_U : SInstWideDSPLong<"svaddlt",   "UsUiUl", "aarch64_sve_uaddlt">;
````
- **L1541 EN**: Closes the current lexical scope or compound statement.
  **L1541 CN**: 结束当前词法作用域或复合语句块。
- **L1542 EN**: Blank line separating nearby declarations or logic blocks.
  **L1542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1543 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1543 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1544 EN**: Declares TableGen defm record `SVABALB_S`.
  **L1544 CN**: 声明 TableGen defm 记录 `SVABALB_S`。
- **L1545 EN**: Declares TableGen defm record `SVABALB_U`.
  **L1545 CN**: 声明 TableGen defm 记录 `SVABALB_U`。
- **L1546 EN**: Declares TableGen defm record `SVABALT_S`.
  **L1546 CN**: 声明 TableGen defm 记录 `SVABALT_S`。
- **L1547 EN**: Declares TableGen defm record `SVABALT_U`.
  **L1547 CN**: 声明 TableGen defm 记录 `SVABALT_U`。
- **L1548 EN**: Declares TableGen defm record `SVMLALB_S`.
  **L1548 CN**: 声明 TableGen defm 记录 `SVMLALB_S`。
- **L1549 EN**: Declares TableGen defm record `SVMLALB_U`.
  **L1549 CN**: 声明 TableGen defm 记录 `SVMLALB_U`。
- **L1550 EN**: Declares TableGen defm record `SVMLALT_S`.
  **L1550 CN**: 声明 TableGen defm 记录 `SVMLALT_S`。
- **L1551 EN**: Declares TableGen defm record `SVMLALT_U`.
  **L1551 CN**: 声明 TableGen defm 记录 `SVMLALT_U`。
- **L1552 EN**: Declares TableGen defm record `SVMLSLB_S`.
  **L1552 CN**: 声明 TableGen defm 记录 `SVMLSLB_S`。
- **L1553 EN**: Declares TableGen defm record `SVMLSLB_U`.
  **L1553 CN**: 声明 TableGen defm 记录 `SVMLSLB_U`。
- **L1554 EN**: Declares TableGen defm record `SVMLSLT_S`.
  **L1554 CN**: 声明 TableGen defm 记录 `SVMLSLT_S`。
- **L1555 EN**: Declares TableGen defm record `SVMLSLT_U`.
  **L1555 CN**: 声明 TableGen defm 记录 `SVMLSLT_U`。
- **L1556 EN**: Declares TableGen defm record `SVQDMLALB`.
  **L1556 CN**: 声明 TableGen defm 记录 `SVQDMLALB`。
- **L1557 EN**: Declares TableGen defm record `SVQDMLALT`.
  **L1557 CN**: 声明 TableGen defm 记录 `SVQDMLALT`。
- **L1558 EN**: Declares TableGen defm record `SVQDMLSLB`.
  **L1558 CN**: 声明 TableGen defm 记录 `SVQDMLSLB`。
- **L1559 EN**: Declares TableGen defm record `SVQDMLSLT`.
  **L1559 CN**: 声明 TableGen defm 记录 `SVQDMLSLT`。
- **L1560 EN**: Blank line separating nearby declarations or logic blocks.
  **L1560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1561 EN**: Declares TableGen defm record `SVABDLB_S`.
  **L1561 CN**: 声明 TableGen defm 记录 `SVABDLB_S`。
- **L1562 EN**: Declares TableGen defm record `SVABDLB_U`.
  **L1562 CN**: 声明 TableGen defm 记录 `SVABDLB_U`。
- **L1563 EN**: Declares TableGen defm record `SVABDLT_S`.
  **L1563 CN**: 声明 TableGen defm 记录 `SVABDLT_S`。
- **L1564 EN**: Declares TableGen defm record `SVABDLT_U`.
  **L1564 CN**: 声明 TableGen defm 记录 `SVABDLT_U`。
- **L1565 EN**: Declares TableGen defm record `SVADDLB_S`.
  **L1565 CN**: 声明 TableGen defm 记录 `SVADDLB_S`。
- **L1566 EN**: Declares TableGen defm record `SVADDLB_U`.
  **L1566 CN**: 声明 TableGen defm 记录 `SVADDLB_U`。
- **L1567 EN**: Declares TableGen defm record `SVADDLT_S`.
  **L1567 CN**: 声明 TableGen defm 记录 `SVADDLT_S`。
- **L1568 EN**: Declares TableGen defm record `SVADDLT_U`.
  **L1568 CN**: 声明 TableGen defm 记录 `SVADDLT_U`。

### Lines 1569-1596

````tablegen
defm SVMULLB_S : SInstWideDSPLong<"svmullb",   "sil",    "aarch64_sve_smullb">;
defm SVMULLB_U : SInstWideDSPLong<"svmullb",   "UsUiUl", "aarch64_sve_umullb">;
defm SVMULLT_S : SInstWideDSPLong<"svmullt",   "sil",    "aarch64_sve_smullt">;
defm SVMULLT_U : SInstWideDSPLong<"svmullt",   "UsUiUl", "aarch64_sve_umullt">;
defm SVQDMULLB : SInstWideDSPLong<"svqdmullb", "sil",    "aarch64_sve_sqdmullb">;
defm SVQDMULLT : SInstWideDSPLong<"svqdmullt", "sil",    "aarch64_sve_sqdmullt">;
defm SVSUBLB_S : SInstWideDSPLong<"svsublb",   "sil",    "aarch64_sve_ssublb">;
defm SVSUBLB_U : SInstWideDSPLong<"svsublb",   "UsUiUl", "aarch64_sve_usublb">;
defm SVSUBLT_S : SInstWideDSPLong<"svsublt",   "sil",    "aarch64_sve_ssublt">;
defm SVSUBLT_U : SInstWideDSPLong<"svsublt",   "UsUiUl", "aarch64_sve_usublt">;

defm SVADDWB_S : SInstWideDSPWide<"svaddwb", "sil",    "aarch64_sve_saddwb">;
defm SVADDWB_U : SInstWideDSPWide<"svaddwb", "UsUiUl", "aarch64_sve_uaddwb">;
defm SVADDWT_S : SInstWideDSPWide<"svaddwt", "sil",    "aarch64_sve_saddwt">;
defm SVADDWT_U : SInstWideDSPWide<"svaddwt", "UsUiUl", "aarch64_sve_uaddwt">;
defm SVSUBWB_S : SInstWideDSPWide<"svsubwb", "sil",    "aarch64_sve_ssubwb">;
defm SVSUBWB_U : SInstWideDSPWide<"svsubwb", "UsUiUl", "aarch64_sve_usubwb">;
defm SVSUBWT_S : SInstWideDSPWide<"svsubwt", "sil",    "aarch64_sve_ssubwt">;
defm SVSUBWT_U : SInstWideDSPWide<"svsubwt", "UsUiUl", "aarch64_sve_usubwt">;

def SVSHLLB_S_N : SInst<"svshllb[_n_{d}]", "dhi", "sil",    MergeNone, "aarch64_sve_sshllb", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftLeft,  0>]>;
def SVSHLLB_U_N : SInst<"svshllb[_n_{d}]", "dhi", "UsUiUl", MergeNone, "aarch64_sve_ushllb", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftLeft,  0>]>;
def SVSHLLT_S_N : SInst<"svshllt[_n_{d}]", "dhi", "sil",    MergeNone, "aarch64_sve_sshllt", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftLeft,  0>]>;
def SVSHLLT_U_N : SInst<"svshllt[_n_{d}]", "dhi", "UsUiUl", MergeNone, "aarch64_sve_ushllt", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftLeft,  0>]>;

def SVMOVLB_S_N : SInst<"svmovlb[_{d}]", "dh", "sil",    MergeNone, "", [VerifyRuntimeMode]>;
def SVMOVLB_U_N : SInst<"svmovlb[_{d}]", "dh", "UsUiUl", MergeNone, "", [VerifyRuntimeMode]>;
def SVMOVLT_S_N : SInst<"svmovlt[_{d}]", "dh", "sil",    MergeNone, "", [VerifyRuntimeMode]>;
````
- **L1569 EN**: Declares TableGen defm record `SVMULLB_S`.
  **L1569 CN**: 声明 TableGen defm 记录 `SVMULLB_S`。
- **L1570 EN**: Declares TableGen defm record `SVMULLB_U`.
  **L1570 CN**: 声明 TableGen defm 记录 `SVMULLB_U`。
- **L1571 EN**: Declares TableGen defm record `SVMULLT_S`.
  **L1571 CN**: 声明 TableGen defm 记录 `SVMULLT_S`。
- **L1572 EN**: Declares TableGen defm record `SVMULLT_U`.
  **L1572 CN**: 声明 TableGen defm 记录 `SVMULLT_U`。
- **L1573 EN**: Declares TableGen defm record `SVQDMULLB`.
  **L1573 CN**: 声明 TableGen defm 记录 `SVQDMULLB`。
- **L1574 EN**: Declares TableGen defm record `SVQDMULLT`.
  **L1574 CN**: 声明 TableGen defm 记录 `SVQDMULLT`。
- **L1575 EN**: Declares TableGen defm record `SVSUBLB_S`.
  **L1575 CN**: 声明 TableGen defm 记录 `SVSUBLB_S`。
- **L1576 EN**: Declares TableGen defm record `SVSUBLB_U`.
  **L1576 CN**: 声明 TableGen defm 记录 `SVSUBLB_U`。
- **L1577 EN**: Declares TableGen defm record `SVSUBLT_S`.
  **L1577 CN**: 声明 TableGen defm 记录 `SVSUBLT_S`。
- **L1578 EN**: Declares TableGen defm record `SVSUBLT_U`.
  **L1578 CN**: 声明 TableGen defm 记录 `SVSUBLT_U`。
- **L1579 EN**: Blank line separating nearby declarations or logic blocks.
  **L1579 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1580 EN**: Declares TableGen defm record `SVADDWB_S`.
  **L1580 CN**: 声明 TableGen defm 记录 `SVADDWB_S`。
- **L1581 EN**: Declares TableGen defm record `SVADDWB_U`.
  **L1581 CN**: 声明 TableGen defm 记录 `SVADDWB_U`。
- **L1582 EN**: Declares TableGen defm record `SVADDWT_S`.
  **L1582 CN**: 声明 TableGen defm 记录 `SVADDWT_S`。
- **L1583 EN**: Declares TableGen defm record `SVADDWT_U`.
  **L1583 CN**: 声明 TableGen defm 记录 `SVADDWT_U`。
- **L1584 EN**: Declares TableGen defm record `SVSUBWB_S`.
  **L1584 CN**: 声明 TableGen defm 记录 `SVSUBWB_S`。
- **L1585 EN**: Declares TableGen defm record `SVSUBWB_U`.
  **L1585 CN**: 声明 TableGen defm 记录 `SVSUBWB_U`。
- **L1586 EN**: Declares TableGen defm record `SVSUBWT_S`.
  **L1586 CN**: 声明 TableGen defm 记录 `SVSUBWT_S`。
- **L1587 EN**: Declares TableGen defm record `SVSUBWT_U`.
  **L1587 CN**: 声明 TableGen defm 记录 `SVSUBWT_U`。
- **L1588 EN**: Blank line separating nearby declarations or logic blocks.
  **L1588 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1589 EN**: Declares TableGen def record `SVSHLLB_S_N`.
  **L1589 CN**: 声明 TableGen def 记录 `SVSHLLB_S_N`。
- **L1590 EN**: Declares TableGen def record `SVSHLLB_U_N`.
  **L1590 CN**: 声明 TableGen def 记录 `SVSHLLB_U_N`。
- **L1591 EN**: Declares TableGen def record `SVSHLLT_S_N`.
  **L1591 CN**: 声明 TableGen def 记录 `SVSHLLT_S_N`。
- **L1592 EN**: Declares TableGen def record `SVSHLLT_U_N`.
  **L1592 CN**: 声明 TableGen def 记录 `SVSHLLT_U_N`。
- **L1593 EN**: Blank line separating nearby declarations or logic blocks.
  **L1593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1594 EN**: Declares TableGen def record `SVMOVLB_S_N`.
  **L1594 CN**: 声明 TableGen def 记录 `SVMOVLB_S_N`。
- **L1595 EN**: Declares TableGen def record `SVMOVLB_U_N`.
  **L1595 CN**: 声明 TableGen def 记录 `SVMOVLB_U_N`。
- **L1596 EN**: Declares TableGen def record `SVMOVLT_S_N`.
  **L1596 CN**: 声明 TableGen def 记录 `SVMOVLT_S_N`。

### Lines 1597-1624

````tablegen
def SVMOVLT_U_N : SInst<"svmovlt[_{d}]", "dh", "UsUiUl", MergeNone, "", [VerifyRuntimeMode]>;

def SVMLALB_S_LANE : SInst<"svmlalb_lane[_{d}]",   "ddhhi", "il",   MergeNone, "aarch64_sve_smlalb_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLALB_U_LANE : SInst<"svmlalb_lane[_{d}]",   "ddhhi", "UiUl", MergeNone, "aarch64_sve_umlalb_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLALT_S_LANE : SInst<"svmlalt_lane[_{d}]",   "ddhhi", "il",   MergeNone, "aarch64_sve_smlalt_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLALT_U_LANE : SInst<"svmlalt_lane[_{d}]",   "ddhhi", "UiUl", MergeNone, "aarch64_sve_umlalt_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLSLB_S_LANE : SInst<"svmlslb_lane[_{d}]",   "ddhhi", "il",   MergeNone, "aarch64_sve_smlslb_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLSLB_U_LANE : SInst<"svmlslb_lane[_{d}]",   "ddhhi", "UiUl", MergeNone, "aarch64_sve_umlslb_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLSLT_S_LANE : SInst<"svmlslt_lane[_{d}]",   "ddhhi", "il",   MergeNone, "aarch64_sve_smlslt_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLSLT_U_LANE : SInst<"svmlslt_lane[_{d}]",   "ddhhi", "UiUl", MergeNone, "aarch64_sve_umlslt_lane",   [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMULLB_S_LANE : SInst<"svmullb_lane[_{d}]",   "dhhi",  "il",   MergeNone, "aarch64_sve_smullb_lane",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def SVMULLB_U_LANE : SInst<"svmullb_lane[_{d}]",   "dhhi",  "UiUl", MergeNone, "aarch64_sve_umullb_lane",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def SVMULLT_S_LANE : SInst<"svmullt_lane[_{d}]",   "dhhi",  "il",   MergeNone, "aarch64_sve_smullt_lane",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def SVMULLT_U_LANE : SInst<"svmullt_lane[_{d}]",   "dhhi",  "UiUl", MergeNone, "aarch64_sve_umullt_lane",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def SVQDMLALB_LANE : SInst<"svqdmlalb_lane[_{d}]", "ddhhi", "il",   MergeNone, "aarch64_sve_sqdmlalb_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVQDMLALT_LANE : SInst<"svqdmlalt_lane[_{d}]", "ddhhi", "il",   MergeNone, "aarch64_sve_sqdmlalt_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVQDMLSLB_LANE : SInst<"svqdmlslb_lane[_{d}]", "ddhhi", "il",   MergeNone, "aarch64_sve_sqdmlslb_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVQDMLSLT_LANE : SInst<"svqdmlslt_lane[_{d}]", "ddhhi", "il",   MergeNone, "aarch64_sve_sqdmlslt_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVQDMULLB_LANE : SInst<"svqdmullb_lane[_{d}]", "dhhi",  "il",   MergeNone, "aarch64_sve_sqdmullb_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def SVQDMULLT_LANE : SInst<"svqdmullt_lane[_{d}]", "dhhi",  "il",   MergeNone, "aarch64_sve_sqdmullt_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Narrowing DSP operations

let SVETargetGuard = "sve2|sme" in {
def SVADDHNB  : SInst<"svaddhnb[_{d}]",     "hdd",  "silUsUiUl", MergeNone, "aarch64_sve_addhnb", [VerifyRuntimeMode]>;
def SVADDHNT  : SInst<"svaddhnt[_{d}]",     "hhdd", "silUsUiUl", MergeNone, "aarch64_sve_addhnt", [VerifyRuntimeMode]>;
````
- **L1597 EN**: Declares TableGen def record `SVMOVLT_U_N`.
  **L1597 CN**: 声明 TableGen def 记录 `SVMOVLT_U_N`。
- **L1598 EN**: Blank line separating nearby declarations or logic blocks.
  **L1598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1599 EN**: Declares TableGen def record `SVMLALB_S_LANE`.
  **L1599 CN**: 声明 TableGen def 记录 `SVMLALB_S_LANE`。
- **L1600 EN**: Declares TableGen def record `SVMLALB_U_LANE`.
  **L1600 CN**: 声明 TableGen def 记录 `SVMLALB_U_LANE`。
- **L1601 EN**: Declares TableGen def record `SVMLALT_S_LANE`.
  **L1601 CN**: 声明 TableGen def 记录 `SVMLALT_S_LANE`。
- **L1602 EN**: Declares TableGen def record `SVMLALT_U_LANE`.
  **L1602 CN**: 声明 TableGen def 记录 `SVMLALT_U_LANE`。
- **L1603 EN**: Declares TableGen def record `SVMLSLB_S_LANE`.
  **L1603 CN**: 声明 TableGen def 记录 `SVMLSLB_S_LANE`。
- **L1604 EN**: Declares TableGen def record `SVMLSLB_U_LANE`.
  **L1604 CN**: 声明 TableGen def 记录 `SVMLSLB_U_LANE`。
- **L1605 EN**: Declares TableGen def record `SVMLSLT_S_LANE`.
  **L1605 CN**: 声明 TableGen def 记录 `SVMLSLT_S_LANE`。
- **L1606 EN**: Declares TableGen def record `SVMLSLT_U_LANE`.
  **L1606 CN**: 声明 TableGen def 记录 `SVMLSLT_U_LANE`。
- **L1607 EN**: Declares TableGen def record `SVMULLB_S_LANE`.
  **L1607 CN**: 声明 TableGen def 记录 `SVMULLB_S_LANE`。
- **L1608 EN**: Declares TableGen def record `SVMULLB_U_LANE`.
  **L1608 CN**: 声明 TableGen def 记录 `SVMULLB_U_LANE`。
- **L1609 EN**: Declares TableGen def record `SVMULLT_S_LANE`.
  **L1609 CN**: 声明 TableGen def 记录 `SVMULLT_S_LANE`。
- **L1610 EN**: Declares TableGen def record `SVMULLT_U_LANE`.
  **L1610 CN**: 声明 TableGen def 记录 `SVMULLT_U_LANE`。
- **L1611 EN**: Declares TableGen def record `SVQDMLALB_LANE`.
  **L1611 CN**: 声明 TableGen def 记录 `SVQDMLALB_LANE`。
- **L1612 EN**: Declares TableGen def record `SVQDMLALT_LANE`.
  **L1612 CN**: 声明 TableGen def 记录 `SVQDMLALT_LANE`。
- **L1613 EN**: Declares TableGen def record `SVQDMLSLB_LANE`.
  **L1613 CN**: 声明 TableGen def 记录 `SVQDMLSLB_LANE`。
- **L1614 EN**: Declares TableGen def record `SVQDMLSLT_LANE`.
  **L1614 CN**: 声明 TableGen def 记录 `SVQDMLSLT_LANE`。
- **L1615 EN**: Declares TableGen def record `SVQDMULLB_LANE`.
  **L1615 CN**: 声明 TableGen def 记录 `SVQDMULLB_LANE`。
- **L1616 EN**: Declares TableGen def record `SVQDMULLT_LANE`.
  **L1616 CN**: 声明 TableGen def 记录 `SVQDMULLT_LANE`。
- **L1617 EN**: Closes the current lexical scope or compound statement.
  **L1617 CN**: 结束当前词法作用域或复合语句块。
- **L1618 EN**: Blank line separating nearby declarations or logic blocks.
  **L1618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1619 EN**: Separator comment used for visual grouping.
  **L1619 CN**: 用于视觉分组的分隔注释。
- **L1620 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Narrowing DSP operations`.
  **L1620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Narrowing DSP operations`。
- **L1621 EN**: Blank line separating nearby declarations or logic blocks.
  **L1621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1622 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1622 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1623 EN**: Declares TableGen def record `SVADDHNB`.
  **L1623 CN**: 声明 TableGen def 记录 `SVADDHNB`。
- **L1624 EN**: Declares TableGen def record `SVADDHNT`.
  **L1624 CN**: 声明 TableGen def 记录 `SVADDHNT`。

### Lines 1625-1652

````tablegen
def SVRADDHNB : SInst<"svraddhnb[_{d}]",    "hdd",  "silUsUiUl", MergeNone, "aarch64_sve_raddhnb", [VerifyRuntimeMode]>;
def SVRADDHNT : SInst<"svraddhnt[_{d}]",    "hhdd", "silUsUiUl", MergeNone, "aarch64_sve_raddhnt", [VerifyRuntimeMode]>;
def SVRSUBHNB : SInst<"svrsubhnb[_{d}]",    "hdd",  "silUsUiUl", MergeNone, "aarch64_sve_rsubhnb", [VerifyRuntimeMode]>;
def SVRSUBHNT : SInst<"svrsubhnt[_{d}]",    "hhdd", "silUsUiUl", MergeNone, "aarch64_sve_rsubhnt", [VerifyRuntimeMode]>;
def SVSUBHNB  : SInst<"svsubhnb[_{d}]",     "hdd",  "silUsUiUl", MergeNone, "aarch64_sve_subhnb", [VerifyRuntimeMode]>;
def SVSUBHNT  : SInst<"svsubhnt[_{d}]",     "hhdd", "silUsUiUl", MergeNone, "aarch64_sve_subhnt", [VerifyRuntimeMode]>;

def SVADDHNB_N  : SInst<"svaddhnb[_n_{d}]",  "hda",  "silUsUiUl", MergeNone, "aarch64_sve_addhnb", [VerifyRuntimeMode]>;
def SVADDHNT_N  : SInst<"svaddhnt[_n_{d}]",  "hhda", "silUsUiUl", MergeNone, "aarch64_sve_addhnt", [VerifyRuntimeMode]>;
def SVRADDHNB_N : SInst<"svraddhnb[_n_{d}]", "hda",  "silUsUiUl", MergeNone, "aarch64_sve_raddhnb", [VerifyRuntimeMode]>;
def SVRADDHNT_N : SInst<"svraddhnt[_n_{d}]", "hhda", "silUsUiUl", MergeNone, "aarch64_sve_raddhnt", [VerifyRuntimeMode]>;
def SVRSUBHNB_N : SInst<"svrsubhnb[_n_{d}]", "hda",  "silUsUiUl", MergeNone, "aarch64_sve_rsubhnb", [VerifyRuntimeMode]>;
def SVRSUBHNT_N : SInst<"svrsubhnt[_n_{d}]", "hhda", "silUsUiUl", MergeNone, "aarch64_sve_rsubhnt", [VerifyRuntimeMode]>;
def SVSUBHNB_N  : SInst<"svsubhnb[_n_{d}]",  "hda",  "silUsUiUl", MergeNone, "aarch64_sve_subhnb", [VerifyRuntimeMode]>;
def SVSUBHNT_N  : SInst<"svsubhnt[_n_{d}]",  "hhda", "silUsUiUl", MergeNone, "aarch64_sve_subhnt", [VerifyRuntimeMode]>;

def SVSHRNB     : SInst<"svshrnb[_n_{d}]",    "hdi",  "silUsUiUl", MergeNone, "aarch64_sve_shrnb",     [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
def SVRSHRNB    : SInst<"svrshrnb[_n_{d}]",   "hdi",  "silUsUiUl", MergeNone, "aarch64_sve_rshrnb",    [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
def SVQSHRUNB   : SInst<"svqshrunb[_n_{d}]",  "edi",  "sil",       MergeNone, "aarch64_sve_sqshrunb",  [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
def SVQRSHRUNB  : SInst<"svqrshrunb[_n_{d}]", "edi",  "sil",       MergeNone, "aarch64_sve_sqrshrunb", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
def SVQSHRNB_S  : SInst<"svqshrnb[_n_{d}]",   "hdi",  "sil",       MergeNone, "aarch64_sve_sqshrnb",   [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
def SVQSHRNB_U  : SInst<"svqshrnb[_n_{d}]",   "hdi",  "UsUiUl",    MergeNone, "aarch64_sve_uqshrnb",   [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
def SVQRSHRNB_S : SInst<"svqrshrnb[_n_{d}]",  "hdi",  "sil",       MergeNone, "aarch64_sve_sqrshrnb",  [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
def SVQRSHRNB_U : SInst<"svqrshrnb[_n_{d}]",  "hdi",  "UsUiUl",    MergeNone, "aarch64_sve_uqrshrnb",  [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;

def SVSHRNT     : SInst<"svshrnt[_n_{d}]",    "hhdi", "silUsUiUl", MergeNone, "aarch64_sve_shrnt",     [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
def SVRSHRNT    : SInst<"svrshrnt[_n_{d}]",   "hhdi", "silUsUiUl", MergeNone, "aarch64_sve_rshrnt",    [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
def SVQSHRUNT   : SInst<"svqshrunt[_n_{d}]",  "eedi", "sil",       MergeNone, "aarch64_sve_sqshrunt",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
````
- **L1625 EN**: Declares TableGen def record `SVRADDHNB`.
  **L1625 CN**: 声明 TableGen def 记录 `SVRADDHNB`。
- **L1626 EN**: Declares TableGen def record `SVRADDHNT`.
  **L1626 CN**: 声明 TableGen def 记录 `SVRADDHNT`。
- **L1627 EN**: Declares TableGen def record `SVRSUBHNB`.
  **L1627 CN**: 声明 TableGen def 记录 `SVRSUBHNB`。
- **L1628 EN**: Declares TableGen def record `SVRSUBHNT`.
  **L1628 CN**: 声明 TableGen def 记录 `SVRSUBHNT`。
- **L1629 EN**: Declares TableGen def record `SVSUBHNB`.
  **L1629 CN**: 声明 TableGen def 记录 `SVSUBHNB`。
- **L1630 EN**: Declares TableGen def record `SVSUBHNT`.
  **L1630 CN**: 声明 TableGen def 记录 `SVSUBHNT`。
- **L1631 EN**: Blank line separating nearby declarations or logic blocks.
  **L1631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1632 EN**: Declares TableGen def record `SVADDHNB_N`.
  **L1632 CN**: 声明 TableGen def 记录 `SVADDHNB_N`。
- **L1633 EN**: Declares TableGen def record `SVADDHNT_N`.
  **L1633 CN**: 声明 TableGen def 记录 `SVADDHNT_N`。
- **L1634 EN**: Declares TableGen def record `SVRADDHNB_N`.
  **L1634 CN**: 声明 TableGen def 记录 `SVRADDHNB_N`。
- **L1635 EN**: Declares TableGen def record `SVRADDHNT_N`.
  **L1635 CN**: 声明 TableGen def 记录 `SVRADDHNT_N`。
- **L1636 EN**: Declares TableGen def record `SVRSUBHNB_N`.
  **L1636 CN**: 声明 TableGen def 记录 `SVRSUBHNB_N`。
- **L1637 EN**: Declares TableGen def record `SVRSUBHNT_N`.
  **L1637 CN**: 声明 TableGen def 记录 `SVRSUBHNT_N`。
- **L1638 EN**: Declares TableGen def record `SVSUBHNB_N`.
  **L1638 CN**: 声明 TableGen def 记录 `SVSUBHNB_N`。
- **L1639 EN**: Declares TableGen def record `SVSUBHNT_N`.
  **L1639 CN**: 声明 TableGen def 记录 `SVSUBHNT_N`。
- **L1640 EN**: Blank line separating nearby declarations or logic blocks.
  **L1640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1641 EN**: Declares TableGen def record `SVSHRNB`.
  **L1641 CN**: 声明 TableGen def 记录 `SVSHRNB`。
- **L1642 EN**: Declares TableGen def record `SVRSHRNB`.
  **L1642 CN**: 声明 TableGen def 记录 `SVRSHRNB`。
- **L1643 EN**: Declares TableGen def record `SVQSHRUNB`.
  **L1643 CN**: 声明 TableGen def 记录 `SVQSHRUNB`。
- **L1644 EN**: Declares TableGen def record `SVQRSHRUNB`.
  **L1644 CN**: 声明 TableGen def 记录 `SVQRSHRUNB`。
- **L1645 EN**: Declares TableGen def record `SVQSHRNB_S`.
  **L1645 CN**: 声明 TableGen def 记录 `SVQSHRNB_S`。
- **L1646 EN**: Declares TableGen def record `SVQSHRNB_U`.
  **L1646 CN**: 声明 TableGen def 记录 `SVQSHRNB_U`。
- **L1647 EN**: Declares TableGen def record `SVQRSHRNB_S`.
  **L1647 CN**: 声明 TableGen def 记录 `SVQRSHRNB_S`。
- **L1648 EN**: Declares TableGen def record `SVQRSHRNB_U`.
  **L1648 CN**: 声明 TableGen def 记录 `SVQRSHRNB_U`。
- **L1649 EN**: Blank line separating nearby declarations or logic blocks.
  **L1649 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1650 EN**: Declares TableGen def record `SVSHRNT`.
  **L1650 CN**: 声明 TableGen def 记录 `SVSHRNT`。
- **L1651 EN**: Declares TableGen def record `SVRSHRNT`.
  **L1651 CN**: 声明 TableGen def 记录 `SVRSHRNT`。
- **L1652 EN**: Declares TableGen def record `SVQSHRUNT`.
  **L1652 CN**: 声明 TableGen def 记录 `SVQSHRUNT`。

### Lines 1653-1680

````tablegen
def SVQRSHRUNT  : SInst<"svqrshrunt[_n_{d}]", "eedi", "sil",       MergeNone, "aarch64_sve_sqrshrunt", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
def SVQSHRNT_S  : SInst<"svqshrnt[_n_{d}]",   "hhdi", "sil",       MergeNone, "aarch64_sve_sqshrnt",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
def SVQSHRNT_U  : SInst<"svqshrnt[_n_{d}]",   "hhdi", "UsUiUl",    MergeNone, "aarch64_sve_uqshrnt",   [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
def SVQRSHRNT_S : SInst<"svqrshrnt[_n_{d}]",  "hhdi", "sil",       MergeNone, "aarch64_sve_sqrshrnt",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
def SVQRSHRNT_U : SInst<"svqrshrnt[_n_{d}]",  "hhdi", "UsUiUl",    MergeNone, "aarch64_sve_uqrshrnt",  [VerifyRuntimeMode], [ImmCheck<2, ImmCheckShiftRightNarrow, 1>]>;
}
////////////////////////////////////////////////////////////////////////////////
// SVE2 - Unary narrowing operations

let SVETargetGuard = "sve2|sme" in {
def SVQXTNB_S  : SInst<"svqxtnb[_{d}]",  "hd",  "sil",    MergeNone, "aarch64_sve_sqxtnb", [VerifyRuntimeMode]>;
def SVQXTNB_U  : SInst<"svqxtnb[_{d}]",  "hd",  "UsUiUl", MergeNone, "aarch64_sve_uqxtnb", [VerifyRuntimeMode]>;
def SVQXTUNB_S : SInst<"svqxtunb[_{d}]", "ed",  "sil",    MergeNone, "aarch64_sve_sqxtunb", [VerifyRuntimeMode]>;

def SVQXTNT_S  : SInst<"svqxtnt[_{d}]",  "hhd", "sil",    MergeNone, "aarch64_sve_sqxtnt", [VerifyRuntimeMode]>;
def SVQXTNT_U  : SInst<"svqxtnt[_{d}]",  "hhd", "UsUiUl", MergeNone, "aarch64_sve_uqxtnt", [VerifyRuntimeMode]>;
def SVQXTUNT_S : SInst<"svqxtunt[_{d}]", "eed", "sil",    MergeNone, "aarch64_sve_sqxtunt", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Widening complex integer arithmetic

let SVETargetGuard = "sve2|sme" in {
defm SVADDLBT : SInstWideDSPLong<"svaddlbt", "sil", "aarch64_sve_saddlbt">;
defm SVSUBLBT : SInstWideDSPLong<"svsublbt", "sil", "aarch64_sve_ssublbt">;
defm SVSUBLTB : SInstWideDSPLong<"svsubltb", "sil", "aarch64_sve_ssubltb">;

defm SVQDMLALBT : SInstWideDSPAcc<"svqdmlalbt", "sil", "aarch64_sve_sqdmlalbt">;
````
- **L1653 EN**: Declares TableGen def record `SVQRSHRUNT`.
  **L1653 CN**: 声明 TableGen def 记录 `SVQRSHRUNT`。
- **L1654 EN**: Declares TableGen def record `SVQSHRNT_S`.
  **L1654 CN**: 声明 TableGen def 记录 `SVQSHRNT_S`。
- **L1655 EN**: Declares TableGen def record `SVQSHRNT_U`.
  **L1655 CN**: 声明 TableGen def 记录 `SVQSHRNT_U`。
- **L1656 EN**: Declares TableGen def record `SVQRSHRNT_S`.
  **L1656 CN**: 声明 TableGen def 记录 `SVQRSHRNT_S`。
- **L1657 EN**: Declares TableGen def record `SVQRSHRNT_U`.
  **L1657 CN**: 声明 TableGen def 记录 `SVQRSHRNT_U`。
- **L1658 EN**: Closes the current lexical scope or compound statement.
  **L1658 CN**: 结束当前词法作用域或复合语句块。
- **L1659 EN**: Separator comment used for visual grouping.
  **L1659 CN**: 用于视觉分组的分隔注释。
- **L1660 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Unary narrowing operations`.
  **L1660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Unary narrowing operations`。
- **L1661 EN**: Blank line separating nearby declarations or logic blocks.
  **L1661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1662 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1662 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1663 EN**: Declares TableGen def record `SVQXTNB_S`.
  **L1663 CN**: 声明 TableGen def 记录 `SVQXTNB_S`。
- **L1664 EN**: Declares TableGen def record `SVQXTNB_U`.
  **L1664 CN**: 声明 TableGen def 记录 `SVQXTNB_U`。
- **L1665 EN**: Declares TableGen def record `SVQXTUNB_S`.
  **L1665 CN**: 声明 TableGen def 记录 `SVQXTUNB_S`。
- **L1666 EN**: Blank line separating nearby declarations or logic blocks.
  **L1666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1667 EN**: Declares TableGen def record `SVQXTNT_S`.
  **L1667 CN**: 声明 TableGen def 记录 `SVQXTNT_S`。
- **L1668 EN**: Declares TableGen def record `SVQXTNT_U`.
  **L1668 CN**: 声明 TableGen def 记录 `SVQXTNT_U`。
- **L1669 EN**: Declares TableGen def record `SVQXTUNT_S`.
  **L1669 CN**: 声明 TableGen def 记录 `SVQXTUNT_S`。
- **L1670 EN**: Closes the current lexical scope or compound statement.
  **L1670 CN**: 结束当前词法作用域或复合语句块。
- **L1671 EN**: Blank line separating nearby declarations or logic blocks.
  **L1671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1672 EN**: Separator comment used for visual grouping.
  **L1672 CN**: 用于视觉分组的分隔注释。
- **L1673 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Widening complex integer arithmetic`.
  **L1673 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Widening complex integer arithmetic`。
- **L1674 EN**: Blank line separating nearby declarations or logic blocks.
  **L1674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1675 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1675 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1676 EN**: Declares TableGen defm record `SVADDLBT`.
  **L1676 CN**: 声明 TableGen defm 记录 `SVADDLBT`。
- **L1677 EN**: Declares TableGen defm record `SVSUBLBT`.
  **L1677 CN**: 声明 TableGen defm 记录 `SVSUBLBT`。
- **L1678 EN**: Declares TableGen defm record `SVSUBLTB`.
  **L1678 CN**: 声明 TableGen defm 记录 `SVSUBLTB`。
- **L1679 EN**: Blank line separating nearby declarations or logic blocks.
  **L1679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1680 EN**: Declares TableGen defm record `SVQDMLALBT`.
  **L1680 CN**: 声明 TableGen defm 记录 `SVQDMLALBT`。

### Lines 1681-1708

````tablegen
defm SVQDMLSLBT : SInstWideDSPAcc<"svqdmlslbt", "sil", "aarch64_sve_sqdmlslbt">;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Non-temporal gather/scatter
let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {
// Non-temporal gather load one vector (vector base)
def SVLDNT1_GATHER_BASES_U   : MInst<"svldnt1_gather[_{2}base]_{0}",   "dPu", "ilUiUlfd", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SB_GATHER_BASES_U : MInst<"svldnt1sb_gather[_{2}base]_{0}", "dPu", "ilUiUl",   [IsGatherLoad],               MemEltTyInt8,    "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UB_GATHER_BASES_U : MInst<"svldnt1ub_gather[_{2}base]_{0}", "dPu", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SH_GATHER_BASES_U : MInst<"svldnt1sh_gather[_{2}base]_{0}", "dPu", "ilUiUl",   [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UH_GATHER_BASES_U : MInst<"svldnt1uh_gather[_{2}base]_{0}", "dPu", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SW_GATHER_BASES_U : MInst<"svldnt1sw_gather[_{2}base]_{0}", "dPu", "lUl",      [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UW_GATHER_BASES_U : MInst<"svldnt1uw_gather[_{2}base]_{0}", "dPu", "lUl",      [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnt1_gather_scalar_offset">;

// Non-temporal gather load one vector (scalar base, signed vector offset in bytes)
def SVLDNT1_GATHER_64B_OFFSETS_S   : MInst<"svldnt1_gather_[{3}]offset[_{0}]", "dPcx", "lUld", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldnt1_gather">;
def SVLDNT1SB_GATHER_64B_OFFSETS_S : MInst<"svldnt1sb_gather_[{3}]offset_{0}", "dPSx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldnt1_gather">;
def SVLDNT1UB_GATHER_64B_OFFSETS_S : MInst<"svldnt1ub_gather_[{3}]offset_{0}", "dPWx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldnt1_gather">;
def SVLDNT1SH_GATHER_64B_OFFSETS_S : MInst<"svldnt1sh_gather_[{3}]offset_{0}", "dPTx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather">;
def SVLDNT1UH_GATHER_64B_OFFSETS_S : MInst<"svldnt1uh_gather_[{3}]offset_{0}", "dPXx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather">;
def SVLDNT1SW_GATHER_64B_OFFSETS_S : MInst<"svldnt1sw_gather_[{3}]offset_{0}", "dPUx", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ldnt1_gather">;
def SVLDNT1UW_GATHER_64B_OFFSETS_S : MInst<"svldnt1uw_gather_[{3}]offset_{0}", "dPYx", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnt1_gather">;

// Non-temporal gather load one vector (scalar base, unsigned vector offset in bytes)
def SVLDNT1_GATHER_64B_OFFSETS_U   : MInst<"svldnt1_gather_[{3}]offset[_{0}]", "dPcu", "lUld", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldnt1_gather">;
def SVLDNT1SB_GATHER_64B_OFFSETS_U : MInst<"svldnt1sb_gather_[{3}]offset_{0}", "dPSu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldnt1_gather">;
def SVLDNT1UB_GATHER_64B_OFFSETS_U : MInst<"svldnt1ub_gather_[{3}]offset_{0}", "dPWu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldnt1_gather">;
````
- **L1681 EN**: Declares TableGen defm record `SVQDMLSLBT`.
  **L1681 CN**: 声明 TableGen defm 记录 `SVQDMLSLBT`。
- **L1682 EN**: Closes the current lexical scope or compound statement.
  **L1682 CN**: 结束当前词法作用域或复合语句块。
- **L1683 EN**: Blank line separating nearby declarations or logic blocks.
  **L1683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1684 EN**: Separator comment used for visual grouping.
  **L1684 CN**: 用于视觉分组的分隔注释。
- **L1685 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Non-temporal gather/scatter`.
  **L1685 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Non-temporal gather/scatter`。
- **L1686 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {`.
  **L1686 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {`。
- **L1687 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal gather load one vector (vector base)`.
  **L1687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal gather load one vector (vector base)`。
- **L1688 EN**: Declares TableGen def record `SVLDNT1_GATHER_BASES_U`.
  **L1688 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_BASES_U`。
- **L1689 EN**: Declares TableGen def record `SVLDNT1SB_GATHER_BASES_U`.
  **L1689 CN**: 声明 TableGen def 记录 `SVLDNT1SB_GATHER_BASES_U`。
- **L1690 EN**: Declares TableGen def record `SVLDNT1UB_GATHER_BASES_U`.
  **L1690 CN**: 声明 TableGen def 记录 `SVLDNT1UB_GATHER_BASES_U`。
- **L1691 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_BASES_U`.
  **L1691 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_BASES_U`。
- **L1692 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_BASES_U`.
  **L1692 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_BASES_U`。
- **L1693 EN**: Declares TableGen def record `SVLDNT1SW_GATHER_BASES_U`.
  **L1693 CN**: 声明 TableGen def 记录 `SVLDNT1SW_GATHER_BASES_U`。
- **L1694 EN**: Declares TableGen def record `SVLDNT1UW_GATHER_BASES_U`.
  **L1694 CN**: 声明 TableGen def 记录 `SVLDNT1UW_GATHER_BASES_U`。
- **L1695 EN**: Blank line separating nearby declarations or logic blocks.
  **L1695 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1696 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal gather load one vector (scalar base, signed vector offset in bytes)`.
  **L1696 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal gather load one vector (scalar base, signed vector offset in bytes)`。
- **L1697 EN**: Declares TableGen def record `SVLDNT1_GATHER_64B_OFFSETS_S`.
  **L1697 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_64B_OFFSETS_S`。
- **L1698 EN**: Declares TableGen def record `SVLDNT1SB_GATHER_64B_OFFSETS_S`.
  **L1698 CN**: 声明 TableGen def 记录 `SVLDNT1SB_GATHER_64B_OFFSETS_S`。
- **L1699 EN**: Declares TableGen def record `SVLDNT1UB_GATHER_64B_OFFSETS_S`.
  **L1699 CN**: 声明 TableGen def 记录 `SVLDNT1UB_GATHER_64B_OFFSETS_S`。
- **L1700 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_64B_OFFSETS_S`.
  **L1700 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_64B_OFFSETS_S`。
- **L1701 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_64B_OFFSETS_S`.
  **L1701 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_64B_OFFSETS_S`。
- **L1702 EN**: Declares TableGen def record `SVLDNT1SW_GATHER_64B_OFFSETS_S`.
  **L1702 CN**: 声明 TableGen def 记录 `SVLDNT1SW_GATHER_64B_OFFSETS_S`。
- **L1703 EN**: Declares TableGen def record `SVLDNT1UW_GATHER_64B_OFFSETS_S`.
  **L1703 CN**: 声明 TableGen def 记录 `SVLDNT1UW_GATHER_64B_OFFSETS_S`。
- **L1704 EN**: Blank line separating nearby declarations or logic blocks.
  **L1704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1705 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal gather load one vector (scalar base, unsigned vector offset in bytes)`.
  **L1705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal gather load one vector (scalar base, unsigned vector offset in bytes)`。
- **L1706 EN**: Declares TableGen def record `SVLDNT1_GATHER_64B_OFFSETS_U`.
  **L1706 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_64B_OFFSETS_U`。
- **L1707 EN**: Declares TableGen def record `SVLDNT1SB_GATHER_64B_OFFSETS_U`.
  **L1707 CN**: 声明 TableGen def 记录 `SVLDNT1SB_GATHER_64B_OFFSETS_U`。
- **L1708 EN**: Declares TableGen def record `SVLDNT1UB_GATHER_64B_OFFSETS_U`.
  **L1708 CN**: 声明 TableGen def 记录 `SVLDNT1UB_GATHER_64B_OFFSETS_U`。

### Lines 1709-1736

````tablegen
def SVLDNT1SH_GATHER_64B_OFFSETS_U : MInst<"svldnt1sh_gather_[{3}]offset_{0}", "dPTu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather">;
def SVLDNT1UH_GATHER_64B_OFFSETS_U : MInst<"svldnt1uh_gather_[{3}]offset_{0}", "dPXu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather">;
def SVLDNT1SW_GATHER_64B_OFFSETS_U : MInst<"svldnt1sw_gather_[{3}]offset_{0}", "dPUu", "lUl",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ldnt1_gather">;
def SVLDNT1UW_GATHER_64B_OFFSETS_U : MInst<"svldnt1uw_gather_[{3}]offset_{0}", "dPYu", "lUl",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnt1_gather">;

def SVLDNT1_GATHER_32B_OFFSETS_U   : MInst<"svldnt1_gather_[{3}]offset[_{0}]", "dPcu", "iUif", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldnt1_gather_uxtw">;
def SVLDNT1SB_GATHER_32B_OFFSETS_U : MInst<"svldnt1sb_gather_[{3}]offset_{0}", "dPSu", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldnt1_gather_uxtw">;
def SVLDNT1UB_GATHER_32B_OFFSETS_U : MInst<"svldnt1ub_gather_[{3}]offset_{0}", "dPWu", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldnt1_gather_uxtw">;
def SVLDNT1SH_GATHER_32B_OFFSETS_U : MInst<"svldnt1sh_gather_[{3}]offset_{0}", "dPTu", "iUi",  [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather_uxtw">;
def SVLDNT1UH_GATHER_32B_OFFSETS_U : MInst<"svldnt1uh_gather_[{3}]offset_{0}", "dPXu", "iUi",  [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather_uxtw">;

// Non-temporal gather load one vector (vector base, scalar offset in bytes)
def SVLDNT1_GATHER_OFFSET_S   : MInst<"svldnt1_gather[_{2}base]_offset_{0}",   "dPul", "ilUiUlfd", [IsGatherLoad, IsByteIndexed],               MemEltTyDefault, "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SB_GATHER_OFFSET_S : MInst<"svldnt1sb_gather[_{2}base]_offset_{0}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed],               MemEltTyInt8,    "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UB_GATHER_OFFSET_S : MInst<"svldnt1ub_gather[_{2}base]_offset_{0}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt8,    "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SH_GATHER_OFFSET_S : MInst<"svldnt1sh_gather[_{2}base]_offset_{0}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UH_GATHER_OFFSET_S : MInst<"svldnt1uh_gather[_{2}base]_offset_{0}", "dPul", "ilUiUl",   [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SW_GATHER_OFFSET_S : MInst<"svldnt1sw_gather[_{2}base]_offset_{0}", "dPul", "lUl",      [IsGatherLoad, IsByteIndexed],               MemEltTyInt32,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UW_GATHER_OFFSET_S : MInst<"svldnt1uw_gather[_{2}base]_offset_{0}", "dPul", "lUl",      [IsGatherLoad, IsByteIndexed, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnt1_gather_scalar_offset">;

// Non-temporal gather load one vector (scalar base, signed vector index)
def SVLDNT1_GATHER_64B_INDICES_S   : MInst<"svldnt1_gather_[{3}]index[_{0}]", "dPcx", "lUld", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1SH_GATHER_64B_INDICES_S : MInst<"svldnt1sh_gather_[{3}]index_{0}", "dPTx", "lUl",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1UH_GATHER_64B_INDICES_S : MInst<"svldnt1uh_gather_[{3}]index_{0}", "dPXx", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1SW_GATHER_64B_INDICES_S : MInst<"svldnt1sw_gather_[{3}]index_{0}", "dPUx", "lUl",  [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1UW_GATHER_64B_INDICES_S : MInst<"svldnt1uw_gather_[{3}]index_{0}", "dPYx", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnt1_gather_index">;

// Non temporal gather load one vector (scalar base, unsigned vector index)
````
- **L1709 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_64B_OFFSETS_U`.
  **L1709 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_64B_OFFSETS_U`。
- **L1710 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_64B_OFFSETS_U`.
  **L1710 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_64B_OFFSETS_U`。
- **L1711 EN**: Declares TableGen def record `SVLDNT1SW_GATHER_64B_OFFSETS_U`.
  **L1711 CN**: 声明 TableGen def 记录 `SVLDNT1SW_GATHER_64B_OFFSETS_U`。
- **L1712 EN**: Declares TableGen def record `SVLDNT1UW_GATHER_64B_OFFSETS_U`.
  **L1712 CN**: 声明 TableGen def 记录 `SVLDNT1UW_GATHER_64B_OFFSETS_U`。
- **L1713 EN**: Blank line separating nearby declarations or logic blocks.
  **L1713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1714 EN**: Declares TableGen def record `SVLDNT1_GATHER_32B_OFFSETS_U`.
  **L1714 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_32B_OFFSETS_U`。
- **L1715 EN**: Declares TableGen def record `SVLDNT1SB_GATHER_32B_OFFSETS_U`.
  **L1715 CN**: 声明 TableGen def 记录 `SVLDNT1SB_GATHER_32B_OFFSETS_U`。
- **L1716 EN**: Declares TableGen def record `SVLDNT1UB_GATHER_32B_OFFSETS_U`.
  **L1716 CN**: 声明 TableGen def 记录 `SVLDNT1UB_GATHER_32B_OFFSETS_U`。
- **L1717 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_32B_OFFSETS_U`.
  **L1717 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_32B_OFFSETS_U`。
- **L1718 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_32B_OFFSETS_U`.
  **L1718 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_32B_OFFSETS_U`。
- **L1719 EN**: Blank line separating nearby declarations or logic blocks.
  **L1719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1720 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal gather load one vector (vector base, scalar offset in bytes)`.
  **L1720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal gather load one vector (vector base, scalar offset in bytes)`。
- **L1721 EN**: Declares TableGen def record `SVLDNT1_GATHER_OFFSET_S`.
  **L1721 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_OFFSET_S`。
- **L1722 EN**: Declares TableGen def record `SVLDNT1SB_GATHER_OFFSET_S`.
  **L1722 CN**: 声明 TableGen def 记录 `SVLDNT1SB_GATHER_OFFSET_S`。
- **L1723 EN**: Declares TableGen def record `SVLDNT1UB_GATHER_OFFSET_S`.
  **L1723 CN**: 声明 TableGen def 记录 `SVLDNT1UB_GATHER_OFFSET_S`。
- **L1724 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_OFFSET_S`.
  **L1724 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_OFFSET_S`。
- **L1725 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_OFFSET_S`.
  **L1725 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_OFFSET_S`。
- **L1726 EN**: Declares TableGen def record `SVLDNT1SW_GATHER_OFFSET_S`.
  **L1726 CN**: 声明 TableGen def 记录 `SVLDNT1SW_GATHER_OFFSET_S`。
- **L1727 EN**: Declares TableGen def record `SVLDNT1UW_GATHER_OFFSET_S`.
  **L1727 CN**: 声明 TableGen def 记录 `SVLDNT1UW_GATHER_OFFSET_S`。
- **L1728 EN**: Blank line separating nearby declarations or logic blocks.
  **L1728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1729 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal gather load one vector (scalar base, signed vector index)`.
  **L1729 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal gather load one vector (scalar base, signed vector index)`。
- **L1730 EN**: Declares TableGen def record `SVLDNT1_GATHER_64B_INDICES_S`.
  **L1730 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_64B_INDICES_S`。
- **L1731 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_64B_INDICES_S`.
  **L1731 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_64B_INDICES_S`。
- **L1732 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_64B_INDICES_S`.
  **L1732 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_64B_INDICES_S`。
- **L1733 EN**: Declares TableGen def record `SVLDNT1SW_GATHER_64B_INDICES_S`.
  **L1733 CN**: 声明 TableGen def 记录 `SVLDNT1SW_GATHER_64B_INDICES_S`。
- **L1734 EN**: Declares TableGen def record `SVLDNT1UW_GATHER_64B_INDICES_S`.
  **L1734 CN**: 声明 TableGen def 记录 `SVLDNT1UW_GATHER_64B_INDICES_S`。
- **L1735 EN**: Blank line separating nearby declarations or logic blocks.
  **L1735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1736 EN**: Comment explains nearby logic, constraints, or intent: `Non temporal gather load one vector (scalar base, unsigned vector index)`.
  **L1736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non temporal gather load one vector (scalar base, unsigned vector index)`。

### Lines 1737-1764

````tablegen
def SVLDNT1_GATHER_64B_INDICES_U   : MInst<"svldnt1_gather_[{3}]index[_{0}]", "dPcu", "lUld", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1SH_GATHER_64B_INDICES_U : MInst<"svldnt1sh_gather_[{3}]index_{0}", "dPTu", "lUl",  [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1UH_GATHER_64B_INDICES_U : MInst<"svldnt1uh_gather_[{3}]index_{0}", "dPXu", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1SW_GATHER_64B_INDICES_U : MInst<"svldnt1sw_gather_[{3}]index_{0}", "dPUu", "lUl",  [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldnt1_gather_index">;
def SVLDNT1UW_GATHER_64B_INDICES_U : MInst<"svldnt1uw_gather_[{3}]index_{0}", "dPYu", "lUl",  [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnt1_gather_index">;

// Non-temporal gather load one vector (vector base, signed scalar index)
def SVLDNT1_GATHER_INDEX_S   : MInst<"svldnt1_gather[_{2}base]_index_{0}",   "dPul", "ilUiUlfd", [IsGatherLoad],               MemEltTyDefault, "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SH_GATHER_INDEX_S : MInst<"svldnt1sh_gather[_{2}base]_index_{0}", "dPul", "ilUiUl",   [IsGatherLoad],               MemEltTyInt16,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UH_GATHER_INDEX_S : MInst<"svldnt1uh_gather[_{2}base]_index_{0}", "dPul", "ilUiUl",   [IsGatherLoad, IsZExtReturn], MemEltTyInt16,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1SW_GATHER_INDEX_S : MInst<"svldnt1sw_gather[_{2}base]_index_{0}", "dPul", "lUl",      [IsGatherLoad],               MemEltTyInt32,   "aarch64_sve_ldnt1_gather_scalar_offset">;
def SVLDNT1UW_GATHER_INDEX_S : MInst<"svldnt1uw_gather[_{2}base]_index_{0}", "dPul", "lUl",      [IsGatherLoad, IsZExtReturn], MemEltTyInt32,   "aarch64_sve_ldnt1_gather_scalar_offset">;

// Non-temporal scatter store one vector (vector base)
def SVSTNT1_SCATTER_BASES_U  : MInst<"svstnt1_scatter[_{2}base_{d}]",  "vPud", "ilUiUlfd", [IsScatterStore], MemEltTyDefault, "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1B_SCATTER_BASES_U : MInst<"svstnt1b_scatter[_{2}base_{d}]", "vPud", "ilUiUl",   [IsScatterStore], MemEltTyInt8,    "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1H_SCATTER_BASES_U : MInst<"svstnt1h_scatter[_{2}base_{d}]", "vPud", "ilUiUl",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1W_SCATTER_BASES_U : MInst<"svstnt1w_scatter[_{2}base_{d}]", "vPud", "lUl",      [IsScatterStore], MemEltTyInt32,   "aarch64_sve_stnt1_scatter_scalar_offset">;

// Non-temporal scatter store one vector (scalar base, signed vector offset in bytes)
def SVSTNT1_SCATTER_64B_OFFSETS_S   : MInst<"svstnt1_scatter_[{3}]offset[_{d}]",  "vPpxd", "lUld", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_stnt1_scatter">;
def SVSTNT1B_SCATTER_64B_OFFSETS_SS : MInst<"svstnt1b_scatter_[{3}]offset[_{d}]", "vPAxd", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_stnt1_scatter">;
def SVSTNT1B_SCATTER_64B_OFFSETS_SU : MInst<"svstnt1b_scatter_[{3}]offset[_{d}]", "vPExd", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_stnt1_scatter">;
def SVSTNT1H_SCATTER_64B_OFFSETS_SS : MInst<"svstnt1h_scatter_[{3}]offset[_{d}]", "vPBxd", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_stnt1_scatter">;
def SVSTNT1H_SCATTER_64B_OFFSETS_SU : MInst<"svstnt1h_scatter_[{3}]offset[_{d}]", "vPFxd", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_stnt1_scatter">;
def SVSTNT1W_SCATTER_64B_OFFSETS_SS : MInst<"svstnt1w_scatter_[{3}]offset[_{d}]", "vPCxd", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_stnt1_scatter">;
def SVSTNT1W_SCATTER_64B_OFFSETS_SU : MInst<"svstnt1w_scatter_[{3}]offset[_{d}]", "vPGxd", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_stnt1_scatter">;

````
- **L1737 EN**: Declares TableGen def record `SVLDNT1_GATHER_64B_INDICES_U`.
  **L1737 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_64B_INDICES_U`。
- **L1738 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_64B_INDICES_U`.
  **L1738 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_64B_INDICES_U`。
- **L1739 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_64B_INDICES_U`.
  **L1739 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_64B_INDICES_U`。
- **L1740 EN**: Declares TableGen def record `SVLDNT1SW_GATHER_64B_INDICES_U`.
  **L1740 CN**: 声明 TableGen def 记录 `SVLDNT1SW_GATHER_64B_INDICES_U`。
- **L1741 EN**: Declares TableGen def record `SVLDNT1UW_GATHER_64B_INDICES_U`.
  **L1741 CN**: 声明 TableGen def 记录 `SVLDNT1UW_GATHER_64B_INDICES_U`。
- **L1742 EN**: Blank line separating nearby declarations or logic blocks.
  **L1742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1743 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal gather load one vector (vector base, signed scalar index)`.
  **L1743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal gather load one vector (vector base, signed scalar index)`。
- **L1744 EN**: Declares TableGen def record `SVLDNT1_GATHER_INDEX_S`.
  **L1744 CN**: 声明 TableGen def 记录 `SVLDNT1_GATHER_INDEX_S`。
- **L1745 EN**: Declares TableGen def record `SVLDNT1SH_GATHER_INDEX_S`.
  **L1745 CN**: 声明 TableGen def 记录 `SVLDNT1SH_GATHER_INDEX_S`。
- **L1746 EN**: Declares TableGen def record `SVLDNT1UH_GATHER_INDEX_S`.
  **L1746 CN**: 声明 TableGen def 记录 `SVLDNT1UH_GATHER_INDEX_S`。
- **L1747 EN**: Declares TableGen def record `SVLDNT1SW_GATHER_INDEX_S`.
  **L1747 CN**: 声明 TableGen def 记录 `SVLDNT1SW_GATHER_INDEX_S`。
- **L1748 EN**: Declares TableGen def record `SVLDNT1UW_GATHER_INDEX_S`.
  **L1748 CN**: 声明 TableGen def 记录 `SVLDNT1UW_GATHER_INDEX_S`。
- **L1749 EN**: Blank line separating nearby declarations or logic blocks.
  **L1749 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1750 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal scatter store one vector (vector base)`.
  **L1750 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal scatter store one vector (vector base)`。
- **L1751 EN**: Declares TableGen def record `SVSTNT1_SCATTER_BASES_U`.
  **L1751 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_BASES_U`。
- **L1752 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_BASES_U`.
  **L1752 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_BASES_U`。
- **L1753 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_BASES_U`.
  **L1753 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_BASES_U`。
- **L1754 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_BASES_U`.
  **L1754 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_BASES_U`。
- **L1755 EN**: Blank line separating nearby declarations or logic blocks.
  **L1755 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1756 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal scatter store one vector (scalar base, signed vector offset in bytes)`.
  **L1756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal scatter store one vector (scalar base, signed vector offset in bytes)`。
- **L1757 EN**: Declares TableGen def record `SVSTNT1_SCATTER_64B_OFFSETS_S`.
  **L1757 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_64B_OFFSETS_S`。
- **L1758 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_64B_OFFSETS_SS`.
  **L1758 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_64B_OFFSETS_SS`。
- **L1759 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_64B_OFFSETS_SU`.
  **L1759 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_64B_OFFSETS_SU`。
- **L1760 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_64B_OFFSETS_SS`.
  **L1760 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_64B_OFFSETS_SS`。
- **L1761 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_64B_OFFSETS_SU`.
  **L1761 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_64B_OFFSETS_SU`。
- **L1762 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_64B_OFFSETS_SS`.
  **L1762 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_64B_OFFSETS_SS`。
- **L1763 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_64B_OFFSETS_SU`.
  **L1763 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_64B_OFFSETS_SU`。
- **L1764 EN**: Blank line separating nearby declarations or logic blocks.
  **L1764 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1765-1792

````tablegen
// Non-temporal scatter store one vector (scalar base, unsigned vector offset in bytes)
def SVSTNT1_SCATTER_64B_OFFSETS_U   : MInst<"svstnt1_scatter_[{3}]offset[_{d}]",  "vPpud", "lUld", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_stnt1_scatter">;
def SVSTNT1B_SCATTER_64B_OFFSETS_US : MInst<"svstnt1b_scatter_[{3}]offset[_{d}]", "vPAud", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_stnt1_scatter">;
def SVSTNT1B_SCATTER_64B_OFFSETS_UU : MInst<"svstnt1b_scatter_[{3}]offset[_{d}]", "vPEud", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_stnt1_scatter">;
def SVSTNT1H_SCATTER_64B_OFFSETS_US : MInst<"svstnt1h_scatter_[{3}]offset[_{d}]", "vPBud", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_stnt1_scatter">;
def SVSTNT1H_SCATTER_64B_OFFSETS_UU : MInst<"svstnt1h_scatter_[{3}]offset[_{d}]", "vPFud", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_stnt1_scatter">;
def SVSTNT1W_SCATTER_64B_OFFSETS_US : MInst<"svstnt1w_scatter_[{3}]offset[_{d}]", "vPCud", "l",    [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_stnt1_scatter">;
def SVSTNT1W_SCATTER_64B_OFFSETS_UU : MInst<"svstnt1w_scatter_[{3}]offset[_{d}]", "vPGud", "Ul",   [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_stnt1_scatter">;

def SVSTNT1_SCATTER_32B_OFFSETS_U   : MInst<"svstnt1_scatter_[{3}]offset[_{d}]",  "vPpud", "iUif", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_stnt1_scatter_uxtw">;
def SVSTNT1B_SCATTER_32B_OFFSETS_US : MInst<"svstnt1b_scatter_[{3}]offset[_{d}]", "vPAud", "i",    [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_stnt1_scatter_uxtw">;
def SVSTNT1B_SCATTER_32B_OFFSETS_UU : MInst<"svstnt1b_scatter_[{3}]offset[_{d}]", "vPEud", "Ui",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_stnt1_scatter_uxtw">;
def SVSTNT1H_SCATTER_32B_OFFSETS_US : MInst<"svstnt1h_scatter_[{3}]offset[_{d}]", "vPBud", "i",    [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_uxtw">;
def SVSTNT1H_SCATTER_32B_OFFSETS_UU : MInst<"svstnt1h_scatter_[{3}]offset[_{d}]", "vPFud", "Ui",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_uxtw">;

// Non-temporal scatter store one vector (vector base, scalar offset in bytes)
def SVSTNT1_SCATTER_OFFSET_S  : MInst<"svstnt1_scatter[_{2}base]_offset[_{d}]",  "vPuld", "ilUiUlfd", [IsScatterStore, IsByteIndexed], MemEltTyDefault, "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1B_SCATTER_OFFSET_S : MInst<"svstnt1b_scatter[_{2}base]_offset[_{d}]", "vPuld", "ilUiUl",   [IsScatterStore, IsByteIndexed], MemEltTyInt8,    "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1H_SCATTER_OFFSET_S : MInst<"svstnt1h_scatter[_{2}base]_offset[_{d}]", "vPuld", "ilUiUl",   [IsScatterStore, IsByteIndexed], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1W_SCATTER_OFFSET_S : MInst<"svstnt1w_scatter[_{2}base]_offset[_{d}]", "vPuld", "lUl",      [IsScatterStore, IsByteIndexed], MemEltTyInt32,   "aarch64_sve_stnt1_scatter_scalar_offset">;

// Non-temporal scatter store one vector (scalar base, signed vector index)
def SVSTNT1_SCATTER_INDICES_S   : MInst<"svstnt1_scatter_[{3}]index[_{d}]",  "vPpxd", "lUld", [IsScatterStore], MemEltTyDefault, "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1H_SCATTER_INDICES_SS : MInst<"svstnt1h_scatter_[{3}]index[_{d}]", "vPBxd", "l",    [IsScatterStore], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1H_SCATTER_INDICES_SU : MInst<"svstnt1h_scatter_[{3}]index[_{d}]", "vPFxd", "Ul",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1W_SCATTER_INDICES_SS : MInst<"svstnt1w_scatter_[{3}]index[_{d}]", "vPCxd", "l",    [IsScatterStore], MemEltTyInt32,   "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1W_SCATTER_INDICES_SU : MInst<"svstnt1w_scatter_[{3}]index[_{d}]", "vPGxd", "Ul",   [IsScatterStore], MemEltTyInt32,   "aarch64_sve_stnt1_scatter_index">;

````
- **L1765 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal scatter store one vector (scalar base, unsigned vector offset in bytes)`.
  **L1765 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal scatter store one vector (scalar base, unsigned vector offset in bytes)`。
- **L1766 EN**: Declares TableGen def record `SVSTNT1_SCATTER_64B_OFFSETS_U`.
  **L1766 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_64B_OFFSETS_U`。
- **L1767 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_64B_OFFSETS_US`.
  **L1767 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_64B_OFFSETS_US`。
- **L1768 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_64B_OFFSETS_UU`.
  **L1768 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_64B_OFFSETS_UU`。
- **L1769 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_64B_OFFSETS_US`.
  **L1769 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_64B_OFFSETS_US`。
- **L1770 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_64B_OFFSETS_UU`.
  **L1770 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_64B_OFFSETS_UU`。
- **L1771 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_64B_OFFSETS_US`.
  **L1771 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_64B_OFFSETS_US`。
- **L1772 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_64B_OFFSETS_UU`.
  **L1772 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_64B_OFFSETS_UU`。
- **L1773 EN**: Blank line separating nearby declarations or logic blocks.
  **L1773 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1774 EN**: Declares TableGen def record `SVSTNT1_SCATTER_32B_OFFSETS_U`.
  **L1774 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_32B_OFFSETS_U`。
- **L1775 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_32B_OFFSETS_US`.
  **L1775 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_32B_OFFSETS_US`。
- **L1776 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_32B_OFFSETS_UU`.
  **L1776 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_32B_OFFSETS_UU`。
- **L1777 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_32B_OFFSETS_US`.
  **L1777 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_32B_OFFSETS_US`。
- **L1778 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_32B_OFFSETS_UU`.
  **L1778 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_32B_OFFSETS_UU`。
- **L1779 EN**: Blank line separating nearby declarations or logic blocks.
  **L1779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1780 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal scatter store one vector (vector base, scalar offset in bytes)`.
  **L1780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal scatter store one vector (vector base, scalar offset in bytes)`。
- **L1781 EN**: Declares TableGen def record `SVSTNT1_SCATTER_OFFSET_S`.
  **L1781 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_OFFSET_S`。
- **L1782 EN**: Declares TableGen def record `SVSTNT1B_SCATTER_OFFSET_S`.
  **L1782 CN**: 声明 TableGen def 记录 `SVSTNT1B_SCATTER_OFFSET_S`。
- **L1783 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_OFFSET_S`.
  **L1783 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_OFFSET_S`。
- **L1784 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_OFFSET_S`.
  **L1784 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_OFFSET_S`。
- **L1785 EN**: Blank line separating nearby declarations or logic blocks.
  **L1785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1786 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal scatter store one vector (scalar base, signed vector index)`.
  **L1786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal scatter store one vector (scalar base, signed vector index)`。
- **L1787 EN**: Declares TableGen def record `SVSTNT1_SCATTER_INDICES_S`.
  **L1787 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_INDICES_S`。
- **L1788 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_INDICES_SS`.
  **L1788 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_INDICES_SS`。
- **L1789 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_INDICES_SU`.
  **L1789 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_INDICES_SU`。
- **L1790 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_INDICES_SS`.
  **L1790 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_INDICES_SS`。
- **L1791 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_INDICES_SU`.
  **L1791 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_INDICES_SU`。
- **L1792 EN**: Blank line separating nearby declarations or logic blocks.
  **L1792 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1793-1820

````tablegen
// Non-temporal scatter store one vector (scalar base, unsigned vector index)
def SVSTNT1_SCATTER_INDICES_U   : MInst<"svstnt1_scatter_[{3}]index[_{d}]",  "vPpud", "lUld", [IsScatterStore], MemEltTyDefault, "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1H_SCATTER_INDICES_US : MInst<"svstnt1h_scatter_[{3}]index[_{d}]", "vPBud", "l",    [IsScatterStore], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1H_SCATTER_INDICES_UU : MInst<"svstnt1h_scatter_[{3}]index[_{d}]", "vPFud", "Ul",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1W_SCATTER_INDICES_US : MInst<"svstnt1w_scatter_[{3}]index[_{d}]", "vPCud", "l",    [IsScatterStore], MemEltTyInt32,   "aarch64_sve_stnt1_scatter_index">;
def SVSTNT1W_SCATTER_INDICES_UU : MInst<"svstnt1w_scatter_[{3}]index[_{d}]", "vPGud", "Ul",   [IsScatterStore], MemEltTyInt32,   "aarch64_sve_stnt1_scatter_index">;

// Non-temporal scatter store one vector (vector base, signed scalar index)
def SVSTNT1_SCATTER_INDEX_S  : MInst<"svstnt1_scatter[_{2}base]_index[_{d}]",  "vPuld", "ilUiUlfd", [IsScatterStore], MemEltTyDefault, "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1H_SCATTER_INDEX_S : MInst<"svstnt1h_scatter[_{2}base]_index[_{d}]", "vPuld", "ilUiUl",   [IsScatterStore], MemEltTyInt16,   "aarch64_sve_stnt1_scatter_scalar_offset">;
def SVSTNT1W_SCATTER_INDEX_S : MInst<"svstnt1w_scatter[_{2}base]_index[_{d}]", "vPuld", "lUl",      [IsScatterStore], MemEltTyInt32,   "aarch64_sve_stnt1_scatter_scalar_offset">;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Polynomial arithmetic

let SVETargetGuard = "sve2|sme" in {
def SVEORBT         : SInst<"sveorbt[_{d}]",         "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_eorbt", [VerifyRuntimeMode]>;
def SVEORBT_N       : SInst<"sveorbt[_n_{d}]",       "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_eorbt", [VerifyRuntimeMode]>;
def SVEORTB         : SInst<"sveortb[_{d}]",         "dddd", "csilUcUsUiUl", MergeNone, "aarch64_sve_eortb", [VerifyRuntimeMode]>;
def SVEORTB_N       : SInst<"sveortb[_n_{d}]",       "ddda", "csilUcUsUiUl", MergeNone, "aarch64_sve_eortb", [VerifyRuntimeMode]>;

def SVPMUL          : SInst<"svpmul[_{d}]",          "ddd",  "Uc",   MergeNone, "aarch64_sve_pmul", [VerifyRuntimeMode]>;
def SVPMUL_N        : SInst<"svpmul[_n_{d}]",        "dda",  "Uc",   MergeNone, "aarch64_sve_pmul", [VerifyRuntimeMode]>;
def SVPMULLB        : SInst<"svpmullb[_{d}]",        "dhh",  "UsUl", MergeNone, "", [VerifyRuntimeMode]>;
def SVPMULLB_N      : SInst<"svpmullb[_n_{d}]",      "dhR",  "UsUl", MergeNone, "", [VerifyRuntimeMode]>;
def SVPMULLB_PAIR   : SInst<"svpmullb_pair[_{d}]",   "ddd",  "UcUi", MergeNone, "aarch64_sve_pmullb_pair", [VerifyRuntimeMode]>;
def SVPMULLB_PAIR_N : SInst<"svpmullb_pair[_n_{d}]", "dda",  "UcUi", MergeNone, "aarch64_sve_pmullb_pair", [VerifyRuntimeMode]>;
````
- **L1793 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal scatter store one vector (scalar base, unsigned vector index)`.
  **L1793 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal scatter store one vector (scalar base, unsigned vector index)`。
- **L1794 EN**: Declares TableGen def record `SVSTNT1_SCATTER_INDICES_U`.
  **L1794 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_INDICES_U`。
- **L1795 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_INDICES_US`.
  **L1795 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_INDICES_US`。
- **L1796 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_INDICES_UU`.
  **L1796 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_INDICES_UU`。
- **L1797 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_INDICES_US`.
  **L1797 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_INDICES_US`。
- **L1798 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_INDICES_UU`.
  **L1798 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_INDICES_UU`。
- **L1799 EN**: Blank line separating nearby declarations or logic blocks.
  **L1799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1800 EN**: Comment explains nearby logic, constraints, or intent: `Non-temporal scatter store one vector (vector base, signed scalar index)`.
  **L1800 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non-temporal scatter store one vector (vector base, signed scalar index)`。
- **L1801 EN**: Declares TableGen def record `SVSTNT1_SCATTER_INDEX_S`.
  **L1801 CN**: 声明 TableGen def 记录 `SVSTNT1_SCATTER_INDEX_S`。
- **L1802 EN**: Declares TableGen def record `SVSTNT1H_SCATTER_INDEX_S`.
  **L1802 CN**: 声明 TableGen def 记录 `SVSTNT1H_SCATTER_INDEX_S`。
- **L1803 EN**: Declares TableGen def record `SVSTNT1W_SCATTER_INDEX_S`.
  **L1803 CN**: 声明 TableGen def 记录 `SVSTNT1W_SCATTER_INDEX_S`。
- **L1804 EN**: Closes the current lexical scope or compound statement.
  **L1804 CN**: 结束当前词法作用域或复合语句块。
- **L1805 EN**: Blank line separating nearby declarations or logic blocks.
  **L1805 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1806 EN**: Separator comment used for visual grouping.
  **L1806 CN**: 用于视觉分组的分隔注释。
- **L1807 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Polynomial arithmetic`.
  **L1807 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Polynomial arithmetic`。
- **L1808 EN**: Blank line separating nearby declarations or logic blocks.
  **L1808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1809 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1809 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1810 EN**: Declares TableGen def record `SVEORBT`.
  **L1810 CN**: 声明 TableGen def 记录 `SVEORBT`。
- **L1811 EN**: Declares TableGen def record `SVEORBT_N`.
  **L1811 CN**: 声明 TableGen def 记录 `SVEORBT_N`。
- **L1812 EN**: Declares TableGen def record `SVEORTB`.
  **L1812 CN**: 声明 TableGen def 记录 `SVEORTB`。
- **L1813 EN**: Declares TableGen def record `SVEORTB_N`.
  **L1813 CN**: 声明 TableGen def 记录 `SVEORTB_N`。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1815 EN**: Declares TableGen def record `SVPMUL`.
  **L1815 CN**: 声明 TableGen def 记录 `SVPMUL`。
- **L1816 EN**: Declares TableGen def record `SVPMUL_N`.
  **L1816 CN**: 声明 TableGen def 记录 `SVPMUL_N`。
- **L1817 EN**: Declares TableGen def record `SVPMULLB`.
  **L1817 CN**: 声明 TableGen def 记录 `SVPMULLB`。
- **L1818 EN**: Declares TableGen def record `SVPMULLB_N`.
  **L1818 CN**: 声明 TableGen def 记录 `SVPMULLB_N`。
- **L1819 EN**: Declares TableGen def record `SVPMULLB_PAIR`.
  **L1819 CN**: 声明 TableGen def 记录 `SVPMULLB_PAIR`。
- **L1820 EN**: Declares TableGen def record `SVPMULLB_PAIR_N`.
  **L1820 CN**: 声明 TableGen def 记录 `SVPMULLB_PAIR_N`。

### Lines 1821-1848

````tablegen
def SVPMULLT        : SInst<"svpmullt[_{d}]",        "dhh",  "UsUl", MergeNone, "", [VerifyRuntimeMode]>;
def SVPMULLT_N      : SInst<"svpmullt[_n_{d}]",      "dhR",  "UsUl", MergeNone, "", [VerifyRuntimeMode]>;
def SVPMULLT_PAIR   : SInst<"svpmullt_pair[_{d}]",   "ddd",  "UcUi", MergeNone, "aarch64_sve_pmullt_pair", [VerifyRuntimeMode]>;
def SVPMULLT_PAIR_N : SInst<"svpmullt_pair[_n_{d}]", "dda",  "UcUi", MergeNone, "aarch64_sve_pmullt_pair", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Complex integer dot product

let SVETargetGuard = "sve2|sme" in {
def SVCDOT      : SInst<"svcdot[_{d}]",      "ddqqi",  "il",   MergeNone, "aarch64_sve_cdot",      [VerifyRuntimeMode], [ImmCheck<3, ImmCheckComplexRotAll90>]>;
def SVCDOT_LANE : SInst<"svcdot_lane[_{d}]", "ddqqii", "il",   MergeNone, "aarch64_sve_cdot_lane", [VerifyRuntimeMode], [ImmCheck<4, ImmCheckComplexRotAll90>, ImmCheck<3, ImmCheckLaneIndexDot, 2>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Floating-point widening multiply-accumulate

let SVETargetGuard = "sve2|sme" in {
def SVMLALB_F      : SInst<"svmlalb[_{d}]",      "ddhh",  "f", MergeNone, "aarch64_sve_fmlalb", [VerifyRuntimeMode]>;
def SVMLALB_F_N    : SInst<"svmlalb[_n_{d}]",    "ddhR",  "f", MergeNone, "aarch64_sve_fmlalb", [VerifyRuntimeMode]>;
def SVMLALB_F_LANE : SInst<"svmlalb_lane[_{d}]", "ddhhi", "f", MergeNone, "aarch64_sve_fmlalb_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLALT_F      : SInst<"svmlalt[_{d}]",      "ddhh",  "f", MergeNone, "aarch64_sve_fmlalt", [VerifyRuntimeMode]>;
def SVMLALT_F_N    : SInst<"svmlalt[_n_{d}]",    "ddhR",  "f", MergeNone, "aarch64_sve_fmlalt", [VerifyRuntimeMode]>;
def SVMLALT_F_LANE : SInst<"svmlalt_lane[_{d}]", "ddhhi", "f", MergeNone, "aarch64_sve_fmlalt_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLSLB_F      : SInst<"svmlslb[_{d}]",      "ddhh",  "f", MergeNone, "aarch64_sve_fmlslb", [VerifyRuntimeMode]>;
def SVMLSLB_F_N    : SInst<"svmlslb[_n_{d}]",    "ddhR",  "f", MergeNone, "aarch64_sve_fmlslb", [VerifyRuntimeMode]>;
def SVMLSLB_F_LANE : SInst<"svmlslb_lane[_{d}]", "ddhhi", "f", MergeNone, "aarch64_sve_fmlslb_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLSLT_F      : SInst<"svmlslt[_{d}]",      "ddhh",  "f", MergeNone, "aarch64_sve_fmlslt", [VerifyRuntimeMode]>;
````
- **L1821 EN**: Declares TableGen def record `SVPMULLT`.
  **L1821 CN**: 声明 TableGen def 记录 `SVPMULLT`。
- **L1822 EN**: Declares TableGen def record `SVPMULLT_N`.
  **L1822 CN**: 声明 TableGen def 记录 `SVPMULLT_N`。
- **L1823 EN**: Declares TableGen def record `SVPMULLT_PAIR`.
  **L1823 CN**: 声明 TableGen def 记录 `SVPMULLT_PAIR`。
- **L1824 EN**: Declares TableGen def record `SVPMULLT_PAIR_N`.
  **L1824 CN**: 声明 TableGen def 记录 `SVPMULLT_PAIR_N`。
- **L1825 EN**: Closes the current lexical scope or compound statement.
  **L1825 CN**: 结束当前词法作用域或复合语句块。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1827 EN**: Separator comment used for visual grouping.
  **L1827 CN**: 用于视觉分组的分隔注释。
- **L1828 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Complex integer dot product`.
  **L1828 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Complex integer dot product`。
- **L1829 EN**: Blank line separating nearby declarations or logic blocks.
  **L1829 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1830 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1830 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1831 EN**: Declares TableGen def record `SVCDOT`.
  **L1831 CN**: 声明 TableGen def 记录 `SVCDOT`。
- **L1832 EN**: Declares TableGen def record `SVCDOT_LANE`.
  **L1832 CN**: 声明 TableGen def 记录 `SVCDOT_LANE`。
- **L1833 EN**: Closes the current lexical scope or compound statement.
  **L1833 CN**: 结束当前词法作用域或复合语句块。
- **L1834 EN**: Blank line separating nearby declarations or logic blocks.
  **L1834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1835 EN**: Separator comment used for visual grouping.
  **L1835 CN**: 用于视觉分组的分隔注释。
- **L1836 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Floating-point widening multiply-accumulate`.
  **L1836 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Floating-point widening multiply-accumulate`。
- **L1837 EN**: Blank line separating nearby declarations or logic blocks.
  **L1837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1838 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1838 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1839 EN**: Declares TableGen def record `SVMLALB_F`.
  **L1839 CN**: 声明 TableGen def 记录 `SVMLALB_F`。
- **L1840 EN**: Declares TableGen def record `SVMLALB_F_N`.
  **L1840 CN**: 声明 TableGen def 记录 `SVMLALB_F_N`。
- **L1841 EN**: Declares TableGen def record `SVMLALB_F_LANE`.
  **L1841 CN**: 声明 TableGen def 记录 `SVMLALB_F_LANE`。
- **L1842 EN**: Declares TableGen def record `SVMLALT_F`.
  **L1842 CN**: 声明 TableGen def 记录 `SVMLALT_F`。
- **L1843 EN**: Declares TableGen def record `SVMLALT_F_N`.
  **L1843 CN**: 声明 TableGen def 记录 `SVMLALT_F_N`。
- **L1844 EN**: Declares TableGen def record `SVMLALT_F_LANE`.
  **L1844 CN**: 声明 TableGen def 记录 `SVMLALT_F_LANE`。
- **L1845 EN**: Declares TableGen def record `SVMLSLB_F`.
  **L1845 CN**: 声明 TableGen def 记录 `SVMLSLB_F`。
- **L1846 EN**: Declares TableGen def record `SVMLSLB_F_N`.
  **L1846 CN**: 声明 TableGen def 记录 `SVMLSLB_F_N`。
- **L1847 EN**: Declares TableGen def record `SVMLSLB_F_LANE`.
  **L1847 CN**: 声明 TableGen def 记录 `SVMLSLB_F_LANE`。
- **L1848 EN**: Declares TableGen def record `SVMLSLT_F`.
  **L1848 CN**: 声明 TableGen def 记录 `SVMLSLT_F`。

### Lines 1849-1876

````tablegen
def SVMLSLT_F_N    : SInst<"svmlslt[_n_{d}]",    "ddhR",  "f", MergeNone, "aarch64_sve_fmlslt", [VerifyRuntimeMode]>;
def SVMLSLT_F_LANE : SInst<"svmlslt_lane[_{d}]", "ddhhi", "f", MergeNone, "aarch64_sve_fmlslt_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Floating-point integer binary logarithm

let SVETargetGuard = "sve2|sme" in {
def SVLOGB_M : SInst<"svlogb[_{d}]", "xxPd", "hfd", MergeOp1,     "aarch64_sve_flogb", [VerifyRuntimeMode]>;
def SVLOGB_X : SInst<"svlogb[_{d}]", "xPd",  "hfd", MergeAnyExp,  "aarch64_sve_flogb", [VerifyRuntimeMode]>;
def SVLOGB_Z : SInst<"svlogb[_{d}]", "xPd",  "hfd", MergeZeroExp, "aarch64_sve_flogb", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Vector Histogram count

let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {
def SVHISTCNT : SInst<"svhistcnt[_{d}]_z", "uPdd", "ilUiUl", MergeNone, "aarch64_sve_histcnt">;
def SVHISTSEG : SInst<"svhistseg[_{d}]",   "udd",  "cUc",    MergeNone, "aarch64_sve_histseg">;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Character match

let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {
def SVMATCH  : SInst<"svmatch[_{d}]",  "PPdd", "csUcUs", MergeNone, "aarch64_sve_match">;
def SVNMATCH : SInst<"svnmatch[_{d}]", "PPdd", "csUcUs", MergeNone, "aarch64_sve_nmatch">;
}
````
- **L1849 EN**: Declares TableGen def record `SVMLSLT_F_N`.
  **L1849 CN**: 声明 TableGen def 记录 `SVMLSLT_F_N`。
- **L1850 EN**: Declares TableGen def record `SVMLSLT_F_LANE`.
  **L1850 CN**: 声明 TableGen def 记录 `SVMLSLT_F_LANE`。
- **L1851 EN**: Closes the current lexical scope or compound statement.
  **L1851 CN**: 结束当前词法作用域或复合语句块。
- **L1852 EN**: Blank line separating nearby declarations or logic blocks.
  **L1852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1853 EN**: Separator comment used for visual grouping.
  **L1853 CN**: 用于视觉分组的分隔注释。
- **L1854 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Floating-point integer binary logarithm`.
  **L1854 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Floating-point integer binary logarithm`。
- **L1855 EN**: Blank line separating nearby declarations or logic blocks.
  **L1855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1856 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1856 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1857 EN**: Declares TableGen def record `SVLOGB_M`.
  **L1857 CN**: 声明 TableGen def 记录 `SVLOGB_M`。
- **L1858 EN**: Declares TableGen def record `SVLOGB_X`.
  **L1858 CN**: 声明 TableGen def 记录 `SVLOGB_X`。
- **L1859 EN**: Declares TableGen def record `SVLOGB_Z`.
  **L1859 CN**: 声明 TableGen def 记录 `SVLOGB_Z`。
- **L1860 EN**: Closes the current lexical scope or compound statement.
  **L1860 CN**: 结束当前词法作用域或复合语句块。
- **L1861 EN**: Blank line separating nearby declarations or logic blocks.
  **L1861 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1862 EN**: Separator comment used for visual grouping.
  **L1862 CN**: 用于视觉分组的分隔注释。
- **L1863 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Vector Histogram count`.
  **L1863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Vector Histogram count`。
- **L1864 EN**: Blank line separating nearby declarations or logic blocks.
  **L1864 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1865 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {`.
  **L1865 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {`。
- **L1866 EN**: Declares TableGen def record `SVHISTCNT`.
  **L1866 CN**: 声明 TableGen def 记录 `SVHISTCNT`。
- **L1867 EN**: Declares TableGen def record `SVHISTSEG`.
  **L1867 CN**: 声明 TableGen def 记录 `SVHISTSEG`。
- **L1868 EN**: Closes the current lexical scope or compound statement.
  **L1868 CN**: 结束当前词法作用域或复合语句块。
- **L1869 EN**: Blank line separating nearby declarations or logic blocks.
  **L1869 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1870 EN**: Separator comment used for visual grouping.
  **L1870 CN**: 用于视觉分组的分隔注释。
- **L1871 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Character match`.
  **L1871 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Character match`。
- **L1872 EN**: Blank line separating nearby declarations or logic blocks.
  **L1872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1873 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {`.
  **L1873 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2", SMETargetGuard = InvalidMode in {`。
- **L1874 EN**: Declares TableGen def record `SVMATCH`.
  **L1874 CN**: 声明 TableGen def 记录 `SVMATCH`。
- **L1875 EN**: Declares TableGen def record `SVNMATCH`.
  **L1875 CN**: 声明 TableGen def 记录 `SVNMATCH`。
- **L1876 EN**: Closes the current lexical scope or compound statement.
  **L1876 CN**: 结束当前词法作用域或复合语句块。

### Lines 1877-1904

````tablegen

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Contiguous conflict detection
let SVETargetGuard = "sve2|sme" in {
def SVWHILERW_B : SInst<"svwhilerw[_{1}]", "Pcc", "cUc",  MergeNone, "aarch64_sve_whilerw_b", [IsOverloadWhileRW, VerifyRuntimeMode]>;
def SVWHILERW_H : SInst<"svwhilerw[_{1}]", "Pcc", "sUshb", MergeNone, "aarch64_sve_whilerw_h", [IsOverloadWhileRW, VerifyRuntimeMode]>;
def SVWHILERW_S : SInst<"svwhilerw[_{1}]", "Pcc", "iUif", MergeNone, "aarch64_sve_whilerw_s", [IsOverloadWhileRW, VerifyRuntimeMode]>;
def SVWHILERW_D : SInst<"svwhilerw[_{1}]", "Pcc", "lUld", MergeNone, "aarch64_sve_whilerw_d", [IsOverloadWhileRW, VerifyRuntimeMode]>;

def SVWHILEWR_B : SInst<"svwhilewr[_{1}]", "Pcc", "cUc",  MergeNone, "aarch64_sve_whilewr_b", [IsOverloadWhileRW, VerifyRuntimeMode]>;
def SVWHILEWR_H : SInst<"svwhilewr[_{1}]", "Pcc", "sUshb", MergeNone, "aarch64_sve_whilewr_h", [IsOverloadWhileRW, VerifyRuntimeMode]>;
def SVWHILEWR_S : SInst<"svwhilewr[_{1}]", "Pcc", "iUif", MergeNone, "aarch64_sve_whilewr_s", [IsOverloadWhileRW, VerifyRuntimeMode]>;
def SVWHILEWR_D : SInst<"svwhilewr[_{1}]", "Pcc", "lUld", MergeNone, "aarch64_sve_whilewr_d", [IsOverloadWhileRW, VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Extended table lookup/permute
let SVETargetGuard = "sve2|sme" in {
def SVTBL2 : SInst<"svtbl2[_{d}]", "d2u",  "csilUcUsUiUlhfdb", MergeNone, "", [VerifyRuntimeMode]>;
def SVTBX  : SInst<"svtbx[_{d}]",  "dddu", "csilUcUsUiUlhfdb", MergeNone, "aarch64_sve_tbx", [VerifyRuntimeMode]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Lookup table
let SVETargetGuard = "(sve2|sme2),lut", SMETargetGuard = "sme2,lut" in {
  def SVLUTI2_B : SInst<"svluti2_lane[_{d}]", "dd[i", "cUc",   MergeNone, "aarch64_sve_luti2_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
  def SVLUTI2_H : SInst<"svluti2_lane[_{d}]", "dd[i", "sUshb", MergeNone, "aarch64_sve_luti2_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_7>]>;

````
- **L1877 EN**: Blank line separating nearby declarations or logic blocks.
  **L1877 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1878 EN**: Separator comment used for visual grouping.
  **L1878 CN**: 用于视觉分组的分隔注释。
- **L1879 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Contiguous conflict detection`.
  **L1879 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Contiguous conflict detection`。
- **L1880 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1880 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1881 EN**: Declares TableGen def record `SVWHILERW_B`.
  **L1881 CN**: 声明 TableGen def 记录 `SVWHILERW_B`。
- **L1882 EN**: Declares TableGen def record `SVWHILERW_H`.
  **L1882 CN**: 声明 TableGen def 记录 `SVWHILERW_H`。
- **L1883 EN**: Declares TableGen def record `SVWHILERW_S`.
  **L1883 CN**: 声明 TableGen def 记录 `SVWHILERW_S`。
- **L1884 EN**: Declares TableGen def record `SVWHILERW_D`.
  **L1884 CN**: 声明 TableGen def 记录 `SVWHILERW_D`。
- **L1885 EN**: Blank line separating nearby declarations or logic blocks.
  **L1885 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1886 EN**: Declares TableGen def record `SVWHILEWR_B`.
  **L1886 CN**: 声明 TableGen def 记录 `SVWHILEWR_B`。
- **L1887 EN**: Declares TableGen def record `SVWHILEWR_H`.
  **L1887 CN**: 声明 TableGen def 记录 `SVWHILEWR_H`。
- **L1888 EN**: Declares TableGen def record `SVWHILEWR_S`.
  **L1888 CN**: 声明 TableGen def 记录 `SVWHILEWR_S`。
- **L1889 EN**: Declares TableGen def record `SVWHILEWR_D`.
  **L1889 CN**: 声明 TableGen def 记录 `SVWHILEWR_D`。
- **L1890 EN**: Closes the current lexical scope or compound statement.
  **L1890 CN**: 结束当前词法作用域或复合语句块。
- **L1891 EN**: Blank line separating nearby declarations or logic blocks.
  **L1891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1892 EN**: Separator comment used for visual grouping.
  **L1892 CN**: 用于视觉分组的分隔注释。
- **L1893 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Extended table lookup/permute`.
  **L1893 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Extended table lookup/permute`。
- **L1894 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2|sme" in {`.
  **L1894 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2|sme" in {`。
- **L1895 EN**: Declares TableGen def record `SVTBL2`.
  **L1895 CN**: 声明 TableGen def 记录 `SVTBL2`。
- **L1896 EN**: Declares TableGen def record `SVTBX`.
  **L1896 CN**: 声明 TableGen def 记录 `SVTBX`。
- **L1897 EN**: Closes the current lexical scope or compound statement.
  **L1897 CN**: 结束当前词法作用域或复合语句块。
- **L1898 EN**: Blank line separating nearby declarations or logic blocks.
  **L1898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1899 EN**: Separator comment used for visual grouping.
  **L1899 CN**: 用于视觉分组的分隔注释。
- **L1900 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Lookup table`.
  **L1900 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Lookup table`。
- **L1901 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "(sve2|sme2),lut", SMETargetGuard = "sme2,lut" in {`.
  **L1901 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "(sve2|sme2),lut", SMETargetGuard = "sme2,lut" in {`。
- **L1902 EN**: Declares TableGen def record `SVLUTI2_B`.
  **L1902 CN**: 声明 TableGen def 记录 `SVLUTI2_B`。
- **L1903 EN**: Declares TableGen def record `SVLUTI2_H`.
  **L1903 CN**: 声明 TableGen def 记录 `SVLUTI2_H`。
- **L1904 EN**: Blank line separating nearby declarations or logic blocks.
  **L1904 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1905-1932

````tablegen
  def SVLUTI4_B : SInst<"svluti4_lane[_{d}]", "dd[i", "cUc",   MergeNone, "aarch64_sve_luti4_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_1>]>;
  def SVLUTI4_H : SInst<"svluti4_lane[_{d}]", "dd[i", "sUshb", MergeNone, "aarch64_sve_luti4_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;

  def SVLUTI4_x2 : SInst<"svluti4_lane[_{d}_x2]", "d2.d[i", "sUshb", MergeNone, "aarch64_sve_luti4_lane_x2", [VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SVE2 - Optional

let SVETargetGuard = "sve-aes", SMETargetGuard = "ssve-aes" in {
def SVAESD   : SInst<"svaesd[_{d}]",   "ddd", "Uc", MergeNone, "aarch64_sve_aesd", [IsOverloadNone, VerifyRuntimeMode]>;
def SVAESIMC : SInst<"svaesimc[_{d}]", "dd",  "Uc", MergeNone, "aarch64_sve_aesimc", [IsOverloadNone, VerifyRuntimeMode]>;
def SVAESE   : SInst<"svaese[_{d}]",   "ddd", "Uc", MergeNone, "aarch64_sve_aese", [IsOverloadNone, VerifyRuntimeMode]>;
def SVAESMC  : SInst<"svaesmc[_{d}]",  "dd",  "Uc", MergeNone, "aarch64_sve_aesmc", [IsOverloadNone, VerifyRuntimeMode]>;

def SVPMULLB_PAIR_U64   : SInst<"svpmullb_pair[_{d}]",   "ddd", "Ul", MergeNone, "aarch64_sve_pmullb_pair", [VerifyRuntimeMode]>;
def SVPMULLB_PAIR_N_U64 : SInst<"svpmullb_pair[_n_{d}]", "dda", "Ul", MergeNone, "aarch64_sve_pmullb_pair", [VerifyRuntimeMode]>;

def SVPMULLT_PAIR_U64   : SInst<"svpmullt_pair[_{d}]",   "ddd", "Ul", MergeNone, "aarch64_sve_pmullt_pair", [VerifyRuntimeMode]>;
def SVPMULLT_PAIR_N_U64 : SInst<"svpmullt_pair[_n_{d}]", "dda", "Ul", MergeNone, "aarch64_sve_pmullt_pair", [VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve-aes2", SMETargetGuard = "sve-aes2,ssve-aes" in {
def SVAESD_X2   : SInst<"svaesd_lane[_{d}_x2]",   "22di", "Uc", MergeNone, "aarch64_sve_aesd_lane_x2", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
def SVAESDIMC_X2 : SInst<"svaesdimc_lane[_{d}_x2]", "22di",  "Uc", MergeNone, "aarch64_sve_aesdimc_lane_x2", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
def SVAESE_X2   : SInst<"svaese_lane[_{d}_x2]",   "22di", "Uc", MergeNone, "aarch64_sve_aese_lane_x2", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
def SVAESEMC_X2  : SInst<"svaesemc_lane[_{d}_x2]",  "22di",  "Uc", MergeNone, "aarch64_sve_aesemc_lane_x2", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;

````
- **L1905 EN**: Declares TableGen def record `SVLUTI4_B`.
  **L1905 CN**: 声明 TableGen def 记录 `SVLUTI4_B`。
- **L1906 EN**: Declares TableGen def record `SVLUTI4_H`.
  **L1906 CN**: 声明 TableGen def 记录 `SVLUTI4_H`。
- **L1907 EN**: Blank line separating nearby declarations or logic blocks.
  **L1907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1908 EN**: Declares TableGen def record `SVLUTI4_x2`.
  **L1908 CN**: 声明 TableGen def 记录 `SVLUTI4_x2`。
- **L1909 EN**: Closes the current lexical scope or compound statement.
  **L1909 CN**: 结束当前词法作用域或复合语句块。
- **L1910 EN**: Blank line separating nearby declarations or logic blocks.
  **L1910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1911 EN**: Separator comment used for visual grouping.
  **L1911 CN**: 用于视觉分组的分隔注释。
- **L1912 EN**: Comment explains nearby logic, constraints, or intent: `SVE2 - Optional`.
  **L1912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE2 - Optional`。
- **L1913 EN**: Blank line separating nearby declarations or logic blocks.
  **L1913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1914 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-aes", SMETargetGuard = "ssve-aes" in {`.
  **L1914 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-aes", SMETargetGuard = "ssve-aes" in {`。
- **L1915 EN**: Declares TableGen def record `SVAESD`.
  **L1915 CN**: 声明 TableGen def 记录 `SVAESD`。
- **L1916 EN**: Declares TableGen def record `SVAESIMC`.
  **L1916 CN**: 声明 TableGen def 记录 `SVAESIMC`。
- **L1917 EN**: Declares TableGen def record `SVAESE`.
  **L1917 CN**: 声明 TableGen def 记录 `SVAESE`。
- **L1918 EN**: Declares TableGen def record `SVAESMC`.
  **L1918 CN**: 声明 TableGen def 记录 `SVAESMC`。
- **L1919 EN**: Blank line separating nearby declarations or logic blocks.
  **L1919 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1920 EN**: Declares TableGen def record `SVPMULLB_PAIR_U64`.
  **L1920 CN**: 声明 TableGen def 记录 `SVPMULLB_PAIR_U64`。
- **L1921 EN**: Declares TableGen def record `SVPMULLB_PAIR_N_U64`.
  **L1921 CN**: 声明 TableGen def 记录 `SVPMULLB_PAIR_N_U64`。
- **L1922 EN**: Blank line separating nearby declarations or logic blocks.
  **L1922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1923 EN**: Declares TableGen def record `SVPMULLT_PAIR_U64`.
  **L1923 CN**: 声明 TableGen def 记录 `SVPMULLT_PAIR_U64`。
- **L1924 EN**: Declares TableGen def record `SVPMULLT_PAIR_N_U64`.
  **L1924 CN**: 声明 TableGen def 记录 `SVPMULLT_PAIR_N_U64`。
- **L1925 EN**: Closes the current lexical scope or compound statement.
  **L1925 CN**: 结束当前词法作用域或复合语句块。
- **L1926 EN**: Blank line separating nearby declarations or logic blocks.
  **L1926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1927 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-aes2", SMETargetGuard = "sve-aes2,ssve-aes" in {`.
  **L1927 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-aes2", SMETargetGuard = "sve-aes2,ssve-aes" in {`。
- **L1928 EN**: Declares TableGen def record `SVAESD_X2`.
  **L1928 CN**: 声明 TableGen def 记录 `SVAESD_X2`。
- **L1929 EN**: Declares TableGen def record `SVAESDIMC_X2`.
  **L1929 CN**: 声明 TableGen def 记录 `SVAESDIMC_X2`。
- **L1930 EN**: Declares TableGen def record `SVAESE_X2`.
  **L1930 CN**: 声明 TableGen def 记录 `SVAESE_X2`。
- **L1931 EN**: Declares TableGen def record `SVAESEMC_X2`.
  **L1931 CN**: 声明 TableGen def 记录 `SVAESEMC_X2`。
- **L1932 EN**: Blank line separating nearby declarations or logic blocks.
  **L1932 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1933-1960

````tablegen
def SVAESD_X4   : SInst<"svaesd_lane[_{d}_x4]",   "44di", "Uc", MergeNone, "aarch64_sve_aesd_lane_x4", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
def SVAESDIMC_X4 : SInst<"svaesdimc_lane[_{d}_x4]", "44di",  "Uc", MergeNone, "aarch64_sve_aesdimc_lane_x4", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
def SVAESE_X4   : SInst<"svaese_lane[_{d}_x4]",   "44di", "Uc", MergeNone, "aarch64_sve_aese_lane_x4", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;
def SVAESEMC_X4  : SInst<"svaesemc_lane[_{d}_x4]",  "44di",  "Uc", MergeNone, "aarch64_sve_aesemc_lane_x4", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck0_3>]>;

def SVPMULL_PAIR_U64   : SInst<"svpmull_pair[_{d}_x2]",   "2dd", "Ul", MergeNone, "aarch64_sve_pmull_pair_x2", [IsOverloadNone, VerifyRuntimeMode]>;
def SVPMULL_PAIR_N_U64 : SInst<"svpmull_pair[_n_{d}_x2]", "2da", "Ul", MergeNone, "aarch64_sve_pmull_pair_x2", [IsOverloadNone, VerifyRuntimeMode]>;
def SVPMLAL_PAIR_U64   : SInst<"svpmlal_pair[_{d}_x2]",   "22dd", "Ul", MergeNone, "aarch64_sve_pmlal_pair_x2", [IsOverloadNone, VerifyRuntimeMode]>;
def SVPMLAL_PAIR_N_U64 : SInst<"svpmlal_pair[_n_{d}_x2]", "22da", "Ul", MergeNone, "aarch64_sve_pmlal_pair_x2", [IsOverloadNone, VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve-sha3", SMETargetGuard = "sme2p1,sve-sha3" in {
def SVRAX1 : SInst<"svrax1[_{d}]",   "ddd", "lUl", MergeNone, "aarch64_sve_rax1", [IsOverloadNone, VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve-sm4", SMETargetGuard = InvalidMode in {
def SVSM4E    : SInst<"svsm4e[_{d}]",    "ddd", "Ui", MergeNone, "aarch64_sve_sm4e", [IsOverloadNone]>;
def SVSM4EKEY : SInst<"svsm4ekey[_{d}]", "ddd", "Ui", MergeNone, "aarch64_sve_sm4ekey", [IsOverloadNone]>;
}

let SVETargetGuard = "sve-bitperm", SMETargetGuard = "ssve-bitperm" in {
def SVBDEP   : SInst<"svbdep[_{d}]",   "ddd", "UcUsUiUl", MergeNone, "aarch64_sve_bdep_x", [VerifyRuntimeMode]>;
def SVBDEP_N : SInst<"svbdep[_n_{d}]", "dda", "UcUsUiUl", MergeNone, "aarch64_sve_bdep_x", [VerifyRuntimeMode]>;
def SVBEXT   : SInst<"svbext[_{d}]",   "ddd", "UcUsUiUl", MergeNone, "aarch64_sve_bext_x", [VerifyRuntimeMode]>;
def SVBEXT_N : SInst<"svbext[_n_{d}]", "dda", "UcUsUiUl", MergeNone, "aarch64_sve_bext_x", [VerifyRuntimeMode]>;
def SVBGRP   : SInst<"svbgrp[_{d}]",   "ddd", "UcUsUiUl", MergeNone, "aarch64_sve_bgrp_x", [VerifyRuntimeMode]>;
def SVBGRP_N : SInst<"svbgrp[_n_{d}]", "dda", "UcUsUiUl", MergeNone, "aarch64_sve_bgrp_x", [VerifyRuntimeMode]>;
}
````
- **L1933 EN**: Declares TableGen def record `SVAESD_X4`.
  **L1933 CN**: 声明 TableGen def 记录 `SVAESD_X4`。
- **L1934 EN**: Declares TableGen def record `SVAESDIMC_X4`.
  **L1934 CN**: 声明 TableGen def 记录 `SVAESDIMC_X4`。
- **L1935 EN**: Declares TableGen def record `SVAESE_X4`.
  **L1935 CN**: 声明 TableGen def 记录 `SVAESE_X4`。
- **L1936 EN**: Declares TableGen def record `SVAESEMC_X4`.
  **L1936 CN**: 声明 TableGen def 记录 `SVAESEMC_X4`。
- **L1937 EN**: Blank line separating nearby declarations or logic blocks.
  **L1937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1938 EN**: Declares TableGen def record `SVPMULL_PAIR_U64`.
  **L1938 CN**: 声明 TableGen def 记录 `SVPMULL_PAIR_U64`。
- **L1939 EN**: Declares TableGen def record `SVPMULL_PAIR_N_U64`.
  **L1939 CN**: 声明 TableGen def 记录 `SVPMULL_PAIR_N_U64`。
- **L1940 EN**: Declares TableGen def record `SVPMLAL_PAIR_U64`.
  **L1940 CN**: 声明 TableGen def 记录 `SVPMLAL_PAIR_U64`。
- **L1941 EN**: Declares TableGen def record `SVPMLAL_PAIR_N_U64`.
  **L1941 CN**: 声明 TableGen def 记录 `SVPMLAL_PAIR_N_U64`。
- **L1942 EN**: Closes the current lexical scope or compound statement.
  **L1942 CN**: 结束当前词法作用域或复合语句块。
- **L1943 EN**: Blank line separating nearby declarations or logic blocks.
  **L1943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1944 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-sha3", SMETargetGuard = "sme2p1,sve-sha3" in {`.
  **L1944 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-sha3", SMETargetGuard = "sme2p1,sve-sha3" in {`。
- **L1945 EN**: Declares TableGen def record `SVRAX1`.
  **L1945 CN**: 声明 TableGen def 记录 `SVRAX1`。
- **L1946 EN**: Closes the current lexical scope or compound statement.
  **L1946 CN**: 结束当前词法作用域或复合语句块。
- **L1947 EN**: Blank line separating nearby declarations or logic blocks.
  **L1947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1948 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-sm4", SMETargetGuard = InvalidMode in {`.
  **L1948 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-sm4", SMETargetGuard = InvalidMode in {`。
- **L1949 EN**: Declares TableGen def record `SVSM4E`.
  **L1949 CN**: 声明 TableGen def 记录 `SVSM4E`。
- **L1950 EN**: Declares TableGen def record `SVSM4EKEY`.
  **L1950 CN**: 声明 TableGen def 记录 `SVSM4EKEY`。
- **L1951 EN**: Closes the current lexical scope or compound statement.
  **L1951 CN**: 结束当前词法作用域或复合语句块。
- **L1952 EN**: Blank line separating nearby declarations or logic blocks.
  **L1952 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1953 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-bitperm", SMETargetGuard = "ssve-bitperm" in {`.
  **L1953 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-bitperm", SMETargetGuard = "ssve-bitperm" in {`。
- **L1954 EN**: Declares TableGen def record `SVBDEP`.
  **L1954 CN**: 声明 TableGen def 记录 `SVBDEP`。
- **L1955 EN**: Declares TableGen def record `SVBDEP_N`.
  **L1955 CN**: 声明 TableGen def 记录 `SVBDEP_N`。
- **L1956 EN**: Declares TableGen def record `SVBEXT`.
  **L1956 CN**: 声明 TableGen def 记录 `SVBEXT`。
- **L1957 EN**: Declares TableGen def record `SVBEXT_N`.
  **L1957 CN**: 声明 TableGen def 记录 `SVBEXT_N`。
- **L1958 EN**: Declares TableGen def record `SVBGRP`.
  **L1958 CN**: 声明 TableGen def 记录 `SVBGRP`。
- **L1959 EN**: Declares TableGen def record `SVBGRP_N`.
  **L1959 CN**: 声明 TableGen def 记录 `SVBGRP_N`。
- **L1960 EN**: Closes the current lexical scope or compound statement.
  **L1960 CN**: 结束当前词法作用域或复合语句块。

### Lines 1961-1988

````tablegen

let SVETargetGuard = "sve2p1|sme" in {
def SVPSEL_B : SInst<"svpsel_lane_b8",  "PPPm", "Pc", MergeNone, "", [VerifyRuntimeMode], []>;
def SVPSEL_H : SInst<"svpsel_lane_b16", "PPPm", "Ps", MergeNone, "", [VerifyRuntimeMode], []>;
def SVPSEL_S : SInst<"svpsel_lane_b32", "PPPm", "Pi", MergeNone, "", [VerifyRuntimeMode], []>;
def SVPSEL_D : SInst<"svpsel_lane_b64", "PPPm", "Pl", MergeNone, "", [VerifyRuntimeMode], []>;
}

// Standalone sve2.1 builtins
let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {
def SVORQV   : SInst<"svorqv[_{d}]",  "{Pd", "csilUcUsUiUl", MergeNone, "aarch64_sve_orqv",   [IsReductionQV, VerifyRuntimeMode]>;
def SVEORQV  : SInst<"sveorqv[_{d}]", "{Pd", "csilUcUsUiUl", MergeNone, "aarch64_sve_eorqv",  [IsReductionQV, VerifyRuntimeMode]>;
def SVADDQV  : SInst<"svaddqv[_{d}]", "{Pd", "csilUcUsUiUl", MergeNone, "aarch64_sve_addqv",  [IsReductionQV, VerifyRuntimeMode]>;
def SVANDQV  : SInst<"svandqv[_{d}]", "{Pd", "csilUcUsUiUl", MergeNone, "aarch64_sve_andqv",  [IsReductionQV, VerifyRuntimeMode]>;
def SVSMAXQV : SInst<"svmaxqv[_{d}]", "{Pd", "csil",         MergeNone, "aarch64_sve_smaxqv", [IsReductionQV, VerifyRuntimeMode]>;
def SVUMAXQV : SInst<"svmaxqv[_{d}]", "{Pd", "UcUsUiUl",     MergeNone, "aarch64_sve_umaxqv", [IsReductionQV, VerifyRuntimeMode]>;
def SVSMINQV : SInst<"svminqv[_{d}]", "{Pd", "csil",         MergeNone, "aarch64_sve_sminqv", [IsReductionQV, VerifyRuntimeMode]>;
def SVUMINQV : SInst<"svminqv[_{d}]", "{Pd", "UcUsUiUl",     MergeNone, "aarch64_sve_uminqv", [IsReductionQV, VerifyRuntimeMode]>;

def SVFADDQV   : SInst<"svaddqv[_{d}]",   "{Pd", "hfd", MergeNone, "aarch64_sve_faddqv",   [IsReductionQV, VerifyRuntimeMode]>;
def SVFMAXNMQV : SInst<"svmaxnmqv[_{d}]", "{Pd", "hfd", MergeNone, "aarch64_sve_fmaxnmqv", [IsReductionQV, VerifyRuntimeMode]>;
def SVFMINNMQV : SInst<"svminnmqv[_{d}]", "{Pd", "hfd", MergeNone, "aarch64_sve_fminnmqv", [IsReductionQV, VerifyRuntimeMode]>;
def SVFMAXQV   : SInst<"svmaxqv[_{d}]",   "{Pd", "hfd", MergeNone, "aarch64_sve_fmaxqv",   [IsReductionQV, VerifyRuntimeMode]>;
def SVFMINQV   : SInst<"svminqv[_{d}]",   "{Pd", "hfd", MergeNone, "aarch64_sve_fminqv",   [IsReductionQV, VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {
def SVPEXT_SINGLE : SInst<"svpext_lane_{d}", "P}i",      "QcQsQiQl", MergeNone, "aarch64_sve_pext",    [VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_3>]>;
````
- **L1961 EN**: Blank line separating nearby declarations or logic blocks.
  **L1961 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1962 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme" in {`.
  **L1962 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme" in {`。
- **L1963 EN**: Declares TableGen def record `SVPSEL_B`.
  **L1963 CN**: 声明 TableGen def 记录 `SVPSEL_B`。
- **L1964 EN**: Declares TableGen def record `SVPSEL_H`.
  **L1964 CN**: 声明 TableGen def 记录 `SVPSEL_H`。
- **L1965 EN**: Declares TableGen def record `SVPSEL_S`.
  **L1965 CN**: 声明 TableGen def 记录 `SVPSEL_S`。
- **L1966 EN**: Declares TableGen def record `SVPSEL_D`.
  **L1966 CN**: 声明 TableGen def 记录 `SVPSEL_D`。
- **L1967 EN**: Closes the current lexical scope or compound statement.
  **L1967 CN**: 结束当前词法作用域或复合语句块。
- **L1968 EN**: Blank line separating nearby declarations or logic blocks.
  **L1968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1969 EN**: Comment explains nearby logic, constraints, or intent: `Standalone sve2.1 builtins`.
  **L1969 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Standalone sve2.1 builtins`。
- **L1970 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`.
  **L1970 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`。
- **L1971 EN**: Declares TableGen def record `SVORQV`.
  **L1971 CN**: 声明 TableGen def 记录 `SVORQV`。
- **L1972 EN**: Declares TableGen def record `SVEORQV`.
  **L1972 CN**: 声明 TableGen def 记录 `SVEORQV`。
- **L1973 EN**: Declares TableGen def record `SVADDQV`.
  **L1973 CN**: 声明 TableGen def 记录 `SVADDQV`。
- **L1974 EN**: Declares TableGen def record `SVANDQV`.
  **L1974 CN**: 声明 TableGen def 记录 `SVANDQV`。
- **L1975 EN**: Declares TableGen def record `SVSMAXQV`.
  **L1975 CN**: 声明 TableGen def 记录 `SVSMAXQV`。
- **L1976 EN**: Declares TableGen def record `SVUMAXQV`.
  **L1976 CN**: 声明 TableGen def 记录 `SVUMAXQV`。
- **L1977 EN**: Declares TableGen def record `SVSMINQV`.
  **L1977 CN**: 声明 TableGen def 记录 `SVSMINQV`。
- **L1978 EN**: Declares TableGen def record `SVUMINQV`.
  **L1978 CN**: 声明 TableGen def 记录 `SVUMINQV`。
- **L1979 EN**: Blank line separating nearby declarations or logic blocks.
  **L1979 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1980 EN**: Declares TableGen def record `SVFADDQV`.
  **L1980 CN**: 声明 TableGen def 记录 `SVFADDQV`。
- **L1981 EN**: Declares TableGen def record `SVFMAXNMQV`.
  **L1981 CN**: 声明 TableGen def 记录 `SVFMAXNMQV`。
- **L1982 EN**: Declares TableGen def record `SVFMINNMQV`.
  **L1982 CN**: 声明 TableGen def 记录 `SVFMINNMQV`。
- **L1983 EN**: Declares TableGen def record `SVFMAXQV`.
  **L1983 CN**: 声明 TableGen def 记录 `SVFMAXQV`。
- **L1984 EN**: Declares TableGen def record `SVFMINQV`.
  **L1984 CN**: 声明 TableGen def 记录 `SVFMINQV`。
- **L1985 EN**: Closes the current lexical scope or compound statement.
  **L1985 CN**: 结束当前词法作用域或复合语句块。
- **L1986 EN**: Blank line separating nearby declarations or logic blocks.
  **L1986 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1987 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`.
  **L1987 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`。
- **L1988 EN**: Declares TableGen def record `SVPEXT_SINGLE`.
  **L1988 CN**: 声明 TableGen def 记录 `SVPEXT_SINGLE`。

### Lines 1989-2016

````tablegen
def SVPEXT_X2     : SInst<"svpext_lane_{d}_x2", "2.P}i", "QcQsQiQl", MergeNone, "aarch64_sve_pext_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_1>]>;

def SVPSEL_COUNT_ALIAS_B : SInst<"svpsel_lane_c8",  "}}Pm", "Pc", MergeNone, "", [VerifyRuntimeMode], []>;
def SVPSEL_COUNT_ALIAS_H : SInst<"svpsel_lane_c16", "}}Pm", "Ps", MergeNone, "", [VerifyRuntimeMode], []>;
def SVPSEL_COUNT_ALIAS_S : SInst<"svpsel_lane_c32", "}}Pm", "Pi", MergeNone, "", [VerifyRuntimeMode], []>;
def SVPSEL_COUNT_ALIAS_D : SInst<"svpsel_lane_c64", "}}Pm", "Pl", MergeNone, "", [VerifyRuntimeMode], []>;

def SVWHILEGE_COUNT : SInst<"svwhilege_{d}[_{1}]",  "}lli", "QcQsQiQl", MergeNone, "aarch64_sve_whilege_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
def SVWHILEGT_COUNT : SInst<"svwhilegt_{d}[_{1}]",  "}lli", "QcQsQiQl", MergeNone, "aarch64_sve_whilegt_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
def SVWHILELE_COUNT : SInst<"svwhilele_{d}[_{1}]",  "}lli", "QcQsQiQl", MergeNone, "aarch64_sve_whilele_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
def SVWHILELT_COUNT : SInst<"svwhilelt_{d}[_{1}]",  "}lli", "QcQsQiQl", MergeNone, "aarch64_sve_whilelt_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
def SVWHILELO_COUNT : SInst<"svwhilelt_{d}[_{1}]",  "}nni", "QcQsQiQl", MergeNone, "aarch64_sve_whilelo_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
def SVWHILELS_COUNT : SInst<"svwhilele_{d}[_{1}]",  "}nni", "QcQsQiQl", MergeNone, "aarch64_sve_whilels_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
def SVWHILEHI_COUNT : SInst<"svwhilegt_{d}[_{1}]",  "}nni", "QcQsQiQl", MergeNone, "aarch64_sve_whilehi_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
def SVWHILEHS_COUNT : SInst<"svwhilege_{d}[_{1}]",  "}nni", "QcQsQiQl", MergeNone, "aarch64_sve_whilehs_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<2, ImmCheck2_4_Mul2>]>;
}

multiclass MultiVecLoad<string i> {
  def SV # NAME # B_X2 : MInst<"sv" # i # "[_{2}]_x2", "2}c", "cUcm",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # H_X2 : MInst<"sv" # i # "[_{2}]_x2", "2}c", "sUshb", [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # W_X2 : MInst<"sv" # i # "[_{2}]_x2", "2}c", "iUif",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # D_X2 : MInst<"sv" # i # "[_{2}]_x2", "2}c", "lUld",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # B_X4 : MInst<"sv" # i # "[_{2}]_x4", "4}c", "cUcm",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # H_X4 : MInst<"sv" # i # "[_{2}]_x4", "4}c", "sUshb", [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # W_X4 : MInst<"sv" # i # "[_{2}]_x4", "4}c", "iUif",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # D_X4 : MInst<"sv" # i # "[_{2}]_x4", "4}c", "lUld",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;

  def SV # NAME # B_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}]_x2", "2}cl", "cUcm",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
````
- **L1989 EN**: Declares TableGen def record `SVPEXT_X2`.
  **L1989 CN**: 声明 TableGen def 记录 `SVPEXT_X2`。
- **L1990 EN**: Blank line separating nearby declarations or logic blocks.
  **L1990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1991 EN**: Declares TableGen def record `SVPSEL_COUNT_ALIAS_B`.
  **L1991 CN**: 声明 TableGen def 记录 `SVPSEL_COUNT_ALIAS_B`。
- **L1992 EN**: Declares TableGen def record `SVPSEL_COUNT_ALIAS_H`.
  **L1992 CN**: 声明 TableGen def 记录 `SVPSEL_COUNT_ALIAS_H`。
- **L1993 EN**: Declares TableGen def record `SVPSEL_COUNT_ALIAS_S`.
  **L1993 CN**: 声明 TableGen def 记录 `SVPSEL_COUNT_ALIAS_S`。
- **L1994 EN**: Declares TableGen def record `SVPSEL_COUNT_ALIAS_D`.
  **L1994 CN**: 声明 TableGen def 记录 `SVPSEL_COUNT_ALIAS_D`。
- **L1995 EN**: Blank line separating nearby declarations or logic blocks.
  **L1995 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1996 EN**: Declares TableGen def record `SVWHILEGE_COUNT`.
  **L1996 CN**: 声明 TableGen def 记录 `SVWHILEGE_COUNT`。
- **L1997 EN**: Declares TableGen def record `SVWHILEGT_COUNT`.
  **L1997 CN**: 声明 TableGen def 记录 `SVWHILEGT_COUNT`。
- **L1998 EN**: Declares TableGen def record `SVWHILELE_COUNT`.
  **L1998 CN**: 声明 TableGen def 记录 `SVWHILELE_COUNT`。
- **L1999 EN**: Declares TableGen def record `SVWHILELT_COUNT`.
  **L1999 CN**: 声明 TableGen def 记录 `SVWHILELT_COUNT`。
- **L2000 EN**: Declares TableGen def record `SVWHILELO_COUNT`.
  **L2000 CN**: 声明 TableGen def 记录 `SVWHILELO_COUNT`。
- **L2001 EN**: Declares TableGen def record `SVWHILELS_COUNT`.
  **L2001 CN**: 声明 TableGen def 记录 `SVWHILELS_COUNT`。
- **L2002 EN**: Declares TableGen def record `SVWHILEHI_COUNT`.
  **L2002 CN**: 声明 TableGen def 记录 `SVWHILEHI_COUNT`。
- **L2003 EN**: Declares TableGen def record `SVWHILEHS_COUNT`.
  **L2003 CN**: 声明 TableGen def 记录 `SVWHILEHS_COUNT`。
- **L2004 EN**: Closes the current lexical scope or compound statement.
  **L2004 CN**: 结束当前词法作用域或复合语句块。
- **L2005 EN**: Blank line separating nearby declarations or logic blocks.
  **L2005 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2006 EN**: Declares TableGen multiclass record `MultiVecLoad`.
  **L2006 CN**: 声明 TableGen multiclass 记录 `MultiVecLoad`。
- **L2007 EN**: Declares TableGen def record `SV # NAME # B_X2`.
  **L2007 CN**: 声明 TableGen def 记录 `SV # NAME # B_X2`。
- **L2008 EN**: Declares TableGen def record `SV # NAME # H_X2`.
  **L2008 CN**: 声明 TableGen def 记录 `SV # NAME # H_X2`。
- **L2009 EN**: Declares TableGen def record `SV # NAME # W_X2`.
  **L2009 CN**: 声明 TableGen def 记录 `SV # NAME # W_X2`。
- **L2010 EN**: Declares TableGen def record `SV # NAME # D_X2`.
  **L2010 CN**: 声明 TableGen def 记录 `SV # NAME # D_X2`。
- **L2011 EN**: Declares TableGen def record `SV # NAME # B_X4`.
  **L2011 CN**: 声明 TableGen def 记录 `SV # NAME # B_X4`。
- **L2012 EN**: Declares TableGen def record `SV # NAME # H_X4`.
  **L2012 CN**: 声明 TableGen def 记录 `SV # NAME # H_X4`。
- **L2013 EN**: Declares TableGen def record `SV # NAME # W_X4`.
  **L2013 CN**: 声明 TableGen def 记录 `SV # NAME # W_X4`。
- **L2014 EN**: Declares TableGen def record `SV # NAME # D_X4`.
  **L2014 CN**: 声明 TableGen def 记录 `SV # NAME # D_X4`。
- **L2015 EN**: Blank line separating nearby declarations or logic blocks.
  **L2015 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2016 EN**: Declares TableGen def record `SV # NAME # B_VNUM_X2`.
  **L2016 CN**: 声明 TableGen def 记录 `SV # NAME # B_VNUM_X2`。

### Lines 2017-2044

````tablegen
  def SV # NAME # H_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}]_x2", "2}cl", "sUshb", [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # W_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}]_x2", "2}cl", "iUif",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # D_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}]_x2", "2}cl", "lUld",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # B_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}]_x4", "4}cl", "cUcm",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # H_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}]_x4", "4}cl", "sUshb", [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # W_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}]_x4", "4}cl", "iUif",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # D_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}]_x4", "4}cl", "lUld",  [IsStructLoad, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
}

let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {
  defm LD1   : MultiVecLoad<"ld1">;
  defm LDNT1 : MultiVecLoad<"ldnt1">;
}

multiclass MultiVecStore<string i> {
  def SV # NAME # B_X2 : MInst<"sv" # i # "[_{2}_x2]", "v}p2", "cUcm",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # H_X2 : MInst<"sv" # i # "[_{2}_x2]", "v}p2", "sUshb", [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # W_X2 : MInst<"sv" # i # "[_{2}_x2]", "v}p2", "iUif",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # D_X2 : MInst<"sv" # i # "[_{2}_x2]", "v}p2", "lUld",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # B_X4 : MInst<"sv" # i # "[_{2}_x4]", "v}p4", "cUcm",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # H_X4 : MInst<"sv" # i # "[_{2}_x4]", "v}p4", "sUshb", [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # W_X4 : MInst<"sv" # i # "[_{2}_x4]", "v}p4", "iUif",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # D_X4 : MInst<"sv" # i # "[_{2}_x4]", "v}p4", "lUld",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;

  def SV # NAME # B_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}_x2]", "v}pl2", "cUcm",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # H_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}_x2]", "v}pl2", "sUshb", [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # W_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}_x2]", "v}pl2", "iUif",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
  def SV # NAME # D_VNUM_X2 : MInst<"sv" # i # "_vnum" # "[_{2}_x2]", "v}pl2", "lUld",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x2">;
````
- **L2017 EN**: Declares TableGen def record `SV # NAME # H_VNUM_X2`.
  **L2017 CN**: 声明 TableGen def 记录 `SV # NAME # H_VNUM_X2`。
- **L2018 EN**: Declares TableGen def record `SV # NAME # W_VNUM_X2`.
  **L2018 CN**: 声明 TableGen def 记录 `SV # NAME # W_VNUM_X2`。
- **L2019 EN**: Declares TableGen def record `SV # NAME # D_VNUM_X2`.
  **L2019 CN**: 声明 TableGen def 记录 `SV # NAME # D_VNUM_X2`。
- **L2020 EN**: Declares TableGen def record `SV # NAME # B_VNUM_X4`.
  **L2020 CN**: 声明 TableGen def 记录 `SV # NAME # B_VNUM_X4`。
- **L2021 EN**: Declares TableGen def record `SV # NAME # H_VNUM_X4`.
  **L2021 CN**: 声明 TableGen def 记录 `SV # NAME # H_VNUM_X4`。
- **L2022 EN**: Declares TableGen def record `SV # NAME # W_VNUM_X4`.
  **L2022 CN**: 声明 TableGen def 记录 `SV # NAME # W_VNUM_X4`。
- **L2023 EN**: Declares TableGen def record `SV # NAME # D_VNUM_X4`.
  **L2023 CN**: 声明 TableGen def 记录 `SV # NAME # D_VNUM_X4`。
- **L2024 EN**: Closes the current lexical scope or compound statement.
  **L2024 CN**: 结束当前词法作用域或复合语句块。
- **L2025 EN**: Blank line separating nearby declarations or logic blocks.
  **L2025 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2026 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2026 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2027 EN**: Declares TableGen defm record `LD1`.
  **L2027 CN**: 声明 TableGen defm 记录 `LD1`。
- **L2028 EN**: Declares TableGen defm record `LDNT1`.
  **L2028 CN**: 声明 TableGen defm 记录 `LDNT1`。
- **L2029 EN**: Closes the current lexical scope or compound statement.
  **L2029 CN**: 结束当前词法作用域或复合语句块。
- **L2030 EN**: Blank line separating nearby declarations or logic blocks.
  **L2030 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2031 EN**: Declares TableGen multiclass record `MultiVecStore`.
  **L2031 CN**: 声明 TableGen multiclass 记录 `MultiVecStore`。
- **L2032 EN**: Declares TableGen def record `SV # NAME # B_X2`.
  **L2032 CN**: 声明 TableGen def 记录 `SV # NAME # B_X2`。
- **L2033 EN**: Declares TableGen def record `SV # NAME # H_X2`.
  **L2033 CN**: 声明 TableGen def 记录 `SV # NAME # H_X2`。
- **L2034 EN**: Declares TableGen def record `SV # NAME # W_X2`.
  **L2034 CN**: 声明 TableGen def 记录 `SV # NAME # W_X2`。
- **L2035 EN**: Declares TableGen def record `SV # NAME # D_X2`.
  **L2035 CN**: 声明 TableGen def 记录 `SV # NAME # D_X2`。
- **L2036 EN**: Declares TableGen def record `SV # NAME # B_X4`.
  **L2036 CN**: 声明 TableGen def 记录 `SV # NAME # B_X4`。
- **L2037 EN**: Declares TableGen def record `SV # NAME # H_X4`.
  **L2037 CN**: 声明 TableGen def 记录 `SV # NAME # H_X4`。
- **L2038 EN**: Declares TableGen def record `SV # NAME # W_X4`.
  **L2038 CN**: 声明 TableGen def 记录 `SV # NAME # W_X4`。
- **L2039 EN**: Declares TableGen def record `SV # NAME # D_X4`.
  **L2039 CN**: 声明 TableGen def 记录 `SV # NAME # D_X4`。
- **L2040 EN**: Blank line separating nearby declarations or logic blocks.
  **L2040 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2041 EN**: Declares TableGen def record `SV # NAME # B_VNUM_X2`.
  **L2041 CN**: 声明 TableGen def 记录 `SV # NAME # B_VNUM_X2`。
- **L2042 EN**: Declares TableGen def record `SV # NAME # H_VNUM_X2`.
  **L2042 CN**: 声明 TableGen def 记录 `SV # NAME # H_VNUM_X2`。
- **L2043 EN**: Declares TableGen def record `SV # NAME # W_VNUM_X2`.
  **L2043 CN**: 声明 TableGen def 记录 `SV # NAME # W_VNUM_X2`。
- **L2044 EN**: Declares TableGen def record `SV # NAME # D_VNUM_X2`.
  **L2044 CN**: 声明 TableGen def 记录 `SV # NAME # D_VNUM_X2`。

### Lines 2045-2072

````tablegen
  def SV # NAME # B_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}_x4]", "v}pl4", "cUcm",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # H_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}_x4]", "v}pl4", "sUshb", [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # W_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}_x4]", "v}pl4", "iUif",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
  def SV # NAME # D_VNUM_X4 : MInst<"sv" # i # "_vnum" # "[_{2}_x4]", "v}pl4", "lUld",  [IsStructStore, VerifyRuntimeMode], MemEltTyDefault, "aarch64_sve_" # i # "_pn_x4">;
}

let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {
  defm ST1   : MultiVecStore<"st1">;
  defm STNT1 : MultiVecStore<"stnt1">;
}

let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {
def SVDOT_X2_S : SInst<"svdot[_{d}_{2}]", "ddhh", "i",  MergeNone, "aarch64_sve_sdot_x2", [VerifyRuntimeMode], []>;
def SVDOT_X2_U : SInst<"svdot[_{d}_{2}]", "ddhh", "Ui", MergeNone, "aarch64_sve_udot_x2", [VerifyRuntimeMode], []>;
def SVDOT_X2_F : SInst<"svdot[_{d}_{2}]", "ddhh", "f",  MergeNone, "aarch64_sve_fdot_x2", [VerifyRuntimeMode], []>;

def SVDOT_LANE_X2_S : SInst<"svdot_lane[_{d}_{2}]", "ddhhi", "i",  MergeNone, "aarch64_sve_sdot_lane_x2", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_3>]>;
def SVDOT_LANE_X2_U : SInst<"svdot_lane[_{d}_{2}]", "ddhhi", "Ui", MergeNone, "aarch64_sve_udot_lane_x2", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_3>]>;
def SVDOT_LANE_X2_F : SInst<"svdot_lane[_{d}_{2}]", "ddhhi", "f",  MergeNone, "aarch64_sve_fdot_lane_x2", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_3>]>;

def SVFCLAMP : SInst<"svclamp[_{d}]", "dddd", "hfd", MergeNone, "aarch64_sve_fclamp", [VerifyRuntimeMode], []>;
}

let SVETargetGuard = "sve2p1|sme" in {
def SVSCLAMP : SInst<"svclamp[_{d}]", "dddd", "csil",     MergeNone, "aarch64_sve_sclamp", [VerifyRuntimeMode], []>;
def SVUCLAMP : SInst<"svclamp[_{d}]", "dddd", "UcUsUiUl", MergeNone, "aarch64_sve_uclamp", [VerifyRuntimeMode], []>;

defm SVREVD : SInstZPZ<"svrevd", "csilUcUsUiUlmbhfd", "aarch64_sve_revd">;
````
- **L2045 EN**: Declares TableGen def record `SV # NAME # B_VNUM_X4`.
  **L2045 CN**: 声明 TableGen def 记录 `SV # NAME # B_VNUM_X4`。
- **L2046 EN**: Declares TableGen def record `SV # NAME # H_VNUM_X4`.
  **L2046 CN**: 声明 TableGen def 记录 `SV # NAME # H_VNUM_X4`。
- **L2047 EN**: Declares TableGen def record `SV # NAME # W_VNUM_X4`.
  **L2047 CN**: 声明 TableGen def 记录 `SV # NAME # W_VNUM_X4`。
- **L2048 EN**: Declares TableGen def record `SV # NAME # D_VNUM_X4`.
  **L2048 CN**: 声明 TableGen def 记录 `SV # NAME # D_VNUM_X4`。
- **L2049 EN**: Closes the current lexical scope or compound statement.
  **L2049 CN**: 结束当前词法作用域或复合语句块。
- **L2050 EN**: Blank line separating nearby declarations or logic blocks.
  **L2050 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2051 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2051 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2052 EN**: Declares TableGen defm record `ST1`.
  **L2052 CN**: 声明 TableGen defm 记录 `ST1`。
- **L2053 EN**: Declares TableGen defm record `STNT1`.
  **L2053 CN**: 声明 TableGen defm 记录 `STNT1`。
- **L2054 EN**: Closes the current lexical scope or compound statement.
  **L2054 CN**: 结束当前词法作用域或复合语句块。
- **L2055 EN**: Blank line separating nearby declarations or logic blocks.
  **L2055 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2056 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2056 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2057 EN**: Declares TableGen def record `SVDOT_X2_S`.
  **L2057 CN**: 声明 TableGen def 记录 `SVDOT_X2_S`。
- **L2058 EN**: Declares TableGen def record `SVDOT_X2_U`.
  **L2058 CN**: 声明 TableGen def 记录 `SVDOT_X2_U`。
- **L2059 EN**: Declares TableGen def record `SVDOT_X2_F`.
  **L2059 CN**: 声明 TableGen def 记录 `SVDOT_X2_F`。
- **L2060 EN**: Blank line separating nearby declarations or logic blocks.
  **L2060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2061 EN**: Declares TableGen def record `SVDOT_LANE_X2_S`.
  **L2061 CN**: 声明 TableGen def 记录 `SVDOT_LANE_X2_S`。
- **L2062 EN**: Declares TableGen def record `SVDOT_LANE_X2_U`.
  **L2062 CN**: 声明 TableGen def 记录 `SVDOT_LANE_X2_U`。
- **L2063 EN**: Declares TableGen def record `SVDOT_LANE_X2_F`.
  **L2063 CN**: 声明 TableGen def 记录 `SVDOT_LANE_X2_F`。
- **L2064 EN**: Blank line separating nearby declarations or logic blocks.
  **L2064 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2065 EN**: Declares TableGen def record `SVFCLAMP`.
  **L2065 CN**: 声明 TableGen def 记录 `SVFCLAMP`。
- **L2066 EN**: Closes the current lexical scope or compound statement.
  **L2066 CN**: 结束当前词法作用域或复合语句块。
- **L2067 EN**: Blank line separating nearby declarations or logic blocks.
  **L2067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2068 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme" in {`.
  **L2068 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme" in {`。
- **L2069 EN**: Declares TableGen def record `SVSCLAMP`.
  **L2069 CN**: 声明 TableGen def 记录 `SVSCLAMP`。
- **L2070 EN**: Declares TableGen def record `SVUCLAMP`.
  **L2070 CN**: 声明 TableGen def 记录 `SVUCLAMP`。
- **L2071 EN**: Blank line separating nearby declarations or logic blocks.
  **L2071 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2072 EN**: Declares TableGen defm record `SVREVD`.
  **L2072 CN**: 声明 TableGen defm 记录 `SVREVD`。

### Lines 2073-2100

````tablegen
}

let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {
  def SVPTRUE_COUNT : SInst<"svptrue_{d}", "}v", "QcQsQiQl", MergeNone, "aarch64_sve_ptrue_{d}", [IsOverloadNone, VerifyRuntimeMode], []>;
  def SVCNTP_COUNT  : SInst<"svcntp_{d}", "n}i", "QcQsQiQl", MergeNone, "aarch64_sve_cntp_{d}", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<1, ImmCheck2_4_Mul2>]>;

  def SVPFALSE_COUNT_ALIAS : SInst<"svpfalse_c", "}v", "", MergeNone, "", [IsOverloadNone, VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve-b16b16", SMETargetGuard = "sme2,sve-b16b16" in {
defm SVMUL_BF   : SInstZPZZ<"svmul",   "b", "aarch64_sve_fmul",   "aarch64_sve_fmul_u", [VerifyRuntimeMode]>;
defm SVADD_BF   : SInstZPZZ<"svadd",   "b", "aarch64_sve_fadd",   "aarch64_sve_fadd_u", [VerifyRuntimeMode]>;
defm SVSUB_BF   : SInstZPZZ<"svsub",   "b", "aarch64_sve_fsub",   "aarch64_sve_fsub_u", [VerifyRuntimeMode]>;
defm SVMAXNM_BF : SInstZPZZ<"svmaxnm", "b", "aarch64_sve_fmaxnm", "aarch64_sve_fmaxnm_u", [VerifyRuntimeMode]>;
defm SVMINNM_BF : SInstZPZZ<"svminnm", "b", "aarch64_sve_fminnm", "aarch64_sve_fminnm_u", [VerifyRuntimeMode]>;
defm SVMAX_BF   : SInstZPZZ<"svmax",   "b", "aarch64_sve_fmax",   "aarch64_sve_fmax_u", [VerifyRuntimeMode]>;
defm SVMIN_BF   : SInstZPZZ<"svmin",   "b", "aarch64_sve_fmin",   "aarch64_sve_fmin_u", [VerifyRuntimeMode]>;
defm SVMLA_BF   : SInstZPZZZ<"svmla",  "b", "aarch64_sve_fmla",   "aarch64_sve_fmla_u", [VerifyRuntimeMode]>;
defm SVMLS_BF   : SInstZPZZZ<"svmls",  "b", "aarch64_sve_fmls",   "aarch64_sve_fmls_u", [VerifyRuntimeMode]>;

def SVMLA_LANE_BF : SInst<"svmla_lane[_{d}]", "ddddi", "b", MergeNone, "aarch64_sve_fmla_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMLS_LANE_BF : SInst<"svmls_lane[_{d}]", "ddddi", "b", MergeNone, "aarch64_sve_fmls_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SVMUL_LANE_BF : SInst<"svmul_lane[_{d}]", "dddi",  "b", MergeNone, "aarch64_sve_fmul_lane", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 1>]>;

def SVFCLAMP_BF : SInst<"svclamp[_{d}]", "dddd", "b", MergeNone, "aarch64_sve_fclamp", [VerifyRuntimeMode], []>;
}

// SME2
````
- **L2073 EN**: Closes the current lexical scope or compound statement.
  **L2073 CN**: 结束当前词法作用域或复合语句块。
- **L2074 EN**: Blank line separating nearby declarations or logic blocks.
  **L2074 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2075 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2075 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2076 EN**: Declares TableGen def record `SVPTRUE_COUNT`.
  **L2076 CN**: 声明 TableGen def 记录 `SVPTRUE_COUNT`。
- **L2077 EN**: Declares TableGen def record `SVCNTP_COUNT`.
  **L2077 CN**: 声明 TableGen def 记录 `SVCNTP_COUNT`。
- **L2078 EN**: Blank line separating nearby declarations or logic blocks.
  **L2078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2079 EN**: Declares TableGen def record `SVPFALSE_COUNT_ALIAS`.
  **L2079 CN**: 声明 TableGen def 记录 `SVPFALSE_COUNT_ALIAS`。
- **L2080 EN**: Closes the current lexical scope or compound statement.
  **L2080 CN**: 结束当前词法作用域或复合语句块。
- **L2081 EN**: Blank line separating nearby declarations or logic blocks.
  **L2081 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2082 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve-b16b16", SMETargetGuard = "sme2,sve-b16b16" in {`.
  **L2082 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve-b16b16", SMETargetGuard = "sme2,sve-b16b16" in {`。
- **L2083 EN**: Declares TableGen defm record `SVMUL_BF`.
  **L2083 CN**: 声明 TableGen defm 记录 `SVMUL_BF`。
- **L2084 EN**: Declares TableGen defm record `SVADD_BF`.
  **L2084 CN**: 声明 TableGen defm 记录 `SVADD_BF`。
- **L2085 EN**: Declares TableGen defm record `SVSUB_BF`.
  **L2085 CN**: 声明 TableGen defm 记录 `SVSUB_BF`。
- **L2086 EN**: Declares TableGen defm record `SVMAXNM_BF`.
  **L2086 CN**: 声明 TableGen defm 记录 `SVMAXNM_BF`。
- **L2087 EN**: Declares TableGen defm record `SVMINNM_BF`.
  **L2087 CN**: 声明 TableGen defm 记录 `SVMINNM_BF`。
- **L2088 EN**: Declares TableGen defm record `SVMAX_BF`.
  **L2088 CN**: 声明 TableGen defm 记录 `SVMAX_BF`。
- **L2089 EN**: Declares TableGen defm record `SVMIN_BF`.
  **L2089 CN**: 声明 TableGen defm 记录 `SVMIN_BF`。
- **L2090 EN**: Declares TableGen defm record `SVMLA_BF`.
  **L2090 CN**: 声明 TableGen defm 记录 `SVMLA_BF`。
- **L2091 EN**: Declares TableGen defm record `SVMLS_BF`.
  **L2091 CN**: 声明 TableGen defm 记录 `SVMLS_BF`。
- **L2092 EN**: Blank line separating nearby declarations or logic blocks.
  **L2092 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2093 EN**: Declares TableGen def record `SVMLA_LANE_BF`.
  **L2093 CN**: 声明 TableGen def 记录 `SVMLA_LANE_BF`。
- **L2094 EN**: Declares TableGen def record `SVMLS_LANE_BF`.
  **L2094 CN**: 声明 TableGen def 记录 `SVMLS_LANE_BF`。
- **L2095 EN**: Declares TableGen def record `SVMUL_LANE_BF`.
  **L2095 CN**: 声明 TableGen def 记录 `SVMUL_LANE_BF`。
- **L2096 EN**: Blank line separating nearby declarations or logic blocks.
  **L2096 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2097 EN**: Declares TableGen def record `SVFCLAMP_BF`.
  **L2097 CN**: 声明 TableGen def 记录 `SVFCLAMP_BF`。
- **L2098 EN**: Closes the current lexical scope or compound statement.
  **L2098 CN**: 结束当前词法作用域或复合语句块。
- **L2099 EN**: Blank line separating nearby declarations or logic blocks.
  **L2099 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2100 EN**: Comment explains nearby logic, constraints, or intent: `SME2`.
  **L2100 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2`。

### Lines 2101-2128

````tablegen

// SME intrinsics which operate only on vectors and do not require ZA should be added here,
// as they could possibly become SVE instructions in the future.

multiclass MinMaxIntr<string i, string zm, string mul, string t> {
  def SVS # NAME : SInst<"sv" # i # "[" # zm # "_{d}_" # mul # "]", t, "csil",     MergeNone, "aarch64_sve_s" # i # zm # "_" # mul, [IsStreaming], []>;
  def SVU # NAME : SInst<"sv" # i # "[" # zm # "_{d}_" # mul # "]", t, "UcUsUiUl", MergeNone, "aarch64_sve_u" # i # zm # "_" # mul, [IsStreaming], []>;
  def SVF # NAME : SInst<"sv" # i # "[" # zm # "_{d}_" # mul # "]", t, "hfd",      MergeNone, "aarch64_sve_f" # i # zm # "_" # mul, [IsStreaming], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
// == SMAX / UMAX / FMAX ==
  defm MAX_SINGLE_X2 : MinMaxIntr<"max", "_single", "x2", "22d">;
  defm MAX_MULTI_X2  : MinMaxIntr<"max", "",        "x2", "222">;
  defm MAX_SINGLE_X4 : MinMaxIntr<"max", "_single", "x4", "44d">;
  defm MAX_MULTI_X4  : MinMaxIntr<"max", "",        "x4", "444">;

// == SMIN / UMIN / FMIN ==
  defm MIN_SINGLE_X2 : MinMaxIntr<"min", "_single", "x2", "22d">;
  defm MIN_MULTI_X2  : MinMaxIntr<"min", "",        "x2", "222">;
  defm MIN_SINGLE_X4 : MinMaxIntr<"min", "_single", "x4", "44d">;
  defm MIN_MULTI_X4  : MinMaxIntr<"min", "",        "x4", "444">;
}

multiclass SInstMinMaxByVector<string name> {
  def NAME # _SINGLE_X2 : SInst<"sv" # name # "nm[_single_{d}_x2]", "22d", "hfd", MergeNone, "aarch64_sve_f" # name # "nm_single_x2", [IsStreaming], []>;
  def NAME # _SINGLE_X4 : SInst<"sv" # name # "nm[_single_{d}_x4]", "44d", "hfd", MergeNone, "aarch64_sve_f" # name # "nm_single_x4", [IsStreaming], []>;

````
- **L2101 EN**: Blank line separating nearby declarations or logic blocks.
  **L2101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2102 EN**: Comment explains nearby logic, constraints, or intent: `SME intrinsics which operate only on vectors and do not require ZA should be added here,`.
  **L2102 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME intrinsics which operate only on vectors and do not require ZA should be added here,`。
- **L2103 EN**: Comment explains nearby logic, constraints, or intent: `as they could possibly become SVE instructions in the future.`.
  **L2103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`as they could possibly become SVE instructions in the future.`。
- **L2104 EN**: Blank line separating nearby declarations or logic blocks.
  **L2104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2105 EN**: Declares TableGen multiclass record `MinMaxIntr`.
  **L2105 CN**: 声明 TableGen multiclass 记录 `MinMaxIntr`。
- **L2106 EN**: Declares TableGen def record `SVS # NAME`.
  **L2106 CN**: 声明 TableGen def 记录 `SVS # NAME`。
- **L2107 EN**: Declares TableGen def record `SVU # NAME`.
  **L2107 CN**: 声明 TableGen def 记录 `SVU # NAME`。
- **L2108 EN**: Declares TableGen def record `SVF # NAME`.
  **L2108 CN**: 声明 TableGen def 记录 `SVF # NAME`。
- **L2109 EN**: Closes the current lexical scope or compound statement.
  **L2109 CN**: 结束当前词法作用域或复合语句块。
- **L2110 EN**: Blank line separating nearby declarations or logic blocks.
  **L2110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2111 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2111 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2112 EN**: Comment explains nearby logic, constraints, or intent: `SMAX / UMAX / FMAX`.
  **L2112 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SMAX / UMAX / FMAX`。
- **L2113 EN**: Declares TableGen defm record `MAX_SINGLE_X2`.
  **L2113 CN**: 声明 TableGen defm 记录 `MAX_SINGLE_X2`。
- **L2114 EN**: Declares TableGen defm record `MAX_MULTI_X2`.
  **L2114 CN**: 声明 TableGen defm 记录 `MAX_MULTI_X2`。
- **L2115 EN**: Declares TableGen defm record `MAX_SINGLE_X4`.
  **L2115 CN**: 声明 TableGen defm 记录 `MAX_SINGLE_X4`。
- **L2116 EN**: Declares TableGen defm record `MAX_MULTI_X4`.
  **L2116 CN**: 声明 TableGen defm 记录 `MAX_MULTI_X4`。
- **L2117 EN**: Blank line separating nearby declarations or logic blocks.
  **L2117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2118 EN**: Comment explains nearby logic, constraints, or intent: `SMIN / UMIN / FMIN`.
  **L2118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SMIN / UMIN / FMIN`。
- **L2119 EN**: Declares TableGen defm record `MIN_SINGLE_X2`.
  **L2119 CN**: 声明 TableGen defm 记录 `MIN_SINGLE_X2`。
- **L2120 EN**: Declares TableGen defm record `MIN_MULTI_X2`.
  **L2120 CN**: 声明 TableGen defm 记录 `MIN_MULTI_X2`。
- **L2121 EN**: Declares TableGen defm record `MIN_SINGLE_X4`.
  **L2121 CN**: 声明 TableGen defm 记录 `MIN_SINGLE_X4`。
- **L2122 EN**: Declares TableGen defm record `MIN_MULTI_X4`.
  **L2122 CN**: 声明 TableGen defm 记录 `MIN_MULTI_X4`。
- **L2123 EN**: Closes the current lexical scope or compound statement.
  **L2123 CN**: 结束当前词法作用域或复合语句块。
- **L2124 EN**: Blank line separating nearby declarations or logic blocks.
  **L2124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2125 EN**: Declares TableGen multiclass record `SInstMinMaxByVector`.
  **L2125 CN**: 声明 TableGen multiclass 记录 `SInstMinMaxByVector`。
- **L2126 EN**: Declares TableGen def record `NAME # _SINGLE_X2`.
  **L2126 CN**: 声明 TableGen def 记录 `NAME # _SINGLE_X2`。
- **L2127 EN**: Declares TableGen def record `NAME # _SINGLE_X4`.
  **L2127 CN**: 声明 TableGen def 记录 `NAME # _SINGLE_X4`。
- **L2128 EN**: Blank line separating nearby declarations or logic blocks.
  **L2128 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2129-2156

````tablegen
  def NAME # _X2 : SInst<"sv" # name # "nm[_{d}_x2]", "222", "hfd", MergeNone, "aarch64_sve_f" # name # "nm_x2", [IsStreaming], []>;
  def NAME # _X4 : SInst<"sv" # name # "nm[_{d}_x4]", "444", "hfd", MergeNone, "aarch64_sve_f" # name # "nm_x4", [IsStreaming], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
// == FMINNM / FMAXNM ==
  defm SVMINNM : SInstMinMaxByVector<"min">;
  defm SVMAXNM : SInstMinMaxByVector<"max">;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  // FRINTA / FRINTM / FRINTN / FRINTP
  def SVRINTA_X2 : SInst<"svrinta[_{d}_x2]", "22", "f", MergeNone, "aarch64_sve_frinta_x2", [IsStreaming], []>;
  def SVRINTA_X4 : SInst<"svrinta[_{d}_x4]", "44", "f", MergeNone, "aarch64_sve_frinta_x4", [IsStreaming], []>;

  def SVRINTM_X2 : SInst<"svrintm[_{d}_x2]", "22", "f", MergeNone, "aarch64_sve_frintm_x2", [IsStreaming], []>;
  def SVRINTM_X4 : SInst<"svrintm[_{d}_x4]", "44", "f", MergeNone, "aarch64_sve_frintm_x4", [IsStreaming], []>;

  def SVRINTN_X2 : SInst<"svrintn[_{d}_x2]", "22", "f", MergeNone, "aarch64_sve_frintn_x2", [IsStreaming], []>;
  def SVRINTN_X4 : SInst<"svrintn[_{d}_x4]", "44", "f", MergeNone, "aarch64_sve_frintn_x4", [IsStreaming], []>;

  def SVRINTP_X2 : SInst<"svrintp[_{d}_x2]", "22", "f", MergeNone, "aarch64_sve_frintp_x2", [IsStreaming], []>;
  def SVRINTP_X4 : SInst<"svrintp[_{d}_x4]", "44", "f", MergeNone, "aarch64_sve_frintp_x4", [IsStreaming], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  def SVSCLAMP_X2 : SInst<"svclamp[_single_{d}_x2]", "22dd", "csil",     MergeNone, "aarch64_sve_sclamp_single_x2", [IsStreaming], []>;
  def SVUCLAMP_X2 : SInst<"svclamp[_single_{d}_x2]", "22dd", "UcUsUiUl", MergeNone, "aarch64_sve_uclamp_single_x2", [IsStreaming], []>;
````
- **L2129 EN**: Declares TableGen def record `NAME # _X2`.
  **L2129 CN**: 声明 TableGen def 记录 `NAME # _X2`。
- **L2130 EN**: Declares TableGen def record `NAME # _X4`.
  **L2130 CN**: 声明 TableGen def 记录 `NAME # _X4`。
- **L2131 EN**: Closes the current lexical scope or compound statement.
  **L2131 CN**: 结束当前词法作用域或复合语句块。
- **L2132 EN**: Blank line separating nearby declarations or logic blocks.
  **L2132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2133 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2133 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2134 EN**: Comment explains nearby logic, constraints, or intent: `FMINNM / FMAXNM`.
  **L2134 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMINNM / FMAXNM`。
- **L2135 EN**: Declares TableGen defm record `SVMINNM`.
  **L2135 CN**: 声明 TableGen defm 记录 `SVMINNM`。
- **L2136 EN**: Declares TableGen defm record `SVMAXNM`.
  **L2136 CN**: 声明 TableGen defm 记录 `SVMAXNM`。
- **L2137 EN**: Closes the current lexical scope or compound statement.
  **L2137 CN**: 结束当前词法作用域或复合语句块。
- **L2138 EN**: Blank line separating nearby declarations or logic blocks.
  **L2138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2139 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2139 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2140 EN**: Comment explains nearby logic, constraints, or intent: `FRINTA / FRINTM / FRINTN / FRINTP`.
  **L2140 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FRINTA / FRINTM / FRINTN / FRINTP`。
- **L2141 EN**: Declares TableGen def record `SVRINTA_X2`.
  **L2141 CN**: 声明 TableGen def 记录 `SVRINTA_X2`。
- **L2142 EN**: Declares TableGen def record `SVRINTA_X4`.
  **L2142 CN**: 声明 TableGen def 记录 `SVRINTA_X4`。
- **L2143 EN**: Blank line separating nearby declarations or logic blocks.
  **L2143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2144 EN**: Declares TableGen def record `SVRINTM_X2`.
  **L2144 CN**: 声明 TableGen def 记录 `SVRINTM_X2`。
- **L2145 EN**: Declares TableGen def record `SVRINTM_X4`.
  **L2145 CN**: 声明 TableGen def 记录 `SVRINTM_X4`。
- **L2146 EN**: Blank line separating nearby declarations or logic blocks.
  **L2146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2147 EN**: Declares TableGen def record `SVRINTN_X2`.
  **L2147 CN**: 声明 TableGen def 记录 `SVRINTN_X2`。
- **L2148 EN**: Declares TableGen def record `SVRINTN_X4`.
  **L2148 CN**: 声明 TableGen def 记录 `SVRINTN_X4`。
- **L2149 EN**: Blank line separating nearby declarations or logic blocks.
  **L2149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2150 EN**: Declares TableGen def record `SVRINTP_X2`.
  **L2150 CN**: 声明 TableGen def 记录 `SVRINTP_X2`。
- **L2151 EN**: Declares TableGen def record `SVRINTP_X4`.
  **L2151 CN**: 声明 TableGen def 记录 `SVRINTP_X4`。
- **L2152 EN**: Closes the current lexical scope or compound statement.
  **L2152 CN**: 结束当前词法作用域或复合语句块。
- **L2153 EN**: Blank line separating nearby declarations or logic blocks.
  **L2153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2154 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2154 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2155 EN**: Declares TableGen def record `SVSCLAMP_X2`.
  **L2155 CN**: 声明 TableGen def 记录 `SVSCLAMP_X2`。
- **L2156 EN**: Declares TableGen def record `SVUCLAMP_X2`.
  **L2156 CN**: 声明 TableGen def 记录 `SVUCLAMP_X2`。

### Lines 2157-2184

````tablegen
  def SVFCLAMP_X2 : SInst<"svclamp[_single_{d}_x2]", "22dd", "hfd",      MergeNone, "aarch64_sve_fclamp_single_x2", [IsStreaming], []>;

  def SVSCLAMP_X4 : SInst<"svclamp[_single_{d}_x4]", "44dd", "csil",     MergeNone, "aarch64_sve_sclamp_single_x4", [IsStreaming], []>;
  def SVUCLAMP_X4 : SInst<"svclamp[_single_{d}_x4]", "44dd", "UcUsUiUl", MergeNone, "aarch64_sve_uclamp_single_x4", [IsStreaming], []>;
  def SVFCLAMP_X4 : SInst<"svclamp[_single_{d}_x4]", "44dd", "hfd",      MergeNone, "aarch64_sve_fclamp_single_x4", [IsStreaming], []>;
}

multiclass BfSingleMultiVector<string name> {
  def NAME # _SINGLE_X2 : SInst<"sv" # name # "[_single_{d}_x2]", "22d", "b", MergeNone, "aarch64_sve_f" # name # "_single_x2", [IsStreaming], []>;
  def NAME # _SINGLE_X4 : SInst<"sv" # name # "[_single_{d}_x4]", "44d", "b", MergeNone, "aarch64_sve_f" # name # "_single_x4", [IsStreaming], []>;

  def NAME # _X2 : SInst<"sv" # name # "[_{d}_x2]", "222", "b", MergeNone, "aarch64_sve_f" # name # "_x2", [IsStreaming], []>;
  def NAME # _X4 : SInst<"sv" # name # "[_{d}_x4]", "444", "b", MergeNone, "aarch64_sve_f" # name # "_x4", [IsStreaming], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,sve-b16b16"in {
  def SVBFCLAMP_X2 : SInst<"svclamp[_single_{d}_x2]",  "22dd",   "b", MergeNone, "aarch64_sve_bfclamp_single_x2",  [IsStreaming], []>;
  def SVBFCLAMP_X4 : SInst<"svclamp[_single_{d}_x4]",  "44dd",   "b", MergeNone, "aarch64_sve_bfclamp_single_x4",  [IsStreaming], []>;

  // bfmin, bfmax (single, multi)
  defm SVBFMIN : BfSingleMultiVector<"min">;
  defm SVBFMAX : BfSingleMultiVector<"max">;

  // bfminnm, bfmaxnm (single, multi)
  defm SVBFMINNM : BfSingleMultiVector<"minnm">;
  defm SVBFMAXNM : BfSingleMultiVector<"maxnm">;
}

````
- **L2157 EN**: Declares TableGen def record `SVFCLAMP_X2`.
  **L2157 CN**: 声明 TableGen def 记录 `SVFCLAMP_X2`。
- **L2158 EN**: Blank line separating nearby declarations or logic blocks.
  **L2158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2159 EN**: Declares TableGen def record `SVSCLAMP_X4`.
  **L2159 CN**: 声明 TableGen def 记录 `SVSCLAMP_X4`。
- **L2160 EN**: Declares TableGen def record `SVUCLAMP_X4`.
  **L2160 CN**: 声明 TableGen def 记录 `SVUCLAMP_X4`。
- **L2161 EN**: Declares TableGen def record `SVFCLAMP_X4`.
  **L2161 CN**: 声明 TableGen def 记录 `SVFCLAMP_X4`。
- **L2162 EN**: Closes the current lexical scope or compound statement.
  **L2162 CN**: 结束当前词法作用域或复合语句块。
- **L2163 EN**: Blank line separating nearby declarations or logic blocks.
  **L2163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2164 EN**: Declares TableGen multiclass record `BfSingleMultiVector`.
  **L2164 CN**: 声明 TableGen multiclass 记录 `BfSingleMultiVector`。
- **L2165 EN**: Declares TableGen def record `NAME # _SINGLE_X2`.
  **L2165 CN**: 声明 TableGen def 记录 `NAME # _SINGLE_X2`。
- **L2166 EN**: Declares TableGen def record `NAME # _SINGLE_X4`.
  **L2166 CN**: 声明 TableGen def 记录 `NAME # _SINGLE_X4`。
- **L2167 EN**: Blank line separating nearby declarations or logic blocks.
  **L2167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2168 EN**: Declares TableGen def record `NAME # _X2`.
  **L2168 CN**: 声明 TableGen def 记录 `NAME # _X2`。
- **L2169 EN**: Declares TableGen def record `NAME # _X4`.
  **L2169 CN**: 声明 TableGen def 记录 `NAME # _X4`。
- **L2170 EN**: Closes the current lexical scope or compound statement.
  **L2170 CN**: 结束当前词法作用域或复合语句块。
- **L2171 EN**: Blank line separating nearby declarations or logic blocks.
  **L2171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2172 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,sve-b16b16"in {`.
  **L2172 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,sve-b16b16"in {`。
- **L2173 EN**: Declares TableGen def record `SVBFCLAMP_X2`.
  **L2173 CN**: 声明 TableGen def 记录 `SVBFCLAMP_X2`。
- **L2174 EN**: Declares TableGen def record `SVBFCLAMP_X4`.
  **L2174 CN**: 声明 TableGen def 记录 `SVBFCLAMP_X4`。
- **L2175 EN**: Blank line separating nearby declarations or logic blocks.
  **L2175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2176 EN**: Comment explains nearby logic, constraints, or intent: `bfmin, bfmax (single, multi)`.
  **L2176 CN**: 注释解释附近代码的逻辑、约束或设计意图：`bfmin, bfmax (single, multi)`。
- **L2177 EN**: Declares TableGen defm record `SVBFMIN`.
  **L2177 CN**: 声明 TableGen defm 记录 `SVBFMIN`。
- **L2178 EN**: Declares TableGen defm record `SVBFMAX`.
  **L2178 CN**: 声明 TableGen defm 记录 `SVBFMAX`。
- **L2179 EN**: Blank line separating nearby declarations or logic blocks.
  **L2179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2180 EN**: Comment explains nearby logic, constraints, or intent: `bfminnm, bfmaxnm (single, multi)`.
  **L2180 CN**: 注释解释附近代码的逻辑、约束或设计意图：`bfminnm, bfmaxnm (single, multi)`。
- **L2181 EN**: Declares TableGen defm record `SVBFMINNM`.
  **L2181 CN**: 声明 TableGen defm 记录 `SVBFMINNM`。
- **L2182 EN**: Declares TableGen defm record `SVBFMAXNM`.
  **L2182 CN**: 声明 TableGen defm 记录 `SVBFMAXNM`。
- **L2183 EN**: Closes the current lexical scope or compound statement.
  **L2183 CN**: 结束当前词法作用域或复合语句块。
- **L2184 EN**: Blank line separating nearby declarations or logic blocks.
  **L2184 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2185-2212

````tablegen
let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,sve-bfscale" in {
  // BFMUL
  defm SVBFMUL : BfSingleMultiVector<"mul">;
  // BFSCALE
  def SVBFSCALE_SINGLE_X2 : SInst<"svscale[_single_{d}_x2]", "22x", "b", MergeNone, "aarch64_sve_fscale_single_x2", [IsStreaming], []>;
  def SVBFSCALE_SINGLE_X4 : SInst<"svscale[_single_{d}_x4]", "44x", "b", MergeNone, "aarch64_sve_fscale_single_x4", [IsStreaming], []>;

  def SVBFSCALE_X2 : SInst<"svscale[_{d}_x2]", "222.x", "b", MergeNone, "aarch64_sve_fscale_x2", [IsStreaming], []>;
  def SVBFSCALE_X4 : SInst<"svscale[_{d}_x4]", "444.x", "b", MergeNone, "aarch64_sve_fscale_x4", [IsStreaming], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  // == ADD (vectors) ==
  def SVADD_SINGLE_X2 : SInst<"svadd[_single_{d}_x2]", "22d", "cUcsUsiUilUl", MergeNone, "aarch64_sve_add_single_x2", [IsStreaming], []>;
  def SVADD_SINGLE_X4 : SInst<"svadd[_single_{d}_x4]", "44d", "cUcsUsiUilUl", MergeNone, "aarch64_sve_add_single_x4", [IsStreaming], []>;

  // 2-way and 4-way selects
  def SVSEL_X2 : SInst<"svsel[_{d}_x2]", "2}22", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_sel_x2", [IsStreaming], []>;
  def SVSEL_X4 : SInst<"svsel[_{d}_x4]", "4}44", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_sel_x4", [IsStreaming], []>;

  // SRSHL / URSHL
  def SVSRSHL_SINGLE_X2 : SInst<"svrshl[_single_{d}_x2]", "22d", "csil",     MergeNone, "aarch64_sve_srshl_single_x2", [IsStreaming], []>;
  def SVURSHL_SINGLE_X2 : SInst<"svrshl[_single_{d}_x2]", "22d", "UcUsUiUl", MergeNone, "aarch64_sve_urshl_single_x2", [IsStreaming], []>;
  def SVSRSHL_SINGLE_X4 : SInst<"svrshl[_single_{d}_x4]", "44d", "csil",     MergeNone, "aarch64_sve_srshl_single_x4", [IsStreaming], []>;
  def SVURSHL_SINGLE_X4 : SInst<"svrshl[_single_{d}_x4]", "44d", "UcUsUiUl", MergeNone, "aarch64_sve_urshl_single_x4", [IsStreaming], []>;

  def SVSRSHL_X2 : SInst<"svrshl[_{d}_x2]", "222", "csil",     MergeNone, "aarch64_sve_srshl_x2", [IsStreaming], []>;
  def SVURSHL_X2 : SInst<"svrshl[_{d}_x2]", "222", "UcUsUiUl", MergeNone, "aarch64_sve_urshl_x2", [IsStreaming], []>;
````
- **L2185 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,sve-bfscale" in {`.
  **L2185 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,sve-bfscale" in {`。
- **L2186 EN**: Comment explains nearby logic, constraints, or intent: `BFMUL`.
  **L2186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`BFMUL`。
- **L2187 EN**: Declares TableGen defm record `SVBFMUL`.
  **L2187 CN**: 声明 TableGen defm 记录 `SVBFMUL`。
- **L2188 EN**: Comment explains nearby logic, constraints, or intent: `BFSCALE`.
  **L2188 CN**: 注释解释附近代码的逻辑、约束或设计意图：`BFSCALE`。
- **L2189 EN**: Declares TableGen def record `SVBFSCALE_SINGLE_X2`.
  **L2189 CN**: 声明 TableGen def 记录 `SVBFSCALE_SINGLE_X2`。
- **L2190 EN**: Declares TableGen def record `SVBFSCALE_SINGLE_X4`.
  **L2190 CN**: 声明 TableGen def 记录 `SVBFSCALE_SINGLE_X4`。
- **L2191 EN**: Blank line separating nearby declarations or logic blocks.
  **L2191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2192 EN**: Declares TableGen def record `SVBFSCALE_X2`.
  **L2192 CN**: 声明 TableGen def 记录 `SVBFSCALE_X2`。
- **L2193 EN**: Declares TableGen def record `SVBFSCALE_X4`.
  **L2193 CN**: 声明 TableGen def 记录 `SVBFSCALE_X4`。
- **L2194 EN**: Closes the current lexical scope or compound statement.
  **L2194 CN**: 结束当前词法作用域或复合语句块。
- **L2195 EN**: Blank line separating nearby declarations or logic blocks.
  **L2195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2196 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2196 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2197 EN**: Comment explains nearby logic, constraints, or intent: `ADD (vectors)`.
  **L2197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ADD (vectors)`。
- **L2198 EN**: Declares TableGen def record `SVADD_SINGLE_X2`.
  **L2198 CN**: 声明 TableGen def 记录 `SVADD_SINGLE_X2`。
- **L2199 EN**: Declares TableGen def record `SVADD_SINGLE_X4`.
  **L2199 CN**: 声明 TableGen def 记录 `SVADD_SINGLE_X4`。
- **L2200 EN**: Blank line separating nearby declarations or logic blocks.
  **L2200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2201 EN**: Comment explains nearby logic, constraints, or intent: `2-way and 4-way selects`.
  **L2201 CN**: 注释解释附近代码的逻辑、约束或设计意图：`2-way and 4-way selects`。
- **L2202 EN**: Declares TableGen def record `SVSEL_X2`.
  **L2202 CN**: 声明 TableGen def 记录 `SVSEL_X2`。
- **L2203 EN**: Declares TableGen def record `SVSEL_X4`.
  **L2203 CN**: 声明 TableGen def 记录 `SVSEL_X4`。
- **L2204 EN**: Blank line separating nearby declarations or logic blocks.
  **L2204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2205 EN**: Comment explains nearby logic, constraints, or intent: `SRSHL / URSHL`.
  **L2205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SRSHL / URSHL`。
- **L2206 EN**: Declares TableGen def record `SVSRSHL_SINGLE_X2`.
  **L2206 CN**: 声明 TableGen def 记录 `SVSRSHL_SINGLE_X2`。
- **L2207 EN**: Declares TableGen def record `SVURSHL_SINGLE_X2`.
  **L2207 CN**: 声明 TableGen def 记录 `SVURSHL_SINGLE_X2`。
- **L2208 EN**: Declares TableGen def record `SVSRSHL_SINGLE_X4`.
  **L2208 CN**: 声明 TableGen def 记录 `SVSRSHL_SINGLE_X4`。
- **L2209 EN**: Declares TableGen def record `SVURSHL_SINGLE_X4`.
  **L2209 CN**: 声明 TableGen def 记录 `SVURSHL_SINGLE_X4`。
- **L2210 EN**: Blank line separating nearby declarations or logic blocks.
  **L2210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2211 EN**: Declares TableGen def record `SVSRSHL_X2`.
  **L2211 CN**: 声明 TableGen def 记录 `SVSRSHL_X2`。
- **L2212 EN**: Declares TableGen def record `SVURSHL_X2`.
  **L2212 CN**: 声明 TableGen def 记录 `SVURSHL_X2`。

### Lines 2213-2240

````tablegen
  def SVSRSHL_X4 : SInst<"svrshl[_{d}_x4]", "444", "csil",     MergeNone, "aarch64_sve_srshl_x4", [IsStreaming], []>;
  def SVURSHL_X4 : SInst<"svrshl[_{d}_x4]", "444", "UcUsUiUl", MergeNone, "aarch64_sve_urshl_x4", [IsStreaming], []>;

  def SVQRSHRN_X4  : SInst<"svqrshrn[_n]_{0}[_{d}_x4]", "q4i", "il",   MergeNone, "aarch64_sve_sqrshrn_x4", [IsStreaming], [ImmCheck<1, ImmCheckShiftRight, 0>]>;
  def SVUQRSHRN_X4 : SInst<"svqrshrn[_n]_{0}[_{d}_x4]", "b4i", "UiUl", MergeNone, "aarch64_sve_uqrshrn_x4", [IsStreaming], [ImmCheck<1, ImmCheckShiftRight, 0>]>;

  // SQRSHR / UQRSHR
  def SVQRSHR_X2  : SInst<"svqrshr[_n]_{0}[_{d}_x2]", "h2i", "i",    MergeNone, "aarch64_sve_sqrshr_x2", [IsStreaming], [ImmCheck<1, ImmCheck1_16>]>;
  def SVUQRSHR_X2 : SInst<"svqrshr[_n]_{0}[_{d}_x2]", "e2i", "Ui",   MergeNone, "aarch64_sve_uqrshr_x2", [IsStreaming], [ImmCheck<1, ImmCheck1_16>]>;
  def SVQRSHR_X4  : SInst<"svqrshr[_n]_{0}[_{d}_x4]", "q4i", "il",   MergeNone, "aarch64_sve_sqrshr_x4", [IsStreaming], [ImmCheck<1, ImmCheckShiftRight, 0>]>;
  def SVUQRSHR_X4 : SInst<"svqrshr[_n]_{0}[_{d}_x4]", "b4i", "UiUl", MergeNone, "aarch64_sve_uqrshr_x4", [IsStreaming], [ImmCheck<1, ImmCheckShiftRight, 0>]>;

  // SQRSHRU
  def SVSQRSHRU_X2 : SInst<"svqrshru[_n]_{0}[_{d}_x2]", "e2i", "i",  MergeNone, "aarch64_sve_sqrshru_x2", [IsStreaming], [ImmCheck<1, ImmCheck1_16>]>;
  def SVSQRSHRU_X4 : SInst<"svqrshru[_n]_{0}[_{d}_x4]", "b4i", "il", MergeNone, "aarch64_sve_sqrshru_x4", [IsStreaming], [ImmCheck<1, ImmCheckShiftRight, 0>]>;

  def SVSQRSHRUN_X4 : SInst<"svqrshrun[_n]_{0}[_{d}_x4]", "b4i", "il", MergeNone, "aarch64_sve_sqrshrun_x4", [IsStreaming], [ImmCheck<1, ImmCheckShiftRight, 0>]>;

  // SQDMULH
  def SVSQDMULH_SINGLE_X2 : SInst<"svqdmulh[_single_{d}_x2]", "22d", "csil", MergeNone, "aarch64_sve_sqdmulh_single_vgx2", [IsStreaming], []>;
  def SVSQDMULH_SINGLE_X4 : SInst<"svqdmulh[_single_{d}_x4]", "44d", "csil", MergeNone, "aarch64_sve_sqdmulh_single_vgx4", [IsStreaming], []>;
  def SVSQDMULH_X2        : SInst<"svqdmulh[_{d}_x2]",        "222", "csil", MergeNone, "aarch64_sve_sqdmulh_vgx2",        [IsStreaming], []>;
  def SVSQDMULH_X4        : SInst<"svqdmulh[_{d}_x4]",        "444", "csil", MergeNone, "aarch64_sve_sqdmulh_vgx4",        [IsStreaming], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,faminmax" in {
  def FAMIN_X2 : Inst<"svamin[_{d}_x2]", "222", "hfd", MergeNone, "aarch64_sme_famin_x2",  [IsStreaming], []>;
  def FAMAX_X2 : Inst<"svamax[_{d}_x2]", "222", "hfd", MergeNone, "aarch64_sme_famax_x2",  [IsStreaming], []>;
````
- **L2213 EN**: Declares TableGen def record `SVSRSHL_X4`.
  **L2213 CN**: 声明 TableGen def 记录 `SVSRSHL_X4`。
- **L2214 EN**: Declares TableGen def record `SVURSHL_X4`.
  **L2214 CN**: 声明 TableGen def 记录 `SVURSHL_X4`。
- **L2215 EN**: Blank line separating nearby declarations or logic blocks.
  **L2215 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2216 EN**: Declares TableGen def record `SVQRSHRN_X4`.
  **L2216 CN**: 声明 TableGen def 记录 `SVQRSHRN_X4`。
- **L2217 EN**: Declares TableGen def record `SVUQRSHRN_X4`.
  **L2217 CN**: 声明 TableGen def 记录 `SVUQRSHRN_X4`。
- **L2218 EN**: Blank line separating nearby declarations or logic blocks.
  **L2218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2219 EN**: Comment explains nearby logic, constraints, or intent: `SQRSHR / UQRSHR`.
  **L2219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SQRSHR / UQRSHR`。
- **L2220 EN**: Declares TableGen def record `SVQRSHR_X2`.
  **L2220 CN**: 声明 TableGen def 记录 `SVQRSHR_X2`。
- **L2221 EN**: Declares TableGen def record `SVUQRSHR_X2`.
  **L2221 CN**: 声明 TableGen def 记录 `SVUQRSHR_X2`。
- **L2222 EN**: Declares TableGen def record `SVQRSHR_X4`.
  **L2222 CN**: 声明 TableGen def 记录 `SVQRSHR_X4`。
- **L2223 EN**: Declares TableGen def record `SVUQRSHR_X4`.
  **L2223 CN**: 声明 TableGen def 记录 `SVUQRSHR_X4`。
- **L2224 EN**: Blank line separating nearby declarations or logic blocks.
  **L2224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2225 EN**: Comment explains nearby logic, constraints, or intent: `SQRSHRU`.
  **L2225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SQRSHRU`。
- **L2226 EN**: Declares TableGen def record `SVSQRSHRU_X2`.
  **L2226 CN**: 声明 TableGen def 记录 `SVSQRSHRU_X2`。
- **L2227 EN**: Declares TableGen def record `SVSQRSHRU_X4`.
  **L2227 CN**: 声明 TableGen def 记录 `SVSQRSHRU_X4`。
- **L2228 EN**: Blank line separating nearby declarations or logic blocks.
  **L2228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2229 EN**: Declares TableGen def record `SVSQRSHRUN_X4`.
  **L2229 CN**: 声明 TableGen def 记录 `SVSQRSHRUN_X4`。
- **L2230 EN**: Blank line separating nearby declarations or logic blocks.
  **L2230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2231 EN**: Comment explains nearby logic, constraints, or intent: `SQDMULH`.
  **L2231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SQDMULH`。
- **L2232 EN**: Declares TableGen def record `SVSQDMULH_SINGLE_X2`.
  **L2232 CN**: 声明 TableGen def 记录 `SVSQDMULH_SINGLE_X2`。
- **L2233 EN**: Declares TableGen def record `SVSQDMULH_SINGLE_X4`.
  **L2233 CN**: 声明 TableGen def 记录 `SVSQDMULH_SINGLE_X4`。
- **L2234 EN**: Declares TableGen def record `SVSQDMULH_X2`.
  **L2234 CN**: 声明 TableGen def 记录 `SVSQDMULH_X2`。
- **L2235 EN**: Declares TableGen def record `SVSQDMULH_X4`.
  **L2235 CN**: 声明 TableGen def 记录 `SVSQDMULH_X4`。
- **L2236 EN**: Closes the current lexical scope or compound statement.
  **L2236 CN**: 结束当前词法作用域或复合语句块。
- **L2237 EN**: Blank line separating nearby declarations or logic blocks.
  **L2237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2238 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,faminmax" in {`.
  **L2238 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,faminmax" in {`。
- **L2239 EN**: Declares TableGen def record `FAMIN_X2`.
  **L2239 CN**: 声明 TableGen def 记录 `FAMIN_X2`。
- **L2240 EN**: Declares TableGen def record `FAMAX_X2`.
  **L2240 CN**: 声明 TableGen def 记录 `FAMAX_X2`。

### Lines 2241-2268

````tablegen
  def FAMIN_X4 : Inst<"svamin[_{d}_x4]", "444", "hfd", MergeNone, "aarch64_sme_famin_x4",  [IsStreaming], []>;
  def FAMAX_X4 : Inst<"svamax[_{d}_x4]", "444", "hfd", MergeNone, "aarch64_sme_famax_x4",  [IsStreaming], []>;
}

let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {
  def REINTERPRET_SVBOOL_TO_SVCOUNT : Inst<"svreinterpret[_c]", "}P", "Pc", MergeNone, "", [VerifyRuntimeMode], []>;
  def REINTERPRET_SVCOUNT_TO_SVBOOL : Inst<"svreinterpret[_b]", "P}", "Pc", MergeNone, "", [VerifyRuntimeMode], []>;
}

let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {
  def SVQRSHRN_X2   : SInst<"svqrshrn[_n]_{0}[_{d}_x2]", "h2i", "i",  MergeNone, "aarch64_sve_sqrshrn_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;
  def SVUQRSHRN_X2  : SInst<"svqrshrn[_n]_{0}[_{d}_x2]", "e2i", "Ui", MergeNone, "aarch64_sve_uqrshrn_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;
  def SVSQRSHRUN_X2 : SInst<"svqrshrun[_n]_{0}[_{d}_x2]", "e2i", "i", MergeNone, "aarch64_sve_sqrshrun_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck1_16>]>;
}

//
// Multi-vector saturating rounding shift right narrow and interleave
//
let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {
  def SVSQRSHRN_X2_S8  : SInst<"svqrshrn[_n]_{0}[_{d}_x2]", "h2i", "s",  MergeNone, "aarch64_sve_sqrshrn_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
  def SVUQRSHRN_X2_U8  : SInst<"svqrshrn[_n]_{0}[_{d}_x2]", "e2i", "Us", MergeNone, "aarch64_sve_uqrshrn_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
  def SVSQRSHRUN_X2_S8 : SInst<"svqrshrun[_n]_{0}[_{d}_x2]", "e2i", "s", MergeNone, "aarch64_sve_sqrshrun_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
}

let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {
  def SVZIPQ1 : SInst<"svzipq1[_{d}]", "ddd", "cUcsUsiUilUlbhfdm", MergeNone, "aarch64_sve_zipq1", [VerifyRuntimeMode], []>;
  def SVZIPQ2 : SInst<"svzipq2[_{d}]", "ddd", "cUcsUsiUilUlbhfdm", MergeNone, "aarch64_sve_zipq2", [VerifyRuntimeMode], []>;
  def SVUZPQ1 : SInst<"svuzpq1[_{d}]", "ddd", "cUcsUsiUilUlbhfdm", MergeNone, "aarch64_sve_uzpq1", [VerifyRuntimeMode], []>;
````
- **L2241 EN**: Declares TableGen def record `FAMIN_X4`.
  **L2241 CN**: 声明 TableGen def 记录 `FAMIN_X4`。
- **L2242 EN**: Declares TableGen def record `FAMAX_X4`.
  **L2242 CN**: 声明 TableGen def 记录 `FAMAX_X4`。
- **L2243 EN**: Closes the current lexical scope or compound statement.
  **L2243 CN**: 结束当前词法作用域或复合语句块。
- **L2244 EN**: Blank line separating nearby declarations or logic blocks.
  **L2244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2245 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2245 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2246 EN**: Declares TableGen def record `REINTERPRET_SVBOOL_TO_SVCOUNT`.
  **L2246 CN**: 声明 TableGen def 记录 `REINTERPRET_SVBOOL_TO_SVCOUNT`。
- **L2247 EN**: Declares TableGen def record `REINTERPRET_SVCOUNT_TO_SVBOOL`.
  **L2247 CN**: 声明 TableGen def 记录 `REINTERPRET_SVCOUNT_TO_SVBOOL`。
- **L2248 EN**: Closes the current lexical scope or compound statement.
  **L2248 CN**: 结束当前词法作用域或复合语句块。
- **L2249 EN**: Blank line separating nearby declarations or logic blocks.
  **L2249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2250 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2250 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2251 EN**: Declares TableGen def record `SVQRSHRN_X2`.
  **L2251 CN**: 声明 TableGen def 记录 `SVQRSHRN_X2`。
- **L2252 EN**: Declares TableGen def record `SVUQRSHRN_X2`.
  **L2252 CN**: 声明 TableGen def 记录 `SVUQRSHRN_X2`。
- **L2253 EN**: Declares TableGen def record `SVSQRSHRUN_X2`.
  **L2253 CN**: 声明 TableGen def 记录 `SVSQRSHRUN_X2`。
- **L2254 EN**: Closes the current lexical scope or compound statement.
  **L2254 CN**: 结束当前词法作用域或复合语句块。
- **L2255 EN**: Blank line separating nearby declarations or logic blocks.
  **L2255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2256 EN**: Separator comment used for visual grouping.
  **L2256 CN**: 用于视觉分组的分隔注释。
- **L2257 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector saturating rounding shift right narrow and interleave`.
  **L2257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector saturating rounding shift right narrow and interleave`。
- **L2258 EN**: Separator comment used for visual grouping.
  **L2258 CN**: 用于视觉分组的分隔注释。
- **L2259 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`.
  **L2259 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`。
- **L2260 EN**: Declares TableGen def record `SVSQRSHRN_X2_S8`.
  **L2260 CN**: 声明 TableGen def 记录 `SVSQRSHRN_X2_S8`。
- **L2261 EN**: Declares TableGen def record `SVUQRSHRN_X2_U8`.
  **L2261 CN**: 声明 TableGen def 记录 `SVUQRSHRN_X2_U8`。
- **L2262 EN**: Declares TableGen def record `SVSQRSHRUN_X2_S8`.
  **L2262 CN**: 声明 TableGen def 记录 `SVSQRSHRUN_X2_S8`。
- **L2263 EN**: Closes the current lexical scope or compound statement.
  **L2263 CN**: 结束当前词法作用域或复合语句块。
- **L2264 EN**: Blank line separating nearby declarations or logic blocks.
  **L2264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2265 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`.
  **L2265 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`。
- **L2266 EN**: Declares TableGen def record `SVZIPQ1`.
  **L2266 CN**: 声明 TableGen def 记录 `SVZIPQ1`。
- **L2267 EN**: Declares TableGen def record `SVZIPQ2`.
  **L2267 CN**: 声明 TableGen def 记录 `SVZIPQ2`。
- **L2268 EN**: Declares TableGen def record `SVUZPQ1`.
  **L2268 CN**: 声明 TableGen def 记录 `SVUZPQ1`。

### Lines 2269-2296

````tablegen
  def SVUZPQ2 : SInst<"svuzpq2[_{d}]", "ddd", "cUcsUsiUilUlbhfdm", MergeNone, "aarch64_sve_uzpq2", [VerifyRuntimeMode], []>;

  def SVTBLQ : SInst<"svtblq[_{d}]", "ddu",  "cUcsUsiUilUlbhfdm", MergeNone, "aarch64_sve_tblq", [VerifyRuntimeMode]>;
  def SVTBXQ : SInst<"svtbxq[_{d}]", "dddu", "cUcsUsiUilUlbhfdm", MergeNone, "aarch64_sve_tbxq", [VerifyRuntimeMode]>;

  def EXTQ : SInst<"svextq[_{d}]", "dddk", "cUcsUsiUilUlbhfdm", MergeNone, "aarch64_sve_extq", [VerifyRuntimeMode], [ImmCheck<2, ImmCheckLaneIndex, 0>]>;


  // Move to Pred
  multiclass PMOV_TO_PRED<string name, string types, string intrinsic, list<FlagType> flags=[], ImmCheckType immCh > {
    def _LANE      : Inst<name # "_lane[_{d}]", "Pdi", types, MergeNone, intrinsic, flags, [ImmCheck<1, immCh>]>;
    def _LANE_ZERO : SInst<name # "[_{d}]", "Pd", types, MergeNone, intrinsic # "_zero", flags, []>;
  }

  defm SVPMOV_B_TO_PRED : PMOV_TO_PRED<"svpmov", "cUc", "aarch64_sve_pmov_to_pred_lane", [VerifyRuntimeMode], ImmCheck0_0>;
  defm SVPMOV_H_TO_PRED : PMOV_TO_PRED<"svpmov", "sUs", "aarch64_sve_pmov_to_pred_lane", [VerifyRuntimeMode], ImmCheck0_1>;
  defm SVPMOV_S_TO_PRED : PMOV_TO_PRED<"svpmov", "iUi", "aarch64_sve_pmov_to_pred_lane", [VerifyRuntimeMode], ImmCheck0_3>;
  defm SVPMOV_D_TO_PRED : PMOV_TO_PRED<"svpmov", "lUl", "aarch64_sve_pmov_to_pred_lane", [VerifyRuntimeMode], ImmCheck0_7>;

  // Move to Vector
  multiclass PMOV_TO_VEC<string name, string types, string intrinsic, list<FlagType> flags=[], ImmCheckType immCh > {
    def _M : SInst<name # "_lane[_{d}]", "ddPi", types, MergeOp1, intrinsic # "_merging", flags, [ImmCheck<2, immCh>]>;
    def _Z : SInst<name # "_{d}_z", "dP",  types, MergeNone, intrinsic # "_zeroing", flags, []>;
  }

  def SVPMOV_TO_VEC_LANE_B : SInst<"svpmov_{d}_z", "dP",  "cUc", MergeNone, "aarch64_sve_pmov_to_vector_lane_zeroing", [VerifyRuntimeMode], []>;
  defm SVPMOV_TO_VEC_LANE_H : PMOV_TO_VEC<"svpmov", "sUs", "aarch64_sve_pmov_to_vector_lane", [VerifyRuntimeMode], ImmCheck1_1>;
  defm SVPMOV_TO_VEC_LANE_S : PMOV_TO_VEC<"svpmov", "iUi", "aarch64_sve_pmov_to_vector_lane", [VerifyRuntimeMode], ImmCheck1_3>;
````
- **L2269 EN**: Declares TableGen def record `SVUZPQ2`.
  **L2269 CN**: 声明 TableGen def 记录 `SVUZPQ2`。
- **L2270 EN**: Blank line separating nearby declarations or logic blocks.
  **L2270 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2271 EN**: Declares TableGen def record `SVTBLQ`.
  **L2271 CN**: 声明 TableGen def 记录 `SVTBLQ`。
- **L2272 EN**: Declares TableGen def record `SVTBXQ`.
  **L2272 CN**: 声明 TableGen def 记录 `SVTBXQ`。
- **L2273 EN**: Blank line separating nearby declarations or logic blocks.
  **L2273 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2274 EN**: Declares TableGen def record `EXTQ`.
  **L2274 CN**: 声明 TableGen def 记录 `EXTQ`。
- **L2275 EN**: Blank line separating nearby declarations or logic blocks.
  **L2275 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2276 EN**: Blank line separating nearby declarations or logic blocks.
  **L2276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2277 EN**: Comment explains nearby logic, constraints, or intent: `Move to Pred`.
  **L2277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Move to Pred`。
- **L2278 EN**: Declares TableGen multiclass record `PMOV_TO_PRED`.
  **L2278 CN**: 声明 TableGen multiclass 记录 `PMOV_TO_PRED`。
- **L2279 EN**: Declares TableGen def record `_LANE`.
  **L2279 CN**: 声明 TableGen def 记录 `_LANE`。
- **L2280 EN**: Declares TableGen def record `_LANE_ZERO`.
  **L2280 CN**: 声明 TableGen def 记录 `_LANE_ZERO`。
- **L2281 EN**: Closes the current lexical scope or compound statement.
  **L2281 CN**: 结束当前词法作用域或复合语句块。
- **L2282 EN**: Blank line separating nearby declarations or logic blocks.
  **L2282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2283 EN**: Declares TableGen defm record `SVPMOV_B_TO_PRED`.
  **L2283 CN**: 声明 TableGen defm 记录 `SVPMOV_B_TO_PRED`。
- **L2284 EN**: Declares TableGen defm record `SVPMOV_H_TO_PRED`.
  **L2284 CN**: 声明 TableGen defm 记录 `SVPMOV_H_TO_PRED`。
- **L2285 EN**: Declares TableGen defm record `SVPMOV_S_TO_PRED`.
  **L2285 CN**: 声明 TableGen defm 记录 `SVPMOV_S_TO_PRED`。
- **L2286 EN**: Declares TableGen defm record `SVPMOV_D_TO_PRED`.
  **L2286 CN**: 声明 TableGen defm 记录 `SVPMOV_D_TO_PRED`。
- **L2287 EN**: Blank line separating nearby declarations or logic blocks.
  **L2287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2288 EN**: Comment explains nearby logic, constraints, or intent: `Move to Vector`.
  **L2288 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Move to Vector`。
- **L2289 EN**: Declares TableGen multiclass record `PMOV_TO_VEC`.
  **L2289 CN**: 声明 TableGen multiclass 记录 `PMOV_TO_VEC`。
- **L2290 EN**: Declares TableGen def record `_M`.
  **L2290 CN**: 声明 TableGen def 记录 `_M`。
- **L2291 EN**: Declares TableGen def record `_Z`.
  **L2291 CN**: 声明 TableGen def 记录 `_Z`。
- **L2292 EN**: Closes the current lexical scope or compound statement.
  **L2292 CN**: 结束当前词法作用域或复合语句块。
- **L2293 EN**: Blank line separating nearby declarations or logic blocks.
  **L2293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2294 EN**: Declares TableGen def record `SVPMOV_TO_VEC_LANE_B`.
  **L2294 CN**: 声明 TableGen def 记录 `SVPMOV_TO_VEC_LANE_B`。
- **L2295 EN**: Declares TableGen defm record `SVPMOV_TO_VEC_LANE_H`.
  **L2295 CN**: 声明 TableGen defm 记录 `SVPMOV_TO_VEC_LANE_H`。
- **L2296 EN**: Declares TableGen defm record `SVPMOV_TO_VEC_LANE_S`.
  **L2296 CN**: 声明 TableGen defm 记录 `SVPMOV_TO_VEC_LANE_S`。

### Lines 2297-2324

````tablegen
  defm SVPMOV_TO_VEC_LANE_D : PMOV_TO_VEC<"svpmov", "lUl", "aarch64_sve_pmov_to_vector_lane" ,[VerifyRuntimeMode], ImmCheck1_7>;
}

let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {
  def SVDUP_LANEQ_B : SInst<"svdup_laneq[_{d}]", "ddi", "cUcm",  MergeNone, "aarch64_sve_dup_laneq", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_15>]>;
  def SVDUP_LANEQ_H : SInst<"svdup_laneq[_{d}]", "ddi", "sUshb", MergeNone, "aarch64_sve_dup_laneq", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_7>]>;
  def SVDUP_LANEQ_S : SInst<"svdup_laneq[_{d}]", "ddi", "iUif",  MergeNone, "aarch64_sve_dup_laneq", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_3>]>;
  def SVDUP_LANEQ_D : SInst<"svdup_laneq[_{d}]", "ddi", "lUld",  MergeNone, "aarch64_sve_dup_laneq", [VerifyRuntimeMode], [ImmCheck<1, ImmCheck0_1>]>;
}

//
// Multi-vector convert to/from floating-point.
//
let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  def SVCVT_F16_X2  : SInst<"svcvt_f16[_f32_x2]",  "h2", "f", MergeNone, "aarch64_sve_fcvt_x2", [IsStreaming],[]>;
  def SVCVT_BF16_X2 : SInst<"svcvt_bf16[_f32_x2]", "$2", "f", MergeNone, "aarch64_sve_bfcvt_x2", [IsOverloadNone, IsStreaming],[]>;

  def SVCVT_F32_U32_X2 : SInst<"svcvt_{d}[_u32_x2]", "2.d2.u", "f",  MergeNone, "aarch64_sve_ucvtf_x2",  [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;
  def SVCVT_U32_F32_X2 : SInst<"svcvt_{d}[_f32_x2]", "2.d2.M", "Ui", MergeNone, "aarch64_sve_fcvtzu_x2", [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;
  def SVCVT_F32_S32_X2 : SInst<"svcvt_{d}[_s32_x2]", "2.d2.x", "f",  MergeNone, "aarch64_sve_scvtf_x2",  [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;
  def SVCVT_S32_F32_X2 : SInst<"svcvt_{d}[_f32_x2]", "2.d2.M", "i",  MergeNone, "aarch64_sve_fcvtzs_x2", [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;

  def SVCVT_F32_U32_X4 : SInst<"svcvt_{d}[_u32_x4]", "4.d4.u", "f",  MergeNone, "aarch64_sve_ucvtf_x4",  [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;
  def SVCVT_U32_F32_X4 : SInst<"svcvt_{d}[_f32_x4]", "4.d4.M", "Ui", MergeNone, "aarch64_sve_fcvtzu_x4", [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;
  def SVCVT_F32_S32_X4 : SInst<"svcvt_{d}[_s32_x4]", "4.d4.x", "f",  MergeNone, "aarch64_sve_scvtf_x4",  [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;
  def SVCVT_S32_F32_X4 : SInst<"svcvt_{d}[_f32_x4]", "4.d4.M", "i",  MergeNone, "aarch64_sve_fcvtzs_x4", [IsStreaming, IsOverloadWhileOrMultiVecCvt], []>;
}

````
- **L2297 EN**: Declares TableGen defm record `SVPMOV_TO_VEC_LANE_D`.
  **L2297 CN**: 声明 TableGen defm 记录 `SVPMOV_TO_VEC_LANE_D`。
- **L2298 EN**: Closes the current lexical scope or compound statement.
  **L2298 CN**: 结束当前词法作用域或复合语句块。
- **L2299 EN**: Blank line separating nearby declarations or logic blocks.
  **L2299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2300 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`.
  **L2300 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2p1", SMETargetGuard = "sve2p1|sme2p1" in {`。
- **L2301 EN**: Declares TableGen def record `SVDUP_LANEQ_B`.
  **L2301 CN**: 声明 TableGen def 记录 `SVDUP_LANEQ_B`。
- **L2302 EN**: Declares TableGen def record `SVDUP_LANEQ_H`.
  **L2302 CN**: 声明 TableGen def 记录 `SVDUP_LANEQ_H`。
- **L2303 EN**: Declares TableGen def record `SVDUP_LANEQ_S`.
  **L2303 CN**: 声明 TableGen def 记录 `SVDUP_LANEQ_S`。
- **L2304 EN**: Declares TableGen def record `SVDUP_LANEQ_D`.
  **L2304 CN**: 声明 TableGen def 记录 `SVDUP_LANEQ_D`。
- **L2305 EN**: Closes the current lexical scope or compound statement.
  **L2305 CN**: 结束当前词法作用域或复合语句块。
- **L2306 EN**: Blank line separating nearby declarations or logic blocks.
  **L2306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2307 EN**: Separator comment used for visual grouping.
  **L2307 CN**: 用于视觉分组的分隔注释。
- **L2308 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector convert to/from floating-point.`.
  **L2308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector convert to/from floating-point.`。
- **L2309 EN**: Separator comment used for visual grouping.
  **L2309 CN**: 用于视觉分组的分隔注释。
- **L2310 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2310 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2311 EN**: Declares TableGen def record `SVCVT_F16_X2`.
  **L2311 CN**: 声明 TableGen def 记录 `SVCVT_F16_X2`。
- **L2312 EN**: Declares TableGen def record `SVCVT_BF16_X2`.
  **L2312 CN**: 声明 TableGen def 记录 `SVCVT_BF16_X2`。
- **L2313 EN**: Blank line separating nearby declarations or logic blocks.
  **L2313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2314 EN**: Declares TableGen def record `SVCVT_F32_U32_X2`.
  **L2314 CN**: 声明 TableGen def 记录 `SVCVT_F32_U32_X2`。
- **L2315 EN**: Declares TableGen def record `SVCVT_U32_F32_X2`.
  **L2315 CN**: 声明 TableGen def 记录 `SVCVT_U32_F32_X2`。
- **L2316 EN**: Declares TableGen def record `SVCVT_F32_S32_X2`.
  **L2316 CN**: 声明 TableGen def 记录 `SVCVT_F32_S32_X2`。
- **L2317 EN**: Declares TableGen def record `SVCVT_S32_F32_X2`.
  **L2317 CN**: 声明 TableGen def 记录 `SVCVT_S32_F32_X2`。
- **L2318 EN**: Blank line separating nearby declarations or logic blocks.
  **L2318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2319 EN**: Declares TableGen def record `SVCVT_F32_U32_X4`.
  **L2319 CN**: 声明 TableGen def 记录 `SVCVT_F32_U32_X4`。
- **L2320 EN**: Declares TableGen def record `SVCVT_U32_F32_X4`.
  **L2320 CN**: 声明 TableGen def 记录 `SVCVT_U32_F32_X4`。
- **L2321 EN**: Declares TableGen def record `SVCVT_F32_S32_X4`.
  **L2321 CN**: 声明 TableGen def 记录 `SVCVT_F32_S32_X4`。
- **L2322 EN**: Declares TableGen def record `SVCVT_S32_F32_X4`.
  **L2322 CN**: 声明 TableGen def 记录 `SVCVT_S32_F32_X4`。
- **L2323 EN**: Closes the current lexical scope or compound statement.
  **L2323 CN**: 结束当前词法作用域或复合语句块。
- **L2324 EN**: Blank line separating nearby declarations or logic blocks.
  **L2324 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2325-2352

````tablegen
let SVETargetGuard = InvalidMode, SMETargetGuard = "sme-f16f16" in {
  def SVCVT_F32_X2 : SInst<"svcvt_{d}[_f16_x2]", "2h", "f", MergeNone, "aarch64_sve_fcvt_widen_x2", [ IsStreaming],[]>;
}

//
// Multi-vector floating-point convert from single-precision to interleaved half-precision/BFloat16
//
let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  def SVCVTN_F16_X2  : SInst<"svcvtn_f16[_f32_x2]",  "h2", "f", MergeNone, "aarch64_sve_fcvtn_x2", [IsStreaming],[]>;
  def SVCVTN_BF16_X2 : SInst<"svcvtn_bf16[_f32_x2]", "$2", "f", MergeNone, "aarch64_sve_bfcvtn_x2", [IsOverloadNone, IsStreaming],[]>;
}

//
//Multi-vector floating-point convert from half-precision to deinterleaved single-precision.
//
let SVETargetGuard = InvalidMode, SMETargetGuard = "sme-f16f16" in {
  def SVCVTL_F32_X2 : SInst<"svcvtl_f32[_f16_x2]", "2h", "f", MergeNone, "aarch64_sve_fcvtl_widen_x2", [ IsStreaming],[]>;
}

//
// Multi-vector saturating shift right narrow and interleave
//
let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {
  def SVSQSHRN_X2 : SInst<"svqshrn[_n]_{0}[_{d}_x2]", "h2i", "is",  MergeNone, "aarch64_sve_sqshrn_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
  def SVUQSHRN_X2 : SInst<"svqshrn[_n]_{0}[_{d}_x2]", "e2i", "UiUs",  MergeNone, "aarch64_sve_uqshrn_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
  def SVSQSHRUN_X2 : SInst<"svqshrun[_n]_{0}[_{d}_x2]", "e2i", "is",  MergeNone, "aarch64_sve_sqshrun_x2", [VerifyRuntimeMode], [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
}

````
- **L2325 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme-f16f16" in {`.
  **L2325 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme-f16f16" in {`。
- **L2326 EN**: Declares TableGen def record `SVCVT_F32_X2`.
  **L2326 CN**: 声明 TableGen def 记录 `SVCVT_F32_X2`。
- **L2327 EN**: Closes the current lexical scope or compound statement.
  **L2327 CN**: 结束当前词法作用域或复合语句块。
- **L2328 EN**: Blank line separating nearby declarations or logic blocks.
  **L2328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2329 EN**: Separator comment used for visual grouping.
  **L2329 CN**: 用于视觉分组的分隔注释。
- **L2330 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector floating-point convert from single-precision to interleaved half-precision/BFloat16`.
  **L2330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector floating-point convert from single-precision to interleaved half-precision/BFloat16`。
- **L2331 EN**: Separator comment used for visual grouping.
  **L2331 CN**: 用于视觉分组的分隔注释。
- **L2332 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2332 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2333 EN**: Declares TableGen def record `SVCVTN_F16_X2`.
  **L2333 CN**: 声明 TableGen def 记录 `SVCVTN_F16_X2`。
- **L2334 EN**: Declares TableGen def record `SVCVTN_BF16_X2`.
  **L2334 CN**: 声明 TableGen def 记录 `SVCVTN_BF16_X2`。
- **L2335 EN**: Closes the current lexical scope or compound statement.
  **L2335 CN**: 结束当前词法作用域或复合语句块。
- **L2336 EN**: Blank line separating nearby declarations or logic blocks.
  **L2336 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2337 EN**: Separator comment used for visual grouping.
  **L2337 CN**: 用于视觉分组的分隔注释。
- **L2338 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector floating-point convert from half-precision to deinterleaved single-precision.`.
  **L2338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector floating-point convert from half-precision to deinterleaved single-precision.`。
- **L2339 EN**: Separator comment used for visual grouping.
  **L2339 CN**: 用于视觉分组的分隔注释。
- **L2340 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme-f16f16" in {`.
  **L2340 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme-f16f16" in {`。
- **L2341 EN**: Declares TableGen def record `SVCVTL_F32_X2`.
  **L2341 CN**: 声明 TableGen def 记录 `SVCVTL_F32_X2`。
- **L2342 EN**: Closes the current lexical scope or compound statement.
  **L2342 CN**: 结束当前词法作用域或复合语句块。
- **L2343 EN**: Blank line separating nearby declarations or logic blocks.
  **L2343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2344 EN**: Separator comment used for visual grouping.
  **L2344 CN**: 用于视觉分组的分隔注释。
- **L2345 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector saturating shift right narrow and interleave`.
  **L2345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector saturating shift right narrow and interleave`。
- **L2346 EN**: Separator comment used for visual grouping.
  **L2346 CN**: 用于视觉分组的分隔注释。
- **L2347 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`.
  **L2347 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`。
- **L2348 EN**: Declares TableGen def record `SVSQSHRN_X2`.
  **L2348 CN**: 声明 TableGen def 记录 `SVSQSHRN_X2`。
- **L2349 EN**: Declares TableGen def record `SVUQSHRN_X2`.
  **L2349 CN**: 声明 TableGen def 记录 `SVUQSHRN_X2`。
- **L2350 EN**: Declares TableGen def record `SVSQSHRUN_X2`.
  **L2350 CN**: 声明 TableGen def 记录 `SVSQSHRUN_X2`。
- **L2351 EN**: Closes the current lexical scope or compound statement.
  **L2351 CN**: 结束当前词法作用域或复合语句块。
- **L2352 EN**: Blank line separating nearby declarations or logic blocks.
  **L2352 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2353-2380

````tablegen
//
// Multi-vector saturating extract narrow
//
let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  def SVQCVT_S16_S32_X2 : SInst<"svqcvt_s16[_{d}_x2]", "h2.d", "i",  MergeNone, "aarch64_sve_sqcvt_x2", [IsStreaming], []>;
  def SVQCVT_U16_U32_X2 : SInst<"svqcvt_u16[_{d}_x2]", "e2.d", "Ui", MergeNone, "aarch64_sve_uqcvt_x2", [IsStreaming], []>;
  def SVQCVT_U16_S32_X2 : SInst<"svqcvt_u16[_{d}_x2]", "e2.d", "i",  MergeNone, "aarch64_sve_sqcvtu_x2", [IsStreaming], []>;

  def SVQCVT_S8_S32_X4 : SInst<"svqcvt_s8[_{d}_x4]", "q4.d", "i",  MergeNone, "aarch64_sve_sqcvt_x4", [IsStreaming], []>;
  def SVQCVT_U8_U32_X4 : SInst<"svqcvt_u8[_{d}_x4]", "b4.d", "Ui", MergeNone, "aarch64_sve_uqcvt_x4", [IsStreaming], []>;
  def SVQCVT_U8_S32_X4 : SInst<"svqcvt_u8[_{d}_x4]", "b4.d", "i",  MergeNone, "aarch64_sve_sqcvtu_x4", [IsStreaming], []>;

  def SVQCVT_S16_S64_X4 : SInst<"svqcvt_s16[_{d}_x4]", "q4.d", "l",  MergeNone, "aarch64_sve_sqcvt_x4", [IsStreaming], []>;
  def SVQCVT_U16_U64_X4 : SInst<"svqcvt_u16[_{d}_x4]", "b4.d", "Ul", MergeNone, "aarch64_sve_uqcvt_x4", [IsStreaming], []>;
  def SVQCVT_U16_S64_X4 : SInst<"svqcvt_u16[_{d}_x4]", "b4.d", "l",  MergeNone, "aarch64_sve_sqcvtu_x4", [IsStreaming], []>;
}

//
// Multi-vector saturating extract narrow and interleave
//
let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {
  def SVQCVTN_S16_S32_X2 : SInst<"svqcvtn_s16[_{d}_x2]", "h2.d", "i",  MergeNone, "aarch64_sve_sqcvtn_x2", [VerifyRuntimeMode], []>;
  def SVQCVTN_U16_U32_X2 : SInst<"svqcvtn_u16[_{d}_x2]", "e2.d", "Ui", MergeNone, "aarch64_sve_uqcvtn_x2", [VerifyRuntimeMode], []>;
  def SVQCVTN_U16_S32_X2 : SInst<"svqcvtn_u16[_{d}_x2]", "e2.d", "i",  MergeNone, "aarch64_sve_sqcvtun_x2", [VerifyRuntimeMode], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  def SVQCVTN_S8_S32_X4 : SInst<"svqcvtn_s8[_{d}_x4]", "q4.d", "i",  MergeNone, "aarch64_sve_sqcvtn_x4", [IsStreaming], []>;
````
- **L2353 EN**: Separator comment used for visual grouping.
  **L2353 CN**: 用于视觉分组的分隔注释。
- **L2354 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector saturating extract narrow`.
  **L2354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector saturating extract narrow`。
- **L2355 EN**: Separator comment used for visual grouping.
  **L2355 CN**: 用于视觉分组的分隔注释。
- **L2356 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2356 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2357 EN**: Declares TableGen def record `SVQCVT_S16_S32_X2`.
  **L2357 CN**: 声明 TableGen def 记录 `SVQCVT_S16_S32_X2`。
- **L2358 EN**: Declares TableGen def record `SVQCVT_U16_U32_X2`.
  **L2358 CN**: 声明 TableGen def 记录 `SVQCVT_U16_U32_X2`。
- **L2359 EN**: Declares TableGen def record `SVQCVT_U16_S32_X2`.
  **L2359 CN**: 声明 TableGen def 记录 `SVQCVT_U16_S32_X2`。
- **L2360 EN**: Blank line separating nearby declarations or logic blocks.
  **L2360 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2361 EN**: Declares TableGen def record `SVQCVT_S8_S32_X4`.
  **L2361 CN**: 声明 TableGen def 记录 `SVQCVT_S8_S32_X4`。
- **L2362 EN**: Declares TableGen def record `SVQCVT_U8_U32_X4`.
  **L2362 CN**: 声明 TableGen def 记录 `SVQCVT_U8_U32_X4`。
- **L2363 EN**: Declares TableGen def record `SVQCVT_U8_S32_X4`.
  **L2363 CN**: 声明 TableGen def 记录 `SVQCVT_U8_S32_X4`。
- **L2364 EN**: Blank line separating nearby declarations or logic blocks.
  **L2364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2365 EN**: Declares TableGen def record `SVQCVT_S16_S64_X4`.
  **L2365 CN**: 声明 TableGen def 记录 `SVQCVT_S16_S64_X4`。
- **L2366 EN**: Declares TableGen def record `SVQCVT_U16_U64_X4`.
  **L2366 CN**: 声明 TableGen def 记录 `SVQCVT_U16_U64_X4`。
- **L2367 EN**: Declares TableGen def record `SVQCVT_U16_S64_X4`.
  **L2367 CN**: 声明 TableGen def 记录 `SVQCVT_U16_S64_X4`。
- **L2368 EN**: Closes the current lexical scope or compound statement.
  **L2368 CN**: 结束当前词法作用域或复合语句块。
- **L2369 EN**: Blank line separating nearby declarations or logic blocks.
  **L2369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2370 EN**: Separator comment used for visual grouping.
  **L2370 CN**: 用于视觉分组的分隔注释。
- **L2371 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector saturating extract narrow and interleave`.
  **L2371 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector saturating extract narrow and interleave`。
- **L2372 EN**: Separator comment used for visual grouping.
  **L2372 CN**: 用于视觉分组的分隔注释。
- **L2373 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2373 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2374 EN**: Declares TableGen def record `SVQCVTN_S16_S32_X2`.
  **L2374 CN**: 声明 TableGen def 记录 `SVQCVTN_S16_S32_X2`。
- **L2375 EN**: Declares TableGen def record `SVQCVTN_U16_U32_X2`.
  **L2375 CN**: 声明 TableGen def 记录 `SVQCVTN_U16_U32_X2`。
- **L2376 EN**: Declares TableGen def record `SVQCVTN_U16_S32_X2`.
  **L2376 CN**: 声明 TableGen def 记录 `SVQCVTN_U16_S32_X2`。
- **L2377 EN**: Closes the current lexical scope or compound statement.
  **L2377 CN**: 结束当前词法作用域或复合语句块。
- **L2378 EN**: Blank line separating nearby declarations or logic blocks.
  **L2378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2379 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2379 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2380 EN**: Declares TableGen def record `SVQCVTN_S8_S32_X4`.
  **L2380 CN**: 声明 TableGen def 记录 `SVQCVTN_S8_S32_X4`。

### Lines 2381-2408

````tablegen
  def SVQCVTN_U8_U32_X4 : SInst<"svqcvtn_u8[_{d}_x4]", "b4.d", "Ui", MergeNone, "aarch64_sve_uqcvtn_x4", [IsStreaming], []>;
  def SVQCVTN_U8_S32_X4 : SInst<"svqcvtn_u8[_{d}_x4]", "b4.d", "i",  MergeNone, "aarch64_sve_sqcvtun_x4", [IsStreaming], []>;

  def SVQCVTN_S16_S64_X4 : SInst<"svqcvtn_s16[_{d}_x4]", "q4.d", "l",  MergeNone, "aarch64_sve_sqcvtn_x4", [IsStreaming], []>;
  def SVQCVTN_U16_U64_X4 : SInst<"svqcvtn_u16[_{d}_x4]", "b4.d", "Ul", MergeNone, "aarch64_sve_uqcvtn_x4", [IsStreaming], []>;
  def SVQCVTN_U16_S64_X4 : SInst<"svqcvtn_u16[_{d}_x4]", "b4.d", "l",  MergeNone, "aarch64_sve_sqcvtun_x4", [IsStreaming], []>;
}

//
// Multi-vector zip/unzip
//

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  def SVZIP_X2  : SInst<"svzip[_{d}_x2]",  "22", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_zip_x2",  [IsStreaming], []>;
  def SVZIPQ_X2 : SInst<"svzipq[_{d}_x2]", "22", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_zipq_x2", [IsStreaming], []>;
  def SVZIP_X4  : SInst<"svzip[_{d}_x4]",  "44", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_zip_x4",  [IsStreaming], []>;
  def SVZIPQ_X4 : SInst<"svzipq[_{d}_x4]", "44", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_zipq_x4", [IsStreaming], []>;

  def SVUZP_X2  : SInst<"svuzp[_{d}_x2]",  "22", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_uzp_x2",  [IsStreaming], []>;
  def SVUZPQ_X2 : SInst<"svuzpq[_{d}_x2]", "22", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_uzpq_x2", [IsStreaming], []>;
  def SVUZP_X4  : SInst<"svuzp[_{d}_x4]",  "44", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_uzp_x4",  [IsStreaming], []>;
  def SVUZPQ_X4 : SInst<"svuzpq[_{d}_x4]", "44", "cUcsUsiUilUlmbhfd", MergeNone, "aarch64_sve_uzpq_x4", [IsStreaming], []>;
}

//
// Multi-vector unpack
//

````
- **L2381 EN**: Declares TableGen def record `SVQCVTN_U8_U32_X4`.
  **L2381 CN**: 声明 TableGen def 记录 `SVQCVTN_U8_U32_X4`。
- **L2382 EN**: Declares TableGen def record `SVQCVTN_U8_S32_X4`.
  **L2382 CN**: 声明 TableGen def 记录 `SVQCVTN_U8_S32_X4`。
- **L2383 EN**: Blank line separating nearby declarations or logic blocks.
  **L2383 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2384 EN**: Declares TableGen def record `SVQCVTN_S16_S64_X4`.
  **L2384 CN**: 声明 TableGen def 记录 `SVQCVTN_S16_S64_X4`。
- **L2385 EN**: Declares TableGen def record `SVQCVTN_U16_U64_X4`.
  **L2385 CN**: 声明 TableGen def 记录 `SVQCVTN_U16_U64_X4`。
- **L2386 EN**: Declares TableGen def record `SVQCVTN_U16_S64_X4`.
  **L2386 CN**: 声明 TableGen def 记录 `SVQCVTN_U16_S64_X4`。
- **L2387 EN**: Closes the current lexical scope or compound statement.
  **L2387 CN**: 结束当前词法作用域或复合语句块。
- **L2388 EN**: Blank line separating nearby declarations or logic blocks.
  **L2388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2389 EN**: Separator comment used for visual grouping.
  **L2389 CN**: 用于视觉分组的分隔注释。
- **L2390 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector zip/unzip`.
  **L2390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector zip/unzip`。
- **L2391 EN**: Separator comment used for visual grouping.
  **L2391 CN**: 用于视觉分组的分隔注释。
- **L2392 EN**: Blank line separating nearby declarations or logic blocks.
  **L2392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2393 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2393 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2394 EN**: Declares TableGen def record `SVZIP_X2`.
  **L2394 CN**: 声明 TableGen def 记录 `SVZIP_X2`。
- **L2395 EN**: Declares TableGen def record `SVZIPQ_X2`.
  **L2395 CN**: 声明 TableGen def 记录 `SVZIPQ_X2`。
- **L2396 EN**: Declares TableGen def record `SVZIP_X4`.
  **L2396 CN**: 声明 TableGen def 记录 `SVZIP_X4`。
- **L2397 EN**: Declares TableGen def record `SVZIPQ_X4`.
  **L2397 CN**: 声明 TableGen def 记录 `SVZIPQ_X4`。
- **L2398 EN**: Blank line separating nearby declarations or logic blocks.
  **L2398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2399 EN**: Declares TableGen def record `SVUZP_X2`.
  **L2399 CN**: 声明 TableGen def 记录 `SVUZP_X2`。
- **L2400 EN**: Declares TableGen def record `SVUZPQ_X2`.
  **L2400 CN**: 声明 TableGen def 记录 `SVUZPQ_X2`。
- **L2401 EN**: Declares TableGen def record `SVUZP_X4`.
  **L2401 CN**: 声明 TableGen def 记录 `SVUZP_X4`。
- **L2402 EN**: Declares TableGen def record `SVUZPQ_X4`.
  **L2402 CN**: 声明 TableGen def 记录 `SVUZPQ_X4`。
- **L2403 EN**: Closes the current lexical scope or compound statement.
  **L2403 CN**: 结束当前词法作用域或复合语句块。
- **L2404 EN**: Blank line separating nearby declarations or logic blocks.
  **L2404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2405 EN**: Separator comment used for visual grouping.
  **L2405 CN**: 用于视觉分组的分隔注释。
- **L2406 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector unpack`.
  **L2406 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector unpack`。
- **L2407 EN**: Separator comment used for visual grouping.
  **L2407 CN**: 用于视觉分组的分隔注释。
- **L2408 EN**: Blank line separating nearby declarations or logic blocks.
  **L2408 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2409-2436

````tablegen
let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {
  def SVSUNPK_X2 : SInst<"svunpk_{d}[_{1}_x2]", "2h",   "sil",    MergeNone, "aarch64_sve_sunpk_x2", [IsStreaming], []>;
  def SVUUNPK_X2 : SInst<"svunpk_{d}[_{1}_x2]", "2h",   "UsUiUl", MergeNone, "aarch64_sve_uunpk_x2", [IsStreaming], []>;
  def SVSUNPK_X4 : SInst<"svunpk_{d}[_{1}_x4]", "42.h", "sil",    MergeNone, "aarch64_sve_sunpk_x4", [IsStreaming], []>;
  def SVUUNPK_X4 : SInst<"svunpk_{d}[_{1}_x4]", "42.h", "UsUiUl", MergeNone, "aarch64_sve_uunpk_x4", [IsStreaming], []>;
}

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,fp8" in {
  // Multi-vector scaling
  def FSCALE_SINGLE_X2 : Inst<"svscale[_single_{d}_x2]", "22x", "fhd", MergeNone, "aarch64_sme_fp8_scale_single_x2", [IsStreaming],[]>;
  def FSCALE_SINGLE_X4 : Inst<"svscale[_single_{d}_x4]", "44x", "fhd", MergeNone, "aarch64_sme_fp8_scale_single_x4", [IsStreaming],[]>;

  def FSCALE_X2 : Inst<"svscale[_{d}_x2]", "222.x", "fhd", MergeNone, "aarch64_sme_fp8_scale_x2", [IsStreaming],[]>;
  def FSCALE_X4 : Inst<"svscale[_{d}_x4]", "444.x", "fhd", MergeNone, "aarch64_sme_fp8_scale_x4", [IsStreaming],[]>;

  // Convert from FP8 to half-precision/BFloat16 multi-vector
  def SVF1CVT_X2 : Inst<"svcvt1_{d}[_mf8]_x2", "2~>", "bh", MergeNone, "aarch64_sve_fp8_cvt1_x2", [IsStreaming], []>;
  def SVF2CVT_X2 : Inst<"svcvt2_{d}[_mf8]_x2", "2~>", "bh", MergeNone, "aarch64_sve_fp8_cvt2_x2", [IsStreaming], []>;

  // Convert from FP8 to deinterleaved half-precision/BFloat16 multi-vector
  def SVF1CVTL_X2 : Inst<"svcvtl1_{d}[_mf8]_x2",  "2~>", "bh", MergeNone, "aarch64_sve_fp8_cvtl1_x2",  [IsStreaming], []>;
  def SVF2CVTL_X2 : Inst<"svcvtl2_{d}[_mf8]_x2",  "2~>", "bh", MergeNone, "aarch64_sve_fp8_cvtl2_x2",  [IsStreaming], []>;

  // Convert from single/half/bfloat multivector to FP8
  def SVFCVT_X2 : Inst<"svcvt_mf8[_{d}_x2]", "~2>", "bh", MergeNone, "aarch64_sve_fp8_cvt_x2", [IsStreaming], []>;
  def SVFCVT_X4 : Inst<"svcvt_mf8[_{d}_x4]", "~4>", "f",  MergeNone, "aarch64_sve_fp8_cvt_x4", [IsOverloadNone, IsStreaming], []>;
  // interleaved
  def SVFCVTN_X4 : Inst<"svcvtn_mf8[_{d}_x4]", "~4>", "f", MergeNone, "aarch64_sve_fp8_cvtn_x4", [IsOverloadNone, IsStreaming], []>;
````
- **L2409 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`.
  **L2409 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2" in {`。
- **L2410 EN**: Declares TableGen def record `SVSUNPK_X2`.
  **L2410 CN**: 声明 TableGen def 记录 `SVSUNPK_X2`。
- **L2411 EN**: Declares TableGen def record `SVUUNPK_X2`.
  **L2411 CN**: 声明 TableGen def 记录 `SVUUNPK_X2`。
- **L2412 EN**: Declares TableGen def record `SVSUNPK_X4`.
  **L2412 CN**: 声明 TableGen def 记录 `SVSUNPK_X4`。
- **L2413 EN**: Declares TableGen def record `SVUUNPK_X4`.
  **L2413 CN**: 声明 TableGen def 记录 `SVUUNPK_X4`。
- **L2414 EN**: Closes the current lexical scope or compound statement.
  **L2414 CN**: 结束当前词法作用域或复合语句块。
- **L2415 EN**: Blank line separating nearby declarations or logic blocks.
  **L2415 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2416 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,fp8" in {`.
  **L2416 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2,fp8" in {`。
- **L2417 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector scaling`.
  **L2417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector scaling`。
- **L2418 EN**: Declares TableGen def record `FSCALE_SINGLE_X2`.
  **L2418 CN**: 声明 TableGen def 记录 `FSCALE_SINGLE_X2`。
- **L2419 EN**: Declares TableGen def record `FSCALE_SINGLE_X4`.
  **L2419 CN**: 声明 TableGen def 记录 `FSCALE_SINGLE_X4`。
- **L2420 EN**: Blank line separating nearby declarations or logic blocks.
  **L2420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2421 EN**: Declares TableGen def record `FSCALE_X2`.
  **L2421 CN**: 声明 TableGen def 记录 `FSCALE_X2`。
- **L2422 EN**: Declares TableGen def record `FSCALE_X4`.
  **L2422 CN**: 声明 TableGen def 记录 `FSCALE_X4`。
- **L2423 EN**: Blank line separating nearby declarations or logic blocks.
  **L2423 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2424 EN**: Comment explains nearby logic, constraints, or intent: `Convert from FP8 to half-precision/BFloat16 multi-vector`.
  **L2424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert from FP8 to half-precision/BFloat16 multi-vector`。
- **L2425 EN**: Declares TableGen def record `SVF1CVT_X2`.
  **L2425 CN**: 声明 TableGen def 记录 `SVF1CVT_X2`。
- **L2426 EN**: Declares TableGen def record `SVF2CVT_X2`.
  **L2426 CN**: 声明 TableGen def 记录 `SVF2CVT_X2`。
- **L2427 EN**: Blank line separating nearby declarations or logic blocks.
  **L2427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2428 EN**: Comment explains nearby logic, constraints, or intent: `Convert from FP8 to deinterleaved half-precision/BFloat16 multi-vector`.
  **L2428 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert from FP8 to deinterleaved half-precision/BFloat16 multi-vector`。
- **L2429 EN**: Declares TableGen def record `SVF1CVTL_X2`.
  **L2429 CN**: 声明 TableGen def 记录 `SVF1CVTL_X2`。
- **L2430 EN**: Declares TableGen def record `SVF2CVTL_X2`.
  **L2430 CN**: 声明 TableGen def 记录 `SVF2CVTL_X2`。
- **L2431 EN**: Blank line separating nearby declarations or logic blocks.
  **L2431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2432 EN**: Comment explains nearby logic, constraints, or intent: `Convert from single/half/bfloat multivector to FP8`.
  **L2432 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert from single/half/bfloat multivector to FP8`。
- **L2433 EN**: Declares TableGen def record `SVFCVT_X2`.
  **L2433 CN**: 声明 TableGen def 记录 `SVFCVT_X2`。
- **L2434 EN**: Declares TableGen def record `SVFCVT_X4`.
  **L2434 CN**: 声明 TableGen def 记录 `SVFCVT_X4`。
- **L2435 EN**: Comment explains nearby logic, constraints, or intent: `interleaved`.
  **L2435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`interleaved`。
- **L2436 EN**: Declares TableGen def record `SVFCVTN_X4`.
  **L2436 CN**: 声明 TableGen def 记录 `SVFCVTN_X4`。

### Lines 2437-2464

````tablegen
}

let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {
// == BFloat16 multiply-subtract ==
  def SVBFMLSLB : SInst<"svbfmlslb[_{d}]", "dd$$", "f", MergeNone, "aarch64_sve_bfmlslb", [IsOverloadNone, VerifyRuntimeMode], []>;
  def SVBFMLSLT : SInst<"svbfmlslt[_{d}]", "dd$$", "f", MergeNone, "aarch64_sve_bfmlslt", [IsOverloadNone, VerifyRuntimeMode], []>;

  def SVBFMLSLB_LANE : SInst<"svbfmlslb_lane[_{d}]", "dd$$i", "f", MergeNone, "aarch64_sve_bfmlslb_lane", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
  def SVBFMLSLT_LANE : SInst<"svbfmlslt_lane[_{d}]", "dd$$i", "f", MergeNone, "aarch64_sve_bfmlslt_lane", [IsOverloadNone, VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
}

let SVETargetGuard = "(sve2|sme2),faminmax", SMETargetGuard = "sme2,faminmax" in {
  defm SVAMIN : SInstZPZZ<"svamin", "hfd", "aarch64_sve_famin", "aarch64_sve_famin_u">;
  defm SVAMAX : SInstZPZZ<"svamax", "hfd", "aarch64_sve_famax", "aarch64_sve_famax_u">;
}

let SVETargetGuard = "(sve2|sme2),fp8", SMETargetGuard = "sme2,fp8" in {
  // SVE FP8 widening conversions

  // 8-bit floating-point convert to BFloat16/Float16
  def SVF1CVT : SInst<"svcvt1_{d}[_mf8]", "d~>", "bh", MergeNone, "aarch64_sve_fp8_cvt1", [VerifyRuntimeMode]>;
  def SVF2CVT : SInst<"svcvt2_{d}[_mf8]", "d~>", "bh", MergeNone, "aarch64_sve_fp8_cvt2", [VerifyRuntimeMode]>;

  // 8-bit floating-point convert to BFloat16/Float16 (top)
  def SVF1CVTLT : SInst<"svcvtlt1_{d}[_mf8]", "d~>", "bh", MergeNone, "aarch64_sve_fp8_cvtlt1", [VerifyRuntimeMode]>;
  def SVF2CVTLT : SInst<"svcvtlt2_{d}[_mf8]", "d~>", "bh", MergeNone, "aarch64_sve_fp8_cvtlt2", [VerifyRuntimeMode]>;

  // BFloat16/Float16 convert, narrow and interleave to 8-bit floating-point
````
- **L2437 EN**: Closes the current lexical scope or compound statement.
  **L2437 CN**: 结束当前词法作用域或复合语句块。
- **L2438 EN**: Blank line separating nearby declarations or logic blocks.
  **L2438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2439 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`.
  **L2439 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p1|sme2", SMETargetGuard = "sve2p1|sme2" in {`。
- **L2440 EN**: Comment explains nearby logic, constraints, or intent: `BFloat16 multiply-subtract`.
  **L2440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`BFloat16 multiply-subtract`。
- **L2441 EN**: Declares TableGen def record `SVBFMLSLB`.
  **L2441 CN**: 声明 TableGen def 记录 `SVBFMLSLB`。
- **L2442 EN**: Declares TableGen def record `SVBFMLSLT`.
  **L2442 CN**: 声明 TableGen def 记录 `SVBFMLSLT`。
- **L2443 EN**: Blank line separating nearby declarations or logic blocks.
  **L2443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2444 EN**: Declares TableGen def record `SVBFMLSLB_LANE`.
  **L2444 CN**: 声明 TableGen def 记录 `SVBFMLSLB_LANE`。
- **L2445 EN**: Declares TableGen def record `SVBFMLSLT_LANE`.
  **L2445 CN**: 声明 TableGen def 记录 `SVBFMLSLT_LANE`。
- **L2446 EN**: Closes the current lexical scope or compound statement.
  **L2446 CN**: 结束当前词法作用域或复合语句块。
- **L2447 EN**: Blank line separating nearby declarations or logic blocks.
  **L2447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2448 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "(sve2|sme2),faminmax", SMETargetGuard = "sme2,faminmax" in {`.
  **L2448 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "(sve2|sme2),faminmax", SMETargetGuard = "sme2,faminmax" in {`。
- **L2449 EN**: Declares TableGen defm record `SVAMIN`.
  **L2449 CN**: 声明 TableGen defm 记录 `SVAMIN`。
- **L2450 EN**: Declares TableGen defm record `SVAMAX`.
  **L2450 CN**: 声明 TableGen defm 记录 `SVAMAX`。
- **L2451 EN**: Closes the current lexical scope or compound statement.
  **L2451 CN**: 结束当前词法作用域或复合语句块。
- **L2452 EN**: Blank line separating nearby declarations or logic blocks.
  **L2452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2453 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "(sve2|sme2),fp8", SMETargetGuard = "sme2,fp8" in {`.
  **L2453 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "(sve2|sme2),fp8", SMETargetGuard = "sme2,fp8" in {`。
- **L2454 EN**: Comment explains nearby logic, constraints, or intent: `SVE FP8 widening conversions`.
  **L2454 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SVE FP8 widening conversions`。
- **L2455 EN**: Blank line separating nearby declarations or logic blocks.
  **L2455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2456 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point convert to BFloat16/Float16`.
  **L2456 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point convert to BFloat16/Float16`。
- **L2457 EN**: Declares TableGen def record `SVF1CVT`.
  **L2457 CN**: 声明 TableGen def 记录 `SVF1CVT`。
- **L2458 EN**: Declares TableGen def record `SVF2CVT`.
  **L2458 CN**: 声明 TableGen def 记录 `SVF2CVT`。
- **L2459 EN**: Blank line separating nearby declarations or logic blocks.
  **L2459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2460 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point convert to BFloat16/Float16 (top)`.
  **L2460 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point convert to BFloat16/Float16 (top)`。
- **L2461 EN**: Declares TableGen def record `SVF1CVTLT`.
  **L2461 CN**: 声明 TableGen def 记录 `SVF1CVTLT`。
- **L2462 EN**: Declares TableGen def record `SVF2CVTLT`.
  **L2462 CN**: 声明 TableGen def 记录 `SVF2CVTLT`。
- **L2463 EN**: Blank line separating nearby declarations or logic blocks.
  **L2463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2464 EN**: Comment explains nearby logic, constraints, or intent: `BFloat16/Float16 convert, narrow and interleave to 8-bit floating-point`.
  **L2464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`BFloat16/Float16 convert, narrow and interleave to 8-bit floating-point`。

### Lines 2465-2492

````tablegen
  def SVFCVTN : SInst<"svcvtn_mf8[_{d}_x2]", "~2>", "bh", MergeNone, "aarch64_sve_fp8_cvtn", [VerifyRuntimeMode]>;

  // Single-precision convert, narrow and interleave to 8-bit floating-point (top and bottom)
  def SVFCVTNB : SInst<"svcvtnb_mf8[_f32_x2]", "~2>",  "f", MergeNone, "aarch64_sve_fp8_cvtnb", [VerifyRuntimeMode]>;
  def SVFCVTNT : SInst<"svcvtnt_mf8[_f32_x2]", "~~2>", "f", MergeNone, "aarch64_sve_fp8_cvtnt", [VerifyRuntimeMode]>;
}

let SVETargetGuard = "sve2,fp8dot2", SMETargetGuard ="ssve-fp8dot2" in {
  // 8-bit floating-point dot product to half-precision (vectors)
  def SVFDOT_2WAY   :  SInst<"svdot[_f16_mf8]",   "dd~~>", "h", MergeNone, "aarch64_sve_fp8_fdot", [VerifyRuntimeMode]>;
  def SVFDOT_N_2WAY :  SInst<"svdot[_n_f16_mf8]", "dd~!>", "h", MergeNone, "aarch64_sve_fp8_fdot", [VerifyRuntimeMode]>;

  // 8-bit floating-point dot product to half-precision (indexed)
  def SVFDOT_LANE_2WAY :  SInst<"svdot_lane[_f16_mf8]", "dd~~i>", "h", MergeNone, "aarch64_sve_fp8_fdot_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
}

let SVETargetGuard = "sve2,fp8dot4", SMETargetGuard ="ssve-fp8dot4" in {
  // 8-bit floating-point dot product to single-precision (vectors)
  def SVFDOT_4WAY   : SInst<"svdot[_f32_mf8]",   "dd~~>", "f", MergeNone, "aarch64_sve_fp8_fdot", [VerifyRuntimeMode]>;
  def SVFDOT_N_4WAY : SInst<"svdot[_n_f32_mf8]", "dd~!>", "f", MergeNone, "aarch64_sve_fp8_fdot", [VerifyRuntimeMode]>;

  // 8-bit floating-point dot product to single-precision (indexed)
  def SVFDOT_LANE_4WAY :  SInst<"svdot_lane[_f32_mf8]", "dd~~i>", "f", MergeNone, "aarch64_sve_fp8_fdot_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_3>]>;
}

let SVETargetGuard = "sve2,fp8fma", SMETargetGuard = "ssve-fp8fma" in {
  // 8-bit floating-point multiply-add long to half-precision (bottom)
  def SVFMLALB   : SInst<"svmlalb[_f16_mf8]",   "dd~~>", "h", MergeNone, "aarch64_sve_fp8_fmlalb", [VerifyRuntimeMode]>;
````
- **L2465 EN**: Declares TableGen def record `SVFCVTN`.
  **L2465 CN**: 声明 TableGen def 记录 `SVFCVTN`。
- **L2466 EN**: Blank line separating nearby declarations or logic blocks.
  **L2466 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2467 EN**: Comment explains nearby logic, constraints, or intent: `Single-precision convert, narrow and interleave to 8-bit floating-point (top and bottom)`.
  **L2467 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Single-precision convert, narrow and interleave to 8-bit floating-point (top and bottom)`。
- **L2468 EN**: Declares TableGen def record `SVFCVTNB`.
  **L2468 CN**: 声明 TableGen def 记录 `SVFCVTNB`。
- **L2469 EN**: Declares TableGen def record `SVFCVTNT`.
  **L2469 CN**: 声明 TableGen def 记录 `SVFCVTNT`。
- **L2470 EN**: Closes the current lexical scope or compound statement.
  **L2470 CN**: 结束当前词法作用域或复合语句块。
- **L2471 EN**: Blank line separating nearby declarations or logic blocks.
  **L2471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2472 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2,fp8dot2", SMETargetGuard ="ssve-fp8dot2" in {`.
  **L2472 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2,fp8dot2", SMETargetGuard ="ssve-fp8dot2" in {`。
- **L2473 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point dot product to half-precision (vectors)`.
  **L2473 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point dot product to half-precision (vectors)`。
- **L2474 EN**: Declares TableGen def record `SVFDOT_2WAY`.
  **L2474 CN**: 声明 TableGen def 记录 `SVFDOT_2WAY`。
- **L2475 EN**: Declares TableGen def record `SVFDOT_N_2WAY`.
  **L2475 CN**: 声明 TableGen def 记录 `SVFDOT_N_2WAY`。
- **L2476 EN**: Blank line separating nearby declarations or logic blocks.
  **L2476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2477 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point dot product to half-precision (indexed)`.
  **L2477 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point dot product to half-precision (indexed)`。
- **L2478 EN**: Declares TableGen def record `SVFDOT_LANE_2WAY`.
  **L2478 CN**: 声明 TableGen def 记录 `SVFDOT_LANE_2WAY`。
- **L2479 EN**: Closes the current lexical scope or compound statement.
  **L2479 CN**: 结束当前词法作用域或复合语句块。
- **L2480 EN**: Blank line separating nearby declarations or logic blocks.
  **L2480 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2481 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2,fp8dot4", SMETargetGuard ="ssve-fp8dot4" in {`.
  **L2481 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2,fp8dot4", SMETargetGuard ="ssve-fp8dot4" in {`。
- **L2482 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point dot product to single-precision (vectors)`.
  **L2482 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point dot product to single-precision (vectors)`。
- **L2483 EN**: Declares TableGen def record `SVFDOT_4WAY`.
  **L2483 CN**: 声明 TableGen def 记录 `SVFDOT_4WAY`。
- **L2484 EN**: Declares TableGen def record `SVFDOT_N_4WAY`.
  **L2484 CN**: 声明 TableGen def 记录 `SVFDOT_N_4WAY`。
- **L2485 EN**: Blank line separating nearby declarations or logic blocks.
  **L2485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2486 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point dot product to single-precision (indexed)`.
  **L2486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point dot product to single-precision (indexed)`。
- **L2487 EN**: Declares TableGen def record `SVFDOT_LANE_4WAY`.
  **L2487 CN**: 声明 TableGen def 记录 `SVFDOT_LANE_4WAY`。
- **L2488 EN**: Closes the current lexical scope or compound statement.
  **L2488 CN**: 结束当前词法作用域或复合语句块。
- **L2489 EN**: Blank line separating nearby declarations or logic blocks.
  **L2489 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2490 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2,fp8fma", SMETargetGuard = "ssve-fp8fma" in {`.
  **L2490 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2,fp8fma", SMETargetGuard = "ssve-fp8fma" in {`。
- **L2491 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point multiply-add long to half-precision (bottom)`.
  **L2491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point multiply-add long to half-precision (bottom)`。
- **L2492 EN**: Declares TableGen def record `SVFMLALB`.
  **L2492 CN**: 声明 TableGen def 记录 `SVFMLALB`。

### Lines 2493-2520

````tablegen
  def SVFMLALB_N : SInst<"svmlalb[_n_f16_mf8]", "dd~!>", "h", MergeNone, "aarch64_sve_fp8_fmlalb", [VerifyRuntimeMode]>;

  // 8-bit floating-point multiply-add long to half-precision (bottom, indexed)
  def SVFMLALB_LANE : SInst<"svmlalb_lane[_f16_mf8]", "dd~~i>", "h", MergeNone, "aarch64_sve_fp8_fmlalb_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_15>]>;

  // 8-bit floating-point multiply-add long to half-precision (top)
  def SVFMLALT   : SInst<"svmlalt[_f16_mf8]",   "dd~~>", "h", MergeNone, "aarch64_sve_fp8_fmlalt", [VerifyRuntimeMode]>;
  def SVFMLALT_N : SInst<"svmlalt[_n_f16_mf8]", "dd~!>", "h", MergeNone, "aarch64_sve_fp8_fmlalt", [VerifyRuntimeMode]>;

  // 8-bit floating-point multiply-add long to half-precision (top, indexed)
  def SVFMLALT_LANE : SInst<"svmlalt_lane[_f16_mf8]", "dd~~i>", "h", MergeNone, "aarch64_sve_fp8_fmlalt_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_15>]>;

  // 8-bit floating-point multiply-add long long to single-precision (all top/bottom variants)
  def SVFMLALLBB   : SInst<"svmlallbb[_f32_mf8]",   "dd~~>", "f", MergeNone, "aarch64_sve_fp8_fmlallbb", [VerifyRuntimeMode]>;
  def SVFMLALLBB_N : SInst<"svmlallbb[_n_f32_mf8]", "dd~!>", "f", MergeNone, "aarch64_sve_fp8_fmlallbb", [VerifyRuntimeMode]>;
  def SVFMLALLBT   : SInst<"svmlallbt[_f32_mf8]",   "dd~~>", "f", MergeNone, "aarch64_sve_fp8_fmlallbt", [VerifyRuntimeMode]>;
  def SVFMLALLBT_N : SInst<"svmlallbt[_n_f32_mf8]", "dd~!>", "f", MergeNone, "aarch64_sve_fp8_fmlallbt", [VerifyRuntimeMode]>;
  def SVFMLALLTB   : SInst<"svmlalltb[_f32_mf8]",   "dd~~>", "f", MergeNone, "aarch64_sve_fp8_fmlalltb", [VerifyRuntimeMode]>;
  def SVFMLALLTB_N : SInst<"svmlalltb[_n_f32_mf8]", "dd~!>", "f", MergeNone, "aarch64_sve_fp8_fmlalltb", [VerifyRuntimeMode]>;
  def SVFMLALLTT   : SInst<"svmlalltt[_f32_mf8]",   "dd~~>", "f", MergeNone, "aarch64_sve_fp8_fmlalltt", [VerifyRuntimeMode]>;
  def SVFMLALLTT_N : SInst<"svmlalltt[_n_f32_mf8]", "dd~!>", "f", MergeNone, "aarch64_sve_fp8_fmlalltt", [VerifyRuntimeMode]>;

  // 8-bit floating-point multiply-add long long to single-precision (indexed, all top/bottom variants)
  def SVFMLALLBB_LANE : SInst<"svmlallbb_lane[_f32_mf8]", "dd~~i>", "f", MergeNone, "aarch64_sve_fp8_fmlallbb_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
  def SVFMLALLBT_LANE : SInst<"svmlallbt_lane[_f32_mf8]", "dd~~i>", "f", MergeNone, "aarch64_sve_fp8_fmlallbt_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
  def SVFMLALLTB_LANE : SInst<"svmlalltb_lane[_f32_mf8]", "dd~~i>", "f", MergeNone, "aarch64_sve_fp8_fmlalltb_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
  def SVFMLALLTT_LANE : SInst<"svmlalltt_lane[_f32_mf8]", "dd~~i>", "f", MergeNone, "aarch64_sve_fp8_fmlalltt_lane", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
}
````
- **L2493 EN**: Declares TableGen def record `SVFMLALB_N`.
  **L2493 CN**: 声明 TableGen def 记录 `SVFMLALB_N`。
- **L2494 EN**: Blank line separating nearby declarations or logic blocks.
  **L2494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2495 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point multiply-add long to half-precision (bottom, indexed)`.
  **L2495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point multiply-add long to half-precision (bottom, indexed)`。
- **L2496 EN**: Declares TableGen def record `SVFMLALB_LANE`.
  **L2496 CN**: 声明 TableGen def 记录 `SVFMLALB_LANE`。
- **L2497 EN**: Blank line separating nearby declarations or logic blocks.
  **L2497 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2498 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point multiply-add long to half-precision (top)`.
  **L2498 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point multiply-add long to half-precision (top)`。
- **L2499 EN**: Declares TableGen def record `SVFMLALT`.
  **L2499 CN**: 声明 TableGen def 记录 `SVFMLALT`。
- **L2500 EN**: Declares TableGen def record `SVFMLALT_N`.
  **L2500 CN**: 声明 TableGen def 记录 `SVFMLALT_N`。
- **L2501 EN**: Blank line separating nearby declarations or logic blocks.
  **L2501 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2502 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point multiply-add long to half-precision (top, indexed)`.
  **L2502 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point multiply-add long to half-precision (top, indexed)`。
- **L2503 EN**: Declares TableGen def record `SVFMLALT_LANE`.
  **L2503 CN**: 声明 TableGen def 记录 `SVFMLALT_LANE`。
- **L2504 EN**: Blank line separating nearby declarations or logic blocks.
  **L2504 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2505 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point multiply-add long long to single-precision (all top/bottom variants)`.
  **L2505 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point multiply-add long long to single-precision (all top/bottom variants)`。
- **L2506 EN**: Declares TableGen def record `SVFMLALLBB`.
  **L2506 CN**: 声明 TableGen def 记录 `SVFMLALLBB`。
- **L2507 EN**: Declares TableGen def record `SVFMLALLBB_N`.
  **L2507 CN**: 声明 TableGen def 记录 `SVFMLALLBB_N`。
- **L2508 EN**: Declares TableGen def record `SVFMLALLBT`.
  **L2508 CN**: 声明 TableGen def 记录 `SVFMLALLBT`。
- **L2509 EN**: Declares TableGen def record `SVFMLALLBT_N`.
  **L2509 CN**: 声明 TableGen def 记录 `SVFMLALLBT_N`。
- **L2510 EN**: Declares TableGen def record `SVFMLALLTB`.
  **L2510 CN**: 声明 TableGen def 记录 `SVFMLALLTB`。
- **L2511 EN**: Declares TableGen def record `SVFMLALLTB_N`.
  **L2511 CN**: 声明 TableGen def 记录 `SVFMLALLTB_N`。
- **L2512 EN**: Declares TableGen def record `SVFMLALLTT`.
  **L2512 CN**: 声明 TableGen def 记录 `SVFMLALLTT`。
- **L2513 EN**: Declares TableGen def record `SVFMLALLTT_N`.
  **L2513 CN**: 声明 TableGen def 记录 `SVFMLALLTT_N`。
- **L2514 EN**: Blank line separating nearby declarations or logic blocks.
  **L2514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2515 EN**: Comment explains nearby logic, constraints, or intent: `8-bit floating-point multiply-add long long to single-precision (indexed, all top/bottom variants)`.
  **L2515 CN**: 注释解释附近代码的逻辑、约束或设计意图：`8-bit floating-point multiply-add long long to single-precision (indexed, all top/bottom variants)`。
- **L2516 EN**: Declares TableGen def record `SVFMLALLBB_LANE`.
  **L2516 CN**: 声明 TableGen def 记录 `SVFMLALLBB_LANE`。
- **L2517 EN**: Declares TableGen def record `SVFMLALLBT_LANE`.
  **L2517 CN**: 声明 TableGen def 记录 `SVFMLALLBT_LANE`。
- **L2518 EN**: Declares TableGen def record `SVFMLALLTB_LANE`.
  **L2518 CN**: 声明 TableGen def 记录 `SVFMLALLTB_LANE`。
- **L2519 EN**: Declares TableGen def record `SVFMLALLTT_LANE`.
  **L2519 CN**: 声明 TableGen def 记录 `SVFMLALLTT_LANE`。
- **L2520 EN**: Closes the current lexical scope or compound statement.
  **L2520 CN**: 结束当前词法作用域或复合语句块。

### Lines 2521-2538

````tablegen

let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2p2" in {
  def FMUL_SINGLE_X2 : SInst<"svmul[_single_{d}_x2]", "22d", "hfd", MergeNone, "aarch64_sve_fmul_single_x2", [IsStreaming], []>;
  def FMUL_SINGLE_X4 : SInst<"svmul[_single_{d}_x4]", "44d", "hfd", MergeNone, "aarch64_sve_fmul_single_x4", [IsStreaming], []>;

  def FMUL_X2 : SInst<"svmul[_{d}_x2]", "222", "hfd", MergeNone, "aarch64_sve_fmul_x2", [IsStreaming], []>;
  def FMUL_X4 : SInst<"svmul[_{d}_x4]", "444", "hfd", MergeNone, "aarch64_sve_fmul_x4", [IsStreaming], []>;
}

let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {
  def SVDOT_X2_SH : SInst<"svdot[_{d}_{2}]", "ddhh", "s",  MergeNone, "aarch64_sve_sdot_x2", [VerifyRuntimeMode], []>;
  def SVDOT_X2_UH : SInst<"svdot[_{d}_{2}]", "ddhh", "Us", MergeNone, "aarch64_sve_udot_x2", [VerifyRuntimeMode], []>;
  def SVDOT_N_X2_SH : SInst<"svdot[_n_{d}_{2}]", "ddhR", "s",  MergeNone, "aarch64_sve_sdot_x2", [VerifyRuntimeMode], []>;
  def SVDOT_N_X2_UH : SInst<"svdot[_n_{d}_{2}]", "ddhR", "Us", MergeNone, "aarch64_sve_udot_x2", [VerifyRuntimeMode], []>;

  def SVDOT_LANE_X2_SH : SInst<"svdot_lane[_{d}_{2}]", "ddhhi", "s",  MergeNone, "aarch64_sve_sdot_lane_x2", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
  def SVDOT_LANE_X2_UH : SInst<"svdot_lane[_{d}_{2}]", "ddhhi", "Us", MergeNone, "aarch64_sve_udot_lane_x2", [VerifyRuntimeMode], [ImmCheck<3, ImmCheck0_7>]>;
}
````
- **L2521 EN**: Blank line separating nearby declarations or logic blocks.
  **L2521 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2522 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2p2" in {`.
  **L2522 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode, SMETargetGuard = "sme2p2" in {`。
- **L2523 EN**: Declares TableGen def record `FMUL_SINGLE_X2`.
  **L2523 CN**: 声明 TableGen def 记录 `FMUL_SINGLE_X2`。
- **L2524 EN**: Declares TableGen def record `FMUL_SINGLE_X4`.
  **L2524 CN**: 声明 TableGen def 记录 `FMUL_SINGLE_X4`。
- **L2525 EN**: Blank line separating nearby declarations or logic blocks.
  **L2525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2526 EN**: Declares TableGen def record `FMUL_X2`.
  **L2526 CN**: 声明 TableGen def 记录 `FMUL_X2`。
- **L2527 EN**: Declares TableGen def record `FMUL_X4`.
  **L2527 CN**: 声明 TableGen def 记录 `FMUL_X4`。
- **L2528 EN**: Closes the current lexical scope or compound statement.
  **L2528 CN**: 结束当前词法作用域或复合语句块。
- **L2529 EN**: Blank line separating nearby declarations or logic blocks.
  **L2529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2530 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`.
  **L2530 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = "sve2p3|sme2p3", SMETargetGuard = "sve2p3|sme2p3" in {`。
- **L2531 EN**: Declares TableGen def record `SVDOT_X2_SH`.
  **L2531 CN**: 声明 TableGen def 记录 `SVDOT_X2_SH`。
- **L2532 EN**: Declares TableGen def record `SVDOT_X2_UH`.
  **L2532 CN**: 声明 TableGen def 记录 `SVDOT_X2_UH`。
- **L2533 EN**: Declares TableGen def record `SVDOT_N_X2_SH`.
  **L2533 CN**: 声明 TableGen def 记录 `SVDOT_N_X2_SH`。
- **L2534 EN**: Declares TableGen def record `SVDOT_N_X2_UH`.
  **L2534 CN**: 声明 TableGen def 记录 `SVDOT_N_X2_UH`。
- **L2535 EN**: Blank line separating nearby declarations or logic blocks.
  **L2535 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2536 EN**: Declares TableGen def record `SVDOT_LANE_X2_SH`.
  **L2536 CN**: 声明 TableGen def 记录 `SVDOT_LANE_X2_SH`。
- **L2537 EN**: Declares TableGen def record `SVDOT_LANE_X2_UH`.
  **L2537 CN**: 声明 TableGen def 记录 `SVDOT_LANE_X2_UH`。
- **L2538 EN**: Closes the current lexical scope or compound statement.
  **L2538 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Compiler configuration / 编译器配置**
  - **EN**: Holds low-level enums, constants, feature switches, and descriptive metadata used across Clang.
  - **CN**: 保存 Clang 各处复用的底层枚举、常量、特性开关与描述性元数据。
- **Target and language metadata / 目标与语言元数据**
  - **EN**: Models ABI, address-space, builtin, and target-feature information shared by many subsystems.
  - **CN**: 建模多个子系统共享的 ABI、地址空间、builtin 与目标特性信息。
- **Generated definition tables / 生成式定义表**
  - **EN**: Many Basic headers use `.td` or `.def` files to describe families of diagnostics, attributes, or builtins declaratively.
  - **CN**: 许多 Basic 头文件使用 `.td` 或 `.def` 文件以声明式方式描述诊断、属性或 builtin 家族。
- **TableGen declarative modeling / TableGen 声明式建模**
  - **EN**: Uses TableGen records, inheritance, and generators to describe families of compiler entities compactly.
  - **CN**: 使用 TableGen 记录、继承与生成器紧凑地描述一组编译器实体。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Indexing and tooling / 索引与工具支持**
  - **EN**: Exposes APIs used by source indexing, diagnostics retrieval, and IDE tooling.
  - **CN**: 暴露源码索引、诊断提取与 IDE 工具使用的 API。
- **AArch64 target support / AArch64 目标支持**
  - **EN**: Encodes AArch64-specific compiler metadata or builtin descriptions.
  - **CN**: 编码 AArch64 专用的编译器元数据或 builtin 描述。
- **Arm target support / Arm 目标支持**
  - **EN**: Encodes Arm-specific language extensions, intrinsics, or target metadata.
  - **CN**: 编码 Arm 专用语言扩展、intrinsic 或目标元数据。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `sat_type`
- **Functions or callables / 函数或可调用对象**: `vector`, `temporal`, `replicate`, `vectors`, `quadword`, `quadwords`, `structure`, `Prefetch`, `listconcat`, `DUP`, `bfmax`, `bfmaxnm`
- **TableGen records / TableGen 记录**: `SVLD1`, `SVLD1SB`, `SVLD1UB`, `SVLD1SH`, `SVLD1UH`, `SVLD1SW`, `SVLD1UW`, `SVLD1_VNUM`, `SVLD1SB_VNUM`, `SVLD1UB_VNUM`, `SVLD1SH_VNUM`, `SVLD1UH_VNUM`, `SVLD1SW_VNUM`, `SVLD1UW_VNUM`, `SVLD1_GATHER_BASES_U`, `SVLD1SB_GATHER_BASES_U`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
