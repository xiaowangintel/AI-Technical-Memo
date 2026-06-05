# IntrinsicsMips.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsMips.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the MIPS-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsMips` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````tablegen
//===- IntrinsicsMips.td - Defines Mips intrinsics ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the MIPS-specific intrinsics.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// MIPS DSP data types
def mips_v2q15_ty: LLVMType<v2i16>;
def mips_v4q7_ty: LLVMType<v4i8>;
def mips_q31_ty: LLVMType<i32>;

let TargetPrefix = "mips" in {  // All intrinsics start with "llvm.mips.".

//===----------------------------------------------------------------------===//
// MIPS DSP Rev 1

//===----------------------------------------------------------------------===//
// Addition/subtraction

def int_mips_addu_qb : ClangBuiltin<"__builtin_mips_addu_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the MIPS-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the MIPS-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Comment explains nearby logic, invariants, or intent: `MIPS DSP data types`.
  **L14 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MIPS DSP data types`。
- **L15 EN**: Declares TableGen def `mips_v2q15_ty`.
  **L15 CN**: 声明 TableGen def `mips_v2q15_ty`。
- **L16 EN**: Declares TableGen def `mips_v4q7_ty`.
  **L16 CN**: 声明 TableGen def `mips_v4q7_ty`。
- **L17 EN**: Declares TableGen def `mips_q31_ty`.
  **L17 CN**: 声明 TableGen def `mips_q31_ty`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L19 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Banner comment marking a file or section boundary.
  **L21 CN**: 横幅注释，用于标记文件或章节边界。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `MIPS DSP Rev 1`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MIPS DSP Rev 1`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Banner comment marking a file or section boundary.
  **L24 CN**: 横幅注释，用于标记文件或章节边界。
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `Addition/subtraction`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Addition/subtraction`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Declares TableGen def `int_mips_addu_qb`.
  **L27 CN**: 声明 TableGen def `int_mips_addu_qb`。
- **L28 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L28 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`。

### Lines 29-56

````tablegen
            [Commutative, IntrNoMem]>;
def int_mips_addu_s_qb : ClangBuiltin<"__builtin_mips_addu_s_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],
            [Commutative, IntrNoMem]>;
def int_mips_subu_qb : ClangBuiltin<"__builtin_mips_subu_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_subu_s_qb : ClangBuiltin<"__builtin_mips_subu_s_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;

def int_mips_addq_ph : ClangBuiltin<"__builtin_mips_addq_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],
            [Commutative, IntrNoMem]>;
def int_mips_addq_s_ph : ClangBuiltin<"__builtin_mips_addq_s_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],
            [Commutative, IntrNoMem]>;
def int_mips_subq_ph : ClangBuiltin<"__builtin_mips_subq_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;
def int_mips_subq_s_ph : ClangBuiltin<"__builtin_mips_subq_s_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;

def int_mips_madd: ClangBuiltin<"__builtin_mips_madd">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, Commutative]>;
def int_mips_maddu: ClangBuiltin<"__builtin_mips_maddu">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, Commutative]>;

def int_mips_msub: ClangBuiltin<"__builtin_mips_msub">,
````
- **L29 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L29 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L30 EN**: Declares TableGen def `int_mips_addu_s_qb`.
  **L30 CN**: 声明 TableGen def `int_mips_addu_s_qb`。
- **L31 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L31 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`。
- **L32 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L32 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L33 EN**: Declares TableGen def `int_mips_subu_qb`.
  **L33 CN**: 声明 TableGen def `int_mips_subu_qb`。
- **L34 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`.
  **L34 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`。
- **L35 EN**: Declares TableGen def `int_mips_subu_s_qb`.
  **L35 CN**: 声明 TableGen def `int_mips_subu_s_qb`。
- **L36 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`.
  **L36 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Declares TableGen def `int_mips_addq_ph`.
  **L38 CN**: 声明 TableGen def `int_mips_addq_ph`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`。
- **L40 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L40 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L41 EN**: Declares TableGen def `int_mips_addq_s_ph`.
  **L41 CN**: 声明 TableGen def `int_mips_addq_s_ph`。
- **L42 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`.
  **L42 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`。
- **L43 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L43 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L44 EN**: Declares TableGen def `int_mips_subq_ph`.
  **L44 CN**: 声明 TableGen def `int_mips_subq_ph`。
- **L45 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`.
  **L45 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`。
- **L46 EN**: Declares TableGen def `int_mips_subq_s_ph`.
  **L46 CN**: 声明 TableGen def `int_mips_subq_s_ph`。
- **L47 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`.
  **L47 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L49 EN**: Declares TableGen def `int_mips_madd`.
  **L49 CN**: 声明 TableGen def `int_mips_madd`。
- **L50 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L50 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L51 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L51 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L52 EN**: Declares TableGen def `int_mips_maddu`.
  **L52 CN**: 声明 TableGen def `int_mips_maddu`。
- **L53 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L53 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L54 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L54 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Declares TableGen def `int_mips_msub`.
  **L56 CN**: 声明 TableGen def `int_mips_msub`。

### Lines 57-84

````tablegen
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem]>;
def int_mips_msubu: ClangBuiltin<"__builtin_mips_msubu">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem]>;

def int_mips_addq_s_w: ClangBuiltin<"__builtin_mips_addq_s_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;
def int_mips_subq_s_w: ClangBuiltin<"__builtin_mips_subq_s_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], []>;

def int_mips_addsc: ClangBuiltin<"__builtin_mips_addsc">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [Commutative]>;
def int_mips_addwc: ClangBuiltin<"__builtin_mips_addwc">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [Commutative]>;

def int_mips_modsub: ClangBuiltin<"__builtin_mips_modsub">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;

def int_mips_raddu_w_qb: ClangBuiltin<"__builtin_mips_raddu_w_qb">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty], [IntrNoMem]>;

//===----------------------------------------------------------------------===//
// Absolute value

def int_mips_absq_s_ph: ClangBuiltin<"__builtin_mips_absq_s_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty], []>;
def int_mips_absq_s_w: ClangBuiltin<"__builtin_mips_absq_s_w">,
````
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L58 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L58 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L59 EN**: Declares TableGen def `int_mips_msubu`.
  **L59 CN**: 声明 TableGen def `int_mips_msubu`。
- **L60 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L60 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L61 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L61 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Declares TableGen def `int_mips_addq_s_w`.
  **L63 CN**: 声明 TableGen def `int_mips_addq_s_w`。
- **L64 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;`.
  **L64 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;`。
- **L65 EN**: Declares TableGen def `int_mips_subq_s_w`.
  **L65 CN**: 声明 TableGen def `int_mips_subq_s_w`。
- **L66 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], []>;`.
  **L66 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], []>;`。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Declares TableGen def `int_mips_addsc`.
  **L68 CN**: 声明 TableGen def `int_mips_addsc`。
- **L69 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [Commutative]>;`.
  **L69 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [Commutative]>;`。
- **L70 EN**: Declares TableGen def `int_mips_addwc`.
  **L70 CN**: 声明 TableGen def `int_mips_addwc`。
- **L71 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [Commutative]>;`.
  **L71 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [Commutative]>;`。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L73 EN**: Declares TableGen def `int_mips_modsub`.
  **L73 CN**: 声明 TableGen def `int_mips_modsub`。
- **L74 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L74 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Declares TableGen def `int_mips_raddu_w_qb`.
  **L76 CN**: 声明 TableGen def `int_mips_raddu_w_qb`。
- **L77 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L77 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L79 EN**: Banner comment marking a file or section boundary.
  **L79 CN**: 横幅注释，用于标记文件或章节边界。
- **L80 EN**: Comment explains nearby logic, invariants, or intent: `Absolute value`.
  **L80 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Absolute value`。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Declares TableGen def `int_mips_absq_s_ph`.
  **L82 CN**: 声明 TableGen def `int_mips_absq_s_ph`。
- **L83 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty], []>;`.
  **L83 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty], []>;`。
- **L84 EN**: Declares TableGen def `int_mips_absq_s_w`.
  **L84 CN**: 声明 TableGen def `int_mips_absq_s_w`。

### Lines 85-112

````tablegen
  Intrinsic<[mips_q31_ty], [mips_q31_ty], []>;

//===----------------------------------------------------------------------===//
// Precision reduce/expand

def int_mips_precrq_qb_ph: ClangBuiltin<"__builtin_mips_precrq_qb_ph">,
  Intrinsic<[llvm_v4i8_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;
def int_mips_precrqu_s_qb_ph: ClangBuiltin<"__builtin_mips_precrqu_s_qb_ph">,
  Intrinsic<[llvm_v4i8_ty], [mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_precrq_ph_w: ClangBuiltin<"__builtin_mips_precrq_ph_w">,
  Intrinsic<[mips_v2q15_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;
def int_mips_precrq_rs_ph_w: ClangBuiltin<"__builtin_mips_precrq_rs_ph_w">,
  Intrinsic<[mips_v2q15_ty], [mips_q31_ty, mips_q31_ty], []>;
def int_mips_preceq_w_phl: ClangBuiltin<"__builtin_mips_preceq_w_phl">,
  Intrinsic<[mips_q31_ty], [mips_v2q15_ty], [IntrNoMem]>;
def int_mips_preceq_w_phr: ClangBuiltin<"__builtin_mips_preceq_w_phr">,
  Intrinsic<[mips_q31_ty], [mips_v2q15_ty], [IntrNoMem]>;
def int_mips_precequ_ph_qbl: ClangBuiltin<"__builtin_mips_precequ_ph_qbl">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_precequ_ph_qbr: ClangBuiltin<"__builtin_mips_precequ_ph_qbr">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_precequ_ph_qbla: ClangBuiltin<"__builtin_mips_precequ_ph_qbla">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_precequ_ph_qbra: ClangBuiltin<"__builtin_mips_precequ_ph_qbra">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_preceu_ph_qbl: ClangBuiltin<"__builtin_mips_preceu_ph_qbl">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_preceu_ph_qbr: ClangBuiltin<"__builtin_mips_preceu_ph_qbr">,
````
- **L85 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty], []>;`.
  **L85 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty], []>;`。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L87 EN**: Banner comment marking a file or section boundary.
  **L87 CN**: 横幅注释，用于标记文件或章节边界。
- **L88 EN**: Comment explains nearby logic, invariants, or intent: `Precision reduce/expand`.
  **L88 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Precision reduce/expand`。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L90 EN**: Declares TableGen def `int_mips_precrq_qb_ph`.
  **L90 CN**: 声明 TableGen def `int_mips_precrq_qb_ph`。
- **L91 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`.
  **L91 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`。
- **L92 EN**: Declares TableGen def `int_mips_precrqu_s_qb_ph`.
  **L92 CN**: 声明 TableGen def `int_mips_precrqu_s_qb_ph`。
- **L93 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L93 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L94 EN**: Declares TableGen def `int_mips_precrq_ph_w`.
  **L94 CN**: 声明 TableGen def `int_mips_precrq_ph_w`。
- **L95 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;`.
  **L95 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;`。
- **L96 EN**: Declares TableGen def `int_mips_precrq_rs_ph_w`.
  **L96 CN**: 声明 TableGen def `int_mips_precrq_rs_ph_w`。
- **L97 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_q31_ty, mips_q31_ty], []>;`.
  **L97 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_q31_ty, mips_q31_ty], []>;`。
- **L98 EN**: Declares TableGen def `int_mips_preceq_w_phl`.
  **L98 CN**: 声明 TableGen def `int_mips_preceq_w_phl`。
- **L99 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_v2q15_ty], [IntrNoMem]>;`.
  **L99 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_v2q15_ty], [IntrNoMem]>;`。
- **L100 EN**: Declares TableGen def `int_mips_preceq_w_phr`.
  **L100 CN**: 声明 TableGen def `int_mips_preceq_w_phr`。
- **L101 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_v2q15_ty], [IntrNoMem]>;`.
  **L101 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_v2q15_ty], [IntrNoMem]>;`。
- **L102 EN**: Declares TableGen def `int_mips_precequ_ph_qbl`.
  **L102 CN**: 声明 TableGen def `int_mips_precequ_ph_qbl`。
- **L103 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L103 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L104 EN**: Declares TableGen def `int_mips_precequ_ph_qbr`.
  **L104 CN**: 声明 TableGen def `int_mips_precequ_ph_qbr`。
- **L105 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L105 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L106 EN**: Declares TableGen def `int_mips_precequ_ph_qbla`.
  **L106 CN**: 声明 TableGen def `int_mips_precequ_ph_qbla`。
- **L107 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L107 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L108 EN**: Declares TableGen def `int_mips_precequ_ph_qbra`.
  **L108 CN**: 声明 TableGen def `int_mips_precequ_ph_qbra`。
- **L109 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L109 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L110 EN**: Declares TableGen def `int_mips_preceu_ph_qbl`.
  **L110 CN**: 声明 TableGen def `int_mips_preceu_ph_qbl`。
- **L111 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L111 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L112 EN**: Declares TableGen def `int_mips_preceu_ph_qbr`.
  **L112 CN**: 声明 TableGen def `int_mips_preceu_ph_qbr`。

### Lines 113-140

````tablegen
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_preceu_ph_qbla: ClangBuiltin<"__builtin_mips_preceu_ph_qbla">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_preceu_ph_qbra: ClangBuiltin<"__builtin_mips_preceu_ph_qbra">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;

//===----------------------------------------------------------------------===//
// Shift

def int_mips_shll_qb: ClangBuiltin<"__builtin_mips_shll_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], []>;
def int_mips_shrl_qb: ClangBuiltin<"__builtin_mips_shrl_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_shll_ph: ClangBuiltin<"__builtin_mips_shll_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], []>;
def int_mips_shll_s_ph: ClangBuiltin<"__builtin_mips_shll_s_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], []>;
def int_mips_shra_ph: ClangBuiltin<"__builtin_mips_shra_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_shra_r_ph: ClangBuiltin<"__builtin_mips_shra_r_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_shll_s_w: ClangBuiltin<"__builtin_mips_shll_s_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, llvm_i32_ty], []>;
def int_mips_shra_r_w: ClangBuiltin<"__builtin_mips_shra_r_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_shilo: ClangBuiltin<"__builtin_mips_shilo">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty], [IntrNoMem]>;

````
- **L113 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L113 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L114 EN**: Declares TableGen def `int_mips_preceu_ph_qbla`.
  **L114 CN**: 声明 TableGen def `int_mips_preceu_ph_qbla`。
- **L115 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L115 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L116 EN**: Declares TableGen def `int_mips_preceu_ph_qbra`.
  **L116 CN**: 声明 TableGen def `int_mips_preceu_ph_qbra`。
- **L117 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`.
  **L117 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty], [IntrNoMem]>;`。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L119 EN**: Banner comment marking a file or section boundary.
  **L119 CN**: 横幅注释，用于标记文件或章节边界。
- **L120 EN**: Comment explains nearby logic, invariants, or intent: `Shift`.
  **L120 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Shift`。
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L122 EN**: Declares TableGen def `int_mips_shll_qb`.
  **L122 CN**: 声明 TableGen def `int_mips_shll_qb`。
- **L123 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], []>;`.
  **L123 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], []>;`。
- **L124 EN**: Declares TableGen def `int_mips_shrl_qb`.
  **L124 CN**: 声明 TableGen def `int_mips_shrl_qb`。
- **L125 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L125 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L126 EN**: Declares TableGen def `int_mips_shll_ph`.
  **L126 CN**: 声明 TableGen def `int_mips_shll_ph`。
- **L127 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], []>;`.
  **L127 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], []>;`。
- **L128 EN**: Declares TableGen def `int_mips_shll_s_ph`.
  **L128 CN**: 声明 TableGen def `int_mips_shll_s_ph`。
- **L129 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], []>;`.
  **L129 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], []>;`。
- **L130 EN**: Declares TableGen def `int_mips_shra_ph`.
  **L130 CN**: 声明 TableGen def `int_mips_shra_ph`。
- **L131 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L131 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L132 EN**: Declares TableGen def `int_mips_shra_r_ph`.
  **L132 CN**: 声明 TableGen def `int_mips_shra_r_ph`。
- **L133 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L133 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L134 EN**: Declares TableGen def `int_mips_shll_s_w`.
  **L134 CN**: 声明 TableGen def `int_mips_shll_s_w`。
- **L135 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, llvm_i32_ty], []>;`.
  **L135 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, llvm_i32_ty], []>;`。
- **L136 EN**: Declares TableGen def `int_mips_shra_r_w`.
  **L136 CN**: 声明 TableGen def `int_mips_shra_r_w`。
- **L137 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L137 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L138 EN**: Declares TableGen def `int_mips_shilo`.
  **L138 CN**: 声明 TableGen def `int_mips_shilo`。
- **L139 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L139 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 141-168

````tablegen
//===----------------------------------------------------------------------===//
// Multiplication

def int_mips_muleu_s_ph_qbl: ClangBuiltin<"__builtin_mips_muleu_s_ph_qbl">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty, mips_v2q15_ty], []>;
def int_mips_muleu_s_ph_qbr: ClangBuiltin<"__builtin_mips_muleu_s_ph_qbr">,
  Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty, mips_v2q15_ty], []>;
def int_mips_mulq_rs_ph: ClangBuiltin<"__builtin_mips_mulq_rs_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;
def int_mips_muleq_s_w_phl: ClangBuiltin<"__builtin_mips_muleq_s_w_phl">,
  Intrinsic<[mips_q31_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;
def int_mips_muleq_s_w_phr: ClangBuiltin<"__builtin_mips_muleq_s_w_phr">,
  Intrinsic<[mips_q31_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;
def int_mips_mulsaq_s_w_ph: ClangBuiltin<"__builtin_mips_mulsaq_s_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_maq_s_w_phl: ClangBuiltin<"__builtin_mips_maq_s_w_phl">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_maq_s_w_phr: ClangBuiltin<"__builtin_mips_maq_s_w_phr">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_maq_sa_w_phl: ClangBuiltin<"__builtin_mips_maq_sa_w_phl">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_maq_sa_w_phr: ClangBuiltin<"__builtin_mips_maq_sa_w_phr">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_mult: ClangBuiltin<"__builtin_mips_mult">,
  Intrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, Commutative]>;
def int_mips_multu: ClangBuiltin<"__builtin_mips_multu">,
  Intrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty],
````
- **L141 EN**: Banner comment marking a file or section boundary.
  **L141 CN**: 横幅注释，用于标记文件或章节边界。
- **L142 EN**: Comment explains nearby logic, invariants, or intent: `Multiplication`.
  **L142 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multiplication`。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Declares TableGen def `int_mips_muleu_s_ph_qbl`.
  **L144 CN**: 声明 TableGen def `int_mips_muleu_s_ph_qbl`。
- **L145 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty, mips_v2q15_ty], []>;`.
  **L145 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty, mips_v2q15_ty], []>;`。
- **L146 EN**: Declares TableGen def `int_mips_muleu_s_ph_qbr`.
  **L146 CN**: 声明 TableGen def `int_mips_muleu_s_ph_qbr`。
- **L147 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty, mips_v2q15_ty], []>;`.
  **L147 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_v4i8_ty, mips_v2q15_ty], []>;`。
- **L148 EN**: Declares TableGen def `int_mips_mulq_rs_ph`.
  **L148 CN**: 声明 TableGen def `int_mips_mulq_rs_ph`。
- **L149 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`.
  **L149 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`。
- **L150 EN**: Declares TableGen def `int_mips_muleq_s_w_phl`.
  **L150 CN**: 声明 TableGen def `int_mips_muleq_s_w_phl`。
- **L151 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`.
  **L151 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`。
- **L152 EN**: Declares TableGen def `int_mips_muleq_s_w_phr`.
  **L152 CN**: 声明 TableGen def `int_mips_muleq_s_w_phr`。
- **L153 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`.
  **L153 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`。
- **L154 EN**: Declares TableGen def `int_mips_mulsaq_s_w_ph`.
  **L154 CN**: 声明 TableGen def `int_mips_mulsaq_s_w_ph`。
- **L155 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L155 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L156 EN**: Declares TableGen def `int_mips_maq_s_w_phl`.
  **L156 CN**: 声明 TableGen def `int_mips_maq_s_w_phl`。
- **L157 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L157 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L158 EN**: Declares TableGen def `int_mips_maq_s_w_phr`.
  **L158 CN**: 声明 TableGen def `int_mips_maq_s_w_phr`。
- **L159 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L159 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L160 EN**: Declares TableGen def `int_mips_maq_sa_w_phl`.
  **L160 CN**: 声明 TableGen def `int_mips_maq_sa_w_phl`。
- **L161 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L161 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L162 EN**: Declares TableGen def `int_mips_maq_sa_w_phr`.
  **L162 CN**: 声明 TableGen def `int_mips_maq_sa_w_phr`。
- **L163 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L163 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L164 EN**: Declares TableGen def `int_mips_mult`.
  **L164 CN**: 声明 TableGen def `int_mips_mult`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L166 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L166 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L167 EN**: Declares TableGen def `int_mips_multu`.
  **L167 CN**: 声明 TableGen def `int_mips_multu`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty],`。

### Lines 169-196

````tablegen
            [IntrNoMem, Commutative]>;

//===----------------------------------------------------------------------===//
// Dot product with accumulate/subtract

def int_mips_dpau_h_qbl: ClangBuiltin<"__builtin_mips_dpau_h_qbl">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],
            [IntrNoMem]>;
def int_mips_dpau_h_qbr: ClangBuiltin<"__builtin_mips_dpau_h_qbr">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],
            [IntrNoMem]>;
def int_mips_dpsu_h_qbl: ClangBuiltin<"__builtin_mips_dpsu_h_qbl">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],
            [IntrNoMem]>;
def int_mips_dpsu_h_qbr: ClangBuiltin<"__builtin_mips_dpsu_h_qbr">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],
            [IntrNoMem]>;
def int_mips_dpaq_s_w_ph: ClangBuiltin<"__builtin_mips_dpaq_s_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_dpsq_s_w_ph: ClangBuiltin<"__builtin_mips_dpsq_s_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_dpaq_sa_l_w: ClangBuiltin<"__builtin_mips_dpaq_sa_l_w">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_q31_ty, mips_q31_ty], []>;
def int_mips_dpsq_sa_l_w: ClangBuiltin<"__builtin_mips_dpsq_sa_l_w">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_q31_ty, mips_q31_ty], []>;

//===----------------------------------------------------------------------===//
// Comparison
````
- **L169 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L169 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Banner comment marking a file or section boundary.
  **L171 CN**: 横幅注释，用于标记文件或章节边界。
- **L172 EN**: Comment explains nearby logic, invariants, or intent: `Dot product with accumulate/subtract`.
  **L172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Dot product with accumulate/subtract`。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L174 EN**: Declares TableGen def `int_mips_dpau_h_qbl`.
  **L174 CN**: 声明 TableGen def `int_mips_dpau_h_qbl`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`。
- **L176 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L176 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L177 EN**: Declares TableGen def `int_mips_dpau_h_qbr`.
  **L177 CN**: 声明 TableGen def `int_mips_dpau_h_qbr`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`。
- **L179 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L179 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L180 EN**: Declares TableGen def `int_mips_dpsu_h_qbl`.
  **L180 CN**: 声明 TableGen def `int_mips_dpsu_h_qbl`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`。
- **L182 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L182 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L183 EN**: Declares TableGen def `int_mips_dpsu_h_qbr`.
  **L183 CN**: 声明 TableGen def `int_mips_dpsu_h_qbr`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v4i8_ty, llvm_v4i8_ty],`。
- **L185 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L185 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L186 EN**: Declares TableGen def `int_mips_dpaq_s_w_ph`.
  **L186 CN**: 声明 TableGen def `int_mips_dpaq_s_w_ph`。
- **L187 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L187 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L188 EN**: Declares TableGen def `int_mips_dpsq_s_w_ph`.
  **L188 CN**: 声明 TableGen def `int_mips_dpsq_s_w_ph`。
- **L189 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L189 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L190 EN**: Declares TableGen def `int_mips_dpaq_sa_l_w`.
  **L190 CN**: 声明 TableGen def `int_mips_dpaq_sa_l_w`。
- **L191 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_q31_ty, mips_q31_ty], []>;`.
  **L191 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_q31_ty, mips_q31_ty], []>;`。
- **L192 EN**: Declares TableGen def `int_mips_dpsq_sa_l_w`.
  **L192 CN**: 声明 TableGen def `int_mips_dpsq_sa_l_w`。
- **L193 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_q31_ty, mips_q31_ty], []>;`.
  **L193 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_q31_ty, mips_q31_ty], []>;`。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L195 EN**: Banner comment marking a file or section boundary.
  **L195 CN**: 横幅注释，用于标记文件或章节边界。
- **L196 EN**: Comment explains nearby logic, invariants, or intent: `Comparison`.
  **L196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Comparison`。

### Lines 197-224

````tablegen

def int_mips_cmpu_eq_qb: ClangBuiltin<"__builtin_mips_cmpu_eq_qb">,
  Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;
def int_mips_cmpu_lt_qb: ClangBuiltin<"__builtin_mips_cmpu_lt_qb">,
  Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], []>;
def int_mips_cmpu_le_qb: ClangBuiltin<"__builtin_mips_cmpu_le_qb">,
  Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], []>;
def int_mips_cmpgu_eq_qb: ClangBuiltin<"__builtin_mips_cmpgu_eq_qb">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;
def int_mips_cmpgu_lt_qb: ClangBuiltin<"__builtin_mips_cmpgu_lt_qb">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;
def int_mips_cmpgu_le_qb: ClangBuiltin<"__builtin_mips_cmpgu_le_qb">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;
def int_mips_cmp_eq_ph: ClangBuiltin<"__builtin_mips_cmp_eq_ph">,
  Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;
def int_mips_cmp_lt_ph: ClangBuiltin<"__builtin_mips_cmp_lt_ph">,
  Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_cmp_le_ph: ClangBuiltin<"__builtin_mips_cmp_le_ph">,
  Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], []>;

//===----------------------------------------------------------------------===//
// Extracting

def int_mips_extr_s_h: ClangBuiltin<"__builtin_mips_extr_s_h">,
  Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;
def int_mips_extr_w: ClangBuiltin<"__builtin_mips_extr_w">,
  Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;
def int_mips_extr_rs_w: ClangBuiltin<"__builtin_mips_extr_rs_w">,
````
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Declares TableGen def `int_mips_cmpu_eq_qb`.
  **L198 CN**: 声明 TableGen def `int_mips_cmpu_eq_qb`。
- **L199 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;`.
  **L199 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;`。
- **L200 EN**: Declares TableGen def `int_mips_cmpu_lt_qb`.
  **L200 CN**: 声明 TableGen def `int_mips_cmpu_lt_qb`。
- **L201 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`.
  **L201 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`。
- **L202 EN**: Declares TableGen def `int_mips_cmpu_le_qb`.
  **L202 CN**: 声明 TableGen def `int_mips_cmpu_le_qb`。
- **L203 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`.
  **L203 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`。
- **L204 EN**: Declares TableGen def `int_mips_cmpgu_eq_qb`.
  **L204 CN**: 声明 TableGen def `int_mips_cmpgu_eq_qb`。
- **L205 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;`.
  **L205 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;`。
- **L206 EN**: Declares TableGen def `int_mips_cmpgu_lt_qb`.
  **L206 CN**: 声明 TableGen def `int_mips_cmpgu_lt_qb`。
- **L207 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`.
  **L207 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`。
- **L208 EN**: Declares TableGen def `int_mips_cmpgu_le_qb`.
  **L208 CN**: 声明 TableGen def `int_mips_cmpgu_le_qb`。
- **L209 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`.
  **L209 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`。
- **L210 EN**: Declares TableGen def `int_mips_cmp_eq_ph`.
  **L210 CN**: 声明 TableGen def `int_mips_cmp_eq_ph`。
- **L211 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`.
  **L211 CN**: 执行一条独立语句或声明：`Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`。
- **L212 EN**: Declares TableGen def `int_mips_cmp_lt_ph`.
  **L212 CN**: 声明 TableGen def `int_mips_cmp_lt_ph`。
- **L213 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L213 CN**: 执行一条独立语句或声明：`Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L214 EN**: Declares TableGen def `int_mips_cmp_le_ph`.
  **L214 CN**: 声明 TableGen def `int_mips_cmp_le_ph`。
- **L215 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L215 CN**: 执行一条独立语句或声明：`Intrinsic<[], [mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L217 EN**: Banner comment marking a file or section boundary.
  **L217 CN**: 横幅注释，用于标记文件或章节边界。
- **L218 EN**: Comment explains nearby logic, invariants, or intent: `Extracting`.
  **L218 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Extracting`。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L220 EN**: Declares TableGen def `int_mips_extr_s_h`.
  **L220 CN**: 声明 TableGen def `int_mips_extr_s_h`。
- **L221 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`.
  **L221 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`。
- **L222 EN**: Declares TableGen def `int_mips_extr_w`.
  **L222 CN**: 声明 TableGen def `int_mips_extr_w`。
- **L223 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`.
  **L223 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`。
- **L224 EN**: Declares TableGen def `int_mips_extr_rs_w`.
  **L224 CN**: 声明 TableGen def `int_mips_extr_rs_w`。

### Lines 225-252

````tablegen
  Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;
def int_mips_extr_r_w: ClangBuiltin<"__builtin_mips_extr_r_w">,
  Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;
def int_mips_extp: ClangBuiltin<"__builtin_mips_extp">,
  Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;
def int_mips_extpdp: ClangBuiltin<"__builtin_mips_extpdp">,
  Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;

//===----------------------------------------------------------------------===//
// Misc

def int_mips_wrdsp: ClangBuiltin<"__builtin_mips_wrdsp">,
  Intrinsic<[], [llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<1>>]>;
def int_mips_rddsp: ClangBuiltin<"__builtin_mips_rddsp">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrReadMem, ImmArg<ArgIndex<0>>]>;

def int_mips_insv: ClangBuiltin<"__builtin_mips_insv">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrReadMem]>;
def int_mips_bitrev: ClangBuiltin<"__builtin_mips_bitrev">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;

def int_mips_packrl_ph: ClangBuiltin<"__builtin_mips_packrl_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;

def int_mips_repl_qb: ClangBuiltin<"__builtin_mips_repl_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_i32_ty], [IntrNoMem]>;
def int_mips_repl_ph: ClangBuiltin<"__builtin_mips_repl_ph">,
  Intrinsic<[mips_v2q15_ty], [llvm_i32_ty], [IntrNoMem]>;
````
- **L225 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`.
  **L225 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`。
- **L226 EN**: Declares TableGen def `int_mips_extr_r_w`.
  **L226 CN**: 声明 TableGen def `int_mips_extr_r_w`。
- **L227 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`.
  **L227 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`。
- **L228 EN**: Declares TableGen def `int_mips_extp`.
  **L228 CN**: 声明 TableGen def `int_mips_extp`。
- **L229 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`.
  **L229 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`。
- **L230 EN**: Declares TableGen def `int_mips_extpdp`.
  **L230 CN**: 声明 TableGen def `int_mips_extpdp`。
- **L231 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`.
  **L231 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_i32_ty], []>;`。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L233 EN**: Banner comment marking a file or section boundary.
  **L233 CN**: 横幅注释，用于标记文件或章节边界。
- **L234 EN**: Comment explains nearby logic, invariants, or intent: `Misc`.
  **L234 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Misc`。
- **L235 EN**: Blank line separating nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L236 EN**: Declares TableGen def `int_mips_wrdsp`.
  **L236 CN**: 声明 TableGen def `int_mips_wrdsp`。
- **L237 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<1>>]>;`.
  **L237 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<1>>]>;`。
- **L238 EN**: Declares TableGen def `int_mips_rddsp`.
  **L238 CN**: 声明 TableGen def `int_mips_rddsp`。
- **L239 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrReadMem, ImmArg<ArgIndex<0>>]>;`.
  **L239 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrReadMem, ImmArg<ArgIndex<0>>]>;`。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L241 EN**: Declares TableGen def `int_mips_insv`.
  **L241 CN**: 声明 TableGen def `int_mips_insv`。
- **L242 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrReadMem]>;`.
  **L242 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrReadMem]>;`。
- **L243 EN**: Declares TableGen def `int_mips_bitrev`.
  **L243 CN**: 声明 TableGen def `int_mips_bitrev`。
- **L244 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L244 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Declares TableGen def `int_mips_packrl_ph`.
  **L246 CN**: 声明 TableGen def `int_mips_packrl_ph`。
- **L247 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`.
  **L247 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L249 EN**: Declares TableGen def `int_mips_repl_qb`.
  **L249 CN**: 声明 TableGen def `int_mips_repl_qb`。
- **L250 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L250 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L251 EN**: Declares TableGen def `int_mips_repl_ph`.
  **L251 CN**: 声明 TableGen def `int_mips_repl_ph`。
- **L252 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L252 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [llvm_i32_ty], [IntrNoMem]>;`。

### Lines 253-280

````tablegen

def int_mips_pick_qb: ClangBuiltin<"__builtin_mips_pick_qb">,
  DefaultAttrsIntrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrReadMem]>;
def int_mips_pick_ph: ClangBuiltin<"__builtin_mips_pick_ph">,
  DefaultAttrsIntrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrReadMem]>;

def int_mips_mthlip: ClangBuiltin<"__builtin_mips_mthlip">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty], []>;

def int_mips_bposge32: ClangBuiltin<"__builtin_mips_bposge32">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrReadMem]>;

def int_mips_lbux: ClangBuiltin<"__builtin_mips_lbux">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;
def int_mips_lhx: ClangBuiltin<"__builtin_mips_lhx">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;
def int_mips_lwx: ClangBuiltin<"__builtin_mips_lwx">,
  DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;

//===----------------------------------------------------------------------===//
// MIPS DSP Rev 2

def int_mips_absq_s_qb: ClangBuiltin<"__builtin_mips_absq_s_qb">,
  Intrinsic<[mips_v4q7_ty], [mips_v4q7_ty], []>;

def int_mips_addqh_ph: ClangBuiltin<"__builtin_mips_addqh_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],
            [IntrNoMem, Commutative]>;
````
- **L253 EN**: Blank line separating nearby declarations or logic blocks.
  **L253 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L254 EN**: Declares TableGen def `int_mips_pick_qb`.
  **L254 CN**: 声明 TableGen def `int_mips_pick_qb`。
- **L255 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrReadMem]>;`.
  **L255 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrReadMem]>;`。
- **L256 EN**: Declares TableGen def `int_mips_pick_ph`.
  **L256 CN**: 声明 TableGen def `int_mips_pick_ph`。
- **L257 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrReadMem]>;`.
  **L257 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrReadMem]>;`。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L259 EN**: Declares TableGen def `int_mips_mthlip`.
  **L259 CN**: 声明 TableGen def `int_mips_mthlip`。
- **L260 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty], []>;`.
  **L260 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i32_ty], []>;`。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L262 EN**: Declares TableGen def `int_mips_bposge32`.
  **L262 CN**: 声明 TableGen def `int_mips_bposge32`。
- **L263 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrReadMem]>;`.
  **L263 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrReadMem]>;`。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L265 EN**: Declares TableGen def `int_mips_lbux`.
  **L265 CN**: 声明 TableGen def `int_mips_lbux`。
- **L266 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L266 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L267 EN**: Declares TableGen def `int_mips_lhx`.
  **L267 CN**: 声明 TableGen def `int_mips_lhx`。
- **L268 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L268 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L269 EN**: Declares TableGen def `int_mips_lwx`.
  **L269 CN**: 声明 TableGen def `int_mips_lwx`。
- **L270 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L270 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L271 EN**: Blank line separating nearby declarations or logic blocks.
  **L271 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L272 EN**: Banner comment marking a file or section boundary.
  **L272 CN**: 横幅注释，用于标记文件或章节边界。
- **L273 EN**: Comment explains nearby logic, invariants, or intent: `MIPS DSP Rev 2`.
  **L273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MIPS DSP Rev 2`。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L275 EN**: Declares TableGen def `int_mips_absq_s_qb`.
  **L275 CN**: 声明 TableGen def `int_mips_absq_s_qb`。
- **L276 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v4q7_ty], [mips_v4q7_ty], []>;`.
  **L276 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v4q7_ty], [mips_v4q7_ty], []>;`。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Declares TableGen def `int_mips_addqh_ph`.
  **L278 CN**: 声明 TableGen def `int_mips_addqh_ph`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`。
- **L280 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L280 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。

### Lines 281-308

````tablegen
def int_mips_addqh_r_ph: ClangBuiltin<"__builtin_mips_addqh_r_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],
            [IntrNoMem, Commutative]>;
def int_mips_addqh_w: ClangBuiltin<"__builtin_mips_addqh_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty],
            [IntrNoMem, Commutative]>;
def int_mips_addqh_r_w: ClangBuiltin<"__builtin_mips_addqh_r_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty],
            [IntrNoMem, Commutative]>;

def int_mips_addu_ph: ClangBuiltin<"__builtin_mips_addu_ph">,
  Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;
def int_mips_addu_s_ph: ClangBuiltin<"__builtin_mips_addu_s_ph">,
  Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;

def int_mips_adduh_qb: ClangBuiltin<"__builtin_mips_adduh_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],
            [IntrNoMem, Commutative]>;
def int_mips_adduh_r_qb: ClangBuiltin<"__builtin_mips_adduh_r_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],
            [IntrNoMem, Commutative]>;

def int_mips_append: ClangBuiltin<"__builtin_mips_append">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
  [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_balign: ClangBuiltin<"__builtin_mips_balign">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
  [IntrNoMem, ImmArg<ArgIndex<2>>]>;
````
- **L281 EN**: Declares TableGen def `int_mips_addqh_r_ph`.
  **L281 CN**: 声明 TableGen def `int_mips_addqh_r_ph`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty],`。
- **L283 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L283 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L284 EN**: Declares TableGen def `int_mips_addqh_w`.
  **L284 CN**: 声明 TableGen def `int_mips_addqh_w`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty],`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty],`。
- **L286 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L286 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L287 EN**: Declares TableGen def `int_mips_addqh_r_w`.
  **L287 CN**: 声明 TableGen def `int_mips_addqh_r_w`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty],`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty],`。
- **L289 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L289 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L291 EN**: Declares TableGen def `int_mips_addu_ph`.
  **L291 CN**: 声明 TableGen def `int_mips_addu_ph`。
- **L292 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`.
  **L292 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`。
- **L293 EN**: Declares TableGen def `int_mips_addu_s_ph`.
  **L293 CN**: 声明 TableGen def `int_mips_addu_s_ph`。
- **L294 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`.
  **L294 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L296 EN**: Declares TableGen def `int_mips_adduh_qb`.
  **L296 CN**: 声明 TableGen def `int_mips_adduh_qb`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`。
- **L298 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L298 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L299 EN**: Declares TableGen def `int_mips_adduh_r_qb`.
  **L299 CN**: 声明 TableGen def `int_mips_adduh_r_qb`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty],`。
- **L301 EN**: Executes a standalone statement or declaration: `[IntrNoMem, Commutative]>;`.
  **L301 CN**: 执行一条独立语句或声明：`[IntrNoMem, Commutative]>;`。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L303 EN**: Declares TableGen def `int_mips_append`.
  **L303 CN**: 声明 TableGen def `int_mips_append`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L305 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L305 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L306 EN**: Declares TableGen def `int_mips_balign`.
  **L306 CN**: 声明 TableGen def `int_mips_balign`。
- **L307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L307 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L308 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L308 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。

### Lines 309-336

````tablegen

def int_mips_cmpgdu_eq_qb: ClangBuiltin<"__builtin_mips_cmpgdu_eq_qb">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;
def int_mips_cmpgdu_lt_qb: ClangBuiltin<"__builtin_mips_cmpgdu_lt_qb">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;
def int_mips_cmpgdu_le_qb: ClangBuiltin<"__builtin_mips_cmpgdu_le_qb">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;

def int_mips_dpa_w_ph: ClangBuiltin<"__builtin_mips_dpa_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],
            [IntrNoMem]>;
def int_mips_dps_w_ph: ClangBuiltin<"__builtin_mips_dps_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],
            [IntrNoMem]>;

def int_mips_dpaqx_s_w_ph: ClangBuiltin<"__builtin_mips_dpaqx_s_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_dpaqx_sa_w_ph: ClangBuiltin<"__builtin_mips_dpaqx_sa_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_dpax_w_ph: ClangBuiltin<"__builtin_mips_dpax_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],
            [IntrNoMem]>;
def int_mips_dpsx_w_ph: ClangBuiltin<"__builtin_mips_dpsx_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],
            [IntrNoMem]>;
def int_mips_dpsqx_s_w_ph: ClangBuiltin<"__builtin_mips_dpsqx_s_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;
def int_mips_dpsqx_sa_w_ph: ClangBuiltin<"__builtin_mips_dpsqx_sa_w_ph">,
````
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Declares TableGen def `int_mips_cmpgdu_eq_qb`.
  **L310 CN**: 声明 TableGen def `int_mips_cmpgdu_eq_qb`。
- **L311 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;`.
  **L311 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [Commutative]>;`。
- **L312 EN**: Declares TableGen def `int_mips_cmpgdu_lt_qb`.
  **L312 CN**: 声明 TableGen def `int_mips_cmpgdu_lt_qb`。
- **L313 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`.
  **L313 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`。
- **L314 EN**: Declares TableGen def `int_mips_cmpgdu_le_qb`.
  **L314 CN**: 声明 TableGen def `int_mips_cmpgdu_le_qb`。
- **L315 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`.
  **L315 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i8_ty, llvm_v4i8_ty], []>;`。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L317 EN**: Declares TableGen def `int_mips_dpa_w_ph`.
  **L317 CN**: 声明 TableGen def `int_mips_dpa_w_ph`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`。
- **L319 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L319 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L320 EN**: Declares TableGen def `int_mips_dps_w_ph`.
  **L320 CN**: 声明 TableGen def `int_mips_dps_w_ph`。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`。
- **L322 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L322 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L324 EN**: Declares TableGen def `int_mips_dpaqx_s_w_ph`.
  **L324 CN**: 声明 TableGen def `int_mips_dpaqx_s_w_ph`。
- **L325 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L325 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L326 EN**: Declares TableGen def `int_mips_dpaqx_sa_w_ph`.
  **L326 CN**: 声明 TableGen def `int_mips_dpaqx_sa_w_ph`。
- **L327 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L327 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L328 EN**: Declares TableGen def `int_mips_dpax_w_ph`.
  **L328 CN**: 声明 TableGen def `int_mips_dpax_w_ph`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`。
- **L330 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L330 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L331 EN**: Declares TableGen def `int_mips_dpsx_w_ph`.
  **L331 CN**: 声明 TableGen def `int_mips_dpsx_w_ph`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`。
- **L333 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L333 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L334 EN**: Declares TableGen def `int_mips_dpsqx_s_w_ph`.
  **L334 CN**: 声明 TableGen def `int_mips_dpsqx_s_w_ph`。
- **L335 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L335 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L336 EN**: Declares TableGen def `int_mips_dpsqx_sa_w_ph`.
  **L336 CN**: 声明 TableGen def `int_mips_dpsqx_sa_w_ph`。

### Lines 337-364

````tablegen
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;

def int_mips_mul_ph: ClangBuiltin<"__builtin_mips_mul_ph">,
  Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;
def int_mips_mul_s_ph: ClangBuiltin<"__builtin_mips_mul_s_ph">,
  Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;

def int_mips_mulq_rs_w: ClangBuiltin<"__builtin_mips_mulq_rs_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;
def int_mips_mulq_s_ph: ClangBuiltin<"__builtin_mips_mulq_s_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;
def int_mips_mulq_s_w: ClangBuiltin<"__builtin_mips_mulq_s_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;
def int_mips_mulsa_w_ph: ClangBuiltin<"__builtin_mips_mulsa_w_ph">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],
            [IntrNoMem]>;

def int_mips_precr_qb_ph: ClangBuiltin<"__builtin_mips_precr_qb_ph">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;
def int_mips_precr_sra_ph_w: ClangBuiltin<"__builtin_mips_precr_sra_ph_w">,
  Intrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_precr_sra_r_ph_w: ClangBuiltin<"__builtin_mips_precr_sra_r_ph_w">,
  Intrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_prepend: ClangBuiltin<"__builtin_mips_prepend">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
````
- **L337 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`.
  **L337 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, mips_v2q15_ty, mips_v2q15_ty], []>;`。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L339 EN**: Declares TableGen def `int_mips_mul_ph`.
  **L339 CN**: 声明 TableGen def `int_mips_mul_ph`。
- **L340 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`.
  **L340 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`。
- **L341 EN**: Declares TableGen def `int_mips_mul_s_ph`.
  **L341 CN**: 声明 TableGen def `int_mips_mul_s_ph`。
- **L342 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`.
  **L342 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], [Commutative]>;`。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L344 EN**: Declares TableGen def `int_mips_mulq_rs_w`.
  **L344 CN**: 声明 TableGen def `int_mips_mulq_rs_w`。
- **L345 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;`.
  **L345 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;`。
- **L346 EN**: Declares TableGen def `int_mips_mulq_s_ph`.
  **L346 CN**: 声明 TableGen def `int_mips_mulq_s_ph`。
- **L347 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`.
  **L347 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [Commutative]>;`。
- **L348 EN**: Declares TableGen def `int_mips_mulq_s_w`.
  **L348 CN**: 声明 TableGen def `int_mips_mulq_s_w`。
- **L349 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;`.
  **L349 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [Commutative]>;`。
- **L350 EN**: Declares TableGen def `int_mips_mulsa_w_ph`.
  **L350 CN**: 声明 TableGen def `int_mips_mulsa_w_ph`。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`.
  **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_v2i16_ty, llvm_v2i16_ty],`。
- **L352 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L352 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L354 EN**: Declares TableGen def `int_mips_precr_qb_ph`.
  **L354 CN**: 声明 TableGen def `int_mips_precr_qb_ph`。
- **L355 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;`.
  **L355 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;`。
- **L356 EN**: Declares TableGen def `int_mips_precr_sra_ph_w`.
  **L356 CN**: 声明 TableGen def `int_mips_precr_sra_ph_w`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L358 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L358 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L359 EN**: Declares TableGen def `int_mips_precr_sra_r_ph_w`.
  **L359 CN**: 声明 TableGen def `int_mips_precr_sra_r_ph_w`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i16_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L361 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L361 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L363 EN**: Declares TableGen def `int_mips_prepend`.
  **L363 CN**: 声明 TableGen def `int_mips_prepend`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。

### Lines 365-392

````tablegen
  [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_shra_qb: ClangBuiltin<"__builtin_mips_shra_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_shra_r_qb: ClangBuiltin<"__builtin_mips_shra_r_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_shrl_ph: ClangBuiltin<"__builtin_mips_shrl_ph">,
  Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_i32_ty], [IntrNoMem]>;

def int_mips_subqh_ph: ClangBuiltin<"__builtin_mips_subqh_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;
def int_mips_subqh_r_ph: ClangBuiltin<"__builtin_mips_subqh_r_ph">,
  Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;
def int_mips_subqh_w: ClangBuiltin<"__builtin_mips_subqh_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;
def int_mips_subqh_r_w: ClangBuiltin<"__builtin_mips_subqh_r_w">,
  Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;

def int_mips_subu_ph: ClangBuiltin<"__builtin_mips_subu_ph">,
  Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;
def int_mips_subu_s_ph: ClangBuiltin<"__builtin_mips_subu_s_ph">,
  Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;

def int_mips_subuh_qb: ClangBuiltin<"__builtin_mips_subuh_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;
def int_mips_subuh_r_qb: ClangBuiltin<"__builtin_mips_subuh_r_qb">,
  Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;

````
- **L365 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L365 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L367 EN**: Declares TableGen def `int_mips_shra_qb`.
  **L367 CN**: 声明 TableGen def `int_mips_shra_qb`。
- **L368 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L368 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L369 EN**: Declares TableGen def `int_mips_shra_r_qb`.
  **L369 CN**: 声明 TableGen def `int_mips_shra_r_qb`。
- **L370 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L370 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L371 EN**: Declares TableGen def `int_mips_shrl_ph`.
  **L371 CN**: 声明 TableGen def `int_mips_shrl_ph`。
- **L372 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L372 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L374 EN**: Declares TableGen def `int_mips_subqh_ph`.
  **L374 CN**: 声明 TableGen def `int_mips_subqh_ph`。
- **L375 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`.
  **L375 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`。
- **L376 EN**: Declares TableGen def `int_mips_subqh_r_ph`.
  **L376 CN**: 声明 TableGen def `int_mips_subqh_r_ph`。
- **L377 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`.
  **L377 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_v2q15_ty], [mips_v2q15_ty, mips_v2q15_ty], [IntrNoMem]>;`。
- **L378 EN**: Declares TableGen def `int_mips_subqh_w`.
  **L378 CN**: 声明 TableGen def `int_mips_subqh_w`。
- **L379 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;`.
  **L379 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;`。
- **L380 EN**: Declares TableGen def `int_mips_subqh_r_w`.
  **L380 CN**: 声明 TableGen def `int_mips_subqh_r_w`。
- **L381 EN**: Executes a standalone statement or declaration: `Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;`.
  **L381 CN**: 执行一条独立语句或声明：`Intrinsic<[mips_q31_ty], [mips_q31_ty, mips_q31_ty], [IntrNoMem]>;`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Declares TableGen def `int_mips_subu_ph`.
  **L383 CN**: 声明 TableGen def `int_mips_subu_ph`。
- **L384 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;`.
  **L384 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;`。
- **L385 EN**: Declares TableGen def `int_mips_subu_s_ph`.
  **L385 CN**: 声明 TableGen def `int_mips_subu_s_ph`。
- **L386 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;`.
  **L386 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i16_ty], [llvm_v2i16_ty, llvm_v2i16_ty], []>;`。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L388 EN**: Declares TableGen def `int_mips_subuh_qb`.
  **L388 CN**: 声明 TableGen def `int_mips_subuh_qb`。
- **L389 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`.
  **L389 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`。
- **L390 EN**: Declares TableGen def `int_mips_subuh_r_qb`.
  **L390 CN**: 声明 TableGen def `int_mips_subuh_r_qb`。
- **L391 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`.
  **L391 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i8_ty], [llvm_v4i8_ty, llvm_v4i8_ty], [IntrNoMem]>;`。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 393-420

````tablegen
//===----------------------------------------------------------------------===//
// MIPS MSA

//===----------------------------------------------------------------------===//
// Addition/subtraction

def int_mips_add_a_b : ClangBuiltin<"__builtin_msa_add_a_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_add_a_h : ClangBuiltin<"__builtin_msa_add_a_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_add_a_w : ClangBuiltin<"__builtin_msa_add_a_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_add_a_d : ClangBuiltin<"__builtin_msa_add_a_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_adds_a_b : ClangBuiltin<"__builtin_msa_adds_a_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_a_h : ClangBuiltin<"__builtin_msa_adds_a_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_a_w : ClangBuiltin<"__builtin_msa_adds_a_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
````
- **L393 EN**: Banner comment marking a file or section boundary.
  **L393 CN**: 横幅注释，用于标记文件或章节边界。
- **L394 EN**: Comment explains nearby logic, invariants, or intent: `MIPS MSA`.
  **L394 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MIPS MSA`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L396 EN**: Banner comment marking a file or section boundary.
  **L396 CN**: 横幅注释，用于标记文件或章节边界。
- **L397 EN**: Comment explains nearby logic, invariants, or intent: `Addition/subtraction`.
  **L397 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Addition/subtraction`。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L399 EN**: Declares TableGen def `int_mips_add_a_b`.
  **L399 CN**: 声明 TableGen def `int_mips_add_a_b`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L401 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L401 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L402 EN**: Declares TableGen def `int_mips_add_a_h`.
  **L402 CN**: 声明 TableGen def `int_mips_add_a_h`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L404 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L404 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L405 EN**: Declares TableGen def `int_mips_add_a_w`.
  **L405 CN**: 声明 TableGen def `int_mips_add_a_w`。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L407 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L407 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L408 EN**: Declares TableGen def `int_mips_add_a_d`.
  **L408 CN**: 声明 TableGen def `int_mips_add_a_d`。
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L410 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L410 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L411 EN**: Blank line separating nearby declarations or logic blocks.
  **L411 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L412 EN**: Declares TableGen def `int_mips_adds_a_b`.
  **L412 CN**: 声明 TableGen def `int_mips_adds_a_b`。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L414 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L414 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L415 EN**: Declares TableGen def `int_mips_adds_a_h`.
  **L415 CN**: 声明 TableGen def `int_mips_adds_a_h`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L417 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L417 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L418 EN**: Declares TableGen def `int_mips_adds_a_w`.
  **L418 CN**: 声明 TableGen def `int_mips_adds_a_w`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L420 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L420 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。

### Lines 421-448

````tablegen
def int_mips_adds_a_d : ClangBuiltin<"__builtin_msa_adds_a_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_adds_s_b : ClangBuiltin<"__builtin_msa_adds_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_s_h : ClangBuiltin<"__builtin_msa_adds_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_s_w : ClangBuiltin<"__builtin_msa_adds_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_s_d : ClangBuiltin<"__builtin_msa_adds_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_adds_u_b : ClangBuiltin<"__builtin_msa_adds_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_u_h : ClangBuiltin<"__builtin_msa_adds_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_u_w : ClangBuiltin<"__builtin_msa_adds_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_adds_u_d : ClangBuiltin<"__builtin_msa_adds_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
````
- **L421 EN**: Declares TableGen def `int_mips_adds_a_d`.
  **L421 CN**: 声明 TableGen def `int_mips_adds_a_d`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L423 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L423 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L425 EN**: Declares TableGen def `int_mips_adds_s_b`.
  **L425 CN**: 声明 TableGen def `int_mips_adds_s_b`。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L427 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L427 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L428 EN**: Declares TableGen def `int_mips_adds_s_h`.
  **L428 CN**: 声明 TableGen def `int_mips_adds_s_h`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L430 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L430 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L431 EN**: Declares TableGen def `int_mips_adds_s_w`.
  **L431 CN**: 声明 TableGen def `int_mips_adds_s_w`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L433 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L433 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L434 EN**: Declares TableGen def `int_mips_adds_s_d`.
  **L434 CN**: 声明 TableGen def `int_mips_adds_s_d`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L436 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L436 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L438 EN**: Declares TableGen def `int_mips_adds_u_b`.
  **L438 CN**: 声明 TableGen def `int_mips_adds_u_b`。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L440 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L440 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L441 EN**: Declares TableGen def `int_mips_adds_u_h`.
  **L441 CN**: 声明 TableGen def `int_mips_adds_u_h`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L443 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L443 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L444 EN**: Declares TableGen def `int_mips_adds_u_w`.
  **L444 CN**: 声明 TableGen def `int_mips_adds_u_w`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L446 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L446 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L447 EN**: Declares TableGen def `int_mips_adds_u_d`.
  **L447 CN**: 声明 TableGen def `int_mips_adds_u_d`。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。

### Lines 449-476

````tablegen
  [Commutative, IntrNoMem]>;

def int_mips_addv_b : ClangBuiltin<"__builtin_msa_addv_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_addv_h : ClangBuiltin<"__builtin_msa_addv_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_addv_w : ClangBuiltin<"__builtin_msa_addv_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_addv_d : ClangBuiltin<"__builtin_msa_addv_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_addvi_b : ClangBuiltin<"__builtin_msa_addvi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],
  [Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_addvi_h : ClangBuiltin<"__builtin_msa_addvi_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty],
  [Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_addvi_w : ClangBuiltin<"__builtin_msa_addvi_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty],
  [Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_addvi_d : ClangBuiltin<"__builtin_msa_addvi_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty],
  [Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;

````
- **L449 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L449 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L451 EN**: Declares TableGen def `int_mips_addv_b`.
  **L451 CN**: 声明 TableGen def `int_mips_addv_b`。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L453 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L453 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L454 EN**: Declares TableGen def `int_mips_addv_h`.
  **L454 CN**: 声明 TableGen def `int_mips_addv_h`。
- **L455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L455 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L456 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L456 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L457 EN**: Declares TableGen def `int_mips_addv_w`.
  **L457 CN**: 声明 TableGen def `int_mips_addv_w`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L459 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L459 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L460 EN**: Declares TableGen def `int_mips_addv_d`.
  **L460 CN**: 声明 TableGen def `int_mips_addv_d`。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L462 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L462 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L464 EN**: Declares TableGen def `int_mips_addvi_b`.
  **L464 CN**: 声明 TableGen def `int_mips_addvi_b`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`。
- **L466 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L466 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L467 EN**: Declares TableGen def `int_mips_addvi_h`.
  **L467 CN**: 声明 TableGen def `int_mips_addvi_h`。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty],`.
  **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty],`。
- **L469 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L469 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L470 EN**: Declares TableGen def `int_mips_addvi_w`.
  **L470 CN**: 声明 TableGen def `int_mips_addvi_w`。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty],`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty],`。
- **L472 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L472 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L473 EN**: Declares TableGen def `int_mips_addvi_d`.
  **L473 CN**: 声明 TableGen def `int_mips_addvi_d`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty],`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty],`。
- **L475 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L475 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 477-504

````tablegen
def int_mips_and_v : ClangBuiltin<"__builtin_msa_and_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;

def int_mips_andi_b : ClangBuiltin<"__builtin_msa_andi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_asub_s_b : ClangBuiltin<"__builtin_msa_asub_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_asub_s_h : ClangBuiltin<"__builtin_msa_asub_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_asub_s_w : ClangBuiltin<"__builtin_msa_asub_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_asub_s_d : ClangBuiltin<"__builtin_msa_asub_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_asub_u_b : ClangBuiltin<"__builtin_msa_asub_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_asub_u_h : ClangBuiltin<"__builtin_msa_asub_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_asub_u_w : ClangBuiltin<"__builtin_msa_asub_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_asub_u_d : ClangBuiltin<"__builtin_msa_asub_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_ave_s_b : ClangBuiltin<"__builtin_msa_ave_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_ave_s_h : ClangBuiltin<"__builtin_msa_ave_s_h">,
````
- **L477 EN**: Declares TableGen def `int_mips_and_v`.
  **L477 CN**: 声明 TableGen def `int_mips_and_v`。
- **L478 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L478 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L480 EN**: Declares TableGen def `int_mips_andi_b`.
  **L480 CN**: 声明 TableGen def `int_mips_andi_b`。
- **L481 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L481 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L483 EN**: Declares TableGen def `int_mips_asub_s_b`.
  **L483 CN**: 声明 TableGen def `int_mips_asub_s_b`。
- **L484 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L484 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L485 EN**: Declares TableGen def `int_mips_asub_s_h`.
  **L485 CN**: 声明 TableGen def `int_mips_asub_s_h`。
- **L486 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L486 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L487 EN**: Declares TableGen def `int_mips_asub_s_w`.
  **L487 CN**: 声明 TableGen def `int_mips_asub_s_w`。
- **L488 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L488 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L489 EN**: Declares TableGen def `int_mips_asub_s_d`.
  **L489 CN**: 声明 TableGen def `int_mips_asub_s_d`。
- **L490 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L490 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L492 EN**: Declares TableGen def `int_mips_asub_u_b`.
  **L492 CN**: 声明 TableGen def `int_mips_asub_u_b`。
- **L493 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L493 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L494 EN**: Declares TableGen def `int_mips_asub_u_h`.
  **L494 CN**: 声明 TableGen def `int_mips_asub_u_h`。
- **L495 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L495 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L496 EN**: Declares TableGen def `int_mips_asub_u_w`.
  **L496 CN**: 声明 TableGen def `int_mips_asub_u_w`。
- **L497 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L497 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L498 EN**: Declares TableGen def `int_mips_asub_u_d`.
  **L498 CN**: 声明 TableGen def `int_mips_asub_u_d`。
- **L499 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L499 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L501 EN**: Declares TableGen def `int_mips_ave_s_b`.
  **L501 CN**: 声明 TableGen def `int_mips_ave_s_b`。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L503 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L503 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L504 EN**: Declares TableGen def `int_mips_ave_s_h`.
  **L504 CN**: 声明 TableGen def `int_mips_ave_s_h`。

### Lines 505-532

````tablegen
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_ave_s_w : ClangBuiltin<"__builtin_msa_ave_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_ave_s_d : ClangBuiltin<"__builtin_msa_ave_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_ave_u_b : ClangBuiltin<"__builtin_msa_ave_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_ave_u_h : ClangBuiltin<"__builtin_msa_ave_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_ave_u_w : ClangBuiltin<"__builtin_msa_ave_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_ave_u_d : ClangBuiltin<"__builtin_msa_ave_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_aver_s_b : ClangBuiltin<"__builtin_msa_aver_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_aver_s_h : ClangBuiltin<"__builtin_msa_aver_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
````
- **L505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L505 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L506 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L506 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L507 EN**: Declares TableGen def `int_mips_ave_s_w`.
  **L507 CN**: 声明 TableGen def `int_mips_ave_s_w`。
- **L508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L508 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L509 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L509 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L510 EN**: Declares TableGen def `int_mips_ave_s_d`.
  **L510 CN**: 声明 TableGen def `int_mips_ave_s_d`。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L512 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L512 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L513 EN**: Blank line separating nearby declarations or logic blocks.
  **L513 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L514 EN**: Declares TableGen def `int_mips_ave_u_b`.
  **L514 CN**: 声明 TableGen def `int_mips_ave_u_b`。
- **L515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L515 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L516 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L516 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L517 EN**: Declares TableGen def `int_mips_ave_u_h`.
  **L517 CN**: 声明 TableGen def `int_mips_ave_u_h`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L519 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L519 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L520 EN**: Declares TableGen def `int_mips_ave_u_w`.
  **L520 CN**: 声明 TableGen def `int_mips_ave_u_w`。
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L522 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L522 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L523 EN**: Declares TableGen def `int_mips_ave_u_d`.
  **L523 CN**: 声明 TableGen def `int_mips_ave_u_d`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L525 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L525 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L527 EN**: Declares TableGen def `int_mips_aver_s_b`.
  **L527 CN**: 声明 TableGen def `int_mips_aver_s_b`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L529 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L529 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L530 EN**: Declares TableGen def `int_mips_aver_s_h`.
  **L530 CN**: 声明 TableGen def `int_mips_aver_s_h`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L532 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L532 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。

### Lines 533-560

````tablegen
def int_mips_aver_s_w : ClangBuiltin<"__builtin_msa_aver_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_aver_s_d : ClangBuiltin<"__builtin_msa_aver_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_aver_u_b : ClangBuiltin<"__builtin_msa_aver_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
  [Commutative, IntrNoMem]>;
def int_mips_aver_u_h : ClangBuiltin<"__builtin_msa_aver_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
  [Commutative, IntrNoMem]>;
def int_mips_aver_u_w : ClangBuiltin<"__builtin_msa_aver_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
  [Commutative, IntrNoMem]>;
def int_mips_aver_u_d : ClangBuiltin<"__builtin_msa_aver_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
  [Commutative, IntrNoMem]>;

def int_mips_bclr_b : ClangBuiltin<"__builtin_msa_bclr_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_bclr_h : ClangBuiltin<"__builtin_msa_bclr_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_bclr_w : ClangBuiltin<"__builtin_msa_bclr_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_bclr_d : ClangBuiltin<"__builtin_msa_bclr_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;
````
- **L533 EN**: Declares TableGen def `int_mips_aver_s_w`.
  **L533 CN**: 声明 TableGen def `int_mips_aver_s_w`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L535 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L535 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L536 EN**: Declares TableGen def `int_mips_aver_s_d`.
  **L536 CN**: 声明 TableGen def `int_mips_aver_s_d`。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L538 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L538 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L540 EN**: Declares TableGen def `int_mips_aver_u_b`.
  **L540 CN**: 声明 TableGen def `int_mips_aver_u_b`。
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L542 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L542 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L543 EN**: Declares TableGen def `int_mips_aver_u_h`.
  **L543 CN**: 声明 TableGen def `int_mips_aver_u_h`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L545 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L545 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L546 EN**: Declares TableGen def `int_mips_aver_u_w`.
  **L546 CN**: 声明 TableGen def `int_mips_aver_u_w`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L548 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L548 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L549 EN**: Declares TableGen def `int_mips_aver_u_d`.
  **L549 CN**: 声明 TableGen def `int_mips_aver_u_d`。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L551 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L551 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L553 EN**: Declares TableGen def `int_mips_bclr_b`.
  **L553 CN**: 声明 TableGen def `int_mips_bclr_b`。
- **L554 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L554 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L555 EN**: Declares TableGen def `int_mips_bclr_h`.
  **L555 CN**: 声明 TableGen def `int_mips_bclr_h`。
- **L556 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L556 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L557 EN**: Declares TableGen def `int_mips_bclr_w`.
  **L557 CN**: 声明 TableGen def `int_mips_bclr_w`。
- **L558 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L558 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L559 EN**: Declares TableGen def `int_mips_bclr_d`.
  **L559 CN**: 声明 TableGen def `int_mips_bclr_d`。
- **L560 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L560 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。

### Lines 561-588

````tablegen

def int_mips_bclri_b : ClangBuiltin<"__builtin_msa_bclri_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bclri_h : ClangBuiltin<"__builtin_msa_bclri_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bclri_w : ClangBuiltin<"__builtin_msa_bclri_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bclri_d : ClangBuiltin<"__builtin_msa_bclri_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_binsl_b : ClangBuiltin<"__builtin_msa_binsl_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
            [IntrNoMem]>;
def int_mips_binsl_h : ClangBuiltin<"__builtin_msa_binsl_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
            [IntrNoMem]>;
def int_mips_binsl_w : ClangBuiltin<"__builtin_msa_binsl_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
            [IntrNoMem]>;
def int_mips_binsl_d : ClangBuiltin<"__builtin_msa_binsl_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
            [IntrNoMem]>;

def int_mips_binsli_b : ClangBuiltin<"__builtin_msa_binsli_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_binsli_h : ClangBuiltin<"__builtin_msa_binsli_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],
````
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L562 EN**: Declares TableGen def `int_mips_bclri_b`.
  **L562 CN**: 声明 TableGen def `int_mips_bclri_b`。
- **L563 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L563 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L564 EN**: Declares TableGen def `int_mips_bclri_h`.
  **L564 CN**: 声明 TableGen def `int_mips_bclri_h`。
- **L565 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L565 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L566 EN**: Declares TableGen def `int_mips_bclri_w`.
  **L566 CN**: 声明 TableGen def `int_mips_bclri_w`。
- **L567 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L567 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L568 EN**: Declares TableGen def `int_mips_bclri_d`.
  **L568 CN**: 声明 TableGen def `int_mips_bclri_d`。
- **L569 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L569 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L570 EN**: Blank line separating nearby declarations or logic blocks.
  **L570 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L571 EN**: Declares TableGen def `int_mips_binsl_b`.
  **L571 CN**: 声明 TableGen def `int_mips_binsl_b`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L573 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L573 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L574 EN**: Declares TableGen def `int_mips_binsl_h`.
  **L574 CN**: 声明 TableGen def `int_mips_binsl_h`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L576 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L576 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L577 EN**: Declares TableGen def `int_mips_binsl_w`.
  **L577 CN**: 声明 TableGen def `int_mips_binsl_w`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L579 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L579 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L580 EN**: Declares TableGen def `int_mips_binsl_d`.
  **L580 CN**: 声明 TableGen def `int_mips_binsl_d`。
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L582 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L582 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L583 EN**: Blank line separating nearby declarations or logic blocks.
  **L583 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L584 EN**: Declares TableGen def `int_mips_binsli_b`.
  **L584 CN**: 声明 TableGen def `int_mips_binsli_b`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L586 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L586 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L587 EN**: Declares TableGen def `int_mips_binsli_h`.
  **L587 CN**: 声明 TableGen def `int_mips_binsli_h`。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`.
  **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`。

### Lines 589-616

````tablegen
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_binsli_w : ClangBuiltin<"__builtin_msa_binsli_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_binsli_d : ClangBuiltin<"__builtin_msa_binsli_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_binsr_b : ClangBuiltin<"__builtin_msa_binsr_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
            [IntrNoMem]>;
def int_mips_binsr_h : ClangBuiltin<"__builtin_msa_binsr_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
            [IntrNoMem]>;
def int_mips_binsr_w : ClangBuiltin<"__builtin_msa_binsr_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
            [IntrNoMem]>;
def int_mips_binsr_d : ClangBuiltin<"__builtin_msa_binsr_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
            [IntrNoMem]>;

def int_mips_binsri_b : ClangBuiltin<"__builtin_msa_binsri_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_binsri_h : ClangBuiltin<"__builtin_msa_binsri_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_binsri_w : ClangBuiltin<"__builtin_msa_binsri_w">,
````
- **L589 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L589 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L590 EN**: Declares TableGen def `int_mips_binsli_w`.
  **L590 CN**: 声明 TableGen def `int_mips_binsli_w`。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`。
- **L592 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L592 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L593 EN**: Declares TableGen def `int_mips_binsli_d`.
  **L593 CN**: 声明 TableGen def `int_mips_binsli_d`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`。
- **L595 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L595 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L597 EN**: Declares TableGen def `int_mips_binsr_b`.
  **L597 CN**: 声明 TableGen def `int_mips_binsr_b`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L599 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L599 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L600 EN**: Declares TableGen def `int_mips_binsr_h`.
  **L600 CN**: 声明 TableGen def `int_mips_binsr_h`。
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L602 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L602 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L603 EN**: Declares TableGen def `int_mips_binsr_w`.
  **L603 CN**: 声明 TableGen def `int_mips_binsr_w`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L605 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L605 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L606 EN**: Declares TableGen def `int_mips_binsr_d`.
  **L606 CN**: 声明 TableGen def `int_mips_binsr_d`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L608 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L608 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L609 EN**: Blank line separating nearby declarations or logic blocks.
  **L609 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L610 EN**: Declares TableGen def `int_mips_binsri_b`.
  **L610 CN**: 声明 TableGen def `int_mips_binsri_b`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L612 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L612 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L613 EN**: Declares TableGen def `int_mips_binsri_h`.
  **L613 CN**: 声明 TableGen def `int_mips_binsri_h`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`。
- **L615 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L615 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L616 EN**: Declares TableGen def `int_mips_binsri_w`.
  **L616 CN**: 声明 TableGen def `int_mips_binsri_w`。

### Lines 617-644

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_binsri_d : ClangBuiltin<"__builtin_msa_binsri_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_bmnz_v : ClangBuiltin<"__builtin_msa_bmnz_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
            [IntrNoMem]>;

def int_mips_bmnzi_b : ClangBuiltin<"__builtin_msa_bmnzi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_bmz_v : ClangBuiltin<"__builtin_msa_bmz_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
            [IntrNoMem]>;

def int_mips_bmzi_b : ClangBuiltin<"__builtin_msa_bmzi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_bneg_b : ClangBuiltin<"__builtin_msa_bneg_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_bneg_h : ClangBuiltin<"__builtin_msa_bneg_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_bneg_w : ClangBuiltin<"__builtin_msa_bneg_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
````
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`.
  **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`。
- **L618 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L618 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L619 EN**: Declares TableGen def `int_mips_binsri_d`.
  **L619 CN**: 声明 TableGen def `int_mips_binsri_d`。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`。
- **L621 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L621 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L623 EN**: Declares TableGen def `int_mips_bmnz_v`.
  **L623 CN**: 声明 TableGen def `int_mips_bmnz_v`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L625 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L625 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L627 EN**: Declares TableGen def `int_mips_bmnzi_b`.
  **L627 CN**: 声明 TableGen def `int_mips_bmnzi_b`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L629 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L629 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Declares TableGen def `int_mips_bmz_v`.
  **L631 CN**: 声明 TableGen def `int_mips_bmz_v`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L633 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L633 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L635 EN**: Declares TableGen def `int_mips_bmzi_b`.
  **L635 CN**: 声明 TableGen def `int_mips_bmzi_b`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L637 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L637 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L639 EN**: Declares TableGen def `int_mips_bneg_b`.
  **L639 CN**: 声明 TableGen def `int_mips_bneg_b`。
- **L640 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L640 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L641 EN**: Declares TableGen def `int_mips_bneg_h`.
  **L641 CN**: 声明 TableGen def `int_mips_bneg_h`。
- **L642 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L642 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L643 EN**: Declares TableGen def `int_mips_bneg_w`.
  **L643 CN**: 声明 TableGen def `int_mips_bneg_w`。
- **L644 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L644 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。

### Lines 645-672

````tablegen
def int_mips_bneg_d : ClangBuiltin<"__builtin_msa_bneg_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_bnegi_b : ClangBuiltin<"__builtin_msa_bnegi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bnegi_h : ClangBuiltin<"__builtin_msa_bnegi_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bnegi_w : ClangBuiltin<"__builtin_msa_bnegi_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bnegi_d : ClangBuiltin<"__builtin_msa_bnegi_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_bnz_b : ClangBuiltin<"__builtin_msa_bnz_b">,
  Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_bnz_h : ClangBuiltin<"__builtin_msa_bnz_h">,
  Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_bnz_w : ClangBuiltin<"__builtin_msa_bnz_w">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_bnz_d : ClangBuiltin<"__builtin_msa_bnz_d">,
  Intrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_bnz_v : ClangBuiltin<"__builtin_msa_bnz_v">,
  Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;

def int_mips_bsel_v : ClangBuiltin<"__builtin_msa_bsel_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
            [IntrNoMem]>;

````
- **L645 EN**: Declares TableGen def `int_mips_bneg_d`.
  **L645 CN**: 声明 TableGen def `int_mips_bneg_d`。
- **L646 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L646 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L648 EN**: Declares TableGen def `int_mips_bnegi_b`.
  **L648 CN**: 声明 TableGen def `int_mips_bnegi_b`。
- **L649 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L649 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L650 EN**: Declares TableGen def `int_mips_bnegi_h`.
  **L650 CN**: 声明 TableGen def `int_mips_bnegi_h`。
- **L651 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L651 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L652 EN**: Declares TableGen def `int_mips_bnegi_w`.
  **L652 CN**: 声明 TableGen def `int_mips_bnegi_w`。
- **L653 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L653 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L654 EN**: Declares TableGen def `int_mips_bnegi_d`.
  **L654 CN**: 声明 TableGen def `int_mips_bnegi_d`。
- **L655 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L655 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L657 EN**: Declares TableGen def `int_mips_bnz_b`.
  **L657 CN**: 声明 TableGen def `int_mips_bnz_b`。
- **L658 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L658 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L659 EN**: Declares TableGen def `int_mips_bnz_h`.
  **L659 CN**: 声明 TableGen def `int_mips_bnz_h`。
- **L660 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L660 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L661 EN**: Declares TableGen def `int_mips_bnz_w`.
  **L661 CN**: 声明 TableGen def `int_mips_bnz_w`。
- **L662 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L662 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L663 EN**: Declares TableGen def `int_mips_bnz_d`.
  **L663 CN**: 声明 TableGen def `int_mips_bnz_d`。
- **L664 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L664 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L666 EN**: Declares TableGen def `int_mips_bnz_v`.
  **L666 CN**: 声明 TableGen def `int_mips_bnz_v`。
- **L667 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L667 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L669 EN**: Declares TableGen def `int_mips_bsel_v`.
  **L669 CN**: 声明 TableGen def `int_mips_bsel_v`。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L671 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L671 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L672 EN**: Blank line separating nearby declarations or logic blocks.
  **L672 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 673-700

````tablegen
def int_mips_bseli_b : ClangBuiltin<"__builtin_msa_bseli_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_bset_b : ClangBuiltin<"__builtin_msa_bset_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_bset_h : ClangBuiltin<"__builtin_msa_bset_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_bset_w : ClangBuiltin<"__builtin_msa_bset_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_bset_d : ClangBuiltin<"__builtin_msa_bset_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_bseti_b : ClangBuiltin<"__builtin_msa_bseti_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bseti_h : ClangBuiltin<"__builtin_msa_bseti_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bseti_w : ClangBuiltin<"__builtin_msa_bseti_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_bseti_d : ClangBuiltin<"__builtin_msa_bseti_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_bz_b : ClangBuiltin<"__builtin_msa_bz_b">,
  Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_bz_h : ClangBuiltin<"__builtin_msa_bz_h">,
  Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_bz_w : ClangBuiltin<"__builtin_msa_bz_w">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
````
- **L673 EN**: Declares TableGen def `int_mips_bseli_b`.
  **L673 CN**: 声明 TableGen def `int_mips_bseli_b`。
- **L674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L674 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L675 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L675 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L677 EN**: Declares TableGen def `int_mips_bset_b`.
  **L677 CN**: 声明 TableGen def `int_mips_bset_b`。
- **L678 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L678 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L679 EN**: Declares TableGen def `int_mips_bset_h`.
  **L679 CN**: 声明 TableGen def `int_mips_bset_h`。
- **L680 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L680 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L681 EN**: Declares TableGen def `int_mips_bset_w`.
  **L681 CN**: 声明 TableGen def `int_mips_bset_w`。
- **L682 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L682 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L683 EN**: Declares TableGen def `int_mips_bset_d`.
  **L683 CN**: 声明 TableGen def `int_mips_bset_d`。
- **L684 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L684 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L685 EN**: Blank line separating nearby declarations or logic blocks.
  **L685 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L686 EN**: Declares TableGen def `int_mips_bseti_b`.
  **L686 CN**: 声明 TableGen def `int_mips_bseti_b`。
- **L687 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L687 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L688 EN**: Declares TableGen def `int_mips_bseti_h`.
  **L688 CN**: 声明 TableGen def `int_mips_bseti_h`。
- **L689 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L689 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L690 EN**: Declares TableGen def `int_mips_bseti_w`.
  **L690 CN**: 声明 TableGen def `int_mips_bseti_w`。
- **L691 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L691 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L692 EN**: Declares TableGen def `int_mips_bseti_d`.
  **L692 CN**: 声明 TableGen def `int_mips_bseti_d`。
- **L693 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L693 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L694 EN**: Blank line separating nearby declarations or logic blocks.
  **L694 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L695 EN**: Declares TableGen def `int_mips_bz_b`.
  **L695 CN**: 声明 TableGen def `int_mips_bz_b`。
- **L696 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L696 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L697 EN**: Declares TableGen def `int_mips_bz_h`.
  **L697 CN**: 声明 TableGen def `int_mips_bz_h`。
- **L698 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L698 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L699 EN**: Declares TableGen def `int_mips_bz_w`.
  **L699 CN**: 声明 TableGen def `int_mips_bz_w`。
- **L700 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L700 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。

### Lines 701-728

````tablegen
def int_mips_bz_d : ClangBuiltin<"__builtin_msa_bz_d">,
  Intrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_bz_v : ClangBuiltin<"__builtin_msa_bz_v">,
  Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;

def int_mips_ceq_b : ClangBuiltin<"__builtin_msa_ceq_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_ceq_h : ClangBuiltin<"__builtin_msa_ceq_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_ceq_w : ClangBuiltin<"__builtin_msa_ceq_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_ceq_d : ClangBuiltin<"__builtin_msa_ceq_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_ceqi_b : ClangBuiltin<"__builtin_msa_ceqi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_ceqi_h : ClangBuiltin<"__builtin_msa_ceqi_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_ceqi_w : ClangBuiltin<"__builtin_msa_ceqi_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_ceqi_d : ClangBuiltin<"__builtin_msa_ceqi_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_cfcmsa : ClangBuiltin<"__builtin_msa_cfcmsa">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

def int_mips_cle_s_b : ClangBuiltin<"__builtin_msa_cle_s_b">,
````
- **L701 EN**: Declares TableGen def `int_mips_bz_d`.
  **L701 CN**: 声明 TableGen def `int_mips_bz_d`。
- **L702 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L702 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L703 EN**: Blank line separating nearby declarations or logic blocks.
  **L703 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L704 EN**: Declares TableGen def `int_mips_bz_v`.
  **L704 CN**: 声明 TableGen def `int_mips_bz_v`。
- **L705 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L705 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L706 EN**: Blank line separating nearby declarations or logic blocks.
  **L706 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L707 EN**: Declares TableGen def `int_mips_ceq_b`.
  **L707 CN**: 声明 TableGen def `int_mips_ceq_b`。
- **L708 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L708 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L709 EN**: Declares TableGen def `int_mips_ceq_h`.
  **L709 CN**: 声明 TableGen def `int_mips_ceq_h`。
- **L710 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L710 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L711 EN**: Declares TableGen def `int_mips_ceq_w`.
  **L711 CN**: 声明 TableGen def `int_mips_ceq_w`。
- **L712 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L712 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L713 EN**: Declares TableGen def `int_mips_ceq_d`.
  **L713 CN**: 声明 TableGen def `int_mips_ceq_d`。
- **L714 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L714 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L716 EN**: Declares TableGen def `int_mips_ceqi_b`.
  **L716 CN**: 声明 TableGen def `int_mips_ceqi_b`。
- **L717 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L717 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L718 EN**: Declares TableGen def `int_mips_ceqi_h`.
  **L718 CN**: 声明 TableGen def `int_mips_ceqi_h`。
- **L719 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L719 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L720 EN**: Declares TableGen def `int_mips_ceqi_w`.
  **L720 CN**: 声明 TableGen def `int_mips_ceqi_w`。
- **L721 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L721 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L722 EN**: Declares TableGen def `int_mips_ceqi_d`.
  **L722 CN**: 声明 TableGen def `int_mips_ceqi_d`。
- **L723 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L723 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L725 EN**: Declares TableGen def `int_mips_cfcmsa`.
  **L725 CN**: 声明 TableGen def `int_mips_cfcmsa`。
- **L726 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L726 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L727 EN**: Blank line separating nearby declarations or logic blocks.
  **L727 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L728 EN**: Declares TableGen def `int_mips_cle_s_b`.
  **L728 CN**: 声明 TableGen def `int_mips_cle_s_b`。

### Lines 729-756

````tablegen
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_cle_s_h : ClangBuiltin<"__builtin_msa_cle_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_cle_s_w : ClangBuiltin<"__builtin_msa_cle_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_cle_s_d : ClangBuiltin<"__builtin_msa_cle_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_cle_u_b : ClangBuiltin<"__builtin_msa_cle_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_cle_u_h : ClangBuiltin<"__builtin_msa_cle_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_cle_u_w : ClangBuiltin<"__builtin_msa_cle_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_cle_u_d : ClangBuiltin<"__builtin_msa_cle_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_clei_s_b : ClangBuiltin<"__builtin_msa_clei_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clei_s_h : ClangBuiltin<"__builtin_msa_clei_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clei_s_w : ClangBuiltin<"__builtin_msa_clei_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clei_s_d : ClangBuiltin<"__builtin_msa_clei_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_clei_u_b : ClangBuiltin<"__builtin_msa_clei_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L729 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L729 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L730 EN**: Declares TableGen def `int_mips_cle_s_h`.
  **L730 CN**: 声明 TableGen def `int_mips_cle_s_h`。
- **L731 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L731 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L732 EN**: Declares TableGen def `int_mips_cle_s_w`.
  **L732 CN**: 声明 TableGen def `int_mips_cle_s_w`。
- **L733 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L733 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L734 EN**: Declares TableGen def `int_mips_cle_s_d`.
  **L734 CN**: 声明 TableGen def `int_mips_cle_s_d`。
- **L735 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L735 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L736 EN**: Blank line separating nearby declarations or logic blocks.
  **L736 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L737 EN**: Declares TableGen def `int_mips_cle_u_b`.
  **L737 CN**: 声明 TableGen def `int_mips_cle_u_b`。
- **L738 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L738 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L739 EN**: Declares TableGen def `int_mips_cle_u_h`.
  **L739 CN**: 声明 TableGen def `int_mips_cle_u_h`。
- **L740 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L740 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L741 EN**: Declares TableGen def `int_mips_cle_u_w`.
  **L741 CN**: 声明 TableGen def `int_mips_cle_u_w`。
- **L742 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L742 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L743 EN**: Declares TableGen def `int_mips_cle_u_d`.
  **L743 CN**: 声明 TableGen def `int_mips_cle_u_d`。
- **L744 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L744 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L746 EN**: Declares TableGen def `int_mips_clei_s_b`.
  **L746 CN**: 声明 TableGen def `int_mips_clei_s_b`。
- **L747 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L747 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L748 EN**: Declares TableGen def `int_mips_clei_s_h`.
  **L748 CN**: 声明 TableGen def `int_mips_clei_s_h`。
- **L749 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L749 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L750 EN**: Declares TableGen def `int_mips_clei_s_w`.
  **L750 CN**: 声明 TableGen def `int_mips_clei_s_w`。
- **L751 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L751 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L752 EN**: Declares TableGen def `int_mips_clei_s_d`.
  **L752 CN**: 声明 TableGen def `int_mips_clei_s_d`。
- **L753 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L753 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L755 EN**: Declares TableGen def `int_mips_clei_u_b`.
  **L755 CN**: 声明 TableGen def `int_mips_clei_u_b`。
- **L756 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L756 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 757-784

````tablegen
def int_mips_clei_u_h : ClangBuiltin<"__builtin_msa_clei_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clei_u_w : ClangBuiltin<"__builtin_msa_clei_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clei_u_d : ClangBuiltin<"__builtin_msa_clei_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_clt_s_b : ClangBuiltin<"__builtin_msa_clt_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_clt_s_h : ClangBuiltin<"__builtin_msa_clt_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_clt_s_w : ClangBuiltin<"__builtin_msa_clt_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_clt_s_d : ClangBuiltin<"__builtin_msa_clt_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_clt_u_b : ClangBuiltin<"__builtin_msa_clt_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_clt_u_h : ClangBuiltin<"__builtin_msa_clt_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_clt_u_w : ClangBuiltin<"__builtin_msa_clt_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_clt_u_d : ClangBuiltin<"__builtin_msa_clt_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_clti_s_b : ClangBuiltin<"__builtin_msa_clti_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clti_s_h : ClangBuiltin<"__builtin_msa_clti_s_h">,
````
- **L757 EN**: Declares TableGen def `int_mips_clei_u_h`.
  **L757 CN**: 声明 TableGen def `int_mips_clei_u_h`。
- **L758 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L758 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L759 EN**: Declares TableGen def `int_mips_clei_u_w`.
  **L759 CN**: 声明 TableGen def `int_mips_clei_u_w`。
- **L760 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L760 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L761 EN**: Declares TableGen def `int_mips_clei_u_d`.
  **L761 CN**: 声明 TableGen def `int_mips_clei_u_d`。
- **L762 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L762 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L763 EN**: Blank line separating nearby declarations or logic blocks.
  **L763 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L764 EN**: Declares TableGen def `int_mips_clt_s_b`.
  **L764 CN**: 声明 TableGen def `int_mips_clt_s_b`。
- **L765 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L765 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L766 EN**: Declares TableGen def `int_mips_clt_s_h`.
  **L766 CN**: 声明 TableGen def `int_mips_clt_s_h`。
- **L767 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L767 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L768 EN**: Declares TableGen def `int_mips_clt_s_w`.
  **L768 CN**: 声明 TableGen def `int_mips_clt_s_w`。
- **L769 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L769 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L770 EN**: Declares TableGen def `int_mips_clt_s_d`.
  **L770 CN**: 声明 TableGen def `int_mips_clt_s_d`。
- **L771 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L771 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L773 EN**: Declares TableGen def `int_mips_clt_u_b`.
  **L773 CN**: 声明 TableGen def `int_mips_clt_u_b`。
- **L774 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L774 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L775 EN**: Declares TableGen def `int_mips_clt_u_h`.
  **L775 CN**: 声明 TableGen def `int_mips_clt_u_h`。
- **L776 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L776 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L777 EN**: Declares TableGen def `int_mips_clt_u_w`.
  **L777 CN**: 声明 TableGen def `int_mips_clt_u_w`。
- **L778 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L778 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L779 EN**: Declares TableGen def `int_mips_clt_u_d`.
  **L779 CN**: 声明 TableGen def `int_mips_clt_u_d`。
- **L780 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L780 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L781 EN**: Blank line separating nearby declarations or logic blocks.
  **L781 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L782 EN**: Declares TableGen def `int_mips_clti_s_b`.
  **L782 CN**: 声明 TableGen def `int_mips_clti_s_b`。
- **L783 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L783 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L784 EN**: Declares TableGen def `int_mips_clti_s_h`.
  **L784 CN**: 声明 TableGen def `int_mips_clti_s_h`。

### Lines 785-812

````tablegen
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clti_s_w : ClangBuiltin<"__builtin_msa_clti_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clti_s_d : ClangBuiltin<"__builtin_msa_clti_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_clti_u_b : ClangBuiltin<"__builtin_msa_clti_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clti_u_h : ClangBuiltin<"__builtin_msa_clti_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clti_u_w : ClangBuiltin<"__builtin_msa_clti_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_clti_u_d : ClangBuiltin<"__builtin_msa_clti_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_copy_s_b : ClangBuiltin<"__builtin_msa_copy_s_b">,
  Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_copy_s_h : ClangBuiltin<"__builtin_msa_copy_s_h">,
  Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_copy_s_w : ClangBuiltin<"__builtin_msa_copy_s_w">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_copy_s_d : ClangBuiltin<"__builtin_msa_copy_s_d">,
  Intrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;

def int_mips_copy_u_b : ClangBuiltin<"__builtin_msa_copy_u_b">,
  Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_copy_u_h : ClangBuiltin<"__builtin_msa_copy_u_h">,
  Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;
````
- **L785 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L785 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L786 EN**: Declares TableGen def `int_mips_clti_s_w`.
  **L786 CN**: 声明 TableGen def `int_mips_clti_s_w`。
- **L787 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L787 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L788 EN**: Declares TableGen def `int_mips_clti_s_d`.
  **L788 CN**: 声明 TableGen def `int_mips_clti_s_d`。
- **L789 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L789 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L790 EN**: Blank line separating nearby declarations or logic blocks.
  **L790 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L791 EN**: Declares TableGen def `int_mips_clti_u_b`.
  **L791 CN**: 声明 TableGen def `int_mips_clti_u_b`。
- **L792 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L792 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L793 EN**: Declares TableGen def `int_mips_clti_u_h`.
  **L793 CN**: 声明 TableGen def `int_mips_clti_u_h`。
- **L794 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L794 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L795 EN**: Declares TableGen def `int_mips_clti_u_w`.
  **L795 CN**: 声明 TableGen def `int_mips_clti_u_w`。
- **L796 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L796 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L797 EN**: Declares TableGen def `int_mips_clti_u_d`.
  **L797 CN**: 声明 TableGen def `int_mips_clti_u_d`。
- **L798 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L798 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L800 EN**: Declares TableGen def `int_mips_copy_s_b`.
  **L800 CN**: 声明 TableGen def `int_mips_copy_s_b`。
- **L801 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L801 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L802 EN**: Declares TableGen def `int_mips_copy_s_h`.
  **L802 CN**: 声明 TableGen def `int_mips_copy_s_h`。
- **L803 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L803 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L804 EN**: Declares TableGen def `int_mips_copy_s_w`.
  **L804 CN**: 声明 TableGen def `int_mips_copy_s_w`。
- **L805 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L805 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L806 EN**: Declares TableGen def `int_mips_copy_s_d`.
  **L806 CN**: 声明 TableGen def `int_mips_copy_s_d`。
- **L807 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L807 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L809 EN**: Declares TableGen def `int_mips_copy_u_b`.
  **L809 CN**: 声明 TableGen def `int_mips_copy_u_b`。
- **L810 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L810 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L811 EN**: Declares TableGen def `int_mips_copy_u_h`.
  **L811 CN**: 声明 TableGen def `int_mips_copy_u_h`。
- **L812 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L812 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`。

### Lines 813-840

````tablegen
def int_mips_copy_u_w : ClangBuiltin<"__builtin_msa_copy_u_w">,
  Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_copy_u_d : ClangBuiltin<"__builtin_msa_copy_u_d">,
  Intrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;

def int_mips_ctcmsa : ClangBuiltin<"__builtin_msa_ctcmsa">,
  Intrinsic<[], [llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

def int_mips_div_s_b : ClangBuiltin<"__builtin_msa_div_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_div_s_h : ClangBuiltin<"__builtin_msa_div_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_div_s_w : ClangBuiltin<"__builtin_msa_div_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_div_s_d : ClangBuiltin<"__builtin_msa_div_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_div_u_b : ClangBuiltin<"__builtin_msa_div_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_div_u_h : ClangBuiltin<"__builtin_msa_div_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_div_u_w : ClangBuiltin<"__builtin_msa_div_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_div_u_d : ClangBuiltin<"__builtin_msa_div_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

// This instruction is part of the MSA spec but it does not share the
// __builtin_msa prefix because it operates on GP registers.
````
- **L813 EN**: Declares TableGen def `int_mips_copy_u_w`.
  **L813 CN**: 声明 TableGen def `int_mips_copy_u_w`。
- **L814 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L814 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L815 EN**: Declares TableGen def `int_mips_copy_u_d`.
  **L815 CN**: 声明 TableGen def `int_mips_copy_u_d`。
- **L816 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L816 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L818 EN**: Declares TableGen def `int_mips_ctcmsa`.
  **L818 CN**: 声明 TableGen def `int_mips_ctcmsa`。
- **L819 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L819 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L820 EN**: Blank line separating nearby declarations or logic blocks.
  **L820 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L821 EN**: Declares TableGen def `int_mips_div_s_b`.
  **L821 CN**: 声明 TableGen def `int_mips_div_s_b`。
- **L822 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L822 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L823 EN**: Declares TableGen def `int_mips_div_s_h`.
  **L823 CN**: 声明 TableGen def `int_mips_div_s_h`。
- **L824 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L824 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L825 EN**: Declares TableGen def `int_mips_div_s_w`.
  **L825 CN**: 声明 TableGen def `int_mips_div_s_w`。
- **L826 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L826 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L827 EN**: Declares TableGen def `int_mips_div_s_d`.
  **L827 CN**: 声明 TableGen def `int_mips_div_s_d`。
- **L828 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L828 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L829 EN**: Blank line separating nearby declarations or logic blocks.
  **L829 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L830 EN**: Declares TableGen def `int_mips_div_u_b`.
  **L830 CN**: 声明 TableGen def `int_mips_div_u_b`。
- **L831 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L831 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L832 EN**: Declares TableGen def `int_mips_div_u_h`.
  **L832 CN**: 声明 TableGen def `int_mips_div_u_h`。
- **L833 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L833 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L834 EN**: Declares TableGen def `int_mips_div_u_w`.
  **L834 CN**: 声明 TableGen def `int_mips_div_u_w`。
- **L835 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L835 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L836 EN**: Declares TableGen def `int_mips_div_u_d`.
  **L836 CN**: 声明 TableGen def `int_mips_div_u_d`。
- **L837 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L837 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L839 EN**: Comment explains nearby logic, invariants, or intent: `This instruction is part of the MSA spec but it does not share the`.
  **L839 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This instruction is part of the MSA spec but it does not share the`。
- **L840 EN**: Comment explains nearby logic, invariants, or intent: `__builtin_msa prefix because it operates on GP registers.`.
  **L840 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__builtin_msa prefix because it operates on GP registers.`。

### Lines 841-868

````tablegen
def int_mips_dlsa : ClangBuiltin<"__builtin_mips_dlsa">,
  Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],
            [IntrNoMem]>;

def int_mips_dotp_s_h : ClangBuiltin<"__builtin_msa_dotp_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_dotp_s_w : ClangBuiltin<"__builtin_msa_dotp_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_dotp_s_d : ClangBuiltin<"__builtin_msa_dotp_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_dotp_u_h : ClangBuiltin<"__builtin_msa_dotp_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_dotp_u_w : ClangBuiltin<"__builtin_msa_dotp_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_dotp_u_d : ClangBuiltin<"__builtin_msa_dotp_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_dpadd_s_h : ClangBuiltin<"__builtin_msa_dpadd_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],
  [IntrNoMem]>;
def int_mips_dpadd_s_w : ClangBuiltin<"__builtin_msa_dpadd_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_dpadd_s_d : ClangBuiltin<"__builtin_msa_dpadd_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;

````
- **L841 EN**: Declares TableGen def `int_mips_dlsa`.
  **L841 CN**: 声明 TableGen def `int_mips_dlsa`。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`。
- **L843 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L843 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L845 EN**: Declares TableGen def `int_mips_dotp_s_h`.
  **L845 CN**: 声明 TableGen def `int_mips_dotp_s_h`。
- **L846 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L846 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L847 EN**: Declares TableGen def `int_mips_dotp_s_w`.
  **L847 CN**: 声明 TableGen def `int_mips_dotp_s_w`。
- **L848 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L848 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L849 EN**: Declares TableGen def `int_mips_dotp_s_d`.
  **L849 CN**: 声明 TableGen def `int_mips_dotp_s_d`。
- **L850 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L850 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L851 EN**: Blank line separating nearby declarations or logic blocks.
  **L851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L852 EN**: Declares TableGen def `int_mips_dotp_u_h`.
  **L852 CN**: 声明 TableGen def `int_mips_dotp_u_h`。
- **L853 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L853 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L854 EN**: Declares TableGen def `int_mips_dotp_u_w`.
  **L854 CN**: 声明 TableGen def `int_mips_dotp_u_w`。
- **L855 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L855 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L856 EN**: Declares TableGen def `int_mips_dotp_u_d`.
  **L856 CN**: 声明 TableGen def `int_mips_dotp_u_d`。
- **L857 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L857 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L859 EN**: Declares TableGen def `int_mips_dpadd_s_h`.
  **L859 CN**: 声明 TableGen def `int_mips_dpadd_s_h`。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L861 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L861 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L862 EN**: Declares TableGen def `int_mips_dpadd_s_w`.
  **L862 CN**: 声明 TableGen def `int_mips_dpadd_s_w`。
- **L863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L863 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L864 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L864 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L865 EN**: Declares TableGen def `int_mips_dpadd_s_d`.
  **L865 CN**: 声明 TableGen def `int_mips_dpadd_s_d`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L867 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L867 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 869-896

````tablegen
def int_mips_dpadd_u_h : ClangBuiltin<"__builtin_msa_dpadd_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],
  [IntrNoMem]>;
def int_mips_dpadd_u_w : ClangBuiltin<"__builtin_msa_dpadd_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_dpadd_u_d : ClangBuiltin<"__builtin_msa_dpadd_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;

def int_mips_dpsub_s_h : ClangBuiltin<"__builtin_msa_dpsub_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],
  [IntrNoMem]>;
def int_mips_dpsub_s_w : ClangBuiltin<"__builtin_msa_dpsub_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_dpsub_s_d : ClangBuiltin<"__builtin_msa_dpsub_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;

def int_mips_dpsub_u_h : ClangBuiltin<"__builtin_msa_dpsub_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],
  [IntrNoMem]>;
def int_mips_dpsub_u_w : ClangBuiltin<"__builtin_msa_dpsub_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_dpsub_u_d : ClangBuiltin<"__builtin_msa_dpsub_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],
````
- **L869 EN**: Declares TableGen def `int_mips_dpadd_u_h`.
  **L869 CN**: 声明 TableGen def `int_mips_dpadd_u_h`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L871 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L871 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L872 EN**: Declares TableGen def `int_mips_dpadd_u_w`.
  **L872 CN**: 声明 TableGen def `int_mips_dpadd_u_w`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L874 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L874 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L875 EN**: Declares TableGen def `int_mips_dpadd_u_d`.
  **L875 CN**: 声明 TableGen def `int_mips_dpadd_u_d`。
- **L876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L876 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L877 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L877 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L878 EN**: Blank line separating nearby declarations or logic blocks.
  **L878 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L879 EN**: Declares TableGen def `int_mips_dpsub_s_h`.
  **L879 CN**: 声明 TableGen def `int_mips_dpsub_s_h`。
- **L880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L880 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L881 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L881 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L882 EN**: Declares TableGen def `int_mips_dpsub_s_w`.
  **L882 CN**: 声明 TableGen def `int_mips_dpsub_s_w`。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L884 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L884 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L885 EN**: Declares TableGen def `int_mips_dpsub_s_d`.
  **L885 CN**: 声明 TableGen def `int_mips_dpsub_s_d`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L887 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L887 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L888 EN**: Blank line separating nearby declarations or logic blocks.
  **L888 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L889 EN**: Declares TableGen def `int_mips_dpsub_u_h`.
  **L889 CN**: 声明 TableGen def `int_mips_dpsub_u_h`。
- **L890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L890 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L891 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L891 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L892 EN**: Declares TableGen def `int_mips_dpsub_u_w`.
  **L892 CN**: 声明 TableGen def `int_mips_dpsub_u_w`。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L894 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L894 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L895 EN**: Declares TableGen def `int_mips_dpsub_u_d`.
  **L895 CN**: 声明 TableGen def `int_mips_dpsub_u_d`。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。

### Lines 897-924

````tablegen
  [IntrNoMem]>;

def int_mips_fadd_w : ClangBuiltin<"__builtin_msa_fadd_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fadd_d : ClangBuiltin<"__builtin_msa_fadd_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcaf_w : ClangBuiltin<"__builtin_msa_fcaf_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcaf_d : ClangBuiltin<"__builtin_msa_fcaf_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fceq_w : ClangBuiltin<"__builtin_msa_fceq_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fceq_d : ClangBuiltin<"__builtin_msa_fceq_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcle_w : ClangBuiltin<"__builtin_msa_fcle_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcle_d : ClangBuiltin<"__builtin_msa_fcle_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fclt_w : ClangBuiltin<"__builtin_msa_fclt_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fclt_d : ClangBuiltin<"__builtin_msa_fclt_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fclass_w : ClangBuiltin<"__builtin_msa_fclass_w">,
````
- **L897 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L897 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L899 EN**: Declares TableGen def `int_mips_fadd_w`.
  **L899 CN**: 声明 TableGen def `int_mips_fadd_w`。
- **L900 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L900 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L901 EN**: Declares TableGen def `int_mips_fadd_d`.
  **L901 CN**: 声明 TableGen def `int_mips_fadd_d`。
- **L902 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L902 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L904 EN**: Declares TableGen def `int_mips_fcaf_w`.
  **L904 CN**: 声明 TableGen def `int_mips_fcaf_w`。
- **L905 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L905 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L906 EN**: Declares TableGen def `int_mips_fcaf_d`.
  **L906 CN**: 声明 TableGen def `int_mips_fcaf_d`。
- **L907 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L907 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L909 EN**: Declares TableGen def `int_mips_fceq_w`.
  **L909 CN**: 声明 TableGen def `int_mips_fceq_w`。
- **L910 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L910 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L911 EN**: Declares TableGen def `int_mips_fceq_d`.
  **L911 CN**: 声明 TableGen def `int_mips_fceq_d`。
- **L912 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L912 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L914 EN**: Declares TableGen def `int_mips_fcle_w`.
  **L914 CN**: 声明 TableGen def `int_mips_fcle_w`。
- **L915 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L915 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L916 EN**: Declares TableGen def `int_mips_fcle_d`.
  **L916 CN**: 声明 TableGen def `int_mips_fcle_d`。
- **L917 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L917 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L918 EN**: Blank line separating nearby declarations or logic blocks.
  **L918 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L919 EN**: Declares TableGen def `int_mips_fclt_w`.
  **L919 CN**: 声明 TableGen def `int_mips_fclt_w`。
- **L920 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L920 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L921 EN**: Declares TableGen def `int_mips_fclt_d`.
  **L921 CN**: 声明 TableGen def `int_mips_fclt_d`。
- **L922 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L922 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L923 EN**: Blank line separating nearby declarations or logic blocks.
  **L923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L924 EN**: Declares TableGen def `int_mips_fclass_w`.
  **L924 CN**: 声明 TableGen def `int_mips_fclass_w`。

### Lines 925-952

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fclass_d : ClangBuiltin<"__builtin_msa_fclass_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcne_w : ClangBuiltin<"__builtin_msa_fcne_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcne_d : ClangBuiltin<"__builtin_msa_fcne_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcor_w : ClangBuiltin<"__builtin_msa_fcor_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcor_d : ClangBuiltin<"__builtin_msa_fcor_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcueq_w : ClangBuiltin<"__builtin_msa_fcueq_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcueq_d : ClangBuiltin<"__builtin_msa_fcueq_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcule_w : ClangBuiltin<"__builtin_msa_fcule_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcule_d : ClangBuiltin<"__builtin_msa_fcule_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcult_w : ClangBuiltin<"__builtin_msa_fcult_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcult_d : ClangBuiltin<"__builtin_msa_fcult_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
````
- **L925 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L925 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L926 EN**: Declares TableGen def `int_mips_fclass_d`.
  **L926 CN**: 声明 TableGen def `int_mips_fclass_d`。
- **L927 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L927 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L928 EN**: Blank line separating nearby declarations or logic blocks.
  **L928 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L929 EN**: Declares TableGen def `int_mips_fcne_w`.
  **L929 CN**: 声明 TableGen def `int_mips_fcne_w`。
- **L930 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L930 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L931 EN**: Declares TableGen def `int_mips_fcne_d`.
  **L931 CN**: 声明 TableGen def `int_mips_fcne_d`。
- **L932 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L932 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L933 EN**: Blank line separating nearby declarations or logic blocks.
  **L933 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L934 EN**: Declares TableGen def `int_mips_fcor_w`.
  **L934 CN**: 声明 TableGen def `int_mips_fcor_w`。
- **L935 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L935 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L936 EN**: Declares TableGen def `int_mips_fcor_d`.
  **L936 CN**: 声明 TableGen def `int_mips_fcor_d`。
- **L937 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L937 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L938 EN**: Blank line separating nearby declarations or logic blocks.
  **L938 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L939 EN**: Declares TableGen def `int_mips_fcueq_w`.
  **L939 CN**: 声明 TableGen def `int_mips_fcueq_w`。
- **L940 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L940 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L941 EN**: Declares TableGen def `int_mips_fcueq_d`.
  **L941 CN**: 声明 TableGen def `int_mips_fcueq_d`。
- **L942 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L942 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L944 EN**: Declares TableGen def `int_mips_fcule_w`.
  **L944 CN**: 声明 TableGen def `int_mips_fcule_w`。
- **L945 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L945 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L946 EN**: Declares TableGen def `int_mips_fcule_d`.
  **L946 CN**: 声明 TableGen def `int_mips_fcule_d`。
- **L947 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L947 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L949 EN**: Declares TableGen def `int_mips_fcult_w`.
  **L949 CN**: 声明 TableGen def `int_mips_fcult_w`。
- **L950 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L950 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L951 EN**: Declares TableGen def `int_mips_fcult_d`.
  **L951 CN**: 声明 TableGen def `int_mips_fcult_d`。
- **L952 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L952 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。

### Lines 953-980

````tablegen

def int_mips_fcun_w : ClangBuiltin<"__builtin_msa_fcun_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcun_d : ClangBuiltin<"__builtin_msa_fcun_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fcune_w : ClangBuiltin<"__builtin_msa_fcune_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fcune_d : ClangBuiltin<"__builtin_msa_fcune_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fdiv_w : ClangBuiltin<"__builtin_msa_fdiv_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fdiv_d : ClangBuiltin<"__builtin_msa_fdiv_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fexdo_h : ClangBuiltin<"__builtin_msa_fexdo_h">,
  Intrinsic<[llvm_v8f16_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fexdo_w : ClangBuiltin<"__builtin_msa_fexdo_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fexp2_w : ClangBuiltin<"__builtin_msa_fexp2_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_fexp2_d : ClangBuiltin<"__builtin_msa_fexp2_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_fexupl_w : ClangBuiltin<"__builtin_msa_fexupl_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v8f16_ty], [IntrNoMem]>;
````
- **L953 EN**: Blank line separating nearby declarations or logic blocks.
  **L953 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L954 EN**: Declares TableGen def `int_mips_fcun_w`.
  **L954 CN**: 声明 TableGen def `int_mips_fcun_w`。
- **L955 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L955 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L956 EN**: Declares TableGen def `int_mips_fcun_d`.
  **L956 CN**: 声明 TableGen def `int_mips_fcun_d`。
- **L957 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L957 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L958 EN**: Blank line separating nearby declarations or logic blocks.
  **L958 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L959 EN**: Declares TableGen def `int_mips_fcune_w`.
  **L959 CN**: 声明 TableGen def `int_mips_fcune_w`。
- **L960 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L960 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L961 EN**: Declares TableGen def `int_mips_fcune_d`.
  **L961 CN**: 声明 TableGen def `int_mips_fcune_d`。
- **L962 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L962 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L964 EN**: Declares TableGen def `int_mips_fdiv_w`.
  **L964 CN**: 声明 TableGen def `int_mips_fdiv_w`。
- **L965 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L965 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L966 EN**: Declares TableGen def `int_mips_fdiv_d`.
  **L966 CN**: 声明 TableGen def `int_mips_fdiv_d`。
- **L967 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L967 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L969 EN**: Declares TableGen def `int_mips_fexdo_h`.
  **L969 CN**: 声明 TableGen def `int_mips_fexdo_h`。
- **L970 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8f16_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L970 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8f16_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L971 EN**: Declares TableGen def `int_mips_fexdo_w`.
  **L971 CN**: 声明 TableGen def `int_mips_fexdo_w`。
- **L972 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L972 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L974 EN**: Declares TableGen def `int_mips_fexp2_w`.
  **L974 CN**: 声明 TableGen def `int_mips_fexp2_w`。
- **L975 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L975 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L976 EN**: Declares TableGen def `int_mips_fexp2_d`.
  **L976 CN**: 声明 TableGen def `int_mips_fexp2_d`。
- **L977 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L977 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L978 EN**: Blank line separating nearby declarations or logic blocks.
  **L978 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L979 EN**: Declares TableGen def `int_mips_fexupl_w`.
  **L979 CN**: 声明 TableGen def `int_mips_fexupl_w`。
- **L980 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v8f16_ty], [IntrNoMem]>;`.
  **L980 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v8f16_ty], [IntrNoMem]>;`。

### Lines 981-1008

````tablegen
def int_mips_fexupl_d : ClangBuiltin<"__builtin_msa_fexupl_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v4f32_ty], [IntrNoMem]>;

def int_mips_fexupr_w : ClangBuiltin<"__builtin_msa_fexupr_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v8f16_ty], [IntrNoMem]>;
def int_mips_fexupr_d : ClangBuiltin<"__builtin_msa_fexupr_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v4f32_ty], [IntrNoMem]>;

def int_mips_ffint_s_w : ClangBuiltin<"__builtin_msa_ffint_s_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_ffint_s_d : ClangBuiltin<"__builtin_msa_ffint_s_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_ffint_u_w : ClangBuiltin<"__builtin_msa_ffint_u_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_ffint_u_d : ClangBuiltin<"__builtin_msa_ffint_u_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_ffql_w : ClangBuiltin<"__builtin_msa_ffql_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_ffql_d : ClangBuiltin<"__builtin_msa_ffql_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_ffqr_w : ClangBuiltin<"__builtin_msa_ffqr_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_ffqr_d : ClangBuiltin<"__builtin_msa_ffqr_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v4i32_ty], [IntrNoMem]>;

````
- **L981 EN**: Declares TableGen def `int_mips_fexupl_d`.
  **L981 CN**: 声明 TableGen def `int_mips_fexupl_d`。
- **L982 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L982 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L984 EN**: Declares TableGen def `int_mips_fexupr_w`.
  **L984 CN**: 声明 TableGen def `int_mips_fexupr_w`。
- **L985 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v8f16_ty], [IntrNoMem]>;`.
  **L985 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v8f16_ty], [IntrNoMem]>;`。
- **L986 EN**: Declares TableGen def `int_mips_fexupr_d`.
  **L986 CN**: 声明 TableGen def `int_mips_fexupr_d`。
- **L987 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L987 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L989 EN**: Declares TableGen def `int_mips_ffint_s_w`.
  **L989 CN**: 声明 TableGen def `int_mips_ffint_s_w`。
- **L990 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L990 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L991 EN**: Declares TableGen def `int_mips_ffint_s_d`.
  **L991 CN**: 声明 TableGen def `int_mips_ffint_s_d`。
- **L992 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L992 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L994 EN**: Declares TableGen def `int_mips_ffint_u_w`.
  **L994 CN**: 声明 TableGen def `int_mips_ffint_u_w`。
- **L995 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L995 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L996 EN**: Declares TableGen def `int_mips_ffint_u_d`.
  **L996 CN**: 声明 TableGen def `int_mips_ffint_u_d`。
- **L997 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L997 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L999 EN**: Declares TableGen def `int_mips_ffql_w`.
  **L999 CN**: 声明 TableGen def `int_mips_ffql_w`。
- **L1000 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1000 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1001 EN**: Declares TableGen def `int_mips_ffql_d`.
  **L1001 CN**: 声明 TableGen def `int_mips_ffql_d`。
- **L1002 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1002 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1003 EN**: Blank line separating nearby declarations or logic blocks.
  **L1003 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1004 EN**: Declares TableGen def `int_mips_ffqr_w`.
  **L1004 CN**: 声明 TableGen def `int_mips_ffqr_w`。
- **L1005 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1005 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1006 EN**: Declares TableGen def `int_mips_ffqr_d`.
  **L1006 CN**: 声明 TableGen def `int_mips_ffqr_d`。
- **L1007 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1007 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1008 EN**: Blank line separating nearby declarations or logic blocks.
  **L1008 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1009-1036

````tablegen
def int_mips_fill_b : ClangBuiltin<"__builtin_msa_fill_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_i32_ty], [IntrNoMem]>;
def int_mips_fill_h : ClangBuiltin<"__builtin_msa_fill_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_i32_ty], [IntrNoMem]>;
def int_mips_fill_w : ClangBuiltin<"__builtin_msa_fill_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_i32_ty], [IntrNoMem]>;
def int_mips_fill_d : ClangBuiltin<"__builtin_msa_fill_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_i64_ty], [IntrNoMem]>;

def int_mips_flog2_w : ClangBuiltin<"__builtin_msa_flog2_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_flog2_d : ClangBuiltin<"__builtin_msa_flog2_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fmadd_w : ClangBuiltin<"__builtin_msa_fmadd_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],
            [IntrNoMem]>;
def int_mips_fmadd_d : ClangBuiltin<"__builtin_msa_fmadd_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],
            [IntrNoMem]>;

def int_mips_fmax_w : ClangBuiltin<"__builtin_msa_fmax_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fmax_d : ClangBuiltin<"__builtin_msa_fmax_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fmax_a_w : ClangBuiltin<"__builtin_msa_fmax_a_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
````
- **L1009 EN**: Declares TableGen def `int_mips_fill_b`.
  **L1009 CN**: 声明 TableGen def `int_mips_fill_b`。
- **L1010 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L1010 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L1011 EN**: Declares TableGen def `int_mips_fill_h`.
  **L1011 CN**: 声明 TableGen def `int_mips_fill_h`。
- **L1012 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L1012 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L1013 EN**: Declares TableGen def `int_mips_fill_w`.
  **L1013 CN**: 声明 TableGen def `int_mips_fill_w`。
- **L1014 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L1014 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L1015 EN**: Declares TableGen def `int_mips_fill_d`.
  **L1015 CN**: 声明 TableGen def `int_mips_fill_d`。
- **L1016 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L1016 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L1017 EN**: Blank line separating nearby declarations or logic blocks.
  **L1017 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1018 EN**: Declares TableGen def `int_mips_flog2_w`.
  **L1018 CN**: 声明 TableGen def `int_mips_flog2_w`。
- **L1019 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1019 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1020 EN**: Declares TableGen def `int_mips_flog2_d`.
  **L1020 CN**: 声明 TableGen def `int_mips_flog2_d`。
- **L1021 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1021 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1023 EN**: Declares TableGen def `int_mips_fmadd_w`.
  **L1023 CN**: 声明 TableGen def `int_mips_fmadd_w`。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L1025 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1025 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1026 EN**: Declares TableGen def `int_mips_fmadd_d`.
  **L1026 CN**: 声明 TableGen def `int_mips_fmadd_d`。
- **L1027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L1027 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],`。
- **L1028 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1028 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1029 EN**: Blank line separating nearby declarations or logic blocks.
  **L1029 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1030 EN**: Declares TableGen def `int_mips_fmax_w`.
  **L1030 CN**: 声明 TableGen def `int_mips_fmax_w`。
- **L1031 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1031 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1032 EN**: Declares TableGen def `int_mips_fmax_d`.
  **L1032 CN**: 声明 TableGen def `int_mips_fmax_d`。
- **L1033 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1033 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1035 EN**: Declares TableGen def `int_mips_fmax_a_w`.
  **L1035 CN**: 声明 TableGen def `int_mips_fmax_a_w`。
- **L1036 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1036 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。

### Lines 1037-1064

````tablegen
def int_mips_fmax_a_d : ClangBuiltin<"__builtin_msa_fmax_a_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fmin_w : ClangBuiltin<"__builtin_msa_fmin_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fmin_d : ClangBuiltin<"__builtin_msa_fmin_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fmin_a_w : ClangBuiltin<"__builtin_msa_fmin_a_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fmin_a_d : ClangBuiltin<"__builtin_msa_fmin_a_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fmsub_w : ClangBuiltin<"__builtin_msa_fmsub_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],
            [IntrNoMem]>;
def int_mips_fmsub_d : ClangBuiltin<"__builtin_msa_fmsub_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],
            [IntrNoMem]>;

def int_mips_fmul_w : ClangBuiltin<"__builtin_msa_fmul_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fmul_d : ClangBuiltin<"__builtin_msa_fmul_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_frint_w : ClangBuiltin<"__builtin_msa_frint_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_frint_d : ClangBuiltin<"__builtin_msa_frint_d">,
````
- **L1037 EN**: Declares TableGen def `int_mips_fmax_a_d`.
  **L1037 CN**: 声明 TableGen def `int_mips_fmax_a_d`。
- **L1038 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1038 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1040 EN**: Declares TableGen def `int_mips_fmin_w`.
  **L1040 CN**: 声明 TableGen def `int_mips_fmin_w`。
- **L1041 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1041 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1042 EN**: Declares TableGen def `int_mips_fmin_d`.
  **L1042 CN**: 声明 TableGen def `int_mips_fmin_d`。
- **L1043 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1043 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1045 EN**: Declares TableGen def `int_mips_fmin_a_w`.
  **L1045 CN**: 声明 TableGen def `int_mips_fmin_a_w`。
- **L1046 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1046 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1047 EN**: Declares TableGen def `int_mips_fmin_a_d`.
  **L1047 CN**: 声明 TableGen def `int_mips_fmin_a_d`。
- **L1048 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1048 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1049 EN**: Blank line separating nearby declarations or logic blocks.
  **L1049 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1050 EN**: Declares TableGen def `int_mips_fmsub_w`.
  **L1050 CN**: 声明 TableGen def `int_mips_fmsub_w`。
- **L1051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L1051 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L1052 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1052 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1053 EN**: Declares TableGen def `int_mips_fmsub_d`.
  **L1053 CN**: 声明 TableGen def `int_mips_fmsub_d`。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty, llvm_v2f64_ty],`。
- **L1055 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1055 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1056 EN**: Blank line separating nearby declarations or logic blocks.
  **L1056 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1057 EN**: Declares TableGen def `int_mips_fmul_w`.
  **L1057 CN**: 声明 TableGen def `int_mips_fmul_w`。
- **L1058 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1058 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1059 EN**: Declares TableGen def `int_mips_fmul_d`.
  **L1059 CN**: 声明 TableGen def `int_mips_fmul_d`。
- **L1060 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1060 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1061 EN**: Blank line separating nearby declarations or logic blocks.
  **L1061 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1062 EN**: Declares TableGen def `int_mips_frint_w`.
  **L1062 CN**: 声明 TableGen def `int_mips_frint_w`。
- **L1063 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1063 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1064 EN**: Declares TableGen def `int_mips_frint_d`.
  **L1064 CN**: 声明 TableGen def `int_mips_frint_d`。

### Lines 1065-1092

````tablegen
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_frcp_w : ClangBuiltin<"__builtin_msa_frcp_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_frcp_d : ClangBuiltin<"__builtin_msa_frcp_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_frsqrt_w : ClangBuiltin<"__builtin_msa_frsqrt_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_frsqrt_d : ClangBuiltin<"__builtin_msa_frsqrt_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsaf_w : ClangBuiltin<"__builtin_msa_fsaf_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsaf_d : ClangBuiltin<"__builtin_msa_fsaf_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fseq_w : ClangBuiltin<"__builtin_msa_fseq_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fseq_d : ClangBuiltin<"__builtin_msa_fseq_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsle_w : ClangBuiltin<"__builtin_msa_fsle_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsle_d : ClangBuiltin<"__builtin_msa_fsle_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fslt_w : ClangBuiltin<"__builtin_msa_fslt_w">,
````
- **L1065 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1065 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1067 EN**: Declares TableGen def `int_mips_frcp_w`.
  **L1067 CN**: 声明 TableGen def `int_mips_frcp_w`。
- **L1068 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1068 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1069 EN**: Declares TableGen def `int_mips_frcp_d`.
  **L1069 CN**: 声明 TableGen def `int_mips_frcp_d`。
- **L1070 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1070 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1071 EN**: Blank line separating nearby declarations or logic blocks.
  **L1071 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1072 EN**: Declares TableGen def `int_mips_frsqrt_w`.
  **L1072 CN**: 声明 TableGen def `int_mips_frsqrt_w`。
- **L1073 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1073 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1074 EN**: Declares TableGen def `int_mips_frsqrt_d`.
  **L1074 CN**: 声明 TableGen def `int_mips_frsqrt_d`。
- **L1075 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1075 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1077 EN**: Declares TableGen def `int_mips_fsaf_w`.
  **L1077 CN**: 声明 TableGen def `int_mips_fsaf_w`。
- **L1078 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1078 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1079 EN**: Declares TableGen def `int_mips_fsaf_d`.
  **L1079 CN**: 声明 TableGen def `int_mips_fsaf_d`。
- **L1080 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1080 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1081 EN**: Blank line separating nearby declarations or logic blocks.
  **L1081 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1082 EN**: Declares TableGen def `int_mips_fseq_w`.
  **L1082 CN**: 声明 TableGen def `int_mips_fseq_w`。
- **L1083 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1083 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1084 EN**: Declares TableGen def `int_mips_fseq_d`.
  **L1084 CN**: 声明 TableGen def `int_mips_fseq_d`。
- **L1085 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1085 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1086 EN**: Blank line separating nearby declarations or logic blocks.
  **L1086 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1087 EN**: Declares TableGen def `int_mips_fsle_w`.
  **L1087 CN**: 声明 TableGen def `int_mips_fsle_w`。
- **L1088 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1088 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1089 EN**: Declares TableGen def `int_mips_fsle_d`.
  **L1089 CN**: 声明 TableGen def `int_mips_fsle_d`。
- **L1090 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1090 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1091 EN**: Blank line separating nearby declarations or logic blocks.
  **L1091 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1092 EN**: Declares TableGen def `int_mips_fslt_w`.
  **L1092 CN**: 声明 TableGen def `int_mips_fslt_w`。

### Lines 1093-1120

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fslt_d : ClangBuiltin<"__builtin_msa_fslt_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsne_w : ClangBuiltin<"__builtin_msa_fsne_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsne_d : ClangBuiltin<"__builtin_msa_fsne_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsor_w : ClangBuiltin<"__builtin_msa_fsor_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsor_d : ClangBuiltin<"__builtin_msa_fsor_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsqrt_w : ClangBuiltin<"__builtin_msa_fsqrt_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsqrt_d : ClangBuiltin<"__builtin_msa_fsqrt_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsub_w : ClangBuiltin<"__builtin_msa_fsub_w">,
  Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsub_d : ClangBuiltin<"__builtin_msa_fsub_d">,
  Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsueq_w : ClangBuiltin<"__builtin_msa_fsueq_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsueq_d : ClangBuiltin<"__builtin_msa_fsueq_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
````
- **L1093 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1093 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1094 EN**: Declares TableGen def `int_mips_fslt_d`.
  **L1094 CN**: 声明 TableGen def `int_mips_fslt_d`。
- **L1095 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1095 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1096 EN**: Blank line separating nearby declarations or logic blocks.
  **L1096 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1097 EN**: Declares TableGen def `int_mips_fsne_w`.
  **L1097 CN**: 声明 TableGen def `int_mips_fsne_w`。
- **L1098 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1098 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1099 EN**: Declares TableGen def `int_mips_fsne_d`.
  **L1099 CN**: 声明 TableGen def `int_mips_fsne_d`。
- **L1100 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1100 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1101 EN**: Blank line separating nearby declarations or logic blocks.
  **L1101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1102 EN**: Declares TableGen def `int_mips_fsor_w`.
  **L1102 CN**: 声明 TableGen def `int_mips_fsor_w`。
- **L1103 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1103 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1104 EN**: Declares TableGen def `int_mips_fsor_d`.
  **L1104 CN**: 声明 TableGen def `int_mips_fsor_d`。
- **L1105 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1105 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1106 EN**: Blank line separating nearby declarations or logic blocks.
  **L1106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1107 EN**: Declares TableGen def `int_mips_fsqrt_w`.
  **L1107 CN**: 声明 TableGen def `int_mips_fsqrt_w`。
- **L1108 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1108 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1109 EN**: Declares TableGen def `int_mips_fsqrt_d`.
  **L1109 CN**: 声明 TableGen def `int_mips_fsqrt_d`。
- **L1110 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1110 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1111 EN**: Blank line separating nearby declarations or logic blocks.
  **L1111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1112 EN**: Declares TableGen def `int_mips_fsub_w`.
  **L1112 CN**: 声明 TableGen def `int_mips_fsub_w`。
- **L1113 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1113 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1114 EN**: Declares TableGen def `int_mips_fsub_d`.
  **L1114 CN**: 声明 TableGen def `int_mips_fsub_d`。
- **L1115 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1115 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1116 EN**: Blank line separating nearby declarations or logic blocks.
  **L1116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1117 EN**: Declares TableGen def `int_mips_fsueq_w`.
  **L1117 CN**: 声明 TableGen def `int_mips_fsueq_w`。
- **L1118 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1118 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1119 EN**: Declares TableGen def `int_mips_fsueq_d`.
  **L1119 CN**: 声明 TableGen def `int_mips_fsueq_d`。
- **L1120 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1120 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。

### Lines 1121-1148

````tablegen

def int_mips_fsule_w : ClangBuiltin<"__builtin_msa_fsule_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsule_d : ClangBuiltin<"__builtin_msa_fsule_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsult_w : ClangBuiltin<"__builtin_msa_fsult_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsult_d : ClangBuiltin<"__builtin_msa_fsult_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsun_w : ClangBuiltin<"__builtin_msa_fsun_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsun_d : ClangBuiltin<"__builtin_msa_fsun_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_fsune_w : ClangBuiltin<"__builtin_msa_fsune_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_fsune_d : ClangBuiltin<"__builtin_msa_fsune_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_ftint_s_w : ClangBuiltin<"__builtin_msa_ftint_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_ftint_s_d : ClangBuiltin<"__builtin_msa_ftint_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_ftint_u_w : ClangBuiltin<"__builtin_msa_ftint_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
````
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1122 EN**: Declares TableGen def `int_mips_fsule_w`.
  **L1122 CN**: 声明 TableGen def `int_mips_fsule_w`。
- **L1123 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1123 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1124 EN**: Declares TableGen def `int_mips_fsule_d`.
  **L1124 CN**: 声明 TableGen def `int_mips_fsule_d`。
- **L1125 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1125 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1126 EN**: Blank line separating nearby declarations or logic blocks.
  **L1126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1127 EN**: Declares TableGen def `int_mips_fsult_w`.
  **L1127 CN**: 声明 TableGen def `int_mips_fsult_w`。
- **L1128 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1128 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1129 EN**: Declares TableGen def `int_mips_fsult_d`.
  **L1129 CN**: 声明 TableGen def `int_mips_fsult_d`。
- **L1130 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1130 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1131 EN**: Blank line separating nearby declarations or logic blocks.
  **L1131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1132 EN**: Declares TableGen def `int_mips_fsun_w`.
  **L1132 CN**: 声明 TableGen def `int_mips_fsun_w`。
- **L1133 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1133 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1134 EN**: Declares TableGen def `int_mips_fsun_d`.
  **L1134 CN**: 声明 TableGen def `int_mips_fsun_d`。
- **L1135 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1135 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1137 EN**: Declares TableGen def `int_mips_fsune_w`.
  **L1137 CN**: 声明 TableGen def `int_mips_fsune_w`。
- **L1138 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1138 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1139 EN**: Declares TableGen def `int_mips_fsune_d`.
  **L1139 CN**: 声明 TableGen def `int_mips_fsune_d`。
- **L1140 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1140 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1141 EN**: Blank line separating nearby declarations or logic blocks.
  **L1141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1142 EN**: Declares TableGen def `int_mips_ftint_s_w`.
  **L1142 CN**: 声明 TableGen def `int_mips_ftint_s_w`。
- **L1143 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1143 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1144 EN**: Declares TableGen def `int_mips_ftint_s_d`.
  **L1144 CN**: 声明 TableGen def `int_mips_ftint_s_d`。
- **L1145 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1145 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1146 EN**: Blank line separating nearby declarations or logic blocks.
  **L1146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1147 EN**: Declares TableGen def `int_mips_ftint_u_w`.
  **L1147 CN**: 声明 TableGen def `int_mips_ftint_u_w`。
- **L1148 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1148 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。

### Lines 1149-1176

````tablegen
def int_mips_ftint_u_d : ClangBuiltin<"__builtin_msa_ftint_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_ftq_h : ClangBuiltin<"__builtin_msa_ftq_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_ftq_w : ClangBuiltin<"__builtin_msa_ftq_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_ftrunc_s_w : ClangBuiltin<"__builtin_msa_ftrunc_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_ftrunc_s_d : ClangBuiltin<"__builtin_msa_ftrunc_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_ftrunc_u_w : ClangBuiltin<"__builtin_msa_ftrunc_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_mips_ftrunc_u_d : ClangBuiltin<"__builtin_msa_ftrunc_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

def int_mips_hadd_s_h : ClangBuiltin<"__builtin_msa_hadd_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_hadd_s_w : ClangBuiltin<"__builtin_msa_hadd_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_hadd_s_d : ClangBuiltin<"__builtin_msa_hadd_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_hadd_u_h : ClangBuiltin<"__builtin_msa_hadd_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_hadd_u_w : ClangBuiltin<"__builtin_msa_hadd_u_w">,
````
- **L1149 EN**: Declares TableGen def `int_mips_ftint_u_d`.
  **L1149 CN**: 声明 TableGen def `int_mips_ftint_u_d`。
- **L1150 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1150 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1151 EN**: Blank line separating nearby declarations or logic blocks.
  **L1151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1152 EN**: Declares TableGen def `int_mips_ftq_h`.
  **L1152 CN**: 声明 TableGen def `int_mips_ftq_h`。
- **L1153 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1153 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1154 EN**: Declares TableGen def `int_mips_ftq_w`.
  **L1154 CN**: 声明 TableGen def `int_mips_ftq_w`。
- **L1155 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1155 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1156 EN**: Blank line separating nearby declarations or logic blocks.
  **L1156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1157 EN**: Declares TableGen def `int_mips_ftrunc_s_w`.
  **L1157 CN**: 声明 TableGen def `int_mips_ftrunc_s_w`。
- **L1158 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1158 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1159 EN**: Declares TableGen def `int_mips_ftrunc_s_d`.
  **L1159 CN**: 声明 TableGen def `int_mips_ftrunc_s_d`。
- **L1160 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1160 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1161 EN**: Blank line separating nearby declarations or logic blocks.
  **L1161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1162 EN**: Declares TableGen def `int_mips_ftrunc_u_w`.
  **L1162 CN**: 声明 TableGen def `int_mips_ftrunc_u_w`。
- **L1163 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1163 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1164 EN**: Declares TableGen def `int_mips_ftrunc_u_d`.
  **L1164 CN**: 声明 TableGen def `int_mips_ftrunc_u_d`。
- **L1165 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1165 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1167 EN**: Declares TableGen def `int_mips_hadd_s_h`.
  **L1167 CN**: 声明 TableGen def `int_mips_hadd_s_h`。
- **L1168 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1168 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1169 EN**: Declares TableGen def `int_mips_hadd_s_w`.
  **L1169 CN**: 声明 TableGen def `int_mips_hadd_s_w`。
- **L1170 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1170 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1171 EN**: Declares TableGen def `int_mips_hadd_s_d`.
  **L1171 CN**: 声明 TableGen def `int_mips_hadd_s_d`。
- **L1172 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1172 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1174 EN**: Declares TableGen def `int_mips_hadd_u_h`.
  **L1174 CN**: 声明 TableGen def `int_mips_hadd_u_h`。
- **L1175 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1175 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1176 EN**: Declares TableGen def `int_mips_hadd_u_w`.
  **L1176 CN**: 声明 TableGen def `int_mips_hadd_u_w`。

### Lines 1177-1204

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_hadd_u_d : ClangBuiltin<"__builtin_msa_hadd_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_hsub_s_h : ClangBuiltin<"__builtin_msa_hsub_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_hsub_s_w : ClangBuiltin<"__builtin_msa_hsub_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_hsub_s_d : ClangBuiltin<"__builtin_msa_hsub_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_hsub_u_h : ClangBuiltin<"__builtin_msa_hsub_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_hsub_u_w : ClangBuiltin<"__builtin_msa_hsub_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_hsub_u_d : ClangBuiltin<"__builtin_msa_hsub_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_ilvev_b : ClangBuiltin<"__builtin_msa_ilvev_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_ilvev_h : ClangBuiltin<"__builtin_msa_ilvev_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_ilvev_w : ClangBuiltin<"__builtin_msa_ilvev_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_ilvev_d : ClangBuiltin<"__builtin_msa_ilvev_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_ilvl_b : ClangBuiltin<"__builtin_msa_ilvl_b">,
````
- **L1177 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1177 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1178 EN**: Declares TableGen def `int_mips_hadd_u_d`.
  **L1178 CN**: 声明 TableGen def `int_mips_hadd_u_d`。
- **L1179 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1179 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1180 EN**: Blank line separating nearby declarations or logic blocks.
  **L1180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1181 EN**: Declares TableGen def `int_mips_hsub_s_h`.
  **L1181 CN**: 声明 TableGen def `int_mips_hsub_s_h`。
- **L1182 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1182 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1183 EN**: Declares TableGen def `int_mips_hsub_s_w`.
  **L1183 CN**: 声明 TableGen def `int_mips_hsub_s_w`。
- **L1184 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1184 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1185 EN**: Declares TableGen def `int_mips_hsub_s_d`.
  **L1185 CN**: 声明 TableGen def `int_mips_hsub_s_d`。
- **L1186 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1186 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1187 EN**: Blank line separating nearby declarations or logic blocks.
  **L1187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1188 EN**: Declares TableGen def `int_mips_hsub_u_h`.
  **L1188 CN**: 声明 TableGen def `int_mips_hsub_u_h`。
- **L1189 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1189 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1190 EN**: Declares TableGen def `int_mips_hsub_u_w`.
  **L1190 CN**: 声明 TableGen def `int_mips_hsub_u_w`。
- **L1191 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1191 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1192 EN**: Declares TableGen def `int_mips_hsub_u_d`.
  **L1192 CN**: 声明 TableGen def `int_mips_hsub_u_d`。
- **L1193 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1193 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1194 EN**: Blank line separating nearby declarations or logic blocks.
  **L1194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1195 EN**: Declares TableGen def `int_mips_ilvev_b`.
  **L1195 CN**: 声明 TableGen def `int_mips_ilvev_b`。
- **L1196 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1196 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1197 EN**: Declares TableGen def `int_mips_ilvev_h`.
  **L1197 CN**: 声明 TableGen def `int_mips_ilvev_h`。
- **L1198 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1198 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1199 EN**: Declares TableGen def `int_mips_ilvev_w`.
  **L1199 CN**: 声明 TableGen def `int_mips_ilvev_w`。
- **L1200 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1200 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1201 EN**: Declares TableGen def `int_mips_ilvev_d`.
  **L1201 CN**: 声明 TableGen def `int_mips_ilvev_d`。
- **L1202 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1202 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1204 EN**: Declares TableGen def `int_mips_ilvl_b`.
  **L1204 CN**: 声明 TableGen def `int_mips_ilvl_b`。

### Lines 1205-1232

````tablegen
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_ilvl_h : ClangBuiltin<"__builtin_msa_ilvl_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_ilvl_w : ClangBuiltin<"__builtin_msa_ilvl_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_ilvl_d : ClangBuiltin<"__builtin_msa_ilvl_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_ilvod_b : ClangBuiltin<"__builtin_msa_ilvod_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_ilvod_h : ClangBuiltin<"__builtin_msa_ilvod_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_ilvod_w : ClangBuiltin<"__builtin_msa_ilvod_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_ilvod_d : ClangBuiltin<"__builtin_msa_ilvod_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_ilvr_b : ClangBuiltin<"__builtin_msa_ilvr_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_ilvr_h : ClangBuiltin<"__builtin_msa_ilvr_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_ilvr_w : ClangBuiltin<"__builtin_msa_ilvr_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_ilvr_d : ClangBuiltin<"__builtin_msa_ilvr_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_insert_b : ClangBuiltin<"__builtin_msa_insert_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],
````
- **L1205 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1205 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1206 EN**: Declares TableGen def `int_mips_ilvl_h`.
  **L1206 CN**: 声明 TableGen def `int_mips_ilvl_h`。
- **L1207 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1207 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1208 EN**: Declares TableGen def `int_mips_ilvl_w`.
  **L1208 CN**: 声明 TableGen def `int_mips_ilvl_w`。
- **L1209 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1209 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1210 EN**: Declares TableGen def `int_mips_ilvl_d`.
  **L1210 CN**: 声明 TableGen def `int_mips_ilvl_d`。
- **L1211 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1211 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1212 EN**: Blank line separating nearby declarations or logic blocks.
  **L1212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1213 EN**: Declares TableGen def `int_mips_ilvod_b`.
  **L1213 CN**: 声明 TableGen def `int_mips_ilvod_b`。
- **L1214 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1214 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1215 EN**: Declares TableGen def `int_mips_ilvod_h`.
  **L1215 CN**: 声明 TableGen def `int_mips_ilvod_h`。
- **L1216 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1216 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1217 EN**: Declares TableGen def `int_mips_ilvod_w`.
  **L1217 CN**: 声明 TableGen def `int_mips_ilvod_w`。
- **L1218 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1218 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1219 EN**: Declares TableGen def `int_mips_ilvod_d`.
  **L1219 CN**: 声明 TableGen def `int_mips_ilvod_d`。
- **L1220 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1220 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1221 EN**: Blank line separating nearby declarations or logic blocks.
  **L1221 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1222 EN**: Declares TableGen def `int_mips_ilvr_b`.
  **L1222 CN**: 声明 TableGen def `int_mips_ilvr_b`。
- **L1223 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1223 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1224 EN**: Declares TableGen def `int_mips_ilvr_h`.
  **L1224 CN**: 声明 TableGen def `int_mips_ilvr_h`。
- **L1225 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1225 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1226 EN**: Declares TableGen def `int_mips_ilvr_w`.
  **L1226 CN**: 声明 TableGen def `int_mips_ilvr_w`。
- **L1227 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1227 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1228 EN**: Declares TableGen def `int_mips_ilvr_d`.
  **L1228 CN**: 声明 TableGen def `int_mips_ilvr_d`。
- **L1229 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1229 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1230 EN**: Blank line separating nearby declarations or logic blocks.
  **L1230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1231 EN**: Declares TableGen def `int_mips_insert_b`.
  **L1231 CN**: 声明 TableGen def `int_mips_insert_b`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`。

### Lines 1233-1260

````tablegen
  [IntrNoMem]>;
def int_mips_insert_h : ClangBuiltin<"__builtin_msa_insert_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],
  [IntrNoMem]>;
def int_mips_insert_w : ClangBuiltin<"__builtin_msa_insert_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],
  [IntrNoMem]>;
def int_mips_insert_d : ClangBuiltin<"__builtin_msa_insert_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty, llvm_i64_ty],
  [IntrNoMem]>;

def int_mips_insve_b : ClangBuiltin<"__builtin_msa_insve_b">,
  Intrinsic<[llvm_v16i8_ty],
            [llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_insve_h : ClangBuiltin<"__builtin_msa_insve_h">,
  Intrinsic<[llvm_v8i16_ty],
            [llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_insve_w : ClangBuiltin<"__builtin_msa_insve_w">,
  Intrinsic<[llvm_v4i32_ty],
            [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_insve_d : ClangBuiltin<"__builtin_msa_insve_d">,
  Intrinsic<[llvm_v2i64_ty],
            [llvm_v2i64_ty, llvm_i32_ty, llvm_v2i64_ty],
            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

````
- **L1233 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1233 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1234 EN**: Declares TableGen def `int_mips_insert_h`.
  **L1234 CN**: 声明 TableGen def `int_mips_insert_h`。
- **L1235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1235 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1236 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1236 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1237 EN**: Declares TableGen def `int_mips_insert_w`.
  **L1237 CN**: 声明 TableGen def `int_mips_insert_w`。
- **L1238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1238 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1239 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1239 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1240 EN**: Declares TableGen def `int_mips_insert_d`.
  **L1240 CN**: 声明 TableGen def `int_mips_insert_d`。
- **L1241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty, llvm_i64_ty],`.
  **L1241 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty, llvm_i64_ty],`。
- **L1242 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1242 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1243 EN**: Blank line separating nearby declarations or logic blocks.
  **L1243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1244 EN**: Declares TableGen def `int_mips_insve_b`.
  **L1244 CN**: 声明 TableGen def `int_mips_insve_b`。
- **L1245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty],`.
  **L1245 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty],`。
- **L1246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],`.
  **L1246 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],`。
- **L1247 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1247 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1248 EN**: Declares TableGen def `int_mips_insve_h`.
  **L1248 CN**: 声明 TableGen def `int_mips_insve_h`。
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty],`.
  **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty],`。
- **L1250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],`.
  **L1250 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],`。
- **L1251 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1251 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1252 EN**: Declares TableGen def `int_mips_insve_w`.
  **L1252 CN**: 声明 TableGen def `int_mips_insve_w`。
- **L1253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty],`.
  **L1253 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty],`。
- **L1254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`.
  **L1254 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`。
- **L1255 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1255 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1256 EN**: Declares TableGen def `int_mips_insve_d`.
  **L1256 CN**: 声明 TableGen def `int_mips_insve_d`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty],`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty],`。
- **L1258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_i32_ty, llvm_v2i64_ty],`.
  **L1258 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_i32_ty, llvm_v2i64_ty],`。
- **L1259 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1259 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1261-1288

````tablegen
def int_mips_ld_b : ClangBuiltin<"__builtin_msa_ld_b">,
  DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],
  [IntrReadMem, IntrArgMemOnly]>;
def int_mips_ld_h : ClangBuiltin<"__builtin_msa_ld_h">,
  DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_ptr_ty, llvm_i32_ty],
  [IntrReadMem, IntrArgMemOnly]>;
def int_mips_ld_w : ClangBuiltin<"__builtin_msa_ld_w">,
  DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],
  [IntrReadMem, IntrArgMemOnly]>;
def int_mips_ld_d : ClangBuiltin<"__builtin_msa_ld_d">,
  DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],
  [IntrReadMem, IntrArgMemOnly]>;

def int_mips_ldr_d : ClangBuiltin<"__builtin_msa_ldr_d">,
  DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],
  [IntrReadMem, IntrArgMemOnly]>;
def int_mips_ldr_w : ClangBuiltin<"__builtin_msa_ldr_w">,
  DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],
  [IntrReadMem, IntrArgMemOnly]>;

def int_mips_ldi_b : ClangBuiltin<"__builtin_msa_ldi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_mips_ldi_h : ClangBuiltin<"__builtin_msa_ldi_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_mips_ldi_w : ClangBuiltin<"__builtin_msa_ldi_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;
def int_mips_ldi_d : ClangBuiltin<"__builtin_msa_ldi_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;
````
- **L1261 EN**: Declares TableGen def `int_mips_ld_b`.
  **L1261 CN**: 声明 TableGen def `int_mips_ld_b`。
- **L1262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1262 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1263 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1263 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1264 EN**: Declares TableGen def `int_mips_ld_h`.
  **L1264 CN**: 声明 TableGen def `int_mips_ld_h`。
- **L1265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1265 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1266 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1266 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1267 EN**: Declares TableGen def `int_mips_ld_w`.
  **L1267 CN**: 声明 TableGen def `int_mips_ld_w`。
- **L1268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1268 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1269 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1269 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1270 EN**: Declares TableGen def `int_mips_ld_d`.
  **L1270 CN**: 声明 TableGen def `int_mips_ld_d`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1272 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1272 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1273 EN**: Blank line separating nearby declarations or logic blocks.
  **L1273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1274 EN**: Declares TableGen def `int_mips_ldr_d`.
  **L1274 CN**: 声明 TableGen def `int_mips_ldr_d`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1276 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1276 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1277 EN**: Declares TableGen def `int_mips_ldr_w`.
  **L1277 CN**: 声明 TableGen def `int_mips_ldr_w`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1279 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1279 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1280 EN**: Blank line separating nearby declarations or logic blocks.
  **L1280 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1281 EN**: Declares TableGen def `int_mips_ldi_b`.
  **L1281 CN**: 声明 TableGen def `int_mips_ldi_b`。
- **L1282 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L1282 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L1283 EN**: Declares TableGen def `int_mips_ldi_h`.
  **L1283 CN**: 声明 TableGen def `int_mips_ldi_h`。
- **L1284 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L1284 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L1285 EN**: Declares TableGen def `int_mips_ldi_w`.
  **L1285 CN**: 声明 TableGen def `int_mips_ldi_w`。
- **L1286 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L1286 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L1287 EN**: Declares TableGen def `int_mips_ldi_d`.
  **L1287 CN**: 声明 TableGen def `int_mips_ldi_d`。
- **L1288 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L1288 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`。

### Lines 1289-1316

````tablegen

// This instruction is part of the MSA spec but it does not share the
// __builtin_msa prefix because it operates on the GPR registers.
def int_mips_lsa : ClangBuiltin<"__builtin_mips_lsa">,
  Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
            [IntrNoMem]>;

def int_mips_madd_q_h : ClangBuiltin<"__builtin_msa_madd_q_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_madd_q_w : ClangBuiltin<"__builtin_msa_madd_q_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;

def int_mips_maddr_q_h : ClangBuiltin<"__builtin_msa_maddr_q_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_maddr_q_w : ClangBuiltin<"__builtin_msa_maddr_q_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;

def int_mips_maddv_b : ClangBuiltin<"__builtin_msa_maddv_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
  [IntrNoMem]>;
def int_mips_maddv_h : ClangBuiltin<"__builtin_msa_maddv_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_maddv_w : ClangBuiltin<"__builtin_msa_maddv_w">,
````
- **L1289 EN**: Blank line separating nearby declarations or logic blocks.
  **L1289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1290 EN**: Comment explains nearby logic, invariants, or intent: `This instruction is part of the MSA spec but it does not share the`.
  **L1290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This instruction is part of the MSA spec but it does not share the`。
- **L1291 EN**: Comment explains nearby logic, invariants, or intent: `__builtin_msa prefix because it operates on the GPR registers.`.
  **L1291 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`__builtin_msa prefix because it operates on the GPR registers.`。
- **L1292 EN**: Declares TableGen def `int_mips_lsa`.
  **L1292 CN**: 声明 TableGen def `int_mips_lsa`。
- **L1293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1293 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1294 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1294 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1295 EN**: Blank line separating nearby declarations or logic blocks.
  **L1295 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1296 EN**: Declares TableGen def `int_mips_madd_q_h`.
  **L1296 CN**: 声明 TableGen def `int_mips_madd_q_h`。
- **L1297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1297 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1298 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1298 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1299 EN**: Declares TableGen def `int_mips_madd_q_w`.
  **L1299 CN**: 声明 TableGen def `int_mips_madd_q_w`。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1301 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1301 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1303 EN**: Declares TableGen def `int_mips_maddr_q_h`.
  **L1303 CN**: 声明 TableGen def `int_mips_maddr_q_h`。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1305 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1305 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1306 EN**: Declares TableGen def `int_mips_maddr_q_w`.
  **L1306 CN**: 声明 TableGen def `int_mips_maddr_q_w`。
- **L1307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1307 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1308 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1308 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1309 EN**: Blank line separating nearby declarations or logic blocks.
  **L1309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1310 EN**: Declares TableGen def `int_mips_maddv_b`.
  **L1310 CN**: 声明 TableGen def `int_mips_maddv_b`。
- **L1311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1311 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1312 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1312 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1313 EN**: Declares TableGen def `int_mips_maddv_h`.
  **L1313 CN**: 声明 TableGen def `int_mips_maddv_h`。
- **L1314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1314 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1315 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1315 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1316 EN**: Declares TableGen def `int_mips_maddv_w`.
  **L1316 CN**: 声明 TableGen def `int_mips_maddv_w`。

### Lines 1317-1344

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;
def int_mips_maddv_d : ClangBuiltin<"__builtin_msa_maddv_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
  [IntrNoMem]>;

def int_mips_max_a_b : ClangBuiltin<"__builtin_msa_max_a_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_max_a_h : ClangBuiltin<"__builtin_msa_max_a_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_max_a_w : ClangBuiltin<"__builtin_msa_max_a_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_max_a_d : ClangBuiltin<"__builtin_msa_max_a_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_max_s_b : ClangBuiltin<"__builtin_msa_max_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_max_s_h : ClangBuiltin<"__builtin_msa_max_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_max_s_w : ClangBuiltin<"__builtin_msa_max_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_max_s_d : ClangBuiltin<"__builtin_msa_max_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_max_u_b : ClangBuiltin<"__builtin_msa_max_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_max_u_h : ClangBuiltin<"__builtin_msa_max_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
````
- **L1317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1317 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1318 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1318 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1319 EN**: Declares TableGen def `int_mips_maddv_d`.
  **L1319 CN**: 声明 TableGen def `int_mips_maddv_d`。
- **L1320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1320 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1321 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1321 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1322 EN**: Blank line separating nearby declarations or logic blocks.
  **L1322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1323 EN**: Declares TableGen def `int_mips_max_a_b`.
  **L1323 CN**: 声明 TableGen def `int_mips_max_a_b`。
- **L1324 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1324 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1325 EN**: Declares TableGen def `int_mips_max_a_h`.
  **L1325 CN**: 声明 TableGen def `int_mips_max_a_h`。
- **L1326 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1326 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1327 EN**: Declares TableGen def `int_mips_max_a_w`.
  **L1327 CN**: 声明 TableGen def `int_mips_max_a_w`。
- **L1328 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1328 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1329 EN**: Declares TableGen def `int_mips_max_a_d`.
  **L1329 CN**: 声明 TableGen def `int_mips_max_a_d`。
- **L1330 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1330 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1331 EN**: Blank line separating nearby declarations or logic blocks.
  **L1331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1332 EN**: Declares TableGen def `int_mips_max_s_b`.
  **L1332 CN**: 声明 TableGen def `int_mips_max_s_b`。
- **L1333 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1333 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1334 EN**: Declares TableGen def `int_mips_max_s_h`.
  **L1334 CN**: 声明 TableGen def `int_mips_max_s_h`。
- **L1335 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1335 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1336 EN**: Declares TableGen def `int_mips_max_s_w`.
  **L1336 CN**: 声明 TableGen def `int_mips_max_s_w`。
- **L1337 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1337 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1338 EN**: Declares TableGen def `int_mips_max_s_d`.
  **L1338 CN**: 声明 TableGen def `int_mips_max_s_d`。
- **L1339 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1339 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1340 EN**: Blank line separating nearby declarations or logic blocks.
  **L1340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1341 EN**: Declares TableGen def `int_mips_max_u_b`.
  **L1341 CN**: 声明 TableGen def `int_mips_max_u_b`。
- **L1342 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1342 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1343 EN**: Declares TableGen def `int_mips_max_u_h`.
  **L1343 CN**: 声明 TableGen def `int_mips_max_u_h`。
- **L1344 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1344 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。

### Lines 1345-1372

````tablegen
def int_mips_max_u_w : ClangBuiltin<"__builtin_msa_max_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_max_u_d : ClangBuiltin<"__builtin_msa_max_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_maxi_s_b : ClangBuiltin<"__builtin_msa_maxi_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_maxi_s_h : ClangBuiltin<"__builtin_msa_maxi_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_maxi_s_w : ClangBuiltin<"__builtin_msa_maxi_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_maxi_s_d : ClangBuiltin<"__builtin_msa_maxi_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_maxi_u_b : ClangBuiltin<"__builtin_msa_maxi_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_maxi_u_h : ClangBuiltin<"__builtin_msa_maxi_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_maxi_u_w : ClangBuiltin<"__builtin_msa_maxi_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_maxi_u_d : ClangBuiltin<"__builtin_msa_maxi_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_min_a_b : ClangBuiltin<"__builtin_msa_min_a_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_min_a_h : ClangBuiltin<"__builtin_msa_min_a_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_min_a_w : ClangBuiltin<"__builtin_msa_min_a_w">,
````
- **L1345 EN**: Declares TableGen def `int_mips_max_u_w`.
  **L1345 CN**: 声明 TableGen def `int_mips_max_u_w`。
- **L1346 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1346 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1347 EN**: Declares TableGen def `int_mips_max_u_d`.
  **L1347 CN**: 声明 TableGen def `int_mips_max_u_d`。
- **L1348 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1348 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1349 EN**: Blank line separating nearby declarations or logic blocks.
  **L1349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1350 EN**: Declares TableGen def `int_mips_maxi_s_b`.
  **L1350 CN**: 声明 TableGen def `int_mips_maxi_s_b`。
- **L1351 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1351 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1352 EN**: Declares TableGen def `int_mips_maxi_s_h`.
  **L1352 CN**: 声明 TableGen def `int_mips_maxi_s_h`。
- **L1353 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1353 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1354 EN**: Declares TableGen def `int_mips_maxi_s_w`.
  **L1354 CN**: 声明 TableGen def `int_mips_maxi_s_w`。
- **L1355 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1355 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1356 EN**: Declares TableGen def `int_mips_maxi_s_d`.
  **L1356 CN**: 声明 TableGen def `int_mips_maxi_s_d`。
- **L1357 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1357 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1358 EN**: Blank line separating nearby declarations or logic blocks.
  **L1358 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1359 EN**: Declares TableGen def `int_mips_maxi_u_b`.
  **L1359 CN**: 声明 TableGen def `int_mips_maxi_u_b`。
- **L1360 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1360 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1361 EN**: Declares TableGen def `int_mips_maxi_u_h`.
  **L1361 CN**: 声明 TableGen def `int_mips_maxi_u_h`。
- **L1362 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1362 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1363 EN**: Declares TableGen def `int_mips_maxi_u_w`.
  **L1363 CN**: 声明 TableGen def `int_mips_maxi_u_w`。
- **L1364 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1364 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1365 EN**: Declares TableGen def `int_mips_maxi_u_d`.
  **L1365 CN**: 声明 TableGen def `int_mips_maxi_u_d`。
- **L1366 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1366 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1367 EN**: Blank line separating nearby declarations or logic blocks.
  **L1367 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1368 EN**: Declares TableGen def `int_mips_min_a_b`.
  **L1368 CN**: 声明 TableGen def `int_mips_min_a_b`。
- **L1369 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1369 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1370 EN**: Declares TableGen def `int_mips_min_a_h`.
  **L1370 CN**: 声明 TableGen def `int_mips_min_a_h`。
- **L1371 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1371 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1372 EN**: Declares TableGen def `int_mips_min_a_w`.
  **L1372 CN**: 声明 TableGen def `int_mips_min_a_w`。

### Lines 1373-1400

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_min_a_d : ClangBuiltin<"__builtin_msa_min_a_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_min_s_b : ClangBuiltin<"__builtin_msa_min_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_min_s_h : ClangBuiltin<"__builtin_msa_min_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_min_s_w : ClangBuiltin<"__builtin_msa_min_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_min_s_d : ClangBuiltin<"__builtin_msa_min_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_min_u_b : ClangBuiltin<"__builtin_msa_min_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_min_u_h : ClangBuiltin<"__builtin_msa_min_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_min_u_w : ClangBuiltin<"__builtin_msa_min_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_min_u_d : ClangBuiltin<"__builtin_msa_min_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_mini_s_b : ClangBuiltin<"__builtin_msa_mini_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_mini_s_h : ClangBuiltin<"__builtin_msa_mini_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_mini_s_w : ClangBuiltin<"__builtin_msa_mini_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L1373 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1373 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1374 EN**: Declares TableGen def `int_mips_min_a_d`.
  **L1374 CN**: 声明 TableGen def `int_mips_min_a_d`。
- **L1375 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1375 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1376 EN**: Blank line separating nearby declarations or logic blocks.
  **L1376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1377 EN**: Declares TableGen def `int_mips_min_s_b`.
  **L1377 CN**: 声明 TableGen def `int_mips_min_s_b`。
- **L1378 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1378 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1379 EN**: Declares TableGen def `int_mips_min_s_h`.
  **L1379 CN**: 声明 TableGen def `int_mips_min_s_h`。
- **L1380 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1380 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1381 EN**: Declares TableGen def `int_mips_min_s_w`.
  **L1381 CN**: 声明 TableGen def `int_mips_min_s_w`。
- **L1382 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1382 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1383 EN**: Declares TableGen def `int_mips_min_s_d`.
  **L1383 CN**: 声明 TableGen def `int_mips_min_s_d`。
- **L1384 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1384 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1386 EN**: Declares TableGen def `int_mips_min_u_b`.
  **L1386 CN**: 声明 TableGen def `int_mips_min_u_b`。
- **L1387 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1387 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1388 EN**: Declares TableGen def `int_mips_min_u_h`.
  **L1388 CN**: 声明 TableGen def `int_mips_min_u_h`。
- **L1389 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1389 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1390 EN**: Declares TableGen def `int_mips_min_u_w`.
  **L1390 CN**: 声明 TableGen def `int_mips_min_u_w`。
- **L1391 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1391 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1392 EN**: Declares TableGen def `int_mips_min_u_d`.
  **L1392 CN**: 声明 TableGen def `int_mips_min_u_d`。
- **L1393 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1393 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1394 EN**: Blank line separating nearby declarations or logic blocks.
  **L1394 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1395 EN**: Declares TableGen def `int_mips_mini_s_b`.
  **L1395 CN**: 声明 TableGen def `int_mips_mini_s_b`。
- **L1396 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1396 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1397 EN**: Declares TableGen def `int_mips_mini_s_h`.
  **L1397 CN**: 声明 TableGen def `int_mips_mini_s_h`。
- **L1398 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1398 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1399 EN**: Declares TableGen def `int_mips_mini_s_w`.
  **L1399 CN**: 声明 TableGen def `int_mips_mini_s_w`。
- **L1400 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1400 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 1401-1428

````tablegen
def int_mips_mini_s_d : ClangBuiltin<"__builtin_msa_mini_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_mini_u_b : ClangBuiltin<"__builtin_msa_mini_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_mini_u_h : ClangBuiltin<"__builtin_msa_mini_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_mini_u_w : ClangBuiltin<"__builtin_msa_mini_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_mini_u_d : ClangBuiltin<"__builtin_msa_mini_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_mod_s_b : ClangBuiltin<"__builtin_msa_mod_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_mod_s_h : ClangBuiltin<"__builtin_msa_mod_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_mod_s_w : ClangBuiltin<"__builtin_msa_mod_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_mod_s_d : ClangBuiltin<"__builtin_msa_mod_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_mod_u_b : ClangBuiltin<"__builtin_msa_mod_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_mod_u_h : ClangBuiltin<"__builtin_msa_mod_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_mod_u_w : ClangBuiltin<"__builtin_msa_mod_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_mod_u_d : ClangBuiltin<"__builtin_msa_mod_u_d">,
````
- **L1401 EN**: Declares TableGen def `int_mips_mini_s_d`.
  **L1401 CN**: 声明 TableGen def `int_mips_mini_s_d`。
- **L1402 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1402 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1403 EN**: Blank line separating nearby declarations or logic blocks.
  **L1403 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1404 EN**: Declares TableGen def `int_mips_mini_u_b`.
  **L1404 CN**: 声明 TableGen def `int_mips_mini_u_b`。
- **L1405 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1405 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1406 EN**: Declares TableGen def `int_mips_mini_u_h`.
  **L1406 CN**: 声明 TableGen def `int_mips_mini_u_h`。
- **L1407 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1407 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1408 EN**: Declares TableGen def `int_mips_mini_u_w`.
  **L1408 CN**: 声明 TableGen def `int_mips_mini_u_w`。
- **L1409 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1409 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1410 EN**: Declares TableGen def `int_mips_mini_u_d`.
  **L1410 CN**: 声明 TableGen def `int_mips_mini_u_d`。
- **L1411 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1411 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1413 EN**: Declares TableGen def `int_mips_mod_s_b`.
  **L1413 CN**: 声明 TableGen def `int_mips_mod_s_b`。
- **L1414 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1414 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1415 EN**: Declares TableGen def `int_mips_mod_s_h`.
  **L1415 CN**: 声明 TableGen def `int_mips_mod_s_h`。
- **L1416 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1416 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1417 EN**: Declares TableGen def `int_mips_mod_s_w`.
  **L1417 CN**: 声明 TableGen def `int_mips_mod_s_w`。
- **L1418 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1418 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1419 EN**: Declares TableGen def `int_mips_mod_s_d`.
  **L1419 CN**: 声明 TableGen def `int_mips_mod_s_d`。
- **L1420 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1420 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1421 EN**: Blank line separating nearby declarations or logic blocks.
  **L1421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1422 EN**: Declares TableGen def `int_mips_mod_u_b`.
  **L1422 CN**: 声明 TableGen def `int_mips_mod_u_b`。
- **L1423 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1423 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1424 EN**: Declares TableGen def `int_mips_mod_u_h`.
  **L1424 CN**: 声明 TableGen def `int_mips_mod_u_h`。
- **L1425 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1425 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1426 EN**: Declares TableGen def `int_mips_mod_u_w`.
  **L1426 CN**: 声明 TableGen def `int_mips_mod_u_w`。
- **L1427 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1427 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1428 EN**: Declares TableGen def `int_mips_mod_u_d`.
  **L1428 CN**: 声明 TableGen def `int_mips_mod_u_d`。

### Lines 1429-1456

````tablegen
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_move_v : ClangBuiltin<"__builtin_msa_move_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;

def int_mips_msub_q_h : ClangBuiltin<"__builtin_msa_msub_q_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_msub_q_w : ClangBuiltin<"__builtin_msa_msub_q_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;

def int_mips_msubr_q_h : ClangBuiltin<"__builtin_msa_msubr_q_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_msubr_q_w : ClangBuiltin<"__builtin_msa_msubr_q_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;

def int_mips_msubv_b : ClangBuiltin<"__builtin_msa_msubv_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
  [IntrNoMem]>;
def int_mips_msubv_h : ClangBuiltin<"__builtin_msa_msubv_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
  [IntrNoMem]>;
def int_mips_msubv_w : ClangBuiltin<"__builtin_msa_msubv_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
  [IntrNoMem]>;
````
- **L1429 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1429 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1431 EN**: Declares TableGen def `int_mips_move_v`.
  **L1431 CN**: 声明 TableGen def `int_mips_move_v`。
- **L1432 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1432 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1433 EN**: Blank line separating nearby declarations or logic blocks.
  **L1433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1434 EN**: Declares TableGen def `int_mips_msub_q_h`.
  **L1434 CN**: 声明 TableGen def `int_mips_msub_q_h`。
- **L1435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1435 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1436 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1436 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1437 EN**: Declares TableGen def `int_mips_msub_q_w`.
  **L1437 CN**: 声明 TableGen def `int_mips_msub_q_w`。
- **L1438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1438 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1439 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1439 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1440 EN**: Blank line separating nearby declarations or logic blocks.
  **L1440 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1441 EN**: Declares TableGen def `int_mips_msubr_q_h`.
  **L1441 CN**: 声明 TableGen def `int_mips_msubr_q_h`。
- **L1442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1442 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1443 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1443 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1444 EN**: Declares TableGen def `int_mips_msubr_q_w`.
  **L1444 CN**: 声明 TableGen def `int_mips_msubr_q_w`。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1446 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1446 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1447 EN**: Blank line separating nearby declarations or logic blocks.
  **L1447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1448 EN**: Declares TableGen def `int_mips_msubv_b`.
  **L1448 CN**: 声明 TableGen def `int_mips_msubv_b`。
- **L1449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1449 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1450 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1450 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1451 EN**: Declares TableGen def `int_mips_msubv_h`.
  **L1451 CN**: 声明 TableGen def `int_mips_msubv_h`。
- **L1452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1452 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1453 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1453 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1454 EN**: Declares TableGen def `int_mips_msubv_w`.
  **L1454 CN**: 声明 TableGen def `int_mips_msubv_w`。
- **L1455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1455 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1456 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1456 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 1457-1484

````tablegen
def int_mips_msubv_d : ClangBuiltin<"__builtin_msa_msubv_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
  [IntrNoMem]>;

def int_mips_mul_q_h : ClangBuiltin<"__builtin_msa_mul_q_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_mul_q_w : ClangBuiltin<"__builtin_msa_mul_q_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_mulr_q_h : ClangBuiltin<"__builtin_msa_mulr_q_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_mulr_q_w : ClangBuiltin<"__builtin_msa_mulr_q_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

def int_mips_mulv_b : ClangBuiltin<"__builtin_msa_mulv_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_mulv_h : ClangBuiltin<"__builtin_msa_mulv_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_mulv_w : ClangBuiltin<"__builtin_msa_mulv_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_mulv_d : ClangBuiltin<"__builtin_msa_mulv_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_nloc_b : ClangBuiltin<"__builtin_msa_nloc_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_nloc_h : ClangBuiltin<"__builtin_msa_nloc_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_nloc_w : ClangBuiltin<"__builtin_msa_nloc_w">,
````
- **L1457 EN**: Declares TableGen def `int_mips_msubv_d`.
  **L1457 CN**: 声明 TableGen def `int_mips_msubv_d`。
- **L1458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1458 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1459 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1459 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1460 EN**: Blank line separating nearby declarations or logic blocks.
  **L1460 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1461 EN**: Declares TableGen def `int_mips_mul_q_h`.
  **L1461 CN**: 声明 TableGen def `int_mips_mul_q_h`。
- **L1462 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1462 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1463 EN**: Declares TableGen def `int_mips_mul_q_w`.
  **L1463 CN**: 声明 TableGen def `int_mips_mul_q_w`。
- **L1464 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1464 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1465 EN**: Blank line separating nearby declarations or logic blocks.
  **L1465 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1466 EN**: Declares TableGen def `int_mips_mulr_q_h`.
  **L1466 CN**: 声明 TableGen def `int_mips_mulr_q_h`。
- **L1467 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1467 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1468 EN**: Declares TableGen def `int_mips_mulr_q_w`.
  **L1468 CN**: 声明 TableGen def `int_mips_mulr_q_w`。
- **L1469 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1469 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1470 EN**: Blank line separating nearby declarations or logic blocks.
  **L1470 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1471 EN**: Declares TableGen def `int_mips_mulv_b`.
  **L1471 CN**: 声明 TableGen def `int_mips_mulv_b`。
- **L1472 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1472 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1473 EN**: Declares TableGen def `int_mips_mulv_h`.
  **L1473 CN**: 声明 TableGen def `int_mips_mulv_h`。
- **L1474 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1474 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1475 EN**: Declares TableGen def `int_mips_mulv_w`.
  **L1475 CN**: 声明 TableGen def `int_mips_mulv_w`。
- **L1476 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1476 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1477 EN**: Declares TableGen def `int_mips_mulv_d`.
  **L1477 CN**: 声明 TableGen def `int_mips_mulv_d`。
- **L1478 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1478 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1479 EN**: Blank line separating nearby declarations or logic blocks.
  **L1479 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1480 EN**: Declares TableGen def `int_mips_nloc_b`.
  **L1480 CN**: 声明 TableGen def `int_mips_nloc_b`。
- **L1481 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1481 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1482 EN**: Declares TableGen def `int_mips_nloc_h`.
  **L1482 CN**: 声明 TableGen def `int_mips_nloc_h`。
- **L1483 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1483 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1484 EN**: Declares TableGen def `int_mips_nloc_w`.
  **L1484 CN**: 声明 TableGen def `int_mips_nloc_w`。

### Lines 1485-1512

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_nloc_d : ClangBuiltin<"__builtin_msa_nloc_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_nlzc_b : ClangBuiltin<"__builtin_msa_nlzc_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_nlzc_h : ClangBuiltin<"__builtin_msa_nlzc_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_nlzc_w : ClangBuiltin<"__builtin_msa_nlzc_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_nlzc_d : ClangBuiltin<"__builtin_msa_nlzc_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_nor_v : ClangBuiltin<"__builtin_msa_nor_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;

def int_mips_nori_b : ClangBuiltin<"__builtin_msa_nori_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_or_v : ClangBuiltin<"__builtin_msa_or_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;

def int_mips_ori_b : ClangBuiltin<"__builtin_msa_ori_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_pckev_b : ClangBuiltin<"__builtin_msa_pckev_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_pckev_h : ClangBuiltin<"__builtin_msa_pckev_h">,
````
- **L1485 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1485 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1486 EN**: Declares TableGen def `int_mips_nloc_d`.
  **L1486 CN**: 声明 TableGen def `int_mips_nloc_d`。
- **L1487 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1487 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1488 EN**: Blank line separating nearby declarations or logic blocks.
  **L1488 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1489 EN**: Declares TableGen def `int_mips_nlzc_b`.
  **L1489 CN**: 声明 TableGen def `int_mips_nlzc_b`。
- **L1490 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1490 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1491 EN**: Declares TableGen def `int_mips_nlzc_h`.
  **L1491 CN**: 声明 TableGen def `int_mips_nlzc_h`。
- **L1492 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1492 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1493 EN**: Declares TableGen def `int_mips_nlzc_w`.
  **L1493 CN**: 声明 TableGen def `int_mips_nlzc_w`。
- **L1494 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1494 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1495 EN**: Declares TableGen def `int_mips_nlzc_d`.
  **L1495 CN**: 声明 TableGen def `int_mips_nlzc_d`。
- **L1496 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1496 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1497 EN**: Blank line separating nearby declarations or logic blocks.
  **L1497 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1498 EN**: Declares TableGen def `int_mips_nor_v`.
  **L1498 CN**: 声明 TableGen def `int_mips_nor_v`。
- **L1499 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1499 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1500 EN**: Blank line separating nearby declarations or logic blocks.
  **L1500 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1501 EN**: Declares TableGen def `int_mips_nori_b`.
  **L1501 CN**: 声明 TableGen def `int_mips_nori_b`。
- **L1502 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1502 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1503 EN**: Blank line separating nearby declarations or logic blocks.
  **L1503 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1504 EN**: Declares TableGen def `int_mips_or_v`.
  **L1504 CN**: 声明 TableGen def `int_mips_or_v`。
- **L1505 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1505 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1506 EN**: Blank line separating nearby declarations or logic blocks.
  **L1506 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1507 EN**: Declares TableGen def `int_mips_ori_b`.
  **L1507 CN**: 声明 TableGen def `int_mips_ori_b`。
- **L1508 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1508 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1509 EN**: Blank line separating nearby declarations or logic blocks.
  **L1509 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1510 EN**: Declares TableGen def `int_mips_pckev_b`.
  **L1510 CN**: 声明 TableGen def `int_mips_pckev_b`。
- **L1511 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1511 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1512 EN**: Declares TableGen def `int_mips_pckev_h`.
  **L1512 CN**: 声明 TableGen def `int_mips_pckev_h`。

### Lines 1513-1540

````tablegen
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_pckev_w : ClangBuiltin<"__builtin_msa_pckev_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_pckev_d : ClangBuiltin<"__builtin_msa_pckev_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_pckod_b : ClangBuiltin<"__builtin_msa_pckod_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_pckod_h : ClangBuiltin<"__builtin_msa_pckod_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_pckod_w : ClangBuiltin<"__builtin_msa_pckod_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_pckod_d : ClangBuiltin<"__builtin_msa_pckod_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_pcnt_b : ClangBuiltin<"__builtin_msa_pcnt_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_pcnt_h : ClangBuiltin<"__builtin_msa_pcnt_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_pcnt_w : ClangBuiltin<"__builtin_msa_pcnt_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_pcnt_d : ClangBuiltin<"__builtin_msa_pcnt_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_sat_s_b : ClangBuiltin<"__builtin_msa_sat_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_sat_s_h : ClangBuiltin<"__builtin_msa_sat_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L1513 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1513 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1514 EN**: Declares TableGen def `int_mips_pckev_w`.
  **L1514 CN**: 声明 TableGen def `int_mips_pckev_w`。
- **L1515 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1515 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1516 EN**: Declares TableGen def `int_mips_pckev_d`.
  **L1516 CN**: 声明 TableGen def `int_mips_pckev_d`。
- **L1517 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1517 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1518 EN**: Blank line separating nearby declarations or logic blocks.
  **L1518 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1519 EN**: Declares TableGen def `int_mips_pckod_b`.
  **L1519 CN**: 声明 TableGen def `int_mips_pckod_b`。
- **L1520 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1520 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1521 EN**: Declares TableGen def `int_mips_pckod_h`.
  **L1521 CN**: 声明 TableGen def `int_mips_pckod_h`。
- **L1522 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1522 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1523 EN**: Declares TableGen def `int_mips_pckod_w`.
  **L1523 CN**: 声明 TableGen def `int_mips_pckod_w`。
- **L1524 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1524 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1525 EN**: Declares TableGen def `int_mips_pckod_d`.
  **L1525 CN**: 声明 TableGen def `int_mips_pckod_d`。
- **L1526 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1526 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1527 EN**: Blank line separating nearby declarations or logic blocks.
  **L1527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1528 EN**: Declares TableGen def `int_mips_pcnt_b`.
  **L1528 CN**: 声明 TableGen def `int_mips_pcnt_b`。
- **L1529 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1529 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1530 EN**: Declares TableGen def `int_mips_pcnt_h`.
  **L1530 CN**: 声明 TableGen def `int_mips_pcnt_h`。
- **L1531 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1531 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1532 EN**: Declares TableGen def `int_mips_pcnt_w`.
  **L1532 CN**: 声明 TableGen def `int_mips_pcnt_w`。
- **L1533 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1533 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1534 EN**: Declares TableGen def `int_mips_pcnt_d`.
  **L1534 CN**: 声明 TableGen def `int_mips_pcnt_d`。
- **L1535 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1535 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1536 EN**: Blank line separating nearby declarations or logic blocks.
  **L1536 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1537 EN**: Declares TableGen def `int_mips_sat_s_b`.
  **L1537 CN**: 声明 TableGen def `int_mips_sat_s_b`。
- **L1538 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1538 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1539 EN**: Declares TableGen def `int_mips_sat_s_h`.
  **L1539 CN**: 声明 TableGen def `int_mips_sat_s_h`。
- **L1540 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1540 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 1541-1568

````tablegen
def int_mips_sat_s_w : ClangBuiltin<"__builtin_msa_sat_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_sat_s_d : ClangBuiltin<"__builtin_msa_sat_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_sat_u_b : ClangBuiltin<"__builtin_msa_sat_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_sat_u_h : ClangBuiltin<"__builtin_msa_sat_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_sat_u_w : ClangBuiltin<"__builtin_msa_sat_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_sat_u_d : ClangBuiltin<"__builtin_msa_sat_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_shf_b : ClangBuiltin<"__builtin_msa_shf_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_shf_h : ClangBuiltin<"__builtin_msa_shf_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_shf_w : ClangBuiltin<"__builtin_msa_shf_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_sld_b : ClangBuiltin<"__builtin_msa_sld_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_sld_h : ClangBuiltin<"__builtin_msa_sld_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_sld_w : ClangBuiltin<"__builtin_msa_sld_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_sld_d : ClangBuiltin<"__builtin_msa_sld_d">,
````
- **L1541 EN**: Declares TableGen def `int_mips_sat_s_w`.
  **L1541 CN**: 声明 TableGen def `int_mips_sat_s_w`。
- **L1542 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1542 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1543 EN**: Declares TableGen def `int_mips_sat_s_d`.
  **L1543 CN**: 声明 TableGen def `int_mips_sat_s_d`。
- **L1544 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1544 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1545 EN**: Blank line separating nearby declarations or logic blocks.
  **L1545 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1546 EN**: Declares TableGen def `int_mips_sat_u_b`.
  **L1546 CN**: 声明 TableGen def `int_mips_sat_u_b`。
- **L1547 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1547 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1548 EN**: Declares TableGen def `int_mips_sat_u_h`.
  **L1548 CN**: 声明 TableGen def `int_mips_sat_u_h`。
- **L1549 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1549 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1550 EN**: Declares TableGen def `int_mips_sat_u_w`.
  **L1550 CN**: 声明 TableGen def `int_mips_sat_u_w`。
- **L1551 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1551 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1552 EN**: Declares TableGen def `int_mips_sat_u_d`.
  **L1552 CN**: 声明 TableGen def `int_mips_sat_u_d`。
- **L1553 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1553 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1554 EN**: Blank line separating nearby declarations or logic blocks.
  **L1554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1555 EN**: Declares TableGen def `int_mips_shf_b`.
  **L1555 CN**: 声明 TableGen def `int_mips_shf_b`。
- **L1556 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1556 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1557 EN**: Declares TableGen def `int_mips_shf_h`.
  **L1557 CN**: 声明 TableGen def `int_mips_shf_h`。
- **L1558 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1558 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1559 EN**: Declares TableGen def `int_mips_shf_w`.
  **L1559 CN**: 声明 TableGen def `int_mips_shf_w`。
- **L1560 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1560 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1561 EN**: Blank line separating nearby declarations or logic blocks.
  **L1561 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1562 EN**: Declares TableGen def `int_mips_sld_b`.
  **L1562 CN**: 声明 TableGen def `int_mips_sld_b`。
- **L1563 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1563 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1564 EN**: Declares TableGen def `int_mips_sld_h`.
  **L1564 CN**: 声明 TableGen def `int_mips_sld_h`。
- **L1565 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1565 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1566 EN**: Declares TableGen def `int_mips_sld_w`.
  **L1566 CN**: 声明 TableGen def `int_mips_sld_w`。
- **L1567 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1567 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1568 EN**: Declares TableGen def `int_mips_sld_d`.
  **L1568 CN**: 声明 TableGen def `int_mips_sld_d`。

### Lines 1569-1596

````tablegen
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;

def int_mips_sldi_b : ClangBuiltin<"__builtin_msa_sldi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_sldi_h : ClangBuiltin<"__builtin_msa_sldi_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_sldi_w : ClangBuiltin<"__builtin_msa_sldi_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
def int_mips_sldi_d : ClangBuiltin<"__builtin_msa_sldi_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],
            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_mips_sll_b : ClangBuiltin<"__builtin_msa_sll_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_sll_h : ClangBuiltin<"__builtin_msa_sll_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_sll_w : ClangBuiltin<"__builtin_msa_sll_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_sll_d : ClangBuiltin<"__builtin_msa_sll_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_slli_b : ClangBuiltin<"__builtin_msa_slli_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_slli_h : ClangBuiltin<"__builtin_msa_slli_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L1569 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1569 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1570 EN**: Blank line separating nearby declarations or logic blocks.
  **L1570 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1571 EN**: Declares TableGen def `int_mips_sldi_b`.
  **L1571 CN**: 声明 TableGen def `int_mips_sldi_b`。
- **L1572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L1572 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L1573 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1573 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1574 EN**: Declares TableGen def `int_mips_sldi_h`.
  **L1574 CN**: 声明 TableGen def `int_mips_sldi_h`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`。
- **L1576 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1576 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1577 EN**: Declares TableGen def `int_mips_sldi_w`.
  **L1577 CN**: 声明 TableGen def `int_mips_sldi_w`。
- **L1578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`.
  **L1578 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`。
- **L1579 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1579 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1580 EN**: Declares TableGen def `int_mips_sldi_d`.
  **L1580 CN**: 声明 TableGen def `int_mips_sldi_d`。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`.
  **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`。
- **L1582 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1582 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1583 EN**: Blank line separating nearby declarations or logic blocks.
  **L1583 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1584 EN**: Declares TableGen def `int_mips_sll_b`.
  **L1584 CN**: 声明 TableGen def `int_mips_sll_b`。
- **L1585 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1585 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1586 EN**: Declares TableGen def `int_mips_sll_h`.
  **L1586 CN**: 声明 TableGen def `int_mips_sll_h`。
- **L1587 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1587 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1588 EN**: Declares TableGen def `int_mips_sll_w`.
  **L1588 CN**: 声明 TableGen def `int_mips_sll_w`。
- **L1589 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1589 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1590 EN**: Declares TableGen def `int_mips_sll_d`.
  **L1590 CN**: 声明 TableGen def `int_mips_sll_d`。
- **L1591 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1591 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1592 EN**: Blank line separating nearby declarations or logic blocks.
  **L1592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1593 EN**: Declares TableGen def `int_mips_slli_b`.
  **L1593 CN**: 声明 TableGen def `int_mips_slli_b`。
- **L1594 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1594 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1595 EN**: Declares TableGen def `int_mips_slli_h`.
  **L1595 CN**: 声明 TableGen def `int_mips_slli_h`。
- **L1596 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1596 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 1597-1624

````tablegen
def int_mips_slli_w : ClangBuiltin<"__builtin_msa_slli_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_slli_d : ClangBuiltin<"__builtin_msa_slli_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_splat_b : ClangBuiltin<"__builtin_msa_splat_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_splat_h : ClangBuiltin<"__builtin_msa_splat_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_splat_w : ClangBuiltin<"__builtin_msa_splat_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_mips_splat_d : ClangBuiltin<"__builtin_msa_splat_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;

def int_mips_splati_b : ClangBuiltin<"__builtin_msa_splati_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_splati_h : ClangBuiltin<"__builtin_msa_splati_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_splati_w : ClangBuiltin<"__builtin_msa_splati_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_splati_d : ClangBuiltin<"__builtin_msa_splati_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_sra_b : ClangBuiltin<"__builtin_msa_sra_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_sra_h : ClangBuiltin<"__builtin_msa_sra_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_sra_w : ClangBuiltin<"__builtin_msa_sra_w">,
````
- **L1597 EN**: Declares TableGen def `int_mips_slli_w`.
  **L1597 CN**: 声明 TableGen def `int_mips_slli_w`。
- **L1598 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1598 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1599 EN**: Declares TableGen def `int_mips_slli_d`.
  **L1599 CN**: 声明 TableGen def `int_mips_slli_d`。
- **L1600 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1600 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1602 EN**: Declares TableGen def `int_mips_splat_b`.
  **L1602 CN**: 声明 TableGen def `int_mips_splat_b`。
- **L1603 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1603 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1604 EN**: Declares TableGen def `int_mips_splat_h`.
  **L1604 CN**: 声明 TableGen def `int_mips_splat_h`。
- **L1605 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1605 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1606 EN**: Declares TableGen def `int_mips_splat_w`.
  **L1606 CN**: 声明 TableGen def `int_mips_splat_w`。
- **L1607 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1607 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1608 EN**: Declares TableGen def `int_mips_splat_d`.
  **L1608 CN**: 声明 TableGen def `int_mips_splat_d`。
- **L1609 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1609 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1610 EN**: Blank line separating nearby declarations or logic blocks.
  **L1610 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1611 EN**: Declares TableGen def `int_mips_splati_b`.
  **L1611 CN**: 声明 TableGen def `int_mips_splati_b`。
- **L1612 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1612 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1613 EN**: Declares TableGen def `int_mips_splati_h`.
  **L1613 CN**: 声明 TableGen def `int_mips_splati_h`。
- **L1614 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1614 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1615 EN**: Declares TableGen def `int_mips_splati_w`.
  **L1615 CN**: 声明 TableGen def `int_mips_splati_w`。
- **L1616 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1616 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1617 EN**: Declares TableGen def `int_mips_splati_d`.
  **L1617 CN**: 声明 TableGen def `int_mips_splati_d`。
- **L1618 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1618 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1619 EN**: Blank line separating nearby declarations or logic blocks.
  **L1619 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1620 EN**: Declares TableGen def `int_mips_sra_b`.
  **L1620 CN**: 声明 TableGen def `int_mips_sra_b`。
- **L1621 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1621 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1622 EN**: Declares TableGen def `int_mips_sra_h`.
  **L1622 CN**: 声明 TableGen def `int_mips_sra_h`。
- **L1623 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1623 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1624 EN**: Declares TableGen def `int_mips_sra_w`.
  **L1624 CN**: 声明 TableGen def `int_mips_sra_w`。

### Lines 1625-1652

````tablegen
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_sra_d : ClangBuiltin<"__builtin_msa_sra_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_srai_b : ClangBuiltin<"__builtin_msa_srai_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srai_h : ClangBuiltin<"__builtin_msa_srai_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srai_w : ClangBuiltin<"__builtin_msa_srai_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srai_d : ClangBuiltin<"__builtin_msa_srai_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_srar_b : ClangBuiltin<"__builtin_msa_srar_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_srar_h : ClangBuiltin<"__builtin_msa_srar_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_srar_w : ClangBuiltin<"__builtin_msa_srar_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_srar_d : ClangBuiltin<"__builtin_msa_srar_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_srari_b : ClangBuiltin<"__builtin_msa_srari_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srari_h : ClangBuiltin<"__builtin_msa_srari_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srari_w : ClangBuiltin<"__builtin_msa_srari_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L1625 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1625 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1626 EN**: Declares TableGen def `int_mips_sra_d`.
  **L1626 CN**: 声明 TableGen def `int_mips_sra_d`。
- **L1627 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1627 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1628 EN**: Blank line separating nearby declarations or logic blocks.
  **L1628 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1629 EN**: Declares TableGen def `int_mips_srai_b`.
  **L1629 CN**: 声明 TableGen def `int_mips_srai_b`。
- **L1630 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1630 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1631 EN**: Declares TableGen def `int_mips_srai_h`.
  **L1631 CN**: 声明 TableGen def `int_mips_srai_h`。
- **L1632 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1632 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1633 EN**: Declares TableGen def `int_mips_srai_w`.
  **L1633 CN**: 声明 TableGen def `int_mips_srai_w`。
- **L1634 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1634 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1635 EN**: Declares TableGen def `int_mips_srai_d`.
  **L1635 CN**: 声明 TableGen def `int_mips_srai_d`。
- **L1636 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1636 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1637 EN**: Blank line separating nearby declarations or logic blocks.
  **L1637 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1638 EN**: Declares TableGen def `int_mips_srar_b`.
  **L1638 CN**: 声明 TableGen def `int_mips_srar_b`。
- **L1639 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1639 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1640 EN**: Declares TableGen def `int_mips_srar_h`.
  **L1640 CN**: 声明 TableGen def `int_mips_srar_h`。
- **L1641 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1641 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1642 EN**: Declares TableGen def `int_mips_srar_w`.
  **L1642 CN**: 声明 TableGen def `int_mips_srar_w`。
- **L1643 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1643 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1644 EN**: Declares TableGen def `int_mips_srar_d`.
  **L1644 CN**: 声明 TableGen def `int_mips_srar_d`。
- **L1645 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1645 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1646 EN**: Blank line separating nearby declarations or logic blocks.
  **L1646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1647 EN**: Declares TableGen def `int_mips_srari_b`.
  **L1647 CN**: 声明 TableGen def `int_mips_srari_b`。
- **L1648 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1648 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1649 EN**: Declares TableGen def `int_mips_srari_h`.
  **L1649 CN**: 声明 TableGen def `int_mips_srari_h`。
- **L1650 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1650 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1651 EN**: Declares TableGen def `int_mips_srari_w`.
  **L1651 CN**: 声明 TableGen def `int_mips_srari_w`。
- **L1652 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1652 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 1653-1680

````tablegen
def int_mips_srari_d : ClangBuiltin<"__builtin_msa_srari_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_srl_b : ClangBuiltin<"__builtin_msa_srl_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_srl_h : ClangBuiltin<"__builtin_msa_srl_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_srl_w : ClangBuiltin<"__builtin_msa_srl_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_srl_d : ClangBuiltin<"__builtin_msa_srl_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_srli_b : ClangBuiltin<"__builtin_msa_srli_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srli_h : ClangBuiltin<"__builtin_msa_srli_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srli_w : ClangBuiltin<"__builtin_msa_srli_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srli_d : ClangBuiltin<"__builtin_msa_srli_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_srlr_b : ClangBuiltin<"__builtin_msa_srlr_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_srlr_h : ClangBuiltin<"__builtin_msa_srlr_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_srlr_w : ClangBuiltin<"__builtin_msa_srlr_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_srlr_d : ClangBuiltin<"__builtin_msa_srlr_d">,
````
- **L1653 EN**: Declares TableGen def `int_mips_srari_d`.
  **L1653 CN**: 声明 TableGen def `int_mips_srari_d`。
- **L1654 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1654 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1655 EN**: Blank line separating nearby declarations or logic blocks.
  **L1655 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1656 EN**: Declares TableGen def `int_mips_srl_b`.
  **L1656 CN**: 声明 TableGen def `int_mips_srl_b`。
- **L1657 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1657 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1658 EN**: Declares TableGen def `int_mips_srl_h`.
  **L1658 CN**: 声明 TableGen def `int_mips_srl_h`。
- **L1659 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1659 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1660 EN**: Declares TableGen def `int_mips_srl_w`.
  **L1660 CN**: 声明 TableGen def `int_mips_srl_w`。
- **L1661 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1661 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1662 EN**: Declares TableGen def `int_mips_srl_d`.
  **L1662 CN**: 声明 TableGen def `int_mips_srl_d`。
- **L1663 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1663 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1664 EN**: Blank line separating nearby declarations or logic blocks.
  **L1664 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1665 EN**: Declares TableGen def `int_mips_srli_b`.
  **L1665 CN**: 声明 TableGen def `int_mips_srli_b`。
- **L1666 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1666 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1667 EN**: Declares TableGen def `int_mips_srli_h`.
  **L1667 CN**: 声明 TableGen def `int_mips_srli_h`。
- **L1668 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1668 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1669 EN**: Declares TableGen def `int_mips_srli_w`.
  **L1669 CN**: 声明 TableGen def `int_mips_srli_w`。
- **L1670 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1670 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1671 EN**: Declares TableGen def `int_mips_srli_d`.
  **L1671 CN**: 声明 TableGen def `int_mips_srli_d`。
- **L1672 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1672 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1673 EN**: Blank line separating nearby declarations or logic blocks.
  **L1673 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1674 EN**: Declares TableGen def `int_mips_srlr_b`.
  **L1674 CN**: 声明 TableGen def `int_mips_srlr_b`。
- **L1675 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1675 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1676 EN**: Declares TableGen def `int_mips_srlr_h`.
  **L1676 CN**: 声明 TableGen def `int_mips_srlr_h`。
- **L1677 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1677 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1678 EN**: Declares TableGen def `int_mips_srlr_w`.
  **L1678 CN**: 声明 TableGen def `int_mips_srlr_w`。
- **L1679 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1679 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1680 EN**: Declares TableGen def `int_mips_srlr_d`.
  **L1680 CN**: 声明 TableGen def `int_mips_srlr_d`。

### Lines 1681-1708

````tablegen
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_srlri_b : ClangBuiltin<"__builtin_msa_srlri_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srlri_h : ClangBuiltin<"__builtin_msa_srlri_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srlri_w : ClangBuiltin<"__builtin_msa_srlri_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_srlri_d : ClangBuiltin<"__builtin_msa_srlri_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

def int_mips_st_b : ClangBuiltin<"__builtin_msa_st_b">,
  Intrinsic<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty],
  [IntrArgMemOnly]>;
def int_mips_st_h : ClangBuiltin<"__builtin_msa_st_h">,
  Intrinsic<[], [llvm_v8i16_ty, llvm_ptr_ty, llvm_i32_ty],
  [IntrArgMemOnly]>;
def int_mips_st_w : ClangBuiltin<"__builtin_msa_st_w">,
  Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty],
  [IntrArgMemOnly]>;
def int_mips_st_d : ClangBuiltin<"__builtin_msa_st_d">,
  Intrinsic<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty],
  [IntrArgMemOnly]>;

def int_mips_str_d : ClangBuiltin<"__builtin_msa_str_d">,
  Intrinsic<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty],
  [IntrArgMemOnly]>;
def int_mips_str_w : ClangBuiltin<"__builtin_msa_str_w">,
````
- **L1681 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1681 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1682 EN**: Blank line separating nearby declarations or logic blocks.
  **L1682 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1683 EN**: Declares TableGen def `int_mips_srlri_b`.
  **L1683 CN**: 声明 TableGen def `int_mips_srlri_b`。
- **L1684 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1684 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1685 EN**: Declares TableGen def `int_mips_srlri_h`.
  **L1685 CN**: 声明 TableGen def `int_mips_srlri_h`。
- **L1686 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1686 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1687 EN**: Declares TableGen def `int_mips_srlri_w`.
  **L1687 CN**: 声明 TableGen def `int_mips_srlri_w`。
- **L1688 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1688 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1689 EN**: Declares TableGen def `int_mips_srlri_d`.
  **L1689 CN**: 声明 TableGen def `int_mips_srlri_d`。
- **L1690 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1690 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1691 EN**: Blank line separating nearby declarations or logic blocks.
  **L1691 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1692 EN**: Declares TableGen def `int_mips_st_b`.
  **L1692 CN**: 声明 TableGen def `int_mips_st_b`。
- **L1693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L1693 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v16i8_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L1694 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L1694 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L1695 EN**: Declares TableGen def `int_mips_st_h`.
  **L1695 CN**: 声明 TableGen def `int_mips_st_h`。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v8i16_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v8i16_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L1697 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L1697 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L1698 EN**: Declares TableGen def `int_mips_st_w`.
  **L1698 CN**: 声明 TableGen def `int_mips_st_w`。
- **L1699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L1699 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L1700 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L1700 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L1701 EN**: Declares TableGen def `int_mips_st_d`.
  **L1701 CN**: 声明 TableGen def `int_mips_st_d`。
- **L1702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L1702 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L1703 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L1703 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L1704 EN**: Blank line separating nearby declarations or logic blocks.
  **L1704 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1705 EN**: Declares TableGen def `int_mips_str_d`.
  **L1705 CN**: 声明 TableGen def `int_mips_str_d`。
- **L1706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L1706 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v2i64_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L1707 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L1707 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L1708 EN**: Declares TableGen def `int_mips_str_w`.
  **L1708 CN**: 声明 TableGen def `int_mips_str_w`。

### Lines 1709-1736

````tablegen
  Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty],
  [IntrArgMemOnly]>;

def int_mips_subs_s_b : ClangBuiltin<"__builtin_msa_subs_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_subs_s_h : ClangBuiltin<"__builtin_msa_subs_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_subs_s_w : ClangBuiltin<"__builtin_msa_subs_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_subs_s_d : ClangBuiltin<"__builtin_msa_subs_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_subs_u_b : ClangBuiltin<"__builtin_msa_subs_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_subs_u_h : ClangBuiltin<"__builtin_msa_subs_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_subs_u_w : ClangBuiltin<"__builtin_msa_subs_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_subs_u_d : ClangBuiltin<"__builtin_msa_subs_u_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_subsus_u_b : ClangBuiltin<"__builtin_msa_subsus_u_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_subsus_u_h : ClangBuiltin<"__builtin_msa_subsus_u_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_subsus_u_w : ClangBuiltin<"__builtin_msa_subsus_u_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_subsus_u_d : ClangBuiltin<"__builtin_msa_subsus_u_d">,
````
- **L1709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty],`.
  **L1709 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i32_ty],`。
- **L1710 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L1710 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L1711 EN**: Blank line separating nearby declarations or logic blocks.
  **L1711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1712 EN**: Declares TableGen def `int_mips_subs_s_b`.
  **L1712 CN**: 声明 TableGen def `int_mips_subs_s_b`。
- **L1713 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1713 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1714 EN**: Declares TableGen def `int_mips_subs_s_h`.
  **L1714 CN**: 声明 TableGen def `int_mips_subs_s_h`。
- **L1715 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1715 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1716 EN**: Declares TableGen def `int_mips_subs_s_w`.
  **L1716 CN**: 声明 TableGen def `int_mips_subs_s_w`。
- **L1717 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1717 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1718 EN**: Declares TableGen def `int_mips_subs_s_d`.
  **L1718 CN**: 声明 TableGen def `int_mips_subs_s_d`。
- **L1719 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1719 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1720 EN**: Blank line separating nearby declarations or logic blocks.
  **L1720 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1721 EN**: Declares TableGen def `int_mips_subs_u_b`.
  **L1721 CN**: 声明 TableGen def `int_mips_subs_u_b`。
- **L1722 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1722 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1723 EN**: Declares TableGen def `int_mips_subs_u_h`.
  **L1723 CN**: 声明 TableGen def `int_mips_subs_u_h`。
- **L1724 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1724 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1725 EN**: Declares TableGen def `int_mips_subs_u_w`.
  **L1725 CN**: 声明 TableGen def `int_mips_subs_u_w`。
- **L1726 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1726 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1727 EN**: Declares TableGen def `int_mips_subs_u_d`.
  **L1727 CN**: 声明 TableGen def `int_mips_subs_u_d`。
- **L1728 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1728 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1729 EN**: Blank line separating nearby declarations or logic blocks.
  **L1729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1730 EN**: Declares TableGen def `int_mips_subsus_u_b`.
  **L1730 CN**: 声明 TableGen def `int_mips_subsus_u_b`。
- **L1731 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1731 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1732 EN**: Declares TableGen def `int_mips_subsus_u_h`.
  **L1732 CN**: 声明 TableGen def `int_mips_subsus_u_h`。
- **L1733 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1733 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1734 EN**: Declares TableGen def `int_mips_subsus_u_w`.
  **L1734 CN**: 声明 TableGen def `int_mips_subsus_u_w`。
- **L1735 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1735 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1736 EN**: Declares TableGen def `int_mips_subsus_u_d`.
  **L1736 CN**: 声明 TableGen def `int_mips_subsus_u_d`。

### Lines 1737-1764

````tablegen
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_subsuu_s_b : ClangBuiltin<"__builtin_msa_subsuu_s_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_subsuu_s_h : ClangBuiltin<"__builtin_msa_subsuu_s_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_subsuu_s_w : ClangBuiltin<"__builtin_msa_subsuu_s_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_subsuu_s_d : ClangBuiltin<"__builtin_msa_subsuu_s_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_subv_b : ClangBuiltin<"__builtin_msa_subv_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;
def int_mips_subv_h : ClangBuiltin<"__builtin_msa_subv_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;
def int_mips_subv_w : ClangBuiltin<"__builtin_msa_subv_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_mips_subv_d : ClangBuiltin<"__builtin_msa_subv_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

def int_mips_subvi_b : ClangBuiltin<"__builtin_msa_subvi_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_subvi_h : ClangBuiltin<"__builtin_msa_subvi_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_subvi_w : ClangBuiltin<"__builtin_msa_subvi_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_mips_subvi_d : ClangBuiltin<"__builtin_msa_subvi_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L1737 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1737 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1738 EN**: Blank line separating nearby declarations or logic blocks.
  **L1738 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1739 EN**: Declares TableGen def `int_mips_subsuu_s_b`.
  **L1739 CN**: 声明 TableGen def `int_mips_subsuu_s_b`。
- **L1740 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1740 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1741 EN**: Declares TableGen def `int_mips_subsuu_s_h`.
  **L1741 CN**: 声明 TableGen def `int_mips_subsuu_s_h`。
- **L1742 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1742 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1743 EN**: Declares TableGen def `int_mips_subsuu_s_w`.
  **L1743 CN**: 声明 TableGen def `int_mips_subsuu_s_w`。
- **L1744 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1744 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1745 EN**: Declares TableGen def `int_mips_subsuu_s_d`.
  **L1745 CN**: 声明 TableGen def `int_mips_subsuu_s_d`。
- **L1746 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1746 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1747 EN**: Blank line separating nearby declarations or logic blocks.
  **L1747 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1748 EN**: Declares TableGen def `int_mips_subv_b`.
  **L1748 CN**: 声明 TableGen def `int_mips_subv_b`。
- **L1749 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1749 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1750 EN**: Declares TableGen def `int_mips_subv_h`.
  **L1750 CN**: 声明 TableGen def `int_mips_subv_h`。
- **L1751 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1751 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1752 EN**: Declares TableGen def `int_mips_subv_w`.
  **L1752 CN**: 声明 TableGen def `int_mips_subv_w`。
- **L1753 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1753 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1754 EN**: Declares TableGen def `int_mips_subv_d`.
  **L1754 CN**: 声明 TableGen def `int_mips_subv_d`。
- **L1755 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1755 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1756 EN**: Blank line separating nearby declarations or logic blocks.
  **L1756 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1757 EN**: Declares TableGen def `int_mips_subvi_b`.
  **L1757 CN**: 声明 TableGen def `int_mips_subvi_b`。
- **L1758 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1758 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1759 EN**: Declares TableGen def `int_mips_subvi_h`.
  **L1759 CN**: 声明 TableGen def `int_mips_subvi_h`。
- **L1760 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1760 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1761 EN**: Declares TableGen def `int_mips_subvi_w`.
  **L1761 CN**: 声明 TableGen def `int_mips_subvi_w`。
- **L1762 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1762 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1763 EN**: Declares TableGen def `int_mips_subvi_d`.
  **L1763 CN**: 声明 TableGen def `int_mips_subvi_d`。
- **L1764 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1764 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 1765-1784

````tablegen

def int_mips_vshf_b : ClangBuiltin<"__builtin_msa_vshf_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
            [IntrNoMem]>;
def int_mips_vshf_h : ClangBuiltin<"__builtin_msa_vshf_h">,
  Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],
            [IntrNoMem]>;
def int_mips_vshf_w : ClangBuiltin<"__builtin_msa_vshf_w">,
  Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
            [IntrNoMem]>;
def int_mips_vshf_d : ClangBuiltin<"__builtin_msa_vshf_d">,
  Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
            [IntrNoMem]>;

def int_mips_xor_v : ClangBuiltin<"__builtin_msa_xor_v">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;

def int_mips_xori_b : ClangBuiltin<"__builtin_msa_xori_b">,
  Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
}
````
- **L1765 EN**: Blank line separating nearby declarations or logic blocks.
  **L1765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1766 EN**: Declares TableGen def `int_mips_vshf_b`.
  **L1766 CN**: 声明 TableGen def `int_mips_vshf_b`。
- **L1767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1767 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1768 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1768 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1769 EN**: Declares TableGen def `int_mips_vshf_h`.
  **L1769 CN**: 声明 TableGen def `int_mips_vshf_h`。
- **L1770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1770 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1771 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1771 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1772 EN**: Declares TableGen def `int_mips_vshf_w`.
  **L1772 CN**: 声明 TableGen def `int_mips_vshf_w`。
- **L1773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1773 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1774 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1774 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1775 EN**: Declares TableGen def `int_mips_vshf_d`.
  **L1775 CN**: 声明 TableGen def `int_mips_vshf_d`。
- **L1776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1776 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1777 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1777 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1778 EN**: Blank line separating nearby declarations or logic blocks.
  **L1778 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1779 EN**: Declares TableGen def `int_mips_xor_v`.
  **L1779 CN**: 声明 TableGen def `int_mips_xor_v`。
- **L1780 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1780 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1781 EN**: Blank line separating nearby declarations or logic blocks.
  **L1781 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1782 EN**: Declares TableGen def `int_mips_xori_b`.
  **L1782 CN**: 声明 TableGen def `int_mips_xori_b`。
- **L1783 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1783 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1784 EN**: Closes the current lexical scope or compound statement.
  **L1784 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
