# IntrinsicsAArch64.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsAArch64.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the AARCH64-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsAArch64` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````tablegen
//===- IntrinsicsAARCH64.td - Defines AARCH64 intrinsics ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the AARCH64-specific intrinsics.
//
//===----------------------------------------------------------------------===//

let TargetPrefix = "aarch64" in {

def int_aarch64_ldxr : Intrinsic<[llvm_i64_ty], [llvm_anyptr_ty],
                                 [IntrNoFree, IntrWillReturn]>;
def int_aarch64_ldaxr : Intrinsic<[llvm_i64_ty], [llvm_anyptr_ty],
                                  [IntrNoFree, IntrWillReturn]>;
def int_aarch64_stxr : Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_anyptr_ty],
                                 [IntrNoFree, IntrWillReturn]>;
def int_aarch64_stlxr : Intrinsic<[llvm_i32_ty], [llvm_i64_ty, llvm_anyptr_ty],
                                  [IntrNoFree, IntrWillReturn]>;

def int_aarch64_ldxp : Intrinsic<[llvm_i64_ty, llvm_i64_ty], [llvm_ptr_ty],
                                 [IntrNoFree, IntrWillReturn]>;
def int_aarch64_ldaxp : Intrinsic<[llvm_i64_ty, llvm_i64_ty], [llvm_ptr_ty],
                                  [IntrNoFree, IntrWillReturn]>;
def int_aarch64_stxp : Intrinsic<[llvm_i32_ty],
                               [llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],
                               [IntrNoFree, IntrWillReturn]>;
def int_aarch64_stlxp : Intrinsic<[llvm_i32_ty],
                                  [llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the AARCH64-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the AARCH64-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L13 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Declares TableGen def `int_aarch64_ldxr`.
  **L15 CN**: 声明 TableGen def `int_aarch64_ldxr`。
- **L16 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L16 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L17 EN**: Declares TableGen def `int_aarch64_ldaxr`.
  **L17 CN**: 声明 TableGen def `int_aarch64_ldaxr`。
- **L18 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L18 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L19 EN**: Declares TableGen def `int_aarch64_stxr`.
  **L19 CN**: 声明 TableGen def `int_aarch64_stxr`。
- **L20 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L20 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L21 EN**: Declares TableGen def `int_aarch64_stlxr`.
  **L21 CN**: 声明 TableGen def `int_aarch64_stlxr`。
- **L22 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L22 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Declares TableGen def `int_aarch64_ldxp`.
  **L24 CN**: 声明 TableGen def `int_aarch64_ldxp`。
- **L25 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L25 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L26 EN**: Declares TableGen def `int_aarch64_ldaxp`.
  **L26 CN**: 声明 TableGen def `int_aarch64_ldaxp`。
- **L27 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L27 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L28 EN**: Declares TableGen def `int_aarch64_stxp`.
  **L28 CN**: 声明 TableGen def `int_aarch64_stxp`。
- **L29 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],`.
  **L29 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],`。
- **L30 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L30 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L31 EN**: Declares TableGen def `int_aarch64_stlxp`.
  **L31 CN**: 声明 TableGen def `int_aarch64_stlxp`。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],`。

### Lines 33-64

````tablegen
                                  [IntrNoFree, IntrWillReturn]>;

def int_aarch64_clrex : Intrinsic<[]>;

def int_aarch64_sdiv : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>,
                                LLVMMatchType<0>], [IntrNoMem]>;
def int_aarch64_udiv : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>,
                                LLVMMatchType<0>], [IntrNoMem]>;

def int_aarch64_fjcvtzs : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_double_ty], [IntrNoMem]>;

def int_aarch64_cls: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;
def int_aarch64_cls64: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty], [IntrNoMem]>;

def int_aarch64_frint32z
    : DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],
                            [ IntrNoMem ]>;
def int_aarch64_frint64z
    : DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],
                            [ IntrNoMem ]>;
def int_aarch64_frint32x
    : DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],
                            [ IntrNoMem ]>;
def int_aarch64_frint64x
    : DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],
                            [ IntrNoMem ]>;

def int_aarch64_sys  : DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_i32_ty,
                       llvm_i32_ty, llvm_i32_ty, llvm_i64_ty],
                       [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>,
                       ImmArg<ArgIndex<3>>]>;
//===----------------------------------------------------------------------===//
````
- **L33 EN**: Executes a standalone statement or declaration: `[IntrNoFree, IntrWillReturn]>;`.
  **L33 CN**: 执行一条独立语句或声明：`[IntrNoFree, IntrWillReturn]>;`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Declares TableGen def `int_aarch64_clrex`.
  **L35 CN**: 声明 TableGen def `int_aarch64_clrex`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Declares TableGen def `int_aarch64_sdiv`.
  **L37 CN**: 声明 TableGen def `int_aarch64_sdiv`。
- **L38 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>], [IntrNoMem]>;`.
  **L38 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>], [IntrNoMem]>;`。
- **L39 EN**: Declares TableGen def `int_aarch64_udiv`.
  **L39 CN**: 声明 TableGen def `int_aarch64_udiv`。
- **L40 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>], [IntrNoMem]>;`.
  **L40 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>], [IntrNoMem]>;`。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Declares TableGen def `int_aarch64_fjcvtzs`.
  **L42 CN**: 声明 TableGen def `int_aarch64_fjcvtzs`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Declares TableGen def `int_aarch64_cls`.
  **L44 CN**: 声明 TableGen def `int_aarch64_cls`。
- **L45 EN**: Declares TableGen def `int_aarch64_cls64`.
  **L45 CN**: 声明 TableGen def `int_aarch64_cls64`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Declares TableGen def `int_aarch64_frint32z`.
  **L47 CN**: 声明 TableGen def `int_aarch64_frint32z`。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`。
- **L49 EN**: Executes a standalone statement or declaration: `[ IntrNoMem ]>;`.
  **L49 CN**: 执行一条独立语句或声明：`[ IntrNoMem ]>;`。
- **L50 EN**: Declares TableGen def `int_aarch64_frint64z`.
  **L50 CN**: 声明 TableGen def `int_aarch64_frint64z`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`。
- **L52 EN**: Executes a standalone statement or declaration: `[ IntrNoMem ]>;`.
  **L52 CN**: 执行一条独立语句或声明：`[ IntrNoMem ]>;`。
- **L53 EN**: Declares TableGen def `int_aarch64_frint32x`.
  **L53 CN**: 声明 TableGen def `int_aarch64_frint32x`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`。
- **L55 EN**: Executes a standalone statement or declaration: `[ IntrNoMem ]>;`.
  **L55 CN**: 执行一条独立语句或声明：`[ IntrNoMem ]>;`。
- **L56 EN**: Declares TableGen def `int_aarch64_frint64x`.
  **L56 CN**: 声明 TableGen def `int_aarch64_frint64x`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[ llvm_anyfloat_ty ], [ LLVMMatchType<0> ],`。
- **L58 EN**: Executes a standalone statement or declaration: `[ IntrNoMem ]>;`.
  **L58 CN**: 执行一条独立语句或声明：`[ IntrNoMem ]>;`。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L60 EN**: Declares TableGen def `int_aarch64_sys`.
  **L60 CN**: 声明 TableGen def `int_aarch64_sys`。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_i64_ty],`.
  **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_i64_ty],`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>,`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>,`。
- **L63 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>]>;`.
  **L63 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>]>;`。
- **L64 EN**: Banner comment marking a file or section boundary.
  **L64 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 65-96

````tablegen
// HINT

def int_aarch64_hint : DefaultAttrsIntrinsic<[], [llvm_i32_ty]>;

def int_aarch64_break : Intrinsic<[], [llvm_i32_ty],
    [IntrNoMem, IntrHasSideEffects, IntrNoReturn, IntrCold, ImmArg<ArgIndex<0>>]>;

def int_aarch64_hlt : Intrinsic<[], [llvm_i32_ty],
    [IntrNoMem, IntrHasSideEffects, IntrNoReturn, IntrCold, ImmArg<ArgIndex<0>>]>;

def int_aarch64_prefetch : Intrinsic<[],
    [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
    [IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, ReadOnly<ArgIndex<0>>,
     ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>
     ]>,
    ClangBuiltin<"__builtin_arm_prefetch">;

def int_aarch64_prefetch_ir : Intrinsic<[], [llvm_ptr_ty],
    [IntrHasSideEffects, IntrInaccessibleMemOrArgMemOnly,
     IntrWillReturn, ReadOnly<ArgIndex<0>>]>,
    ClangBuiltin<"__builtin_arm_prefetch_ir">;

def int_aarch64_range_prefetch : Intrinsic<[],
    [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_i64_ty],
    [IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, ReadOnly<ArgIndex<0>>,
     ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>,
    ClangBuiltin<"__builtin_arm_range_prefetch">;

//===----------------------------------------------------------------------===//
// Data Barrier Instructions

def int_aarch64_dmb : ClangBuiltin<"__builtin_arm_dmb">, MSBuiltin<"__dmb">,
````
- **L65 EN**: Comment explains nearby logic, invariants, or intent: `HINT`.
  **L65 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`HINT`。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Declares TableGen def `int_aarch64_hint`.
  **L67 CN**: 声明 TableGen def `int_aarch64_hint`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Declares TableGen def `int_aarch64_break`.
  **L69 CN**: 声明 TableGen def `int_aarch64_break`。
- **L70 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects, IntrNoReturn, IntrCold, ImmArg<ArgIndex<0>>]>;`.
  **L70 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects, IntrNoReturn, IntrCold, ImmArg<ArgIndex<0>>]>;`。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L72 EN**: Declares TableGen def `int_aarch64_hlt`.
  **L72 CN**: 声明 TableGen def `int_aarch64_hlt`。
- **L73 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects, IntrNoReturn, IntrCold, ImmArg<ArgIndex<0>>]>;`.
  **L73 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects, IntrNoReturn, IntrCold, ImmArg<ArgIndex<0>>]>;`。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Declares TableGen def `int_aarch64_prefetch`.
  **L75 CN**: 声明 TableGen def `int_aarch64_prefetch`。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L77 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, ReadOnly<ArgIndex<0>>,`.
  **L77 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, ReadOnly<ArgIndex<0>>,`。
- **L78 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>`.
  **L78 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>`。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `]>,`.
  **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`]>,`。
- **L80 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_arm_prefetch">;`.
  **L80 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_arm_prefetch">;`。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Declares TableGen def `int_aarch64_prefetch_ir`.
  **L82 CN**: 声明 TableGen def `int_aarch64_prefetch_ir`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrHasSideEffects, IntrInaccessibleMemOrArgMemOnly,`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrHasSideEffects, IntrInaccessibleMemOrArgMemOnly,`。
- **L84 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrWillReturn, ReadOnly<ArgIndex<0>>]>,`.
  **L84 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrWillReturn, ReadOnly<ArgIndex<0>>]>,`。
- **L85 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_arm_prefetch_ir">;`.
  **L85 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_arm_prefetch_ir">;`。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L87 EN**: Declares TableGen def `int_aarch64_range_prefetch`.
  **L87 CN**: 声明 TableGen def `int_aarch64_range_prefetch`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_i64_ty],`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty, llvm_i64_ty],`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, ReadOnly<ArgIndex<0>>,`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOrArgMemOnly, IntrWillReturn, ReadOnly<ArgIndex<0>>,`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>,`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>,`。
- **L91 EN**: Executes a standalone statement or declaration: `ClangBuiltin<"__builtin_arm_range_prefetch">;`.
  **L91 CN**: 执行一条独立语句或声明：`ClangBuiltin<"__builtin_arm_range_prefetch">;`。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Banner comment marking a file or section boundary.
  **L93 CN**: 横幅注释，用于标记文件或章节边界。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `Data Barrier Instructions`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Data Barrier Instructions`。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Declares TableGen def `int_aarch64_dmb`.
  **L96 CN**: 声明 TableGen def `int_aarch64_dmb`。

### Lines 97-128

````tablegen
                      Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;
def int_aarch64_dsb : ClangBuiltin<"__builtin_arm_dsb">, MSBuiltin<"__dsb">,
                      Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;
def int_aarch64_isb : ClangBuiltin<"__builtin_arm_isb">, MSBuiltin<"__isb">,
                      Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;

// A space-consuming intrinsic primarily for testing block and jump table
// placements. The first argument is the number of bytes this "instruction"
// takes up, the second and return value are essentially chains, used to force
// ordering during ISel.
def int_aarch64_space : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i64_ty], []>;

//===----------------------------------------------------------------------===//
// Guarded Control Stack

def int_aarch64_chkfeat : ClangBuiltin<"__builtin_arm_chkfeat">,
                          DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty],
                                                [IntrNoMem]>;

// FIXME: This should be marked as [IntrReadMem, IntrHasSideEffects], as it has
// the side-effect of updating gcspr, but this combination doesn't work
// correctly.
def int_aarch64_gcspopm : ClangBuiltin<"__builtin_arm_gcspopm">,
                          DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty],
                                                []>;

def int_aarch64_gcsss : ClangBuiltin<"__builtin_arm_gcsss">,
                        DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty], []>;

}

//===----------------------------------------------------------------------===//
````
- **L97 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;`.
  **L97 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;`。
- **L98 EN**: Declares TableGen def `int_aarch64_dsb`.
  **L98 CN**: 声明 TableGen def `int_aarch64_dsb`。
- **L99 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;`.
  **L99 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;`。
- **L100 EN**: Declares TableGen def `int_aarch64_isb`.
  **L100 CN**: 声明 TableGen def `int_aarch64_isb`。
- **L101 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;`.
  **L101 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [IntrNoFree, IntrWillReturn]>;`。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L103 EN**: Comment explains nearby logic, invariants, or intent: `A space-consuming intrinsic primarily for testing block and jump table`.
  **L103 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A space-consuming intrinsic primarily for testing block and jump table`。
- **L104 EN**: Comment explains nearby logic, invariants, or intent: `placements. The first argument is the number of bytes this "instruction"`.
  **L104 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`placements. The first argument is the number of bytes this "instruction"`。
- **L105 EN**: Comment explains nearby logic, invariants, or intent: `takes up, the second and return value are essentially chains, used to force`.
  **L105 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`takes up, the second and return value are essentially chains, used to force`。
- **L106 EN**: Comment explains nearby logic, invariants, or intent: `ordering during ISel.`.
  **L106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ordering during ISel.`。
- **L107 EN**: Declares TableGen def `int_aarch64_space`.
  **L107 CN**: 声明 TableGen def `int_aarch64_space`。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L109 EN**: Banner comment marking a file or section boundary.
  **L109 CN**: 横幅注释，用于标记文件或章节边界。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `Guarded Control Stack`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Guarded Control Stack`。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L112 EN**: Declares TableGen def `int_aarch64_chkfeat`.
  **L112 CN**: 声明 TableGen def `int_aarch64_chkfeat`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty],`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty],`。
- **L114 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L114 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L115 EN**: Blank line separating nearby declarations or logic blocks.
  **L115 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L116 EN**: Comment records a pending task or caution: `FIXME: This should be marked as [IntrReadMem, IntrHasSideEffects], as it has`.
  **L116 CN**: 注释记录了待办事项或注意点：`FIXME: This should be marked as [IntrReadMem, IntrHasSideEffects], as it has`。
- **L117 EN**: Comment explains nearby logic, invariants, or intent: `the side-effect of updating gcspr, but this combination doesn't work`.
  **L117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the side-effect of updating gcspr, but this combination doesn't work`。
- **L118 EN**: Comment explains nearby logic, invariants, or intent: `correctly.`.
  **L118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`correctly.`。
- **L119 EN**: Declares TableGen def `int_aarch64_gcspopm`.
  **L119 CN**: 声明 TableGen def `int_aarch64_gcspopm`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty],`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty],`。
- **L121 EN**: Executes a standalone statement or declaration: `[]>;`.
  **L121 CN**: 执行一条独立语句或声明：`[]>;`。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L123 EN**: Declares TableGen def `int_aarch64_gcsss`.
  **L123 CN**: 声明 TableGen def `int_aarch64_gcsss`。
- **L124 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty], []>;`.
  **L124 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty], []>;`。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L126 EN**: Closes the current lexical scope or compound statement.
  **L126 CN**: 结束当前词法作用域或复合语句块。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L128 EN**: Banner comment marking a file or section boundary.
  **L128 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 129-160

````tablegen
// Advanced SIMD (NEON)

let TargetPrefix = "aarch64" in {  // All intrinsics start with "llvm.aarch64.".
  class AdvSIMD_2Scalar_Float_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrNoMem]>;

  class AdvSIMD_FPToIntRounding_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty], [IntrNoMem]>;

  class AdvSIMD_1IntArg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrNoMem]>;
  class AdvSIMD_1FloatArg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem]>;
  class AdvSIMD_1VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMMatchType<0>], !listconcat(Attrs, [IntrNoMem])>;
  class AdvSIMD_1VectorArg_Expand_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;
  class AdvSIMD_1IntArg_Narrow_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_any_ty], [llvm_any_ty], [IntrNoMem]>;
  class AdvSIMD_1VectorArg_Narrow_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMExtendedType<0>], [IntrNoMem]>;
  class AdvSIMD_1VectorArg_Int_Across_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyvector_ty], [IntrNoMem]>;
  class AdvSIMD_1VectorArg_Float_Across_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyvector_ty], [IntrNoMem]>;

  class AdvSIMD_2IntArg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrNoMem]>;
  class AdvSIMD_2FloatArg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>],
````
- **L129 EN**: Comment explains nearby logic, invariants, or intent: `Advanced SIMD (NEON)`.
  **L129 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Advanced SIMD (NEON)`。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L131 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L132 EN**: Declares class `AdvSIMD_2Scalar_Float_Intrinsic`.
  **L132 CN**: 声明 class `AdvSIMD_2Scalar_Float_Intrinsic`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L134 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L134 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Declares class `AdvSIMD_FPToIntRounding_Intrinsic`.
  **L136 CN**: 声明 class `AdvSIMD_FPToIntRounding_Intrinsic`。
- **L137 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty], [IntrNoMem]>;`.
  **L137 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty], [IntrNoMem]>;`。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L139 EN**: Declares class `AdvSIMD_1IntArg_Intrinsic`.
  **L139 CN**: 声明 class `AdvSIMD_1IntArg_Intrinsic`。
- **L140 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrNoMem]>;`.
  **L140 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>], [IntrNoMem]>;`。
- **L141 EN**: Declares class `AdvSIMD_1FloatArg_Intrinsic`.
  **L141 CN**: 声明 class `AdvSIMD_1FloatArg_Intrinsic`。
- **L142 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem]>;`.
  **L142 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>], [IntrNoMem]>;`。
- **L143 EN**: Declares class `AdvSIMD_1VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L143 CN**: 声明 class `AdvSIMD_1VectorArg_Intrinsic<list<IntrinsicProperty>`。
- **L144 EN**: Executes a call or declaration centered on `!listconcat`.
  **L144 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L145 EN**: Declares class `AdvSIMD_1VectorArg_Expand_Intrinsic`.
  **L145 CN**: 声明 class `AdvSIMD_1VectorArg_Expand_Intrinsic`。
- **L146 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L146 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。
- **L147 EN**: Declares class `AdvSIMD_1IntArg_Narrow_Intrinsic`.
  **L147 CN**: 声明 class `AdvSIMD_1IntArg_Narrow_Intrinsic`。
- **L148 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_any_ty], [llvm_any_ty], [IntrNoMem]>;`.
  **L148 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_any_ty], [llvm_any_ty], [IntrNoMem]>;`。
- **L149 EN**: Declares class `AdvSIMD_1VectorArg_Narrow_Intrinsic`.
  **L149 CN**: 声明 class `AdvSIMD_1VectorArg_Narrow_Intrinsic`。
- **L150 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMExtendedType<0>], [IntrNoMem]>;`.
  **L150 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMExtendedType<0>], [IntrNoMem]>;`。
- **L151 EN**: Declares class `AdvSIMD_1VectorArg_Int_Across_Intrinsic`.
  **L151 CN**: 声明 class `AdvSIMD_1VectorArg_Int_Across_Intrinsic`。
- **L152 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L152 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。
- **L153 EN**: Declares class `AdvSIMD_1VectorArg_Float_Across_Intrinsic`.
  **L153 CN**: 声明 class `AdvSIMD_1VectorArg_Float_Across_Intrinsic`。
- **L154 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L154 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L156 EN**: Declares class `AdvSIMD_2IntArg_Intrinsic`.
  **L156 CN**: 声明 class `AdvSIMD_2IntArg_Intrinsic`。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L158 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L158 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L159 EN**: Declares class `AdvSIMD_2FloatArg_Intrinsic`.
  **L159 CN**: 声明 class `AdvSIMD_2FloatArg_Intrinsic`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`。

### Lines 161-192

````tablegen
                [IntrNoMem]>;
  class AdvSIMD_2VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;
  class AdvSIMD_2Arg_FloatCompare_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, LLVMMatchType<1>],
                [IntrNoMem]>;
  class AdvSIMD_2VectorArg_Long_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMTruncatedType<0>, LLVMTruncatedType<0>],
                [IntrNoMem]>;
  class AdvSIMD_2VectorArg_Narrow_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMExtendedType<0>, LLVMExtendedType<0>],
                [IntrNoMem]>;
  class AdvSIMD_2Arg_Scalar_Narrow_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                [LLVMExtendedType<0>, llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  class AdvSIMD_2VectorArg_Scalar_Wide_BySize_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMTruncatedType<0>],
                [IntrNoMem]>;
  class AdvSIMD_2VectorArg_Scalar_Wide_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMTruncatedType<0>, llvm_i32_ty],
                [IntrNoMem]>;
  class AdvSIMD_2VectorArg_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                [LLVMMatchType<0>, llvm_anyint_ty, llvm_i32_ty],
                [IntrNoMem]>;
  class AdvSIMD_3IntArg_Intrinsic
````
- **L161 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L161 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L162 EN**: Declares class `AdvSIMD_2VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L162 CN**: 声明 class `AdvSIMD_2VectorArg_Intrinsic<list<IntrinsicProperty>`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L164 EN**: Executes a call or declaration centered on `!listconcat`.
  **L164 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L165 EN**: Declares class `AdvSIMD_2Arg_FloatCompare_Intrinsic`.
  **L165 CN**: 声明 class `AdvSIMD_2Arg_FloatCompare_Intrinsic`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, LLVMMatchType<1>],`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, LLVMMatchType<1>],`。
- **L167 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L167 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L168 EN**: Declares class `AdvSIMD_2VectorArg_Long_Intrinsic`.
  **L168 CN**: 声明 class `AdvSIMD_2VectorArg_Long_Intrinsic`。
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMTruncatedType<0>, LLVMTruncatedType<0>],`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMTruncatedType<0>, LLVMTruncatedType<0>],`。
- **L171 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L171 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L172 EN**: Declares class `AdvSIMD_2VectorArg_Narrow_Intrinsic`.
  **L172 CN**: 声明 class `AdvSIMD_2VectorArg_Narrow_Intrinsic`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMExtendedType<0>, LLVMExtendedType<0>],`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMExtendedType<0>, LLVMExtendedType<0>],`。
- **L175 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L175 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L176 EN**: Declares class `AdvSIMD_2Arg_Scalar_Narrow_Intrinsic`.
  **L176 CN**: 声明 class `AdvSIMD_2Arg_Scalar_Narrow_Intrinsic`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMExtendedType<0>, llvm_i32_ty],`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMExtendedType<0>, llvm_i32_ty],`。
- **L179 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L179 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L180 EN**: Declares class `AdvSIMD_2VectorArg_Scalar_Wide_BySize_Intrinsic`.
  **L180 CN**: 声明 class `AdvSIMD_2VectorArg_Scalar_Wide_BySize_Intrinsic`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMTruncatedType<0>],`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMTruncatedType<0>],`。
- **L183 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L183 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L184 EN**: Declares class `AdvSIMD_2VectorArg_Scalar_Wide_Intrinsic`.
  **L184 CN**: 声明 class `AdvSIMD_2VectorArg_Scalar_Wide_Intrinsic`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMTruncatedType<0>, llvm_i32_ty],`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMTruncatedType<0>, llvm_i32_ty],`。
- **L187 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L187 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L188 EN**: Declares class `AdvSIMD_2VectorArg_Lane_Intrinsic`.
  **L188 CN**: 声明 class `AdvSIMD_2VectorArg_Lane_Intrinsic`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyint_ty, llvm_i32_ty],`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyint_ty, llvm_i32_ty],`。
- **L191 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L191 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L192 EN**: Declares class `AdvSIMD_3IntArg_Intrinsic`.
  **L192 CN**: 声明 class `AdvSIMD_3IntArg_Intrinsic`。

### Lines 193-224

````tablegen
    : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrNoMem]>;
  class AdvSIMD_3VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
               [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
               !listconcat(Attrs, [IntrNoMem])>;
  class AdvSIMD_3VectorArg_Scalar_Intrinsic
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
               [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty],
               [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  class AdvSIMD_CvtFxToFP_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyint_ty, llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  class AdvSIMD_CvtFPToFx_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  class AdvSIMD_1Arg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrNoMem]>;

  class AdvSIMD_Dot_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],
                [IntrNoMem]>;

  class AdvSIMD_FP16FML_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],
                [IntrNoMem]>;

  class AdvSIMD_MatMul_Intrinsic
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L195 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L195 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L196 EN**: Declares class `AdvSIMD_3VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L196 CN**: 声明 class `AdvSIMD_3VectorArg_Intrinsic<list<IntrinsicProperty>`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L199 EN**: Executes a call or declaration centered on `!listconcat`.
  **L199 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L200 EN**: Declares class `AdvSIMD_3VectorArg_Scalar_Intrinsic`.
  **L200 CN**: 声明 class `AdvSIMD_3VectorArg_Scalar_Intrinsic`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty],`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty],`。
- **L203 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L203 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L204 EN**: Declares class `AdvSIMD_CvtFxToFP_Intrinsic`.
  **L204 CN**: 声明 class `AdvSIMD_CvtFxToFP_Intrinsic`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyint_ty, llvm_i32_ty],`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyint_ty, llvm_i32_ty],`。
- **L206 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L206 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L207 EN**: Declares class `AdvSIMD_CvtFPToFx_Intrinsic`.
  **L207 CN**: 声明 class `AdvSIMD_CvtFPToFx_Intrinsic`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, llvm_i32_ty],`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, llvm_i32_ty],`。
- **L209 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L209 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Declares class `AdvSIMD_1Arg_Intrinsic`.
  **L211 CN**: 声明 class `AdvSIMD_1Arg_Intrinsic`。
- **L212 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrNoMem]>;`.
  **L212 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_any_ty], [LLVMMatchType<0>], [IntrNoMem]>;`。
- **L213 EN**: Blank line separating nearby declarations or logic blocks.
  **L213 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L214 EN**: Declares class `AdvSIMD_Dot_Intrinsic`.
  **L214 CN**: 声明 class `AdvSIMD_Dot_Intrinsic`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`。
- **L217 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L217 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L219 EN**: Declares class `AdvSIMD_FP16FML_Intrinsic`.
  **L219 CN**: 声明 class `AdvSIMD_FP16FML_Intrinsic`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`。
- **L222 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L222 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Declares class `AdvSIMD_MatMul_Intrinsic`.
  **L224 CN**: 声明 class `AdvSIMD_MatMul_Intrinsic`。

### Lines 225-256

````tablegen
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],
                [IntrNoMem]>;

  class AdvSIMD_BF16FML_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v4f32_ty],
                [llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],
                [IntrNoMem]>;
}

// Arithmetic ops

let TargetPrefix = "aarch64" in {
  // Vector Add Across Lanes
  def int_aarch64_neon_saddv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;
  def int_aarch64_neon_uaddv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;
  def int_aarch64_neon_faddv : AdvSIMD_1VectorArg_Float_Across_Intrinsic;

  // Vector Long Add Across Lanes
  def int_aarch64_neon_saddlv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;
  def int_aarch64_neon_uaddlv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;

  // Vector Halving Add
  def int_aarch64_neon_shadd : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_uhadd : AdvSIMD_2VectorArg_Intrinsic;

  // Vector Rounding Halving Add
  def int_aarch64_neon_srhadd : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_urhadd : AdvSIMD_2VectorArg_Intrinsic;

  // Vector Saturating Add
  def int_aarch64_neon_sqadd : AdvSIMD_2IntArg_Intrinsic;
````
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`。
- **L227 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L227 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Declares class `AdvSIMD_BF16FML_Intrinsic`.
  **L229 CN**: 声明 class `AdvSIMD_BF16FML_Intrinsic`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`。
- **L232 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L232 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L235 EN**: Comment explains nearby logic, invariants, or intent: `Arithmetic ops`.
  **L235 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Arithmetic ops`。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L237 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L237 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L238 EN**: Comment explains nearby logic, invariants, or intent: `Vector Add Across Lanes`.
  **L238 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Add Across Lanes`。
- **L239 EN**: Declares TableGen def `int_aarch64_neon_saddv`.
  **L239 CN**: 声明 TableGen def `int_aarch64_neon_saddv`。
- **L240 EN**: Declares TableGen def `int_aarch64_neon_uaddv`.
  **L240 CN**: 声明 TableGen def `int_aarch64_neon_uaddv`。
- **L241 EN**: Declares TableGen def `int_aarch64_neon_faddv`.
  **L241 CN**: 声明 TableGen def `int_aarch64_neon_faddv`。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L243 EN**: Comment explains nearby logic, invariants, or intent: `Vector Long Add Across Lanes`.
  **L243 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Long Add Across Lanes`。
- **L244 EN**: Declares TableGen def `int_aarch64_neon_saddlv`.
  **L244 CN**: 声明 TableGen def `int_aarch64_neon_saddlv`。
- **L245 EN**: Declares TableGen def `int_aarch64_neon_uaddlv`.
  **L245 CN**: 声明 TableGen def `int_aarch64_neon_uaddlv`。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L247 EN**: Comment explains nearby logic, invariants, or intent: `Vector Halving Add`.
  **L247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Halving Add`。
- **L248 EN**: Declares TableGen def `int_aarch64_neon_shadd`.
  **L248 CN**: 声明 TableGen def `int_aarch64_neon_shadd`。
- **L249 EN**: Declares TableGen def `int_aarch64_neon_uhadd`.
  **L249 CN**: 声明 TableGen def `int_aarch64_neon_uhadd`。
- **L250 EN**: Blank line separating nearby declarations or logic blocks.
  **L250 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L251 EN**: Comment explains nearby logic, invariants, or intent: `Vector Rounding Halving Add`.
  **L251 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Rounding Halving Add`。
- **L252 EN**: Declares TableGen def `int_aarch64_neon_srhadd`.
  **L252 CN**: 声明 TableGen def `int_aarch64_neon_srhadd`。
- **L253 EN**: Declares TableGen def `int_aarch64_neon_urhadd`.
  **L253 CN**: 声明 TableGen def `int_aarch64_neon_urhadd`。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L255 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Add`.
  **L255 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Add`。
- **L256 EN**: Declares TableGen def `int_aarch64_neon_sqadd`.
  **L256 CN**: 声明 TableGen def `int_aarch64_neon_sqadd`。

### Lines 257-288

````tablegen
  def int_aarch64_neon_suqadd : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_usqadd : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_uqadd : AdvSIMD_2IntArg_Intrinsic;

  // Vector Add High-Half
  // FIXME: this is a legacy intrinsic for aarch64_simd.h. Remove it when that
  // header is no longer supported.
  def int_aarch64_neon_addhn : AdvSIMD_2VectorArg_Narrow_Intrinsic;

  // Vector Rounding Add High-Half
  def int_aarch64_neon_raddhn : AdvSIMD_2VectorArg_Narrow_Intrinsic;

  // Vector Saturating Doubling Multiply High
  def int_aarch64_neon_sqdmulh : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_sqdmulh_lane : AdvSIMD_2VectorArg_Lane_Intrinsic;
  def int_aarch64_neon_sqdmulh_laneq : AdvSIMD_2VectorArg_Lane_Intrinsic;

  // Vector Saturating Rounding Doubling Multiply High
  def int_aarch64_neon_sqrdmulh : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_sqrdmulh_lane : AdvSIMD_2VectorArg_Lane_Intrinsic;
  def int_aarch64_neon_sqrdmulh_laneq : AdvSIMD_2VectorArg_Lane_Intrinsic;

  def int_aarch64_neon_sqrdmlah : AdvSIMD_3IntArg_Intrinsic;
  def int_aarch64_neon_sqrdmlsh : AdvSIMD_3IntArg_Intrinsic;

  let IntrProperties = [IntrNoMem, Commutative] in {
    // Vector Polynominal Multiply
    def int_aarch64_neon_pmul : AdvSIMD_2VectorArg_Intrinsic;

    // Vector Long Multiply
    def int_aarch64_neon_smull : AdvSIMD_2VectorArg_Long_Intrinsic;
    def int_aarch64_neon_umull : AdvSIMD_2VectorArg_Long_Intrinsic;
````
- **L257 EN**: Declares TableGen def `int_aarch64_neon_suqadd`.
  **L257 CN**: 声明 TableGen def `int_aarch64_neon_suqadd`。
- **L258 EN**: Declares TableGen def `int_aarch64_neon_usqadd`.
  **L258 CN**: 声明 TableGen def `int_aarch64_neon_usqadd`。
- **L259 EN**: Declares TableGen def `int_aarch64_neon_uqadd`.
  **L259 CN**: 声明 TableGen def `int_aarch64_neon_uqadd`。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L261 EN**: Comment explains nearby logic, invariants, or intent: `Vector Add High-Half`.
  **L261 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Add High-Half`。
- **L262 EN**: Comment records a pending task or caution: `FIXME: this is a legacy intrinsic for aarch64_simd.h. Remove it when that`.
  **L262 CN**: 注释记录了待办事项或注意点：`FIXME: this is a legacy intrinsic for aarch64_simd.h. Remove it when that`。
- **L263 EN**: Comment explains nearby logic, invariants, or intent: `header is no longer supported.`.
  **L263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`header is no longer supported.`。
- **L264 EN**: Declares TableGen def `int_aarch64_neon_addhn`.
  **L264 CN**: 声明 TableGen def `int_aarch64_neon_addhn`。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Comment explains nearby logic, invariants, or intent: `Vector Rounding Add High-Half`.
  **L266 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Rounding Add High-Half`。
- **L267 EN**: Declares TableGen def `int_aarch64_neon_raddhn`.
  **L267 CN**: 声明 TableGen def `int_aarch64_neon_raddhn`。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L269 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Doubling Multiply High`.
  **L269 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Doubling Multiply High`。
- **L270 EN**: Declares TableGen def `int_aarch64_neon_sqdmulh`.
  **L270 CN**: 声明 TableGen def `int_aarch64_neon_sqdmulh`。
- **L271 EN**: Declares TableGen def `int_aarch64_neon_sqdmulh_lane`.
  **L271 CN**: 声明 TableGen def `int_aarch64_neon_sqdmulh_lane`。
- **L272 EN**: Declares TableGen def `int_aarch64_neon_sqdmulh_laneq`.
  **L272 CN**: 声明 TableGen def `int_aarch64_neon_sqdmulh_laneq`。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L274 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Rounding Doubling Multiply High`.
  **L274 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Rounding Doubling Multiply High`。
- **L275 EN**: Declares TableGen def `int_aarch64_neon_sqrdmulh`.
  **L275 CN**: 声明 TableGen def `int_aarch64_neon_sqrdmulh`。
- **L276 EN**: Declares TableGen def `int_aarch64_neon_sqrdmulh_lane`.
  **L276 CN**: 声明 TableGen def `int_aarch64_neon_sqrdmulh_lane`。
- **L277 EN**: Declares TableGen def `int_aarch64_neon_sqrdmulh_laneq`.
  **L277 CN**: 声明 TableGen def `int_aarch64_neon_sqrdmulh_laneq`。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L279 EN**: Declares TableGen def `int_aarch64_neon_sqrdmlah`.
  **L279 CN**: 声明 TableGen def `int_aarch64_neon_sqrdmlah`。
- **L280 EN**: Declares TableGen def `int_aarch64_neon_sqrdmlsh`.
  **L280 CN**: 声明 TableGen def `int_aarch64_neon_sqrdmlsh`。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L282 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L282 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L283 EN**: Comment explains nearby logic, invariants, or intent: `Vector Polynominal Multiply`.
  **L283 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Polynominal Multiply`。
- **L284 EN**: Declares TableGen def `int_aarch64_neon_pmul`.
  **L284 CN**: 声明 TableGen def `int_aarch64_neon_pmul`。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L286 EN**: Comment explains nearby logic, invariants, or intent: `Vector Long Multiply`.
  **L286 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Long Multiply`。
- **L287 EN**: Declares TableGen def `int_aarch64_neon_smull`.
  **L287 CN**: 声明 TableGen def `int_aarch64_neon_smull`。
- **L288 EN**: Declares TableGen def `int_aarch64_neon_umull`.
  **L288 CN**: 声明 TableGen def `int_aarch64_neon_umull`。

### Lines 289-320

````tablegen
    def int_aarch64_neon_pmull : AdvSIMD_2VectorArg_Long_Intrinsic;

    // 64-bit polynomial multiply really returns an i128, which is not legal.
    // Fake it with a v16i8.
    def int_aarch64_neon_pmull64
      : DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i64_ty, llvm_i64_ty]>;

    // Vector Extending Multiply
    def int_aarch64_neon_fmulx : AdvSIMD_2FloatArg_Intrinsic;
  }

  // Vector Saturating Doubling Long Multiply
  def int_aarch64_neon_sqdmull : AdvSIMD_2VectorArg_Long_Intrinsic;
  def int_aarch64_neon_sqdmulls_scalar
    : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;

  // Vector Halving Subtract
  def int_aarch64_neon_shsub : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_uhsub : AdvSIMD_2VectorArg_Intrinsic;

  // Vector Saturating Subtract
  def int_aarch64_neon_sqsub : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_uqsub : AdvSIMD_2IntArg_Intrinsic;

  // Vector Subtract High-Half
  // FIXME: this is a legacy intrinsic for aarch64_simd.h. Remove it when that
  // header is no longer supported.
  def int_aarch64_neon_subhn : AdvSIMD_2VectorArg_Narrow_Intrinsic;

  // Vector Rounding Subtract High-Half
  def int_aarch64_neon_rsubhn : AdvSIMD_2VectorArg_Narrow_Intrinsic;

````
- **L289 EN**: Declares TableGen def `int_aarch64_neon_pmull`.
  **L289 CN**: 声明 TableGen def `int_aarch64_neon_pmull`。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L291 EN**: Comment explains nearby logic, invariants, or intent: `64-bit polynomial multiply really returns an i128, which is not legal.`.
  **L291 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64-bit polynomial multiply really returns an i128, which is not legal.`。
- **L292 EN**: Comment explains nearby logic, invariants, or intent: `Fake it with a v16i8.`.
  **L292 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Fake it with a v16i8.`。
- **L293 EN**: Declares TableGen def `int_aarch64_neon_pmull64`.
  **L293 CN**: 声明 TableGen def `int_aarch64_neon_pmull64`。
- **L294 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i64_ty, llvm_i64_ty]>;`.
  **L294 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i64_ty, llvm_i64_ty]>;`。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L296 EN**: Comment explains nearby logic, invariants, or intent: `Vector Extending Multiply`.
  **L296 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Extending Multiply`。
- **L297 EN**: Declares TableGen def `int_aarch64_neon_fmulx`.
  **L297 CN**: 声明 TableGen def `int_aarch64_neon_fmulx`。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L300 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Doubling Long Multiply`.
  **L300 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Doubling Long Multiply`。
- **L301 EN**: Declares TableGen def `int_aarch64_neon_sqdmull`.
  **L301 CN**: 声明 TableGen def `int_aarch64_neon_sqdmull`。
- **L302 EN**: Declares TableGen def `int_aarch64_neon_sqdmulls_scalar`.
  **L302 CN**: 声明 TableGen def `int_aarch64_neon_sqdmulls_scalar`。
- **L303 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L303 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L305 EN**: Comment explains nearby logic, invariants, or intent: `Vector Halving Subtract`.
  **L305 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Halving Subtract`。
- **L306 EN**: Declares TableGen def `int_aarch64_neon_shsub`.
  **L306 CN**: 声明 TableGen def `int_aarch64_neon_shsub`。
- **L307 EN**: Declares TableGen def `int_aarch64_neon_uhsub`.
  **L307 CN**: 声明 TableGen def `int_aarch64_neon_uhsub`。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L309 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Subtract`.
  **L309 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Subtract`。
- **L310 EN**: Declares TableGen def `int_aarch64_neon_sqsub`.
  **L310 CN**: 声明 TableGen def `int_aarch64_neon_sqsub`。
- **L311 EN**: Declares TableGen def `int_aarch64_neon_uqsub`.
  **L311 CN**: 声明 TableGen def `int_aarch64_neon_uqsub`。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L313 EN**: Comment explains nearby logic, invariants, or intent: `Vector Subtract High-Half`.
  **L313 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Subtract High-Half`。
- **L314 EN**: Comment records a pending task or caution: `FIXME: this is a legacy intrinsic for aarch64_simd.h. Remove it when that`.
  **L314 CN**: 注释记录了待办事项或注意点：`FIXME: this is a legacy intrinsic for aarch64_simd.h. Remove it when that`。
- **L315 EN**: Comment explains nearby logic, invariants, or intent: `header is no longer supported.`.
  **L315 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`header is no longer supported.`。
- **L316 EN**: Declares TableGen def `int_aarch64_neon_subhn`.
  **L316 CN**: 声明 TableGen def `int_aarch64_neon_subhn`。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L318 EN**: Comment explains nearby logic, invariants, or intent: `Vector Rounding Subtract High-Half`.
  **L318 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Rounding Subtract High-Half`。
- **L319 EN**: Declares TableGen def `int_aarch64_neon_rsubhn`.
  **L319 CN**: 声明 TableGen def `int_aarch64_neon_rsubhn`。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 321-352

````tablegen
  // Vector Compare Absolute Greater-than-or-equal
  def int_aarch64_neon_facge : AdvSIMD_2Arg_FloatCompare_Intrinsic;

  // Vector Compare Absolute Greater-than
  def int_aarch64_neon_facgt : AdvSIMD_2Arg_FloatCompare_Intrinsic;

  // Vector Absolute Difference
  def int_aarch64_neon_sabd : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_uabd : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_fabd : AdvSIMD_2VectorArg_Intrinsic;

  // Scalar Absolute Difference
  def int_aarch64_sisd_fabd : AdvSIMD_2Scalar_Float_Intrinsic;

  // Vector Max
  def int_aarch64_neon_smax : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_umax : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_fmax : AdvSIMD_2FloatArg_Intrinsic;
  def int_aarch64_neon_fmaxnmp : AdvSIMD_2VectorArg_Intrinsic;

  // Vector Max Across Lanes
  def int_aarch64_neon_smaxv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;
  def int_aarch64_neon_umaxv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;
  def int_aarch64_neon_fmaxv : AdvSIMD_1VectorArg_Float_Across_Intrinsic;
  def int_aarch64_neon_fmaxnmv : AdvSIMD_1VectorArg_Float_Across_Intrinsic;

  // Vector Min
  def int_aarch64_neon_smin : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_umin : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_fmin : AdvSIMD_2FloatArg_Intrinsic;
  def int_aarch64_neon_fminnmp : AdvSIMD_2VectorArg_Intrinsic;

````
- **L321 EN**: Comment explains nearby logic, invariants, or intent: `Vector Compare Absolute Greater-than-or-equal`.
  **L321 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Compare Absolute Greater-than-or-equal`。
- **L322 EN**: Declares TableGen def `int_aarch64_neon_facge`.
  **L322 CN**: 声明 TableGen def `int_aarch64_neon_facge`。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L324 EN**: Comment explains nearby logic, invariants, or intent: `Vector Compare Absolute Greater-than`.
  **L324 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Compare Absolute Greater-than`。
- **L325 EN**: Declares TableGen def `int_aarch64_neon_facgt`.
  **L325 CN**: 声明 TableGen def `int_aarch64_neon_facgt`。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L327 EN**: Comment explains nearby logic, invariants, or intent: `Vector Absolute Difference`.
  **L327 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Absolute Difference`。
- **L328 EN**: Declares TableGen def `int_aarch64_neon_sabd`.
  **L328 CN**: 声明 TableGen def `int_aarch64_neon_sabd`。
- **L329 EN**: Declares TableGen def `int_aarch64_neon_uabd`.
  **L329 CN**: 声明 TableGen def `int_aarch64_neon_uabd`。
- **L330 EN**: Declares TableGen def `int_aarch64_neon_fabd`.
  **L330 CN**: 声明 TableGen def `int_aarch64_neon_fabd`。
- **L331 EN**: Blank line separating nearby declarations or logic blocks.
  **L331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L332 EN**: Comment explains nearby logic, invariants, or intent: `Scalar Absolute Difference`.
  **L332 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar Absolute Difference`。
- **L333 EN**: Declares TableGen def `int_aarch64_sisd_fabd`.
  **L333 CN**: 声明 TableGen def `int_aarch64_sisd_fabd`。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L335 EN**: Comment explains nearby logic, invariants, or intent: `Vector Max`.
  **L335 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Max`。
- **L336 EN**: Declares TableGen def `int_aarch64_neon_smax`.
  **L336 CN**: 声明 TableGen def `int_aarch64_neon_smax`。
- **L337 EN**: Declares TableGen def `int_aarch64_neon_umax`.
  **L337 CN**: 声明 TableGen def `int_aarch64_neon_umax`。
- **L338 EN**: Declares TableGen def `int_aarch64_neon_fmax`.
  **L338 CN**: 声明 TableGen def `int_aarch64_neon_fmax`。
- **L339 EN**: Declares TableGen def `int_aarch64_neon_fmaxnmp`.
  **L339 CN**: 声明 TableGen def `int_aarch64_neon_fmaxnmp`。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L341 EN**: Comment explains nearby logic, invariants, or intent: `Vector Max Across Lanes`.
  **L341 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Max Across Lanes`。
- **L342 EN**: Declares TableGen def `int_aarch64_neon_smaxv`.
  **L342 CN**: 声明 TableGen def `int_aarch64_neon_smaxv`。
- **L343 EN**: Declares TableGen def `int_aarch64_neon_umaxv`.
  **L343 CN**: 声明 TableGen def `int_aarch64_neon_umaxv`。
- **L344 EN**: Declares TableGen def `int_aarch64_neon_fmaxv`.
  **L344 CN**: 声明 TableGen def `int_aarch64_neon_fmaxv`。
- **L345 EN**: Declares TableGen def `int_aarch64_neon_fmaxnmv`.
  **L345 CN**: 声明 TableGen def `int_aarch64_neon_fmaxnmv`。
- **L346 EN**: Blank line separating nearby declarations or logic blocks.
  **L346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L347 EN**: Comment explains nearby logic, invariants, or intent: `Vector Min`.
  **L347 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Min`。
- **L348 EN**: Declares TableGen def `int_aarch64_neon_smin`.
  **L348 CN**: 声明 TableGen def `int_aarch64_neon_smin`。
- **L349 EN**: Declares TableGen def `int_aarch64_neon_umin`.
  **L349 CN**: 声明 TableGen def `int_aarch64_neon_umin`。
- **L350 EN**: Declares TableGen def `int_aarch64_neon_fmin`.
  **L350 CN**: 声明 TableGen def `int_aarch64_neon_fmin`。
- **L351 EN**: Declares TableGen def `int_aarch64_neon_fminnmp`.
  **L351 CN**: 声明 TableGen def `int_aarch64_neon_fminnmp`。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 353-384

````tablegen
  // Vector Min/Max Number
  def int_aarch64_neon_fminnm : AdvSIMD_2FloatArg_Intrinsic;
  def int_aarch64_neon_fmaxnm : AdvSIMD_2FloatArg_Intrinsic;

  // Vector Min Across Lanes
  def int_aarch64_neon_sminv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;
  def int_aarch64_neon_uminv : AdvSIMD_1VectorArg_Int_Across_Intrinsic;
  def int_aarch64_neon_fminv : AdvSIMD_1VectorArg_Float_Across_Intrinsic;
  def int_aarch64_neon_fminnmv : AdvSIMD_1VectorArg_Float_Across_Intrinsic;

  // Pairwise Add
  def int_aarch64_neon_addp : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_faddp : AdvSIMD_2VectorArg_Intrinsic;

  // Long Pairwise Add
  // FIXME: In theory, we shouldn't need intrinsics for saddlp or
  // uaddlp, but tblgen's type inference currently can't handle the
  // pattern fragments this ends up generating.
  def int_aarch64_neon_saddlp : AdvSIMD_1VectorArg_Expand_Intrinsic;
  def int_aarch64_neon_uaddlp : AdvSIMD_1VectorArg_Expand_Intrinsic;

  // Folding Maximum
  def int_aarch64_neon_smaxp : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_umaxp : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_fmaxp : AdvSIMD_2VectorArg_Intrinsic;

  // Folding Minimum
  def int_aarch64_neon_sminp : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_uminp : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_fminp : AdvSIMD_2VectorArg_Intrinsic;

  // Reciprocal Estimate/Step
````
- **L353 EN**: Comment explains nearby logic, invariants, or intent: `Vector Min/Max Number`.
  **L353 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Min/Max Number`。
- **L354 EN**: Declares TableGen def `int_aarch64_neon_fminnm`.
  **L354 CN**: 声明 TableGen def `int_aarch64_neon_fminnm`。
- **L355 EN**: Declares TableGen def `int_aarch64_neon_fmaxnm`.
  **L355 CN**: 声明 TableGen def `int_aarch64_neon_fmaxnm`。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L357 EN**: Comment explains nearby logic, invariants, or intent: `Vector Min Across Lanes`.
  **L357 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Min Across Lanes`。
- **L358 EN**: Declares TableGen def `int_aarch64_neon_sminv`.
  **L358 CN**: 声明 TableGen def `int_aarch64_neon_sminv`。
- **L359 EN**: Declares TableGen def `int_aarch64_neon_uminv`.
  **L359 CN**: 声明 TableGen def `int_aarch64_neon_uminv`。
- **L360 EN**: Declares TableGen def `int_aarch64_neon_fminv`.
  **L360 CN**: 声明 TableGen def `int_aarch64_neon_fminv`。
- **L361 EN**: Declares TableGen def `int_aarch64_neon_fminnmv`.
  **L361 CN**: 声明 TableGen def `int_aarch64_neon_fminnmv`。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L363 EN**: Comment explains nearby logic, invariants, or intent: `Pairwise Add`.
  **L363 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Pairwise Add`。
- **L364 EN**: Declares TableGen def `int_aarch64_neon_addp`.
  **L364 CN**: 声明 TableGen def `int_aarch64_neon_addp`。
- **L365 EN**: Declares TableGen def `int_aarch64_neon_faddp`.
  **L365 CN**: 声明 TableGen def `int_aarch64_neon_faddp`。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L367 EN**: Comment explains nearby logic, invariants, or intent: `Long Pairwise Add`.
  **L367 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Long Pairwise Add`。
- **L368 EN**: Comment records a pending task or caution: `FIXME: In theory, we shouldn't need intrinsics for saddlp or`.
  **L368 CN**: 注释记录了待办事项或注意点：`FIXME: In theory, we shouldn't need intrinsics for saddlp or`。
- **L369 EN**: Comment explains nearby logic, invariants, or intent: `uaddlp, but tblgen's type inference currently can't handle the`.
  **L369 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`uaddlp, but tblgen's type inference currently can't handle the`。
- **L370 EN**: Comment explains nearby logic, invariants, or intent: `pattern fragments this ends up generating.`.
  **L370 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`pattern fragments this ends up generating.`。
- **L371 EN**: Declares TableGen def `int_aarch64_neon_saddlp`.
  **L371 CN**: 声明 TableGen def `int_aarch64_neon_saddlp`。
- **L372 EN**: Declares TableGen def `int_aarch64_neon_uaddlp`.
  **L372 CN**: 声明 TableGen def `int_aarch64_neon_uaddlp`。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L374 EN**: Comment explains nearby logic, invariants, or intent: `Folding Maximum`.
  **L374 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Folding Maximum`。
- **L375 EN**: Declares TableGen def `int_aarch64_neon_smaxp`.
  **L375 CN**: 声明 TableGen def `int_aarch64_neon_smaxp`。
- **L376 EN**: Declares TableGen def `int_aarch64_neon_umaxp`.
  **L376 CN**: 声明 TableGen def `int_aarch64_neon_umaxp`。
- **L377 EN**: Declares TableGen def `int_aarch64_neon_fmaxp`.
  **L377 CN**: 声明 TableGen def `int_aarch64_neon_fmaxp`。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L379 EN**: Comment explains nearby logic, invariants, or intent: `Folding Minimum`.
  **L379 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Folding Minimum`。
- **L380 EN**: Declares TableGen def `int_aarch64_neon_sminp`.
  **L380 CN**: 声明 TableGen def `int_aarch64_neon_sminp`。
- **L381 EN**: Declares TableGen def `int_aarch64_neon_uminp`.
  **L381 CN**: 声明 TableGen def `int_aarch64_neon_uminp`。
- **L382 EN**: Declares TableGen def `int_aarch64_neon_fminp`.
  **L382 CN**: 声明 TableGen def `int_aarch64_neon_fminp`。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L384 EN**: Comment explains nearby logic, invariants, or intent: `Reciprocal Estimate/Step`.
  **L384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reciprocal Estimate/Step`。

### Lines 385-416

````tablegen
  def int_aarch64_neon_frecps : AdvSIMD_2FloatArg_Intrinsic;
  def int_aarch64_neon_frsqrts : AdvSIMD_2FloatArg_Intrinsic;

  // Reciprocal Exponent
  def int_aarch64_neon_frecpx : AdvSIMD_1FloatArg_Intrinsic;

  // Vector Saturating Shift Left
  def int_aarch64_neon_sqshl : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_uqshl : AdvSIMD_2IntArg_Intrinsic;

  // Vector Rounding Shift Left
  def int_aarch64_neon_srshl : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_urshl : AdvSIMD_2IntArg_Intrinsic;

  // Vector Saturating Rounding Shift Left
  def int_aarch64_neon_sqrshl : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_uqrshl : AdvSIMD_2IntArg_Intrinsic;

  // Vector Signed->Unsigned Shift Left by Constant
  def int_aarch64_neon_sqshlu : AdvSIMD_2IntArg_Intrinsic;

  // Vector Signed->Unsigned Narrowing Saturating Shift Right by Constant
  def int_aarch64_neon_sqshrun : AdvSIMD_2Arg_Scalar_Narrow_Intrinsic;

  // Vector Signed->Unsigned Rounding Narrowing Saturating Shift Right by Const
  def int_aarch64_neon_sqrshrun : AdvSIMD_2Arg_Scalar_Narrow_Intrinsic;

  // Vector Narrowing Shift Right by Constant
  def int_aarch64_neon_sqshrn : AdvSIMD_2Arg_Scalar_Narrow_Intrinsic;
  def int_aarch64_neon_uqshrn : AdvSIMD_2Arg_Scalar_Narrow_Intrinsic;

  // Vector Rounding Narrowing Shift Right by Constant
````
- **L385 EN**: Declares TableGen def `int_aarch64_neon_frecps`.
  **L385 CN**: 声明 TableGen def `int_aarch64_neon_frecps`。
- **L386 EN**: Declares TableGen def `int_aarch64_neon_frsqrts`.
  **L386 CN**: 声明 TableGen def `int_aarch64_neon_frsqrts`。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L388 EN**: Comment explains nearby logic, invariants, or intent: `Reciprocal Exponent`.
  **L388 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reciprocal Exponent`。
- **L389 EN**: Declares TableGen def `int_aarch64_neon_frecpx`.
  **L389 CN**: 声明 TableGen def `int_aarch64_neon_frecpx`。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L391 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Shift Left`.
  **L391 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Shift Left`。
- **L392 EN**: Declares TableGen def `int_aarch64_neon_sqshl`.
  **L392 CN**: 声明 TableGen def `int_aarch64_neon_sqshl`。
- **L393 EN**: Declares TableGen def `int_aarch64_neon_uqshl`.
  **L393 CN**: 声明 TableGen def `int_aarch64_neon_uqshl`。
- **L394 EN**: Blank line separating nearby declarations or logic blocks.
  **L394 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L395 EN**: Comment explains nearby logic, invariants, or intent: `Vector Rounding Shift Left`.
  **L395 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Rounding Shift Left`。
- **L396 EN**: Declares TableGen def `int_aarch64_neon_srshl`.
  **L396 CN**: 声明 TableGen def `int_aarch64_neon_srshl`。
- **L397 EN**: Declares TableGen def `int_aarch64_neon_urshl`.
  **L397 CN**: 声明 TableGen def `int_aarch64_neon_urshl`。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L399 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Rounding Shift Left`.
  **L399 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Rounding Shift Left`。
- **L400 EN**: Declares TableGen def `int_aarch64_neon_sqrshl`.
  **L400 CN**: 声明 TableGen def `int_aarch64_neon_sqrshl`。
- **L401 EN**: Declares TableGen def `int_aarch64_neon_uqrshl`.
  **L401 CN**: 声明 TableGen def `int_aarch64_neon_uqrshl`。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L403 EN**: Comment explains nearby logic, invariants, or intent: `Vector Signed->Unsigned Shift Left by Constant`.
  **L403 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Signed->Unsigned Shift Left by Constant`。
- **L404 EN**: Declares TableGen def `int_aarch64_neon_sqshlu`.
  **L404 CN**: 声明 TableGen def `int_aarch64_neon_sqshlu`。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L406 EN**: Comment explains nearby logic, invariants, or intent: `Vector Signed->Unsigned Narrowing Saturating Shift Right by Constant`.
  **L406 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Signed->Unsigned Narrowing Saturating Shift Right by Constant`。
- **L407 EN**: Declares TableGen def `int_aarch64_neon_sqshrun`.
  **L407 CN**: 声明 TableGen def `int_aarch64_neon_sqshrun`。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L409 EN**: Comment explains nearby logic, invariants, or intent: `Vector Signed->Unsigned Rounding Narrowing Saturating Shift Right by Const`.
  **L409 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Signed->Unsigned Rounding Narrowing Saturating Shift Right by Const`。
- **L410 EN**: Declares TableGen def `int_aarch64_neon_sqrshrun`.
  **L410 CN**: 声明 TableGen def `int_aarch64_neon_sqrshrun`。
- **L411 EN**: Blank line separating nearby declarations or logic blocks.
  **L411 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L412 EN**: Comment explains nearby logic, invariants, or intent: `Vector Narrowing Shift Right by Constant`.
  **L412 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Narrowing Shift Right by Constant`。
- **L413 EN**: Declares TableGen def `int_aarch64_neon_sqshrn`.
  **L413 CN**: 声明 TableGen def `int_aarch64_neon_sqshrn`。
- **L414 EN**: Declares TableGen def `int_aarch64_neon_uqshrn`.
  **L414 CN**: 声明 TableGen def `int_aarch64_neon_uqshrn`。
- **L415 EN**: Blank line separating nearby declarations or logic blocks.
  **L415 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L416 EN**: Comment explains nearby logic, invariants, or intent: `Vector Rounding Narrowing Shift Right by Constant`.
  **L416 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Rounding Narrowing Shift Right by Constant`。

### Lines 417-448

````tablegen
  def int_aarch64_neon_rshrn : AdvSIMD_2Arg_Scalar_Narrow_Intrinsic;

  // Vector Rounding Narrowing Saturating Shift Right by Constant
  def int_aarch64_neon_sqrshrn : AdvSIMD_2Arg_Scalar_Narrow_Intrinsic;
  def int_aarch64_neon_uqrshrn : AdvSIMD_2Arg_Scalar_Narrow_Intrinsic;

  // Vector Shift Left
  def int_aarch64_neon_sshl : AdvSIMD_2IntArg_Intrinsic;
  def int_aarch64_neon_ushl : AdvSIMD_2IntArg_Intrinsic;

  // Vector Widening Shift Left by Constant
  def int_aarch64_neon_shll : AdvSIMD_2VectorArg_Scalar_Wide_BySize_Intrinsic;
  def int_aarch64_neon_sshll : AdvSIMD_2VectorArg_Scalar_Wide_Intrinsic;
  def int_aarch64_neon_ushll : AdvSIMD_2VectorArg_Scalar_Wide_Intrinsic;

  // Vector Shift Right by Constant and Insert
  def int_aarch64_neon_vsri : AdvSIMD_3VectorArg_Scalar_Intrinsic;

  // Vector Shift Left by Constant and Insert
  def int_aarch64_neon_vsli : AdvSIMD_3VectorArg_Scalar_Intrinsic;

  // Vector Saturating Narrow
  def int_aarch64_neon_scalar_sqxtn: AdvSIMD_1IntArg_Narrow_Intrinsic;
  def int_aarch64_neon_scalar_uqxtn : AdvSIMD_1IntArg_Narrow_Intrinsic;
  def int_aarch64_neon_sqxtn : AdvSIMD_1VectorArg_Narrow_Intrinsic;
  def int_aarch64_neon_uqxtn : AdvSIMD_1VectorArg_Narrow_Intrinsic;

  // Vector Saturating Extract and Unsigned Narrow
  def int_aarch64_neon_scalar_sqxtun : AdvSIMD_1IntArg_Narrow_Intrinsic;
  def int_aarch64_neon_sqxtun : AdvSIMD_1VectorArg_Narrow_Intrinsic;

  // Vector Absolute Value
````
- **L417 EN**: Declares TableGen def `int_aarch64_neon_rshrn`.
  **L417 CN**: 声明 TableGen def `int_aarch64_neon_rshrn`。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L419 EN**: Comment explains nearby logic, invariants, or intent: `Vector Rounding Narrowing Saturating Shift Right by Constant`.
  **L419 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Rounding Narrowing Saturating Shift Right by Constant`。
- **L420 EN**: Declares TableGen def `int_aarch64_neon_sqrshrn`.
  **L420 CN**: 声明 TableGen def `int_aarch64_neon_sqrshrn`。
- **L421 EN**: Declares TableGen def `int_aarch64_neon_uqrshrn`.
  **L421 CN**: 声明 TableGen def `int_aarch64_neon_uqrshrn`。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L423 EN**: Comment explains nearby logic, invariants, or intent: `Vector Shift Left`.
  **L423 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Shift Left`。
- **L424 EN**: Declares TableGen def `int_aarch64_neon_sshl`.
  **L424 CN**: 声明 TableGen def `int_aarch64_neon_sshl`。
- **L425 EN**: Declares TableGen def `int_aarch64_neon_ushl`.
  **L425 CN**: 声明 TableGen def `int_aarch64_neon_ushl`。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L427 EN**: Comment explains nearby logic, invariants, or intent: `Vector Widening Shift Left by Constant`.
  **L427 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Widening Shift Left by Constant`。
- **L428 EN**: Declares TableGen def `int_aarch64_neon_shll`.
  **L428 CN**: 声明 TableGen def `int_aarch64_neon_shll`。
- **L429 EN**: Declares TableGen def `int_aarch64_neon_sshll`.
  **L429 CN**: 声明 TableGen def `int_aarch64_neon_sshll`。
- **L430 EN**: Declares TableGen def `int_aarch64_neon_ushll`.
  **L430 CN**: 声明 TableGen def `int_aarch64_neon_ushll`。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L432 EN**: Comment explains nearby logic, invariants, or intent: `Vector Shift Right by Constant and Insert`.
  **L432 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Shift Right by Constant and Insert`。
- **L433 EN**: Declares TableGen def `int_aarch64_neon_vsri`.
  **L433 CN**: 声明 TableGen def `int_aarch64_neon_vsri`。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L435 EN**: Comment explains nearby logic, invariants, or intent: `Vector Shift Left by Constant and Insert`.
  **L435 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Shift Left by Constant and Insert`。
- **L436 EN**: Declares TableGen def `int_aarch64_neon_vsli`.
  **L436 CN**: 声明 TableGen def `int_aarch64_neon_vsli`。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L438 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Narrow`.
  **L438 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Narrow`。
- **L439 EN**: Declares TableGen def `int_aarch64_neon_scalar_sqxtn`.
  **L439 CN**: 声明 TableGen def `int_aarch64_neon_scalar_sqxtn`。
- **L440 EN**: Declares TableGen def `int_aarch64_neon_scalar_uqxtn`.
  **L440 CN**: 声明 TableGen def `int_aarch64_neon_scalar_uqxtn`。
- **L441 EN**: Declares TableGen def `int_aarch64_neon_sqxtn`.
  **L441 CN**: 声明 TableGen def `int_aarch64_neon_sqxtn`。
- **L442 EN**: Declares TableGen def `int_aarch64_neon_uqxtn`.
  **L442 CN**: 声明 TableGen def `int_aarch64_neon_uqxtn`。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L444 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Extract and Unsigned Narrow`.
  **L444 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Extract and Unsigned Narrow`。
- **L445 EN**: Declares TableGen def `int_aarch64_neon_scalar_sqxtun`.
  **L445 CN**: 声明 TableGen def `int_aarch64_neon_scalar_sqxtun`。
- **L446 EN**: Declares TableGen def `int_aarch64_neon_sqxtun`.
  **L446 CN**: 声明 TableGen def `int_aarch64_neon_sqxtun`。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L448 EN**: Comment explains nearby logic, invariants, or intent: `Vector Absolute Value`.
  **L448 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Absolute Value`。

### Lines 449-480

````tablegen
  def int_aarch64_neon_abs : AdvSIMD_1Arg_Intrinsic;

  // Vector Saturating Absolute Value
  def int_aarch64_neon_sqabs : AdvSIMD_1IntArg_Intrinsic;

  // Vector Saturating Negation
  def int_aarch64_neon_sqneg : AdvSIMD_1IntArg_Intrinsic;

  // Vector Count Leading Sign Bits
  def int_aarch64_neon_cls : AdvSIMD_1VectorArg_Intrinsic;

  // Vector Reciprocal Estimate
  def int_aarch64_neon_urecpe : AdvSIMD_1VectorArg_Intrinsic;
  def int_aarch64_neon_frecpe : AdvSIMD_1FloatArg_Intrinsic;

  // Vector Square Root Estimate
  def int_aarch64_neon_ursqrte : AdvSIMD_1VectorArg_Intrinsic;
  def int_aarch64_neon_frsqrte : AdvSIMD_1FloatArg_Intrinsic;

  // Vector Conversions Between Half-Precision and Single-Precision.
  def int_aarch64_neon_vcvtfp2hf
    : DefaultAttrsIntrinsic<[llvm_v4i16_ty], [llvm_v4f32_ty], [IntrNoMem]>;
  def int_aarch64_neon_vcvthf2fp
    : DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i16_ty], [IntrNoMem]>;

  // Vector Conversions Between Floating-point and Fixed-point.
  def int_aarch64_neon_vcvtfp2fxs : AdvSIMD_CvtFPToFx_Intrinsic;
  def int_aarch64_neon_vcvtfp2fxu : AdvSIMD_CvtFPToFx_Intrinsic;
  def int_aarch64_neon_vcvtfxs2fp : AdvSIMD_CvtFxToFP_Intrinsic;
  def int_aarch64_neon_vcvtfxu2fp : AdvSIMD_CvtFxToFP_Intrinsic;

  // Vector FP->Int Conversions
````
- **L449 EN**: Declares TableGen def `int_aarch64_neon_abs`.
  **L449 CN**: 声明 TableGen def `int_aarch64_neon_abs`。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L451 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Absolute Value`.
  **L451 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Absolute Value`。
- **L452 EN**: Declares TableGen def `int_aarch64_neon_sqabs`.
  **L452 CN**: 声明 TableGen def `int_aarch64_neon_sqabs`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L454 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Negation`.
  **L454 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Negation`。
- **L455 EN**: Declares TableGen def `int_aarch64_neon_sqneg`.
  **L455 CN**: 声明 TableGen def `int_aarch64_neon_sqneg`。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L457 EN**: Comment explains nearby logic, invariants, or intent: `Vector Count Leading Sign Bits`.
  **L457 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Count Leading Sign Bits`。
- **L458 EN**: Declares TableGen def `int_aarch64_neon_cls`.
  **L458 CN**: 声明 TableGen def `int_aarch64_neon_cls`。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L460 EN**: Comment explains nearby logic, invariants, or intent: `Vector Reciprocal Estimate`.
  **L460 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Reciprocal Estimate`。
- **L461 EN**: Declares TableGen def `int_aarch64_neon_urecpe`.
  **L461 CN**: 声明 TableGen def `int_aarch64_neon_urecpe`。
- **L462 EN**: Declares TableGen def `int_aarch64_neon_frecpe`.
  **L462 CN**: 声明 TableGen def `int_aarch64_neon_frecpe`。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L464 EN**: Comment explains nearby logic, invariants, or intent: `Vector Square Root Estimate`.
  **L464 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Square Root Estimate`。
- **L465 EN**: Declares TableGen def `int_aarch64_neon_ursqrte`.
  **L465 CN**: 声明 TableGen def `int_aarch64_neon_ursqrte`。
- **L466 EN**: Declares TableGen def `int_aarch64_neon_frsqrte`.
  **L466 CN**: 声明 TableGen def `int_aarch64_neon_frsqrte`。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L468 EN**: Comment explains nearby logic, invariants, or intent: `Vector Conversions Between Half-Precision and Single-Precision.`.
  **L468 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Conversions Between Half-Precision and Single-Precision.`。
- **L469 EN**: Declares TableGen def `int_aarch64_neon_vcvtfp2hf`.
  **L469 CN**: 声明 TableGen def `int_aarch64_neon_vcvtfp2hf`。
- **L470 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v4i16_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L470 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v4i16_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L471 EN**: Declares TableGen def `int_aarch64_neon_vcvthf2fp`.
  **L471 CN**: 声明 TableGen def `int_aarch64_neon_vcvthf2fp`。
- **L472 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i16_ty], [IntrNoMem]>;`.
  **L472 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i16_ty], [IntrNoMem]>;`。
- **L473 EN**: Blank line separating nearby declarations or logic blocks.
  **L473 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L474 EN**: Comment explains nearby logic, invariants, or intent: `Vector Conversions Between Floating-point and Fixed-point.`.
  **L474 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Conversions Between Floating-point and Fixed-point.`。
- **L475 EN**: Declares TableGen def `int_aarch64_neon_vcvtfp2fxs`.
  **L475 CN**: 声明 TableGen def `int_aarch64_neon_vcvtfp2fxs`。
- **L476 EN**: Declares TableGen def `int_aarch64_neon_vcvtfp2fxu`.
  **L476 CN**: 声明 TableGen def `int_aarch64_neon_vcvtfp2fxu`。
- **L477 EN**: Declares TableGen def `int_aarch64_neon_vcvtfxs2fp`.
  **L477 CN**: 声明 TableGen def `int_aarch64_neon_vcvtfxs2fp`。
- **L478 EN**: Declares TableGen def `int_aarch64_neon_vcvtfxu2fp`.
  **L478 CN**: 声明 TableGen def `int_aarch64_neon_vcvtfxu2fp`。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L480 EN**: Comment explains nearby logic, invariants, or intent: `Vector FP->Int Conversions`.
  **L480 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector FP->Int Conversions`。

### Lines 481-512

````tablegen
  def int_aarch64_neon_fcvtas : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtau : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtms : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtmu : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtns : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtnu : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtps : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtpu : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtzs : AdvSIMD_FPToIntRounding_Intrinsic;
  def int_aarch64_neon_fcvtzu : AdvSIMD_FPToIntRounding_Intrinsic;

  // v8.5-A Vector FP Rounding
  def int_aarch64_neon_frint32x : AdvSIMD_1FloatArg_Intrinsic;
  def int_aarch64_neon_frint32z : AdvSIMD_1FloatArg_Intrinsic;
  def int_aarch64_neon_frint64x : AdvSIMD_1FloatArg_Intrinsic;
  def int_aarch64_neon_frint64z : AdvSIMD_1FloatArg_Intrinsic;

  // Scalar FP->Int conversions

  // Vector FP Inexact Narrowing
  def int_aarch64_neon_fcvtxn : AdvSIMD_1VectorArg_Expand_Intrinsic;

  // Scalar FP Inexact Narrowing
  def int_aarch64_sisd_fcvtxn : DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_double_ty],
                                        [IntrNoMem]>;

  // v8.2-A Dot Product
  def int_aarch64_neon_udot : AdvSIMD_Dot_Intrinsic;
  def int_aarch64_neon_sdot : AdvSIMD_Dot_Intrinsic;

  // v8.6-A Matrix Multiply Intrinsics
  def int_aarch64_neon_ummla : AdvSIMD_MatMul_Intrinsic;
````
- **L481 EN**: Declares TableGen def `int_aarch64_neon_fcvtas`.
  **L481 CN**: 声明 TableGen def `int_aarch64_neon_fcvtas`。
- **L482 EN**: Declares TableGen def `int_aarch64_neon_fcvtau`.
  **L482 CN**: 声明 TableGen def `int_aarch64_neon_fcvtau`。
- **L483 EN**: Declares TableGen def `int_aarch64_neon_fcvtms`.
  **L483 CN**: 声明 TableGen def `int_aarch64_neon_fcvtms`。
- **L484 EN**: Declares TableGen def `int_aarch64_neon_fcvtmu`.
  **L484 CN**: 声明 TableGen def `int_aarch64_neon_fcvtmu`。
- **L485 EN**: Declares TableGen def `int_aarch64_neon_fcvtns`.
  **L485 CN**: 声明 TableGen def `int_aarch64_neon_fcvtns`。
- **L486 EN**: Declares TableGen def `int_aarch64_neon_fcvtnu`.
  **L486 CN**: 声明 TableGen def `int_aarch64_neon_fcvtnu`。
- **L487 EN**: Declares TableGen def `int_aarch64_neon_fcvtps`.
  **L487 CN**: 声明 TableGen def `int_aarch64_neon_fcvtps`。
- **L488 EN**: Declares TableGen def `int_aarch64_neon_fcvtpu`.
  **L488 CN**: 声明 TableGen def `int_aarch64_neon_fcvtpu`。
- **L489 EN**: Declares TableGen def `int_aarch64_neon_fcvtzs`.
  **L489 CN**: 声明 TableGen def `int_aarch64_neon_fcvtzs`。
- **L490 EN**: Declares TableGen def `int_aarch64_neon_fcvtzu`.
  **L490 CN**: 声明 TableGen def `int_aarch64_neon_fcvtzu`。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L492 EN**: Comment explains nearby logic, invariants, or intent: `v8.5-A Vector FP Rounding`.
  **L492 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.5-A Vector FP Rounding`。
- **L493 EN**: Declares TableGen def `int_aarch64_neon_frint32x`.
  **L493 CN**: 声明 TableGen def `int_aarch64_neon_frint32x`。
- **L494 EN**: Declares TableGen def `int_aarch64_neon_frint32z`.
  **L494 CN**: 声明 TableGen def `int_aarch64_neon_frint32z`。
- **L495 EN**: Declares TableGen def `int_aarch64_neon_frint64x`.
  **L495 CN**: 声明 TableGen def `int_aarch64_neon_frint64x`。
- **L496 EN**: Declares TableGen def `int_aarch64_neon_frint64z`.
  **L496 CN**: 声明 TableGen def `int_aarch64_neon_frint64z`。
- **L497 EN**: Blank line separating nearby declarations or logic blocks.
  **L497 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L498 EN**: Comment explains nearby logic, invariants, or intent: `Scalar FP->Int conversions`.
  **L498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar FP->Int conversions`。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L500 EN**: Comment explains nearby logic, invariants, or intent: `Vector FP Inexact Narrowing`.
  **L500 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector FP Inexact Narrowing`。
- **L501 EN**: Declares TableGen def `int_aarch64_neon_fcvtxn`.
  **L501 CN**: 声明 TableGen def `int_aarch64_neon_fcvtxn`。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L503 EN**: Comment explains nearby logic, invariants, or intent: `Scalar FP Inexact Narrowing`.
  **L503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar FP Inexact Narrowing`。
- **L504 EN**: Declares TableGen def `int_aarch64_sisd_fcvtxn`.
  **L504 CN**: 声明 TableGen def `int_aarch64_sisd_fcvtxn`。
- **L505 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L505 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L507 EN**: Comment explains nearby logic, invariants, or intent: `v8.2-A Dot Product`.
  **L507 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.2-A Dot Product`。
- **L508 EN**: Declares TableGen def `int_aarch64_neon_udot`.
  **L508 CN**: 声明 TableGen def `int_aarch64_neon_udot`。
- **L509 EN**: Declares TableGen def `int_aarch64_neon_sdot`.
  **L509 CN**: 声明 TableGen def `int_aarch64_neon_sdot`。
- **L510 EN**: Blank line separating nearby declarations or logic blocks.
  **L510 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L511 EN**: Comment explains nearby logic, invariants, or intent: `v8.6-A Matrix Multiply Intrinsics`.
  **L511 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.6-A Matrix Multiply Intrinsics`。
- **L512 EN**: Declares TableGen def `int_aarch64_neon_ummla`.
  **L512 CN**: 声明 TableGen def `int_aarch64_neon_ummla`。

### Lines 513-544

````tablegen
  def int_aarch64_neon_smmla : AdvSIMD_MatMul_Intrinsic;
  def int_aarch64_neon_usmmla : AdvSIMD_MatMul_Intrinsic;
  def int_aarch64_neon_fmmla : AdvSIMD_MatMul_Intrinsic;
  def int_aarch64_neon_usdot : AdvSIMD_Dot_Intrinsic;
  def int_aarch64_neon_bfdot : AdvSIMD_Dot_Intrinsic;
  def int_aarch64_neon_bfmmla
    : DefaultAttrsIntrinsic<[llvm_v4f32_ty],
                [llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],
                [IntrNoMem]>;
  def int_aarch64_neon_bfmlalb : AdvSIMD_BF16FML_Intrinsic;
  def int_aarch64_neon_bfmlalt : AdvSIMD_BF16FML_Intrinsic;

  // v8.2-A FP16 Fused Multiply-Add Long
  def int_aarch64_neon_fmlal : AdvSIMD_FP16FML_Intrinsic;
  def int_aarch64_neon_fmlsl : AdvSIMD_FP16FML_Intrinsic;
  def int_aarch64_neon_fmlal2 : AdvSIMD_FP16FML_Intrinsic;
  def int_aarch64_neon_fmlsl2 : AdvSIMD_FP16FML_Intrinsic;

  // v8.3-A Floating-point complex add
  def int_aarch64_neon_vcadd_rot90  : AdvSIMD_2VectorArg_Intrinsic;
  def int_aarch64_neon_vcadd_rot270 : AdvSIMD_2VectorArg_Intrinsic;

  def int_aarch64_neon_vcmla_rot0   : AdvSIMD_3VectorArg_Intrinsic;
  def int_aarch64_neon_vcmla_rot90  : AdvSIMD_3VectorArg_Intrinsic;
  def int_aarch64_neon_vcmla_rot180 : AdvSIMD_3VectorArg_Intrinsic;
  def int_aarch64_neon_vcmla_rot270 : AdvSIMD_3VectorArg_Intrinsic;

  // FP8 fscale
  def int_aarch64_neon_fp8_fscale : DefaultAttrsIntrinsic<
                                    [llvm_anyvector_ty],
                                    [LLVMMatchType<0>,
                                    LLVMVectorOfBitcastsToInt<0>],
````
- **L513 EN**: Declares TableGen def `int_aarch64_neon_smmla`.
  **L513 CN**: 声明 TableGen def `int_aarch64_neon_smmla`。
- **L514 EN**: Declares TableGen def `int_aarch64_neon_usmmla`.
  **L514 CN**: 声明 TableGen def `int_aarch64_neon_usmmla`。
- **L515 EN**: Declares TableGen def `int_aarch64_neon_fmmla`.
  **L515 CN**: 声明 TableGen def `int_aarch64_neon_fmmla`。
- **L516 EN**: Declares TableGen def `int_aarch64_neon_usdot`.
  **L516 CN**: 声明 TableGen def `int_aarch64_neon_usdot`。
- **L517 EN**: Declares TableGen def `int_aarch64_neon_bfdot`.
  **L517 CN**: 声明 TableGen def `int_aarch64_neon_bfdot`。
- **L518 EN**: Declares TableGen def `int_aarch64_neon_bfmmla`.
  **L518 CN**: 声明 TableGen def `int_aarch64_neon_bfmmla`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`。
- **L520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`.
  **L520 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`。
- **L521 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L521 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L522 EN**: Declares TableGen def `int_aarch64_neon_bfmlalb`.
  **L522 CN**: 声明 TableGen def `int_aarch64_neon_bfmlalb`。
- **L523 EN**: Declares TableGen def `int_aarch64_neon_bfmlalt`.
  **L523 CN**: 声明 TableGen def `int_aarch64_neon_bfmlalt`。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L525 EN**: Comment explains nearby logic, invariants, or intent: `v8.2-A FP16 Fused Multiply-Add Long`.
  **L525 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.2-A FP16 Fused Multiply-Add Long`。
- **L526 EN**: Declares TableGen def `int_aarch64_neon_fmlal`.
  **L526 CN**: 声明 TableGen def `int_aarch64_neon_fmlal`。
- **L527 EN**: Declares TableGen def `int_aarch64_neon_fmlsl`.
  **L527 CN**: 声明 TableGen def `int_aarch64_neon_fmlsl`。
- **L528 EN**: Declares TableGen def `int_aarch64_neon_fmlal2`.
  **L528 CN**: 声明 TableGen def `int_aarch64_neon_fmlal2`。
- **L529 EN**: Declares TableGen def `int_aarch64_neon_fmlsl2`.
  **L529 CN**: 声明 TableGen def `int_aarch64_neon_fmlsl2`。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L531 EN**: Comment explains nearby logic, invariants, or intent: `v8.3-A Floating-point complex add`.
  **L531 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.3-A Floating-point complex add`。
- **L532 EN**: Declares TableGen def `int_aarch64_neon_vcadd_rot90`.
  **L532 CN**: 声明 TableGen def `int_aarch64_neon_vcadd_rot90`。
- **L533 EN**: Declares TableGen def `int_aarch64_neon_vcadd_rot270`.
  **L533 CN**: 声明 TableGen def `int_aarch64_neon_vcadd_rot270`。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L535 EN**: Declares TableGen def `int_aarch64_neon_vcmla_rot0`.
  **L535 CN**: 声明 TableGen def `int_aarch64_neon_vcmla_rot0`。
- **L536 EN**: Declares TableGen def `int_aarch64_neon_vcmla_rot90`.
  **L536 CN**: 声明 TableGen def `int_aarch64_neon_vcmla_rot90`。
- **L537 EN**: Declares TableGen def `int_aarch64_neon_vcmla_rot180`.
  **L537 CN**: 声明 TableGen def `int_aarch64_neon_vcmla_rot180`。
- **L538 EN**: Declares TableGen def `int_aarch64_neon_vcmla_rot270`.
  **L538 CN**: 声明 TableGen def `int_aarch64_neon_vcmla_rot270`。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L540 EN**: Comment explains nearby logic, invariants, or intent: `FP8 fscale`.
  **L540 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP8 fscale`。
- **L541 EN**: Declares TableGen def `int_aarch64_neon_fp8_fscale`.
  **L541 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fscale`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>],`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>],`。

### Lines 545-576

````tablegen
                                    [IntrNoMem]>;
}

let TargetPrefix = "aarch64" in {
def int_aarch64_neon_vluti2_lane : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                       [llvm_anyvector_ty, llvm_v8i8_ty,
                                        llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_aarch64_neon_vluti2_laneq : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                       [llvm_anyvector_ty, llvm_v16i8_ty,
                                        llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_aarch64_neon_vluti4q_lane: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                       [LLVMMatchType<0>, llvm_v8i8_ty,
                                        llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<2>>]>;

def int_aarch64_neon_vluti4q_laneq: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                       [LLVMMatchType<0>, llvm_v16i8_ty,
                                        llvm_i32_ty],
                                       [IntrNoMem, ImmArg<ArgIndex<2>>]>;


def int_aarch64_neon_vluti4q_lane_x2:
    DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, LLVMMatchType<0>,
                           llvm_v8i8_ty, llvm_i32_ty],
                          [IntrNoMem, ImmArg<ArgIndex<3>>]>;

def int_aarch64_neon_vluti4q_laneq_x2:
````
- **L545 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L545 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L546 EN**: Closes the current lexical scope or compound statement.
  **L546 CN**: 结束当前词法作用域或复合语句块。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L548 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L548 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L549 EN**: Declares TableGen def `int_aarch64_neon_vluti2_lane`.
  **L549 CN**: 声明 TableGen def `int_aarch64_neon_vluti2_lane`。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_v8i8_ty,`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_v8i8_ty,`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L552 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L552 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L554 EN**: Declares TableGen def `int_aarch64_neon_vluti2_laneq`.
  **L554 CN**: 声明 TableGen def `int_aarch64_neon_vluti2_laneq`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_v16i8_ty,`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_v16i8_ty,`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L557 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L557 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L558 EN**: Blank line separating nearby declarations or logic blocks.
  **L558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L559 EN**: Declares TableGen def `int_aarch64_neon_vluti4q_lane`.
  **L559 CN**: 声明 TableGen def `int_aarch64_neon_vluti4q_lane`。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_v8i8_ty,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_v8i8_ty,`。
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L562 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L562 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L563 EN**: Blank line separating nearby declarations or logic blocks.
  **L563 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L564 EN**: Declares TableGen def `int_aarch64_neon_vluti4q_laneq`.
  **L564 CN**: 声明 TableGen def `int_aarch64_neon_vluti4q_laneq`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_v16i8_ty,`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_v16i8_ty,`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L567 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L567 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L570 EN**: Declares TableGen def `int_aarch64_neon_vluti4q_lane_x2`.
  **L570 CN**: 声明 TableGen def `int_aarch64_neon_vluti4q_lane_x2`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v8i8_ty, llvm_i32_ty],`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v8i8_ty, llvm_i32_ty],`。
- **L574 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L574 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L576 EN**: Declares TableGen def `int_aarch64_neon_vluti4q_laneq_x2`.
  **L576 CN**: 声明 TableGen def `int_aarch64_neon_vluti4q_laneq_x2`。

### Lines 577-608

````tablegen
    DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, LLVMMatchType<0>,
                           llvm_v16i8_ty, llvm_i32_ty],
                          [IntrNoMem, ImmArg<ArgIndex<3>>]>;
}

let TargetPrefix = "aarch64" in {  // All intrinsics start with "llvm.aarch64.".
  class AdvSIMD_2Vector2Index_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [llvm_anyvector_ty, llvm_i64_ty, LLVMMatchType<0>, llvm_i64_ty],
                [IntrNoMem]>;
}

// Vector element to element moves
def int_aarch64_neon_vcopy_lane: AdvSIMD_2Vector2Index_Intrinsic;

let TargetPrefix = "aarch64" in {  // All intrinsics start with "llvm.aarch64.".
  class AdvSIMD_1Vec_Load_Intrinsic
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyptr_ty],
                  [IntrReadMem, IntrArgMemOnly]>;
  class AdvSIMD_1Vec_Store_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, llvm_i64_ty, llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;

  class AdvSIMD_2Vec_Load_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMMatchType<0>, llvm_anyvector_ty],
                [llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;
  class AdvSIMD_2Vec_Load_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>],
                [LLVMMatchType<0>, llvm_anyvector_ty,
                 llvm_i64_ty, llvm_anyptr_ty],
````
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty, llvm_i32_ty],`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty, llvm_i32_ty],`。
- **L580 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L580 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L581 EN**: Closes the current lexical scope or compound statement.
  **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Blank line separating nearby declarations or logic blocks.
  **L582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L583 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L583 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L584 EN**: Declares class `AdvSIMD_2Vector2Index_Intrinsic`.
  **L584 CN**: 声明 class `AdvSIMD_2Vector2Index_Intrinsic`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i64_ty, LLVMMatchType<0>, llvm_i64_ty],`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i64_ty, LLVMMatchType<0>, llvm_i64_ty],`。
- **L587 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L587 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L588 EN**: Closes the current lexical scope or compound statement.
  **L588 CN**: 结束当前词法作用域或复合语句块。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L590 EN**: Comment explains nearby logic, invariants, or intent: `Vector element to element moves`.
  **L590 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector element to element moves`。
- **L591 EN**: Declares TableGen def `int_aarch64_neon_vcopy_lane`.
  **L591 CN**: 声明 TableGen def `int_aarch64_neon_vcopy_lane`。
- **L592 EN**: Blank line separating nearby declarations or logic blocks.
  **L592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L593 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L593 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L594 EN**: Declares class `AdvSIMD_1Vec_Load_Intrinsic`.
  **L594 CN**: 声明 class `AdvSIMD_1Vec_Load_Intrinsic`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyptr_ty],`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyptr_ty],`。
- **L596 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L596 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L597 EN**: Declares class `AdvSIMD_1Vec_Store_Lane_Intrinsic`.
  **L597 CN**: 声明 class `AdvSIMD_1Vec_Store_Lane_Intrinsic`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, llvm_i64_ty, llvm_anyptr_ty],`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, llvm_i64_ty, llvm_anyptr_ty],`。
- **L599 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`.
  **L599 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L601 EN**: Declares class `AdvSIMD_2Vec_Load_Intrinsic`.
  **L601 CN**: 声明 class `AdvSIMD_2Vec_Load_Intrinsic`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<0>, llvm_anyvector_ty],`.
  **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<0>, llvm_anyvector_ty],`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty],`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty],`。
- **L604 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L604 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L605 EN**: Declares class `AdvSIMD_2Vec_Load_Lane_Intrinsic`.
  **L605 CN**: 声明 class `AdvSIMD_2Vec_Load_Lane_Intrinsic`。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_anyptr_ty],`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_anyptr_ty],`。

### Lines 609-640

````tablegen
                [IntrReadMem, IntrArgMemOnly]>;
  class AdvSIMD_2Vec_Store_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,
                     llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;
  class AdvSIMD_2Vec_Store_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,
                 llvm_i64_ty, llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;

  class AdvSIMD_3Vec_Load_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],
                [llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;
  class AdvSIMD_3Vec_Load_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty,
                 llvm_i64_ty, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;
  class AdvSIMD_3Vec_Store_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,
                     LLVMMatchType<0>, llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;
  class AdvSIMD_3Vec_Store_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 llvm_i64_ty, llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;

  class AdvSIMD_4Vec_Load_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, llvm_anyvector_ty],
````
- **L609 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L609 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L610 EN**: Declares class `AdvSIMD_2Vec_Store_Intrinsic`.
  **L610 CN**: 声明 class `AdvSIMD_2Vec_Store_Intrinsic`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty],`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty],`。
- **L613 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`.
  **L613 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`。
- **L614 EN**: Declares class `AdvSIMD_2Vec_Store_Lane_Intrinsic`.
  **L614 CN**: 声明 class `AdvSIMD_2Vec_Store_Lane_Intrinsic`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_anyptr_ty],`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_anyptr_ty],`。
- **L617 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`.
  **L617 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L619 EN**: Declares class `AdvSIMD_3Vec_Load_Intrinsic`.
  **L619 CN**: 声明 class `AdvSIMD_3Vec_Load_Intrinsic`。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty],`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty],`。
- **L622 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L622 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L623 EN**: Declares class `AdvSIMD_3Vec_Load_Lane_Intrinsic`.
  **L623 CN**: 声明 class `AdvSIMD_3Vec_Load_Lane_Intrinsic`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_anyptr_ty],`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_anyptr_ty],`。
- **L627 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L627 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L628 EN**: Declares class `AdvSIMD_3Vec_Store_Intrinsic`.
  **L628 CN**: 声明 class `AdvSIMD_3Vec_Store_Intrinsic`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_anyptr_ty],`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_anyptr_ty],`。
- **L631 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`.
  **L631 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`。
- **L632 EN**: Declares class `AdvSIMD_3Vec_Store_Lane_Intrinsic`.
  **L632 CN**: 声明 class `AdvSIMD_3Vec_Store_Lane_Intrinsic`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty,`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty,`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_anyptr_ty],`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_anyptr_ty],`。
- **L636 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`.
  **L636 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L638 EN**: Declares class `AdvSIMD_4Vec_Load_Intrinsic`.
  **L638 CN**: 声明 class `AdvSIMD_4Vec_Load_Intrinsic`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_anyvector_ty],`.
  **L640 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_anyvector_ty],`。

### Lines 641-672

````tablegen
                [llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;
  class AdvSIMD_4Vec_Load_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, llvm_anyvector_ty,
                 llvm_i64_ty, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;
  class AdvSIMD_4Vec_Store_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;
  class AdvSIMD_4Vec_Store_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 llvm_i64_ty, llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<5>>]>;
}

// Memory ops

def int_aarch64_neon_ld1x2 : AdvSIMD_2Vec_Load_Intrinsic;
def int_aarch64_neon_ld1x3 : AdvSIMD_3Vec_Load_Intrinsic;
def int_aarch64_neon_ld1x4 : AdvSIMD_4Vec_Load_Intrinsic;

def int_aarch64_neon_st1x2 : AdvSIMD_2Vec_Store_Intrinsic;
def int_aarch64_neon_st1x3 : AdvSIMD_3Vec_Store_Intrinsic;
def int_aarch64_neon_st1x4 : AdvSIMD_4Vec_Store_Intrinsic;

def int_aarch64_neon_ld2 : AdvSIMD_2Vec_Load_Intrinsic;
````
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty],`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty],`。
- **L642 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L642 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L643 EN**: Declares class `AdvSIMD_4Vec_Load_Lane_Intrinsic`.
  **L643 CN**: 声明 class `AdvSIMD_4Vec_Load_Lane_Intrinsic`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_anyptr_ty],`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_anyptr_ty],`。
- **L649 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L649 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L650 EN**: Declares class `AdvSIMD_4Vec_Store_Intrinsic`.
  **L650 CN**: 声明 class `AdvSIMD_4Vec_Store_Intrinsic`。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty],`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty],`。
- **L654 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`.
  **L654 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`。
- **L655 EN**: Declares class `AdvSIMD_4Vec_Store_Lane_Intrinsic`.
  **L655 CN**: 声明 class `AdvSIMD_4Vec_Store_Lane_Intrinsic`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_anyptr_ty],`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_anyptr_ty],`。
- **L659 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<5>>]>;`.
  **L659 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<5>>]>;`。
- **L660 EN**: Closes the current lexical scope or compound statement.
  **L660 CN**: 结束当前词法作用域或复合语句块。
- **L661 EN**: Blank line separating nearby declarations or logic blocks.
  **L661 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L662 EN**: Comment explains nearby logic, invariants, or intent: `Memory ops`.
  **L662 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Memory ops`。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L664 EN**: Declares TableGen def `int_aarch64_neon_ld1x2`.
  **L664 CN**: 声明 TableGen def `int_aarch64_neon_ld1x2`。
- **L665 EN**: Declares TableGen def `int_aarch64_neon_ld1x3`.
  **L665 CN**: 声明 TableGen def `int_aarch64_neon_ld1x3`。
- **L666 EN**: Declares TableGen def `int_aarch64_neon_ld1x4`.
  **L666 CN**: 声明 TableGen def `int_aarch64_neon_ld1x4`。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L668 EN**: Declares TableGen def `int_aarch64_neon_st1x2`.
  **L668 CN**: 声明 TableGen def `int_aarch64_neon_st1x2`。
- **L669 EN**: Declares TableGen def `int_aarch64_neon_st1x3`.
  **L669 CN**: 声明 TableGen def `int_aarch64_neon_st1x3`。
- **L670 EN**: Declares TableGen def `int_aarch64_neon_st1x4`.
  **L670 CN**: 声明 TableGen def `int_aarch64_neon_st1x4`。
- **L671 EN**: Blank line separating nearby declarations or logic blocks.
  **L671 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L672 EN**: Declares TableGen def `int_aarch64_neon_ld2`.
  **L672 CN**: 声明 TableGen def `int_aarch64_neon_ld2`。

### Lines 673-704

````tablegen
def int_aarch64_neon_ld3 : AdvSIMD_3Vec_Load_Intrinsic;
def int_aarch64_neon_ld4 : AdvSIMD_4Vec_Load_Intrinsic;

def int_aarch64_neon_ld2lane : AdvSIMD_2Vec_Load_Lane_Intrinsic;
def int_aarch64_neon_ld3lane : AdvSIMD_3Vec_Load_Lane_Intrinsic;
def int_aarch64_neon_ld4lane : AdvSIMD_4Vec_Load_Lane_Intrinsic;

def int_aarch64_neon_ld2r : AdvSIMD_2Vec_Load_Intrinsic;
def int_aarch64_neon_ld3r : AdvSIMD_3Vec_Load_Intrinsic;
def int_aarch64_neon_ld4r : AdvSIMD_4Vec_Load_Intrinsic;

def int_aarch64_neon_st2  : AdvSIMD_2Vec_Store_Intrinsic;
def int_aarch64_neon_st3  : AdvSIMD_3Vec_Store_Intrinsic;
def int_aarch64_neon_st4  : AdvSIMD_4Vec_Store_Intrinsic;

def int_aarch64_neon_st2lane  : AdvSIMD_2Vec_Store_Lane_Intrinsic;
def int_aarch64_neon_st3lane  : AdvSIMD_3Vec_Store_Lane_Intrinsic;
def int_aarch64_neon_st4lane  : AdvSIMD_4Vec_Store_Lane_Intrinsic;

let TargetPrefix = "aarch64" in {  // All intrinsics start with "llvm.aarch64.".
  class AdvSIMD_Tbl1_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_v16i8_ty, LLVMMatchType<0>],
                [IntrNoMem]>;
  class AdvSIMD_Tbl2_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [llvm_v16i8_ty, llvm_v16i8_ty, LLVMMatchType<0>], [IntrNoMem]>;
  class AdvSIMD_Tbl3_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,
                 LLVMMatchType<0>],
                [IntrNoMem]>;
  class AdvSIMD_Tbl4_Intrinsic
````
- **L673 EN**: Declares TableGen def `int_aarch64_neon_ld3`.
  **L673 CN**: 声明 TableGen def `int_aarch64_neon_ld3`。
- **L674 EN**: Declares TableGen def `int_aarch64_neon_ld4`.
  **L674 CN**: 声明 TableGen def `int_aarch64_neon_ld4`。
- **L675 EN**: Blank line separating nearby declarations or logic blocks.
  **L675 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L676 EN**: Declares TableGen def `int_aarch64_neon_ld2lane`.
  **L676 CN**: 声明 TableGen def `int_aarch64_neon_ld2lane`。
- **L677 EN**: Declares TableGen def `int_aarch64_neon_ld3lane`.
  **L677 CN**: 声明 TableGen def `int_aarch64_neon_ld3lane`。
- **L678 EN**: Declares TableGen def `int_aarch64_neon_ld4lane`.
  **L678 CN**: 声明 TableGen def `int_aarch64_neon_ld4lane`。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L680 EN**: Declares TableGen def `int_aarch64_neon_ld2r`.
  **L680 CN**: 声明 TableGen def `int_aarch64_neon_ld2r`。
- **L681 EN**: Declares TableGen def `int_aarch64_neon_ld3r`.
  **L681 CN**: 声明 TableGen def `int_aarch64_neon_ld3r`。
- **L682 EN**: Declares TableGen def `int_aarch64_neon_ld4r`.
  **L682 CN**: 声明 TableGen def `int_aarch64_neon_ld4r`。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L684 EN**: Declares TableGen def `int_aarch64_neon_st2`.
  **L684 CN**: 声明 TableGen def `int_aarch64_neon_st2`。
- **L685 EN**: Declares TableGen def `int_aarch64_neon_st3`.
  **L685 CN**: 声明 TableGen def `int_aarch64_neon_st3`。
- **L686 EN**: Declares TableGen def `int_aarch64_neon_st4`.
  **L686 CN**: 声明 TableGen def `int_aarch64_neon_st4`。
- **L687 EN**: Blank line separating nearby declarations or logic blocks.
  **L687 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L688 EN**: Declares TableGen def `int_aarch64_neon_st2lane`.
  **L688 CN**: 声明 TableGen def `int_aarch64_neon_st2lane`。
- **L689 EN**: Declares TableGen def `int_aarch64_neon_st3lane`.
  **L689 CN**: 声明 TableGen def `int_aarch64_neon_st3lane`。
- **L690 EN**: Declares TableGen def `int_aarch64_neon_st4lane`.
  **L690 CN**: 声明 TableGen def `int_aarch64_neon_st4lane`。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L692 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L692 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L693 EN**: Declares class `AdvSIMD_Tbl1_Intrinsic`.
  **L693 CN**: 声明 class `AdvSIMD_Tbl1_Intrinsic`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_v16i8_ty, LLVMMatchType<0>],`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_v16i8_ty, LLVMMatchType<0>],`。
- **L695 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L695 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L696 EN**: Declares class `AdvSIMD_Tbl2_Intrinsic`.
  **L696 CN**: 声明 class `AdvSIMD_Tbl2_Intrinsic`。
- **L697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L697 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L698 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty, llvm_v16i8_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L698 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty, llvm_v16i8_ty, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L699 EN**: Declares class `AdvSIMD_Tbl3_Intrinsic`.
  **L699 CN**: 声明 class `AdvSIMD_Tbl3_Intrinsic`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L703 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L703 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L704 EN**: Declares class `AdvSIMD_Tbl4_Intrinsic`.
  **L704 CN**: 声明 class `AdvSIMD_Tbl4_Intrinsic`。

### Lines 705-736

````tablegen
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,
                 LLVMMatchType<0>],
                [IntrNoMem]>;

  class AdvSIMD_Tbx1_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, llvm_v16i8_ty, LLVMMatchType<0>],
                [IntrNoMem]>;
  class AdvSIMD_Tbx2_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,
                 LLVMMatchType<0>],
                [IntrNoMem]>;
  class AdvSIMD_Tbx3_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,
                 llvm_v16i8_ty, LLVMMatchType<0>],
                [IntrNoMem]>;
  class AdvSIMD_Tbx4_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,
                 llvm_v16i8_ty, llvm_v16i8_ty, LLVMMatchType<0>],
                [IntrNoMem]>;
}
def int_aarch64_neon_tbl1 : AdvSIMD_Tbl1_Intrinsic;
def int_aarch64_neon_tbl2 : AdvSIMD_Tbl2_Intrinsic;
def int_aarch64_neon_tbl3 : AdvSIMD_Tbl3_Intrinsic;
def int_aarch64_neon_tbl4 : AdvSIMD_Tbl4_Intrinsic;

def int_aarch64_neon_tbx1 : AdvSIMD_Tbx1_Intrinsic;
def int_aarch64_neon_tbx2 : AdvSIMD_Tbx2_Intrinsic;
````
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L707 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L708 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L708 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L709 EN**: Blank line separating nearby declarations or logic blocks.
  **L709 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L710 EN**: Declares class `AdvSIMD_Tbx1_Intrinsic`.
  **L710 CN**: 声明 class `AdvSIMD_Tbx1_Intrinsic`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_v16i8_ty, LLVMMatchType<0>],`.
  **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_v16i8_ty, LLVMMatchType<0>],`。
- **L713 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L713 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L714 EN**: Declares class `AdvSIMD_Tbx2_Intrinsic`.
  **L714 CN**: 声明 class `AdvSIMD_Tbx2_Intrinsic`。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L718 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L718 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L719 EN**: Declares class `AdvSIMD_Tbx3_Intrinsic`.
  **L719 CN**: 声明 class `AdvSIMD_Tbx3_Intrinsic`。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty, LLVMMatchType<0>],`.
  **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty, LLVMMatchType<0>],`。
- **L723 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L723 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L724 EN**: Declares class `AdvSIMD_Tbx4_Intrinsic`.
  **L724 CN**: 声明 class `AdvSIMD_Tbx4_Intrinsic`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty, llvm_v16i8_ty, LLVMMatchType<0>],`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty, llvm_v16i8_ty, LLVMMatchType<0>],`。
- **L728 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L728 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L729 EN**: Closes the current lexical scope or compound statement.
  **L729 CN**: 结束当前词法作用域或复合语句块。
- **L730 EN**: Declares TableGen def `int_aarch64_neon_tbl1`.
  **L730 CN**: 声明 TableGen def `int_aarch64_neon_tbl1`。
- **L731 EN**: Declares TableGen def `int_aarch64_neon_tbl2`.
  **L731 CN**: 声明 TableGen def `int_aarch64_neon_tbl2`。
- **L732 EN**: Declares TableGen def `int_aarch64_neon_tbl3`.
  **L732 CN**: 声明 TableGen def `int_aarch64_neon_tbl3`。
- **L733 EN**: Declares TableGen def `int_aarch64_neon_tbl4`.
  **L733 CN**: 声明 TableGen def `int_aarch64_neon_tbl4`。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L735 EN**: Declares TableGen def `int_aarch64_neon_tbx1`.
  **L735 CN**: 声明 TableGen def `int_aarch64_neon_tbx1`。
- **L736 EN**: Declares TableGen def `int_aarch64_neon_tbx2`.
  **L736 CN**: 声明 TableGen def `int_aarch64_neon_tbx2`。

### Lines 737-768

````tablegen
def int_aarch64_neon_tbx3 : AdvSIMD_Tbx3_Intrinsic;
def int_aarch64_neon_tbx4 : AdvSIMD_Tbx4_Intrinsic;

let TargetPrefix = "aarch64" in {
  class FPENV_Get_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects]>;
  class FPENV_Set_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i64_ty], [IntrNoMem, IntrHasSideEffects]>;
  class RNDR_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_i64_ty, llvm_i1_ty], [], [IntrNoMem, IntrHasSideEffects]>;
  class FPMR_Set_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i64_ty], [IntrWriteMem, IntrInaccessibleMemOnly]>;
}

// FP environment registers.
def int_aarch64_get_fpcr : FPENV_Get_Intrinsic;
def int_aarch64_set_fpcr : FPENV_Set_Intrinsic;
def int_aarch64_get_fpsr : FPENV_Get_Intrinsic;
def int_aarch64_set_fpsr : FPENV_Set_Intrinsic;
def int_aarch64_set_fpmr : FPMR_Set_Intrinsic;

// Armv8.5-A Random number generation intrinsics
def int_aarch64_rndr : RNDR_Intrinsic;
def int_aarch64_rndrrs : RNDR_Intrinsic;

let TargetPrefix = "aarch64" in {
  class Crypto_AES_DataKey_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;

  class Crypto_AES_Data_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;

````
- **L737 EN**: Declares TableGen def `int_aarch64_neon_tbx3`.
  **L737 CN**: 声明 TableGen def `int_aarch64_neon_tbx3`。
- **L738 EN**: Declares TableGen def `int_aarch64_neon_tbx4`.
  **L738 CN**: 声明 TableGen def `int_aarch64_neon_tbx4`。
- **L739 EN**: Blank line separating nearby declarations or logic blocks.
  **L739 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L740 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L740 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L741 EN**: Declares class `FPENV_Get_Intrinsic`.
  **L741 CN**: 声明 class `FPENV_Get_Intrinsic`。
- **L742 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L742 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L743 EN**: Declares class `FPENV_Set_Intrinsic`.
  **L743 CN**: 声明 class `FPENV_Set_Intrinsic`。
- **L744 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [llvm_i64_ty], [IntrNoMem, IntrHasSideEffects]>;`.
  **L744 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [llvm_i64_ty], [IntrNoMem, IntrHasSideEffects]>;`。
- **L745 EN**: Declares class `RNDR_Intrinsic`.
  **L745 CN**: 声明 class `RNDR_Intrinsic`。
- **L746 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i64_ty, llvm_i1_ty], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L746 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i64_ty, llvm_i1_ty], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L747 EN**: Declares class `FPMR_Set_Intrinsic`.
  **L747 CN**: 声明 class `FPMR_Set_Intrinsic`。
- **L748 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [llvm_i64_ty], [IntrWriteMem, IntrInaccessibleMemOnly]>;`.
  **L748 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [llvm_i64_ty], [IntrWriteMem, IntrInaccessibleMemOnly]>;`。
- **L749 EN**: Closes the current lexical scope or compound statement.
  **L749 CN**: 结束当前词法作用域或复合语句块。
- **L750 EN**: Blank line separating nearby declarations or logic blocks.
  **L750 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L751 EN**: Comment explains nearby logic, invariants, or intent: `FP environment registers.`.
  **L751 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP environment registers.`。
- **L752 EN**: Declares TableGen def `int_aarch64_get_fpcr`.
  **L752 CN**: 声明 TableGen def `int_aarch64_get_fpcr`。
- **L753 EN**: Declares TableGen def `int_aarch64_set_fpcr`.
  **L753 CN**: 声明 TableGen def `int_aarch64_set_fpcr`。
- **L754 EN**: Declares TableGen def `int_aarch64_get_fpsr`.
  **L754 CN**: 声明 TableGen def `int_aarch64_get_fpsr`。
- **L755 EN**: Declares TableGen def `int_aarch64_set_fpsr`.
  **L755 CN**: 声明 TableGen def `int_aarch64_set_fpsr`。
- **L756 EN**: Declares TableGen def `int_aarch64_set_fpmr`.
  **L756 CN**: 声明 TableGen def `int_aarch64_set_fpmr`。
- **L757 EN**: Blank line separating nearby declarations or logic blocks.
  **L757 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L758 EN**: Comment explains nearby logic, invariants, or intent: `Armv8.5-A Random number generation intrinsics`.
  **L758 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Armv8.5-A Random number generation intrinsics`。
- **L759 EN**: Declares TableGen def `int_aarch64_rndr`.
  **L759 CN**: 声明 TableGen def `int_aarch64_rndr`。
- **L760 EN**: Declares TableGen def `int_aarch64_rndrrs`.
  **L760 CN**: 声明 TableGen def `int_aarch64_rndrrs`。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L762 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L762 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L763 EN**: Declares class `Crypto_AES_DataKey_Intrinsic`.
  **L763 CN**: 声明 class `Crypto_AES_DataKey_Intrinsic`。
- **L764 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L764 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L766 EN**: Declares class `Crypto_AES_Data_Intrinsic`.
  **L766 CN**: 声明 class `Crypto_AES_Data_Intrinsic`。
- **L767 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L767 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L768 EN**: Blank line separating nearby declarations or logic blocks.
  **L768 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 769-800

````tablegen
  // SHA intrinsic taking 5 words of the hash (v4i32, i32) and 4 of the schedule
  // (v4i32).
  class Crypto_SHA_5Hash4Schedule_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],
                [IntrNoMem]>;

  // SHA intrinsic taking 5 words of the hash (v4i32, i32) and 4 of the schedule
  // (v4i32).
  class Crypto_SHA_1Hash_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;

  // SHA intrinsic taking 8 words of the schedule
  class Crypto_SHA_8Schedule_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;

  // SHA intrinsic taking 12 words of the schedule
  class Crypto_SHA_12Schedule_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
                [IntrNoMem]>;

  // SHA intrinsic taking 8 words of the hash and 4 of the schedule.
  class Crypto_SHA_8Hash4Schedule_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
                [IntrNoMem]>;

  // SHA512 intrinsic taking 2 arguments
  class Crypto_SHA512_2Arg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;

  // SHA512 intrinsic taking 3 Arguments
  class Crypto_SHA512_3Arg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
````
- **L769 EN**: Comment explains nearby logic, invariants, or intent: `SHA intrinsic taking 5 words of the hash (v4i32, i32) and 4 of the schedule`.
  **L769 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA intrinsic taking 5 words of the hash (v4i32, i32) and 4 of the schedule`。
- **L770 EN**: Comment explains nearby logic, invariants, or intent: `(v4i32).`.
  **L770 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(v4i32).`。
- **L771 EN**: Declares class `Crypto_SHA_5Hash4Schedule_Intrinsic`.
  **L771 CN**: 声明 class `Crypto_SHA_5Hash4Schedule_Intrinsic`。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`.
  **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`。
- **L773 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L773 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L775 EN**: Comment explains nearby logic, invariants, or intent: `SHA intrinsic taking 5 words of the hash (v4i32, i32) and 4 of the schedule`.
  **L775 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA intrinsic taking 5 words of the hash (v4i32, i32) and 4 of the schedule`。
- **L776 EN**: Comment explains nearby logic, invariants, or intent: `(v4i32).`.
  **L776 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(v4i32).`。
- **L777 EN**: Declares class `Crypto_SHA_1Hash_Intrinsic`.
  **L777 CN**: 声明 class `Crypto_SHA_1Hash_Intrinsic`。
- **L778 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L778 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L780 EN**: Comment explains nearby logic, invariants, or intent: `SHA intrinsic taking 8 words of the schedule`.
  **L780 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA intrinsic taking 8 words of the schedule`。
- **L781 EN**: Declares class `Crypto_SHA_8Schedule_Intrinsic`.
  **L781 CN**: 声明 class `Crypto_SHA_8Schedule_Intrinsic`。
- **L782 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L782 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L784 EN**: Comment explains nearby logic, invariants, or intent: `SHA intrinsic taking 12 words of the schedule`.
  **L784 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA intrinsic taking 12 words of the schedule`。
- **L785 EN**: Declares class `Crypto_SHA_12Schedule_Intrinsic`.
  **L785 CN**: 声明 class `Crypto_SHA_12Schedule_Intrinsic`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L787 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L787 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L789 EN**: Comment explains nearby logic, invariants, or intent: `SHA intrinsic taking 8 words of the hash and 4 of the schedule.`.
  **L789 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA intrinsic taking 8 words of the hash and 4 of the schedule.`。
- **L790 EN**: Declares class `Crypto_SHA_8Hash4Schedule_Intrinsic`.
  **L790 CN**: 声明 class `Crypto_SHA_8Hash4Schedule_Intrinsic`。
- **L791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L791 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L792 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L792 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L793 EN**: Blank line separating nearby declarations or logic blocks.
  **L793 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L794 EN**: Comment explains nearby logic, invariants, or intent: `SHA512 intrinsic taking 2 arguments`.
  **L794 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA512 intrinsic taking 2 arguments`。
- **L795 EN**: Declares class `Crypto_SHA512_2Arg_Intrinsic`.
  **L795 CN**: 声明 class `Crypto_SHA512_2Arg_Intrinsic`。
- **L796 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L796 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L798 EN**: Comment explains nearby logic, invariants, or intent: `SHA512 intrinsic taking 3 Arguments`.
  **L798 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA512 intrinsic taking 3 Arguments`。
- **L799 EN**: Declares class `Crypto_SHA512_3Arg_Intrinsic`.
  **L799 CN**: 声明 class `Crypto_SHA512_3Arg_Intrinsic`。
- **L800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L800 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。

### Lines 801-832

````tablegen
                [IntrNoMem]>;

  // SHA3 Intrinsics taking 3 arguments
  class Crypto_SHA3_3Arg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
               [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
               [IntrNoMem]>;

  // SHA3 Intrinsic taking 2 arguments
  class Crypto_SHA3_2Arg_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
               [IntrNoMem]>;

  // SHA3 Intrinsic taking 3 Arguments 1 immediate
  class Crypto_SHA3_2ArgImm_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i64_ty],
               [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  class Crypto_SM3_3Vector_Intrinsic
    : Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
                [IntrNoMem]>;

  class Crypto_SM3_3VectorIndexed_Intrinsic
    : Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty, llvm_i64_ty],
                [IntrNoMem, ImmArg<ArgIndex<3>>]>;

  class Crypto_SM4_2Vector_Intrinsic
    : Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
}

// AES
def int_aarch64_crypto_aese   : Crypto_AES_DataKey_Intrinsic;
````
- **L801 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L801 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L803 EN**: Comment explains nearby logic, invariants, or intent: `SHA3 Intrinsics taking 3 arguments`.
  **L803 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA3 Intrinsics taking 3 arguments`。
- **L804 EN**: Declares class `Crypto_SHA3_3Arg_Intrinsic`.
  **L804 CN**: 声明 class `Crypto_SHA3_3Arg_Intrinsic`。
- **L805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L805 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L807 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L807 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L809 EN**: Comment explains nearby logic, invariants, or intent: `SHA3 Intrinsic taking 2 arguments`.
  **L809 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA3 Intrinsic taking 2 arguments`。
- **L810 EN**: Declares class `Crypto_SHA3_2Arg_Intrinsic`.
  **L810 CN**: 声明 class `Crypto_SHA3_2Arg_Intrinsic`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L812 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L812 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L814 EN**: Comment explains nearby logic, invariants, or intent: `SHA3 Intrinsic taking 3 Arguments 1 immediate`.
  **L814 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA3 Intrinsic taking 3 Arguments 1 immediate`。
- **L815 EN**: Declares class `Crypto_SHA3_2ArgImm_Intrinsic`.
  **L815 CN**: 声明 class `Crypto_SHA3_2ArgImm_Intrinsic`。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i64_ty],`.
  **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i64_ty],`。
- **L817 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L817 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L819 EN**: Declares class `Crypto_SM3_3Vector_Intrinsic`.
  **L819 CN**: 声明 class `Crypto_SM3_3Vector_Intrinsic`。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L821 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L821 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L823 EN**: Declares class `Crypto_SM3_3VectorIndexed_Intrinsic`.
  **L823 CN**: 声明 class `Crypto_SM3_3VectorIndexed_Intrinsic`。
- **L824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty, llvm_i64_ty],`.
  **L824 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty, llvm_i64_ty],`。
- **L825 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L825 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L826 EN**: Blank line separating nearby declarations or logic blocks.
  **L826 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L827 EN**: Declares class `Crypto_SM4_2Vector_Intrinsic`.
  **L827 CN**: 声明 class `Crypto_SM4_2Vector_Intrinsic`。
- **L828 EN**: Executes a standalone statement or declaration: `: Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L828 CN**: 执行一条独立语句或声明：`: Intrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L829 EN**: Closes the current lexical scope or compound statement.
  **L829 CN**: 结束当前词法作用域或复合语句块。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L831 EN**: Comment explains nearby logic, invariants, or intent: `AES`.
  **L831 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AES`。
- **L832 EN**: Declares TableGen def `int_aarch64_crypto_aese`.
  **L832 CN**: 声明 TableGen def `int_aarch64_crypto_aese`。

### Lines 833-864

````tablegen
def int_aarch64_crypto_aesd   : Crypto_AES_DataKey_Intrinsic;
def int_aarch64_crypto_aesmc  : Crypto_AES_Data_Intrinsic;
def int_aarch64_crypto_aesimc : Crypto_AES_Data_Intrinsic;

// SHA1
def int_aarch64_crypto_sha1c  : Crypto_SHA_5Hash4Schedule_Intrinsic;
def int_aarch64_crypto_sha1p  : Crypto_SHA_5Hash4Schedule_Intrinsic;
def int_aarch64_crypto_sha1m  : Crypto_SHA_5Hash4Schedule_Intrinsic;
def int_aarch64_crypto_sha1h  : Crypto_SHA_1Hash_Intrinsic;

def int_aarch64_crypto_sha1su0 : Crypto_SHA_12Schedule_Intrinsic;
def int_aarch64_crypto_sha1su1 : Crypto_SHA_8Schedule_Intrinsic;

// SHA256
def int_aarch64_crypto_sha256h   : Crypto_SHA_8Hash4Schedule_Intrinsic;
def int_aarch64_crypto_sha256h2  : Crypto_SHA_8Hash4Schedule_Intrinsic;
def int_aarch64_crypto_sha256su0 : Crypto_SHA_8Schedule_Intrinsic;
def int_aarch64_crypto_sha256su1 : Crypto_SHA_12Schedule_Intrinsic;

//SHA3
def int_aarch64_crypto_eor3s : Crypto_SHA3_3Arg_Intrinsic;
def int_aarch64_crypto_eor3u : Crypto_SHA3_3Arg_Intrinsic;
def int_aarch64_crypto_bcaxs : Crypto_SHA3_3Arg_Intrinsic;
def int_aarch64_crypto_bcaxu : Crypto_SHA3_3Arg_Intrinsic;
def int_aarch64_crypto_rax1 : Crypto_SHA3_2Arg_Intrinsic;
def int_aarch64_crypto_xar : Crypto_SHA3_2ArgImm_Intrinsic;

// SHA512
def int_aarch64_crypto_sha512h : Crypto_SHA512_3Arg_Intrinsic;
def int_aarch64_crypto_sha512h2 : Crypto_SHA512_3Arg_Intrinsic;
def int_aarch64_crypto_sha512su0 : Crypto_SHA512_2Arg_Intrinsic;
def int_aarch64_crypto_sha512su1 : Crypto_SHA512_3Arg_Intrinsic;
````
- **L833 EN**: Declares TableGen def `int_aarch64_crypto_aesd`.
  **L833 CN**: 声明 TableGen def `int_aarch64_crypto_aesd`。
- **L834 EN**: Declares TableGen def `int_aarch64_crypto_aesmc`.
  **L834 CN**: 声明 TableGen def `int_aarch64_crypto_aesmc`。
- **L835 EN**: Declares TableGen def `int_aarch64_crypto_aesimc`.
  **L835 CN**: 声明 TableGen def `int_aarch64_crypto_aesimc`。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L837 EN**: Comment explains nearby logic, invariants, or intent: `SHA1`.
  **L837 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA1`。
- **L838 EN**: Declares TableGen def `int_aarch64_crypto_sha1c`.
  **L838 CN**: 声明 TableGen def `int_aarch64_crypto_sha1c`。
- **L839 EN**: Declares TableGen def `int_aarch64_crypto_sha1p`.
  **L839 CN**: 声明 TableGen def `int_aarch64_crypto_sha1p`。
- **L840 EN**: Declares TableGen def `int_aarch64_crypto_sha1m`.
  **L840 CN**: 声明 TableGen def `int_aarch64_crypto_sha1m`。
- **L841 EN**: Declares TableGen def `int_aarch64_crypto_sha1h`.
  **L841 CN**: 声明 TableGen def `int_aarch64_crypto_sha1h`。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L843 EN**: Declares TableGen def `int_aarch64_crypto_sha1su0`.
  **L843 CN**: 声明 TableGen def `int_aarch64_crypto_sha1su0`。
- **L844 EN**: Declares TableGen def `int_aarch64_crypto_sha1su1`.
  **L844 CN**: 声明 TableGen def `int_aarch64_crypto_sha1su1`。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L846 EN**: Comment explains nearby logic, invariants, or intent: `SHA256`.
  **L846 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA256`。
- **L847 EN**: Declares TableGen def `int_aarch64_crypto_sha256h`.
  **L847 CN**: 声明 TableGen def `int_aarch64_crypto_sha256h`。
- **L848 EN**: Declares TableGen def `int_aarch64_crypto_sha256h2`.
  **L848 CN**: 声明 TableGen def `int_aarch64_crypto_sha256h2`。
- **L849 EN**: Declares TableGen def `int_aarch64_crypto_sha256su0`.
  **L849 CN**: 声明 TableGen def `int_aarch64_crypto_sha256su0`。
- **L850 EN**: Declares TableGen def `int_aarch64_crypto_sha256su1`.
  **L850 CN**: 声明 TableGen def `int_aarch64_crypto_sha256su1`。
- **L851 EN**: Blank line separating nearby declarations or logic blocks.
  **L851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L852 EN**: Comment explains nearby logic, invariants, or intent: `SHA3`.
  **L852 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA3`。
- **L853 EN**: Declares TableGen def `int_aarch64_crypto_eor3s`.
  **L853 CN**: 声明 TableGen def `int_aarch64_crypto_eor3s`。
- **L854 EN**: Declares TableGen def `int_aarch64_crypto_eor3u`.
  **L854 CN**: 声明 TableGen def `int_aarch64_crypto_eor3u`。
- **L855 EN**: Declares TableGen def `int_aarch64_crypto_bcaxs`.
  **L855 CN**: 声明 TableGen def `int_aarch64_crypto_bcaxs`。
- **L856 EN**: Declares TableGen def `int_aarch64_crypto_bcaxu`.
  **L856 CN**: 声明 TableGen def `int_aarch64_crypto_bcaxu`。
- **L857 EN**: Declares TableGen def `int_aarch64_crypto_rax1`.
  **L857 CN**: 声明 TableGen def `int_aarch64_crypto_rax1`。
- **L858 EN**: Declares TableGen def `int_aarch64_crypto_xar`.
  **L858 CN**: 声明 TableGen def `int_aarch64_crypto_xar`。
- **L859 EN**: Blank line separating nearby declarations or logic blocks.
  **L859 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L860 EN**: Comment explains nearby logic, invariants, or intent: `SHA512`.
  **L860 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHA512`。
- **L861 EN**: Declares TableGen def `int_aarch64_crypto_sha512h`.
  **L861 CN**: 声明 TableGen def `int_aarch64_crypto_sha512h`。
- **L862 EN**: Declares TableGen def `int_aarch64_crypto_sha512h2`.
  **L862 CN**: 声明 TableGen def `int_aarch64_crypto_sha512h2`。
- **L863 EN**: Declares TableGen def `int_aarch64_crypto_sha512su0`.
  **L863 CN**: 声明 TableGen def `int_aarch64_crypto_sha512su0`。
- **L864 EN**: Declares TableGen def `int_aarch64_crypto_sha512su1`.
  **L864 CN**: 声明 TableGen def `int_aarch64_crypto_sha512su1`。

### Lines 865-896

````tablegen

//SM3 & SM4
def int_aarch64_crypto_sm3partw1 : Crypto_SM3_3Vector_Intrinsic;
def int_aarch64_crypto_sm3partw2 : Crypto_SM3_3Vector_Intrinsic;
def int_aarch64_crypto_sm3ss1    : Crypto_SM3_3Vector_Intrinsic;
def int_aarch64_crypto_sm3tt1a   : Crypto_SM3_3VectorIndexed_Intrinsic;
def int_aarch64_crypto_sm3tt1b   : Crypto_SM3_3VectorIndexed_Intrinsic;
def int_aarch64_crypto_sm3tt2a   : Crypto_SM3_3VectorIndexed_Intrinsic;
def int_aarch64_crypto_sm3tt2b   : Crypto_SM3_3VectorIndexed_Intrinsic;
def int_aarch64_crypto_sm4e      : Crypto_SM4_2Vector_Intrinsic;
def int_aarch64_crypto_sm4ekey   : Crypto_SM4_2Vector_Intrinsic;

//===----------------------------------------------------------------------===//
// CRC32

let TargetPrefix = "aarch64" in {

def int_aarch64_crc32b  : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem]>;
def int_aarch64_crc32cb : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem]>;
def int_aarch64_crc32h  : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem]>;
def int_aarch64_crc32ch : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem]>;
def int_aarch64_crc32w  : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem]>;
def int_aarch64_crc32cw : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
    [IntrNoMem]>;
def int_aarch64_crc32x  : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i64_ty],
    [IntrNoMem]>;
def int_aarch64_crc32cx : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i64_ty],
````
- **L865 EN**: Blank line separating nearby declarations or logic blocks.
  **L865 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L866 EN**: Comment explains nearby logic, invariants, or intent: `SM3 & SM4`.
  **L866 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SM3 & SM4`。
- **L867 EN**: Declares TableGen def `int_aarch64_crypto_sm3partw1`.
  **L867 CN**: 声明 TableGen def `int_aarch64_crypto_sm3partw1`。
- **L868 EN**: Declares TableGen def `int_aarch64_crypto_sm3partw2`.
  **L868 CN**: 声明 TableGen def `int_aarch64_crypto_sm3partw2`。
- **L869 EN**: Declares TableGen def `int_aarch64_crypto_sm3ss1`.
  **L869 CN**: 声明 TableGen def `int_aarch64_crypto_sm3ss1`。
- **L870 EN**: Declares TableGen def `int_aarch64_crypto_sm3tt1a`.
  **L870 CN**: 声明 TableGen def `int_aarch64_crypto_sm3tt1a`。
- **L871 EN**: Declares TableGen def `int_aarch64_crypto_sm3tt1b`.
  **L871 CN**: 声明 TableGen def `int_aarch64_crypto_sm3tt1b`。
- **L872 EN**: Declares TableGen def `int_aarch64_crypto_sm3tt2a`.
  **L872 CN**: 声明 TableGen def `int_aarch64_crypto_sm3tt2a`。
- **L873 EN**: Declares TableGen def `int_aarch64_crypto_sm3tt2b`.
  **L873 CN**: 声明 TableGen def `int_aarch64_crypto_sm3tt2b`。
- **L874 EN**: Declares TableGen def `int_aarch64_crypto_sm4e`.
  **L874 CN**: 声明 TableGen def `int_aarch64_crypto_sm4e`。
- **L875 EN**: Declares TableGen def `int_aarch64_crypto_sm4ekey`.
  **L875 CN**: 声明 TableGen def `int_aarch64_crypto_sm4ekey`。
- **L876 EN**: Blank line separating nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L877 EN**: Banner comment marking a file or section boundary.
  **L877 CN**: 横幅注释，用于标记文件或章节边界。
- **L878 EN**: Comment explains nearby logic, invariants, or intent: `CRC32`.
  **L878 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CRC32`。
- **L879 EN**: Blank line separating nearby declarations or logic blocks.
  **L879 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L880 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L880 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L881 EN**: Blank line separating nearby declarations or logic blocks.
  **L881 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L882 EN**: Declares TableGen def `int_aarch64_crc32b`.
  **L882 CN**: 声明 TableGen def `int_aarch64_crc32b`。
- **L883 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L883 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L884 EN**: Declares TableGen def `int_aarch64_crc32cb`.
  **L884 CN**: 声明 TableGen def `int_aarch64_crc32cb`。
- **L885 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L885 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L886 EN**: Declares TableGen def `int_aarch64_crc32h`.
  **L886 CN**: 声明 TableGen def `int_aarch64_crc32h`。
- **L887 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L887 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L888 EN**: Declares TableGen def `int_aarch64_crc32ch`.
  **L888 CN**: 声明 TableGen def `int_aarch64_crc32ch`。
- **L889 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L889 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L890 EN**: Declares TableGen def `int_aarch64_crc32w`.
  **L890 CN**: 声明 TableGen def `int_aarch64_crc32w`。
- **L891 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L891 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L892 EN**: Declares TableGen def `int_aarch64_crc32cw`.
  **L892 CN**: 声明 TableGen def `int_aarch64_crc32cw`。
- **L893 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L893 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L894 EN**: Declares TableGen def `int_aarch64_crc32x`.
  **L894 CN**: 声明 TableGen def `int_aarch64_crc32x`。
- **L895 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L895 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L896 EN**: Declares TableGen def `int_aarch64_crc32cx`.
  **L896 CN**: 声明 TableGen def `int_aarch64_crc32cx`。

### Lines 897-928

````tablegen
    [IntrNoMem]>;
}

//===----------------------------------------------------------------------===//
// Memory Tagging Extensions (MTE) Intrinsics
let TargetPrefix = "aarch64" in {
def int_aarch64_irg   : DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty],
    [IntrNoMem, IntrHasSideEffects]>;
def int_aarch64_addg  : DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty, llvm_i64_ty],
    [IntrNoMem]>;
def int_aarch64_gmi   : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_ptr_ty, llvm_i64_ty],
    [IntrNoMem]>;
def int_aarch64_ldg   : DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty, llvm_ptr_ty],
    [IntrReadMem]>;
def int_aarch64_stg   : DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_ptr_ty],
    [IntrWriteMem]>;
def int_aarch64_subp :  DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_ptr_ty, llvm_ptr_ty],
    [IntrNoMem]>;

// The following are codegen-only intrinsics for stack instrumentation.

// Generate a randomly tagged stack base pointer.
def int_aarch64_irg_sp   : DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_i64_ty],
    [IntrNoMem, IntrHasSideEffects]>;

// Transfer pointer tag with offset.
// ptr1 = tagp(ptr0, baseptr, tag_offset) returns a pointer where
// * address is the address in ptr0
// * tag is a function of (tag in baseptr, tag_offset).
// ** Beware, this is not the same function as implemented by the ADDG instruction!
//    Backend optimizations may change tag_offset; the only guarantee is that calls
//    to tagp with the same pair of (baseptr, tag_offset) will produce pointers
````
- **L897 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L897 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L898 EN**: Closes the current lexical scope or compound statement.
  **L898 CN**: 结束当前词法作用域或复合语句块。
- **L899 EN**: Blank line separating nearby declarations or logic blocks.
  **L899 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L900 EN**: Banner comment marking a file or section boundary.
  **L900 CN**: 横幅注释，用于标记文件或章节边界。
- **L901 EN**: Comment explains nearby logic, invariants, or intent: `Memory Tagging Extensions (MTE) Intrinsics`.
  **L901 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Memory Tagging Extensions (MTE) Intrinsics`。
- **L902 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L902 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L903 EN**: Declares TableGen def `int_aarch64_irg`.
  **L903 CN**: 声明 TableGen def `int_aarch64_irg`。
- **L904 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L904 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L905 EN**: Declares TableGen def `int_aarch64_addg`.
  **L905 CN**: 声明 TableGen def `int_aarch64_addg`。
- **L906 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L906 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L907 EN**: Declares TableGen def `int_aarch64_gmi`.
  **L907 CN**: 声明 TableGen def `int_aarch64_gmi`。
- **L908 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L908 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L909 EN**: Declares TableGen def `int_aarch64_ldg`.
  **L909 CN**: 声明 TableGen def `int_aarch64_ldg`。
- **L910 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L910 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L911 EN**: Declares TableGen def `int_aarch64_stg`.
  **L911 CN**: 声明 TableGen def `int_aarch64_stg`。
- **L912 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L912 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L913 EN**: Declares TableGen def `int_aarch64_subp`.
  **L913 CN**: 声明 TableGen def `int_aarch64_subp`。
- **L914 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L914 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L915 EN**: Blank line separating nearby declarations or logic blocks.
  **L915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L916 EN**: Comment explains nearby logic, invariants, or intent: `The following are codegen-only intrinsics for stack instrumentation.`.
  **L916 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following are codegen-only intrinsics for stack instrumentation.`。
- **L917 EN**: Blank line separating nearby declarations or logic blocks.
  **L917 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L918 EN**: Comment explains nearby logic, invariants, or intent: `Generate a randomly tagged stack base pointer.`.
  **L918 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generate a randomly tagged stack base pointer.`。
- **L919 EN**: Declares TableGen def `int_aarch64_irg_sp`.
  **L919 CN**: 声明 TableGen def `int_aarch64_irg_sp`。
- **L920 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L920 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L921 EN**: Blank line separating nearby declarations or logic blocks.
  **L921 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L922 EN**: Comment explains nearby logic, invariants, or intent: `Transfer pointer tag with offset.`.
  **L922 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Transfer pointer tag with offset.`。
- **L923 EN**: Comment explains nearby logic, invariants, or intent: `ptr1 = tagp(ptr0, baseptr, tag_offset) returns a pointer where`.
  **L923 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ptr1 = tagp(ptr0, baseptr, tag_offset) returns a pointer where`。
- **L924 EN**: Comment explains nearby logic, invariants, or intent: `* address is the address in ptr0`.
  **L924 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* address is the address in ptr0`。
- **L925 EN**: Comment explains nearby logic, invariants, or intent: `* tag is a function of (tag in baseptr, tag_offset).`.
  **L925 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* tag is a function of (tag in baseptr, tag_offset).`。
- **L926 EN**: Comment explains nearby logic, invariants, or intent: `** Beware, this is not the same function as implemented by the ADDG instruction!`.
  **L926 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`** Beware, this is not the same function as implemented by the ADDG instruction!`。
- **L927 EN**: Comment explains nearby logic, invariants, or intent: `Backend optimizations may change tag_offset; the only guarantee is that calls`.
  **L927 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Backend optimizations may change tag_offset; the only guarantee is that calls`。
- **L928 EN**: Comment explains nearby logic, invariants, or intent: `to tagp with the same pair of (baseptr, tag_offset) will produce pointers`.
  **L928 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to tagp with the same pair of (baseptr, tag_offset) will produce pointers`。

### Lines 929-960

````tablegen
//    with the same tag value, assuming the set of excluded tags has not changed.
// Address bits in baseptr and tag bits in ptr0 are ignored.
// When offset between ptr0 and baseptr is a compile time constant, this can be emitted as
//   ADDG ptr1, baseptr, (ptr0 - baseptr), tag_offset
// It is intended that ptr0 is an alloca address, and baseptr is the direct output of llvm.aarch64.irg.sp.
def int_aarch64_tagp : DefaultAttrsIntrinsic<[llvm_anyptr_ty], [LLVMMatchType<0>, llvm_ptr_ty, llvm_i64_ty],
    [IntrNoMem, ImmArg<ArgIndex<2>>]>;

// Update allocation tags for the memory range to match the tag in the pointer argument.
def int_aarch64_settag  : DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty],
    [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;

// Update allocation tags for the memory range to match the tag in the pointer argument,
// and set memory contents to zero.
def int_aarch64_settag_zero  : DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty],
    [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;

// Update allocation tags for 16-aligned, 16-sized memory region, and store a pair 8-byte values.
def int_aarch64_stgp  : DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty, llvm_i64_ty],
    [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;
}

//===----------------------------------------------------------------------===//
// Memory Operations (MOPS) Intrinsics
let TargetPrefix = "aarch64" in {
  // Sizes are chosen to correspond to the llvm.memset intrinsic: ptr, i8, i64
  def int_aarch64_mops_memset_tag : DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty, llvm_i8_ty, llvm_i64_ty],
      [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;
}

let TargetPrefix = "aarch64" in {
// Armv8.7-A load/store 64-byte intrinsics
````
- **L929 EN**: Comment explains nearby logic, invariants, or intent: `with the same tag value, assuming the set of excluded tags has not changed.`.
  **L929 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`with the same tag value, assuming the set of excluded tags has not changed.`。
- **L930 EN**: Comment explains nearby logic, invariants, or intent: `Address bits in baseptr and tag bits in ptr0 are ignored.`.
  **L930 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Address bits in baseptr and tag bits in ptr0 are ignored.`。
- **L931 EN**: Comment explains nearby logic, invariants, or intent: `When offset between ptr0 and baseptr is a compile time constant, this can be emitted as`.
  **L931 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`When offset between ptr0 and baseptr is a compile time constant, this can be emitted as`。
- **L932 EN**: Comment explains nearby logic, invariants, or intent: `ADDG ptr1, baseptr, (ptr0 - baseptr), tag_offset`.
  **L932 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ADDG ptr1, baseptr, (ptr0 - baseptr), tag_offset`。
- **L933 EN**: Comment explains nearby logic, invariants, or intent: `It is intended that ptr0 is an alloca address, and baseptr is the direct output of llvm.aarch64.irg.sp.`.
  **L933 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It is intended that ptr0 is an alloca address, and baseptr is the direct output of llvm.aarch64.irg.sp.`。
- **L934 EN**: Declares TableGen def `int_aarch64_tagp`.
  **L934 CN**: 声明 TableGen def `int_aarch64_tagp`。
- **L935 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L935 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L937 EN**: Comment explains nearby logic, invariants, or intent: `Update allocation tags for the memory range to match the tag in the pointer argument.`.
  **L937 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Update allocation tags for the memory range to match the tag in the pointer argument.`。
- **L938 EN**: Declares TableGen def `int_aarch64_settag`.
  **L938 CN**: 声明 TableGen def `int_aarch64_settag`。
- **L939 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`.
  **L939 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`。
- **L940 EN**: Blank line separating nearby declarations or logic blocks.
  **L940 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L941 EN**: Comment explains nearby logic, invariants, or intent: `Update allocation tags for the memory range to match the tag in the pointer argument,`.
  **L941 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Update allocation tags for the memory range to match the tag in the pointer argument,`。
- **L942 EN**: Comment explains nearby logic, invariants, or intent: `and set memory contents to zero.`.
  **L942 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and set memory contents to zero.`。
- **L943 EN**: Declares TableGen def `int_aarch64_settag_zero`.
  **L943 CN**: 声明 TableGen def `int_aarch64_settag_zero`。
- **L944 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`.
  **L944 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`。
- **L945 EN**: Blank line separating nearby declarations or logic blocks.
  **L945 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L946 EN**: Comment explains nearby logic, invariants, or intent: `Update allocation tags for 16-aligned, 16-sized memory region, and store a pair 8-byte values.`.
  **L946 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Update allocation tags for 16-aligned, 16-sized memory region, and store a pair 8-byte values.`。
- **L947 EN**: Declares TableGen def `int_aarch64_stgp`.
  **L947 CN**: 声明 TableGen def `int_aarch64_stgp`。
- **L948 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`.
  **L948 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Blank line separating nearby declarations or logic blocks.
  **L950 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L951 EN**: Banner comment marking a file or section boundary.
  **L951 CN**: 横幅注释，用于标记文件或章节边界。
- **L952 EN**: Comment explains nearby logic, invariants, or intent: `Memory Operations (MOPS) Intrinsics`.
  **L952 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Memory Operations (MOPS) Intrinsics`。
- **L953 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L953 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L954 EN**: Comment explains nearby logic, invariants, or intent: `Sizes are chosen to correspond to the llvm.memset intrinsic: ptr, i8, i64`.
  **L954 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sizes are chosen to correspond to the llvm.memset intrinsic: ptr, i8, i64`。
- **L955 EN**: Declares TableGen def `int_aarch64_mops_memset_tag`.
  **L955 CN**: 声明 TableGen def `int_aarch64_mops_memset_tag`。
- **L956 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`.
  **L956 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>, WriteOnly<ArgIndex<0>>]>;`。
- **L957 EN**: Closes the current lexical scope or compound statement.
  **L957 CN**: 结束当前词法作用域或复合语句块。
- **L958 EN**: Blank line separating nearby declarations or logic blocks.
  **L958 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L959 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L959 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L960 EN**: Comment explains nearby logic, invariants, or intent: `Armv8.7-A load/store 64-byte intrinsics`.
  **L960 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Armv8.7-A load/store 64-byte intrinsics`。

### Lines 961-992

````tablegen
defvar data512 = !listsplat(llvm_i64_ty, 8);
def int_aarch64_ld64b: Intrinsic<data512, [llvm_ptr_ty]>;
def int_aarch64_st64b: Intrinsic<[], !listconcat([llvm_ptr_ty], data512)>;
def int_aarch64_st64bv: Intrinsic<[llvm_i64_ty], !listconcat([llvm_ptr_ty], data512)>;
def int_aarch64_st64bv0: Intrinsic<[llvm_i64_ty], !listconcat([llvm_ptr_ty], data512)>;

  //
  // Neon FP8 intrinsics
  //

  // Conversions
  class AdvSIMD_FP8_1VectorArg_Long_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrReadMem, IntrInaccessibleMemOnly]>;

  def int_aarch64_neon_fp8_cvtl1   : AdvSIMD_FP8_1VectorArg_Long_Intrinsic;
  def int_aarch64_neon_fp8_cvtl2   : AdvSIMD_FP8_1VectorArg_Long_Intrinsic;

  def int_aarch64_neon_fp8_fcvtn
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [llvm_anyvector_ty,
                             LLVMMatchType<1>],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;
  def int_aarch64_neon_fp8_fcvtn2
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>,
                             llvm_anyvector_ty,
                             LLVMMatchType<1>],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  // Dot-product
  class AdvSIMD_FP8_DOT_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
````
- **L961 EN**: Initializes variable `data512` from the right-hand expression.
  **L961 CN**: 使用右侧表达式初始化变量 `data512`。
- **L962 EN**: Declares TableGen def `int_aarch64_ld64b`.
  **L962 CN**: 声明 TableGen def `int_aarch64_ld64b`。
- **L963 EN**: Declares TableGen def `int_aarch64_st64b`.
  **L963 CN**: 声明 TableGen def `int_aarch64_st64b`。
- **L964 EN**: Declares TableGen def `int_aarch64_st64bv`.
  **L964 CN**: 声明 TableGen def `int_aarch64_st64bv`。
- **L965 EN**: Declares TableGen def `int_aarch64_st64bv0`.
  **L965 CN**: 声明 TableGen def `int_aarch64_st64bv0`。
- **L966 EN**: Blank line separating nearby declarations or logic blocks.
  **L966 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L967 EN**: Separator comment used for visual grouping.
  **L967 CN**: 用于视觉分组的分隔注释。
- **L968 EN**: Comment explains nearby logic, invariants, or intent: `Neon FP8 intrinsics`.
  **L968 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Neon FP8 intrinsics`。
- **L969 EN**: Separator comment used for visual grouping.
  **L969 CN**: 用于视觉分组的分隔注释。
- **L970 EN**: Blank line separating nearby declarations or logic blocks.
  **L970 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L971 EN**: Comment explains nearby logic, invariants, or intent: `Conversions`.
  **L971 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Conversions`。
- **L972 EN**: Declares class `AdvSIMD_FP8_1VectorArg_Long_Intrinsic`.
  **L972 CN**: 声明 class `AdvSIMD_FP8_1VectorArg_Long_Intrinsic`。
- **L973 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L973 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L975 EN**: Declares TableGen def `int_aarch64_neon_fp8_cvtl1`.
  **L975 CN**: 声明 TableGen def `int_aarch64_neon_fp8_cvtl1`。
- **L976 EN**: Declares TableGen def `int_aarch64_neon_fp8_cvtl2`.
  **L976 CN**: 声明 TableGen def `int_aarch64_neon_fp8_cvtl2`。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L978 EN**: Declares TableGen def `int_aarch64_neon_fp8_fcvtn`.
  **L978 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fcvtn`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L980 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty,`.
  **L980 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty,`。
- **L981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L981 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L982 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L982 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L983 EN**: Declares TableGen def `int_aarch64_neon_fp8_fcvtn2`.
  **L983 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fcvtn2`。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L985 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L986 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L987 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L988 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L988 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L989 EN**: Blank line separating nearby declarations or logic blocks.
  **L989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L990 EN**: Comment explains nearby logic, invariants, or intent: `Dot-product`.
  **L990 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Dot-product`。
- **L991 EN**: Declares class `AdvSIMD_FP8_DOT_Intrinsic`.
  **L991 CN**: 声明 class `AdvSIMD_FP8_DOT_Intrinsic`。
- **L992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L992 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。

### Lines 993-1024

````tablegen
                            [LLVMMatchType<0>,
                             llvm_anyvector_ty,
                             LLVMMatchType<1>],
                             [IntrReadMem, IntrInaccessibleMemOnly]>;
  class AdvSIMD_FP8_DOT_LANE_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>,
                             llvm_anyvector_ty,
                             llvm_v16i8_ty,
                             llvm_i32_ty],
                             [IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;

  def int_aarch64_neon_fp8_fdot2 : AdvSIMD_FP8_DOT_Intrinsic;
  def int_aarch64_neon_fp8_fdot2_lane : AdvSIMD_FP8_DOT_LANE_Intrinsic;

  def int_aarch64_neon_fp8_fdot4 : AdvSIMD_FP8_DOT_Intrinsic;
  def int_aarch64_neon_fp8_fdot4_lane : AdvSIMD_FP8_DOT_LANE_Intrinsic;


// Fused multiply-add
  class AdvSIMD_FP8_FMLA_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>,
                             llvm_v16i8_ty,
                             llvm_v16i8_ty],
                             [IntrReadMem, IntrInaccessibleMemOnly]>;

  class AdvSIMD_FP8_FMLA_LANE_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>,
                             llvm_v16i8_ty,
                             llvm_v16i8_ty,
````
- **L993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L993 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L994 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L995 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L996 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L996 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L997 EN**: Declares class `AdvSIMD_FP8_DOT_LANE_Intrinsic`.
  **L997 CN**: 声明 class `AdvSIMD_FP8_DOT_LANE_Intrinsic`。
- **L998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L998 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty,`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1003 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`.
  **L1003 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1005 EN**: Declares TableGen def `int_aarch64_neon_fp8_fdot2`.
  **L1005 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fdot2`。
- **L1006 EN**: Declares TableGen def `int_aarch64_neon_fp8_fdot2_lane`.
  **L1006 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fdot2_lane`。
- **L1007 EN**: Blank line separating nearby declarations or logic blocks.
  **L1007 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1008 EN**: Declares TableGen def `int_aarch64_neon_fp8_fdot4`.
  **L1008 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fdot4`。
- **L1009 EN**: Declares TableGen def `int_aarch64_neon_fp8_fdot4_lane`.
  **L1009 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fdot4_lane`。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1012 EN**: Comment explains nearby logic, invariants, or intent: `Fused multiply-add`.
  **L1012 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Fused multiply-add`。
- **L1013 EN**: Declares class `AdvSIMD_FP8_FMLA_Intrinsic`.
  **L1013 CN**: 声明 class `AdvSIMD_FP8_FMLA_Intrinsic`。
- **L1014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1014 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1015 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty,`.
  **L1016 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty,`。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty],`.
  **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty],`。
- **L1018 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L1018 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L1019 EN**: Blank line separating nearby declarations or logic blocks.
  **L1019 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1020 EN**: Declares class `AdvSIMD_FP8_FMLA_LANE_Intrinsic`.
  **L1020 CN**: 声明 class `AdvSIMD_FP8_FMLA_LANE_Intrinsic`。
- **L1021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1021 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1022 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty,`.
  **L1023 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty,`。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty,`.
  **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty,`。

### Lines 1025-1056

````tablegen
                             llvm_i32_ty],
                             [IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;

  def int_aarch64_neon_fp8_fmlalb : AdvSIMD_FP8_FMLA_Intrinsic;
  def int_aarch64_neon_fp8_fmlalt : AdvSIMD_FP8_FMLA_Intrinsic;

  def int_aarch64_neon_fp8_fmlallbb : AdvSIMD_FP8_FMLA_Intrinsic;
  def int_aarch64_neon_fp8_fmlallbt : AdvSIMD_FP8_FMLA_Intrinsic;
  def int_aarch64_neon_fp8_fmlalltb : AdvSIMD_FP8_FMLA_Intrinsic;
  def int_aarch64_neon_fp8_fmlalltt : AdvSIMD_FP8_FMLA_Intrinsic;

  def int_aarch64_neon_fp8_fmlalb_lane : AdvSIMD_FP8_FMLA_LANE_Intrinsic;
  def int_aarch64_neon_fp8_fmlalt_lane : AdvSIMD_FP8_FMLA_LANE_Intrinsic;

  def int_aarch64_neon_fp8_fmlallbb_lane : AdvSIMD_FP8_FMLA_LANE_Intrinsic;
  def int_aarch64_neon_fp8_fmlallbt_lane : AdvSIMD_FP8_FMLA_LANE_Intrinsic;
  def int_aarch64_neon_fp8_fmlalltb_lane : AdvSIMD_FP8_FMLA_LANE_Intrinsic;
  def int_aarch64_neon_fp8_fmlalltt_lane : AdvSIMD_FP8_FMLA_LANE_Intrinsic;
}

def llvm_nxv1i1_ty  : LLVMType<nxv1i1>;
def llvm_nxv2i1_ty  : LLVMType<nxv2i1>;
def llvm_nxv4i1_ty  : LLVMType<nxv4i1>;
def llvm_nxv8i1_ty  : LLVMType<nxv8i1>;
def llvm_nxv16i1_ty : LLVMType<nxv16i1>;
def llvm_nxv16i8_ty : LLVMType<nxv16i8>;
def llvm_nxv4i32_ty : LLVMType<nxv4i32>;
def llvm_nxv2i64_ty : LLVMType<nxv2i64>;
def llvm_nxv8f16_ty : LLVMType<nxv8f16>;
def llvm_nxv8bf16_ty : LLVMType<nxv8bf16>;
def llvm_nxv4f32_ty : LLVMType<nxv4f32>;
def llvm_nxv2f64_ty : LLVMType<nxv2f64>;
````
- **L1025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1025 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1026 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`.
  **L1026 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`。
- **L1027 EN**: Blank line separating nearby declarations or logic blocks.
  **L1027 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1028 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalb`.
  **L1028 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalb`。
- **L1029 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalt`.
  **L1029 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalt`。
- **L1030 EN**: Blank line separating nearby declarations or logic blocks.
  **L1030 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1031 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlallbb`.
  **L1031 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlallbb`。
- **L1032 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlallbt`.
  **L1032 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlallbt`。
- **L1033 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalltb`.
  **L1033 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalltb`。
- **L1034 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalltt`.
  **L1034 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalltt`。
- **L1035 EN**: Blank line separating nearby declarations or logic blocks.
  **L1035 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1036 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalb_lane`.
  **L1036 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalb_lane`。
- **L1037 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalt_lane`.
  **L1037 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalt_lane`。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1039 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlallbb_lane`.
  **L1039 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlallbb_lane`。
- **L1040 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlallbt_lane`.
  **L1040 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlallbt_lane`。
- **L1041 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalltb_lane`.
  **L1041 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalltb_lane`。
- **L1042 EN**: Declares TableGen def `int_aarch64_neon_fp8_fmlalltt_lane`.
  **L1042 CN**: 声明 TableGen def `int_aarch64_neon_fp8_fmlalltt_lane`。
- **L1043 EN**: Closes the current lexical scope or compound statement.
  **L1043 CN**: 结束当前词法作用域或复合语句块。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1045 EN**: Declares TableGen def `llvm_nxv1i1_ty`.
  **L1045 CN**: 声明 TableGen def `llvm_nxv1i1_ty`。
- **L1046 EN**: Declares TableGen def `llvm_nxv2i1_ty`.
  **L1046 CN**: 声明 TableGen def `llvm_nxv2i1_ty`。
- **L1047 EN**: Declares TableGen def `llvm_nxv4i1_ty`.
  **L1047 CN**: 声明 TableGen def `llvm_nxv4i1_ty`。
- **L1048 EN**: Declares TableGen def `llvm_nxv8i1_ty`.
  **L1048 CN**: 声明 TableGen def `llvm_nxv8i1_ty`。
- **L1049 EN**: Declares TableGen def `llvm_nxv16i1_ty`.
  **L1049 CN**: 声明 TableGen def `llvm_nxv16i1_ty`。
- **L1050 EN**: Declares TableGen def `llvm_nxv16i8_ty`.
  **L1050 CN**: 声明 TableGen def `llvm_nxv16i8_ty`。
- **L1051 EN**: Declares TableGen def `llvm_nxv4i32_ty`.
  **L1051 CN**: 声明 TableGen def `llvm_nxv4i32_ty`。
- **L1052 EN**: Declares TableGen def `llvm_nxv2i64_ty`.
  **L1052 CN**: 声明 TableGen def `llvm_nxv2i64_ty`。
- **L1053 EN**: Declares TableGen def `llvm_nxv8f16_ty`.
  **L1053 CN**: 声明 TableGen def `llvm_nxv8f16_ty`。
- **L1054 EN**: Declares TableGen def `llvm_nxv8bf16_ty`.
  **L1054 CN**: 声明 TableGen def `llvm_nxv8bf16_ty`。
- **L1055 EN**: Declares TableGen def `llvm_nxv4f32_ty`.
  **L1055 CN**: 声明 TableGen def `llvm_nxv4f32_ty`。
- **L1056 EN**: Declares TableGen def `llvm_nxv2f64_ty`.
  **L1056 CN**: 声明 TableGen def `llvm_nxv2f64_ty`。

### Lines 1057-1088

````tablegen

let TargetPrefix = "aarch64" in {  // All intrinsics start with "llvm.aarch64.".

  class AdvSIMD_1Vec_PredLoad_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;

  class AdvSIMD_2Vec_PredLoad_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;

  class AdvSIMD_3Vec_PredLoad_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;

  class AdvSIMD_4Vec_PredLoad_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;

  class AdvSIMD_1Vec_PredLoad_WriteFFR_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                [IntrInaccessibleMemOrArgMemOnly]>;

  class AdvSIMD_1Vec_PredStore_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_anyvector_ty,
````
- **L1057 EN**: Blank line separating nearby declarations or logic blocks.
  **L1057 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1058 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1058 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1059 EN**: Blank line separating nearby declarations or logic blocks.
  **L1059 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1060 EN**: Declares class `AdvSIMD_1Vec_PredLoad_Intrinsic`.
  **L1060 CN**: 声明 class `AdvSIMD_1Vec_PredLoad_Intrinsic`。
- **L1061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1061 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1063 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1063 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1065 EN**: Declares class `AdvSIMD_2Vec_PredLoad_Intrinsic`.
  **L1065 CN**: 声明 class `AdvSIMD_2Vec_PredLoad_Intrinsic`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1068 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1068 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1070 EN**: Declares class `AdvSIMD_3Vec_PredLoad_Intrinsic`.
  **L1070 CN**: 声明 class `AdvSIMD_3Vec_PredLoad_Intrinsic`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1073 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1073 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1074 EN**: Blank line separating nearby declarations or logic blocks.
  **L1074 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1075 EN**: Declares class `AdvSIMD_4Vec_PredLoad_Intrinsic`.
  **L1075 CN**: 声明 class `AdvSIMD_4Vec_PredLoad_Intrinsic`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L1077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1077 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1078 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1079 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1079 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1080 EN**: Blank line separating nearby declarations or logic blocks.
  **L1080 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1081 EN**: Declares class `AdvSIMD_1Vec_PredLoad_WriteFFR_Intrinsic`.
  **L1081 CN**: 声明 class `AdvSIMD_1Vec_PredLoad_WriteFFR_Intrinsic`。
- **L1082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1082 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1083 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1084 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1084 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1085 EN**: Blank line separating nearby declarations or logic blocks.
  **L1085 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1086 EN**: Declares class `AdvSIMD_1Vec_PredStore_Intrinsic`.
  **L1086 CN**: 声明 class `AdvSIMD_1Vec_PredStore_Intrinsic`。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty,`.
  **L1088 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty,`。

### Lines 1089-1120

````tablegen
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                [IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;

  class AdvSIMD_2Vec_PredStore_Intrinsic
      : DefaultAttrsIntrinsic<[],
                  [llvm_anyvector_ty, LLVMMatchType<0>,
                   LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                  [IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;

  class AdvSIMD_3Vec_PredStore_Intrinsic
      : DefaultAttrsIntrinsic<[],
                  [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,
                   LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                  [IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;

  class AdvSIMD_4Vec_PredStore_Intrinsic
      : DefaultAttrsIntrinsic<[],
                  [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,
                   LLVMMatchType<0>,
                   LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],
                  [IntrArgMemOnly, NoCapture<ArgIndex<5>>]>;

  class AdvSIMD_SVE_Index_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMVectorElementType<0>,
                 LLVMVectorElementType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_Merged1VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
````
- **L1089 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1089 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1090 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`.
  **L1090 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<2>>]>;`。
- **L1091 EN**: Blank line separating nearby declarations or logic blocks.
  **L1091 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1092 EN**: Declares class `AdvSIMD_2Vec_PredStore_Intrinsic`.
  **L1092 CN**: 声明 class `AdvSIMD_2Vec_PredStore_Intrinsic`。
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1096 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`.
  **L1096 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<3>>]>;`。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1098 EN**: Declares class `AdvSIMD_3Vec_PredStore_Intrinsic`.
  **L1098 CN**: 声明 class `AdvSIMD_3Vec_PredStore_Intrinsic`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L1101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1101 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1102 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`.
  **L1102 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<4>>]>;`。
- **L1103 EN**: Blank line separating nearby declarations or logic blocks.
  **L1103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1104 EN**: Declares class `AdvSIMD_4Vec_PredStore_Intrinsic`.
  **L1104 CN**: 声明 class `AdvSIMD_4Vec_PredStore_Intrinsic`。
- **L1105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1105 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L1106 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L1107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1107 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`.
  **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_anyptr_ty],`。
- **L1109 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<5>>]>;`.
  **L1109 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<5>>]>;`。
- **L1110 EN**: Blank line separating nearby declarations or logic blocks.
  **L1110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1111 EN**: Declares class `AdvSIMD_SVE_Index_Intrinsic<list<IntrinsicProperty>`.
  **L1111 CN**: 声明 class `AdvSIMD_SVE_Index_Intrinsic<list<IntrinsicProperty>`。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMVectorElementType<0>,`.
  **L1113 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMVectorElementType<0>,`。
- **L1114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>],`.
  **L1114 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>],`。
- **L1115 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1115 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1116 EN**: Blank line separating nearby declarations or logic blocks.
  **L1116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1117 EN**: Declares class `AdvSIMD_Merged1VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L1117 CN**: 声明 class `AdvSIMD_Merged1VectorArg_Intrinsic<list<IntrinsicProperty>`。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1119 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。

### Lines 1121-1152

````tablegen
                 LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_2VectorArgIndexed_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<2>>])>;

  class AdvSIMD_3VectorArgIndexed_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<3>>])>;

  class AdvSIMD_Pred1VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_Pred2VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_Pred3VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
````
- **L1121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1121 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1122 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1122 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1123 EN**: Blank line separating nearby declarations or logic blocks.
  **L1123 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1124 EN**: Declares class `AdvSIMD_2VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`.
  **L1124 CN**: 声明 class `AdvSIMD_2VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`。
- **L1125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1125 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1126 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1127 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1128 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1129 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1129 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1131 EN**: Declares class `AdvSIMD_3VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`.
  **L1131 CN**: 声明 class `AdvSIMD_3VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1136 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1137 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1137 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1138 EN**: Blank line separating nearby declarations or logic blocks.
  **L1138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1139 EN**: Declares class `AdvSIMD_Pred1VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L1139 CN**: 声明 class `AdvSIMD_Pred1VectorArg_Intrinsic<list<IntrinsicProperty>`。
- **L1140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1140 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1143 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1143 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1144 EN**: Blank line separating nearby declarations or logic blocks.
  **L1144 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1145 EN**: Declares class `AdvSIMD_Pred2VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L1145 CN**: 声明 class `AdvSIMD_Pred2VectorArg_Intrinsic<list<IntrinsicProperty>`。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1148 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1150 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1150 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1151 EN**: Blank line separating nearby declarations or logic blocks.
  **L1151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1152 EN**: Declares class `AdvSIMD_Pred3VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L1152 CN**: 声明 class `AdvSIMD_Pred3VectorArg_Intrinsic<list<IntrinsicProperty>`。

### Lines 1153-1184

````tablegen
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_Compare_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyvector_ty,
                 LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_CompareWide_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyvector_ty,
                 llvm_nxv2i64_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_Saturating_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 llvm_i32_ty,
                 llvm_i32_ty],
````
- **L1153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1153 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1155 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1156 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1158 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1158 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1160 EN**: Declares class `AdvSIMD_SVE_Compare_Intrinsic<list<IntrinsicProperty>`.
  **L1160 CN**: 声明 class `AdvSIMD_SVE_Compare_Intrinsic<list<IntrinsicProperty>`。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L1162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1162 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1163 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1164 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1165 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1165 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1167 EN**: Declares class `AdvSIMD_SVE_CompareWide_Intrinsic<list<IntrinsicProperty>`.
  **L1167 CN**: 声明 class `AdvSIMD_SVE_CompareWide_Intrinsic<list<IntrinsicProperty>`。
- **L1168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L1168 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L1169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1169 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1170 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv2i64_ty],`.
  **L1171 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv2i64_ty],`。
- **L1172 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1172 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1174 EN**: Declares class `AdvSIMD_SVE_Saturating_Intrinsic<list<IntrinsicProperty>`.
  **L1174 CN**: 声明 class `AdvSIMD_SVE_Saturating_Intrinsic<list<IntrinsicProperty>`。
- **L1175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1175 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1176 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L1177 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L1178 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1178 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1179 EN**: Blank line separating nearby declarations or logic blocks.
  **L1179 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1180 EN**: Declares class `AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<list<IntrinsicProperty>`.
  **L1180 CN**: 声明 class `AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<list<IntrinsicProperty>`。
- **L1181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1181 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,`.
  **L1183 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,`。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。

### Lines 1185-1216

````tablegen
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>])>;

  class AdvSIMD_SVE_Saturating_N_Intrinsic<LLVMType T, list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[T],
                [T, llvm_anyvector_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<LLVMType T, list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[T],
                [T, llvm_i32_ty, llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>])>;

  class AdvSIMD_SVE_CNT_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],
                [LLVMVectorOfBitcastsToInt<0>,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyvector_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_ReduceWithInit_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMVectorElementType<0>,
                 llvm_anyvector_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_ShiftByImm_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<2>>])>;
````
- **L1185 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1185 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1187 EN**: Declares class `AdvSIMD_SVE_Saturating_N_Intrinsic<LLVMType`.
  **L1187 CN**: 声明 class `AdvSIMD_SVE_Saturating_N_Intrinsic<LLVMType`。
- **L1188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[T],`.
  **L1188 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[T],`。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[T, llvm_anyvector_ty],`.
  **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`[T, llvm_anyvector_ty],`。
- **L1190 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1190 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1191 EN**: Blank line separating nearby declarations or logic blocks.
  **L1191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1192 EN**: Declares class `AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<LLVMType`.
  **L1192 CN**: 声明 class `AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<LLVMType`。
- **L1193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[T],`.
  **L1193 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[T],`。
- **L1194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[T, llvm_i32_ty, llvm_i32_ty],`.
  **L1194 CN**: 继续一个多行参数列表、初始化器或聚合项：`[T, llvm_i32_ty, llvm_i32_ty],`。
- **L1195 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1195 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1196 EN**: Blank line separating nearby declarations or logic blocks.
  **L1196 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1197 EN**: Declares class `AdvSIMD_SVE_CNT_Intrinsic<list<IntrinsicProperty>`.
  **L1197 CN**: 声明 class `AdvSIMD_SVE_CNT_Intrinsic<list<IntrinsicProperty>`。
- **L1198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],`.
  **L1198 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMVectorOfBitcastsToInt<0>],`。
- **L1199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMVectorOfBitcastsToInt<0>,`.
  **L1199 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMVectorOfBitcastsToInt<0>,`。
- **L1200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1200 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1201 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1202 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1202 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1204 EN**: Declares class `AdvSIMD_SVE_ReduceWithInit_Intrinsic<list<IntrinsicProperty>`.
  **L1204 CN**: 声明 class `AdvSIMD_SVE_ReduceWithInit_Intrinsic<list<IntrinsicProperty>`。
- **L1205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`.
  **L1205 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`。
- **L1206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1206 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>,`.
  **L1207 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>,`。
- **L1208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1208 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1209 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1209 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1210 EN**: Blank line separating nearby declarations or logic blocks.
  **L1210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1211 EN**: Declares class `AdvSIMD_SVE_ShiftByImm_Intrinsic<list<IntrinsicProperty>`.
  **L1211 CN**: 声明 class `AdvSIMD_SVE_ShiftByImm_Intrinsic<list<IntrinsicProperty>`。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1213 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1214 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1215 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1216 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1216 CN**: 执行以 `!listconcat` 为核心的调用或声明。

### Lines 1217-1248

````tablegen

  class AdvSIMD_SVE_ShiftWide_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 llvm_nxv2i64_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_Unpack_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
               [LLVMSubdivide2VectorType<0>],
               !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_CADD_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<3>>]>;

  class AdvSIMD_SVE_CMLA_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<4>>]>;

  class AdvSIMD_SVE_CMLA_LANE_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
````
- **L1217 EN**: Blank line separating nearby declarations or logic blocks.
  **L1217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1218 EN**: Declares class `AdvSIMD_SVE_ShiftWide_Intrinsic<list<IntrinsicProperty>`.
  **L1218 CN**: 声明 class `AdvSIMD_SVE_ShiftWide_Intrinsic<list<IntrinsicProperty>`。
- **L1219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1219 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1220 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1221 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv2i64_ty],`.
  **L1222 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv2i64_ty],`。
- **L1223 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1223 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1225 EN**: Declares class `AdvSIMD_SVE_Unpack_Intrinsic<list<IntrinsicProperty>`.
  **L1225 CN**: 声明 class `AdvSIMD_SVE_Unpack_Intrinsic<list<IntrinsicProperty>`。
- **L1226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1226 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>],`.
  **L1227 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>],`。
- **L1228 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1228 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1229 EN**: Blank line separating nearby declarations or logic blocks.
  **L1229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1230 EN**: Declares class `AdvSIMD_SVE_CADD_Intrinsic`.
  **L1230 CN**: 声明 class `AdvSIMD_SVE_CADD_Intrinsic`。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1234 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1235 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1236 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L1236 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L1237 EN**: Blank line separating nearby declarations or logic blocks.
  **L1237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1238 EN**: Declares class `AdvSIMD_SVE_CMLA_Intrinsic`.
  **L1238 CN**: 声明 class `AdvSIMD_SVE_CMLA_Intrinsic`。
- **L1239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1239 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1240 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1241 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1242 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1243 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1244 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1245 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<4>>]>;`.
  **L1245 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<4>>]>;`。
- **L1246 EN**: Blank line separating nearby declarations or logic blocks.
  **L1246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1247 EN**: Declares class `AdvSIMD_SVE_CMLA_LANE_Intrinsic<list<IntrinsicProperty>`.
  **L1247 CN**: 声明 class `AdvSIMD_SVE_CMLA_LANE_Intrinsic<list<IntrinsicProperty>`。
- **L1248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1248 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。

### Lines 1249-1280

````tablegen
                [LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 llvm_i32_ty,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>])>;

  class AdvSIMD_SVE_DUP_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMVectorElementType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_DUP_Unpred_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMVectorElementType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_DUPQ_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 llvm_i64_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_EXPA_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMVectorOfBitcastsToInt<0>],
                [IntrNoMem]>;

  class AdvSIMD_SVE_FCVT_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
````
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1250 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1251 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,`.
  **L1252 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,`。
- **L1253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1253 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1254 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1254 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1256 EN**: Declares class `AdvSIMD_SVE_DUP_Intrinsic<list<IntrinsicProperty>`.
  **L1256 CN**: 声明 class `AdvSIMD_SVE_DUP_Intrinsic<list<IntrinsicProperty>`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1258 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1259 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>],`.
  **L1260 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>],`。
- **L1261 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1261 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1262 EN**: Blank line separating nearby declarations or logic blocks.
  **L1262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1263 EN**: Declares class `AdvSIMD_SVE_DUP_Unpred_Intrinsic<list<IntrinsicProperty>`.
  **L1263 CN**: 声明 class `AdvSIMD_SVE_DUP_Unpred_Intrinsic<list<IntrinsicProperty>`。
- **L1264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMVectorElementType<0>],`.
  **L1264 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMVectorElementType<0>],`。
- **L1265 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1265 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1266 EN**: Blank line separating nearby declarations or logic blocks.
  **L1266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1267 EN**: Declares class `AdvSIMD_SVE_DUPQ_Intrinsic<list<IntrinsicProperty>`.
  **L1267 CN**: 声明 class `AdvSIMD_SVE_DUPQ_Intrinsic<list<IntrinsicProperty>`。
- **L1268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1268 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1269 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty],`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty],`。
- **L1271 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1271 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1273 EN**: Declares class `AdvSIMD_SVE_EXPA_Intrinsic`.
  **L1273 CN**: 声明 class `AdvSIMD_SVE_EXPA_Intrinsic`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMVectorOfBitcastsToInt<0>],`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMVectorOfBitcastsToInt<0>],`。
- **L1276 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1276 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1277 EN**: Blank line separating nearby declarations or logic blocks.
  **L1277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1278 EN**: Declares class `AdvSIMD_SVE_FCVT_Intrinsic`.
  **L1278 CN**: 声明 class `AdvSIMD_SVE_FCVT_Intrinsic`。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。

### Lines 1281-1312

````tablegen
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyvector_ty],
                [IntrNoMem]>;

  class AdvSIMD_SVE_FCVTZS_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMVectorOfBitcastsToInt<0>,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyvector_ty],
                [IntrNoMem]>;

  class AdvSIMD_SVE_INSR_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMVectorElementType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_PTRUE_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<0>>]>;

  class AdvSIMD_SVE_PUNPKHI_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMOneNthElementsVectorType<0, 2>],
                [llvm_anyvector_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_SCALE_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 LLVMVectorOfBitcastsToInt<0>],
````
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1282 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1283 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1283 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1285 EN**: Declares class `AdvSIMD_SVE_FCVTZS_Intrinsic`.
  **L1285 CN**: 声明 class `AdvSIMD_SVE_FCVTZS_Intrinsic`。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMVectorOfBitcastsToInt<0>,`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMVectorOfBitcastsToInt<0>,`。
- **L1288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1288 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1290 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1290 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1291 EN**: Blank line separating nearby declarations or logic blocks.
  **L1291 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1292 EN**: Declares class `AdvSIMD_SVE_INSR_Intrinsic<list<IntrinsicProperty>`.
  **L1292 CN**: 声明 class `AdvSIMD_SVE_INSR_Intrinsic<list<IntrinsicProperty>`。
- **L1293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1293 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1294 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorElementType<0>],`.
  **L1295 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorElementType<0>],`。
- **L1296 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1296 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1297 EN**: Blank line separating nearby declarations or logic blocks.
  **L1297 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1298 EN**: Declares class `AdvSIMD_SVE_PTRUE_Intrinsic`.
  **L1298 CN**: 声明 class `AdvSIMD_SVE_PTRUE_Intrinsic`。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L1301 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L1301 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L1302 EN**: Blank line separating nearby declarations or logic blocks.
  **L1302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1303 EN**: Declares class `AdvSIMD_SVE_PUNPKHI_Intrinsic<list<IntrinsicProperty>`.
  **L1303 CN**: 声明 class `AdvSIMD_SVE_PUNPKHI_Intrinsic<list<IntrinsicProperty>`。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMOneNthElementsVectorType<0, 2>],`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMOneNthElementsVectorType<0, 2>],`。
- **L1305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1305 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1306 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1306 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1307 EN**: Blank line separating nearby declarations or logic blocks.
  **L1307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1308 EN**: Declares class `AdvSIMD_SVE_SCALE_Intrinsic`.
  **L1308 CN**: 声明 class `AdvSIMD_SVE_SCALE_Intrinsic`。
- **L1309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1309 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1310 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1311 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>],`.
  **L1312 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>],`。

### Lines 1313-1344

````tablegen
                [IntrNoMem]>;

  class AdvSIMD_SVE_SCVTF_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyvector_ty],
                [IntrNoMem]>;

  class AdvSIMD_SVE_TSMUL_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMVectorOfBitcastsToInt<0>],
                [IntrNoMem]>;

  class AdvSIMD_SVE_CNTB_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_i64_ty],
                [llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<0>>])>;

  class AdvSIMD_SVE_CNTP_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_i64_ty],
                [llvm_anyvector_ty, LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_DOT_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMSubdivide4VectorType<0>,
                 LLVMSubdivide4VectorType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

````
- **L1313 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1313 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1314 EN**: Blank line separating nearby declarations or logic blocks.
  **L1314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1315 EN**: Declares class `AdvSIMD_SVE_SCVTF_Intrinsic`.
  **L1315 CN**: 声明 class `AdvSIMD_SVE_SCVTF_Intrinsic`。
- **L1316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1316 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1317 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1318 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1319 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1320 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1320 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1321 EN**: Blank line separating nearby declarations or logic blocks.
  **L1321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1322 EN**: Declares class `AdvSIMD_SVE_TSMUL_Intrinsic`.
  **L1322 CN**: 声明 class `AdvSIMD_SVE_TSMUL_Intrinsic`。
- **L1323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1323 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1324 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>],`.
  **L1325 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>],`。
- **L1326 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1326 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1327 EN**: Blank line separating nearby declarations or logic blocks.
  **L1327 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1328 EN**: Declares class `AdvSIMD_SVE_CNTB_Intrinsic<list<IntrinsicProperty>`.
  **L1328 CN**: 声明 class `AdvSIMD_SVE_CNTB_Intrinsic<list<IntrinsicProperty>`。
- **L1329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty],`.
  **L1329 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty],`。
- **L1330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L1330 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L1331 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1331 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1332 EN**: Blank line separating nearby declarations or logic blocks.
  **L1332 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1333 EN**: Declares class `AdvSIMD_SVE_CNTP_Intrinsic<list<IntrinsicProperty>`.
  **L1333 CN**: 声明 class `AdvSIMD_SVE_CNTP_Intrinsic<list<IntrinsicProperty>`。
- **L1334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty],`.
  **L1334 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty],`。
- **L1335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1335 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1336 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1336 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1337 EN**: Blank line separating nearby declarations or logic blocks.
  **L1337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1338 EN**: Declares class `AdvSIMD_SVE_DOT_Intrinsic<list<IntrinsicProperty>`.
  **L1338 CN**: 声明 class `AdvSIMD_SVE_DOT_Intrinsic<list<IntrinsicProperty>`。
- **L1339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1339 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1340 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide4VectorType<0>,`.
  **L1341 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide4VectorType<0>,`。
- **L1342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide4VectorType<0>],`.
  **L1342 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide4VectorType<0>],`。
- **L1343 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1343 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1344 EN**: Blank line separating nearby declarations or logic blocks.
  **L1344 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1345-1376

````tablegen
  class AdvSIMD_SVE_DOT_Indexed_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMSubdivide4VectorType<0>,
                 LLVMSubdivide4VectorType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<3>>])>;

  class AdvSIMD_SVE_PTEST_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_i1_ty],
                [llvm_anyvector_ty,
                 LLVMMatchType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE_TBL_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMVectorOfBitcastsToInt<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class AdvSIMD_SVE2_TBX_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMMatchType<0>,
                 LLVMVectorOfBitcastsToInt<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_LUTI_Inrinsic<list<IntrinsicProperty> Attrs = []>
    :  DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 llvm_nxv16i8_ty,
                 llvm_i32_ty],
````
- **L1345 EN**: Declares class `AdvSIMD_SVE_DOT_Indexed_Intrinsic<list<IntrinsicProperty>`.
  **L1345 CN**: 声明 class `AdvSIMD_SVE_DOT_Indexed_Intrinsic<list<IntrinsicProperty>`。
- **L1346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1346 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1347 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide4VectorType<0>,`.
  **L1348 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide4VectorType<0>,`。
- **L1349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide4VectorType<0>,`.
  **L1349 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide4VectorType<0>,`。
- **L1350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1350 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1351 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1351 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1352 EN**: Blank line separating nearby declarations or logic blocks.
  **L1352 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1353 EN**: Declares class `AdvSIMD_SVE_PTEST_Intrinsic<list<IntrinsicProperty>`.
  **L1353 CN**: 声明 class `AdvSIMD_SVE_PTEST_Intrinsic<list<IntrinsicProperty>`。
- **L1354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i1_ty],`.
  **L1354 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i1_ty],`。
- **L1355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty,`.
  **L1355 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty,`。
- **L1356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1356 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1357 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1357 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1358 EN**: Blank line separating nearby declarations or logic blocks.
  **L1358 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1359 EN**: Declares class `AdvSIMD_SVE_TBL_Intrinsic<list<IntrinsicProperty>`.
  **L1359 CN**: 声明 class `AdvSIMD_SVE_TBL_Intrinsic<list<IntrinsicProperty>`。
- **L1360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1360 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1361 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>],`.
  **L1362 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>],`。
- **L1363 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1363 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1364 EN**: Blank line separating nearby declarations or logic blocks.
  **L1364 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1365 EN**: Declares class `AdvSIMD_SVE2_TBX_Intrinsic<list<IntrinsicProperty>`.
  **L1365 CN**: 声明 class `AdvSIMD_SVE2_TBX_Intrinsic<list<IntrinsicProperty>`。
- **L1366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1366 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1368 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>],`.
  **L1369 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>],`。
- **L1370 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1370 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1371 EN**: Blank line separating nearby declarations or logic blocks.
  **L1371 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1372 EN**: Declares class `SVE2_LUTI_Inrinsic<list<IntrinsicProperty>`.
  **L1372 CN**: 声明 class `SVE2_LUTI_Inrinsic<list<IntrinsicProperty>`。
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `:  DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`:  DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L1375 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L1376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1376 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。

### Lines 1377-1408

````tablegen
                 !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<2>>])>;

  class SVE2_1VectorArg_Long_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMSubdivide2VectorType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<1>>])>;

  class SVE2_2VectorArg_Long_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMSubdivide2VectorType<0>,
                 LLVMSubdivide2VectorType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_2VectorArgIndexed_Long_Intrinsic<list<IntrinsicProperty> Attrs = []>
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
              [LLVMSubdivide2VectorType<0>,
               LLVMSubdivide2VectorType<0>,
               llvm_i32_ty],
              !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<2>>])>;

  class SVE2_2VectorArg_Wide_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMSubdivide2VectorType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_2VectorArg_Pred_Long_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 LLVMMatchType<0>,
                 LLVMSubdivide2VectorType<0>],
````
- **L1377 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1377 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1378 EN**: Blank line separating nearby declarations or logic blocks.
  **L1378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1379 EN**: Declares class `SVE2_1VectorArg_Long_Intrinsic<list<IntrinsicProperty>`.
  **L1379 CN**: 声明 class `SVE2_1VectorArg_Long_Intrinsic<list<IntrinsicProperty>`。
- **L1380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1380 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>,`.
  **L1381 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>,`。
- **L1382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1382 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1383 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1383 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1384 EN**: Blank line separating nearby declarations or logic blocks.
  **L1384 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1385 EN**: Declares class `SVE2_2VectorArg_Long_Intrinsic<list<IntrinsicProperty>`.
  **L1385 CN**: 声明 class `SVE2_2VectorArg_Long_Intrinsic<list<IntrinsicProperty>`。
- **L1386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1386 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>,`.
  **L1387 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>,`。
- **L1388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>],`.
  **L1388 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>],`。
- **L1389 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1389 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1390 EN**: Blank line separating nearby declarations or logic blocks.
  **L1390 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1391 EN**: Declares class `SVE2_2VectorArgIndexed_Long_Intrinsic<list<IntrinsicProperty>`.
  **L1391 CN**: 声明 class `SVE2_2VectorArgIndexed_Long_Intrinsic<list<IntrinsicProperty>`。
- **L1392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1392 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>,`.
  **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>,`。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>,`.
  **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>,`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1396 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1396 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1397 EN**: Blank line separating nearby declarations or logic blocks.
  **L1397 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1398 EN**: Declares class `SVE2_2VectorArg_Wide_Intrinsic<list<IntrinsicProperty>`.
  **L1398 CN**: 声明 class `SVE2_2VectorArg_Wide_Intrinsic<list<IntrinsicProperty>`。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1400 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>],`.
  **L1401 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>],`。
- **L1402 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1402 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1403 EN**: Blank line separating nearby declarations or logic blocks.
  **L1403 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1404 EN**: Declares class `SVE2_2VectorArg_Pred_Long_Intrinsic<list<IntrinsicProperty>`.
  **L1404 CN**: 声明 class `SVE2_2VectorArg_Pred_Long_Intrinsic<list<IntrinsicProperty>`。
- **L1405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1405 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1406 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L1407 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L1408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>],`.
  **L1408 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>],`。

### Lines 1409-1440

````tablegen
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_3VectorArg_Long_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMSubdivide2VectorType<0>,
                 LLVMSubdivide2VectorType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_3VectorArgIndexed_Long_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMSubdivide2VectorType<0>,
                 LLVMSubdivide2VectorType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<3>>])>;

  class SVE2_1VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],
                [llvm_anyvector_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_Merged1VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],
                [LLVMSubdivide2VectorType<0>,
                 llvm_anyvector_ty],
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_2VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty> Attrs = []>
      : DefaultAttrsIntrinsic<
            [LLVMSubdivide2VectorType<0>],
            [llvm_anyvector_ty, LLVMMatchType<0>],
````
- **L1409 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1409 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1410 EN**: Blank line separating nearby declarations or logic blocks.
  **L1410 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1411 EN**: Declares class `SVE2_3VectorArg_Long_Intrinsic<list<IntrinsicProperty>`.
  **L1411 CN**: 声明 class `SVE2_3VectorArg_Long_Intrinsic<list<IntrinsicProperty>`。
- **L1412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1412 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1413 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>,`.
  **L1414 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>,`。
- **L1415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>],`.
  **L1415 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>],`。
- **L1416 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1416 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1417 EN**: Blank line separating nearby declarations or logic blocks.
  **L1417 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1418 EN**: Declares class `SVE2_3VectorArgIndexed_Long_Intrinsic<list<IntrinsicProperty>`.
  **L1418 CN**: 声明 class `SVE2_3VectorArgIndexed_Long_Intrinsic<list<IntrinsicProperty>`。
- **L1419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1419 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1420 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>,`.
  **L1421 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>,`。
- **L1422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>,`.
  **L1422 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>,`。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1424 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1424 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1425 EN**: Blank line separating nearby declarations or logic blocks.
  **L1425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1426 EN**: Declares class `SVE2_1VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`.
  **L1426 CN**: 声明 class `SVE2_1VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`。
- **L1427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`.
  **L1427 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`。
- **L1428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1428 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1429 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1429 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1431 EN**: Declares class `SVE2_Merged1VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`.
  **L1431 CN**: 声明 class `SVE2_Merged1VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`。
- **L1432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`.
  **L1432 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`。
- **L1433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>,`.
  **L1433 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>,`。
- **L1434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1434 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1435 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1435 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1436 EN**: Blank line separating nearby declarations or logic blocks.
  **L1436 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1437 EN**: Declares class `SVE2_2VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`.
  **L1437 CN**: 声明 class `SVE2_2VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`。
- **L1438 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L1438 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L1439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>],`.
  **L1439 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>],`。
- **L1440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1440 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。

### Lines 1441-1472

````tablegen
            !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_Merged2VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty> Attrs = []>
      : DefaultAttrsIntrinsic<
            [LLVMSubdivide2VectorType<0>],
            [LLVMSubdivide2VectorType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
            !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_1VectorArg_Imm_Narrowing_Intrinsic<list<IntrinsicProperty> Attrs = []>
      : DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],
                  [llvm_anyvector_ty, llvm_i32_ty],
                  !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<1>>])>;

  class SVE2_2VectorArg_Imm_Narrowing_Intrinsic<list<IntrinsicProperty> Attrs = []>
      : DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],
                  [LLVMSubdivide2VectorType<0>, llvm_anyvector_ty,
                   llvm_i32_ty],
                  !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<2>>])>;

  class SVE2_CONFLICT_DETECT_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [llvm_anyptr_ty, LLVMMatchType<1>],
                !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_3VectorArg_Indexed_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMSubdivide2VectorType<0>,
                 LLVMSubdivide2VectorType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<3>>])>;

````
- **L1441 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1441 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1442 EN**: Blank line separating nearby declarations or logic blocks.
  **L1442 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1443 EN**: Declares class `SVE2_Merged2VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`.
  **L1443 CN**: 声明 class `SVE2_Merged2VectorArg_Narrowing_Intrinsic<list<IntrinsicProperty>`。
- **L1444 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L1444 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>],`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>],`。
- **L1446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1446 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1447 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1447 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1448 EN**: Blank line separating nearby declarations or logic blocks.
  **L1448 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1449 EN**: Declares class `SVE2_1VectorArg_Imm_Narrowing_Intrinsic<list<IntrinsicProperty>`.
  **L1449 CN**: 声明 class `SVE2_1VectorArg_Imm_Narrowing_Intrinsic<list<IntrinsicProperty>`。
- **L1450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`.
  **L1450 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`。
- **L1451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty],`.
  **L1451 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty],`。
- **L1452 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1452 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1453 EN**: Blank line separating nearby declarations or logic blocks.
  **L1453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1454 EN**: Declares class `SVE2_2VectorArg_Imm_Narrowing_Intrinsic<list<IntrinsicProperty>`.
  **L1454 CN**: 声明 class `SVE2_2VectorArg_Imm_Narrowing_Intrinsic<list<IntrinsicProperty>`。
- **L1455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`.
  **L1455 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`。
- **L1456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>, llvm_anyvector_ty,`.
  **L1456 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>, llvm_anyvector_ty,`。
- **L1457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1457 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1458 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1458 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1459 EN**: Blank line separating nearby declarations or logic blocks.
  **L1459 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1460 EN**: Declares class `SVE2_CONFLICT_DETECT_Intrinsic<list<IntrinsicProperty>`.
  **L1460 CN**: 声明 class `SVE2_CONFLICT_DETECT_Intrinsic<list<IntrinsicProperty>`。
- **L1461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1461 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMMatchType<1>],`.
  **L1462 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMMatchType<1>],`。
- **L1463 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1463 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1464 EN**: Blank line separating nearby declarations or logic blocks.
  **L1464 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1465 EN**: Declares class `SVE2_3VectorArg_Indexed_Intrinsic<list<IntrinsicProperty>`.
  **L1465 CN**: 声明 class `SVE2_3VectorArg_Indexed_Intrinsic<list<IntrinsicProperty>`。
- **L1466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1466 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1467 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>,`.
  **L1468 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>,`。
- **L1469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide2VectorType<0>,`.
  **L1469 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide2VectorType<0>,`。
- **L1470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1470 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1471 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1471 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1472 EN**: Blank line separating nearby declarations or logic blocks.
  **L1472 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1473-1504

````tablegen
  class SVE2_1VectorArgIndexed_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<1>>])>;

  class AdvSIMD_SVE_CDOT_LANE_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>,
                 LLVMSubdivide4VectorType<0>,
                 LLVMSubdivide4VectorType<0>,
                 llvm_i32_ty,
                 llvm_i32_ty],
                !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>])>;

  class SVE2_1VectorArg_Pred_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                            [llvm_anyvector_ty],
                            !listconcat(Attrs, [IntrNoMem])>;

  class SVE2_1VectorArgIndexed_Pred_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                            [llvm_anyvector_ty, llvm_i32_ty],
                            !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<1>>])>;

  class SVE2_Pred_1VectorArgIndexed_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>,
                             LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_i32_ty],
                            !listconcat(Attrs, [IntrNoMem, ImmArg<ArgIndex<2>>])>;

  class SVE2_Pred_1VectorArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
````
- **L1473 EN**: Declares class `SVE2_1VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`.
  **L1473 CN**: 声明 class `SVE2_1VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`。
- **L1474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1474 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1475 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1476 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1477 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1477 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1478 EN**: Blank line separating nearby declarations or logic blocks.
  **L1478 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1479 EN**: Declares class `AdvSIMD_SVE_CDOT_LANE_Intrinsic<list<IntrinsicProperty>`.
  **L1479 CN**: 声明 class `AdvSIMD_SVE_CDOT_LANE_Intrinsic<list<IntrinsicProperty>`。
- **L1480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1480 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1481 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide4VectorType<0>,`.
  **L1482 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide4VectorType<0>,`。
- **L1483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMSubdivide4VectorType<0>,`.
  **L1483 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMSubdivide4VectorType<0>,`。
- **L1484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,`.
  **L1484 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,`。
- **L1485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1485 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1486 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1486 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1487 EN**: Blank line separating nearby declarations or logic blocks.
  **L1487 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1488 EN**: Declares class `SVE2_1VectorArg_Pred_Intrinsic<list<IntrinsicProperty>`.
  **L1488 CN**: 声明 class `SVE2_1VectorArg_Pred_Intrinsic<list<IntrinsicProperty>`。
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L1490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1490 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1491 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1491 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1492 EN**: Blank line separating nearby declarations or logic blocks.
  **L1492 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1493 EN**: Declares class `SVE2_1VectorArgIndexed_Pred_Intrinsic<list<IntrinsicProperty>`.
  **L1493 CN**: 声明 class `SVE2_1VectorArgIndexed_Pred_Intrinsic<list<IntrinsicProperty>`。
- **L1494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L1494 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L1495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty],`.
  **L1495 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty],`。
- **L1496 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1496 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1497 EN**: Blank line separating nearby declarations or logic blocks.
  **L1497 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1498 EN**: Declares class `SVE2_Pred_1VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`.
  **L1498 CN**: 声明 class `SVE2_Pred_1VectorArgIndexed_Intrinsic<list<IntrinsicProperty>`。
- **L1499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1499 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L1500 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L1501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_i32_ty],`.
  **L1501 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, llvm_i32_ty],`。
- **L1502 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1502 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1503 EN**: Blank line separating nearby declarations or logic blocks.
  **L1503 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1504 EN**: Declares class `SVE2_Pred_1VectorArg_Intrinsic<list<IntrinsicProperty>`.
  **L1504 CN**: 声明 class `SVE2_Pred_1VectorArg_Intrinsic<list<IntrinsicProperty>`。

### Lines 1505-1536

````tablegen
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],
                            !listconcat(Attrs, [IntrNoMem])>;

  // NOTE: There is no relationship between these intrinsics beyond an attempt
  // to reuse currently identical class definitions.
  class AdvSIMD_SVE_LOGB_Intrinsic : AdvSIMD_SVE_CNT_Intrinsic;
  class AdvSIMD_SVE2_CADD_Intrinsic<list<IntrinsicProperty> Attrs = []> : AdvSIMD_2VectorArgIndexed_Intrinsic<Attrs>;
  class AdvSIMD_SVE2_CMLA_Intrinsic<list<IntrinsicProperty> Attrs = []> : AdvSIMD_3VectorArgIndexed_Intrinsic<Attrs>;

  // This class of intrinsics are not intended to be useful within LLVM IR but
  // are instead here to support some of the more regid parts of the ACLE.
  class Builtin_SVCVT<LLVMType OUT, LLVMType PRED, LLVMType IN>
      : DefaultAttrsIntrinsic<[OUT], [OUT, PRED, IN], [IntrNoMem]>;
}

//===----------------------------------------------------------------------===//
// SVE

let TargetPrefix = "aarch64" in {  // All intrinsics start with "llvm.aarch64.".

class AdvSIMD_SVE_2SVBoolArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
  : DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],
                          [llvm_nxv16i1_ty],
                          !listconcat(Attrs, [IntrNoMem])>;

class AdvSIMD_SVE_3SVBoolArg_Intrinsic<list<IntrinsicProperty> Attrs = []>
  : DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],
                          [llvm_nxv16i1_ty, llvm_nxv16i1_ty],
                          !listconcat(Attrs, [IntrNoMem])>;

class AdvSIMD_SVE_Reduce_Intrinsic<list<IntrinsicProperty> Attrs = []>
````
- **L1505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1505 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`.
  **L1506 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>],`。
- **L1507 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1507 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1508 EN**: Blank line separating nearby declarations or logic blocks.
  **L1508 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1509 EN**: Comment highlights an implementation note: `NOTE: There is no relationship between these intrinsics beyond an attempt`.
  **L1509 CN**: 注释强调了一条实现说明：`NOTE: There is no relationship between these intrinsics beyond an attempt`。
- **L1510 EN**: Comment explains nearby logic, invariants, or intent: `to reuse currently identical class definitions.`.
  **L1510 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to reuse currently identical class definitions.`。
- **L1511 EN**: Declares class `AdvSIMD_SVE_LOGB_Intrinsic`.
  **L1511 CN**: 声明 class `AdvSIMD_SVE_LOGB_Intrinsic`。
- **L1512 EN**: Declares class `AdvSIMD_SVE2_CADD_Intrinsic<list<IntrinsicProperty>`.
  **L1512 CN**: 声明 class `AdvSIMD_SVE2_CADD_Intrinsic<list<IntrinsicProperty>`。
- **L1513 EN**: Declares class `AdvSIMD_SVE2_CMLA_Intrinsic<list<IntrinsicProperty>`.
  **L1513 CN**: 声明 class `AdvSIMD_SVE2_CMLA_Intrinsic<list<IntrinsicProperty>`。
- **L1514 EN**: Blank line separating nearby declarations or logic blocks.
  **L1514 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1515 EN**: Comment explains nearby logic, invariants, or intent: `This class of intrinsics are not intended to be useful within LLVM IR but`.
  **L1515 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This class of intrinsics are not intended to be useful within LLVM IR but`。
- **L1516 EN**: Comment explains nearby logic, invariants, or intent: `are instead here to support some of the more regid parts of the ACLE.`.
  **L1516 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`are instead here to support some of the more regid parts of the ACLE.`。
- **L1517 EN**: Declares class `Builtin_SVCVT<LLVMType`.
  **L1517 CN**: 声明 class `Builtin_SVCVT<LLVMType`。
- **L1518 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[OUT], [OUT, PRED, IN], [IntrNoMem]>;`.
  **L1518 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[OUT], [OUT, PRED, IN], [IntrNoMem]>;`。
- **L1519 EN**: Closes the current lexical scope or compound statement.
  **L1519 CN**: 结束当前词法作用域或复合语句块。
- **L1520 EN**: Blank line separating nearby declarations or logic blocks.
  **L1520 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1521 EN**: Banner comment marking a file or section boundary.
  **L1521 CN**: 横幅注释，用于标记文件或章节边界。
- **L1522 EN**: Comment explains nearby logic, invariants, or intent: `SVE`.
  **L1522 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE`。
- **L1523 EN**: Blank line separating nearby declarations or logic blocks.
  **L1523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1524 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1524 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1525 EN**: Blank line separating nearby declarations or logic blocks.
  **L1525 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1526 EN**: Declares class `AdvSIMD_SVE_2SVBoolArg_Intrinsic<list<IntrinsicProperty>`.
  **L1526 CN**: 声明 class `AdvSIMD_SVE_2SVBoolArg_Intrinsic<list<IntrinsicProperty>`。
- **L1527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],`.
  **L1527 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],`。
- **L1528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i1_ty],`.
  **L1528 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i1_ty],`。
- **L1529 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1529 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1530 EN**: Blank line separating nearby declarations or logic blocks.
  **L1530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1531 EN**: Declares class `AdvSIMD_SVE_3SVBoolArg_Intrinsic<list<IntrinsicProperty>`.
  **L1531 CN**: 声明 class `AdvSIMD_SVE_3SVBoolArg_Intrinsic<list<IntrinsicProperty>`。
- **L1532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],`.
  **L1532 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],`。
- **L1533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i1_ty, llvm_nxv16i1_ty],`.
  **L1533 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i1_ty, llvm_nxv16i1_ty],`。
- **L1534 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1534 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1535 EN**: Blank line separating nearby declarations or logic blocks.
  **L1535 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1536 EN**: Declares class `AdvSIMD_SVE_Reduce_Intrinsic<list<IntrinsicProperty>`.
  **L1536 CN**: 声明 class `AdvSIMD_SVE_Reduce_Intrinsic<list<IntrinsicProperty>`。

### Lines 1537-1568

````tablegen
  : DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],
              [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
               llvm_anyvector_ty],
              !listconcat(Attrs, [IntrNoMem])>;

class AdvSIMD_SVE_V128_Reduce_Intrinsic<list<IntrinsicProperty> Attrs = []>
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
              [LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,
               llvm_anyvector_ty],
               !listconcat(Attrs, [IntrNoMem])>;


class AdvSIMD_SVE_SADDV_Reduce_Intrinsic<list<IntrinsicProperty> Attrs = []>
  : DefaultAttrsIntrinsic<[llvm_i64_ty],
              [LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
               llvm_anyvector_ty],
              !listconcat(Attrs, [IntrNoMem])>;

class AdvSIMD_SVE_WHILE_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [llvm_anyint_ty, LLVMMatchType<1>],
                !listconcat(Attrs, [IntrNoMem])>;

class AdvSIMD_GatherLoad_SV_64b_Offsets_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                  llvm_anyptr_ty,
                  LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>
                ],
                [IntrReadMem, IntrArgMemOnly]>;

````
- **L1537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`.
  **L1537 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMVectorElementType<0>],`。
- **L1538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1538 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1539 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1540 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1540 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1541 EN**: Blank line separating nearby declarations or logic blocks.
  **L1541 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1542 EN**: Declares class `AdvSIMD_SVE_V128_Reduce_Intrinsic<list<IntrinsicProperty>`.
  **L1542 CN**: 声明 class `AdvSIMD_SVE_V128_Reduce_Intrinsic<list<IntrinsicProperty>`。
- **L1543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1543 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,`.
  **L1544 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>,`。
- **L1545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1545 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1546 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1546 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1547 EN**: Blank line separating nearby declarations or logic blocks.
  **L1547 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1548 EN**: Blank line separating nearby declarations or logic blocks.
  **L1548 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1549 EN**: Declares class `AdvSIMD_SVE_SADDV_Reduce_Intrinsic<list<IntrinsicProperty>`.
  **L1549 CN**: 声明 class `AdvSIMD_SVE_SADDV_Reduce_Intrinsic<list<IntrinsicProperty>`。
- **L1550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty],`.
  **L1550 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty],`。
- **L1551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1551 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty],`.
  **L1552 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty],`。
- **L1553 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1553 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1554 EN**: Blank line separating nearby declarations or logic blocks.
  **L1554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1555 EN**: Declares class `AdvSIMD_SVE_WHILE_Intrinsic<list<IntrinsicProperty>`.
  **L1555 CN**: 声明 class `AdvSIMD_SVE_WHILE_Intrinsic<list<IntrinsicProperty>`。
- **L1556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1556 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyint_ty, LLVMMatchType<1>],`.
  **L1557 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyint_ty, LLVMMatchType<1>],`。
- **L1558 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1558 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1559 EN**: Blank line separating nearby declarations or logic blocks.
  **L1559 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1560 EN**: Declares class `AdvSIMD_GatherLoad_SV_64b_Offsets_Intrinsic`.
  **L1560 CN**: 声明 class `AdvSIMD_GatherLoad_SV_64b_Offsets_Intrinsic`。
- **L1561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1561 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1562 EN**: Continues the surrounding expression or declaration: `[`.
  **L1562 CN**: 继续构造周围的表达式或声明：`[`。
- **L1563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1563 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1564 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1565 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>`.
  **L1565 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>`。
- **L1566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1566 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1567 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1567 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1568 EN**: Blank line separating nearby declarations or logic blocks.
  **L1568 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1569-1600

````tablegen
class AdvSIMD_GatherLoad_SV_64b_Offsets_WriteFFR_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                  llvm_anyptr_ty,
                  LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>
                ],
                [IntrInaccessibleMemOrArgMemOnly]>;

class AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                  llvm_anyptr_ty,
                  LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>
                ],
                [IntrReadMem, IntrArgMemOnly]>;

class AdvSIMD_GatherLoad_SV_32b_Offsets_WriteFFR_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                  llvm_anyptr_ty,
                  LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>
                ],
                [IntrInaccessibleMemOrArgMemOnly]>;

class AdvSIMD_GatherLoad_VS_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                  llvm_anyvector_ty,
````
- **L1569 EN**: Declares class `AdvSIMD_GatherLoad_SV_64b_Offsets_WriteFFR_Intrinsic`.
  **L1569 CN**: 声明 class `AdvSIMD_GatherLoad_SV_64b_Offsets_WriteFFR_Intrinsic`。
- **L1570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1570 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1571 EN**: Continues the surrounding expression or declaration: `[`.
  **L1571 CN**: 继续构造周围的表达式或声明：`[`。
- **L1572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1572 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1573 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1574 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>`.
  **L1574 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1576 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1576 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1577 EN**: Blank line separating nearby declarations or logic blocks.
  **L1577 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1578 EN**: Declares class `AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic`.
  **L1578 CN**: 声明 class `AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic`。
- **L1579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1579 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1580 EN**: Continues the surrounding expression or declaration: `[`.
  **L1580 CN**: 继续构造周围的表达式或声明：`[`。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1582 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1583 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>`.
  **L1583 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>`。
- **L1584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1584 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1585 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1585 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1586 EN**: Blank line separating nearby declarations or logic blocks.
  **L1586 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1587 EN**: Declares class `AdvSIMD_GatherLoad_SV_32b_Offsets_WriteFFR_Intrinsic`.
  **L1587 CN**: 声明 class `AdvSIMD_GatherLoad_SV_32b_Offsets_WriteFFR_Intrinsic`。
- **L1588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1588 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1589 EN**: Continues the surrounding expression or declaration: `[`.
  **L1589 CN**: 继续构造周围的表达式或声明：`[`。
- **L1590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1590 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1591 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1592 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>`.
  **L1592 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>`。
- **L1593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1593 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1594 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1594 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1595 EN**: Blank line separating nearby declarations or logic blocks.
  **L1595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1596 EN**: Declares class `AdvSIMD_GatherLoad_VS_Intrinsic`.
  **L1596 CN**: 声明 class `AdvSIMD_GatherLoad_VS_Intrinsic`。
- **L1597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1597 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1598 EN**: Continues the surrounding expression or declaration: `[`.
  **L1598 CN**: 继续构造周围的表达式或声明：`[`。
- **L1599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1599 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1600 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。

### Lines 1601-1632

````tablegen
                  llvm_i64_ty
                ],
                [IntrReadMem]>;

class AdvSIMD_GatherLoadQ_VS_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  llvm_nxv1i1_ty,
                  llvm_anyvector_ty,
                  llvm_i64_ty
                ],
                [IntrReadMem]>;

class AdvSIMD_GatherLoadQ_SV_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  llvm_nxv1i1_ty,
                  llvm_anyptr_ty,
                  llvm_nxv2i64_ty
                ],
                [IntrReadMem, IntrArgMemOnly]>;

class AdvSIMD_GatherLoad_VS_WriteFFR_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [
                  LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                  llvm_anyvector_ty,
                  llvm_i64_ty
                ],
                [IntrInaccessibleMemOrArgMemOnly]>;

class AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic
````
- **L1601 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty`.
  **L1601 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty`。
- **L1602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1602 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1603 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L1603 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L1604 EN**: Blank line separating nearby declarations or logic blocks.
  **L1604 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1605 EN**: Declares class `AdvSIMD_GatherLoadQ_VS_Intrinsic`.
  **L1605 CN**: 声明 class `AdvSIMD_GatherLoadQ_VS_Intrinsic`。
- **L1606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1606 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1607 EN**: Continues the surrounding expression or declaration: `[`.
  **L1607 CN**: 继续构造周围的表达式或声明：`[`。
- **L1608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv1i1_ty,`.
  **L1608 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv1i1_ty,`。
- **L1609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1609 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1610 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty`.
  **L1610 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty`。
- **L1611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1611 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1612 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L1612 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L1613 EN**: Blank line separating nearby declarations or logic blocks.
  **L1613 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1614 EN**: Declares class `AdvSIMD_GatherLoadQ_SV_Intrinsic`.
  **L1614 CN**: 声明 class `AdvSIMD_GatherLoadQ_SV_Intrinsic`。
- **L1615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1615 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1616 EN**: Continues the surrounding expression or declaration: `[`.
  **L1616 CN**: 继续构造周围的表达式或声明：`[`。
- **L1617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv1i1_ty,`.
  **L1617 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv1i1_ty,`。
- **L1618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1618 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1619 EN**: Continues the surrounding expression or declaration: `llvm_nxv2i64_ty`.
  **L1619 CN**: 继续构造周围的表达式或声明：`llvm_nxv2i64_ty`。
- **L1620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1620 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1621 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1621 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1622 EN**: Blank line separating nearby declarations or logic blocks.
  **L1622 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1623 EN**: Declares class `AdvSIMD_GatherLoad_VS_WriteFFR_Intrinsic`.
  **L1623 CN**: 声明 class `AdvSIMD_GatherLoad_VS_WriteFFR_Intrinsic`。
- **L1624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1624 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1625 EN**: Continues the surrounding expression or declaration: `[`.
  **L1625 CN**: 继续构造周围的表达式或声明：`[`。
- **L1626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1626 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1627 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1628 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty`.
  **L1628 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty`。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1630 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1630 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1631 EN**: Blank line separating nearby declarations or logic blocks.
  **L1631 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1632 EN**: Declares class `AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic`.
  **L1632 CN**: 声明 class `AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic`。

### Lines 1633-1664

````tablegen
    : DefaultAttrsIntrinsic<[],
               [
                 llvm_anyvector_ty,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyptr_ty,
                 LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>
               ],
               [IntrWriteMem, IntrArgMemOnly]>;

class AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic
    : DefaultAttrsIntrinsic<[],
               [
                 llvm_anyvector_ty,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyptr_ty,
                 LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>
               ],
               [IntrWriteMem, IntrArgMemOnly]>;

class AdvSIMD_ScatterStore_VS_Intrinsic
    : DefaultAttrsIntrinsic<[],
               [
                 llvm_anyvector_ty,
                 LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
                 llvm_anyvector_ty, llvm_i64_ty
               ],
               [IntrWriteMem]>;

class AdvSIMD_ScatterStoreQ_VS_Intrinsic
    : DefaultAttrsIntrinsic<[],
               [
                 llvm_anyvector_ty,
````
- **L1633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1633 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1634 EN**: Continues the surrounding expression or declaration: `[`.
  **L1634 CN**: 继续构造周围的表达式或声明：`[`。
- **L1635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1635 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1636 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1637 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1638 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>`.
  **L1638 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<0, llvm_i64_ty>`。
- **L1639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1639 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1640 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1640 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1641 EN**: Blank line separating nearby declarations or logic blocks.
  **L1641 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1642 EN**: Declares class `AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic`.
  **L1642 CN**: 声明 class `AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic`。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1644 EN**: Continues the surrounding expression or declaration: `[`.
  **L1644 CN**: 继续构造周围的表达式或声明：`[`。
- **L1645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1645 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1646 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1647 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1648 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>`.
  **L1648 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<0, llvm_i32_ty>`。
- **L1649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1649 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1650 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1650 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1651 EN**: Blank line separating nearby declarations or logic blocks.
  **L1651 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1652 EN**: Declares class `AdvSIMD_ScatterStore_VS_Intrinsic`.
  **L1652 CN**: 声明 class `AdvSIMD_ScatterStore_VS_Intrinsic`。
- **L1653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1653 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1654 EN**: Continues the surrounding expression or declaration: `[`.
  **L1654 CN**: 继续构造周围的表达式或声明：`[`。
- **L1655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1655 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L1656 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L1657 EN**: Continues the surrounding expression or declaration: `llvm_anyvector_ty, llvm_i64_ty`.
  **L1657 CN**: 继续构造周围的表达式或声明：`llvm_anyvector_ty, llvm_i64_ty`。
- **L1658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1658 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1659 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L1659 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L1660 EN**: Blank line separating nearby declarations or logic blocks.
  **L1660 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1661 EN**: Declares class `AdvSIMD_ScatterStoreQ_VS_Intrinsic`.
  **L1661 CN**: 声明 class `AdvSIMD_ScatterStoreQ_VS_Intrinsic`。
- **L1662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1662 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1663 EN**: Continues the surrounding expression or declaration: `[`.
  **L1663 CN**: 继续构造周围的表达式或声明：`[`。
- **L1664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1664 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。

### Lines 1665-1696

````tablegen
                 llvm_nxv1i1_ty,
                 llvm_anyvector_ty,
                 llvm_i64_ty
               ],
               [IntrWriteMem]>;

class AdvSIMD_ScatterStoreQ_SV_Intrinsic
    : DefaultAttrsIntrinsic<[],
               [
                 llvm_anyvector_ty,
                 llvm_nxv1i1_ty,
                 llvm_anyptr_ty,
                 llvm_nxv2i64_ty
               ],
               [IntrWriteMem, IntrArgMemOnly]>;

class SVE_gather_prf_SV
    : DefaultAttrsIntrinsic<[],
                [
                  LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, // Predicate
                  llvm_anyptr_ty, // Base address
                  llvm_anyvector_ty, // Offsets
                  llvm_i32_ty // Prfop
                ],
                [IntrInaccessibleMemOrArgMemOnly, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<3>>]>;

class SVE_gather_prf_VS
    : DefaultAttrsIntrinsic<[],
                [
                  LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, // Predicate
                  llvm_anyvector_ty, // Base addresses
                  llvm_i64_ty, // Scalar offset
````
- **L1665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv1i1_ty,`.
  **L1665 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv1i1_ty,`。
- **L1666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1666 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1667 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty`.
  **L1667 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty`。
- **L1668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1668 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1669 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L1669 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L1670 EN**: Blank line separating nearby declarations or logic blocks.
  **L1670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1671 EN**: Declares class `AdvSIMD_ScatterStoreQ_SV_Intrinsic`.
  **L1671 CN**: 声明 class `AdvSIMD_ScatterStoreQ_SV_Intrinsic`。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1673 EN**: Continues the surrounding expression or declaration: `[`.
  **L1673 CN**: 继续构造周围的表达式或声明：`[`。
- **L1674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L1674 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L1675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv1i1_ty,`.
  **L1675 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv1i1_ty,`。
- **L1676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyptr_ty,`.
  **L1676 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyptr_ty,`。
- **L1677 EN**: Continues the surrounding expression or declaration: `llvm_nxv2i64_ty`.
  **L1677 CN**: 继续构造周围的表达式或声明：`llvm_nxv2i64_ty`。
- **L1678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1678 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1679 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1679 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1680 EN**: Blank line separating nearby declarations or logic blocks.
  **L1680 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1681 EN**: Declares class `SVE_gather_prf_SV`.
  **L1681 CN**: 声明 class `SVE_gather_prf_SV`。
- **L1682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1682 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1683 EN**: Continues the surrounding expression or declaration: `[`.
  **L1683 CN**: 继续构造周围的表达式或声明：`[`。
- **L1684 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, // Predicate`.
  **L1684 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<1, llvm_i1_ty>, // Predicate`。
- **L1685 EN**: Continues the surrounding expression or declaration: `llvm_anyptr_ty, // Base address`.
  **L1685 CN**: 继续构造周围的表达式或声明：`llvm_anyptr_ty, // Base address`。
- **L1686 EN**: Continues the surrounding expression or declaration: `llvm_anyvector_ty, // Offsets`.
  **L1686 CN**: 继续构造周围的表达式或声明：`llvm_anyvector_ty, // Offsets`。
- **L1687 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty // Prfop`.
  **L1687 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty // Prfop`。
- **L1688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1688 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1689 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<3>>]>;`.
  **L1689 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly, NoCapture<ArgIndex<1>>, ImmArg<ArgIndex<3>>]>;`。
- **L1690 EN**: Blank line separating nearby declarations or logic blocks.
  **L1690 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1691 EN**: Declares class `SVE_gather_prf_VS`.
  **L1691 CN**: 声明 class `SVE_gather_prf_VS`。
- **L1692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L1692 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L1693 EN**: Continues the surrounding expression or declaration: `[`.
  **L1693 CN**: 继续构造周围的表达式或声明：`[`。
- **L1694 EN**: Continues the surrounding expression or declaration: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, // Predicate`.
  **L1694 CN**: 继续构造周围的表达式或声明：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>, // Predicate`。
- **L1695 EN**: Continues the surrounding expression or declaration: `llvm_anyvector_ty, // Base addresses`.
  **L1695 CN**: 继续构造周围的表达式或声明：`llvm_anyvector_ty, // Base addresses`。
- **L1696 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty, // Scalar offset`.
  **L1696 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty, // Scalar offset`。

### Lines 1697-1728

````tablegen
                  llvm_i32_ty // Prfop
                ],
                [IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<3>>]>;

class SVE_MatMul_Intrinsic<list<IntrinsicProperty> Attrs = []>
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                [LLVMMatchType<0>, LLVMSubdivide4VectorType<0>, LLVMSubdivide4VectorType<0>],
                !listconcat(Attrs, [IntrNoMem])>;

class SVE_4Vec_BF16
    : DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],
                [llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty],
                [IntrNoMem]>;

class SVE_4Vec_BF16_Indexed
    : DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],
                [llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty, llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<3>>]>;

//
// Loads
//

def int_aarch64_sve_ld1   : AdvSIMD_1Vec_PredLoad_Intrinsic;

def int_aarch64_sve_ld2_sret : AdvSIMD_2Vec_PredLoad_Intrinsic;
def int_aarch64_sve_ld3_sret : AdvSIMD_3Vec_PredLoad_Intrinsic;
def int_aarch64_sve_ld4_sret : AdvSIMD_4Vec_PredLoad_Intrinsic;

def int_aarch64_sve_ldnt1 : AdvSIMD_1Vec_PredLoad_Intrinsic;
def int_aarch64_sve_ldnf1 : AdvSIMD_1Vec_PredLoad_WriteFFR_Intrinsic;
def int_aarch64_sve_ldff1 : AdvSIMD_1Vec_PredLoad_WriteFFR_Intrinsic;
````
- **L1697 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty // Prfop`.
  **L1697 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty // Prfop`。
- **L1698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `],`.
  **L1698 CN**: 继续一个多行参数列表、初始化器或聚合项：`],`。
- **L1699 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<3>>]>;`.
  **L1699 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<3>>]>;`。
- **L1700 EN**: Blank line separating nearby declarations or logic blocks.
  **L1700 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1701 EN**: Declares class `SVE_MatMul_Intrinsic<list<IntrinsicProperty>`.
  **L1701 CN**: 声明 class `SVE_MatMul_Intrinsic<list<IntrinsicProperty>`。
- **L1702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L1702 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L1703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMSubdivide4VectorType<0>, LLVMSubdivide4VectorType<0>],`.
  **L1703 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMSubdivide4VectorType<0>, LLVMSubdivide4VectorType<0>],`。
- **L1704 EN**: Executes a call or declaration centered on `!listconcat`.
  **L1704 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L1705 EN**: Blank line separating nearby declarations or logic blocks.
  **L1705 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1706 EN**: Declares class `SVE_4Vec_BF16`.
  **L1706 CN**: 声明 class `SVE_4Vec_BF16`。
- **L1707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`.
  **L1707 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`。
- **L1708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty],`.
  **L1708 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty],`。
- **L1709 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1709 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1710 EN**: Blank line separating nearby declarations or logic blocks.
  **L1710 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1711 EN**: Declares class `SVE_4Vec_BF16_Indexed`.
  **L1711 CN**: 声明 class `SVE_4Vec_BF16_Indexed`。
- **L1712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`.
  **L1712 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`。
- **L1713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty, llvm_i32_ty],`.
  **L1713 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty, llvm_i32_ty],`。
- **L1714 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L1714 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L1715 EN**: Blank line separating nearby declarations or logic blocks.
  **L1715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1716 EN**: Separator comment used for visual grouping.
  **L1716 CN**: 用于视觉分组的分隔注释。
- **L1717 EN**: Comment explains nearby logic, invariants, or intent: `Loads`.
  **L1717 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Loads`。
- **L1718 EN**: Separator comment used for visual grouping.
  **L1718 CN**: 用于视觉分组的分隔注释。
- **L1719 EN**: Blank line separating nearby declarations or logic blocks.
  **L1719 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1720 EN**: Declares TableGen def `int_aarch64_sve_ld1`.
  **L1720 CN**: 声明 TableGen def `int_aarch64_sve_ld1`。
- **L1721 EN**: Blank line separating nearby declarations or logic blocks.
  **L1721 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1722 EN**: Declares TableGen def `int_aarch64_sve_ld2_sret`.
  **L1722 CN**: 声明 TableGen def `int_aarch64_sve_ld2_sret`。
- **L1723 EN**: Declares TableGen def `int_aarch64_sve_ld3_sret`.
  **L1723 CN**: 声明 TableGen def `int_aarch64_sve_ld3_sret`。
- **L1724 EN**: Declares TableGen def `int_aarch64_sve_ld4_sret`.
  **L1724 CN**: 声明 TableGen def `int_aarch64_sve_ld4_sret`。
- **L1725 EN**: Blank line separating nearby declarations or logic blocks.
  **L1725 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1726 EN**: Declares TableGen def `int_aarch64_sve_ldnt1`.
  **L1726 CN**: 声明 TableGen def `int_aarch64_sve_ldnt1`。
- **L1727 EN**: Declares TableGen def `int_aarch64_sve_ldnf1`.
  **L1727 CN**: 声明 TableGen def `int_aarch64_sve_ldnf1`。
- **L1728 EN**: Declares TableGen def `int_aarch64_sve_ldff1`.
  **L1728 CN**: 声明 TableGen def `int_aarch64_sve_ldff1`。

### Lines 1729-1760

````tablegen

def int_aarch64_sve_ld1rq : AdvSIMD_1Vec_PredLoad_Intrinsic;
def int_aarch64_sve_ld1ro : AdvSIMD_1Vec_PredLoad_Intrinsic;

//
// Stores
//

def int_aarch64_sve_st1  : AdvSIMD_1Vec_PredStore_Intrinsic;
def int_aarch64_sve_st2  : AdvSIMD_2Vec_PredStore_Intrinsic;
def int_aarch64_sve_st3  : AdvSIMD_3Vec_PredStore_Intrinsic;
def int_aarch64_sve_st4  : AdvSIMD_4Vec_PredStore_Intrinsic;

def int_aarch64_sve_stnt1 : AdvSIMD_1Vec_PredStore_Intrinsic;

//
// Prefetches
//

def int_aarch64_sve_prf
  : DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, llvm_anyptr_ty, llvm_i32_ty],
                  [IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;

// Scalar + 32-bit scaled offset vector, zero extend, packed and
// unpacked.
def int_aarch64_sve_prfb_gather_uxtw_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfh_gather_uxtw_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfw_gather_uxtw_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfd_gather_uxtw_index : SVE_gather_prf_SV;

// Scalar + 32-bit scaled offset vector, sign extend, packed and
// unpacked.
````
- **L1729 EN**: Blank line separating nearby declarations or logic blocks.
  **L1729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1730 EN**: Declares TableGen def `int_aarch64_sve_ld1rq`.
  **L1730 CN**: 声明 TableGen def `int_aarch64_sve_ld1rq`。
- **L1731 EN**: Declares TableGen def `int_aarch64_sve_ld1ro`.
  **L1731 CN**: 声明 TableGen def `int_aarch64_sve_ld1ro`。
- **L1732 EN**: Blank line separating nearby declarations or logic blocks.
  **L1732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1733 EN**: Separator comment used for visual grouping.
  **L1733 CN**: 用于视觉分组的分隔注释。
- **L1734 EN**: Comment explains nearby logic, invariants, or intent: `Stores`.
  **L1734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Stores`。
- **L1735 EN**: Separator comment used for visual grouping.
  **L1735 CN**: 用于视觉分组的分隔注释。
- **L1736 EN**: Blank line separating nearby declarations or logic blocks.
  **L1736 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1737 EN**: Declares TableGen def `int_aarch64_sve_st1`.
  **L1737 CN**: 声明 TableGen def `int_aarch64_sve_st1`。
- **L1738 EN**: Declares TableGen def `int_aarch64_sve_st2`.
  **L1738 CN**: 声明 TableGen def `int_aarch64_sve_st2`。
- **L1739 EN**: Declares TableGen def `int_aarch64_sve_st3`.
  **L1739 CN**: 声明 TableGen def `int_aarch64_sve_st3`。
- **L1740 EN**: Declares TableGen def `int_aarch64_sve_st4`.
  **L1740 CN**: 声明 TableGen def `int_aarch64_sve_st4`。
- **L1741 EN**: Blank line separating nearby declarations or logic blocks.
  **L1741 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1742 EN**: Declares TableGen def `int_aarch64_sve_stnt1`.
  **L1742 CN**: 声明 TableGen def `int_aarch64_sve_stnt1`。
- **L1743 EN**: Blank line separating nearby declarations or logic blocks.
  **L1743 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1744 EN**: Separator comment used for visual grouping.
  **L1744 CN**: 用于视觉分组的分隔注释。
- **L1745 EN**: Comment explains nearby logic, invariants, or intent: `Prefetches`.
  **L1745 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Prefetches`。
- **L1746 EN**: Separator comment used for visual grouping.
  **L1746 CN**: 用于视觉分组的分隔注释。
- **L1747 EN**: Blank line separating nearby declarations or logic blocks.
  **L1747 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1748 EN**: Declares TableGen def `int_aarch64_sve_prf`.
  **L1748 CN**: 声明 TableGen def `int_aarch64_sve_prf`。
- **L1749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, llvm_anyptr_ty, llvm_i32_ty],`.
  **L1749 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_anyvector_ty, llvm_anyptr_ty, llvm_i32_ty],`。
- **L1750 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L1750 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L1751 EN**: Blank line separating nearby declarations or logic blocks.
  **L1751 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1752 EN**: Comment explains nearby logic, invariants, or intent: `Scalar + 32-bit scaled offset vector, zero extend, packed and`.
  **L1752 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar + 32-bit scaled offset vector, zero extend, packed and`。
- **L1753 EN**: Comment explains nearby logic, invariants, or intent: `unpacked.`.
  **L1753 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`unpacked.`。
- **L1754 EN**: Declares TableGen def `int_aarch64_sve_prfb_gather_uxtw_index`.
  **L1754 CN**: 声明 TableGen def `int_aarch64_sve_prfb_gather_uxtw_index`。
- **L1755 EN**: Declares TableGen def `int_aarch64_sve_prfh_gather_uxtw_index`.
  **L1755 CN**: 声明 TableGen def `int_aarch64_sve_prfh_gather_uxtw_index`。
- **L1756 EN**: Declares TableGen def `int_aarch64_sve_prfw_gather_uxtw_index`.
  **L1756 CN**: 声明 TableGen def `int_aarch64_sve_prfw_gather_uxtw_index`。
- **L1757 EN**: Declares TableGen def `int_aarch64_sve_prfd_gather_uxtw_index`.
  **L1757 CN**: 声明 TableGen def `int_aarch64_sve_prfd_gather_uxtw_index`。
- **L1758 EN**: Blank line separating nearby declarations or logic blocks.
  **L1758 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1759 EN**: Comment explains nearby logic, invariants, or intent: `Scalar + 32-bit scaled offset vector, sign extend, packed and`.
  **L1759 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar + 32-bit scaled offset vector, sign extend, packed and`。
- **L1760 EN**: Comment explains nearby logic, invariants, or intent: `unpacked.`.
  **L1760 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`unpacked.`。

### Lines 1761-1792

````tablegen
def int_aarch64_sve_prfb_gather_sxtw_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfw_gather_sxtw_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfh_gather_sxtw_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfd_gather_sxtw_index : SVE_gather_prf_SV;

// Scalar + 64-bit scaled offset vector.
def int_aarch64_sve_prfb_gather_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfh_gather_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfw_gather_index : SVE_gather_prf_SV;
def int_aarch64_sve_prfd_gather_index : SVE_gather_prf_SV;

// Vector + scalar.
def int_aarch64_sve_prfb_gather_scalar_offset : SVE_gather_prf_VS;
def int_aarch64_sve_prfh_gather_scalar_offset : SVE_gather_prf_VS;
def int_aarch64_sve_prfw_gather_scalar_offset : SVE_gather_prf_VS;
def int_aarch64_sve_prfd_gather_scalar_offset : SVE_gather_prf_VS;

//
// Scalar to vector operations
//

def int_aarch64_sve_dup : AdvSIMD_SVE_DUP_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_dup_x : AdvSIMD_SVE_DUP_Unpred_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_index : AdvSIMD_SVE_Index_Intrinsic<[IntrSpeculatable]>;

//
// Address calculation
//

def int_aarch64_sve_adrb : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_adrh : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L1761 EN**: Declares TableGen def `int_aarch64_sve_prfb_gather_sxtw_index`.
  **L1761 CN**: 声明 TableGen def `int_aarch64_sve_prfb_gather_sxtw_index`。
- **L1762 EN**: Declares TableGen def `int_aarch64_sve_prfw_gather_sxtw_index`.
  **L1762 CN**: 声明 TableGen def `int_aarch64_sve_prfw_gather_sxtw_index`。
- **L1763 EN**: Declares TableGen def `int_aarch64_sve_prfh_gather_sxtw_index`.
  **L1763 CN**: 声明 TableGen def `int_aarch64_sve_prfh_gather_sxtw_index`。
- **L1764 EN**: Declares TableGen def `int_aarch64_sve_prfd_gather_sxtw_index`.
  **L1764 CN**: 声明 TableGen def `int_aarch64_sve_prfd_gather_sxtw_index`。
- **L1765 EN**: Blank line separating nearby declarations or logic blocks.
  **L1765 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1766 EN**: Comment explains nearby logic, invariants, or intent: `Scalar + 64-bit scaled offset vector.`.
  **L1766 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar + 64-bit scaled offset vector.`。
- **L1767 EN**: Declares TableGen def `int_aarch64_sve_prfb_gather_index`.
  **L1767 CN**: 声明 TableGen def `int_aarch64_sve_prfb_gather_index`。
- **L1768 EN**: Declares TableGen def `int_aarch64_sve_prfh_gather_index`.
  **L1768 CN**: 声明 TableGen def `int_aarch64_sve_prfh_gather_index`。
- **L1769 EN**: Declares TableGen def `int_aarch64_sve_prfw_gather_index`.
  **L1769 CN**: 声明 TableGen def `int_aarch64_sve_prfw_gather_index`。
- **L1770 EN**: Declares TableGen def `int_aarch64_sve_prfd_gather_index`.
  **L1770 CN**: 声明 TableGen def `int_aarch64_sve_prfd_gather_index`。
- **L1771 EN**: Blank line separating nearby declarations or logic blocks.
  **L1771 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1772 EN**: Comment explains nearby logic, invariants, or intent: `Vector + scalar.`.
  **L1772 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector + scalar.`。
- **L1773 EN**: Declares TableGen def `int_aarch64_sve_prfb_gather_scalar_offset`.
  **L1773 CN**: 声明 TableGen def `int_aarch64_sve_prfb_gather_scalar_offset`。
- **L1774 EN**: Declares TableGen def `int_aarch64_sve_prfh_gather_scalar_offset`.
  **L1774 CN**: 声明 TableGen def `int_aarch64_sve_prfh_gather_scalar_offset`。
- **L1775 EN**: Declares TableGen def `int_aarch64_sve_prfw_gather_scalar_offset`.
  **L1775 CN**: 声明 TableGen def `int_aarch64_sve_prfw_gather_scalar_offset`。
- **L1776 EN**: Declares TableGen def `int_aarch64_sve_prfd_gather_scalar_offset`.
  **L1776 CN**: 声明 TableGen def `int_aarch64_sve_prfd_gather_scalar_offset`。
- **L1777 EN**: Blank line separating nearby declarations or logic blocks.
  **L1777 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1778 EN**: Separator comment used for visual grouping.
  **L1778 CN**: 用于视觉分组的分隔注释。
- **L1779 EN**: Comment explains nearby logic, invariants, or intent: `Scalar to vector operations`.
  **L1779 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalar to vector operations`。
- **L1780 EN**: Separator comment used for visual grouping.
  **L1780 CN**: 用于视觉分组的分隔注释。
- **L1781 EN**: Blank line separating nearby declarations or logic blocks.
  **L1781 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1782 EN**: Declares TableGen def `int_aarch64_sve_dup`.
  **L1782 CN**: 声明 TableGen def `int_aarch64_sve_dup`。
- **L1783 EN**: Declares TableGen def `int_aarch64_sve_dup_x`.
  **L1783 CN**: 声明 TableGen def `int_aarch64_sve_dup_x`。
- **L1784 EN**: Blank line separating nearby declarations or logic blocks.
  **L1784 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1785 EN**: Declares TableGen def `int_aarch64_sve_index`.
  **L1785 CN**: 声明 TableGen def `int_aarch64_sve_index`。
- **L1786 EN**: Blank line separating nearby declarations or logic blocks.
  **L1786 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1787 EN**: Separator comment used for visual grouping.
  **L1787 CN**: 用于视觉分组的分隔注释。
- **L1788 EN**: Comment explains nearby logic, invariants, or intent: `Address calculation`.
  **L1788 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Address calculation`。
- **L1789 EN**: Separator comment used for visual grouping.
  **L1789 CN**: 用于视觉分组的分隔注释。
- **L1790 EN**: Blank line separating nearby declarations or logic blocks.
  **L1790 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1791 EN**: Declares TableGen def `int_aarch64_sve_adrb`.
  **L1791 CN**: 声明 TableGen def `int_aarch64_sve_adrb`。
- **L1792 EN**: Declares TableGen def `int_aarch64_sve_adrh`.
  **L1792 CN**: 声明 TableGen def `int_aarch64_sve_adrh`。

### Lines 1793-1824

````tablegen
def int_aarch64_sve_adrw : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_adrd : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// Integer arithmetic
//

def int_aarch64_sve_add   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_add_u : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sub   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sub_u : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_subr  : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_pmul       : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_mul        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mul_u      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mul_lane   : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smulh      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smulh_u    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umulh      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umulh_u    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sdiv       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sdiv_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_udiv       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_udiv_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sdivr      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_udivr      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_smax       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smax_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L1793 EN**: Declares TableGen def `int_aarch64_sve_adrw`.
  **L1793 CN**: 声明 TableGen def `int_aarch64_sve_adrw`。
- **L1794 EN**: Declares TableGen def `int_aarch64_sve_adrd`.
  **L1794 CN**: 声明 TableGen def `int_aarch64_sve_adrd`。
- **L1795 EN**: Blank line separating nearby declarations or logic blocks.
  **L1795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1796 EN**: Separator comment used for visual grouping.
  **L1796 CN**: 用于视觉分组的分隔注释。
- **L1797 EN**: Comment explains nearby logic, invariants, or intent: `Integer arithmetic`.
  **L1797 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer arithmetic`。
- **L1798 EN**: Separator comment used for visual grouping.
  **L1798 CN**: 用于视觉分组的分隔注释。
- **L1799 EN**: Blank line separating nearby declarations or logic blocks.
  **L1799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1800 EN**: Declares TableGen def `int_aarch64_sve_add`.
  **L1800 CN**: 声明 TableGen def `int_aarch64_sve_add`。
- **L1801 EN**: Declares TableGen def `int_aarch64_sve_add_u`.
  **L1801 CN**: 声明 TableGen def `int_aarch64_sve_add_u`。
- **L1802 EN**: Declares TableGen def `int_aarch64_sve_sub`.
  **L1802 CN**: 声明 TableGen def `int_aarch64_sve_sub`。
- **L1803 EN**: Declares TableGen def `int_aarch64_sve_sub_u`.
  **L1803 CN**: 声明 TableGen def `int_aarch64_sve_sub_u`。
- **L1804 EN**: Declares TableGen def `int_aarch64_sve_subr`.
  **L1804 CN**: 声明 TableGen def `int_aarch64_sve_subr`。
- **L1805 EN**: Blank line separating nearby declarations or logic blocks.
  **L1805 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1806 EN**: Declares TableGen def `int_aarch64_sve_pmul`.
  **L1806 CN**: 声明 TableGen def `int_aarch64_sve_pmul`。
- **L1807 EN**: Blank line separating nearby declarations or logic blocks.
  **L1807 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1808 EN**: Declares TableGen def `int_aarch64_sve_mul`.
  **L1808 CN**: 声明 TableGen def `int_aarch64_sve_mul`。
- **L1809 EN**: Declares TableGen def `int_aarch64_sve_mul_u`.
  **L1809 CN**: 声明 TableGen def `int_aarch64_sve_mul_u`。
- **L1810 EN**: Declares TableGen def `int_aarch64_sve_mul_lane`.
  **L1810 CN**: 声明 TableGen def `int_aarch64_sve_mul_lane`。
- **L1811 EN**: Declares TableGen def `int_aarch64_sve_smulh`.
  **L1811 CN**: 声明 TableGen def `int_aarch64_sve_smulh`。
- **L1812 EN**: Declares TableGen def `int_aarch64_sve_smulh_u`.
  **L1812 CN**: 声明 TableGen def `int_aarch64_sve_smulh_u`。
- **L1813 EN**: Declares TableGen def `int_aarch64_sve_umulh`.
  **L1813 CN**: 声明 TableGen def `int_aarch64_sve_umulh`。
- **L1814 EN**: Declares TableGen def `int_aarch64_sve_umulh_u`.
  **L1814 CN**: 声明 TableGen def `int_aarch64_sve_umulh_u`。
- **L1815 EN**: Blank line separating nearby declarations or logic blocks.
  **L1815 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1816 EN**: Declares TableGen def `int_aarch64_sve_sdiv`.
  **L1816 CN**: 声明 TableGen def `int_aarch64_sve_sdiv`。
- **L1817 EN**: Declares TableGen def `int_aarch64_sve_sdiv_u`.
  **L1817 CN**: 声明 TableGen def `int_aarch64_sve_sdiv_u`。
- **L1818 EN**: Declares TableGen def `int_aarch64_sve_udiv`.
  **L1818 CN**: 声明 TableGen def `int_aarch64_sve_udiv`。
- **L1819 EN**: Declares TableGen def `int_aarch64_sve_udiv_u`.
  **L1819 CN**: 声明 TableGen def `int_aarch64_sve_udiv_u`。
- **L1820 EN**: Declares TableGen def `int_aarch64_sve_sdivr`.
  **L1820 CN**: 声明 TableGen def `int_aarch64_sve_sdivr`。
- **L1821 EN**: Declares TableGen def `int_aarch64_sve_udivr`.
  **L1821 CN**: 声明 TableGen def `int_aarch64_sve_udivr`。
- **L1822 EN**: Blank line separating nearby declarations or logic blocks.
  **L1822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1823 EN**: Declares TableGen def `int_aarch64_sve_smax`.
  **L1823 CN**: 声明 TableGen def `int_aarch64_sve_smax`。
- **L1824 EN**: Declares TableGen def `int_aarch64_sve_smax_u`.
  **L1824 CN**: 声明 TableGen def `int_aarch64_sve_smax_u`。

### Lines 1825-1856

````tablegen
def int_aarch64_sve_umax       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umax_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smin       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smin_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umin       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umin_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sabd       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sabd_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uabd       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uabd_u     : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_mad        : AdvSIMD_Pred3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_msb        : AdvSIMD_Pred3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mla        : AdvSIMD_Pred3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mla_u      : AdvSIMD_Pred3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mla_lane   : AdvSIMD_3VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mls        : AdvSIMD_Pred3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mls_u      : AdvSIMD_Pred3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_mls_lane   : AdvSIMD_3VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_saddv      : AdvSIMD_SVE_SADDV_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uaddv      : AdvSIMD_SVE_SADDV_Reduce_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_smaxv      : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umaxv      : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sminv      : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uminv      : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_orv        : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_eorv       : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_andv       : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;

````
- **L1825 EN**: Declares TableGen def `int_aarch64_sve_umax`.
  **L1825 CN**: 声明 TableGen def `int_aarch64_sve_umax`。
- **L1826 EN**: Declares TableGen def `int_aarch64_sve_umax_u`.
  **L1826 CN**: 声明 TableGen def `int_aarch64_sve_umax_u`。
- **L1827 EN**: Declares TableGen def `int_aarch64_sve_smin`.
  **L1827 CN**: 声明 TableGen def `int_aarch64_sve_smin`。
- **L1828 EN**: Declares TableGen def `int_aarch64_sve_smin_u`.
  **L1828 CN**: 声明 TableGen def `int_aarch64_sve_smin_u`。
- **L1829 EN**: Declares TableGen def `int_aarch64_sve_umin`.
  **L1829 CN**: 声明 TableGen def `int_aarch64_sve_umin`。
- **L1830 EN**: Declares TableGen def `int_aarch64_sve_umin_u`.
  **L1830 CN**: 声明 TableGen def `int_aarch64_sve_umin_u`。
- **L1831 EN**: Declares TableGen def `int_aarch64_sve_sabd`.
  **L1831 CN**: 声明 TableGen def `int_aarch64_sve_sabd`。
- **L1832 EN**: Declares TableGen def `int_aarch64_sve_sabd_u`.
  **L1832 CN**: 声明 TableGen def `int_aarch64_sve_sabd_u`。
- **L1833 EN**: Declares TableGen def `int_aarch64_sve_uabd`.
  **L1833 CN**: 声明 TableGen def `int_aarch64_sve_uabd`。
- **L1834 EN**: Declares TableGen def `int_aarch64_sve_uabd_u`.
  **L1834 CN**: 声明 TableGen def `int_aarch64_sve_uabd_u`。
- **L1835 EN**: Blank line separating nearby declarations or logic blocks.
  **L1835 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1836 EN**: Declares TableGen def `int_aarch64_sve_mad`.
  **L1836 CN**: 声明 TableGen def `int_aarch64_sve_mad`。
- **L1837 EN**: Declares TableGen def `int_aarch64_sve_msb`.
  **L1837 CN**: 声明 TableGen def `int_aarch64_sve_msb`。
- **L1838 EN**: Declares TableGen def `int_aarch64_sve_mla`.
  **L1838 CN**: 声明 TableGen def `int_aarch64_sve_mla`。
- **L1839 EN**: Declares TableGen def `int_aarch64_sve_mla_u`.
  **L1839 CN**: 声明 TableGen def `int_aarch64_sve_mla_u`。
- **L1840 EN**: Declares TableGen def `int_aarch64_sve_mla_lane`.
  **L1840 CN**: 声明 TableGen def `int_aarch64_sve_mla_lane`。
- **L1841 EN**: Declares TableGen def `int_aarch64_sve_mls`.
  **L1841 CN**: 声明 TableGen def `int_aarch64_sve_mls`。
- **L1842 EN**: Declares TableGen def `int_aarch64_sve_mls_u`.
  **L1842 CN**: 声明 TableGen def `int_aarch64_sve_mls_u`。
- **L1843 EN**: Declares TableGen def `int_aarch64_sve_mls_lane`.
  **L1843 CN**: 声明 TableGen def `int_aarch64_sve_mls_lane`。
- **L1844 EN**: Blank line separating nearby declarations or logic blocks.
  **L1844 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1845 EN**: Declares TableGen def `int_aarch64_sve_saddv`.
  **L1845 CN**: 声明 TableGen def `int_aarch64_sve_saddv`。
- **L1846 EN**: Declares TableGen def `int_aarch64_sve_uaddv`.
  **L1846 CN**: 声明 TableGen def `int_aarch64_sve_uaddv`。
- **L1847 EN**: Blank line separating nearby declarations or logic blocks.
  **L1847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1848 EN**: Declares TableGen def `int_aarch64_sve_smaxv`.
  **L1848 CN**: 声明 TableGen def `int_aarch64_sve_smaxv`。
- **L1849 EN**: Declares TableGen def `int_aarch64_sve_umaxv`.
  **L1849 CN**: 声明 TableGen def `int_aarch64_sve_umaxv`。
- **L1850 EN**: Declares TableGen def `int_aarch64_sve_sminv`.
  **L1850 CN**: 声明 TableGen def `int_aarch64_sve_sminv`。
- **L1851 EN**: Declares TableGen def `int_aarch64_sve_uminv`.
  **L1851 CN**: 声明 TableGen def `int_aarch64_sve_uminv`。
- **L1852 EN**: Blank line separating nearby declarations or logic blocks.
  **L1852 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1853 EN**: Declares TableGen def `int_aarch64_sve_orv`.
  **L1853 CN**: 声明 TableGen def `int_aarch64_sve_orv`。
- **L1854 EN**: Declares TableGen def `int_aarch64_sve_eorv`.
  **L1854 CN**: 声明 TableGen def `int_aarch64_sve_eorv`。
- **L1855 EN**: Declares TableGen def `int_aarch64_sve_andv`.
  **L1855 CN**: 声明 TableGen def `int_aarch64_sve_andv`。
- **L1856 EN**: Blank line separating nearby declarations or logic blocks.
  **L1856 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1857-1888

````tablegen
def int_aarch64_sve_abs : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_neg : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sdot      : AdvSIMD_SVE_DOT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sdot_lane : AdvSIMD_SVE_DOT_Indexed_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_udot      : AdvSIMD_SVE_DOT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_udot_lane : AdvSIMD_SVE_DOT_Indexed_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sqadd_x   : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqsub_x   : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqadd_x   : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqsub_x   : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_orqv      : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_eorqv     : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_andqv     : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_addqv     : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smaxqv    : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umaxqv    : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sminqv    : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uminqv    : AdvSIMD_SVE_V128_Reduce_Intrinsic<[IntrSpeculatable]>;

// Shifts

def int_aarch64_sve_asr      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_asr_u    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_asr_wide : AdvSIMD_SVE_ShiftWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_asrd     : AdvSIMD_SVE_ShiftByImm_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_insr     : AdvSIMD_SVE_INSR_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lsl      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lsl_u    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L1857 EN**: Declares TableGen def `int_aarch64_sve_abs`.
  **L1857 CN**: 声明 TableGen def `int_aarch64_sve_abs`。
- **L1858 EN**: Declares TableGen def `int_aarch64_sve_neg`.
  **L1858 CN**: 声明 TableGen def `int_aarch64_sve_neg`。
- **L1859 EN**: Blank line separating nearby declarations or logic blocks.
  **L1859 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1860 EN**: Declares TableGen def `int_aarch64_sve_sdot`.
  **L1860 CN**: 声明 TableGen def `int_aarch64_sve_sdot`。
- **L1861 EN**: Declares TableGen def `int_aarch64_sve_sdot_lane`.
  **L1861 CN**: 声明 TableGen def `int_aarch64_sve_sdot_lane`。
- **L1862 EN**: Blank line separating nearby declarations or logic blocks.
  **L1862 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1863 EN**: Declares TableGen def `int_aarch64_sve_udot`.
  **L1863 CN**: 声明 TableGen def `int_aarch64_sve_udot`。
- **L1864 EN**: Declares TableGen def `int_aarch64_sve_udot_lane`.
  **L1864 CN**: 声明 TableGen def `int_aarch64_sve_udot_lane`。
- **L1865 EN**: Blank line separating nearby declarations or logic blocks.
  **L1865 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1866 EN**: Declares TableGen def `int_aarch64_sve_sqadd_x`.
  **L1866 CN**: 声明 TableGen def `int_aarch64_sve_sqadd_x`。
- **L1867 EN**: Declares TableGen def `int_aarch64_sve_sqsub_x`.
  **L1867 CN**: 声明 TableGen def `int_aarch64_sve_sqsub_x`。
- **L1868 EN**: Declares TableGen def `int_aarch64_sve_uqadd_x`.
  **L1868 CN**: 声明 TableGen def `int_aarch64_sve_uqadd_x`。
- **L1869 EN**: Declares TableGen def `int_aarch64_sve_uqsub_x`.
  **L1869 CN**: 声明 TableGen def `int_aarch64_sve_uqsub_x`。
- **L1870 EN**: Blank line separating nearby declarations or logic blocks.
  **L1870 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1871 EN**: Declares TableGen def `int_aarch64_sve_orqv`.
  **L1871 CN**: 声明 TableGen def `int_aarch64_sve_orqv`。
- **L1872 EN**: Declares TableGen def `int_aarch64_sve_eorqv`.
  **L1872 CN**: 声明 TableGen def `int_aarch64_sve_eorqv`。
- **L1873 EN**: Declares TableGen def `int_aarch64_sve_andqv`.
  **L1873 CN**: 声明 TableGen def `int_aarch64_sve_andqv`。
- **L1874 EN**: Declares TableGen def `int_aarch64_sve_addqv`.
  **L1874 CN**: 声明 TableGen def `int_aarch64_sve_addqv`。
- **L1875 EN**: Declares TableGen def `int_aarch64_sve_smaxqv`.
  **L1875 CN**: 声明 TableGen def `int_aarch64_sve_smaxqv`。
- **L1876 EN**: Declares TableGen def `int_aarch64_sve_umaxqv`.
  **L1876 CN**: 声明 TableGen def `int_aarch64_sve_umaxqv`。
- **L1877 EN**: Declares TableGen def `int_aarch64_sve_sminqv`.
  **L1877 CN**: 声明 TableGen def `int_aarch64_sve_sminqv`。
- **L1878 EN**: Declares TableGen def `int_aarch64_sve_uminqv`.
  **L1878 CN**: 声明 TableGen def `int_aarch64_sve_uminqv`。
- **L1879 EN**: Blank line separating nearby declarations or logic blocks.
  **L1879 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1880 EN**: Comment explains nearby logic, invariants, or intent: `Shifts`.
  **L1880 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Shifts`。
- **L1881 EN**: Blank line separating nearby declarations or logic blocks.
  **L1881 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1882 EN**: Declares TableGen def `int_aarch64_sve_asr`.
  **L1882 CN**: 声明 TableGen def `int_aarch64_sve_asr`。
- **L1883 EN**: Declares TableGen def `int_aarch64_sve_asr_u`.
  **L1883 CN**: 声明 TableGen def `int_aarch64_sve_asr_u`。
- **L1884 EN**: Declares TableGen def `int_aarch64_sve_asr_wide`.
  **L1884 CN**: 声明 TableGen def `int_aarch64_sve_asr_wide`。
- **L1885 EN**: Declares TableGen def `int_aarch64_sve_asrd`.
  **L1885 CN**: 声明 TableGen def `int_aarch64_sve_asrd`。
- **L1886 EN**: Declares TableGen def `int_aarch64_sve_insr`.
  **L1886 CN**: 声明 TableGen def `int_aarch64_sve_insr`。
- **L1887 EN**: Declares TableGen def `int_aarch64_sve_lsl`.
  **L1887 CN**: 声明 TableGen def `int_aarch64_sve_lsl`。
- **L1888 EN**: Declares TableGen def `int_aarch64_sve_lsl_u`.
  **L1888 CN**: 声明 TableGen def `int_aarch64_sve_lsl_u`。

### Lines 1889-1920

````tablegen
def int_aarch64_sve_lsl_wide : AdvSIMD_SVE_ShiftWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lsr      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lsr_u    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lsr_wide : AdvSIMD_SVE_ShiftWide_Intrinsic<[IntrSpeculatable]>;

//
// Integer comparisons
//

def int_aarch64_sve_cmpeq : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmpge : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmpgt : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmphi : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmphs : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmpne : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_cmpeq_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmpge_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmpgt_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmphi_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmphs_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmple_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmplo_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmpls_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmplt_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmpne_wide : AdvSIMD_SVE_CompareWide_Intrinsic<[IntrSpeculatable]>;

//
// Counting bits
//

def int_aarch64_sve_cls : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L1889 EN**: Declares TableGen def `int_aarch64_sve_lsl_wide`.
  **L1889 CN**: 声明 TableGen def `int_aarch64_sve_lsl_wide`。
- **L1890 EN**: Declares TableGen def `int_aarch64_sve_lsr`.
  **L1890 CN**: 声明 TableGen def `int_aarch64_sve_lsr`。
- **L1891 EN**: Declares TableGen def `int_aarch64_sve_lsr_u`.
  **L1891 CN**: 声明 TableGen def `int_aarch64_sve_lsr_u`。
- **L1892 EN**: Declares TableGen def `int_aarch64_sve_lsr_wide`.
  **L1892 CN**: 声明 TableGen def `int_aarch64_sve_lsr_wide`。
- **L1893 EN**: Blank line separating nearby declarations or logic blocks.
  **L1893 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1894 EN**: Separator comment used for visual grouping.
  **L1894 CN**: 用于视觉分组的分隔注释。
- **L1895 EN**: Comment explains nearby logic, invariants, or intent: `Integer comparisons`.
  **L1895 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer comparisons`。
- **L1896 EN**: Separator comment used for visual grouping.
  **L1896 CN**: 用于视觉分组的分隔注释。
- **L1897 EN**: Blank line separating nearby declarations or logic blocks.
  **L1897 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1898 EN**: Declares TableGen def `int_aarch64_sve_cmpeq`.
  **L1898 CN**: 声明 TableGen def `int_aarch64_sve_cmpeq`。
- **L1899 EN**: Declares TableGen def `int_aarch64_sve_cmpge`.
  **L1899 CN**: 声明 TableGen def `int_aarch64_sve_cmpge`。
- **L1900 EN**: Declares TableGen def `int_aarch64_sve_cmpgt`.
  **L1900 CN**: 声明 TableGen def `int_aarch64_sve_cmpgt`。
- **L1901 EN**: Declares TableGen def `int_aarch64_sve_cmphi`.
  **L1901 CN**: 声明 TableGen def `int_aarch64_sve_cmphi`。
- **L1902 EN**: Declares TableGen def `int_aarch64_sve_cmphs`.
  **L1902 CN**: 声明 TableGen def `int_aarch64_sve_cmphs`。
- **L1903 EN**: Declares TableGen def `int_aarch64_sve_cmpne`.
  **L1903 CN**: 声明 TableGen def `int_aarch64_sve_cmpne`。
- **L1904 EN**: Blank line separating nearby declarations or logic blocks.
  **L1904 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1905 EN**: Declares TableGen def `int_aarch64_sve_cmpeq_wide`.
  **L1905 CN**: 声明 TableGen def `int_aarch64_sve_cmpeq_wide`。
- **L1906 EN**: Declares TableGen def `int_aarch64_sve_cmpge_wide`.
  **L1906 CN**: 声明 TableGen def `int_aarch64_sve_cmpge_wide`。
- **L1907 EN**: Declares TableGen def `int_aarch64_sve_cmpgt_wide`.
  **L1907 CN**: 声明 TableGen def `int_aarch64_sve_cmpgt_wide`。
- **L1908 EN**: Declares TableGen def `int_aarch64_sve_cmphi_wide`.
  **L1908 CN**: 声明 TableGen def `int_aarch64_sve_cmphi_wide`。
- **L1909 EN**: Declares TableGen def `int_aarch64_sve_cmphs_wide`.
  **L1909 CN**: 声明 TableGen def `int_aarch64_sve_cmphs_wide`。
- **L1910 EN**: Declares TableGen def `int_aarch64_sve_cmple_wide`.
  **L1910 CN**: 声明 TableGen def `int_aarch64_sve_cmple_wide`。
- **L1911 EN**: Declares TableGen def `int_aarch64_sve_cmplo_wide`.
  **L1911 CN**: 声明 TableGen def `int_aarch64_sve_cmplo_wide`。
- **L1912 EN**: Declares TableGen def `int_aarch64_sve_cmpls_wide`.
  **L1912 CN**: 声明 TableGen def `int_aarch64_sve_cmpls_wide`。
- **L1913 EN**: Declares TableGen def `int_aarch64_sve_cmplt_wide`.
  **L1913 CN**: 声明 TableGen def `int_aarch64_sve_cmplt_wide`。
- **L1914 EN**: Declares TableGen def `int_aarch64_sve_cmpne_wide`.
  **L1914 CN**: 声明 TableGen def `int_aarch64_sve_cmpne_wide`。
- **L1915 EN**: Blank line separating nearby declarations or logic blocks.
  **L1915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1916 EN**: Separator comment used for visual grouping.
  **L1916 CN**: 用于视觉分组的分隔注释。
- **L1917 EN**: Comment explains nearby logic, invariants, or intent: `Counting bits`.
  **L1917 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Counting bits`。
- **L1918 EN**: Separator comment used for visual grouping.
  **L1918 CN**: 用于视觉分组的分隔注释。
- **L1919 EN**: Blank line separating nearby declarations or logic blocks.
  **L1919 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1920 EN**: Declares TableGen def `int_aarch64_sve_cls`.
  **L1920 CN**: 声明 TableGen def `int_aarch64_sve_cls`。

### Lines 1921-1952

````tablegen
def int_aarch64_sve_clz : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cnt : AdvSIMD_SVE_CNT_Intrinsic<[IntrSpeculatable]>;

//
// Counting elements
//

def int_aarch64_sve_cntb : AdvSIMD_SVE_CNTB_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cnth : AdvSIMD_SVE_CNTB_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cntw : AdvSIMD_SVE_CNTB_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cntd : AdvSIMD_SVE_CNTB_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_cntp : AdvSIMD_SVE_CNTP_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_firstp : AdvSIMD_SVE_CNTP_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lastp  : AdvSIMD_SVE_CNTP_Intrinsic<[IntrSpeculatable]>;

//
// FFR manipulation
//

def int_aarch64_sve_rdffr   : ClangBuiltin<"__builtin_sve_svrdffr">,   DefaultAttrsIntrinsic<[llvm_nxv16i1_ty], [], [IntrReadMem, IntrInaccessibleMemOnly]>;
def int_aarch64_sve_rdffr_z : ClangBuiltin<"__builtin_sve_svrdffr_z">, DefaultAttrsIntrinsic<[llvm_nxv16i1_ty], [llvm_nxv16i1_ty], [IntrReadMem, IntrInaccessibleMemOnly]>;
def int_aarch64_sve_setffr  : ClangBuiltin<"__builtin_sve_svsetffr">,  DefaultAttrsIntrinsic<[], [], [IntrWriteMem, IntrInaccessibleMemOnly]>;
def int_aarch64_sve_wrffr   : ClangBuiltin<"__builtin_sve_svwrffr">,   DefaultAttrsIntrinsic<[], [llvm_nxv16i1_ty], [IntrWriteMem, IntrInaccessibleMemOnly]>;

//
// Saturating scalar arithmetic
//

def int_aarch64_sve_sqdech : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdecw : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdecd : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
````
- **L1921 EN**: Declares TableGen def `int_aarch64_sve_clz`.
  **L1921 CN**: 声明 TableGen def `int_aarch64_sve_clz`。
- **L1922 EN**: Declares TableGen def `int_aarch64_sve_cnt`.
  **L1922 CN**: 声明 TableGen def `int_aarch64_sve_cnt`。
- **L1923 EN**: Blank line separating nearby declarations or logic blocks.
  **L1923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1924 EN**: Separator comment used for visual grouping.
  **L1924 CN**: 用于视觉分组的分隔注释。
- **L1925 EN**: Comment explains nearby logic, invariants, or intent: `Counting elements`.
  **L1925 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Counting elements`。
- **L1926 EN**: Separator comment used for visual grouping.
  **L1926 CN**: 用于视觉分组的分隔注释。
- **L1927 EN**: Blank line separating nearby declarations or logic blocks.
  **L1927 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1928 EN**: Declares TableGen def `int_aarch64_sve_cntb`.
  **L1928 CN**: 声明 TableGen def `int_aarch64_sve_cntb`。
- **L1929 EN**: Declares TableGen def `int_aarch64_sve_cnth`.
  **L1929 CN**: 声明 TableGen def `int_aarch64_sve_cnth`。
- **L1930 EN**: Declares TableGen def `int_aarch64_sve_cntw`.
  **L1930 CN**: 声明 TableGen def `int_aarch64_sve_cntw`。
- **L1931 EN**: Declares TableGen def `int_aarch64_sve_cntd`.
  **L1931 CN**: 声明 TableGen def `int_aarch64_sve_cntd`。
- **L1932 EN**: Blank line separating nearby declarations or logic blocks.
  **L1932 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1933 EN**: Declares TableGen def `int_aarch64_sve_cntp`.
  **L1933 CN**: 声明 TableGen def `int_aarch64_sve_cntp`。
- **L1934 EN**: Declares TableGen def `int_aarch64_sve_firstp`.
  **L1934 CN**: 声明 TableGen def `int_aarch64_sve_firstp`。
- **L1935 EN**: Declares TableGen def `int_aarch64_sve_lastp`.
  **L1935 CN**: 声明 TableGen def `int_aarch64_sve_lastp`。
- **L1936 EN**: Blank line separating nearby declarations or logic blocks.
  **L1936 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1937 EN**: Separator comment used for visual grouping.
  **L1937 CN**: 用于视觉分组的分隔注释。
- **L1938 EN**: Comment explains nearby logic, invariants, or intent: `FFR manipulation`.
  **L1938 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FFR manipulation`。
- **L1939 EN**: Separator comment used for visual grouping.
  **L1939 CN**: 用于视觉分组的分隔注释。
- **L1940 EN**: Blank line separating nearby declarations or logic blocks.
  **L1940 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1941 EN**: Declares TableGen def `int_aarch64_sve_rdffr`.
  **L1941 CN**: 声明 TableGen def `int_aarch64_sve_rdffr`。
- **L1942 EN**: Declares TableGen def `int_aarch64_sve_rdffr_z`.
  **L1942 CN**: 声明 TableGen def `int_aarch64_sve_rdffr_z`。
- **L1943 EN**: Declares TableGen def `int_aarch64_sve_setffr`.
  **L1943 CN**: 声明 TableGen def `int_aarch64_sve_setffr`。
- **L1944 EN**: Declares TableGen def `int_aarch64_sve_wrffr`.
  **L1944 CN**: 声明 TableGen def `int_aarch64_sve_wrffr`。
- **L1945 EN**: Blank line separating nearby declarations or logic blocks.
  **L1945 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1946 EN**: Separator comment used for visual grouping.
  **L1946 CN**: 用于视觉分组的分隔注释。
- **L1947 EN**: Comment explains nearby logic, invariants, or intent: `Saturating scalar arithmetic`.
  **L1947 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating scalar arithmetic`。
- **L1948 EN**: Separator comment used for visual grouping.
  **L1948 CN**: 用于视觉分组的分隔注释。
- **L1949 EN**: Blank line separating nearby declarations or logic blocks.
  **L1949 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1950 EN**: Declares TableGen def `int_aarch64_sve_sqdech`.
  **L1950 CN**: 声明 TableGen def `int_aarch64_sve_sqdech`。
- **L1951 EN**: Declares TableGen def `int_aarch64_sve_sqdecw`.
  **L1951 CN**: 声明 TableGen def `int_aarch64_sve_sqdecw`。
- **L1952 EN**: Declares TableGen def `int_aarch64_sve_sqdecd`.
  **L1952 CN**: 声明 TableGen def `int_aarch64_sve_sqdecd`。

### Lines 1953-1984

````tablegen
def int_aarch64_sve_sqdecp : AdvSIMD_SVE_Saturating_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sqdecb_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdecb_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdech_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdech_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdecw_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdecw_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdecd_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdecd_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdecp_n32 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqdecp_n64 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;

def int_aarch64_sve_sqinch : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqincw : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqincd : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqincp : AdvSIMD_SVE_Saturating_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sqincb_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqincb_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqinch_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqinch_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqincw_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqincw_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqincd_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqincd_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqincp_n32 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_sqincp_n64 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;

def int_aarch64_sve_uqdech : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqdecw : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqdecd : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
````
- **L1953 EN**: Declares TableGen def `int_aarch64_sve_sqdecp`.
  **L1953 CN**: 声明 TableGen def `int_aarch64_sve_sqdecp`。
- **L1954 EN**: Blank line separating nearby declarations or logic blocks.
  **L1954 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1955 EN**: Declares TableGen def `int_aarch64_sve_sqdecb_n32`.
  **L1955 CN**: 声明 TableGen def `int_aarch64_sve_sqdecb_n32`。
- **L1956 EN**: Declares TableGen def `int_aarch64_sve_sqdecb_n64`.
  **L1956 CN**: 声明 TableGen def `int_aarch64_sve_sqdecb_n64`。
- **L1957 EN**: Declares TableGen def `int_aarch64_sve_sqdech_n32`.
  **L1957 CN**: 声明 TableGen def `int_aarch64_sve_sqdech_n32`。
- **L1958 EN**: Declares TableGen def `int_aarch64_sve_sqdech_n64`.
  **L1958 CN**: 声明 TableGen def `int_aarch64_sve_sqdech_n64`。
- **L1959 EN**: Declares TableGen def `int_aarch64_sve_sqdecw_n32`.
  **L1959 CN**: 声明 TableGen def `int_aarch64_sve_sqdecw_n32`。
- **L1960 EN**: Declares TableGen def `int_aarch64_sve_sqdecw_n64`.
  **L1960 CN**: 声明 TableGen def `int_aarch64_sve_sqdecw_n64`。
- **L1961 EN**: Declares TableGen def `int_aarch64_sve_sqdecd_n32`.
  **L1961 CN**: 声明 TableGen def `int_aarch64_sve_sqdecd_n32`。
- **L1962 EN**: Declares TableGen def `int_aarch64_sve_sqdecd_n64`.
  **L1962 CN**: 声明 TableGen def `int_aarch64_sve_sqdecd_n64`。
- **L1963 EN**: Declares TableGen def `int_aarch64_sve_sqdecp_n32`.
  **L1963 CN**: 声明 TableGen def `int_aarch64_sve_sqdecp_n32`。
- **L1964 EN**: Declares TableGen def `int_aarch64_sve_sqdecp_n64`.
  **L1964 CN**: 声明 TableGen def `int_aarch64_sve_sqdecp_n64`。
- **L1965 EN**: Blank line separating nearby declarations or logic blocks.
  **L1965 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1966 EN**: Declares TableGen def `int_aarch64_sve_sqinch`.
  **L1966 CN**: 声明 TableGen def `int_aarch64_sve_sqinch`。
- **L1967 EN**: Declares TableGen def `int_aarch64_sve_sqincw`.
  **L1967 CN**: 声明 TableGen def `int_aarch64_sve_sqincw`。
- **L1968 EN**: Declares TableGen def `int_aarch64_sve_sqincd`.
  **L1968 CN**: 声明 TableGen def `int_aarch64_sve_sqincd`。
- **L1969 EN**: Declares TableGen def `int_aarch64_sve_sqincp`.
  **L1969 CN**: 声明 TableGen def `int_aarch64_sve_sqincp`。
- **L1970 EN**: Blank line separating nearby declarations or logic blocks.
  **L1970 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1971 EN**: Declares TableGen def `int_aarch64_sve_sqincb_n32`.
  **L1971 CN**: 声明 TableGen def `int_aarch64_sve_sqincb_n32`。
- **L1972 EN**: Declares TableGen def `int_aarch64_sve_sqincb_n64`.
  **L1972 CN**: 声明 TableGen def `int_aarch64_sve_sqincb_n64`。
- **L1973 EN**: Declares TableGen def `int_aarch64_sve_sqinch_n32`.
  **L1973 CN**: 声明 TableGen def `int_aarch64_sve_sqinch_n32`。
- **L1974 EN**: Declares TableGen def `int_aarch64_sve_sqinch_n64`.
  **L1974 CN**: 声明 TableGen def `int_aarch64_sve_sqinch_n64`。
- **L1975 EN**: Declares TableGen def `int_aarch64_sve_sqincw_n32`.
  **L1975 CN**: 声明 TableGen def `int_aarch64_sve_sqincw_n32`。
- **L1976 EN**: Declares TableGen def `int_aarch64_sve_sqincw_n64`.
  **L1976 CN**: 声明 TableGen def `int_aarch64_sve_sqincw_n64`。
- **L1977 EN**: Declares TableGen def `int_aarch64_sve_sqincd_n32`.
  **L1977 CN**: 声明 TableGen def `int_aarch64_sve_sqincd_n32`。
- **L1978 EN**: Declares TableGen def `int_aarch64_sve_sqincd_n64`.
  **L1978 CN**: 声明 TableGen def `int_aarch64_sve_sqincd_n64`。
- **L1979 EN**: Declares TableGen def `int_aarch64_sve_sqincp_n32`.
  **L1979 CN**: 声明 TableGen def `int_aarch64_sve_sqincp_n32`。
- **L1980 EN**: Declares TableGen def `int_aarch64_sve_sqincp_n64`.
  **L1980 CN**: 声明 TableGen def `int_aarch64_sve_sqincp_n64`。
- **L1981 EN**: Blank line separating nearby declarations or logic blocks.
  **L1981 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1982 EN**: Declares TableGen def `int_aarch64_sve_uqdech`.
  **L1982 CN**: 声明 TableGen def `int_aarch64_sve_uqdech`。
- **L1983 EN**: Declares TableGen def `int_aarch64_sve_uqdecw`.
  **L1983 CN**: 声明 TableGen def `int_aarch64_sve_uqdecw`。
- **L1984 EN**: Declares TableGen def `int_aarch64_sve_uqdecd`.
  **L1984 CN**: 声明 TableGen def `int_aarch64_sve_uqdecd`。

### Lines 1985-2016

````tablegen
def int_aarch64_sve_uqdecp : AdvSIMD_SVE_Saturating_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_uqdecb_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdecb_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdech_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdech_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdecw_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdecw_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdecd_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdecd_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdecp_n32 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqdecp_n64 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;

def int_aarch64_sve_uqinch : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqincw : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqincd : AdvSIMD_SVE_SaturatingWithPattern_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqincp : AdvSIMD_SVE_Saturating_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_uqincb_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqincb_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqinch_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqinch_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqincw_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqincw_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqincd_n32 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqincd_n64 : AdvSIMD_SVE_SaturatingWithPattern_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqincp_n32 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i32_ty, [IntrSpeculatable]>;
def int_aarch64_sve_uqincp_n64 : AdvSIMD_SVE_Saturating_N_Intrinsic<llvm_i64_ty, [IntrSpeculatable]>;

//
// Reversal
//
````
- **L1985 EN**: Declares TableGen def `int_aarch64_sve_uqdecp`.
  **L1985 CN**: 声明 TableGen def `int_aarch64_sve_uqdecp`。
- **L1986 EN**: Blank line separating nearby declarations or logic blocks.
  **L1986 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1987 EN**: Declares TableGen def `int_aarch64_sve_uqdecb_n32`.
  **L1987 CN**: 声明 TableGen def `int_aarch64_sve_uqdecb_n32`。
- **L1988 EN**: Declares TableGen def `int_aarch64_sve_uqdecb_n64`.
  **L1988 CN**: 声明 TableGen def `int_aarch64_sve_uqdecb_n64`。
- **L1989 EN**: Declares TableGen def `int_aarch64_sve_uqdech_n32`.
  **L1989 CN**: 声明 TableGen def `int_aarch64_sve_uqdech_n32`。
- **L1990 EN**: Declares TableGen def `int_aarch64_sve_uqdech_n64`.
  **L1990 CN**: 声明 TableGen def `int_aarch64_sve_uqdech_n64`。
- **L1991 EN**: Declares TableGen def `int_aarch64_sve_uqdecw_n32`.
  **L1991 CN**: 声明 TableGen def `int_aarch64_sve_uqdecw_n32`。
- **L1992 EN**: Declares TableGen def `int_aarch64_sve_uqdecw_n64`.
  **L1992 CN**: 声明 TableGen def `int_aarch64_sve_uqdecw_n64`。
- **L1993 EN**: Declares TableGen def `int_aarch64_sve_uqdecd_n32`.
  **L1993 CN**: 声明 TableGen def `int_aarch64_sve_uqdecd_n32`。
- **L1994 EN**: Declares TableGen def `int_aarch64_sve_uqdecd_n64`.
  **L1994 CN**: 声明 TableGen def `int_aarch64_sve_uqdecd_n64`。
- **L1995 EN**: Declares TableGen def `int_aarch64_sve_uqdecp_n32`.
  **L1995 CN**: 声明 TableGen def `int_aarch64_sve_uqdecp_n32`。
- **L1996 EN**: Declares TableGen def `int_aarch64_sve_uqdecp_n64`.
  **L1996 CN**: 声明 TableGen def `int_aarch64_sve_uqdecp_n64`。
- **L1997 EN**: Blank line separating nearby declarations or logic blocks.
  **L1997 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1998 EN**: Declares TableGen def `int_aarch64_sve_uqinch`.
  **L1998 CN**: 声明 TableGen def `int_aarch64_sve_uqinch`。
- **L1999 EN**: Declares TableGen def `int_aarch64_sve_uqincw`.
  **L1999 CN**: 声明 TableGen def `int_aarch64_sve_uqincw`。
- **L2000 EN**: Declares TableGen def `int_aarch64_sve_uqincd`.
  **L2000 CN**: 声明 TableGen def `int_aarch64_sve_uqincd`。
- **L2001 EN**: Declares TableGen def `int_aarch64_sve_uqincp`.
  **L2001 CN**: 声明 TableGen def `int_aarch64_sve_uqincp`。
- **L2002 EN**: Blank line separating nearby declarations or logic blocks.
  **L2002 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2003 EN**: Declares TableGen def `int_aarch64_sve_uqincb_n32`.
  **L2003 CN**: 声明 TableGen def `int_aarch64_sve_uqincb_n32`。
- **L2004 EN**: Declares TableGen def `int_aarch64_sve_uqincb_n64`.
  **L2004 CN**: 声明 TableGen def `int_aarch64_sve_uqincb_n64`。
- **L2005 EN**: Declares TableGen def `int_aarch64_sve_uqinch_n32`.
  **L2005 CN**: 声明 TableGen def `int_aarch64_sve_uqinch_n32`。
- **L2006 EN**: Declares TableGen def `int_aarch64_sve_uqinch_n64`.
  **L2006 CN**: 声明 TableGen def `int_aarch64_sve_uqinch_n64`。
- **L2007 EN**: Declares TableGen def `int_aarch64_sve_uqincw_n32`.
  **L2007 CN**: 声明 TableGen def `int_aarch64_sve_uqincw_n32`。
- **L2008 EN**: Declares TableGen def `int_aarch64_sve_uqincw_n64`.
  **L2008 CN**: 声明 TableGen def `int_aarch64_sve_uqincw_n64`。
- **L2009 EN**: Declares TableGen def `int_aarch64_sve_uqincd_n32`.
  **L2009 CN**: 声明 TableGen def `int_aarch64_sve_uqincd_n32`。
- **L2010 EN**: Declares TableGen def `int_aarch64_sve_uqincd_n64`.
  **L2010 CN**: 声明 TableGen def `int_aarch64_sve_uqincd_n64`。
- **L2011 EN**: Declares TableGen def `int_aarch64_sve_uqincp_n32`.
  **L2011 CN**: 声明 TableGen def `int_aarch64_sve_uqincp_n32`。
- **L2012 EN**: Declares TableGen def `int_aarch64_sve_uqincp_n64`.
  **L2012 CN**: 声明 TableGen def `int_aarch64_sve_uqincp_n64`。
- **L2013 EN**: Blank line separating nearby declarations or logic blocks.
  **L2013 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2014 EN**: Separator comment used for visual grouping.
  **L2014 CN**: 用于视觉分组的分隔注释。
- **L2015 EN**: Comment explains nearby logic, invariants, or intent: `Reversal`.
  **L2015 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reversal`。
- **L2016 EN**: Separator comment used for visual grouping.
  **L2016 CN**: 用于视觉分组的分隔注释。

### Lines 2017-2048

````tablegen

def int_aarch64_sve_rbit : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_revb : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_revh : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_revw : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// Permutations and selection
//

def int_aarch64_sve_clasta    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_clasta_n  : AdvSIMD_SVE_ReduceWithInit_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_clastb    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_clastb_n  : AdvSIMD_SVE_ReduceWithInit_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_compact   : AdvSIMD_Pred1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_expand    : AdvSIMD_Pred1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_dupq_lane : AdvSIMD_SVE_DUPQ_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_dup_laneq : SVE2_1VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ext       : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sel       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lasta     : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_lastb     : AdvSIMD_SVE_Reduce_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_rev_b16   : AdvSIMD_SVE_2SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_rev_b32   : AdvSIMD_SVE_2SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_rev_b64   : AdvSIMD_SVE_2SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_splice    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sunpkhi   : AdvSIMD_SVE_Unpack_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sunpklo   : AdvSIMD_SVE_Unpack_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_tbl       : AdvSIMD_SVE_TBL_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn1      : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn1_b16  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn1_b32  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
````
- **L2017 EN**: Blank line separating nearby declarations or logic blocks.
  **L2017 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2018 EN**: Declares TableGen def `int_aarch64_sve_rbit`.
  **L2018 CN**: 声明 TableGen def `int_aarch64_sve_rbit`。
- **L2019 EN**: Declares TableGen def `int_aarch64_sve_revb`.
  **L2019 CN**: 声明 TableGen def `int_aarch64_sve_revb`。
- **L2020 EN**: Declares TableGen def `int_aarch64_sve_revh`.
  **L2020 CN**: 声明 TableGen def `int_aarch64_sve_revh`。
- **L2021 EN**: Declares TableGen def `int_aarch64_sve_revw`.
  **L2021 CN**: 声明 TableGen def `int_aarch64_sve_revw`。
- **L2022 EN**: Blank line separating nearby declarations or logic blocks.
  **L2022 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2023 EN**: Separator comment used for visual grouping.
  **L2023 CN**: 用于视觉分组的分隔注释。
- **L2024 EN**: Comment explains nearby logic, invariants, or intent: `Permutations and selection`.
  **L2024 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Permutations and selection`。
- **L2025 EN**: Separator comment used for visual grouping.
  **L2025 CN**: 用于视觉分组的分隔注释。
- **L2026 EN**: Blank line separating nearby declarations or logic blocks.
  **L2026 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2027 EN**: Declares TableGen def `int_aarch64_sve_clasta`.
  **L2027 CN**: 声明 TableGen def `int_aarch64_sve_clasta`。
- **L2028 EN**: Declares TableGen def `int_aarch64_sve_clasta_n`.
  **L2028 CN**: 声明 TableGen def `int_aarch64_sve_clasta_n`。
- **L2029 EN**: Declares TableGen def `int_aarch64_sve_clastb`.
  **L2029 CN**: 声明 TableGen def `int_aarch64_sve_clastb`。
- **L2030 EN**: Declares TableGen def `int_aarch64_sve_clastb_n`.
  **L2030 CN**: 声明 TableGen def `int_aarch64_sve_clastb_n`。
- **L2031 EN**: Declares TableGen def `int_aarch64_sve_compact`.
  **L2031 CN**: 声明 TableGen def `int_aarch64_sve_compact`。
- **L2032 EN**: Declares TableGen def `int_aarch64_sve_expand`.
  **L2032 CN**: 声明 TableGen def `int_aarch64_sve_expand`。
- **L2033 EN**: Declares TableGen def `int_aarch64_sve_dupq_lane`.
  **L2033 CN**: 声明 TableGen def `int_aarch64_sve_dupq_lane`。
- **L2034 EN**: Declares TableGen def `int_aarch64_sve_dup_laneq`.
  **L2034 CN**: 声明 TableGen def `int_aarch64_sve_dup_laneq`。
- **L2035 EN**: Declares TableGen def `int_aarch64_sve_ext`.
  **L2035 CN**: 声明 TableGen def `int_aarch64_sve_ext`。
- **L2036 EN**: Declares TableGen def `int_aarch64_sve_sel`.
  **L2036 CN**: 声明 TableGen def `int_aarch64_sve_sel`。
- **L2037 EN**: Declares TableGen def `int_aarch64_sve_lasta`.
  **L2037 CN**: 声明 TableGen def `int_aarch64_sve_lasta`。
- **L2038 EN**: Declares TableGen def `int_aarch64_sve_lastb`.
  **L2038 CN**: 声明 TableGen def `int_aarch64_sve_lastb`。
- **L2039 EN**: Declares TableGen def `int_aarch64_sve_rev_b16`.
  **L2039 CN**: 声明 TableGen def `int_aarch64_sve_rev_b16`。
- **L2040 EN**: Declares TableGen def `int_aarch64_sve_rev_b32`.
  **L2040 CN**: 声明 TableGen def `int_aarch64_sve_rev_b32`。
- **L2041 EN**: Declares TableGen def `int_aarch64_sve_rev_b64`.
  **L2041 CN**: 声明 TableGen def `int_aarch64_sve_rev_b64`。
- **L2042 EN**: Declares TableGen def `int_aarch64_sve_splice`.
  **L2042 CN**: 声明 TableGen def `int_aarch64_sve_splice`。
- **L2043 EN**: Declares TableGen def `int_aarch64_sve_sunpkhi`.
  **L2043 CN**: 声明 TableGen def `int_aarch64_sve_sunpkhi`。
- **L2044 EN**: Declares TableGen def `int_aarch64_sve_sunpklo`.
  **L2044 CN**: 声明 TableGen def `int_aarch64_sve_sunpklo`。
- **L2045 EN**: Declares TableGen def `int_aarch64_sve_tbl`.
  **L2045 CN**: 声明 TableGen def `int_aarch64_sve_tbl`。
- **L2046 EN**: Declares TableGen def `int_aarch64_sve_trn1`.
  **L2046 CN**: 声明 TableGen def `int_aarch64_sve_trn1`。
- **L2047 EN**: Declares TableGen def `int_aarch64_sve_trn1_b16`.
  **L2047 CN**: 声明 TableGen def `int_aarch64_sve_trn1_b16`。
- **L2048 EN**: Declares TableGen def `int_aarch64_sve_trn1_b32`.
  **L2048 CN**: 声明 TableGen def `int_aarch64_sve_trn1_b32`。

### Lines 2049-2080

````tablegen
def int_aarch64_sve_trn1_b64  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn2      : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn2_b16  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn2_b32  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn2_b64  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn1q     : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_trn2q     : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uunpkhi   : AdvSIMD_SVE_Unpack_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uunpklo   : AdvSIMD_SVE_Unpack_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp1      : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp1_b16  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp1_b32  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp1_b64  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp2      : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp2_b16  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp2_b32  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp2_b64  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp1q     : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzp2q     : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip1      : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip1_b16  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip1_b32  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip1_b64  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip2      : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip2_b16  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip2_b32  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip2_b64  : AdvSIMD_SVE_3SVBoolArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip1q     : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zip2q     : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// Logical operations
````
- **L2049 EN**: Declares TableGen def `int_aarch64_sve_trn1_b64`.
  **L2049 CN**: 声明 TableGen def `int_aarch64_sve_trn1_b64`。
- **L2050 EN**: Declares TableGen def `int_aarch64_sve_trn2`.
  **L2050 CN**: 声明 TableGen def `int_aarch64_sve_trn2`。
- **L2051 EN**: Declares TableGen def `int_aarch64_sve_trn2_b16`.
  **L2051 CN**: 声明 TableGen def `int_aarch64_sve_trn2_b16`。
- **L2052 EN**: Declares TableGen def `int_aarch64_sve_trn2_b32`.
  **L2052 CN**: 声明 TableGen def `int_aarch64_sve_trn2_b32`。
- **L2053 EN**: Declares TableGen def `int_aarch64_sve_trn2_b64`.
  **L2053 CN**: 声明 TableGen def `int_aarch64_sve_trn2_b64`。
- **L2054 EN**: Declares TableGen def `int_aarch64_sve_trn1q`.
  **L2054 CN**: 声明 TableGen def `int_aarch64_sve_trn1q`。
- **L2055 EN**: Declares TableGen def `int_aarch64_sve_trn2q`.
  **L2055 CN**: 声明 TableGen def `int_aarch64_sve_trn2q`。
- **L2056 EN**: Declares TableGen def `int_aarch64_sve_uunpkhi`.
  **L2056 CN**: 声明 TableGen def `int_aarch64_sve_uunpkhi`。
- **L2057 EN**: Declares TableGen def `int_aarch64_sve_uunpklo`.
  **L2057 CN**: 声明 TableGen def `int_aarch64_sve_uunpklo`。
- **L2058 EN**: Declares TableGen def `int_aarch64_sve_uzp1`.
  **L2058 CN**: 声明 TableGen def `int_aarch64_sve_uzp1`。
- **L2059 EN**: Declares TableGen def `int_aarch64_sve_uzp1_b16`.
  **L2059 CN**: 声明 TableGen def `int_aarch64_sve_uzp1_b16`。
- **L2060 EN**: Declares TableGen def `int_aarch64_sve_uzp1_b32`.
  **L2060 CN**: 声明 TableGen def `int_aarch64_sve_uzp1_b32`。
- **L2061 EN**: Declares TableGen def `int_aarch64_sve_uzp1_b64`.
  **L2061 CN**: 声明 TableGen def `int_aarch64_sve_uzp1_b64`。
- **L2062 EN**: Declares TableGen def `int_aarch64_sve_uzp2`.
  **L2062 CN**: 声明 TableGen def `int_aarch64_sve_uzp2`。
- **L2063 EN**: Declares TableGen def `int_aarch64_sve_uzp2_b16`.
  **L2063 CN**: 声明 TableGen def `int_aarch64_sve_uzp2_b16`。
- **L2064 EN**: Declares TableGen def `int_aarch64_sve_uzp2_b32`.
  **L2064 CN**: 声明 TableGen def `int_aarch64_sve_uzp2_b32`。
- **L2065 EN**: Declares TableGen def `int_aarch64_sve_uzp2_b64`.
  **L2065 CN**: 声明 TableGen def `int_aarch64_sve_uzp2_b64`。
- **L2066 EN**: Declares TableGen def `int_aarch64_sve_uzp1q`.
  **L2066 CN**: 声明 TableGen def `int_aarch64_sve_uzp1q`。
- **L2067 EN**: Declares TableGen def `int_aarch64_sve_uzp2q`.
  **L2067 CN**: 声明 TableGen def `int_aarch64_sve_uzp2q`。
- **L2068 EN**: Declares TableGen def `int_aarch64_sve_zip1`.
  **L2068 CN**: 声明 TableGen def `int_aarch64_sve_zip1`。
- **L2069 EN**: Declares TableGen def `int_aarch64_sve_zip1_b16`.
  **L2069 CN**: 声明 TableGen def `int_aarch64_sve_zip1_b16`。
- **L2070 EN**: Declares TableGen def `int_aarch64_sve_zip1_b32`.
  **L2070 CN**: 声明 TableGen def `int_aarch64_sve_zip1_b32`。
- **L2071 EN**: Declares TableGen def `int_aarch64_sve_zip1_b64`.
  **L2071 CN**: 声明 TableGen def `int_aarch64_sve_zip1_b64`。
- **L2072 EN**: Declares TableGen def `int_aarch64_sve_zip2`.
  **L2072 CN**: 声明 TableGen def `int_aarch64_sve_zip2`。
- **L2073 EN**: Declares TableGen def `int_aarch64_sve_zip2_b16`.
  **L2073 CN**: 声明 TableGen def `int_aarch64_sve_zip2_b16`。
- **L2074 EN**: Declares TableGen def `int_aarch64_sve_zip2_b32`.
  **L2074 CN**: 声明 TableGen def `int_aarch64_sve_zip2_b32`。
- **L2075 EN**: Declares TableGen def `int_aarch64_sve_zip2_b64`.
  **L2075 CN**: 声明 TableGen def `int_aarch64_sve_zip2_b64`。
- **L2076 EN**: Declares TableGen def `int_aarch64_sve_zip1q`.
  **L2076 CN**: 声明 TableGen def `int_aarch64_sve_zip1q`。
- **L2077 EN**: Declares TableGen def `int_aarch64_sve_zip2q`.
  **L2077 CN**: 声明 TableGen def `int_aarch64_sve_zip2q`。
- **L2078 EN**: Blank line separating nearby declarations or logic blocks.
  **L2078 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2079 EN**: Separator comment used for visual grouping.
  **L2079 CN**: 用于视觉分组的分隔注释。
- **L2080 EN**: Comment explains nearby logic, invariants, or intent: `Logical operations`.
  **L2080 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Logical operations`。

### Lines 2081-2112

````tablegen
//

def int_aarch64_sve_and  : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_and_u: AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bic  : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bic_u: AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cnot : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_eor  : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_eor_u: AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_not  : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_orr  : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_orr_u: AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// Conversion
//

def int_aarch64_sve_sxtb : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sxth : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sxtw : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uxtb : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uxth : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uxtw : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// While comparisons
//

def int_aarch64_sve_whilele : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilelo : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilels : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilelt : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;
````
- **L2081 EN**: Separator comment used for visual grouping.
  **L2081 CN**: 用于视觉分组的分隔注释。
- **L2082 EN**: Blank line separating nearby declarations or logic blocks.
  **L2082 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2083 EN**: Declares TableGen def `int_aarch64_sve_and`.
  **L2083 CN**: 声明 TableGen def `int_aarch64_sve_and`。
- **L2084 EN**: Declares TableGen def `int_aarch64_sve_and_u`.
  **L2084 CN**: 声明 TableGen def `int_aarch64_sve_and_u`。
- **L2085 EN**: Declares TableGen def `int_aarch64_sve_bic`.
  **L2085 CN**: 声明 TableGen def `int_aarch64_sve_bic`。
- **L2086 EN**: Declares TableGen def `int_aarch64_sve_bic_u`.
  **L2086 CN**: 声明 TableGen def `int_aarch64_sve_bic_u`。
- **L2087 EN**: Declares TableGen def `int_aarch64_sve_cnot`.
  **L2087 CN**: 声明 TableGen def `int_aarch64_sve_cnot`。
- **L2088 EN**: Declares TableGen def `int_aarch64_sve_eor`.
  **L2088 CN**: 声明 TableGen def `int_aarch64_sve_eor`。
- **L2089 EN**: Declares TableGen def `int_aarch64_sve_eor_u`.
  **L2089 CN**: 声明 TableGen def `int_aarch64_sve_eor_u`。
- **L2090 EN**: Declares TableGen def `int_aarch64_sve_not`.
  **L2090 CN**: 声明 TableGen def `int_aarch64_sve_not`。
- **L2091 EN**: Declares TableGen def `int_aarch64_sve_orr`.
  **L2091 CN**: 声明 TableGen def `int_aarch64_sve_orr`。
- **L2092 EN**: Declares TableGen def `int_aarch64_sve_orr_u`.
  **L2092 CN**: 声明 TableGen def `int_aarch64_sve_orr_u`。
- **L2093 EN**: Blank line separating nearby declarations or logic blocks.
  **L2093 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2094 EN**: Separator comment used for visual grouping.
  **L2094 CN**: 用于视觉分组的分隔注释。
- **L2095 EN**: Comment explains nearby logic, invariants, or intent: `Conversion`.
  **L2095 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Conversion`。
- **L2096 EN**: Separator comment used for visual grouping.
  **L2096 CN**: 用于视觉分组的分隔注释。
- **L2097 EN**: Blank line separating nearby declarations or logic blocks.
  **L2097 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2098 EN**: Declares TableGen def `int_aarch64_sve_sxtb`.
  **L2098 CN**: 声明 TableGen def `int_aarch64_sve_sxtb`。
- **L2099 EN**: Declares TableGen def `int_aarch64_sve_sxth`.
  **L2099 CN**: 声明 TableGen def `int_aarch64_sve_sxth`。
- **L2100 EN**: Declares TableGen def `int_aarch64_sve_sxtw`.
  **L2100 CN**: 声明 TableGen def `int_aarch64_sve_sxtw`。
- **L2101 EN**: Declares TableGen def `int_aarch64_sve_uxtb`.
  **L2101 CN**: 声明 TableGen def `int_aarch64_sve_uxtb`。
- **L2102 EN**: Declares TableGen def `int_aarch64_sve_uxth`.
  **L2102 CN**: 声明 TableGen def `int_aarch64_sve_uxth`。
- **L2103 EN**: Declares TableGen def `int_aarch64_sve_uxtw`.
  **L2103 CN**: 声明 TableGen def `int_aarch64_sve_uxtw`。
- **L2104 EN**: Blank line separating nearby declarations or logic blocks.
  **L2104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2105 EN**: Separator comment used for visual grouping.
  **L2105 CN**: 用于视觉分组的分隔注释。
- **L2106 EN**: Comment explains nearby logic, invariants, or intent: `While comparisons`.
  **L2106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`While comparisons`。
- **L2107 EN**: Separator comment used for visual grouping.
  **L2107 CN**: 用于视觉分组的分隔注释。
- **L2108 EN**: Blank line separating nearby declarations or logic blocks.
  **L2108 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2109 EN**: Declares TableGen def `int_aarch64_sve_whilele`.
  **L2109 CN**: 声明 TableGen def `int_aarch64_sve_whilele`。
- **L2110 EN**: Declares TableGen def `int_aarch64_sve_whilelo`.
  **L2110 CN**: 声明 TableGen def `int_aarch64_sve_whilelo`。
- **L2111 EN**: Declares TableGen def `int_aarch64_sve_whilels`.
  **L2111 CN**: 声明 TableGen def `int_aarch64_sve_whilels`。
- **L2112 EN**: Declares TableGen def `int_aarch64_sve_whilelt`.
  **L2112 CN**: 声明 TableGen def `int_aarch64_sve_whilelt`。

### Lines 2113-2144

````tablegen
def int_aarch64_sve_whilege : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilegt : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilehs : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilehi : AdvSIMD_SVE_WHILE_Intrinsic<[IntrSpeculatable]>;

//
// Floating-point arithmetic
//

def int_aarch64_sve_fabd       : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fabd_u     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fabs       : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_fadd       : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fadd_u     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fcadd      : AdvSIMD_SVE_CADD_Intrinsic;
def int_aarch64_sve_fcmla      : AdvSIMD_SVE_CMLA_Intrinsic;
def int_aarch64_sve_fcmla_lane : AdvSIMD_SVE_CMLA_LANE_Intrinsic;
def int_aarch64_sve_fdiv       : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fdiv_u     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fdivr      : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fexpa_x    : AdvSIMD_SVE_EXPA_Intrinsic;
def int_aarch64_sve_fmad       : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fmax       : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmax_u     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmaxnm     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmaxnm_u   : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmin       : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmin_u     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fminnm     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fminnm_u   : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmla       : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fmla_lane  : AdvSIMD_3VectorArgIndexed_Intrinsic;
````
- **L2113 EN**: Declares TableGen def `int_aarch64_sve_whilege`.
  **L2113 CN**: 声明 TableGen def `int_aarch64_sve_whilege`。
- **L2114 EN**: Declares TableGen def `int_aarch64_sve_whilegt`.
  **L2114 CN**: 声明 TableGen def `int_aarch64_sve_whilegt`。
- **L2115 EN**: Declares TableGen def `int_aarch64_sve_whilehs`.
  **L2115 CN**: 声明 TableGen def `int_aarch64_sve_whilehs`。
- **L2116 EN**: Declares TableGen def `int_aarch64_sve_whilehi`.
  **L2116 CN**: 声明 TableGen def `int_aarch64_sve_whilehi`。
- **L2117 EN**: Blank line separating nearby declarations or logic blocks.
  **L2117 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2118 EN**: Separator comment used for visual grouping.
  **L2118 CN**: 用于视觉分组的分隔注释。
- **L2119 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point arithmetic`.
  **L2119 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point arithmetic`。
- **L2120 EN**: Separator comment used for visual grouping.
  **L2120 CN**: 用于视觉分组的分隔注释。
- **L2121 EN**: Blank line separating nearby declarations or logic blocks.
  **L2121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2122 EN**: Declares TableGen def `int_aarch64_sve_fabd`.
  **L2122 CN**: 声明 TableGen def `int_aarch64_sve_fabd`。
- **L2123 EN**: Declares TableGen def `int_aarch64_sve_fabd_u`.
  **L2123 CN**: 声明 TableGen def `int_aarch64_sve_fabd_u`。
- **L2124 EN**: Declares TableGen def `int_aarch64_sve_fabs`.
  **L2124 CN**: 声明 TableGen def `int_aarch64_sve_fabs`。
- **L2125 EN**: Declares TableGen def `int_aarch64_sve_fadd`.
  **L2125 CN**: 声明 TableGen def `int_aarch64_sve_fadd`。
- **L2126 EN**: Declares TableGen def `int_aarch64_sve_fadd_u`.
  **L2126 CN**: 声明 TableGen def `int_aarch64_sve_fadd_u`。
- **L2127 EN**: Declares TableGen def `int_aarch64_sve_fcadd`.
  **L2127 CN**: 声明 TableGen def `int_aarch64_sve_fcadd`。
- **L2128 EN**: Declares TableGen def `int_aarch64_sve_fcmla`.
  **L2128 CN**: 声明 TableGen def `int_aarch64_sve_fcmla`。
- **L2129 EN**: Declares TableGen def `int_aarch64_sve_fcmla_lane`.
  **L2129 CN**: 声明 TableGen def `int_aarch64_sve_fcmla_lane`。
- **L2130 EN**: Declares TableGen def `int_aarch64_sve_fdiv`.
  **L2130 CN**: 声明 TableGen def `int_aarch64_sve_fdiv`。
- **L2131 EN**: Declares TableGen def `int_aarch64_sve_fdiv_u`.
  **L2131 CN**: 声明 TableGen def `int_aarch64_sve_fdiv_u`。
- **L2132 EN**: Declares TableGen def `int_aarch64_sve_fdivr`.
  **L2132 CN**: 声明 TableGen def `int_aarch64_sve_fdivr`。
- **L2133 EN**: Declares TableGen def `int_aarch64_sve_fexpa_x`.
  **L2133 CN**: 声明 TableGen def `int_aarch64_sve_fexpa_x`。
- **L2134 EN**: Declares TableGen def `int_aarch64_sve_fmad`.
  **L2134 CN**: 声明 TableGen def `int_aarch64_sve_fmad`。
- **L2135 EN**: Declares TableGen def `int_aarch64_sve_fmax`.
  **L2135 CN**: 声明 TableGen def `int_aarch64_sve_fmax`。
- **L2136 EN**: Declares TableGen def `int_aarch64_sve_fmax_u`.
  **L2136 CN**: 声明 TableGen def `int_aarch64_sve_fmax_u`。
- **L2137 EN**: Declares TableGen def `int_aarch64_sve_fmaxnm`.
  **L2137 CN**: 声明 TableGen def `int_aarch64_sve_fmaxnm`。
- **L2138 EN**: Declares TableGen def `int_aarch64_sve_fmaxnm_u`.
  **L2138 CN**: 声明 TableGen def `int_aarch64_sve_fmaxnm_u`。
- **L2139 EN**: Declares TableGen def `int_aarch64_sve_fmin`.
  **L2139 CN**: 声明 TableGen def `int_aarch64_sve_fmin`。
- **L2140 EN**: Declares TableGen def `int_aarch64_sve_fmin_u`.
  **L2140 CN**: 声明 TableGen def `int_aarch64_sve_fmin_u`。
- **L2141 EN**: Declares TableGen def `int_aarch64_sve_fminnm`.
  **L2141 CN**: 声明 TableGen def `int_aarch64_sve_fminnm`。
- **L2142 EN**: Declares TableGen def `int_aarch64_sve_fminnm_u`.
  **L2142 CN**: 声明 TableGen def `int_aarch64_sve_fminnm_u`。
- **L2143 EN**: Declares TableGen def `int_aarch64_sve_fmla`.
  **L2143 CN**: 声明 TableGen def `int_aarch64_sve_fmla`。
- **L2144 EN**: Declares TableGen def `int_aarch64_sve_fmla_lane`.
  **L2144 CN**: 声明 TableGen def `int_aarch64_sve_fmla_lane`。

### Lines 2145-2176

````tablegen
def int_aarch64_sve_fmla_u     : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fmls       : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fmls_lane  : AdvSIMD_3VectorArgIndexed_Intrinsic;
def int_aarch64_sve_fmls_u     : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fmsb       : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fmul       : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmul_lane  : AdvSIMD_2VectorArgIndexed_Intrinsic;
def int_aarch64_sve_fmul_u     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmulx      : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmulx_u    : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fneg       : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_fnmad      : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fnmla      : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fnmla_u    : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fnmls      : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fnmls_u    : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_fnmsb      : AdvSIMD_Pred3VectorArg_Intrinsic;
def int_aarch64_sve_frecpe_x   : AdvSIMD_1VectorArg_Intrinsic;
def int_aarch64_sve_frecps_x   : AdvSIMD_2VectorArg_Intrinsic;
def int_aarch64_sve_frecpx     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frinta     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frinti     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frintm     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frintn     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frintp     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frintx     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frintz     : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frint32x   : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frint32z   : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frint64x   : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frint64z   : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_frsqrte_x  : AdvSIMD_1VectorArg_Intrinsic;
````
- **L2145 EN**: Declares TableGen def `int_aarch64_sve_fmla_u`.
  **L2145 CN**: 声明 TableGen def `int_aarch64_sve_fmla_u`。
- **L2146 EN**: Declares TableGen def `int_aarch64_sve_fmls`.
  **L2146 CN**: 声明 TableGen def `int_aarch64_sve_fmls`。
- **L2147 EN**: Declares TableGen def `int_aarch64_sve_fmls_lane`.
  **L2147 CN**: 声明 TableGen def `int_aarch64_sve_fmls_lane`。
- **L2148 EN**: Declares TableGen def `int_aarch64_sve_fmls_u`.
  **L2148 CN**: 声明 TableGen def `int_aarch64_sve_fmls_u`。
- **L2149 EN**: Declares TableGen def `int_aarch64_sve_fmsb`.
  **L2149 CN**: 声明 TableGen def `int_aarch64_sve_fmsb`。
- **L2150 EN**: Declares TableGen def `int_aarch64_sve_fmul`.
  **L2150 CN**: 声明 TableGen def `int_aarch64_sve_fmul`。
- **L2151 EN**: Declares TableGen def `int_aarch64_sve_fmul_lane`.
  **L2151 CN**: 声明 TableGen def `int_aarch64_sve_fmul_lane`。
- **L2152 EN**: Declares TableGen def `int_aarch64_sve_fmul_u`.
  **L2152 CN**: 声明 TableGen def `int_aarch64_sve_fmul_u`。
- **L2153 EN**: Declares TableGen def `int_aarch64_sve_fmulx`.
  **L2153 CN**: 声明 TableGen def `int_aarch64_sve_fmulx`。
- **L2154 EN**: Declares TableGen def `int_aarch64_sve_fmulx_u`.
  **L2154 CN**: 声明 TableGen def `int_aarch64_sve_fmulx_u`。
- **L2155 EN**: Declares TableGen def `int_aarch64_sve_fneg`.
  **L2155 CN**: 声明 TableGen def `int_aarch64_sve_fneg`。
- **L2156 EN**: Declares TableGen def `int_aarch64_sve_fnmad`.
  **L2156 CN**: 声明 TableGen def `int_aarch64_sve_fnmad`。
- **L2157 EN**: Declares TableGen def `int_aarch64_sve_fnmla`.
  **L2157 CN**: 声明 TableGen def `int_aarch64_sve_fnmla`。
- **L2158 EN**: Declares TableGen def `int_aarch64_sve_fnmla_u`.
  **L2158 CN**: 声明 TableGen def `int_aarch64_sve_fnmla_u`。
- **L2159 EN**: Declares TableGen def `int_aarch64_sve_fnmls`.
  **L2159 CN**: 声明 TableGen def `int_aarch64_sve_fnmls`。
- **L2160 EN**: Declares TableGen def `int_aarch64_sve_fnmls_u`.
  **L2160 CN**: 声明 TableGen def `int_aarch64_sve_fnmls_u`。
- **L2161 EN**: Declares TableGen def `int_aarch64_sve_fnmsb`.
  **L2161 CN**: 声明 TableGen def `int_aarch64_sve_fnmsb`。
- **L2162 EN**: Declares TableGen def `int_aarch64_sve_frecpe_x`.
  **L2162 CN**: 声明 TableGen def `int_aarch64_sve_frecpe_x`。
- **L2163 EN**: Declares TableGen def `int_aarch64_sve_frecps_x`.
  **L2163 CN**: 声明 TableGen def `int_aarch64_sve_frecps_x`。
- **L2164 EN**: Declares TableGen def `int_aarch64_sve_frecpx`.
  **L2164 CN**: 声明 TableGen def `int_aarch64_sve_frecpx`。
- **L2165 EN**: Declares TableGen def `int_aarch64_sve_frinta`.
  **L2165 CN**: 声明 TableGen def `int_aarch64_sve_frinta`。
- **L2166 EN**: Declares TableGen def `int_aarch64_sve_frinti`.
  **L2166 CN**: 声明 TableGen def `int_aarch64_sve_frinti`。
- **L2167 EN**: Declares TableGen def `int_aarch64_sve_frintm`.
  **L2167 CN**: 声明 TableGen def `int_aarch64_sve_frintm`。
- **L2168 EN**: Declares TableGen def `int_aarch64_sve_frintn`.
  **L2168 CN**: 声明 TableGen def `int_aarch64_sve_frintn`。
- **L2169 EN**: Declares TableGen def `int_aarch64_sve_frintp`.
  **L2169 CN**: 声明 TableGen def `int_aarch64_sve_frintp`。
- **L2170 EN**: Declares TableGen def `int_aarch64_sve_frintx`.
  **L2170 CN**: 声明 TableGen def `int_aarch64_sve_frintx`。
- **L2171 EN**: Declares TableGen def `int_aarch64_sve_frintz`.
  **L2171 CN**: 声明 TableGen def `int_aarch64_sve_frintz`。
- **L2172 EN**: Declares TableGen def `int_aarch64_sve_frint32x`.
  **L2172 CN**: 声明 TableGen def `int_aarch64_sve_frint32x`。
- **L2173 EN**: Declares TableGen def `int_aarch64_sve_frint32z`.
  **L2173 CN**: 声明 TableGen def `int_aarch64_sve_frint32z`。
- **L2174 EN**: Declares TableGen def `int_aarch64_sve_frint64x`.
  **L2174 CN**: 声明 TableGen def `int_aarch64_sve_frint64x`。
- **L2175 EN**: Declares TableGen def `int_aarch64_sve_frint64z`.
  **L2175 CN**: 声明 TableGen def `int_aarch64_sve_frint64z`。
- **L2176 EN**: Declares TableGen def `int_aarch64_sve_frsqrte_x`.
  **L2176 CN**: 声明 TableGen def `int_aarch64_sve_frsqrte_x`。

### Lines 2177-2208

````tablegen
def int_aarch64_sve_frsqrts_x  : AdvSIMD_2VectorArg_Intrinsic;
def int_aarch64_sve_fscale     : AdvSIMD_SVE_SCALE_Intrinsic;
def int_aarch64_sve_fsqrt      : AdvSIMD_Merged1VectorArg_Intrinsic;
def int_aarch64_sve_fsub       : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fsub_u     : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fsubr      : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_ftmad_x    : AdvSIMD_2VectorArgIndexed_Intrinsic;
def int_aarch64_sve_ftsmul_x   : AdvSIMD_SVE_TSMUL_Intrinsic;
def int_aarch64_sve_ftssel_x   : AdvSIMD_SVE_TSMUL_Intrinsic;

//
// Floating-point reductions
//

def int_aarch64_sve_fadda   : AdvSIMD_SVE_ReduceWithInit_Intrinsic;
def int_aarch64_sve_faddv   : AdvSIMD_SVE_Reduce_Intrinsic;
def int_aarch64_sve_fmaxv   : AdvSIMD_SVE_Reduce_Intrinsic;
def int_aarch64_sve_fmaxnmv : AdvSIMD_SVE_Reduce_Intrinsic;
def int_aarch64_sve_fminv   : AdvSIMD_SVE_Reduce_Intrinsic;
def int_aarch64_sve_fminnmv : AdvSIMD_SVE_Reduce_Intrinsic;

def int_aarch64_sve_faddqv   : AdvSIMD_SVE_V128_Reduce_Intrinsic;
def int_aarch64_sve_fmaxnmqv : AdvSIMD_SVE_V128_Reduce_Intrinsic;
def int_aarch64_sve_fminnmqv : AdvSIMD_SVE_V128_Reduce_Intrinsic;
def int_aarch64_sve_fmaxqv   : AdvSIMD_SVE_V128_Reduce_Intrinsic;
def int_aarch64_sve_fminqv   : AdvSIMD_SVE_V128_Reduce_Intrinsic;

//
// Floating-point conversions
//

def int_aarch64_sve_fcvt   : AdvSIMD_SVE_FCVT_Intrinsic;
````
- **L2177 EN**: Declares TableGen def `int_aarch64_sve_frsqrts_x`.
  **L2177 CN**: 声明 TableGen def `int_aarch64_sve_frsqrts_x`。
- **L2178 EN**: Declares TableGen def `int_aarch64_sve_fscale`.
  **L2178 CN**: 声明 TableGen def `int_aarch64_sve_fscale`。
- **L2179 EN**: Declares TableGen def `int_aarch64_sve_fsqrt`.
  **L2179 CN**: 声明 TableGen def `int_aarch64_sve_fsqrt`。
- **L2180 EN**: Declares TableGen def `int_aarch64_sve_fsub`.
  **L2180 CN**: 声明 TableGen def `int_aarch64_sve_fsub`。
- **L2181 EN**: Declares TableGen def `int_aarch64_sve_fsub_u`.
  **L2181 CN**: 声明 TableGen def `int_aarch64_sve_fsub_u`。
- **L2182 EN**: Declares TableGen def `int_aarch64_sve_fsubr`.
  **L2182 CN**: 声明 TableGen def `int_aarch64_sve_fsubr`。
- **L2183 EN**: Declares TableGen def `int_aarch64_sve_ftmad_x`.
  **L2183 CN**: 声明 TableGen def `int_aarch64_sve_ftmad_x`。
- **L2184 EN**: Declares TableGen def `int_aarch64_sve_ftsmul_x`.
  **L2184 CN**: 声明 TableGen def `int_aarch64_sve_ftsmul_x`。
- **L2185 EN**: Declares TableGen def `int_aarch64_sve_ftssel_x`.
  **L2185 CN**: 声明 TableGen def `int_aarch64_sve_ftssel_x`。
- **L2186 EN**: Blank line separating nearby declarations or logic blocks.
  **L2186 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2187 EN**: Separator comment used for visual grouping.
  **L2187 CN**: 用于视觉分组的分隔注释。
- **L2188 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point reductions`.
  **L2188 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point reductions`。
- **L2189 EN**: Separator comment used for visual grouping.
  **L2189 CN**: 用于视觉分组的分隔注释。
- **L2190 EN**: Blank line separating nearby declarations or logic blocks.
  **L2190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2191 EN**: Declares TableGen def `int_aarch64_sve_fadda`.
  **L2191 CN**: 声明 TableGen def `int_aarch64_sve_fadda`。
- **L2192 EN**: Declares TableGen def `int_aarch64_sve_faddv`.
  **L2192 CN**: 声明 TableGen def `int_aarch64_sve_faddv`。
- **L2193 EN**: Declares TableGen def `int_aarch64_sve_fmaxv`.
  **L2193 CN**: 声明 TableGen def `int_aarch64_sve_fmaxv`。
- **L2194 EN**: Declares TableGen def `int_aarch64_sve_fmaxnmv`.
  **L2194 CN**: 声明 TableGen def `int_aarch64_sve_fmaxnmv`。
- **L2195 EN**: Declares TableGen def `int_aarch64_sve_fminv`.
  **L2195 CN**: 声明 TableGen def `int_aarch64_sve_fminv`。
- **L2196 EN**: Declares TableGen def `int_aarch64_sve_fminnmv`.
  **L2196 CN**: 声明 TableGen def `int_aarch64_sve_fminnmv`。
- **L2197 EN**: Blank line separating nearby declarations or logic blocks.
  **L2197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2198 EN**: Declares TableGen def `int_aarch64_sve_faddqv`.
  **L2198 CN**: 声明 TableGen def `int_aarch64_sve_faddqv`。
- **L2199 EN**: Declares TableGen def `int_aarch64_sve_fmaxnmqv`.
  **L2199 CN**: 声明 TableGen def `int_aarch64_sve_fmaxnmqv`。
- **L2200 EN**: Declares TableGen def `int_aarch64_sve_fminnmqv`.
  **L2200 CN**: 声明 TableGen def `int_aarch64_sve_fminnmqv`。
- **L2201 EN**: Declares TableGen def `int_aarch64_sve_fmaxqv`.
  **L2201 CN**: 声明 TableGen def `int_aarch64_sve_fmaxqv`。
- **L2202 EN**: Declares TableGen def `int_aarch64_sve_fminqv`.
  **L2202 CN**: 声明 TableGen def `int_aarch64_sve_fminqv`。
- **L2203 EN**: Blank line separating nearby declarations or logic blocks.
  **L2203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2204 EN**: Separator comment used for visual grouping.
  **L2204 CN**: 用于视觉分组的分隔注释。
- **L2205 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point conversions`.
  **L2205 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point conversions`。
- **L2206 EN**: Separator comment used for visual grouping.
  **L2206 CN**: 用于视觉分组的分隔注释。
- **L2207 EN**: Blank line separating nearby declarations or logic blocks.
  **L2207 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2208 EN**: Declares TableGen def `int_aarch64_sve_fcvt`.
  **L2208 CN**: 声明 TableGen def `int_aarch64_sve_fcvt`。

### Lines 2209-2240

````tablegen
def int_aarch64_sve_fcvtzs : AdvSIMD_SVE_FCVTZS_Intrinsic;
def int_aarch64_sve_fcvtzu : AdvSIMD_SVE_FCVTZS_Intrinsic;
def int_aarch64_sve_scvtf  : AdvSIMD_SVE_SCVTF_Intrinsic;
def int_aarch64_sve_ucvtf  : AdvSIMD_SVE_SCVTF_Intrinsic;

//
// Floating-point comparisons
//

def int_aarch64_sve_facge : AdvSIMD_SVE_Compare_Intrinsic;
def int_aarch64_sve_facgt : AdvSIMD_SVE_Compare_Intrinsic;

def int_aarch64_sve_fcmpeq : AdvSIMD_SVE_Compare_Intrinsic;
def int_aarch64_sve_fcmpge : AdvSIMD_SVE_Compare_Intrinsic;
def int_aarch64_sve_fcmpgt : AdvSIMD_SVE_Compare_Intrinsic;
def int_aarch64_sve_fcmpne : AdvSIMD_SVE_Compare_Intrinsic;
def int_aarch64_sve_fcmpuo : AdvSIMD_SVE_Compare_Intrinsic;

def int_aarch64_sve_fcvtzs_i32f16   : Builtin_SVCVT<llvm_nxv4i32_ty, llvm_nxv4i1_ty, llvm_nxv8f16_ty>;
def int_aarch64_sve_fcvtzs_i32f64   : Builtin_SVCVT<llvm_nxv4i32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;
def int_aarch64_sve_fcvtzs_i64f16   : Builtin_SVCVT<llvm_nxv2i64_ty, llvm_nxv2i1_ty, llvm_nxv8f16_ty>;
def int_aarch64_sve_fcvtzs_i64f32   : Builtin_SVCVT<llvm_nxv2i64_ty, llvm_nxv2i1_ty, llvm_nxv4f32_ty>;

def int_aarch64_sve_fcvt_bf16f32_v2   : Builtin_SVCVT<llvm_nxv8bf16_ty, llvm_nxv4i1_ty, llvm_nxv4f32_ty>;
def int_aarch64_sve_fcvtnt_bf16f32_v2 : Builtin_SVCVT<llvm_nxv8bf16_ty, llvm_nxv4i1_ty, llvm_nxv4f32_ty>;
def int_aarch64_sve_fcvtnt_z_bf16f32  : Builtin_SVCVT<llvm_nxv8bf16_ty, llvm_nxv4i1_ty, llvm_nxv4f32_ty>;

def int_aarch64_sve_fcvtzu_i32f16   : Builtin_SVCVT<llvm_nxv4i32_ty, llvm_nxv4i1_ty, llvm_nxv8f16_ty>;
def int_aarch64_sve_fcvtzu_i32f64   : Builtin_SVCVT<llvm_nxv4i32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;
def int_aarch64_sve_fcvtzu_i64f16   : Builtin_SVCVT<llvm_nxv2i64_ty, llvm_nxv2i1_ty, llvm_nxv8f16_ty>;
def int_aarch64_sve_fcvtzu_i64f32   : Builtin_SVCVT<llvm_nxv2i64_ty, llvm_nxv2i1_ty, llvm_nxv4f32_ty>;

````
- **L2209 EN**: Declares TableGen def `int_aarch64_sve_fcvtzs`.
  **L2209 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzs`。
- **L2210 EN**: Declares TableGen def `int_aarch64_sve_fcvtzu`.
  **L2210 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzu`。
- **L2211 EN**: Declares TableGen def `int_aarch64_sve_scvtf`.
  **L2211 CN**: 声明 TableGen def `int_aarch64_sve_scvtf`。
- **L2212 EN**: Declares TableGen def `int_aarch64_sve_ucvtf`.
  **L2212 CN**: 声明 TableGen def `int_aarch64_sve_ucvtf`。
- **L2213 EN**: Blank line separating nearby declarations or logic blocks.
  **L2213 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2214 EN**: Separator comment used for visual grouping.
  **L2214 CN**: 用于视觉分组的分隔注释。
- **L2215 EN**: Comment explains nearby logic, invariants, or intent: `Floating-point comparisons`.
  **L2215 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-point comparisons`。
- **L2216 EN**: Separator comment used for visual grouping.
  **L2216 CN**: 用于视觉分组的分隔注释。
- **L2217 EN**: Blank line separating nearby declarations or logic blocks.
  **L2217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2218 EN**: Declares TableGen def `int_aarch64_sve_facge`.
  **L2218 CN**: 声明 TableGen def `int_aarch64_sve_facge`。
- **L2219 EN**: Declares TableGen def `int_aarch64_sve_facgt`.
  **L2219 CN**: 声明 TableGen def `int_aarch64_sve_facgt`。
- **L2220 EN**: Blank line separating nearby declarations or logic blocks.
  **L2220 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2221 EN**: Declares TableGen def `int_aarch64_sve_fcmpeq`.
  **L2221 CN**: 声明 TableGen def `int_aarch64_sve_fcmpeq`。
- **L2222 EN**: Declares TableGen def `int_aarch64_sve_fcmpge`.
  **L2222 CN**: 声明 TableGen def `int_aarch64_sve_fcmpge`。
- **L2223 EN**: Declares TableGen def `int_aarch64_sve_fcmpgt`.
  **L2223 CN**: 声明 TableGen def `int_aarch64_sve_fcmpgt`。
- **L2224 EN**: Declares TableGen def `int_aarch64_sve_fcmpne`.
  **L2224 CN**: 声明 TableGen def `int_aarch64_sve_fcmpne`。
- **L2225 EN**: Declares TableGen def `int_aarch64_sve_fcmpuo`.
  **L2225 CN**: 声明 TableGen def `int_aarch64_sve_fcmpuo`。
- **L2226 EN**: Blank line separating nearby declarations or logic blocks.
  **L2226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2227 EN**: Declares TableGen def `int_aarch64_sve_fcvtzs_i32f16`.
  **L2227 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzs_i32f16`。
- **L2228 EN**: Declares TableGen def `int_aarch64_sve_fcvtzs_i32f64`.
  **L2228 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzs_i32f64`。
- **L2229 EN**: Declares TableGen def `int_aarch64_sve_fcvtzs_i64f16`.
  **L2229 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzs_i64f16`。
- **L2230 EN**: Declares TableGen def `int_aarch64_sve_fcvtzs_i64f32`.
  **L2230 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzs_i64f32`。
- **L2231 EN**: Blank line separating nearby declarations or logic blocks.
  **L2231 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2232 EN**: Declares TableGen def `int_aarch64_sve_fcvt_bf16f32_v2`.
  **L2232 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_bf16f32_v2`。
- **L2233 EN**: Declares TableGen def `int_aarch64_sve_fcvtnt_bf16f32_v2`.
  **L2233 CN**: 声明 TableGen def `int_aarch64_sve_fcvtnt_bf16f32_v2`。
- **L2234 EN**: Declares TableGen def `int_aarch64_sve_fcvtnt_z_bf16f32`.
  **L2234 CN**: 声明 TableGen def `int_aarch64_sve_fcvtnt_z_bf16f32`。
- **L2235 EN**: Blank line separating nearby declarations or logic blocks.
  **L2235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2236 EN**: Declares TableGen def `int_aarch64_sve_fcvtzu_i32f16`.
  **L2236 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzu_i32f16`。
- **L2237 EN**: Declares TableGen def `int_aarch64_sve_fcvtzu_i32f64`.
  **L2237 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzu_i32f64`。
- **L2238 EN**: Declares TableGen def `int_aarch64_sve_fcvtzu_i64f16`.
  **L2238 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzu_i64f16`。
- **L2239 EN**: Declares TableGen def `int_aarch64_sve_fcvtzu_i64f32`.
  **L2239 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzu_i64f32`。
- **L2240 EN**: Blank line separating nearby declarations or logic blocks.
  **L2240 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2241-2272

````tablegen
def int_aarch64_sve_fcvt_f16f32     : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv4i1_ty, llvm_nxv4f32_ty>;
def int_aarch64_sve_fcvt_f16f64     : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;
def int_aarch64_sve_fcvt_f32f64     : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;

def int_aarch64_sve_fcvt_f32f16     : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv4i1_ty, llvm_nxv8f16_ty>;
def int_aarch64_sve_fcvt_f64f16     : Builtin_SVCVT<llvm_nxv2f64_ty, llvm_nxv2i1_ty, llvm_nxv8f16_ty>;
def int_aarch64_sve_fcvt_f64f32     : Builtin_SVCVT<llvm_nxv2f64_ty, llvm_nxv2i1_ty, llvm_nxv4f32_ty>;

def int_aarch64_sve_fcvtlt_f32f16   : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv4i1_ty, llvm_nxv8f16_ty>;
def int_aarch64_sve_fcvtlt_f64f32   : Builtin_SVCVT<llvm_nxv2f64_ty, llvm_nxv2i1_ty, llvm_nxv4f32_ty>;
def int_aarch64_sve_fcvtnt_f16f32   : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv4i1_ty, llvm_nxv4f32_ty>;
def int_aarch64_sve_fcvtnt_z_f16f32 : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv4i1_ty, llvm_nxv4f32_ty>;
def int_aarch64_sve_fcvtnt_f32f64   : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;
def int_aarch64_sve_fcvtnt_z_f32f64 : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;

def int_aarch64_sve_fcvtx_f32f64     : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;
def int_aarch64_sve_fcvtxnt_f32f64   : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;
def int_aarch64_sve_fcvtxnt_z_f32f64 : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2f64_ty>;

def int_aarch64_sve_scvtf_f16i32    : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv4i1_ty, llvm_nxv4i32_ty>;
def int_aarch64_sve_scvtf_f16i64    : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv2i1_ty, llvm_nxv2i64_ty>;
def int_aarch64_sve_scvtf_f32i64    : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2i64_ty>;
def int_aarch64_sve_scvtf_f64i32    : Builtin_SVCVT<llvm_nxv2f64_ty, llvm_nxv2i1_ty, llvm_nxv4i32_ty>;

def int_aarch64_sve_ucvtf_f16i32    : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv4i1_ty, llvm_nxv4i32_ty>;
def int_aarch64_sve_ucvtf_f16i64    : Builtin_SVCVT<llvm_nxv8f16_ty, llvm_nxv2i1_ty, llvm_nxv2i64_ty>;
def int_aarch64_sve_ucvtf_f32i64    : Builtin_SVCVT<llvm_nxv4f32_ty, llvm_nxv2i1_ty, llvm_nxv2i64_ty>;
def int_aarch64_sve_ucvtf_f64i32    : Builtin_SVCVT<llvm_nxv2f64_ty, llvm_nxv2i1_ty, llvm_nxv4i32_ty>;

//
// Predicate creation
//
````
- **L2241 EN**: Declares TableGen def `int_aarch64_sve_fcvt_f16f32`.
  **L2241 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_f16f32`。
- **L2242 EN**: Declares TableGen def `int_aarch64_sve_fcvt_f16f64`.
  **L2242 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_f16f64`。
- **L2243 EN**: Declares TableGen def `int_aarch64_sve_fcvt_f32f64`.
  **L2243 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_f32f64`。
- **L2244 EN**: Blank line separating nearby declarations or logic blocks.
  **L2244 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2245 EN**: Declares TableGen def `int_aarch64_sve_fcvt_f32f16`.
  **L2245 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_f32f16`。
- **L2246 EN**: Declares TableGen def `int_aarch64_sve_fcvt_f64f16`.
  **L2246 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_f64f16`。
- **L2247 EN**: Declares TableGen def `int_aarch64_sve_fcvt_f64f32`.
  **L2247 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_f64f32`。
- **L2248 EN**: Blank line separating nearby declarations or logic blocks.
  **L2248 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2249 EN**: Declares TableGen def `int_aarch64_sve_fcvtlt_f32f16`.
  **L2249 CN**: 声明 TableGen def `int_aarch64_sve_fcvtlt_f32f16`。
- **L2250 EN**: Declares TableGen def `int_aarch64_sve_fcvtlt_f64f32`.
  **L2250 CN**: 声明 TableGen def `int_aarch64_sve_fcvtlt_f64f32`。
- **L2251 EN**: Declares TableGen def `int_aarch64_sve_fcvtnt_f16f32`.
  **L2251 CN**: 声明 TableGen def `int_aarch64_sve_fcvtnt_f16f32`。
- **L2252 EN**: Declares TableGen def `int_aarch64_sve_fcvtnt_z_f16f32`.
  **L2252 CN**: 声明 TableGen def `int_aarch64_sve_fcvtnt_z_f16f32`。
- **L2253 EN**: Declares TableGen def `int_aarch64_sve_fcvtnt_f32f64`.
  **L2253 CN**: 声明 TableGen def `int_aarch64_sve_fcvtnt_f32f64`。
- **L2254 EN**: Declares TableGen def `int_aarch64_sve_fcvtnt_z_f32f64`.
  **L2254 CN**: 声明 TableGen def `int_aarch64_sve_fcvtnt_z_f32f64`。
- **L2255 EN**: Blank line separating nearby declarations or logic blocks.
  **L2255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2256 EN**: Declares TableGen def `int_aarch64_sve_fcvtx_f32f64`.
  **L2256 CN**: 声明 TableGen def `int_aarch64_sve_fcvtx_f32f64`。
- **L2257 EN**: Declares TableGen def `int_aarch64_sve_fcvtxnt_f32f64`.
  **L2257 CN**: 声明 TableGen def `int_aarch64_sve_fcvtxnt_f32f64`。
- **L2258 EN**: Declares TableGen def `int_aarch64_sve_fcvtxnt_z_f32f64`.
  **L2258 CN**: 声明 TableGen def `int_aarch64_sve_fcvtxnt_z_f32f64`。
- **L2259 EN**: Blank line separating nearby declarations or logic blocks.
  **L2259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2260 EN**: Declares TableGen def `int_aarch64_sve_scvtf_f16i32`.
  **L2260 CN**: 声明 TableGen def `int_aarch64_sve_scvtf_f16i32`。
- **L2261 EN**: Declares TableGen def `int_aarch64_sve_scvtf_f16i64`.
  **L2261 CN**: 声明 TableGen def `int_aarch64_sve_scvtf_f16i64`。
- **L2262 EN**: Declares TableGen def `int_aarch64_sve_scvtf_f32i64`.
  **L2262 CN**: 声明 TableGen def `int_aarch64_sve_scvtf_f32i64`。
- **L2263 EN**: Declares TableGen def `int_aarch64_sve_scvtf_f64i32`.
  **L2263 CN**: 声明 TableGen def `int_aarch64_sve_scvtf_f64i32`。
- **L2264 EN**: Blank line separating nearby declarations or logic blocks.
  **L2264 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2265 EN**: Declares TableGen def `int_aarch64_sve_ucvtf_f16i32`.
  **L2265 CN**: 声明 TableGen def `int_aarch64_sve_ucvtf_f16i32`。
- **L2266 EN**: Declares TableGen def `int_aarch64_sve_ucvtf_f16i64`.
  **L2266 CN**: 声明 TableGen def `int_aarch64_sve_ucvtf_f16i64`。
- **L2267 EN**: Declares TableGen def `int_aarch64_sve_ucvtf_f32i64`.
  **L2267 CN**: 声明 TableGen def `int_aarch64_sve_ucvtf_f32i64`。
- **L2268 EN**: Declares TableGen def `int_aarch64_sve_ucvtf_f64i32`.
  **L2268 CN**: 声明 TableGen def `int_aarch64_sve_ucvtf_f64i32`。
- **L2269 EN**: Blank line separating nearby declarations or logic blocks.
  **L2269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2270 EN**: Separator comment used for visual grouping.
  **L2270 CN**: 用于视觉分组的分隔注释。
- **L2271 EN**: Comment explains nearby logic, invariants, or intent: `Predicate creation`.
  **L2271 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Predicate creation`。
- **L2272 EN**: Separator comment used for visual grouping.
  **L2272 CN**: 用于视觉分组的分隔注释。

### Lines 2273-2304

````tablegen

def int_aarch64_sve_ptrue : AdvSIMD_SVE_PTRUE_Intrinsic;

//
// Predicate operations
//

def int_aarch64_sve_and_z   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bic_z   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_brka    : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_brka_z  : AdvSIMD_Pred1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_brkb    : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_brkb_z  : AdvSIMD_Pred1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_brkn_z  : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_brkpa_z : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_brkpb_z : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_eor_z   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_nand_z  : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_nor_z   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_orn_z   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_orr_z   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_pfirst  : AdvSIMD_Pred1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_pnext   : AdvSIMD_Pred1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_punpkhi : AdvSIMD_SVE_PUNPKHI_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_punpklo : AdvSIMD_SVE_PUNPKHI_Intrinsic<[IntrSpeculatable]>;

//
// Testing predicates
//

def int_aarch64_sve_ptest_any   : AdvSIMD_SVE_PTEST_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ptest_first : AdvSIMD_SVE_PTEST_Intrinsic<[IntrSpeculatable]>;
````
- **L2273 EN**: Blank line separating nearby declarations or logic blocks.
  **L2273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2274 EN**: Declares TableGen def `int_aarch64_sve_ptrue`.
  **L2274 CN**: 声明 TableGen def `int_aarch64_sve_ptrue`。
- **L2275 EN**: Blank line separating nearby declarations or logic blocks.
  **L2275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2276 EN**: Separator comment used for visual grouping.
  **L2276 CN**: 用于视觉分组的分隔注释。
- **L2277 EN**: Comment explains nearby logic, invariants, or intent: `Predicate operations`.
  **L2277 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Predicate operations`。
- **L2278 EN**: Separator comment used for visual grouping.
  **L2278 CN**: 用于视觉分组的分隔注释。
- **L2279 EN**: Blank line separating nearby declarations or logic blocks.
  **L2279 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2280 EN**: Declares TableGen def `int_aarch64_sve_and_z`.
  **L2280 CN**: 声明 TableGen def `int_aarch64_sve_and_z`。
- **L2281 EN**: Declares TableGen def `int_aarch64_sve_bic_z`.
  **L2281 CN**: 声明 TableGen def `int_aarch64_sve_bic_z`。
- **L2282 EN**: Declares TableGen def `int_aarch64_sve_brka`.
  **L2282 CN**: 声明 TableGen def `int_aarch64_sve_brka`。
- **L2283 EN**: Declares TableGen def `int_aarch64_sve_brka_z`.
  **L2283 CN**: 声明 TableGen def `int_aarch64_sve_brka_z`。
- **L2284 EN**: Declares TableGen def `int_aarch64_sve_brkb`.
  **L2284 CN**: 声明 TableGen def `int_aarch64_sve_brkb`。
- **L2285 EN**: Declares TableGen def `int_aarch64_sve_brkb_z`.
  **L2285 CN**: 声明 TableGen def `int_aarch64_sve_brkb_z`。
- **L2286 EN**: Declares TableGen def `int_aarch64_sve_brkn_z`.
  **L2286 CN**: 声明 TableGen def `int_aarch64_sve_brkn_z`。
- **L2287 EN**: Declares TableGen def `int_aarch64_sve_brkpa_z`.
  **L2287 CN**: 声明 TableGen def `int_aarch64_sve_brkpa_z`。
- **L2288 EN**: Declares TableGen def `int_aarch64_sve_brkpb_z`.
  **L2288 CN**: 声明 TableGen def `int_aarch64_sve_brkpb_z`。
- **L2289 EN**: Declares TableGen def `int_aarch64_sve_eor_z`.
  **L2289 CN**: 声明 TableGen def `int_aarch64_sve_eor_z`。
- **L2290 EN**: Declares TableGen def `int_aarch64_sve_nand_z`.
  **L2290 CN**: 声明 TableGen def `int_aarch64_sve_nand_z`。
- **L2291 EN**: Declares TableGen def `int_aarch64_sve_nor_z`.
  **L2291 CN**: 声明 TableGen def `int_aarch64_sve_nor_z`。
- **L2292 EN**: Declares TableGen def `int_aarch64_sve_orn_z`.
  **L2292 CN**: 声明 TableGen def `int_aarch64_sve_orn_z`。
- **L2293 EN**: Declares TableGen def `int_aarch64_sve_orr_z`.
  **L2293 CN**: 声明 TableGen def `int_aarch64_sve_orr_z`。
- **L2294 EN**: Declares TableGen def `int_aarch64_sve_pfirst`.
  **L2294 CN**: 声明 TableGen def `int_aarch64_sve_pfirst`。
- **L2295 EN**: Declares TableGen def `int_aarch64_sve_pnext`.
  **L2295 CN**: 声明 TableGen def `int_aarch64_sve_pnext`。
- **L2296 EN**: Declares TableGen def `int_aarch64_sve_punpkhi`.
  **L2296 CN**: 声明 TableGen def `int_aarch64_sve_punpkhi`。
- **L2297 EN**: Declares TableGen def `int_aarch64_sve_punpklo`.
  **L2297 CN**: 声明 TableGen def `int_aarch64_sve_punpklo`。
- **L2298 EN**: Blank line separating nearby declarations or logic blocks.
  **L2298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2299 EN**: Separator comment used for visual grouping.
  **L2299 CN**: 用于视觉分组的分隔注释。
- **L2300 EN**: Comment explains nearby logic, invariants, or intent: `Testing predicates`.
  **L2300 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Testing predicates`。
- **L2301 EN**: Separator comment used for visual grouping.
  **L2301 CN**: 用于视觉分组的分隔注释。
- **L2302 EN**: Blank line separating nearby declarations or logic blocks.
  **L2302 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2303 EN**: Declares TableGen def `int_aarch64_sve_ptest_any`.
  **L2303 CN**: 声明 TableGen def `int_aarch64_sve_ptest_any`。
- **L2304 EN**: Declares TableGen def `int_aarch64_sve_ptest_first`.
  **L2304 CN**: 声明 TableGen def `int_aarch64_sve_ptest_first`。

### Lines 2305-2336

````tablegen
def int_aarch64_sve_ptest_last  : AdvSIMD_SVE_PTEST_Intrinsic<[IntrSpeculatable]>;

//
// Reinterpreting data
//

def int_aarch64_sve_convert_from_svbool : DefaultAttrsIntrinsic<[llvm_any_ty],
                                                    [llvm_nxv16i1_ty],
                                                    [IntrNoMem, IntrSpeculatable]>;

def int_aarch64_sve_convert_to_svbool : DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],
                                                  [llvm_any_ty],
                                                  [IntrNoMem, IntrSpeculatable]>;

//
// Gather loads: scalar base + vector offsets
//

// 64 bit unscaled offsets
def int_aarch64_sve_ld1_gather : AdvSIMD_GatherLoad_SV_64b_Offsets_Intrinsic;

// 64 bit scaled offsets
def int_aarch64_sve_ld1_gather_index : AdvSIMD_GatherLoad_SV_64b_Offsets_Intrinsic;

// 32 bit unscaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits
def int_aarch64_sve_ld1_gather_sxtw : AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic;
def int_aarch64_sve_ld1_gather_uxtw : AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic;

// 32 bit scaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits
def int_aarch64_sve_ld1_gather_sxtw_index : AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic;
def int_aarch64_sve_ld1_gather_uxtw_index : AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic;

````
- **L2305 EN**: Declares TableGen def `int_aarch64_sve_ptest_last`.
  **L2305 CN**: 声明 TableGen def `int_aarch64_sve_ptest_last`。
- **L2306 EN**: Blank line separating nearby declarations or logic blocks.
  **L2306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2307 EN**: Separator comment used for visual grouping.
  **L2307 CN**: 用于视觉分组的分隔注释。
- **L2308 EN**: Comment explains nearby logic, invariants, or intent: `Reinterpreting data`.
  **L2308 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reinterpreting data`。
- **L2309 EN**: Separator comment used for visual grouping.
  **L2309 CN**: 用于视觉分组的分隔注释。
- **L2310 EN**: Blank line separating nearby declarations or logic blocks.
  **L2310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2311 EN**: Declares TableGen def `int_aarch64_sve_convert_from_svbool`.
  **L2311 CN**: 声明 TableGen def `int_aarch64_sve_convert_from_svbool`。
- **L2312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i1_ty],`.
  **L2312 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i1_ty],`。
- **L2313 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2313 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2314 EN**: Blank line separating nearby declarations or logic blocks.
  **L2314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2315 EN**: Declares TableGen def `int_aarch64_sve_convert_to_svbool`.
  **L2315 CN**: 声明 TableGen def `int_aarch64_sve_convert_to_svbool`。
- **L2316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_any_ty],`.
  **L2316 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_any_ty],`。
- **L2317 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2317 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2318 EN**: Blank line separating nearby declarations or logic blocks.
  **L2318 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2319 EN**: Separator comment used for visual grouping.
  **L2319 CN**: 用于视觉分组的分隔注释。
- **L2320 EN**: Comment explains nearby logic, invariants, or intent: `Gather loads: scalar base + vector offsets`.
  **L2320 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Gather loads: scalar base + vector offsets`。
- **L2321 EN**: Separator comment used for visual grouping.
  **L2321 CN**: 用于视觉分组的分隔注释。
- **L2322 EN**: Blank line separating nearby declarations or logic blocks.
  **L2322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2323 EN**: Comment explains nearby logic, invariants, or intent: `64 bit unscaled offsets`.
  **L2323 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit unscaled offsets`。
- **L2324 EN**: Declares TableGen def `int_aarch64_sve_ld1_gather`.
  **L2324 CN**: 声明 TableGen def `int_aarch64_sve_ld1_gather`。
- **L2325 EN**: Blank line separating nearby declarations or logic blocks.
  **L2325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2326 EN**: Comment explains nearby logic, invariants, or intent: `64 bit scaled offsets`.
  **L2326 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit scaled offsets`。
- **L2327 EN**: Declares TableGen def `int_aarch64_sve_ld1_gather_index`.
  **L2327 CN**: 声明 TableGen def `int_aarch64_sve_ld1_gather_index`。
- **L2328 EN**: Blank line separating nearby declarations or logic blocks.
  **L2328 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2329 EN**: Comment explains nearby logic, invariants, or intent: `32 bit unscaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`.
  **L2329 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit unscaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`。
- **L2330 EN**: Declares TableGen def `int_aarch64_sve_ld1_gather_sxtw`.
  **L2330 CN**: 声明 TableGen def `int_aarch64_sve_ld1_gather_sxtw`。
- **L2331 EN**: Declares TableGen def `int_aarch64_sve_ld1_gather_uxtw`.
  **L2331 CN**: 声明 TableGen def `int_aarch64_sve_ld1_gather_uxtw`。
- **L2332 EN**: Blank line separating nearby declarations or logic blocks.
  **L2332 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2333 EN**: Comment explains nearby logic, invariants, or intent: `32 bit scaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`.
  **L2333 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit scaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`。
- **L2334 EN**: Declares TableGen def `int_aarch64_sve_ld1_gather_sxtw_index`.
  **L2334 CN**: 声明 TableGen def `int_aarch64_sve_ld1_gather_sxtw_index`。
- **L2335 EN**: Declares TableGen def `int_aarch64_sve_ld1_gather_uxtw_index`.
  **L2335 CN**: 声明 TableGen def `int_aarch64_sve_ld1_gather_uxtw_index`。
- **L2336 EN**: Blank line separating nearby declarations or logic blocks.
  **L2336 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2337-2368

````tablegen
// 128-bit loads, scaled offsets (indices)
def int_aarch64_sve_ld1q_gather_index : AdvSIMD_GatherLoadQ_SV_Intrinsic;

// 128-bit loads, unscaled offsets
def int_aarch64_sve_ld1q_gather_vector_offset : AdvSIMD_GatherLoadQ_SV_Intrinsic;

//
// Gather loads: vector base + scalar offset
//

def int_aarch64_sve_ld1_gather_scalar_offset : AdvSIMD_GatherLoad_VS_Intrinsic;

// 128-bit loads, unscaled offsets
def int_aarch64_sve_ld1q_gather_scalar_offset : AdvSIMD_GatherLoadQ_VS_Intrinsic;

//
// First-faulting gather loads: scalar base + vector offsets
//

// 64 bit unscaled offsets
def int_aarch64_sve_ldff1_gather : AdvSIMD_GatherLoad_SV_64b_Offsets_WriteFFR_Intrinsic;

// 64 bit scaled offsets
def int_aarch64_sve_ldff1_gather_index : AdvSIMD_GatherLoad_SV_64b_Offsets_WriteFFR_Intrinsic;

// 32 bit unscaled offsets, sign (sxtw) or zero (uxtw) extended to 64 bits
def int_aarch64_sve_ldff1_gather_sxtw : AdvSIMD_GatherLoad_SV_32b_Offsets_WriteFFR_Intrinsic;
def int_aarch64_sve_ldff1_gather_uxtw : AdvSIMD_GatherLoad_SV_32b_Offsets_WriteFFR_Intrinsic;

// 32 bit scaled offsets, sign (sxtw) or zero (uxtw) extended to 64 bits
def int_aarch64_sve_ldff1_gather_sxtw_index : AdvSIMD_GatherLoad_SV_32b_Offsets_WriteFFR_Intrinsic;
def int_aarch64_sve_ldff1_gather_uxtw_index : AdvSIMD_GatherLoad_SV_32b_Offsets_WriteFFR_Intrinsic;
````
- **L2337 EN**: Comment explains nearby logic, invariants, or intent: `128-bit loads, scaled offsets (indices)`.
  **L2337 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`128-bit loads, scaled offsets (indices)`。
- **L2338 EN**: Declares TableGen def `int_aarch64_sve_ld1q_gather_index`.
  **L2338 CN**: 声明 TableGen def `int_aarch64_sve_ld1q_gather_index`。
- **L2339 EN**: Blank line separating nearby declarations or logic blocks.
  **L2339 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2340 EN**: Comment explains nearby logic, invariants, or intent: `128-bit loads, unscaled offsets`.
  **L2340 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`128-bit loads, unscaled offsets`。
- **L2341 EN**: Declares TableGen def `int_aarch64_sve_ld1q_gather_vector_offset`.
  **L2341 CN**: 声明 TableGen def `int_aarch64_sve_ld1q_gather_vector_offset`。
- **L2342 EN**: Blank line separating nearby declarations or logic blocks.
  **L2342 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2343 EN**: Separator comment used for visual grouping.
  **L2343 CN**: 用于视觉分组的分隔注释。
- **L2344 EN**: Comment explains nearby logic, invariants, or intent: `Gather loads: vector base + scalar offset`.
  **L2344 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Gather loads: vector base + scalar offset`。
- **L2345 EN**: Separator comment used for visual grouping.
  **L2345 CN**: 用于视觉分组的分隔注释。
- **L2346 EN**: Blank line separating nearby declarations or logic blocks.
  **L2346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2347 EN**: Declares TableGen def `int_aarch64_sve_ld1_gather_scalar_offset`.
  **L2347 CN**: 声明 TableGen def `int_aarch64_sve_ld1_gather_scalar_offset`。
- **L2348 EN**: Blank line separating nearby declarations or logic blocks.
  **L2348 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2349 EN**: Comment explains nearby logic, invariants, or intent: `128-bit loads, unscaled offsets`.
  **L2349 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`128-bit loads, unscaled offsets`。
- **L2350 EN**: Declares TableGen def `int_aarch64_sve_ld1q_gather_scalar_offset`.
  **L2350 CN**: 声明 TableGen def `int_aarch64_sve_ld1q_gather_scalar_offset`。
- **L2351 EN**: Blank line separating nearby declarations or logic blocks.
  **L2351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2352 EN**: Separator comment used for visual grouping.
  **L2352 CN**: 用于视觉分组的分隔注释。
- **L2353 EN**: Comment explains nearby logic, invariants, or intent: `First-faulting gather loads: scalar base + vector offsets`.
  **L2353 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`First-faulting gather loads: scalar base + vector offsets`。
- **L2354 EN**: Separator comment used for visual grouping.
  **L2354 CN**: 用于视觉分组的分隔注释。
- **L2355 EN**: Blank line separating nearby declarations or logic blocks.
  **L2355 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2356 EN**: Comment explains nearby logic, invariants, or intent: `64 bit unscaled offsets`.
  **L2356 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit unscaled offsets`。
- **L2357 EN**: Declares TableGen def `int_aarch64_sve_ldff1_gather`.
  **L2357 CN**: 声明 TableGen def `int_aarch64_sve_ldff1_gather`。
- **L2358 EN**: Blank line separating nearby declarations or logic blocks.
  **L2358 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2359 EN**: Comment explains nearby logic, invariants, or intent: `64 bit scaled offsets`.
  **L2359 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit scaled offsets`。
- **L2360 EN**: Declares TableGen def `int_aarch64_sve_ldff1_gather_index`.
  **L2360 CN**: 声明 TableGen def `int_aarch64_sve_ldff1_gather_index`。
- **L2361 EN**: Blank line separating nearby declarations or logic blocks.
  **L2361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2362 EN**: Comment explains nearby logic, invariants, or intent: `32 bit unscaled offsets, sign (sxtw) or zero (uxtw) extended to 64 bits`.
  **L2362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit unscaled offsets, sign (sxtw) or zero (uxtw) extended to 64 bits`。
- **L2363 EN**: Declares TableGen def `int_aarch64_sve_ldff1_gather_sxtw`.
  **L2363 CN**: 声明 TableGen def `int_aarch64_sve_ldff1_gather_sxtw`。
- **L2364 EN**: Declares TableGen def `int_aarch64_sve_ldff1_gather_uxtw`.
  **L2364 CN**: 声明 TableGen def `int_aarch64_sve_ldff1_gather_uxtw`。
- **L2365 EN**: Blank line separating nearby declarations or logic blocks.
  **L2365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2366 EN**: Comment explains nearby logic, invariants, or intent: `32 bit scaled offsets, sign (sxtw) or zero (uxtw) extended to 64 bits`.
  **L2366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit scaled offsets, sign (sxtw) or zero (uxtw) extended to 64 bits`。
- **L2367 EN**: Declares TableGen def `int_aarch64_sve_ldff1_gather_sxtw_index`.
  **L2367 CN**: 声明 TableGen def `int_aarch64_sve_ldff1_gather_sxtw_index`。
- **L2368 EN**: Declares TableGen def `int_aarch64_sve_ldff1_gather_uxtw_index`.
  **L2368 CN**: 声明 TableGen def `int_aarch64_sve_ldff1_gather_uxtw_index`。

### Lines 2369-2400

````tablegen

//
// First-faulting gather loads: vector base + scalar offset
//

def int_aarch64_sve_ldff1_gather_scalar_offset : AdvSIMD_GatherLoad_VS_WriteFFR_Intrinsic;


//
// Non-temporal gather loads: scalar base + vector offsets
//

// 64 bit unscaled offsets
def int_aarch64_sve_ldnt1_gather : AdvSIMD_GatherLoad_SV_64b_Offsets_Intrinsic;

// 64 bit indices
def int_aarch64_sve_ldnt1_gather_index : AdvSIMD_GatherLoad_SV_64b_Offsets_Intrinsic;

// 32 bit unscaled offsets, zero (zxtw) extended to 64 bits
def int_aarch64_sve_ldnt1_gather_uxtw : AdvSIMD_GatherLoad_SV_32b_Offsets_Intrinsic;

//
// Non-temporal gather loads: vector base + scalar offset
//

def int_aarch64_sve_ldnt1_gather_scalar_offset  : AdvSIMD_GatherLoad_VS_Intrinsic;

//
// Scatter stores: scalar base + vector offsets
//

// 64 bit unscaled offsets
````
- **L2369 EN**: Blank line separating nearby declarations or logic blocks.
  **L2369 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2370 EN**: Separator comment used for visual grouping.
  **L2370 CN**: 用于视觉分组的分隔注释。
- **L2371 EN**: Comment explains nearby logic, invariants, or intent: `First-faulting gather loads: vector base + scalar offset`.
  **L2371 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`First-faulting gather loads: vector base + scalar offset`。
- **L2372 EN**: Separator comment used for visual grouping.
  **L2372 CN**: 用于视觉分组的分隔注释。
- **L2373 EN**: Blank line separating nearby declarations or logic blocks.
  **L2373 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2374 EN**: Declares TableGen def `int_aarch64_sve_ldff1_gather_scalar_offset`.
  **L2374 CN**: 声明 TableGen def `int_aarch64_sve_ldff1_gather_scalar_offset`。
- **L2375 EN**: Blank line separating nearby declarations or logic blocks.
  **L2375 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2376 EN**: Blank line separating nearby declarations or logic blocks.
  **L2376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2377 EN**: Separator comment used for visual grouping.
  **L2377 CN**: 用于视觉分组的分隔注释。
- **L2378 EN**: Comment explains nearby logic, invariants, or intent: `Non-temporal gather loads: scalar base + vector offsets`.
  **L2378 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-temporal gather loads: scalar base + vector offsets`。
- **L2379 EN**: Separator comment used for visual grouping.
  **L2379 CN**: 用于视觉分组的分隔注释。
- **L2380 EN**: Blank line separating nearby declarations or logic blocks.
  **L2380 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2381 EN**: Comment explains nearby logic, invariants, or intent: `64 bit unscaled offsets`.
  **L2381 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit unscaled offsets`。
- **L2382 EN**: Declares TableGen def `int_aarch64_sve_ldnt1_gather`.
  **L2382 CN**: 声明 TableGen def `int_aarch64_sve_ldnt1_gather`。
- **L2383 EN**: Blank line separating nearby declarations or logic blocks.
  **L2383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2384 EN**: Comment explains nearby logic, invariants, or intent: `64 bit indices`.
  **L2384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit indices`。
- **L2385 EN**: Declares TableGen def `int_aarch64_sve_ldnt1_gather_index`.
  **L2385 CN**: 声明 TableGen def `int_aarch64_sve_ldnt1_gather_index`。
- **L2386 EN**: Blank line separating nearby declarations or logic blocks.
  **L2386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2387 EN**: Comment explains nearby logic, invariants, or intent: `32 bit unscaled offsets, zero (zxtw) extended to 64 bits`.
  **L2387 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit unscaled offsets, zero (zxtw) extended to 64 bits`。
- **L2388 EN**: Declares TableGen def `int_aarch64_sve_ldnt1_gather_uxtw`.
  **L2388 CN**: 声明 TableGen def `int_aarch64_sve_ldnt1_gather_uxtw`。
- **L2389 EN**: Blank line separating nearby declarations or logic blocks.
  **L2389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2390 EN**: Separator comment used for visual grouping.
  **L2390 CN**: 用于视觉分组的分隔注释。
- **L2391 EN**: Comment explains nearby logic, invariants, or intent: `Non-temporal gather loads: vector base + scalar offset`.
  **L2391 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-temporal gather loads: vector base + scalar offset`。
- **L2392 EN**: Separator comment used for visual grouping.
  **L2392 CN**: 用于视觉分组的分隔注释。
- **L2393 EN**: Blank line separating nearby declarations or logic blocks.
  **L2393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2394 EN**: Declares TableGen def `int_aarch64_sve_ldnt1_gather_scalar_offset`.
  **L2394 CN**: 声明 TableGen def `int_aarch64_sve_ldnt1_gather_scalar_offset`。
- **L2395 EN**: Blank line separating nearby declarations or logic blocks.
  **L2395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2396 EN**: Separator comment used for visual grouping.
  **L2396 CN**: 用于视觉分组的分隔注释。
- **L2397 EN**: Comment explains nearby logic, invariants, or intent: `Scatter stores: scalar base + vector offsets`.
  **L2397 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scatter stores: scalar base + vector offsets`。
- **L2398 EN**: Separator comment used for visual grouping.
  **L2398 CN**: 用于视觉分组的分隔注释。
- **L2399 EN**: Blank line separating nearby declarations or logic blocks.
  **L2399 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2400 EN**: Comment explains nearby logic, invariants, or intent: `64 bit unscaled offsets`.
  **L2400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit unscaled offsets`。

### Lines 2401-2432

````tablegen
def int_aarch64_sve_st1_scatter : AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;

// 64 bit scaled offsets
def int_aarch64_sve_st1_scatter_index
    : AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;

// 32 bit unscaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits
def int_aarch64_sve_st1_scatter_sxtw
    : AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;

def int_aarch64_sve_st1_scatter_uxtw
    : AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;

// 32 bit scaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits
def int_aarch64_sve_st1_scatter_sxtw_index
    : AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;

def int_aarch64_sve_st1_scatter_uxtw_index
    : AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;

// 128-bit stores, scaled offsets (indices)
def int_aarch64_sve_st1q_scatter_index : AdvSIMD_ScatterStoreQ_SV_Intrinsic;

// 128-bit stores, unscaled offsets
def int_aarch64_sve_st1q_scatter_vector_offset : AdvSIMD_ScatterStoreQ_SV_Intrinsic;

//
// Scatter stores: vector base + scalar offset
//

def int_aarch64_sve_st1_scatter_scalar_offset : AdvSIMD_ScatterStore_VS_Intrinsic;

````
- **L2401 EN**: Declares TableGen def `int_aarch64_sve_st1_scatter`.
  **L2401 CN**: 声明 TableGen def `int_aarch64_sve_st1_scatter`。
- **L2402 EN**: Blank line separating nearby declarations or logic blocks.
  **L2402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2403 EN**: Comment explains nearby logic, invariants, or intent: `64 bit scaled offsets`.
  **L2403 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit scaled offsets`。
- **L2404 EN**: Declares TableGen def `int_aarch64_sve_st1_scatter_index`.
  **L2404 CN**: 声明 TableGen def `int_aarch64_sve_st1_scatter_index`。
- **L2405 EN**: Executes a standalone statement or declaration: `: AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;`.
  **L2405 CN**: 执行一条独立语句或声明：`: AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;`。
- **L2406 EN**: Blank line separating nearby declarations or logic blocks.
  **L2406 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2407 EN**: Comment explains nearby logic, invariants, or intent: `32 bit unscaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`.
  **L2407 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit unscaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`。
- **L2408 EN**: Declares TableGen def `int_aarch64_sve_st1_scatter_sxtw`.
  **L2408 CN**: 声明 TableGen def `int_aarch64_sve_st1_scatter_sxtw`。
- **L2409 EN**: Executes a standalone statement or declaration: `: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`.
  **L2409 CN**: 执行一条独立语句或声明：`: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`。
- **L2410 EN**: Blank line separating nearby declarations or logic blocks.
  **L2410 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2411 EN**: Declares TableGen def `int_aarch64_sve_st1_scatter_uxtw`.
  **L2411 CN**: 声明 TableGen def `int_aarch64_sve_st1_scatter_uxtw`。
- **L2412 EN**: Executes a standalone statement or declaration: `: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`.
  **L2412 CN**: 执行一条独立语句或声明：`: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`。
- **L2413 EN**: Blank line separating nearby declarations or logic blocks.
  **L2413 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2414 EN**: Comment explains nearby logic, invariants, or intent: `32 bit scaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`.
  **L2414 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit scaled offsets, sign (sxtw) or zero (zxtw) extended to 64 bits`。
- **L2415 EN**: Declares TableGen def `int_aarch64_sve_st1_scatter_sxtw_index`.
  **L2415 CN**: 声明 TableGen def `int_aarch64_sve_st1_scatter_sxtw_index`。
- **L2416 EN**: Executes a standalone statement or declaration: `: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`.
  **L2416 CN**: 执行一条独立语句或声明：`: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`。
- **L2417 EN**: Blank line separating nearby declarations or logic blocks.
  **L2417 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2418 EN**: Declares TableGen def `int_aarch64_sve_st1_scatter_uxtw_index`.
  **L2418 CN**: 声明 TableGen def `int_aarch64_sve_st1_scatter_uxtw_index`。
- **L2419 EN**: Executes a standalone statement or declaration: `: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`.
  **L2419 CN**: 执行一条独立语句或声明：`: AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;`。
- **L2420 EN**: Blank line separating nearby declarations or logic blocks.
  **L2420 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2421 EN**: Comment explains nearby logic, invariants, or intent: `128-bit stores, scaled offsets (indices)`.
  **L2421 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`128-bit stores, scaled offsets (indices)`。
- **L2422 EN**: Declares TableGen def `int_aarch64_sve_st1q_scatter_index`.
  **L2422 CN**: 声明 TableGen def `int_aarch64_sve_st1q_scatter_index`。
- **L2423 EN**: Blank line separating nearby declarations or logic blocks.
  **L2423 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2424 EN**: Comment explains nearby logic, invariants, or intent: `128-bit stores, unscaled offsets`.
  **L2424 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`128-bit stores, unscaled offsets`。
- **L2425 EN**: Declares TableGen def `int_aarch64_sve_st1q_scatter_vector_offset`.
  **L2425 CN**: 声明 TableGen def `int_aarch64_sve_st1q_scatter_vector_offset`。
- **L2426 EN**: Blank line separating nearby declarations or logic blocks.
  **L2426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2427 EN**: Separator comment used for visual grouping.
  **L2427 CN**: 用于视觉分组的分隔注释。
- **L2428 EN**: Comment explains nearby logic, invariants, or intent: `Scatter stores: vector base + scalar offset`.
  **L2428 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scatter stores: vector base + scalar offset`。
- **L2429 EN**: Separator comment used for visual grouping.
  **L2429 CN**: 用于视觉分组的分隔注释。
- **L2430 EN**: Blank line separating nearby declarations or logic blocks.
  **L2430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2431 EN**: Declares TableGen def `int_aarch64_sve_st1_scatter_scalar_offset`.
  **L2431 CN**: 声明 TableGen def `int_aarch64_sve_st1_scatter_scalar_offset`。
- **L2432 EN**: Blank line separating nearby declarations or logic blocks.
  **L2432 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2433-2464

````tablegen
// 128-bit stores, unscaled offsets
def int_aarch64_sve_st1q_scatter_scalar_offset : AdvSIMD_ScatterStoreQ_VS_Intrinsic;

//
// Non-temporal scatter stores: scalar base + vector offsets
//

// 64 bit unscaled offsets
def int_aarch64_sve_stnt1_scatter : AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;

// 64 bit indices
def int_aarch64_sve_stnt1_scatter_index
    : AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;

// 32 bit unscaled offsets, zero (zxtw) extended to 64 bits
def int_aarch64_sve_stnt1_scatter_uxtw : AdvSIMD_ScatterStore_SV_32b_Offsets_Intrinsic;

//
// Non-temporal scatter stores: vector base + scalar offset
//

def int_aarch64_sve_stnt1_scatter_scalar_offset  : AdvSIMD_ScatterStore_VS_Intrinsic;

//
// SVE2 - Uniform DSP operations
//

def int_aarch64_sve_saba          : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sabal         : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uabal         : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_shadd         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_shsub         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L2433 EN**: Comment explains nearby logic, invariants, or intent: `128-bit stores, unscaled offsets`.
  **L2433 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`128-bit stores, unscaled offsets`。
- **L2434 EN**: Declares TableGen def `int_aarch64_sve_st1q_scatter_scalar_offset`.
  **L2434 CN**: 声明 TableGen def `int_aarch64_sve_st1q_scatter_scalar_offset`。
- **L2435 EN**: Blank line separating nearby declarations or logic blocks.
  **L2435 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2436 EN**: Separator comment used for visual grouping.
  **L2436 CN**: 用于视觉分组的分隔注释。
- **L2437 EN**: Comment explains nearby logic, invariants, or intent: `Non-temporal scatter stores: scalar base + vector offsets`.
  **L2437 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-temporal scatter stores: scalar base + vector offsets`。
- **L2438 EN**: Separator comment used for visual grouping.
  **L2438 CN**: 用于视觉分组的分隔注释。
- **L2439 EN**: Blank line separating nearby declarations or logic blocks.
  **L2439 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2440 EN**: Comment explains nearby logic, invariants, or intent: `64 bit unscaled offsets`.
  **L2440 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit unscaled offsets`。
- **L2441 EN**: Declares TableGen def `int_aarch64_sve_stnt1_scatter`.
  **L2441 CN**: 声明 TableGen def `int_aarch64_sve_stnt1_scatter`。
- **L2442 EN**: Blank line separating nearby declarations or logic blocks.
  **L2442 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2443 EN**: Comment explains nearby logic, invariants, or intent: `64 bit indices`.
  **L2443 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`64 bit indices`。
- **L2444 EN**: Declares TableGen def `int_aarch64_sve_stnt1_scatter_index`.
  **L2444 CN**: 声明 TableGen def `int_aarch64_sve_stnt1_scatter_index`。
- **L2445 EN**: Executes a standalone statement or declaration: `: AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;`.
  **L2445 CN**: 执行一条独立语句或声明：`: AdvSIMD_ScatterStore_SV_64b_Offsets_Intrinsic;`。
- **L2446 EN**: Blank line separating nearby declarations or logic blocks.
  **L2446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2447 EN**: Comment explains nearby logic, invariants, or intent: `32 bit unscaled offsets, zero (zxtw) extended to 64 bits`.
  **L2447 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`32 bit unscaled offsets, zero (zxtw) extended to 64 bits`。
- **L2448 EN**: Declares TableGen def `int_aarch64_sve_stnt1_scatter_uxtw`.
  **L2448 CN**: 声明 TableGen def `int_aarch64_sve_stnt1_scatter_uxtw`。
- **L2449 EN**: Blank line separating nearby declarations or logic blocks.
  **L2449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2450 EN**: Separator comment used for visual grouping.
  **L2450 CN**: 用于视觉分组的分隔注释。
- **L2451 EN**: Comment explains nearby logic, invariants, or intent: `Non-temporal scatter stores: vector base + scalar offset`.
  **L2451 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-temporal scatter stores: vector base + scalar offset`。
- **L2452 EN**: Separator comment used for visual grouping.
  **L2452 CN**: 用于视觉分组的分隔注释。
- **L2453 EN**: Blank line separating nearby declarations or logic blocks.
  **L2453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2454 EN**: Declares TableGen def `int_aarch64_sve_stnt1_scatter_scalar_offset`.
  **L2454 CN**: 声明 TableGen def `int_aarch64_sve_stnt1_scatter_scalar_offset`。
- **L2455 EN**: Blank line separating nearby declarations or logic blocks.
  **L2455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2456 EN**: Separator comment used for visual grouping.
  **L2456 CN**: 用于视觉分组的分隔注释。
- **L2457 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Uniform DSP operations`.
  **L2457 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Uniform DSP operations`。
- **L2458 EN**: Separator comment used for visual grouping.
  **L2458 CN**: 用于视觉分组的分隔注释。
- **L2459 EN**: Blank line separating nearby declarations or logic blocks.
  **L2459 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2460 EN**: Declares TableGen def `int_aarch64_sve_saba`.
  **L2460 CN**: 声明 TableGen def `int_aarch64_sve_saba`。
- **L2461 EN**: Declares TableGen def `int_aarch64_sve_sabal`.
  **L2461 CN**: 声明 TableGen def `int_aarch64_sve_sabal`。
- **L2462 EN**: Declares TableGen def `int_aarch64_sve_uabal`.
  **L2462 CN**: 声明 TableGen def `int_aarch64_sve_uabal`。
- **L2463 EN**: Declares TableGen def `int_aarch64_sve_shadd`.
  **L2463 CN**: 声明 TableGen def `int_aarch64_sve_shadd`。
- **L2464 EN**: Declares TableGen def `int_aarch64_sve_shsub`.
  **L2464 CN**: 声明 TableGen def `int_aarch64_sve_shsub`。

### Lines 2465-2496

````tablegen
def int_aarch64_sve_shsub_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_shsubr        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sli           : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqabs         : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqadd         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmulh       : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmulh_lane  : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqneg         : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdmlah      : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdmlah_lane : AdvSIMD_3VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdmlsh      : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdmlsh_lane : AdvSIMD_3VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdmulh      : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdmulh_lane : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrshl        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrshl_u      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqshl         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqshl_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqshlu        : AdvSIMD_SVE_ShiftByImm_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqsub         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqsub_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqsubr        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_srhadd        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sri           : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_srshl         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_srshl_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_srshr         : AdvSIMD_SVE_ShiftByImm_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_srsra         : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ssra          : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_suqadd        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uaba          : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uhadd         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L2465 EN**: Declares TableGen def `int_aarch64_sve_shsub_u`.
  **L2465 CN**: 声明 TableGen def `int_aarch64_sve_shsub_u`。
- **L2466 EN**: Declares TableGen def `int_aarch64_sve_shsubr`.
  **L2466 CN**: 声明 TableGen def `int_aarch64_sve_shsubr`。
- **L2467 EN**: Declares TableGen def `int_aarch64_sve_sli`.
  **L2467 CN**: 声明 TableGen def `int_aarch64_sve_sli`。
- **L2468 EN**: Declares TableGen def `int_aarch64_sve_sqabs`.
  **L2468 CN**: 声明 TableGen def `int_aarch64_sve_sqabs`。
- **L2469 EN**: Declares TableGen def `int_aarch64_sve_sqadd`.
  **L2469 CN**: 声明 TableGen def `int_aarch64_sve_sqadd`。
- **L2470 EN**: Declares TableGen def `int_aarch64_sve_sqdmulh`.
  **L2470 CN**: 声明 TableGen def `int_aarch64_sve_sqdmulh`。
- **L2471 EN**: Declares TableGen def `int_aarch64_sve_sqdmulh_lane`.
  **L2471 CN**: 声明 TableGen def `int_aarch64_sve_sqdmulh_lane`。
- **L2472 EN**: Declares TableGen def `int_aarch64_sve_sqneg`.
  **L2472 CN**: 声明 TableGen def `int_aarch64_sve_sqneg`。
- **L2473 EN**: Declares TableGen def `int_aarch64_sve_sqrdmlah`.
  **L2473 CN**: 声明 TableGen def `int_aarch64_sve_sqrdmlah`。
- **L2474 EN**: Declares TableGen def `int_aarch64_sve_sqrdmlah_lane`.
  **L2474 CN**: 声明 TableGen def `int_aarch64_sve_sqrdmlah_lane`。
- **L2475 EN**: Declares TableGen def `int_aarch64_sve_sqrdmlsh`.
  **L2475 CN**: 声明 TableGen def `int_aarch64_sve_sqrdmlsh`。
- **L2476 EN**: Declares TableGen def `int_aarch64_sve_sqrdmlsh_lane`.
  **L2476 CN**: 声明 TableGen def `int_aarch64_sve_sqrdmlsh_lane`。
- **L2477 EN**: Declares TableGen def `int_aarch64_sve_sqrdmulh`.
  **L2477 CN**: 声明 TableGen def `int_aarch64_sve_sqrdmulh`。
- **L2478 EN**: Declares TableGen def `int_aarch64_sve_sqrdmulh_lane`.
  **L2478 CN**: 声明 TableGen def `int_aarch64_sve_sqrdmulh_lane`。
- **L2479 EN**: Declares TableGen def `int_aarch64_sve_sqrshl`.
  **L2479 CN**: 声明 TableGen def `int_aarch64_sve_sqrshl`。
- **L2480 EN**: Declares TableGen def `int_aarch64_sve_sqrshl_u`.
  **L2480 CN**: 声明 TableGen def `int_aarch64_sve_sqrshl_u`。
- **L2481 EN**: Declares TableGen def `int_aarch64_sve_sqshl`.
  **L2481 CN**: 声明 TableGen def `int_aarch64_sve_sqshl`。
- **L2482 EN**: Declares TableGen def `int_aarch64_sve_sqshl_u`.
  **L2482 CN**: 声明 TableGen def `int_aarch64_sve_sqshl_u`。
- **L2483 EN**: Declares TableGen def `int_aarch64_sve_sqshlu`.
  **L2483 CN**: 声明 TableGen def `int_aarch64_sve_sqshlu`。
- **L2484 EN**: Declares TableGen def `int_aarch64_sve_sqsub`.
  **L2484 CN**: 声明 TableGen def `int_aarch64_sve_sqsub`。
- **L2485 EN**: Declares TableGen def `int_aarch64_sve_sqsub_u`.
  **L2485 CN**: 声明 TableGen def `int_aarch64_sve_sqsub_u`。
- **L2486 EN**: Declares TableGen def `int_aarch64_sve_sqsubr`.
  **L2486 CN**: 声明 TableGen def `int_aarch64_sve_sqsubr`。
- **L2487 EN**: Declares TableGen def `int_aarch64_sve_srhadd`.
  **L2487 CN**: 声明 TableGen def `int_aarch64_sve_srhadd`。
- **L2488 EN**: Declares TableGen def `int_aarch64_sve_sri`.
  **L2488 CN**: 声明 TableGen def `int_aarch64_sve_sri`。
- **L2489 EN**: Declares TableGen def `int_aarch64_sve_srshl`.
  **L2489 CN**: 声明 TableGen def `int_aarch64_sve_srshl`。
- **L2490 EN**: Declares TableGen def `int_aarch64_sve_srshl_u`.
  **L2490 CN**: 声明 TableGen def `int_aarch64_sve_srshl_u`。
- **L2491 EN**: Declares TableGen def `int_aarch64_sve_srshr`.
  **L2491 CN**: 声明 TableGen def `int_aarch64_sve_srshr`。
- **L2492 EN**: Declares TableGen def `int_aarch64_sve_srsra`.
  **L2492 CN**: 声明 TableGen def `int_aarch64_sve_srsra`。
- **L2493 EN**: Declares TableGen def `int_aarch64_sve_ssra`.
  **L2493 CN**: 声明 TableGen def `int_aarch64_sve_ssra`。
- **L2494 EN**: Declares TableGen def `int_aarch64_sve_suqadd`.
  **L2494 CN**: 声明 TableGen def `int_aarch64_sve_suqadd`。
- **L2495 EN**: Declares TableGen def `int_aarch64_sve_uaba`.
  **L2495 CN**: 声明 TableGen def `int_aarch64_sve_uaba`。
- **L2496 EN**: Declares TableGen def `int_aarch64_sve_uhadd`.
  **L2496 CN**: 声明 TableGen def `int_aarch64_sve_uhadd`。

### Lines 2497-2528

````tablegen
def int_aarch64_sve_uhsub         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uhsub_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uhsubr        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqadd         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqrshl        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqrshl_u      : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqshl         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqshl_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqsub         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqsub_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqsubr        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_urecpe        : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_urhadd        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_urshl         : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_urshl_u       : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_urshr         : AdvSIMD_SVE_ShiftByImm_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ursqrte       : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ursra         : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usqadd        : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usra          : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Widening DSP operations
//

def int_aarch64_sve_sabalb : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sabalt : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sabdlb : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sabdlt : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_saddlb : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_saddlt : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_saddwb : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;
````
- **L2497 EN**: Declares TableGen def `int_aarch64_sve_uhsub`.
  **L2497 CN**: 声明 TableGen def `int_aarch64_sve_uhsub`。
- **L2498 EN**: Declares TableGen def `int_aarch64_sve_uhsub_u`.
  **L2498 CN**: 声明 TableGen def `int_aarch64_sve_uhsub_u`。
- **L2499 EN**: Declares TableGen def `int_aarch64_sve_uhsubr`.
  **L2499 CN**: 声明 TableGen def `int_aarch64_sve_uhsubr`。
- **L2500 EN**: Declares TableGen def `int_aarch64_sve_uqadd`.
  **L2500 CN**: 声明 TableGen def `int_aarch64_sve_uqadd`。
- **L2501 EN**: Declares TableGen def `int_aarch64_sve_uqrshl`.
  **L2501 CN**: 声明 TableGen def `int_aarch64_sve_uqrshl`。
- **L2502 EN**: Declares TableGen def `int_aarch64_sve_uqrshl_u`.
  **L2502 CN**: 声明 TableGen def `int_aarch64_sve_uqrshl_u`。
- **L2503 EN**: Declares TableGen def `int_aarch64_sve_uqshl`.
  **L2503 CN**: 声明 TableGen def `int_aarch64_sve_uqshl`。
- **L2504 EN**: Declares TableGen def `int_aarch64_sve_uqshl_u`.
  **L2504 CN**: 声明 TableGen def `int_aarch64_sve_uqshl_u`。
- **L2505 EN**: Declares TableGen def `int_aarch64_sve_uqsub`.
  **L2505 CN**: 声明 TableGen def `int_aarch64_sve_uqsub`。
- **L2506 EN**: Declares TableGen def `int_aarch64_sve_uqsub_u`.
  **L2506 CN**: 声明 TableGen def `int_aarch64_sve_uqsub_u`。
- **L2507 EN**: Declares TableGen def `int_aarch64_sve_uqsubr`.
  **L2507 CN**: 声明 TableGen def `int_aarch64_sve_uqsubr`。
- **L2508 EN**: Declares TableGen def `int_aarch64_sve_urecpe`.
  **L2508 CN**: 声明 TableGen def `int_aarch64_sve_urecpe`。
- **L2509 EN**: Declares TableGen def `int_aarch64_sve_urhadd`.
  **L2509 CN**: 声明 TableGen def `int_aarch64_sve_urhadd`。
- **L2510 EN**: Declares TableGen def `int_aarch64_sve_urshl`.
  **L2510 CN**: 声明 TableGen def `int_aarch64_sve_urshl`。
- **L2511 EN**: Declares TableGen def `int_aarch64_sve_urshl_u`.
  **L2511 CN**: 声明 TableGen def `int_aarch64_sve_urshl_u`。
- **L2512 EN**: Declares TableGen def `int_aarch64_sve_urshr`.
  **L2512 CN**: 声明 TableGen def `int_aarch64_sve_urshr`。
- **L2513 EN**: Declares TableGen def `int_aarch64_sve_ursqrte`.
  **L2513 CN**: 声明 TableGen def `int_aarch64_sve_ursqrte`。
- **L2514 EN**: Declares TableGen def `int_aarch64_sve_ursra`.
  **L2514 CN**: 声明 TableGen def `int_aarch64_sve_ursra`。
- **L2515 EN**: Declares TableGen def `int_aarch64_sve_usqadd`.
  **L2515 CN**: 声明 TableGen def `int_aarch64_sve_usqadd`。
- **L2516 EN**: Declares TableGen def `int_aarch64_sve_usra`.
  **L2516 CN**: 声明 TableGen def `int_aarch64_sve_usra`。
- **L2517 EN**: Blank line separating nearby declarations or logic blocks.
  **L2517 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2518 EN**: Separator comment used for visual grouping.
  **L2518 CN**: 用于视觉分组的分隔注释。
- **L2519 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Widening DSP operations`.
  **L2519 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Widening DSP operations`。
- **L2520 EN**: Separator comment used for visual grouping.
  **L2520 CN**: 用于视觉分组的分隔注释。
- **L2521 EN**: Blank line separating nearby declarations or logic blocks.
  **L2521 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2522 EN**: Declares TableGen def `int_aarch64_sve_sabalb`.
  **L2522 CN**: 声明 TableGen def `int_aarch64_sve_sabalb`。
- **L2523 EN**: Declares TableGen def `int_aarch64_sve_sabalt`.
  **L2523 CN**: 声明 TableGen def `int_aarch64_sve_sabalt`。
- **L2524 EN**: Declares TableGen def `int_aarch64_sve_sabdlb`.
  **L2524 CN**: 声明 TableGen def `int_aarch64_sve_sabdlb`。
- **L2525 EN**: Declares TableGen def `int_aarch64_sve_sabdlt`.
  **L2525 CN**: 声明 TableGen def `int_aarch64_sve_sabdlt`。
- **L2526 EN**: Declares TableGen def `int_aarch64_sve_saddlb`.
  **L2526 CN**: 声明 TableGen def `int_aarch64_sve_saddlb`。
- **L2527 EN**: Declares TableGen def `int_aarch64_sve_saddlt`.
  **L2527 CN**: 声明 TableGen def `int_aarch64_sve_saddlt`。
- **L2528 EN**: Declares TableGen def `int_aarch64_sve_saddwb`.
  **L2528 CN**: 声明 TableGen def `int_aarch64_sve_saddwb`。

### Lines 2529-2560

````tablegen
def int_aarch64_sve_saddwt : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sshllb : SVE2_1VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sshllt : SVE2_1VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ssublb : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ssublt : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ssubwb : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ssubwt : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uabalb : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uabalt : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uabdlb : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uabdlt : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uaddlb : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uaddlt : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uaddwb : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uaddwt : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ushllb : SVE2_1VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ushllt : SVE2_1VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usublb : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usublt : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usubwb : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usubwt : SVE2_2VectorArg_Wide_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Non-widening pairwise arithmetic
//

def int_aarch64_sve_addp    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_faddp   : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmaxp   : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fmaxnmp : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fminp   : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_fminnmp : AdvSIMD_Pred2VectorArg_Intrinsic;
````
- **L2529 EN**: Declares TableGen def `int_aarch64_sve_saddwt`.
  **L2529 CN**: 声明 TableGen def `int_aarch64_sve_saddwt`。
- **L2530 EN**: Declares TableGen def `int_aarch64_sve_sshllb`.
  **L2530 CN**: 声明 TableGen def `int_aarch64_sve_sshllb`。
- **L2531 EN**: Declares TableGen def `int_aarch64_sve_sshllt`.
  **L2531 CN**: 声明 TableGen def `int_aarch64_sve_sshllt`。
- **L2532 EN**: Declares TableGen def `int_aarch64_sve_ssublb`.
  **L2532 CN**: 声明 TableGen def `int_aarch64_sve_ssublb`。
- **L2533 EN**: Declares TableGen def `int_aarch64_sve_ssublt`.
  **L2533 CN**: 声明 TableGen def `int_aarch64_sve_ssublt`。
- **L2534 EN**: Declares TableGen def `int_aarch64_sve_ssubwb`.
  **L2534 CN**: 声明 TableGen def `int_aarch64_sve_ssubwb`。
- **L2535 EN**: Declares TableGen def `int_aarch64_sve_ssubwt`.
  **L2535 CN**: 声明 TableGen def `int_aarch64_sve_ssubwt`。
- **L2536 EN**: Declares TableGen def `int_aarch64_sve_uabalb`.
  **L2536 CN**: 声明 TableGen def `int_aarch64_sve_uabalb`。
- **L2537 EN**: Declares TableGen def `int_aarch64_sve_uabalt`.
  **L2537 CN**: 声明 TableGen def `int_aarch64_sve_uabalt`。
- **L2538 EN**: Declares TableGen def `int_aarch64_sve_uabdlb`.
  **L2538 CN**: 声明 TableGen def `int_aarch64_sve_uabdlb`。
- **L2539 EN**: Declares TableGen def `int_aarch64_sve_uabdlt`.
  **L2539 CN**: 声明 TableGen def `int_aarch64_sve_uabdlt`。
- **L2540 EN**: Declares TableGen def `int_aarch64_sve_uaddlb`.
  **L2540 CN**: 声明 TableGen def `int_aarch64_sve_uaddlb`。
- **L2541 EN**: Declares TableGen def `int_aarch64_sve_uaddlt`.
  **L2541 CN**: 声明 TableGen def `int_aarch64_sve_uaddlt`。
- **L2542 EN**: Declares TableGen def `int_aarch64_sve_uaddwb`.
  **L2542 CN**: 声明 TableGen def `int_aarch64_sve_uaddwb`。
- **L2543 EN**: Declares TableGen def `int_aarch64_sve_uaddwt`.
  **L2543 CN**: 声明 TableGen def `int_aarch64_sve_uaddwt`。
- **L2544 EN**: Declares TableGen def `int_aarch64_sve_ushllb`.
  **L2544 CN**: 声明 TableGen def `int_aarch64_sve_ushllb`。
- **L2545 EN**: Declares TableGen def `int_aarch64_sve_ushllt`.
  **L2545 CN**: 声明 TableGen def `int_aarch64_sve_ushllt`。
- **L2546 EN**: Declares TableGen def `int_aarch64_sve_usublb`.
  **L2546 CN**: 声明 TableGen def `int_aarch64_sve_usublb`。
- **L2547 EN**: Declares TableGen def `int_aarch64_sve_usublt`.
  **L2547 CN**: 声明 TableGen def `int_aarch64_sve_usublt`。
- **L2548 EN**: Declares TableGen def `int_aarch64_sve_usubwb`.
  **L2548 CN**: 声明 TableGen def `int_aarch64_sve_usubwb`。
- **L2549 EN**: Declares TableGen def `int_aarch64_sve_usubwt`.
  **L2549 CN**: 声明 TableGen def `int_aarch64_sve_usubwt`。
- **L2550 EN**: Blank line separating nearby declarations or logic blocks.
  **L2550 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2551 EN**: Separator comment used for visual grouping.
  **L2551 CN**: 用于视觉分组的分隔注释。
- **L2552 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Non-widening pairwise arithmetic`.
  **L2552 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Non-widening pairwise arithmetic`。
- **L2553 EN**: Separator comment used for visual grouping.
  **L2553 CN**: 用于视觉分组的分隔注释。
- **L2554 EN**: Blank line separating nearby declarations or logic blocks.
  **L2554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2555 EN**: Declares TableGen def `int_aarch64_sve_addp`.
  **L2555 CN**: 声明 TableGen def `int_aarch64_sve_addp`。
- **L2556 EN**: Declares TableGen def `int_aarch64_sve_faddp`.
  **L2556 CN**: 声明 TableGen def `int_aarch64_sve_faddp`。
- **L2557 EN**: Declares TableGen def `int_aarch64_sve_fmaxp`.
  **L2557 CN**: 声明 TableGen def `int_aarch64_sve_fmaxp`。
- **L2558 EN**: Declares TableGen def `int_aarch64_sve_fmaxnmp`.
  **L2558 CN**: 声明 TableGen def `int_aarch64_sve_fmaxnmp`。
- **L2559 EN**: Declares TableGen def `int_aarch64_sve_fminp`.
  **L2559 CN**: 声明 TableGen def `int_aarch64_sve_fminp`。
- **L2560 EN**: Declares TableGen def `int_aarch64_sve_fminnmp`.
  **L2560 CN**: 声明 TableGen def `int_aarch64_sve_fminnmp`。

### Lines 2561-2592

````tablegen
def int_aarch64_sve_smaxp   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sminp   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umaxp   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uminp   : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_addqp   : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_addsubp : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_subp    : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Widening pairwise arithmetic
//

def int_aarch64_sve_sadalp : SVE2_2VectorArg_Pred_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uadalp : SVE2_2VectorArg_Pred_Long_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Uniform complex integer arithmetic
//

def int_aarch64_sve_cadd_x           : AdvSIMD_SVE2_CADD_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqcadd_x         : AdvSIMD_SVE2_CADD_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmla_x           : AdvSIMD_SVE2_CMLA_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cmla_lane_x      : AdvSIMD_SVE_CMLA_LANE_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdcmlah_x      : AdvSIMD_SVE2_CMLA_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrdcmlah_lane_x : AdvSIMD_SVE_CMLA_LANE_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Widening complex integer arithmetic
//

def int_aarch64_sve_saddlbt   : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
````
- **L2561 EN**: Declares TableGen def `int_aarch64_sve_smaxp`.
  **L2561 CN**: 声明 TableGen def `int_aarch64_sve_smaxp`。
- **L2562 EN**: Declares TableGen def `int_aarch64_sve_sminp`.
  **L2562 CN**: 声明 TableGen def `int_aarch64_sve_sminp`。
- **L2563 EN**: Declares TableGen def `int_aarch64_sve_umaxp`.
  **L2563 CN**: 声明 TableGen def `int_aarch64_sve_umaxp`。
- **L2564 EN**: Declares TableGen def `int_aarch64_sve_uminp`.
  **L2564 CN**: 声明 TableGen def `int_aarch64_sve_uminp`。
- **L2565 EN**: Blank line separating nearby declarations or logic blocks.
  **L2565 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2566 EN**: Declares TableGen def `int_aarch64_sve_addqp`.
  **L2566 CN**: 声明 TableGen def `int_aarch64_sve_addqp`。
- **L2567 EN**: Declares TableGen def `int_aarch64_sve_addsubp`.
  **L2567 CN**: 声明 TableGen def `int_aarch64_sve_addsubp`。
- **L2568 EN**: Declares TableGen def `int_aarch64_sve_subp`.
  **L2568 CN**: 声明 TableGen def `int_aarch64_sve_subp`。
- **L2569 EN**: Blank line separating nearby declarations or logic blocks.
  **L2569 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2570 EN**: Separator comment used for visual grouping.
  **L2570 CN**: 用于视觉分组的分隔注释。
- **L2571 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Widening pairwise arithmetic`.
  **L2571 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Widening pairwise arithmetic`。
- **L2572 EN**: Separator comment used for visual grouping.
  **L2572 CN**: 用于视觉分组的分隔注释。
- **L2573 EN**: Blank line separating nearby declarations or logic blocks.
  **L2573 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2574 EN**: Declares TableGen def `int_aarch64_sve_sadalp`.
  **L2574 CN**: 声明 TableGen def `int_aarch64_sve_sadalp`。
- **L2575 EN**: Declares TableGen def `int_aarch64_sve_uadalp`.
  **L2575 CN**: 声明 TableGen def `int_aarch64_sve_uadalp`。
- **L2576 EN**: Blank line separating nearby declarations or logic blocks.
  **L2576 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2577 EN**: Separator comment used for visual grouping.
  **L2577 CN**: 用于视觉分组的分隔注释。
- **L2578 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Uniform complex integer arithmetic`.
  **L2578 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Uniform complex integer arithmetic`。
- **L2579 EN**: Separator comment used for visual grouping.
  **L2579 CN**: 用于视觉分组的分隔注释。
- **L2580 EN**: Blank line separating nearby declarations or logic blocks.
  **L2580 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2581 EN**: Declares TableGen def `int_aarch64_sve_cadd_x`.
  **L2581 CN**: 声明 TableGen def `int_aarch64_sve_cadd_x`。
- **L2582 EN**: Declares TableGen def `int_aarch64_sve_sqcadd_x`.
  **L2582 CN**: 声明 TableGen def `int_aarch64_sve_sqcadd_x`。
- **L2583 EN**: Declares TableGen def `int_aarch64_sve_cmla_x`.
  **L2583 CN**: 声明 TableGen def `int_aarch64_sve_cmla_x`。
- **L2584 EN**: Declares TableGen def `int_aarch64_sve_cmla_lane_x`.
  **L2584 CN**: 声明 TableGen def `int_aarch64_sve_cmla_lane_x`。
- **L2585 EN**: Declares TableGen def `int_aarch64_sve_sqrdcmlah_x`.
  **L2585 CN**: 声明 TableGen def `int_aarch64_sve_sqrdcmlah_x`。
- **L2586 EN**: Declares TableGen def `int_aarch64_sve_sqrdcmlah_lane_x`.
  **L2586 CN**: 声明 TableGen def `int_aarch64_sve_sqrdcmlah_lane_x`。
- **L2587 EN**: Blank line separating nearby declarations or logic blocks.
  **L2587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2588 EN**: Separator comment used for visual grouping.
  **L2588 CN**: 用于视觉分组的分隔注释。
- **L2589 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Widening complex integer arithmetic`.
  **L2589 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Widening complex integer arithmetic`。
- **L2590 EN**: Separator comment used for visual grouping.
  **L2590 CN**: 用于视觉分组的分隔注释。
- **L2591 EN**: Blank line separating nearby declarations or logic blocks.
  **L2591 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2592 EN**: Declares TableGen def `int_aarch64_sve_saddlbt`.
  **L2592 CN**: 声明 TableGen def `int_aarch64_sve_saddlbt`。

### Lines 2593-2624

````tablegen
def int_aarch64_sve_ssublbt   : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_ssubltb   : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Widening complex integer dot product
//

def int_aarch64_sve_cdot      : AdvSIMD_SVE_DOT_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_cdot_lane : AdvSIMD_SVE_CDOT_LANE_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Floating-point widening multiply-accumulate
//

def int_aarch64_sve_fmlalb        : SVE2_3VectorArg_Long_Intrinsic;
def int_aarch64_sve_fmlalb_lane   : SVE2_3VectorArgIndexed_Long_Intrinsic;
def int_aarch64_sve_fmlalt        : SVE2_3VectorArg_Long_Intrinsic;
def int_aarch64_sve_fmlalt_lane   : SVE2_3VectorArgIndexed_Long_Intrinsic;
def int_aarch64_sve_fmlslb        : SVE2_3VectorArg_Long_Intrinsic;
def int_aarch64_sve_fmlslb_lane   : SVE2_3VectorArgIndexed_Long_Intrinsic;
def int_aarch64_sve_fmlslt        : SVE2_3VectorArg_Long_Intrinsic;
def int_aarch64_sve_fmlslt_lane   : SVE2_3VectorArgIndexed_Long_Intrinsic;

//
// SVE2 - Floating-point integer binary logarithm
//

def int_aarch64_sve_flogb : AdvSIMD_SVE_LOGB_Intrinsic;

//
// SVE2 - Vector histogram count
//
````
- **L2593 EN**: Declares TableGen def `int_aarch64_sve_ssublbt`.
  **L2593 CN**: 声明 TableGen def `int_aarch64_sve_ssublbt`。
- **L2594 EN**: Declares TableGen def `int_aarch64_sve_ssubltb`.
  **L2594 CN**: 声明 TableGen def `int_aarch64_sve_ssubltb`。
- **L2595 EN**: Blank line separating nearby declarations or logic blocks.
  **L2595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2596 EN**: Separator comment used for visual grouping.
  **L2596 CN**: 用于视觉分组的分隔注释。
- **L2597 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Widening complex integer dot product`.
  **L2597 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Widening complex integer dot product`。
- **L2598 EN**: Separator comment used for visual grouping.
  **L2598 CN**: 用于视觉分组的分隔注释。
- **L2599 EN**: Blank line separating nearby declarations or logic blocks.
  **L2599 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2600 EN**: Declares TableGen def `int_aarch64_sve_cdot`.
  **L2600 CN**: 声明 TableGen def `int_aarch64_sve_cdot`。
- **L2601 EN**: Declares TableGen def `int_aarch64_sve_cdot_lane`.
  **L2601 CN**: 声明 TableGen def `int_aarch64_sve_cdot_lane`。
- **L2602 EN**: Blank line separating nearby declarations or logic blocks.
  **L2602 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2603 EN**: Separator comment used for visual grouping.
  **L2603 CN**: 用于视觉分组的分隔注释。
- **L2604 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Floating-point widening multiply-accumulate`.
  **L2604 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Floating-point widening multiply-accumulate`。
- **L2605 EN**: Separator comment used for visual grouping.
  **L2605 CN**: 用于视觉分组的分隔注释。
- **L2606 EN**: Blank line separating nearby declarations or logic blocks.
  **L2606 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2607 EN**: Declares TableGen def `int_aarch64_sve_fmlalb`.
  **L2607 CN**: 声明 TableGen def `int_aarch64_sve_fmlalb`。
- **L2608 EN**: Declares TableGen def `int_aarch64_sve_fmlalb_lane`.
  **L2608 CN**: 声明 TableGen def `int_aarch64_sve_fmlalb_lane`。
- **L2609 EN**: Declares TableGen def `int_aarch64_sve_fmlalt`.
  **L2609 CN**: 声明 TableGen def `int_aarch64_sve_fmlalt`。
- **L2610 EN**: Declares TableGen def `int_aarch64_sve_fmlalt_lane`.
  **L2610 CN**: 声明 TableGen def `int_aarch64_sve_fmlalt_lane`。
- **L2611 EN**: Declares TableGen def `int_aarch64_sve_fmlslb`.
  **L2611 CN**: 声明 TableGen def `int_aarch64_sve_fmlslb`。
- **L2612 EN**: Declares TableGen def `int_aarch64_sve_fmlslb_lane`.
  **L2612 CN**: 声明 TableGen def `int_aarch64_sve_fmlslb_lane`。
- **L2613 EN**: Declares TableGen def `int_aarch64_sve_fmlslt`.
  **L2613 CN**: 声明 TableGen def `int_aarch64_sve_fmlslt`。
- **L2614 EN**: Declares TableGen def `int_aarch64_sve_fmlslt_lane`.
  **L2614 CN**: 声明 TableGen def `int_aarch64_sve_fmlslt_lane`。
- **L2615 EN**: Blank line separating nearby declarations or logic blocks.
  **L2615 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2616 EN**: Separator comment used for visual grouping.
  **L2616 CN**: 用于视觉分组的分隔注释。
- **L2617 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Floating-point integer binary logarithm`.
  **L2617 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Floating-point integer binary logarithm`。
- **L2618 EN**: Separator comment used for visual grouping.
  **L2618 CN**: 用于视觉分组的分隔注释。
- **L2619 EN**: Blank line separating nearby declarations or logic blocks.
  **L2619 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2620 EN**: Declares TableGen def `int_aarch64_sve_flogb`.
  **L2620 CN**: 声明 TableGen def `int_aarch64_sve_flogb`。
- **L2621 EN**: Blank line separating nearby declarations or logic blocks.
  **L2621 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2622 EN**: Separator comment used for visual grouping.
  **L2622 CN**: 用于视觉分组的分隔注释。
- **L2623 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Vector histogram count`.
  **L2623 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Vector histogram count`。
- **L2624 EN**: Separator comment used for visual grouping.
  **L2624 CN**: 用于视觉分组的分隔注释。

### Lines 2625-2656

````tablegen

def int_aarch64_sve_histcnt : AdvSIMD_Pred2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_histseg : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Character match
//

def int_aarch64_sve_match   : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_nmatch  : AdvSIMD_SVE_Compare_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Unary narrowing operations
//

def int_aarch64_sve_sqxtnb  : SVE2_1VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqxtnt  : SVE2_Merged1VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqxtunb : SVE2_1VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqxtunt : SVE2_Merged1VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqxtnb  : SVE2_1VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqxtnt  : SVE2_Merged1VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Binary narrowing DSP operations
//
def int_aarch64_sve_addhnb    : SVE2_2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_addhnt    : SVE2_Merged2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_raddhnb   : SVE2_2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_raddhnt   : SVE2_Merged2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_subhnb    : SVE2_2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
````
- **L2625 EN**: Blank line separating nearby declarations or logic blocks.
  **L2625 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2626 EN**: Declares TableGen def `int_aarch64_sve_histcnt`.
  **L2626 CN**: 声明 TableGen def `int_aarch64_sve_histcnt`。
- **L2627 EN**: Declares TableGen def `int_aarch64_sve_histseg`.
  **L2627 CN**: 声明 TableGen def `int_aarch64_sve_histseg`。
- **L2628 EN**: Blank line separating nearby declarations or logic blocks.
  **L2628 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2629 EN**: Separator comment used for visual grouping.
  **L2629 CN**: 用于视觉分组的分隔注释。
- **L2630 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Character match`.
  **L2630 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Character match`。
- **L2631 EN**: Separator comment used for visual grouping.
  **L2631 CN**: 用于视觉分组的分隔注释。
- **L2632 EN**: Blank line separating nearby declarations or logic blocks.
  **L2632 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2633 EN**: Declares TableGen def `int_aarch64_sve_match`.
  **L2633 CN**: 声明 TableGen def `int_aarch64_sve_match`。
- **L2634 EN**: Declares TableGen def `int_aarch64_sve_nmatch`.
  **L2634 CN**: 声明 TableGen def `int_aarch64_sve_nmatch`。
- **L2635 EN**: Blank line separating nearby declarations or logic blocks.
  **L2635 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2636 EN**: Separator comment used for visual grouping.
  **L2636 CN**: 用于视觉分组的分隔注释。
- **L2637 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Unary narrowing operations`.
  **L2637 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Unary narrowing operations`。
- **L2638 EN**: Separator comment used for visual grouping.
  **L2638 CN**: 用于视觉分组的分隔注释。
- **L2639 EN**: Blank line separating nearby declarations or logic blocks.
  **L2639 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2640 EN**: Declares TableGen def `int_aarch64_sve_sqxtnb`.
  **L2640 CN**: 声明 TableGen def `int_aarch64_sve_sqxtnb`。
- **L2641 EN**: Declares TableGen def `int_aarch64_sve_sqxtnt`.
  **L2641 CN**: 声明 TableGen def `int_aarch64_sve_sqxtnt`。
- **L2642 EN**: Declares TableGen def `int_aarch64_sve_sqxtunb`.
  **L2642 CN**: 声明 TableGen def `int_aarch64_sve_sqxtunb`。
- **L2643 EN**: Declares TableGen def `int_aarch64_sve_sqxtunt`.
  **L2643 CN**: 声明 TableGen def `int_aarch64_sve_sqxtunt`。
- **L2644 EN**: Declares TableGen def `int_aarch64_sve_uqxtnb`.
  **L2644 CN**: 声明 TableGen def `int_aarch64_sve_uqxtnb`。
- **L2645 EN**: Declares TableGen def `int_aarch64_sve_uqxtnt`.
  **L2645 CN**: 声明 TableGen def `int_aarch64_sve_uqxtnt`。
- **L2646 EN**: Blank line separating nearby declarations or logic blocks.
  **L2646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2647 EN**: Separator comment used for visual grouping.
  **L2647 CN**: 用于视觉分组的分隔注释。
- **L2648 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Binary narrowing DSP operations`.
  **L2648 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Binary narrowing DSP operations`。
- **L2649 EN**: Separator comment used for visual grouping.
  **L2649 CN**: 用于视觉分组的分隔注释。
- **L2650 EN**: Declares TableGen def `int_aarch64_sve_addhnb`.
  **L2650 CN**: 声明 TableGen def `int_aarch64_sve_addhnb`。
- **L2651 EN**: Declares TableGen def `int_aarch64_sve_addhnt`.
  **L2651 CN**: 声明 TableGen def `int_aarch64_sve_addhnt`。
- **L2652 EN**: Blank line separating nearby declarations or logic blocks.
  **L2652 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2653 EN**: Declares TableGen def `int_aarch64_sve_raddhnb`.
  **L2653 CN**: 声明 TableGen def `int_aarch64_sve_raddhnb`。
- **L2654 EN**: Declares TableGen def `int_aarch64_sve_raddhnt`.
  **L2654 CN**: 声明 TableGen def `int_aarch64_sve_raddhnt`。
- **L2655 EN**: Blank line separating nearby declarations or logic blocks.
  **L2655 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2656 EN**: Declares TableGen def `int_aarch64_sve_subhnb`.
  **L2656 CN**: 声明 TableGen def `int_aarch64_sve_subhnb`。

### Lines 2657-2688

````tablegen
def int_aarch64_sve_subhnt    : SVE2_Merged2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_rsubhnb   : SVE2_2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_rsubhnt   : SVE2_Merged2VectorArg_Narrowing_Intrinsic<[IntrSpeculatable]>;

// Narrowing shift right
def int_aarch64_sve_shrnb     : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_shrnt     : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_rshrnb    : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_rshrnt    : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;

// Saturating shift right - signed input/output
def int_aarch64_sve_sqshrnb   : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqshrnt   : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sqrshrnb  : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrshrnt  : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;

// Saturating shift right - unsigned input/output
def int_aarch64_sve_uqshrnb   : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqshrnt   : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_uqrshrnb  : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uqrshrnt  : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;

// Saturating shift right - signed input, unsigned output
def int_aarch64_sve_sqshrunb  : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqshrunt  : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sqrshrunb : SVE2_1VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqrshrunt : SVE2_2VectorArg_Imm_Narrowing_Intrinsic<[IntrSpeculatable]>;
````
- **L2657 EN**: Declares TableGen def `int_aarch64_sve_subhnt`.
  **L2657 CN**: 声明 TableGen def `int_aarch64_sve_subhnt`。
- **L2658 EN**: Blank line separating nearby declarations or logic blocks.
  **L2658 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2659 EN**: Declares TableGen def `int_aarch64_sve_rsubhnb`.
  **L2659 CN**: 声明 TableGen def `int_aarch64_sve_rsubhnb`。
- **L2660 EN**: Declares TableGen def `int_aarch64_sve_rsubhnt`.
  **L2660 CN**: 声明 TableGen def `int_aarch64_sve_rsubhnt`。
- **L2661 EN**: Blank line separating nearby declarations or logic blocks.
  **L2661 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2662 EN**: Comment explains nearby logic, invariants, or intent: `Narrowing shift right`.
  **L2662 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Narrowing shift right`。
- **L2663 EN**: Declares TableGen def `int_aarch64_sve_shrnb`.
  **L2663 CN**: 声明 TableGen def `int_aarch64_sve_shrnb`。
- **L2664 EN**: Declares TableGen def `int_aarch64_sve_shrnt`.
  **L2664 CN**: 声明 TableGen def `int_aarch64_sve_shrnt`。
- **L2665 EN**: Blank line separating nearby declarations or logic blocks.
  **L2665 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2666 EN**: Declares TableGen def `int_aarch64_sve_rshrnb`.
  **L2666 CN**: 声明 TableGen def `int_aarch64_sve_rshrnb`。
- **L2667 EN**: Declares TableGen def `int_aarch64_sve_rshrnt`.
  **L2667 CN**: 声明 TableGen def `int_aarch64_sve_rshrnt`。
- **L2668 EN**: Blank line separating nearby declarations or logic blocks.
  **L2668 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2669 EN**: Comment explains nearby logic, invariants, or intent: `Saturating shift right - signed input/output`.
  **L2669 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating shift right - signed input/output`。
- **L2670 EN**: Declares TableGen def `int_aarch64_sve_sqshrnb`.
  **L2670 CN**: 声明 TableGen def `int_aarch64_sve_sqshrnb`。
- **L2671 EN**: Declares TableGen def `int_aarch64_sve_sqshrnt`.
  **L2671 CN**: 声明 TableGen def `int_aarch64_sve_sqshrnt`。
- **L2672 EN**: Blank line separating nearby declarations or logic blocks.
  **L2672 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2673 EN**: Declares TableGen def `int_aarch64_sve_sqrshrnb`.
  **L2673 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrnb`。
- **L2674 EN**: Declares TableGen def `int_aarch64_sve_sqrshrnt`.
  **L2674 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrnt`。
- **L2675 EN**: Blank line separating nearby declarations or logic blocks.
  **L2675 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2676 EN**: Comment explains nearby logic, invariants, or intent: `Saturating shift right - unsigned input/output`.
  **L2676 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating shift right - unsigned input/output`。
- **L2677 EN**: Declares TableGen def `int_aarch64_sve_uqshrnb`.
  **L2677 CN**: 声明 TableGen def `int_aarch64_sve_uqshrnb`。
- **L2678 EN**: Declares TableGen def `int_aarch64_sve_uqshrnt`.
  **L2678 CN**: 声明 TableGen def `int_aarch64_sve_uqshrnt`。
- **L2679 EN**: Blank line separating nearby declarations or logic blocks.
  **L2679 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2680 EN**: Declares TableGen def `int_aarch64_sve_uqrshrnb`.
  **L2680 CN**: 声明 TableGen def `int_aarch64_sve_uqrshrnb`。
- **L2681 EN**: Declares TableGen def `int_aarch64_sve_uqrshrnt`.
  **L2681 CN**: 声明 TableGen def `int_aarch64_sve_uqrshrnt`。
- **L2682 EN**: Blank line separating nearby declarations or logic blocks.
  **L2682 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2683 EN**: Comment explains nearby logic, invariants, or intent: `Saturating shift right - signed input, unsigned output`.
  **L2683 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating shift right - signed input, unsigned output`。
- **L2684 EN**: Declares TableGen def `int_aarch64_sve_sqshrunb`.
  **L2684 CN**: 声明 TableGen def `int_aarch64_sve_sqshrunb`。
- **L2685 EN**: Declares TableGen def `int_aarch64_sve_sqshrunt`.
  **L2685 CN**: 声明 TableGen def `int_aarch64_sve_sqshrunt`。
- **L2686 EN**: Blank line separating nearby declarations or logic blocks.
  **L2686 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2687 EN**: Declares TableGen def `int_aarch64_sve_sqrshrunb`.
  **L2687 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrunb`。
- **L2688 EN**: Declares TableGen def `int_aarch64_sve_sqrshrunt`.
  **L2688 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrunt`。

### Lines 2689-2720

````tablegen

// SVE2 MLA LANE.
def int_aarch64_sve_smlalb_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smlalt_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlalb_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlalt_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smlslb_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smlslt_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlslb_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlslt_lane   : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smullb_lane   : SVE2_2VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smullt_lane   : SVE2_2VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umullb_lane   : SVE2_2VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umullt_lane   : SVE2_2VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlalb_lane : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlalt_lane : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlslb_lane : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlslt_lane : SVE2_3VectorArg_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmullb_lane : SVE2_2VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmullt_lane : SVE2_2VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;

// SVE2 MLA Unpredicated.
def int_aarch64_sve_smlalb      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smlalt      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlalb      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlalt      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smlslb      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smlslt      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlslb      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umlslt      : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smullb      : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smullt      : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
````
- **L2689 EN**: Blank line separating nearby declarations or logic blocks.
  **L2689 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2690 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 MLA LANE.`.
  **L2690 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 MLA LANE.`。
- **L2691 EN**: Declares TableGen def `int_aarch64_sve_smlalb_lane`.
  **L2691 CN**: 声明 TableGen def `int_aarch64_sve_smlalb_lane`。
- **L2692 EN**: Declares TableGen def `int_aarch64_sve_smlalt_lane`.
  **L2692 CN**: 声明 TableGen def `int_aarch64_sve_smlalt_lane`。
- **L2693 EN**: Declares TableGen def `int_aarch64_sve_umlalb_lane`.
  **L2693 CN**: 声明 TableGen def `int_aarch64_sve_umlalb_lane`。
- **L2694 EN**: Declares TableGen def `int_aarch64_sve_umlalt_lane`.
  **L2694 CN**: 声明 TableGen def `int_aarch64_sve_umlalt_lane`。
- **L2695 EN**: Declares TableGen def `int_aarch64_sve_smlslb_lane`.
  **L2695 CN**: 声明 TableGen def `int_aarch64_sve_smlslb_lane`。
- **L2696 EN**: Declares TableGen def `int_aarch64_sve_smlslt_lane`.
  **L2696 CN**: 声明 TableGen def `int_aarch64_sve_smlslt_lane`。
- **L2697 EN**: Declares TableGen def `int_aarch64_sve_umlslb_lane`.
  **L2697 CN**: 声明 TableGen def `int_aarch64_sve_umlslb_lane`。
- **L2698 EN**: Declares TableGen def `int_aarch64_sve_umlslt_lane`.
  **L2698 CN**: 声明 TableGen def `int_aarch64_sve_umlslt_lane`。
- **L2699 EN**: Declares TableGen def `int_aarch64_sve_smullb_lane`.
  **L2699 CN**: 声明 TableGen def `int_aarch64_sve_smullb_lane`。
- **L2700 EN**: Declares TableGen def `int_aarch64_sve_smullt_lane`.
  **L2700 CN**: 声明 TableGen def `int_aarch64_sve_smullt_lane`。
- **L2701 EN**: Declares TableGen def `int_aarch64_sve_umullb_lane`.
  **L2701 CN**: 声明 TableGen def `int_aarch64_sve_umullb_lane`。
- **L2702 EN**: Declares TableGen def `int_aarch64_sve_umullt_lane`.
  **L2702 CN**: 声明 TableGen def `int_aarch64_sve_umullt_lane`。
- **L2703 EN**: Declares TableGen def `int_aarch64_sve_sqdmlalb_lane`.
  **L2703 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlalb_lane`。
- **L2704 EN**: Declares TableGen def `int_aarch64_sve_sqdmlalt_lane`.
  **L2704 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlalt_lane`。
- **L2705 EN**: Declares TableGen def `int_aarch64_sve_sqdmlslb_lane`.
  **L2705 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlslb_lane`。
- **L2706 EN**: Declares TableGen def `int_aarch64_sve_sqdmlslt_lane`.
  **L2706 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlslt_lane`。
- **L2707 EN**: Declares TableGen def `int_aarch64_sve_sqdmullb_lane`.
  **L2707 CN**: 声明 TableGen def `int_aarch64_sve_sqdmullb_lane`。
- **L2708 EN**: Declares TableGen def `int_aarch64_sve_sqdmullt_lane`.
  **L2708 CN**: 声明 TableGen def `int_aarch64_sve_sqdmullt_lane`。
- **L2709 EN**: Blank line separating nearby declarations or logic blocks.
  **L2709 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2710 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 MLA Unpredicated.`.
  **L2710 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 MLA Unpredicated.`。
- **L2711 EN**: Declares TableGen def `int_aarch64_sve_smlalb`.
  **L2711 CN**: 声明 TableGen def `int_aarch64_sve_smlalb`。
- **L2712 EN**: Declares TableGen def `int_aarch64_sve_smlalt`.
  **L2712 CN**: 声明 TableGen def `int_aarch64_sve_smlalt`。
- **L2713 EN**: Declares TableGen def `int_aarch64_sve_umlalb`.
  **L2713 CN**: 声明 TableGen def `int_aarch64_sve_umlalb`。
- **L2714 EN**: Declares TableGen def `int_aarch64_sve_umlalt`.
  **L2714 CN**: 声明 TableGen def `int_aarch64_sve_umlalt`。
- **L2715 EN**: Declares TableGen def `int_aarch64_sve_smlslb`.
  **L2715 CN**: 声明 TableGen def `int_aarch64_sve_smlslb`。
- **L2716 EN**: Declares TableGen def `int_aarch64_sve_smlslt`.
  **L2716 CN**: 声明 TableGen def `int_aarch64_sve_smlslt`。
- **L2717 EN**: Declares TableGen def `int_aarch64_sve_umlslb`.
  **L2717 CN**: 声明 TableGen def `int_aarch64_sve_umlslb`。
- **L2718 EN**: Declares TableGen def `int_aarch64_sve_umlslt`.
  **L2718 CN**: 声明 TableGen def `int_aarch64_sve_umlslt`。
- **L2719 EN**: Declares TableGen def `int_aarch64_sve_smullb`.
  **L2719 CN**: 声明 TableGen def `int_aarch64_sve_smullb`。
- **L2720 EN**: Declares TableGen def `int_aarch64_sve_smullt`.
  **L2720 CN**: 声明 TableGen def `int_aarch64_sve_smullt`。

### Lines 2721-2752

````tablegen
def int_aarch64_sve_umullb      : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_umullt      : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_sqdmlalb    : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlalt    : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlslb    : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlslt    : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmullb    : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmullt    : SVE2_2VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlalbt   : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sqdmlslbt   : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;

// SVE2 ADDSUB Long Unpredicated.
def int_aarch64_sve_adclb       : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_adclt       : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sbclb       : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sbclt       : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Polynomial arithmetic
//
def int_aarch64_sve_eorbt       : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_eortb       : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_pmullb_pair : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_pmullt_pair : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 bitwise ternary operations.
//
def int_aarch64_sve_eor3   : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bcax   : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bsl    : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L2721 EN**: Declares TableGen def `int_aarch64_sve_umullb`.
  **L2721 CN**: 声明 TableGen def `int_aarch64_sve_umullb`。
- **L2722 EN**: Declares TableGen def `int_aarch64_sve_umullt`.
  **L2722 CN**: 声明 TableGen def `int_aarch64_sve_umullt`。
- **L2723 EN**: Blank line separating nearby declarations or logic blocks.
  **L2723 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2724 EN**: Declares TableGen def `int_aarch64_sve_sqdmlalb`.
  **L2724 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlalb`。
- **L2725 EN**: Declares TableGen def `int_aarch64_sve_sqdmlalt`.
  **L2725 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlalt`。
- **L2726 EN**: Declares TableGen def `int_aarch64_sve_sqdmlslb`.
  **L2726 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlslb`。
- **L2727 EN**: Declares TableGen def `int_aarch64_sve_sqdmlslt`.
  **L2727 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlslt`。
- **L2728 EN**: Declares TableGen def `int_aarch64_sve_sqdmullb`.
  **L2728 CN**: 声明 TableGen def `int_aarch64_sve_sqdmullb`。
- **L2729 EN**: Declares TableGen def `int_aarch64_sve_sqdmullt`.
  **L2729 CN**: 声明 TableGen def `int_aarch64_sve_sqdmullt`。
- **L2730 EN**: Declares TableGen def `int_aarch64_sve_sqdmlalbt`.
  **L2730 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlalbt`。
- **L2731 EN**: Declares TableGen def `int_aarch64_sve_sqdmlslbt`.
  **L2731 CN**: 声明 TableGen def `int_aarch64_sve_sqdmlslbt`。
- **L2732 EN**: Blank line separating nearby declarations or logic blocks.
  **L2732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2733 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 ADDSUB Long Unpredicated.`.
  **L2733 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 ADDSUB Long Unpredicated.`。
- **L2734 EN**: Declares TableGen def `int_aarch64_sve_adclb`.
  **L2734 CN**: 声明 TableGen def `int_aarch64_sve_adclb`。
- **L2735 EN**: Declares TableGen def `int_aarch64_sve_adclt`.
  **L2735 CN**: 声明 TableGen def `int_aarch64_sve_adclt`。
- **L2736 EN**: Declares TableGen def `int_aarch64_sve_sbclb`.
  **L2736 CN**: 声明 TableGen def `int_aarch64_sve_sbclb`。
- **L2737 EN**: Declares TableGen def `int_aarch64_sve_sbclt`.
  **L2737 CN**: 声明 TableGen def `int_aarch64_sve_sbclt`。
- **L2738 EN**: Blank line separating nearby declarations or logic blocks.
  **L2738 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2739 EN**: Separator comment used for visual grouping.
  **L2739 CN**: 用于视觉分组的分隔注释。
- **L2740 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Polynomial arithmetic`.
  **L2740 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Polynomial arithmetic`。
- **L2741 EN**: Separator comment used for visual grouping.
  **L2741 CN**: 用于视觉分组的分隔注释。
- **L2742 EN**: Declares TableGen def `int_aarch64_sve_eorbt`.
  **L2742 CN**: 声明 TableGen def `int_aarch64_sve_eorbt`。
- **L2743 EN**: Declares TableGen def `int_aarch64_sve_eortb`.
  **L2743 CN**: 声明 TableGen def `int_aarch64_sve_eortb`。
- **L2744 EN**: Declares TableGen def `int_aarch64_sve_pmullb_pair`.
  **L2744 CN**: 声明 TableGen def `int_aarch64_sve_pmullb_pair`。
- **L2745 EN**: Declares TableGen def `int_aarch64_sve_pmullt_pair`.
  **L2745 CN**: 声明 TableGen def `int_aarch64_sve_pmullt_pair`。
- **L2746 EN**: Blank line separating nearby declarations or logic blocks.
  **L2746 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2747 EN**: Separator comment used for visual grouping.
  **L2747 CN**: 用于视觉分组的分隔注释。
- **L2748 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 bitwise ternary operations.`.
  **L2748 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 bitwise ternary operations.`。
- **L2749 EN**: Separator comment used for visual grouping.
  **L2749 CN**: 用于视觉分组的分隔注释。
- **L2750 EN**: Declares TableGen def `int_aarch64_sve_eor3`.
  **L2750 CN**: 声明 TableGen def `int_aarch64_sve_eor3`。
- **L2751 EN**: Declares TableGen def `int_aarch64_sve_bcax`.
  **L2751 CN**: 声明 TableGen def `int_aarch64_sve_bcax`。
- **L2752 EN**: Declares TableGen def `int_aarch64_sve_bsl`.
  **L2752 CN**: 声明 TableGen def `int_aarch64_sve_bsl`。

### Lines 2753-2784

````tablegen
def int_aarch64_sve_bsl1n  : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bsl2n  : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_nbsl   : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_xar    : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Optional AES, SHA-3 and SM4
//

def int_aarch64_sve_aesd    : ClangBuiltin<"__builtin_sve_svaesd_u8">,
                              DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                                        [llvm_nxv16i8_ty, llvm_nxv16i8_ty],
                                        [IntrNoMem, IntrSpeculatable]>;
def int_aarch64_sve_aesimc  : ClangBuiltin<"__builtin_sve_svaesimc_u8">,
                              DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                                        [llvm_nxv16i8_ty],
                                        [IntrNoMem, IntrSpeculatable]>;
def int_aarch64_sve_aese    : ClangBuiltin<"__builtin_sve_svaese_u8">,
                              DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                                        [llvm_nxv16i8_ty, llvm_nxv16i8_ty],
                                        [IntrNoMem, IntrSpeculatable]>;
def int_aarch64_sve_aesmc   : ClangBuiltin<"__builtin_sve_svaesmc_u8">,
                              DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                                        [llvm_nxv16i8_ty],
                                        [IntrNoMem, IntrSpeculatable]>;
def int_aarch64_sve_rax1    : ClangBuiltin<"__builtin_sve_svrax1_u64">,
                              DefaultAttrsIntrinsic<[llvm_nxv2i64_ty],
                                        [llvm_nxv2i64_ty, llvm_nxv2i64_ty],
                                        [IntrNoMem, IntrSpeculatable]>;
def int_aarch64_sve_sm4e    : ClangBuiltin<"__builtin_sve_svsm4e_u32">,
                              DefaultAttrsIntrinsic<[llvm_nxv4i32_ty],
                                        [llvm_nxv4i32_ty, llvm_nxv4i32_ty],
````
- **L2753 EN**: Declares TableGen def `int_aarch64_sve_bsl1n`.
  **L2753 CN**: 声明 TableGen def `int_aarch64_sve_bsl1n`。
- **L2754 EN**: Declares TableGen def `int_aarch64_sve_bsl2n`.
  **L2754 CN**: 声明 TableGen def `int_aarch64_sve_bsl2n`。
- **L2755 EN**: Declares TableGen def `int_aarch64_sve_nbsl`.
  **L2755 CN**: 声明 TableGen def `int_aarch64_sve_nbsl`。
- **L2756 EN**: Declares TableGen def `int_aarch64_sve_xar`.
  **L2756 CN**: 声明 TableGen def `int_aarch64_sve_xar`。
- **L2757 EN**: Blank line separating nearby declarations or logic blocks.
  **L2757 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2758 EN**: Separator comment used for visual grouping.
  **L2758 CN**: 用于视觉分组的分隔注释。
- **L2759 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Optional AES, SHA-3 and SM4`.
  **L2759 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Optional AES, SHA-3 and SM4`。
- **L2760 EN**: Separator comment used for visual grouping.
  **L2760 CN**: 用于视觉分组的分隔注释。
- **L2761 EN**: Blank line separating nearby declarations or logic blocks.
  **L2761 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2762 EN**: Declares TableGen def `int_aarch64_sve_aesd`.
  **L2762 CN**: 声明 TableGen def `int_aarch64_sve_aesd`。
- **L2763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L2763 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L2764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L2764 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L2765 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2765 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2766 EN**: Declares TableGen def `int_aarch64_sve_aesimc`.
  **L2766 CN**: 声明 TableGen def `int_aarch64_sve_aesimc`。
- **L2767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L2767 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L2768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty],`.
  **L2768 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty],`。
- **L2769 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2769 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2770 EN**: Declares TableGen def `int_aarch64_sve_aese`.
  **L2770 CN**: 声明 TableGen def `int_aarch64_sve_aese`。
- **L2771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L2771 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L2772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L2772 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L2773 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2773 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2774 EN**: Declares TableGen def `int_aarch64_sve_aesmc`.
  **L2774 CN**: 声明 TableGen def `int_aarch64_sve_aesmc`。
- **L2775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L2775 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L2776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty],`.
  **L2776 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty],`。
- **L2777 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2777 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2778 EN**: Declares TableGen def `int_aarch64_sve_rax1`.
  **L2778 CN**: 声明 TableGen def `int_aarch64_sve_rax1`。
- **L2779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_nxv2i64_ty],`.
  **L2779 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_nxv2i64_ty],`。
- **L2780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv2i64_ty, llvm_nxv2i64_ty],`.
  **L2780 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv2i64_ty, llvm_nxv2i64_ty],`。
- **L2781 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2781 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2782 EN**: Declares TableGen def `int_aarch64_sve_sm4e`.
  **L2782 CN**: 声明 TableGen def `int_aarch64_sve_sm4e`。
- **L2783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_nxv4i32_ty],`.
  **L2783 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_nxv4i32_ty],`。
- **L2784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4i32_ty, llvm_nxv4i32_ty],`.
  **L2784 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4i32_ty, llvm_nxv4i32_ty],`。

### Lines 2785-2816

````tablegen
                                        [IntrNoMem, IntrSpeculatable]>;
def int_aarch64_sve_sm4ekey : ClangBuiltin<"__builtin_sve_svsm4ekey_u32">,
                              DefaultAttrsIntrinsic<[llvm_nxv4i32_ty],
                                        [llvm_nxv4i32_ty, llvm_nxv4i32_ty],
                                        [IntrNoMem, IntrSpeculatable]>;
//
// SVE2 - Extended table lookup/permute
//

def int_aarch64_sve_tbl2 : AdvSIMD_SVE2_TBX_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_tbx  : AdvSIMD_SVE2_TBX_Intrinsic<[IntrSpeculatable]>;

//
// SVE2 - Lookup Table
//

def int_aarch64_sve_luti2_lane : SVE2_LUTI_Inrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_luti4_lane : SVE2_LUTI_Inrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_luti4_lane_x2 : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                                    [LLVMMatchType<0>,
                                    LLVMMatchType<0>,
                                    llvm_nxv16i8_ty,
                                    llvm_i32_ty],
                                    [IntrNoMem, ImmArg<ArgIndex<3>>, IntrSpeculatable]>;

//
// SVE2 - Optional bit permutation
//

def int_aarch64_sve_bdep_x : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bext_x : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_bgrp_x : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
````
- **L2785 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2785 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2786 EN**: Declares TableGen def `int_aarch64_sve_sm4ekey`.
  **L2786 CN**: 声明 TableGen def `int_aarch64_sve_sm4ekey`。
- **L2787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_nxv4i32_ty],`.
  **L2787 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_nxv4i32_ty],`。
- **L2788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4i32_ty, llvm_nxv4i32_ty],`.
  **L2788 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4i32_ty, llvm_nxv4i32_ty],`。
- **L2789 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrSpeculatable]>;`.
  **L2789 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrSpeculatable]>;`。
- **L2790 EN**: Separator comment used for visual grouping.
  **L2790 CN**: 用于视觉分组的分隔注释。
- **L2791 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Extended table lookup/permute`.
  **L2791 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Extended table lookup/permute`。
- **L2792 EN**: Separator comment used for visual grouping.
  **L2792 CN**: 用于视觉分组的分隔注释。
- **L2793 EN**: Blank line separating nearby declarations or logic blocks.
  **L2793 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2794 EN**: Declares TableGen def `int_aarch64_sve_tbl2`.
  **L2794 CN**: 声明 TableGen def `int_aarch64_sve_tbl2`。
- **L2795 EN**: Declares TableGen def `int_aarch64_sve_tbx`.
  **L2795 CN**: 声明 TableGen def `int_aarch64_sve_tbx`。
- **L2796 EN**: Blank line separating nearby declarations or logic blocks.
  **L2796 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2797 EN**: Separator comment used for visual grouping.
  **L2797 CN**: 用于视觉分组的分隔注释。
- **L2798 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Lookup Table`.
  **L2798 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Lookup Table`。
- **L2799 EN**: Separator comment used for visual grouping.
  **L2799 CN**: 用于视觉分组的分隔注释。
- **L2800 EN**: Blank line separating nearby declarations or logic blocks.
  **L2800 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2801 EN**: Declares TableGen def `int_aarch64_sve_luti2_lane`.
  **L2801 CN**: 声明 TableGen def `int_aarch64_sve_luti2_lane`。
- **L2802 EN**: Declares TableGen def `int_aarch64_sve_luti4_lane`.
  **L2802 CN**: 声明 TableGen def `int_aarch64_sve_luti4_lane`。
- **L2803 EN**: Declares TableGen def `int_aarch64_sve_luti4_lane_x2`.
  **L2803 CN**: 声明 TableGen def `int_aarch64_sve_luti4_lane_x2`。
- **L2804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L2804 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L2805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L2805 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L2806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L2806 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L2807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L2807 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L2808 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>, IntrSpeculatable]>;`.
  **L2808 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>, IntrSpeculatable]>;`。
- **L2809 EN**: Blank line separating nearby declarations or logic blocks.
  **L2809 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2810 EN**: Separator comment used for visual grouping.
  **L2810 CN**: 用于视觉分组的分隔注释。
- **L2811 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Optional bit permutation`.
  **L2811 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Optional bit permutation`。
- **L2812 EN**: Separator comment used for visual grouping.
  **L2812 CN**: 用于视觉分组的分隔注释。
- **L2813 EN**: Blank line separating nearby declarations or logic blocks.
  **L2813 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2814 EN**: Declares TableGen def `int_aarch64_sve_bdep_x`.
  **L2814 CN**: 声明 TableGen def `int_aarch64_sve_bdep_x`。
- **L2815 EN**: Declares TableGen def `int_aarch64_sve_bext_x`.
  **L2815 CN**: 声明 TableGen def `int_aarch64_sve_bext_x`。
- **L2816 EN**: Declares TableGen def `int_aarch64_sve_bgrp_x`.
  **L2816 CN**: 声明 TableGen def `int_aarch64_sve_bgrp_x`。

### Lines 2817-2848

````tablegen


//
// SVE ACLE: 7.3. INT8 matrix multiply extensions
//
def int_aarch64_sve_ummla : SVE_MatMul_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_smmla : SVE_MatMul_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usmmla : SVE_MatMul_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_usdot : AdvSIMD_SVE_DOT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_usdot_lane : AdvSIMD_SVE_DOT_Indexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_sudot_lane : AdvSIMD_SVE_DOT_Indexed_Intrinsic<[IntrSpeculatable]>;

//
// SVE ACLE: 7.4/5. FP64/FP32 matrix multiply extensions
//

def int_aarch64_sve_fmmla
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],
                          [IntrNoMem]>;

def int_aarch64_sve_fp8_fmmla
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, llvm_nxv16i8_ty, llvm_nxv16i8_ty],
                          [IntrReadMem, IntrInaccessibleMemOnly]>;

//
// SVE ACLE: 7.2. BFloat16 extensions
//

def int_aarch64_sve_bfdot   : SVE_4Vec_BF16;
````
- **L2817 EN**: Blank line separating nearby declarations or logic blocks.
  **L2817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2818 EN**: Blank line separating nearby declarations or logic blocks.
  **L2818 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2819 EN**: Separator comment used for visual grouping.
  **L2819 CN**: 用于视觉分组的分隔注释。
- **L2820 EN**: Comment explains nearby logic, invariants, or intent: `SVE ACLE: 7.3. INT8 matrix multiply extensions`.
  **L2820 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE ACLE: 7.3. INT8 matrix multiply extensions`。
- **L2821 EN**: Separator comment used for visual grouping.
  **L2821 CN**: 用于视觉分组的分隔注释。
- **L2822 EN**: Declares TableGen def `int_aarch64_sve_ummla`.
  **L2822 CN**: 声明 TableGen def `int_aarch64_sve_ummla`。
- **L2823 EN**: Declares TableGen def `int_aarch64_sve_smmla`.
  **L2823 CN**: 声明 TableGen def `int_aarch64_sve_smmla`。
- **L2824 EN**: Declares TableGen def `int_aarch64_sve_usmmla`.
  **L2824 CN**: 声明 TableGen def `int_aarch64_sve_usmmla`。
- **L2825 EN**: Blank line separating nearby declarations or logic blocks.
  **L2825 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2826 EN**: Declares TableGen def `int_aarch64_sve_usdot`.
  **L2826 CN**: 声明 TableGen def `int_aarch64_sve_usdot`。
- **L2827 EN**: Declares TableGen def `int_aarch64_sve_usdot_lane`.
  **L2827 CN**: 声明 TableGen def `int_aarch64_sve_usdot_lane`。
- **L2828 EN**: Declares TableGen def `int_aarch64_sve_sudot_lane`.
  **L2828 CN**: 声明 TableGen def `int_aarch64_sve_sudot_lane`。
- **L2829 EN**: Blank line separating nearby declarations or logic blocks.
  **L2829 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2830 EN**: Separator comment used for visual grouping.
  **L2830 CN**: 用于视觉分组的分隔注释。
- **L2831 EN**: Comment explains nearby logic, invariants, or intent: `SVE ACLE: 7.4/5. FP64/FP32 matrix multiply extensions`.
  **L2831 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE ACLE: 7.4/5. FP64/FP32 matrix multiply extensions`。
- **L2832 EN**: Separator comment used for visual grouping.
  **L2832 CN**: 用于视觉分组的分隔注释。
- **L2833 EN**: Blank line separating nearby declarations or logic blocks.
  **L2833 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2834 EN**: Declares TableGen def `int_aarch64_sve_fmmla`.
  **L2834 CN**: 声明 TableGen def `int_aarch64_sve_fmmla`。
- **L2835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L2835 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L2836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`.
  **L2836 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<1>],`。
- **L2837 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2837 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2838 EN**: Blank line separating nearby declarations or logic blocks.
  **L2838 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2839 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmmla`.
  **L2839 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmmla`。
- **L2840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L2840 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L2841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L2841 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L2842 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L2842 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L2843 EN**: Blank line separating nearby declarations or logic blocks.
  **L2843 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2844 EN**: Separator comment used for visual grouping.
  **L2844 CN**: 用于视觉分组的分隔注释。
- **L2845 EN**: Comment explains nearby logic, invariants, or intent: `SVE ACLE: 7.2. BFloat16 extensions`.
  **L2845 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE ACLE: 7.2. BFloat16 extensions`。
- **L2846 EN**: Separator comment used for visual grouping.
  **L2846 CN**: 用于视觉分组的分隔注释。
- **L2847 EN**: Blank line separating nearby declarations or logic blocks.
  **L2847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2848 EN**: Declares TableGen def `int_aarch64_sve_bfdot`.
  **L2848 CN**: 声明 TableGen def `int_aarch64_sve_bfdot`。

### Lines 2849-2880

````tablegen
def int_aarch64_sve_bfmlalb : SVE_4Vec_BF16;
def int_aarch64_sve_bfmlalt : SVE_4Vec_BF16;

def int_aarch64_sve_bfdot_lane_v2   : SVE_4Vec_BF16_Indexed;
def int_aarch64_sve_bfmlalb_lane_v2 : SVE_4Vec_BF16_Indexed;
def int_aarch64_sve_bfmlalt_lane_v2 : SVE_4Vec_BF16_Indexed;

//
// SVE2.1 - Contiguous loads to multiple consecutive vectors
//

  class SVE2p1_Load_PN_X2_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [llvm_aarch64_svcount_ty, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;

  class SVE2p1_Load_PN_X4_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>, LLVMMatchType<0>],
                [llvm_aarch64_svcount_ty, llvm_anyptr_ty],
                [IntrReadMem, IntrArgMemOnly]>;

def int_aarch64_sve_ld1_pn_x2 : SVE2p1_Load_PN_X2_Intrinsic;
def int_aarch64_sve_ld1_pn_x4 : SVE2p1_Load_PN_X4_Intrinsic;
def int_aarch64_sve_ldnt1_pn_x2 : SVE2p1_Load_PN_X2_Intrinsic;
def int_aarch64_sve_ldnt1_pn_x4 : SVE2p1_Load_PN_X4_Intrinsic;

//
// SVE2.1 - Contiguous loads to quadword (single vector)
//

class SVE2p1_Single_Load_Quadword
````
- **L2849 EN**: Declares TableGen def `int_aarch64_sve_bfmlalb`.
  **L2849 CN**: 声明 TableGen def `int_aarch64_sve_bfmlalb`。
- **L2850 EN**: Declares TableGen def `int_aarch64_sve_bfmlalt`.
  **L2850 CN**: 声明 TableGen def `int_aarch64_sve_bfmlalt`。
- **L2851 EN**: Blank line separating nearby declarations or logic blocks.
  **L2851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2852 EN**: Declares TableGen def `int_aarch64_sve_bfdot_lane_v2`.
  **L2852 CN**: 声明 TableGen def `int_aarch64_sve_bfdot_lane_v2`。
- **L2853 EN**: Declares TableGen def `int_aarch64_sve_bfmlalb_lane_v2`.
  **L2853 CN**: 声明 TableGen def `int_aarch64_sve_bfmlalb_lane_v2`。
- **L2854 EN**: Declares TableGen def `int_aarch64_sve_bfmlalt_lane_v2`.
  **L2854 CN**: 声明 TableGen def `int_aarch64_sve_bfmlalt_lane_v2`。
- **L2855 EN**: Blank line separating nearby declarations or logic blocks.
  **L2855 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2856 EN**: Separator comment used for visual grouping.
  **L2856 CN**: 用于视觉分组的分隔注释。
- **L2857 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - Contiguous loads to multiple consecutive vectors`.
  **L2857 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - Contiguous loads to multiple consecutive vectors`。
- **L2858 EN**: Separator comment used for visual grouping.
  **L2858 CN**: 用于视觉分组的分隔注释。
- **L2859 EN**: Blank line separating nearby declarations or logic blocks.
  **L2859 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2860 EN**: Declares class `SVE2p1_Load_PN_X2_Intrinsic`.
  **L2860 CN**: 声明 class `SVE2p1_Load_PN_X2_Intrinsic`。
- **L2861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L2861 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L2862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_aarch64_svcount_ty, llvm_anyptr_ty],`.
  **L2862 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_aarch64_svcount_ty, llvm_anyptr_ty],`。
- **L2863 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L2863 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L2864 EN**: Blank line separating nearby declarations or logic blocks.
  **L2864 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2865 EN**: Declares class `SVE2p1_Load_PN_X4_Intrinsic`.
  **L2865 CN**: 声明 class `SVE2p1_Load_PN_X4_Intrinsic`。
- **L2866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L2866 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L2867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L2867 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L2868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_aarch64_svcount_ty, llvm_anyptr_ty],`.
  **L2868 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_aarch64_svcount_ty, llvm_anyptr_ty],`。
- **L2869 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L2869 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L2870 EN**: Blank line separating nearby declarations or logic blocks.
  **L2870 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2871 EN**: Declares TableGen def `int_aarch64_sve_ld1_pn_x2`.
  **L2871 CN**: 声明 TableGen def `int_aarch64_sve_ld1_pn_x2`。
- **L2872 EN**: Declares TableGen def `int_aarch64_sve_ld1_pn_x4`.
  **L2872 CN**: 声明 TableGen def `int_aarch64_sve_ld1_pn_x4`。
- **L2873 EN**: Declares TableGen def `int_aarch64_sve_ldnt1_pn_x2`.
  **L2873 CN**: 声明 TableGen def `int_aarch64_sve_ldnt1_pn_x2`。
- **L2874 EN**: Declares TableGen def `int_aarch64_sve_ldnt1_pn_x4`.
  **L2874 CN**: 声明 TableGen def `int_aarch64_sve_ldnt1_pn_x4`。
- **L2875 EN**: Blank line separating nearby declarations or logic blocks.
  **L2875 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2876 EN**: Separator comment used for visual grouping.
  **L2876 CN**: 用于视觉分组的分隔注释。
- **L2877 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - Contiguous loads to quadword (single vector)`.
  **L2877 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - Contiguous loads to quadword (single vector)`。
- **L2878 EN**: Separator comment used for visual grouping.
  **L2878 CN**: 用于视觉分组的分隔注释。
- **L2879 EN**: Blank line separating nearby declarations or logic blocks.
  **L2879 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2880 EN**: Declares class `SVE2p1_Single_Load_Quadword`.
  **L2880 CN**: 声明 class `SVE2p1_Single_Load_Quadword`。

### Lines 2881-2912

````tablegen
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [llvm_nxv1i1_ty, llvm_anyptr_ty],
                            [IntrReadMem, IntrArgMemOnly]>;
def int_aarch64_sve_ld1uwq : SVE2p1_Single_Load_Quadword;
def int_aarch64_sve_ld1udq : SVE2p1_Single_Load_Quadword;

//
// SVE2.1 - Contiguous store from quadword (single vector)
//

class SVE2p1_Single_Store_Quadword
    : DefaultAttrsIntrinsic<[],
                            [llvm_anyvector_ty, llvm_nxv1i1_ty, llvm_anyptr_ty],
                            [IntrWriteMem, IntrArgMemOnly]>;
def int_aarch64_sve_st1wq : SVE2p1_Single_Store_Quadword;
def int_aarch64_sve_st1dq : SVE2p1_Single_Store_Quadword;


def int_aarch64_sve_ld2q_sret : AdvSIMD_2Vec_PredLoad_Intrinsic;
def int_aarch64_sve_ld3q_sret : AdvSIMD_3Vec_PredLoad_Intrinsic;
def int_aarch64_sve_ld4q_sret : AdvSIMD_4Vec_PredLoad_Intrinsic;

def int_aarch64_sve_st2q : AdvSIMD_2Vec_PredStore_Intrinsic;
def int_aarch64_sve_st3q : AdvSIMD_3Vec_PredStore_Intrinsic;
def int_aarch64_sve_st4q : AdvSIMD_4Vec_PredStore_Intrinsic;

//
// SVE2.1 - Contiguous stores to multiple consecutive vectors
//

  class SVE2p1_Store_PN_X2_Intrinsic
    : DefaultAttrsIntrinsic<[], [ llvm_anyvector_ty, LLVMMatchType<0>,
````
- **L2881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L2881 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L2882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv1i1_ty, llvm_anyptr_ty],`.
  **L2882 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv1i1_ty, llvm_anyptr_ty],`。
- **L2883 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L2883 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L2884 EN**: Declares TableGen def `int_aarch64_sve_ld1uwq`.
  **L2884 CN**: 声明 TableGen def `int_aarch64_sve_ld1uwq`。
- **L2885 EN**: Declares TableGen def `int_aarch64_sve_ld1udq`.
  **L2885 CN**: 声明 TableGen def `int_aarch64_sve_ld1udq`。
- **L2886 EN**: Blank line separating nearby declarations or logic blocks.
  **L2886 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2887 EN**: Separator comment used for visual grouping.
  **L2887 CN**: 用于视觉分组的分隔注释。
- **L2888 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - Contiguous store from quadword (single vector)`.
  **L2888 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - Contiguous store from quadword (single vector)`。
- **L2889 EN**: Separator comment used for visual grouping.
  **L2889 CN**: 用于视觉分组的分隔注释。
- **L2890 EN**: Blank line separating nearby declarations or logic blocks.
  **L2890 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2891 EN**: Declares class `SVE2p1_Single_Store_Quadword`.
  **L2891 CN**: 声明 class `SVE2p1_Single_Store_Quadword`。
- **L2892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L2892 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L2893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_nxv1i1_ty, llvm_anyptr_ty],`.
  **L2893 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_nxv1i1_ty, llvm_anyptr_ty],`。
- **L2894 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L2894 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L2895 EN**: Declares TableGen def `int_aarch64_sve_st1wq`.
  **L2895 CN**: 声明 TableGen def `int_aarch64_sve_st1wq`。
- **L2896 EN**: Declares TableGen def `int_aarch64_sve_st1dq`.
  **L2896 CN**: 声明 TableGen def `int_aarch64_sve_st1dq`。
- **L2897 EN**: Blank line separating nearby declarations or logic blocks.
  **L2897 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2898 EN**: Blank line separating nearby declarations or logic blocks.
  **L2898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2899 EN**: Declares TableGen def `int_aarch64_sve_ld2q_sret`.
  **L2899 CN**: 声明 TableGen def `int_aarch64_sve_ld2q_sret`。
- **L2900 EN**: Declares TableGen def `int_aarch64_sve_ld3q_sret`.
  **L2900 CN**: 声明 TableGen def `int_aarch64_sve_ld3q_sret`。
- **L2901 EN**: Declares TableGen def `int_aarch64_sve_ld4q_sret`.
  **L2901 CN**: 声明 TableGen def `int_aarch64_sve_ld4q_sret`。
- **L2902 EN**: Blank line separating nearby declarations or logic blocks.
  **L2902 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2903 EN**: Declares TableGen def `int_aarch64_sve_st2q`.
  **L2903 CN**: 声明 TableGen def `int_aarch64_sve_st2q`。
- **L2904 EN**: Declares TableGen def `int_aarch64_sve_st3q`.
  **L2904 CN**: 声明 TableGen def `int_aarch64_sve_st3q`。
- **L2905 EN**: Declares TableGen def `int_aarch64_sve_st4q`.
  **L2905 CN**: 声明 TableGen def `int_aarch64_sve_st4q`。
- **L2906 EN**: Blank line separating nearby declarations or logic blocks.
  **L2906 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2907 EN**: Separator comment used for visual grouping.
  **L2907 CN**: 用于视觉分组的分隔注释。
- **L2908 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - Contiguous stores to multiple consecutive vectors`.
  **L2908 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - Contiguous stores to multiple consecutive vectors`。
- **L2909 EN**: Separator comment used for visual grouping.
  **L2909 CN**: 用于视觉分组的分隔注释。
- **L2910 EN**: Blank line separating nearby declarations or logic blocks.
  **L2910 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2911 EN**: Declares class `SVE2p1_Store_PN_X2_Intrinsic`.
  **L2911 CN**: 声明 class `SVE2p1_Store_PN_X2_Intrinsic`。
- **L2912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [ llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L2912 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [ llvm_anyvector_ty, LLVMMatchType<0>,`。

### Lines 2913-2944

````tablegen
                                  llvm_aarch64_svcount_ty, llvm_anyptr_ty ],
                [IntrWriteMem, IntrArgMemOnly]>;

  class SVE2p1_Store_PN_X4_Intrinsic
    : DefaultAttrsIntrinsic<[], [ llvm_anyvector_ty, LLVMMatchType<0>,
                                  LLVMMatchType<0>, LLVMMatchType<0>,
                                  llvm_aarch64_svcount_ty, llvm_anyptr_ty],
                [IntrWriteMem, IntrArgMemOnly]>;

def int_aarch64_sve_st1_pn_x2 : SVE2p1_Store_PN_X2_Intrinsic;
def int_aarch64_sve_st1_pn_x4 : SVE2p1_Store_PN_X4_Intrinsic;
def int_aarch64_sve_stnt1_pn_x2 : SVE2p1_Store_PN_X2_Intrinsic;
def int_aarch64_sve_stnt1_pn_x4 : SVE2p1_Store_PN_X4_Intrinsic;
}

//
// SVE2 - Contiguous conflict detection
//

def int_aarch64_sve_whilerw_b : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilerw_h : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilerw_s : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilerw_d : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilewr_b : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilewr_h : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilewr_s : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_whilewr_d : SVE2_CONFLICT_DETECT_Intrinsic<[IntrSpeculatable]>;

// Scalable Matrix Extension (SME) Intrinsics
let TargetPrefix = "aarch64" in {
  class SME_Load_Store_Intrinsic<LLVMType pred_ty>
    : DefaultAttrsIntrinsic<[],
````
- **L2913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_aarch64_svcount_ty, llvm_anyptr_ty ],`.
  **L2913 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_aarch64_svcount_ty, llvm_anyptr_ty ],`。
- **L2914 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L2914 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L2915 EN**: Blank line separating nearby declarations or logic blocks.
  **L2915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2916 EN**: Declares class `SVE2p1_Store_PN_X4_Intrinsic`.
  **L2916 CN**: 声明 class `SVE2p1_Store_PN_X4_Intrinsic`。
- **L2917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [ llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L2917 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [ llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L2918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L2918 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L2919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_aarch64_svcount_ty, llvm_anyptr_ty],`.
  **L2919 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_aarch64_svcount_ty, llvm_anyptr_ty],`。
- **L2920 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L2920 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L2921 EN**: Blank line separating nearby declarations or logic blocks.
  **L2921 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2922 EN**: Declares TableGen def `int_aarch64_sve_st1_pn_x2`.
  **L2922 CN**: 声明 TableGen def `int_aarch64_sve_st1_pn_x2`。
- **L2923 EN**: Declares TableGen def `int_aarch64_sve_st1_pn_x4`.
  **L2923 CN**: 声明 TableGen def `int_aarch64_sve_st1_pn_x4`。
- **L2924 EN**: Declares TableGen def `int_aarch64_sve_stnt1_pn_x2`.
  **L2924 CN**: 声明 TableGen def `int_aarch64_sve_stnt1_pn_x2`。
- **L2925 EN**: Declares TableGen def `int_aarch64_sve_stnt1_pn_x4`.
  **L2925 CN**: 声明 TableGen def `int_aarch64_sve_stnt1_pn_x4`。
- **L2926 EN**: Closes the current lexical scope or compound statement.
  **L2926 CN**: 结束当前词法作用域或复合语句块。
- **L2927 EN**: Blank line separating nearby declarations or logic blocks.
  **L2927 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2928 EN**: Separator comment used for visual grouping.
  **L2928 CN**: 用于视觉分组的分隔注释。
- **L2929 EN**: Comment explains nearby logic, invariants, or intent: `SVE2 - Contiguous conflict detection`.
  **L2929 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2 - Contiguous conflict detection`。
- **L2930 EN**: Separator comment used for visual grouping.
  **L2930 CN**: 用于视觉分组的分隔注释。
- **L2931 EN**: Blank line separating nearby declarations or logic blocks.
  **L2931 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2932 EN**: Declares TableGen def `int_aarch64_sve_whilerw_b`.
  **L2932 CN**: 声明 TableGen def `int_aarch64_sve_whilerw_b`。
- **L2933 EN**: Declares TableGen def `int_aarch64_sve_whilerw_h`.
  **L2933 CN**: 声明 TableGen def `int_aarch64_sve_whilerw_h`。
- **L2934 EN**: Declares TableGen def `int_aarch64_sve_whilerw_s`.
  **L2934 CN**: 声明 TableGen def `int_aarch64_sve_whilerw_s`。
- **L2935 EN**: Declares TableGen def `int_aarch64_sve_whilerw_d`.
  **L2935 CN**: 声明 TableGen def `int_aarch64_sve_whilerw_d`。
- **L2936 EN**: Declares TableGen def `int_aarch64_sve_whilewr_b`.
  **L2936 CN**: 声明 TableGen def `int_aarch64_sve_whilewr_b`。
- **L2937 EN**: Declares TableGen def `int_aarch64_sve_whilewr_h`.
  **L2937 CN**: 声明 TableGen def `int_aarch64_sve_whilewr_h`。
- **L2938 EN**: Declares TableGen def `int_aarch64_sve_whilewr_s`.
  **L2938 CN**: 声明 TableGen def `int_aarch64_sve_whilewr_s`。
- **L2939 EN**: Declares TableGen def `int_aarch64_sve_whilewr_d`.
  **L2939 CN**: 声明 TableGen def `int_aarch64_sve_whilewr_d`。
- **L2940 EN**: Blank line separating nearby declarations or logic blocks.
  **L2940 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2941 EN**: Comment explains nearby logic, invariants, or intent: `Scalable Matrix Extension (SME) Intrinsics`.
  **L2941 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalable Matrix Extension (SME) Intrinsics`。
- **L2942 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2942 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2943 EN**: Declares class `SME_Load_Store_Intrinsic<LLVMType`.
  **L2943 CN**: 声明 class `SME_Load_Store_Intrinsic<LLVMType`。
- **L2944 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L2944 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。

### Lines 2945-2976

````tablegen
        [pred_ty, llvm_anyptr_ty, llvm_i32_ty, llvm_i32_ty], [IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<2>>]>;

  // Loads
  def int_aarch64_sme_ld1b_horiz : SME_Load_Store_Intrinsic<llvm_nxv16i1_ty>;
  def int_aarch64_sme_ld1h_horiz : SME_Load_Store_Intrinsic<llvm_nxv8i1_ty>;
  def int_aarch64_sme_ld1w_horiz : SME_Load_Store_Intrinsic<llvm_nxv4i1_ty>;
  def int_aarch64_sme_ld1d_horiz : SME_Load_Store_Intrinsic<llvm_nxv2i1_ty>;
  def int_aarch64_sme_ld1q_horiz : SME_Load_Store_Intrinsic<llvm_nxv1i1_ty>;
  def int_aarch64_sme_ld1b_vert  : SME_Load_Store_Intrinsic<llvm_nxv16i1_ty>;
  def int_aarch64_sme_ld1h_vert  : SME_Load_Store_Intrinsic<llvm_nxv8i1_ty>;
  def int_aarch64_sme_ld1w_vert  : SME_Load_Store_Intrinsic<llvm_nxv4i1_ty>;
  def int_aarch64_sme_ld1d_vert  : SME_Load_Store_Intrinsic<llvm_nxv2i1_ty>;
  def int_aarch64_sme_ld1q_vert  : SME_Load_Store_Intrinsic<llvm_nxv1i1_ty>;

  // Stores
  def int_aarch64_sme_st1b_horiz : SME_Load_Store_Intrinsic<llvm_nxv16i1_ty>;
  def int_aarch64_sme_st1h_horiz : SME_Load_Store_Intrinsic<llvm_nxv8i1_ty>;
  def int_aarch64_sme_st1w_horiz : SME_Load_Store_Intrinsic<llvm_nxv4i1_ty>;
  def int_aarch64_sme_st1d_horiz : SME_Load_Store_Intrinsic<llvm_nxv2i1_ty>;
  def int_aarch64_sme_st1q_horiz : SME_Load_Store_Intrinsic<llvm_nxv1i1_ty>;
  def int_aarch64_sme_st1b_vert  : SME_Load_Store_Intrinsic<llvm_nxv16i1_ty>;
  def int_aarch64_sme_st1h_vert  : SME_Load_Store_Intrinsic<llvm_nxv8i1_ty>;
  def int_aarch64_sme_st1w_vert  : SME_Load_Store_Intrinsic<llvm_nxv4i1_ty>;
  def int_aarch64_sme_st1d_vert  : SME_Load_Store_Intrinsic<llvm_nxv2i1_ty>;
  def int_aarch64_sme_st1q_vert  : SME_Load_Store_Intrinsic<llvm_nxv1i1_ty>;

  // Spill + fill
  class SME_LDR_STR_ZA_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyptr_ty, llvm_i32_ty], [IntrInaccessibleMemOrArgMemOnly]>;
  def int_aarch64_sme_ldr : SME_LDR_STR_ZA_Intrinsic;
  def int_aarch64_sme_str : SME_LDR_STR_ZA_Intrinsic;

````
- **L2945 EN**: Executes a standalone statement or declaration: `[pred_ty, llvm_anyptr_ty, llvm_i32_ty, llvm_i32_ty], [IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L2945 CN**: 执行一条独立语句或声明：`[pred_ty, llvm_anyptr_ty, llvm_i32_ty, llvm_i32_ty], [IntrInaccessibleMemOrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L2946 EN**: Blank line separating nearby declarations or logic blocks.
  **L2946 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2947 EN**: Comment explains nearby logic, invariants, or intent: `Loads`.
  **L2947 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Loads`。
- **L2948 EN**: Declares TableGen def `int_aarch64_sme_ld1b_horiz`.
  **L2948 CN**: 声明 TableGen def `int_aarch64_sme_ld1b_horiz`。
- **L2949 EN**: Declares TableGen def `int_aarch64_sme_ld1h_horiz`.
  **L2949 CN**: 声明 TableGen def `int_aarch64_sme_ld1h_horiz`。
- **L2950 EN**: Declares TableGen def `int_aarch64_sme_ld1w_horiz`.
  **L2950 CN**: 声明 TableGen def `int_aarch64_sme_ld1w_horiz`。
- **L2951 EN**: Declares TableGen def `int_aarch64_sme_ld1d_horiz`.
  **L2951 CN**: 声明 TableGen def `int_aarch64_sme_ld1d_horiz`。
- **L2952 EN**: Declares TableGen def `int_aarch64_sme_ld1q_horiz`.
  **L2952 CN**: 声明 TableGen def `int_aarch64_sme_ld1q_horiz`。
- **L2953 EN**: Declares TableGen def `int_aarch64_sme_ld1b_vert`.
  **L2953 CN**: 声明 TableGen def `int_aarch64_sme_ld1b_vert`。
- **L2954 EN**: Declares TableGen def `int_aarch64_sme_ld1h_vert`.
  **L2954 CN**: 声明 TableGen def `int_aarch64_sme_ld1h_vert`。
- **L2955 EN**: Declares TableGen def `int_aarch64_sme_ld1w_vert`.
  **L2955 CN**: 声明 TableGen def `int_aarch64_sme_ld1w_vert`。
- **L2956 EN**: Declares TableGen def `int_aarch64_sme_ld1d_vert`.
  **L2956 CN**: 声明 TableGen def `int_aarch64_sme_ld1d_vert`。
- **L2957 EN**: Declares TableGen def `int_aarch64_sme_ld1q_vert`.
  **L2957 CN**: 声明 TableGen def `int_aarch64_sme_ld1q_vert`。
- **L2958 EN**: Blank line separating nearby declarations or logic blocks.
  **L2958 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2959 EN**: Comment explains nearby logic, invariants, or intent: `Stores`.
  **L2959 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Stores`。
- **L2960 EN**: Declares TableGen def `int_aarch64_sme_st1b_horiz`.
  **L2960 CN**: 声明 TableGen def `int_aarch64_sme_st1b_horiz`。
- **L2961 EN**: Declares TableGen def `int_aarch64_sme_st1h_horiz`.
  **L2961 CN**: 声明 TableGen def `int_aarch64_sme_st1h_horiz`。
- **L2962 EN**: Declares TableGen def `int_aarch64_sme_st1w_horiz`.
  **L2962 CN**: 声明 TableGen def `int_aarch64_sme_st1w_horiz`。
- **L2963 EN**: Declares TableGen def `int_aarch64_sme_st1d_horiz`.
  **L2963 CN**: 声明 TableGen def `int_aarch64_sme_st1d_horiz`。
- **L2964 EN**: Declares TableGen def `int_aarch64_sme_st1q_horiz`.
  **L2964 CN**: 声明 TableGen def `int_aarch64_sme_st1q_horiz`。
- **L2965 EN**: Declares TableGen def `int_aarch64_sme_st1b_vert`.
  **L2965 CN**: 声明 TableGen def `int_aarch64_sme_st1b_vert`。
- **L2966 EN**: Declares TableGen def `int_aarch64_sme_st1h_vert`.
  **L2966 CN**: 声明 TableGen def `int_aarch64_sme_st1h_vert`。
- **L2967 EN**: Declares TableGen def `int_aarch64_sme_st1w_vert`.
  **L2967 CN**: 声明 TableGen def `int_aarch64_sme_st1w_vert`。
- **L2968 EN**: Declares TableGen def `int_aarch64_sme_st1d_vert`.
  **L2968 CN**: 声明 TableGen def `int_aarch64_sme_st1d_vert`。
- **L2969 EN**: Declares TableGen def `int_aarch64_sme_st1q_vert`.
  **L2969 CN**: 声明 TableGen def `int_aarch64_sme_st1q_vert`。
- **L2970 EN**: Blank line separating nearby declarations or logic blocks.
  **L2970 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2971 EN**: Comment explains nearby logic, invariants, or intent: `Spill + fill`.
  **L2971 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Spill + fill`。
- **L2972 EN**: Declares class `SME_LDR_STR_ZA_Intrinsic`.
  **L2972 CN**: 声明 class `SME_LDR_STR_ZA_Intrinsic`。
- **L2973 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyptr_ty, llvm_i32_ty], [IntrInaccessibleMemOrArgMemOnly]>;`.
  **L2973 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyptr_ty, llvm_i32_ty], [IntrInaccessibleMemOrArgMemOnly]>;`。
- **L2974 EN**: Declares TableGen def `int_aarch64_sme_ldr`.
  **L2974 CN**: 声明 TableGen def `int_aarch64_sme_ldr`。
- **L2975 EN**: Declares TableGen def `int_aarch64_sme_str`.
  **L2975 CN**: 声明 TableGen def `int_aarch64_sme_str`。
- **L2976 EN**: Blank line separating nearby declarations or logic blocks.
  **L2976 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2977-3008

````tablegen
  class SME_TileToVector_Intrinsic
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
          [LLVMMatchType<0>, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
           llvm_i32_ty, llvm_i32_ty], [IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<2>>]>;
  class SME_VectorToTile_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty, llvm_i32_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
           llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

  def int_aarch64_sme_read_horiz  : SME_TileToVector_Intrinsic;
  def int_aarch64_sme_read_vert   : SME_TileToVector_Intrinsic;
  def int_aarch64_sme_write_horiz : SME_VectorToTile_Intrinsic;
  def int_aarch64_sme_write_vert  : SME_VectorToTile_Intrinsic;

  def int_aarch64_sme_readq_horiz  : SME_TileToVector_Intrinsic;
  def int_aarch64_sme_readq_vert   : SME_TileToVector_Intrinsic;
  def int_aarch64_sme_writeq_horiz : SME_VectorToTile_Intrinsic;
  def int_aarch64_sme_writeq_vert  : SME_VectorToTile_Intrinsic;

  class SME_MOVAZ_TileToVector_X2_Intrinsic
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
          [llvm_i32_ty, llvm_i32_ty],
          [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

  class SME_MOVAZ_TileToVector_X4_Intrinsic
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
           LLVMMatchType<0>,LLVMMatchType<0>],
          [llvm_i32_ty, llvm_i32_ty],
          [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

  def int_aarch64_sme_readz_horiz_x2 : SME_MOVAZ_TileToVector_X2_Intrinsic;
  def int_aarch64_sme_readz_vert_x2  : SME_MOVAZ_TileToVector_X2_Intrinsic;
````
- **L2977 EN**: Declares class `SME_TileToVector_Intrinsic`.
  **L2977 CN**: 声明 class `SME_TileToVector_Intrinsic`。
- **L2978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L2978 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L2979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L2979 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L2980 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty], [IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L2980 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty], [IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L2981 EN**: Declares class `SME_VectorToTile_Intrinsic`.
  **L2981 CN**: 声明 class `SME_VectorToTile_Intrinsic`。
- **L2982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L2982 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L2983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L2983 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L2984 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L2984 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L2985 EN**: Blank line separating nearby declarations or logic blocks.
  **L2985 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2986 EN**: Declares TableGen def `int_aarch64_sme_read_horiz`.
  **L2986 CN**: 声明 TableGen def `int_aarch64_sme_read_horiz`。
- **L2987 EN**: Declares TableGen def `int_aarch64_sme_read_vert`.
  **L2987 CN**: 声明 TableGen def `int_aarch64_sme_read_vert`。
- **L2988 EN**: Declares TableGen def `int_aarch64_sme_write_horiz`.
  **L2988 CN**: 声明 TableGen def `int_aarch64_sme_write_horiz`。
- **L2989 EN**: Declares TableGen def `int_aarch64_sme_write_vert`.
  **L2989 CN**: 声明 TableGen def `int_aarch64_sme_write_vert`。
- **L2990 EN**: Blank line separating nearby declarations or logic blocks.
  **L2990 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2991 EN**: Declares TableGen def `int_aarch64_sme_readq_horiz`.
  **L2991 CN**: 声明 TableGen def `int_aarch64_sme_readq_horiz`。
- **L2992 EN**: Declares TableGen def `int_aarch64_sme_readq_vert`.
  **L2992 CN**: 声明 TableGen def `int_aarch64_sme_readq_vert`。
- **L2993 EN**: Declares TableGen def `int_aarch64_sme_writeq_horiz`.
  **L2993 CN**: 声明 TableGen def `int_aarch64_sme_writeq_horiz`。
- **L2994 EN**: Declares TableGen def `int_aarch64_sme_writeq_vert`.
  **L2994 CN**: 声明 TableGen def `int_aarch64_sme_writeq_vert`。
- **L2995 EN**: Blank line separating nearby declarations or logic blocks.
  **L2995 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2996 EN**: Declares class `SME_MOVAZ_TileToVector_X2_Intrinsic`.
  **L2996 CN**: 声明 class `SME_MOVAZ_TileToVector_X2_Intrinsic`。
- **L2997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L2997 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L2998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L2998 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L2999 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L2999 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L3000 EN**: Blank line separating nearby declarations or logic blocks.
  **L3000 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3001 EN**: Declares class `SME_MOVAZ_TileToVector_X4_Intrinsic`.
  **L3001 CN**: 声明 class `SME_MOVAZ_TileToVector_X4_Intrinsic`。
- **L3002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3002 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,LLVMMatchType<0>],`.
  **L3003 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,LLVMMatchType<0>],`。
- **L3004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3004 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3005 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L3005 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L3006 EN**: Blank line separating nearby declarations or logic blocks.
  **L3006 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3007 EN**: Declares TableGen def `int_aarch64_sme_readz_horiz_x2`.
  **L3007 CN**: 声明 TableGen def `int_aarch64_sme_readz_horiz_x2`。
- **L3008 EN**: Declares TableGen def `int_aarch64_sme_readz_vert_x2`.
  **L3008 CN**: 声明 TableGen def `int_aarch64_sme_readz_vert_x2`。

### Lines 3009-3040

````tablegen

  def int_aarch64_sme_readz_horiz_x4 : SME_MOVAZ_TileToVector_X4_Intrinsic;
  def int_aarch64_sme_readz_vert_x4  : SME_MOVAZ_TileToVector_X4_Intrinsic;

  class SME_MOVAZ_TileToVector_Intrinsic
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
          [llvm_i32_ty, llvm_i32_ty],
          [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

  def int_aarch64_sme_readz_horiz : SME_MOVAZ_TileToVector_Intrinsic;
  def int_aarch64_sme_readz_vert  : SME_MOVAZ_TileToVector_Intrinsic;

  def int_aarch64_sme_readz_q_horiz : SME_MOVAZ_TileToVector_Intrinsic;
  def int_aarch64_sme_readz_q_vert  : SME_MOVAZ_TileToVector_Intrinsic;

  def int_aarch64_sme_readz_x2
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
          [llvm_i32_ty],
          [IntrInaccessibleMemOnly]>;

  def int_aarch64_sme_readz_x4
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
          [llvm_i32_ty],
          [IntrInaccessibleMemOnly]>;

  def int_aarch64_sme_write_lane_zt
       :  DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyvector_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly]>;

  def int_aarch64_sme_write_zt
       :  DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyvector_ty],
            [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrWriteMem]>;
````
- **L3009 EN**: Blank line separating nearby declarations or logic blocks.
  **L3009 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3010 EN**: Declares TableGen def `int_aarch64_sme_readz_horiz_x4`.
  **L3010 CN**: 声明 TableGen def `int_aarch64_sme_readz_horiz_x4`。
- **L3011 EN**: Declares TableGen def `int_aarch64_sme_readz_vert_x4`.
  **L3011 CN**: 声明 TableGen def `int_aarch64_sme_readz_vert_x4`。
- **L3012 EN**: Blank line separating nearby declarations or logic blocks.
  **L3012 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3013 EN**: Declares class `SME_MOVAZ_TileToVector_Intrinsic`.
  **L3013 CN**: 声明 class `SME_MOVAZ_TileToVector_Intrinsic`。
- **L3014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L3014 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L3015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3015 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3016 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L3016 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L3017 EN**: Blank line separating nearby declarations or logic blocks.
  **L3017 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3018 EN**: Declares TableGen def `int_aarch64_sme_readz_horiz`.
  **L3018 CN**: 声明 TableGen def `int_aarch64_sme_readz_horiz`。
- **L3019 EN**: Declares TableGen def `int_aarch64_sme_readz_vert`.
  **L3019 CN**: 声明 TableGen def `int_aarch64_sme_readz_vert`。
- **L3020 EN**: Blank line separating nearby declarations or logic blocks.
  **L3020 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3021 EN**: Declares TableGen def `int_aarch64_sme_readz_q_horiz`.
  **L3021 CN**: 声明 TableGen def `int_aarch64_sme_readz_q_horiz`。
- **L3022 EN**: Declares TableGen def `int_aarch64_sme_readz_q_vert`.
  **L3022 CN**: 声明 TableGen def `int_aarch64_sme_readz_q_vert`。
- **L3023 EN**: Blank line separating nearby declarations or logic blocks.
  **L3023 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3024 EN**: Declares TableGen def `int_aarch64_sme_readz_x2`.
  **L3024 CN**: 声明 TableGen def `int_aarch64_sme_readz_x2`。
- **L3025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3025 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L3026 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L3027 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3027 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3028 EN**: Blank line separating nearby declarations or logic blocks.
  **L3028 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3029 EN**: Declares TableGen def `int_aarch64_sme_readz_x4`.
  **L3029 CN**: 声明 TableGen def `int_aarch64_sme_readz_x4`。
- **L3030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3030 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L3031 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L3032 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3032 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3033 EN**: Blank line separating nearby declarations or logic blocks.
  **L3033 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3034 EN**: Declares TableGen def `int_aarch64_sme_write_lane_zt`.
  **L3034 CN**: 声明 TableGen def `int_aarch64_sme_write_lane_zt`。
- **L3035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `:  DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyvector_ty, llvm_i32_ty],`.
  **L3035 CN**: 继续一个多行参数列表、初始化器或聚合项：`:  DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyvector_ty, llvm_i32_ty],`。
- **L3036 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly]>;`.
  **L3036 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly]>;`。
- **L3037 EN**: Blank line separating nearby declarations or logic blocks.
  **L3037 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3038 EN**: Declares TableGen def `int_aarch64_sme_write_zt`.
  **L3038 CN**: 声明 TableGen def `int_aarch64_sme_write_zt`。
- **L3039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `:  DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyvector_ty],`.
  **L3039 CN**: 继续一个多行参数列表、初始化器或聚合项：`:  DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyvector_ty],`。
- **L3040 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrWriteMem]>;`.
  **L3040 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrWriteMem]>;`。

### Lines 3041-3072

````tablegen


  def int_aarch64_sme_zero : DefaultAttrsIntrinsic<[], [llvm_i32_ty], [IntrWriteMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;
  def int_aarch64_sme_in_streaming_mode : DefaultAttrsIntrinsic<[llvm_i1_ty], [], [IntrNoMem]>, ClangBuiltin<"__builtin_arm_in_streaming_mode">;

  class SME_OuterProduct_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
           LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
           LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
           LLVMMatchType<0>,
           llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

  def int_aarch64_sme_mopa : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_mops : SME_OuterProduct_Intrinsic;

  def int_aarch64_sme_mopa_wide : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_mops_wide : SME_OuterProduct_Intrinsic;

  def int_aarch64_sme_smopa_wide  : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_smops_wide  : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_umopa_wide  : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_umops_wide  : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_sumopa_wide : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_sumops_wide : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_usmopa_wide : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_usmops_wide : SME_OuterProduct_Intrinsic;

  class SME_OuterProduct_QuarterTile_Single_Single
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_anyvector_ty,
````
- **L3041 EN**: Blank line separating nearby declarations or logic blocks.
  **L3041 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3042 EN**: Blank line separating nearby declarations or logic blocks.
  **L3042 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3043 EN**: Declares TableGen def `int_aarch64_sme_zero`.
  **L3043 CN**: 声明 TableGen def `int_aarch64_sme_zero`。
- **L3044 EN**: Declares TableGen def `int_aarch64_sme_in_streaming_mode`.
  **L3044 CN**: 声明 TableGen def `int_aarch64_sme_in_streaming_mode`。
- **L3045 EN**: Blank line separating nearby declarations or logic blocks.
  **L3045 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3046 EN**: Declares class `SME_OuterProduct_Intrinsic`.
  **L3046 CN**: 声明 class `SME_OuterProduct_Intrinsic`。
- **L3047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3047 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3048 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L3049 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L3050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L3050 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L3051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L3051 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L3052 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L3052 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L3053 EN**: Blank line separating nearby declarations or logic blocks.
  **L3053 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3054 EN**: Declares TableGen def `int_aarch64_sme_mopa`.
  **L3054 CN**: 声明 TableGen def `int_aarch64_sme_mopa`。
- **L3055 EN**: Declares TableGen def `int_aarch64_sme_mops`.
  **L3055 CN**: 声明 TableGen def `int_aarch64_sme_mops`。
- **L3056 EN**: Blank line separating nearby declarations or logic blocks.
  **L3056 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3057 EN**: Declares TableGen def `int_aarch64_sme_mopa_wide`.
  **L3057 CN**: 声明 TableGen def `int_aarch64_sme_mopa_wide`。
- **L3058 EN**: Declares TableGen def `int_aarch64_sme_mops_wide`.
  **L3058 CN**: 声明 TableGen def `int_aarch64_sme_mops_wide`。
- **L3059 EN**: Blank line separating nearby declarations or logic blocks.
  **L3059 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3060 EN**: Declares TableGen def `int_aarch64_sme_smopa_wide`.
  **L3060 CN**: 声明 TableGen def `int_aarch64_sme_smopa_wide`。
- **L3061 EN**: Declares TableGen def `int_aarch64_sme_smops_wide`.
  **L3061 CN**: 声明 TableGen def `int_aarch64_sme_smops_wide`。
- **L3062 EN**: Declares TableGen def `int_aarch64_sme_umopa_wide`.
  **L3062 CN**: 声明 TableGen def `int_aarch64_sme_umopa_wide`。
- **L3063 EN**: Declares TableGen def `int_aarch64_sme_umops_wide`.
  **L3063 CN**: 声明 TableGen def `int_aarch64_sme_umops_wide`。
- **L3064 EN**: Declares TableGen def `int_aarch64_sme_sumopa_wide`.
  **L3064 CN**: 声明 TableGen def `int_aarch64_sme_sumopa_wide`。
- **L3065 EN**: Declares TableGen def `int_aarch64_sme_sumops_wide`.
  **L3065 CN**: 声明 TableGen def `int_aarch64_sme_sumops_wide`。
- **L3066 EN**: Declares TableGen def `int_aarch64_sme_usmopa_wide`.
  **L3066 CN**: 声明 TableGen def `int_aarch64_sme_usmopa_wide`。
- **L3067 EN**: Declares TableGen def `int_aarch64_sme_usmops_wide`.
  **L3067 CN**: 声明 TableGen def `int_aarch64_sme_usmops_wide`。
- **L3068 EN**: Blank line separating nearby declarations or logic blocks.
  **L3068 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3069 EN**: Declares class `SME_OuterProduct_QuarterTile_Single_Single`.
  **L3069 CN**: 声明 class `SME_OuterProduct_QuarterTile_Single_Single`。
- **L3070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3070 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3071 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L3072 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。

### Lines 3073-3104

````tablegen
          LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;

  class SME_OuterProduct_QuarterTile_Single_Multi
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_anyvector_ty,
          LLVMMatchType<0>,
          LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;

  class SME_OuterProduct_QuarterTile_Multi_Multi
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_anyvector_ty,
          LLVMMatchType<0>,
          LLVMMatchType<0>,
          LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;

  // 2-way and 4-way multi-vector signed/unsigned Quarter Tile Quarter Product A/S
  foreach mode = ["s", "a"] in {
    foreach za = ["", "_za64"] in {
      foreach ty = ["s", "u", "su", "us"] in {
        def int_aarch64_sme_ # ty # "mop4" # mode # za # "_wide_1x1" : SME_OuterProduct_QuarterTile_Single_Single;
        def int_aarch64_sme_ # ty # "mop4" # mode # za # "_wide_1x2" : SME_OuterProduct_QuarterTile_Single_Multi;
        def int_aarch64_sme_ # ty # "mop4" # mode # za # "_wide_2x1" : SME_OuterProduct_QuarterTile_Single_Multi;
        def int_aarch64_sme_ # ty # "mop4" # mode # za # "_wide_2x2" : SME_OuterProduct_QuarterTile_Multi_Multi;
      }
    }
  }

  // 2-way and 4-way multi-vector floating point Quarter Tile Quarter Product A/S
  foreach mode = ["s", "a"] in {
    foreach wide = ["", "_wide"] in {
````
- **L3073 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;`.
  **L3073 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;`。
- **L3074 EN**: Blank line separating nearby declarations or logic blocks.
  **L3074 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3075 EN**: Declares class `SME_OuterProduct_QuarterTile_Single_Multi`.
  **L3075 CN**: 声明 class `SME_OuterProduct_QuarterTile_Single_Multi`。
- **L3076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3076 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3077 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L3078 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L3079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L3079 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L3080 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;`.
  **L3080 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;`。
- **L3081 EN**: Blank line separating nearby declarations or logic blocks.
  **L3081 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3082 EN**: Declares class `SME_OuterProduct_QuarterTile_Multi_Multi`.
  **L3082 CN**: 声明 class `SME_OuterProduct_QuarterTile_Multi_Multi`。
- **L3083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3083 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3084 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L3085 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L3086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L3086 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L3087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L3087 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L3088 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;`.
  **L3088 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrHasSideEffects]>;`。
- **L3089 EN**: Blank line separating nearby declarations or logic blocks.
  **L3089 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3090 EN**: Comment explains nearby logic, invariants, or intent: `2-way and 4-way multi-vector signed/unsigned Quarter Tile Quarter Product A/S`.
  **L3090 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2-way and 4-way multi-vector signed/unsigned Quarter Tile Quarter Product A/S`。
- **L3091 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3091 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3092 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3092 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3093 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3093 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3094 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3094 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3095 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3095 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3096 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3096 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3097 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3097 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3098 EN**: Closes the current lexical scope or compound statement.
  **L3098 CN**: 结束当前词法作用域或复合语句块。
- **L3099 EN**: Closes the current lexical scope or compound statement.
  **L3099 CN**: 结束当前词法作用域或复合语句块。
- **L3100 EN**: Closes the current lexical scope or compound statement.
  **L3100 CN**: 结束当前词法作用域或复合语句块。
- **L3101 EN**: Blank line separating nearby declarations or logic blocks.
  **L3101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3102 EN**: Comment explains nearby logic, invariants, or intent: `2-way and 4-way multi-vector floating point Quarter Tile Quarter Product A/S`.
  **L3102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2-way and 4-way multi-vector floating point Quarter Tile Quarter Product A/S`。
- **L3103 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3103 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3104 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3104 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 3105-3136

````tablegen
      def int_aarch64_sme_mop4 # mode # wide # "_1x1" : SME_OuterProduct_QuarterTile_Single_Single;
      def int_aarch64_sme_mop4 # mode # wide # "_1x2" : SME_OuterProduct_QuarterTile_Single_Multi;
      def int_aarch64_sme_mop4 # mode # wide # "_2x1" : SME_OuterProduct_QuarterTile_Single_Multi;
      def int_aarch64_sme_mop4 # mode # wide # "_2x2" : SME_OuterProduct_QuarterTile_Multi_Multi;
    }
  }

  class SME_OuterProduct_TMOP_Intrinsic
     : DefaultAttrsIntrinsic<[],
         [llvm_i32_ty,
          llvm_anyvector_ty,
          LLVMMatchType<0>,
          LLVMMatchType<0>,
          llvm_nxv16i8_ty,
          llvm_i32_ty],
         [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<5>>,
          IntrInaccessibleMemOnly]>;

  def int_aarch64_sme_ftmopa_za16 : SME_OuterProduct_TMOP_Intrinsic;
  def int_aarch64_sme_ftmopa_za32 : SME_OuterProduct_TMOP_Intrinsic;
  def int_aarch64_sme_stmopa_za32 : SME_OuterProduct_TMOP_Intrinsic;
  def int_aarch64_sme_utmopa_za32 : SME_OuterProduct_TMOP_Intrinsic;
  def int_aarch64_sme_sutmopa_za32 : SME_OuterProduct_TMOP_Intrinsic;
  def int_aarch64_sme_ustmopa_za32 : SME_OuterProduct_TMOP_Intrinsic;

  // 16 and 32 bit multi-vector floating point 8 Quarter Tile Quarter Product
  foreach za = ["za16", "za32"] in {
    def int_aarch64_sme_fp8_fmop4a_ # za # "_1x1" : SME_OuterProduct_QuarterTile_Single_Single;
    def int_aarch64_sme_fp8_fmop4a_ # za # "_1x2" : SME_OuterProduct_QuarterTile_Single_Multi;
    def int_aarch64_sme_fp8_fmop4a_ # za # "_2x1" : SME_OuterProduct_QuarterTile_Single_Multi;
    def int_aarch64_sme_fp8_fmop4a_ # za # "_2x2" : SME_OuterProduct_QuarterTile_Multi_Multi;
  }
````
- **L3105 EN**: Declares TableGen def `int_aarch64_sme_mop4`.
  **L3105 CN**: 声明 TableGen def `int_aarch64_sme_mop4`。
- **L3106 EN**: Declares TableGen def `int_aarch64_sme_mop4`.
  **L3106 CN**: 声明 TableGen def `int_aarch64_sme_mop4`。
- **L3107 EN**: Declares TableGen def `int_aarch64_sme_mop4`.
  **L3107 CN**: 声明 TableGen def `int_aarch64_sme_mop4`。
- **L3108 EN**: Declares TableGen def `int_aarch64_sme_mop4`.
  **L3108 CN**: 声明 TableGen def `int_aarch64_sme_mop4`。
- **L3109 EN**: Closes the current lexical scope or compound statement.
  **L3109 CN**: 结束当前词法作用域或复合语句块。
- **L3110 EN**: Closes the current lexical scope or compound statement.
  **L3110 CN**: 结束当前词法作用域或复合语句块。
- **L3111 EN**: Blank line separating nearby declarations or logic blocks.
  **L3111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3112 EN**: Declares class `SME_OuterProduct_TMOP_Intrinsic`.
  **L3112 CN**: 声明 class `SME_OuterProduct_TMOP_Intrinsic`。
- **L3113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3113 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3114 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L3115 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L3116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L3116 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L3117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,`.
  **L3117 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,`。
- **L3118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L3118 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L3119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L3119 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L3120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<5>>,`.
  **L3120 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<5>>,`。
- **L3121 EN**: Executes a standalone statement or declaration: `IntrInaccessibleMemOnly]>;`.
  **L3121 CN**: 执行一条独立语句或声明：`IntrInaccessibleMemOnly]>;`。
- **L3122 EN**: Blank line separating nearby declarations or logic blocks.
  **L3122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3123 EN**: Declares TableGen def `int_aarch64_sme_ftmopa_za16`.
  **L3123 CN**: 声明 TableGen def `int_aarch64_sme_ftmopa_za16`。
- **L3124 EN**: Declares TableGen def `int_aarch64_sme_ftmopa_za32`.
  **L3124 CN**: 声明 TableGen def `int_aarch64_sme_ftmopa_za32`。
- **L3125 EN**: Declares TableGen def `int_aarch64_sme_stmopa_za32`.
  **L3125 CN**: 声明 TableGen def `int_aarch64_sme_stmopa_za32`。
- **L3126 EN**: Declares TableGen def `int_aarch64_sme_utmopa_za32`.
  **L3126 CN**: 声明 TableGen def `int_aarch64_sme_utmopa_za32`。
- **L3127 EN**: Declares TableGen def `int_aarch64_sme_sutmopa_za32`.
  **L3127 CN**: 声明 TableGen def `int_aarch64_sme_sutmopa_za32`。
- **L3128 EN**: Declares TableGen def `int_aarch64_sme_ustmopa_za32`.
  **L3128 CN**: 声明 TableGen def `int_aarch64_sme_ustmopa_za32`。
- **L3129 EN**: Blank line separating nearby declarations or logic blocks.
  **L3129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3130 EN**: Comment explains nearby logic, invariants, or intent: `16 and 32 bit multi-vector floating point 8 Quarter Tile Quarter Product`.
  **L3130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`16 and 32 bit multi-vector floating point 8 Quarter Tile Quarter Product`。
- **L3131 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3131 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3132 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmop4a_`.
  **L3132 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmop4a_`。
- **L3133 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmop4a_`.
  **L3133 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmop4a_`。
- **L3134 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmop4a_`.
  **L3134 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmop4a_`。
- **L3135 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmop4a_`.
  **L3135 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmop4a_`。
- **L3136 EN**: Closes the current lexical scope or compound statement.
  **L3136 CN**: 结束当前词法作用域或复合语句块。

### Lines 3137-3168

````tablegen

  class SME_AddVectorToTile_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
           LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
           LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,
           llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

  def int_aarch64_sme_addha : SME_AddVectorToTile_Intrinsic;
  def int_aarch64_sme_addva : SME_AddVectorToTile_Intrinsic;

  //
  // Counting elements
  //

  def int_aarch64_sme_cntsd
      : DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem]>;

  //
  // PSTATE Functions
  //

  def int_aarch64_sme_get_tpidr2
      : DefaultAttrsIntrinsic<[llvm_i64_ty], [],
                              [IntrNoMem, IntrHasSideEffects]>;
  def int_aarch64_sme_set_tpidr2
      : DefaultAttrsIntrinsic<[], [llvm_i64_ty],
                              [IntrNoMem, IntrHasSideEffects]>;

  def int_aarch64_sme_za_enable
      : DefaultAttrsIntrinsic<[], [], [IntrWriteMem, IntrInaccessibleMemOnly]>;
  def int_aarch64_sme_za_disable
````
- **L3137 EN**: Blank line separating nearby declarations or logic blocks.
  **L3137 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3138 EN**: Declares class `SME_AddVectorToTile_Intrinsic`.
  **L3138 CN**: 声明 class `SME_AddVectorToTile_Intrinsic`。
- **L3139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3139 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3140 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L3141 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L3142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`.
  **L3142 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMScalarOrSameVectorWidth<0, llvm_i1_ty>,`。
- **L3143 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L3143 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty], [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L3144 EN**: Blank line separating nearby declarations or logic blocks.
  **L3144 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3145 EN**: Declares TableGen def `int_aarch64_sme_addha`.
  **L3145 CN**: 声明 TableGen def `int_aarch64_sme_addha`。
- **L3146 EN**: Declares TableGen def `int_aarch64_sme_addva`.
  **L3146 CN**: 声明 TableGen def `int_aarch64_sme_addva`。
- **L3147 EN**: Blank line separating nearby declarations or logic blocks.
  **L3147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3148 EN**: Separator comment used for visual grouping.
  **L3148 CN**: 用于视觉分组的分隔注释。
- **L3149 EN**: Comment explains nearby logic, invariants, or intent: `Counting elements`.
  **L3149 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Counting elements`。
- **L3150 EN**: Separator comment used for visual grouping.
  **L3150 CN**: 用于视觉分组的分隔注释。
- **L3151 EN**: Blank line separating nearby declarations or logic blocks.
  **L3151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3152 EN**: Declares TableGen def `int_aarch64_sme_cntsd`.
  **L3152 CN**: 声明 TableGen def `int_aarch64_sme_cntsd`。
- **L3153 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem]>;`.
  **L3153 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [], [IntrNoMem]>;`。
- **L3154 EN**: Blank line separating nearby declarations or logic blocks.
  **L3154 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3155 EN**: Separator comment used for visual grouping.
  **L3155 CN**: 用于视觉分组的分隔注释。
- **L3156 EN**: Comment explains nearby logic, invariants, or intent: `PSTATE Functions`.
  **L3156 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PSTATE Functions`。
- **L3157 EN**: Separator comment used for visual grouping.
  **L3157 CN**: 用于视觉分组的分隔注释。
- **L3158 EN**: Blank line separating nearby declarations or logic blocks.
  **L3158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3159 EN**: Declares TableGen def `int_aarch64_sme_get_tpidr2`.
  **L3159 CN**: 声明 TableGen def `int_aarch64_sme_get_tpidr2`。
- **L3160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [],`.
  **L3160 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [],`。
- **L3161 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L3161 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L3162 EN**: Declares TableGen def `int_aarch64_sme_set_tpidr2`.
  **L3162 CN**: 声明 TableGen def `int_aarch64_sme_set_tpidr2`。
- **L3163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i64_ty],`.
  **L3163 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i64_ty],`。
- **L3164 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L3164 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L3165 EN**: Blank line separating nearby declarations or logic blocks.
  **L3165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3166 EN**: Declares TableGen def `int_aarch64_sme_za_enable`.
  **L3166 CN**: 声明 TableGen def `int_aarch64_sme_za_enable`。
- **L3167 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrWriteMem, IntrInaccessibleMemOnly]>;`.
  **L3167 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrWriteMem, IntrInaccessibleMemOnly]>;`。
- **L3168 EN**: Declares TableGen def `int_aarch64_sme_za_disable`.
  **L3168 CN**: 声明 TableGen def `int_aarch64_sme_za_disable`。

### Lines 3169-3200

````tablegen
      : DefaultAttrsIntrinsic<[], [], [IntrWriteMem, IntrInaccessibleMemOnly]>;

  // Clamp
  //

  def int_aarch64_sve_sclamp : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
  def int_aarch64_sve_uclamp : AdvSIMD_3VectorArg_Intrinsic<[IntrSpeculatable]>;
  def int_aarch64_sve_fclamp : AdvSIMD_3VectorArg_Intrinsic;


  //
  // Reversal
  //

  def int_aarch64_sve_revd : AdvSIMD_Merged1VectorArg_Intrinsic<[IntrSpeculatable]>;

  //
  // Predicate selection
  //

  def int_aarch64_sve_psel
      : DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],
                              [llvm_nxv16i1_ty,
                               llvm_anyvector_ty, llvm_i32_ty],
                              [IntrNoMem]>;

  //
  // Predicate-pair intrinsics
  //
  foreach cmp = ["ge", "gt", "hi", "hs", "le", "lo", "ls", "lt"] in {
    def int_aarch64_sve_while # cmp # _x2
        : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
````
- **L3169 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [], [IntrWriteMem, IntrInaccessibleMemOnly]>;`.
  **L3169 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [], [IntrWriteMem, IntrInaccessibleMemOnly]>;`。
- **L3170 EN**: Blank line separating nearby declarations or logic blocks.
  **L3170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3171 EN**: Comment explains nearby logic, invariants, or intent: `Clamp`.
  **L3171 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Clamp`。
- **L3172 EN**: Separator comment used for visual grouping.
  **L3172 CN**: 用于视觉分组的分隔注释。
- **L3173 EN**: Blank line separating nearby declarations or logic blocks.
  **L3173 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3174 EN**: Declares TableGen def `int_aarch64_sve_sclamp`.
  **L3174 CN**: 声明 TableGen def `int_aarch64_sve_sclamp`。
- **L3175 EN**: Declares TableGen def `int_aarch64_sve_uclamp`.
  **L3175 CN**: 声明 TableGen def `int_aarch64_sve_uclamp`。
- **L3176 EN**: Declares TableGen def `int_aarch64_sve_fclamp`.
  **L3176 CN**: 声明 TableGen def `int_aarch64_sve_fclamp`。
- **L3177 EN**: Blank line separating nearby declarations or logic blocks.
  **L3177 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3178 EN**: Blank line separating nearby declarations or logic blocks.
  **L3178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3179 EN**: Separator comment used for visual grouping.
  **L3179 CN**: 用于视觉分组的分隔注释。
- **L3180 EN**: Comment explains nearby logic, invariants, or intent: `Reversal`.
  **L3180 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reversal`。
- **L3181 EN**: Separator comment used for visual grouping.
  **L3181 CN**: 用于视觉分组的分隔注释。
- **L3182 EN**: Blank line separating nearby declarations or logic blocks.
  **L3182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3183 EN**: Declares TableGen def `int_aarch64_sve_revd`.
  **L3183 CN**: 声明 TableGen def `int_aarch64_sve_revd`。
- **L3184 EN**: Blank line separating nearby declarations or logic blocks.
  **L3184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3185 EN**: Separator comment used for visual grouping.
  **L3185 CN**: 用于视觉分组的分隔注释。
- **L3186 EN**: Comment explains nearby logic, invariants, or intent: `Predicate selection`.
  **L3186 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Predicate selection`。
- **L3187 EN**: Separator comment used for visual grouping.
  **L3187 CN**: 用于视觉分组的分隔注释。
- **L3188 EN**: Blank line separating nearby declarations or logic blocks.
  **L3188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3189 EN**: Declares TableGen def `int_aarch64_sve_psel`.
  **L3189 CN**: 声明 TableGen def `int_aarch64_sve_psel`。
- **L3190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],`.
  **L3190 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i1_ty],`。
- **L3191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i1_ty,`.
  **L3191 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i1_ty,`。
- **L3192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, llvm_i32_ty],`.
  **L3192 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, llvm_i32_ty],`。
- **L3193 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3193 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3194 EN**: Blank line separating nearby declarations or logic blocks.
  **L3194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3195 EN**: Separator comment used for visual grouping.
  **L3195 CN**: 用于视觉分组的分隔注释。
- **L3196 EN**: Comment explains nearby logic, invariants, or intent: `Predicate-pair intrinsics`.
  **L3196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Predicate-pair intrinsics`。
- **L3197 EN**: Separator comment used for visual grouping.
  **L3197 CN**: 用于视觉分组的分隔注释。
- **L3198 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3198 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3199 EN**: Declares TableGen def `int_aarch64_sve_while`.
  **L3199 CN**: 声明 TableGen def `int_aarch64_sve_while`。
- **L3200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3200 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。

### Lines 3201-3232

````tablegen
                                [llvm_i64_ty, llvm_i64_ty], [IntrNoMem]>;
  }

  //
  // Predicate-as-counter intrinsics
  //

  def int_aarch64_sve_pext
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                              [llvm_aarch64_svcount_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  def int_aarch64_sve_pext_x2
      : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                              [llvm_aarch64_svcount_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  def int_aarch64_sve_ptrue_c8
      : DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;
  def int_aarch64_sve_ptrue_c16
      : DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;
  def int_aarch64_sve_ptrue_c32
      : DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;
  def int_aarch64_sve_ptrue_c64
      : DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;

  def int_aarch64_sve_cntp_c8
      : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_aarch64_sve_cntp_c16
      : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<1>>]>;
````
- **L3201 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i64_ty], [IntrNoMem]>;`.
  **L3201 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i64_ty], [IntrNoMem]>;`。
- **L3202 EN**: Closes the current lexical scope or compound statement.
  **L3202 CN**: 结束当前词法作用域或复合语句块。
- **L3203 EN**: Blank line separating nearby declarations or logic blocks.
  **L3203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3204 EN**: Separator comment used for visual grouping.
  **L3204 CN**: 用于视觉分组的分隔注释。
- **L3205 EN**: Comment explains nearby logic, invariants, or intent: `Predicate-as-counter intrinsics`.
  **L3205 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Predicate-as-counter intrinsics`。
- **L3206 EN**: Separator comment used for visual grouping.
  **L3206 CN**: 用于视觉分组的分隔注释。
- **L3207 EN**: Blank line separating nearby declarations or logic blocks.
  **L3207 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3208 EN**: Declares TableGen def `int_aarch64_sve_pext`.
  **L3208 CN**: 声明 TableGen def `int_aarch64_sve_pext`。
- **L3209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L3209 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L3210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_aarch64_svcount_ty, llvm_i32_ty],`.
  **L3210 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_aarch64_svcount_ty, llvm_i32_ty],`。
- **L3211 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3211 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3212 EN**: Blank line separating nearby declarations or logic blocks.
  **L3212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3213 EN**: Declares TableGen def `int_aarch64_sve_pext_x2`.
  **L3213 CN**: 声明 TableGen def `int_aarch64_sve_pext_x2`。
- **L3214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3214 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_aarch64_svcount_ty, llvm_i32_ty],`.
  **L3215 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_aarch64_svcount_ty, llvm_i32_ty],`。
- **L3216 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3216 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3217 EN**: Blank line separating nearby declarations or logic blocks.
  **L3217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3218 EN**: Declares TableGen def `int_aarch64_sve_ptrue_c8`.
  **L3218 CN**: 声明 TableGen def `int_aarch64_sve_ptrue_c8`。
- **L3219 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`.
  **L3219 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`。
- **L3220 EN**: Declares TableGen def `int_aarch64_sve_ptrue_c16`.
  **L3220 CN**: 声明 TableGen def `int_aarch64_sve_ptrue_c16`。
- **L3221 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`.
  **L3221 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`。
- **L3222 EN**: Declares TableGen def `int_aarch64_sve_ptrue_c32`.
  **L3222 CN**: 声明 TableGen def `int_aarch64_sve_ptrue_c32`。
- **L3223 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`.
  **L3223 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`。
- **L3224 EN**: Declares TableGen def `int_aarch64_sve_ptrue_c64`.
  **L3224 CN**: 声明 TableGen def `int_aarch64_sve_ptrue_c64`。
- **L3225 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`.
  **L3225 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty], [], [IntrNoMem]>;`。
- **L3226 EN**: Blank line separating nearby declarations or logic blocks.
  **L3226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3227 EN**: Declares TableGen def `int_aarch64_sve_cntp_c8`.
  **L3227 CN**: 声明 TableGen def `int_aarch64_sve_cntp_c8`。
- **L3228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`.
  **L3228 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`。
- **L3229 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3229 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3230 EN**: Declares TableGen def `int_aarch64_sve_cntp_c16`.
  **L3230 CN**: 声明 TableGen def `int_aarch64_sve_cntp_c16`。
- **L3231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`.
  **L3231 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`。
- **L3232 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3232 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。

### Lines 3233-3264

````tablegen
  def int_aarch64_sve_cntp_c32
      : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_aarch64_sve_cntp_c64
      : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  // While (predicate-as-counter) intrinsics
  foreach cmp = ["ge", "gt", "hi", "hs", "le", "lo", "ls", "lt"] in {
    foreach ty = ["c8", "c16", "c32", "c64"] in {
      def int_aarch64_sve_while # cmp # _ # ty
          : DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty],
                                  [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],
                                  [IntrNoMem, ImmArg<ArgIndex<2>>]>;
    }
  }

  //
  // SME2 Intrinsics
  //

  class SME2_Matrix_ArrayVector_Single_Single_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_i32_ty,
                 llvm_anyvector_ty, LLVMMatchType<0>],
                [IntrInaccessibleMemOnly]>;

  class SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_i32_ty,
                 llvm_anyvector_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>],
````
- **L3233 EN**: Declares TableGen def `int_aarch64_sve_cntp_c32`.
  **L3233 CN**: 声明 TableGen def `int_aarch64_sve_cntp_c32`。
- **L3234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`.
  **L3234 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`。
- **L3235 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3235 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3236 EN**: Declares TableGen def `int_aarch64_sve_cntp_c64`.
  **L3236 CN**: 声明 TableGen def `int_aarch64_sve_cntp_c64`。
- **L3237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`.
  **L3237 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_aarch64_svcount_ty, llvm_i32_ty],`。
- **L3238 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L3238 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L3239 EN**: Blank line separating nearby declarations or logic blocks.
  **L3239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3240 EN**: Comment explains nearby logic, invariants, or intent: `While (predicate-as-counter) intrinsics`.
  **L3240 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`While (predicate-as-counter) intrinsics`。
- **L3241 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3241 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3242 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3242 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3243 EN**: Declares TableGen def `int_aarch64_sve_while`.
  **L3243 CN**: 声明 TableGen def `int_aarch64_sve_while`。
- **L3244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty],`.
  **L3244 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_aarch64_svcount_ty],`。
- **L3245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`.
  **L3245 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`。
- **L3246 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L3246 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L3247 EN**: Closes the current lexical scope or compound statement.
  **L3247 CN**: 结束当前词法作用域或复合语句块。
- **L3248 EN**: Closes the current lexical scope or compound statement.
  **L3248 CN**: 结束当前词法作用域或复合语句块。
- **L3249 EN**: Blank line separating nearby declarations or logic blocks.
  **L3249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3250 EN**: Separator comment used for visual grouping.
  **L3250 CN**: 用于视觉分组的分隔注释。
- **L3251 EN**: Comment explains nearby logic, invariants, or intent: `SME2 Intrinsics`.
  **L3251 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SME2 Intrinsics`。
- **L3252 EN**: Separator comment used for visual grouping.
  **L3252 CN**: 用于视觉分组的分隔注释。
- **L3253 EN**: Blank line separating nearby declarations or logic blocks.
  **L3253 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3254 EN**: Declares class `SME2_Matrix_ArrayVector_Single_Single_Intrinsic`.
  **L3254 CN**: 声明 class `SME2_Matrix_ArrayVector_Single_Single_Intrinsic`。
- **L3255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3255 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3256 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3257 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3258 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3258 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3259 EN**: Blank line separating nearby declarations or logic blocks.
  **L3259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3260 EN**: Declares class `SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic`.
  **L3260 CN**: 声明 class `SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic`。
- **L3261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3261 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3262 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3263 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L3264 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。

### Lines 3265-3296

````tablegen
                [IntrInaccessibleMemOnly]>;

  class SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_i32_ty,
                 llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>],
                [IntrInaccessibleMemOnly]>;

  class SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_i32_ty,
                 llvm_anyvector_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrInaccessibleMemOnly]>;

  class SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_i32_ty,
                 llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrInaccessibleMemOnly]>;

  class SME2_Matrix_ArrayVector_Single_Index_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_i32_ty,
                llvm_anyvector_ty,
                LLVMMatchType<0>, llvm_i32_ty],
                [IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;

  class SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic
    : DefaultAttrsIntrinsic<[],
````
- **L3265 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3265 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3266 EN**: Blank line separating nearby declarations or logic blocks.
  **L3266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3267 EN**: Declares class `SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic`.
  **L3267 CN**: 声明 class `SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic`。
- **L3268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3268 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3269 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3270 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L3271 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L3272 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3272 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3273 EN**: Blank line separating nearby declarations or logic blocks.
  **L3273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3274 EN**: Declares class `SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic`.
  **L3274 CN**: 声明 class `SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic`。
- **L3275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3275 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3276 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3277 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3278 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3279 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3279 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3280 EN**: Blank line separating nearby declarations or logic blocks.
  **L3280 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3281 EN**: Declares class `SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic`.
  **L3281 CN**: 声明 class `SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic`。
- **L3282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3282 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3283 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3284 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3285 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3286 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3286 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3287 EN**: Blank line separating nearby declarations or logic blocks.
  **L3287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3288 EN**: Declares class `SME2_Matrix_ArrayVector_Single_Index_Intrinsic`.
  **L3288 CN**: 声明 class `SME2_Matrix_ArrayVector_Single_Index_Intrinsic`。
- **L3289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3289 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3290 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty,`.
  **L3291 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty,`。
- **L3292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_i32_ty],`.
  **L3292 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_i32_ty],`。
- **L3293 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`.
  **L3293 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`。
- **L3294 EN**: Blank line separating nearby declarations or logic blocks.
  **L3294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3295 EN**: Declares class `SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic`.
  **L3295 CN**: 声明 class `SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic`。
- **L3296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3296 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。

### Lines 3297-3328

````tablegen
                [llvm_i32_ty,
                 llvm_anyvector_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>, llvm_i32_ty],
                [IntrInaccessibleMemOnly, ImmArg<ArgIndex<4>>]>;

  class SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic
    : DefaultAttrsIntrinsic<[],
                [llvm_i32_ty,
                 llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, llvm_i32_ty],
                [IntrInaccessibleMemOnly, ImmArg<ArgIndex<6>>]>;

  class SVE2_VG2_Multi_Imm_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],
                [llvm_anyvector_ty, LLVMMatchType<0>,
                 llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  class SVE2_VG4_Multi_Imm_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMSubdivide4VectorType<0>],
                [llvm_anyvector_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 llvm_i32_ty],
                [IntrNoMem, ImmArg<ArgIndex<4>>]>;

  class SME2_ZA_Write_VG2_Intrinsic
   : DefaultAttrsIntrinsic<[],
               [llvm_i32_ty,
                llvm_anyvector_ty, LLVMMatchType<0>],
               [IntrInaccessibleMemOnly]>;

  class SME2_ZA_Write_VG4_Intrinsic
````
- **L3297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3297 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3298 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_i32_ty],`.
  **L3299 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_i32_ty],`。
- **L3300 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<4>>]>;`.
  **L3300 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<4>>]>;`。
- **L3301 EN**: Blank line separating nearby declarations or logic blocks.
  **L3301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3302 EN**: Declares class `SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic`.
  **L3302 CN**: 声明 class `SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic`。
- **L3303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3303 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3304 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3305 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_i32_ty],`.
  **L3306 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_i32_ty],`。
- **L3307 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<6>>]>;`.
  **L3307 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<6>>]>;`。
- **L3308 EN**: Blank line separating nearby declarations or logic blocks.
  **L3308 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3309 EN**: Declares class `SVE2_VG2_Multi_Imm_Intrinsic`.
  **L3309 CN**: 声明 class `SVE2_VG2_Multi_Imm_Intrinsic`。
- **L3310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`.
  **L3310 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`。
- **L3311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3311 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L3312 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L3313 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L3313 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L3314 EN**: Blank line separating nearby declarations or logic blocks.
  **L3314 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3315 EN**: Declares class `SVE2_VG4_Multi_Imm_Intrinsic`.
  **L3315 CN**: 声明 class `SVE2_VG4_Multi_Imm_Intrinsic`。
- **L3316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide4VectorType<0>],`.
  **L3316 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide4VectorType<0>],`。
- **L3317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3317 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3318 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L3319 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L3320 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<4>>]>;`.
  **L3320 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<4>>]>;`。
- **L3321 EN**: Blank line separating nearby declarations or logic blocks.
  **L3321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3322 EN**: Declares class `SME2_ZA_Write_VG2_Intrinsic`.
  **L3322 CN**: 声明 class `SME2_ZA_Write_VG2_Intrinsic`。
- **L3323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3323 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3324 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3325 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3326 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3326 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3327 EN**: Blank line separating nearby declarations or logic blocks.
  **L3327 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3328 EN**: Declares class `SME2_ZA_Write_VG4_Intrinsic`.
  **L3328 CN**: 声明 class `SME2_ZA_Write_VG4_Intrinsic`。

### Lines 3329-3360

````tablegen
   : DefaultAttrsIntrinsic<[],
               [llvm_i32_ty,
                llvm_anyvector_ty, LLVMMatchType<0>,
                LLVMMatchType<0>,  LLVMMatchType<0>],
               [IntrInaccessibleMemOnly]>;

  class SVE2_VG2_Multi_Single_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>],
                [IntrNoMem]>;

  class SVE2_VG2_Multi_Single_Scale_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMVectorOfBitcastsToInt<0>],
                [IntrNoMem]>;

  class SVE2_VG4_Multi_Single_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>],
                            [LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMMatchType<0>],
                            [IntrNoMem]>;

  class SVE2_VG4_Multi_Single_Scale_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>],
                            [LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMVectorOfBitcastsToInt<0>],
````
- **L3329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3329 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3330 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3331 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3332 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3333 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L3333 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L3334 EN**: Blank line separating nearby declarations or logic blocks.
  **L3334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3335 EN**: Declares class `SVE2_VG2_Multi_Single_Intrinsic`.
  **L3335 CN**: 声明 class `SVE2_VG2_Multi_Single_Intrinsic`。
- **L3336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3336 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3337 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L3338 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L3339 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3339 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3340 EN**: Blank line separating nearby declarations or logic blocks.
  **L3340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3341 EN**: Declares class `SVE2_VG2_Multi_Single_Scale_Intrinsic`.
  **L3341 CN**: 声明 class `SVE2_VG2_Multi_Single_Scale_Intrinsic`。
- **L3342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3342 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3343 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>],`.
  **L3344 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>],`。
- **L3345 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3345 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3346 EN**: Blank line separating nearby declarations or logic blocks.
  **L3346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3347 EN**: Declares class `SVE2_VG4_Multi_Single_Intrinsic`.
  **L3347 CN**: 声明 class `SVE2_VG4_Multi_Single_Intrinsic`。
- **L3348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3348 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3349 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3350 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3351 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L3352 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L3353 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3353 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3354 EN**: Blank line separating nearby declarations or logic blocks.
  **L3354 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3355 EN**: Declares class `SVE2_VG4_Multi_Single_Scale_Intrinsic`.
  **L3355 CN**: 声明 class `SVE2_VG4_Multi_Single_Scale_Intrinsic`。
- **L3356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3356 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3357 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3358 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3359 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>],`.
  **L3360 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>],`。

### Lines 3361-3392

````tablegen
                            [IntrNoMem]>;

  class SVE2_VG2_Multi_Multi_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrNoMem]>;

  class SVE2_VG2_Multi_Multi_Scale_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],
                [IntrNoMem]>;

  class SVE2_VG4_Multi_Multi_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>],
                            [LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMMatchType<0>, LLVMMatchType<0>,
                             LLVMMatchType<0>, LLVMMatchType<0>],
                            [IntrNoMem]>;

  class SVE2_VG4_Multi_Multi_Scale_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>],
                            [LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>,
                             LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>,
                             LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],
                            [IntrNoMem]>;

````
- **L3361 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3361 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3362 EN**: Blank line separating nearby declarations or logic blocks.
  **L3362 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3363 EN**: Declares class `SVE2_VG2_Multi_Multi_Intrinsic`.
  **L3363 CN**: 声明 class `SVE2_VG2_Multi_Multi_Intrinsic`。
- **L3364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3364 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3365 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3366 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3367 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3367 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3368 EN**: Blank line separating nearby declarations or logic blocks.
  **L3368 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3369 EN**: Declares class `SVE2_VG2_Multi_Multi_Scale_Intrinsic`.
  **L3369 CN**: 声明 class `SVE2_VG2_Multi_Multi_Scale_Intrinsic`。
- **L3370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3370 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3371 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`.
  **L3372 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`。
- **L3373 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3373 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3374 EN**: Blank line separating nearby declarations or logic blocks.
  **L3374 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3375 EN**: Declares class `SVE2_VG4_Multi_Multi_Intrinsic`.
  **L3375 CN**: 声明 class `SVE2_VG4_Multi_Multi_Intrinsic`。
- **L3376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3376 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3377 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3378 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3379 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3380 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3381 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3382 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3382 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3383 EN**: Blank line separating nearby declarations or logic blocks.
  **L3383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3384 EN**: Declares class `SVE2_VG4_Multi_Multi_Scale_Intrinsic`.
  **L3384 CN**: 声明 class `SVE2_VG4_Multi_Multi_Scale_Intrinsic`。
- **L3385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3385 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3386 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3387 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>,`.
  **L3388 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>,`。
- **L3389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>,`.
  **L3389 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>,`。
- **L3390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`.
  **L3390 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`。
- **L3391 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3391 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3392 EN**: Blank line separating nearby declarations or logic blocks.
  **L3392 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3393-3424

````tablegen
  class SVE2_VG2_Sel_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [llvm_aarch64_svcount_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>], [IntrNoMem]>;

  class SVE2_VG4_Sel_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>, LLVMMatchType<0>],
                [llvm_aarch64_svcount_ty, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>], [IntrNoMem]>;

  class SVE2_CVT_VG2_SINGLE_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],
                            [llvm_anyvector_ty, LLVMMatchType<0>],
                            [IntrNoMem]>;

  class SVE2_CVT_VG2_SINGLE_BF16_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_nxv8bf16_ty],
                            [llvm_nxv4f32_ty, llvm_nxv4f32_ty],
                            [IntrNoMem]>;

  class SVE2_CVT_WIDENING_VG2_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                            [LLVMSubdivide2VectorType<0>], [IntrNoMem]>;


  class SVE2_CVT_VG4_SINGLE_Intrinsic
    : DefaultAttrsIntrinsic<[LLVMSubdivide4VectorType<0>],
````
- **L3393 EN**: Declares class `SVE2_VG2_Sel_Intrinsic`.
  **L3393 CN**: 声明 class `SVE2_VG2_Sel_Intrinsic`。
- **L3394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3394 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_aarch64_svcount_ty, LLVMMatchType<0>,`.
  **L3395 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_aarch64_svcount_ty, LLVMMatchType<0>,`。
- **L3396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3396 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3397 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>], [IntrNoMem]>;`.
  **L3397 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>], [IntrNoMem]>;`。
- **L3398 EN**: Blank line separating nearby declarations or logic blocks.
  **L3398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3399 EN**: Declares class `SVE2_VG4_Sel_Intrinsic`.
  **L3399 CN**: 声明 class `SVE2_VG4_Sel_Intrinsic`。
- **L3400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3400 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3401 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_aarch64_svcount_ty, LLVMMatchType<0>,`.
  **L3402 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_aarch64_svcount_ty, LLVMMatchType<0>,`。
- **L3403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3403 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3404 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3405 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3406 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>], [IntrNoMem]>;`.
  **L3406 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>], [IntrNoMem]>;`。
- **L3407 EN**: Blank line separating nearby declarations or logic blocks.
  **L3407 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3408 EN**: Declares class `SVE2_CVT_VG2_SINGLE_Intrinsic`.
  **L3408 CN**: 声明 class `SVE2_CVT_VG2_SINGLE_Intrinsic`。
- **L3409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`.
  **L3409 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide2VectorType<0>],`。
- **L3410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3410 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3411 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3411 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3412 EN**: Blank line separating nearby declarations or logic blocks.
  **L3412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3413 EN**: Declares class `SVE2_CVT_VG2_SINGLE_BF16_Intrinsic`.
  **L3413 CN**: 声明 class `SVE2_CVT_VG2_SINGLE_BF16_Intrinsic`。
- **L3414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv8bf16_ty],`.
  **L3414 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv8bf16_ty],`。
- **L3415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4f32_ty, llvm_nxv4f32_ty],`.
  **L3415 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4f32_ty, llvm_nxv4f32_ty],`。
- **L3416 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3416 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3417 EN**: Blank line separating nearby declarations or logic blocks.
  **L3417 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3418 EN**: Declares class `SVE2_CVT_WIDENING_VG2_Intrinsic`.
  **L3418 CN**: 声明 class `SVE2_CVT_WIDENING_VG2_Intrinsic`。
- **L3419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3419 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3420 EN**: Executes a standalone statement or declaration: `[LLVMSubdivide2VectorType<0>], [IntrNoMem]>;`.
  **L3420 CN**: 执行一条独立语句或声明：`[LLVMSubdivide2VectorType<0>], [IntrNoMem]>;`。
- **L3421 EN**: Blank line separating nearby declarations or logic blocks.
  **L3421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3422 EN**: Blank line separating nearby declarations or logic blocks.
  **L3422 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3423 EN**: Declares class `SVE2_CVT_VG4_SINGLE_Intrinsic`.
  **L3423 CN**: 声明 class `SVE2_CVT_VG4_SINGLE_Intrinsic`。
- **L3424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[LLVMSubdivide4VectorType<0>],`.
  **L3424 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[LLVMSubdivide4VectorType<0>],`。

### Lines 3425-3456

````tablegen
                            [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                            [IntrNoMem]>;

  class SVE2_CVT_X2_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                            [llvm_anyvector_ty, LLVMMatchType<1>],
                            [IntrNoMem]>;

  class SVE2_CVT_X4_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                            [llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>, LLVMMatchType<1>],
                            [IntrNoMem]>;

  class SVE2_BFMLS_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],
                            [llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty],
                            [IntrNoMem]>;

  class SVE2_BFMLS_Lane_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],
                            [llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<3>>]>;

  class SME2_ZA_ArrayVector_Read_VG2_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [llvm_i32_ty],
                [IntrReadMem, IntrInaccessibleMemOnly]>;

  class SME2_ZA_ArrayVector_Read_VG4_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>],
                [llvm_i32_ty],
````
- **L3425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3425 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3426 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3426 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3427 EN**: Blank line separating nearby declarations or logic blocks.
  **L3427 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3428 EN**: Declares class `SVE2_CVT_X2_Intrinsic`.
  **L3428 CN**: 声明 class `SVE2_CVT_X2_Intrinsic`。
- **L3429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3429 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<1>],`.
  **L3430 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<1>],`。
- **L3431 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3431 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3432 EN**: Blank line separating nearby declarations or logic blocks.
  **L3432 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3433 EN**: Declares class `SVE2_CVT_X4_Intrinsic`.
  **L3433 CN**: 声明 class `SVE2_CVT_X4_Intrinsic`。
- **L3434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3434 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L3435 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L3436 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3436 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3437 EN**: Blank line separating nearby declarations or logic blocks.
  **L3437 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3438 EN**: Declares class `SVE2_BFMLS_Intrinsic`.
  **L3438 CN**: 声明 class `SVE2_BFMLS_Intrinsic`。
- **L3439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`.
  **L3439 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`。
- **L3440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty],`.
  **L3440 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty],`。
- **L3441 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3441 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3442 EN**: Blank line separating nearby declarations or logic blocks.
  **L3442 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3443 EN**: Declares class `SVE2_BFMLS_Lane_Intrinsic`.
  **L3443 CN**: 声明 class `SVE2_BFMLS_Lane_Intrinsic`。
- **L3444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`.
  **L3444 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv4f32_ty],`。
- **L3445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty, llvm_i32_ty],`.
  **L3445 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4f32_ty, llvm_nxv8bf16_ty, llvm_nxv8bf16_ty, llvm_i32_ty],`。
- **L3446 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L3446 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L3447 EN**: Blank line separating nearby declarations or logic blocks.
  **L3447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3448 EN**: Declares class `SME2_ZA_ArrayVector_Read_VG2_Intrinsic`.
  **L3448 CN**: 声明 class `SME2_ZA_ArrayVector_Read_VG2_Intrinsic`。
- **L3449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3449 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L3450 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L3451 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L3451 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L3452 EN**: Blank line separating nearby declarations or logic blocks.
  **L3452 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3453 EN**: Declares class `SME2_ZA_ArrayVector_Read_VG4_Intrinsic`.
  **L3453 CN**: 声明 class `SME2_ZA_ArrayVector_Read_VG4_Intrinsic`。
- **L3454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3454 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3455 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L3456 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。

### Lines 3457-3488

````tablegen
                [IntrReadMem, IntrInaccessibleMemOnly]>;

  class SME2_Matrix_TileVector_Read_VG2_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [llvm_i32_ty, llvm_i32_ty],
                [IntrReadMem, IntrInaccessibleMemOnly]>;

  class SME2_Matrix_TileVector_Read_VG4_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>,  LLVMMatchType<0>],
                [llvm_i32_ty, llvm_i32_ty],
                [IntrReadMem, IntrInaccessibleMemOnly]>;

  class SME2_ZA_ArrayVector_Write_VG2_Intrinsic
   : DefaultAttrsIntrinsic<[],
               [llvm_i32_ty,
                llvm_anyvector_ty, LLVMMatchType<0>],
               [IntrWriteMem, IntrInaccessibleMemOnly]>;

  class SME2_ZA_ArrayVector_Write_VG4_Intrinsic
   : DefaultAttrsIntrinsic<[],
               [llvm_i32_ty,
                llvm_anyvector_ty, LLVMMatchType<0>,
                LLVMMatchType<0>,  LLVMMatchType<0>],
               [IntrWriteMem, IntrInaccessibleMemOnly]>;

  class SME2_Matrix_TileVector_Write_VG2_Intrinsic
   : DefaultAttrsIntrinsic<[],
               [llvm_i32_ty, llvm_i32_ty,
                llvm_anyvector_ty, LLVMMatchType<0>],
               [IntrWriteMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

````
- **L3457 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L3457 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L3458 EN**: Blank line separating nearby declarations or logic blocks.
  **L3458 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3459 EN**: Declares class `SME2_Matrix_TileVector_Read_VG2_Intrinsic`.
  **L3459 CN**: 声明 class `SME2_Matrix_TileVector_Read_VG2_Intrinsic`。
- **L3460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3460 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3461 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3462 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L3462 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L3463 EN**: Blank line separating nearby declarations or logic blocks.
  **L3463 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3464 EN**: Declares class `SME2_Matrix_TileVector_Read_VG4_Intrinsic`.
  **L3464 CN**: 声明 class `SME2_Matrix_TileVector_Read_VG4_Intrinsic`。
- **L3465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3465 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3466 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L3467 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L3468 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L3468 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L3469 EN**: Blank line separating nearby declarations or logic blocks.
  **L3469 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3470 EN**: Declares class `SME2_ZA_ArrayVector_Write_VG2_Intrinsic`.
  **L3470 CN**: 声明 class `SME2_ZA_ArrayVector_Write_VG2_Intrinsic`。
- **L3471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3471 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3472 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3473 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3474 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrInaccessibleMemOnly]>;`.
  **L3474 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrInaccessibleMemOnly]>;`。
- **L3475 EN**: Blank line separating nearby declarations or logic blocks.
  **L3475 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3476 EN**: Declares class `SME2_ZA_ArrayVector_Write_VG4_Intrinsic`.
  **L3476 CN**: 声明 class `SME2_ZA_ArrayVector_Write_VG4_Intrinsic`。
- **L3477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3477 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3478 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3479 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3480 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3481 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrInaccessibleMemOnly]>;`.
  **L3481 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrInaccessibleMemOnly]>;`。
- **L3482 EN**: Blank line separating nearby declarations or logic blocks.
  **L3482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3483 EN**: Declares class `SME2_Matrix_TileVector_Write_VG2_Intrinsic`.
  **L3483 CN**: 声明 class `SME2_Matrix_TileVector_Write_VG2_Intrinsic`。
- **L3484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3484 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty,`.
  **L3485 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty,`。
- **L3486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3486 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3487 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L3487 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L3488 EN**: Blank line separating nearby declarations or logic blocks.
  **L3488 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3489-3520

````tablegen
  class SME2_Matrix_TileVector_Write_VG4_Intrinsic
   : DefaultAttrsIntrinsic<[],
               [llvm_i32_ty, llvm_i32_ty,
                llvm_anyvector_ty, LLVMMatchType<0>,
                LLVMMatchType<0>,  LLVMMatchType<0>],
               [IntrWriteMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

  class SVE2_VG2_Multi_Single_Single_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrNoMem]>;

  class SVE2_VG4_Multi_Single_Single_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>],
                [IntrNoMem]>;

  class SVE2_VG2_ZipUzp_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;

  class SVE2_VG4_ZipUzp_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>, LLVMMatchType<0>],
                [LLVMMatchType<0>, LLVMMatchType<0>,
                 LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;

  class SVE2_VG2_Unpk_Intrinsic
````
- **L3489 EN**: Declares class `SME2_Matrix_TileVector_Write_VG4_Intrinsic`.
  **L3489 CN**: 声明 class `SME2_Matrix_TileVector_Write_VG4_Intrinsic`。
- **L3490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3490 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty,`.
  **L3491 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty,`。
- **L3492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3492 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>,  LLVMMatchType<0>],`.
  **L3493 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>,  LLVMMatchType<0>],`。
- **L3494 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L3494 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L3495 EN**: Blank line separating nearby declarations or logic blocks.
  **L3495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3496 EN**: Declares class `SVE2_VG2_Multi_Single_Single_Intrinsic`.
  **L3496 CN**: 声明 class `SVE2_VG2_Multi_Single_Single_Intrinsic`。
- **L3497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3497 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3498 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3499 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3500 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3500 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3501 EN**: Blank line separating nearby declarations or logic blocks.
  **L3501 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3502 EN**: Declares class `SVE2_VG4_Multi_Single_Single_Intrinsic`.
  **L3502 CN**: 声明 class `SVE2_VG4_Multi_Single_Single_Intrinsic`。
- **L3503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3503 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3504 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3505 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3506 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3507 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3508 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3508 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3509 EN**: Blank line separating nearby declarations or logic blocks.
  **L3509 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3510 EN**: Declares class `SVE2_VG2_ZipUzp_Intrinsic`.
  **L3510 CN**: 声明 class `SVE2_VG2_ZipUzp_Intrinsic`。
- **L3511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3511 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3512 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L3512 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L3513 EN**: Blank line separating nearby declarations or logic blocks.
  **L3513 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3514 EN**: Declares class `SVE2_VG4_ZipUzp_Intrinsic`.
  **L3514 CN**: 声明 class `SVE2_VG4_ZipUzp_Intrinsic`。
- **L3515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3515 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3516 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3517 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3518 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L3518 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L3519 EN**: Blank line separating nearby declarations or logic blocks.
  **L3519 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3520 EN**: Declares class `SVE2_VG2_Unpk_Intrinsic`.
  **L3520 CN**: 声明 class `SVE2_VG2_Unpk_Intrinsic`。

### Lines 3521-3552

````tablegen
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                [LLVMSubdivide2VectorType<0>], [IntrNoMem]>;

  class SVE2_VG4_Unpk_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,
                             LLVMMatchType<0>, LLVMMatchType<0>],
                [LLVMSubdivide2VectorType<0>, LLVMSubdivide2VectorType<0>],
                [IntrNoMem]>;

  //
  // Multi-vector fused multiply-add/subtract
  //

  def int_aarch64_sme_fmla_single_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sme_fmls_single_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sme_fmla_single_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;
  def int_aarch64_sme_fmls_single_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

  def int_aarch64_sme_fmla_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sme_fmls_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sme_fmla_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;
  def int_aarch64_sme_fmls_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;

  def int_aarch64_sme_fmla_lane_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
  def int_aarch64_sme_fmls_lane_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
  def int_aarch64_sme_fmla_lane_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
  def int_aarch64_sme_fmls_lane_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;

  //
  // Outer product and accumulate/subtract intrinsics
  //

````
- **L3521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L3521 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L3522 EN**: Executes a standalone statement or declaration: `[LLVMSubdivide2VectorType<0>], [IntrNoMem]>;`.
  **L3522 CN**: 执行一条独立语句或声明：`[LLVMSubdivide2VectorType<0>], [IntrNoMem]>;`。
- **L3523 EN**: Blank line separating nearby declarations or logic blocks.
  **L3523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3524 EN**: Declares class `SVE2_VG4_Unpk_Intrinsic`.
  **L3524 CN**: 声明 class `SVE2_VG4_Unpk_Intrinsic`。
- **L3525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3525 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3526 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMSubdivide2VectorType<0>, LLVMSubdivide2VectorType<0>],`.
  **L3527 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMSubdivide2VectorType<0>, LLVMSubdivide2VectorType<0>],`。
- **L3528 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L3528 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L3529 EN**: Blank line separating nearby declarations or logic blocks.
  **L3529 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3530 EN**: Separator comment used for visual grouping.
  **L3530 CN**: 用于视觉分组的分隔注释。
- **L3531 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector fused multiply-add/subtract`.
  **L3531 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector fused multiply-add/subtract`。
- **L3532 EN**: Separator comment used for visual grouping.
  **L3532 CN**: 用于视觉分组的分隔注释。
- **L3533 EN**: Blank line separating nearby declarations or logic blocks.
  **L3533 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3534 EN**: Declares TableGen def `int_aarch64_sme_fmla_single_vg1x2`.
  **L3534 CN**: 声明 TableGen def `int_aarch64_sme_fmla_single_vg1x2`。
- **L3535 EN**: Declares TableGen def `int_aarch64_sme_fmls_single_vg1x2`.
  **L3535 CN**: 声明 TableGen def `int_aarch64_sme_fmls_single_vg1x2`。
- **L3536 EN**: Declares TableGen def `int_aarch64_sme_fmla_single_vg1x4`.
  **L3536 CN**: 声明 TableGen def `int_aarch64_sme_fmla_single_vg1x4`。
- **L3537 EN**: Declares TableGen def `int_aarch64_sme_fmls_single_vg1x4`.
  **L3537 CN**: 声明 TableGen def `int_aarch64_sme_fmls_single_vg1x4`。
- **L3538 EN**: Blank line separating nearby declarations or logic blocks.
  **L3538 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3539 EN**: Declares TableGen def `int_aarch64_sme_fmla_vg1x2`.
  **L3539 CN**: 声明 TableGen def `int_aarch64_sme_fmla_vg1x2`。
- **L3540 EN**: Declares TableGen def `int_aarch64_sme_fmls_vg1x2`.
  **L3540 CN**: 声明 TableGen def `int_aarch64_sme_fmls_vg1x2`。
- **L3541 EN**: Declares TableGen def `int_aarch64_sme_fmla_vg1x4`.
  **L3541 CN**: 声明 TableGen def `int_aarch64_sme_fmla_vg1x4`。
- **L3542 EN**: Declares TableGen def `int_aarch64_sme_fmls_vg1x4`.
  **L3542 CN**: 声明 TableGen def `int_aarch64_sme_fmls_vg1x4`。
- **L3543 EN**: Blank line separating nearby declarations or logic blocks.
  **L3543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3544 EN**: Declares TableGen def `int_aarch64_sme_fmla_lane_vg1x2`.
  **L3544 CN**: 声明 TableGen def `int_aarch64_sme_fmla_lane_vg1x2`。
- **L3545 EN**: Declares TableGen def `int_aarch64_sme_fmls_lane_vg1x2`.
  **L3545 CN**: 声明 TableGen def `int_aarch64_sme_fmls_lane_vg1x2`。
- **L3546 EN**: Declares TableGen def `int_aarch64_sme_fmla_lane_vg1x4`.
  **L3546 CN**: 声明 TableGen def `int_aarch64_sme_fmla_lane_vg1x4`。
- **L3547 EN**: Declares TableGen def `int_aarch64_sme_fmls_lane_vg1x4`.
  **L3547 CN**: 声明 TableGen def `int_aarch64_sme_fmls_lane_vg1x4`。
- **L3548 EN**: Blank line separating nearby declarations or logic blocks.
  **L3548 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3549 EN**: Separator comment used for visual grouping.
  **L3549 CN**: 用于视觉分组的分隔注释。
- **L3550 EN**: Comment explains nearby logic, invariants, or intent: `Outer product and accumulate/subtract intrinsics`.
  **L3550 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Outer product and accumulate/subtract intrinsics`。
- **L3551 EN**: Separator comment used for visual grouping.
  **L3551 CN**: 用于视觉分组的分隔注释。
- **L3552 EN**: Blank line separating nearby declarations or logic blocks.
  **L3552 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3553-3584

````tablegen
  def int_aarch64_sme_smopa_za32 : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_umopa_za32 : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_smops_za32 : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_umops_za32 : SME_OuterProduct_Intrinsic;

  def int_aarch64_sme_bmopa_za32 : SME_OuterProduct_Intrinsic;
  def int_aarch64_sme_bmops_za32 : SME_OuterProduct_Intrinsic;

  //
  // Multi-vector rounding shift left intrinsics
  //

  def int_aarch64_sve_srshl_single_x2 : SVE2_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sve_urshl_single_x2 : SVE2_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sve_srshl_single_x4 : SVE2_VG4_Multi_Single_Intrinsic;
  def int_aarch64_sve_urshl_single_x4 : SVE2_VG4_Multi_Single_Intrinsic;

  def int_aarch64_sve_srshl_x2 : SVE2_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sve_urshl_x2 : SVE2_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sve_srshl_x4 : SVE2_VG4_Multi_Multi_Intrinsic;
  def int_aarch64_sve_urshl_x4 : SVE2_VG4_Multi_Multi_Intrinsic;

  // Multi-vector saturating rounding shift right intrinsics

  def int_aarch64_sve_sqrshr_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_uqrshr_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_sqrshr_x4 : SVE2_VG4_Multi_Imm_Intrinsic;
  def int_aarch64_sve_uqrshr_x4 : SVE2_VG4_Multi_Imm_Intrinsic;

  def int_aarch64_sve_sqrshrn_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_uqrshrn_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_sqrshrn_x4 : SVE2_VG4_Multi_Imm_Intrinsic;
````
- **L3553 EN**: Declares TableGen def `int_aarch64_sme_smopa_za32`.
  **L3553 CN**: 声明 TableGen def `int_aarch64_sme_smopa_za32`。
- **L3554 EN**: Declares TableGen def `int_aarch64_sme_umopa_za32`.
  **L3554 CN**: 声明 TableGen def `int_aarch64_sme_umopa_za32`。
- **L3555 EN**: Declares TableGen def `int_aarch64_sme_smops_za32`.
  **L3555 CN**: 声明 TableGen def `int_aarch64_sme_smops_za32`。
- **L3556 EN**: Declares TableGen def `int_aarch64_sme_umops_za32`.
  **L3556 CN**: 声明 TableGen def `int_aarch64_sme_umops_za32`。
- **L3557 EN**: Blank line separating nearby declarations or logic blocks.
  **L3557 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3558 EN**: Declares TableGen def `int_aarch64_sme_bmopa_za32`.
  **L3558 CN**: 声明 TableGen def `int_aarch64_sme_bmopa_za32`。
- **L3559 EN**: Declares TableGen def `int_aarch64_sme_bmops_za32`.
  **L3559 CN**: 声明 TableGen def `int_aarch64_sme_bmops_za32`。
- **L3560 EN**: Blank line separating nearby declarations or logic blocks.
  **L3560 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3561 EN**: Separator comment used for visual grouping.
  **L3561 CN**: 用于视觉分组的分隔注释。
- **L3562 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector rounding shift left intrinsics`.
  **L3562 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector rounding shift left intrinsics`。
- **L3563 EN**: Separator comment used for visual grouping.
  **L3563 CN**: 用于视觉分组的分隔注释。
- **L3564 EN**: Blank line separating nearby declarations or logic blocks.
  **L3564 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3565 EN**: Declares TableGen def `int_aarch64_sve_srshl_single_x2`.
  **L3565 CN**: 声明 TableGen def `int_aarch64_sve_srshl_single_x2`。
- **L3566 EN**: Declares TableGen def `int_aarch64_sve_urshl_single_x2`.
  **L3566 CN**: 声明 TableGen def `int_aarch64_sve_urshl_single_x2`。
- **L3567 EN**: Declares TableGen def `int_aarch64_sve_srshl_single_x4`.
  **L3567 CN**: 声明 TableGen def `int_aarch64_sve_srshl_single_x4`。
- **L3568 EN**: Declares TableGen def `int_aarch64_sve_urshl_single_x4`.
  **L3568 CN**: 声明 TableGen def `int_aarch64_sve_urshl_single_x4`。
- **L3569 EN**: Blank line separating nearby declarations or logic blocks.
  **L3569 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3570 EN**: Declares TableGen def `int_aarch64_sve_srshl_x2`.
  **L3570 CN**: 声明 TableGen def `int_aarch64_sve_srshl_x2`。
- **L3571 EN**: Declares TableGen def `int_aarch64_sve_urshl_x2`.
  **L3571 CN**: 声明 TableGen def `int_aarch64_sve_urshl_x2`。
- **L3572 EN**: Declares TableGen def `int_aarch64_sve_srshl_x4`.
  **L3572 CN**: 声明 TableGen def `int_aarch64_sve_srshl_x4`。
- **L3573 EN**: Declares TableGen def `int_aarch64_sve_urshl_x4`.
  **L3573 CN**: 声明 TableGen def `int_aarch64_sve_urshl_x4`。
- **L3574 EN**: Blank line separating nearby declarations or logic blocks.
  **L3574 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3575 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector saturating rounding shift right intrinsics`.
  **L3575 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector saturating rounding shift right intrinsics`。
- **L3576 EN**: Blank line separating nearby declarations or logic blocks.
  **L3576 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3577 EN**: Declares TableGen def `int_aarch64_sve_sqrshr_x2`.
  **L3577 CN**: 声明 TableGen def `int_aarch64_sve_sqrshr_x2`。
- **L3578 EN**: Declares TableGen def `int_aarch64_sve_uqrshr_x2`.
  **L3578 CN**: 声明 TableGen def `int_aarch64_sve_uqrshr_x2`。
- **L3579 EN**: Declares TableGen def `int_aarch64_sve_sqrshr_x4`.
  **L3579 CN**: 声明 TableGen def `int_aarch64_sve_sqrshr_x4`。
- **L3580 EN**: Declares TableGen def `int_aarch64_sve_uqrshr_x4`.
  **L3580 CN**: 声明 TableGen def `int_aarch64_sve_uqrshr_x4`。
- **L3581 EN**: Blank line separating nearby declarations or logic blocks.
  **L3581 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3582 EN**: Declares TableGen def `int_aarch64_sve_sqrshrn_x2`.
  **L3582 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrn_x2`。
- **L3583 EN**: Declares TableGen def `int_aarch64_sve_uqrshrn_x2`.
  **L3583 CN**: 声明 TableGen def `int_aarch64_sve_uqrshrn_x2`。
- **L3584 EN**: Declares TableGen def `int_aarch64_sve_sqrshrn_x4`.
  **L3584 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrn_x4`。

### Lines 3585-3616

````tablegen
  def int_aarch64_sve_uqrshrn_x4 : SVE2_VG4_Multi_Imm_Intrinsic;

  def int_aarch64_sve_sqrshru_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_sqrshru_x4 : SVE2_VG4_Multi_Imm_Intrinsic;

  def int_aarch64_sve_sqrshrun_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_sqrshrun_x4 : SVE2_VG4_Multi_Imm_Intrinsic;

  //
  // Multi-vector multiply-add/subtract long
  //

  foreach ty = ["f", "s", "u"] in {
    foreach instr = ["mlal", "mlsl"] in {
      def int_aarch64_sme_ # ty # instr # _single_vg2x1  : SME2_Matrix_ArrayVector_Single_Single_Intrinsic;
      def int_aarch64_sme_ # ty # instr # _single_vg2x2  : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
      def int_aarch64_sme_ # ty # instr # _single_vg2x4  : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

      def int_aarch64_sme_ # ty # instr # _vg2x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
      def int_aarch64_sme_ # ty # instr # _vg2x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;

      def int_aarch64_sme_ # ty # instr # _lane_vg2x1  : SME2_Matrix_ArrayVector_Single_Index_Intrinsic;
      def int_aarch64_sme_ # ty # instr # _lane_vg2x2  : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
      def int_aarch64_sme_ # ty # instr # _lane_vg2x4  : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
    }
  }

  //
  // Multi-vector multiply-add long long
  //

  foreach ty = ["s", "u"] in {
````
- **L3585 EN**: Declares TableGen def `int_aarch64_sve_uqrshrn_x4`.
  **L3585 CN**: 声明 TableGen def `int_aarch64_sve_uqrshrn_x4`。
- **L3586 EN**: Blank line separating nearby declarations or logic blocks.
  **L3586 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3587 EN**: Declares TableGen def `int_aarch64_sve_sqrshru_x2`.
  **L3587 CN**: 声明 TableGen def `int_aarch64_sve_sqrshru_x2`。
- **L3588 EN**: Declares TableGen def `int_aarch64_sve_sqrshru_x4`.
  **L3588 CN**: 声明 TableGen def `int_aarch64_sve_sqrshru_x4`。
- **L3589 EN**: Blank line separating nearby declarations or logic blocks.
  **L3589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3590 EN**: Declares TableGen def `int_aarch64_sve_sqrshrun_x2`.
  **L3590 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrun_x2`。
- **L3591 EN**: Declares TableGen def `int_aarch64_sve_sqrshrun_x4`.
  **L3591 CN**: 声明 TableGen def `int_aarch64_sve_sqrshrun_x4`。
- **L3592 EN**: Blank line separating nearby declarations or logic blocks.
  **L3592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3593 EN**: Separator comment used for visual grouping.
  **L3593 CN**: 用于视觉分组的分隔注释。
- **L3594 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector multiply-add/subtract long`.
  **L3594 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector multiply-add/subtract long`。
- **L3595 EN**: Separator comment used for visual grouping.
  **L3595 CN**: 用于视觉分组的分隔注释。
- **L3596 EN**: Blank line separating nearby declarations or logic blocks.
  **L3596 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3597 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3597 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3598 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3598 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3599 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3599 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3600 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3600 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3601 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3601 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3602 EN**: Blank line separating nearby declarations or logic blocks.
  **L3602 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3603 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3603 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3604 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3604 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3605 EN**: Blank line separating nearby declarations or logic blocks.
  **L3605 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3606 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3606 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3607 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3607 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3608 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3608 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3609 EN**: Closes the current lexical scope or compound statement.
  **L3609 CN**: 结束当前词法作用域或复合语句块。
- **L3610 EN**: Closes the current lexical scope or compound statement.
  **L3610 CN**: 结束当前词法作用域或复合语句块。
- **L3611 EN**: Blank line separating nearby declarations or logic blocks.
  **L3611 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3612 EN**: Separator comment used for visual grouping.
  **L3612 CN**: 用于视觉分组的分隔注释。
- **L3613 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector multiply-add long long`.
  **L3613 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector multiply-add long long`。
- **L3614 EN**: Separator comment used for visual grouping.
  **L3614 CN**: 用于视觉分组的分隔注释。
- **L3615 EN**: Blank line separating nearby declarations or logic blocks.
  **L3615 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3616 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3616 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 3617-3648

````tablegen
    foreach instr = ["mla", "mls"] in {
      foreach za = ["za32", "za64"] in {
        def int_aarch64_sme_ # ty # instr # _ # za # _single_vg4x1 : SME2_Matrix_ArrayVector_Single_Single_Intrinsic;
        def int_aarch64_sme_ # ty # instr # _ # za # _single_vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
        def int_aarch64_sme_ # ty # instr # _ # za # _single_vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

        def int_aarch64_sme_ # ty # instr # _ # za # _vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
        def int_aarch64_sme_ # ty # instr # _ # za # _vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;

        def int_aarch64_sme_ # ty # instr # _ # za # _lane_vg4x1 : SME2_Matrix_ArrayVector_Single_Index_Intrinsic;
        def int_aarch64_sme_ # ty # instr # _ # za # _lane_vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
        def int_aarch64_sme_ # ty # instr # _ # za # _lane_vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
      }
    }
  }

  def int_aarch64_sme_sumla_za32_single_vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sme_sumla_za32_single_vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

  def int_aarch64_sme_sumla_za32_lane_vg4x1 : SME2_Matrix_ArrayVector_Single_Index_Intrinsic;
  def int_aarch64_sme_sumla_za32_lane_vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
  def int_aarch64_sme_sumla_za32_lane_vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;

  def int_aarch64_sme_usmla_za32_single_vg4x1 : SME2_Matrix_ArrayVector_Single_Single_Intrinsic;
  def int_aarch64_sme_usmla_za32_single_vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sme_usmla_za32_single_vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

  def int_aarch64_sme_usmla_za32_vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sme_usmla_za32_vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;

  def int_aarch64_sme_usmla_za32_lane_vg4x1 : SME2_Matrix_ArrayVector_Single_Index_Intrinsic;
  def int_aarch64_sme_usmla_za32_lane_vg4x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
````
- **L3617 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3617 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3618 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3618 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3619 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3619 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3620 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3620 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3621 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3621 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3622 EN**: Blank line separating nearby declarations or logic blocks.
  **L3622 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3623 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3623 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3624 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3624 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3625 EN**: Blank line separating nearby declarations or logic blocks.
  **L3625 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3626 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3626 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3627 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3627 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3628 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3628 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3629 EN**: Closes the current lexical scope or compound statement.
  **L3629 CN**: 结束当前词法作用域或复合语句块。
- **L3630 EN**: Closes the current lexical scope or compound statement.
  **L3630 CN**: 结束当前词法作用域或复合语句块。
- **L3631 EN**: Closes the current lexical scope or compound statement.
  **L3631 CN**: 结束当前词法作用域或复合语句块。
- **L3632 EN**: Blank line separating nearby declarations or logic blocks.
  **L3632 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3633 EN**: Declares TableGen def `int_aarch64_sme_sumla_za32_single_vg4x2`.
  **L3633 CN**: 声明 TableGen def `int_aarch64_sme_sumla_za32_single_vg4x2`。
- **L3634 EN**: Declares TableGen def `int_aarch64_sme_sumla_za32_single_vg4x4`.
  **L3634 CN**: 声明 TableGen def `int_aarch64_sme_sumla_za32_single_vg4x4`。
- **L3635 EN**: Blank line separating nearby declarations or logic blocks.
  **L3635 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3636 EN**: Declares TableGen def `int_aarch64_sme_sumla_za32_lane_vg4x1`.
  **L3636 CN**: 声明 TableGen def `int_aarch64_sme_sumla_za32_lane_vg4x1`。
- **L3637 EN**: Declares TableGen def `int_aarch64_sme_sumla_za32_lane_vg4x2`.
  **L3637 CN**: 声明 TableGen def `int_aarch64_sme_sumla_za32_lane_vg4x2`。
- **L3638 EN**: Declares TableGen def `int_aarch64_sme_sumla_za32_lane_vg4x4`.
  **L3638 CN**: 声明 TableGen def `int_aarch64_sme_sumla_za32_lane_vg4x4`。
- **L3639 EN**: Blank line separating nearby declarations or logic blocks.
  **L3639 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3640 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_single_vg4x1`.
  **L3640 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_single_vg4x1`。
- **L3641 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_single_vg4x2`.
  **L3641 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_single_vg4x2`。
- **L3642 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_single_vg4x4`.
  **L3642 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_single_vg4x4`。
- **L3643 EN**: Blank line separating nearby declarations or logic blocks.
  **L3643 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3644 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_vg4x2`.
  **L3644 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_vg4x2`。
- **L3645 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_vg4x4`.
  **L3645 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_vg4x4`。
- **L3646 EN**: Blank line separating nearby declarations or logic blocks.
  **L3646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3647 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_lane_vg4x1`.
  **L3647 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_lane_vg4x1`。
- **L3648 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_lane_vg4x2`.
  **L3648 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_lane_vg4x2`。

### Lines 3649-3680

````tablegen
  def int_aarch64_sme_usmla_za32_lane_vg4x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;

  def int_aarch64_sve_bfmlslb : SVE2_BFMLS_Intrinsic;
  def int_aarch64_sve_bfmlslb_lane : SVE2_BFMLS_Lane_Intrinsic;

  def int_aarch64_sve_bfmlslt : SVE2_BFMLS_Intrinsic;
  def int_aarch64_sve_bfmlslt_lane : SVE2_BFMLS_Lane_Intrinsic;

  // Multi-vector zeroing

  foreach vg = ["vg1x2", "vg1x4", "vg2x1", "vg2x2", "vg2x4", "vg4x1", "vg4x2", "vg4x4"] in {
    def int_aarch64_sme_zero_za64_ # vg : DefaultAttrsIntrinsic<[], [llvm_i32_ty],  [IntrWriteMem, IntrInaccessibleMemOnly]>;
  }
  // Multi-vector signed saturating doubling multiply high
  def int_aarch64_sve_sqdmulh_single_vgx2 : SVE2_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sve_sqdmulh_single_vgx4 : SVE2_VG4_Multi_Single_Intrinsic;

  def int_aarch64_sve_sqdmulh_vgx2 : SVE2_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sve_sqdmulh_vgx4 : SVE2_VG4_Multi_Multi_Intrinsic;

  // Multi-vector floating-point round to integral value

  foreach inst = ["a", "m", "n", "p"] in {
    def int_aarch64_sve_frint # inst # _x2 : SVE2_VG2_ZipUzp_Intrinsic;
    def int_aarch64_sve_frint # inst # _x4 : SVE2_VG4_ZipUzp_Intrinsic;
  }

  //
  // Multi-vector min/max
  //

  foreach ty = ["f", "s", "u"] in {
````
- **L3649 EN**: Declares TableGen def `int_aarch64_sme_usmla_za32_lane_vg4x4`.
  **L3649 CN**: 声明 TableGen def `int_aarch64_sme_usmla_za32_lane_vg4x4`。
- **L3650 EN**: Blank line separating nearby declarations or logic blocks.
  **L3650 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3651 EN**: Declares TableGen def `int_aarch64_sve_bfmlslb`.
  **L3651 CN**: 声明 TableGen def `int_aarch64_sve_bfmlslb`。
- **L3652 EN**: Declares TableGen def `int_aarch64_sve_bfmlslb_lane`.
  **L3652 CN**: 声明 TableGen def `int_aarch64_sve_bfmlslb_lane`。
- **L3653 EN**: Blank line separating nearby declarations or logic blocks.
  **L3653 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3654 EN**: Declares TableGen def `int_aarch64_sve_bfmlslt`.
  **L3654 CN**: 声明 TableGen def `int_aarch64_sve_bfmlslt`。
- **L3655 EN**: Declares TableGen def `int_aarch64_sve_bfmlslt_lane`.
  **L3655 CN**: 声明 TableGen def `int_aarch64_sve_bfmlslt_lane`。
- **L3656 EN**: Blank line separating nearby declarations or logic blocks.
  **L3656 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3657 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector zeroing`.
  **L3657 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector zeroing`。
- **L3658 EN**: Blank line separating nearby declarations or logic blocks.
  **L3658 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3659 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3659 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3660 EN**: Declares TableGen def `int_aarch64_sme_zero_za64_`.
  **L3660 CN**: 声明 TableGen def `int_aarch64_sme_zero_za64_`。
- **L3661 EN**: Closes the current lexical scope or compound statement.
  **L3661 CN**: 结束当前词法作用域或复合语句块。
- **L3662 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector signed saturating doubling multiply high`.
  **L3662 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector signed saturating doubling multiply high`。
- **L3663 EN**: Declares TableGen def `int_aarch64_sve_sqdmulh_single_vgx2`.
  **L3663 CN**: 声明 TableGen def `int_aarch64_sve_sqdmulh_single_vgx2`。
- **L3664 EN**: Declares TableGen def `int_aarch64_sve_sqdmulh_single_vgx4`.
  **L3664 CN**: 声明 TableGen def `int_aarch64_sve_sqdmulh_single_vgx4`。
- **L3665 EN**: Blank line separating nearby declarations or logic blocks.
  **L3665 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3666 EN**: Declares TableGen def `int_aarch64_sve_sqdmulh_vgx2`.
  **L3666 CN**: 声明 TableGen def `int_aarch64_sve_sqdmulh_vgx2`。
- **L3667 EN**: Declares TableGen def `int_aarch64_sve_sqdmulh_vgx4`.
  **L3667 CN**: 声明 TableGen def `int_aarch64_sve_sqdmulh_vgx4`。
- **L3668 EN**: Blank line separating nearby declarations or logic blocks.
  **L3668 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3669 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector floating-point round to integral value`.
  **L3669 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector floating-point round to integral value`。
- **L3670 EN**: Blank line separating nearby declarations or logic blocks.
  **L3670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3671 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3671 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3672 EN**: Declares TableGen def `int_aarch64_sve_frint`.
  **L3672 CN**: 声明 TableGen def `int_aarch64_sve_frint`。
- **L3673 EN**: Declares TableGen def `int_aarch64_sve_frint`.
  **L3673 CN**: 声明 TableGen def `int_aarch64_sve_frint`。
- **L3674 EN**: Closes the current lexical scope or compound statement.
  **L3674 CN**: 结束当前词法作用域或复合语句块。
- **L3675 EN**: Blank line separating nearby declarations or logic blocks.
  **L3675 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3676 EN**: Separator comment used for visual grouping.
  **L3676 CN**: 用于视觉分组的分隔注释。
- **L3677 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector min/max`.
  **L3677 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector min/max`。
- **L3678 EN**: Separator comment used for visual grouping.
  **L3678 CN**: 用于视觉分组的分隔注释。
- **L3679 EN**: Blank line separating nearby declarations or logic blocks.
  **L3679 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3680 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3680 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 3681-3712

````tablegen
    foreach instr = ["max", "min"] in {
      def int_aarch64_sve_ # ty # instr # _single_x2 : SVE2_VG2_Multi_Single_Intrinsic;
      def int_aarch64_sve_ # ty # instr # _single_x4 : SVE2_VG4_Multi_Single_Intrinsic;

      def int_aarch64_sve_ # ty # instr # _x2 : SVE2_VG2_Multi_Multi_Intrinsic;
      def int_aarch64_sve_ # ty # instr # _x4 : SVE2_VG4_Multi_Multi_Intrinsic;
    }
  }

  //
  // Multi-vector floating point min/max number, scale, and multiply
  //

  foreach instr = ["fmaxnm", "fminnm", "fmul"] in {
    def int_aarch64_sve_ # instr # _single_x2 : SVE2_VG2_Multi_Single_Intrinsic;
    def int_aarch64_sve_ # instr # _single_x4 : SVE2_VG4_Multi_Single_Intrinsic;

    def int_aarch64_sve_ # instr # _x2 : SVE2_VG2_Multi_Multi_Intrinsic;
    def int_aarch64_sve_ # instr # _x4 : SVE2_VG4_Multi_Multi_Intrinsic;
  }

  //
  // Multi-vector floating point scale
  //

  def int_aarch64_sve_fscale_single_x2 : SVE2_VG2_Multi_Single_Scale_Intrinsic;
  def int_aarch64_sve_fscale_single_x4 : SVE2_VG4_Multi_Single_Scale_Intrinsic;

  def int_aarch64_sve_fscale_x2 : SVE2_VG2_Multi_Multi_Scale_Intrinsic;
  def int_aarch64_sve_fscale_x4 : SVE2_VG4_Multi_Multi_Scale_Intrinsic;
  
  //
````
- **L3681 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3681 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3682 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3682 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3683 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3683 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3684 EN**: Blank line separating nearby declarations or logic blocks.
  **L3684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3685 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3685 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3686 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3686 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3687 EN**: Closes the current lexical scope or compound statement.
  **L3687 CN**: 结束当前词法作用域或复合语句块。
- **L3688 EN**: Closes the current lexical scope or compound statement.
  **L3688 CN**: 结束当前词法作用域或复合语句块。
- **L3689 EN**: Blank line separating nearby declarations or logic blocks.
  **L3689 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3690 EN**: Separator comment used for visual grouping.
  **L3690 CN**: 用于视觉分组的分隔注释。
- **L3691 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector floating point min/max number, scale, and multiply`.
  **L3691 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector floating point min/max number, scale, and multiply`。
- **L3692 EN**: Separator comment used for visual grouping.
  **L3692 CN**: 用于视觉分组的分隔注释。
- **L3693 EN**: Blank line separating nearby declarations or logic blocks.
  **L3693 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3694 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3694 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3695 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3695 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3696 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3696 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3697 EN**: Blank line separating nearby declarations or logic blocks.
  **L3697 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3698 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3698 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3699 EN**: Declares TableGen def `int_aarch64_sve_`.
  **L3699 CN**: 声明 TableGen def `int_aarch64_sve_`。
- **L3700 EN**: Closes the current lexical scope or compound statement.
  **L3700 CN**: 结束当前词法作用域或复合语句块。
- **L3701 EN**: Blank line separating nearby declarations or logic blocks.
  **L3701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3702 EN**: Separator comment used for visual grouping.
  **L3702 CN**: 用于视觉分组的分隔注释。
- **L3703 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector floating point scale`.
  **L3703 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector floating point scale`。
- **L3704 EN**: Separator comment used for visual grouping.
  **L3704 CN**: 用于视觉分组的分隔注释。
- **L3705 EN**: Blank line separating nearby declarations or logic blocks.
  **L3705 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3706 EN**: Declares TableGen def `int_aarch64_sve_fscale_single_x2`.
  **L3706 CN**: 声明 TableGen def `int_aarch64_sve_fscale_single_x2`。
- **L3707 EN**: Declares TableGen def `int_aarch64_sve_fscale_single_x4`.
  **L3707 CN**: 声明 TableGen def `int_aarch64_sve_fscale_single_x4`。
- **L3708 EN**: Blank line separating nearby declarations or logic blocks.
  **L3708 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3709 EN**: Declares TableGen def `int_aarch64_sve_fscale_x2`.
  **L3709 CN**: 声明 TableGen def `int_aarch64_sve_fscale_x2`。
- **L3710 EN**: Declares TableGen def `int_aarch64_sve_fscale_x4`.
  **L3710 CN**: 声明 TableGen def `int_aarch64_sve_fscale_x4`。
- **L3711 EN**: Blank line separating nearby declarations or logic blocks.
  **L3711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3712 EN**: Separator comment used for visual grouping.
  **L3712 CN**: 用于视觉分组的分隔注释。

### Lines 3713-3744

````tablegen
  // Multi-vector floating point absolute min/max number
  //

  foreach instr = ["famax", "famin"] in {
    def int_aarch64_sme_ # instr # _x2 : SVE2_VG2_Multi_Multi_Intrinsic;
    def int_aarch64_sme_ # instr # _x4 : SVE2_VG4_Multi_Multi_Intrinsic;
  }

  //
  // Multi-vector vertical dot-products
  //

  def int_aarch64_sme_fvdot_lane_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;

  foreach ty = ["s", "u"] in {
    def int_aarch64_sme_ #ty # vdot_lane_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
    def int_aarch64_sme_ #ty # vdot_lane_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
    def int_aarch64_sme_ #ty # vdot_lane_za64_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
  }

  def int_aarch64_sme_suvdot_lane_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
  def int_aarch64_sme_usvdot_lane_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;


  //
  //Multi-vector floating-point convert from half-precision to deinterleaved single-precision.
  //
  def int_aarch64_sve_fcvtl_widen_x2  : SVE2_CVT_WIDENING_VG2_Intrinsic;

  //
  // Multi-vector floating-point CVT from single-precision to interleaved half-precision/BFloat16
  //
````
- **L3713 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector floating point absolute min/max number`.
  **L3713 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector floating point absolute min/max number`。
- **L3714 EN**: Separator comment used for visual grouping.
  **L3714 CN**: 用于视觉分组的分隔注释。
- **L3715 EN**: Blank line separating nearby declarations or logic blocks.
  **L3715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3716 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3716 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3717 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3717 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3718 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3718 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3719 EN**: Closes the current lexical scope or compound statement.
  **L3719 CN**: 结束当前词法作用域或复合语句块。
- **L3720 EN**: Blank line separating nearby declarations or logic blocks.
  **L3720 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3721 EN**: Separator comment used for visual grouping.
  **L3721 CN**: 用于视觉分组的分隔注释。
- **L3722 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector vertical dot-products`.
  **L3722 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector vertical dot-products`。
- **L3723 EN**: Separator comment used for visual grouping.
  **L3723 CN**: 用于视觉分组的分隔注释。
- **L3724 EN**: Blank line separating nearby declarations or logic blocks.
  **L3724 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3725 EN**: Declares TableGen def `int_aarch64_sme_fvdot_lane_za32_vg1x2`.
  **L3725 CN**: 声明 TableGen def `int_aarch64_sme_fvdot_lane_za32_vg1x2`。
- **L3726 EN**: Blank line separating nearby declarations or logic blocks.
  **L3726 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3727 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3727 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3728 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3728 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3729 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3729 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3730 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3730 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3731 EN**: Closes the current lexical scope or compound statement.
  **L3731 CN**: 结束当前词法作用域或复合语句块。
- **L3732 EN**: Blank line separating nearby declarations or logic blocks.
  **L3732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3733 EN**: Declares TableGen def `int_aarch64_sme_suvdot_lane_za32_vg1x4`.
  **L3733 CN**: 声明 TableGen def `int_aarch64_sme_suvdot_lane_za32_vg1x4`。
- **L3734 EN**: Declares TableGen def `int_aarch64_sme_usvdot_lane_za32_vg1x4`.
  **L3734 CN**: 声明 TableGen def `int_aarch64_sme_usvdot_lane_za32_vg1x4`。
- **L3735 EN**: Blank line separating nearby declarations or logic blocks.
  **L3735 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3736 EN**: Blank line separating nearby declarations or logic blocks.
  **L3736 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3737 EN**: Separator comment used for visual grouping.
  **L3737 CN**: 用于视觉分组的分隔注释。
- **L3738 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector floating-point convert from half-precision to deinterleaved single-precision.`.
  **L3738 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector floating-point convert from half-precision to deinterleaved single-precision.`。
- **L3739 EN**: Separator comment used for visual grouping.
  **L3739 CN**: 用于视觉分组的分隔注释。
- **L3740 EN**: Declares TableGen def `int_aarch64_sve_fcvtl_widen_x2`.
  **L3740 CN**: 声明 TableGen def `int_aarch64_sve_fcvtl_widen_x2`。
- **L3741 EN**: Blank line separating nearby declarations or logic blocks.
  **L3741 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3742 EN**: Separator comment used for visual grouping.
  **L3742 CN**: 用于视觉分组的分隔注释。
- **L3743 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector floating-point CVT from single-precision to interleaved half-precision/BFloat16`.
  **L3743 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector floating-point CVT from single-precision to interleaved half-precision/BFloat16`。
- **L3744 EN**: Separator comment used for visual grouping.
  **L3744 CN**: 用于视觉分组的分隔注释。

### Lines 3745-3776

````tablegen
  def int_aarch64_sve_fcvtn_x2  : SVE2_CVT_VG2_SINGLE_Intrinsic;
  def int_aarch64_sve_bfcvtn_x2 : SVE2_CVT_VG2_SINGLE_BF16_Intrinsic;

  //
  // Multi-vector convert to/from floating-point.
  //
  def int_aarch64_sve_fcvt_x2  : SVE2_CVT_VG2_SINGLE_Intrinsic;
  def int_aarch64_sve_bfcvt_x2 : SVE2_CVT_VG2_SINGLE_BF16_Intrinsic;
  def int_aarch64_sve_fcvtzs_x2 : SVE2_CVT_X2_Intrinsic;
  def int_aarch64_sve_fcvtzu_x2 : SVE2_CVT_X2_Intrinsic;
  def int_aarch64_sve_scvtf_x2  : SVE2_CVT_X2_Intrinsic;
  def int_aarch64_sve_ucvtf_x2  : SVE2_CVT_X2_Intrinsic;
  def int_aarch64_sve_fcvtzs_x4 : SVE2_CVT_X4_Intrinsic;
  def int_aarch64_sve_fcvtzu_x4 : SVE2_CVT_X4_Intrinsic;
  def int_aarch64_sve_scvtf_x4  : SVE2_CVT_X4_Intrinsic;
  def int_aarch64_sve_ucvtf_x4  : SVE2_CVT_X4_Intrinsic;
  def int_aarch64_sve_fcvt_widen_x2 : SVE2_CVT_WIDENING_VG2_Intrinsic;
  //
  // Multi-vector saturating extract narrow
  //
  def int_aarch64_sve_sqcvt_x2  : SVE2_CVT_VG2_SINGLE_Intrinsic;
  def int_aarch64_sve_uqcvt_x2  : SVE2_CVT_VG2_SINGLE_Intrinsic;
  def int_aarch64_sve_sqcvtu_x2 : SVE2_CVT_VG2_SINGLE_Intrinsic;
  def int_aarch64_sve_sqcvt_x4  : SVE2_CVT_VG4_SINGLE_Intrinsic;
  def int_aarch64_sve_uqcvt_x4  : SVE2_CVT_VG4_SINGLE_Intrinsic;
  def int_aarch64_sve_sqcvtu_x4 : SVE2_CVT_VG4_SINGLE_Intrinsic;

  //
  // Multi-vector saturating extract narrow and interleave
  //
  def int_aarch64_sve_sqcvtn_x2  : SVE2_CVT_VG2_SINGLE_Intrinsic;
  def int_aarch64_sve_uqcvtn_x2  : SVE2_CVT_VG2_SINGLE_Intrinsic;
````
- **L3745 EN**: Declares TableGen def `int_aarch64_sve_fcvtn_x2`.
  **L3745 CN**: 声明 TableGen def `int_aarch64_sve_fcvtn_x2`。
- **L3746 EN**: Declares TableGen def `int_aarch64_sve_bfcvtn_x2`.
  **L3746 CN**: 声明 TableGen def `int_aarch64_sve_bfcvtn_x2`。
- **L3747 EN**: Blank line separating nearby declarations or logic blocks.
  **L3747 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3748 EN**: Separator comment used for visual grouping.
  **L3748 CN**: 用于视觉分组的分隔注释。
- **L3749 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector convert to/from floating-point.`.
  **L3749 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector convert to/from floating-point.`。
- **L3750 EN**: Separator comment used for visual grouping.
  **L3750 CN**: 用于视觉分组的分隔注释。
- **L3751 EN**: Declares TableGen def `int_aarch64_sve_fcvt_x2`.
  **L3751 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_x2`。
- **L3752 EN**: Declares TableGen def `int_aarch64_sve_bfcvt_x2`.
  **L3752 CN**: 声明 TableGen def `int_aarch64_sve_bfcvt_x2`。
- **L3753 EN**: Declares TableGen def `int_aarch64_sve_fcvtzs_x2`.
  **L3753 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzs_x2`。
- **L3754 EN**: Declares TableGen def `int_aarch64_sve_fcvtzu_x2`.
  **L3754 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzu_x2`。
- **L3755 EN**: Declares TableGen def `int_aarch64_sve_scvtf_x2`.
  **L3755 CN**: 声明 TableGen def `int_aarch64_sve_scvtf_x2`。
- **L3756 EN**: Declares TableGen def `int_aarch64_sve_ucvtf_x2`.
  **L3756 CN**: 声明 TableGen def `int_aarch64_sve_ucvtf_x2`。
- **L3757 EN**: Declares TableGen def `int_aarch64_sve_fcvtzs_x4`.
  **L3757 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzs_x4`。
- **L3758 EN**: Declares TableGen def `int_aarch64_sve_fcvtzu_x4`.
  **L3758 CN**: 声明 TableGen def `int_aarch64_sve_fcvtzu_x4`。
- **L3759 EN**: Declares TableGen def `int_aarch64_sve_scvtf_x4`.
  **L3759 CN**: 声明 TableGen def `int_aarch64_sve_scvtf_x4`。
- **L3760 EN**: Declares TableGen def `int_aarch64_sve_ucvtf_x4`.
  **L3760 CN**: 声明 TableGen def `int_aarch64_sve_ucvtf_x4`。
- **L3761 EN**: Declares TableGen def `int_aarch64_sve_fcvt_widen_x2`.
  **L3761 CN**: 声明 TableGen def `int_aarch64_sve_fcvt_widen_x2`。
- **L3762 EN**: Separator comment used for visual grouping.
  **L3762 CN**: 用于视觉分组的分隔注释。
- **L3763 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector saturating extract narrow`.
  **L3763 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector saturating extract narrow`。
- **L3764 EN**: Separator comment used for visual grouping.
  **L3764 CN**: 用于视觉分组的分隔注释。
- **L3765 EN**: Declares TableGen def `int_aarch64_sve_sqcvt_x2`.
  **L3765 CN**: 声明 TableGen def `int_aarch64_sve_sqcvt_x2`。
- **L3766 EN**: Declares TableGen def `int_aarch64_sve_uqcvt_x2`.
  **L3766 CN**: 声明 TableGen def `int_aarch64_sve_uqcvt_x2`。
- **L3767 EN**: Declares TableGen def `int_aarch64_sve_sqcvtu_x2`.
  **L3767 CN**: 声明 TableGen def `int_aarch64_sve_sqcvtu_x2`。
- **L3768 EN**: Declares TableGen def `int_aarch64_sve_sqcvt_x4`.
  **L3768 CN**: 声明 TableGen def `int_aarch64_sve_sqcvt_x4`。
- **L3769 EN**: Declares TableGen def `int_aarch64_sve_uqcvt_x4`.
  **L3769 CN**: 声明 TableGen def `int_aarch64_sve_uqcvt_x4`。
- **L3770 EN**: Declares TableGen def `int_aarch64_sve_sqcvtu_x4`.
  **L3770 CN**: 声明 TableGen def `int_aarch64_sve_sqcvtu_x4`。
- **L3771 EN**: Blank line separating nearby declarations or logic blocks.
  **L3771 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3772 EN**: Separator comment used for visual grouping.
  **L3772 CN**: 用于视觉分组的分隔注释。
- **L3773 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector saturating extract narrow and interleave`.
  **L3773 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector saturating extract narrow and interleave`。
- **L3774 EN**: Separator comment used for visual grouping.
  **L3774 CN**: 用于视觉分组的分隔注释。
- **L3775 EN**: Declares TableGen def `int_aarch64_sve_sqcvtn_x2`.
  **L3775 CN**: 声明 TableGen def `int_aarch64_sve_sqcvtn_x2`。
- **L3776 EN**: Declares TableGen def `int_aarch64_sve_uqcvtn_x2`.
  **L3776 CN**: 声明 TableGen def `int_aarch64_sve_uqcvtn_x2`。

### Lines 3777-3808

````tablegen
  def int_aarch64_sve_sqcvtun_x2 : SVE2_CVT_VG2_SINGLE_Intrinsic;
  def int_aarch64_sve_sqcvtn_x4  : SVE2_CVT_VG4_SINGLE_Intrinsic;
  def int_aarch64_sve_uqcvtn_x4  : SVE2_CVT_VG4_SINGLE_Intrinsic;
  def int_aarch64_sve_sqcvtun_x4 : SVE2_CVT_VG4_SINGLE_Intrinsic;

  //
  // Multi-vector saturating shift right narrow and interleave
  //
  def int_aarch64_sve_sqshrn_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_uqshrn_x2 : SVE2_VG2_Multi_Imm_Intrinsic;
  def int_aarch64_sve_sqshrun_x2 : SVE2_VG2_Multi_Imm_Intrinsic;

  //
  // Multi-Single add/sub
  //

  class SME2_Add_Sub_Write_VG2_Multi_Single_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_anyvector_ty, LLVMMatchType<0>,
          LLVMMatchType<0>],
          [IntrInaccessibleMemOnly, IntrWriteMem]>;

  class SME2_Add_Sub_Write_VG4_Multi_Single_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
          LLVMMatchType<0>],
        [IntrInaccessibleMemOnly, IntrWriteMem]>;

  def int_aarch64_sme_add_write_single_za_vg1x2 : SME2_Add_Sub_Write_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sme_sub_write_single_za_vg1x2 : SME2_Add_Sub_Write_VG2_Multi_Single_Intrinsic;
````
- **L3777 EN**: Declares TableGen def `int_aarch64_sve_sqcvtun_x2`.
  **L3777 CN**: 声明 TableGen def `int_aarch64_sve_sqcvtun_x2`。
- **L3778 EN**: Declares TableGen def `int_aarch64_sve_sqcvtn_x4`.
  **L3778 CN**: 声明 TableGen def `int_aarch64_sve_sqcvtn_x4`。
- **L3779 EN**: Declares TableGen def `int_aarch64_sve_uqcvtn_x4`.
  **L3779 CN**: 声明 TableGen def `int_aarch64_sve_uqcvtn_x4`。
- **L3780 EN**: Declares TableGen def `int_aarch64_sve_sqcvtun_x4`.
  **L3780 CN**: 声明 TableGen def `int_aarch64_sve_sqcvtun_x4`。
- **L3781 EN**: Blank line separating nearby declarations or logic blocks.
  **L3781 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3782 EN**: Separator comment used for visual grouping.
  **L3782 CN**: 用于视觉分组的分隔注释。
- **L3783 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector saturating shift right narrow and interleave`.
  **L3783 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector saturating shift right narrow and interleave`。
- **L3784 EN**: Separator comment used for visual grouping.
  **L3784 CN**: 用于视觉分组的分隔注释。
- **L3785 EN**: Declares TableGen def `int_aarch64_sve_sqshrn_x2`.
  **L3785 CN**: 声明 TableGen def `int_aarch64_sve_sqshrn_x2`。
- **L3786 EN**: Declares TableGen def `int_aarch64_sve_uqshrn_x2`.
  **L3786 CN**: 声明 TableGen def `int_aarch64_sve_uqshrn_x2`。
- **L3787 EN**: Declares TableGen def `int_aarch64_sve_sqshrun_x2`.
  **L3787 CN**: 声明 TableGen def `int_aarch64_sve_sqshrun_x2`。
- **L3788 EN**: Blank line separating nearby declarations or logic blocks.
  **L3788 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3789 EN**: Separator comment used for visual grouping.
  **L3789 CN**: 用于视觉分组的分隔注释。
- **L3790 EN**: Comment explains nearby logic, invariants, or intent: `Multi-Single add/sub`.
  **L3790 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-Single add/sub`。
- **L3791 EN**: Separator comment used for visual grouping.
  **L3791 CN**: 用于视觉分组的分隔注释。
- **L3792 EN**: Blank line separating nearby declarations or logic blocks.
  **L3792 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3793 EN**: Declares class `SME2_Add_Sub_Write_VG2_Multi_Single_Intrinsic`.
  **L3793 CN**: 声明 class `SME2_Add_Sub_Write_VG2_Multi_Single_Intrinsic`。
- **L3794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3794 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3795 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3796 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L3797 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L3798 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, IntrWriteMem]>;`.
  **L3798 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, IntrWriteMem]>;`。
- **L3799 EN**: Blank line separating nearby declarations or logic blocks.
  **L3799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3800 EN**: Declares class `SME2_Add_Sub_Write_VG4_Multi_Single_Intrinsic`.
  **L3800 CN**: 声明 class `SME2_Add_Sub_Write_VG4_Multi_Single_Intrinsic`。
- **L3801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3801 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3802 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3803 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L3804 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L3805 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, IntrWriteMem]>;`.
  **L3805 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, IntrWriteMem]>;`。
- **L3806 EN**: Blank line separating nearby declarations or logic blocks.
  **L3806 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3807 EN**: Declares TableGen def `int_aarch64_sme_add_write_single_za_vg1x2`.
  **L3807 CN**: 声明 TableGen def `int_aarch64_sme_add_write_single_za_vg1x2`。
- **L3808 EN**: Declares TableGen def `int_aarch64_sme_sub_write_single_za_vg1x2`.
  **L3808 CN**: 声明 TableGen def `int_aarch64_sme_sub_write_single_za_vg1x2`。

### Lines 3809-3840

````tablegen
  def int_aarch64_sme_add_write_single_za_vg1x4 : SME2_Add_Sub_Write_VG4_Multi_Single_Intrinsic;
  def int_aarch64_sme_sub_write_single_za_vg1x4 : SME2_Add_Sub_Write_VG4_Multi_Single_Intrinsic;

  //
  // Multi-Multi add/sub
  //
  class SME2_Add_Sub_Write_VG2_Multi_Multi_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_anyvector_ty, LLVMMatchType<0>,
          LLVMMatchType<0>, LLVMMatchType<0>],
          [IntrInaccessibleMemOnly, IntrWriteMem]>;

  class SME2_Add_Sub_Write_VG4_Multi_Multi_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,
          LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
          LLVMMatchType<0>, LLVMMatchType<0>],
          [IntrInaccessibleMemOnly, IntrWriteMem]>;

  def int_aarch64_sme_add_write_za_vg1x2 : SME2_Add_Sub_Write_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sme_sub_write_za_vg1x2 : SME2_Add_Sub_Write_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sme_add_write_za_vg1x4 : SME2_Add_Sub_Write_VG4_Multi_Multi_Intrinsic;
  def int_aarch64_sme_sub_write_za_vg1x4 : SME2_Add_Sub_Write_VG4_Multi_Multi_Intrinsic;

  // Multi-vector clamps
  def int_aarch64_sve_sclamp_single_x2 : SVE2_VG2_Multi_Single_Single_Intrinsic;
  def int_aarch64_sve_uclamp_single_x2 : SVE2_VG2_Multi_Single_Single_Intrinsic;
  def int_aarch64_sve_fclamp_single_x2 : SVE2_VG2_Multi_Single_Single_Intrinsic;
  def int_aarch64_sve_bfclamp_single_x2 : SVE2_VG2_Multi_Single_Single_Intrinsic;

````
- **L3809 EN**: Declares TableGen def `int_aarch64_sme_add_write_single_za_vg1x4`.
  **L3809 CN**: 声明 TableGen def `int_aarch64_sme_add_write_single_za_vg1x4`。
- **L3810 EN**: Declares TableGen def `int_aarch64_sme_sub_write_single_za_vg1x4`.
  **L3810 CN**: 声明 TableGen def `int_aarch64_sme_sub_write_single_za_vg1x4`。
- **L3811 EN**: Blank line separating nearby declarations or logic blocks.
  **L3811 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3812 EN**: Separator comment used for visual grouping.
  **L3812 CN**: 用于视觉分组的分隔注释。
- **L3813 EN**: Comment explains nearby logic, invariants, or intent: `Multi-Multi add/sub`.
  **L3813 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-Multi add/sub`。
- **L3814 EN**: Separator comment used for visual grouping.
  **L3814 CN**: 用于视觉分组的分隔注释。
- **L3815 EN**: Declares class `SME2_Add_Sub_Write_VG2_Multi_Multi_Intrinsic`.
  **L3815 CN**: 声明 class `SME2_Add_Sub_Write_VG2_Multi_Multi_Intrinsic`。
- **L3816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3816 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3817 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>,`.
  **L3818 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>,`。
- **L3819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3819 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3820 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, IntrWriteMem]>;`.
  **L3820 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, IntrWriteMem]>;`。
- **L3821 EN**: Blank line separating nearby declarations or logic blocks.
  **L3821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3822 EN**: Declares class `SME2_Add_Sub_Write_VG4_Multi_Multi_Intrinsic`.
  **L3822 CN**: 声明 class `SME2_Add_Sub_Write_VG4_Multi_Multi_Intrinsic`。
- **L3823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3823 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L3824 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L3825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3825 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L3826 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L3827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3827 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3828 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, IntrWriteMem]>;`.
  **L3828 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, IntrWriteMem]>;`。
- **L3829 EN**: Blank line separating nearby declarations or logic blocks.
  **L3829 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3830 EN**: Declares TableGen def `int_aarch64_sme_add_write_za_vg1x2`.
  **L3830 CN**: 声明 TableGen def `int_aarch64_sme_add_write_za_vg1x2`。
- **L3831 EN**: Declares TableGen def `int_aarch64_sme_sub_write_za_vg1x2`.
  **L3831 CN**: 声明 TableGen def `int_aarch64_sme_sub_write_za_vg1x2`。
- **L3832 EN**: Declares TableGen def `int_aarch64_sme_add_write_za_vg1x4`.
  **L3832 CN**: 声明 TableGen def `int_aarch64_sme_add_write_za_vg1x4`。
- **L3833 EN**: Declares TableGen def `int_aarch64_sme_sub_write_za_vg1x4`.
  **L3833 CN**: 声明 TableGen def `int_aarch64_sme_sub_write_za_vg1x4`。
- **L3834 EN**: Blank line separating nearby declarations or logic blocks.
  **L3834 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3835 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector clamps`.
  **L3835 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector clamps`。
- **L3836 EN**: Declares TableGen def `int_aarch64_sve_sclamp_single_x2`.
  **L3836 CN**: 声明 TableGen def `int_aarch64_sve_sclamp_single_x2`。
- **L3837 EN**: Declares TableGen def `int_aarch64_sve_uclamp_single_x2`.
  **L3837 CN**: 声明 TableGen def `int_aarch64_sve_uclamp_single_x2`。
- **L3838 EN**: Declares TableGen def `int_aarch64_sve_fclamp_single_x2`.
  **L3838 CN**: 声明 TableGen def `int_aarch64_sve_fclamp_single_x2`。
- **L3839 EN**: Declares TableGen def `int_aarch64_sve_bfclamp_single_x2`.
  **L3839 CN**: 声明 TableGen def `int_aarch64_sve_bfclamp_single_x2`。
- **L3840 EN**: Blank line separating nearby declarations or logic blocks.
  **L3840 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3841-3872

````tablegen
  def int_aarch64_sve_sclamp_single_x4 : SVE2_VG4_Multi_Single_Single_Intrinsic;
  def int_aarch64_sve_uclamp_single_x4 : SVE2_VG4_Multi_Single_Single_Intrinsic;
  def int_aarch64_sve_fclamp_single_x4 : SVE2_VG4_Multi_Single_Single_Intrinsic;
  def int_aarch64_sve_bfclamp_single_x4 : SVE2_VG4_Multi_Single_Single_Intrinsic;

  //
  // Multi-vector add/sub and accumulate into ZA
  //
  foreach intr = ["add", "sub"] in {
    foreach za = ["za16","za32", "za64"] in {
      def int_aarch64_sme_ # intr # _ # za # _vg1x2 : SME2_ZA_Write_VG2_Intrinsic;
      def int_aarch64_sme_ # intr # _ # za # _vg1x4 : SME2_ZA_Write_VG4_Intrinsic;
    }
  }

  //
  // Move multi-vectors to/from ZA
  //

  def int_aarch64_sme_read_hor_vg2   : SME2_Matrix_TileVector_Read_VG2_Intrinsic;
  def int_aarch64_sme_read_hor_vg4   : SME2_Matrix_TileVector_Read_VG4_Intrinsic;

  def int_aarch64_sme_read_ver_vg2   : SME2_Matrix_TileVector_Read_VG2_Intrinsic;
  def int_aarch64_sme_read_ver_vg4   : SME2_Matrix_TileVector_Read_VG4_Intrinsic;

  def int_aarch64_sme_read_vg1x2 : SME2_ZA_ArrayVector_Read_VG2_Intrinsic;
  def int_aarch64_sme_read_vg1x4 : SME2_ZA_ArrayVector_Read_VG4_Intrinsic;

  def int_aarch64_sme_write_hor_vg2 : SME2_Matrix_TileVector_Write_VG2_Intrinsic;
  def int_aarch64_sme_write_hor_vg4 : SME2_Matrix_TileVector_Write_VG4_Intrinsic;

  def int_aarch64_sme_write_ver_vg2 : SME2_Matrix_TileVector_Write_VG2_Intrinsic;
````
- **L3841 EN**: Declares TableGen def `int_aarch64_sve_sclamp_single_x4`.
  **L3841 CN**: 声明 TableGen def `int_aarch64_sve_sclamp_single_x4`。
- **L3842 EN**: Declares TableGen def `int_aarch64_sve_uclamp_single_x4`.
  **L3842 CN**: 声明 TableGen def `int_aarch64_sve_uclamp_single_x4`。
- **L3843 EN**: Declares TableGen def `int_aarch64_sve_fclamp_single_x4`.
  **L3843 CN**: 声明 TableGen def `int_aarch64_sve_fclamp_single_x4`。
- **L3844 EN**: Declares TableGen def `int_aarch64_sve_bfclamp_single_x4`.
  **L3844 CN**: 声明 TableGen def `int_aarch64_sve_bfclamp_single_x4`。
- **L3845 EN**: Blank line separating nearby declarations or logic blocks.
  **L3845 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3846 EN**: Separator comment used for visual grouping.
  **L3846 CN**: 用于视觉分组的分隔注释。
- **L3847 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector add/sub and accumulate into ZA`.
  **L3847 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector add/sub and accumulate into ZA`。
- **L3848 EN**: Separator comment used for visual grouping.
  **L3848 CN**: 用于视觉分组的分隔注释。
- **L3849 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3849 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3850 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3850 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3851 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3851 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3852 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3852 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3853 EN**: Closes the current lexical scope or compound statement.
  **L3853 CN**: 结束当前词法作用域或复合语句块。
- **L3854 EN**: Closes the current lexical scope or compound statement.
  **L3854 CN**: 结束当前词法作用域或复合语句块。
- **L3855 EN**: Blank line separating nearby declarations or logic blocks.
  **L3855 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3856 EN**: Separator comment used for visual grouping.
  **L3856 CN**: 用于视觉分组的分隔注释。
- **L3857 EN**: Comment explains nearby logic, invariants, or intent: `Move multi-vectors to/from ZA`.
  **L3857 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Move multi-vectors to/from ZA`。
- **L3858 EN**: Separator comment used for visual grouping.
  **L3858 CN**: 用于视觉分组的分隔注释。
- **L3859 EN**: Blank line separating nearby declarations or logic blocks.
  **L3859 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3860 EN**: Declares TableGen def `int_aarch64_sme_read_hor_vg2`.
  **L3860 CN**: 声明 TableGen def `int_aarch64_sme_read_hor_vg2`。
- **L3861 EN**: Declares TableGen def `int_aarch64_sme_read_hor_vg4`.
  **L3861 CN**: 声明 TableGen def `int_aarch64_sme_read_hor_vg4`。
- **L3862 EN**: Blank line separating nearby declarations or logic blocks.
  **L3862 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3863 EN**: Declares TableGen def `int_aarch64_sme_read_ver_vg2`.
  **L3863 CN**: 声明 TableGen def `int_aarch64_sme_read_ver_vg2`。
- **L3864 EN**: Declares TableGen def `int_aarch64_sme_read_ver_vg4`.
  **L3864 CN**: 声明 TableGen def `int_aarch64_sme_read_ver_vg4`。
- **L3865 EN**: Blank line separating nearby declarations or logic blocks.
  **L3865 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3866 EN**: Declares TableGen def `int_aarch64_sme_read_vg1x2`.
  **L3866 CN**: 声明 TableGen def `int_aarch64_sme_read_vg1x2`。
- **L3867 EN**: Declares TableGen def `int_aarch64_sme_read_vg1x4`.
  **L3867 CN**: 声明 TableGen def `int_aarch64_sme_read_vg1x4`。
- **L3868 EN**: Blank line separating nearby declarations or logic blocks.
  **L3868 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3869 EN**: Declares TableGen def `int_aarch64_sme_write_hor_vg2`.
  **L3869 CN**: 声明 TableGen def `int_aarch64_sme_write_hor_vg2`。
- **L3870 EN**: Declares TableGen def `int_aarch64_sme_write_hor_vg4`.
  **L3870 CN**: 声明 TableGen def `int_aarch64_sme_write_hor_vg4`。
- **L3871 EN**: Blank line separating nearby declarations or logic blocks.
  **L3871 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3872 EN**: Declares TableGen def `int_aarch64_sme_write_ver_vg2`.
  **L3872 CN**: 声明 TableGen def `int_aarch64_sme_write_ver_vg2`。

### Lines 3873-3904

````tablegen
  def int_aarch64_sme_write_ver_vg4 : SME2_Matrix_TileVector_Write_VG4_Intrinsic;

  def int_aarch64_sme_write_vg1x2 : SME2_ZA_ArrayVector_Write_VG2_Intrinsic;
  def int_aarch64_sme_write_vg1x4 : SME2_ZA_ArrayVector_Write_VG4_Intrinsic;

  //
  // Multi-Single Vector add
  //
  def int_aarch64_sve_add_single_x2 : SVE2_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sve_add_single_x4 : SVE2_VG4_Multi_Single_Intrinsic;

  // 2-way and 4-way multi-vector signed/unsigned integer dot-product
  foreach ty = ["s", "u"] in {
    foreach sz = ["za32", "za64"] in {
      def int_aarch64_sme_ # ty # dot_single_ # sz # _vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
      def int_aarch64_sme_ # ty # dot_single_ # sz # _vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

      def int_aarch64_sme_ # ty # dot_ # sz # _vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
      def int_aarch64_sme_ # ty # dot_ # sz # _vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;

      def int_aarch64_sme_ # ty # dot_lane_ # sz # _vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
      def int_aarch64_sme_ # ty # dot_lane_ # sz # _vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
    }
  }

  foreach ty = ["su", "us"] in {
    def int_aarch64_sme_ # ty # dot_single_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
    def int_aarch64_sme_ # ty # dot_single_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

    def int_aarch64_sme_ # ty # dot_lane_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
    def int_aarch64_sme_ # ty # dot_lane_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;
  }
````
- **L3873 EN**: Declares TableGen def `int_aarch64_sme_write_ver_vg4`.
  **L3873 CN**: 声明 TableGen def `int_aarch64_sme_write_ver_vg4`。
- **L3874 EN**: Blank line separating nearby declarations or logic blocks.
  **L3874 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3875 EN**: Declares TableGen def `int_aarch64_sme_write_vg1x2`.
  **L3875 CN**: 声明 TableGen def `int_aarch64_sme_write_vg1x2`。
- **L3876 EN**: Declares TableGen def `int_aarch64_sme_write_vg1x4`.
  **L3876 CN**: 声明 TableGen def `int_aarch64_sme_write_vg1x4`。
- **L3877 EN**: Blank line separating nearby declarations or logic blocks.
  **L3877 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3878 EN**: Separator comment used for visual grouping.
  **L3878 CN**: 用于视觉分组的分隔注释。
- **L3879 EN**: Comment explains nearby logic, invariants, or intent: `Multi-Single Vector add`.
  **L3879 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-Single Vector add`。
- **L3880 EN**: Separator comment used for visual grouping.
  **L3880 CN**: 用于视觉分组的分隔注释。
- **L3881 EN**: Declares TableGen def `int_aarch64_sve_add_single_x2`.
  **L3881 CN**: 声明 TableGen def `int_aarch64_sve_add_single_x2`。
- **L3882 EN**: Declares TableGen def `int_aarch64_sve_add_single_x4`.
  **L3882 CN**: 声明 TableGen def `int_aarch64_sve_add_single_x4`。
- **L3883 EN**: Blank line separating nearby declarations or logic blocks.
  **L3883 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3884 EN**: Comment explains nearby logic, invariants, or intent: `2-way and 4-way multi-vector signed/unsigned integer dot-product`.
  **L3884 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2-way and 4-way multi-vector signed/unsigned integer dot-product`。
- **L3885 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3885 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3886 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3886 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3887 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3887 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3888 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3888 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3889 EN**: Blank line separating nearby declarations or logic blocks.
  **L3889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3890 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3890 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3891 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3891 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3892 EN**: Blank line separating nearby declarations or logic blocks.
  **L3892 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3893 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3893 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3894 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3894 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3895 EN**: Closes the current lexical scope or compound statement.
  **L3895 CN**: 结束当前词法作用域或复合语句块。
- **L3896 EN**: Closes the current lexical scope or compound statement.
  **L3896 CN**: 结束当前词法作用域或复合语句块。
- **L3897 EN**: Blank line separating nearby declarations or logic blocks.
  **L3897 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3898 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3898 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3899 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3899 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3900 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3900 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3901 EN**: Blank line separating nearby declarations or logic blocks.
  **L3901 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3902 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3902 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3903 EN**: Declares TableGen def `int_aarch64_sme_`.
  **L3903 CN**: 声明 TableGen def `int_aarch64_sme_`。
- **L3904 EN**: Closes the current lexical scope or compound statement.
  **L3904 CN**: 结束当前词法作用域或复合语句块。

### Lines 3905-3936

````tablegen

  def int_aarch64_sme_usdot_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sme_usdot_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;

  // Multi-vector half-precision or bfloat floating-point dot-product
  def int_aarch64_sme_fdot_single_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Single_Intrinsic;
  def int_aarch64_sme_fdot_single_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Single_Intrinsic;

  def int_aarch64_sme_fdot_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Multi_Intrinsic;
  def int_aarch64_sme_fdot_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Multi_Intrinsic;

  def int_aarch64_sme_fdot_lane_za32_vg1x2 : SME2_Matrix_ArrayVector_VG2_Multi_Index_Intrinsic;
  def int_aarch64_sme_fdot_lane_za32_vg1x4 : SME2_Matrix_ArrayVector_VG4_Multi_Index_Intrinsic;

  // Multi-vector zip and unzips
  def int_aarch64_sve_zip_x2  : SVE2_VG2_ZipUzp_Intrinsic;
  def int_aarch64_sve_zipq_x2 : SVE2_VG2_ZipUzp_Intrinsic;
  def int_aarch64_sve_zip_x4  : SVE2_VG4_ZipUzp_Intrinsic;
  def int_aarch64_sve_zipq_x4 : SVE2_VG4_ZipUzp_Intrinsic;
  def int_aarch64_sve_uzp_x2  : SVE2_VG2_ZipUzp_Intrinsic;
  def int_aarch64_sve_uzpq_x2 : SVE2_VG2_ZipUzp_Intrinsic;
  def int_aarch64_sve_uzp_x4  : SVE2_VG4_ZipUzp_Intrinsic;
  def int_aarch64_sve_uzpq_x4 : SVE2_VG4_ZipUzp_Intrinsic;

  // Vector dot-products (2-way)
  def int_aarch64_sve_sdot_x2 : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
  def int_aarch64_sve_udot_x2 : SVE2_3VectorArg_Long_Intrinsic<[IntrSpeculatable]>;
  def int_aarch64_sve_fdot_x2 : SVE2_3VectorArg_Long_Intrinsic;
  def int_aarch64_sve_sdot_lane_x2 : SVE2_3VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;
  def int_aarch64_sve_udot_lane_x2 : SVE2_3VectorArgIndexed_Long_Intrinsic<[IntrSpeculatable]>;
  def int_aarch64_sve_fdot_lane_x2 : SVE2_3VectorArgIndexed_Long_Intrinsic;

````
- **L3905 EN**: Blank line separating nearby declarations or logic blocks.
  **L3905 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3906 EN**: Declares TableGen def `int_aarch64_sme_usdot_za32_vg1x2`.
  **L3906 CN**: 声明 TableGen def `int_aarch64_sme_usdot_za32_vg1x2`。
- **L3907 EN**: Declares TableGen def `int_aarch64_sme_usdot_za32_vg1x4`.
  **L3907 CN**: 声明 TableGen def `int_aarch64_sme_usdot_za32_vg1x4`。
- **L3908 EN**: Blank line separating nearby declarations or logic blocks.
  **L3908 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3909 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector half-precision or bfloat floating-point dot-product`.
  **L3909 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector half-precision or bfloat floating-point dot-product`。
- **L3910 EN**: Declares TableGen def `int_aarch64_sme_fdot_single_za32_vg1x2`.
  **L3910 CN**: 声明 TableGen def `int_aarch64_sme_fdot_single_za32_vg1x2`。
- **L3911 EN**: Declares TableGen def `int_aarch64_sme_fdot_single_za32_vg1x4`.
  **L3911 CN**: 声明 TableGen def `int_aarch64_sme_fdot_single_za32_vg1x4`。
- **L3912 EN**: Blank line separating nearby declarations or logic blocks.
  **L3912 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3913 EN**: Declares TableGen def `int_aarch64_sme_fdot_za32_vg1x2`.
  **L3913 CN**: 声明 TableGen def `int_aarch64_sme_fdot_za32_vg1x2`。
- **L3914 EN**: Declares TableGen def `int_aarch64_sme_fdot_za32_vg1x4`.
  **L3914 CN**: 声明 TableGen def `int_aarch64_sme_fdot_za32_vg1x4`。
- **L3915 EN**: Blank line separating nearby declarations or logic blocks.
  **L3915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3916 EN**: Declares TableGen def `int_aarch64_sme_fdot_lane_za32_vg1x2`.
  **L3916 CN**: 声明 TableGen def `int_aarch64_sme_fdot_lane_za32_vg1x2`。
- **L3917 EN**: Declares TableGen def `int_aarch64_sme_fdot_lane_za32_vg1x4`.
  **L3917 CN**: 声明 TableGen def `int_aarch64_sme_fdot_lane_za32_vg1x4`。
- **L3918 EN**: Blank line separating nearby declarations or logic blocks.
  **L3918 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3919 EN**: Comment explains nearby logic, invariants, or intent: `Multi-vector zip and unzips`.
  **L3919 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-vector zip and unzips`。
- **L3920 EN**: Declares TableGen def `int_aarch64_sve_zip_x2`.
  **L3920 CN**: 声明 TableGen def `int_aarch64_sve_zip_x2`。
- **L3921 EN**: Declares TableGen def `int_aarch64_sve_zipq_x2`.
  **L3921 CN**: 声明 TableGen def `int_aarch64_sve_zipq_x2`。
- **L3922 EN**: Declares TableGen def `int_aarch64_sve_zip_x4`.
  **L3922 CN**: 声明 TableGen def `int_aarch64_sve_zip_x4`。
- **L3923 EN**: Declares TableGen def `int_aarch64_sve_zipq_x4`.
  **L3923 CN**: 声明 TableGen def `int_aarch64_sve_zipq_x4`。
- **L3924 EN**: Declares TableGen def `int_aarch64_sve_uzp_x2`.
  **L3924 CN**: 声明 TableGen def `int_aarch64_sve_uzp_x2`。
- **L3925 EN**: Declares TableGen def `int_aarch64_sve_uzpq_x2`.
  **L3925 CN**: 声明 TableGen def `int_aarch64_sve_uzpq_x2`。
- **L3926 EN**: Declares TableGen def `int_aarch64_sve_uzp_x4`.
  **L3926 CN**: 声明 TableGen def `int_aarch64_sve_uzp_x4`。
- **L3927 EN**: Declares TableGen def `int_aarch64_sve_uzpq_x4`.
  **L3927 CN**: 声明 TableGen def `int_aarch64_sve_uzpq_x4`。
- **L3928 EN**: Blank line separating nearby declarations or logic blocks.
  **L3928 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3929 EN**: Comment explains nearby logic, invariants, or intent: `Vector dot-products (2-way)`.
  **L3929 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector dot-products (2-way)`。
- **L3930 EN**: Declares TableGen def `int_aarch64_sve_sdot_x2`.
  **L3930 CN**: 声明 TableGen def `int_aarch64_sve_sdot_x2`。
- **L3931 EN**: Declares TableGen def `int_aarch64_sve_udot_x2`.
  **L3931 CN**: 声明 TableGen def `int_aarch64_sve_udot_x2`。
- **L3932 EN**: Declares TableGen def `int_aarch64_sve_fdot_x2`.
  **L3932 CN**: 声明 TableGen def `int_aarch64_sve_fdot_x2`。
- **L3933 EN**: Declares TableGen def `int_aarch64_sve_sdot_lane_x2`.
  **L3933 CN**: 声明 TableGen def `int_aarch64_sve_sdot_lane_x2`。
- **L3934 EN**: Declares TableGen def `int_aarch64_sve_udot_lane_x2`.
  **L3934 CN**: 声明 TableGen def `int_aarch64_sve_udot_lane_x2`。
- **L3935 EN**: Declares TableGen def `int_aarch64_sve_fdot_lane_x2`.
  **L3935 CN**: 声明 TableGen def `int_aarch64_sve_fdot_lane_x2`。
- **L3936 EN**: Blank line separating nearby declarations or logic blocks.
  **L3936 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3937-3968

````tablegen
  //
  // Signed/unsigned multi-vector unpacks
  //
  def int_aarch64_sve_sunpk_x2 : SVE2_VG2_Unpk_Intrinsic;
  def int_aarch64_sve_uunpk_x2 : SVE2_VG2_Unpk_Intrinsic;
  def int_aarch64_sve_sunpk_x4 : SVE2_VG4_Unpk_Intrinsic;
  def int_aarch64_sve_uunpk_x4 : SVE2_VG4_Unpk_Intrinsic;

  // 2-way and 4-way vector selects
  def int_aarch64_sve_sel_x2  : SVE2_VG2_Sel_Intrinsic;
  def int_aarch64_sve_sel_x4  : SVE2_VG4_Sel_Intrinsic;

  class SME_LDR_STR_ZT_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;
  def int_aarch64_sme_ldr_zt : SME_LDR_STR_ZT_Intrinsic;
  def int_aarch64_sme_str_zt : SME_LDR_STR_ZT_Intrinsic;

  //
  //  Zero ZT0
  //
  def int_aarch64_sme_zero_zt : DefaultAttrsIntrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrWriteMem]>;

  //
  // Lookup table expand one register
  //
  def int_aarch64_sme_luti2_lane_zt
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],
                            [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;
  def int_aarch64_sme_luti4_lane_zt
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],
                            [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;

````
- **L3937 EN**: Separator comment used for visual grouping.
  **L3937 CN**: 用于视觉分组的分隔注释。
- **L3938 EN**: Comment explains nearby logic, invariants, or intent: `Signed/unsigned multi-vector unpacks`.
  **L3938 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Signed/unsigned multi-vector unpacks`。
- **L3939 EN**: Separator comment used for visual grouping.
  **L3939 CN**: 用于视觉分组的分隔注释。
- **L3940 EN**: Declares TableGen def `int_aarch64_sve_sunpk_x2`.
  **L3940 CN**: 声明 TableGen def `int_aarch64_sve_sunpk_x2`。
- **L3941 EN**: Declares TableGen def `int_aarch64_sve_uunpk_x2`.
  **L3941 CN**: 声明 TableGen def `int_aarch64_sve_uunpk_x2`。
- **L3942 EN**: Declares TableGen def `int_aarch64_sve_sunpk_x4`.
  **L3942 CN**: 声明 TableGen def `int_aarch64_sve_sunpk_x4`。
- **L3943 EN**: Declares TableGen def `int_aarch64_sve_uunpk_x4`.
  **L3943 CN**: 声明 TableGen def `int_aarch64_sve_uunpk_x4`。
- **L3944 EN**: Blank line separating nearby declarations or logic blocks.
  **L3944 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3945 EN**: Comment explains nearby logic, invariants, or intent: `2-way and 4-way vector selects`.
  **L3945 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2-way and 4-way vector selects`。
- **L3946 EN**: Declares TableGen def `int_aarch64_sve_sel_x2`.
  **L3946 CN**: 声明 TableGen def `int_aarch64_sve_sel_x2`。
- **L3947 EN**: Declares TableGen def `int_aarch64_sve_sel_x4`.
  **L3947 CN**: 声明 TableGen def `int_aarch64_sve_sel_x4`。
- **L3948 EN**: Blank line separating nearby declarations or logic blocks.
  **L3948 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3949 EN**: Declares class `SME_LDR_STR_ZT_Intrinsic`.
  **L3949 CN**: 声明 class `SME_LDR_STR_ZT_Intrinsic`。
- **L3950 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`.
  **L3950 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`。
- **L3951 EN**: Declares TableGen def `int_aarch64_sme_ldr_zt`.
  **L3951 CN**: 声明 TableGen def `int_aarch64_sme_ldr_zt`。
- **L3952 EN**: Declares TableGen def `int_aarch64_sme_str_zt`.
  **L3952 CN**: 声明 TableGen def `int_aarch64_sme_str_zt`。
- **L3953 EN**: Blank line separating nearby declarations or logic blocks.
  **L3953 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3954 EN**: Separator comment used for visual grouping.
  **L3954 CN**: 用于视觉分组的分隔注释。
- **L3955 EN**: Comment explains nearby logic, invariants, or intent: `Zero ZT0`.
  **L3955 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Zero ZT0`。
- **L3956 EN**: Separator comment used for visual grouping.
  **L3956 CN**: 用于视觉分组的分隔注释。
- **L3957 EN**: Declares TableGen def `int_aarch64_sme_zero_zt`.
  **L3957 CN**: 声明 TableGen def `int_aarch64_sme_zero_zt`。
- **L3958 EN**: Blank line separating nearby declarations or logic blocks.
  **L3958 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3959 EN**: Separator comment used for visual grouping.
  **L3959 CN**: 用于视觉分组的分隔注释。
- **L3960 EN**: Comment explains nearby logic, invariants, or intent: `Lookup table expand one register`.
  **L3960 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lookup table expand one register`。
- **L3961 EN**: Separator comment used for visual grouping.
  **L3961 CN**: 用于视觉分组的分隔注释。
- **L3962 EN**: Declares TableGen def `int_aarch64_sme_luti2_lane_zt`.
  **L3962 CN**: 声明 TableGen def `int_aarch64_sme_luti2_lane_zt`。
- **L3963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L3963 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L3964 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`.
  **L3964 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`。
- **L3965 EN**: Declares TableGen def `int_aarch64_sme_luti4_lane_zt`.
  **L3965 CN**: 声明 TableGen def `int_aarch64_sme_luti4_lane_zt`。
- **L3966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L3966 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L3967 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`.
  **L3967 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`。
- **L3968 EN**: Blank line separating nearby declarations or logic blocks.
  **L3968 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3969-4000

````tablegen
  // Lookup table expand two registers
  //
  def int_aarch64_sme_luti2_lane_zt_x2
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],
                            [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;
  def int_aarch64_sme_luti4_lane_zt_x2
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],
                            [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;

  //
  // Lookup table expand four registers
  //
  def int_aarch64_sme_luti2_lane_zt_x4
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                            [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],
                            [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;
  def int_aarch64_sme_luti4_lane_zt_x4
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                            [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],
                            [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;

  def int_aarch64_sme_luti4_zt_x4
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                            [llvm_i32_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],
                            [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrReadMem]>;


  //
  // Register scaling
  //
  def int_aarch64_sme_fp8_scale_single_x2
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
````
- **L3969 EN**: Comment explains nearby logic, invariants, or intent: `Lookup table expand two registers`.
  **L3969 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lookup table expand two registers`。
- **L3970 EN**: Separator comment used for visual grouping.
  **L3970 CN**: 用于视觉分组的分隔注释。
- **L3971 EN**: Declares TableGen def `int_aarch64_sme_luti2_lane_zt_x2`.
  **L3971 CN**: 声明 TableGen def `int_aarch64_sme_luti2_lane_zt_x2`。
- **L3972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L3972 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L3973 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`.
  **L3973 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`。
- **L3974 EN**: Declares TableGen def `int_aarch64_sme_luti4_lane_zt_x2`.
  **L3974 CN**: 声明 TableGen def `int_aarch64_sme_luti4_lane_zt_x2`。
- **L3975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L3975 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L3976 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`.
  **L3976 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`。
- **L3977 EN**: Blank line separating nearby declarations or logic blocks.
  **L3977 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3978 EN**: Separator comment used for visual grouping.
  **L3978 CN**: 用于视觉分组的分隔注释。
- **L3979 EN**: Comment explains nearby logic, invariants, or intent: `Lookup table expand four registers`.
  **L3979 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Lookup table expand four registers`。
- **L3980 EN**: Separator comment used for visual grouping.
  **L3980 CN**: 用于视觉分组的分隔注释。
- **L3981 EN**: Declares TableGen def `int_aarch64_sme_luti2_lane_zt_x4`.
  **L3981 CN**: 声明 TableGen def `int_aarch64_sme_luti2_lane_zt_x4`。
- **L3982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3982 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L3983 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L3984 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`.
  **L3984 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`。
- **L3985 EN**: Declares TableGen def `int_aarch64_sme_luti4_lane_zt_x4`.
  **L3985 CN**: 声明 TableGen def `int_aarch64_sme_luti4_lane_zt_x4`。
- **L3986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3986 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L3987 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L3988 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`.
  **L3988 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<2>>, IntrInaccessibleMemOnly, IntrReadMem]>;`。
- **L3989 EN**: Blank line separating nearby declarations or logic blocks.
  **L3989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3990 EN**: Declares TableGen def `int_aarch64_sme_luti4_zt_x4`.
  **L3990 CN**: 声明 TableGen def `int_aarch64_sme_luti4_zt_x4`。
- **L3991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L3991 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L3992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L3992 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L3993 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrReadMem]>;`.
  **L3993 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly, IntrReadMem]>;`。
- **L3994 EN**: Blank line separating nearby declarations or logic blocks.
  **L3994 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3995 EN**: Blank line separating nearby declarations or logic blocks.
  **L3995 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3996 EN**: Separator comment used for visual grouping.
  **L3996 CN**: 用于视觉分组的分隔注释。
- **L3997 EN**: Comment explains nearby logic, invariants, or intent: `Register scaling`.
  **L3997 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Register scaling`。
- **L3998 EN**: Separator comment used for visual grouping.
  **L3998 CN**: 用于视觉分组的分隔注释。
- **L3999 EN**: Declares TableGen def `int_aarch64_sme_fp8_scale_single_x2`.
  **L3999 CN**: 声明 TableGen def `int_aarch64_sme_fp8_scale_single_x2`。
- **L4000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L4000 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。

### Lines 4001-4032

````tablegen
                            [LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>],
                            [IntrNoMem]>;

  def int_aarch64_sme_fp8_scale_single_x4
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                            [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>],
                            [IntrNoMem]>;

  def int_aarch64_sme_fp8_scale_x2
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                            [LLVMMatchType<0>, LLVMMatchType<0>,
                             LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],
                            [IntrNoMem]>;

  def int_aarch64_sme_fp8_scale_x4
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
                            [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
                             LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],
                            [IntrNoMem]>;

}

// SVE2.1 - ZIPQ1, ZIPQ2, UZPQ1, UZPQ2
//
def int_aarch64_sve_zipq1 : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_zipq2 : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzpq1 : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_uzpq2 : AdvSIMD_2VectorArg_Intrinsic<[IntrSpeculatable]>;

// SVE2.1 - Programmable table lookup within each quadword vector segment
// (zeroing)/(merging)
//
````
- **L4001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>],`.
  **L4001 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>],`。
- **L4002 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L4002 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L4003 EN**: Blank line separating nearby declarations or logic blocks.
  **L4003 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4004 EN**: Declares TableGen def `int_aarch64_sme_fp8_scale_single_x4`.
  **L4004 CN**: 声明 TableGen def `int_aarch64_sme_fp8_scale_single_x4`。
- **L4005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L4005 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L4006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>],`.
  **L4006 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMVectorOfBitcastsToInt<0>],`。
- **L4007 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L4007 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L4008 EN**: Blank line separating nearby declarations or logic blocks.
  **L4008 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4009 EN**: Declares TableGen def `int_aarch64_sme_fp8_scale_x2`.
  **L4009 CN**: 声明 TableGen def `int_aarch64_sme_fp8_scale_x2`。
- **L4010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L4010 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L4011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L4011 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L4012 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`.
  **L4012 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`。
- **L4013 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L4013 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L4014 EN**: Blank line separating nearby declarations or logic blocks.
  **L4014 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4015 EN**: Declares TableGen def `int_aarch64_sme_fp8_scale_x4`.
  **L4015 CN**: 声明 TableGen def `int_aarch64_sme_fp8_scale_x4`。
- **L4016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L4016 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L4017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L4017 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L4018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`.
  **L4018 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>, LLVMVectorOfBitcastsToInt<0>],`。
- **L4019 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L4019 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L4020 EN**: Blank line separating nearby declarations or logic blocks.
  **L4020 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4021 EN**: Closes the current lexical scope or compound statement.
  **L4021 CN**: 结束当前词法作用域或复合语句块。
- **L4022 EN**: Blank line separating nearby declarations or logic blocks.
  **L4022 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4023 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - ZIPQ1, ZIPQ2, UZPQ1, UZPQ2`.
  **L4023 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - ZIPQ1, ZIPQ2, UZPQ1, UZPQ2`。
- **L4024 EN**: Separator comment used for visual grouping.
  **L4024 CN**: 用于视觉分组的分隔注释。
- **L4025 EN**: Declares TableGen def `int_aarch64_sve_zipq1`.
  **L4025 CN**: 声明 TableGen def `int_aarch64_sve_zipq1`。
- **L4026 EN**: Declares TableGen def `int_aarch64_sve_zipq2`.
  **L4026 CN**: 声明 TableGen def `int_aarch64_sve_zipq2`。
- **L4027 EN**: Declares TableGen def `int_aarch64_sve_uzpq1`.
  **L4027 CN**: 声明 TableGen def `int_aarch64_sve_uzpq1`。
- **L4028 EN**: Declares TableGen def `int_aarch64_sve_uzpq2`.
  **L4028 CN**: 声明 TableGen def `int_aarch64_sve_uzpq2`。
- **L4029 EN**: Blank line separating nearby declarations or logic blocks.
  **L4029 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4030 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - Programmable table lookup within each quadword vector segment`.
  **L4030 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - Programmable table lookup within each quadword vector segment`。
- **L4031 EN**: Comment explains nearby logic, invariants, or intent: `(zeroing)/(merging)`.
  **L4031 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(zeroing)/(merging)`。
- **L4032 EN**: Separator comment used for visual grouping.
  **L4032 CN**: 用于视觉分组的分隔注释。

### Lines 4033-4064

````tablegen
def int_aarch64_sve_tblq : AdvSIMD_SVE_TBL_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_tbxq : AdvSIMD_SVE2_TBX_Intrinsic<[IntrSpeculatable]>;

// SVE2.1 - Extract vector segment from each pair of quadword segments.
//
def int_aarch64_sve_extq : AdvSIMD_2VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;

//
// SVE2.1 - Move predicate to/from vector
//
def int_aarch64_sve_pmov_to_pred_lane : SVE2_1VectorArgIndexed_Pred_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_pmov_to_pred_lane_zero : SVE2_1VectorArg_Pred_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sve_pmov_to_vector_lane_merging : SVE2_Pred_1VectorArgIndexed_Intrinsic<[IntrSpeculatable]>;
def int_aarch64_sve_pmov_to_vector_lane_zeroing : SVE2_Pred_1VectorArg_Intrinsic<[IntrSpeculatable]>;

def int_aarch64_sme_mopa_nonwide : SME_OuterProduct_Intrinsic;
def int_aarch64_sme_mops_nonwide : SME_OuterProduct_Intrinsic;

// SVE2/SME2 - Floating point absolute maximum and minimum

def int_aarch64_sve_famax   : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_famax_u : AdvSIMD_Pred2VectorArg_Intrinsic;

def int_aarch64_sve_famin   : AdvSIMD_Pred2VectorArg_Intrinsic;
def int_aarch64_sve_famin_u : AdvSIMD_Pred2VectorArg_Intrinsic;
// Neon absolute maximum and minimum
def int_aarch64_neon_famax :  AdvSIMD_2VectorArg_Intrinsic;
def int_aarch64_neon_famin :  AdvSIMD_2VectorArg_Intrinsic;

//
````
- **L4033 EN**: Declares TableGen def `int_aarch64_sve_tblq`.
  **L4033 CN**: 声明 TableGen def `int_aarch64_sve_tblq`。
- **L4034 EN**: Declares TableGen def `int_aarch64_sve_tbxq`.
  **L4034 CN**: 声明 TableGen def `int_aarch64_sve_tbxq`。
- **L4035 EN**: Blank line separating nearby declarations or logic blocks.
  **L4035 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4036 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - Extract vector segment from each pair of quadword segments.`.
  **L4036 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - Extract vector segment from each pair of quadword segments.`。
- **L4037 EN**: Separator comment used for visual grouping.
  **L4037 CN**: 用于视觉分组的分隔注释。
- **L4038 EN**: Declares TableGen def `int_aarch64_sve_extq`.
  **L4038 CN**: 声明 TableGen def `int_aarch64_sve_extq`。
- **L4039 EN**: Blank line separating nearby declarations or logic blocks.
  **L4039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4040 EN**: Separator comment used for visual grouping.
  **L4040 CN**: 用于视觉分组的分隔注释。
- **L4041 EN**: Comment explains nearby logic, invariants, or intent: `SVE2.1 - Move predicate to/from vector`.
  **L4041 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2.1 - Move predicate to/from vector`。
- **L4042 EN**: Separator comment used for visual grouping.
  **L4042 CN**: 用于视觉分组的分隔注释。
- **L4043 EN**: Declares TableGen def `int_aarch64_sve_pmov_to_pred_lane`.
  **L4043 CN**: 声明 TableGen def `int_aarch64_sve_pmov_to_pred_lane`。
- **L4044 EN**: Blank line separating nearby declarations or logic blocks.
  **L4044 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4045 EN**: Declares TableGen def `int_aarch64_sve_pmov_to_pred_lane_zero`.
  **L4045 CN**: 声明 TableGen def `int_aarch64_sve_pmov_to_pred_lane_zero`。
- **L4046 EN**: Blank line separating nearby declarations or logic blocks.
  **L4046 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4047 EN**: Declares TableGen def `int_aarch64_sve_pmov_to_vector_lane_merging`.
  **L4047 CN**: 声明 TableGen def `int_aarch64_sve_pmov_to_vector_lane_merging`。
- **L4048 EN**: Declares TableGen def `int_aarch64_sve_pmov_to_vector_lane_zeroing`.
  **L4048 CN**: 声明 TableGen def `int_aarch64_sve_pmov_to_vector_lane_zeroing`。
- **L4049 EN**: Blank line separating nearby declarations or logic blocks.
  **L4049 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4050 EN**: Declares TableGen def `int_aarch64_sme_mopa_nonwide`.
  **L4050 CN**: 声明 TableGen def `int_aarch64_sme_mopa_nonwide`。
- **L4051 EN**: Declares TableGen def `int_aarch64_sme_mops_nonwide`.
  **L4051 CN**: 声明 TableGen def `int_aarch64_sme_mops_nonwide`。
- **L4052 EN**: Blank line separating nearby declarations or logic blocks.
  **L4052 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4053 EN**: Comment explains nearby logic, invariants, or intent: `SVE2/SME2 - Floating point absolute maximum and minimum`.
  **L4053 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE2/SME2 - Floating point absolute maximum and minimum`。
- **L4054 EN**: Blank line separating nearby declarations or logic blocks.
  **L4054 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4055 EN**: Declares TableGen def `int_aarch64_sve_famax`.
  **L4055 CN**: 声明 TableGen def `int_aarch64_sve_famax`。
- **L4056 EN**: Declares TableGen def `int_aarch64_sve_famax_u`.
  **L4056 CN**: 声明 TableGen def `int_aarch64_sve_famax_u`。
- **L4057 EN**: Blank line separating nearby declarations or logic blocks.
  **L4057 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4058 EN**: Declares TableGen def `int_aarch64_sve_famin`.
  **L4058 CN**: 声明 TableGen def `int_aarch64_sve_famin`。
- **L4059 EN**: Declares TableGen def `int_aarch64_sve_famin_u`.
  **L4059 CN**: 声明 TableGen def `int_aarch64_sve_famin_u`。
- **L4060 EN**: Comment explains nearby logic, invariants, or intent: `Neon absolute maximum and minimum`.
  **L4060 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Neon absolute maximum and minimum`。
- **L4061 EN**: Declares TableGen def `int_aarch64_neon_famax`.
  **L4061 CN**: 声明 TableGen def `int_aarch64_neon_famax`。
- **L4062 EN**: Declares TableGen def `int_aarch64_neon_famin`.
  **L4062 CN**: 声明 TableGen def `int_aarch64_neon_famin`。
- **L4063 EN**: Blank line separating nearby declarations or logic blocks.
  **L4063 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4064 EN**: Separator comment used for visual grouping.
  **L4064 CN**: 用于视觉分组的分隔注释。

### Lines 4065-4096

````tablegen
// FP8 Intrinsics
//
let TargetPrefix = "aarch64" in {

  // SVE Widening Conversions
  class SVE2_FP8_Cvt
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [llvm_nxv16i8_ty],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  def int_aarch64_sve_fp8_cvt1   : SVE2_FP8_Cvt;
  def int_aarch64_sve_fp8_cvt2   : SVE2_FP8_Cvt;
  def int_aarch64_sve_fp8_cvtlt1 : SVE2_FP8_Cvt;
  def int_aarch64_sve_fp8_cvtlt2 : SVE2_FP8_Cvt;

  // SVE Narrowing Conversions
  class SVE2_FP8_Narrow_Cvt
    : DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                            [llvm_anyvector_ty, LLVMMatchType<0>],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  def int_aarch64_sve_fp8_cvtn  : SVE2_FP8_Narrow_Cvt;
  def int_aarch64_sve_fp8_cvtnb : SVE2_FP8_Narrow_Cvt;

  def int_aarch64_sve_fp8_cvtnt
    : DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                            [llvm_nxv16i8_ty, llvm_anyvector_ty, LLVMMatchType<0>],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  // Dot product
  class SVE2_FP8_FMLA_FDOT
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
````
- **L4065 EN**: Comment explains nearby logic, invariants, or intent: `FP8 Intrinsics`.
  **L4065 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP8 Intrinsics`。
- **L4066 EN**: Separator comment used for visual grouping.
  **L4066 CN**: 用于视觉分组的分隔注释。
- **L4067 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L4067 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L4068 EN**: Blank line separating nearby declarations or logic blocks.
  **L4068 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4069 EN**: Comment explains nearby logic, invariants, or intent: `SVE Widening Conversions`.
  **L4069 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE Widening Conversions`。
- **L4070 EN**: Declares class `SVE2_FP8_Cvt`.
  **L4070 CN**: 声明 class `SVE2_FP8_Cvt`。
- **L4071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L4071 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L4072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty],`.
  **L4072 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty],`。
- **L4073 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L4073 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L4074 EN**: Blank line separating nearby declarations or logic blocks.
  **L4074 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4075 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvt1`.
  **L4075 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvt1`。
- **L4076 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvt2`.
  **L4076 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvt2`。
- **L4077 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtlt1`.
  **L4077 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtlt1`。
- **L4078 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtlt2`.
  **L4078 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtlt2`。
- **L4079 EN**: Blank line separating nearby declarations or logic blocks.
  **L4079 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4080 EN**: Comment explains nearby logic, invariants, or intent: `SVE Narrowing Conversions`.
  **L4080 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SVE Narrowing Conversions`。
- **L4081 EN**: Declares class `SVE2_FP8_Narrow_Cvt`.
  **L4081 CN**: 声明 class `SVE2_FP8_Narrow_Cvt`。
- **L4082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L4082 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L4083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L4083 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L4084 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L4084 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L4085 EN**: Blank line separating nearby declarations or logic blocks.
  **L4085 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4086 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtn`.
  **L4086 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtn`。
- **L4087 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtnb`.
  **L4087 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtnb`。
- **L4088 EN**: Blank line separating nearby declarations or logic blocks.
  **L4088 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4089 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtnt`.
  **L4089 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtnt`。
- **L4090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L4090 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L4091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L4091 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L4092 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L4092 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L4093 EN**: Blank line separating nearby declarations or logic blocks.
  **L4093 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4094 EN**: Comment explains nearby logic, invariants, or intent: `Dot product`.
  **L4094 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Dot product`。
- **L4095 EN**: Declares class `SVE2_FP8_FMLA_FDOT`.
  **L4095 CN**: 声明 class `SVE2_FP8_FMLA_FDOT`。
- **L4096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L4096 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。

### Lines 4097-4128

````tablegen
                            [LLVMMatchType<0>,
                             llvm_nxv16i8_ty, llvm_nxv16i8_ty],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  class SVE2_FP8_FMLA_FDOT_Lane
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                            [LLVMMatchType<0>,
                             llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],
                            [IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;

  def int_aarch64_sve_fp8_fdot      : SVE2_FP8_FMLA_FDOT;
  def int_aarch64_sve_fp8_fdot_lane : SVE2_FP8_FMLA_FDOT_Lane;

  // Fused multiply-add
  def int_aarch64_sve_fp8_fmlalb      : SVE2_FP8_FMLA_FDOT;
  def int_aarch64_sve_fp8_fmlalb_lane : SVE2_FP8_FMLA_FDOT_Lane;

  def int_aarch64_sve_fp8_fmlalt      : SVE2_FP8_FMLA_FDOT;
  def int_aarch64_sve_fp8_fmlalt_lane : SVE2_FP8_FMLA_FDOT_Lane;

  def int_aarch64_sve_fp8_fmlallbb      : SVE2_FP8_FMLA_FDOT;
  def int_aarch64_sve_fp8_fmlallbb_lane : SVE2_FP8_FMLA_FDOT_Lane;

  def int_aarch64_sve_fp8_fmlallbt      : SVE2_FP8_FMLA_FDOT;
  def int_aarch64_sve_fp8_fmlallbt_lane : SVE2_FP8_FMLA_FDOT_Lane;

  def int_aarch64_sve_fp8_fmlalltb      : SVE2_FP8_FMLA_FDOT;
  def int_aarch64_sve_fp8_fmlalltb_lane : SVE2_FP8_FMLA_FDOT_Lane;

  def int_aarch64_sve_fp8_fmlalltt      : SVE2_FP8_FMLA_FDOT;
  def int_aarch64_sve_fp8_fmlalltt_lane : SVE2_FP8_FMLA_FDOT_Lane;

````
- **L4097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L4097 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L4098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L4098 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L4099 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L4099 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L4100 EN**: Blank line separating nearby declarations or logic blocks.
  **L4100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4101 EN**: Declares class `SVE2_FP8_FMLA_FDOT_Lane`.
  **L4101 CN**: 声明 class `SVE2_FP8_FMLA_FDOT_Lane`。
- **L4102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L4102 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L4103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>,`.
  **L4103 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>,`。
- **L4104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L4104 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L4105 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`.
  **L4105 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`。
- **L4106 EN**: Blank line separating nearby declarations or logic blocks.
  **L4106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4107 EN**: Declares TableGen def `int_aarch64_sve_fp8_fdot`.
  **L4107 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fdot`。
- **L4108 EN**: Declares TableGen def `int_aarch64_sve_fp8_fdot_lane`.
  **L4108 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fdot_lane`。
- **L4109 EN**: Blank line separating nearby declarations or logic blocks.
  **L4109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4110 EN**: Comment explains nearby logic, invariants, or intent: `Fused multiply-add`.
  **L4110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Fused multiply-add`。
- **L4111 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalb`.
  **L4111 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalb`。
- **L4112 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalb_lane`.
  **L4112 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalb_lane`。
- **L4113 EN**: Blank line separating nearby declarations or logic blocks.
  **L4113 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4114 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalt`.
  **L4114 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalt`。
- **L4115 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalt_lane`.
  **L4115 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalt_lane`。
- **L4116 EN**: Blank line separating nearby declarations or logic blocks.
  **L4116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4117 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlallbb`.
  **L4117 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlallbb`。
- **L4118 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlallbb_lane`.
  **L4118 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlallbb_lane`。
- **L4119 EN**: Blank line separating nearby declarations or logic blocks.
  **L4119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4120 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlallbt`.
  **L4120 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlallbt`。
- **L4121 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlallbt_lane`.
  **L4121 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlallbt_lane`。
- **L4122 EN**: Blank line separating nearby declarations or logic blocks.
  **L4122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4123 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalltb`.
  **L4123 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalltb`。
- **L4124 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalltb_lane`.
  **L4124 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalltb_lane`。
- **L4125 EN**: Blank line separating nearby declarations or logic blocks.
  **L4125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4126 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalltt`.
  **L4126 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalltt`。
- **L4127 EN**: Declares TableGen def `int_aarch64_sve_fp8_fmlalltt_lane`.
  **L4127 CN**: 声明 TableGen def `int_aarch64_sve_fp8_fmlalltt_lane`。
- **L4128 EN**: Blank line separating nearby declarations or logic blocks.
  **L4128 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 4129-4160

````tablegen
  class SVE2_FP8_CVT_X2_Single_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],
                            [llvm_nxv16i8_ty],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  class SVE2_FP8_CVT_Single_X4_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                            [llvm_nxv4f32_ty, llvm_nxv4f32_ty, llvm_nxv4f32_ty, llvm_nxv4f32_ty],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  class SME_FP8_OuterProduct_Intrinsic
      : DefaultAttrsIntrinsic<[],
          [llvm_i32_ty,
          llvm_nxv16i1_ty, llvm_nxv16i1_ty,
          llvm_nxv16i8_ty, llvm_nxv16i8_ty],
          [ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly]>;

  class SME_FP8_ZA_LANE_VGx1_Intrinsic
   : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                               llvm_nxv16i8_ty,
                               llvm_nxv16i8_ty,
                               llvm_i32_ty],
                          [IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;

  class SME_FP8_ZA_LANE_VGx2_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                                llvm_nxv16i8_ty, llvm_nxv16i8_ty,
                                llvm_nxv16i8_ty,
                                llvm_i32_ty],
                            [IntrInaccessibleMemOnly, ImmArg<ArgIndex<4>>]>;

  class SME_FP8_ZA_LANE_VGx4_Intrinsic
````
- **L4129 EN**: Declares class `SVE2_FP8_CVT_X2_Single_Intrinsic`.
  **L4129 CN**: 声明 class `SVE2_FP8_CVT_X2_Single_Intrinsic`。
- **L4130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L4130 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L4131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty],`.
  **L4131 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty],`。
- **L4132 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L4132 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L4133 EN**: Blank line separating nearby declarations or logic blocks.
  **L4133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4134 EN**: Declares class `SVE2_FP8_CVT_Single_X4_Intrinsic`.
  **L4134 CN**: 声明 class `SVE2_FP8_CVT_Single_X4_Intrinsic`。
- **L4135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L4135 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L4136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv4f32_ty, llvm_nxv4f32_ty, llvm_nxv4f32_ty, llvm_nxv4f32_ty],`.
  **L4136 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv4f32_ty, llvm_nxv4f32_ty, llvm_nxv4f32_ty, llvm_nxv4f32_ty],`。
- **L4137 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L4137 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L4138 EN**: Blank line separating nearby declarations or logic blocks.
  **L4138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4139 EN**: Declares class `SME_FP8_OuterProduct_Intrinsic`.
  **L4139 CN**: 声明 class `SME_FP8_OuterProduct_Intrinsic`。
- **L4140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L4140 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L4141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,`.
  **L4141 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,`。
- **L4142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i1_ty, llvm_nxv16i1_ty,`.
  **L4142 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i1_ty, llvm_nxv16i1_ty,`。
- **L4143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L4143 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L4144 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly]>;`.
  **L4144 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<0>>, IntrInaccessibleMemOnly]>;`。
- **L4145 EN**: Blank line separating nearby declarations or logic blocks.
  **L4145 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4146 EN**: Declares class `SME_FP8_ZA_LANE_VGx1_Intrinsic`.
  **L4146 CN**: 声明 class `SME_FP8_ZA_LANE_VGx1_Intrinsic`。
- **L4147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4147 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L4148 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L4149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L4149 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L4150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L4150 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L4151 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`.
  **L4151 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<3>>]>;`。
- **L4152 EN**: Blank line separating nearby declarations or logic blocks.
  **L4152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4153 EN**: Declares class `SME_FP8_ZA_LANE_VGx2_Intrinsic`.
  **L4153 CN**: 声明 class `SME_FP8_ZA_LANE_VGx2_Intrinsic`。
- **L4154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4154 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty,`.
  **L4155 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty,`。
- **L4156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L4156 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L4157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L4157 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L4158 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<4>>]>;`.
  **L4158 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<4>>]>;`。
- **L4159 EN**: Blank line separating nearby declarations or logic blocks.
  **L4159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4160 EN**: Declares class `SME_FP8_ZA_LANE_VGx4_Intrinsic`.
  **L4160 CN**: 声明 class `SME_FP8_ZA_LANE_VGx4_Intrinsic`。

### Lines 4161-4192

````tablegen
   : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                                llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,
                                llvm_nxv16i8_ty,
                                llvm_i32_ty],
                            [IntrInaccessibleMemOnly, ImmArg<ArgIndex<6>>]>;
  class SME_FP8_ZA_VGx1_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                                llvm_nxv16i8_ty,
                                llvm_nxv16i8_ty],
                            [IntrInaccessibleMemOnly]>;

  class SME_FP8_ZA_SINGLE_VGx2_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                                llvm_nxv16i8_ty, llvm_nxv16i8_ty,
                                llvm_nxv16i8_ty],
                            [IntrInaccessibleMemOnly]>;

  class SME_FP8_ZA_SINGLE_VGx4_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                                llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,
                                llvm_nxv16i8_ty],
                              [IntrInaccessibleMemOnly]>;

  class SME_FP8_ZA_MULTI_VGx2_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                                 llvm_nxv16i8_ty, llvm_nxv16i8_ty,
                                 llvm_nxv16i8_ty, llvm_nxv16i8_ty],
                            [IntrInaccessibleMemOnly]>;

  class SME_FP8_ZA_MULTI_VGx4_Intrinsic
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty,
                                 llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,
````
- **L4161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4161 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`.
  **L4162 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`。
- **L4163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L4163 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L4164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L4164 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L4165 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<6>>]>;`.
  **L4165 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<6>>]>;`。
- **L4166 EN**: Declares class `SME_FP8_ZA_VGx1_Intrinsic`.
  **L4166 CN**: 声明 class `SME_FP8_ZA_VGx1_Intrinsic`。
- **L4167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4167 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty,`.
  **L4168 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty,`。
- **L4169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty],`.
  **L4169 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty],`。
- **L4170 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L4170 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L4171 EN**: Blank line separating nearby declarations or logic blocks.
  **L4171 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4172 EN**: Declares class `SME_FP8_ZA_SINGLE_VGx2_Intrinsic`.
  **L4172 CN**: 声明 class `SME_FP8_ZA_SINGLE_VGx2_Intrinsic`。
- **L4173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4173 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty,`.
  **L4174 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty,`。
- **L4175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty],`.
  **L4175 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty],`。
- **L4176 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L4176 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L4177 EN**: Blank line separating nearby declarations or logic blocks.
  **L4177 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4178 EN**: Declares class `SME_FP8_ZA_SINGLE_VGx4_Intrinsic`.
  **L4178 CN**: 声明 class `SME_FP8_ZA_SINGLE_VGx4_Intrinsic`。
- **L4179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4179 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`.
  **L4180 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`。
- **L4181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty],`.
  **L4181 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty],`。
- **L4182 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L4182 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L4183 EN**: Blank line separating nearby declarations or logic blocks.
  **L4183 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4184 EN**: Declares class `SME_FP8_ZA_MULTI_VGx2_Intrinsic`.
  **L4184 CN**: 声明 class `SME_FP8_ZA_MULTI_VGx2_Intrinsic`。
- **L4185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4185 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty,`.
  **L4186 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty,`。
- **L4187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L4187 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L4188 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L4188 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L4189 EN**: Blank line separating nearby declarations or logic blocks.
  **L4189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4190 EN**: Declares class `SME_FP8_ZA_MULTI_VGx4_Intrinsic`.
  **L4190 CN**: 声明 class `SME_FP8_ZA_MULTI_VGx4_Intrinsic`。
- **L4191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`.
  **L4191 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty,`。
- **L4192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`.
  **L4192 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`。

### Lines 4193-4224

````tablegen
                                 llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],
                            [IntrInaccessibleMemOnly]>;
  //
  // CVT from FP8 to half-precision/BFloat16 multi-vector
  //
  def int_aarch64_sve_fp8_cvt1_x2 : SVE2_FP8_CVT_X2_Single_Intrinsic;
  def int_aarch64_sve_fp8_cvt2_x2 : SVE2_FP8_CVT_X2_Single_Intrinsic;

  //
  // CVT from FP8 to deinterleaved half-precision/BFloat16 multi-vector
  //
  def int_aarch64_sve_fp8_cvtl1_x2 : SVE2_FP8_CVT_X2_Single_Intrinsic;
  def int_aarch64_sve_fp8_cvtl2_x2 : SVE2_FP8_CVT_X2_Single_Intrinsic;

  //
  // CVT to FP8 from half-precision/BFloat16/single-precision multi-vector
  //
  def int_aarch64_sve_fp8_cvt_x2
    : DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],
                            [llvm_anyvector_ty, LLVMMatchType<0>],
                            [IntrReadMem, IntrInaccessibleMemOnly]>;

  def int_aarch64_sve_fp8_cvt_x4  : SVE2_FP8_CVT_Single_X4_Intrinsic;
  def int_aarch64_sve_fp8_cvtn_x4 : SVE2_FP8_CVT_Single_X4_Intrinsic;

  // FP8 outer product
  def int_aarch64_sme_fp8_fmopa_za16 : SME_FP8_OuterProduct_Intrinsic;
  def int_aarch64_sme_fp8_fmopa_za32 : SME_FP8_OuterProduct_Intrinsic;

  //
  // ZA multiply-add
  //
````
- **L4193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L4193 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L4194 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly]>;`.
  **L4194 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly]>;`。
- **L4195 EN**: Separator comment used for visual grouping.
  **L4195 CN**: 用于视觉分组的分隔注释。
- **L4196 EN**: Comment explains nearby logic, invariants, or intent: `CVT from FP8 to half-precision/BFloat16 multi-vector`.
  **L4196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CVT from FP8 to half-precision/BFloat16 multi-vector`。
- **L4197 EN**: Separator comment used for visual grouping.
  **L4197 CN**: 用于视觉分组的分隔注释。
- **L4198 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvt1_x2`.
  **L4198 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvt1_x2`。
- **L4199 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvt2_x2`.
  **L4199 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvt2_x2`。
- **L4200 EN**: Blank line separating nearby declarations or logic blocks.
  **L4200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4201 EN**: Separator comment used for visual grouping.
  **L4201 CN**: 用于视觉分组的分隔注释。
- **L4202 EN**: Comment explains nearby logic, invariants, or intent: `CVT from FP8 to deinterleaved half-precision/BFloat16 multi-vector`.
  **L4202 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CVT from FP8 to deinterleaved half-precision/BFloat16 multi-vector`。
- **L4203 EN**: Separator comment used for visual grouping.
  **L4203 CN**: 用于视觉分组的分隔注释。
- **L4204 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtl1_x2`.
  **L4204 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtl1_x2`。
- **L4205 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtl2_x2`.
  **L4205 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtl2_x2`。
- **L4206 EN**: Blank line separating nearby declarations or logic blocks.
  **L4206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4207 EN**: Separator comment used for visual grouping.
  **L4207 CN**: 用于视觉分组的分隔注释。
- **L4208 EN**: Comment explains nearby logic, invariants, or intent: `CVT to FP8 from half-precision/BFloat16/single-precision multi-vector`.
  **L4208 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CVT to FP8 from half-precision/BFloat16/single-precision multi-vector`。
- **L4209 EN**: Separator comment used for visual grouping.
  **L4209 CN**: 用于视觉分组的分隔注释。
- **L4210 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvt_x2`.
  **L4210 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvt_x2`。
- **L4211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`.
  **L4211 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty],`。
- **L4212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L4212 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L4213 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrInaccessibleMemOnly]>;`.
  **L4213 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrInaccessibleMemOnly]>;`。
- **L4214 EN**: Blank line separating nearby declarations or logic blocks.
  **L4214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4215 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvt_x4`.
  **L4215 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvt_x4`。
- **L4216 EN**: Declares TableGen def `int_aarch64_sve_fp8_cvtn_x4`.
  **L4216 CN**: 声明 TableGen def `int_aarch64_sve_fp8_cvtn_x4`。
- **L4217 EN**: Blank line separating nearby declarations or logic blocks.
  **L4217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4218 EN**: Comment explains nearby logic, invariants, or intent: `FP8 outer product`.
  **L4218 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP8 outer product`。
- **L4219 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmopa_za16`.
  **L4219 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmopa_za16`。
- **L4220 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmopa_za32`.
  **L4220 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmopa_za32`。
- **L4221 EN**: Blank line separating nearby declarations or logic blocks.
  **L4221 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4222 EN**: Separator comment used for visual grouping.
  **L4222 CN**: 用于视觉分组的分隔注释。
- **L4223 EN**: Comment explains nearby logic, invariants, or intent: `ZA multiply-add`.
  **L4223 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ZA multiply-add`。
- **L4224 EN**: Separator comment used for visual grouping.
  **L4224 CN**: 用于视觉分组的分隔注释。

### Lines 4225-4256

````tablegen
  // Double-vector groups (F8F16)
  def int_aarch64_sme_fp8_fmlal_lane_za16_vg2x1 : SME_FP8_ZA_LANE_VGx1_Intrinsic;
  def int_aarch64_sme_fp8_fmlal_lane_za16_vg2x2 : SME_FP8_ZA_LANE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fmlal_lane_za16_vg2x4 : SME_FP8_ZA_LANE_VGx4_Intrinsic;
  // Single-Single
  def int_aarch64_sme_fp8_fmlal_za16_vg2x1 : SME_FP8_ZA_VGx1_Intrinsic;
  // Multi-Single
  def int_aarch64_sme_fp8_fmlal_single_za16_vg2x2 : SME_FP8_ZA_SINGLE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fmlal_single_za16_vg2x4 : SME_FP8_ZA_SINGLE_VGx4_Intrinsic;
  // Multi-Multi
  def int_aarch64_sme_fp8_fmlal_multi_za16_vg2x2 : SME_FP8_ZA_MULTI_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fmlal_multi_za16_vg2x4 : SME_FP8_ZA_MULTI_VGx4_Intrinsic;

  // Quad-vector groups (F8F32)
  def int_aarch64_sme_fp8_fmlall_lane_za32_vg4x1 : SME_FP8_ZA_LANE_VGx1_Intrinsic;
  def int_aarch64_sme_fp8_fmlall_lane_za32_vg4x2 : SME_FP8_ZA_LANE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fmlall_lane_za32_vg4x4 : SME_FP8_ZA_LANE_VGx4_Intrinsic;
  // Single-Single
  def int_aarch64_sme_fp8_fmlall_za32_vg4x1 : SME_FP8_ZA_VGx1_Intrinsic;
  // Multi-Single
  def int_aarch64_sme_fp8_fmlall_single_za32_vg4x2 : SME_FP8_ZA_SINGLE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fmlall_single_za32_vg4x4 : SME_FP8_ZA_SINGLE_VGx4_Intrinsic;
  // Multi-Multi
  def int_aarch64_sme_fp8_fmlall_multi_za32_vg4x2 : SME_FP8_ZA_MULTI_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fmlall_multi_za32_vg4x4 : SME_FP8_ZA_MULTI_VGx4_Intrinsic;

  //
  // FP8 FDOT intrinsics
  //
  // (indexed)
  def int_aarch64_sme_fp8_fdot_lane_za16_vg1x2 : SME_FP8_ZA_LANE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fdot_lane_za32_vg1x2 : SME_FP8_ZA_LANE_VGx2_Intrinsic;
````
- **L4225 EN**: Comment explains nearby logic, invariants, or intent: `Double-vector groups (F8F16)`.
  **L4225 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Double-vector groups (F8F16)`。
- **L4226 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_lane_za16_vg2x1`.
  **L4226 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_lane_za16_vg2x1`。
- **L4227 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_lane_za16_vg2x2`.
  **L4227 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_lane_za16_vg2x2`。
- **L4228 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_lane_za16_vg2x4`.
  **L4228 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_lane_za16_vg2x4`。
- **L4229 EN**: Comment explains nearby logic, invariants, or intent: `Single-Single`.
  **L4229 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single-Single`。
- **L4230 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_za16_vg2x1`.
  **L4230 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_za16_vg2x1`。
- **L4231 EN**: Comment explains nearby logic, invariants, or intent: `Multi-Single`.
  **L4231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-Single`。
- **L4232 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_single_za16_vg2x2`.
  **L4232 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_single_za16_vg2x2`。
- **L4233 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_single_za16_vg2x4`.
  **L4233 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_single_za16_vg2x4`。
- **L4234 EN**: Comment explains nearby logic, invariants, or intent: `Multi-Multi`.
  **L4234 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-Multi`。
- **L4235 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_multi_za16_vg2x2`.
  **L4235 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_multi_za16_vg2x2`。
- **L4236 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlal_multi_za16_vg2x4`.
  **L4236 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlal_multi_za16_vg2x4`。
- **L4237 EN**: Blank line separating nearby declarations or logic blocks.
  **L4237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4238 EN**: Comment explains nearby logic, invariants, or intent: `Quad-vector groups (F8F32)`.
  **L4238 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Quad-vector groups (F8F32)`。
- **L4239 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_lane_za32_vg4x1`.
  **L4239 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_lane_za32_vg4x1`。
- **L4240 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_lane_za32_vg4x2`.
  **L4240 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_lane_za32_vg4x2`。
- **L4241 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_lane_za32_vg4x4`.
  **L4241 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_lane_za32_vg4x4`。
- **L4242 EN**: Comment explains nearby logic, invariants, or intent: `Single-Single`.
  **L4242 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single-Single`。
- **L4243 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_za32_vg4x1`.
  **L4243 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_za32_vg4x1`。
- **L4244 EN**: Comment explains nearby logic, invariants, or intent: `Multi-Single`.
  **L4244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-Single`。
- **L4245 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_single_za32_vg4x2`.
  **L4245 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_single_za32_vg4x2`。
- **L4246 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_single_za32_vg4x4`.
  **L4246 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_single_za32_vg4x4`。
- **L4247 EN**: Comment explains nearby logic, invariants, or intent: `Multi-Multi`.
  **L4247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi-Multi`。
- **L4248 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_multi_za32_vg4x2`.
  **L4248 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_multi_za32_vg4x2`。
- **L4249 EN**: Declares TableGen def `int_aarch64_sme_fp8_fmlall_multi_za32_vg4x4`.
  **L4249 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fmlall_multi_za32_vg4x4`。
- **L4250 EN**: Blank line separating nearby declarations or logic blocks.
  **L4250 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4251 EN**: Separator comment used for visual grouping.
  **L4251 CN**: 用于视觉分组的分隔注释。
- **L4252 EN**: Comment explains nearby logic, invariants, or intent: `FP8 FDOT intrinsics`.
  **L4252 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP8 FDOT intrinsics`。
- **L4253 EN**: Separator comment used for visual grouping.
  **L4253 CN**: 用于视觉分组的分隔注释。
- **L4254 EN**: Comment explains nearby logic, invariants, or intent: `(indexed)`.
  **L4254 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(indexed)`。
- **L4255 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_lane_za16_vg1x2`.
  **L4255 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_lane_za16_vg1x2`。
- **L4256 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_lane_za32_vg1x2`.
  **L4256 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_lane_za32_vg1x2`。

### Lines 4257-4288

````tablegen

  def int_aarch64_sme_fp8_fdot_lane_za16_vg1x4 : SME_FP8_ZA_LANE_VGx4_Intrinsic;
  def int_aarch64_sme_fp8_fdot_lane_za32_vg1x4 : SME_FP8_ZA_LANE_VGx4_Intrinsic;
  // Single
  def int_aarch64_sme_fp8_fdot_single_za16_vg1x2 : SME_FP8_ZA_SINGLE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fdot_single_za32_vg1x2 : SME_FP8_ZA_SINGLE_VGx2_Intrinsic;

  def int_aarch64_sme_fp8_fdot_single_za16_vg1x4 : SME_FP8_ZA_SINGLE_VGx4_Intrinsic;
  def int_aarch64_sme_fp8_fdot_single_za32_vg1x4 : SME_FP8_ZA_SINGLE_VGx4_Intrinsic;
  // Multi
  def int_aarch64_sme_fp8_fdot_multi_za16_vg1x2 : SME_FP8_ZA_MULTI_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fdot_multi_za32_vg1x2 : SME_FP8_ZA_MULTI_VGx2_Intrinsic;

  def int_aarch64_sme_fp8_fdot_multi_za16_vg1x4 : SME_FP8_ZA_MULTI_VGx4_Intrinsic;
  def int_aarch64_sme_fp8_fdot_multi_za32_vg1x4 : SME_FP8_ZA_MULTI_VGx4_Intrinsic;

  // FVDOT
  def int_aarch64_sme_fp8_fvdot_lane_za16_vg1x2  : SME_FP8_ZA_LANE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fvdotb_lane_za32_vg1x4 : SME_FP8_ZA_LANE_VGx2_Intrinsic;
  def int_aarch64_sme_fp8_fvdott_lane_za32_vg1x4 : SME_FP8_ZA_LANE_VGx2_Intrinsic;
  
  // AES2
  class SVE2_Crypto_LANE_X2_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_nxv16i8_ty, llvm_nxv16i8_ty],
      [llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],
      [ImmArg<ArgIndex<3>>, IntrNoMem]>;
  class SVE2_Crypto_LANE_X4_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],
      [llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, 
       llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],
      [ImmArg<ArgIndex<5>>, IntrNoMem]>;

````
- **L4257 EN**: Blank line separating nearby declarations or logic blocks.
  **L4257 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4258 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_lane_za16_vg1x4`.
  **L4258 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_lane_za16_vg1x4`。
- **L4259 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_lane_za32_vg1x4`.
  **L4259 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_lane_za32_vg1x4`。
- **L4260 EN**: Comment explains nearby logic, invariants, or intent: `Single`.
  **L4260 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Single`。
- **L4261 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_single_za16_vg1x2`.
  **L4261 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_single_za16_vg1x2`。
- **L4262 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_single_za32_vg1x2`.
  **L4262 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_single_za32_vg1x2`。
- **L4263 EN**: Blank line separating nearby declarations or logic blocks.
  **L4263 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4264 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_single_za16_vg1x4`.
  **L4264 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_single_za16_vg1x4`。
- **L4265 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_single_za32_vg1x4`.
  **L4265 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_single_za32_vg1x4`。
- **L4266 EN**: Comment explains nearby logic, invariants, or intent: `Multi`.
  **L4266 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multi`。
- **L4267 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_multi_za16_vg1x2`.
  **L4267 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_multi_za16_vg1x2`。
- **L4268 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_multi_za32_vg1x2`.
  **L4268 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_multi_za32_vg1x2`。
- **L4269 EN**: Blank line separating nearby declarations or logic blocks.
  **L4269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4270 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_multi_za16_vg1x4`.
  **L4270 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_multi_za16_vg1x4`。
- **L4271 EN**: Declares TableGen def `int_aarch64_sme_fp8_fdot_multi_za32_vg1x4`.
  **L4271 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fdot_multi_za32_vg1x4`。
- **L4272 EN**: Blank line separating nearby declarations or logic blocks.
  **L4272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4273 EN**: Comment explains nearby logic, invariants, or intent: `FVDOT`.
  **L4273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FVDOT`。
- **L4274 EN**: Declares TableGen def `int_aarch64_sme_fp8_fvdot_lane_za16_vg1x2`.
  **L4274 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fvdot_lane_za16_vg1x2`。
- **L4275 EN**: Declares TableGen def `int_aarch64_sme_fp8_fvdotb_lane_za32_vg1x4`.
  **L4275 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fvdotb_lane_za32_vg1x4`。
- **L4276 EN**: Declares TableGen def `int_aarch64_sme_fp8_fvdott_lane_za32_vg1x4`.
  **L4276 CN**: 声明 TableGen def `int_aarch64_sme_fp8_fvdott_lane_za32_vg1x4`。
- **L4277 EN**: Blank line separating nearby declarations or logic blocks.
  **L4277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4278 EN**: Comment explains nearby logic, invariants, or intent: `AES2`.
  **L4278 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AES2`。
- **L4279 EN**: Declares class `SVE2_Crypto_LANE_X2_Intrinsic`.
  **L4279 CN**: 声明 class `SVE2_Crypto_LANE_X2_Intrinsic`。
- **L4280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L4280 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L4281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L4281 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L4282 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<3>>, IntrNoMem]>;`.
  **L4282 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<3>>, IntrNoMem]>;`。
- **L4283 EN**: Declares class `SVE2_Crypto_LANE_X4_Intrinsic`.
  **L4283 CN**: 声明 class `SVE2_Crypto_LANE_X4_Intrinsic`。
- **L4284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`.
  **L4284 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty],`。
- **L4285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`.
  **L4285 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_nxv16i8_ty,`。
- **L4286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],`.
  **L4286 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_nxv16i8_ty, llvm_nxv16i8_ty, llvm_i32_ty],`。
- **L4287 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<5>>, IntrNoMem]>;`.
  **L4287 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<5>>, IntrNoMem]>;`。
- **L4288 EN**: Blank line separating nearby declarations or logic blocks.
  **L4288 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 4289-4303

````tablegen
  def int_aarch64_sve_aesd_lane_x2 : SVE2_Crypto_LANE_X2_Intrinsic;
  def int_aarch64_sve_aesdimc_lane_x2 : SVE2_Crypto_LANE_X2_Intrinsic;
  def int_aarch64_sve_aese_lane_x2 : SVE2_Crypto_LANE_X2_Intrinsic;
  def int_aarch64_sve_aesemc_lane_x2 : SVE2_Crypto_LANE_X2_Intrinsic;

  def int_aarch64_sve_aesd_lane_x4 : SVE2_Crypto_LANE_X4_Intrinsic;
  def int_aarch64_sve_aesdimc_lane_x4 : SVE2_Crypto_LANE_X4_Intrinsic;
  def int_aarch64_sve_aese_lane_x4 : SVE2_Crypto_LANE_X4_Intrinsic;
  def int_aarch64_sve_aesemc_lane_x4 : SVE2_Crypto_LANE_X4_Intrinsic;

  def int_aarch64_sve_pmull_pair_x2 : DefaultAttrsIntrinsic<[llvm_nxv2i64_ty, llvm_nxv2i64_ty],
      [llvm_nxv2i64_ty, llvm_nxv2i64_ty], [IntrNoMem]>;
  def int_aarch64_sve_pmlal_pair_x2 : DefaultAttrsIntrinsic<[llvm_nxv2i64_ty, llvm_nxv2i64_ty],
      [llvm_nxv2i64_ty, llvm_nxv2i64_ty, llvm_nxv2i64_ty, llvm_nxv2i64_ty], [IntrNoMem]>;
}
````
- **L4289 EN**: Declares TableGen def `int_aarch64_sve_aesd_lane_x2`.
  **L4289 CN**: 声明 TableGen def `int_aarch64_sve_aesd_lane_x2`。
- **L4290 EN**: Declares TableGen def `int_aarch64_sve_aesdimc_lane_x2`.
  **L4290 CN**: 声明 TableGen def `int_aarch64_sve_aesdimc_lane_x2`。
- **L4291 EN**: Declares TableGen def `int_aarch64_sve_aese_lane_x2`.
  **L4291 CN**: 声明 TableGen def `int_aarch64_sve_aese_lane_x2`。
- **L4292 EN**: Declares TableGen def `int_aarch64_sve_aesemc_lane_x2`.
  **L4292 CN**: 声明 TableGen def `int_aarch64_sve_aesemc_lane_x2`。
- **L4293 EN**: Blank line separating nearby declarations or logic blocks.
  **L4293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4294 EN**: Declares TableGen def `int_aarch64_sve_aesd_lane_x4`.
  **L4294 CN**: 声明 TableGen def `int_aarch64_sve_aesd_lane_x4`。
- **L4295 EN**: Declares TableGen def `int_aarch64_sve_aesdimc_lane_x4`.
  **L4295 CN**: 声明 TableGen def `int_aarch64_sve_aesdimc_lane_x4`。
- **L4296 EN**: Declares TableGen def `int_aarch64_sve_aese_lane_x4`.
  **L4296 CN**: 声明 TableGen def `int_aarch64_sve_aese_lane_x4`。
- **L4297 EN**: Declares TableGen def `int_aarch64_sve_aesemc_lane_x4`.
  **L4297 CN**: 声明 TableGen def `int_aarch64_sve_aesemc_lane_x4`。
- **L4298 EN**: Blank line separating nearby declarations or logic blocks.
  **L4298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L4299 EN**: Declares TableGen def `int_aarch64_sve_pmull_pair_x2`.
  **L4299 CN**: 声明 TableGen def `int_aarch64_sve_pmull_pair_x2`。
- **L4300 EN**: Executes a standalone statement or declaration: `[llvm_nxv2i64_ty, llvm_nxv2i64_ty], [IntrNoMem]>;`.
  **L4300 CN**: 执行一条独立语句或声明：`[llvm_nxv2i64_ty, llvm_nxv2i64_ty], [IntrNoMem]>;`。
- **L4301 EN**: Declares TableGen def `int_aarch64_sve_pmlal_pair_x2`.
  **L4301 CN**: 声明 TableGen def `int_aarch64_sve_pmlal_pair_x2`。
- **L4302 EN**: Executes a standalone statement or declaration: `[llvm_nxv2i64_ty, llvm_nxv2i64_ty, llvm_nxv2i64_ty, llvm_nxv2i64_ty], [IntrNoMem]>;`.
  **L4302 CN**: 执行一条独立语句或声明：`[llvm_nxv2i64_ty, llvm_nxv2i64_ty, llvm_nxv2i64_ty, llvm_nxv2i64_ty], [IntrNoMem]>;`。
- **L4303 EN**: Closes the current lexical scope or compound statement.
  **L4303 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **ORC JIT runtime and remote execution / ORC JIT 运行时与远程执行**
- **Function-level IR management / 函数级 IR 管理**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
