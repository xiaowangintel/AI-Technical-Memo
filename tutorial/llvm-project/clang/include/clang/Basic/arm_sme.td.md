# arm_sme.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/arm_sme.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: ARM SME compiler interface.
- **Purpose (CN)**: 声明与 `arm_sme` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1034

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//===--- arm_sme.td - ARM SME compiler interface ------------------------===//
//
//  Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
//  See https://llvm.org/LICENSE.txt for license information.
//  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file defines the TableGen definitions from which the ARM SME header
//  file will be generated.  See:
//
//      https://developer.arm.com/architectures/system-architectures/software-standards/acle
//
//===----------------------------------------------------------------------===//

include "arm_sve_sme_incl.td"

let SVETargetGuard = InvalidMode in {

////////////////////////////////////////////////////////////////////////////////
// Loads

multiclass ZALoad<string n_suffix, string t, string i_prefix, list<ImmCheck> ch> {
  def NAME # _H : MInst<"svld1_hor_" # n_suffix, "vimPQ", t,
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the TableGen definitions from which the ARM SME header`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the TableGen definitions from which the ARM SME header`。
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
- **L18 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SVETargetGuard = InvalidMode in {`.
  **L18 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SVETargetGuard = InvalidMode in {`。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L20 EN**: Separator comment used for visual grouping.
  **L20 CN**: 用于视觉分组的分隔注释。
- **L21 EN**: Comment explains nearby logic, constraints, or intent: `Loads`.
  **L21 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Loads`。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Declares TableGen multiclass record `ZALoad`.
  **L23 CN**: 声明 TableGen multiclass 记录 `ZALoad`。
- **L24 EN**: Declares TableGen def record `NAME # _H`.
  **L24 CN**: 声明 TableGen def 记录 `NAME # _H`。

### Lines 25-48

````tablegen
                        [IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],
                        MemEltTyDefault, i_prefix # "_horiz", ch>;

  def NAME # _H_VNUM : MInst<"svld1_hor_vnum_" # n_suffix, "vimPQl", t,
                             [IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],
                             MemEltTyDefault, i_prefix # "_horiz", ch>;

  def NAME # _V : MInst<"svld1_ver_" # n_suffix, "vimPQ", t,
                        [IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],
                        MemEltTyDefault, i_prefix # "_vert", ch>;

  def NAME # _V_VNUM : MInst<"svld1_ver_vnum_" # n_suffix, "vimPQl", t,
                             [IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],
                             MemEltTyDefault, i_prefix # "_vert", ch>;
}

defm SVLD1_ZA8 : ZALoad<"za8", "c", "aarch64_sme_ld1b", [ImmCheck<0, ImmCheck0_0>]>;
defm SVLD1_ZA16 : ZALoad<"za16", "s", "aarch64_sme_ld1h", [ImmCheck<0, ImmCheck0_1>]>;
defm SVLD1_ZA32 : ZALoad<"za32", "i", "aarch64_sme_ld1w", [ImmCheck<0, ImmCheck0_3>]>;
defm SVLD1_ZA64 : ZALoad<"za64", "l", "aarch64_sme_ld1d", [ImmCheck<0, ImmCheck0_7>]>;
defm SVLD1_ZA128 : ZALoad<"za128", "q", "aarch64_sme_ld1q", [ImmCheck<0, ImmCheck0_15>]>;

def SVLDR_VNUM_ZA : MInst<"svldr_vnum_za", "vmQl", "",
                          [IsOverloadNone, IsStreamingCompatible, IsInOutZA],
````
- **L25 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`.
  **L25 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`。
- **L26 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_horiz", ch>;`.
  **L26 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_horiz", ch>;`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Declares TableGen def record `NAME # _H_VNUM`.
  **L28 CN**: 声明 TableGen def 记录 `NAME # _H_VNUM`。
- **L29 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`.
  **L29 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`。
- **L30 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_horiz", ch>;`.
  **L30 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_horiz", ch>;`。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L32 EN**: Declares TableGen def record `NAME # _V`.
  **L32 CN**: 声明 TableGen def 记录 `NAME # _V`。
- **L33 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`.
  **L33 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`。
- **L34 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_vert", ch>;`.
  **L34 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_vert", ch>;`。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L36 EN**: Declares TableGen def record `NAME # _V_VNUM`.
  **L36 CN**: 声明 TableGen def 记录 `NAME # _V_VNUM`。
- **L37 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`.
  **L37 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsLoad, IsOverloadNone, IsStreaming, IsInOutZA],`。
- **L38 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_vert", ch>;`.
  **L38 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_vert", ch>;`。
- **L39 EN**: Closes the current lexical scope or compound statement.
  **L39 CN**: 结束当前词法作用域或复合语句块。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L41 EN**: Declares TableGen defm record `SVLD1_ZA8`.
  **L41 CN**: 声明 TableGen defm 记录 `SVLD1_ZA8`。
- **L42 EN**: Declares TableGen defm record `SVLD1_ZA16`.
  **L42 CN**: 声明 TableGen defm 记录 `SVLD1_ZA16`。
- **L43 EN**: Declares TableGen defm record `SVLD1_ZA32`.
  **L43 CN**: 声明 TableGen defm 记录 `SVLD1_ZA32`。
- **L44 EN**: Declares TableGen defm record `SVLD1_ZA64`.
  **L44 CN**: 声明 TableGen defm 记录 `SVLD1_ZA64`。
- **L45 EN**: Declares TableGen defm record `SVLD1_ZA128`.
  **L45 CN**: 声明 TableGen defm 记录 `SVLD1_ZA128`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Declares TableGen def record `SVLDR_VNUM_ZA`.
  **L47 CN**: 声明 TableGen def 记录 `SVLDR_VNUM_ZA`。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsOverloadNone, IsStreamingCompatible, IsInOutZA],`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsOverloadNone, IsStreamingCompatible, IsInOutZA],`。

### Lines 49-72

````tablegen
                          MemEltTyDefault, "aarch64_sme_ldr">;

def SVLDR_ZA : MInst<"svldr_za", "vmQ", "",
                          [IsOverloadNone, IsStreamingCompatible, IsInOutZA],
                          MemEltTyDefault, "aarch64_sme_ldr", []>;

////////////////////////////////////////////////////////////////////////////////
// Stores

multiclass ZAStore<string n_suffix, string t, string i_prefix, list<ImmCheck> ch> {
  def NAME # _H : MInst<"svst1_hor_" # n_suffix, "vimP%", t,
                        [IsStore, IsOverloadNone, IsStreaming, IsInZA],
                        MemEltTyDefault, i_prefix # "_horiz", ch>;

  def NAME # _H_VNUM : MInst<"svst1_hor_vnum_" # n_suffix, "vimP%l", t,
                             [IsStore, IsOverloadNone, IsStreaming, IsInZA],
                             MemEltTyDefault, i_prefix # "_horiz", ch>;

  def NAME # _V : MInst<"svst1_ver_" # n_suffix, "vimP%", t,
                        [IsStore, IsOverloadNone, IsStreaming, IsInZA],
                        MemEltTyDefault, i_prefix # "_vert", ch>;

  def NAME # _V_VNUM : MInst<"svst1_ver_vnum_" # n_suffix, "vimP%l", t,
                             [IsStore, IsOverloadNone, IsStreaming, IsInZA],
````
- **L49 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, "aarch64_sme_ldr">;`.
  **L49 CN**: 添加一条独立语句或声明：`MemEltTyDefault, "aarch64_sme_ldr">;`。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L51 EN**: Declares TableGen def record `SVLDR_ZA`.
  **L51 CN**: 声明 TableGen def 记录 `SVLDR_ZA`。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsOverloadNone, IsStreamingCompatible, IsInOutZA],`.
  **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsOverloadNone, IsStreamingCompatible, IsInOutZA],`。
- **L53 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, "aarch64_sme_ldr", []>;`.
  **L53 CN**: 添加一条独立语句或声明：`MemEltTyDefault, "aarch64_sme_ldr", []>;`。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L55 EN**: Separator comment used for visual grouping.
  **L55 CN**: 用于视觉分组的分隔注释。
- **L56 EN**: Comment explains nearby logic, constraints, or intent: `Stores`.
  **L56 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Stores`。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L58 EN**: Declares TableGen multiclass record `ZAStore`.
  **L58 CN**: 声明 TableGen multiclass 记录 `ZAStore`。
- **L59 EN**: Declares TableGen def record `NAME # _H`.
  **L59 CN**: 声明 TableGen def 记录 `NAME # _H`。
- **L60 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStore, IsOverloadNone, IsStreaming, IsInZA],`.
  **L60 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStore, IsOverloadNone, IsStreaming, IsInZA],`。
- **L61 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_horiz", ch>;`.
  **L61 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_horiz", ch>;`。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Declares TableGen def record `NAME # _H_VNUM`.
  **L63 CN**: 声明 TableGen def 记录 `NAME # _H_VNUM`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStore, IsOverloadNone, IsStreaming, IsInZA],`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStore, IsOverloadNone, IsStreaming, IsInZA],`。
- **L65 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_horiz", ch>;`.
  **L65 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_horiz", ch>;`。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Declares TableGen def record `NAME # _V`.
  **L67 CN**: 声明 TableGen def 记录 `NAME # _V`。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStore, IsOverloadNone, IsStreaming, IsInZA],`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStore, IsOverloadNone, IsStreaming, IsInZA],`。
- **L69 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_vert", ch>;`.
  **L69 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_vert", ch>;`。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L71 EN**: Declares TableGen def record `NAME # _V_VNUM`.
  **L71 CN**: 声明 TableGen def 记录 `NAME # _V_VNUM`。
- **L72 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStore, IsOverloadNone, IsStreaming, IsInZA],`.
  **L72 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStore, IsOverloadNone, IsStreaming, IsInZA],`。

### Lines 73-96

````tablegen
                             MemEltTyDefault, i_prefix # "_vert", ch>;
}

defm SVST1_ZA8 : ZAStore<"za8", "c", "aarch64_sme_st1b", [ImmCheck<0, ImmCheck0_0>]>;
defm SVST1_ZA16 : ZAStore<"za16", "s", "aarch64_sme_st1h", [ImmCheck<0, ImmCheck0_1>]>;
defm SVST1_ZA32 : ZAStore<"za32", "i", "aarch64_sme_st1w", [ImmCheck<0, ImmCheck0_3>]>;
defm SVST1_ZA64 : ZAStore<"za64", "l", "aarch64_sme_st1d", [ImmCheck<0, ImmCheck0_7>]>;
defm SVST1_ZA128 : ZAStore<"za128", "q", "aarch64_sme_st1q", [ImmCheck<0, ImmCheck0_15>]>;

def SVSTR_VNUM_ZA : MInst<"svstr_vnum_za", "vm%l", "",
                          [IsOverloadNone, IsStreamingCompatible, IsInZA],
                          MemEltTyDefault, "aarch64_sme_str">;

def SVSTR_ZA : MInst<"svstr_za", "vm%", "",
                      [IsOverloadNone, IsStreamingCompatible, IsInZA],
                      MemEltTyDefault, "aarch64_sme_str", []>;

////////////////////////////////////////////////////////////////////////////////
// Read horizontal/vertical ZA slices

multiclass ZARead<string n_suffix, string t, string i_prefix, list<ImmCheck> ch> {
  def NAME # _H : SInst<"svread_hor_" # n_suffix # "[_{d}]", "ddPim", t,
                        MergeOp1, i_prefix # "_horiz",
                        [IsReadZA, IsStreaming, IsInZA], ch>;
````
- **L73 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, i_prefix # "_vert", ch>;`.
  **L73 CN**: 添加一条独立语句或声明：`MemEltTyDefault, i_prefix # "_vert", ch>;`。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L76 EN**: Declares TableGen defm record `SVST1_ZA8`.
  **L76 CN**: 声明 TableGen defm 记录 `SVST1_ZA8`。
- **L77 EN**: Declares TableGen defm record `SVST1_ZA16`.
  **L77 CN**: 声明 TableGen defm 记录 `SVST1_ZA16`。
- **L78 EN**: Declares TableGen defm record `SVST1_ZA32`.
  **L78 CN**: 声明 TableGen defm 记录 `SVST1_ZA32`。
- **L79 EN**: Declares TableGen defm record `SVST1_ZA64`.
  **L79 CN**: 声明 TableGen defm 记录 `SVST1_ZA64`。
- **L80 EN**: Declares TableGen defm record `SVST1_ZA128`.
  **L80 CN**: 声明 TableGen defm 记录 `SVST1_ZA128`。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L82 EN**: Declares TableGen def record `SVSTR_VNUM_ZA`.
  **L82 CN**: 声明 TableGen def 记录 `SVSTR_VNUM_ZA`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsOverloadNone, IsStreamingCompatible, IsInZA],`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsOverloadNone, IsStreamingCompatible, IsInZA],`。
- **L84 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, "aarch64_sme_str">;`.
  **L84 CN**: 添加一条独立语句或声明：`MemEltTyDefault, "aarch64_sme_str">;`。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L86 EN**: Declares TableGen def record `SVSTR_ZA`.
  **L86 CN**: 声明 TableGen def 记录 `SVSTR_ZA`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsOverloadNone, IsStreamingCompatible, IsInZA],`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsOverloadNone, IsStreamingCompatible, IsInZA],`。
- **L88 EN**: Adds a standalone statement or declaration: `MemEltTyDefault, "aarch64_sme_str", []>;`.
  **L88 CN**: 添加一条独立语句或声明：`MemEltTyDefault, "aarch64_sme_str", []>;`。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L90 EN**: Separator comment used for visual grouping.
  **L90 CN**: 用于视觉分组的分隔注释。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `Read horizontal/vertical ZA slices`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Read horizontal/vertical ZA slices`。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L93 EN**: Declares TableGen multiclass record `ZARead`.
  **L93 CN**: 声明 TableGen multiclass 记录 `ZARead`。
- **L94 EN**: Declares TableGen def record `NAME # _H`.
  **L94 CN**: 声明 TableGen def 记录 `NAME # _H`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, i_prefix # "_horiz",`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, i_prefix # "_horiz",`。
- **L96 EN**: Adds a standalone statement or declaration: `[IsReadZA, IsStreaming, IsInZA], ch>;`.
  **L96 CN**: 添加一条独立语句或声明：`[IsReadZA, IsStreaming, IsInZA], ch>;`。

### Lines 97-120

````tablegen

  def NAME # _V : SInst<"svread_ver_" # n_suffix # "[_{d}]", "ddPim", t,
                        MergeOp1, i_prefix # "_vert",
                        [IsReadZA, IsStreaming, IsInZA], ch>;
}

defm SVREAD_ZA8 : ZARead<"za8", "cUcm", "aarch64_sme_read", [ImmCheck<2, ImmCheck0_0>]>;
defm SVREAD_ZA16 : ZARead<"za16", "sUshb", "aarch64_sme_read", [ImmCheck<2, ImmCheck0_1>]>;
defm SVREAD_ZA32 : ZARead<"za32", "iUif", "aarch64_sme_read", [ImmCheck<2, ImmCheck0_3>]>;
defm SVREAD_ZA64 : ZARead<"za64", "lUld", "aarch64_sme_read", [ImmCheck<2, ImmCheck0_7>]>;
defm SVREAD_ZA128 : ZARead<"za128", "csilUcUsUiUlmhbfd", "aarch64_sme_readq", [ImmCheck<2, ImmCheck0_15>]>;

////////////////////////////////////////////////////////////////////////////////
// Write horizontal/vertical ZA slices

multiclass ZAWrite<string n_suffix, string t, string i_prefix, list<ImmCheck> ch> {
  def NAME # _H : SInst<"svwrite_hor_" # n_suffix # "[_{d}]", "vimPd", t,
                        MergeOp1, i_prefix # "_horiz",
                        [IsWriteZA, IsStreaming, IsInOutZA], ch>;

  def NAME # _V : SInst<"svwrite_ver_" # n_suffix # "[_{d}]", "vimPd", t,
                        MergeOp1, i_prefix # "_vert",
                        [IsWriteZA, IsStreaming, IsInOutZA], ch>;
}
````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L98 EN**: Declares TableGen def record `NAME # _V`.
  **L98 CN**: 声明 TableGen def 记录 `NAME # _V`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, i_prefix # "_vert",`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, i_prefix # "_vert",`。
- **L100 EN**: Adds a standalone statement or declaration: `[IsReadZA, IsStreaming, IsInZA], ch>;`.
  **L100 CN**: 添加一条独立语句或声明：`[IsReadZA, IsStreaming, IsInZA], ch>;`。
- **L101 EN**: Closes the current lexical scope or compound statement.
  **L101 CN**: 结束当前词法作用域或复合语句块。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L103 EN**: Declares TableGen defm record `SVREAD_ZA8`.
  **L103 CN**: 声明 TableGen defm 记录 `SVREAD_ZA8`。
- **L104 EN**: Declares TableGen defm record `SVREAD_ZA16`.
  **L104 CN**: 声明 TableGen defm 记录 `SVREAD_ZA16`。
- **L105 EN**: Declares TableGen defm record `SVREAD_ZA32`.
  **L105 CN**: 声明 TableGen defm 记录 `SVREAD_ZA32`。
- **L106 EN**: Declares TableGen defm record `SVREAD_ZA64`.
  **L106 CN**: 声明 TableGen defm 记录 `SVREAD_ZA64`。
- **L107 EN**: Declares TableGen defm record `SVREAD_ZA128`.
  **L107 CN**: 声明 TableGen defm 记录 `SVREAD_ZA128`。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L109 EN**: Separator comment used for visual grouping.
  **L109 CN**: 用于视觉分组的分隔注释。
- **L110 EN**: Comment explains nearby logic, constraints, or intent: `Write horizontal/vertical ZA slices`.
  **L110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Write horizontal/vertical ZA slices`。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L112 EN**: Declares TableGen multiclass record `ZAWrite`.
  **L112 CN**: 声明 TableGen multiclass 记录 `ZAWrite`。
- **L113 EN**: Declares TableGen def record `NAME # _H`.
  **L113 CN**: 声明 TableGen def 记录 `NAME # _H`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, i_prefix # "_horiz",`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, i_prefix # "_horiz",`。
- **L115 EN**: Adds a standalone statement or declaration: `[IsWriteZA, IsStreaming, IsInOutZA], ch>;`.
  **L115 CN**: 添加一条独立语句或声明：`[IsWriteZA, IsStreaming, IsInOutZA], ch>;`。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L117 EN**: Declares TableGen def record `NAME # _V`.
  **L117 CN**: 声明 TableGen def 记录 `NAME # _V`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, i_prefix # "_vert",`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, i_prefix # "_vert",`。
- **L119 EN**: Adds a standalone statement or declaration: `[IsWriteZA, IsStreaming, IsInOutZA], ch>;`.
  **L119 CN**: 添加一条独立语句或声明：`[IsWriteZA, IsStreaming, IsInOutZA], ch>;`。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-144

````tablegen

defm SVWRITE_ZA8 : ZAWrite<"za8", "cUcm", "aarch64_sme_write", [ImmCheck<0, ImmCheck0_0>]>;
defm SVWRITE_ZA16 : ZAWrite<"za16", "sUshb", "aarch64_sme_write", [ImmCheck<0, ImmCheck0_1>]>;
defm SVWRITE_ZA32 : ZAWrite<"za32", "iUif", "aarch64_sme_write", [ImmCheck<0, ImmCheck0_3>]>;
defm SVWRITE_ZA64 : ZAWrite<"za64", "lUld", "aarch64_sme_write", [ImmCheck<0, ImmCheck0_7>]>;
defm SVWRITE_ZA128 : ZAWrite<"za128", "csilUcUsUiUlmhbfd", "aarch64_sme_writeq", [ImmCheck<0, ImmCheck0_15>]>;

////////////////////////////////////////////////////////////////////////////////
// SME - Zero

def SVZERO_MASK_ZA : SInst<"svzero_mask_za", "vi", "", MergeNone, "aarch64_sme_zero",
                           [IsOverloadNone, IsStreamingCompatible, IsInOutZA],
                           [ImmCheck<0, ImmCheck0_255>]>;
def SVZERO_ZA      : SInst<"svzero_za", "vv", "", MergeNone, "aarch64_sme_zero",
                           [IsOverloadNone, IsStreamingCompatible, IsOutZA]>;

let SMETargetGuard = "sme2p1" in {
  def SVZERO_ZA64_VG1x2 : SInst<"svzero_za64_vg1x2", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg1x2",
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
  def SVZERO_ZA64_VG1x4 : SInst<"svzero_za64_vg1x4", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg1x4",
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
  def SVZERO_ZA64_VG2x1 : SInst<"svzero_za64_vg2x1", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg2x1",
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
  def SVZERO_ZA64_VG2x2 : SInst<"svzero_za64_vg2x2", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg2x2",
````
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L122 EN**: Declares TableGen defm record `SVWRITE_ZA8`.
  **L122 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA8`。
- **L123 EN**: Declares TableGen defm record `SVWRITE_ZA16`.
  **L123 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA16`。
- **L124 EN**: Declares TableGen defm record `SVWRITE_ZA32`.
  **L124 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA32`。
- **L125 EN**: Declares TableGen defm record `SVWRITE_ZA64`.
  **L125 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA64`。
- **L126 EN**: Declares TableGen defm record `SVWRITE_ZA128`.
  **L126 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA128`。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L128 EN**: Separator comment used for visual grouping.
  **L128 CN**: 用于视觉分组的分隔注释。
- **L129 EN**: Comment explains nearby logic, constraints, or intent: `SME - Zero`.
  **L129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME - Zero`。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L131 EN**: Declares TableGen def record `SVZERO_MASK_ZA`.
  **L131 CN**: 声明 TableGen def 记录 `SVZERO_MASK_ZA`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsOverloadNone, IsStreamingCompatible, IsInOutZA],`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsOverloadNone, IsStreamingCompatible, IsInOutZA],`。
- **L133 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_255>]>;`.
  **L133 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_255>]>;`。
- **L134 EN**: Declares TableGen def record `SVZERO_ZA`.
  **L134 CN**: 声明 TableGen def 记录 `SVZERO_ZA`。
- **L135 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreamingCompatible, IsOutZA]>;`.
  **L135 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreamingCompatible, IsOutZA]>;`。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L137 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2p1" in {`.
  **L137 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2p1" in {`。
- **L138 EN**: Declares TableGen def record `SVZERO_ZA64_VG1x2`.
  **L138 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG1x2`。
- **L139 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L139 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L140 EN**: Declares TableGen def record `SVZERO_ZA64_VG1x4`.
  **L140 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG1x4`。
- **L141 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L141 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L142 EN**: Declares TableGen def record `SVZERO_ZA64_VG2x1`.
  **L142 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG2x1`。
- **L143 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L143 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L144 EN**: Declares TableGen def record `SVZERO_ZA64_VG2x2`.
  **L144 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG2x2`。

### Lines 145-168

````tablegen
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
  def SVZERO_ZA64_VG2x4 : SInst<"svzero_za64_vg2x4", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg2x4",
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
  def SVZERO_ZA64_VG4x1 : SInst<"svzero_za64_vg4x1", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg4x1",
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
  def SVZERO_ZA64_VG4x2 : SInst<"svzero_za64_vg4x2", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg4x2",
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
  def SVZERO_ZA64_VG4x4 : SInst<"svzero_za64_vg4x4", "vm", "", MergeNone, "aarch64_sme_zero_za64_vg4x4",
                            [IsOverloadNone, IsStreaming, IsInOutZA]>;
}

////////////////////////////////////////////////////////////////////////////////
// SME - Counting elements in a streaming vector

def SVCNTSB : SInst<"svcntsb", "nv", "", MergeNone, "", [IsOverloadNone, IsStreamingCompatible]>;
def SVCNTSH : SInst<"svcntsh", "nv", "", MergeNone, "", [IsOverloadNone, IsStreamingCompatible]>;
def SVCNTSW : SInst<"svcntsw", "nv", "", MergeNone, "", [IsOverloadNone, IsStreamingCompatible]>;
def SVCNTSD : SInst<"svcntsd", "nv", "", MergeNone, "aarch64_sme_cntsd", [IsOverloadNone, IsStreamingCompatible]>;

////////////////////////////////////////////////////////////////////////////////
// SME - ADDHA/ADDVA

multiclass ZAAdd<string n_suffix> {
  def NAME # _ZA32: SInst<"sv" # n_suffix # "_za32[_{d}]", "viPPd", "iUi", MergeOp1,
````
- **L145 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L145 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L146 EN**: Declares TableGen def record `SVZERO_ZA64_VG2x4`.
  **L146 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG2x4`。
- **L147 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L147 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L148 EN**: Declares TableGen def record `SVZERO_ZA64_VG4x1`.
  **L148 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG4x1`。
- **L149 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L149 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L150 EN**: Declares TableGen def record `SVZERO_ZA64_VG4x2`.
  **L150 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG4x2`。
- **L151 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L151 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L152 EN**: Declares TableGen def record `SVZERO_ZA64_VG4x4`.
  **L152 CN**: 声明 TableGen def 记录 `SVZERO_ZA64_VG4x4`。
- **L153 EN**: Adds a standalone statement or declaration: `[IsOverloadNone, IsStreaming, IsInOutZA]>;`.
  **L153 CN**: 添加一条独立语句或声明：`[IsOverloadNone, IsStreaming, IsInOutZA]>;`。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L156 EN**: Separator comment used for visual grouping.
  **L156 CN**: 用于视觉分组的分隔注释。
- **L157 EN**: Comment explains nearby logic, constraints, or intent: `SME - Counting elements in a streaming vector`.
  **L157 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME - Counting elements in a streaming vector`。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L159 EN**: Declares TableGen def record `SVCNTSB`.
  **L159 CN**: 声明 TableGen def 记录 `SVCNTSB`。
- **L160 EN**: Declares TableGen def record `SVCNTSH`.
  **L160 CN**: 声明 TableGen def 记录 `SVCNTSH`。
- **L161 EN**: Declares TableGen def record `SVCNTSW`.
  **L161 CN**: 声明 TableGen def 记录 `SVCNTSW`。
- **L162 EN**: Declares TableGen def record `SVCNTSD`.
  **L162 CN**: 声明 TableGen def 记录 `SVCNTSD`。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L164 EN**: Separator comment used for visual grouping.
  **L164 CN**: 用于视觉分组的分隔注释。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `SME - ADDHA/ADDVA`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME - ADDHA/ADDVA`。
- **L166 EN**: Blank line separating nearby declarations or logic blocks.
  **L166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L167 EN**: Declares TableGen multiclass record `ZAAdd`.
  **L167 CN**: 声明 TableGen multiclass 记录 `ZAAdd`。
- **L168 EN**: Declares TableGen def record `NAME # _ZA32`.
  **L168 CN**: 声明 TableGen def 记录 `NAME # _ZA32`。

### Lines 169-192

````tablegen
                    "aarch64_sme_" # n_suffix, [IsStreaming, IsInOutZA],
                    [ImmCheck<0, ImmCheck0_3>]>;

  let SMETargetGuard = "sme-i16i64" in {
    def NAME # _ZA64: SInst<"sv" # n_suffix # "_za64[_{d}]", "viPPd", "lUl", MergeOp1,
                     "aarch64_sme_" # n_suffix, [IsStreaming, IsInOutZA],
                     [ImmCheck<0, ImmCheck0_7>]>;
  }
}

defm SVADDHA : ZAAdd<"addha">;
defm SVADDVA : ZAAdd<"addva">;

////////////////////////////////////////////////////////////////////////////////
// SME - SMOPA, SMOPS, UMOPA, UMOPS

multiclass ZAIntOuterProd<string n_suffix1, string n_suffix2> {
  def NAME # _ZA32_B: SInst<"sv" # n_suffix2 # "_za32[_{d}]",
                            "viPPdd", !cond(!eq(n_suffix1, "s") : "", true: "U") # "c",
                            MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",
                            [IsStreaming, IsInOutZA],
                            [ImmCheck<0, ImmCheck0_3>]>;

  let SMETargetGuard = "sme-i16i64" in {
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"aarch64_sme_" # n_suffix, [IsStreaming, IsInOutZA],`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`"aarch64_sme_" # n_suffix, [IsStreaming, IsInOutZA],`。
- **L170 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_3>]>;`.
  **L170 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_3>]>;`。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-i16i64" in {`.
  **L172 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-i16i64" in {`。
- **L173 EN**: Declares TableGen def record `NAME # _ZA64`.
  **L173 CN**: 声明 TableGen def 记录 `NAME # _ZA64`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"aarch64_sme_" # n_suffix, [IsStreaming, IsInOutZA],`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`"aarch64_sme_" # n_suffix, [IsStreaming, IsInOutZA],`。
- **L175 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_7>]>;`.
  **L175 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_7>]>;`。
- **L176 EN**: Closes the current lexical scope or compound statement.
  **L176 CN**: 结束当前词法作用域或复合语句块。
- **L177 EN**: Closes the current lexical scope or compound statement.
  **L177 CN**: 结束当前词法作用域或复合语句块。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L179 EN**: Declares TableGen defm record `SVADDHA`.
  **L179 CN**: 声明 TableGen defm 记录 `SVADDHA`。
- **L180 EN**: Declares TableGen defm record `SVADDVA`.
  **L180 CN**: 声明 TableGen defm 记录 `SVADDVA`。
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L182 EN**: Separator comment used for visual grouping.
  **L182 CN**: 用于视觉分组的分隔注释。
- **L183 EN**: Comment explains nearby logic, constraints, or intent: `SME - SMOPA, SMOPS, UMOPA, UMOPS`.
  **L183 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME - SMOPA, SMOPS, UMOPA, UMOPS`。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L185 EN**: Declares TableGen multiclass record `ZAIntOuterProd`.
  **L185 CN**: 声明 TableGen multiclass 记录 `ZAIntOuterProd`。
- **L186 EN**: Declares TableGen def record `NAME # _ZA32_B`.
  **L186 CN**: 声明 TableGen def 记录 `NAME # _ZA32_B`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"viPPdd", !cond(!eq(n_suffix1, "s") : "", true: "U") # "c",`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`"viPPdd", !cond(!eq(n_suffix1, "s") : "", true: "U") # "c",`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L190 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_3>]>;`.
  **L190 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_3>]>;`。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L192 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-i16i64" in {`.
  **L192 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-i16i64" in {`。

### Lines 193-216

````tablegen
    def NAME # _ZA64_H: SInst<"sv" # n_suffix2 # "_za64[_{d}]",
                              "viPPdd", !cond(!eq(n_suffix1, "s") : "", true: "U") # "s",
                              MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",
                              [IsStreaming, IsInOutZA],
                              [ImmCheck<0, ImmCheck0_7>]>;
  }
}

defm SVSMOPA : ZAIntOuterProd<"s", "mopa">;
defm SVSMOPS : ZAIntOuterProd<"s", "mops">;
defm SVUMOPA : ZAIntOuterProd<"u", "mopa">;
defm SVUMOPS : ZAIntOuterProd<"u", "mops">;

////////////////////////////////////////////////////////////////////////////////
// SME - SUMOPA, SUMOPS, USMOPA, USMOPS

multiclass ZAIntOuterProdMixedSigns<string n_suffix1, string n_suffix2> {
  def NAME # _ZA32_B: SInst<"sv" # n_suffix1 # n_suffix2 # "_za32[_{d}]",
                            "viPPd" # !cond(!eq(n_suffix1, "su") : "u", true: "x"),
                            !cond(!eq(n_suffix1, "su") : "", true: "U") # "c",
                            MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",
                            [IsStreaming, IsInOutZA],
                            [ImmCheck<0, ImmCheck0_3>]>;

````
- **L193 EN**: Declares TableGen def record `NAME # _ZA64_H`.
  **L193 CN**: 声明 TableGen def 记录 `NAME # _ZA64_H`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"viPPdd", !cond(!eq(n_suffix1, "s") : "", true: "U") # "s",`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`"viPPdd", !cond(!eq(n_suffix1, "s") : "", true: "U") # "s",`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L197 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_7>]>;`.
  **L197 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_7>]>;`。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L201 EN**: Declares TableGen defm record `SVSMOPA`.
  **L201 CN**: 声明 TableGen defm 记录 `SVSMOPA`。
- **L202 EN**: Declares TableGen defm record `SVSMOPS`.
  **L202 CN**: 声明 TableGen defm 记录 `SVSMOPS`。
- **L203 EN**: Declares TableGen defm record `SVUMOPA`.
  **L203 CN**: 声明 TableGen defm 记录 `SVUMOPA`。
- **L204 EN**: Declares TableGen defm record `SVUMOPS`.
  **L204 CN**: 声明 TableGen defm 记录 `SVUMOPS`。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Separator comment used for visual grouping.
  **L206 CN**: 用于视觉分组的分隔注释。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `SME - SUMOPA, SUMOPS, USMOPA, USMOPS`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME - SUMOPA, SUMOPS, USMOPA, USMOPS`。
- **L208 EN**: Blank line separating nearby declarations or logic blocks.
  **L208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L209 EN**: Declares TableGen multiclass record `ZAIntOuterProdMixedSigns`.
  **L209 CN**: 声明 TableGen multiclass 记录 `ZAIntOuterProdMixedSigns`。
- **L210 EN**: Declares TableGen def record `NAME # _ZA32_B`.
  **L210 CN**: 声明 TableGen def 记录 `NAME # _ZA32_B`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"viPPd" # !cond(!eq(n_suffix1, "su") : "u", true: "x"),`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`"viPPd" # !cond(!eq(n_suffix1, "su") : "u", true: "x"),`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!cond(!eq(n_suffix1, "su") : "", true: "U") # "c",`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`!cond(!eq(n_suffix1, "su") : "", true: "U") # "c",`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L215 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_3>]>;`.
  **L215 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_3>]>;`。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 217-240

````tablegen
  let SMETargetGuard = "sme-i16i64" in {
    def NAME # _ZA64_H: SInst<"sv" # n_suffix1 # n_suffix2 # "_za64[_{d}]",
                              "viPPd" # !cond(!eq(n_suffix1, "su") : "u", true: "x"),
                              !cond(!eq(n_suffix1, "su") : "", true: "U") # "s",
                              MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",
                              [IsStreaming, IsInOutZA],
                              [ImmCheck<0, ImmCheck0_7>]>;
  }
}

defm SVSUMOPA : ZAIntOuterProdMixedSigns<"su", "mopa">;
defm SVSUMOPS : ZAIntOuterProdMixedSigns<"su", "mops">;
defm SVUSMOPA : ZAIntOuterProdMixedSigns<"us", "mopa">;
defm SVUSMOPS : ZAIntOuterProdMixedSigns<"us", "mops">;

////////////////////////////////////////////////////////////////////////////////
// SME - FMOPA, FMOPS

multiclass ZAFPOuterProd<string n_suffix> {
  def NAME # _ZA32_B: SInst<"sv" # n_suffix # "_za32[_{d}]", "viPPdd", "h",
                            MergeOp1, "aarch64_sme_" # n_suffix # "_wide",
                            [IsStreaming, IsInOutZA],
                            [ImmCheck<0, ImmCheck0_3>]>;

````
- **L217 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-i16i64" in {`.
  **L217 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-i16i64" in {`。
- **L218 EN**: Declares TableGen def record `NAME # _ZA64_H`.
  **L218 CN**: 声明 TableGen def 记录 `NAME # _ZA64_H`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"viPPd" # !cond(!eq(n_suffix1, "su") : "u", true: "x"),`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`"viPPd" # !cond(!eq(n_suffix1, "su") : "u", true: "x"),`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!cond(!eq(n_suffix1, "su") : "", true: "U") # "s",`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`!cond(!eq(n_suffix1, "su") : "", true: "U") # "s",`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix1 # n_suffix2 # "_wide",`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L223 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_7>]>;`.
  **L223 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_7>]>;`。
- **L224 EN**: Closes the current lexical scope or compound statement.
  **L224 CN**: 结束当前词法作用域或复合语句块。
- **L225 EN**: Closes the current lexical scope or compound statement.
  **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L227 EN**: Declares TableGen defm record `SVSUMOPA`.
  **L227 CN**: 声明 TableGen defm 记录 `SVSUMOPA`。
- **L228 EN**: Declares TableGen defm record `SVSUMOPS`.
  **L228 CN**: 声明 TableGen defm 记录 `SVSUMOPS`。
- **L229 EN**: Declares TableGen defm record `SVUSMOPA`.
  **L229 CN**: 声明 TableGen defm 记录 `SVUSMOPA`。
- **L230 EN**: Declares TableGen defm record `SVUSMOPS`.
  **L230 CN**: 声明 TableGen defm 记录 `SVUSMOPS`。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L232 EN**: Separator comment used for visual grouping.
  **L232 CN**: 用于视觉分组的分隔注释。
- **L233 EN**: Comment explains nearby logic, constraints, or intent: `SME - FMOPA, FMOPS`.
  **L233 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME - FMOPA, FMOPS`。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L235 EN**: Declares TableGen multiclass record `ZAFPOuterProd`.
  **L235 CN**: 声明 TableGen multiclass 记录 `ZAFPOuterProd`。
- **L236 EN**: Declares TableGen def record `NAME # _ZA32_B`.
  **L236 CN**: 声明 TableGen def 记录 `NAME # _ZA32_B`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix # "_wide",`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix # "_wide",`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L239 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_3>]>;`.
  **L239 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_3>]>;`。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 241-264

````tablegen
  def NAME # _ZA32_H: SInst<"sv" # n_suffix # "_za32[_{d}]", "viPPdd", "b",
                            MergeOp1, "aarch64_sme_" # n_suffix # "_wide",
                            [IsStreaming, IsInOutZA],
                            [ImmCheck<0, ImmCheck0_3>]>;

  def NAME # _ZA32_S: SInst<"sv" # n_suffix # "_za32[_{d}]", "viPPdd", "f",
                            MergeOp1, "aarch64_sme_" # n_suffix,
                            [IsStreaming, IsInOutZA],
                            [ImmCheck<0, ImmCheck0_3>]>;

  let SMETargetGuard = "sme-f64f64" in {
    def NAME # _ZA64_D: SInst<"sv" # n_suffix # "_za64[_{d}]", "viPPdd", "d",
                              MergeOp1, "aarch64_sme_" # n_suffix,
                              [IsStreaming, IsInOutZA],
                              [ImmCheck<0, ImmCheck0_7>]>;
  }
}

defm SVMOPA : ZAFPOuterProd<"mopa">;
defm SVMOPS : ZAFPOuterProd<"mops">;

////////////////////////////////////////////////////////////////////////////////
// SME2 - FMOP4A, FMOP4S, BFMOP4A, BFMOP4S

````
- **L241 EN**: Declares TableGen def record `NAME # _ZA32_H`.
  **L241 CN**: 声明 TableGen def 记录 `NAME # _ZA32_H`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix # "_wide",`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix # "_wide",`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L244 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_3>]>;`.
  **L244 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_3>]>;`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Declares TableGen def record `NAME # _ZA32_S`.
  **L246 CN**: 声明 TableGen def 记录 `NAME # _ZA32_S`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix,`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L249 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_3>]>;`.
  **L249 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_3>]>;`。
- **L250 EN**: Blank line separating nearby declarations or logic blocks.
  **L250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L251 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f64f64" in {`.
  **L251 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f64f64" in {`。
- **L252 EN**: Declares TableGen def record `NAME # _ZA64_D`.
  **L252 CN**: 声明 TableGen def 记录 `NAME # _ZA64_D`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeOp1, "aarch64_sme_" # n_suffix,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeOp1, "aarch64_sme_" # n_suffix,`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L255 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_7>]>;`.
  **L255 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_7>]>;`。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Closes the current lexical scope or compound statement.
  **L257 CN**: 结束当前词法作用域或复合语句块。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L259 EN**: Declares TableGen defm record `SVMOPA`.
  **L259 CN**: 声明 TableGen defm 记录 `SVMOPA`。
- **L260 EN**: Declares TableGen defm record `SVMOPS`.
  **L260 CN**: 声明 TableGen defm 记录 `SVMOPS`。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L262 EN**: Separator comment used for visual grouping.
  **L262 CN**: 用于视觉分组的分隔注释。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `SME2 - FMOP4A, FMOP4S, BFMOP4A, BFMOP4S`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 - FMOP4A, FMOP4S, BFMOP4A, BFMOP4S`。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 265-288

````tablegen
multiclass MOP4<string mode, string za, string t, string i, list<ImmCheck> checks> {
  def _1x1 : Inst<"svmop4" # mode # "[_1x1]" # za # "[_{d}_{d}]", "vidd", t, MergeNone, i # "_1x1", [IsInOutZA, IsStreaming], checks>;
  def _1x2 : Inst<"svmop4" # mode # "[_1x2]" # za # "[_{d}_{d}]", "vid2", t, MergeNone, i # "_1x2", [IsInOutZA, IsStreaming], checks>;
  def _2x1 : Inst<"svmop4" # mode # "[_2x1]" # za # "[_{d}_{d}]", "vi2d", t, MergeNone, i # "_2x1", [IsInOutZA, IsStreaming], checks>;
  def _2x2 : Inst<"svmop4" # mode # "[_2x2]" # za # "[_{d}_{d}]", "vi22", t, MergeNone, i # "_2x2", [IsInOutZA, IsStreaming], checks>;
}

let SMETargetGuard = "sme2,sme-mop4" in {
  defm SVFMOP4A_HtoS  : MOP4<"a", "_za32", "hb", "aarch64_sme_mop4a_wide", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVFMOP4S_HtoS  : MOP4<"s", "_za32", "hb", "aarch64_sme_mop4s_wide", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVFMOP4A_S     : MOP4<"a", "_za32", "f", "aarch64_sme_mop4a", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVFMOP4S_S     : MOP4<"s", "_za32", "f", "aarch64_sme_mop4s", [ImmCheck<0, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-mop4,sme-f64f64" in {
  defm SVFMOP4A_D : MOP4<"a", "_za64", "d", "aarch64_sme_mop4a", [ImmCheck<0, ImmCheck0_7>]>;
  defm SVFMOP4S_D : MOP4<"s", "_za64", "d", "aarch64_sme_mop4s", [ImmCheck<0, ImmCheck0_7>]>;
}

let SMETargetGuard = "sme2,sme-mop4,sme-f16f16" in {
  defm SVFMOP4A_H : MOP4<"a", "_za16", "h", "aarch64_sme_mop4a", [ImmCheck<0, ImmCheck0_1>]>;
  defm SVFMOP4S_H : MOP4<"s", "_za16", "h", "aarch64_sme_mop4s", [ImmCheck<0, ImmCheck0_1>]>;
}

````
- **L265 EN**: Declares TableGen multiclass record `MOP4`.
  **L265 CN**: 声明 TableGen multiclass 记录 `MOP4`。
- **L266 EN**: Declares TableGen def record `_1x1`.
  **L266 CN**: 声明 TableGen def 记录 `_1x1`。
- **L267 EN**: Declares TableGen def record `_1x2`.
  **L267 CN**: 声明 TableGen def 记录 `_1x2`。
- **L268 EN**: Declares TableGen def record `_2x1`.
  **L268 CN**: 声明 TableGen def 记录 `_2x1`。
- **L269 EN**: Declares TableGen def record `_2x2`.
  **L269 CN**: 声明 TableGen def 记录 `_2x2`。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Blank line separating nearby declarations or logic blocks.
  **L271 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L272 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4" in {`.
  **L272 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4" in {`。
- **L273 EN**: Declares TableGen defm record `SVFMOP4A_HtoS`.
  **L273 CN**: 声明 TableGen defm 记录 `SVFMOP4A_HtoS`。
- **L274 EN**: Declares TableGen defm record `SVFMOP4S_HtoS`.
  **L274 CN**: 声明 TableGen defm 记录 `SVFMOP4S_HtoS`。
- **L275 EN**: Declares TableGen defm record `SVFMOP4A_S`.
  **L275 CN**: 声明 TableGen defm 记录 `SVFMOP4A_S`。
- **L276 EN**: Declares TableGen defm record `SVFMOP4S_S`.
  **L276 CN**: 声明 TableGen defm 记录 `SVFMOP4S_S`。
- **L277 EN**: Closes the current lexical scope or compound statement.
  **L277 CN**: 结束当前词法作用域或复合语句块。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L279 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4,sme-f64f64" in {`.
  **L279 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4,sme-f64f64" in {`。
- **L280 EN**: Declares TableGen defm record `SVFMOP4A_D`.
  **L280 CN**: 声明 TableGen defm 记录 `SVFMOP4A_D`。
- **L281 EN**: Declares TableGen defm record `SVFMOP4S_D`.
  **L281 CN**: 声明 TableGen defm 记录 `SVFMOP4S_D`。
- **L282 EN**: Closes the current lexical scope or compound statement.
  **L282 CN**: 结束当前词法作用域或复合语句块。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L284 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4,sme-f16f16" in {`.
  **L284 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4,sme-f16f16" in {`。
- **L285 EN**: Declares TableGen defm record `SVFMOP4A_H`.
  **L285 CN**: 声明 TableGen defm 记录 `SVFMOP4A_H`。
- **L286 EN**: Declares TableGen defm record `SVFMOP4S_H`.
  **L286 CN**: 声明 TableGen defm 记录 `SVFMOP4S_H`。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 289-312

````tablegen
let SMETargetGuard = "sme2,sme-mop4,sme-b16b16" in {
  defm SVBMOP4A_H : MOP4<"a", "_za16", "b", "aarch64_sme_mop4a", [ImmCheck<0, ImmCheck0_1>]>;
  defm SVBMOP4S_H : MOP4<"s", "_za16", "b", "aarch64_sme_mop4s", [ImmCheck<0, ImmCheck0_1>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SME2 - FP8 FMOP4A, FMOP4S

multiclass MOP4_FP8<string za, list<ImmCheck> checks> {
  def _1x1 : Inst<"svmop4a" # "[_1x1]" # za # "[_{d}_{d}]", "vidd>", "m", MergeNone, "aarch64_sme_fp8_fmop4a" # za # "_1x1", [IsInOutZA, IsStreaming], checks>;
  def _1x2 : Inst<"svmop4a" # "[_1x2]" # za # "[_{d}_{d}]", "vid2>", "m", MergeNone, "aarch64_sme_fp8_fmop4a" # za # "_1x2", [IsInOutZA, IsStreaming], checks>;
  def _2x1 : Inst<"svmop4a" # "[_2x1]" # za # "[_{d}_{d}]", "vi2d>", "m", MergeNone, "aarch64_sme_fp8_fmop4a" # za # "_2x1", [IsInOutZA, IsStreaming], checks>;
  def _2x2 : Inst<"svmop4a" # "[_2x2]" # za # "[_{d}_{d}]", "vi22>", "m", MergeNone, "aarch64_sme_fp8_fmop4a" # za # "_2x2", [IsInOutZA, IsStreaming], checks>;
}

let SMETargetGuard = "sme2,sme-mop4,sme-f8f32" in {
  defm SVMOP4A_FP8_ZA32 : MOP4_FP8<"_za32", [ImmCheck<0, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-mop4,sme-f8f16" in {
  defm SVMOP4A_FP8_ZA16 : MOP4_FP8<"_za16", [ImmCheck<0, ImmCheck0_1>]>;
}

////////////////////////////////////////////////////////////////////////////////
````
- **L289 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4,sme-b16b16" in {`.
  **L289 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4,sme-b16b16" in {`。
- **L290 EN**: Declares TableGen defm record `SVBMOP4A_H`.
  **L290 CN**: 声明 TableGen defm 记录 `SVBMOP4A_H`。
- **L291 EN**: Declares TableGen defm record `SVBMOP4S_H`.
  **L291 CN**: 声明 TableGen defm 记录 `SVBMOP4S_H`。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L294 EN**: Separator comment used for visual grouping.
  **L294 CN**: 用于视觉分组的分隔注释。
- **L295 EN**: Comment explains nearby logic, constraints, or intent: `SME2 - FP8 FMOP4A, FMOP4S`.
  **L295 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 - FP8 FMOP4A, FMOP4S`。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Declares TableGen multiclass record `MOP4_FP8`.
  **L297 CN**: 声明 TableGen multiclass 记录 `MOP4_FP8`。
- **L298 EN**: Declares TableGen def record `_1x1`.
  **L298 CN**: 声明 TableGen def 记录 `_1x1`。
- **L299 EN**: Declares TableGen def record `_1x2`.
  **L299 CN**: 声明 TableGen def 记录 `_1x2`。
- **L300 EN**: Declares TableGen def record `_2x1`.
  **L300 CN**: 声明 TableGen def 记录 `_2x1`。
- **L301 EN**: Declares TableGen def record `_2x2`.
  **L301 CN**: 声明 TableGen def 记录 `_2x2`。
- **L302 EN**: Closes the current lexical scope or compound statement.
  **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L304 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4,sme-f8f32" in {`.
  **L304 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4,sme-f8f32" in {`。
- **L305 EN**: Declares TableGen defm record `SVMOP4A_FP8_ZA32`.
  **L305 CN**: 声明 TableGen defm 记录 `SVMOP4A_FP8_ZA32`。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L308 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4,sme-f8f16" in {`.
  **L308 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4,sme-f8f16" in {`。
- **L309 EN**: Declares TableGen defm record `SVMOP4A_FP8_ZA16`.
  **L309 CN**: 声明 TableGen defm 记录 `SVMOP4A_FP8_ZA16`。
- **L310 EN**: Closes the current lexical scope or compound statement.
  **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L312 EN**: Separator comment used for visual grouping.
  **L312 CN**: 用于视觉分组的分隔注释。

### Lines 313-336

````tablegen
// SME2 - SMOP4A, SMOP4S, UMOP4A, UMOP4S

let SMETargetGuard = "sme2,sme-mop4" in {
  defm SVSMOP4A_H  : MOP4<"a", "_za32", "cs", "aarch64_sme_smop4a_wide", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVSMOP4S_H  : MOP4<"s", "_za32", "cs", "aarch64_sme_smop4s_wide", [ImmCheck<0, ImmCheck0_3>]>;

  defm SVUMOP4A_H  : MOP4<"a", "_za32", "UcUs", "aarch64_sme_umop4a_wide", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVUMOP4S_H  : MOP4<"s", "_za32", "UcUs", "aarch64_sme_umop4s_wide", [ImmCheck<0, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-mop4,sme-i16i64" in {
  defm SVSMOP4A_HtoD  : MOP4<"a", "_za64", "s", "aarch64_sme_smop4a_za64_wide", [ImmCheck<0, ImmCheck0_7>]>;
  defm SVSMOP4S_HtoD  : MOP4<"s", "_za64", "s", "aarch64_sme_smop4s_za64_wide", [ImmCheck<0, ImmCheck0_7>]>;

  defm SVUMOP4A_HtoD  : MOP4<"a", "_za64", "Us", "aarch64_sme_umop4a_za64_wide", [ImmCheck<0, ImmCheck0_7>]>;
  defm SVUMOP4S_HtoD  : MOP4<"s", "_za64", "Us", "aarch64_sme_umop4s_za64_wide", [ImmCheck<0, ImmCheck0_7>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SME2 - SUMOP4A, SUMOP4S, USMOP4A, USMOP4S

multiclass SUMOP4<string mode, string za, string t, string i, list<ImmCheck> checks> {
  def _1x1 : SInst<"svmop4" # mode # "[_1x1]" # za # "[_{d}_{3}]",
                              "vidu", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_1x1",
````
- **L313 EN**: Comment explains nearby logic, constraints, or intent: `SME2 - SMOP4A, SMOP4S, UMOP4A, UMOP4S`.
  **L313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 - SMOP4A, SMOP4S, UMOP4A, UMOP4S`。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4" in {`.
  **L315 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4" in {`。
- **L316 EN**: Declares TableGen defm record `SVSMOP4A_H`.
  **L316 CN**: 声明 TableGen defm 记录 `SVSMOP4A_H`。
- **L317 EN**: Declares TableGen defm record `SVSMOP4S_H`.
  **L317 CN**: 声明 TableGen defm 记录 `SVSMOP4S_H`。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L319 EN**: Declares TableGen defm record `SVUMOP4A_H`.
  **L319 CN**: 声明 TableGen defm 记录 `SVUMOP4A_H`。
- **L320 EN**: Declares TableGen defm record `SVUMOP4S_H`.
  **L320 CN**: 声明 TableGen defm 记录 `SVUMOP4S_H`。
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4,sme-i16i64" in {`.
  **L323 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4,sme-i16i64" in {`。
- **L324 EN**: Declares TableGen defm record `SVSMOP4A_HtoD`.
  **L324 CN**: 声明 TableGen defm 记录 `SVSMOP4A_HtoD`。
- **L325 EN**: Declares TableGen defm record `SVSMOP4S_HtoD`.
  **L325 CN**: 声明 TableGen defm 记录 `SVSMOP4S_HtoD`。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Declares TableGen defm record `SVUMOP4A_HtoD`.
  **L327 CN**: 声明 TableGen defm 记录 `SVUMOP4A_HtoD`。
- **L328 EN**: Declares TableGen defm record `SVUMOP4S_HtoD`.
  **L328 CN**: 声明 TableGen defm 记录 `SVUMOP4S_HtoD`。
- **L329 EN**: Closes the current lexical scope or compound statement.
  **L329 CN**: 结束当前词法作用域或复合语句块。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L331 EN**: Separator comment used for visual grouping.
  **L331 CN**: 用于视觉分组的分隔注释。
- **L332 EN**: Comment explains nearby logic, constraints, or intent: `SME2 - SUMOP4A, SUMOP4S, USMOP4A, USMOP4S`.
  **L332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 - SUMOP4A, SUMOP4S, USMOP4A, USMOP4S`。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L334 EN**: Declares TableGen multiclass record `SUMOP4`.
  **L334 CN**: 声明 TableGen multiclass 记录 `SUMOP4`。
- **L335 EN**: Declares TableGen def record `_1x1`.
  **L335 CN**: 声明 TableGen def 记录 `_1x1`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vidu", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_1x1",`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vidu", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_1x1",`。

### Lines 337-360

````tablegen
                              [IsStreaming, IsInOutZA],
                              checks>;
  def _1x2 : SInst<"svmop4" # mode # "[_1x2]" # za # "[_{d}_{3}]",
                              "vid2.u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_1x2",
                              [IsStreaming, IsInOutZA],
                              checks>;
  def _2x1 : SInst<"svmop4" # mode # "[_2x1]" # za # "[_{d}_{3}]",
                              "vi2u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_2x1",
                              [IsStreaming, IsInOutZA],
                              checks>;
  def _2x2 : SInst<"svmop4" # mode # "[_2x2]" # za # "[_{d}_{3}]",
                              "vi2.x2.u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_2x2",
                              [IsStreaming, IsInOutZA],
                              checks>;
}

multiclass USMOP4<string mode, string za, string t, string i, list<ImmCheck> checks> {
  def _1x1 : SInst<"svmop4" # mode # "[_1x1]" # za # "[_{d}_{3}]",
                              "vidx", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_1x1",
                              [IsStreaming, IsInOutZA],
                              checks>;
  def _1x2 : SInst<"svmop4" # mode # "[_1x2]" # za # "[_{d}_{3}]",
                              "vid2.x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_1x2",
                              [IsStreaming, IsInOutZA],
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L338 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L338 CN**: 添加一条独立语句或声明：`checks>;`。
- **L339 EN**: Declares TableGen def record `_1x2`.
  **L339 CN**: 声明 TableGen def 记录 `_1x2`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vid2.u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_1x2",`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vid2.u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_1x2",`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L342 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L342 CN**: 添加一条独立语句或声明：`checks>;`。
- **L343 EN**: Declares TableGen def record `_2x1`.
  **L343 CN**: 声明 TableGen def 记录 `_2x1`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vi2u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_2x1",`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vi2u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_2x1",`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L346 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L346 CN**: 添加一条独立语句或声明：`checks>;`。
- **L347 EN**: Declares TableGen def record `_2x2`.
  **L347 CN**: 声明 TableGen def 记录 `_2x2`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vi2.x2.u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_2x2",`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vi2.x2.u", t, MergeNone, "aarch64_sme_sumop4" # mode # i # "_wide_2x2",`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L350 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L350 CN**: 添加一条独立语句或声明：`checks>;`。
- **L351 EN**: Closes the current lexical scope or compound statement.
  **L351 CN**: 结束当前词法作用域或复合语句块。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Declares TableGen multiclass record `USMOP4`.
  **L353 CN**: 声明 TableGen multiclass 记录 `USMOP4`。
- **L354 EN**: Declares TableGen def record `_1x1`.
  **L354 CN**: 声明 TableGen def 记录 `_1x1`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vidx", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_1x1",`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vidx", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_1x1",`。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L357 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L357 CN**: 添加一条独立语句或声明：`checks>;`。
- **L358 EN**: Declares TableGen def record `_1x2`.
  **L358 CN**: 声明 TableGen def 记录 `_1x2`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vid2.x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_1x2",`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vid2.x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_1x2",`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。

### Lines 361-384

````tablegen
                              checks>;
  def _2x1 : SInst<"svmop4" # mode # "[_2x1]" # za # "[_{d}_{3}]",
                              "vi2x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_2x1",
                              [IsStreaming, IsInOutZA],
                              checks>;
  def _2x2 : SInst<"svmop4" # mode # "[_2x2]" # za # "[_{d}_{3}]",
                              "vi2.u2.x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_2x2",
                              [IsStreaming, IsInOutZA],
                              checks>;
}

let SMETargetGuard = "sme2,sme-mop4" in {
  defm SVSUMOP4A_S : SUMOP4<"a", "_za32", "c", "", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVSUMOP4S_S : SUMOP4<"s", "_za32", "c", "", [ImmCheck<0, ImmCheck0_3>]>;

  defm SVUSMOP4A_S : USMOP4<"a", "_za32", "Uc", "", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVUSMOP4S_S : USMOP4<"s", "_za32", "Uc", "", [ImmCheck<0, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-mop4,sme-i16i64" in {
  defm SVSUMOP4A_D : SUMOP4<"a", "_za64", "s", "_za64", [ImmCheck<0, ImmCheck0_7>]>;
  defm SVSUMOP4S_D : SUMOP4<"s", "_za64", "s", "_za64", [ImmCheck<0, ImmCheck0_7>]>;

  defm SVUSMOP4A_D : USMOP4<"a", "_za64", "Us", "_za64", [ImmCheck<0, ImmCheck0_7>]>;
````
- **L361 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L361 CN**: 添加一条独立语句或声明：`checks>;`。
- **L362 EN**: Declares TableGen def record `_2x1`.
  **L362 CN**: 声明 TableGen def 记录 `_2x1`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vi2x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_2x1",`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vi2x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_2x1",`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L365 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L365 CN**: 添加一条独立语句或声明：`checks>;`。
- **L366 EN**: Declares TableGen def record `_2x2`.
  **L366 CN**: 声明 TableGen def 记录 `_2x2`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"vi2.u2.x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_2x2",`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`"vi2.u2.x", t, MergeNone, "aarch64_sme_usmop4" # mode # i # "_wide_2x2",`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L369 EN**: Adds a standalone statement or declaration: `checks>;`.
  **L369 CN**: 添加一条独立语句或声明：`checks>;`。
- **L370 EN**: Closes the current lexical scope or compound statement.
  **L370 CN**: 结束当前词法作用域或复合语句块。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L372 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4" in {`.
  **L372 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4" in {`。
- **L373 EN**: Declares TableGen defm record `SVSUMOP4A_S`.
  **L373 CN**: 声明 TableGen defm 记录 `SVSUMOP4A_S`。
- **L374 EN**: Declares TableGen defm record `SVSUMOP4S_S`.
  **L374 CN**: 声明 TableGen defm 记录 `SVSUMOP4S_S`。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L376 EN**: Declares TableGen defm record `SVUSMOP4A_S`.
  **L376 CN**: 声明 TableGen defm 记录 `SVUSMOP4A_S`。
- **L377 EN**: Declares TableGen defm record `SVUSMOP4S_S`.
  **L377 CN**: 声明 TableGen defm 记录 `SVUSMOP4S_S`。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L380 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-mop4,sme-i16i64" in {`.
  **L380 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-mop4,sme-i16i64" in {`。
- **L381 EN**: Declares TableGen defm record `SVSUMOP4A_D`.
  **L381 CN**: 声明 TableGen defm 记录 `SVSUMOP4A_D`。
- **L382 EN**: Declares TableGen defm record `SVSUMOP4S_D`.
  **L382 CN**: 声明 TableGen defm 记录 `SVSUMOP4S_D`。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L384 EN**: Declares TableGen defm record `SVUSMOP4A_D`.
  **L384 CN**: 声明 TableGen defm 记录 `SVUSMOP4A_D`。

### Lines 385-408

````tablegen
  defm SVUSMOP4S_D : USMOP4<"s", "_za64", "Us", "_za64", [ImmCheck<0, ImmCheck0_7>]>;
}

////////////////////////////////////////////////////////////////////////////////
// SME2 - ADD, SUB

multiclass ZAAddSub<string n_suffix> {
  let SMETargetGuard = "sme2" in {
    def NAME # _WRITE_SINGLE_ZA32_VG1X2_I32 : Inst<"sv" # n_suffix # "_write[_single]_za32[_{d}]_vg1x2", "vm2d", "iUi", MergeNone, "aarch64_sme_" # n_suffix # "_write_single_za_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _WRITE_SINGLE_ZA32_VG1X4_I32 : Inst<"sv" # n_suffix # "_write[_single]_za32[_{d}]_vg1x4", "vm4d", "iUi", MergeNone, "aarch64_sme_" # n_suffix # "_write_single_za_vg1x4", [IsStreaming, IsInOutZA], []>;

    def NAME # _WRITE_ZA32_VG1X2_I32 : Inst<"sv" # n_suffix # "_write_za32[_{d}]_vg1x2", "vm22", "iUi", MergeNone, "aarch64_sme_" # n_suffix # "_write_za_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _WRITE_ZA32_VG1X4_I32 : Inst<"sv" # n_suffix # "_write_za32[_{d}]_vg1x4", "vm44", "iUi", MergeNone, "aarch64_sme_" # n_suffix # "_write_za_vg1x4", [IsStreaming, IsInOutZA], []>;

    def NAME # _ZA32_VG1x2_I32 : Inst<"sv" # n_suffix # "_za32[_{d}]_vg1x2", "vm2", "iUif", MergeNone, "aarch64_sme_" # n_suffix # "_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _ZA32_VG1X4_I32 : Inst<"sv" # n_suffix # "_za32[_{d}]_vg1x4", "vm4", "iUif", MergeNone, "aarch64_sme_" # n_suffix # "_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  }

  let SMETargetGuard = "sme2,sme-i16i64" in {
    def NAME # _WRITE_SINGLE_ZA64_VG1X2_I64 : Inst<"sv" # n_suffix # "_write[_single]_za64[_{d}]_vg1x2", "vm2d", "lUl", MergeNone, "aarch64_sme_" # n_suffix # "_write_single_za_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _WRITE_SINGLE_ZA64_VG1X4_I64 : Inst<"sv" # n_suffix # "_write[_single]_za64[_{d}]_vg1x4", "vm4d", "lUl", MergeNone, "aarch64_sme_" # n_suffix # "_write_single_za_vg1x4", [IsStreaming, IsInOutZA], []>;

    def NAME # _WRITE_ZA64_VG1x2_I64 : Inst<"sv" # n_suffix # "_write_za64[_{d}]_vg1x2", "vm22", "lUl", MergeNone, "aarch64_sme_" # n_suffix # "_write_za_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _WRITE_ZA64_VG1x4_I64 : Inst<"sv" # n_suffix # "_write_za64[_{d}]_vg1x4", "vm44", "lUl", MergeNone, "aarch64_sme_" # n_suffix # "_write_za_vg1x4", [IsStreaming, IsInOutZA], []>;
````
- **L385 EN**: Declares TableGen defm record `SVUSMOP4S_D`.
  **L385 CN**: 声明 TableGen defm 记录 `SVUSMOP4S_D`。
- **L386 EN**: Closes the current lexical scope or compound statement.
  **L386 CN**: 结束当前词法作用域或复合语句块。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L388 EN**: Separator comment used for visual grouping.
  **L388 CN**: 用于视觉分组的分隔注释。
- **L389 EN**: Comment explains nearby logic, constraints, or intent: `SME2 - ADD, SUB`.
  **L389 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 - ADD, SUB`。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Declares TableGen multiclass record `ZAAddSub`.
  **L391 CN**: 声明 TableGen multiclass 记录 `ZAAddSub`。
- **L392 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L392 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L393 EN**: Declares TableGen def record `NAME # _WRITE_SINGLE_ZA32_VG1X2_I32`.
  **L393 CN**: 声明 TableGen def 记录 `NAME # _WRITE_SINGLE_ZA32_VG1X2_I32`。
- **L394 EN**: Declares TableGen def record `NAME # _WRITE_SINGLE_ZA32_VG1X4_I32`.
  **L394 CN**: 声明 TableGen def 记录 `NAME # _WRITE_SINGLE_ZA32_VG1X4_I32`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Declares TableGen def record `NAME # _WRITE_ZA32_VG1X2_I32`.
  **L396 CN**: 声明 TableGen def 记录 `NAME # _WRITE_ZA32_VG1X2_I32`。
- **L397 EN**: Declares TableGen def record `NAME # _WRITE_ZA32_VG1X4_I32`.
  **L397 CN**: 声明 TableGen def 记录 `NAME # _WRITE_ZA32_VG1X4_I32`。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L399 EN**: Declares TableGen def record `NAME # _ZA32_VG1x2_I32`.
  **L399 CN**: 声明 TableGen def 记录 `NAME # _ZA32_VG1x2_I32`。
- **L400 EN**: Declares TableGen def record `NAME # _ZA32_VG1X4_I32`.
  **L400 CN**: 声明 TableGen def 记录 `NAME # _ZA32_VG1X4_I32`。
- **L401 EN**: Closes the current lexical scope or compound statement.
  **L401 CN**: 结束当前词法作用域或复合语句块。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L403 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-i16i64" in {`.
  **L403 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-i16i64" in {`。
- **L404 EN**: Declares TableGen def record `NAME # _WRITE_SINGLE_ZA64_VG1X2_I64`.
  **L404 CN**: 声明 TableGen def 记录 `NAME # _WRITE_SINGLE_ZA64_VG1X2_I64`。
- **L405 EN**: Declares TableGen def record `NAME # _WRITE_SINGLE_ZA64_VG1X4_I64`.
  **L405 CN**: 声明 TableGen def 记录 `NAME # _WRITE_SINGLE_ZA64_VG1X4_I64`。
- **L406 EN**: Blank line separating nearby declarations or logic blocks.
  **L406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L407 EN**: Declares TableGen def record `NAME # _WRITE_ZA64_VG1x2_I64`.
  **L407 CN**: 声明 TableGen def 记录 `NAME # _WRITE_ZA64_VG1x2_I64`。
- **L408 EN**: Declares TableGen def record `NAME # _WRITE_ZA64_VG1x4_I64`.
  **L408 CN**: 声明 TableGen def 记录 `NAME # _WRITE_ZA64_VG1x4_I64`。

### Lines 409-432

````tablegen

    def NAME # _ZA64_VG1X2_I64 : Inst<"sv" # n_suffix # "_za64[_{d}]_vg1x2", "vm2", "lUl", MergeNone, "aarch64_sme_" # n_suffix # "_za64_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _ZA64_VG1X4_I64 : Inst<"sv" # n_suffix # "_za64[_{d}]_vg1x4", "vm4", "lUl", MergeNone, "aarch64_sme_" # n_suffix # "_za64_vg1x4", [IsStreaming, IsInOutZA], []>;
  }

  let SMETargetGuard = "sme2,sme-f64f64" in {
    def NAME # _ZA64_VG1X2_F64 : Inst<"sv" # n_suffix # "_za64[_{d}]_vg1x2", "vm2", "d", MergeNone, "aarch64_sme_" # n_suffix # "_za64_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _ZA64_VG1X4_F64 : Inst<"sv" # n_suffix # "_za64[_{d}]_vg1x4", "vm4", "d", MergeNone, "aarch64_sme_" # n_suffix # "_za64_vg1x4", [IsStreaming, IsInOutZA], []>;
  }

  let SMETargetGuard = "sme-f16f16|sme-f8f16" in {
    def NAME # _ZA16_VG1X2_F16 : Inst<"sv" # n_suffix # "_za16[_{d}]_vg1x2", "vm2", "h", MergeNone, "aarch64_sme_" # n_suffix # "_za16_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _ZA16_VG1X4_F16 : Inst<"sv" # n_suffix # "_za16[_{d}]_vg1x4", "vm4", "h", MergeNone, "aarch64_sme_" # n_suffix # "_za16_vg1x4", [IsStreaming, IsInOutZA], []>;
  }

  let SMETargetGuard = "sme-b16b16" in {
    def NAME # _ZA16_VG1X2_BF16 : Inst<"sv" # n_suffix # "_za16[_{d}]_vg1x2", "vm2", "b", MergeNone, "aarch64_sme_" # n_suffix # "_za16_vg1x2", [IsStreaming, IsInOutZA], []>;
    def NAME # _ZA16_VG1X4_BF16 : Inst<"sv" # n_suffix # "_za16[_{d}]_vg1x4", "vm4", "b", MergeNone, "aarch64_sme_" # n_suffix # "_za16_vg1x4", [IsStreaming, IsInOutZA], []>;
  }
}

defm SVADD : ZAAddSub<"add">;
defm SVSUB : ZAAddSub<"sub">;

````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L410 EN**: Declares TableGen def record `NAME # _ZA64_VG1X2_I64`.
  **L410 CN**: 声明 TableGen def 记录 `NAME # _ZA64_VG1X2_I64`。
- **L411 EN**: Declares TableGen def record `NAME # _ZA64_VG1X4_I64`.
  **L411 CN**: 声明 TableGen def 记录 `NAME # _ZA64_VG1X4_I64`。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-f64f64" in {`.
  **L414 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-f64f64" in {`。
- **L415 EN**: Declares TableGen def record `NAME # _ZA64_VG1X2_F64`.
  **L415 CN**: 声明 TableGen def 记录 `NAME # _ZA64_VG1X2_F64`。
- **L416 EN**: Declares TableGen def record `NAME # _ZA64_VG1X4_F64`.
  **L416 CN**: 声明 TableGen def 记录 `NAME # _ZA64_VG1X4_F64`。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L419 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f16f16|sme-f8f16" in {`.
  **L419 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f16f16|sme-f8f16" in {`。
- **L420 EN**: Declares TableGen def record `NAME # _ZA16_VG1X2_F16`.
  **L420 CN**: 声明 TableGen def 记录 `NAME # _ZA16_VG1X2_F16`。
- **L421 EN**: Declares TableGen def record `NAME # _ZA16_VG1X4_F16`.
  **L421 CN**: 声明 TableGen def 记录 `NAME # _ZA16_VG1X4_F16`。
- **L422 EN**: Closes the current lexical scope or compound statement.
  **L422 CN**: 结束当前词法作用域或复合语句块。
- **L423 EN**: Blank line separating nearby declarations or logic blocks.
  **L423 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L424 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-b16b16" in {`.
  **L424 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-b16b16" in {`。
- **L425 EN**: Declares TableGen def record `NAME # _ZA16_VG1X2_BF16`.
  **L425 CN**: 声明 TableGen def 记录 `NAME # _ZA16_VG1X2_BF16`。
- **L426 EN**: Declares TableGen def record `NAME # _ZA16_VG1X4_BF16`.
  **L426 CN**: 声明 TableGen def 记录 `NAME # _ZA16_VG1X4_BF16`。
- **L427 EN**: Closes the current lexical scope or compound statement.
  **L427 CN**: 结束当前词法作用域或复合语句块。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Declares TableGen defm record `SVADD`.
  **L430 CN**: 声明 TableGen defm 记录 `SVADD`。
- **L431 EN**: Declares TableGen defm record `SVSUB`.
  **L431 CN**: 声明 TableGen defm 记录 `SVSUB`。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````tablegen
// SME2 - MOVA

//
// Single, 2 and 4 vector-group read/write intrinsics.
//

multiclass ZAWrite_VG<string n, string t, string i, list<ImmCheck> checks> {
  def NAME # _VG2_H : Inst<"svwrite_hor_" # n # "[_{d}]_vg2",   "vim2", t, MergeNone, i # "_hor_vg2", [IsInOutZA, IsStreaming], checks>;
  def NAME # _VG2_V : Inst<"svwrite_ver_" # n # "[_{d}]_vg2",   "vim2", t, MergeNone, i # "_ver_vg2", [IsInOutZA, IsStreaming], checks>;
  def NAME # _VG4_H : Inst<"svwrite_hor_" # n # "[_{d}]_vg4",   "vim4", t, MergeNone, i # "_hor_vg4", [IsInOutZA, IsStreaming], checks>;
  def NAME # _VG4_V : Inst<"svwrite_ver_" # n # "[_{d}]_vg4",   "vim4", t, MergeNone, i # "_ver_vg4", [IsInOutZA, IsStreaming], checks>;
  def NAME # _VG1x2 : Inst<"svwrite_"     # n # "[_{d}]_vg1x2", "vm2",  t, MergeNone, i # "_vg1x2",   [IsInOutZA, IsStreaming], []>;
  def NAME # _VG1x4 : Inst<"svwrite_"     # n # "[_{d}]_vg1x4", "vm4",  t, MergeNone, i # "_vg1x4",   [IsInOutZA, IsStreaming], []>;
}

let SMETargetGuard = "sme2" in {
  defm SVWRITE_ZA8  : ZAWrite_VG<"za8",  "cUcm",   "aarch64_sme_write", [ImmCheck<0, ImmCheck0_0>]>;
  defm SVWRITE_ZA16 : ZAWrite_VG<"za16", "sUshb", "aarch64_sme_write", [ImmCheck<0, ImmCheck0_1>]>;
  defm SVWRITE_ZA32 : ZAWrite_VG<"za32", "iUif",  "aarch64_sme_write", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVWRITE_ZA64 : ZAWrite_VG<"za64", "lUld",  "aarch64_sme_write", [ImmCheck<0, ImmCheck0_7>]>;
}

multiclass ZARead_VG<string n, string t, string i, list<ImmCheck> checks> {
  def NAME # _VG2_H : Inst<"svread_hor_" # n # "_{d}_vg2",   "2im", t, MergeNone, i # "_hor_vg2", [IsInZA, IsStreaming], checks>;
````
- **L433 EN**: Comment explains nearby logic, constraints, or intent: `SME2 - MOVA`.
  **L433 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 - MOVA`。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Separator comment used for visual grouping.
  **L435 CN**: 用于视觉分组的分隔注释。
- **L436 EN**: Comment explains nearby logic, constraints, or intent: `Single, 2 and 4 vector-group read/write intrinsics.`.
  **L436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Single, 2 and 4 vector-group read/write intrinsics.`。
- **L437 EN**: Separator comment used for visual grouping.
  **L437 CN**: 用于视觉分组的分隔注释。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L439 EN**: Declares TableGen multiclass record `ZAWrite_VG`.
  **L439 CN**: 声明 TableGen multiclass 记录 `ZAWrite_VG`。
- **L440 EN**: Declares TableGen def record `NAME # _VG2_H`.
  **L440 CN**: 声明 TableGen def 记录 `NAME # _VG2_H`。
- **L441 EN**: Declares TableGen def record `NAME # _VG2_V`.
  **L441 CN**: 声明 TableGen def 记录 `NAME # _VG2_V`。
- **L442 EN**: Declares TableGen def record `NAME # _VG4_H`.
  **L442 CN**: 声明 TableGen def 记录 `NAME # _VG4_H`。
- **L443 EN**: Declares TableGen def record `NAME # _VG4_V`.
  **L443 CN**: 声明 TableGen def 记录 `NAME # _VG4_V`。
- **L444 EN**: Declares TableGen def record `NAME # _VG1x2`.
  **L444 CN**: 声明 TableGen def 记录 `NAME # _VG1x2`。
- **L445 EN**: Declares TableGen def record `NAME # _VG1x4`.
  **L445 CN**: 声明 TableGen def 记录 `NAME # _VG1x4`。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L448 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L449 EN**: Declares TableGen defm record `SVWRITE_ZA8`.
  **L449 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA8`。
- **L450 EN**: Declares TableGen defm record `SVWRITE_ZA16`.
  **L450 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA16`。
- **L451 EN**: Declares TableGen defm record `SVWRITE_ZA32`.
  **L451 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA32`。
- **L452 EN**: Declares TableGen defm record `SVWRITE_ZA64`.
  **L452 CN**: 声明 TableGen defm 记录 `SVWRITE_ZA64`。
- **L453 EN**: Closes the current lexical scope or compound statement.
  **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Declares TableGen multiclass record `ZARead_VG`.
  **L455 CN**: 声明 TableGen multiclass 记录 `ZARead_VG`。
- **L456 EN**: Declares TableGen def record `NAME # _VG2_H`.
  **L456 CN**: 声明 TableGen def 记录 `NAME # _VG2_H`。

### Lines 457-480

````tablegen
  def NAME # _VG2_V : Inst<"svread_ver_" # n # "_{d}_vg2",   "2im", t, MergeNone, i # "_ver_vg2", [IsInZA, IsStreaming], checks>;
  def NAME # _VG4_H : Inst<"svread_hor_" # n # "_{d}_vg4",   "4im", t, MergeNone, i # "_hor_vg4", [IsInZA, IsStreaming], checks>;
  def NAME # _VG4_V : Inst<"svread_ver_" # n # "_{d}_vg4",   "4im", t, MergeNone, i # "_ver_vg4", [IsInZA, IsStreaming], checks>;
  def NAME # _VG1x2 : Inst<"svread_"     # n # "_{d}_vg1x2", "2m",  t, MergeNone, i # "_vg1x2",   [IsInZA, IsStreaming], []>;
  def NAME # _VG1x4 : Inst<"svread_"     # n # "_{d}_vg1x4", "4m",  t, MergeNone, i # "_vg1x4",   [IsInZA, IsStreaming], []>;
}

let SMETargetGuard = "sme2" in {
  defm SVREAD_ZA8  : ZARead_VG<"za8",  "cUcm",   "aarch64_sme_read", [ImmCheck<0, ImmCheck0_0>]>;
  defm SVREAD_ZA16 : ZARead_VG<"za16", "sUshb", "aarch64_sme_read", [ImmCheck<0, ImmCheck0_1>]>;
  defm SVREAD_ZA32 : ZARead_VG<"za32", "iUif",  "aarch64_sme_read", [ImmCheck<0, ImmCheck0_3>]>;
  defm SVREAD_ZA64 : ZARead_VG<"za64", "lUld",  "aarch64_sme_read", [ImmCheck<0, ImmCheck0_7>]>;
}

//
// Outer product and accumulate/subtract
//

let SMETargetGuard = "sme2" in {
  def SVSMOPA  : Inst<"svmopa_za32[_{d}]_m", "viPPdd", "s", MergeNone, "aarch64_sme_smopa_za32", [IsInOutZA, IsStreaming], [ImmCheck<0, ImmCheck0_3>]>;
  def SVUSMOPA : Inst<"svmopa_za32[_{d}]_m", "viPPdd", "Us", MergeNone, "aarch64_sme_umopa_za32", [IsInOutZA, IsStreaming], [ImmCheck<0, ImmCheck0_3>]>;

  def SVSMOPS  : Inst<"svmops_za32[_{d}]_m", "viPPdd", "s", MergeNone, "aarch64_sme_smops_za32", [IsInOutZA, IsStreaming], [ImmCheck<0, ImmCheck0_3>]>;
  def SVUSMOPS : Inst<"svmops_za32[_{d}]_m", "viPPdd", "Us", MergeNone, "aarch64_sme_umops_za32", [IsInOutZA, IsStreaming], [ImmCheck<0, ImmCheck0_3>]>;
````
- **L457 EN**: Declares TableGen def record `NAME # _VG2_V`.
  **L457 CN**: 声明 TableGen def 记录 `NAME # _VG2_V`。
- **L458 EN**: Declares TableGen def record `NAME # _VG4_H`.
  **L458 CN**: 声明 TableGen def 记录 `NAME # _VG4_H`。
- **L459 EN**: Declares TableGen def record `NAME # _VG4_V`.
  **L459 CN**: 声明 TableGen def 记录 `NAME # _VG4_V`。
- **L460 EN**: Declares TableGen def record `NAME # _VG1x2`.
  **L460 CN**: 声明 TableGen def 记录 `NAME # _VG1x2`。
- **L461 EN**: Declares TableGen def record `NAME # _VG1x4`.
  **L461 CN**: 声明 TableGen def 记录 `NAME # _VG1x4`。
- **L462 EN**: Closes the current lexical scope or compound statement.
  **L462 CN**: 结束当前词法作用域或复合语句块。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L464 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L464 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L465 EN**: Declares TableGen defm record `SVREAD_ZA8`.
  **L465 CN**: 声明 TableGen defm 记录 `SVREAD_ZA8`。
- **L466 EN**: Declares TableGen defm record `SVREAD_ZA16`.
  **L466 CN**: 声明 TableGen defm 记录 `SVREAD_ZA16`。
- **L467 EN**: Declares TableGen defm record `SVREAD_ZA32`.
  **L467 CN**: 声明 TableGen defm 记录 `SVREAD_ZA32`。
- **L468 EN**: Declares TableGen defm record `SVREAD_ZA64`.
  **L468 CN**: 声明 TableGen defm 记录 `SVREAD_ZA64`。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Separator comment used for visual grouping.
  **L471 CN**: 用于视觉分组的分隔注释。
- **L472 EN**: Comment explains nearby logic, constraints, or intent: `Outer product and accumulate/subtract`.
  **L472 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Outer product and accumulate/subtract`。
- **L473 EN**: Separator comment used for visual grouping.
  **L473 CN**: 用于视觉分组的分隔注释。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L475 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L476 EN**: Declares TableGen def record `SVSMOPA`.
  **L476 CN**: 声明 TableGen def 记录 `SVSMOPA`。
- **L477 EN**: Declares TableGen def record `SVUSMOPA`.
  **L477 CN**: 声明 TableGen def 记录 `SVUSMOPA`。
- **L478 EN**: Blank line separating nearby declarations or logic blocks.
  **L478 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L479 EN**: Declares TableGen def record `SVSMOPS`.
  **L479 CN**: 声明 TableGen def 记录 `SVSMOPS`。
- **L480 EN**: Declares TableGen def record `SVUSMOPS`.
  **L480 CN**: 声明 TableGen def 记录 `SVUSMOPS`。

### Lines 481-504

````tablegen

  def SVBMOPA : Inst<"svbmopa_za32[_{d}]_m", "viPPdd", "iUi", MergeNone, "aarch64_sme_bmopa_za32", [IsInOutZA, IsStreaming], [ImmCheck<0, ImmCheck0_3>]>;

  def SVBMOPS : Inst<"svbmops_za32[_{d}]_m", "viPPdd", "iUi", MergeNone, "aarch64_sme_bmops_za32", [IsInOutZA, IsStreaming], [ImmCheck<0, ImmCheck0_3>]>;

  // VERTICAL DOT-PRODUCT
  def SVVDOT_LANE_ZA32_VG1x2_S : Inst<"svvdot_lane_za32[_{d}]_vg1x2", "vm2di", "s", MergeNone, "aarch64_sme_svdot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVVDOT_LANE_ZA32_VG1x4_S : Inst<"svvdot_lane_za32[_{d}]_vg1x4", "vm4di", "c", MergeNone, "aarch64_sme_svdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVVDOT_LANE_ZA32_VG1x2_U : Inst<"svvdot_lane_za32[_{d}]_vg1x2", "vm2di", "Us", MergeNone, "aarch64_sme_uvdot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVVDOT_LANE_ZA32_VG1x4_U : Inst<"svvdot_lane_za32[_{d}]_vg1x4", "vm4di", "Uc", MergeNone, "aarch64_sme_uvdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVVDOT_LANE_ZA32_VG1x2_F : Inst<"svvdot_lane_za32[_{d}]_vg1x2", "vm2di", "hb", MergeNone, "aarch64_sme_fvdot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVSUVDOT_LANE_ZA32_VG1x4 : Inst<"svsuvdot_lane_za32[_{d}]_vg1x4", "vm4di", "c", MergeNone, "aarch64_sme_suvdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVUSVDOT_LANE_ZA32_VG1x4 : Inst<"svusvdot_lane_za32[_{d}]_vg1x4", "vm4di", "Uc", MergeNone, "aarch64_sme_usvdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;

  // Multi-vector signed & unsigned integer dot-product
  def SVDOT_MULTI_ZA32_VG1x2_S  : Inst<"svdot_za32[_{d}]_vg1x2", "vm22", "cs", MergeNone, "aarch64_sme_sdot_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_MULTI_ZA32_VG1x4_S  : Inst<"svdot_za32[_{d}]_vg1x4", "vm44", "cs", MergeNone, "aarch64_sme_sdot_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_MULTI_ZA32_VG1x2_U  : Inst<"svdot_za32[_{d}]_vg1x2", "vm22", "UcUs", MergeNone, "aarch64_sme_udot_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_MULTI_ZA32_VG1x4_U  : Inst<"svdot_za32[_{d}]_vg1x4", "vm44", "UcUs", MergeNone, "aarch64_sme_udot_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA32_VG1x2_S : Inst<"svdot[_single]_za32[_{d}]_vg1x2", "vm2d", "cs", MergeNone, "aarch64_sme_sdot_single_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA32_VG1x4_S : Inst<"svdot[_single]_za32[_{d}]_vg1x4", "vm4d", "cs", MergeNone, "aarch64_sme_sdot_single_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA32_VG1x2_U : Inst<"svdot[_single]_za32[_{d}]_vg1x2", "vm2d", "UcUs", MergeNone, "aarch64_sme_udot_single_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA32_VG1x4_U : Inst<"svdot[_single]_za32[_{d}]_vg1x4", "vm4d", "UcUs", MergeNone, "aarch64_sme_udot_single_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_LANE_ZA32_VG1x2_S   : Inst<"svdot_lane_za32[_{d}]_vg1x2", "vm2di", "cs", MergeNone, "aarch64_sme_sdot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
````
- **L481 EN**: Blank line separating nearby declarations or logic blocks.
  **L481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L482 EN**: Declares TableGen def record `SVBMOPA`.
  **L482 CN**: 声明 TableGen def 记录 `SVBMOPA`。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Declares TableGen def record `SVBMOPS`.
  **L484 CN**: 声明 TableGen def 记录 `SVBMOPS`。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L486 EN**: Comment explains nearby logic, constraints, or intent: `VERTICAL DOT-PRODUCT`.
  **L486 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VERTICAL DOT-PRODUCT`。
- **L487 EN**: Declares TableGen def record `SVVDOT_LANE_ZA32_VG1x2_S`.
  **L487 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_ZA32_VG1x2_S`。
- **L488 EN**: Declares TableGen def record `SVVDOT_LANE_ZA32_VG1x4_S`.
  **L488 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_ZA32_VG1x4_S`。
- **L489 EN**: Declares TableGen def record `SVVDOT_LANE_ZA32_VG1x2_U`.
  **L489 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_ZA32_VG1x2_U`。
- **L490 EN**: Declares TableGen def record `SVVDOT_LANE_ZA32_VG1x4_U`.
  **L490 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_ZA32_VG1x4_U`。
- **L491 EN**: Declares TableGen def record `SVVDOT_LANE_ZA32_VG1x2_F`.
  **L491 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_ZA32_VG1x2_F`。
- **L492 EN**: Declares TableGen def record `SVSUVDOT_LANE_ZA32_VG1x4`.
  **L492 CN**: 声明 TableGen def 记录 `SVSUVDOT_LANE_ZA32_VG1x4`。
- **L493 EN**: Declares TableGen def record `SVUSVDOT_LANE_ZA32_VG1x4`.
  **L493 CN**: 声明 TableGen def 记录 `SVUSVDOT_LANE_ZA32_VG1x4`。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L495 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector signed & unsigned integer dot-product`.
  **L495 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector signed & unsigned integer dot-product`。
- **L496 EN**: Declares TableGen def record `SVDOT_MULTI_ZA32_VG1x2_S`.
  **L496 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA32_VG1x2_S`。
- **L497 EN**: Declares TableGen def record `SVDOT_MULTI_ZA32_VG1x4_S`.
  **L497 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA32_VG1x4_S`。
- **L498 EN**: Declares TableGen def record `SVDOT_MULTI_ZA32_VG1x2_U`.
  **L498 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA32_VG1x2_U`。
- **L499 EN**: Declares TableGen def record `SVDOT_MULTI_ZA32_VG1x4_U`.
  **L499 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA32_VG1x4_U`。
- **L500 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA32_VG1x2_S`.
  **L500 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA32_VG1x2_S`。
- **L501 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA32_VG1x4_S`.
  **L501 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA32_VG1x4_S`。
- **L502 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA32_VG1x2_U`.
  **L502 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA32_VG1x2_U`。
- **L503 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA32_VG1x4_U`.
  **L503 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA32_VG1x4_U`。
- **L504 EN**: Declares TableGen def record `SVDOT_LANE_ZA32_VG1x2_S`.
  **L504 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA32_VG1x2_S`。

### Lines 505-528

````tablegen
  def SVDOT_LANE_ZA32_VG1x4_S   : Inst<"svdot_lane_za32[_{d}]_vg1x4", "vm4di", "cs", MergeNone, "aarch64_sme_sdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVDOT_LANE_ZA32_VG1x2_U   : Inst<"svdot_lane_za32[_{d}]_vg1x2", "vm2di", "UcUs", MergeNone, "aarch64_sme_udot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVDOT_LANE_ZA32_VG1x4_U   : Inst<"svdot_lane_za32[_{d}]_vg1x4", "vm4di", "UcUs", MergeNone, "aarch64_sme_udot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;

  def SVUSDOT_SINGLE_ZA32_VG1x2 : Inst<"svusdot[_single]_za32[_{d}]_vg1x2", "vm2.dx", "Uc", MergeNone, "aarch64_sme_usdot_single_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVUSDOT_SINGLE_ZA32_VG1x4 : Inst<"svusdot[_single]_za32[_{d}]_vg1x4", "vm4.dx", "Uc", MergeNone, "aarch64_sme_usdot_single_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVUSDOT_MULTI_ZA32_VG1x2  : Inst<"svusdot_za32[_{d}]_vg1x2", "vm2.d2.x", "Uc", MergeNone, "aarch64_sme_usdot_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVUSDOT_MULTI_ZA32_VG1x4  : Inst<"svusdot_za32[_{d}]_vg1x4", "vm4.d4.x", "Uc", MergeNone, "aarch64_sme_usdot_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVUSDOT_LANE_ZA32_VG1x2   : Inst<"svusdot_lane_za32[_{d}]_vg1x2", "vm2.dxi", "Uc", MergeNone, "aarch64_sme_usdot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVUSDOT_LANE_ZA32_VG1x4   : Inst<"svusdot_lane_za32[_{d}]_vg1x4", "vm4.dxi", "Uc", MergeNone, "aarch64_sme_usdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;

  def SVSUDOT_SINGLE_ZA32_VG1x2 : Inst<"svsudot[_single]_za32[_{d}]_vg1x2", "vm2.du", "c", MergeNone, "aarch64_sme_sudot_single_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVSUDOT_SINGLE_ZA32_VG1x4 : Inst<"svsudot[_single]_za32[_{d}]_vg1x4", "vm4.du", "c", MergeNone, "aarch64_sme_sudot_single_za32_vg1x4", [IsStreaming, IsInOutZA], []>;

  // Multi-multi sudot builtins are mapped to usdot, with zn & zm operands swapped
  def SVSUDOT_MULTI_ZA32_VG1x2  : Inst<"svsudot_za32[_{d}]_vg1x2", "vm2.d2.u", "c", MergeNone, "aarch64_sme_usdot_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVSUDOT_MULTI_ZA32_VG1x4  : Inst<"svsudot_za32[_{d}]_vg1x4", "vm4.d4.u", "c", MergeNone, "aarch64_sme_usdot_za32_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVSUDOT_LANE_ZA32_VG1x2   : Inst<"svsudot_lane_za32[_{d}]_vg1x2", "vm2.dui", "c", MergeNone, "aarch64_sme_sudot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVSUDOT_LANE_ZA32_VG1x4   : Inst<"svsudot_lane_za32[_{d}]_vg1x4", "vm4.dui", "c", MergeNone, "aarch64_sme_sudot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;

  // Multi-vector half-precision/BFloat16 floating-point dot-product
  def SVDOT_MULTI_ZA32_VG1x2_F16  : Inst<"svdot_za32[_{d}]_vg1x2", "vm22", "bh", MergeNone, "aarch64_sme_fdot_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_MULTI_ZA32_VG1x4_F16  : Inst<"svdot_za32[_{d}]_vg1x4", "vm44", "bh", MergeNone, "aarch64_sme_fdot_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
````
- **L505 EN**: Declares TableGen def record `SVDOT_LANE_ZA32_VG1x4_S`.
  **L505 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA32_VG1x4_S`。
- **L506 EN**: Declares TableGen def record `SVDOT_LANE_ZA32_VG1x2_U`.
  **L506 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA32_VG1x2_U`。
- **L507 EN**: Declares TableGen def record `SVDOT_LANE_ZA32_VG1x4_U`.
  **L507 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA32_VG1x4_U`。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L509 EN**: Declares TableGen def record `SVUSDOT_SINGLE_ZA32_VG1x2`.
  **L509 CN**: 声明 TableGen def 记录 `SVUSDOT_SINGLE_ZA32_VG1x2`。
- **L510 EN**: Declares TableGen def record `SVUSDOT_SINGLE_ZA32_VG1x4`.
  **L510 CN**: 声明 TableGen def 记录 `SVUSDOT_SINGLE_ZA32_VG1x4`。
- **L511 EN**: Declares TableGen def record `SVUSDOT_MULTI_ZA32_VG1x2`.
  **L511 CN**: 声明 TableGen def 记录 `SVUSDOT_MULTI_ZA32_VG1x2`。
- **L512 EN**: Declares TableGen def record `SVUSDOT_MULTI_ZA32_VG1x4`.
  **L512 CN**: 声明 TableGen def 记录 `SVUSDOT_MULTI_ZA32_VG1x4`。
- **L513 EN**: Declares TableGen def record `SVUSDOT_LANE_ZA32_VG1x2`.
  **L513 CN**: 声明 TableGen def 记录 `SVUSDOT_LANE_ZA32_VG1x2`。
- **L514 EN**: Declares TableGen def record `SVUSDOT_LANE_ZA32_VG1x4`.
  **L514 CN**: 声明 TableGen def 记录 `SVUSDOT_LANE_ZA32_VG1x4`。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L516 EN**: Declares TableGen def record `SVSUDOT_SINGLE_ZA32_VG1x2`.
  **L516 CN**: 声明 TableGen def 记录 `SVSUDOT_SINGLE_ZA32_VG1x2`。
- **L517 EN**: Declares TableGen def record `SVSUDOT_SINGLE_ZA32_VG1x4`.
  **L517 CN**: 声明 TableGen def 记录 `SVSUDOT_SINGLE_ZA32_VG1x4`。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Comment explains nearby logic, constraints, or intent: `Multi-multi sudot builtins are mapped to usdot, with zn & zm operands swapped`.
  **L519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-multi sudot builtins are mapped to usdot, with zn & zm operands swapped`。
- **L520 EN**: Declares TableGen def record `SVSUDOT_MULTI_ZA32_VG1x2`.
  **L520 CN**: 声明 TableGen def 记录 `SVSUDOT_MULTI_ZA32_VG1x2`。
- **L521 EN**: Declares TableGen def record `SVSUDOT_MULTI_ZA32_VG1x4`.
  **L521 CN**: 声明 TableGen def 记录 `SVSUDOT_MULTI_ZA32_VG1x4`。
- **L522 EN**: Blank line separating nearby declarations or logic blocks.
  **L522 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L523 EN**: Declares TableGen def record `SVSUDOT_LANE_ZA32_VG1x2`.
  **L523 CN**: 声明 TableGen def 记录 `SVSUDOT_LANE_ZA32_VG1x2`。
- **L524 EN**: Declares TableGen def record `SVSUDOT_LANE_ZA32_VG1x4`.
  **L524 CN**: 声明 TableGen def 记录 `SVSUDOT_LANE_ZA32_VG1x4`。
- **L525 EN**: Blank line separating nearby declarations or logic blocks.
  **L525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L526 EN**: Comment explains nearby logic, constraints, or intent: `Multi-vector half-precision/BFloat16 floating-point dot-product`.
  **L526 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-vector half-precision/BFloat16 floating-point dot-product`。
- **L527 EN**: Declares TableGen def record `SVDOT_MULTI_ZA32_VG1x2_F16`.
  **L527 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA32_VG1x2_F16`。
- **L528 EN**: Declares TableGen def record `SVDOT_MULTI_ZA32_VG1x4_F16`.
  **L528 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA32_VG1x4_F16`。

### Lines 529-552

````tablegen
  def SVDOT_SINGLE_ZA32_VG1x2_F16 : Inst<"svdot[_single]_za32[_{d}]_vg1x2", "vm2d", "bh", MergeNone, "aarch64_sme_fdot_single_za32_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA32_VG1x4_F16 : Inst<"svdot[_single]_za32[_{d}]_vg1x4", "vm4d", "bh", MergeNone, "aarch64_sme_fdot_single_za32_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_LANE_ZA32_VG1x2_F16   : Inst<"svdot_lane_za32[_{d}]_vg1x2", "vm2di", "bh", MergeNone, "aarch64_sme_fdot_lane_za32_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVDOT_LANE_ZA32_VG1x4_F16   : Inst<"svdot_lane_za32[_{d}]_vg1x4", "vm4di", "bh", MergeNone, "aarch64_sme_fdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-i16i64" in {
  def SVVDOT_LANE_ZA64_VG1x4_S : Inst<"svvdot_lane_za64[_{d}]_vg1x4", "vm4di", "s", MergeNone, "aarch64_sme_svdot_lane_za64_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
  def SVVDOT_LANE_ZA64_VG1x4_U : Inst<"svvdot_lane_za64[_{d}]_vg1x4", "vm4di", "Us", MergeNone, "aarch64_sme_uvdot_lane_za64_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;

  def SVDOT_MULTI_ZA64_VG1x2_S16  : Inst<"svdot_za64[_{d}]_vg1x2", "vm22", "s", MergeNone, "aarch64_sme_sdot_za64_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_MULTI_ZA64_VG1x4_S16  : Inst<"svdot_za64[_{d}]_vg1x4", "vm44", "s", MergeNone, "aarch64_sme_sdot_za64_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_MULTI_ZA64_VG1x2_U16  : Inst<"svdot_za64[_{d}]_vg1x2", "vm22", "Us", MergeNone, "aarch64_sme_udot_za64_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_MULTI_ZA64_VG1x4_U16  : Inst<"svdot_za64[_{d}]_vg1x4", "vm44", "Us", MergeNone, "aarch64_sme_udot_za64_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA64_VG1x2_S16 : Inst<"svdot[_single]_za64[_{d}]_vg1x2", "vm2d", "s", MergeNone, "aarch64_sme_sdot_single_za64_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA64_VG1x4_S16 : Inst<"svdot[_single]_za64[_{d}]_vg1x4", "vm4d", "s", MergeNone, "aarch64_sme_sdot_single_za64_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA64_VG1x2_U16 : Inst<"svdot[_single]_za64[_{d}]_vg1x2", "vm2d", "Us", MergeNone, "aarch64_sme_udot_single_za64_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVDOT_SINGLE_ZA64_VG1x4_U16 : Inst<"svdot[_single]_za64[_{d}]_vg1x4", "vm4d", "Us", MergeNone, "aarch64_sme_udot_single_za64_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVDOT_LANE_ZA64_VG1x2_S16   : Inst<"svdot_lane_za64[_{d}]_vg1x2", "vm2di", "s", MergeNone, "aarch64_sme_sdot_lane_za64_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
  def SVDOT_LANE_ZA64_VG1x4_S16   : Inst<"svdot_lane_za64[_{d}]_vg1x4", "vm4di", "s", MergeNone, "aarch64_sme_sdot_lane_za64_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
  def SVDOT_LANE_ZA64_VG1x2_U16   : Inst<"svdot_lane_za64[_{d}]_vg1x2", "vm2di", "Us", MergeNone, "aarch64_sme_udot_lane_za64_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
  def SVDOT_LANE_ZA64_VG1x4_U16   : Inst<"svdot_lane_za64[_{d}]_vg1x4", "vm4di", "Us", MergeNone, "aarch64_sme_udot_lane_za64_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
}

````
- **L529 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA32_VG1x2_F16`.
  **L529 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA32_VG1x2_F16`。
- **L530 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA32_VG1x4_F16`.
  **L530 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA32_VG1x4_F16`。
- **L531 EN**: Declares TableGen def record `SVDOT_LANE_ZA32_VG1x2_F16`.
  **L531 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA32_VG1x2_F16`。
- **L532 EN**: Declares TableGen def record `SVDOT_LANE_ZA32_VG1x4_F16`.
  **L532 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA32_VG1x4_F16`。
- **L533 EN**: Closes the current lexical scope or compound statement.
  **L533 CN**: 结束当前词法作用域或复合语句块。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L535 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-i16i64" in {`.
  **L535 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-i16i64" in {`。
- **L536 EN**: Declares TableGen def record `SVVDOT_LANE_ZA64_VG1x4_S`.
  **L536 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_ZA64_VG1x4_S`。
- **L537 EN**: Declares TableGen def record `SVVDOT_LANE_ZA64_VG1x4_U`.
  **L537 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_ZA64_VG1x4_U`。
- **L538 EN**: Blank line separating nearby declarations or logic blocks.
  **L538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L539 EN**: Declares TableGen def record `SVDOT_MULTI_ZA64_VG1x2_S16`.
  **L539 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA64_VG1x2_S16`。
- **L540 EN**: Declares TableGen def record `SVDOT_MULTI_ZA64_VG1x4_S16`.
  **L540 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA64_VG1x4_S16`。
- **L541 EN**: Declares TableGen def record `SVDOT_MULTI_ZA64_VG1x2_U16`.
  **L541 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA64_VG1x2_U16`。
- **L542 EN**: Declares TableGen def record `SVDOT_MULTI_ZA64_VG1x4_U16`.
  **L542 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_ZA64_VG1x4_U16`。
- **L543 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA64_VG1x2_S16`.
  **L543 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA64_VG1x2_S16`。
- **L544 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA64_VG1x4_S16`.
  **L544 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA64_VG1x4_S16`。
- **L545 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA64_VG1x2_U16`.
  **L545 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA64_VG1x2_U16`。
- **L546 EN**: Declares TableGen def record `SVDOT_SINGLE_ZA64_VG1x4_U16`.
  **L546 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_ZA64_VG1x4_U16`。
- **L547 EN**: Declares TableGen def record `SVDOT_LANE_ZA64_VG1x2_S16`.
  **L547 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA64_VG1x2_S16`。
- **L548 EN**: Declares TableGen def record `SVDOT_LANE_ZA64_VG1x4_S16`.
  **L548 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA64_VG1x4_S16`。
- **L549 EN**: Declares TableGen def record `SVDOT_LANE_ZA64_VG1x2_U16`.
  **L549 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA64_VG1x2_U16`。
- **L550 EN**: Declares TableGen def record `SVDOT_LANE_ZA64_VG1x4_U16`.
  **L550 CN**: 声明 TableGen def 记录 `SVDOT_LANE_ZA64_VG1x4_U16`。
- **L551 EN**: Closes the current lexical scope or compound statement.
  **L551 CN**: 结束当前词法作用域或复合语句块。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 553-576

````tablegen
// FMLA/FMLS
let SMETargetGuard = "sme2" in {
  def SVMLA_MULTI_VG1x2_F32 : Inst<"svmla_za32[_{d}]_vg1x2", "vm22", "f", MergeNone, "aarch64_sme_fmla_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_MULTI_VG1x4_F32 : Inst<"svmla_za32[_{d}]_vg1x4", "vm44", "f", MergeNone, "aarch64_sme_fmla_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x2_F32 : Inst<"svmls_za32[_{d}]_vg1x2", "vm22", "f", MergeNone, "aarch64_sme_fmls_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x4_F32 : Inst<"svmls_za32[_{d}]_vg1x4", "vm44", "f", MergeNone, "aarch64_sme_fmls_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVMLA_SINGLE_VG1x2_F32 : Inst<"svmla[_single]_za32[_{d}]_vg1x2", "vm2d", "f", MergeNone, "aarch64_sme_fmla_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_SINGLE_VG1x4_F32 : Inst<"svmla[_single]_za32[_{d}]_vg1x4", "vm4d", "f", MergeNone, "aarch64_sme_fmla_single_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x2_F32 : Inst<"svmls[_single]_za32[_{d}]_vg1x2", "vm2d", "f", MergeNone, "aarch64_sme_fmls_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x4_F32 : Inst<"svmls[_single]_za32[_{d}]_vg1x4", "vm4d", "f", MergeNone, "aarch64_sme_fmls_single_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVMLA_LANE_VG1x2_F32 : Inst<"svmla_lane_za32[_{d}]_vg1x2", "vm2di", "f", MergeNone, "aarch64_sme_fmla_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVMLA_LANE_VG1x4_F32 : Inst<"svmla_lane_za32[_{d}]_vg1x4", "vm4di", "f", MergeNone, "aarch64_sme_fmla_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVMLS_LANE_VG1x2_F32 : Inst<"svmls_lane_za32[_{d}]_vg1x2", "vm2di", "f", MergeNone, "aarch64_sme_fmls_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVMLS_LANE_VG1x4_F32 : Inst<"svmls_lane_za32[_{d}]_vg1x4", "vm4di", "f", MergeNone, "aarch64_sme_fmls_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-f64f64" in {
  def SVMLA_MULTI_VG1x2_F64 : Inst<"svmla_za64[_{d}]_vg1x2", "vm22", "d", MergeNone, "aarch64_sme_fmla_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_MULTI_VG1x4_F64 : Inst<"svmla_za64[_{d}]_vg1x4", "vm44", "d", MergeNone, "aarch64_sme_fmla_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x2_F64 : Inst<"svmls_za64[_{d}]_vg1x2", "vm22", "d", MergeNone, "aarch64_sme_fmls_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x4_F64 : Inst<"svmls_za64[_{d}]_vg1x4", "vm44", "d", MergeNone, "aarch64_sme_fmls_vg1x4", [IsStreaming, IsInOutZA], []>;

````
- **L553 EN**: Comment explains nearby logic, constraints, or intent: `FMLA/FMLS`.
  **L553 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLA/FMLS`。
- **L554 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L554 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L555 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x2_F32`.
  **L555 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x2_F32`。
- **L556 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x4_F32`.
  **L556 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x4_F32`。
- **L557 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x2_F32`.
  **L557 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x2_F32`。
- **L558 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x4_F32`.
  **L558 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x4_F32`。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L560 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x2_F32`.
  **L560 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x2_F32`。
- **L561 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x4_F32`.
  **L561 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x4_F32`。
- **L562 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x2_F32`.
  **L562 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x2_F32`。
- **L563 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x4_F32`.
  **L563 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x4_F32`。
- **L564 EN**: Blank line separating nearby declarations or logic blocks.
  **L564 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L565 EN**: Declares TableGen def record `SVMLA_LANE_VG1x2_F32`.
  **L565 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x2_F32`。
- **L566 EN**: Declares TableGen def record `SVMLA_LANE_VG1x4_F32`.
  **L566 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x4_F32`。
- **L567 EN**: Declares TableGen def record `SVMLS_LANE_VG1x2_F32`.
  **L567 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x2_F32`。
- **L568 EN**: Declares TableGen def record `SVMLS_LANE_VG1x4_F32`.
  **L568 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x4_F32`。
- **L569 EN**: Closes the current lexical scope or compound statement.
  **L569 CN**: 结束当前词法作用域或复合语句块。
- **L570 EN**: Blank line separating nearby declarations or logic blocks.
  **L570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L571 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-f64f64" in {`.
  **L571 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-f64f64" in {`。
- **L572 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x2_F64`.
  **L572 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x2_F64`。
- **L573 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x4_F64`.
  **L573 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x4_F64`。
- **L574 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x2_F64`.
  **L574 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x2_F64`。
- **L575 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x4_F64`.
  **L575 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x4_F64`。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 577-600

````tablegen
  def SVMLA_SINGLE_VG1x2_F64 : Inst<"svmla[_single]_za64[_{d}]_vg1x2", "vm2d", "d", MergeNone, "aarch64_sme_fmla_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_SINGLE_VG1x4_F64 : Inst<"svmla[_single]_za64[_{d}]_vg1x4", "vm4d", "d", MergeNone, "aarch64_sme_fmla_single_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x2_F64 : Inst<"svmls[_single]_za64[_{d}]_vg1x2", "vm2d", "d", MergeNone, "aarch64_sme_fmls_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x4_F64 : Inst<"svmls[_single]_za64[_{d}]_vg1x4", "vm4d", "d", MergeNone, "aarch64_sme_fmls_single_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVMLA_LANE_VG1x2_F64 : Inst<"svmla_lane_za64[_{d}]_vg1x2", "vm2di", "d", MergeNone, "aarch64_sme_fmla_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
  def SVMLA_LANE_VG1x4_F64 : Inst<"svmla_lane_za64[_{d}]_vg1x4", "vm4di", "d", MergeNone, "aarch64_sme_fmla_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
  def SVMLS_LANE_VG1x2_F64 : Inst<"svmls_lane_za64[_{d}]_vg1x2", "vm2di", "d", MergeNone, "aarch64_sme_fmls_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
  def SVMLS_LANE_VG1x4_F64 : Inst<"svmls_lane_za64[_{d}]_vg1x4", "vm4di", "d", MergeNone, "aarch64_sme_fmls_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_1>]>;
}

let SMETargetGuard = "sme-f16f16" in {
  def SVMLA_MULTI_VG1x2_F16 : Inst<"svmla_za16[_f16]_vg1x2", "vm22", "h", MergeNone, "aarch64_sme_fmla_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_MULTI_VG1x4_F16 : Inst<"svmla_za16[_f16]_vg1x4", "vm44", "h", MergeNone, "aarch64_sme_fmla_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x2_F16 : Inst<"svmls_za16[_f16]_vg1x2", "vm22", "h", MergeNone, "aarch64_sme_fmls_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x4_F16 : Inst<"svmls_za16[_f16]_vg1x4", "vm44", "h", MergeNone, "aarch64_sme_fmls_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVMLA_SINGLE_VG1x2_F16 : Inst<"svmla[_single]_za16[_f16]_vg1x2", "vm2d", "h", MergeNone, "aarch64_sme_fmla_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_SINGLE_VG1x4_F16 : Inst<"svmla[_single]_za16[_f16]_vg1x4", "vm4d", "h", MergeNone, "aarch64_sme_fmla_single_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x2_F16 : Inst<"svmls[_single]_za16[_f16]_vg1x2", "vm2d", "h", MergeNone, "aarch64_sme_fmls_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x4_F16 : Inst<"svmls[_single]_za16[_f16]_vg1x4", "vm4d", "h", MergeNone, "aarch64_sme_fmls_single_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVMLA_LANE_VG1x2_F16 : Inst<"svmla_lane_za16[_f16]_vg1x2", "vm2di", "h", MergeNone, "aarch64_sme_fmla_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLA_LANE_VG1x4_F16 : Inst<"svmla_lane_za16[_f16]_vg1x4", "vm4di", "h", MergeNone, "aarch64_sme_fmla_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
````
- **L577 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x2_F64`.
  **L577 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x2_F64`。
- **L578 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x4_F64`.
  **L578 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x4_F64`。
- **L579 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x2_F64`.
  **L579 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x2_F64`。
- **L580 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x4_F64`.
  **L580 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x4_F64`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Declares TableGen def record `SVMLA_LANE_VG1x2_F64`.
  **L582 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x2_F64`。
- **L583 EN**: Declares TableGen def record `SVMLA_LANE_VG1x4_F64`.
  **L583 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x4_F64`。
- **L584 EN**: Declares TableGen def record `SVMLS_LANE_VG1x2_F64`.
  **L584 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x2_F64`。
- **L585 EN**: Declares TableGen def record `SVMLS_LANE_VG1x4_F64`.
  **L585 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x4_F64`。
- **L586 EN**: Closes the current lexical scope or compound statement.
  **L586 CN**: 结束当前词法作用域或复合语句块。
- **L587 EN**: Blank line separating nearby declarations or logic blocks.
  **L587 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L588 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f16f16" in {`.
  **L588 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f16f16" in {`。
- **L589 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x2_F16`.
  **L589 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x2_F16`。
- **L590 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x4_F16`.
  **L590 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x4_F16`。
- **L591 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x2_F16`.
  **L591 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x2_F16`。
- **L592 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x4_F16`.
  **L592 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x4_F16`。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x2_F16`.
  **L594 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x2_F16`。
- **L595 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x4_F16`.
  **L595 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x4_F16`。
- **L596 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x2_F16`.
  **L596 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x2_F16`。
- **L597 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x4_F16`.
  **L597 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x4_F16`。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L599 EN**: Declares TableGen def record `SVMLA_LANE_VG1x2_F16`.
  **L599 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x2_F16`。
- **L600 EN**: Declares TableGen def record `SVMLA_LANE_VG1x4_F16`.
  **L600 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x4_F16`。

### Lines 601-624

````tablegen
  def SVMLS_LANE_VG1x2_F16 : Inst<"svmls_lane_za16[_f16]_vg1x2", "vm2di", "h", MergeNone, "aarch64_sme_fmls_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLS_LANE_VG1x4_F16 : Inst<"svmls_lane_za16[_f16]_vg1x4", "vm4di", "h", MergeNone, "aarch64_sme_fmls_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
}

let SMETargetGuard = "sme-b16b16" in {
  def SVMLA_MULTI_VG1x2_BF16 : Inst<"svmla_za16[_bf16]_vg1x2", "vm22", "b", MergeNone, "aarch64_sme_fmla_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_MULTI_VG1x4_BF16 : Inst<"svmla_za16[_bf16]_vg1x4", "vm44", "b", MergeNone, "aarch64_sme_fmla_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x2_BF16 : Inst<"svmls_za16[_bf16]_vg1x2", "vm22", "b", MergeNone, "aarch64_sme_fmls_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_MULTI_VG1x4_BF16 : Inst<"svmls_za16[_bf16]_vg1x4", "vm44", "b", MergeNone, "aarch64_sme_fmls_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVMLA_SINGLE_VG1x2_BF16 : Inst<"svmla[_single]_za16[_bf16]_vg1x2", "vm2d", "b", MergeNone, "aarch64_sme_fmla_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLA_SINGLE_VG1x4_BF16 : Inst<"svmla[_single]_za16[_bf16]_vg1x4", "vm4d", "b", MergeNone, "aarch64_sme_fmla_single_vg1x4", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x2_BF16 : Inst<"svmls[_single]_za16[_bf16]_vg1x2", "vm2d", "b", MergeNone, "aarch64_sme_fmls_single_vg1x2", [IsStreaming, IsInOutZA], []>;
  def SVMLS_SINGLE_VG1x4_BF16 : Inst<"svmls[_single]_za16[_bf16]_vg1x4", "vm4d", "b", MergeNone, "aarch64_sme_fmls_single_vg1x4", [IsStreaming, IsInOutZA], []>;

  def SVMLA_LANE_VG1x2_BF16 : Inst<"svmla_lane_za16[_bf16]_vg1x2", "vm2di", "b", MergeNone, "aarch64_sme_fmla_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLA_LANE_VG1x4_BF16 : Inst<"svmla_lane_za16[_bf16]_vg1x4", "vm4di", "b", MergeNone, "aarch64_sme_fmla_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLS_LANE_VG1x2_BF16 : Inst<"svmls_lane_za16[_bf16]_vg1x2", "vm2di", "b", MergeNone, "aarch64_sme_fmls_lane_vg1x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLS_LANE_VG1x4_BF16 : Inst<"svmls_lane_za16[_bf16]_vg1x4", "vm4di", "b", MergeNone, "aarch64_sme_fmls_lane_vg1x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
}

// FMLAL/FMLSL/UMLAL/SMLAL
// SMLALL/UMLALL/USMLALL/SUMLALL
let SMETargetGuard = "sme2" in {
````
- **L601 EN**: Declares TableGen def record `SVMLS_LANE_VG1x2_F16`.
  **L601 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x2_F16`。
- **L602 EN**: Declares TableGen def record `SVMLS_LANE_VG1x4_F16`.
  **L602 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x4_F16`。
- **L603 EN**: Closes the current lexical scope or compound statement.
  **L603 CN**: 结束当前词法作用域或复合语句块。
- **L604 EN**: Blank line separating nearby declarations or logic blocks.
  **L604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L605 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-b16b16" in {`.
  **L605 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-b16b16" in {`。
- **L606 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x2_BF16`.
  **L606 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x2_BF16`。
- **L607 EN**: Declares TableGen def record `SVMLA_MULTI_VG1x4_BF16`.
  **L607 CN**: 声明 TableGen def 记录 `SVMLA_MULTI_VG1x4_BF16`。
- **L608 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x2_BF16`.
  **L608 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x2_BF16`。
- **L609 EN**: Declares TableGen def record `SVMLS_MULTI_VG1x4_BF16`.
  **L609 CN**: 声明 TableGen def 记录 `SVMLS_MULTI_VG1x4_BF16`。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L611 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x2_BF16`.
  **L611 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x2_BF16`。
- **L612 EN**: Declares TableGen def record `SVMLA_SINGLE_VG1x4_BF16`.
  **L612 CN**: 声明 TableGen def 记录 `SVMLA_SINGLE_VG1x4_BF16`。
- **L613 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x2_BF16`.
  **L613 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x2_BF16`。
- **L614 EN**: Declares TableGen def record `SVMLS_SINGLE_VG1x4_BF16`.
  **L614 CN**: 声明 TableGen def 记录 `SVMLS_SINGLE_VG1x4_BF16`。
- **L615 EN**: Blank line separating nearby declarations or logic blocks.
  **L615 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L616 EN**: Declares TableGen def record `SVMLA_LANE_VG1x2_BF16`.
  **L616 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x2_BF16`。
- **L617 EN**: Declares TableGen def record `SVMLA_LANE_VG1x4_BF16`.
  **L617 CN**: 声明 TableGen def 记录 `SVMLA_LANE_VG1x4_BF16`。
- **L618 EN**: Declares TableGen def record `SVMLS_LANE_VG1x2_BF16`.
  **L618 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x2_BF16`。
- **L619 EN**: Declares TableGen def record `SVMLS_LANE_VG1x4_BF16`.
  **L619 CN**: 声明 TableGen def 记录 `SVMLS_LANE_VG1x4_BF16`。
- **L620 EN**: Closes the current lexical scope or compound statement.
  **L620 CN**: 结束当前词法作用域或复合语句块。
- **L621 EN**: Blank line separating nearby declarations or logic blocks.
  **L621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L622 EN**: Comment explains nearby logic, constraints, or intent: `FMLAL/FMLSL/UMLAL/SMLAL`.
  **L622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLAL/FMLSL/UMLAL/SMLAL`。
- **L623 EN**: Comment explains nearby logic, constraints, or intent: `SMLALL/UMLALL/USMLALL/SUMLALL`.
  **L623 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SMLALL/UMLALL/USMLALL/SUMLALL`。
- **L624 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L624 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。

### Lines 625-648

````tablegen
  // MULTI MLAL
  def SVMLAL_MULTI_VG2x2_F16 : Inst<"svmla_za32[_{d}]_vg2x2", "vm22", "bh", MergeNone, "aarch64_sme_fmlal_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG2x4_F16 : Inst<"svmla_za32[_{d}]_vg2x4", "vm44", "bh", MergeNone, "aarch64_sme_fmlal_vg2x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG2x2_S16 : Inst<"svmla_za32[_{d}]_vg2x2", "vm22", "s", MergeNone, "aarch64_sme_smlal_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG2x4_S16 : Inst<"svmla_za32[_{d}]_vg2x4", "vm44", "s", MergeNone, "aarch64_sme_smlal_vg2x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG2x2_U16 : Inst<"svmla_za32[_{d}]_vg2x2", "vm22", "Us", MergeNone, "aarch64_sme_umlal_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG2x4_U16 : Inst<"svmla_za32[_{d}]_vg2x4", "vm44", "Us", MergeNone, "aarch64_sme_umlal_vg2x4", [IsStreaming, IsInOutZA], []>;

  def SVMLAL_MULTI_VG4x2_S8 : Inst<"svmla_za32[_{d}]_vg4x2", "vm22", "c", MergeNone, "aarch64_sme_smla_za32_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG4x2_U8 : Inst<"svmla_za32[_{d}]_vg4x2", "vm22", "Uc", MergeNone, "aarch64_sme_umla_za32_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG4x4_S8 : Inst<"svmla_za32[_{d}]_vg4x4", "vm44", "c", MergeNone, "aarch64_sme_smla_za32_vg4x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG4x4_U8 : Inst<"svmla_za32[_{d}]_vg4x4", "vm44", "Uc", MergeNone, "aarch64_sme_umla_za32_vg4x4", [IsStreaming, IsInOutZA], []>;

  // MULTI MLSL
  def SVMLSL_MULTI_VG2x2_F16 : Inst<"svmls_za32[_{d}]_vg2x2", "vm22", "bh", MergeNone, "aarch64_sme_fmlsl_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG2x4_F16 : Inst<"svmls_za32[_{d}]_vg2x4", "vm44", "bh", MergeNone, "aarch64_sme_fmlsl_vg2x4", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG2x2_S16 : Inst<"svmls_za32[_{d}]_vg2x2", "vm22", "s", MergeNone, "aarch64_sme_smlsl_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG2x4_S16 : Inst<"svmls_za32[_{d}]_vg2x4", "vm44", "s", MergeNone, "aarch64_sme_smlsl_vg2x4", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG2x2_U16 : Inst<"svmls_za32[_{d}]_vg2x2", "vm22", "Us", MergeNone, "aarch64_sme_umlsl_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG2x4_U16 : Inst<"svmls_za32[_{d}]_vg2x4", "vm44", "Us", MergeNone, "aarch64_sme_umlsl_vg2x4", [IsStreaming, IsInOutZA], []>;

  def SVMLSL_MULTI_VG4x2_S8 : Inst<"svmls_za32[_{d}]_vg4x2", "vm22", "c", MergeNone, "aarch64_sme_smls_za32_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG4x2_U8 : Inst<"svmls_za32[_{d}]_vg4x2", "vm22", "Uc", MergeNone, "aarch64_sme_umls_za32_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG4x4_S8 : Inst<"svmls_za32[_{d}]_vg4x4", "vm44", "c", MergeNone, "aarch64_sme_smls_za32_vg4x4", [IsStreaming, IsInOutZA], []>;
````
- **L625 EN**: Comment explains nearby logic, constraints, or intent: `MULTI MLAL`.
  **L625 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MULTI MLAL`。
- **L626 EN**: Declares TableGen def record `SVMLAL_MULTI_VG2x2_F16`.
  **L626 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG2x2_F16`。
- **L627 EN**: Declares TableGen def record `SVMLAL_MULTI_VG2x4_F16`.
  **L627 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG2x4_F16`。
- **L628 EN**: Declares TableGen def record `SVMLAL_MULTI_VG2x2_S16`.
  **L628 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG2x2_S16`。
- **L629 EN**: Declares TableGen def record `SVMLAL_MULTI_VG2x4_S16`.
  **L629 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG2x4_S16`。
- **L630 EN**: Declares TableGen def record `SVMLAL_MULTI_VG2x2_U16`.
  **L630 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG2x2_U16`。
- **L631 EN**: Declares TableGen def record `SVMLAL_MULTI_VG2x4_U16`.
  **L631 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG2x4_U16`。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x2_S8`.
  **L633 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x2_S8`。
- **L634 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x2_U8`.
  **L634 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x2_U8`。
- **L635 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x4_S8`.
  **L635 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x4_S8`。
- **L636 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x4_U8`.
  **L636 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x4_U8`。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L638 EN**: Comment explains nearby logic, constraints, or intent: `MULTI MLSL`.
  **L638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MULTI MLSL`。
- **L639 EN**: Declares TableGen def record `SVMLSL_MULTI_VG2x2_F16`.
  **L639 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG2x2_F16`。
- **L640 EN**: Declares TableGen def record `SVMLSL_MULTI_VG2x4_F16`.
  **L640 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG2x4_F16`。
- **L641 EN**: Declares TableGen def record `SVMLSL_MULTI_VG2x2_S16`.
  **L641 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG2x2_S16`。
- **L642 EN**: Declares TableGen def record `SVMLSL_MULTI_VG2x4_S16`.
  **L642 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG2x4_S16`。
- **L643 EN**: Declares TableGen def record `SVMLSL_MULTI_VG2x2_U16`.
  **L643 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG2x2_U16`。
- **L644 EN**: Declares TableGen def record `SVMLSL_MULTI_VG2x4_U16`.
  **L644 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG2x4_U16`。
- **L645 EN**: Blank line separating nearby declarations or logic blocks.
  **L645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L646 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x2_S8`.
  **L646 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x2_S8`。
- **L647 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x2_U8`.
  **L647 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x2_U8`。
- **L648 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x4_S8`.
  **L648 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x4_S8`。

### Lines 649-672

````tablegen
  def SVMLSL_MULTI_VG4x4_U8 : Inst<"svmls_za32[_{d}]_vg4x4", "vm44", "Uc", MergeNone, "aarch64_sme_umls_za32_vg4x4", [IsStreaming, IsInOutZA], []>;

  // SINGLE MLAL
  def SVMLAL_SINGLE_VG2x1_F16 : Inst<"svmla_za32[_{d}]_vg2x1",          "vmdd", "bh", MergeNone, "aarch64_sme_fmlal_single_vg2x1", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x2_F16 : Inst<"svmla[_single]_za32[_{d}]_vg2x2", "vm2d", "bh", MergeNone, "aarch64_sme_fmlal_single_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x4_F16 : Inst<"svmla[_single]_za32[_{d}]_vg2x4", "vm4d", "bh", MergeNone, "aarch64_sme_fmlal_single_vg2x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x1_S16 : Inst<"svmla_za32[_{d}]_vg2x1",          "vmdd", "s", MergeNone, "aarch64_sme_smlal_single_vg2x1", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x2_S16 : Inst<"svmla[_single]_za32[_{d}]_vg2x2", "vm2d", "s", MergeNone, "aarch64_sme_smlal_single_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x4_S16 : Inst<"svmla[_single]_za32[_{d}]_vg2x4", "vm4d", "s", MergeNone, "aarch64_sme_smlal_single_vg2x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x1_U16 : Inst<"svmla_za32[_{d}]_vg2x1",          "vmdd", "Us", MergeNone, "aarch64_sme_umlal_single_vg2x1", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x2_U16 : Inst<"svmla[_single]_za32[_{d}]_vg2x2", "vm2d", "Us", MergeNone, "aarch64_sme_umlal_single_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG2x4_U16 : Inst<"svmla[_single]_za32[_{d}]_vg2x4", "vm4d", "Us", MergeNone, "aarch64_sme_umlal_single_vg2x4", [IsStreaming, IsInOutZA], []>;

  def SVMLAL_SINGLE_VG4x1_S8  : Inst<"svmla_za32[_{d}]_vg4x1",          "vmdd", "c", MergeNone, "aarch64_sme_smla_za32_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x1_U8  : Inst<"svmla_za32[_{d}]_vg4x1",          "vmdd", "Uc", MergeNone, "aarch64_sme_umla_za32_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x2_S8  : Inst<"svmla[_single]_za32[_{d}]_vg4x2", "vm2d", "c", MergeNone, "aarch64_sme_smla_za32_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x2_U8  : Inst<"svmla[_single]_za32[_{d}]_vg4x2", "vm2d", "Uc", MergeNone, "aarch64_sme_umla_za32_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x4_S8  : Inst<"svmla[_single]_za32[_{d}]_vg4x4", "vm4d", "c", MergeNone, "aarch64_sme_smla_za32_single_vg4x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x4_U8  : Inst<"svmla[_single]_za32[_{d}]_vg4x4", "vm4d", "Uc", MergeNone, "aarch64_sme_umla_za32_single_vg4x4", [IsStreaming, IsInOutZA], []>;

  // SINGLE MLSL
  def SVMLSL_SINGLE_VG2x1_F16 : Inst<"svmls_za32[_{d}]_vg2x1",          "vmdd", "bh", MergeNone, "aarch64_sme_fmlsl_single_vg2x1", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG2x2_F16 : Inst<"svmls[_single]_za32[_{d}]_vg2x2", "vm2d", "bh", MergeNone, "aarch64_sme_fmlsl_single_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG2x4_F16 : Inst<"svmls[_single]_za32[_{d}]_vg2x4", "vm4d", "bh", MergeNone, "aarch64_sme_fmlsl_single_vg2x4", [IsStreaming, IsInOutZA], []>;
````
- **L649 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x4_U8`.
  **L649 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x4_U8`。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Comment explains nearby logic, constraints, or intent: `SINGLE MLAL`.
  **L651 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SINGLE MLAL`。
- **L652 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x1_F16`.
  **L652 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x1_F16`。
- **L653 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x2_F16`.
  **L653 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x2_F16`。
- **L654 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x4_F16`.
  **L654 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x4_F16`。
- **L655 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x1_S16`.
  **L655 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x1_S16`。
- **L656 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x2_S16`.
  **L656 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x2_S16`。
- **L657 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x4_S16`.
  **L657 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x4_S16`。
- **L658 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x1_U16`.
  **L658 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x1_U16`。
- **L659 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x2_U16`.
  **L659 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x2_U16`。
- **L660 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG2x4_U16`.
  **L660 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG2x4_U16`。
- **L661 EN**: Blank line separating nearby declarations or logic blocks.
  **L661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L662 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x1_S8`.
  **L662 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x1_S8`。
- **L663 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x1_U8`.
  **L663 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x1_U8`。
- **L664 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x2_S8`.
  **L664 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x2_S8`。
- **L665 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x2_U8`.
  **L665 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x2_U8`。
- **L666 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x4_S8`.
  **L666 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x4_S8`。
- **L667 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x4_U8`.
  **L667 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x4_U8`。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Comment explains nearby logic, constraints, or intent: `SINGLE MLSL`.
  **L669 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SINGLE MLSL`。
- **L670 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x1_F16`.
  **L670 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x1_F16`。
- **L671 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x2_F16`.
  **L671 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x2_F16`。
- **L672 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x4_F16`.
  **L672 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x4_F16`。

### Lines 673-696

````tablegen
  def SVMLSL_SINGLE_VG2x1_S16 : Inst<"svmls_za32[_{d}]_vg2x1",          "vmdd", "s", MergeNone, "aarch64_sme_smlsl_single_vg2x1", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG2x2_S16 : Inst<"svmls[_single]_za32[_{d}]_vg2x2", "vm2d", "s", MergeNone, "aarch64_sme_smlsl_single_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG2x4_S16 : Inst<"svmls[_single]_za32[_{d}]_vg2x4", "vm4d", "s", MergeNone, "aarch64_sme_smlsl_single_vg2x4", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG2x1_U16 : Inst<"svmls_za32[_{d}]_vg2x1",          "vmdd", "Us", MergeNone, "aarch64_sme_umlsl_single_vg2x1", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG2x2_U16 : Inst<"svmls[_single]_za32[_{d}]_vg2x2", "vm2d", "Us", MergeNone, "aarch64_sme_umlsl_single_vg2x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG2x4_U16 : Inst<"svmls[_single]_za32[_{d}]_vg2x4", "vm4d", "Us", MergeNone, "aarch64_sme_umlsl_single_vg2x4", [IsStreaming, IsInOutZA], []>;

  def SVMLSL_SINGLE_VG4x1_S8  : Inst<"svmls_za32[_{d}]_vg4x1",          "vmdd", "c", MergeNone, "aarch64_sme_smls_za32_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x1_U8  : Inst<"svmls_za32[_{d}]_vg4x1",          "vmdd", "Uc", MergeNone, "aarch64_sme_umls_za32_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x2_S8  : Inst<"svmls[_single]_za32[_{d}]_vg4x2", "vm2d", "c", MergeNone, "aarch64_sme_smls_za32_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x2_U8  : Inst<"svmls[_single]_za32[_{d}]_vg4x2", "vm2d", "Uc", MergeNone, "aarch64_sme_umls_za32_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x4_S8  : Inst<"svmls[_single]_za32[_{d}]_vg4x4", "vm4d", "c", MergeNone, "aarch64_sme_smls_za32_single_vg4x4", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x4_U8  : Inst<"svmls[_single]_za32[_{d}]_vg4x4", "vm4d", "Uc", MergeNone, "aarch64_sme_umls_za32_single_vg4x4", [IsStreaming, IsInOutZA], []>;

  // INDEXED MLAL
  def SVMLAL_LANE_VG2x1_F16 : Inst<"svmla_lane_za32[_{d}]_vg2x1", "vmddi", "bh", MergeNone, "aarch64_sme_fmlal_lane_vg2x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x2_F16 : Inst<"svmla_lane_za32[_{d}]_vg2x2", "vm2di", "bh", MergeNone, "aarch64_sme_fmlal_lane_vg2x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x4_F16 : Inst<"svmla_lane_za32[_{d}]_vg2x4", "vm4di", "bh", MergeNone, "aarch64_sme_fmlal_lane_vg2x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x1_S16 : Inst<"svmla_lane_za32[_{d}]_vg2x1", "vmddi", "s", MergeNone, "aarch64_sme_smlal_lane_vg2x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x2_S16 : Inst<"svmla_lane_za32[_{d}]_vg2x2", "vm2di", "s", MergeNone, "aarch64_sme_smlal_lane_vg2x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x4_S16 : Inst<"svmla_lane_za32[_{d}]_vg2x4", "vm4di", "s", MergeNone, "aarch64_sme_smlal_lane_vg2x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x1_U16 : Inst<"svmla_lane_za32[_{d}]_vg2x1", "vmddi", "Us", MergeNone, "aarch64_sme_umlal_lane_vg2x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x2_U16 : Inst<"svmla_lane_za32[_{d}]_vg2x2", "vm2di", "Us", MergeNone, "aarch64_sme_umlal_lane_vg2x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG2x4_U16 : Inst<"svmla_lane_za32[_{d}]_vg2x4", "vm4di", "Us", MergeNone, "aarch64_sme_umlal_lane_vg2x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
````
- **L673 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x1_S16`.
  **L673 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x1_S16`。
- **L674 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x2_S16`.
  **L674 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x2_S16`。
- **L675 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x4_S16`.
  **L675 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x4_S16`。
- **L676 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x1_U16`.
  **L676 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x1_U16`。
- **L677 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x2_U16`.
  **L677 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x2_U16`。
- **L678 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG2x4_U16`.
  **L678 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG2x4_U16`。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L680 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x1_S8`.
  **L680 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x1_S8`。
- **L681 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x1_U8`.
  **L681 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x1_U8`。
- **L682 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x2_S8`.
  **L682 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x2_S8`。
- **L683 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x2_U8`.
  **L683 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x2_U8`。
- **L684 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x4_S8`.
  **L684 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x4_S8`。
- **L685 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x4_U8`.
  **L685 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x4_U8`。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L687 EN**: Comment explains nearby logic, constraints, or intent: `INDEXED MLAL`.
  **L687 CN**: 注释解释附近代码的逻辑、约束或设计意图：`INDEXED MLAL`。
- **L688 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x1_F16`.
  **L688 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x1_F16`。
- **L689 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x2_F16`.
  **L689 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x2_F16`。
- **L690 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x4_F16`.
  **L690 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x4_F16`。
- **L691 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x1_S16`.
  **L691 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x1_S16`。
- **L692 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x2_S16`.
  **L692 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x2_S16`。
- **L693 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x4_S16`.
  **L693 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x4_S16`。
- **L694 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x1_U16`.
  **L694 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x1_U16`。
- **L695 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x2_U16`.
  **L695 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x2_U16`。
- **L696 EN**: Declares TableGen def record `SVMLAL_LANE_VG2x4_U16`.
  **L696 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG2x4_U16`。

### Lines 697-720

````tablegen

  def SVMLAL_LANE_VG4x1_S8 : Inst<"svmla_lane_za32[_{d}]_vg4x1", "vmddi", "c", MergeNone, "aarch64_sme_smla_za32_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLAL_LANE_VG4x1_U8 : Inst<"svmla_lane_za32[_{d}]_vg4x1", "vmddi", "Uc", MergeNone, "aarch64_sme_umla_za32_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLAL_LANE_VG4x2_S8 : Inst<"svmla_lane_za32[_{d}]_vg4x2", "vm2di", "c", MergeNone, "aarch64_sme_smla_za32_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLAL_LANE_VG4x2_U8 : Inst<"svmla_lane_za32[_{d}]_vg4x2", "vm2di", "Uc", MergeNone, "aarch64_sme_umla_za32_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLAL_LANE_VG4x4_S8 : Inst<"svmla_lane_za32[_{d}]_vg4x4", "vm4di", "c", MergeNone, "aarch64_sme_smla_za32_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLAL_LANE_VG4x4_U8 : Inst<"svmla_lane_za32[_{d}]_vg4x4", "vm4di", "Uc", MergeNone, "aarch64_sme_umla_za32_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;

  // INDEXED MLSL
  def SVMLSL_LANE_VG2x1_F16 : Inst<"svmls_lane_za32[_{d}]_vg2x1", "vmddi", "bh", MergeNone, "aarch64_sme_fmlsl_lane_vg2x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x2_F16 : Inst<"svmls_lane_za32[_{d}]_vg2x2", "vm2di", "bh", MergeNone, "aarch64_sme_fmlsl_lane_vg2x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x4_F16 : Inst<"svmls_lane_za32[_{d}]_vg2x4", "vm4di", "bh", MergeNone, "aarch64_sme_fmlsl_lane_vg2x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x1_S16 : Inst<"svmls_lane_za32[_{d}]_vg2x1", "vmddi", "s", MergeNone, "aarch64_sme_smlsl_lane_vg2x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x2_S16 : Inst<"svmls_lane_za32[_{d}]_vg2x2", "vm2di", "s", MergeNone, "aarch64_sme_smlsl_lane_vg2x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x4_S16 : Inst<"svmls_lane_za32[_{d}]_vg2x4", "vm4di", "s", MergeNone, "aarch64_sme_smlsl_lane_vg2x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x1_U16 : Inst<"svmls_lane_za32[_{d}]_vg2x1", "vmddi", "Us", MergeNone, "aarch64_sme_umlsl_lane_vg2x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x2_U16 : Inst<"svmls_lane_za32[_{d}]_vg2x2", "vm2di", "Us", MergeNone, "aarch64_sme_umlsl_lane_vg2x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG2x4_U16 : Inst<"svmls_lane_za32[_{d}]_vg2x4", "vm4di", "Us", MergeNone, "aarch64_sme_umlsl_lane_vg2x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;

  def SVMLSL_LANE_VG4x1_S8 : Inst<"svmls_lane_za32[_{d}]_vg4x1", "vmddi", "c", MergeNone, "aarch64_sme_smls_za32_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLSL_LANE_VG4x1_U8 : Inst<"svmls_lane_za32[_{d}]_vg4x1", "vmddi", "Uc", MergeNone, "aarch64_sme_umls_za32_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLSL_LANE_VG4x2_S8 : Inst<"svmls_lane_za32[_{d}]_vg4x2", "vm2di", "c", MergeNone, "aarch64_sme_smls_za32_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLSL_LANE_VG4x2_U8 : Inst<"svmls_lane_za32[_{d}]_vg4x2", "vm2di", "Uc", MergeNone, "aarch64_sme_umls_za32_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLSL_LANE_VG4x4_S8 : Inst<"svmls_lane_za32[_{d}]_vg4x4", "vm4di", "c", MergeNone, "aarch64_sme_smls_za32_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L698 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x1_S8`.
  **L698 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x1_S8`。
- **L699 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x1_U8`.
  **L699 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x1_U8`。
- **L700 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x2_S8`.
  **L700 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x2_S8`。
- **L701 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x2_U8`.
  **L701 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x2_U8`。
- **L702 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x4_S8`.
  **L702 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x4_S8`。
- **L703 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x4_U8`.
  **L703 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x4_U8`。
- **L704 EN**: Blank line separating nearby declarations or logic blocks.
  **L704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L705 EN**: Comment explains nearby logic, constraints, or intent: `INDEXED MLSL`.
  **L705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`INDEXED MLSL`。
- **L706 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x1_F16`.
  **L706 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x1_F16`。
- **L707 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x2_F16`.
  **L707 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x2_F16`。
- **L708 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x4_F16`.
  **L708 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x4_F16`。
- **L709 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x1_S16`.
  **L709 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x1_S16`。
- **L710 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x2_S16`.
  **L710 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x2_S16`。
- **L711 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x4_S16`.
  **L711 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x4_S16`。
- **L712 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x1_U16`.
  **L712 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x1_U16`。
- **L713 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x2_U16`.
  **L713 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x2_U16`。
- **L714 EN**: Declares TableGen def record `SVMLSL_LANE_VG2x4_U16`.
  **L714 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG2x4_U16`。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L716 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x1_S8`.
  **L716 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x1_S8`。
- **L717 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x1_U8`.
  **L717 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x1_U8`。
- **L718 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x2_S8`.
  **L718 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x2_S8`。
- **L719 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x2_U8`.
  **L719 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x2_U8`。
- **L720 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x4_S8`.
  **L720 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x4_S8`。

### Lines 721-744

````tablegen
  def SVMLSL_LANE_VG4x4_U8 : Inst<"svmls_lane_za32[_{d}]_vg4x4", "vm4di", "Uc", MergeNone, "aarch64_sme_umls_za32_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;

  // SINGLE SUMLALL
  // Single sumla maps to usmla, with zn & zm operands swapped
  def SVSUMLALL_SINGLE_VG4x1 : Inst<"svsumla_za32[_{d}]_vg4x1",          "vmdu",   "c", MergeNone, "aarch64_sme_usmla_za32_single_vg4x1", [IsStreaming, IsInOutZA], []>;

  def SVSUMLALL_SINGLE_VG4x2 : Inst<"svsumla[_single]_za32[_{d}]_vg4x2", "vm2.du", "c", MergeNone, "aarch64_sme_sumla_za32_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVSUMLALL_SINGLE_VG4x4 : Inst<"svsumla[_single]_za32[_{d}]_vg4x4", "vm4.du", "c", MergeNone, "aarch64_sme_sumla_za32_single_vg4x4", [IsStreaming, IsInOutZA], []>;

  // Multi-multi sumla builtins are mapped to usmla, with zn & zm operands swapped
  def SVSUMLALL_MULTI_VG4x2 : Inst<"svsumla_za32[_{d}]_vg4x2", "vm2.d2.u", "c", MergeNone, "aarch64_sme_usmla_za32_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVSUMLALL_MULTI_VG4x4 : Inst<"svsumla_za32[_{d}]_vg4x4", "vm4.d4.u", "c", MergeNone, "aarch64_sme_usmla_za32_vg4x4", [IsStreaming, IsInOutZA], []>;

  // INDEXED SUMLALL
  def SVSUMLALL_LANE_VG4x1 : Inst<"svsumla_lane_za32[_{d}]_vg4x1", "vmdui", "c", MergeNone, "aarch64_sme_sumla_za32_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVSUMLALL_LANE_VG4x2 : Inst<"svsumla_lane_za32[_{d}]_vg4x2", "vm2ui", "c", MergeNone, "aarch64_sme_sumla_za32_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVSUMLALL_LANE_VG4x4 : Inst<"svsumla_lane_za32[_{d}]_vg4x4", "vm4ui", "c", MergeNone, "aarch64_sme_sumla_za32_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;

  // SINGLE USMLALL
  def SVUSMLALL_SINGLE_VG4x1 : Inst<"svusmla_za32[_{d}]_vg4x1",          "vmdx",   "Uc", MergeNone, "aarch64_sme_usmla_za32_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVUSMLALL_SINGLE_VG4x2 : Inst<"svusmla[_single]_za32[_{d}]_vg4x2", "vm2.dx", "Uc", MergeNone, "aarch64_sme_usmla_za32_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVUSMLALL_SINGLE_VG4x4 : Inst<"svusmla[_single]_za32[_{d}]_vg4x4", "vm4.dx", "Uc", MergeNone, "aarch64_sme_usmla_za32_single_vg4x4", [IsStreaming, IsInOutZA], []>;

  // MULTI USMLALL
````
- **L721 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x4_U8`.
  **L721 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x4_U8`。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L723 EN**: Comment explains nearby logic, constraints, or intent: `SINGLE SUMLALL`.
  **L723 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SINGLE SUMLALL`。
- **L724 EN**: Comment explains nearby logic, constraints, or intent: `Single sumla maps to usmla, with zn & zm operands swapped`.
  **L724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Single sumla maps to usmla, with zn & zm operands swapped`。
- **L725 EN**: Declares TableGen def record `SVSUMLALL_SINGLE_VG4x1`.
  **L725 CN**: 声明 TableGen def 记录 `SVSUMLALL_SINGLE_VG4x1`。
- **L726 EN**: Blank line separating nearby declarations or logic blocks.
  **L726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L727 EN**: Declares TableGen def record `SVSUMLALL_SINGLE_VG4x2`.
  **L727 CN**: 声明 TableGen def 记录 `SVSUMLALL_SINGLE_VG4x2`。
- **L728 EN**: Declares TableGen def record `SVSUMLALL_SINGLE_VG4x4`.
  **L728 CN**: 声明 TableGen def 记录 `SVSUMLALL_SINGLE_VG4x4`。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L730 EN**: Comment explains nearby logic, constraints, or intent: `Multi-multi sumla builtins are mapped to usmla, with zn & zm operands swapped`.
  **L730 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multi-multi sumla builtins are mapped to usmla, with zn & zm operands swapped`。
- **L731 EN**: Declares TableGen def record `SVSUMLALL_MULTI_VG4x2`.
  **L731 CN**: 声明 TableGen def 记录 `SVSUMLALL_MULTI_VG4x2`。
- **L732 EN**: Declares TableGen def record `SVSUMLALL_MULTI_VG4x4`.
  **L732 CN**: 声明 TableGen def 记录 `SVSUMLALL_MULTI_VG4x4`。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L734 EN**: Comment explains nearby logic, constraints, or intent: `INDEXED SUMLALL`.
  **L734 CN**: 注释解释附近代码的逻辑、约束或设计意图：`INDEXED SUMLALL`。
- **L735 EN**: Declares TableGen def record `SVSUMLALL_LANE_VG4x1`.
  **L735 CN**: 声明 TableGen def 记录 `SVSUMLALL_LANE_VG4x1`。
- **L736 EN**: Declares TableGen def record `SVSUMLALL_LANE_VG4x2`.
  **L736 CN**: 声明 TableGen def 记录 `SVSUMLALL_LANE_VG4x2`。
- **L737 EN**: Declares TableGen def record `SVSUMLALL_LANE_VG4x4`.
  **L737 CN**: 声明 TableGen def 记录 `SVSUMLALL_LANE_VG4x4`。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L739 EN**: Comment explains nearby logic, constraints, or intent: `SINGLE USMLALL`.
  **L739 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SINGLE USMLALL`。
- **L740 EN**: Declares TableGen def record `SVUSMLALL_SINGLE_VG4x1`.
  **L740 CN**: 声明 TableGen def 记录 `SVUSMLALL_SINGLE_VG4x1`。
- **L741 EN**: Declares TableGen def record `SVUSMLALL_SINGLE_VG4x2`.
  **L741 CN**: 声明 TableGen def 记录 `SVUSMLALL_SINGLE_VG4x2`。
- **L742 EN**: Declares TableGen def record `SVUSMLALL_SINGLE_VG4x4`.
  **L742 CN**: 声明 TableGen def 记录 `SVUSMLALL_SINGLE_VG4x4`。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Comment explains nearby logic, constraints, or intent: `MULTI USMLALL`.
  **L744 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MULTI USMLALL`。

### Lines 745-768

````tablegen
  def SVUSMLALL_MULTI_VG4x2 : Inst<"svusmla_za32[_{d}]_vg4x2", "vm2.d2.x", "Uc", MergeNone, "aarch64_sme_usmla_za32_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVUSMLALL_MULTI_VG4x4 : Inst<"svusmla_za32[_{d}]_vg4x4", "vm4.d4.x", "Uc", MergeNone, "aarch64_sme_usmla_za32_vg4x4", [IsStreaming, IsInOutZA], []>;

  // INDEXED USMLALL
  def SVUSMLALL_LANE_VG4x1 : Inst<"svusmla_lane_za32[_{d}]_vg4x1", "vmdxi", "Uc", MergeNone, "aarch64_sme_usmla_za32_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVUSMLALL_LANE_VG4x2 : Inst<"svusmla_lane_za32[_{d}]_vg4x2", "vm2xi", "Uc", MergeNone, "aarch64_sme_usmla_za32_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
  def SVUSMLALL_LANE_VG4x4 : Inst<"svusmla_lane_za32[_{d}]_vg4x4", "vm4xi", "Uc", MergeNone, "aarch64_sme_usmla_za32_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_15>]>;
}

let SMETargetGuard = "sme2,sme-i16i64" in {
  // MULTI MLAL
  def SVMLAL_MULTI_VG4x2_S16 : Inst<"svmla_za64[_{d}]_vg4x2", "vm22", "s", MergeNone, "aarch64_sme_smla_za64_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG4x2_U16 : Inst<"svmla_za64[_{d}]_vg4x2", "vm22", "Us", MergeNone, "aarch64_sme_umla_za64_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG4x4_S16 : Inst<"svmla_za64[_{d}]_vg4x4", "vm44", "s", MergeNone, "aarch64_sme_smla_za64_vg4x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_MULTI_VG4x4_U16 : Inst<"svmla_za64[_{d}]_vg4x4", "vm44", "Us", MergeNone, "aarch64_sme_umla_za64_vg4x4", [IsStreaming, IsInOutZA], []>;

  // MULTI MLSL
  def SVMLSL_MULTI_VG4x2_S16 : Inst<"svmls_za64[_{d}]_vg4x2", "vm22", "s", MergeNone, "aarch64_sme_smls_za64_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG4x2_U16 : Inst<"svmls_za64[_{d}]_vg4x2", "vm22", "Us", MergeNone, "aarch64_sme_umls_za64_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG4x4_S16 : Inst<"svmls_za64[_{d}]_vg4x4", "vm44", "s", MergeNone, "aarch64_sme_smls_za64_vg4x4", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_MULTI_VG4x4_U16 : Inst<"svmls_za64[_{d}]_vg4x4", "vm44", "Us", MergeNone, "aarch64_sme_umls_za64_vg4x4", [IsStreaming, IsInOutZA], []>;

  // SINGLE MLAL
  def SVMLAL_SINGLE_VG4x1_S16 : Inst<"svmla_za64[_{d}]_vg4x1",          "vmdd", "s", MergeNone, "aarch64_sme_smla_za64_single_vg4x1", [IsStreaming, IsInOutZA], []>;
````
- **L745 EN**: Declares TableGen def record `SVUSMLALL_MULTI_VG4x2`.
  **L745 CN**: 声明 TableGen def 记录 `SVUSMLALL_MULTI_VG4x2`。
- **L746 EN**: Declares TableGen def record `SVUSMLALL_MULTI_VG4x4`.
  **L746 CN**: 声明 TableGen def 记录 `SVUSMLALL_MULTI_VG4x4`。
- **L747 EN**: Blank line separating nearby declarations or logic blocks.
  **L747 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L748 EN**: Comment explains nearby logic, constraints, or intent: `INDEXED USMLALL`.
  **L748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`INDEXED USMLALL`。
- **L749 EN**: Declares TableGen def record `SVUSMLALL_LANE_VG4x1`.
  **L749 CN**: 声明 TableGen def 记录 `SVUSMLALL_LANE_VG4x1`。
- **L750 EN**: Declares TableGen def record `SVUSMLALL_LANE_VG4x2`.
  **L750 CN**: 声明 TableGen def 记录 `SVUSMLALL_LANE_VG4x2`。
- **L751 EN**: Declares TableGen def record `SVUSMLALL_LANE_VG4x4`.
  **L751 CN**: 声明 TableGen def 记录 `SVUSMLALL_LANE_VG4x4`。
- **L752 EN**: Closes the current lexical scope or compound statement.
  **L752 CN**: 结束当前词法作用域或复合语句块。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-i16i64" in {`.
  **L754 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-i16i64" in {`。
- **L755 EN**: Comment explains nearby logic, constraints, or intent: `MULTI MLAL`.
  **L755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MULTI MLAL`。
- **L756 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x2_S16`.
  **L756 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x2_S16`。
- **L757 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x2_U16`.
  **L757 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x2_U16`。
- **L758 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x4_S16`.
  **L758 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x4_S16`。
- **L759 EN**: Declares TableGen def record `SVMLAL_MULTI_VG4x4_U16`.
  **L759 CN**: 声明 TableGen def 记录 `SVMLAL_MULTI_VG4x4_U16`。
- **L760 EN**: Blank line separating nearby declarations or logic blocks.
  **L760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L761 EN**: Comment explains nearby logic, constraints, or intent: `MULTI MLSL`.
  **L761 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MULTI MLSL`。
- **L762 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x2_S16`.
  **L762 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x2_S16`。
- **L763 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x2_U16`.
  **L763 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x2_U16`。
- **L764 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x4_S16`.
  **L764 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x4_S16`。
- **L765 EN**: Declares TableGen def record `SVMLSL_MULTI_VG4x4_U16`.
  **L765 CN**: 声明 TableGen def 记录 `SVMLSL_MULTI_VG4x4_U16`。
- **L766 EN**: Blank line separating nearby declarations or logic blocks.
  **L766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L767 EN**: Comment explains nearby logic, constraints, or intent: `SINGLE MLAL`.
  **L767 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SINGLE MLAL`。
- **L768 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x1_S16`.
  **L768 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x1_S16`。

### Lines 769-792

````tablegen
  def SVMLAL_SINGLE_VG4x1_U16 : Inst<"svmla_za64[_{d}]_vg4x1",          "vmdd", "Us", MergeNone, "aarch64_sme_umla_za64_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x2_S16 : Inst<"svmla[_single]_za64[_{d}]_vg4x2", "vm2d", "s", MergeNone, "aarch64_sme_smla_za64_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x2_U16 : Inst<"svmla[_single]_za64[_{d}]_vg4x2", "vm2d", "Us", MergeNone, "aarch64_sme_umla_za64_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x4_S16 : Inst<"svmla[_single]_za64[_{d}]_vg4x4", "vm4d", "s", MergeNone, "aarch64_sme_smla_za64_single_vg4x4", [IsStreaming, IsInOutZA], []>;
  def SVMLAL_SINGLE_VG4x4_U16 : Inst<"svmla[_single]_za64[_{d}]_vg4x4", "vm4d", "Us", MergeNone, "aarch64_sme_umla_za64_single_vg4x4", [IsStreaming, IsInOutZA], []>;

  // SINGLE MLSL
  def SVMLSL_SINGLE_VG4x1_S16 : Inst<"svmls_za64[_{d}]_vg4x1",          "vmdd", "s", MergeNone, "aarch64_sme_smls_za64_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x1_U16 : Inst<"svmls_za64[_{d}]_vg4x1",          "vmdd", "Us", MergeNone, "aarch64_sme_umls_za64_single_vg4x1", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x2_S16 : Inst<"svmls[_single]_za64[_{d}]_vg4x2", "vm2d", "s", MergeNone, "aarch64_sme_smls_za64_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x2_U16 : Inst<"svmls[_single]_za64[_{d}]_vg4x2", "vm2d", "Us", MergeNone, "aarch64_sme_umls_za64_single_vg4x2", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x4_S16 : Inst<"svmls[_single]_za64[_{d}]_vg4x4", "vm4d", "s", MergeNone, "aarch64_sme_smls_za64_single_vg4x4", [IsStreaming, IsInOutZA], []>;
  def SVMLSL_SINGLE_VG4x4_U16 : Inst<"svmls[_single]_za64[_{d}]_vg4x4", "vm4d", "Us", MergeNone, "aarch64_sme_umls_za64_single_vg4x4", [IsStreaming, IsInOutZA], []>;

  // INDEXED MLAL
  def SVMLAL_LANE_VG4x1_S16 : Inst<"svmla_lane_za64[_{d}]_vg4x1", "vmddi", "s", MergeNone, "aarch64_sme_smla_za64_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG4x1_U16 : Inst<"svmla_lane_za64[_{d}]_vg4x1", "vmddi", "Us", MergeNone, "aarch64_sme_umla_za64_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG4x2_S16 : Inst<"svmla_lane_za64[_{d}]_vg4x2", "vm2di", "s", MergeNone, "aarch64_sme_smla_za64_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG4x2_U16 : Inst<"svmla_lane_za64[_{d}]_vg4x2", "vm2di", "Us", MergeNone, "aarch64_sme_umla_za64_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG4x4_S16 : Inst<"svmla_lane_za64[_{d}]_vg4x4", "vm4di", "s", MergeNone, "aarch64_sme_smla_za64_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLAL_LANE_VG4x4_U16 : Inst<"svmla_lane_za64[_{d}]_vg4x4", "vm4di", "Us", MergeNone, "aarch64_sme_umla_za64_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;

  // INDEXED MLSL
  def SVMLSL_LANE_VG4x1_S16 : Inst<"svmls_lane_za64[_{d}]_vg4x1", "vmddi", "s", MergeNone, "aarch64_sme_smls_za64_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
````
- **L769 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x1_U16`.
  **L769 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x1_U16`。
- **L770 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x2_S16`.
  **L770 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x2_S16`。
- **L771 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x2_U16`.
  **L771 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x2_U16`。
- **L772 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x4_S16`.
  **L772 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x4_S16`。
- **L773 EN**: Declares TableGen def record `SVMLAL_SINGLE_VG4x4_U16`.
  **L773 CN**: 声明 TableGen def 记录 `SVMLAL_SINGLE_VG4x4_U16`。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L775 EN**: Comment explains nearby logic, constraints, or intent: `SINGLE MLSL`.
  **L775 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SINGLE MLSL`。
- **L776 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x1_S16`.
  **L776 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x1_S16`。
- **L777 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x1_U16`.
  **L777 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x1_U16`。
- **L778 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x2_S16`.
  **L778 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x2_S16`。
- **L779 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x2_U16`.
  **L779 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x2_U16`。
- **L780 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x4_S16`.
  **L780 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x4_S16`。
- **L781 EN**: Declares TableGen def record `SVMLSL_SINGLE_VG4x4_U16`.
  **L781 CN**: 声明 TableGen def 记录 `SVMLSL_SINGLE_VG4x4_U16`。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L783 EN**: Comment explains nearby logic, constraints, or intent: `INDEXED MLAL`.
  **L783 CN**: 注释解释附近代码的逻辑、约束或设计意图：`INDEXED MLAL`。
- **L784 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x1_S16`.
  **L784 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x1_S16`。
- **L785 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x1_U16`.
  **L785 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x1_U16`。
- **L786 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x2_S16`.
  **L786 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x2_S16`。
- **L787 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x2_U16`.
  **L787 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x2_U16`。
- **L788 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x4_S16`.
  **L788 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x4_S16`。
- **L789 EN**: Declares TableGen def record `SVMLAL_LANE_VG4x4_U16`.
  **L789 CN**: 声明 TableGen def 记录 `SVMLAL_LANE_VG4x4_U16`。
- **L790 EN**: Blank line separating nearby declarations or logic blocks.
  **L790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L791 EN**: Comment explains nearby logic, constraints, or intent: `INDEXED MLSL`.
  **L791 CN**: 注释解释附近代码的逻辑、约束或设计意图：`INDEXED MLSL`。
- **L792 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x1_S16`.
  **L792 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x1_S16`。

### Lines 793-816

````tablegen
  def SVMLSL_LANE_VG4x1_U16 : Inst<"svmls_lane_za64[_{d}]_vg4x1", "vmddi", "Us", MergeNone, "aarch64_sme_umls_za64_lane_vg4x1", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG4x2_S16 : Inst<"svmls_lane_za64[_{d}]_vg4x2", "vm2di", "s", MergeNone, "aarch64_sme_smls_za64_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG4x2_U16 : Inst<"svmls_lane_za64[_{d}]_vg4x2", "vm2di", "Us", MergeNone, "aarch64_sme_umls_za64_lane_vg4x2", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG4x4_S16 : Inst<"svmls_lane_za64[_{d}]_vg4x4", "vm4di", "s", MergeNone, "aarch64_sme_smls_za64_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
  def SVMLSL_LANE_VG4x4_U16 : Inst<"svmls_lane_za64[_{d}]_vg4x4", "vm4di", "Us", MergeNone, "aarch64_sme_umls_za64_lane_vg4x4", [IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
}

//
// Spill and fill of ZT0
//
let SMETargetGuard = "sme2" in {
  def SVLDR_ZT : Inst<"svldr_zt", "viQ", "", MergeNone, "aarch64_sme_ldr_zt", [IsOverloadNone, IsStreamingCompatible, IsInOutZT0], [ImmCheck<0, ImmCheck0_0>]>;
  def SVSTR_ZT : Inst<"svstr_zt", "vi%", "", MergeNone, "aarch64_sme_str_zt", [IsOverloadNone, IsStreamingCompatible, IsInZT0], [ImmCheck<0, ImmCheck0_0>]>;
}

//
// Zero ZT0
//
let SMETargetGuard = "sme2" in {
  def SVZERO_ZT : Inst<"svzero_zt", "vi", "", MergeNone, "aarch64_sme_zero_zt", [IsOverloadNone, IsStreamingCompatible, IsOutZT0], [ImmCheck<0, ImmCheck0_0>]>;
}

def IN_STREAMING_MODE :  Inst<"__arm_in_streaming_mode", "sv", "Pc", MergeNone, "aarch64_sme_in_streaming_mode", [IsOverloadNone, IsStreamingCompatible], []>;

````
- **L793 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x1_U16`.
  **L793 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x1_U16`。
- **L794 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x2_S16`.
  **L794 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x2_S16`。
- **L795 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x2_U16`.
  **L795 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x2_U16`。
- **L796 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x4_S16`.
  **L796 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x4_S16`。
- **L797 EN**: Declares TableGen def record `SVMLSL_LANE_VG4x4_U16`.
  **L797 CN**: 声明 TableGen def 记录 `SVMLSL_LANE_VG4x4_U16`。
- **L798 EN**: Closes the current lexical scope or compound statement.
  **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L800 EN**: Separator comment used for visual grouping.
  **L800 CN**: 用于视觉分组的分隔注释。
- **L801 EN**: Comment explains nearby logic, constraints, or intent: `Spill and fill of ZT0`.
  **L801 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Spill and fill of ZT0`。
- **L802 EN**: Separator comment used for visual grouping.
  **L802 CN**: 用于视觉分组的分隔注释。
- **L803 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L803 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L804 EN**: Declares TableGen def record `SVLDR_ZT`.
  **L804 CN**: 声明 TableGen def 记录 `SVLDR_ZT`。
- **L805 EN**: Declares TableGen def record `SVSTR_ZT`.
  **L805 CN**: 声明 TableGen def 记录 `SVSTR_ZT`。
- **L806 EN**: Closes the current lexical scope or compound statement.
  **L806 CN**: 结束当前词法作用域或复合语句块。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Separator comment used for visual grouping.
  **L808 CN**: 用于视觉分组的分隔注释。
- **L809 EN**: Comment explains nearby logic, constraints, or intent: `Zero ZT0`.
  **L809 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zero ZT0`。
- **L810 EN**: Separator comment used for visual grouping.
  **L810 CN**: 用于视觉分组的分隔注释。
- **L811 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L811 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L812 EN**: Declares TableGen def record `SVZERO_ZT`.
  **L812 CN**: 声明 TableGen def 记录 `SVZERO_ZT`。
- **L813 EN**: Closes the current lexical scope or compound statement.
  **L813 CN**: 结束当前词法作用域或复合语句块。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Declares TableGen def record `IN_STREAMING_MODE`.
  **L815 CN**: 声明 TableGen def 记录 `IN_STREAMING_MODE`。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 817-840

````tablegen
//
// lookup table expand four contiguous registers
//
let SMETargetGuard = "sme2" in {
  def SVLUTI2_LANE_ZT_X4 : Inst<"svluti2_lane_zt_{d}_x4", "4.di[i", "cUcsUsiUimbhf", MergeNone, "aarch64_sme_luti2_lane_zt_x4", [IsStreaming, IsInZT0], [ImmCheck<0, ImmCheck0_0>, ImmCheck<2, ImmCheck0_3>]>;
  def SVLUTI4_LANE_ZT_X4 : Inst<"svluti4_lane_zt_{d}_x4", "4.di[i", "sUsiUibhf", MergeNone, "aarch64_sme_luti4_lane_zt_x4", [IsStreaming, IsInZT0], [ImmCheck<0, ImmCheck0_0>, ImmCheck<2, ImmCheck0_1>]>;
}

//
// lookup table expand one register
//
let SMETargetGuard = "sme2" in {
  def SVLUTI2_LANE_ZT : Inst<"svluti2_lane_zt_{d}", "di[i", "cUcsUsiUimbhf", MergeNone, "aarch64_sme_luti2_lane_zt", [IsStreaming, IsInZT0], [ImmCheck<0, ImmCheck0_0>, ImmCheck<2, ImmCheck0_15>]>;
  def SVLUTI4_LANE_ZT : Inst<"svluti4_lane_zt_{d}", "di[i", "cUcsUsiUimbhf", MergeNone, "aarch64_sme_luti4_lane_zt", [IsStreaming, IsInZT0], [ImmCheck<0, ImmCheck0_0>, ImmCheck<2, ImmCheck0_7>]>;
}

//
// lookup table expand two contiguous registers
//
let SMETargetGuard = "sme2" in {
  def SVLUTI2_LANE_ZT_X2 : Inst<"svluti2_lane_zt_{d}_x2", "2.di[i", "cUcsUsiUimbhf", MergeNone, "aarch64_sme_luti2_lane_zt_x2", [IsStreaming, IsInZT0], [ImmCheck<0, ImmCheck0_0>, ImmCheck<2, ImmCheck0_7>]>;
  def SVLUTI4_LANE_ZT_X2 : Inst<"svluti4_lane_zt_{d}_x2", "2.di[i", "cUcsUsiUimbhf", MergeNone, "aarch64_sme_luti4_lane_zt_x2", [IsStreaming, IsInZT0], [ImmCheck<0, ImmCheck0_0>, ImmCheck<2, ImmCheck0_3>]>;
}

````
- **L817 EN**: Separator comment used for visual grouping.
  **L817 CN**: 用于视觉分组的分隔注释。
- **L818 EN**: Comment explains nearby logic, constraints, or intent: `lookup table expand four contiguous registers`.
  **L818 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lookup table expand four contiguous registers`。
- **L819 EN**: Separator comment used for visual grouping.
  **L819 CN**: 用于视觉分组的分隔注释。
- **L820 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L820 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L821 EN**: Declares TableGen def record `SVLUTI2_LANE_ZT_X4`.
  **L821 CN**: 声明 TableGen def 记录 `SVLUTI2_LANE_ZT_X4`。
- **L822 EN**: Declares TableGen def record `SVLUTI4_LANE_ZT_X4`.
  **L822 CN**: 声明 TableGen def 记录 `SVLUTI4_LANE_ZT_X4`。
- **L823 EN**: Closes the current lexical scope or compound statement.
  **L823 CN**: 结束当前词法作用域或复合语句块。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L825 EN**: Separator comment used for visual grouping.
  **L825 CN**: 用于视觉分组的分隔注释。
- **L826 EN**: Comment explains nearby logic, constraints, or intent: `lookup table expand one register`.
  **L826 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lookup table expand one register`。
- **L827 EN**: Separator comment used for visual grouping.
  **L827 CN**: 用于视觉分组的分隔注释。
- **L828 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L828 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L829 EN**: Declares TableGen def record `SVLUTI2_LANE_ZT`.
  **L829 CN**: 声明 TableGen def 记录 `SVLUTI2_LANE_ZT`。
- **L830 EN**: Declares TableGen def record `SVLUTI4_LANE_ZT`.
  **L830 CN**: 声明 TableGen def 记录 `SVLUTI4_LANE_ZT`。
- **L831 EN**: Closes the current lexical scope or compound statement.
  **L831 CN**: 结束当前词法作用域或复合语句块。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L833 EN**: Separator comment used for visual grouping.
  **L833 CN**: 用于视觉分组的分隔注释。
- **L834 EN**: Comment explains nearby logic, constraints, or intent: `lookup table expand two contiguous registers`.
  **L834 CN**: 注释解释附近代码的逻辑、约束或设计意图：`lookup table expand two contiguous registers`。
- **L835 EN**: Separator comment used for visual grouping.
  **L835 CN**: 用于视觉分组的分隔注释。
- **L836 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2" in {`.
  **L836 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2" in {`。
- **L837 EN**: Declares TableGen def record `SVLUTI2_LANE_ZT_X2`.
  **L837 CN**: 声明 TableGen def 记录 `SVLUTI2_LANE_ZT_X2`。
- **L838 EN**: Declares TableGen def record `SVLUTI4_LANE_ZT_X2`.
  **L838 CN**: 声明 TableGen def 记录 `SVLUTI4_LANE_ZT_X2`。
- **L839 EN**: Closes the current lexical scope or compound statement.
  **L839 CN**: 结束当前词法作用域或复合语句块。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 841-864

````tablegen
//
// SME2 FP8 instructions
//

// FDOT
let SMETargetGuard = "sme-f8f32" in {
  def SVDOT_LANE_FP8_ZA32_VG1x2 : Inst<"svdot_lane_za32[_mf8]_vg1x2", "vm2di>", "m", MergeNone, "aarch64_sme_fp8_fdot_lane_za32_vg1x2", [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_3>]>;
  def SVDOT_LANE_FP8_ZA32_VG1x4 : Inst<"svdot_lane_za32[_mf8]_vg1x4", "vm4di>", "m", MergeNone, "aarch64_sme_fp8_fdot_lane_za32_vg1x4", [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_3>]>;

  def SVVDOTB_LANE_FP8_ZA32_VG1x4 : Inst<"svvdotb_lane_za32[_mf8]_vg1x4", "vm2di>", "m", MergeNone, "aarch64_sme_fp8_fvdotb_lane_za32_vg1x4", [IsOverloadNone, IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;
  def SVVDOTT_LANE_FP8_ZA32_VG1x4 : Inst<"svvdott_lane_za32[_mf8]_vg1x4", "vm2di>", "m", MergeNone, "aarch64_sme_fp8_fvdott_lane_za32_vg1x4", [IsOverloadNone, IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_3>]>;

  def SVDOT_SINGLE_FP8_ZA32_VG1x2 : Inst<"svdot[_single]_za32[_mf8]_vg1x2", "vm2d>", "m", MergeNone, "aarch64_sme_fp8_fdot_single_za32_vg1x2", [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVDOT_SINGLE_FP8_ZA32_VG1x4 : Inst<"svdot[_single]_za32[_mf8]_vg1x4", "vm4d>", "m", MergeNone, "aarch64_sme_fp8_fdot_single_za32_vg1x4", [IsStreaming, IsInOutZA, IsOverloadNone], []>;

  def SVDOT_MULTI_FP8_ZA32_VG1x2 : Inst<"svdot_za32[_mf8]_vg1x2", "vm22>", "m", MergeNone, "aarch64_sme_fp8_fdot_multi_za32_vg1x2", [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVDOT_MULTI_FP8_ZA32_VG1x4 : Inst<"svdot_za32[_mf8]_vg1x4", "vm44>", "m", MergeNone, "aarch64_sme_fp8_fdot_multi_za32_vg1x4", [IsStreaming, IsInOutZA, IsOverloadNone], []>;
}

let SMETargetGuard = "sme-f8f16" in {
  def SVDOT_LANE_FP8_ZA16_VG1x2 : Inst<"svdot_lane_za16[_mf8]_vg1x2", "vm2di>", "m", MergeNone, "aarch64_sme_fp8_fdot_lane_za16_vg1x2", [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_7>]>;
  def SVDOT_LANE_FP8_ZA16_VG1x4 : Inst<"svdot_lane_za16[_mf8]_vg1x4", "vm4di>", "m", MergeNone, "aarch64_sme_fp8_fdot_lane_za16_vg1x4", [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_7>]>;

  def SVVDOT_LANE_FP8_ZA16_VG1x2 : Inst<"svvdot_lane_za16[_mf8]_vg1x2", "vm2di>", "m", MergeNone, "aarch64_sme_fp8_fvdot_lane_za16_vg1x2", [IsOverloadNone, IsStreaming, IsInOutZA], [ImmCheck<3, ImmCheck0_7>]>;
````
- **L841 EN**: Separator comment used for visual grouping.
  **L841 CN**: 用于视觉分组的分隔注释。
- **L842 EN**: Comment explains nearby logic, constraints, or intent: `SME2 FP8 instructions`.
  **L842 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 FP8 instructions`。
- **L843 EN**: Separator comment used for visual grouping.
  **L843 CN**: 用于视觉分组的分隔注释。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L845 EN**: Comment explains nearby logic, constraints, or intent: `FDOT`.
  **L845 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FDOT`。
- **L846 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f8f32" in {`.
  **L846 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f8f32" in {`。
- **L847 EN**: Declares TableGen def record `SVDOT_LANE_FP8_ZA32_VG1x2`.
  **L847 CN**: 声明 TableGen def 记录 `SVDOT_LANE_FP8_ZA32_VG1x2`。
- **L848 EN**: Declares TableGen def record `SVDOT_LANE_FP8_ZA32_VG1x4`.
  **L848 CN**: 声明 TableGen def 记录 `SVDOT_LANE_FP8_ZA32_VG1x4`。
- **L849 EN**: Blank line separating nearby declarations or logic blocks.
  **L849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L850 EN**: Declares TableGen def record `SVVDOTB_LANE_FP8_ZA32_VG1x4`.
  **L850 CN**: 声明 TableGen def 记录 `SVVDOTB_LANE_FP8_ZA32_VG1x4`。
- **L851 EN**: Declares TableGen def record `SVVDOTT_LANE_FP8_ZA32_VG1x4`.
  **L851 CN**: 声明 TableGen def 记录 `SVVDOTT_LANE_FP8_ZA32_VG1x4`。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Declares TableGen def record `SVDOT_SINGLE_FP8_ZA32_VG1x2`.
  **L853 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_FP8_ZA32_VG1x2`。
- **L854 EN**: Declares TableGen def record `SVDOT_SINGLE_FP8_ZA32_VG1x4`.
  **L854 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_FP8_ZA32_VG1x4`。
- **L855 EN**: Blank line separating nearby declarations or logic blocks.
  **L855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L856 EN**: Declares TableGen def record `SVDOT_MULTI_FP8_ZA32_VG1x2`.
  **L856 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_FP8_ZA32_VG1x2`。
- **L857 EN**: Declares TableGen def record `SVDOT_MULTI_FP8_ZA32_VG1x4`.
  **L857 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_FP8_ZA32_VG1x4`。
- **L858 EN**: Closes the current lexical scope or compound statement.
  **L858 CN**: 结束当前词法作用域或复合语句块。
- **L859 EN**: Blank line separating nearby declarations or logic blocks.
  **L859 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L860 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f8f16" in {`.
  **L860 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f8f16" in {`。
- **L861 EN**: Declares TableGen def record `SVDOT_LANE_FP8_ZA16_VG1x2`.
  **L861 CN**: 声明 TableGen def 记录 `SVDOT_LANE_FP8_ZA16_VG1x2`。
- **L862 EN**: Declares TableGen def record `SVDOT_LANE_FP8_ZA16_VG1x4`.
  **L862 CN**: 声明 TableGen def 记录 `SVDOT_LANE_FP8_ZA16_VG1x4`。
- **L863 EN**: Blank line separating nearby declarations or logic blocks.
  **L863 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L864 EN**: Declares TableGen def record `SVVDOT_LANE_FP8_ZA16_VG1x2`.
  **L864 CN**: 声明 TableGen def 记录 `SVVDOT_LANE_FP8_ZA16_VG1x2`。

### Lines 865-888

````tablegen

  def SVDOT_SINGLE_FP8_ZA16_VG1x2 : Inst<"svdot[_single]_za16[_mf8]_vg1x2", "vm2d>", "m", MergeNone, "aarch64_sme_fp8_fdot_single_za16_vg1x2", [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVDOT_SINGLE_FP8_ZA16_VG1x4 : Inst<"svdot[_single]_za16[_mf8]_vg1x4", "vm4d>", "m", MergeNone, "aarch64_sme_fp8_fdot_single_za16_vg1x4", [IsStreaming, IsInOutZA, IsOverloadNone], []>;

  def SVDOT_MULTI_FP8_ZA16_VG1x2 : Inst<"svdot_za16[_mf8]_vg1x2", "vm22>", "m", MergeNone, "aarch64_sme_fp8_fdot_multi_za16_vg1x2", [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVDOT_MULTI_FP8_ZA16_VG1x4 : Inst<"svdot_za16[_mf8]_vg1x4", "vm44>", "m", MergeNone, "aarch64_sme_fp8_fdot_multi_za16_vg1x4", [IsStreaming, IsInOutZA, IsOverloadNone], []>;
}

////////////////////////////////////////////////////////////////////////////////
// SME2p1 - FMOPA, FMOPS (non-widening)
let SMETargetGuard = "sme-b16b16" in {
  def SVMOPA_BF16_NW : SInst<"svmopa_za16[_bf16]_m", "viPPdd", "b",
                             MergeNone, "aarch64_sme_mopa",
                             [IsStreaming, IsInOutZA],
                             [ImmCheck<0, ImmCheck0_1>]>;
  def SVMOPS_BF16_NW : SInst<"svmops_za16[_bf16]_m", "viPPdd", "b",
                             MergeNone, "aarch64_sme_mops",
                             [IsStreaming, IsInOutZA],
                             [ImmCheck<0, ImmCheck0_1>]>;
}

let SMETargetGuard = "sme-f16f16" in {
  def SVMOPA_F16_NW : SInst<"svmopa_za16[_f16]_m", "viPPdd", "h",
                            MergeNone, "aarch64_sme_mopa",
````
- **L865 EN**: Blank line separating nearby declarations or logic blocks.
  **L865 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L866 EN**: Declares TableGen def record `SVDOT_SINGLE_FP8_ZA16_VG1x2`.
  **L866 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_FP8_ZA16_VG1x2`。
- **L867 EN**: Declares TableGen def record `SVDOT_SINGLE_FP8_ZA16_VG1x4`.
  **L867 CN**: 声明 TableGen def 记录 `SVDOT_SINGLE_FP8_ZA16_VG1x4`。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L869 EN**: Declares TableGen def record `SVDOT_MULTI_FP8_ZA16_VG1x2`.
  **L869 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_FP8_ZA16_VG1x2`。
- **L870 EN**: Declares TableGen def record `SVDOT_MULTI_FP8_ZA16_VG1x4`.
  **L870 CN**: 声明 TableGen def 记录 `SVDOT_MULTI_FP8_ZA16_VG1x4`。
- **L871 EN**: Closes the current lexical scope or compound statement.
  **L871 CN**: 结束当前词法作用域或复合语句块。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Separator comment used for visual grouping.
  **L873 CN**: 用于视觉分组的分隔注释。
- **L874 EN**: Comment explains nearby logic, constraints, or intent: `SME2p1 - FMOPA, FMOPS (non-widening)`.
  **L874 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2p1 - FMOPA, FMOPS (non-widening)`。
- **L875 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-b16b16" in {`.
  **L875 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-b16b16" in {`。
- **L876 EN**: Declares TableGen def record `SVMOPA_BF16_NW`.
  **L876 CN**: 声明 TableGen def 记录 `SVMOPA_BF16_NW`。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, "aarch64_sme_mopa",`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, "aarch64_sme_mopa",`。
- **L878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L878 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L879 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_1>]>;`.
  **L879 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_1>]>;`。
- **L880 EN**: Declares TableGen def record `SVMOPS_BF16_NW`.
  **L880 CN**: 声明 TableGen def 记录 `SVMOPS_BF16_NW`。
- **L881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, "aarch64_sme_mops",`.
  **L881 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, "aarch64_sme_mops",`。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L883 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_1>]>;`.
  **L883 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_1>]>;`。
- **L884 EN**: Closes the current lexical scope or compound statement.
  **L884 CN**: 结束当前词法作用域或复合语句块。
- **L885 EN**: Blank line separating nearby declarations or logic blocks.
  **L885 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L886 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f16f16" in {`.
  **L886 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f16f16" in {`。
- **L887 EN**: Declares TableGen def record `SVMOPA_F16_NW`.
  **L887 CN**: 声明 TableGen def 记录 `SVMOPA_F16_NW`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, "aarch64_sme_mopa",`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, "aarch64_sme_mopa",`。

### Lines 889-912

````tablegen
                            [IsStreaming, IsInOutZA],
                            [ImmCheck<0, ImmCheck0_1>]>;
  def SVMOPS_F16_NW : SInst<"svmops_za16[_f16]_m", "viPPdd", "h",
                            MergeNone, "aarch64_sme_mops",
                            [IsStreaming, IsInOutZA],
                            [ImmCheck<0, ImmCheck0_1>]>;
}


////////////////////////////////////////////////////////////////////////////////
// SME2 - TMOP, SUTMOP, USTMOP

let SMETargetGuard = "sme2,sme-tmop" in {
  def SVTMOPA_ZA32   : Inst<"svtmopa_lane_za32[_{d}_{d}]", "vi2d[i", "hbf",  MergeNone, "aarch64_sme_ftmopa_za32",  [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_3>, ImmCheck<4, ImmCheck0_3>]>;
  def SVSTMOPA_ZA32  : Inst<"svtmopa_lane_za32[_{d}_{d}]", "vi2d[i", "cs",   MergeNone, "aarch64_sme_stmopa_za32",  [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_3>, ImmCheck<4, ImmCheck0_3>]>;
  def SVUTMOPA_ZA32  : Inst<"svtmopa_lane_za32[_{d}_{d}]", "vi2d[i", "UcUs", MergeNone, "aarch64_sme_utmopa_za32",  [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_3>, ImmCheck<4, ImmCheck0_3>]>;
  def SVSUTMOPA_ZA32 : Inst<"svtmopa_lane_za32[_{d}_{3}]", "vi2u[i", "c",    MergeNone, "aarch64_sme_sutmopa_za32", [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_3>, ImmCheck<4, ImmCheck0_3>]>;
  def SVUSTMOPA_ZA32 : Inst<"svtmopa_lane_za32[_{d}_{3}]", "vi2x[i", "Uc",   MergeNone, "aarch64_sme_ustmopa_za32", [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_3>, ImmCheck<4, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-tmop,sme-f16f16" in {
  def SVTMOPA_F16 : Inst<"svtmopa_lane_za16[_{d}_{d}]", "vi2d[i", "h", MergeNone, "aarch64_sme_ftmopa_za16", [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_1>, ImmCheck<4, ImmCheck0_3>]>;
}

````
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L890 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_1>]>;`.
  **L890 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_1>]>;`。
- **L891 EN**: Declares TableGen def record `SVMOPS_F16_NW`.
  **L891 CN**: 声明 TableGen def 记录 `SVMOPS_F16_NW`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, "aarch64_sme_mops",`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, "aarch64_sme_mops",`。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IsStreaming, IsInOutZA],`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IsStreaming, IsInOutZA],`。
- **L894 EN**: Adds a standalone statement or declaration: `[ImmCheck<0, ImmCheck0_1>]>;`.
  **L894 CN**: 添加一条独立语句或声明：`[ImmCheck<0, ImmCheck0_1>]>;`。
- **L895 EN**: Closes the current lexical scope or compound statement.
  **L895 CN**: 结束当前词法作用域或复合语句块。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Blank line separating nearby declarations or logic blocks.
  **L897 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L898 EN**: Separator comment used for visual grouping.
  **L898 CN**: 用于视觉分组的分隔注释。
- **L899 EN**: Comment explains nearby logic, constraints, or intent: `SME2 - TMOP, SUTMOP, USTMOP`.
  **L899 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME2 - TMOP, SUTMOP, USTMOP`。
- **L900 EN**: Blank line separating nearby declarations or logic blocks.
  **L900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L901 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-tmop" in {`.
  **L901 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-tmop" in {`。
- **L902 EN**: Declares TableGen def record `SVTMOPA_ZA32`.
  **L902 CN**: 声明 TableGen def 记录 `SVTMOPA_ZA32`。
- **L903 EN**: Declares TableGen def record `SVSTMOPA_ZA32`.
  **L903 CN**: 声明 TableGen def 记录 `SVSTMOPA_ZA32`。
- **L904 EN**: Declares TableGen def record `SVUTMOPA_ZA32`.
  **L904 CN**: 声明 TableGen def 记录 `SVUTMOPA_ZA32`。
- **L905 EN**: Declares TableGen def record `SVSUTMOPA_ZA32`.
  **L905 CN**: 声明 TableGen def 记录 `SVSUTMOPA_ZA32`。
- **L906 EN**: Declares TableGen def record `SVUSTMOPA_ZA32`.
  **L906 CN**: 声明 TableGen def 记录 `SVUSTMOPA_ZA32`。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-tmop,sme-f16f16" in {`.
  **L909 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-tmop,sme-f16f16" in {`。
- **L910 EN**: Declares TableGen def record `SVTMOPA_F16`.
  **L910 CN**: 声明 TableGen def 记录 `SVTMOPA_F16`。
- **L911 EN**: Closes the current lexical scope or compound statement.
  **L911 CN**: 结束当前词法作用域或复合语句块。
- **L912 EN**: Blank line separating nearby declarations or logic blocks.
  **L912 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 913-936

````tablegen
let SMETargetGuard = "sme2,sme-tmop,sme-b16b16" in {
  def SVTMOPA_BF16 : Inst<"svtmopa_lane_za16[_{d}_{d}]", "vi2d[i", "b", MergeNone, "aarch64_sme_ftmopa_za16", [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_1>, ImmCheck<4, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-tmop,sme-f8f16" in {
  def SVTMOPA_ZA16_FPM : Inst<"svtmopa_lane_za16[_{d}_{d}]", "vi2.dd[i>", "m", MergeNone, "aarch64_sme_ftmopa_za16", [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_1>, ImmCheck<4, ImmCheck0_3>]>;
}

let SMETargetGuard = "sme2,sme-tmop,sme-f8f32" in {
  def SVTMOPA_ZA32_FPM : Inst<"svtmopa_lane_za32[_{d}_{d}]", "vi2.dd[i>", "m", MergeNone, "aarch64_sme_ftmopa_za32", [IsStreaming, IsInOutZA], [ImmCheck<0, ImmCheck0_3>, ImmCheck<4, ImmCheck0_3>]>;
}

multiclass ZAReadz<string n_suffix, string vg_num, string t, string i_prefix, list<ImmCheck> ch> {
  let SMETargetGuard = "sme2p1" in {
    def NAME # _H : SInst<"svreadz_hor_" # n_suffix # "_{d}_vg" # vg_num, vg_num # "im", t,
                          MergeNone, i_prefix # "_horiz_x" # vg_num,
                          [IsStreaming, IsInOutZA], ch>;

    def NAME # _V : SInst<"svreadz_ver_" # n_suffix # "_{d}_vg" # vg_num, vg_num # "im", t,
                          MergeNone, i_prefix # "_vert_x" #vg_num,
                          [IsStreaming, IsInOutZA], ch>;
  }
}

````
- **L913 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-tmop,sme-b16b16" in {`.
  **L913 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-tmop,sme-b16b16" in {`。
- **L914 EN**: Declares TableGen def record `SVTMOPA_BF16`.
  **L914 CN**: 声明 TableGen def 记录 `SVTMOPA_BF16`。
- **L915 EN**: Closes the current lexical scope or compound statement.
  **L915 CN**: 结束当前词法作用域或复合语句块。
- **L916 EN**: Blank line separating nearby declarations or logic blocks.
  **L916 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L917 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-tmop,sme-f8f16" in {`.
  **L917 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-tmop,sme-f8f16" in {`。
- **L918 EN**: Declares TableGen def record `SVTMOPA_ZA16_FPM`.
  **L918 CN**: 声明 TableGen def 记录 `SVTMOPA_ZA16_FPM`。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L921 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2,sme-tmop,sme-f8f32" in {`.
  **L921 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2,sme-tmop,sme-f8f32" in {`。
- **L922 EN**: Declares TableGen def record `SVTMOPA_ZA32_FPM`.
  **L922 CN**: 声明 TableGen def 记录 `SVTMOPA_ZA32_FPM`。
- **L923 EN**: Closes the current lexical scope or compound statement.
  **L923 CN**: 结束当前词法作用域或复合语句块。
- **L924 EN**: Blank line separating nearby declarations or logic blocks.
  **L924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L925 EN**: Declares TableGen multiclass record `ZAReadz`.
  **L925 CN**: 声明 TableGen multiclass 记录 `ZAReadz`。
- **L926 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2p1" in {`.
  **L926 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2p1" in {`。
- **L927 EN**: Declares TableGen def record `NAME # _H`.
  **L927 CN**: 声明 TableGen def 记录 `NAME # _H`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, i_prefix # "_horiz_x" # vg_num,`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, i_prefix # "_horiz_x" # vg_num,`。
- **L929 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA], ch>;`.
  **L929 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA], ch>;`。
- **L930 EN**: Blank line separating nearby declarations or logic blocks.
  **L930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L931 EN**: Declares TableGen def record `NAME # _V`.
  **L931 CN**: 声明 TableGen def 记录 `NAME # _V`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, i_prefix # "_vert_x" #vg_num,`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, i_prefix # "_vert_x" #vg_num,`。
- **L933 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA], ch>;`.
  **L933 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA], ch>;`。
- **L934 EN**: Closes the current lexical scope or compound statement.
  **L934 CN**: 结束当前词法作用域或复合语句块。
- **L935 EN**: Closes the current lexical scope or compound statement.
  **L935 CN**: 结束当前词法作用域或复合语句块。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 937-960

````tablegen
defm SVREADZ_ZA8_X2  : ZAReadz<"za8",  "2", "cUcm",   "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_0>]>;
defm SVREADZ_ZA16_X2 : ZAReadz<"za16", "2", "sUshb", "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_1>]>;
defm SVREADZ_ZA32_X2 : ZAReadz<"za32", "2", "iUif",  "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_3>]>;
defm SVREADZ_ZA64_X2 : ZAReadz<"za64", "2", "lUld",  "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_7>]>;

defm SVREADZ_ZA8_X4  : ZAReadz<"za8",  "4", "cUcm",   "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_0>]>;
defm SVREADZ_ZA16_X4 : ZAReadz<"za16", "4", "sUshb", "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_1>]>;
defm SVREADZ_ZA32_X4 : ZAReadz<"za32", "4", "iUif",  "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_3>]>;
defm SVREADZ_ZA64_X4 : ZAReadz<"za64", "4", "lUld",  "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_7>]>;


multiclass ZAReadzSingle<string n_suffix, string t, string i_prefix, list<ImmCheck> ch> {
  let SMETargetGuard = "sme2p1" in {
    def NAME # _H : SInst<"svreadz_hor_" # n_suffix # "_{d}", "dim", t,
                          MergeNone, i_prefix # "_horiz",
                          [IsStreaming, IsInOutZA], ch>;

    def NAME # _V : SInst<"svreadz_ver_" # n_suffix # "_{d}", "dim", t,
                          MergeNone, i_prefix # "_vert",
                          [IsStreaming, IsInOutZA], ch>;
  }
}

defm SVREADZ_ZA8   : ZAReadzSingle<"za8", "cUcm", "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_0>]>;
````
- **L937 EN**: Declares TableGen defm record `SVREADZ_ZA8_X2`.
  **L937 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA8_X2`。
- **L938 EN**: Declares TableGen defm record `SVREADZ_ZA16_X2`.
  **L938 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA16_X2`。
- **L939 EN**: Declares TableGen defm record `SVREADZ_ZA32_X2`.
  **L939 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA32_X2`。
- **L940 EN**: Declares TableGen defm record `SVREADZ_ZA64_X2`.
  **L940 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA64_X2`。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L942 EN**: Declares TableGen defm record `SVREADZ_ZA8_X4`.
  **L942 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA8_X4`。
- **L943 EN**: Declares TableGen defm record `SVREADZ_ZA16_X4`.
  **L943 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA16_X4`。
- **L944 EN**: Declares TableGen defm record `SVREADZ_ZA32_X4`.
  **L944 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA32_X4`。
- **L945 EN**: Declares TableGen defm record `SVREADZ_ZA64_X4`.
  **L945 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA64_X4`。
- **L946 EN**: Blank line separating nearby declarations or logic blocks.
  **L946 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Declares TableGen multiclass record `ZAReadzSingle`.
  **L948 CN**: 声明 TableGen multiclass 记录 `ZAReadzSingle`。
- **L949 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2p1" in {`.
  **L949 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2p1" in {`。
- **L950 EN**: Declares TableGen def record `NAME # _H`.
  **L950 CN**: 声明 TableGen def 记录 `NAME # _H`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, i_prefix # "_horiz",`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, i_prefix # "_horiz",`。
- **L952 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA], ch>;`.
  **L952 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA], ch>;`。
- **L953 EN**: Blank line separating nearby declarations or logic blocks.
  **L953 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L954 EN**: Declares TableGen def record `NAME # _V`.
  **L954 CN**: 声明 TableGen def 记录 `NAME # _V`。
- **L955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MergeNone, i_prefix # "_vert",`.
  **L955 CN**: 继续一个多行参数列表、初始化器或聚合项：`MergeNone, i_prefix # "_vert",`。
- **L956 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA], ch>;`.
  **L956 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA], ch>;`。
- **L957 EN**: Closes the current lexical scope or compound statement.
  **L957 CN**: 结束当前词法作用域或复合语句块。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Declares TableGen defm record `SVREADZ_ZA8`.
  **L960 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA8`。

### Lines 961-984

````tablegen
defm SVREADZ_ZA16  : ZAReadzSingle<"za16", "sUshb", "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_1>]>;
defm SVREADZ_ZA32  : ZAReadzSingle<"za32", "iUif", "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_3>]>;
defm SVREADZ_ZA64  : ZAReadzSingle<"za64", "lUld", "aarch64_sme_readz", [ImmCheck<0, ImmCheck0_7>]>;
defm SVREADZ_ZA128 : ZAReadzSingle<"za128", "csilUcUiUsUlmbhfd", "aarch64_sme_readz_q", [ImmCheck<0, ImmCheck0_15>]>;

multiclass ZAReadzArray<string vg_num>{
  let SMETargetGuard = "sme2p1" in {
    def NAME # _B : SInst<"svreadz_za8_{d}_vg1x" # vg_num, vg_num # "m", "cUcm", MergeNone, "aarch64_sme_readz_x" # vg_num, [IsStreaming, IsInOutZA]>;
    def NAME # _H : SInst<"svreadz_za16_{d}_vg1x" # vg_num, vg_num # "m", "sUsbh", MergeNone, "aarch64_sme_readz_x" # vg_num, [IsStreaming, IsInOutZA]>;
    def NAME # _S : SInst<"svreadz_za32_{d}_vg1x" # vg_num, vg_num # "m", "iUif", MergeNone, "aarch64_sme_readz_x" # vg_num, [IsStreaming, IsInOutZA]>;
    def NAME # _D : SInst<"svreadz_za64_{d}_vg1x" # vg_num, vg_num # "m", "lUld", MergeNone, "aarch64_sme_readz_x" # vg_num, [IsStreaming, IsInOutZA]>;
  }
}

defm SVREADZ_VG2 :  ZAReadzArray<"2">;
defm SVREADZ_VG4 :  ZAReadzArray<"4">;

let SMETargetGuard = "sme-lutv2" in {
  def SVWRITE_LANE_ZT : SInst<"svwrite_lane_zt[_{d}]", "vidi", "cUcsUsiUilUlfhdb", MergeNone, "aarch64_sme_write_lane_zt", [IsStreaming, IsInOutZT0], [ImmCheck<0, ImmCheck0_0>, ImmCheck<2, ImmCheck1_3>]>;
  def SVWRITE_ZT : SInst<"svwrite_zt[_{d}]", "vid", "cUcsUsiUilUlfhdb", MergeNone, "aarch64_sme_write_zt", [IsStreaming, IsOutZT0], [ImmCheck<0, ImmCheck0_0>]>;
  def SVLUTI4_ZT_X4 : SInst<"svluti4_zt_{d}_x4", "4i2.u", "cUc", MergeNone, "aarch64_sme_luti4_zt_x4", [IsStreaming, IsInZT0], [ImmCheck<0, ImmCheck0_0>]>;
}

let SMETargetGuard = "sme-f8f32" in {
````
- **L961 EN**: Declares TableGen defm record `SVREADZ_ZA16`.
  **L961 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA16`。
- **L962 EN**: Declares TableGen defm record `SVREADZ_ZA32`.
  **L962 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA32`。
- **L963 EN**: Declares TableGen defm record `SVREADZ_ZA64`.
  **L963 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA64`。
- **L964 EN**: Declares TableGen defm record `SVREADZ_ZA128`.
  **L964 CN**: 声明 TableGen defm 记录 `SVREADZ_ZA128`。
- **L965 EN**: Blank line separating nearby declarations or logic blocks.
  **L965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L966 EN**: Declares TableGen multiclass record `ZAReadzArray`.
  **L966 CN**: 声明 TableGen multiclass 记录 `ZAReadzArray`。
- **L967 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme2p1" in {`.
  **L967 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme2p1" in {`。
- **L968 EN**: Declares TableGen def record `NAME # _B`.
  **L968 CN**: 声明 TableGen def 记录 `NAME # _B`。
- **L969 EN**: Declares TableGen def record `NAME # _H`.
  **L969 CN**: 声明 TableGen def 记录 `NAME # _H`。
- **L970 EN**: Declares TableGen def record `NAME # _S`.
  **L970 CN**: 声明 TableGen def 记录 `NAME # _S`。
- **L971 EN**: Declares TableGen def record `NAME # _D`.
  **L971 CN**: 声明 TableGen def 记录 `NAME # _D`。
- **L972 EN**: Closes the current lexical scope or compound statement.
  **L972 CN**: 结束当前词法作用域或复合语句块。
- **L973 EN**: Closes the current lexical scope or compound statement.
  **L973 CN**: 结束当前词法作用域或复合语句块。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Declares TableGen defm record `SVREADZ_VG2`.
  **L975 CN**: 声明 TableGen defm 记录 `SVREADZ_VG2`。
- **L976 EN**: Declares TableGen defm record `SVREADZ_VG4`.
  **L976 CN**: 声明 TableGen defm 记录 `SVREADZ_VG4`。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L978 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-lutv2" in {`.
  **L978 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-lutv2" in {`。
- **L979 EN**: Declares TableGen def record `SVWRITE_LANE_ZT`.
  **L979 CN**: 声明 TableGen def 记录 `SVWRITE_LANE_ZT`。
- **L980 EN**: Declares TableGen def record `SVWRITE_ZT`.
  **L980 CN**: 声明 TableGen def 记录 `SVWRITE_ZT`。
- **L981 EN**: Declares TableGen def record `SVLUTI4_ZT_X4`.
  **L981 CN**: 声明 TableGen def 记录 `SVLUTI4_ZT_X4`。
- **L982 EN**: Closes the current lexical scope or compound statement.
  **L982 CN**: 结束当前词法作用域或复合语句块。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f8f32" in {`.
  **L984 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f8f32" in {`。

### Lines 985-1008

````tablegen
  def SVMOPA_FP8_ZA32 : Inst<"svmopa_za32[_mf8]_m", "viPPdd>", "m", MergeNone, "aarch64_sme_fp8_fmopa_za32",
                             [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<0, ImmCheck0_3>]>;
   // FMLALL (indexed)
  def SVMLA_FP8_LANE_ZA32_VG4x1 : Inst<"svmla_lane_za32[_mf8]_vg4x1", "vmddi>", "m", MergeNone, "aarch64_sme_fp8_fmlall_lane_za32_vg4x1",
                                       [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLA_FP8_LANE_ZA32_VG4x2 : Inst<"svmla_lane_za32[_mf8]_vg4x2", "vm2di>", "m", MergeNone, "aarch64_sme_fp8_fmlall_lane_za32_vg4x2",
                                       [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLA_FP8_LANE_ZA16_VG4x4 : Inst<"svmla_lane_za32[_mf8]_vg4x4", "vm4di>", "m", MergeNone, "aarch64_sme_fp8_fmlall_lane_za32_vg4x4",
                                       [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;
  // FMLALL
  def SVMLA_FP8_ZA32_VG4x1 : Inst<"svmla_za32[_mf8]_vg4x1", "vmdd>", "m", MergeNone, "aarch64_sme_fp8_fmlall_za32_vg4x1",
                                  [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  // FMLALL (single)
  def SVMLA_FP8_SINGLE_ZA32_VG4x2 : Inst<"svmla[_single]_za32[_mf8]_vg4x2", "vm2d>", "m", MergeNone, "aarch64_sme_fp8_fmlall_single_za32_vg4x2",
                                         [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVMLA_FP8_SINGLE_ZA32_VG4x4 : Inst<"svmla[_single]_za32[_mf8]_vg4x4", "vm4d>", "m", MergeNone, "aarch64_sme_fp8_fmlall_single_za32_vg4x4",
                                         [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  // FMLALL (multiple)
  def SVMLA_FP8_MULTI_ZA32_VG4x2 : Inst<"svmla_za32[_mf8]_vg4x2", "vm22>", "m", MergeNone, "aarch64_sme_fp8_fmlall_multi_za32_vg4x2",
                                        [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVMLA_FP8_MULTI_ZA32_VG4x4 : Inst<"svmla_za32[_mf8]_vg4x4", "vm44>", "m", MergeNone, "aarch64_sme_fp8_fmlall_multi_za32_vg4x4",
                                        [IsStreaming, IsInOutZA, IsOverloadNone], []>;
}

````
- **L985 EN**: Declares TableGen def record `SVMOPA_FP8_ZA32`.
  **L985 CN**: 声明 TableGen def 记录 `SVMOPA_FP8_ZA32`。
- **L986 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<0, ImmCheck0_3>]>;`.
  **L986 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<0, ImmCheck0_3>]>;`。
- **L987 EN**: Comment explains nearby logic, constraints, or intent: `FMLALL (indexed)`.
  **L987 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLALL (indexed)`。
- **L988 EN**: Declares TableGen def record `SVMLA_FP8_LANE_ZA32_VG4x1`.
  **L988 CN**: 声明 TableGen def 记录 `SVMLA_FP8_LANE_ZA32_VG4x1`。
- **L989 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`.
  **L989 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`。
- **L990 EN**: Declares TableGen def record `SVMLA_FP8_LANE_ZA32_VG4x2`.
  **L990 CN**: 声明 TableGen def 记录 `SVMLA_FP8_LANE_ZA32_VG4x2`。
- **L991 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`.
  **L991 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`。
- **L992 EN**: Declares TableGen def record `SVMLA_FP8_LANE_ZA16_VG4x4`.
  **L992 CN**: 声明 TableGen def 记录 `SVMLA_FP8_LANE_ZA16_VG4x4`。
- **L993 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`.
  **L993 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`。
- **L994 EN**: Comment explains nearby logic, constraints, or intent: `FMLALL`.
  **L994 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLALL`。
- **L995 EN**: Declares TableGen def record `SVMLA_FP8_ZA32_VG4x1`.
  **L995 CN**: 声明 TableGen def 记录 `SVMLA_FP8_ZA32_VG4x1`。
- **L996 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L996 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L997 EN**: Comment explains nearby logic, constraints, or intent: `FMLALL (single)`.
  **L997 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLALL (single)`。
- **L998 EN**: Declares TableGen def record `SVMLA_FP8_SINGLE_ZA32_VG4x2`.
  **L998 CN**: 声明 TableGen def 记录 `SVMLA_FP8_SINGLE_ZA32_VG4x2`。
- **L999 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L999 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1000 EN**: Declares TableGen def record `SVMLA_FP8_SINGLE_ZA32_VG4x4`.
  **L1000 CN**: 声明 TableGen def 记录 `SVMLA_FP8_SINGLE_ZA32_VG4x4`。
- **L1001 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1001 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1002 EN**: Comment explains nearby logic, constraints, or intent: `FMLALL (multiple)`.
  **L1002 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLALL (multiple)`。
- **L1003 EN**: Declares TableGen def record `SVMLA_FP8_MULTI_ZA32_VG4x2`.
  **L1003 CN**: 声明 TableGen def 记录 `SVMLA_FP8_MULTI_ZA32_VG4x2`。
- **L1004 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1004 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1005 EN**: Declares TableGen def record `SVMLA_FP8_MULTI_ZA32_VG4x4`.
  **L1005 CN**: 声明 TableGen def 记录 `SVMLA_FP8_MULTI_ZA32_VG4x4`。
- **L1006 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1006 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1007 EN**: Closes the current lexical scope or compound statement.
  **L1007 CN**: 结束当前词法作用域或复合语句块。
- **L1008 EN**: Blank line separating nearby declarations or logic blocks.
  **L1008 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1009-1032

````tablegen
let SMETargetGuard = "sme-f8f16" in {
  def SVMOPA_FP8_ZA16 : Inst<"svmopa_za16[_mf8]_m", "viPPdd>", "m", MergeNone, "aarch64_sme_fp8_fmopa_za16",
                             [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<0, ImmCheck0_1>]>;
  // FMLAL (indexed)
  def SVMLA_FP8_LANE_ZA16_VG2x1 : Inst<"svmla_lane_za16[_mf8]_vg2x1", "vmddi>", "m", MergeNone, "aarch64_sme_fp8_fmlal_lane_za16_vg2x1",
                                       [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLA_FP8_LANE_ZA16_VG2x2 : Inst<"svmla_lane_za16[_mf8]_vg2x2", "vm2di>", "m", MergeNone, "aarch64_sme_fp8_fmlal_lane_za16_vg2x2",
                                       [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;
  def SVMLA_FP8_LANE_ZA16_VG2x4 : Inst<"svmla_lane_za16[_mf8]_vg2x4", "vm4di>", "m", MergeNone, "aarch64_sme_fp8_fmlal_lane_za16_vg2x4",
                                       [IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;
  // FMLAL
  def SVMLA_FP8_ZA16_VG2x1 : Inst<"svmla_za16[_mf8]_vg2x1", "vmdd>", "m", MergeNone, "aarch64_sme_fp8_fmlal_za16_vg2x1",
                                  [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  // FMLAL (single)
  def SVMLA_FP8_SINGLE_ZA16_VG2x2 : Inst<"svmla[_single]_za16[_mf8]_vg2x2", "vm2d>", "m", MergeNone, "aarch64_sme_fp8_fmlal_single_za16_vg2x2",
                                         [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVMLA_FP8_SINGLE_ZA16_VG2x4 : Inst<"svmla[_single]_za16[_mf8]_vg2x4", "vm4d>", "m", MergeNone, "aarch64_sme_fp8_fmlal_single_za16_vg2x4",
                                         [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  // FMLAL (multiple)
  def SVMLA_FP8_MULTI_ZA16_VG2x2 : Inst<"svmla_za16[_mf8]_vg2x2", "vm22>", "m", MergeNone, "aarch64_sme_fp8_fmlal_multi_za16_vg2x2",
                                        [IsStreaming, IsInOutZA, IsOverloadNone], []>;
  def SVMLA_FP8_MULTI_ZA16_VG2x4 : Inst<"svmla_za16[_mf8]_vg2x4", "vm44>", "m", MergeNone, "aarch64_sme_fp8_fmlal_multi_za16_vg2x4",
                                        [IsStreaming, IsInOutZA, IsOverloadNone], []>;
}
````
- **L1009 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SMETargetGuard = "sme-f8f16" in {`.
  **L1009 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SMETargetGuard = "sme-f8f16" in {`。
- **L1010 EN**: Declares TableGen def record `SVMOPA_FP8_ZA16`.
  **L1010 CN**: 声明 TableGen def 记录 `SVMOPA_FP8_ZA16`。
- **L1011 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<0, ImmCheck0_1>]>;`.
  **L1011 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<0, ImmCheck0_1>]>;`。
- **L1012 EN**: Comment explains nearby logic, constraints, or intent: `FMLAL (indexed)`.
  **L1012 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLAL (indexed)`。
- **L1013 EN**: Declares TableGen def record `SVMLA_FP8_LANE_ZA16_VG2x1`.
  **L1013 CN**: 声明 TableGen def 记录 `SVMLA_FP8_LANE_ZA16_VG2x1`。
- **L1014 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`.
  **L1014 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`。
- **L1015 EN**: Declares TableGen def record `SVMLA_FP8_LANE_ZA16_VG2x2`.
  **L1015 CN**: 声明 TableGen def 记录 `SVMLA_FP8_LANE_ZA16_VG2x2`。
- **L1016 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`.
  **L1016 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`。
- **L1017 EN**: Declares TableGen def record `SVMLA_FP8_LANE_ZA16_VG2x4`.
  **L1017 CN**: 声明 TableGen def 记录 `SVMLA_FP8_LANE_ZA16_VG2x4`。
- **L1018 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`.
  **L1018 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], [ImmCheck<3, ImmCheck0_15>]>;`。
- **L1019 EN**: Comment explains nearby logic, constraints, or intent: `FMLAL`.
  **L1019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLAL`。
- **L1020 EN**: Declares TableGen def record `SVMLA_FP8_ZA16_VG2x1`.
  **L1020 CN**: 声明 TableGen def 记录 `SVMLA_FP8_ZA16_VG2x1`。
- **L1021 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1021 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1022 EN**: Comment explains nearby logic, constraints, or intent: `FMLAL (single)`.
  **L1022 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLAL (single)`。
- **L1023 EN**: Declares TableGen def record `SVMLA_FP8_SINGLE_ZA16_VG2x2`.
  **L1023 CN**: 声明 TableGen def 记录 `SVMLA_FP8_SINGLE_ZA16_VG2x2`。
- **L1024 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1024 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1025 EN**: Declares TableGen def record `SVMLA_FP8_SINGLE_ZA16_VG2x4`.
  **L1025 CN**: 声明 TableGen def 记录 `SVMLA_FP8_SINGLE_ZA16_VG2x4`。
- **L1026 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1026 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1027 EN**: Comment explains nearby logic, constraints, or intent: `FMLAL (multiple)`.
  **L1027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMLAL (multiple)`。
- **L1028 EN**: Declares TableGen def record `SVMLA_FP8_MULTI_ZA16_VG2x2`.
  **L1028 CN**: 声明 TableGen def 记录 `SVMLA_FP8_MULTI_ZA16_VG2x2`。
- **L1029 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1029 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1030 EN**: Declares TableGen def record `SVMLA_FP8_MULTI_ZA16_VG2x4`.
  **L1030 CN**: 声明 TableGen def 记录 `SVMLA_FP8_MULTI_ZA16_VG2x4`。
- **L1031 EN**: Adds a standalone statement or declaration: `[IsStreaming, IsInOutZA, IsOverloadNone], []>;`.
  **L1031 CN**: 添加一条独立语句或声明：`[IsStreaming, IsInOutZA, IsOverloadNone], []>;`。
- **L1032 EN**: Closes the current lexical scope or compound statement.
  **L1032 CN**: 结束当前词法作用域或复合语句块。

### Lines 1033-1034

````tablegen

} // let SVETargetGuard = InvalidMode
````
- **L1033 EN**: Blank line separating nearby declarations or logic blocks.
  **L1033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1034 EN**: Continues the surrounding expression or declaration: `} // let SVETargetGuard = InvalidMode`.
  **L1034 CN**: 继续构造周围的表达式或声明：`} // let SVETargetGuard = InvalidMode`。

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
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `cond`, `FMOPS`, `FMLALL`, `FMLAL`
- **TableGen records / TableGen 记录**: `ZALoad`, `NAME`, `SVLD1_ZA8`, `SVLD1_ZA16`, `SVLD1_ZA32`, `SVLD1_ZA64`, `SVLD1_ZA128`, `SVLDR_VNUM_ZA`, `SVLDR_ZA`, `ZAStore`, `SVST1_ZA8`, `SVST1_ZA16`, `SVST1_ZA32`, `SVST1_ZA64`, `SVST1_ZA128`, `SVSTR_VNUM_ZA`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
