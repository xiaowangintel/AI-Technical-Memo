# IntrinsicsPowerPC.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsPowerPC.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the PowerPC-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsPowerPC` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````tablegen
//===- IntrinsicsPowerPC.td - Defines PowerPC intrinsics ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the PowerPC-specific intrinsics.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Definitions for all PowerPC intrinsics.
//

// Non-altivec intrinsics.
let TargetPrefix = "ppc" in {  // All intrinsics start with "llvm.ppc.".
  // dcba/dcbf/dcbi/dcbst/dcbt/dcbz/dcbzl(PPC970) instructions.
  def int_ppc_dcba  : Intrinsic<[], [llvm_ptr_ty], []>;
  def int_ppc_dcbf : ClangBuiltin<"__builtin_dcbf">,
                      Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;
  def int_ppc_dcbfps : Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;
  def int_ppc_dcbstps : Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;
  def int_ppc_dcbi  : Intrinsic<[], [llvm_ptr_ty], []>;
  def int_ppc_dcbt_with_hint: Intrinsic<[], [llvm_ptr_ty, llvm_i32_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
  def int_ppc_dcbtst_with_hint: Intrinsic<[], [llvm_ptr_ty, llvm_i32_ty],
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the PowerPC-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the PowerPC-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Comment explains nearby logic, invariants, or intent: `Definitions for all PowerPC intrinsics.`.
  **L14 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definitions for all PowerPC intrinsics.`。
- **L15 EN**: Separator comment used for visual grouping.
  **L15 CN**: 用于视觉分组的分隔注释。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, invariants, or intent: `Non-altivec intrinsics.`.
  **L17 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-altivec intrinsics.`。
- **L18 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L18 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L19 EN**: Comment explains nearby logic, invariants, or intent: `dcba/dcbf/dcbi/dcbst/dcbt/dcbz/dcbzl(PPC970) instructions.`.
  **L19 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`dcba/dcbf/dcbi/dcbst/dcbt/dcbz/dcbzl(PPC970) instructions.`。
- **L20 EN**: Declares TableGen def `int_ppc_dcba`.
  **L20 CN**: 声明 TableGen def `int_ppc_dcba`。
- **L21 EN**: Declares TableGen def `int_ppc_dcbf`.
  **L21 CN**: 声明 TableGen def `int_ppc_dcbf`。
- **L22 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;`.
  **L22 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;`。
- **L23 EN**: Declares TableGen def `int_ppc_dcbfps`.
  **L23 CN**: 声明 TableGen def `int_ppc_dcbfps`。
- **L24 EN**: Declares TableGen def `int_ppc_dcbstps`.
  **L24 CN**: 声明 TableGen def `int_ppc_dcbstps`。
- **L25 EN**: Declares TableGen def `int_ppc_dcbi`.
  **L25 CN**: 声明 TableGen def `int_ppc_dcbi`。
- **L26 EN**: Declares TableGen def `int_ppc_dcbt_with_hint`.
  **L26 CN**: 声明 TableGen def `int_ppc_dcbt_with_hint`。
- **L27 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L27 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L28 EN**: Declares TableGen def `int_ppc_dcbtst_with_hint`.
  **L28 CN**: 声明 TableGen def `int_ppc_dcbtst_with_hint`。

### Lines 29-56

````tablegen
    [IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;
  def int_ppc_dcbzl : Intrinsic<[], [llvm_ptr_ty], []>;

  // Emit pseudo instruction as fence of instruction motion
  def int_ppc_fence : ClangBuiltin<"__builtin_ppc_fence">,
                      DefaultAttrsIntrinsic<[], [],
                                            [IntrNoMerge, IntrHasSideEffects]>;

  // Get content from current FPSCR register
  def int_ppc_readflm : ClangBuiltin<"__builtin_readflm">,
                        DefaultAttrsIntrinsic<[llvm_double_ty], [],
                                              [IntrNoMerge, IntrHasSideEffects]>;
  def int_ppc_mffsl : ClangBuiltin<"__builtin_ppc_mffsl">,
                      DefaultAttrsIntrinsic<[llvm_double_ty], [],
                                            [IntrNoMerge, IntrHasSideEffects]>;

  // Set FPSCR register, and return previous content
  def int_ppc_setflm : ClangBuiltin<"__builtin_setflm">,
                       DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty],
                                             [IntrHasSideEffects]>;

  // Intrinsics for [double]word extended forms of divide instructions
  def int_ppc_divwe : ClangBuiltin<"__builtin_divwe">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_divweu : ClangBuiltin<"__builtin_divweu">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
````
- **L29 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`.
  **L29 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>]>;`。
- **L30 EN**: Declares TableGen def `int_ppc_dcbzl`.
  **L30 CN**: 声明 TableGen def `int_ppc_dcbzl`。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Comment explains nearby logic, invariants, or intent: `Emit pseudo instruction as fence of instruction motion`.
  **L32 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Emit pseudo instruction as fence of instruction motion`。
- **L33 EN**: Declares TableGen def `int_ppc_fence`.
  **L33 CN**: 声明 TableGen def `int_ppc_fence`。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [],`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [],`。
- **L35 EN**: Executes a standalone statement or declaration: `[IntrNoMerge, IntrHasSideEffects]>;`.
  **L35 CN**: 执行一条独立语句或声明：`[IntrNoMerge, IntrHasSideEffects]>;`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Comment explains nearby logic, invariants, or intent: `Get content from current FPSCR register`.
  **L37 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get content from current FPSCR register`。
- **L38 EN**: Declares TableGen def `int_ppc_readflm`.
  **L38 CN**: 声明 TableGen def `int_ppc_readflm`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty], [],`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty], [],`。
- **L40 EN**: Executes a standalone statement or declaration: `[IntrNoMerge, IntrHasSideEffects]>;`.
  **L40 CN**: 执行一条独立语句或声明：`[IntrNoMerge, IntrHasSideEffects]>;`。
- **L41 EN**: Declares TableGen def `int_ppc_mffsl`.
  **L41 CN**: 声明 TableGen def `int_ppc_mffsl`。
- **L42 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty], [],`.
  **L42 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty], [],`。
- **L43 EN**: Executes a standalone statement or declaration: `[IntrNoMerge, IntrHasSideEffects]>;`.
  **L43 CN**: 执行一条独立语句或声明：`[IntrNoMerge, IntrHasSideEffects]>;`。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L45 EN**: Comment explains nearby logic, invariants, or intent: `Set FPSCR register, and return previous content`.
  **L45 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Set FPSCR register, and return previous content`。
- **L46 EN**: Declares TableGen def `int_ppc_setflm`.
  **L46 CN**: 声明 TableGen def `int_ppc_setflm`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty],`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty],`。
- **L48 EN**: Executes a standalone statement or declaration: `[IntrHasSideEffects]>;`.
  **L48 CN**: 执行一条独立语句或声明：`[IntrHasSideEffects]>;`。
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsics for [double]word extended forms of divide instructions`.
  **L50 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsics for [double]word extended forms of divide instructions`。
- **L51 EN**: Declares TableGen def `int_ppc_divwe`.
  **L51 CN**: 声明 TableGen def `int_ppc_divwe`。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L53 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L53 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L54 EN**: Declares TableGen def `int_ppc_divweu`.
  **L54 CN**: 声明 TableGen def `int_ppc_divweu`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L56 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L56 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 57-84

````tablegen
  def int_ppc_divde : ClangBuiltin<"__builtin_divde">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                            [IntrNoMem]>;
  def int_ppc_divdeu : ClangBuiltin<"__builtin_divdeu">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                            [IntrNoMem]>;

  def int_ppc_unpack_longdouble : ClangBuiltin<"__builtin_unpack_longdouble">,
      DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_ppcf128_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_pack_longdouble : ClangBuiltin<"__builtin_pack_longdouble">,
      DefaultAttrsIntrinsic<[llvm_ppcf128_ty], [llvm_double_ty, llvm_double_ty],
                            [IntrNoMem]>;

  // Generate a random number
  def int_ppc_darn : ClangBuiltin<"__builtin_darn">,
                     DefaultAttrsIntrinsic<[llvm_i64_ty], [],
                                           [IntrNoMerge, IntrHasSideEffects]>;
  def int_ppc_darnraw : ClangBuiltin<"__builtin_darn_raw">,
                     DefaultAttrsIntrinsic<[llvm_i64_ty], [],
                                           [IntrNoMerge, IntrHasSideEffects]>;
  def int_ppc_darn32 : ClangBuiltin<"__builtin_darn_32">,
                     DefaultAttrsIntrinsic<[llvm_i32_ty], [],
                                           [IntrNoMerge, IntrHasSideEffects]>;

  // Bit permute doubleword
  def int_ppc_bpermd : ClangBuiltin<"__builtin_bpermd">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
````
- **L57 EN**: Declares TableGen def `int_ppc_divde`.
  **L57 CN**: 声明 TableGen def `int_ppc_divde`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L59 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L59 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L60 EN**: Declares TableGen def `int_ppc_divdeu`.
  **L60 CN**: 声明 TableGen def `int_ppc_divdeu`。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L62 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L62 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L64 EN**: Declares TableGen def `int_ppc_unpack_longdouble`.
  **L64 CN**: 声明 TableGen def `int_ppc_unpack_longdouble`。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_ppcf128_ty, llvm_i32_ty],`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_ppcf128_ty, llvm_i32_ty],`。
- **L66 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L66 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L67 EN**: Declares TableGen def `int_ppc_pack_longdouble`.
  **L67 CN**: 声明 TableGen def `int_ppc_pack_longdouble`。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_ppcf128_ty], [llvm_double_ty, llvm_double_ty],`.
  **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_ppcf128_ty], [llvm_double_ty, llvm_double_ty],`。
- **L69 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L69 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Comment explains nearby logic, invariants, or intent: `Generate a random number`.
  **L71 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generate a random number`。
- **L72 EN**: Declares TableGen def `int_ppc_darn`.
  **L72 CN**: 声明 TableGen def `int_ppc_darn`。
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [],`.
  **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [],`。
- **L74 EN**: Executes a standalone statement or declaration: `[IntrNoMerge, IntrHasSideEffects]>;`.
  **L74 CN**: 执行一条独立语句或声明：`[IntrNoMerge, IntrHasSideEffects]>;`。
- **L75 EN**: Declares TableGen def `int_ppc_darnraw`.
  **L75 CN**: 声明 TableGen def `int_ppc_darnraw`。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [],`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [],`。
- **L77 EN**: Executes a standalone statement or declaration: `[IntrNoMerge, IntrHasSideEffects]>;`.
  **L77 CN**: 执行一条独立语句或声明：`[IntrNoMerge, IntrHasSideEffects]>;`。
- **L78 EN**: Declares TableGen def `int_ppc_darn32`.
  **L78 CN**: 声明 TableGen def `int_ppc_darn32`。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [],`.
  **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [],`。
- **L80 EN**: Executes a standalone statement or declaration: `[IntrNoMerge, IntrHasSideEffects]>;`.
  **L80 CN**: 执行一条独立语句或声明：`[IntrNoMerge, IntrHasSideEffects]>;`。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Comment explains nearby logic, invariants, or intent: `Bit permute doubleword`.
  **L82 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bit permute doubleword`。
- **L83 EN**: Declares TableGen def `int_ppc_bpermd`.
  **L83 CN**: 声明 TableGen def `int_ppc_bpermd`。
- **L84 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L84 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。

### Lines 85-112

````tablegen
                            [IntrNoMem]>;

  // Parallel Bits Deposit/Extract Doubleword Builtins.
  def int_ppc_pdepd
      : ClangBuiltin<"__builtin_pdepd">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;
  def int_ppc_pextd
      : ClangBuiltin<"__builtin_pextd">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;

  // Centrifuge Doubleword Builtin.
  def int_ppc_cfuged
      : ClangBuiltin<"__builtin_cfuged">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;

  // Count Leading / Trailing Zeroes under bit Mask Builtins.
  def int_ppc_cntlzdm
      : ClangBuiltin<"__builtin_cntlzdm">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;
  def int_ppc_cnttzdm
      : ClangBuiltin<"__builtin_cnttzdm">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;

````
- **L85 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L85 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L87 EN**: Comment explains nearby logic, invariants, or intent: `Parallel Bits Deposit/Extract Doubleword Builtins.`.
  **L87 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parallel Bits Deposit/Extract Doubleword Builtins.`。
- **L88 EN**: Declares TableGen def `int_ppc_pdepd`.
  **L88 CN**: 声明 TableGen def `int_ppc_pdepd`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_pdepd">,`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_pdepd">,`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L91 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L91 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L92 EN**: Declares TableGen def `int_ppc_pextd`.
  **L92 CN**: 声明 TableGen def `int_ppc_pextd`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_pextd">,`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_pextd">,`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L95 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L95 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L97 EN**: Comment explains nearby logic, invariants, or intent: `Centrifuge Doubleword Builtin.`.
  **L97 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Centrifuge Doubleword Builtin.`。
- **L98 EN**: Declares TableGen def `int_ppc_cfuged`.
  **L98 CN**: 声明 TableGen def `int_ppc_cfuged`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_cfuged">,`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_cfuged">,`。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L101 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L101 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L103 EN**: Comment explains nearby logic, invariants, or intent: `Count Leading / Trailing Zeroes under bit Mask Builtins.`.
  **L103 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Count Leading / Trailing Zeroes under bit Mask Builtins.`。
- **L104 EN**: Declares TableGen def `int_ppc_cntlzdm`.
  **L104 CN**: 声明 TableGen def `int_ppc_cntlzdm`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_cntlzdm">,`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_cntlzdm">,`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L107 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L107 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L108 EN**: Declares TableGen def `int_ppc_cnttzdm`.
  **L108 CN**: 声明 TableGen def `int_ppc_cnttzdm`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_cnttzdm">,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_cnttzdm">,`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L111 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L111 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 113-140

````tablegen
  def int_ppc_truncf128_round_to_odd
      : ClangBuiltin<"__builtin_truncf128_round_to_odd">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_f128_ty], [IntrNoMem]>;
  def int_ppc_sqrtf128_round_to_odd
      : ClangBuiltin<"__builtin_sqrtf128_round_to_odd">,
        DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty], [IntrNoMem]>;
  def int_ppc_addf128_round_to_odd
      : ClangBuiltin<"__builtin_addf128_round_to_odd">,
        DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],
                              [IntrNoMem]>;
  def int_ppc_subf128_round_to_odd
      : ClangBuiltin<"__builtin_subf128_round_to_odd">,
        DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],
                              [IntrNoMem]>;
  def int_ppc_mulf128_round_to_odd
      : ClangBuiltin<"__builtin_mulf128_round_to_odd">,
        DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],
                              [IntrNoMem]>;
  def int_ppc_divf128_round_to_odd
      : ClangBuiltin<"__builtin_divf128_round_to_odd">,
        DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],
                              [IntrNoMem]>;
  def int_ppc_fmaf128_round_to_odd
      : ClangBuiltin<"__builtin_fmaf128_round_to_odd">,
        DefaultAttrsIntrinsic<[llvm_f128_ty],
                              [llvm_f128_ty,llvm_f128_ty,llvm_f128_ty],
                              [IntrNoMem]>;
  def int_ppc_scalar_extract_expq
````
- **L113 EN**: Declares TableGen def `int_ppc_truncf128_round_to_odd`.
  **L113 CN**: 声明 TableGen def `int_ppc_truncf128_round_to_odd`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_truncf128_round_to_odd">,`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_truncf128_round_to_odd">,`。
- **L115 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_f128_ty], [IntrNoMem]>;`.
  **L115 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_f128_ty], [IntrNoMem]>;`。
- **L116 EN**: Declares TableGen def `int_ppc_sqrtf128_round_to_odd`.
  **L116 CN**: 声明 TableGen def `int_ppc_sqrtf128_round_to_odd`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_sqrtf128_round_to_odd">,`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_sqrtf128_round_to_odd">,`。
- **L118 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty], [IntrNoMem]>;`.
  **L118 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty], [IntrNoMem]>;`。
- **L119 EN**: Declares TableGen def `int_ppc_addf128_round_to_odd`.
  **L119 CN**: 声明 TableGen def `int_ppc_addf128_round_to_odd`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_addf128_round_to_odd">,`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_addf128_round_to_odd">,`。
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`。
- **L122 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L122 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L123 EN**: Declares TableGen def `int_ppc_subf128_round_to_odd`.
  **L123 CN**: 声明 TableGen def `int_ppc_subf128_round_to_odd`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_subf128_round_to_odd">,`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_subf128_round_to_odd">,`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`。
- **L126 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L126 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L127 EN**: Declares TableGen def `int_ppc_mulf128_round_to_odd`.
  **L127 CN**: 声明 TableGen def `int_ppc_mulf128_round_to_odd`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_mulf128_round_to_odd">,`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_mulf128_round_to_odd">,`。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`。
- **L130 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L130 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L131 EN**: Declares TableGen def `int_ppc_divf128_round_to_odd`.
  **L131 CN**: 声明 TableGen def `int_ppc_divf128_round_to_odd`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_divf128_round_to_odd">,`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_divf128_round_to_odd">,`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty,llvm_f128_ty],`。
- **L134 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L134 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L135 EN**: Declares TableGen def `int_ppc_fmaf128_round_to_odd`.
  **L135 CN**: 声明 TableGen def `int_ppc_fmaf128_round_to_odd`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_fmaf128_round_to_odd">,`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_fmaf128_round_to_odd">,`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_f128_ty],`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_f128_ty],`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_f128_ty,llvm_f128_ty,llvm_f128_ty],`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_f128_ty,llvm_f128_ty,llvm_f128_ty],`。
- **L139 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L139 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L140 EN**: Declares TableGen def `int_ppc_scalar_extract_expq`.
  **L140 CN**: 声明 TableGen def `int_ppc_scalar_extract_expq`。

### Lines 141-168

````tablegen
      : ClangBuiltin<"__builtin_vsx_scalar_extract_expq">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_f128_ty], [IntrNoMem]>;
  def int_ppc_scalar_insert_exp_qp
      : ClangBuiltin<"__builtin_vsx_scalar_insert_exp_qp">,
        DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty, llvm_i64_ty],
                              [IntrNoMem]>;

  // Intrinsics defined to maintain XL compatibility
  def int_ppc_tdw
      : ClangBuiltin<"__builtin_ppc_tdw">,
        Intrinsic <[], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty], [ImmArg<ArgIndex<2>>]>;
  def int_ppc_tw
      : ClangBuiltin<"__builtin_ppc_tw">,
        Intrinsic <[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<2>>]>;
  def int_ppc_trapd
      : ClangBuiltin<"__builtin_ppc_trapd">,
        Intrinsic <[], [llvm_i64_ty], []>;
  def int_ppc_trap
      : ClangBuiltin<"__builtin_ppc_trap">,
        Intrinsic <[], [llvm_i32_ty], []>;
  def int_ppc_fcfid
      : ClangBuiltin<"__builtin_ppc_fcfid">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fcfud
      : ClangBuiltin<"__builtin_ppc_fcfud">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fctid
      : ClangBuiltin<"__builtin_ppc_fctid">,
````
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_vsx_scalar_extract_expq">,`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_vsx_scalar_extract_expq">,`。
- **L142 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_f128_ty], [IntrNoMem]>;`.
  **L142 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_f128_ty], [IntrNoMem]>;`。
- **L143 EN**: Declares TableGen def `int_ppc_scalar_insert_exp_qp`.
  **L143 CN**: 声明 TableGen def `int_ppc_scalar_insert_exp_qp`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_vsx_scalar_insert_exp_qp">,`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_vsx_scalar_insert_exp_qp">,`。
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty, llvm_i64_ty],`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_f128_ty, llvm_i64_ty],`。
- **L146 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L146 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsics defined to maintain XL compatibility`.
  **L148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsics defined to maintain XL compatibility`。
- **L149 EN**: Declares TableGen def `int_ppc_tdw`.
  **L149 CN**: 声明 TableGen def `int_ppc_tdw`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_tdw">,`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_tdw">,`。
- **L151 EN**: Executes a standalone statement or declaration: `Intrinsic <[], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty], [ImmArg<ArgIndex<2>>]>;`.
  **L151 CN**: 执行一条独立语句或声明：`Intrinsic <[], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty], [ImmArg<ArgIndex<2>>]>;`。
- **L152 EN**: Declares TableGen def `int_ppc_tw`.
  **L152 CN**: 声明 TableGen def `int_ppc_tw`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_tw">,`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_tw">,`。
- **L154 EN**: Executes a standalone statement or declaration: `Intrinsic <[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<2>>]>;`.
  **L154 CN**: 执行一条独立语句或声明：`Intrinsic <[], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], [ImmArg<ArgIndex<2>>]>;`。
- **L155 EN**: Declares TableGen def `int_ppc_trapd`.
  **L155 CN**: 声明 TableGen def `int_ppc_trapd`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_trapd">,`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_trapd">,`。
- **L157 EN**: Executes a standalone statement or declaration: `Intrinsic <[], [llvm_i64_ty], []>;`.
  **L157 CN**: 执行一条独立语句或声明：`Intrinsic <[], [llvm_i64_ty], []>;`。
- **L158 EN**: Declares TableGen def `int_ppc_trap`.
  **L158 CN**: 声明 TableGen def `int_ppc_trap`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_trap">,`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_trap">,`。
- **L160 EN**: Executes a standalone statement or declaration: `Intrinsic <[], [llvm_i32_ty], []>;`.
  **L160 CN**: 执行一条独立语句或声明：`Intrinsic <[], [llvm_i32_ty], []>;`。
- **L161 EN**: Declares TableGen def `int_ppc_fcfid`.
  **L161 CN**: 声明 TableGen def `int_ppc_fcfid`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fcfid">,`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fcfid">,`。
- **L163 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L163 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L164 EN**: Declares TableGen def `int_ppc_fcfud`.
  **L164 CN**: 声明 TableGen def `int_ppc_fcfud`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fcfud">,`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fcfud">,`。
- **L166 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L166 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L167 EN**: Declares TableGen def `int_ppc_fctid`.
  **L167 CN**: 声明 TableGen def `int_ppc_fctid`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fctid">,`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fctid">,`。

### Lines 169-196

````tablegen
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fctidz
      : ClangBuiltin<"__builtin_ppc_fctidz">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fctiw
      : ClangBuiltin<"__builtin_ppc_fctiw">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fctiwz
      : ClangBuiltin<"__builtin_ppc_fctiwz">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fctudz
      : ClangBuiltin<"__builtin_ppc_fctudz">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fctuwz
      : ClangBuiltin<"__builtin_ppc_fctuwz">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_rlwimi
      : ClangBuiltin<"__builtin_ppc_rlwimi">,
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;
  def int_ppc_rlwnm
      : ClangBuiltin<"__builtin_ppc_rlwnm">,
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_ppc_rldimi
      : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty, llvm_i64_ty],
                              [IntrNoMem, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;

````
- **L169 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L169 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L170 EN**: Declares TableGen def `int_ppc_fctidz`.
  **L170 CN**: 声明 TableGen def `int_ppc_fctidz`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fctidz">,`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fctidz">,`。
- **L172 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L172 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L173 EN**: Declares TableGen def `int_ppc_fctiw`.
  **L173 CN**: 声明 TableGen def `int_ppc_fctiw`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fctiw">,`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fctiw">,`。
- **L175 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L175 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L176 EN**: Declares TableGen def `int_ppc_fctiwz`.
  **L176 CN**: 声明 TableGen def `int_ppc_fctiwz`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fctiwz">,`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fctiwz">,`。
- **L178 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L178 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L179 EN**: Declares TableGen def `int_ppc_fctudz`.
  **L179 CN**: 声明 TableGen def `int_ppc_fctudz`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fctudz">,`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fctudz">,`。
- **L181 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L181 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L182 EN**: Declares TableGen def `int_ppc_fctuwz`.
  **L182 CN**: 声明 TableGen def `int_ppc_fctuwz`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fctuwz">,`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fctuwz">,`。
- **L184 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L184 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L185 EN**: Declares TableGen def `int_ppc_rlwimi`.
  **L185 CN**: 声明 TableGen def `int_ppc_rlwimi`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_rlwimi">,`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_rlwimi">,`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L188 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L188 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L189 EN**: Declares TableGen def `int_ppc_rlwnm`.
  **L189 CN**: 声明 TableGen def `int_ppc_rlwnm`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_rlwnm">,`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_rlwnm">,`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L192 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L192 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L193 EN**: Declares TableGen def `int_ppc_rldimi`.
  **L193 CN**: 声明 TableGen def `int_ppc_rldimi`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty, llvm_i64_ty],`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty, llvm_i64_ty],`。
- **L195 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`.
  **L195 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>, ImmArg<ArgIndex<3>>]>;`。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 197-224

````tablegen
  // XL compatible select functions
  // TODO: Add llvm_f128_ty support.
  def int_ppc_maxfe
      : DefaultAttrsIntrinsic<
            [llvm_ppcf128_ty],
            [llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_vararg_ty],
            [IntrNoMem]>;
  def int_ppc_maxfl
      : DefaultAttrsIntrinsic<
            [llvm_double_ty],
            [llvm_double_ty, llvm_double_ty, llvm_double_ty, llvm_vararg_ty],
            [IntrNoMem]>;
  def int_ppc_maxfs
      : DefaultAttrsIntrinsic<
            [llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_vararg_ty],
            [IntrNoMem]>;
  def int_ppc_minfe
      : DefaultAttrsIntrinsic<
            [llvm_ppcf128_ty],
            [llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_vararg_ty],
            [IntrNoMem]>;
  def int_ppc_minfl
      : DefaultAttrsIntrinsic<
            [llvm_double_ty],
            [llvm_double_ty, llvm_double_ty, llvm_double_ty, llvm_vararg_ty],
            [IntrNoMem]>;
  def int_ppc_minfs
````
- **L197 EN**: Comment explains nearby logic, invariants, or intent: `XL compatible select functions`.
  **L197 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`XL compatible select functions`。
- **L198 EN**: Comment records a pending task or caution: `TODO: Add llvm_f128_ty support.`.
  **L198 CN**: 注释记录了待办事项或注意点：`TODO: Add llvm_f128_ty support.`。
- **L199 EN**: Declares TableGen def `int_ppc_maxfe`.
  **L199 CN**: 声明 TableGen def `int_ppc_maxfe`。
- **L200 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L200 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ppcf128_ty],`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ppcf128_ty],`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_vararg_ty],`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_vararg_ty],`。
- **L203 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L203 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L204 EN**: Declares TableGen def `int_ppc_maxfl`.
  **L204 CN**: 声明 TableGen def `int_ppc_maxfl`。
- **L205 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L205 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty],`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty],`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty, llvm_double_ty, llvm_vararg_ty],`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty, llvm_double_ty, llvm_vararg_ty],`。
- **L208 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L208 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L209 EN**: Declares TableGen def `int_ppc_maxfs`.
  **L209 CN**: 声明 TableGen def `int_ppc_maxfs`。
- **L210 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L210 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty],`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty],`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_vararg_ty],`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_vararg_ty],`。
- **L213 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L213 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L214 EN**: Declares TableGen def `int_ppc_minfe`.
  **L214 CN**: 声明 TableGen def `int_ppc_minfe`。
- **L215 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L215 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ppcf128_ty],`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ppcf128_ty],`。
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_vararg_ty],`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_ppcf128_ty, llvm_vararg_ty],`。
- **L218 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L218 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L219 EN**: Declares TableGen def `int_ppc_minfl`.
  **L219 CN**: 声明 TableGen def `int_ppc_minfl`。
- **L220 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L220 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty],`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty],`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty, llvm_double_ty, llvm_vararg_ty],`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty, llvm_double_ty, llvm_vararg_ty],`。
- **L223 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L223 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L224 EN**: Declares TableGen def `int_ppc_minfs`.
  **L224 CN**: 声明 TableGen def `int_ppc_minfs`。

### Lines 225-252

````tablegen
      : DefaultAttrsIntrinsic<
            [llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_vararg_ty],
            [IntrNoMem]>;
  // Load of a value provided by the system library at a fixed address. Used for
  // accessing things like the HWCAP word provided by Glibc. The immediate
  // argument is not an address but a value defined in
  // include/llvm/TargetParser/PPCTargetParser.def. Each of the values provided
  // by Glibc is a 32-bit word.
  def int_ppc_fixed_addr_ld
      : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty],
                              [IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;

}

let TargetPrefix = "ppc" in {  // All PPC intrinsics start with "llvm.ppc.".
  /// PowerPC_Vec_Intrinsic - Base class for all altivec intrinsics.
  class PowerPC_Vec_Intrinsic<string GCCIntSuffix, list<LLVMType> ret_types,
                              list<LLVMType> param_types,
                              list<IntrinsicProperty> properties>
    : ClangBuiltin<!strconcat("__builtin_altivec_", GCCIntSuffix)>,
      DefaultAttrsIntrinsic<ret_types, param_types, properties>;

  /// PowerPC_VSX_Intrinsic - Base class for all VSX intrinsics.
  class PowerPC_VSX_Intrinsic<string GCCIntSuffix, list<LLVMType> ret_types,
                              list<LLVMType> param_types,
                              list<IntrinsicProperty> properties>
    : ClangBuiltin<!strconcat("__builtin_vsx_", GCCIntSuffix)>,
````
- **L225 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L225 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty],`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty],`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_vararg_ty],`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty, llvm_vararg_ty],`。
- **L228 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L228 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L229 EN**: Comment explains nearby logic, invariants, or intent: `Load of a value provided by the system library at a fixed address. Used for`.
  **L229 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Load of a value provided by the system library at a fixed address. Used for`。
- **L230 EN**: Comment explains nearby logic, invariants, or intent: `accessing things like the HWCAP word provided by Glibc. The immediate`.
  **L230 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`accessing things like the HWCAP word provided by Glibc. The immediate`。
- **L231 EN**: Comment explains nearby logic, invariants, or intent: `argument is not an address but a value defined in`.
  **L231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`argument is not an address but a value defined in`。
- **L232 EN**: Comment explains nearby logic, invariants, or intent: `include/llvm/TargetParser/PPCTargetParser.def. Each of the values provided`.
  **L232 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`include/llvm/TargetParser/PPCTargetParser.def. Each of the values provided`。
- **L233 EN**: Comment explains nearby logic, invariants, or intent: `by Glibc is a 32-bit word.`.
  **L233 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`by Glibc is a 32-bit word.`。
- **L234 EN**: Declares TableGen def `int_ppc_fixed_addr_ld`.
  **L234 CN**: 声明 TableGen def `int_ppc_fixed_addr_ld`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty],`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty],`。
- **L236 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`.
  **L236 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, ImmArg<ArgIndex<0>>]>;`。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L238 EN**: Closes the current lexical scope or compound statement.
  **L238 CN**: 结束当前词法作用域或复合语句块。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L240 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L240 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L241 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_Intrinsic - Base class for all altivec intrinsics.`.
  **L241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_Intrinsic - Base class for all altivec intrinsics.`。
- **L242 EN**: Declares class `PowerPC_Vec_Intrinsic<string`.
  **L242 CN**: 声明 class `PowerPC_Vec_Intrinsic<string`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> param_types,`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> param_types,`。
- **L244 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> properties>`.
  **L244 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> properties>`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<!strconcat("__builtin_altivec_", GCCIntSuffix)>,`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<!strconcat("__builtin_altivec_", GCCIntSuffix)>,`。
- **L246 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<ret_types, param_types, properties>;`.
  **L246 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<ret_types, param_types, properties>;`。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L248 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_VSX_Intrinsic - Base class for all VSX intrinsics.`.
  **L248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_VSX_Intrinsic - Base class for all VSX intrinsics.`。
- **L249 EN**: Declares class `PowerPC_VSX_Intrinsic<string`.
  **L249 CN**: 声明 class `PowerPC_VSX_Intrinsic<string`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> param_types,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> param_types,`。
- **L251 EN**: Continues the surrounding expression or declaration: `list<IntrinsicProperty> properties>`.
  **L251 CN**: 继续构造周围的表达式或声明：`list<IntrinsicProperty> properties>`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<!strconcat("__builtin_vsx_", GCCIntSuffix)>,`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<!strconcat("__builtin_vsx_", GCCIntSuffix)>,`。

### Lines 253-280

````tablegen
      DefaultAttrsIntrinsic<ret_types, param_types, properties>;
}

//===----------------------------------------------------------------------===//
// PowerPC MMA Intrinsic Multi Class Definitions.
//

multiclass PowerPC_MMA_ACC_Intrinsic<list<LLVMType> args> {
  def NAME: DefaultAttrsIntrinsic<[llvm_v512i1_ty], args, [IntrNoMem]>;
  def pp : DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                                 !listconcat([llvm_v512i1_ty], args),
                                 [IntrNoMem]>;
  def pn : DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                                 !listconcat([llvm_v512i1_ty], args),
                                 [IntrNoMem]>;
  def np : DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                                 !listconcat([llvm_v512i1_ty], args),
                                 [IntrNoMem]>;
  def nn : DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                                 !listconcat([llvm_v512i1_ty], args),
                                 [IntrNoMem]>;
}

multiclass PowerPC_MMA_ACC_PP_Intrinsic<list<LLVMType> args> {
  def NAME: DefaultAttrsIntrinsic<[llvm_v512i1_ty], args, [IntrNoMem]>;
  def pp : DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                                 !listconcat([llvm_v512i1_ty], args),
                                 [IntrNoMem]>;
````
- **L253 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<ret_types, param_types, properties>;`.
  **L253 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<ret_types, param_types, properties>;`。
- **L254 EN**: Closes the current lexical scope or compound statement.
  **L254 CN**: 结束当前词法作用域或复合语句块。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Banner comment marking a file or section boundary.
  **L256 CN**: 横幅注释，用于标记文件或章节边界。
- **L257 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC MMA Intrinsic Multi Class Definitions.`.
  **L257 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC MMA Intrinsic Multi Class Definitions.`。
- **L258 EN**: Separator comment used for visual grouping.
  **L258 CN**: 用于视觉分组的分隔注释。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L260 EN**: Declares TableGen multiclass `PowerPC_MMA_ACC_Intrinsic`.
  **L260 CN**: 声明 TableGen multiclass `PowerPC_MMA_ACC_Intrinsic`。
- **L261 EN**: Declares TableGen def `NAME`.
  **L261 CN**: 声明 TableGen def `NAME`。
- **L262 EN**: Declares TableGen def `pp`.
  **L262 CN**: 声明 TableGen def `pp`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v512i1_ty], args),`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v512i1_ty], args),`。
- **L264 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L264 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L265 EN**: Declares TableGen def `pn`.
  **L265 CN**: 声明 TableGen def `pn`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v512i1_ty], args),`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v512i1_ty], args),`。
- **L267 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L267 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L268 EN**: Declares TableGen def `np`.
  **L268 CN**: 声明 TableGen def `np`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v512i1_ty], args),`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v512i1_ty], args),`。
- **L270 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L270 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L271 EN**: Declares TableGen def `nn`.
  **L271 CN**: 声明 TableGen def `nn`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v512i1_ty], args),`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v512i1_ty], args),`。
- **L273 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L273 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L274 EN**: Closes the current lexical scope or compound statement.
  **L274 CN**: 结束当前词法作用域或复合语句块。
- **L275 EN**: Blank line separating nearby declarations or logic blocks.
  **L275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L276 EN**: Declares TableGen multiclass `PowerPC_MMA_ACC_PP_Intrinsic`.
  **L276 CN**: 声明 TableGen multiclass `PowerPC_MMA_ACC_PP_Intrinsic`。
- **L277 EN**: Declares TableGen def `NAME`.
  **L277 CN**: 声明 TableGen def `NAME`。
- **L278 EN**: Declares TableGen def `pp`.
  **L278 CN**: 声明 TableGen def `pp`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v512i1_ty], args),`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v512i1_ty], args),`。
- **L280 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L280 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 281-308

````tablegen
}

multiclass PowerPC_MMA_DMR_Intrinsic<list<LLVMType> args> {
  def NAME: DefaultAttrsIntrinsic<[llvm_v1024i1_ty], args, [IntrNoMem]>;
  def pp : DefaultAttrsIntrinsic<[llvm_v1024i1_ty],
                                 !listconcat([llvm_v1024i1_ty], args),
                                 [IntrNoMem]>;
  def pn : DefaultAttrsIntrinsic<[llvm_v1024i1_ty],
                                 !listconcat([llvm_v1024i1_ty], args),
                                 [IntrNoMem]>;
  def np : DefaultAttrsIntrinsic<[llvm_v1024i1_ty],
                                 !listconcat([llvm_v1024i1_ty], args),
                                 [IntrNoMem]>;
  def nn : DefaultAttrsIntrinsic<[llvm_v1024i1_ty],
                                 !listconcat([llvm_v1024i1_ty], args),
                                 [IntrNoMem]>;
}

multiclass PowerPC_MMA_DMR_PP_Intrinsic<list<LLVMType> args> {
  def NAME: DefaultAttrsIntrinsic<[llvm_v1024i1_ty], args, [IntrNoMem]>;
  def pp : DefaultAttrsIntrinsic<[llvm_v1024i1_ty],
                                 !listconcat([llvm_v1024i1_ty], args),
                                 [IntrNoMem]>;
}

//===----------------------------------------------------------------------===//
// PowerPC Altivec Intrinsic Class Definitions.
//
````
- **L281 EN**: Closes the current lexical scope or compound statement.
  **L281 CN**: 结束当前词法作用域或复合语句块。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L283 EN**: Declares TableGen multiclass `PowerPC_MMA_DMR_Intrinsic`.
  **L283 CN**: 声明 TableGen multiclass `PowerPC_MMA_DMR_Intrinsic`。
- **L284 EN**: Declares TableGen def `NAME`.
  **L284 CN**: 声明 TableGen def `NAME`。
- **L285 EN**: Declares TableGen def `pp`.
  **L285 CN**: 声明 TableGen def `pp`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v1024i1_ty], args),`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v1024i1_ty], args),`。
- **L287 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L287 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L288 EN**: Declares TableGen def `pn`.
  **L288 CN**: 声明 TableGen def `pn`。
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v1024i1_ty], args),`.
  **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v1024i1_ty], args),`。
- **L290 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L290 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L291 EN**: Declares TableGen def `np`.
  **L291 CN**: 声明 TableGen def `np`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v1024i1_ty], args),`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v1024i1_ty], args),`。
- **L293 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L293 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L294 EN**: Declares TableGen def `nn`.
  **L294 CN**: 声明 TableGen def `nn`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v1024i1_ty], args),`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v1024i1_ty], args),`。
- **L296 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L296 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L297 EN**: Closes the current lexical scope or compound statement.
  **L297 CN**: 结束当前词法作用域或复合语句块。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L299 EN**: Declares TableGen multiclass `PowerPC_MMA_DMR_PP_Intrinsic`.
  **L299 CN**: 声明 TableGen multiclass `PowerPC_MMA_DMR_PP_Intrinsic`。
- **L300 EN**: Declares TableGen def `NAME`.
  **L300 CN**: 声明 TableGen def `NAME`。
- **L301 EN**: Declares TableGen def `pp`.
  **L301 CN**: 声明 TableGen def `pp`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_v1024i1_ty], args),`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_v1024i1_ty], args),`。
- **L303 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L303 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L304 EN**: Closes the current lexical scope or compound statement.
  **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L306 EN**: Banner comment marking a file or section boundary.
  **L306 CN**: 横幅注释，用于标记文件或章节边界。
- **L307 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC Altivec Intrinsic Class Definitions.`.
  **L307 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC Altivec Intrinsic Class Definitions.`。
- **L308 EN**: Separator comment used for visual grouping.
  **L308 CN**: 用于视觉分组的分隔注释。

### Lines 309-336

````tablegen

/// PowerPC_Vec_FF_Intrinsic - A PowerPC intrinsic that takes one v4f32
/// vector and returns one.  These intrinsics have no side effects.
class PowerPC_Vec_FF_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                          [llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;

/// PowerPC_Vec_FFF_Intrinsic - A PowerPC intrinsic that takes two v4f32
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_Vec_FFF_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                          [llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],
                          [IntrNoMem]>;

/// PowerPC_Vec_BBB_Intrinsic - A PowerPC intrinsic that takes two v16i8
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_Vec_BBB_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                          [llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                          [IntrNoMem]>;

/// PowerPC_Vec_HHH_Intrinsic - A PowerPC intrinsic that takes two v8i16
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_Vec_HHH_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                          [llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                          [IntrNoMem]>;

````
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_FF_Intrinsic - A PowerPC intrinsic that takes one v4f32`.
  **L310 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_FF_Intrinsic - A PowerPC intrinsic that takes one v4f32`。
- **L311 EN**: Comment explains nearby logic, invariants, or intent: `vector and returns one.  These intrinsics have no side effects.`.
  **L311 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vector and returns one.  These intrinsics have no side effects.`。
- **L312 EN**: Declares class `PowerPC_Vec_FF_Intrinsic<string`.
  **L312 CN**: 声明 class `PowerPC_Vec_FF_Intrinsic<string`。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L314 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L314 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L316 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_FFF_Intrinsic - A PowerPC intrinsic that takes two v4f32`.
  **L316 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_FFF_Intrinsic - A PowerPC intrinsic that takes two v4f32`。
- **L317 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L317 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L318 EN**: Declares class `PowerPC_Vec_FFF_Intrinsic<string`.
  **L318 CN**: 声明 class `PowerPC_Vec_FFF_Intrinsic<string`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L321 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L321 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L323 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_BBB_Intrinsic - A PowerPC intrinsic that takes two v16i8`.
  **L323 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_BBB_Intrinsic - A PowerPC intrinsic that takes two v16i8`。
- **L324 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L324 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L325 EN**: Declares class `PowerPC_Vec_BBB_Intrinsic<string`.
  **L325 CN**: 声明 class `PowerPC_Vec_BBB_Intrinsic<string`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L328 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L328 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_HHH_Intrinsic - A PowerPC intrinsic that takes two v8i16`.
  **L330 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_HHH_Intrinsic - A PowerPC intrinsic that takes two v8i16`。
- **L331 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L331 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L332 EN**: Declares class `PowerPC_Vec_HHH_Intrinsic<string`.
  **L332 CN**: 声明 class `PowerPC_Vec_HHH_Intrinsic<string`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L335 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L335 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L336 EN**: Blank line separating nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 337-364

````tablegen
/// PowerPC_Vec_WWW_Intrinsic - A PowerPC intrinsic that takes two v4i32
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_Vec_WWW_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                          [llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                          [IntrNoMem]>;

/// PowerPC_Vec_DDD_Intrinsic - A PowerPC intrinsic that takes two v2i64
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_Vec_DDD_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                          [llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                          [IntrNoMem]>;

/// PowerPC_Vec_QQQ_Intrinsic - A PowerPC intrinsic that takes two v1i128
/// vectors and returns one. These intrinsics have no side effects.
class PowerPC_Vec_QQQ_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                         [llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],
                         [IntrNoMem]>;

/// PowerPC_Vec_QDD_Intrinsic - A PowerPC intrinsic that takes two v2i64
/// vectors and returns one v1i128. These intrinsics have no side effects.
class PowerPC_Vec_QDD_Intrinsic<string GCCIntSuffix>
  : PowerPC_Vec_Intrinsic<GCCIntSuffix,
                          [llvm_v1i128_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                          [IntrNoMem]>;

````
- **L337 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_WWW_Intrinsic - A PowerPC intrinsic that takes two v4i32`.
  **L337 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_WWW_Intrinsic - A PowerPC intrinsic that takes two v4i32`。
- **L338 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L338 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L339 EN**: Declares class `PowerPC_Vec_WWW_Intrinsic<string`.
  **L339 CN**: 声明 class `PowerPC_Vec_WWW_Intrinsic<string`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L342 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L342 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L344 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_DDD_Intrinsic - A PowerPC intrinsic that takes two v2i64`.
  **L344 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_DDD_Intrinsic - A PowerPC intrinsic that takes two v2i64`。
- **L345 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L345 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L346 EN**: Declares class `PowerPC_Vec_DDD_Intrinsic<string`.
  **L346 CN**: 声明 class `PowerPC_Vec_DDD_Intrinsic<string`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L349 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L349 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L351 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_QQQ_Intrinsic - A PowerPC intrinsic that takes two v1i128`.
  **L351 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_QQQ_Intrinsic - A PowerPC intrinsic that takes two v1i128`。
- **L352 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one. These intrinsics have no side effects.`.
  **L352 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one. These intrinsics have no side effects.`。
- **L353 EN**: Declares class `PowerPC_Vec_QQQ_Intrinsic<string`.
  **L353 CN**: 声明 class `PowerPC_Vec_QQQ_Intrinsic<string`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L356 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L356 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L357 EN**: Blank line separating nearby declarations or logic blocks.
  **L357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L358 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_Vec_QDD_Intrinsic - A PowerPC intrinsic that takes two v2i64`.
  **L358 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_Vec_QDD_Intrinsic - A PowerPC intrinsic that takes two v2i64`。
- **L359 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one v1i128. These intrinsics have no side effects.`.
  **L359 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one v1i128. These intrinsics have no side effects.`。
- **L360 EN**: Declares class `PowerPC_Vec_QDD_Intrinsic<string`.
  **L360 CN**: 声明 class `PowerPC_Vec_QDD_Intrinsic<string`。
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_Vec_Intrinsic<GCCIntSuffix,`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_Vec_Intrinsic<GCCIntSuffix,`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L363 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L363 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 365-392

````tablegen
//===----------------------------------------------------------------------===//
// PowerPC VSX Intrinsic Class Definitions.
//

/// PowerPC_VSX_Vec_DDD_Intrinsic - A PowerPC intrinsic that takes two v2f64
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_VSX_Vec_DDD_Intrinsic<string GCCIntSuffix>
  : PowerPC_VSX_Intrinsic<GCCIntSuffix,
                          [llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty],
                          [IntrNoMem]>;

/// PowerPC_VSX_Vec_FFF_Intrinsic - A PowerPC intrinsic that takes two v4f32
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_VSX_Vec_FFF_Intrinsic<string GCCIntSuffix>
  : PowerPC_VSX_Intrinsic<GCCIntSuffix,
                          [llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],
                          [IntrNoMem]>;

/// PowerPC_VSX_Sca_DDD_Intrinsic - A PowerPC intrinsic that takes two f64
/// scalars and returns one.  These intrinsics have no side effects.
class PowerPC_VSX_Sca_DDD_Intrinsic<string GCCIntSuffix>
  : PowerPC_VSX_Intrinsic<GCCIntSuffix,
                          [llvm_double_ty], [llvm_double_ty, llvm_double_ty],
                          [IntrNoMem]>;

/// PowerPC_VSX_WWW_Intrinsic - A PowerPC intrinsic that takes two v4i32
/// vectors and returns one.  These intrinsics have no side effects.
class PowerPC_VSX_WWW_Intrinsic<string GCCIntSuffix>
````
- **L365 EN**: Banner comment marking a file or section boundary.
  **L365 CN**: 横幅注释，用于标记文件或章节边界。
- **L366 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC VSX Intrinsic Class Definitions.`.
  **L366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC VSX Intrinsic Class Definitions.`。
- **L367 EN**: Separator comment used for visual grouping.
  **L367 CN**: 用于视觉分组的分隔注释。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L369 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_VSX_Vec_DDD_Intrinsic - A PowerPC intrinsic that takes two v2f64`.
  **L369 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_VSX_Vec_DDD_Intrinsic - A PowerPC intrinsic that takes two v2f64`。
- **L370 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L370 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L371 EN**: Declares class `PowerPC_VSX_Vec_DDD_Intrinsic<string`.
  **L371 CN**: 声明 class `PowerPC_VSX_Vec_DDD_Intrinsic<string`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_VSX_Intrinsic<GCCIntSuffix,`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_VSX_Intrinsic<GCCIntSuffix,`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty],`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty], [llvm_v2f64_ty, llvm_v2f64_ty],`。
- **L374 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L374 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L376 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_VSX_Vec_FFF_Intrinsic - A PowerPC intrinsic that takes two v4f32`.
  **L376 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_VSX_Vec_FFF_Intrinsic - A PowerPC intrinsic that takes two v4f32`。
- **L377 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L377 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L378 EN**: Declares class `PowerPC_VSX_Vec_FFF_Intrinsic<string`.
  **L378 CN**: 声明 class `PowerPC_VSX_Vec_FFF_Intrinsic<string`。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_VSX_Intrinsic<GCCIntSuffix,`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_VSX_Intrinsic<GCCIntSuffix,`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L381 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L381 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_VSX_Sca_DDD_Intrinsic - A PowerPC intrinsic that takes two f64`.
  **L383 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_VSX_Sca_DDD_Intrinsic - A PowerPC intrinsic that takes two f64`。
- **L384 EN**: Comment explains nearby logic, invariants, or intent: `scalars and returns one.  These intrinsics have no side effects.`.
  **L384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scalars and returns one.  These intrinsics have no side effects.`。
- **L385 EN**: Declares class `PowerPC_VSX_Sca_DDD_Intrinsic<string`.
  **L385 CN**: 声明 class `PowerPC_VSX_Sca_DDD_Intrinsic<string`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_VSX_Intrinsic<GCCIntSuffix,`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_VSX_Intrinsic<GCCIntSuffix,`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty], [llvm_double_ty, llvm_double_ty],`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty], [llvm_double_ty, llvm_double_ty],`。
- **L388 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L388 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L390 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC_VSX_WWW_Intrinsic - A PowerPC intrinsic that takes two v4i32`.
  **L390 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC_VSX_WWW_Intrinsic - A PowerPC intrinsic that takes two v4i32`。
- **L391 EN**: Comment explains nearby logic, invariants, or intent: `vectors and returns one.  These intrinsics have no side effects.`.
  **L391 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vectors and returns one.  These intrinsics have no side effects.`。
- **L392 EN**: Declares class `PowerPC_VSX_WWW_Intrinsic<string`.
  **L392 CN**: 声明 class `PowerPC_VSX_WWW_Intrinsic<string`。

### Lines 393-420

````tablegen
  : PowerPC_VSX_Intrinsic<GCCIntSuffix,
                          [llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                          [IntrNoMem]>;
//===----------------------------------------------------------------------===//
// PowerPC Altivec Intrinsic Definitions.

let TargetPrefix = "ppc" in {  // All intrinsics start with "llvm.ppc.".
  // Data Stream Control.
  def int_ppc_altivec_dss : ClangBuiltin<"__builtin_altivec_dss">,
              Intrinsic<[], [llvm_i32_ty], []>;
  def int_ppc_altivec_dssall : ClangBuiltin<"__builtin_altivec_dssall">,
              Intrinsic<[], [], []>;
  def int_ppc_altivec_dst : ClangBuiltin<"__builtin_altivec_dst">,
              Intrinsic<[],
                        [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
                        []>;
  def int_ppc_altivec_dstt : ClangBuiltin<"__builtin_altivec_dstt">,
              Intrinsic<[],
                        [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
                        []>;
  def int_ppc_altivec_dstst : ClangBuiltin<"__builtin_altivec_dstst">,
              Intrinsic<[],
                        [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
                        []>;
  def int_ppc_altivec_dststt : ClangBuiltin<"__builtin_altivec_dststt">,
              Intrinsic<[],
                        [llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],
                        []>;
````
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: PowerPC_VSX_Intrinsic<GCCIntSuffix,`.
  **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`: PowerPC_VSX_Intrinsic<GCCIntSuffix,`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L395 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L395 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L396 EN**: Banner comment marking a file or section boundary.
  **L396 CN**: 横幅注释，用于标记文件或章节边界。
- **L397 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC Altivec Intrinsic Definitions.`.
  **L397 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC Altivec Intrinsic Definitions.`。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L399 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L399 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L400 EN**: Comment explains nearby logic, invariants, or intent: `Data Stream Control.`.
  **L400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Data Stream Control.`。
- **L401 EN**: Declares TableGen def `int_ppc_altivec_dss`.
  **L401 CN**: 声明 TableGen def `int_ppc_altivec_dss`。
- **L402 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], []>;`.
  **L402 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], []>;`。
- **L403 EN**: Declares TableGen def `int_ppc_altivec_dssall`.
  **L403 CN**: 声明 TableGen def `int_ppc_altivec_dssall`。
- **L404 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], []>;`.
  **L404 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], []>;`。
- **L405 EN**: Declares TableGen def `int_ppc_altivec_dst`.
  **L405 CN**: 声明 TableGen def `int_ppc_altivec_dst`。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L408 EN**: Executes a standalone statement or declaration: `[]>;`.
  **L408 CN**: 执行一条独立语句或声明：`[]>;`。
- **L409 EN**: Declares TableGen def `int_ppc_altivec_dstt`.
  **L409 CN**: 声明 TableGen def `int_ppc_altivec_dstt`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L412 EN**: Executes a standalone statement or declaration: `[]>;`.
  **L412 CN**: 执行一条独立语句或声明：`[]>;`。
- **L413 EN**: Declares TableGen def `int_ppc_altivec_dstst`.
  **L413 CN**: 声明 TableGen def `int_ppc_altivec_dstst`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L416 EN**: Executes a standalone statement or declaration: `[]>;`.
  **L416 CN**: 执行一条独立语句或声明：`[]>;`。
- **L417 EN**: Declares TableGen def `int_ppc_altivec_dststt`.
  **L417 CN**: 声明 TableGen def `int_ppc_altivec_dststt`。
- **L418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L418 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L420 EN**: Executes a standalone statement or declaration: `[]>;`.
  **L420 CN**: 执行一条独立语句或声明：`[]>;`。

### Lines 421-448

````tablegen

  // VSCR access.
  def int_ppc_altivec_mfvscr : ClangBuiltin<"__builtin_altivec_mfvscr">,
              Intrinsic<[llvm_v8i16_ty], [], [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_mtvscr : ClangBuiltin<"__builtin_altivec_mtvscr">,
              Intrinsic<[], [llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;


  // Loads.  These don't map directly to GCC builtins because they represent the
  // source address with a single pointer.
  def int_ppc_altivec_lvx :
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],
                            [IntrReadMem, IntrArgMemOnly]>;
  def int_ppc_altivec_lvxl :
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],
                            [IntrReadMem, IntrArgMemOnly]>;
  def int_ppc_altivec_lvebx :
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty],
                            [IntrReadMem, IntrArgMemOnly]>;
  def int_ppc_altivec_lvehx :
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_ptr_ty],
                            [IntrReadMem, IntrArgMemOnly]>;
  def int_ppc_altivec_lvewx :
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],
                            [IntrReadMem, IntrArgMemOnly]>;

  // Stores.  These don't map directly to GCC builtins because they represent the
  // source address with a single pointer.
````
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L422 EN**: Comment explains nearby logic, invariants, or intent: `VSCR access.`.
  **L422 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VSCR access.`。
- **L423 EN**: Declares TableGen def `int_ppc_altivec_mfvscr`.
  **L423 CN**: 声明 TableGen def `int_ppc_altivec_mfvscr`。
- **L424 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_v8i16_ty], [], [IntrNoMem, IntrHasSideEffects]>;`.
  **L424 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_v8i16_ty], [], [IntrNoMem, IntrHasSideEffects]>;`。
- **L425 EN**: Declares TableGen def `int_ppc_altivec_mtvscr`.
  **L425 CN**: 声明 TableGen def `int_ppc_altivec_mtvscr`。
- **L426 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;`.
  **L426 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;`。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L428 EN**: Blank line separating nearby declarations or logic blocks.
  **L428 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L429 EN**: Comment explains nearby logic, invariants, or intent: `Loads.  These don't map directly to GCC builtins because they represent the`.
  **L429 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Loads.  These don't map directly to GCC builtins because they represent the`。
- **L430 EN**: Comment explains nearby logic, invariants, or intent: `source address with a single pointer.`.
  **L430 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`source address with a single pointer.`。
- **L431 EN**: Declares TableGen def `int_ppc_altivec_lvx`.
  **L431 CN**: 声明 TableGen def `int_ppc_altivec_lvx`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`。
- **L433 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L433 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L434 EN**: Declares TableGen def `int_ppc_altivec_lvxl`.
  **L434 CN**: 声明 TableGen def `int_ppc_altivec_lvxl`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`.
  **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`。
- **L436 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L436 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L437 EN**: Declares TableGen def `int_ppc_altivec_lvebx`.
  **L437 CN**: 声明 TableGen def `int_ppc_altivec_lvebx`。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty],`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty],`。
- **L439 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L439 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L440 EN**: Declares TableGen def `int_ppc_altivec_lvehx`.
  **L440 CN**: 声明 TableGen def `int_ppc_altivec_lvehx`。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_ptr_ty],`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_ptr_ty],`。
- **L442 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L442 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L443 EN**: Declares TableGen def `int_ppc_altivec_lvewx`.
  **L443 CN**: 声明 TableGen def `int_ppc_altivec_lvewx`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`。
- **L445 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L445 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L447 EN**: Comment explains nearby logic, invariants, or intent: `Stores.  These don't map directly to GCC builtins because they represent the`.
  **L447 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Stores.  These don't map directly to GCC builtins because they represent the`。
- **L448 EN**: Comment explains nearby logic, invariants, or intent: `source address with a single pointer.`.
  **L448 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`source address with a single pointer.`。

### Lines 449-476

````tablegen
  def int_ppc_altivec_stvx :
              DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],
                                    [IntrWriteMem, IntrArgMemOnly]>;
  def int_ppc_altivec_stvxl :
              DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],
                                    [IntrWriteMem, IntrArgMemOnly]>;
  def int_ppc_altivec_stvebx :
              DefaultAttrsIntrinsic<[], [llvm_v16i8_ty, llvm_ptr_ty],
                                    [IntrWriteMem, IntrArgMemOnly]>;
  def int_ppc_altivec_stvehx :
              DefaultAttrsIntrinsic<[], [llvm_v8i16_ty, llvm_ptr_ty],
                                    [IntrWriteMem, IntrArgMemOnly]>;
  def int_ppc_altivec_stvewx :
              DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],
                                    [IntrWriteMem, IntrArgMemOnly]>;

  // Comparisons setting a vector.
  def int_ppc_altivec_vcmpbfp : ClangBuiltin<"__builtin_altivec_vcmpbfp">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpeqfp : ClangBuiltin<"__builtin_altivec_vcmpeqfp">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgefp : ClangBuiltin<"__builtin_altivec_vcmpgefp">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtfp : ClangBuiltin<"__builtin_altivec_vcmpgtfp">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],
````
- **L449 EN**: Declares TableGen def `int_ppc_altivec_stvx`.
  **L449 CN**: 声明 TableGen def `int_ppc_altivec_stvx`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],`。
- **L451 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L451 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L452 EN**: Declares TableGen def `int_ppc_altivec_stvxl`.
  **L452 CN**: 声明 TableGen def `int_ppc_altivec_stvxl`。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],`。
- **L454 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L454 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L455 EN**: Declares TableGen def `int_ppc_altivec_stvebx`.
  **L455 CN**: 声明 TableGen def `int_ppc_altivec_stvebx`。
- **L456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_v16i8_ty, llvm_ptr_ty],`.
  **L456 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_v16i8_ty, llvm_ptr_ty],`。
- **L457 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L457 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L458 EN**: Declares TableGen def `int_ppc_altivec_stvehx`.
  **L458 CN**: 声明 TableGen def `int_ppc_altivec_stvehx`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_v8i16_ty, llvm_ptr_ty],`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_v8i16_ty, llvm_ptr_ty],`。
- **L460 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L460 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L461 EN**: Declares TableGen def `int_ppc_altivec_stvewx`.
  **L461 CN**: 声明 TableGen def `int_ppc_altivec_stvewx`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],`。
- **L463 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L463 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L465 EN**: Comment explains nearby logic, invariants, or intent: `Comparisons setting a vector.`.
  **L465 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Comparisons setting a vector.`。
- **L466 EN**: Declares TableGen def `int_ppc_altivec_vcmpbfp`.
  **L466 CN**: 声明 TableGen def `int_ppc_altivec_vcmpbfp`。
- **L467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L467 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L468 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L468 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L469 EN**: Declares TableGen def `int_ppc_altivec_vcmpeqfp`.
  **L469 CN**: 声明 TableGen def `int_ppc_altivec_vcmpeqfp`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L471 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L471 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L472 EN**: Declares TableGen def `int_ppc_altivec_vcmpgefp`.
  **L472 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgefp`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`。
- **L474 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L474 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L475 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtfp`.
  **L475 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtfp`。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_v4f32_ty],`。

### Lines 477-504

````tablegen
                            [IntrNoMem]>;

  def int_ppc_altivec_vcmpequd : ClangBuiltin<"__builtin_altivec_vcmpequd">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsd : ClangBuiltin<"__builtin_altivec_vcmpgtsd">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtud : ClangBuiltin<"__builtin_altivec_vcmpgtud">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;

  def int_ppc_altivec_vcmpequw : ClangBuiltin<"__builtin_altivec_vcmpequw">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsw : ClangBuiltin<"__builtin_altivec_vcmpgtsw">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtuw : ClangBuiltin<"__builtin_altivec_vcmpgtuw">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpnew : ClangBuiltin<"__builtin_altivec_vcmpnew">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpnezw : ClangBuiltin<"__builtin_altivec_vcmpnezw">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;

````
- **L477 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L477 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L478 EN**: Blank line separating nearby declarations or logic blocks.
  **L478 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L479 EN**: Declares TableGen def `int_ppc_altivec_vcmpequd`.
  **L479 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequd`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L481 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L481 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L482 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsd`.
  **L482 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsd`。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L484 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L484 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L485 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtud`.
  **L485 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtud`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L487 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L487 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L489 EN**: Declares TableGen def `int_ppc_altivec_vcmpequw`.
  **L489 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequw`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L491 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L491 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L492 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsw`.
  **L492 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsw`。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L494 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L494 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L495 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtuw`.
  **L495 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtuw`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L497 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L497 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L498 EN**: Declares TableGen def `int_ppc_altivec_vcmpnew`.
  **L498 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnew`。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L500 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L500 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L501 EN**: Declares TableGen def `int_ppc_altivec_vcmpnezw`.
  **L501 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnezw`。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L503 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L503 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 505-532

````tablegen
  def int_ppc_altivec_vcmpequh : ClangBuiltin<"__builtin_altivec_vcmpequh">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsh : ClangBuiltin<"__builtin_altivec_vcmpgtsh">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtuh : ClangBuiltin<"__builtin_altivec_vcmpgtuh">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpneh : ClangBuiltin<"__builtin_altivec_vcmpneh">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpnezh : ClangBuiltin<"__builtin_altivec_vcmpnezh">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;

  def int_ppc_altivec_vcmpequb : ClangBuiltin<"__builtin_altivec_vcmpequb">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsb : ClangBuiltin<"__builtin_altivec_vcmpgtsb">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtub : ClangBuiltin<"__builtin_altivec_vcmpgtub">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpneb : ClangBuiltin<"__builtin_altivec_vcmpneb">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
````
- **L505 EN**: Declares TableGen def `int_ppc_altivec_vcmpequh`.
  **L505 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequh`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L507 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L507 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L508 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsh`.
  **L508 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsh`。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L510 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L510 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L511 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtuh`.
  **L511 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtuh`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L513 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L513 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L514 EN**: Declares TableGen def `int_ppc_altivec_vcmpneh`.
  **L514 CN**: 声明 TableGen def `int_ppc_altivec_vcmpneh`。
- **L515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L515 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L516 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L516 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L517 EN**: Declares TableGen def `int_ppc_altivec_vcmpnezh`.
  **L517 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnezh`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L519 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L519 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L521 EN**: Declares TableGen def `int_ppc_altivec_vcmpequb`.
  **L521 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequb`。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L523 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L523 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L524 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsb`.
  **L524 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsb`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L526 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L526 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L527 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtub`.
  **L527 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtub`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L529 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L529 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L530 EN**: Declares TableGen def `int_ppc_altivec_vcmpneb`.
  **L530 CN**: 声明 TableGen def `int_ppc_altivec_vcmpneb`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L532 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L532 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 533-560

````tablegen
  def int_ppc_altivec_vcmpnezb : ClangBuiltin<"__builtin_altivec_vcmpnezb">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;

  def int_ppc_altivec_vcmpequq : ClangBuiltin<"__builtin_altivec_vcmpequq">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsq : ClangBuiltin<"__builtin_altivec_vcmpgtsq">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtuq : ClangBuiltin<"__builtin_altivec_vcmpgtuq">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpequq_p : ClangBuiltin<"__builtin_altivec_vcmpequq_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsq_p : ClangBuiltin<"__builtin_altivec_vcmpgtsq_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtuq_p : ClangBuiltin<"__builtin_altivec_vcmpgtuq_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],
                            [IntrNoMem]>;

  // Predicate Comparisons.  The first operand specifies interpretation of CR6.
  def int_ppc_altivec_vcmpbfp_p : ClangBuiltin<"__builtin_altivec_vcmpbfp_p">,
````
- **L533 EN**: Declares TableGen def `int_ppc_altivec_vcmpnezb`.
  **L533 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnezb`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L535 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L535 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L537 EN**: Declares TableGen def `int_ppc_altivec_vcmpequq`.
  **L537 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequq`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L539 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L539 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L540 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsq`.
  **L540 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsq`。
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L542 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L542 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L543 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtuq`.
  **L543 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtuq`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L545 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L545 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L546 EN**: Declares TableGen def `int_ppc_altivec_vcmpequq_p`.
  **L546 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequq_p`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],`。
- **L549 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L549 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L550 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsq_p`.
  **L550 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsq_p`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],`.
  **L552 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],`。
- **L553 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L553 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L554 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtuq_p`.
  **L554 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtuq_p`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v1i128_ty,llvm_v1i128_ty],`。
- **L557 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L557 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L558 EN**: Blank line separating nearby declarations or logic blocks.
  **L558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L559 EN**: Comment explains nearby logic, invariants, or intent: `Predicate Comparisons.  The first operand specifies interpretation of CR6.`.
  **L559 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Predicate Comparisons.  The first operand specifies interpretation of CR6.`。
- **L560 EN**: Declares TableGen def `int_ppc_altivec_vcmpbfp_p`.
  **L560 CN**: 声明 TableGen def `int_ppc_altivec_vcmpbfp_p`。

### Lines 561-588

````tablegen
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpeqfp_p : ClangBuiltin<"__builtin_altivec_vcmpeqfp_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgefp_p : ClangBuiltin<"__builtin_altivec_vcmpgefp_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtfp_p : ClangBuiltin<"__builtin_altivec_vcmpgtfp_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],
                            [IntrNoMem]>;

  def int_ppc_altivec_vcmpequd_p : ClangBuiltin<"__builtin_altivec_vcmpequd_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsd_p : ClangBuiltin<"__builtin_altivec_vcmpgtsd_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtud_p : ClangBuiltin<"__builtin_altivec_vcmpgtud_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],
                            [IntrNoMem]>;
````
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`.
  **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`。
- **L563 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L563 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L564 EN**: Declares TableGen def `int_ppc_altivec_vcmpeqfp_p`.
  **L564 CN**: 声明 TableGen def `int_ppc_altivec_vcmpeqfp_p`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`。
- **L567 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L567 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L568 EN**: Declares TableGen def `int_ppc_altivec_vcmpgefp_p`.
  **L568 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgefp_p`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`。
- **L571 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L571 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L572 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtfp_p`.
  **L572 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtfp_p`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`.
  **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`。
- **L575 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L575 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L577 EN**: Declares TableGen def `int_ppc_altivec_vcmpequd_p`.
  **L577 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequd_p`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],`。
- **L580 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L580 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L581 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsd_p`.
  **L581 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsd_p`。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],`.
  **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],`。
- **L584 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L584 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L585 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtud_p`.
  **L585 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtud_p`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v2i64_ty,llvm_v2i64_ty],`。
- **L588 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L588 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 589-616

````tablegen

  def int_ppc_altivec_vcmpequw_p : ClangBuiltin<"__builtin_altivec_vcmpequw_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsw_p : ClangBuiltin<"__builtin_altivec_vcmpgtsw_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtuw_p : ClangBuiltin<"__builtin_altivec_vcmpgtuw_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpnew_p : ClangBuiltin<"__builtin_altivec_vcmpnew_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpnezw_p : ClangBuiltin<"__builtin_altivec_vcmpnezw_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],
                            [IntrNoMem]>;

  def int_ppc_altivec_vcmpequh_p : ClangBuiltin<"__builtin_altivec_vcmpequh_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsh_p : ClangBuiltin<"__builtin_altivec_vcmpgtsh_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
````
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L590 EN**: Declares TableGen def `int_ppc_altivec_vcmpequw_p`.
  **L590 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequw_p`。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`。
- **L593 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L593 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L594 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsw_p`.
  **L594 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsw_p`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`。
- **L597 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L597 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L598 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtuw_p`.
  **L598 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtuw_p`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`。
- **L601 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L601 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L602 EN**: Declares TableGen def `int_ppc_altivec_vcmpnew_p`.
  **L602 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnew_p`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`。
- **L605 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L605 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L606 EN**: Declares TableGen def `int_ppc_altivec_vcmpnezw_p`.
  **L606 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnezw_p`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4i32_ty,llvm_v4i32_ty],`。
- **L609 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L609 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L611 EN**: Declares TableGen def `int_ppc_altivec_vcmpequh_p`.
  **L611 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequh_p`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`。
- **L614 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L614 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L615 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsh_p`.
  **L615 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsh_p`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。

### Lines 617-644

````tablegen
                            [llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtuh_p : ClangBuiltin<"__builtin_altivec_vcmpgtuh_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpneh_p : ClangBuiltin<"__builtin_altivec_vcmpneh_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpnezh_p : ClangBuiltin<"__builtin_altivec_vcmpnezh_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],
                            [IntrNoMem]>;

  def int_ppc_altivec_vcmpequb_p : ClangBuiltin<"__builtin_altivec_vcmpequb_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtsb_p : ClangBuiltin<"__builtin_altivec_vcmpgtsb_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpgtub_p : ClangBuiltin<"__builtin_altivec_vcmpgtub_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpneb_p : ClangBuiltin<"__builtin_altivec_vcmpneb_p">,
````
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`.
  **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`。
- **L618 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L618 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L619 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtuh_p`.
  **L619 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtuh_p`。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`。
- **L622 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L622 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L623 EN**: Declares TableGen def `int_ppc_altivec_vcmpneh_p`.
  **L623 CN**: 声明 TableGen def `int_ppc_altivec_vcmpneh_p`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`。
- **L626 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L626 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L627 EN**: Declares TableGen def `int_ppc_altivec_vcmpnezh_p`.
  **L627 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnezh_p`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v8i16_ty,llvm_v8i16_ty],`。
- **L630 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L630 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L631 EN**: Blank line separating nearby declarations or logic blocks.
  **L631 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L632 EN**: Declares TableGen def `int_ppc_altivec_vcmpequb_p`.
  **L632 CN**: 声明 TableGen def `int_ppc_altivec_vcmpequb_p`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`。
- **L635 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L635 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L636 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtsb_p`.
  **L636 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtsb_p`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`。
- **L639 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L639 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L640 EN**: Declares TableGen def `int_ppc_altivec_vcmpgtub_p`.
  **L640 CN**: 声明 TableGen def `int_ppc_altivec_vcmpgtub_p`。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`。
- **L643 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L643 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L644 EN**: Declares TableGen def `int_ppc_altivec_vcmpneb_p`.
  **L644 CN**: 声明 TableGen def `int_ppc_altivec_vcmpneb_p`。

### Lines 645-672

````tablegen
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vcmpnezb_p : ClangBuiltin<"__builtin_altivec_vcmpnezb_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vclzlsbb : ClangBuiltin<"__builtin_altivec_vclzlsbb">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_v16i8_ty],[IntrNoMem]>;
  def int_ppc_altivec_vctzlsbb : ClangBuiltin<"__builtin_altivec_vctzlsbb">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_v16i8_ty],[IntrNoMem]>;
  def int_ppc_altivec_vprtybw : ClangBuiltin<"__builtin_altivec_vprtybw">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty],[llvm_v4i32_ty],[IntrNoMem]>;
  def int_ppc_altivec_vprtybd : ClangBuiltin<"__builtin_altivec_vprtybd">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],[llvm_v2i64_ty],[IntrNoMem]>;
  def int_ppc_altivec_vprtybq : ClangBuiltin<"__builtin_altivec_vprtybq">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty],[llvm_v1i128_ty],[IntrNoMem]>;

  // BCD intrinsics.
  def int_ppc_national2packed: ClangBuiltin<"__builtin_ppc_national2packed">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_packed2national: ClangBuiltin<"__builtin_ppc_packed2national">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_packed2zoned: ClangBuiltin<"__builtin_ppc_packed2zoned">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_zoned2packed: ClangBuiltin<"__builtin_ppc_zoned2packed">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_cdtbcdd : ClangBuiltin<"__builtin_ppc_cdtbcd">,
````
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`。
- **L647 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L647 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L648 EN**: Declares TableGen def `int_ppc_altivec_vcmpnezb_p`.
  **L648 CN**: 声明 TableGen def `int_ppc_altivec_vcmpnezb_p`。
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v16i8_ty,llvm_v16i8_ty],`。
- **L651 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L651 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L652 EN**: Declares TableGen def `int_ppc_altivec_vclzlsbb`.
  **L652 CN**: 声明 TableGen def `int_ppc_altivec_vclzlsbb`。
- **L653 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_v16i8_ty],[IntrNoMem]>;`.
  **L653 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_v16i8_ty],[IntrNoMem]>;`。
- **L654 EN**: Declares TableGen def `int_ppc_altivec_vctzlsbb`.
  **L654 CN**: 声明 TableGen def `int_ppc_altivec_vctzlsbb`。
- **L655 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_v16i8_ty],[IntrNoMem]>;`.
  **L655 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_v16i8_ty],[IntrNoMem]>;`。
- **L656 EN**: Declares TableGen def `int_ppc_altivec_vprtybw`.
  **L656 CN**: 声明 TableGen def `int_ppc_altivec_vprtybw`。
- **L657 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],[llvm_v4i32_ty],[IntrNoMem]>;`.
  **L657 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],[llvm_v4i32_ty],[IntrNoMem]>;`。
- **L658 EN**: Declares TableGen def `int_ppc_altivec_vprtybd`.
  **L658 CN**: 声明 TableGen def `int_ppc_altivec_vprtybd`。
- **L659 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],[llvm_v2i64_ty],[IntrNoMem]>;`.
  **L659 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],[llvm_v2i64_ty],[IntrNoMem]>;`。
- **L660 EN**: Declares TableGen def `int_ppc_altivec_vprtybq`.
  **L660 CN**: 声明 TableGen def `int_ppc_altivec_vprtybq`。
- **L661 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v1i128_ty],[llvm_v1i128_ty],[IntrNoMem]>;`.
  **L661 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v1i128_ty],[llvm_v1i128_ty],[IntrNoMem]>;`。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L663 EN**: Comment explains nearby logic, invariants, or intent: `BCD intrinsics.`.
  **L663 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BCD intrinsics.`。
- **L664 EN**: Declares TableGen def `int_ppc_national2packed`.
  **L664 CN**: 声明 TableGen def `int_ppc_national2packed`。
- **L665 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L665 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L666 EN**: Declares TableGen def `int_ppc_packed2national`.
  **L666 CN**: 声明 TableGen def `int_ppc_packed2national`。
- **L667 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty], [IntrNoMem]>;`.
  **L667 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty], [IntrNoMem]>;`。
- **L668 EN**: Declares TableGen def `int_ppc_packed2zoned`.
  **L668 CN**: 声明 TableGen def `int_ppc_packed2zoned`。
- **L669 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L669 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L670 EN**: Declares TableGen def `int_ppc_zoned2packed`.
  **L670 CN**: 声明 TableGen def `int_ppc_zoned2packed`。
- **L671 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L671 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L672 EN**: Declares TableGen def `int_ppc_cdtbcdd`.
  **L672 CN**: 声明 TableGen def `int_ppc_cdtbcdd`。

### Lines 673-700

````tablegen
    DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_cbcdtdd: ClangBuiltin<"__builtin_ppc_cbcdtd">,
    DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_addg6sd: ClangBuiltin<"__builtin_ppc_addg6s">,
    DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_cdtbcd : ClangBuiltin<"__builtin_cdtbcd">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;
  def int_ppc_cbcdtd: ClangBuiltin<"__builtin_cbcdtd">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;
  def int_ppc_addg6s: ClangBuiltin<"__builtin_addg6s">,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;

  // BCD Format conversion intrinsics 
  def int_ppc_bcdcopysign : ClangBuiltin<"__builtin_ppc_bcdcopysign">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>; 
  def int_ppc_bcdsetsign : ClangBuiltin<"__builtin_ppc_bcdsetsign">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty], 
    [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  def int_ppc_bcdshift : ClangBuiltin<"__builtin_ppc_bcdshift">,
		DefaultAttrsIntrinsic<
		[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],
		[IntrNoMem, ImmArg<ArgIndex<2>>]>;

	def int_ppc_bcdshiftround : ClangBuiltin<"__builtin_ppc_bcdshiftround">,
		DefaultAttrsIntrinsic<
    [llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],
		[IntrNoMem, ImmArg<ArgIndex<2>>]>;
````
- **L673 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L673 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L674 EN**: Declares TableGen def `int_ppc_cbcdtdd`.
  **L674 CN**: 声明 TableGen def `int_ppc_cbcdtdd`。
- **L675 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L675 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L676 EN**: Declares TableGen def `int_ppc_addg6sd`.
  **L676 CN**: 声明 TableGen def `int_ppc_addg6sd`。
- **L677 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty], [IntrNoMem]>;`.
  **L677 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty], [IntrNoMem]>;`。
- **L678 EN**: Declares TableGen def `int_ppc_cdtbcd`.
  **L678 CN**: 声明 TableGen def `int_ppc_cdtbcd`。
- **L679 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L679 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L680 EN**: Declares TableGen def `int_ppc_cbcdtd`.
  **L680 CN**: 声明 TableGen def `int_ppc_cbcdtd`。
- **L681 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`.
  **L681 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty], [IntrNoMem]>;`。
- **L682 EN**: Declares TableGen def `int_ppc_addg6s`.
  **L682 CN**: 声明 TableGen def `int_ppc_addg6s`。
- **L683 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L683 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L685 EN**: Comment explains nearby logic, invariants, or intent: `BCD Format conversion intrinsics`.
  **L685 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BCD Format conversion intrinsics`。
- **L686 EN**: Declares TableGen def `int_ppc_bcdcopysign`.
  **L686 CN**: 声明 TableGen def `int_ppc_bcdcopysign`。
- **L687 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L687 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L688 EN**: Declares TableGen def `int_ppc_bcdsetsign`.
  **L688 CN**: 声明 TableGen def `int_ppc_bcdsetsign`。
- **L689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`.
  **L689 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`。
- **L690 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L690 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L692 EN**: Declares TableGen def `int_ppc_bcdshift`.
  **L692 CN**: 声明 TableGen def `int_ppc_bcdshift`。
- **L693 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L693 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L695 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L695 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L696 EN**: Blank line separating nearby declarations or logic blocks.
  **L696 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L697 EN**: Declares TableGen def `int_ppc_bcdshiftround`.
  **L697 CN**: 声明 TableGen def `int_ppc_bcdshiftround`。
- **L698 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L698 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L700 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L700 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。

### Lines 701-728

````tablegen

	def int_ppc_bcdtruncate : ClangBuiltin<"__builtin_ppc_bcdtruncate">,
		DefaultAttrsIntrinsic<
		[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],
		[IntrNoMem, ImmArg<ArgIndex<2>>]>;

	def int_ppc_bcdunsignedtruncate : ClangBuiltin<"__builtin_ppc_bcdunsignedtruncate">,
		DefaultAttrsIntrinsic<
		[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],
		[IntrNoMem]>;

	def int_ppc_bcdunsignedshift : ClangBuiltin<"__builtin_ppc_bcdunsignedshift">,
		DefaultAttrsIntrinsic<
		[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],
		[IntrNoMem]>;

  def int_ppc_bcdadd : ClangBuiltin<"__builtin_ppc_bcdadd">,
    DefaultAttrsIntrinsic<
    [llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
    [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_ppc_bcdadd_p : ClangBuiltin<"__builtin_ppc_bcdadd_p">,
    DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty, llvm_v16i8_ty],
    [IntrNoMem, ImmArg<ArgIndex<0>>]>;
  def int_ppc_bcdsub : ClangBuiltin<"__builtin_ppc_bcdsub">,
    DefaultAttrsIntrinsic<
    [llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
    [IntrNoMem, ImmArg<ArgIndex<2>>]>;
````
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L702 EN**: Declares TableGen def `int_ppc_bcdtruncate`.
  **L702 CN**: 声明 TableGen def `int_ppc_bcdtruncate`。
- **L703 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L703 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L705 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L705 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L706 EN**: Blank line separating nearby declarations or logic blocks.
  **L706 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L707 EN**: Declares TableGen def `int_ppc_bcdunsignedtruncate`.
  **L707 CN**: 声明 TableGen def `int_ppc_bcdunsignedtruncate`。
- **L708 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L708 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`。
- **L710 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L710 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L712 EN**: Declares TableGen def `int_ppc_bcdunsignedshift`.
  **L712 CN**: 声明 TableGen def `int_ppc_bcdunsignedshift`。
- **L713 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L713 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`.
  **L714 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`。
- **L715 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L715 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L716 EN**: Blank line separating nearby declarations or logic blocks.
  **L716 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L717 EN**: Declares TableGen def `int_ppc_bcdadd`.
  **L717 CN**: 声明 TableGen def `int_ppc_bcdadd`。
- **L718 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L718 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L720 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L720 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L721 EN**: Declares TableGen def `int_ppc_bcdadd_p`.
  **L721 CN**: 声明 TableGen def `int_ppc_bcdadd_p`。
- **L722 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L722 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L724 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L724 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L725 EN**: Declares TableGen def `int_ppc_bcdsub`.
  **L725 CN**: 声明 TableGen def `int_ppc_bcdsub`。
- **L726 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L726 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L728 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L728 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。

### Lines 729-756

````tablegen
  def int_ppc_bcdsub_p : ClangBuiltin<"__builtin_ppc_bcdsub_p">,
    DefaultAttrsIntrinsic<
    [llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty, llvm_v16i8_ty],
    [IntrNoMem, ImmArg<ArgIndex<0>>]>;

  // P10 Vector Extract with Mask
  def int_ppc_altivec_vextractbm : ClangBuiltin<"__builtin_altivec_vextractbm">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextracthm : ClangBuiltin<"__builtin_altivec_vextracthm">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextractwm : ClangBuiltin<"__builtin_altivec_vextractwm">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextractdm : ClangBuiltin<"__builtin_altivec_vextractdm">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextractqm : ClangBuiltin<"__builtin_altivec_vextractqm">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v1i128_ty], [IntrNoMem]>;

  // P10 Vector Expand with Mask
  def int_ppc_altivec_vexpandbm : ClangBuiltin<"__builtin_altivec_vexpandbm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vexpandhm : ClangBuiltin<"__builtin_altivec_vexpandhm">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vexpandwm : ClangBuiltin<"__builtin_altivec_vexpandwm">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vexpanddm : ClangBuiltin<"__builtin_altivec_vexpanddm">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;
  def int_ppc_altivec_vexpandqm : ClangBuiltin<"__builtin_altivec_vexpandqm">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty], [IntrNoMem]>;
````
- **L729 EN**: Declares TableGen def `int_ppc_bcdsub_p`.
  **L729 CN**: 声明 TableGen def `int_ppc_bcdsub_p`。
- **L730 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsic<`.
  **L730 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsic<`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L732 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L732 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L734 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Extract with Mask`.
  **L734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Extract with Mask`。
- **L735 EN**: Declares TableGen def `int_ppc_altivec_vextractbm`.
  **L735 CN**: 声明 TableGen def `int_ppc_altivec_vextractbm`。
- **L736 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L736 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L737 EN**: Declares TableGen def `int_ppc_altivec_vextracthm`.
  **L737 CN**: 声明 TableGen def `int_ppc_altivec_vextracthm`。
- **L738 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L738 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L739 EN**: Declares TableGen def `int_ppc_altivec_vextractwm`.
  **L739 CN**: 声明 TableGen def `int_ppc_altivec_vextractwm`。
- **L740 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L740 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L741 EN**: Declares TableGen def `int_ppc_altivec_vextractdm`.
  **L741 CN**: 声明 TableGen def `int_ppc_altivec_vextractdm`。
- **L742 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L742 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L743 EN**: Declares TableGen def `int_ppc_altivec_vextractqm`.
  **L743 CN**: 声明 TableGen def `int_ppc_altivec_vextractqm`。
- **L744 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v1i128_ty], [IntrNoMem]>;`.
  **L744 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_v1i128_ty], [IntrNoMem]>;`。
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L746 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Expand with Mask`.
  **L746 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Expand with Mask`。
- **L747 EN**: Declares TableGen def `int_ppc_altivec_vexpandbm`.
  **L747 CN**: 声明 TableGen def `int_ppc_altivec_vexpandbm`。
- **L748 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L748 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L749 EN**: Declares TableGen def `int_ppc_altivec_vexpandhm`.
  **L749 CN**: 声明 TableGen def `int_ppc_altivec_vexpandhm`。
- **L750 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L750 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L751 EN**: Declares TableGen def `int_ppc_altivec_vexpandwm`.
  **L751 CN**: 声明 TableGen def `int_ppc_altivec_vexpandwm`。
- **L752 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L752 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L753 EN**: Declares TableGen def `int_ppc_altivec_vexpanddm`.
  **L753 CN**: 声明 TableGen def `int_ppc_altivec_vexpanddm`。
- **L754 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L754 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L755 EN**: Declares TableGen def `int_ppc_altivec_vexpandqm`.
  **L755 CN**: 声明 TableGen def `int_ppc_altivec_vexpandqm`。
- **L756 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty], [IntrNoMem]>;`.
  **L756 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v1i128_ty], [IntrNoMem]>;`。

### Lines 757-784

````tablegen

  // P10 Vector Count with Mask intrinsics.
  def int_ppc_altivec_vcntmbb : ClangBuiltin<"__builtin_altivec_vcntmbb">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_altivec_vcntmbh : ClangBuiltin<"__builtin_altivec_vcntmbh">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v8i16_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_altivec_vcntmbw : ClangBuiltin<"__builtin_altivec_vcntmbw">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v4i32_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_altivec_vcntmbd : ClangBuiltin<"__builtin_altivec_vcntmbd">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  // P10 Move to VSR with Mask Intrinsics.
  def int_ppc_altivec_mtvsrbm : ClangBuiltin<"__builtin_altivec_mtvsrbm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_altivec_mtvsrhm : ClangBuiltin<"__builtin_altivec_mtvsrhm">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_altivec_mtvsrwm : ClangBuiltin<"__builtin_altivec_mtvsrwm">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_altivec_mtvsrdm : ClangBuiltin<"__builtin_altivec_mtvsrdm">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_altivec_mtvsrqm : ClangBuiltin<"__builtin_altivec_mtvsrqm">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_i64_ty], [IntrNoMem]>;

  // P10 Vector Parallel Bits Deposit/Extract Doubleword Builtins.
````
- **L757 EN**: Blank line separating nearby declarations or logic blocks.
  **L757 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L758 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Count with Mask intrinsics.`.
  **L758 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Count with Mask intrinsics.`。
- **L759 EN**: Declares TableGen def `int_ppc_altivec_vcntmbb`.
  **L759 CN**: 声明 TableGen def `int_ppc_altivec_vcntmbb`。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v16i8_ty, llvm_i32_ty],`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v16i8_ty, llvm_i32_ty],`。
- **L761 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L761 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L762 EN**: Declares TableGen def `int_ppc_altivec_vcntmbh`.
  **L762 CN**: 声明 TableGen def `int_ppc_altivec_vcntmbh`。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v8i16_ty, llvm_i32_ty],`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v8i16_ty, llvm_i32_ty],`。
- **L764 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L764 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L765 EN**: Declares TableGen def `int_ppc_altivec_vcntmbw`.
  **L765 CN**: 声明 TableGen def `int_ppc_altivec_vcntmbw`。
- **L766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v4i32_ty, llvm_i32_ty],`.
  **L766 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v4i32_ty, llvm_i32_ty],`。
- **L767 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L767 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L768 EN**: Declares TableGen def `int_ppc_altivec_vcntmbd`.
  **L768 CN**: 声明 TableGen def `int_ppc_altivec_vcntmbd`。
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty],`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v2i64_ty, llvm_i32_ty],`。
- **L770 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L770 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L772 EN**: Comment explains nearby logic, invariants, or intent: `P10 Move to VSR with Mask Intrinsics.`.
  **L772 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Move to VSR with Mask Intrinsics.`。
- **L773 EN**: Declares TableGen def `int_ppc_altivec_mtvsrbm`.
  **L773 CN**: 声明 TableGen def `int_ppc_altivec_mtvsrbm`。
- **L774 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L774 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L775 EN**: Declares TableGen def `int_ppc_altivec_mtvsrhm`.
  **L775 CN**: 声明 TableGen def `int_ppc_altivec_mtvsrhm`。
- **L776 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L776 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L777 EN**: Declares TableGen def `int_ppc_altivec_mtvsrwm`.
  **L777 CN**: 声明 TableGen def `int_ppc_altivec_mtvsrwm`。
- **L778 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L778 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L779 EN**: Declares TableGen def `int_ppc_altivec_mtvsrdm`.
  **L779 CN**: 声明 TableGen def `int_ppc_altivec_mtvsrdm`。
- **L780 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L780 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L781 EN**: Declares TableGen def `int_ppc_altivec_mtvsrqm`.
  **L781 CN**: 声明 TableGen def `int_ppc_altivec_mtvsrqm`。
- **L782 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L782 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L784 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Parallel Bits Deposit/Extract Doubleword Builtins.`.
  **L784 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Parallel Bits Deposit/Extract Doubleword Builtins.`。

### Lines 785-812

````tablegen
  def int_ppc_altivec_vpdepd : ClangBuiltin<"__builtin_altivec_vpdepd">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vpextd : ClangBuiltin<"__builtin_altivec_vpextd">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;

  // P10 Vector String Isolate Intrinsics.
  def int_ppc_altivec_vstribr : ClangBuiltin<"__builtin_altivec_vstribr">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vstribl : ClangBuiltin<"__builtin_altivec_vstribl">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vstrihr : ClangBuiltin<"__builtin_altivec_vstrihr">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vstrihl : ClangBuiltin<"__builtin_altivec_vstrihl">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  // Predicate Intrinsics: The first operand specifies interpretation of CR6.
  def int_ppc_altivec_vstribr_p : ClangBuiltin<"__builtin_altivec_vstribr_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vstribl_p : ClangBuiltin<"__builtin_altivec_vstribl_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vstrihr_p : ClangBuiltin<"__builtin_altivec_vstrihr_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vstrihl_p : ClangBuiltin<"__builtin_altivec_vstrihl_p">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v8i16_ty],
````
- **L785 EN**: Declares TableGen def `int_ppc_altivec_vpdepd`.
  **L785 CN**: 声明 TableGen def `int_ppc_altivec_vpdepd`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L787 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L787 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L788 EN**: Declares TableGen def `int_ppc_altivec_vpextd`.
  **L788 CN**: 声明 TableGen def `int_ppc_altivec_vpextd`。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L790 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L790 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L791 EN**: Blank line separating nearby declarations or logic blocks.
  **L791 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L792 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector String Isolate Intrinsics.`.
  **L792 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector String Isolate Intrinsics.`。
- **L793 EN**: Declares TableGen def `int_ppc_altivec_vstribr`.
  **L793 CN**: 声明 TableGen def `int_ppc_altivec_vstribr`。
- **L794 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L794 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L795 EN**: Declares TableGen def `int_ppc_altivec_vstribl`.
  **L795 CN**: 声明 TableGen def `int_ppc_altivec_vstribl`。
- **L796 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L796 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L797 EN**: Declares TableGen def `int_ppc_altivec_vstrihr`.
  **L797 CN**: 声明 TableGen def `int_ppc_altivec_vstrihr`。
- **L798 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L798 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L799 EN**: Declares TableGen def `int_ppc_altivec_vstrihl`.
  **L799 CN**: 声明 TableGen def `int_ppc_altivec_vstrihl`。
- **L800 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L800 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L801 EN**: Comment explains nearby logic, invariants, or intent: `Predicate Intrinsics: The first operand specifies interpretation of CR6.`.
  **L801 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Predicate Intrinsics: The first operand specifies interpretation of CR6.`。
- **L802 EN**: Declares TableGen def `int_ppc_altivec_vstribr_p`.
  **L802 CN**: 声明 TableGen def `int_ppc_altivec_vstribr_p`。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty],`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty],`。
- **L804 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L804 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L805 EN**: Declares TableGen def `int_ppc_altivec_vstribl_p`.
  **L805 CN**: 声明 TableGen def `int_ppc_altivec_vstribl_p`。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty],`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v16i8_ty],`。
- **L807 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L807 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L808 EN**: Declares TableGen def `int_ppc_altivec_vstrihr_p`.
  **L808 CN**: 声明 TableGen def `int_ppc_altivec_vstrihr_p`。
- **L809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v8i16_ty],`.
  **L809 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v8i16_ty],`。
- **L810 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L810 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L811 EN**: Declares TableGen def `int_ppc_altivec_vstrihl_p`.
  **L811 CN**: 声明 TableGen def `int_ppc_altivec_vstrihl_p`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v8i16_ty],`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_v8i16_ty],`。

### Lines 813-840

````tablegen
                            [IntrNoMem]>;

  // P10 Vector Centrifuge Builtin.
  def int_ppc_altivec_vcfuged : ClangBuiltin<"__builtin_altivec_vcfuged">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;

  // P10 Vector Gather Every Nth Bit Builtin.
  def int_ppc_altivec_vgnb : ClangBuiltin<"__builtin_altivec_vgnb">,
      DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v1i128_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

   // P10 Vector Clear Bytes
   def int_ppc_altivec_vclrlb :  ClangBuiltin<"__builtin_altivec_vclrlb">,
       DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],
                             [IntrNoMem]>;
   def int_ppc_altivec_vclrrb :  ClangBuiltin<"__builtin_altivec_vclrrb">,
       DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],
                             [IntrNoMem]>;

  // P10 Vector Shift Double Bit Immediate.
  def int_ppc_altivec_vsldbi : ClangBuiltin<"__builtin_altivec_vsldbi">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_ppc_altivec_vsrdbi : ClangBuiltin<"__builtin_altivec_vsrdbi">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
````
- **L813 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L813 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L815 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Centrifuge Builtin.`.
  **L815 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Centrifuge Builtin.`。
- **L816 EN**: Declares TableGen def `int_ppc_altivec_vcfuged`.
  **L816 CN**: 声明 TableGen def `int_ppc_altivec_vcfuged`。
- **L817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L817 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L818 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L818 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L820 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Gather Every Nth Bit Builtin.`.
  **L820 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Gather Every Nth Bit Builtin.`。
- **L821 EN**: Declares TableGen def `int_ppc_altivec_vgnb`.
  **L821 CN**: 声明 TableGen def `int_ppc_altivec_vgnb`。
- **L822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v1i128_ty, llvm_i32_ty],`.
  **L822 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_v1i128_ty, llvm_i32_ty],`。
- **L823 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L823 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L825 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Clear Bytes`.
  **L825 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Clear Bytes`。
- **L826 EN**: Declares TableGen def `int_ppc_altivec_vclrlb`.
  **L826 CN**: 声明 TableGen def `int_ppc_altivec_vclrlb`。
- **L827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`.
  **L827 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`。
- **L828 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L828 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L829 EN**: Declares TableGen def `int_ppc_altivec_vclrrb`.
  **L829 CN**: 声明 TableGen def `int_ppc_altivec_vclrrb`。
- **L830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`.
  **L830 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_i32_ty],`。
- **L831 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L831 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L833 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Shift Double Bit Immediate.`.
  **L833 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Shift Double Bit Immediate.`。
- **L834 EN**: Declares TableGen def `int_ppc_altivec_vsldbi`.
  **L834 CN**: 声明 TableGen def `int_ppc_altivec_vsldbi`。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L836 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L837 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L837 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L838 EN**: Declares TableGen def `int_ppc_altivec_vsrdbi`.
  **L838 CN**: 声明 TableGen def `int_ppc_altivec_vsrdbi`。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L840 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。

### Lines 841-868

````tablegen
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  // P10 Vector Insert.
  def int_ppc_altivec_vinsblx : ClangBuiltin<"__builtin_altivec_vinsblx">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinsbrx : ClangBuiltin<"__builtin_altivec_vinsbrx">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinshlx : ClangBuiltin<"__builtin_altivec_vinshlx">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty],
                            [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinshrx : ClangBuiltin<"__builtin_altivec_vinshrx">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty],
                            [llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinswlx : ClangBuiltin<"__builtin_altivec_vinswlx">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinswrx : ClangBuiltin<"__builtin_altivec_vinswrx">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinsdlx : ClangBuiltin<"__builtin_altivec_vinsdlx">,
````
- **L841 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L841 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L843 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Insert.`.
  **L843 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Insert.`。
- **L844 EN**: Declares TableGen def `int_ppc_altivec_vinsblx`.
  **L844 CN**: 声明 TableGen def `int_ppc_altivec_vinsblx`。
- **L845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L845 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L846 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L847 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L847 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L848 EN**: Declares TableGen def `int_ppc_altivec_vinsbrx`.
  **L848 CN**: 声明 TableGen def `int_ppc_altivec_vinsbrx`。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L850 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L851 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L851 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L852 EN**: Declares TableGen def `int_ppc_altivec_vinshlx`.
  **L852 CN**: 声明 TableGen def `int_ppc_altivec_vinshlx`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty],`.
  **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty],`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L855 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L855 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L856 EN**: Declares TableGen def `int_ppc_altivec_vinshrx`.
  **L856 CN**: 声明 TableGen def `int_ppc_altivec_vinshrx`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty],`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty],`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L859 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L859 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L860 EN**: Declares TableGen def `int_ppc_altivec_vinswlx`.
  **L860 CN**: 声明 TableGen def `int_ppc_altivec_vinswlx`。
- **L861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],`.
  **L861 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L863 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L863 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L864 EN**: Declares TableGen def `int_ppc_altivec_vinswrx`.
  **L864 CN**: 声明 TableGen def `int_ppc_altivec_vinswrx`。
- **L865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],`.
  **L865 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L867 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L867 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L868 EN**: Declares TableGen def `int_ppc_altivec_vinsdlx`.
  **L868 CN**: 声明 TableGen def `int_ppc_altivec_vinsdlx`。

### Lines 869-896

````tablegen
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_i64_ty, llvm_i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinsdrx : ClangBuiltin<"__builtin_altivec_vinsdrx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_i64_ty, llvm_i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinsbvlx : ClangBuiltin<"__builtin_altivec_vinsbvlx">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinsbvrx : ClangBuiltin<"__builtin_altivec_vinsbvrx">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinshvlx : ClangBuiltin<"__builtin_altivec_vinshvlx">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty],
                            [llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
 def int_ppc_altivec_vinshvrx : ClangBuiltin<"__builtin_altivec_vinshvrx">,
     DefaultAttrsIntrinsic<[llvm_v8i16_ty],
                            [llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinswvlx : ClangBuiltin<"__builtin_altivec_vinswvlx">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vinswvrx : ClangBuiltin<"__builtin_altivec_vinswvrx">,
````
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_i64_ty, llvm_i64_ty],`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_i64_ty, llvm_i64_ty],`。
- **L871 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L871 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L872 EN**: Declares TableGen def `int_ppc_altivec_vinsdrx`.
  **L872 CN**: 声明 TableGen def `int_ppc_altivec_vinsdrx`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_i64_ty, llvm_i64_ty],`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_i64_ty, llvm_i64_ty],`。
- **L875 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L875 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L876 EN**: Declares TableGen def `int_ppc_altivec_vinsbvlx`.
  **L876 CN**: 声明 TableGen def `int_ppc_altivec_vinsbvlx`。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],`.
  **L878 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],`。
- **L879 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L879 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L880 EN**: Declares TableGen def `int_ppc_altivec_vinsbvrx`.
  **L880 CN**: 声明 TableGen def `int_ppc_altivec_vinsbvrx`。
- **L881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L881 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty, llvm_v16i8_ty],`。
- **L883 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L883 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L884 EN**: Declares TableGen def `int_ppc_altivec_vinshvlx`.
  **L884 CN**: 声明 TableGen def `int_ppc_altivec_vinshvlx`。
- **L885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty],`.
  **L885 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty],`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],`。
- **L887 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L887 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L888 EN**: Declares TableGen def `int_ppc_altivec_vinshvrx`.
  **L888 CN**: 声明 TableGen def `int_ppc_altivec_vinshvrx`。
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty],`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty],`。
- **L890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],`.
  **L890 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty, llvm_v8i16_ty],`。
- **L891 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L891 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L892 EN**: Declares TableGen def `int_ppc_altivec_vinswvlx`.
  **L892 CN**: 声明 TableGen def `int_ppc_altivec_vinswvlx`。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],`。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`。
- **L895 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L895 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L896 EN**: Declares TableGen def `int_ppc_altivec_vinswvrx`.
  **L896 CN**: 声明 TableGen def `int_ppc_altivec_vinswvrx`。

### Lines 897-924

````tablegen
      DefaultAttrsIntrinsic<[llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  // P10 Vector Insert with immediate.
  def int_ppc_altivec_vinsw :
      DefaultAttrsIntrinsic<[llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_ppc_altivec_vinsd :
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_i64_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  // P10 Vector Extract.
  def int_ppc_altivec_vextdubvlx : ClangBuiltin<"__builtin_altivec_vextdubvlx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vextdubvrx : ClangBuiltin<"__builtin_altivec_vextdubvrx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vextduhvlx : ClangBuiltin<"__builtin_altivec_vextduhvlx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vextduhvrx : ClangBuiltin<"__builtin_altivec_vextduhvrx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],
````
- **L897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],`.
  **L897 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty, llvm_v4i32_ty],`。
- **L899 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L899 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L900 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Insert with immediate.`.
  **L900 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Insert with immediate.`。
- **L901 EN**: Declares TableGen def `int_ppc_altivec_vinsw`.
  **L901 CN**: 声明 TableGen def `int_ppc_altivec_vinsw`。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],`.
  **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],`。
- **L903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L903 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L904 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L904 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L905 EN**: Declares TableGen def `int_ppc_altivec_vinsd`.
  **L905 CN**: 声明 TableGen def `int_ppc_altivec_vinsd`。
- **L906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L906 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_i64_ty, llvm_i32_ty],`.
  **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_i64_ty, llvm_i32_ty],`。
- **L908 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L908 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L909 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Extract.`.
  **L909 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Extract.`。
- **L910 EN**: Declares TableGen def `int_ppc_altivec_vextdubvlx`.
  **L910 CN**: 声明 TableGen def `int_ppc_altivec_vextdubvlx`。
- **L911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L911 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L913 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L913 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L914 EN**: Declares TableGen def `int_ppc_altivec_vextdubvrx`.
  **L914 CN**: 声明 TableGen def `int_ppc_altivec_vextdubvrx`。
- **L915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L915 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L916 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L917 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L917 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L918 EN**: Declares TableGen def `int_ppc_altivec_vextduhvlx`.
  **L918 CN**: 声明 TableGen def `int_ppc_altivec_vextduhvlx`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`。
- **L921 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L921 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L922 EN**: Declares TableGen def `int_ppc_altivec_vextduhvrx`.
  **L922 CN**: 声明 TableGen def `int_ppc_altivec_vextduhvrx`。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`.
  **L924 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty, llvm_i32_ty],`。

### Lines 925-952

````tablegen
                            [IntrNoMem]>;
  def int_ppc_altivec_vextduwvlx : ClangBuiltin<"__builtin_altivec_vextduwvlx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vextduwvrx : ClangBuiltin<"__builtin_altivec_vextduwvrx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vextddvlx : ClangBuiltin<"__builtin_altivec_vextddvlx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vextddvrx : ClangBuiltin<"__builtin_altivec_vextddvrx">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],
                            [IntrNoMem]>;
}

// Vector average.
def int_ppc_altivec_vavgsb : PowerPC_Vec_BBB_Intrinsic<"vavgsb">;
def int_ppc_altivec_vavgsh : PowerPC_Vec_HHH_Intrinsic<"vavgsh">;
def int_ppc_altivec_vavgsw : PowerPC_Vec_WWW_Intrinsic<"vavgsw">;
def int_ppc_altivec_vavgub : PowerPC_Vec_BBB_Intrinsic<"vavgub">;
def int_ppc_altivec_vavguh : PowerPC_Vec_HHH_Intrinsic<"vavguh">;
def int_ppc_altivec_vavguw : PowerPC_Vec_WWW_Intrinsic<"vavguw">;

// Vector maximum.
````
- **L925 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L925 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L926 EN**: Declares TableGen def `int_ppc_altivec_vextduwvlx`.
  **L926 CN**: 声明 TableGen def `int_ppc_altivec_vextduwvlx`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`。
- **L929 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L929 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L930 EN**: Declares TableGen def `int_ppc_altivec_vextduwvrx`.
  **L930 CN**: 声明 TableGen def `int_ppc_altivec_vextduwvrx`。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty, llvm_i32_ty],`。
- **L933 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L933 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L934 EN**: Declares TableGen def `int_ppc_altivec_vextddvlx`.
  **L934 CN**: 声明 TableGen def `int_ppc_altivec_vextddvlx`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`。
- **L937 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L937 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L938 EN**: Declares TableGen def `int_ppc_altivec_vextddvrx`.
  **L938 CN**: 声明 TableGen def `int_ppc_altivec_vextddvrx`。
- **L939 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L939 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`.
  **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`。
- **L941 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L941 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L942 EN**: Closes the current lexical scope or compound statement.
  **L942 CN**: 结束当前词法作用域或复合语句块。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L944 EN**: Comment explains nearby logic, invariants, or intent: `Vector average.`.
  **L944 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector average.`。
- **L945 EN**: Declares TableGen def `int_ppc_altivec_vavgsb`.
  **L945 CN**: 声明 TableGen def `int_ppc_altivec_vavgsb`。
- **L946 EN**: Declares TableGen def `int_ppc_altivec_vavgsh`.
  **L946 CN**: 声明 TableGen def `int_ppc_altivec_vavgsh`。
- **L947 EN**: Declares TableGen def `int_ppc_altivec_vavgsw`.
  **L947 CN**: 声明 TableGen def `int_ppc_altivec_vavgsw`。
- **L948 EN**: Declares TableGen def `int_ppc_altivec_vavgub`.
  **L948 CN**: 声明 TableGen def `int_ppc_altivec_vavgub`。
- **L949 EN**: Declares TableGen def `int_ppc_altivec_vavguh`.
  **L949 CN**: 声明 TableGen def `int_ppc_altivec_vavguh`。
- **L950 EN**: Declares TableGen def `int_ppc_altivec_vavguw`.
  **L950 CN**: 声明 TableGen def `int_ppc_altivec_vavguw`。
- **L951 EN**: Blank line separating nearby declarations or logic blocks.
  **L951 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L952 EN**: Comment explains nearby logic, invariants, or intent: `Vector maximum.`.
  **L952 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector maximum.`。

### Lines 953-980

````tablegen
def int_ppc_altivec_vmaxfp : PowerPC_Vec_FFF_Intrinsic<"vmaxfp">;
def int_ppc_altivec_vmaxsb : PowerPC_Vec_BBB_Intrinsic<"vmaxsb">;
def int_ppc_altivec_vmaxsh : PowerPC_Vec_HHH_Intrinsic<"vmaxsh">;
def int_ppc_altivec_vmaxsw : PowerPC_Vec_WWW_Intrinsic<"vmaxsw">;
def int_ppc_altivec_vmaxsd : PowerPC_Vec_DDD_Intrinsic<"vmaxsd">;
def int_ppc_altivec_vmaxub : PowerPC_Vec_BBB_Intrinsic<"vmaxub">;
def int_ppc_altivec_vmaxuh : PowerPC_Vec_HHH_Intrinsic<"vmaxuh">;
def int_ppc_altivec_vmaxuw : PowerPC_Vec_WWW_Intrinsic<"vmaxuw">;
def int_ppc_altivec_vmaxud : PowerPC_Vec_DDD_Intrinsic<"vmaxud">;

// Vector minimum.
def int_ppc_altivec_vminfp : PowerPC_Vec_FFF_Intrinsic<"vminfp">;
def int_ppc_altivec_vminsb : PowerPC_Vec_BBB_Intrinsic<"vminsb">;
def int_ppc_altivec_vminsh : PowerPC_Vec_HHH_Intrinsic<"vminsh">;
def int_ppc_altivec_vminsw : PowerPC_Vec_WWW_Intrinsic<"vminsw">;
def int_ppc_altivec_vminsd : PowerPC_Vec_DDD_Intrinsic<"vminsd">;
def int_ppc_altivec_vminub : PowerPC_Vec_BBB_Intrinsic<"vminub">;
def int_ppc_altivec_vminuh : PowerPC_Vec_HHH_Intrinsic<"vminuh">;
def int_ppc_altivec_vminuw : PowerPC_Vec_WWW_Intrinsic<"vminuw">;
def int_ppc_altivec_vminud : PowerPC_Vec_DDD_Intrinsic<"vminud">;

// Saturating adds.
def int_ppc_altivec_vaddubs : PowerPC_Vec_BBB_Intrinsic<"vaddubs">;
def int_ppc_altivec_vaddsbs : PowerPC_Vec_BBB_Intrinsic<"vaddsbs">;
def int_ppc_altivec_vadduhs : PowerPC_Vec_HHH_Intrinsic<"vadduhs">;
def int_ppc_altivec_vaddshs : PowerPC_Vec_HHH_Intrinsic<"vaddshs">;
def int_ppc_altivec_vadduws : PowerPC_Vec_WWW_Intrinsic<"vadduws">;
def int_ppc_altivec_vaddsws : PowerPC_Vec_WWW_Intrinsic<"vaddsws">;
````
- **L953 EN**: Declares TableGen def `int_ppc_altivec_vmaxfp`.
  **L953 CN**: 声明 TableGen def `int_ppc_altivec_vmaxfp`。
- **L954 EN**: Declares TableGen def `int_ppc_altivec_vmaxsb`.
  **L954 CN**: 声明 TableGen def `int_ppc_altivec_vmaxsb`。
- **L955 EN**: Declares TableGen def `int_ppc_altivec_vmaxsh`.
  **L955 CN**: 声明 TableGen def `int_ppc_altivec_vmaxsh`。
- **L956 EN**: Declares TableGen def `int_ppc_altivec_vmaxsw`.
  **L956 CN**: 声明 TableGen def `int_ppc_altivec_vmaxsw`。
- **L957 EN**: Declares TableGen def `int_ppc_altivec_vmaxsd`.
  **L957 CN**: 声明 TableGen def `int_ppc_altivec_vmaxsd`。
- **L958 EN**: Declares TableGen def `int_ppc_altivec_vmaxub`.
  **L958 CN**: 声明 TableGen def `int_ppc_altivec_vmaxub`。
- **L959 EN**: Declares TableGen def `int_ppc_altivec_vmaxuh`.
  **L959 CN**: 声明 TableGen def `int_ppc_altivec_vmaxuh`。
- **L960 EN**: Declares TableGen def `int_ppc_altivec_vmaxuw`.
  **L960 CN**: 声明 TableGen def `int_ppc_altivec_vmaxuw`。
- **L961 EN**: Declares TableGen def `int_ppc_altivec_vmaxud`.
  **L961 CN**: 声明 TableGen def `int_ppc_altivec_vmaxud`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L963 EN**: Comment explains nearby logic, invariants, or intent: `Vector minimum.`.
  **L963 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector minimum.`。
- **L964 EN**: Declares TableGen def `int_ppc_altivec_vminfp`.
  **L964 CN**: 声明 TableGen def `int_ppc_altivec_vminfp`。
- **L965 EN**: Declares TableGen def `int_ppc_altivec_vminsb`.
  **L965 CN**: 声明 TableGen def `int_ppc_altivec_vminsb`。
- **L966 EN**: Declares TableGen def `int_ppc_altivec_vminsh`.
  **L966 CN**: 声明 TableGen def `int_ppc_altivec_vminsh`。
- **L967 EN**: Declares TableGen def `int_ppc_altivec_vminsw`.
  **L967 CN**: 声明 TableGen def `int_ppc_altivec_vminsw`。
- **L968 EN**: Declares TableGen def `int_ppc_altivec_vminsd`.
  **L968 CN**: 声明 TableGen def `int_ppc_altivec_vminsd`。
- **L969 EN**: Declares TableGen def `int_ppc_altivec_vminub`.
  **L969 CN**: 声明 TableGen def `int_ppc_altivec_vminub`。
- **L970 EN**: Declares TableGen def `int_ppc_altivec_vminuh`.
  **L970 CN**: 声明 TableGen def `int_ppc_altivec_vminuh`。
- **L971 EN**: Declares TableGen def `int_ppc_altivec_vminuw`.
  **L971 CN**: 声明 TableGen def `int_ppc_altivec_vminuw`。
- **L972 EN**: Declares TableGen def `int_ppc_altivec_vminud`.
  **L972 CN**: 声明 TableGen def `int_ppc_altivec_vminud`。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L974 EN**: Comment explains nearby logic, invariants, or intent: `Saturating adds.`.
  **L974 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating adds.`。
- **L975 EN**: Declares TableGen def `int_ppc_altivec_vaddubs`.
  **L975 CN**: 声明 TableGen def `int_ppc_altivec_vaddubs`。
- **L976 EN**: Declares TableGen def `int_ppc_altivec_vaddsbs`.
  **L976 CN**: 声明 TableGen def `int_ppc_altivec_vaddsbs`。
- **L977 EN**: Declares TableGen def `int_ppc_altivec_vadduhs`.
  **L977 CN**: 声明 TableGen def `int_ppc_altivec_vadduhs`。
- **L978 EN**: Declares TableGen def `int_ppc_altivec_vaddshs`.
  **L978 CN**: 声明 TableGen def `int_ppc_altivec_vaddshs`。
- **L979 EN**: Declares TableGen def `int_ppc_altivec_vadduws`.
  **L979 CN**: 声明 TableGen def `int_ppc_altivec_vadduws`。
- **L980 EN**: Declares TableGen def `int_ppc_altivec_vaddsws`.
  **L980 CN**: 声明 TableGen def `int_ppc_altivec_vaddsws`。

### Lines 981-1008

````tablegen
def int_ppc_altivec_vaddcuw : PowerPC_Vec_WWW_Intrinsic<"vaddcuw">;
def int_ppc_altivec_vaddcuq : PowerPC_Vec_QQQ_Intrinsic<"vaddcuq">;

// Saturating subs.
def int_ppc_altivec_vsububs : PowerPC_Vec_BBB_Intrinsic<"vsububs">;
def int_ppc_altivec_vsubsbs : PowerPC_Vec_BBB_Intrinsic<"vsubsbs">;
def int_ppc_altivec_vsubuhs : PowerPC_Vec_HHH_Intrinsic<"vsubuhs">;
def int_ppc_altivec_vsubshs : PowerPC_Vec_HHH_Intrinsic<"vsubshs">;
def int_ppc_altivec_vsubuws : PowerPC_Vec_WWW_Intrinsic<"vsubuws">;
def int_ppc_altivec_vsubsws : PowerPC_Vec_WWW_Intrinsic<"vsubsws">;
def int_ppc_altivec_vsubcuw : PowerPC_Vec_WWW_Intrinsic<"vsubcuw">;
def int_ppc_altivec_vsubcuq : PowerPC_Vec_QQQ_Intrinsic<"vsubcuq">;

let TargetPrefix = "ppc" in {  // All PPC intrinsics start with "llvm.ppc.".
  // Saturating multiply-adds.
  def int_ppc_altivec_vmhaddshs : ClangBuiltin<"__builtin_altivec_vmhaddshs">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty,
                             llvm_v8i16_ty, llvm_v8i16_ty],
                             [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vmhraddshs : ClangBuiltin<"__builtin_altivec_vmhraddshs">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty,
                             llvm_v8i16_ty, llvm_v8i16_ty],
                             [IntrNoMem, IntrHasSideEffects]>;

  def int_ppc_altivec_vmaddfp : ClangBuiltin<"__builtin_altivec_vmaddfp">,
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty,
                             llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vnmsubfp : ClangBuiltin<"__builtin_altivec_vnmsubfp">,
````
- **L981 EN**: Declares TableGen def `int_ppc_altivec_vaddcuw`.
  **L981 CN**: 声明 TableGen def `int_ppc_altivec_vaddcuw`。
- **L982 EN**: Declares TableGen def `int_ppc_altivec_vaddcuq`.
  **L982 CN**: 声明 TableGen def `int_ppc_altivec_vaddcuq`。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L984 EN**: Comment explains nearby logic, invariants, or intent: `Saturating subs.`.
  **L984 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating subs.`。
- **L985 EN**: Declares TableGen def `int_ppc_altivec_vsububs`.
  **L985 CN**: 声明 TableGen def `int_ppc_altivec_vsububs`。
- **L986 EN**: Declares TableGen def `int_ppc_altivec_vsubsbs`.
  **L986 CN**: 声明 TableGen def `int_ppc_altivec_vsubsbs`。
- **L987 EN**: Declares TableGen def `int_ppc_altivec_vsubuhs`.
  **L987 CN**: 声明 TableGen def `int_ppc_altivec_vsubuhs`。
- **L988 EN**: Declares TableGen def `int_ppc_altivec_vsubshs`.
  **L988 CN**: 声明 TableGen def `int_ppc_altivec_vsubshs`。
- **L989 EN**: Declares TableGen def `int_ppc_altivec_vsubuws`.
  **L989 CN**: 声明 TableGen def `int_ppc_altivec_vsubuws`。
- **L990 EN**: Declares TableGen def `int_ppc_altivec_vsubsws`.
  **L990 CN**: 声明 TableGen def `int_ppc_altivec_vsubsws`。
- **L991 EN**: Declares TableGen def `int_ppc_altivec_vsubcuw`.
  **L991 CN**: 声明 TableGen def `int_ppc_altivec_vsubcuw`。
- **L992 EN**: Declares TableGen def `int_ppc_altivec_vsubcuq`.
  **L992 CN**: 声明 TableGen def `int_ppc_altivec_vsubcuq`。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L994 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L994 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L995 EN**: Comment explains nearby logic, invariants, or intent: `Saturating multiply-adds.`.
  **L995 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturating multiply-adds.`。
- **L996 EN**: Declares TableGen def `int_ppc_altivec_vmhaddshs`.
  **L996 CN**: 声明 TableGen def `int_ppc_altivec_vmhaddshs`。
- **L997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty,`.
  **L997 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty,`。
- **L998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L998 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L999 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L999 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1000 EN**: Declares TableGen def `int_ppc_altivec_vmhraddshs`.
  **L1000 CN**: 声明 TableGen def `int_ppc_altivec_vmhraddshs`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty,`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1003 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1003 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1005 EN**: Declares TableGen def `int_ppc_altivec_vmaddfp`.
  **L1005 CN**: 声明 TableGen def `int_ppc_altivec_vmaddfp`。
- **L1006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty,`.
  **L1006 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty,`。
- **L1007 EN**: Executes a standalone statement or declaration: `llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1007 CN**: 执行一条独立语句或声明：`llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1008 EN**: Declares TableGen def `int_ppc_altivec_vnmsubfp`.
  **L1008 CN**: 声明 TableGen def `int_ppc_altivec_vnmsubfp`。

### Lines 1009-1036

````tablegen
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty,
                             llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;

  // Vector Multiply Sum Instructions.
  def int_ppc_altivec_vmsummbm : ClangBuiltin<"__builtin_altivec_vmsummbm">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vmsumshm : ClangBuiltin<"__builtin_altivec_vmsumshm">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,
                             llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vmsumshs : ClangBuiltin<"__builtin_altivec_vmsumshs">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,
                             llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vmsumubm : ClangBuiltin<"__builtin_altivec_vmsumubm">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vmsumuhm : ClangBuiltin<"__builtin_altivec_vmsumuhm">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,
                             llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vmsumudm : ClangBuiltin<"__builtin_altivec_vmsumudm">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v2i64_ty, llvm_v2i64_ty,
                             llvm_v1i128_ty], [IntrNoMem]>;
  def int_ppc_altivec_vmsumuhs : ClangBuiltin<"__builtin_altivec_vmsumuhs">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,
                             llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vmsumcud : ClangBuiltin<"__builtin_altivec_vmsumcud">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty],
                            [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v1i128_ty],
````
- **L1009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty,`.
  **L1009 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty,`。
- **L1010 EN**: Executes a standalone statement or declaration: `llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1010 CN**: 执行一条独立语句或声明：`llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1012 EN**: Comment explains nearby logic, invariants, or intent: `Vector Multiply Sum Instructions.`.
  **L1012 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Multiply Sum Instructions.`。
- **L1013 EN**: Declares TableGen def `int_ppc_altivec_vmsummbm`.
  **L1013 CN**: 声明 TableGen def `int_ppc_altivec_vmsummbm`。
- **L1014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1014 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1015 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1015 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1016 EN**: Declares TableGen def `int_ppc_altivec_vmsumshm`.
  **L1016 CN**: 声明 TableGen def `int_ppc_altivec_vmsumshm`。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`.
  **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`。
- **L1018 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1018 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1019 EN**: Declares TableGen def `int_ppc_altivec_vmsumshs`.
  **L1019 CN**: 声明 TableGen def `int_ppc_altivec_vmsumshs`。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`.
  **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`。
- **L1021 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;`.
  **L1021 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;`。
- **L1022 EN**: Declares TableGen def `int_ppc_altivec_vmsumubm`.
  **L1022 CN**: 声明 TableGen def `int_ppc_altivec_vmsumubm`。
- **L1023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1023 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1024 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1024 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1025 EN**: Declares TableGen def `int_ppc_altivec_vmsumuhm`.
  **L1025 CN**: 声明 TableGen def `int_ppc_altivec_vmsumuhm`。
- **L1026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`.
  **L1026 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`。
- **L1027 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1027 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1028 EN**: Declares TableGen def `int_ppc_altivec_vmsumudm`.
  **L1028 CN**: 声明 TableGen def `int_ppc_altivec_vmsumudm`。
- **L1029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v2i64_ty, llvm_v2i64_ty,`.
  **L1029 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v2i64_ty, llvm_v2i64_ty,`。
- **L1030 EN**: Executes a standalone statement or declaration: `llvm_v1i128_ty], [IntrNoMem]>;`.
  **L1030 CN**: 执行一条独立语句或声明：`llvm_v1i128_ty], [IntrNoMem]>;`。
- **L1031 EN**: Declares TableGen def `int_ppc_altivec_vmsumuhs`.
  **L1031 CN**: 声明 TableGen def `int_ppc_altivec_vmsumuhs`。
- **L1032 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`.
  **L1032 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`。
- **L1033 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;`.
  **L1033 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty], [IntrNoMem, IntrHasSideEffects]>;`。
- **L1034 EN**: Declares TableGen def `int_ppc_altivec_vmsumcud`.
  **L1034 CN**: 声明 TableGen def `int_ppc_altivec_vmsumcud`。
- **L1035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty],`.
  **L1035 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty],`。
- **L1036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v1i128_ty],`.
  **L1036 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v1i128_ty],`。

### Lines 1037-1064

````tablegen
                            [IntrNoMem]>;

  // Vector Multiply Instructions.
  def int_ppc_altivec_vmulesb : ClangBuiltin<"__builtin_altivec_vmulesb">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulesh : ClangBuiltin<"__builtin_altivec_vmulesh">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulesw : ClangBuiltin<"__builtin_altivec_vmulesw">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulesd : PowerPC_Vec_QDD_Intrinsic<"vmulesd">;
  def int_ppc_altivec_vmuleub : ClangBuiltin<"__builtin_altivec_vmuleub">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmuleuh : ClangBuiltin<"__builtin_altivec_vmuleuh">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmuleuw : ClangBuiltin<"__builtin_altivec_vmuleuw">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmuleud : PowerPC_Vec_QDD_Intrinsic<"vmuleud">;

  def int_ppc_altivec_vmulosb : ClangBuiltin<"__builtin_altivec_vmulosb">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulosh : ClangBuiltin<"__builtin_altivec_vmulosh">,
````
- **L1037 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1037 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1039 EN**: Comment explains nearby logic, invariants, or intent: `Vector Multiply Instructions.`.
  **L1039 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Multiply Instructions.`。
- **L1040 EN**: Declares TableGen def `int_ppc_altivec_vmulesb`.
  **L1040 CN**: 声明 TableGen def `int_ppc_altivec_vmulesb`。
- **L1041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1041 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1042 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1042 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1043 EN**: Declares TableGen def `int_ppc_altivec_vmulesh`.
  **L1043 CN**: 声明 TableGen def `int_ppc_altivec_vmulesh`。
- **L1044 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1044 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1045 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1045 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1046 EN**: Declares TableGen def `int_ppc_altivec_vmulesw`.
  **L1046 CN**: 声明 TableGen def `int_ppc_altivec_vmulesw`。
- **L1047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1047 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1048 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1048 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1049 EN**: Declares TableGen def `int_ppc_altivec_vmulesd`.
  **L1049 CN**: 声明 TableGen def `int_ppc_altivec_vmulesd`。
- **L1050 EN**: Declares TableGen def `int_ppc_altivec_vmuleub`.
  **L1050 CN**: 声明 TableGen def `int_ppc_altivec_vmuleub`。
- **L1051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1051 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1052 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1052 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1053 EN**: Declares TableGen def `int_ppc_altivec_vmuleuh`.
  **L1053 CN**: 声明 TableGen def `int_ppc_altivec_vmuleuh`。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1055 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1055 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1056 EN**: Declares TableGen def `int_ppc_altivec_vmuleuw`.
  **L1056 CN**: 声明 TableGen def `int_ppc_altivec_vmuleuw`。
- **L1057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1057 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1058 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1058 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1059 EN**: Declares TableGen def `int_ppc_altivec_vmuleud`.
  **L1059 CN**: 声明 TableGen def `int_ppc_altivec_vmuleud`。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1061 EN**: Declares TableGen def `int_ppc_altivec_vmulosb`.
  **L1061 CN**: 声明 TableGen def `int_ppc_altivec_vmulosb`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1063 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1063 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1064 EN**: Declares TableGen def `int_ppc_altivec_vmulosh`.
  **L1064 CN**: 声明 TableGen def `int_ppc_altivec_vmulosh`。

### Lines 1065-1092

````tablegen
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulosw : ClangBuiltin<"__builtin_altivec_vmulosw">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulosd : PowerPC_Vec_QDD_Intrinsic<"vmulosd">;
  def int_ppc_altivec_vmuloub : ClangBuiltin<"__builtin_altivec_vmuloub">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulouh : ClangBuiltin<"__builtin_altivec_vmulouh">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmulouw : ClangBuiltin<"__builtin_altivec_vmulouw">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vmuloud : PowerPC_Vec_QDD_Intrinsic<"vmuloud">;

  // Vector Sum Instructions.
  def int_ppc_altivec_vsumsws : ClangBuiltin<"__builtin_altivec_vsumsws">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vsum2sws : ClangBuiltin<"__builtin_altivec_vsum2sws">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vsum4sbs : ClangBuiltin<"__builtin_altivec_vsum4sbs">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vsum4shs : ClangBuiltin<"__builtin_altivec_vsum4shs">,
````
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1066 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1066 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1067 EN**: Declares TableGen def `int_ppc_altivec_vmulosw`.
  **L1067 CN**: 声明 TableGen def `int_ppc_altivec_vmulosw`。
- **L1068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1068 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1069 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1069 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1070 EN**: Declares TableGen def `int_ppc_altivec_vmulosd`.
  **L1070 CN**: 声明 TableGen def `int_ppc_altivec_vmulosd`。
- **L1071 EN**: Declares TableGen def `int_ppc_altivec_vmuloub`.
  **L1071 CN**: 声明 TableGen def `int_ppc_altivec_vmuloub`。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1073 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1073 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1074 EN**: Declares TableGen def `int_ppc_altivec_vmulouh`.
  **L1074 CN**: 声明 TableGen def `int_ppc_altivec_vmulouh`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1076 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1076 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1077 EN**: Declares TableGen def `int_ppc_altivec_vmulouw`.
  **L1077 CN**: 声明 TableGen def `int_ppc_altivec_vmulouw`。
- **L1078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1078 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1079 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1079 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1080 EN**: Declares TableGen def `int_ppc_altivec_vmuloud`.
  **L1080 CN**: 声明 TableGen def `int_ppc_altivec_vmuloud`。
- **L1081 EN**: Blank line separating nearby declarations or logic blocks.
  **L1081 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1082 EN**: Comment explains nearby logic, invariants, or intent: `Vector Sum Instructions.`.
  **L1082 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Sum Instructions.`。
- **L1083 EN**: Declares TableGen def `int_ppc_altivec_vsumsws`.
  **L1083 CN**: 声明 TableGen def `int_ppc_altivec_vsumsws`。
- **L1084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1084 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1085 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1085 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1086 EN**: Declares TableGen def `int_ppc_altivec_vsum2sws`.
  **L1086 CN**: 声明 TableGen def `int_ppc_altivec_vsum2sws`。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1088 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1088 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1089 EN**: Declares TableGen def `int_ppc_altivec_vsum4sbs`.
  **L1089 CN**: 声明 TableGen def `int_ppc_altivec_vsum4sbs`。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v4i32_ty],`.
  **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v4i32_ty],`。
- **L1091 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1091 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1092 EN**: Declares TableGen def `int_ppc_altivec_vsum4shs`.
  **L1092 CN**: 声明 TableGen def `int_ppc_altivec_vsum4shs`。

### Lines 1093-1120

````tablegen
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vsum4ubs : ClangBuiltin<"__builtin_altivec_vsum4ubs">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;

  // Vector Sign Extension Instructions
  def int_ppc_altivec_vextsb2w : ClangBuiltin<"__builtin_altivec_vextsb2w">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextsb2d : ClangBuiltin<"__builtin_altivec_vextsb2d">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextsh2w : ClangBuiltin<"__builtin_altivec_vextsh2w">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextsh2d : ClangBuiltin<"__builtin_altivec_vextsh2d">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextsw2d : ClangBuiltin<"__builtin_altivec_vextsw2d">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vextsd2q : ClangBuiltin<"__builtin_altivec_vextsd2q">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v2i64_ty], [IntrNoMem]>;

  // Other multiplies.
  def int_ppc_altivec_vmladduhm : ClangBuiltin<"__builtin_altivec_vmladduhm">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty,
                             llvm_v8i16_ty], [IntrNoMem]>;

  // Packs.
  def int_ppc_altivec_vpkpx : ClangBuiltin<"__builtin_altivec_vpkpx">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
````
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v4i32_ty],`.
  **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty, llvm_v4i32_ty],`。
- **L1094 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1094 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1095 EN**: Declares TableGen def `int_ppc_altivec_vsum4ubs`.
  **L1095 CN**: 声明 TableGen def `int_ppc_altivec_vsum4ubs`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v4i32_ty],`.
  **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty, llvm_v4i32_ty],`。
- **L1097 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1097 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1098 EN**: Blank line separating nearby declarations or logic blocks.
  **L1098 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1099 EN**: Comment explains nearby logic, invariants, or intent: `Vector Sign Extension Instructions`.
  **L1099 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Sign Extension Instructions`。
- **L1100 EN**: Declares TableGen def `int_ppc_altivec_vextsb2w`.
  **L1100 CN**: 声明 TableGen def `int_ppc_altivec_vextsb2w`。
- **L1101 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1101 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1102 EN**: Declares TableGen def `int_ppc_altivec_vextsb2d`.
  **L1102 CN**: 声明 TableGen def `int_ppc_altivec_vextsb2d`。
- **L1103 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1103 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1104 EN**: Declares TableGen def `int_ppc_altivec_vextsh2w`.
  **L1104 CN**: 声明 TableGen def `int_ppc_altivec_vextsh2w`。
- **L1105 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1105 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1106 EN**: Declares TableGen def `int_ppc_altivec_vextsh2d`.
  **L1106 CN**: 声明 TableGen def `int_ppc_altivec_vextsh2d`。
- **L1107 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1107 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1108 EN**: Declares TableGen def `int_ppc_altivec_vextsw2d`.
  **L1108 CN**: 声明 TableGen def `int_ppc_altivec_vextsw2d`。
- **L1109 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1109 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1110 EN**: Declares TableGen def `int_ppc_altivec_vextsd2q`.
  **L1110 CN**: 声明 TableGen def `int_ppc_altivec_vextsd2q`。
- **L1111 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1111 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v1i128_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1112 EN**: Blank line separating nearby declarations or logic blocks.
  **L1112 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1113 EN**: Comment explains nearby logic, invariants, or intent: `Other multiplies.`.
  **L1113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Other multiplies.`。
- **L1114 EN**: Declares TableGen def `int_ppc_altivec_vmladduhm`.
  **L1114 CN**: 声明 TableGen def `int_ppc_altivec_vmladduhm`。
- **L1115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`.
  **L1115 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v8i16_ty, llvm_v8i16_ty,`。
- **L1116 EN**: Executes a standalone statement or declaration: `llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1116 CN**: 执行一条独立语句或声明：`llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1117 EN**: Blank line separating nearby declarations or logic blocks.
  **L1117 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1118 EN**: Comment explains nearby logic, invariants, or intent: `Packs.`.
  **L1118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Packs.`。
- **L1119 EN**: Declares TableGen def `int_ppc_altivec_vpkpx`.
  **L1119 CN**: 声明 TableGen def `int_ppc_altivec_vpkpx`。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。

### Lines 1121-1148

````tablegen
                            [IntrNoMem]>;
  def int_ppc_altivec_vpkshss : ClangBuiltin<"__builtin_altivec_vpkshss">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vpkshus : ClangBuiltin<"__builtin_altivec_vpkshus">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vpkswss : ClangBuiltin<"__builtin_altivec_vpkswss">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vpkswus : ClangBuiltin<"__builtin_altivec_vpkswus">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vpksdss : ClangBuiltin<"__builtin_altivec_vpksdss">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  def int_ppc_altivec_vpksdus : ClangBuiltin<"__builtin_altivec_vpksdus">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  // vpkuhum is lowered to a shuffle.
  def int_ppc_altivec_vpkuhus : ClangBuiltin<"__builtin_altivec_vpkuhus">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  // vpkuwum is lowered to a shuffle.
  def int_ppc_altivec_vpkuwus : ClangBuiltin<"__builtin_altivec_vpkuwus">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem, IntrHasSideEffects]>;
  // vpkudum is lowered to a shuffle.
````
- **L1121 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1121 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1122 EN**: Declares TableGen def `int_ppc_altivec_vpkshss`.
  **L1122 CN**: 声明 TableGen def `int_ppc_altivec_vpkshss`。
- **L1123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1123 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1124 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1124 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1125 EN**: Declares TableGen def `int_ppc_altivec_vpkshus`.
  **L1125 CN**: 声明 TableGen def `int_ppc_altivec_vpkshus`。
- **L1126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1126 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1127 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1127 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1128 EN**: Declares TableGen def `int_ppc_altivec_vpkswss`.
  **L1128 CN**: 声明 TableGen def `int_ppc_altivec_vpkswss`。
- **L1129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1129 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1130 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1130 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1131 EN**: Declares TableGen def `int_ppc_altivec_vpkswus`.
  **L1131 CN**: 声明 TableGen def `int_ppc_altivec_vpkswus`。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1133 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1133 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1134 EN**: Declares TableGen def `int_ppc_altivec_vpksdss`.
  **L1134 CN**: 声明 TableGen def `int_ppc_altivec_vpksdss`。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1136 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1136 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1137 EN**: Declares TableGen def `int_ppc_altivec_vpksdus`.
  **L1137 CN**: 声明 TableGen def `int_ppc_altivec_vpksdus`。
- **L1138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1138 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1139 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1139 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1140 EN**: Comment explains nearby logic, invariants, or intent: `vpkuhum is lowered to a shuffle.`.
  **L1140 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vpkuhum is lowered to a shuffle.`。
- **L1141 EN**: Declares TableGen def `int_ppc_altivec_vpkuhus`.
  **L1141 CN**: 声明 TableGen def `int_ppc_altivec_vpkuhus`。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v8i16_ty, llvm_v8i16_ty],`。
- **L1143 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1143 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1144 EN**: Comment explains nearby logic, invariants, or intent: `vpkuwum is lowered to a shuffle.`.
  **L1144 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vpkuwum is lowered to a shuffle.`。
- **L1145 EN**: Declares TableGen def `int_ppc_altivec_vpkuwus`.
  **L1145 CN**: 声明 TableGen def `int_ppc_altivec_vpkuwus`。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1147 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1147 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1148 EN**: Comment explains nearby logic, invariants, or intent: `vpkudum is lowered to a shuffle.`.
  **L1148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`vpkudum is lowered to a shuffle.`。

### Lines 1149-1176

````tablegen
  def int_ppc_altivec_vpkudus : ClangBuiltin<"__builtin_altivec_vpkudus">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem, IntrHasSideEffects]>;

  // Unpacks.
  def int_ppc_altivec_vupkhpx : ClangBuiltin<"__builtin_altivec_vupkhpx">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vupkhsb : ClangBuiltin<"__builtin_altivec_vupkhsb">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vupkhsh : ClangBuiltin<"__builtin_altivec_vupkhsh">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vupkhsw : ClangBuiltin<"__builtin_altivec_vupkhsw">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vupklpx : ClangBuiltin<"__builtin_altivec_vupklpx">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vupklsb : ClangBuiltin<"__builtin_altivec_vupklsb">,
      DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vupklsh : ClangBuiltin<"__builtin_altivec_vupklsh">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;
  def int_ppc_altivec_vupklsw : ClangBuiltin<"__builtin_altivec_vupklsw">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;


  // FP <-> integer conversion.
  def int_ppc_altivec_vcfsx : ClangBuiltin<"__builtin_altivec_vcfsx">,
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_altivec_vcfux : ClangBuiltin<"__builtin_altivec_vcfux">,
````
- **L1149 EN**: Declares TableGen def `int_ppc_altivec_vpkudus`.
  **L1149 CN**: 声明 TableGen def `int_ppc_altivec_vpkudus`。
- **L1150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1150 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1151 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L1151 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L1152 EN**: Blank line separating nearby declarations or logic blocks.
  **L1152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1153 EN**: Comment explains nearby logic, invariants, or intent: `Unpacks.`.
  **L1153 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unpacks.`。
- **L1154 EN**: Declares TableGen def `int_ppc_altivec_vupkhpx`.
  **L1154 CN**: 声明 TableGen def `int_ppc_altivec_vupkhpx`。
- **L1155 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1155 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1156 EN**: Declares TableGen def `int_ppc_altivec_vupkhsb`.
  **L1156 CN**: 声明 TableGen def `int_ppc_altivec_vupkhsb`。
- **L1157 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1157 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1158 EN**: Declares TableGen def `int_ppc_altivec_vupkhsh`.
  **L1158 CN**: 声明 TableGen def `int_ppc_altivec_vupkhsh`。
- **L1159 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1159 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1160 EN**: Declares TableGen def `int_ppc_altivec_vupkhsw`.
  **L1160 CN**: 声明 TableGen def `int_ppc_altivec_vupkhsw`。
- **L1161 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1161 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1162 EN**: Declares TableGen def `int_ppc_altivec_vupklpx`.
  **L1162 CN**: 声明 TableGen def `int_ppc_altivec_vupklpx`。
- **L1163 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1163 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1164 EN**: Declares TableGen def `int_ppc_altivec_vupklsb`.
  **L1164 CN**: 声明 TableGen def `int_ppc_altivec_vupklsb`。
- **L1165 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1165 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v8i16_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1166 EN**: Declares TableGen def `int_ppc_altivec_vupklsh`.
  **L1166 CN**: 声明 TableGen def `int_ppc_altivec_vupklsh`。
- **L1167 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`.
  **L1167 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v8i16_ty], [IntrNoMem]>;`。
- **L1168 EN**: Declares TableGen def `int_ppc_altivec_vupklsw`.
  **L1168 CN**: 声明 TableGen def `int_ppc_altivec_vupklsw`。
- **L1169 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1169 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1170 EN**: Blank line separating nearby declarations or logic blocks.
  **L1170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1171 EN**: Blank line separating nearby declarations or logic blocks.
  **L1171 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1172 EN**: Comment explains nearby logic, invariants, or intent: `FP <-> integer conversion.`.
  **L1172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP <-> integer conversion.`。
- **L1173 EN**: Declares TableGen def `int_ppc_altivec_vcfsx`.
  **L1173 CN**: 声明 TableGen def `int_ppc_altivec_vcfsx`。
- **L1174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty, llvm_i32_ty],`.
  **L1174 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty, llvm_i32_ty],`。
- **L1175 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1175 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1176 EN**: Declares TableGen def `int_ppc_altivec_vcfux`.
  **L1176 CN**: 声明 TableGen def `int_ppc_altivec_vcfux`。

### Lines 1177-1204

````tablegen
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_altivec_vctsxs : ClangBuiltin<"__builtin_altivec_vctsxs">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_altivec_vctuxs : ClangBuiltin<"__builtin_altivec_vctuxs">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  def int_ppc_altivec_vrfim : ClangBuiltin<"__builtin_altivec_vrfim">,
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vrfin : ClangBuiltin<"__builtin_altivec_vrfin">,
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vrfip : ClangBuiltin<"__builtin_altivec_vrfip">,
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vrfiz : ClangBuiltin<"__builtin_altivec_vrfiz">,
      DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;

  // Add Extended Quadword
  def int_ppc_altivec_vaddeuqm : ClangBuiltin<"__builtin_altivec_vaddeuqm">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty],
                            [llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vaddecuq : ClangBuiltin<"__builtin_altivec_vaddecuq">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty],
                            [llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;

````
- **L1177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty, llvm_i32_ty],`.
  **L1177 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4i32_ty, llvm_i32_ty],`。
- **L1178 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1178 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1179 EN**: Declares TableGen def `int_ppc_altivec_vctsxs`.
  **L1179 CN**: 声明 TableGen def `int_ppc_altivec_vctsxs`。
- **L1180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_i32_ty],`.
  **L1180 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_i32_ty],`。
- **L1181 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1181 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1182 EN**: Declares TableGen def `int_ppc_altivec_vctuxs`.
  **L1182 CN**: 声明 TableGen def `int_ppc_altivec_vctuxs`。
- **L1183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_i32_ty],`.
  **L1183 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4f32_ty, llvm_i32_ty],`。
- **L1184 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1184 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1186 EN**: Declares TableGen def `int_ppc_altivec_vrfim`.
  **L1186 CN**: 声明 TableGen def `int_ppc_altivec_vrfim`。
- **L1187 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1187 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1188 EN**: Declares TableGen def `int_ppc_altivec_vrfin`.
  **L1188 CN**: 声明 TableGen def `int_ppc_altivec_vrfin`。
- **L1189 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1189 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1190 EN**: Declares TableGen def `int_ppc_altivec_vrfip`.
  **L1190 CN**: 声明 TableGen def `int_ppc_altivec_vrfip`。
- **L1191 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1191 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1192 EN**: Declares TableGen def `int_ppc_altivec_vrfiz`.
  **L1192 CN**: 声明 TableGen def `int_ppc_altivec_vrfiz`。
- **L1193 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1193 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1194 EN**: Blank line separating nearby declarations or logic blocks.
  **L1194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1195 EN**: Comment explains nearby logic, invariants, or intent: `Add Extended Quadword`.
  **L1195 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Add Extended Quadword`。
- **L1196 EN**: Declares TableGen def `int_ppc_altivec_vaddeuqm`.
  **L1196 CN**: 声明 TableGen def `int_ppc_altivec_vaddeuqm`。
- **L1197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty],`.
  **L1197 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty],`。
- **L1198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L1198 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L1199 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1199 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1200 EN**: Declares TableGen def `int_ppc_altivec_vaddecuq`.
  **L1200 CN**: 声明 TableGen def `int_ppc_altivec_vaddecuq`。
- **L1201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty],`.
  **L1201 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty],`。
- **L1202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L1202 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L1203 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1203 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1204 EN**: Blank line separating nearby declarations or logic blocks.
  **L1204 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1205-1232

````tablegen
  // Sub Extended Quadword
  def int_ppc_altivec_vsubeuqm : ClangBuiltin<"__builtin_altivec_vsubeuqm">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty],
                            [llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vsubecuq : ClangBuiltin<"__builtin_altivec_vsubecuq">,
      DefaultAttrsIntrinsic<[llvm_v1i128_ty],
                            [llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;

  // P10 Vector Count Leading / Trailing Zeroes under bit Mask Builtins.
  def int_ppc_altivec_vclzdm : ClangBuiltin<"__builtin_altivec_vclzdm">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vctzdm : ClangBuiltin<"__builtin_altivec_vctzdm">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;
}

def int_ppc_altivec_vsl   : PowerPC_Vec_WWW_Intrinsic<"vsl">;
def int_ppc_altivec_vslo  : PowerPC_Vec_WWW_Intrinsic<"vslo">;

def int_ppc_altivec_vslb  : PowerPC_Vec_BBB_Intrinsic<"vslb">;
def int_ppc_altivec_vslv  : PowerPC_Vec_BBB_Intrinsic<"vslv">;
def int_ppc_altivec_vsrv  : PowerPC_Vec_BBB_Intrinsic<"vsrv">;
def int_ppc_altivec_vslh  : PowerPC_Vec_HHH_Intrinsic<"vslh">;
def int_ppc_altivec_vslw  : PowerPC_Vec_WWW_Intrinsic<"vslw">;

````
- **L1205 EN**: Comment explains nearby logic, invariants, or intent: `Sub Extended Quadword`.
  **L1205 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sub Extended Quadword`。
- **L1206 EN**: Declares TableGen def `int_ppc_altivec_vsubeuqm`.
  **L1206 CN**: 声明 TableGen def `int_ppc_altivec_vsubeuqm`。
- **L1207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty],`.
  **L1207 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty],`。
- **L1208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L1208 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L1209 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1209 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1210 EN**: Declares TableGen def `int_ppc_altivec_vsubecuq`.
  **L1210 CN**: 声明 TableGen def `int_ppc_altivec_vsubecuq`。
- **L1211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1i128_ty],`.
  **L1211 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1i128_ty],`。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L1213 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1213 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1215 EN**: Comment explains nearby logic, invariants, or intent: `P10 Vector Count Leading / Trailing Zeroes under bit Mask Builtins.`.
  **L1215 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 Vector Count Leading / Trailing Zeroes under bit Mask Builtins.`。
- **L1216 EN**: Declares TableGen def `int_ppc_altivec_vclzdm`.
  **L1216 CN**: 声明 TableGen def `int_ppc_altivec_vclzdm`。
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1218 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1218 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1219 EN**: Declares TableGen def `int_ppc_altivec_vctzdm`.
  **L1219 CN**: 声明 TableGen def `int_ppc_altivec_vctzdm`。
- **L1220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1220 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1221 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1221 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1222 EN**: Closes the current lexical scope or compound statement.
  **L1222 CN**: 结束当前词法作用域或复合语句块。
- **L1223 EN**: Blank line separating nearby declarations or logic blocks.
  **L1223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1224 EN**: Declares TableGen def `int_ppc_altivec_vsl`.
  **L1224 CN**: 声明 TableGen def `int_ppc_altivec_vsl`。
- **L1225 EN**: Declares TableGen def `int_ppc_altivec_vslo`.
  **L1225 CN**: 声明 TableGen def `int_ppc_altivec_vslo`。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1227 EN**: Declares TableGen def `int_ppc_altivec_vslb`.
  **L1227 CN**: 声明 TableGen def `int_ppc_altivec_vslb`。
- **L1228 EN**: Declares TableGen def `int_ppc_altivec_vslv`.
  **L1228 CN**: 声明 TableGen def `int_ppc_altivec_vslv`。
- **L1229 EN**: Declares TableGen def `int_ppc_altivec_vsrv`.
  **L1229 CN**: 声明 TableGen def `int_ppc_altivec_vsrv`。
- **L1230 EN**: Declares TableGen def `int_ppc_altivec_vslh`.
  **L1230 CN**: 声明 TableGen def `int_ppc_altivec_vslh`。
- **L1231 EN**: Declares TableGen def `int_ppc_altivec_vslw`.
  **L1231 CN**: 声明 TableGen def `int_ppc_altivec_vslw`。
- **L1232 EN**: Blank line separating nearby declarations or logic blocks.
  **L1232 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1233-1260

````tablegen
// Right Shifts.
def int_ppc_altivec_vsr   : PowerPC_Vec_WWW_Intrinsic<"vsr">;
def int_ppc_altivec_vsro  : PowerPC_Vec_WWW_Intrinsic<"vsro">;

def int_ppc_altivec_vsrb  : PowerPC_Vec_BBB_Intrinsic<"vsrb">;
def int_ppc_altivec_vsrh  : PowerPC_Vec_HHH_Intrinsic<"vsrh">;
def int_ppc_altivec_vsrw  : PowerPC_Vec_WWW_Intrinsic<"vsrw">;
def int_ppc_altivec_vsrab : PowerPC_Vec_BBB_Intrinsic<"vsrab">;
def int_ppc_altivec_vsrah : PowerPC_Vec_HHH_Intrinsic<"vsrah">;
def int_ppc_altivec_vsraw : PowerPC_Vec_WWW_Intrinsic<"vsraw">;

// Rotates.
def int_ppc_altivec_vrlb  : PowerPC_Vec_BBB_Intrinsic<"vrlb">;
def int_ppc_altivec_vrlh  : PowerPC_Vec_HHH_Intrinsic<"vrlh">;
def int_ppc_altivec_vrlw  : PowerPC_Vec_WWW_Intrinsic<"vrlw">;
def int_ppc_vsx_xvrlw     : PowerPC_VSX_WWW_Intrinsic<"xvrlw">;
def int_ppc_altivec_vrld  : PowerPC_Vec_DDD_Intrinsic<"vrld">;

let TargetPrefix = "ppc" in {  // All PPC intrinsics start with "llvm.ppc.".
  // Miscellaneous.
  def int_ppc_altivec_lvsl :
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty], [IntrNoMem]>;
  def int_ppc_altivec_lvsr :
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty], [IntrNoMem]>;

  def int_ppc_altivec_vperm : ClangBuiltin<"__builtin_altivec_vperm_4si">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty,
                             llvm_v4i32_ty, llvm_v16i8_ty], [IntrNoMem]>;
````
- **L1233 EN**: Comment explains nearby logic, invariants, or intent: `Right Shifts.`.
  **L1233 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Right Shifts.`。
- **L1234 EN**: Declares TableGen def `int_ppc_altivec_vsr`.
  **L1234 CN**: 声明 TableGen def `int_ppc_altivec_vsr`。
- **L1235 EN**: Declares TableGen def `int_ppc_altivec_vsro`.
  **L1235 CN**: 声明 TableGen def `int_ppc_altivec_vsro`。
- **L1236 EN**: Blank line separating nearby declarations or logic blocks.
  **L1236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1237 EN**: Declares TableGen def `int_ppc_altivec_vsrb`.
  **L1237 CN**: 声明 TableGen def `int_ppc_altivec_vsrb`。
- **L1238 EN**: Declares TableGen def `int_ppc_altivec_vsrh`.
  **L1238 CN**: 声明 TableGen def `int_ppc_altivec_vsrh`。
- **L1239 EN**: Declares TableGen def `int_ppc_altivec_vsrw`.
  **L1239 CN**: 声明 TableGen def `int_ppc_altivec_vsrw`。
- **L1240 EN**: Declares TableGen def `int_ppc_altivec_vsrab`.
  **L1240 CN**: 声明 TableGen def `int_ppc_altivec_vsrab`。
- **L1241 EN**: Declares TableGen def `int_ppc_altivec_vsrah`.
  **L1241 CN**: 声明 TableGen def `int_ppc_altivec_vsrah`。
- **L1242 EN**: Declares TableGen def `int_ppc_altivec_vsraw`.
  **L1242 CN**: 声明 TableGen def `int_ppc_altivec_vsraw`。
- **L1243 EN**: Blank line separating nearby declarations or logic blocks.
  **L1243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1244 EN**: Comment explains nearby logic, invariants, or intent: `Rotates.`.
  **L1244 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rotates.`。
- **L1245 EN**: Declares TableGen def `int_ppc_altivec_vrlb`.
  **L1245 CN**: 声明 TableGen def `int_ppc_altivec_vrlb`。
- **L1246 EN**: Declares TableGen def `int_ppc_altivec_vrlh`.
  **L1246 CN**: 声明 TableGen def `int_ppc_altivec_vrlh`。
- **L1247 EN**: Declares TableGen def `int_ppc_altivec_vrlw`.
  **L1247 CN**: 声明 TableGen def `int_ppc_altivec_vrlw`。
- **L1248 EN**: Declares TableGen def `int_ppc_vsx_xvrlw`.
  **L1248 CN**: 声明 TableGen def `int_ppc_vsx_xvrlw`。
- **L1249 EN**: Declares TableGen def `int_ppc_altivec_vrld`.
  **L1249 CN**: 声明 TableGen def `int_ppc_altivec_vrld`。
- **L1250 EN**: Blank line separating nearby declarations or logic blocks.
  **L1250 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1251 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1251 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1252 EN**: Comment explains nearby logic, invariants, or intent: `Miscellaneous.`.
  **L1252 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Miscellaneous.`。
- **L1253 EN**: Declares TableGen def `int_ppc_altivec_lvsl`.
  **L1253 CN**: 声明 TableGen def `int_ppc_altivec_lvsl`。
- **L1254 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty], [IntrNoMem]>;`.
  **L1254 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty], [IntrNoMem]>;`。
- **L1255 EN**: Declares TableGen def `int_ppc_altivec_lvsr`.
  **L1255 CN**: 声明 TableGen def `int_ppc_altivec_lvsr`。
- **L1256 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty], [IntrNoMem]>;`.
  **L1256 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty], [IntrNoMem]>;`。
- **L1257 EN**: Blank line separating nearby declarations or logic blocks.
  **L1257 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1258 EN**: Declares TableGen def `int_ppc_altivec_vperm`.
  **L1258 CN**: 声明 TableGen def `int_ppc_altivec_vperm`。
- **L1259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty,`.
  **L1259 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty,`。
- **L1260 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1260 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty, llvm_v16i8_ty], [IntrNoMem]>;`。

### Lines 1261-1288

````tablegen
  def int_ppc_altivec_vsel : ClangBuiltin<"__builtin_altivec_vsel_4si">,
      DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty,
                             llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
  def int_ppc_altivec_vgbbd : ClangBuiltin<"__builtin_altivec_vgbbd">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;
  def int_ppc_altivec_vbpermq : ClangBuiltin<"__builtin_altivec_vbpermq">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_altivec_vbpermd : ClangBuiltin<"__builtin_altivec_vbpermd">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
}

def int_ppc_altivec_vexptefp  : PowerPC_Vec_FF_Intrinsic<"vexptefp">;
def int_ppc_altivec_vlogefp   : PowerPC_Vec_FF_Intrinsic<"vlogefp">;
def int_ppc_altivec_vrefp     : PowerPC_Vec_FF_Intrinsic<"vrefp">;
def int_ppc_altivec_vrsqrtefp : PowerPC_Vec_FF_Intrinsic<"vrsqrtefp">;

// Power8 Intrinsics
// Crypto
let TargetPrefix = "ppc" in {  // All PPC intrinsics start with "llvm.ppc.".
  def int_ppc_altivec_crypto_vsbox :
      ClangBuiltin<"__builtin_altivec_crypto_vsbox">,
      DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;
  def int_ppc_altivec_crypto_vpermxor :
      ClangBuiltin<"__builtin_altivec_crypto_vpermxor">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v16i8_ty], [IntrNoMem]>;
````
- **L1261 EN**: Declares TableGen def `int_ppc_altivec_vsel`.
  **L1261 CN**: 声明 TableGen def `int_ppc_altivec_vsel`。
- **L1262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty,`.
  **L1262 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty,`。
- **L1263 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1263 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1264 EN**: Declares TableGen def `int_ppc_altivec_vgbbd`.
  **L1264 CN**: 声明 TableGen def `int_ppc_altivec_vgbbd`。
- **L1265 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1265 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1266 EN**: Declares TableGen def `int_ppc_altivec_vbpermq`.
  **L1266 CN**: 声明 TableGen def `int_ppc_altivec_vbpermq`。
- **L1267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1267 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1268 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1268 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1269 EN**: Declares TableGen def `int_ppc_altivec_vbpermd`.
  **L1269 CN**: 声明 TableGen def `int_ppc_altivec_vbpermd`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v16i8_ty],`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_v16i8_ty],`。
- **L1271 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1271 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1272 EN**: Closes the current lexical scope or compound statement.
  **L1272 CN**: 结束当前词法作用域或复合语句块。
- **L1273 EN**: Blank line separating nearby declarations or logic blocks.
  **L1273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1274 EN**: Declares TableGen def `int_ppc_altivec_vexptefp`.
  **L1274 CN**: 声明 TableGen def `int_ppc_altivec_vexptefp`。
- **L1275 EN**: Declares TableGen def `int_ppc_altivec_vlogefp`.
  **L1275 CN**: 声明 TableGen def `int_ppc_altivec_vlogefp`。
- **L1276 EN**: Declares TableGen def `int_ppc_altivec_vrefp`.
  **L1276 CN**: 声明 TableGen def `int_ppc_altivec_vrefp`。
- **L1277 EN**: Declares TableGen def `int_ppc_altivec_vrsqrtefp`.
  **L1277 CN**: 声明 TableGen def `int_ppc_altivec_vrsqrtefp`。
- **L1278 EN**: Blank line separating nearby declarations or logic blocks.
  **L1278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1279 EN**: Comment explains nearby logic, invariants, or intent: `Power8 Intrinsics`.
  **L1279 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Power8 Intrinsics`。
- **L1280 EN**: Comment explains nearby logic, invariants, or intent: `Crypto`.
  **L1280 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Crypto`。
- **L1281 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1281 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1282 EN**: Declares TableGen def `int_ppc_altivec_crypto_vsbox`.
  **L1282 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vsbox`。
- **L1283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_altivec_crypto_vsbox">,`.
  **L1283 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_altivec_crypto_vsbox">,`。
- **L1284 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1284 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1285 EN**: Declares TableGen def `int_ppc_altivec_crypto_vpermxor`.
  **L1285 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vpermxor`。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_altivec_crypto_vpermxor">,`.
  **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_altivec_crypto_vpermxor">,`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1288 EN**: Executes a standalone statement or declaration: `llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1288 CN**: 执行一条独立语句或声明：`llvm_v16i8_ty], [IntrNoMem]>;`。

### Lines 1289-1316

````tablegen
  def int_ppc_altivec_crypto_vpermxor_be :
      ClangBuiltin<"__builtin_altivec_crypto_vpermxor_be">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v16i8_ty], [IntrNoMem]>;

def int_ppc_altivec_crypto_vshasigmad :
    ClangBuiltin<"__builtin_altivec_crypto_vshasigmad">,
    DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty,
                           llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>,
                           ImmArg<ArgIndex<2>>]>;
def int_ppc_altivec_crypto_vshasigmaw :
    ClangBuiltin<"__builtin_altivec_crypto_vshasigmaw">,
    DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty,
                           llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>,
                           ImmArg<ArgIndex<2>>]>;
}
def int_ppc_altivec_crypto_vcipher :
            PowerPC_Vec_DDD_Intrinsic<"crypto_vcipher">;
def int_ppc_altivec_crypto_vcipherlast :
            PowerPC_Vec_DDD_Intrinsic<"crypto_vcipherlast">;
def int_ppc_altivec_crypto_vncipher :
            PowerPC_Vec_DDD_Intrinsic<"crypto_vncipher">;
def int_ppc_altivec_crypto_vncipherlast :
            PowerPC_Vec_DDD_Intrinsic<"crypto_vncipherlast">;
def int_ppc_altivec_crypto_vpmsumb :
            PowerPC_Vec_BBB_Intrinsic<"crypto_vpmsumb">;
def int_ppc_altivec_crypto_vpmsumh :
            PowerPC_Vec_HHH_Intrinsic<"crypto_vpmsumh">;
````
- **L1289 EN**: Declares TableGen def `int_ppc_altivec_crypto_vpermxor_be`.
  **L1289 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vpermxor_be`。
- **L1290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_altivec_crypto_vpermxor_be">,`.
  **L1290 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_altivec_crypto_vpermxor_be">,`。
- **L1291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1291 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1292 EN**: Executes a standalone statement or declaration: `llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1292 CN**: 执行一条独立语句或声明：`llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1294 EN**: Declares TableGen def `int_ppc_altivec_crypto_vshasigmad`.
  **L1294 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vshasigmad`。
- **L1295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_altivec_crypto_vshasigmad">,`.
  **L1295 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_altivec_crypto_vshasigmad">,`。
- **L1296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty,`.
  **L1296 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty], [llvm_v2i64_ty, llvm_i32_ty,`。
- **L1297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>,`.
  **L1297 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>,`。
- **L1298 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>]>;`.
  **L1298 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>]>;`。
- **L1299 EN**: Declares TableGen def `int_ppc_altivec_crypto_vshasigmaw`.
  **L1299 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vshasigmaw`。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_altivec_crypto_vshasigmaw">,`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_altivec_crypto_vshasigmaw">,`。
- **L1301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty,`.
  **L1301 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_v4i32_ty, llvm_i32_ty,`。
- **L1302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>,`.
  **L1302 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>,`。
- **L1303 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<2>>]>;`.
  **L1303 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<2>>]>;`。
- **L1304 EN**: Closes the current lexical scope or compound statement.
  **L1304 CN**: 结束当前词法作用域或复合语句块。
- **L1305 EN**: Declares TableGen def `int_ppc_altivec_crypto_vcipher`.
  **L1305 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vcipher`。
- **L1306 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_DDD_Intrinsic<"crypto_vcipher">;`.
  **L1306 CN**: 执行一条独立语句或声明：`PowerPC_Vec_DDD_Intrinsic<"crypto_vcipher">;`。
- **L1307 EN**: Declares TableGen def `int_ppc_altivec_crypto_vcipherlast`.
  **L1307 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vcipherlast`。
- **L1308 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_DDD_Intrinsic<"crypto_vcipherlast">;`.
  **L1308 CN**: 执行一条独立语句或声明：`PowerPC_Vec_DDD_Intrinsic<"crypto_vcipherlast">;`。
- **L1309 EN**: Declares TableGen def `int_ppc_altivec_crypto_vncipher`.
  **L1309 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vncipher`。
- **L1310 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_DDD_Intrinsic<"crypto_vncipher">;`.
  **L1310 CN**: 执行一条独立语句或声明：`PowerPC_Vec_DDD_Intrinsic<"crypto_vncipher">;`。
- **L1311 EN**: Declares TableGen def `int_ppc_altivec_crypto_vncipherlast`.
  **L1311 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vncipherlast`。
- **L1312 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_DDD_Intrinsic<"crypto_vncipherlast">;`.
  **L1312 CN**: 执行一条独立语句或声明：`PowerPC_Vec_DDD_Intrinsic<"crypto_vncipherlast">;`。
- **L1313 EN**: Declares TableGen def `int_ppc_altivec_crypto_vpmsumb`.
  **L1313 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vpmsumb`。
- **L1314 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_BBB_Intrinsic<"crypto_vpmsumb">;`.
  **L1314 CN**: 执行一条独立语句或声明：`PowerPC_Vec_BBB_Intrinsic<"crypto_vpmsumb">;`。
- **L1315 EN**: Declares TableGen def `int_ppc_altivec_crypto_vpmsumh`.
  **L1315 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vpmsumh`。
- **L1316 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_HHH_Intrinsic<"crypto_vpmsumh">;`.
  **L1316 CN**: 执行一条独立语句或声明：`PowerPC_Vec_HHH_Intrinsic<"crypto_vpmsumh">;`。

### Lines 1317-1344

````tablegen
def int_ppc_altivec_crypto_vpmsumw :
            PowerPC_Vec_WWW_Intrinsic<"crypto_vpmsumw">;
def int_ppc_altivec_crypto_vpmsumd :
            PowerPC_Vec_DDD_Intrinsic<"crypto_vpmsumd">;

// Absolute Difference intrinsics
def int_ppc_altivec_vabsdub : PowerPC_Vec_BBB_Intrinsic<"vabsdub">;
def int_ppc_altivec_vabsduh : PowerPC_Vec_HHH_Intrinsic<"vabsduh">;
def int_ppc_altivec_vabsduw : PowerPC_Vec_WWW_Intrinsic<"vabsduw">;

// Vector rotates
def int_ppc_altivec_vrlwnm :
      PowerPC_Vec_Intrinsic<"vrlwnm", [llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_ppc_altivec_vrlwmi :
      PowerPC_Vec_Intrinsic<"vrlwmi", [llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
                            [IntrNoMem]>;
def int_ppc_altivec_vrldnm :
      PowerPC_Vec_Intrinsic<"vrldnm", [llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;
def int_ppc_altivec_vrldmi :
      PowerPC_Vec_Intrinsic<"vrldmi", [llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
                            [IntrNoMem]>;

def int_ppc_altivec_vrlqnm :
      PowerPC_Vec_Intrinsic<"vrlqnm", [llvm_v1i128_ty],
````
- **L1317 EN**: Declares TableGen def `int_ppc_altivec_crypto_vpmsumw`.
  **L1317 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vpmsumw`。
- **L1318 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_WWW_Intrinsic<"crypto_vpmsumw">;`.
  **L1318 CN**: 执行一条独立语句或声明：`PowerPC_Vec_WWW_Intrinsic<"crypto_vpmsumw">;`。
- **L1319 EN**: Declares TableGen def `int_ppc_altivec_crypto_vpmsumd`.
  **L1319 CN**: 声明 TableGen def `int_ppc_altivec_crypto_vpmsumd`。
- **L1320 EN**: Executes a standalone statement or declaration: `PowerPC_Vec_DDD_Intrinsic<"crypto_vpmsumd">;`.
  **L1320 CN**: 执行一条独立语句或声明：`PowerPC_Vec_DDD_Intrinsic<"crypto_vpmsumd">;`。
- **L1321 EN**: Blank line separating nearby declarations or logic blocks.
  **L1321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1322 EN**: Comment explains nearby logic, invariants, or intent: `Absolute Difference intrinsics`.
  **L1322 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Absolute Difference intrinsics`。
- **L1323 EN**: Declares TableGen def `int_ppc_altivec_vabsdub`.
  **L1323 CN**: 声明 TableGen def `int_ppc_altivec_vabsdub`。
- **L1324 EN**: Declares TableGen def `int_ppc_altivec_vabsduh`.
  **L1324 CN**: 声明 TableGen def `int_ppc_altivec_vabsduh`。
- **L1325 EN**: Declares TableGen def `int_ppc_altivec_vabsduw`.
  **L1325 CN**: 声明 TableGen def `int_ppc_altivec_vabsduw`。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1327 EN**: Comment explains nearby logic, invariants, or intent: `Vector rotates`.
  **L1327 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector rotates`。
- **L1328 EN**: Declares TableGen def `int_ppc_altivec_vrlwnm`.
  **L1328 CN**: 声明 TableGen def `int_ppc_altivec_vrlwnm`。
- **L1329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vrlwnm", [llvm_v4i32_ty],`.
  **L1329 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vrlwnm", [llvm_v4i32_ty],`。
- **L1330 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1330 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1331 EN**: Declares TableGen def `int_ppc_altivec_vrlwmi`.
  **L1331 CN**: 声明 TableGen def `int_ppc_altivec_vrlwmi`。
- **L1332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vrlwmi", [llvm_v4i32_ty],`.
  **L1332 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vrlwmi", [llvm_v4i32_ty],`。
- **L1333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1333 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1334 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1334 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1335 EN**: Declares TableGen def `int_ppc_altivec_vrldnm`.
  **L1335 CN**: 声明 TableGen def `int_ppc_altivec_vrldnm`。
- **L1336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vrldnm", [llvm_v2i64_ty],`.
  **L1336 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vrldnm", [llvm_v2i64_ty],`。
- **L1337 EN**: Executes a standalone statement or declaration: `[llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1337 CN**: 执行一条独立语句或声明：`[llvm_v2i64_ty, llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1338 EN**: Declares TableGen def `int_ppc_altivec_vrldmi`.
  **L1338 CN**: 声明 TableGen def `int_ppc_altivec_vrldmi`。
- **L1339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vrldmi", [llvm_v2i64_ty],`.
  **L1339 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vrldmi", [llvm_v2i64_ty],`。
- **L1340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1340 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1341 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1341 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1342 EN**: Blank line separating nearby declarations or logic blocks.
  **L1342 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1343 EN**: Declares TableGen def `int_ppc_altivec_vrlqnm`.
  **L1343 CN**: 声明 TableGen def `int_ppc_altivec_vrlqnm`。
- **L1344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vrlqnm", [llvm_v1i128_ty],`.
  **L1344 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vrlqnm", [llvm_v1i128_ty],`。

### Lines 1345-1372

````tablegen
                           [llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;
def int_ppc_altivec_vrlqmi :
      PowerPC_Vec_Intrinsic<"vrlqmi", [llvm_v1i128_ty],
                            [llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],
                            [IntrNoMem]>;

// Vector Divide Extended Intrinsics.
def int_ppc_altivec_vdivesw : PowerPC_Vec_WWW_Intrinsic<"vdivesw">;
def int_ppc_altivec_vdiveuw : PowerPC_Vec_WWW_Intrinsic<"vdiveuw">;
def int_ppc_altivec_vdivesd : PowerPC_Vec_DDD_Intrinsic<"vdivesd">;
def int_ppc_altivec_vdiveud : PowerPC_Vec_DDD_Intrinsic<"vdiveud">;
def int_ppc_altivec_vdivesq : PowerPC_Vec_QQQ_Intrinsic<"vdivesq">;
def int_ppc_altivec_vdiveuq : PowerPC_Vec_QQQ_Intrinsic<"vdiveuq">;

// Vector Multiply High Intrinsics.
def int_ppc_altivec_vmulhsw : PowerPC_Vec_WWW_Intrinsic<"vmulhsw">;
def int_ppc_altivec_vmulhuw : PowerPC_Vec_WWW_Intrinsic<"vmulhuw">;
def int_ppc_altivec_vmulhsd : PowerPC_Vec_DDD_Intrinsic<"vmulhsd">;
def int_ppc_altivec_vmulhud : PowerPC_Vec_DDD_Intrinsic<"vmulhud">;

// Deeply Compressed Weights Intrinsics.
def int_ppc_altivec_vucmprhn : PowerPC_Vec_BBB_Intrinsic<"vucmprhn">;
def int_ppc_altivec_vucmprln : PowerPC_Vec_BBB_Intrinsic<"vucmprln">;
def int_ppc_altivec_vucmprhb : PowerPC_Vec_BBB_Intrinsic<"vucmprhb">;
def int_ppc_altivec_vucmprlb : PowerPC_Vec_BBB_Intrinsic<"vucmprlb">;
def int_ppc_altivec_vucmprhh : PowerPC_Vec_BBB_Intrinsic<"vucmprhh">;
def int_ppc_altivec_vucmprlh : PowerPC_Vec_BBB_Intrinsic<"vucmprlh">;
````
- **L1345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L1345 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L1346 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1346 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1347 EN**: Declares TableGen def `int_ppc_altivec_vrlqmi`.
  **L1347 CN**: 声明 TableGen def `int_ppc_altivec_vrlqmi`。
- **L1348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vrlqmi", [llvm_v1i128_ty],`.
  **L1348 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vrlqmi", [llvm_v1i128_ty],`。
- **L1349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`.
  **L1349 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1i128_ty, llvm_v1i128_ty, llvm_v1i128_ty],`。
- **L1350 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1350 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1351 EN**: Blank line separating nearby declarations or logic blocks.
  **L1351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1352 EN**: Comment explains nearby logic, invariants, or intent: `Vector Divide Extended Intrinsics.`.
  **L1352 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Divide Extended Intrinsics.`。
- **L1353 EN**: Declares TableGen def `int_ppc_altivec_vdivesw`.
  **L1353 CN**: 声明 TableGen def `int_ppc_altivec_vdivesw`。
- **L1354 EN**: Declares TableGen def `int_ppc_altivec_vdiveuw`.
  **L1354 CN**: 声明 TableGen def `int_ppc_altivec_vdiveuw`。
- **L1355 EN**: Declares TableGen def `int_ppc_altivec_vdivesd`.
  **L1355 CN**: 声明 TableGen def `int_ppc_altivec_vdivesd`。
- **L1356 EN**: Declares TableGen def `int_ppc_altivec_vdiveud`.
  **L1356 CN**: 声明 TableGen def `int_ppc_altivec_vdiveud`。
- **L1357 EN**: Declares TableGen def `int_ppc_altivec_vdivesq`.
  **L1357 CN**: 声明 TableGen def `int_ppc_altivec_vdivesq`。
- **L1358 EN**: Declares TableGen def `int_ppc_altivec_vdiveuq`.
  **L1358 CN**: 声明 TableGen def `int_ppc_altivec_vdiveuq`。
- **L1359 EN**: Blank line separating nearby declarations or logic blocks.
  **L1359 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1360 EN**: Comment explains nearby logic, invariants, or intent: `Vector Multiply High Intrinsics.`.
  **L1360 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector Multiply High Intrinsics.`。
- **L1361 EN**: Declares TableGen def `int_ppc_altivec_vmulhsw`.
  **L1361 CN**: 声明 TableGen def `int_ppc_altivec_vmulhsw`。
- **L1362 EN**: Declares TableGen def `int_ppc_altivec_vmulhuw`.
  **L1362 CN**: 声明 TableGen def `int_ppc_altivec_vmulhuw`。
- **L1363 EN**: Declares TableGen def `int_ppc_altivec_vmulhsd`.
  **L1363 CN**: 声明 TableGen def `int_ppc_altivec_vmulhsd`。
- **L1364 EN**: Declares TableGen def `int_ppc_altivec_vmulhud`.
  **L1364 CN**: 声明 TableGen def `int_ppc_altivec_vmulhud`。
- **L1365 EN**: Blank line separating nearby declarations or logic blocks.
  **L1365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1366 EN**: Comment explains nearby logic, invariants, or intent: `Deeply Compressed Weights Intrinsics.`.
  **L1366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Deeply Compressed Weights Intrinsics.`。
- **L1367 EN**: Declares TableGen def `int_ppc_altivec_vucmprhn`.
  **L1367 CN**: 声明 TableGen def `int_ppc_altivec_vucmprhn`。
- **L1368 EN**: Declares TableGen def `int_ppc_altivec_vucmprln`.
  **L1368 CN**: 声明 TableGen def `int_ppc_altivec_vucmprln`。
- **L1369 EN**: Declares TableGen def `int_ppc_altivec_vucmprhb`.
  **L1369 CN**: 声明 TableGen def `int_ppc_altivec_vucmprhb`。
- **L1370 EN**: Declares TableGen def `int_ppc_altivec_vucmprlb`.
  **L1370 CN**: 声明 TableGen def `int_ppc_altivec_vucmprlb`。
- **L1371 EN**: Declares TableGen def `int_ppc_altivec_vucmprhh`.
  **L1371 CN**: 声明 TableGen def `int_ppc_altivec_vucmprhh`。
- **L1372 EN**: Declares TableGen def `int_ppc_altivec_vucmprlh`.
  **L1372 CN**: 声明 TableGen def `int_ppc_altivec_vucmprlh`。

### Lines 1373-1400

````tablegen
def int_ppc_altivec_vupkhsntob :
    PowerPC_Vec_Intrinsic<"vupkhsntob", [llvm_v16i8_ty],
                          [llvm_v16i8_ty], [IntrNoMem]>;
def int_ppc_altivec_vupklsntob :
    PowerPC_Vec_Intrinsic<"vupklsntob", [llvm_v16i8_ty],
                          [llvm_v16i8_ty], [IntrNoMem]>;
def int_ppc_altivec_vupkint4tobf16 :
    PowerPC_Vec_Intrinsic<"vupkint4tobf16", [llvm_v16i8_ty],
                          [llvm_v16i8_ty, llvm_i32_ty],
                          [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_ppc_altivec_vupkint8tobf16 :
    PowerPC_Vec_Intrinsic<"vupkint8tobf16", [llvm_v16i8_ty],
                          [llvm_v16i8_ty, llvm_i32_ty],
                          [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_ppc_altivec_vupkint4tofp32 :
    PowerPC_Vec_Intrinsic<"vupkint4tofp32", [llvm_v16i8_ty],
                          [llvm_v16i8_ty, llvm_i32_ty],
                          [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_ppc_altivec_vupkint8tofp32 :
    PowerPC_Vec_Intrinsic<"vupkint8tofp32", [llvm_v16i8_ty],
                          [llvm_v16i8_ty, llvm_i32_ty],
                          [IntrNoMem, ImmArg<ArgIndex<1>>]>;

// Post Quantum Cryptography Acceleration.
def int_ppc_altivec_vmulhsh : PowerPC_Vec_HHH_Intrinsic<"vmulhsh">;
def int_ppc_altivec_vmulhuh : PowerPC_Vec_HHH_Intrinsic<"vmulhuh">;

//===----------------------------------------------------------------------===//
````
- **L1373 EN**: Declares TableGen def `int_ppc_altivec_vupkhsntob`.
  **L1373 CN**: 声明 TableGen def `int_ppc_altivec_vupkhsntob`。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vupkhsntob", [llvm_v16i8_ty],`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vupkhsntob", [llvm_v16i8_ty],`。
- **L1375 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1375 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1376 EN**: Declares TableGen def `int_ppc_altivec_vupklsntob`.
  **L1376 CN**: 声明 TableGen def `int_ppc_altivec_vupklsntob`。
- **L1377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vupklsntob", [llvm_v16i8_ty],`.
  **L1377 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vupklsntob", [llvm_v16i8_ty],`。
- **L1378 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1378 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1379 EN**: Declares TableGen def `int_ppc_altivec_vupkint4tobf16`.
  **L1379 CN**: 声明 TableGen def `int_ppc_altivec_vupkint4tobf16`。
- **L1380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vupkint4tobf16", [llvm_v16i8_ty],`.
  **L1380 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vupkint4tobf16", [llvm_v16i8_ty],`。
- **L1381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty],`.
  **L1381 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty],`。
- **L1382 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1382 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1383 EN**: Declares TableGen def `int_ppc_altivec_vupkint8tobf16`.
  **L1383 CN**: 声明 TableGen def `int_ppc_altivec_vupkint8tobf16`。
- **L1384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vupkint8tobf16", [llvm_v16i8_ty],`.
  **L1384 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vupkint8tobf16", [llvm_v16i8_ty],`。
- **L1385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty],`.
  **L1385 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty],`。
- **L1386 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1386 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1387 EN**: Declares TableGen def `int_ppc_altivec_vupkint4tofp32`.
  **L1387 CN**: 声明 TableGen def `int_ppc_altivec_vupkint4tofp32`。
- **L1388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vupkint4tofp32", [llvm_v16i8_ty],`.
  **L1388 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vupkint4tofp32", [llvm_v16i8_ty],`。
- **L1389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty],`.
  **L1389 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty],`。
- **L1390 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1390 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1391 EN**: Declares TableGen def `int_ppc_altivec_vupkint8tofp32`.
  **L1391 CN**: 声明 TableGen def `int_ppc_altivec_vupkint8tofp32`。
- **L1392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_Vec_Intrinsic<"vupkint8tofp32", [llvm_v16i8_ty],`.
  **L1392 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_Vec_Intrinsic<"vupkint8tofp32", [llvm_v16i8_ty],`。
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_i32_ty],`.
  **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_i32_ty],`。
- **L1394 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1394 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1395 EN**: Blank line separating nearby declarations or logic blocks.
  **L1395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1396 EN**: Comment explains nearby logic, invariants, or intent: `Post Quantum Cryptography Acceleration.`.
  **L1396 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Post Quantum Cryptography Acceleration.`。
- **L1397 EN**: Declares TableGen def `int_ppc_altivec_vmulhsh`.
  **L1397 CN**: 声明 TableGen def `int_ppc_altivec_vmulhsh`。
- **L1398 EN**: Declares TableGen def `int_ppc_altivec_vmulhuh`.
  **L1398 CN**: 声明 TableGen def `int_ppc_altivec_vmulhuh`。
- **L1399 EN**: Blank line separating nearby declarations or logic blocks.
  **L1399 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1400 EN**: Banner comment marking a file or section boundary.
  **L1400 CN**: 横幅注释，用于标记文件或章节边界。

### Lines 1401-1428

````tablegen
// PowerPC VSX Intrinsic Definitions.

let TargetPrefix = "ppc" in {  // All intrinsics start with "llvm.ppc.".

// Vector load.
def int_ppc_vsx_lxvw4x :
    DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvd2x :
    DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_ptr_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvw4x_be :
    DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvd2x_be :
    DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_ptr_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvl :
    DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvll :
    DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvp :
    DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvrl :
    DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],
````
- **L1401 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC VSX Intrinsic Definitions.`.
  **L1401 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC VSX Intrinsic Definitions.`。
- **L1402 EN**: Blank line separating nearby declarations or logic blocks.
  **L1402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1403 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1403 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1404 EN**: Blank line separating nearby declarations or logic blocks.
  **L1404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1405 EN**: Comment explains nearby logic, invariants, or intent: `Vector load.`.
  **L1405 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector load.`。
- **L1406 EN**: Declares TableGen def `int_ppc_vsx_lxvw4x`.
  **L1406 CN**: 声明 TableGen def `int_ppc_vsx_lxvw4x`。
- **L1407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`.
  **L1407 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`。
- **L1408 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1408 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1409 EN**: Declares TableGen def `int_ppc_vsx_lxvd2x`.
  **L1409 CN**: 声明 TableGen def `int_ppc_vsx_lxvd2x`。
- **L1410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_ptr_ty],`.
  **L1410 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_ptr_ty],`。
- **L1411 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1411 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1412 EN**: Declares TableGen def `int_ppc_vsx_lxvw4x_be`.
  **L1412 CN**: 声明 TableGen def `int_ppc_vsx_lxvw4x_be`。
- **L1413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`.
  **L1413 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty],`。
- **L1414 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1414 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1415 EN**: Declares TableGen def `int_ppc_vsx_lxvd2x_be`.
  **L1415 CN**: 声明 TableGen def `int_ppc_vsx_lxvd2x_be`。
- **L1416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_ptr_ty],`.
  **L1416 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_ptr_ty],`。
- **L1417 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1417 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1418 EN**: Declares TableGen def `int_ppc_vsx_lxvl`.
  **L1418 CN**: 声明 TableGen def `int_ppc_vsx_lxvl`。
- **L1419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1419 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L1420 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1420 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1421 EN**: Declares TableGen def `int_ppc_vsx_lxvll`.
  **L1421 CN**: 声明 TableGen def `int_ppc_vsx_lxvll`。
- **L1422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1422 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L1423 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1423 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1424 EN**: Declares TableGen def `int_ppc_vsx_lxvp`.
  **L1424 CN**: 声明 TableGen def `int_ppc_vsx_lxvp`。
- **L1425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty],`.
  **L1425 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty],`。
- **L1426 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1426 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1427 EN**: Declares TableGen def `int_ppc_vsx_lxvrl`.
  **L1427 CN**: 声明 TableGen def `int_ppc_vsx_lxvrl`。
- **L1428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1428 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`。

### Lines 1429-1456

````tablegen
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvrll :
    DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvprl :
    DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty, llvm_i64_ty],
                          [IntrReadMem, IntrArgMemOnly]>;
def int_ppc_vsx_lxvprll :
    DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty, llvm_i64_ty],
                          [IntrReadMem, IntrArgMemOnly]>;

// Vector store.
def int_ppc_vsx_stxvw4x : Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],
                                    [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvd2x : Intrinsic<[], [llvm_v2f64_ty, llvm_ptr_ty],
                                    [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvw4x_be : Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty],
                                       [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvd2x_be : Intrinsic<[], [llvm_v2f64_ty, llvm_ptr_ty],
                                       [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvl :
      Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],
      [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvll :
      Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],
      [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvp :
      Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty], [IntrWriteMem,
````
- **L1429 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1429 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1430 EN**: Declares TableGen def `int_ppc_vsx_lxvrll`.
  **L1430 CN**: 声明 TableGen def `int_ppc_vsx_lxvrll`。
- **L1431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1431 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L1432 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1432 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1433 EN**: Declares TableGen def `int_ppc_vsx_lxvprl`.
  **L1433 CN**: 声明 TableGen def `int_ppc_vsx_lxvprl`。
- **L1434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1434 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L1435 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1435 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1436 EN**: Declares TableGen def `int_ppc_vsx_lxvprll`.
  **L1436 CN**: 声明 TableGen def `int_ppc_vsx_lxvprll`。
- **L1437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1437 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L1438 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L1438 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L1439 EN**: Blank line separating nearby declarations or logic blocks.
  **L1439 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1440 EN**: Comment explains nearby logic, invariants, or intent: `Vector store.`.
  **L1440 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector store.`。
- **L1441 EN**: Declares TableGen def `int_ppc_vsx_stxvw4x`.
  **L1441 CN**: 声明 TableGen def `int_ppc_vsx_stxvw4x`。
- **L1442 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1442 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1443 EN**: Declares TableGen def `int_ppc_vsx_stxvd2x`.
  **L1443 CN**: 声明 TableGen def `int_ppc_vsx_stxvd2x`。
- **L1444 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1444 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1445 EN**: Declares TableGen def `int_ppc_vsx_stxvw4x_be`.
  **L1445 CN**: 声明 TableGen def `int_ppc_vsx_stxvw4x_be`。
- **L1446 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1446 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1447 EN**: Declares TableGen def `int_ppc_vsx_stxvd2x_be`.
  **L1447 CN**: 声明 TableGen def `int_ppc_vsx_stxvd2x_be`。
- **L1448 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1448 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1449 EN**: Declares TableGen def `int_ppc_vsx_stxvl`.
  **L1449 CN**: 声明 TableGen def `int_ppc_vsx_stxvl`。
- **L1450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`.
  **L1450 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`。
- **L1451 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1451 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1452 EN**: Declares TableGen def `int_ppc_vsx_stxvll`.
  **L1452 CN**: 声明 TableGen def `int_ppc_vsx_stxvll`。
- **L1453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`.
  **L1453 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`。
- **L1454 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1454 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1455 EN**: Declares TableGen def `int_ppc_vsx_stxvp`.
  **L1455 CN**: 声明 TableGen def `int_ppc_vsx_stxvp`。
- **L1456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty], [IntrWriteMem,`.
  **L1456 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty], [IntrWriteMem,`。

### Lines 1457-1484

````tablegen
      IntrArgMemOnly]>;
def int_ppc_vsx_stxvrl :
      Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],
      [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvrll :
      Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],
      [IntrWriteMem, IntrArgMemOnly]>;
def int_ppc_vsx_stxvprl :
      Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty, llvm_i64_ty], [IntrWriteMem,
      IntrArgMemOnly]>;
def int_ppc_vsx_stxvprll :
      Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty, llvm_i64_ty], [IntrWriteMem,
      IntrArgMemOnly]>;

// Vector and scalar maximum.
def int_ppc_vsx_xvmaxdp : PowerPC_VSX_Vec_DDD_Intrinsic<"xvmaxdp">;
def int_ppc_vsx_xvmaxsp : PowerPC_VSX_Vec_FFF_Intrinsic<"xvmaxsp">;
def int_ppc_vsx_xsmaxdp : PowerPC_VSX_Sca_DDD_Intrinsic<"xsmaxdp">;

// Vector and scalar minimum.
def int_ppc_vsx_xvmindp : PowerPC_VSX_Vec_DDD_Intrinsic<"xvmindp">;
def int_ppc_vsx_xvminsp : PowerPC_VSX_Vec_FFF_Intrinsic<"xvminsp">;
def int_ppc_vsx_xsmindp : PowerPC_VSX_Sca_DDD_Intrinsic<"xsmindp">;

// Vector divide.
def int_ppc_vsx_xvdivdp : PowerPC_VSX_Vec_DDD_Intrinsic<"xvdivdp">;
def int_ppc_vsx_xvdivsp : PowerPC_VSX_Vec_FFF_Intrinsic<"xvdivsp">;

````
- **L1457 EN**: Executes a standalone statement or declaration: `IntrArgMemOnly]>;`.
  **L1457 CN**: 执行一条独立语句或声明：`IntrArgMemOnly]>;`。
- **L1458 EN**: Declares TableGen def `int_ppc_vsx_stxvrl`.
  **L1458 CN**: 声明 TableGen def `int_ppc_vsx_stxvrl`。
- **L1459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`.
  **L1459 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`。
- **L1460 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1460 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1461 EN**: Declares TableGen def `int_ppc_vsx_stxvrll`.
  **L1461 CN**: 声明 TableGen def `int_ppc_vsx_stxvrll`。
- **L1462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`.
  **L1462 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v4i32_ty, llvm_ptr_ty, llvm_i64_ty],`。
- **L1463 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1463 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1464 EN**: Declares TableGen def `int_ppc_vsx_stxvprl`.
  **L1464 CN**: 声明 TableGen def `int_ppc_vsx_stxvprl`。
- **L1465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty, llvm_i64_ty], [IntrWriteMem,`.
  **L1465 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty, llvm_i64_ty], [IntrWriteMem,`。
- **L1466 EN**: Executes a standalone statement or declaration: `IntrArgMemOnly]>;`.
  **L1466 CN**: 执行一条独立语句或声明：`IntrArgMemOnly]>;`。
- **L1467 EN**: Declares TableGen def `int_ppc_vsx_stxvprll`.
  **L1467 CN**: 声明 TableGen def `int_ppc_vsx_stxvprll`。
- **L1468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty, llvm_i64_ty], [IntrWriteMem,`.
  **L1468 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v256i1_ty, llvm_ptr_ty, llvm_i64_ty], [IntrWriteMem,`。
- **L1469 EN**: Executes a standalone statement or declaration: `IntrArgMemOnly]>;`.
  **L1469 CN**: 执行一条独立语句或声明：`IntrArgMemOnly]>;`。
- **L1470 EN**: Blank line separating nearby declarations or logic blocks.
  **L1470 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1471 EN**: Comment explains nearby logic, invariants, or intent: `Vector and scalar maximum.`.
  **L1471 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector and scalar maximum.`。
- **L1472 EN**: Declares TableGen def `int_ppc_vsx_xvmaxdp`.
  **L1472 CN**: 声明 TableGen def `int_ppc_vsx_xvmaxdp`。
- **L1473 EN**: Declares TableGen def `int_ppc_vsx_xvmaxsp`.
  **L1473 CN**: 声明 TableGen def `int_ppc_vsx_xvmaxsp`。
- **L1474 EN**: Declares TableGen def `int_ppc_vsx_xsmaxdp`.
  **L1474 CN**: 声明 TableGen def `int_ppc_vsx_xsmaxdp`。
- **L1475 EN**: Blank line separating nearby declarations or logic blocks.
  **L1475 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1476 EN**: Comment explains nearby logic, invariants, or intent: `Vector and scalar minimum.`.
  **L1476 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector and scalar minimum.`。
- **L1477 EN**: Declares TableGen def `int_ppc_vsx_xvmindp`.
  **L1477 CN**: 声明 TableGen def `int_ppc_vsx_xvmindp`。
- **L1478 EN**: Declares TableGen def `int_ppc_vsx_xvminsp`.
  **L1478 CN**: 声明 TableGen def `int_ppc_vsx_xvminsp`。
- **L1479 EN**: Declares TableGen def `int_ppc_vsx_xsmindp`.
  **L1479 CN**: 声明 TableGen def `int_ppc_vsx_xsmindp`。
- **L1480 EN**: Blank line separating nearby declarations or logic blocks.
  **L1480 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1481 EN**: Comment explains nearby logic, invariants, or intent: `Vector divide.`.
  **L1481 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector divide.`。
- **L1482 EN**: Declares TableGen def `int_ppc_vsx_xvdivdp`.
  **L1482 CN**: 声明 TableGen def `int_ppc_vsx_xvdivdp`。
- **L1483 EN**: Declares TableGen def `int_ppc_vsx_xvdivsp`.
  **L1483 CN**: 声明 TableGen def `int_ppc_vsx_xvdivsp`。
- **L1484 EN**: Blank line separating nearby declarations or logic blocks.
  **L1484 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1485-1512

````tablegen
// Vector round-to-infinity (ceil)
def int_ppc_vsx_xvrspip :
    DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvrdpip :
    DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

// Vector reciprocal estimate
def int_ppc_vsx_xvresp : ClangBuiltin<"__builtin_vsx_xvresp">,
    DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvredp : ClangBuiltin<"__builtin_vsx_xvredp">,
    DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

// Vector rsqrte
def int_ppc_vsx_xvrsqrtesp : ClangBuiltin<"__builtin_vsx_xvrsqrtesp">,
    DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvrsqrtedp : ClangBuiltin<"__builtin_vsx_xvrsqrtedp">,
    DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;

// Vector compare
def int_ppc_vsx_xvcmpeqdp :
      PowerPC_VSX_Intrinsic<"xvcmpeqdp", [llvm_v2i64_ty],
                            [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcmpeqdp_p : ClangBuiltin<"__builtin_vsx_xvcmpeqdp_p">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],
                          [llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xvcmpeqsp :
      PowerPC_VSX_Intrinsic<"xvcmpeqsp", [llvm_v4i32_ty],
````
- **L1485 EN**: Comment explains nearby logic, invariants, or intent: `Vector round-to-infinity (ceil)`.
  **L1485 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector round-to-infinity (ceil)`。
- **L1486 EN**: Declares TableGen def `int_ppc_vsx_xvrspip`.
  **L1486 CN**: 声明 TableGen def `int_ppc_vsx_xvrspip`。
- **L1487 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1487 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1488 EN**: Declares TableGen def `int_ppc_vsx_xvrdpip`.
  **L1488 CN**: 声明 TableGen def `int_ppc_vsx_xvrdpip`。
- **L1489 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1489 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1490 EN**: Blank line separating nearby declarations or logic blocks.
  **L1490 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1491 EN**: Comment explains nearby logic, invariants, or intent: `Vector reciprocal estimate`.
  **L1491 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector reciprocal estimate`。
- **L1492 EN**: Declares TableGen def `int_ppc_vsx_xvresp`.
  **L1492 CN**: 声明 TableGen def `int_ppc_vsx_xvresp`。
- **L1493 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1493 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1494 EN**: Declares TableGen def `int_ppc_vsx_xvredp`.
  **L1494 CN**: 声明 TableGen def `int_ppc_vsx_xvredp`。
- **L1495 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1495 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1496 EN**: Blank line separating nearby declarations or logic blocks.
  **L1496 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1497 EN**: Comment explains nearby logic, invariants, or intent: `Vector rsqrte`.
  **L1497 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector rsqrte`。
- **L1498 EN**: Declares TableGen def `int_ppc_vsx_xvrsqrtesp`.
  **L1498 CN**: 声明 TableGen def `int_ppc_vsx_xvrsqrtesp`。
- **L1499 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1499 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v4f32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1500 EN**: Declares TableGen def `int_ppc_vsx_xvrsqrtedp`.
  **L1500 CN**: 声明 TableGen def `int_ppc_vsx_xvrsqrtedp`。
- **L1501 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1501 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2f64_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1502 EN**: Blank line separating nearby declarations or logic blocks.
  **L1502 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1503 EN**: Comment explains nearby logic, invariants, or intent: `Vector compare`.
  **L1503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector compare`。
- **L1504 EN**: Declares TableGen def `int_ppc_vsx_xvcmpeqdp`.
  **L1504 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpeqdp`。
- **L1505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcmpeqdp", [llvm_v2i64_ty],`.
  **L1505 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcmpeqdp", [llvm_v2i64_ty],`。
- **L1506 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1506 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1507 EN**: Declares TableGen def `int_ppc_vsx_xvcmpeqdp_p`.
  **L1507 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpeqdp_p`。
- **L1508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1508 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],`.
  **L1509 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],`。
- **L1510 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1510 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1511 EN**: Declares TableGen def `int_ppc_vsx_xvcmpeqsp`.
  **L1511 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpeqsp`。
- **L1512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcmpeqsp", [llvm_v4i32_ty],`.
  **L1512 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcmpeqsp", [llvm_v4i32_ty],`。

### Lines 1513-1540

````tablegen
                            [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcmpeqsp_p : ClangBuiltin<"__builtin_vsx_xvcmpeqsp_p">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],
                          [llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xvcmpgedp :
      PowerPC_VSX_Intrinsic<"xvcmpgedp", [llvm_v2i64_ty],
                            [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcmpgedp_p : ClangBuiltin<"__builtin_vsx_xvcmpgedp_p">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],
                          [llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xvcmpgesp :
      PowerPC_VSX_Intrinsic<"xvcmpgesp", [llvm_v4i32_ty],
                            [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcmpgesp_p : ClangBuiltin<"__builtin_vsx_xvcmpgesp_p">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],
                          [llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xvcmpgtdp :
      PowerPC_VSX_Intrinsic<"xvcmpgtdp", [llvm_v2i64_ty],
                            [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcmpgtdp_p : ClangBuiltin<"__builtin_vsx_xvcmpgtdp_p">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],
                          [llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xvcmpgtsp :
      PowerPC_VSX_Intrinsic<"xvcmpgtsp", [llvm_v4i32_ty],
````
- **L1513 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1513 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1514 EN**: Declares TableGen def `int_ppc_vsx_xvcmpeqsp_p`.
  **L1514 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpeqsp_p`。
- **L1515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1515 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`.
  **L1516 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`。
- **L1517 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1517 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1518 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgedp`.
  **L1518 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgedp`。
- **L1519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcmpgedp", [llvm_v2i64_ty],`.
  **L1519 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcmpgedp", [llvm_v2i64_ty],`。
- **L1520 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1520 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1521 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgedp_p`.
  **L1521 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgedp_p`。
- **L1522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1522 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],`.
  **L1523 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],`。
- **L1524 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1524 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1525 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgesp`.
  **L1525 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgesp`。
- **L1526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcmpgesp", [llvm_v4i32_ty],`.
  **L1526 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcmpgesp", [llvm_v4i32_ty],`。
- **L1527 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1527 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1528 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgesp_p`.
  **L1528 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgesp_p`。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`.
  **L1530 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`。
- **L1531 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1531 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1532 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgtdp`.
  **L1532 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgtdp`。
- **L1533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcmpgtdp", [llvm_v2i64_ty],`.
  **L1533 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcmpgtdp", [llvm_v2i64_ty],`。
- **L1534 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1534 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1535 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgtdp_p`.
  **L1535 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgtdp_p`。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],`.
  **L1537 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v2f64_ty,llvm_v2f64_ty],`。
- **L1538 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1538 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1539 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgtsp`.
  **L1539 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgtsp`。
- **L1540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcmpgtsp", [llvm_v4i32_ty],`.
  **L1540 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcmpgtsp", [llvm_v4i32_ty],`。

### Lines 1541-1568

````tablegen
                            [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcmpgtsp_p : ClangBuiltin<"__builtin_vsx_xvcmpgtsp_p">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],
                          [llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xxleqv :
      PowerPC_VSX_Intrinsic<"xxleqv", [llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;
def int_ppc_vsx_xviexpdp :
      PowerPC_VSX_Intrinsic<"xviexpdp",[llvm_v2f64_ty],
                            [llvm_v2i64_ty, llvm_v2i64_ty],[IntrNoMem]>;
def int_ppc_vsx_xviexpsp :
      PowerPC_VSX_Intrinsic<"xviexpsp",[llvm_v4f32_ty],
                            [llvm_v4i32_ty, llvm_v4i32_ty],[IntrNoMem]>;
def int_ppc_vsx_xvcvdpsxws :
      PowerPC_VSX_Intrinsic<"xvcvdpsxws", [llvm_v4i32_ty],
                            [llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvdpuxws :
      PowerPC_VSX_Intrinsic<"xvcvdpuxws", [llvm_v4i32_ty],
                            [llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvspsxds :
      PowerPC_VSX_Intrinsic<"xvcvspsxds", [llvm_v2i64_ty],
                            [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvspuxds :
      PowerPC_VSX_Intrinsic<"xvcvspuxds", [llvm_v2i64_ty],
                            [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvsxwdp :
      PowerPC_VSX_Intrinsic<"xvcvsxwdp", [llvm_v2f64_ty],
````
- **L1541 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1541 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1542 EN**: Declares TableGen def `int_ppc_vsx_xvcmpgtsp_p`.
  **L1542 CN**: 声明 TableGen def `int_ppc_vsx_xvcmpgtsp_p`。
- **L1543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L1543 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L1544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`.
  **L1544 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty,llvm_v4f32_ty,llvm_v4f32_ty],`。
- **L1545 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1545 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1546 EN**: Declares TableGen def `int_ppc_vsx_xxleqv`.
  **L1546 CN**: 声明 TableGen def `int_ppc_vsx_xxleqv`。
- **L1547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxleqv", [llvm_v4i32_ty],`.
  **L1547 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxleqv", [llvm_v4i32_ty],`。
- **L1548 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1548 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty, llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1549 EN**: Declares TableGen def `int_ppc_vsx_xviexpdp`.
  **L1549 CN**: 声明 TableGen def `int_ppc_vsx_xviexpdp`。
- **L1550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xviexpdp",[llvm_v2f64_ty],`.
  **L1550 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xviexpdp",[llvm_v2f64_ty],`。
- **L1551 EN**: Executes a standalone statement or declaration: `[llvm_v2i64_ty, llvm_v2i64_ty],[IntrNoMem]>;`.
  **L1551 CN**: 执行一条独立语句或声明：`[llvm_v2i64_ty, llvm_v2i64_ty],[IntrNoMem]>;`。
- **L1552 EN**: Declares TableGen def `int_ppc_vsx_xviexpsp`.
  **L1552 CN**: 声明 TableGen def `int_ppc_vsx_xviexpsp`。
- **L1553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xviexpsp",[llvm_v4f32_ty],`.
  **L1553 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xviexpsp",[llvm_v4f32_ty],`。
- **L1554 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty, llvm_v4i32_ty],[IntrNoMem]>;`.
  **L1554 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty, llvm_v4i32_ty],[IntrNoMem]>;`。
- **L1555 EN**: Declares TableGen def `int_ppc_vsx_xvcvdpsxws`.
  **L1555 CN**: 声明 TableGen def `int_ppc_vsx_xvcvdpsxws`。
- **L1556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvdpsxws", [llvm_v4i32_ty],`.
  **L1556 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvdpsxws", [llvm_v4i32_ty],`。
- **L1557 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1557 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1558 EN**: Declares TableGen def `int_ppc_vsx_xvcvdpuxws`.
  **L1558 CN**: 声明 TableGen def `int_ppc_vsx_xvcvdpuxws`。
- **L1559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvdpuxws", [llvm_v4i32_ty],`.
  **L1559 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvdpuxws", [llvm_v4i32_ty],`。
- **L1560 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1560 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1561 EN**: Declares TableGen def `int_ppc_vsx_xvcvspsxds`.
  **L1561 CN**: 声明 TableGen def `int_ppc_vsx_xvcvspsxds`。
- **L1562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvspsxds", [llvm_v2i64_ty],`.
  **L1562 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvspsxds", [llvm_v2i64_ty],`。
- **L1563 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1563 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1564 EN**: Declares TableGen def `int_ppc_vsx_xvcvspuxds`.
  **L1564 CN**: 声明 TableGen def `int_ppc_vsx_xvcvspuxds`。
- **L1565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvspuxds", [llvm_v2i64_ty],`.
  **L1565 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvspuxds", [llvm_v2i64_ty],`。
- **L1566 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1566 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1567 EN**: Declares TableGen def `int_ppc_vsx_xvcvsxwdp`.
  **L1567 CN**: 声明 TableGen def `int_ppc_vsx_xvcvsxwdp`。
- **L1568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvsxwdp", [llvm_v2f64_ty],`.
  **L1568 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvsxwdp", [llvm_v2f64_ty],`。

### Lines 1569-1596

````tablegen
                            [llvm_v4i32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvuxwdp :
      PowerPC_VSX_Intrinsic<"xvcvuxwdp", [llvm_v2f64_ty],
                            [llvm_v4i32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvspdp :
      PowerPC_VSX_Intrinsic<"xvcvspdp", [llvm_v2f64_ty],
                            [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvsxdsp :
      PowerPC_VSX_Intrinsic<"xvcvsxdsp", [llvm_v4f32_ty],
                            [llvm_v2i64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvuxdsp :
      PowerPC_VSX_Intrinsic<"xvcvuxdsp", [llvm_v4f32_ty],
                            [llvm_v2i64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvdpsp :
      PowerPC_VSX_Intrinsic<"xvcvdpsp", [llvm_v4f32_ty],
                            [llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvsphp :
      PowerPC_VSX_Intrinsic<"xvcvsphp", [llvm_v4f32_ty],
                            [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvxexpdp :
      PowerPC_VSX_Intrinsic<"xvxexpdp", [llvm_v2i64_ty],
                            [llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvxexpsp :
      PowerPC_VSX_Intrinsic<"xvxexpsp", [llvm_v4i32_ty],
                            [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvxsigdp :
      PowerPC_VSX_Intrinsic<"xvxsigdp", [llvm_v2i64_ty],
                            [llvm_v2f64_ty], [IntrNoMem]>;
````
- **L1569 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1569 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1570 EN**: Declares TableGen def `int_ppc_vsx_xvcvuxwdp`.
  **L1570 CN**: 声明 TableGen def `int_ppc_vsx_xvcvuxwdp`。
- **L1571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvuxwdp", [llvm_v2f64_ty],`.
  **L1571 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvuxwdp", [llvm_v2f64_ty],`。
- **L1572 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty], [IntrNoMem]>;`.
  **L1572 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty], [IntrNoMem]>;`。
- **L1573 EN**: Declares TableGen def `int_ppc_vsx_xvcvspdp`.
  **L1573 CN**: 声明 TableGen def `int_ppc_vsx_xvcvspdp`。
- **L1574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvspdp", [llvm_v2f64_ty],`.
  **L1574 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvspdp", [llvm_v2f64_ty],`。
- **L1575 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1575 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1576 EN**: Declares TableGen def `int_ppc_vsx_xvcvsxdsp`.
  **L1576 CN**: 声明 TableGen def `int_ppc_vsx_xvcvsxdsp`。
- **L1577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvsxdsp", [llvm_v4f32_ty],`.
  **L1577 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvsxdsp", [llvm_v4f32_ty],`。
- **L1578 EN**: Executes a standalone statement or declaration: `[llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1578 CN**: 执行一条独立语句或声明：`[llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1579 EN**: Declares TableGen def `int_ppc_vsx_xvcvuxdsp`.
  **L1579 CN**: 声明 TableGen def `int_ppc_vsx_xvcvuxdsp`。
- **L1580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvuxdsp", [llvm_v4f32_ty],`.
  **L1580 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvuxdsp", [llvm_v4f32_ty],`。
- **L1581 EN**: Executes a standalone statement or declaration: `[llvm_v2i64_ty], [IntrNoMem]>;`.
  **L1581 CN**: 执行一条独立语句或声明：`[llvm_v2i64_ty], [IntrNoMem]>;`。
- **L1582 EN**: Declares TableGen def `int_ppc_vsx_xvcvdpsp`.
  **L1582 CN**: 声明 TableGen def `int_ppc_vsx_xvcvdpsp`。
- **L1583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvdpsp", [llvm_v4f32_ty],`.
  **L1583 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvdpsp", [llvm_v4f32_ty],`。
- **L1584 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1584 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1585 EN**: Declares TableGen def `int_ppc_vsx_xvcvsphp`.
  **L1585 CN**: 声明 TableGen def `int_ppc_vsx_xvcvsphp`。
- **L1586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvsphp", [llvm_v4f32_ty],`.
  **L1586 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvsphp", [llvm_v4f32_ty],`。
- **L1587 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1587 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1588 EN**: Declares TableGen def `int_ppc_vsx_xvxexpdp`.
  **L1588 CN**: 声明 TableGen def `int_ppc_vsx_xvxexpdp`。
- **L1589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvxexpdp", [llvm_v2i64_ty],`.
  **L1589 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvxexpdp", [llvm_v2i64_ty],`。
- **L1590 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1590 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1591 EN**: Declares TableGen def `int_ppc_vsx_xvxexpsp`.
  **L1591 CN**: 声明 TableGen def `int_ppc_vsx_xvxexpsp`。
- **L1592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvxexpsp", [llvm_v4i32_ty],`.
  **L1592 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvxexpsp", [llvm_v4i32_ty],`。
- **L1593 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1593 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1594 EN**: Declares TableGen def `int_ppc_vsx_xvxsigdp`.
  **L1594 CN**: 声明 TableGen def `int_ppc_vsx_xvxsigdp`。
- **L1595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvxsigdp", [llvm_v2i64_ty],`.
  **L1595 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvxsigdp", [llvm_v2i64_ty],`。
- **L1596 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1596 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty], [IntrNoMem]>;`。

### Lines 1597-1624

````tablegen
def int_ppc_vsx_xvxsigsp :
      PowerPC_VSX_Intrinsic<"xvxsigsp", [llvm_v4i32_ty],
                            [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvtstdcdp :
      PowerPC_VSX_Intrinsic<"xvtstdcdp", [llvm_v2i64_ty],
                            [llvm_v2f64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_ppc_vsx_xvtstdcsp :
      PowerPC_VSX_Intrinsic<"xvtstdcsp", [llvm_v4i32_ty],
                            [llvm_v4f32_ty,llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;
def int_ppc_vsx_xvcvhpsp :
      PowerPC_VSX_Intrinsic<"xvcvhpsp", [llvm_v4f32_ty],
                            [llvm_v8i16_ty],[IntrNoMem]>;
def int_ppc_vsx_xvcvspbf16 :
      PowerPC_VSX_Intrinsic<"xvcvspbf16", [llvm_v16i8_ty],
                            [llvm_v16i8_ty], [IntrNoMem]>;
def int_ppc_vsx_xvcvbf16spn :
      PowerPC_VSX_Intrinsic<"xvcvbf16spn", [llvm_v16i8_ty],
                            [llvm_v16i8_ty], [IntrNoMem]>;
def int_ppc_vsx_xxextractuw :
      PowerPC_VSX_Intrinsic<"xxextractuw",[llvm_v2i64_ty],
                            [llvm_v2i64_ty,llvm_i32_ty], [IntrNoMem]>;
def int_ppc_vsx_xxinsertw :
      PowerPC_VSX_Intrinsic<"xxinsertw",[llvm_v4i32_ty],
                            [llvm_v4i32_ty,llvm_v2i64_ty,llvm_i32_ty],
                            [IntrNoMem]>;
def int_ppc_vsx_xvtlsbb :
      PowerPC_VSX_Intrinsic<"xvtlsbb", [llvm_i32_ty],
                            [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;
````
- **L1597 EN**: Declares TableGen def `int_ppc_vsx_xvxsigsp`.
  **L1597 CN**: 声明 TableGen def `int_ppc_vsx_xvxsigsp`。
- **L1598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvxsigsp", [llvm_v4i32_ty],`.
  **L1598 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvxsigsp", [llvm_v4i32_ty],`。
- **L1599 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1599 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1600 EN**: Declares TableGen def `int_ppc_vsx_xvtstdcdp`.
  **L1600 CN**: 声明 TableGen def `int_ppc_vsx_xvtstdcdp`。
- **L1601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvtstdcdp", [llvm_v2i64_ty],`.
  **L1601 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvtstdcdp", [llvm_v2i64_ty],`。
- **L1602 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1602 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1603 EN**: Declares TableGen def `int_ppc_vsx_xvtstdcsp`.
  **L1603 CN**: 声明 TableGen def `int_ppc_vsx_xvtstdcsp`。
- **L1604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvtstdcsp", [llvm_v4i32_ty],`.
  **L1604 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvtstdcsp", [llvm_v4i32_ty],`。
- **L1605 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty,llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1605 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty,llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1606 EN**: Declares TableGen def `int_ppc_vsx_xvcvhpsp`.
  **L1606 CN**: 声明 TableGen def `int_ppc_vsx_xvcvhpsp`。
- **L1607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvhpsp", [llvm_v4f32_ty],`.
  **L1607 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvhpsp", [llvm_v4f32_ty],`。
- **L1608 EN**: Executes a standalone statement or declaration: `[llvm_v8i16_ty],[IntrNoMem]>;`.
  **L1608 CN**: 执行一条独立语句或声明：`[llvm_v8i16_ty],[IntrNoMem]>;`。
- **L1609 EN**: Declares TableGen def `int_ppc_vsx_xvcvspbf16`.
  **L1609 CN**: 声明 TableGen def `int_ppc_vsx_xvcvspbf16`。
- **L1610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvspbf16", [llvm_v16i8_ty],`.
  **L1610 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvspbf16", [llvm_v16i8_ty],`。
- **L1611 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1611 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1612 EN**: Declares TableGen def `int_ppc_vsx_xvcvbf16spn`.
  **L1612 CN**: 声明 TableGen def `int_ppc_vsx_xvcvbf16spn`。
- **L1613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvcvbf16spn", [llvm_v16i8_ty],`.
  **L1613 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvcvbf16spn", [llvm_v16i8_ty],`。
- **L1614 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1614 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1615 EN**: Declares TableGen def `int_ppc_vsx_xxextractuw`.
  **L1615 CN**: 声明 TableGen def `int_ppc_vsx_xxextractuw`。
- **L1616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxextractuw",[llvm_v2i64_ty],`.
  **L1616 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxextractuw",[llvm_v2i64_ty],`。
- **L1617 EN**: Executes a standalone statement or declaration: `[llvm_v2i64_ty,llvm_i32_ty], [IntrNoMem]>;`.
  **L1617 CN**: 执行一条独立语句或声明：`[llvm_v2i64_ty,llvm_i32_ty], [IntrNoMem]>;`。
- **L1618 EN**: Declares TableGen def `int_ppc_vsx_xxinsertw`.
  **L1618 CN**: 声明 TableGen def `int_ppc_vsx_xxinsertw`。
- **L1619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxinsertw",[llvm_v4i32_ty],`.
  **L1619 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxinsertw",[llvm_v4i32_ty],`。
- **L1620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty,llvm_v2i64_ty,llvm_i32_ty],`.
  **L1620 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty,llvm_v2i64_ty,llvm_i32_ty],`。
- **L1621 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1621 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1622 EN**: Declares TableGen def `int_ppc_vsx_xvtlsbb`.
  **L1622 CN**: 声明 TableGen def `int_ppc_vsx_xvtlsbb`。
- **L1623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvtlsbb", [llvm_i32_ty],`.
  **L1623 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvtlsbb", [llvm_i32_ty],`。
- **L1624 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1624 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`。

### Lines 1625-1652

````tablegen
def int_ppc_vsx_xvtdivdp :
      PowerPC_VSX_Intrinsic<"xvtdivdp", [llvm_i32_ty],
                            [llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvtdivsp :
      PowerPC_VSX_Intrinsic<"xvtdivsp", [llvm_i32_ty],
                            [llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xvtsqrtdp :
      PowerPC_VSX_Intrinsic<"xvtsqrtdp", [llvm_i32_ty], [llvm_v2f64_ty], [IntrNoMem]>;
def int_ppc_vsx_xvtsqrtsp :
      PowerPC_VSX_Intrinsic<"xvtsqrtsp", [llvm_i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;
def int_ppc_vsx_xxeval :
      PowerPC_VSX_Intrinsic<"xxeval", [llvm_v2i64_ty],
                           [llvm_v2i64_ty, llvm_v2i64_ty,
                            llvm_v2i64_ty, llvm_i32_ty],
                           [IntrNoMem, ImmArg<ArgIndex<3>>]>;
def int_ppc_vsx_xxgenpcvbm :
      PowerPC_VSX_Intrinsic<"xxgenpcvbm", [llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;
def int_ppc_vsx_xxgenpcvhm :
      PowerPC_VSX_Intrinsic<"xxgenpcvhm", [llvm_v8i16_ty],
                            [llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;
def int_ppc_vsx_xxgenpcvwm :
      PowerPC_VSX_Intrinsic<"xxgenpcvwm", [llvm_v4i32_ty],
                            [llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;
def int_ppc_vsx_xxgenpcvdm :
      PowerPC_VSX_Intrinsic<"xxgenpcvdm", [llvm_v2i64_ty],
                            [llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;

````
- **L1625 EN**: Declares TableGen def `int_ppc_vsx_xvtdivdp`.
  **L1625 CN**: 声明 TableGen def `int_ppc_vsx_xvtdivdp`。
- **L1626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvtdivdp", [llvm_i32_ty],`.
  **L1626 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvtdivdp", [llvm_i32_ty],`。
- **L1627 EN**: Executes a standalone statement or declaration: `[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1627 CN**: 执行一条独立语句或声明：`[llvm_v2f64_ty, llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1628 EN**: Declares TableGen def `int_ppc_vsx_xvtdivsp`.
  **L1628 CN**: 声明 TableGen def `int_ppc_vsx_xvtdivsp`。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xvtdivsp", [llvm_i32_ty],`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xvtdivsp", [llvm_i32_ty],`。
- **L1630 EN**: Executes a standalone statement or declaration: `[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1630 CN**: 执行一条独立语句或声明：`[llvm_v4f32_ty, llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1631 EN**: Declares TableGen def `int_ppc_vsx_xvtsqrtdp`.
  **L1631 CN**: 声明 TableGen def `int_ppc_vsx_xvtsqrtdp`。
- **L1632 EN**: Executes a standalone statement or declaration: `PowerPC_VSX_Intrinsic<"xvtsqrtdp", [llvm_i32_ty], [llvm_v2f64_ty], [IntrNoMem]>;`.
  **L1632 CN**: 执行一条独立语句或声明：`PowerPC_VSX_Intrinsic<"xvtsqrtdp", [llvm_i32_ty], [llvm_v2f64_ty], [IntrNoMem]>;`。
- **L1633 EN**: Declares TableGen def `int_ppc_vsx_xvtsqrtsp`.
  **L1633 CN**: 声明 TableGen def `int_ppc_vsx_xvtsqrtsp`。
- **L1634 EN**: Executes a standalone statement or declaration: `PowerPC_VSX_Intrinsic<"xvtsqrtsp", [llvm_i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`.
  **L1634 CN**: 执行一条独立语句或声明：`PowerPC_VSX_Intrinsic<"xvtsqrtsp", [llvm_i32_ty], [llvm_v4f32_ty], [IntrNoMem]>;`。
- **L1635 EN**: Declares TableGen def `int_ppc_vsx_xxeval`.
  **L1635 CN**: 声明 TableGen def `int_ppc_vsx_xxeval`。
- **L1636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxeval", [llvm_v2i64_ty],`.
  **L1636 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxeval", [llvm_v2i64_ty],`。
- **L1637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty,`.
  **L1637 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty,`。
- **L1638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v2i64_ty, llvm_i32_ty],`.
  **L1638 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v2i64_ty, llvm_i32_ty],`。
- **L1639 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L1639 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L1640 EN**: Declares TableGen def `int_ppc_vsx_xxgenpcvbm`.
  **L1640 CN**: 声明 TableGen def `int_ppc_vsx_xxgenpcvbm`。
- **L1641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxgenpcvbm", [llvm_v16i8_ty],`.
  **L1641 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxgenpcvbm", [llvm_v16i8_ty],`。
- **L1642 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1642 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1643 EN**: Declares TableGen def `int_ppc_vsx_xxgenpcvhm`.
  **L1643 CN**: 声明 TableGen def `int_ppc_vsx_xxgenpcvhm`。
- **L1644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxgenpcvhm", [llvm_v8i16_ty],`.
  **L1644 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxgenpcvhm", [llvm_v8i16_ty],`。
- **L1645 EN**: Executes a standalone statement or declaration: `[llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1645 CN**: 执行一条独立语句或声明：`[llvm_v8i16_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1646 EN**: Declares TableGen def `int_ppc_vsx_xxgenpcvwm`.
  **L1646 CN**: 声明 TableGen def `int_ppc_vsx_xxgenpcvwm`。
- **L1647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxgenpcvwm", [llvm_v4i32_ty],`.
  **L1647 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxgenpcvwm", [llvm_v4i32_ty],`。
- **L1648 EN**: Executes a standalone statement or declaration: `[llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1648 CN**: 执行一条独立语句或声明：`[llvm_v4i32_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1649 EN**: Declares TableGen def `int_ppc_vsx_xxgenpcvdm`.
  **L1649 CN**: 声明 TableGen def `int_ppc_vsx_xxgenpcvdm`。
- **L1650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_VSX_Intrinsic<"xxgenpcvdm", [llvm_v2i64_ty],`.
  **L1650 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_VSX_Intrinsic<"xxgenpcvdm", [llvm_v2i64_ty],`。
- **L1651 EN**: Executes a standalone statement or declaration: `[llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1651 CN**: 执行一条独立语句或声明：`[llvm_v2i64_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1652 EN**: Blank line separating nearby declarations or logic blocks.
  **L1652 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1653-1680

````tablegen
// P10 VSX Vector permute extended.
def int_ppc_vsx_xxpermx : 
    ClangBuiltin<"__builtin_vsx_xxpermx">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                          [llvm_v16i8_ty,llvm_v16i8_ty,llvm_v16i8_ty,
                           llvm_i32_ty],
                          [IntrNoMem, ImmArg<ArgIndex<3>>]>;
// P10 VSX Vector Blend Variable.
def int_ppc_vsx_xxblendvb: ClangBuiltin<"__builtin_vsx_xxblendvb">,
    DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                          [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xxblendvh: ClangBuiltin<"__builtin_vsx_xxblendvh">,
    DefaultAttrsIntrinsic<[llvm_v8i16_ty],
                          [llvm_v8i16_ty, llvm_v8i16_ty,llvm_v8i16_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xxblendvw: ClangBuiltin<"__builtin_vsx_xxblendvw">,
    DefaultAttrsIntrinsic<[llvm_v4i32_ty],
                          [llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],
                          [IntrNoMem]>;
def int_ppc_vsx_xxblendvd: ClangBuiltin<"__builtin_vsx_xxblendvd">,
    DefaultAttrsIntrinsic<[llvm_v2i64_ty],
                          [llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],
                          [IntrNoMem]>;
}

//===----------------------------------------------------------------------===//
// PowerPC HTM Intrinsic Definitions.
````
- **L1653 EN**: Comment explains nearby logic, invariants, or intent: `P10 VSX Vector permute extended.`.
  **L1653 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 VSX Vector permute extended.`。
- **L1654 EN**: Declares TableGen def `int_ppc_vsx_xxpermx`.
  **L1654 CN**: 声明 TableGen def `int_ppc_vsx_xxpermx`。
- **L1655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_vsx_xxpermx">,`.
  **L1655 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_vsx_xxpermx">,`。
- **L1656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L1656 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L1657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty,llvm_v16i8_ty,llvm_v16i8_ty,`.
  **L1657 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty,llvm_v16i8_ty,llvm_v16i8_ty,`。
- **L1658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L1658 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L1659 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L1659 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L1660 EN**: Comment explains nearby logic, invariants, or intent: `P10 VSX Vector Blend Variable.`.
  **L1660 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`P10 VSX Vector Blend Variable.`。
- **L1661 EN**: Declares TableGen def `int_ppc_vsx_xxblendvb`.
  **L1661 CN**: 声明 TableGen def `int_ppc_vsx_xxblendvb`。
- **L1662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L1662 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L1663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1663 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1664 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1664 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1665 EN**: Declares TableGen def `int_ppc_vsx_xxblendvh`.
  **L1665 CN**: 声明 TableGen def `int_ppc_vsx_xxblendvh`。
- **L1666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v8i16_ty],`.
  **L1666 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v8i16_ty],`。
- **L1667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_v8i16_ty,llvm_v8i16_ty],`.
  **L1667 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_v8i16_ty,llvm_v8i16_ty],`。
- **L1668 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1668 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1669 EN**: Declares TableGen def `int_ppc_vsx_xxblendvw`.
  **L1669 CN**: 声明 TableGen def `int_ppc_vsx_xxblendvw`。
- **L1670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v4i32_ty],`.
  **L1670 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v4i32_ty],`。
- **L1671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`.
  **L1671 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4i32_ty, llvm_v4i32_ty, llvm_v4i32_ty],`。
- **L1672 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1672 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1673 EN**: Declares TableGen def `int_ppc_vsx_xxblendvd`.
  **L1673 CN**: 声明 TableGen def `int_ppc_vsx_xxblendvd`。
- **L1674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2i64_ty],`.
  **L1674 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2i64_ty],`。
- **L1675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`.
  **L1675 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_v2i64_ty],`。
- **L1676 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1676 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1677 EN**: Closes the current lexical scope or compound statement.
  **L1677 CN**: 结束当前词法作用域或复合语句块。
- **L1678 EN**: Blank line separating nearby declarations or logic blocks.
  **L1678 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1679 EN**: Banner comment marking a file or section boundary.
  **L1679 CN**: 横幅注释，用于标记文件或章节边界。
- **L1680 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC HTM Intrinsic Definitions.`.
  **L1680 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC HTM Intrinsic Definitions.`。

### Lines 1681-1708

````tablegen

let TargetPrefix = "ppc" in {  // All intrinsics start with "llvm.ppc.".

def int_ppc_tbegin : ClangBuiltin<"__builtin_tbegin">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;
def int_ppc_tend : ClangBuiltin<"__builtin_tend">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

def int_ppc_tabort : ClangBuiltin<"__builtin_tabort">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;
def int_ppc_tabortwc : ClangBuiltin<"__builtin_tabortwc">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;
def int_ppc_tabortwci : ClangBuiltin<"__builtin_tabortwci">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;
def int_ppc_tabortdc : ClangBuiltin<"__builtin_tabortdc">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;
def int_ppc_tabortdci : ClangBuiltin<"__builtin_tabortdci">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;

def int_ppc_tcheck : ClangBuiltin<"__builtin_tcheck">,
      Intrinsic<[llvm_i32_ty], [], []>;
def int_ppc_treclaim : ClangBuiltin<"__builtin_treclaim">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;
def int_ppc_trechkpt : ClangBuiltin<"__builtin_trechkpt">,
      Intrinsic<[llvm_i32_ty], [], []>;
def int_ppc_tsr : ClangBuiltin<"__builtin_tsr">,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;

````
- **L1681 EN**: Blank line separating nearby declarations or logic blocks.
  **L1681 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1682 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1682 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1683 EN**: Blank line separating nearby declarations or logic blocks.
  **L1683 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1684 EN**: Declares TableGen def `int_ppc_tbegin`.
  **L1684 CN**: 声明 TableGen def `int_ppc_tbegin`。
- **L1685 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L1685 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L1686 EN**: Declares TableGen def `int_ppc_tend`.
  **L1686 CN**: 声明 TableGen def `int_ppc_tend`。
- **L1687 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L1687 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L1688 EN**: Blank line separating nearby declarations or logic blocks.
  **L1688 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1689 EN**: Declares TableGen def `int_ppc_tabort`.
  **L1689 CN**: 声明 TableGen def `int_ppc_tabort`。
- **L1690 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;`.
  **L1690 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;`。
- **L1691 EN**: Declares TableGen def `int_ppc_tabortwc`.
  **L1691 CN**: 声明 TableGen def `int_ppc_tabortwc`。
- **L1692 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`.
  **L1692 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`。
- **L1693 EN**: Declares TableGen def `int_ppc_tabortwci`.
  **L1693 CN**: 声明 TableGen def `int_ppc_tabortwci`。
- **L1694 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`.
  **L1694 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`。
- **L1695 EN**: Declares TableGen def `int_ppc_tabortdc`.
  **L1695 CN**: 声明 TableGen def `int_ppc_tabortdc`。
- **L1696 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`.
  **L1696 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`。
- **L1697 EN**: Declares TableGen def `int_ppc_tabortdci`.
  **L1697 CN**: 声明 TableGen def `int_ppc_tabortdci`。
- **L1698 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`.
  **L1698 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty], []>;`。
- **L1699 EN**: Blank line separating nearby declarations or logic blocks.
  **L1699 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1700 EN**: Declares TableGen def `int_ppc_tcheck`.
  **L1700 CN**: 声明 TableGen def `int_ppc_tcheck`。
- **L1701 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], []>;`.
  **L1701 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], []>;`。
- **L1702 EN**: Declares TableGen def `int_ppc_treclaim`.
  **L1702 CN**: 声明 TableGen def `int_ppc_treclaim`。
- **L1703 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;`.
  **L1703 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;`。
- **L1704 EN**: Declares TableGen def `int_ppc_trechkpt`.
  **L1704 CN**: 声明 TableGen def `int_ppc_trechkpt`。
- **L1705 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], []>;`.
  **L1705 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], []>;`。
- **L1706 EN**: Declares TableGen def `int_ppc_tsr`.
  **L1706 CN**: 声明 TableGen def `int_ppc_tsr`。
- **L1707 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;`.
  **L1707 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty], []>;`。
- **L1708 EN**: Blank line separating nearby declarations or logic blocks.
  **L1708 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1709-1736

````tablegen
def int_ppc_get_texasr : ClangBuiltin<"__builtin_get_texasr">,
      Intrinsic<[llvm_i64_ty], [], []>;
def int_ppc_get_texasru : ClangBuiltin<"__builtin_get_texasru">,
      Intrinsic<[llvm_i64_ty], [], []>;
def int_ppc_get_tfhar : ClangBuiltin<"__builtin_get_tfhar">,
      Intrinsic<[llvm_i64_ty], [], []>;
def int_ppc_get_tfiar : ClangBuiltin<"__builtin_get_tfiar">,
      Intrinsic<[llvm_i64_ty], [], []>;

def int_ppc_set_texasr : ClangBuiltin<"__builtin_set_texasr">,
      Intrinsic<[], [llvm_i64_ty], []>;
def int_ppc_set_texasru : ClangBuiltin<"__builtin_set_texasru">,
      Intrinsic<[], [llvm_i64_ty], []>;
def int_ppc_set_tfhar : ClangBuiltin<"__builtin_set_tfhar">,
      Intrinsic<[], [llvm_i64_ty], []>;
def int_ppc_set_tfiar : ClangBuiltin<"__builtin_set_tfiar">,
      Intrinsic<[], [llvm_i64_ty], []>;

// Extended mnemonics
def int_ppc_tendall : ClangBuiltin<"__builtin_tendall">,
      Intrinsic<[llvm_i32_ty], [], []>;
def int_ppc_tresume : ClangBuiltin<"__builtin_tresume">,
      Intrinsic<[llvm_i32_ty], [], []>;
def int_ppc_tsuspend : ClangBuiltin<"__builtin_tsuspend">,
      Intrinsic<[llvm_i32_ty], [], []>;

def int_ppc_ttest : ClangBuiltin<"__builtin_ttest">,
      Intrinsic<[llvm_i64_ty], [], []>;
````
- **L1709 EN**: Declares TableGen def `int_ppc_get_texasr`.
  **L1709 CN**: 声明 TableGen def `int_ppc_get_texasr`。
- **L1710 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [], []>;`.
  **L1710 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [], []>;`。
- **L1711 EN**: Declares TableGen def `int_ppc_get_texasru`.
  **L1711 CN**: 声明 TableGen def `int_ppc_get_texasru`。
- **L1712 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [], []>;`.
  **L1712 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [], []>;`。
- **L1713 EN**: Declares TableGen def `int_ppc_get_tfhar`.
  **L1713 CN**: 声明 TableGen def `int_ppc_get_tfhar`。
- **L1714 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [], []>;`.
  **L1714 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [], []>;`。
- **L1715 EN**: Declares TableGen def `int_ppc_get_tfiar`.
  **L1715 CN**: 声明 TableGen def `int_ppc_get_tfiar`。
- **L1716 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [], []>;`.
  **L1716 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [], []>;`。
- **L1717 EN**: Blank line separating nearby declarations or logic blocks.
  **L1717 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1718 EN**: Declares TableGen def `int_ppc_set_texasr`.
  **L1718 CN**: 声明 TableGen def `int_ppc_set_texasr`。
- **L1719 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty], []>;`.
  **L1719 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty], []>;`。
- **L1720 EN**: Declares TableGen def `int_ppc_set_texasru`.
  **L1720 CN**: 声明 TableGen def `int_ppc_set_texasru`。
- **L1721 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty], []>;`.
  **L1721 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty], []>;`。
- **L1722 EN**: Declares TableGen def `int_ppc_set_tfhar`.
  **L1722 CN**: 声明 TableGen def `int_ppc_set_tfhar`。
- **L1723 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty], []>;`.
  **L1723 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty], []>;`。
- **L1724 EN**: Declares TableGen def `int_ppc_set_tfiar`.
  **L1724 CN**: 声明 TableGen def `int_ppc_set_tfiar`。
- **L1725 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty], []>;`.
  **L1725 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty], []>;`。
- **L1726 EN**: Blank line separating nearby declarations or logic blocks.
  **L1726 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1727 EN**: Comment explains nearby logic, invariants, or intent: `Extended mnemonics`.
  **L1727 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Extended mnemonics`。
- **L1728 EN**: Declares TableGen def `int_ppc_tendall`.
  **L1728 CN**: 声明 TableGen def `int_ppc_tendall`。
- **L1729 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], []>;`.
  **L1729 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], []>;`。
- **L1730 EN**: Declares TableGen def `int_ppc_tresume`.
  **L1730 CN**: 声明 TableGen def `int_ppc_tresume`。
- **L1731 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], []>;`.
  **L1731 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], []>;`。
- **L1732 EN**: Declares TableGen def `int_ppc_tsuspend`.
  **L1732 CN**: 声明 TableGen def `int_ppc_tsuspend`。
- **L1733 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], []>;`.
  **L1733 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], []>;`。
- **L1734 EN**: Blank line separating nearby declarations or logic blocks.
  **L1734 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1735 EN**: Declares TableGen def `int_ppc_ttest`.
  **L1735 CN**: 声明 TableGen def `int_ppc_ttest`。
- **L1736 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [], []>;`.
  **L1736 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [], []>;`。

### Lines 1737-1764

````tablegen

// We currently use llvm.ppc.cfence in the context of atomic load which
// in LLVM IR requires its type to be one of integer, pointer and
// float point type. So llvm_any_ty here refers to type mentioned above.
// Backend is supposed to lower these types to appropriate MVTs.
def int_ppc_cfence : Intrinsic<[], [llvm_any_ty], []>;

// PowerPC set FPSCR Intrinsic Definitions.
def int_ppc_setrnd : ClangBuiltin<"__builtin_setrnd">,
      DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_i32_ty], [IntrHasSideEffects]>;
}

let TargetPrefix = "ppc" in {
  def int_ppc_vsx_assemble_pair :
      DefaultAttrsIntrinsic<[llvm_v256i1_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;

  def int_ppc_vsx_disassemble_pair :
      DefaultAttrsIntrinsic<[llvm_v16i8_ty, llvm_v16i8_ty],
                            [llvm_v256i1_ty], [IntrNoMem]>;

  // AES Encrypt Paired Instructions.
  def int_ppc_aes_encrypt_paired :
      DefaultAttrsIntrinsic<[llvm_v256i1_ty],
                            [llvm_v256i1_ty, llvm_v256i1_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  // AES Decrypt Paired Instructions.
  def int_ppc_aes_decrypt_paired :
````
- **L1737 EN**: Blank line separating nearby declarations or logic blocks.
  **L1737 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1738 EN**: Comment explains nearby logic, invariants, or intent: `We currently use llvm.ppc.cfence in the context of atomic load which`.
  **L1738 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`We currently use llvm.ppc.cfence in the context of atomic load which`。
- **L1739 EN**: Comment explains nearby logic, invariants, or intent: `in LLVM IR requires its type to be one of integer, pointer and`.
  **L1739 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in LLVM IR requires its type to be one of integer, pointer and`。
- **L1740 EN**: Comment explains nearby logic, invariants, or intent: `float point type. So llvm_any_ty here refers to type mentioned above.`.
  **L1740 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`float point type. So llvm_any_ty here refers to type mentioned above.`。
- **L1741 EN**: Comment explains nearby logic, invariants, or intent: `Backend is supposed to lower these types to appropriate MVTs.`.
  **L1741 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Backend is supposed to lower these types to appropriate MVTs.`。
- **L1742 EN**: Declares TableGen def `int_ppc_cfence`.
  **L1742 CN**: 声明 TableGen def `int_ppc_cfence`。
- **L1743 EN**: Blank line separating nearby declarations or logic blocks.
  **L1743 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1744 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC set FPSCR Intrinsic Definitions.`.
  **L1744 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC set FPSCR Intrinsic Definitions.`。
- **L1745 EN**: Declares TableGen def `int_ppc_setrnd`.
  **L1745 CN**: 声明 TableGen def `int_ppc_setrnd`。
- **L1746 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_i32_ty], [IntrHasSideEffects]>;`.
  **L1746 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_i32_ty], [IntrHasSideEffects]>;`。
- **L1747 EN**: Closes the current lexical scope or compound statement.
  **L1747 CN**: 结束当前词法作用域或复合语句块。
- **L1748 EN**: Blank line separating nearby declarations or logic blocks.
  **L1748 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1749 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1749 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1750 EN**: Declares TableGen def `int_ppc_vsx_assemble_pair`.
  **L1750 CN**: 声明 TableGen def `int_ppc_vsx_assemble_pair`。
- **L1751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty],`.
  **L1751 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty],`。
- **L1752 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1752 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty, llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1753 EN**: Blank line separating nearby declarations or logic blocks.
  **L1753 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1754 EN**: Declares TableGen def `int_ppc_vsx_disassemble_pair`.
  **L1754 CN**: 声明 TableGen def `int_ppc_vsx_disassemble_pair`。
- **L1755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1755 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1756 EN**: Executes a standalone statement or declaration: `[llvm_v256i1_ty], [IntrNoMem]>;`.
  **L1756 CN**: 执行一条独立语句或声明：`[llvm_v256i1_ty], [IntrNoMem]>;`。
- **L1757 EN**: Blank line separating nearby declarations or logic blocks.
  **L1757 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1758 EN**: Comment explains nearby logic, invariants, or intent: `AES Encrypt Paired Instructions.`.
  **L1758 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AES Encrypt Paired Instructions.`。
- **L1759 EN**: Declares TableGen def `int_ppc_aes_encrypt_paired`.
  **L1759 CN**: 声明 TableGen def `int_ppc_aes_encrypt_paired`。
- **L1760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty],`.
  **L1760 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty],`。
- **L1761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v256i1_ty, llvm_v256i1_ty, llvm_i32_ty],`.
  **L1761 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v256i1_ty, llvm_v256i1_ty, llvm_i32_ty],`。
- **L1762 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1762 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1763 EN**: Comment explains nearby logic, invariants, or intent: `AES Decrypt Paired Instructions.`.
  **L1763 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AES Decrypt Paired Instructions.`。
- **L1764 EN**: Declares TableGen def `int_ppc_aes_decrypt_paired`.
  **L1764 CN**: 声明 TableGen def `int_ppc_aes_decrypt_paired`。

### Lines 1765-1792

````tablegen
      DefaultAttrsIntrinsic<[llvm_v256i1_ty],
                            [llvm_v256i1_ty, llvm_v256i1_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  // AES Generate Last Key Paired Instructions.
  def int_ppc_aes_genlastkey_paired :
      DefaultAttrsIntrinsic<[llvm_v256i1_ty],
                            [llvm_v256i1_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  // Galois Field Multiplication Instructions.
  def int_ppc_galois_field_mult :
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  def int_ppc_mma_assemble_acc :
      DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v16i8_ty], [IntrNoMem]>;

  def int_ppc_mma_disassemble_acc :
      DefaultAttrsIntrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v16i8_ty], [llvm_v512i1_ty], [IntrNoMem]>;

  def int_ppc_mma_xxmtacc :
      DefaultAttrsIntrinsic<[llvm_v512i1_ty], [llvm_v512i1_ty], [IntrNoMem]>;

  def int_ppc_mma_xxmfacc :
      DefaultAttrsIntrinsic<[llvm_v512i1_ty], [llvm_v512i1_ty], [IntrNoMem]>;
````
- **L1765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty],`.
  **L1765 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty],`。
- **L1766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v256i1_ty, llvm_v256i1_ty, llvm_i32_ty],`.
  **L1766 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v256i1_ty, llvm_v256i1_ty, llvm_i32_ty],`。
- **L1767 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1767 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1768 EN**: Comment explains nearby logic, invariants, or intent: `AES Generate Last Key Paired Instructions.`.
  **L1768 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AES Generate Last Key Paired Instructions.`。
- **L1769 EN**: Declares TableGen def `int_ppc_aes_genlastkey_paired`.
  **L1769 CN**: 声明 TableGen def `int_ppc_aes_genlastkey_paired`。
- **L1770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty],`.
  **L1770 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty],`。
- **L1771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v256i1_ty, llvm_i32_ty],`.
  **L1771 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v256i1_ty, llvm_i32_ty],`。
- **L1772 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1772 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1773 EN**: Comment explains nearby logic, invariants, or intent: `Galois Field Multiplication Instructions.`.
  **L1773 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Galois Field Multiplication Instructions.`。
- **L1774 EN**: Declares TableGen def `int_ppc_galois_field_mult`.
  **L1774 CN**: 声明 TableGen def `int_ppc_galois_field_mult`。
- **L1775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L1775 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L1776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L1776 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L1777 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1777 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1778 EN**: Blank line separating nearby declarations or logic blocks.
  **L1778 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1779 EN**: Declares TableGen def `int_ppc_mma_assemble_acc`.
  **L1779 CN**: 声明 TableGen def `int_ppc_mma_assemble_acc`。
- **L1780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v512i1_ty],`.
  **L1780 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v512i1_ty],`。
- **L1781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1781 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1782 EN**: Executes a standalone statement or declaration: `llvm_v16i8_ty], [IntrNoMem]>;`.
  **L1782 CN**: 执行一条独立语句或声明：`llvm_v16i8_ty], [IntrNoMem]>;`。
- **L1783 EN**: Blank line separating nearby declarations or logic blocks.
  **L1783 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1784 EN**: Declares TableGen def `int_ppc_mma_disassemble_acc`.
  **L1784 CN**: 声明 TableGen def `int_ppc_mma_disassemble_acc`。
- **L1785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1785 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1786 EN**: Executes a standalone statement or declaration: `llvm_v16i8_ty], [llvm_v512i1_ty], [IntrNoMem]>;`.
  **L1786 CN**: 执行一条独立语句或声明：`llvm_v16i8_ty], [llvm_v512i1_ty], [IntrNoMem]>;`。
- **L1787 EN**: Blank line separating nearby declarations or logic blocks.
  **L1787 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1788 EN**: Declares TableGen def `int_ppc_mma_xxmtacc`.
  **L1788 CN**: 声明 TableGen def `int_ppc_mma_xxmtacc`。
- **L1789 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v512i1_ty], [llvm_v512i1_ty], [IntrNoMem]>;`.
  **L1789 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v512i1_ty], [llvm_v512i1_ty], [IntrNoMem]>;`。
- **L1790 EN**: Blank line separating nearby declarations or logic blocks.
  **L1790 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1791 EN**: Declares TableGen def `int_ppc_mma_xxmfacc`.
  **L1791 CN**: 声明 TableGen def `int_ppc_mma_xxmfacc`。
- **L1792 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v512i1_ty], [llvm_v512i1_ty], [IntrNoMem]>;`.
  **L1792 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v512i1_ty], [llvm_v512i1_ty], [IntrNoMem]>;`。

### Lines 1793-1820

````tablegen

  def int_ppc_mma_xxsetaccz :
      DefaultAttrsIntrinsic<[llvm_v512i1_ty], [], [IntrNoMem]>;

  def int_ppc_dmsetdmrz :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [], [IntrNoMem]>;

  def int_ppc_dmmr :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty], [IntrNoMem]>;

  def int_ppc_dmxor :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,
                             llvm_v1024i1_ty], [IntrNoMem]>;

  def int_ppc_mma_dmxxextfdmr512 :
      DefaultAttrsIntrinsic<[llvm_v256i1_ty, llvm_v256i1_ty], [llvm_v1024i1_ty,
                             llvm_i32_ty], [IntrNoMem]>;

  def int_ppc_mma_dmxxinstdmr512 :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty, llvm_v256i1_ty,
                             llvm_v256i1_ty, llvm_i32_ty], [IntrNoMem]>;

  def int_ppc_mma_dmxxextfdmr256 :
      DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_v1024i1_ty, llvm_i32_ty],
                            [IntrNoMem]>;

  def int_ppc_mma_dmxxinstdmr256 :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty, llvm_v256i1_ty,
````
- **L1793 EN**: Blank line separating nearby declarations or logic blocks.
  **L1793 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1794 EN**: Declares TableGen def `int_ppc_mma_xxsetaccz`.
  **L1794 CN**: 声明 TableGen def `int_ppc_mma_xxsetaccz`。
- **L1795 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v512i1_ty], [], [IntrNoMem]>;`.
  **L1795 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v512i1_ty], [], [IntrNoMem]>;`。
- **L1796 EN**: Blank line separating nearby declarations or logic blocks.
  **L1796 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1797 EN**: Declares TableGen def `int_ppc_dmsetdmrz`.
  **L1797 CN**: 声明 TableGen def `int_ppc_dmsetdmrz`。
- **L1798 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [], [IntrNoMem]>;`.
  **L1798 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [], [IntrNoMem]>;`。
- **L1799 EN**: Blank line separating nearby declarations or logic blocks.
  **L1799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1800 EN**: Declares TableGen def `int_ppc_dmmr`.
  **L1800 CN**: 声明 TableGen def `int_ppc_dmmr`。
- **L1801 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty], [IntrNoMem]>;`.
  **L1801 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty], [IntrNoMem]>;`。
- **L1802 EN**: Blank line separating nearby declarations or logic blocks.
  **L1802 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1803 EN**: Declares TableGen def `int_ppc_dmxor`.
  **L1803 CN**: 声明 TableGen def `int_ppc_dmxor`。
- **L1804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,`.
  **L1804 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,`。
- **L1805 EN**: Executes a standalone statement or declaration: `llvm_v1024i1_ty], [IntrNoMem]>;`.
  **L1805 CN**: 执行一条独立语句或声明：`llvm_v1024i1_ty], [IntrNoMem]>;`。
- **L1806 EN**: Blank line separating nearby declarations or logic blocks.
  **L1806 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1807 EN**: Declares TableGen def `int_ppc_mma_dmxxextfdmr512`.
  **L1807 CN**: 声明 TableGen def `int_ppc_mma_dmxxextfdmr512`。
- **L1808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty, llvm_v256i1_ty], [llvm_v1024i1_ty,`.
  **L1808 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty, llvm_v256i1_ty], [llvm_v1024i1_ty,`。
- **L1809 EN**: Executes a standalone statement or declaration: `llvm_i32_ty], [IntrNoMem]>;`.
  **L1809 CN**: 执行一条独立语句或声明：`llvm_i32_ty], [IntrNoMem]>;`。
- **L1810 EN**: Blank line separating nearby declarations or logic blocks.
  **L1810 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1811 EN**: Declares TableGen def `int_ppc_mma_dmxxinstdmr512`.
  **L1811 CN**: 声明 TableGen def `int_ppc_mma_dmxxinstdmr512`。
- **L1812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty, llvm_v256i1_ty,`.
  **L1812 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty, llvm_v256i1_ty,`。
- **L1813 EN**: Executes a standalone statement or declaration: `llvm_v256i1_ty, llvm_i32_ty], [IntrNoMem]>;`.
  **L1813 CN**: 执行一条独立语句或声明：`llvm_v256i1_ty, llvm_i32_ty], [IntrNoMem]>;`。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1815 EN**: Declares TableGen def `int_ppc_mma_dmxxextfdmr256`.
  **L1815 CN**: 声明 TableGen def `int_ppc_mma_dmxxextfdmr256`。
- **L1816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_v1024i1_ty, llvm_i32_ty],`.
  **L1816 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v256i1_ty], [llvm_v1024i1_ty, llvm_i32_ty],`。
- **L1817 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1817 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1818 EN**: Blank line separating nearby declarations or logic blocks.
  **L1818 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1819 EN**: Declares TableGen def `int_ppc_mma_dmxxinstdmr256`.
  **L1819 CN**: 声明 TableGen def `int_ppc_mma_dmxxinstdmr256`。
- **L1820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty, llvm_v256i1_ty,`.
  **L1820 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty, llvm_v256i1_ty,`。

### Lines 1821-1848

````tablegen
                             llvm_i32_ty], [IntrNoMem]>;

  def int_ppc_disassemble_dmr :
      DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_v1024i1_ty],
                            [IntrWriteMem, IntrArgMemOnly]>;

  def int_ppc_build_dmr :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;

  // MMA Reduced-Precision: Outer Product Intrinsic Definitions.
  defm int_ppc_mma_xvi4ger8 :
        PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvi4ger8 :
        PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
                                      llvm_i32_ty, llvm_i32_ty]>;

  defm int_ppc_mma_xvi8ger4 :
       PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvi8ger4 :
       PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
                                     llvm_i32_ty, llvm_i32_ty]>;

  defm int_ppc_mma_xvi16ger2s :
       PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvi16ger2s :
````
- **L1821 EN**: Executes a standalone statement or declaration: `llvm_i32_ty], [IntrNoMem]>;`.
  **L1821 CN**: 执行一条独立语句或声明：`llvm_i32_ty], [IntrNoMem]>;`。
- **L1822 EN**: Blank line separating nearby declarations or logic blocks.
  **L1822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1823 EN**: Declares TableGen def `int_ppc_disassemble_dmr`.
  **L1823 CN**: 声明 TableGen def `int_ppc_disassemble_dmr`。
- **L1824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_v1024i1_ty],`.
  **L1824 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_v1024i1_ty],`。
- **L1825 EN**: Executes a standalone statement or declaration: `[IntrWriteMem, IntrArgMemOnly]>;`.
  **L1825 CN**: 执行一条独立语句或声明：`[IntrWriteMem, IntrArgMemOnly]>;`。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1827 EN**: Declares TableGen def `int_ppc_build_dmr`.
  **L1827 CN**: 声明 TableGen def `int_ppc_build_dmr`。
- **L1828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1828 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1829 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1830 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1831 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1831 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1832 EN**: Blank line separating nearby declarations or logic blocks.
  **L1832 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1833 EN**: Comment explains nearby logic, invariants, or intent: `MMA Reduced-Precision: Outer Product Intrinsic Definitions.`.
  **L1833 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA Reduced-Precision: Outer Product Intrinsic Definitions.`。
- **L1834 EN**: Declares TableGen defm `int_ppc_mma_xvi4ger8`.
  **L1834 CN**: 声明 TableGen defm `int_ppc_mma_xvi4ger8`。
- **L1835 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`.
  **L1835 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`。
- **L1836 EN**: Declares TableGen defm `int_ppc_mma_pmxvi4ger8`.
  **L1836 CN**: 声明 TableGen defm `int_ppc_mma_pmxvi4ger8`。
- **L1837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1837 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1838 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1838 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1839 EN**: Blank line separating nearby declarations or logic blocks.
  **L1839 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1840 EN**: Declares TableGen defm `int_ppc_mma_xvi8ger4`.
  **L1840 CN**: 声明 TableGen defm `int_ppc_mma_xvi8ger4`。
- **L1841 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`.
  **L1841 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`。
- **L1842 EN**: Declares TableGen defm `int_ppc_mma_pmxvi8ger4`.
  **L1842 CN**: 声明 TableGen defm `int_ppc_mma_pmxvi8ger4`。
- **L1843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1843 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1844 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1844 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1845 EN**: Blank line separating nearby declarations or logic blocks.
  **L1845 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1846 EN**: Declares TableGen defm `int_ppc_mma_xvi16ger2s`.
  **L1846 CN**: 声明 TableGen defm `int_ppc_mma_xvi16ger2s`。
- **L1847 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`.
  **L1847 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`。
- **L1848 EN**: Declares TableGen defm `int_ppc_mma_pmxvi16ger2s`.
  **L1848 CN**: 声明 TableGen defm `int_ppc_mma_pmxvi16ger2s`。

### Lines 1849-1876

````tablegen
       PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
                                     llvm_i32_ty, llvm_i32_ty]>;

  defm int_ppc_mma_xvf16ger2 :
       PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvf16ger2 :
       PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
                                  llvm_i32_ty, llvm_i32_ty]>;
  defm int_ppc_mma_xvf32ger :
       PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvf32ger :
       PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
                                  llvm_i32_ty]>;
  defm int_ppc_mma_xvf64ger :
       PowerPC_MMA_ACC_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvf64ger :
       PowerPC_MMA_ACC_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,
                                  llvm_i32_ty]>;

  // MMA Reduced-Precision: bfloat16 Outer Product Intrinsic Definitions.
  defm int_ppc_mma_xvbf16ger2 :
         PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvbf16ger2 :
         PowerPC_MMA_ACC_Intrinsic<
           [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

  // MMA Reduced-Precision: Missing Integer-based Outer Product Operations.
  defm int_ppc_mma_xvi16ger2 :
````
- **L1849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1849 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1850 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1850 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1851 EN**: Blank line separating nearby declarations or logic blocks.
  **L1851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1852 EN**: Declares TableGen defm `int_ppc_mma_xvf16ger2`.
  **L1852 CN**: 声明 TableGen defm `int_ppc_mma_xvf16ger2`。
- **L1853 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`.
  **L1853 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`。
- **L1854 EN**: Declares TableGen defm `int_ppc_mma_pmxvf16ger2`.
  **L1854 CN**: 声明 TableGen defm `int_ppc_mma_pmxvf16ger2`。
- **L1855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1855 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1856 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1856 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1857 EN**: Declares TableGen defm `int_ppc_mma_xvf32ger`.
  **L1857 CN**: 声明 TableGen defm `int_ppc_mma_xvf32ger`。
- **L1858 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`.
  **L1858 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`。
- **L1859 EN**: Declares TableGen defm `int_ppc_mma_pmxvf32ger`.
  **L1859 CN**: 声明 TableGen defm `int_ppc_mma_pmxvf32ger`。
- **L1860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1860 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1861 EN**: Executes a standalone statement or declaration: `llvm_i32_ty]>;`.
  **L1861 CN**: 执行一条独立语句或声明：`llvm_i32_ty]>;`。
- **L1862 EN**: Declares TableGen defm `int_ppc_mma_xvf64ger`.
  **L1862 CN**: 声明 TableGen defm `int_ppc_mma_xvf64ger`。
- **L1863 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`.
  **L1863 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`。
- **L1864 EN**: Declares TableGen defm `int_ppc_mma_pmxvf64ger`.
  **L1864 CN**: 声明 TableGen defm `int_ppc_mma_pmxvf64ger`。
- **L1865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_ACC_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1865 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_ACC_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1866 EN**: Executes a standalone statement or declaration: `llvm_i32_ty]>;`.
  **L1866 CN**: 执行一条独立语句或声明：`llvm_i32_ty]>;`。
- **L1867 EN**: Blank line separating nearby declarations or logic blocks.
  **L1867 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1868 EN**: Comment explains nearby logic, invariants, or intent: `MMA Reduced-Precision: bfloat16 Outer Product Intrinsic Definitions.`.
  **L1868 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA Reduced-Precision: bfloat16 Outer Product Intrinsic Definitions.`。
- **L1869 EN**: Declares TableGen defm `int_ppc_mma_xvbf16ger2`.
  **L1869 CN**: 声明 TableGen defm `int_ppc_mma_xvbf16ger2`。
- **L1870 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`.
  **L1870 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`。
- **L1871 EN**: Declares TableGen defm `int_ppc_mma_pmxvbf16ger2`.
  **L1871 CN**: 声明 TableGen defm `int_ppc_mma_pmxvbf16ger2`。
- **L1872 EN**: Continues the surrounding expression or declaration: `PowerPC_MMA_ACC_Intrinsic<`.
  **L1872 CN**: 继续构造周围的表达式或声明：`PowerPC_MMA_ACC_Intrinsic<`。
- **L1873 EN**: Executes a standalone statement or declaration: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L1873 CN**: 执行一条独立语句或声明：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L1874 EN**: Blank line separating nearby declarations or logic blocks.
  **L1874 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1875 EN**: Comment explains nearby logic, invariants, or intent: `MMA Reduced-Precision: Missing Integer-based Outer Product Operations.`.
  **L1875 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA Reduced-Precision: Missing Integer-based Outer Product Operations.`。
- **L1876 EN**: Declares TableGen defm `int_ppc_mma_xvi16ger2`.
  **L1876 CN**: 声明 TableGen defm `int_ppc_mma_xvi16ger2`。

### Lines 1877-1904

````tablegen
         PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmxvi16ger2 :
         PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
                                       llvm_i32_ty, llvm_i32_ty]>;
  def int_ppc_mma_xvi8ger4spp :
      DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                            [llvm_v512i1_ty, llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_mma_pmxvi8ger4spp :
      DefaultAttrsIntrinsic<[llvm_v512i1_ty],
                            [llvm_v512i1_ty, llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;
  defm int_ppc_mma_dmxvi8gerx4 :
       PowerPC_MMA_DMR_PP_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmdmxvi8gerx4 :
       PowerPC_MMA_DMR_PP_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,
                                     llvm_i32_ty, llvm_i32_ty]>;
  def int_ppc_mma_dmxvi8gerx4spp :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty],
                            [llvm_v1024i1_ty, llvm_v256i1_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_mma_pmdmxvi8gerx4spp :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty],
                            [llvm_v1024i1_ty, llvm_v256i1_ty, llvm_v16i8_ty,
                             llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem]>;

````
- **L1877 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`.
  **L1877 CN**: 执行一条独立语句或声明：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty]>;`。
- **L1878 EN**: Declares TableGen defm `int_ppc_mma_pmxvi16ger2`.
  **L1878 CN**: 声明 TableGen defm `int_ppc_mma_pmxvi16ger2`。
- **L1879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1879 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_ACC_PP_Intrinsic<[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1880 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1880 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1881 EN**: Declares TableGen def `int_ppc_mma_xvi8ger4spp`.
  **L1881 CN**: 声明 TableGen def `int_ppc_mma_xvi8ger4spp`。
- **L1882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v512i1_ty],`.
  **L1882 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v512i1_ty],`。
- **L1883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v512i1_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L1883 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v512i1_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L1884 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1884 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1885 EN**: Declares TableGen def `int_ppc_mma_pmxvi8ger4spp`.
  **L1885 CN**: 声明 TableGen def `int_ppc_mma_pmxvi8ger4spp`。
- **L1886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v512i1_ty],`.
  **L1886 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v512i1_ty],`。
- **L1887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v512i1_ty, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L1887 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v512i1_ty, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L1888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1888 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1889 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1889 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1890 EN**: Declares TableGen defm `int_ppc_mma_dmxvi8gerx4`.
  **L1890 CN**: 声明 TableGen defm `int_ppc_mma_dmxvi8gerx4`。
- **L1891 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_DMR_PP_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`.
  **L1891 CN**: 执行一条独立语句或声明：`PowerPC_MMA_DMR_PP_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`。
- **L1892 EN**: Declares TableGen defm `int_ppc_mma_pmdmxvi8gerx4`.
  **L1892 CN**: 声明 TableGen defm `int_ppc_mma_pmdmxvi8gerx4`。
- **L1893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_DMR_PP_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1893 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_DMR_PP_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1894 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1894 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1895 EN**: Declares TableGen def `int_ppc_mma_dmxvi8gerx4spp`.
  **L1895 CN**: 声明 TableGen def `int_ppc_mma_dmxvi8gerx4spp`。
- **L1896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty],`.
  **L1896 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty],`。
- **L1897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1024i1_ty, llvm_v256i1_ty, llvm_v16i8_ty],`.
  **L1897 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1024i1_ty, llvm_v256i1_ty, llvm_v16i8_ty],`。
- **L1898 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1898 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1899 EN**: Declares TableGen def `int_ppc_mma_pmdmxvi8gerx4spp`.
  **L1899 CN**: 声明 TableGen def `int_ppc_mma_pmdmxvi8gerx4spp`。
- **L1900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty],`.
  **L1900 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty],`。
- **L1901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v1024i1_ty, llvm_v256i1_ty, llvm_v16i8_ty,`.
  **L1901 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v1024i1_ty, llvm_v256i1_ty, llvm_v16i8_ty,`。
- **L1902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L1902 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L1903 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L1903 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L1904 EN**: Blank line separating nearby declarations or logic blocks.
  **L1904 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1905-1932

````tablegen
  // MMA+ Reduced-Precision: bfloat16 Outer Product Intrinsic Definitions.
  defm int_ppc_mma_dmxvbf16gerx2 :
       PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmdmxvbf16gerx2 :
       PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,
                                     llvm_i32_ty, llvm_i32_ty]>;

  // MMA+ Half-precision Outer Product Intrinsic Definitions.
  defm int_ppc_mma_dmxvf16gerx2 :
       PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;
  defm int_ppc_mma_pmdmxvf16gerx2 :
       PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,
                                     llvm_i32_ty, llvm_i32_ty]>;
  def int_ppc_dmsha2hash :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,
                             llvm_v1024i1_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  def int_ppc_dmsha3hash :
      DefaultAttrsIntrinsic<[llvm_v2048i1_ty], [llvm_v2048i1_ty,
                             llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  def int_ppc_dmxxshapad :
      DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,
                             llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty,
                             llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<2>>,
                             ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;
}
````
- **L1905 EN**: Comment explains nearby logic, invariants, or intent: `MMA+ Reduced-Precision: bfloat16 Outer Product Intrinsic Definitions.`.
  **L1905 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA+ Reduced-Precision: bfloat16 Outer Product Intrinsic Definitions.`。
- **L1906 EN**: Declares TableGen defm `int_ppc_mma_dmxvbf16gerx2`.
  **L1906 CN**: 声明 TableGen defm `int_ppc_mma_dmxvbf16gerx2`。
- **L1907 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`.
  **L1907 CN**: 执行一条独立语句或声明：`PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`。
- **L1908 EN**: Declares TableGen defm `int_ppc_mma_pmdmxvbf16gerx2`.
  **L1908 CN**: 声明 TableGen defm `int_ppc_mma_pmdmxvbf16gerx2`。
- **L1909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1909 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1910 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1910 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1911 EN**: Blank line separating nearby declarations or logic blocks.
  **L1911 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1912 EN**: Comment explains nearby logic, invariants, or intent: `MMA+ Half-precision Outer Product Intrinsic Definitions.`.
  **L1912 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA+ Half-precision Outer Product Intrinsic Definitions.`。
- **L1913 EN**: Declares TableGen defm `int_ppc_mma_dmxvf16gerx2`.
  **L1913 CN**: 声明 TableGen defm `int_ppc_mma_dmxvf16gerx2`。
- **L1914 EN**: Executes a standalone statement or declaration: `PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`.
  **L1914 CN**: 执行一条独立语句或声明：`PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty]>;`。
- **L1915 EN**: Declares TableGen defm `int_ppc_mma_pmdmxvf16gerx2`.
  **L1915 CN**: 声明 TableGen defm `int_ppc_mma_pmdmxvf16gerx2`。
- **L1916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L1916 CN**: 继续一个多行参数列表、初始化器或聚合项：`PowerPC_MMA_DMR_Intrinsic<[llvm_v256i1_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L1917 EN**: Executes a standalone statement or declaration: `llvm_i32_ty, llvm_i32_ty]>;`.
  **L1917 CN**: 执行一条独立语句或声明：`llvm_i32_ty, llvm_i32_ty]>;`。
- **L1918 EN**: Declares TableGen def `int_ppc_dmsha2hash`.
  **L1918 CN**: 声明 TableGen def `int_ppc_dmsha2hash`。
- **L1919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,`.
  **L1919 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,`。
- **L1920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v1024i1_ty, llvm_i32_ty],`.
  **L1920 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v1024i1_ty, llvm_i32_ty],`。
- **L1921 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L1921 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L1922 EN**: Blank line separating nearby declarations or logic blocks.
  **L1922 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1923 EN**: Declares TableGen def `int_ppc_dmsha3hash`.
  **L1923 CN**: 声明 TableGen def `int_ppc_dmsha3hash`。
- **L1924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v2048i1_ty], [llvm_v2048i1_ty,`.
  **L1924 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v2048i1_ty], [llvm_v2048i1_ty,`。
- **L1925 EN**: Executes a standalone statement or declaration: `llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L1925 CN**: 执行一条独立语句或声明：`llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L1926 EN**: Blank line separating nearby declarations or logic blocks.
  **L1926 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1927 EN**: Declares TableGen def `int_ppc_dmxxshapad`.
  **L1927 CN**: 声明 TableGen def `int_ppc_dmxxshapad`。
- **L1928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,`.
  **L1928 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v1024i1_ty], [llvm_v1024i1_ty,`。
- **L1929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty,`.
  **L1929 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_v16i8_ty, llvm_i32_ty, llvm_i32_ty,`。
- **L1930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<2>>,`.
  **L1930 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<2>>,`。
- **L1931 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`.
  **L1931 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`。
- **L1932 EN**: Closes the current lexical scope or compound statement.
  **L1932 CN**: 结束当前词法作用域或复合语句块。

### Lines 1933-1960

````tablegen

// XL Compat intrinsics.
let TargetPrefix = "ppc" in {
  def int_ppc_dcbfl : ClangBuiltin<"__builtin_ppc_dcbfl">,
                      Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;
  def int_ppc_dcbflp : ClangBuiltin<"__builtin_ppc_dcbflp">,
                       Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;
  def int_ppc_dcbst : ClangBuiltin<"__builtin_ppc_dcbst">,
                      Intrinsic<[], [llvm_ptr_ty], []>;
  def int_ppc_dcbt  : ClangBuiltin<"__builtin_ppc_dcbt">,
                      Intrinsic<[], [llvm_ptr_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_ppc_dcbtst : ClangBuiltin<"__builtin_ppc_dcbtst">,
                       Intrinsic<[], [llvm_ptr_ty],
    [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_ppc_dcbz  : ClangBuiltin<"__builtin_ppc_dcbz">,
                      Intrinsic<[], [llvm_ptr_ty], []>;
  def int_ppc_icbt : ClangBuiltin<"__builtin_ppc_icbt">,
                     Intrinsic<[], [llvm_ptr_ty], []>;
  
  // Population Count in each Byte.
  def int_ppc_popcntb :
      DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem]>;
  
  // sync instruction (i.e. sync 0, a.k.a hwsync)
  def int_ppc_sync : ClangBuiltin<"__builtin_ppc_sync">,
                     Intrinsic<[], [], []>;
  def int_ppc_iospace_sync : ClangBuiltin<"__builtin_ppc_iospace_sync">,
````
- **L1933 EN**: Blank line separating nearby declarations or logic blocks.
  **L1933 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1934 EN**: Comment explains nearby logic, invariants, or intent: `XL Compat intrinsics.`.
  **L1934 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`XL Compat intrinsics.`。
- **L1935 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1935 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1936 EN**: Declares TableGen def `int_ppc_dcbfl`.
  **L1936 CN**: 声明 TableGen def `int_ppc_dcbfl`。
- **L1937 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;`.
  **L1937 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;`。
- **L1938 EN**: Declares TableGen def `int_ppc_dcbflp`.
  **L1938 CN**: 声明 TableGen def `int_ppc_dcbflp`。
- **L1939 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;`.
  **L1939 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty], [IntrArgMemOnly]>;`。
- **L1940 EN**: Declares TableGen def `int_ppc_dcbst`.
  **L1940 CN**: 声明 TableGen def `int_ppc_dcbst`。
- **L1941 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty], []>;`.
  **L1941 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty], []>;`。
- **L1942 EN**: Declares TableGen def `int_ppc_dcbt`.
  **L1942 CN**: 声明 TableGen def `int_ppc_dcbt`。
- **L1943 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_ptr_ty],`.
  **L1943 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_ptr_ty],`。
- **L1944 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L1944 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L1945 EN**: Declares TableGen def `int_ppc_dcbtst`.
  **L1945 CN**: 声明 TableGen def `int_ppc_dcbtst`。
- **L1946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_ptr_ty],`.
  **L1946 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_ptr_ty],`。
- **L1947 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L1947 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L1948 EN**: Declares TableGen def `int_ppc_dcbz`.
  **L1948 CN**: 声明 TableGen def `int_ppc_dcbz`。
- **L1949 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty], []>;`.
  **L1949 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty], []>;`。
- **L1950 EN**: Declares TableGen def `int_ppc_icbt`.
  **L1950 CN**: 声明 TableGen def `int_ppc_icbt`。
- **L1951 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty], []>;`.
  **L1951 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty], []>;`。
- **L1952 EN**: Blank line separating nearby declarations or logic blocks.
  **L1952 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1953 EN**: Comment explains nearby logic, invariants, or intent: `Population Count in each Byte.`.
  **L1953 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Population Count in each Byte.`。
- **L1954 EN**: Declares TableGen def `int_ppc_popcntb`.
  **L1954 CN**: 声明 TableGen def `int_ppc_popcntb`。
- **L1955 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem]>;`.
  **L1955 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_anyint_ty], [IntrNoMem]>;`。
- **L1956 EN**: Blank line separating nearby declarations or logic blocks.
  **L1956 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1957 EN**: Comment explains nearby logic, invariants, or intent: `sync instruction (i.e. sync 0, a.k.a hwsync)`.
  **L1957 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sync instruction (i.e. sync 0, a.k.a hwsync)`。
- **L1958 EN**: Declares TableGen def `int_ppc_sync`.
  **L1958 CN**: 声明 TableGen def `int_ppc_sync`。
- **L1959 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], []>;`.
  **L1959 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], []>;`。
- **L1960 EN**: Declares TableGen def `int_ppc_iospace_sync`.
  **L1960 CN**: 声明 TableGen def `int_ppc_iospace_sync`。

### Lines 1961-1988

````tablegen
                             Intrinsic<[], [], []>;
  // isync instruction
  def int_ppc_isync : ClangBuiltin<"__builtin_ppc_isync">,
                      Intrinsic<[], [], []>;
  // lwsync is sync 1
  def int_ppc_lwsync : ClangBuiltin<"__builtin_ppc_lwsync">,
                       Intrinsic<[], [], []>;
  def int_ppc_iospace_lwsync : ClangBuiltin<"__builtin_ppc_iospace_lwsync">,
                               Intrinsic<[], [], []>;
  // eieio instruction
  def int_ppc_eieio : ClangBuiltin<"__builtin_ppc_eieio">,
                      Intrinsic<[],[],[]>;
  def int_ppc_iospace_eieio : ClangBuiltin<"__builtin_ppc_iospace_eieio">,
                              Intrinsic<[],[],[]>;
  def int_ppc_lbarx :
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;
  def int_ppc_lharx :
    Intrinsic<[llvm_i32_ty],[llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;
  def int_ppc_lwarx :
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;
  def int_ppc_ldarx :
    Intrinsic<[llvm_i64_ty],[llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;

  def int_ppc_stdcx :
    ClangBuiltin<"__builtin_ppc_stdcx">,
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i64_ty],
              [IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;
  def int_ppc_stwcx :
````
- **L1961 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], []>;`.
  **L1961 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], []>;`。
- **L1962 EN**: Comment explains nearby logic, invariants, or intent: `isync instruction`.
  **L1962 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`isync instruction`。
- **L1963 EN**: Declares TableGen def `int_ppc_isync`.
  **L1963 CN**: 声明 TableGen def `int_ppc_isync`。
- **L1964 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], []>;`.
  **L1964 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], []>;`。
- **L1965 EN**: Comment explains nearby logic, invariants, or intent: `lwsync is sync 1`.
  **L1965 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`lwsync is sync 1`。
- **L1966 EN**: Declares TableGen def `int_ppc_lwsync`.
  **L1966 CN**: 声明 TableGen def `int_ppc_lwsync`。
- **L1967 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], []>;`.
  **L1967 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], []>;`。
- **L1968 EN**: Declares TableGen def `int_ppc_iospace_lwsync`.
  **L1968 CN**: 声明 TableGen def `int_ppc_iospace_lwsync`。
- **L1969 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], []>;`.
  **L1969 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], []>;`。
- **L1970 EN**: Comment explains nearby logic, invariants, or intent: `eieio instruction`.
  **L1970 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`eieio instruction`。
- **L1971 EN**: Declares TableGen def `int_ppc_eieio`.
  **L1971 CN**: 声明 TableGen def `int_ppc_eieio`。
- **L1972 EN**: Executes a standalone statement or declaration: `Intrinsic<[],[],[]>;`.
  **L1972 CN**: 执行一条独立语句或声明：`Intrinsic<[],[],[]>;`。
- **L1973 EN**: Declares TableGen def `int_ppc_iospace_eieio`.
  **L1973 CN**: 声明 TableGen def `int_ppc_iospace_eieio`。
- **L1974 EN**: Executes a standalone statement or declaration: `Intrinsic<[],[],[]>;`.
  **L1974 CN**: 执行一条独立语句或声明：`Intrinsic<[],[],[]>;`。
- **L1975 EN**: Declares TableGen def `int_ppc_lbarx`.
  **L1975 CN**: 声明 TableGen def `int_ppc_lbarx`。
- **L1976 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1976 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1977 EN**: Declares TableGen def `int_ppc_lharx`.
  **L1977 CN**: 声明 TableGen def `int_ppc_lharx`。
- **L1978 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty],[llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1978 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty],[llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1979 EN**: Declares TableGen def `int_ppc_lwarx`.
  **L1979 CN**: 声明 TableGen def `int_ppc_lwarx`。
- **L1980 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1980 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1981 EN**: Declares TableGen def `int_ppc_ldarx`.
  **L1981 CN**: 声明 TableGen def `int_ppc_ldarx`。
- **L1982 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty],[llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`.
  **L1982 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty],[llvm_ptr_ty], [IntrInaccessibleMemOrArgMemOnly]>;`。
- **L1983 EN**: Blank line separating nearby declarations or logic blocks.
  **L1983 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1984 EN**: Declares TableGen def `int_ppc_stdcx`.
  **L1984 CN**: 声明 TableGen def `int_ppc_stdcx`。
- **L1985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_ppc_stdcx">,`.
  **L1985 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_ppc_stdcx">,`。
- **L1986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i64_ty],`.
  **L1986 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i64_ty],`。
- **L1987 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`.
  **L1987 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`。
- **L1988 EN**: Declares TableGen def `int_ppc_stwcx`.
  **L1988 CN**: 声明 TableGen def `int_ppc_stwcx`。

### Lines 1989-2016

````tablegen
    ClangBuiltin<"__builtin_ppc_stwcx">,
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],
              [IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;
  def int_ppc_sthcx :
    Intrinsic<[llvm_i32_ty], [ llvm_ptr_ty, llvm_i32_ty],
              [IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;
  def int_ppc_stbcx :
    ClangBuiltin<"__builtin_ppc_stbcx">,
    Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],
              [IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;
  def int_ppc_dcbtstt : ClangBuiltin<"__builtin_ppc_dcbtstt">,
                        Intrinsic<[], [llvm_ptr_ty],
                                  [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_ppc_dcbtt : ClangBuiltin<"__builtin_ppc_dcbtt">,
                      Intrinsic<[], [llvm_ptr_ty],
                                [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_ppc_mftbu : ClangBuiltin<"__builtin_ppc_mftbu">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem]>;
  def int_ppc_mfmsr : ClangBuiltin<"__builtin_ppc_mfmsr">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem]>;
  def int_ppc_mfspr
      : DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;
  def int_ppc_mtmsr
      : ClangBuiltin<"__builtin_ppc_mtmsr">, Intrinsic<[], [llvm_i32_ty], []>;
  def int_ppc_mtspr
      : DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyint_ty], [ImmArg<ArgIndex<0>>]>;
  def int_ppc_stfiw : ClangBuiltin<"__builtin_ppc_stfiw">,
                      DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_double_ty],
````
- **L1989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_ppc_stwcx">,`.
  **L1989 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_ppc_stwcx">,`。
- **L1990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1990 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1991 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`.
  **L1991 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`。
- **L1992 EN**: Declares TableGen def `int_ppc_sthcx`.
  **L1992 CN**: 声明 TableGen def `int_ppc_sthcx`。
- **L1993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [ llvm_ptr_ty, llvm_i32_ty],`.
  **L1993 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [ llvm_ptr_ty, llvm_i32_ty],`。
- **L1994 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`.
  **L1994 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`。
- **L1995 EN**: Declares TableGen def `int_ppc_stbcx`.
  **L1995 CN**: 声明 TableGen def `int_ppc_stbcx`。
- **L1996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ClangBuiltin<"__builtin_ppc_stbcx">,`.
  **L1996 CN**: 继续一个多行参数列表、初始化器或聚合项：`ClangBuiltin<"__builtin_ppc_stbcx">,`。
- **L1997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L1997 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L1998 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`.
  **L1998 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOrArgMemOnly, IntrNoDuplicate]>;`。
- **L1999 EN**: Declares TableGen def `int_ppc_dcbtstt`.
  **L1999 CN**: 声明 TableGen def `int_ppc_dcbtstt`。
- **L2000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_ptr_ty],`.
  **L2000 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_ptr_ty],`。
- **L2001 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L2001 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L2002 EN**: Declares TableGen def `int_ppc_dcbtt`.
  **L2002 CN**: 声明 TableGen def `int_ppc_dcbtt`。
- **L2003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_ptr_ty],`.
  **L2003 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_ptr_ty],`。
- **L2004 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L2004 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L2005 EN**: Declares TableGen def `int_ppc_mftbu`.
  **L2005 CN**: 声明 TableGen def `int_ppc_mftbu`。
- **L2006 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem]>;`.
  **L2006 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem]>;`。
- **L2007 EN**: Declares TableGen def `int_ppc_mfmsr`.
  **L2007 CN**: 声明 TableGen def `int_ppc_mfmsr`。
- **L2008 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem]>;`.
  **L2008 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [], [IntrNoMem]>;`。
- **L2009 EN**: Declares TableGen def `int_ppc_mfspr`.
  **L2009 CN**: 声明 TableGen def `int_ppc_mfspr`。
- **L2010 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L2010 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_anyint_ty], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L2011 EN**: Declares TableGen def `int_ppc_mtmsr`.
  **L2011 CN**: 声明 TableGen def `int_ppc_mtmsr`。
- **L2012 EN**: Executes a standalone statement or declaration: `: ClangBuiltin<"__builtin_ppc_mtmsr">, Intrinsic<[], [llvm_i32_ty], []>;`.
  **L2012 CN**: 执行一条独立语句或声明：`: ClangBuiltin<"__builtin_ppc_mtmsr">, Intrinsic<[], [llvm_i32_ty], []>;`。
- **L2013 EN**: Declares TableGen def `int_ppc_mtspr`.
  **L2013 CN**: 声明 TableGen def `int_ppc_mtspr`。
- **L2014 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyint_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L2014 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_anyint_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L2015 EN**: Declares TableGen def `int_ppc_stfiw`.
  **L2015 CN**: 声明 TableGen def `int_ppc_stfiw`。
- **L2016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_double_ty],`.
  **L2016 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_double_ty],`。

### Lines 2017-2044

````tablegen
                                            [IntrWriteMem]>;
  // compare
  def int_ppc_cmpeqb
      : ClangBuiltin<"__builtin_ppc_cmpeqb">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;
  def int_ppc_cmprb
      : ClangBuiltin<"__builtin_ppc_cmprb">,
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,
                              llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;
  def int_ppc_setb
      : ClangBuiltin<"__builtin_ppc_setb">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;
  def int_ppc_cmpb
      : DefaultAttrsIntrinsic<[llvm_anyint_ty],
                              [llvm_anyint_ty, llvm_anyint_ty], [IntrNoMem]>;
  // multiply
  def int_ppc_mulhd
      : ClangBuiltin<"__builtin_ppc_mulhd">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;
  def int_ppc_mulhdu
      : ClangBuiltin<"__builtin_ppc_mulhdu">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                              [IntrNoMem]>;
  def int_ppc_mulhw
      : ClangBuiltin<"__builtin_ppc_mulhw">,
````
- **L2017 EN**: Executes a standalone statement or declaration: `[IntrWriteMem]>;`.
  **L2017 CN**: 执行一条独立语句或声明：`[IntrWriteMem]>;`。
- **L2018 EN**: Comment explains nearby logic, invariants, or intent: `compare`.
  **L2018 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`compare`。
- **L2019 EN**: Declares TableGen def `int_ppc_cmpeqb`.
  **L2019 CN**: 声明 TableGen def `int_ppc_cmpeqb`。
- **L2020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_cmpeqb">,`.
  **L2020 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_cmpeqb">,`。
- **L2021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L2021 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L2022 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2022 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2023 EN**: Declares TableGen def `int_ppc_cmprb`.
  **L2023 CN**: 声明 TableGen def `int_ppc_cmprb`。
- **L2024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_cmprb">,`.
  **L2024 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_cmprb">,`。
- **L2025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`.
  **L2025 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty,`。
- **L2026 EN**: Executes a standalone statement or declaration: `llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`.
  **L2026 CN**: 执行一条独立语句或声明：`llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<0>>]>;`。
- **L2027 EN**: Declares TableGen def `int_ppc_setb`.
  **L2027 CN**: 声明 TableGen def `int_ppc_setb`。
- **L2028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_setb">,`.
  **L2028 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_setb">,`。
- **L2029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L2029 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L2030 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2030 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2031 EN**: Declares TableGen def `int_ppc_cmpb`.
  **L2031 CN**: 声明 TableGen def `int_ppc_cmpb`。
- **L2032 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyint_ty],`.
  **L2032 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyint_ty],`。
- **L2033 EN**: Executes a standalone statement or declaration: `[llvm_anyint_ty, llvm_anyint_ty], [IntrNoMem]>;`.
  **L2033 CN**: 执行一条独立语句或声明：`[llvm_anyint_ty, llvm_anyint_ty], [IntrNoMem]>;`。
- **L2034 EN**: Comment explains nearby logic, invariants, or intent: `multiply`.
  **L2034 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`multiply`。
- **L2035 EN**: Declares TableGen def `int_ppc_mulhd`.
  **L2035 CN**: 声明 TableGen def `int_ppc_mulhd`。
- **L2036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_mulhd">,`.
  **L2036 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_mulhd">,`。
- **L2037 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L2037 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L2038 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2038 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2039 EN**: Declares TableGen def `int_ppc_mulhdu`.
  **L2039 CN**: 声明 TableGen def `int_ppc_mulhdu`。
- **L2040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_mulhdu">,`.
  **L2040 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_mulhdu">,`。
- **L2041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L2041 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L2042 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2042 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2043 EN**: Declares TableGen def `int_ppc_mulhw`.
  **L2043 CN**: 声明 TableGen def `int_ppc_mulhw`。
- **L2044 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_mulhw">,`.
  **L2044 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_mulhw">,`。

### Lines 2045-2072

````tablegen
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                              [IntrNoMem]>;
  def int_ppc_mulhwu
      : ClangBuiltin<"__builtin_ppc_mulhwu">,
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],
                              [IntrNoMem]>;
  def int_ppc_maddhd
      : ClangBuiltin<"__builtin_ppc_maddhd">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,
                               llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_maddhdu
      : ClangBuiltin<"__builtin_ppc_maddhdu">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,
                               llvm_i64_ty], [IntrNoMem]>;
  def int_ppc_maddld
      : ClangBuiltin<"__builtin_ppc_maddld">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,
                               llvm_i64_ty], [IntrNoMem]>;
  // load
  def int_ppc_load2r
      : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty],
                              [IntrReadMem, IntrArgMemOnly]>;
  def int_ppc_load4r
      : ClangBuiltin<"__builtin_ppc_load4r">,
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty],
                              [IntrReadMem, IntrArgMemOnly]>;
  def int_ppc_load8r
      : ClangBuiltin<"__builtin_ppc_load8r">,
````
- **L2045 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2045 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2046 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2046 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2047 EN**: Declares TableGen def `int_ppc_mulhwu`.
  **L2047 CN**: 声明 TableGen def `int_ppc_mulhwu`。
- **L2048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_mulhwu">,`.
  **L2048 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_mulhwu">,`。
- **L2049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`.
  **L2049 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty],`。
- **L2050 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2050 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2051 EN**: Declares TableGen def `int_ppc_maddhd`.
  **L2051 CN**: 声明 TableGen def `int_ppc_maddhd`。
- **L2052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_maddhd">,`.
  **L2052 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_maddhd">,`。
- **L2053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,`.
  **L2053 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,`。
- **L2054 EN**: Executes a standalone statement or declaration: `llvm_i64_ty], [IntrNoMem]>;`.
  **L2054 CN**: 执行一条独立语句或声明：`llvm_i64_ty], [IntrNoMem]>;`。
- **L2055 EN**: Declares TableGen def `int_ppc_maddhdu`.
  **L2055 CN**: 声明 TableGen def `int_ppc_maddhdu`。
- **L2056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_maddhdu">,`.
  **L2056 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_maddhdu">,`。
- **L2057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,`.
  **L2057 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,`。
- **L2058 EN**: Executes a standalone statement or declaration: `llvm_i64_ty], [IntrNoMem]>;`.
  **L2058 CN**: 执行一条独立语句或声明：`llvm_i64_ty], [IntrNoMem]>;`。
- **L2059 EN**: Declares TableGen def `int_ppc_maddld`.
  **L2059 CN**: 声明 TableGen def `int_ppc_maddld`。
- **L2060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_maddld">,`.
  **L2060 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_maddld">,`。
- **L2061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,`.
  **L2061 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty,`。
- **L2062 EN**: Executes a standalone statement or declaration: `llvm_i64_ty], [IntrNoMem]>;`.
  **L2062 CN**: 执行一条独立语句或声明：`llvm_i64_ty], [IntrNoMem]>;`。
- **L2063 EN**: Comment explains nearby logic, invariants, or intent: `load`.
  **L2063 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`load`。
- **L2064 EN**: Declares TableGen def `int_ppc_load2r`.
  **L2064 CN**: 声明 TableGen def `int_ppc_load2r`。
- **L2065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty],`.
  **L2065 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty],`。
- **L2066 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L2066 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L2067 EN**: Declares TableGen def `int_ppc_load4r`.
  **L2067 CN**: 声明 TableGen def `int_ppc_load4r`。
- **L2068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_load4r">,`.
  **L2068 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_load4r">,`。
- **L2069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty],`.
  **L2069 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty],`。
- **L2070 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L2070 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L2071 EN**: Declares TableGen def `int_ppc_load8r`.
  **L2071 CN**: 声明 TableGen def `int_ppc_load8r`。
- **L2072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_load8r">,`.
  **L2072 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_load8r">,`。

### Lines 2073-2100

````tablegen
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_ptr_ty],
                              [IntrReadMem, IntrArgMemOnly]>;
  // store
  def int_ppc_store2r
      : ClangBuiltin<"__builtin_ppc_store2r">,
        Intrinsic<[], [llvm_i32_ty, llvm_ptr_ty], [IntrWriteMem]>;
  def int_ppc_store4r
      : ClangBuiltin<"__builtin_ppc_store4r">,
        Intrinsic<[], [llvm_i32_ty, llvm_ptr_ty], [IntrWriteMem]>;
  def int_ppc_store8r
      : ClangBuiltin<"__builtin_ppc_store8r">,
        Intrinsic<[], [llvm_i64_ty, llvm_ptr_ty], [IntrWriteMem]>;
  def int_ppc_insert_exp
      : ClangBuiltin<"__builtin_ppc_insert_exp">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_i64_ty],
                              [IntrNoMem]>;
  def int_ppc_extract_exp
      : ClangBuiltin<"__builtin_ppc_extract_exp">,
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_extract_sig
      : ClangBuiltin<"__builtin_ppc_extract_sig">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_mtfsb0
      : ClangBuiltin<"__builtin_ppc_mtfsb0">,
        DefaultAttrsIntrinsic<[], [llvm_i32_ty],
                              [IntrNoMem, IntrHasSideEffects,
                               ImmArg<ArgIndex<0>>]>;
  def int_ppc_mtfsb1
````
- **L2073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_ptr_ty],`.
  **L2073 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_ptr_ty],`。
- **L2074 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L2074 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L2075 EN**: Comment explains nearby logic, invariants, or intent: `store`.
  **L2075 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`store`。
- **L2076 EN**: Declares TableGen def `int_ppc_store2r`.
  **L2076 CN**: 声明 TableGen def `int_ppc_store2r`。
- **L2077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_store2r">,`.
  **L2077 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_store2r">,`。
- **L2078 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_ptr_ty], [IntrWriteMem]>;`.
  **L2078 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_ptr_ty], [IntrWriteMem]>;`。
- **L2079 EN**: Declares TableGen def `int_ppc_store4r`.
  **L2079 CN**: 声明 TableGen def `int_ppc_store4r`。
- **L2080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_store4r">,`.
  **L2080 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_store4r">,`。
- **L2081 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty, llvm_ptr_ty], [IntrWriteMem]>;`.
  **L2081 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty, llvm_ptr_ty], [IntrWriteMem]>;`。
- **L2082 EN**: Declares TableGen def `int_ppc_store8r`.
  **L2082 CN**: 声明 TableGen def `int_ppc_store8r`。
- **L2083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_store8r">,`.
  **L2083 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_store8r">,`。
- **L2084 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty, llvm_ptr_ty], [IntrWriteMem]>;`.
  **L2084 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty, llvm_ptr_ty], [IntrWriteMem]>;`。
- **L2085 EN**: Declares TableGen def `int_ppc_insert_exp`.
  **L2085 CN**: 声明 TableGen def `int_ppc_insert_exp`。
- **L2086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_insert_exp">,`.
  **L2086 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_insert_exp">,`。
- **L2087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_i64_ty],`.
  **L2087 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_i64_ty],`。
- **L2088 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2088 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2089 EN**: Declares TableGen def `int_ppc_extract_exp`.
  **L2089 CN**: 声明 TableGen def `int_ppc_extract_exp`。
- **L2090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_extract_exp">,`.
  **L2090 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_extract_exp">,`。
- **L2091 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L2091 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L2092 EN**: Declares TableGen def `int_ppc_extract_sig`.
  **L2092 CN**: 声明 TableGen def `int_ppc_extract_sig`。
- **L2093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_extract_sig">,`.
  **L2093 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_extract_sig">,`。
- **L2094 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L2094 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L2095 EN**: Declares TableGen def `int_ppc_mtfsb0`.
  **L2095 CN**: 声明 TableGen def `int_ppc_mtfsb0`。
- **L2096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_mtfsb0">,`.
  **L2096 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_mtfsb0">,`。
- **L2097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty],`.
  **L2097 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty],`。
- **L2098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrHasSideEffects,`.
  **L2098 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrHasSideEffects,`。
- **L2099 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>]>;`.
  **L2099 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>]>;`。
- **L2100 EN**: Declares TableGen def `int_ppc_mtfsb1`.
  **L2100 CN**: 声明 TableGen def `int_ppc_mtfsb1`。

### Lines 2101-2128

````tablegen
      : ClangBuiltin<"__builtin_ppc_mtfsb1">,
        DefaultAttrsIntrinsic<[], [llvm_i32_ty],
                              [IntrNoMem, IntrHasSideEffects,
                               ImmArg<ArgIndex<0>>]>;
  def int_ppc_mtfsf :
        DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_double_ty],
                              [IntrNoMem, IntrHasSideEffects,
                               ImmArg<ArgIndex<0>>]>;
  def int_ppc_mtfsfi
      : ClangBuiltin<"__builtin_ppc_mtfsfi">,
        DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_i32_ty],
                              [IntrNoMem, IntrHasSideEffects,
                               ImmArg<ArgIndex<0>>,ImmArg<ArgIndex<1>>]>;
  def int_ppc_fmsub
      : ClangBuiltin<"__builtin_ppc_fmsub">,
        DefaultAttrsIntrinsic<[llvm_double_ty],
                              [llvm_double_ty, llvm_double_ty, llvm_double_ty],
                              [IntrNoMem]>;
  def int_ppc_fmsubs
      : ClangBuiltin<"__builtin_ppc_fmsubs">,
        DefaultAttrsIntrinsic<[llvm_float_ty],
                              [llvm_float_ty, llvm_float_ty, llvm_float_ty],
                              [IntrNoMem]>;
  def int_ppc_fnmadd
      : ClangBuiltin<"__builtin_ppc_fnmadd">,
        DefaultAttrsIntrinsic<[llvm_double_ty],
                              [llvm_double_ty, llvm_double_ty, llvm_double_ty],
                              [IntrNoMem]>;
````
- **L2101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_mtfsb1">,`.
  **L2101 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_mtfsb1">,`。
- **L2102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty],`.
  **L2102 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty],`。
- **L2103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrHasSideEffects,`.
  **L2103 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrHasSideEffects,`。
- **L2104 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>]>;`.
  **L2104 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>]>;`。
- **L2105 EN**: Declares TableGen def `int_ppc_mtfsf`.
  **L2105 CN**: 声明 TableGen def `int_ppc_mtfsf`。
- **L2106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_double_ty],`.
  **L2106 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_double_ty],`。
- **L2107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrHasSideEffects,`.
  **L2107 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrHasSideEffects,`。
- **L2108 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>]>;`.
  **L2108 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>]>;`。
- **L2109 EN**: Declares TableGen def `int_ppc_mtfsfi`.
  **L2109 CN**: 声明 TableGen def `int_ppc_mtfsfi`。
- **L2110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_mtfsfi">,`.
  **L2110 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_mtfsfi">,`。
- **L2111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_i32_ty],`.
  **L2111 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty, llvm_i32_ty],`。
- **L2112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, IntrHasSideEffects,`.
  **L2112 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, IntrHasSideEffects,`。
- **L2113 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>,ImmArg<ArgIndex<1>>]>;`.
  **L2113 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>,ImmArg<ArgIndex<1>>]>;`。
- **L2114 EN**: Declares TableGen def `int_ppc_fmsub`.
  **L2114 CN**: 声明 TableGen def `int_ppc_fmsub`。
- **L2115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fmsub">,`.
  **L2115 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fmsub">,`。
- **L2116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty],`.
  **L2116 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty],`。
- **L2117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty, llvm_double_ty],`.
  **L2117 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty, llvm_double_ty],`。
- **L2118 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2118 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2119 EN**: Declares TableGen def `int_ppc_fmsubs`.
  **L2119 CN**: 声明 TableGen def `int_ppc_fmsubs`。
- **L2120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fmsubs">,`.
  **L2120 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fmsubs">,`。
- **L2121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2121 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty],`.
  **L2122 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty],`。
- **L2123 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2123 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2124 EN**: Declares TableGen def `int_ppc_fnmadd`.
  **L2124 CN**: 声明 TableGen def `int_ppc_fnmadd`。
- **L2125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fnmadd">,`.
  **L2125 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fnmadd">,`。
- **L2126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty],`.
  **L2126 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty],`。
- **L2127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty, llvm_double_ty],`.
  **L2127 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty, llvm_double_ty],`。
- **L2128 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2128 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 2129-2156

````tablegen
  def int_ppc_fnmadds
      : ClangBuiltin<"__builtin_ppc_fnmadds">,
        DefaultAttrsIntrinsic<[llvm_float_ty],
                              [llvm_float_ty, llvm_float_ty, llvm_float_ty],
                              [IntrNoMem]>;
  def int_ppc_fnmsub
      : DefaultAttrsIntrinsic<[llvm_anyfloat_ty],
                              [LLVMMatchType<0>, LLVMMatchType<0>,
                               LLVMMatchType<0>],
                              [IntrNoMem]>;
  def int_ppc_fre
      : ClangBuiltin<"__builtin_ppc_fre">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fres
      : ClangBuiltin<"__builtin_ppc_fres">,
        DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;
  def int_ppc_addex
      : ClangBuiltin<"__builtin_ppc_addex">,
        DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],
                              [IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<2>>]>;
  def int_ppc_fsel : ClangBuiltin<"__builtin_ppc_fsel">,
      DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty, 
                             llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fsels : ClangBuiltin<"__builtin_ppc_fsels">,
      DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty,
                             llvm_float_ty], [IntrNoMem]>;
  def int_ppc_frsqrte : ClangBuiltin<"__builtin_ppc_frsqrte">,
      DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
````
- **L2129 EN**: Declares TableGen def `int_ppc_fnmadds`.
  **L2129 CN**: 声明 TableGen def `int_ppc_fnmadds`。
- **L2130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fnmadds">,`.
  **L2130 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fnmadds">,`。
- **L2131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty],`.
  **L2131 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty],`。
- **L2132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_float_ty, llvm_float_ty, llvm_float_ty],`.
  **L2132 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_float_ty, llvm_float_ty, llvm_float_ty],`。
- **L2133 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2133 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2134 EN**: Declares TableGen def `int_ppc_fnmsub`.
  **L2134 CN**: 声明 TableGen def `int_ppc_fnmsub`。
- **L2135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_anyfloat_ty],`.
  **L2135 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_anyfloat_ty],`。
- **L2136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[LLVMMatchType<0>, LLVMMatchType<0>,`.
  **L2136 CN**: 继续一个多行参数列表、初始化器或聚合项：`[LLVMMatchType<0>, LLVMMatchType<0>,`。
- **L2137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMMatchType<0>],`.
  **L2137 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMMatchType<0>],`。
- **L2138 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2138 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2139 EN**: Declares TableGen def `int_ppc_fre`.
  **L2139 CN**: 声明 TableGen def `int_ppc_fre`。
- **L2140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fre">,`.
  **L2140 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fre">,`。
- **L2141 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L2141 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L2142 EN**: Declares TableGen def `int_ppc_fres`.
  **L2142 CN**: 声明 TableGen def `int_ppc_fres`。
- **L2143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fres">,`.
  **L2143 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fres">,`。
- **L2144 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;`.
  **L2144 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;`。
- **L2145 EN**: Declares TableGen def `int_ppc_addex`.
  **L2145 CN**: 声明 TableGen def `int_ppc_addex`。
- **L2146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_addex">,`.
  **L2146 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_addex">,`。
- **L2147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`.
  **L2147 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i32_ty],`。
- **L2148 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<2>>]>;`.
  **L2148 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects, ImmArg<ArgIndex<2>>]>;`。
- **L2149 EN**: Declares TableGen def `int_ppc_fsel`.
  **L2149 CN**: 声明 TableGen def `int_ppc_fsel`。
- **L2150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty,`.
  **L2150 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty,`。
- **L2151 EN**: Executes a standalone statement or declaration: `llvm_double_ty], [IntrNoMem]>;`.
  **L2151 CN**: 执行一条独立语句或声明：`llvm_double_ty], [IntrNoMem]>;`。
- **L2152 EN**: Declares TableGen def `int_ppc_fsels`.
  **L2152 CN**: 声明 TableGen def `int_ppc_fsels`。
- **L2153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty,`.
  **L2153 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty,`。
- **L2154 EN**: Executes a standalone statement or declaration: `llvm_float_ty], [IntrNoMem]>;`.
  **L2154 CN**: 执行一条独立语句或声明：`llvm_float_ty], [IntrNoMem]>;`。
- **L2155 EN**: Declares TableGen def `int_ppc_frsqrte`.
  **L2155 CN**: 声明 TableGen def `int_ppc_frsqrte`。
- **L2156 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L2156 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。

### Lines 2157-2184

````tablegen
  def int_ppc_frsqrtes : ClangBuiltin<"__builtin_ppc_frsqrtes">,
      DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;
  def int_ppc_compare_exp_uo : ClangBuiltin<"__builtin_ppc_compare_exp_uo">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_double_ty, llvm_double_ty],
                            [IntrNoMem]>;
  def int_ppc_compare_exp_lt : ClangBuiltin<"__builtin_ppc_compare_exp_lt">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], 
                            [llvm_double_ty, llvm_double_ty],
                            [IntrNoMem]>;
  def int_ppc_compare_exp_gt : ClangBuiltin<"__builtin_ppc_compare_exp_gt">,
      DefaultAttrsIntrinsic<[llvm_i32_ty],
                            [llvm_double_ty, llvm_double_ty],
                            [IntrNoMem]>;
  def int_ppc_compare_exp_eq : ClangBuiltin<"__builtin_ppc_compare_exp_eq">,
      DefaultAttrsIntrinsic<[llvm_i32_ty], 
                            [llvm_double_ty, llvm_double_ty],
                            [IntrNoMem]>;
  def int_ppc_test_data_class
      : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_anyfloat_ty, llvm_i32_ty],
                              [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_ppc_fnabs
      : ClangBuiltin<"__builtin_ppc_fnabs">,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;
  def int_ppc_fnabss
      : ClangBuiltin<"__builtin_ppc_fnabss">,
        DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;

````
- **L2157 EN**: Declares TableGen def `int_ppc_frsqrtes`.
  **L2157 CN**: 声明 TableGen def `int_ppc_frsqrtes`。
- **L2158 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;`.
  **L2158 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;`。
- **L2159 EN**: Declares TableGen def `int_ppc_compare_exp_uo`.
  **L2159 CN**: 声明 TableGen def `int_ppc_compare_exp_uo`。
- **L2160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L2160 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L2161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty],`.
  **L2161 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty],`。
- **L2162 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2162 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2163 EN**: Declares TableGen def `int_ppc_compare_exp_lt`.
  **L2163 CN**: 声明 TableGen def `int_ppc_compare_exp_lt`。
- **L2164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L2164 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L2165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty],`.
  **L2165 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty],`。
- **L2166 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2166 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2167 EN**: Declares TableGen def `int_ppc_compare_exp_gt`.
  **L2167 CN**: 声明 TableGen def `int_ppc_compare_exp_gt`。
- **L2168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L2168 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L2169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty],`.
  **L2169 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty],`。
- **L2170 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2170 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2171 EN**: Declares TableGen def `int_ppc_compare_exp_eq`.
  **L2171 CN**: 声明 TableGen def `int_ppc_compare_exp_eq`。
- **L2172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],`.
  **L2172 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],`。
- **L2173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_double_ty, llvm_double_ty],`.
  **L2173 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_double_ty, llvm_double_ty],`。
- **L2174 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2174 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2175 EN**: Declares TableGen def `int_ppc_test_data_class`.
  **L2175 CN**: 声明 TableGen def `int_ppc_test_data_class`。
- **L2176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_anyfloat_ty, llvm_i32_ty],`.
  **L2176 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_anyfloat_ty, llvm_i32_ty],`。
- **L2177 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L2177 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L2178 EN**: Declares TableGen def `int_ppc_fnabs`.
  **L2178 CN**: 声明 TableGen def `int_ppc_fnabs`。
- **L2179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fnabs">,`.
  **L2179 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fnabs">,`。
- **L2180 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`.
  **L2180 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty], [IntrNoMem]>;`。
- **L2181 EN**: Declares TableGen def `int_ppc_fnabss`.
  **L2181 CN**: 声明 TableGen def `int_ppc_fnabss`。
- **L2182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_ppc_fnabss">,`.
  **L2182 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_ppc_fnabss">,`。
- **L2183 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;`.
  **L2183 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty], [IntrNoMem]>;`。
- **L2184 EN**: Blank line separating nearby declarations or logic blocks.
  **L2184 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2185-2212

````tablegen
  def int_ppc_convert_f128_to_ppcf128
      : DefaultAttrsIntrinsic<[llvm_ppcf128_ty], [llvm_f128_ty], [IntrNoMem]>;
  def int_ppc_convert_ppcf128_to_f128
      : DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_ppcf128_ty], [IntrNoMem]>;
}

//===----------------------------------------------------------------------===//
// PowerPC Atomic Intrinsic Definitions.
let TargetPrefix = "ppc" in {
  class AtomicRMW128Intrinsic
    : Intrinsic<[llvm_i64_ty, llvm_i64_ty],
                [llvm_ptr_ty, llvm_i64_ty, llvm_i64_ty],
                [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_ppc_atomicrmw_xchg_i128 : AtomicRMW128Intrinsic;
  def int_ppc_atomicrmw_add_i128  : AtomicRMW128Intrinsic;
  def int_ppc_atomicrmw_sub_i128  : AtomicRMW128Intrinsic;
  def int_ppc_atomicrmw_and_i128  : AtomicRMW128Intrinsic;
  def int_ppc_atomicrmw_or_i128   : AtomicRMW128Intrinsic;
  def int_ppc_atomicrmw_xor_i128  : AtomicRMW128Intrinsic;
  def int_ppc_atomicrmw_nand_i128 : AtomicRMW128Intrinsic;
  def int_ppc_cmpxchg_i128 : Intrinsic<[llvm_i64_ty, llvm_i64_ty],
                                       [llvm_ptr_ty,
                                        llvm_i64_ty, llvm_i64_ty,
                                        llvm_i64_ty, llvm_i64_ty],
                                       [IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;
  def int_ppc_atomic_load_i128 :
    Intrinsic<[llvm_i64_ty, llvm_i64_ty],
              [llvm_ptr_ty],
````
- **L2185 EN**: Declares TableGen def `int_ppc_convert_f128_to_ppcf128`.
  **L2185 CN**: 声明 TableGen def `int_ppc_convert_f128_to_ppcf128`。
- **L2186 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_ppcf128_ty], [llvm_f128_ty], [IntrNoMem]>;`.
  **L2186 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_ppcf128_ty], [llvm_f128_ty], [IntrNoMem]>;`。
- **L2187 EN**: Declares TableGen def `int_ppc_convert_ppcf128_to_f128`.
  **L2187 CN**: 声明 TableGen def `int_ppc_convert_ppcf128_to_f128`。
- **L2188 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_ppcf128_ty], [IntrNoMem]>;`.
  **L2188 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_f128_ty], [llvm_ppcf128_ty], [IntrNoMem]>;`。
- **L2189 EN**: Closes the current lexical scope or compound statement.
  **L2189 CN**: 结束当前词法作用域或复合语句块。
- **L2190 EN**: Blank line separating nearby declarations or logic blocks.
  **L2190 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2191 EN**: Banner comment marking a file or section boundary.
  **L2191 CN**: 横幅注释，用于标记文件或章节边界。
- **L2192 EN**: Comment explains nearby logic, invariants, or intent: `PowerPC Atomic Intrinsic Definitions.`.
  **L2192 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PowerPC Atomic Intrinsic Definitions.`。
- **L2193 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2193 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2194 EN**: Declares class `AtomicRMW128Intrinsic`.
  **L2194 CN**: 声明 class `AtomicRMW128Intrinsic`。
- **L2195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[llvm_i64_ty, llvm_i64_ty],`.
  **L2195 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[llvm_i64_ty, llvm_i64_ty],`。
- **L2196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i64_ty, llvm_i64_ty],`.
  **L2196 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i64_ty, llvm_i64_ty],`。
- **L2197 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L2197 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L2198 EN**: Declares TableGen def `int_ppc_atomicrmw_xchg_i128`.
  **L2198 CN**: 声明 TableGen def `int_ppc_atomicrmw_xchg_i128`。
- **L2199 EN**: Declares TableGen def `int_ppc_atomicrmw_add_i128`.
  **L2199 CN**: 声明 TableGen def `int_ppc_atomicrmw_add_i128`。
- **L2200 EN**: Declares TableGen def `int_ppc_atomicrmw_sub_i128`.
  **L2200 CN**: 声明 TableGen def `int_ppc_atomicrmw_sub_i128`。
- **L2201 EN**: Declares TableGen def `int_ppc_atomicrmw_and_i128`.
  **L2201 CN**: 声明 TableGen def `int_ppc_atomicrmw_and_i128`。
- **L2202 EN**: Declares TableGen def `int_ppc_atomicrmw_or_i128`.
  **L2202 CN**: 声明 TableGen def `int_ppc_atomicrmw_or_i128`。
- **L2203 EN**: Declares TableGen def `int_ppc_atomicrmw_xor_i128`.
  **L2203 CN**: 声明 TableGen def `int_ppc_atomicrmw_xor_i128`。
- **L2204 EN**: Declares TableGen def `int_ppc_atomicrmw_nand_i128`.
  **L2204 CN**: 声明 TableGen def `int_ppc_atomicrmw_nand_i128`。
- **L2205 EN**: Declares TableGen def `int_ppc_cmpxchg_i128`.
  **L2205 CN**: 声明 TableGen def `int_ppc_cmpxchg_i128`。
- **L2206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty,`.
  **L2206 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty,`。
- **L2207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_i64_ty,`.
  **L2207 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_i64_ty,`。
- **L2208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_i64_ty],`.
  **L2208 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_i64_ty],`。
- **L2209 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L2209 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L2210 EN**: Declares TableGen def `int_ppc_atomic_load_i128`.
  **L2210 CN**: 声明 TableGen def `int_ppc_atomic_load_i128`。
- **L2211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty, llvm_i64_ty],`.
  **L2211 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty, llvm_i64_ty],`。
- **L2212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty],`.
  **L2212 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty],`。

### Lines 2213-2240

````tablegen
              [IntrArgMemOnly, IntrReadMem, NoCapture<ArgIndex<0>>]>;
  def int_ppc_atomic_store_i128 :
    Intrinsic<[], [llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],
              [IntrArgMemOnly, IntrWriteMem, NoCapture<ArgIndex<2>>]>;
}

// AMO intrisics
let TargetPrefix = "ppc" in {
 def int_ppc_amo_lwat : ClangBuiltin<"__builtin_amo_lwat">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,
                           llvm_i32_ty, llvm_i32_ty],
                          [IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;
 def int_ppc_amo_ldat : ClangBuiltin<"__builtin_amo_ldat">,
    DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,
                           llvm_i64_ty, llvm_i32_ty],
                          [IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;
  def int_ppc_amo_lwat_cond : ClangBuiltin<"__builtin_amo_lwat_cond">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,
                           llvm_i32_ty],
                          [IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;
 def int_ppc_amo_ldat_cond : ClangBuiltin<"__builtin_amo_ldat_cond">,
    DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,
                           llvm_i32_ty],
                          [IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;

  def int_ppc_amo_stwat : ClangBuiltin<"__builtin_amo_stwat">,
    DefaultAttrsIntrinsic<[],[llvm_ptr_ty,
                           llvm_i32_ty, llvm_i32_ty],
````
- **L2213 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, IntrReadMem, NoCapture<ArgIndex<0>>]>;`.
  **L2213 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, IntrReadMem, NoCapture<ArgIndex<0>>]>;`。
- **L2214 EN**: Declares TableGen def `int_ppc_atomic_store_i128`.
  **L2214 CN**: 声明 TableGen def `int_ppc_atomic_store_i128`。
- **L2215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],`.
  **L2215 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_i64_ty, llvm_i64_ty, llvm_ptr_ty],`。
- **L2216 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, IntrWriteMem, NoCapture<ArgIndex<2>>]>;`.
  **L2216 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, IntrWriteMem, NoCapture<ArgIndex<2>>]>;`。
- **L2217 EN**: Closes the current lexical scope or compound statement.
  **L2217 CN**: 结束当前词法作用域或复合语句块。
- **L2218 EN**: Blank line separating nearby declarations or logic blocks.
  **L2218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2219 EN**: Comment explains nearby logic, invariants, or intent: `AMO intrisics`.
  **L2219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`AMO intrisics`。
- **L2220 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2220 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2221 EN**: Declares TableGen def `int_ppc_amo_lwat`.
  **L2221 CN**: 声明 TableGen def `int_ppc_amo_lwat`。
- **L2222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,`.
  **L2222 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,`。
- **L2223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L2223 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L2224 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L2224 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L2225 EN**: Declares TableGen def `int_ppc_amo_ldat`.
  **L2225 CN**: 声明 TableGen def `int_ppc_amo_ldat`。
- **L2226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,`.
  **L2226 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,`。
- **L2227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_i32_ty],`.
  **L2227 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_i32_ty],`。
- **L2228 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L2228 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L2229 EN**: Declares TableGen def `int_ppc_amo_lwat_cond`.
  **L2229 CN**: 声明 TableGen def `int_ppc_amo_lwat_cond`。
- **L2230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,`.
  **L2230 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,`。
- **L2231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L2231 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L2232 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L2232 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L2233 EN**: Declares TableGen def `int_ppc_amo_ldat_cond`.
  **L2233 CN**: 声明 TableGen def `int_ppc_amo_ldat_cond`。
- **L2234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,`.
  **L2234 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,`。
- **L2235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L2235 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L2236 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L2236 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L2237 EN**: Blank line separating nearby declarations or logic blocks.
  **L2237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2238 EN**: Declares TableGen def `int_ppc_amo_stwat`.
  **L2238 CN**: 声明 TableGen def `int_ppc_amo_stwat`。
- **L2239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[],[llvm_ptr_ty,`.
  **L2239 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[],[llvm_ptr_ty,`。
- **L2240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L2240 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。

### Lines 2241-2268

````tablegen
                          [IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;
 def int_ppc_amo_stdat : ClangBuiltin<"__builtin_amo_stdat">,
    DefaultAttrsIntrinsic<[],[llvm_ptr_ty,
                           llvm_i64_ty, llvm_i32_ty],
                          [IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;
 def int_ppc_amo_lwat_csne : ClangBuiltin<"__builtin_amo_lwat_csne">,
    DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,
                           llvm_i32_ty, llvm_i32_ty],
                          [IntrArgMemOnly]>;
 def int_ppc_amo_ldat_csne : ClangBuiltin<"__builtin_amo_ldat_csne">,
    DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,
                           llvm_i64_ty, llvm_i64_ty],
                          [IntrArgMemOnly]>;

  // Elliptic Curve Cryptography Intrinsics
  def int_ppc_xxmulmul : ClangBuiltin<"__builtin_xxmulmul">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_ppc_xxmulmulhiadd : ClangBuiltin<"__builtin_xxmulmulhiadd">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
                             llvm_i32_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>,
                             ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;
  def int_ppc_xxmulmulloadd : ClangBuiltin<"__builtin_xxmulmulloadd">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,
````
- **L2241 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L2241 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L2242 EN**: Declares TableGen def `int_ppc_amo_stdat`.
  **L2242 CN**: 声明 TableGen def `int_ppc_amo_stdat`。
- **L2243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[],[llvm_ptr_ty,`.
  **L2243 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[],[llvm_ptr_ty,`。
- **L2244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_i32_ty],`.
  **L2244 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_i32_ty],`。
- **L2245 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`.
  **L2245 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, ImmArg<ArgIndex<2>>]>;`。
- **L2246 EN**: Declares TableGen def `int_ppc_amo_lwat_csne`.
  **L2246 CN**: 声明 TableGen def `int_ppc_amo_lwat_csne`。
- **L2247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,`.
  **L2247 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty],[llvm_ptr_ty,`。
- **L2248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L2248 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L2249 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L2249 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L2250 EN**: Declares TableGen def `int_ppc_amo_ldat_csne`.
  **L2250 CN**: 声明 TableGen def `int_ppc_amo_ldat_csne`。
- **L2251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,`.
  **L2251 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i64_ty],[llvm_ptr_ty,`。
- **L2252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i64_ty, llvm_i64_ty],`.
  **L2252 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i64_ty, llvm_i64_ty],`。
- **L2253 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly]>;`.
  **L2253 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly]>;`。
- **L2254 EN**: Blank line separating nearby declarations or logic blocks.
  **L2254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2255 EN**: Comment explains nearby logic, invariants, or intent: `Elliptic Curve Cryptography Intrinsics`.
  **L2255 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Elliptic Curve Cryptography Intrinsics`。
- **L2256 EN**: Declares TableGen def `int_ppc_xxmulmul`.
  **L2256 CN**: 声明 TableGen def `int_ppc_xxmulmul`。
- **L2257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2257 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L2258 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L2259 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L2259 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L2260 EN**: Declares TableGen def `int_ppc_xxmulmulhiadd`.
  **L2260 CN**: 声明 TableGen def `int_ppc_xxmulmulhiadd`。
- **L2261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2261 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L2262 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。
- **L2263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty, llvm_i32_ty],`.
  **L2263 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty, llvm_i32_ty],`。
- **L2264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, ImmArg<ArgIndex<2>>,`.
  **L2264 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, ImmArg<ArgIndex<2>>,`。
- **L2265 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`.
  **L2265 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>, ImmArg<ArgIndex<4>>]>;`。
- **L2266 EN**: Declares TableGen def `int_ppc_xxmulmulloadd`.
  **L2266 CN**: 声明 TableGen def `int_ppc_xxmulmulloadd`。
- **L2267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2267 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`.
  **L2268 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty,`。

### Lines 2269-2296

````tablegen
                             llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>,
                             ImmArg<ArgIndex<3>>]>;
  def int_ppc_xxssumudm : ClangBuiltin<"__builtin_xxssumudm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_ppc_xxssumudmc : ClangBuiltin<"__builtin_xxssumudmc">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_ppc_xxssumudmcext : ClangBuiltin<"__builtin_xxssumudmcext">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,
                             llvm_i32_ty],
                            [IntrNoMem, ImmArg<ArgIndex<3>>]>;
  def int_ppc_xsaddadduqm : ClangBuiltin<"__builtin_xsaddadduqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsaddaddsuqm : ClangBuiltin<"__builtin_xsaddaddsuqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsaddsubuqm : ClangBuiltin<"__builtin_xsaddsubuqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
````
- **L2269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L2269 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L2270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrNoMem, ImmArg<ArgIndex<2>>,`.
  **L2270 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrNoMem, ImmArg<ArgIndex<2>>,`。
- **L2271 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<3>>]>;`.
  **L2271 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<3>>]>;`。
- **L2272 EN**: Declares TableGen def `int_ppc_xxssumudm`.
  **L2272 CN**: 声明 TableGen def `int_ppc_xxssumudm`。
- **L2273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2273 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L2274 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L2275 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L2275 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L2276 EN**: Declares TableGen def `int_ppc_xxssumudmc`.
  **L2276 CN**: 声明 TableGen def `int_ppc_xxssumudmc`。
- **L2277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2277 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L2278 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L2279 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L2279 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L2280 EN**: Declares TableGen def `int_ppc_xxssumudmcext`.
  **L2280 CN**: 声明 TableGen def `int_ppc_xxssumudmcext`。
- **L2281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2281 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`.
  **L2282 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty,`。
- **L2283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty],`.
  **L2283 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty],`。
- **L2284 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L2284 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L2285 EN**: Declares TableGen def `int_ppc_xsaddadduqm`.
  **L2285 CN**: 声明 TableGen def `int_ppc_xsaddadduqm`。
- **L2286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2286 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2287 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2288 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2288 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2289 EN**: Declares TableGen def `int_ppc_xsaddaddsuqm`.
  **L2289 CN**: 声明 TableGen def `int_ppc_xsaddaddsuqm`。
- **L2290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2290 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2291 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2292 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2292 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2293 EN**: Declares TableGen def `int_ppc_xsaddsubuqm`.
  **L2293 CN**: 声明 TableGen def `int_ppc_xsaddsubuqm`。
- **L2294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2294 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2295 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2296 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2296 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 2297-2324

````tablegen
  def int_ppc_xsaddsubsuqm : ClangBuiltin<"__builtin_xsaddsubsuqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsmerge2t1uqm : ClangBuiltin<"__builtin_xsmerge2t1uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsmerge2t2uqm : ClangBuiltin<"__builtin_xsmerge2t2uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsmerge2t3uqm : ClangBuiltin<"__builtin_xsmerge2t3uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsmerge3t1uqm : ClangBuiltin<"__builtin_xsmerge3t1uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsrebase2t1uqm : ClangBuiltin<"__builtin_xsrebase2t1uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsrebase2t2uqm : ClangBuiltin<"__builtin_xsrebase2t2uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
````
- **L2297 EN**: Declares TableGen def `int_ppc_xsaddsubsuqm`.
  **L2297 CN**: 声明 TableGen def `int_ppc_xsaddsubsuqm`。
- **L2298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2298 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2299 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2300 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2300 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2301 EN**: Declares TableGen def `int_ppc_xsmerge2t1uqm`.
  **L2301 CN**: 声明 TableGen def `int_ppc_xsmerge2t1uqm`。
- **L2302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2302 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2303 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2304 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2304 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2305 EN**: Declares TableGen def `int_ppc_xsmerge2t2uqm`.
  **L2305 CN**: 声明 TableGen def `int_ppc_xsmerge2t2uqm`。
- **L2306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2306 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2307 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2308 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2308 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2309 EN**: Declares TableGen def `int_ppc_xsmerge2t3uqm`.
  **L2309 CN**: 声明 TableGen def `int_ppc_xsmerge2t3uqm`。
- **L2310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2310 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2311 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2312 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2312 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2313 EN**: Declares TableGen def `int_ppc_xsmerge3t1uqm`.
  **L2313 CN**: 声明 TableGen def `int_ppc_xsmerge3t1uqm`。
- **L2314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2314 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2315 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2316 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2316 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2317 EN**: Declares TableGen def `int_ppc_xsrebase2t1uqm`.
  **L2317 CN**: 声明 TableGen def `int_ppc_xsrebase2t1uqm`。
- **L2318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2318 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2319 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2320 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2320 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2321 EN**: Declares TableGen def `int_ppc_xsrebase2t2uqm`.
  **L2321 CN**: 声明 TableGen def `int_ppc_xsrebase2t2uqm`。
- **L2322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2322 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2323 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2324 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2324 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。

### Lines 2325-2345

````tablegen
  def int_ppc_xsrebase2t3uqm : ClangBuiltin<"__builtin_xsrebase2t3uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsrebase2t4uqm : ClangBuiltin<"__builtin_xsrebase2t4uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsrebase3t1uqm : ClangBuiltin<"__builtin_xsrebase3t1uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsrebase3t2uqm : ClangBuiltin<"__builtin_xsrebase3t2uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
  def int_ppc_xsrebase3t3uqm : ClangBuiltin<"__builtin_xsrebase3t3uqm">,
      DefaultAttrsIntrinsic<[llvm_v16i8_ty],
                            [llvm_v16i8_ty, llvm_v16i8_ty],
                            [IntrNoMem]>;
}
````
- **L2325 EN**: Declares TableGen def `int_ppc_xsrebase2t3uqm`.
  **L2325 CN**: 声明 TableGen def `int_ppc_xsrebase2t3uqm`。
- **L2326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2326 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2327 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2328 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2328 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2329 EN**: Declares TableGen def `int_ppc_xsrebase2t4uqm`.
  **L2329 CN**: 声明 TableGen def `int_ppc_xsrebase2t4uqm`。
- **L2330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2330 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2331 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2332 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2332 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2333 EN**: Declares TableGen def `int_ppc_xsrebase3t1uqm`.
  **L2333 CN**: 声明 TableGen def `int_ppc_xsrebase3t1uqm`。
- **L2334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2334 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2335 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2336 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2336 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2337 EN**: Declares TableGen def `int_ppc_xsrebase3t2uqm`.
  **L2337 CN**: 声明 TableGen def `int_ppc_xsrebase3t2uqm`。
- **L2338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2338 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2339 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2340 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2340 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2341 EN**: Declares TableGen def `int_ppc_xsrebase3t3uqm`.
  **L2341 CN**: 声明 TableGen def `int_ppc_xsrebase3t3uqm`。
- **L2342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty],`.
  **L2342 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty],`。
- **L2343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L2343 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L2344 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L2344 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L2345 EN**: Closes the current lexical scope or compound statement.
  **L2345 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Function-level IR management / 函数级 IR 管理**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
