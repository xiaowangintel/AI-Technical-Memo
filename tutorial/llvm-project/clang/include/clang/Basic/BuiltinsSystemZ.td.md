# BuiltinsSystemZ.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsSystemZ.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: SystemZ Builtin function database *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsSystemZ` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 377

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````tablegen
//===-- BuiltinsSystemZ.td - SystemZ Builtin function database --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the SystemZ-specific builtin function database.
//
//===----------------------------------------------------------------------===//

include "clang/Basic/BuiltinsBase.td"

def SystemZPrefix : NamePrefix<"__builtin_s390_">;

class SystemZTargetBuiltin<string prototype> : TargetBuiltin {
  let Spellings = [NAME];
  let Prototype = prototype;
  let RequiredNamePrefix = SystemZPrefix;
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the SystemZ-specific builtin function database.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the SystemZ-specific builtin function database.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Includes TableGen file `"clang/Basic/BuiltinsBase.td"` so later records can reuse shared definitions.
  **L13 CN**: 引入 TableGen 文件 `"clang/Basic/BuiltinsBase.td"`，以便后续记录复用共享定义。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Declares TableGen def record `SystemZPrefix`.
  **L15 CN**: 声明 TableGen def 记录 `SystemZPrefix`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Declares TableGen class record `SystemZTargetBuiltin`.
  **L17 CN**: 声明 TableGen class 记录 `SystemZTargetBuiltin`。
- **L18 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Spellings = [NAME];`.
  **L18 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Spellings = [NAME];`。
- **L19 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Prototype = prototype;`.
  **L19 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Prototype = prototype;`。
- **L20 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredNamePrefix = SystemZPrefix;`.
  **L20 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredNamePrefix = SystemZPrefix;`。

### Lines 21-40

````tablegen
}

class SystemZNoPrefixTargetBuiltin<string prototype> : TargetBuiltin {
  let Spellings = [NAME];
  let Prototype = prototype;
}

// Transactional-memory intrinsics
let Features = "transactional-execution" in {
  let Attributes = [ReturnsTwice] in {
    def __builtin_tbegin : SystemZNoPrefixTargetBuiltin<"int(void *)">;
    def __builtin_tbegin_nofloat : SystemZNoPrefixTargetBuiltin<"int(void *)">;
  }
  let Attributes = [NoThrow, ReturnsTwice] in
    def __builtin_tbeginc : SystemZNoPrefixTargetBuiltin<"void()">;
  let Attributes = [NoReturn] in
    def __builtin_tabort : SystemZNoPrefixTargetBuiltin<"void(int)">;
  let Attributes = [NoThrow] in
    def __builtin_tend : SystemZNoPrefixTargetBuiltin<"int()">;
  let Attributes = [NoThrow, Const] in
````
- **L21 EN**: Closes the current lexical scope or compound statement.
  **L21 CN**: 结束当前词法作用域或复合语句块。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Declares TableGen class record `SystemZNoPrefixTargetBuiltin`.
  **L23 CN**: 声明 TableGen class 记录 `SystemZNoPrefixTargetBuiltin`。
- **L24 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Spellings = [NAME];`.
  **L24 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Spellings = [NAME];`。
- **L25 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Prototype = prototype;`.
  **L25 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Prototype = prototype;`。
- **L26 EN**: Closes the current lexical scope or compound statement.
  **L26 CN**: 结束当前词法作用域或复合语句块。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `Transactional-memory intrinsics`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Transactional-memory intrinsics`。
- **L29 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "transactional-execution" in {`.
  **L29 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "transactional-execution" in {`。
- **L30 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [ReturnsTwice] in {`.
  **L30 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [ReturnsTwice] in {`。
- **L31 EN**: Declares TableGen def record `__builtin_tbegin`.
  **L31 CN**: 声明 TableGen def 记录 `__builtin_tbegin`。
- **L32 EN**: Declares TableGen def record `__builtin_tbegin_nofloat`.
  **L32 CN**: 声明 TableGen def 记录 `__builtin_tbegin_nofloat`。
- **L33 EN**: Closes the current lexical scope or compound statement.
  **L33 CN**: 结束当前词法作用域或复合语句块。
- **L34 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, ReturnsTwice] in`.
  **L34 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, ReturnsTwice] in`。
- **L35 EN**: Declares TableGen def record `__builtin_tbeginc`.
  **L35 CN**: 声明 TableGen def 记录 `__builtin_tbeginc`。
- **L36 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoReturn] in`.
  **L36 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoReturn] in`。
- **L37 EN**: Declares TableGen def record `__builtin_tabort`.
  **L37 CN**: 声明 TableGen def 记录 `__builtin_tabort`。
- **L38 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in`.
  **L38 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in`。
- **L39 EN**: Declares TableGen def record `__builtin_tend`.
  **L39 CN**: 声明 TableGen def 记录 `__builtin_tend`。
- **L40 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in`.
  **L40 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in`。

### Lines 41-60

````tablegen
    def __builtin_tx_nesting_depth : SystemZNoPrefixTargetBuiltin<"int()">;
  let Attributes = [NoThrow] in
    def __builtin_tx_assist : SystemZNoPrefixTargetBuiltin<"void(int)">;
  def __builtin_non_tx_store : SystemZNoPrefixTargetBuiltin<"void(unsigned long int *, unsigned long int)">;
}

// Vector intrinsics.
// These all map directly to z instructions, except that some variants ending
// in "s" have a final "int *" that receives the post-instruction CC value.

// Vector support instructions (chapter 21 of the PoP)
let Attributes = [NoThrow, Const], Features = "vector" in
  def lcbb : SystemZTargetBuiltin<"unsigned int(void const *, int)">;
let Features = "vector" in {
  def vlbb : SystemZTargetBuiltin<"_Vector<16, signed char>(void const *, int)">;
  def vll : SystemZTargetBuiltin<"_Vector<16, signed char>(unsigned int, void const *)">;
  def vstl : SystemZTargetBuiltin<"void(_Vector<16, signed char>, unsigned int, void *)">;
}
let Attributes = [NoThrow, Const], Features = "vector" in {
  def vperm : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
````
- **L41 EN**: Declares TableGen def record `__builtin_tx_nesting_depth`.
  **L41 CN**: 声明 TableGen def 记录 `__builtin_tx_nesting_depth`。
- **L42 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in`.
  **L42 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in`。
- **L43 EN**: Declares TableGen def record `__builtin_tx_assist`.
  **L43 CN**: 声明 TableGen def 记录 `__builtin_tx_assist`。
- **L44 EN**: Declares TableGen def record `__builtin_non_tx_store`.
  **L44 CN**: 声明 TableGen def 记录 `__builtin_non_tx_store`。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Comment explains nearby logic, constraints, or intent: `Vector intrinsics.`.
  **L47 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector intrinsics.`。
- **L48 EN**: Comment explains nearby logic, constraints, or intent: `These all map directly to z instructions, except that some variants ending`.
  **L48 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These all map directly to z instructions, except that some variants ending`。
- **L49 EN**: Comment explains nearby logic, constraints, or intent: `in "s" have a final "int *" that receives the post-instruction CC value.`.
  **L49 CN**: 注释解释附近代码的逻辑、约束或设计意图：`in "s" have a final "int *" that receives the post-instruction CC value.`。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L51 EN**: Comment explains nearby logic, constraints, or intent: `Vector support instructions (chapter 21 of the PoP)`.
  **L51 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector support instructions (chapter 21 of the PoP)`。
- **L52 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector" in`.
  **L52 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector" in`。
- **L53 EN**: Declares TableGen def record `lcbb`.
  **L53 CN**: 声明 TableGen def 记录 `lcbb`。
- **L54 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "vector" in {`.
  **L54 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "vector" in {`。
- **L55 EN**: Declares TableGen def record `vlbb`.
  **L55 CN**: 声明 TableGen def 记录 `vlbb`。
- **L56 EN**: Declares TableGen def record `vll`.
  **L56 CN**: 声明 TableGen def 记录 `vll`。
- **L57 EN**: Declares TableGen def record `vstl`.
  **L57 CN**: 声明 TableGen def 记录 `vstl`。
- **L58 EN**: Closes the current lexical scope or compound statement.
  **L58 CN**: 结束当前词法作用域或复合语句块。
- **L59 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector" in {`.
  **L59 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector" in {`。
- **L60 EN**: Declares TableGen def record `vperm`.
  **L60 CN**: 声明 TableGen def 记录 `vperm`。

### Lines 61-80

````tablegen
  def vpdi : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, int)">;
  def vpksh : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<8, signed short>, _Vector<8, signed short>)">;
  def vpkshs : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<8, signed short>, _Vector<8, signed short>, int *)">;
  def vpksf : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<4, signed int>, _Vector<4, signed int>)">;
  def vpksfs : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<4, signed int>, _Vector<4, signed int>, int *)">;
  def vpksg : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<2, signed long long int>, _Vector<2, signed long long int>)">;
  def vpksgs : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<2, signed long long int>, _Vector<2, signed long long int>, int *)">;
  def vpklsh : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vpklshs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int *)">;
  def vpklsf : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vpklsfs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int *)">;
  def vpklsg : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vpklsgs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, int *)">;
  def vuphb : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<16, signed char>)">;
  def vuphh : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<8, signed short>)">;
  def vuphf : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<4, signed int>)">;
  def vuplb : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<16, signed char>)">;
  def vuplhw : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<8, signed short>)">;
  def vuplf : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<4, signed int>)">;
  def vuplhb : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>)">;
````
- **L61 EN**: Declares TableGen def record `vpdi`.
  **L61 CN**: 声明 TableGen def 记录 `vpdi`。
- **L62 EN**: Declares TableGen def record `vpksh`.
  **L62 CN**: 声明 TableGen def 记录 `vpksh`。
- **L63 EN**: Declares TableGen def record `vpkshs`.
  **L63 CN**: 声明 TableGen def 记录 `vpkshs`。
- **L64 EN**: Declares TableGen def record `vpksf`.
  **L64 CN**: 声明 TableGen def 记录 `vpksf`。
- **L65 EN**: Declares TableGen def record `vpksfs`.
  **L65 CN**: 声明 TableGen def 记录 `vpksfs`。
- **L66 EN**: Declares TableGen def record `vpksg`.
  **L66 CN**: 声明 TableGen def 记录 `vpksg`。
- **L67 EN**: Declares TableGen def record `vpksgs`.
  **L67 CN**: 声明 TableGen def 记录 `vpksgs`。
- **L68 EN**: Declares TableGen def record `vpklsh`.
  **L68 CN**: 声明 TableGen def 记录 `vpklsh`。
- **L69 EN**: Declares TableGen def record `vpklshs`.
  **L69 CN**: 声明 TableGen def 记录 `vpklshs`。
- **L70 EN**: Declares TableGen def record `vpklsf`.
  **L70 CN**: 声明 TableGen def 记录 `vpklsf`。
- **L71 EN**: Declares TableGen def record `vpklsfs`.
  **L71 CN**: 声明 TableGen def 记录 `vpklsfs`。
- **L72 EN**: Declares TableGen def record `vpklsg`.
  **L72 CN**: 声明 TableGen def 记录 `vpklsg`。
- **L73 EN**: Declares TableGen def record `vpklsgs`.
  **L73 CN**: 声明 TableGen def 记录 `vpklsgs`。
- **L74 EN**: Declares TableGen def record `vuphb`.
  **L74 CN**: 声明 TableGen def 记录 `vuphb`。
- **L75 EN**: Declares TableGen def record `vuphh`.
  **L75 CN**: 声明 TableGen def 记录 `vuphh`。
- **L76 EN**: Declares TableGen def record `vuphf`.
  **L76 CN**: 声明 TableGen def 记录 `vuphf`。
- **L77 EN**: Declares TableGen def record `vuplb`.
  **L77 CN**: 声明 TableGen def 记录 `vuplb`。
- **L78 EN**: Declares TableGen def record `vuplhw`.
  **L78 CN**: 声明 TableGen def 记录 `vuplhw`。
- **L79 EN**: Declares TableGen def record `vuplf`.
  **L79 CN**: 声明 TableGen def 记录 `vuplf`。
- **L80 EN**: Declares TableGen def record `vuplhb`.
  **L80 CN**: 声明 TableGen def 记录 `vuplhb`。

### Lines 81-100

````tablegen
  def vuplhh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>)">;
  def vuplhf : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>)">;
  def vupllb : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>)">;
  def vupllh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>)">;
  def vupllf : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>)">;
}

// Vector integer instructions (chapter 22 of the PoP)
let Attributes = [NoThrow, Const], Features = "vector" in {
  def vaq : SystemZTargetBuiltin<"__int128_t(__int128_t, __int128_t)">;
  def vacq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t, __uint128_t)">;
  def vaccb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vacch : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vaccf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vaccg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vaccq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t)">;
  def vacccq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t, __uint128_t)">;
  def vavgb : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<16, signed char>, _Vector<16, signed char>)">;
  def vavgh : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<8, signed short>, _Vector<8, signed short>)">;
  def vavgf : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, signed int>, _Vector<4, signed int>)">;
````
- **L81 EN**: Declares TableGen def record `vuplhh`.
  **L81 CN**: 声明 TableGen def 记录 `vuplhh`。
- **L82 EN**: Declares TableGen def record `vuplhf`.
  **L82 CN**: 声明 TableGen def 记录 `vuplhf`。
- **L83 EN**: Declares TableGen def record `vupllb`.
  **L83 CN**: 声明 TableGen def 记录 `vupllb`。
- **L84 EN**: Declares TableGen def record `vupllh`.
  **L84 CN**: 声明 TableGen def 记录 `vupllh`。
- **L85 EN**: Declares TableGen def record `vupllf`.
  **L85 CN**: 声明 TableGen def 记录 `vupllf`。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L88 EN**: Comment explains nearby logic, constraints, or intent: `Vector integer instructions (chapter 22 of the PoP)`.
  **L88 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector integer instructions (chapter 22 of the PoP)`。
- **L89 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector" in {`.
  **L89 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector" in {`。
- **L90 EN**: Declares TableGen def record `vaq`.
  **L90 CN**: 声明 TableGen def 记录 `vaq`。
- **L91 EN**: Declares TableGen def record `vacq`.
  **L91 CN**: 声明 TableGen def 记录 `vacq`。
- **L92 EN**: Declares TableGen def record `vaccb`.
  **L92 CN**: 声明 TableGen def 记录 `vaccb`。
- **L93 EN**: Declares TableGen def record `vacch`.
  **L93 CN**: 声明 TableGen def 记录 `vacch`。
- **L94 EN**: Declares TableGen def record `vaccf`.
  **L94 CN**: 声明 TableGen def 记录 `vaccf`。
- **L95 EN**: Declares TableGen def record `vaccg`.
  **L95 CN**: 声明 TableGen def 记录 `vaccg`。
- **L96 EN**: Declares TableGen def record `vaccq`.
  **L96 CN**: 声明 TableGen def 记录 `vaccq`。
- **L97 EN**: Declares TableGen def record `vacccq`.
  **L97 CN**: 声明 TableGen def 记录 `vacccq`。
- **L98 EN**: Declares TableGen def record `vavgb`.
  **L98 CN**: 声明 TableGen def 记录 `vavgb`。
- **L99 EN**: Declares TableGen def record `vavgh`.
  **L99 CN**: 声明 TableGen def 记录 `vavgh`。
- **L100 EN**: Declares TableGen def record `vavgf`.
  **L100 CN**: 声明 TableGen def 记录 `vavgf`。

### Lines 101-120

````tablegen
  def vavgg : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, signed long long int>, _Vector<2, signed long long int>)">;
  def vavglb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vavglh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vavglf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vavglg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vceqbs : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
  def vceqhs : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int *)">;
  def vceqfs : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int *)">;
  def vceqgs : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, int *)">;
  def vchbs : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<16, signed char>, _Vector<16, signed char>, int *)">;
  def vchhs : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<8, signed short>, _Vector<8, signed short>, int *)">;
  def vchfs : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, signed int>, _Vector<4, signed int>, int *)">;
  def vchgs : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, signed long long int>, _Vector<2, signed long long int>, int *)">;
  def vchlbs : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
  def vchlhs : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int *)">;
  def vchlfs : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int *)">;
  def vchlgs : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, int *)">;
  def vcksm : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vclzb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>)">;
  def vclzh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>)">;
````
- **L101 EN**: Declares TableGen def record `vavgg`.
  **L101 CN**: 声明 TableGen def 记录 `vavgg`。
- **L102 EN**: Declares TableGen def record `vavglb`.
  **L102 CN**: 声明 TableGen def 记录 `vavglb`。
- **L103 EN**: Declares TableGen def record `vavglh`.
  **L103 CN**: 声明 TableGen def 记录 `vavglh`。
- **L104 EN**: Declares TableGen def record `vavglf`.
  **L104 CN**: 声明 TableGen def 记录 `vavglf`。
- **L105 EN**: Declares TableGen def record `vavglg`.
  **L105 CN**: 声明 TableGen def 记录 `vavglg`。
- **L106 EN**: Declares TableGen def record `vceqbs`.
  **L106 CN**: 声明 TableGen def 记录 `vceqbs`。
- **L107 EN**: Declares TableGen def record `vceqhs`.
  **L107 CN**: 声明 TableGen def 记录 `vceqhs`。
- **L108 EN**: Declares TableGen def record `vceqfs`.
  **L108 CN**: 声明 TableGen def 记录 `vceqfs`。
- **L109 EN**: Declares TableGen def record `vceqgs`.
  **L109 CN**: 声明 TableGen def 记录 `vceqgs`。
- **L110 EN**: Declares TableGen def record `vchbs`.
  **L110 CN**: 声明 TableGen def 记录 `vchbs`。
- **L111 EN**: Declares TableGen def record `vchhs`.
  **L111 CN**: 声明 TableGen def 记录 `vchhs`。
- **L112 EN**: Declares TableGen def record `vchfs`.
  **L112 CN**: 声明 TableGen def 记录 `vchfs`。
- **L113 EN**: Declares TableGen def record `vchgs`.
  **L113 CN**: 声明 TableGen def 记录 `vchgs`。
- **L114 EN**: Declares TableGen def record `vchlbs`.
  **L114 CN**: 声明 TableGen def 记录 `vchlbs`。
- **L115 EN**: Declares TableGen def record `vchlhs`.
  **L115 CN**: 声明 TableGen def 记录 `vchlhs`。
- **L116 EN**: Declares TableGen def record `vchlfs`.
  **L116 CN**: 声明 TableGen def 记录 `vchlfs`。
- **L117 EN**: Declares TableGen def record `vchlgs`.
  **L117 CN**: 声明 TableGen def 记录 `vchlgs`。
- **L118 EN**: Declares TableGen def record `vcksm`.
  **L118 CN**: 声明 TableGen def 记录 `vcksm`。
- **L119 EN**: Declares TableGen def record `vclzb`.
  **L119 CN**: 声明 TableGen def 记录 `vclzb`。
- **L120 EN**: Declares TableGen def record `vclzh`.
  **L120 CN**: 声明 TableGen def 记录 `vclzh`。

### Lines 121-140

````tablegen
  def vclzf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>)">;
  def vclzg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>)">;
  def vctzb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>)">;
  def vctzh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>)">;
  def vctzf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>)">;
  def vctzg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>)">;
  def verimb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def verimh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<8, unsigned short>, int)">;
  def verimf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<4, unsigned int>, int)">;
  def verimg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, int)">;
  def verllb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, unsigned char)">;
  def verllh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, unsigned char)">;
  def verllf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, unsigned char)">;
  def verllg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, unsigned char)">;
  def verllvb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def verllvh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def verllvf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def verllvg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vgfmb : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vgfmh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
````
- **L121 EN**: Declares TableGen def record `vclzf`.
  **L121 CN**: 声明 TableGen def 记录 `vclzf`。
- **L122 EN**: Declares TableGen def record `vclzg`.
  **L122 CN**: 声明 TableGen def 记录 `vclzg`。
- **L123 EN**: Declares TableGen def record `vctzb`.
  **L123 CN**: 声明 TableGen def 记录 `vctzb`。
- **L124 EN**: Declares TableGen def record `vctzh`.
  **L124 CN**: 声明 TableGen def 记录 `vctzh`。
- **L125 EN**: Declares TableGen def record `vctzf`.
  **L125 CN**: 声明 TableGen def 记录 `vctzf`。
- **L126 EN**: Declares TableGen def record `vctzg`.
  **L126 CN**: 声明 TableGen def 记录 `vctzg`。
- **L127 EN**: Declares TableGen def record `verimb`.
  **L127 CN**: 声明 TableGen def 记录 `verimb`。
- **L128 EN**: Declares TableGen def record `verimh`.
  **L128 CN**: 声明 TableGen def 记录 `verimh`。
- **L129 EN**: Declares TableGen def record `verimf`.
  **L129 CN**: 声明 TableGen def 记录 `verimf`。
- **L130 EN**: Declares TableGen def record `verimg`.
  **L130 CN**: 声明 TableGen def 记录 `verimg`。
- **L131 EN**: Declares TableGen def record `verllb`.
  **L131 CN**: 声明 TableGen def 记录 `verllb`。
- **L132 EN**: Declares TableGen def record `verllh`.
  **L132 CN**: 声明 TableGen def 记录 `verllh`。
- **L133 EN**: Declares TableGen def record `verllf`.
  **L133 CN**: 声明 TableGen def 记录 `verllf`。
- **L134 EN**: Declares TableGen def record `verllg`.
  **L134 CN**: 声明 TableGen def 记录 `verllg`。
- **L135 EN**: Declares TableGen def record `verllvb`.
  **L135 CN**: 声明 TableGen def 记录 `verllvb`。
- **L136 EN**: Declares TableGen def record `verllvh`.
  **L136 CN**: 声明 TableGen def 记录 `verllvh`。
- **L137 EN**: Declares TableGen def record `verllvf`.
  **L137 CN**: 声明 TableGen def 记录 `verllvf`。
- **L138 EN**: Declares TableGen def record `verllvg`.
  **L138 CN**: 声明 TableGen def 记录 `verllvg`。
- **L139 EN**: Declares TableGen def record `vgfmb`.
  **L139 CN**: 声明 TableGen def 记录 `vgfmb`。
- **L140 EN**: Declares TableGen def record `vgfmh`.
  **L140 CN**: 声明 TableGen def 记录 `vgfmh`。

### Lines 141-160

````tablegen
  def vgfmf : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vgfmg : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vgfmab : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<8, unsigned short>)">;
  def vgfmah : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<4, unsigned int>)">;
  def vgfmaf : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<2, unsigned long long int>)">;
  def vgfmag : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, __uint128_t)">;
  def vmahb : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<16, signed char>, _Vector<16, signed char>, _Vector<16, signed char>)">;
  def vmahh : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<8, signed short>, _Vector<8, signed short>, _Vector<8, signed short>)">;
  def vmahf : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, signed int>, _Vector<4, signed int>, _Vector<4, signed int>)">;
  def vmalhb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vmalhh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vmalhf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vmaeb : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<16, signed char>, _Vector<16, signed char>, _Vector<8, signed short>)">;
  def vmaeh : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<8, signed short>, _Vector<8, signed short>, _Vector<4, signed int>)">;
  def vmaef : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<4, signed int>, _Vector<4, signed int>, _Vector<2, signed long long int>)">;
  def vmaleb : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<8, unsigned short>)">;
  def vmaleh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<4, unsigned int>)">;
  def vmalef : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<2, unsigned long long int>)">;
  def vmaob : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<16, signed char>, _Vector<16, signed char>, _Vector<8, signed short>)">;
  def vmaoh : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<8, signed short>, _Vector<8, signed short>, _Vector<4, signed int>)">;
````
- **L141 EN**: Declares TableGen def record `vgfmf`.
  **L141 CN**: 声明 TableGen def 记录 `vgfmf`。
- **L142 EN**: Declares TableGen def record `vgfmg`.
  **L142 CN**: 声明 TableGen def 记录 `vgfmg`。
- **L143 EN**: Declares TableGen def record `vgfmab`.
  **L143 CN**: 声明 TableGen def 记录 `vgfmab`。
- **L144 EN**: Declares TableGen def record `vgfmah`.
  **L144 CN**: 声明 TableGen def 记录 `vgfmah`。
- **L145 EN**: Declares TableGen def record `vgfmaf`.
  **L145 CN**: 声明 TableGen def 记录 `vgfmaf`。
- **L146 EN**: Declares TableGen def record `vgfmag`.
  **L146 CN**: 声明 TableGen def 记录 `vgfmag`。
- **L147 EN**: Declares TableGen def record `vmahb`.
  **L147 CN**: 声明 TableGen def 记录 `vmahb`。
- **L148 EN**: Declares TableGen def record `vmahh`.
  **L148 CN**: 声明 TableGen def 记录 `vmahh`。
- **L149 EN**: Declares TableGen def record `vmahf`.
  **L149 CN**: 声明 TableGen def 记录 `vmahf`。
- **L150 EN**: Declares TableGen def record `vmalhb`.
  **L150 CN**: 声明 TableGen def 记录 `vmalhb`。
- **L151 EN**: Declares TableGen def record `vmalhh`.
  **L151 CN**: 声明 TableGen def 记录 `vmalhh`。
- **L152 EN**: Declares TableGen def record `vmalhf`.
  **L152 CN**: 声明 TableGen def 记录 `vmalhf`。
- **L153 EN**: Declares TableGen def record `vmaeb`.
  **L153 CN**: 声明 TableGen def 记录 `vmaeb`。
- **L154 EN**: Declares TableGen def record `vmaeh`.
  **L154 CN**: 声明 TableGen def 记录 `vmaeh`。
- **L155 EN**: Declares TableGen def record `vmaef`.
  **L155 CN**: 声明 TableGen def 记录 `vmaef`。
- **L156 EN**: Declares TableGen def record `vmaleb`.
  **L156 CN**: 声明 TableGen def 记录 `vmaleb`。
- **L157 EN**: Declares TableGen def record `vmaleh`.
  **L157 CN**: 声明 TableGen def 记录 `vmaleh`。
- **L158 EN**: Declares TableGen def record `vmalef`.
  **L158 CN**: 声明 TableGen def 记录 `vmalef`。
- **L159 EN**: Declares TableGen def record `vmaob`.
  **L159 CN**: 声明 TableGen def 记录 `vmaob`。
- **L160 EN**: Declares TableGen def record `vmaoh`.
  **L160 CN**: 声明 TableGen def 记录 `vmaoh`。

### Lines 161-180

````tablegen
  def vmaof : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<4, signed int>, _Vector<4, signed int>, _Vector<2, signed long long int>)">;
  def vmalob : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<8, unsigned short>)">;
  def vmaloh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<4, unsigned int>)">;
  def vmalof : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<2, unsigned long long int>)">;
  def vmhb : SystemZTargetBuiltin<"_Vector<16, signed char>(_Vector<16, signed char>, _Vector<16, signed char>)">;
  def vmhh : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<8, signed short>, _Vector<8, signed short>)">;
  def vmhf : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, signed int>, _Vector<4, signed int>)">;
  def vmlhb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vmlhh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vmlhf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vmeb : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<16, signed char>, _Vector<16, signed char>)">;
  def vmeh : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<8, signed short>, _Vector<8, signed short>)">;
  def vmef : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<4, signed int>, _Vector<4, signed int>)">;
  def vmleb : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vmleh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vmlef : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vmob : SystemZTargetBuiltin<"_Vector<8, signed short>(_Vector<16, signed char>, _Vector<16, signed char>)">;
  def vmoh : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<8, signed short>, _Vector<8, signed short>)">;
  def vmof : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<4, signed int>, _Vector<4, signed int>)">;
  def vmlob : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
````
- **L161 EN**: Declares TableGen def record `vmaof`.
  **L161 CN**: 声明 TableGen def 记录 `vmaof`。
- **L162 EN**: Declares TableGen def record `vmalob`.
  **L162 CN**: 声明 TableGen def 记录 `vmalob`。
- **L163 EN**: Declares TableGen def record `vmaloh`.
  **L163 CN**: 声明 TableGen def 记录 `vmaloh`。
- **L164 EN**: Declares TableGen def record `vmalof`.
  **L164 CN**: 声明 TableGen def 记录 `vmalof`。
- **L165 EN**: Declares TableGen def record `vmhb`.
  **L165 CN**: 声明 TableGen def 记录 `vmhb`。
- **L166 EN**: Declares TableGen def record `vmhh`.
  **L166 CN**: 声明 TableGen def 记录 `vmhh`。
- **L167 EN**: Declares TableGen def record `vmhf`.
  **L167 CN**: 声明 TableGen def 记录 `vmhf`。
- **L168 EN**: Declares TableGen def record `vmlhb`.
  **L168 CN**: 声明 TableGen def 记录 `vmlhb`。
- **L169 EN**: Declares TableGen def record `vmlhh`.
  **L169 CN**: 声明 TableGen def 记录 `vmlhh`。
- **L170 EN**: Declares TableGen def record `vmlhf`.
  **L170 CN**: 声明 TableGen def 记录 `vmlhf`。
- **L171 EN**: Declares TableGen def record `vmeb`.
  **L171 CN**: 声明 TableGen def 记录 `vmeb`。
- **L172 EN**: Declares TableGen def record `vmeh`.
  **L172 CN**: 声明 TableGen def 记录 `vmeh`。
- **L173 EN**: Declares TableGen def record `vmef`.
  **L173 CN**: 声明 TableGen def 记录 `vmef`。
- **L174 EN**: Declares TableGen def record `vmleb`.
  **L174 CN**: 声明 TableGen def 记录 `vmleb`。
- **L175 EN**: Declares TableGen def record `vmleh`.
  **L175 CN**: 声明 TableGen def 记录 `vmleh`。
- **L176 EN**: Declares TableGen def record `vmlef`.
  **L176 CN**: 声明 TableGen def 记录 `vmlef`。
- **L177 EN**: Declares TableGen def record `vmob`.
  **L177 CN**: 声明 TableGen def 记录 `vmob`。
- **L178 EN**: Declares TableGen def record `vmoh`.
  **L178 CN**: 声明 TableGen def 记录 `vmoh`。
- **L179 EN**: Declares TableGen def record `vmof`.
  **L179 CN**: 声明 TableGen def 记录 `vmof`。
- **L180 EN**: Declares TableGen def record `vmlob`.
  **L180 CN**: 声明 TableGen def 记录 `vmlob`。

### Lines 181-200

````tablegen
  def vmloh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vmlof : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vsq : SystemZTargetBuiltin<"__int128_t(__int128_t, __int128_t)">;
  def vsbcbiq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t, __uint128_t)">;
  def vsbiq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t, __uint128_t)">;
  def vscbib : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vscbih : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vscbif : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vscbig : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vscbiq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t)">;
  def vsl : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vslb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vsldb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vsra : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vsrab : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vsrl : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vsrlb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vsumb : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vsumh : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vsumgh : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
````
- **L181 EN**: Declares TableGen def record `vmloh`.
  **L181 CN**: 声明 TableGen def 记录 `vmloh`。
- **L182 EN**: Declares TableGen def record `vmlof`.
  **L182 CN**: 声明 TableGen def 记录 `vmlof`。
- **L183 EN**: Declares TableGen def record `vsq`.
  **L183 CN**: 声明 TableGen def 记录 `vsq`。
- **L184 EN**: Declares TableGen def record `vsbcbiq`.
  **L184 CN**: 声明 TableGen def 记录 `vsbcbiq`。
- **L185 EN**: Declares TableGen def record `vsbiq`.
  **L185 CN**: 声明 TableGen def 记录 `vsbiq`。
- **L186 EN**: Declares TableGen def record `vscbib`.
  **L186 CN**: 声明 TableGen def 记录 `vscbib`。
- **L187 EN**: Declares TableGen def record `vscbih`.
  **L187 CN**: 声明 TableGen def 记录 `vscbih`。
- **L188 EN**: Declares TableGen def record `vscbif`.
  **L188 CN**: 声明 TableGen def 记录 `vscbif`。
- **L189 EN**: Declares TableGen def record `vscbig`.
  **L189 CN**: 声明 TableGen def 记录 `vscbig`。
- **L190 EN**: Declares TableGen def record `vscbiq`.
  **L190 CN**: 声明 TableGen def 记录 `vscbiq`。
- **L191 EN**: Declares TableGen def record `vsl`.
  **L191 CN**: 声明 TableGen def 记录 `vsl`。
- **L192 EN**: Declares TableGen def record `vslb`.
  **L192 CN**: 声明 TableGen def 记录 `vslb`。
- **L193 EN**: Declares TableGen def record `vsldb`.
  **L193 CN**: 声明 TableGen def 记录 `vsldb`。
- **L194 EN**: Declares TableGen def record `vsra`.
  **L194 CN**: 声明 TableGen def 记录 `vsra`。
- **L195 EN**: Declares TableGen def record `vsrab`.
  **L195 CN**: 声明 TableGen def 记录 `vsrab`。
- **L196 EN**: Declares TableGen def record `vsrl`.
  **L196 CN**: 声明 TableGen def 记录 `vsrl`。
- **L197 EN**: Declares TableGen def record `vsrlb`.
  **L197 CN**: 声明 TableGen def 记录 `vsrlb`。
- **L198 EN**: Declares TableGen def record `vsumb`.
  **L198 CN**: 声明 TableGen def 记录 `vsumb`。
- **L199 EN**: Declares TableGen def record `vsumh`.
  **L199 CN**: 声明 TableGen def 记录 `vsumh`。
- **L200 EN**: Declares TableGen def record `vsumgh`.
  **L200 CN**: 声明 TableGen def 记录 `vsumgh`。

### Lines 201-220

````tablegen
  def vsumgf : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vsumqf : SystemZTargetBuiltin<"__uint128_t(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vsumqg : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vtm : SystemZTargetBuiltin<"int(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
}

// Vector string instructions (chapter 23 of the PoP)
let Attributes = [NoThrow, Const], Features = "vector" in {
  def vfaeb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vfaebs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int, int *)">;
  def vfaeh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int)">;
  def vfaehs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int, int *)">;
  def vfaef : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int)">;
  def vfaefs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int, int *)">;
  def vfaezb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vfaezbs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int, int *)">;
  def vfaezh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int)">;
  def vfaezhs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int, int *)">;
  def vfaezf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int)">;
  def vfaezfs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int, int *)">;
````
- **L201 EN**: Declares TableGen def record `vsumgf`.
  **L201 CN**: 声明 TableGen def 记录 `vsumgf`。
- **L202 EN**: Declares TableGen def record `vsumqf`.
  **L202 CN**: 声明 TableGen def 记录 `vsumqf`。
- **L203 EN**: Declares TableGen def record `vsumqg`.
  **L203 CN**: 声明 TableGen def 记录 `vsumqg`。
- **L204 EN**: Declares TableGen def record `vtm`.
  **L204 CN**: 声明 TableGen def 记录 `vtm`。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Comment explains nearby logic, constraints, or intent: `Vector string instructions (chapter 23 of the PoP)`.
  **L207 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector string instructions (chapter 23 of the PoP)`。
- **L208 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector" in {`.
  **L208 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector" in {`。
- **L209 EN**: Declares TableGen def record `vfaeb`.
  **L209 CN**: 声明 TableGen def 记录 `vfaeb`。
- **L210 EN**: Declares TableGen def record `vfaebs`.
  **L210 CN**: 声明 TableGen def 记录 `vfaebs`。
- **L211 EN**: Declares TableGen def record `vfaeh`.
  **L211 CN**: 声明 TableGen def 记录 `vfaeh`。
- **L212 EN**: Declares TableGen def record `vfaehs`.
  **L212 CN**: 声明 TableGen def 记录 `vfaehs`。
- **L213 EN**: Declares TableGen def record `vfaef`.
  **L213 CN**: 声明 TableGen def 记录 `vfaef`。
- **L214 EN**: Declares TableGen def record `vfaefs`.
  **L214 CN**: 声明 TableGen def 记录 `vfaefs`。
- **L215 EN**: Declares TableGen def record `vfaezb`.
  **L215 CN**: 声明 TableGen def 记录 `vfaezb`。
- **L216 EN**: Declares TableGen def record `vfaezbs`.
  **L216 CN**: 声明 TableGen def 记录 `vfaezbs`。
- **L217 EN**: Declares TableGen def record `vfaezh`.
  **L217 CN**: 声明 TableGen def 记录 `vfaezh`。
- **L218 EN**: Declares TableGen def record `vfaezhs`.
  **L218 CN**: 声明 TableGen def 记录 `vfaezhs`。
- **L219 EN**: Declares TableGen def record `vfaezf`.
  **L219 CN**: 声明 TableGen def 记录 `vfaezf`。
- **L220 EN**: Declares TableGen def record `vfaezfs`.
  **L220 CN**: 声明 TableGen def 记录 `vfaezfs`。

### Lines 221-240

````tablegen
  def vfeeb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vfeebs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
  def vfeeh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vfeehs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int *)">;
  def vfeef : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vfeefs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int *)">;
  def vfeezb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vfeezbs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
  def vfeezh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vfeezhs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int *)">;
  def vfeezf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vfeezfs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int *)">;
  def vfeneb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vfenebs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
  def vfeneh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vfenehs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int *)">;
  def vfenef : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vfenefs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int *)">;
  def vfenezb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vfenezbs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
````
- **L221 EN**: Declares TableGen def record `vfeeb`.
  **L221 CN**: 声明 TableGen def 记录 `vfeeb`。
- **L222 EN**: Declares TableGen def record `vfeebs`.
  **L222 CN**: 声明 TableGen def 记录 `vfeebs`。
- **L223 EN**: Declares TableGen def record `vfeeh`.
  **L223 CN**: 声明 TableGen def 记录 `vfeeh`。
- **L224 EN**: Declares TableGen def record `vfeehs`.
  **L224 CN**: 声明 TableGen def 记录 `vfeehs`。
- **L225 EN**: Declares TableGen def record `vfeef`.
  **L225 CN**: 声明 TableGen def 记录 `vfeef`。
- **L226 EN**: Declares TableGen def record `vfeefs`.
  **L226 CN**: 声明 TableGen def 记录 `vfeefs`。
- **L227 EN**: Declares TableGen def record `vfeezb`.
  **L227 CN**: 声明 TableGen def 记录 `vfeezb`。
- **L228 EN**: Declares TableGen def record `vfeezbs`.
  **L228 CN**: 声明 TableGen def 记录 `vfeezbs`。
- **L229 EN**: Declares TableGen def record `vfeezh`.
  **L229 CN**: 声明 TableGen def 记录 `vfeezh`。
- **L230 EN**: Declares TableGen def record `vfeezhs`.
  **L230 CN**: 声明 TableGen def 记录 `vfeezhs`。
- **L231 EN**: Declares TableGen def record `vfeezf`.
  **L231 CN**: 声明 TableGen def 记录 `vfeezf`。
- **L232 EN**: Declares TableGen def record `vfeezfs`.
  **L232 CN**: 声明 TableGen def 记录 `vfeezfs`。
- **L233 EN**: Declares TableGen def record `vfeneb`.
  **L233 CN**: 声明 TableGen def 记录 `vfeneb`。
- **L234 EN**: Declares TableGen def record `vfenebs`.
  **L234 CN**: 声明 TableGen def 记录 `vfenebs`。
- **L235 EN**: Declares TableGen def record `vfeneh`.
  **L235 CN**: 声明 TableGen def 记录 `vfeneh`。
- **L236 EN**: Declares TableGen def record `vfenehs`.
  **L236 CN**: 声明 TableGen def 记录 `vfenehs`。
- **L237 EN**: Declares TableGen def record `vfenef`.
  **L237 CN**: 声明 TableGen def 记录 `vfenef`。
- **L238 EN**: Declares TableGen def record `vfenefs`.
  **L238 CN**: 声明 TableGen def 记录 `vfenefs`。
- **L239 EN**: Declares TableGen def record `vfenezb`.
  **L239 CN**: 声明 TableGen def 记录 `vfenezb`。
- **L240 EN**: Declares TableGen def record `vfenezbs`.
  **L240 CN**: 声明 TableGen def 记录 `vfenezbs`。

### Lines 241-260

````tablegen
  def vfenezh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>)">;
  def vfenezhs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, int *)">;
  def vfenezf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>)">;
  def vfenezfs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, int *)">;
  def vistrb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>)">;
  def vistrbs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, int *)">;
  def vistrh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>)">;
  def vistrhs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, int *)">;
  def vistrf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>)">;
  def vistrfs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, int *)">;
  def vstrcb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vstrcbs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int, int *)">;
  def vstrch : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<8, unsigned short>, int)">;
  def vstrchs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<8, unsigned short>, int, int *)">;
  def vstrcf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<4, unsigned int>, int)">;
  def vstrcfs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<4, unsigned int>, int, int *)">;
  def vstrczb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vstrczbs : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int, int *)">;
  def vstrczh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<8, unsigned short>, int)">;
  def vstrczhs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<8, unsigned short>, int, int *)">;
````
- **L241 EN**: Declares TableGen def record `vfenezh`.
  **L241 CN**: 声明 TableGen def 记录 `vfenezh`。
- **L242 EN**: Declares TableGen def record `vfenezhs`.
  **L242 CN**: 声明 TableGen def 记录 `vfenezhs`。
- **L243 EN**: Declares TableGen def record `vfenezf`.
  **L243 CN**: 声明 TableGen def 记录 `vfenezf`。
- **L244 EN**: Declares TableGen def record `vfenezfs`.
  **L244 CN**: 声明 TableGen def 记录 `vfenezfs`。
- **L245 EN**: Declares TableGen def record `vistrb`.
  **L245 CN**: 声明 TableGen def 记录 `vistrb`。
- **L246 EN**: Declares TableGen def record `vistrbs`.
  **L246 CN**: 声明 TableGen def 记录 `vistrbs`。
- **L247 EN**: Declares TableGen def record `vistrh`.
  **L247 CN**: 声明 TableGen def 记录 `vistrh`。
- **L248 EN**: Declares TableGen def record `vistrhs`.
  **L248 CN**: 声明 TableGen def 记录 `vistrhs`。
- **L249 EN**: Declares TableGen def record `vistrf`.
  **L249 CN**: 声明 TableGen def 记录 `vistrf`。
- **L250 EN**: Declares TableGen def record `vistrfs`.
  **L250 CN**: 声明 TableGen def 记录 `vistrfs`。
- **L251 EN**: Declares TableGen def record `vstrcb`.
  **L251 CN**: 声明 TableGen def 记录 `vstrcb`。
- **L252 EN**: Declares TableGen def record `vstrcbs`.
  **L252 CN**: 声明 TableGen def 记录 `vstrcbs`。
- **L253 EN**: Declares TableGen def record `vstrch`.
  **L253 CN**: 声明 TableGen def 记录 `vstrch`。
- **L254 EN**: Declares TableGen def record `vstrchs`.
  **L254 CN**: 声明 TableGen def 记录 `vstrchs`。
- **L255 EN**: Declares TableGen def record `vstrcf`.
  **L255 CN**: 声明 TableGen def 记录 `vstrcf`。
- **L256 EN**: Declares TableGen def record `vstrcfs`.
  **L256 CN**: 声明 TableGen def 记录 `vstrcfs`。
- **L257 EN**: Declares TableGen def record `vstrczb`.
  **L257 CN**: 声明 TableGen def 记录 `vstrczb`。
- **L258 EN**: Declares TableGen def record `vstrczbs`.
  **L258 CN**: 声明 TableGen def 记录 `vstrczbs`。
- **L259 EN**: Declares TableGen def record `vstrczh`.
  **L259 CN**: 声明 TableGen def 记录 `vstrczh`。
- **L260 EN**: Declares TableGen def record `vstrczhs`.
  **L260 CN**: 声明 TableGen def 记录 `vstrczhs`。

### Lines 261-280

````tablegen
  def vstrczf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<4, unsigned int>, int)">;
  def vstrczfs : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<4, unsigned int>, int, int *)">;
}

// Vector floating-point instructions (chapter 24 of the PoP)
let Attributes = [NoThrow, Const], Features = "vector" in {
  def vfcedbs : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, double>, _Vector<2, double>, int *)">;
  def vfchdbs : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, double>, _Vector<2, double>, int *)">;
  def vfchedbs : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, double>, _Vector<2, double>, int *)">;
  def vfidb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>, int, int)">;
  def vflndb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>)">;
  def vflpdb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>)">;
  def vfmadb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>, _Vector<2, double>, _Vector<2, double>)">;
  def vfmsdb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>, _Vector<2, double>, _Vector<2, double>)">;
  def vfsqdb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>)">;
  def vftcidb : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, double>, int, int *)">;
}

// Vector-enhancements facility 1 intrinsics.
let Features = "vector-enhancements-1" in {
````
- **L261 EN**: Declares TableGen def record `vstrczf`.
  **L261 CN**: 声明 TableGen def 记录 `vstrczf`。
- **L262 EN**: Declares TableGen def record `vstrczfs`.
  **L262 CN**: 声明 TableGen def 记录 `vstrczfs`。
- **L263 EN**: Closes the current lexical scope or compound statement.
  **L263 CN**: 结束当前词法作用域或复合语句块。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L265 EN**: Comment explains nearby logic, constraints, or intent: `Vector floating-point instructions (chapter 24 of the PoP)`.
  **L265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector floating-point instructions (chapter 24 of the PoP)`。
- **L266 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector" in {`.
  **L266 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector" in {`。
- **L267 EN**: Declares TableGen def record `vfcedbs`.
  **L267 CN**: 声明 TableGen def 记录 `vfcedbs`。
- **L268 EN**: Declares TableGen def record `vfchdbs`.
  **L268 CN**: 声明 TableGen def 记录 `vfchdbs`。
- **L269 EN**: Declares TableGen def record `vfchedbs`.
  **L269 CN**: 声明 TableGen def 记录 `vfchedbs`。
- **L270 EN**: Declares TableGen def record `vfidb`.
  **L270 CN**: 声明 TableGen def 记录 `vfidb`。
- **L271 EN**: Declares TableGen def record `vflndb`.
  **L271 CN**: 声明 TableGen def 记录 `vflndb`。
- **L272 EN**: Declares TableGen def record `vflpdb`.
  **L272 CN**: 声明 TableGen def 记录 `vflpdb`。
- **L273 EN**: Declares TableGen def record `vfmadb`.
  **L273 CN**: 声明 TableGen def 记录 `vfmadb`。
- **L274 EN**: Declares TableGen def record `vfmsdb`.
  **L274 CN**: 声明 TableGen def 记录 `vfmsdb`。
- **L275 EN**: Declares TableGen def record `vfsqdb`.
  **L275 CN**: 声明 TableGen def 记录 `vfsqdb`。
- **L276 EN**: Declares TableGen def record `vftcidb`.
  **L276 CN**: 声明 TableGen def 记录 `vftcidb`。
- **L277 EN**: Closes the current lexical scope or compound statement.
  **L277 CN**: 结束当前词法作用域或复合语句块。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L279 EN**: Comment explains nearby logic, constraints, or intent: `Vector-enhancements facility 1 intrinsics.`.
  **L279 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector-enhancements facility 1 intrinsics.`。
- **L280 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "vector-enhancements-1" in {`.
  **L280 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "vector-enhancements-1" in {`。

### Lines 281-300

````tablegen
  def vlrlr : SystemZTargetBuiltin<"_Vector<16, signed char>(unsigned int, void const *)">;
  def vstrlr : SystemZTargetBuiltin<"void(_Vector<16, signed char>, unsigned int, void *)">;
}
let Attributes = [NoThrow, Const], Features = "vector-enhancements-1" in {
  def vbperm : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<16, unsigned char>, _Vector<16, unsigned char>)">;
  def vmslg : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, __uint128_t, int)">;
  def vfmaxdb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>, _Vector<2, double>, int)">;
  def vfmindb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>, _Vector<2, double>, int)">;
  def vfnmadb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>, _Vector<2, double>, _Vector<2, double>)">;
  def vfnmsdb : SystemZTargetBuiltin<"_Vector<2, double>(_Vector<2, double>, _Vector<2, double>, _Vector<2, double>)">;
  def vfcesbs : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, float>, _Vector<4, float>, int *)">;
  def vfchsbs : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, float>, _Vector<4, float>, int *)">;
  def vfchesbs : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, float>, _Vector<4, float>, int *)">;
  def vfisb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>, int, int)">;
  def vfmaxsb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>, _Vector<4, float>, int)">;
  def vfminsb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>, _Vector<4, float>, int)">;
  def vflnsb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>)">;
  def vflpsb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>)">;
  def vfmasb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>, _Vector<4, float>, _Vector<4, float>)">;
  def vfmssb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>, _Vector<4, float>, _Vector<4, float>)">;
````
- **L281 EN**: Declares TableGen def record `vlrlr`.
  **L281 CN**: 声明 TableGen def 记录 `vlrlr`。
- **L282 EN**: Declares TableGen def record `vstrlr`.
  **L282 CN**: 声明 TableGen def 记录 `vstrlr`。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector-enhancements-1" in {`.
  **L284 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector-enhancements-1" in {`。
- **L285 EN**: Declares TableGen def record `vbperm`.
  **L285 CN**: 声明 TableGen def 记录 `vbperm`。
- **L286 EN**: Declares TableGen def record `vmslg`.
  **L286 CN**: 声明 TableGen def 记录 `vmslg`。
- **L287 EN**: Declares TableGen def record `vfmaxdb`.
  **L287 CN**: 声明 TableGen def 记录 `vfmaxdb`。
- **L288 EN**: Declares TableGen def record `vfmindb`.
  **L288 CN**: 声明 TableGen def 记录 `vfmindb`。
- **L289 EN**: Declares TableGen def record `vfnmadb`.
  **L289 CN**: 声明 TableGen def 记录 `vfnmadb`。
- **L290 EN**: Declares TableGen def record `vfnmsdb`.
  **L290 CN**: 声明 TableGen def 记录 `vfnmsdb`。
- **L291 EN**: Declares TableGen def record `vfcesbs`.
  **L291 CN**: 声明 TableGen def 记录 `vfcesbs`。
- **L292 EN**: Declares TableGen def record `vfchsbs`.
  **L292 CN**: 声明 TableGen def 记录 `vfchsbs`。
- **L293 EN**: Declares TableGen def record `vfchesbs`.
  **L293 CN**: 声明 TableGen def 记录 `vfchesbs`。
- **L294 EN**: Declares TableGen def record `vfisb`.
  **L294 CN**: 声明 TableGen def 记录 `vfisb`。
- **L295 EN**: Declares TableGen def record `vfmaxsb`.
  **L295 CN**: 声明 TableGen def 记录 `vfmaxsb`。
- **L296 EN**: Declares TableGen def record `vfminsb`.
  **L296 CN**: 声明 TableGen def 记录 `vfminsb`。
- **L297 EN**: Declares TableGen def record `vflnsb`.
  **L297 CN**: 声明 TableGen def 记录 `vflnsb`。
- **L298 EN**: Declares TableGen def record `vflpsb`.
  **L298 CN**: 声明 TableGen def 记录 `vflpsb`。
- **L299 EN**: Declares TableGen def record `vfmasb`.
  **L299 CN**: 声明 TableGen def 记录 `vfmasb`。
- **L300 EN**: Declares TableGen def record `vfmssb`.
  **L300 CN**: 声明 TableGen def 记录 `vfmssb`。

### Lines 301-320

````tablegen
  def vfnmasb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>, _Vector<4, float>, _Vector<4, float>)">;
  def vfnmssb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>, _Vector<4, float>, _Vector<4, float>)">;
  def vfsqsb : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<4, float>)">;
  def vftcisb : SystemZTargetBuiltin<"_Vector<4, signed int>(_Vector<4, float>, int, int *)">;
}

// Vector-enhancements facility 2 intrinsics.
let Attributes = [NoThrow, Const], Features = "vector-enhancements-2" in {
  def vsld : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vsrd : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vstrsb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
  def vstrsh : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<16, unsigned char>, int *)">;
  def vstrsf : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<16, unsigned char>, int *)">;
  def vstrszb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int *)">;
  def vstrszh : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<8, unsigned short>, _Vector<8, unsigned short>, _Vector<16, unsigned char>, int *)">;
  def vstrszf : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<4, unsigned int>, _Vector<4, unsigned int>, _Vector<16, unsigned char>, int *)">;
}

// Helpers to implement vec_revb.
let Attributes = [NoThrow, Const], Features = "vector" in {
````
- **L301 EN**: Declares TableGen def record `vfnmasb`.
  **L301 CN**: 声明 TableGen def 记录 `vfnmasb`。
- **L302 EN**: Declares TableGen def record `vfnmssb`.
  **L302 CN**: 声明 TableGen def 记录 `vfnmssb`。
- **L303 EN**: Declares TableGen def record `vfsqsb`.
  **L303 CN**: 声明 TableGen def 记录 `vfsqsb`。
- **L304 EN**: Declares TableGen def record `vftcisb`.
  **L304 CN**: 声明 TableGen def 记录 `vftcisb`。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L307 EN**: Comment explains nearby logic, constraints, or intent: `Vector-enhancements facility 2 intrinsics.`.
  **L307 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector-enhancements facility 2 intrinsics.`。
- **L308 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector-enhancements-2" in {`.
  **L308 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector-enhancements-2" in {`。
- **L309 EN**: Declares TableGen def record `vsld`.
  **L309 CN**: 声明 TableGen def 记录 `vsld`。
- **L310 EN**: Declares TableGen def record `vsrd`.
  **L310 CN**: 声明 TableGen def 记录 `vsrd`。
- **L311 EN**: Declares TableGen def record `vstrsb`.
  **L311 CN**: 声明 TableGen def 记录 `vstrsb`。
- **L312 EN**: Declares TableGen def record `vstrsh`.
  **L312 CN**: 声明 TableGen def 记录 `vstrsh`。
- **L313 EN**: Declares TableGen def record `vstrsf`.
  **L313 CN**: 声明 TableGen def 记录 `vstrsf`。
- **L314 EN**: Declares TableGen def record `vstrszb`.
  **L314 CN**: 声明 TableGen def 记录 `vstrszb`。
- **L315 EN**: Declares TableGen def record `vstrszh`.
  **L315 CN**: 声明 TableGen def 记录 `vstrszh`。
- **L316 EN**: Declares TableGen def record `vstrszf`.
  **L316 CN**: 声明 TableGen def 记录 `vstrszf`。
- **L317 EN**: Closes the current lexical scope or compound statement.
  **L317 CN**: 结束当前词法作用域或复合语句块。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L319 EN**: Comment explains nearby logic, constraints, or intent: `Helpers to implement vec_revb.`.
  **L319 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Helpers to implement vec_revb.`。
- **L320 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector" in {`.
  **L320 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector" in {`。

### Lines 321-340

````tablegen
  def vlbrh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>)">;
  def vlbrf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<4, unsigned int>)">;
  def vlbrg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>)">;
  def vlbrq : SystemZTargetBuiltin<"__uint128_t(__uint128_t)">;
}

// NNP-assist facility intrinsics.
let Attributes = [NoThrow, Const], Features = "nnp-assist" in {
  def vclfnhs : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<8, unsigned short>, int)">;
  def vclfnls : SystemZTargetBuiltin<"_Vector<4, float>(_Vector<8, unsigned short>, int)">;
  def vcrnfs : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<4, float>, _Vector<4, float>, int)">;
  def vcfn : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, int)">;
  def vcnf : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<8, unsigned short>, int)">;
}

// Miscellaneous instruction extensions facility 4 intrinsics.
let Attributes = [NoThrow, Const], Features = "miscellaneous-extensions-4" in {
  def bdepg : SystemZTargetBuiltin<"unsigned long int(unsigned long int, unsigned long int)">;
  def bextg : SystemZTargetBuiltin<"unsigned long int(unsigned long int, unsigned long int)">;
}
````
- **L321 EN**: Declares TableGen def record `vlbrh`.
  **L321 CN**: 声明 TableGen def 记录 `vlbrh`。
- **L322 EN**: Declares TableGen def record `vlbrf`.
  **L322 CN**: 声明 TableGen def 记录 `vlbrf`。
- **L323 EN**: Declares TableGen def record `vlbrg`.
  **L323 CN**: 声明 TableGen def 记录 `vlbrg`。
- **L324 EN**: Declares TableGen def record `vlbrq`.
  **L324 CN**: 声明 TableGen def 记录 `vlbrq`。
- **L325 EN**: Closes the current lexical scope or compound statement.
  **L325 CN**: 结束当前词法作用域或复合语句块。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Comment explains nearby logic, constraints, or intent: `NNP-assist facility intrinsics.`.
  **L327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`NNP-assist facility intrinsics.`。
- **L328 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "nnp-assist" in {`.
  **L328 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "nnp-assist" in {`。
- **L329 EN**: Declares TableGen def record `vclfnhs`.
  **L329 CN**: 声明 TableGen def 记录 `vclfnhs`。
- **L330 EN**: Declares TableGen def record `vclfnls`.
  **L330 CN**: 声明 TableGen def 记录 `vclfnls`。
- **L331 EN**: Declares TableGen def record `vcrnfs`.
  **L331 CN**: 声明 TableGen def 记录 `vcrnfs`。
- **L332 EN**: Declares TableGen def record `vcfn`.
  **L332 CN**: 声明 TableGen def 记录 `vcfn`。
- **L333 EN**: Declares TableGen def record `vcnf`.
  **L333 CN**: 声明 TableGen def 记录 `vcnf`。
- **L334 EN**: Closes the current lexical scope or compound statement.
  **L334 CN**: 结束当前词法作用域或复合语句块。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L336 EN**: Comment explains nearby logic, constraints, or intent: `Miscellaneous instruction extensions facility 4 intrinsics.`.
  **L336 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Miscellaneous instruction extensions facility 4 intrinsics.`。
- **L337 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "miscellaneous-extensions-4" in {`.
  **L337 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "miscellaneous-extensions-4" in {`。
- **L338 EN**: Declares TableGen def record `bdepg`.
  **L338 CN**: 声明 TableGen def 记录 `bdepg`。
- **L339 EN**: Declares TableGen def record `bextg`.
  **L339 CN**: 声明 TableGen def 记录 `bextg`。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。

### Lines 341-360

````tablegen

// Vector-enhancements facility 3 intrinsics.
let Attributes = [NoThrow, Const], Features = "vector-enhancements-3" in {
  def vgemb : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<8, unsigned short>)">;
  def vgemh : SystemZTargetBuiltin<"_Vector<8, unsigned short>(_Vector<16, unsigned char>)">;
  def vgemf : SystemZTargetBuiltin<"_Vector<4, unsigned int>(_Vector<16, unsigned char>)">;
  def vgemg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<16, unsigned char>)">;
  def vgemq : SystemZTargetBuiltin<"__uint128_t(_Vector<16, unsigned char>)">;
  def vuplg : SystemZTargetBuiltin<"__int128_t(_Vector<2, signed long long int>)">;
  def vupllg : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>)">;
  def vuphg : SystemZTargetBuiltin<"__int128_t(_Vector<2, signed long long int>)">;
  def vuplhg : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>)">;
  def vavgq : SystemZTargetBuiltin<"__int128_t(__int128_t, __int128_t)">;
  def vavglq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t)">;
  def veval : SystemZTargetBuiltin<"_Vector<16, unsigned char>(_Vector<16, unsigned char>, _Vector<16, unsigned char>, _Vector<16, unsigned char>, int)">;
  def vmahg : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, signed long long int>, _Vector<2, signed long long int>, _Vector<2, signed long long int>)">;
  def vmahq : SystemZTargetBuiltin<"__int128_t(__int128_t, __int128_t, __int128_t)">;
  def vmalhg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vmalhq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t, __uint128_t)">;
  def vmaeg : SystemZTargetBuiltin<"__int128_t(_Vector<2, signed long long int>, _Vector<2, signed long long int>, __int128_t)">;
````
- **L341 EN**: Blank line separating nearby declarations or logic blocks.
  **L341 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L342 EN**: Comment explains nearby logic, constraints, or intent: `Vector-enhancements facility 3 intrinsics.`.
  **L342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector-enhancements facility 3 intrinsics.`。
- **L343 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "vector-enhancements-3" in {`.
  **L343 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "vector-enhancements-3" in {`。
- **L344 EN**: Declares TableGen def record `vgemb`.
  **L344 CN**: 声明 TableGen def 记录 `vgemb`。
- **L345 EN**: Declares TableGen def record `vgemh`.
  **L345 CN**: 声明 TableGen def 记录 `vgemh`。
- **L346 EN**: Declares TableGen def record `vgemf`.
  **L346 CN**: 声明 TableGen def 记录 `vgemf`。
- **L347 EN**: Declares TableGen def record `vgemg`.
  **L347 CN**: 声明 TableGen def 记录 `vgemg`。
- **L348 EN**: Declares TableGen def record `vgemq`.
  **L348 CN**: 声明 TableGen def 记录 `vgemq`。
- **L349 EN**: Declares TableGen def record `vuplg`.
  **L349 CN**: 声明 TableGen def 记录 `vuplg`。
- **L350 EN**: Declares TableGen def record `vupllg`.
  **L350 CN**: 声明 TableGen def 记录 `vupllg`。
- **L351 EN**: Declares TableGen def record `vuphg`.
  **L351 CN**: 声明 TableGen def 记录 `vuphg`。
- **L352 EN**: Declares TableGen def record `vuplhg`.
  **L352 CN**: 声明 TableGen def 记录 `vuplhg`。
- **L353 EN**: Declares TableGen def record `vavgq`.
  **L353 CN**: 声明 TableGen def 记录 `vavgq`。
- **L354 EN**: Declares TableGen def record `vavglq`.
  **L354 CN**: 声明 TableGen def 记录 `vavglq`。
- **L355 EN**: Declares TableGen def record `veval`.
  **L355 CN**: 声明 TableGen def 记录 `veval`。
- **L356 EN**: Declares TableGen def record `vmahg`.
  **L356 CN**: 声明 TableGen def 记录 `vmahg`。
- **L357 EN**: Declares TableGen def record `vmahq`.
  **L357 CN**: 声明 TableGen def 记录 `vmahq`。
- **L358 EN**: Declares TableGen def record `vmalhg`.
  **L358 CN**: 声明 TableGen def 记录 `vmalhg`。
- **L359 EN**: Declares TableGen def record `vmalhq`.
  **L359 CN**: 声明 TableGen def 记录 `vmalhq`。
- **L360 EN**: Declares TableGen def record `vmaeg`.
  **L360 CN**: 声明 TableGen def 记录 `vmaeg`。

### Lines 361-377

````tablegen
  def vmaleg : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, __uint128_t)">;
  def vmaog : SystemZTargetBuiltin<"__int128_t(_Vector<2, signed long long int>, _Vector<2, signed long long int>, __int128_t)">;
  def vmalog : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>, __uint128_t)">;
  def vmhg : SystemZTargetBuiltin<"_Vector<2, signed long long int>(_Vector<2, signed long long int>, _Vector<2, signed long long int>)">;
  def vmhq : SystemZTargetBuiltin<"__int128_t(__int128_t, __int128_t)">;
  def vmlhg : SystemZTargetBuiltin<"_Vector<2, unsigned long long int>(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vmlhq : SystemZTargetBuiltin<"__uint128_t(__uint128_t, __uint128_t)">;
  def vmeg : SystemZTargetBuiltin<"__int128_t(_Vector<2, signed long long int>, _Vector<2, signed long long int>)">;
  def vmleg : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vmog : SystemZTargetBuiltin<"__int128_t(_Vector<2, signed long long int>, _Vector<2, signed long long int>)">;
  def vmlog : SystemZTargetBuiltin<"__uint128_t(_Vector<2, unsigned long long int>, _Vector<2, unsigned long long int>)">;
  def vceqqs : SystemZTargetBuiltin<"__int128_t(__uint128_t, __uint128_t, int *)">;
  def vchqs : SystemZTargetBuiltin<"__int128_t(__int128_t, __int128_t, int *)">;
  def vchlqs : SystemZTargetBuiltin<"__int128_t(__uint128_t, __uint128_t, int *)">;
  def vclzq : SystemZTargetBuiltin<"__uint128_t(__uint128_t)">;
  def vctzq : SystemZTargetBuiltin<"__uint128_t(__uint128_t)">;
}
````
- **L361 EN**: Declares TableGen def record `vmaleg`.
  **L361 CN**: 声明 TableGen def 记录 `vmaleg`。
- **L362 EN**: Declares TableGen def record `vmaog`.
  **L362 CN**: 声明 TableGen def 记录 `vmaog`。
- **L363 EN**: Declares TableGen def record `vmalog`.
  **L363 CN**: 声明 TableGen def 记录 `vmalog`。
- **L364 EN**: Declares TableGen def record `vmhg`.
  **L364 CN**: 声明 TableGen def 记录 `vmhg`。
- **L365 EN**: Declares TableGen def record `vmhq`.
  **L365 CN**: 声明 TableGen def 记录 `vmhq`。
- **L366 EN**: Declares TableGen def record `vmlhg`.
  **L366 CN**: 声明 TableGen def 记录 `vmlhg`。
- **L367 EN**: Declares TableGen def record `vmlhq`.
  **L367 CN**: 声明 TableGen def 记录 `vmlhq`。
- **L368 EN**: Declares TableGen def record `vmeg`.
  **L368 CN**: 声明 TableGen def 记录 `vmeg`。
- **L369 EN**: Declares TableGen def record `vmleg`.
  **L369 CN**: 声明 TableGen def 记录 `vmleg`。
- **L370 EN**: Declares TableGen def record `vmog`.
  **L370 CN**: 声明 TableGen def 记录 `vmog`。
- **L371 EN**: Declares TableGen def record `vmlog`.
  **L371 CN**: 声明 TableGen def 记录 `vmlog`。
- **L372 EN**: Declares TableGen def record `vceqqs`.
  **L372 CN**: 声明 TableGen def 记录 `vceqqs`。
- **L373 EN**: Declares TableGen def record `vchqs`.
  **L373 CN**: 声明 TableGen def 记录 `vchqs`。
- **L374 EN**: Declares TableGen def record `vchlqs`.
  **L374 CN**: 声明 TableGen def 记录 `vchlqs`。
- **L375 EN**: Declares TableGen def record `vclzq`.
  **L375 CN**: 声明 TableGen def 记录 `vclzq`。
- **L376 EN**: Declares TableGen def record `vctzq`.
  **L376 CN**: 声明 TableGen def 记录 `vctzq`。
- **L377 EN**: Closes the current lexical scope or compound statement.
  **L377 CN**: 结束当前词法作用域或复合语句块。

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
- **Attribute metadata / 属性元数据**
  - **EN**: Captures attribute spellings, subjects, semantic flags, and generated helper data.
  - **CN**: 刻画属性的拼写、适用对象、语义标志与生成的辅助数据。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `SystemZTargetBuiltin`, `SystemZNoPrefixTargetBuiltin`
- **Functions or callables / 函数或可调用对象**: `int`, `void`, `instructions`, `char>`, `int>`, `short>`, `__int128_t`, `__uint128_t`, `double>`, `float>`
- **TableGen records / TableGen 记录**: `SystemZPrefix`, `SystemZTargetBuiltin`, `SystemZNoPrefixTargetBuiltin`, `__builtin_tbegin`, `__builtin_tbegin_nofloat`, `__builtin_tbeginc`, `__builtin_tabort`, `__builtin_tend`, `__builtin_tx_nesting_depth`, `__builtin_tx_assist`, `__builtin_non_tx_store`, `lcbb`, `vlbb`, `vll`, `vstl`, `vperm`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
