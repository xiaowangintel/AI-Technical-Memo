# IntrinsicsARM.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsARM.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the ARM-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsARM` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//===- IntrinsicsARM.td - Defines ARM intrinsics -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the ARM-specific intrinsics.
//
//===----------------------------------------------------------------------===//


//===----------------------------------------------------------------------===//
// TLS

let TargetPrefix = "arm" in {  // All intrinsics start with "llvm.arm.".

// A space-consuming intrinsic primarily for testing ARMConstantIslands. The
// first argument is the number of bytes this "instruction" takes up, the second
// and return value are essentially chains, used to force ordering during ISel.
def int_arm_space : Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

// 16-bit multiplications
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the ARM-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the ARM-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L14 EN**: Banner comment marking a file or section boundary.
  **L14 CN**: 横幅注释，用于标记文件或章节边界。
- **L15 EN**: Comment explains nearby logic, invariants, or intent: `TLS`.
  **L15 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TLS`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L17 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, invariants, or intent: `A space-consuming intrinsic primarily for testing ARMConstantIslands. The`.
  **L19 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A space-consuming intrinsic primarily for testing ARMConstantIslands. The`。
- **L20 EN**: Comment explains nearby logic, invariants, or intent: `first argument is the number of bytes this "instruction" takes up, the second`.
  **L20 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`first argument is the number of bytes this "instruction" takes up, the second`。
- **L21 EN**: Comment explains nearby logic, invariants, or intent: `and return value are essentially chains, used to force ordering during ISel.`.
  **L21 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and return value are essentially chains, used to force ordering during ISel.`。
- **L22 EN**: Declares TableGen def `int_arm_space`.
  **L22 CN**: 声明 TableGen def `int_arm_space`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `16-bit multiplications`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`16-bit multiplications`。

### Lines 25-48

````tablegen
def int_arm_smulbb : ClangBuiltin<"__builtin_arm_smulbb">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smulbt : ClangBuiltin<"__builtin_arm_smulbt">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smultb : ClangBuiltin<"__builtin_arm_smultb">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smultt : ClangBuiltin<"__builtin_arm_smultt">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smulwb : ClangBuiltin<"__builtin_arm_smulwb">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smulwt : ClangBuiltin<"__builtin_arm_smulwt">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;

//===----------------------------------------------------------------------===//
// Saturating Arithmetic

def int_arm_qadd : ClangBuiltin<"__builtin_arm_qadd">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
````
- **L25 EN**: Declares TableGen def `int_arm_smulbb`.
  **L25 CN**: 声明 TableGen def `int_arm_smulbb`。
- **L26 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L26 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L27 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L27 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L28 EN**: Declares TableGen def `int_arm_smulbt`.
  **L28 CN**: 声明 TableGen def `int_arm_smulbt`。
- **L29 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L29 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L30 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L30 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L31 EN**: Declares TableGen def `int_arm_smultb`.
  **L31 CN**: 声明 TableGen def `int_arm_smultb`。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L33 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L33 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L34 EN**: Declares TableGen def `int_arm_smultt`.
  **L34 CN**: 声明 TableGen def `int_arm_smultt`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L36 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L36 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L37 EN**: Declares TableGen def `int_arm_smulwb`.
  **L37 CN**: 声明 TableGen def `int_arm_smulwb`。
- **L38 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L38 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L39 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L39 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L40 EN**: Declares TableGen def `int_arm_smulwt`.
  **L40 CN**: 声明 TableGen def `int_arm_smulwt`。
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L42 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L42 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Banner comment marking a file or section boundary.
  **L44 CN**: 横幅注释，用于标记文件或章节边界。
- **L45 EN**: Comment explains nearby logic, invariants, or intent: `Saturating Arithmetic`.
  **L45 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating Arithmetic`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Declares TableGen def `int_arm_qadd`.
  **L47 CN**: 声明 TableGen def `int_arm_qadd`。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。

### Lines 49-72

````tablegen
                          [Commutative, IntrNoMem]>;
def int_arm_qsub : ClangBuiltin<"__builtin_arm_qsub">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_ssat : ClangBuiltin<"__builtin_arm_ssat">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_usat : ClangBuiltin<"__builtin_arm_usat">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;

// Accumulating multiplications
def int_arm_smlabb : ClangBuiltin<"__builtin_arm_smlabb">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smlabt : ClangBuiltin<"__builtin_arm_smlabt">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smlatb : ClangBuiltin<"__builtin_arm_smlatb">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
````
- **L49 EN**: Executes a standalone statement or declaration: `[Commutative, IntrNoMem]>;`.
  **L49 CN**: 执行一条独立语句或声明：`[Commutative, IntrNoMem]>;`。
- **L50 EN**: Declares TableGen def `int_arm_qsub`.
  **L50 CN**: 声明 TableGen def `int_arm_qsub`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L52 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L52 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L53 EN**: Declares TableGen def `int_arm_ssat`.
  **L53 CN**: 声明 TableGen def `int_arm_ssat`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L55 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L55 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L56 EN**: Declares TableGen def `int_arm_usat`.
  **L56 CN**: 声明 TableGen def `int_arm_usat`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L58 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L58 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L60 EN**: Comment explains nearby logic, invariants, or intent: `Accumulating multiplications`.
  **L60 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Accumulating multiplications`。
- **L61 EN**: Declares TableGen def `int_arm_smlabb`.
  **L61 CN**: 声明 TableGen def `int_arm_smlabb`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L64 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L64 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L65 EN**: Declares TableGen def `int_arm_smlabt`.
  **L65 CN**: 声明 TableGen def `int_arm_smlabt`。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L68 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L68 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L69 EN**: Declares TableGen def `int_arm_smlatb`.
  **L69 CN**: 声明 TableGen def `int_arm_smlatb`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L71 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L71 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L72 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L72 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 73-96

````tablegen
def int_arm_smlatt : ClangBuiltin<"__builtin_arm_smlatt">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smlawb : ClangBuiltin<"__builtin_arm_smlawb">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smlawt : ClangBuiltin<"__builtin_arm_smlawt">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;

// Parallel 16-bit saturation
def int_arm_ssat16 : ClangBuiltin<"__builtin_arm_ssat16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_usat16 : ClangBuiltin<"__builtin_arm_usat16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;

// Packing and unpacking
def int_arm_sxtab16 : ClangBuiltin<"__builtin_arm_sxtab16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
````
- **L73 EN**: Declares TableGen def `int_arm_smlatt`.
  **L73 CN**: 声明 TableGen def `int_arm_smlatt`。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L75 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L75 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L76 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L76 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L77 EN**: Declares TableGen def `int_arm_smlawb`.
  **L77 CN**: 声明 TableGen def `int_arm_smlawb`。
- **L78 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L78 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L80 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L80 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L81 EN**: Declares TableGen def `int_arm_smlawt`.
  **L81 CN**: 声明 TableGen def `int_arm_smlawt`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L84 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L84 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L86 EN**: Comment explains nearby logic, invariants, or intent: `Parallel 16-bit saturation`.
  **L86 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parallel 16-bit saturation`。
- **L87 EN**: Declares TableGen def `int_arm_ssat16`.
  **L87 CN**: 声明 TableGen def `int_arm_ssat16`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L89 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L89 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L90 EN**: Declares TableGen def `int_arm_usat16`.
  **L90 CN**: 声明 TableGen def `int_arm_usat16`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L92 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L92 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `Packing and unpacking`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Packing and unpacking`。
- **L95 EN**: Declares TableGen def `int_arm_sxtab16`.
  **L95 CN**: 声明 TableGen def `int_arm_sxtab16`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。

### Lines 97-120

````tablegen
                          [IntrNoMem]>;
def int_arm_sxtb16 : ClangBuiltin<"__builtin_arm_sxtb16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;
def int_arm_uxtab16 : ClangBuiltin<"__builtin_arm_uxtab16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uxtb16 : ClangBuiltin<"__builtin_arm_uxtb16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;

// Parallel selection, reads the GE flags.
def int_arm_sel : ClangBuiltin<"__builtin_arm_sel">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrReadMem]>;

// Parallel 8-bit addition and subtraction
def int_arm_qadd8  : ClangBuiltin<"__builtin_arm_qadd8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_qsub8  : ClangBuiltin<"__builtin_arm_qsub8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
// Writes to the GE bits.
def int_arm_sadd8  : ClangBuiltin<"__builtin_arm_sadd8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
````
- **L97 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L97 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L98 EN**: Declares TableGen def `int_arm_sxtb16`.
  **L98 CN**: 声明 TableGen def `int_arm_sxtb16`。
- **L99 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L99 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L100 EN**: Declares TableGen def `int_arm_uxtab16`.
  **L100 CN**: 声明 TableGen def `int_arm_uxtab16`。
- **L101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L101 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L102 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L102 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L103 EN**: Declares TableGen def `int_arm_uxtb16`.
  **L103 CN**: 声明 TableGen def `int_arm_uxtb16`。
- **L104 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L104 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L106 EN**: Comment explains nearby logic, invariants, or intent: `Parallel selection, reads the GE flags.`.
  **L106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parallel selection, reads the GE flags.`。
- **L107 EN**: Declares TableGen def `int_arm_sel`.
  **L107 CN**: 声明 TableGen def `int_arm_sel`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L109 EN**: Executes a standalone statement or declaration: `[IntrReadMem]>;`.
  **L109 CN**: 执行一条独立语句或声明：`[IntrReadMem]>;`。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `Parallel 8-bit addition and subtraction`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parallel 8-bit addition and subtraction`。
- **L112 EN**: Declares TableGen def `int_arm_qadd8`.
  **L112 CN**: 声明 TableGen def `int_arm_qadd8`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L114 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L114 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L115 EN**: Declares TableGen def `int_arm_qsub8`.
  **L115 CN**: 声明 TableGen def `int_arm_qsub8`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L117 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L117 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L118 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L119 EN**: Declares TableGen def `int_arm_sadd8`.
  **L119 CN**: 声明 TableGen def `int_arm_sadd8`。
- **L120 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L120 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。

### Lines 121-144

````tablegen
def int_arm_shadd8  : ClangBuiltin<"__builtin_arm_shadd8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_shsub8  : ClangBuiltin<"__builtin_arm_shsub8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
// Writes to the GE bits.
def int_arm_ssub8  : ClangBuiltin<"__builtin_arm_ssub8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
// Writes to the GE bits.
def int_arm_uadd8  : ClangBuiltin<"__builtin_arm_uadd8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
def int_arm_uhadd8  : ClangBuiltin<"__builtin_arm_uhadd8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uhsub8  : ClangBuiltin<"__builtin_arm_uhsub8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uqadd8  : ClangBuiltin<"__builtin_arm_uqadd8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uqsub8  : ClangBuiltin<"__builtin_arm_uqsub8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
````
- **L121 EN**: Declares TableGen def `int_arm_shadd8`.
  **L121 CN**: 声明 TableGen def `int_arm_shadd8`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L123 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L123 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L124 EN**: Declares TableGen def `int_arm_shsub8`.
  **L124 CN**: 声明 TableGen def `int_arm_shsub8`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L126 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L126 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L127 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L127 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L128 EN**: Declares TableGen def `int_arm_ssub8`.
  **L128 CN**: 声明 TableGen def `int_arm_ssub8`。
- **L129 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L129 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L130 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L131 EN**: Declares TableGen def `int_arm_uadd8`.
  **L131 CN**: 声明 TableGen def `int_arm_uadd8`。
- **L132 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L132 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L133 EN**: Declares TableGen def `int_arm_uhadd8`.
  **L133 CN**: 声明 TableGen def `int_arm_uhadd8`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L135 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L135 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L136 EN**: Declares TableGen def `int_arm_uhsub8`.
  **L136 CN**: 声明 TableGen def `int_arm_uhsub8`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L138 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L138 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L139 EN**: Declares TableGen def `int_arm_uqadd8`.
  **L139 CN**: 声明 TableGen def `int_arm_uqadd8`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L141 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L141 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L142 EN**: Declares TableGen def `int_arm_uqsub8`.
  **L142 CN**: 声明 TableGen def `int_arm_uqsub8`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L144 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L144 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 145-168

````tablegen
// Writes to the GE bits.
def int_arm_usub8  : ClangBuiltin<"__builtin_arm_usub8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;

// Sum of 8-bit absolute differences
def int_arm_usad8  : ClangBuiltin<"__builtin_arm_usad8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_usada8  : ClangBuiltin<"__builtin_arm_usada8">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;

// Parallel 16-bit addition and subtraction
def int_arm_qadd16  : ClangBuiltin<"__builtin_arm_qadd16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_qasx  : ClangBuiltin<"__builtin_arm_qasx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_qsax  : ClangBuiltin<"__builtin_arm_qsax">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_qsub16  : ClangBuiltin<"__builtin_arm_qsub16">,
````
- **L145 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L145 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L146 EN**: Declares TableGen def `int_arm_usub8`.
  **L146 CN**: 声明 TableGen def `int_arm_usub8`。
- **L147 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L147 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L149 EN**: Comment explains nearby logic, invariants, or intent: `Sum of 8-bit absolute differences`.
  **L149 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sum of 8-bit absolute differences`。
- **L150 EN**: Declares TableGen def `int_arm_usad8`.
  **L150 CN**: 声明 TableGen def `int_arm_usad8`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L152 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L152 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L153 EN**: Declares TableGen def `int_arm_usada8`.
  **L153 CN**: 声明 TableGen def `int_arm_usada8`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L156 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L156 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L158 EN**: Comment explains nearby logic, invariants, or intent: `Parallel 16-bit addition and subtraction`.
  **L158 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parallel 16-bit addition and subtraction`。
- **L159 EN**: Declares TableGen def `int_arm_qadd16`.
  **L159 CN**: 声明 TableGen def `int_arm_qadd16`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L161 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L161 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L162 EN**: Declares TableGen def `int_arm_qasx`.
  **L162 CN**: 声明 TableGen def `int_arm_qasx`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L164 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L164 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L165 EN**: Declares TableGen def `int_arm_qsax`.
  **L165 CN**: 声明 TableGen def `int_arm_qsax`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L167 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L167 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L168 EN**: Declares TableGen def `int_arm_qsub16`.
  **L168 CN**: 声明 TableGen def `int_arm_qsub16`。

### Lines 169-192

````tablegen
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
// Writes to the GE bits.
def int_arm_sadd16  : ClangBuiltin<"__builtin_arm_sadd16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
// Writes to the GE bits.
def int_arm_sasx  : ClangBuiltin<"__builtin_arm_sasx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
def int_arm_shadd16  : ClangBuiltin<"__builtin_arm_shadd16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_shasx  : ClangBuiltin<"__builtin_arm_shasx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_shsax  : ClangBuiltin<"__builtin_arm_shsax">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_shsub16  : ClangBuiltin<"__builtin_arm_shsub16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
// Writes to the GE bits.
def int_arm_ssax  : ClangBuiltin<"__builtin_arm_ssax">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
// Writes to the GE bits.
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L170 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L170 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L171 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L171 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L172 EN**: Declares TableGen def `int_arm_sadd16`.
  **L172 CN**: 声明 TableGen def `int_arm_sadd16`。
- **L173 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L173 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L174 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L174 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L175 EN**: Declares TableGen def `int_arm_sasx`.
  **L175 CN**: 声明 TableGen def `int_arm_sasx`。
- **L176 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L176 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L177 EN**: Declares TableGen def `int_arm_shadd16`.
  **L177 CN**: 声明 TableGen def `int_arm_shadd16`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L179 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L179 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L180 EN**: Declares TableGen def `int_arm_shasx`.
  **L180 CN**: 声明 TableGen def `int_arm_shasx`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L182 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L182 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L183 EN**: Declares TableGen def `int_arm_shsax`.
  **L183 CN**: 声明 TableGen def `int_arm_shsax`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L185 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L185 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L186 EN**: Declares TableGen def `int_arm_shsub16`.
  **L186 CN**: 声明 TableGen def `int_arm_shsub16`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L188 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L188 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L189 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L189 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L190 EN**: Declares TableGen def `int_arm_ssax`.
  **L190 CN**: 声明 TableGen def `int_arm_ssax`。
- **L191 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L191 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L192 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L192 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。

### Lines 193-216

````tablegen
def int_arm_ssub16  : ClangBuiltin<"__builtin_arm_ssub16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
// Writes to the GE bits.
def int_arm_uadd16  : ClangBuiltin<"__builtin_arm_uadd16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
// Writes to the GE bits.
def int_arm_uasx  : ClangBuiltin<"__builtin_arm_uasx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
def int_arm_uhadd16  : ClangBuiltin<"__builtin_arm_uhadd16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uhasx  : ClangBuiltin<"__builtin_arm_uhasx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uhsax  : ClangBuiltin<"__builtin_arm_uhsax">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uhsub16  : ClangBuiltin<"__builtin_arm_uhsub16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uqadd16  : ClangBuiltin<"__builtin_arm_uqadd16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uqasx  : ClangBuiltin<"__builtin_arm_uqasx">,
````
- **L193 EN**: Declares TableGen def `int_arm_ssub16`.
  **L193 CN**: 声明 TableGen def `int_arm_ssub16`。
- **L194 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L194 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L195 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L195 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L196 EN**: Declares TableGen def `int_arm_uadd16`.
  **L196 CN**: 声明 TableGen def `int_arm_uadd16`。
- **L197 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L197 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L198 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L198 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L199 EN**: Declares TableGen def `int_arm_uasx`.
  **L199 CN**: 声明 TableGen def `int_arm_uasx`。
- **L200 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L200 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L201 EN**: Declares TableGen def `int_arm_uhadd16`.
  **L201 CN**: 声明 TableGen def `int_arm_uhadd16`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L203 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L203 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L204 EN**: Declares TableGen def `int_arm_uhasx`.
  **L204 CN**: 声明 TableGen def `int_arm_uhasx`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L206 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L206 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L207 EN**: Declares TableGen def `int_arm_uhsax`.
  **L207 CN**: 声明 TableGen def `int_arm_uhsax`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L209 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L209 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L210 EN**: Declares TableGen def `int_arm_uhsub16`.
  **L210 CN**: 声明 TableGen def `int_arm_uhsub16`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L212 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L212 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L213 EN**: Declares TableGen def `int_arm_uqadd16`.
  **L213 CN**: 声明 TableGen def `int_arm_uqadd16`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L215 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L215 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L216 EN**: Declares TableGen def `int_arm_uqasx`.
  **L216 CN**: 声明 TableGen def `int_arm_uqasx`。

### Lines 217-240

````tablegen
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uqsax  : ClangBuiltin<"__builtin_arm_uqsax">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_uqsub16  : ClangBuiltin<"__builtin_arm_uqsub16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
// Writes to the GE bits.
def int_arm_usax  : ClangBuiltin<"__builtin_arm_usax">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;
// Writes to the GE bits.
def int_arm_usub16  : ClangBuiltin<"__builtin_arm_usub16">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;

// Parallel 16-bit multiplication
def int_arm_smlad : ClangBuiltin<"__builtin_arm_smlad">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smladx : ClangBuiltin<"__builtin_arm_smladx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L218 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L218 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L219 EN**: Declares TableGen def `int_arm_uqsax`.
  **L219 CN**: 声明 TableGen def `int_arm_uqsax`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L221 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L221 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L222 EN**: Declares TableGen def `int_arm_uqsub16`.
  **L222 CN**: 声明 TableGen def `int_arm_uqsub16`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L224 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L224 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L225 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L225 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L226 EN**: Declares TableGen def `int_arm_usax`.
  **L226 CN**: 声明 TableGen def `int_arm_usax`。
- **L227 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L227 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L228 EN**: Comment explains nearby logic, invariants, or intent: `Writes to the GE bits.`.
  **L228 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes to the GE bits.`。
- **L229 EN**: Declares TableGen def `int_arm_usub16`.
  **L229 CN**: 声明 TableGen def `int_arm_usub16`。
- **L230 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`.
  **L230 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], []>;`。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L232 EN**: Comment explains nearby logic, invariants, or intent: `Parallel 16-bit multiplication`.
  **L232 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parallel 16-bit multiplication`。
- **L233 EN**: Declares TableGen def `int_arm_smlad`.
  **L233 CN**: 声明 TableGen def `int_arm_smlad`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L236 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L236 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L237 EN**: Declares TableGen def `int_arm_smladx`.
  **L237 CN**: 声明 TableGen def `int_arm_smladx`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L240 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L240 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 241-264

````tablegen
def int_arm_smlald : ClangBuiltin<"__builtin_arm_smlald">,
    DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i64_ty],
                          [IntrNoMem]>;
def int_arm_smlaldx : ClangBuiltin<"__builtin_arm_smlaldx">,
    DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i64_ty],
                          [IntrNoMem]>;
def int_arm_smlsd : ClangBuiltin<"__builtin_arm_smlsd">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smlsdx : ClangBuiltin<"__builtin_arm_smlsdx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smlsld : ClangBuiltin<"__builtin_arm_smlsld">,
    DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i64_ty],
                          [IntrNoMem]>;
def int_arm_smlsldx : ClangBuiltin<"__builtin_arm_smlsldx">,
    DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,
                           llvm_i64_ty],
                          [IntrNoMem]>;
````
- **L241 EN**: Declares TableGen def `int_arm_smlald`.
  **L241 CN**: 声明 TableGen def `int_arm_smlald`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty],`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty],`。
- **L244 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L244 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L245 EN**: Declares TableGen def `int_arm_smlaldx`.
  **L245 CN**: 声明 TableGen def `int_arm_smlaldx`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty],`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty],`。
- **L248 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L248 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L249 EN**: Declares TableGen def `int_arm_smlsd`.
  **L249 CN**: 声明 TableGen def `int_arm_smlsd`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L252 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L252 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L253 EN**: Declares TableGen def `int_arm_smlsdx`.
  **L253 CN**: 声明 TableGen def `int_arm_smlsdx`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L256 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L256 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L257 EN**: Declares TableGen def `int_arm_smlsld`.
  **L257 CN**: 声明 TableGen def `int_arm_smlsld`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty],`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty],`。
- **L260 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L260 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L261 EN**: Declares TableGen def `int_arm_smlsldx`.
  **L261 CN**: 声明 TableGen def `int_arm_smlsldx`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty],`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty],`。
- **L264 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L264 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 265-288

````tablegen
def int_arm_smuad : ClangBuiltin<"__builtin_arm_smuad">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smuadx : ClangBuiltin<"__builtin_arm_smuadx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smusd : ClangBuiltin<"__builtin_arm_smusd">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;
def int_arm_smusdx : ClangBuiltin<"__builtin_arm_smusdx">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                          [IntrNoMem]>;


//===----------------------------------------------------------------------===//
// Load, Store and Clear exclusive

// TODO: Add applicable default attributes.
def int_arm_ldrex : Intrinsic<[llvm_i32_ty], [llvm_anyptr_ty]>;
def int_arm_strex : Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_anyptr_ty]>;

def int_arm_ldaex : Intrinsic<[llvm_i32_ty], [llvm_anyptr_ty]>;
def int_arm_stlex : Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_anyptr_ty]>;

````
- **L265 EN**: Declares TableGen def `int_arm_smuad`.
  **L265 CN**: 声明 TableGen def `int_arm_smuad`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L267 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L267 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L268 EN**: Declares TableGen def `int_arm_smuadx`.
  **L268 CN**: 声明 TableGen def `int_arm_smuadx`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L270 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L270 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L271 EN**: Declares TableGen def `int_arm_smusd`.
  **L271 CN**: 声明 TableGen def `int_arm_smusd`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L273 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L273 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L274 EN**: Declares TableGen def `int_arm_smusdx`.
  **L274 CN**: 声明 TableGen def `int_arm_smusdx`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L276 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L276 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L279 EN**: Banner comment marking a file or section boundary.
  **L279 CN**: 横幅注释，用于标记文件或章节边界。
- **L280 EN**: Comment explains nearby logic, invariants, or intent: `Load, Store and Clear exclusive`.
  **L280 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Load, Store and Clear exclusive`。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L282 EN**: Comment records a pending task or caution: `TODO: Add applicable default attributes.`.
  **L282 CN**: 注释记录了待办事项或注意点：`TODO: Add applicable default attributes.`。
- **L283 EN**: Declares TableGen def `int_arm_ldrex`.
  **L283 CN**: 声明 TableGen def `int_arm_ldrex`。
- **L284 EN**: Declares TableGen def `int_arm_strex`.
  **L284 CN**: 声明 TableGen def `int_arm_strex`。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L286 EN**: Declares TableGen def `int_arm_ldaex`.
  **L286 CN**: 声明 TableGen def `int_arm_ldaex`。
- **L287 EN**: Declares TableGen def `int_arm_stlex`.
  **L287 CN**: 声明 TableGen def `int_arm_stlex`。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 289-312

````tablegen
def int_arm_clrex : Intrinsic<[]>;

def int_arm_strexd : Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
    llvm_ptr_ty]>;
def int_arm_ldrexd : Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_ptr_ty]>;

def int_arm_stlexd : Intrinsic<[llvm_i32_ty],
                               [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty]>;
def int_arm_ldaexd : Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_ptr_ty]>;

//===----------------------------------------------------------------------===//
// Data barrier instructions

// TODO: Add applicable default attributes.
def int_arm_dmb : ClangBuiltin<"__builtin_arm_dmb">, MSBuiltin<"__dmb">,
                  Intrinsic<[], [llvm_i32_ty]>;
def int_arm_dsb : ClangBuiltin<"__builtin_arm_dsb">, MSBuiltin<"__dsb">,
                  Intrinsic<[], [llvm_i32_ty]>;
def int_arm_isb : ClangBuiltin<"__builtin_arm_isb">, MSBuiltin<"__isb">,
                  Intrinsic<[], [llvm_i32_ty]>;

//===----------------------------------------------------------------------===//
// VFP

````
- **L289 EN**: Declares TableGen def `int_arm_clrex`.
  **L289 CN**: 声明 TableGen def `int_arm_clrex`。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L291 EN**: Declares TableGen def `int_arm_strexd`.
  **L291 CN**: 声明 TableGen def `int_arm_strexd`。
- **L292 EN**: Executes a standalone statement or declaration: `llvm_ptr_ty]>;`.
  **L292 CN**: 执行一条独立语句或声明：`llvm_ptr_ty]>;`。
- **L293 EN**: Declares TableGen def `int_arm_ldrexd`.
  **L293 CN**: 声明 TableGen def `int_arm_ldrexd`。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Declares TableGen def `int_arm_stlexd`.
  **L295 CN**: 声明 TableGen def `int_arm_stlexd`。
- **L296 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty]>;`.
  **L296 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty]>;`。
- **L297 EN**: Declares TableGen def `int_arm_ldaexd`.
  **L297 CN**: 声明 TableGen def `int_arm_ldaexd`。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L299 EN**: Banner comment marking a file or section boundary.
  **L299 CN**: 横幅注释，用于标记文件或章节边界。
- **L300 EN**: Comment explains nearby logic, invariants, or intent: `Data barrier instructions`.
  **L300 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Data barrier instructions`。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L302 EN**: Comment records a pending task or caution: `TODO: Add applicable default attributes.`.
  **L302 CN**: 注释记录了待办事项或注意点：`TODO: Add applicable default attributes.`。
- **L303 EN**: Declares TableGen def `int_arm_dmb`.
  **L303 CN**: 声明 TableGen def `int_arm_dmb`。
- **L304 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty]>;`.
  **L304 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty]>;`。
- **L305 EN**: Declares TableGen def `int_arm_dsb`.
  **L305 CN**: 声明 TableGen def `int_arm_dsb`。
- **L306 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty]>;`.
  **L306 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty]>;`。
- **L307 EN**: Declares TableGen def `int_arm_isb`.
  **L307 CN**: 声明 TableGen def `int_arm_isb`。
- **L308 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty]>;`.
  **L308 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty]>;`。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Banner comment marking a file or section boundary.
  **L310 CN**: 横幅注释，用于标记文件或章节边界。
- **L311 EN**: Comment explains nearby logic, invariants, or intent: `VFP`.
  **L311 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VFP`。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 313-336

````tablegen
def int_arm_get_fpscr : ClangBuiltin<"__builtin_arm_get_fpscr">,
                       DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrReadMem]>;
def int_arm_set_fpscr : ClangBuiltin<"__builtin_arm_set_fpscr">,
                       DefaultAttrsIntrinsic<[], [llvm_i32_ty], []>;
def int_arm_vcvtr : DefaultAttrsIntrinsic<[llvm_float_ty],
                                          [llvm_anyfloat_ty], [IntrNoMem]>;
def int_arm_vcvtru : DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_anyfloat_ty],
                                           [IntrNoMem]>;

//===----------------------------------------------------------------------===//
// Coprocessor

// TODO: Add applicable default attributes.
def int_arm_ldc : ClangBuiltin<"__builtin_arm_ldc">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
def int_arm_ldcl : ClangBuiltin<"__builtin_arm_ldcl">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
def int_arm_ldc2 : ClangBuiltin<"__builtin_arm_ldc2">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
def int_arm_ldc2l : ClangBuiltin<"__builtin_arm_ldc2l">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;

def int_arm_stc : ClangBuiltin<"__builtin_arm_stc">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
````
- **L313 EN**: Declares TableGen def `int_arm_get_fpscr`.
  **L313 CN**: 声明 TableGen def `int_arm_get_fpscr`。
- **L314 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrReadMem]>;`.
  **L314 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrReadMem]>;`。
- **L315 EN**: Declares TableGen def `int_arm_set_fpscr`.
  **L315 CN**: 声明 TableGen def `int_arm_set_fpscr`。
- **L316 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [llvm_i32_ty], []>;`.
  **L316 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [llvm_i32_ty], []>;`。
- **L317 EN**: Declares TableGen def `int_arm_vcvtr`.
  **L317 CN**: 声明 TableGen def `int_arm_vcvtr`。
- **L318 EN**: Executes a standalone statement or declaration: `[llvm_anyfloat_ty], [IntrNoMem]>;`.
  **L318 CN**: 执行一条独立语句或声明：`[llvm_anyfloat_ty], [IntrNoMem]>;`。
- **L319 EN**: Declares TableGen def `int_arm_vcvtru`.
  **L319 CN**: 声明 TableGen def `int_arm_vcvtru`。
- **L320 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L320 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L322 EN**: Banner comment marking a file or section boundary.
  **L322 CN**: 横幅注释，用于标记文件或章节边界。
- **L323 EN**: Comment explains nearby logic, invariants, or intent: `Coprocessor`.
  **L323 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Coprocessor`。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L325 EN**: Comment records a pending task or caution: `TODO: Add applicable default attributes.`.
  **L325 CN**: 注释记录了待办事项或注意点：`TODO: Add applicable default attributes.`。
- **L326 EN**: Declares TableGen def `int_arm_ldc`.
  **L326 CN**: 声明 TableGen def `int_arm_ldc`。
- **L327 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L327 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L328 EN**: Declares TableGen def `int_arm_ldcl`.
  **L328 CN**: 声明 TableGen def `int_arm_ldcl`。
- **L329 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L329 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L330 EN**: Declares TableGen def `int_arm_ldc2`.
  **L330 CN**: 声明 TableGen def `int_arm_ldc2`。
- **L331 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L331 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L332 EN**: Declares TableGen def `int_arm_ldc2l`.
  **L332 CN**: 声明 TableGen def `int_arm_ldc2l`。
- **L333 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L333 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L335 EN**: Declares TableGen def `int_arm_stc`.
  **L335 CN**: 声明 TableGen def `int_arm_stc`。
- **L336 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L336 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。

### Lines 337-360

````tablegen
def int_arm_stcl : ClangBuiltin<"__builtin_arm_stcl">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
def int_arm_stc2 : ClangBuiltin<"__builtin_arm_stc2">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
def int_arm_stc2l : ClangBuiltin<"__builtin_arm_stc2l">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;

// Move to coprocessor
def int_arm_mcr : ClangBuiltin<"__builtin_arm_mcr">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                  llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;
def int_arm_mcr2 : ClangBuiltin<"__builtin_arm_mcr2">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                  llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;

// Move from coprocessor
def int_arm_mrc : ClangBuiltin<"__builtin_arm_mrc">,
                  MSBuiltin<"_MoveFromCoprocessor">,
   Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                             llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;
def int_arm_mrc2 : ClangBuiltin<"__builtin_arm_mrc2">,
                   MSBuiltin<"_MoveFromCoprocessor2">,
   Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                             llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;
````
- **L337 EN**: Declares TableGen def `int_arm_stcl`.
  **L337 CN**: 声明 TableGen def `int_arm_stcl`。
- **L338 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L338 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L339 EN**: Declares TableGen def `int_arm_stc2`.
  **L339 CN**: 声明 TableGen def `int_arm_stc2`。
- **L340 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L340 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L341 EN**: Declares TableGen def `int_arm_stc2l`.
  **L341 CN**: 声明 TableGen def `int_arm_stc2l`。
- **L342 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L342 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_ptr_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L344 EN**: Comment explains nearby logic, invariants, or intent: `Move to coprocessor`.
  **L344 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Move to coprocessor`。
- **L345 EN**: Declares TableGen def `int_arm_mcr`.
  **L345 CN**: 声明 TableGen def `int_arm_mcr`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L347 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`.
  **L347 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`。
- **L348 EN**: Declares TableGen def `int_arm_mcr2`.
  **L348 CN**: 声明 TableGen def `int_arm_mcr2`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L350 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`.
  **L350 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L352 EN**: Comment explains nearby logic, invariants, or intent: `Move from coprocessor`.
  **L352 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Move from coprocessor`。
- **L353 EN**: Declares TableGen def `int_arm_mrc`.
  **L353 CN**: 声明 TableGen def `int_arm_mrc`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MSBuiltin<"_MoveFromCoprocessor">,`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`MSBuiltin<"_MoveFromCoprocessor">,`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L356 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`.
  **L356 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`。
- **L357 EN**: Declares TableGen def `int_arm_mrc2`.
  **L357 CN**: 声明 TableGen def `int_arm_mrc2`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MSBuiltin<"_MoveFromCoprocessor2">,`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`MSBuiltin<"_MoveFromCoprocessor2">,`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L360 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`.
  **L360 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`。

### Lines 361-384

````tablegen

// Coprocessor data processing
def int_arm_cdp : ClangBuiltin<"__builtin_arm_cdp">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                  llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;
def int_arm_cdp2 : ClangBuiltin<"__builtin_arm_cdp2">,
   Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                  llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;

// Move from two registers to coprocessor
def int_arm_mcrr : Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                                  llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>;
def int_arm_mcrr2 : Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
                                   llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>;

def int_arm_mrrc : Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty,
                              llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;
def int_arm_mrrc2 : Intrinsic<[llvm_i32_ty, llvm_i32_ty], [llvm_i32_ty,
                               llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;

//===----------------------------------------------------------------------===//
// CRC32

def int_arm_crc32b : DefaultAttrsIntrinsic<
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L362 EN**: Comment explains nearby logic, invariants, or intent: `Coprocessor data processing`.
  **L362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Coprocessor data processing`。
- **L363 EN**: Declares TableGen def `int_arm_cdp`.
  **L363 CN**: 声明 TableGen def `int_arm_cdp`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L365 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`.
  **L365 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`。
- **L366 EN**: Declares TableGen def `int_arm_cdp2`.
  **L366 CN**: 声明 TableGen def `int_arm_cdp2`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L368 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`.
  **L368 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>, ImmArg<ArgIndex<5>>]>;`。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L370 EN**: Comment explains nearby logic, invariants, or intent: `Move from two registers to coprocessor`.
  **L370 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Move from two registers to coprocessor`。
- **L371 EN**: Declares TableGen def `int_arm_mcrr`.
  **L371 CN**: 声明 TableGen def `int_arm_mcrr`。
- **L372 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>;`.
  **L372 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>;`。
- **L373 EN**: Declares TableGen def `int_arm_mcrr2`.
  **L373 CN**: 声明 TableGen def `int_arm_mcrr2`。
- **L374 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>;`.
  **L374 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<4>>]>;`。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L376 EN**: Declares TableGen def `int_arm_mrrc`.
  **L376 CN**: 声明 TableGen def `int_arm_mrrc`。
- **L377 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`.
  **L377 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`。
- **L378 EN**: Declares TableGen def `int_arm_mrrc2`.
  **L378 CN**: 声明 TableGen def `int_arm_mrrc2`。
- **L379 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`.
  **L379 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L381 EN**: Banner comment marking a file or section boundary.
  **L381 CN**: 横幅注释，用于标记文件或章节边界。
- **L382 EN**: Comment explains nearby logic, invariants, or intent: `CRC32`.
  **L382 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CRC32`。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L384 EN**: Declares TableGen def `int_arm_crc32b`.
  **L384 CN**: 声明 TableGen def `int_arm_crc32b`。

### Lines 385-408

````tablegen
    [llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_arm_crc32cb : DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_arm_crc32h  : DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_arm_crc32ch : DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_arm_crc32w  : DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_arm_crc32cw : DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;

//===----------------------------------------------------------------------===//
// CMSE

// TODO: Add applicable default attributes.
def int_arm_cmse_tt : ClangBuiltin<"__builtin_arm_cmse_TT">,
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;
def int_arm_cmse_ttt : ClangBuiltin<"__builtin_arm_cmse_TTT">,
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;
def int_arm_cmse_tta : ClangBuiltin<"__builtin_arm_cmse_TTA">,
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;
def int_arm_cmse_ttat : ClangBuiltin<"__builtin_arm_cmse_TTAT">,
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;
````
- **L385 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L385 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L386 EN**: Declares TableGen def `int_arm_crc32cb`.
  **L386 CN**: 声明 TableGen def `int_arm_crc32cb`。
- **L387 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L387 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L388 EN**: Declares TableGen def `int_arm_crc32h`.
  **L388 CN**: 声明 TableGen def `int_arm_crc32h`。
- **L389 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L389 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L390 EN**: Declares TableGen def `int_arm_crc32ch`.
  **L390 CN**: 声明 TableGen def `int_arm_crc32ch`。
- **L391 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L391 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L392 EN**: Declares TableGen def `int_arm_crc32w`.
  **L392 CN**: 声明 TableGen def `int_arm_crc32w`。
- **L393 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L393 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L394 EN**: Declares TableGen def `int_arm_crc32cw`.
  **L394 CN**: 声明 TableGen def `int_arm_crc32cw`。
- **L395 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L395 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L396 EN**: Blank line separating nearby declarations or logic blocks.
  **L396 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L397 EN**: Banner comment marking a file or section boundary.
  **L397 CN**: 横幅注释，用于标记文件或章节边界。
- **L398 EN**: Comment explains nearby logic, invariants, or intent: `CMSE`.
  **L398 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CMSE`。
- **L399 EN**: Blank line separating nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L400 EN**: Comment records a pending task or caution: `TODO: Add applicable default attributes.`.
  **L400 CN**: 注释记录了待办事项或注意点：`TODO: Add applicable default attributes.`。
- **L401 EN**: Declares TableGen def `int_arm_cmse_tt`.
  **L401 CN**: 声明 TableGen def `int_arm_cmse_tt`。
- **L402 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`.
  **L402 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`。
- **L403 EN**: Declares TableGen def `int_arm_cmse_ttt`.
  **L403 CN**: 声明 TableGen def `int_arm_cmse_ttt`。
- **L404 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`.
  **L404 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`。
- **L405 EN**: Declares TableGen def `int_arm_cmse_tta`.
  **L405 CN**: 声明 TableGen def `int_arm_cmse_tta`。
- **L406 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`.
  **L406 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`。
- **L407 EN**: Declares TableGen def `int_arm_cmse_ttat`.
  **L407 CN**: 声明 TableGen def `int_arm_cmse_ttat`。
- **L408 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`.
  **L408 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrNoMem]>;`。

### Lines 409-432

````tablegen

//===----------------------------------------------------------------------===//
// HINT

// TODO: Add applicable default attributes.
def int_arm_hint : Intrinsic<[], [llvm_i32_ty]>;
def int_arm_dbg : Intrinsic<[], [llvm_i32_ty]>;

//===----------------------------------------------------------------------===//
// UND (reserved undefined sequence)

// TODO: Add applicable default attributes.
def int_arm_undefined : Intrinsic<[], [llvm_i32_ty]>;

//===----------------------------------------------------------------------===//
// Advanced SIMD (NEON)

// The following classes do not correspond directly to GCC builtins.
class Neon_1Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMMatchType<0>], [IntrNoMem]>;
class Neon_1Arg_Narrow_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMExtendedType<0>],
                          [IntrNoMem]>;
class Neon_2Arg_Intrinsic
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L410 EN**: Banner comment marking a file or section boundary.
  **L410 CN**: 横幅注释，用于标记文件或章节边界。
- **L411 EN**: Comment explains nearby logic, invariants, or intent: `HINT`.
  **L411 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`HINT`。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L413 EN**: Comment records a pending task or caution: `TODO: Add applicable default attributes.`.
  **L413 CN**: 注释记录了待办事项或注意点：`TODO: Add applicable default attributes.`。
- **L414 EN**: Declares TableGen def `int_arm_hint`.
  **L414 CN**: 声明 TableGen def `int_arm_hint`。
- **L415 EN**: Declares TableGen def `int_arm_dbg`.
  **L415 CN**: 声明 TableGen def `int_arm_dbg`。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L417 EN**: Banner comment marking a file or section boundary.
  **L417 CN**: 横幅注释，用于标记文件或章节边界。
- **L418 EN**: Comment explains nearby logic, invariants, or intent: `UND (reserved undefined sequence)`.
  **L418 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`UND (reserved undefined sequence)`。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L420 EN**: Comment records a pending task or caution: `TODO: Add applicable default attributes.`.
  **L420 CN**: 注释记录了待办事项或注意点：`TODO: Add applicable default attributes.`。
- **L421 EN**: Declares TableGen def `int_arm_undefined`.
  **L421 CN**: 声明 TableGen def `int_arm_undefined`。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L423 EN**: Banner comment marking a file or section boundary.
  **L423 CN**: 横幅注释，用于标记文件或章节边界。
- **L424 EN**: Comment explains nearby logic, invariants, or intent: `Advanced SIMD (NEON)`.
  **L424 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Advanced SIMD (NEON)`。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L426 EN**: Comment explains nearby logic, invariants, or intent: `The following classes do not correspond directly to GCC builtins.`.
  **L426 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following classes do not correspond directly to GCC builtins.`。
- **L427 EN**: Declares class `Neon_1Arg_Intrinsic`.
  **L427 CN**: 声明 class `Neon_1Arg_Intrinsic`。
- **L428 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMMatchType<0>], [IntrNoMem]>;`.
  **L428 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMMatchType<0>], [IntrNoMem]>;`。
- **L429 EN**: Declares class `Neon_1Arg_Narrow_Intrinsic`.
  **L429 CN**: 声明 class `Neon_1Arg_Narrow_Intrinsic`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMExtendedType<0>],`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [LLVMExtendedType<0>],`。
- **L431 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L431 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L432 EN**: Declares class `Neon_2Arg_Intrinsic`.
  **L432 CN**: 声明 class `Neon_2Arg_Intrinsic`。

### Lines 433-456

````tablegen
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;
class Neon_2Arg_Narrow_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMExtendedType<0>, LLVMExtendedType<0>],
                          [IntrNoMem]>;
class Neon_2Arg_Long_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMTruncatedType<0>, LLVMTruncatedType<0>],
                          [IntrNoMem]>;
class Neon_3Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, LLVMMatchType<0>,
                           LLVMMatchType<0>],
                          [IntrNoMem]>;
class Neon_3Arg_Long_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, LLVMTruncatedType<0>,
                           LLVMTruncatedType<0>],
                          [IntrNoMem]>;

class Neon_CvtFxToFP_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyint_ty, llvm_i32_ty],
                          [IntrNoMem]>;
````
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L434 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L434 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L435 EN**: Declares class `Neon_2Arg_Narrow_Intrinsic`.
  **L435 CN**: 声明 class `Neon_2Arg_Narrow_Intrinsic`。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMExtendedType<0>, LLVMExtendedType<0>],`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMExtendedType<0>, LLVMExtendedType<0>],`。
- **L438 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L438 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L439 EN**: Declares class `Neon_2Arg_Long_Intrinsic`.
  **L439 CN**: 声明 class `Neon_2Arg_Long_Intrinsic`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMTruncatedType<0>, LLVMTruncatedType<0>],`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMTruncatedType<0>, LLVMTruncatedType<0>],`。
- **L442 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L442 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L443 EN**: Declares class `Neon_3Arg_Intrinsic`.
  **L443 CN**: 声明 class `Neon_3Arg_Intrinsic`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L447 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L447 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L448 EN**: Declares class `Neon_3Arg_Long_Intrinsic`.
  **L448 CN**: 声明 class `Neon_3Arg_Long_Intrinsic`。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMTruncatedType<0>,`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMTruncatedType<0>,`。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMTruncatedType<0>],`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMTruncatedType<0>],`。
- **L452 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L452 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L454 EN**: Declares class `Neon_CvtFxToFP_Intrinsic`.
  **L454 CN**: 声明 class `Neon_CvtFxToFP_Intrinsic`。
- **L455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyint_ty, llvm_i32_ty],`.
  **L455 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyfloat_ty], [llvm_anyint_ty, llvm_i32_ty],`。
- **L456 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L456 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 457-480

````tablegen
class Neon_CvtFPToFx_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, llvm_i32_ty],
                          [IntrNoMem]>;
class Neon_CvtFPtoInt_1Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty],
                          [IntrNoMem]>;

class Neon_Compare_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [llvm_anyvector_ty, LLVMMatchType<1>], [IntrNoMem]>;

// The table operands for VTBL and VTBX consist of 1 to 4 v8i8 vectors.
// Besides the table, VTBL has one other v8i8 argument and VTBX has two.
// Overall, the classes range from 2 to 6 v8i8 arguments.
class Neon_Tbl2Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_v8i8_ty],
                          [llvm_v8i8_ty, llvm_v8i8_ty], [IntrNoMem]>;
class Neon_Tbl3Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_v8i8_ty],
                          [llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty],
                          [IntrNoMem]>;
class Neon_Tbl4Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_v8i8_ty],
                          [llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,
````
- **L457 EN**: Declares class `Neon_CvtFPToFx_Intrinsic`.
  **L457 CN**: 声明 class `Neon_CvtFPToFx_Intrinsic`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, llvm_i32_ty],`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyfloat_ty, llvm_i32_ty],`。
- **L459 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L459 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L460 EN**: Declares class `Neon_CvtFPtoInt_1Arg_Intrinsic`.
  **L460 CN**: 声明 class `Neon_CvtFPtoInt_1Arg_Intrinsic`。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty],`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_anyvector_ty],`。
- **L462 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L462 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L464 EN**: Declares class `Neon_Compare_Intrinsic`.
  **L464 CN**: 声明 class `Neon_Compare_Intrinsic`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L466 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty, LLVMMatchType<1>], [IntrNoMem]>;`.
  **L466 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty, LLVMMatchType<1>], [IntrNoMem]>;`。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L468 EN**: Comment explains nearby logic, invariants, or intent: `The table operands for VTBL and VTBX consist of 1 to 4 v8i8 vectors.`.
  **L468 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The table operands for VTBL and VTBX consist of 1 to 4 v8i8 vectors.`。
- **L469 EN**: Comment explains nearby logic, invariants, or intent: `Besides the table, VTBL has one other v8i8 argument and VTBX has two.`.
  **L469 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Besides the table, VTBL has one other v8i8 argument and VTBX has two.`。
- **L470 EN**: Comment explains nearby logic, invariants, or intent: `Overall, the classes range from 2 to 6 v8i8 arguments.`.
  **L470 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Overall, the classes range from 2 to 6 v8i8 arguments.`。
- **L471 EN**: Declares class `Neon_Tbl2Arg_Intrinsic`.
  **L471 CN**: 声明 class `Neon_Tbl2Arg_Intrinsic`。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`。
- **L473 EN**: Executes a standalone statement or declaration: `[llvm_v8i8_ty, llvm_v8i8_ty], [IntrNoMem]>;`.
  **L473 CN**: 执行一条独立语句或声明：`[llvm_v8i8_ty, llvm_v8i8_ty], [IntrNoMem]>;`。
- **L474 EN**: Declares class `Neon_Tbl3Arg_Intrinsic`.
  **L474 CN**: 声明 class `Neon_Tbl3Arg_Intrinsic`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`.
  **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty],`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty],`。
- **L477 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L477 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L478 EN**: Declares class `Neon_Tbl4Arg_Intrinsic`.
  **L478 CN**: 声明 class `Neon_Tbl4Arg_Intrinsic`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,`。

### Lines 481-504

````tablegen
                           llvm_v8i8_ty],
                          [IntrNoMem]>;
class Neon_Tbl5Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_v8i8_ty],
                          [llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,
                           llvm_v8i8_ty, llvm_v8i8_ty],
                          [IntrNoMem]>;
class Neon_Tbl6Arg_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_v8i8_ty],
                          [llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,
                           llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty],
                          [IntrNoMem]>;

// Arithmetic ops

let IntrProperties = [IntrNoMem, Commutative] in {

  // Vector Add.
  def int_arm_neon_vhadds : Neon_2Arg_Intrinsic;
  def int_arm_neon_vhaddu : Neon_2Arg_Intrinsic;
  def int_arm_neon_vrhadds : Neon_2Arg_Intrinsic;
  def int_arm_neon_vrhaddu : Neon_2Arg_Intrinsic;
  def int_arm_neon_vraddhn : Neon_2Arg_Narrow_Intrinsic;

````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v8i8_ty],`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v8i8_ty],`。
- **L482 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L482 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L483 EN**: Declares class `Neon_Tbl5Arg_Intrinsic`.
  **L483 CN**: 声明 class `Neon_Tbl5Arg_Intrinsic`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v8i8_ty, llvm_v8i8_ty],`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v8i8_ty, llvm_v8i8_ty],`。
- **L487 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L487 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L488 EN**: Declares class `Neon_Tbl6Arg_Intrinsic`.
  **L488 CN**: 声明 class `Neon_Tbl6Arg_Intrinsic`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v8i8_ty],`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty,`。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty],`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v8i8_ty, llvm_v8i8_ty, llvm_v8i8_ty],`。
- **L492 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L492 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L494 EN**: Comment explains nearby logic, invariants, or intent: `Arithmetic ops`.
  **L494 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Arithmetic ops`。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L496 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L496 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L497 EN**: Blank line separating nearby declarations or logic blocks.
  **L497 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L498 EN**: Comment explains nearby logic, invariants, or intent: `Vector Add.`.
  **L498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Add.`。
- **L499 EN**: Declares TableGen def `int_arm_neon_vhadds`.
  **L499 CN**: 声明 TableGen def `int_arm_neon_vhadds`。
- **L500 EN**: Declares TableGen def `int_arm_neon_vhaddu`.
  **L500 CN**: 声明 TableGen def `int_arm_neon_vhaddu`。
- **L501 EN**: Declares TableGen def `int_arm_neon_vrhadds`.
  **L501 CN**: 声明 TableGen def `int_arm_neon_vrhadds`。
- **L502 EN**: Declares TableGen def `int_arm_neon_vrhaddu`.
  **L502 CN**: 声明 TableGen def `int_arm_neon_vrhaddu`。
- **L503 EN**: Declares TableGen def `int_arm_neon_vraddhn`.
  **L503 CN**: 声明 TableGen def `int_arm_neon_vraddhn`。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 505-528

````tablegen
  // Vector Multiply.
  def int_arm_neon_vmulp : Neon_2Arg_Intrinsic;
  def int_arm_neon_vqdmulh : Neon_2Arg_Intrinsic;
  def int_arm_neon_vqrdmulh : Neon_2Arg_Intrinsic;
  def int_arm_neon_vmulls : Neon_2Arg_Long_Intrinsic;
  def int_arm_neon_vmullu : Neon_2Arg_Long_Intrinsic;
  def int_arm_neon_vmullp : Neon_2Arg_Long_Intrinsic;
  def int_arm_neon_vqdmull : Neon_2Arg_Long_Intrinsic;

  // Vector Maximum.
  def int_arm_neon_vmaxs : Neon_2Arg_Intrinsic;
  def int_arm_neon_vmaxu : Neon_2Arg_Intrinsic;
  def int_arm_neon_vmaxnm : Neon_2Arg_Intrinsic;

  // Vector Minimum.
  def int_arm_neon_vmins : Neon_2Arg_Intrinsic;
  def int_arm_neon_vminu : Neon_2Arg_Intrinsic;
  def int_arm_neon_vminnm : Neon_2Arg_Intrinsic;

  // Vector Reciprocal Step.
  def int_arm_neon_vrecps : Neon_2Arg_Intrinsic;

  // Vector Reciprocal Square Root Step.
  def int_arm_neon_vrsqrts : Neon_2Arg_Intrinsic;
````
- **L505 EN**: Comment explains nearby logic, invariants, or intent: `Vector Multiply.`.
  **L505 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Multiply.`。
- **L506 EN**: Declares TableGen def `int_arm_neon_vmulp`.
  **L506 CN**: 声明 TableGen def `int_arm_neon_vmulp`。
- **L507 EN**: Declares TableGen def `int_arm_neon_vqdmulh`.
  **L507 CN**: 声明 TableGen def `int_arm_neon_vqdmulh`。
- **L508 EN**: Declares TableGen def `int_arm_neon_vqrdmulh`.
  **L508 CN**: 声明 TableGen def `int_arm_neon_vqrdmulh`。
- **L509 EN**: Declares TableGen def `int_arm_neon_vmulls`.
  **L509 CN**: 声明 TableGen def `int_arm_neon_vmulls`。
- **L510 EN**: Declares TableGen def `int_arm_neon_vmullu`.
  **L510 CN**: 声明 TableGen def `int_arm_neon_vmullu`。
- **L511 EN**: Declares TableGen def `int_arm_neon_vmullp`.
  **L511 CN**: 声明 TableGen def `int_arm_neon_vmullp`。
- **L512 EN**: Declares TableGen def `int_arm_neon_vqdmull`.
  **L512 CN**: 声明 TableGen def `int_arm_neon_vqdmull`。
- **L513 EN**: Blank line separating nearby declarations or logic blocks.
  **L513 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L514 EN**: Comment explains nearby logic, invariants, or intent: `Vector Maximum.`.
  **L514 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Maximum.`。
- **L515 EN**: Declares TableGen def `int_arm_neon_vmaxs`.
  **L515 CN**: 声明 TableGen def `int_arm_neon_vmaxs`。
- **L516 EN**: Declares TableGen def `int_arm_neon_vmaxu`.
  **L516 CN**: 声明 TableGen def `int_arm_neon_vmaxu`。
- **L517 EN**: Declares TableGen def `int_arm_neon_vmaxnm`.
  **L517 CN**: 声明 TableGen def `int_arm_neon_vmaxnm`。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L519 EN**: Comment explains nearby logic, invariants, or intent: `Vector Minimum.`.
  **L519 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Minimum.`。
- **L520 EN**: Declares TableGen def `int_arm_neon_vmins`.
  **L520 CN**: 声明 TableGen def `int_arm_neon_vmins`。
- **L521 EN**: Declares TableGen def `int_arm_neon_vminu`.
  **L521 CN**: 声明 TableGen def `int_arm_neon_vminu`。
- **L522 EN**: Declares TableGen def `int_arm_neon_vminnm`.
  **L522 CN**: 声明 TableGen def `int_arm_neon_vminnm`。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L524 EN**: Comment explains nearby logic, invariants, or intent: `Vector Reciprocal Step.`.
  **L524 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Reciprocal Step.`。
- **L525 EN**: Declares TableGen def `int_arm_neon_vrecps`.
  **L525 CN**: 声明 TableGen def `int_arm_neon_vrecps`。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L527 EN**: Comment explains nearby logic, invariants, or intent: `Vector Reciprocal Square Root Step.`.
  **L527 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Reciprocal Square Root Step.`。
- **L528 EN**: Declares TableGen def `int_arm_neon_vrsqrts`.
  **L528 CN**: 声明 TableGen def `int_arm_neon_vrsqrts`。

### Lines 529-552

````tablegen
}

// Vector Subtract.
def int_arm_neon_vhsubs : Neon_2Arg_Intrinsic;
def int_arm_neon_vhsubu : Neon_2Arg_Intrinsic;
def int_arm_neon_vrsubhn : Neon_2Arg_Narrow_Intrinsic;

// Vector Absolute Compare.
def int_arm_neon_vacge : Neon_Compare_Intrinsic;
def int_arm_neon_vacgt : Neon_Compare_Intrinsic;

// Vector Absolute Differences.
def int_arm_neon_vabds : Neon_2Arg_Intrinsic;
def int_arm_neon_vabdu : Neon_2Arg_Intrinsic;

// Vector Pairwise Add.
def int_arm_neon_vpadd : Neon_2Arg_Intrinsic;

// Vector Pairwise Add Long.
// Note: This is different than the other "long" NEON intrinsics because
// the result vector has half as many elements as the source vector.
// The source and destination vector types must be specified separately.
def int_arm_neon_vpaddls : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L531 EN**: Comment explains nearby logic, invariants, or intent: `Vector Subtract.`.
  **L531 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Subtract.`。
- **L532 EN**: Declares TableGen def `int_arm_neon_vhsubs`.
  **L532 CN**: 声明 TableGen def `int_arm_neon_vhsubs`。
- **L533 EN**: Declares TableGen def `int_arm_neon_vhsubu`.
  **L533 CN**: 声明 TableGen def `int_arm_neon_vhsubu`。
- **L534 EN**: Declares TableGen def `int_arm_neon_vrsubhn`.
  **L534 CN**: 声明 TableGen def `int_arm_neon_vrsubhn`。
- **L535 EN**: Blank line separating nearby declarations or logic blocks.
  **L535 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L536 EN**: Comment explains nearby logic, invariants, or intent: `Vector Absolute Compare.`.
  **L536 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Absolute Compare.`。
- **L537 EN**: Declares TableGen def `int_arm_neon_vacge`.
  **L537 CN**: 声明 TableGen def `int_arm_neon_vacge`。
- **L538 EN**: Declares TableGen def `int_arm_neon_vacgt`.
  **L538 CN**: 声明 TableGen def `int_arm_neon_vacgt`。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L540 EN**: Comment explains nearby logic, invariants, or intent: `Vector Absolute Differences.`.
  **L540 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Absolute Differences.`。
- **L541 EN**: Declares TableGen def `int_arm_neon_vabds`.
  **L541 CN**: 声明 TableGen def `int_arm_neon_vabds`。
- **L542 EN**: Declares TableGen def `int_arm_neon_vabdu`.
  **L542 CN**: 声明 TableGen def `int_arm_neon_vabdu`。
- **L543 EN**: Blank line separating nearby declarations or logic blocks.
  **L543 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L544 EN**: Comment explains nearby logic, invariants, or intent: `Vector Pairwise Add.`.
  **L544 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Pairwise Add.`。
- **L545 EN**: Declares TableGen def `int_arm_neon_vpadd`.
  **L545 CN**: 声明 TableGen def `int_arm_neon_vpadd`。
- **L546 EN**: Blank line separating nearby declarations or logic blocks.
  **L546 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L547 EN**: Comment explains nearby logic, invariants, or intent: `Vector Pairwise Add Long.`.
  **L547 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Pairwise Add Long.`。
- **L548 EN**: Comment explains nearby logic, invariants, or intent: `Note: This is different than the other "long" NEON intrinsics because`.
  **L548 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: This is different than the other "long" NEON intrinsics because`。
- **L549 EN**: Comment explains nearby logic, invariants, or intent: `the result vector has half as many elements as the source vector.`.
  **L549 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the result vector has half as many elements as the source vector.`。
- **L550 EN**: Comment explains nearby logic, invariants, or intent: `The source and destination vector types must be specified separately.`.
  **L550 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The source and destination vector types must be specified separately.`。
- **L551 EN**: Declares TableGen def `int_arm_neon_vpaddls`.
  **L551 CN**: 声明 TableGen def `int_arm_neon_vpaddls`。
- **L552 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L552 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。

### Lines 553-576

````tablegen
def int_arm_neon_vpaddlu : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;

// Vector Pairwise Add and Accumulate Long.
// Note: This is similar to vpaddl but the destination vector also appears
// as the first argument.
def int_arm_neon_vpadals : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty], [IntrNoMem]>;
def int_arm_neon_vpadalu : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty], [IntrNoMem]>;

// Vector Pairwise Maximum and Minimum.
def int_arm_neon_vpmaxs : Neon_2Arg_Intrinsic;
def int_arm_neon_vpmaxu : Neon_2Arg_Intrinsic;
def int_arm_neon_vpmins : Neon_2Arg_Intrinsic;
def int_arm_neon_vpminu : Neon_2Arg_Intrinsic;

// Vector Shifts:
//
// The various saturating and rounding vector shift operations need to be
// represented by intrinsics in LLVM, and even the basic VSHL variable shift
// operation cannot be safely translated to LLVM's shift operators.  VSHL can
// be used for both left and right shifts, or even combinations of the two,
// depending on the signs of the shift amounts.  It also has well-defined
````
- **L553 EN**: Declares TableGen def `int_arm_neon_vpaddlu`.
  **L553 CN**: 声明 TableGen def `int_arm_neon_vpaddlu`。
- **L554 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L554 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。
- **L555 EN**: Blank line separating nearby declarations or logic blocks.
  **L555 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L556 EN**: Comment explains nearby logic, invariants, or intent: `Vector Pairwise Add and Accumulate Long.`.
  **L556 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Pairwise Add and Accumulate Long.`。
- **L557 EN**: Comment explains nearby logic, invariants, or intent: `Note: This is similar to vpaddl but the destination vector also appears`.
  **L557 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: This is similar to vpaddl but the destination vector also appears`。
- **L558 EN**: Comment explains nearby logic, invariants, or intent: `as the first argument.`.
  **L558 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`as the first argument.`。
- **L559 EN**: Declares TableGen def `int_arm_neon_vpadals`.
  **L559 CN**: 声明 TableGen def `int_arm_neon_vpadals`。
- **L560 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty], [IntrNoMem]>;`.
  **L560 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty], [IntrNoMem]>;`。
- **L561 EN**: Declares TableGen def `int_arm_neon_vpadalu`.
  **L561 CN**: 声明 TableGen def `int_arm_neon_vpadalu`。
- **L562 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty], [IntrNoMem]>;`.
  **L562 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty], [IntrNoMem]>;`。
- **L563 EN**: Blank line separating nearby declarations or logic blocks.
  **L563 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L564 EN**: Comment explains nearby logic, invariants, or intent: `Vector Pairwise Maximum and Minimum.`.
  **L564 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Pairwise Maximum and Minimum.`。
- **L565 EN**: Declares TableGen def `int_arm_neon_vpmaxs`.
  **L565 CN**: 声明 TableGen def `int_arm_neon_vpmaxs`。
- **L566 EN**: Declares TableGen def `int_arm_neon_vpmaxu`.
  **L566 CN**: 声明 TableGen def `int_arm_neon_vpmaxu`。
- **L567 EN**: Declares TableGen def `int_arm_neon_vpmins`.
  **L567 CN**: 声明 TableGen def `int_arm_neon_vpmins`。
- **L568 EN**: Declares TableGen def `int_arm_neon_vpminu`.
  **L568 CN**: 声明 TableGen def `int_arm_neon_vpminu`。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L570 EN**: Comment explains nearby logic, invariants, or intent: `Vector Shifts:`.
  **L570 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Shifts:`。
- **L571 EN**: Separator comment used for visual grouping.
  **L571 CN**: 用于视觉分组的分隔注释。
- **L572 EN**: Comment explains nearby logic, invariants, or intent: `The various saturating and rounding vector shift operations need to be`.
  **L572 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The various saturating and rounding vector shift operations need to be`。
- **L573 EN**: Comment explains nearby logic, invariants, or intent: `represented by intrinsics in LLVM, and even the basic VSHL variable shift`.
  **L573 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`represented by intrinsics in LLVM, and even the basic VSHL variable shift`。
- **L574 EN**: Comment explains nearby logic, invariants, or intent: `operation cannot be safely translated to LLVM's shift operators.  VSHL can`.
  **L574 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operation cannot be safely translated to LLVM's shift operators.  VSHL can`。
- **L575 EN**: Comment explains nearby logic, invariants, or intent: `be used for both left and right shifts, or even combinations of the two,`.
  **L575 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`be used for both left and right shifts, or even combinations of the two,`。
- **L576 EN**: Comment explains nearby logic, invariants, or intent: `depending on the signs of the shift amounts.  It also has well-defined`.
  **L576 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`depending on the signs of the shift amounts.  It also has well-defined`。

### Lines 577-600

````tablegen
// behavior for shift amounts that LLVM leaves undefined.  Only basic shifts
// by constants can be represented with LLVM's shift operators.
//
// The shift counts for these intrinsics are always vectors, even for constant
// shifts, where the constant is replicated.  For consistency with VSHL (and
// other variable shift instructions), left shifts have positive shift counts
// and right shifts have negative shift counts.  This convention is also used
// for constant right shift intrinsics, and to help preserve sanity, the
// intrinsic names use "shift" instead of either "shl" or "shr".  Where
// applicable, signed and unsigned versions of the intrinsics are
// distinguished with "s" and "u" suffixes.  A few NEON shift instructions,
// such as VQSHLU, take signed operands but produce unsigned results; these
// use a "su" suffix.

// Vector Shift.
def int_arm_neon_vshifts : Neon_2Arg_Intrinsic;
def int_arm_neon_vshiftu : Neon_2Arg_Intrinsic;

// Vector Rounding Shift.
def int_arm_neon_vrshifts : Neon_2Arg_Intrinsic;
def int_arm_neon_vrshiftu : Neon_2Arg_Intrinsic;
def int_arm_neon_vrshiftn : Neon_2Arg_Narrow_Intrinsic;

// Vector Saturating Shift.
````
- **L577 EN**: Comment explains nearby logic, invariants, or intent: `behavior for shift amounts that LLVM leaves undefined.  Only basic shifts`.
  **L577 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`behavior for shift amounts that LLVM leaves undefined.  Only basic shifts`。
- **L578 EN**: Comment explains nearby logic, invariants, or intent: `by constants can be represented with LLVM's shift operators.`.
  **L578 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`by constants can be represented with LLVM's shift operators.`。
- **L579 EN**: Separator comment used for visual grouping.
  **L579 CN**: 用于视觉分组的分隔注释。
- **L580 EN**: Comment explains nearby logic, invariants, or intent: `The shift counts for these intrinsics are always vectors, even for constant`.
  **L580 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The shift counts for these intrinsics are always vectors, even for constant`。
- **L581 EN**: Comment explains nearby logic, invariants, or intent: `shifts, where the constant is replicated.  For consistency with VSHL (and`.
  **L581 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`shifts, where the constant is replicated.  For consistency with VSHL (and`。
- **L582 EN**: Comment explains nearby logic, invariants, or intent: `other variable shift instructions), left shifts have positive shift counts`.
  **L582 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`other variable shift instructions), left shifts have positive shift counts`。
- **L583 EN**: Comment explains nearby logic, invariants, or intent: `and right shifts have negative shift counts.  This convention is also used`.
  **L583 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and right shifts have negative shift counts.  This convention is also used`。
- **L584 EN**: Comment explains nearby logic, invariants, or intent: `for constant right shift intrinsics, and to help preserve sanity, the`.
  **L584 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for constant right shift intrinsics, and to help preserve sanity, the`。
- **L585 EN**: Comment explains nearby logic, invariants, or intent: `intrinsic names use "shift" instead of either "shl" or "shr".  Where`.
  **L585 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`intrinsic names use "shift" instead of either "shl" or "shr".  Where`。
- **L586 EN**: Comment explains nearby logic, invariants, or intent: `applicable, signed and unsigned versions of the intrinsics are`.
  **L586 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`applicable, signed and unsigned versions of the intrinsics are`。
- **L587 EN**: Comment explains nearby logic, invariants, or intent: `distinguished with "s" and "u" suffixes.  A few NEON shift instructions,`.
  **L587 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`distinguished with "s" and "u" suffixes.  A few NEON shift instructions,`。
- **L588 EN**: Comment explains nearby logic, invariants, or intent: `such as VQSHLU, take signed operands but produce unsigned results; these`.
  **L588 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`such as VQSHLU, take signed operands but produce unsigned results; these`。
- **L589 EN**: Comment explains nearby logic, invariants, or intent: `use a "su" suffix.`.
  **L589 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`use a "su" suffix.`。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L591 EN**: Comment explains nearby logic, invariants, or intent: `Vector Shift.`.
  **L591 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Shift.`。
- **L592 EN**: Declares TableGen def `int_arm_neon_vshifts`.
  **L592 CN**: 声明 TableGen def `int_arm_neon_vshifts`。
- **L593 EN**: Declares TableGen def `int_arm_neon_vshiftu`.
  **L593 CN**: 声明 TableGen def `int_arm_neon_vshiftu`。
- **L594 EN**: Blank line separating nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L595 EN**: Comment explains nearby logic, invariants, or intent: `Vector Rounding Shift.`.
  **L595 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Rounding Shift.`。
- **L596 EN**: Declares TableGen def `int_arm_neon_vrshifts`.
  **L596 CN**: 声明 TableGen def `int_arm_neon_vrshifts`。
- **L597 EN**: Declares TableGen def `int_arm_neon_vrshiftu`.
  **L597 CN**: 声明 TableGen def `int_arm_neon_vrshiftu`。
- **L598 EN**: Declares TableGen def `int_arm_neon_vrshiftn`.
  **L598 CN**: 声明 TableGen def `int_arm_neon_vrshiftn`。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L600 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Shift.`.
  **L600 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Shift.`。

### Lines 601-624

````tablegen
def int_arm_neon_vqshifts : Neon_2Arg_Intrinsic;
def int_arm_neon_vqshiftu : Neon_2Arg_Intrinsic;
def int_arm_neon_vqshiftsu : Neon_2Arg_Intrinsic;
def int_arm_neon_vqshiftns : Neon_2Arg_Narrow_Intrinsic;
def int_arm_neon_vqshiftnu : Neon_2Arg_Narrow_Intrinsic;
def int_arm_neon_vqshiftnsu : Neon_2Arg_Narrow_Intrinsic;

// Vector Saturating Rounding Shift.
def int_arm_neon_vqrshifts : Neon_2Arg_Intrinsic;
def int_arm_neon_vqrshiftu : Neon_2Arg_Intrinsic;
def int_arm_neon_vqrshiftns : Neon_2Arg_Narrow_Intrinsic;
def int_arm_neon_vqrshiftnu : Neon_2Arg_Narrow_Intrinsic;
def int_arm_neon_vqrshiftnsu : Neon_2Arg_Narrow_Intrinsic;

// Vector Shift and Insert.
def int_arm_neon_vshiftins : Neon_3Arg_Intrinsic;

// Vector Absolute Value and Saturating Absolute Value.
def int_arm_neon_vabs : Neon_1Arg_Intrinsic;
def int_arm_neon_vqabs : Neon_1Arg_Intrinsic;

// Vector Saturating Negate.
def int_arm_neon_vqneg : Neon_1Arg_Intrinsic;

````
- **L601 EN**: Declares TableGen def `int_arm_neon_vqshifts`.
  **L601 CN**: 声明 TableGen def `int_arm_neon_vqshifts`。
- **L602 EN**: Declares TableGen def `int_arm_neon_vqshiftu`.
  **L602 CN**: 声明 TableGen def `int_arm_neon_vqshiftu`。
- **L603 EN**: Declares TableGen def `int_arm_neon_vqshiftsu`.
  **L603 CN**: 声明 TableGen def `int_arm_neon_vqshiftsu`。
- **L604 EN**: Declares TableGen def `int_arm_neon_vqshiftns`.
  **L604 CN**: 声明 TableGen def `int_arm_neon_vqshiftns`。
- **L605 EN**: Declares TableGen def `int_arm_neon_vqshiftnu`.
  **L605 CN**: 声明 TableGen def `int_arm_neon_vqshiftnu`。
- **L606 EN**: Declares TableGen def `int_arm_neon_vqshiftnsu`.
  **L606 CN**: 声明 TableGen def `int_arm_neon_vqshiftnsu`。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L608 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Rounding Shift.`.
  **L608 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Rounding Shift.`。
- **L609 EN**: Declares TableGen def `int_arm_neon_vqrshifts`.
  **L609 CN**: 声明 TableGen def `int_arm_neon_vqrshifts`。
- **L610 EN**: Declares TableGen def `int_arm_neon_vqrshiftu`.
  **L610 CN**: 声明 TableGen def `int_arm_neon_vqrshiftu`。
- **L611 EN**: Declares TableGen def `int_arm_neon_vqrshiftns`.
  **L611 CN**: 声明 TableGen def `int_arm_neon_vqrshiftns`。
- **L612 EN**: Declares TableGen def `int_arm_neon_vqrshiftnu`.
  **L612 CN**: 声明 TableGen def `int_arm_neon_vqrshiftnu`。
- **L613 EN**: Declares TableGen def `int_arm_neon_vqrshiftnsu`.
  **L613 CN**: 声明 TableGen def `int_arm_neon_vqrshiftnsu`。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L615 EN**: Comment explains nearby logic, invariants, or intent: `Vector Shift and Insert.`.
  **L615 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Shift and Insert.`。
- **L616 EN**: Declares TableGen def `int_arm_neon_vshiftins`.
  **L616 CN**: 声明 TableGen def `int_arm_neon_vshiftins`。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L618 EN**: Comment explains nearby logic, invariants, or intent: `Vector Absolute Value and Saturating Absolute Value.`.
  **L618 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Absolute Value and Saturating Absolute Value.`。
- **L619 EN**: Declares TableGen def `int_arm_neon_vabs`.
  **L619 CN**: 声明 TableGen def `int_arm_neon_vabs`。
- **L620 EN**: Declares TableGen def `int_arm_neon_vqabs`.
  **L620 CN**: 声明 TableGen def `int_arm_neon_vqabs`。
- **L621 EN**: Blank line separating nearby declarations or logic blocks.
  **L621 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L622 EN**: Comment explains nearby logic, invariants, or intent: `Vector Saturating Negate.`.
  **L622 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Saturating Negate.`。
- **L623 EN**: Declares TableGen def `int_arm_neon_vqneg`.
  **L623 CN**: 声明 TableGen def `int_arm_neon_vqneg`。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 625-648

````tablegen
// Vector Count Leading Sign/Zero Bits.
def int_arm_neon_vcls : Neon_1Arg_Intrinsic;

// Vector Reciprocal Estimate.
def int_arm_neon_vrecpe : Neon_1Arg_Intrinsic;

// Vector Reciprocal Square Root Estimate.
def int_arm_neon_vrsqrte : Neon_1Arg_Intrinsic;

// Vector Conversions Between Floating-point and Integer
def int_arm_neon_vcvtau : Neon_CvtFPtoInt_1Arg_Intrinsic;
def int_arm_neon_vcvtas : Neon_CvtFPtoInt_1Arg_Intrinsic;
def int_arm_neon_vcvtnu : Neon_CvtFPtoInt_1Arg_Intrinsic;
def int_arm_neon_vcvtns : Neon_CvtFPtoInt_1Arg_Intrinsic;
def int_arm_neon_vcvtpu : Neon_CvtFPtoInt_1Arg_Intrinsic;
def int_arm_neon_vcvtps : Neon_CvtFPtoInt_1Arg_Intrinsic;
def int_arm_neon_vcvtmu : Neon_CvtFPtoInt_1Arg_Intrinsic;
def int_arm_neon_vcvtms : Neon_CvtFPtoInt_1Arg_Intrinsic;

// Vector Conversions Between Floating-point and Fixed-point.
def int_arm_neon_vcvtfp2fxs : Neon_CvtFPToFx_Intrinsic;
def int_arm_neon_vcvtfp2fxu : Neon_CvtFPToFx_Intrinsic;
def int_arm_neon_vcvtfxs2fp : Neon_CvtFxToFP_Intrinsic;
def int_arm_neon_vcvtfxu2fp : Neon_CvtFxToFP_Intrinsic;
````
- **L625 EN**: Comment explains nearby logic, invariants, or intent: `Vector Count Leading Sign/Zero Bits.`.
  **L625 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Count Leading Sign/Zero Bits.`。
- **L626 EN**: Declares TableGen def `int_arm_neon_vcls`.
  **L626 CN**: 声明 TableGen def `int_arm_neon_vcls`。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L628 EN**: Comment explains nearby logic, invariants, or intent: `Vector Reciprocal Estimate.`.
  **L628 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Reciprocal Estimate.`。
- **L629 EN**: Declares TableGen def `int_arm_neon_vrecpe`.
  **L629 CN**: 声明 TableGen def `int_arm_neon_vrecpe`。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Comment explains nearby logic, invariants, or intent: `Vector Reciprocal Square Root Estimate.`.
  **L631 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Reciprocal Square Root Estimate.`。
- **L632 EN**: Declares TableGen def `int_arm_neon_vrsqrte`.
  **L632 CN**: 声明 TableGen def `int_arm_neon_vrsqrte`。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L634 EN**: Comment explains nearby logic, invariants, or intent: `Vector Conversions Between Floating-point and Integer`.
  **L634 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Conversions Between Floating-point and Integer`。
- **L635 EN**: Declares TableGen def `int_arm_neon_vcvtau`.
  **L635 CN**: 声明 TableGen def `int_arm_neon_vcvtau`。
- **L636 EN**: Declares TableGen def `int_arm_neon_vcvtas`.
  **L636 CN**: 声明 TableGen def `int_arm_neon_vcvtas`。
- **L637 EN**: Declares TableGen def `int_arm_neon_vcvtnu`.
  **L637 CN**: 声明 TableGen def `int_arm_neon_vcvtnu`。
- **L638 EN**: Declares TableGen def `int_arm_neon_vcvtns`.
  **L638 CN**: 声明 TableGen def `int_arm_neon_vcvtns`。
- **L639 EN**: Declares TableGen def `int_arm_neon_vcvtpu`.
  **L639 CN**: 声明 TableGen def `int_arm_neon_vcvtpu`。
- **L640 EN**: Declares TableGen def `int_arm_neon_vcvtps`.
  **L640 CN**: 声明 TableGen def `int_arm_neon_vcvtps`。
- **L641 EN**: Declares TableGen def `int_arm_neon_vcvtmu`.
  **L641 CN**: 声明 TableGen def `int_arm_neon_vcvtmu`。
- **L642 EN**: Declares TableGen def `int_arm_neon_vcvtms`.
  **L642 CN**: 声明 TableGen def `int_arm_neon_vcvtms`。
- **L643 EN**: Blank line separating nearby declarations or logic blocks.
  **L643 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L644 EN**: Comment explains nearby logic, invariants, or intent: `Vector Conversions Between Floating-point and Fixed-point.`.
  **L644 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Conversions Between Floating-point and Fixed-point.`。
- **L645 EN**: Declares TableGen def `int_arm_neon_vcvtfp2fxs`.
  **L645 CN**: 声明 TableGen def `int_arm_neon_vcvtfp2fxs`。
- **L646 EN**: Declares TableGen def `int_arm_neon_vcvtfp2fxu`.
  **L646 CN**: 声明 TableGen def `int_arm_neon_vcvtfp2fxu`。
- **L647 EN**: Declares TableGen def `int_arm_neon_vcvtfxs2fp`.
  **L647 CN**: 声明 TableGen def `int_arm_neon_vcvtfxs2fp`。
- **L648 EN**: Declares TableGen def `int_arm_neon_vcvtfxu2fp`.
  **L648 CN**: 声明 TableGen def `int_arm_neon_vcvtfxu2fp`。

### Lines 649-672

````tablegen

// Vector Conversions Between Half-Precision and Single-Precision.
def int_arm_neon_vcvtfp2hf
    : DefaultAttrsIntrinsic<[llvm_v4i16_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_arm_neon_vcvthf2fp
    : DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i16_ty], [IntrNoMem]>;

// Narrowing Saturating Vector Moves.
def int_arm_neon_vqmovns : Neon_1Arg_Narrow_Intrinsic;
def int_arm_neon_vqmovnu : Neon_1Arg_Narrow_Intrinsic;
def int_arm_neon_vqmovnsu : Neon_1Arg_Narrow_Intrinsic;

// Vector Table Lookup.
// The first 1-4 arguments are the table.
def int_arm_neon_vtbl1 : Neon_Tbl2Arg_Intrinsic;
def int_arm_neon_vtbl2 : Neon_Tbl3Arg_Intrinsic;
def int_arm_neon_vtbl3 : Neon_Tbl4Arg_Intrinsic;
def int_arm_neon_vtbl4 : Neon_Tbl5Arg_Intrinsic;

// Vector Table Extension.
// Some elements of the destination vector may not be updated, so the original
// value of that vector is passed as the first argument.  The next 1-4
// arguments after that are the table.
def int_arm_neon_vtbx1 : Neon_Tbl3Arg_Intrinsic;
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L650 EN**: Comment explains nearby logic, invariants, or intent: `Vector Conversions Between Half-Precision and Single-Precision.`.
  **L650 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Conversions Between Half-Precision and Single-Precision.`。
- **L651 EN**: Declares TableGen def `int_arm_neon_vcvtfp2hf`.
  **L651 CN**: 声明 TableGen def `int_arm_neon_vcvtfp2hf`。
- **L652 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v4i16_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L652 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v4i16_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L653 EN**: Declares TableGen def `int_arm_neon_vcvthf2fp`.
  **L653 CN**: 声明 TableGen def `int_arm_neon_vcvthf2fp`。
- **L654 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i16_ty], [IntrNoMem]>;`.
  **L654 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i16_ty], [IntrNoMem]>;`。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L656 EN**: Comment explains nearby logic, invariants, or intent: `Narrowing Saturating Vector Moves.`.
  **L656 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Narrowing Saturating Vector Moves.`。
- **L657 EN**: Declares TableGen def `int_arm_neon_vqmovns`.
  **L657 CN**: 声明 TableGen def `int_arm_neon_vqmovns`。
- **L658 EN**: Declares TableGen def `int_arm_neon_vqmovnu`.
  **L658 CN**: 声明 TableGen def `int_arm_neon_vqmovnu`。
- **L659 EN**: Declares TableGen def `int_arm_neon_vqmovnsu`.
  **L659 CN**: 声明 TableGen def `int_arm_neon_vqmovnsu`。
- **L660 EN**: Blank line separating nearby declarations or logic blocks.
  **L660 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L661 EN**: Comment explains nearby logic, invariants, or intent: `Vector Table Lookup.`.
  **L661 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Table Lookup.`。
- **L662 EN**: Comment explains nearby logic, invariants, or intent: `The first 1-4 arguments are the table.`.
  **L662 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first 1-4 arguments are the table.`。
- **L663 EN**: Declares TableGen def `int_arm_neon_vtbl1`.
  **L663 CN**: 声明 TableGen def `int_arm_neon_vtbl1`。
- **L664 EN**: Declares TableGen def `int_arm_neon_vtbl2`.
  **L664 CN**: 声明 TableGen def `int_arm_neon_vtbl2`。
- **L665 EN**: Declares TableGen def `int_arm_neon_vtbl3`.
  **L665 CN**: 声明 TableGen def `int_arm_neon_vtbl3`。
- **L666 EN**: Declares TableGen def `int_arm_neon_vtbl4`.
  **L666 CN**: 声明 TableGen def `int_arm_neon_vtbl4`。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L668 EN**: Comment explains nearby logic, invariants, or intent: `Vector Table Extension.`.
  **L668 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Table Extension.`。
- **L669 EN**: Comment explains nearby logic, invariants, or intent: `Some elements of the destination vector may not be updated, so the original`.
  **L669 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Some elements of the destination vector may not be updated, so the original`。
- **L670 EN**: Comment explains nearby logic, invariants, or intent: `value of that vector is passed as the first argument.  The next 1-4`.
  **L670 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value of that vector is passed as the first argument.  The next 1-4`。
- **L671 EN**: Comment explains nearby logic, invariants, or intent: `arguments after that are the table.`.
  **L671 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`arguments after that are the table.`。
- **L672 EN**: Declares TableGen def `int_arm_neon_vtbx1`.
  **L672 CN**: 声明 TableGen def `int_arm_neon_vtbx1`。

### Lines 673-696

````tablegen
def int_arm_neon_vtbx2 : Neon_Tbl4Arg_Intrinsic;
def int_arm_neon_vtbx3 : Neon_Tbl5Arg_Intrinsic;
def int_arm_neon_vtbx4 : Neon_Tbl6Arg_Intrinsic;

// De-interleaving vector loads from N-element structures.
// Source operands are the address and alignment.
def int_arm_neon_vld1 : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty], [llvm_anyptr_ty, llvm_i32_ty],
    [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld2 : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty, llvm_i32_ty],
    [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld3 : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],
    [llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld4 : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
    [llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;

def int_arm_neon_vld1x2 : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>],
    [llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld1x3 : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],
````
- **L673 EN**: Declares TableGen def `int_arm_neon_vtbx2`.
  **L673 CN**: 声明 TableGen def `int_arm_neon_vtbx2`。
- **L674 EN**: Declares TableGen def `int_arm_neon_vtbx3`.
  **L674 CN**: 声明 TableGen def `int_arm_neon_vtbx3`。
- **L675 EN**: Declares TableGen def `int_arm_neon_vtbx4`.
  **L675 CN**: 声明 TableGen def `int_arm_neon_vtbx4`。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L677 EN**: Comment explains nearby logic, invariants, or intent: `De-interleaving vector loads from N-element structures.`.
  **L677 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`De-interleaving vector loads from N-element structures.`。
- **L678 EN**: Comment explains nearby logic, invariants, or intent: `Source operands are the address and alignment.`.
  **L678 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Source operands are the address and alignment.`。
- **L679 EN**: Declares TableGen def `int_arm_neon_vld1`.
  **L679 CN**: 声明 TableGen def `int_arm_neon_vld1`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [llvm_anyptr_ty, llvm_i32_ty],`。
- **L681 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L681 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L682 EN**: Declares TableGen def `int_arm_neon_vld2`.
  **L682 CN**: 声明 TableGen def `int_arm_neon_vld2`。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty, llvm_i32_ty],`。
- **L684 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L684 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L685 EN**: Declares TableGen def `int_arm_neon_vld3`.
  **L685 CN**: 声明 TableGen def `int_arm_neon_vld3`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L687 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L687 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L688 EN**: Declares TableGen def `int_arm_neon_vld4`.
  **L688 CN**: 声明 TableGen def `int_arm_neon_vld4`。
- **L689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L689 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L690 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L690 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L692 EN**: Declares TableGen def `int_arm_neon_vld1x2`.
  **L692 CN**: 声明 TableGen def `int_arm_neon_vld1x2`。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L694 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L694 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L695 EN**: Declares TableGen def `int_arm_neon_vld1x3`.
  **L695 CN**: 声明 TableGen def `int_arm_neon_vld1x3`。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`。

### Lines 697-720

````tablegen
    [llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld1x4 : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
    [llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;

// Vector load N-element structure to one lane.
// Source operands are: the address, the N input vectors (since only one
// lane is assigned), the lane number, and the alignment.
def int_arm_neon_vld2lane : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>],
    [llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty,
     llvm_i32_ty],
    [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld3lane : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],
    [llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
     llvm_i32_ty, llvm_i32_ty],
    [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld4lane : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
    [llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
     LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],
    [IntrReadMem, IntrArgMemOnly]>;

````
- **L697 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L697 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L698 EN**: Declares TableGen def `int_arm_neon_vld1x4`.
  **L698 CN**: 声明 TableGen def `int_arm_neon_vld1x4`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L700 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L700 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L702 EN**: Comment explains nearby logic, invariants, or intent: `Vector load N-element structure to one lane.`.
  **L702 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector load N-element structure to one lane.`。
- **L703 EN**: Comment explains nearby logic, invariants, or intent: `Source operands are: the address, the N input vectors (since only one`.
  **L703 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Source operands are: the address, the N input vectors (since only one`。
- **L704 EN**: Comment explains nearby logic, invariants, or intent: `lane is assigned), the lane number, and the alignment.`.
  **L704 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lane is assigned), the lane number, and the alignment.`。
- **L705 EN**: Declares TableGen def `int_arm_neon_vld2lane`.
  **L705 CN**: 声明 TableGen def `int_arm_neon_vld2lane`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty,`.
  **L707 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty,`。
- **L708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L708 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L709 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L709 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L710 EN**: Declares TableGen def `int_arm_neon_vld3lane`.
  **L710 CN**: 声明 TableGen def `int_arm_neon_vld3lane`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L714 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L714 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L715 EN**: Declares TableGen def `int_arm_neon_vld4lane`.
  **L715 CN**: 声明 TableGen def `int_arm_neon_vld4lane`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty],`。
- **L719 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L719 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 721-744

````tablegen
// Vector load N-element structure to all lanes.
// Source operands are the address and alignment.
def int_arm_neon_vld2dup : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty, llvm_i32_ty],
    [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld3dup : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],
    [llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;
def int_arm_neon_vld4dup : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
    [llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;

// Interleaving vector stores from N-element structures.
// Source operands are: the address, the N vectors, and the alignment.
def int_arm_neon_vst1 : DefaultAttrsIntrinsic<
    [], [llvm_anyptr_ty, llvm_anyvector_ty, llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly]>;
def int_arm_neon_vst2 : DefaultAttrsIntrinsic<
    [], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly]>;
def int_arm_neon_vst3 : DefaultAttrsIntrinsic<
    [],
    [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,
     llvm_i32_ty],
````
- **L721 EN**: Comment explains nearby logic, invariants, or intent: `Vector load N-element structure to all lanes.`.
  **L721 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector load N-element structure to all lanes.`。
- **L722 EN**: Comment explains nearby logic, invariants, or intent: `Source operands are the address and alignment.`.
  **L722 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Source operands are the address and alignment.`。
- **L723 EN**: Declares TableGen def `int_arm_neon_vld2dup`.
  **L723 CN**: 声明 TableGen def `int_arm_neon_vld2dup`。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty, llvm_i32_ty],`。
- **L725 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L725 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L726 EN**: Declares TableGen def `int_arm_neon_vld3dup`.
  **L726 CN**: 声明 TableGen def `int_arm_neon_vld3dup`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L728 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L728 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L729 EN**: Declares TableGen def `int_arm_neon_vld4dup`.
  **L729 CN**: 声明 TableGen def `int_arm_neon_vld4dup`。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L731 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L731 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty, llvm_i32_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L732 EN**: Blank line separating nearby declarations or logic blocks.
  **L732 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L733 EN**: Comment explains nearby logic, invariants, or intent: `Interleaving vector stores from N-element structures.`.
  **L733 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Interleaving vector stores from N-element structures.`。
- **L734 EN**: Comment explains nearby logic, invariants, or intent: `Source operands are: the address, the N vectors, and the alignment.`.
  **L734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Source operands are: the address, the N vectors, and the alignment.`。
- **L735 EN**: Declares TableGen def `int_arm_neon_vst1`.
  **L735 CN**: 声明 TableGen def `int_arm_neon_vst1`。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_anyptr_ty, llvm_anyvector_ty, llvm_i32_ty],`.
  **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_anyptr_ty, llvm_anyvector_ty, llvm_i32_ty],`。
- **L737 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L737 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L738 EN**: Declares TableGen def `int_arm_neon_vst2`.
  **L738 CN**: 声明 TableGen def `int_arm_neon_vst2`。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],`。
- **L740 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L740 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L741 EN**: Declares TableGen def `int_arm_neon_vst3`.
  **L741 CN**: 声明 TableGen def `int_arm_neon_vst3`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`.
  **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。

### Lines 745-768

````tablegen
    [IntrWriteMem, IntrArgMemOnly]>;
def int_arm_neon_vst4 : DefaultAttrsIntrinsic<
    [],
    [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,
     LLVMMatchType<1>, llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly]>;

def int_arm_neon_vst1x2 : DefaultAttrsIntrinsic<
    [], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>],
    [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
def int_arm_neon_vst1x3 : DefaultAttrsIntrinsic<
    [], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>],
    [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
def int_arm_neon_vst1x4 : DefaultAttrsIntrinsic<
    [],
    [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,
     LLVMMatchType<1>],
    [IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;

// Vector store N-element structure from one lane.
// Source operands are: the address, the N vectors, the lane number, and
// the alignment.
def int_arm_neon_vst2lane : DefaultAttrsIntrinsic<
    [],
````
- **L745 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L745 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L746 EN**: Declares TableGen def `int_arm_neon_vst4`.
  **L746 CN**: 声明 TableGen def `int_arm_neon_vst4`。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, llvm_i32_ty],`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, llvm_i32_ty],`。
- **L750 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L750 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L751 EN**: Blank line separating nearby declarations or logic blocks.
  **L751 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L752 EN**: Declares TableGen def `int_arm_neon_vst1x2`.
  **L752 CN**: 声明 TableGen def `int_arm_neon_vst1x2`。
- **L753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>],`.
  **L753 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>],`。
- **L754 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L754 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L755 EN**: Declares TableGen def `int_arm_neon_vst1x3`.
  **L755 CN**: 声明 TableGen def `int_arm_neon_vst1x3`。
- **L756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>],`.
  **L756 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>],`。
- **L757 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L757 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L758 EN**: Declares TableGen def `int_arm_neon_vst1x4`.
  **L758 CN**: 声明 TableGen def `int_arm_neon_vst1x4`。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L762 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L762 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L763 EN**: Blank line separating nearby declarations or logic blocks.
  **L763 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L764 EN**: Comment explains nearby logic, invariants, or intent: `Vector store N-element structure from one lane.`.
  **L764 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector store N-element structure from one lane.`。
- **L765 EN**: Comment explains nearby logic, invariants, or intent: `Source operands are: the address, the N vectors, the lane number, and`.
  **L765 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Source operands are: the address, the N vectors, the lane number, and`。
- **L766 EN**: Comment explains nearby logic, invariants, or intent: `the alignment.`.
  **L766 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the alignment.`。
- **L767 EN**: Declares TableGen def `int_arm_neon_vst2lane`.
  **L767 CN**: 声明 TableGen def `int_arm_neon_vst2lane`。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。

### Lines 769-792

````tablegen
    [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty,
     llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly]>;
def int_arm_neon_vst3lane : DefaultAttrsIntrinsic<
    [],
    [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,
     llvm_i32_ty, llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly]>;
def int_arm_neon_vst4lane : DefaultAttrsIntrinsic<
    [],
    [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,
     LLVMMatchType<1>, llvm_i32_ty, llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly]>;

// Vector bitwise select.
def int_arm_neon_vbsl : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
    [IntrNoMem]>;


// Crypto instructions
class AES_1Arg_Intrinsic : DefaultAttrsIntrinsic<
    [llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
class AES_2Arg_Intrinsic : DefaultAttrsIntrinsic<
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty,`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty,`。
- **L770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L770 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L771 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L771 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L772 EN**: Declares TableGen def `int_arm_neon_vst3lane`.
  **L772 CN**: 声明 TableGen def `int_arm_neon_vst3lane`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`。
- **L775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L775 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L776 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L776 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L777 EN**: Declares TableGen def `int_arm_neon_vst4lane`.
  **L777 CN**: 声明 TableGen def `int_arm_neon_vst4lane`。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`。
- **L780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, llvm_i32_ty, llvm_i32_ty],`.
  **L780 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, llvm_i32_ty, llvm_i32_ty],`。
- **L781 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L781 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L783 EN**: Comment explains nearby logic, invariants, or intent: `Vector bitwise select.`.
  **L783 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector bitwise select.`。
- **L784 EN**: Declares TableGen def `int_arm_neon_vbsl`.
  **L784 CN**: 声明 TableGen def `int_arm_neon_vbsl`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L786 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L786 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L787 EN**: Blank line separating nearby declarations or logic blocks.
  **L787 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L789 EN**: Comment explains nearby logic, invariants, or intent: `Crypto instructions`.
  **L789 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Crypto instructions`。
- **L790 EN**: Declares class `AES_1Arg_Intrinsic`.
  **L790 CN**: 声明 class `AES_1Arg_Intrinsic`。
- **L791 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L791 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L792 EN**: Declares class `AES_2Arg_Intrinsic`.
  **L792 CN**: 声明 class `AES_2Arg_Intrinsic`。

### Lines 793-816

````tablegen
    [llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;

class SHA_1Arg_Intrinsic : DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;
class SHA_2Arg_Intrinsic : DefaultAttrsIntrinsic<
    [llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
class SHA_3Arg_i32_Intrinsic : DefaultAttrsIntrinsic<
    [llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
class SHA_3Arg_v4i32_Intrinsic : DefaultAttrsIntrinsic<
    [llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty,llvm_v4i32_ty], [IntrNoMem]>;

def int_arm_neon_aesd : AES_2Arg_Intrinsic;
def int_arm_neon_aese : AES_2Arg_Intrinsic;
def int_arm_neon_aesimc : AES_1Arg_Intrinsic;
def int_arm_neon_aesmc : AES_1Arg_Intrinsic;
def int_arm_neon_sha1h : SHA_1Arg_Intrinsic;
def int_arm_neon_sha1su1 : SHA_2Arg_Intrinsic;
def int_arm_neon_sha256su0 : SHA_2Arg_Intrinsic;
def int_arm_neon_sha1c : SHA_3Arg_i32_Intrinsic;
def int_arm_neon_sha1m : SHA_3Arg_i32_Intrinsic;
def int_arm_neon_sha1p : SHA_3Arg_i32_Intrinsic;
def int_arm_neon_sha1su0: SHA_3Arg_v4i32_Intrinsic;
def int_arm_neon_sha256h: SHA_3Arg_v4i32_Intrinsic;
def int_arm_neon_sha256h2: SHA_3Arg_v4i32_Intrinsic;
````
- **L793 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L793 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L795 EN**: Declares class `SHA_1Arg_Intrinsic`.
  **L795 CN**: 声明 class `SHA_1Arg_Intrinsic`。
- **L796 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L796 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L797 EN**: Declares class `SHA_2Arg_Intrinsic`.
  **L797 CN**: 声明 class `SHA_2Arg_Intrinsic`。
- **L798 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L798 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L799 EN**: Declares class `SHA_3Arg_i32_Intrinsic`.
  **L799 CN**: 声明 class `SHA_3Arg_i32_Intrinsic`。
- **L800 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L800 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L801 EN**: Declares class `SHA_3Arg_v4i32_Intrinsic`.
  **L801 CN**: 声明 class `SHA_3Arg_v4i32_Intrinsic`。
- **L802 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty,llvm_v4i32_ty], [IntrNoMem]>;`.
  **L802 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty,llvm_v4i32_ty], [IntrNoMem]>;`。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L804 EN**: Declares TableGen def `int_arm_neon_aesd`.
  **L804 CN**: 声明 TableGen def `int_arm_neon_aesd`。
- **L805 EN**: Declares TableGen def `int_arm_neon_aese`.
  **L805 CN**: 声明 TableGen def `int_arm_neon_aese`。
- **L806 EN**: Declares TableGen def `int_arm_neon_aesimc`.
  **L806 CN**: 声明 TableGen def `int_arm_neon_aesimc`。
- **L807 EN**: Declares TableGen def `int_arm_neon_aesmc`.
  **L807 CN**: 声明 TableGen def `int_arm_neon_aesmc`。
- **L808 EN**: Declares TableGen def `int_arm_neon_sha1h`.
  **L808 CN**: 声明 TableGen def `int_arm_neon_sha1h`。
- **L809 EN**: Declares TableGen def `int_arm_neon_sha1su1`.
  **L809 CN**: 声明 TableGen def `int_arm_neon_sha1su1`。
- **L810 EN**: Declares TableGen def `int_arm_neon_sha256su0`.
  **L810 CN**: 声明 TableGen def `int_arm_neon_sha256su0`。
- **L811 EN**: Declares TableGen def `int_arm_neon_sha1c`.
  **L811 CN**: 声明 TableGen def `int_arm_neon_sha1c`。
- **L812 EN**: Declares TableGen def `int_arm_neon_sha1m`.
  **L812 CN**: 声明 TableGen def `int_arm_neon_sha1m`。
- **L813 EN**: Declares TableGen def `int_arm_neon_sha1p`.
  **L813 CN**: 声明 TableGen def `int_arm_neon_sha1p`。
- **L814 EN**: Declares TableGen def `int_arm_neon_sha1su0`.
  **L814 CN**: 声明 TableGen def `int_arm_neon_sha1su0`。
- **L815 EN**: Declares TableGen def `int_arm_neon_sha256h`.
  **L815 CN**: 声明 TableGen def `int_arm_neon_sha256h`。
- **L816 EN**: Declares TableGen def `int_arm_neon_sha256h2`.
  **L816 CN**: 声明 TableGen def `int_arm_neon_sha256h2`。

### Lines 817-840

````tablegen
def int_arm_neon_sha256su1: SHA_3Arg_v4i32_Intrinsic;

def int_arm_neon_vqrdmlah : Neon_3Arg_Intrinsic;
def int_arm_neon_vqrdmlsh : Neon_3Arg_Intrinsic;

// Armv8.2-A dot product instructions
class Neon_Dot_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, llvm_anyvector_ty,
                           LLVMMatchType<1>],
                          [IntrNoMem]>;
def int_arm_neon_udot : Neon_Dot_Intrinsic;
def int_arm_neon_sdot : Neon_Dot_Intrinsic;

// v8.6-A Matrix Multiply Intrinsics
class Neon_MatMul_Intrinsic
  : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
                          [LLVMMatchType<0>, llvm_anyvector_ty,
                           LLVMMatchType<1>],
                          [IntrNoMem]>;
def int_arm_neon_ummla  : Neon_MatMul_Intrinsic;
def int_arm_neon_smmla  : Neon_MatMul_Intrinsic;
def int_arm_neon_usmmla : Neon_MatMul_Intrinsic;
def int_arm_neon_usdot  : Neon_Dot_Intrinsic;
````
- **L817 EN**: Declares TableGen def `int_arm_neon_sha256su1`.
  **L817 CN**: 声明 TableGen def `int_arm_neon_sha256su1`。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L819 EN**: Declares TableGen def `int_arm_neon_vqrdmlah`.
  **L819 CN**: 声明 TableGen def `int_arm_neon_vqrdmlah`。
- **L820 EN**: Declares TableGen def `int_arm_neon_vqrdmlsh`.
  **L820 CN**: 声明 TableGen def `int_arm_neon_vqrdmlsh`。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L822 EN**: Comment explains nearby logic, invariants, or intent: `Armv8.2-A dot product instructions`.
  **L822 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Armv8.2-A dot product instructions`。
- **L823 EN**: Declares class `Neon_Dot_Intrinsic`.
  **L823 CN**: 声明 class `Neon_Dot_Intrinsic`。
- **L824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L824 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L825 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L826 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L827 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L827 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L828 EN**: Declares TableGen def `int_arm_neon_udot`.
  **L828 CN**: 声明 TableGen def `int_arm_neon_udot`。
- **L829 EN**: Declares TableGen def `int_arm_neon_sdot`.
  **L829 CN**: 声明 TableGen def `int_arm_neon_sdot`。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L831 EN**: Comment explains nearby logic, invariants, or intent: `v8.6-A Matrix Multiply Intrinsics`.
  **L831 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.6-A Matrix Multiply Intrinsics`。
- **L832 EN**: Declares class `Neon_MatMul_Intrinsic`.
  **L832 CN**: 声明 class `Neon_MatMul_Intrinsic`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty],`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>],`.
  **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>],`。
- **L836 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L836 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L837 EN**: Declares TableGen def `int_arm_neon_ummla`.
  **L837 CN**: 声明 TableGen def `int_arm_neon_ummla`。
- **L838 EN**: Declares TableGen def `int_arm_neon_smmla`.
  **L838 CN**: 声明 TableGen def `int_arm_neon_smmla`。
- **L839 EN**: Declares TableGen def `int_arm_neon_usmmla`.
  **L839 CN**: 声明 TableGen def `int_arm_neon_usmmla`。
- **L840 EN**: Declares TableGen def `int_arm_neon_usdot`.
  **L840 CN**: 声明 TableGen def `int_arm_neon_usdot`。

### Lines 841-864

````tablegen

// v8.6-A Bfloat Intrinsics
def int_arm_neon_vcvtfp2bf
    : DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_arm_neon_vcvtbfp2bf
    : DefaultAttrsIntrinsic<[llvm_bfloat_ty], [llvm_float_ty], [IntrNoMem]>;

def int_arm_neon_bfdot : Neon_Dot_Intrinsic;
def int_arm_neon_bfmmla
    : DefaultAttrsIntrinsic<[llvm_v4f32_ty],
                            [llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],
                            [IntrNoMem]>;

class Neon_BF16FML_Intrinsic
    : DefaultAttrsIntrinsic<[llvm_v4f32_ty],
                            [llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],
                            [IntrNoMem]>;
def int_arm_neon_bfmlalb : Neon_BF16FML_Intrinsic;
def int_arm_neon_bfmlalt : Neon_BF16FML_Intrinsic;

def int_arm_cls: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty],
                                       [IntrNoMem]>;
def int_arm_cls64: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty],
                                         [IntrNoMem]>;
````
- **L841 EN**: Blank line separating nearby declarations or logic blocks.
  **L841 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L842 EN**: Comment explains nearby logic, invariants, or intent: `v8.6-A Bfloat Intrinsics`.
  **L842 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.6-A Bfloat Intrinsics`。
- **L843 EN**: Declares TableGen def `int_arm_neon_vcvtfp2bf`.
  **L843 CN**: 声明 TableGen def `int_arm_neon_vcvtfp2bf`。
- **L844 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L844 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyvector_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L845 EN**: Declares TableGen def `int_arm_neon_vcvtbfp2bf`.
  **L845 CN**: 声明 TableGen def `int_arm_neon_vcvtbfp2bf`。
- **L846 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_bfloat_ty], [llvm_float_ty], [IntrNoMem]>;`.
  **L846 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_bfloat_ty], [llvm_float_ty], [IntrNoMem]>;`。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L848 EN**: Declares TableGen def `int_arm_neon_bfdot`.
  **L848 CN**: 声明 TableGen def `int_arm_neon_bfdot`。
- **L849 EN**: Declares TableGen def `int_arm_neon_bfmmla`.
  **L849 CN**: 声明 TableGen def `int_arm_neon_bfmmla`。
- **L850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`.
  **L850 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`。
- **L852 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L852 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L853 EN**: Blank line separating nearby declarations or logic blocks.
  **L853 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L854 EN**: Declares class `Neon_BF16FML_Intrinsic`.
  **L854 CN**: 声明 class `Neon_BF16FML_Intrinsic`。
- **L855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`.
  **L855 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_v4f32_ty],`。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v8bf16_ty, llvm_v8bf16_ty],`。
- **L857 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L857 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L858 EN**: Declares TableGen def `int_arm_neon_bfmlalb`.
  **L858 CN**: 声明 TableGen def `int_arm_neon_bfmlalb`。
- **L859 EN**: Declares TableGen def `int_arm_neon_bfmlalt`.
  **L859 CN**: 声明 TableGen def `int_arm_neon_bfmlalt`。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L861 EN**: Declares TableGen def `int_arm_cls`.
  **L861 CN**: 声明 TableGen def `int_arm_cls`。
- **L862 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L862 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L863 EN**: Declares TableGen def `int_arm_cls64`.
  **L863 CN**: 声明 TableGen def `int_arm_cls64`。
- **L864 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L864 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 865-888

````tablegen

def int_arm_mve_vctp8  : DefaultAttrsIntrinsic<[llvm_v16i1_ty], [llvm_i32_ty],
                                               [IntrNoMem]>;
def int_arm_mve_vctp16 : DefaultAttrsIntrinsic<[llvm_v8i1_ty], [llvm_i32_ty],
                                               [IntrNoMem]>;
def int_arm_mve_vctp32 : DefaultAttrsIntrinsic<[llvm_v4i1_ty], [llvm_i32_ty],
                                               [IntrNoMem]>;
def int_arm_mve_vctp64 : DefaultAttrsIntrinsic<[llvm_v2i1_ty], [llvm_i32_ty],
                                               [IntrNoMem]>;

// v8.3-A Floating-point complex add
def int_arm_neon_vcadd_rot90  : Neon_2Arg_Intrinsic;
def int_arm_neon_vcadd_rot270 : Neon_2Arg_Intrinsic;

// GNU eabi mcount
// TODO: Add applicable default attributes.
def int_arm_gnu_eabi_mcount : Intrinsic<[], [], []>;

def int_arm_mve_pred_i2v : DefaultAttrsIntrinsic<
  [llvm_anyvector_ty], [llvm_i32_ty], [IntrNoMem]>;
def int_arm_mve_pred_v2i : DefaultAttrsIntrinsic<
  [llvm_i32_ty], [llvm_anyvector_ty], [IntrNoMem]>;
def int_arm_mve_vreinterpretq : DefaultAttrsIntrinsic<
  [llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;
````
- **L865 EN**: Blank line separating nearby declarations or logic blocks.
  **L865 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L866 EN**: Declares TableGen def `int_arm_mve_vctp8`.
  **L866 CN**: 声明 TableGen def `int_arm_mve_vctp8`。
- **L867 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L867 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L868 EN**: Declares TableGen def `int_arm_mve_vctp16`.
  **L868 CN**: 声明 TableGen def `int_arm_mve_vctp16`。
- **L869 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L869 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L870 EN**: Declares TableGen def `int_arm_mve_vctp32`.
  **L870 CN**: 声明 TableGen def `int_arm_mve_vctp32`。
- **L871 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L871 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L872 EN**: Declares TableGen def `int_arm_mve_vctp64`.
  **L872 CN**: 声明 TableGen def `int_arm_mve_vctp64`。
- **L873 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L873 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L874 EN**: Blank line separating nearby declarations or logic blocks.
  **L874 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L875 EN**: Comment explains nearby logic, invariants, or intent: `v8.3-A Floating-point complex add`.
  **L875 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`v8.3-A Floating-point complex add`。
- **L876 EN**: Declares TableGen def `int_arm_neon_vcadd_rot90`.
  **L876 CN**: 声明 TableGen def `int_arm_neon_vcadd_rot90`。
- **L877 EN**: Declares TableGen def `int_arm_neon_vcadd_rot270`.
  **L877 CN**: 声明 TableGen def `int_arm_neon_vcadd_rot270`。
- **L878 EN**: Blank line separating nearby declarations or logic blocks.
  **L878 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L879 EN**: Comment explains nearby logic, invariants, or intent: `GNU eabi mcount`.
  **L879 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GNU eabi mcount`。
- **L880 EN**: Comment records a pending task or caution: `TODO: Add applicable default attributes.`.
  **L880 CN**: 注释记录了待办事项或注意点：`TODO: Add applicable default attributes.`。
- **L881 EN**: Declares TableGen def `int_arm_gnu_eabi_mcount`.
  **L881 CN**: 声明 TableGen def `int_arm_gnu_eabi_mcount`。
- **L882 EN**: Blank line separating nearby declarations or logic blocks.
  **L882 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L883 EN**: Declares TableGen def `int_arm_mve_pred_i2v`.
  **L883 CN**: 声明 TableGen def `int_arm_mve_pred_i2v`。
- **L884 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L884 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L885 EN**: Declares TableGen def `int_arm_mve_pred_v2i`.
  **L885 CN**: 声明 TableGen def `int_arm_mve_pred_v2i`。
- **L886 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L886 CN**: 执行一条独立语句或声明：`[llvm_i32_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。
- **L887 EN**: Declares TableGen def `int_arm_mve_vreinterpretq`.
  **L887 CN**: 声明 TableGen def `int_arm_mve_vreinterpretq`。
- **L888 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`.
  **L888 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [llvm_anyvector_ty], [IntrNoMem]>;`。

### Lines 889-912

````tablegen

def int_arm_mve_min_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_max_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_abd_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_add_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_and_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_bic_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_eor_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L890 EN**: Declares TableGen def `int_arm_mve_min_predicated`.
  **L890 CN**: 声明 TableGen def `int_arm_mve_min_predicated`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L893 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L893 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L894 EN**: Declares TableGen def `int_arm_mve_max_predicated`.
  **L894 CN**: 声明 TableGen def `int_arm_mve_max_predicated`。
- **L895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L895 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L897 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L897 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L898 EN**: Declares TableGen def `int_arm_mve_abd_predicated`.
  **L898 CN**: 声明 TableGen def `int_arm_mve_abd_predicated`。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L900 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L900 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L901 EN**: Declares TableGen def `int_arm_mve_add_predicated`.
  **L901 CN**: 声明 TableGen def `int_arm_mve_add_predicated`。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L903 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L903 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L904 EN**: Declares TableGen def `int_arm_mve_and_predicated`.
  **L904 CN**: 声明 TableGen def `int_arm_mve_and_predicated`。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L906 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L906 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L907 EN**: Declares TableGen def `int_arm_mve_bic_predicated`.
  **L907 CN**: 声明 TableGen def `int_arm_mve_bic_predicated`。
- **L908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L908 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L909 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L909 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L910 EN**: Declares TableGen def `int_arm_mve_eor_predicated`.
  **L910 CN**: 声明 TableGen def `int_arm_mve_eor_predicated`。
- **L911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L911 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L912 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L912 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 913-936

````tablegen
def int_arm_mve_orn_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_orr_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_sub_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_mul_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_mulh_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_qdmulh_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_rmulh_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_qrdmulh_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
````
- **L913 EN**: Declares TableGen def `int_arm_mve_orn_predicated`.
  **L913 CN**: 声明 TableGen def `int_arm_mve_orn_predicated`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L915 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L915 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L916 EN**: Declares TableGen def `int_arm_mve_orr_predicated`.
  **L916 CN**: 声明 TableGen def `int_arm_mve_orr_predicated`。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L918 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L918 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L919 EN**: Declares TableGen def `int_arm_mve_sub_predicated`.
  **L919 CN**: 声明 TableGen def `int_arm_mve_sub_predicated`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L921 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L921 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L922 EN**: Declares TableGen def `int_arm_mve_mul_predicated`.
  **L922 CN**: 声明 TableGen def `int_arm_mve_mul_predicated`。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L924 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L924 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L925 EN**: Declares TableGen def `int_arm_mve_mulh_predicated`.
  **L925 CN**: 声明 TableGen def `int_arm_mve_mulh_predicated`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L928 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L928 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L929 EN**: Declares TableGen def `int_arm_mve_qdmulh_predicated`.
  **L929 CN**: 声明 TableGen def `int_arm_mve_qdmulh_predicated`。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L931 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L931 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L932 EN**: Declares TableGen def `int_arm_mve_rmulh_predicated`.
  **L932 CN**: 声明 TableGen def `int_arm_mve_rmulh_predicated`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L935 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L935 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L936 EN**: Declares TableGen def `int_arm_mve_qrdmulh_predicated`.
  **L936 CN**: 声明 TableGen def `int_arm_mve_qrdmulh_predicated`。

### Lines 937-960

````tablegen
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_mull_int_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty /* unsigned */,
    llvm_i32_ty /* top */, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_mull_poly_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty, llvm_anyvector_ty,
    LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_qadd_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_hadd_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_rhadd_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_qsub_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
    llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_hsub_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,
````
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L938 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L938 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L939 EN**: Declares TableGen def `int_arm_mve_mull_int_predicated`.
  **L939 CN**: 声明 TableGen def `int_arm_mve_mull_int_predicated`。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty /* unsigned */,`.
  **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty /* unsigned */,`。
- **L941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /* top */, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L941 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /* top */, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L942 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L942 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L943 EN**: Declares TableGen def `int_arm_mve_mull_poly_predicated`.
  **L943 CN**: 声明 TableGen def `int_arm_mve_mull_poly_predicated`。
- **L944 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty, llvm_anyvector_ty,`.
  **L944 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty, llvm_anyvector_ty,`。
- **L945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L945 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L946 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L946 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L947 EN**: Declares TableGen def `int_arm_mve_qadd_predicated`.
  **L947 CN**: 声明 TableGen def `int_arm_mve_qadd_predicated`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L949 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L949 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L950 EN**: Declares TableGen def `int_arm_mve_hadd_predicated`.
  **L950 CN**: 声明 TableGen def `int_arm_mve_hadd_predicated`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L952 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L952 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L953 EN**: Declares TableGen def `int_arm_mve_rhadd_predicated`.
  **L953 CN**: 声明 TableGen def `int_arm_mve_rhadd_predicated`。
- **L954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L954 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L955 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L955 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L956 EN**: Declares TableGen def `int_arm_mve_qsub_predicated`.
  **L956 CN**: 声明 TableGen def `int_arm_mve_qsub_predicated`。
- **L957 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L957 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。
- **L958 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L958 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L959 EN**: Declares TableGen def `int_arm_mve_hsub_predicated`.
  **L959 CN**: 声明 TableGen def `int_arm_mve_hsub_predicated`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */,`。

### Lines 961-984

````tablegen
    llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_vmina_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],
    [IntrNoMem]>;
def int_arm_mve_vmaxa_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],
    [IntrNoMem]>;
def int_arm_mve_vminnma_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],
    [IntrNoMem]>;
def int_arm_mve_vmaxnma_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],
    [IntrNoMem]>;

def int_arm_mve_vminnm: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>],
    [IntrNoMem]>;
def int_arm_mve_vmaxnm: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>],
    [IntrNoMem]>;

multiclass MVEPredicated<list<LLVMType> rets, list<LLVMType> params,
                         LLVMType pred = llvm_anyvector_ty,
                         list<IntrinsicProperty> props = [IntrNoMem],
````
- **L961 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L961 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L962 EN**: Declares TableGen def `int_arm_mve_vmina_predicated`.
  **L962 CN**: 声明 TableGen def `int_arm_mve_vmina_predicated`。
- **L963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`.
  **L963 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`。
- **L964 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L964 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L965 EN**: Declares TableGen def `int_arm_mve_vmaxa_predicated`.
  **L965 CN**: 声明 TableGen def `int_arm_mve_vmaxa_predicated`。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`.
  **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`。
- **L967 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L967 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L968 EN**: Declares TableGen def `int_arm_mve_vminnma_predicated`.
  **L968 CN**: 声明 TableGen def `int_arm_mve_vminnma_predicated`。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`.
  **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`。
- **L970 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L970 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L971 EN**: Declares TableGen def `int_arm_mve_vmaxnma_predicated`.
  **L971 CN**: 声明 TableGen def `int_arm_mve_vmaxnma_predicated`。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`.
  **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_anyvector_ty],`。
- **L973 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L973 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L975 EN**: Declares TableGen def `int_arm_mve_vminnm`.
  **L975 CN**: 声明 TableGen def `int_arm_mve_vminnm`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L977 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L977 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L978 EN**: Declares TableGen def `int_arm_mve_vmaxnm`.
  **L978 CN**: 声明 TableGen def `int_arm_mve_vmaxnm`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L980 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L980 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L981 EN**: Blank line separating nearby declarations or logic blocks.
  **L981 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L982 EN**: Declares TableGen multiclass `MVEPredicated`.
  **L982 CN**: 声明 TableGen multiclass `MVEPredicated`。
- **L983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMType pred = llvm_anyvector_ty,`.
  **L983 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMType pred = llvm_anyvector_ty,`。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<IntrinsicProperty> props = [IntrNoMem],`.
  **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<IntrinsicProperty> props = [IntrNoMem],`。

### Lines 985-1008

````tablegen
                         list<SDNodeProperty> sdprops = []> {
  def "": DefaultAttrsIntrinsic<rets, params, props, "", sdprops>;
  def _predicated: DefaultAttrsIntrinsic<rets, params # [pred], props, "",
                                         sdprops>;
}
multiclass MVEPredicatedM<list<LLVMType> rets, list<LLVMType> params,
                          LLVMType pred = llvm_anyvector_ty,
                          list<IntrinsicProperty> props = [IntrNoMem]> {
  def "": DefaultAttrsIntrinsic<rets, params, props>;
  def _predicated: DefaultAttrsIntrinsic<rets, params # [pred,
      !if(!eq(rets[0], llvm_anyvector_ty),
          LLVMMatchType<0>, rets[0])], props>;
}

multiclass MVE_minmaxv {
  defm v: MVEPredicated<[llvm_i32_ty],
     [llvm_i32_ty, llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;
  defm av: MVEPredicated<[llvm_i32_ty],
     [llvm_i32_ty, llvm_anyvector_ty]>;
  defm nmv: MVEPredicated<[llvm_anyfloat_ty],
     [LLVMMatchType<0>, llvm_anyvector_ty]>;
  defm nmav: MVEPredicated<[llvm_anyfloat_ty],
     [LLVMMatchType<0>, llvm_anyvector_ty]>;
}
````
- **L985 EN**: Continues the surrounding expression or declaration: `list<SDNodeProperty> sdprops = []> {`.
  **L985 CN**: 继续构造周围的表达式或声明：`list<SDNodeProperty> sdprops = []> {`。
- **L986 EN**: Declares TableGen def `""`.
  **L986 CN**: 声明 TableGen def `""`。
- **L987 EN**: Declares TableGen def `_predicated`.
  **L987 CN**: 声明 TableGen def `_predicated`。
- **L988 EN**: Executes a standalone statement or declaration: `sdprops>;`.
  **L988 CN**: 执行一条独立语句或声明：`sdprops>;`。
- **L989 EN**: Closes the current lexical scope or compound statement.
  **L989 CN**: 结束当前词法作用域或复合语句块。
- **L990 EN**: Declares TableGen multiclass `MVEPredicatedM`.
  **L990 CN**: 声明 TableGen multiclass `MVEPredicatedM`。
- **L991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMType pred = llvm_anyvector_ty,`.
  **L991 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMType pred = llvm_anyvector_ty,`。
- **L992 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> props = [IntrNoMem]> {`.
  **L992 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> props = [IntrNoMem]> {`。
- **L993 EN**: Declares TableGen def `""`.
  **L993 CN**: 声明 TableGen def `""`。
- **L994 EN**: Declares TableGen def `_predicated`.
  **L994 CN**: 声明 TableGen def `_predicated`。
- **L995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(rets[0], llvm_anyvector_ty),`.
  **L995 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(rets[0], llvm_anyvector_ty),`。
- **L996 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, rets[0])], props>;`.
  **L996 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, rets[0])], props>;`。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L999 EN**: Declares TableGen multiclass `MVE_minmaxv`.
  **L999 CN**: 声明 TableGen multiclass `MVE_minmaxv`。
- **L1000 EN**: Declares TableGen defm `v`.
  **L1000 CN**: 声明 TableGen defm `v`。
- **L1001 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;`.
  **L1001 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;`。
- **L1002 EN**: Declares TableGen defm `av`.
  **L1002 CN**: 声明 TableGen defm `av`。
- **L1003 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_anyvector_ty]>;`.
  **L1003 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_anyvector_ty]>;`。
- **L1004 EN**: Declares TableGen defm `nmv`.
  **L1004 CN**: 声明 TableGen defm `nmv`。
- **L1005 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_anyvector_ty]>;`.
  **L1005 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_anyvector_ty]>;`。
- **L1006 EN**: Declares TableGen defm `nmav`.
  **L1006 CN**: 声明 TableGen defm `nmav`。
- **L1007 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_anyvector_ty]>;`.
  **L1007 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_anyvector_ty]>;`。
- **L1008 EN**: Closes the current lexical scope or compound statement.
  **L1008 CN**: 结束当前词法作用域或复合语句块。

### Lines 1009-1032

````tablegen
defm int_arm_mve_min: MVE_minmaxv;
defm int_arm_mve_max: MVE_minmaxv;

defm int_arm_mve_addv: MVEPredicated<[llvm_i32_ty],
   [llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;
defm int_arm_mve_addlv: MVEPredicated<[llvm_i64_ty],
   [llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;

// Intrinsic with a predicated and a non-predicated case. The predicated case
// has two additional parameters: inactive (the value for inactive lanes, can
// be undef) and predicate.
multiclass MVEMXPredicated<list<LLVMType> rets, list<LLVMType> flags,
                           list<LLVMType> params, LLVMType inactive,
                           LLVMType predicate,
                           list<IntrinsicProperty> props = [IntrNoMem]> {
  def "":          DefaultAttrsIntrinsic<rets, flags # params, props>;
  def _predicated: DefaultAttrsIntrinsic<
      rets, flags # [inactive] # params # [predicate], props>;
}

defm int_arm_mve_vcvt_narrow: MVEPredicated<[llvm_v8f16_ty],
   [llvm_v8f16_ty, llvm_v4f32_ty, llvm_i32_ty], llvm_v4i1_ty>;
defm int_arm_mve_vcvt_widen: MVEMXPredicated<[llvm_v4f32_ty], [],
   [llvm_v8f16_ty, llvm_i32_ty], llvm_v4f32_ty, llvm_v4i1_ty>;
````
- **L1009 EN**: Declares TableGen defm `int_arm_mve_min`.
  **L1009 CN**: 声明 TableGen defm `int_arm_mve_min`。
- **L1010 EN**: Declares TableGen defm `int_arm_mve_max`.
  **L1010 CN**: 声明 TableGen defm `int_arm_mve_max`。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1012 EN**: Declares TableGen defm `int_arm_mve_addv`.
  **L1012 CN**: 声明 TableGen defm `int_arm_mve_addv`。
- **L1013 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;`.
  **L1013 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;`。
- **L1014 EN**: Declares TableGen defm `int_arm_mve_addlv`.
  **L1014 CN**: 声明 TableGen defm `int_arm_mve_addlv`。
- **L1015 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;`.
  **L1015 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty, llvm_i32_ty /* unsigned */]>;`。
- **L1016 EN**: Blank line separating nearby declarations or logic blocks.
  **L1016 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1017 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsic with a predicated and a non-predicated case. The predicated case`.
  **L1017 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsic with a predicated and a non-predicated case. The predicated case`。
- **L1018 EN**: Comment explains nearby logic, invariants, or intent: `has two additional parameters: inactive (the value for inactive lanes, can`.
  **L1018 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`has two additional parameters: inactive (the value for inactive lanes, can`。
- **L1019 EN**: Comment explains nearby logic, invariants, or intent: `be undef) and predicate.`.
  **L1019 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`be undef) and predicate.`。
- **L1020 EN**: Declares TableGen multiclass `MVEMXPredicated`.
  **L1020 CN**: 声明 TableGen multiclass `MVEMXPredicated`。
- **L1021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> params, LLVMType inactive,`.
  **L1021 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> params, LLVMType inactive,`。
- **L1022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMType predicate,`.
  **L1022 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMType predicate,`。
- **L1023 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> props = [IntrNoMem]> {`.
  **L1023 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> props = [IntrNoMem]> {`。
- **L1024 EN**: Declares TableGen def `""`.
  **L1024 CN**: 声明 TableGen def `""`。
- **L1025 EN**: Declares TableGen def `_predicated`.
  **L1025 CN**: 声明 TableGen def `_predicated`。
- **L1026 EN**: Executes a standalone statement or declaration: `rets, flags # [inactive] # params # [predicate], props>;`.
  **L1026 CN**: 执行一条独立语句或声明：`rets, flags # [inactive] # params # [predicate], props>;`。
- **L1027 EN**: Closes the current lexical scope or compound statement.
  **L1027 CN**: 结束当前词法作用域或复合语句块。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1029 EN**: Declares TableGen defm `int_arm_mve_vcvt_narrow`.
  **L1029 CN**: 声明 TableGen defm `int_arm_mve_vcvt_narrow`。
- **L1030 EN**: Executes a standalone statement or declaration: `[llvm_v8f16_ty, llvm_v4f32_ty, llvm_i32_ty], llvm_v4i1_ty>;`.
  **L1030 CN**: 执行一条独立语句或声明：`[llvm_v8f16_ty, llvm_v4f32_ty, llvm_i32_ty], llvm_v4i1_ty>;`。
- **L1031 EN**: Declares TableGen defm `int_arm_mve_vcvt_widen`.
  **L1031 CN**: 声明 TableGen defm `int_arm_mve_vcvt_widen`。
- **L1032 EN**: Executes a standalone statement or declaration: `[llvm_v8f16_ty, llvm_i32_ty], llvm_v4f32_ty, llvm_v4i1_ty>;`.
  **L1032 CN**: 执行一条独立语句或声明：`[llvm_v8f16_ty, llvm_i32_ty], llvm_v4f32_ty, llvm_v4i1_ty>;`。

### Lines 1033-1056

````tablegen

defm int_arm_mve_vldr_gather_base: MVEPredicated<
   [llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty],
   llvm_anyvector_ty, [IntrReadMem], [SDNPMemOperand]>;
defm int_arm_mve_vldr_gather_base_wb: MVEPredicated<
   [llvm_anyvector_ty, llvm_anyvector_ty],
   [LLVMMatchType<1>, llvm_i32_ty], llvm_anyvector_ty, [IntrReadMem],
   [SDNPMemOperand]>;
defm int_arm_mve_vstr_scatter_base: MVEPredicated<
   [], [llvm_anyvector_ty, llvm_i32_ty, llvm_anyvector_ty],
   llvm_anyvector_ty, [IntrWriteMem], [SDNPMemOperand]>;
defm int_arm_mve_vstr_scatter_base_wb: MVEPredicated<
   [llvm_anyvector_ty], [LLVMMatchType<0>, llvm_i32_ty, llvm_anyvector_ty],
   llvm_anyvector_ty, [IntrWriteMem], [SDNPMemOperand]>;

// gather_offset takes three i32 parameters. The first is the size of
// memory element loaded, in bits. The second is a left bit shift to
// apply to each offset in the vector parameter (must be either 0, or
// correspond to the element size of the destination vector type). The
// last is 1 to indicate zero extension (if the load is widening), or
// 0 for sign extension.
//
// scatter_offset has the first two of those parameters, but since it
// narrows rather than widening, it doesn't have the last one.
````
- **L1033 EN**: Blank line separating nearby declarations or logic blocks.
  **L1033 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1034 EN**: Declares TableGen defm `int_arm_mve_vldr_gather_base`.
  **L1034 CN**: 声明 TableGen defm `int_arm_mve_vldr_gather_base`。
- **L1035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty],`.
  **L1035 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty],`。
- **L1036 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, [IntrReadMem], [SDNPMemOperand]>;`.
  **L1036 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, [IntrReadMem], [SDNPMemOperand]>;`。
- **L1037 EN**: Declares TableGen defm `int_arm_mve_vldr_gather_base_wb`.
  **L1037 CN**: 声明 TableGen defm `int_arm_mve_vldr_gather_base_wb`。
- **L1038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_anyvector_ty],`.
  **L1038 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_anyvector_ty],`。
- **L1039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<1>, llvm_i32_ty], llvm_anyvector_ty, [IntrReadMem],`.
  **L1039 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<1>, llvm_i32_ty], llvm_anyvector_ty, [IntrReadMem],`。
- **L1040 EN**: Executes a standalone statement or declaration: `[SDNPMemOperand]>;`.
  **L1040 CN**: 执行一条独立语句或声明：`[SDNPMemOperand]>;`。
- **L1041 EN**: Declares TableGen defm `int_arm_mve_vstr_scatter_base`.
  **L1041 CN**: 声明 TableGen defm `int_arm_mve_vstr_scatter_base`。
- **L1042 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_anyvector_ty, llvm_i32_ty, llvm_anyvector_ty],`.
  **L1042 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_anyvector_ty, llvm_i32_ty, llvm_anyvector_ty],`。
- **L1043 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, [IntrWriteMem], [SDNPMemOperand]>;`.
  **L1043 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, [IntrWriteMem], [SDNPMemOperand]>;`。
- **L1044 EN**: Declares TableGen defm `int_arm_mve_vstr_scatter_base_wb`.
  **L1044 CN**: 声明 TableGen defm `int_arm_mve_vstr_scatter_base_wb`。
- **L1045 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_i32_ty, llvm_anyvector_ty],`.
  **L1045 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_i32_ty, llvm_anyvector_ty],`。
- **L1046 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, [IntrWriteMem], [SDNPMemOperand]>;`.
  **L1046 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, [IntrWriteMem], [SDNPMemOperand]>;`。
- **L1047 EN**: Blank line separating nearby declarations or logic blocks.
  **L1047 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1048 EN**: Comment explains nearby logic, invariants, or intent: `gather_offset takes three i32 parameters. The first is the size of`.
  **L1048 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gather_offset takes three i32 parameters. The first is the size of`。
- **L1049 EN**: Comment explains nearby logic, invariants, or intent: `memory element loaded, in bits. The second is a left bit shift to`.
  **L1049 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`memory element loaded, in bits. The second is a left bit shift to`。
- **L1050 EN**: Comment explains nearby logic, invariants, or intent: `apply to each offset in the vector parameter (must be either 0, or`.
  **L1050 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`apply to each offset in the vector parameter (must be either 0, or`。
- **L1051 EN**: Comment explains nearby logic, invariants, or intent: `correspond to the element size of the destination vector type). The`.
  **L1051 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`correspond to the element size of the destination vector type). The`。
- **L1052 EN**: Comment explains nearby logic, invariants, or intent: `last is 1 to indicate zero extension (if the load is widening), or`.
  **L1052 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`last is 1 to indicate zero extension (if the load is widening), or`。
- **L1053 EN**: Comment explains nearby logic, invariants, or intent: `0 for sign extension.`.
  **L1053 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`0 for sign extension.`。
- **L1054 EN**: Separator comment used for visual grouping.
  **L1054 CN**: 用于视觉分组的分隔注释。
- **L1055 EN**: Comment explains nearby logic, invariants, or intent: `scatter_offset has the first two of those parameters, but since it`.
  **L1055 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scatter_offset has the first two of those parameters, but since it`。
- **L1056 EN**: Comment explains nearby logic, invariants, or intent: `narrows rather than widening, it doesn't have the last one.`.
  **L1056 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`narrows rather than widening, it doesn't have the last one.`。

### Lines 1057-1080

````tablegen
defm int_arm_mve_vldr_gather_offset: MVEPredicated<
   [llvm_anyvector_ty], [llvm_anyptr_ty, llvm_anyvector_ty,
   llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], llvm_anyvector_ty, [IntrReadMem],
   [SDNPMemOperand]>;
defm int_arm_mve_vstr_scatter_offset: MVEPredicated<
   [], [llvm_anyptr_ty, llvm_anyvector_ty, llvm_anyvector_ty,
   llvm_i32_ty, llvm_i32_ty], llvm_anyvector_ty, [IntrWriteMem],
   [SDNPMemOperand]>;

def int_arm_mve_shl_imm_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_shr_imm_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, // extra i32 is unsigned flag
    llvm_anyvector_ty, LLVMMatchType<0>],
   [IntrNoMem]>;

defm int_arm_mve_vqshl_imm: MVEPredicatedM<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/]>;
defm int_arm_mve_vrshr_imm: MVEPredicatedM<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/]>;
defm int_arm_mve_vqshlu_imm: MVEPredicatedM<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/]>;
defm int_arm_mve_vshll_imm: MVEPredicatedM<[llvm_anyvector_ty],
````
- **L1057 EN**: Declares TableGen defm `int_arm_mve_vldr_gather_offset`.
  **L1057 CN**: 声明 TableGen defm `int_arm_mve_vldr_gather_offset`。
- **L1058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [llvm_anyptr_ty, llvm_anyvector_ty,`.
  **L1058 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [llvm_anyptr_ty, llvm_anyvector_ty,`。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], llvm_anyvector_ty, [IntrReadMem],`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], llvm_anyvector_ty, [IntrReadMem],`。
- **L1060 EN**: Executes a standalone statement or declaration: `[SDNPMemOperand]>;`.
  **L1060 CN**: 执行一条独立语句或声明：`[SDNPMemOperand]>;`。
- **L1061 EN**: Declares TableGen defm `int_arm_mve_vstr_scatter_offset`.
  **L1061 CN**: 声明 TableGen defm `int_arm_mve_vstr_scatter_offset`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_anyptr_ty, llvm_anyvector_ty, llvm_anyvector_ty,`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_anyptr_ty, llvm_anyvector_ty, llvm_anyvector_ty,`。
- **L1063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty], llvm_anyvector_ty, [IntrWriteMem],`.
  **L1063 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty], llvm_anyvector_ty, [IntrWriteMem],`。
- **L1064 EN**: Executes a standalone statement or declaration: `[SDNPMemOperand]>;`.
  **L1064 CN**: 执行一条独立语句或声明：`[SDNPMemOperand]>;`。
- **L1065 EN**: Blank line separating nearby declarations or logic blocks.
  **L1065 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1066 EN**: Declares TableGen def `int_arm_mve_shl_imm_predicated`.
  **L1066 CN**: 声明 TableGen def `int_arm_mve_shl_imm_predicated`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1068 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1068 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1069 EN**: Declares TableGen def `int_arm_mve_shr_imm_predicated`.
  **L1069 CN**: 声明 TableGen def `int_arm_mve_shr_imm_predicated`。
- **L1070 EN**: Continues the surrounding expression or declaration: `[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, // extra i32 is unsigned flag`.
  **L1070 CN**: 继续构造周围的表达式或声明：`[LLVMMatchType<0>, llvm_i32_ty, llvm_i32_ty, // extra i32 is unsigned flag`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1072 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1072 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1073 EN**: Blank line separating nearby declarations or logic blocks.
  **L1073 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1074 EN**: Declares TableGen defm `int_arm_mve_vqshl_imm`.
  **L1074 CN**: 声明 TableGen defm `int_arm_mve_vqshl_imm`。
- **L1075 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/]>;`.
  **L1075 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/]>;`。
- **L1076 EN**: Declares TableGen defm `int_arm_mve_vrshr_imm`.
  **L1076 CN**: 声明 TableGen defm `int_arm_mve_vrshr_imm`。
- **L1077 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/]>;`.
  **L1077 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/]>;`。
- **L1078 EN**: Declares TableGen defm `int_arm_mve_vqshlu_imm`.
  **L1078 CN**: 声明 TableGen defm `int_arm_mve_vqshlu_imm`。
- **L1079 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/]>;`.
  **L1079 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/]>;`。
- **L1080 EN**: Declares TableGen defm `int_arm_mve_vshll_imm`.
  **L1080 CN**: 声明 TableGen defm `int_arm_mve_vshll_imm`。

### Lines 1081-1104

````tablegen
   [llvm_anyvector_ty, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/,
                       llvm_i32_ty /*top-half*/]>;

defm int_arm_mve_vsli: MVEPredicated<
   [llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty]>;
defm int_arm_mve_vsri: MVEPredicated<
   [llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty]>;

defm int_arm_mve_vshrn: MVEPredicated<
   [llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty,
    llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/,
    llvm_i32_ty /*unsigned-out*/, llvm_i32_ty /*unsigned-in*/,
    llvm_i32_ty /*top-half*/]>;

defm int_arm_mve_vshl_scalar: MVEPredicated<
   [llvm_anyvector_ty], [LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/,
    llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/, llvm_i32_ty /*unsigned*/]>;
defm int_arm_mve_vshl_vector: MVEPredicatedM<
   [llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty /*shiftcounts*/,
    llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/, llvm_i32_ty /*unsigned*/]>;

// MVE scalar shifts.
class ARM_MVE_qrshift_single<list<LLVMType> value,
                             list<LLVMType> saturate = []> :
````
- **L1081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/,`.
  **L1081 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*unsigned*/,`。
- **L1082 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /*top-half*/]>;`.
  **L1082 CN**: 执行一条独立语句或声明：`llvm_i32_ty /*top-half*/]>;`。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1084 EN**: Declares TableGen defm `int_arm_mve_vsli`.
  **L1084 CN**: 声明 TableGen defm `int_arm_mve_vsli`。
- **L1085 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty]>;`.
  **L1085 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty]>;`。
- **L1086 EN**: Declares TableGen defm `int_arm_mve_vsri`.
  **L1086 CN**: 声明 TableGen defm `int_arm_mve_vsri`。
- **L1087 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty]>;`.
  **L1087 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty]>;`。
- **L1088 EN**: Blank line separating nearby declarations or logic blocks.
  **L1088 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1089 EN**: Declares TableGen defm `int_arm_mve_vshrn`.
  **L1089 CN**: 声明 TableGen defm `int_arm_mve_vshrn`。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/,`.
  **L1091 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /*shiftcount*/, llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/,`。
- **L1092 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /*unsigned-out*/, llvm_i32_ty /*unsigned-in*/,`.
  **L1092 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /*unsigned-out*/, llvm_i32_ty /*unsigned-in*/,`。
- **L1093 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /*top-half*/]>;`.
  **L1093 CN**: 执行一条独立语句或声明：`llvm_i32_ty /*top-half*/]>;`。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1095 EN**: Declares TableGen defm `int_arm_mve_vshl_scalar`.
  **L1095 CN**: 声明 TableGen defm `int_arm_mve_vshl_scalar`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/,`.
  **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_i32_ty /*shiftcount*/,`。
- **L1097 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/, llvm_i32_ty /*unsigned*/]>;`.
  **L1097 CN**: 执行一条独立语句或声明：`llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/, llvm_i32_ty /*unsigned*/]>;`。
- **L1098 EN**: Declares TableGen defm `int_arm_mve_vshl_vector`.
  **L1098 CN**: 声明 TableGen defm `int_arm_mve_vshl_vector`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty /*shiftcounts*/,`.
  **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [LLVMMatchType<0>, llvm_anyvector_ty /*shiftcounts*/,`。
- **L1100 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/, llvm_i32_ty /*unsigned*/]>;`.
  **L1100 CN**: 执行一条独立语句或声明：`llvm_i32_ty /*saturate*/, llvm_i32_ty /*round*/, llvm_i32_ty /*unsigned*/]>;`。
- **L1101 EN**: Blank line separating nearby declarations or logic blocks.
  **L1101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1102 EN**: Comment explains nearby logic, invariants, or intent: `MVE scalar shifts.`.
  **L1102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MVE scalar shifts.`。
- **L1103 EN**: Declares class `ARM_MVE_qrshift_single<list<LLVMType>`.
  **L1103 CN**: 声明 class `ARM_MVE_qrshift_single<list<LLVMType>`。
- **L1104 EN**: Continues the surrounding expression or declaration: `list<LLVMType> saturate = []> :`.
  **L1104 CN**: 继续构造周围的表达式或声明：`list<LLVMType> saturate = []> :`。

### Lines 1105-1128

````tablegen
  DefaultAttrsIntrinsic<value, value # [llvm_i32_ty] # saturate, [IntrNoMem]>;
multiclass ARM_MVE_qrshift<list<LLVMType> saturate = []> {
  // Most of these shifts come in 32- and 64-bit versions. But only
  // the 64-bit ones have the extra saturation argument (if any).
  def "": ARM_MVE_qrshift_single<[llvm_i32_ty]>;
  def l:  ARM_MVE_qrshift_single<[llvm_i32_ty, llvm_i32_ty], saturate>;
}
defm int_arm_mve_urshr: ARM_MVE_qrshift;
defm int_arm_mve_uqshl: ARM_MVE_qrshift;
defm int_arm_mve_srshr: ARM_MVE_qrshift;
defm int_arm_mve_sqshl: ARM_MVE_qrshift;
defm int_arm_mve_uqrshl: ARM_MVE_qrshift<[llvm_i32_ty]>;
defm int_arm_mve_sqrshr: ARM_MVE_qrshift<[llvm_i32_ty]>;
// LSLL and ASRL only have 64-bit versions, not 32.
def int_arm_mve_lsll: ARM_MVE_qrshift_single<[llvm_i32_ty, llvm_i32_ty]>;
def int_arm_mve_asrl: ARM_MVE_qrshift_single<[llvm_i32_ty, llvm_i32_ty]>;

def int_arm_mve_vabd: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],
   [IntrNoMem]>;
def int_arm_mve_vadc: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty, llvm_i32_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty], [IntrNoMem]>;
````
- **L1105 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<value, value # [llvm_i32_ty] # saturate, [IntrNoMem]>;`.
  **L1105 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<value, value # [llvm_i32_ty] # saturate, [IntrNoMem]>;`。
- **L1106 EN**: Declares TableGen multiclass `ARM_MVE_qrshift`.
  **L1106 CN**: 声明 TableGen multiclass `ARM_MVE_qrshift`。
- **L1107 EN**: Comment explains nearby logic, invariants, or intent: `Most of these shifts come in 32- and 64-bit versions. But only`.
  **L1107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Most of these shifts come in 32- and 64-bit versions. But only`。
- **L1108 EN**: Comment explains nearby logic, invariants, or intent: `the 64-bit ones have the extra saturation argument (if any).`.
  **L1108 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the 64-bit ones have the extra saturation argument (if any).`。
- **L1109 EN**: Declares TableGen def `""`.
  **L1109 CN**: 声明 TableGen def `""`。
- **L1110 EN**: Declares TableGen def `l`.
  **L1110 CN**: 声明 TableGen def `l`。
- **L1111 EN**: Closes the current lexical scope or compound statement.
  **L1111 CN**: 结束当前词法作用域或复合语句块。
- **L1112 EN**: Declares TableGen defm `int_arm_mve_urshr`.
  **L1112 CN**: 声明 TableGen defm `int_arm_mve_urshr`。
- **L1113 EN**: Declares TableGen defm `int_arm_mve_uqshl`.
  **L1113 CN**: 声明 TableGen defm `int_arm_mve_uqshl`。
- **L1114 EN**: Declares TableGen defm `int_arm_mve_srshr`.
  **L1114 CN**: 声明 TableGen defm `int_arm_mve_srshr`。
- **L1115 EN**: Declares TableGen defm `int_arm_mve_sqshl`.
  **L1115 CN**: 声明 TableGen defm `int_arm_mve_sqshl`。
- **L1116 EN**: Declares TableGen defm `int_arm_mve_uqrshl`.
  **L1116 CN**: 声明 TableGen defm `int_arm_mve_uqrshl`。
- **L1117 EN**: Declares TableGen defm `int_arm_mve_sqrshr`.
  **L1117 CN**: 声明 TableGen defm `int_arm_mve_sqrshr`。
- **L1118 EN**: Comment explains nearby logic, invariants, or intent: `LSLL and ASRL only have 64-bit versions, not 32.`.
  **L1118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LSLL and ASRL only have 64-bit versions, not 32.`。
- **L1119 EN**: Declares TableGen def `int_arm_mve_lsll`.
  **L1119 CN**: 声明 TableGen def `int_arm_mve_lsll`。
- **L1120 EN**: Declares TableGen def `int_arm_mve_asrl`.
  **L1120 CN**: 声明 TableGen def `int_arm_mve_asrl`。
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1122 EN**: Declares TableGen def `int_arm_mve_vabd`.
  **L1122 CN**: 声明 TableGen def `int_arm_mve_vabd`。
- **L1123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1123 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`.
  **L1124 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`。
- **L1125 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1125 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1126 EN**: Declares TableGen def `int_arm_mve_vadc`.
  **L1126 CN**: 声明 TableGen def `int_arm_mve_vadc`。
- **L1127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty],`.
  **L1127 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty],`。
- **L1128 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty], [IntrNoMem]>;`.
  **L1128 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty], [IntrNoMem]>;`。

### Lines 1129-1152

````tablegen
def int_arm_mve_vsbc: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty, llvm_i32_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty], [IntrNoMem]>;
def int_arm_mve_vadc_predicated: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty, llvm_i32_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
    llvm_i32_ty, llvm_anyvector_ty], [IntrNoMem]>;
def int_arm_mve_vsbc_predicated: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty, llvm_i32_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
    llvm_i32_ty, llvm_anyvector_ty], [IntrNoMem]>;
def int_arm_mve_vshlc: DefaultAttrsIntrinsic<
   [llvm_i32_ty /* bits shifted out */, llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty /* bits shifted in */,
    llvm_i32_ty /* shift count */], [IntrNoMem]>;
def int_arm_mve_vshlc_predicated: DefaultAttrsIntrinsic<
   [llvm_i32_ty /* bits shifted out */, llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty /* bits shifted in */,
    llvm_i32_ty /* shift count */, llvm_anyvector_ty], [IntrNoMem]>;
def int_arm_mve_vadd: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_vsub: DefaultAttrsIntrinsic<
````
- **L1129 EN**: Declares TableGen def `int_arm_mve_vsbc`.
  **L1129 CN**: 声明 TableGen def `int_arm_mve_vsbc`。
- **L1130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty],`.
  **L1130 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty],`。
- **L1131 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty], [IntrNoMem]>;`.
  **L1131 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty], [IntrNoMem]>;`。
- **L1132 EN**: Declares TableGen def `int_arm_mve_vadc_predicated`.
  **L1132 CN**: 声明 TableGen def `int_arm_mve_vadc_predicated`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty],`.
  **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty],`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L1135 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_anyvector_ty], [IntrNoMem]>;`.
  **L1135 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_anyvector_ty], [IntrNoMem]>;`。
- **L1136 EN**: Declares TableGen def `int_arm_mve_vsbc_predicated`.
  **L1136 CN**: 声明 TableGen def `int_arm_mve_vsbc_predicated`。
- **L1137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty],`.
  **L1137 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty],`。
- **L1138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L1138 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L1139 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_anyvector_ty], [IntrNoMem]>;`.
  **L1139 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_anyvector_ty], [IntrNoMem]>;`。
- **L1140 EN**: Declares TableGen def `int_arm_mve_vshlc`.
  **L1140 CN**: 声明 TableGen def `int_arm_mve_vshlc`。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* bits shifted out */, llvm_anyvector_ty],`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* bits shifted out */, llvm_anyvector_ty],`。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty /* bits shifted in */,`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty /* bits shifted in */,`。
- **L1143 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /* shift count */], [IntrNoMem]>;`.
  **L1143 CN**: 执行一条独立语句或声明：`llvm_i32_ty /* shift count */], [IntrNoMem]>;`。
- **L1144 EN**: Declares TableGen def `int_arm_mve_vshlc_predicated`.
  **L1144 CN**: 声明 TableGen def `int_arm_mve_vshlc_predicated`。
- **L1145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* bits shifted out */, llvm_anyvector_ty],`.
  **L1145 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* bits shifted out */, llvm_anyvector_ty],`。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty /* bits shifted in */,`.
  **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty /* bits shifted in */,`。
- **L1147 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /* shift count */, llvm_anyvector_ty], [IntrNoMem]>;`.
  **L1147 CN**: 执行一条独立语句或声明：`llvm_i32_ty /* shift count */, llvm_anyvector_ty], [IntrNoMem]>;`。
- **L1148 EN**: Declares TableGen def `int_arm_mve_vadd`.
  **L1148 CN**: 声明 TableGen def `int_arm_mve_vadd`。
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1150 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1151 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1151 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1152 EN**: Declares TableGen def `int_arm_mve_vsub`.
  **L1152 CN**: 声明 TableGen def `int_arm_mve_vsub`。

### Lines 1153-1176

````tablegen
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_vmul: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>],
   [IntrNoMem]>;
def int_arm_mve_vmulh: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],
   [IntrNoMem]>;
def int_arm_mve_vqdmulh: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_vhadd: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],
   [IntrNoMem]>;
def int_arm_mve_vrhadd: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],
   [IntrNoMem]>;
def int_arm_mve_vhsub: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
````
- **L1153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1153 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1155 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1155 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1156 EN**: Declares TableGen def `int_arm_mve_vmul`.
  **L1156 CN**: 声明 TableGen def `int_arm_mve_vmul`。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1158 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1159 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1159 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1160 EN**: Declares TableGen def `int_arm_mve_vmulh`.
  **L1160 CN**: 声明 TableGen def `int_arm_mve_vmulh`。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`.
  **L1162 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`。
- **L1163 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1163 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1164 EN**: Declares TableGen def `int_arm_mve_vqdmulh`.
  **L1164 CN**: 声明 TableGen def `int_arm_mve_vqdmulh`。
- **L1165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1165 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1166 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L1166 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L1167 EN**: Declares TableGen def `int_arm_mve_vhadd`.
  **L1167 CN**: 声明 TableGen def `int_arm_mve_vhadd`。
- **L1168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1168 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`.
  **L1169 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`。
- **L1170 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1170 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1171 EN**: Declares TableGen def `int_arm_mve_vrhadd`.
  **L1171 CN**: 声明 TableGen def `int_arm_mve_vrhadd`。
- **L1172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1172 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`.
  **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`。
- **L1174 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1174 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1175 EN**: Declares TableGen def `int_arm_mve_vhsub`.
  **L1175 CN**: 声明 TableGen def `int_arm_mve_vhsub`。
- **L1176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1176 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。

### Lines 1177-1200

````tablegen
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],
   [IntrNoMem]>;
def int_arm_mve_vrmulh: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],
   [IntrNoMem]>;
def int_arm_mve_vqrdmulh: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_vmull: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty /* unsigned */,
    llvm_i32_ty /* top */], [IntrNoMem]>;
def int_arm_mve_vmull_poly: DefaultAttrsIntrinsic<
   [llvm_anyvector_ty],
   [llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty], [IntrNoMem]>;

// The first two parameters are compile-time constants:
// * Halving: 0 means  halving (vhcaddq), 1 means non-halving (vcaddq) 
//            instruction. Note: the flag is inverted to match the corresponding
//            bit in the instruction encoding
// * Rotation angle: 0 mean 90 deg, 1 means 180 deg
defm int_arm_mve_vcaddq : MVEMXPredicated<
  [llvm_anyvector_ty],
````
- **L1177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`.
  **L1177 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`。
- **L1178 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1178 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1179 EN**: Declares TableGen def `int_arm_mve_vrmulh`.
  **L1179 CN**: 声明 TableGen def `int_arm_mve_vrmulh`。
- **L1180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1180 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`.
  **L1181 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, llvm_i32_ty /* unsigned */],`。
- **L1182 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1182 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1183 EN**: Declares TableGen def `int_arm_mve_vqrdmulh`.
  **L1183 CN**: 声明 TableGen def `int_arm_mve_vqrdmulh`。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1185 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L1185 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L1186 EN**: Declares TableGen def `int_arm_mve_vmull`.
  **L1186 CN**: 声明 TableGen def `int_arm_mve_vmull`。
- **L1187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1187 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty /* unsigned */,`.
  **L1188 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty /* unsigned */,`。
- **L1189 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /* top */], [IntrNoMem]>;`.
  **L1189 CN**: 执行一条独立语句或声明：`llvm_i32_ty /* top */], [IntrNoMem]>;`。
- **L1190 EN**: Declares TableGen def `int_arm_mve_vmull_poly`.
  **L1190 CN**: 声明 TableGen def `int_arm_mve_vmull_poly`。
- **L1191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1191 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1192 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty], [IntrNoMem]>;`.
  **L1192 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty], [IntrNoMem]>;`。
- **L1193 EN**: Blank line separating nearby declarations or logic blocks.
  **L1193 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1194 EN**: Comment explains nearby logic, invariants, or intent: `The first two parameters are compile-time constants:`.
  **L1194 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first two parameters are compile-time constants:`。
- **L1195 EN**: Comment explains nearby logic, invariants, or intent: `* Halving: 0 means  halving (vhcaddq), 1 means non-halving (vcaddq)`.
  **L1195 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* Halving: 0 means  halving (vhcaddq), 1 means non-halving (vcaddq)`。
- **L1196 EN**: Comment explains nearby logic, invariants, or intent: `instruction. Note: the flag is inverted to match the corresponding`.
  **L1196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`instruction. Note: the flag is inverted to match the corresponding`。
- **L1197 EN**: Comment explains nearby logic, invariants, or intent: `bit in the instruction encoding`.
  **L1197 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`bit in the instruction encoding`。
- **L1198 EN**: Comment explains nearby logic, invariants, or intent: `* Rotation angle: 0 mean 90 deg, 1 means 180 deg`.
  **L1198 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* Rotation angle: 0 mean 90 deg, 1 means 180 deg`。
- **L1199 EN**: Declares TableGen defm `int_arm_mve_vcaddq`.
  **L1199 CN**: 声明 TableGen defm `int_arm_mve_vcaddq`。
- **L1200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1200 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。

### Lines 1201-1224

````tablegen
  [llvm_i32_ty, llvm_i32_ty], [LLVMMatchType<0>, LLVMMatchType<0>],
   LLVMMatchType<0>, llvm_anyvector_ty>;

// The first operand of the following two intrinsics is the rotation angle
// (must be a compile-time constant):
// 0 - 0 deg
// 1 - 90 deg
// 2 - 180 deg
// 3 - 270 deg
defm int_arm_mve_vcmulq : MVEMXPredicated<
  [llvm_anyvector_ty],
  [llvm_i32_ty], [LLVMMatchType<0>, LLVMMatchType<0>],
   LLVMMatchType<0>, llvm_anyvector_ty>;

defm int_arm_mve_vcmlaq : MVEPredicated<
  [llvm_anyvector_ty],
  [llvm_i32_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
   llvm_anyvector_ty>;

def int_arm_mve_vld2q: DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty],
    [IntrReadMem, IntrArgMemOnly]>;
def int_arm_mve_vld4q: DefaultAttrsIntrinsic<
    [llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],
````
- **L1201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1201 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1202 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1202 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1204 EN**: Comment explains nearby logic, invariants, or intent: `The first operand of the following two intrinsics is the rotation angle`.
  **L1204 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first operand of the following two intrinsics is the rotation angle`。
- **L1205 EN**: Comment explains nearby logic, invariants, or intent: `(must be a compile-time constant):`.
  **L1205 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(must be a compile-time constant):`。
- **L1206 EN**: Comment explains nearby logic, invariants, or intent: `0 - 0 deg`.
  **L1206 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`0 - 0 deg`。
- **L1207 EN**: Comment explains nearby logic, invariants, or intent: `1 - 90 deg`.
  **L1207 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`1 - 90 deg`。
- **L1208 EN**: Comment explains nearby logic, invariants, or intent: `2 - 180 deg`.
  **L1208 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2 - 180 deg`。
- **L1209 EN**: Comment explains nearby logic, invariants, or intent: `3 - 270 deg`.
  **L1209 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`3 - 270 deg`。
- **L1210 EN**: Declares TableGen defm `int_arm_mve_vcmulq`.
  **L1210 CN**: 声明 TableGen defm `int_arm_mve_vcmulq`。
- **L1211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1211 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty], [LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1213 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1213 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1215 EN**: Declares TableGen defm `int_arm_mve_vcmlaq`.
  **L1215 CN**: 声明 TableGen defm `int_arm_mve_vcmlaq`。
- **L1216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1216 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1218 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty>;`.
  **L1218 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty>;`。
- **L1219 EN**: Blank line separating nearby declarations or logic blocks.
  **L1219 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1220 EN**: Declares TableGen def `int_arm_mve_vld2q`.
  **L1220 CN**: 声明 TableGen def `int_arm_mve_vld2q`。
- **L1221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty],`.
  **L1221 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>], [llvm_anyptr_ty],`。
- **L1222 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1222 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1223 EN**: Declares TableGen def `int_arm_mve_vld4q`.
  **L1223 CN**: 声明 TableGen def `int_arm_mve_vld4q`。
- **L1224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1224 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>],`。

### Lines 1225-1248

````tablegen
    [llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;

def int_arm_mve_vst2q: DefaultAttrsIntrinsic<
    [], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly], "", [SDNPMemOperand]>;
def int_arm_mve_vst4q: DefaultAttrsIntrinsic<
    [],
    [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,
     LLVMMatchType<1>, llvm_i32_ty],
    [IntrWriteMem, IntrArgMemOnly], "", [SDNPMemOperand]>;

// MVE vector absolute difference and accumulate across vector
// The first operand is an 'unsigned' flag. The remaining operands are:
// * accumulator
// * first vector operand
// * second vector operand
// * mask (only in predicated versions)
defm int_arm_mve_vabav: MVEPredicated<
  [llvm_i32_ty],
  [llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>], llvm_anyvector_ty>;

// The following 3 intrinsics are MVE vector reductions with two vector
// operands.
// The first 3 operands are boolean flags (must be compile-time constants):
````
- **L1225 EN**: Executes a standalone statement or declaration: `[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`.
  **L1225 CN**: 执行一条独立语句或声明：`[llvm_anyptr_ty], [IntrReadMem, IntrArgMemOnly]>;`。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1227 EN**: Declares TableGen def `int_arm_mve_vst2q`.
  **L1227 CN**: 声明 TableGen def `int_arm_mve_vst2q`。
- **L1228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],`.
  **L1228 CN**: 继续一个多行参数列表、初始化器或聚合项：`[], [llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],`。
- **L1229 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly], "", [SDNPMemOperand]>;`.
  **L1229 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly], "", [SDNPMemOperand]>;`。
- **L1230 EN**: Declares TableGen def `int_arm_mve_vst4q`.
  **L1230 CN**: 声明 TableGen def `int_arm_mve_vst4q`。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[],`.
  **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`[],`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`.
  **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, llvm_anyvector_ty, LLVMMatchType<1>, LLVMMatchType<1>,`。
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<1>, llvm_i32_ty],`.
  **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<1>, llvm_i32_ty],`。
- **L1234 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly], "", [SDNPMemOperand]>;`.
  **L1234 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly], "", [SDNPMemOperand]>;`。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1236 EN**: Comment explains nearby logic, invariants, or intent: `MVE vector absolute difference and accumulate across vector`.
  **L1236 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MVE vector absolute difference and accumulate across vector`。
- **L1237 EN**: Comment explains nearby logic, invariants, or intent: `The first operand is an 'unsigned' flag. The remaining operands are:`.
  **L1237 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first operand is an 'unsigned' flag. The remaining operands are:`。
- **L1238 EN**: Comment explains nearby logic, invariants, or intent: `* accumulator`.
  **L1238 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* accumulator`。
- **L1239 EN**: Comment explains nearby logic, invariants, or intent: `* first vector operand`.
  **L1239 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* first vector operand`。
- **L1240 EN**: Comment explains nearby logic, invariants, or intent: `* second vector operand`.
  **L1240 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* second vector operand`。
- **L1241 EN**: Comment explains nearby logic, invariants, or intent: `* mask (only in predicated versions)`.
  **L1241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* mask (only in predicated versions)`。
- **L1242 EN**: Declares TableGen defm `int_arm_mve_vabav`.
  **L1242 CN**: 声明 TableGen defm `int_arm_mve_vabav`。
- **L1243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L1243 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L1244 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>], llvm_anyvector_ty>;`.
  **L1244 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>], llvm_anyvector_ty>;`。
- **L1245 EN**: Blank line separating nearby declarations or logic blocks.
  **L1245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1246 EN**: Comment explains nearby logic, invariants, or intent: `The following 3 intrinsics are MVE vector reductions with two vector`.
  **L1246 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following 3 intrinsics are MVE vector reductions with two vector`。
- **L1247 EN**: Comment explains nearby logic, invariants, or intent: `operands.`.
  **L1247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operands.`。
- **L1248 EN**: Comment explains nearby logic, invariants, or intent: `The first 3 operands are boolean flags (must be compile-time constants):`.
  **L1248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The first 3 operands are boolean flags (must be compile-time constants):`。

### Lines 1249-1272

````tablegen
// * unsigned - the instruction operates on vectors of unsigned values and
//              unsigned scalars
// * subtract - the instruction performs subtraction after multiplication of
//              lane pairs (e.g., vmlsdav vs vmladav)
// * exchange - the instruction exchanges successive even and odd lanes of
//              the first operands before multiplication of lane pairs
//              (e.g., vmladavx vs vmladav)
// The remaining operands are:
// * accumulator
// * first vector operand
// * second vector operand
// * mask (only in predicated versions)

// Version with 32-bit result, vml{a,s}dav[a][x]
defm int_arm_mve_vmldava: MVEPredicated<
  [llvm_i32_ty],
  [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
   llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],
  llvm_anyvector_ty>;

// Version with 64-bit result, vml{a,s}ldav[a][x]
defm int_arm_mve_vmlldava: MVEPredicated<
  [llvm_i32_ty, llvm_i32_ty],
  [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
````
- **L1249 EN**: Comment explains nearby logic, invariants, or intent: `* unsigned - the instruction operates on vectors of unsigned values and`.
  **L1249 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* unsigned - the instruction operates on vectors of unsigned values and`。
- **L1250 EN**: Comment explains nearby logic, invariants, or intent: `unsigned scalars`.
  **L1250 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`unsigned scalars`。
- **L1251 EN**: Comment explains nearby logic, invariants, or intent: `* subtract - the instruction performs subtraction after multiplication of`.
  **L1251 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* subtract - the instruction performs subtraction after multiplication of`。
- **L1252 EN**: Comment explains nearby logic, invariants, or intent: `lane pairs (e.g., vmlsdav vs vmladav)`.
  **L1252 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lane pairs (e.g., vmlsdav vs vmladav)`。
- **L1253 EN**: Comment explains nearby logic, invariants, or intent: `* exchange - the instruction exchanges successive even and odd lanes of`.
  **L1253 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* exchange - the instruction exchanges successive even and odd lanes of`。
- **L1254 EN**: Comment explains nearby logic, invariants, or intent: `the first operands before multiplication of lane pairs`.
  **L1254 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the first operands before multiplication of lane pairs`。
- **L1255 EN**: Comment explains nearby logic, invariants, or intent: `(e.g., vmladavx vs vmladav)`.
  **L1255 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(e.g., vmladavx vs vmladav)`。
- **L1256 EN**: Comment explains nearby logic, invariants, or intent: `The remaining operands are:`.
  **L1256 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The remaining operands are:`。
- **L1257 EN**: Comment explains nearby logic, invariants, or intent: `* accumulator`.
  **L1257 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* accumulator`。
- **L1258 EN**: Comment explains nearby logic, invariants, or intent: `* first vector operand`.
  **L1258 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* first vector operand`。
- **L1259 EN**: Comment explains nearby logic, invariants, or intent: `* second vector operand`.
  **L1259 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* second vector operand`。
- **L1260 EN**: Comment explains nearby logic, invariants, or intent: `* mask (only in predicated versions)`.
  **L1260 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* mask (only in predicated versions)`。
- **L1261 EN**: Blank line separating nearby declarations or logic blocks.
  **L1261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1262 EN**: Comment explains nearby logic, invariants, or intent: `Version with 32-bit result, vml{a,s}dav[a][x]`.
  **L1262 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Version with 32-bit result, vml{a,s}dav[a][x]`。
- **L1263 EN**: Declares TableGen defm `int_arm_mve_vmldava`.
  **L1263 CN**: 声明 TableGen defm `int_arm_mve_vmldava`。
- **L1264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L1264 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L1265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L1265 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L1266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1266 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1267 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty>;`.
  **L1267 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty>;`。
- **L1268 EN**: Blank line separating nearby declarations or logic blocks.
  **L1268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1269 EN**: Comment explains nearby logic, invariants, or intent: `Version with 64-bit result, vml{a,s}ldav[a][x]`.
  **L1269 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Version with 64-bit result, vml{a,s}ldav[a][x]`。
- **L1270 EN**: Declares TableGen defm `int_arm_mve_vmlldava`.
  **L1270 CN**: 声明 TableGen defm `int_arm_mve_vmlldava`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L1272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L1272 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。

### Lines 1273-1296

````tablegen
   llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],
  llvm_anyvector_ty>;

// Version with 72-bit rounded result, vrml{a,s}ldavh[a][x]
defm int_arm_mve_vrmlldavha: MVEPredicated<
  [llvm_i32_ty, llvm_i32_ty],
  [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,
   llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],
  llvm_anyvector_ty>;

defm int_arm_mve_vidup: MVEMXPredicated<
   [llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],
   [llvm_i32_ty /* base */, llvm_i32_ty /* step */],
   LLVMMatchType<0>, llvm_anyvector_ty>;
defm int_arm_mve_vddup: MVEMXPredicated<
   [llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],
   [llvm_i32_ty /* base */, llvm_i32_ty /* step */],
   LLVMMatchType<0>, llvm_anyvector_ty>;
defm int_arm_mve_viwdup: MVEMXPredicated<
   [llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],
   [llvm_i32_ty /* base */, llvm_i32_ty /* limit */, llvm_i32_ty /* step */],
   LLVMMatchType<0>, llvm_anyvector_ty>;
defm int_arm_mve_vdwdup: MVEMXPredicated<
   [llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],
````
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1274 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty>;`.
  **L1274 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty>;`。
- **L1275 EN**: Blank line separating nearby declarations or logic blocks.
  **L1275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1276 EN**: Comment explains nearby logic, invariants, or intent: `Version with 72-bit rounded result, vrml{a,s}ldavh[a][x]`.
  **L1276 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Version with 72-bit rounded result, vrml{a,s}ldavh[a][x]`。
- **L1277 EN**: Declares TableGen defm `int_arm_mve_vrmlldavha`.
  **L1277 CN**: 声明 TableGen defm `int_arm_mve_vrmlldavha`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty],`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty],`。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`.
  **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_anyvector_ty, LLVMMatchType<0>],`。
- **L1281 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty>;`.
  **L1281 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty>;`。
- **L1282 EN**: Blank line separating nearby declarations or logic blocks.
  **L1282 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1283 EN**: Declares TableGen defm `int_arm_mve_vidup`.
  **L1283 CN**: 声明 TableGen defm `int_arm_mve_vidup`。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`.
  **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`。
- **L1285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* base */, llvm_i32_ty /* step */],`.
  **L1285 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* base */, llvm_i32_ty /* step */],`。
- **L1286 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1286 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1287 EN**: Declares TableGen defm `int_arm_mve_vddup`.
  **L1287 CN**: 声明 TableGen defm `int_arm_mve_vddup`。
- **L1288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`.
  **L1288 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`。
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* base */, llvm_i32_ty /* step */],`.
  **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* base */, llvm_i32_ty /* step */],`。
- **L1290 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1290 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1291 EN**: Declares TableGen defm `int_arm_mve_viwdup`.
  **L1291 CN**: 声明 TableGen defm `int_arm_mve_viwdup`。
- **L1292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`.
  **L1292 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`。
- **L1293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* base */, llvm_i32_ty /* limit */, llvm_i32_ty /* step */],`.
  **L1293 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* base */, llvm_i32_ty /* limit */, llvm_i32_ty /* step */],`。
- **L1294 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1294 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1295 EN**: Declares TableGen defm `int_arm_mve_vdwdup`.
  **L1295 CN**: 声明 TableGen defm `int_arm_mve_vdwdup`。
- **L1296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`.
  **L1296 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty /* output */, llvm_i32_ty /* written-back base */], [],`。

### Lines 1297-1320

````tablegen
   [llvm_i32_ty /* base */, llvm_i32_ty /* limit */, llvm_i32_ty /* step */],
   LLVMMatchType<0>, llvm_anyvector_ty>;

// Flags:
// * unsigned
defm int_arm_mve_vcvt_fix: MVEMXPredicated<
  [llvm_anyvector_ty /* output */], [llvm_i32_ty],
  [llvm_anyvector_ty /* input vector */, llvm_i32_ty /* scale */],
  LLVMMatchType<0>, llvm_anyvector_ty>;

def int_arm_mve_vcvt_fp_int: DefaultAttrsIntrinsic<
  [llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */],
  [IntrNoMem]>;
def int_arm_mve_vcvt_int_fp: DefaultAttrsIntrinsic<
  [llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */],
  [IntrNoMem]>;
def int_arm_mve_vcvt_fp_int_predicated: DefaultAttrsIntrinsic<
  [llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */,
   llvm_anyvector_ty /* predicate */, LLVMMatchType<0> /* inactive */],
  [IntrNoMem]>;

foreach suffix = ["a","n","p","m"] in {
  defm "int_arm_mve_vcvt"#suffix: MVEMXPredicated<
    [llvm_anyvector_ty /* output */], [llvm_i32_ty /* unsigned */],
````
- **L1297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* base */, llvm_i32_ty /* limit */, llvm_i32_ty /* step */],`.
  **L1297 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* base */, llvm_i32_ty /* limit */, llvm_i32_ty /* step */],`。
- **L1298 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1298 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1299 EN**: Blank line separating nearby declarations or logic blocks.
  **L1299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1300 EN**: Comment explains nearby logic, invariants, or intent: `Flags:`.
  **L1300 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Flags:`。
- **L1301 EN**: Comment explains nearby logic, invariants, or intent: `* unsigned`.
  **L1301 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* unsigned`。
- **L1302 EN**: Declares TableGen defm `int_arm_mve_vcvt_fix`.
  **L1302 CN**: 声明 TableGen defm `int_arm_mve_vcvt_fix`。
- **L1303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty /* output */], [llvm_i32_ty],`.
  **L1303 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty /* output */], [llvm_i32_ty],`。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty /* input vector */, llvm_i32_ty /* scale */],`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty /* input vector */, llvm_i32_ty /* scale */],`。
- **L1305 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1305 CN**: 执行一条独立语句或声明：`LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1306 EN**: Blank line separating nearby declarations or logic blocks.
  **L1306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1307 EN**: Declares TableGen def `int_arm_mve_vcvt_fp_int`.
  **L1307 CN**: 声明 TableGen def `int_arm_mve_vcvt_fp_int`。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */],`.
  **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */],`。
- **L1309 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1309 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1310 EN**: Declares TableGen def `int_arm_mve_vcvt_int_fp`.
  **L1310 CN**: 声明 TableGen def `int_arm_mve_vcvt_int_fp`。
- **L1311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */],`.
  **L1311 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */],`。
- **L1312 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1312 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1313 EN**: Declares TableGen def `int_arm_mve_vcvt_fp_int_predicated`.
  **L1313 CN**: 声明 TableGen def `int_arm_mve_vcvt_fp_int_predicated`。
- **L1314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */,`.
  **L1314 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [llvm_anyvector_ty, llvm_i32_ty /* unsigned */,`。
- **L1315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_anyvector_ty /* predicate */, LLVMMatchType<0> /* inactive */],`.
  **L1315 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_anyvector_ty /* predicate */, LLVMMatchType<0> /* inactive */],`。
- **L1316 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1316 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1317 EN**: Blank line separating nearby declarations or logic blocks.
  **L1317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1318 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1318 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1319 EN**: Declares TableGen defm `"int_arm_mve_vcvt"#suffix`.
  **L1319 CN**: 声明 TableGen defm `"int_arm_mve_vcvt"#suffix`。
- **L1320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty /* output */], [llvm_i32_ty /* unsigned */],`.
  **L1320 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty /* output */], [llvm_i32_ty /* unsigned */],`。

### Lines 1321-1344

````tablegen
    [llvm_anyvector_ty /* input */], LLVMMatchType<0>, llvm_anyvector_ty>;
}

def int_arm_mve_vcls: DefaultAttrsIntrinsic<
  [llvm_anyvector_ty], [LLVMMatchType<0>], [IntrNoMem]>;

defm int_arm_mve_vbrsr: MVEMXPredicated<
  [llvm_anyvector_ty], [],
  [LLVMMatchType<0>, llvm_i32_ty], LLVMMatchType<0>, llvm_anyvector_ty>;

def int_arm_mve_vqdmull: DefaultAttrsIntrinsic<
  [llvm_anyvector_ty],
  [llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],
  [IntrNoMem]>;
def int_arm_mve_vqdmull_predicated: DefaultAttrsIntrinsic<
  [llvm_anyvector_ty],
  [llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty, llvm_anyvector_ty,
   LLVMMatchType<0>],
  [IntrNoMem]>;

class MVESimpleUnary: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>], [IntrNoMem]>;
class MVESimpleUnaryPredicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
````
- **L1321 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty /* input */], LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1321 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty /* input */], LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1322 EN**: Closes the current lexical scope or compound statement.
  **L1322 CN**: 结束当前词法作用域或复合语句块。
- **L1323 EN**: Blank line separating nearby declarations or logic blocks.
  **L1323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1324 EN**: Declares TableGen def `int_arm_mve_vcls`.
  **L1324 CN**: 声明 TableGen def `int_arm_mve_vcls`。
- **L1325 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty], [LLVMMatchType<0>], [IntrNoMem]>;`.
  **L1325 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty], [LLVMMatchType<0>], [IntrNoMem]>;`。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1327 EN**: Declares TableGen defm `int_arm_mve_vbrsr`.
  **L1327 CN**: 声明 TableGen defm `int_arm_mve_vbrsr`。
- **L1328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty], [],`.
  **L1328 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty], [],`。
- **L1329 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_i32_ty], LLVMMatchType<0>, llvm_anyvector_ty>;`.
  **L1329 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_i32_ty], LLVMMatchType<0>, llvm_anyvector_ty>;`。
- **L1330 EN**: Blank line separating nearby declarations or logic blocks.
  **L1330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1331 EN**: Declares TableGen def `int_arm_mve_vqdmull`.
  **L1331 CN**: 声明 TableGen def `int_arm_mve_vqdmull`。
- **L1332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1332 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],`.
  **L1333 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty],`。
- **L1334 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1334 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1335 EN**: Declares TableGen def `int_arm_mve_vqdmull_predicated`.
  **L1335 CN**: 声明 TableGen def `int_arm_mve_vqdmull_predicated`。
- **L1336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1336 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty, llvm_anyvector_ty,`.
  **L1337 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, LLVMMatchType<1>, llvm_i32_ty, llvm_anyvector_ty,`。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L1339 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1339 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1340 EN**: Blank line separating nearby declarations or logic blocks.
  **L1340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1341 EN**: Declares class `MVESimpleUnary`.
  **L1341 CN**: 声明 class `MVESimpleUnary`。
- **L1342 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>], [IntrNoMem]>;`.
  **L1342 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>], [IntrNoMem]>;`。
- **L1343 EN**: Declares class `MVESimpleUnaryPredicated`.
  **L1343 CN**: 声明 class `MVESimpleUnaryPredicated`。
- **L1344 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L1344 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。

### Lines 1345-1368

````tablegen

def int_arm_mve_mvn_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_abs_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_neg_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_qabs_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_qneg_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_clz_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_cls_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_vrintz: MVESimpleUnary;
def int_arm_mve_vrintm: MVESimpleUnary;
def int_arm_mve_vrintp: MVESimpleUnary;
def int_arm_mve_vrinta: MVESimpleUnary;
def int_arm_mve_vrintx: MVESimpleUnary;
def int_arm_mve_vrintn: MVESimpleUnary;
def int_arm_mve_vrintz_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_vrintm_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_vrintp_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_vrinta_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_vrintx_predicated: MVESimpleUnaryPredicated;
def int_arm_mve_vrintn_predicated: MVESimpleUnaryPredicated;

def int_arm_mve_vrev_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_i32_ty /* size to reverse */,
    llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;
````
- **L1345 EN**: Blank line separating nearby declarations or logic blocks.
  **L1345 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1346 EN**: Declares TableGen def `int_arm_mve_mvn_predicated`.
  **L1346 CN**: 声明 TableGen def `int_arm_mve_mvn_predicated`。
- **L1347 EN**: Declares TableGen def `int_arm_mve_abs_predicated`.
  **L1347 CN**: 声明 TableGen def `int_arm_mve_abs_predicated`。
- **L1348 EN**: Declares TableGen def `int_arm_mve_neg_predicated`.
  **L1348 CN**: 声明 TableGen def `int_arm_mve_neg_predicated`。
- **L1349 EN**: Declares TableGen def `int_arm_mve_qabs_predicated`.
  **L1349 CN**: 声明 TableGen def `int_arm_mve_qabs_predicated`。
- **L1350 EN**: Declares TableGen def `int_arm_mve_qneg_predicated`.
  **L1350 CN**: 声明 TableGen def `int_arm_mve_qneg_predicated`。
- **L1351 EN**: Declares TableGen def `int_arm_mve_clz_predicated`.
  **L1351 CN**: 声明 TableGen def `int_arm_mve_clz_predicated`。
- **L1352 EN**: Declares TableGen def `int_arm_mve_cls_predicated`.
  **L1352 CN**: 声明 TableGen def `int_arm_mve_cls_predicated`。
- **L1353 EN**: Declares TableGen def `int_arm_mve_vrintz`.
  **L1353 CN**: 声明 TableGen def `int_arm_mve_vrintz`。
- **L1354 EN**: Declares TableGen def `int_arm_mve_vrintm`.
  **L1354 CN**: 声明 TableGen def `int_arm_mve_vrintm`。
- **L1355 EN**: Declares TableGen def `int_arm_mve_vrintp`.
  **L1355 CN**: 声明 TableGen def `int_arm_mve_vrintp`。
- **L1356 EN**: Declares TableGen def `int_arm_mve_vrinta`.
  **L1356 CN**: 声明 TableGen def `int_arm_mve_vrinta`。
- **L1357 EN**: Declares TableGen def `int_arm_mve_vrintx`.
  **L1357 CN**: 声明 TableGen def `int_arm_mve_vrintx`。
- **L1358 EN**: Declares TableGen def `int_arm_mve_vrintn`.
  **L1358 CN**: 声明 TableGen def `int_arm_mve_vrintn`。
- **L1359 EN**: Declares TableGen def `int_arm_mve_vrintz_predicated`.
  **L1359 CN**: 声明 TableGen def `int_arm_mve_vrintz_predicated`。
- **L1360 EN**: Declares TableGen def `int_arm_mve_vrintm_predicated`.
  **L1360 CN**: 声明 TableGen def `int_arm_mve_vrintm_predicated`。
- **L1361 EN**: Declares TableGen def `int_arm_mve_vrintp_predicated`.
  **L1361 CN**: 声明 TableGen def `int_arm_mve_vrintp_predicated`。
- **L1362 EN**: Declares TableGen def `int_arm_mve_vrinta_predicated`.
  **L1362 CN**: 声明 TableGen def `int_arm_mve_vrinta_predicated`。
- **L1363 EN**: Declares TableGen def `int_arm_mve_vrintx_predicated`.
  **L1363 CN**: 声明 TableGen def `int_arm_mve_vrintx_predicated`。
- **L1364 EN**: Declares TableGen def `int_arm_mve_vrintn_predicated`.
  **L1364 CN**: 声明 TableGen def `int_arm_mve_vrintn_predicated`。
- **L1365 EN**: Blank line separating nearby declarations or logic blocks.
  **L1365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1366 EN**: Declares TableGen def `int_arm_mve_vrev_predicated`.
  **L1366 CN**: 声明 TableGen def `int_arm_mve_vrev_predicated`。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_i32_ty /* size to reverse */,`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_i32_ty /* size to reverse */,`。
- **L1368 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L1368 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty, LLVMMatchType<0>], [IntrNoMem]>;`。

### Lines 1369-1392

````tablegen

def int_arm_mve_vmovl_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [llvm_anyvector_ty, llvm_i32_ty /* unsigned */, llvm_i32_ty /* top half */,
    llvm_anyvector_ty /* predicate */, LLVMMatchType<0>], [IntrNoMem]>;
def int_arm_mve_vmovn_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_anyvector_ty, llvm_i32_ty /* top half */,
    llvm_anyvector_ty /* predicate */], [IntrNoMem]>;

def int_arm_mve_vqmovn: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_anyvector_ty,
    llvm_i32_ty /* unsigned output */, llvm_i32_ty /* unsigned input */,
    llvm_i32_ty /* top half */], [IntrNoMem]>;
def int_arm_mve_vqmovn_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0>, llvm_anyvector_ty,
    llvm_i32_ty /* unsigned output */, llvm_i32_ty /* unsigned input */,
    llvm_i32_ty /* top half */, llvm_anyvector_ty /* pred */], [IntrNoMem]>;

def int_arm_mve_fma: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,
    LLVMMatchType<0> /* addend */], [IntrNoMem]>;
// fma_predicated returns the add operand for disabled lanes.
def int_arm_mve_fma_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,
    LLVMMatchType<0> /* addend */, llvm_anyvector_ty /* pred */], [IntrNoMem]>;
````
- **L1369 EN**: Blank line separating nearby declarations or logic blocks.
  **L1369 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1370 EN**: Declares TableGen def `int_arm_mve_vmovl_predicated`.
  **L1370 CN**: 声明 TableGen def `int_arm_mve_vmovl_predicated`。
- **L1371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty, llvm_i32_ty /* unsigned */, llvm_i32_ty /* top half */,`.
  **L1371 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty, llvm_i32_ty /* unsigned */, llvm_i32_ty /* top half */,`。
- **L1372 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty /* predicate */, LLVMMatchType<0>], [IntrNoMem]>;`.
  **L1372 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty /* predicate */, LLVMMatchType<0>], [IntrNoMem]>;`。
- **L1373 EN**: Declares TableGen def `int_arm_mve_vmovn_predicated`.
  **L1373 CN**: 声明 TableGen def `int_arm_mve_vmovn_predicated`。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty, llvm_i32_ty /* top half */,`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty, llvm_i32_ty /* top half */,`。
- **L1375 EN**: Executes a standalone statement or declaration: `llvm_anyvector_ty /* predicate */], [IntrNoMem]>;`.
  **L1375 CN**: 执行一条独立语句或声明：`llvm_anyvector_ty /* predicate */], [IntrNoMem]>;`。
- **L1376 EN**: Blank line separating nearby declarations or logic blocks.
  **L1376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1377 EN**: Declares TableGen def `int_arm_mve_vqmovn`.
  **L1377 CN**: 声明 TableGen def `int_arm_mve_vqmovn`。
- **L1378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1378 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /* unsigned output */, llvm_i32_ty /* unsigned input */,`.
  **L1379 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /* unsigned output */, llvm_i32_ty /* unsigned input */,`。
- **L1380 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /* top half */], [IntrNoMem]>;`.
  **L1380 CN**: 执行一条独立语句或声明：`llvm_i32_ty /* top half */], [IntrNoMem]>;`。
- **L1381 EN**: Declares TableGen def `int_arm_mve_vqmovn_predicated`.
  **L1381 CN**: 声明 TableGen def `int_arm_mve_vqmovn_predicated`。
- **L1382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, llvm_anyvector_ty,`.
  **L1382 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, llvm_anyvector_ty,`。
- **L1383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /* unsigned output */, llvm_i32_ty /* unsigned input */,`.
  **L1383 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /* unsigned output */, llvm_i32_ty /* unsigned input */,`。
- **L1384 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /* top half */, llvm_anyvector_ty /* pred */], [IntrNoMem]>;`.
  **L1384 CN**: 执行一条独立语句或声明：`llvm_i32_ty /* top half */, llvm_anyvector_ty /* pred */], [IntrNoMem]>;`。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1386 EN**: Declares TableGen def `int_arm_mve_fma`.
  **L1386 CN**: 声明 TableGen def `int_arm_mve_fma`。
- **L1387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`.
  **L1387 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`。
- **L1388 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0> /* addend */], [IntrNoMem]>;`.
  **L1388 CN**: 执行一条独立语句或声明：`LLVMMatchType<0> /* addend */], [IntrNoMem]>;`。
- **L1389 EN**: Comment explains nearby logic, invariants, or intent: `fma_predicated returns the add operand for disabled lanes.`.
  **L1389 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`fma_predicated returns the add operand for disabled lanes.`。
- **L1390 EN**: Declares TableGen def `int_arm_mve_fma_predicated`.
  **L1390 CN**: 声明 TableGen def `int_arm_mve_fma_predicated`。
- **L1391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`.
  **L1391 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`。
- **L1392 EN**: Executes a standalone statement or declaration: `LLVMMatchType<0> /* addend */, llvm_anyvector_ty /* pred */], [IntrNoMem]>;`.
  **L1392 CN**: 执行一条独立语句或声明：`LLVMMatchType<0> /* addend */, llvm_anyvector_ty /* pred */], [IntrNoMem]>;`。

### Lines 1393-1416

````tablegen
def int_arm_mve_vmla_n_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,
    llvm_i32_ty /* mult op #2 (scalar) */, llvm_anyvector_ty /* pred */],
   [IntrNoMem]>;
def int_arm_mve_vmlas_n_predicated: DefaultAttrsIntrinsic<[llvm_anyvector_ty],
   [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,
    llvm_i32_ty /* addend (scalar) */, llvm_anyvector_ty /* pred */],
   [IntrNoMem]>;

defm int_arm_mve_vqdmlah: MVEPredicated<[llvm_anyvector_ty],
  [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,
   llvm_i32_ty /* mult op #2 (scalar) */]>;
defm int_arm_mve_vqrdmlah: MVEPredicated<[llvm_anyvector_ty],
  [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,
   llvm_i32_ty /* mult op #2 (scalar) */]>;
defm int_arm_mve_vqdmlash: MVEPredicated<[llvm_anyvector_ty],
  [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,
   llvm_i32_ty /* addend (scalar) */]>;
defm int_arm_mve_vqrdmlash: MVEPredicated<[llvm_anyvector_ty],
  [LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,
   llvm_i32_ty /* addend (scalar) */]>;

defm int_arm_mve_vqdmlad: MVEPredicated<[llvm_anyvector_ty],
  [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,
````
- **L1393 EN**: Declares TableGen def `int_arm_mve_vmla_n_predicated`.
  **L1393 CN**: 声明 TableGen def `int_arm_mve_vmla_n_predicated`。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,`.
  **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /* mult op #2 (scalar) */, llvm_anyvector_ty /* pred */],`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /* mult op #2 (scalar) */, llvm_anyvector_ty /* pred */],`。
- **L1396 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1396 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1397 EN**: Declares TableGen def `int_arm_mve_vmlas_n_predicated`.
  **L1397 CN**: 声明 TableGen def `int_arm_mve_vmlas_n_predicated`。
- **L1398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`.
  **L1398 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /* addend (scalar) */, llvm_anyvector_ty /* pred */],`.
  **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /* addend (scalar) */, llvm_anyvector_ty /* pred */],`。
- **L1400 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1400 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1401 EN**: Blank line separating nearby declarations or logic blocks.
  **L1401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1402 EN**: Declares TableGen defm `int_arm_mve_vqdmlah`.
  **L1402 CN**: 声明 TableGen defm `int_arm_mve_vqdmlah`。
- **L1403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,`.
  **L1403 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,`。
- **L1404 EN**: Executes a call or declaration centered on `#2`.
  **L1404 CN**: 执行以 `#2` 为核心的调用或声明。
- **L1405 EN**: Declares TableGen defm `int_arm_mve_vqrdmlah`.
  **L1405 CN**: 声明 TableGen defm `int_arm_mve_vqrdmlah`。
- **L1406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,`.
  **L1406 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* addend */,`。
- **L1407 EN**: Executes a call or declaration centered on `#2`.
  **L1407 CN**: 执行以 `#2` 为核心的调用或声明。
- **L1408 EN**: Declares TableGen defm `int_arm_mve_vqdmlash`.
  **L1408 CN**: 声明 TableGen defm `int_arm_mve_vqdmlash`。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`.
  **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`。
- **L1410 EN**: Executes a call or declaration centered on `addend`.
  **L1410 CN**: 执行以 `addend` 为核心的调用或声明。
- **L1411 EN**: Declares TableGen defm `int_arm_mve_vqrdmlash`.
  **L1411 CN**: 声明 TableGen defm `int_arm_mve_vqrdmlash`。
- **L1412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`.
  **L1412 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0> /* mult op #1 */, LLVMMatchType<0> /* mult op #2 */,`。
- **L1413 EN**: Executes a call or declaration centered on `addend`.
  **L1413 CN**: 执行以 `addend` 为核心的调用或声明。
- **L1414 EN**: Blank line separating nearby declarations or logic blocks.
  **L1414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1415 EN**: Declares TableGen defm `int_arm_mve_vqdmlad`.
  **L1415 CN**: 声明 TableGen defm `int_arm_mve_vqdmlad`。
- **L1416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L1416 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>,`。

### Lines 1417-1440

````tablegen
   llvm_i32_ty /* exchange */, llvm_i32_ty /* round */,
   llvm_i32_ty /* subtract */]>;

class MVE_cmp : DefaultAttrsIntrinsic<[llvm_anyvector_ty],
    [llvm_anyvector_ty, LLVMMatchType<1>], [IntrNoMem]>;
def int_arm_mve_cmp_eq: MVE_cmp;
def int_arm_mve_cmp_ne: MVE_cmp;
def int_arm_mve_cmp_gt: MVE_cmp;
def int_arm_mve_cmp_ge: MVE_cmp;
def int_arm_mve_cmp_lt: MVE_cmp;
def int_arm_mve_cmp_le: MVE_cmp;

// CDE (Custom Datapath Extension)

multiclass CDEGPRIntrinsics<list<LLVMType> args> {
  def "" : DefaultAttrsIntrinsic<
    [llvm_i32_ty],
    !listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 1)>>]>;
  def a : DefaultAttrsIntrinsic<
    [llvm_i32_ty],
    !listconcat([llvm_i32_ty /* coproc */, llvm_i32_ty /* acc */], args,
                [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 2)>>]>;
````
- **L1417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /* exchange */, llvm_i32_ty /* round */,`.
  **L1417 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /* exchange */, llvm_i32_ty /* round */,`。
- **L1418 EN**: Executes a standalone statement or declaration: `llvm_i32_ty /* subtract */]>;`.
  **L1418 CN**: 执行一条独立语句或声明：`llvm_i32_ty /* subtract */]>;`。
- **L1419 EN**: Blank line separating nearby declarations or logic blocks.
  **L1419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1420 EN**: Declares class `MVE_cmp`.
  **L1420 CN**: 声明 class `MVE_cmp`。
- **L1421 EN**: Executes a standalone statement or declaration: `[llvm_anyvector_ty, LLVMMatchType<1>], [IntrNoMem]>;`.
  **L1421 CN**: 执行一条独立语句或声明：`[llvm_anyvector_ty, LLVMMatchType<1>], [IntrNoMem]>;`。
- **L1422 EN**: Declares TableGen def `int_arm_mve_cmp_eq`.
  **L1422 CN**: 声明 TableGen def `int_arm_mve_cmp_eq`。
- **L1423 EN**: Declares TableGen def `int_arm_mve_cmp_ne`.
  **L1423 CN**: 声明 TableGen def `int_arm_mve_cmp_ne`。
- **L1424 EN**: Declares TableGen def `int_arm_mve_cmp_gt`.
  **L1424 CN**: 声明 TableGen def `int_arm_mve_cmp_gt`。
- **L1425 EN**: Declares TableGen def `int_arm_mve_cmp_ge`.
  **L1425 CN**: 声明 TableGen def `int_arm_mve_cmp_ge`。
- **L1426 EN**: Declares TableGen def `int_arm_mve_cmp_lt`.
  **L1426 CN**: 声明 TableGen def `int_arm_mve_cmp_lt`。
- **L1427 EN**: Declares TableGen def `int_arm_mve_cmp_le`.
  **L1427 CN**: 声明 TableGen def `int_arm_mve_cmp_le`。
- **L1428 EN**: Blank line separating nearby declarations or logic blocks.
  **L1428 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1429 EN**: Comment explains nearby logic, invariants, or intent: `CDE (Custom Datapath Extension)`.
  **L1429 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CDE (Custom Datapath Extension)`。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1431 EN**: Declares TableGen multiclass `CDEGPRIntrinsics`.
  **L1431 CN**: 声明 TableGen multiclass `CDEGPRIntrinsics`。
- **L1432 EN**: Declares TableGen def `""`.
  **L1432 CN**: 声明 TableGen def `""`。
- **L1433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L1433 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L1434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`.
  **L1434 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`。
- **L1435 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1435 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1436 EN**: Declares TableGen def `a`.
  **L1436 CN**: 声明 TableGen def `a`。
- **L1437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty],`.
  **L1437 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty],`。
- **L1438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */, llvm_i32_ty /* acc */], args,`.
  **L1438 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */, llvm_i32_ty /* acc */], args,`。
- **L1439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* imm */]),`.
  **L1439 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* imm */]),`。
- **L1440 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1440 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。

### Lines 1441-1464

````tablegen

  def d: DefaultAttrsIntrinsic<
    [llvm_i32_ty /* lo */, llvm_i32_ty /* hi */],
    !listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 1)>>]>;
  def da: DefaultAttrsIntrinsic<
    [llvm_i32_ty /* lo */, llvm_i32_ty /* hi */],
    !listconcat([llvm_i32_ty /* coproc */, llvm_i32_ty /* acc_lo */,
                 llvm_i32_ty /* acc_hi */], args, [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 3)>>]>;
}

defm int_arm_cde_cx1: CDEGPRIntrinsics<[]>;
defm int_arm_cde_cx2: CDEGPRIntrinsics<[llvm_i32_ty]>;
defm int_arm_cde_cx3: CDEGPRIntrinsics<[llvm_i32_ty, llvm_i32_ty]>;

multiclass CDEVCXIntrinsics<list<LLVMType> args> {
  def "" : DefaultAttrsIntrinsic<
    [llvm_anyfloat_ty],
    !listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 1)>>]>;
  def a : DefaultAttrsIntrinsic<
    [llvm_anyfloat_ty],
    !listconcat([llvm_i32_ty /* coproc */,  LLVMMatchType<0> /* acc */],
````
- **L1441 EN**: Blank line separating nearby declarations or logic blocks.
  **L1441 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1442 EN**: Declares TableGen def `d`.
  **L1442 CN**: 声明 TableGen def `d`。
- **L1443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* lo */, llvm_i32_ty /* hi */],`.
  **L1443 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* lo */, llvm_i32_ty /* hi */],`。
- **L1444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`.
  **L1444 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`。
- **L1445 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1445 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1446 EN**: Declares TableGen def `da`.
  **L1446 CN**: 声明 TableGen def `da`。
- **L1447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty /* lo */, llvm_i32_ty /* hi */],`.
  **L1447 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty /* lo */, llvm_i32_ty /* hi */],`。
- **L1448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */, llvm_i32_ty /* acc_lo */,`.
  **L1448 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */, llvm_i32_ty /* acc_lo */,`。
- **L1449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty /* acc_hi */], args, [llvm_i32_ty /* imm */]),`.
  **L1449 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty /* acc_hi */], args, [llvm_i32_ty /* imm */]),`。
- **L1450 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1450 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1451 EN**: Closes the current lexical scope or compound statement.
  **L1451 CN**: 结束当前词法作用域或复合语句块。
- **L1452 EN**: Blank line separating nearby declarations or logic blocks.
  **L1452 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1453 EN**: Declares TableGen defm `int_arm_cde_cx1`.
  **L1453 CN**: 声明 TableGen defm `int_arm_cde_cx1`。
- **L1454 EN**: Declares TableGen defm `int_arm_cde_cx2`.
  **L1454 CN**: 声明 TableGen defm `int_arm_cde_cx2`。
- **L1455 EN**: Declares TableGen defm `int_arm_cde_cx3`.
  **L1455 CN**: 声明 TableGen defm `int_arm_cde_cx3`。
- **L1456 EN**: Blank line separating nearby declarations or logic blocks.
  **L1456 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1457 EN**: Declares TableGen multiclass `CDEVCXIntrinsics`.
  **L1457 CN**: 声明 TableGen multiclass `CDEVCXIntrinsics`。
- **L1458 EN**: Declares TableGen def `""`.
  **L1458 CN**: 声明 TableGen def `""`。
- **L1459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyfloat_ty],`.
  **L1459 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyfloat_ty],`。
- **L1460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`.
  **L1460 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`。
- **L1461 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1461 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1462 EN**: Declares TableGen def `a`.
  **L1462 CN**: 声明 TableGen def `a`。
- **L1463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyfloat_ty],`.
  **L1463 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyfloat_ty],`。
- **L1464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */,  LLVMMatchType<0> /* acc */],`.
  **L1464 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */,  LLVMMatchType<0> /* acc */],`。

### Lines 1465-1488

````tablegen
                args, [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 2)>>]>;
}

defm int_arm_cde_vcx1 : CDEVCXIntrinsics<[]>;
defm int_arm_cde_vcx2 : CDEVCXIntrinsics<[LLVMMatchType<0>]>;
defm int_arm_cde_vcx3 : CDEVCXIntrinsics<[LLVMMatchType<0>, LLVMMatchType<0>]>;

multiclass CDEVCXVecIntrinsics<list<LLVMType> args> {
  def "" : DefaultAttrsIntrinsic<
    [llvm_v16i8_ty],
    !listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 1)>>]>;
  def a : DefaultAttrsIntrinsic<
    [llvm_v16i8_ty],
    !listconcat([llvm_i32_ty /* coproc */, llvm_v16i8_ty /* acc */],
                args, [llvm_i32_ty /* imm */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 2)>>]>;

  def _predicated : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty],
    !listconcat([llvm_i32_ty /* coproc */, LLVMMatchType<0> /* inactive */],
                args, [llvm_i32_ty /* imm */, llvm_anyvector_ty /* mask */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 2)>>]>;
````
- **L1465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `args, [llvm_i32_ty /* imm */]),`.
  **L1465 CN**: 继续一个多行参数列表、初始化器或聚合项：`args, [llvm_i32_ty /* imm */]),`。
- **L1466 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1466 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1467 EN**: Closes the current lexical scope or compound statement.
  **L1467 CN**: 结束当前词法作用域或复合语句块。
- **L1468 EN**: Blank line separating nearby declarations or logic blocks.
  **L1468 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1469 EN**: Declares TableGen defm `int_arm_cde_vcx1`.
  **L1469 CN**: 声明 TableGen defm `int_arm_cde_vcx1`。
- **L1470 EN**: Declares TableGen defm `int_arm_cde_vcx2`.
  **L1470 CN**: 声明 TableGen defm `int_arm_cde_vcx2`。
- **L1471 EN**: Declares TableGen defm `int_arm_cde_vcx3`.
  **L1471 CN**: 声明 TableGen defm `int_arm_cde_vcx3`。
- **L1472 EN**: Blank line separating nearby declarations or logic blocks.
  **L1472 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1473 EN**: Declares TableGen multiclass `CDEVCXVecIntrinsics`.
  **L1473 CN**: 声明 TableGen multiclass `CDEVCXVecIntrinsics`。
- **L1474 EN**: Declares TableGen def `""`.
  **L1474 CN**: 声明 TableGen def `""`。
- **L1475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty],`.
  **L1475 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty],`。
- **L1476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`.
  **L1476 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */], args, [llvm_i32_ty /* imm */]),`。
- **L1477 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1477 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1478 EN**: Declares TableGen def `a`.
  **L1478 CN**: 声明 TableGen def `a`。
- **L1479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty],`.
  **L1479 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty],`。
- **L1480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */, llvm_v16i8_ty /* acc */],`.
  **L1480 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */, llvm_v16i8_ty /* acc */],`。
- **L1481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `args, [llvm_i32_ty /* imm */]),`.
  **L1481 CN**: 继续一个多行参数列表、初始化器或聚合项：`args, [llvm_i32_ty /* imm */]),`。
- **L1482 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1482 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1483 EN**: Blank line separating nearby declarations or logic blocks.
  **L1483 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1484 EN**: Declares TableGen def `_predicated`.
  **L1484 CN**: 声明 TableGen def `_predicated`。
- **L1485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1485 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */, LLVMMatchType<0> /* inactive */],`.
  **L1486 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */, LLVMMatchType<0> /* inactive */],`。
- **L1487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `args, [llvm_i32_ty /* imm */, llvm_anyvector_ty /* mask */]),`.
  **L1487 CN**: 继续一个多行参数列表、初始化器或聚合项：`args, [llvm_i32_ty /* imm */, llvm_anyvector_ty /* mask */]),`。
- **L1488 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1488 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。

### Lines 1489-1500

````tablegen
  def a_predicated : DefaultAttrsIntrinsic<
    [llvm_anyvector_ty],
    !listconcat([llvm_i32_ty /* coproc */, LLVMMatchType<0> /* acc */],
                args, [llvm_i32_ty /* imm */, llvm_anyvector_ty /* mask */]),
    [IntrNoMem, ImmArg<ArgIndex<0>>, ImmArg<ArgIndex<!add(!size(args), 2)>>]>;
}

defm int_arm_cde_vcx1q : CDEVCXVecIntrinsics<[]>;
defm int_arm_cde_vcx2q : CDEVCXVecIntrinsics<[llvm_v16i8_ty]>;
defm int_arm_cde_vcx3q : CDEVCXVecIntrinsics<[llvm_v16i8_ty, llvm_v16i8_ty]>;

} // end TargetPrefix
````
- **L1489 EN**: Declares TableGen def `a_predicated`.
  **L1489 CN**: 声明 TableGen def `a_predicated`。
- **L1490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyvector_ty],`.
  **L1490 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyvector_ty],`。
- **L1491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_i32_ty /* coproc */, LLVMMatchType<0> /* acc */],`.
  **L1491 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_i32_ty /* coproc */, LLVMMatchType<0> /* acc */],`。
- **L1492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `args, [llvm_i32_ty /* imm */, llvm_anyvector_ty /* mask */]),`.
  **L1492 CN**: 继续一个多行参数列表、初始化器或聚合项：`args, [llvm_i32_ty /* imm */, llvm_anyvector_ty /* mask */]),`。
- **L1493 EN**: Executes a call or declaration centered on `ImmArg<ArgIndex<!add`.
  **L1493 CN**: 执行以 `ImmArg<ArgIndex<!add` 为核心的调用或声明。
- **L1494 EN**: Closes the current lexical scope or compound statement.
  **L1494 CN**: 结束当前词法作用域或复合语句块。
- **L1495 EN**: Blank line separating nearby declarations or logic blocks.
  **L1495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1496 EN**: Declares TableGen defm `int_arm_cde_vcx1q`.
  **L1496 CN**: 声明 TableGen defm `int_arm_cde_vcx1q`。
- **L1497 EN**: Declares TableGen defm `int_arm_cde_vcx2q`.
  **L1497 CN**: 声明 TableGen defm `int_arm_cde_vcx2q`。
- **L1498 EN**: Declares TableGen defm `int_arm_cde_vcx3q`.
  **L1498 CN**: 声明 TableGen defm `int_arm_cde_vcx3q`。
- **L1499 EN**: Blank line separating nearby declarations or logic blocks.
  **L1499 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1500 EN**: Continues the surrounding expression or declaration: `} // end TargetPrefix`.
  **L1500 CN**: 继续构造周围的表达式或声明：`} // end TargetPrefix`。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **ORC JIT runtime and remote execution / ORC JIT 运行时与远程执行**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
