# BuiltinsAArch64.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsAArch64.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: AArch64 Builtin function database *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsAArch64` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 405

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````tablegen
//==- BuiltinsAArch64.td - AArch64 Builtin function database ----*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the AArch64-specific builtin function database.  Users of
// this file must define the BUILTIN macro to make use of this information.
//
//===----------------------------------------------------------------------===//

// The format of this database matches clang/Basic/Builtins.def.

include "clang/Basic/BuiltinsAArch64Base.td"

// In libgcc
let Attributes = [RuntimeLibraryFunction] in {
    def __clear_cache : AArch64NoPrefixBuiltin<"void (void *, void *)">;
````
- **L1 EN**: Comment explains nearby logic, constraints, or intent: `BuiltinsAArch64.td - AArch64 Builtin function database *- C++`.
  **L1 CN**: 注释解释附近代码的逻辑、约束或设计意图：`BuiltinsAArch64.td - AArch64 Builtin function database *- C++`。
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the AArch64-specific builtin function database. Users of`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the AArch64-specific builtin function database. Users of`。
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
- **L16 EN**: Includes TableGen file `"clang/Basic/BuiltinsAArch64Base.td"` so later records can reuse shared definitions.
  **L16 CN**: 引入 TableGen 文件 `"clang/Basic/BuiltinsAArch64Base.td"`，以便后续记录复用共享定义。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `In libgcc`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`In libgcc`。
- **L19 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [RuntimeLibraryFunction] in {`.
  **L19 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [RuntimeLibraryFunction] in {`。
- **L20 EN**: Declares TableGen def record `__clear_cache`.
  **L20 CN**: 声明 TableGen def 记录 `__clear_cache`。

### Lines 21-40

````tablegen
}

let Attributes = [CustomTypeChecking] in {
	def ldrex : AArch64Builtin<"void (...)">;
	def ldaex : AArch64Builtin<"void (...)">;
	def strex : AArch64Builtin<"int (...)">;
	def stlex : AArch64Builtin<"int (...)">;
}

def clrex : AArch64Builtin<"void ()">;

// Bit manipulation
let Attributes = [NoThrow, Const] in {
	def rbit   : AArch64Builtin<"unsigned int (unsigned int)">;
	def rbit64 : AArch64Builtin<"uint64_t (uint64_t)">;
	def cls    : AArch64Builtin<"unsigned int (uint32_t)">;
	def cls64  : AArch64Builtin<"unsigned int (uint64_t)">;
	def clz    : AArch64Builtin<"unsigned int (uint32_t)">;
	def clz64  : AArch64Builtin<"unsigned int (uint64_t)">;
}
````
- **L21 EN**: Closes the current lexical scope or compound statement.
  **L21 CN**: 结束当前词法作用域或复合语句块。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [CustomTypeChecking] in {`.
  **L23 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [CustomTypeChecking] in {`。
- **L24 EN**: Declares TableGen def record `ldrex`.
  **L24 CN**: 声明 TableGen def 记录 `ldrex`。
- **L25 EN**: Declares TableGen def record `ldaex`.
  **L25 CN**: 声明 TableGen def 记录 `ldaex`。
- **L26 EN**: Declares TableGen def record `strex`.
  **L26 CN**: 声明 TableGen def 记录 `strex`。
- **L27 EN**: Declares TableGen def record `stlex`.
  **L27 CN**: 声明 TableGen def 记录 `stlex`。
- **L28 EN**: Closes the current lexical scope or compound statement.
  **L28 CN**: 结束当前词法作用域或复合语句块。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Declares TableGen def record `clrex`.
  **L30 CN**: 声明 TableGen def 记录 `clrex`。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L32 EN**: Comment explains nearby logic, constraints, or intent: `Bit manipulation`.
  **L32 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit manipulation`。
- **L33 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in {`.
  **L33 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in {`。
- **L34 EN**: Declares TableGen def record `rbit`.
  **L34 CN**: 声明 TableGen def 记录 `rbit`。
- **L35 EN**: Declares TableGen def record `rbit64`.
  **L35 CN**: 声明 TableGen def 记录 `rbit64`。
- **L36 EN**: Declares TableGen def record `cls`.
  **L36 CN**: 声明 TableGen def 记录 `cls`。
- **L37 EN**: Declares TableGen def record `cls64`.
  **L37 CN**: 声明 TableGen def 记录 `cls64`。
- **L38 EN**: Declares TableGen def record `clz`.
  **L38 CN**: 声明 TableGen def 记录 `clz`。
- **L39 EN**: Declares TableGen def record `clz64`.
  **L39 CN**: 声明 TableGen def 记录 `clz64`。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。

### Lines 41-60

````tablegen

// HINT
def nop     : AArch64Builtin<"void ()">;
def yield   : AArch64Builtin<"void ()">;
def wfe     : AArch64Builtin<"void ()">;
def wfi     : AArch64Builtin<"void ()">;
def sev     : AArch64Builtin<"void ()">;
def sevl    : AArch64Builtin<"void ()">;
def chkfeat : AArch64Builtin<"uint64_t (uint64_t)">;

let Attributes = [RequireDeclaration], Languages = "ALL_LANGUAGES", Header = "arm_acle.h" in {
	def __yield : AArch64NoPrefixTargetLibBuiltin<"void ()">;
	def __wfe   : AArch64NoPrefixTargetLibBuiltin<"void ()">;
	def __wfi   : AArch64NoPrefixTargetLibBuiltin<"void ()">;
	def __sev   : AArch64NoPrefixTargetLibBuiltin<"void ()">;
	def __sevl  : AArch64NoPrefixTargetLibBuiltin<"void ()">;
}

// Like __builtin_trap but provide an 16-bit immediate reason code (which goes into `brk #N`).
let Attributes = [NoThrow, NoReturn] in {
````
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Comment explains nearby logic, constraints, or intent: `HINT`.
  **L42 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HINT`。
- **L43 EN**: Declares TableGen def record `nop`.
  **L43 CN**: 声明 TableGen def 记录 `nop`。
- **L44 EN**: Declares TableGen def record `yield`.
  **L44 CN**: 声明 TableGen def 记录 `yield`。
- **L45 EN**: Declares TableGen def record `wfe`.
  **L45 CN**: 声明 TableGen def 记录 `wfe`。
- **L46 EN**: Declares TableGen def record `wfi`.
  **L46 CN**: 声明 TableGen def 记录 `wfi`。
- **L47 EN**: Declares TableGen def record `sev`.
  **L47 CN**: 声明 TableGen def 记录 `sev`。
- **L48 EN**: Declares TableGen def record `sevl`.
  **L48 CN**: 声明 TableGen def 记录 `sevl`。
- **L49 EN**: Declares TableGen def record `chkfeat`.
  **L49 CN**: 声明 TableGen def 记录 `chkfeat`。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L51 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [RequireDeclaration], Languages = "ALL_LANGUAGES", Header = "arm_acle.h" in {`.
  **L51 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [RequireDeclaration], Languages = "ALL_LANGUAGES", Header = "arm_acle.h" in {`。
- **L52 EN**: Declares TableGen def record `__yield`.
  **L52 CN**: 声明 TableGen def 记录 `__yield`。
- **L53 EN**: Declares TableGen def record `__wfe`.
  **L53 CN**: 声明 TableGen def 记录 `__wfe`。
- **L54 EN**: Declares TableGen def record `__wfi`.
  **L54 CN**: 声明 TableGen def 记录 `__wfi`。
- **L55 EN**: Declares TableGen def record `__sev`.
  **L55 CN**: 声明 TableGen def 记录 `__sev`。
- **L56 EN**: Declares TableGen def record `__sevl`.
  **L56 CN**: 声明 TableGen def 记录 `__sevl`。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `Like __builtin_trap but provide an 16-bit immediate reason code (which goes into `brk #N`).`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Like __builtin_trap but provide an 16-bit immediate reason code (which goes into `brk #N`).`。
- **L60 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, NoReturn] in {`.
  **L60 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, NoReturn] in {`。

### Lines 61-80

````tablegen
	def trap : AArch64Builtin<"void (unsigned _Constant short)">;
}

// CRC32
let Attributes = [NoThrow, Const], Features = "crc" in {
	def crc32b  : AArch64TargetBuiltin<"unsigned int (unsigned int, unsigned char)">;
	def crc32cb : AArch64TargetBuiltin<"unsigned int (unsigned int, unsigned char)">;
	def crc32h  : AArch64TargetBuiltin<"unsigned int (unsigned int, unsigned short)">;
	def crc32ch : AArch64TargetBuiltin<"unsigned int (unsigned int, unsigned short)">;
	def crc32w  : AArch64TargetBuiltin<"unsigned int (unsigned int, unsigned int)">;
	def crc32cw : AArch64TargetBuiltin<"unsigned int (unsigned int, unsigned int)">;
	def crc32d  : AArch64TargetBuiltin<"unsigned int (unsigned int, uint64_t)">;
	def crc32cd : AArch64TargetBuiltin<"unsigned int (unsigned int, uint64_t)">;
}

// Memory Tagging Extensions (MTE)
let Attributes = [CustomTypeChecking], Features = "mte" in {
	def irg  : AArch64TargetBuiltin<"void * (void *, uint64_t)">;
	def addg : AArch64TargetBuiltin<"void * (void *, unsigned int)">;
	def gmi  : AArch64TargetBuiltin<"uint64_t (void *, uint64_t)">;
````
- **L61 EN**: Declares TableGen def record `trap`.
  **L61 CN**: 声明 TableGen def 记录 `trap`。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `CRC32`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`CRC32`。
- **L65 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "crc" in {`.
  **L65 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "crc" in {`。
- **L66 EN**: Declares TableGen def record `crc32b`.
  **L66 CN**: 声明 TableGen def 记录 `crc32b`。
- **L67 EN**: Declares TableGen def record `crc32cb`.
  **L67 CN**: 声明 TableGen def 记录 `crc32cb`。
- **L68 EN**: Declares TableGen def record `crc32h`.
  **L68 CN**: 声明 TableGen def 记录 `crc32h`。
- **L69 EN**: Declares TableGen def record `crc32ch`.
  **L69 CN**: 声明 TableGen def 记录 `crc32ch`。
- **L70 EN**: Declares TableGen def record `crc32w`.
  **L70 CN**: 声明 TableGen def 记录 `crc32w`。
- **L71 EN**: Declares TableGen def record `crc32cw`.
  **L71 CN**: 声明 TableGen def 记录 `crc32cw`。
- **L72 EN**: Declares TableGen def record `crc32d`.
  **L72 CN**: 声明 TableGen def 记录 `crc32d`。
- **L73 EN**: Declares TableGen def record `crc32cd`.
  **L73 CN**: 声明 TableGen def 记录 `crc32cd`。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `Memory Tagging Extensions (MTE)`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Memory Tagging Extensions (MTE)`。
- **L77 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [CustomTypeChecking], Features = "mte" in {`.
  **L77 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [CustomTypeChecking], Features = "mte" in {`。
- **L78 EN**: Declares TableGen def record `irg`.
  **L78 CN**: 声明 TableGen def 记录 `irg`。
- **L79 EN**: Declares TableGen def record `addg`.
  **L79 CN**: 声明 TableGen def 记录 `addg`。
- **L80 EN**: Declares TableGen def record `gmi`.
  **L80 CN**: 声明 TableGen def 记录 `gmi`。

### Lines 81-100

````tablegen
	def ldg  : AArch64TargetBuiltin<"void * (void *)">;
	def stg  : AArch64TargetBuiltin<"void (void *)">;
	def subp : AArch64TargetBuiltin<"int64_t (void *, void *)">;
}

// SME state function
let Attributes = [NoThrow] in {
	def get_sme_state : AArch64Builtin<"void (uint64_t *, uint64_t *)">;
}

// Memory Operations
let Features = "mte,mops" in {
	def mops_memset_tag : AArch64TargetBuiltin<"void * (void *, int, size_t)">;
}

// Memory barrier
let Attributes = [NoThrow, Const] in {
	def dmb : AArch64Builtin<"void (unsigned int)">;
	def dsb : AArch64Builtin<"void (unsigned int)">;
	def isb : AArch64Builtin<"void (unsigned int)">;
````
- **L81 EN**: Declares TableGen def record `ldg`.
  **L81 CN**: 声明 TableGen def 记录 `ldg`。
- **L82 EN**: Declares TableGen def record `stg`.
  **L82 CN**: 声明 TableGen def 记录 `stg`。
- **L83 EN**: Declares TableGen def record `subp`.
  **L83 CN**: 声明 TableGen def 记录 `subp`。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L86 EN**: Comment explains nearby logic, constraints, or intent: `SME state function`.
  **L86 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SME state function`。
- **L87 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L87 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L88 EN**: Declares TableGen def record `get_sme_state`.
  **L88 CN**: 声明 TableGen def 记录 `get_sme_state`。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L91 EN**: Comment explains nearby logic, constraints, or intent: `Memory Operations`.
  **L91 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Memory Operations`。
- **L92 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "mte,mops" in {`.
  **L92 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "mte,mops" in {`。
- **L93 EN**: Declares TableGen def record `mops_memset_tag`.
  **L93 CN**: 声明 TableGen def 记录 `mops_memset_tag`。
- **L94 EN**: Closes the current lexical scope or compound statement.
  **L94 CN**: 结束当前词法作用域或复合语句块。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `Memory barrier`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Memory barrier`。
- **L97 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in {`.
  **L97 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in {`。
- **L98 EN**: Declares TableGen def record `dmb`.
  **L98 CN**: 声明 TableGen def 记录 `dmb`。
- **L99 EN**: Declares TableGen def record `dsb`.
  **L99 CN**: 声明 TableGen def 记录 `dsb`。
- **L100 EN**: Declares TableGen def record `isb`.
  **L100 CN**: 声明 TableGen def 记录 `isb`。

### Lines 101-120

````tablegen
}

let Attributes = [NoThrow, Const, RequireDeclaration], Languages = "ALL_LANGUAGES", Header = "arm_acle.h" in {
	def __dmb : AArch64NoPrefixTargetLibBuiltin<"void (unsigned int)">;
	def __dsb : AArch64NoPrefixTargetLibBuiltin<"void (unsigned int)">;
	def __isb : AArch64NoPrefixTargetLibBuiltin<"void (unsigned int)">;
}

let Attributes = [NoThrow, Const], Features = "v8.3a" in {
	def jcvt : AArch64TargetBuiltin<"int32_t (double)">;
}

// Prefetch
let Attributes = [NoThrow, Const] in {
	def prefetch : AArch64Builtin<"void (void const *, unsigned int, unsigned int, unsigned int, unsigned int)">;
}

let Attributes = [NoThrow, Const] in {
	def prefetch_ir : AArch64Builtin<"void (void const *)">;
}
````
- **L101 EN**: Closes the current lexical scope or compound statement.
  **L101 CN**: 结束当前词法作用域或复合语句块。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L103 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const, RequireDeclaration], Languages = "ALL_LANGUAGES", Header = "arm_acle.h" in {`.
  **L103 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const, RequireDeclaration], Languages = "ALL_LANGUAGES", Header = "arm_acle.h" in {`。
- **L104 EN**: Declares TableGen def record `__dmb`.
  **L104 CN**: 声明 TableGen def 记录 `__dmb`。
- **L105 EN**: Declares TableGen def record `__dsb`.
  **L105 CN**: 声明 TableGen def 记录 `__dsb`。
- **L106 EN**: Declares TableGen def record `__isb`.
  **L106 CN**: 声明 TableGen def 记录 `__isb`。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L109 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "v8.3a" in {`.
  **L109 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "v8.3a" in {`。
- **L110 EN**: Declares TableGen def record `jcvt`.
  **L110 CN**: 声明 TableGen def 记录 `jcvt`。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L113 EN**: Comment explains nearby logic, constraints, or intent: `Prefetch`.
  **L113 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetch`。
- **L114 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in {`.
  **L114 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in {`。
- **L115 EN**: Declares TableGen def record `prefetch`.
  **L115 CN**: 声明 TableGen def 记录 `prefetch`。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L118 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in {`.
  **L118 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in {`。
- **L119 EN**: Declares TableGen def record `prefetch_ir`.
  **L119 CN**: 声明 TableGen def 记录 `prefetch_ir`。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-140

````tablegen

// Range Prefetch
let Attributes = [NoThrow] in {
	def range_prefetch_x : AArch64Builtin<"void (void const *, unsigned int, unsigned int, int, unsigned int, int, size_t)">;
	def range_prefetch   : AArch64Builtin<"void (void const *, unsigned int, unsigned int, uint64_t)">;
}

// System Registers
let Attributes = [NoThrow, Const] in {
	def rsr   : AArch64Builtin<"unsigned int (char const *)">;
	def rsr64 : AArch64Builtin<"uint64_t (char const *)">;
	def rsrp  : AArch64Builtin<"void * (char const *)">;
	def wsr   : AArch64Builtin<"void (char const *, unsigned int)">;
	def wsr64 : AArch64Builtin<"void (char const *, uint64_t)">;
	def wsrp  : AArch64Builtin<"void (char const *, void const *)">;
}

let Attributes = [NoThrow, Const], Features = "d128" in {
	def rsr128 : AArch64TargetBuiltin<"unsigned __int128_t (char const *)">; // __int128_t
	def wsr128 : AArch64TargetBuiltin<"void (char const *, unsigned__int128_t)">;
````
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L122 EN**: Comment explains nearby logic, constraints, or intent: `Range Prefetch`.
  **L122 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Range Prefetch`。
- **L123 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L123 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L124 EN**: Declares TableGen def record `range_prefetch_x`.
  **L124 CN**: 声明 TableGen def 记录 `range_prefetch_x`。
- **L125 EN**: Declares TableGen def record `range_prefetch`.
  **L125 CN**: 声明 TableGen def 记录 `range_prefetch`。
- **L126 EN**: Closes the current lexical scope or compound statement.
  **L126 CN**: 结束当前词法作用域或复合语句块。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L128 EN**: Comment explains nearby logic, constraints, or intent: `System Registers`.
  **L128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`System Registers`。
- **L129 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in {`.
  **L129 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in {`。
- **L130 EN**: Declares TableGen def record `rsr`.
  **L130 CN**: 声明 TableGen def 记录 `rsr`。
- **L131 EN**: Declares TableGen def record `rsr64`.
  **L131 CN**: 声明 TableGen def 记录 `rsr64`。
- **L132 EN**: Declares TableGen def record `rsrp`.
  **L132 CN**: 声明 TableGen def 记录 `rsrp`。
- **L133 EN**: Declares TableGen def record `wsr`.
  **L133 CN**: 声明 TableGen def 记录 `wsr`。
- **L134 EN**: Declares TableGen def record `wsr64`.
  **L134 CN**: 声明 TableGen def 记录 `wsr64`。
- **L135 EN**: Declares TableGen def record `wsrp`.
  **L135 CN**: 声明 TableGen def 记录 `wsrp`。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const], Features = "d128" in {`.
  **L138 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const], Features = "d128" in {`。
- **L139 EN**: Declares TableGen def record `rsr128`.
  **L139 CN**: 声明 TableGen def 记录 `rsr128`。
- **L140 EN**: Declares TableGen def record `wsr128`.
  **L140 CN**: 声明 TableGen def 记录 `wsr128`。

### Lines 141-160

````tablegen
}

// Misc
let Attributes = [Const] in {
	def __builtin_sponentry : AArch64NoPrefixBuiltin<"void * ()">;
}

// Armv8.5-A FP rounding intrinsics
let Features = "v8.5a" in {
	def rint32zf : AArch64TargetBuiltin<"float (float)">;
	def rint32z  : AArch64TargetBuiltin<"double (double)">;
	def rint64zf : AArch64TargetBuiltin<"float (float)">;
	def rint64z  : AArch64TargetBuiltin<"double (double)">;
	def rint32xf : AArch64TargetBuiltin<"float (float)">;
	def rint32x  : AArch64TargetBuiltin<"double (double)">;
	def rint64xf : AArch64TargetBuiltin<"float (float)">;
	def rint64x  : AArch64TargetBuiltin<"double (double)">;
}

// Armv8.5-A Random number generation intrinsics
````
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `Misc`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Misc`。
- **L144 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [Const] in {`.
  **L144 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [Const] in {`。
- **L145 EN**: Declares TableGen def record `__builtin_sponentry`.
  **L145 CN**: 声明 TableGen def 记录 `__builtin_sponentry`。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Comment explains nearby logic, constraints, or intent: `Armv8.5-A FP rounding intrinsics`.
  **L148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Armv8.5-A FP rounding intrinsics`。
- **L149 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "v8.5a" in {`.
  **L149 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "v8.5a" in {`。
- **L150 EN**: Declares TableGen def record `rint32zf`.
  **L150 CN**: 声明 TableGen def 记录 `rint32zf`。
- **L151 EN**: Declares TableGen def record `rint32z`.
  **L151 CN**: 声明 TableGen def 记录 `rint32z`。
- **L152 EN**: Declares TableGen def record `rint64zf`.
  **L152 CN**: 声明 TableGen def 记录 `rint64zf`。
- **L153 EN**: Declares TableGen def record `rint64z`.
  **L153 CN**: 声明 TableGen def 记录 `rint64z`。
- **L154 EN**: Declares TableGen def record `rint32xf`.
  **L154 CN**: 声明 TableGen def 记录 `rint32xf`。
- **L155 EN**: Declares TableGen def record `rint32x`.
  **L155 CN**: 声明 TableGen def 记录 `rint32x`。
- **L156 EN**: Declares TableGen def record `rint64xf`.
  **L156 CN**: 声明 TableGen def 记录 `rint64xf`。
- **L157 EN**: Declares TableGen def record `rint64x`.
  **L157 CN**: 声明 TableGen def 记录 `rint64x`。
- **L158 EN**: Closes the current lexical scope or compound statement.
  **L158 CN**: 结束当前词法作用域或复合语句块。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L160 EN**: Comment explains nearby logic, constraints, or intent: `Armv8.5-A Random number generation intrinsics`.
  **L160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Armv8.5-A Random number generation intrinsics`。

### Lines 161-180

````tablegen
let Attributes = [NoThrow], Features = "rand" in {
	def rndr   : AArch64TargetBuiltin<"int (uint64_t *)">;
	def rndrrs : AArch64TargetBuiltin<"int (uint64_t *)">;
}

// Armv8.7-A load/store 64-byte intrinsics
let Attributes = [NoThrow], Features = "ls64" in {
	def ld64b   : AArch64TargetBuiltin<"void (void const *, uint64_t *)">;
	def st64b   : AArch64TargetBuiltin<"void (void *, uint64_t const *)">;
	def st64bv  : AArch64TargetBuiltin<"uint64_t (void *, uint64_t const *)">;
	def st64bv0 : AArch64TargetBuiltin<"uint64_t (void *, uint64_t const *)">;
}

// Armv9.3-A Guarded Control Stack
let Attributes = [NoThrow], Features = "gcs" in {
	def gcspopm : AArch64TargetBuiltin<"uint64_t (uint64_t)">;
	def gcsss   : AArch64TargetBuiltin<"void * (void *)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
````
- **L161 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow], Features = "rand" in {`.
  **L161 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow], Features = "rand" in {`。
- **L162 EN**: Declares TableGen def record `rndr`.
  **L162 CN**: 声明 TableGen def 记录 `rndr`。
- **L163 EN**: Declares TableGen def record `rndrrs`.
  **L163 CN**: 声明 TableGen def 记录 `rndrrs`。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `Armv8.7-A load/store 64-byte intrinsics`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Armv8.7-A load/store 64-byte intrinsics`。
- **L167 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow], Features = "ls64" in {`.
  **L167 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow], Features = "ls64" in {`。
- **L168 EN**: Declares TableGen def record `ld64b`.
  **L168 CN**: 声明 TableGen def 记录 `ld64b`。
- **L169 EN**: Declares TableGen def record `st64b`.
  **L169 CN**: 声明 TableGen def 记录 `st64b`。
- **L170 EN**: Declares TableGen def record `st64bv`.
  **L170 CN**: 声明 TableGen def 记录 `st64bv`。
- **L171 EN**: Declares TableGen def record `st64bv0`.
  **L171 CN**: 声明 TableGen def 记录 `st64bv0`。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L174 EN**: Comment explains nearby logic, constraints, or intent: `Armv9.3-A Guarded Control Stack`.
  **L174 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Armv9.3-A Guarded Control Stack`。
- **L175 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow], Features = "gcs" in {`.
  **L175 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow], Features = "gcs" in {`。
- **L176 EN**: Declares TableGen def record `gcspopm`.
  **L176 CN**: 声明 TableGen def 记录 `gcspopm`。
- **L177 EN**: Declares TableGen def record `gcsss`.
  **L177 CN**: 声明 TableGen def 记录 `gcsss`。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Blank line separating nearby declarations or logic blocks.
  **L179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L180 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L180 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。

### Lines 181-200

````tablegen
	def _BitScanForward   : AArch64NoPrefixTargetLibBuiltin<"unsigned char (msuint32_t *, msuint32_t)">;
	def _BitScanReverse   : AArch64NoPrefixTargetLibBuiltin<"unsigned char (msuint32_t *, msuint32_t)">;
	def _BitScanForward64 : AArch64NoPrefixTargetLibBuiltin<"unsigned char (msuint32_t *, unsigned long long int)">;
	def _BitScanReverse64 : AArch64NoPrefixTargetLibBuiltin<"unsigned char (msuint32_t *, unsigned long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedAdd           : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedAdd64         : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedAnd64         : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedDecrement64   : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
	def _InterlockedExchange64    : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedExchangeAdd64 : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedExchangeSub64 : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedIncrement64   : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
	def _InterlockedOr64          : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedXor64         : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
````
- **L181 EN**: Declares TableGen def record `_BitScanForward`.
  **L181 CN**: 声明 TableGen def 记录 `_BitScanForward`。
- **L182 EN**: Declares TableGen def record `_BitScanReverse`.
  **L182 CN**: 声明 TableGen def 记录 `_BitScanReverse`。
- **L183 EN**: Declares TableGen def record `_BitScanForward64`.
  **L183 CN**: 声明 TableGen def 记录 `_BitScanForward64`。
- **L184 EN**: Declares TableGen def record `_BitScanReverse64`.
  **L184 CN**: 声明 TableGen def 记录 `_BitScanReverse64`。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Blank line separating nearby declarations or logic blocks.
  **L186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L187 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L187 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L188 EN**: Declares TableGen def record `_InterlockedAdd`.
  **L188 CN**: 声明 TableGen def 记录 `_InterlockedAdd`。
- **L189 EN**: Declares TableGen def record `_InterlockedAdd64`.
  **L189 CN**: 声明 TableGen def 记录 `_InterlockedAdd64`。
- **L190 EN**: Declares TableGen def record `_InterlockedAnd64`.
  **L190 CN**: 声明 TableGen def 记录 `_InterlockedAnd64`。
- **L191 EN**: Declares TableGen def record `_InterlockedDecrement64`.
  **L191 CN**: 声明 TableGen def 记录 `_InterlockedDecrement64`。
- **L192 EN**: Declares TableGen def record `_InterlockedExchange64`.
  **L192 CN**: 声明 TableGen def 记录 `_InterlockedExchange64`。
- **L193 EN**: Declares TableGen def record `_InterlockedExchangeAdd64`.
  **L193 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd64`。
- **L194 EN**: Declares TableGen def record `_InterlockedExchangeSub64`.
  **L194 CN**: 声明 TableGen def 记录 `_InterlockedExchangeSub64`。
- **L195 EN**: Declares TableGen def record `_InterlockedIncrement64`.
  **L195 CN**: 声明 TableGen def 记录 `_InterlockedIncrement64`。
- **L196 EN**: Declares TableGen def record `_InterlockedOr64`.
  **L196 CN**: 声明 TableGen def 记录 `_InterlockedOr64`。
- **L197 EN**: Declares TableGen def record `_InterlockedXor64`.
  **L197 CN**: 声明 TableGen def 记录 `_InterlockedXor64`。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L200 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L200 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。

### Lines 201-220

````tablegen
	def _InterlockedAdd_acq   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedAdd_rel   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedAdd_nf    : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedAdd64_acq : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedAdd64_rel : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedAdd64_nf  : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedExchangeAdd_acq   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedExchangeAdd_rel   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedExchangeAdd_nf    : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedExchangeAdd8_acq  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedExchangeAdd8_rel  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedExchangeAdd8_nf   : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedExchangeAdd16_acq : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedExchangeAdd16_rel : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedExchangeAdd16_nf  : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedExchangeAdd64_acq : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedExchangeAdd64_rel : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
````
- **L201 EN**: Declares TableGen def record `_InterlockedAdd_acq`.
  **L201 CN**: 声明 TableGen def 记录 `_InterlockedAdd_acq`。
- **L202 EN**: Declares TableGen def record `_InterlockedAdd_rel`.
  **L202 CN**: 声明 TableGen def 记录 `_InterlockedAdd_rel`。
- **L203 EN**: Declares TableGen def record `_InterlockedAdd_nf`.
  **L203 CN**: 声明 TableGen def 记录 `_InterlockedAdd_nf`。
- **L204 EN**: Declares TableGen def record `_InterlockedAdd64_acq`.
  **L204 CN**: 声明 TableGen def 记录 `_InterlockedAdd64_acq`。
- **L205 EN**: Declares TableGen def record `_InterlockedAdd64_rel`.
  **L205 CN**: 声明 TableGen def 记录 `_InterlockedAdd64_rel`。
- **L206 EN**: Declares TableGen def record `_InterlockedAdd64_nf`.
  **L206 CN**: 声明 TableGen def 记录 `_InterlockedAdd64_nf`。
- **L207 EN**: Closes the current lexical scope or compound statement.
  **L207 CN**: 结束当前词法作用域或复合语句块。
- **L208 EN**: Blank line separating nearby declarations or logic blocks.
  **L208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L209 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L209 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L210 EN**: Declares TableGen def record `_InterlockedExchangeAdd_acq`.
  **L210 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd_acq`。
- **L211 EN**: Declares TableGen def record `_InterlockedExchangeAdd_rel`.
  **L211 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd_rel`。
- **L212 EN**: Declares TableGen def record `_InterlockedExchangeAdd_nf`.
  **L212 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd_nf`。
- **L213 EN**: Declares TableGen def record `_InterlockedExchangeAdd8_acq`.
  **L213 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd8_acq`。
- **L214 EN**: Declares TableGen def record `_InterlockedExchangeAdd8_rel`.
  **L214 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd8_rel`。
- **L215 EN**: Declares TableGen def record `_InterlockedExchangeAdd8_nf`.
  **L215 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd8_nf`。
- **L216 EN**: Declares TableGen def record `_InterlockedExchangeAdd16_acq`.
  **L216 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd16_acq`。
- **L217 EN**: Declares TableGen def record `_InterlockedExchangeAdd16_rel`.
  **L217 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd16_rel`。
- **L218 EN**: Declares TableGen def record `_InterlockedExchangeAdd16_nf`.
  **L218 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd16_nf`。
- **L219 EN**: Declares TableGen def record `_InterlockedExchangeAdd64_acq`.
  **L219 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd64_acq`。
- **L220 EN**: Declares TableGen def record `_InterlockedExchangeAdd64_rel`.
  **L220 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd64_rel`。

### Lines 221-240

````tablegen
	def _InterlockedExchangeAdd64_nf  : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedExchange8_acq       : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedExchange8_nf        : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedExchange8_rel       : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedExchange16_acq      : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedExchange16_nf       : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedExchange16_rel      : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedExchange_acq        : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedExchange_nf         : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedExchange_rel        : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedExchange64_acq      : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedExchange64_nf       : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedExchange64_rel      : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedExchangePointer_acq : AArch64NoPrefixTargetLibBuiltin<"void * (void * volatile *, void *)">;
	def _InterlockedExchangePointer_nf  : AArch64NoPrefixTargetLibBuiltin<"void * (void * volatile *, void *)">;
	def _InterlockedExchangePointer_rel : AArch64NoPrefixTargetLibBuiltin<"void * (void * volatile *, void *)">;
}
````
- **L221 EN**: Declares TableGen def record `_InterlockedExchangeAdd64_nf`.
  **L221 CN**: 声明 TableGen def 记录 `_InterlockedExchangeAdd64_nf`。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L224 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L225 EN**: Declares TableGen def record `_InterlockedExchange8_acq`.
  **L225 CN**: 声明 TableGen def 记录 `_InterlockedExchange8_acq`。
- **L226 EN**: Declares TableGen def record `_InterlockedExchange8_nf`.
  **L226 CN**: 声明 TableGen def 记录 `_InterlockedExchange8_nf`。
- **L227 EN**: Declares TableGen def record `_InterlockedExchange8_rel`.
  **L227 CN**: 声明 TableGen def 记录 `_InterlockedExchange8_rel`。
- **L228 EN**: Declares TableGen def record `_InterlockedExchange16_acq`.
  **L228 CN**: 声明 TableGen def 记录 `_InterlockedExchange16_acq`。
- **L229 EN**: Declares TableGen def record `_InterlockedExchange16_nf`.
  **L229 CN**: 声明 TableGen def 记录 `_InterlockedExchange16_nf`。
- **L230 EN**: Declares TableGen def record `_InterlockedExchange16_rel`.
  **L230 CN**: 声明 TableGen def 记录 `_InterlockedExchange16_rel`。
- **L231 EN**: Declares TableGen def record `_InterlockedExchange_acq`.
  **L231 CN**: 声明 TableGen def 记录 `_InterlockedExchange_acq`。
- **L232 EN**: Declares TableGen def record `_InterlockedExchange_nf`.
  **L232 CN**: 声明 TableGen def 记录 `_InterlockedExchange_nf`。
- **L233 EN**: Declares TableGen def record `_InterlockedExchange_rel`.
  **L233 CN**: 声明 TableGen def 记录 `_InterlockedExchange_rel`。
- **L234 EN**: Declares TableGen def record `_InterlockedExchange64_acq`.
  **L234 CN**: 声明 TableGen def 记录 `_InterlockedExchange64_acq`。
- **L235 EN**: Declares TableGen def record `_InterlockedExchange64_nf`.
  **L235 CN**: 声明 TableGen def 记录 `_InterlockedExchange64_nf`。
- **L236 EN**: Declares TableGen def record `_InterlockedExchange64_rel`.
  **L236 CN**: 声明 TableGen def 记录 `_InterlockedExchange64_rel`。
- **L237 EN**: Declares TableGen def record `_InterlockedExchangePointer_acq`.
  **L237 CN**: 声明 TableGen def 记录 `_InterlockedExchangePointer_acq`。
- **L238 EN**: Declares TableGen def record `_InterlockedExchangePointer_nf`.
  **L238 CN**: 声明 TableGen def 记录 `_InterlockedExchangePointer_nf`。
- **L239 EN**: Declares TableGen def record `_InterlockedExchangePointer_rel`.
  **L239 CN**: 声明 TableGen def 记录 `_InterlockedExchangePointer_rel`。
- **L240 EN**: Closes the current lexical scope or compound statement.
  **L240 CN**: 结束当前词法作用域或复合语句块。

### Lines 241-260

````tablegen

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedCompareExchange8_acq       : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char, char)">;
	def _InterlockedCompareExchange8_nf        : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char, char)">;
	def _InterlockedCompareExchange8_rel       : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char, char)">;
	def _InterlockedCompareExchange16_acq      : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short, short)">;
	def _InterlockedCompareExchange16_nf       : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short, short)">;
	def _InterlockedCompareExchange16_rel      : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short, short)">;
	def _InterlockedCompareExchange_acq        : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t, msint32_t)">;
	def _InterlockedCompareExchange_nf         : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t, msint32_t)">;
	def _InterlockedCompareExchange_rel        : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t, msint32_t)">;
	def _InterlockedCompareExchange64_acq      : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int, long long int)">;
	def _InterlockedCompareExchange64_nf       : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int, long long int)">;
	def _InterlockedCompareExchange64_rel      : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int, long long int)">;
	def _InterlockedCompareExchangePointer_acq : AArch64NoPrefixTargetLibBuiltin<"void * (void * volatile *, void *, void *)">;
	def _InterlockedCompareExchangePointer_rel : AArch64NoPrefixTargetLibBuiltin<"void * (void * volatile *, void *, void *)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedCompareExchange128     : AArch64NoPrefixTargetLibBuiltin<"unsigned char (long long int volatile *, long long int, long long int, long long int *)">;
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L242 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L242 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L243 EN**: Declares TableGen def record `_InterlockedCompareExchange8_acq`.
  **L243 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange8_acq`。
- **L244 EN**: Declares TableGen def record `_InterlockedCompareExchange8_nf`.
  **L244 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange8_nf`。
- **L245 EN**: Declares TableGen def record `_InterlockedCompareExchange8_rel`.
  **L245 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange8_rel`。
- **L246 EN**: Declares TableGen def record `_InterlockedCompareExchange16_acq`.
  **L246 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange16_acq`。
- **L247 EN**: Declares TableGen def record `_InterlockedCompareExchange16_nf`.
  **L247 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange16_nf`。
- **L248 EN**: Declares TableGen def record `_InterlockedCompareExchange16_rel`.
  **L248 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange16_rel`。
- **L249 EN**: Declares TableGen def record `_InterlockedCompareExchange_acq`.
  **L249 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange_acq`。
- **L250 EN**: Declares TableGen def record `_InterlockedCompareExchange_nf`.
  **L250 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange_nf`。
- **L251 EN**: Declares TableGen def record `_InterlockedCompareExchange_rel`.
  **L251 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange_rel`。
- **L252 EN**: Declares TableGen def record `_InterlockedCompareExchange64_acq`.
  **L252 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange64_acq`。
- **L253 EN**: Declares TableGen def record `_InterlockedCompareExchange64_nf`.
  **L253 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange64_nf`。
- **L254 EN**: Declares TableGen def record `_InterlockedCompareExchange64_rel`.
  **L254 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange64_rel`。
- **L255 EN**: Declares TableGen def record `_InterlockedCompareExchangePointer_acq`.
  **L255 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchangePointer_acq`。
- **L256 EN**: Declares TableGen def record `_InterlockedCompareExchangePointer_rel`.
  **L256 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchangePointer_rel`。
- **L257 EN**: Closes the current lexical scope or compound statement.
  **L257 CN**: 结束当前词法作用域或复合语句块。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L259 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L259 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L260 EN**: Declares TableGen def record `_InterlockedCompareExchange128`.
  **L260 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange128`。

### Lines 261-280

````tablegen
	def _InterlockedCompareExchange128_acq : AArch64NoPrefixTargetLibBuiltin<"unsigned char (long long int volatile *, long long int, long long int, long long int *)">;
	def _InterlockedCompareExchange128_nf  : AArch64NoPrefixTargetLibBuiltin<"unsigned char (long long int volatile *, long long int, long long int, long long int *)">;
	def _InterlockedCompareExchange128_rel : AArch64NoPrefixTargetLibBuiltin<"unsigned char (long long int volatile *, long long int, long long int, long long int *)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedOr8_acq  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedOr8_nf   : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedOr8_rel  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedOr16_acq : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedOr16_nf  : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedOr16_rel : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedOr_acq   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedOr_nf    : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedOr_rel   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedOr64_acq : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedOr64_nf  : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedOr64_rel : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
}

````
- **L261 EN**: Declares TableGen def record `_InterlockedCompareExchange128_acq`.
  **L261 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange128_acq`。
- **L262 EN**: Declares TableGen def record `_InterlockedCompareExchange128_nf`.
  **L262 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange128_nf`。
- **L263 EN**: Declares TableGen def record `_InterlockedCompareExchange128_rel`.
  **L263 CN**: 声明 TableGen def 记录 `_InterlockedCompareExchange128_rel`。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L266 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L266 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L267 EN**: Declares TableGen def record `_InterlockedOr8_acq`.
  **L267 CN**: 声明 TableGen def 记录 `_InterlockedOr8_acq`。
- **L268 EN**: Declares TableGen def record `_InterlockedOr8_nf`.
  **L268 CN**: 声明 TableGen def 记录 `_InterlockedOr8_nf`。
- **L269 EN**: Declares TableGen def record `_InterlockedOr8_rel`.
  **L269 CN**: 声明 TableGen def 记录 `_InterlockedOr8_rel`。
- **L270 EN**: Declares TableGen def record `_InterlockedOr16_acq`.
  **L270 CN**: 声明 TableGen def 记录 `_InterlockedOr16_acq`。
- **L271 EN**: Declares TableGen def record `_InterlockedOr16_nf`.
  **L271 CN**: 声明 TableGen def 记录 `_InterlockedOr16_nf`。
- **L272 EN**: Declares TableGen def record `_InterlockedOr16_rel`.
  **L272 CN**: 声明 TableGen def 记录 `_InterlockedOr16_rel`。
- **L273 EN**: Declares TableGen def record `_InterlockedOr_acq`.
  **L273 CN**: 声明 TableGen def 记录 `_InterlockedOr_acq`。
- **L274 EN**: Declares TableGen def record `_InterlockedOr_nf`.
  **L274 CN**: 声明 TableGen def 记录 `_InterlockedOr_nf`。
- **L275 EN**: Declares TableGen def record `_InterlockedOr_rel`.
  **L275 CN**: 声明 TableGen def 记录 `_InterlockedOr_rel`。
- **L276 EN**: Declares TableGen def record `_InterlockedOr64_acq`.
  **L276 CN**: 声明 TableGen def 记录 `_InterlockedOr64_acq`。
- **L277 EN**: Declares TableGen def record `_InterlockedOr64_nf`.
  **L277 CN**: 声明 TableGen def 记录 `_InterlockedOr64_nf`。
- **L278 EN**: Declares TableGen def record `_InterlockedOr64_rel`.
  **L278 CN**: 声明 TableGen def 记录 `_InterlockedOr64_rel`。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 281-300

````tablegen
let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedXor8_acq  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedXor8_nf   : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedXor8_rel  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedXor16_acq : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedXor16_nf  : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedXor16_rel : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedXor_acq   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedXor_nf    : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedXor_rel   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedXor64_acq : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedXor64_nf  : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedXor64_rel : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedAnd8_acq  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedAnd8_nf   : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedAnd8_rel  : AArch64NoPrefixTargetLibBuiltin<"char (char volatile *, char)">;
	def _InterlockedAnd16_acq : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
````
- **L281 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L281 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L282 EN**: Declares TableGen def record `_InterlockedXor8_acq`.
  **L282 CN**: 声明 TableGen def 记录 `_InterlockedXor8_acq`。
- **L283 EN**: Declares TableGen def record `_InterlockedXor8_nf`.
  **L283 CN**: 声明 TableGen def 记录 `_InterlockedXor8_nf`。
- **L284 EN**: Declares TableGen def record `_InterlockedXor8_rel`.
  **L284 CN**: 声明 TableGen def 记录 `_InterlockedXor8_rel`。
- **L285 EN**: Declares TableGen def record `_InterlockedXor16_acq`.
  **L285 CN**: 声明 TableGen def 记录 `_InterlockedXor16_acq`。
- **L286 EN**: Declares TableGen def record `_InterlockedXor16_nf`.
  **L286 CN**: 声明 TableGen def 记录 `_InterlockedXor16_nf`。
- **L287 EN**: Declares TableGen def record `_InterlockedXor16_rel`.
  **L287 CN**: 声明 TableGen def 记录 `_InterlockedXor16_rel`。
- **L288 EN**: Declares TableGen def record `_InterlockedXor_acq`.
  **L288 CN**: 声明 TableGen def 记录 `_InterlockedXor_acq`。
- **L289 EN**: Declares TableGen def record `_InterlockedXor_nf`.
  **L289 CN**: 声明 TableGen def 记录 `_InterlockedXor_nf`。
- **L290 EN**: Declares TableGen def record `_InterlockedXor_rel`.
  **L290 CN**: 声明 TableGen def 记录 `_InterlockedXor_rel`。
- **L291 EN**: Declares TableGen def record `_InterlockedXor64_acq`.
  **L291 CN**: 声明 TableGen def 记录 `_InterlockedXor64_acq`。
- **L292 EN**: Declares TableGen def record `_InterlockedXor64_nf`.
  **L292 CN**: 声明 TableGen def 记录 `_InterlockedXor64_nf`。
- **L293 EN**: Declares TableGen def record `_InterlockedXor64_rel`.
  **L293 CN**: 声明 TableGen def 记录 `_InterlockedXor64_rel`。
- **L294 EN**: Closes the current lexical scope or compound statement.
  **L294 CN**: 结束当前词法作用域或复合语句块。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L296 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L296 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L297 EN**: Declares TableGen def record `_InterlockedAnd8_acq`.
  **L297 CN**: 声明 TableGen def 记录 `_InterlockedAnd8_acq`。
- **L298 EN**: Declares TableGen def record `_InterlockedAnd8_nf`.
  **L298 CN**: 声明 TableGen def 记录 `_InterlockedAnd8_nf`。
- **L299 EN**: Declares TableGen def record `_InterlockedAnd8_rel`.
  **L299 CN**: 声明 TableGen def 记录 `_InterlockedAnd8_rel`。
- **L300 EN**: Declares TableGen def record `_InterlockedAnd16_acq`.
  **L300 CN**: 声明 TableGen def 记录 `_InterlockedAnd16_acq`。

### Lines 301-320

````tablegen
	def _InterlockedAnd16_nf  : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedAnd16_rel : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *, short)">;
	def _InterlockedAnd_acq   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedAnd_nf    : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedAnd_rel   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *, msint32_t)">;
	def _InterlockedAnd64_acq : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedAnd64_nf  : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
	def _InterlockedAnd64_rel : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *, long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedIncrement16_acq : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *)">;
	def _InterlockedIncrement16_nf  : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *)">;
	def _InterlockedIncrement16_rel : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *)">;
	def _InterlockedIncrement_acq   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *)">;
	def _InterlockedIncrement_nf    : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *)">;
	def _InterlockedIncrement_rel   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *)">;
	def _InterlockedIncrement64_acq : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
	def _InterlockedIncrement64_nf  : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
	def _InterlockedIncrement64_rel : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
````
- **L301 EN**: Declares TableGen def record `_InterlockedAnd16_nf`.
  **L301 CN**: 声明 TableGen def 记录 `_InterlockedAnd16_nf`。
- **L302 EN**: Declares TableGen def record `_InterlockedAnd16_rel`.
  **L302 CN**: 声明 TableGen def 记录 `_InterlockedAnd16_rel`。
- **L303 EN**: Declares TableGen def record `_InterlockedAnd_acq`.
  **L303 CN**: 声明 TableGen def 记录 `_InterlockedAnd_acq`。
- **L304 EN**: Declares TableGen def record `_InterlockedAnd_nf`.
  **L304 CN**: 声明 TableGen def 记录 `_InterlockedAnd_nf`。
- **L305 EN**: Declares TableGen def record `_InterlockedAnd_rel`.
  **L305 CN**: 声明 TableGen def 记录 `_InterlockedAnd_rel`。
- **L306 EN**: Declares TableGen def record `_InterlockedAnd64_acq`.
  **L306 CN**: 声明 TableGen def 记录 `_InterlockedAnd64_acq`。
- **L307 EN**: Declares TableGen def record `_InterlockedAnd64_nf`.
  **L307 CN**: 声明 TableGen def 记录 `_InterlockedAnd64_nf`。
- **L308 EN**: Declares TableGen def record `_InterlockedAnd64_rel`.
  **L308 CN**: 声明 TableGen def 记录 `_InterlockedAnd64_rel`。
- **L309 EN**: Closes the current lexical scope or compound statement.
  **L309 CN**: 结束当前词法作用域或复合语句块。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L311 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L311 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L312 EN**: Declares TableGen def record `_InterlockedIncrement16_acq`.
  **L312 CN**: 声明 TableGen def 记录 `_InterlockedIncrement16_acq`。
- **L313 EN**: Declares TableGen def record `_InterlockedIncrement16_nf`.
  **L313 CN**: 声明 TableGen def 记录 `_InterlockedIncrement16_nf`。
- **L314 EN**: Declares TableGen def record `_InterlockedIncrement16_rel`.
  **L314 CN**: 声明 TableGen def 记录 `_InterlockedIncrement16_rel`。
- **L315 EN**: Declares TableGen def record `_InterlockedIncrement_acq`.
  **L315 CN**: 声明 TableGen def 记录 `_InterlockedIncrement_acq`。
- **L316 EN**: Declares TableGen def record `_InterlockedIncrement_nf`.
  **L316 CN**: 声明 TableGen def 记录 `_InterlockedIncrement_nf`。
- **L317 EN**: Declares TableGen def record `_InterlockedIncrement_rel`.
  **L317 CN**: 声明 TableGen def 记录 `_InterlockedIncrement_rel`。
- **L318 EN**: Declares TableGen def record `_InterlockedIncrement64_acq`.
  **L318 CN**: 声明 TableGen def 记录 `_InterlockedIncrement64_acq`。
- **L319 EN**: Declares TableGen def record `_InterlockedIncrement64_nf`.
  **L319 CN**: 声明 TableGen def 记录 `_InterlockedIncrement64_nf`。
- **L320 EN**: Declares TableGen def record `_InterlockedIncrement64_rel`.
  **L320 CN**: 声明 TableGen def 记录 `_InterlockedIncrement64_rel`。

### Lines 321-340

````tablegen
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _InterlockedDecrement16_acq : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *)">;
	def _InterlockedDecrement16_nf  : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *)">;
	def _InterlockedDecrement16_rel : AArch64NoPrefixTargetLibBuiltin<"short (short volatile *)">;
	def _InterlockedDecrement_acq   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *)">;
	def _InterlockedDecrement_nf    : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *)">;
	def _InterlockedDecrement_rel   : AArch64NoPrefixTargetLibBuiltin<"msint32_t (msint32_t volatile *)">;
	def _InterlockedDecrement64_acq : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
	def _InterlockedDecrement64_nf  : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
	def _InterlockedDecrement64_rel : AArch64NoPrefixTargetLibBuiltin<"long long int (long long int volatile *)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _ReadWriteBarrier       : AArch64NoPrefixTargetLibBuiltin<"void ()">;
	def __getReg                : AArch64NoPrefixTargetLibBuiltin<"unsigned long long int (int)">;
	def _ReadStatusReg          : AArch64NoPrefixTargetLibBuiltin<"long long int (int)">;
	def _WriteStatusReg         : AArch64NoPrefixTargetLibBuiltin<"void (int, long long int)">;
	def __sys                   : AArch64NoPrefixTargetLibBuiltin<"unsigned int (int, long long int)">;
````
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L323 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L324 EN**: Declares TableGen def record `_InterlockedDecrement16_acq`.
  **L324 CN**: 声明 TableGen def 记录 `_InterlockedDecrement16_acq`。
- **L325 EN**: Declares TableGen def record `_InterlockedDecrement16_nf`.
  **L325 CN**: 声明 TableGen def 记录 `_InterlockedDecrement16_nf`。
- **L326 EN**: Declares TableGen def record `_InterlockedDecrement16_rel`.
  **L326 CN**: 声明 TableGen def 记录 `_InterlockedDecrement16_rel`。
- **L327 EN**: Declares TableGen def record `_InterlockedDecrement_acq`.
  **L327 CN**: 声明 TableGen def 记录 `_InterlockedDecrement_acq`。
- **L328 EN**: Declares TableGen def record `_InterlockedDecrement_nf`.
  **L328 CN**: 声明 TableGen def 记录 `_InterlockedDecrement_nf`。
- **L329 EN**: Declares TableGen def record `_InterlockedDecrement_rel`.
  **L329 CN**: 声明 TableGen def 记录 `_InterlockedDecrement_rel`。
- **L330 EN**: Declares TableGen def record `_InterlockedDecrement64_acq`.
  **L330 CN**: 声明 TableGen def 记录 `_InterlockedDecrement64_acq`。
- **L331 EN**: Declares TableGen def record `_InterlockedDecrement64_nf`.
  **L331 CN**: 声明 TableGen def 记录 `_InterlockedDecrement64_nf`。
- **L332 EN**: Declares TableGen def record `_InterlockedDecrement64_rel`.
  **L332 CN**: 声明 TableGen def 记录 `_InterlockedDecrement64_rel`。
- **L333 EN**: Closes the current lexical scope or compound statement.
  **L333 CN**: 结束当前词法作用域或复合语句块。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L335 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L335 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L336 EN**: Declares TableGen def record `_ReadWriteBarrier`.
  **L336 CN**: 声明 TableGen def 记录 `_ReadWriteBarrier`。
- **L337 EN**: Declares TableGen def record `__getReg`.
  **L337 CN**: 声明 TableGen def 记录 `__getReg`。
- **L338 EN**: Declares TableGen def record `_ReadStatusReg`.
  **L338 CN**: 声明 TableGen def 记录 `_ReadStatusReg`。
- **L339 EN**: Declares TableGen def record `_WriteStatusReg`.
  **L339 CN**: 声明 TableGen def 记录 `_WriteStatusReg`。
- **L340 EN**: Declares TableGen def record `__sys`.
  **L340 CN**: 声明 TableGen def 记录 `__sys`。

### Lines 341-360

````tablegen
	def _AddressOfReturnAddress : AArch64NoPrefixTargetLibBuiltin<"void * ()">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def __mulh  : AArch64NoPrefixTargetLibBuiltin<"signed long long int (signed long long int, signed long long int)">;
	def __umulh : AArch64NoPrefixTargetLibBuiltin<"unsigned long long int (unsigned long long int, unsigned long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def __break : AArch64NoPrefixTargetLibBuiltin<"void (int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def __writex18byte  : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, unsigned char)">;
	def __writex18word  : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, unsigned short)">;
	def __writex18dword : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, msuint32_t)">;
	def __writex18qword : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, unsigned long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
````
- **L341 EN**: Declares TableGen def record `_AddressOfReturnAddress`.
  **L341 CN**: 声明 TableGen def 记录 `_AddressOfReturnAddress`。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L344 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L344 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L345 EN**: Declares TableGen def record `__mulh`.
  **L345 CN**: 声明 TableGen def 记录 `__mulh`。
- **L346 EN**: Declares TableGen def record `__umulh`.
  **L346 CN**: 声明 TableGen def 记录 `__umulh`。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L349 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L350 EN**: Declares TableGen def record `__break`.
  **L350 CN**: 声明 TableGen def 记录 `__break`。
- **L351 EN**: Closes the current lexical scope or compound statement.
  **L351 CN**: 结束当前词法作用域或复合语句块。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L353 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L354 EN**: Declares TableGen def record `__writex18byte`.
  **L354 CN**: 声明 TableGen def 记录 `__writex18byte`。
- **L355 EN**: Declares TableGen def record `__writex18word`.
  **L355 CN**: 声明 TableGen def 记录 `__writex18word`。
- **L356 EN**: Declares TableGen def record `__writex18dword`.
  **L356 CN**: 声明 TableGen def 记录 `__writex18dword`。
- **L357 EN**: Declares TableGen def record `__writex18qword`.
  **L357 CN**: 声明 TableGen def 记录 `__writex18qword`。
- **L358 EN**: Closes the current lexical scope or compound statement.
  **L358 CN**: 结束当前词法作用域或复合语句块。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L360 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。

### Lines 361-380

````tablegen
	def __readx18byte  : AArch64NoPrefixTargetLibBuiltin<"unsigned char (msuint32_t)">;
	def __readx18word  : AArch64NoPrefixTargetLibBuiltin<"unsigned short (msuint32_t)">;
	def __readx18dword : AArch64NoPrefixTargetLibBuiltin<"msuint32_t (msuint32_t)">;
	def __readx18qword : AArch64NoPrefixTargetLibBuiltin<"unsigned long long int (msuint32_t)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def __addx18byte  : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, unsigned char)">;
	def __addx18word  : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, unsigned short)">;
	def __addx18dword : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, msuint32_t)">;
	def __addx18qword : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t, unsigned long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def __incx18byte  : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t)">;
	def __incx18word  : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t)">;
	def __incx18dword : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t)">;
	def __incx18qword : AArch64NoPrefixTargetLibBuiltin<"void (msuint32_t)">;
}

````
- **L361 EN**: Declares TableGen def record `__readx18byte`.
  **L361 CN**: 声明 TableGen def 记录 `__readx18byte`。
- **L362 EN**: Declares TableGen def record `__readx18word`.
  **L362 CN**: 声明 TableGen def 记录 `__readx18word`。
- **L363 EN**: Declares TableGen def record `__readx18dword`.
  **L363 CN**: 声明 TableGen def 记录 `__readx18dword`。
- **L364 EN**: Declares TableGen def record `__readx18qword`.
  **L364 CN**: 声明 TableGen def 记录 `__readx18qword`。
- **L365 EN**: Closes the current lexical scope or compound statement.
  **L365 CN**: 结束当前词法作用域或复合语句块。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L367 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L367 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L368 EN**: Declares TableGen def record `__addx18byte`.
  **L368 CN**: 声明 TableGen def 记录 `__addx18byte`。
- **L369 EN**: Declares TableGen def record `__addx18word`.
  **L369 CN**: 声明 TableGen def 记录 `__addx18word`。
- **L370 EN**: Declares TableGen def record `__addx18dword`.
  **L370 CN**: 声明 TableGen def 记录 `__addx18dword`。
- **L371 EN**: Declares TableGen def record `__addx18qword`.
  **L371 CN**: 声明 TableGen def 记录 `__addx18qword`。
- **L372 EN**: Closes the current lexical scope or compound statement.
  **L372 CN**: 结束当前词法作用域或复合语句块。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L374 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L374 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L375 EN**: Declares TableGen def record `__incx18byte`.
  **L375 CN**: 声明 TableGen def 记录 `__incx18byte`。
- **L376 EN**: Declares TableGen def record `__incx18word`.
  **L376 CN**: 声明 TableGen def 记录 `__incx18word`。
- **L377 EN**: Declares TableGen def record `__incx18dword`.
  **L377 CN**: 声明 TableGen def 记录 `__incx18dword`。
- **L378 EN**: Declares TableGen def record `__incx18qword`.
  **L378 CN**: 声明 TableGen def 记录 `__incx18qword`。
- **L379 EN**: Closes the current lexical scope or compound statement.
  **L379 CN**: 结束当前词法作用域或复合语句块。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 381-400

````tablegen
let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _CopyDoubleFromInt64 : AArch64NoPrefixTargetLibBuiltin<"double (signed long long int)">;
	def _CopyFloatFromInt32  : AArch64NoPrefixTargetLibBuiltin<"float (signed int)">;
	def _CopyInt32FromFloat  : AArch64NoPrefixTargetLibBuiltin<"signed int (float)">;
	def _CopyInt64FromDouble : AArch64NoPrefixTargetLibBuiltin<"signed long long int (double)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def _CountLeadingOnes    : AArch64NoPrefixTargetLibBuiltin<"unsigned int (msuint32_t)">;
	def _CountLeadingOnes64  : AArch64NoPrefixTargetLibBuiltin<"unsigned int (unsigned long long int)">;
	def _CountLeadingSigns   : AArch64NoPrefixTargetLibBuiltin<"unsigned int (signed msint32_t)">;
	def _CountLeadingSigns64 : AArch64NoPrefixTargetLibBuiltin<"unsigned int (signed long long int)">;
	def _CountLeadingZeros   : AArch64NoPrefixTargetLibBuiltin<"unsigned int (msuint32_t)">;
	def _CountLeadingZeros64 : AArch64NoPrefixTargetLibBuiltin<"unsigned int (unsigned long long int)">;
	def _CountOneBits        : AArch64NoPrefixTargetLibBuiltin<"unsigned int (msuint32_t)">;
	def _CountOneBits64      : AArch64NoPrefixTargetLibBuiltin<"unsigned int (unsigned long long int)">;
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def __prefetch : AArch64NoPrefixTargetLibBuiltin<"void (void const *)">;
````
- **L381 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L381 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L382 EN**: Declares TableGen def record `_CopyDoubleFromInt64`.
  **L382 CN**: 声明 TableGen def 记录 `_CopyDoubleFromInt64`。
- **L383 EN**: Declares TableGen def record `_CopyFloatFromInt32`.
  **L383 CN**: 声明 TableGen def 记录 `_CopyFloatFromInt32`。
- **L384 EN**: Declares TableGen def record `_CopyInt32FromFloat`.
  **L384 CN**: 声明 TableGen def 记录 `_CopyInt32FromFloat`。
- **L385 EN**: Declares TableGen def record `_CopyInt64FromDouble`.
  **L385 CN**: 声明 TableGen def 记录 `_CopyInt64FromDouble`。
- **L386 EN**: Closes the current lexical scope or compound statement.
  **L386 CN**: 结束当前词法作用域或复合语句块。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L388 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L388 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L389 EN**: Declares TableGen def record `_CountLeadingOnes`.
  **L389 CN**: 声明 TableGen def 记录 `_CountLeadingOnes`。
- **L390 EN**: Declares TableGen def record `_CountLeadingOnes64`.
  **L390 CN**: 声明 TableGen def 记录 `_CountLeadingOnes64`。
- **L391 EN**: Declares TableGen def record `_CountLeadingSigns`.
  **L391 CN**: 声明 TableGen def 记录 `_CountLeadingSigns`。
- **L392 EN**: Declares TableGen def record `_CountLeadingSigns64`.
  **L392 CN**: 声明 TableGen def 记录 `_CountLeadingSigns64`。
- **L393 EN**: Declares TableGen def record `_CountLeadingZeros`.
  **L393 CN**: 声明 TableGen def 记录 `_CountLeadingZeros`。
- **L394 EN**: Declares TableGen def record `_CountLeadingZeros64`.
  **L394 CN**: 声明 TableGen def 记录 `_CountLeadingZeros64`。
- **L395 EN**: Declares TableGen def record `_CountOneBits`.
  **L395 CN**: 声明 TableGen def 记录 `_CountOneBits`。
- **L396 EN**: Declares TableGen def record `_CountOneBits64`.
  **L396 CN**: 声明 TableGen def 记录 `_CountOneBits64`。
- **L397 EN**: Closes the current lexical scope or compound statement.
  **L397 CN**: 结束当前词法作用域或复合语句块。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L399 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L399 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L400 EN**: Declares TableGen def record `__prefetch`.
  **L400 CN**: 声明 TableGen def 记录 `__prefetch`。

### Lines 401-405

````tablegen
}

let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {
	def __hlt : AArch64NoPrefixTargetLibBuiltin<"unsigned int (unsigned int, ...)">;
}
````
- **L401 EN**: Closes the current lexical scope or compound statement.
  **L401 CN**: 结束当前词法作用域或复合语句块。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L403 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`.
  **L403 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, RequireDeclaration], Languages = "ALL_MS_LANGUAGES", Header = "intrin.h" in {`。
- **L404 EN**: Declares TableGen def record `__hlt`.
  **L404 CN**: 声明 TableGen def 记录 `__hlt`。
- **L405 EN**: Closes the current lexical scope or compound statement.
  **L405 CN**: 结束当前词法作用域或复合语句块。

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
- **Functions or callables / 函数或可调用对象**: `void`, `int`, `uint64_t`, `code`, `Extensions`, `int64_t`, `int32_t`, `__int128_t`, `float`, `double`, `char`, `msint32_t`
- **TableGen records / TableGen 记录**: `__clear_cache`, `ldrex`, `ldaex`, `strex`, `stlex`, `clrex`, `rbit`, `rbit64`, `cls`, `cls64`, `clz`, `clz64`, `nop`, `yield`, `wfe`, `wfi`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
