# BuiltinsX86_64.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsX86_64.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: X86-64 Builtin function database *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsX86_64` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 399

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````tablegen
//===--- BuiltinsX86_64.td - X86-64 Builtin function database ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the X86-64-specific builtin function database.
//
//===----------------------------------------------------------------------===//

include "clang/Basic/BuiltinsX86Base.td"

let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {
  def _BitScanForward64 : X86LibBuiltin<"unsigned char(msuint32_t *, unsigned long long int)">;
  def _BitScanReverse64 : X86LibBuiltin<"unsigned char(msuint32_t *, unsigned long long int)">;
}

let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, Const, RequireDeclaration] in {
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the X86-64-specific builtin function database.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the X86-64-specific builtin function database.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Includes TableGen file `"clang/Basic/BuiltinsX86Base.td"` so later records can reuse shared definitions.
  **L13 CN**: 引入 TableGen 文件 `"clang/Basic/BuiltinsX86Base.td"`，以便后续记录复用共享定义。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {`.
  **L15 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {`。
- **L16 EN**: Declares TableGen def record `_BitScanForward64`.
  **L16 CN**: 声明 TableGen def 记录 `_BitScanForward64`。
- **L17 EN**: Declares TableGen def record `_BitScanReverse64`.
  **L17 CN**: 声明 TableGen def 记录 `_BitScanReverse64`。
- **L18 EN**: Closes the current lexical scope or compound statement.
  **L18 CN**: 结束当前词法作用域或复合语句块。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L20 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, Const, RequireDeclaration] in {`.
  **L20 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, Const, RequireDeclaration] in {`。

### Lines 21-40

````tablegen
  def __mulh : X86LibBuiltin<"long long int(long long int, long long int)">;
  def __umulh : X86LibBuiltin<"unsigned long long int(unsigned long long int, unsigned long long int)">;
  def _mul128 : X86LibBuiltin<"long long int(long long int, long long int, long long int *)">;
  def _umul128 : X86LibBuiltin<"unsigned long long int(unsigned long long int, unsigned long long int, unsigned long long int *)">;
}

let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {
  def __faststorefence : X86LibBuiltin<"void()">;
}

let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, Const, RequireDeclaration] in {
  def __shiftleft128 : X86LibBuiltin<"unsigned long long int(unsigned long long int, unsigned long long int, unsigned char)">;
  def __shiftright128 : X86LibBuiltin<"unsigned long long int(unsigned long long int, unsigned long long int, unsigned char)">;
}

let Features = "cx16", Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {
  def _InterlockedCompareExchange128 : X86LibBuiltin<"unsigned char(long long int volatile *, long long int, long long int, long long int *)">;
}

let Attributes = [NoThrow] in {
````
- **L21 EN**: Declares TableGen def record `__mulh`.
  **L21 CN**: 声明 TableGen def 记录 `__mulh`。
- **L22 EN**: Declares TableGen def record `__umulh`.
  **L22 CN**: 声明 TableGen def 记录 `__umulh`。
- **L23 EN**: Declares TableGen def record `_mul128`.
  **L23 CN**: 声明 TableGen def 记录 `_mul128`。
- **L24 EN**: Declares TableGen def record `_umul128`.
  **L24 CN**: 声明 TableGen def 记录 `_umul128`。
- **L25 EN**: Closes the current lexical scope or compound statement.
  **L25 CN**: 结束当前词法作用域或复合语句块。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {`.
  **L27 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {`。
- **L28 EN**: Declares TableGen def record `__faststorefence`.
  **L28 CN**: 声明 TableGen def 记录 `__faststorefence`。
- **L29 EN**: Closes the current lexical scope or compound statement.
  **L29 CN**: 结束当前词法作用域或复合语句块。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, Const, RequireDeclaration] in {`.
  **L31 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, Const, RequireDeclaration] in {`。
- **L32 EN**: Declares TableGen def record `__shiftleft128`.
  **L32 CN**: 声明 TableGen def 记录 `__shiftleft128`。
- **L33 EN**: Declares TableGen def record `__shiftright128`.
  **L33 CN**: 声明 TableGen def 记录 `__shiftright128`。
- **L34 EN**: Closes the current lexical scope or compound statement.
  **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L36 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "cx16", Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {`.
  **L36 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "cx16", Header = "intrin.h", Languages = "ALL_MS_LANGUAGES", Attributes = [NoThrow, RequireDeclaration] in {`。
- **L37 EN**: Declares TableGen def record `_InterlockedCompareExchange128`.
  **L37 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange128`。
- **L38 EN**: Closes the current lexical scope or compound statement.
  **L38 CN**: 结束当前词法作用域或复合语句块。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L40 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L40 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。

### Lines 41-60

````tablegen
  def readeflags_u64 : X86Builtin<"unsigned long long int()">;
  def writeeflags_u64 : X86Builtin<"void(unsigned long long int)">;
}

let Features = "sse", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {
  def cvtss2si64 : X86Builtin<"long long int(_Vector<4, float>)">;
  def cvttss2si64 : X86Builtin<"long long int(_Vector<4, float>)">;
}

let Features = "sse2", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {
  def cvtsd2si64 : X86Builtin<"long long int(_Vector<2, double>)">;
  def cvttsd2si64 : X86Builtin<"long long int(_Vector<2, double>)">;
}

let Features = "sse2", Attributes = [NoThrow] in {
  def movnti64 : X86Builtin<"void(long long int *, long long int)">;
}

let Features = "sse4.1", Attributes = [NoThrow, Const, Constexpr, RequiredVectorWidth<128>] in {
  def vec_set_v2di : X86Builtin<"_Vector<2, long long int>(_Vector<2, long long int>, long long int, _Constant int)">;
````
- **L41 EN**: Declares TableGen def record `readeflags_u64`.
  **L41 CN**: 声明 TableGen def 记录 `readeflags_u64`。
- **L42 EN**: Declares TableGen def record `writeeflags_u64`.
  **L42 CN**: 声明 TableGen def 记录 `writeeflags_u64`。
- **L43 EN**: Closes the current lexical scope or compound statement.
  **L43 CN**: 结束当前词法作用域或复合语句块。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "sse", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`.
  **L45 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "sse", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`。
- **L46 EN**: Declares TableGen def record `cvtss2si64`.
  **L46 CN**: 声明 TableGen def 记录 `cvtss2si64`。
- **L47 EN**: Declares TableGen def record `cvttss2si64`.
  **L47 CN**: 声明 TableGen def 记录 `cvttss2si64`。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L50 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "sse2", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`.
  **L50 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "sse2", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`。
- **L51 EN**: Declares TableGen def record `cvtsd2si64`.
  **L51 CN**: 声明 TableGen def 记录 `cvtsd2si64`。
- **L52 EN**: Declares TableGen def record `cvttsd2si64`.
  **L52 CN**: 声明 TableGen def 记录 `cvttsd2si64`。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L55 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "sse2", Attributes = [NoThrow] in {`.
  **L55 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "sse2", Attributes = [NoThrow] in {`。
- **L56 EN**: Declares TableGen def record `movnti64`.
  **L56 CN**: 声明 TableGen def 记录 `movnti64`。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L59 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "sse4.1", Attributes = [NoThrow, Const, Constexpr, RequiredVectorWidth<128>] in {`.
  **L59 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "sse4.1", Attributes = [NoThrow, Const, Constexpr, RequiredVectorWidth<128>] in {`。
- **L60 EN**: Declares TableGen def record `vec_set_v2di`.
  **L60 CN**: 声明 TableGen def 记录 `vec_set_v2di`。

### Lines 61-80

````tablegen
}

let Features = "crc32", Attributes = [NoThrow, Const, Constexpr] in {
  def crc32di : X86Builtin<"unsigned long long int(unsigned long long int, unsigned long long int)">;
}

let Features = "avx", Attributes = [NoThrow, Const, Constexpr, RequiredVectorWidth<256>] in {
  def vec_ext_v4di : X86Builtin<"long long int(_Vector<4, long long int>, _Constant int)">;
  def vec_set_v4di : X86Builtin<"_Vector<4, long long int>(_Vector<4, long long int>, long long int, _Constant int)">;
}

let Features = "fsgsbase", Attributes = [NoThrow] in {
  def rdfsbase32 : X86Builtin<"unsigned int()">;
  def rdfsbase64 : X86Builtin<"unsigned long long int()">;
  def rdgsbase32 : X86Builtin<"unsigned int()">;
  def rdgsbase64 : X86Builtin<"unsigned long long int()">;
  def wrfsbase32 : X86Builtin<"void(unsigned int)">;
  def wrfsbase64 : X86Builtin<"void(unsigned long long int)">;
  def wrgsbase32 : X86Builtin<"void(unsigned int)">;
  def wrgsbase64 : X86Builtin<"void(unsigned long long int)">;
````
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "crc32", Attributes = [NoThrow, Const, Constexpr] in {`.
  **L63 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "crc32", Attributes = [NoThrow, Const, Constexpr] in {`。
- **L64 EN**: Declares TableGen def record `crc32di`.
  **L64 CN**: 声明 TableGen def 记录 `crc32di`。
- **L65 EN**: Closes the current lexical scope or compound statement.
  **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "avx", Attributes = [NoThrow, Const, Constexpr, RequiredVectorWidth<256>] in {`.
  **L67 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "avx", Attributes = [NoThrow, Const, Constexpr, RequiredVectorWidth<256>] in {`。
- **L68 EN**: Declares TableGen def record `vec_ext_v4di`.
  **L68 CN**: 声明 TableGen def 记录 `vec_ext_v4di`。
- **L69 EN**: Declares TableGen def record `vec_set_v4di`.
  **L69 CN**: 声明 TableGen def 记录 `vec_set_v4di`。
- **L70 EN**: Closes the current lexical scope or compound statement.
  **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "fsgsbase", Attributes = [NoThrow] in {`.
  **L72 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "fsgsbase", Attributes = [NoThrow] in {`。
- **L73 EN**: Declares TableGen def record `rdfsbase32`.
  **L73 CN**: 声明 TableGen def 记录 `rdfsbase32`。
- **L74 EN**: Declares TableGen def record `rdfsbase64`.
  **L74 CN**: 声明 TableGen def 记录 `rdfsbase64`。
- **L75 EN**: Declares TableGen def record `rdgsbase32`.
  **L75 CN**: 声明 TableGen def 记录 `rdgsbase32`。
- **L76 EN**: Declares TableGen def record `rdgsbase64`.
  **L76 CN**: 声明 TableGen def 记录 `rdgsbase64`。
- **L77 EN**: Declares TableGen def record `wrfsbase32`.
  **L77 CN**: 声明 TableGen def 记录 `wrfsbase32`。
- **L78 EN**: Declares TableGen def record `wrfsbase64`.
  **L78 CN**: 声明 TableGen def 记录 `wrfsbase64`。
- **L79 EN**: Declares TableGen def record `wrgsbase32`.
  **L79 CN**: 声明 TableGen def 记录 `wrgsbase32`。
- **L80 EN**: Declares TableGen def record `wrgsbase64`.
  **L80 CN**: 声明 TableGen def 记录 `wrgsbase64`。

### Lines 81-100

````tablegen
}

let Features = "fxsr", Attributes = [NoThrow] in {
  def fxrstor64 : X86Builtin<"void(void *)">;
  def fxsave64 : X86Builtin<"void(void *)">;
}

let Features = "xsave", Attributes = [NoThrow] in {
  def xsave64 : X86Builtin<"void(void *, unsigned long long int)">;
  def xrstor64 : X86Builtin<"void(void *, unsigned long long int)">;
}

let Features = "xsaveopt", Attributes = [NoThrow] in {
  def xsaveopt64 : X86Builtin<"void(void *, unsigned long long int)">;
}

let Features = "xsaves", Attributes = [NoThrow] in {
  def xrstors64 : X86Builtin<"void(void *, unsigned long long int)">;
}

````
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L83 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "fxsr", Attributes = [NoThrow] in {`.
  **L83 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "fxsr", Attributes = [NoThrow] in {`。
- **L84 EN**: Declares TableGen def record `fxrstor64`.
  **L84 CN**: 声明 TableGen def 记录 `fxrstor64`。
- **L85 EN**: Declares TableGen def record `fxsave64`.
  **L85 CN**: 声明 TableGen def 记录 `fxsave64`。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L88 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "xsave", Attributes = [NoThrow] in {`.
  **L88 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "xsave", Attributes = [NoThrow] in {`。
- **L89 EN**: Declares TableGen def record `xsave64`.
  **L89 CN**: 声明 TableGen def 记录 `xsave64`。
- **L90 EN**: Declares TableGen def record `xrstor64`.
  **L90 CN**: 声明 TableGen def 记录 `xrstor64`。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L93 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "xsaveopt", Attributes = [NoThrow] in {`.
  **L93 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "xsaveopt", Attributes = [NoThrow] in {`。
- **L94 EN**: Declares TableGen def record `xsaveopt64`.
  **L94 CN**: 声明 TableGen def 记录 `xsaveopt64`。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L97 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "xsaves", Attributes = [NoThrow] in {`.
  **L97 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "xsaves", Attributes = [NoThrow] in {`。
- **L98 EN**: Declares TableGen def record `xrstors64`.
  **L98 CN**: 声明 TableGen def 记录 `xrstors64`。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 101-120

````tablegen
let Features = "xsavec", Attributes = [NoThrow] in {
  def xsavec64 : X86Builtin<"void(void *, unsigned long long int)">;
}

let Features = "xsaves", Attributes = [NoThrow] in {
  def xsaves64 : X86Builtin<"void(void *, unsigned long long int)">;
}

let Features = "shstk", Attributes = [NoThrow] in {
  def incsspq : X86Builtin<"void(unsigned long long int)">;
  def rdsspq : X86Builtin<"unsigned long long int(unsigned long long int)">;
  def wrssq : X86Builtin<"void(unsigned long long int, void *)">;
  def wrussq : X86Builtin<"void(unsigned long long int, void *)">;
}

let Attributes = [NoThrow, Constexpr] in {
  def addcarryx_u64 : X86Builtin<"unsigned char(unsigned char, unsigned long long int, unsigned long long int, unsigned long long int *)">;
  def subborrow_u64 : X86Builtin<"unsigned char(unsigned char, unsigned long long int, unsigned long long int, unsigned long long int *)">;
}

````
- **L101 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "xsavec", Attributes = [NoThrow] in {`.
  **L101 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "xsavec", Attributes = [NoThrow] in {`。
- **L102 EN**: Declares TableGen def record `xsavec64`.
  **L102 CN**: 声明 TableGen def 记录 `xsavec64`。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "xsaves", Attributes = [NoThrow] in {`.
  **L105 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "xsaves", Attributes = [NoThrow] in {`。
- **L106 EN**: Declares TableGen def record `xsaves64`.
  **L106 CN**: 声明 TableGen def 记录 `xsaves64`。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L109 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "shstk", Attributes = [NoThrow] in {`.
  **L109 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "shstk", Attributes = [NoThrow] in {`。
- **L110 EN**: Declares TableGen def record `incsspq`.
  **L110 CN**: 声明 TableGen def 记录 `incsspq`。
- **L111 EN**: Declares TableGen def record `rdsspq`.
  **L111 CN**: 声明 TableGen def 记录 `rdsspq`。
- **L112 EN**: Declares TableGen def record `wrssq`.
  **L112 CN**: 声明 TableGen def 记录 `wrssq`。
- **L113 EN**: Declares TableGen def record `wrussq`.
  **L113 CN**: 声明 TableGen def 记录 `wrussq`。
- **L114 EN**: Closes the current lexical scope or compound statement.
  **L114 CN**: 结束当前词法作用域或复合语句块。
- **L115 EN**: Blank line separating nearby declarations or logic blocks.
  **L115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L116 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Constexpr] in {`.
  **L116 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Constexpr] in {`。
- **L117 EN**: Declares TableGen def record `addcarryx_u64`.
  **L117 CN**: 声明 TableGen def 记录 `addcarryx_u64`。
- **L118 EN**: Declares TableGen def record `subborrow_u64`.
  **L118 CN**: 声明 TableGen def 记录 `subborrow_u64`。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 121-140

````tablegen
let Features = "rdrnd", Attributes = [NoThrow] in {
  def rdrand64_step : X86Builtin<"unsigned int(unsigned long long int *)">;
}

let Features = "rdseed", Attributes = [NoThrow] in {
  def rdseed64_step : X86Builtin<"unsigned int(unsigned long long int *)">;
}

let Features = "bmi", Attributes = [NoThrow, Const, Constexpr] in {
  def bextr_u64 : X86Builtin<"unsigned long long int(unsigned long long int, unsigned long long int)">;
}

let Attributes = [NoThrow, Const, Constexpr] in {
  def tzcnt_u64 : X86Builtin<"unsigned long long int(unsigned long long int)">;
  def lzcnt_u64 : X86Builtin<"unsigned long long int(unsigned long long int)">;
}

let Features = "bmi2", Attributes = [NoThrow, Const, Constexpr] in {
  def bzhi_di : X86Builtin<"unsigned long long int(unsigned long long int, unsigned long long int)">;
  def pdep_di : X86Builtin<"unsigned long long int(unsigned long long int, unsigned long long int)">;
````
- **L121 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "rdrnd", Attributes = [NoThrow] in {`.
  **L121 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "rdrnd", Attributes = [NoThrow] in {`。
- **L122 EN**: Declares TableGen def record `rdrand64_step`.
  **L122 CN**: 声明 TableGen def 记录 `rdrand64_step`。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L125 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "rdseed", Attributes = [NoThrow] in {`.
  **L125 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "rdseed", Attributes = [NoThrow] in {`。
- **L126 EN**: Declares TableGen def record `rdseed64_step`.
  **L126 CN**: 声明 TableGen def 记录 `rdseed64_step`。
- **L127 EN**: Closes the current lexical scope or compound statement.
  **L127 CN**: 结束当前词法作用域或复合语句块。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L129 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "bmi", Attributes = [NoThrow, Const, Constexpr] in {`.
  **L129 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "bmi", Attributes = [NoThrow, Const, Constexpr] in {`。
- **L130 EN**: Declares TableGen def record `bextr_u64`.
  **L130 CN**: 声明 TableGen def 记录 `bextr_u64`。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L133 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const, Constexpr] in {`.
  **L133 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const, Constexpr] in {`。
- **L134 EN**: Declares TableGen def record `tzcnt_u64`.
  **L134 CN**: 声明 TableGen def 记录 `tzcnt_u64`。
- **L135 EN**: Declares TableGen def record `lzcnt_u64`.
  **L135 CN**: 声明 TableGen def 记录 `lzcnt_u64`。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "bmi2", Attributes = [NoThrow, Const, Constexpr] in {`.
  **L138 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "bmi2", Attributes = [NoThrow, Const, Constexpr] in {`。
- **L139 EN**: Declares TableGen def record `bzhi_di`.
  **L139 CN**: 声明 TableGen def 记录 `bzhi_di`。
- **L140 EN**: Declares TableGen def record `pdep_di`.
  **L140 CN**: 声明 TableGen def 记录 `pdep_di`。

### Lines 141-160

````tablegen
  def pext_di : X86Builtin<"unsigned long long int(unsigned long long int, unsigned long long int)">;
}

let Features = "tbm", Attributes = [NoThrow, Const, Constexpr] in {
  def bextri_u64 : X86Builtin<"unsigned long long int(unsigned long long int, _Constant unsigned long long int)">;
}

let Features = "lwp", Attributes = [NoThrow] in {
  def lwpins64 : X86Builtin<"unsigned char(unsigned long long int, unsigned int, _Constant unsigned int)">;
  def lwpval64 : X86Builtin<"void(unsigned long long int, unsigned int, _Constant unsigned int)">;
}

let Features = "avx512f", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {
  def vcvtsd2si64 : X86Builtin<"long long int(_Vector<2, double>, _Constant int)">;
  def vcvtsd2usi64 : X86Builtin<"unsigned long long int(_Vector<2, double>, _Constant int)">;
  def vcvtss2si64 : X86Builtin<"long long int(_Vector<4, float>, _Constant int)">;
  def vcvtss2usi64 : X86Builtin<"unsigned long long int(_Vector<4, float>, _Constant int)">;
  def vcvttsd2si64 : X86Builtin<"long long int(_Vector<2, double>, _Constant int)">;
  def vcvttsd2usi64 : X86Builtin<"unsigned long long int(_Vector<2, double>, _Constant int)">;
  def vcvttss2si64 : X86Builtin<"long long int(_Vector<4, float>, _Constant int)">;
````
- **L141 EN**: Declares TableGen def record `pext_di`.
  **L141 CN**: 声明 TableGen def 记录 `pext_di`。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L144 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "tbm", Attributes = [NoThrow, Const, Constexpr] in {`.
  **L144 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "tbm", Attributes = [NoThrow, Const, Constexpr] in {`。
- **L145 EN**: Declares TableGen def record `bextri_u64`.
  **L145 CN**: 声明 TableGen def 记录 `bextri_u64`。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "lwp", Attributes = [NoThrow] in {`.
  **L148 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "lwp", Attributes = [NoThrow] in {`。
- **L149 EN**: Declares TableGen def record `lwpins64`.
  **L149 CN**: 声明 TableGen def 记录 `lwpins64`。
- **L150 EN**: Declares TableGen def record `lwpval64`.
  **L150 CN**: 声明 TableGen def 记录 `lwpval64`。
- **L151 EN**: Closes the current lexical scope or compound statement.
  **L151 CN**: 结束当前词法作用域或复合语句块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "avx512f", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`.
  **L153 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "avx512f", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`。
- **L154 EN**: Declares TableGen def record `vcvtsd2si64`.
  **L154 CN**: 声明 TableGen def 记录 `vcvtsd2si64`。
- **L155 EN**: Declares TableGen def record `vcvtsd2usi64`.
  **L155 CN**: 声明 TableGen def 记录 `vcvtsd2usi64`。
- **L156 EN**: Declares TableGen def record `vcvtss2si64`.
  **L156 CN**: 声明 TableGen def 记录 `vcvtss2si64`。
- **L157 EN**: Declares TableGen def record `vcvtss2usi64`.
  **L157 CN**: 声明 TableGen def 记录 `vcvtss2usi64`。
- **L158 EN**: Declares TableGen def record `vcvttsd2si64`.
  **L158 CN**: 声明 TableGen def 记录 `vcvttsd2si64`。
- **L159 EN**: Declares TableGen def record `vcvttsd2usi64`.
  **L159 CN**: 声明 TableGen def 记录 `vcvttsd2usi64`。
- **L160 EN**: Declares TableGen def record `vcvttss2si64`.
  **L160 CN**: 声明 TableGen def 记录 `vcvttss2si64`。

### Lines 161-180

````tablegen
  def vcvttss2usi64 : X86Builtin<"unsigned long long int(_Vector<4, float>, _Constant int)">;
  def cvtsi2sd64 : X86Builtin<"_Vector<2, double>(_Vector<2, double>, long long int, _Constant int)">;
  def cvtsi2ss64 : X86Builtin<"_Vector<4, float>(_Vector<4, float>, long long int, _Constant int)">;
  def cvtusi2sd64 : X86Builtin<"_Vector<2, double>(_Vector<2, double>, unsigned long long int, _Constant int)">;
  def cvtusi2ss64 : X86Builtin<"_Vector<4, float>(_Vector<4, float>, unsigned long long int, _Constant int)">;
}

let Features = "avx512fp16", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {
  def vcvtsh2si64 : X86Builtin<"long long int(_Vector<8, _Float16>, _Constant int)">;
  def vcvtsh2usi64 : X86Builtin<"unsigned long long int(_Vector<8, _Float16>, _Constant int)">;
  def vcvtusi642sh : X86Builtin<"_Vector<8, _Float16>(_Vector<8, _Float16>, unsigned long long int, _Constant int)">;
  def vcvtsi642sh : X86Builtin<"_Vector<8, _Float16>(_Vector<8, _Float16>, long long int, _Constant int)">;
  def vcvttsh2si64 : X86Builtin<"long long int(_Vector<8, _Float16>, _Constant int)">;
  def vcvttsh2usi64 : X86Builtin<"unsigned long long int(_Vector<8, _Float16>, _Constant int)">;
}

let Features = "movdiri", Attributes = [NoThrow] in {
  def directstore_u64 : X86Builtin<"void(unsigned long int *, unsigned long int)">;
}

````
- **L161 EN**: Declares TableGen def record `vcvttss2usi64`.
  **L161 CN**: 声明 TableGen def 记录 `vcvttss2usi64`。
- **L162 EN**: Declares TableGen def record `cvtsi2sd64`.
  **L162 CN**: 声明 TableGen def 记录 `cvtsi2sd64`。
- **L163 EN**: Declares TableGen def record `cvtsi2ss64`.
  **L163 CN**: 声明 TableGen def 记录 `cvtsi2ss64`。
- **L164 EN**: Declares TableGen def record `cvtusi2sd64`.
  **L164 CN**: 声明 TableGen def 记录 `cvtusi2sd64`。
- **L165 EN**: Declares TableGen def record `cvtusi2ss64`.
  **L165 CN**: 声明 TableGen def 记录 `cvtusi2ss64`。
- **L166 EN**: Closes the current lexical scope or compound statement.
  **L166 CN**: 结束当前词法作用域或复合语句块。
- **L167 EN**: Blank line separating nearby declarations or logic blocks.
  **L167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L168 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "avx512fp16", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`.
  **L168 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "avx512fp16", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`。
- **L169 EN**: Declares TableGen def record `vcvtsh2si64`.
  **L169 CN**: 声明 TableGen def 记录 `vcvtsh2si64`。
- **L170 EN**: Declares TableGen def record `vcvtsh2usi64`.
  **L170 CN**: 声明 TableGen def 记录 `vcvtsh2usi64`。
- **L171 EN**: Declares TableGen def record `vcvtusi642sh`.
  **L171 CN**: 声明 TableGen def 记录 `vcvtusi642sh`。
- **L172 EN**: Declares TableGen def record `vcvtsi642sh`.
  **L172 CN**: 声明 TableGen def 记录 `vcvtsi642sh`。
- **L173 EN**: Declares TableGen def record `vcvttsh2si64`.
  **L173 CN**: 声明 TableGen def 记录 `vcvttsh2si64`。
- **L174 EN**: Declares TableGen def record `vcvttsh2usi64`.
  **L174 CN**: 声明 TableGen def 记录 `vcvttsh2usi64`。
- **L175 EN**: Closes the current lexical scope or compound statement.
  **L175 CN**: 结束当前词法作用域或复合语句块。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L177 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movdiri", Attributes = [NoThrow] in {`.
  **L177 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movdiri", Attributes = [NoThrow] in {`。
- **L178 EN**: Declares TableGen def record `directstore_u64`.
  **L178 CN**: 声明 TableGen def 记录 `directstore_u64`。
- **L179 EN**: Closes the current lexical scope or compound statement.
  **L179 CN**: 结束当前词法作用域或复合语句块。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 181-200

````tablegen
let Features = "avx10.2", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {
  def vcvttsd2sis64 : X86Builtin<"long long int(_Vector<2, double>, _Constant int)">;
  def vcvttsd2usis64 : X86Builtin<"unsigned long long int(_Vector<2, double>, _Constant int)">;
  def vcvttss2sis64 : X86Builtin<"long long int(_Vector<4, float>, _Constant int)">;
  def vcvttss2usis64 : X86Builtin<"unsigned long long int(_Vector<4, float>, _Constant int)">;
}

let Features = "uintr", Attributes = [NoThrow] in {
  def clui : X86Builtin<"void()">;
  def stui : X86Builtin<"void()">;
  def testui : X86Builtin<"unsigned char()">;
  def senduipi : X86Builtin<"void(uint64_t)">;
}

let Features = "usermsr", Attributes = [NoThrow] in {
  def urdmsr : X86Builtin<"unsigned long long int(unsigned long long int)">;
  def uwrmsr : X86Builtin<"void(unsigned long long int, unsigned long long int)">;
}

let Features = "amx-tile", Attributes = [NoThrow] in {
````
- **L181 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "avx10.2", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`.
  **L181 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "avx10.2", Attributes = [NoThrow, Const, RequiredVectorWidth<128>] in {`。
- **L182 EN**: Declares TableGen def record `vcvttsd2sis64`.
  **L182 CN**: 声明 TableGen def 记录 `vcvttsd2sis64`。
- **L183 EN**: Declares TableGen def record `vcvttsd2usis64`.
  **L183 CN**: 声明 TableGen def 记录 `vcvttsd2usis64`。
- **L184 EN**: Declares TableGen def record `vcvttss2sis64`.
  **L184 CN**: 声明 TableGen def 记录 `vcvttss2sis64`。
- **L185 EN**: Declares TableGen def record `vcvttss2usis64`.
  **L185 CN**: 声明 TableGen def 记录 `vcvttss2usis64`。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L188 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "uintr", Attributes = [NoThrow] in {`.
  **L188 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "uintr", Attributes = [NoThrow] in {`。
- **L189 EN**: Declares TableGen def record `clui`.
  **L189 CN**: 声明 TableGen def 记录 `clui`。
- **L190 EN**: Declares TableGen def record `stui`.
  **L190 CN**: 声明 TableGen def 记录 `stui`。
- **L191 EN**: Declares TableGen def record `testui`.
  **L191 CN**: 声明 TableGen def 记录 `testui`。
- **L192 EN**: Declares TableGen def record `senduipi`.
  **L192 CN**: 声明 TableGen def 记录 `senduipi`。
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L195 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "usermsr", Attributes = [NoThrow] in {`.
  **L195 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "usermsr", Attributes = [NoThrow] in {`。
- **L196 EN**: Declares TableGen def record `urdmsr`.
  **L196 CN**: 声明 TableGen def 记录 `urdmsr`。
- **L197 EN**: Declares TableGen def record `uwrmsr`.
  **L197 CN**: 声明 TableGen def 记录 `uwrmsr`。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L200 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-tile", Attributes = [NoThrow] in {`.
  **L200 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-tile", Attributes = [NoThrow] in {`。

### Lines 201-220

````tablegen
  def tile_loadconfig_internal : X86Builtin<"void(void const *)">;
  def tileloadd64_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, void const *, size_t)">;
}

let Features = "amx-movrs", Attributes = [NoThrow] in {
  def tileloaddrs64_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, void const *, size_t)">;
}

let Features = "amx-tile", Attributes = [NoThrow] in {
  def tileloaddt164_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, void const *, size_t)">;
}

let Features = "amx-movrs", Attributes = [NoThrow] in {
  def tileloaddrst164_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, void const *, size_t)">;
}

let Features = "amx-int8", Attributes = [NoThrow] in {
  def tdpbssd_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
  def tdpbsud_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
  def tdpbusd_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
````
- **L201 EN**: Declares TableGen def record `tile_loadconfig_internal`.
  **L201 CN**: 声明 TableGen def 记录 `tile_loadconfig_internal`。
- **L202 EN**: Declares TableGen def record `tileloadd64_internal`.
  **L202 CN**: 声明 TableGen def 记录 `tileloadd64_internal`。
- **L203 EN**: Closes the current lexical scope or compound statement.
  **L203 CN**: 结束当前词法作用域或复合语句块。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L205 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-movrs", Attributes = [NoThrow] in {`.
  **L205 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-movrs", Attributes = [NoThrow] in {`。
- **L206 EN**: Declares TableGen def record `tileloaddrs64_internal`.
  **L206 CN**: 声明 TableGen def 记录 `tileloaddrs64_internal`。
- **L207 EN**: Closes the current lexical scope or compound statement.
  **L207 CN**: 结束当前词法作用域或复合语句块。
- **L208 EN**: Blank line separating nearby declarations or logic blocks.
  **L208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L209 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-tile", Attributes = [NoThrow] in {`.
  **L209 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-tile", Attributes = [NoThrow] in {`。
- **L210 EN**: Declares TableGen def record `tileloaddt164_internal`.
  **L210 CN**: 声明 TableGen def 记录 `tileloaddt164_internal`。
- **L211 EN**: Closes the current lexical scope or compound statement.
  **L211 CN**: 结束当前词法作用域或复合语句块。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L213 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-movrs", Attributes = [NoThrow] in {`.
  **L213 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-movrs", Attributes = [NoThrow] in {`。
- **L214 EN**: Declares TableGen def record `tileloaddrst164_internal`.
  **L214 CN**: 声明 TableGen def 记录 `tileloaddrst164_internal`。
- **L215 EN**: Closes the current lexical scope or compound statement.
  **L215 CN**: 结束当前词法作用域或复合语句块。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L217 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-int8", Attributes = [NoThrow] in {`.
  **L217 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-int8", Attributes = [NoThrow] in {`。
- **L218 EN**: Declares TableGen def record `tdpbssd_internal`.
  **L218 CN**: 声明 TableGen def 记录 `tdpbssd_internal`。
- **L219 EN**: Declares TableGen def record `tdpbsud_internal`.
  **L219 CN**: 声明 TableGen def 记录 `tdpbsud_internal`。
- **L220 EN**: Declares TableGen def record `tdpbusd_internal`.
  **L220 CN**: 声明 TableGen def 记录 `tdpbusd_internal`。

### Lines 221-240

````tablegen
  def tdpbuud_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
}

let Features = "amx-tile", Attributes = [NoThrow] in {
  def tilestored64_internal : X86Builtin<"void(unsigned short, unsigned short, void *, size_t, _Vector<256, int>)">;
  def tilezero_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short)">;
}

let Features = "amx-bf16", Attributes = [NoThrow] in {
  def tdpbf16ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
}

let Features = "amx-fp16", Attributes = [NoThrow] in {
  def tdpfp16ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
}

let Features = "amx-complex", Attributes = [NoThrow] in {
  def tcmmimfp16ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
  def tcmmrlfp16ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
}
````
- **L221 EN**: Declares TableGen def record `tdpbuud_internal`.
  **L221 CN**: 声明 TableGen def 记录 `tdpbuud_internal`。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-tile", Attributes = [NoThrow] in {`.
  **L224 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-tile", Attributes = [NoThrow] in {`。
- **L225 EN**: Declares TableGen def record `tilestored64_internal`.
  **L225 CN**: 声明 TableGen def 记录 `tilestored64_internal`。
- **L226 EN**: Declares TableGen def record `tilezero_internal`.
  **L226 CN**: 声明 TableGen def 记录 `tilezero_internal`。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L229 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-bf16", Attributes = [NoThrow] in {`.
  **L229 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-bf16", Attributes = [NoThrow] in {`。
- **L230 EN**: Declares TableGen def record `tdpbf16ps_internal`.
  **L230 CN**: 声明 TableGen def 记录 `tdpbf16ps_internal`。
- **L231 EN**: Closes the current lexical scope or compound statement.
  **L231 CN**: 结束当前词法作用域或复合语句块。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-fp16", Attributes = [NoThrow] in {`.
  **L233 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-fp16", Attributes = [NoThrow] in {`。
- **L234 EN**: Declares TableGen def record `tdpfp16ps_internal`.
  **L234 CN**: 声明 TableGen def 记录 `tdpfp16ps_internal`。
- **L235 EN**: Closes the current lexical scope or compound statement.
  **L235 CN**: 结束当前词法作用域或复合语句块。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-complex", Attributes = [NoThrow] in {`.
  **L237 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-complex", Attributes = [NoThrow] in {`。
- **L238 EN**: Declares TableGen def record `tcmmimfp16ps_internal`.
  **L238 CN**: 声明 TableGen def 记录 `tcmmimfp16ps_internal`。
- **L239 EN**: Declares TableGen def record `tcmmrlfp16ps_internal`.
  **L239 CN**: 声明 TableGen def 记录 `tcmmrlfp16ps_internal`。
- **L240 EN**: Closes the current lexical scope or compound statement.
  **L240 CN**: 结束当前词法作用域或复合语句块。

### Lines 241-260

````tablegen

let Features = "amx-avx512,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {
  def tcvtrowd2ps_internal : X86Builtin<"_Vector<16, float>(unsigned short, unsigned short, _Vector<256, int>, unsigned int)">;
  def tcvtrowps2bf16h_internal : X86Builtin<"_Vector<32, __bf16>(unsigned short, unsigned short, _Vector<256, int>, unsigned int)">;
  def tcvtrowps2bf16l_internal : X86Builtin<"_Vector<32, __bf16>(unsigned short, unsigned short, _Vector<256, int>, unsigned int)">;
  def tcvtrowps2phh_internal : X86Builtin<"_Vector<32, _Float16>(unsigned short, unsigned short, _Vector<256, int>, unsigned int)">;
  def tcvtrowps2phl_internal : X86Builtin<"_Vector<32, _Float16>(unsigned short, unsigned short, _Vector<256, int>, unsigned int)">;
  def tilemovrow_internal : X86Builtin<"_Vector<16, int>(unsigned short, unsigned short, _Vector<256, int>, unsigned int)">;
}

let Features = "amx-tf32", Attributes = [NoThrow] in {
  def tmmultf32ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
}

let Features = "amx-fp8", Attributes = [NoThrow] in {
  def tdpbf8ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
  def tdpbhf8ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
  def tdphbf8ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
  def tdphf8ps_internal : X86Builtin<"_Vector<256, int>(unsigned short, unsigned short, unsigned short, _Vector<256, int>, _Vector<256, int>, _Vector<256, int>)">;
}
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L242 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-avx512,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`.
  **L242 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-avx512,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`。
- **L243 EN**: Declares TableGen def record `tcvtrowd2ps_internal`.
  **L243 CN**: 声明 TableGen def 记录 `tcvtrowd2ps_internal`。
- **L244 EN**: Declares TableGen def record `tcvtrowps2bf16h_internal`.
  **L244 CN**: 声明 TableGen def 记录 `tcvtrowps2bf16h_internal`。
- **L245 EN**: Declares TableGen def record `tcvtrowps2bf16l_internal`.
  **L245 CN**: 声明 TableGen def 记录 `tcvtrowps2bf16l_internal`。
- **L246 EN**: Declares TableGen def record `tcvtrowps2phh_internal`.
  **L246 CN**: 声明 TableGen def 记录 `tcvtrowps2phh_internal`。
- **L247 EN**: Declares TableGen def record `tcvtrowps2phl_internal`.
  **L247 CN**: 声明 TableGen def 记录 `tcvtrowps2phl_internal`。
- **L248 EN**: Declares TableGen def record `tilemovrow_internal`.
  **L248 CN**: 声明 TableGen def 记录 `tilemovrow_internal`。
- **L249 EN**: Closes the current lexical scope or compound statement.
  **L249 CN**: 结束当前词法作用域或复合语句块。
- **L250 EN**: Blank line separating nearby declarations or logic blocks.
  **L250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L251 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-tf32", Attributes = [NoThrow] in {`.
  **L251 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-tf32", Attributes = [NoThrow] in {`。
- **L252 EN**: Declares TableGen def record `tmmultf32ps_internal`.
  **L252 CN**: 声明 TableGen def 记录 `tmmultf32ps_internal`。
- **L253 EN**: Closes the current lexical scope or compound statement.
  **L253 CN**: 结束当前词法作用域或复合语句块。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L255 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-fp8", Attributes = [NoThrow] in {`.
  **L255 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-fp8", Attributes = [NoThrow] in {`。
- **L256 EN**: Declares TableGen def record `tdpbf8ps_internal`.
  **L256 CN**: 声明 TableGen def 记录 `tdpbf8ps_internal`。
- **L257 EN**: Declares TableGen def record `tdpbhf8ps_internal`.
  **L257 CN**: 声明 TableGen def 记录 `tdpbhf8ps_internal`。
- **L258 EN**: Declares TableGen def record `tdphbf8ps_internal`.
  **L258 CN**: 声明 TableGen def 记录 `tdphbf8ps_internal`。
- **L259 EN**: Declares TableGen def record `tdphf8ps_internal`.
  **L259 CN**: 声明 TableGen def 记录 `tdphf8ps_internal`。
- **L260 EN**: Closes the current lexical scope or compound statement.
  **L260 CN**: 结束当前词法作用域或复合语句块。

### Lines 261-280

````tablegen

let Features = "amx-tile", Attributes = [NoThrow] in {
  def tile_loadconfig : X86Builtin<"void(void const *)">;
  def tile_storeconfig : X86Builtin<"void(void const *)">;
  def tilerelease : X86Builtin<"void()">;
  def tilezero : X86Builtin<"void(unsigned char)">;
}

let Features = "amx-movrs", Attributes = [NoThrow] in {
  def tileloaddrs64 : X86Builtin<"void(_Constant unsigned char, void const *, size_t)">;
  def tileloaddrst164 : X86Builtin<"void(_Constant unsigned char, void const *, size_t)">;
}

let Features = "amx-tile", Attributes = [NoThrow] in {
  def tileloadd64 : X86Builtin<"void(_Constant unsigned char, void const *, size_t)">;
  def tileloaddt164 : X86Builtin<"void(_Constant unsigned char, void const *, size_t)">;
  def tilestored64 : X86Builtin<"void(_Constant unsigned char, void *, size_t)">;
}

let Features = "amx-int8", Attributes = [NoThrow] in {
````
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L262 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-tile", Attributes = [NoThrow] in {`.
  **L262 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-tile", Attributes = [NoThrow] in {`。
- **L263 EN**: Declares TableGen def record `tile_loadconfig`.
  **L263 CN**: 声明 TableGen def 记录 `tile_loadconfig`。
- **L264 EN**: Declares TableGen def record `tile_storeconfig`.
  **L264 CN**: 声明 TableGen def 记录 `tile_storeconfig`。
- **L265 EN**: Declares TableGen def record `tilerelease`.
  **L265 CN**: 声明 TableGen def 记录 `tilerelease`。
- **L266 EN**: Declares TableGen def record `tilezero`.
  **L266 CN**: 声明 TableGen def 记录 `tilezero`。
- **L267 EN**: Closes the current lexical scope or compound statement.
  **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L269 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-movrs", Attributes = [NoThrow] in {`.
  **L269 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-movrs", Attributes = [NoThrow] in {`。
- **L270 EN**: Declares TableGen def record `tileloaddrs64`.
  **L270 CN**: 声明 TableGen def 记录 `tileloaddrs64`。
- **L271 EN**: Declares TableGen def record `tileloaddrst164`.
  **L271 CN**: 声明 TableGen def 记录 `tileloaddrst164`。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L274 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-tile", Attributes = [NoThrow] in {`.
  **L274 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-tile", Attributes = [NoThrow] in {`。
- **L275 EN**: Declares TableGen def record `tileloadd64`.
  **L275 CN**: 声明 TableGen def 记录 `tileloadd64`。
- **L276 EN**: Declares TableGen def record `tileloaddt164`.
  **L276 CN**: 声明 TableGen def 记录 `tileloaddt164`。
- **L277 EN**: Declares TableGen def record `tilestored64`.
  **L277 CN**: 声明 TableGen def 记录 `tilestored64`。
- **L278 EN**: Closes the current lexical scope or compound statement.
  **L278 CN**: 结束当前词法作用域或复合语句块。
- **L279 EN**: Blank line separating nearby declarations or logic blocks.
  **L279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L280 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-int8", Attributes = [NoThrow] in {`.
  **L280 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-int8", Attributes = [NoThrow] in {`。

### Lines 281-300

````tablegen
  def tdpbssd : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
  def tdpbsud : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
  def tdpbusd : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
  def tdpbuud : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
}

let Features = "amx-bf16", Attributes = [NoThrow] in {
  def tdpbf16ps : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
}

let Features = "ptwrite", Attributes = [NoThrow] in {
  def ptwrite64 : X86Builtin<"void(unsigned long long int)">;
}

let Features = "amx-complex", Attributes = [NoThrow] in {
  def tcmmimfp16ps : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
  def tcmmrlfp16ps : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
}

let Features = "amx-avx512,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {
````
- **L281 EN**: Declares TableGen def record `tdpbssd`.
  **L281 CN**: 声明 TableGen def 记录 `tdpbssd`。
- **L282 EN**: Declares TableGen def record `tdpbsud`.
  **L282 CN**: 声明 TableGen def 记录 `tdpbsud`。
- **L283 EN**: Declares TableGen def record `tdpbusd`.
  **L283 CN**: 声明 TableGen def 记录 `tdpbusd`。
- **L284 EN**: Declares TableGen def record `tdpbuud`.
  **L284 CN**: 声明 TableGen def 记录 `tdpbuud`。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L287 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-bf16", Attributes = [NoThrow] in {`.
  **L287 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-bf16", Attributes = [NoThrow] in {`。
- **L288 EN**: Declares TableGen def record `tdpbf16ps`.
  **L288 CN**: 声明 TableGen def 记录 `tdpbf16ps`。
- **L289 EN**: Closes the current lexical scope or compound statement.
  **L289 CN**: 结束当前词法作用域或复合语句块。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "ptwrite", Attributes = [NoThrow] in {`.
  **L291 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "ptwrite", Attributes = [NoThrow] in {`。
- **L292 EN**: Declares TableGen def record `ptwrite64`.
  **L292 CN**: 声明 TableGen def 记录 `ptwrite64`。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-complex", Attributes = [NoThrow] in {`.
  **L295 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-complex", Attributes = [NoThrow] in {`。
- **L296 EN**: Declares TableGen def record `tcmmimfp16ps`.
  **L296 CN**: 声明 TableGen def 记录 `tcmmimfp16ps`。
- **L297 EN**: Declares TableGen def record `tcmmrlfp16ps`.
  **L297 CN**: 声明 TableGen def 记录 `tcmmrlfp16ps`。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L300 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-avx512,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`.
  **L300 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-avx512,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`。

### Lines 301-320

````tablegen
  def tcvtrowd2ps : X86Builtin<"_Vector<16, float>(_Constant unsigned char, unsigned int)">;
  def tcvtrowps2bf16h : X86Builtin<"_Vector<32, __bf16>(_Constant unsigned char, unsigned int)">;
  def tcvtrowps2bf16l : X86Builtin<"_Vector<32, __bf16>(_Constant unsigned char, unsigned int)">;
  def tcvtrowps2phh : X86Builtin<"_Vector<32, _Float16>(_Constant unsigned char, unsigned int)">;
  def tcvtrowps2phl : X86Builtin<"_Vector<32, _Float16>(_Constant unsigned char, unsigned int)">;
  def tilemovrow : X86Builtin<"_Vector<16, int>(_Constant unsigned char, unsigned int)">;
  def tcvtrowd2psi : X86Builtin<"_Vector<16, float>(_Constant unsigned char, _Constant unsigned int)">;
  def tcvtrowps2bf16hi : X86Builtin<"_Vector<32, __bf16>(_Constant unsigned char, _Constant unsigned int)">;
  def tcvtrowps2bf16li : X86Builtin<"_Vector<32, __bf16>(_Constant unsigned char, _Constant unsigned int)">;
  def tcvtrowps2phhi : X86Builtin<"_Vector<32, _Float16>(_Constant unsigned char, _Constant unsigned int)">;
  def tcvtrowps2phli : X86Builtin<"_Vector<32, _Float16>(_Constant unsigned char, _Constant unsigned int)">;
  def tilemovrowi : X86Builtin<"_Vector<16, int>(_Constant unsigned char, _Constant unsigned int)">;
}

let Features = "amx-fp16", Attributes = [NoThrow] in {
  def tdpfp16ps : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
}

let Features = "amx-fp8", Attributes = [NoThrow] in {
  def tdpbf8ps : X86Builtin<"void(_Constant unsigned char, unsigned _Constant char, unsigned _Constant char)">;
````
- **L301 EN**: Declares TableGen def record `tcvtrowd2ps`.
  **L301 CN**: 声明 TableGen def 记录 `tcvtrowd2ps`。
- **L302 EN**: Declares TableGen def record `tcvtrowps2bf16h`.
  **L302 CN**: 声明 TableGen def 记录 `tcvtrowps2bf16h`。
- **L303 EN**: Declares TableGen def record `tcvtrowps2bf16l`.
  **L303 CN**: 声明 TableGen def 记录 `tcvtrowps2bf16l`。
- **L304 EN**: Declares TableGen def record `tcvtrowps2phh`.
  **L304 CN**: 声明 TableGen def 记录 `tcvtrowps2phh`。
- **L305 EN**: Declares TableGen def record `tcvtrowps2phl`.
  **L305 CN**: 声明 TableGen def 记录 `tcvtrowps2phl`。
- **L306 EN**: Declares TableGen def record `tilemovrow`.
  **L306 CN**: 声明 TableGen def 记录 `tilemovrow`。
- **L307 EN**: Declares TableGen def record `tcvtrowd2psi`.
  **L307 CN**: 声明 TableGen def 记录 `tcvtrowd2psi`。
- **L308 EN**: Declares TableGen def record `tcvtrowps2bf16hi`.
  **L308 CN**: 声明 TableGen def 记录 `tcvtrowps2bf16hi`。
- **L309 EN**: Declares TableGen def record `tcvtrowps2bf16li`.
  **L309 CN**: 声明 TableGen def 记录 `tcvtrowps2bf16li`。
- **L310 EN**: Declares TableGen def record `tcvtrowps2phhi`.
  **L310 CN**: 声明 TableGen def 记录 `tcvtrowps2phhi`。
- **L311 EN**: Declares TableGen def record `tcvtrowps2phli`.
  **L311 CN**: 声明 TableGen def 记录 `tcvtrowps2phli`。
- **L312 EN**: Declares TableGen def record `tilemovrowi`.
  **L312 CN**: 声明 TableGen def 记录 `tilemovrowi`。
- **L313 EN**: Closes the current lexical scope or compound statement.
  **L313 CN**: 结束当前词法作用域或复合语句块。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-fp16", Attributes = [NoThrow] in {`.
  **L315 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-fp16", Attributes = [NoThrow] in {`。
- **L316 EN**: Declares TableGen def record `tdpfp16ps`.
  **L316 CN**: 声明 TableGen def 记录 `tdpfp16ps`。
- **L317 EN**: Closes the current lexical scope or compound statement.
  **L317 CN**: 结束当前词法作用域或复合语句块。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L319 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-fp8", Attributes = [NoThrow] in {`.
  **L319 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-fp8", Attributes = [NoThrow] in {`。
- **L320 EN**: Declares TableGen def record `tdpbf8ps`.
  **L320 CN**: 声明 TableGen def 记录 `tdpbf8ps`。

### Lines 321-340

````tablegen
  def tdpbhf8ps : X86Builtin<"void(_Constant unsigned char, unsigned _Constant char, unsigned _Constant char)">;
  def tdphbf8ps : X86Builtin<"void(_Constant unsigned char, unsigned _Constant char, unsigned _Constant char)">;
  def tdphf8ps : X86Builtin<"void(_Constant unsigned char, unsigned _Constant char, unsigned _Constant char)">;
}

let Features = "amx-tf32", Attributes = [NoThrow] in {
  def tmmultf32ps : X86Builtin<"void(_Constant unsigned char, _Constant unsigned char, _Constant unsigned char)">;
}

let Features = "prefetchi", Attributes = [NoThrow, Const] in {
  def prefetchi : X86Builtin<"void(void const *, unsigned int)">;
}

let Features = "cmpccxadd", Attributes = [NoThrow] in {
  def cmpccxadd32 : X86Builtin<"signed int(void *, signed int, signed int, _Constant int)">;
  def cmpccxadd64 : X86Builtin<"signed long long int(signed long long int *, signed long long int, signed long long int, _Constant int)">;
}

let Features = "raoint", Attributes = [NoThrow] in {
  def aadd64 : X86Builtin<"void(void *, signed long long int)">;
````
- **L321 EN**: Declares TableGen def record `tdpbhf8ps`.
  **L321 CN**: 声明 TableGen def 记录 `tdpbhf8ps`。
- **L322 EN**: Declares TableGen def record `tdphbf8ps`.
  **L322 CN**: 声明 TableGen def 记录 `tdphbf8ps`。
- **L323 EN**: Declares TableGen def record `tdphf8ps`.
  **L323 CN**: 声明 TableGen def 记录 `tdphf8ps`。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L326 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "amx-tf32", Attributes = [NoThrow] in {`.
  **L326 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "amx-tf32", Attributes = [NoThrow] in {`。
- **L327 EN**: Declares TableGen def record `tmmultf32ps`.
  **L327 CN**: 声明 TableGen def 记录 `tmmultf32ps`。
- **L328 EN**: Closes the current lexical scope or compound statement.
  **L328 CN**: 结束当前词法作用域或复合语句块。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L330 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "prefetchi", Attributes = [NoThrow, Const] in {`.
  **L330 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "prefetchi", Attributes = [NoThrow, Const] in {`。
- **L331 EN**: Declares TableGen def record `prefetchi`.
  **L331 CN**: 声明 TableGen def 记录 `prefetchi`。
- **L332 EN**: Closes the current lexical scope or compound statement.
  **L332 CN**: 结束当前词法作用域或复合语句块。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L334 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "cmpccxadd", Attributes = [NoThrow] in {`.
  **L334 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "cmpccxadd", Attributes = [NoThrow] in {`。
- **L335 EN**: Declares TableGen def record `cmpccxadd32`.
  **L335 CN**: 声明 TableGen def 记录 `cmpccxadd32`。
- **L336 EN**: Declares TableGen def record `cmpccxadd64`.
  **L336 CN**: 声明 TableGen def 记录 `cmpccxadd64`。
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "raoint", Attributes = [NoThrow] in {`.
  **L339 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "raoint", Attributes = [NoThrow] in {`。
- **L340 EN**: Declares TableGen def record `aadd64`.
  **L340 CN**: 声明 TableGen def 记录 `aadd64`。

### Lines 341-360

````tablegen
  def aand64 : X86Builtin<"void(void *, signed long long int)">;
  def aor64 : X86Builtin<"void(void *, signed long long int)">;
  def axor64 : X86Builtin<"void(void *, signed long long int)">;
}

let Features = "movrs", Attributes = [NoThrow] in {
  def movrsqi : X86Builtin<"signed char(void const *)">;
  def movrshi : X86Builtin<"signed short(void const *)">;
  def movrssi : X86Builtin<"signed int(void const *)">;
  def movrsdi : X86Builtin<"signed long long int(void const *)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {
  def vmovrsb128 : X86Builtin<"_Vector<16, char>(_Vector<16, char const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {
  def vmovrsb256 : X86Builtin<"_Vector<32, char>(_Vector<32, char const *>)">;
}

````
- **L341 EN**: Declares TableGen def record `aand64`.
  **L341 CN**: 声明 TableGen def 记录 `aand64`。
- **L342 EN**: Declares TableGen def record `aor64`.
  **L342 CN**: 声明 TableGen def 记录 `aor64`。
- **L343 EN**: Declares TableGen def record `axor64`.
  **L343 CN**: 声明 TableGen def 记录 `axor64`。
- **L344 EN**: Closes the current lexical scope or compound statement.
  **L344 CN**: 结束当前词法作用域或复合语句块。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs", Attributes = [NoThrow] in {`.
  **L346 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs", Attributes = [NoThrow] in {`。
- **L347 EN**: Declares TableGen def record `movrsqi`.
  **L347 CN**: 声明 TableGen def 记录 `movrsqi`。
- **L348 EN**: Declares TableGen def record `movrshi`.
  **L348 CN**: 声明 TableGen def 记录 `movrshi`。
- **L349 EN**: Declares TableGen def record `movrssi`.
  **L349 CN**: 声明 TableGen def 记录 `movrssi`。
- **L350 EN**: Declares TableGen def record `movrsdi`.
  **L350 CN**: 声明 TableGen def 记录 `movrsdi`。
- **L351 EN**: Closes the current lexical scope or compound statement.
  **L351 CN**: 结束当前词法作用域或复合语句块。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`.
  **L353 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`。
- **L354 EN**: Declares TableGen def record `vmovrsb128`.
  **L354 CN**: 声明 TableGen def 记录 `vmovrsb128`。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L357 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`.
  **L357 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`。
- **L358 EN**: Declares TableGen def record `vmovrsb256`.
  **L358 CN**: 声明 TableGen def 记录 `vmovrsb256`。
- **L359 EN**: Closes the current lexical scope or compound statement.
  **L359 CN**: 结束当前词法作用域或复合语句块。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 361-380

````tablegen
let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {
  def vmovrsb512 : X86Builtin<"_Vector<64, char>(_Vector<64, char const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {
  def vmovrsd128 : X86Builtin<"_Vector<4, int>(_Vector<4, int const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {
  def vmovrsd256 : X86Builtin<"_Vector<8, int>(_Vector<8, int const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {
  def vmovrsd512 : X86Builtin<"_Vector<16, int>(_Vector<16, int const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {
  def vmovrsq128 : X86Builtin<"_Vector<2, long long int>(_Vector<2, long long int const *>)">;
}

````
- **L361 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`.
  **L361 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`。
- **L362 EN**: Declares TableGen def record `vmovrsb512`.
  **L362 CN**: 声明 TableGen def 记录 `vmovrsb512`。
- **L363 EN**: Closes the current lexical scope or compound statement.
  **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L365 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`.
  **L365 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`。
- **L366 EN**: Declares TableGen def record `vmovrsd128`.
  **L366 CN**: 声明 TableGen def 记录 `vmovrsd128`。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L369 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`.
  **L369 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`。
- **L370 EN**: Declares TableGen def record `vmovrsd256`.
  **L370 CN**: 声明 TableGen def 记录 `vmovrsd256`。
- **L371 EN**: Closes the current lexical scope or compound statement.
  **L371 CN**: 结束当前词法作用域或复合语句块。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L373 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`.
  **L373 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`。
- **L374 EN**: Declares TableGen def record `vmovrsd512`.
  **L374 CN**: 声明 TableGen def 记录 `vmovrsd512`。
- **L375 EN**: Closes the current lexical scope or compound statement.
  **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L377 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`.
  **L377 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`。
- **L378 EN**: Declares TableGen def record `vmovrsq128`.
  **L378 CN**: 声明 TableGen def 记录 `vmovrsq128`。
- **L379 EN**: Closes the current lexical scope or compound statement.
  **L379 CN**: 结束当前词法作用域或复合语句块。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 381-399

````tablegen
let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {
  def vmovrsq256 : X86Builtin<"_Vector<4, long long int>(_Vector<4, long long int const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {
  def vmovrsq512 : X86Builtin<"_Vector<8, long long int>(_Vector<8, long long int const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {
  def vmovrsw128 : X86Builtin<"_Vector<8, short>(_Vector<8, short const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {
  def vmovrsw256 : X86Builtin<"_Vector<16, short>(_Vector<16, short const *>)">;
}

let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {
  def vmovrsw512 : X86Builtin<"_Vector<32, short>(_Vector<32, short const *>)">;
}
````
- **L381 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`.
  **L381 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`。
- **L382 EN**: Declares TableGen def record `vmovrsq256`.
  **L382 CN**: 声明 TableGen def 记录 `vmovrsq256`。
- **L383 EN**: Closes the current lexical scope or compound statement.
  **L383 CN**: 结束当前词法作用域或复合语句块。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L385 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`.
  **L385 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`。
- **L386 EN**: Declares TableGen def record `vmovrsq512`.
  **L386 CN**: 声明 TableGen def 记录 `vmovrsq512`。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L389 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`.
  **L389 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<128>] in {`。
- **L390 EN**: Declares TableGen def record `vmovrsw128`.
  **L390 CN**: 声明 TableGen def 记录 `vmovrsw128`。
- **L391 EN**: Closes the current lexical scope or compound statement.
  **L391 CN**: 结束当前词法作用域或复合语句块。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`.
  **L393 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<256>] in {`。
- **L394 EN**: Declares TableGen def record `vmovrsw256`.
  **L394 CN**: 声明 TableGen def 记录 `vmovrsw256`。
- **L395 EN**: Closes the current lexical scope or compound statement.
  **L395 CN**: 结束当前词法作用域或复合语句块。
- **L396 EN**: Blank line separating nearby declarations or logic blocks.
  **L396 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L397 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`.
  **L397 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "movrs,avx10.2", Attributes = [NoThrow, RequiredVectorWidth<512>] in {`。
- **L398 EN**: Declares TableGen def record `vmovrsw512`.
  **L398 CN**: 声明 TableGen def 记录 `vmovrsw512`。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。

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
- **Construction vtables / 构造期虚表**
  - **EN**: Tracks VTT-related structures used during complex C++ object construction.
  - **CN**: 跟踪复杂 C++ 对象构造期间使用的 VTT 相关结构。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `char`, `int`, `void`, `int>`, `double>`, `float>`, `_Float16>`, `__bf16>`, `short`, `char>`, `short>`
- **TableGen records / TableGen 记录**: `_BitScanForward64`, `_BitScanReverse64`, `__mulh`, `__umulh`, `_mul128`, `_umul128`, `__faststorefence`, `__shiftleft128`, `__shiftright128`, `_InterlockedCompareExchange128`, `readeflags_u64`, `writeeflags_u64`, `cvtss2si64`, `cvttss2si64`, `cvtsd2si64`, `cvttsd2si64`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
