# IntrinsicsSystemZ.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsSystemZ.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the SystemZ-specific intrinsics.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsSystemZ` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````tablegen
//===- IntrinsicsSystemZ.td - Defines SystemZ intrinsics ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the SystemZ-specific intrinsics.
//
//===----------------------------------------------------------------------===//

class SystemZUnaryConv<string name, LLVMType result, LLVMType arg>
  : ClangBuiltin<"__builtin_s390_" # name>,
    Intrinsic<[result], [arg], [IntrNoMem]>;

class SystemZUnary<string name, LLVMType type>
  : SystemZUnaryConv<name, type, type>;

class SystemZUnaryConvCC<LLVMType result, LLVMType arg>
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the SystemZ-specific intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the SystemZ-specific intrinsics.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Declares class `SystemZUnaryConv<string`.
  **L13 CN**: 声明 class `SystemZUnaryConv<string`。
- **L14 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_s390_" # name>,`.
  **L14 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_s390_" # name>,`。
- **L15 EN**: Executes a standalone statement or declaration: `Intrinsic<[result], [arg], [IntrNoMem]>;`.
  **L15 CN**: 执行一条独立语句或声明：`Intrinsic<[result], [arg], [IntrNoMem]>;`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Declares class `SystemZUnary<string`.
  **L17 CN**: 声明 class `SystemZUnary<string`。
- **L18 EN**: Executes a standalone statement or declaration: `: SystemZUnaryConv<name, type, type>;`.
  **L18 CN**: 执行一条独立语句或声明：`: SystemZUnaryConv<name, type, type>;`。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Declares class `SystemZUnaryConvCC<LLVMType`.
  **L20 CN**: 声明 class `SystemZUnaryConvCC<LLVMType`。

### Lines 21-40

````tablegen
  : Intrinsic<[result, llvm_i32_ty], [arg], [IntrNoMem]>;

class SystemZUnaryCC<LLVMType type>
  : SystemZUnaryConvCC<type, type>;

class SystemZBinaryConv<string name, LLVMType result, LLVMType arg>
  : ClangBuiltin<"__builtin_s390_" # name>,
    Intrinsic<[result], [arg, arg], [IntrNoMem]>;

class SystemZBinary<string name, LLVMType type>
  : SystemZBinaryConv<name, type, type>;

class SystemZBinaryConvCC<LLVMType result, LLVMType arg>
  : Intrinsic<[result, llvm_i32_ty], [arg, arg], [IntrNoMem]>;

class SystemZBinaryConvIntCC<LLVMType result, LLVMType arg>
  : Intrinsic<[result, llvm_i32_ty], [arg, llvm_i32_ty],
              [IntrNoMem, ImmArg<ArgIndex<1>>]>;

class SystemZBinaryCC<LLVMType type>
````
- **L21 EN**: Executes a standalone statement or declaration: `: Intrinsic<[result, llvm_i32_ty], [arg], [IntrNoMem]>;`.
  **L21 CN**: 执行一条独立语句或声明：`: Intrinsic<[result, llvm_i32_ty], [arg], [IntrNoMem]>;`。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Declares class `SystemZUnaryCC<LLVMType`.
  **L23 CN**: 声明 class `SystemZUnaryCC<LLVMType`。
- **L24 EN**: Executes a standalone statement or declaration: `: SystemZUnaryConvCC<type, type>;`.
  **L24 CN**: 执行一条独立语句或声明：`: SystemZUnaryConvCC<type, type>;`。
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Declares class `SystemZBinaryConv<string`.
  **L26 CN**: 声明 class `SystemZBinaryConv<string`。
- **L27 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_s390_" # name>,`.
  **L27 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_s390_" # name>,`。
- **L28 EN**: Executes a standalone statement or declaration: `Intrinsic<[result], [arg, arg], [IntrNoMem]>;`.
  **L28 CN**: 执行一条独立语句或声明：`Intrinsic<[result], [arg, arg], [IntrNoMem]>;`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Declares class `SystemZBinary<string`.
  **L30 CN**: 声明 class `SystemZBinary<string`。
- **L31 EN**: Executes a standalone statement or declaration: `: SystemZBinaryConv<name, type, type>;`.
  **L31 CN**: 执行一条独立语句或声明：`: SystemZBinaryConv<name, type, type>;`。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Declares class `SystemZBinaryConvCC<LLVMType`.
  **L33 CN**: 声明 class `SystemZBinaryConvCC<LLVMType`。
- **L34 EN**: Executes a standalone statement or declaration: `: Intrinsic<[result, llvm_i32_ty], [arg, arg], [IntrNoMem]>;`.
  **L34 CN**: 执行一条独立语句或声明：`: Intrinsic<[result, llvm_i32_ty], [arg, arg], [IntrNoMem]>;`。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Declares class `SystemZBinaryConvIntCC<LLVMType`.
  **L36 CN**: 声明 class `SystemZBinaryConvIntCC<LLVMType`。
- **L37 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[result, llvm_i32_ty], [arg, llvm_i32_ty],`.
  **L37 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[result, llvm_i32_ty], [arg, llvm_i32_ty],`。
- **L38 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L38 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Declares class `SystemZBinaryCC<LLVMType`.
  **L40 CN**: 声明 class `SystemZBinaryCC<LLVMType`。

### Lines 41-60

````tablegen
  : SystemZBinaryConvCC<type, type>;

class SystemZTernaryConv<string name, LLVMType result, LLVMType arg>
  : ClangBuiltin<"__builtin_s390_" # name>,
    Intrinsic<[result], [arg, arg, result], [IntrNoMem]>;

class SystemZTernaryConvCC<LLVMType result, LLVMType arg>
  : Intrinsic<[result, llvm_i32_ty], [arg, arg, result], [IntrNoMem]>;

class SystemZTernary<string name, LLVMType type>
  : SystemZTernaryConv<name, type, type>;

class SystemZTernaryInt<string name, LLVMType type>
  : ClangBuiltin<"__builtin_s390_" # name>,
    Intrinsic<[type], [type, type, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<2>>]>;

class SystemZTernaryIntCC<LLVMType type>
  : Intrinsic<[type, llvm_i32_ty], [type, type, llvm_i32_ty],
              [IntrNoMem, ImmArg<ArgIndex<2>>]>;

````
- **L41 EN**: Executes a standalone statement or declaration: `: SystemZBinaryConvCC<type, type>;`.
  **L41 CN**: 执行一条独立语句或声明：`: SystemZBinaryConvCC<type, type>;`。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Declares class `SystemZTernaryConv<string`.
  **L43 CN**: 声明 class `SystemZTernaryConv<string`。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_s390_" # name>,`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_s390_" # name>,`。
- **L45 EN**: Executes a standalone statement or declaration: `Intrinsic<[result], [arg, arg, result], [IntrNoMem]>;`.
  **L45 CN**: 执行一条独立语句或声明：`Intrinsic<[result], [arg, arg, result], [IntrNoMem]>;`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Declares class `SystemZTernaryConvCC<LLVMType`.
  **L47 CN**: 声明 class `SystemZTernaryConvCC<LLVMType`。
- **L48 EN**: Executes a standalone statement or declaration: `: Intrinsic<[result, llvm_i32_ty], [arg, arg, result], [IntrNoMem]>;`.
  **L48 CN**: 执行一条独立语句或声明：`: Intrinsic<[result, llvm_i32_ty], [arg, arg, result], [IntrNoMem]>;`。
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Declares class `SystemZTernary<string`.
  **L50 CN**: 声明 class `SystemZTernary<string`。
- **L51 EN**: Executes a standalone statement or declaration: `: SystemZTernaryConv<name, type, type>;`.
  **L51 CN**: 执行一条独立语句或声明：`: SystemZTernaryConv<name, type, type>;`。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Declares class `SystemZTernaryInt<string`.
  **L53 CN**: 声明 class `SystemZTernaryInt<string`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_s390_" # name>,`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_s390_" # name>,`。
- **L55 EN**: Executes a standalone statement or declaration: `Intrinsic<[type], [type, type, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L55 CN**: 执行一条独立语句或声明：`Intrinsic<[type], [type, type, llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Declares class `SystemZTernaryIntCC<LLVMType`.
  **L57 CN**: 声明 class `SystemZTernaryIntCC<LLVMType`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[type, llvm_i32_ty], [type, type, llvm_i32_ty],`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[type, llvm_i32_ty], [type, type, llvm_i32_ty],`。
- **L59 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L59 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 61-80

````tablegen
class SystemZQuaternaryInt<string name, LLVMType type>
  : ClangBuiltin<"__builtin_s390_" # name>,
    Intrinsic<[type], [type, type, type, llvm_i32_ty],
    [IntrNoMem, ImmArg<ArgIndex<3>>]>;

class SystemZQuaternaryIntCC<LLVMType type>
  : Intrinsic<[type, llvm_i32_ty], [type, type, type, llvm_i32_ty],
              [IntrNoMem, ImmArg<ArgIndex<3>>]>;

multiclass SystemZUnaryExtBHF<string name> {
  def b : SystemZUnaryConv<name#"b", llvm_v8i16_ty, llvm_v16i8_ty>;
  def h : SystemZUnaryConv<name#"h", llvm_v4i32_ty, llvm_v8i16_ty>;
  def f : SystemZUnaryConv<name#"f", llvm_v2i64_ty, llvm_v4i32_ty>;
}

multiclass SystemZUnaryExtBHWF<string name> {
  def b  : SystemZUnaryConv<name#"b",  llvm_v8i16_ty, llvm_v16i8_ty>;
  def hw : SystemZUnaryConv<name#"hw", llvm_v4i32_ty, llvm_v8i16_ty>;
  def f  : SystemZUnaryConv<name#"f",  llvm_v2i64_ty, llvm_v4i32_ty>;
}
````
- **L61 EN**: Declares class `SystemZQuaternaryInt<string`.
  **L61 CN**: 声明 class `SystemZQuaternaryInt<string`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: ClangBuiltin<"__builtin_s390_" # name>,`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`: ClangBuiltin<"__builtin_s390_" # name>,`。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[type], [type, type, type, llvm_i32_ty],`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[type], [type, type, type, llvm_i32_ty],`。
- **L64 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L64 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L66 EN**: Declares class `SystemZQuaternaryIntCC<LLVMType`.
  **L66 CN**: 声明 class `SystemZQuaternaryIntCC<LLVMType`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[type, llvm_i32_ty], [type, type, type, llvm_i32_ty],`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[type, llvm_i32_ty], [type, type, type, llvm_i32_ty],`。
- **L68 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L68 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Declares TableGen multiclass `SystemZUnaryExtBHF`.
  **L70 CN**: 声明 TableGen multiclass `SystemZUnaryExtBHF`。
- **L71 EN**: Declares TableGen def `b`.
  **L71 CN**: 声明 TableGen def `b`。
- **L72 EN**: Declares TableGen def `h`.
  **L72 CN**: 声明 TableGen def `h`。
- **L73 EN**: Declares TableGen def `f`.
  **L73 CN**: 声明 TableGen def `f`。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Declares TableGen multiclass `SystemZUnaryExtBHWF`.
  **L76 CN**: 声明 TableGen multiclass `SystemZUnaryExtBHWF`。
- **L77 EN**: Declares TableGen def `b`.
  **L77 CN**: 声明 TableGen def `b`。
- **L78 EN**: Declares TableGen def `hw`.
  **L78 CN**: 声明 TableGen def `hw`。
- **L79 EN**: Declares TableGen def `f`.
  **L79 CN**: 声明 TableGen def `f`。
- **L80 EN**: Closes the current lexical scope or compound statement.
  **L80 CN**: 结束当前词法作用域或复合语句块。

### Lines 81-100

````tablegen

multiclass SystemZUnaryBHF<string name> {
  def b : SystemZUnary<name#"b", llvm_v16i8_ty>;
  def h : SystemZUnary<name#"h", llvm_v8i16_ty>;
  def f : SystemZUnary<name#"f", llvm_v4i32_ty>;
}

multiclass SystemZUnaryBHFG<string name> : SystemZUnaryBHF<name> {
  def g : SystemZUnary<name#"g", llvm_v2i64_ty>;
}

multiclass SystemZUnaryCCBHF {
  def bs : SystemZUnaryCC<llvm_v16i8_ty>;
  def hs : SystemZUnaryCC<llvm_v8i16_ty>;
  def fs : SystemZUnaryCC<llvm_v4i32_ty>;
}

multiclass SystemZBinaryTruncHFG<string name> {
  def h : SystemZBinaryConv<name#"h", llvm_v16i8_ty, llvm_v8i16_ty>;
  def f : SystemZBinaryConv<name#"f", llvm_v8i16_ty, llvm_v4i32_ty>;
````
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Declares TableGen multiclass `SystemZUnaryBHF`.
  **L82 CN**: 声明 TableGen multiclass `SystemZUnaryBHF`。
- **L83 EN**: Declares TableGen def `b`.
  **L83 CN**: 声明 TableGen def `b`。
- **L84 EN**: Declares TableGen def `h`.
  **L84 CN**: 声明 TableGen def `h`。
- **L85 EN**: Declares TableGen def `f`.
  **L85 CN**: 声明 TableGen def `f`。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L88 EN**: Declares TableGen multiclass `SystemZUnaryBHFG`.
  **L88 CN**: 声明 TableGen multiclass `SystemZUnaryBHFG`。
- **L89 EN**: Declares TableGen def `g`.
  **L89 CN**: 声明 TableGen def `g`。
- **L90 EN**: Closes the current lexical scope or compound statement.
  **L90 CN**: 结束当前词法作用域或复合语句块。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Declares TableGen multiclass `SystemZUnaryCCBHF`.
  **L92 CN**: 声明 TableGen multiclass `SystemZUnaryCCBHF`。
- **L93 EN**: Declares TableGen def `bs`.
  **L93 CN**: 声明 TableGen def `bs`。
- **L94 EN**: Declares TableGen def `hs`.
  **L94 CN**: 声明 TableGen def `hs`。
- **L95 EN**: Declares TableGen def `fs`.
  **L95 CN**: 声明 TableGen def `fs`。
- **L96 EN**: Closes the current lexical scope or compound statement.
  **L96 CN**: 结束当前词法作用域或复合语句块。
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Declares TableGen multiclass `SystemZBinaryTruncHFG`.
  **L98 CN**: 声明 TableGen multiclass `SystemZBinaryTruncHFG`。
- **L99 EN**: Declares TableGen def `h`.
  **L99 CN**: 声明 TableGen def `h`。
- **L100 EN**: Declares TableGen def `f`.
  **L100 CN**: 声明 TableGen def `f`。

### Lines 101-120

````tablegen
  def g : SystemZBinaryConv<name#"g", llvm_v4i32_ty, llvm_v2i64_ty>;
}

multiclass SystemZBinaryTruncCCHFG {
  def hs : SystemZBinaryConvCC<llvm_v16i8_ty, llvm_v8i16_ty>;
  def fs : SystemZBinaryConvCC<llvm_v8i16_ty, llvm_v4i32_ty>;
  def gs : SystemZBinaryConvCC<llvm_v4i32_ty, llvm_v2i64_ty>;
}

multiclass SystemZBinaryExtBHF<string name> {
  def b : SystemZBinaryConv<name#"b", llvm_v8i16_ty, llvm_v16i8_ty>;
  def h : SystemZBinaryConv<name#"h", llvm_v4i32_ty, llvm_v8i16_ty>;
  def f : SystemZBinaryConv<name#"f", llvm_v2i64_ty, llvm_v4i32_ty>;
}

multiclass SystemZBinaryExtBHFG<string name> : SystemZBinaryExtBHF<name> {
  def g : SystemZBinaryConv<name#"g", llvm_i128_ty, llvm_v2i64_ty>;
}

multiclass SystemZBinaryBHF<string name> {
````
- **L101 EN**: Declares TableGen def `g`.
  **L101 CN**: 声明 TableGen def `g`。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L104 EN**: Declares TableGen multiclass `SystemZBinaryTruncCCHFG`.
  **L104 CN**: 声明 TableGen multiclass `SystemZBinaryTruncCCHFG`。
- **L105 EN**: Declares TableGen def `hs`.
  **L105 CN**: 声明 TableGen def `hs`。
- **L106 EN**: Declares TableGen def `fs`.
  **L106 CN**: 声明 TableGen def `fs`。
- **L107 EN**: Declares TableGen def `gs`.
  **L107 CN**: 声明 TableGen def `gs`。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L110 EN**: Declares TableGen multiclass `SystemZBinaryExtBHF`.
  **L110 CN**: 声明 TableGen multiclass `SystemZBinaryExtBHF`。
- **L111 EN**: Declares TableGen def `b`.
  **L111 CN**: 声明 TableGen def `b`。
- **L112 EN**: Declares TableGen def `h`.
  **L112 CN**: 声明 TableGen def `h`。
- **L113 EN**: Declares TableGen def `f`.
  **L113 CN**: 声明 TableGen def `f`。
- **L114 EN**: Closes the current lexical scope or compound statement.
  **L114 CN**: 结束当前词法作用域或复合语句块。
- **L115 EN**: Blank line separating nearby declarations or logic blocks.
  **L115 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L116 EN**: Declares TableGen multiclass `SystemZBinaryExtBHFG`.
  **L116 CN**: 声明 TableGen multiclass `SystemZBinaryExtBHFG`。
- **L117 EN**: Declares TableGen def `g`.
  **L117 CN**: 声明 TableGen def `g`。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L120 EN**: Declares TableGen multiclass `SystemZBinaryBHF`.
  **L120 CN**: 声明 TableGen multiclass `SystemZBinaryBHF`。

### Lines 121-140

````tablegen
  def b : SystemZBinary<name#"b", llvm_v16i8_ty>;
  def h : SystemZBinary<name#"h", llvm_v8i16_ty>;
  def f : SystemZBinary<name#"f", llvm_v4i32_ty>;
}

multiclass SystemZBinaryBHFG<string name> : SystemZBinaryBHF<name> {
  def g : SystemZBinary<name#"g", llvm_v2i64_ty>;
}

multiclass SystemZBinaryCCBHF {
  def bs : SystemZBinaryCC<llvm_v16i8_ty>;
  def hs : SystemZBinaryCC<llvm_v8i16_ty>;
  def fs : SystemZBinaryCC<llvm_v4i32_ty>;
}

multiclass SystemZCompareBHFG {
  def bs : SystemZBinaryCC<llvm_v16i8_ty>;
  def hs : SystemZBinaryCC<llvm_v8i16_ty>;
  def fs : SystemZBinaryCC<llvm_v4i32_ty>;
  def gs : SystemZBinaryCC<llvm_v2i64_ty>;
````
- **L121 EN**: Declares TableGen def `b`.
  **L121 CN**: 声明 TableGen def `b`。
- **L122 EN**: Declares TableGen def `h`.
  **L122 CN**: 声明 TableGen def `h`。
- **L123 EN**: Declares TableGen def `f`.
  **L123 CN**: 声明 TableGen def `f`。
- **L124 EN**: Closes the current lexical scope or compound statement.
  **L124 CN**: 结束当前词法作用域或复合语句块。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L126 EN**: Declares TableGen multiclass `SystemZBinaryBHFG`.
  **L126 CN**: 声明 TableGen multiclass `SystemZBinaryBHFG`。
- **L127 EN**: Declares TableGen def `g`.
  **L127 CN**: 声明 TableGen def `g`。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L130 EN**: Declares TableGen multiclass `SystemZBinaryCCBHF`.
  **L130 CN**: 声明 TableGen multiclass `SystemZBinaryCCBHF`。
- **L131 EN**: Declares TableGen def `bs`.
  **L131 CN**: 声明 TableGen def `bs`。
- **L132 EN**: Declares TableGen def `hs`.
  **L132 CN**: 声明 TableGen def `hs`。
- **L133 EN**: Declares TableGen def `fs`.
  **L133 CN**: 声明 TableGen def `fs`。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Declares TableGen multiclass `SystemZCompareBHFG`.
  **L136 CN**: 声明 TableGen multiclass `SystemZCompareBHFG`。
- **L137 EN**: Declares TableGen def `bs`.
  **L137 CN**: 声明 TableGen def `bs`。
- **L138 EN**: Declares TableGen def `hs`.
  **L138 CN**: 声明 TableGen def `hs`。
- **L139 EN**: Declares TableGen def `fs`.
  **L139 CN**: 声明 TableGen def `fs`。
- **L140 EN**: Declares TableGen def `gs`.
  **L140 CN**: 声明 TableGen def `gs`。

### Lines 141-160

````tablegen
}

multiclass SystemZTernaryExtBHF<string name> {
  def b : SystemZTernaryConv<name#"b", llvm_v8i16_ty, llvm_v16i8_ty>;
  def h : SystemZTernaryConv<name#"h", llvm_v4i32_ty, llvm_v8i16_ty>;
  def f : SystemZTernaryConv<name#"f", llvm_v2i64_ty, llvm_v4i32_ty>;
}

multiclass SystemZTernaryExtBHFG<string name> : SystemZTernaryExtBHF<name> {
  def g : SystemZTernaryConv<name#"g", llvm_i128_ty, llvm_v2i64_ty>;
}

multiclass SystemZTernaryBHF<string name> {
  def b : SystemZTernary<name#"b", llvm_v16i8_ty>;
  def h : SystemZTernary<name#"h", llvm_v8i16_ty>;
  def f : SystemZTernary<name#"f", llvm_v4i32_ty>;
}

multiclass SystemZTernaryIntBHF<string name> {
  def b : SystemZTernaryInt<name#"b", llvm_v16i8_ty>;
````
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L143 EN**: Declares TableGen multiclass `SystemZTernaryExtBHF`.
  **L143 CN**: 声明 TableGen multiclass `SystemZTernaryExtBHF`。
- **L144 EN**: Declares TableGen def `b`.
  **L144 CN**: 声明 TableGen def `b`。
- **L145 EN**: Declares TableGen def `h`.
  **L145 CN**: 声明 TableGen def `h`。
- **L146 EN**: Declares TableGen def `f`.
  **L146 CN**: 声明 TableGen def `f`。
- **L147 EN**: Closes the current lexical scope or compound statement.
  **L147 CN**: 结束当前词法作用域或复合语句块。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L149 EN**: Declares TableGen multiclass `SystemZTernaryExtBHFG`.
  **L149 CN**: 声明 TableGen multiclass `SystemZTernaryExtBHFG`。
- **L150 EN**: Declares TableGen def `g`.
  **L150 CN**: 声明 TableGen def `g`。
- **L151 EN**: Closes the current lexical scope or compound statement.
  **L151 CN**: 结束当前词法作用域或复合语句块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L153 EN**: Declares TableGen multiclass `SystemZTernaryBHF`.
  **L153 CN**: 声明 TableGen multiclass `SystemZTernaryBHF`。
- **L154 EN**: Declares TableGen def `b`.
  **L154 CN**: 声明 TableGen def `b`。
- **L155 EN**: Declares TableGen def `h`.
  **L155 CN**: 声明 TableGen def `h`。
- **L156 EN**: Declares TableGen def `f`.
  **L156 CN**: 声明 TableGen def `f`。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L159 EN**: Declares TableGen multiclass `SystemZTernaryIntBHF`.
  **L159 CN**: 声明 TableGen multiclass `SystemZTernaryIntBHF`。
- **L160 EN**: Declares TableGen def `b`.
  **L160 CN**: 声明 TableGen def `b`。

### Lines 161-180

````tablegen
  def h : SystemZTernaryInt<name#"h", llvm_v8i16_ty>;
  def f : SystemZTernaryInt<name#"f", llvm_v4i32_ty>;
}

multiclass SystemZTernaryIntCCBHF {
  def bs : SystemZTernaryIntCC<llvm_v16i8_ty>;
  def hs : SystemZTernaryIntCC<llvm_v8i16_ty>;
  def fs : SystemZTernaryIntCC<llvm_v4i32_ty>;
}

multiclass SystemZQuaternaryIntBHF<string name> {
  def b : SystemZQuaternaryInt<name#"b", llvm_v16i8_ty>;
  def h : SystemZQuaternaryInt<name#"h", llvm_v8i16_ty>;
  def f : SystemZQuaternaryInt<name#"f", llvm_v4i32_ty>;
}

multiclass SystemZQuaternaryIntBHFG<string name> :
  SystemZQuaternaryIntBHF<name> {
  def g : SystemZQuaternaryInt<name#"g", llvm_v2i64_ty>;
}
````
- **L161 EN**: Declares TableGen def `h`.
  **L161 CN**: 声明 TableGen def `h`。
- **L162 EN**: Declares TableGen def `f`.
  **L162 CN**: 声明 TableGen def `f`。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L165 EN**: Declares TableGen multiclass `SystemZTernaryIntCCBHF`.
  **L165 CN**: 声明 TableGen multiclass `SystemZTernaryIntCCBHF`。
- **L166 EN**: Declares TableGen def `bs`.
  **L166 CN**: 声明 TableGen def `bs`。
- **L167 EN**: Declares TableGen def `hs`.
  **L167 CN**: 声明 TableGen def `hs`。
- **L168 EN**: Declares TableGen def `fs`.
  **L168 CN**: 声明 TableGen def `fs`。
- **L169 EN**: Closes the current lexical scope or compound statement.
  **L169 CN**: 结束当前词法作用域或复合语句块。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Declares TableGen multiclass `SystemZQuaternaryIntBHF`.
  **L171 CN**: 声明 TableGen multiclass `SystemZQuaternaryIntBHF`。
- **L172 EN**: Declares TableGen def `b`.
  **L172 CN**: 声明 TableGen def `b`。
- **L173 EN**: Declares TableGen def `h`.
  **L173 CN**: 声明 TableGen def `h`。
- **L174 EN**: Declares TableGen def `f`.
  **L174 CN**: 声明 TableGen def `f`。
- **L175 EN**: Closes the current lexical scope or compound statement.
  **L175 CN**: 结束当前词法作用域或复合语句块。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L177 EN**: Declares TableGen multiclass `SystemZQuaternaryIntBHFG`.
  **L177 CN**: 声明 TableGen multiclass `SystemZQuaternaryIntBHFG`。
- **L178 EN**: Continues the surrounding expression or declaration: `SystemZQuaternaryIntBHF<name> {`.
  **L178 CN**: 继续构造周围的表达式或声明：`SystemZQuaternaryIntBHF<name> {`。
- **L179 EN**: Declares TableGen def `g`.
  **L179 CN**: 声明 TableGen def `g`。
- **L180 EN**: Closes the current lexical scope or compound statement.
  **L180 CN**: 结束当前词法作用域或复合语句块。

### Lines 181-200

````tablegen

multiclass SystemZQuaternaryIntCCBHF {
  def bs : SystemZQuaternaryIntCC<llvm_v16i8_ty>;
  def hs : SystemZQuaternaryIntCC<llvm_v8i16_ty>;
  def fs : SystemZQuaternaryIntCC<llvm_v4i32_ty>;
}

//===----------------------------------------------------------------------===//
//
// Transactional-execution intrinsics
//
//===----------------------------------------------------------------------===//

let TargetPrefix = "s390" in {
  def int_s390_tbegin : Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],
                                  [IntrNoDuplicate, IntrWriteMem]>;

  def int_s390_tbegin_nofloat : Intrinsic<[llvm_i32_ty],
                                          [llvm_ptr_ty, llvm_i32_ty],
                                          [IntrNoDuplicate, IntrWriteMem]>;
````
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L182 EN**: Declares TableGen multiclass `SystemZQuaternaryIntCCBHF`.
  **L182 CN**: 声明 TableGen multiclass `SystemZQuaternaryIntCCBHF`。
- **L183 EN**: Declares TableGen def `bs`.
  **L183 CN**: 声明 TableGen def `bs`。
- **L184 EN**: Declares TableGen def `hs`.
  **L184 CN**: 声明 TableGen def `hs`。
- **L185 EN**: Declares TableGen def `fs`.
  **L185 CN**: 声明 TableGen def `fs`。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L188 EN**: Banner comment marking a file or section boundary.
  **L188 CN**: 横幅注释，用于标记文件或章节边界。
- **L189 EN**: Separator comment used for visual grouping.
  **L189 CN**: 用于视觉分组的分隔注释。
- **L190 EN**: Comment explains nearby logic, invariants, or intent: `Transactional-execution intrinsics`.
  **L190 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Transactional-execution intrinsics`。
- **L191 EN**: Separator comment used for visual grouping.
  **L191 CN**: 用于视觉分组的分隔注释。
- **L192 EN**: Banner comment marking a file or section boundary.
  **L192 CN**: 横幅注释，用于标记文件或章节边界。
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L194 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L194 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L195 EN**: Declares TableGen def `int_s390_tbegin`.
  **L195 CN**: 声明 TableGen def `int_s390_tbegin`。
- **L196 EN**: Executes a standalone statement or declaration: `[IntrNoDuplicate, IntrWriteMem]>;`.
  **L196 CN**: 执行一条独立语句或声明：`[IntrNoDuplicate, IntrWriteMem]>;`。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L198 EN**: Declares TableGen def `int_s390_tbegin_nofloat`.
  **L198 CN**: 声明 TableGen def `int_s390_tbegin_nofloat`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_ptr_ty, llvm_i32_ty],`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_ptr_ty, llvm_i32_ty],`。
- **L200 EN**: Executes a standalone statement or declaration: `[IntrNoDuplicate, IntrWriteMem]>;`.
  **L200 CN**: 执行一条独立语句或声明：`[IntrNoDuplicate, IntrWriteMem]>;`。

### Lines 201-220

````tablegen

  def int_s390_tbeginc : Intrinsic<[], [llvm_ptr_ty, llvm_i32_ty],
                                   [IntrNoDuplicate, IntrWriteMem]>;

  def int_s390_tabort : Intrinsic<[], [llvm_i64_ty],
                                  [IntrNoReturn, Throws, IntrWriteMem]>;

  def int_s390_tend : ClangBuiltin<"__builtin_tend">,
                      Intrinsic<[llvm_i32_ty], []>;

  def int_s390_etnd : ClangBuiltin<"__builtin_tx_nesting_depth">,
                      Intrinsic<[llvm_i32_ty], [], [IntrNoMem]>;

  def int_s390_ntstg : Intrinsic<[], [llvm_i64_ty, llvm_ptr_ty],
                                 [IntrArgMemOnly, IntrWriteMem]>;

  def int_s390_ppa_txassist : ClangBuiltin<"__builtin_tx_assist">,
                              Intrinsic<[], [llvm_i32_ty]>;
}

````
- **L201 EN**: Blank line separating nearby declarations or logic blocks.
  **L201 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L202 EN**: Declares TableGen def `int_s390_tbeginc`.
  **L202 CN**: 声明 TableGen def `int_s390_tbeginc`。
- **L203 EN**: Executes a standalone statement or declaration: `[IntrNoDuplicate, IntrWriteMem]>;`.
  **L203 CN**: 执行一条独立语句或声明：`[IntrNoDuplicate, IntrWriteMem]>;`。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L205 EN**: Declares TableGen def `int_s390_tabort`.
  **L205 CN**: 声明 TableGen def `int_s390_tabort`。
- **L206 EN**: Executes a standalone statement or declaration: `[IntrNoReturn, Throws, IntrWriteMem]>;`.
  **L206 CN**: 执行一条独立语句或声明：`[IntrNoReturn, Throws, IntrWriteMem]>;`。
- **L207 EN**: Blank line separating nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L208 EN**: Declares TableGen def `int_s390_tend`.
  **L208 CN**: 声明 TableGen def `int_s390_tend`。
- **L209 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], []>;`.
  **L209 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], []>;`。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Declares TableGen def `int_s390_etnd`.
  **L211 CN**: 声明 TableGen def `int_s390_etnd`。
- **L212 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], [IntrNoMem]>;`.
  **L212 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], [IntrNoMem]>;`。
- **L213 EN**: Blank line separating nearby declarations or logic blocks.
  **L213 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L214 EN**: Declares TableGen def `int_s390_ntstg`.
  **L214 CN**: 声明 TableGen def `int_s390_ntstg`。
- **L215 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, IntrWriteMem]>;`.
  **L215 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, IntrWriteMem]>;`。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L217 EN**: Declares TableGen def `int_s390_ppa_txassist`.
  **L217 CN**: 声明 TableGen def `int_s390_ppa_txassist`。
- **L218 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty]>;`.
  **L218 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty]>;`。
- **L219 EN**: Closes the current lexical scope or compound statement.
  **L219 CN**: 结束当前词法作用域或复合语句块。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 221-240

````tablegen
//===----------------------------------------------------------------------===//
//
// Vector intrinsics
//
//===----------------------------------------------------------------------===//

let TargetPrefix = "s390" in {
  def int_s390_lcbb : ClangBuiltin<"__builtin_s390_lcbb">,
                      Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],
                                [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  def int_s390_vlbb : ClangBuiltin<"__builtin_s390_vlbb">,
                      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],
                                [IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;

  def int_s390_vll : ClangBuiltin<"__builtin_s390_vll">,
                     DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i32_ty, llvm_ptr_ty],
                               [IntrReadMem, IntrArgMemOnly]>;

  def int_s390_vpdi : ClangBuiltin<"__builtin_s390_vpdi">,
````
- **L221 EN**: Banner comment marking a file or section boundary.
  **L221 CN**: 横幅注释，用于标记文件或章节边界。
- **L222 EN**: Separator comment used for visual grouping.
  **L222 CN**: 用于视觉分组的分隔注释。
- **L223 EN**: Comment explains nearby logic, invariants, or intent: `Vector intrinsics`.
  **L223 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Vector intrinsics`。
- **L224 EN**: Separator comment used for visual grouping.
  **L224 CN**: 用于视觉分组的分隔注释。
- **L225 EN**: Banner comment marking a file or section boundary.
  **L225 CN**: 横幅注释，用于标记文件或章节边界。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L227 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L227 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L228 EN**: Declares TableGen def `int_s390_lcbb`.
  **L228 CN**: 声明 TableGen def `int_s390_lcbb`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L230 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L230 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L232 EN**: Declares TableGen def `int_s390_vlbb`.
  **L232 CN**: 声明 TableGen def `int_s390_vlbb`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_ptr_ty, llvm_i32_ty],`。
- **L234 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`.
  **L234 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly, ImmArg<ArgIndex<1>>]>;`。
- **L235 EN**: Blank line separating nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L236 EN**: Declares TableGen def `int_s390_vll`.
  **L236 CN**: 声明 TableGen def `int_s390_vll`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i32_ty, llvm_ptr_ty],`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i32_ty, llvm_ptr_ty],`。
- **L238 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L238 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L240 EN**: Declares TableGen def `int_s390_vpdi`.
  **L240 CN**: 声明 TableGen def `int_s390_vpdi`。

### Lines 241-260

````tablegen
                      Intrinsic<[llvm_v2i64_ty],
                                [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],
                                [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  def int_s390_vperm : ClangBuiltin<"__builtin_s390_vperm">,
                       Intrinsic<[llvm_v16i8_ty],
                                 [llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],
                                 [IntrNoMem]>;

  defm int_s390_vpks : SystemZBinaryTruncHFG<"vpks">;
  defm int_s390_vpks : SystemZBinaryTruncCCHFG;

  defm int_s390_vpkls : SystemZBinaryTruncHFG<"vpkls">;
  defm int_s390_vpkls : SystemZBinaryTruncCCHFG;

  def int_s390_vstl : ClangBuiltin<"__builtin_s390_vstl">,
                      Intrinsic<[], [llvm_v16i8_ty, llvm_i32_ty, llvm_ptr_ty],
                                [IntrArgMemOnly, IntrWriteMem]>;

  defm int_s390_vupl  : SystemZUnaryExtBHWF<"vupl">;
````
- **L241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v2i64_ty],`.
  **L241 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v2i64_ty],`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i32_ty],`。
- **L243 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L243 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L244 EN**: Blank line separating nearby declarations or logic blocks.
  **L244 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L245 EN**: Declares TableGen def `int_s390_vperm`.
  **L245 CN**: 声明 TableGen def `int_s390_vperm`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty],`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty],`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_v16i8_ty],`。
- **L248 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L248 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Declares TableGen defm `int_s390_vpks`.
  **L250 CN**: 声明 TableGen defm `int_s390_vpks`。
- **L251 EN**: Declares TableGen defm `int_s390_vpks`.
  **L251 CN**: 声明 TableGen defm `int_s390_vpks`。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L253 EN**: Declares TableGen defm `int_s390_vpkls`.
  **L253 CN**: 声明 TableGen defm `int_s390_vpkls`。
- **L254 EN**: Declares TableGen defm `int_s390_vpkls`.
  **L254 CN**: 声明 TableGen defm `int_s390_vpkls`。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Declares TableGen def `int_s390_vstl`.
  **L256 CN**: 声明 TableGen def `int_s390_vstl`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v16i8_ty, llvm_i32_ty, llvm_ptr_ty],`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v16i8_ty, llvm_i32_ty, llvm_ptr_ty],`。
- **L258 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, IntrWriteMem]>;`.
  **L258 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, IntrWriteMem]>;`。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L260 EN**: Declares TableGen defm `int_s390_vupl`.
  **L260 CN**: 声明 TableGen defm `int_s390_vupl`。

### Lines 261-280

````tablegen
  defm int_s390_vupll : SystemZUnaryExtBHF<"vupll">;

  defm int_s390_vuph  : SystemZUnaryExtBHF<"vuph">;
  defm int_s390_vuplh : SystemZUnaryExtBHF<"vuplh">;

  defm int_s390_vacc : SystemZBinaryBHFG<"vacc">;

  def int_s390_vaq    : SystemZBinary<"vaq",     llvm_i128_ty>;
  def int_s390_vacq   : SystemZTernary<"vacq",   llvm_i128_ty>;
  def int_s390_vaccq  : SystemZBinary<"vaccq",   llvm_i128_ty>;
  def int_s390_vacccq : SystemZTernary<"vacccq", llvm_i128_ty>;

  defm int_s390_vavg  : SystemZBinaryBHFG<"vavg">;
  defm int_s390_vavgl : SystemZBinaryBHFG<"vavgl">;

  def int_s390_vcksm : SystemZBinary<"vcksm", llvm_v4i32_ty>;

  defm int_s390_vgfm  : SystemZBinaryExtBHFG<"vgfm">;
  defm int_s390_vgfma : SystemZTernaryExtBHFG<"vgfma">;

````
- **L261 EN**: Declares TableGen defm `int_s390_vupll`.
  **L261 CN**: 声明 TableGen defm `int_s390_vupll`。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L263 EN**: Declares TableGen defm `int_s390_vuph`.
  **L263 CN**: 声明 TableGen defm `int_s390_vuph`。
- **L264 EN**: Declares TableGen defm `int_s390_vuplh`.
  **L264 CN**: 声明 TableGen defm `int_s390_vuplh`。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Declares TableGen defm `int_s390_vacc`.
  **L266 CN**: 声明 TableGen defm `int_s390_vacc`。
- **L267 EN**: Blank line separating nearby declarations or logic blocks.
  **L267 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L268 EN**: Declares TableGen def `int_s390_vaq`.
  **L268 CN**: 声明 TableGen def `int_s390_vaq`。
- **L269 EN**: Declares TableGen def `int_s390_vacq`.
  **L269 CN**: 声明 TableGen def `int_s390_vacq`。
- **L270 EN**: Declares TableGen def `int_s390_vaccq`.
  **L270 CN**: 声明 TableGen def `int_s390_vaccq`。
- **L271 EN**: Declares TableGen def `int_s390_vacccq`.
  **L271 CN**: 声明 TableGen def `int_s390_vacccq`。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L273 EN**: Declares TableGen defm `int_s390_vavg`.
  **L273 CN**: 声明 TableGen defm `int_s390_vavg`。
- **L274 EN**: Declares TableGen defm `int_s390_vavgl`.
  **L274 CN**: 声明 TableGen defm `int_s390_vavgl`。
- **L275 EN**: Blank line separating nearby declarations or logic blocks.
  **L275 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L276 EN**: Declares TableGen def `int_s390_vcksm`.
  **L276 CN**: 声明 TableGen def `int_s390_vcksm`。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Declares TableGen defm `int_s390_vgfm`.
  **L278 CN**: 声明 TableGen defm `int_s390_vgfm`。
- **L279 EN**: Declares TableGen defm `int_s390_vgfma`.
  **L279 CN**: 声明 TableGen defm `int_s390_vgfma`。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 281-300

````tablegen
  defm int_s390_vmah  : SystemZTernaryBHF<"vmah">;
  defm int_s390_vmalh : SystemZTernaryBHF<"vmalh">;
  defm int_s390_vmae  : SystemZTernaryExtBHF<"vmae">;
  defm int_s390_vmale : SystemZTernaryExtBHF<"vmale">;
  defm int_s390_vmao  : SystemZTernaryExtBHF<"vmao">;
  defm int_s390_vmalo : SystemZTernaryExtBHF<"vmalo">;

  defm int_s390_vmh  : SystemZBinaryBHF<"vmh">;
  defm int_s390_vmlh : SystemZBinaryBHF<"vmlh">;
  defm int_s390_vme  : SystemZBinaryExtBHF<"vme">;
  defm int_s390_vmle : SystemZBinaryExtBHF<"vmle">;
  defm int_s390_vmo  : SystemZBinaryExtBHF<"vmo">;
  defm int_s390_vmlo : SystemZBinaryExtBHF<"vmlo">;

  defm int_s390_verim  : SystemZQuaternaryIntBHFG<"verim">;

  def int_s390_vsl   : SystemZBinary<"vsl",   llvm_v16i8_ty>;
  def int_s390_vslb  : SystemZBinary<"vslb",  llvm_v16i8_ty>;
  def int_s390_vsra  : SystemZBinary<"vsra",  llvm_v16i8_ty>;
  def int_s390_vsrab : SystemZBinary<"vsrab", llvm_v16i8_ty>;
````
- **L281 EN**: Declares TableGen defm `int_s390_vmah`.
  **L281 CN**: 声明 TableGen defm `int_s390_vmah`。
- **L282 EN**: Declares TableGen defm `int_s390_vmalh`.
  **L282 CN**: 声明 TableGen defm `int_s390_vmalh`。
- **L283 EN**: Declares TableGen defm `int_s390_vmae`.
  **L283 CN**: 声明 TableGen defm `int_s390_vmae`。
- **L284 EN**: Declares TableGen defm `int_s390_vmale`.
  **L284 CN**: 声明 TableGen defm `int_s390_vmale`。
- **L285 EN**: Declares TableGen defm `int_s390_vmao`.
  **L285 CN**: 声明 TableGen defm `int_s390_vmao`。
- **L286 EN**: Declares TableGen defm `int_s390_vmalo`.
  **L286 CN**: 声明 TableGen defm `int_s390_vmalo`。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L288 EN**: Declares TableGen defm `int_s390_vmh`.
  **L288 CN**: 声明 TableGen defm `int_s390_vmh`。
- **L289 EN**: Declares TableGen defm `int_s390_vmlh`.
  **L289 CN**: 声明 TableGen defm `int_s390_vmlh`。
- **L290 EN**: Declares TableGen defm `int_s390_vme`.
  **L290 CN**: 声明 TableGen defm `int_s390_vme`。
- **L291 EN**: Declares TableGen defm `int_s390_vmle`.
  **L291 CN**: 声明 TableGen defm `int_s390_vmle`。
- **L292 EN**: Declares TableGen defm `int_s390_vmo`.
  **L292 CN**: 声明 TableGen defm `int_s390_vmo`。
- **L293 EN**: Declares TableGen defm `int_s390_vmlo`.
  **L293 CN**: 声明 TableGen defm `int_s390_vmlo`。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L295 EN**: Declares TableGen defm `int_s390_verim`.
  **L295 CN**: 声明 TableGen defm `int_s390_verim`。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L297 EN**: Declares TableGen def `int_s390_vsl`.
  **L297 CN**: 声明 TableGen def `int_s390_vsl`。
- **L298 EN**: Declares TableGen def `int_s390_vslb`.
  **L298 CN**: 声明 TableGen def `int_s390_vslb`。
- **L299 EN**: Declares TableGen def `int_s390_vsra`.
  **L299 CN**: 声明 TableGen def `int_s390_vsra`。
- **L300 EN**: Declares TableGen def `int_s390_vsrab`.
  **L300 CN**: 声明 TableGen def `int_s390_vsrab`。

### Lines 301-320

````tablegen
  def int_s390_vsrl  : SystemZBinary<"vsrl",  llvm_v16i8_ty>;
  def int_s390_vsrlb : SystemZBinary<"vsrlb", llvm_v16i8_ty>;

  def int_s390_vsldb : ClangBuiltin<"__builtin_s390_vsldb">,
                       Intrinsic<[llvm_v16i8_ty],
                                 [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                                 [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  defm int_s390_vscbi : SystemZBinaryBHFG<"vscbi">;

  def int_s390_vsq     : SystemZBinary<"vsq",      llvm_i128_ty>;
  def int_s390_vsbiq   : SystemZTernary<"vsbiq",   llvm_i128_ty>;
  def int_s390_vscbiq  : SystemZBinary<"vscbiq",   llvm_i128_ty>;
  def int_s390_vsbcbiq : SystemZTernary<"vsbcbiq", llvm_i128_ty>;

  def int_s390_vsumb : SystemZBinaryConv<"vsumb", llvm_v4i32_ty, llvm_v16i8_ty>;
  def int_s390_vsumh : SystemZBinaryConv<"vsumh", llvm_v4i32_ty, llvm_v8i16_ty>;

  def int_s390_vsumgh : SystemZBinaryConv<"vsumgh", llvm_v2i64_ty,
                                          llvm_v8i16_ty>;
````
- **L301 EN**: Declares TableGen def `int_s390_vsrl`.
  **L301 CN**: 声明 TableGen def `int_s390_vsrl`。
- **L302 EN**: Declares TableGen def `int_s390_vsrlb`.
  **L302 CN**: 声明 TableGen def `int_s390_vsrlb`。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L304 EN**: Declares TableGen def `int_s390_vsldb`.
  **L304 CN**: 声明 TableGen def `int_s390_vsldb`。
- **L305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty],`.
  **L305 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty],`。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L307 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L307 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L309 EN**: Declares TableGen defm `int_s390_vscbi`.
  **L309 CN**: 声明 TableGen defm `int_s390_vscbi`。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L311 EN**: Declares TableGen def `int_s390_vsq`.
  **L311 CN**: 声明 TableGen def `int_s390_vsq`。
- **L312 EN**: Declares TableGen def `int_s390_vsbiq`.
  **L312 CN**: 声明 TableGen def `int_s390_vsbiq`。
- **L313 EN**: Declares TableGen def `int_s390_vscbiq`.
  **L313 CN**: 声明 TableGen def `int_s390_vscbiq`。
- **L314 EN**: Declares TableGen def `int_s390_vsbcbiq`.
  **L314 CN**: 声明 TableGen def `int_s390_vsbcbiq`。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L316 EN**: Declares TableGen def `int_s390_vsumb`.
  **L316 CN**: 声明 TableGen def `int_s390_vsumb`。
- **L317 EN**: Declares TableGen def `int_s390_vsumh`.
  **L317 CN**: 声明 TableGen def `int_s390_vsumh`。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L319 EN**: Declares TableGen def `int_s390_vsumgh`.
  **L319 CN**: 声明 TableGen def `int_s390_vsumgh`。
- **L320 EN**: Executes a standalone statement or declaration: `llvm_v8i16_ty>;`.
  **L320 CN**: 执行一条独立语句或声明：`llvm_v8i16_ty>;`。

### Lines 321-340

````tablegen
  def int_s390_vsumgf : SystemZBinaryConv<"vsumgf", llvm_v2i64_ty,
                                          llvm_v4i32_ty>;

  def int_s390_vsumqf : SystemZBinaryConv<"vsumqf", llvm_i128_ty,
                                          llvm_v4i32_ty>;
  def int_s390_vsumqg : SystemZBinaryConv<"vsumqg", llvm_i128_ty,
                                          llvm_v2i64_ty>;

  def int_s390_vtm : SystemZBinaryConv<"vtm", llvm_i32_ty, llvm_v16i8_ty>;

  defm int_s390_vceq : SystemZCompareBHFG;
  defm int_s390_vch  : SystemZCompareBHFG;
  defm int_s390_vchl : SystemZCompareBHFG;

  defm int_s390_vfae  : SystemZTernaryIntBHF<"vfae">;
  defm int_s390_vfae  : SystemZTernaryIntCCBHF;
  defm int_s390_vfaez : SystemZTernaryIntBHF<"vfaez">;
  defm int_s390_vfaez : SystemZTernaryIntCCBHF;

  defm int_s390_vfee  : SystemZBinaryBHF<"vfee">;
````
- **L321 EN**: Declares TableGen def `int_s390_vsumgf`.
  **L321 CN**: 声明 TableGen def `int_s390_vsumgf`。
- **L322 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty>;`.
  **L322 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty>;`。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L324 EN**: Declares TableGen def `int_s390_vsumqf`.
  **L324 CN**: 声明 TableGen def `int_s390_vsumqf`。
- **L325 EN**: Executes a standalone statement or declaration: `llvm_v4i32_ty>;`.
  **L325 CN**: 执行一条独立语句或声明：`llvm_v4i32_ty>;`。
- **L326 EN**: Declares TableGen def `int_s390_vsumqg`.
  **L326 CN**: 声明 TableGen def `int_s390_vsumqg`。
- **L327 EN**: Executes a standalone statement or declaration: `llvm_v2i64_ty>;`.
  **L327 CN**: 执行一条独立语句或声明：`llvm_v2i64_ty>;`。
- **L328 EN**: Blank line separating nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L329 EN**: Declares TableGen def `int_s390_vtm`.
  **L329 CN**: 声明 TableGen def `int_s390_vtm`。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L331 EN**: Declares TableGen defm `int_s390_vceq`.
  **L331 CN**: 声明 TableGen defm `int_s390_vceq`。
- **L332 EN**: Declares TableGen defm `int_s390_vch`.
  **L332 CN**: 声明 TableGen defm `int_s390_vch`。
- **L333 EN**: Declares TableGen defm `int_s390_vchl`.
  **L333 CN**: 声明 TableGen defm `int_s390_vchl`。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L335 EN**: Declares TableGen defm `int_s390_vfae`.
  **L335 CN**: 声明 TableGen defm `int_s390_vfae`。
- **L336 EN**: Declares TableGen defm `int_s390_vfae`.
  **L336 CN**: 声明 TableGen defm `int_s390_vfae`。
- **L337 EN**: Declares TableGen defm `int_s390_vfaez`.
  **L337 CN**: 声明 TableGen defm `int_s390_vfaez`。
- **L338 EN**: Declares TableGen defm `int_s390_vfaez`.
  **L338 CN**: 声明 TableGen defm `int_s390_vfaez`。
- **L339 EN**: Blank line separating nearby declarations or logic blocks.
  **L339 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L340 EN**: Declares TableGen defm `int_s390_vfee`.
  **L340 CN**: 声明 TableGen defm `int_s390_vfee`。

### Lines 341-360

````tablegen
  defm int_s390_vfee  : SystemZBinaryCCBHF;
  defm int_s390_vfeez : SystemZBinaryBHF<"vfeez">;
  defm int_s390_vfeez : SystemZBinaryCCBHF;

  defm int_s390_vfene  : SystemZBinaryBHF<"vfene">;
  defm int_s390_vfene  : SystemZBinaryCCBHF;
  defm int_s390_vfenez : SystemZBinaryBHF<"vfenez">;
  defm int_s390_vfenez : SystemZBinaryCCBHF;

  defm int_s390_vistr : SystemZUnaryBHF<"vistr">;
  defm int_s390_vistr : SystemZUnaryCCBHF;

  defm int_s390_vstrc  : SystemZQuaternaryIntBHF<"vstrc">;
  defm int_s390_vstrc  : SystemZQuaternaryIntCCBHF;
  defm int_s390_vstrcz : SystemZQuaternaryIntBHF<"vstrcz">;
  defm int_s390_vstrcz : SystemZQuaternaryIntCCBHF;

  def int_s390_vfcedbs  : SystemZBinaryConvCC<llvm_v2i64_ty, llvm_v2f64_ty>;
  def int_s390_vfchdbs  : SystemZBinaryConvCC<llvm_v2i64_ty, llvm_v2f64_ty>;
  def int_s390_vfchedbs : SystemZBinaryConvCC<llvm_v2i64_ty, llvm_v2f64_ty>;
````
- **L341 EN**: Declares TableGen defm `int_s390_vfee`.
  **L341 CN**: 声明 TableGen defm `int_s390_vfee`。
- **L342 EN**: Declares TableGen defm `int_s390_vfeez`.
  **L342 CN**: 声明 TableGen defm `int_s390_vfeez`。
- **L343 EN**: Declares TableGen defm `int_s390_vfeez`.
  **L343 CN**: 声明 TableGen defm `int_s390_vfeez`。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L345 EN**: Declares TableGen defm `int_s390_vfene`.
  **L345 CN**: 声明 TableGen defm `int_s390_vfene`。
- **L346 EN**: Declares TableGen defm `int_s390_vfene`.
  **L346 CN**: 声明 TableGen defm `int_s390_vfene`。
- **L347 EN**: Declares TableGen defm `int_s390_vfenez`.
  **L347 CN**: 声明 TableGen defm `int_s390_vfenez`。
- **L348 EN**: Declares TableGen defm `int_s390_vfenez`.
  **L348 CN**: 声明 TableGen defm `int_s390_vfenez`。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Declares TableGen defm `int_s390_vistr`.
  **L350 CN**: 声明 TableGen defm `int_s390_vistr`。
- **L351 EN**: Declares TableGen defm `int_s390_vistr`.
  **L351 CN**: 声明 TableGen defm `int_s390_vistr`。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L353 EN**: Declares TableGen defm `int_s390_vstrc`.
  **L353 CN**: 声明 TableGen defm `int_s390_vstrc`。
- **L354 EN**: Declares TableGen defm `int_s390_vstrc`.
  **L354 CN**: 声明 TableGen defm `int_s390_vstrc`。
- **L355 EN**: Declares TableGen defm `int_s390_vstrcz`.
  **L355 CN**: 声明 TableGen defm `int_s390_vstrcz`。
- **L356 EN**: Declares TableGen defm `int_s390_vstrcz`.
  **L356 CN**: 声明 TableGen defm `int_s390_vstrcz`。
- **L357 EN**: Blank line separating nearby declarations or logic blocks.
  **L357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L358 EN**: Declares TableGen def `int_s390_vfcedbs`.
  **L358 CN**: 声明 TableGen def `int_s390_vfcedbs`。
- **L359 EN**: Declares TableGen def `int_s390_vfchdbs`.
  **L359 CN**: 声明 TableGen def `int_s390_vfchdbs`。
- **L360 EN**: Declares TableGen def `int_s390_vfchedbs`.
  **L360 CN**: 声明 TableGen def `int_s390_vfchedbs`。

### Lines 361-380

````tablegen

  def int_s390_vftcidb : SystemZBinaryConvIntCC<llvm_v2i64_ty, llvm_v2f64_ty>;

  def int_s390_vfidb : Intrinsic<[llvm_v2f64_ty],
                                 [llvm_v2f64_ty, llvm_i32_ty, llvm_i32_ty],
                                 [IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;

  // Instructions from the Vector Enhancements Facility 1
  def int_s390_vbperm : SystemZBinaryConv<"vbperm", llvm_v2i64_ty,
                                          llvm_v16i8_ty>;

  def int_s390_vmslg  : ClangBuiltin<"__builtin_s390_vmslg">,
                        Intrinsic<[llvm_i128_ty],
                                  [llvm_v2i64_ty, llvm_v2i64_ty, llvm_i128_ty,
                                   llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<3>>]>;

  def int_s390_vfmaxdb : Intrinsic<[llvm_v2f64_ty],
                                   [llvm_v2f64_ty, llvm_v2f64_ty, llvm_i32_ty],
                                   [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_s390_vfmindb : Intrinsic<[llvm_v2f64_ty],
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L362 EN**: Declares TableGen def `int_s390_vftcidb`.
  **L362 CN**: 声明 TableGen def `int_s390_vftcidb`。
- **L363 EN**: Blank line separating nearby declarations or logic blocks.
  **L363 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L364 EN**: Declares TableGen def `int_s390_vfidb`.
  **L364 CN**: 声明 TableGen def `int_s390_vfidb`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L366 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`.
  **L366 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L368 EN**: Comment explains nearby logic, invariants, or intent: `Instructions from the Vector Enhancements Facility 1`.
  **L368 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Instructions from the Vector Enhancements Facility 1`。
- **L369 EN**: Declares TableGen def `int_s390_vbperm`.
  **L369 CN**: 声明 TableGen def `int_s390_vbperm`。
- **L370 EN**: Executes a standalone statement or declaration: `llvm_v16i8_ty>;`.
  **L370 CN**: 执行一条独立语句或声明：`llvm_v16i8_ty>;`。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L372 EN**: Declares TableGen def `int_s390_vmslg`.
  **L372 CN**: 声明 TableGen def `int_s390_vmslg`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i128_ty],`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i128_ty],`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i128_ty,`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2i64_ty, llvm_v2i64_ty, llvm_i128_ty,`。
- **L375 EN**: Executes a standalone statement or declaration: `llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<3>>]>;`.
  **L375 CN**: 执行一条独立语句或声明：`llvm_i32_ty], [IntrNoMem, ImmArg<ArgIndex<3>>]>;`。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L377 EN**: Declares TableGen def `int_s390_vfmaxdb`.
  **L377 CN**: 声明 TableGen def `int_s390_vfmaxdb`。
- **L378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_v2f64_ty, llvm_i32_ty],`.
  **L378 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_v2f64_ty, llvm_i32_ty],`。
- **L379 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L379 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L380 EN**: Declares TableGen def `int_s390_vfmindb`.
  **L380 CN**: 声明 TableGen def `int_s390_vfmindb`。

### Lines 381-400

````tablegen
                                   [llvm_v2f64_ty, llvm_v2f64_ty, llvm_i32_ty],
                                   [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_s390_vfmaxsb : Intrinsic<[llvm_v4f32_ty],
                                   [llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],
                                   [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_s390_vfminsb : Intrinsic<[llvm_v4f32_ty],
                                   [llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],
                                   [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  def int_s390_vfcesbs  : SystemZBinaryConvCC<llvm_v4i32_ty, llvm_v4f32_ty>;
  def int_s390_vfchsbs  : SystemZBinaryConvCC<llvm_v4i32_ty, llvm_v4f32_ty>;
  def int_s390_vfchesbs : SystemZBinaryConvCC<llvm_v4i32_ty, llvm_v4f32_ty>;

  def int_s390_vftcisb : SystemZBinaryConvIntCC<llvm_v4i32_ty, llvm_v4f32_ty>;

  def int_s390_vfisb : Intrinsic<[llvm_v4f32_ty],
                                 [llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty],
                                 [IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;

  // Instructions from the Vector Packed Decimal Facility
````
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v2f64_ty, llvm_v2f64_ty, llvm_i32_ty],`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v2f64_ty, llvm_v2f64_ty, llvm_i32_ty],`。
- **L382 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L382 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L383 EN**: Declares TableGen def `int_s390_vfmaxsb`.
  **L383 CN**: 声明 TableGen def `int_s390_vfmaxsb`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],`。
- **L385 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L385 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L386 EN**: Declares TableGen def `int_s390_vfminsb`.
  **L386 CN**: 声明 TableGen def `int_s390_vfminsb`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],`。
- **L388 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L388 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L390 EN**: Declares TableGen def `int_s390_vfcesbs`.
  **L390 CN**: 声明 TableGen def `int_s390_vfcesbs`。
- **L391 EN**: Declares TableGen def `int_s390_vfchsbs`.
  **L391 CN**: 声明 TableGen def `int_s390_vfchsbs`。
- **L392 EN**: Declares TableGen def `int_s390_vfchesbs`.
  **L392 CN**: 声明 TableGen def `int_s390_vfchesbs`。
- **L393 EN**: Blank line separating nearby declarations or logic blocks.
  **L393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L394 EN**: Declares TableGen def `int_s390_vftcisb`.
  **L394 CN**: 声明 TableGen def `int_s390_vftcisb`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L396 EN**: Declares TableGen def `int_s390_vfisb`.
  **L396 CN**: 声明 TableGen def `int_s390_vfisb`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty],`.
  **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_i32_ty, llvm_i32_ty],`。
- **L398 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`.
  **L398 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>]>;`。
- **L399 EN**: Blank line separating nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L400 EN**: Comment explains nearby logic, invariants, or intent: `Instructions from the Vector Packed Decimal Facility`.
  **L400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Instructions from the Vector Packed Decimal Facility`。

### Lines 401-420

````tablegen
  def int_s390_vlrl : ClangBuiltin<"__builtin_s390_vlrlr">,
                      DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i32_ty, llvm_ptr_ty],
                                [IntrReadMem, IntrArgMemOnly]>;

  def int_s390_vstrl : ClangBuiltin<"__builtin_s390_vstrlr">,
                       Intrinsic<[], [llvm_v16i8_ty, llvm_i32_ty, llvm_ptr_ty],
                                 [IntrArgMemOnly, IntrWriteMem]>;

  // Instructions from the Vector Enhancements Facility 2
  def int_s390_vsld : ClangBuiltin<"__builtin_s390_vsld">,
                      Intrinsic<[llvm_v16i8_ty],
                                [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                                [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  def int_s390_vsrd : ClangBuiltin<"__builtin_s390_vsrd">,
                      Intrinsic<[llvm_v16i8_ty],
                                [llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],
                                [IntrNoMem, ImmArg<ArgIndex<2>>]>;

  def int_s390_vstrsb : SystemZTernaryConvCC<llvm_v16i8_ty, llvm_v16i8_ty>;
````
- **L401 EN**: Declares TableGen def `int_s390_vlrl`.
  **L401 CN**: 声明 TableGen def `int_s390_vlrl`。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i32_ty, llvm_ptr_ty],`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_v16i8_ty], [llvm_i32_ty, llvm_ptr_ty],`。
- **L403 EN**: Executes a standalone statement or declaration: `[IntrReadMem, IntrArgMemOnly]>;`.
  **L403 CN**: 执行一条独立语句或声明：`[IntrReadMem, IntrArgMemOnly]>;`。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L405 EN**: Declares TableGen def `int_s390_vstrl`.
  **L405 CN**: 声明 TableGen def `int_s390_vstrl`。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_v16i8_ty, llvm_i32_ty, llvm_ptr_ty],`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_v16i8_ty, llvm_i32_ty, llvm_ptr_ty],`。
- **L407 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, IntrWriteMem]>;`.
  **L407 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, IntrWriteMem]>;`。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L409 EN**: Comment explains nearby logic, invariants, or intent: `Instructions from the Vector Enhancements Facility 2`.
  **L409 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Instructions from the Vector Enhancements Facility 2`。
- **L410 EN**: Declares TableGen def `int_s390_vsld`.
  **L410 CN**: 声明 TableGen def `int_s390_vsld`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty],`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty],`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L413 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L413 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L415 EN**: Declares TableGen def `int_s390_vsrd`.
  **L415 CN**: 声明 TableGen def `int_s390_vsrd`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v16i8_ty],`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v16i8_ty],`。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v16i8_ty, llvm_v16i8_ty, llvm_i32_ty],`。
- **L418 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L418 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L420 EN**: Declares TableGen def `int_s390_vstrsb`.
  **L420 CN**: 声明 TableGen def `int_s390_vstrsb`。

### Lines 421-440

````tablegen
  def int_s390_vstrsh : SystemZTernaryConvCC<llvm_v16i8_ty, llvm_v8i16_ty>;
  def int_s390_vstrsf : SystemZTernaryConvCC<llvm_v16i8_ty, llvm_v4i32_ty>;
  def int_s390_vstrszb : SystemZTernaryConvCC<llvm_v16i8_ty, llvm_v16i8_ty>;
  def int_s390_vstrszh : SystemZTernaryConvCC<llvm_v16i8_ty, llvm_v8i16_ty>;
  def int_s390_vstrszf : SystemZTernaryConvCC<llvm_v16i8_ty, llvm_v4i32_ty>;

  // Instructions from the NNP-assist Facility
  def int_s390_vclfnhs : ClangBuiltin<"__builtin_s390_vclfnhs">,
                         Intrinsic<[llvm_v4f32_ty],
                                   [llvm_v8i16_ty, llvm_i32_ty],
                                   [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_s390_vclfnls : ClangBuiltin<"__builtin_s390_vclfnls">,
                         Intrinsic<[llvm_v4f32_ty],
                                   [llvm_v8i16_ty, llvm_i32_ty],
                                   [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_s390_vcrnfs : ClangBuiltin<"__builtin_s390_vcrnfs">,
                        Intrinsic<[llvm_v8i16_ty],
                                  [llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],
                                  [IntrNoMem, ImmArg<ArgIndex<2>>]>;
  def int_s390_vcfn : ClangBuiltin<"__builtin_s390_vcfn">,
````
- **L421 EN**: Declares TableGen def `int_s390_vstrsh`.
  **L421 CN**: 声明 TableGen def `int_s390_vstrsh`。
- **L422 EN**: Declares TableGen def `int_s390_vstrsf`.
  **L422 CN**: 声明 TableGen def `int_s390_vstrsf`。
- **L423 EN**: Declares TableGen def `int_s390_vstrszb`.
  **L423 CN**: 声明 TableGen def `int_s390_vstrszb`。
- **L424 EN**: Declares TableGen def `int_s390_vstrszh`.
  **L424 CN**: 声明 TableGen def `int_s390_vstrszh`。
- **L425 EN**: Declares TableGen def `int_s390_vstrszf`.
  **L425 CN**: 声明 TableGen def `int_s390_vstrszf`。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L427 EN**: Comment explains nearby logic, invariants, or intent: `Instructions from the NNP-assist Facility`.
  **L427 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Instructions from the NNP-assist Facility`。
- **L428 EN**: Declares TableGen def `int_s390_vclfnhs`.
  **L428 CN**: 声明 TableGen def `int_s390_vclfnhs`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4f32_ty],`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4f32_ty],`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty],`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty],`。
- **L431 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L431 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L432 EN**: Declares TableGen def `int_s390_vclfnls`.
  **L432 CN**: 声明 TableGen def `int_s390_vclfnls`。
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v4f32_ty],`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v4f32_ty],`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty],`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty],`。
- **L435 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L435 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L436 EN**: Declares TableGen def `int_s390_vcrnfs`.
  **L436 CN**: 声明 TableGen def `int_s390_vcrnfs`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty],`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty],`。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v4f32_ty, llvm_v4f32_ty, llvm_i32_ty],`。
- **L439 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<2>>]>;`.
  **L439 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<2>>]>;`。
- **L440 EN**: Declares TableGen def `int_s390_vcfn`.
  **L440 CN**: 声明 TableGen def `int_s390_vcfn`。

### Lines 441-460

````tablegen
                      Intrinsic<[llvm_v8i16_ty],
                                [llvm_v8i16_ty, llvm_i32_ty],
                                [IntrNoMem, ImmArg<ArgIndex<1>>]>;
  def int_s390_vcnf : ClangBuiltin<"__builtin_s390_vcnf">,
                      Intrinsic<[llvm_v8i16_ty],
                                [llvm_v8i16_ty, llvm_i32_ty],
                                [IntrNoMem, ImmArg<ArgIndex<1>>]>;

  // Instructions from the Vector Enhancements Facility 3
  def int_s390_vgemb  : SystemZUnaryConv<"vgemb", llvm_v16i8_ty, llvm_v8i16_ty>;
  def int_s390_vgemh  : SystemZUnaryConv<"vgemh", llvm_v8i16_ty, llvm_v16i8_ty>;
  def int_s390_vgemf  : SystemZUnaryConv<"vgemf", llvm_v4i32_ty, llvm_v16i8_ty>;
  def int_s390_vgemg  : SystemZUnaryConv<"vgemg", llvm_v2i64_ty, llvm_v16i8_ty>;
  def int_s390_vgemq  : SystemZUnaryConv<"vgemq", llvm_i128_ty, llvm_v16i8_ty>;
  def int_s390_vuplg  : SystemZUnaryConv<"vuplg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vupllg : SystemZUnaryConv<"vupllg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vuphg  : SystemZUnaryConv<"vuphg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vuplhg : SystemZUnaryConv<"vuplhg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vavgq  : SystemZBinary<"vavgq", llvm_i128_ty>;
  def int_s390_vavglq : SystemZBinary<"vavglq", llvm_i128_ty>;
````
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty],`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty],`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty],`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty],`。
- **L443 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L443 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L444 EN**: Declares TableGen def `int_s390_vcnf`.
  **L444 CN**: 声明 TableGen def `int_s390_vcnf`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_v8i16_ty],`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_v8i16_ty],`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_v8i16_ty, llvm_i32_ty],`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_v8i16_ty, llvm_i32_ty],`。
- **L447 EN**: Executes a standalone statement or declaration: `[IntrNoMem, ImmArg<ArgIndex<1>>]>;`.
  **L447 CN**: 执行一条独立语句或声明：`[IntrNoMem, ImmArg<ArgIndex<1>>]>;`。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L449 EN**: Comment explains nearby logic, invariants, or intent: `Instructions from the Vector Enhancements Facility 3`.
  **L449 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Instructions from the Vector Enhancements Facility 3`。
- **L450 EN**: Declares TableGen def `int_s390_vgemb`.
  **L450 CN**: 声明 TableGen def `int_s390_vgemb`。
- **L451 EN**: Declares TableGen def `int_s390_vgemh`.
  **L451 CN**: 声明 TableGen def `int_s390_vgemh`。
- **L452 EN**: Declares TableGen def `int_s390_vgemf`.
  **L452 CN**: 声明 TableGen def `int_s390_vgemf`。
- **L453 EN**: Declares TableGen def `int_s390_vgemg`.
  **L453 CN**: 声明 TableGen def `int_s390_vgemg`。
- **L454 EN**: Declares TableGen def `int_s390_vgemq`.
  **L454 CN**: 声明 TableGen def `int_s390_vgemq`。
- **L455 EN**: Declares TableGen def `int_s390_vuplg`.
  **L455 CN**: 声明 TableGen def `int_s390_vuplg`。
- **L456 EN**: Declares TableGen def `int_s390_vupllg`.
  **L456 CN**: 声明 TableGen def `int_s390_vupllg`。
- **L457 EN**: Declares TableGen def `int_s390_vuphg`.
  **L457 CN**: 声明 TableGen def `int_s390_vuphg`。
- **L458 EN**: Declares TableGen def `int_s390_vuplhg`.
  **L458 CN**: 声明 TableGen def `int_s390_vuplhg`。
- **L459 EN**: Declares TableGen def `int_s390_vavgq`.
  **L459 CN**: 声明 TableGen def `int_s390_vavgq`。
- **L460 EN**: Declares TableGen def `int_s390_vavglq`.
  **L460 CN**: 声明 TableGen def `int_s390_vavglq`。

### Lines 461-480

````tablegen
  def int_s390_veval  : SystemZQuaternaryInt<"veval", llvm_v16i8_ty>;
  def int_s390_vmahg  : SystemZTernary<"vmahg", llvm_v2i64_ty>;
  def int_s390_vmahq  : SystemZTernary<"vmahq", llvm_i128_ty>;
  def int_s390_vmalhg : SystemZTernary<"vmalhg", llvm_v2i64_ty>;
  def int_s390_vmalhq : SystemZTernary<"vmalhq", llvm_i128_ty>;
  def int_s390_vmaeg  : SystemZTernaryConv<"vmaeg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vmaleg : SystemZTernaryConv<"vmaleg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vmaog  : SystemZTernaryConv<"vmaog", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vmalog : SystemZTernaryConv<"vmalog", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vmhg   : SystemZBinary<"vmhg", llvm_v2i64_ty>;
  def int_s390_vmhq   : SystemZBinary<"vmhq", llvm_i128_ty>;
  def int_s390_vmlhg  : SystemZBinary<"vmlhg", llvm_v2i64_ty>;
  def int_s390_vmlhq  : SystemZBinary<"vmlhq", llvm_i128_ty>;
  def int_s390_vmeg   : SystemZBinaryConv<"vmeg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vmleg  : SystemZBinaryConv<"vmleg", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vmog   : SystemZBinaryConv<"vmog", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vmlog  : SystemZBinaryConv<"vmlog", llvm_i128_ty, llvm_v2i64_ty>;
  def int_s390_vceqqs : SystemZBinaryCC<llvm_i128_ty>;
  def int_s390_vchqs  : SystemZBinaryCC<llvm_i128_ty>;
  def int_s390_vchlqs : SystemZBinaryCC<llvm_i128_ty>;
````
- **L461 EN**: Declares TableGen def `int_s390_veval`.
  **L461 CN**: 声明 TableGen def `int_s390_veval`。
- **L462 EN**: Declares TableGen def `int_s390_vmahg`.
  **L462 CN**: 声明 TableGen def `int_s390_vmahg`。
- **L463 EN**: Declares TableGen def `int_s390_vmahq`.
  **L463 CN**: 声明 TableGen def `int_s390_vmahq`。
- **L464 EN**: Declares TableGen def `int_s390_vmalhg`.
  **L464 CN**: 声明 TableGen def `int_s390_vmalhg`。
- **L465 EN**: Declares TableGen def `int_s390_vmalhq`.
  **L465 CN**: 声明 TableGen def `int_s390_vmalhq`。
- **L466 EN**: Declares TableGen def `int_s390_vmaeg`.
  **L466 CN**: 声明 TableGen def `int_s390_vmaeg`。
- **L467 EN**: Declares TableGen def `int_s390_vmaleg`.
  **L467 CN**: 声明 TableGen def `int_s390_vmaleg`。
- **L468 EN**: Declares TableGen def `int_s390_vmaog`.
  **L468 CN**: 声明 TableGen def `int_s390_vmaog`。
- **L469 EN**: Declares TableGen def `int_s390_vmalog`.
  **L469 CN**: 声明 TableGen def `int_s390_vmalog`。
- **L470 EN**: Declares TableGen def `int_s390_vmhg`.
  **L470 CN**: 声明 TableGen def `int_s390_vmhg`。
- **L471 EN**: Declares TableGen def `int_s390_vmhq`.
  **L471 CN**: 声明 TableGen def `int_s390_vmhq`。
- **L472 EN**: Declares TableGen def `int_s390_vmlhg`.
  **L472 CN**: 声明 TableGen def `int_s390_vmlhg`。
- **L473 EN**: Declares TableGen def `int_s390_vmlhq`.
  **L473 CN**: 声明 TableGen def `int_s390_vmlhq`。
- **L474 EN**: Declares TableGen def `int_s390_vmeg`.
  **L474 CN**: 声明 TableGen def `int_s390_vmeg`。
- **L475 EN**: Declares TableGen def `int_s390_vmleg`.
  **L475 CN**: 声明 TableGen def `int_s390_vmleg`。
- **L476 EN**: Declares TableGen def `int_s390_vmog`.
  **L476 CN**: 声明 TableGen def `int_s390_vmog`。
- **L477 EN**: Declares TableGen def `int_s390_vmlog`.
  **L477 CN**: 声明 TableGen def `int_s390_vmlog`。
- **L478 EN**: Declares TableGen def `int_s390_vceqqs`.
  **L478 CN**: 声明 TableGen def `int_s390_vceqqs`。
- **L479 EN**: Declares TableGen def `int_s390_vchqs`.
  **L479 CN**: 声明 TableGen def `int_s390_vchqs`。
- **L480 EN**: Declares TableGen def `int_s390_vchlqs`.
  **L480 CN**: 声明 TableGen def `int_s390_vchlqs`。

### Lines 481-500

````tablegen
}

//===----------------------------------------------------------------------===//
//
// Misc intrinsics
//
//===----------------------------------------------------------------------===//

let TargetPrefix = "s390" in {
  def int_s390_sfpc : ClangBuiltin<"__builtin_s390_sfpc">,
                      Intrinsic<[], [llvm_i32_ty], []>;
  def int_s390_efpc : ClangBuiltin<"__builtin_s390_efpc">,
                      Intrinsic<[llvm_i32_ty], [], []>;

  def int_s390_tdc : Intrinsic<[llvm_i32_ty], [llvm_anyfloat_ty, llvm_i64_ty],
                               [IntrNoMem]>;

  // Instructions from the Miscellaneous Instruction Extensions Facility 4
  def int_s390_bdepg : ClangBuiltin<"__builtin_s390_bdepg">,
                       Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L483 EN**: Banner comment marking a file or section boundary.
  **L483 CN**: 横幅注释，用于标记文件或章节边界。
- **L484 EN**: Separator comment used for visual grouping.
  **L484 CN**: 用于视觉分组的分隔注释。
- **L485 EN**: Comment explains nearby logic, invariants, or intent: `Misc intrinsics`.
  **L485 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Misc intrinsics`。
- **L486 EN**: Separator comment used for visual grouping.
  **L486 CN**: 用于视觉分组的分隔注释。
- **L487 EN**: Banner comment marking a file or section boundary.
  **L487 CN**: 横幅注释，用于标记文件或章节边界。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L489 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L489 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L490 EN**: Declares TableGen def `int_s390_sfpc`.
  **L490 CN**: 声明 TableGen def `int_s390_sfpc`。
- **L491 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], []>;`.
  **L491 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], []>;`。
- **L492 EN**: Declares TableGen def `int_s390_efpc`.
  **L492 CN**: 声明 TableGen def `int_s390_efpc`。
- **L493 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [], []>;`.
  **L493 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [], []>;`。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L495 EN**: Declares TableGen def `int_s390_tdc`.
  **L495 CN**: 声明 TableGen def `int_s390_tdc`。
- **L496 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L496 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L497 EN**: Blank line separating nearby declarations or logic blocks.
  **L497 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L498 EN**: Comment explains nearby logic, invariants, or intent: `Instructions from the Miscellaneous Instruction Extensions Facility 4`.
  **L498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Instructions from the Miscellaneous Instruction Extensions Facility 4`。
- **L499 EN**: Declares TableGen def `int_s390_bdepg`.
  **L499 CN**: 声明 TableGen def `int_s390_bdepg`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。

### Lines 501-505

````tablegen
                                 [IntrNoMem]>;
  def int_s390_bextg : ClangBuiltin<"__builtin_s390_bextg">,
                       Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],
                                 [IntrNoMem]>;
}
````
- **L501 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L501 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L502 EN**: Declares TableGen def `int_s390_bextg`.
  **L502 CN**: 声明 TableGen def `int_s390_bextg`。
- **L503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`.
  **L503 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty],`。
- **L504 EN**: Executes a standalone statement or declaration: `[IntrNoMem]>;`.
  **L504 CN**: 执行一条独立语句或声明：`[IntrNoMem]>;`。
- **L505 EN**: Closes the current lexical scope or compound statement.
  **L505 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
