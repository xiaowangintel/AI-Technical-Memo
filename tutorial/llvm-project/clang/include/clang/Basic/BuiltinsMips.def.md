# BuiltinsMips.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsMips.def`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: Mips Builtin function database *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsMips` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 905

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- BuiltinsMips.def - Mips Builtin function database --------*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the MIPS-specific builtin function database. Users of
// this file must define the BUILTIN macro to make use of this information.
//
//===----------------------------------------------------------------------===//

// The format of this database matches clang/Basic/Builtins.def.

// MIPS DSP Rev 1

// Add/subtract with optional saturation
BUILTIN(__builtin_mips_addu_qb, "V4ScV4ScV4Sc", "n")
BUILTIN(__builtin_mips_addu_s_qb, "V4ScV4ScV4Sc", "n")
BUILTIN(__builtin_mips_subu_qb, "V4ScV4ScV4Sc", "n")
BUILTIN(__builtin_mips_subu_s_qb, "V4ScV4ScV4Sc", "n")

BUILTIN(__builtin_mips_addq_ph, "V2sV2sV2s", "n")
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the MIPS-specific builtin function database. Users of`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the MIPS-specific builtin function database. Users of`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `this file must define the BUILTIN macro to make use of this information.`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`this file must define the BUILTIN macro to make use of this information.`。
- **L11 EN**: Separator comment used for visual grouping.
  **L11 CN**: 用于视觉分组的分隔注释。
- **L12 EN**: Banner comment marking a file or section boundary.
  **L12 CN**: 横幅注释，用于标记文件或章节边界。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `The format of this database matches clang/Basic/Builtins.def.`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The format of this database matches clang/Basic/Builtins.def.`。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Comment explains nearby logic, constraints, or intent: `MIPS DSP Rev 1`.
  **L16 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MIPS DSP Rev 1`。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `Add/subtract with optional saturation`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add/subtract with optional saturation`。
- **L19 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L19 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L20 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L20 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L21 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L21 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L22 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L22 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L24 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 25-48

````cpp
BUILTIN(__builtin_mips_addq_s_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_subq_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_subq_s_ph, "V2sV2sV2s", "n")

BUILTIN(__builtin_mips_madd, "LLiLLiii", "nc")
BUILTIN(__builtin_mips_maddu, "LLiLLiUiUi", "nc")
BUILTIN(__builtin_mips_msub, "LLiLLiii", "nc")
BUILTIN(__builtin_mips_msubu, "LLiLLiUiUi", "nc")

BUILTIN(__builtin_mips_addq_s_w, "iii", "n")
BUILTIN(__builtin_mips_subq_s_w, "iii", "n")

BUILTIN(__builtin_mips_addsc, "iii", "n")
BUILTIN(__builtin_mips_addwc, "iii", "n")

BUILTIN(__builtin_mips_modsub, "iii", "nc")

BUILTIN(__builtin_mips_raddu_w_qb, "iV4Sc", "nc")

BUILTIN(__builtin_mips_absq_s_ph, "V2sV2s", "n")
BUILTIN(__builtin_mips_absq_s_w, "ii", "n")

BUILTIN(__builtin_mips_precrq_qb_ph, "V4ScV2sV2s", "nc")
BUILTIN(__builtin_mips_precrqu_s_qb_ph, "V4ScV2sV2s", "n")
````
- **L25 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L25 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L26 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L26 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L27 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L27 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L29 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L29 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L30 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L30 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L31 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L31 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L32 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L32 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L34 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L34 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L35 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L35 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L37 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L38 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L38 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L40 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L40 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L42 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L44 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L44 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L45 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L45 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L47 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L48 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L48 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 49-72

````cpp
BUILTIN(__builtin_mips_precrq_ph_w, "V2sii", "nc")
BUILTIN(__builtin_mips_precrq_rs_ph_w, "V2sii", "n")
BUILTIN(__builtin_mips_preceq_w_phl, "iV2s", "nc")
BUILTIN(__builtin_mips_preceq_w_phr, "iV2s", "nc")
BUILTIN(__builtin_mips_precequ_ph_qbl, "V2sV4Sc", "nc")
BUILTIN(__builtin_mips_precequ_ph_qbr, "V2sV4Sc", "nc")
BUILTIN(__builtin_mips_precequ_ph_qbla, "V2sV4Sc", "nc")
BUILTIN(__builtin_mips_precequ_ph_qbra, "V2sV4Sc", "nc")
BUILTIN(__builtin_mips_preceu_ph_qbl, "V2sV4Sc", "nc")
BUILTIN(__builtin_mips_preceu_ph_qbr, "V2sV4Sc", "nc")
BUILTIN(__builtin_mips_preceu_ph_qbla, "V2sV4Sc", "nc")
BUILTIN(__builtin_mips_preceu_ph_qbra, "V2sV4Sc", "nc")

BUILTIN(__builtin_mips_shll_qb, "V4ScV4Sci", "n")
BUILTIN(__builtin_mips_shrl_qb, "V4ScV4Sci", "nc")
BUILTIN(__builtin_mips_shll_ph, "V2sV2si", "n")
BUILTIN(__builtin_mips_shll_s_ph, "V2sV2si", "n")
BUILTIN(__builtin_mips_shra_ph, "V2sV2si", "nc")
BUILTIN(__builtin_mips_shra_r_ph, "V2sV2si", "nc")
BUILTIN(__builtin_mips_shll_s_w, "iii", "n")
BUILTIN(__builtin_mips_shra_r_w, "iii", "nc")
BUILTIN(__builtin_mips_shilo, "LLiLLii", "nc")

BUILTIN(__builtin_mips_muleu_s_ph_qbl, "V2sV4ScV2s", "n")
````
- **L49 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L49 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L50 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L50 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L51 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L51 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L52 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L52 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L53 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L53 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L54 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L54 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L55 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L55 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L56 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L56 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L57 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L57 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L58 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L58 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L59 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L59 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L60 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L60 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L62 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L62 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L63 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L63 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L64 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L64 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L65 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L65 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L66 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L66 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L67 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L67 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L68 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L68 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L69 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L69 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L70 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L70 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L72 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 73-96

````cpp
BUILTIN(__builtin_mips_muleu_s_ph_qbr, "V2sV4ScV2s", "n")
BUILTIN(__builtin_mips_mulq_rs_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_muleq_s_w_phl, "iV2sV2s", "n")
BUILTIN(__builtin_mips_muleq_s_w_phr, "iV2sV2s", "n")
BUILTIN(__builtin_mips_mulsaq_s_w_ph, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_maq_s_w_phl, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_maq_s_w_phr, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_maq_sa_w_phl, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_maq_sa_w_phr, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_mult, "LLiii", "nc")
BUILTIN(__builtin_mips_multu, "LLiUiUi", "nc")

BUILTIN(__builtin_mips_dpau_h_qbl, "LLiLLiV4ScV4Sc", "nc")
BUILTIN(__builtin_mips_dpau_h_qbr, "LLiLLiV4ScV4Sc", "nc")
BUILTIN(__builtin_mips_dpsu_h_qbl, "LLiLLiV4ScV4Sc", "nc")
BUILTIN(__builtin_mips_dpsu_h_qbr, "LLiLLiV4ScV4Sc", "nc")
BUILTIN(__builtin_mips_dpaq_s_w_ph, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_dpsq_s_w_ph, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_dpaq_sa_l_w, "LLiLLiii", "n")
BUILTIN(__builtin_mips_dpsq_sa_l_w, "LLiLLiii", "n")

BUILTIN(__builtin_mips_cmpu_eq_qb, "vV4ScV4Sc", "n")
BUILTIN(__builtin_mips_cmpu_lt_qb, "vV4ScV4Sc", "n")
BUILTIN(__builtin_mips_cmpu_le_qb, "vV4ScV4Sc", "n")
````
- **L73 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L73 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L74 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L74 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L75 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L75 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L76 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L76 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L77 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L77 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L78 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L78 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L79 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L79 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L80 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L80 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L81 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L81 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L82 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L82 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L83 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L83 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L85 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L86 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L86 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L87 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L87 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L88 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L88 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L89 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L89 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L90 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L90 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L91 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L91 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L92 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L92 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L94 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L94 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L95 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L95 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L96 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L96 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 97-120

````cpp
BUILTIN(__builtin_mips_cmpgu_eq_qb, "iV4ScV4Sc", "n")
BUILTIN(__builtin_mips_cmpgu_lt_qb, "iV4ScV4Sc", "n")
BUILTIN(__builtin_mips_cmpgu_le_qb, "iV4ScV4Sc", "n")
BUILTIN(__builtin_mips_cmp_eq_ph, "vV2sV2s", "n")
BUILTIN(__builtin_mips_cmp_lt_ph, "vV2sV2s", "n")
BUILTIN(__builtin_mips_cmp_le_ph, "vV2sV2s", "n")

BUILTIN(__builtin_mips_extr_s_h, "iLLii", "n")
BUILTIN(__builtin_mips_extr_w, "iLLii", "n")
BUILTIN(__builtin_mips_extr_rs_w, "iLLii", "n")
BUILTIN(__builtin_mips_extr_r_w, "iLLii", "n")
BUILTIN(__builtin_mips_extp, "iLLii", "n")
BUILTIN(__builtin_mips_extpdp, "iLLii", "n")

BUILTIN(__builtin_mips_wrdsp, "viIi", "n")
BUILTIN(__builtin_mips_rddsp, "iIi", "n")
BUILTIN(__builtin_mips_insv, "iii", "n")
BUILTIN(__builtin_mips_bitrev, "ii", "nc")
BUILTIN(__builtin_mips_packrl_ph, "V2sV2sV2s", "nc")
BUILTIN(__builtin_mips_repl_qb, "V4Sci", "nc")
BUILTIN(__builtin_mips_repl_ph, "V2si", "nc")
BUILTIN(__builtin_mips_pick_qb, "V4ScV4ScV4Sc", "n")
BUILTIN(__builtin_mips_pick_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_mthlip, "LLiLLii", "n")
````
- **L97 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L97 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L98 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L98 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L99 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L99 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L100 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L100 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L101 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L101 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L102 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L102 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L104 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L104 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L105 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L105 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L106 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L106 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L107 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L107 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L108 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L108 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L109 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L109 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L111 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L111 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L112 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L112 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L113 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L113 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L114 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L114 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L115 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L115 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L116 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L116 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L117 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L117 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L118 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L118 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L119 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L119 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L120 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L120 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 121-144

````cpp
BUILTIN(__builtin_mips_bposge32, "i", "n")
BUILTIN(__builtin_mips_lbux, "iv*i", "n")
BUILTIN(__builtin_mips_lhx, "iv*i", "n")
BUILTIN(__builtin_mips_lwx, "iv*i", "n")

// MIPS DSP Rev 2

BUILTIN(__builtin_mips_absq_s_qb, "V4ScV4Sc", "n")

BUILTIN(__builtin_mips_addqh_ph, "V2sV2sV2s", "nc")
BUILTIN(__builtin_mips_addqh_r_ph, "V2sV2sV2s", "nc")
BUILTIN(__builtin_mips_addqh_w, "iii", "nc")
BUILTIN(__builtin_mips_addqh_r_w, "iii", "nc")

BUILTIN(__builtin_mips_addu_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_addu_s_ph, "V2sV2sV2s", "n")

BUILTIN(__builtin_mips_adduh_qb, "V4ScV4ScV4Sc", "nc")
BUILTIN(__builtin_mips_adduh_r_qb, "V4ScV4ScV4Sc", "nc")

BUILTIN(__builtin_mips_append, "iiiIi", "nc")
BUILTIN(__builtin_mips_balign, "iiiIi", "nc")

BUILTIN(__builtin_mips_cmpgdu_eq_qb, "iV4ScV4Sc", "n")
````
- **L121 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L121 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L122 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L122 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L123 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L123 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L124 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L124 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L126 EN**: Comment explains nearby logic, constraints, or intent: `MIPS DSP Rev 2`.
  **L126 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MIPS DSP Rev 2`。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L128 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L128 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L130 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L130 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L131 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L131 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L132 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L132 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L133 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L133 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L134 EN**: Blank line separating nearby declarations or logic blocks.
  **L134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L135 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L135 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L136 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L136 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L138 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L139 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L139 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L141 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L141 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L142 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L142 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L144 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L144 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 145-168

````cpp
BUILTIN(__builtin_mips_cmpgdu_lt_qb, "iV4ScV4Sc", "n")
BUILTIN(__builtin_mips_cmpgdu_le_qb, "iV4ScV4Sc", "n")

BUILTIN(__builtin_mips_dpa_w_ph, "LLiLLiV2sV2s", "nc")
BUILTIN(__builtin_mips_dps_w_ph, "LLiLLiV2sV2s", "nc")

BUILTIN(__builtin_mips_dpaqx_s_w_ph, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_dpaqx_sa_w_ph, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_dpax_w_ph, "LLiLLiV2sV2s", "nc")
BUILTIN(__builtin_mips_dpsx_w_ph, "LLiLLiV2sV2s", "nc")
BUILTIN(__builtin_mips_dpsqx_s_w_ph, "LLiLLiV2sV2s", "n")
BUILTIN(__builtin_mips_dpsqx_sa_w_ph, "LLiLLiV2sV2s", "n")

BUILTIN(__builtin_mips_mul_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_mul_s_ph, "V2sV2sV2s", "n")

BUILTIN(__builtin_mips_mulq_rs_w, "iii", "n")
BUILTIN(__builtin_mips_mulq_s_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_mulq_s_w, "iii", "n")
BUILTIN(__builtin_mips_mulsa_w_ph, "LLiLLiV2sV2s", "nc")

BUILTIN(__builtin_mips_precr_qb_ph, "V4ScV2sV2s", "n")
BUILTIN(__builtin_mips_precr_sra_ph_w, "V2siiIi", "nc")
BUILTIN(__builtin_mips_precr_sra_r_ph_w, "V2siiIi", "nc")
````
- **L145 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L145 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L146 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L146 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L148 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L149 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L149 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L151 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L151 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L152 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L152 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L153 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L153 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L154 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L154 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L155 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L155 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L156 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L156 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L158 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L158 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L159 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L159 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L160 EN**: Blank line separating nearby declarations or logic blocks.
  **L160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L161 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L161 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L162 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L162 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L163 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L163 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L164 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L164 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L166 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L166 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L167 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L167 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L168 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L168 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 169-192

````cpp

BUILTIN(__builtin_mips_prepend, "iiiIi", "nc")

BUILTIN(__builtin_mips_shra_qb, "V4ScV4Sci", "nc")
BUILTIN(__builtin_mips_shra_r_qb, "V4ScV4Sci", "nc")
BUILTIN(__builtin_mips_shrl_ph, "V2sV2si", "nc")

BUILTIN(__builtin_mips_subqh_ph, "V2sV2sV2s", "nc")
BUILTIN(__builtin_mips_subqh_r_ph, "V2sV2sV2s", "nc")
BUILTIN(__builtin_mips_subqh_w, "iii", "nc")
BUILTIN(__builtin_mips_subqh_r_w, "iii", "nc")

BUILTIN(__builtin_mips_subu_ph, "V2sV2sV2s", "n")
BUILTIN(__builtin_mips_subu_s_ph, "V2sV2sV2s", "n")

BUILTIN(__builtin_mips_subuh_qb, "V4ScV4ScV4Sc", "nc")
BUILTIN(__builtin_mips_subuh_r_qb, "V4ScV4ScV4Sc", "nc")

// MIPS MSA

BUILTIN(__builtin_msa_add_a_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_add_a_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_add_a_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_add_a_d, "V2SLLiV2SLLiV2SLLi", "nc")
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L170 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L170 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L172 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L173 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L173 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L174 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L174 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L175 EN**: Blank line separating nearby declarations or logic blocks.
  **L175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L176 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L176 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L177 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L177 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L178 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L178 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L179 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L179 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L181 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L181 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L182 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L182 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L183 EN**: Blank line separating nearby declarations or logic blocks.
  **L183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L184 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L184 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L185 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L185 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L186 EN**: Blank line separating nearby declarations or logic blocks.
  **L186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L187 EN**: Comment explains nearby logic, constraints, or intent: `MIPS MSA`.
  **L187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MIPS MSA`。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L189 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L189 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L190 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L190 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L191 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L191 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L192 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L192 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 193-216

````cpp

BUILTIN(__builtin_msa_adds_a_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_adds_a_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_adds_a_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_adds_a_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_adds_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_adds_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_adds_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_adds_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_adds_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_adds_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_adds_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_adds_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_addv_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_addv_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_addv_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_addv_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_addvi_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_addvi_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_addvi_w, "V4iV4iIUi", "nc")
````
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L194 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L194 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L195 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L195 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L196 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L196 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L197 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L197 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L198 EN**: Blank line separating nearby declarations or logic blocks.
  **L198 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L199 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L199 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L200 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L200 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L201 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L201 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L202 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L202 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L204 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L204 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L205 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L205 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L206 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L206 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L207 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L207 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L208 EN**: Blank line separating nearby declarations or logic blocks.
  **L208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L209 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L209 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L210 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L210 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L211 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L211 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L212 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L212 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L213 EN**: Blank line separating nearby declarations or logic blocks.
  **L213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L214 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L214 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L215 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L215 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L216 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L216 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 217-240

````cpp
BUILTIN(__builtin_msa_addvi_d, "V2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_and_v, "V16UcV16UcV16Uc", "nc")

BUILTIN(__builtin_msa_andi_b, "V16UcV16UcIUi", "nc")

BUILTIN(__builtin_msa_asub_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_asub_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_asub_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_asub_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_asub_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_asub_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_asub_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_asub_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_ave_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_ave_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_ave_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_ave_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_ave_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_ave_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_ave_u_w, "V4UiV4UiV4Ui", "nc")
````
- **L217 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L217 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L219 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L219 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L221 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L221 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L222 EN**: Blank line separating nearby declarations or logic blocks.
  **L222 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L223 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L223 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L224 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L224 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L225 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L225 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L226 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L226 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L228 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L228 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L229 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L229 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L230 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L230 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L231 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L231 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L233 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L234 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L234 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L235 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L235 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L236 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L236 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L238 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L238 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L239 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L239 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L240 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L240 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 241-264

````cpp
BUILTIN(__builtin_msa_ave_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_aver_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_aver_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_aver_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_aver_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_aver_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_aver_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_aver_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_aver_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_bclr_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_bclr_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_bclr_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_bclr_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_bclri_b, "V16UcV16UcIUi", "nc")
BUILTIN(__builtin_msa_bclri_h, "V8UsV8UsIUi", "nc")
BUILTIN(__builtin_msa_bclri_w, "V4UiV4UiIUi", "nc")
BUILTIN(__builtin_msa_bclri_d, "V2ULLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_binsl_b, "V16UcV16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_binsl_h, "V8UsV8UsV8UsV8Us", "nc")
````
- **L241 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L241 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L243 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L243 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L244 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L244 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L245 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L245 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L246 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L246 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L248 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L248 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L249 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L249 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L250 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L250 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L251 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L251 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L253 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L254 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L254 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L255 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L255 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L256 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L256 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L257 EN**: Blank line separating nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L258 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L258 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L259 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L259 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L260 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L260 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L261 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L261 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L263 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L263 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L264 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L264 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 265-288

````cpp
BUILTIN(__builtin_msa_binsl_w, "V4UiV4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_binsl_d, "V2ULLiV2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_binsli_b, "V16UcV16UcV16UcIUi", "nc")
BUILTIN(__builtin_msa_binsli_h, "V8UsV8UsV8UsIUi", "nc")
BUILTIN(__builtin_msa_binsli_w, "V4UiV4UiV4UiIUi", "nc")
BUILTIN(__builtin_msa_binsli_d, "V2ULLiV2ULLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_binsr_b, "V16UcV16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_binsr_h, "V8UsV8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_binsr_w, "V4UiV4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_binsr_d, "V2ULLiV2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_binsri_b, "V16UcV16UcV16UcIUi", "nc")
BUILTIN(__builtin_msa_binsri_h, "V8UsV8UsV8UsIUi", "nc")
BUILTIN(__builtin_msa_binsri_w, "V4UiV4UiV4UiIUi", "nc")
BUILTIN(__builtin_msa_binsri_d, "V2ULLiV2ULLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_bmnz_v, "V16UcV16UcV16UcV16Uc", "nc")

BUILTIN(__builtin_msa_bmnzi_b, "V16UcV16UcV16UcIUi", "nc")

BUILTIN(__builtin_msa_bmz_v, "V16UcV16UcV16UcV16Uc", "nc")

````
- **L265 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L265 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L266 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L266 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L267 EN**: Blank line separating nearby declarations or logic blocks.
  **L267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L268 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L268 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L269 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L269 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L270 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L270 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L271 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L271 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L273 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L274 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L274 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L275 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L275 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L276 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L276 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L278 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L278 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L279 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L279 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L280 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L280 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L281 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L281 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L283 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L283 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L285 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L285 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L287 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L287 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 289-312

````cpp
BUILTIN(__builtin_msa_bmzi_b, "V16UcV16UcV16UcIUi", "nc")

BUILTIN(__builtin_msa_bneg_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_bneg_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_bneg_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_bneg_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_bnegi_b, "V16UcV16UcIUi", "nc")
BUILTIN(__builtin_msa_bnegi_h, "V8UsV8UsIUi", "nc")
BUILTIN(__builtin_msa_bnegi_w, "V4UiV4UiIUi", "nc")
BUILTIN(__builtin_msa_bnegi_d, "V2ULLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_bnz_b, "iV16Uc", "nc")
BUILTIN(__builtin_msa_bnz_h, "iV8Us", "nc")
BUILTIN(__builtin_msa_bnz_w, "iV4Ui", "nc")
BUILTIN(__builtin_msa_bnz_d, "iV2ULLi", "nc")

BUILTIN(__builtin_msa_bnz_v, "iV16Uc", "nc")

BUILTIN(__builtin_msa_bsel_v, "V16UcV16UcV16UcV16Uc", "nc")

BUILTIN(__builtin_msa_bseli_b, "V16UcV16UcV16UcIUi", "nc")

BUILTIN(__builtin_msa_bset_b, "V16UcV16UcV16Uc", "nc")
````
- **L289 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L289 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L291 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L292 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L292 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L293 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L293 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L294 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L294 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L296 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L296 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L297 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L297 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L298 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L298 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L299 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L299 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L300 EN**: Blank line separating nearby declarations or logic blocks.
  **L300 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L301 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L301 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L302 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L302 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L303 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L303 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L304 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L304 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L306 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L306 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L308 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L308 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L310 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L312 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L312 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 313-336

````cpp
BUILTIN(__builtin_msa_bset_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_bset_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_bset_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_bseti_b, "V16UcV16UcIUi", "nc")
BUILTIN(__builtin_msa_bseti_h, "V8UsV8UsIUi", "nc")
BUILTIN(__builtin_msa_bseti_w, "V4UiV4UiIUi", "nc")
BUILTIN(__builtin_msa_bseti_d, "V2ULLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_bz_b, "iV16Uc", "nc")
BUILTIN(__builtin_msa_bz_h, "iV8Us", "nc")
BUILTIN(__builtin_msa_bz_w, "iV4Ui", "nc")
BUILTIN(__builtin_msa_bz_d, "iV2ULLi", "nc")

BUILTIN(__builtin_msa_bz_v, "iV16Uc", "nc")

BUILTIN(__builtin_msa_ceq_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_ceq_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_ceq_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_ceq_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_ceqi_b, "V16ScV16ScISi", "nc")
BUILTIN(__builtin_msa_ceqi_h, "V8SsV8SsISi", "nc")
BUILTIN(__builtin_msa_ceqi_w, "V4SiV4SiISi", "nc")
````
- **L313 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L313 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L314 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L314 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L315 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L315 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L317 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L317 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L318 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L318 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L319 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L319 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L320 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L320 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L322 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L322 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L323 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L323 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L324 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L324 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L325 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L325 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L327 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L328 EN**: Blank line separating nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L329 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L329 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L330 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L330 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L331 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L331 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L332 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L332 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L334 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L334 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L335 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L335 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L336 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L336 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 337-360

````cpp
BUILTIN(__builtin_msa_ceqi_d, "V2SLLiV2SLLiISi", "nc")

BUILTIN(__builtin_msa_cfcmsa, "iIi", "n")

BUILTIN(__builtin_msa_cle_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_cle_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_cle_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_cle_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_cle_u_b, "V16ScV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_cle_u_h, "V8SsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_cle_u_w, "V4SiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_cle_u_d, "V2SLLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_clei_s_b, "V16ScV16ScISi", "nc")
BUILTIN(__builtin_msa_clei_s_h, "V8SsV8SsISi", "nc")
BUILTIN(__builtin_msa_clei_s_w, "V4SiV4SiISi", "nc")
BUILTIN(__builtin_msa_clei_s_d, "V2SLLiV2SLLiISi", "nc")

BUILTIN(__builtin_msa_clei_u_b, "V16ScV16UcIUi", "nc")
BUILTIN(__builtin_msa_clei_u_h, "V8SsV8UsIUi", "nc")
BUILTIN(__builtin_msa_clei_u_w, "V4SiV4UiIUi", "nc")
BUILTIN(__builtin_msa_clei_u_d, "V2SLLiV2ULLiIUi", "nc")

````
- **L337 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L337 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L339 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L341 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L341 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L342 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L342 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L343 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L343 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L344 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L344 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L346 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L347 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L347 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L348 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L348 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L349 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L349 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L351 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L352 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L352 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L353 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L353 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L354 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L354 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L356 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L356 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L357 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L357 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L358 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L358 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L359 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L359 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 361-384

````cpp
BUILTIN(__builtin_msa_clt_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_clt_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_clt_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_clt_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_clt_u_b, "V16ScV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_clt_u_h, "V8SsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_clt_u_w, "V4SiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_clt_u_d, "V2SLLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_clti_s_b, "V16ScV16ScISi", "nc")
BUILTIN(__builtin_msa_clti_s_h, "V8SsV8SsISi", "nc")
BUILTIN(__builtin_msa_clti_s_w, "V4SiV4SiISi", "nc")
BUILTIN(__builtin_msa_clti_s_d, "V2SLLiV2SLLiISi", "nc")

BUILTIN(__builtin_msa_clti_u_b, "V16ScV16UcIUi", "nc")
BUILTIN(__builtin_msa_clti_u_h, "V8SsV8UsIUi", "nc")
BUILTIN(__builtin_msa_clti_u_w, "V4SiV4UiIUi", "nc")
BUILTIN(__builtin_msa_clti_u_d, "V2SLLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_copy_s_b, "iV16ScIUi", "nc")
BUILTIN(__builtin_msa_copy_s_h, "iV8SsIUi", "nc")
BUILTIN(__builtin_msa_copy_s_w, "iV4SiIUi", "nc")
BUILTIN(__builtin_msa_copy_s_d, "LLiV2SLLiIUi", "nc")
````
- **L361 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L361 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L362 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L362 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L363 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L363 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L364 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L364 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L366 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L366 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L367 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L367 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L368 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L368 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L369 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L369 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L371 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L371 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L372 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L372 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L373 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L373 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L374 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L374 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L376 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L376 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L377 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L377 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L378 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L378 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L379 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L379 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L381 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L381 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L382 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L382 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L383 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L383 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L384 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L384 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 385-408

````cpp

BUILTIN(__builtin_msa_copy_u_b, "iV16UcIUi", "nc")
BUILTIN(__builtin_msa_copy_u_h, "iV8UsIUi", "nc")
BUILTIN(__builtin_msa_copy_u_w, "iV4UiIUi", "nc")
BUILTIN(__builtin_msa_copy_u_d, "LLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_ctcmsa, "vIii", "n")

BUILTIN(__builtin_msa_div_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_div_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_div_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_div_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_div_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_div_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_div_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_div_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_dotp_s_h, "V8SsV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_dotp_s_w, "V4SiV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_dotp_s_d, "V2SLLiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_dotp_u_h, "V8UsV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_dotp_u_w, "V4UiV8UsV8Us", "nc")
````
- **L385 EN**: Blank line separating nearby declarations or logic blocks.
  **L385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L386 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L386 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L387 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L387 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L388 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L388 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L389 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L389 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L391 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L393 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L394 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L394 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L395 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L395 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L396 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L396 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L398 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L398 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L399 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L399 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L400 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L400 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L401 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L401 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L403 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L403 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L404 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L404 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L405 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L405 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L406 EN**: Blank line separating nearby declarations or logic blocks.
  **L406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L407 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L407 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L408 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L408 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 409-432

````cpp
BUILTIN(__builtin_msa_dotp_u_d, "V2ULLiV4UiV4Ui", "nc")

BUILTIN(__builtin_msa_dpadd_s_h, "V8SsV8SsV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_dpadd_s_w, "V4SiV4SiV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_dpadd_s_d, "V2SLLiV2SLLiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_dpadd_u_h, "V8UsV8UsV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_dpadd_u_w, "V4UiV4UiV8UsV8Us", "nc")
BUILTIN(__builtin_msa_dpadd_u_d, "V2ULLiV2ULLiV4UiV4Ui", "nc")

BUILTIN(__builtin_msa_dpsub_s_h, "V8SsV8SsV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_dpsub_s_w, "V4SiV4SiV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_dpsub_s_d, "V2SLLiV2SLLiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_dpsub_u_h, "V8UsV8UsV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_dpsub_u_w, "V4UiV4UiV8UsV8Us", "nc")
BUILTIN(__builtin_msa_dpsub_u_d, "V2ULLiV2ULLiV4UiV4Ui", "nc")

BUILTIN(__builtin_msa_fadd_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fadd_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fcaf_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcaf_d, "V2LLiV2dV2d", "nc")

````
- **L409 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L409 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L410 EN**: Blank line separating nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L411 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L411 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L412 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L412 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L413 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L413 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L415 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L415 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L416 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L416 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L417 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L417 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L419 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L419 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L420 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L420 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L421 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L421 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L423 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L423 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L424 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L424 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L425 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L425 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L427 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L427 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L428 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L428 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L430 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L431 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L431 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````cpp
BUILTIN(__builtin_msa_fceq_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fceq_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fclass_w, "V4iV4f", "nc")
BUILTIN(__builtin_msa_fclass_d, "V2LLiV2d", "nc")

BUILTIN(__builtin_msa_fcle_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcle_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fclt_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fclt_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fcne_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcne_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fcor_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcor_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fcueq_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcueq_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fcule_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcule_d, "V2LLiV2dV2d", "nc")

````
- **L433 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L433 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L434 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L434 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L435 EN**: Blank line separating nearby declarations or logic blocks.
  **L435 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L436 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L436 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L437 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L437 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L439 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L439 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L440 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L440 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L442 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L442 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L443 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L443 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L445 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L445 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L446 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L446 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L448 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L449 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L449 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L451 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L451 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L452 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L452 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L454 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L454 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L455 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L455 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 457-480

````cpp
BUILTIN(__builtin_msa_fcult_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcult_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fcun_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcun_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fcune_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fcune_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fdiv_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fdiv_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fexdo_h, "V8hV4fV4f", "nc")
BUILTIN(__builtin_msa_fexdo_w, "V4fV2dV2d", "nc")

BUILTIN(__builtin_msa_fexp2_w, "V4fV4fV4i", "nc")
BUILTIN(__builtin_msa_fexp2_d, "V2dV2dV2LLi", "nc")

BUILTIN(__builtin_msa_fexupl_w, "V4fV8h", "nc")
BUILTIN(__builtin_msa_fexupl_d, "V2dV4f", "nc")

BUILTIN(__builtin_msa_fexupr_w, "V4fV8h", "nc")
BUILTIN(__builtin_msa_fexupr_d, "V2dV4f", "nc")

````
- **L457 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L457 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L458 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L458 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L460 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L460 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L461 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L461 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L463 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L463 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L464 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L464 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L465 EN**: Blank line separating nearby declarations or logic blocks.
  **L465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L466 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L466 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L467 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L467 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L468 EN**: Blank line separating nearby declarations or logic blocks.
  **L468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L469 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L469 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L470 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L470 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L472 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L472 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L473 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L473 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L475 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L476 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L476 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L478 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L478 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L479 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L479 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 481-504

````cpp
BUILTIN(__builtin_msa_ffint_s_w, "V4fV4Si", "nc")
BUILTIN(__builtin_msa_ffint_s_d, "V2dV2SLLi", "nc")

BUILTIN(__builtin_msa_ffint_u_w, "V4fV4Ui", "nc")
BUILTIN(__builtin_msa_ffint_u_d, "V2dV2ULLi", "nc")

// ffql uses integers since long _Fract is not implemented
BUILTIN(__builtin_msa_ffql_w, "V4fV8Ss", "nc")
BUILTIN(__builtin_msa_ffql_d, "V2dV4Si", "nc")

// ffqr uses integers since long _Fract is not implemented
BUILTIN(__builtin_msa_ffqr_w, "V4fV8Ss", "nc")
BUILTIN(__builtin_msa_ffqr_d, "V2dV4Si", "nc")

BUILTIN(__builtin_msa_fill_b, "V16Sci", "nc")
BUILTIN(__builtin_msa_fill_h, "V8Ssi", "nc")
BUILTIN(__builtin_msa_fill_w, "V4Sii", "nc")
BUILTIN(__builtin_msa_fill_d, "V2SLLiLLi", "nc")

BUILTIN(__builtin_msa_flog2_w, "V4fV4f", "nc")
BUILTIN(__builtin_msa_flog2_d, "V2dV2d", "nc")

BUILTIN(__builtin_msa_fmadd_w, "V4fV4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fmadd_d, "V2dV2dV2dV2d", "nc")
````
- **L481 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L481 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L482 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L482 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L484 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L485 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L485 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L487 EN**: Comment explains nearby logic, constraints, or intent: `ffql uses integers since long _Fract is not implemented`.
  **L487 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ffql uses integers since long _Fract is not implemented`。
- **L488 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L488 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L489 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L489 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L490 EN**: Blank line separating nearby declarations or logic blocks.
  **L490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L491 EN**: Comment explains nearby logic, constraints, or intent: `ffqr uses integers since long _Fract is not implemented`.
  **L491 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ffqr uses integers since long _Fract is not implemented`。
- **L492 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L492 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L493 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L493 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L495 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L495 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L496 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L496 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L497 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L497 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L498 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L498 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L500 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L500 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L501 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L501 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L503 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L503 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L504 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L504 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 505-528

````cpp

BUILTIN(__builtin_msa_fmax_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fmax_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fmax_a_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fmax_a_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fmin_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fmin_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fmin_a_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fmin_a_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fmsub_w, "V4fV4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fmsub_d, "V2dV2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fmul_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fmul_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_frint_w, "V4fV4f", "nc")
BUILTIN(__builtin_msa_frint_d, "V2dV2d", "nc")

BUILTIN(__builtin_msa_frcp_w, "V4fV4f", "nc")
BUILTIN(__builtin_msa_frcp_d, "V2dV2d", "nc")
````
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L506 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L506 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L507 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L507 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L509 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L509 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L510 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L510 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L512 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L512 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L513 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L513 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L514 EN**: Blank line separating nearby declarations or logic blocks.
  **L514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L515 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L515 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L516 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L516 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L518 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L518 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L519 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L519 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L521 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L521 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L522 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L522 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L524 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L524 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L525 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L525 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L527 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L527 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L528 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L528 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 529-552

````cpp

BUILTIN(__builtin_msa_frsqrt_w, "V4fV4f", "nc")
BUILTIN(__builtin_msa_frsqrt_d, "V2dV2d", "nc")

BUILTIN(__builtin_msa_fsaf_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsaf_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fseq_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fseq_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsle_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsle_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fslt_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fslt_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsne_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsne_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsor_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsor_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsqrt_w, "V4fV4f", "nc")
BUILTIN(__builtin_msa_fsqrt_d, "V2dV2d", "nc")
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L530 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L531 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L531 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L533 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L533 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L534 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L534 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L536 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L536 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L537 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L537 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L538 EN**: Blank line separating nearby declarations or logic blocks.
  **L538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L539 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L539 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L540 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L540 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L541 EN**: Blank line separating nearby declarations or logic blocks.
  **L541 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L542 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L542 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L543 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L543 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L545 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L545 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L546 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L546 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L548 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L548 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L549 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L549 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L550 EN**: Blank line separating nearby declarations or logic blocks.
  **L550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L551 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L551 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L552 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L552 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 553-576

````cpp

BUILTIN(__builtin_msa_fsub_w, "V4fV4fV4f", "nc")
BUILTIN(__builtin_msa_fsub_d, "V2dV2dV2d", "nc")

BUILTIN(__builtin_msa_fsueq_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsueq_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsule_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsule_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsult_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsult_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsun_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsun_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_fsune_w, "V4iV4fV4f", "nc")
BUILTIN(__builtin_msa_fsune_d, "V2LLiV2dV2d", "nc")

BUILTIN(__builtin_msa_ftint_s_w, "V4SiV4f", "nc")
BUILTIN(__builtin_msa_ftint_s_d, "V2SLLiV2d", "nc")

BUILTIN(__builtin_msa_ftint_u_w, "V4UiV4f", "nc")
BUILTIN(__builtin_msa_ftint_u_d, "V2ULLiV2d", "nc")
````
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L554 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L554 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L555 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L555 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L557 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L557 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L558 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L558 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L560 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L560 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L561 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L561 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L562 EN**: Blank line separating nearby declarations or logic blocks.
  **L562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L563 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L563 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L564 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L564 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L566 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L566 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L567 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L567 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L569 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L569 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L570 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L570 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L572 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L573 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L573 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L575 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L576 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L576 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 577-600

````cpp

BUILTIN(__builtin_msa_ftq_h, "V4UiV4fV4f", "nc")
BUILTIN(__builtin_msa_ftq_w, "V2ULLiV2dV2d", "nc")

BUILTIN(__builtin_msa_ftrunc_s_w, "V4SiV4f", "nc")
BUILTIN(__builtin_msa_ftrunc_s_d, "V2SLLiV2d", "nc")

BUILTIN(__builtin_msa_ftrunc_u_w, "V4UiV4f", "nc")
BUILTIN(__builtin_msa_ftrunc_u_d, "V2ULLiV2d", "nc")

BUILTIN(__builtin_msa_hadd_s_h, "V8SsV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_hadd_s_w, "V4SiV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_hadd_s_d, "V2SLLiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_hadd_u_h, "V8UsV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_hadd_u_w, "V4UiV8UsV8Us", "nc")
BUILTIN(__builtin_msa_hadd_u_d, "V2ULLiV4UiV4Ui", "nc")

BUILTIN(__builtin_msa_hsub_s_h, "V8SsV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_hsub_s_w, "V4SiV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_hsub_s_d, "V2SLLiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_hsub_u_h, "V8UsV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_hsub_u_w, "V4UiV8UsV8Us", "nc")
````
- **L577 EN**: Blank line separating nearby declarations or logic blocks.
  **L577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L578 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L578 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L579 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L579 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L581 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L581 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L582 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L582 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L583 EN**: Blank line separating nearby declarations or logic blocks.
  **L583 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L584 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L584 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L585 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L585 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L586 EN**: Blank line separating nearby declarations or logic blocks.
  **L586 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L587 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L587 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L588 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L588 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L589 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L589 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L591 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L592 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L592 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L593 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L593 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L594 EN**: Blank line separating nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L595 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L595 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L596 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L596 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L597 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L597 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L599 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L599 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L600 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L600 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 601-624

````cpp
BUILTIN(__builtin_msa_hsub_u_d, "V2ULLiV4UiV4Ui", "nc")

BUILTIN(__builtin_msa_ilvev_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_ilvev_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_ilvev_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_ilvev_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_ilvl_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_ilvl_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_ilvl_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_ilvl_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_ilvod_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_ilvod_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_ilvod_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_ilvod_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_ilvr_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_ilvr_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_ilvr_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_ilvr_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_insert_b, "V16ScV16ScIUii", "nc")
BUILTIN(__builtin_msa_insert_h, "V8SsV8SsIUii", "nc")
````
- **L601 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L601 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L603 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L603 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L604 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L604 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L605 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L605 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L606 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L606 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L608 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L608 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L609 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L609 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L610 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L610 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L611 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L611 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L612 EN**: Blank line separating nearby declarations or logic blocks.
  **L612 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L613 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L613 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L614 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L614 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L615 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L615 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L616 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L616 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L618 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L619 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L619 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L620 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L620 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L621 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L621 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L623 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L623 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L624 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L624 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 625-648

````cpp
BUILTIN(__builtin_msa_insert_w, "V4SiV4SiIUii", "nc")
BUILTIN(__builtin_msa_insert_d, "V2SLLiV2SLLiIUiLLi", "nc")

BUILTIN(__builtin_msa_insve_b, "V16ScV16ScIUiV16Sc", "nc")
BUILTIN(__builtin_msa_insve_h, "V8SsV8SsIUiV8Ss", "nc")
BUILTIN(__builtin_msa_insve_w, "V4SiV4SiIUiV4Si", "nc")
BUILTIN(__builtin_msa_insve_d, "V2SLLiV2SLLiIUiV2SLLi", "nc")

BUILTIN(__builtin_msa_ld_b, "V16Scv*Ii", "nc")
BUILTIN(__builtin_msa_ld_h, "V8Ssv*Ii", "nc")
BUILTIN(__builtin_msa_ld_w, "V4Siv*Ii", "nc")
BUILTIN(__builtin_msa_ld_d, "V2SLLiv*Ii", "nc")

BUILTIN(__builtin_msa_ldr_d, "V2SLLiv*Ii", "nc")
BUILTIN(__builtin_msa_ldr_w, "V4Siv*Ii", "nc")

BUILTIN(__builtin_msa_ldi_b, "V16cIi", "nc")
BUILTIN(__builtin_msa_ldi_h, "V8sIi", "nc")
BUILTIN(__builtin_msa_ldi_w, "V4iIi", "nc")
BUILTIN(__builtin_msa_ldi_d, "V2LLiIi", "nc")

BUILTIN(__builtin_msa_madd_q_h, "V8SsV8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_madd_q_w, "V4SiV4SiV4SiV4Si", "nc")

````
- **L625 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L625 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L626 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L626 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L628 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L628 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L629 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L629 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L630 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L630 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L631 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L631 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L633 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L634 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L634 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L635 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L635 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L636 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L636 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L638 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L638 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L639 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L639 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L641 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L641 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L642 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L642 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L643 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L643 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L644 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L644 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L645 EN**: Blank line separating nearby declarations or logic blocks.
  **L645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L646 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L646 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L647 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L647 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L648 EN**: Blank line separating nearby declarations or logic blocks.
  **L648 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 649-672

````cpp
BUILTIN(__builtin_msa_maddr_q_h, "V8SsV8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_maddr_q_w, "V4SiV4SiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_maddv_b, "V16ScV16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_maddv_h, "V8SsV8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_maddv_w, "V4SiV4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_maddv_d, "V2SLLiV2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_max_a_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_max_a_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_max_a_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_max_a_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_max_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_max_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_max_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_max_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_max_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_max_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_max_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_max_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_maxi_s_b, "V16ScV16ScIi", "nc")
````
- **L649 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L649 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L650 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L650 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L651 EN**: Blank line separating nearby declarations or logic blocks.
  **L651 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L652 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L652 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L653 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L653 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L654 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L654 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L655 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L655 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L657 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L658 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L658 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L659 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L659 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L660 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L660 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L661 EN**: Blank line separating nearby declarations or logic blocks.
  **L661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L662 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L662 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L663 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L663 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L664 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L664 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L665 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L665 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L666 EN**: Blank line separating nearby declarations or logic blocks.
  **L666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L667 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L667 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L668 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L668 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L669 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L669 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L670 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L670 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L672 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L672 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 673-696

````cpp
BUILTIN(__builtin_msa_maxi_s_h, "V8SsV8SsIi", "nc")
BUILTIN(__builtin_msa_maxi_s_w, "V4SiV4SiIi", "nc")
BUILTIN(__builtin_msa_maxi_s_d, "V2SLLiV2SLLiIi", "nc")

BUILTIN(__builtin_msa_maxi_u_b, "V16UcV16UcIi", "nc")
BUILTIN(__builtin_msa_maxi_u_h, "V8UsV8UsIi", "nc")
BUILTIN(__builtin_msa_maxi_u_w, "V4UiV4UiIi", "nc")
BUILTIN(__builtin_msa_maxi_u_d, "V2ULLiV2ULLiIi", "nc")

BUILTIN(__builtin_msa_min_a_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_min_a_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_min_a_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_min_a_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_min_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_min_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_min_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_min_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_min_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_min_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_min_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_min_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

````
- **L673 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L673 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L674 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L674 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L675 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L675 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L677 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L677 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L678 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L678 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L679 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L679 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L680 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L680 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L681 EN**: Blank line separating nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L682 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L682 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L683 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L683 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L684 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L684 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L685 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L685 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L687 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L687 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L688 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L688 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L689 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L689 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L690 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L690 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L692 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L693 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L693 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L694 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L694 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L695 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L695 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L696 EN**: Blank line separating nearby declarations or logic blocks.
  **L696 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 697-720

````cpp
BUILTIN(__builtin_msa_mini_s_b, "V16ScV16ScIi", "nc")
BUILTIN(__builtin_msa_mini_s_h, "V8SsV8SsIi", "nc")
BUILTIN(__builtin_msa_mini_s_w, "V4SiV4SiIi", "nc")
BUILTIN(__builtin_msa_mini_s_d, "V2SLLiV2SLLiIi", "nc")

BUILTIN(__builtin_msa_mini_u_b, "V16UcV16UcIi", "nc")
BUILTIN(__builtin_msa_mini_u_h, "V8UsV8UsIi", "nc")
BUILTIN(__builtin_msa_mini_u_w, "V4UiV4UiIi", "nc")
BUILTIN(__builtin_msa_mini_u_d, "V2ULLiV2ULLiIi", "nc")

BUILTIN(__builtin_msa_mod_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_mod_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_mod_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_mod_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_mod_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_mod_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_mod_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_mod_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_move_v, "V16ScV16Sc", "nc")

BUILTIN(__builtin_msa_msub_q_h, "V8SsV8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_msub_q_w, "V4SiV4SiV4SiV4Si", "nc")
````
- **L697 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L697 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L698 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L698 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L699 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L699 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L700 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L700 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L702 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L703 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L703 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L704 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L704 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L705 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L705 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L706 EN**: Blank line separating nearby declarations or logic blocks.
  **L706 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L707 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L707 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L708 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L708 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L709 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L709 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L710 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L710 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L712 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L712 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L713 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L713 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L714 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L714 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L715 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L715 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L716 EN**: Blank line separating nearby declarations or logic blocks.
  **L716 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L717 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L717 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L719 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L719 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L720 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L720 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 721-744

````cpp

BUILTIN(__builtin_msa_msubr_q_h, "V8SsV8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_msubr_q_w, "V4SiV4SiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_msubv_b, "V16ScV16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_msubv_h, "V8SsV8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_msubv_w, "V4SiV4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_msubv_d, "V2SLLiV2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_mul_q_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_mul_q_w, "V4SiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_mulr_q_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_mulr_q_w, "V4SiV4SiV4Si", "nc")

BUILTIN(__builtin_msa_mulv_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_mulv_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_mulv_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_mulv_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_nloc_b, "V16ScV16Sc", "nc")
BUILTIN(__builtin_msa_nloc_h, "V8SsV8Ss", "nc")
BUILTIN(__builtin_msa_nloc_w, "V4SiV4Si", "nc")
BUILTIN(__builtin_msa_nloc_d, "V2SLLiV2SLLi", "nc")
````
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L722 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L723 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L723 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L725 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L725 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L726 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L726 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L727 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L727 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L728 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L728 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L730 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L730 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L731 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L731 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L732 EN**: Blank line separating nearby declarations or logic blocks.
  **L732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L733 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L733 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L734 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L734 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L736 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L736 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L737 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L737 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L738 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L738 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L739 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L739 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L741 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L742 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L742 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L743 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L743 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L744 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L744 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 745-768

````cpp

BUILTIN(__builtin_msa_nlzc_b, "V16ScV16Sc", "nc")
BUILTIN(__builtin_msa_nlzc_h, "V8SsV8Ss", "nc")
BUILTIN(__builtin_msa_nlzc_w, "V4SiV4Si", "nc")
BUILTIN(__builtin_msa_nlzc_d, "V2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_nor_v, "V16UcV16UcV16Uc", "nc")

BUILTIN(__builtin_msa_nori_b, "V16UcV16cIUi", "nc")

BUILTIN(__builtin_msa_or_v, "V16UcV16UcV16Uc", "nc")

BUILTIN(__builtin_msa_ori_b, "V16UcV16UcIUi", "nc")

BUILTIN(__builtin_msa_pckev_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_pckev_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_pckev_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_pckev_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_pckod_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_pckod_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_pckod_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_pckod_d, "V2LLiV2LLiV2LLi", "nc")

````
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L746 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L746 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L747 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L747 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L748 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L748 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L749 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L749 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L750 EN**: Blank line separating nearby declarations or logic blocks.
  **L750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L751 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L751 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L752 EN**: Blank line separating nearby declarations or logic blocks.
  **L752 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L753 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L753 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L755 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L755 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L757 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L757 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L759 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L760 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L760 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L761 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L761 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L762 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L762 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L763 EN**: Blank line separating nearby declarations or logic blocks.
  **L763 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L764 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L764 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L765 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L765 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L766 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L766 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L767 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L767 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L768 EN**: Blank line separating nearby declarations or logic blocks.
  **L768 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 769-792

````cpp
BUILTIN(__builtin_msa_pcnt_b, "V16ScV16Sc", "nc")
BUILTIN(__builtin_msa_pcnt_h, "V8SsV8Ss", "nc")
BUILTIN(__builtin_msa_pcnt_w, "V4SiV4Si", "nc")
BUILTIN(__builtin_msa_pcnt_d, "V2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_sat_s_b, "V16ScV16ScIUi", "nc")
BUILTIN(__builtin_msa_sat_s_h, "V8SsV8SsIUi", "nc")
BUILTIN(__builtin_msa_sat_s_w, "V4SiV4SiIUi", "nc")
BUILTIN(__builtin_msa_sat_s_d, "V2SLLiV2SLLiIUi", "nc")

BUILTIN(__builtin_msa_sat_u_b, "V16UcV16UcIUi", "nc")
BUILTIN(__builtin_msa_sat_u_h, "V8UsV8UsIUi", "nc")
BUILTIN(__builtin_msa_sat_u_w, "V4UiV4UiIUi", "nc")
BUILTIN(__builtin_msa_sat_u_d, "V2ULLiV2ULLiIUi", "nc")

BUILTIN(__builtin_msa_shf_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_shf_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_shf_w, "V4iV4iIUi", "nc")

BUILTIN(__builtin_msa_sld_b, "V16cV16cV16cUi", "nc")
BUILTIN(__builtin_msa_sld_h, "V8sV8sV8sUi", "nc")
BUILTIN(__builtin_msa_sld_w, "V4iV4iV4iUi", "nc")
BUILTIN(__builtin_msa_sld_d, "V2LLiV2LLiV2LLiUi", "nc")

````
- **L769 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L769 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L770 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L770 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L771 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L771 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L772 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L772 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L773 EN**: Blank line separating nearby declarations or logic blocks.
  **L773 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L774 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L774 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L775 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L775 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L776 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L776 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L777 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L777 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L778 EN**: Blank line separating nearby declarations or logic blocks.
  **L778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L779 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L779 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L780 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L780 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L781 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L781 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L782 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L782 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L784 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L784 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L785 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L785 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L786 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L786 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L787 EN**: Blank line separating nearby declarations or logic blocks.
  **L787 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L788 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L788 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L789 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L789 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L790 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L790 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L791 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L791 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L792 EN**: Blank line separating nearby declarations or logic blocks.
  **L792 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 793-816

````cpp
BUILTIN(__builtin_msa_sldi_b, "V16cV16cV16cIUi", "nc")
BUILTIN(__builtin_msa_sldi_h, "V8sV8sV8sIUi", "nc")
BUILTIN(__builtin_msa_sldi_w, "V4iV4iV4iIUi", "nc")
BUILTIN(__builtin_msa_sldi_d, "V2LLiV2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_sll_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_sll_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_sll_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_sll_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_slli_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_slli_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_slli_w, "V4iV4iIUi", "nc")
BUILTIN(__builtin_msa_slli_d, "V2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_splat_b, "V16cV16cUi", "nc")
BUILTIN(__builtin_msa_splat_h, "V8sV8sUi", "nc")
BUILTIN(__builtin_msa_splat_w, "V4iV4iUi", "nc")
BUILTIN(__builtin_msa_splat_d, "V2LLiV2LLiUi", "nc")

BUILTIN(__builtin_msa_splati_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_splati_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_splati_w, "V4iV4iIUi", "nc")
BUILTIN(__builtin_msa_splati_d, "V2LLiV2LLiIUi", "nc")
````
- **L793 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L793 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L794 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L794 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L795 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L795 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L796 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L796 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L798 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L798 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L799 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L799 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L800 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L800 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L801 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L801 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L803 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L803 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L804 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L804 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L805 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L805 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L806 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L806 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L808 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L809 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L809 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L810 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L810 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L811 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L811 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L813 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L814 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L814 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L815 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L815 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L816 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L816 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 817-840

````cpp

BUILTIN(__builtin_msa_sra_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_sra_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_sra_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_sra_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_srai_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_srai_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_srai_w, "V4iV4iIUi", "nc")
BUILTIN(__builtin_msa_srai_d, "V2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_srar_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_srar_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_srar_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_srar_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_srari_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_srari_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_srari_w, "V4iV4iIUi", "nc")
BUILTIN(__builtin_msa_srari_d, "V2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_srl_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_srl_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_srl_w, "V4iV4iV4i", "nc")
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L818 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L818 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L819 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L819 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L820 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L820 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L821 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L821 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L823 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L824 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L824 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L825 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L825 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L826 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L826 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L827 EN**: Blank line separating nearby declarations or logic blocks.
  **L827 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L828 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L828 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L829 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L829 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L830 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L830 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L831 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L831 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L833 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L833 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L834 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L834 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L835 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L835 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L836 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L836 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L837 EN**: Blank line separating nearby declarations or logic blocks.
  **L837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L838 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L838 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L839 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L839 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L840 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L840 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 841-864

````cpp
BUILTIN(__builtin_msa_srl_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_srli_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_srli_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_srli_w, "V4iV4iIUi", "nc")
BUILTIN(__builtin_msa_srli_d, "V2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_srlr_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_srlr_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_srlr_w, "V4iV4iV4i", "nc")
BUILTIN(__builtin_msa_srlr_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_srlri_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_srlri_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_srlri_w, "V4iV4iIUi", "nc")
BUILTIN(__builtin_msa_srlri_d, "V2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_st_b, "vV16Scv*Ii", "nc")
BUILTIN(__builtin_msa_st_h, "vV8Ssv*Ii", "nc")
BUILTIN(__builtin_msa_st_w, "vV4Siv*Ii", "nc")
BUILTIN(__builtin_msa_st_d, "vV2SLLiv*Ii", "nc")

BUILTIN(__builtin_msa_str_d, "vV2SLLiv*Ii", "nc")
BUILTIN(__builtin_msa_str_w, "vV4Siv*Ii", "nc")
````
- **L841 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L841 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L843 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L843 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L844 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L844 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L845 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L845 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L846 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L846 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L848 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L848 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L849 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L849 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L850 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L850 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L851 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L851 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L853 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L854 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L854 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L855 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L855 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L856 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L856 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L857 EN**: Blank line separating nearby declarations or logic blocks.
  **L857 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L858 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L858 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L859 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L859 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L860 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L860 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L861 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L861 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L863 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L864 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L864 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 865-888

````cpp

BUILTIN(__builtin_msa_subs_s_b, "V16ScV16ScV16Sc", "nc")
BUILTIN(__builtin_msa_subs_s_h, "V8SsV8SsV8Ss", "nc")
BUILTIN(__builtin_msa_subs_s_w, "V4SiV4SiV4Si", "nc")
BUILTIN(__builtin_msa_subs_s_d, "V2SLLiV2SLLiV2SLLi", "nc")

BUILTIN(__builtin_msa_subs_u_b, "V16UcV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_subs_u_h, "V8UsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_subs_u_w, "V4UiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_subs_u_d, "V2ULLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_subsus_u_b, "V16UcV16UcV16Sc", "nc")
BUILTIN(__builtin_msa_subsus_u_h, "V8UsV8UsV8Ss", "nc")
BUILTIN(__builtin_msa_subsus_u_w, "V4UiV4UiV4Si", "nc")
BUILTIN(__builtin_msa_subsus_u_d, "V2ULLiV2ULLiV2SLLi", "nc")

BUILTIN(__builtin_msa_subsuu_s_b, "V16ScV16UcV16Uc", "nc")
BUILTIN(__builtin_msa_subsuu_s_h, "V8SsV8UsV8Us", "nc")
BUILTIN(__builtin_msa_subsuu_s_w, "V4SiV4UiV4Ui", "nc")
BUILTIN(__builtin_msa_subsuu_s_d, "V2SLLiV2ULLiV2ULLi", "nc")

BUILTIN(__builtin_msa_subv_b, "V16cV16cV16c", "nc")
BUILTIN(__builtin_msa_subv_h, "V8sV8sV8s", "nc")
BUILTIN(__builtin_msa_subv_w, "V4iV4iV4i", "nc")
````
- **L865 EN**: Blank line separating nearby declarations or logic blocks.
  **L865 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L866 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L866 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L867 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L867 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L868 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L868 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L869 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L869 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L870 EN**: Blank line separating nearby declarations or logic blocks.
  **L870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L871 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L871 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L872 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L872 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L873 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L873 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L874 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L874 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L875 EN**: Blank line separating nearby declarations or logic blocks.
  **L875 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L876 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L876 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L877 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L877 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L878 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L878 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L879 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L879 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L881 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L881 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L882 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L882 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L883 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L883 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L884 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L884 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L885 EN**: Blank line separating nearby declarations or logic blocks.
  **L885 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L886 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L886 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L887 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L887 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L888 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L888 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 889-905

````cpp
BUILTIN(__builtin_msa_subv_d, "V2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_subvi_b, "V16cV16cIUi", "nc")
BUILTIN(__builtin_msa_subvi_h, "V8sV8sIUi", "nc")
BUILTIN(__builtin_msa_subvi_w, "V4iV4iIUi", "nc")
BUILTIN(__builtin_msa_subvi_d, "V2LLiV2LLiIUi", "nc")

BUILTIN(__builtin_msa_vshf_b, "V16cV16cV16cV16c", "nc")
BUILTIN(__builtin_msa_vshf_h, "V8sV8sV8sV8s", "nc")
BUILTIN(__builtin_msa_vshf_w, "V4iV4iV4iV4i", "nc")
BUILTIN(__builtin_msa_vshf_d, "V2LLiV2LLiV2LLiV2LLi", "nc")

BUILTIN(__builtin_msa_xor_v, "V16cV16cV16c", "nc")

BUILTIN(__builtin_msa_xori_b, "V16cV16cIUi", "nc")

#undef BUILTIN
````
- **L889 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L889 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L890 EN**: Blank line separating nearby declarations or logic blocks.
  **L890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L891 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L891 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L892 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L892 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L893 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L893 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L894 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L894 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L895 EN**: Blank line separating nearby declarations or logic blocks.
  **L895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L896 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L896 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L897 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L897 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L898 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L898 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L899 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L899 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L900 EN**: Blank line separating nearby declarations or logic blocks.
  **L900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L901 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L901 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L902 EN**: Blank line separating nearby declarations or logic blocks.
  **L902 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L903 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L903 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L904 EN**: Blank line separating nearby declarations or logic blocks.
  **L904 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L905 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef BUILTIN`.
  **L905 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef BUILTIN`。

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
- **Macro-driven definition lists / 宏驱动定义列表**
  - **EN**: Provides reusable entry lists that different includers expand under different macro definitions.
  - **CN**: 提供可复用的条目列表，不同包含者可在不同宏定义下对其展开。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `BUILTIN`
- **TableGen records / TableGen 记录**: No prominent TableGen record names detected. / 未检测到明显的 TableGen 记录名称。
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
